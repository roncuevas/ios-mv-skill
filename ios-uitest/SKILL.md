---
name: ios-uitest
description: "Genera UI tests iOS con Robot Pattern para tests legibles y mantenibles. Usar cuando el usuario pida: crear UI tests, testear una pantalla, agregar tests de interfaz, verificar flujos de usuario, implementar tests end-to-end, o crear robots para automatización de UI con XCTest y accessibility identifiers."
---

# Generar iOS UI Test con Robot Pattern

$ARGUMENTS = Nombre de la feature a testear (ej: "Profile" o "Login")

## Instrucciones

1. Parsear el nombre de la feature de $ARGUMENTS
2. Buscar si ya existe estructura de UI tests en el proyecto
3. Generar Robot, Elements y Tests para la feature
4. Indicar qué accessibility identifiers agregar a las Views

## Estructura a Generar

```
{App}UITests/
├── Extensions/
│   └── XCUIElement+Extensions.swift
├── Elements/
│   └── {Feature}Elements.swift
├── Robots/
│   ├── Robot.swift              # Si no existe
│   ├── AppRobot.swift           # Si no existe
│   └── {Feature}Robot.swift
└── Tests/
    └── {Feature}Tests.swift
```

## Template: Robot.swift (Base)

```swift
import XCTest

protocol Robot {
    var app: XCUIApplication { get }
}

extension Robot {
    var app: XCUIApplication { XCUIApplication() }
}
```

## Template: AppRobot.swift

```swift
import XCTest

final class AppRobot: Robot {
    let app: XCUIApplication

    init() {
        self.app = XCUIApplication()
    }

    @discardableResult
    func launchApp() -> Self {
        app.launch()
        return self
    }

    @discardableResult
    func launchForUITests() -> Self {
        app.launchArguments = ["-UITests"]
        app.launch()
        return self
    }

    @discardableResult
    func tapTabBar(_ tab: String) -> Self {
        app.tabBars.buttons[tab].tap()
        return self
    }

    func goTo{Feature}() -> {Feature}Robot {
        app.buttons[{Feature}Elements.navigationButton].tap()
        return {Feature}Robot(app: app)
    }
}
```

## Template: {Feature}Elements.swift

```swift
import Foundation

enum {Feature}Elements {
    // Navigation
    static let navigationButton = "{Feature}NavigationButton"
    static let backButton = "Back"

    // List
    static let list = "{Feature}List"
    static let addButton = "Add"
    static let deleteButton = "Delete"

    // Form fields
    static let titleField = "{Feature}TitleField"
    static let descriptionField = "{Feature}DescriptionField"

    // Actions
    static let saveButton = "Save"
    static let cancelButton = "Cancel"
    static let editButton = "Edit"
}
```

## Template: {Feature}Robot.swift

```swift
import XCTest

final class {Feature}Robot: Robot {
    let app: XCUIApplication

    init(app: XCUIApplication) {
        self.app = app
        XCTAssertTrue(
            app.navigationBars["{Feature}"].waitForExistence(timeout: 5),
            "Not on {Feature} screen"
        )
    }

    // MARK: - Actions

    @discardableResult
    func tapAdd() -> Self {
        app.buttons[{Feature}Elements.addButton].tap()
        return self
    }

    @discardableResult
    func tapItem(named name: String) -> Self {
        app.cells.staticTexts[name].tap()
        return self
    }

    @discardableResult
    func inputTitle(_ text: String) -> Self {
        let field = app.textFields[{Feature}Elements.titleField]
        field.tap()
        field.typeText(text)
        return self
    }

    @discardableResult
    func clearAndInputTitle(_ text: String) -> Self {
        let field = app.textFields[{Feature}Elements.titleField]
        field.tap()
        field.clearText()
        field.typeText(text)
        return self
    }

    @discardableResult
    func tapSave() -> Self {
        app.buttons[{Feature}Elements.saveButton].tap()
        return self
    }

    @discardableResult
    func tapCancel() -> Self {
        app.buttons[{Feature}Elements.cancelButton].tap()
        return self
    }

    @discardableResult
    func deleteItem(named name: String) -> Self {
        let cell = app.cells.containing(.staticText, identifier: name).firstMatch
        cell.swipeLeft()
        app.buttons[{Feature}Elements.deleteButton].tap()
        return self
    }

    // MARK: - Verifications

    @discardableResult
    func verifyItemExists(named name: String) -> Self {
        XCTAssertTrue(
            app.cells.staticTexts[name].waitForExistence(timeout: 5),
            "Item '\(name)' should exist"
        )
        return self
    }

    @discardableResult
    func verifyItemNotExists(named name: String) -> Self {
        XCTAssertFalse(
            app.cells.staticTexts[name].exists,
            "Item '\(name)' should not exist"
        )
        return self
    }

    @discardableResult
    func verifyTitle(_ title: String) -> Self {
        XCTAssertTrue(
            app.navigationBars[title].waitForExistence(timeout: 5),
            "Title should be '\(title)'"
        )
        return self
    }

    @discardableResult
    func verifyEmptyState() -> Self {
        XCTAssertTrue(
            app.staticTexts["No items"].waitForExistence(timeout: 5),
            "Empty state should be visible"
        )
        return self
    }

    // MARK: - Navigation

    func goToDetail(named name: String) -> {Feature}DetailRobot {
        app.cells.staticTexts[name].tap()
        return {Feature}DetailRobot(app: app)
    }

    func goToEdit() -> {Feature}EditRobot {
        app.buttons[{Feature}Elements.editButton].tap()
        return {Feature}EditRobot(app: app)
    }

    func goBack() -> AppRobot {
        app.buttons[{Feature}Elements.backButton].tap()
        return AppRobot()
    }
}
```

## Template: {Feature}Tests.swift

```swift
import XCTest

final class {Feature}Tests: XCTestCase {

    override func setUpWithError() throws {
        continueAfterFailure = false
    }

    // MARK: - List Tests

    func test_addNew{Feature}_shouldAppearInList() {
        AppRobot()
            .launchForUITests()
            .goTo{Feature}()
            .tapAdd()
            .inputTitle("Test Item")
            .tapSave()
            .verifyItemExists(named: "Test Item")
    }

    func test_delete{Feature}_shouldRemoveFromList() {
        AppRobot()
            .launchForUITests()
            .goTo{Feature}()
            .tapAdd()
            .inputTitle("To Delete")
            .tapSave()
            .verifyItemExists(named: "To Delete")
            .deleteItem(named: "To Delete")
            .verifyItemNotExists(named: "To Delete")
    }

    func test_cancel_shouldNotAddItem() {
        AppRobot()
            .launchForUITests()
            .goTo{Feature}()
            .tapAdd()
            .inputTitle("Cancelled")
            .tapCancel()
            .verifyItemNotExists(named: "Cancelled")
    }

    // MARK: - Detail Tests

    func test_tapItem_shouldShowDetail() {
        AppRobot()
            .launchForUITests()
            .goTo{Feature}()
            .tapAdd()
            .inputTitle("Detail Test")
            .tapSave()
            .goToDetail(named: "Detail Test")
            .verifyTitle("Detail Test")
    }

    // MARK: - Edit Tests

    func test_editItem_shouldUpdateInList() {
        AppRobot()
            .launchForUITests()
            .goTo{Feature}()
            .tapAdd()
            .inputTitle("Original")
            .tapSave()
            .goToDetail(named: "Original")
            .goToEdit()
            .clearAndInputTitle("Updated")
            .tapSave()
            .goBack()
            .verifyItemExists(named: "Updated")
            .verifyItemNotExists(named: "Original")
    }
}
```

## XCUIElement Extensions

```swift
// {App}UITests/Extensions/XCUIElement+Extensions.swift
import XCTest

extension XCUIElement {
    func clearText() {
        guard let value = self.value as? String else { return }
        typeText(String(repeating: XCUIKeyboardKey.delete.rawValue, count: value.count))
    }

    func waitAndTap(timeout: TimeInterval = 5) {
        XCTAssertTrue(self.waitForExistence(timeout: timeout))
        self.tap()
    }
}
```

## Accessibility Identifiers a Agregar en Views

```swift
// Contenedor principal
.accessibilityIdentifier("{Feature}Screen")

// TextFields
TextField("Title", text: $title)
    .accessibilityIdentifier({Feature}Elements.titleField)

// Buttons
Button("Save") { }
    .accessibilityIdentifier({Feature}Elements.saveButton)

// Lists
List { }
    .accessibilityIdentifier({Feature}Elements.list)
```

## Naming Conventions

- **Robot names**: `{Feature}Robot`, `{Feature}DetailRobot`, `{Feature}EditRobot`
- **Element enums**: `{Feature}Elements`
- **Test classes**: `{Feature}Tests`
- **Test methods**: `test_{action}_should{Outcome}`
- **Accessibility IDs**: PascalCase descriptivo, ej: `ProfileTitleField`

## Para Proyectos Existentes

- Si ya tienen UI tests: revisar el patrón existente y adaptarse
- Si no usan Robot Pattern: sugerir migración gradual
- Si no tienen UI tests: crear el target de UI tests y generar toda la estructura base

## Tips

1. Cada Robot verifica en `init` que está en la pantalla correcta
2. Métodos de acción retornan `Self` para encadenar (fluent API)
3. Métodos de navegación retornan el Robot de destino
4. Verificaciones retornan `Self` para continuar el flujo
5. Usar `@discardableResult` para flexibilidad

Generar los UI tests para: $ARGUMENTS
