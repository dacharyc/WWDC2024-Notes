# Track model changes with SwiftData history

Presenter: David Stites, SwiftData engineer

Link: https://developer.apple.com/wwdc24/10075

SwiftData history is a new technology that lets app track modifications to its data
Use for:
- Syncing with server
- Responding to changes from app extension

## Fundamentals

- When model context is changed, pending changes saved to the data store
- Over time, models may change or be deleted
- At any time, the app can query the data in the store
- A queries results represent what's *currently* in the data store - without history or manual diffing, no way to know which models may have been added, deleted, or updated since a previous one

- Helps explore changes to a store
- Transaction format
- Transactions contain metadata about changes
  - Transactions are ordered by when they occurred
- SwiftData history has a concept of a "token" which acts as a bookmark for transactions in the history
- A token can help your app keep track of the last transaction it processed in the stream of history
- Tokens are:
  - Constrained to a specific data store
  - Become expired when the relevant part of history they pertain to is deleted
    - When this happens, SwiftData operations involving an expired token throw a `historyTokenExpired` error
    - Solution is to discard the token and retrieve a new one
- You can preserve specific attributes on a model for essential data like identifiers to use when processing history information
  - SwiftData calls these "tombstone values"
  - Attributes that have been marked `.preserveValueOnDeletion` are preserved in the tombstone
  - Tombstones are also parameterized by a PersistentModel so their keypaths can be used to retrieve the tombstone values or iterated as desired

```swift
// Add .preserveValueOnDeletion to capture unique columns
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

## Transactions and changes

Fetch transactions from history

```swift
private func findTransactions(after token: DefaultHistoryToken?, author: String) -> [DefaultHistoryTransaction] {
    var historyDescriptor = HistoryDescriptor<DefaultHistoryTransaction>() 
    if let token {
        historyDescriptor.predicate = #Predicate { transaction in
            (transaction.token > token) && (transaction.author == author)
        }
    }
    
    var transactions: [DefaultHistoryTransaction] = []
    let taskContext = ModelContext(modelContainer)
    do {
        transactions = try taskContext.fetchHistory(historyDescriptor)
    } catch let error {
        print(error)
    }

    return transactions
}
```

Process history changes

```swift
private func findTrips(in transactions: [DefaultHistoryTransaction]) -> (Set<Trip>, DefaultHistoryToken?) {
        let taskContext = ModelContext(modelContainer)
        var resultTrips: Set<Trip> = []
        for transaction in transactions {
            for change in transaction.changes {
                let modelID = change.changedPersistentIdentifier
                let fetchDescriptor = FetchDescriptor<Trip>(predicate: #Predicate { trip in
                    trip.livingAccommodation?.persistentModelID == modelID
                })
                let fetchResults = try? taskContext.fetch(fetchDescriptor)
                guard let matchedTrip = fetchResults?.first else {
                    continue
                }
                switch change {
                case .insert(_ as DefaultHistoryInsert<LivingAccommodation>):
                    resultTrips.insert(matchedTrip)
                case .update(_ as DefaultHistoryUpdate<LivingAccommodation>):
                    resultTrips.update(with: matchedTrip)
                case .delete(_ as DefaultHistoryDelete<LivingAccommodation>):
                    resultTrips.remove(matchedTrip)
                default: break
                }
            }
        }
        return (resultTrips, transactions.last?.token)
    }
```

Save and use a history token

```swift
private func findUnreadTrips() -> Set<Trip> {
    let tokenData = UserDefaults.standard.data(forKey: UserDefaultsKey.historyToken)
    
    var historyToken: DefaultHistoryToken? = nil
    if let tokenData {
        historyToken = try? JSONDecoder().decode(DefaultHistoryToken.self, from: tokenData)
    }
    let transactions = findTransactions(after: historyToken, author: TransactionAuthor.widget)
    let (unreadTrips, newToken) = findTrips(in: transactions)
    
    if let newToken {
        let newTokenData = try? JSONEncoder().encode(newToken)
        UserDefaults.standard.set(newTokenData, forKey: UserDefaultsKey.historyToken)
    }
    return unreadTrips
}
```

Update the user interface

```swift
struct ContentView: View {
    @Environment(\.scenePhase) private var scenePhase
    @State private var showAddTrip = false
    @State private var selection: Trip?
    @State private var searchText: String = ""
    @State private var tripCount = 0
    @State private var unreadTripIdentifiers: [PersistentIdentifier] = []

    var body: some View {
        NavigationSplitView {
            TripListView(selection: $selection, tripCount: $tripCount,
                         unreadTripIdentifiers: $unreadTripIdentifiers,
                         searchText: searchText)
            .toolbar {
                ToolbarItem(placement: .topBarLeading) {
                    EditButton()
                        .disabled(tripCount == 0)
                }
                ToolbarItemGroup(placement: .topBarTrailing) {
                    Spacer()
                    Button {
                        showAddTrip = true
                    } label: {
                        Label("Add trip", systemImage: "plus")
                    }
                }
            }
        } detail: {
            if let selection = selection {
                NavigationStack {
                    TripDetailView(trip: selection)
                }
            }
        }
        .task {
            unreadTripIdentifiers = await DataModel.shared.unreadTripIdentifiersInUserDefaults
        }
        .searchable(text: $searchText, placement: .sidebar)
        .sheet(isPresented: $showAddTrip) {
            NavigationStack {
                AddTripView()
            }
            .presentationDetents([.medium, .large])
        }
        .onChange(of: selection) { _, newValue in
            if let newSelection = newValue {
                if let index = unreadTripIdentifiers.firstIndex(where: {
                    $0 == newSelection.persistentModelID
                }) {
                    unreadTripIdentifiers.remove(at: index)
                }
            }
        }
        .onChange(of: scenePhase) { _, newValue in
            Task {
                if newValue == .active {
                    unreadTripIdentifiers += await DataModel.shared.findUnreadTripIdentifiers()
                } else {
                    // Persist the unread trip names for the next launch session.
                    await DataModel.shared.setUnreadTripIdentifiersInUserDefaults(unreadTripIdentifiers)
                }
            }
        }
        #if os(macOS)
        .onReceive(NotificationCenter.default.publisher(for: NSApplication.didBecomeActiveNotification)) { _ in
            Task {
                unreadTripIdentifiers += await DataModel.shared.findUnreadTripIdentifiers()
            }
        }
        .onReceive(NotificationCenter.default.publisher(for: NSApplication.willTerminateNotification)) { _ in
            Task {
                await DataModel.shared.setUnreadTripIdentifiersInUserDefaults(unreadTripIdentifiers)
            }
        }
        #endif
    }
}
```

## Custom stores

- Custom stores can support history. You must implement types to represent the fundamental elements of the SwiftData History API
  - Transactions (`HistoryTransaction`)
  - Each type of change (`HistoryChange`)
  - A token to act as a bookmark between transactions (`HistoryToken`)
  - Custom store must conform to `HistoryProviding` protocol to vend history
