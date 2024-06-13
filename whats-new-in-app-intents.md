# What's new in App Intents

Presenter: Kenny York, App Intents Engineering

Link: https://developer.apple.com/wwdc24/10134

- Spotlight integration
- Entities and files
- Universal links
- Developer improvements

## Spotlight integration

- `IndexedEntity`
  - Easy way to index app entities to see a searchable thing
  - New protocol

```swift
// Add conformance to the new protocol
extension TrailEntity: IndexedEntity { }

// In App's init method, index the trail entities in the data manager
try await CSSearchableIndex
    .default()
    .indexAppEntities(
        trailDataManager
            .trails
            .map(TrailEntity.init(trail:))
    )
```

Can optionally provide attributes with additional details:

```swift
extension TrailEntity: IndexedEntity {
    var attributeSet: CSSearchableItemAttributeSet {
        let attributes = CSSearchableItemAttributeSet()

        attributes.city = self.city
        attributes.stateOrProvince = self.state
        attributes.keywords = activities.map(\.rawValue)

        return attributes
    }
}
```

Can add semantic search support with new `associateAppEntity` API

```swift
public extension CSearchableItem {
    func associateAppEntity(
        _ appEntity: some IndexedEntity,
        priority: Int
    )
}
```

## Entities and files

### Make entities meaningful

- Transferable AppEntity
  - Export your AppEntity to different types and use in different places

```swift
extension ActivityStatisticsSummary: Transferable {
    static var transferRepresentation: some TransferRepresentation {
        DataRepresentation(exportedContentType: .rtf) { summary in
            try summary.asRTFData
        }

        FileRepresentation(exportedContentType: .png) { summary in
            SentTransferredFile(try summary.asImageFile)
        }
    }
}
```

### IntentFile

- Check what content types are available, and use whatever type it needs

```swift
struct AppendToNote: AppIntent {
    @Parameter var not: NoteEntity

    @Parameter(title: "content to append", supportedContentTypes: [.jpeg, .rtf])
    var attachment: IntentFile

    // ...
}
```

```swift
struct AppendToNote: AppIntent {
    // ...
    public func perform() async throws -> some IntentResult {
        if attachment.availableContentTypes.contains(.png) {
            let png = try await attachment
                .withFile(contentType: .png) { url, openedInPlace in
                    guard let image = UIImage(contentsOfFile: url.absolutePath) else {
                        throw Error.unableToLoadImage
                    }
                    returnImage
                }
        }
        return .result()
    }
}
```

### FileEntity

- Great for document-based apps or apps that manage files
- For cases where the file is the canonical version of the entity

```swift
struct PhotoEntity: FileEntity {
    static var typeDisplayRepresentation: TypeDisplayRepresentation = "My Photo Entity"

    static var supportedContentTypes: [UTType] = [.png]

    var id: FileEntityIdentifier

    @Property(title: "Width")
    var width: Double

    @Property(title: "Height")
    var height: Double
}
```

## Universal links

- Help people access your content

Example: `https://trailsapp.example/trail/1/details`

New APIs:

- `URLRepresentableEntity`
- `URLRepresentableEnum`
- `URLRepresentableIntent`

Make the entity conform to URLRepresentableEntity:

```swift
extension TrailEntity: URLRepresentableEntity {
    static var urlRepresentation: URLRepresentation {
        "https://trailsapp.example/trail/\(.id)/details"
    }
}
```

Then you can deep link to the content:

```swift
struct OpenTrailIntent: OpenIntent, URLRepresentableIntent {
    static var title: LocalizedStringResource = "Open Trail"

    static var parameterSummary: some ParameterSummary {
        Summary("Open \(\.$target)")
    }

    @Parameter(title: "Trail")
    var target: TrailEntity
}
```

Or return an `OpenURLIntent` from your perform:

```swift
func perform() async throws -> some OpensIntent {
    let newTrail = TrailEntity(name: name)
    .result(
        // You can open a URL directly
        /* opensIntent: OpenURLIntent(
         *    "https://developer.apple.com"
         * ) */
        // Or use urlRepresentable to
        opensIntent: OpenURLIntent(urlRepresentable: newTrail)
    )
}
```

## Developer improvements

### New UnionValue

- Could be more than one type
- Each case in the enum must have exactly one associated value
- Must be unique

```swift
@UnionValue
enum DayPassType {
    case park(ParkEntity)
    case trail(TrailEntity)
}

struct BuyDayPass: AppIntent {
    // ...
    @Parameter var passType: DayPassType
    // ...

    func perform() async throws -> some IntentResult {
        switch passType {
        case let .park(park):
            // purchase for park
        case let .trail(trail):
            // purchase for trail
        }
    }
}
```

### Generated titles

- You no longer have to provide a title for your AppEntity properties or parameters

This:

```swift
struct SuggestTrails: AppIntent {
    @Parameter(title: "Activity")
    var activity: ActivityStyle

    @Parameter(title: "Search Radius")
    var searchRadius: Measurement<UnitLength>?

    @Parameter(title: "Location")
    var location: String?

    @Parameter(title: "Featured Collection")
    var trailCollection: TrailCollection?
}
```

Becomes this:

```swift
struct SuggestTrails: AppIntent {
    @Parameter var activity: ActivityStyle
    @Parameter var searchRadius: Measurement<UnitLength>?
    @Parameter var location: String?

    @Parameter(title: "Featured Collection")
    var trailCollection: TrailCollection?
}
```

### Framework improvements

AppIntent types no longer have to be in the same module - you can use AppEntities in frameworks and reference from app and extension targets

