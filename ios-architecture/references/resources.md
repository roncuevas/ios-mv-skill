# Organización de Recursos

## Table of Contents
- [SF Symbols (Enum + Extensions)](#sf-symbols-enum--extensions)
- [Strings (LocalizedStringKey)](#strings-localizedstringkey)
- [Colors con Theme](#colors-con-theme)
- [Media.xcassets Estructura](#mediaxcassets-estructura)

## SF Symbols (Enum + Extensions)

```swift
// Library/Sources/Resources/Image+SFSymbol.swift
import SwiftUI

package enum SFSymbol: String {
    case calendar
    case clock
    case person
    case person3 = "person.3"
    case plus
    case pencil
    case trash
    case micActive = "mic"
    case micInactive = "mic.slash"
}

public extension Image {
    init(_ symbol: SFSymbol) {
        self.init(systemName: symbol.rawValue)
    }

    static let plus = Image(.plus)
    static let pencil = Image(.pencil)
    static let trash = Image(.trash)
}
```

```swift
// Library/Sources/Resources/Button+SFSymbol.swift
public extension Button where Label == SwiftUI.Label<Text, Image> {
    init(_ title: LocalizedStringKey, symbol: SFSymbol, action: @escaping () -> Void) {
        self.init(action: action) {
            Label(title, systemImage: symbol.rawValue)
        }
    }
}
```

```swift
// Library/Sources/Resources/Label+SFSymbol.swift
public extension Label where Title == Text, Icon == Image {
    init(_ title: LocalizedStringKey, symbol: SFSymbol) {
        self.init(title, systemImage: symbol.rawValue)
    }
}
```

## Strings (LocalizedStringKey)

```swift
// Library/Sources/Resources/LocalizedStringKey+Constants.swift
import SwiftUI

// MARK: - A
public extension LocalizedStringKey {
    static let add: Self = "Add"
    static let attendees: Self = "Attendees"
    static func attendeeCount(_ count: Int) -> Self { "\(count) attendees" }
}

// MARK: - C
public extension LocalizedStringKey {
    static let cancel: Self = "Cancel"
    static let create: Self = "Create"
}

// MARK: - D
public extension LocalizedStringKey {
    static let delete: Self = "Delete"
    static let done: Self = "Done"
}

// MARK: - E
public extension LocalizedStringKey {
    static let edit: Self = "Edit"
}

// MARK: - S
public extension LocalizedStringKey {
    static let save: Self = "Save"
}
```

### String+Constants.swift (para strings no localizables)

```swift
// Library/Sources/Resources/String+Constants.swift
public extension String {
    static let appName = "MyApp"
    static let defaultDateFormat = "yyyy-MM-dd"
}
```

## Colors con Theme

```swift
// Library/Sources/Resources/Color+Theme.swift
import SwiftUI
import Models

public extension Color {
    init(_ theme: Theme) {
        self.init(theme.rawValue, bundle: .module)
    }
}

public extension Theme {
    var mainColor: Color { Color(self) }
}
```

```swift
// Models/Sources/Enumerations/Theme.swift
public enum Theme: String, CaseIterable, Identifiable, Hashable, Codable {
    case bubblegum, buttercup, indigo, lavender, magenta
    case navy, orange, oxblood, periwinkle, poppy
    case purple, seafoam, sky, tan, teal, yellow

    public var id: String { rawValue }
    public var name: String { rawValue.capitalized }

    public var accentColor: Color {
        switch self {
        case .bubblegum, .buttercup, .lavender, .orange,
             .periwinkle, .poppy, .seafoam, .sky, .tan, .teal, .yellow:
            return .black
        case .indigo, .magenta, .navy, .oxblood, .purple:
            return .white
        }
    }
}
```

## Media.xcassets Estructura

```
Library/Sources/Resources/Media.xcassets/
├── Contents.json
└── Themes/
    ├── bubblegum.colorset/
    │   └── Contents.json
    ├── buttercup.colorset/
    │   └── Contents.json
    └── ... (un colorset por cada Theme case)
```
