---
name: ios-feature
description: "Genera una feature iOS completa con Screen, Route, Views y Components siguiendo arquitectura modular. Usar cuando el usuario pida: crear una nueva pantalla, agregar una feature, generar un módulo con lista/detalle/edición, crear una sección de la app con navegación, o cualquier nueva pantalla con su estructura de archivos."
---

# Generar iOS Feature

$ARGUMENTS = Nombre de la feature y descripción (ej: "Profile con vista, edición y settings")

## Instrucciones

1. Parsear el nombre y las vistas necesarias de $ARGUMENTS
2. Determinar la ubicación (Library/Sources/Features/ o estructura existente)
3. Generar los archivos de la feature
4. Agregar los strings necesarios
5. Indicar cómo integrar con la navegación principal

## Estructura a Generar

```
Library/Sources/Features/{Feature}/
├── {Feature}Screen.swift
├── {Feature}Route.swift
├── Views/
│   ├── {Feature}ListView.swift
│   ├── {Feature}DetailView.swift
│   └── {Feature}EditView.swift
└── Components/
    └── {Feature}Card.swift
```

## Template: {Feature}Route.swift

```swift
import SwiftUI
import Models

public enum {Feature}Route: Routable {
    case list
    case detail({Model})
    case edit({Model})
    case add

    @ViewBuilder
    public var body: some View {
        switch self {
        case .list:
            {Feature}ListView()
        case .detail(let item):
            {Feature}DetailView(item: item)
        case .edit(let item):
            {Feature}EditView(item: item)
        case .add:
            {Feature}AddView()
        }
    }
}
```

## Template: {Feature}Screen.swift

```swift
import SwiftUI
import Models

public struct {Feature}Screen: View {
    @State private var navigationPath = NavigationPath()
    @State private var items: [{Model}] = []

    public init() {}

    public var body: some View {
        NavigationStack(path: $navigationPath) {
            {Feature}ListView(items: $items)
                .navigationDestination(for: {Feature}Route.self) { route in
                    route.body
                }
        }
    }
}
```

## Template: {Feature}ListView.swift

```swift
import SwiftUI
import Models
import ViewComponents

struct {Feature}ListView: View {
    @Binding var items: [{Model}]
    @Environment(\.{serviceName}Service) private var service

    var body: some View {
        List {
            ForEach(items) { item in
                NavigationLink(value: {Feature}Route.detail(item)) {
                    {Feature}Card(item: item)
                }
            }
            .onDelete(perform: deleteItems)
        }
        .navigationTitle(.{feature}Title)
        .toolbar {
            ToolbarItem(placement: .primaryAction) {
                NavigationLink(value: {Feature}Route.add) {
                    Image.plus
                }
            }
        }
    }

    private func deleteItems(at offsets: IndexSet) {
        items.remove(atOffsets: offsets)
    }
}
```

## Template: {Feature}DetailView.swift

```swift
import SwiftUI
import Models

struct {Feature}DetailView: View {
    let item: {Model}
    @State private var showingEdit = false

    var body: some View {
        List {
            Section {
                // Contenido del detalle
            }
        }
        .navigationTitle(item.title)
        .toolbar {
            ToolbarItem(placement: .primaryAction) {
                Button { showingEdit = true } label: {
                    Image.pencil
                }
            }
        }
        .sheet(isPresented: $showingEdit) {
            NavigationStack {
                {Feature}EditView(item: item) { updated in
                    // Guardar cambios
                }
            }
        }
    }
}
```

## Template: {Feature}EditView.swift

```swift
import SwiftUI
import Models

struct {Feature}EditView: View {
    @Environment(\.dismiss) private var dismiss
    @State private var draft: {Model}
    let onSave: ({Model}) -> Void

    init(item: {Model}, onSave: @escaping ({Model}) -> Void) {
        _draft = State(initialValue: item)
        self.onSave = onSave
    }

    var body: some View {
        Form {
            // Campos editables
        }
        .navigationTitle(.edit)
        .toolbar {
            ToolbarItem(placement: .cancellationAction) {
                Button(.cancel) { dismiss() }
            }
            ToolbarItem(placement: .confirmationAction) {
                Button(.save) {
                    onSave(draft)
                    dismiss()
                }
            }
        }
    }
}
```

## Template: {Feature}Card.swift

```swift
import SwiftUI
import Models

struct {Feature}Card: View {
    let item: {Model}

    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            Text(item.title)
                .font(.headline)
            HStack {
                // Metadata labels
            }
            .font(.caption)
        }
        .padding(.vertical, 4)
    }
}
```

## Strings a Agregar

```swift
// En Library/Sources/Resources/LocalizedStringKey+Constants.swift
// MARK: - {Feature}
public extension LocalizedStringKey {
    static let {feature}Title: Self = "{Feature}"
    static let {feature}Empty: Self = "No items yet"
    static let {feature}Add: Self = "Add {Feature}"
}
```

## Integración con Navegación Principal

Agregar case al Route principal:

```swift
case {featureName}
case {featureName}Detail({Model})

@ViewBuilder
var body: some View {
    switch self {
    // ... otros cases
    case .{featureName}:
        {Feature}Screen()
    case .{featureName}Detail(let item):
        {Feature}DetailView(item: item)
    }
}
```

## Para Proyectos Existentes

- Si no usa Routable: usar NavigationLink directo o el sistema de navegación existente
- Si no tiene estructura modular: crear la carpeta de la feature en el lugar apropiado y ajustar imports
- Adaptar el patrón al estilo del proyecto

Generar la feature basándote en: $ARGUMENTS
