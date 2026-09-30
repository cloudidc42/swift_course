# Part 12: Enumerations (Enums) ใน Swift

## บทนำ

Enumeration (หรือเรียกสั้นๆ ว่า enum) เป็นชนิดข้อมูลที่ให้คุณกำหนดกลุ่มของค่าที่เกี่ยวข้องกัน Swift enum มีความทรงพลังมากกว่าภาษาอื่น เพราะรองรับ associated values, computed properties, methods และ initializers

---

## 1. Enum Basics (พื้นฐาน Enum)

### 1.1 การสร้าง Enum พื้นฐาน

```swift
// สร้าง enum ง่ายๆ
enum Direction {
    case north
    case south
    case east
    case west
}

// หรือเขียนในบรรทัดเดียว
enum Season {
    case spring, summer, autumn, winter
}

// การใช้งาน
var myDirection = Direction.north
myDirection = .south  // type inference ทำให้ไม่ต้องเขียน Direction.

// ตรวจสอบค่า
if myDirection == .south {
    print("กำลังไปทางใต้")
}

// ใน switch statement
switch myDirection {
case .north:
    print("ไปทางเหนือ")
case .south:
    print("ไปทางใต้")
case .east:
    print("ไปทางตะวันออก")
case .west:
    print("ไปทางตะวันตก")
}
```

### 1.2 Enum เป็น First-class Type

```swift
// Enum สามารถส่งเป็น parameter ได้
func describeDirection(_ direction: Direction) -> String {
    switch direction {
    case .north: return "เหนือ"
    case .south: return "ใต้"
    case .east:  return "ตะวันออก"
    case .west:  return "ตะวันตก"
    }
}

print(describeDirection(.north))  // "เหนือ"

// Enum เป็น return type ได้
func oppositeDirection(_ direction: Direction) -> Direction {
    switch direction {
    case .north: return .south
    case .south: return .north
    case .east:  return .west
    case .west:  return .east
    }
}

print(oppositeDirection(.north))  // .south

// Enum ใน array
let route: [Direction] = [.north, .north, .east, .south]
for step in route {
    print("ก้าวไป \(describeDirection(step))")
}

// Enum กับ optional
var currentSeason: Season? = .summer
if let season = currentSeason {
    print("ฤดูกาลปัจจุบัน: \(season)")
}
```

---

## 2. Enum with Raw Values (Enum พร้อมค่าดิบ)

### 2.1 Raw Values แบบ Int

```swift
// Raw values เริ่มจาก 0 โดย default
enum Planet: Int {
    case mercury = 1   // กำหนดค่าเริ่มต้นที่ 1
    case venus
    case earth
    case mars
    case jupiter
    case saturn
    case uranus
    case neptune
}

// เข้าถึง raw value
print(Planet.earth.rawValue)  // 3
print(Planet.mars.rawValue)   // 4

// สร้างจาก raw value (returns Optional)
if let planet = Planet(rawValue: 3) {
    print("ดาวเคราะห์ที่ 3 คือ \(planet)")  // earth
}

// ถ้า raw value ไม่ตรง จะได้ nil
let unknown = Planet(rawValue: 99)
print(unknown)  // nil

// Auto-increment
enum Month: Int {
    case january = 1, february, march, april, may, june
    case july, august, september, october, november, december
}

print(Month.june.rawValue)     // 6
print(Month.december.rawValue) // 12
```

### 2.2 Raw Values แบบ String

```swift
enum CompassPoint: String {
    case north = "เหนือ"
    case south = "ใต้"
    case east  = "ตะวันออก"
    case west  = "ตะวันตก"
}

print(CompassPoint.north.rawValue)  // "เหนือ"

// เมื่อไม่กำหนดค่า String raw value จะใช้ชื่อ case เป็น default
enum Color: String {
    case red, green, blue
}

print(Color.red.rawValue)    // "red"
print(Color.green.rawValue)  // "green"

// สร้างจาก string raw value
if let color = Color(rawValue: "blue") {
    print("สีที่พบ: \(color)")
}

// ใช้ใน JSON decoding
enum HTTPMethod: String {
    case get    = "GET"
    case post   = "POST"
    case put    = "PUT"
    case delete = "DELETE"
    case patch  = "PATCH"
}

func makeRequest(method: HTTPMethod, url: String) {
    print("\(method.rawValue) \(url)")
}

makeRequest(method: .post, url: "/api/users")  // "POST /api/users"
```

### 2.3 Raw Values แบบ Double

```swift
enum Angle: Double {
    case zero     = 0.0
    case right    = 90.0
    case straight = 180.0
    case full     = 360.0
}

print(Angle.right.rawValue)   // 90.0

// คำนวณด้วย raw value
func toRadians(_ angle: Angle) -> Double {
    return angle.rawValue * .pi / 180.0
}

print(toRadians(.right))  // 1.5707963... (π/2)
```

---

## 3. Enum with Associated Values (Enum พร้อมค่าที่เชื่อมโยง)

### 3.1 Associated Values พื้นฐาน

```swift
// Enum ที่มี associated values
enum Shape {
    case circle(radius: Double)
    case rectangle(width: Double, height: Double)
    case triangle(base: Double, height: Double)
}

// สร้าง enum ด้วย associated values
let circle = Shape.circle(radius: 5.0)
let rect = Shape.rectangle(width: 10.0, height: 5.0)
let triangle = Shape.triangle(base: 6.0, height: 4.0)

// ดึง associated values ด้วย switch
func area(of shape: Shape) -> Double {
    switch shape {
    case .circle(let radius):
        return .pi * radius * radius
    case .rectangle(let width, let height):
        return width * height
    case .triangle(let base, let height):
        return 0.5 * base * height
    }
}

print(area(of: circle))    // 78.53...
print(area(of: rect))      // 50.0
print(area(of: triangle))  // 12.0

// pattern matching กับ if case
if case .circle(let radius) = circle {
    print("วงกลมรัศมี \(radius)")
}

// guard case
func processShape(_ shape: Shape) {
    guard case .rectangle(let w, let h) = shape else {
        print("ไม่ใช่สี่เหลี่ยม")
        return
    }
    print("สี่เหลี่ยม \(w) x \(h)")
}
```

### 3.2 Associated Values ที่ซับซ้อน

```swift
// Associated values หลายชนิด
enum NetworkResponse {
    case success(statusCode: Int, data: Data)
    case failure(statusCode: Int, error: Error)
    case redirect(to: String)
    case timeout
}

// Enum สำหรับ event
enum UserEvent {
    case login(username: String, timestamp: Date)
    case logout(username: String, reason: String?)
    case purchase(itemId: String, amount: Double, currency: String)
    case error(code: Int, message: String)
}

// การใช้งาน
let event = UserEvent.purchase(itemId: "ITEM001", amount: 150.0, currency: "THB")

switch event {
case .login(let username, let timestamp):
    print("เข้าสู่ระบบ: \(username) เวลา \(timestamp)")
    
case .logout(let username, let reason):
    if let reason = reason {
        print("ออกจากระบบ: \(username) เหตุผล: \(reason)")
    } else {
        print("ออกจากระบบ: \(username)")
    }
    
case .purchase(let itemId, let amount, let currency):
    print("ซื้อสินค้า \(itemId) ราคา \(amount) \(currency)")
    
case .error(let code, let message):
    print("ข้อผิดพลาด \(code): \(message)")
}
```

---

## 4. Enum Methods (เมธอดของ Enum)

```swift
enum Weekday {
    case monday, tuesday, wednesday, thursday, friday, saturday, sunday
    
    // Method ใน enum
    var isWeekend: Bool {
        switch self {
        case .saturday, .sunday: return true
        default: return false
        }
    }
    
    var name: String {
        switch self {
        case .monday:    return "วันจันทร์"
        case .tuesday:   return "วันอังคาร"
        case .wednesday: return "วันพุธ"
        case .thursday:  return "วันพฤหัสบดี"
        case .friday:    return "วันศุกร์"
        case .saturday:  return "วันเสาร์"
        case .sunday:    return "วันอาทิตย์"
        }
    }
    
    var shortName: String {
        return String(name.prefix(3))
    }
    
    // Method ที่ return enum
    var next: Weekday {
        switch self {
        case .monday:    return .tuesday
        case .tuesday:   return .wednesday
        case .wednesday: return .thursday
        case .thursday:  return .friday
        case .friday:    return .saturday
        case .saturday:  return .sunday
        case .sunday:    return .monday
        }
    }
    
    func daysUntil(_ target: Weekday) -> Int {
        if self == target { return 0 }
        
        var count = 0
        var current = self.next
        
        while current != target {
            count += 1
            current = current.next
        }
        
        return count + 1
    }
}

// ต้องใช้ == จึงต้อง conform Equatable
extension Weekday: Equatable {}

let today = Weekday.wednesday
print(today.name)         // "วันพุธ"
print(today.isWeekend)    // false
print(today.next.name)    // "วันพฤหัสบดี"
print(today.daysUntil(.saturday))  // 3
```

---

## 5. Enum Computed Properties (Properties ที่คำนวณ)

```swift
enum Temperature {
    case celsius(Double)
    case fahrenheit(Double)
    case kelvin(Double)
    
    // Computed property
    var celsius: Double {
        switch self {
        case .celsius(let c):
            return c
        case .fahrenheit(let f):
            return (f - 32) * 5 / 9
        case .kelvin(let k):
            return k - 273.15
        }
    }
    
    var fahrenheit: Double {
        return celsius * 9 / 5 + 32
    }
    
    var kelvin: Double {
        return celsius + 273.15
    }
    
    var description: String {
        switch self {
        case .celsius(let c):
            return "\(c)°C"
        case .fahrenheit(let f):
            return "\(f)°F"
        case .kelvin(let k):
            return "\(k)K"
        }
    }
    
    var isFreezing: Bool {
        return celsius <= 0
    }
    
    var isBoiling: Bool {
        return celsius >= 100
    }
    
    var category: String {
        switch celsius {
        case ..<0:        return "หนาวจัด"
        case 0..<10:      return "หนาว"
        case 10..<20:     return "เย็น"
        case 20..<30:     return "อบอุ่น"
        case 30..<40:     return "ร้อน"
        default:          return "ร้อนจัด"
        }
    }
}

let bodyTemp = Temperature.celsius(37.0)
print(bodyTemp.description)   // "37.0°C"
print(bodyTemp.fahrenheit)     // 98.6
print(bodyTemp.kelvin)         // 310.15
print(bodyTemp.category)       // "ร้อน"

let boilingPoint = Temperature.fahrenheit(212)
print(boilingPoint.celsius)    // 100.0
print(boilingPoint.isBoiling)  // true
```

---

## 6. Enum Initializers (Constructors)

```swift
enum Coin {
    case satang25
    case satang50
    case baht1
    case baht5
    case baht10
    
    var value: Double {
        switch self {
        case .satang25: return 0.25
        case .satang50: return 0.50
        case .baht1:    return 1.0
        case .baht5:    return 5.0
        case .baht10:   return 10.0
        }
    }
    
    // Custom initializer จาก value
    init?(value: Double) {
        switch value {
        case 0.25: self = .satang25
        case 0.50: self = .satang50
        case 1.0:  self = .baht1
        case 5.0:  self = .baht5
        case 10.0: self = .baht10
        default:   return nil
        }
    }
    
    // Initializer จาก string
    init?(name: String) {
        switch name.lowercased() {
        case "25 สตางค์": self = .satang25
        case "50 สตางค์": self = .satang50
        case "1 บาท":     self = .baht1
        case "5 บาท":     self = .baht5
        case "10 บาท":    self = .baht10
        default:           return nil
        }
    }
}

if let coin = Coin(value: 5.0) {
    print("เหรียญ: \(coin), มูลค่า: \(coin.value)")
}

if let coin = Coin(name: "10 บาท") {
    print("เหรียญ 10 บาท = \(coin.value) บาท")
}

let invalidCoin = Coin(value: 3.0)
print(invalidCoin)  // nil
```

---

## 7. CaseIterable Protocol

```swift
// CaseIterable ให้สามารถวนลูปผ่านทุก case ได้
enum CardSuit: CaseIterable {
    case clubs, diamonds, hearts, spades
    
    var symbol: String {
        switch self {
        case .clubs:    return "♣"
        case .diamonds: return "♦"
        case .hearts:   return "♥"
        case .spades:   return "♠"
        }
    }
    
    var name: String {
        switch self {
        case .clubs:    return "ดอกจิก"
        case .diamonds: return "ข้าวหลามตัด"
        case .hearts:   return "หัวใจ"
        case .spades:   return "โพดำ"
        }
    }
}

// วนลูปผ่านทุก case
for suit in CardSuit.allCases {
    print("\(suit.symbol) = \(suit.name)")
}
// ♣ = ดอกจิก
// ♦ = ข้าวหลามตัด
// ♥ = หัวใจ
// ♠ = โพดำ

print("จำนวนไพ่ suit: \(CardSuit.allCases.count)")  // 4

// CaseIterable กับ Raw Values
enum Planet2: String, CaseIterable {
    case mercury = "ดาวพุธ"
    case venus   = "ดาวศุกร์"
    case earth   = "โลก"
    case mars    = "ดาวอังคาร"
}

let planets = Planet2.allCases.map { $0.rawValue }
print(planets)  // ["ดาวพุธ", "ดาวศุกร์", "โลก", "ดาวอังคาร"]

// Random case
let randomSuit = CardSuit.allCases.randomElement()!
print("สุ่มได้: \(randomSuit.symbol)")

// หา index ของ case
if let index = CardSuit.allCases.firstIndex(of: .hearts) {
    print("hearts อยู่ที่ index: \(index)")  // 2
}
```

---

## 8. Enum ใน Switch Statements

```swift
enum TrafficLight {
    case red
    case yellow  
    case green
    
    var nextLight: TrafficLight {
        switch self {
        case .red:    return .green
        case .yellow: return .red
        case .green:  return .yellow
        }
    }
    
    var duration: Int {  // seconds
        switch self {
        case .red:    return 60
        case .yellow: return 5
        case .green:  return 45
        }
    }
}

// Exhaustive switch - ต้องครบทุก case
let light = TrafficLight.red

switch light {
case .red:
    print("หยุด! (\(light.duration) วินาที)")
case .yellow:
    print("เตรียมพร้อม (\(light.duration) วินาที)")
case .green:
    print("ไปได้! (\(light.duration) วินาที)")
}

// Switch กับ where clause
enum Score {
    case points(Int)
    case grade(String)
}

let myScore = Score.points(85)

switch myScore {
case .points(let p) where p >= 90:
    print("เกรด A")
case .points(let p) where p >= 80:
    print("เกรด B")
case .points(let p) where p >= 70:
    print("เกรด C")
case .points(let p):
    print("ไม่ผ่าน: \(p) คะแนน")
case .grade(let g):
    print("เกรด: \(g)")
}
// "เกรด B"

// Switch กับ multiple patterns
enum Permission {
    case read, write, execute, admin
}

let userPermission = Permission.write

switch userPermission {
case .read, .write:
    print("ผู้ใช้ทั่วไป")
case .execute:
    print("ผู้ใช้ขั้นสูง")
case .admin:
    print("ผู้ดูแลระบบ")
}
```

---

## 9. Nested Enums (Enum ซ้อนกัน)

```swift
// Nested enum
enum Vehicle {
    enum FuelType {
        case gasoline, diesel, electric, hybrid
    }
    
    enum Category {
        case motorcycle
        case car(doors: Int)
        case truck(payload: Double)
        case bus(capacity: Int)
    }
    
    case personal(category: Category, fuel: FuelType)
    case commercial(category: Category, fuel: FuelType, licensePlate: String)
    
    var description: String {
        switch self {
        case .personal(let category, let fuel):
            return "ยานพาหนะส่วนตัว (\(categoryName(category))) ใช้ \(fuelName(fuel))"
        case .commercial(let category, let fuel, let plate):
            return "ยานพาหนะพาณิชย์ (\(categoryName(category))) ทะเบียน \(plate)"
        }
    }
    
    private func categoryName(_ category: Category) -> String {
        switch category {
        case .motorcycle: return "จักรยานยนต์"
        case .car(let doors): return "รถยนต์ \(doors) ประตู"
        case .truck(let payload): return "รถบรรทุก \(payload) ตัน"
        case .bus(let capacity): return "รถบัส \(capacity) ที่นั่ง"
        }
    }
    
    private func fuelName(_ fuel: FuelType) -> String {
        switch fuel {
        case .gasoline: return "น้ำมันเบนซิน"
        case .diesel:   return "น้ำมันดีเซล"
        case .electric: return "ไฟฟ้า"
        case .hybrid:   return "ไฮบริด"
        }
    }
}

let car = Vehicle.personal(category: .car(doors: 4), fuel: .gasoline)
let bus = Vehicle.commercial(category: .bus(capacity: 50), fuel: .diesel, licensePlate: "10-1234")

print(car.description)
print(bus.description)
```

---

## 10. Recursive Enums (Indirect)

```swift
// Recursive enum ต้องใช้ indirect keyword
indirect enum ArithmeticExpression {
    case number(Double)
    case addition(ArithmeticExpression, ArithmeticExpression)
    case subtraction(ArithmeticExpression, ArithmeticExpression)
    case multiplication(ArithmeticExpression, ArithmeticExpression)
    case division(ArithmeticExpression, ArithmeticExpression)
}

func evaluate(_ expression: ArithmeticExpression) -> Double {
    switch expression {
    case .number(let value):
        return value
    case .addition(let left, let right):
        return evaluate(left) + evaluate(right)
    case .subtraction(let left, let right):
        return evaluate(left) - evaluate(right)
    case .multiplication(let left, let right):
        return evaluate(left) * evaluate(right)
    case .division(let left, let right):
        let divisor = evaluate(right)
        guard divisor != 0 else { return 0 }
        return evaluate(left) / divisor
    }
}

// (3 + 5) * 2
let expression = ArithmeticExpression.multiplication(
    .addition(.number(3), .number(5)),
    .number(2)
)

print(evaluate(expression))  // 16.0

// Binary Tree ด้วย recursive enum
indirect enum BinaryTree<T> {
    case empty
    case node(value: T, left: BinaryTree<T>, right: BinaryTree<T>)
    
    var count: Int {
        switch self {
        case .empty: return 0
        case .node(_, let left, let right):
            return 1 + left.count + right.count
        }
    }
    
    func contains(_ value: T) -> Bool where T: Equatable {
        switch self {
        case .empty: return false
        case .node(let nodeValue, let left, let right):
            if nodeValue == value { return true }
            return left.contains(value) || right.contains(value)
        }
    }
    
    func inorder(visit: (T) -> Void) {
        switch self {
        case .empty: break
        case .node(let value, let left, let right):
            left.inorder(visit: visit)
            visit(value)
            right.inorder(visit: visit)
        }
    }
}

let tree = BinaryTree.node(
    value: 5,
    left: .node(value: 3, left: .node(value: 1, left: .empty, right: .empty), right: .empty),
    right: .node(value: 8, left: .empty, right: .node(value: 9, left: .empty, right: .empty))
)

print("จำนวน nodes: \(tree.count)")  // 5
print("มี 3 ใน tree: \(tree.contains(3))")  // true

print("Inorder traversal: ", terminator: "")
tree.inorder { print($0, terminator: " ") }
print()  // 1 3 5 8 9
```

---

## 11. Generic Enums (Enum แบบ Generic)

```swift
// Generic enum - Option type
enum Option<T> {
    case some(T)
    case none
    
    var hasValue: Bool {
        switch self {
        case .some: return true
        case .none: return false
        }
    }
    
    func map<U>(_ transform: (T) -> U) -> Option<U> {
        switch self {
        case .some(let value): return .some(transform(value))
        case .none:            return .none
        }
    }
    
    func flatMap<U>(_ transform: (T) -> Option<U>) -> Option<U> {
        switch self {
        case .some(let value): return transform(value)
        case .none:            return .none
        }
    }
    
    func unwrap(default defaultValue: T) -> T {
        switch self {
        case .some(let value): return value
        case .none:            return defaultValue
        }
    }
}

let name: Option<String> = .some("Swift")
let empty: Option<String> = .none

print(name.hasValue)   // true
print(empty.hasValue)  // false

let uppercased = name.map { $0.uppercased() }
print(uppercased)  // .some("SWIFT")

// Generic Either type
enum Either<Left, Right> {
    case left(Left)
    case right(Right)
    
    var isLeft: Bool {
        if case .left = self { return true }
        return false
    }
    
    var leftValue: Left? {
        if case .left(let value) = self { return value }
        return nil
    }
    
    var rightValue: Right? {
        if case .right(let value) = self { return value }
        return nil
    }
    
    func mapLeft<NewLeft>(_ transform: (Left) -> NewLeft) -> Either<NewLeft, Right> {
        switch self {
        case .left(let value):  return .left(transform(value))
        case .right(let value): return .right(value)
        }
    }
}

let intOrString: Either<Int, String> = .left(42)
print(intOrString.leftValue ?? 0)  // 42
```

---

## 12. Comparable Enums

```swift
// Enum ที่ conform Comparable
enum Priority: Int, Comparable {
    case low = 1
    case medium = 2
    case high = 3
    case critical = 4
    
    static func < (lhs: Priority, rhs: Priority) -> Bool {
        return lhs.rawValue < rhs.rawValue
    }
    
    var label: String {
        switch self {
        case .low:      return "ต่ำ"
        case .medium:   return "ปานกลาง"
        case .high:     return "สูง"
        case .critical: return "วิกฤต"
        }
    }
}

// ใช้งาน Comparable
let p1 = Priority.low
let p2 = Priority.high

print(p1 < p2)   // true
print(p2 > p1)   // true
print(p1 == p1)  // true

// เรียงลำดับ tasks ตาม priority
struct Task {
    let name: String
    let priority: Priority
}

let tasks = [
    Task(name: "อัปเดตเอกสาร", priority: .low),
    Task(name: "แก้ bug วิกฤต", priority: .critical),
    Task(name: "พัฒนาฟีเจอร์ใหม่", priority: .medium),
    Task(name: "ตรวจสอบความปลอดภัย", priority: .high)
]

let sortedTasks = tasks.sorted { $0.priority > $1.priority }
for task in sortedTasks {
    print("[\(task.priority.label)] \(task.name)")
}
// [วิกฤต] แก้ bug วิกฤต
// [สูง] ตรวจสอบความปลอดภัย
// [ปานกลาง] พัฒนาฟีเจอร์ใหม่
// [ต่ำ] อัปเดตเอกสาร

// Comparable ด้วย CaseIterable (โดยไม่ใช้ raw value)
enum Size: CaseIterable, Comparable {
    case small, medium, large, extraLarge
    
    static func < (lhs: Size, rhs: Size) -> Bool {
        let all = Size.allCases
        return all.firstIndex(of: lhs)! < all.firstIndex(of: rhs)!
    }
}

print(Size.small < Size.large)   // true
print(Size.large > Size.medium)  // true
```

---

## 13. Codable Enums

```swift
import Foundation

// Enum ที่ conform Codable
enum Status: String, Codable {
    case active   = "active"
    case inactive = "inactive"
    case pending  = "pending"
    case banned   = "banned"
}

struct User: Codable {
    let id: Int
    let name: String
    let status: Status
}

// Encode
let user = User(id: 1, name: "สมชาย", status: .active)

do {
    let encoder = JSONEncoder()
    encoder.outputFormatting = .prettyPrinted
    let jsonData = try encoder.encode(user)
    let jsonString = String(data: jsonData, encoding: .utf8)!
    print(jsonString)
    // {
    //   "id" : 1,
    //   "name" : "สมชาย",
    //   "status" : "active"
    // }
    
    // Decode
    let decoder = JSONDecoder()
    let decoded = try decoder.decode(User.self, from: jsonData)
    print("Decoded: \(decoded.name) - \(decoded.status)")
    
} catch {
    print("Error: \(error)")
}

// Enum with associated values + Codable (manual)
enum PaymentMethod: Codable {
    case creditCard(number: String, expiry: String)
    case bankTransfer(accountNumber: String, bankCode: String)
    case cash
    
    private enum CodingKeys: String, CodingKey {
        case type, creditCardNumber, expiry, accountNumber, bankCode
    }
    
    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        let type = try container.decode(String.self, forKey: .type)
        
        switch type {
        case "creditCard":
            let number = try container.decode(String.self, forKey: .creditCardNumber)
            let expiry = try container.decode(String.self, forKey: .expiry)
            self = .creditCard(number: number, expiry: expiry)
        case "bankTransfer":
            let account = try container.decode(String.self, forKey: .accountNumber)
            let bank = try container.decode(String.self, forKey: .bankCode)
            self = .bankTransfer(accountNumber: account, bankCode: bank)
        case "cash":
            self = .cash
        default:
            throw DecodingError.dataCorruptedError(forKey: .type, in: container, debugDescription: "Unknown type")
        }
    }
    
    func encode(to encoder: Encoder) throws {
        var container = encoder.container(keyedBy: CodingKeys.self)
        
        switch self {
        case .creditCard(let number, let expiry):
            try container.encode("creditCard", forKey: .type)
            try container.encode(number, forKey: .creditCardNumber)
            try container.encode(expiry, forKey: .expiry)
        case .bankTransfer(let account, let bank):
            try container.encode("bankTransfer", forKey: .type)
            try container.encode(account, forKey: .accountNumber)
            try container.encode(bank, forKey: .bankCode)
        case .cash:
            try container.encode("cash", forKey: .type)
        }
    }
}

// ทดสอบ
let payment = PaymentMethod.creditCard(number: "1234-5678-9012-3456", expiry: "12/26")
let encoder = JSONEncoder()
encoder.outputFormatting = .prettyPrinted

if let data = try? encoder.encode(payment),
   let json = String(data: data, encoding: .utf8) {
    print(json)
}
```

---

## 14. Optional เป็น Enum

```swift
// Optional ใน Swift คือ enum จริงๆ
// enum Optional<Wrapped> {
//     case none
//     case some(Wrapped)
// }

// ดังนั้น Optional สามารถใช้ switch ได้
let optionalValue: Int? = 42

switch optionalValue {
case .none:
    print("ไม่มีค่า")
case .some(let value):
    print("มีค่า: \(value)")
}

// เหมือนกับ:
if let value = optionalValue {
    print("มีค่า: \(value)")
} else {
    print("ไม่มีค่า")
}

// Optional chaining ก็เป็น syntactic sugar
let names: [String]? = ["Alice", "Bob"]
let count = names?.count  // Optional<Int>

// Pattern matching กับ optional
let numbers = [1, 2, nil, 4, nil, 6]
for number in numbers {
    switch number {
    case .some(let n) where n > 3:
        print("ตัวเลขมากกว่า 3: \(n)")
    case .some(let n):
        print("ตัวเลข: \(n)")
    case .none:
        print("ไม่มีค่า")
    }
}

// flatMap กับ Optional
func parseAge(_ str: String) -> Int? {
    return Int(str).flatMap { $0 > 0 ? $0 : nil }
}

print(parseAge("25"))   // Optional(25)
print(parseAge("-5"))   // nil
print(parseAge("abc"))  // nil
```

---

## 15. Result Type (Success/Failure)

```swift
import Foundation

// Result type ใน Swift Standard Library
// enum Result<Success, Failure: Error> {
//     case success(Success)
//     case failure(Failure)
// }

// กำหนด Error enum
enum NetworkError: Error {
    case invalidURL
    case noConnection
    case serverError(statusCode: Int)
    case decodingError(String)
    case timeout
}

extension NetworkError: LocalizedError {
    var errorDescription: String? {
        switch self {
        case .invalidURL:
            return "URL ไม่ถูกต้อง"
        case .noConnection:
            return "ไม่มีการเชื่อมต่ออินเทอร์เน็ต"
        case .serverError(let code):
            return "Server error: \(code)"
        case .decodingError(let detail):
            return "ไม่สามารถอ่านข้อมูล: \(detail)"
        case .timeout:
            return "หมดเวลาการเชื่อมต่อ"
        }
    }
}

// ฟังก์ชันที่ return Result
func fetchUser(id: Int) -> Result<String, NetworkError> {
    guard id > 0 else {
        return .failure(.invalidURL)
    }
    
    if id == 999 {
        return .failure(.serverError(statusCode: 404))
    }
    
    return .success("ผู้ใช้ ID: \(id)")
}

// การใช้งาน Result
let result = fetchUser(id: 42)

switch result {
case .success(let user):
    print("สำเร็จ: \(user)")
case .failure(let error):
    print("ผิดพลาด: \(error.localizedDescription)")
}

// map และ flatMap กับ Result
let processedResult = fetchUser(id: 10)
    .map { user in user.uppercased() }
    .mapError { error in NetworkError.serverError(statusCode: 500) }

// get() method
do {
    let value = try result.get()
    print("ค่า: \(value)")
} catch {
    print("Error: \(error)")
}

// Result ในการทำงานแบบ async simulation
enum DatabaseError: Error {
    case notFound
    case duplicateKey
    case connectionFailed
}

func findRecord(key: String) -> Result<[String: Any], DatabaseError> {
    let database: [String: [String: Any]] = [
        "user1": ["name": "สมชาย", "age": 25],
        "user2": ["name": "สมหญิง", "age": 30]
    ]
    
    guard let record = database[key] else {
        return .failure(.notFound)
    }
    
    return .success(record)
}

// Chain operations ด้วย flatMap
let chainResult = findRecord(key: "user1")
    .flatMap { record -> Result<String, DatabaseError> in
        guard let name = record["name"] as? String else {
            return .failure(.notFound)
        }
        return .success(name)
    }
    .map { name in "ยินดีต้อนรับ, \(name)!" }

switch chainResult {
case .success(let message):
    print(message)  // "ยินดีต้อนรับ, สมชาย!"
case .failure(let error):
    print("Error: \(error)")
}
```

---

## 16. State Machines กับ Enums

```swift
// State machine ด้วย enum
enum TrafficLightState {
    case red(remaining: Int)
    case yellow(remaining: Int)
    case green(remaining: Int)
    
    var next: TrafficLightState {
        switch self {
        case .red:
            return .green(remaining: 45)
        case .yellow:
            return .red(remaining: 60)
        case .green:
            return .yellow(remaining: 5)
        }
    }
    
    var color: String {
        switch self {
        case .red:    return "🔴"
        case .yellow: return "🟡"
        case .green:  return "🟢"
        }
    }
    
    var instruction: String {
        switch self {
        case .red:    return "หยุด"
        case .yellow: return "เตรียมพร้อม"
        case .green:  return "ไปได้"
        }
    }
    
    var remaining: Int {
        switch self {
        case .red(let s), .yellow(let s), .green(let s): return s
        }
    }
}

var light = TrafficLightState.red(remaining: 60)
print("\(light.color) \(light.instruction) (\(light.remaining)s)")

for _ in 0..<3 {
    light = light.next
    print("\(light.color) \(light.instruction) (\(light.remaining)s)")
}

// Order State Machine
enum OrderStatus {
    case pending
    case confirmed(orderId: String)
    case processing(startedAt: Date)
    case shipped(trackingNumber: String, carrier: String)
    case delivered(deliveredAt: Date)
    case cancelled(reason: String)
    
    var canTransitionTo: [OrderStatus.Type_] {
        switch self {
        case .pending: return [.confirmed, .cancelled]
        case .confirmed: return [.processing, .cancelled]
        case .processing: return [.shipped, .cancelled]
        case .shipped: return [.delivered]
        case .delivered: return []
        case .cancelled: return []
        }
    }
    
    enum Type_ {
        case confirmed, cancelled, processing, shipped, delivered
    }
    
    var displayName: String {
        switch self {
        case .pending:              return "รอดำเนินการ"
        case .confirmed(let id):   return "ยืนยันแล้ว (#\(id))"
        case .processing:           return "กำลังดำเนินการ"
        case .shipped(let tracking, let carrier):
            return "จัดส่งแล้ว (\(carrier): \(tracking))"
        case .delivered(let date):  return "ส่งถึงแล้ว"
        case .cancelled(let reason): return "ยกเลิก: \(reason)"
        }
    }
}

// ตัวอย่าง order workflow
var orderStatus = OrderStatus.pending
print("สถานะ: \(orderStatus.displayName)")

orderStatus = .confirmed(orderId: "ORD-2024-001")
print("สถานะ: \(orderStatus.displayName)")

orderStatus = .processing(startedAt: Date())
print("สถานะ: \(orderStatus.displayName)")

orderStatus = .shipped(trackingNumber: "TH123456789", carrier: "ไปรษณีย์ไทย")
print("สถานะ: \(orderStatus.displayName)")

orderStatus = .delivered(deliveredAt: Date())
print("สถานะ: \(orderStatus.displayName)")
```

---

## 17. Enum Best Practices (แนวปฏิบัติที่ดี)

```swift
// 1. ใช้ enum แทน magic constants
// ไม่ดี:
func processStatusCode(_ code: Int) {
    if code == 200 { }
    if code == 404 { }
}

// ดีกว่า:
enum HTTPStatusCode: Int {
    case ok                  = 200
    case created             = 201
    case noContent           = 204
    case badRequest          = 400
    case unauthorized        = 401
    case forbidden           = 403
    case notFound            = 404
    case internalServerError = 500
    
    var isSuccess: Bool { (200...299).contains(rawValue) }
    var isClientError: Bool { (400...499).contains(rawValue) }
    var isServerError: Bool { (500...599).contains(rawValue) }
    
    var description: String {
        switch self {
        case .ok:                  return "OK"
        case .created:             return "Created"
        case .noContent:           return "No Content"
        case .badRequest:          return "Bad Request"
        case .unauthorized:        return "Unauthorized"
        case .forbidden:           return "Forbidden"
        case .notFound:            return "Not Found"
        case .internalServerError: return "Internal Server Error"
        }
    }
}

let statusCode = HTTPStatusCode(rawValue: 404)!
print("\(statusCode.rawValue): \(statusCode.description)")  // "404: Not Found"
print("เป็น error: \(statusCode.isClientError)")  // true

// 2. ใช้ associated values สำหรับ context
// ไม่ดี:
var errorMessage: String = ""
var hasError: Bool = false

// ดีกว่า:
enum AppState {
    case idle
    case loading
    case loaded(data: [String])
    case error(message: String, retryable: Bool)
}

// 3. Namespace ด้วย enum ที่ไม่มี cases
enum Analytics {
    enum Event {
        case pageView(page: String)
        case buttonTap(name: String)
        case purchase(amount: Double)
    }
    
    enum Property {
        static let userId = "user_id"
        static let sessionId = "session_id"
        static let platform = "platform"
    }
}

let event = Analytics.Event.pageView(page: "Home")
let property = Analytics.Property.userId
```

---

## 18. Real-world Examples

### 18.1 Network Status

```swift
import Foundation

// Network Status enum
enum NetworkStatus {
    case connected(type: ConnectionType)
    case disconnected
    case restricted
    case unknown
    
    enum ConnectionType {
        case wifi
        case cellular(speed: CellularSpeed)
        case ethernet
        case vpn
        
        enum CellularSpeed {
            case slow2G
            case medium3G
            case fast4G
            case ultraFast5G
        }
    }
    
    var isAvailable: Bool {
        if case .connected = self { return true }
        return false
    }
    
    var description: String {
        switch self {
        case .connected(let type):
            return "เชื่อมต่อผ่าน \(connectionTypeName(type))"
        case .disconnected:
            return "ไม่ได้เชื่อมต่อ"
        case .restricted:
            return "การเชื่อมต่อถูกจำกัด"
        case .unknown:
            return "ไม่ทราบสถานะ"
        }
    }
    
    private func connectionTypeName(_ type: ConnectionType) -> String {
        switch type {
        case .wifi:           return "Wi-Fi"
        case .ethernet:       return "Ethernet"
        case .vpn:            return "VPN"
        case .cellular(let speed):
            switch speed {
            case .slow2G:      return "2G"
            case .medium3G:    return "3G"
            case .fast4G:      return "4G"
            case .ultraFast5G: return "5G"
            }
        }
    }
}

// ทดสอบ
let status = NetworkStatus.connected(type: .cellular(speed: .fast4G))
print(status.description)  // "เชื่อมต่อผ่าน 4G"
print(status.isAvailable)  // true

let noNet = NetworkStatus.disconnected
print(noNet.description)   // "ไม่ได้เชื่อมต่อ"
```

### 18.2 Payment Methods

```swift
// Payment Methods สำหรับ e-commerce
enum PaymentMethod2 {
    case creditCard(CardInfo)
    case debitCard(CardInfo)
    case bankTransfer(BankInfo)
    case qrCode(provider: QRProvider)
    case cash
    case cryptocurrency(CryptoInfo)
    
    struct CardInfo {
        let number: String
        let holderName: String
        let expiryMonth: Int
        let expiryYear: Int
        let cvv: String
        
        var maskedNumber: String {
            let last4 = String(number.suffix(4))
            return "**** **** **** \(last4)"
        }
        
        var isExpired: Bool {
            let calendar = Calendar.current
            let now = Date()
            let year = calendar.component(.year, from: now)
            let month = calendar.component(.month, from: now)
            return expiryYear < year || (expiryYear == year && expiryMonth < month)
        }
    }
    
    struct BankInfo {
        let bankCode: String
        let accountNumber: String
        let accountName: String
    }
    
    enum QRProvider: String {
        case promptPay = "PromptPay"
        case trueMoney = "TrueMoney"
        case linePay   = "LINE Pay"
        case kPlus     = "K PLUS"
    }
    
    struct CryptoInfo {
        let currency: String  // BTC, ETH, USDT
        let address: String
        let network: String   // mainnet, testnet
    }
    
    var displayName: String {
        switch self {
        case .creditCard(let info):
            return "บัตรเครดิต \(info.maskedNumber)"
        case .debitCard(let info):
            return "บัตรเดบิต \(info.maskedNumber)"
        case .bankTransfer(let info):
            return "โอนเงิน \(info.bankCode) ****\(String(info.accountNumber.suffix(4)))"
        case .qrCode(let provider):
            return "QR Code (\(provider.rawValue))"
        case .cash:
            return "เงินสด"
        case .cryptocurrency(let info):
            return "\(info.currency) (\(info.network))"
        }
    }
    
    var processingFee: Double {
        switch self {
        case .creditCard:  return 0.025  // 2.5%
        case .debitCard:   return 0.01   // 1%
        case .bankTransfer: return 0.0   // ฟรี
        case .qrCode:      return 0.0   // ฟรี
        case .cash:        return 0.0   // ฟรี
        case .cryptocurrency: return 0.005  // 0.5%
        }
    }
    
    var requiresOTP: Bool {
        switch self {
        case .creditCard, .debitCard, .bankTransfer: return true
        default: return false
        }
    }
}

// ทดสอบ
let creditCard = PaymentMethod2.creditCard(
    PaymentMethod2.CardInfo(
        number: "4111111111111111",
        holderName: "SOMCHAI JAIDEE",
        expiryMonth: 12,
        expiryYear: 2027,
        cvv: "123"
    )
)

let qr = PaymentMethod2.qrCode(provider: .promptPay)

let methods: [PaymentMethod2] = [creditCard, qr, .cash]
for method in methods {
    print("\(method.displayName) - ค่าธรรมเนียม: \(method.processingFee * 100)%")
}
```

### 18.3 Directions / Navigation

```swift
// Navigation enum
enum NavigationAction {
    case move(Direction2, distance: Double)
    case turn(Rotation)
    case stop
    case wait(seconds: Double)
    case landmark(name: String, action: LandmarkAction)
    
    enum Direction2 {
        case forward
        case backward
        case north
        case south
        case east
        case west
        case northeast
        case northwest
        case southeast
        case southwest
    }
    
    enum Rotation {
        case left(degrees: Double)
        case right(degrees: Double)
        case uturn
    }
    
    enum LandmarkAction {
        case turnLeft
        case turnRight
        case continueThrough
        case stop
    }
    
    var instruction: String {
        switch self {
        case .move(let dir, let distance):
            return "เดินไป \(directionName(dir)) \(distance) เมตร"
        case .turn(let rotation):
            return rotationInstruction(rotation)
        case .stop:
            return "หยุด"
        case .wait(let seconds):
            return "รอ \(Int(seconds)) วินาที"
        case .landmark(let name, let action):
            return "ที่ \(name): \(landmarkInstruction(action))"
        }
    }
    
    private func directionName(_ dir: Direction2) -> String {
        switch dir {
        case .forward:    return "ตรงไป"
        case .backward:   return "ถอยหลัง"
        case .north:      return "เหนือ"
        case .south:      return "ใต้"
        case .east:       return "ตะวันออก"
        case .west:       return "ตะวันตก"
        case .northeast:  return "ตะวันออกเฉียงเหนือ"
        case .northwest:  return "ตะวันตกเฉียงเหนือ"
        case .southeast:  return "ตะวันออกเฉียงใต้"
        case .southwest:  return "ตะวันตกเฉียงใต้"
        }
    }
    
    private func rotationInstruction(_ rotation: Rotation) -> String {
        switch rotation {
        case .left(let degrees):  return "เลี้ยวซ้าย \(Int(degrees))°"
        case .right(let degrees): return "เลี้ยวขวา \(Int(degrees))°"
        case .uturn:              return "กลับรถ"
        }
    }
    
    private func landmarkInstruction(_ action: LandmarkAction) -> String {
        switch action {
        case .turnLeft:        return "เลี้ยวซ้าย"
        case .turnRight:       return "เลี้ยวขวา"
        case .continueThrough: return "เดินตรงต่อไป"
        case .stop:            return "หยุด"
        }
    }
}

// แผนการนำทาง
let navigationPlan: [NavigationAction] = [
    .move(.forward, distance: 100),
    .turn(.right(degrees: 90)),
    .move(.forward, distance: 50),
    .landmark(name: "ตึก A", action: .turnLeft),
    .move(.forward, distance: 200),
    .stop
]

print("คำแนะนำการนำทาง:")
for (index, action) in navigationPlan.enumerated() {
    print("\(index + 1). \(action.instruction)")
}
```

---

## 19. Practical Exercises (แบบฝึกหัดพร้อมเฉลย)

### Exercise 1: Card Game

```swift
// โจทย์: สร้าง enum สำหรับเกมไพ่

enum CardValue: Int, CaseIterable, Comparable {
    case two = 2
    case three, four, five, six, seven, eight, nine, ten
    case jack = 11
    case queen = 12
    case king = 13
    case ace = 14
    
    static func < (lhs: CardValue, rhs: CardValue) -> Bool {
        return lhs.rawValue < rhs.rawValue
    }
    
    var name: String {
        switch self {
        case .jack:  return "J"
        case .queen: return "Q"
        case .king:  return "K"
        case .ace:   return "A"
        default:     return "\(rawValue)"
        }
    }
}

enum Suit: String, CaseIterable {
    case clubs = "♣"
    case diamonds = "♦"
    case hearts = "♥"
    case spades = "♠"
}

struct Card: CustomStringConvertible {
    let value: CardValue
    let suit: Suit
    
    var description: String {
        return "\(value.name)\(suit.rawValue)"
    }
}

// สร้างสำรับไพ่
func createDeck() -> [Card] {
    var deck: [Card] = []
    for suit in Suit.allCases {
        for value in CardValue.allCases {
            deck.append(Card(value: value, suit: suit))
        }
    }
    return deck.shuffled()
}

let deck = createDeck()
print("จำนวนไพ่ทั้งหมด: \(deck.count)")  // 52

let hand = Array(deck.prefix(5))
print("ไพ่ในมือ: \(hand.map(\.description).joined(separator: ", "))")

// หา highest card
let highest = hand.max { $0.value < $1.value }!
print("ไพ่สูงสุด: \(highest)")
```

### Exercise 2: File System

```swift
// โจทย์: จำลอง file system ด้วย enum

indirect enum FileSystem {
    case file(name: String, size: Int)
    case directory(name: String, contents: [FileSystem])
    
    var name: String {
        switch self {
        case .file(let name, _):       return name
        case .directory(let name, _): return name
        }
    }
    
    var totalSize: Int {
        switch self {
        case .file(_, let size): return size
        case .directory(_, let contents):
            return contents.reduce(0) { $0 + $1.totalSize }
        }
    }
    
    var isDirectory: Bool {
        if case .directory = self { return true }
        return false
    }
    
    func find(name: String) -> [FileSystem] {
        switch self {
        case .file(let fileName, _):
            return fileName == name ? [self] : []
        case .directory(_, let contents):
            var results: [FileSystem] = []
            if self.name == name { results.append(self) }
            for item in contents {
                results.append(contentsOf: item.find(name: name))
            }
            return results
        }
    }
    
    func printTree(indent: Int = 0) {
        let prefix = String(repeating: "  ", count: indent)
        switch self {
        case .file(let name, let size):
            print("\(prefix)📄 \(name) (\(size) bytes)")
        case .directory(let name, let contents):
            print("\(prefix)📁 \(name)/")
            for item in contents {
                item.printTree(indent: indent + 1)
            }
        }
    }
}

// ทดสอบ
let root = FileSystem.directory(name: "root", contents: [
    .directory(name: "Documents", contents: [
        .file(name: "resume.pdf", size: 1024),
        .file(name: "cover_letter.docx", size: 512)
    ]),
    .directory(name: "Pictures", contents: [
        .file(name: "photo1.jpg", size: 2048),
        .file(name: "photo2.jpg", size: 3072)
    ]),
    .file(name: "README.md", size: 256)
])

root.printTree()
print("\nขนาดรวม: \(root.totalSize) bytes")

let found = root.find(name: "photo1.jpg")
print("ค้นหา photo1.jpg: พบ \(found.count) ไฟล์")
```

### Exercise 3: API Response Handler

```swift
// โจทย์: สร้าง enum สำหรับจัดการ API responses

enum APIResponse<T> {
    case success(T)
    case empty
    case error(APIError)
    case loading
    
    enum APIError: Error {
        case networkError(String)
        case serverError(Int)
        case parseError(String)
        case unauthorized
        case rateLimited(retryAfter: Int)
    }
    
    var isLoading: Bool {
        if case .loading = self { return true }
        return false
    }
    
    var data: T? {
        if case .success(let data) = self { return data }
        return nil
    }
    
    var error: APIError? {
        if case .error(let err) = self { return err }
        return nil
    }
    
    func map<U>(_ transform: (T) -> U) -> APIResponse<U> {
        switch self {
        case .success(let data):   return .success(transform(data))
        case .empty:               return .empty
        case .error(let err):      return .error(err)
        case .loading:             return .loading
        }
    }
    
    var userFriendlyMessage: String {
        switch self {
        case .success:     return "โหลดข้อมูลสำเร็จ"
        case .empty:       return "ไม่มีข้อมูล"
        case .loading:     return "กำลังโหลด..."
        case .error(let err):
            switch err {
            case .networkError(let msg):
                return "เครือข่ายขัดข้อง: \(msg)"
            case .serverError(let code):
                return "เซิร์ฟเวอร์มีปัญหา (รหัส: \(code))"
            case .parseError(let detail):
                return "ข้อมูลผิดรูปแบบ: \(detail)"
            case .unauthorized:
                return "กรุณาเข้าสู่ระบบก่อน"
            case .rateLimited(let seconds):
                return "ส่งคำขอมากเกินไป กรุณารออีก \(seconds) วินาที"
            }
        }
    }
}

// ทดสอบ
struct Product {
    let id: Int
    let name: String
    let price: Double
}

func loadProducts() -> APIResponse<[Product]> {
    // จำลองการโหลด
    let products = [
        Product(id: 1, name: "iPhone 15", price: 35000),
        Product(id: 2, name: "MacBook Air", price: 45000)
    ]
    return .success(products)
}

let response = loadProducts()
print(response.userFriendlyMessage)

if let products = response.data {
    for product in products {
        print("- \(product.name): ฿\(product.price)")
    }
}

// Test error case
let errorResponse = APIResponse<[Product]>.error(.rateLimited(retryAfter: 60))
print(errorResponse.userFriendlyMessage)
```

---

## 20. สรุป (Summary)

Enumerations เป็นหนึ่งในฟีเจอร์ที่ทรงพลังที่สุดของ Swift ในบทนี้เราได้เรียนรู้:

### หัวข้อที่ครอบคลุม

1. **Enum Basics**: การสร้างและใช้งาน enum พื้นฐาน
2. **Raw Values**: Int, String, Double raw values พร้อม auto-increment
3. **Associated Values**: การเก็บข้อมูลเพิ่มเติมใน enum case
4. **Methods**: การเพิ่ม method ใน enum
5. **Computed Properties**: การเพิ่ม property ที่คำนวณค่า
6. **Initializers**: Custom init พร้อม failable initializer
7. **CaseIterable**: การวนลูปผ่านทุก case
8. **Switch Statements**: Exhaustive matching, where clauses
9. **Nested Enums**: Enum ภายใน Enum
10. **Recursive Enums**: `indirect` keyword สำหรับโครงสร้างแบบ recursive
11. **Generic Enums**: Type parameter ใน enum
12. **Comparable Enums**: การเปรียบเทียบและเรียงลำดับ
13. **Codable Enums**: JSON encoding/decoding
14. **Optional as Enum**: ความสัมพันธ์ระหว่าง Optional และ enum
15. **Result Type**: Success/Failure pattern
16. **State Machines**: การจำลอง state machine ด้วย enum
17. **Best Practices**: Namespace, magic constants, context

### Key Takeaways

```swift
// 1. Enum ทำให้ type-safe และ readable
enum Status { case active, inactive, pending }
// แทน: let status: Int = 1  // 1 หมายถึงอะไร?

// 2. Associated values ให้ context ที่จำเป็น
enum Result2 {
    case success(data: Data)
    case failure(error: Error, code: Int)
}

// 3. Switch กับ enum ต้อง exhaustive
func handle(_ status: Status) {
    switch status {
    case .active:   print("ใช้งาน")
    case .inactive: print("ไม่ใช้งาน")
    case .pending:  print("รอดำเนินการ")
    // ไม่ต้อง default เพราะ exhaustive แล้ว
    }
}

// 4. ใช้ enum สำหรับ namespace
enum Config {
    enum API {
        static let baseURL = "https://api.example.com"
        static let version = "v1"
    }
}

// 5. Result type สำหรับ error handling
func divide(_ a: Double, _ b: Double) -> Result<Double, DivisionError> {
    guard b != 0 else { return .failure(.divisionByZero) }
    return .success(a / b)
}

enum DivisionError: Error { case divisionByZero }
```

### Best Practices สรุป

- ใช้ enum แทน `Bool` flags ที่มีหลาย states
- ใช้ associated values เมื่อต้องการเก็บ context
- ใช้ `CaseIterable` เมื่อต้องการวนลูปหรือสร้าง UI จาก cases
- ใช้ `Result` สำหรับ operations ที่อาจล้มเหลว
- ออกแบบ enum ให้ exhaustive เพื่อให้ compiler ช่วย catch errors
- ใช้ nested enum เพื่อจัดกลุ่ม related cases
- ใช้ `indirect` เมื่อ enum ต้องการ recursive structure
- เพิ่ม `CustomStringConvertible` สำหรับ debug-friendly descriptions

---

*จบ Part 12: Enumerations*

> **ต่อไป**: Part 13 จะอธิบาย Structures และ Classes ซึ่งเป็นพื้นฐานสำคัญของ Swift OOP
