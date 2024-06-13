# Support semantic search with Core Spotlight

Presenters: Jennifer Moore, Spotlight Engineer

Link: https://developer.apple.com/wwdc24/10131

- Framework that allows your app to donate searchable content to Spotlight

## Provide content to the search index

- Donate searchable content to Spotlight that represents what people will want to search
- Index items in a way that can be retrieved from a query
- Semantic search works best on text and media
- Use system-defined attributes when possible

### Creating CCSearchableItem

```swift
// Creating searchable items for donation
let item = CSSearchableItem(uniqueIdentifier: uniqueIdentifier, domainIdentifier: domainIdentifier, attributeSet: attributeSet)
```

### Creating CCSearchableAttributeSet

```swift
// Creating searchable content for donation
let attributeSet = CSSearchableItemAttributeSet(contentType: UTType.text)
attributeSet.contentType = UTType.text.identifier
```

### Searchable items with type

```swift
// Searchable items with text
attributeSet.title
attributeSet.textContent

// Searchable items with media
attributeSet.contentType
attributeSet.contentURL

// Searchable items with links
attributeSet.contentURL
attributeSet.relatedUniqueIdentifier
```

## Indexing items

## Batch indexing with client state

```swift
// Batch indexing with client state
let index = CSSearchableIndex(name: "SpotlightSearchSample")        
index.fetchLastClientState { state, error in         
    if state == nil {
        index.beginBatch()
        index.indexSearchableItems(items)
        index.endIndexBatch(expectedClientState: state, newClientState: newState) { error in
        }
    }
}
```

## Avoid overwriting existing attributes

```swift
// Make it an update to avoid overwriting existing attributes
item.isUpdate = true
```

## Configure a query

```swift
// Configure a query
let queryContext = CSUserQueryContext()
queryContext.fetchAttributes = ["title", "contentDescription"]
```

### Ranked results

```swift
// Ranked results
queryContext.enableRankedResults = true
queryContext.maxRankedResultCount = 2
```

### Suggestions

```swift
// Suggestions
queryContext.maxSuggestionCount = 4
```

### Filter queries

```swift
// Filter queries
queryContext.filterQueries = ["contentTypeTree=\"public.image\""]
```

## Query for searchable items and suggestions

```swift
// Query for searchable items and suggestions
let query = CSUserQuery(userQueryString: "windsurfing carmel", userQueryContext: queryContext)
for try await element in query.responses {
    switch(element) {
        case .item(let item):
            self.items.append(item)
            break
        case .suggestion(let suggestion):
            self.suggestions.append(suggestion)
            break
    }
}
```

### Suggestions

```swift
// Suggestions
suggestion.localizedAttributedSuggestion
```

## Preparing for queries

Models must be downloaded, so call this before search appears

```swift
// Preparing for queries
CSUserQuery.prepare
CSUserQuery.prepareWithProtectionClasses
```

## Freshness signals for ranking

### Set the lastUsedDate

```swift
// Set the lastUsedDate when the user interacts with the item
item.attributeSet.lastUsedDate = Date.now
item.isUpdate = true
```

### Interactions with items and suggestions from a query

```swift
// Interactions with items and suggestions from a query
query.userEngaged(item, visibleItems: visibleItems, interaction: CSUserQuery.UserInteractionKind.select)
query.userEngaged(suggestion, visibleSuggestions: visibleSuggestions, interaction: CSUserQuery.UserInteractionKind.select)
```
