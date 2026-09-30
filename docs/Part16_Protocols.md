# Part 16: Protocols ใน Swift

## บทนำ

Protocol คือหนึ่งในคุณสมบัติที่ทรงพลังที่สุดของ Swift ซึ่งเป็นแกนหลักของ **Protocol-Oriented Programming (POP)** — แนวคิดที่ Apple นำเสนอใน WWDC 2015 Protocol กำหนด "สัญญา" ว่า Type ที่ adopt Protocol ต้องทำอะไรได้บ้าง โดยไม่สนใจว่าจะ implement อย่างไร

ใน Swift ทุก Type — Class, Struct, Enum — สามารถ adopt Protocol ได้ ซึ่งต่างจาก Inheritance ที่ใช้ได้เฉพาะ Class เท่านั้น

ในบทนี้เราจะเรียนรู้:
- พื้นฐาน Protocol และ syntax
- Protocol requirements ต่างๆ
- Protocol extensions และ default implementations
- Associated types
- Protocol-Oriented Programming
- Standard Library protocols ที่ใช้บ่อย
- แบบฝึกหัดและตัวอย่างจริง

---

## 16.1 Protocol คืออะไร?

### คำนิยาม

Protocol คือ **blueprint** ที่กำหนดชุดของ method, property, และ requirements อื่นๆ ที่เหมาะสมกับงานหรือฟังก์ชันการทำงานบางอย่าง จากนั้น Class, Struct, หรือ Enum สามารถ **adopt** Protocol นั้นเพื่อ implement requirements เหล่านั้น

คิดว่า Protocol เป็น "interface" หรือ "contract" ที่กำหนดว่า "ต้องทำอะไรได้" โดยไม่บอกว่า "ทำอย่างไร"

```swift
// Protocol กำหนด "สัญญา"
protocol Greetable {
    var name: String { get }
    func greet() -> String
}

// Class adopt Protocol
class Person: Greetable {
    var name: String
    var age: Int
    
    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }
    
    func greet() -> String {
        return "สวัสดีครับ ผมชื่อ \(name)"
    }
}

// Struct adopt Protocol
struct Robot: Greetable {
    var name: String
    var model: String
    
    func greet() -> String {
        return "Hello, I am \(name) model \(model)"
    }
}

// Enum adopt Protocol
enum Language: Greetable {
    case thai
    case english
    case japanese
    
    var name: String {
        switch self {
        case .thai: return "ภาษาไทย"
        case .english: return "English"
        case .japanese: return "日本語"
        }
    }
    
    func greet() -> String {
        switch self {
        case .thai: return "สวัสดีครับ/ค่ะ"
        case .english: return "Hello!"
        case .japanese: return "こんにちは"
        }
    }
}

// ทุก Type สามารถถูกใช้เป็น Greetable
let greetables: [Greetable] = [
    Person(name: "สมชาย", age: 30),
    Robot(name: "R2D2", model: "Astromech"),
    Language.thai,
    Language.english
]

for item in greetables {
    print(item.greet())
}
```

### Protocol vs Inheritance

| คุณสมบัติ | Protocol | Inheritance |
|-----------|----------|-------------|
| ใช้กับ | Class, Struct, Enum | Class เท่านั้น |
| หลาย Protocol/Superclass | หลาย Protocol ได้ | Single Inheritance |
| Implementation | ไม่มี (default ได้ผ่าน extension) | มีใน Superclass |
| Value Types | ✅ | ❌ |
| "สัญญา" | ✅ | ❌ (ส่วนใหญ่) |

---

## 16.2 Protocol Syntax

### รูปแบบพื้นฐาน

```swift
protocol ProtocolName {
    // Property requirements
    var propertyName: Type { get }
    var anotherProperty: Type { get set }
    
    // Method requirements
    func methodName() -> ReturnType
    func methodWithParams(param: Type) -> ReturnType
    
    // Initializer requirements
    init(param: Type)
    
    // Subscript requirements
    subscript(index: Int) -> Type { get }
}
```

### ตัวอย่าง Protocol ง่ายๆ

```swift
// Protocol สำหรับ Printable object
protocol Describable {
    var description: String { get }
    func printDescription()
}

// Default implementation ผ่าน extension
extension Describable {
    func printDescription() {
        print(description)
    }
}

struct Point: Describable {
    var x: Double
    var y: Double
    
    var description: String {
        return "จุด (\(x), \(y))"
    }
}

struct Size: Describable {
    var width: Double
    var height: Double
    
    var description: String {
        return "ขนาด \(width) x \(height)"
    }
}

let point = Point(x: 3, y: 4)
let size = Size(width: 100, height: 50)

point.printDescription()  // ใช้ default implementation
size.printDescription()   // ใช้ default implementation
```

---

## 16.3 Protocol Requirements

### 16.3.1 Property Requirements

```swift
protocol PropertyDemo {
    // gettable only property
    var readOnly: String { get }
    
    // gettable and settable property
    var readWrite: Int { get set }
    
    // Computed หรือ Stored property ก็ได้
    var name: String { get }
}

// Stored properties
struct ConcreteA: PropertyDemo {
    let readOnly: String  // let satisfies { get }
    var readWrite: Int
    var name: String
}

// Computed properties
struct ConcreteB: PropertyDemo {
    private var _value: Int = 0
    
    var readOnly: String { return "constant value" }
    
    var readWrite: Int {
        get { return _value }
        set { _value = newValue }
    }
    
    var name: String { return "ConcreteB" }
}
```

### 16.3.2 Method Requirements

```swift
protocol Calculable {
    // Instance methods
    func add(_ a: Double, _ b: Double) -> Double
    func subtract(_ a: Double, _ b: Double) -> Double
    
    // Mutating method (สำหรับ Struct/Enum)
    mutating func reset()
    
    // Static method
    static func zero() -> Self
}

struct SimpleCalc: Calculable {
    var currentValue: Double = 0
    
    func add(_ a: Double, _ b: Double) -> Double { return a + b }
    func subtract(_ a: Double, _ b: Double) -> Double { return a - b }
    
    mutating func reset() {
        currentValue = 0
    }
    
    static func zero() -> SimpleCalc {
        return SimpleCalc(currentValue: 0)
    }
}

// Class ไม่ต้องใช้ mutating
class AdvancedCalc: Calculable {
    var currentValue: Double = 0
    
    func add(_ a: Double, _ b: Double) -> Double { return a + b }
    func subtract(_ a: Double, _ b: Double) -> Double { return a - b }
    func reset() { currentValue = 0 }  // ไม่ต้อง mutating สำหรับ class
    
    static func zero() -> AdvancedCalc {
        return AdvancedCalc()
    }
}
```

### 16.3.3 Initializer Requirements

```swift
protocol Configurable {
    init(config: [String: Any])
    init()
}

class ServerConfig: Configurable {
    var host: String
    var port: Int
    var timeout: TimeInterval
    
    required init(config: [String: Any]) {
        self.host = config["host"] as? String ?? "localhost"
        self.port = config["port"] as? Int ?? 8080
        self.timeout = config["timeout"] as? TimeInterval ?? 30
    }
    
    required init() {
        self.host = "localhost"
        self.port = 8080
        self.timeout = 30
    }
}

// Subclass ต้อง implement required initializers
class SecureServerConfig: ServerConfig {
    var sslEnabled: Bool
    
    required init(config: [String: Any]) {
        self.sslEnabled = config["ssl"] as? Bool ?? true
        super.init(config: config)
    }
    
    required init() {
        self.sslEnabled = true
        super.init()
    }
}

let config = SecureServerConfig(config: ["host": "example.com", "port": 443, "ssl": true])
print("Host: \(config.host), Port: \(config.port), SSL: \(config.sslEnabled)")
```

### 16.3.4 Subscript Requirements

```swift
protocol Container {
    associatedtype Element
    subscript(index: Int) -> Element { get }
    var count: Int { get }
}

struct Stack<T>: Container {
    private var items: [T] = []
    
    mutating func push(_ item: T) {
        items.append(item)
    }
    
    mutating func pop() -> T? {
        return items.popLast()
    }
    
    subscript(index: Int) -> T {
        return items[index]
    }
    
    var count: Int { return items.count }
}

var stack = Stack<Int>()
stack.push(10)
stack.push(20)
stack.push(30)

print("จำนวน: \(stack.count)")
print("รายการที่ 1: \(stack[1])")  // 20
```

---

## 16.4 การ Adopt Protocols

### Adopting Single Protocol

```swift
protocol Flyable {
    var maxAltitude: Double { get }
    var currentAltitude: Double { get set }
    
    mutating func takeOff()
    mutating func land()
    mutating func fly(to altitude: Double)
}

struct Airplane: Flyable {
    let model: String
    let maxAltitude: Double = 12000  // เมตร
    var currentAltitude: Double = 0
    
    mutating func takeOff() {
        currentAltitude = 300
        print("\(model) ขึ้นบิน ที่ความสูง \(currentAltitude) เมตร")
    }
    
    mutating func land() {
        currentAltitude = 0
        print("\(model) ลงจอดแล้ว")
    }
    
    mutating func fly(to altitude: Double) {
        let clampedAltitude = min(altitude, maxAltitude)
        currentAltitude = clampedAltitude
        print("\(model) บินอยู่ที่ความสูง \(currentAltitude) เมตร")
    }
}
```

### Adopting Multiple Protocols

```swift
protocol Swimmable {
    var maxDepth: Double { get }
    mutating func dive(to depth: Double)
    mutating func surface()
}

protocol Runnable {
    var maxSpeed: Double { get }
    mutating func run(speed: Double)
    mutating func stop()
}

// ใน Swift เราสามารถ adopt หลาย Protocol ได้
struct Triathlete: Swimmable, Runnable, Flyable {
    var name: String
    
    // Flyable
    let maxAltitude: Double = 0  // ไม่บิน แต่ต้อง implement
    var currentAltitude: Double = 0
    
    // Swimmable
    let maxDepth: Double = 5
    
    // Runnable
    let maxSpeed: Double = 20  // km/h
    var currentSpeed: Double = 0
    var currentDepth: Double = 0
    
    mutating func takeOff() { print("\(name) ไม่บิน") }
    mutating func land() { print("\(name) ไม่ลง") }
    mutating func fly(to altitude: Double) { print("\(name) ไม่บิน") }
    
    mutating func dive(to depth: Double) {
        currentDepth = min(depth, maxDepth)
        print("\(name) ดำน้ำลึก \(currentDepth) เมตร")
    }
    
    mutating func surface() {
        currentDepth = 0
        print("\(name) ขึ้นจากน้ำ")
    }
    
    mutating func run(speed: Double) {
        currentSpeed = min(speed, maxSpeed)
        print("\(name) วิ่งด้วยความเร็ว \(currentSpeed) km/h")
    }
    
    mutating func stop() {
        currentSpeed = 0
        print("\(name) หยุดวิ่ง")
    }
}

var athlete = Triathlete(name: "สมชาย")
athlete.dive(to: 3)
athlete.surface()
athlete.run(speed: 15)
athlete.stop()
```

---

## 16.5 Protocol Conformance

### Explicit Conformance

```swift
protocol JSONSerializable {
    func toJSON() -> [String: Any]
    static func fromJSON(_ json: [String: Any]) -> Self?
}

struct User: JSONSerializable {
    var id: Int
    var username: String
    var email: String
    
    func toJSON() -> [String: Any] {
        return [
            "id": id,
            "username": username,
            "email": email
        ]
    }
    
    static func fromJSON(_ json: [String: Any]) -> User? {
        guard let id = json["id"] as? Int,
              let username = json["username"] as? String,
              let email = json["email"] as? String else {
            return nil
        }
        return User(id: id, username: username, email: email)
    }
}

let user = User(id: 1, username: "somchai", email: "somchai@example.com")
let json = user.toJSON()
print("JSON: \(json)")

if let reconstructed = User.fromJSON(json) {
    print("Reconstructed: \(reconstructed.username)")
}
```

### Retroactive Conformance (Extension)

```swift
// เพิ่ม Protocol conformance ให้กับ Type ที่มีอยู่แล้ว
struct Coordinate {
    var latitude: Double
    var longitude: Double
}

// เพิ่ม conformance ในภายหลังผ่าน extension
extension Coordinate: CustomStringConvertible {
    var description: String {
        return "(\(latitude)°N, \(longitude)°E)"
    }
}

extension Coordinate: JSONSerializable {
    func toJSON() -> [String: Any] {
        return ["lat": latitude, "lng": longitude]
    }
    
    static func fromJSON(_ json: [String: Any]) -> Coordinate? {
        guard let lat = json["lat"] as? Double,
              let lng = json["lng"] as? Double else { return nil }
        return Coordinate(latitude: lat, longitude: lng)
    }
}

let bangkok = Coordinate(latitude: 13.7563, longitude: 100.5018)
print(bangkok)  // ใช้ CustomStringConvertible
print(bangkok.toJSON())
```

---

## 16.6 Protocol เป็น Type

Protocol สามารถใช้เป็น Type ได้ในหลายบริบท

```swift
protocol Drawable {
    func draw()
    var bounds: (width: Double, height: Double) { get }
}

// ใช้ Protocol เป็น type ของ variable
var drawableObject: Drawable

struct Square: Drawable {
    var side: Double
    var bounds: (width: Double, height: Double) { return (side, side) }
    func draw() { print("วาดสี่เหลี่ยมจัตุรัส ด้าน \(side)") }
}

struct Triangle: Drawable {
    var base: Double
    var height: Double
    var bounds: (width: Double, height: Double) { return (base, height) }
    func draw() { print("วาดสามเหลี่ยม ฐาน \(base) สูง \(height)") }
}

// ใช้เป็น type ของ Array
var shapes: [Drawable] = [
    Square(side: 5),
    Triangle(base: 4, height: 3),
    Square(side: 2)
]

// ใช้เป็น return type
func createShape(type: String) -> Drawable {
    if type == "square" {
        return Square(side: 10)
    } else {
        return Triangle(base: 6, height: 8)
    }
}

// ใช้เป็น parameter type
func printShapeInfo(_ shape: Drawable) {
    shape.draw()
    print("  ขนาด: \(shape.bounds.width) x \(shape.bounds.height)")
}

for shape in shapes {
    printShapeInfo(shape)
}

// ใช้เป็น Dictionary value
var drawingRegistry: [String: Drawable] = [
    "square1": Square(side: 5),
    "triangle1": Triangle(base: 3, height: 4)
]
```

---

## 16.7 Protocol Composition (&)

เชื่อม Protocol หลายตัวเพื่อกำหนด requirement ที่เจาะจงมากขึ้น

```swift
protocol Named {
    var name: String { get }
}

protocol Aged {
    var age: Int { get }
}

protocol Employable {
    var jobTitle: String { get }
    var salary: Double { get }
}

struct Employee: Named, Aged, Employable {
    var name: String
    var age: Int
    var jobTitle: String
    var salary: Double
}

// Protocol Composition - ต้องการทั้ง Named และ Aged
func greetPerson(_ person: Named & Aged) {
    print("สวัสดี \(person.name) อายุ \(person.age) ปี")
}

// ต้องการทั้งสาม Protocol
func processEmployee(_ emp: Named & Aged & Employable) {
    print("\(emp.name) ตำแหน่ง: \(emp.jobTitle) เงินเดือน: \(emp.salary) บาท")
}

let emp = Employee(name: "สมชาย", age: 30, jobTitle: "Developer", salary: 60000)
greetPerson(emp)
processEmployee(emp)

// Type alias สำหรับ Protocol Composition ที่ใช้บ่อย
typealias PersonInfo = Named & Aged
typealias EmployeeInfo = Named & Aged & Employable

func displayPersonInfo(_ info: PersonInfo) {
    print("\(info.name), \(info.age) ปี")
}

displayPersonInfo(emp)

// ใช้กับ Array
let employees: [EmployeeInfo] = [
    Employee(name: "สมชาย", age: 30, jobTitle: "Developer", salary: 60000),
    Employee(name: "สมหญิง", age: 28, jobTitle: "Designer", salary: 55000)
]

let highEarners = employees.filter { $0.salary >= 58000 }
highEarners.forEach { print("\($0.name): \($0.salary) บาท") }
```

---

## 16.8 Protocol Extensions

Protocol extensions ช่วยให้เพิ่ม implementation ลงใน Protocol ได้

```swift
protocol Validatable {
    var isValid: Bool { get }
    var validationErrors: [String] { get }
}

// Extension เพิ่ม method ให้ทุก Type ที่ adopt Validatable
extension Validatable {
    func validate() -> Bool {
        if !isValid {
            print("Validation ล้มเหลว:")
            validationErrors.forEach { print("  - \($0)") }
        }
        return isValid
    }
    
    func assertValid() {
        precondition(isValid, "Object ไม่ valid: \(validationErrors.joined(separator: ", "))")
    }
}

struct EmailAddress: Validatable {
    var address: String
    
    var isValid: Bool {
        return address.contains("@") && address.contains(".")
    }
    
    var validationErrors: [String] {
        var errors: [String] = []
        if !address.contains("@") { errors.append("ต้องมี @") }
        if !address.contains(".") { errors.append("ต้องมี . (dot)") }
        if address.count < 5 { errors.append("Email สั้นเกินไป") }
        return errors
    }
}

struct PasswordField: Validatable {
    var password: String
    
    var isValid: Bool {
        return password.count >= 8 && 
               password.contains(where: { $0.isUppercase }) &&
               password.contains(where: { $0.isNumber })
    }
    
    var validationErrors: [String] {
        var errors: [String] = []
        if password.count < 8 { errors.append("รหัสผ่านต้องมีอย่างน้อย 8 ตัวอักษร") }
        if !password.contains(where: { $0.isUppercase }) {
            errors.append("ต้องมีตัวอักษรพิมพ์ใหญ่อย่างน้อย 1 ตัว")
        }
        if !password.contains(where: { $0.isNumber }) {
            errors.append("ต้องมีตัวเลขอย่างน้อย 1 ตัว")
        }
        return errors
    }
}

let email = EmailAddress(address: "test@example.com")
print("Email valid: \(email.validate())")

let badEmail = EmailAddress(address: "notanemail")
print("Bad email valid: \(badEmail.validate())")

let password = PasswordField(password: "MyPass123")
print("Password valid: \(password.validate())")

let weakPassword = PasswordField(password: "weak")
weakPassword.validate()
```

---

## 16.9 Default Implementations

```swift
protocol Sorting {
    associatedtype Element: Comparable
    var elements: [Element] { get }
    
    // Method ที่ต้อง implement
    mutating func sort()
    
    // Method ที่มี default implementation
    func sorted() -> [Element]
    var minimum: Element? { get }
    var maximum: Element? { get }
}

extension Sorting {
    // Default implementations
    func sorted() -> [Element] {
        return elements.sorted()
    }
    
    var minimum: Element? {
        return elements.min()
    }
    
    var maximum: Element? {
        return elements.max()
    }
}

struct NumberList: Sorting {
    var elements: [Int]
    
    // ต้อง implement sort()
    mutating func sort() {
        elements.sort()
    }
    
    // ไม่ต้อง implement sorted(), minimum, maximum (มี default แล้ว)
}

var numbers = NumberList(elements: [3, 1, 4, 1, 5, 9, 2, 6])
print("Unsorted: \(numbers.elements)")
print("Sorted copy: \(numbers.sorted())")
print("Min: \(numbers.minimum ?? 0)")
print("Max: \(numbers.maximum ?? 0)")
numbers.sort()
print("After sort: \(numbers.elements)")
```

### Default Implementation กับ Override

```swift
protocol Greeting {
    var language: String { get }
    func hello() -> String
    func goodbye() -> String
    func greetAndFarewell(name: String) -> String
}

extension Greeting {
    // Default implementation
    func greetAndFarewell(name: String) -> String {
        return "\(hello()), \(name)! ... \(goodbye())"
    }
}

struct ThaiGreeting: Greeting {
    var language: String = "ไทย"
    func hello() -> String { return "สวัสดีครับ" }
    func goodbye() -> String { return "ลาก่อนครับ" }
    // ใช้ default implementation ของ greetAndFarewell
}

struct EnglishGreeting: Greeting {
    var language: String = "English"
    func hello() -> String { return "Hello" }
    func goodbye() -> String { return "Goodbye" }
    
    // Override default implementation
    func greetAndFarewell(name: String) -> String {
        return "\(hello()), \(name)! Have a great day. \(goodbye())!"
    }
}

let thai = ThaiGreeting()
let eng = EnglishGreeting()

print(thai.greetAndFarewell(name: "สมชาย"))
print(eng.greetAndFarewell(name: "John"))
```

---

## 16.10 Protocol Inheritance

Protocol สามารถสืบทอดจาก Protocol อื่นได้

```swift
protocol Vehicle {
    var make: String { get }
    var model: String { get }
    var year: Int { get }
    var fuelType: String { get }
}

protocol MotorVehicle: Vehicle {
    var horsepower: Int { get }
    var torque: Double { get }
    func startEngine()
    func stopEngine()
}

protocol ElectricVehicle: Vehicle {
    var batteryCapacity: Double { get }
    var chargeLevel: Double { get set }
    func charge()
    func checkRange() -> Double
}

// ต้อง implement ทั้ง MotorVehicle และ Vehicle
struct GasCar: MotorVehicle {
    var make: String
    var model: String
    var year: Int
    var fuelType: String = "น้ำมัน"
    var horsepower: Int
    var torque: Double
    var isRunning: Bool = false
    
    mutating func startEngine() {
        isRunning = true
        print("\(make) \(model) สตาร์ทเครื่องยนต์แล้ว")
    }
    
    mutating func stopEngine() {
        isRunning = false
        print("\(make) \(model) ดับเครื่องยนต์แล้ว")
    }
}

struct EVCar: ElectricVehicle {
    var make: String
    var model: String
    var year: Int
    var fuelType: String = "ไฟฟ้า"
    var batteryCapacity: Double
    var chargeLevel: Double = 80.0
    
    mutating func charge() {
        chargeLevel = 100.0
        print("ชาร์จ \(make) \(model) เต็มแล้ว")
    }
    
    func checkRange() -> Double {
        return batteryCapacity * chargeLevel / 100 * 6.0
    }
}

// Hybrid Protocol
protocol HybridVehicle: MotorVehicle, ElectricVehicle {
    var mode: String { get set }
    func switchMode(to mode: String)
}

struct HybridCar: HybridVehicle {
    var make: String
    var model: String
    var year: Int
    var fuelType: String = "Hybrid"
    var horsepower: Int
    var torque: Double
    var batteryCapacity: Double
    var chargeLevel: Double = 50.0
    var mode: String = "auto"
    var isRunning: Bool = false
    
    mutating func startEngine() {
        isRunning = true
        print("Hybrid mode เริ่มทำงาน")
    }
    
    mutating func stopEngine() {
        isRunning = false
        print("Hybrid mode หยุดทำงาน")
    }
    
    mutating func charge() {
        chargeLevel = 100.0
    }
    
    func checkRange() -> Double {
        return batteryCapacity * chargeLevel / 100 * 4.0 + 500  // บวกระยะจากน้ำมัน
    }
    
    mutating func switchMode(to newMode: String) {
        self.mode = newMode
        print("เปลี่ยนโหมดเป็น: \(newMode)")
    }
}

var hybrid = HybridCar(make: "Toyota", model: "Prius", year: 2024,
                        horsepower: 121, torque: 142, batteryCapacity: 8.8)
hybrid.startEngine()
hybrid.switchMode(to: "EV")
print("ระยะทาง: \(hybrid.checkRange()) km")
```

---

## 16.11 Conditional Conformance

Type สามารถ conform Protocol ได้เฉพาะเมื่อตรงตามเงื่อนไข

```swift
// Generic type conform Protocol เฉพาะเมื่อ Element conform Equatable
struct Bag<Element> {
    private var items: [Element] = []
    
    mutating func insert(_ item: Element) {
        items.append(item)
    }
    
    func contains(_ item: Element) -> Bool where Element: Equatable {
        return items.contains(item)
    }
    
    var count: Int { return items.count }
}

// Bag conform Equatable เฉพาะเมื่อ Element conform Equatable
extension Bag: Equatable where Element: Equatable {
    static func == (lhs: Bag<Element>, rhs: Bag<Element>) -> Bool {
        return lhs.items == rhs.items
    }
}

var bag1 = Bag<Int>()
bag1.insert(1)
bag1.insert(2)
bag1.insert(3)

var bag2 = Bag<Int>()
bag2.insert(1)
bag2.insert(2)
bag2.insert(3)

print("bag1 == bag2: \(bag1 == bag2)")  // true - conditional conformance

// Array conditional conformance
extension Array: JSONSerializable where Element: JSONSerializable {
    func toJSON() -> [String: Any] {
        return ["items": self.map { $0.toJSON() }]
    }
    
    static func fromJSON(_ json: [String: Any]) -> [Element]? {
        guard let items = json["items"] as? [[String: Any]] else { return nil }
        return items.compactMap { Element.fromJSON($0) }
    }
}

// Optional conditional conformance
extension Optional: JSONSerializable where Wrapped: JSONSerializable {
    func toJSON() -> [String: Any] {
        switch self {
        case .some(let value):
            return ["value": value.toJSON(), "hasValue": true]
        case .none:
            return ["hasValue": false]
        }
    }
    
    static func fromJSON(_ json: [String: Any]) -> Optional<Wrapped>? {
        guard let hasValue = json["hasValue"] as? Bool else { return nil }
        if hasValue, let valueJSON = json["value"] as? [String: Any] {
            return Wrapped.fromJSON(valueJSON)
        }
        return .none
    }
}
```

---

## 16.12 Associated Types

Associated types กำหนด placeholder type ใน Protocol

```swift
// Protocol ที่มี associated type
protocol Collection2 {
    associatedtype Element
    associatedtype Index: Comparable
    
    var startIndex: Index { get }
    var endIndex: Index { get }
    func element(at index: Index) -> Element
    func index(after i: Index) -> Index
}

// Concrete implementation
struct CircularBuffer<T>: Collection2 {
    typealias Element = T
    typealias Index = Int
    
    private var buffer: [T]
    private var readIndex: Int = 0
    private var writeIndex: Int = 0
    private var count: Int = 0
    private let capacity: Int
    
    init(capacity: Int) {
        self.capacity = capacity
        self.buffer = Array(repeating: Optional<T>.none as! T, count: capacity)
    }
    
    var startIndex: Int { return readIndex }
    var endIndex: Int { return writeIndex }
    
    func element(at index: Int) -> T {
        return buffer[index % capacity]
    }
    
    func index(after i: Int) -> Int {
        return i + 1
    }
}

// ตัวอย่างที่เข้าใจง่ายกว่า
protocol Stack2 {
    associatedtype Element
    
    mutating func push(_ element: Element)
    mutating func pop() -> Element?
    func peek() -> Element?
    var isEmpty: Bool { get }
    var count: Int { get }
}

// Int Stack
struct IntStack: Stack2 {
    typealias Element = Int  // explicit type alias (ไม่จำเป็นถ้า infer ได้)
    private var items: [Int] = []
    
    mutating func push(_ element: Int) { items.append(element) }
    mutating func pop() -> Int? { return items.popLast() }
    func peek() -> Int? { return items.last }
    var isEmpty: Bool { return items.isEmpty }
    var count: Int { return items.count }
}

// Generic Stack
struct GenericStack<T>: Stack2 {
    private var items: [T] = []
    
    mutating func push(_ element: T) { items.append(element) }
    mutating func pop() -> T? { return items.popLast() }
    func peek() -> T? { return items.last }
    var isEmpty: Bool { return items.isEmpty }
    var count: Int { return items.count }
}

var intStack = IntStack()
intStack.push(1)
intStack.push(2)
intStack.push(3)
print("Top: \(intStack.peek() ?? 0)")  // 3
print("Popped: \(intStack.pop() ?? 0)")  // 3

var stringStack = GenericStack<String>()
stringStack.push("Hello")
stringStack.push("World")
print("String top: \(stringStack.peek() ?? "")")
```

---

## 16.13 Type Constraints

จำกัด Generic type ให้ต้อง conform Protocol ที่กำหนด

```swift
// ฟังก์ชันที่รับ type ที่ต้อง conform Comparable
func findMax<T: Comparable>(in array: [T]) -> T? {
    return array.max()
}

func findMin<T: Comparable>(in array: [T]) -> T? {
    return array.min()
}

print(findMax(in: [3, 1, 4, 1, 5, 9, 2, 6]) ?? 0)  // 9
print(findMax(in: ["banana", "apple", "cherry"]) ?? "")  // cherry

// Multiple constraints
func processItem<T>(item: T) where T: Comparable & Hashable & CustomStringConvertible {
    print("Processing: \(item.description)")
}

// Constraint บน associated type
protocol DataSource {
    associatedtype Item: Equatable & Codable
    func fetch(id: Int) -> Item?
    func fetchAll() -> [Item]
}

struct UserDataSource: DataSource {
    typealias Item = User
    
    private var users: [Int: User] = [
        1: User(id: 1, username: "admin", email: "admin@example.com"),
        2: User(id: 2, username: "user1", email: "user1@example.com")
    ]
    
    func fetch(id: Int) -> User? {
        return users[id]
    }
    
    func fetchAll() -> [User] {
        return Array(users.values)
    }
}

// ต้องเพิ่ม Equatable และ Codable ให้ User
extension User: Equatable {
    static func == (lhs: User, rhs: User) -> Bool {
        return lhs.id == rhs.id
    }
}

extension User: Codable {}

let dataSource = UserDataSource()
if let user = dataSource.fetch(id: 1) {
    print("Found: \(user.username)")
}
```

---

## 16.14 where Clauses

```swift
// where ใน extension
extension Array where Element: Numeric {
    var sum: Element {
        return reduce(0, +)
    }
    
    var average: Double {
        guard !isEmpty else { return 0 }
        return Double(sum as! Int) / Double(count)  // simplified
    }
}

extension Array where Element: Equatable {
    func removeDuplicates() -> [Element] {
        var seen: [Element] = []
        return filter { element in
            if seen.contains(element) {
                return false
            }
            seen.append(element)
            return true
        }
    }
    
    func commonElements(with other: [Element]) -> [Element] {
        return filter { other.contains($0) }
    }
}

let numbers = [1, 2, 3, 4, 5]
print("Sum: \(numbers.sum)")

let withDuplicates = [1, 2, 3, 2, 1, 4, 3, 5]
print("No duplicates: \(withDuplicates.removeDuplicates())")

let arr1 = [1, 2, 3, 4, 5]
let arr2 = [3, 4, 5, 6, 7]
print("Common: \(arr1.commonElements(with: arr2))")

// where ใน Generic Function
func printBothIfEqual<T, U>(_ a: T, _ b: U) where T: CustomStringConvertible, U: CustomStringConvertible, T == U, T: Equatable {
    if a == b {
        print("Equal: \(a.description)")
    } else {
        print("Not equal: \(a.description) vs \(b.description)")
    }
}

printBothIfEqual(42, 42)
printBothIfEqual(42, 43)

// where ใน Protocol ด้วย associated types
protocol SortedCollection {
    associatedtype Element
    
    func contains(_ element: Element) -> Bool where Element: Equatable
    func sorted() -> [Element] where Element: Comparable
}
```

---

## 16.15 @objc Protocols

ใช้เมื่อต้องการ interoperability กับ Objective-C หรือเมื่อต้องการ optional requirements

```swift
import Foundation

// @objc protocol ใช้กับ class เท่านั้น
@objc protocol DataDelegate: AnyObject {
    @objc optional func dataDidLoad(_ data: [String])
    @objc optional func dataDidFail(error: Error)
    func dataSource() -> String
}

class DataManager {
    weak var delegate: DataDelegate?
    
    func loadData() {
        // Simulate loading
        let data = ["Item1", "Item2", "Item3"]
        
        // เรียก optional method ถ้ามี
        delegate?.dataDidLoad?(data)
        
        // Required method
        if let source = delegate?.dataSource() {
            print("Data from: \(source)")
        }
    }
}

class ViewController: DataDelegate {
    func dataSource() -> String {
        return "Remote API"
    }
    
    // Optionally implement optional methods
    func dataDidLoad(_ data: [String]) {
        print("โหลดข้อมูลสำเร็จ: \(data)")
    }
    // dataDidFail ไม่ต้อง implement เพราะเป็น optional
}

let manager = DataManager()
let vc = ViewController()
manager.delegate = vc
manager.loadData()
```

---

## 16.16 Protocol-Oriented Programming (POP)

### หลักการ POP

POP เน้นการออกแบบโดยใช้ Protocol เป็นหลัก แทนที่จะใช้ Class hierarchy

```swift
// === ตัวอย่าง: ระบบ Rendering ===

// Protocol สำหรับ rendering
protocol Renderable {
    var position: (x: Double, y: Double) { get set }
    func render(in context: String)
}

protocol Colorable {
    var fillColor: String { get set }
    var strokeColor: String { get set }
}

protocol Resizable {
    mutating func resize(by factor: Double)
    var currentSize: (width: Double, height: Double) { get }
}

protocol Animatable {
    var duration: Double { get set }
    mutating func animate(to state: String)
}

// Composing behaviors ผ่าน Protocol Composition
typealias UIElement = Renderable & Colorable & Resizable

struct Button: UIElement, Animatable {
    var position: (x: Double, y: Double)
    var fillColor: String = "น้ำเงิน"
    var strokeColor: String = "ขาว"
    var width: Double
    var height: Double
    var duration: Double = 0.3
    var title: String
    
    var currentSize: (width: Double, height: Double) {
        return (width, height)
    }
    
    func render(in context: String) {
        print("[\(context)] วาดปุ่ม '\(title)' ที่ (\(position.x),\(position.y)) ขนาด \(width)x\(height) สี\(fillColor)")
    }
    
    mutating func resize(by factor: Double) {
        width *= factor
        height *= factor
    }
    
    mutating func animate(to state: String) {
        print("Animate ปุ่ม '\(title)' ไปที่ state: \(state) ใน \(duration)s")
    }
}

struct ImageView: UIElement {
    var position: (x: Double, y: Double)
    var fillColor: String = "ใส"
    var strokeColor: String = "เทา"
    var width: Double
    var height: Double
    var imageName: String
    
    var currentSize: (width: Double, height: Double) {
        return (width, height)
    }
    
    func render(in context: String) {
        print("[\(context)] วาดรูปภาพ '\(imageName)' ที่ (\(position.x),\(position.y)) ขนาด \(width)x\(height)")
    }
    
    mutating func resize(by factor: Double) {
        width *= factor
        height *= factor
    }
}

// Protocol extension สำหรับ default behavior
extension Renderable where Self: Colorable {
    func renderWithColors(in context: String) {
        render(in: context)
        print("  สีเติม: \(fillColor), สีเส้น: \(strokeColor)")
    }
}

var button = Button(position: (10, 20), width: 100, height: 44, title: "ยืนยัน")
var imageView = ImageView(position: (0, 0), width: 200, height: 200, imageName: "profile.jpg")

let elements: [UIElement] = [button, imageView]
for element in elements {
    element.render(in: "Screen")
}

button.animate(to: "pressed")
button.resize(by: 1.2)
button.renderWithColors(in: "Print")
```

---

## 16.17 POP vs OOP

```swift
// === OOP Approach ===
class OOP_Animal {
    var name: String
    init(name: String) { self.name = name }
    
    func makeSound() -> String { return "..." }
    func move() { print("\(name) เคลื่อนที่") }
}

class OOP_Dog: OOP_Animal {
    override func makeSound() -> String { return "โฮ่ง" }
    func fetch() { print("\(name) เอาลูกบอลมา") }
}

class OOP_Cat: OOP_Animal {
    override func makeSound() -> String { return "เมี้ยว" }
    func purr() { print("\(name) กรู๋") }
}

// ปัญหา: ถ้าต้องการเพิ่ม FlyingAnimal ต้องเพิ่ม method ใน base class
// หรือสร้าง class กลาง ทำให้ hierarchy ซับซ้อน

// === POP Approach ===
protocol POP_Animal {
    var name: String { get }
    func makeSound() -> String
}

protocol POP_Runnable {
    var maxSpeed: Double { get }
    func run()
}

protocol POP_Flyable {
    func fly()
    func land()
}

protocol POP_Swimmable {
    func swim()
}

// Concrete types ใช้ Protocol composition
struct POP_Dog: POP_Animal, POP_Runnable {
    var name: String
    var maxSpeed: Double = 45
    
    func makeSound() -> String { return "โฮ่ง" }
    func run() { print("\(name) วิ่งด้วยความเร็ว \(maxSpeed) km/h") }
    func fetch() { print("\(name) เอาลูกบอลมา") }
}

struct POP_Duck: POP_Animal, POP_Runnable, POP_Flyable, POP_Swimmable {
    var name: String
    var maxSpeed: Double = 20
    
    func makeSound() -> String { return "กา กา" }
    func run() { print("\(name) วิ่งอุ้ยอ้าย") }
    func fly() { print("\(name) บิน") }
    func land() { print("\(name) ลงจอด") }
    func swim() { print("\(name) ว่ายน้ำ") }
}

struct POP_Fish: POP_Animal, POP_Swimmable {
    var name: String
    
    func makeSound() -> String { return "..." }
    func swim() { print("\(name) ว่ายน้ำอย่างอิสระ") }
}

// ใช้งาน Protocol เป็น type
let animals: [POP_Animal] = [
    POP_Dog(name: "หมา"), 
    POP_Duck(name: "เป็ด"), 
    POP_Fish(name: "ปลา")
]
animals.forEach { print("\($0.name): \($0.makeSound())") }

// Protocol composition
let swimmers: [POP_Swimmable] = [POP_Duck(name: "เป็ด"), POP_Fish(name: "ปลา")]
swimmers.forEach { $0.swim() }
```

---

## 16.18 Comparable, Equatable, Hashable

### Equatable

```swift
struct Point2D: Equatable {
    var x: Double
    var y: Double
    
    // Swift auto-synthesizes == เมื่อทุก stored property เป็น Equatable
    // แต่เราสามารถ customize ได้:
    static func == (lhs: Point2D, rhs: Point2D) -> Bool {
        return lhs.x == rhs.x && lhs.y == rhs.y
    }
}

let p1 = Point2D(x: 1, y: 2)
let p2 = Point2D(x: 1, y: 2)
let p3 = Point2D(x: 3, y: 4)

print("p1 == p2: \(p1 == p2)")  // true
print("p1 == p3: \(p1 == p3)")  // false
print("p1 != p3: \(p1 != p3)")  // true (auto-generated)
```

### Comparable

```swift
struct Student: Comparable {
    var name: String
    var gpa: Double
    var yearOfStudy: Int
    
    // ต้อง implement < (และ Equatable)
    static func < (lhs: Student, rhs: Student) -> Bool {
        if lhs.gpa != rhs.gpa {
            return lhs.gpa < rhs.gpa
        }
        return lhs.name < rhs.name
    }
    
    static func == (lhs: Student, rhs: Student) -> Bool {
        return lhs.name == rhs.name && lhs.gpa == rhs.gpa
    }
}

var students = [
    Student(name: "สมชาย", gpa: 3.5, yearOfStudy: 2),
    Student(name: "สมหญิง", gpa: 3.8, yearOfStudy: 1),
    Student(name: "ประสิทธิ์", gpa: 3.5, yearOfStudy: 3),
    Student(name: "วิชัย", gpa: 4.0, yearOfStudy: 4)
]

students.sort()  // ใช้ < ที่ implement ไว้
students.forEach { print("\($0.name): \($0.gpa)") }

let best = students.max()
print("นักเรียนดีเด่น: \(best?.name ?? "ไม่มี")")
```

### Hashable

```swift
struct ProductID: Hashable {
    var category: String
    var sku: String
    var version: Int
    
    // Swift auto-synthesizes hash(into:) ถ้าทุก stored property เป็น Hashable
    // แต่เราสามารถ customize ได้:
    func hash(into hasher: inout Hasher) {
        hasher.combine(category)
        hasher.combine(sku)
        // ไม่รวม version ในการ hash (ถือว่า product เดิม ต่าง version)
    }
    
    static func == (lhs: ProductID, rhs: ProductID) -> Bool {
        return lhs.category == rhs.category && lhs.sku == rhs.sku
    }
}

// ใช้ใน Set
var productSet: Set<ProductID> = []
productSet.insert(ProductID(category: "Electronics", sku: "PHONE001", version: 1))
productSet.insert(ProductID(category: "Electronics", sku: "PHONE001", version: 2))  // ซ้ำ (hash เหมือนกัน)
productSet.insert(ProductID(category: "Clothing", sku: "SHIRT001", version: 1))
print("Unique products: \(productSet.count)")  // 2

// ใช้เป็น Dictionary key
var inventory: [ProductID: Int] = [:]
let phone = ProductID(category: "Electronics", sku: "PHONE001", version: 1)
inventory[phone] = 100
print("Stock: \(inventory[phone] ?? 0)")
```

---

## 16.19 Codable (Encodable + Decodable)

```swift
import Foundation

// Codable = Encodable & Decodable
struct Product: Codable {
    let id: Int
    var name: String
    var price: Double
    var category: String
    var tags: [String]
    var isAvailable: Bool
    
    // Nested struct ที่เป็น Codable ด้วย
    struct Dimensions: Codable {
        var width: Double
        var height: Double
        var depth: Double
    }
    
    var dimensions: Dimensions?
}

// ตัวอย่าง JSON encoding/decoding
let product = Product(
    id: 1,
    name: "MacBook Pro",
    price: 49900.0,
    category: "คอมพิวเตอร์",
    tags: ["laptop", "apple", "premium"],
    isAvailable: true,
    dimensions: Product.Dimensions(width: 30.41, height: 0.61, depth: 21.24)
)

// Encode เป็น JSON
let encoder = JSONEncoder()
encoder.outputFormatting = .prettyPrinted

if let jsonData = try? encoder.encode(product),
   let jsonString = String(data: jsonData, encoding: .utf8) {
    print("JSON:\n\(jsonString)")
}

// Decode จาก JSON
let jsonString = """
{
    "id": 2,
    "name": "iPhone 15",
    "price": 29900.0,
    "category": "มือถือ",
    "tags": ["phone", "apple"],
    "isAvailable": true
}
"""

let decoder = JSONDecoder()
if let jsonData = jsonString.data(using: .utf8),
   let decodedProduct = try? decoder.decode(Product.self, from: jsonData) {
    print("Decoded: \(decodedProduct.name) ราคา \(decodedProduct.price) บาท")
}

// Custom Coding Keys
struct APIUser: Codable {
    var id: Int
    var username: String
    var firstName: String
    var lastName: String
    var emailAddress: String
    
    // Map Swift property names ไปยัง JSON keys
    enum CodingKeys: String, CodingKey {
        case id
        case username
        case firstName = "first_name"   // snake_case ใน JSON
        case lastName = "last_name"
        case emailAddress = "email"
    }
}

let apiJSON = """
{
    "id": 101,
    "username": "somchai_01",
    "first_name": "สมชาย",
    "last_name": "ใจดี",
    "email": "somchai@example.com"
}
"""

if let data = apiJSON.data(using: .utf8),
   let user = try? decoder.decode(APIUser.self, from: data) {
    print("User: \(user.firstName) \(user.lastName)")
}

// Custom Encoding/Decoding
struct FlexibleDate: Codable {
    var date: Date
    
    init(date: Date) {
        self.date = date
    }
    
    init(from decoder: Decoder) throws {
        let container = try decoder.singleValueContainer()
        if let timestamp = try? container.decode(Double.self) {
            date = Date(timeIntervalSince1970: timestamp)
        } else if let dateString = try? container.decode(String.self) {
            let formatter = ISO8601DateFormatter()
            guard let parsedDate = formatter.date(from: dateString) else {
                throw DecodingError.dataCorruptedError(in: container,
                    debugDescription: "ไม่สามารถ parse date ได้")
            }
            date = parsedDate
        } else {
            throw DecodingError.dataCorruptedError(in: container,
                debugDescription: "รูปแบบ date ไม่ถูกต้อง")
        }
    }
    
    func encode(to encoder: Encoder) throws {
        var container = encoder.singleValueContainer()
        try container.encode(date.timeIntervalSince1970)
    }
}
```

---

## 16.20 CustomStringConvertible

```swift
// Protocol สำหรับ custom string representation
protocol CustomStringConvertible {
    var description: String { get }
}

struct Matrix: CustomStringConvertible {
    private var data: [[Double]]
    let rows: Int
    let cols: Int
    
    init(rows: Int, cols: Int, fill: Double = 0) {
        self.rows = rows
        self.cols = cols
        self.data = Array(repeating: Array(repeating: fill, count: cols), count: rows)
    }
    
    subscript(row: Int, col: Int) -> Double {
        get { return data[row][col] }
        set { data[row][col] = newValue }
    }
    
    var description: String {
        var result = "Matrix \(rows)x\(cols):\n"
        for row in data {
            let rowStr = row.map { String(format: "%6.1f", $0) }.joined(separator: " ")
            result += "  [\(rowStr)]\n"
        }
        return result
    }
}

extension Matrix: CustomDebugStringConvertible {
    var debugDescription: String {
        return "Matrix<\(rows)x\(cols)>: \(data)"
    }
}

var m = Matrix(rows: 3, cols: 3)
m[0, 0] = 1; m[0, 1] = 2; m[0, 2] = 3
m[1, 0] = 4; m[1, 1] = 5; m[1, 2] = 6
m[2, 0] = 7; m[2, 1] = 8; m[2, 2] = 9

print(m)  // ใช้ description
```

---

## 16.21 Sequence Protocol

```swift
// Custom type ที่ conform Sequence
struct CountdownSequence: Sequence {
    let start: Int
    let end: Int
    let step: Int
    
    // ต้อง implement makeIterator()
    func makeIterator() -> CountdownIterator {
        return CountdownIterator(current: start, end: end, step: step)
    }
}

// Iterator สำหรับ CountdownSequence
struct CountdownIterator: IteratorProtocol {
    var current: Int
    let end: Int
    let step: Int
    
    // ต้อง implement next()
    mutating func next() -> Int? {
        guard current >= end else { return nil }
        let value = current
        current -= step
        return value
    }
}

// การใช้งาน
let countdown = CountdownSequence(start: 10, end: 1, step: 1)
for number in countdown {
    print(number, terminator: " ")
}
print()

// ได้ method จาก Sequence protocol ฟรี!
print("Sum: \(countdown.reduce(0, +))")
print("Max: \(countdown.max() ?? 0)")
print("Filter even: \(countdown.filter { $0 % 2 == 0 })")
print("Map doubled: \(countdown.map { $0 * 2 })")

// ตัวอย่างที่ซับซ้อนกว่า: Fibonacci Sequence
struct FibonacciSequence: Sequence {
    let count: Int
    
    func makeIterator() -> FibonacciIterator {
        return FibonacciIterator(remaining: count)
    }
}

struct FibonacciIterator: IteratorProtocol {
    var remaining: Int
    var current: Int = 0
    var next_val: Int = 1
    
    mutating func next() -> Int? {
        guard remaining > 0 else { return nil }
        remaining -= 1
        let value = current
        let temp = current + next_val
        current = next_val
        next_val = temp
        return value
    }
}

let fibs = FibonacciSequence(count: 10)
print("Fibonacci: \(Array(fibs))")
// [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```

---

## 16.22 Collection Protocol

```swift
// Custom Collection ต้อง conform Collection Protocol
struct FixedQueue<T>: Collection {
    private var items: [T]
    private let capacity: Int
    
    init(capacity: Int) {
        self.capacity = capacity
        self.items = []
        self.items.reserveCapacity(capacity)
    }
    
    // Collection requirements
    typealias Index = Int
    
    var startIndex: Int { return 0 }
    var endIndex: Int { return items.count }
    
    subscript(index: Int) -> T {
        precondition(index >= 0 && index < items.count)
        return items[index]
    }
    
    func index(after i: Int) -> Int {
        return i + 1
    }
    
    // Custom methods
    mutating func enqueue(_ item: T) -> Bool {
        guard items.count < capacity else {
            print("Queue เต็ม (capacity: \(capacity))")
            return false
        }
        items.append(item)
        return true
    }
    
    mutating func dequeue() -> T? {
        guard !items.isEmpty else { return nil }
        return items.removeFirst()
    }
    
    var isFull: Bool { return items.count == capacity }
}

var queue = FixedQueue<String>(capacity: 3)
queue.enqueue("Task 1")
queue.enqueue("Task 2")
queue.enqueue("Task 3")
queue.enqueue("Task 4")  // เต็ม

// Collection protocol ให้ method เหล่านี้ฟรี
print("Count: \(queue.count)")
print("isEmpty: \(queue.isEmpty)")
print("First: \(queue.first ?? "")")
print("Contains Task 2: \(queue.contains("Task 2"))")

for item in queue {
    print("  - \(item)")
}

// ได้ higher-order functions ฟรีด้วย
let filtered = queue.filter { $0.contains("1") }
print("Filtered: \(filtered)")
```

---

## 16.23 ตัวอย่างจริง: Drawable Protocol System

```swift
// MARK: - Comprehensive Drawable System

import Foundation

// Core Protocols
protocol Renderable {
    func render() -> String
}

protocol Transformable {
    var transform: CGAffineTransform { get set }
    mutating func translate(by offset: CGPoint)
    mutating func scale(by factor: CGFloat)
    mutating func rotate(by angle: CGFloat)
}

protocol Styleable {
    var fillColor: String { get set }
    var strokeColor: String { get set }
    var lineWidth: CGFloat { get set }
    var opacity: Double { get set }
}

// Simple CGPoint/CGAffineTransform substitutes for this example
struct CGPoint { var x: Double; var y: Double }
struct CGAffineTransform { var tx: Double; var ty: Double; var scaleX: Double; var scaleY: Double; var rotation: Double }
typealias CGFloat = Double

// Protocol Composition
typealias DrawableElement = Renderable & Transformable & Styleable

// Base implementation via Protocol Extension
extension Transformable {
    mutating func translate(by offset: CGPoint) {
        transform.tx += offset.x
        transform.ty += offset.y
    }
    
    mutating func scale(by factor: CGFloat) {
        transform.scaleX *= factor
        transform.scaleY *= factor
    }
    
    mutating func rotate(by angle: CGFloat) {
        transform.rotation += angle
    }
}

// Concrete Types
struct DrawableCircle: DrawableElement {
    var center: CGPoint
    var radius: Double
    var fillColor: String = "น้ำเงิน"
    var strokeColor: String = "ดำ"
    var lineWidth: CGFloat = 1.0
    var opacity: Double = 1.0
    var transform = CGAffineTransform(tx: 0, ty: 0, scaleX: 1, scaleY: 1, rotation: 0)
    
    func render() -> String {
        let effectiveRadius = radius * transform.scaleX
        return "<circle cx='\(center.x + transform.tx)' cy='\(center.y + transform.ty)' r='\(effectiveRadius)' fill='\(fillColor)' stroke='\(strokeColor)' opacity='\(opacity)'/>"
    }
}

struct DrawableRect: DrawableElement {
    var origin: CGPoint
    var size: (width: Double, height: Double)
    var fillColor: String = "แดง"
    var strokeColor: String = "ดำ"
    var lineWidth: CGFloat = 1.0
    var opacity: Double = 1.0
    var transform = CGAffineTransform(tx: 0, ty: 0, scaleX: 1, scaleY: 1, rotation: 0)
    
    func render() -> String {
        let w = size.width * transform.scaleX
        let h = size.height * transform.scaleY
        return "<rect x='\(origin.x + transform.tx)' y='\(origin.y + transform.ty)' width='\(w)' height='\(h)' fill='\(fillColor)' opacity='\(opacity)'/>"
    }
}

struct DrawableText: DrawableElement {
    var position: CGPoint
    var text: String
    var fontSize: Double
    var fillColor: String = "ดำ"
    var strokeColor: String = "ดำ"
    var lineWidth: CGFloat = 0.5
    var opacity: Double = 1.0
    var transform = CGAffineTransform(tx: 0, ty: 0, scaleX: 1, scaleY: 1, rotation: 0)
    
    func render() -> String {
        let effectiveFontSize = fontSize * transform.scaleX
        return "<text x='\(position.x + transform.tx)' y='\(position.y + transform.ty)' font-size='\(effectiveFontSize)' fill='\(fillColor)'>\(text)</text>"
    }
}

// Drawing Canvas
class DrawingCanvas {
    var elements: [any DrawableElement] = []
    var width: Double
    var height: Double
    var title: String
    
    init(width: Double, height: Double, title: String) {
        self.width = width
        self.height = height
        self.title = title
    }
    
    func addElement(_ element: any DrawableElement) {
        elements.append(element)
    }
    
    func renderSVG() -> String {
        var svg = """
        <svg width='\(width)' height='\(height)' xmlns='http://www.w3.org/2000/svg'>
          <title>\(title)</title>
        """
        for element in elements {
            svg += "\n  " + element.render()
        }
        svg += "\n</svg>"
        return svg
    }
    
    func elementCount() -> Int { return elements.count }
    
    func applyGlobalTransform(_ transform: CGAffineTransform) {
        elements = elements.map { element in
            var mutable = element
            mutable.transform.tx += transform.tx
            mutable.transform.ty += transform.ty
            return mutable
        }
    }
}

// การใช้งาน
let canvas = DrawingCanvas(width: 800, height: 600, title: "My Drawing")

var circle = DrawableCircle(center: CGPoint(x: 100, y: 100), radius: 50)
circle.fillColor = "ฟ้า"
circle.opacity = 0.8
circle.translate(by: CGPoint(x: 20, y: 10))

var rect = DrawableRect(origin: CGPoint(x: 200, y: 150), size: (width: 100, height: 60))
rect.scale(by: 1.5)

var text = DrawableText(position: CGPoint(x: 50, y: 50), text: "สวัสดี Swift!", fontSize: 20)

canvas.addElement(circle)
canvas.addElement(rect)
canvas.addElement(text)

print("SVG Output:")
print(canvas.renderSVG())
print("\nจำนวน elements: \(canvas.elementCount())")
```

---

## 16.24 ตัวอย่างจริง: Flyable และ Serializable

```swift
// MARK: - Flyable System

enum FlightStatus {
    case grounded
    case takingOff
    case cruising
    case landing
}

protocol Flyable2 {
    var maxAltitude: Double { get }
    var maxSpeed: Double { get }
    var flightStatus: FlightStatus { get }
    
    mutating func takeOff() throws
    mutating func cruiseAt(altitude: Double, speed: Double) throws
    mutating func land() throws
}

enum FlightError: Error {
    case alreadyFlying
    case alreadyGrounded
    case altitudeTooHigh(max: Double, requested: Double)
    case speedTooFast(max: Double, requested: Double)
}

extension Flyable2 {
    var isFlying: Bool {
        return flightStatus != .grounded
    }
    
    func flightStatusDescription() -> String {
        switch flightStatus {
        case .grounded: return "จอดอยู่"
        case .takingOff: return "กำลังขึ้นบิน"
        case .cruising: return "บินระดับ"
        case .landing: return "กำลังลงจอด"
        }
    }
}

class AircraftImpl: Flyable2 {
    var name: String
    var maxAltitude: Double
    var maxSpeed: Double
    var flightStatus: FlightStatus = .grounded
    var currentAltitude: Double = 0
    var currentSpeed: Double = 0
    
    init(name: String, maxAltitude: Double, maxSpeed: Double) {
        self.name = name
        self.maxAltitude = maxAltitude
        self.maxSpeed = maxSpeed
    }
    
    func takeOff() throws {
        guard flightStatus == .grounded else {
            throw FlightError.alreadyFlying
        }
        flightStatus = .takingOff
        currentAltitude = 300
        currentSpeed = 250
        print("\(name) ขึ้นบินที่ความสูง \(currentAltitude)m ความเร็ว \(currentSpeed)km/h")
        flightStatus = .cruising
    }
    
    func cruiseAt(altitude: Double, speed: Double) throws {
        guard altitude <= maxAltitude else {
            throw FlightError.altitudeTooHigh(max: maxAltitude, requested: altitude)
        }
        guard speed <= maxSpeed else {
            throw FlightError.speedTooFast(max: maxSpeed, requested: speed)
        }
        currentAltitude = altitude
        currentSpeed = speed
        print("\(name) บินระดับที่ \(altitude)m ความเร็ว \(speed)km/h")
    }
    
    func land() throws {
        guard flightStatus != .grounded else {
            throw FlightError.alreadyGrounded
        }
        flightStatus = .landing
        currentAltitude = 0
        currentSpeed = 0
        flightStatus = .grounded
        print("\(name) ลงจอดแล้ว")
    }
}

// MARK: - Serializable Protocol

protocol Serializable {
    func serialize() -> Data?
    static func deserialize(from data: Data) -> Self?
    func serializeToString() -> String
}

extension Serializable where Self: Codable {
    func serialize() -> Data? {
        return try? JSONEncoder().encode(self)
    }
    
    static func deserialize(from data: Data) -> Self? {
        return try? JSONDecoder().decode(Self.self, from: data)
    }
    
    func serializeToString() -> String {
        guard let data = serialize() else { return "{}" }
        return String(data: data, encoding: .utf8) ?? "{}"
    }
}

struct FlightRecord: Codable, Serializable {
    var flightNumber: String
    var origin: String
    var destination: String
    var departureTime: TimeInterval
    var arrivalTime: TimeInterval
    var aircraftType: String
    var passengerCount: Int
}

let record = FlightRecord(
    flightNumber: "TG101",
    origin: "BKK",
    destination: "NRT",
    departureTime: Date().timeIntervalSince1970,
    arrivalTime: Date().timeIntervalSince1970 + 6 * 3600,
    aircraftType: "B777",
    passengerCount: 250
)

print("Serialized: \(record.serializeToString())")

if let data = record.serialize(),
   let restored = FlightRecord.deserialize(from: data) {
    print("Restored: \(restored.flightNumber) \(restored.origin) → \(restored.destination)")
}

// การใช้งาน Flyable
let aircraft = AircraftImpl(name: "TG101", maxAltitude: 12000, maxSpeed: 900)
do {
    try aircraft.takeOff()
    try aircraft.cruiseAt(altitude: 10000, speed: 850)
    print("Status: \(aircraft.flightStatusDescription())")
    try aircraft.land()
} catch FlightError.altitudeTooHigh(let max, let requested) {
    print("Error: ความสูง \(requested)m เกิน \(max)m")
} catch FlightError.speedTooFast(let max, let requested) {
    print("Error: ความเร็ว \(requested) เกิน \(max)")
} catch {
    print("Error: \(error)")
}
```

---

## 16.25 แบบฝึกหัด

### แบบฝึกหัดที่ 1: Payment System

```swift
// MARK: - Exercise 1: Payment System

protocol PaymentMethod {
    var methodName: String { get }
    var isAvailable: Bool { get }
    var processingFeePercent: Double { get }
    
    func processPayment(amount: Double) throws -> TransactionResult
    func validatePaymentDetails() -> [String]
}

struct TransactionResult {
    var transactionID: String
    var amount: Double
    var fee: Double
    var status: String
    var timestamp: Date
    
    var totalCharged: Double { return amount + fee }
}

enum PaymentError: Error {
    case insufficientFunds(balance: Double, required: Double)
    case invalidCard(reason: String)
    case networkError
    case methodUnavailable
}

extension PaymentMethod {
    func calculateFee(for amount: Double) -> Double {
        return amount * processingFeePercent / 100
    }
    
    func validate() -> Bool {
        return validatePaymentDetails().isEmpty
    }
}

struct CreditCard: PaymentMethod {
    var methodName: String = "บัตรเครดิต"
    var cardNumber: String
    var expiryMonth: Int
    var expiryYear: Int
    var cvv: String
    var creditLimit: Double
    var currentBalance: Double
    
    var isAvailable: Bool {
        let currentYear = 2024
        let currentMonth = 3
        return (expiryYear > currentYear) || 
               (expiryYear == currentYear && expiryMonth >= currentMonth)
    }
    
    var processingFeePercent: Double { return 1.5 }
    
    func processPayment(amount: Double) throws -> TransactionResult {
        guard isAvailable else {
            throw PaymentError.invalidCard(reason: "บัตรหมดอายุ")
        }
        guard currentBalance + amount <= creditLimit else {
            throw PaymentError.insufficientFunds(
                balance: creditLimit - currentBalance,
                required: amount
            )
        }
        let fee = calculateFee(for: amount)
        return TransactionResult(
            transactionID: "CC-\(Int.random(in: 100000...999999))",
            amount: amount,
            fee: fee,
            status: "สำเร็จ",
            timestamp: Date()
        )
    }
    
    func validatePaymentDetails() -> [String] {
        var errors: [String] = []
        if cardNumber.count != 16 { errors.append("หมายเลขบัตรต้องมี 16 หลัก") }
        if cvv.count != 3 { errors.append("CVV ต้องมี 3 หลัก") }
        if !isAvailable { errors.append("บัตรหมดอายุ") }
        return errors
    }
}

struct BankTransfer: PaymentMethod {
    var methodName: String = "โอนเงินธนาคาร"
    var accountNumber: String
    var bankCode: String
    var accountName: String
    var balance: Double
    
    var isAvailable: Bool { return true }
    var processingFeePercent: Double { return 0.5 }
    
    func processPayment(amount: Double) throws -> TransactionResult {
        guard balance >= amount else {
            throw PaymentError.insufficientFunds(balance: balance, required: amount)
        }
        let fee = calculateFee(for: amount)
        return TransactionResult(
            transactionID: "BT-\(Int.random(in: 100000...999999))",
            amount: amount,
            fee: fee,
            status: "สำเร็จ",
            timestamp: Date()
        )
    }
    
    func validatePaymentDetails() -> [String] {
        var errors: [String] = []
        if accountNumber.count < 10 { errors.append("หมายเลขบัญชีไม่ถูกต้อง") }
        if bankCode.isEmpty { errors.append("รหัสธนาคารไม่ถูกต้อง") }
        return errors
    }
}

struct DigitalWallet: PaymentMethod {
    var methodName: String
    var walletID: String
    var balance: Double
    var isAvailable: Bool = true
    var processingFeePercent: Double = 0.0
    
    func processPayment(amount: Double) throws -> TransactionResult {
        guard balance >= amount else {
            throw PaymentError.insufficientFunds(balance: balance, required: amount)
        }
        return TransactionResult(
            transactionID: "DW-\(Int.random(in: 100000...999999))",
            amount: amount,
            fee: 0,
            status: "สำเร็จ",
            timestamp: Date()
        )
    }
    
    func validatePaymentDetails() -> [String] {
        return walletID.isEmpty ? ["Wallet ID ไม่ถูกต้อง"] : []
    }
}

// Payment Processor ใช้ Protocol
class PaymentProcessor {
    var availableMethods: [PaymentMethod]
    
    init(methods: [PaymentMethod]) {
        self.availableMethods = methods
    }
    
    func processPayment(amount: Double, using method: PaymentMethod) {
        print("\nประมวลผลการชำระเงิน \(String(format: "%.2f", amount)) บาท ด้วย\(method.methodName)")
        
        guard method.validate() else {
            print("Validation ล้มเหลว: \(method.validatePaymentDetails())")
            return
        }
        
        do {
            let result = try method.processPayment(amount: amount)
            print("✅ Transaction ID: \(result.transactionID)")
            print("   ยอด: \(result.amount) บาท + ค่าธรรมเนียม: \(result.fee) บาท")
            print("   รวม: \(result.totalCharged) บาท")
        } catch PaymentError.insufficientFunds(let balance, let required) {
            print("❌ เงินไม่พอ (มี \(balance) บาท ต้องการ \(required) บาท)")
        } catch PaymentError.invalidCard(let reason) {
            print("❌ บัตรไม่ถูกต้อง: \(reason)")
        } catch {
            print("❌ เกิดข้อผิดพลาด: \(error)")
        }
    }
    
    func findCheapestMethod(for amount: Double) -> PaymentMethod? {
        return availableMethods
            .filter { $0.isAvailable && $0.validate() }
            .min { $0.calculateFee(for: amount) < $1.calculateFee(for: amount) }
    }
}

// การใช้งาน
let creditCard = CreditCard(
    cardNumber: "1234567890123456",
    expiryMonth: 12, expiryYear: 2026,
    cvv: "123",
    creditLimit: 50000,
    currentBalance: 10000
)

let bankTransfer = BankTransfer(
    accountNumber: "1234567890",
    bankCode: "SCB",
    accountName: "สมชาย ใจดี",
    balance: 100000
)

let promptPay = DigitalWallet(
    methodName: "พร้อมเพย์",
    walletID: "0812345678",
    balance: 5000
)

let processor = PaymentProcessor(methods: [creditCard, bankTransfer, promptPay])
processor.processPayment(amount: 1500, using: creditCard)
processor.processPayment(amount: 99999, using: creditCard)  // เกิน credit limit
processor.processPayment(amount: 3000, using: promptPay)   // เกิน balance

if let cheapest = processor.findCheapestMethod(for: 1000) {
    print("\nวิธีชำระเงินที่ถูกที่สุด: \(cheapest.methodName) (ค่าธรรมเนียม \(cheapest.calculateFee(for: 1000)) บาท)")
}
```

### แบบฝึกหัดที่ 2: Plugin System

```swift
// MARK: - Exercise 2: Plugin System

protocol Plugin {
    var name: String { get }
    var version: String { get }
    var author: String { get }
    var dependencies: [String] { get }
    
    func initialize() throws
    func execute(input: [String: Any]) throws -> [String: Any]
    func cleanup()
}

extension Plugin {
    var identifier: String {
        return "\(name)@\(version)"
    }
    
    func cleanup() {
        print("[\(name)] cleanup เสร็จสิ้น")
    }
}

enum PluginError: Error {
    case initializationFailed(reason: String)
    case executionFailed(reason: String)
    case missingDependency(name: String)
}

struct LoggerPlugin: Plugin {
    var name: String = "Logger"
    var version: String = "1.0.0"
    var author: String = "Dev Team"
    var dependencies: [String] = []
    
    var logLevel: String = "INFO"
    
    func initialize() throws {
        print("[\(name)] เริ่มต้น Logger plugin")
    }
    
    func execute(input: [String: Any]) throws -> [String: Any] {
        guard let message = input["message"] as? String else {
            throw PluginError.executionFailed(reason: "ต้องมี 'message' ใน input")
        }
        print("[\(logLevel)] \(message)")
        return ["logged": true, "timestamp": Date().timeIntervalSince1970]
    }
}

struct TransformPlugin: Plugin {
    var name: String = "Transform"
    var version: String = "2.1.0"
    var author: String = "Plugin Author"
    var dependencies: [String] = ["Logger"]
    
    enum TransformType: String {
        case uppercase, lowercase, reverse, titleCase
    }
    
    var transformType: TransformType
    
    func initialize() throws {
        print("[\(name)] เตรียมพร้อม Transform plugin (\(transformType.rawValue))")
    }
    
    func execute(input: [String: Any]) throws -> [String: Any] {
        guard let text = input["text"] as? String else {
            throw PluginError.executionFailed(reason: "ต้องมี 'text' ใน input")
        }
        
        let result: String
        switch transformType {
        case .uppercase: result = text.uppercased()
        case .lowercase: result = text.lowercased()
        case .reverse: result = String(text.reversed())
        case .titleCase:
            result = text.split(separator: " ")
                .map { $0.prefix(1).uppercased() + $0.dropFirst().lowercased() }
                .joined(separator: " ")
        }
        return ["result": result, "original": text]
    }
}

class PluginManager {
    private var plugins: [String: Plugin] = [:]
    private var initialized: Set<String> = []
    
    func register(_ plugin: Plugin) throws {
        // ตรวจสอบ dependencies
        for dep in plugin.dependencies {
            guard plugins[dep] != nil else {
                throw PluginError.missingDependency(name: dep)
            }
        }
        plugins[plugin.name] = plugin
        print("ลงทะเบียน plugin: \(plugin.identifier)")
    }
    
    func initializePlugin(named name: String) throws {
        guard let plugin = plugins[name] else {
            throw PluginError.initializationFailed(reason: "ไม่พบ plugin: \(name)")
        }
        try plugin.initialize()
        initialized.insert(name)
    }
    
    func execute(plugin name: String, input: [String: Any]) throws -> [String: Any] {
        guard let plugin = plugins[name] else {
            throw PluginError.executionFailed(reason: "ไม่พบ plugin: \(name)")
        }
        guard initialized.contains(name) else {
            throw PluginError.executionFailed(reason: "Plugin ยังไม่ได้ initialize")
        }
        return try plugin.execute(input: input)
    }
    
    func listPlugins() {
        print("\n=== Plugin Registry ===")
        for (_, plugin) in plugins {
            print("  \(plugin.identifier) by \(plugin.author)")
            if !plugin.dependencies.isEmpty {
                print("    Dependencies: \(plugin.dependencies.joined(separator: ", "))")
            }
        }
    }
}

// การใช้งาน
let manager = PluginManager()

do {
    try manager.register(LoggerPlugin())
    try manager.register(TransformPlugin(transformType: .titleCase))
    
    manager.listPlugins()
    
    try manager.initializePlugin(named: "Logger")
    try manager.initializePlugin(named: "Transform")
    
    let logResult = try manager.execute(plugin: "Logger", input: ["message": "Hello from Plugin!"])
    print("Log result: \(logResult)")
    
    let transformResult = try manager.execute(plugin: "Transform", input: ["text": "hello world from swift"])
    print("Transform result: \(transformResult["result"] ?? "")")
    
} catch PluginError.missingDependency(let name) {
    print("Missing dependency: \(name)")
} catch PluginError.executionFailed(let reason) {
    print("Execution failed: \(reason)")
} catch {
    print("Error: \(error)")
}
```

---

## 16.26 สรุป

ในบทนี้เราได้เรียนรู้ทุกด้านของ Protocol ใน Swift:

### สิ่งที่ได้เรียนรู้

1. **Protocol คืออะไร** — Blueprint ที่กำหนด requirements สำหรับ Type
2. **Protocol Syntax** — รูปแบบการเขียน Protocol และ requirements
3. **Property Requirements** — `{ get }` และ `{ get set }`
4. **Method Requirements** — Instance, static, mutating methods
5. **Initializer Requirements** — `init` ใน Protocol
6. **Protocol Adoption** — วิธี adopt Protocol ใน Class, Struct, Enum
7. **Protocol Conformance** — Explicit และ Retroactive conformance
8. **Protocol as Type** — ใช้ Protocol เป็น type ของ variable/parameter/return
9. **Protocol Composition** — รวมหลาย Protocol ด้วย `&`
10. **Protocol Extensions** — เพิ่ม method ให้ทุก Type ที่ adopt Protocol
11. **Default Implementations** — Implementation ที่ override ได้
12. **Protocol Inheritance** — Protocol สืบทอดจาก Protocol อื่น
13. **Conditional Conformance** — Conform Protocol ตามเงื่อนไข
14. **Associated Types** — Placeholder type ใน Protocol
15. **Type Constraints** — จำกัด Generic type ด้วย Protocol
16. **where Clauses** — เพิ่มเงื่อนไขใน Generic
17. **@objc Protocols** — Optional requirements และ ObjC interop
18. **POP** — Protocol-Oriented Programming แนวคิดหลักของ Swift
19. **Standard Protocols** — Equatable, Comparable, Hashable, Codable
20. **Sequence & Collection** — สร้าง Custom sequence ที่ใช้ for-in ได้

### Key Takeaways

```swift
// 1. Protocol เป็น Blueprint ไม่ใช่ Implementation
protocol CanFly { func fly() }

// 2. Protocol เป็น Type แบบ First-class
let flyables: [CanFly] = [...]

// 3. Protocol Composition เพิ่มความยืดหยุ่น
func operate(_ obj: CanFly & Describable) { ... }

// 4. Extension ให้ Default Implementation
extension CanFly { func maxAltitude() -> Double { return 1000 } }

// 5. Associated Types สำหรับ Generic Protocols
protocol Container { associatedtype Element; func add(_ item: Element) }
```

### Protocol-Oriented Programming Principles

| หลักการ | คำอธิบาย |
|---------|----------|
| **Favor Composition** | ใช้ Protocol หลายตัวแทน Deep inheritance |
| **Program to Protocol** | ใช้ Protocol type แทน Concrete type |
| **Default Implementation** | แบ่งปัน logic ผ่าน Extension |
| **Value Semantics** | ใช้ Struct + Protocol แทน Class |
| **Testability** | Protocol ทำให้ Mock ง่าย |

### เปรียบเทียบ Protocol vs Class Inheritance

| | Protocol | Class Inheritance |
|---|---------|-----------------|
| Value Types | ✅ Struct, Enum | ❌ Class เท่านั้น |
| Multiple Adoption | ✅ หลาย Protocol | ❌ Single Inheritance |
| Implementation Sharing | ✅ via Extension | ✅ ใน Superclass |
| Runtime Cost | ✅ ต่ำกว่า | ตามปกติ |
| Retroactive | ✅ เพิ่มทีหลังได้ | ❌ ต้องแก้โค้ดเดิม |
| Optional Requirements | ✅ (`@objc`) | ❌ |

### บทต่อไป
ใน Part 17 เราจะเรียนรู้เกี่ยวกับ **Generics** ซึ่งช่วยให้เขียนโค้ดที่ยืดหยุ่นและนำกลับมาใช้ซ้ำได้สูงสุด โดยทำงานได้กับ Type ต่างๆ โดยไม่ต้องเขียนโค้ดซ้ำ

---

*หมายเหตุ: โค้ดทั้งหมดในบทนี้สามารถรันได้ใน Swift Playgrounds หรือ Xcode*
