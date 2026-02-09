---
name: ios-unittest
description: "Genera unit tests iOS usando Swift Testing framework (recomendado) o XCTest. Usar cuando el usuario pida: crear tests unitarios, testear un modelo, testear extensiones, testear servicios con mocks, agregar cobertura de tests, o cualquier test que no involucre UI."
---

# Generar iOS Unit Tests

$ARGUMENTS = Nombre del módulo/clase a testear (ej: "DailyScrum extensions", "AuthService")

## Instrucciones

1. Parsear qué se va a testear de $ARGUMENTS
2. Identificar si el proyecto usa Swift Testing o XCTest
3. Generar tests en la ubicación correcta (Models/Tests/, Services/Tests/, etc.)
4. Seguir el patrón de nombrado existente

## Estructura de Tests

```
Models/
├── Sources/
│   └── Extensions/
│       └── DailyScrum+SecondsPerSpeaker.swift
└── Tests/
    └── ExtensionsTests/
        └── DailyScrumTests.swift
```

## Swift Testing Framework (Recomendado - Swift 6+)

### Template Básico

```swift
import Testing
@testable import Extensions
@testable import Models

struct {ClassName}Tests {

    let sampleItem = Item(id: UUID(), name: "Test")

    // MARK: - {PropertyOrMethod} Tests

    @Test func {methodName}_with{Condition}_returns{Expected}() {
        // Arrange
        let input = ...
        // Act
        let result = input.methodName()
        // Assert
        #expect(result == expectedValue)
    }

    @Test func {methodName}_with{EdgeCase}_throws{Error}() throws {
        let invalidInput = ...
        #expect(throws: SomeError.self) {
            try invalidInput.methodName()
        }
    }
}
```

### Assertions

```swift
#expect(result == expected)          // Igualdad
#expect(result != unexpected)
#expect(condition)                   // Booleanos
#expect(optional == nil)             // Nil
#expect(optional != nil)
#expect(throws: ErrorType.self) { try riskyOperation() }    // Throws
#expect(throws: Never.self) { try safeOperation() }
#expect(result == expected, "Descripción del fallo")         // Con mensaje

// Async
@Test func asyncTest() async throws {
    let result = await asyncOperation()
    #expect(result == expected)
}
```

### Tags y Organización

```swift
extension Tag {
    @Tag static var model: Self
    @Tag static var integration: Self
}

struct ItemTests {
    @Test(.tags(.model))
    func basicTest() { }

    @Test(.tags(.integration))
    func integrationTest() async { }
}
```

### Parametrized Tests

```swift
@Test(arguments: [1, 2, 3, 5, 8, 13])
func fibonacciNumbers(value: Int) {
    #expect(isFibonacci(value))
}

@Test(arguments: [
    ("hello", 5),
    ("world", 5),
    ("", 0)
])
func stringLength(input: String, expected: Int) {
    #expect(input.count == expected)
}
```

## XCTest (Legacy)

### Template Básico

```swift
import XCTest
@testable import Extensions
@testable import Models

final class {ClassName}Tests: XCTestCase {

    var sut: SystemUnderTest!

    override func setUpWithError() throws {
        try super.setUpWithError()
        sut = SystemUnderTest()
    }

    override func tearDownWithError() throws {
        sut = nil
        try super.tearDownWithError()
    }

    func test_{methodName}_with{Condition}_returns{Expected}() {
        let input = ...
        let result = sut.methodName(input)
        XCTAssertEqual(result, expected)
    }
}
```

### Assertions

```swift
XCTAssertEqual(result, expected)
XCTAssertTrue(condition)
XCTAssertFalse(condition)
XCTAssertNil(optional)
XCTAssertNotNil(optional)
XCTAssertGreaterThan(a, b)
XCTAssertThrowsError(try riskyOperation())
XCTAssertNoThrow(try safeOperation())
```

## Ejemplo: Extension Tests

```swift
import Testing
@testable import Extensions
@testable import Models

struct DailyScrumTests {

    @Test func secondsPerSpeaker_withMultipleAttendees_dividesEvenly() {
        let scrum = DailyScrum(
            title: "Test",
            attendees: ["Alice", "Bob"],
            length: .minutes(10),
            theme: .bubblegum
        )
        #expect(scrum.secondsPerSpeaker == .minutes(5))
    }

    @Test func secondsPerSpeaker_withNoAttendees_returnsZero() {
        let scrum = DailyScrum(
            title: "Empty",
            attendees: [],
            length: .minutes(10),
            theme: .navy
        )
        #expect(scrum.secondsPerSpeaker == 0)
    }
}
```

## Ejemplo: Service Tests con Mocks

```swift
import Testing
@testable import Services

struct AuthServiceTests {

    let mockService = MockAuthService()

    @Test func login_withValidCredentials_returnsUser() async throws {
        mockService.shouldFail = false
        let user = try await mockService.login(email: "test@test.com", password: "password")
        #expect(user.email == "test@test.com")
    }

    @Test func login_withInvalidCredentials_throwsError() async {
        mockService.shouldFail = true
        await #expect(throws: AuthServiceError.self) {
            try await mockService.login(email: "bad", password: "bad")
        }
    }
}
```

## Package.swift Test Target

```swift
.testTarget(
    name: "ExtensionsTests",
    dependencies: [
        .target(name: "Extensions"),
        .target(name: "Models")
    ]
)
```

## Naming Conventions

- **Test structs/classes**: `{ClassName}Tests`
- **Swift Testing**: `{method}_with{Condition}_{expectedResult}`
- **XCTest**: `test_{method}_with{Condition}_{expectedResult}`
- **Edge cases**: `_withEmptyInput_`, `_withNilValue_`, `_withInvalidData_`
- **Expected results**: `_returnsNil`, `_throwsError`, `_updatesState`

## Para Proyectos Existentes

1. Revisar si ya tienen tests y qué framework usan
2. Seguir el mismo patrón de organización
3. Si no tienen tests, empezar por:
   - Extensions (fáciles de testear, sin dependencias)
   - Models (lógica de negocio pura)
   - Services (usando mocks)

Generar los unit tests basándote en: $ARGUMENTS
