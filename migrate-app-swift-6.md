# Migrate your app to Swift 6

Presenter: Ben Cohen, Swift Team

Link: https://developer.apple.com/wwdc24/10169

- Swift 6 language mode introduces full data race safety
- Opt-in for both existing and new projects
- Useful if you're experiencing hard-to-reproduce crashes
- Check out Swift 6 adoption in popular packages on swiftpackageindex.com
  - https://swiftpackageindex.com/ready-for-swift-6
- Data race safety lets you keep new code free of bugs

## Xcode 16 & Swift 6 Language Mode

- Download the latest Xcode, build. Shouldn't have to do any changes.
- Enable Swift 6 language mode. Compiler will guide you toward fixing issues.


## Step-by-step migration

Migrate each target individually

- Enable complete checking
- Fix errors
- Enable Swift 6 mode
- Audit unsafe opt-outs
- Avoid refactoring AND data race safety fixes. Do it one at a time.

### Enable complete checking

- Warnings about concurrency issues that came up as you adopt concurrency features
- Can use `nonisolated(unsafe)` keyword if you are safeguarding things in other ways, such as using a dispatch queue
- In Swift, global variables are lazily initialized on first use.
- Use show callers to see call sites
- Lots of examples for fixing different types of common concurrency issues

