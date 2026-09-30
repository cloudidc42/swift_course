# Part 15: Inheritance และ Polymorphism ใน Swift

## บทนำ

Inheritance (การสืบทอด) และ Polymorphism (พหุสัณฐาน) เป็นหลักการสำคัญของ Object-Oriented Programming (OOP) ที่ช่วยให้เราสร้างโค้ดที่มีโครงสร้างดี นำกลับมาใช้ใหม่ได้ และง่ายต่อการบำรุงรักษา ใน Swift ระบบ Inheritance มีความปลอดภัยสูงและมีเครื่องมือช่วยป้องกันข้อผิดพลาดต่างๆ

ในบทนี้เราจะเรียนรู้:
- พื้นฐานการสืบทอดคลาส
- การ Override method และ property
- Polymorphism และ Dynamic dispatch
- การตรวจสอบประเภทและการแปลงประเภท
- รูปแบบการออกแบบ Class hierarchy
- แบบฝึกหัดและตัวอย่างจริง

---

## 15.1 พื้นฐานการสืบทอด (Inheritance Basics)

### ทำไมต้องใช้ Inheritance?

Inheritance ช่วยให้เราสามารถ:
1. **นำโค้ดกลับมาใช้ใหม่** — Subclass สืบทอด property และ method จาก Superclass
2. **สร้างลำดับชั้น** — จัดระเบียบโค้ดในรูปแบบ "is-a" relationship
3. **ขยายฟังก์ชันการทำงาน** — เพิ่มหรือปรับแต่งพฤติกรรมใน Subclass
4. **ลดการซ้ำซ้อน** — รวมโค้ดที่ใช้ร่วมกันไว้ใน Superclass

### Single Inheritance ใน Swift

Swift รองรับเฉพาะ **Single Inheritance** คือ Class หนึ่งสามารถสืบทอดจาก Class แม่ได้เพียงหนึ่ง Class เท่านั้น (ต่างจาก C++ ที่รองรับ Multiple Inheritance)

```swift
// Superclass (คลาสแม่)
class Animal {
    var name: String
    var age: Int
    
    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }
    
    func makeSound() {
        print("\(name) กำลังส่งเสียง...")
    }
    
    func describe() {
        print("ชื่อ: \(name), อายุ: \(age) ปี")
    }
}

// Subclass (คลาสลูก)
class Dog: Animal {
    var breed: String
    
    init(name: String, age: Int, breed: String) {
        self.breed = breed
        super.init(name: name, age: age)  // เรียก initializer ของ Superclass
    }
}

// การใช้งาน
let dog = Dog(name: "บัดดี้", age: 3, breed: "โกลเดน รีทรีฟเวอร์")
dog.describe()     // สืบทอดจาก Animal
dog.makeSound()    // สืบทอดจาก Animal
print(dog.breed)   // property ของตัวเอง
```

**ผลลัพธ์:**
```
ชื่อ: บัดดี้, อายุ: 3 ปี
บัดดี้ กำลังส่งเสียง...
โกลเดน รีทรีฟเวอร์
```

---

## 15.2 นิยาม Subclass (Subclass Definition)

### รูปแบบการเขียน Subclass

```swift
class SubclassName: SuperclassName {
    // properties เพิ่มเติม
    // methods เพิ่มเติม
    // initializers
}
```

### ตัวอย่าง: ลำดับชั้น Vehicle

```swift
// Base Class
class Vehicle {
    var make: String
    var model: String
    var year: Int
    var speed: Double = 0.0
    
    init(make: String, model: String, year: Int) {
        self.make = make
        self.model = model
        self.year = year
    }
    
    func accelerate(by amount: Double) {
        speed += amount
        print("\(make) \(model) เพิ่มความเร็วเป็น \(speed) km/h")
    }
    
    func brake(by amount: Double) {
        speed = max(0, speed - amount)
        print("\(make) \(model) ลดความเร็วเหลือ \(speed) km/h")
    }
    
    func describe() {
        print("รถ: \(year) \(make) \(model)")
    }
}

// Subclass ระดับ 1
class Car: Vehicle {
    var numberOfDoors: Int
    var isConvertible: Bool
    
    init(make: String, model: String, year: Int, doors: Int, convertible: Bool = false) {
        self.numberOfDoors = doors
        self.isConvertible = convertible
        super.init(make: make, model: model, year: year)
    }
    
    func openSunroof() {
        if isConvertible {
            print("เปิดหลังคา \(make) \(model)")
        } else {
            print("รถคันนี้ไม่มีหลังคาที่เปิดได้")
        }
    }
}

// Subclass ระดับ 2
class ElectricCar: Car {
    var batteryCapacity: Double  // กิโลวัตต์-ชั่วโมง
    var chargeLevel: Double = 100.0  // เปอร์เซ็นต์
    
    init(make: String, model: String, year: Int, doors: Int, battery: Double) {
        self.batteryCapacity = battery
        super.init(make: make, model: model, year: year, doors: doors)
    }
    
    func charge() {
        chargeLevel = 100.0
        print("ชาร์จ \(make) \(model) เต็มแล้ว (\(batteryCapacity) kWh)")
    }
    
    func checkRange() -> Double {
        // คำนวณระยะทางประมาณจากระดับแบตเตอรี่
        let estimatedRange = batteryCapacity * chargeLevel / 100 * 5.5
        return estimatedRange
    }
}

// การใช้งาน
let myCar = Car(make: "Toyota", model: "Camry", year: 2023, doors: 4)
myCar.describe()
myCar.accelerate(by: 60)
myCar.openSunroof()

print("---")

let myEV = ElectricCar(make: "Tesla", model: "Model 3", year: 2024, doors: 4, battery: 75)
myEV.describe()       // สืบทอดจาก Vehicle
myEV.accelerate(by: 100)  // สืบทอดจาก Vehicle
myEV.charge()
print("ระยะทางโดยประมาณ: \(myEV.checkRange()) km")
```

---

## 15.3 การเรียก Superclass Initializers

### กฎการเรียก super.init()

เมื่อ Subclass มี Designated Initializer ของตัวเอง จะต้องเรียก `super.init()` เพื่อ initialize property ของ Superclass

```swift
class Person {
    var firstName: String
    var lastName: String
    var birthYear: Int
    
    init(firstName: String, lastName: String, birthYear: Int) {
        self.firstName = firstName
        self.lastName = lastName
        self.birthYear = birthYear
    }
    
    var fullName: String {
        return "\(firstName) \(lastName)"
    }
    
    var age: Int {
        return 2024 - birthYear
    }
}

class Employee: Person {
    var employeeID: String
    var department: String
    var salary: Double
    
    init(firstName: String, lastName: String, birthYear: Int,
         employeeID: String, department: String, salary: Double) {
        // Step 1: Initialize properties ของ Subclass ก่อน
        self.employeeID = employeeID
        self.department = department
        self.salary = salary
        // Step 2: เรียก super.init()
        super.init(firstName: firstName, lastName: lastName, birthYear: birthYear)
        // Step 3: หลังจาก super.init() ถึงจะใช้ self ได้อย่างเต็มที่
    }
    
    func introduce() {
        print("สวัสดี ผม/หนู \(fullName) รหัสพนักงาน \(employeeID) แผนก \(department)")
    }
}

class Manager: Employee {
    var teamSize: Int
    var directReports: [String]
    
    init(firstName: String, lastName: String, birthYear: Int,
         employeeID: String, department: String, salary: Double,
         teamSize: Int) {
        self.teamSize = teamSize
        self.directReports = []
        super.init(firstName: firstName, lastName: lastName, birthYear: birthYear,
                   employeeID: employeeID, department: department, salary: salary)
    }
    
    func addReport(_ name: String) {
        directReports.append(name)
    }
    
    func listTeam() {
        print("ทีมของ \(fullName):")
        for member in directReports {
            print("  - \(member)")
        }
    }
}

// การใช้งาน
let emp = Employee(firstName: "สมชาย", lastName: "ใจดี", birthYear: 1990,
                   employeeID: "EMP001", department: "IT", salary: 50000)
emp.introduce()

let mgr = Manager(firstName: "สมหญิง", lastName: "รักงาน", birthYear: 1985,
                  employeeID: "MGR001", department: "Engineering", salary: 80000,
                  teamSize: 5)
mgr.addReport("สมชาย ใจดี")
mgr.addReport("ประสิทธิ์ เก่งมาก")
mgr.listTeam()
```

---

## 15.4 การ Override Methods

### override keyword

ใช้ `override` เพื่อเขียน method ใหม่ใน Subclass ที่มีชื่อเดียวกับ Superclass

```swift
class Shape {
    var color: String
    
    init(color: String = "ขาว") {
        self.color = color
    }
    
    func area() -> Double {
        return 0.0
    }
    
    func perimeter() -> Double {
        return 0.0
    }
    
    func describe() {
        print("รูปทรง: สี\(color), พื้นที่: \(String(format: "%.2f", area())), เส้นรอบรูป: \(String(format: "%.2f", perimeter()))")
    }
}

class Circle: Shape {
    var radius: Double
    
    init(radius: Double, color: String = "แดง") {
        self.radius = radius
        super.init(color: color)
    }
    
    // Override method จาก Shape
    override func area() -> Double {
        return Double.pi * radius * radius
    }
    
    override func perimeter() -> Double {
        return 2 * Double.pi * radius
    }
    
    override func describe() {
        print("วงกลม (รัศมี \(radius) หน่วย):")
        super.describe()  // เรียก method ของ Superclass
    }
}

class Rectangle: Shape {
    var width: Double
    var height: Double
    
    init(width: Double, height: Double, color: String = "น้ำเงิน") {
        self.width = width
        self.height = height
        super.init(color: color)
    }
    
    override func area() -> Double {
        return width * height
    }
    
    override func perimeter() -> Double {
        return 2 * (width + height)
    }
    
    override func describe() {
        print("สี่เหลี่ยม (\(width) x \(height) หน่วย):")
        super.describe()
    }
}

class Square: Rectangle {
    init(side: Double, color: String = "เขียว") {
        super.init(width: side, height: side, color: color)
    }
    
    override func describe() {
        print("สี่เหลี่ยมจัตุรัส (ด้าน \(width) หน่วย):")
        // เรียก Shape's describe โดยข้าม Rectangle (ไม่สามารถทำได้โดยตรง)
        // ต้องเรียกผ่าน super ซึ่งจะเรียก Rectangle's describe
        super.describe()
    }
}

// การใช้งาน
let circle = Circle(radius: 5)
circle.describe()

let rect = Rectangle(width: 4, height: 6)
rect.describe()

let square = Square(side: 5)
square.describe()
```

**ผลลัพธ์:**
```
วงกลม (รัศมี 5.0 หน่วย):
รูปทรง: สีแดง, พื้นที่: 78.54, เส้นรอบรูป: 31.42
สี่เหลี่ยม (4.0 x 6.0 หน่วย):
รูปทรง: สีน้ำเงิน, พื้นที่: 24.00, เส้นรอบรูป: 20.00
สี่เหลี่ยมจัตุรัส (ด้าน 5.0 หน่วย):
สี่เหลี่ยม (5.0 x 5.0 หน่วย):
รูปทรง: สีเขียว, พื้นที่: 25.00, เส้นรอบรูป: 20.00
```

---

## 15.5 การ Override Properties

### Override Computed Properties

```swift
class BankAccount {
    var owner: String
    var balance: Double
    
    init(owner: String, balance: Double = 0) {
        self.owner = owner
        self.balance = balance
    }
    
    var interestRate: Double {
        return 0.01  // 1% ต่อปี
    }
    
    var annualInterest: Double {
        return balance * interestRate
    }
    
    func deposit(_ amount: Double) {
        balance += amount
        print("ฝากเงิน \(amount) บาท ยอดคงเหลือ: \(balance) บาท")
    }
    
    func withdraw(_ amount: Double) -> Bool {
        if amount <= balance {
            balance -= amount
            print("ถอนเงิน \(amount) บาท ยอดคงเหลือ: \(balance) บาท")
            return true
        }
        print("ยอดเงินไม่เพียงพอ")
        return false
    }
}

class SavingsAccount: BankAccount {
    // Override computed property
    override var interestRate: Double {
        return 0.025  // 2.5% ต่อปี
    }
    
    var withdrawalLimit: Int = 3
    var withdrawalCount: Int = 0
    
    override func withdraw(_ amount: Double) -> Bool {
        if withdrawalCount >= withdrawalLimit {
            print("เกินจำนวนครั้งถอนเงินสูงสุดต่อเดือน (\(withdrawalLimit) ครั้ง)")
            return false
        }
        let result = super.withdraw(amount)
        if result {
            withdrawalCount += 1
            print("ถอนครั้งที่ \(withdrawalCount) (เหลือ \(withdrawalLimit - withdrawalCount) ครั้ง)")
        }
        return result
    }
}

class PremiumAccount: BankAccount {
    var tier: String
    
    init(owner: String, balance: Double, tier: String) {
        self.tier = tier
        super.init(owner: owner, balance: balance)
    }
    
    override var interestRate: Double {
        switch tier {
        case "Silver": return 0.03
        case "Gold": return 0.04
        case "Platinum": return 0.05
        default: return 0.02
        }
    }
}

// การใช้งาน
let savings = SavingsAccount(owner: "คุณสมชาย", balance: 100000)
print("อัตราดอกเบี้ย: \(savings.interestRate * 100)%")
print("ดอกเบี้ยต่อปี: \(savings.annualInterest) บาท")

savings.withdraw(5000)
savings.withdraw(3000)
savings.withdraw(2000)
savings.withdraw(1000)  // เกินลิมิต

print("---")
let premium = PremiumAccount(owner: "คุณสมหญิง", balance: 500000, tier: "Platinum")
print("อัตราดอกเบี้ย Platinum: \(premium.interestRate * 100)%")
print("ดอกเบี้ยต่อปี: \(premium.annualInterest) บาท")
```

### Override Stored Property (willSet/didSet)

```swift
class TemperatureSensor {
    var temperature: Double = 0.0 {
        didSet {
            if temperature > 100 {
                print("คำเตือน: อุณหภูมิสูงเกินไป (\(temperature)°C)")
            }
        }
    }
    
    func readTemperature() {
        print("อุณหภูมิปัจจุบัน: \(temperature)°C")
    }
}

class SmartSensor: TemperatureSensor {
    var alertThreshold: Double = 80.0
    var alertHistory: [Double] = []
    
    // Override stored property โดยเพิ่ม observer
    override var temperature: Double {
        willSet {
            print("กำลังอัพเดตอุณหภูมิจาก \(temperature)°C เป็น \(newValue)°C")
        }
        didSet {
            if temperature >= alertThreshold {
                alertHistory.append(temperature)
                print("แจ้งเตือน: อุณหภูมิ \(temperature)°C เกินค่า \(alertThreshold)°C")
            }
            // เรียก super's didSet ไม่ได้โดยตรง แต่ผลของ super ก็ยังทำงาน
        }
    }
}

let smartSensor = SmartSensor()
smartSensor.temperature = 75.0
smartSensor.temperature = 85.0
smartSensor.temperature = 95.0
print("ประวัติการแจ้งเตือน: \(smartSensor.alertHistory)")
```

---

## 15.6 การ Override Subscripts

```swift
class Matrix {
    var rows: Int
    var cols: Int
    var data: [[Double]]
    
    init(rows: Int, cols: Int) {
        self.rows = rows
        self.cols = cols
        self.data = Array(repeating: Array(repeating: 0.0, count: cols), count: rows)
    }
    
    subscript(row: Int, col: Int) -> Double {
        get {
            precondition(row >= 0 && row < rows && col >= 0 && col < cols, "Index out of bounds")
            return data[row][col]
        }
        set {
            precondition(row >= 0 && row < rows && col >= 0 && col < cols, "Index out of bounds")
            data[row][col] = newValue
        }
    }
    
    func printMatrix() {
        for row in data {
            print(row.map { String(format: "%6.1f", $0) }.joined(separator: " "))
        }
    }
}

class BoundedMatrix: Matrix {
    var minValue: Double
    var maxValue: Double
    
    init(rows: Int, cols: Int, min: Double, max: Double) {
        self.minValue = min
        self.maxValue = max
        super.init(rows: rows, cols: cols)
    }
    
    // Override subscript เพื่อ clamp ค่า
    override subscript(row: Int, col: Int) -> Double {
        get {
            return super[row, col]
        }
        set {
            // จำกัดค่าอยู่ระหว่าง min และ max
            let clampedValue = min(max(newValue, minValue), maxValue)
            super[row, col] = clampedValue
        }
    }
}

// การใช้งาน
let matrix = Matrix(rows: 3, cols: 3)
matrix[0, 0] = 1.0
matrix[1, 1] = 5.0
matrix[2, 2] = 9.0
matrix.printMatrix()

print("---")
let bounded = BoundedMatrix(rows: 2, cols: 2, min: 0, max: 10)
bounded[0, 0] = 5.0
bounded[0, 1] = 15.0   // จะถูก clamp เป็น 10
bounded[1, 0] = -3.0   // จะถูก clamp เป็น 0
bounded[1, 1] = 7.5
bounded.printMatrix()
```

---

## 15.7 การป้องกันการ Override ด้วย final

### final class

```swift
final class Configuration {
    static let shared = Configuration()
    private var settings: [String: Any] = [:]
    
    private init() {
        settings["theme"] = "dark"
        settings["language"] = "th"
    }
    
    func set(key: String, value: Any) {
        settings[key] = value
    }
    
    func get(key: String) -> Any? {
        return settings[key]
    }
}

// Error: ไม่สามารถสืบทอด final class ได้
// class CustomConfig: Configuration { }  // Compile Error!
```

### final method และ final property

```swift
class Animal {
    var name: String
    
    init(name: String) {
        self.name = name
    }
    
    // final method - ไม่สามารถ override ได้
    final func breathe() {
        print("\(name) กำลังหายใจ")
    }
    
    // final property - ไม่สามารถ override ได้
    final var isAlive: Bool {
        return true
    }
    
    // method ที่ override ได้
    func makeSound() {
        print("\(name) ส่งเสียง...")
    }
}

class Dog: Animal {
    init(name: String) {
        super.init(name: name)
    }
    
    override func makeSound() {
        print("\(name): โฮ่ง โฮ่ง!")
    }
    
    // Error: ไม่สามารถ override final method
    // override func breathe() { }  // Compile Error!
    
    // Error: ไม่สามารถ override final property
    // override var isAlive: Bool { return false }  // Compile Error!
}
```

---

## 15.8 Polymorphism (พหุสัณฐาน)

Polymorphism ช่วยให้ Object ของ Subclass สามารถถูกใช้งานเป็น Type ของ Superclass ได้

```swift
class Shape {
    var color: String
    
    init(color: String) {
        self.color = color
    }
    
    func area() -> Double { return 0 }
    func draw() {
        print("วาดรูปทรงสี\(color)")
    }
}

class Circle: Shape {
    var radius: Double
    
    init(radius: Double, color: String) {
        self.radius = radius
        super.init(color: color)
    }
    
    override func area() -> Double {
        return Double.pi * radius * radius
    }
    
    override func draw() {
        print("วาดวงกลมสี\(color) รัศมี \(radius)")
    }
}

class Rectangle: Shape {
    var width: Double
    var height: Double
    
    init(width: Double, height: Double, color: String) {
        self.width = width
        self.height = height
        super.init(color: color)
    }
    
    override func area() -> Double {
        return width * height
    }
    
    override func draw() {
        print("วาดสี่เหลี่ยมสี\(color) \(width)x\(height)")
    }
}

class Triangle: Shape {
    var base: Double
    var height: Double
    
    init(base: Double, height: Double, color: String) {
        self.base = base
        self.height = height
        super.init(color: color)
    }
    
    override func area() -> Double {
        return 0.5 * base * height
    }
    
    override func draw() {
        print("วาดสามเหลี่ยมสี\(color) ฐาน \(base) สูง \(height)")
    }
}

// Polymorphism ในการทำงาน
var shapes: [Shape] = [
    Circle(radius: 5, color: "แดง"),
    Rectangle(width: 4, height: 6, color: "น้ำเงิน"),
    Triangle(base: 3, height: 8, color: "เขียว"),
    Circle(radius: 2, color: "เหลือง"),
    Rectangle(width: 10, height: 2, color: "ม่วง")
]

// Loop เดียวสามารถทำงานกับ Object ต่างประเภทได้
for shape in shapes {
    shape.draw()
    print("  พื้นที่: \(String(format: "%.2f", shape.area())) ตร.หน่วย")
}

// คำนวณพื้นที่รวม
let totalArea = shapes.reduce(0) { $0 + $1.area() }
print("\nพื้นที่รวมทั้งหมด: \(String(format: "%.2f", totalArea)) ตร.หน่วย")
```

---

## 15.9 Dynamic Dispatch และ Static Dispatch

### Dynamic Dispatch

เกิดขึ้นเมื่อ Swift ตัดสินใจว่าจะเรียก method implementation ใดในขณะ runtime (ไม่ใช่ compile time)

```swift
class Logger {
    func log(_ message: String) {
        print("[LOG]: \(message)")
    }
}

class FileLogger: Logger {
    var filename: String
    
    init(filename: String) {
        self.filename = filename
    }
    
    override func log(_ message: String) {
        print("[FILE:\(filename)]: \(message)")
        // ในระบบจริงจะเขียนลงไฟล์
    }
}

class NetworkLogger: Logger {
    var endpoint: String
    
    init(endpoint: String) {
        self.endpoint = endpoint
    }
    
    override func log(_ message: String) {
        print("[NETWORK:\(endpoint)]: \(message)")
        // ในระบบจริงจะส่งผ่าน network
    }
}

// Dynamic dispatch: Swift รู้ตอน runtime ว่าจะเรียก method ใด
func logMessage(_ message: String, using logger: Logger) {
    logger.log(message)  // Dynamic dispatch!
}

let loggers: [Logger] = [
    Logger(),
    FileLogger(filename: "app.log"),
    NetworkLogger(endpoint: "https://logs.example.com")
]

for logger in loggers {
    logMessage("เกิดข้อผิดพลาด", using: logger)
}
```

### Static Dispatch

เกิดขึ้นกับ `final` methods, `static` methods และ global functions

```swift
class Calculator {
    // Static dispatch - รู้ตอน compile time
    final func add(_ a: Double, _ b: Double) -> Double {
        return a + b
    }
    
    static func multiply(_ a: Double, _ b: Double) -> Double {
        return a * b
    }
}

// Protocol extension ใช้ static dispatch (ในบางกรณี)
protocol Printable {
    func print()
}

extension Printable {
    // Static dispatch ผ่าน protocol extension
    func printDescription() {
        Swift.print("กำลัง print...")
        self.print()
    }
}
```

---

## 15.10 Method Resolution

Swift ใช้ vtable (virtual dispatch table) สำหรับ class methods เพื่อ resolve method calls ในขณะ runtime

```swift
// ตัวอย่างการทำความเข้าใจ Method Resolution
class Base {
    func method1() { print("Base.method1") }
    func method2() { print("Base.method2") }
}

class Derived: Base {
    override func method1() { print("Derived.method1") }
    // method2 ไม่ได้ override - ยังคงใช้ Base.method2
    func method3() { print("Derived.method3") }
}

let obj1: Base = Base()
let obj2: Base = Derived()  // Polymorphism - type เป็น Base แต่ object เป็น Derived
let obj3: Derived = Derived()

obj1.method1()  // Base.method1
obj1.method2()  // Base.method2

obj2.method1()  // Derived.method1 (Dynamic dispatch!)
obj2.method2()  // Base.method2 (ไม่ได้ override)

obj3.method1()  // Derived.method1
obj3.method2()  // Base.method2
obj3.method3()  // Derived.method3 (เรียกได้เพราะ type เป็น Derived)
// obj2.method3()  // Error! type เป็น Base ไม่รู้จัก method3
```

---

## 15.11 Abstract Class Pattern โดยใช้ Protocols

Swift ไม่มี `abstract class` โดยตรง แต่เราสามารถสร้าง pattern ที่คล้ายกันได้

```swift
// Protocol แทน Abstract Class
protocol AbstractShape {
    var color: String { get }
    func area() -> Double      // Abstract method - ต้อง implement
    func perimeter() -> Double // Abstract method - ต้อง implement
}

extension AbstractShape {
    // Default implementations (เหมือน concrete methods ใน abstract class)
    func describe() {
        print("รูปทรง: สี\(color)")
        print("  พื้นที่: \(String(format: "%.2f", area()))")
        print("  เส้นรอบรูป: \(String(format: "%.2f", perimeter()))")
    }
    
    func isLargerThan(_ other: AbstractShape) -> Bool {
        return area() > other.area()
    }
}

// Concrete implementations
struct ConcreteCircle: AbstractShape {
    let color: String
    let radius: Double
    
    func area() -> Double { return Double.pi * radius * radius }
    func perimeter() -> Double { return 2 * Double.pi * radius }
}

struct ConcreteRectangle: AbstractShape {
    let color: String
    let width: Double
    let height: Double
    
    func area() -> Double { return width * height }
    func perimeter() -> Double { return 2 * (width + height) }
}

// อีกวิธี: ใช้ class พร้อม fatalError สำหรับ "abstract" methods
class AbstractAnimal {
    var name: String
    
    init(name: String) {
        self.name = name
    }
    
    // "Abstract" method - ต้อง override ใน subclass
    func makeSound() -> String {
        fatalError("Subclass ต้อง implement makeSound()")
    }
    
    // Concrete method
    func describe() {
        print("\(name) พูดว่า: \(makeSound())")
    }
}

class Cat: AbstractAnimal {
    override func makeSound() -> String {
        return "เมี้ยว!"
    }
}

class Dog: AbstractAnimal {
    override func makeSound() -> String {
        return "โฮ่ง!"
    }
}

let animals: [AbstractAnimal] = [Cat(name: "แมวส้ม"), Dog(name: "หมาดำ")]
animals.forEach { $0.describe() }
```

---

## 15.12 Downcasting (as?, as!)

### is operator - ตรวจสอบประเภท

```swift
class MediaItem {
    var title: String
    init(title: String) {
        self.title = title
    }
}

class Movie: MediaItem {
    var director: String
    var duration: Int  // นาที
    
    init(title: String, director: String, duration: Int) {
        self.director = director
        self.duration = duration
        super.init(title: title)
    }
}

class Song: MediaItem {
    var artist: String
    var albumName: String
    
    init(title: String, artist: String, albumName: String) {
        self.artist = artist
        self.albumName = albumName
        super.init(title: title)
    }
}

class Podcast: MediaItem {
    var host: String
    var episodeNumber: Int
    
    init(title: String, host: String, episodeNumber: Int) {
        self.host = host
        self.episodeNumber = episodeNumber
        super.init(title: title)
    }
}

let library: [MediaItem] = [
    Movie(title: "อวตาร", director: "เจมส์ คาเมรอน", duration: 162),
    Song(title: "ลาก่อน", artist: "บอดี้สแลม", albumName: "Best Of"),
    Podcast(title: "Tech Talk", host: "สมชาย", episodeNumber: 42),
    Movie(title: "Inception", director: "คริสโตเฟอร์ โนแลน", duration: 148),
    Song(title: "คิดถึง", artist: "เบิร์ด ธงไชย", albumName: "Classic"),
]

// ใช้ is เพื่อตรวจสอบประเภท
var movieCount = 0
var songCount = 0
var podcastCount = 0

for item in library {
    if item is Movie { movieCount += 1 }
    else if item is Song { songCount += 1 }
    else if item is Podcast { podcastCount += 1 }
}

print("ภาพยนตร์: \(movieCount), เพลง: \(songCount), พอดแคสต์: \(podcastCount)")
```

### as? - Optional Downcasting (ปลอดภัย)

```swift
// ใช้ as? กับ if let
for item in library {
    if let movie = item as? Movie {
        print("ภาพยนตร์: \(movie.title) กำกับโดย \(movie.director) (\(movie.duration) นาที)")
    } else if let song = item as? Song {
        print("เพลง: \(song.title) โดย \(song.artist)")
    } else if let podcast = item as? Podcast {
        print("พอดแคสต์: \(podcast.title) EP.\(podcast.episodeNumber)")
    }
}

// ใช้ as? กับ switch
print("\n--- ใช้ switch ---")
for item in library {
    switch item {
    case let movie as Movie:
        print("🎬 \(movie.title) (\(movie.duration) นาที)")
    case let song as Song:
        print("🎵 \(song.title) - \(song.artist)")
    case let podcast as Podcast:
        print("🎙 \(podcast.title) EP.\(podcast.episodeNumber)")
    default:
        print("❓ ไม่รู้จักประเภท")
    }
}
```

### as! - Forced Downcasting (อันตราย)

```swift
// as! จะ crash ถ้า downcast ล้มเหลว - ใช้เมื่อแน่ใจ 100%
let firstItem = library[0]

// แน่ใจว่า item แรกเป็น Movie
let firstMovie = firstItem as! Movie
print("ภาพยนตร์แรก: \(firstMovie.title)")

// ถ้าไม่แน่ใจ ควรใช้ as? แทน
if let movie = library[1] as? Movie {
    print("รายการที่ 2 เป็นภาพยนตร์: \(movie.title)")
} else {
    print("รายการที่ 2 ไม่ใช่ภาพยนตร์")
}
```

---

## 15.13 Type Checking (is)

```swift
class Vehicle {
    var name: String
    init(name: String) { self.name = name }
}
class Car: Vehicle {}
class Truck: Vehicle {}
class Bus: Vehicle {}

let vehicles: [Vehicle] = [
    Car(name: "Toyota Camry"),
    Truck(name: "Isuzu D-Max"),
    Car(name: "Honda Civic"),
    Bus(name: "รถเมล์สาย 8"),
    Truck(name: "Hino 300"),
]

// นับแต่ละประเภท
let cars = vehicles.filter { $0 is Car }
let trucks = vehicles.filter { $0 is Truck }
let buses = vehicles.filter { $0 is Bus }

print("รถยนต์: \(cars.count) คัน")
print("รถบรรทุก: \(trucks.count) คัน")
print("รถบัส: \(buses.count) คัน")

// ตรวจสอบ type hierarchy
class A {}
class B: A {}
class C: B {}

let c = C()
print("\nc is C: \(c is C)")  // true
print("c is B: \(c is B)")  // true - B เป็น ancestor ของ C
print("c is A: \(c is A)")  // true - A เป็น ancestor ของ C
```

---

## 15.14 Multiple Levels of Inheritance

```swift
// ลำดับชั้นหลายระดับ
class LivingThing {
    var isAlive: Bool = true
    
    func breathe() {
        print("กำลังหายใจ")
    }
}

class Animal: LivingThing {
    var name: String
    var age: Int
    
    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }
    
    func eat() {
        print("\(name) กำลังกินอาหาร")
    }
    
    func move() {
        print("\(name) กำลังเคลื่อนที่")
    }
}

class Mammal: Animal {
    var furColor: String
    
    init(name: String, age: Int, furColor: String) {
        self.furColor = furColor
        super.init(name: name, age: age)
    }
    
    func nurseYoung() {
        print("\(name) เลี้ยงลูกด้วยนม")
    }
}

class Pet: Mammal {
    var owner: String
    
    init(name: String, age: Int, furColor: String, owner: String) {
        self.owner = owner
        super.init(name: name, age: age, furColor: furColor)
    }
    
    func greetOwner() {
        print("\(name) ต้อนรับ \(owner) กลับบ้าน!")
    }
}

class GoldenRetriever: Pet {
    var trainingLevel: Int
    
    init(name: String, age: Int, owner: String, trainingLevel: Int) {
        self.trainingLevel = trainingLevel
        super.init(name: name, age: age, furColor: "ทอง", owner: owner)
    }
    
    override func move() {
        print("\(name) วิ่งเล่นสนุกสนาน!")
    }
    
    func fetch() {
        if trainingLevel >= 3 {
            print("\(name) ไปเอาลูกบอลมาให้แล้ว!")
        } else {
            print("\(name) ยังไม่ได้รับการฝึก fetch")
        }
    }
}

// การใช้งาน
let buddy = GoldenRetriever(name: "บัดดี้", age: 2, owner: "คุณสมชาย", trainingLevel: 4)

// Method จากทุกระดับ
buddy.breathe()       // LivingThing
buddy.eat()           // Animal
buddy.nurseYoung()    // Mammal
buddy.greetOwner()    // Pet
buddy.fetch()         // GoldenRetriever
buddy.move()          // Override จาก Animal

print("\nขนสี: \(buddy.furColor)")  // Property จาก Mammal
print("เจ้าของ: \(buddy.owner)")   // Property จาก Pet
print("ยังมีชีวิต: \(buddy.isAlive)") // Property จาก LivingThing
```

---

## 15.15 กฎการสืบทอด Initializer

### Designated vs Convenience Initializers

```swift
class Food {
    var name: String
    var calories: Int
    var isVegetarian: Bool
    
    // Designated Initializer
    init(name: String, calories: Int, isVegetarian: Bool = false) {
        self.name = name
        self.calories = calories
        self.isVegetarian = isVegetarian
    }
    
    // Convenience Initializer
    convenience init(name: String) {
        self.init(name: name, calories: 0)  // เรียก designated initializer
    }
}

class Recipe: Food {
    var ingredients: [String]
    var cookingTime: Int  // นาที
    
    // Designated Initializer ของ Recipe
    init(name: String, calories: Int, isVegetarian: Bool,
         ingredients: [String], cookingTime: Int) {
        self.ingredients = ingredients
        self.cookingTime = cookingTime
        super.init(name: name, calories: calories, isVegetarian: isVegetarian)
    }
    
    // Convenience Initializer ใน Subclass
    convenience init(name: String, ingredients: [String]) {
        self.init(name: name, calories: 0, isVegetarian: false,
                  ingredients: ingredients, cookingTime: 30)
    }
    
    func printRecipe() {
        print("สูตร: \(name) (\(calories) แคลอรี่)")
        print("ส่วนผสม: \(ingredients.joined(separator: ", "))")
        print("เวลาปรุง: \(cookingTime) นาที")
    }
}

let recipe1 = Recipe(name: "ผัดไทย", calories: 450, isVegetarian: false,
                     ingredients: ["เส้น", "กุ้ง", "ไข่", "ถั่วงอก"], cookingTime: 20)
recipe1.printRecipe()

let recipe2 = Recipe(name: "ต้มยำ", ingredients: ["กุ้ง", "เห็ด", "ข่า", "ตะไคร้"])
recipe2.printRecipe()
```

---

## 15.16 Required Initializers

```swift
class UIControl {
    var isEnabled: Bool
    var tag: Int
    
    // required init ต้องถูก implement ใน subclass ทุกตัว
    required init(isEnabled: Bool = true, tag: Int = 0) {
        self.isEnabled = isEnabled
        self.tag = tag
    }
    
    func handleTap() {
        if isEnabled {
            print("Control ถูกแตะ (tag: \(tag))")
        }
    }
}

class UIButton: UIControl {
    var title: String
    
    init(title: String, isEnabled: Bool = true, tag: Int = 0) {
        self.title = title
        super.init(isEnabled: isEnabled, tag: tag)
    }
    
    // ต้อง implement required init
    required init(isEnabled: Bool = true, tag: Int = 0) {
        self.title = "Button"
        super.init(isEnabled: isEnabled, tag: tag)
    }
    
    override func handleTap() {
        if isEnabled {
            print("ปุ่ม '\(title)' ถูกกด")
        }
    }
}

class UITextField: UIControl {
    var placeholder: String
    var text: String = ""
    
    init(placeholder: String) {
        self.placeholder = placeholder
        super.init()
    }
    
    required init(isEnabled: Bool = true, tag: Int = 0) {
        self.placeholder = "กรอกข้อมูล..."
        super.init(isEnabled: isEnabled, tag: tag)
    }
}

let button = UIButton(title: "ยืนยัน")
button.handleTap()

let textField = UITextField(placeholder: "ชื่อผู้ใช้")
print("Placeholder: \(textField.placeholder)")
```

---

## 15.17 Convenience Initializers ใน Subclasses

```swift
class Color {
    var red: Double
    var green: Double
    var blue: Double
    var alpha: Double
    
    init(red: Double, green: Double, blue: Double, alpha: Double = 1.0) {
        self.red = red
        self.green = green
        self.blue = blue
        self.alpha = alpha
    }
    
    convenience init(white: Double, alpha: Double = 1.0) {
        self.init(red: white, green: white, blue: white, alpha: alpha)
    }
    
    convenience init(hex: String) {
        var hexString = hex.trimmingCharacters(in: .whitespaces)
        if hexString.hasPrefix("#") {
            hexString = String(hexString.dropFirst())
        }
        let scanner = Scanner(string: hexString)
        var hexNumber: UInt64 = 0
        scanner.scanHexInt64(&hexNumber)
        let r = Double((hexNumber & 0xFF0000) >> 16) / 255
        let g = Double((hexNumber & 0x00FF00) >> 8) / 255
        let b = Double(hexNumber & 0x0000FF) / 255
        self.init(red: r, green: g, blue: b)
    }
}

class ThemedColor: Color {
    var name: String
    var isDark: Bool
    
    init(name: String, red: Double, green: Double, blue: Double) {
        self.name = name
        let brightness = (red * 0.299 + green * 0.587 + blue * 0.114)
        self.isDark = brightness < 0.5
        super.init(red: red, green: green, blue: blue)
    }
    
    // Convenience initializer ใน subclass
    convenience init(name: String, hex: String) {
        // ต้องเรียก designated initializer ของ subclass ก่อน
        // แต่เราไม่มีข้อมูล RGB ตอนนี้ ดังนั้นต้อง parse ก่อน
        var hexString = hex.hasPrefix("#") ? String(hex.dropFirst()) : hex
        let scanner = Scanner(string: hexString)
        var hexNumber: UInt64 = 0
        scanner.scanHexInt64(&hexNumber)
        let r = Double((hexNumber & 0xFF0000) >> 16) / 255
        let g = Double((hexNumber & 0x00FF00) >> 8) / 255
        let b = Double(hexNumber & 0x0000FF) / 255
        self.init(name: name, red: r, green: g, blue: b)
    }
}

let primaryColor = ThemedColor(name: "ฟ้าสว่าง", hex: "#007AFF")
print("สี: \(primaryColor.name)")
print("มืด: \(primaryColor.isDark)")
print("RGB: (\(String(format: "%.2f", primaryColor.red)), \(String(format: "%.2f", primaryColor.green)), \(String(format: "%.2f", primaryColor.blue)))")
```

---

## 15.18 การออกแบบ Class Hierarchies

### หลักการ Liskov Substitution Principle (LSP)

```swift
// ตัวอย่างที่ถูกต้อง: Bird hierarchy
class Bird {
    var name: String
    var wingspan: Double  // ซม.
    
    init(name: String, wingspan: Double) {
        self.name = name
        self.wingspan = wingspan
    }
    
    func eat() {
        print("\(name) กำลังกินอาหาร")
    }
    
    func sleep() {
        print("\(name) กำลังนอนหลับ")
    }
}

// Protocol สำหรับความสามารถบิน - ไม่ใช่ทุก Bird บินได้
protocol Flyable {
    var maxAltitude: Double { get }
    func fly()
    func land()
}

protocol Swimmable {
    var maxDepth: Double { get }
    func swim()
    func dive()
}

class Eagle: Bird, Flyable {
    var maxAltitude: Double = 3000  // เมตร
    
    init(name: String, wingspan: Double) {
        super.init(name: name, wingspan: wingspan)
    }
    
    func fly() {
        print("\(name) บินสูง \(maxAltitude) เมตร")
    }
    
    func land() {
        print("\(name) ลงจอดบนหิน")
    }
    
    func hunt() {
        print("\(name) โฉบล่าเหยื่อ")
    }
}

class Penguin: Bird, Swimmable {
    var maxDepth: Double = 500  // เมตร
    
    init(name: String, wingspan: Double) {
        super.init(name: name, wingspan: wingspan)
    }
    
    func swim() {
        print("\(name) ว่ายน้ำอย่างคล่องแคล่ว")
    }
    
    func dive() {
        print("\(name) ดำน้ำลึก \(maxDepth) เมตร")
    }
}

class Duck: Bird, Flyable, Swimmable {
    var maxAltitude: Double = 500
    var maxDepth: Double = 2
    
    init(name: String, wingspan: Double) {
        super.init(name: name, wingspan: wingspan)
    }
    
    func fly() { print("\(name) บิน") }
    func land() { print("\(name) ลงน้ำ") }
    func swim() { print("\(name) ว่ายน้ำ") }
    func dive() { print("\(name) ดำน้ำ") }
}

// การใช้งาน
let birds: [Bird] = [
    Eagle(name: "อินทรี", wingspan: 200),
    Penguin(name: "เพนกวิน", wingspan: 30),
    Duck(name: "เป็ด", wingspan: 60)
]

for bird in birds {
    bird.eat()
    if let flyer = bird as? Flyable {
        flyer.fly()
    }
    if let swimmer = bird as? Swimmable {
        swimmer.swim()
    }
}
```

---

## 15.19 ตัวอย่างจริง: Shape Hierarchy

```swift
import Foundation

// MARK: - Base Shape
class Shape2D {
    private static var nextID = 1
    
    let id: Int
    var color: String
    var fillColor: String?
    var strokeWidth: Double
    
    init(color: String, strokeWidth: Double = 1.0) {
        self.id = Shape2D.nextID
        Shape2D.nextID += 1
        self.color = color
        self.strokeWidth = strokeWidth
    }
    
    // Abstract-like methods
    var area: Double { return 0 }
    var perimeter: Double { return 0 }
    var boundingBox: (width: Double, height: Double) { return (0, 0) }
    
    func draw() {
        print("กำลังวาดรูปทรง ID:\(id) สี:\(color)")
    }
    
    func scale(by factor: Double) {
        print("ขยาย/ย่อ Shape ID:\(id) ด้วย factor \(factor)")
    }
    
    func move(dx: Double, dy: Double) {
        print("ย้าย Shape ID:\(id) โดย dx:\(dx), dy:\(dy)")
    }
}

// MARK: - Circle
class Circle2D: Shape2D {
    var center: (x: Double, y: Double)
    var radius: Double
    
    init(center: (Double, Double), radius: Double, color: String) {
        self.center = center
        self.radius = radius
        super.init(color: color)
    }
    
    override var area: Double {
        return Double.pi * radius * radius
    }
    
    override var perimeter: Double {
        return 2 * Double.pi * radius
    }
    
    override var boundingBox: (width: Double, height: Double) {
        return (radius * 2, radius * 2)
    }
    
    override func draw() {
        print("วงกลม ID:\(id) ศูนย์กลาง(\(center.x),\(center.y)) รัศมี:\(radius)")
    }
    
    override func scale(by factor: Double) {
        radius *= factor
        print("ขยายวงกลม ID:\(id) รัศมีใหม่: \(radius)")
    }
    
    override func move(dx: Double, dy: Double) {
        center.x += dx
        center.y += dy
        print("ย้ายวงกลม ID:\(id) ไปที่ (\(center.x),\(center.y))")
    }
}

// MARK: - Polygon
class Polygon: Shape2D {
    var vertices: [(x: Double, y: Double)]
    
    init(vertices: [(Double, Double)], color: String) {
        self.vertices = vertices
        super.init(color: color)
    }
    
    override var perimeter: Double {
        var total = 0.0
        let n = vertices.count
        for i in 0..<n {
            let next = (i + 1) % n
            let dx = vertices[next].x - vertices[i].x
            let dy = vertices[next].y - vertices[i].y
            total += sqrt(dx*dx + dy*dy)
        }
        return total
    }
    
    override var area: Double {
        // Shoelace formula
        var sum = 0.0
        let n = vertices.count
        for i in 0..<n {
            let next = (i + 1) % n
            sum += vertices[i].x * vertices[next].y
            sum -= vertices[next].x * vertices[i].y
        }
        return abs(sum) / 2
    }
    
    override var boundingBox: (width: Double, height: Double) {
        let xs = vertices.map { $0.x }
        let ys = vertices.map { $0.y }
        return (xs.max()! - xs.min()!, ys.max()! - ys.min()!)
    }
    
    override func move(dx: Double, dy: Double) {
        vertices = vertices.map { (x: $0.x + dx, y: $0.y + dy) }
        print("ย้าย Polygon ID:\(id)")
    }
}

// MARK: - Rectangle (Polygon subclass)
class Rectangle2D: Polygon {
    var width: Double
    var height: Double
    
    init(x: Double, y: Double, width: Double, height: Double, color: String) {
        self.width = width
        self.height = height
        super.init(vertices: [(x, y), (x+width, y), (x+width, y+height), (x, y+height)],
                   color: color)
    }
    
    override var area: Double { return width * height }
    override var perimeter: Double { return 2 * (width + height) }
    
    override func scale(by factor: Double) {
        width *= factor
        height *= factor
        // อัพเดต vertices
        let origin = vertices[0]
        vertices = [
            origin,
            (x: origin.x + width, y: origin.y),
            (x: origin.x + width, y: origin.y + height),
            (x: origin.x, y: origin.y + height)
        ]
    }
    
    override func draw() {
        print("สี่เหลี่ยม ID:\(id) ที่ (\(vertices[0].x),\(vertices[0].y)) ขนาด \(width)x\(height)")
    }
}

// MARK: - Canvas (manages shapes)
class Canvas {
    private var shapes: [Shape2D] = []
    var backgroundColor: String = "ขาว"
    
    func addShape(_ shape: Shape2D) {
        shapes.append(shape)
        print("เพิ่ม Shape ID:\(shape.id) ลงใน Canvas")
    }
    
    func removeShape(id: Int) {
        shapes.removeAll { $0.id == id }
    }
    
    func drawAll() {
        print("\n=== วาด Canvas (พื้นหลังสี\(backgroundColor)) ===")
        shapes.forEach { $0.draw() }
    }
    
    func totalArea() -> Double {
        return shapes.reduce(0) { $0 + $1.area }
    }
    
    func largestShape() -> Shape2D? {
        return shapes.max { $0.area < $1.area }
    }
    
    func shapesOfType<T: Shape2D>(_ type: T.Type) -> [T] {
        return shapes.compactMap { $0 as? T }
    }
}

// การใช้งาน
let canvas = Canvas()
canvas.addShape(Circle2D(center: (0, 0), radius: 5, color: "แดง"))
canvas.addShape(Rectangle2D(x: 0, y: 0, width: 10, height: 6, color: "น้ำเงิน"))
canvas.addShape(Polygon(vertices: [(0,0), (5,0), (2.5,4)], color: "เขียว"))

canvas.drawAll()
print("\nพื้นที่รวม: \(String(format: "%.2f", canvas.totalArea()))")

if let largest = canvas.largestShape() {
    print("รูปทรงใหญ่ที่สุด: ID\(largest.id) พื้นที่ \(String(format: "%.2f", largest.area))")
}

let circles = canvas.shapesOfType(Circle2D.self)
print("จำนวนวงกลม: \(circles.count)")
```

---

## 15.20 ตัวอย่างจริง: Employee Hierarchy

```swift
// MARK: - Employee System

enum Department: String, CaseIterable {
    case engineering = "วิศวกรรม"
    case marketing = "การตลาด"
    case hr = "ทรัพยากรมนุษย์"
    case finance = "การเงิน"
    case operations = "ปฏิบัติการ"
}

class Employee {
    let employeeID: String
    var firstName: String
    var lastName: String
    var department: Department
    var startDate: Date
    var baseSalary: Double
    
    static var employeeCount = 0
    
    init(firstName: String, lastName: String, department: Department,
         baseSalary: Double, startDate: Date = Date()) {
        Employee.employeeCount += 1
        self.employeeID = "EMP\(String(format: "%04d", Employee.employeeCount))"
        self.firstName = firstName
        self.lastName = lastName
        self.department = department
        self.baseSalary = baseSalary
        self.startDate = startDate
    }
    
    var fullName: String { "\(firstName) \(lastName)" }
    
    var yearsOfService: Int {
        let calendar = Calendar.current
        return calendar.dateComponents([.year], from: startDate, to: Date()).year ?? 0
    }
    
    // Override ใน subclass
    func calculateBonus() -> Double {
        return baseSalary * 0.05  // 5% ของเงินเดือน
    }
    
    func calculateTotalCompensation() -> Double {
        return baseSalary + calculateBonus()
    }
    
    func performanceReview() -> String {
        return "ผลการประเมิน \(fullName): ดี"
    }
    
    func describe() {
        print("พนักงาน: \(fullName) (ID: \(employeeID))")
        print("  แผนก: \(department.rawValue)")
        print("  เงินเดือน: \(String(format: "%.2f", baseSalary)) บาท")
        print("  โบนัส: \(String(format: "%.2f", calculateBonus())) บาท")
        print("  ค่าตอบแทนรวม: \(String(format: "%.2f", calculateTotalCompensation())) บาท")
    }
}

class Manager: Employee {
    var teamMembers: [Employee] = []
    var managementBonus: Double
    
    init(firstName: String, lastName: String, department: Department,
         baseSalary: Double, managementBonus: Double) {
        self.managementBonus = managementBonus
        super.init(firstName: firstName, lastName: lastName,
                   department: department, baseSalary: baseSalary)
    }
    
    override func calculateBonus() -> Double {
        let performanceBonus = super.calculateBonus()
        let teamSizeBonus = Double(teamMembers.count) * 2000
        return performanceBonus + managementBonus + teamSizeBonus
    }
    
    func addTeamMember(_ employee: Employee) {
        teamMembers.append(employee)
        print("เพิ่ม \(employee.fullName) เข้าทีมของ \(fullName)")
    }
    
    func listTeam() {
        print("\nทีมของผู้จัดการ \(fullName):")
        for member in teamMembers {
            print("  - \(member.fullName) (\(member.department.rawValue))")
        }
    }
    
    override func performanceReview() -> String {
        return "ผู้จัดการ \(fullName): ยอดเยี่ยม (ดูแล \(teamMembers.count) คน)"
    }
}

class Director: Manager {
    var budget: Double
    var directReportManagers: [Manager] = []
    
    init(firstName: String, lastName: String, department: Department,
         baseSalary: Double, budget: Double) {
        self.budget = budget
        super.init(firstName: firstName, lastName: lastName, department: department,
                   baseSalary: baseSalary, managementBonus: 50000)
    }
    
    override func calculateBonus() -> Double {
        return super.calculateBonus() + budget * 0.001  // 0.1% ของ budget
    }
    
    func addDirectReportManager(_ manager: Manager) {
        directReportManagers.append(manager)
        teamMembers.append(manager)
    }
    
    var totalHeadcount: Int {
        return directReportManagers.reduce(0) { $0 + $1.teamMembers.count + 1 } + 1
    }
    
    override func describe() {
        super.describe()
        print("  งบประมาณ: \(String(format: "%.2f", budget)) บาท")
        print("  จำนวนพนักงานรวม: \(totalHeadcount) คน")
    }
}

// การใช้งาน
let eng1 = Employee(firstName: "สมชาย", lastName: "เก่ง", department: .engineering, baseSalary: 45000)
let eng2 = Employee(firstName: "สมหญิง", lastName: "ดี", department: .engineering, baseSalary: 50000)
let mkt1 = Employee(firstName: "ประสิทธิ์", lastName: "รวย", department: .marketing, baseSalary: 40000)

let mgr = Manager(firstName: "วิชัย", lastName: "เก่งมาก", department: .engineering,
                  baseSalary: 80000, managementBonus: 20000)
mgr.addTeamMember(eng1)
mgr.addTeamMember(eng2)

let director = Director(firstName: "ดร.สมศักดิ์", lastName: "ยิ่งใหญ่", department: .engineering,
                        baseSalary: 150000, budget: 10000000)
director.addDirectReportManager(mgr)

print("=== รายงานพนักงาน ===")
director.describe()
print("\n\(mgr.performanceReview())")
mgr.listTeam()
```

---

## 15.21 แบบฝึกหัด

### แบบฝึกหัดที่ 1: Animal Kingdom

**โจทย์:** สร้าง class hierarchy สำหรับอาณาจักรสัตว์

```swift
// MARK: - Exercise 1 Solution: Animal Kingdom

class LivingOrganism {
    var species: String
    var scientificName: String
    var isEndangered: Bool
    
    init(species: String, scientificName: String, isEndangered: Bool = false) {
        self.species = species
        self.scientificName = scientificName
        self.isEndangered = isEndangered
    }
    
    func describe() {
        print("\(species) (\(scientificName))\(isEndangered ? " - ใกล้สูญพันธุ์" : "")")
    }
}

class AnimalKingdom: LivingOrganism {
    enum FeedingType: String {
        case herbivore = "กินพืช"
        case carnivore = "กินเนื้อ"
        case omnivore = "กินทั้งพืชและเนื้อ"
    }
    
    var feedingType: FeedingType
    var habitat: String
    var averageLifespan: Int
    
    init(species: String, scientificName: String, feedingType: FeedingType,
         habitat: String, averageLifespan: Int, isEndangered: Bool = false) {
        self.feedingType = feedingType
        self.habitat = habitat
        self.averageLifespan = averageLifespan
        super.init(species: species, scientificName: scientificName, isEndangered: isEndangered)
    }
    
    override func describe() {
        super.describe()
        print("  อาหาร: \(feedingType.rawValue)")
        print("  ถิ่นที่อยู่: \(habitat)")
        print("  อายุขัยเฉลี่ย: \(averageLifespan) ปี")
    }
}

class Reptile: AnimalKingdom {
    var isVenomous: Bool
    var scaleType: String
    
    init(species: String, scientificName: String, feedingType: FeedingType,
         habitat: String, lifespan: Int, isVenomous: Bool, scaleType: String) {
        self.isVenomous = isVenomous
        self.scaleType = scaleType
        super.init(species: species, scientificName: scientificName,
                   feedingType: feedingType, habitat: habitat, averageLifespan: lifespan)
    }
    
    override func describe() {
        super.describe()
        print("  มีพิษ: \(isVenomous ? "ใช่" : "ไม่")")
        print("  ชนิดเกล็ด: \(scaleType)")
    }
}

class Python: Reptile {
    var maxLength: Double  // เมตร
    var constrictor: Bool = true
    
    init(species: String, scientificName: String, maxLength: Double) {
        self.maxLength = maxLength
        super.init(species: species, scientificName: scientificName,
                   feedingType: .carnivore, habitat: "ป่าเขตร้อน",
                   lifespan: 25, isVenomous: false, scaleType: "เกล็ดเรียบ")
    }
    
    func constrict() {
        print("\(species) รัดเหยื่อด้วยความยาว \(maxLength) เมตร")
    }
}

let python = Python(species: "งูหลาม", scientificName: "Python reticulatus", maxLength: 7.5)
python.describe()
python.constrict()

print("\nตรวจสอบ type:")
print("python is Python: \(python is Python)")
print("python is Reptile: \(python is Reptile)")
print("python is AnimalKingdom: \(python is AnimalKingdom)")
print("python is LivingOrganism: \(python is LivingOrganism)")
```

### แบบฝึกหัดที่ 2: Library System

```swift
// MARK: - Exercise 2 Solution: Library System

class LibraryItem {
    let itemID: String
    var title: String
    var publishYear: Int
    var isAvailable: Bool = true
    private static var nextID = 1
    
    init(title: String, publishYear: Int) {
        self.itemID = "LIB\(String(format: "%04d", LibraryItem.nextID))"
        LibraryItem.nextID += 1
        self.title = title
        self.publishYear = publishYear
    }
    
    func checkOut(to patron: String) {
        guard isAvailable else {
            print("\(title) ไม่ว่าง")
            return
        }
        isAvailable = false
        print("\(patron) ยืม '\(title)' แล้ว")
    }
    
    func returnItem() {
        isAvailable = true
        print("'\(title)' ถูกคืนแล้ว")
    }
    
    func describe() {
        print("[\(itemID)] \(title) (\(publishYear)) - \(isAvailable ? "ว่าง" : "ถูกยืม")")
    }
}

class Book: LibraryItem {
    var author: String
    var isbn: String
    var pageCount: Int
    var genre: String
    
    init(title: String, author: String, isbn: String, 
         year: Int, pages: Int, genre: String) {
        self.author = author
        self.isbn = isbn
        self.pageCount = pages
        self.genre = genre
        super.init(title: title, publishYear: year)
    }
    
    override func describe() {
        super.describe()
        print("  ประเภท: หนังสือ | ผู้เขียน: \(author) | \(pageCount) หน้า | ประเภท: \(genre)")
    }
}

class Magazine: LibraryItem {
    var publisher: String
    var issueNumber: Int
    var month: Int
    
    init(title: String, publisher: String, issueNumber: Int, 
         year: Int, month: Int) {
        self.publisher = publisher
        self.issueNumber = issueNumber
        self.month = month
        super.init(title: title, publishYear: year)
    }
    
    override func describe() {
        super.describe()
        print("  ประเภท: นิตยสาร | สำนักพิมพ์: \(publisher) | ฉบับที่ \(issueNumber)")
    }
}

class DVD: LibraryItem {
    var director: String
    var durationMinutes: Int
    var rating: String
    
    init(title: String, director: String, year: Int, 
         duration: Int, rating: String) {
        self.director = director
        self.durationMinutes = duration
        self.rating = rating
        super.init(title: title, publishYear: year)
    }
    
    override func describe() {
        super.describe()
        print("  ประเภท: DVD | ผู้กำกับ: \(director) | \(durationMinutes) นาที | เรต: \(rating)")
    }
}

class Library {
    var name: String
    private var items: [LibraryItem] = []
    
    init(name: String) {
        self.name = name
    }
    
    func addItem(_ item: LibraryItem) {
        items.append(item)
    }
    
    func search(keyword: String) -> [LibraryItem] {
        return items.filter { $0.title.lowercased().contains(keyword.lowercased()) }
    }
    
    func availableItems() -> [LibraryItem] {
        return items.filter { $0.isAvailable }
    }
    
    func catalog() {
        print("\n=== รายการ\(name) ===")
        items.forEach { $0.describe(); print() }
    }
    
    func statistics() {
        let books = items.filter { $0 is Book }.count
        let magazines = items.filter { $0 is Magazine }.count
        let dvds = items.filter { $0 is DVD }.count
        let available = items.filter { $0.isAvailable }.count
        
        print("\n=== สถิติห้องสมุด \(name) ===")
        print("หนังสือ: \(books) เล่ม")
        print("นิตยสาร: \(magazines) ฉบับ")
        print("DVD: \(dvds) แผ่น")
        print("ว่าง: \(available)/\(items.count) รายการ")
    }
}

// การใช้งาน
let library = Library(name: "ห้องสมุดสาธารณะ")

library.addItem(Book(title: "Harry Potter", author: "J.K. Rowling", isbn: "978-0439708180", 
                     year: 1997, pages: 309, genre: "Fantasy"))
library.addItem(Book(title: "ชีวิตติดปีก", author: "สมชาย", isbn: "978-9748",
                     year: 2020, pages: 250, genre: "แรงบันดาลใจ"))
library.addItem(Magazine(title: "National Geographic", publisher: "National Geographic Society",
                          issueNumber: 234, year: 2024, month: 3))
library.addItem(DVD(title: "The Dark Knight", director: "Christopher Nolan",
                     year: 2008, duration: 152, rating: "PG-13"))

library.catalog()

// ยืม/คืน
library.items[0].checkOut(to: "คุณสมชาย")
library.statistics()
```

---

## 15.22 สรุป

ในบทนี้เราได้เรียนรู้:

### สิ่งที่ได้เรียนรู้
1. **Inheritance** — การสืบทอด property และ method จาก Superclass
2. **Single Inheritance** — Swift รองรับการสืบทอดจาก Class เดียว
3. **super keyword** — เรียกใช้ method/initializer ของ Superclass
4. **Method Overriding** — เขียน method ใหม่ใน Subclass ด้วย `override`
5. **Property Overriding** — Override computed properties และเพิ่ม observers
6. **Subscript Overriding** — ปรับแต่งการ access ด้วย subscript
7. **final** — ป้องกัน inheritance หรือ overriding
8. **Polymorphism** — Object ของ Subclass ทำงานเป็น Superclass ได้
9. **Dynamic Dispatch** — Swift เลือก method implementation ตอน runtime
10. **Static Dispatch** — ตัดสินใจตอน compile time (final, static)
11. **Downcasting** — `as?` (ปลอดภัย) และ `as!` (อาจ crash)
12. **Type Checking** — `is` operator
13. **Multiple Inheritance Levels** — หลายระดับชั้น
14. **Required Initializers** — บังคับให้ Subclass implement
15. **Class Hierarchy Design** — หลักการออกแบบที่ดี

### ตารางเปรียบเทียบ

| คุณสมบัติ | Class | Struct |
|-----------|-------|--------|
| Inheritance | ✅ | ❌ |
| Polymorphism | ✅ | ❌ |
| Reference Type | ✅ | ❌ (Value Type) |
| Dynamic Dispatch | ✅ | ❌ |
| Deinit | ✅ | ❌ |

### เมื่อใดควรใช้ Inheritance
- เมื่อมีความสัมพันธ์แบบ "is-a" ที่ชัดเจน
- เมื่อต้องการ Polymorphism
- เมื่อต้องการแบ่งปัน implementation จริงๆ
- เมื่อทำงานกับ Framework ที่กำหนด Class hierarchy มาแล้ว

### เมื่อใดควรใช้ Protocol แทน Inheritance
- เมื่อต้องการ "can-do" relationship แทน "is-a"
- เมื่อต้องการความยืดหยุ่นมากขึ้น
- เมื่อต้องการให้ Struct/Enum ใช้ได้ด้วย
- เมื่อต้องการหลีกเลี่ยงปัญหา deep inheritance

### บทต่อไป
ใน Part 16 เราจะเรียนรู้เกี่ยวกับ **Protocols** ซึ่งเป็นหัวใจของ Protocol-Oriented Programming (POP) ใน Swift ที่ช่วยให้โค้ดมีความยืดหยุ่นและนำกลับมาใช้ใหม่ได้มากยิ่งขึ้น

---

*หมายเหตุ: โค้ดทั้งหมดในบทนี้สามารถรันได้ใน Swift Playgrounds หรือ Xcode*
