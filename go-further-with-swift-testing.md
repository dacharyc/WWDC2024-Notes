# Go Further with Swift Testing

Presenters:
- Jonathan Grynspan, Developer Tools
- Dorothy Fu, Developer Tools

Link: https://developer.apple.com/wwdc24/10195

## Challenges in testing

- Readability
- Code coverage
- Organization
- Fragiity

## Readability

### Expressive code

- Expectations check for expected values and outcomes in tests

#### Expect failing tests

- expect throws macro

##### Check for any error to be thrown

- If the test throws an error, it passes. If it does not throw an error, the test fails.

```swift
import Testing

@Test func brewTeaError() throws {
    let teaLeaves = TeaLeaves(name: "EarlGrey", optimalBrewTime: 4)
    #expect(throws: (any Error).self) {
        try teaLeaves.brew(forMinutes: 200)
    }
}
```

##### Check for a specific error to be thrown

- If the test throws the `BrewingError` error type, it passes. If the test does not throw an error, or if it throws the wrong type of error, it fails.

```swift
import Testing

@Test func brewTeaError() throws {
    let teaLeaves = TeaLeaves(name: "EarlGrey", optimalBrewTime: 4)
    #expect(throws: BrewingError.self) {
        try teaLeaves.brew(forMinutes: 200)
    }
}
```

Or in a more extreme case...

- Check for specific types
- Cases of errors
- Associated values or properties are correct

```swift
import Testing

@Test func brewTeaError() throws {
    let teaLeaves = TeaLeaves(name: "EarlGrey", optimalBrewTime: 4)
    #expect {
        try teaLeaves.brew(forMinutes: 3)
    } throws: {
        guard let error = error as? BrewingError,
            case let .needsMoreTime(optimalBrewTime) = error else {
          return false
        }
        return optimalBrewTime == 4
    }
}
```

#### Required expectations

In this case, it doesn't make sense to examine a value that ends up being nil. So you can end the test early if the requirement is not met.

```swift
import Testing

@Test func brewTea() throws {
    let teaLeaves = TeaLeaves(name: "Sencha", optimalBrewTime: 2)
    let brewedTea = try teaLeaves.brew(forMinutes: 2)
    let color = try #require(brewedTea.color)
    #expect(color == .green)
}
```

##### Known issues

- With known issue function
- Use this instead of disabling tests
- Shows as an expected failure in test inspector, doesn't add noise to tests
- Continues to verify the code compiles, etc.

```swift
import Testing

@Test func softServeIceCreamInCone() throws {
    withKnownIssue {
        try softServeMachine.makeSoftServe(in: .cone)
    }
}
```

Good option when a test is checking more than one thing, just wrap the code that you're expecting to fail in the withKnownIssue function

```swift
import Testing

@Test func softServeIceCreamInCone() throws {
    let iceCreamBatter = IceCreambatter(flavor: .chocolate)
    try #require(iceCreamBatter != nil)
    #expect(iceCreamBatter.flavor == .chocolate)

    withKnownIssue {
        try softServeMachine.makeSoftServe(in: .cone)
    }
}
```

##### Custom test descriptions

- Tests fail. Custom test descriptions let you see at a glance what's going on inside a test and guide you to a solution
- Call out important bits that distinguish one value from another
- Make the type conform to `CustomTestStringConvertible` and provide a custom `testDescription`

```swift
struct SoftServe: CustomTestStringConvertible {
    let flavor: Flavor
    let container: Container
    let toppings: [Topping]

    var testDescription: String {
        "\(flavor) in a \(container)"
    }
}
```

With this, the test navigator and test report displays "chocolate in a cone" instead of the entire struct's properties and values

### Parameterized testing

Run tests under various conditions to make sure you catch all the edge cases
Run a single test function with many different arguments
Test cases run independently in parallel
Re-run individual test cases when the type of input conforms to `Codable`

```swift
import Testing

extension IceCream {
    enum Flavor {
        case vanilla, chocolate, strawberry, mintChip, rockyRoad, pistachio
    }
}

@Test(arguments: [IceCream.Flavor.vanilla, .chocolate, .strawberry, .mintChip]) 
func doesNotContainNuts(flavor: IceCream.Flavor) throws {
    try #require(!flavor.containsNuts)
}

@Test(arguments: [IceCream.Flavor.rockyRoad, .pistachio]) 
func containsNuts(flavor: IceCream.Flavor) throws {
    try #require(flavor.containsNuts)
}
```

Parameter input types can be any Sendable collection
- Array
- Set
- OptionSet
- Dictionary
- Range

You can add more than one argument:

```swift
import Testing

enum Ingredient: CaseIterable {
    case rice, potato, lettuce, egg
}

enum Dish: CaseIterable {
    case onigiri, fries, salad, omelette
}

@Test(arguments: Ingredient.allCases, Dish.allCases) 
func cook(_ ingredient: Ingredient, into dish: Dish) throws {
    #expect(ingredient.isFresh)
    let result = try cook(ingredient)
    try #require(result.isDelicious)
    try #require(result == dish)
}
```

- Test functions accept a maximum of two collections
- You can use `zip()` in parameterized testing
- Pair each element in your first collection with its counterpart in second collection, and nothing else

```swift
import Testing

enum Ingredient: CaseIterable {
    case rice, potato, lettuce, egg
}

enum Dish: CaseIterable {
    case onigiri, fries, salad, omelette
}

@Test(arguments: zip(Ingredient.allCases, Dish.allCases)) 
func cook(_ ingredient: Ingredient, into dish: Dish) throws {
    #expect(ingredient.isFresh)
    let result = try cook(ingredient)
    try #require(result.isDelicious)
    try #require(result == dish)
}
```

### Organizing tests

- Suites contain test functions
- You can document them with traits
- Suites can contain other suites
- Add subsuites to group related tests together
- Tag traits to connect related functionality across files or suites
  - Not a replacement for suites. Don't impose structure, but do let you associate tests with one another

#### Create a tag

Extend the Tag type

```swift
extension Tag {
    @Tag static var caffienated: Self
}
```

Use it in suites or tests

```swift
import Testing

// Add the tag to a suite, and all of its child tests inherit the tag
@Suite(.tags(.caffienated)) struct DrinkTests {
    // Tests
}

// Or you can add the tag to individual tests
@Test(.tags(.caffienated)) func espressoBrownieTexture() throws {
    // Test stuff here
}
```

Tests can have more than one tag

#### Use tags in Xcode

- Use the Filter field at the bottom of the test navigator to find tests related to tags
- If you click a tag in the test navigator, it removes all tests that don't have that tag
- You can switch the default hierarchical view to view by tags instead using the Tag icon at the top right of the test navigator
- You can run all tests for a tag, or run individual tests
- You can add tags to your test plan
  - Include or exclude tags from the test plan
  - You can match "all" or "any" tags
- You can analyze results for tags across test targets
  - Tags appear in the test report
  - You can filter by tags in the test report
  - There's a new section in "Insights" that can surface things related to tags

Xcode Cloud supports Swift Testing
- Youc an view tests related to tags in Xcode Cloud

### Testing in parallel

- Parallel testing is enabled by default in Swift Testing
- The order in which tests run is random (on purpose)
- Swift 6 can help you find some problems with existing code
- Convert existing code to Swift Testing and come back and fix later

#### Run code serially

- Add the serialized trait to indicate that tests need to be run serially

```swift
import Testing

// Add the tag to a suite, and all of its child tests inherit the tag
@Suite("Cupcake tests", .serialized) 
struct CupcakeTests {
    // Tests
}
```

#### Async code

- Code that has `await` can be suspended, so you can't rely on code with a completion handler having what it needs if it comes after an `await`
- Swift Testing provides an awaitable overload for continuations to allow most continuations to be used awync/await style
- If that doesn't work for your case, you can use `withCheckedContinuation` and friends

```swift
import Testing

@Test func bakeCookies() async throws {
    let cookies = await Cookie.bake(count: 10)
    try await withCheckedThrowingContinuation { continuation in
        if let result {
            cotninuation.resume(returning: result)
        } else if let error {
            continuation.resume(throwing: error)
        }
    }
}
```

To invoke a callback more than once, use a confirmation with an expected count:

```swift
import Testing

@Test func bakeCookies() async throws {
    let cookies = await Cookie.bake(count: 10)
    try await confirmation("Ate cookies", expectedCount: 10) { ateCookie in
        try await eat(cookies, with: .milk) { cookie, crumbs in
            #expect(!crumbs.in(.milk))
            ateCookie()
        }
    }
}
```
