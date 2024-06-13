# Go small with Embedded Swift

Presenter: Kuba Mracek, Swift Compiler Engineer

Link: https://developer.apple.com/wwdc24/10197

## Why Embedded Swift?

- A new compilation mode suited for resource-constrained devices
- C and C++ were historically the programming languages of choice, adding Swift as an option
- Embedded devices
- Kernel-level code
- Low-level library code
- Used on Secure Enclave Processor in Apple devices

It's a language subset covering most of Swift

## Showcase

- Currently experimental, not source stable
- swift.org nightly builds
- Building a simple HomeKit accessory: an LED light
- Use a mac to connect to the device with a cable, write something that you can flash the device with
- Using Neovim and CMake
- Using a 3rd party SDK written in C that he got from device vendor
- Bridging headers
- Editor has LSP so shows documentation, semantic autocomplete, errors and warnings
- Can call existing C APIs directly
- Wrap C APIs into a Swift layer for ergonomics

- Use Matter C++ APIs
- Demo project and setup available on GitHub
  - https://github.com/apple/swift-matter-examples

## How Embedded Swift differs

- Embedded Swift disallows certain features to fit on resource-constrained devices
- Reflection is disallowed
- Metatypes and "any" types are disallowed
- Prefer generics over "any" types

## Explore more

- There's a [vision document](https://github.com/swiftlang/swift-evolution/blob/main/visions/embedded-swift.md) for Embedded Swift as part of Swift evolution
- Check out the [Embedded Swift User Manual](https://github.com/apple/swift/blob/main/docs/EmbeddedSwift/UserManual.md)
- Take a look at [Embedded Swift example projects](https://github.com/apple/swift-embedded-examples)
- [Swift MMIO](https://github.com/apple/swift-mmio) to interact with memory-mapped registers
- Swift Forums have a new [Embedded Swift subcategory](https://forums.swift.org/c/development/embedded/)
