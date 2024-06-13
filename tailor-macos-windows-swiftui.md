# Tailor macOS windows with SwiftUI

Presenter: Haotian Zheng, SwiftUI Engineer

Link: https://developer.apple.com/wwdc24/10148

New SwiftUI APIs to tailor your macOS windows

Check out:
Work with Windows in SwiftUI

## Three main windows in example app structure

```swift
@main
struct DestinationVideo: App {
    var body: some Scene {
        WindowGroup(id: "catalog") {...}

        Window("About Destination Video", id: "about") {...}

        WindowGroup("Video Player", id: "player") {...}
    }
}
```

## Style Toolbars

Remove title:

```swift
.toolbar(removing: title)
```

Remove toolbar background:

```swift
.toolbarBackgroundVisibility(.hidden, for: .windowToolbar)
```

## Refine window behavior

Add container background:

```swift
.containerBackground(.thickMaterial, for: .window)
```

Customize the minimize behavior:

```swift
.windowMinimizeBehavior(.disabled)
```

Customize the restoration behavior:

```swift
.restorationBehavior(.disabled)
```

## Adjust placement

Default placement:

```swift
.defaultWindowPlacement { content, context in
    var size = content.sizeThatFits(.unspecified)
    let displayBounds = context.defaultDisplay.visibleRect
    // modify size based on display bounds
    return WindowPlacement(size: size)
}
```

Ideal placement:

```swift
.windowIdealPlacement { content, context in
    var size = content.sizeThatFits(.unspecified)
    let displayBounds = context.defaultDisplay.visibleRect
    // modify size based on display bounds
    return WindowPlacement(size: size)
}
```

## Borderless window

```swift
.windowStyle(.plain)
```

## Default launch behavior

```swift
.defaultLaunchBehavior(.presented)
```
