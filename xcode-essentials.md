# Xcode essentials

Presenters:
- Cheech Minniear, Xcode Design Team
- Myke Savage, Engineering Project Manager

Link: https://developer.apple.com/wwdc24/10181

## Edit

### Find the right content

- Check out specialized filters at the bottom of navigators - i.e. project navigator or test navigator
- Find navigator
  - use bottom bar to narrow a search
  - Hold command key to see only filenames
  - Select and press delete to remove files from a find result - the files still exist but are hidden from the current query
  - Choose groups to focus the search
  - "Descendent types" to get a hierarchy view
  - Select text and press command + E to put text into the Find field without having to copy it and potentially overwrite what's in your clipboard
- To the left of the tab bar:
  - Use the Related Files menu (the one that looks like a bunch of squares) - it has recent files, but also other symbolic relationships, including callers of the current function
  - Hold down the back button to see all the recent files you've passed through
- Can filter menus
- Press command + shift + j to create a new file near the current file
- You can hold option and then click and drag a file to duplicate it in the dragged location
- You can create a new file with the contents of the clipboard

### Create your own warnings

Type `#warning` with a string to create your own warning:

```swift
#warning("Remove workaround before committing")
```

### Open Files

- Open quickly - cmd + shift + o, then type part of a file or symbol name
  - Hold option when pressing return to open the file in a new split
- Quick actions - cmd + shift + a, then do a search to find the action
- For quick help, option + click

### Multi-cursor editing

- This is a thing you can do, including vim support. I don't remember how. Look it up sometime.

## Debugging

- Use command + control + r to run the app again without building (like when it crashes)

### Breakpoint debugging

- Add a Swift error breakpoint to stop at an error throw instead of a catch
- Can double click a breakpoint and add a condition
- Can add debugger expressions at a breakpoint temporarily (from breakpoint editor) - like temporary print statements tied to the breakpoint

### Print statement debugging

- Can use Swift standard library macros in print statements for additional helpful info:
  - `#fileID`: a unique identifier for the source file in which the macro (print statement) appears
  - `#function`: the name of the declaration in which it appears
- Instead of print statements, consider switching to `os_log()`, which gives you a debug level for each message that you set

Check out:
- Run, Break, Inspect: Explore effective debugging in LLDB (WWDC24)
- Debug with structured logging (WWDC23)
- Debug Swift debugging with LLDB (WWDC22)

## Test

- Dig into test plans/schemes when projects grow

## Distribute

- Paid developer account includes TestFlight
