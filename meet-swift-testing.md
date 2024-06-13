# Meet Swift Testing

Presenter: Stuart Montgomery, Developer Tools

Link: https://developer.apple.com/wwdc24/10179

- Designed for Swift
- Supports all major platforms
- Open development process (open source)

## Building blocks

When you add a new unit test bundle, Swift Testing is now the default testing system (instead of XCTest) in Xcode 16

`import Testing`
`@testable import DestinationVideo` (import target)

### Test attribute

Write a function with any name
Add the `@Test` attribute, indicates it's a test
Functions can be global functions or methods in a type, can be global or isolated to an actor, etc.

### Expect macro

`#expect` macro to validate that an expected condition is true
accepts ordinary expressions and operators
captures source code and subexpression values upon failure

### Require macro

`#require` macro
Use the try keyword, stop the test if something fails
Unwrap an optional value safely and stop the test if it's nil

```swift
let method = try #require(paumentMethods.first)

#expect(method.isDefault)
```

### Traits

- Add descriptive information about a test
- Customize whether a test runs
- Modify how a test behaves

#### Display name

Can add a custom display name to tests that displays in the test navigator

```swift
@Test("Check video metadata") func videoMetadata() {
    // Test stuff here
}
```

#### Reference an issue from a bug tracker

```swift
@Test(.bug("example.com/issues/99999", "Title")) func videoMetadata() {
    // Test stuff here
}
```

#### Add a custom tag to a test

```swift
@Test(.tags(.critical)) func videoMetadata() {
    // Test stuff here
}
```

#### Specify a runtime condition

```swift
@Test(.enabled(if: Server.isOnline)) func videoMetadata() {
    // Test stuff here
}
```

#### Unconditionally disable a test

```swift
@Test(.disabled("Check video metadata")) func videoMetadata() {
    // Test stuff here
}
```

#### Limit a test to certain OS versions

```swift
@Test(...) @available(macOS 15, *) func videoMetadata() {
    // Test stuff here
}
```

#### Set a maximum time limit for a test

```swift
@Test(.timeLimit(.minutes(3))) func videoMetadata() {
    // Test stuff here
}
```

#### Run the tests in a suite one at a time, without paralellization

```swift
@Suite(.serialized)
```

### Test Suite

- Can group together tests by wrapping them in a named struct
- Can run them as a group
- A type that contains tests
- Annotated using `@Suite`
- Implicit for types containing `@Test` functions or suites
- May have stored instance properties
- Use init and deinit for set-up and tear-down logic respectively

## Common workflows

### Tests with conditions

- Specify a runtime-evaluated condition for a test using `.enabled(if: ...)`
- Test will be skipped if condition is false

```swift
@Test(.enabled(if: AppFeatures.isCommentingEnabled)) func videoCommenting() {
    // Test stuff here
}
```

- Unconditionally disable a test usin `.disabled(...)`
- Comment is shown in results
- Confirm that the code still compiles but don't run the test


Use `@available` instead of checking at runtime

### Tests with common characteristics

- Assign custom tags to tests

```swift
@Test(.tags(.formatting)) func videoMetadata() {
    // Test stuff here
}
```

- Can group related tests into a sub-suite and move the tag up to the suite

```swift
@Suite(.tags(.formatting))
struct MetadataPresentation {
    // Tests that use the same state can have that state moved out into the suite
    // Each test runs in paralell with its own version of the state
    let video = Video(fileName: "By the Lake.mov")

    @Test func videoMetadata() {
        // Test stuff here
    }

    @Test func formattedDuration() {
        // Test stuff here
    }
}
```

Associate tests which have things in common to:
- Run all tests with a specific tag
- Filter by tag or see insights in Test Report

Shared among tests anywhere in a project or shared among multiple products
Prefer tags over test names when including/excluding from a test plan
Use the most appropriate trait type

Check out:
Go further with Swift testing

### Tests with different arguments

Consolidate a bunch of similar tests into a parameterized test

#### Before

```swift
@Test func mentionsFor_By_the_Lake() async throws {
    let videoLibrary = try await VideoLibrary()
    let video = try #require(await videoLibrary.video(named: "By the Lake"))
    #expect(!videomentionedContinents.isEmpty)
    #expect(video.mentionedContinents.count <= 3>)
}

@Test func mentionsFor_Camping_in_the_Woods() async throws {
    let videoLibrary = try await VideoLibrary()
    let video = try #require(await videoLibrary.video(named: "Camping in the Woods"))
    #expect(!videomentionedContinents.isEmpty)
    #expect(video.mentionedContinents.count <= 3>)
}

...etc...
```

#### After

- Add the argument to the test func function
- Add arguments to the `@Test`
- Replace the test-specific name with the passed-in argument

```swift
@Test(arguments: [
    "A Beach",
    "By the Lake",
    "Camping in the Woods"
])
func mentionedContinentCounts(videoName: String) async throws {
    let videoLibrary = try await VideoLibrary()
    let video = try #require(await videoLibrary.video(named: videoName))
    #expect(!videomentionedContinents.isEmpty)
    #expect(video.mentionedContinents.count <= 3>)
}
```

- Can run an individual argument by clicking its name in the test navigator
- Conceptually, a parameterized test is similar to a test that happens multiple times using a for... in loop
- Parameterized testing allows you to see details of each arguments in results
- Re-run individual arguments to debug
- Run arguments in paralell

Check out:
Go further with Swift Testing

## Swift Testing and XCTest

### Test Function

- Test name - in XCTest, names must begin with `test` but in Swift Testing you use `@Test` annotation
- XCTest supports instance methods, Swift testing supports instance methods, static/class methods, and global functions
- XCTest doesn't support traits, but Swift Testing does
- XCTest does multi-process testing for macOS and Simulator only, while Swift Testing supports paralell execution in-process with swift concurrency and also supports devices

### Expectations

XCTest uses assertions with many options, Swift uses only `#expect` and `#require` with standard language options

### Suites

- XCTest only supports classes, Swift Testing supports structs, actors, and classes
- XCTest must subclass XCTestCase, Swift has the `@Suite` attribute and implicit typing when a type contains tests
- XCTest has several setup func options, Swift uses `init()` and `async` `throws` variations
- XCTest has several teardown func options, Swift uses `deinit` (but deinit can only be used with explicit `@Suite` attribute annotation)
- XCTest doesn't support subgroups, but Swift test supports them via type nesting

### Migrating

- Can share a single unit test target
- Consolidate similar XCTests into a parameterized test
- Migrate each XCTest class with only one test method to a global @Test function
- Remove redundant `test` from names

Continue using XCTests for:
- UI automation APIs, such as XCUIApplication
- Performance testing APIs, such as XCTMetric
- Tests that can only be written in Objective-C

Check out:
[Migrating a test from XCTest](https://developer.apple.com/documentation/testing/migratingfromxctest) in the Apple developer documentation

## Open source

- New package, open source
- Currently in the Apple org, but will soon transition to the Swift lang organization
  - [github.com/apple/swift-testing](https://github.com/apple/swift-testing)
- Works on Apple platforms that support Swift concurrency, also Linux and Windows
- Common codebase when moving between platforms, better functional area
- Integrated in SPM, Xcode 16, VS Code in recent versions of the Swift extension
