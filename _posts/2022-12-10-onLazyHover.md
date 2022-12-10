---
title: "`.onLazyHover`"
layout: post
excerpt: A less eager `.onHover` for SwiftUI
---

[Switch](https://github.com/numist/Switch) supports changing the window selection by moving the cursor, but it can't use hover state to accomplish this because its views sometimes get added _underneath the cursor_, which can result in a quick ⌥⇥ raising a long-forgotten window instead of the next one in line.

In the legacy codebase I'd resolved this using an Event Tap that updated the selection state whenever the cursor moved. I'm trying to do better in the rewrite, and my procrastination paid off with the introduction of [`.onContinuousHover`](https://developer.apple.com/documentation/swiftui/view/oncontinuoushover(coordinatespace:perform:)) in macOS Ventura. Combining it with some `@State` variables allows us to create a view modifier that only invokes its action when the cursor actually travels into/out of the view:

```swift
import SwiftUI

struct LazyHover: ViewModifier {
  let action: (Bool) -> Void
  @State private var hoverLocation: CGPoint = .zero
  @State private var actionPerformed = false

  func body(content: Content) -> some View {
    content.onContinuousHover { phase in
      switch phase {
      case .active(let location):
        if hoverLocation != .zero && !actionPerformed {
          actionPerformed = true
          action(true)
        }
        hoverLocation = location
      case .ended:
        hoverLocation = .zero
        if actionPerformed {
          actionPerformed = false
          action(false)
        }
      }
    }
  }
}

extension View {
  func onLazyHover(_ perform: @escaping (Bool) -> Void) -> some View {
    modifier(LazyHover(action: perform))
  }
}
```