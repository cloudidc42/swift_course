# ส่วนที่ 17: Extensions (การขยายความสามารถของ Type)

## บทนำ

Extensions เป็นหนึ่งในฟีเจอร์ที่ทรงพลังที่สุดของ Swift ที่ช่วยให้เราสามารถเพิ่มความสามารถใหม่ให้กับ type ที่มีอยู่แล้วได้ โดยไม่ต้องแก้ไข source code เดิม ไม่ว่าจะเป็น type ที่เราสร้างเอง หรือ type จาก Standard Library หรือแม้แต่ type จาก framework ของ Apple

---

## 17.1 Extensions คืออะไร?

Extension คือการเพิ่ม functionality ใหม่ให้กับ type ที่มีอยู่แล้ว ซึ่งได้แก่:
- `class`
- `struct`  
- `enum`
- `protocol`

แนวคิดนี้คล้ายกับ "categories" ใน Objective-C แต่มีความสามารถมากกว่า

### สิ่งที่ Extensions สามารถทำได้:

1. เพิ่ม computed instance properties และ computed type properties
2. เพิ่ม instance methods และ type methods
3. เพิ่ม initializers ใหม่
4. เพิ่ม subscripts
5. เพิ่ม nested types ใหม่
6. ทำให้ type ที่มีอยู่แล้ว conform to protocol

### สิ่งที่ Extensions ไม่สามารถทำได้:

- เพิ่ม stored properties
- Override ของที่มีอยู่แล้ว (ทำได้เฉพาะใน subclass)

---

## 17.2 Syntax ของ Extension

รูปแบบพื้นฐานของ extension:

```swift
extension TypeName {
    // เพิ่มความสามารถใหม่ที่นี่
}
```

ตัวอย่างง่ายๆ:

```swift
extension Int {
    func isEven() -> Bool {
        return self % 2 == 0
    }
}

let number = 4
print(number.isEven())  // true

let oddNumber = 7
print(oddNumber.isEven())  // false
```

---

## 17.3 การเพิ่ม Computed Properties ให้กับ Type ที่มีอยู่

เราสามารถเพิ่ม computed properties ได้ แต่ไม่สามารถเพิ่ม stored properties ได้

### ตัวอย่าง: Extension บน Double

```swift
extension Double {
    var km: Double { return self * 1_000.0 }
    var m: Double { return self }
    var cm: Double { return self / 100.0 }
    var mm: Double { return self / 1_000.0 }
    var ft: Double { return self / 3.28084 }
}

let oneInch = 25.4.mm
print("หนึ่งนิ้ว = \(oneInch) เมตร")
// หนึ่งนิ้ว = 0.0254 เมตร

let threeFeet = 3.ft
print("สามฟุต = \(threeFeet) เมตร")
// สามฟุต = 0.914399970739201 เมตร

let aMarathon = 42.km + 195.m
print("ระยะมาราธอน = \(aMarathon) เมตร")
// ระยะมาราธอน = 42195.0 เมตร
```

### ตัวอย่าง: Extension บน Int

```swift
extension Int {
    var squared: Int { return self * self }
    var cubed: Int { return self * self * self }
    var isPositive: Bool { return self > 0 }
    var isNegative: Bool { return self < 0 }
    var isZero: Bool { return self == 0 }
    
    var asDouble: Double { return Double(self) }
    var asString: String { return String(self) }
}

let x = 5
print(x.squared)    // 25
print(x.cubed)      // 125
print(x.isPositive) // true

let y = -3
print(y.isNegative) // true
```

### ตัวอย่าง: Extension บน String

```swift
extension String {
    var wordCount: Int {
        let words = self.split(separator: " ")
        return words.count
    }
    
    var isEmailFormat: Bool {
        return self.contains("@") && self.contains(".")
    }
    
    var reversed: String {
        return String(self.reversed())
    }
    
    var isPalindrome: Bool {
        let cleaned = self.lowercased().filter { $0.isLetter }
        return cleaned == String(cleaned.reversed())
    }
}

let text = "Hello World Swift Programming"
print(text.wordCount)  // 4

let email = "user@example.com"
print(email.isEmailFormat)  // true

let word = "racecar"
print(word.isPalindrome)  // true

let sentence = "A man a plan a canal Panama"
// ต้อง clean ก่อน
print("racecar".isPalindrome)  // true
```

---

## 17.4 การเพิ่ม Methods ให้กับ Type ที่มีอยู่

### Instance Methods

```swift
extension Int {
    func repetitions(task: () -> Void) {
        for _ in 0..<self {
            task()
        }
    }
}

3.repetitions {
    print("สวัสดี!")
}
// สวัสดี!
// สวัสดี!
// สวัสดี!
```

### Mutating Methods บน Value Types

เมื่อเพิ่ม method ให้กับ struct หรือ enum และต้องการแก้ไข `self` ต้องใช้ `mutating`:

```swift
extension Int {
    mutating func square() {
        self = self * self
    }
    
    mutating func double() {
        self = self * 2
    }
}

var someInt = 3
someInt.square()
print(someInt)  // 9

var anotherInt = 5
anotherInt.double()
print(anotherInt)  // 10
```

### Type Methods

```swift
extension Int {
    static func random(in range: ClosedRange<Int>) -> Int {
        return Int.random(in: range)
    }
    
    static func factorial(_ n: Int) -> Int {
        guard n > 0 else { return 1 }
        return n * factorial(n - 1)
    }
}

let randomNumber = Int.random(in: 1...100)
print("ตัวเลขสุ่ม: \(randomNumber)")

print(Int.factorial(5))  // 120
print(Int.factorial(10)) // 3628800
```

### Extension บน Array

```swift
extension Array {
    func safeIndex(_ index: Int) -> Element? {
        guard index >= 0 && index < count else { return nil }
        return self[index]
    }
    
    var second: Element? {
        return safeIndex(1)
    }
    
    var last: Element? {
        return safeIndex(count - 1)
    }
}

let fruits = ["แอปเปิ้ล", "กล้วย", "ส้ม"]
print(fruits.safeIndex(1) ?? "ไม่มี")   // กล้วย
print(fruits.safeIndex(10) ?? "ไม่มี")  // ไม่มี
print(fruits.second ?? "ไม่มี")         // กล้วย
```

---

## 17.5 การเพิ่ม Initializers

Extensions สามารถเพิ่ม initializers ใหม่ให้กับ type ได้ แต่สำหรับ class ไม่สามารถเพิ่ม designated initializer หรือ deinitializer ได้ (ทำได้เฉพาะ convenience initializer)

### ตัวอย่าง: Initializer บน Struct

```swift
struct Point {
    var x: Double
    var y: Double
}

struct Size {
    var width: Double
    var height: Double
}

struct Rect {
    var origin: Point
    var size: Size
}

extension Rect {
    // Convenience initializer
    init(center: Point, size: Size) {
        let originX = center.x - (size.width / 2)
        let originY = center.y - (size.height / 2)
        self.init(origin: Point(x: originX, y: originY), size: size)
    }
    
    init(x: Double, y: Double, width: Double, height: Double) {
        self.init(
            origin: Point(x: x, y: y),
            size: Size(width: width, height: height)
        )
    }
}

let centerRect = Rect(
    center: Point(x: 4.0, y: 4.0),
    size: Size(width: 3.0, height: 3.0)
)
print("Origin: (\(centerRect.origin.x), \(centerRect.origin.y))")
// Origin: (2.5, 2.5)
```

### ตัวอย่าง: Initializer บน Class

```swift
class UIColor {
    var red: Double
    var green: Double
    var blue: Double
    var alpha: Double
    
    init(red: Double, green: Double, blue: Double, alpha: Double) {
        self.red = red
        self.green = green
        self.blue = blue
        self.alpha = alpha
    }
}

extension UIColor {
    // Convenience initializer เท่านั้นใน extension
    convenience init(hex: String) {
        var hexSanitized = hex.trimmingCharacters(in: .whitespacesAndNewlines)
        hexSanitized = hexSanitized.replacingOccurrences(of: "#", with: "")
        
        var rgb: UInt64 = 0
        Scanner(string: hexSanitized).scanHexInt64(&rgb)
        
        let r = Double((rgb & 0xFF0000) >> 16) / 255.0
        let g = Double((rgb & 0x00FF00) >> 8) / 255.0
        let b = Double(rgb & 0x0000FF) / 255.0
        
        self.init(red: r, green: g, blue: b, alpha: 1.0)
    }
    
    convenience init(gray: Double) {
        self.init(red: gray, green: gray, blue: gray, alpha: 1.0)
    }
}

let redColor = UIColor(hex: "FF0000")
print("Red: \(redColor.red), Green: \(redColor.green), Blue: \(redColor.blue)")

let grayColor = UIColor(gray: 0.5)
print("Gray: \(grayColor.red), \(grayColor.green), \(grayColor.blue)")
```

---

## 17.6 การเพิ่ม Subscripts

Extensions สามารถเพิ่ม subscripts ใหม่ได้:

```swift
extension Int {
    subscript(digitIndex: Int) -> Int {
        var decimalBase = 1
        for _ in 0..<digitIndex {
            decimalBase *= 10
        }
        return (self / decimalBase) % 10
    }
}

let number = 746381295
print(number[0])  // 5 (หลักหน่วย)
print(number[1])  // 9 (หลักสิบ)
print(number[2])  // 2 (หลักร้อย)
print(number[8])  // 7 (หลักร้อยล้าน)
```

### Subscript บน String

```swift
extension String {
    subscript(i: Int) -> Character? {
        guard i >= 0 && i < count else { return nil }
        return self[index(startIndex, offsetBy: i)]
    }
    
    subscript(range: Range<Int>) -> String {
        let start = index(startIndex, offsetBy: max(0, range.lowerBound))
        let end = index(startIndex, offsetBy: min(count, range.upperBound))
        return String(self[start..<end])
    }
}

let greeting = "สวัสดีชาวโลก"
print(greeting[0] ?? " ")   // ส
print(greeting[1...6])       // วัสดีชา (ขึ้นอยู่กับ encoding)

let ascii = "Hello, World!"
print(ascii[0] ?? " ")  // H
print(ascii[7..<12])    // World
```

---

## 17.7 การเพิ่ม Nested Types

```swift
extension Int {
    enum Kind {
        case negative
        case zero
        case positive
    }
    
    var kind: Kind {
        switch self {
        case 0:
            return .zero
        case let x where x < 0:
            return .negative
        default:
            return .positive
        }
    }
}

print((-5).kind)  // negative
print(0.kind)     // zero
print(7.kind)     // positive

func printIntegerKinds(_ numbers: [Int]) {
    for number in numbers {
        switch number.kind {
        case .negative:
            print("-", terminator: " ")
        case .zero:
            print("0", terminator: " ")
        case .positive:
            print("+", terminator: " ")
        }
    }
    print("")
}

printIntegerKinds([3, 19, -27, 0, -6, 0, 7])
// + + - 0 - 0 +
```

---

## 17.8 การทำให้ Type Conform to Protocol

นี่เป็นหนึ่งในการใช้งาน extensions ที่ทรงพลังที่สุด:

```swift
protocol Describable {
    var description: String { get }
}

struct Person {
    var name: String
    var age: Int
}

// ทำให้ Person conform to Describable ผ่าน extension
extension Person: Describable {
    var description: String {
        return "\(name) อายุ \(age) ปี"
    }
}

let person = Person(name: "สมชาย", age: 30)
print(person.description)  // สมชาย อายุ 30 ปี
```

### Protocol Conformance ที่ซับซ้อนมากขึ้น

```swift
protocol Printable {
    func printInfo()
}

protocol Serializable {
    func toJSON() -> String
}

struct Product {
    var name: String
    var price: Double
    var quantity: Int
}

extension Product: Printable {
    func printInfo() {
        print("สินค้า: \(name)")
        print("ราคา: \(price) บาท")
        print("จำนวน: \(quantity) ชิ้น")
    }
}

extension Product: Serializable {
    func toJSON() -> String {
        return """
        {
            "name": "\(name)",
            "price": \(price),
            "quantity": \(quantity)
        }
        """
    }
}

let product = Product(name: "iPhone", price: 35000, quantity: 5)
product.printInfo()
print(product.toJSON())
```

---

## 17.9 Protocol Extensions

Protocol extensions ช่วยให้เราสามารถ provide default implementations ให้กับ protocol methods ได้:

```swift
protocol Greetable {
    var name: String { get }
    func greet() -> String
    func greetFormal() -> String
}

// Default implementation ผ่าน protocol extension
extension Greetable {
    func greet() -> String {
        return "สวัสดี \(name)!"
    }
    
    func greetFormal() -> String {
        return "เรียน คุณ\(name) ที่นับถือ"
    }
}

struct Thai: Greetable {
    var name: String
    // ไม่ต้อง implement greet() เพราะมี default
}

struct English: Greetable {
    var name: String
    
    // Override default implementation
    func greet() -> String {
        return "Hello, \(name)!"
    }
}

let thai = Thai(name: "สมหมาย")
print(thai.greet())        // สวัสดี สมหมาย!
print(thai.greetFormal())  // เรียน คุณสมหมาย ที่นับถือ

let eng = English(name: "John")
print(eng.greet())        // Hello, John!
print(eng.greetFormal())  // เรียน คุณJohn ที่นับถือ
```

### Protocol Extension ที่มี Constraints

```swift
protocol Container {
    associatedtype Element
    var items: [Element] { get }
    var count: Int { get }
}

extension Container {
    var isEmpty: Bool {
        return count == 0
    }
    
    var first: Element? {
        return items.first
    }
    
    var last: Element? {
        return items.last
    }
}

// เพิ่ม method เฉพาะเมื่อ Element เป็น Equatable
extension Container where Element: Equatable {
    func contains(_ element: Element) -> Bool {
        return items.contains(element)
    }
    
    func firstIndex(of element: Element) -> Int? {
        return items.firstIndex(of: element)
    }
}

struct NumberBox: Container {
    var items: [Int]
    var count: Int { return items.count }
}

let box = NumberBox(items: [1, 2, 3, 4, 5])
print(box.isEmpty)           // false
print(box.first ?? 0)        // 1
print(box.contains(3))       // true
print(box.firstIndex(of: 4) ?? -1)  // 3
```

---

## 17.10 Conditional Extensions

เราสามารถเพิ่ม extension โดยมีเงื่อนไขได้โดยใช้ `where` clause:

```swift
// Extension เฉพาะเมื่อ Element เป็น Numeric
extension Array where Element: Numeric {
    var sum: Element {
        return reduce(0, +)
    }
    
    var product: Element {
        return reduce(1, *)
    }
}

// Extension เฉพาะเมื่อ Element เป็น Comparable
extension Array where Element: Comparable {
    var sorted: [Element] {
        return self.sorted()
    }
    
    var minimum: Element? {
        return self.min()
    }
    
    var maximum: Element? {
        return self.max()
    }
}

let numbers = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3]
print(numbers.sum)      // 39
print(numbers.product)  // 32400
print(numbers.sorted)   // [1, 1, 2, 3, 3, 4, 5, 5, 6, 9]
print(numbers.minimum ?? 0)  // 1
print(numbers.maximum ?? 0)  // 9

let strings = ["banana", "apple", "cherry", "date"]
print(strings.sorted)   // ["apple", "banana", "cherry", "date"]
print(strings.minimum ?? "")  // apple
```

### Conditional Conformance

```swift
struct Wrapper<T> {
    var value: T
}

// Wrapper conform to Equatable เฉพาะเมื่อ T เป็น Equatable
extension Wrapper: Equatable where T: Equatable {
    static func == (lhs: Wrapper<T>, rhs: Wrapper<T>) -> Bool {
        return lhs.value == rhs.value
    }
}

// Wrapper conform to Comparable เฉพาะเมื่อ T เป็น Comparable
extension Wrapper: Comparable where T: Comparable {
    static func < (lhs: Wrapper<T>, rhs: Wrapper<T>) -> Bool {
        return lhs.value < rhs.value
    }
}

let w1 = Wrapper(value: 5)
let w2 = Wrapper(value: 5)
let w3 = Wrapper(value: 10)

print(w1 == w2)  // true
print(w1 < w3)   // true

let ws1 = Wrapper(value: "hello")
let ws2 = Wrapper(value: "world")
print(ws1 < ws2)  // true
```

---

## 17.11 การขยาย Generic Types

```swift
// Generic Stack
struct Stack<Element> {
    private var storage: [Element] = []
    
    mutating func push(_ element: Element) {
        storage.append(element)
    }
    
    mutating func pop() -> Element? {
        return storage.popLast()
    }
    
    var top: Element? {
        return storage.last
    }
    
    var isEmpty: Bool {
        return storage.isEmpty
    }
    
    var count: Int {
        return storage.count
    }
}

// Extension สำหรับ Stack ทั่วไป
extension Stack {
    func toArray() -> [Element] {
        return storage
    }
    
    mutating func pushAll(_ elements: [Element]) {
        elements.forEach { push($0) }
    }
}

// Extension เฉพาะเมื่อ Element เป็น Equatable
extension Stack where Element: Equatable {
    func contains(_ element: Element) -> Bool {
        return storage.contains(element)
    }
}

// Extension เฉพาะเมื่อ Element เป็น CustomStringConvertible
extension Stack: CustomStringConvertible where Element: CustomStringConvertible {
    var description: String {
        let items = storage.map { $0.description }.joined(separator: ", ")
        return "Stack[\(items)]"
    }
}

var intStack = Stack<Int>()
intStack.pushAll([1, 2, 3, 4, 5])
print(intStack.toArray())     // [1, 2, 3, 4, 5]
print(intStack.contains(3))   // true
print(intStack.description)   // Stack[1, 2, 3, 4, 5]

var stringStack = Stack<String>()
stringStack.push("แอปเปิ้ล")
stringStack.push("กล้วย")
stringStack.push("ส้ม")
print(stringStack.description)  // Stack[แอปเปิ้ล, กล้วย, ส้ม]
```

---

## 17.12 Retroactive Modeling (การทำ Conformance ย้อนหลัง)

Retroactive modeling คือการทำให้ type จาก module อื่น (ที่เราไม่ได้เป็นเจ้าของ) conform to protocol:

```swift
// สมมติว่า CLLocationCoordinate2D มาจาก framework อื่น
struct CLLocationCoordinate2D {
    var latitude: Double
    var longitude: Double
}

// เราทำให้มัน conform to Codable ผ่าน extension
extension CLLocationCoordinate2D: Codable {
    enum CodingKeys: String, CodingKey {
        case latitude = "lat"
        case longitude = "lng"
    }
    
    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        latitude = try container.decode(Double.self, forKey: .latitude)
        longitude = try container.decode(Double.self, forKey: .longitude)
    }
    
    func encode(to encoder: Encoder) throws {
        var container = encoder.container(keyedBy: CodingKeys.self)
        try container.encode(latitude, forKey: .latitude)
        try container.encode(longitude, forKey: .longitude)
    }
}

// ทำให้ conformance to CustomStringConvertible
extension CLLocationCoordinate2D: CustomStringConvertible {
    var description: String {
        return "(\(latitude)°N, \(longitude)°E)"
    }
}

let bangkok = CLLocationCoordinate2D(latitude: 13.7563, longitude: 100.5018)
print(bangkok.description)  // (13.7563°N, 100.5018°E)
```

---

## 17.13 Best Practices สำหรับ Extensions

### 1. ใช้ extension เพื่อจัดกลุ่ม Protocol Conformance

```swift
// แทนที่จะใส่ทุกอย่างในนิยามหลัก
class ViewController {
    var data: [String] = []
}

// แยก extension ตาม protocol
extension ViewController: UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return data.count
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        // ...
        return UITableViewCell()
    }
}

extension ViewController: UITableViewDelegate {
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        // ...
    }
}
```

### 2. ใช้ extension เพื่อแบ่ง logic ตามหัวข้อ

```swift
class NetworkManager {
    var baseURL: String
    
    init(baseURL: String) {
        self.baseURL = baseURL
    }
}

// MARK: - Authentication
extension NetworkManager {
    func login(username: String, password: String) {
        // login logic
    }
    
    func logout() {
        // logout logic
    }
}

// MARK: - Data Fetching
extension NetworkManager {
    func fetchUser(id: Int) {
        // fetch user
    }
    
    func fetchProducts() {
        // fetch products
    }
}

// MARK: - File Upload
extension NetworkManager {
    func uploadImage(data: Data) {
        // upload image
    }
}
```

### 3. ใช้ MARK Comments เพื่อจัดระเบียบ

```swift
class DataManager {
    // ...
}

// MARK: - CRUD Operations
extension DataManager {
    func create() { }
    func read() { }
    func update() { }
    func delete() { }
}

// MARK: - Validation
extension DataManager {
    func validate() -> Bool { return true }
    func sanitize() { }
}
```

---

## 17.14 การจัด Code ด้วย Extensions

### แยก File ตาม Extension

ในโปรเจคจริง เราสามารถแบ่ง extensions เป็น files แยกกัน:

```
MyProject/
├── Models/
│   ├── User.swift              // นิยามหลัก
│   ├── User+Codable.swift      // Codable conformance
│   ├── User+Validation.swift   // Validation methods
│   └── User+Display.swift      // Display helpers
```

**User.swift:**
```swift
struct User {
    var id: Int
    var name: String
    var email: String
    var age: Int
    var createdAt: Date
}
```

**User+Codable.swift:**
```swift
extension User: Codable {
    enum CodingKeys: String, CodingKey {
        case id
        case name
        case email
        case age
        case createdAt = "created_at"
    }
}
```

**User+Validation.swift:**
```swift
extension User {
    var isValidEmail: Bool {
        return email.contains("@") && email.contains(".")
    }
    
    var isAdult: Bool {
        return age >= 18
    }
    
    var isValidAge: Bool {
        return age >= 0 && age <= 150
    }
    
    func validate() throws -> Bool {
        guard !name.isEmpty else {
            throw ValidationError.emptyName
        }
        guard isValidEmail else {
            throw ValidationError.invalidEmail
        }
        guard isValidAge else {
            throw ValidationError.invalidAge
        }
        return true
    }
    
    enum ValidationError: Error {
        case emptyName
        case invalidEmail
        case invalidAge
    }
}
```

**User+Display.swift:**
```swift
extension User: CustomStringConvertible {
    var description: String {
        return "User(id: \(id), name: \(name), email: \(email))"
    }
}

extension User {
    var displayName: String {
        return name.isEmpty ? "ผู้ใช้ไม่ระบุชื่อ" : name
    }
    
    var ageDescription: String {
        switch age {
        case ..<18:
            return "ผู้เยาว์"
        case 18..<60:
            return "ผู้ใหญ่"
        default:
            return "ผู้สูงอายุ"
        }
    }
}
```

---

## 17.15 Extensions vs Subclassing

| ลักษณะ | Extension | Subclassing |
|--------|-----------|-------------|
| ใช้ได้กับ | Class, Struct, Enum, Protocol | Class เท่านั้น |
| เพิ่ม stored property | ไม่ได้ | ได้ |
| Override method | ไม่ได้ | ได้ |
| หลาย conformances | ได้ | ทำได้แต่ซับซ้อน |
| Type ดั้งเดิม | ไม่เปลี่ยน | สร้าง type ใหม่ |

### เมื่อใช้ Extension:

```swift
// เพิ่มความสามารถให้ type ที่มีอยู่โดยไม่ต้อง subclass
extension String {
    func truncated(to maxLength: Int) -> String {
        if count <= maxLength { return self }
        return String(prefix(maxLength)) + "..."
    }
}

let longText = "นี่คือข้อความที่ยาวมากๆ ซึ่งต้องการตัดให้สั้นลง"
print(longText.truncated(to: 20))
```

### เมื่อใช้ Subclassing:

```swift
class Animal {
    var name: String
    
    init(name: String) {
        self.name = name
    }
    
    func speak() -> String {
        return "..."
    }
}

class Dog: Animal {
    override func speak() -> String {
        return "โฮ่ง!"
    }
    
    func fetch() {
        print("\(name) วิ่งไปหาลูกบอล")
    }
}

class Cat: Animal {
    override func speak() -> String {
        return "เมี้ยว!"
    }
    
    var isIndoor: Bool = true
}
```

---

## 17.16 Common Standard Library Extensions

### Int Extensions ที่มีประโยชน์

```swift
extension Int {
    // แปลงเป็น Roman Numerals
    var romanNumeral: String {
        let values = [(1000, "M"), (900, "CM"), (500, "D"), (400, "CD"),
                      (100, "C"), (90, "XC"), (50, "L"), (40, "XL"),
                      (10, "X"), (9, "IX"), (5, "V"), (4, "IV"), (1, "I")]
        
        var result = ""
        var remaining = self
        
        for (value, numeral) in values {
            while remaining >= value {
                result += numeral
                remaining -= value
            }
        }
        return result
    }
    
    // ตรวจสอบจำนวนเฉพาะ
    var isPrime: Bool {
        guard self > 1 else { return false }
        guard self > 3 else { return true }
        guard self % 2 != 0 && self % 3 != 0 else { return false }
        
        var i = 5
        while i * i <= self {
            if self % i == 0 || self % (i + 2) == 0 {
                return false
            }
            i += 6
        }
        return true
    }
    
    // Fibonacci
    static func fibonacci(_ n: Int) -> Int {
        guard n > 1 else { return n }
        var a = 0, b = 1
        for _ in 2...n {
            let temp = a + b
            a = b
            b = temp
        }
        return b
    }
}

print(42.romanNumeral)   // XLII
print(2024.romanNumeral) // MMXXIV
print(7.isPrime)         // true
print(10.isPrime)        // false
print(Int.fibonacci(10)) // 55
```

### String Extensions ที่มีประโยชน์

```swift
extension String {
    // ตัดช่องว่างหัวท้าย
    var trimmed: String {
        return trimmingCharacters(in: .whitespacesAndNewlines)
    }
    
    // แปลงเป็น camelCase
    var camelCased: String {
        let words = components(separatedBy: CharacterSet.alphanumerics.inverted)
        let first = words.first?.lowercased() ?? ""
        let rest = words.dropFirst().map { $0.capitalized }
        return ([first] + rest).joined()
    }
    
    // แปลงเป็น snake_case
    var snakeCased: String {
        var result = ""
        for (index, char) in enumerated() {
            if char.isUppercase && index > 0 {
                result += "_"
            }
            result += char.lowercased()
        }
        return result
    }
    
    // ตรวจสอบว่าเป็นตัวเลขหรือไม่
    var isNumeric: Bool {
        return !isEmpty && allSatisfy { $0.isNumber }
    }
    
    // แทนที่หลาย pattern พร้อมกัน
    func replacingOccurrences(of replacements: [String: String]) -> String {
        var result = self
        for (old, new) in replacements {
            result = result.replacingOccurrences(of: old, with: new)
        }
        return result
    }
    
    // แปลงเป็น URL safe string
    var urlEncoded: String {
        return addingPercentEncoding(withAllowedCharacters: .urlQueryAllowed) ?? self
    }
    
    // Repeat string
    func repeated(_ times: Int) -> String {
        return String(repeating: self, count: times)
    }
}

let messy = "  Hello World  "
print(messy.trimmed)  // "Hello World"

let title = "hello world foo bar"
print(title.camelCased)  // "helloWorldFooBar"

let camelCase = "helloWorldFooBar"
print(camelCase.snakeCased)  // "hello_world_foo_bar"

print("12345".isNumeric)   // true
print("123a5".isNumeric)   // false

let template = "สวัสดี {name} คุณอายุ {age} ปี"
print(template.replacingOccurrences(of: ["{name}": "สมชาย", "{age}": "30"]))
// สวัสดี สมชาย คุณอายุ 30 ปี
```

### Array Extensions ที่มีประโยชน์

```swift
extension Array {
    // แบ่ง array เป็น chunks
    func chunked(into size: Int) -> [[Element]] {
        return stride(from: 0, to: count, by: size).map {
            Array(self[$0..<Swift.min($0 + size, count)])
        }
    }
    
    // สุ่มลำดับ
    func shuffled() -> [Element] {
        var result = self
        for i in stride(from: count - 1, through: 1, by: -1) {
            let j = Int.random(in: 0...i)
            result.swapAt(i, j)
        }
        return result
    }
    
    // ลบ element ที่ index ที่กำหนด แล้วคืน array ใหม่
    func removing(at index: Int) -> [Element] {
        var result = self
        result.remove(at: index)
        return result
    }
    
    // เพิ่ม element เฉพาะเมื่อเป็น Unique (ต้องใช้กับ Equatable)
    mutating func appendIfNotExists(_ element: Element) where Element: Equatable {
        if !contains(element) {
            append(element)
        }
    }
}

extension Array where Element: Hashable {
    // ลบ duplicate
    var unique: [Element] {
        var seen = Set<Element>()
        return filter { seen.insert($0).inserted }
    }
    
    // นับจำนวนแต่ละ element
    var frequencies: [Element: Int] {
        return reduce(into: [:]) { counts, element in
            counts[element, default: 0] += 1
        }
    }
}

let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
print(numbers.chunked(into: 3))
// [[1, 2, 3], [4, 5, 6], [7, 8, 9], [10]]

let withDuplicates = [1, 2, 3, 2, 4, 3, 5, 1]
print(withDuplicates.unique)     // [1, 2, 3, 4, 5]
print(withDuplicates.frequencies)  // [1: 2, 2: 2, 3: 2, 4: 1, 5: 1]
```

---

## 17.17 แบบฝึกหัด

### แบบฝึกหัดที่ 1: Date Extensions

สร้าง extension บน `Date` ที่มีความสามารถดังนี้:

```swift
import Foundation

extension Date {
    // คืนวันเริ่มต้นของวัน (midnight)
    var startOfDay: Date {
        return Calendar.current.startOfDay(for: self)
    }
    
    // คืนวันสิ้นสุดของวัน
    var endOfDay: Date {
        var components = DateComponents()
        components.day = 1
        components.second = -1
        return Calendar.current.date(byAdding: components, to: startOfDay)!
    }
    
    // ตรวจสอบว่าเป็นวันนี้หรือไม่
    var isToday: Bool {
        return Calendar.current.isDateInToday(self)
    }
    
    // ตรวจสอบว่าเป็นวันหยุดสุดสัปดาห์หรือไม่
    var isWeekend: Bool {
        return Calendar.current.isDateInWeekend(self)
    }
    
    // เพิ่มวัน
    func adding(days: Int) -> Date {
        return Calendar.current.date(byAdding: .day, value: days, to: self)!
    }
    
    // แปลงเป็น String
    func formatted(as format: String) -> String {
        let formatter = DateFormatter()
        formatter.dateFormat = format
        return formatter.string(from: self)
    }
    
    // จำนวนวันระหว่าง 2 วัน
    func days(until date: Date) -> Int {
        let components = Calendar.current.dateComponents([.day], from: self, to: date)
        return abs(components.day ?? 0)
    }
}

let today = Date()
print(today.formatted(as: "dd/MM/yyyy"))
print(today.isToday)    // true
print(today.isWeekend)  // ขึ้นกับวันที่รัน

let nextWeek = today.adding(days: 7)
print(today.days(until: nextWeek))  // 7
```

### แบบฝึกหัดที่ 2: Collection Protocol Extension

```swift
protocol Stackable {
    associatedtype Element
    mutating func push(_ element: Element)
    mutating func pop() -> Element?
    var peek: Element? { get }
    var isEmpty: Bool { get }
    var count: Int { get }
}

extension Stackable {
    var isEmpty: Bool {
        return count == 0
    }
    
    mutating func pushAll<S: Sequence>(_ sequence: S) where S.Element == Element {
        sequence.forEach { push($0) }
    }
    
    func toArray() -> [Element] {
        var result: [Element] = []
        var copy = self
        while let element = copy.pop() {
            result.insert(element, at: 0)
        }
        return result
    }
}

struct MinStack: Stackable {
    private var storage: [Int] = []
    private var minStorage: [Int] = []
    
    mutating func push(_ element: Int) {
        storage.append(element)
        if let currentMin = minStorage.last {
            minStorage.append(min(element, currentMin))
        } else {
            minStorage.append(element)
        }
    }
    
    mutating func pop() -> Int? {
        minStorage.popLast()
        return storage.popLast()
    }
    
    var peek: Int? { storage.last }
    var count: Int { storage.count }
    
    var minimum: Int? { minStorage.last }
}

var stack = MinStack()
stack.pushAll([5, 3, 8, 1, 4])
print(stack.minimum ?? 0)  // 1
print(stack.toArray())     // [5, 3, 8, 1, 4]
```

### แบบฝึกหัดที่ 3: Network Response Extensions

```swift
enum HTTPStatusCode: Int {
    case ok = 200
    case created = 201
    case noContent = 204
    case badRequest = 400
    case unauthorized = 401
    case forbidden = 403
    case notFound = 404
    case internalServerError = 500
    case serviceUnavailable = 503
}

extension HTTPStatusCode {
    var isSuccess: Bool {
        return (200...299).contains(rawValue)
    }
    
    var isClientError: Bool {
        return (400...499).contains(rawValue)
    }
    
    var isServerError: Bool {
        return (500...599).contains(rawValue)
    }
    
    var description: String {
        switch self {
        case .ok: return "OK - สำเร็จ"
        case .created: return "Created - สร้างสำเร็จ"
        case .noContent: return "No Content - ไม่มีข้อมูล"
        case .badRequest: return "Bad Request - คำขอไม่ถูกต้อง"
        case .unauthorized: return "Unauthorized - ไม่ได้รับอนุญาต"
        case .forbidden: return "Forbidden - ห้ามเข้าถึง"
        case .notFound: return "Not Found - ไม่พบข้อมูล"
        case .internalServerError: return "Internal Server Error - เกิดข้อผิดพลาดที่เซิร์ฟเวอร์"
        case .serviceUnavailable: return "Service Unavailable - บริการไม่พร้อมใช้งาน"
        }
    }
}

if let status = HTTPStatusCode(rawValue: 200) {
    print(status.description)   // OK - สำเร็จ
    print(status.isSuccess)     // true
}

if let status = HTTPStatusCode(rawValue: 404) {
    print(status.description)   // Not Found - ไม่พบข้อมูล
    print(status.isClientError) // true
}
```

---

## 17.18 ตัวอย่าง Real-World: Shopping Cart

```swift
struct Money {
    var amount: Double
    var currency: String
    
    init(_ amount: Double, _ currency: String = "THB") {
        self.amount = amount
        self.currency = currency
    }
}

extension Money {
    static func + (lhs: Money, rhs: Money) -> Money {
        assert(lhs.currency == rhs.currency, "ไม่สามารถบวกเงินต่างสกุลได้")
        return Money(lhs.amount + rhs.amount, lhs.currency)
    }
    
    static func * (lhs: Money, rhs: Int) -> Money {
        return Money(lhs.amount * Double(rhs), lhs.currency)
    }
    
    var formatted: String {
        let formatter = NumberFormatter()
        formatter.numberStyle = .currency
        formatter.currencyCode = currency
        return formatter.string(from: NSNumber(value: amount)) ?? "\(amount) \(currency)"
    }
    
    var withVAT: Money {
        return Money(amount * 1.07, currency)
    }
    
    func discounted(by percentage: Double) -> Money {
        return Money(amount * (1 - percentage / 100), currency)
    }
}

extension Money: CustomStringConvertible {
    var description: String { return formatted }
}

extension Money: Comparable {
    static func < (lhs: Money, rhs: Money) -> Bool {
        return lhs.amount < rhs.amount
    }
    
    static func == (lhs: Money, rhs: Money) -> Bool {
        return lhs.amount == rhs.amount && lhs.currency == rhs.currency
    }
}

let price = Money(100)
let total = price * 3
print(total)                    // ฿300.00
print(total.withVAT)           // ฿321.00
print(total.discounted(by: 10)) // ฿270.00

let price1 = Money(200)
let price2 = Money(150)
print(price1 + price2)          // ฿350.00
print(price1 > price2)          // true
```

---

## 17.19 สรุป

Extensions เป็น feature ที่ทรงพลังใน Swift ที่ช่วยให้:

1. **เพิ่มความสามารถ** ให้กับ type ที่มีอยู่แล้วโดยไม่ต้องแก้ไข source code เดิม
2. **จัดระเบียบ code** โดยแบ่ง implementation ตามหัวข้อหรือ protocol conformance
3. **ทำ Retroactive Modeling** โดยทำให้ type จาก framework อื่น conform to protocol ที่เราต้องการ
4. **Protocol Extensions** ช่วยให้ provide default implementations ได้
5. **Conditional Extensions** ช่วยให้เพิ่มความสามารถเฉพาะเมื่อมีเงื่อนไขที่กำหนด

### Guidelines สำคัญ:

- ใช้ extensions เพื่อ **organize code** ไม่ใช่เพื่อหลีกเลี่ยงการเขียน code ที่ดี
- **แยก extension** ตาม concern (Codable, Display, Validation เป็นต้น)
- ระวัง **naming conflicts** เมื่อ extend type จาก library อื่น
- ใช้ **MARK comments** เพื่อจัดระเบียบในไฟล์เดียวกัน
- Extensions ไม่สามารถเพิ่ม **stored properties** ได้
- สำหรับ class ใน extension ทำได้แค่ **convenience initializers**

---

*ต่อไปใน Part 18: Error Handling - การจัดการข้อผิดพลาด*
