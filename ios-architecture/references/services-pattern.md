# Patrón de Servicios e Inyección de Dependencias

## AppScene con Inyección de Dependencias

```swift
// Library/Sources/Configuration/AppScene.swift
import SwiftUI
import Services
import Models

@MainActor
public struct AppScene<Content: View>: Scene {
    private let speechService: any SpeechServiceProtocol
    private let audioService: any AudioServiceProtocol
    private let fileService: any FileServiceProtocol

    @DataStore(key: .dataKey, defaultValue: [])
    private var items: [Item]

    private let content: ([Item]) -> Content

    public init(@ViewBuilder content: @escaping ([Item]) -> Content) {
        self.content = content

        if RuntimeEnvironment.current.isPhysicalDevice {
            self.speechService = SpeechService()
            self.audioService = AudioService()
            self.fileService = FileService()
        } else {
            self.speechService = MockSpeechService()
            self.audioService = MockAudioService()
            self.fileService = MockFileService()
        }
    }

    public var body: some Scene {
        WindowGroup {
            content(items)
                .environment(\.speechService, speechService)
                .environment(\.audioService, audioService)
                .environment(\.fileService, fileService)
        }
    }
}
```

## App Entry Point

```swift
// {ProjectName}/App/{ProjectName}App.swift
import SwiftUI
import Library

@main
struct MyAppApp: App {
    var body: some Scene {
        AppScene()
    }
}
```

## EnvironmentKey Pattern

Cada servicio expone su protocolo via SwiftUI Environment:

```swift
// Services/Sources/{Name}Service/EnvironmentValues+{Name}Service.swift
import SwiftUI

private struct {Name}ServiceKey: EnvironmentKey {
    static let defaultValue: any {Name}ServiceProtocol = Mock{Name}Service()
}

public extension EnvironmentValues {
    var {camelCase}Service: any {Name}ServiceProtocol {
        get { self[{Name}ServiceKey.self] }
        set { self[{Name}ServiceKey.self] = newValue }
    }
}
```

### Uso en Views

```swift
struct MyView: View {
    @Environment(\.networkService) private var networkService

    var body: some View {
        // ...
    }
}
```
