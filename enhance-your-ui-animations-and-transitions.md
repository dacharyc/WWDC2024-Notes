# Enhance your UI animations and transitions

Presenters: Russell Ladd, UI Frameworks

Link: https://developer.apple.com/wwdc24/10145

New features and APIs

## Transitions

### Zoom transition

Cell morphs with incoming view
- Add `.navigationTransitionStyle` and specify `.zoom`
- Connect the modifier to a source view

```swift
NavigationLink {
    BraceletEditor(bracelet)
        .navigationTransitionStyle(
            .zoom(
                sourceID: bracelet.id,
                in: braceletList
            )
        )
} label: {
    BraceletPreview(bracelet)
}
.matchedTransitionSource(
    id: bracelet.id,
    in: braceletList
)
```

I skipped all the UIKit stuff because BOO UIKIT!

## SwiftUI Animation

```swift
withAnimation(.spring(duration: 0.5)) {
    beads.append(Bead())
}
```

## Animating representables

```swift
struct BeadBoxWrapper: UIViewRepresentable {
    @Binding var isOpen: Bool

    func updateUIView(_ box: BeadBox, context: Context) {
        context.animate {
            box.lid.center.y = isOpen ? -100 : 100
		    }
    }
}

struct BraceletEditor: View {
    @State private var isBeadBoxOpen = false
    var body: some View {
        BeadBoxWrapper($isBeadBoxOpen.animated())
            .onTapGesture {
                isBeadBoxOpen.toggle()
            }
    }
}
```

## Gesture-driven animations

```swift
switch gesture.state {
case .changed:
    UIView.animate(.interactiveSpring) {
        bead.center = gesture.translation
    }

case .ended:
    UIView.animate(.spring) {
        bead.center = endOfBracelet
    }
}
```
