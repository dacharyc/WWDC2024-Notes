# What's new in Swift

Presenters:
- Mishal Shah, Swift Team
- Meghana Gupta, Swift Team

Link: https://developer.apple.com/wwdc24/10136

10 years since Swift was announced!

## Swift project update

- New platform steering group to focus on bringing Swift to more places
- Core team is working on putting together a new ecosystem steering group
- Also, a new embedded working group

Check it out on swift.org/community

- Guides and tutorials on the website
- Swift.org/packages to explore the package ecosystem

## Swift Everywhere

- Swift officially supported on Apple, Linux and Windows
- Additional supported platforms expanded to Fedora and Debian (and a bunch of other things)
- Developed SourceKit LSP to integrate Swift support
- Community has adopted in VSCode, Neovim, and more

### Cross compiling

- Introducing a static Linux SDK for Swift to build Linux binaries on macOS
- Demo of cross-compiling a service built for macOS for Linux

### Foundation

- Foundation
- swift-corelibs-foundation (for other platforms)
- Rewrote to a modern single codebase - swift-foundation
- Shipped in iOS and macOS last fall
- swift-foundation is open source

### Swift Testing

- Takes advantage of macros, integrates with concurrency
- Developed in open source with cross-platform in mind
- Vision is to become the official default testing solution

### Building

- Improvements to how Xcode builds your code
- Implicitly built modules
- Instead, adding explicitly built modules
- More parallelism, better visibility into build steps, improved reliability

## Swift is moving

github.com/swiftlang

Starting migration soon, details posted on swift.org

## Swift 6

- Noncopyable types
- Embedded Swift
- C++ interoperability
- Typed throws
- Data-race safety

### Noncopyable types

- All Swift types are copyable by default
- Noncopyable types let you express unique ownership
- Swift 6 introduces support for all generic contexts
- You can now write failable initializers for generic types (Optional, Result, Unsafe Pointers)
- Fine-grained control over performance

### Embedded Swift

- C and C++ have been choices for resource-constrained environments
- Embedded Swift is a new language subset - turns off certain language features that need runtime (reflection, "any" types)
- Special compiler techniques
- Embedded Swift is close to full Swift
- Brings memory safety to embedded systems
- Incremental adoption with Swift interoperability with C++

### C++ Interoperabililty

- Virtual methods, default arguments, move-only types can be directly imported into Swift
- Virtual & default mapped to equivalent Swift versions
- Move-only types mapped to noncopyable types
- Can incrementally adopt Swift in C++ projects

### Typed throws

- Swift 6 introduces typed throws to let you specify the error type
- No type erasure, the typed error appears
- You can continue using untyped throws to evolve APIs

### Data-race safety

- Swift 6 language mode
- Achieves data race safety by default, turning data race issues into compile-time errors
- Can adopt module-by-module and interoperate with dependencies that may not have updated
- Big improvements to data race safety
- Swift 6 recognizes that it's safe to pass non-Sendable values sometimes
- New Atomic low-level synchronization primitives
- Synchronization module also introduces Mutex

Check out:
Migrate your app to Swift 6
