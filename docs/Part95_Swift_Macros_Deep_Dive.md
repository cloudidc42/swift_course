# Part 95: Swift Macros Deep Dive

## บทนำ: ยุคใหม่ของ Swift Metaprogramming

Swift 5.9 แนะนำ **Macros** ซึ่งเป็นหนึ่งในฟีเจอร์ที่ทรงพลังที่สุดในประวัติศาสตร์ของภาษา Swift Macros ช่วยให้เราเขียนโค้ดที่สร้างโค้ดได้ในระหว่าง compile time โดยใช้ SwiftSyntax library ซึ่งเป็น type-safe way ในการจัดการกับ Swift source code

ในบทนี้เราจะเรียนรู้ Swift Macros อย่างละเอียดตั้งแต่พื้นฐานจนถึงการสร้าง production-ready macros ที่ซับซ้อน

---

## 1. Swift Macros Overview (Swift 5.9+)

### 1.1 ปัญหาที่ Macros แก้ไข

ก่อนที่จะมี Macros นักพัฒนา Swift ต้องเผชิญกับปัญหาหลายอย่าง:

**Boilerplate Code Problem:**
```swift
// ก่อน Macros: ต้องเขียน Equatable ด้วยมือทุกครั้ง
struct Person {
    let name: String
    let age: Int
    let email: String
}

extension Person: Equatable {
    static func == (lhs: Person, rhs: Person) -> Bool {
        return lhs.name == rhs.name &&
               lhs.age == rhs.age &&
               lhs.email == rhs.email
    }
}

extension Person: Hashable {
    func hash(into hasher: inout Hasher) {
        hasher.combine(name)
        hasher.combine(age)
        hasher.combine(email)
    }
}

// หลัง Macros: ใช้แค่ @AutoEquatable
@AutoEquatable
@AutoHashable
struct Person {
    let name: String
    let age: Int
    let email: String
}
```

**Code Generation Problems ก่อน Macros:**
1. ต้องใช้ External tools (Sourcery, SwiftGen) ที่ต้อง configure แยกต่างหาก
2. Generated code ไม่ได้ type-safe
3. Build pipeline ซับซ้อน
4. Xcode ไม่เข้าใจ generated code อย่างสมบูรณ์

**ประโยชน์ของ Swift Macros:**
- Type-safe code generation ใน compile time
- Integrated กับ compiler อย่างสมบูรณ์
- IDE support เต็มรูปแบบ (code completion, refactoring)
- Error messages ที่มีความหมาย
- Testable และ debuggable

### 1.2 Macro Roles Taxonomy

Swift Macros แบ่งออกเป็น 2 ประเภทหลัก:

**Freestanding Macros** (ใช้ `#` prefix):
```swift
// Expression Macros - ใช้ใน expression context
let url = #URL("https://www.apple.com")
let value = #stringify(1 + 2)

// Declaration Macros - สร้าง declarations
#generateCases(for: UserRole.self)
```

**Attached Macros** (ใช้ `@` prefix):
```swift
// Member Macros - เพิ่ม members ให้กับ type
@AutoEquatable
struct Point { ... }

// Accessor Macros - เพิ่ม property accessors
@Observable
class ViewModel { ... }

// Conformance Macros - เพิ่ม protocol conformances
@AutoCodable
struct Config { ... }
```

**Macro Roles ทั้งหมด:**

| Role | Syntax | ใช้สำหรับ |
|------|--------|-----------|
| `@freestanding(expression)` | `#macroName(...)` | Expressions ที่ return ค่า |
| `@freestanding(declaration)` | `#macroName(...)` | สร้าง declarations ใหม่ |
| `@attached(member)` | `@MacroName` บน type | เพิ่ม members ให้ type |
| `@attached(memberAttribute)` | `@MacroName` บน type | เพิ่ม attributes ให้ members |
| `@attached(accessor)` | `@MacroName` บน property | เพิ่ม get/set/willSet/didSet |
| `@attached(peer)` | `@MacroName` บน declaration | สร้าง peer declarations |
| `@attached(conformance)` | `@MacroName` บน type | เพิ่ม protocol conformances |
| `@attached(extension)` | `@MacroName` บน type | เพิ่ม extensions |

### 1.3 Macros vs Code Generation

**Code Generation (Sourcery):**
```yaml
# sourcery.yml - ต้อง configure แยก
sources:
  - Sources/
templates:
  - Templates/AutoEquatable.stencil
output: Generated/
```

```stencil
// AutoEquatable.stencil - template ที่แยกออกมา
{% for type in types.structs where type.implements.AutoEquatable %}
extension {{ type.name }}: Equatable {
    static func == (lhs: {{ type.name }}, rhs: {{ type.name }}) -> Bool {
        {% for variable in type.variables %}
        guard lhs.{{ variable.name }} == rhs.{{ variable.name }} else { return false }
        {% endfor %}
        return true
    }
}
{% endfor %}
```

**Swift Macros:**
```swift
// ทุกอย่างอยู่ใน Swift โดยตรง
@attached(conformance)
@attached(member, names: named(==))
public macro AutoEquatable() = #externalMacro(
    module: "MyMacroImpl",
    type: "AutoEquatableMacro"
)

// การใช้งาน
@AutoEquatable
struct Point {
    var x: Double
    var y: Double
}
// Compiler จะ expand เป็น:
// extension Point: Equatable {
//     static func == (lhs: Point, rhs: Point) -> Bool {
//         return lhs.x == rhs.x && lhs.y == rhs.y
//     }
// }
```

### 1.4 SwiftSyntax Under the Hood

SwiftSyntax คือ library ที่ Apple พัฒนาขึ้นเพื่อ parse และ manipulate Swift source code ในรูปแบบ AST (Abstract Syntax Tree)

**Swift Source Code → AST:**
```swift
// Source code
let x = 1 + 2

// AST representation
VariableDeclSyntax(
    bindingSpecifier: .keyword(.let),
    bindings: [
        PatternBindingSyntax(
            pattern: IdentifierPatternSyntax(identifier: "x"),
            initializer: InitializerClauseSyntax(
                value: InfixOperatorExprSyntax(
                    leftOperand: IntegerLiteralExprSyntax(literal: "1"),
                    operator: BinaryOperatorExprSyntax(operator: "+"),
                    rightOperand: IntegerLiteralExprSyntax(literal: "2")
                )
            )
        )
    ]
)
```

**SwiftSyntax Key Types:**
```swift
import SwiftSyntax

// SyntaxProtocol - base protocol สำหรับทุก syntax node
protocol SyntaxProtocol { ... }

// Common syntax types
StructDeclSyntax       // struct declaration
ClassDeclSyntax        // class declaration
FunctionDeclSyntax     // function declaration
VariableDeclSyntax     // variable declaration
ExprSyntax             // expressions
StmtSyntax             // statements
TypeSyntax             // types
```

---

## 2. Setting Up Macro Development

### 2.1 Package.swift for Macro Targets

```swift
// Package.swift
// swift-tools-version: 5.9
import PackageDescription
import CompilerPluginSupport

let package = Package(
    name: "MyMacros",
    platforms: [
        .macOS(.v13),
        .iOS(.v16),
        .watchOS(.v9),
        .tvOS(.v16),
    ],
    products: [
        // Public API library
        .library(
            name: "MyMacros",
            targets: ["MyMacros"]
        ),
        // Executable สำหรับทดสอบ
        .executable(
            name: "MacroClient",
            targets: ["MacroClient"]
        ),
    ],
    dependencies: [
        // SwiftSyntax dependency
        .package(
            url: "https://github.com/apple/swift-syntax.git",
            from: "509.0.0"
        ),
    ],
    targets: [
        // Macro implementation target
        .macro(
            name: "MyMacrosImpl",
            dependencies: [
                .product(name: "SwiftSyntaxMacros", package: "swift-syntax"),
                .product(name: "SwiftCompilerPlugin", package: "swift-syntax"),
            ]
        ),
        // Public-facing library
        .target(
            name: "MyMacros",
            dependencies: ["MyMacrosImpl"]
        ),
        // Test target
        .testTarget(
            name: "MyMacrosTests",
            dependencies: [
                "MyMacrosImpl",
                .product(name: "SwiftSyntaxMacrosTestSupport", package: "swift-syntax"),
            ]
        ),
        // Example executable
        .executableTarget(
            name: "MacroClient",
            dependencies: ["MyMacros"]
        ),
    ]
)
```

### 2.2 SwiftSyntaxMacros Dependencies

```swift
// Package.swift dependencies สำหรับ advanced macro development
dependencies: [
    .package(
        url: "https://github.com/apple/swift-syntax.git",
        from: "509.0.0"
    ),
],

// Targets ต้องการ dependencies เหล่านี้:
// สำหรับ Macro Implementation:
.product(name: "SwiftSyntaxMacros", package: "swift-syntax"),
.product(name: "SwiftCompilerPlugin", package: "swift-syntax"),

// สำหรับ Testing:
.product(name: "SwiftSyntaxMacrosTestSupport", package: "swift-syntax"),

// Optional - สำหรับ advanced parsing:
.product(name: "SwiftSyntax", package: "swift-syntax"),
.product(name: "SwiftParser", package: "swift-syntax"),
.product(name: "SwiftDiagnostics", package: "swift-syntax"),
```

### 2.3 Macro Target Structure

```
MyMacros/
├── Package.swift
├── Sources/
│   ├── MyMacros/                    # Public API
│   │   └── MyMacros.swift           # Macro declarations
│   ├── MyMacrosImpl/                # Implementation
│   │   ├── MyMacrosPlugin.swift     # Compiler plugin entry point
│   │   ├── StringifyMacro.swift     # Individual macro implementations
│   │   ├── URLMacro.swift
│   │   └── AutoEquatableMacro.swift
│   └── MacroClient/                 # Example usage
│       └── main.swift
└── Tests/
    └── MyMacrosTests/
        ├── StringifyMacroTests.swift
        └── AutoEquatableMacroTests.swift
```

**MyMacros.swift (Public API):**
```swift
// Sources/MyMacros/MyMacros.swift

/// Macro ที่แปลง expression เป็น string พร้อม value
@freestanding(expression)
public macro stringify<T>(_ value: T) -> (T, String) =
    #externalMacro(module: "MyMacrosImpl", type: "StringifyMacro")

/// Macro ที่สร้าง URL อย่างปลอดภัย
@freestanding(expression)
public macro URL(_ stringLiteral: String) -> URL =
    #externalMacro(module: "MyMacrosImpl", type: "URLMacro")

/// Macro ที่สร้าง Equatable conformance อัตโนมัติ
@attached(conformance)
@attached(member, names: named(==))
public macro AutoEquatable() =
    #externalMacro(module: "MyMacrosImpl", type: "AutoEquatableMacro")
```

**MyMacrosPlugin.swift (Compiler Plugin Entry Point):**
```swift
// Sources/MyMacrosImpl/MyMacrosPlugin.swift
import SwiftCompilerPlugin
import SwiftSyntaxMacros

@main
struct MyMacrosPlugin: CompilerPlugin {
    let providingMacros: [Macro.Type] = [
        StringifyMacro.self,
        URLMacro.self,
        AutoEquatableMacro.self,
    ]
}
```

### 2.4 Testing Macros with SwiftSyntaxMacrosTestSupport

```swift
// Tests/MyMacrosTests/StringifyMacroTests.swift
import XCTest
import SwiftSyntaxMacros
import SwiftSyntaxMacrosTestSupport
import MyMacrosImpl

final class StringifyMacroTests: XCTestCase {
    
    // testMacros dictionary - map ชื่อ macro กับ implementation
    let testMacros: [String: Macro.Type] = [
        "stringify": StringifyMacro.self,
    ]
    
    func testStringifyWithInteger() {
        assertMacroExpansion(
            """
            let result = #stringify(1 + 2)
            """,
            expandedSource: """
            let result = (1 + 2, "1 + 2")
            """,
            macros: testMacros
        )
    }
    
    func testStringifyWithString() {
        assertMacroExpansion(
            """
            let result = #stringify("hello")
            """,
            expandedSource: """
            let result = ("hello", "\\"hello\\"")
            """,
            macros: testMacros
        )
    }
    
    func testStringifyWithExpression() {
        assertMacroExpansion(
            """
            let x = 5
            let result = #stringify(x * 2)
            """,
            expandedSource: """
            let x = 5
            let result = (x * 2, "x * 2")
            """,
            macros: testMacros
        )
    }
}
```

---

## 3. Expression Macros

### 3.1 ExpressionMacro Protocol

```swift
// Protocol definition จาก SwiftSyntaxMacros
public protocol ExpressionMacro: FreestandingMacro {
    static func expansion(
        of node: some FreestandingMacroExpansionSyntax,
        in context: some MacroExpansionContext
    ) throws -> ExprSyntax
}
```

**Key Types ที่ต้องรู้:**
- `FreestandingMacroExpansionSyntax`: node ที่แทน macro call เช่น `#stringify(1 + 2)`
- `MacroExpansionContext`: context สำหรับ expansion (สร้าง unique names, ส่ง diagnostics)
- `ExprSyntax`: ผลลัพธ์ที่เป็น expression syntax

### 3.2 #stringify Example (Official)

```swift
// Sources/MyMacrosImpl/StringifyMacro.swift
import SwiftSyntax
import SwiftSyntaxMacros

/// Implementation ของ #stringify macro
public struct StringifyMacro: ExpressionMacro {
    
    public static func expansion(
        of node: some FreestandingMacroExpansionSyntax,
        in context: some MacroExpansionContext
    ) throws -> ExprSyntax {
        
        // ตรวจสอบว่ามี argument อย่างน้อย 1 ตัว
        guard let argument = node.arguments.first?.expression else {
            throw MacroExpansionErrorMessage("#stringify requires an argument")
        }
        
        // สร้าง string representation ของ argument
        let sourceString = argument.description
            .trimmingCharacters(in: .whitespaces)
        
        // Return tuple expression: (value, "source")
        return "(\(argument), \(literal: sourceString))"
    }
}
```

**การใช้งาน:**
```swift
import MyMacros

// ตัวอย่างการใช้ #stringify
let (value, sourceCode) = #stringify(1 + 2)
print(value)      // 3
print(sourceCode) // "1 + 2"

// ใช้ใน debugging
func debugValue<T>(_ expr: T, _ source: String) {
    print("[\(source)] = \(expr)")
}

let x = 42
let y = 10
debugValue(#stringify(x + y))
// ผลลัพธ์: [x + y] = 52
```

### 3.3 #URL Macro (Safe URL Creation)

```swift
// Sources/MyMacrosImpl/URLMacro.swift
import SwiftSyntax
import SwiftSyntaxMacros
import Foundation

/// Macro ที่ validate URL ใน compile time
public struct URLMacro: ExpressionMacro {
    
    public static func expansion(
        of node: some FreestandingMacroExpansionSyntax,
        in context: some MacroExpansionContext
    ) throws -> ExprSyntax {
        
        // ดึง string literal argument
        guard let argument = node.arguments.first?.expression,
              let stringLiteral = argument.as(StringLiteralExprSyntax.self),
              stringLiteral.segments.count == 1,
              let segment = stringLiteral.segments.first?.as(StringSegmentSyntax.self) else {
            throw MacroExpansionErrorMessage(
                "#URL requires a static string literal"
            )
        }
        
        // ดึงค่า string
        let urlString = segment.content.text
        
        // Validate URL ใน compile time
        guard URL(string: urlString) != nil else {
            throw MacroExpansionErrorMessage(
                "Invalid URL: '\(urlString)'"
            )
        }
        
        // Return URL creation expression
        return "URL(string: \(argument))!"
    }
}
```

**Declaration ใน Public API:**
```swift
// Sources/MyMacros/MyMacros.swift
import Foundation

/// สร้าง URL จาก string literal และ validate ใน compile time
/// - Parameter stringLiteral: URL string ที่ต้องการสร้าง
/// - Returns: URL object
/// - Note: จะเกิด compile error ถ้า URL ไม่ valid
@freestanding(expression)
public macro URL(_ stringLiteral: String) -> Foundation.URL =
    #externalMacro(module: "MyMacrosImpl", type: "URLMacro")
```

**การใช้งาน:**
```swift
import MyMacros

// ✅ Valid URL - compile ผ่าน
let apiURL = #URL("https://api.example.com/v1/users")

// ✅ Valid URL with path
let imageURL = #URL("https://cdn.example.com/images/avatar.png")

// ❌ Invalid URL - compile error!
// let badURL = #URL("not a valid url!!!")
// Error: Invalid URL: 'not a valid url!!!'

// เปรียบเทียบกับวิธีเดิม
let oldWay = URL(string: "https://api.example.com/v1/users")!
// ถ้า URL ผิด จะ crash ตอน runtime!
```

**Test สำหรับ #URL:**
```swift
import XCTest
import SwiftSyntaxMacros
import SwiftSyntaxMacrosTestSupport
import MyMacrosImpl

final class URLMacroTests: XCTestCase {
    
    let testMacros: [String: Macro.Type] = [
        "URL": URLMacro.self,
    ]
    
    func testValidURL() {
        assertMacroExpansion(
            """
            let url = #URL("https://www.apple.com")
            """,
            expandedSource: """
            let url = URL(string: "https://www.apple.com")!
            """,
            macros: testMacros
        )
    }
    
    func testInvalidURL() {
        assertMacroExpansion(
            """
            let url = #URL("not valid")
            """,
            expandedSource: """
            let url = #URL("not valid")
            """,
            diagnostics: [
                DiagnosticSpec(
                    message: "Invalid URL: 'not valid'",
                    line: 1,
                    column: 11
                )
            ],
            macros: testMacros
        )
    }
    
    func testNonLiteralURL() {
        assertMacroExpansion(
            """
            let str = "https://example.com"
            let url = #URL(str)
            """,
            expandedSource: """
            let str = "https://example.com"
            let url = #URL(str)
            """,
            diagnostics: [
                DiagnosticSpec(
                    message: "#URL requires a static string literal",
                    line: 2,
                    column: 11
                )
            ],
            macros: testMacros
        )
    }
}
```

### 3.4 #assert with Message Macro

```swift
// Sources/MyMacrosImpl/AssertMacro.swift
import SwiftSyntax
import SwiftSyntaxMacros

/// Macro ที่เพิ่ม message อัตโนมัติเมื่อ assertion ล้มเหลว
public struct AssertMacro: ExpressionMacro {
    
    public static func expansion(
        of node: some FreestandingMacroExpansionSyntax,
        in context: some MacroExpansionContext
    ) throws -> ExprSyntax {
        
        guard let condition = node.arguments.first?.expression else {
            throw MacroExpansionErrorMessage("#assert requires a condition")
        }
        
        // สร้าง message จาก source code ของ condition
        let conditionSource = condition.description
            .trimmingCharacters(in: .whitespaces)
        
        let message = "Assertion failed: \(conditionSource)"
        
        // Return Swift assert call พร้อม message
        return """
        Swift.assert(\(condition), \(literal: message))
        """
    }
}
```

**Declaration:**
```swift
/// Enhanced assert ที่แสดง source code ของ condition
@freestanding(expression)
public macro assert(_ condition: Bool) -> Void =
    #externalMacro(module: "MyMacrosImpl", type: "AssertMacro")
```

**การใช้งาน:**
```swift
import MyMacros

let x = 5
let y = 3

// ✅ ผ่าน
#assert(x > y)

// ❌ ล้มเหลวพร้อม message ที่ชัดเจน
#assert(x < y)
// ผลลัพธ์: Assertion failed: x < y
// เทียบกับ standard assert ที่แสดงแค่ "Assertion failed"
```

---

## 4. Freestanding Declaration Macros

### 4.1 DeclarationMacro Protocol

```swift
// Protocol definition
public protocol DeclarationMacro: FreestandingMacro {
    static func expansion(
        of node: some FreestandingMacroExpansionSyntax,
        in context: some MacroExpansionContext
    ) throws -> [DeclSyntax]
    // Return array ของ declarations แทน single expression
}
```

### 4.2 Generating Enum Cases

```swift
// Sources/MyMacrosImpl/GenerateCasesMacro.swift
import SwiftSyntax
import SwiftSyntaxMacros

/// Macro ที่ generate enum cases จาก array of strings
public struct GenerateCasesMacro: DeclarationMacro {
    
    public static func expansion(
        of node: some FreestandingMacroExpansionSyntax,
        in context: some MacroExpansionContext
    ) throws -> [DeclSyntax] {
        
        // ดึง string arguments
        let cases = node.arguments.compactMap { arg -> String? in
            guard let strLiteral = arg.expression.as(StringLiteralExprSyntax.self),
                  let segment = strLiteral.segments.first?.as(StringSegmentSyntax.self) else {
                return nil
            }
            return segment.content.text
        }
        
        guard !cases.isEmpty else {
            throw MacroExpansionErrorMessage(
                "#generateCases requires at least one string argument"
            )
        }
        
        // สร้าง enum case declarations
        return cases.map { caseName in
            "case \(raw: caseName)"
        }
    }
}
```

**Declaration:**
```swift
/// Generate enum cases จาก string literals
@freestanding(declaration, names: arbitrary)
public macro generateCases(_ cases: String...) =
    #externalMacro(module: "MyMacrosImpl", type: "GenerateCasesMacro")
```

**การใช้งาน:**
```swift
import MyMacros

enum Permission {
    #generateCases("read", "write", "execute", "admin")
    // Expands to:
    // case read
    // case write
    // case execute
    // case admin
}

// ตัวอย่างที่ซับซ้อนขึ้น
enum HTTPMethod {
    #generateCases("get", "post", "put", "delete", "patch", "head", "options")
    
    var uppercased: String {
        return rawValue.uppercased()
    }
}
```

### 4.3 Generating Static Factory Methods

```swift
// Sources/MyMacrosImpl/FactoryMethodMacro.swift
import SwiftSyntax
import SwiftSyntaxBuilder
import SwiftSyntaxMacros

/// Macro ที่ generate static factory methods
public struct FactoryMethodMacro: DeclarationMacro {
    
    public static func expansion(
        of node: some FreestandingMacroExpansionSyntax,
        in context: some MacroExpansionContext
    ) throws -> [DeclSyntax] {
        
        // Parse arguments: type name และ property names
        guard let typeArg = node.arguments.first?.expression,
              let typeName = typeArg.as(DeclReferenceExprSyntax.self)?.baseName.text else {
            throw MacroExpansionErrorMessage("#factoryMethod requires a type name")
        }
        
        // สร้าง factory method
        let factoryMethod: DeclSyntax = """
        static func make() -> \(raw: typeName) {
            return \(raw: typeName)()
        }
        """
        
        return [factoryMethod]
    }
}
```

---

## 5. Attached Macros

### 5.1 @attached(member) — Adding Members

```swift
// Declaration
@attached(member, names: named(description), named(debugDescription))
public macro CustomStringConvertible() =
    #externalMacro(module: "MyMacrosImpl", type: "CustomStringConvertibleMacro")
```

```swift
// Implementation
import SwiftSyntax
import SwiftSyntaxBuilder
import SwiftSyntaxMacros

public struct CustomStringConvertibleMacro: MemberMacro {
    
    public static func expansion(
        of node: AttributeSyntax,
        providingMembersOf declaration: some DeclGroupSyntax,
        in context: some MacroExpansionContext
    ) throws -> [DeclSyntax] {
        
        // ดึง type name
        guard let structDecl = declaration.as(StructDeclSyntax.self) else {
            throw MacroExpansionErrorMessage(
                "@CustomStringConvertible can only be applied to structs"
            )
        }
        
        let typeName = structDecl.name.text
        
        // ดึง property names
        let properties = structDecl.memberBlock.members
            .compactMap { member -> String? in
                guard let varDecl = member.decl.as(VariableDeclSyntax.self),
                      let binding = varDecl.bindings.first,
                      let identifier = binding.pattern.as(IdentifierPatternSyntax.self) else {
                    return nil
                }
                return identifier.identifier.text
            }
        
        // สร้าง description string
        let propertiesString = properties
            .map { prop in "\(prop): \\(\(prop))" }
            .joined(separator: ", ")
        
        // Return description computed property
        let description: DeclSyntax = """
        var description: String {
            return "\(raw: typeName)(\(raw: propertiesString))"
        }
        """
        
        return [description]
    }
}
```

**การใช้งาน:**
```swift
import MyMacros

@CustomStringConvertible
struct Point {
    var x: Double
    var y: Double
    var z: Double
}

// Expands to:
// struct Point {
//     var x: Double
//     var y: Double
//     var z: Double
//
//     var description: String {
//         return "Point(x: \(x), y: \(y), z: \(z))"
//     }
// }

let point = Point(x: 1.0, y: 2.0, z: 3.0)
print(point.description) // "Point(x: 1.0, y: 2.0, z: 3.0)"
```

### 5.2 @attached(memberAttribute) — Adding Attributes to Members

```swift
// Declaration
@attached(memberAttribute)
public macro NonEscaping() =
    #externalMacro(module: "MyMacrosImpl", type: "NonEscapingMacro")
```

```swift
// Implementation
public struct NonEscapingMacro: MemberAttributeMacro {
    
    public static func expansion(
        of node: AttributeSyntax,
        attachedTo declaration: some DeclGroupSyntax,
        providingAttributesFor member: some DeclSyntaxProtocol,
        in context: some MacroExpansionContext
    ) throws -> [AttributeSyntax] {
        
        // เพิ่ม @MainActor ให้ทุก method
        guard member.is(FunctionDeclSyntax.self) else {
            return []
        }
        
        return ["@MainActor"]
    }
}
```

**การใช้งาน:**
```swift
@NonEscaping
class UIComponent {
    func update() {
        // ...
    }
    func refresh() {
        // ...
    }
}

// Expands to:
// class UIComponent {
//     @MainActor func update() { ... }
//     @MainActor func refresh() { ... }
// }
```

### 5.3 @attached(accessor) — Adding Property Accessors

```swift
// Declaration - สำหรับ lazy loading pattern
@attached(accessor, names: named(get), named(set))
public macro Persisted(key: String) =
    #externalMacro(module: "MyMacrosImpl", type: "PersistedMacro")
```

```swift
// Implementation
import SwiftSyntax
import SwiftSyntaxMacros

public struct PersistedMacro: AccessorMacro {
    
    public static func expansion(
        of node: AttributeSyntax,
        providingAccessorsOf declaration: some DeclSyntaxProtocol,
        in context: some MacroExpansionContext
    ) throws -> [AccessorDeclSyntax] {
        
        // ดึง key argument
        guard let keyArg = node.arguments?.as(LabeledExprListSyntax.self)?.first?.expression,
              let keyLiteral = keyArg.as(StringLiteralExprSyntax.self),
              let keySegment = keyLiteral.segments.first?.as(StringSegmentSyntax.self) else {
            throw MacroExpansionErrorMessage("@Persisted requires a key: String argument")
        }
        
        let key = keySegment.content.text
        
        // ดึง property type
        guard let varDecl = declaration.as(VariableDeclSyntax.self),
              let binding = varDecl.bindings.first,
              let typeAnnotation = binding.typeAnnotation else {
            throw MacroExpansionErrorMessage("@Persisted requires explicit type annotation")
        }
        
        let typeName = typeAnnotation.type.description.trimmingCharacters(in: .whitespaces)
        
        // สร้าง getter
        let getter: AccessorDeclSyntax = """
        get {
            return UserDefaults.standard.object(forKey: \(literal: key)) as? \(raw: typeName)
        }
        """
        
        // สร้าง setter
        let setter: AccessorDeclSyntax = """
        set {
            UserDefaults.standard.set(newValue, forKey: \(literal: key))
        }
        """
        
        return [getter, setter]
    }
}
```

**การใช้งาน:**
```swift
struct UserPreferences {
    @Persisted(key: "theme")
    var theme: String?
    
    @Persisted(key: "fontSize")
    var fontSize: Int?
    
    @Persisted(key: "isDarkMode")
    var isDarkMode: Bool?
}

// Expands to:
// struct UserPreferences {
//     var theme: String? {
//         get { UserDefaults.standard.object(forKey: "theme") as? String }
//         set { UserDefaults.standard.set(newValue, forKey: "theme") }
//     }
//     // ...
// }

var prefs = UserPreferences()
prefs.theme = "dark"
print(prefs.theme) // "dark"
```

### 5.4 @attached(peer) — Adding Peer Declarations

```swift
// Declaration
@attached(peer, names: prefixed(Async))
public macro AddAsyncOverload() =
    #externalMacro(module: "MyMacrosImpl", type: "AddAsyncOverloadMacro")
```

```swift
// Implementation
import SwiftSyntax
import SwiftSyntaxBuilder
import SwiftSyntaxMacros

public struct AddAsyncOverloadMacro: PeerMacro {
    
    public static func expansion(
        of node: AttributeSyntax,
        providingPeersOf declaration: some DeclSyntaxProtocol,
        in context: some MacroExpansionContext
    ) throws -> [DeclSyntax] {
        
        // ดึง function declaration
        guard let funcDecl = declaration.as(FunctionDeclSyntax.self) else {
            throw MacroExpansionErrorMessage(
                "@AddAsyncOverload can only be applied to functions"
            )
        }
        
        // ตรวจสอบว่ามี completion handler parameter
        let params = funcDecl.signature.parameterClause.parameters
        guard let lastParam = params.last,
              lastParam.firstName.text == "completion" else {
            throw MacroExpansionErrorMessage(
                "@AddAsyncOverload requires a 'completion' parameter"
            )
        }
        
        let funcName = funcDecl.name.text
        
        // สร้าง async version
        let asyncFunc: DeclSyntax = """
        func \(raw: funcName)Async() async {
            await withCheckedContinuation { continuation in
                \(raw: funcName) { 
                    continuation.resume()
                }
            }
        }
        """
        
        return [asyncFunc]
    }
}
```

**การใช้งาน:**
```swift
class NetworkManager {
    @AddAsyncOverload
    func fetchData(completion: @escaping () -> Void) {
        // callback-based implementation
        DispatchQueue.global().asyncAfter(deadline: .now() + 1) {
            completion()
        }
    }
    
    // Macro generates:
    // func fetchDataAsync() async {
    //     await withCheckedContinuation { continuation in
    //         fetchData { continuation.resume() }
    //     }
    // }
}

// ใช้งาน async version
Task {
    let manager = NetworkManager()
    await manager.fetchDataAsync()
    print("Done!")
}
```

### 5.5 @attached(conformance) — Adding Protocol Conformances

```swift
// Declaration
@attached(conformance)
@attached(member, names: named(==))
public macro AutoEquatable() =
    #externalMacro(module: "MyMacrosImpl", type: "AutoEquatableMacro")
```

```swift
// Implementation
import SwiftSyntax
import SwiftSyntaxBuilder
import SwiftSyntaxMacros

public struct AutoEquatableMacro: ConformanceMacro, MemberMacro {
    
    // ConformanceMacro - บอกว่า conform กับ protocol อะไร
    public static func expansion(
        of node: AttributeSyntax,
        providingConformancesOf declaration: some DeclGroupSyntax,
        in context: some MacroExpansionContext
    ) throws -> [(TypeSyntax, GenericWhereClauseSyntax?)] {
        return [("Equatable", nil)]
    }
    
    // MemberMacro - สร้าง == operator
    public static func expansion(
        of node: AttributeSyntax,
        providingMembersOf declaration: some DeclGroupSyntax,
        in context: some MacroExpansionContext
    ) throws -> [DeclSyntax] {
        
        guard let structDecl = declaration.as(StructDeclSyntax.self) else {
            throw MacroExpansionErrorMessage(
                "@AutoEquatable can only be applied to structs"
            )
        }
        
        let typeName = structDecl.name.text
        
        // ดึง stored properties ทั้งหมด
        let properties = getStoredProperties(from: structDecl)
        
        if properties.isEmpty {
            // ถ้าไม่มี properties ให้ return true เสมอ
            let equatableImpl: DeclSyntax = """
            static func == (lhs: \(raw: typeName), rhs: \(raw: typeName)) -> Bool {
                return true
            }
            """
            return [equatableImpl]
        }
        
        // สร้าง comparison expressions
        let comparisons = properties
            .map { prop in "lhs.\(prop) == rhs.\(prop)" }
            .joined(separator: " &&\n            ")
        
        let equatableImpl: DeclSyntax = """
        static func == (lhs: \(raw: typeName), rhs: \(raw: typeName)) -> Bool {
            return \(raw: comparisons)
        }
        """
        
        return [equatableImpl]
    }
    
    // Helper function
    private static func getStoredProperties(from decl: StructDeclSyntax) -> [String] {
        return decl.memberBlock.members
            .compactMap { member -> String? in
                guard let varDecl = member.decl.as(VariableDeclSyntax.self),
                      varDecl.bindingSpecifier.tokenKind != .keyword(.let) ||
                      varDecl.bindings.first?.accessorBlock == nil,
                      let binding = varDecl.bindings.first,
                      let identifier = binding.pattern.as(IdentifierPatternSyntax.self),
                      binding.accessorBlock == nil else {
                    return nil
                }
                return identifier.identifier.text
            }
    }
}
```

### 5.6 @attached(extension) — Adding Extensions

```swift
// Declaration
@attached(extension, conformances: CustomStringConvertible, names: named(description))
public macro Describable() =
    #externalMacro(module: "MyMacrosImpl", type: "DescribableMacro")
```

```swift
// Implementation
public struct DescribableMacro: ExtensionMacro {
    
    public static func expansion(
        of node: AttributeSyntax,
        attachedTo declaration: some DeclGroupSyntax,
        providingExtensionsOf type: some TypeSyntaxProtocol,
        conformingTo protocols: [TypeSyntax],
        in context: some MacroExpansionContext
    ) throws -> [ExtensionDeclSyntax] {
        
        guard let structDecl = declaration.as(StructDeclSyntax.self) else {
            return []
        }
        
        let typeName = structDecl.name.text
        
        let properties = structDecl.memberBlock.members
            .compactMap { member -> String? in
                guard let varDecl = member.decl.as(VariableDeclSyntax.self),
                      let binding = varDecl.bindings.first,
                      let identifier = binding.pattern.as(IdentifierPatternSyntax.self) else {
                    return nil
                }
                return identifier.identifier.text
            }
        
        let propsDesc = properties
            .map { "\($0): \\(\($0))" }
            .joined(separator: ", ")
        
        let extensionDecl = try ExtensionDeclSyntax("extension \(raw: typeName): CustomStringConvertible") {
            """
            var description: String {
                return "\(raw: typeName)(\(raw: propsDesc))"
            }
            """
        }
        
        return [extensionDecl]
    }
}
```

---

## 6. Building @AutoEquatable Macro

### 6.1 Complete Implementation

```swift
// Sources/MyMacrosImpl/AutoEquatableMacro.swift
import SwiftSyntax
import SwiftSyntaxBuilder
import SwiftSyntaxMacros
import SwiftDiagnostics

/// Full implementation ของ @AutoEquatable
public struct AutoEquatableMacro: ConformanceMacro, MemberMacro {
    
    // MARK: - ConformanceMacro
    
    public static func expansion(
        of node: AttributeSyntax,
        providingConformancesOf declaration: some DeclGroupSyntax,
        in context: some MacroExpansionContext
    ) throws -> [(TypeSyntax, GenericWhereClauseSyntax?)] {
        
        // ตรวจสอบว่าเป็น struct
        try validateDeclaration(declaration)
        
        return [("Equatable", nil)]
    }
    
    // MARK: - MemberMacro
    
    public static func expansion(
        of node: AttributeSyntax,
        providingMembersOf declaration: some DeclGroupSyntax,
        in context: some MacroExpansionContext
    ) throws -> [DeclSyntax] {
        
        guard let structDecl = declaration.as(StructDeclSyntax.self) else {
            return []
        }
        
        let typeName = structDecl.name.text
        
        // ดึง properties ทั้งหมดรวมถึง stored properties
        let storedProperties = extractStoredProperties(from: structDecl)
        
        // สร้าง == operator
        let equatableImpl = try generateEquatableImpl(
            typeName: typeName,
            properties: storedProperties
        )
        
        return [equatableImpl]
    }
    
    // MARK: - Helpers
    
    private static func validateDeclaration(_ declaration: some DeclGroupSyntax) throws {
        guard declaration.is(StructDeclSyntax.self) else {
            throw MacroExpansionErrorMessage(
                "@AutoEquatable can only be applied to structs. " +
                "For classes, implement Equatable manually."
            )
        }
    }
    
    private static func extractStoredProperties(
        from structDecl: StructDeclSyntax
    ) -> [(name: String, type: String)] {
        
        return structDecl.memberBlock.members.compactMap { member -> (String, String)? in
            guard let varDecl = member.decl.as(VariableDeclSyntax.self) else {
                return nil
            }
            
            // Skip computed properties
            for binding in varDecl.bindings {
                if binding.accessorBlock != nil {
                    return nil
                }
            }
            
            // Skip static properties
            if varDecl.modifiers.contains(where: {
                $0.name.tokenKind == .keyword(.static)
            }) {
                return nil
            }
            
            guard let binding = varDecl.bindings.first,
                  let identifier = binding.pattern.as(IdentifierPatternSyntax.self),
                  let typeAnnotation = binding.typeAnnotation else {
                return nil
            }
            
            let name = identifier.identifier.text
            let typeName = typeAnnotation.type.description
                .trimmingCharacters(in: .whitespaces)
            
            return (name, typeName)
        }
    }
    
    private static func generateEquatableImpl(
        typeName: String,
        properties: [(name: String, type: String)]
    ) throws -> DeclSyntax {
        
        if properties.isEmpty {
            return """
            static func == (lhs: \(raw: typeName), rhs: \(raw: typeName)) -> Bool {
                return true
            }
            """
        }
        
        // สร้าง individual comparisons
        let comparisons = properties
            .map { prop in
                "lhs.\(prop.name) == rhs.\(prop.name)"
            }
            .joined(separator: " &&\n        ")
        
        return """
        static func == (lhs: \(raw: typeName), rhs: \(raw: typeName)) -> Bool {
            return \(raw: comparisons)
        }
        """
    }
}
```

### 6.2 การใช้งาน @AutoEquatable

```swift
import MyMacros

@AutoEquatable
struct Address {
    let street: String
    let city: String
    let country: String
    let postalCode: String
}

@AutoEquatable
struct Person {
    let name: String
    let age: Int
    let address: Address
    // computed property - จะถูก skip
    var displayName: String { name.uppercased() }
}

// ทดสอบ
let addr1 = Address(street: "123 Main St", city: "Bangkok", country: "TH", postalCode: "10110")
let addr2 = Address(street: "123 Main St", city: "Bangkok", country: "TH", postalCode: "10110")
print(addr1 == addr2) // true

let person1 = Person(name: "Alice", age: 30, address: addr1)
let person2 = Person(name: "Alice", age: 30, address: addr2)
let person3 = Person(name: "Bob", age: 25, address: addr1)

print(person1 == person2) // true
print(person1 == person3) // false
```

### 6.3 Handling Optional Properties

```swift
@AutoEquatable
struct Configuration {
    var host: String
    var port: Int
    var timeout: Double?    // Optional
    var apiKey: String?     // Optional
    var tags: [String]      // Array
    var metadata: [String: Any]  // Dictionary
}

// Generated:
// static func == (lhs: Configuration, rhs: Configuration) -> Bool {
//     return lhs.host == rhs.host &&
//            lhs.port == rhs.port &&
//            lhs.timeout == rhs.timeout &&
//            lhs.apiKey == rhs.apiKey &&
//            lhs.tags == rhs.tags
//            // metadata ข้ามเพราะ Any ไม่ Equatable
// }
```

---

## 7. Building @AutoHashable Macro

### 7.1 Complete Implementation

```swift
// Sources/MyMacrosImpl/AutoHashableMacro.swift
import SwiftSyntax
import SwiftSyntaxBuilder
import SwiftSyntaxMacros

public struct AutoHashableMacro: ConformanceMacro, MemberMacro {
    
    // MARK: - ConformanceMacro
    
    public static func expansion(
        of node: AttributeSyntax,
        providingConformancesOf declaration: some DeclGroupSyntax,
        in context: some MacroExpansionContext
    ) throws -> [(TypeSyntax, GenericWhereClauseSyntax?)] {
        return [("Hashable", nil)]
    }
    
    // MARK: - MemberMacro
    
    public static func expansion(
        of node: AttributeSyntax,
        providingMembersOf declaration: some DeclGroupSyntax,
        in context: some MacroExpansionContext
    ) throws -> [DeclSyntax] {
        
        guard let structDecl = declaration.as(StructDeclSyntax.self) else {
            throw MacroExpansionErrorMessage(
                "@AutoHashable can only be applied to structs"
            )
        }
        
        let properties = extractHashableProperties(from: structDecl)
        return [generateHashImpl(properties: properties)]
    }
    
    // MARK: - Helpers
    
    private static func extractHashableProperties(
        from structDecl: StructDeclSyntax
    ) -> [String] {
        return structDecl.memberBlock.members.compactMap { member -> String? in
            guard let varDecl = member.decl.as(VariableDeclSyntax.self) else {
                return nil
            }
            
            // Skip computed properties
            for binding in varDecl.bindings where binding.accessorBlock != nil {
                return nil
            }
            
            // Skip static properties
            guard !varDecl.modifiers.contains(where: {
                $0.name.tokenKind == .keyword(.static)
            }) else {
                return nil
            }
            
            guard let binding = varDecl.bindings.first,
                  let identifier = binding.pattern.as(IdentifierPatternSyntax.self) else {
                return nil
            }
            
            return identifier.identifier.text
        }
    }
    
    private static func generateHashImpl(properties: [String]) -> DeclSyntax {
        let combineStatements = properties
            .map { "hasher.combine(\($0))" }
            .joined(separator: "\n    ")
        
        return """
        func hash(into hasher: inout Hasher) {
            \(raw: combineStatements)
        }
        """
    }
}
```

### 7.2 การใช้งาน @AutoHashable

```swift
import MyMacros

@AutoEquatable
@AutoHashable
struct Color {
    let red: Int
    let green: Int
    let blue: Int
    let alpha: Double
}

// ใช้ใน Set
let colorSet: Set<Color> = [
    Color(red: 255, green: 0, blue: 0, alpha: 1.0),
    Color(red: 0, green: 255, blue: 0, alpha: 1.0),
    Color(red: 255, green: 0, blue: 0, alpha: 1.0), // Duplicate!
]
print(colorSet.count) // 2 (ไม่นับ duplicate)

// ใช้ใน Dictionary
var colorNames: [Color: String] = [:]
colorNames[Color(red: 255, green: 0, blue: 0, alpha: 1.0)] = "Red"
colorNames[Color(red: 0, green: 0, blue: 255, alpha: 1.0)] = "Blue"
```

---

## 8. Building @Observable-like Macro

### 8.1 willSet/didSet Addition via Accessor Macro

```swift
// Sources/MyMacros/Observable.swift

/// Macro ที่เพิ่ม observation support
@attached(accessor, names: named(get), named(set), named(_$observationRegistrar))
@attached(member, names: named(_$observationRegistrar), named(access), named(withMutation))
public macro Observable() =
    #externalMacro(module: "MyMacrosImpl", type: "ObservableMacro")
```

```swift
// Sources/MyMacrosImpl/ObservableMacro.swift
import SwiftSyntax
import SwiftSyntaxBuilder
import SwiftSyntaxMacros

public struct ObservableMacro: MemberMacro {
    
    public static func expansion(
        of node: AttributeSyntax,
        providingMembersOf declaration: some DeclGroupSyntax,
        in context: some MacroExpansionContext
    ) throws -> [DeclSyntax] {
        
        guard declaration.is(ClassDeclSyntax.self) else {
            throw MacroExpansionErrorMessage(
                "@Observable can only be applied to classes"
            )
        }
        
        // เพิ่ม _$observationRegistrar
        let registrar: DeclSyntax = """
        @ObservationIgnored
        private let _$observationRegistrar = ObservationRegistrar()
        """
        
        // เพิ่ม access method
        let access: DeclSyntax = """
        internal nonisolated func access<Member>(
            keyPath: KeyPath<Self, Member>
        ) {
            _$observationRegistrar.access(self, keyPath: keyPath)
        }
        """
        
        // เพิ่ม withMutation method
        let withMutation: DeclSyntax = """
        internal nonisolated func withMutation<Member, T>(
            keyPath: KeyPath<Self, Member>,
            _ mutation: () throws -> T
        ) rethrows -> T {
            try _$observationRegistrar.withMutation(of: self, keyPath: keyPath, mutation)
        }
        """
        
        return [registrar, access, withMutation]
    }
}

public struct ObservablePropertyMacro: AccessorMacro {
    
    public static func expansion(
        of node: AttributeSyntax,
        providingAccessorsOf declaration: some DeclSyntaxProtocol,
        in context: some MacroExpansionContext
    ) throws -> [AccessorDeclSyntax] {
        
        guard let varDecl = declaration.as(VariableDeclSyntax.self),
              let binding = varDecl.bindings.first,
              let identifier = binding.pattern.as(IdentifierPatternSyntax.self) else {
            return []
        }
        
        let propertyName = identifier.identifier.text
        
        let getter: AccessorDeclSyntax = """
        get {
            access(keyPath: \\.\(raw: propertyName))
            return _\(raw: propertyName)
        }
        """
        
        let setter: AccessorDeclSyntax = """
        set {
            withMutation(keyPath: \\.\(raw: propertyName)) {
                _\(raw: propertyName) = newValue
            }
        }
        """
        
        return [getter, setter]
    }
}
```

### 8.2 การใช้งาน @Observable

```swift
import Observation

// ใช้ Swift's built-in @Observable (iOS 17+)
@Observable
class TodoViewModel {
    var items: [TodoItem] = []
    var isLoading: Bool = false
    var errorMessage: String?
    
    func addItem(_ item: TodoItem) {
        items.append(item)
    }
    
    func loadItems() async {
        isLoading = true
        defer { isLoading = false }
        
        do {
            items = try await fetchItems()
        } catch {
            errorMessage = error.localizedDescription
        }
    }
}

// ใน SwiftUI View - auto-updates เมื่อ property เปลี่ยน
struct TodoListView: View {
    @State private var viewModel = TodoViewModel()
    
    var body: some View {
        List(viewModel.items) { item in
            Text(item.title)
        }
        .overlay {
            if viewModel.isLoading {
                ProgressView()
            }
        }
    }
}
```

---

## 9. Building @Codable Enhancement Macros

### 9.1 @CodingKey for Custom Key Names

```swift
// Declaration
@attached(peer, names: arbitrary)
public macro CodingKey(_ key: String) =
    #externalMacro(module: "MyMacrosImpl", type: "CodingKeyMacro")
```

```swift
// แต่วิธีที่ดีกว่าคือใช้ @attached(member) ใน struct level
@attached(member, names: named(CodingKeys), named(init(from:)), named(encode(to:)))
public macro CustomCodable() =
    #externalMacro(module: "MyMacrosImpl", type: "CustomCodableMacro")
```

```swift
// Implementation
import SwiftSyntax
import SwiftSyntaxBuilder
import SwiftSyntaxMacros

// Marker attribute (ใช้ parser เพื่อดึง key)
public struct CodingKeyAttributeMacro: PeerMacro {
    public static func expansion(
        of node: AttributeSyntax,
        providingPeersOf declaration: some DeclSyntaxProtocol,
        in context: some MacroExpansionContext
    ) throws -> [DeclSyntax] {
        // Marker only - ไม่สร้าง code
        return []
    }
}

public struct CustomCodableMacro: MemberMacro {
    
    public static func expansion(
        of node: AttributeSyntax,
        providingMembersOf declaration: some DeclGroupSyntax,
        in context: some MacroExpansionContext
    ) throws -> [DeclSyntax] {
        
        guard let structDecl = declaration.as(StructDeclSyntax.self) else {
            throw MacroExpansionErrorMessage(
                "@CustomCodable can only be applied to structs"
            )
        }
        
        // ดึง properties พร้อม custom coding keys
        let properties = try extractPropertiesWithCodingKeys(from: structDecl)
        
        // สร้าง CodingKeys enum
        let codingKeysEnum = generateCodingKeysEnum(properties: properties)
        
        // สร้าง init(from:) 
        let initDecoder = generateInitFromDecoder(properties: properties, typeName: structDecl.name.text)
        
        // สร้าง encode(to:)
        let encodeTo = generateEncodeTo(properties: properties)
        
        return [codingKeysEnum, initDecoder, encodeTo]
    }
    
    private static func extractPropertiesWithCodingKeys(
        from structDecl: StructDeclSyntax
    ) throws -> [(name: String, codingKey: String, type: String, defaultValue: String?, ignored: Bool)] {
        
        return structDecl.memberBlock.members.compactMap { member -> (String, String, String, String?, Bool)? in
            guard let varDecl = member.decl.as(VariableDeclSyntax.self),
                  let binding = varDecl.bindings.first,
                  let identifier = binding.pattern.as(IdentifierPatternSyntax.self),
                  let typeAnnotation = binding.typeAnnotation else {
                return nil
            }
            
            let propertyName = identifier.identifier.text
            let typeName = typeAnnotation.type.description.trimmingCharacters(in: .whitespaces)
            
            // Check for @IgnoreCoding
            let isIgnored = varDecl.attributes.contains { attr in
                guard case .attribute(let attrSyntax) = attr,
                      let name = attrSyntax.attributeName.as(IdentifierTypeSyntax.self) else {
                    return false
                }
                return name.name.text == "IgnoreCoding"
            }
            
            // Check for @CodingKey("customKey")
            var customKey: String? = nil
            for attr in varDecl.attributes {
                guard case .attribute(let attrSyntax) = attr,
                      let name = attrSyntax.attributeName.as(IdentifierTypeSyntax.self),
                      name.name.text == "CodingKey",
                      let args = attrSyntax.arguments?.as(LabeledExprListSyntax.self),
                      let firstArg = args.first?.expression,
                      let strLiteral = firstArg.as(StringLiteralExprSyntax.self),
                      let segment = strLiteral.segments.first?.as(StringSegmentSyntax.self) else {
                    continue
                }
                customKey = segment.content.text
            }
            
            // Check for @DefaultValue(...)
            var defaultValue: String? = nil
            for attr in varDecl.attributes {
                guard case .attribute(let attrSyntax) = attr,
                      let name = attrSyntax.attributeName.as(IdentifierTypeSyntax.self),
                      name.name.text == "DefaultValue",
                      let args = attrSyntax.arguments?.as(LabeledExprListSyntax.self),
                      let firstArg = args.first?.expression else {
                    continue
                }
                defaultValue = firstArg.description.trimmingCharacters(in: .whitespaces)
            }
            
            let codingKey = customKey ?? propertyName.toSnakeCase()
            
            return (propertyName, codingKey, typeName, defaultValue, isIgnored)
        }
    }
    
    private static func generateCodingKeysEnum(
        properties: [(name: String, codingKey: String, type: String, defaultValue: String?, ignored: Bool)]
    ) -> DeclSyntax {
        
        let nonIgnoredProperties = properties.filter { !$0.ignored }
        let cases = nonIgnoredProperties
            .map { prop in
                if prop.name == prop.codingKey {
                    return "case \(prop.name)"
                } else {
                    return "case \(prop.name) = \"\(prop.codingKey)\""
                }
            }
            .joined(separator: "\n    ")
        
        return """
        enum CodingKeys: String, CodingKey {
            \(raw: cases)
        }
        """
    }
    
    private static func generateInitFromDecoder(
        properties: [(name: String, codingKey: String, type: String, defaultValue: String?, ignored: Bool)],
        typeName: String
    ) -> DeclSyntax {
        
        let assignments = properties.map { prop -> String in
            if prop.ignored {
                if let defaultValue = prop.defaultValue {
                    return "self.\(prop.name) = \(defaultValue)"
                } else {
                    return "// \(prop.name) is ignored from coding"
                }
            }
            
            let typeName = prop.type
            let isOptional = typeName.hasSuffix("?")
            
            if isOptional {
                if let defaultValue = prop.defaultValue {
                    return "self.\(prop.name) = try container.decodeIfPresent(\(typeName.dropLast()).self, forKey: .\(prop.name)) ?? \(defaultValue)"
                } else {
                    return "self.\(prop.name) = try container.decodeIfPresent(\(typeName.dropLast()).self, forKey: .\(prop.name))"
                }
            } else {
                if let defaultValue = prop.defaultValue {
                    return "self.\(prop.name) = try container.decodeIfPresent(\(typeName).self, forKey: .\(prop.name)) ?? \(defaultValue)"
                } else {
                    return "self.\(prop.name) = try container.decode(\(typeName).self, forKey: .\(prop.name))"
                }
            }
        }.joined(separator: "\n    ")
        
        return """
        init(from decoder: Decoder) throws {
            let container = try decoder.container(keyedBy: CodingKeys.self)
            \(raw: assignments)
        }
        """
    }
    
    private static func generateEncodeTo(
        properties: [(name: String, codingKey: String, type: String, defaultValue: String?, ignored: Bool)]
    ) -> DeclSyntax {
        
        let encodings = properties
            .filter { !$0.ignored }
            .map { prop -> String in
                let typeName = prop.type
                if typeName.hasSuffix("?") {
                    return "try container.encodeIfPresent(\(prop.name), forKey: .\(prop.name))"
                } else {
                    return "try container.encode(\(prop.name), forKey: .\(prop.name))"
                }
            }
            .joined(separator: "\n    ")
        
        return """
        func encode(to encoder: Encoder) throws {
            var container = encoder.container(keyedBy: CodingKeys.self)
            \(raw: encodings)
        }
        """
    }
}

// Extension helper
extension String {
    func toSnakeCase() -> String {
        var result = ""
        for (index, char) in self.enumerated() {
            if char.isUppercase && index > 0 {
                result += "_"
            }
            result += char.lowercased()
        }
        return result
    }
}
```

### 9.2 การใช้งาน @CustomCodable

```swift
import MyMacros
import Foundation

@CustomCodable
struct UserProfile: Codable {
    @CodingKey("user_name")
    var username: String
    
    @CodingKey("email_address")
    var email: String
    
    @DefaultValue(0)
    var age: Int
    
    @DefaultValue("Unknown")
    var country: String
    
    @IgnoreCoding
    var sessionToken: String = ""
    
    var bio: String?
}

// Expands to CodingKeys enum และ init/encode implementations

// JSON ที่รับ
let json = """
{
    "user_name": "john_doe",
    "email_address": "john@example.com",
    "bio": "Swift developer"
}
"""

let decoder = JSONDecoder()
let user = try! decoder.decode(UserProfile.self, from: json.data(using: .utf8)!)
print(user.username)  // "john_doe"
print(user.age)       // 0 (default value)
print(user.country)   // "Unknown" (default value)
print(user.bio)       // "Swift developer"
```

---

## 10. Building @Builder Macro

### 10.1 Fluent Builder Pattern Generation

```swift
// Declaration
@attached(member, names: arbitrary)
public macro Builder() =
    #externalMacro(module: "MyMacrosImpl", type: "BuilderMacro")
```

```swift
// Implementation
import SwiftSyntax
import SwiftSyntaxBuilder
import SwiftSyntaxMacros

public struct BuilderMacro: MemberMacro {
    
    public static func expansion(
        of node: AttributeSyntax,
        providingMembersOf declaration: some DeclGroupSyntax,
        in context: some MacroExpansionContext
    ) throws -> [DeclSyntax] {
        
        guard let structDecl = declaration.as(StructDeclSyntax.self) else {
            throw MacroExpansionErrorMessage(
                "@Builder can only be applied to structs"
            )
        }
        
        let typeName = structDecl.name.text
        
        // ดึง properties
        let properties = extractMutableProperties(from: structDecl)
        
        // สร้าง builder methods ทุก property
        var declarations: [DeclSyntax] = []
        
        for prop in properties {
            let builderMethod: DeclSyntax = """
            @discardableResult
            func with\(raw: prop.name.capitalizedFirst)(_ value: \(raw: prop.type)) -> \(raw: typeName) {
                var copy = self
                copy.\(raw: prop.name) = value
                return copy
            }
            """
            declarations.append(builderMethod)
        }
        
        return declarations
    }
    
    private static func extractMutableProperties(
        from structDecl: StructDeclSyntax
    ) -> [(name: String, type: String)] {
        
        return structDecl.memberBlock.members.compactMap { member -> (String, String)? in
            guard let varDecl = member.decl.as(VariableDeclSyntax.self),
                  varDecl.bindingSpecifier.tokenKind == .keyword(.var),
                  let binding = varDecl.bindings.first,
                  let identifier = binding.pattern.as(IdentifierPatternSyntax.self),
                  let typeAnnotation = binding.typeAnnotation,
                  binding.accessorBlock == nil else {
                return nil
            }
            
            return (
                identifier.identifier.text,
                typeAnnotation.type.description.trimmingCharacters(in: .whitespaces)
            )
        }
    }
}

extension String {
    var capitalizedFirst: String {
        guard let first = self.first else { return self }
        return first.uppercased() + dropFirst()
    }
}
```

### 10.2 การใช้งาน @Builder

```swift
import MyMacros

@Builder
struct NetworkRequest {
    var url: String = ""
    var method: String = "GET"
    var headers: [String: String] = [:]
    var body: Data? = nil
    var timeout: Double = 30.0
}

// ใช้ builder pattern
let request = NetworkRequest()
    .withUrl("https://api.example.com/users")
    .withMethod("POST")
    .withHeaders(["Authorization": "Bearer token123"])
    .withBody(jsonData)
    .withTimeout(60.0)

// สวยงามกว่าการ assign ทีละ property
print(request.url)     // "https://api.example.com/users"
print(request.method)  // "POST"
print(request.timeout) // 60.0

// ใช้ใน test
let testRequest = NetworkRequest()
    .withUrl("https://test.example.com")
    .withTimeout(5.0)
```

---

## 11. Diagnostics in Macros

### 11.1 Macro Expansion Diagnostics

```swift
import SwiftDiagnostics
import SwiftSyntax
import SwiftSyntaxMacros

// ประเภทของ diagnostic
enum MacroDiagnostic: DiagnosticMessage {
    case invalidType(expected: String, got: String)
    case missingAnnotation(annotation: String)
    case incompatibleOptions(option1: String, option2: String)
    
    var message: String {
        switch self {
        case .invalidType(let expected, let got):
            return "Expected \(expected) but got \(got)"
        case .missingAnnotation(let annotation):
            return "Missing required annotation '@\(annotation)'"
        case .incompatibleOptions(let o1, let o2):
            return "Options '\(o1)' and '\(o2)' cannot be used together"
        }
    }
    
    var diagnosticID: MessageID {
        MessageID(domain: "MyMacros", id: String(describing: self))
    }
    
    var severity: DiagnosticSeverity {
        return .error
    }
}
```

### 11.2 Error and Warning Messages

```swift
public struct StrictMacro: MemberMacro {
    
    public static func expansion(
        of node: AttributeSyntax,
        providingMembersOf declaration: some DeclGroupSyntax,
        in context: some MacroExpansionContext
    ) throws -> [DeclSyntax] {
        
        // Warning: การใช้ macro บน class แบบนี้อาจมีปัญหา
        if let classDecl = declaration.as(ClassDeclSyntax.self) {
            let classNode = Syntax(classDecl)
            context.diagnose(
                Diagnostic(
                    node: classNode,
                    message: MacroWarning.classUsage
                )
            )
        }
        
        // Error: ไม่ support enum
        if declaration.is(EnumDeclSyntax.self) {
            let enumNode = Syntax(declaration)
            context.diagnose(
                Diagnostic(
                    node: enumNode,
                    message: MacroDiagnostic.invalidType(
                        expected: "struct or class",
                        got: "enum"
                    )
                )
            )
            return []
        }
        
        return []
    }
}

enum MacroWarning: DiagnosticMessage {
    case classUsage
    
    var message: String {
        switch self {
        case .classUsage:
            return "This macro works better with structs. Consider using a struct instead."
        }
    }
    
    var diagnosticID: MessageID {
        MessageID(domain: "MyMacros", id: "warning_\(String(describing: self))")
    }
    
    var severity: DiagnosticSeverity {
        return .warning
    }
}
```

### 11.3 Fix-it Suggestions

```swift
import SwiftDiagnostics
import SwiftSyntax

public struct FixItMacro: MemberMacro {
    
    public static func expansion(
        of node: AttributeSyntax,
        providingMembersOf declaration: some DeclGroupSyntax,
        in context: some MacroExpansionContext
    ) throws -> [DeclSyntax] {
        
        // ตรวจสอบ access control
        if let structDecl = declaration.as(StructDeclSyntax.self) {
            for member in structDecl.memberBlock.members {
                guard let varDecl = member.decl.as(VariableDeclSyntax.self),
                      varDecl.bindingSpecifier.tokenKind == .keyword(.var) else {
                    continue
                }
                
                // ถ้า property ไม่มี access modifier
                if varDecl.modifiers.isEmpty {
                    let fixIt = FixIt(
                        message: FixItMessage.addPrivate,
                        changes: [
                            .replace(
                                oldNode: Syntax(varDecl),
                                newNode: Syntax(
                                    varDecl.with(\.modifiers, [
                                        DeclModifierSyntax(
                                            name: .keyword(.private)
                                        )
                                    ])
                                )
                            )
                        ]
                    )
                    
                    context.diagnose(
                        Diagnostic(
                            node: Syntax(varDecl),
                            message: MacroDiagnostic.missingAnnotation(annotation: "private"),
                            fixIts: [fixIt]
                        )
                    )
                }
            }
        }
        
        return []
    }
}

enum FixItMessage: FixItMessage {
    case addPrivate
    
    var message: String {
        switch self {
        case .addPrivate: return "Add 'private' modifier"
        }
    }
    
    var fixItID: MessageID {
        MessageID(domain: "MyMacros", id: "fixit_\(String(describing: self))")
    }
}
```

---

## 12. Testing Macros Comprehensively

### 12.1 assertMacroExpansion Helper

```swift
import XCTest
import SwiftSyntaxMacros
import SwiftSyntaxMacrosTestSupport
import MyMacrosImpl

final class AutoEquatableMacroTests: XCTestCase {
    
    let testMacros: [String: Macro.Type] = [
        "AutoEquatable": AutoEquatableMacro.self,
    ]
    
    // Test basic expansion
    func testBasicExpansion() {
        assertMacroExpansion(
            """
            @AutoEquatable
            struct Point {
                var x: Double
                var y: Double
            }
            """,
            expandedSource: """
            struct Point {
                var x: Double
                var y: Double
            
                static func == (lhs: Point, rhs: Point) -> Bool {
                    return lhs.x == rhs.x &&
                        lhs.y == rhs.y
                }
            }
            
            extension Point: Equatable {
            }
            """,
            macros: testMacros
        )
    }
    
    // Test empty struct
    func testEmptyStruct() {
        assertMacroExpansion(
            """
            @AutoEquatable
            struct Empty {
            }
            """,
            expandedSource: """
            struct Empty {
            
                static func == (lhs: Empty, rhs: Empty) -> Bool {
                    return true
                }
            }
            
            extension Empty: Equatable {
            }
            """,
            macros: testMacros
        )
    }
    
    // Test struct with computed properties (should be skipped)
    func testSkipsComputedProperties() {
        assertMacroExpansion(
            """
            @AutoEquatable
            struct Rectangle {
                var width: Double
                var height: Double
                var area: Double { width * height }
            }
            """,
            expandedSource: """
            struct Rectangle {
                var width: Double
                var height: Double
                var area: Double { width * height }
            
                static func == (lhs: Rectangle, rhs: Rectangle) -> Bool {
                    return lhs.width == rhs.width &&
                        lhs.height == rhs.height
                }
            }
            
            extension Rectangle: Equatable {
            }
            """,
            macros: testMacros
        )
    }
}
```

### 12.2 Testing Error Cases

```swift
final class AutoEquatableMacroErrorTests: XCTestCase {
    
    let testMacros: [String: Macro.Type] = [
        "AutoEquatable": AutoEquatableMacro.self,
    ]
    
    // Test error when applied to class
    func testErrorOnClass() {
        assertMacroExpansion(
            """
            @AutoEquatable
            class Person {
                var name: String = ""
            }
            """,
            expandedSource: """
            class Person {
                var name: String = ""
            }
            """,
            diagnostics: [
                DiagnosticSpec(
                    message: "@AutoEquatable can only be applied to structs. For classes, implement Equatable manually.",
                    line: 1,
                    column: 1,
                    severity: .error
                )
            ],
            macros: testMacros
        )
    }
    
    // Test error when applied to enum
    func testErrorOnEnum() {
        assertMacroExpansion(
            """
            @AutoEquatable
            enum Status {
                case active
                case inactive
            }
            """,
            expandedSource: """
            enum Status {
                case active
                case inactive
            }
            """,
            diagnostics: [
                DiagnosticSpec(
                    message: "@AutoEquatable can only be applied to structs. For classes, implement Equatable manually.",
                    line: 1,
                    column: 1,
                    severity: .error
                )
            ],
            macros: testMacros
        )
    }
}
```

### 12.3 Testing with Different Inputs

```swift
final class URLMacroComprehensiveTests: XCTestCase {
    
    let testMacros: [String: Macro.Type] = [
        "URL": URLMacro.self,
    ]
    
    // Test ประเภท URL ต่างๆ
    func testHTTPURL() {
        assertMacroExpansion(
            """
            let url = #URL("http://example.com")
            """,
            expandedSource: """
            let url = URL(string: "http://example.com")!
            """,
            macros: testMacros
        )
    }
    
    func testHTTPSURL() {
        assertMacroExpansion(
            """
            let url = #URL("https://api.example.com/v2/users?page=1")
            """,
            expandedSource: """
            let url = URL(string: "https://api.example.com/v2/users?page=1")!
            """,
            macros: testMacros
        )
    }
    
    func testFileURL() {
        assertMacroExpansion(
            """
            let url = #URL("file:///tmp/document.pdf")
            """,
            expandedSource: """
            let url = URL(string: "file:///tmp/document.pdf")!
            """,
            macros: testMacros
        )
    }
    
    func testURLWithSpecialCharacters() {
        assertMacroExpansion(
            """
            let url = #URL("https://example.com/path%20with%20spaces")
            """,
            expandedSource: """
            let url = URL(string: "https://example.com/path%20with%20spaces")!
            """,
            macros: testMacros
        )
    }
    
    // Test invalid URLs
    func testEmptyStringFails() {
        assertMacroExpansion(
            """
            let url = #URL("")
            """,
            expandedSource: """
            let url = #URL("")
            """,
            diagnostics: [
                DiagnosticSpec(
                    message: "Invalid URL: ''",
                    line: 1,
                    column: 11
                )
            ],
            macros: testMacros
        )
    }
    
    func testURLWithSpacesInHostFails() {
        assertMacroExpansion(
            """
            let url = #URL("https://example .com")
            """,
            expandedSource: """
            let url = #URL("https://example .com")
            """,
            diagnostics: [
                DiagnosticSpec(
                    message: "Invalid URL: 'https://example .com'",
                    line: 1,
                    column: 11
                )
            ],
            macros: testMacros
        )
    }
}
```

---

## 13. Macro Composition

### 13.1 Applying Multiple Macros to One Type

```swift
import MyMacros

// ใช้หลาย macros ด้วยกัน
@AutoEquatable          // เพิ่ม Equatable conformance
@AutoHashable           // เพิ่ม Hashable conformance
@Builder                // เพิ่ม builder methods
@CustomStringConvertible // เพิ่ม description property
struct Product {
    var id: UUID
    var name: String
    var price: Decimal
    var category: String
    var isAvailable: Bool
}

// ผลลัพธ์: struct ที่มี
// - Equatable conformance ผ่าน == operator
// - Hashable conformance ผ่าน hash(into:)
// - Builder methods: withId(), withName(), withPrice(), etc.
// - description property

// การใช้งาน
let product = Product(
    id: UUID(),
    name: "iPhone",
    price: 999.99,
    category: "Electronics",
    isAvailable: true
)

let updatedProduct = product
    .withName("iPhone Pro")
    .withPrice(1199.99)

// ใช้ใน Set เพราะ Hashable
var productSet: Set<Product> = [product, updatedProduct]

// Print ออกมาสวย
print(product.description)
// "Product(id: ..., name: iPhone, price: 999.99, category: Electronics, isAvailable: true)"
```

### 13.2 Order of Application

```swift
// ลำดับของ macros สำคัญในบางกรณี

// ✅ ลำดับที่ถูกต้อง - conformance macros ก่อน member macros
@AutoHashable  // เพิ่ม Hashable (ต้องการ Equatable ก่อน)
@AutoEquatable // เพิ่ม Equatable
struct Point {
    var x: Double
    var y: Double
}

// ⚠️ ลำดับที่อาจมีปัญหา
// @AutoHashable ต้องการให้ Equatable มีอยู่แล้ว
// ถ้า expansion order ไม่ถูกต้อง อาจเกิด error

// การจัดการ:
// ใช้ composite macro ที่รวมทั้งสองเข้าด้วยกัน
@attached(conformance)
@attached(member, names: named(==), named(hash(into:)))
public macro AutoEquatableHashable() =
    #externalMacro(module: "MyMacrosImpl", type: "AutoEquatableHashableMacro")
```

```swift
// Implementation ของ composite macro
public struct AutoEquatableHashableMacro: ConformanceMacro, MemberMacro {
    
    public static func expansion(
        of node: AttributeSyntax,
        providingConformancesOf declaration: some DeclGroupSyntax,
        in context: some MacroExpansionContext
    ) throws -> [(TypeSyntax, GenericWhereClauseSyntax?)] {
        return [("Equatable", nil), ("Hashable", nil)]
    }
    
    public static func expansion(
        of node: AttributeSyntax,
        providingMembersOf declaration: some DeclGroupSyntax,
        in context: some MacroExpansionContext
    ) throws -> [DeclSyntax] {
        
        guard let structDecl = declaration.as(StructDeclSyntax.self) else {
            throw MacroExpansionErrorMessage("Can only be applied to structs")
        }
        
        let typeName = structDecl.name.text
        let properties = getProperties(from: structDecl)
        
        // == operator
        let comparisons = properties.map { "lhs.\($0) == rhs.\($0)" }.joined(separator: " && ")
        let equatable: DeclSyntax = """
        static func == (lhs: \(raw: typeName), rhs: \(raw: typeName)) -> Bool {
            return \(raw: comparisons)
        }
        """
        
        // hash(into:)
        let combines = properties.map { "hasher.combine(\($0))" }.joined(separator: "\n    ")
        let hashable: DeclSyntax = """
        func hash(into hasher: inout Hasher) {
            \(raw: combines)
        }
        """
        
        return [equatable, hashable]
    }
    
    private static func getProperties(from decl: StructDeclSyntax) -> [String] {
        return decl.memberBlock.members.compactMap { member -> String? in
            guard let varDecl = member.decl.as(VariableDeclSyntax.self),
                  let binding = varDecl.bindings.first,
                  let identifier = binding.pattern.as(IdentifierPatternSyntax.self),
                  binding.accessorBlock == nil else {
                return nil
            }
            return identifier.identifier.text
        }
    }
}
```

---

## 14. Limitations and Best Practices

### 14.1 When NOT to Use Macros

```swift
// ❌ Macro ไม่เหมาะสำหรับ runtime behavior
// อย่าใช้ macro สำหรับ logic ที่ต้องทำงานใน runtime
@macro
func validateAtRuntime(_ value: Any) -> Bool {
    // ❌ ไม่ถูกต้อง - macro ทำงานใน compile time เท่านั้น
}

// ✅ ใช้ function ปกติแทน
func validate(_ value: Any) -> Bool {
    // Logic ที่ทำงานใน runtime
}

// ❌ Macro ไม่ควรใช้สำหรับ simple utilities
// ถ้า code น้อยกว่า 5 บรรทัด ไม่ต้องใช้ macro
@StringHelper
var name: String // ❌ Overkill

// ✅ แค่ใช้ property ปกติ
var name: String

// ❌ Macro ไม่เหมาะสำหรับ logic ที่ซับซ้อนมาก
// ถ้า expansion มีหลายร้อยบรรทัด ให้แยกเป็น helper functions

// ✅ Macro เหมาะสำหรับ:
// 1. Boilerplate elimination (Equatable, Hashable, Codable)
// 2. Type-safe wrappers (#URL, #Regex)
// 3. Code patterns ที่ซ้ำๆ แต่แตกต่างกัน per-type
```

### 14.2 Macro Visibility

```swift
// Macros ต้อง public เพื่อใช้นอก module
public struct MyMacro: ExpressionMacro {
    public static func expansion(...) throws -> ExprSyntax {
        // ...
    }
}

// Macro declaration ใน public API:
@freestanding(expression)
public macro myMacro() = #externalMacro(module: "...", type: "...")
//    ^^^^^^ ต้องเป็น public

// Internal macro สำหรับใช้ภายใน module เท่านั้น:
@freestanding(expression)
macro internalMacro() = #externalMacro(module: "...", type: "...")
//  ไม่มี public = internal by default
```

### 14.3 Compilation Time Impact

```swift
// Macros compile ช้าลงเล็กน้อย เพราะต้อง:
// 1. Compile macro implementation (เกิดขึ้นครั้งแรก)
// 2. Run macro expansion สำหรับแต่ละ use site

// Best Practices เพื่อลด compile time:
// 1. Cache expensive computations ใน macro implementation
// 2. ใช้ early returns เมื่อ input ไม่ valid
// 3. ทดสอบกับ large codebases

// ตัวอย่าง optimized macro:
public struct OptimizedMacro: MemberMacro {
    
    // ใช้ dictionary สำหรับ caching (ระหว่าง expansion runs)
    private static var cache: [String: [DeclSyntax]] = [:]
    
    public static func expansion(
        of node: AttributeSyntax,
        providingMembersOf declaration: some DeclGroupSyntax,
        in context: some MacroExpansionContext
    ) throws -> [DeclSyntax] {
        
        // Early return ถ้าไม่ใช่ struct
        guard let structDecl = declaration.as(StructDeclSyntax.self) else {
            return []
        }
        
        let key = structDecl.description
        
        // ใช้ cached result ถ้ามี
        if let cached = cache[key] {
            return cached
        }
        
        let result = try generateMembers(for: structDecl)
        cache[key] = result
        return result
    }
    
    private static func generateMembers(for decl: StructDeclSyntax) throws -> [DeclSyntax] {
        // Actual implementation
        return []
    }
}
```

---

## 15. Complete Macro Package: SwiftMacros Utility Library

### 15.1 Package Structure

```
SwiftMacros/
├── Package.swift
├── Sources/
│   ├── SwiftMacros/
│   │   ├── SwiftMacros.swift           # Public declarations
│   │   ├── Codable+Macros.swift        # Codable-related macros
│   │   ├── Equatable+Macros.swift      # Equatable macros
│   │   └── Builder+Macros.swift        # Builder macros
│   └── SwiftMacrosImpl/
│       ├── Plugin.swift
│       ├── Codable/
│       │   ├── CustomCodableMacro.swift
│       │   ├── CodingKeyMacro.swift
│       │   └── DefaultValueMacro.swift
│       ├── Equatable/
│       │   ├── AutoEquatableMacro.swift
│       │   └── AutoHashableMacro.swift
│       └── Builder/
│           └── BuilderMacro.swift
└── Tests/
    └── SwiftMacrosTests/
        ├── CodableTests.swift
        ├── EquatableTests.swift
        └── BuilderTests.swift
```

### 15.2 Complete Plugin

```swift
// Sources/SwiftMacrosImpl/Plugin.swift
import SwiftCompilerPlugin
import SwiftSyntaxMacros

@main
struct SwiftMacrosPlugin: CompilerPlugin {
    let providingMacros: [Macro.Type] = [
        // Expression macros
        StringifyMacro.self,
        URLMacro.self,
        AssertMacro.self,
        
        // Equatable & Hashable
        AutoEquatableMacro.self,
        AutoHashableMacro.self,
        AutoEquatableHashableMacro.self,
        
        // Codable
        CustomCodableMacro.self,
        CodingKeyAttributeMacro.self,
        DefaultValueMacro.self,
        IgnoreCodingMacro.self,
        
        // Builder
        BuilderMacro.self,
        
        // Observable
        ObservableMacro.self,
    ]
}
```

### 15.3 Public API ครบถ้วน

```swift
// Sources/SwiftMacros/SwiftMacros.swift
import Foundation

// MARK: - Expression Macros

/// แปลง expression เป็น (value, source_string)
@freestanding(expression)
public macro stringify<T>(_ value: T) -> (T, String) =
    #externalMacro(module: "SwiftMacrosImpl", type: "StringifyMacro")

/// สร้าง URL อย่างปลอดภัยพร้อม compile-time validation
@freestanding(expression)
public macro URL(_ stringLiteral: String) -> Foundation.URL =
    #externalMacro(module: "SwiftMacrosImpl", type: "URLMacro")

/// Enhanced assert ที่แสดง source code
@freestanding(expression)
public macro assert(_ condition: Bool) -> Void =
    #externalMacro(module: "SwiftMacrosImpl", type: "AssertMacro")

// MARK: - Equatable & Hashable Macros

/// Auto-generate Equatable conformance
@attached(conformance)
@attached(member, names: named(==))
public macro AutoEquatable() =
    #externalMacro(module: "SwiftMacrosImpl", type: "AutoEquatableMacro")

/// Auto-generate Hashable conformance
@attached(conformance)
@attached(member, names: named(hash(into:)))
public macro AutoHashable() =
    #externalMacro(module: "SwiftMacrosImpl", type: "AutoHashableMacro")

/// Auto-generate both Equatable and Hashable
@attached(conformance)
@attached(member, names: named(==), named(hash(into:)))
public macro AutoEquatableHashable() =
    #externalMacro(module: "SwiftMacrosImpl", type: "AutoEquatableHashableMacro")

// MARK: - Codable Macros

/// Custom Codable implementation with attribute support
@attached(member, names: named(CodingKeys), named(init(from:)), named(encode(to:)))
public macro CustomCodable() =
    #externalMacro(module: "SwiftMacrosImpl", type: "CustomCodableMacro")

/// Custom coding key for a property
@attached(peer)
public macro CodingKey(_ key: String) =
    #externalMacro(module: "SwiftMacrosImpl", type: "CodingKeyAttributeMacro")

/// Default value when key is missing
@attached(peer)
public macro DefaultValue<T>(_ value: T) =
    #externalMacro(module: "SwiftMacrosImpl", type: "DefaultValueMacro")

/// Skip this property during coding
@attached(peer)
public macro IgnoreCoding() =
    #externalMacro(module: "SwiftMacrosImpl", type: "IgnoreCodingMacro")

// MARK: - Builder Macros

/// Generate fluent builder methods
@attached(member, names: arbitrary)
public macro Builder() =
    #externalMacro(module: "SwiftMacrosImpl", type: "BuilderMacro")
```

### 15.4 ตัวอย่างการใช้งาน Library ครบถ้วน

```swift
import SwiftMacros
import Foundation

// Complete data model ที่ใช้ macros หลายตัว
@AutoEquatableHashable
@Builder
@CustomCodable
struct APIResponse {
    @CodingKey("user_id")
    var userId: String
    
    @CodingKey("full_name")
    var fullName: String
    
    @CodingKey("email_address")
    var email: String
    
    @DefaultValue(0)
    var followersCount: Int
    
    @DefaultValue(false)
    var isVerified: Bool
    
    @IgnoreCoding
    var cachedAvatarImage: UIImage? = nil
    
    var bio: String?
    var joinedAt: Date?
}

// การใช้งาน
let jsonData = """
{
    "user_id": "u123",
    "full_name": "John Doe",
    "email_address": "john@example.com",
    "followers_count": 1500,
    "is_verified": true,
    "bio": "iOS Developer"
}
""".data(using: .utf8)!

let decoder = JSONDecoder()
decoder.dateDecodingStrategy = .iso8601
let response = try decoder.decode(APIResponse.self, from: jsonData)

print(response.userId)        // "u123"
print(response.fullName)      // "John Doe"
print(response.followersCount) // 1500

// Builder
let updated = response
    .withFullName("Jane Doe")
    .withEmail("jane@example.com")
    .withFollowersCount(2000)

// Equatable & Hashable
print(response == updated)    // false
var responseSet: Set<APIResponse> = [response, updated]
print(responseSet.count)      // 2

// URL macro ใน action
let (apiUrl, urlSource) = #stringify(#URL("https://api.example.com/users/\(response.userId)"))
print("Calling: \(urlSource)")
```

---

## 16. Exercises with Solutions

### Exercise 1: @Singleton Macro

**โจทย์:** สร้าง `@Singleton` macro ที่ generate singleton pattern สำหรับ class

```swift
// Expected usage:
@Singleton
class DatabaseManager {
    var connectionString: String = ""
    func connect() { ... }
}

// Expected expansion:
class DatabaseManager {
    static let shared = DatabaseManager()
    private init() {}
    var connectionString: String = ""
    func connect() { ... }
}
```

**Solution:**
```swift
// Declaration
@attached(member, names: named(shared), named(init()))
public macro Singleton() =
    #externalMacro(module: "MacroImpl", type: "SingletonMacro")

// Implementation
public struct SingletonMacro: MemberMacro {
    
    public static func expansion(
        of node: AttributeSyntax,
        providingMembersOf declaration: some DeclGroupSyntax,
        in context: some MacroExpansionContext
    ) throws -> [DeclSyntax] {
        
        guard let classDecl = declaration.as(ClassDeclSyntax.self) else {
            throw MacroExpansionErrorMessage(
                "@Singleton can only be applied to classes"
            )
        }
        
        let typeName = classDecl.name.text
        
        let sharedInstance: DeclSyntax = """
        static let shared = \(raw: typeName)()
        """
        
        let privateInit: DeclSyntax = """
        private init() {}
        """
        
        return [sharedInstance, privateInit]
    }
}
```

### Exercise 2: @UserDefaultsBacked Macro

**โจทย์:** สร้าง `@UserDefaultsBacked` macro ที่ backup property ไปยัง UserDefaults อัตโนมัติ

```swift
// Expected usage:
struct Settings {
    @UserDefaultsBacked(key: "app_theme", defaultValue: "light")
    var theme: String
    
    @UserDefaultsBacked(key: "font_size", defaultValue: 14)
    var fontSize: Int
}

// Expected expansion:
struct Settings {
    var theme: String {
        get {
            return UserDefaults.standard.string(forKey: "app_theme") ?? "light"
        }
        set {
            UserDefaults.standard.set(newValue, forKey: "app_theme")
        }
    }
    
    var fontSize: Int {
        get {
            return UserDefaults.standard.integer(forKey: "font_size") != 0 
                   ? UserDefaults.standard.integer(forKey: "font_size") 
                   : 14
        }
        set {
            UserDefaults.standard.set(newValue, forKey: "font_size")
        }
    }
}
```

**Solution:**
```swift
// Declaration
@attached(accessor, names: named(get), named(set))
public macro UserDefaultsBacked<T>(key: String, defaultValue: T) =
    #externalMacro(module: "MacroImpl", type: "UserDefaultsBackedMacro")

// Implementation
public struct UserDefaultsBackedMacro: AccessorMacro {
    
    public static func expansion(
        of node: AttributeSyntax,
        providingAccessorsOf declaration: some DeclSyntaxProtocol,
        in context: some MacroExpansionContext
    ) throws -> [AccessorDeclSyntax] {
        
        // ดึง arguments
        guard let args = node.arguments?.as(LabeledExprListSyntax.self) else {
            throw MacroExpansionErrorMessage("Missing arguments")
        }
        
        var key: String = ""
        var defaultValue: String = ""
        
        for arg in args {
            if arg.label?.text == "key",
               let strLit = arg.expression.as(StringLiteralExprSyntax.self),
               let segment = strLit.segments.first?.as(StringSegmentSyntax.self) {
                key = segment.content.text
            } else if arg.label?.text == "defaultValue" {
                defaultValue = arg.expression.description.trimmingCharacters(in: .whitespaces)
            }
        }
        
        // ดึง property type
        guard let varDecl = declaration.as(VariableDeclSyntax.self),
              let binding = varDecl.bindings.first,
              let typeAnnotation = binding.typeAnnotation else {
            throw MacroExpansionErrorMessage("Requires explicit type annotation")
        }
        
        let typeName = typeAnnotation.type.description.trimmingCharacters(in: .whitespaces)
        
        // สร้าง getter/setter based on type
        let getter: AccessorDeclSyntax = """
        get {
            return UserDefaults.standard.object(forKey: \(literal: key)) as? \(raw: typeName) ?? \(raw: defaultValue)
        }
        """
        
        let setter: AccessorDeclSyntax = """
        set {
            UserDefaults.standard.set(newValue, forKey: \(literal: key))
        }
        """
        
        return [getter, setter]
    }
}
```

### Exercise 3: @Logging Macro

**โจทย์:** สร้าง `@Logging` macro ที่เพิ่ม logging ให้ function อัตโนมัติ

```swift
// Expected usage:
class UserService {
    @Logging
    func createUser(_ name: String, email: String) -> User {
        // ...
    }
}

// Expected expansion:
class UserService {
    func createUser(_ name: String, email: String) -> User {
        print("[\(type(of: self))] createUser called with name: \(name), email: \(email)")
        let result = _createUser_impl(name, email: email)
        print("[\(type(of: self))] createUser returned: \(result)")
        return result
    }
    
    private func _createUser_impl(_ name: String, email: String) -> User {
        // Original implementation
    }
}
```

**Solution:**
```swift
// Declaration
@attached(peer, names: prefixed(_), suffixed(_impl))
public macro Logging() =
    #externalMacro(module: "MacroImpl", type: "LoggingMacro")

// Implementation  
public struct LoggingMacro: PeerMacro {
    
    public static func expansion(
        of node: AttributeSyntax,
        providingPeersOf declaration: some DeclSyntaxProtocol,
        in context: some MacroExpansionContext
    ) throws -> [DeclSyntax] {
        
        guard let funcDecl = declaration.as(FunctionDeclSyntax.self) else {
            throw MacroExpansionErrorMessage(
                "@Logging can only be applied to functions"
            )
        }
        
        let funcName = funcDecl.name.text
        let params = funcDecl.signature.parameterClause.parameters
        
        // สร้าง log message สำหรับ parameters
        let paramLog = params
            .map { param -> String in
                let label = param.firstName.text
                let paramName = (param.secondName ?? param.firstName).text
                return "\(label): \\(\(paramName))"
            }
            .joined(separator: ", ")
        
        // สร้าง call ไปยัง original function
        let callArgs = params
            .map { param -> String in
                let label = param.firstName.text
                let paramName = (param.secondName ?? param.firstName).text
                if label == "_" {
                    return paramName
                }
                return "\(label): \(paramName)"
            }
            .joined(separator: ", ")
        
        let returnType = funcDecl.signature.returnClause?.type.description
            .trimmingCharacters(in: .whitespaces) ?? "Void"
        
        let hasReturn = returnType != "Void" && returnType != "()"
        
        let loggedFunction: DeclSyntax
        
        if hasReturn {
            loggedFunction = """
            func \(raw: funcName)\(funcDecl.signature) {
                print("[\\(type(of: self))] \(raw: funcName) called with \(raw: paramLog)")
                let result = _\(raw: funcName)_impl(\(raw: callArgs))
                print("[\\(type(of: self))] \(raw: funcName) returned: \\(result)")
                return result
            }
            """
        } else {
            loggedFunction = """
            func \(raw: funcName)\(funcDecl.signature) {
                print("[\\(type(of: self))] \(raw: funcName) called with \(raw: paramLog)")
                _\(raw: funcName)_impl(\(raw: callArgs))
                print("[\\(type(of: self))] \(raw: funcName) completed")
            }
            """
        }
        
        return [loggedFunction]
    }
}
```

### Exercise 4: @CaseIterable Enhancement

**โจทย์:** สร้าง macro ที่ generate `allCases` พร้อม metadata สำหรับ enum

```swift
// Expected usage:
@EnhancedCaseIterable
enum Planet {
    case mercury
    case venus
    case earth
    case mars
}

// Expected expansion:
extension Planet: CaseIterable {
    static var allCases: [Planet] = [.mercury, .venus, .earth, .mars]
    
    var name: String {
        switch self {
        case .mercury: return "Mercury"
        case .venus: return "Venus"
        case .earth: return "Earth"
        case .mars: return "Mars"
        }
    }
    
    var index: Int {
        return Self.allCases.firstIndex(of: self) ?? 0
    }
}
```

**Solution:**
```swift
// Declaration
@attached(conformance)
@attached(extension, conformances: CaseIterable, names: named(allCases), named(name), named(index))
public macro EnhancedCaseIterable() =
    #externalMacro(module: "MacroImpl", type: "EnhancedCaseIterableMacro")

// Implementation
public struct EnhancedCaseIterableMacro: ExtensionMacro {
    
    public static func expansion(
        of node: AttributeSyntax,
        attachedTo declaration: some DeclGroupSyntax,
        providingExtensionsOf type: some TypeSyntaxProtocol,
        conformingTo protocols: [TypeSyntax],
        in context: some MacroExpansionContext
    ) throws -> [ExtensionDeclSyntax] {
        
        guard let enumDecl = declaration.as(EnumDeclSyntax.self) else {
            throw MacroExpansionErrorMessage(
                "@EnhancedCaseIterable can only be applied to enums"
            )
        }
        
        let typeName = enumDecl.name.text
        
        // ดึง cases
        let cases = enumDecl.memberBlock.members.compactMap { member -> String? in
            guard let caseDecl = member.decl.as(EnumCaseDeclSyntax.self),
                  let element = caseDecl.elements.first else {
                return nil
            }
            return element.name.text
        }
        
        // สร้าง allCases
        let allCasesStr = cases.map { ".\($0)" }.joined(separator: ", ")
        
        // สร้าง name switch
        let nameCases = cases
            .map { c in "case .\(c): return \"\(c.capitalizedFirst)\"" }
            .joined(separator: "\n        ")
        
        let extensionDecl = try ExtensionDeclSyntax(
            "extension \(raw: typeName): CaseIterable"
        ) {
            """
            static var allCases: [\(raw: typeName)] = [\(raw: allCasesStr)]
            
            var name: String {
                switch self {
                \(raw: nameCases)
                }
            }
            
            var index: Int {
                return Self.allCases.firstIndex(of: self) ?? 0
            }
            """
        }
        
        return [extensionDecl]
    }
}
```

### Exercise 5: @ThreadSafe Macro

**โจทย์:** สร้าง `@ThreadSafe` macro ที่ wrap property access ด้วย NSLock

```swift
// Expected usage:
class Cache {
    @ThreadSafe
    var items: [String: Any] = [:]
}

// Expected expansion:
class Cache {
    private let _itemsLock = NSLock()
    private var _items: [String: Any] = [:]
    
    var items: [String: Any] {
        get {
            _itemsLock.lock()
            defer { _itemsLock.unlock() }
            return _items
        }
        set {
            _itemsLock.lock()
            defer { _itemsLock.unlock() }
            _items = newValue
        }
    }
}
```

**Solution:**
```swift
// Declaration
@attached(accessor, names: named(get), named(set))
@attached(peer, names: prefixed(_), suffixed(Lock))
public macro ThreadSafe() =
    #externalMacro(module: "MacroImpl", type: "ThreadSafeMacro")

// Implementation
public struct ThreadSafeMacro: AccessorMacro, PeerMacro {
    
    // สร้าง accessor
    public static func expansion(
        of node: AttributeSyntax,
        providingAccessorsOf declaration: some DeclSyntaxProtocol,
        in context: some MacroExpansionContext
    ) throws -> [AccessorDeclSyntax] {
        
        guard let varDecl = declaration.as(VariableDeclSyntax.self),
              let binding = varDecl.bindings.first,
              let identifier = binding.pattern.as(IdentifierPatternSyntax.self) else {
            return []
        }
        
        let name = identifier.identifier.text
        
        let getter: AccessorDeclSyntax = """
        get {
            _\(raw: name)Lock.lock()
            defer { _\(raw: name)Lock.unlock() }
            return _\(raw: name)
        }
        """
        
        let setter: AccessorDeclSyntax = """
        set {
            _\(raw: name)Lock.lock()
            defer { _\(raw: name)Lock.unlock() }
            _\(raw: name) = newValue
        }
        """
        
        return [getter, setter]
    }
    
    // สร้าง backing storage และ lock
    public static func expansion(
        of node: AttributeSyntax,
        providingPeersOf declaration: some DeclSyntaxProtocol,
        in context: some MacroExpansionContext
    ) throws -> [DeclSyntax] {
        
        guard let varDecl = declaration.as(VariableDeclSyntax.self),
              let binding = varDecl.bindings.first,
              let identifier = binding.pattern.as(IdentifierPatternSyntax.self),
              let typeAnnotation = binding.typeAnnotation else {
            return []
        }
        
        let name = identifier.identifier.text
        let typeName = typeAnnotation.type.description.trimmingCharacters(in: .whitespaces)
        let initialValue = binding.initializer?.value.description
            .trimmingCharacters(in: .whitespaces) ?? ""
        
        let lock: DeclSyntax = """
        private let _\(raw: name)Lock = NSLock()
        """
        
        let storage: DeclSyntax
        if !initialValue.isEmpty {
            storage = """
            private var _\(raw: name): \(raw: typeName) = \(raw: initialValue)
            """
        } else {
            storage = """
            private var _\(raw: name): \(raw: typeName)
            """
        }
        
        return [lock, storage]
    }
}
```

---

## สรุป

Swift Macros เป็นฟีเจอร์ที่ทรงพลังมากใน Swift 5.9+ ที่ช่วยให้เราสามารถ:

1. **ลด boilerplate code** โดยการ generate code อัตโนมัติ
2. **เพิ่ม type safety** ด้วย compile-time validation
3. **IDE integration** เต็มรูปแบบผ่าน SwiftSyntax
4. **Testing** ที่ดีด้วย `assertMacroExpansion`

### Key Takeaways

- Macro roles แตกต่างกันตามวัตถุประสงค์: expression, declaration, member, accessor, peer, conformance, extension
- SwiftSyntax เป็น foundation ที่ macros ทำงาน
- Test macros อย่างละเอียดด้วย `SwiftSyntaxMacrosTestSupport`
- ใช้ diagnostics เพื่อให้ error messages ที่มีความหมาย
- อย่าใช้ macros เกินความจำเป็น - บางครั้ง function ธรรมดาดีกว่า

### Resources

- [Swift Evolution SE-0382: Expression macros](https://github.com/apple/swift-evolution/blob/main/proposals/0382-expression-macros.md)
- [Swift Evolution SE-0389: Attached macros](https://github.com/apple/swift-evolution/blob/main/proposals/0389-attached-macros.md)
- [SwiftSyntax Documentation](https://swiftpackageindex.com/apple/swift-syntax/documentation)
- [WWDC23: Write Swift macros](https://developer.apple.com/videos/play/wwdc2023/10166/)
- [WWDC23: Expand on Swift macros](https://developer.apple.com/videos/play/wwdc2023/10167/)

---

*บทต่อไป: Part 96 - SwiftData (iOS 17+)*
