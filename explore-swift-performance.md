# Explore Swift performance

Presenter: John McCall, Swift Team

Link: https://developer.apple.com/wwdc24/10217

Swift abstractions have non-trivial implementations with costs that aren't as visible as an explicit call to malloc

## What is performance?

- Macroscopic goals:
  - Reduce latency
  - Reduce power consumption
  - Stay within memory limits

- Top-down performance evaluation
  - Measure, measure, measure
  - Identify hot spots
  - Often fixed with algorithmic improvements
    - Better asymptotic performance
    - Avoiding unnecessary or redundant work
- Bottom-up performance evaluation
  - Why are we spending so long in this function?
  - Why does this do an allocation, and what can we do about it?

- Principles of microscopic performance:
  - Function calls: doing a lot of calls that aren't being optimized effectively
  - Memory layout: wasting a lot of time or memory because of how our data is represented
  - Memory allocation: spending too much time allocating memory
  - Value copying: spending a lot of time unnecessarily copying and destroying values

## Low-level principles

### Function calls

Four costs associated with function calls:
1. Set up the arguments for the call
2. Resolve the address of the function we're calling
3. Allocate space for the function's local state
4. Optimization restrictions

![Breakdown of the low-level costs of function calls](/images/low-level-costs-of-function-calls.png)

#### Argument passing

- Calling convention: we have to put the arguments in the right place for the calling convention
- Compiler may have to add copies of values to match the ownership conventions of the function
  - This may show up in profiles as extra retains and releases

#### Call dispatch

Do we know at compile time exactly which function we're calling?
- If yes, we say the call uses static dispatch; if no, it uses dynamic dispatch
- Static (direct) dispatch is more efficient
  - Call always goes to a specific function definition
  - Compiler can inline, specialize, etc. if it can see that definition
- Dynamic (indirect or virtual) dispatch: enables polymorphism and other powerful tools for abstraction
  - Call can go to different function definitions
  - Compiler must make conservative assumptions about the call
  - Only specific kinds of calls use dynamic dispatch - everything else is static. Calls that use dynamic dispatch:
    - Opaque function values
    - Overridable class methods
    - Protocol requirements
    - Objective-C or virtual C++ methods

A Swift function that calls a method on a value of protocol type:

```swift
func updateAll(models: [any DataModel], from source: DataSource) {
    for model in models {
        model.update(from: source)
    }
}
```

A declaration of the method where it's a protocol requirement using dynamic dispatch:

```swift
protocol DataModel {
    func update(from source: DataSource)
}
```

A declaration of the method where it's a protocol extension method using static dispatch:

```swift
protocol DataModel {
    func update(from source: DataSource, quickly: Bool)
}

extension DataModel {
    func update(from source: DataSource) {
        self.update(from: source, quickly: true)
    }
}
```

#### Local memory

- Function needs memory to run
- Allocating on the C stack is done by subtracting from the stack pointer
  - Subtract some number from the stack pointer (208 bytes)
  - This is called the call frame, and gives the function some room to execute
  - Right before we return, we add 208 bytes back to the stack pointer to deallocate the memory we just allocated

This function:

```swift
func updateAll(models: [any DataModel],
               from source: DataSource) {
    for model in models {
        model.update(from: source)
    }
}
```

Becomes this call frame:

```C
// sizeof(CallFrame) == 208
struct CallFrame {
    Array<AnyDataModel> models;
    DataSource source;
    AnyDataModel model;
    ArrayIterator iterator;
    ...
    void *savedX29;
    void *savedX30;
};
```

The compiler will always emit the subtraction at the start of the function. Changing the size of the memory makes no difference. This makes it as close as possible to a free operation.

### Memory allocation

Three kinds of memory:
- Global
- Stack
- Heap

#### Global memory

- Allocated and initialized during program load
- Almost free
- Only usable for a fixed amount of memory that will never be freed
- Used for:
  - `let` and `var` declared at the global scope
  - `static` stored properties

#### Stack memory

- Allocated and freed by adjusting the C stack pointer
- Almost free
- Only usable for memory that does not need to outlive current scope
  - Local `let` and `var`
  - Parameters
  - Other temporary values

#### Heap

- Allocated and freed with malloc and free
- Substantially more expensive
- Used for:
  - Class and actor instances
  - Whenever we can't prove a scope restriction is ok
- Reference counting
  - Heap memory often has shared ownership
  - Managed with reference counting
    - In Swift, "retain" means incrementing the reference count
    - "release" means decrementing the reference count

### Memory layout

High-level values
- "value" is a high-level concept of information content
  ```swift
  var array = [ 1.0, 2.0 ]
  ```
Low-level representations
- "representation" is how a value looks in memory

![Visual representation of what a value looks like in memory, showing an array of two doubles and its ContiguousArrayStorage](/images/memory-layout-low-level-representation.png)

- "inline representation" is the inline portion of the representation (without following any pointers) - a single buffer reference without following any pointers, ignoring what the buffer actually contains
- The MemoryLayout type in the standard library just measures inline representation
  - For array, it's just 8 bytes - the size of a single 64-bit pointer

Every value in Swift is part of some containing context:
- Local scope (local variables, intermediate results of expressions, etc.)
- Instance context (non-static stored properties, etc.)
- Global context (global variables, static stored properties, etc.)
- Dynamic context (buffers managed by Array and Dictionary)

Types and representations
- Every value in Swift has a static type
- The rules of the type dictate the representation of the value
- The rules of the context provide the memory to hold the inline representation

Inline versus out-of-line storage
- Structs, enums, and tuples use inline storage
  - Inline representation includes the inline representations of *all* stored properties
- Classes and actors use out-of-line storage
  - Inline representation is just a pointer to an object
  - Stored properties are contained by that object

### Value ownership

Ownership of a value means responsibility for managing that value's representation

The ownership of using a value - using a value can:
- Consume it
- Mutate it
- Borrow it

#### Consuming values

- Takes ownership of the representation
  - Assigning a value into storage
  - Passing a value to a `consuming` parameter
  - Can use the `consume` operator to explicitly transfer a value to a new variable

#### Mutating values

- Temporarily taking ownership of the value of a mutable variable
  - Passing storage to an `inout` parameter
  - Mutating a stored property of a struct (recursively)

#### Borrowing values

- Asserting that nothing else has ownership of the value
  - Calling a normal method on a value or class reference
  - Passing a value to a normal or `borrowing` parameter
  - Reading the value of a property

##### Defensive copies

To borrow a value, Swift has to prove that there aren't any simultaneous attempts to mutate or consume it
- In a simple example, the compiler should be able to do that reliably

```swift
func makeArray() {
    var array = [ 1.0, 2.0 ]
    print(array)
}
```

- In a more complex example, such as when storage is in a class property, it can be hard for the compiler to know that the property isn't modified at the same time, so it may need to make a defensive copy

```swift
func makeArray(object: MyClass) {
    object.array = [ 1.0, 2.0 ]
    print(object.array)
}
```

#### Mechanics of copying

Copying a value means copying the inline representation
- For types using out-of-line storage (i.e. classes)
  - Copies (retains) the object reference
- For types using inline storage (i.e. structs)
  - Recursively copies the inline representation of all stored properties

Tradeoffs of inline versus out-of-line storage
- Inline storage:
  - Avoids heap allocations
  - Great for small types
  - Copies get more expensive the more properties you have
- No hard-and-fast rules for optimal performance
- Sometimes out-of-line storage is better for large objects

#### Value and reference semantics

In Swift, we encourage you to write types with value semantics, where a copy of the value behaves like it's unrelated to where you copied it from
- Structs behave this way, but use inline storage
- Class types use out-of-line storage, and naturally have reference semantics
- One way to get both out-of-line storage and value semantics is to wrap the class in a struct and then use copy-on-write
  - The standard library uses this technique in all of Swift's fundamental data structures, like Array, Dictionary, and String

## Putting it together

### Dynamically-sized types

Structs in C are always constant-size, but Swift types can have a size determined at runtime. That's important in two cases:
1. Many value types in the SDK reserve the right to change their stored properties in a future OS update. Everything about their layout has to be treated as unknown at compile time.
2. A type parameter of a generic type can usually be replaced by any type with any possible representation, so we have to treat its layout as unknown.
  - AN exception to this rule when a type parameter is constrained to be a class. It has to have the representation of a class type, which is always a pointer. This can lead to more efficient code.

How does Swift handle memory layout and allocation when teh compiler doesn't statically know the representation of a type?
- Depends on the type of container storign the value
- For most containers, Swift can do the layout at runtime
- Some containers must have constant size. In these cases, the compiler must allocate memory for the value separately from the main allocation for the container.
  - Global memory
  - Call frames
  - Container stores a pointer to the allocation

### Async functions

Designed to be able to abandon a C thread when they need to suspend
- Keep local state on a special stack
- Split functions into partial functions that run between suspensions

Async tasks hold onto one or more slabs of memory
- It asks the task for memory
- Stack tries to satisfy that from the current slab and give it to the function
- If most of the slab is allocated, that function may not fit
- The task then has to allocate a new slab with malloc

Performance is similar to sync functions, but with higher overhead for calls

### Closures

Closures are always passed around as values of function type
- Function values in Swift are always a pair of function pointer and a context pointer
- If we know the function value will not be used after the call completes, it does not need to be memory-managed and we can allocate the context with a scoped allocation
- For escaping closures, we don't know that the closure is only used within the duration of the call. Therefore, the context object must be heap-allocated and managed with retains and releases
  - Context essentially behaves like an instance of an anonymous Swift class
  - In an escaping closure, vars that are captured by an escaping closure must also be heap-allocated, and the closure context has to retain a reference to that object

### Generics

Any time we have a protocol constraint, we're passing around a pointer to the appropriate table
In a generic function, the type and witness tables become hidden extra parameters
When we work with values of protocol type, it's different
- The inline representation of a protocol has storage for the values and fields to record the value's type and any conformances we know it has
- This has to be a fixed-size type in its representation and can't change sizes in order to support different types of data model
- No matter how large we make the value storage, there's potentially going to be a data model taht won't fit into it
- If the value stored in a protocol type can fit in the default Swift buffer size of 3 pointers, the protocol is put there, inline
- If the value stored in a protocol type *can't* fit in the default Swift buffer size, it allocates space for the value on the heap and just stores the pointer in the buffer

Abstraction is powerful, and if that power is worth the cost for your application, go ahead and use the protocol type!
