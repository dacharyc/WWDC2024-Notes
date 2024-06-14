# Get started with writing tools

Presenter: Dongyuan Liu, Text Input & Internationalization

Link: https://developer.apple.com/wwdc24/10168

New suite of features available in text views across all kinds of apps
- Helps users polish text they're working on

## Introducing writing tools

- Help users proofread, rewrite, or transform text
- Suggestions appear inline
- When you hover over selected text, it shows up with an option to open the Writing Tools panel
- Can review and apply or reject suggestions one-by-one
- Can also work with non-editable text - user can copy results

## Native text views

- If you're using a supported text view, it just shows up and works

New delegate methods for Writing Tools:

```swift
func textViewWritingToolsWillBegin(_ textView: UITextView) {
    // Take necessary steps to prepare. For example, disable iCloud sync.
}

func textViewWritingToolsDidEnd(_ textView: UITextView) {
    // Take necessary steps to recover. For example, reenable iCloud sync.
}

if !textView.isWritingToolsActive {
    // Do work that needs to be avoided when Writing Tools is interacting with text view
    // For example, in the textViewDidChange callback, app may want to avoid certain things
       when Writing Tools is active
}
```

### Text Storage

- Chunking with animations
- "Original" button to toggle between original and rewritten text
- Proofreading where user can choose whether or not to apply
- Do not persist data while writing tools are active

## Controlling behavior

By default, the system offers writing tools by default. You can opt out of the full experience:

```swift
textView.writingToolsBehavior = .limited

textView.writingToolsBehavior = .none
```

You can specify accepted text formats:

```swift
textView.writingToolsAllowedInputOptions = [.plainText]

textView.writingToolsAllowedInputOptions = [.plainText, .richText, .table]
```

Similar APIs for WKWebView:

```swift
// For `WKWebView`, the `default` behavior is equivalent to `.limited`

extension WKWebViewConfiguration {
    @available(iOS 18.0, *)
    open var writingToolsBehavior: UIWritingToolsBehavior { get set }
}

extension WKWebViewConfiguration {
    @available(macOS 15.0, *)
    open var writingToolsBehavior: NSWritingToolsBehavior { get set }
}

extension WKWebView {
    /// @discussion If the Writing Tools behavior on the configuration is `.limited`, this will always be `false`.
    @available(iOS 18.0, macOS 15.0, *)
    open var isWritingToolsActive: Bool { get }
}
```

## Protecting ranges

- Specify ranges to ignore, such as quotes or code blocks

```swift
// Returned `NSRange`s are relative to the substring of the textView’s textStorage from `enclosingRange`
func textView(_ textView: UITextView, writingToolsIgnoredRangesIn
        enclosingRange: NSRange) -> [NSRange] {
    let text = textView.textStorage.attributedSubstring(from: enclosingRange)
    return rangesInappropriateForWritingTools(in: text)
}
```

## Custom text views

On iOS and iPadOS, as long as your app adopts `UITextInteraction`, you get writing tools in the callout bar or context menu for free
If you can't use this, you can alternately adopt `UITextSelectionDisplayInteraction` or `UIEditMenuInteraction`
For text views that don't use text interactions, new property `isEditable` in `UITextInput` protocol to indicate if your view supports editing

```swift
protocol UITextInput {
    @available(iOS 18.0, macOS 15.0, *)
    optional var isEditable: Bool { get }
}
```

On Mac, we show writing tools automatically in context menu and Edit menu for custom text views
- Make sure your text view adopts `NSServicesMenuRequestor`

```swift
class CustomTextView: NSView, NSServicesMenuRequestor {
    required init?(coder: NSCoder) {
        super.init(coder: coder)
        
        self.menu = NSMenu()
        self.menu?.addItem(NSMenuItem(title: "Custom Text View", action: nil,
            keyEquivalent: ""))
        self.menu?.addItem(NSMenuItem(title: "Copy", action: #selector(copy(_:)), 
            keyEquivalent: ""))
    }
  
    override func draw(_ dirtyRect: NSRect) {
        super.draw(dirtyRect)
        
        // Custom text drawing code...
    }
}
```

You can override `validRequestor(forSendType: returnType:)` to add writing tools for free:

```swift
class CustomTextView: NSView, NSServicesMenuRequestor {
    override func validRequestor(forSendType sendType: NSPasteboard.PasteboardType?, 
                                 returnType: NSPasteboard.PasteboardType?) -> Any? {
        if sendType == .string || sendType == .rtf {
            return self
        }
        return super.validRequestor(forSendType: sendType, returnType: returnType)
    }
    
    nonisolated func writeSelection(to pboard: NSPasteboard,
                                    types: [NSPasteboard.PasteboardType]) -> Bool {
        // Write plain text and/or rich text to pasteboard
        return true
    }

    // Implement readSelection(from pboard: NSPasteboard)
       as well for editable view
}
```
