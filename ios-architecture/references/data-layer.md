# Data Layer

## Table of Contents
- [RuntimeEnvironment](#runtimeenvironment)
- [DataStorage (Clase Observable)](#datastorage-clase-observable)
- [DataStore Property Wrapper](#datastore-property-wrapper)

## RuntimeEnvironment

```swift
// Models/Sources/Enumerations/RuntimeEnvironment.swift
public enum RuntimeEnvironment: CustomStringConvertible {
    case previews
    case simulator(String)
    case physicalDevice

    public static var current: RuntimeEnvironment {
        if ProcessInfo.processInfo.environment["XCODE_RUNNING_FOR_PREVIEWS"] == "1" {
            return .previews
        }
        #if targetEnvironment(simulator)
        let model = ProcessInfo.processInfo.environment["SIMULATOR_MODEL_IDENTIFIER"] ?? "Unknown"
        return .simulator(model)
        #else
        return .physicalDevice
        #endif
    }

    public var isPhysicalDevice: Bool {
        if case .physicalDevice = self { return true }
        return false
    }

    public var isSimulator: Bool {
        if case .simulator = self { return true }
        return false
    }

    public var isPreview: Bool { self == .previews }

    public var description: String {
        switch self {
        case .previews: return "Xcode Previews"
        case .simulator(let model): return "Simulator (\(model))"
        case .physicalDevice: return "Physical Device"
        }
    }
}
```

## DataStorage (Clase Observable)

```swift
// Models/Sources/Models/DataStorage.swift
import SwiftUI

@MainActor
@Observable
public final class DataStorage<T: Identifiable & Equatable & Codable>: Equatable {
    public private(set) var items: [T]
    private let key: String

    public init(key: String, defaultValue: [T] = []) {
        self.key = key
        if let data = UserDefaults.standard.data(forKey: key),
           let decoded = try? JSONDecoder().decode([T].self, from: data) {
            self.items = decoded
        } else {
            self.items = defaultValue
        }
    }

    public func upsert(_ item: T) {
        if let index = items.firstIndex(where: { $0.id == item.id }) {
            items[index] = item
        } else {
            items.append(item)
        }
        save()
    }

    public func delete(_ item: T) {
        items.removeAll { $0.id == item.id }
        save()
    }

    public func replace(with newItems: [T]) {
        items = newItems
        save()
    }

    private func save() {
        if let encoded = try? JSONEncoder().encode(items) {
            UserDefaults.standard.set(encoded, forKey: key)
        }
    }

    public static func == (lhs: DataStorage, rhs: DataStorage) -> Bool {
        lhs.items == rhs.items
    }
}
```

## DataStore Property Wrapper

```swift
// Models/Sources/Protocols/DataStore.swift
import SwiftUI

@propertyWrapper
public struct DataStore<T: Identifiable & Equatable & Codable>: DynamicProperty {
    @State private var storage: DataStorage<T>

    public init(key: String, defaultValue: [T] = []) {
        _storage = State(initialValue: DataStorage(key: key, defaultValue: defaultValue))
    }

    public var wrappedValue: [T] {
        get { storage.items }
        nonmutating set { storage.replace(with: newValue) }
    }

    public var projectedValue: Binding<[T]> {
        Binding(
            get: { storage.items },
            set: { storage.replace(with: $0) }
        )
    }
}
```
