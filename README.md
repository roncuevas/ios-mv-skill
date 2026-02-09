# iOS MV Skill

Claude Code skills for iOS development with modular SPM architecture. Generates features, services (protocol + mock + environment key), unit tests (Swift Testing/XCTest), and UI tests (Robot Pattern). Based on [JamesSedlacek/Scrumdinger](https://github.com/JamesSedlacek/Scrumdinger).

## Skills

### `/ios`

Index of all available iOS commands. Shows a quick reference table with usage examples and the base architecture overview.

### `/ios-architecture`

Guides the creation or migration of iOS projects to a modular SPM architecture with 4 packages: **Library**, **Models**, **Services**, and **ViewComponents**. Includes `Package.swift` templates with enums for target consistency, migration order for existing projects, and references for detailed patterns (RuntimeEnvironment, DataStorage, Routable, AppScene DI, SF Symbols, SwiftLint).

### `/ios-feature`

Generates a complete feature module with **Screen**, **Route**, **Views** (List, Detail, Edit), and **Components** (Card). Follows the Routable navigation pattern, includes LocalizedStringKey constants, and provides integration guidance for the main navigation.

Usage: `/ios-feature Profile con vista, edición y settings`

### `/ios-service`

Generates a complete service layer with **Protocol**, **Error enum**, **Implementation** (`@unchecked Sendable`), **Mock** (configurable for tests), and **EnvironmentValues extension** for SwiftUI dependency injection. Supports typed throws, `AsyncThrowingStream` for streaming, and optional system permissions (camera, microphone, speech).

Usage: `/ios-service Speech con transcripción en streaming`

### `/ios-unittest`

Generates unit tests using **Swift Testing** (recommended for Swift 6+) or **XCTest**. Includes templates with Arrange/Act/Assert pattern, parameterized tests, tags for organization, and examples for extensions, models, and services with mocks.

Usage: `/ios-unittest DailyScrum extensions`

### `/ios-uitest`

Generates UI tests following the **Robot Pattern** for readable and maintainable tests. Creates **Elements** (accessibility identifiers), **Robots** (fluent API with `@discardableResult`), and **Tests** (chained actions and verifications). Includes `XCUIElement` extensions and accessibility identifier guidance.

Usage: `/ios-uitest Profile`

## Architecture Overview

```
Project/
├── App/                    # Entry point (@main)
├── Library/                # Features + Configuration + Resources
│   ├── Configuration/      # AppScene with dependency injection
│   ├── Features/           # Screens organized by feature
│   └── Resources/          # Colors, Strings, SFSymbols
├── Models/                 # Data models + Enums + Protocols + Tests
├── Services/               # Protocol-based services with mocks
└── ViewComponents/         # Reusable UI (Buttons, Labels, Styles)
```

## Installation

Add skills to your Claude Code configuration:

```bash
claude mcp add-skill /path/to/ios-mv-skill/ios
claude mcp add-skill /path/to/ios-mv-skill/ios-architecture
claude mcp add-skill /path/to/ios-mv-skill/ios-feature
claude mcp add-skill /path/to/ios-mv-skill/ios-service
claude mcp add-skill /path/to/ios-mv-skill/ios-unittest
claude mcp add-skill /path/to/ios-mv-skill/ios-uitest
```

## License

MIT
