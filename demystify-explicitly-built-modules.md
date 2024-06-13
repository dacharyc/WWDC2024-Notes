# Demystify explicitly built modules

Presenter: Michael Spencer, Compiler Engineer

Link: https://developer.apple.com/wwdc24/10171

- New way that Swift and Clang modules built in Xcode

## Modules

- Units of code distribution
- Multiple Swift files that together represent a module
  - In general, all the Swift files in a single target or framework are part of the same module
  - Interface is explicitly marked with access specifiers
  - Public is visible to importers
  - Modules can import other modules
- Modules in the C interface are hand-modeled
  - Headers & module map
- Modules share the parsing of interfaces between source files
  - In Swift, this is represented as a `.swiftmodule`, while in Clang, it's represented as a `.pcm` or precompiled module file

## Using modules

- Implicitly built modules
  - Compilers coordinate among themselves to build modules
  - This is how Swift and Clang have built modules to date
- Xcode 16 changes to explicitly built modules
  - Takes the work of building modules and splits up the compilation of source files into three phases:
    - Scan
      - Construct a module graph across the entire project
    - Build modules
      - Explicit tasks in the build log directly provided with the compiled modules they depend on
    - Build source
      - Execution of the original compilation task after they have been modified to add the compiled modules they depend on

Benefits
- Builds are more reliable
- Clean build rebuilds modules
- More efficient builds as the build system is responsible for scheduling
- Build system can now pass modules to the debugger, speeding up startup

## Module build log

- In Xcode 16, C and Objective-C is explicitly built
- Can be enabled for Swift
  - Select the project in the project navigator
  - Select Build Settings
  - Type "explicitly built" in filter box
  - Set the setting to Yes

- Build log contains many scan tasks
- New compile module tasks - build system spawns a separate compiler process for each task
  - Diagnostics attach here
  - Some modules may be built multiple times when different build settings require different versions of a module to be built

## Optimize your build

- Remove extra variants of modules that are being built to reduce/eliminate those tasks

- Get info about module build tasks
  - Clean build folder to rebuild all modules in project
  - Go to Product -> Perform Action -> Build with Timing Summary to collect additional information about build performance
  - In the build log, go to Filter and type in "modules report"
  - Select the report - i.e. Clang modules report - and you can see variants for modules
  - Check build settings for common sources of module variants

- Common Sources of module variants
  - Macros
  - Language mode and version
  - ARC enabling/disabling

- In general, move build settings as broadly as possible within project to share modules between source files as much as possible
