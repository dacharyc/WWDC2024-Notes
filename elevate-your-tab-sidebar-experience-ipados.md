# Elevate your tab and sidebar experience in iPadOS

Presenters: Andy Liang, UIKit Engineer

Link: https://developer.apple.com/wwdc24/10147

Enhancements to tab bar and sidebar
Tab view bar is at the top of the app, redesigned
Shares a space with the top bar
Tabs should be consistent beween iPad and iPhone for consistent navigation
Avoid adding too many tabs
Sidebars provide quick access to top-level destinations
When a sidebar is hidden, it animates back into the tab bar

Tab bar and sidebar:
- Represent the same hierarchy
- Content-focused apps

Sidebar supports customization features that allow people to show or hide tabs, reorder them, add tabs through drag-and-drop to personalize experience

## Tab bars and sidebars

- iPadOS 18 SDK updates existing tab bars to new look without any code changes
- Toolbar items appear inline with tab bar and move to overflow if there isn't enough room
- TabView has a new SwiftUI syntax
  - `Tab` has a string title and a system image
  - Optionally takes a value for programmatic navigation

```swift
TabView {
    Tab("Watch Now", systemImage: "play", value: .watchNow) {
        WatchNowView()
    }
    Tab("Library", systemImage: "books.vertical", value: .library) {
        LibraryView()
    }
    // ...
}
```

New UIKit UI, too, but I'm not building with UIKit so I'm not noting the details here!

UI guidelines:
- Tab bars prefer filled glyphs
- Sidebars prefer outline glyphs
- System automatically selects a fill variant when displayed in a tab bar

Search role tab configures a search field with a default:
- Title
- Image
- Pinned placement

```swift
// SwiftUI
Tab(role: .search) {
    SearchView()
}
```

To add a sidebar in SwiftUI:
(Search automatically displays before any groups)

```swift
TabView {
    Tab("Watch Now", systemImage: "play") {
        // ...
    }
    Tab("Library", systemImage: "books.vertical") {
        // ...
    }
    // ...
    TabSection("Collections") {
        Tab("Cinematic Shots", systemImage: "list.and.film") {
            // ...
        }
        Tab("Forest Life", systemImage: "list.and.film") {
            // ...
        }
        // ...
    }
    TabSection("Animations") {
        // ...
    }
    Tab(role: .search) {
        // ...
    }
}
.tabViewStyle(.sidebarAdaptable)
```

You can add actions to sections in the sidebar to add convenient access to common tasks:

```swift
TabSection(...) {
    // ...
}
.sectionActions {
    Button("New Station", ...) {
        // action
    }
}
```

Tabs are also drop destinations:

```swift
Tab(collection.name, image: collection.image) {
    CollectionDetailView(collection)
}
.dropDestination(for: Photo.self) { photos in
    // Add 'photos' to the specified collection
}
```

You can also customize sidebars:
- Header and footer
- Swipe actions
- Context menus
- Popover source item

## User customization

Sidebar customization:
- Allow hiding nonessential tabs
- Support reordering of groups
- Persisted automatically

Tab bar customization:
- Drag and drop tabs
- Tab placements

Tab bar contains three sections:
- Fixed section
- Customizable section
- Pinned section (search)

You can use a sidebar only placement to prevent something from being added to a tab bar

To enable user customization of a tab view in SwiftUI:

```swift
@AppStorage("MyTabViewCustomization")
private var customization: TabViewCustomization

TabView {
    Tab("Watch Now", systemImage: "play", value: .watchNow) {
        // ...
    }
    .customizationID("Tab.watchNow")
    // ...
    TabSection("Collections") {
        ForEach(MyCollectionsTab.allCases) { tab in
            Tab(...) {
                // ...
            }
            .customizationID(tab.customizationID)
        }
    }
    .customizationID("Tab.collections")
    // ...
}
.tabViewCustomization($customization)
```

To disable customization on individual tabs (i.e. make them fixed):

```swift
Tab("Watch Now", systemImage: "play", value: .watchNow) {
    // ...
}
.customizationBehavior(.disabled, for: .sidebar, .tabBar)


Tab("Optional Tab", ...) {
    // ...
}
.customizationID("Tab.example.optional")
.defaultVisibility(.hidden, for: .tabBar)
```

Tabs that aren't customizable don't need the customizationID.
Tabs can be hidden.

## Platform considerations

- macOS Sequoia - if your TabView or TabBarController supports a sidebar, it adopts the standard Mac sidebar appearance
- Tabs in sidebar can be reordered through drag-and-drop, just like on iPad
