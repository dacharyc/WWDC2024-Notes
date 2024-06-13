# Demystify SwiftUI containers

Presenter: Matt Ricketson, SwiftUI Engineer

Link: https://developer.apple.com/wwdc24/10146

SwiftUI provides many kinds of full-featured container views
SwiftUI use view builders with trailing closures to build content
Use new APIs to build custom container views

## Example app

### Data-driven DisplayBoard

```swift
@State private var songs: [Song] = [
    Song("Scrolling in the Deep"),
    Song("Born to Build & Run"),
    Song("Some Body Like View")
]

var body: some View {
    DisplayBoard(songs) { song in
        Text(song.title)
    }
}
```

### DisplayBoard implementation

```swift
var data: Data
@ViewBuilder var content: (Data.Element) -> Content

var body: some View {
    DisplayBoardCardLayout {
        ForEach(data) { item in
            CardView {
                content(item)
            }
        }
    }
    .background { BoardBackgroundView() }
}
```

## Composition

List composition

```swift
List(songsFromSam) { song in
    Text(song.title)
}
```

```swift
List {
    Text("Scrolling in the Deep")
    Text("Born to Build & Run")
    Text("Some Body Like View")
}
```

You can rewrite the data driven list using a for each view

```swift
List {
    ForEach(songsFromSam) { song in
        Text(song.title)
    }
}
```

You can compose them together to form a single view of all the song ideas:

```swift
List {
    Text("Scrolling in the Deep")
    Text("Born to Build & Run")
    Text("Some Body Like View")

    ForEach(songsFromSam) { song in
        Text(song.title)
    }
}
```

To support flexible composition in the DisplayBoard implementation, have to refactor. Use the new `ForEach(subviewOf:)` API:

```swift
@ViewBuilder var content: Content

var body: some View {
    DisplayBoardCardLayout {
        ForEach(subviewOf: content) { subview in
            CardView {
                content(subview)
            }
        }
    }
    .background { BoardBackgroundView() }
}
```

Then, you can replace the `List` with the custom view type:

```swift
DisplayBoard {
    Text("Scrolling in the Deep")
    Text("Born to Build & Run")
    Text("Some Body Like View")

    ForEach(songsFromSam) { song in
        Text(song.title)
    }
}
```

- A subview is a view contained within another view
  - Declared subviews: explicitly defined
  - Resolved subviews: all views, including result of defined views - i.e. all of ForEach's values
- The new API iterates over the resolved subviews

To scale down the size of the cards when there are too many resolved views, you can use another new API - `Group(subviewsOf:)`:

```swift
@ViewBuilder var content: Content

var body: some View {
    DisplayBoardCardLayout {
        Group(subviewsOf: content) { subviews in
            ForEach(subviews) { subview in
                CardView(
                    scale: subviews.count > 15 ? .small : .normal
                ) {
                    content(subview)
                }
            }
        }
    }
    .background { BoardBackgroundView() }
}
```

- If the total number of cards is more than 15, use a smaller size

## Sections

A list is an example of a built-in container that supports sections:

```swift
List {
    Section("Favorite Songs") {
        Text("Scrolling in the Deep")
        Text("Born to Build & Run")
        Text("Some Body Like View")
    }

    Section("Other Songs") {
        ForEach(songsFromSam) { song in
            Text(song.title)
        }
    }
}
```

Goal for the display board is to create a separate section for each person's favorite songs:

```swift
DisplayBoard {
    Section("Matt's Favorites") {
        Text("Scrolling in the Deep")
        Text("Born to Build & Run")
        Text("Some Body Like View")
    }

    Section("Sam's Favorites") {
        ForEach(songsFromSam) { song in
            Text(song.title)
        }
    }

    Section("Sommer's Favorites") {
        ForEach(songsFromSommer) { song in
            Text(song.title)
        }
    }
}
```

Custom containers don't support sections by default, so have to do some extra work to design.
- Factor out card layout view into its own view
- Wrap each section into a horizontal stack for dividing into new layout
- Use another new API, `ForEach(sectionOf:)`

```swift
@ViewBuilder var content: Content

var body: some View {
    HStack(spacing: 80) {
        ForEach(sectionOf: content) { section in
            DisplayBoardSectionContent {
                section.content
            }
            .background { BoardSectionBackgroundView() }
        }
    }
    .background { BoardBackgroundView() }
}

struct DisplayBoardSectionContent<Content: View>: View {
    @ViewBuilder var content: Content
    ...
}
```

Then we can add section headers:
- Wrap each section's content in a VStack
- Add a header

```swift
@ViewBuilder var content: Content

var body: some View {
    HStack(spacing: 80) {
        ForEach(sectionOf: content) { section in
            VStack(spacing: 20) {
                if !section.header.isEmpty {
                    DisplayBoardSectionHeaderCard { section.header }
                }
                DisplayBoardSectionContent {
                    section.content
                }
                .background { BoardSectionBackgroundView() }
            }
        }
    }
    .background { BoardBackgroundView() }
}

struct DisplayBoardSectionContent<Content: View>: View {
    @ViewBuilder var content: Content
    ...
}
```

## Customization

Container values (stay contained to a container)

To create a new custom container value:
- Extend the new `ContainerValues` type
- Use the new `@Entry` macro
- New custom view modifier

```swift
@Entry var isDisplayBoardCardRejected: Bool = false

extension View {
    func displayBoardCardRejected(_ isRejected: Bool) -> some View {
        containerValue(\.isDisplayBoardCardRejected, isRejected)
    }
}
```

Then customize the view to use the new view modifier:

```swift
struct DisplayBoardSectionContent<Content: View>: View {
    @ViewBuilder var content: Content
    
    var body: some View {
        DisplayBoardCardLayout {
            Group(subviewsOf: content) { subviews in
                ForEach(subviews) { subview in
                    let values = subview.containerValues
                    CardView(
                        scale: (subviews.count > 15) ? .small : .normal,
                        isRejected: values.isDisplayBoardCardRejected
                    ) {
                        subview
                    }
                }
            }
        }
    }
}
```

Now apply the view modifier to apply a custom decorator:

```swift
DisplayBoard {
    Section("Matt's Favorites") {
        Text("Scrolling in the Deep")
            .displayBoardCardRejected(true)
        Text("Born to Build & Run")
        Text("Some Body Like View")
    }

    Section("Sam's Favorites") {
        ForEach(songsFromSam) { song in
            Text(song.title)
                .displayBoardCardRejected(song.samHasDibs)
        }
    }

    Section("Sommer's Favorites") {
        ForEach(songsFromSommer) { song in
            Text(song.title)
        }
    }
    .displayBoardCardRejected(true)
}
```

## Next steps

- Use `ForEach` and `Group` to access subviews and sections
- Opt in to support sections
- Use container values to customize and decorate the individual pieces of content
