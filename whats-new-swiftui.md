# What's New in SwiftUI

Presenters:
- Sam Lazarus, SwiftUI Engineer
- Sommer Panage, SwiftUI Engineer

Link: https://developer.apple.com/wwdc24/10144

## Freshen Up Apps

- New tab view
- Mesh gradient
- Snap view controls
- More flexible sidebar in iOS 18
  - Tab bar floats above the content
  - People can reorder and customize options
  - New TabView/Tab type-safe syntax
    `.tabViewStyle()`
    `.tabViewCustomization()`
  - Appear as a sidebar or presentation control
- New zoom navigation style

Check out 
- Improve your tab and sidebar experience on iPad
- Enhance your UI animations and transitions

### Controls

- New kind of widget  you can build with App Intents

`ControlWidgetButton`

Check out
- Access your app's controls across the system

### Charts

- Function plotting
- Vecrtorized plots

Check out:
Swift Charts: Vectorized and function plots

### Dynamic table columns

- Table column for each

### Gradient support

- Composing mesh gradients: `MeshGradient`

### Document-based apps

- New document launcher

Check out:
Evolve your document launch experience

### SF Symbol Effects

- New symbol effects
  - wiggle
  - breathe
  - rotate
  - replace

Check out:
What's new in SF Symbols

## Harness the platform

- Improved windowing
- Input methods

### Window styling

- Window
  - windowStyle()
  - windowLevel
  - defaultWindowPlacement
  - WindowDragGesture

Check out:
Tailor macoOS windows with SwiftUI

### visionOS

- Push window: open a window and hide the originating window
  pushWindow() environment action

Check out:
Work with windows in SwiftUI

## Input methods

- In visionOS, react to 
  - hoverEffect - customize how the view looks 

Check out:
Create custom hover effects in visionOS

### Keyboard support

- modifierKeyAlternate
- onModifierKeysChanged

### Pointer interaction

- pointerStyle API to customize

### Apple Pencil and Apple Pencil Pro

- onPencilSqueeze

Check out:
Squeeze the most out of Apple Pencil

## Widgets and Live Activities

### Live activities on watchOS

- supplementalActivityFamilies
- Double tap modifier
- Additional date format styles
- Widget relevance

## Framework foundations

### Cusstom containers

- For each (subviewOf)
- Make custom containers with same capabilities as SwiftUI
- Support sections, add container-specific modifiers

Check out:
Demystify SwiftUI Containers

### Ease of use

- Entry macro - extension on EnvironmentValues, FocusValues, Transaction, and new container
- Default accessibility label augmentation

Check out:
Catch up on accessibility in SwiftUI

- Use state directly in previews with `@Previewable` macro
- Programmatic access to text selection
- Check if a search field is focused, programmatically move focus
- Add text suggestions to any text box
- New graphics capabilities (combine colors)
- Precompile shaders before first use

### Scrolling enhancements

- React to scroll geometry changes
- React to content moving on or off screen
- More scroll positions to programmatically scroll to

### Swift 6 language mode support

- SwiftUI view protocol now marked with MainActor
- All types conforming to View now conform to MainActor by default
- Can remove MainActor annotation

Check out:
Migrate your app to Swift 6

### Improved interoperability

- Gesture interoperability
- APIs for bridging animation

Check out:
Enhance your UI animations and transitions

## Crafting experiences

### Improvements to volumes and immersive spaces

Check out:
Dive deep into volumes and immersive spaces

### Text views

- Custom text renderers

Check out:
Create custom visual effects with SwiftUI
