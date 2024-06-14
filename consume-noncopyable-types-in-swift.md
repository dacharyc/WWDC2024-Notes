# Consume noncopyable types in Swift

Presenter: Kavon Farvardin, Swift Team

Link: https://developer.apple.com/wwdc24/10170

Values in Swift are not unique - they can be copied
Having a guarantee that values are unique can be a powerful concept when programming
We've introduced noncopyable types to Swift!

## Copying

- Structs are value types
  - When you copy a variable, you're copying its contents
- If an object is instead a reference type (i.e. class)
  - You can still copy it, but you're copying the reference instead of the value of the type (sometimes called a shallow copy)
  - Changing the value changes the value of the object that the reference is pointing to, which changes it for all references
  - You can make a reference type behave like a value type by defining an initializer to do a deep copy
  - Copy-on-write gives you independence under mutation, so you get the same behavior as a value type
- When designing a new type, you have control over whether someone can deeply copy its values. But you haven't been able to control whether Swift can make an automatic copy of it
- `Copyable` is a new protocol that describes the ability for a type to be automatically copied
- Like `Sendable`, it does not have member requirements
- Everything is intended to be `Copyable` in Swift by default. You don't have to make an object conform to `Copyable`. It's implicitly `Copyable`.
- When copying makes your code error-prone (bank transfer used as example) - consider making it a non-copyable type

## Noncopyable types

To make a type non-copyable, suppress the automatic copyable conformance like so:

```swift
struct FloppyDisk: ~Copyable { }
```

If you try to copy it, this isn't supported, so Swift conumes the variable, instead
- Takes the value, leaving the variable uninitialized
- Using the uninitialized variable after consuming it is an error

With copyable parameters, calling a function that uses a parameter of the type, you don't have to care about ownership because we can just copy it
With noncopyable parameters, calling a function that uses a noncopyable parameter can't just copy it, so we need to account for ownership

### Consuming

- Take ownership of the argument away from its caller
- It belongs to the function, so the function can mutate it
- The function in the example doesn't return anything, so consuming isn't the right call

```swift
struct FloppyDisk: ~Copyable { }

func newDisk() -> FloppyDisk {
  let result = FloppyDisk()
  format(result)
  return result
}

func format(_ disk: consuming FloppyDisk) {
  // ...
}
```

### Borrowing

In this example, the function doesn't need permanent ownership of it, so we could consider
- Borrowing gives you read access to the argument, like a `let` binding

```swift
struct FloppyDisk: ~Copyable { }

func newDisk() -> FloppyDisk {
  let result = FloppyDisk()
  format(result)
  return result
}

func format(_ disk: borrowing FloppyDisk) {
  var tempDisk = disk
  // ...
}
```

### Inout

For the example, we need to be able to mutate the argument, so we need inout (mutating) ownership
- Because you have write access, you can consume the parameter
- You must reinitialize the parameter at some point before the function ends, because the caller expects a value there when you return

```swift
struct FloppyDisk: ~Copyable { }

func newDisk() -> FloppyDisk {
  var result = FloppyDisk()
  format(&result)
  return result
}

func format(_ disk: inout FloppyDisk) {
  var tempDisk = disk
  // ...
  disk = tempDisk
}
```

## Generics

Conformance constraints describe generic types
By default, `T` conforms to `Copyable`
You can define additional conformance for types that need to be consumable or have other forms of ownership transfer

```swift
protocol Runnable: ~Copyable {
    consuming func run()
}

extension BankTransfer: Runnable {}

func execute<T>(_ t: consuming T) where T: Runnable, T: ~Copyable {
    t.run()
}
```

Conditionally copyable:

```swift
struct Job<Action: Runnable & ~Copyable>: ~Copyable {
    var action: Action?
}

extension Job: Copyable where Action: Copyable {}

func runEndlessly(_ job: consuming Job<Command>) {
    while true {
        let current = copy job
        current.action?.run()
    }
}
```

## Extensions

Extensions default to things where type is Copyable
- Any generic parameters in scope of the extended type are constrained to Copyable
- That includes Selfg in a protocol

```swift
extension Job {
  func getAction() -> Action? {
    return action
  }
}

func inspectCmd(_ cmdJob: Job<Command>) {
  let _ = cmdJob.getAction()
  let _ = cmdJob.getAction()
}

func inspectXfer(_ transferJob: borrowing Job<BankTransfer>) {
  let _ = transferJob.getAction() // expected error: method 'getAction' requires that 'BankTransfer' conform to 'Copyable'
}


struct Job<Action: Runnable & ~Copyable>: ~Copyable {
  var action: Action?
}

extension Job: Copyable where Action: Copyable {}

protocol Runnable: ~Copyable {
  consuming func run()
}

struct Command: Runnable {
  func run() { /* ... */ }
}

struct BankTransfer: ~Copyable, Runnable {
  consuming func run() { /* ... */ }
}
```

We can implement cancellable for jobs with Copyable actions

```swift
protocol Cancellable {
  mutating func cancel()
}

extension Job: Cancellable {
  mutating func cancel() {
    action = nil
  }
}
```

Or cancellable for all jobs, including jobs where type is noncopyable

```swift
protocol Cancellable: ~Copyable {
  mutating func cancel()
}

extension Job: Cancellable where Action: ~Copyable {
  mutating func cancel() {
    action = nil
  }
}
```
