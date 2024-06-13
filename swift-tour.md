# A Swift tour: explore Swift's features and design

Presenter: Allan Shortlidge, Swift Compiler Engineer

Link: https://developer.apple.com/wwdc24/10184

## Example app to show features

- Data model library
- HTTP server
- Command-line utility

## Value types

- Use `var` keyword to introduce variable
- What are value types?
  - No shared state
  - Equal values are interchangeable
  - Used for more than basic types in Swift
- Use `let` keyword for immutable types
- Structs aggregate multiple values into a single object
- Arrays are value types in Swift
- Most Swift types are value types
- Reference types like classes exist but are used in specialized cases

## Errors and optionals

- Code structure handles all possibilities
- The `throws` and `try` keywords make error-handling explicit
- Unwrapping optionals ensures that values exist

### Errors

- Sources of errors should be clearly marked
- Errors contain actionable information for the user
- Programmer mistakes are not recoverable errors
- Enums are great error types - they must conform to Error protocol
- Guard statements are a good way to halt execution when an error occurs

### Optionals

- A value that's either `nil` or some value
- Value must be unwrapped
- Compile-time safety
- `if let` syntax to unwrap optional
-  Force-unwrap with `!`

## Code organization

- Modules and packages
- Module is a collection of source files that are always built together
- Modules can have dependencies
- Swift Package Manager is a tool for managing packages
- Builds, tests, and runs code
- Swift Package Index has a list of open-source libraries

- Access control levels: private, internal, package, public

## Classes

- Reference types for shared mutable state
- Classes support inheritance
- Methods can be overridden by class implementation
- Automatic reference counting (ARC) - ensures an object remains alive as long as there is a reference to it
- When no more references, things deallocate
- Avoid reference cycles, which prevent dealloc
- Can use weak reference, which becomes optional, property becomes nil when thing is deallocated

## Protocols

- Abstract set of requirements for a type
- To conform to a protocol, it must provide implementations of protocol's requirements
- Use extensions to add stuff to a type regardless of where the type is defined
- Collections (array, dictionary, set, string)
- Every type that conforms to Collections shares features - i.e. iteration through a for loop, accessing an index
- Shorthand syntax for closures

## Concurrency

- Task is an independent concurrent execution context
- Tasks are lightweight
- Support awaiting/cancellation
- Tasks can execute concurrently
- When adding a UserStore, concurrency error related to shared mutable state not being Sendable
- Actors can encapsulate shared mutable state, and protect the state by serializing `Task` execution

## Extensibility

- Property wrappers
  - Reusable abstraction for property behaviors
  - Eliminate boilerplate via annotation
- Result builders
  - Build up values iusing a lightweight syntax
  - Create declarative sub-languages
- Macros
  - Swift code that act as a compiler plugin taking the syntax tree as input and returning transformed code as output
