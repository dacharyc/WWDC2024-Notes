# Create a custom data store with SwiftData

Presenter: Luvena Huo, SwiftData Engineer

Link: https://developer.apple.com/wwdc24/10138

Use SwiftData with your own persistence backend
Use any document, file format, or persistence backend of your choice
Change only the configuration

```swift
import SwiftUI
import SwiftData

@main
struct TripsApp: App {
    var container: ModelContainer = {
        do {
            // Replace ModelConfiguration with custom data store configuration - i.e.
            let configuration = JSONStoreConfiguration(schema: Schema([Trip,self]), url: fileURL)
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

## Overview

Model context powers the view, and is backed by model container
Trips have a persistent identifier in the model context and saves to the model container (store) when needed

The role of the store is to:
- Perform fetch and save
- Persist model values
- Communicate through `DataStoreSnapshot` instances

Snapshots capture model values

## Meet DataStore protocol

Three key parts to a store:

- Configuration: `DataStoreConfiguration` protocol (`ModelConfiguration`)
- Communication: `DataStoreSnapshot` protocol (`DefaultSnapshot`)
- Implementation: `DataStore` protocol (`DefaultStore`)

### DataStore Protocol

- Save
- Fetch
- Caching

Additional protocols:
- History
- Batch deletes

## Example store

JSON file as a store
- Archival sstore
- Array of snapshots - using JSON encoders provided by Foundation

### Declare configuration and store type 

```swift
class JSONStoreConfiguration: DataStoreConfiguration {
    typealias Store = JSONStore

}

class JSONStore: DataStore {
    typealias Configuration = JSONStoreConfiguration
    // Here, we're using the DefaultSnapshot because we don't have to customize how the data store communicates with the model context
    typealias Snapshot = DefaultSnapshot

    func fetch<T>(_ request: DataStoreFetchRequest<T>) throws -> DataStoreFetchResult<T, DefaultSnapshot> where T : PersistentModel {
        if request.descriptor.predicate != nil {
            throw DataStoreError.preferInMemoryFilter
        } else if request.descriptor.sortBy.count > 0 {
            throw DataStoreError.preferInMemorySort
        }

        let decoder = JSONDecoder()
        let data = try Data(contentsOf: configuration.fileURL)
        let trips = try decoder.decode([DefaultSnapshot].self, from: data)
        return DataStoreFetchResult(descriptor: request.descriptor, fetchedSnapshots: trips)
    }

    func save(_ request: DataStoreSaveChangesRequest<DefaultSnapshot>) throws -> DataStoreSaveChangesResult<DefaultSnapshot> {
        var snapshotsByIdentifier = [PersistentIdentifier: DefaultSnapshot]()
        try self.read().forEach { snapshotsByIdentifier[$0.persistentIdentifier] = $0 }

        var remappedIdentifiers = [PersistentIdentifer: PersistentIdentifier]()
        for snapshot in request.inserted {
            let entityName = snapshot.persistentIdentifier.entityName
            let permanentIdentifier = try PersistentIdentifier.identifier(for: identifier, entityName: entityName, primaryKey: UUID())
            let snapshotCopy = snapshot.copy(persistentIdentifier: permanentIdentifier)
            remappedIdentifiers[snapshot.persistentIdentifier] = permanentIdentifier
            snapshotsByIdentifier[permanentIdentifier] = snapshotCopy
        }

        for snapshot in request.updated {
            snapshotsByIdentifier[snapshot.persistentIdentifier] = snapshot
        }

        for snapshot in request.deleted {
            snapshotsByIdentifier[snapshot.persistentIdentifier] = nil
        }

        let snapshots = snapshotsByIdentifier.values.map({ $0 })
        let encoder = JSONEncoder()
        let jsonData = try encoder.encode(snapshots)
        try jsonData.write(to: configuration.fileURL)
        return DataStoreSaveChangesResult(for: self.identifier, remappedIdentifiers: remappedIdentifiers)
    }
}
```

## Next steps

- Adopt custom stores
- Implement support for any storage

Check out:
- What's new in SwiftData
- Track model changes with SwiftData history
