# Run, Break, Inspect: Explore effective debugging in LLDB

Presenter: Felipe Piovezan, Debugging Technologies

Link: https://developer.apple.com/wwdc24/10198

LLDB is the underlying debugger that ships with Xcode

## Debugging Model

- Debugging as a search problem
- Somewhere between the start of the program execution, and the point in which the incorrect behavior is observed, faulty code was executed
- Our goal is to find that piece of code
- Inspecting the state of the program at different points in time brings us closer to the problematic code

Techniques:
- Log file
- Print debugging
- Debugger
  - Backtraces
  - Variable inspection
  - Breakpoints
  - Expression evaluation

## Starting the debug session

Debugger workflow:
- Run
- Break
- Inspect

To start a debug session:
- Press the start button in Xcode
- Using Terminal:
  `lldb -- <executable> <arguments>`

- Check the crash logs before you start debugging.
  - For Apps in the app store, you can access crash logs through the Organizer
  - If a crash log is sent, you can right-click the file and open with Xcode
- Opening a crash log in Xcode uses LLDB to create a debugging session with the state of the program at the time of the crash
  - The line of the crash is highlighted
  - You can follow the backgrace through the stack frames that led to the program state
    - Provides a view into what each function was doing, where they were called, and where they would return to
  - Current frame (where the crash occurs) appears at the top - follow the stack trace from top to bottom to go back through the sequence of frames
- Find the current backtrace in the Debug Navigator of Xcode

- Make sure th repo is on the same commit as the version of the app that created the crash log
- Make sure the dSYM bundle for the build is available
  Check out: Symbolication: beyond the basics (WWDC21)

## Breaking

- In the content view, there is an add to watch later button:

```swift
Button(action: { watchLater.toggle(video: video) }) {
  let inList = watchLater.isInList(video: video)
  Label(inList ? "In Watch Later" : "Add to Watch Later",
  systemImage: inList ? "checkmark" : "plus")
}
```

- Set a breakpoint by clicking on the relevant line number. Breakpoint is displayed on the breakpoint navigator
- When run the app, see 3 different things - may stop at this place through three different paths
  - Can get more information about the three breakpoints with the breakpoint list command:
    `(lldb) break list`
    - Describes the breakpoint we set on line 70, but also the 3 locations associated with that breakpoint using line and column numbers
    - The first in the list is where the program is currently stopped
    - lldb assigns an ID to each breakpoint location
      - This is the same ID used by Xcode when it highlights the breakpoint line
    - Second breakpoint in the list refers to the first argumnt of the constructor, the action closure. Gets triggered when we click the button.
    - Third location refers to the trailing closure in the constructor.
    - Continue program execution to try to hit the other locations. (Press Play)
  - Multiple code regions attributed to the same line is common in declarative code making heavy use of closures, like SwiftUI:
    - The call to the Button constructor
    - The trailing closure called by the constructor
    - The action closure which is called when the button is clicked
  - Adding a line breakpoint inside the closure's body is a good way to pause when it does
- Using LLDB, we can pause the 2nd and 3rd breakpoints and inspect the size of the watch list with the p command:
  ```console
  p watchLater.count
  p watchLater.last!.name
  ```
- Can add breakpoint actions to do things whenever we hit the breakpoint. For example, print the name of the most recently added video:
  ```console
  p "last video is \(watchLater.last?.name)"
  ```
  - Use "Edit breakpoints" menu by right-clicking a breakpoint
  - Add a debugger command action to do this thing
  - Can continue execution after hitting the breakpoint
- Can use breakpoint actions through the command line:
  ```console
  b DetailView.swift:70
  break command add
  p "last video is \(watchLater.last?.name)"
  continue
  DONE
  ```
  - Can get detailed descriptions of lldb commands:
    ```console
    (lldb) help <command>
    ```
    Or options:
    ```console
    (lldb) help <command> <option>
    ```
    To discover features:
    ```console
    (lldb) apropros <keyword>
    ```

### High-firing breakpoints

- When debugging, we often create a breakpoint that is triggered many times, but we are only interested in a subset of those
  - When a breakpoint is placed inside a loop and, instead of breaking on every iteration, we only want to break when certain events happen

Techniques for handling high-firing breakpoints:
- Set a line breakpoint and modify it with a breakpoint condition. Can do on command line or in Edit breakpoint UI.
  ```console
  (lldb) break modify <breakpoint ID> --condition "video.length > 60"
  ```
- Stop at a call only if we also executed a related call. Set a breakpoint but add a breakpoint action to create a temporary breakpoint. Can do on command line or in Edit breakpoint UI. Use tbreak command
  ```console
  tbreak Importer.swift:67
  ```
- Ignore breakpoints for a fixed number of times, stop only on subsequent hits. Can do on command line or in Edit breakpoint UI.
  ```console
  (lldb) break modify <breakpoint ID> --ignore-count 10
  ```
- In functions called lots of times (like thousands/millions), debugger must evaluate at every execution whether to pause or continue, and it can slow down dramatically. In this case, recompile code - compute the stop condition, and set a breakpoint inside an if statement that executes only when the condition is true. i.e. `raise` function with the `SIGSTOP` signal

```swift
for video in videos {
    if (/*break_condition*/) {
        raise(SIGSTOP)
    }
    if (video.hasRemoteMedia) {
        video.loadRemoteMedia()
    }
    processVideo()
}
```

## Inspect program state

- Lots of different commands, but `(lldb) p` is the right command for most situations
  Check out: Debug with structured logging (WWDC23)

- Use `p <variablename>` to inspect a variable while paused at a breakpoint
- Can do the same thing by expanding things in the variable viewer
- Can do the same thing as a hover display by mousing over the variable name in the code

- Can add a Swift Error breakpoint. This instructs LLDB to stop execution as soon as a Swift Error is thrown.
  - In the Breakpoint Navigator:
    - Press the `+` button in the bottom left corner of the navigator
    - Select `Swift Error`

- Types that contain too much data are cumbersome to inspect during a debug inspection
  - Have many properties
  - Are frequently stored inside collections
  - With Swift 6, the output of `p` can be customized: Debug description macro
    - Annotate the type with `@DebugDescription`
    - Provide a `debugDescription` string property

      Debug description macro example:

      ```swift
      // Type summaries
      @DebugDescription
      struct WatchLaterItem {
          let video: Video
          let name: String
          let addedOn: Date

          var debugDescription: String {
              "\(name) - \(addedOn)"
          }
      }
      ```
