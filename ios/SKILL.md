---
name: ios
description: "Lista de comandos disponibles para desarrollo iOS con arquitectura modular SPM. Usar cuando el usuario pregunte qué comandos iOS hay disponibles, quiera ver un resumen de herramientas, o pida ayuda general sobre el flujo de desarrollo iOS modular."
---

# iOS Development Commands

| Comando | Descripción | Ejemplo |
|---------|-------------|---------|
| `/ios-architecture` | Guía completa de arquitectura modular SPM | `/ios-architecture` |
| `/ios-service` | Genera servicio con protocolo + mock + permisos | `/ios-service Auth con login y logout` |
| `/ios-feature` | Genera feature con Screen, Route, Views | `/ios-feature Profile con vista y edición` |
| `/ios-unittest` | Genera unit tests con Swift Testing | `/ios-unittest DailyScrum extensions` |
| `/ios-uitest` | Genera UI tests con Robot Pattern | `/ios-uitest Profile` |

## Uso Rápido

```
/ios-architecture              # Analizar proyecto y aplicar arquitectura modular
/ios-service {Nombre} con {métodos}   # Ej: /ios-service Speech con transcripción en streaming
/ios-feature {Nombre} con {vistas}    # Ej: /ios-feature Settings con lista, detalle y about
/ios-unittest {Módulo o clase}        # Ej: /ios-unittest History model
/ios-uitest {Feature}                 # Ej: /ios-uitest Settings
```

## Arquitectura Base

```
Project/
├── App/                    # Entry point (@main)
├── Library/                # Features + Configuration + Resources
├── Models/                 # Data models + Enums + Protocols + Tests
├── Services/               # Protocol-based services with mocks
└── ViewComponents/         # Reusable UI components
```

Ejecutar `/ios-architecture` para ver la guía completa con patrones y templates.
