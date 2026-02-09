---
name: ios-architecture
description: "Guía completa para crear o adaptar proyectos iOS con arquitectura modular usando Swift Package Manager. Usar cuando el usuario quiera: crear un nuevo proyecto iOS modular, migrar un proyecto existente a SPM, entender la estructura de packages (Library, Models, Services, ViewComponents), configurar inyección de dependencias con SwiftUI Environment, implementar RuntimeEnvironment, configurar Package.swift con enums, o cualquier decisión arquitectónica del proyecto iOS."
---

# iOS Modular Architecture

Analizar el proyecto actual y determinar:
1. Si es un proyecto nuevo o existente
2. Qué packages ya existen (si hay)
3. Qué se necesita crear o refactorizar

## Estructura Target

```
{ProjectName}/
├── {ProjectName}/                # App bundle principal
│   └── App/{ProjectName}App.swift
├── Library/                      # Features principales
│   ├── Sources/
│   │   ├── Configuration/        # AppScene, inyección de dependencias
│   │   ├── Features/             # Screens por feature
│   │   └── Resources/            # Colors, Strings, SFSymbols, Media.xcassets
│   └── Package.swift
├── Models/                       # Modelos de datos
│   ├── Sources/
│   │   ├── Models/               # Structs + DataStorage
│   │   ├── Enumerations/         # Theme, RuntimeEnvironment
│   │   ├── Extensions/           # TimeInterval, etc.
│   │   └── Protocols/            # Routable, ToolbarView, ViewLifecycle
│   ├── Tests/                    # Unit tests con Swift Testing
│   └── Package.swift
├── Services/                     # Capa de servicios
│   ├── Sources/
│   │   └── {Name}Service/
│   │       ├── {Name}Service.swift
│   │       ├── {Name}ServiceProtocol.swift
│   │       ├── {Name}ServiceError.swift
│   │       ├── Mock{Name}Service.swift
│   │       ├── EnvironmentValues+{Name}Service.swift
│   │       └── Permissions.swift          # Si requiere permisos
│   └── Package.swift
└── ViewComponents/               # UI reutilizable
    ├── Sources/
    │   ├── Buttons/
    │   ├── Labels/
    │   └── ProgressViews/
    └── Package.swift
```

## Para Proyectos Existentes

Evaluar la estructura actual e identificar qué código puede moverse a packages.

### Orden recomendado de migración:
1. `Models/` - Modelos, enums, protocolos (sin dependencias UI)
2. `Services/` - Servicios existentes, crear protocolos y mocks
3. `ViewComponents/` - Componentes UI reutilizables
4. `Library/` - Features y configuración

Principio: **no romper el build** - cada paso debe compilar.

## Package.swift con Enums

```swift
// swift-tools-version: 6.0
import PackageDescription

private enum InternalTarget: String {
    case models = "Models"
    case enumerations = "Enumerations"
    case protocols = "Protocols"
    case extensions = "Extensions"

    var dependency: Target.Dependency { .target(name: rawValue) }
}

let package = Package(
    name: "Models",
    defaultLocalization: "en",
    platforms: [.iOS(.v17), .macOS(.v14)],
    products: [
        .library(
            name: "Models",
            targets: [
                InternalTarget.models.rawValue,
                InternalTarget.enumerations.rawValue,
                InternalTarget.protocols.rawValue,
                InternalTarget.extensions.rawValue
            ]
        )
    ],
    targets: [
        .target(name: InternalTarget.enumerations.rawValue),
        .target(name: InternalTarget.protocols.rawValue),
        .target(
            name: InternalTarget.models.rawValue,
            dependencies: [
                InternalTarget.enumerations.dependency,
                InternalTarget.protocols.dependency
            ]
        ),
        .target(
            name: InternalTarget.extensions.rawValue,
            dependencies: [InternalTarget.models.dependency]
        ),
        .testTarget(
            name: "ExtensionsTests",
            dependencies: [InternalTarget.extensions.dependency]
        )
    ]
)
```

### Library Package.swift

```swift
// swift-tools-version: 6.0
import PackageDescription

let package = Package(
    name: "Library",
    defaultLocalization: "en",
    platforms: [.iOS(.v17), .macOS(.v14)],
    products: [
        .library(name: "Library", targets: ["Features"])
    ],
    dependencies: [
        .package(path: "../Models"),
        .package(path: "../Services"),
        .package(path: "../ViewComponents")
    ],
    targets: [
        .target(name: "Configuration", dependencies: ["Features", "Services"]),
        .target(name: "Resources", dependencies: ["Models"]),
        .target(name: "Features", dependencies: ["Models", "Resources", "ViewComponents", "Services"])
    ]
)
```

## Patrones Incluidos

- **Swift Package Manager** modular con 4 packages
- **Inyección de dependencias** via SwiftUI Environment
- **RuntimeEnvironment** para detectar simulator/device/previews
- **Package.swift con enums** para consistencia de targets
- **DataStorage** clase `@Observable` + **DataStore** property wrapper
- **Routable** para navegación type-safe
- **ToolbarView** y **ViewLifecycle** protocolos
- **Protocol + Mock** para cada servicio con `@unchecked Sendable`
- **AsyncThrowingStream** para streaming de datos
- **LocalizedStringKey** extensions organizadas A-Z
- **SFSymbol enum** + extensions para Image/Button/Label
- **Theme enum** con colores en xcassets
- **SwiftLint** con pre-commit hook
- **Swift 6.0** con strict concurrency

## Referencias Detalladas

Consultar según necesidad:

- **Data layer** (RuntimeEnvironment, DataStorage, DataStore): ver [references/data-layer.md](references/data-layer.md)
- **Protocolos** (Routable, ToolbarView, ViewLifecycle): ver [references/protocols.md](references/protocols.md)
- **Patrón de servicios** (AppScene, EnvironmentKey, DI): ver [references/services-pattern.md](references/services-pattern.md)
- **Recursos** (SF Symbols, Strings, Colors, Theme): ver [references/resources.md](references/resources.md)
- **Tooling** (SwiftLint, pre-commit hook): ver [references/tooling.md](references/tooling.md)

## Checklist para Nuevo Proyecto

1. Crear proyecto Xcode con App template
2. Crear carpetas: Library/, Models/, Services/, ViewComponents/
3. Agregar Package.swift a cada carpeta
4. Configurar dependencias entre packages en Xcode
5. Crear RuntimeEnvironment en Models (ver references/data-layer.md)
6. Crear protocolos compartidos (ver references/protocols.md)
7. Crear AppScene con inyección de dependencias (ver references/services-pattern.md)
8. Mover @main App para usar AppScene
9. Crear primer servicio con `/ios-service`
10. Organizar Resources (ver references/resources.md)
11. Configurar SwiftLint (ver references/tooling.md)
