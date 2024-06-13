# What's new in SwiftData

Presenter: Rishi Verma, SwiftData Engineer

Link: https://developer.apple.com/wwdc24/10137

## Adopt SwiftData

- import SwiftData
- Add the `@Model` macro to data model
- Inject a `modelContainer` into the view hierarchy
- `@Query` the modelContainer to load data in views

## Customize the schema

Schema macros:
- `@Attribute`
- `@Relationship`
- `@Transient`
- `#Unique` - NEW - define key paths that represent the same data. Collisions become updates.

```swift
import SwiftData

@Model
class Trip {
    #Unique<Trip>([\.name, \.startDate, \.endDate])

    var name: String
    var destination: String
    var startDate: Date
    var endDate: Date

    var bucketList: [BucketListItem] = [BucketListItem]()
    var livingAccommodation: LivingAccommodation?
}
```

You can also specify behavior for updates on unique properties. In this case, we preserve a value even when an object is deleted so it won't break "history"

```swift
import SwiftData

@Model
class Trip {
    #Unique<Trip>([\.name, \.startDate, \.endDate])

    @Attribute(.preserveValueOnDeletion)
    var name: String
    var destination: String

    @Attribute(.preserveValueOnDeletion)
    var startDate: Date

    @Attribute(.preserveValueOnDeletion)
    var endDate: Date

    var bucketList: [BucketListItem] = [BucketListItem]()
    var livingAccommodation: LivingAccommodation?
}
```

## Reveal history with SwiftData

- Track inserted, updated, and deleted models
- Models marked preserved can be saved with opt-in (@Attribute tag)
- Works with custom data stores

Check out:
Track model changes with SwiftData history

## Tailor the model container

- You can customize some of the properties of the model container

```swift
import SwiftUI
import SwiftData

@main
struct TripsApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(for: Trip.self,
                        inMemory: true,
                        isAutosaveEnabled: true,
                        isUndoEnabled: true)
    }
}

```

You can optionally create your own model container:

```swift
import SwiftUI
import SwiftData

@main
struct TripsApp: App {
    var container: ModelContainer = {
        do {
            let configuration = ModelConfiguration(schema: Schema([Trip,self]), url: fileURL)
            return try ModelContainer(for: Trip.self, configurations: configuration)
        } catch {...}
    }()

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(container)
    }
}

```

You can create your own data store in iOS 18

- Use familiar SwiftData API
- Adopt features incrementally

Check out:
Create a custom data store with SwiftData

### Containers for Previews

You can also create custom containers for Previews with SwiftData. This requires:

- Create a struct that conforms to the PreviewModifier protocol
- It must implement:
  - func makeSharedContext() that returns a ModelContainer
  - func body() that returns a View
- Extend PreviewTrait with the sample data

```swift
struct SampleData: PreviewModifier {
    static func makeSharedContext() throws -> ModelContainer {
        let configuration = ModelConfiguration(isStoredInMemoryOnly: true)
        let container = try ModelContainer(for: Trip.self, configurations: config)
        Trip.makeSampleTrips(in: container)
        return container
    }

    func body(content: Content, context: ModelContainer) -> some View {
        content.modelContainer(context)
    }
}

extension PreviewTrait where T == Preview.ViewTraits {
    @MainActor static var sampleData: Self = .modifier(SampleData())
}
```

Now, to use sample data in a Preview, add the modifer as a trait:

```swift
import SwiftUI
import SwiftData

struct ContentView: View {
    @Query
    var trips: [Trip]

    var body: some View {
        ...
    }
}

#Preview(traits: .sampleData) {
    ContentView()
}
```

You can use the `@Previewable` macro to pass sample data to a view that is expecting to have it passed in:

```swift
import SwiftUI
import SwiftData

#Preview(traits: .sampleData) {
    @Previewable @Query var trips: [Trip]
    BucketListItemView(trip: trips.first)
}
```

## Optimize queries

- Search bar can be used to build a predicate

```swift
let predicate = #Predicate<Trip> {
    searchText.isEmpty ? true : $0.name.localizedStandardContains(searchText)
}
```

- Create a compound predicate to find a trip based on search text

```swift
let predicate = #Predicate<Trip> {
    searchText.isEmpty ? true : 
    $0.name.localizedStandardContains(searchText) ||
    $0.destination.localizedStandardContains(searchText)
}
```

### New `#Expression` macro

- Express complex queries that do not evaluate to true or false, but produce complex types
- Compose predicates

```swift
let unplannedItemsExpression = #Expression<[BucketListItem], Int> { items in
    items.filter {
        !$0.isInPlan
    }.count
}

let today = Date.now
let tripsWithUnplannedItems = #Predicate<Trip> { trip in
    // The current date falls within the trip
    (trip.startDate ..< trip.endDate).contains(today)

    // The trip has at least one BucketListItem where `isInPlan` is false
    unplannedItemsExpression.evaluate(trip.bucketList) > 0
}
```

### New `Index` schema macro

- Create a simple or compound index on the model
- Adds metadata to the model
- Makes queries for specified keypaths faster and more efficient
- Specify the properties used in queries - typically sort and filter

```swift
import SwiftData

@Model
class Trip {
    #Unique<Trip>([\.name, \.startDate, \.endDate])
    #Index<Trip>([\.name], [\.startDate], [\.endDate], [\.name, \.startDate, \.endDate])

    var name: String
    var destination: String
    var startDate: Date
    var endDate: Date

    var bucketList: [BucketListItem] = [BucketListItem]()
    var livingAccommodation: LivingAccommodation?
}
```
