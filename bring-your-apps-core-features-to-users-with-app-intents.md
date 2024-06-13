# Bring your app's core features to users with App Intents

Presenter: Christopher Nebel, App Intents Engineering

Link: https://developer.apple.com/wwdc24/10210

## Friction versus flow

- On a device, the next thing you need is always nearby and easy to get to
- Switching apps introduces friction
- Making your app's features available outside of your app reduces friction

Who decides which features to elevate?
- With Spotlight, you the developer choose which features to elevate
- With widgets and controls, people choose. The developer provides options but users personalize their experience of your app.
- Adopt App Intents to add these features.

## Understanding the framework

Adopting App Intents involves these three core concepts:
- Intents
- Entities
- App shortcuts

## Building the code

- Shortcuts action
- Parameterized action
- Home Screen widget
- Control Center control
- Spotlight and Siri

### Shortcuts action

Add a shortcuts action to open a pinned trail in the app. Must conform to `AppIntent`:
- provide a `title`
- provide an action to perform

```swift
struct OpenPinnedTrail: AppIntent {
    static let title: LocalizedStringResource = "Open Pinned Trail"

    func perform() async throws -> some IntentResult {
        NavigationModel.shared.navigate(to: .pinned)
        return .result()
    }

    static let openAppWhenRun: Bool = true
}
```

### Parameterized action

Intents that open a parameter in the app have a special protocol, `OpenIntent`
- A parameter should be named for a user concept, not an implementation detail - i.e. "trail" versus table row or other implementation detail
- An entity should not be a UUID or other thing - it should be the actual entity

```swift
struct OpenTrail: AppIntent, OpenIntent {
    static let title: LocalizedStringResource = "Open Trail"

    @Parameter(title: "Trail")
    var target: TrailEntity

    func perform() async throws -> some IntentResult {
        NavigationModel.shared.navigate(to: target)
        return .result()
    }

    static var parameterSummary: some ParameterSummary {
        Summary("Open \(\.$trail)")
    }
}

struct TrailEntity: AppEntity {
    @Property(title: "Trail name")
    var name: String

    static let typeDisplayRepresentation: TypeDisplayRepresentation = "Trail"

    var displayRepresentation: DisplayRepresentation {
        DisplayRepresentation(title: name, image: Image(named: imageName))
    }

    var id: Trail.ID

    static var defaultQuery = TrailEntityQuery()
}
```

A query turns questions asking for entities into the actual entities
- What entities are there?
- What entity has this ID?

```swift
struct TrailEntityQuery: EntityQuery {
    func entities(for identifiers: [TrailEntity.ID]) async throws -> [TrailEntity] {
        TrailDataManager.shared.trails(with: identifiers)
            .map { TrailEntity(trail: $0) }
    }
}

/* There are other protocols. This is the automatically derived entity.
 * Your entire model must fit into memory when using this protocol. If it
 * can't, use a different protocol instead. */
extension TrailEntityQuery: EnumerableEntityQuery {
    func allEntities() async throws -> [TrailEntity] {
        TrailDataManager.shared.trails.map { TrailEntity(trail: $0) }
    }
}
```

### Home Screen widget

Widgets are designed for glancable information that changes often
In this case, we want to make it configurable so you can set a trail

```swift
struct TrailConditionsWidget: Widget {
    static let kind = "TrailConditionsWidget"

    var body: some WidgetConfiguration {
        AppIntentConfiguration(
            kind: Self.kind,
            intent: TrailConditionsConfiguration.self,
            provider: Provider()
        ) {
            TrailConditionsEntryView(entry: $0)
        }
    }
}
```

Check out:
Explore enhancements to App Intents (WWDC23)

```swift
struct TrailConditionsConfiguration: WidgetConfigurationIntent {
    static var title: LocalizedStringResource = "Trail Conditions"

    @Parameter(title: "Trail")
    var trail: TrailEntity?
}
```

### Control Center control

- New API in iOS 18

```swift
struct TrailConditionsControl: ControlWidget {
    var body: some ControlWidgetConfiguration {
        AppIntentControlConfiguration(
            kind: Self.kind,
            intent: OpenTrail.self
        ) { configuration in
            ControlWidgetButton(action: configuration) {
                Image(systemName: configuration.target.glyph)
                Text(configuration.target.name)
            }
        }
    }
}

extension OpenTrail: ControlConfigurationIntent { }
```

Check out:
Extend your app's controls across the system

### Spotlight and Siri

Create an App shortcut to expose an action

```swift
struct TrailShortcuts: AppShortcutsProvider {
    static var appShortcuts: [AppShortcut] {
        AppShortcut(
            intent: OpenPinnedTrail(),
            phrases: [
                "Open my pinned trail in \(.applicationName)",
                "Show my pinned trail in \(.applicationName)"
            ],
            shortTitle: "Open Pinned Trail",
            systemImageName: "pin"
        )
    }
}
```

For Siri, return the information in a spoken way instead of opening the app:

```swift
struct GetPinnedTrail: AppIntent {
    static let title: LocalizedStringResource = "Get Pinned Trail"

    func perform() async throws -> some IntentResult & ProvidesDialog & ShowsSnippetView {
        let pinned = TrailDataManager.shared.pinned
        return .result(
            dialog: """
                The latest reported condition of \(pinned.name) is \(pinned.currentConditions)
                """ 
            view: trailConditionsSnippetView()
        )
    }
}
```

Check out:
- Designing App Intents
- Bring your app to Siri
- What's new in App Intents
