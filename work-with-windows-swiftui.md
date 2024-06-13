# Work with windows in SwiftUI

Presenter: Andrew Sawyer, SwiftUI Engineer

Link: https://developer.apple.com/wwdc24/10149

This session applies to multi-window platforms (macOS, visionOS)

## Fundamentals

- People can use system controls to manipulate windows
- Windows can have platform-specific features (visionOS, volumetric vision style)
- TabView can simplify the experience

### Example app structure

Example app has two primary scenes:
- Editor window
- Game

```swift
@main
struct BOTanistApp: App {
    var body: some Scene {
        WindowGroup(id: "editor") {
            EditorContentView()
        }

        WindowGroup(id: "game") {
            GameContentView()
        }
        .windowStyle(.volumetric)
    }
}
```

Adding a new feature - a 3D cut scene in a new window group.
- Create a new WindowGroup and give it the ID "movie"
- Can use the ID in an environment action to do something with the window

```swift
@main
struct BOTanistApp: App {
    var body: some Scene {
        WindowGroup(id: "editor") {
            EditorContentView()
        }

        WindowGroup(id: "game") {
            GameContentView()
        }
        .windowStyle(.volumetric)

        WindowGroup(id: "movie") {
            MovieContentView()
        }
    }
}
```

### Open a window

Opening a movie window:

```swift
struct EditorContentView: View {
    @Environment(\.openWindow) private var openWindow

    var body: some View {
        Button("Open Movie", systemImage: "tv") {
            openWindow(id: "movie")
        }
    }
}
```

To close a window, use `.dismissWindow()`

### Push a window

Push a movie window to open the new window and hide the originating window (visionOS only?):

```swift
struct EditorContentView: View {
    @Environment(\.pushWindow) private var pushWindow

    var body: some View {
        Button("Open Movie", systemImage: "tv") {
            pushWindow(id: "movie")
        }
    }
}
```

Use this action when the window doesn't need to be displayed at the same time as the original window

### Use a toolbar to display controls along the bottom edge of a window (visionOS)

```swift
CanvasView()
    .toolbar {
        ToolbarItem {
            Button(...)
        }
        ...
    }
```

### Use ToolbarTitleMenu to present actions related to a document without crowding the canvas (visionOS)

```swift
CanvasView()
    .toolbar {
        ToolbarTitleMenu {
            Button(...)
        }
        ...
    }
```

### Hide window controls

```swift
WindowGroup(id: "movie") {
    ...
}
.persistentSystemOverlays(.hidden)
```

## Placements

New feature: add an optional controller window. Add a new window:

```swift
@main
struct BOTanistApp: App {
    var body: some Scene {
        ...

        WindowGroup(id: "movie") {
            MovieContentView()
        }

        WindowGroup(id: "controller") {
            ControllerContentView()
        }
    }
}
```

Open it:

```swift
struct GameContentView: View {
    @Environment(\.openWindow) private var openWindow

    var body: some View {
        ...
        Button("Open Controller", systemImage: "gamecontroller.fill") {
            openWindow(id: "controller")
        }
    }
}
```

Position it:

```swift
WindowGroup(id: "controller") {
    ControllerContentView()
}
.defaultWindowPlacement { content, context in
    #if os(visionOS)
    return WindowPlacement(.utilityPanel)
    #elseif os(macOS)
    ...
    #endif
}
```

Or position it within display bounds:

```swift
WindowGroup(id: "controller") {
    ControllerContentView()
}
.defaultWindowPlacement { content, context in
    #if os(visionOS)
    return WindowPlacement(.utilityPanel)
    #elseif os(macOS)
    let displayBounds = context.defaultDisplay.visibleRect
    let size = content.sizeThatFits(.unspecified)
    let position = CGPoint(
        x: displayBounds.midX - (size.width / 2),
        y: displayBounds.maxY - size.height - 20
    )
    return WindowPlacement(position, size: size)
    #endif
}
```

## Sizing

Set a default size:

```swift
@main
struct BOTanistApp: App {
    var body: some Scene {
        ...
        WindowGroup(id: "movie") {
            MovieContentView()
        }
        .defaultSize(width: 1166, height: 680)
    }
}
```

Set limits on resizing the movie window:

```swift
@main
struct BOTanistApp: App {
    var body: some Scene {
        ...
        WindowGroup(id: "movie") {
            MovieContentView()
                .frame(
                    minWidth: 680, maxWidth: 2720,
                    minHeight: 680, maxHeight: 1020
                )
        }
        .windowResizability(.contentSize)
    }
}
```

Set different limits on resizing the controller window:

```swift
@main
struct BOTanistApp: App {
    var body: some Scene {
        ...
        WindowGroup(id: "controller") {
            ControllerContentView()
        }
        .windowResizability(.contentSize)
    }
}
```
