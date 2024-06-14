# Analyze heap memory

Presenters:
- Ben Troller, Memory Tools Engineer
- Daniel Delwood, Runtime Tools Manager

Link: https://developer.apple.com/wwdc24/10173

Focus: Heap
- Memory allocated by `malloc()`, directly or indirectly
- Memory with the most developer control. Where your apps reference types are stored.
- Almost always dirty, counts against limit
- Focus on measuring and reducing heap memory.

## Measurement

- The heap is made up of multiple virtual memory regions
- Each region of memory is broken up into individual heap allocations
- Each region is made up of 16kb memory pages from the operating system, but each allocation can be bigger or smaller
- Memory pages can be in one of three states:
  - Clean: memory that hasn't been written to
  - Dirty: Memory that has been written to recently by the application
  - Swapped: when dirty pages haven't been used for a while, the system can swap them - compress them or write them to disk
- Only dirty and swapped count toward an application's memory footprint
- Developers don't call `malloc()` and friends directly, but the compiler and runtime use them
- malloc:
  - Lifetime beyond the current scope, until `free()`
  - 16-byte minimum quanta and alignment
  - Zeroes smaller blocks on free
- Language runtimes use the heap to allocate long-lived memory. Swift, for example, expands this class initializer to a series of runtime functions, which end up calling malloc:
  ```swift
  class ExampleClass {}
  let exampleInstance = ExampleClass()
  ```
- Malloc has debugging features:
  - MallocStackLogging: records call stacks and timestamps for each allocation
    - In Xcode, can enable with a checkbox in the scheme diagnostics tab

- Tracking memory usage:
  - Xcode memory report: shows an application's footprint over time
  - Memory Graph Debugger: can capture a snapshot of all allocations and the references between them
    - With MallockStackLogging, this includes backtraces for each allocation
  - Command-line tools for memory analysis. Leaks, heap, vmmap, malloc_history can analyze macOS and Simulator processes directly, or investigate issues using already-captured memory graphs.
  - Instruments: profiling memory use over time
    - Allocations: records the history of all allocation and free events over time
    - Leaks: takes periodic snapshots of your app's memory to detect memory leaks

### Investigate in Instruments

- Use the Product -> Profile menu item to perform a release build of the app and open Instruments with it selected as the target
- When Instruments opens, choose a template for profiling. Allocations.
  - Press record to run the app and record data
  - Save the trace, and you can share it with someone (File -> Save)

## Transient growth

- Memory spikes are a type of transient memory growth. Transient memory growth is bad because:
  - Sudden memory pressure
  - Out-of-memory crashes
  - Fragmentation
  - Terminating background tasks
  - Termination of your app

- Two ways to track transient growth:
  - Look at a specific spike to find the allocations Created & Still Living from the low point to the high point
  - In aggregate, select a large range and find all allocations that were Created & Destroyed in that range
- Autorelease pools can be a source of transient memory growth
- Implicit autorelease pools can have long lifetimes
- Fix is often to define an explicit autorelease pool to reduce the lifetime of the allocated objects

## Persistent growth

- Memory that doesn't get deallocated
- Persistent growth often resembles a stairstep pattern, with memory increasing over time
- The growth is made up of multiple allocations
- Use the Mark Generation feature in the Allocations instrument to break down the growth by timespan

- Types of references
  - Strong: definitely a pointer, explicit ownership
  - Weak/Unowned: definitely a pointer, explcit non-ownership
  - Unmanaged: probably a pointer, manually managed
  - Conservative: might be a pointer, tools must assume it is

## Leaked memory

- Memory leaks: in Instruments, allocations with a yellow triangle next to them
- Reachability:
  - Finding paths from roots to heap blocks
- Three types of memory on your heap:
  - Useful memory: reachable allocations that will be used again
  - Abandoned memory: reachable allocations that will not ever be used again
  - Leaked memory: unreachable memory that can't ever be used again - typically when the last pointer is lost, either to a manually managed allocation or a reference cycle of objects
- For most leaks, the goal is to find and fix one reference in a cycle
  - Removing an accidental reference
  - Changing ownership qualifier from strong to weak or unowned
- Instruments: use the "Show only leaked allocations" button in the filter bar
  - Filter to only project types by clicking the other filter button
- Common Swift runtime types
  - `Closure context`
    - Paired 1:1 with active closure
    - Reference qualifiers, no capture names
    - Closures capture references strongly by default, making it possible to create reference cycles
    - You can break these cycles using weak or unowned captures instead
      ```swift
      let swallow = Swallow()
      swallow.completion = {
          print("\(swallow) finished carrying a coconut")
      }
      ```

Why isn't a specific allocation shown as leaked?
- Leak scanning is conservative, meaning it allows uncertain references

How can the number of leaks go down over time?
- Conservative references aren't deterministic

Why is my `noreturn` function leaking?
- Compiler calls to `noreturn` or `-> Never` don't cleanup local references
- Can explicitly store this into a global that the reference tools can see

## Performance

### weak versus unowned

- Common tools to use in Swift to avoid creating strong reference cycles - but when to use each?

If not seeing weak or unowned references reported in the memory graph, check the project Reflection Metadata Level in Build settings
- Default to 'All' level if possible

![Safety and performance chart comparing weak versus unowned references](/images/weak-versus-unowned-reference-safety-and-performance.png)

#### weak references

Weak references are:
- Always optional types
- Become nil after their destinations are deinitialized
- You can always use a weak reference, regardless of source and destination lifetimes
- Does come with overhead - Swift allocates a Swift weak reference storage for the destination object, which sits between the object and all of its incoming weak references

#### unowned references

Unowned references:
- Directly hold their destinations
  - They don't use any extra memory, and take less time to access than weak references
- Can be non-optional
- Can be constant
- It's not always valid to use an unowned reference
  - If the object holding the unowned reference goes away before the reference, the object is deinitialized but not deallocated
  - The unowned reference must point to something, so the runtime keeps the object around
  - If you try to access the deinitialized object, you get a deterministic crash
    - In this way, unowned references are a lot like force-unwrapping weak references
  - As long as the unowned reference exists, the destination can't be deallocated and it wastes memory
  - If you don't know how long the destination will live, the small overhead of a weak reference is worthwhile

### Example

Implicit used of self by a method causing a reference cycle:

```swift
class ByteProducer {
  let data: Data
  private var generator: ((Data) -> UInt8)? = nil

  init(data: Data) {
    self.data = data
    generator = defaultAction // Implicitly uses `self`
  }

  func defaultAction(_ data: Data) -> UInt8 {
    // ...
  }
}
```

Break the reference cycle using a weak reference:

```swift
class ByteProducer {
  let data: Data
  private var generator: ((Data) -> UInt8)? = nil

  init(data: Data) {
    self.data = data
    generator = { [weak self] data in
      return self?.defaultAction(data)
    }
  }

  func defaultAction(_ data: Data) -> UInt8 {
    // ...
  }
}
```

Break the reference cycle using an unowned reference:

```swift
class ByteProducer {
  let data: Data
  private var generator: ((Data) -> UInt8)? = nil

  init(data: Data) {
    self.data = data
    generator = { [unowned self] data in
      return self.defaultAction(data)
    }
  }

  func defaultAction(_ data: Data) -> UInt8 {
    // ...
  }
}
```

### Reference counting

- Don't circumvent Automatic Reference Counting (ARC)
- Ensure `-whole-module-optimization` in Swift
- Fewer `any` boxes, retainable references in Swift structs
  ```swift
  struct Nontrivial {
      var number: Int64
      var simple: CGPoint?
      var complex: String // Copy-on-write, requires non-trivial struct init/copy/destroy
  }
  ```

## Next steps

Check out:
- Explore Swift performance
- Consume noncopyable types in Swift
