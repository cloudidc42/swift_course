# Part 14: Classes (คลาส) ใน Swift

## สารบัญ
1. [Classes คืออะไร?](#1-classes-คืออะไร)
2. [Class Definition Syntax](#2-class-definition-syntax)
3. [Class Properties](#3-class-properties)
4. [Class Methods](#4-class-methods)
5. [Class Initializers](#5-class-initializers)
6. [Deinitializers (deinit)](#6-deinitializers-deinit)
7. [Class as Reference Types](#7-class-as-reference-types)
8. [Inheritance Basics](#8-inheritance-basics)
9. [Method Overriding](#9-method-overriding)
10. [Property Overriding](#10-property-overriding)
11. [Calling super](#11-calling-super)
12. [Preventing Overrides (final)](#12-preventing-overrides-final)
13. [Type Casting](#13-type-casting)
14. [Identity Operators](#14-identity-operators)
15. [Weak and Unowned References](#15-weak-and-unowned-references)
16. [ARC Basics](#16-arc-basics)
17. [Retain Cycles](#17-retain-cycles)
18. [Nested Classes](#18-nested-classes)
19. [Class Subscripts](#19-class-subscripts)
20. [Class vs Struct Decision Guide](#20-class-vs-struct-decision-guide)
21. [Real-World Examples](#21-real-world-examples)
22. [Practical Exercises](#22-practical-exercises)
23. [Summary](#23-summary)

---

## 1. Classes คืออะไร?

**Class** คือ blueprint หรือ template สำหรับสร้าง objects ที่เป็น **reference type** หมายความว่าตัวแปรที่ชี้ไปยัง class instance จะ **share** object เดียวกัน ไม่ได้ copy เหมือน struct

### ความสำคัญของ Class

Class มีความสามารถพิเศษที่ Struct ไม่มี:
- **Inheritance**: ส่บทอด properties และ methods จาก parent class
- **Reference Semantics**: หลาย variables สามารถชี้ไปยัง object เดียวกัน
- **Deinitializer**: ทำความสะอาดทรัพยากรเมื่อ object ถูก deallocate
- **Type Casting**: เปลี่ยน type ขณะ runtime
- **ARC Integration**: Automatic Reference Counting จัดการ memory ให้อัตโนมัติ

### Class vs Struct ภาพรวม

```swift
// Struct: Value Type
struct StructPoint {
    var x: Int
    var y: Int
}

// Class: Reference Type
class ClassPoint {
    var x: Int
    var y: Int
    init(x: Int, y: Int) {
        self.x = x
        self.y = y
    }
}

// ความแตกต่างชัดเจน:
var sp1 = StructPoint(x: 1, y: 2)
var sp2 = sp1          // Copy
sp2.x = 10
print(sp1.x)           // 1 (ไม่เปลี่ยน)
print(sp2.x)           // 10

var cp1 = ClassPoint(x: 1, y: 2)
var cp2 = cp1          // Same reference!
cp2.x = 10
print(cp1.x)           // 10 (เปลี่ยนด้วย!)
print(cp2.x)           // 10
```

### Class ใน Swift Standard Library และ Frameworks

```swift
// ตัวอย่าง classes ที่ใช้บ่อย:
// UIViewController, UIView, UIButton (UIKit)
// NSObject (Foundation)
// URLSession, URLSessionDataTask (Networking)
// OperationQueue, Operation (Concurrency)
// NotificationCenter (Event System)
```

---

## 2. Class Definition Syntax

### รูปแบบพื้นฐาน

```swift
class ClassName {
    // Properties
    var propertyName: Type
    
    // Initializer (ต้องเขียนเองเสมอถ้ามี stored properties)
    init(parameter: Type) {
        self.propertyName = parameter
    }
    
    // Methods
    func methodName() {
        // implementation
    }
    
    // Deinitializer
    deinit {
        // cleanup
    }
}
```

### Class พื้นฐาน

```swift
class Person {
    // Properties
    var firstName: String
    var lastName: String
    var age: Int
    var email: String?
    
    // Computed property
    var fullName: String {
        return "\(firstName) \(lastName)"
    }
    
    // Initializer
    init(firstName: String, lastName: String, age: Int) {
        self.firstName = firstName
        self.lastName = lastName
        self.age = age
    }
    
    // Methods
    func introduce() -> String {
        return "Hi, I'm \(fullName), \(age) years old."
    }
    
    func birthday() {
        age += 1
        print("\(fullName) is now \(age)!")
    }
}

// สร้าง instance
let alice = Person(firstName: "Alice", lastName: "Smith", age: 30)
alice.email = "alice@example.com"

print(alice.fullName)           // Alice Smith
print(alice.introduce())        // Hi, I'm Alice Smith, 30 years old.

alice.birthday()                // Alice Smith is now 31!
print("Age: \(alice.age)")      // Age: 31

// alice เป็น reference
let aliceRef = alice
aliceRef.age = 35
print("alice.age: \(alice.age)")    // 35 (เปลี่ยนด้วย!)
print("aliceRef.age: \(aliceRef.age)")  // 35
```

### Class ที่ inherit จาก class อื่น

```swift
// Base class
class Vehicle {
    var make: String
    var model: String
    var year: Int
    var speed: Double = 0
    
    var description: String {
        return "\(year) \(make) \(model)"
    }
    
    init(make: String, model: String, year: Int) {
        self.make = make
        self.model = model
        self.year = year
    }
    
    func accelerate(by amount: Double) {
        speed += amount
        print("\(description) accelerating to \(speed) km/h")
    }
    
    func brake(by amount: Double) {
        speed = max(0, speed - amount)
        print("\(description) slowing to \(speed) km/h")
    }
}

// Subclass
class ElectricVehicle: Vehicle {
    var batteryLevel: Double
    
    var range: Double {
        return batteryLevel * 3.5  // km per % battery
    }
    
    init(make: String, model: String, year: Int, batteryLevel: Double) {
        self.batteryLevel = batteryLevel
        super.init(make: make, model: model, year: year)
    }
    
    func charge(hours: Double) {
        batteryLevel = min(100, batteryLevel + hours * 10)
        print("Charged to \(batteryLevel)%")
    }
}

let tesla = ElectricVehicle(make: "Tesla", model: "Model 3", year: 2024, batteryLevel: 80)
print(tesla.description)   // 2024 Tesla Model 3
print("Range: \(tesla.range) km")  // Range: 280.0 km
tesla.accelerate(by: 60)   // 2024 Tesla Model 3 accelerating to 60.0 km/h
```

---

## 3. Class Properties

### 3.1 Stored Properties

```swift
class BankAccount {
    // ไม่มี memberwise initializer อัตโนมัติ - ต้องเขียน init เอง
    var accountNumber: String
    var balance: Double
    var owner: String
    var isActive: Bool = true
    
    // Optional property
    var phone: String?
    
    // Constant property (ค่าไม่เปลี่ยนหลัง init)
    let accountType: String
    
    init(accountNumber: String, owner: String, accountType: String, initialBalance: Double = 0) {
        self.accountNumber = accountNumber
        self.owner = owner
        self.accountType = accountType
        self.balance = initialBalance
    }
}

let account = BankAccount(accountNumber: "001-234567", owner: "Bob", accountType: "Savings", initialBalance: 5000)
account.balance += 1000  // ✅ ได้ (var property)
// account.accountType = "Checking"  // ❌ Error (let property)
print("Balance: \(account.balance)")  // 6000.0
```

### 3.2 Computed Properties

```swift
class Circle {
    var radius: Double
    
    init(radius: Double) {
        self.radius = radius
    }
    
    // Read-only computed property
    var area: Double {
        return Double.pi * radius * radius
    }
    
    // Read-write computed property
    var diameter: Double {
        get { return radius * 2 }
        set { radius = newValue / 2 }
    }
    
    var circumference: Double {
        return 2 * Double.pi * radius
    }
}

let circle = Circle(radius: 5)
print("Area: \(circle.area)")         // 78.54...
print("Diameter: \(circle.diameter)") // 10.0

circle.diameter = 20  // setter
print("Radius: \(circle.radius)")     // 10.0
```

### 3.3 Lazy Properties

```swift
class DataManager {
    var dataSource: String
    
    // Lazy: จะ initialize เมื่อถูกเรียกใช้ครั้งแรก
    lazy var processedItems: [String] = {
        print("Processing \(dataSource)...")
        return dataSource.components(separatedBy: ",")
    }()
    
    // Lazy ที่ reference class อื่น
    lazy var formatter: NumberFormatter = {
        let f = NumberFormatter()
        f.numberStyle = .currency
        f.currencyCode = "THB"
        return f
    }()
    
    init(dataSource: String) {
        self.dataSource = dataSource
    }
}

let manager = DataManager(dataSource: "apple,banana,cherry")
print("Manager created")         // Manager created
print(manager.processedItems)    // Processing apple,banana,cherry...
                                 // ["apple", "banana", "cherry"]
```

### 3.4 Type Properties (Static)

```swift
class AppSettings {
    // Type properties
    static var appName: String = "My App"
    static var version: String = "1.0.0"
    static private(set) var instanceCount: Int = 0
    
    // Singleton
    static let shared = AppSettings()
    
    var userPreferences: [String: Any] = [:]
    
    init() {
        AppSettings.instanceCount += 1
    }
    
    static func printInfo() {
        print("\(appName) v\(version)")
    }
}

AppSettings.printInfo()  // My App v1.0.0
let settings = AppSettings()
print("Instances: \(AppSettings.instanceCount)")  // 2 (shared + settings)
```

### 3.5 Property Observers

```swift
class TemperatureSensor {
    var currentTemp: Double = 20.0 {
        willSet(newTemp) {
            print("Temperature changing: \(currentTemp)°C → \(newTemp)°C")
        }
        didSet {
            if currentTemp > 35 {
                print("⚠️ WARNING: High temperature! \(currentTemp)°C")
                triggerAlert()
            } else if currentTemp < 5 {
                print("⚠️ WARNING: Low temperature! \(currentTemp)°C")
                triggerAlert()
            }
        }
    }
    
    var alertCallback: ((Double) -> Void)?
    
    func triggerAlert() {
        alertCallback?(currentTemp)
    }
    
    func updateReading(_ temp: Double) {
        currentTemp = temp
    }
}

let sensor = TemperatureSensor()
sensor.alertCallback = { temp in
    print("Alert sent for temp: \(temp)°C")
}

sensor.updateReading(25.0)  // Temperature changing: 20.0 → 25.0
sensor.updateReading(40.0)  // Temperature changing: 25.0 → 40.0
                             // ⚠️ WARNING: High temperature! 40.0°C
                             // Alert sent for temp: 40.0°C
```

---

## 4. Class Methods

### 4.1 Instance Methods

```swift
class TextEditor {
    var text: String = ""
    private var history: [String] = []
    
    func type(_ newText: String) {
        history.append(text)  // save for undo
        text += newText
    }
    
    func delete(characters count: Int) {
        history.append(text)
        let endIndex = text.index(text.endIndex, offsetBy: -min(count, text.count))
        text = String(text[..<endIndex])
    }
    
    func undo() -> Bool {
        guard !history.isEmpty else { return false }
        text = history.removeLast()
        return true
    }
    
    func clear() {
        history.append(text)
        text = ""
    }
    
    func wordCount() -> Int {
        return text.components(separatedBy: .whitespaces)
                   .filter { !$0.isEmpty }
                   .count
    }
    
    func lineCount() -> Int {
        return text.components(separatedBy: "\n").count
    }
}

let editor = TextEditor()
editor.type("Hello, ")
editor.type("World!")
print(editor.text)        // Hello, World!

editor.delete(characters: 6)
print(editor.text)        // Hello,

editor.undo()
print(editor.text)        // Hello, World!

print("Words: \(editor.wordCount())")  // 2
```

### 4.2 Class Methods (Type Methods)

```swift
class MathHelper {
    // ใช้ class func (สามารถ override ได้ใน subclass)
    class func factorial(_ n: Int) -> Int {
        guard n > 1 else { return 1 }
        return n * factorial(n - 1)
    }
    
    // ใช้ static func (ไม่สามารถ override ได้)
    static func fibonacci(_ n: Int) -> Int {
        guard n > 1 else { return n }
        var a = 0, b = 1
        for _ in 2...n {
            (a, b) = (b, a + b)
        }
        return b
    }
    
    static func isPrime(_ n: Int) -> Bool {
        guard n > 1 else { return false }
        guard n > 3 else { return true }
        guard n % 2 != 0 && n % 3 != 0 else { return false }
        var i = 5
        while i * i <= n {
            if n % i == 0 || n % (i + 2) == 0 { return false }
            i += 6
        }
        return true
    }
}

print(MathHelper.factorial(5))    // 120
print(MathHelper.fibonacci(10))   // 55
print(MathHelper.isPrime(17))     // true
print(MathHelper.isPrime(15))     // false
```

---

## 5. Class Initializers

### 5.1 Designated Initializers

**Designated initializer** คือ initializer หลักที่ initialize ทุก stored property และ call super.init()

```swift
class Animal {
    var name: String
    var sound: String
    var legs: Int
    
    // Designated initializer
    init(name: String, sound: String, legs: Int) {
        self.name = name
        self.sound = sound
        self.legs = legs
    }
}

class Dog: Animal {
    var breed: String
    var isVaccinated: Bool
    
    // Designated initializer ของ subclass
    // ต้อง call super.init() หลังจาก initialize properties ของตัวเอง
    init(name: String, breed: String, isVaccinated: Bool = false) {
        // Phase 1: initialize stored properties ของ subclass ก่อน
        self.breed = breed
        self.isVaccinated = isVaccinated
        // Phase 2: call super.init()
        super.init(name: name, sound: "Woof", legs: 4)
        // Phase 3: สามารถ customize inherited properties ได้
        // (เพราะ super.init() เสร็จแล้ว)
    }
    
    func bark() -> String {
        return "\(name) says: \(sound)!"
    }
}

let dog = Dog(name: "Max", breed: "Labrador", isVaccinated: true)
print(dog.bark())        // Max says: Woof!
print("Breed: \(dog.breed)")  // Labrador
print("Legs: \(dog.legs)")    // 4
```

### 5.2 Convenience Initializers

**Convenience initializer** เป็น initializer เสริมที่เรียก designated initializer ภายใน class เดียวกัน

```swift
class Size {
    var width: Double
    var height: Double
    
    // Designated initializer
    init(width: Double, height: Double) {
        self.width = width
        self.height = height
    }
    
    // Convenience initializer: square
    convenience init(square side: Double) {
        self.init(width: side, height: side)  // เรียก designated
    }
    
    // Convenience initializer: from another Size
    convenience init(doubling other: Size) {
        self.init(width: other.width * 2, height: other.height * 2)
    }
    
    // Convenience initializer: default
    convenience init() {
        self.init(width: 0, height: 0)
    }
}

let s1 = Size(width: 100, height: 50)
let s2 = Size(square: 75)
let s3 = Size(doubling: s1)
let s4 = Size()

print("s1: \(s1.width) x \(s1.height)")  // 100.0 x 50.0
print("s2: \(s2.width) x \(s2.height)")  // 75.0 x 75.0
print("s3: \(s3.width) x \(s3.height)")  // 200.0 x 100.0
print("s4: \(s4.width) x \(s4.height)")  // 0.0 x 0.0
```

### 5.3 Required Initializers

**Required initializer** บังคับให้ subclass ทุกตัวต้อง implement

```swift
class Shape {
    var color: String
    
    // required: ทุก subclass ต้อง implement
    required init(color: String) {
        self.color = color
    }
    
    var area: Double { return 0 }
    
    func describe() {
        print("\(color) \(type(of: self)) with area \(String(format: "%.2f", area))")
    }
}

class Circle: Shape {
    var radius: Double
    
    // ต้องมี required init เพราะ superclass บังคับ
    required init(color: String) {
        self.radius = 1.0
        super.init(color: color)
    }
    
    // สามารถมี designated init เพิ่มได้
    init(color: String, radius: Double) {
        self.radius = radius
        super.init(color: color)
    }
    
    override var area: Double {
        return Double.pi * radius * radius
    }
}

class Rectangle: Shape {
    var width: Double
    var height: Double
    
    required init(color: String) {
        self.width = 1.0
        self.height = 1.0
        super.init(color: color)
    }
    
    init(color: String, width: Double, height: Double) {
        self.width = width
        self.height = height
        super.init(color: color)
    }
    
    override var area: Double {
        return width * height
    }
}

// Factory function ที่ต้องการ required init
func createShape<T: Shape>(type: T.Type, color: String) -> T {
    return T(color: color)
}

let redCircle = createShape(type: Circle.self, color: "Red")
redCircle.describe()  // Red Circle with area 3.14

let blueRect = createShape(type: Rectangle.self, color: "Blue")
blueRect.describe()   // Blue Rectangle with area 1.00
```

### 5.4 Failable Initializers

```swift
class User {
    let username: String
    let email: String
    let age: Int
    
    // Failable initializer
    init?(username: String, email: String, age: Int) {
        // Validate
        guard !username.isEmpty,
              username.count >= 3 else {
            print("Error: Username too short")
            return nil
        }
        
        guard email.contains("@") && email.contains(".") else {
            print("Error: Invalid email")
            return nil
        }
        
        guard age >= 18 else {
            print("Error: Must be 18 or older")
            return nil
        }
        
        self.username = username
        self.email = email
        self.age = age
    }
}

if let user = User(username: "alice_wonder", email: "alice@example.com", age: 25) {
    print("Created user: \(user.username)")  // Created user: alice_wonder
}

if let young = User(username: "bob", email: "bob@test.com", age: 15) {
    print("Created: \(young.username)")
} else {
    print("Failed to create user")  // Error: Must be 18 or older
                                    // Failed to create user
}
```

---

## 6. Deinitializers (deinit)

### ความหมายและการใช้งาน

**Deinitializer** (deinit) ถูกเรียกอัตโนมัติเมื่อ instance ถูก deallocate ใช้สำหรับ cleanup เช่น ปิดไฟล์ ยกเลิก subscriptions หยุด timers

```swift
class Resource {
    let name: String
    
    init(name: String) {
        self.name = name
        print("[\(name)] Created")
    }
    
    deinit {
        print("[\(name)] Deallocated")
    }
}

// ทดสอบ deinit
print("Before scope")
do {
    let r1 = Resource(name: "Resource1")  // Created
    let r2 = Resource(name: "Resource2")  // Created
    print("Inside scope")
    // r1, r2 ออก scope → deinit ถูกเรียก
}
print("After scope")
// Output:
// Before scope
// [Resource1] Created
// [Resource2] Created
// Inside scope
// [Resource2] Deallocated
// [Resource1] Deallocated
// After scope
```

### Deinit สำหรับ Resource Management

```swift
class FileLogger {
    let filename: String
    private var fileHandle: FileHandle?
    
    init(filename: String) {
        self.filename = filename
        // เปิดไฟล์
        let path = "/tmp/\(filename)"
        FileHandle.forWritingAtPath(path).map { fileHandle = $0 }
        print("Opened log file: \(filename)")
    }
    
    func log(_ message: String) {
        let line = "[\(Date())] \(message)\n"
        fileHandle?.write(line.data(using: .utf8) ?? Data())
    }
    
    deinit {
        // ปิดไฟล์เมื่อ deallocate
        fileHandle?.closeFile()
        print("Closed log file: \(filename)")
    }
}

class NetworkClient {
    var timer: Timer?
    var isConnected: Bool = false
    
    init() {
        print("NetworkClient created")
        // เริ่ม timer สำหรับ heartbeat
        timer = Timer.scheduledTimer(withTimeInterval: 5.0, repeats: true) { _ in
            print("Heartbeat sent")
        }
    }
    
    func connect() {
        isConnected = true
        print("Connected!")
    }
    
    func disconnect() {
        isConnected = false
        print("Disconnected")
    }
    
    deinit {
        // หยุด timer และ disconnect
        timer?.invalidate()
        timer = nil
        if isConnected {
            disconnect()
        }
        print("NetworkClient deallocated")
    }
}
```

### Deinit Chain

```swift
class Base {
    init() { print("Base init") }
    deinit { print("Base deinit") }
}

class Middle: Base {
    override init() {
        super.init()
        print("Middle init")
    }
    deinit { print("Middle deinit") }
}

class Top: Middle {
    override init() {
        super.init()
        print("Top init")
    }
    deinit { print("Top deinit") }
}

// Order of init: Base → Middle → Top
// Order of deinit: Top → Middle → Base (ย้อนกลับ)
var obj: Top? = Top()
// Base init
// Middle init
// Top init

obj = nil
// Top deinit
// Middle deinit
// Base deinit
```

---

## 7. Class as Reference Types

### Reference Semantics อธิบายอย่างละเอียด

```swift
class Counter {
    var count: Int
    var name: String
    
    init(name: String, count: Int = 0) {
        self.name = name
        self.count = count
    }
    
    func increment() { count += 1 }
    func decrement() { count = max(0, count - 1) }
}

// Reference Type: หลาย variables ชี้ไปยัง object เดียวกัน
let counter = Counter(name: "Main")
let reference = counter  // Same object!

counter.increment()
counter.increment()
print("counter.count: \(counter.count)")    // 2
print("reference.count: \(reference.count)")  // 2 (same object!)

reference.increment()
print("counter.count: \(counter.count)")    // 3
print("reference.count: \(reference.count)")  // 3

// ตรวจสอบว่าเป็น object เดียวกันหรือไม่
print(counter === reference)  // true
```

### Reference ใน Collections

```swift
class Item {
    var name: String
    var quantity: Int
    
    init(name: String, quantity: Int) {
        self.name = name
        self.quantity = quantity
    }
}

// Array ของ class: array copy แต่ objects ยังเป็น references
var list1 = [Item(name: "Apple", quantity: 10)]
var list2 = list1  // array ถูก copy แต่ items ไม่ถูก copy

list2.append(Item(name: "Banana", quantity: 5))

print("list1 count: \(list1.count)")  // 1
print("list2 count: \(list2.count)")  // 2

// แต่การเปลี่ยน item ใน list1 กระทบ list2 ด้วย!
list1[0].quantity = 20
print("list1[0].quantity: \(list1[0].quantity)")  // 20
print("list2[0].quantity: \(list2[0].quantity)")  // 20 (same reference!)
```

### Passing Classes to Functions

```swift
class GameScore {
    var player: String
    var score: Int
    var lives: Int
    
    init(player: String) {
        self.player = player
        self.score = 0
        self.lives = 3
    }
}

// Class ส่งผ่าน reference
func addBonus(_ gameScore: GameScore, points: Int) {
    gameScore.score += points  // เปลี่ยนได้แม้ parameter ไม่มี inout
    print("\(gameScore.player) got \(points) bonus points!")
}

let playerScore = GameScore(player: "Alice")
playerScore.score = 100

addBonus(playerScore, points: 50)
print("Final score: \(playerScore.score)")  // 150 (เปลี่ยนแล้ว!)

// เปรียบเทียบกับ Struct
struct StructScore {
    var player: String
    var score: Int
}

func addBonusStruct(_ score: StructScore, points: Int) -> StructScore {
    var updated = score
    updated.score += points  // ต้อง return กลับ
    return updated
}
```

---

## 8. Inheritance Basics

### การ inherit จาก base class

```swift
// Base class (Superclass)
class Animal {
    var name: String
    var age: Int
    private var isAlive: Bool = true
    
    var description: String {
        return "\(name) (age: \(age))"
    }
    
    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }
    
    func makeSound() -> String {
        return "..."
    }
    
    func eat(food: String) {
        print("\(name) is eating \(food)")
    }
    
    func sleep() {
        print("\(name) is sleeping")
    }
}

// Subclass
class Cat: Animal {
    var isIndoor: Bool
    var preferredFood: String
    
    init(name: String, age: Int, isIndoor: Bool, preferredFood: String = "fish") {
        self.isIndoor = isIndoor
        self.preferredFood = preferredFood
        super.init(name: name, age: age)
    }
    
    override func makeSound() -> String {
        return "Meow!"
    }
    
    func purr() {
        print("\(name) is purring... 😸")
    }
    
    func scratch() {
        print("\(name) is scratching furniture!")
    }
}

class Dog: Animal {
    var breed: String
    var isGoodBoy: Bool = true
    
    init(name: String, age: Int, breed: String) {
        self.breed = breed
        super.init(name: name, age: age)
    }
    
    override func makeSound() -> String {
        return "Woof!"
    }
    
    func fetch(item: String) {
        print("\(name) fetches the \(item)!")
    }
    
    func wagTail() {
        print("\(name) wags tail happily!")
    }
    
    override var description: String {
        return "\(super.description) [\(breed)]"
    }
}

// ใช้งาน inheritance
let cat = Cat(name: "Whiskers", age: 3, isIndoor: true)
let dog = Dog(name: "Buddy", age: 5, breed: "Golden Retriever")

print(cat.makeSound())    // Meow!
print(dog.makeSound())    // Woof!
print(dog.description)    // Buddy (age: 5) [Golden Retriever]

// Polymorphism
let animals: [Animal] = [cat, dog, Animal(name: "Mystery", age: 1)]
for animal in animals {
    print("\(animal.name): \(animal.makeSound())")
}
// Whiskers: Meow!
// Buddy: Woof!
// Mystery: ...
```

### Multi-level Inheritance

```swift
// Level 1
class LivingThing {
    var isAlive: Bool = true
    
    func breathe() {
        print("Breathing...")
    }
}

// Level 2
class Animal: LivingThing {
    var name: String
    
    init(name: String) {
        self.name = name
    }
    
    func move() {
        print("\(name) is moving")
    }
}

// Level 3
class Mammal: Animal {
    var bodyTemp: Double = 37.0
    
    func regulateTemperature() {
        print("\(name) regulates body temperature to \(bodyTemp)°C")
    }
}

// Level 4
class Human: Mammal {
    var language: String
    
    init(name: String, language: String) {
        self.language = language
        super.init(name: name)
    }
    
    func speak(_ text: String) {
        print("\(name) says in \(language): '\(text)'")
    }
}

let human = Human(name: "Alice", language: "Thai")
human.breathe()               // Breathing... (from LivingThing)
human.move()                  // Alice is moving (from Animal)
human.regulateTemperature()   // Alice regulates body temperature to 37.0°C (from Mammal)
human.speak("สวัสดี!")        // Alice says in Thai: 'สวัสดี!'
```

---

## 9. Method Overriding

### override keyword

```swift
class Shape {
    var color: String = "black"
    
    func draw() {
        print("Drawing a shape in \(color)")
    }
    
    func area() -> Double {
        return 0.0
    }
    
    func describe() {
        print("Shape: color=\(color), area=\(area())")
    }
}

class Circle: Shape {
    var radius: Double
    
    init(radius: Double, color: String = "black") {
        self.radius = radius
        super.init()
        self.color = color
    }
    
    // Override method
    override func draw() {
        print("Drawing a \(color) circle with radius \(radius)")
    }
    
    override func area() -> Double {
        return Double.pi * radius * radius
    }
}

class Rectangle: Shape {
    var width: Double
    var height: Double
    
    init(width: Double, height: Double, color: String = "black") {
        self.width = width
        self.height = height
        super.init()
        self.color = color
    }
    
    override func draw() {
        print("Drawing a \(color) rectangle \(width)x\(height)")
    }
    
    override func area() -> Double {
        return width * height
    }
}

let shapes: [Shape] = [
    Circle(radius: 5, color: "red"),
    Rectangle(width: 10, height: 6, color: "blue"),
    Shape()
]

for shape in shapes {
    shape.draw()
    shape.describe()
    print()
}
// Drawing a red circle with radius 5.0
// Shape: color=red, area=78.5398...
//
// Drawing a blue rectangle 10.0x6.0
// Shape: color=blue, area=60.0
//
// Drawing a shape in black
// Shape: color=black, area=0.0
```

### Override กับ Computed Properties

```swift
class Vehicle {
    var speed: Double = 0
    var name: String
    
    init(name: String) {
        self.name = name
    }
    
    var speedDescription: String {
        return "\(name) is going \(speed) km/h"
    }
    
    var isMoving: Bool {
        return speed > 0
    }
    
    var maxSpeed: Double {
        return 120.0
    }
}

class SportsCar: Vehicle {
    var turboEnabled: Bool = false
    
    override var maxSpeed: Double {
        return turboEnabled ? 300.0 : 250.0
    }
    
    override var speedDescription: String {
        let turbo = turboEnabled ? " [TURBO]" : ""
        return "\(name)\(turbo) roaring at \(speed) km/h!"
    }
    
    func activateTurbo() {
        turboEnabled = true
        print("Turbo activated!")
    }
}

let car = SportsCar(name: "Ferrari")
car.speed = 200
print(car.speedDescription)  // Ferrari roaring at 200.0 km/h!
print("Max: \(car.maxSpeed)") // Max: 250.0

car.activateTurbo()
print("New max: \(car.maxSpeed)")  // New max: 300.0
```

---

## 10. Property Overriding

### Override stored property ด้วย computed property

```swift
class BaseClass {
    var value: Int = 0
    
    var description: String {
        return "Base value: \(value)"
    }
}

class DerivedClass: BaseClass {
    // Override computed property เพื่อเพิ่ม property observer
    override var value: Int {
        willSet {
            print("Value about to change from \(value) to \(newValue)")
        }
        didSet {
            print("Value changed from \(oldValue) to \(value)")
        }
    }
    
    override var description: String {
        return "Derived value: \(value) (override)"
    }
}

let derived = DerivedClass()
derived.value = 5
// Value about to change from 0 to 5
// Value changed from 0 to 5

print(derived.description)  // Derived value: 5 (override)
```

### Override กฎ

```swift
class Account {
    var balance: Double = 0
    
    // read-only computed property
    var summary: String {
        return "Balance: \(balance)"
    }
}

class PremiumAccount: Account {
    var bonusPoints: Int = 0
    
    // สามารถ override read-only เป็น read-write ได้
    // แต่ override read-write เป็น read-only ไม่ได้
    override var summary: String {
        return "Premium Account - Balance: \(balance), Points: \(bonusPoints)"
    }
}

let premium = PremiumAccount()
premium.balance = 10000
premium.bonusPoints = 500
print(premium.summary)  // Premium Account - Balance: 10000.0, Points: 500
```

---

## 11. Calling super

### เรียก super ใน Methods

```swift
class Logger {
    var prefix: String
    
    init(prefix: String) {
        self.prefix = prefix
    }
    
    func log(_ message: String) {
        print("[\(prefix)] \(message)")
    }
    
    func logError(_ message: String) {
        log("ERROR: \(message)")
    }
}

class TimestampLogger: Logger {
    override func log(_ message: String) {
        let timestamp = ISO8601DateFormatter().string(from: Date())
        // เรียก super.log() และเพิ่ม timestamp
        super.log("[\(timestamp)] \(message)")
    }
}

class FileLogger: TimestampLogger {
    var filename: String
    
    init(prefix: String, filename: String) {
        self.filename = filename
        super.init(prefix: prefix)
    }
    
    override func log(_ message: String) {
        // เพิ่ม behavior แล้วเรียก super
        saveToFile(message)
        super.log(message)  // calls TimestampLogger.log() → Logger.log()
    }
    
    private func saveToFile(_ message: String) {
        // จำลองการบันทึกไฟล์
        print("Saved to \(filename): \(message)")
    }
}

let logger = FileLogger(prefix: "APP", filename: "app.log")
logger.log("Application started")
// Saved to app.log: Application started
// [APP] [2024-01-15T10:30:00Z] Application started
```

### เรียก super ใน Initializers

```swift
class Vehicle {
    var make: String
    var model: String
    var year: Int
    
    init(make: String, model: String, year: Int) {
        self.make = make
        self.model = model
        self.year = year
        print("Vehicle initialized: \(year) \(make) \(model)")
    }
    
    func start() {
        print("\(make) \(model) starting...")
    }
}

class ElectricCar: Vehicle {
    var batteryCapacity: Double  // kWh
    var chargingLevel: Double    // 0-100%
    
    init(make: String, model: String, year: Int, batteryCapacity: Double) {
        // 1. Initialize stored properties ของ subclass ก่อน
        self.batteryCapacity = batteryCapacity
        self.chargingLevel = 100.0
        
        // 2. เรียก super.init()
        super.init(make: make, model: model, year: year)
        
        // 3. Customize เพิ่มเติม (optional)
        print("Electric car ready with \(batteryCapacity)kWh battery")
    }
    
    override func start() {
        super.start()  // เรียก parent's start()
        print("Running silently on electricity")
    }
    
    func charge(hours: Double) {
        chargingLevel = min(100, chargingLevel + hours * 20)
        print("Charged to \(chargingLevel)%")
    }
}

let tesla = ElectricCar(make: "Tesla", model: "Model S", year: 2024, batteryCapacity: 100)
// Vehicle initialized: 2024 Tesla Model S
// Electric car ready with 100.0kWh battery

tesla.start()
// Tesla Model S starting...
// Running silently on electricity
```

---

## 12. Preventing Overrides (final)

### final class, final func, final var

```swift
// final class: ไม่สามารถ subclass ได้
final class Singleton {
    static let shared = Singleton()
    private init() {}
    
    var data: String = ""
    
    func process() {
        print("Processing: \(data)")
    }
}

// ❌ Error: final class ไม่สามารถ inherit ได้
// class MySingleton: Singleton {}

class BaseDocument {
    var title: String
    var content: String
    
    init(title: String, content: String) {
        self.title = title
        self.content = content
    }
    
    // final method: ไม่สามารถ override ได้
    final func save() {
        print("Saving '\(title)' to storage...")
        // ใช้ template method pattern: saveContent() สามารถ override ได้
        saveContent()
        print("Saved successfully")
    }
    
    func saveContent() {
        print("Default save: \(content)")
    }
    
    // final property: ไม่สามารถ override ได้
    final var id: String {
        return title.lowercased().replacingOccurrences(of: " ", with: "-")
    }
}

class HTMLDocument: BaseDocument {
    override func saveContent() {
        print("Saving as HTML: <html>\(content)</html>")
    }
    
    // ❌ Error: ไม่สามารถ override final method
    // override func save() {}
    
    // ❌ Error: ไม่สามารถ override final property
    // override var id: String { return "html-\(title)" }
}

let doc = HTMLDocument(title: "My Page", content: "Hello World")
doc.save()
// Saving 'My Page' to storage...
// Saving as HTML: <html>Hello World</html>
// Saved successfully

print("ID: \(doc.id)")  // my-page
```

### @objc และ dynamic

```swift
class AnimatedView {
    // @objc dynamic: ใช้สำหรับ KVO และ Objective-C interop
    @objc dynamic var opacity: Double = 1.0
    @objc dynamic var isHidden: Bool = false
    
    func animate(to opacity: Double, duration: Double) {
        print("Animating to opacity \(opacity) over \(duration)s")
        self.opacity = opacity
    }
}
```

---

## 13. Type Casting

### is, as, as?, as!

**Type casting** ใช้ตรวจสอบ type ของ instance หรือแปลง type

```swift
class Media {
    var title: String
    init(title: String) { self.title = title }
}

class Movie: Media {
    var director: String
    var duration: Int  // minutes
    
    init(title: String, director: String, duration: Int) {
        self.director = director
        self.duration = duration
        super.init(title: title)
    }
}

class Song: Media {
    var artist: String
    var duration: TimeInterval  // seconds
    
    init(title: String, artist: String, duration: TimeInterval) {
        self.artist = artist
        self.duration = duration
        super.init(title: title)
    }
}

class Podcast: Media {
    var host: String
    var episodeNumber: Int
    
    init(title: String, host: String, episodeNumber: Int) {
        self.host = host
        self.episodeNumber = episodeNumber
        super.init(title: title)
    }
}

// สร้าง collection แบบ heterogeneous
let library: [Media] = [
    Movie(title: "Inception", director: "Nolan", duration: 148),
    Song(title: "Bohemian Rhapsody", artist: "Queen", duration: 354),
    Podcast(title: "Swift Talk", host: "Chris", episodeNumber: 100),
    Movie(title: "Interstellar", director: "Nolan", duration: 169),
    Song(title: "Yesterday", artist: "Beatles", duration: 125)
]

// is: ตรวจสอบ type
var movieCount = 0
var songCount = 0
var podcastCount = 0

for item in library {
    if item is Movie { movieCount += 1 }
    else if item is Song { songCount += 1 }
    else if item is Podcast { podcastCount += 1 }
}

print("Movies: \(movieCount), Songs: \(songCount), Podcasts: \(podcastCount)")
// Movies: 2, Songs: 2, Podcasts: 1

// as?: Optional downcast
for item in library {
    if let movie = item as? Movie {
        print("Movie: \(movie.title) by \(movie.director) (\(movie.duration) min)")
    } else if let song = item as? Song {
        print("Song: \(song.title) by \(song.artist)")
    } else if let podcast = item as? Podcast {
        print("Podcast: \(podcast.title) ep.\(podcast.episodeNumber)")
    }
}

// as!: Forced downcast (ระวัง! crash ถ้า type ไม่ตรง)
let firstItem = library[0]
let firstMovie = firstItem as! Movie  // OK เพราะรู้ว่าเป็น Movie
print("First movie: \(firstMovie.director)")  // Nolan

// ดีกว่าใช้ guard
guard let secondSong = library[1] as? Song else {
    fatalError("Expected a Song")
}
print("Song: \(secondSong.title)")  // Bohemian Rhapsody
```

### Type Casting กับ Generic Types

```swift
class Container<T> {
    var value: T
    init(_ value: T) { self.value = value }
}

func processAny(_ item: Any) {
    switch item {
    case let string as String:
        print("String: \(string.uppercased())")
    case let int as Int:
        print("Int: \(int * 2)")
    case let double as Double:
        print("Double: \(String(format: "%.2f", double))")
    case let array as [Int]:
        print("Array of Int: \(array)")
    case let dict as [String: Any]:
        print("Dictionary with keys: \(dict.keys.joined(separator: ", "))")
    default:
        print("Unknown type: \(type(of: item))")
    }
}

processAny("Hello")             // String: HELLO
processAny(42)                  // Int: 84
processAny(3.14)                // Double: 3.14
processAny([1, 2, 3])          // Array of Int: [1, 2, 3]
processAny(["name": "Alice"])   // Dictionary with keys: name
```

---

## 14. Identity Operators

### === และ !==

**Identity operators** ตรวจสอบว่า references ชี้ไปยัง object เดียวกันหรือไม่

```swift
class Person {
    var name: String
    var age: Int
    
    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }
}

let alice = Person(name: "Alice", age: 30)
let aliceRef = alice      // Same reference
let anotherAlice = Person(name: "Alice", age: 30)  // Different object

// === ตรวจสอบว่าเป็น object เดียวกัน
print(alice === aliceRef)        // true (same reference)
print(alice === anotherAlice)    // false (different objects)
print(alice !== anotherAlice)    // true (not same)

// == ตรวจสอบว่า value เท่ากัน (ต้อง implement Equatable)
// alice == anotherAlice  // ต้อง conform to Equatable

// ตัวอย่าง: ป้องกัน self-assignment
class Node {
    var value: Int
    var next: Node?
    
    init(value: Int) {
        self.value = value
    }
    
    func append(_ node: Node) {
        // ตรวจสอบว่าไม่ใช่ตัวเอง
        guard node !== self else {
            print("Cannot append node to itself!")
            return
        }
        next = node
    }
}

let n1 = Node(value: 1)
let n2 = Node(value: 2)

n1.append(n2)   // OK
n1.append(n1)   // Cannot append node to itself!
```

### Identity ใน Collection

```swift
class Task {
    var id: Int
    var title: String
    var isCompleted: Bool = false
    
    init(id: Int, title: String) {
        self.id = id
        self.title = title
    }
}

var tasks = [
    Task(id: 1, title: "Buy groceries"),
    Task(id: 2, title: "Exercise"),
    Task(id: 3, title: "Read book")
]

let targetTask = tasks[1]  // Reference to task 2

// ค้นหาด้วย identity
if let index = tasks.firstIndex(where: { $0 === targetTask }) {
    tasks[index].isCompleted = true
    print("Completed: \(tasks[index].title)")  // Completed: Exercise
}

// ลบ task ด้วย identity
tasks.removeAll { $0 === targetTask }
print("Tasks remaining: \(tasks.count)")  // 2
```

---

## 15. Weak and Unowned References

### ปัญหา Strong Reference

```swift
// ปัญหา: Strong reference cycle
class ParentNode {
    var name: String
    var child: ChildNode?
    
    init(name: String) {
        self.name = name
        print("\(name) created")
    }
    
    deinit {
        print("\(name) deallocated")
    }
}

class ChildNode {
    var name: String
    var parent: ParentNode?  // Strong reference → ปัญหา!
    
    init(name: String) {
        self.name = name
        print("\(name) created")
    }
    
    deinit {
        print("\(name) deallocated")
    }
}

// Strong reference cycle:
var parent: ParentNode? = ParentNode(name: "Parent")
var child: ChildNode? = ChildNode(name: "Child")

parent?.child = child  // parent → child (strong)
child?.parent = parent // child → parent (strong) ← ปัญหา!

parent = nil  // Parent ยังไม่ถูก deallocate!
child = nil   // Child ยังไม่ถูก deallocate!
// Memory leak!
```

### Weak References

**weak** ป้องกัน strong reference cycle โดยไม่เพิ่ม reference count

```swift
class Department {
    var name: String
    var employees: [Employee] = []
    
    init(name: String) {
        self.name = name
        print("Department '\(name)' created")
    }
    
    deinit {
        print("Department '\(name)' deallocated")
    }
    
    func hire(_ employee: Employee) {
        employees.append(employee)
        employee.department = self
    }
}

class Employee {
    var name: String
    weak var department: Department?  // weak: ไม่เพิ่ม retain count
    
    init(name: String) {
        self.name = name
        print("Employee '\(name)' created")
    }
    
    deinit {
        print("Employee '\(name)' deallocated")
    }
    
    func introduce() {
        if let dept = department {
            print("\(name) works in \(dept.name)")
        } else {
            print("\(name) has no department")
        }
    }
}

var dept: Department? = Department(name: "Engineering")
var emp: Employee? = Employee(name: "Alice")

dept?.hire(emp!)
emp?.introduce()  // Alice works in Engineering

dept = nil  // Department deallocated
emp?.introduce()  // Alice has no department (weak ref เป็น nil อัตโนมัติ)
emp = nil   // Employee deallocated
```

### Unowned References

**unowned** ไม่เพิ่ม reference count แต่สมมติว่า object ยังมีอยู่เสมอ (ไม่เป็น nil)

```swift
class CreditCard {
    let number: String
    let cardholderName: String
    unowned let owner: Customer  // unowned: owner จะ outlive CreditCard
    
    init(number: String, name: String, owner: Customer) {
        self.number = number
        self.cardholderName = name
        self.owner = owner
        print("Card \(number) created for \(name)")
    }
    
    deinit {
        print("Card \(number) deallocated")
    }
}

class Customer {
    var name: String
    var creditCard: CreditCard?
    
    init(name: String) {
        self.name = name
        print("Customer '\(name)' created")
    }
    
    deinit {
        print("Customer '\(name)' deallocated")
    }
    
    func addCreditCard(number: String) {
        creditCard = CreditCard(number: number, name: name, owner: self)
    }
}

var customer: Customer? = Customer(name: "Bob")
customer?.addCreditCard(number: "4111-1111-1111-1111")

print(customer?.creditCard?.cardholderName ?? "No card")
// Bob

customer = nil
// Customer 'Bob' deallocated
// Card 4111-1111-1111-1111 deallocated
// ✅ ไม่มี memory leak!
```

### เมื่อใช้ weak vs unowned

```swift
// weak: ใช้เมื่อ reference อาจเป็น nil ได้
class ViewControllerA {
    weak var delegate: SomeDelegate?  // delegate อาจหายไปได้
}

// unowned: ใช้เมื่อ reference มีชีวิตยาวกว่าเสมอ
class Request {
    unowned let session: Session  // session outlives request
    
    init(session: Session) {
        self.session = session
    }
}

// Closure capture list
class Timer {
    var callback: (() -> Void)?
    var count = 0
    
    func start() {
        // [weak self]: ถ้า self อาจ deallocate
        callback = { [weak self] in
            guard let self = self else { return }
            self.count += 1
            print("Tick: \(self.count)")
        }
    }
}
```

---

## 16. ARC Basics

### Automatic Reference Counting

**ARC** จัดการ memory โดยนับจำนวน strong references ไปยัง object เมื่อ count = 0 ก็ deallocate

```swift
class Person {
    let name: String
    
    init(name: String) {
        self.name = name
        print("'\(name)' initialized - ARC count: 1")
    }
    
    deinit {
        print("'\(name)' deinitialized - ARC count: 0")
    }
}

// ARC in action
var ref1: Person?
var ref2: Person?
var ref3: Person?

ref1 = Person(name: "Alice")   // ARC count: 1
ref2 = ref1                     // ARC count: 2
ref3 = ref1                     // ARC count: 3

ref1 = nil  // ARC count: 2
ref2 = nil  // ARC count: 1
ref3 = nil  // ARC count: 0 → deinit called!
// 'Alice' deinitialized
```

### ARC กับ Closures

Closures capture references ซึ่งอาจสร้าง retain cycle

```swift
class HTMLGenerator {
    var text: String
    lazy var convertedHTML: () -> String = {
        // ❌ Capture self strongly → retain cycle
        // return "<p>\(self.text)</p>"
        
        // ✅ Capture self weakly
        [weak self] in
        guard let self = self else { return "" }
        return "<p>\(self.text)</p>"
    }
    
    init(text: String) {
        self.text = text
        print("HTMLGenerator created")
    }
    
    deinit {
        print("HTMLGenerator deallocated")
    }
}

var gen: HTMLGenerator? = HTMLGenerator(text: "Hello, World!")
print(gen!.convertedHTML())  // <p>Hello, World!</p>

gen = nil  // HTMLGenerator deallocated ✅
```

### Strong Reference ใน Closure

```swift
class Counter {
    var count = 0
    var name: String
    
    init(name: String) {
        self.name = name
    }
    
    // Retain cycle version (ไม่ดี)
    func makeIncrementorBad() -> () -> Void {
        return {
            self.count += 1  // Strong capture of self
            print("\(self.name): \(self.count)")
        }
    }
    
    // No retain cycle version (ดี)
    func makeIncrementor() -> () -> Void {
        return { [weak self] in
            self?.count += 1
            if let self = self {
                print("\(self.name): \(self.count)")
            }
        }
    }
    
    deinit { print("\(name) deallocated") }
}

var counter: Counter? = Counter(name: "MyCounter")
let incrementor = counter!.makeIncrementor()

incrementor()   // MyCounter: 1
incrementor()   // MyCounter: 2

counter = nil   // Counter deallocated ✅ (weak capture)
incrementor()   // ไม่ทำอะไร (self เป็น nil แล้ว)
```

---

## 17. Retain Cycles

### ประเภทของ Retain Cycles

```swift
// 1. Direct retain cycle (Object ↔ Object)
class DirectA {
    var b: DirectB?
    deinit { print("DirectA deallocated") }
}

class DirectB {
    var a: DirectA?  // Strong → cycle!
    deinit { print("DirectB deallocated") }
}

// แก้ไขด้วย weak
class FixedA {
    var b: FixedB?
    deinit { print("FixedA deallocated") }
}

class FixedB {
    weak var a: FixedA?  // weak → no cycle
    deinit { print("FixedB deallocated") }
}

// 2. Delegate pattern retain cycle
class DataSource {
    var delegate: DataSourceDelegate?  // Strong → cycle!
}

protocol DataSourceDelegate: AnyObject {
    func dataLoaded()
}

class ViewController: DataSourceDelegate {
    lazy var dataSource = DataSource()
    
    init() {
        dataSource.delegate = self  // VC → DataSource → VC (cycle!)
    }
    
    func dataLoaded() {
        print("Data loaded!")
    }
}

// แก้ไข: ใช้ weak delegate
class BetterDataSource {
    weak var delegate: DataSourceDelegate?  // weak → no cycle ✅
}
```

### Detect Retain Cycles

```swift
// เครื่องมือตรวจสอบ:
// 1. Xcode Memory Graph Debugger
// 2. Instruments - Leaks
// 3. Debug Navigator - Memory

// Pattern สำหรับหลีกเลี่ยง retain cycles:

// Pattern 1: Delegate เป็น weak เสมอ
protocol MyDelegate: AnyObject { }  // AnyObject: reference type เท่านั้น

class MyClass {
    weak var delegate: MyDelegate?  // ✅
}

// Pattern 2: Closure capture list
class MyViewController {
    func loadData() {
        URLSession.shared.dataTask(with: URL(string: "https://api.example.com")!) { [weak self] data, _, _ in
            DispatchQueue.main.async { [weak self] in
                self?.updateUI(with: data)
            }
        }.resume()
    }
    
    func updateUI(with data: Data?) {
        print("Updating UI...")
    }
}

// Pattern 3: unowned ในกรณีที่แน่ใจ
class ImageLoader {
    var completion: (() -> Void)?
    
    func loadImage(url: URL, owner: UIViewController) {
        // unowned เมื่อ owner outlives ImageLoader
        completion = { [unowned owner] in
            print("Image loaded for \(type(of: owner))")
        }
    }
}

class UIViewController {}  // จำลอง
```

---

## 18. Nested Classes

### Class ภายใน Class

```swift
class LinkedList<T> {
    // Nested class สำหรับ Node
    class Node {
        var value: T
        var next: Node?
        weak var prev: Node?  // weak เพื่อป้องกัน cycle ใน doubly-linked list
        
        init(value: T) {
            self.value = value
        }
    }
    
    private var head: Node?
    private var tail: Node?
    private(set) var count: Int = 0
    
    var isEmpty: Bool { head == nil }
    
    func append(_ value: T) {
        let node = Node(value: value)
        
        if let tail = tail {
            tail.next = node
            node.prev = tail
            self.tail = node
        } else {
            head = node
            tail = node
        }
        count += 1
    }
    
    func prepend(_ value: T) {
        let node = Node(value: value)
        
        if let head = head {
            node.next = head
            head.prev = node
            self.head = node
        } else {
            head = node
            tail = node
        }
        count += 1
    }
    
    func removeFirst() -> T? {
        guard let head = head else { return nil }
        self.head = head.next
        self.head?.prev = nil
        count -= 1
        return head.value
    }
    
    func toArray() -> [T] {
        var result: [T] = []
        var current = head
        while let node = current {
            result.append(node.value)
            current = node.next
        }
        return result
    }
}

var list = LinkedList<Int>()
list.append(1)
list.append(2)
list.append(3)
list.prepend(0)

print(list.toArray())   // [0, 1, 2, 3]
print("Count: \(list.count)")  // 4

list.removeFirst()
print(list.toArray())   // [1, 2, 3]
```

### Nested Class สำหรับ Builder Pattern

```swift
class Alert {
    let title: String
    let message: String
    let actions: [AlertAction]
    let style: Style
    
    enum Style {
        case info, warning, error, success
    }
    
    struct AlertAction {
        let title: String
        let style: ActionStyle
        let handler: (() -> Void)?
        
        enum ActionStyle {
            case `default`, cancel, destructive
        }
        
        init(title: String, style: ActionStyle = .default, handler: (() -> Void)? = nil) {
            self.title = title
            self.style = style
            self.handler = handler
        }
    }
    
    private init(title: String, message: String, actions: [AlertAction], style: Style) {
        self.title = title
        self.message = message
        self.actions = actions
        self.style = style
    }
    
    // Nested Builder class
    class Builder {
        private var title: String = ""
        private var message: String = ""
        private var actions: [AlertAction] = []
        private var style: Style = .info
        
        func setTitle(_ title: String) -> Builder {
            self.title = title
            return self
        }
        
        func setMessage(_ message: String) -> Builder {
            self.message = message
            return self
        }
        
        func setStyle(_ style: Style) -> Builder {
            self.style = style
            return self
        }
        
        func addAction(_ action: AlertAction) -> Builder {
            actions.append(action)
            return self
        }
        
        func addAction(title: String, style: AlertAction.ActionStyle = .default, handler: (() -> Void)? = nil) -> Builder {
            actions.append(AlertAction(title: title, style: style, handler: handler))
            return self
        }
        
        func build() -> Alert {
            return Alert(title: title, message: message, actions: actions, style: style)
        }
    }
    
    func show() {
        print("[\(style)] \(title): \(message)")
        for action in actions {
            print("  → [\(action.style)] \(action.title)")
        }
    }
}

// ใช้ Builder
let alert = Alert.Builder()
    .setTitle("Delete Confirmation")
    .setMessage("Are you sure you want to delete this item?")
    .setStyle(.warning)
    .addAction(title: "Cancel", style: .cancel)
    .addAction(title: "Delete", style: .destructive, handler: { print("Deleted!") })
    .build()

alert.show()
// [warning] Delete Confirmation: Are you sure you want to delete this item?
//   → [cancel] Cancel
//   → [destructive] Delete
```

---

## 19. Class Subscripts

### Custom Subscripts ใน Class

```swift
class Cache<Key: Hashable, Value> {
    private var storage: [Key: Value] = [:]
    private var accessOrder: [Key] = []
    let capacity: Int
    
    init(capacity: Int) {
        self.capacity = capacity
    }
    
    subscript(key: Key) -> Value? {
        get {
            return storage[key]
        }
        set {
            if let value = newValue {
                if storage[key] == nil && storage.count >= capacity {
                    // Remove least recently used
                    if let lruKey = accessOrder.first {
                        storage.removeValue(forKey: lruKey)
                        accessOrder.removeFirst()
                    }
                }
                storage[key] = value
                
                // Update access order
                accessOrder.removeAll { $0 == key }
                accessOrder.append(key)
            } else {
                storage.removeValue(forKey: key)
                accessOrder.removeAll { $0 == key }
            }
        }
    }
    
    var count: Int { storage.count }
    
    func keys() -> [Key] {
        return Array(storage.keys)
    }
}

let cache = Cache<String, Int>(capacity: 3)
cache["a"] = 1
cache["b"] = 2
cache["c"] = 3

print(cache["a"] ?? "nil")  // 1

cache["d"] = 4  // "a" ถูกลบ (LRU)
print(cache["a"] ?? "nil")  // nil (ถูกลบ)
print(cache["d"] ?? "nil")  // 4

// ลบ item
cache["b"] = nil
print(cache.count)  // 2
```

### Static Subscripts

```swift
class Configuration {
    private static var settings: [String: Any] = [:]
    
    // Class subscript: ใช้กับ type ไม่ใช่ instance
    class subscript(key: String) -> Any? {
        get { return settings[key] }
        set { settings[key] = newValue }
    }
    
    class func reset() {
        settings.removeAll()
    }
}

// ใช้ผ่าน type
Configuration["theme"] = "dark"
Configuration["language"] = "th"
Configuration["fontSize"] = 16

print(Configuration["theme"] ?? "unknown")     // dark
print(Configuration["language"] ?? "unknown")  // th
```

---

## 20. Class vs Struct Decision Guide

### Decision Tree

```swift
// คำถาม 1: ต้องการ inheritance ไหม?
//   ใช่ → Class
//   ไม่ใช่ → ไปคำถาม 2

// คำถาม 2: ต้องการ shared mutable state ไหม?
//   ใช่ → Class
//   ไม่ใช่ → ไปคำถาม 3

// คำถาม 3: ต้องการ deinitializer ไหม?
//   ใช่ → Class
//   ไม่ใช่ → ไปคำถาม 4

// คำถาม 4: ต้องการ Objective-C interoperability ไหม?
//   ใช่ → Class
//   ไม่ใช่ → Struct

// กฎง่ายๆ: "เริ่มด้วย Struct เสมอ เว้นแต่จำเป็นต้องใช้ Class"
```

### ตัวอย่างเปรียบเทียบ

```swift
// ✅ Struct: Model data
struct UserProfile {
    var id: Int
    var name: String
    var email: String
    
    // Codable สำหรับ JSON
    var jsonRepresentation: Data? {
        try? JSONEncoder().encode(self)
    }
}
extension UserProfile: Codable, Equatable {}

// ✅ Class: Service / Manager
class UserService {
    private var users: [Int: UserProfile] = [:]
    
    func getUser(id: Int) -> UserProfile? {
        return users[id]
    }
    
    func saveUser(_ user: UserProfile) {
        users[user.id] = user
    }
    
    func deleteUser(id: Int) {
        users.removeValue(forKey: id)
    }
}

// ✅ Struct: Configuration
struct NetworkConfig {
    var baseURL: URL
    var timeout: TimeInterval = 30
    var maxRetries: Int = 3
    var headers: [String: String] = [:]
}

// ✅ Class: Network Manager (shared state, singleton)
class NetworkManager {
    static let shared = NetworkManager()
    private var config: NetworkConfig
    
    private init() {
        config = NetworkConfig(baseURL: URL(string: "https://api.example.com")!)
    }
    
    func configure(with config: NetworkConfig) {
        self.config = config
    }
    
    func request(path: String) async throws -> Data {
        let url = config.baseURL.appendingPathComponent(path)
        let (data, _) = try await URLSession.shared.data(from: url)
        return data
    }
}
```

### Performance Considerations

```swift
// Struct: Stack allocation (เร็วกว่าสำหรับ small data)
struct Point { var x, y: Double }  // 16 bytes on stack

// Class: Heap allocation (ช้ากว่า แต่ flexible มากกว่า)
class PointClass {
    var x, y: Double
    init(x: Double, y: Double) { self.x = x; self.y = y }
}

// เมื่อ Struct มี reference type property → ไม่ได้เร็วขึ้นเสมอ
struct View {
    var image: UIImage  // Reference type property
    var text: String    // Value type (CoW)
}

class UIImage {}  // จำลอง
```

---

## 21. Real-World Examples

### 21.1 Person Hierarchy

```swift
// Base class
class Person {
    var firstName: String
    var lastName: String
    var dateOfBirth: Date
    var email: String?
    var phone: String?
    
    var fullName: String {
        return "\(firstName) \(lastName)"
    }
    
    var age: Int {
        let calendar = Calendar.current
        let components = calendar.dateComponents([.year], from: dateOfBirth, to: Date())
        return components.year ?? 0
    }
    
    init(firstName: String, lastName: String, dateOfBirth: Date) {
        self.firstName = firstName
        self.lastName = lastName
        self.dateOfBirth = dateOfBirth
    }
    
    func contactInfo() -> String {
        var info = "Name: \(fullName)\nAge: \(age)"
        if let email = email { info += "\nEmail: \(email)" }
        if let phone = phone { info += "\nPhone: \(phone)" }
        return info
    }
    
    deinit {
        print("Person '\(fullName)' removed from memory")
    }
}

// Student ที่ inherit จาก Person
class Student: Person {
    var studentID: String
    var major: String
    var gpa: Double = 0.0
    var courses: [String] = []
    
    init(firstName: String, lastName: String, dateOfBirth: Date, studentID: String, major: String) {
        self.studentID = studentID
        self.major = major
        super.init(firstName: firstName, lastName: lastName, dateOfBirth: dateOfBirth)
    }
    
    func enroll(in course: String) {
        courses.append(course)
        print("\(fullName) enrolled in \(course)")
    }
    
    func updateGPA(_ newGPA: Double) {
        gpa = max(0, min(4.0, newGPA))
    }
    
    override func contactInfo() -> String {
        return super.contactInfo() + "\nStudent ID: \(studentID)\nMajor: \(major)\nGPA: \(gpa)"
    }
}

// Teacher ที่ inherit จาก Person
class Teacher: Person {
    var employeeID: String
    var department: String
    var salary: Double
    private(set) var courses: [String] = []
    
    init(firstName: String, lastName: String, dateOfBirth: Date,
         employeeID: String, department: String, salary: Double) {
        self.employeeID = employeeID
        self.department = department
        self.salary = salary
        super.init(firstName: firstName, lastName: lastName, dateOfBirth: dateOfBirth)
    }
    
    func assignCourse(_ course: String) {
        courses.append(course)
        print("Assigned \(course) to \(fullName)")
    }
    
    func giveFeedback(to student: Student, for course: String) -> String {
        return "\(student.fullName), your work in \(course) is noted by \(fullName)."
    }
    
    override func contactInfo() -> String {
        return super.contactInfo() + "\nEmployee ID: \(employeeID)\nDepartment: \(department)"
    }
}

// GraduateStudent ที่ inherit จาก Student
class GraduateStudent: Student {
    var thesisTitle: String?
    var advisor: Teacher?
    
    func setAdvisor(_ teacher: Teacher) {
        advisor = teacher
        print("\(teacher.fullName) is now advisor for \(fullName)")
    }
    
    func startThesis(title: String) {
        thesisTitle = title
        print("\(fullName) started thesis: '\(title)'")
    }
    
    override func contactInfo() -> String {
        var info = super.contactInfo()
        if let thesis = thesisTitle {
            info += "\nThesis: \(thesis)"
        }
        if let advisor = advisor {
            info += "\nAdvisor: \(advisor.fullName)"
        }
        return info
    }
}

// ทดสอบ
let birthDate = Calendar.current.date(from: DateComponents(year: 2000, month: 6, day: 15))!
let student = GraduateStudent(
    firstName: "Charlie", lastName: "Brown",
    dateOfBirth: birthDate,
    studentID: "STU001", major: "Computer Science"
)
student.email = "charlie@university.edu"

let teacher = Teacher(
    firstName: "Dr. Jane", lastName: "Smith",
    dateOfBirth: Calendar.current.date(from: DateComponents(year: 1975, month: 3, day: 20))!,
    employeeID: "TCH001", department: "CS", salary: 85000
)

student.setAdvisor(teacher)
student.startThesis(title: "Machine Learning in Swift")
student.enroll(in: "Advanced AI")
student.updateGPA(3.85)

print(student.contactInfo())
```

### 21.2 Animal Hierarchy

```swift
class Animal {
    var name: String
    var species: String
    var age: Int
    var weight: Double  // kg
    var isEndangered: Bool = false
    
    enum DietType {
        case herbivore, carnivore, omnivore
    }
    
    var diet: DietType
    
    init(name: String, species: String, age: Int, weight: Double, diet: DietType) {
        self.name = name
        self.species = species
        self.age = age
        self.weight = weight
        self.diet = diet
    }
    
    func makeSound() -> String { "..." }
    
    func eat(_ food: String) {
        print("\(name) (\(species)) is eating \(food)")
    }
    
    func move() {
        print("\(name) is moving")
    }
    
    var description: String {
        return """
        Name: \(name)
        Species: \(species)
        Age: \(age) years
        Weight: \(weight) kg
        Diet: \(diet)
        Endangered: \(isEndangered)
        """
    }
}

class Mammal: Animal {
    var furColor: String
    var hasLiveYoung: Bool = true
    
    init(name: String, species: String, age: Int, weight: Double, diet: DietType, furColor: String) {
        self.furColor = furColor
        super.init(name: name, species: species, age: age, weight: weight, diet: diet)
    }
    
    func suckle() {
        print("\(name) is nursing young")
    }
}

class Bird: Animal {
    var wingspan: Double  // cm
    var canFly: Bool
    var featherColor: String
    
    init(name: String, species: String, age: Int, weight: Double, diet: DietType,
         wingspan: Double, canFly: Bool, featherColor: String) {
        self.wingspan = wingspan
        self.canFly = canFly
        self.featherColor = featherColor
        super.init(name: name, species: species, age: age, weight: weight, diet: diet)
    }
    
    override func move() {
        if canFly {
            print("\(name) is flying")
        } else {
            print("\(name) is running (cannot fly)")
        }
    }
}

class Lion: Mammal {
    var prideSize: Int
    var isAlphaMale: Bool
    
    init(name: String, age: Int, weight: Double, prideSize: Int, isAlphaMale: Bool = false) {
        self.prideSize = prideSize
        self.isAlphaMale = isAlphaMale
        super.init(name: name, species: "Panthera leo", age: age, weight: weight,
                  diet: .carnivore, furColor: "tawny")
    }
    
    override func makeSound() -> String {
        return "ROAAARRR!"
    }
    
    override func hunt() {
        print("\(name) is stalking prey with the pride of \(prideSize)")
    }
    
    func hunt() {
        print("\(name) is hunting!")
    }
    
    override var description: String {
        return super.description + "\nPride size: \(prideSize)\nAlpha: \(isAlphaMale)"
    }
}

class Penguin: Bird {
    var colony: String
    var swimSpeed: Double  // km/h
    
    init(name: String, age: Int, weight: Double, colony: String, swimSpeed: Double) {
        self.colony = colony
        self.swimSpeed = swimSpeed
        super.init(name: name, species: "Aptenodytes forsteri", age: age, weight: weight,
                  diet: .carnivore, wingspan: 100, canFly: false, featherColor: "black and white")
    }
    
    override func makeSound() -> String {
        return "AAAAK!"
    }
    
    func swim() {
        print("\(name) is swimming at \(swimSpeed) km/h")
    }
}

// Polymorphism
let animals: [Animal] = [
    Lion(name: "Simba", age: 5, weight: 190, prideSize: 12, isAlphaMale: true),
    Penguin(name: "Pingu", age: 3, weight: 30, colony: "Antarctica", swimSpeed: 25),
    Mammal(name: "Kanga", species: "Macropus rufus", age: 4, weight: 50, diet: .herbivore, furColor: "red")
]

for animal in animals {
    print("\(animal.name): \(animal.makeSound())")
    animal.move()
    print()
}

// Type checking
for animal in animals {
    if let lion = animal as? Lion {
        print("Lion: \(lion.isAlphaMale ? "Alpha" : "Regular") male")
    } else if let penguin = animal as? Penguin {
        print("Penguin from \(penguin.colony)")
        penguin.swim()
    }
}
```

### 21.3 Vehicle Hierarchy

```swift
class Vehicle {
    var make: String
    var model: String
    var year: Int
    var color: String
    var mileage: Double = 0  // km
    var fuelLevel: Double = 100  // percentage
    
    var description: String {
        return "\(year) \(make) \(model) (\(color))"
    }
    
    var isRunning: Bool = false
    
    init(make: String, model: String, year: Int, color: String) {
        self.make = make
        self.model = model
        self.year = year
        self.color = color
    }
    
    func start() {
        guard !isRunning else {
            print("\(description) is already running!")
            return
        }
        isRunning = true
        print("\(description) started")
    }
    
    func stop() {
        guard isRunning else {
            print("\(description) is not running!")
            return
        }
        isRunning = false
        print("\(description) stopped")
    }
    
    func drive(distance: Double) {
        guard isRunning else {
            print("Start the vehicle first!")
            return
        }
        mileage += distance
        print("\(description) drove \(distance) km. Total: \(mileage) km")
    }
    
    func refuel(amount: Double) {
        fuelLevel = min(100, fuelLevel + amount)
        print("Refueled to \(fuelLevel)%")
    }
    
    deinit {
        print("Vehicle \(description) removed from registry")
    }
}

class Car: Vehicle {
    var numberOfDoors: Int
    var transmission: String  // automatic, manual
    var horsepower: Int
    
    init(make: String, model: String, year: Int, color: String,
         doors: Int, transmission: String, horsepower: Int) {
        self.numberOfDoors = doors
        self.transmission = transmission
        self.horsepower = horsepower
        super.init(make: make, model: model, year: year, color: color)
    }
    
    func openTrunk() {
        print("\(description) trunk opened")
    }
    
    override var description: String {
        return super.description + " (\(horsepower)hp, \(transmission))"
    }
}

class Truck: Vehicle {
    var payloadCapacity: Double  // tons
    var isLoaded: Bool = false
    var currentLoad: Double = 0
    
    init(make: String, model: String, year: Int, color: String, payloadCapacity: Double) {
        self.payloadCapacity = payloadCapacity
        super.init(make: make, model: model, year: year, color: color)
    }
    
    func loadCargo(weight: Double) -> Bool {
        guard currentLoad + weight <= payloadCapacity else {
            print("Cannot load \(weight) tons: exceeds capacity!")
            return false
        }
        currentLoad += weight
        isLoaded = true
        print("Loaded \(weight) tons. Current load: \(currentLoad) tons")
        return true
    }
    
    func unloadCargo() {
        print("Unloaded \(currentLoad) tons")
        currentLoad = 0
        isLoaded = false
    }
    
    override func drive(distance: Double) {
        let fuelMultiplier = isLoaded ? 1.5 : 1.0  // ใช้น้ำมันมากขึ้นถ้าบรรทุก
        let fuelUsed = distance / 100 * 15 * fuelMultiplier
        fuelLevel = max(0, fuelLevel - fuelUsed)
        super.drive(distance: distance)
        print("Fuel remaining: \(String(format: "%.1f", fuelLevel))%")
    }
}

class ElectricCar: Car {
    var batteryCapacity: Double  // kWh
    var chargeLevel: Double = 100  // percentage
    var estimatedRange: Double {
        return batteryCapacity * chargeLevel / 100 * 6  // km per kWh
    }
    
    init(make: String, model: String, year: Int, color: String, batteryCapacity: Double) {
        self.batteryCapacity = batteryCapacity
        super.init(make: make, model: model, year: year, color: color,
                  doors: 4, transmission: "automatic", horsepower: 450)
    }
    
    func charge(hours: Double) {
        let chargeRate = 20.0  // % per hour
        chargeLevel = min(100, chargeLevel + hours * chargeRate)
        print("Charged to \(chargeLevel)%. Estimated range: \(estimatedRange) km")
    }
    
    override func start() {
        super.start()
        if isRunning {
            print("Running silently on electric power")
        }
    }
    
    override func drive(distance: Double) {
        // ใช้ไฟฟ้าแทนน้ำมัน
        let consumption = distance / estimatedRange * 100
        chargeLevel = max(0, chargeLevel - consumption)
        mileage += distance
        print("\(description) drove \(distance) km. Battery: \(String(format: "%.1f", chargeLevel))%")
    }
    
    override var description: String {
        return "\(year) \(make) \(model) (Electric, \(horsepower)hp)"
    }
}

// ทดสอบ
let sedan = Car(make: "Toyota", model: "Camry", year: 2023, color: "Silver",
                doors: 4, transmission: "automatic", horsepower: 203)
let truck = Truck(make: "Ford", model: "F-150", year: 2023, color: "Red",
                  payloadCapacity: 0.9)
let ev = ElectricCar(make: "Tesla", model: "Model 3", year: 2024, color: "White",
                     batteryCapacity: 82)

// Polymorphism
let vehicles: [Vehicle] = [sedan, truck, ev]

for vehicle in vehicles {
    vehicle.start()
    vehicle.drive(distance: 50)
    vehicle.stop()
    print()
}

// Type casting
for vehicle in vehicles {
    if let electricCar = vehicle as? ElectricCar {
        electricCar.charge(hours: 2)
    } else if let truck = vehicle as? Truck {
        truck.loadCargo(weight: 0.5)
    }
}
```

---

## 22. Practical Exercises

### Exercise 1: Shape Calculator with Hierarchy

```swift
// โจทย์: สร้าง shape hierarchy ที่สมบูรณ์

class Shape {
    var color: String
    var borderWidth: Double
    var isVisible: Bool = true
    
    static var totalShapes: Int = 0
    
    init(color: String, borderWidth: Double = 1.0) {
        self.color = color
        self.borderWidth = borderWidth
        Shape.totalShapes += 1
    }
    
    var area: Double { 0 }
    var perimeter: Double { 0 }
    
    func draw() {
        guard isVisible else { return }
        print("Drawing \(type(of: self)) in \(color)")
    }
    
    func scale(by factor: Double) -> Shape {
        fatalError("Subclasses must implement scale(by:)")
    }
    
    func describe() {
        print("""
        \(type(of: self)):
          Color: \(color)
          Area: \(String(format: "%.2f", area))
          Perimeter: \(String(format: "%.2f", perimeter))
        """)
    }
    
    deinit {
        Shape.totalShapes -= 1
    }
}

class Circle: Shape {
    var radius: Double
    
    init(radius: Double, color: String = "black") {
        self.radius = radius
        super.init(color: color)
    }
    
    override var area: Double { Double.pi * radius * radius }
    override var perimeter: Double { 2 * Double.pi * radius }
    
    override func draw() {
        super.draw()
        print("  ○ Circle with radius \(radius)")
    }
    
    override func scale(by factor: Double) -> Circle {
        return Circle(radius: radius * factor, color: color)
    }
}

class Rectangle: Shape {
    var width: Double
    var height: Double
    
    var isSquare: Bool { width == height }
    
    init(width: Double, height: Double, color: String = "black") {
        self.width = width
        self.height = height
        super.init(color: color)
    }
    
    override var area: Double { width * height }
    override var perimeter: Double { 2 * (width + height) }
    
    override func draw() {
        super.draw()
        let shape = isSquare ? "□ Square" : "▭ Rectangle"
        print("  \(shape): \(width) × \(height)")
    }
    
    override func scale(by factor: Double) -> Rectangle {
        return Rectangle(width: width * factor, height: height * factor, color: color)
    }
}

class Triangle: Shape {
    var base: Double
    var height: Double
    var sideA: Double
    var sideB: Double
    
    init(base: Double, height: Double, sideA: Double, sideB: Double, color: String = "black") {
        self.base = base
        self.height = height
        self.sideA = sideA
        self.sideB = sideB
        super.init(color: color)
    }
    
    convenience init(equilateral side: Double, color: String = "black") {
        let h = side * sqrt(3) / 2
        self.init(base: side, height: h, sideA: side, sideB: side, color: color)
    }
    
    override var area: Double { 0.5 * base * height }
    override var perimeter: Double { base + sideA + sideB }
    
    override func draw() {
        super.draw()
        print("  △ Triangle: base \(base), height \(height)")
    }
    
    override func scale(by factor: Double) -> Triangle {
        return Triangle(base: base * factor, height: height * factor,
                       sideA: sideA * factor, sideB: sideB * factor,
                       color: color)
    }
}

// Canvas ที่จัดการ shapes
class Canvas {
    var name: String
    private var shapes: [Shape] = []
    
    init(name: String) {
        self.name = name
    }
    
    func add(_ shape: Shape) {
        shapes.append(shape)
    }
    
    func removeAll() {
        shapes.removeAll()
    }
    
    var totalArea: Double {
        return shapes.reduce(0) { $0 + $1.area }
    }
    
    func drawAll() {
        print("=== Canvas: \(name) ===")
        for shape in shapes {
            shape.draw()
        }
    }
    
    func describe() {
        print("Canvas '\(name)' with \(shapes.count) shapes, total area: \(String(format: "%.2f", totalArea))")
        for shape in shapes {
            shape.describe()
        }
    }
    
    func findLargest() -> Shape? {
        return shapes.max { $0.area < $1.area }
    }
    
    func shapes<T: Shape>(ofType type: T.Type) -> [T] {
        return shapes.compactMap { $0 as? T }
    }
}

// ทดสอบ
let canvas = Canvas(name: "Main Canvas")
canvas.add(Circle(radius: 5, color: "red"))
canvas.add(Rectangle(width: 10, height: 6, color: "blue"))
canvas.add(Triangle(equilateral: 8, color: "green"))
canvas.add(Rectangle(width: 4, height: 4, color: "yellow"))

canvas.drawAll()
print()

if let largest = canvas.findLargest() {
    print("Largest shape: \(type(of: largest)) with area \(String(format: "%.2f", largest.area))")
}

let rectangles = canvas.shapes(ofType: Rectangle.self)
print("\nRectangles (\(rectangles.count)):")
for rect in rectangles {
    print("  \(rect.width) × \(rect.height) (isSquare: \(rect.isSquare))")
}

print("\nTotal shapes created: \(Shape.totalShapes)")
```

### Exercise 2: Library Management System

```swift
// โจทย์: สร้าง Library Management System

class LibraryItem {
    let itemID: String
    var title: String
    var isAvailable: Bool = true
    var borrowedBy: Member?
    var returnDate: Date?
    
    init(itemID: String, title: String) {
        self.itemID = itemID
        self.title = title
    }
    
    func checkout(by member: Member, for days: Int) -> Bool {
        guard isAvailable else {
            print("'\(title)' is not available")
            return false
        }
        isAvailable = false
        borrowedBy = member
        returnDate = Calendar.current.date(byAdding: .day, value: days, to: Date())
        print("\(member.name) checked out '\(title)'")
        return true
    }
    
    func returnItem() {
        guard let member = borrowedBy else {
            print("Item was not borrowed")
            return
        }
        print("\(member.name) returned '\(title)'")
        isAvailable = true
        borrowedBy = nil
        returnDate = nil
    }
    
    var status: String {
        if isAvailable {
            return "Available"
        } else if let member = borrowedBy {
            return "Borrowed by \(member.name)"
        }
        return "Unknown"
    }
    
    deinit {
        print("Item '\(title)' removed from system")
    }
}

class Book: LibraryItem {
    var author: String
    var isbn: String
    var pages: Int
    var genre: String
    
    init(itemID: String, title: String, author: String, isbn: String, pages: Int, genre: String) {
        self.author = author
        self.isbn = isbn
        self.pages = pages
        self.genre = genre
        super.init(itemID: itemID, title: title)
    }
    
    override func checkout(by member: Member, for days: Int = 14) -> Bool {
        return super.checkout(by: member, for: days)
    }
    
    var description: String {
        return "'\(title)' by \(author) (\(pages) pages)"
    }
}

class DVD: LibraryItem {
    var director: String
    var duration: Int  // minutes
    var rating: String
    
    init(itemID: String, title: String, director: String, duration: Int, rating: String) {
        self.director = director
        self.duration = duration
        self.rating = rating
        super.init(itemID: itemID, title: title)
    }
    
    override func checkout(by member: Member, for days: Int = 7) -> Bool {
        return super.checkout(by: member, for: days)
    }
}

class Member {
    let memberID: String
    var name: String
    var email: String
    private(set) var borrowedItems: [LibraryItem] = []
    var maxBorrowLimit: Int = 5
    
    init(memberID: String, name: String, email: String) {
        self.memberID = memberID
        self.name = name
        self.email = email
    }
    
    func borrow(_ item: LibraryItem, for days: Int) -> Bool {
        guard borrowedItems.count < maxBorrowLimit else {
            print("\(name) has reached borrow limit (\(maxBorrowLimit))")
            return false
        }
        
        if item.checkout(by: self, for: days) {
            borrowedItems.append(item)
            return true
        }
        return false
    }
    
    func returnItem(_ item: LibraryItem) {
        item.returnItem()
        borrowedItems.removeAll { $0 === item }
    }
    
    func listBorrowed() {
        print("\(name)'s borrowed items (\(borrowedItems.count)/\(maxBorrowLimit)):")
        for item in borrowedItems {
            let returnStr = item.returnDate.map { " - due \($0)" } ?? ""
            print("  - \(item.title)\(returnStr)")
        }
    }
}

class Library {
    var name: String
    private var items: [LibraryItem] = []
    private var members: [Member] = []
    
    init(name: String) {
        self.name = name
    }
    
    func addItem(_ item: LibraryItem) {
        items.append(item)
    }
    
    func registerMember(_ member: Member) {
        members.append(member)
        print("\(member.name) registered at \(name)")
    }
    
    func findItem(title: String) -> LibraryItem? {
        return items.first { $0.title.lowercased().contains(title.lowercased()) }
    }
    
    func availableBooks() -> [Book] {
        return items.compactMap { $0 as? Book }.filter { $0.isAvailable }
    }
    
    func availableDVDs() -> [DVD] {
        return items.compactMap { $0 as? DVD }.filter { $0.isAvailable }
    }
    
    func generateReport() {
        let totalItems = items.count
        let available = items.filter { $0.isAvailable }.count
        let borrowed = totalItems - available
        
        print("=== \(name) Library Report ===")
        print("Total Items: \(totalItems)")
        print("Available: \(available)")
        print("Borrowed: \(borrowed)")
        print("Registered Members: \(members.count)")
        
        if borrowed > 0 {
            print("\nCurrently Borrowed:")
            for item in items where !item.isAvailable {
                print("  - \(item.title): \(item.status)")
            }
        }
    }
}

// ทดสอบ
let library = Library(name: "Bangkok Public Library")

// เพิ่มหนังสือ
library.addItem(Book(itemID: "B001", title: "Swift Programming", author: "Apple", isbn: "978-0-12345-678-9", pages: 500, genre: "Technology"))
library.addItem(Book(itemID: "B002", title: "Design Patterns", author: "Gang of Four", isbn: "978-0-20163-361-0", pages: 395, genre: "Technology"))
library.addItem(DVD(itemID: "D001", title: "WWDC 2023", director: "Apple", duration: 120, rating: "G"))

// ลงทะเบียนสมาชิก
let alice = Member(memberID: "M001", name: "Alice", email: "alice@email.com")
let bob = Member(memberID: "M002", name: "Bob", email: "bob@email.com")
library.registerMember(alice)
library.registerMember(bob)

// ยืม-คืน
if let swiftBook = library.findItem(title: "Swift") {
    alice.borrow(swiftBook, for: 14)
}

if let dvd = library.findItem(title: "WWDC") {
    bob.borrow(dvd, for: 7)
}

alice.listBorrowed()
bob.listBorrowed()
library.generateReport()
```

---

## 23. Summary

### สิ่งที่ได้เรียนรู้ในบทนี้

1. **Class คืออะไร**: Reference type ที่มี inheritance และ ARC

2. **Reference Type Semantics**:
   - หลาย variables ชี้ไปยัง object เดียวกัน
   - การเปลี่ยน object จาก reference หนึ่ง กระทบทุก reference

3. **Initializers 3 ประเภท**:
   - **Designated**: initializer หลัก initialize ทุก property
   - **Convenience**: เรียก designated initializer
   - **Required**: ทุก subclass ต้อง implement

4. **Deinitializer**: ทำความสะอาดทรัพยากรก่อน deallocate

5. **Inheritance**: ส่บทอด properties และ methods จาก parent class

6. **Overriding**: ใช้ `override` เพื่อเขียน method/property ใหม่

7. **final**: ป้องกันการ override หรือ subclassing

8. **Type Casting**: `is`, `as`, `as?`, `as!` สำหรับตรวจสอบและแปลง type

9. **ARC**: Automatic Reference Counting จัดการ memory อัตโนมัติ

10. **Retain Cycles**: ป้องกันด้วย `weak` และ `unowned`

### Quick Reference

```swift
// ✅ Class Checklist

// 1. Designated init - initialize ทุก stored property
class MyClass {
    var value: Int
    
    init(value: Int) {
        self.value = value  // designated
    }
    
    // 2. Convenience init - delegate to designated
    convenience init() {
        self.init(value: 0)  // convenience
    }
    
    // 3. Deinit - cleanup
    deinit {
        print("Cleaning up")
    }
}

// 4. Inheritance
class Subclass: MyClass {
    var extra: String
    
    init(value: Int, extra: String) {
        self.extra = extra      // initialize own properties first
        super.init(value: value) // then call super
    }
    
    // 5. Override
    override var description: String {
        return "Subclass: \(value), \(extra)"
    }
    
    var description: String { "Base" }
}

// 6. Type casting
func process(_ obj: MyClass) {
    if let sub = obj as? Subclass {
        print("Subclass: \(sub.extra)")
    }
}

// 7. Memory management
class Owner {
    weak var delegate: SomeProtocol?  // weak for optional
    unowned let parent: Owner         // unowned for guaranteed lifetime
    
    init(parent: Owner) {
        self.parent = parent
    }
}

protocol SomeProtocol: AnyObject {}
```

### Struct vs Class ตัดสินใจ

```
โครงสร้างข้อมูลง่ายๆ → Struct
  ✅ Point, Size, Color, Range
  ✅ Model data (User, Product)
  ✅ Configuration

มี inheritance → Class
  ✅ UIViewController, UIView
  ✅ Animal hierarchy
  ✅ Vehicle hierarchy

Shared mutable state → Class
  ✅ NetworkManager (singleton)
  ✅ DatabaseManager
  ✅ EventBus

ต้องการ lifecycle → Class
  ✅ File handles
  ✅ Network connections
  ✅ Timer management
```

---

> **ถัดไป**: [Part 15: Protocols](Part15_Protocols.md) - เรียนรู้เกี่ยวกับ Protocol-Oriented Programming

---

*เนื้อหานี้เป็นส่วนหนึ่งของ Swift Programming Course ภาษาไทย*
