# What's new in Xcode 16

Presenters:
- Daisy Hernandez, Xcode Previews Engineer
- Jake Petroules, Build Experience Manager

Link: https://developer.apple.com/wwdc24/10135

## Edit

### Code completion

- More thorough code suggestions
- On-device coding models specifically trained for Swift and Apple SDKs
- Available running Xcode 16 on macOS Sequoia

### Swift 6

- New language mode, concurrency stuff
- Compile-time diagnosis of data race issues
- Incrementally enable warnings for each upcoming language feature
  - In Build Settings, navigate to "Upcoming" - this shows Swift 6 features
  - You can enable them one-by-one to diagnose and fix issues incrementally

Check out:
Migrate your ap to Swift 6

### Previews

Two new APIs:
- @Previewable: macro that can be attached to property wrappers to use them directly in a preview. You don't have to wrap them in container views.
- PreviewModifier: new protocol that makes it easier to share environments or data for Previews. Enables Preview system to cache data.
  - Use makeSharedContext to asynchronously load data
  - Use body method to wrap preview with shared context
  - Can define an extension on PreviewTrait to use the PreviewModifier at each call site

## Build

### Explicit modules

- Improved parallelism
- Better diagnostics
- Faster debugging
- For C/Objective-C: On by default
- Swift: opt-in
  - In Build Settings, enable "Explicitly Built Modules"

- In Xcode 16, don't have to wait for SPM to finish loading packages

- Xcode splits up the processing of each compilation unit into three distinct phases:
  - Scan
  - Build modules
  - Build source

Check out:
Demystify explicitly built modules

## Debug

- Faster debugging with explicit modules
- Smaller, faster debug symbols with DWARF5 on macOS Sequoia and iOS 18
- Thread performance checker now also adds disk write diagnostics and launch diagnostics
- New Unified Backtrace view that lets you follow the call stack (icon in bottom bar looks like three stacked horizontal lines)

- New RealityKit debugger
- View entities

Check out:
- Break into the RealityKit debugger
- Run, break, and inspect: Explore effective debugging in LLDB

## Test

Swift Testing
- New framework uses Swift language features to make testing more powerful and precise
- Can name function whatever you want, just add the `@Test` macro
- Expect macro takes any boolean expression `#expect(plant == expected)`
- View results to see values
- Use Quick Actions (cmd + shift + a) to rerun specific test
- Provide arguments to a test function
- Instead of having multiple test functions, have a single test with arguments to test multiple states
- Tag-based organization to group tests across different suites
  - Extend the `Tag` type to add a custom tag - i.e.:
    ```swift
    extension Tag {
        @Tag static var planting: Self
    }
    ```
  - Add tag to Test macro
  - You can use tags to include or exclude tests from test plans

Check out:
- Meet Swift Testing
- Go further with Swift Testing

## Profile

Diagnose performance problems with Instruments, which you can access from the Profile action in Xcode

- You can set a range to look at a portion of the data
- Use a blame graph to spot issues at a glance
- Left side of graph always visualizes code that is executed the most
- Blame graphs work for every instrument that uses call trees

Check out release notes or download Xcode 16 and give it a try!
