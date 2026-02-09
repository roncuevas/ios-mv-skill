---
name: ios-service
description: "Genera un servicio iOS completo con protocolo, implementación real, mock, error enum, environment key y permisos opcionales. Usar cuando el usuario pida: crear un servicio, agregar networking, crear un mock, integrar un API, manejar permisos del sistema (cámara, micrófono, speech), implementar streaming con AsyncThrowingStream, o cualquier capa de datos/servicios con protocol + mock pattern."
---

# Generar iOS Service

$ARGUMENTS = Nombre del servicio y métodos (ej: "Auth con login y logout", "Speech con transcripción en streaming")

## Instrucciones

1. Parsear el nombre del servicio y los métodos de $ARGUMENTS
2. Determinar la ubicación correcta (Services/ package o carpeta existente)
3. Generar los archivos del servicio (4-6 archivos según necesidad)
4. Si requiere permisos, generar Permissions.swift
5. Si tiene assets (audio, etc.), configurar Resources/
6. Actualizar Package.swift si existe
7. Indicar cómo actualizar AppScene para inyectar el servicio

## Estructura de Archivos

```
Services/Sources/{Name}Service/
├── {Name}ServiceProtocol.swift
├── {Name}ServiceError.swift
├── {Name}Service.swift
├── Mock{Name}Service.swift
├── EnvironmentValues+{Name}Service.swift
├── Permissions.swift              # Si requiere permisos
└── Resources/                     # Si tiene assets
```

## Template: {Name}ServiceProtocol.swift

```swift
import Foundation

public protocol {Name}ServiceProtocol: Sendable {
    // Métodos síncronos
    func doSomething() throws({Name}ServiceError)

    // Métodos async
    func fetchData() async throws({Name}ServiceError) -> Data

    // Streaming (para datos continuos)
    func startStreaming() async throws({Name}ServiceError) -> AsyncThrowingStream<String, Error>
}
```

## Template: {Name}ServiceError.swift

```swift
import Foundation

public enum {Name}ServiceError: Error, LocalizedError {
    case notAuthorized
    case notAvailable
    case invalidInput(String)
    case networkError(underlying: Error)
    case unknown

    public var errorDescription: String? {
        switch self {
        case .notAuthorized: return "{Name} access not authorized"
        case .notAvailable: return "{Name} service not available"
        case .invalidInput(let message): return "Invalid input: \(message)"
        case .networkError(let error): return "Network error: \(error.localizedDescription)"
        case .unknown: return "An unknown error occurred"
        }
    }
}
```

## Template: {Name}Service.swift

```swift
import Foundation

public final class {Name}Service: @unchecked Sendable, {Name}ServiceProtocol {
    public init() {}

    public func doSomething() throws({Name}ServiceError) {
        // Implementación real
    }

    public func fetchData() async throws({Name}ServiceError) -> Data {
        // Implementación real con async/await
    }

    public func startStreaming() async throws({Name}ServiceError) -> AsyncThrowingStream<String, Error> {
        AsyncThrowingStream { continuation in
            // continuation.yield(value)
            // continuation.finish() o continuation.finish(throwing: error)
        }
    }
}
```

## Template: Mock{Name}Service.swift

```swift
import Foundation

public final class Mock{Name}Service: {Name}ServiceProtocol {
    public var mockData: Data = Data()
    public var shouldFail: Bool = false

    public init() {}

    public func doSomething() throws({Name}ServiceError) {
        if shouldFail { throw .unknown }
    }

    public func fetchData() async throws({Name}ServiceError) -> Data {
        try await Task.sleep(for: .milliseconds(500))
        if shouldFail { throw .networkError(underlying: URLError(.notConnectedToInternet)) }
        return mockData
    }

    public func startStreaming() async throws({Name}ServiceError) -> AsyncThrowingStream<String, Error> {
        AsyncThrowingStream { continuation in
            Task {
                for text in ["Hello", "World", "Streaming", "Complete"] {
                    try await Task.sleep(for: .milliseconds(300))
                    continuation.yield(text)
                }
                continuation.finish()
            }
        }
    }
}
```

## Template: EnvironmentValues+{Name}Service.swift

```swift
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

## Template: Permissions.swift (si requiere permisos)

```swift
import AVFoundation
import Speech  // o el framework necesario

extension AVAudioSession {
    static func hasRecordPermission() async -> Bool {
        await withCheckedContinuation { continuation in
            AVAudioSession.sharedInstance().requestRecordPermission { granted in
                continuation.resume(returning: granted)
            }
        }
    }
}

extension SFSpeechRecognizer {
    static func hasPermission() async -> Bool {
        await withCheckedContinuation { continuation in
            SFSpeechRecognizer.requestAuthorization { status in
                continuation.resume(returning: status == .authorized)
            }
        }
    }
}

extension AVCaptureDevice {
    static func hasCameraPermission() async -> Bool {
        let status = AVCaptureDevice.authorizationStatus(for: .video)
        if status == .authorized { return true }
        return await AVCaptureDevice.requestAccess(for: .video)
    }
}
```

## Actualizar Package.swift

```swift
// Agregar target
.target(
    name: "{Name}Service",
    dependencies: [],
    resources: [.process("Resources")]  // Solo si tiene assets
),

// Agregar al producto
products: [
    .library(
        name: "Services",
        targets: ["ExistingService", "{Name}Service"]
    )
],
```

Para acceder a recursos del bundle:
```swift
guard let url = Bundle.module.url(forResource: "sound", withExtension: "wav") else {
    throw {Name}ServiceError.notAvailable
}
```

## Actualizar AppScene

```swift
// Agregar propiedad
private let {camelCase}Service: any {Name}ServiceProtocol

// En init()
if RuntimeEnvironment.current.isPhysicalDevice {
    self.{camelCase}Service = {Name}Service()
} else {
    self.{camelCase}Service = Mock{Name}Service()
}

// En body
.environment(\.{camelCase}Service, {camelCase}Service)
```

## Para Proyectos sin SwiftUI Environment

### Alternativa: Singleton
```swift
public final class {Name}Service {
    public static let shared = {Name}Service()
    private init() {}
}
```

### Alternativa: Constructor Injection
```swift
class ViewModel {
    private let service: {Name}ServiceProtocol
    init(service: {Name}ServiceProtocol = {Name}Service()) {
        self.service = service
    }
}
```

Generar el servicio basándote en: $ARGUMENTS
