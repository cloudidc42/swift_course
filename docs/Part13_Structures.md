# Part 13: Structures (โครงสร้าง) ใน Swift

## สารบัญ
1. [Structures คืออะไร?](#1-structures-คืออะไร)
2. [Struct Definition Syntax](#2-struct-definition-syntax)
3. [Struct Properties](#3-struct-properties)
4. [Struct Methods](#4-struct-methods)
5. [Struct Initializers](#5-struct-initializers)
6. [Struct as Value Types](#6-struct-as-value-types)
7. [Mutating Methods](#7-mutating-methods)
8. [Static Properties and Methods](#8-static-properties-and-methods)
9. [Struct และ Protocols](#9-struct-และ-protocols)
10. [Copy-on-Write Semantics](#10-copy-on-write-semantics)
11. [Property Observers](#11-property-observers)
12. [Nested Types](#12-nested-types)
13. [Subscripts](#13-subscripts)
14. [Type Properties](#14-type-properties)
15. [Self vs self](#15-self-vs-self)
16. [Struct with Generics](#16-struct-with-generics)
17. [Struct vs Class Comparison](#17-struct-vs-class-comparison)
18. [Real-World Examples](#18-real-world-examples)
19. [Practical Exercises](#19-practical-exercises)
20. [Summary](#20-summary)

---

## 1. Structures คืออะไร?

**Structures** (หรือ `struct`) คือ data type พื้นฐานใน Swift ที่ใช้จัดกลุ่มข้อมูลและพฤติกรรมที่เกี่ยวข้องกันไว้ด้วยกัน Struct เป็น **value type** ซึ่งหมายความว่าเมื่อมีการกำหนดค่าหรือส่งผ่านไปยังฟังก์ชัน จะมีการ **copy** ข้อมูลทั้งหมด

### ความสำคัญของ Struct ใน Swift

Swift ถูกออกแบบมาให้ใช้ Struct เป็นหลักมากกว่า Class เนื่องจาก:
- **ปลอดภัยกว่า**: ไม่มีปัญหา shared state
- **คาดเดาได้ง่ายกว่า**: พฤติกรรมแบบ value type ทำให้โค้ดเข้าใจง่าย
- **ประสิทธิภาพดีกว่า**: ในหลายกรณี value type ทำงานได้เร็วกว่า reference type
- **Thread-safe**: เพราะแต่ละ thread ได้รับ copy ของตัวเอง

### ตัวอย่างแรก: Struct พื้นฐาน

```swift
// การสร้าง Struct ง่ายๆ
struct Point {
    var x: Double
    var y: Double
}

// การสร้าง instance
let origin = Point(x: 0, y: 0)
let position = Point(x: 3.0, y: 4.0)

print("Origin: (\(origin.x), \(origin.y))")     // Origin: (0.0, 0.0)
print("Position: (\(position.x), \(position.y))") // Position: (3.0, 4.0)
```

### Struct ในชีวิตประจำวัน

ใน Swift Standard Library เองก็ใช้ Struct อย่างแพร่หลาย:
- `Int`, `Double`, `Float`, `Bool` - ทุก numeric types เป็น Struct
- `String` - เป็น Struct
- `Array`, `Dictionary`, `Set` - collections ทั้งหมดเป็น Struct
- `CGPoint`, `CGRect`, `CGSize` - ใน UIKit/SwiftUI ก็เป็น Struct

```swift
// ตัวอย่าง: String เป็น Struct
var greeting = "Hello"
var anotherGreeting = greeting  // copy!
anotherGreeting = "Hi"

print(greeting)         // "Hello" - ไม่เปลี่ยน
print(anotherGreeting)  // "Hi"
```

---

## 2. Struct Definition Syntax

### รูปแบบพื้นฐาน

```swift
struct StructName {
    // Properties
    var propertyName: Type
    
    // Methods
    func methodName() {
        // implementation
    }
    
    // Initializers
    init(parameter: Type) {
        // setup
    }
}
```

### ตัวอย่างการสร้าง Struct หลายรูปแบบ

```swift
// Struct ว่างเปล่า (ไม่ค่อยได้ใช้ แต่ valid)
struct EmptyStruct {}

// Struct ที่มีแค่ properties
struct Color {
    var red: Double
    var green: Double
    var blue: Double
    var alpha: Double
}

// Struct ที่มี properties พร้อม default values
struct Configuration {
    var timeout: Double = 30.0
    var maxRetries: Int = 3
    var isDebugMode: Bool = false
    var baseURL: String = "https://api.example.com"
}

// สร้าง instance ได้หลายแบบ
let defaultConfig = Configuration()
print(defaultConfig.timeout)  // 30.0

let customConfig = Configuration(
    timeout: 60.0,
    maxRetries: 5,
    isDebugMode: true,
    baseURL: "https://staging.example.com"
)
print(customConfig.timeout)  // 60.0
```

### Naming Conventions

```swift
// ✅ ถูกต้อง: ชื่อ Struct ขึ้นต้นด้วยตัวพิมพ์ใหญ่ (PascalCase)
struct BankAccount {}
struct UserProfile {}
struct NetworkRequest {}

// ✅ ถูกต้อง: properties และ methods ขึ้นต้นด้วยตัวพิมพ์เล็ก (camelCase)
struct Person {
    var firstName: String
    var lastName: String
    
    func getFullName() -> String {
        return "\(firstName) \(lastName)"
    }
}
```

### Multiple Properties with Same Type

```swift
struct Vector3D {
    var x, y, z: Double  // สามารถประกาศพร้อมกันได้
    
    init(_ x: Double, _ y: Double, _ z: Double) {
        self.x = x
        self.y = y
        self.z = z
    }
}

let v = Vector3D(1.0, 2.0, 3.0)
print("Vector: (\(v.x), \(v.y), \(v.z))")  // Vector: (1.0, 2.0, 3.0)
```

---

## 3. Struct Properties

### 3.1 Stored Properties

**Stored properties** คือ properties ที่เก็บค่าจริงๆ ใน instance

```swift
struct Rectangle {
    // Stored properties
    var width: Double   // สามารถเปลี่ยนค่าได้
    var height: Double  // สามารถเปลี่ยนค่าได้
    let id: String      // ค่าคงที่ ไม่สามารถเปลี่ยนได้
    
    // Optional stored property
    var label: String?
}

var rect = Rectangle(width: 100.0, height: 50.0, id: "rect-001")
rect.width = 200.0  // ✅ ได้เพราะเป็น var
// rect.id = "rect-002"  // ❌ Error: ไม่สามารถเปลี่ยนค่า let ได้

rect.label = "Main Rectangle"
print(rect.label ?? "No label")  // Main Rectangle
```

### 3.2 Computed Properties

**Computed properties** ไม่ได้เก็บค่า แต่คำนวณค่าจาก properties อื่น

```swift
struct Rectangle {
    var width: Double
    var height: Double
    
    // Computed property (read-only)
    var area: Double {
        return width * height
    }
    
    // Computed property (read-write)
    var perimeter: Double {
        get {
            return 2 * (width + height)
        }
        set {
            // สมมติว่า width:height = 2:1 เสมอ
            let ratio = width / height
            height = newValue / (2 * (ratio + 1))
            width = height * ratio
        }
    }
    
    // Shorthand getter (ไม่ต้องเขียน return เมื่อมีแค่บรรทัดเดียว)
    var isSquare: Bool {
        width == height
    }
    
    // Computed property แบบ read-only (shorthand)
    var diagonal: Double {
        (width * width + height * height).squareRoot()
    }
}

var rect = Rectangle(width: 10.0, height: 5.0)
print("Area: \(rect.area)")           // Area: 50.0
print("Perimeter: \(rect.perimeter)") // Perimeter: 30.0
print("Is Square: \(rect.isSquare)")  // Is Square: false
print("Diagonal: \(rect.diagonal)")   // Diagonal: 11.180339...

// การใช้ setter
rect.perimeter = 60.0
print("New width: \(rect.width)")   // New width: 20.0
print("New height: \(rect.height)") // New height: 10.0
```

### 3.3 Lazy Stored Properties

**Lazy properties** จะไม่ถูก initialize จนกว่าจะถูกเข้าถึงครั้งแรก ใช้สำหรับการสร้าง object ที่มีราคาแพง

```swift
struct DataProcessor {
    var dataSource: String
    
    // lazy property: จะสร้าง array ก็ต่อเมื่อถูกเรียกใช้ครั้งแรก
    lazy var processedData: [String] = {
        print("Processing data...")
        // จำลองการประมวลผลที่ใช้เวลานาน
        return dataSource.components(separatedBy: ",")
    }()
}

var processor = DataProcessor(dataSource: "apple,banana,cherry")
print("Processor created")   // Processor created (ยังไม่ process)
print(processor.processedData) // Processing data... จากนั้นพิมพ์ array

// หมายเหตุ: lazy ต้องเป็น var เสมอ เพราะค่าเริ่มต้นเปลี่ยนได้
```

### 3.4 Property Wrappers

Property wrappers ใน Swift ช่วยให้เราเพิ่ม logic ซ้ำๆ ให้ properties

```swift
// สร้าง property wrapper
@propertyWrapper
struct Clamped<T: Comparable> {
    private var value: T
    let min: T
    let max: T
    
    var wrappedValue: T {
        get { value }
        set { value = Swift.min(Swift.max(newValue, min), max) }
    }
    
    init(wrappedValue: T, min: T, max: T) {
        self.min = min
        self.max = max
        self.value = Swift.min(Swift.max(wrappedValue, min), max)
    }
}

struct Volume {
    @Clamped(min: 0, max: 100)
    var level: Int = 50
}

var v = Volume()
print(v.level)  // 50

v.level = 150   // จะถูก clamp เป็น 100
print(v.level)  // 100

v.level = -10   // จะถูก clamp เป็น 0
print(v.level)  // 0
```

---

## 4. Struct Methods

### 4.1 Instance Methods

```swift
struct Circle {
    var radius: Double
    
    // Instance method
    func area() -> Double {
        return Double.pi * radius * radius
    }
    
    func circumference() -> Double {
        return 2 * Double.pi * radius
    }
    
    // Method ที่รับ parameter
    func scale(by factor: Double) -> Circle {
        return Circle(radius: radius * factor)
    }
    
    // Method ที่ return ตัวเองสำหรับ method chaining
    func description() -> String {
        return "Circle with radius \(radius), area: \(String(format: "%.2f", area()))"
    }
}

let circle = Circle(radius: 5.0)
print(circle.area())           // 78.539...
print(circle.circumference())  // 31.415...
print(circle.description())    // Circle with radius 5.0, area: 78.54

let biggerCircle = circle.scale(by: 2.0)
print(biggerCircle.radius)     // 10.0
```

### 4.2 Methods with Multiple Parameters

```swift
struct Temperature {
    var celsius: Double
    
    init(celsius: Double) {
        self.celsius = celsius
    }
    
    init(fahrenheit: Double) {
        self.celsius = (fahrenheit - 32) * 5/9
    }
    
    init(kelvin: Double) {
        self.celsius = kelvin - 273.15
    }
    
    // Methods แปลงหน่วย
    func toFahrenheit() -> Double {
        return celsius * 9/5 + 32
    }
    
    func toKelvin() -> Double {
        return celsius + 273.15
    }
    
    func isFreezing() -> Bool {
        return celsius <= 0
    }
    
    func isBoiling() -> Bool {
        return celsius >= 100
    }
    
    // Method ที่มีหลาย parameter
    func difference(from other: Temperature) -> Double {
        return abs(celsius - other.celsius)
    }
    
    func isWarmerThan(_ other: Temperature) -> Bool {
        return celsius > other.celsius
    }
}

let bodyTemp = Temperature(celsius: 37.0)
let boiling = Temperature(fahrenheit: 212.0)
let absoluteZero = Temperature(kelvin: 0)

print("Body temp in °F: \(bodyTemp.toFahrenheit())")  // 98.6
print("Boiling in °C: \(boiling.celsius)")              // 100.0
print("Absolute zero: \(absoluteZero.celsius)°C")       // -273.15
print("Difference: \(bodyTemp.difference(from: boiling))°C")  // 63.0
print("Body warmer than absolute zero: \(bodyTemp.isWarmerThan(absoluteZero))")  // true
```

### 4.3 Nested Method Calls

```swift
struct StringProcessor {
    var text: String
    
    func reversed() -> StringProcessor {
        return StringProcessor(text: String(text.reversed()))
    }
    
    func uppercased() -> StringProcessor {
        return StringProcessor(text: text.uppercased())
    }
    
    func trimmed() -> StringProcessor {
        return StringProcessor(text: text.trimmingCharacters(in: .whitespaces))
    }
    
    func result() -> String {
        return text
    }
}

let processed = StringProcessor(text: "  hello world  ")
    .trimmed()
    .reversed()
    .uppercased()
    .result()

print(processed)  // DLROW OLLEH
```

---

## 5. Struct Initializers

### 5.1 Memberwise Initializer (อัตโนมัติ)

Swift สร้าง memberwise initializer ให้อัตโนมัติเมื่อไม่ได้กำหนด custom initializer

```swift
struct Book {
    var title: String
    var author: String
    var pages: Int
    var price: Double
}

// Swift สร้าง memberwise initializer ให้อัตโนมัติ:
// init(title: String, author: String, pages: Int, price: Double)
let swiftBook = Book(
    title: "Swift Programming",
    author: "Apple Inc.",
    pages: 500,
    price: 39.99
)

print(swiftBook.title)   // Swift Programming
print(swiftBook.author)  // Apple Inc.
```

### 5.2 Custom Initializers

```swift
struct Point {
    var x: Double
    var y: Double
    
    // Custom initializer แบบที่ 1: ตั้งค่าเริ่มต้น
    init() {
        x = 0.0
        y = 0.0
    }
    
    // Custom initializer แบบที่ 2: รับ parameter
    init(x: Double, y: Double) {
        self.x = x
        self.y = y
    }
    
    // Custom initializer แบบที่ 3: polar coordinates
    init(distance: Double, angle: Double) {
        x = distance * cos(angle)
        y = distance * sin(angle)
    }
    
    // Custom initializer แบบที่ 4: จาก string
    init?(string: String) {
        let components = string.components(separatedBy: ",")
        guard components.count == 2,
              let x = Double(components[0].trimmingCharacters(in: .whitespaces)),
              let y = Double(components[1].trimmingCharacters(in: .whitespaces))
        else {
            return nil  // Failable initializer
        }
        self.x = x
        self.y = y
    }
}

let origin = Point()
print("Origin: (\(origin.x), \(origin.y))")  // Origin: (0.0, 0.0)

let point1 = Point(x: 3.0, y: 4.0)
print("Point1: (\(point1.x), \(point1.y))")  // Point1: (3.0, 4.0)

let point2 = Point(distance: 5.0, angle: Double.pi / 4)
print("Point2: (\(String(format: "%.2f", point2.x)), \(String(format: "%.2f", point2.y)))")
// Point2: (3.54, 3.54)

if let point3 = Point(string: "10.5, 20.3") {
    print("Point3: (\(point3.x), \(point3.y))")  // Point3: (10.5, 20.3)
}

if let invalid = Point(string: "invalid") {
    print("Created: \(invalid)")
} else {
    print("Failed to create point")  // Failed to create point
}
```

### 5.3 Initializer Delegation

Initializer หนึ่งสามารถเรียกใช้ initializer อื่นในตัวเดียวกัน

```swift
struct Size {
    var width: Double
    var height: Double
    
    init(width: Double, height: Double) {
        self.width = width
        self.height = height
    }
    
    // Delegate ไปยัง initializer หลัก
    init(square side: Double) {
        self.init(width: side, height: side)
    }
    
    // Delegate อีกแบบ
    init(doubled size: Size) {
        self.init(width: size.width * 2, height: size.height * 2)
    }
}

let size1 = Size(width: 100, height: 50)
let square = Size(square: 75)
let doubled = Size(doubled: size1)

print("size1: \(size1.width) x \(size1.height)")   // 100.0 x 50.0
print("square: \(square.width) x \(square.height)") // 75.0 x 75.0
print("doubled: \(doubled.width) x \(doubled.height)") // 200.0 x 100.0
```

### 5.4 Failable Initializers

```swift
struct Percentage {
    let value: Double
    
    // Failable initializer: return nil ถ้าค่าไม่ valid
    init?(_ value: Double) {
        guard value >= 0 && value <= 100 else {
            return nil
        }
        self.value = value
    }
    
    var formatted: String {
        return String(format: "%.1f%%", value)
    }
}

if let grade = Percentage(85.5) {
    print("Grade: \(grade.formatted)")  // Grade: 85.5%
}

if let invalid = Percentage(150) {
    print("Created: \(invalid)")
} else {
    print("Invalid percentage")  // Invalid percentage
}

// ใช้กับ guard
func processGrade(_ value: Double) {
    guard let percentage = Percentage(value) else {
        print("Error: \(value) is not a valid percentage")
        return
    }
    print("Processing grade: \(percentage.formatted)")
}

processGrade(75.0)   // Processing grade: 75.0%
processGrade(-5.0)   // Error: -5.0 is not a valid percentage
```

---

## 6. Struct as Value Types

### ความหมายของ Value Type

เมื่อ assign Struct ให้กับตัวแปรใหม่หรือส่งเป็น parameter ระบบจะทำการ **copy** ข้อมูลทั้งหมด

```swift
struct Player {
    var name: String
    var score: Int
    var level: Int
}

var player1 = Player(name: "Alice", score: 100, level: 5)
var player2 = player1  // Copy!

// การเปลี่ยน player2 ไม่กระทบ player1
player2.name = "Bob"
player2.score = 200

print("player1: \(player1.name), score: \(player1.score)")  // Alice, 100
print("player2: \(player2.name), score: \(player2.score)")  // Bob, 200
```

### Value Type ใน Functions

```swift
struct Counter {
    var count: Int = 0
    
    mutating func increment() {
        count += 1
    }
}

func processCounter(_ counter: Counter) {
    // counter เป็น copy ของ original
    // ไม่สามารถ mutate ได้โดยตรง (เป็น let)
    print("Processing counter with value: \(counter.count)")
}

var myCounter = Counter()
myCounter.increment()
myCounter.increment()
print("Before: \(myCounter.count)")  // 2

processCounter(myCounter)  // Processing counter with value: 2

print("After: \(myCounter.count)")   // 2 (ไม่เปลี่ยน)
```

### Value Type กับ Collections

```swift
struct Product {
    var name: String
    var price: Double
    var quantity: Int
}

var inventory = [
    Product(name: "Apple", price: 0.5, quantity: 100),
    Product(name: "Banana", price: 0.3, quantity: 150),
    Product(name: "Cherry", price: 2.0, quantity: 50)
]

// Copy ของ array
var backupInventory = inventory

// เปลี่ยน inventory
inventory[0].price = 0.6
inventory[0].quantity = 90

// backupInventory ไม่เปลี่ยน
print("Current Apple price: \(inventory[0].price)")       // 0.6
print("Backup Apple price: \(backupInventory[0].price)")  // 0.5
```

### ความแตกต่างกับ Reference Type

```swift
// Value Type (Struct)
struct ValuePoint {
    var x: Int
    var y: Int
}

// Reference Type (Class) - จะอธิบายในบท Class
class ReferencePoint {
    var x: Int
    var y: Int
    init(x: Int, y: Int) {
        self.x = x
        self.y = y
    }
}

// Value Type: copy
var vp1 = ValuePoint(x: 1, y: 2)
var vp2 = vp1
vp2.x = 10
print("vp1.x = \(vp1.x)")  // 1 (ไม่เปลี่ยน)
print("vp2.x = \(vp2.x)")  // 10

// Reference Type: shared reference
var rp1 = ReferencePoint(x: 1, y: 2)
var rp2 = rp1
rp2.x = 10
print("rp1.x = \(rp1.x)")  // 10 (เปลี่ยนด้วย!)
print("rp2.x = \(rp2.x)")  // 10
```

---

## 7. Mutating Methods

### ทำไมต้องมี `mutating`?

เนื่องจาก Struct เป็น value type โดยค่าเริ่มต้น methods ไม่สามารถเปลี่ยน properties ได้ ต้องเพิ่ม `mutating` keyword

```swift
struct Stack<T> {
    private var items: [T] = []
    
    // ❌ Error: ไม่มี mutating
    // func push(_ item: T) {
    //     items.append(item)  // Cannot assign to property: 'self' is immutable
    // }
    
    // ✅ ถูกต้อง: มี mutating
    mutating func push(_ item: T) {
        items.append(item)
    }
    
    mutating func pop() -> T? {
        return items.popLast()
    }
    
    // Non-mutating methods ไม่ต้องมี mutating
    func peek() -> T? {
        return items.last
    }
    
    var isEmpty: Bool {
        return items.isEmpty
    }
    
    var count: Int {
        return items.count
    }
}

var stack = Stack<Int>()
stack.push(1)
stack.push(2)
stack.push(3)

print("Count: \(stack.count)")   // 3
print("Peek: \(stack.peek()!)")  // 3

let popped = stack.pop()
print("Popped: \(popped!)")      // 3
print("Count: \(stack.count)")   // 2
```

### Mutating กับ let instance

```swift
struct Counter {
    var value: Int = 0
    
    mutating func increment() {
        value += 1
    }
    
    mutating func reset() {
        value = 0
    }
}

var mutableCounter = Counter()
mutableCounter.increment()
mutableCounter.increment()
print(mutableCounter.value)  // 2

let immutableCounter = Counter()
// immutableCounter.increment()  // ❌ Error: cannot use mutating member on immutable value
```

### Mutating กับ self

```swift
struct Compass {
    enum Direction { case north, south, east, west }
    
    var direction: Direction = .north
    
    mutating func turnRight() {
        switch direction {
        case .north: direction = .east
        case .east:  direction = .south
        case .south: direction = .west
        case .west:  direction = .north
        }
    }
    
    mutating func turnLeft() {
        switch direction {
        case .north: direction = .west
        case .west:  direction = .south
        case .south: direction = .east
        case .east:  direction = .north
        }
    }
    
    // Mutating method สามารถ assign ให้ self ทั้งหมดได้
    mutating func reset() {
        self = Compass()
    }
}

var compass = Compass()
print(compass.direction)  // north

compass.turnRight()
print(compass.direction)  // east

compass.turnRight()
print(compass.direction)  // south

compass.reset()
print(compass.direction)  // north
```

---

## 8. Static Properties and Methods

### Type Properties (static)

**Static properties** เป็นของ Type ไม่ใช่ instance

```swift
struct MathConstants {
    static let pi = 3.14159265358979
    static let e = 2.71828182845905
    static let goldenRatio = 1.61803398874989
    
    static func circleArea(radius: Double) -> Double {
        return pi * radius * radius
    }
    
    static func sphereVolume(radius: Double) -> Double {
        return (4.0/3.0) * pi * radius * radius * radius
    }
}

// ใช้ผ่าน type name ไม่ใช่ instance
print(MathConstants.pi)                           // 3.14159...
print(MathConstants.circleArea(radius: 5))        // 78.539...
print(MathConstants.sphereVolume(radius: 3))      // 113.097...
```

### Static กับ Singleton Pattern

```swift
struct AppConfiguration {
    // Shared instance (Singleton)
    static let shared = AppConfiguration(
        environment: "production",
        apiURL: "https://api.example.com",
        timeout: 30
    )
    
    // Development instance
    static let development = AppConfiguration(
        environment: "development",
        apiURL: "https://dev.api.example.com",
        timeout: 60
    )
    
    let environment: String
    let apiURL: String
    let timeout: Int
    
    var isProduction: Bool {
        return environment == "production"
    }
}

// ใช้ shared instance
let config = AppConfiguration.shared
print("URL: \(config.apiURL)")
print("Is Production: \(config.isProduction)")

// ใช้ development
let devConfig = AppConfiguration.development
print("Dev URL: \(devConfig.apiURL)")
```

### Static Variable (ค่าเปลี่ยนได้)

```swift
struct Counter {
    // Static variable: นับ instance ทั้งหมดที่สร้าง
    static var instanceCount = 0
    
    let id: Int
    
    init() {
        Counter.instanceCount += 1
        self.id = Counter.instanceCount
    }
    
    static func resetCount() {
        instanceCount = 0
    }
}

let c1 = Counter()
let c2 = Counter()
let c3 = Counter()

print("Total instances: \(Counter.instanceCount)")  // 3
print("c1 id: \(c1.id)")  // 1
print("c2 id: \(c2.id)")  // 2
print("c3 id: \(c3.id)")  // 3

Counter.resetCount()
print("After reset: \(Counter.instanceCount)")  // 0
```

### Computed Static Properties

```swift
struct DeviceInfo {
    static var currentTime: Date {
        return Date()
    }
    
    static var isWeekend: Bool {
        let calendar = Calendar.current
        let weekday = calendar.component(.weekday, from: Date())
        return weekday == 1 || weekday == 7
    }
    
    static var systemInfo: String {
        return "Swift \(swiftVersion) on \(platform)"
    }
    
    private static let swiftVersion = "5.9"
    private static let platform = "macOS/iOS"
}

print("Current time: \(DeviceInfo.currentTime)")
print("Is weekend: \(DeviceInfo.isWeekend)")
print("System: \(DeviceInfo.systemInfo)")
```

---

## 9. Struct และ Protocols

### Struct ไม่มี Inheritance แต่ใช้ Protocols แทน

```swift
// Protocol กำหนด interface
protocol Drawable {
    func draw()
    var color: String { get }
}

protocol Resizable {
    mutating func resize(by factor: Double)
}

protocol Describable {
    var description: String { get }
}

// Struct conform to protocols
struct Circle: Drawable, Resizable, Describable {
    var radius: Double
    var color: String
    
    func draw() {
        print("Drawing \(color) circle with radius \(radius)")
    }
    
    mutating func resize(by factor: Double) {
        radius *= factor
    }
    
    var description: String {
        return "\(color) Circle (radius: \(radius))"
    }
}

struct Square: Drawable, Resizable, Describable {
    var side: Double
    var color: String
    
    func draw() {
        print("Drawing \(color) square with side \(side)")
    }
    
    mutating func resize(by factor: Double) {
        side *= factor
    }
    
    var description: String {
        return "\(color) Square (side: \(side))"
    }
}

// ใช้ polymorphism ผ่าน protocol
var shapes: [Drawable] = [
    Circle(radius: 5.0, color: "Red"),
    Square(side: 10.0, color: "Blue"),
    Circle(radius: 3.0, color: "Green")
]

for shape in shapes {
    shape.draw()
}
// Drawing Red circle with radius 5.0
// Drawing Blue square with side 10.0
// Drawing Green circle with radius 3.0
```

### Protocol Extensions กับ Struct

```swift
protocol Greetable {
    var name: String { get }
}

// Extension เพิ่ม default implementation
extension Greetable {
    func greet() -> String {
        return "Hello, I'm \(name)!"
    }
    
    func formalGreet() -> String {
        return "Good day. My name is \(name)."
    }
}

struct Person: Greetable {
    var name: String
    var age: Int
}

struct Robot: Greetable {
    var name: String
    var model: String
    
    // Override default implementation
    func greet() -> String {
        return "BEEP BOOP. I AM \(name.uppercased())."
    }
}

let person = Person(name: "Alice", age: 30)
let robot = Robot(name: "R2D2", model: "Astromech")

print(person.greet())       // Hello, I'm Alice!
print(person.formalGreet()) // Good day. My name is Alice.
print(robot.greet())        // BEEP BOOP. I AM R2D2.
print(robot.formalGreet())  // Good day. My name is R2D2.
```

### Struct กับ Codable Protocol

```swift
struct UserProfile: Codable {
    var id: Int
    var username: String
    var email: String
    var createdAt: Date
    
    // Custom coding keys
    enum CodingKeys: String, CodingKey {
        case id
        case username
        case email
        case createdAt = "created_at"
    }
}

// Encode เป็น JSON
let user = UserProfile(
    id: 1,
    username: "swift_lover",
    email: "swift@example.com",
    createdAt: Date()
)

let encoder = JSONEncoder()
encoder.dateEncodingStrategy = .iso8601
encoder.outputFormatting = .prettyPrinted

if let jsonData = try? encoder.encode(user),
   let jsonString = String(data: jsonData, encoding: .utf8) {
    print(jsonString)
    // {
    //   "id" : 1,
    //   "username" : "swift_lover",
    //   "email" : "swift@example.com",
    //   "created_at" : "2024-01-15T10:30:00Z"
    // }
}

// Decode จาก JSON
let jsonString = """
{
    "id": 2,
    "username": "code_master",
    "email": "master@example.com",
    "created_at": "2024-01-15T10:30:00Z"
}
"""

let decoder = JSONDecoder()
decoder.dateDecodingStrategy = .iso8601

if let jsonData = jsonString.data(using: .utf8),
   let decoded = try? decoder.decode(UserProfile.self, from: jsonData) {
    print("Decoded user: \(decoded.username)")  // code_master
}
```

---

## 10. Copy-on-Write Semantics

### ความหมายของ Copy-on-Write (CoW)

**Copy-on-Write** เป็น optimization technique ที่ Swift ใช้กับ collections เช่น Array, Dictionary, String โดย copy จะเกิดขึ้นจริงๆ ก็ต่อเมื่อมีการแก้ไขข้อมูล

```swift
// ตัวอย่าง Copy-on-Write กับ Array
var array1 = [1, 2, 3, 4, 5]
var array2 = array1  // ยังไม่ copy จริงๆ (share storage)

// Copy เกิดขึ้นเมื่อมีการแก้ไข
array2.append(6)  // ตอนนี้จึง copy

print(array1)  // [1, 2, 3, 4, 5]
print(array2)  // [1, 2, 3, 4, 5, 6]
```

### สร้าง Custom Type ที่มี Copy-on-Write

```swift
// Custom Reference Type สำหรับ CoW
final class DataStorage {
    var data: [Int]
    
    init(_ data: [Int]) {
        self.data = data
    }
    
    // Copy constructor
    init(copying other: DataStorage) {
        self.data = other.data
    }
}

// Value Type ที่ใช้ CoW
struct EfficientArray {
    private var storage: DataStorage
    
    init(_ data: [Int] = []) {
        storage = DataStorage(data)
    }
    
    // CoW helper: สร้าง unique copy ถ้าจำเป็น
    private mutating func makeUniqueStorage() {
        if !isKnownUniquelyReferenced(&storage) {
            storage = DataStorage(copying: storage)
            print("Copied storage!")
        }
    }
    
    // Read operation: ไม่ต้อง copy
    var count: Int {
        return storage.data.count
    }
    
    subscript(index: Int) -> Int {
        return storage.data[index]
    }
    
    // Write operation: copy ถ้าจำเป็น
    mutating func append(_ value: Int) {
        makeUniqueStorage()
        storage.data.append(value)
    }
    
    mutating func remove(at index: Int) {
        makeUniqueStorage()
        storage.data.remove(at: index)
    }
}

var arr1 = EfficientArray([1, 2, 3])
var arr2 = arr1  // Share storage, no copy yet

print("arr1[0]: \(arr1[0])")  // 1 (no copy)
print("arr2[0]: \(arr2[0])")  // 1 (no copy)

arr2.append(4)  // Copied storage! (copy happens here)
print("arr1 count: \(arr1.count)")  // 3
print("arr2 count: \(arr2.count)")  // 4
```

---

## 11. Property Observers

### willSet และ didSet

**Property observers** ให้เราตอบสนองต่อการเปลี่ยนแปลงค่าของ property

```swift
struct StepCounter {
    var totalSteps: Int = 0 {
        willSet(newSteps) {
            print("กำลังจะอัพเดทเป็น \(newSteps) ก้าว")
        }
        didSet {
            if totalSteps > oldValue {
                let added = totalSteps - oldValue
                print("เพิ่มมา \(added) ก้าว! รวม \(totalSteps) ก้าว")
            } else if totalSteps < oldValue {
                print("ลดลงจาก \(oldValue) เป็น \(totalSteps) ก้าว")
            }
        }
    }
    
    var dailyGoal: Int = 10000 {
        didSet {
            print("เปลี่ยน daily goal เป็น \(dailyGoal) ก้าว")
        }
    }
    
    var progress: Double {
        return Double(totalSteps) / Double(dailyGoal) * 100
    }
}

var counter = StepCounter()
counter.totalSteps = 200
// กำลังจะอัพเดทเป็น 200 ก้าว
// เพิ่มมา 200 ก้าว! รวม 200 ก้าว

counter.totalSteps = 1000
// กำลังจะอัพเดทเป็น 1000 ก้าว
// เพิ่มมา 800 ก้าว! รวม 1000 ก้าว

print("Progress: \(counter.progress)%")  // Progress: 10.0%
```

### Property Observers ใน Struct ที่ซับซ้อน

```swift
struct NetworkMonitor {
    enum ConnectionStatus {
        case connected, disconnected, connecting
    }
    
    var status: ConnectionStatus = .disconnected {
        willSet {
            print("Status changing from \(status) to \(newValue)")
        }
        didSet {
            switch status {
            case .connected:
                onConnect?()
            case .disconnected:
                onDisconnect?()
            case .connecting:
                break
            }
        }
    }
    
    var signalStrength: Int = 0 {
        didSet {
            signalStrength = max(0, min(100, signalStrength))  // Clamp 0-100
            
            if signalStrength < 20 && oldValue >= 20 {
                print("Warning: Low signal strength!")
            }
        }
    }
    
    var onConnect: (() -> Void)?
    var onDisconnect: (() -> Void)?
}

var monitor = NetworkMonitor()
monitor.onConnect = { print("Connected! Ready to use.") }
monitor.onDisconnect = { print("Disconnected! Check your network.") }

monitor.status = .connecting  // Status changing from disconnected to connecting
monitor.status = .connected   // Status changing from connecting to connected
                               // Connected! Ready to use.

monitor.signalStrength = 150  // จะถูก clamp เป็น 100
print("Signal: \(monitor.signalStrength)")  // 100

monitor.signalStrength = 10   // Warning: Low signal strength!
```

---

## 12. Nested Types

### Struct ภายใน Struct

```swift
struct Card {
    // Nested enum
    enum Suit: String, CaseIterable {
        case hearts = "♥"
        case diamonds = "♦"
        case clubs = "♣"
        case spades = "♠"
    }
    
    // Nested enum อีกอัน
    enum Rank: Int, CaseIterable {
        case two = 2, three, four, five, six, seven, eight, nine, ten
        case jack = 11, queen, king, ace = 14
        
        var name: String {
            switch self {
            case .jack: return "Jack"
            case .queen: return "Queen"
            case .king: return "King"
            case .ace: return "Ace"
            default: return "\(rawValue)"
            }
        }
    }
    
    // Properties
    let rank: Rank
    let suit: Suit
    
    var description: String {
        return "\(rank.name) of \(suit.rawValue)"
    }
    
    var pointValue: Int {
        return rank.rawValue
    }
    
    // Nested struct
    struct Deck {
        var cards: [Card] = []
        
        static func standardDeck() -> Deck {
            var deck = Deck()
            for suit in Suit.allCases {
                for rank in Rank.allCases {
                    deck.cards.append(Card(rank: rank, suit: suit))
                }
            }
            return deck
        }
        
        mutating func shuffle() {
            cards.shuffle()
        }
        
        mutating func deal() -> Card? {
            return cards.isEmpty ? nil : cards.removeLast()
        }
    }
}

// ใช้งาน
let aceOfSpades = Card(rank: .ace, suit: .spades)
print(aceOfSpades.description)  // Ace of ♠

var deck = Card.Deck.standardDeck()
print("Cards in deck: \(deck.cards.count)")  // 52

deck.shuffle()
if let firstCard = deck.deal() {
    print("Dealt: \(firstCard.description)")
}
```

### Nested Types ในระดับที่ซับซ้อน

```swift
struct Organization {
    struct Department {
        struct Team {
            var name: String
            var members: [String]
            
            var size: Int { members.count }
        }
        
        var name: String
        var teams: [Team]
        
        var totalMembers: Int {
            teams.reduce(0) { $0 + $1.size }
        }
    }
    
    var name: String
    var departments: [Department]
    
    var totalEmployees: Int {
        departments.reduce(0) { $0 + $1.totalMembers }
    }
}

let org = Organization(
    name: "Tech Corp",
    departments: [
        Organization.Department(
            name: "Engineering",
            teams: [
                Organization.Department.Team(name: "iOS", members: ["Alice", "Bob", "Charlie"]),
                Organization.Department.Team(name: "Android", members: ["Dave", "Eve"])
            ]
        ),
        Organization.Department(
            name: "Design",
            teams: [
                Organization.Department.Team(name: "UI/UX", members: ["Frank", "Grace"])
            ]
        )
    ]
)

print("\(org.name) has \(org.totalEmployees) employees")  // Tech Corp has 7 employees
print("Engineering: \(org.departments[0].totalMembers) members")  // Engineering: 5 members
```

---

## 13. Subscripts

### การสร้าง Custom Subscript

```swift
struct Matrix {
    var rows: Int
    var columns: Int
    private var grid: [Double]
    
    init(rows: Int, columns: Int) {
        self.rows = rows
        self.columns = columns
        grid = Array(repeating: 0.0, count: rows * columns)
    }
    
    func indexIsValid(row: Int, column: Int) -> Bool {
        return row >= 0 && row < rows && column >= 0 && column < columns
    }
    
    // Subscript ด้วย 2 parameters
    subscript(row: Int, column: Int) -> Double {
        get {
            assert(indexIsValid(row: row, column: column), "Index out of range")
            return grid[(row * columns) + column]
        }
        set {
            assert(indexIsValid(row: row, column: column), "Index out of range")
            grid[(row * columns) + column] = newValue
        }
    }
    
    // Subscript สำหรับดึงแถวทั้งแถว
    subscript(row row: Int) -> [Double] {
        let startIndex = row * columns
        let endIndex = startIndex + columns
        return Array(grid[startIndex..<endIndex])
    }
}

var matrix = Matrix(rows: 3, columns: 3)
matrix[0, 0] = 1.0
matrix[0, 1] = 2.0
matrix[1, 0] = 3.0
matrix[1, 1] = 4.0
matrix[2, 2] = 9.0

print(matrix[0, 0])  // 1.0
print(matrix[1, 1])  // 4.0
print(matrix[row: 0])  // [1.0, 2.0, 0.0]
```

### Subscripts ที่ซับซ้อน

```swift
struct StringDictionary {
    private var storage: [String: String] = [:]
    
    // Subscript พื้นฐาน
    subscript(key: String) -> String? {
        get { storage[key] }
        set { storage[key] = newValue }
    }
    
    // Subscript พร้อม default value
    subscript(key: String, default defaultValue: String) -> String {
        return storage[key] ?? defaultValue
    }
    
    // Subscript หลาย keys
    subscript(keys: String...) -> [String: String] {
        var result: [String: String] = [:]
        for key in keys {
            result[key] = storage[key]
        }
        return result
    }
}

var dict = StringDictionary()
dict["name"] = "Alice"
dict["age"] = "30"
dict["city"] = "Bangkok"

print(dict["name"] ?? "unknown")           // Alice
print(dict["name", default: "unknown"])    // Alice
print(dict["email", default: "no email"])  // no email

let subset = dict["name", "age"]
print(subset)  // ["name": "Alice", "age": "30"]
```

---

## 14. Type Properties

### Static vs Instance Properties

```swift
struct Circle {
    // Type properties (เป็นของ type ไม่ใช่ instance)
    static let minimumRadius: Double = 0.1
    static var totalCircles: Int = 0
    
    // Instance properties
    var radius: Double
    var color: String
    
    // Computed type property
    static var description: String {
        return "Circle type with \(totalCircles) instances created"
    }
    
    init(radius: Double, color: String = "black") {
        self.radius = max(Circle.minimumRadius, radius)
        self.color = color
        Circle.totalCircles += 1
    }
    
    var area: Double {
        return Double.pi * radius * radius
    }
}

let c1 = Circle(radius: 5.0)
let c2 = Circle(radius: 3.0, color: "red")
let c3 = Circle(radius: -1.0)  // จะ clamp เป็น minimumRadius

print(Circle.totalCircles)   // 3
print(Circle.description)    // Circle type with 3 instances created
print(c3.radius)             // 0.1 (clamp)
```

---

## 15. Self vs self

### `self` (ตัวพิมพ์เล็ก) = instance ปัจจุบัน

```swift
struct Rectangle {
    var width: Double
    var height: Double
    
    init(width: Double, height: Double) {
        self.width = width   // self.width = property, width = parameter
        self.height = height
    }
    
    func scaled(by factor: Double) -> Rectangle {
        // self ใช้อ้างถึง instance ปัจจุบัน
        return Rectangle(width: self.width * factor, height: self.height * factor)
    }
    
    // ปกติไม่ต้องเขียน self ก็ได้ (Swift รู้เองว่าหมายถึง property)
    var area: Double {
        return width * height  // ไม่ต้อง self.width * self.height
    }
}
```

### `Self` (ตัวพิมพ์ใหญ่) = type ปัจจุบัน

```swift
struct Builder {
    var config: [String: Any] = [:]
    
    // Self หมายถึง type "Builder" ตัวเอง
    func with(key: String, value: Any) -> Self {
        var copy = self  // copy instance ปัจจุบัน
        copy.config[key] = value
        return copy  // return ค่าที่เป็น Builder
    }
    
    func build() -> [String: Any] {
        return config
    }
}

let result = Builder()
    .with(key: "name", value: "Alice")
    .with(key: "age", value: 30)
    .with(key: "city", value: "Bangkok")
    .build()

print(result)  // ["name": "Alice", "age": 30, "city": "Bangkok"]
```

### Self ใน Protocol Extensions

```swift
protocol Copyable {
    // Self ใน protocol หมายถึง type ที่ conform
    func copy() -> Self
}

extension Copyable {
    func duplicated() -> [Self] {
        return [self, copy()]
    }
}

struct Configuration: Copyable {
    var settings: [String: String]
    
    func copy() -> Configuration {
        return Configuration(settings: self.settings)
    }
}

let config = Configuration(settings: ["theme": "dark", "language": "th"])
let copies = config.duplicated()
print(copies.count)  // 2
```

---

## 16. Struct with Generics

### Generic Struct พื้นฐาน

```swift
// Generic Pair
struct Pair<First, Second> {
    var first: First
    var second: Second
    
    init(_ first: First, _ second: Second) {
        self.first = first
        self.second = second
    }
    
    func swapped() -> Pair<Second, First> {
        return Pair<Second, First>(second, first)
    }
}

let intAndString = Pair(42, "Hello")
print(intAndString.first)    // 42
print(intAndString.second)   // Hello

let swapped = intAndString.swapped()
print(swapped.first)   // Hello
print(swapped.second)  // 42

// Generic Stack
struct Stack<Element> {
    private var items: [Element] = []
    
    mutating func push(_ item: Element) {
        items.append(item)
    }
    
    @discardableResult
    mutating func pop() -> Element? {
        return items.popLast()
    }
    
    func peek() -> Element? {
        return items.last
    }
    
    var isEmpty: Bool {
        return items.isEmpty
    }
    
    var count: Int {
        return items.count
    }
}

var intStack = Stack<Int>()
intStack.push(1)
intStack.push(2)
intStack.push(3)
print(intStack.peek()!)  // 3
print(intStack.count)    // 3
intStack.pop()
print(intStack.count)    // 2

var stringStack = Stack<String>()
stringStack.push("Hello")
stringStack.push("World")
print(stringStack.peek()!)  // World
```

### Generic Constraints

```swift
// Generic ที่มี constraint
struct SortedArray<T: Comparable> {
    private var items: [T] = []
    
    mutating func insert(_ item: T) {
        if let index = items.firstIndex(where: { $0 > item }) {
            items.insert(item, at: index)
        } else {
            items.append(item)
        }
    }
    
    func contains(_ item: T) -> Bool {
        // Binary search
        var low = 0
        var high = items.count - 1
        
        while low <= high {
            let mid = (low + high) / 2
            if items[mid] == item {
                return true
            } else if items[mid] < item {
                low = mid + 1
            } else {
                high = mid - 1
            }
        }
        return false
    }
    
    var min: T? { items.first }
    var max: T? { items.last }
    
    var sorted: [T] { items }
}

var numbers = SortedArray<Int>()
numbers.insert(5)
numbers.insert(2)
numbers.insert(8)
numbers.insert(1)
numbers.insert(9)

print(numbers.sorted)  // [1, 2, 5, 8, 9]
print("Min: \(numbers.min!), Max: \(numbers.max!)")  // Min: 1, Max: 9
print(numbers.contains(5))  // true
print(numbers.contains(7))  // false
```

### Generic Result Type

```swift
// Generic Result/Either type
enum Result<Success, Failure: Error> {
    case success(Success)
    case failure(Failure)
    
    func map<NewSuccess>(_ transform: (Success) -> NewSuccess) -> Result<NewSuccess, Failure> {
        switch self {
        case .success(let value):
            return .success(transform(value))
        case .failure(let error):
            return .failure(error)
        }
    }
    
    var value: Success? {
        if case .success(let v) = self { return v }
        return nil
    }
}

struct NetworkError: Error {
    var message: String
}

func fetchUser(id: Int) -> Result<String, NetworkError> {
    if id > 0 {
        return .success("User \(id)")
    } else {
        return .failure(NetworkError(message: "Invalid ID"))
    }
}

let result = fetchUser(id: 5)
let upperResult = result.map { $0.uppercased() }

switch upperResult {
case .success(let user):
    print("Got user: \(user)")  // Got user: USER 5
case .failure(let error):
    print("Error: \(error.message)")
}
```

---

## 17. Struct vs Class Comparison

### ตารางเปรียบเทียบ

| Feature | Struct | Class |
|---------|--------|-------|
| Type | Value Type | Reference Type |
| Inheritance | ไม่มี | มี |
| Initializer | Memberwise อัตโนมัติ | ต้องเขียนเอง |
| Mutating | ต้องระบุ `mutating` | ไม่ต้อง |
| Deinitializer | ไม่มี | มี `deinit` |
| ARC | ไม่ใช้ | ใช้ |
| Thread Safety | ปลอดภัย (copy) | ต้องจัดการเอง |

### เมื่อไรใช้ Struct?

```swift
// ✅ ใช้ Struct เมื่อ:

// 1. แทนค่าง่ายๆ (coordinates, sizes, colors)
struct Point { var x, y: Double }
struct Size { var width, height: Double }

// 2. ข้อมูลที่ไม่ต้องการ shared state
struct UserSettings {
    var theme: String
    var language: String
    var notifications: Bool
}

// 3. ข้อมูลที่ copy ไปในแต่ละ scope มีความหมาย
struct GameState {
    var score: Int
    var lives: Int
    var level: Int
}

// เก็บ snapshot ของ game state
func saveCheckpoint(_ state: GameState) {
    // state เป็น copy อิสระ
    print("Saving: score=\(state.score), level=\(state.level)")
}

var game = GameState(score: 0, lives: 3, level: 1)
saveCheckpoint(game)  // save ณ ปัจจุบัน

game.score = 100
game.level = 2
// checkpoint ยังเป็นค่าเดิม
```

### เมื่อไรใช้ Class?

```swift
// ✅ ใช้ Class เมื่อ:

// 1. ต้องการ shared state
class NetworkManager {
    static let shared = NetworkManager()
    var session: URLSession
    
    private init() {
        session = URLSession.shared
    }
    
    func request(url: URL) {
        // ...
    }
}

// 2. ต้องการ inheritance
class Animal {
    var name: String
    init(name: String) { self.name = name }
    func sound() -> String { return "..." }
}

class Dog: Animal {
    override func sound() -> String { return "Woof!" }
}

// 3. ต้องการ deinitializer
class FileHandle {
    let path: String
    
    init(path: String) {
        self.path = path
        print("Opened: \(path)")
    }
    
    deinit {
        print("Closed: \(path)")
    }
}
```

### Struct กับ Performance

```swift
// Value types มักเร็วกว่าสำหรับ small data
struct SmallPoint {
    var x, y: Int  // 16 bytes
}

// Value type ไม่ต้องผ่าน heap allocation
func processPoints(_ points: [SmallPoint]) -> SmallPoint {
    var result = SmallPoint(x: 0, y: 0)
    for point in points {
        result.x += point.x
        result.y += point.y
    }
    return result
}

// การวัด performance จริงๆ ต้องใช้ Instruments
```

---

## 18. Real-World Examples

### 18.1 Point และ Vector

```swift
struct Point {
    var x: Double
    var y: Double
    
    static let zero = Point(x: 0, y: 0)
    
    func distance(to other: Point) -> Double {
        let dx = x - other.x
        let dy = y - other.y
        return (dx * dx + dy * dy).squareRoot()
    }
    
    func midpoint(to other: Point) -> Point {
        return Point(x: (x + other.x) / 2, y: (y + other.y) / 2)
    }
    
    static func + (lhs: Point, rhs: Point) -> Point {
        return Point(x: lhs.x + rhs.x, y: lhs.y + rhs.y)
    }
    
    static func - (lhs: Point, rhs: Point) -> Point {
        return Point(x: lhs.x - rhs.x, y: lhs.y - rhs.y)
    }
    
    static func * (point: Point, scalar: Double) -> Point {
        return Point(x: point.x * scalar, y: point.y * scalar)
    }
}

extension Point: CustomStringConvertible {
    var description: String {
        return "(\(x), \(y))"
    }
}

extension Point: Equatable {}
extension Point: Hashable {}

let a = Point(x: 0, y: 0)
let b = Point(x: 3, y: 4)

print("Distance: \(a.distance(to: b))")  // 5.0
print("Midpoint: \(a.midpoint(to: b))")  // (1.5, 2.0)
print("Sum: \(a + b)")                   // (3.0, 4.0)

// ใช้ใน Set เพราะ Hashable
var pointSet: Set<Point> = [.zero, b, Point(x: 1, y: 1)]
print("Unique points: \(pointSet.count)")  // 3
```

### 18.2 Rectangle

```swift
struct Rectangle {
    var origin: Point
    var size: Size
    
    struct Size {
        var width: Double
        var height: Double
        
        static let zero = Size(width: 0, height: 0)
    }
    
    init(origin: Point = .zero, size: Size) {
        self.origin = origin
        self.size = size
    }
    
    init(x: Double, y: Double, width: Double, height: Double) {
        self.origin = Point(x: x, y: y)
        self.size = Size(width: width, height: height)
    }
    
    var minX: Double { origin.x }
    var minY: Double { origin.y }
    var maxX: Double { origin.x + size.width }
    var maxY: Double { origin.y + size.height }
    var midX: Double { origin.x + size.width / 2 }
    var midY: Double { origin.y + size.height / 2 }
    var center: Point { Point(x: midX, y: midY) }
    
    var area: Double { size.width * size.height }
    var perimeter: Double { 2 * (size.width + size.height) }
    
    func contains(_ point: Point) -> Bool {
        return point.x >= minX && point.x <= maxX &&
               point.y >= minY && point.y <= maxY
    }
    
    func intersects(_ other: Rectangle) -> Bool {
        return minX < other.maxX && maxX > other.minX &&
               minY < other.maxY && maxY > other.minY
    }
    
    func intersection(with other: Rectangle) -> Rectangle? {
        let x = max(minX, other.minX)
        let y = max(minY, other.minY)
        let maxX = min(self.maxX, other.maxX)
        let maxY = min(self.maxY, other.maxY)
        
        guard x < maxX && y < maxY else { return nil }
        
        return Rectangle(x: x, y: y, width: maxX - x, height: maxY - y)
    }
    
    mutating func inset(by amount: Double) {
        origin.x += amount
        origin.y += amount
        size.width -= 2 * amount
        size.height -= 2 * amount
    }
}

extension Rectangle: CustomStringConvertible {
    var description: String {
        return "Rect(\(origin.x), \(origin.y), \(size.width)x\(size.height))"
    }
}

let rect1 = Rectangle(x: 0, y: 0, width: 100, height: 100)
let rect2 = Rectangle(x: 50, y: 50, width: 100, height: 100)

print("Rect1: \(rect1)")
print("Area: \(rect1.area)")
print("Contains (25, 25): \(rect1.contains(Point(x: 25, y: 25)))")
print("Intersects rect2: \(rect1.intersects(rect2))")

if let intersection = rect1.intersection(with: rect2) {
    print("Intersection: \(intersection)")
}
```

### 18.3 Color

```swift
struct Color {
    var red: Double   // 0.0 - 1.0
    var green: Double // 0.0 - 1.0
    var blue: Double  // 0.0 - 1.0
    var alpha: Double // 0.0 - 1.0
    
    static let black = Color(red: 0, green: 0, blue: 0)
    static let white = Color(red: 1, green: 1, blue: 1)
    static let red = Color(red: 1, green: 0, blue: 0)
    static let green = Color(red: 0, green: 1, blue: 0)
    static let blue = Color(red: 0, green: 0, blue: 1)
    static let transparent = Color(red: 0, green: 0, blue: 0, alpha: 0)
    
    init(red: Double, green: Double, blue: Double, alpha: Double = 1.0) {
        self.red = min(1, max(0, red))
        self.green = min(1, max(0, green))
        self.blue = min(1, max(0, blue))
        self.alpha = min(1, max(0, alpha))
    }
    
    // สร้างจาก hex string
    init?(hex: String) {
        var hexString = hex.trimmingCharacters(in: .whitespaces)
        if hexString.hasPrefix("#") {
            hexString = String(hexString.dropFirst())
        }
        
        guard hexString.count == 6,
              let hexValue = UInt32(hexString, radix: 16) else {
            return nil
        }
        
        red = Double((hexValue >> 16) & 0xFF) / 255.0
        green = Double((hexValue >> 8) & 0xFF) / 255.0
        blue = Double(hexValue & 0xFF) / 255.0
        alpha = 1.0
    }
    
    // สร้างจาก HSL
    init(hue: Double, saturation: Double, lightness: Double) {
        if saturation == 0 {
            red = lightness
            green = lightness
            blue = lightness
            alpha = 1.0
            return
        }
        
        let q = lightness < 0.5 ? lightness * (1 + saturation) : lightness + saturation - lightness * saturation
        let p = 2 * lightness - q
        
        func hue2rgb(_ p: Double, _ q: Double, _ t: Double) -> Double {
            var t = t
            if t < 0 { t += 1 }
            if t > 1 { t -= 1 }
            if t < 1/6 { return p + (q - p) * 6 * t }
            if t < 1/2 { return q }
            if t < 2/3 { return p + (q - p) * (2/3 - t) * 6 }
            return p
        }
        
        red = hue2rgb(p, q, hue + 1/3)
        green = hue2rgb(p, q, hue)
        blue = hue2rgb(p, q, hue - 1/3)
        alpha = 1.0
    }
    
    var hex: String {
        let r = Int(red * 255)
        let g = Int(green * 255)
        let b = Int(blue * 255)
        return String(format: "#%02X%02X%02X", r, g, b)
    }
    
    func mixed(with other: Color, ratio: Double = 0.5) -> Color {
        return Color(
            red: red * (1 - ratio) + other.red * ratio,
            green: green * (1 - ratio) + other.green * ratio,
            blue: blue * (1 - ratio) + other.blue * ratio,
            alpha: alpha * (1 - ratio) + other.alpha * ratio
        )
    }
    
    func withAlpha(_ alpha: Double) -> Color {
        return Color(red: red, green: green, blue: blue, alpha: alpha)
    }
    
    var inverted: Color {
        return Color(red: 1 - red, green: 1 - green, blue: 1 - blue, alpha: alpha)
    }
    
    var brightness: Double {
        return 0.299 * red + 0.587 * green + 0.114 * blue
    }
    
    var isDark: Bool {
        return brightness < 0.5
    }
}

extension Color: CustomStringConvertible {
    var description: String {
        return "Color(r:\(String(format: "%.2f", red)), g:\(String(format: "%.2f", green)), b:\(String(format: "%.2f", blue)), a:\(String(format: "%.2f", alpha)))"
    }
}

// ใช้งาน Color
if let swiftOrange = Color(hex: "#FF6B00") {
    print("Swift orange: \(swiftOrange)")
    print("Hex: \(swiftOrange.hex)")
    print("Is dark: \(swiftOrange.isDark)")
}

let purple = Color.red.mixed(with: .blue)
print("Mixed purple: \(purple)")
print("Inverted: \(purple.inverted)")
```

### 18.4 Temperature Converter

```swift
struct Temperature: Comparable, Hashable {
    private let kelvin: Double
    
    static let absoluteZero = Temperature(kelvin: 0)
    static let freezingPoint = Temperature(celsius: 0)
    static let boilingPoint = Temperature(celsius: 100)
    static let bodyTemperature = Temperature(celsius: 37)
    
    init(kelvin: Double) {
        self.kelvin = max(0, kelvin)  // ไม่ต่ำกว่า absolute zero
    }
    
    init(celsius: Double) {
        self.init(kelvin: celsius + 273.15)
    }
    
    init(fahrenheit: Double) {
        self.init(celsius: (fahrenheit - 32) * 5/9)
    }
    
    var celsius: Double { kelvin - 273.15 }
    var fahrenheit: Double { celsius * 9/5 + 32 }
    
    static func < (lhs: Temperature, rhs: Temperature) -> Bool {
        return lhs.kelvin < rhs.kelvin
    }
    
    func hash(into hasher: inout Hasher) {
        hasher.combine(kelvin)
    }
    
    enum Unit {
        case celsius, fahrenheit, kelvin
    }
    
    func value(in unit: Unit) -> Double {
        switch unit {
        case .celsius: return celsius
        case .fahrenheit: return fahrenheit
        case .kelvin: return kelvin
        }
    }
    
    func formatted(unit: Unit = .celsius, decimals: Int = 1) -> String {
        let value = self.value(in: unit)
        let symbol: String
        switch unit {
        case .celsius: symbol = "°C"
        case .fahrenheit: symbol = "°F"
        case .kelvin: symbol = "K"
        }
        return String(format: "%.\(decimals)f%@", value, symbol)
    }
    
    var description: String {
        return "\(formatted(unit: .celsius)) / \(formatted(unit: .fahrenheit)) / \(formatted(unit: .kelvin))"
    }
}

let bodyTemp = Temperature.bodyTemperature
print("Body: \(bodyTemp.description)")
// Body: 37.0°C / 98.6°F / 310.1K

let boiling = Temperature.boilingPoint
print("Boiling: \(boiling.description)")
// Boiling: 100.0°C / 212.0°F / 373.1K

let temps: [Temperature] = [
    Temperature(celsius: 100),
    Temperature(celsius: -40),
    Temperature(celsius: 37),
    Temperature(celsius: 0)
]

let sorted = temps.sorted()
for temp in sorted {
    print(temp.formatted())
}
// -40.0°C
// 0.0°C
// 37.0°C
// 100.0°C
```

---

## 19. Practical Exercises

### Exercise 1: BankAccount

```swift
// โจทย์: สร้าง BankAccount struct ที่มีฟีเจอร์ครบถ้วน

struct BankAccount {
    let accountNumber: String
    var owner: String
    private(set) var balance: Double  // read-only จากภายนอก
    
    // Transaction history
    struct Transaction {
        enum TransactionType {
            case deposit, withdrawal, transfer
        }
        
        let type: TransactionType
        let amount: Double
        let date: Date
        var description: String
        
        var isDebit: Bool {
            return type == .withdrawal || type == .transfer
        }
    }
    
    private(set) var transactions: [Transaction] = []
    
    enum AccountError: Error {
        case insufficientFunds(Double)
        case negativeAmount
        case accountClosed
    }
    
    var isActive: Bool = true
    
    init(accountNumber: String, owner: String, initialBalance: Double = 0) {
        self.accountNumber = accountNumber
        self.owner = owner
        self.balance = initialBalance
        
        if initialBalance > 0 {
            transactions.append(Transaction(
                type: .deposit,
                amount: initialBalance,
                date: Date(),
                description: "Initial deposit"
            ))
        }
    }
    
    mutating func deposit(amount: Double, description: String = "Deposit") throws {
        guard isActive else { throw AccountError.accountClosed }
        guard amount > 0 else { throw AccountError.negativeAmount }
        
        balance += amount
        transactions.append(Transaction(
            type: .deposit,
            amount: amount,
            date: Date(),
            description: description
        ))
    }
    
    mutating func withdraw(amount: Double, description: String = "Withdrawal") throws {
        guard isActive else { throw AccountError.accountClosed }
        guard amount > 0 else { throw AccountError.negativeAmount }
        guard balance >= amount else {
            throw AccountError.insufficientFunds(balance)
        }
        
        balance -= amount
        transactions.append(Transaction(
            type: .withdrawal,
            amount: amount,
            date: Date(),
            description: description
        ))
    }
    
    func printStatement() {
        print("=== Statement for \(owner) ===")
        print("Account: \(accountNumber)")
        print("Balance: \(String(format: "%.2f", balance)) THB")
        print("\nTransactions:")
        for trans in transactions {
            let sign = trans.isDebit ? "-" : "+"
            print("  \(sign)\(String(format: "%.2f", trans.amount)) - \(trans.description)")
        }
    }
}

// ทดสอบ
var account = BankAccount(accountNumber: "001-234567", owner: "Alice", initialBalance: 1000)

do {
    try account.deposit(amount: 500, description: "Salary")
    try account.withdraw(amount: 200, description: "Bills")
    try account.withdraw(amount: 800, description: "Rent")
    account.printStatement()
} catch BankAccount.AccountError.insufficientFunds(let balance) {
    print("Error: Insufficient funds. Current balance: \(balance)")
} catch {
    print("Error: \(error)")
}
```

### Exercise 2: Inventory Management

```swift
// โจทย์: สร้าง Inventory management system

struct Product {
    let id: String
    var name: String
    var price: Double
    var quantity: Int
    var category: String
    
    var totalValue: Double {
        return price * Double(quantity)
    }
    
    var isInStock: Bool {
        return quantity > 0
    }
    
    var isLowStock: Bool {
        return quantity > 0 && quantity <= 5
    }
}

struct Inventory {
    private var products: [String: Product] = [:]
    
    var totalProducts: Int { products.count }
    var totalValue: Double { products.values.reduce(0) { $0 + $1.totalValue } }
    var outOfStockCount: Int { products.values.filter { !$0.isInStock }.count }
    
    mutating func addProduct(_ product: Product) {
        products[product.id] = product
    }
    
    mutating func removeProduct(id: String) -> Product? {
        return products.removeValue(forKey: id)
    }
    
    mutating func updateQuantity(id: String, delta: Int) -> Bool {
        guard var product = products[id] else { return false }
        let newQuantity = product.quantity + delta
        guard newQuantity >= 0 else { return false }
        product.quantity = newQuantity
        products[id] = product
        return true
    }
    
    func search(query: String) -> [Product] {
        return products.values.filter {
            $0.name.lowercased().contains(query.lowercased()) ||
            $0.category.lowercased().contains(query.lowercased())
        }.sorted { $0.name < $1.name }
    }
    
    func getLowStockItems() -> [Product] {
        return products.values.filter { $0.isLowStock }.sorted { $0.name < $1.name }
    }
    
    func getByCategory(_ category: String) -> [Product] {
        return products.values.filter { $0.category == category }.sorted { $0.name < $1.name }
    }
    
    func printReport() {
        print("=== Inventory Report ===")
        print("Total Products: \(totalProducts)")
        print("Total Value: \(String(format: "%.2f", totalValue)) THB")
        print("Out of Stock: \(outOfStockCount)")
        print()
        
        let lowStock = getLowStockItems()
        if !lowStock.isEmpty {
            print("⚠️ Low Stock Items:")
            for item in lowStock {
                print("  - \(item.name): \(item.quantity) remaining")
            }
        }
    }
}

// ทดสอบ
var inventory = Inventory()

inventory.addProduct(Product(id: "P001", name: "iPhone Case", price: 299, quantity: 50, category: "Accessories"))
inventory.addProduct(Product(id: "P002", name: "USB-C Cable", price: 199, quantity: 3, category: "Accessories"))
inventory.addProduct(Product(id: "P003", name: "Wireless Charger", price: 899, quantity: 0, category: "Chargers"))
inventory.addProduct(Product(id: "P004", name: "Screen Protector", price: 150, quantity: 25, category: "Accessories"))

inventory.printReport()

let accessories = inventory.getByCategory("Accessories")
print("\nAccessories:")
for item in accessories {
    print("  \(item.name): \(item.quantity) pcs @ \(item.price) THB")
}
```

### Exercise 3: Geometric Shapes Calculator

```swift
// โจทย์: สร้าง geometric shapes ด้วย protocols

protocol Shape {
    var area: Double { get }
    var perimeter: Double { get }
    var name: String { get }
    
    func scale(by factor: Double) -> Self
}

extension Shape {
    func describe() -> String {
        return """
        \(name):
          Area: \(String(format: "%.2f", area))
          Perimeter: \(String(format: "%.2f", perimeter))
        """
    }
}

struct Circle: Shape {
    var radius: Double
    var name: String { "Circle (r=\(radius))" }
    var area: Double { Double.pi * radius * radius }
    var perimeter: Double { 2 * Double.pi * radius }
    
    func scale(by factor: Double) -> Circle {
        return Circle(radius: radius * factor)
    }
}

struct Rectangle: Shape {
    var width: Double
    var height: Double
    var name: String { "Rectangle (\(width)x\(height))" }
    var area: Double { width * height }
    var perimeter: Double { 2 * (width + height) }
    
    func scale(by factor: Double) -> Rectangle {
        return Rectangle(width: width * factor, height: height * factor)
    }
}

struct Triangle: Shape {
    var sideA: Double
    var sideB: Double
    var sideC: Double
    
    var name: String { "Triangle (\(sideA), \(sideB), \(sideC))" }
    
    var perimeter: Double { sideA + sideB + sideC }
    
    var area: Double {
        let s = perimeter / 2  // semi-perimeter
        return (s * (s - sideA) * (s - sideB) * (s - sideC)).squareRoot()
    }
    
    var isValid: Bool {
        return sideA + sideB > sideC &&
               sideB + sideC > sideA &&
               sideA + sideC > sideB
    }
    
    func scale(by factor: Double) -> Triangle {
        return Triangle(sideA: sideA * factor, sideB: sideB * factor, sideC: sideC * factor)
    }
}

// ใช้งาน
let shapes: [any Shape] = [
    Circle(radius: 5.0),
    Rectangle(width: 10.0, height: 6.0),
    Triangle(sideA: 3.0, sideB: 4.0, sideC: 5.0)
]

for shape in shapes {
    print(shape.describe())
    print()
}

// หา shape ที่มีพื้นที่มากที่สุด
if let largest = shapes.max(by: { $0.area < $1.area }) {
    print("Largest shape: \(largest.name) with area \(String(format: "%.2f", largest.area))")
}

let totalArea = shapes.reduce(0) { $0 + $1.area }
print("Total area: \(String(format: "%.2f", totalArea))")
```

---

## 20. Summary

### สิ่งที่ได้เรียนรู้ในบทนี้

1. **Struct คืออะไร**: Value type ที่จัดกลุ่มข้อมูลและ methods ไว้ด้วยกัน

2. **Value Type Semantics**: Copy เมื่อ assign หรือส่งผ่าน function

3. **Properties หลายประเภท**:
   - Stored properties: เก็บค่าจริงๆ
   - Computed properties: คำนวณจาก properties อื่น
   - Lazy properties: คำนวณครั้งเดียวเมื่อถูกเรียกใช้ครั้งแรก

4. **Mutating Methods**: ต้องใช้ `mutating` เมื่อต้องการเปลี่ยน properties

5. **Static Properties/Methods**: เป็นของ type ไม่ใช่ instance

6. **Protocols แทน Inheritance**: Struct ไม่มี inheritance แต่ใช้ protocols

7. **Property Observers**: `willSet` และ `didSet` สำหรับตอบสนองการเปลี่ยนแปลง

8. **Nested Types**: สร้าง types ภายใน struct

9. **Subscripts**: Custom subscript สำหรับ collection-like access

10. **Generics**: สร้าง generic struct ที่ทำงานกับหลาย types

### Best Practices

```swift
// 1. ใช้ struct สำหรับ value semantics
struct Coordinates {
    let latitude: Double
    let longitude: Double
}

// 2. ใช้ static factory methods แทน init หลายๆ อัน
struct Color {
    var r, g, b: Double
    
    static func fromHex(_ hex: String) -> Color? { /* ... */ }
    static var red: Color { Color(r: 1, g: 0, b: 0) }
    static var blue: Color { Color(r: 0, g: 0, b: 1) }
}

// 3. Conform to common protocols
struct Point: Equatable, Hashable, Codable, CustomStringConvertible {
    var x, y: Double
    var description: String { "(\(x), \(y))" }
}

// 4. ใช้ private(set) สำหรับ read-only จากภายนอก
struct SafeCounter {
    private(set) var count: Int = 0
    
    mutating func increment() { count += 1 }
    mutating func decrement() { count = max(0, count - 1) }
}
```

### เปรียบเทียบ Struct vs Class อย่างง่าย

```
ใช้ Struct เมื่อ:
✅ ข้อมูลง่ายๆ (coordinates, sizes, colors)
✅ ต้องการ copy semantics
✅ ไม่ต้องการ inheritance
✅ ต้องการ thread safety

ใช้ Class เมื่อ:
✅ ต้องการ inheritance
✅ ต้องการ shared state
✅ ต้องการ deinitializer
✅ ต้องการ identity (===)
```

---

> **ถัดไป**: [Part 14: Classes](Part14_Classes.md) - เรียนรู้เกี่ยวกับ Reference Types, Inheritance, และ ARC

---

*เนื้อหานี้เป็นส่วนหนึ่งของ Swift Programming Course ภาษาไทย*
