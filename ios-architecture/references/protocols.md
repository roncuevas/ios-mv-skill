# Protocolos Compartidos

## Routable (Navegación Type-Safe)

```swift
// Models/Sources/Protocols/Routable.swift
import SwiftUI

public protocol Routable: View, Identifiable, Hashable {
    var id: Self { get }
    associatedtype Body: View
    @MainActor @ViewBuilder static func destination(for route: Self) -> Body
}

public extension Routable {
    var id: Self { self }
    func hash(into hasher: inout Hasher) {
        hasher.combine(id)
    }
}
```

### Uso en Features

```swift
enum AppRoute: Routable {
    case home
    case detail(Item)
    case settings

    @ViewBuilder
    var body: some View {
        switch self {
        case .home: HomeScreen()
        case .detail(let item): DetailScreen(item: item)
        case .settings: SettingsScreen()
        }
    }
}
```

## ToolbarView (Toolbars Estandarizadas)

```swift
// Models/Sources/Protocols/ToolbarView.swift
import SwiftUI

public protocol ToolbarView {
    associatedtype Content: ToolbarContent
    @MainActor @ToolbarContentBuilder func toolbarContent() -> Content
}
```

## ViewLifecycle (Ciclo de Vida)

```swift
// Models/Sources/Protocols/ViewLifecycle.swift
import SwiftUI

@MainActor
public protocol ViewLifecycle {
    func onTask() async
    func onAppear()
    func onDisappear()
}

public extension ViewLifecycle {
    func onTask() async {}
    func onAppear() {}
    func onDisappear() {}
}
```
