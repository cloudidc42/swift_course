# ตอนที่ 6: Functions ใน Swift

## บทนำ

Function (ฟังก์ชัน) คือบล็อกของโค้ดที่ถูกตั้งชื่อและสามารถนำมาใช้ซ้ำได้ Functions เป็นหัวใจสำคัญของการเขียนโปรแกรม ช่วยให้โค้ดมีความเป็นระเบียบ อ่านง่าย และบำรุงรักษาง่ายขึ้น

ใน Swift, Functions เป็น "First-Class Citizens" หมายความว่าสามารถส่งเป็น Parameter, เก็บในตัวแปร, หรือ Return จาก Function อื่นได้ นี่ทำให้ Swift รองรับการเขียนโปรแกรมแบบ Functional Programming ได้อย่างสวยงาม

ในบทนี้เราจะเรียนรู้:
- การสร้างและเรียกใช้ Functions พื้นฐาน
- Parameters และ Return Values ต่างๆ
- Advanced Features เช่น Variadic, In-out, Default Parameters
- Functions เป็น First-Class Citizens
- Higher-Order Functions
- Error Handling ด้วย Throwing Functions
- Advanced Attributes เช่น `@discardableResult`, `@autoclosure`, `@escaping`
- Recursive Functions และ Pure Functions

---

## 6.1 Function Syntax พื้นฐาน

### รูปแบบการสร้าง Function

```swift
func functionName(parameters) -> ReturnType {
    // โค้ดภายใน function
    return value
}
```

### Function ที่ไม่รับและไม่คืนค่า

```swift
// Function ง่ายที่สุด
func sayHello() {
    print("สวัสดี!")
}

// เรียกใช้
sayHello()  // Output: สวัสดี!

// Function แสดงข้อมูล
func printDivider() {
    print(String(repeating: "=", count: 40))
}

func showWelcome() {
    printDivider()
    print("ยินดีต้อนรับสู่ระบบ")
    printDivider()
}

showWelcome()
```

### Function ที่รับ Parameter แต่ไม่คืนค่า

```swift
func greet(name: String) {
    print("สวัสดี, \(name)!")
}

greet(name: "สมชาย")

func printMultiplied(number: Int, times: Int) {
    for _ in 1...times {
        print(number, terminator: " ")
    }
    print()
}

printMultiplied(number: 7, times: 5)
// Output: 7 7 7 7 7
```

### Function ที่คืนค่า

```swift
func square(_ number: Int) -> Int {
    return number * number
}

let result = square(5)
print("5² = \(result)")  // Output: 5² = 25

// ถ้า body มีแค่บรรทัดเดียว สามารถละ return ได้
func cube(_ number: Int) -> Int {
    number * number * number
}

print("3³ = \(cube(3))")  // Output: 3³ = 27
```

---

## 6.2 Function Parameters

### Single Parameter

```swift
func double(_ x: Int) -> Int {
    return x * 2
}

print(double(7))  // 14
```

### Multiple Parameters

```swift
func add(_ a: Int, _ b: Int) -> Int {
    return a + b
}

func calculateArea(width: Double, height: Double) -> Double {
    return width * height
}

print(add(3, 4))  // 7
print(calculateArea(width: 5.0, height: 3.0))  // 15.0
```

### Named Parameters

```swift
// Named parameters ทำให้อ่านโค้ดง่ายขึ้น
func createFullName(firstName: String, lastName: String) -> String {
    return "\(firstName) \(lastName)"
}

let fullName = createFullName(firstName: "สมชาย", lastName: "ใจดี")
print(fullName)  // สมชาย ใจดี
```

### Parameters ที่มี Type ต่างๆ

```swift
func describe(name: String, age: Int, score: Double, isActive: Bool) -> String {
    let status = isActive ? "ยังใช้งานอยู่" : "ไม่ได้ใช้งาน"
    return "\(name) (อายุ \(age)) คะแนน: \(score) - \(status)"
}

let description = describe(name: "วิชัย", age: 25, score: 92.5, isActive: true)
print(description)
```

---

## 6.3 Return Values

### Return ค่าเดี่ยว

```swift
func max(_ a: Int, _ b: Int) -> Int {
    return a > b ? a : b
}

func min(_ a: Int, _ b: Int) -> Int {
    a < b ? a : b  // implicit return
}

print(max(10, 7))   // 10
print(min(10, 7))   // 7
```

### Return String

```swift
func gradeDescription(for score: Int) -> String {
    switch score {
    case 90...100: return "ดีเยี่ยม (A)"
    case 80..<90:  return "ดี (B)"
    case 70..<80:  return "พอใช้ (C)"
    case 60..<70:  return "ผ่าน (D)"
    default:       return "ไม่ผ่าน (F)"
    }
}

print(gradeDescription(for: 87))   // ดี (B)
print(gradeDescription(for: 45))   // ไม่ผ่าน (F)
```

### Return Optional

```swift
func findUser(withID id: Int) -> String? {
    let users = [1: "สมชาย", 2: "สมหญิง", 3: "วิชัย"]
    return users[id]
}

if let user = findUser(withID: 2) {
    print("พบผู้ใช้: \(user)")
} else {
    print("ไม่พบผู้ใช้")
}

// ด้วย nil coalescing
let userName = findUser(withID: 99) ?? "ไม่ทราบชื่อ"
print("ชื่อ: \(userName)")
```

### Void Return

```swift
// Void เป็น Return Type เมื่อไม่คืนค่า (ละได้)
func printLine() -> Void {
    print(String(repeating: "-", count: 30))
}

// เขียนแบบนี้ก็เหมือนกัน
func printLine2() {
    print(String(repeating: "-", count: 30))
}
```

---

## 6.4 Multiple Return Values ด้วย Tuples

Tuple ทำให้ Function คืนค่าหลายค่าพร้อมกันได้

### Return Tuple พื้นฐาน

```swift
func minMax(in array: [Int]) -> (min: Int, max: Int) {
    var currentMin = array[0]
    var currentMax = array[0]
    
    for value in array[1...] {
        if value < currentMin {
            currentMin = value
        } else if value > currentMax {
            currentMax = value
        }
    }
    
    return (min: currentMin, max: currentMax)
}

let result = minMax(in: [3, 7, 1, 9, 5, 2, 8, 4, 6])
print("ต่ำสุด: \(result.min), สูงสุด: \(result.max)")

// Destructuring
let (minimum, maximum) = minMax(in: [10, 3, 27, 15])
print("Min: \(minimum), Max: \(maximum)")
```

### Return Tuple ที่ซับซ้อน

```swift
struct Statistics {
    let count: Int
    let sum: Double
    let mean: Double
    let min: Double
    let max: Double
}

func calculateStats(_ numbers: [Double]) -> (sum: Double, mean: Double, min: Double, max: Double) {
    guard !numbers.isEmpty else {
        return (sum: 0, mean: 0, min: 0, max: 0)
    }
    
    var sum = 0.0
    var min = numbers[0]
    var max = numbers[0]
    
    for n in numbers {
        sum += n
        if n < min { min = n }
        if n > max { max = n }
    }
    
    return (sum: sum, mean: sum / Double(numbers.count), min: min, max: max)
}

let data = [4.5, 7.2, 3.1, 9.8, 5.6, 2.3, 8.7]
let stats = calculateStats(data)
print("ผลรวม: \(stats.sum)")
print("เฉลี่ย: \(String(format: "%.2f", stats.mean))")
print("ต่ำสุด: \(stats.min)")
print("สูงสุด: \(stats.max)")
```

### Return Optional Tuple

```swift
func divide(_ a: Double, by b: Double) -> (quotient: Double, remainder: Double)? {
    guard b != 0 else { return nil }
    let quotient = (a / b).rounded(.down)
    let remainder = a.truncatingRemainder(dividingBy: b)
    return (quotient: quotient, remainder: remainder)
}

if let result = divide(17, by: 5) {
    print("17 ÷ 5 = \(result.quotient) เศษ \(result.remainder)")
} else {
    print("ไม่สามารถหารด้วย 0 ได้")
}
```

---

## 6.5 Argument Labels และ Parameter Names

Swift มีระบบตั้งชื่อ Parameter ที่ยืดหยุ่น ทำให้โค้ดอ่านออกเสียงได้เป็นธรรมชาติ

### Argument Label และ Parameter Name แยกกัน

```swift
// รูปแบบ: func name(argumentLabel parameterName: Type)
func greet(person name: String) -> String {
    return "สวัสดี, \(name)!"
}

// เรียกใช้ด้วย argument label
print(greet(person: "สมชาย"))

// ตัวอย่างที่ทำให้อ่านออกเสียงได้ดีขึ้น
func move(from startPoint: String, to endPoint: String) {
    print("เดินทางจาก \(startPoint) ไป \(endPoint)")
}

move(from: "กรุงเทพ", to: "เชียงใหม่")
```

### ละ Argument Label ด้วย `_`

```swift
// ใช้ _ เพื่อไม่ต้องพิมพ์ label ตอนเรียกใช้
func add(_ a: Int, _ b: Int) -> Int {
    return a + b
}

print(add(3, 5))  // ไม่ต้องพิมพ์ label

// ผสม: บาง parameter มี label บางตัวไม่มี
func setColor(_ red: Int, green: Int, blue: Int) {
    print("RGB: \(red), \(green), \(blue)")
}

setColor(255, green: 128, blue: 0)
```

### ตัวอย่างการออกแบบ API ที่ดี

```swift
// API ที่อ่านได้เป็นธรรมชาติ
func insert(_ newElement: String, at index: Int, into array: inout [String]) {
    array.insert(newElement, at: index)
}

var list = ["a", "b", "d", "e"]
insert("c", at: 2, into: &list)
print(list)  // ["a", "b", "c", "d", "e"]

// การใช้ with, for, by, in, from, to เป็น label
func multiply(_ number: Int, by factor: Int) -> Int {
    return number * factor
}

func search(for keyword: String, in text: String) -> Bool {
    return text.contains(keyword)
}

print(multiply(6, by: 7))  // 42
print(search(for: "Swift", in: "Hello Swift!"))  // true
```

---

## 6.6 Default Parameter Values

Parameters สามารถมีค่าเริ่มต้นได้ เมื่อเรียกใช้ไม่จำเป็นต้องส่งค่านั้น

### Default Parameter พื้นฐาน

```swift
func createGreeting(name: String, greeting: String = "สวัสดี") -> String {
    return "\(greeting), \(name)!"
}

print(createGreeting(name: "สมชาย"))                   // สวัสดี, สมชาย!
print(createGreeting(name: "สมหญิง", greeting: "ดีจ้า"))  // ดีจ้า, สมหญิง!
```

### หลาย Default Parameters

```swift
func formatNumber(_ number: Double, 
                  decimalPlaces: Int = 2,
                  prefix: String = "",
                  suffix: String = "") -> String {
    let formatted = String(format: "%.\(decimalPlaces)f", number)
    return "\(prefix)\(formatted)\(suffix)"
}

print(formatNumber(1234.5678))                    // 1234.57
print(formatNumber(1234.5678, decimalPlaces: 4))  // 1234.5678
print(formatNumber(9999.99, prefix: "฿"))         // ฿9999.99
print(formatNumber(98.6, suffix: "°F"))           // 98.60°F
```

### Default Parameters กับ Optional

```swift
func log(message: String, level: String = "INFO", timestamp: Date? = nil) {
    let time = timestamp ?? Date()
    print("[\(level)] \(time): \(message)")
}

log(message: "โปรแกรมเริ่มทำงาน")
log(message: "เกิดข้อผิดพลาด", level: "ERROR")
```

### Best Practice: ใส่ Default Parameters ไว้ท้าย

```swift
// ดี: Required parameters มาก่อน, Default parameters มาหลัง
func sendEmail(to recipient: String,
               subject: String,
               body: String,
               cc: String = "",
               isHTML: Bool = false) {
    print("ส่งอีเมลถึง: \(recipient)")
    print("หัวข้อ: \(subject)")
    if !cc.isEmpty { print("CC: \(cc)") }
}

sendEmail(to: "test@example.com", subject: "สวัสดี", body: "เนื้อหาอีเมล")
```

---

## 6.7 Variadic Parameters

Variadic Parameter รับค่าได้หลายตัวในรูปแบบ Array

### Variadic Parameter พื้นฐาน

```swift
// ใช้ ... เพื่อกำหนด variadic
func sum(_ numbers: Int...) -> Int {
    return numbers.reduce(0, +)
}

print(sum(1, 2, 3))           // 6
print(sum(10, 20, 30, 40))    // 100
print(sum())                  // 0

// ชื่อ parameters เป็น Array ภายใน function
func average(_ numbers: Double...) -> Double {
    guard !numbers.isEmpty else { return 0 }
    return numbers.reduce(0, +) / Double(numbers.count)
}

print(average(80, 90, 75, 85))  // 82.5
```

### Variadic กับ Parameters อื่น

```swift
// Variadic ต้องอยู่หลัง regular parameters
func printList(title: String, _ items: String...) {
    print("\(title):")
    for (index, item) in items.enumerated() {
        print("  \(index + 1). \(item)")
    }
}

printList(title: "รายการผลไม้", "แอปเปิ้ล", "กล้วย", "ส้ม", "มะม่วง")

// Variadic ที่มี Type ต่างๆ
func logValues(_ values: Any...) {
    for value in values {
        print(type(of: value): \(value)")
    }
}

logValues(42, "Hello", 3.14, true)
```

### ประยุกต์ใช้ Variadic

```swift
// สร้าง SQL-like WHERE clause
func whereClause(_ conditions: String...) -> String {
    guard !conditions.isEmpty else { return "" }
    return "WHERE " + conditions.joined(separator: " AND ")
}

let clause = whereClause("age > 18", "status = 'active'", "city = 'Bangkok'")
print(clause)

// ตรวจสอบว่ามีค่าใดค่าหนึ่งอยู่ใน list
func any<T: Equatable>(_ value: T, in options: T...) -> Bool {
    return options.contains(value)
}

print(any(3, in: 1, 2, 3, 4, 5))  // true
print(any(7, in: 1, 2, 3, 4, 5))  // false
```

---

## 6.8 In-Out Parameters

In-out parameters ช่วยให้ Function สามารถแก้ไขค่าของตัวแปรภายนอกได้

### In-Out Parameter พื้นฐาน

```swift
// ใช้ inout keyword และ & เมื่อส่งค่า
func doubleInPlace(_ number: inout Int) {
    number *= 2
}

var myNumber = 10
print("ก่อน: \(myNumber)")    // 10
doubleInPlace(&myNumber)
print("หลัง: \(myNumber)")   // 20

// Swap สองค่า
func swap<T>(_ a: inout T, _ b: inout T) {
    let temp = a
    a = b
    b = temp
}

var x = "Hello"
var y = "World"
swap(&x, &y)
print("x = \(x), y = \(y)")  // x = World, y = Hello
```

### In-Out กับ Array

```swift
func sortAndRemoveDuplicates(_ array: inout [Int]) {
    array.sort()
    array = Array(Set(array)).sorted()
}

var numbers = [5, 3, 8, 3, 1, 9, 5, 2, 8]
print("ก่อน: \(numbers)")
sortAndRemoveDuplicates(&numbers)
print("หลัง: \(numbers)")

// เพิ่มหรือแก้ไข Element ใน Array
func appendIfNotExists(_ element: Int, to array: inout [Int]) {
    if !array.contains(element) {
        array.append(element)
    }
}

var uniqueList = [1, 2, 3]
appendIfNotExists(4, to: &uniqueList)  // เพิ่ม
appendIfNotExists(2, to: &uniqueList)  // ไม่เพิ่ม ซ้ำ
print(uniqueList)  // [1, 2, 3, 4]
```

### ข้อควรระวัง

```swift
// ไม่สามารถส่งค่าที่เป็น let หรือ literal
let constant = 5
// doubleInPlace(&constant)  // Error: Cannot pass immutable value

// ไม่สามารถส่ง expression
// doubleInPlace(&(x + 1))  // Error: Cannot pass non-lvalue

// in-out parameters ไม่ใช่ reference ที่ต่อเนื่อง
// มีการ copy-in และ copy-out
```

---

## 6.9 Function Types

ใน Swift ทุก Function มี Type ที่ประกอบด้วย Parameter Types และ Return Type

### Function Type คืออะไร

```swift
// Function นี้มี Type: (Int, Int) -> Int
func addInts(_ a: Int, _ b: Int) -> Int {
    return a + b
}

// Function นี้มี Type: (String) -> Void หรือ (String) -> ()
func printString(_ s: String) {
    print(s)
}

// Function ที่ไม่รับและไม่คืนค่า: () -> Void
func doNothing() { }
```

### เก็บ Function ในตัวแปร

```swift
// เก็บ function reference ในตัวแปร
var mathOperation: (Int, Int) -> Int = addInts

print(mathOperation(3, 4))  // 7

// เปลี่ยน function ที่ตัวแปรชี้ไป
func multiplyInts(_ a: Int, _ b: Int) -> Int {
    return a * b
}

mathOperation = multiplyInts
print(mathOperation(3, 4))  // 12
```

### Function Type เป็น Parameter

```swift
func applyOperation(_ op: (Int, Int) -> Int, to a: Int, and b: Int) -> Int {
    return op(a, b)
}

func add(_ a: Int, _ b: Int) -> Int { a + b }
func multiply(_ a: Int, _ b: Int) -> Int { a * b }

print(applyOperation(add, to: 5, and: 3))       // 8
print(applyOperation(multiply, to: 5, and: 3))  // 15
```

### Function Type เป็น Return Value

```swift
func chooseMathOperation(useAddition: Bool) -> (Int, Int) -> Int {
    if useAddition {
        return { a, b in a + b }
    } else {
        return { a, b in a * b }
    }
}

let operation = chooseMathOperation(useAddition: true)
print(operation(10, 5))  // 15

let operation2 = chooseMathOperation(useAddition: false)
print(operation2(10, 5))  // 50
```

---

## 6.10 Functions เป็น First-Class Citizens

ใน Swift Functions สามารถส่งผ่าน เก็บ และสร้างในระหว่าง runtime ได้เหมือนค่าทั่วไป

### ส่ง Function เป็น Argument

```swift
// ใช้ function เป็น argument
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

func isEven(_ n: Int) -> Bool { n % 2 == 0 }
func isOdd(_ n: Int) -> Bool { n % 2 != 0 }
func isGreaterThanFive(_ n: Int) -> Bool { n > 5 }

let evens = numbers.filter(isEven)
let odds = numbers.filter(isOdd)
let big = numbers.filter(isGreaterThanFive)

print("เลขคู่: \(evens)")
print("เลขคี่: \(odds)")
print("มากกว่า 5: \(big)")
```

### เก็บ Functions ใน Array

```swift
// Array ของ functions
let validators: [(String) -> Bool] = [
    { !$0.isEmpty },                          // ไม่ว่าง
    { $0.count >= 3 },                        // ยาวอย่างน้อย 3 ตัว
    { $0.first?.isUppercase ?? false },       // ขึ้นต้นด้วยตัวพิมพ์ใหญ่
    { $0.contains { $0.isNumber } }           // มีตัวเลขอย่างน้อย 1 ตัว
]

func validateAll(_ value: String, against validators: [(String) -> Bool]) -> Bool {
    return validators.allSatisfy { $0(value) }
}

print(validateAll("Hello123", against: validators))  // true
print(validateAll("hi", against: validators))        // false
```

### Functions ใน Dictionary

```swift
// Dictionary ของ functions (Command Pattern)
var commands: [String: () -> Void] = [
    "hello": { print("Hello, World!") },
    "time": { print("เวลา: \(Date())") },
    "bye": { print("Goodbye!") }
]

func executeCommand(_ cmd: String) {
    if let action = commands[cmd] {
        action()
    } else {
        print("ไม่รู้จำคำสั่ง: \(cmd)")
    }
}

executeCommand("hello")
executeCommand("bye")
executeCommand("unknown")
```

---

## 6.11 Higher-Order Functions

Higher-Order Functions คือ Functions ที่รับหรือคืน Functions อื่น

### map - แปลงทุก Element

```swift
let numbers = [1, 2, 3, 4, 5]

// แปลงเป็นกำลังสอง
let squares = numbers.map { $0 * $0 }
print("กำลังสอง: \(squares)")  // [1, 4, 9, 16, 25]

// แปลงเป็น String
let strings = numbers.map { "Number \($0)" }
print(strings)

// แปลง Array of Struct
struct Employee {
    let name: String
    let salary: Double
}

let employees = [
    Employee(name: "สมชาย", salary: 35000),
    Employee(name: "สมหญิง", salary: 42000),
    Employee(name: "วิชัย", salary: 38000)
]

let names = employees.map { $0.name }
let salaries = employees.map { $0.salary }
let afterRaise = employees.map { Employee(name: $0.name, salary: $0.salary * 1.1) }

print("ชื่อ: \(names)")
print("เงินเดือนหลังขึ้น 10%: \(afterRaise.map { $0.salary })")
```

### filter - คัดกรอง Elements

```swift
let scores = [55, 72, 88, 45, 91, 63, 79, 84]

// กรองผู้ที่ผ่าน
let passing = scores.filter { $0 >= 60 }
print("ผ่าน: \(passing)")

// กรองแบบซับซ้อน
let words = ["swift", "java", "python", "kotlin", "go", "typescript"]
let shortAndS = words.filter { $0.count <= 5 && $0.hasPrefix("s") }
print("คำสั้นที่ขึ้นต้นด้วย s: \(shortAndS)")
```

### reduce - รวบรวมเป็นค่าเดียว

```swift
let prices = [150.0, 250.0, 75.0, 320.0, 95.0]

// ผลรวม
let total = prices.reduce(0, +)
print("รวม: \(total)")

// สร้าง String จาก Array
let items = ["แอปเปิ้ล", "กล้วย", "ส้ม"]
let sentence = items.reduce("รายการ: ") { result, item in
    result.isEmpty ? item : result + ", " + item
}
print(sentence)

// สร้าง Dictionary จาก Array
let words2 = ["apple", "banana", "cherry", "date"]
let wordLengths = words2.reduce(into: [String: Int]()) { dict, word in
    dict[word] = word.count
}
print(wordLengths)
```

### flatMap และ compactMap

```swift
// flatMap - flatten nested arrays
let nested = [[1, 2, 3], [4, 5], [6, 7, 8, 9]]
let flat = nested.flatMap { $0 }
print("แบน: \(flat)")  // [1, 2, 3, 4, 5, 6, 7, 8, 9]

// compactMap - กรอง nil ออก
let optionals: [String?] = ["one", nil, "two", nil, "three"]
let nonNil = optionals.compactMap { $0 }
print("ไม่มี nil: \(nonNil)")

// แปลงและกรองพร้อมกัน
let stringNumbers = ["1", "2", "abc", "4", "five", "6"]
let validNumbers = stringNumbers.compactMap { Int($0) }
print("ตัวเลขที่ถูกต้อง: \(validNumbers)")
```

### Chaining Higher-Order Functions

```swift
struct Product {
    let name: String
    let category: String
    let price: Double
    let inStock: Bool
}

let products = [
    Product(name: "iPhone", category: "Electronics", price: 35000, inStock: true),
    Product(name: "MacBook", category: "Electronics", price: 55000, inStock: false),
    Product(name: "T-Shirt", category: "Clothing", price: 599, inStock: true),
    Product(name: "Jeans", category: "Clothing", price: 1299, inStock: true),
    Product(name: "Watch", category: "Electronics", price: 12000, inStock: true)
]

// หาสินค้า Electronics ที่มีในสต็อก และแสดงชื่อเรียงตามราคา
let availableElectronics = products
    .filter { $0.category == "Electronics" && $0.inStock }
    .sorted { $0.price < $1.price }
    .map { "\($0.name): ฿\($0.price)" }

print("Electronics ที่มีในสต็อก (เรียงตามราคา):")
availableElectronics.forEach { print("  \($0)") }

// คำนวณมูลค่ารวมของสต็อก
let totalValue = products
    .filter { $0.inStock }
    .reduce(0.0) { $0 + $1.price }
print("มูลค่ารวมสต็อก: ฿\(totalValue)")
```

---

## 6.12 Nested Functions

Nested Functions คือ Functions ที่ประกาศอยู่ภายใน Function อื่น

### Nested Function พื้นฐาน

```swift
func processData(_ data: [Int]) -> [Int] {
    // Helper function ที่ใช้เฉพาะภายใน processData
    func isValid(_ n: Int) -> Bool {
        return n >= 0 && n <= 100
    }
    
    func normalize(_ n: Int) -> Int {
        return min(max(n, 0), 100)
    }
    
    return data
        .filter(isValid)
        .map(normalize)
}

let input = [-5, 20, 150, 75, 0, 100, 45, -10, 99]
let output = processData(input)
print("ข้อมูลที่ประมวลผลแล้ว: \(output)")
```

### Nested Function เข้าถึง Outer Variables

```swift
func makeCounter(startingAt initial: Int = 0, step: Int = 1) -> () -> Int {
    var count = initial
    
    func increment() -> Int {
        count += step
        return count
    }
    
    return increment
}

let counter = makeCounter(startingAt: 0, step: 5)
print(counter())  // 5
print(counter())  // 10
print(counter())  // 15

let countdown = makeCounter(startingAt: 100, step: -10)
print(countdown())  // 90
print(countdown())  // 80
```

### Nested Function สำหรับ Recursive Logic

```swift
func calculateFactorial(_ n: Int) -> Int {
    func factorial(_ n: Int) -> Int {
        if n <= 1 { return 1 }
        return n * factorial(n - 1)
    }
    
    guard n >= 0 else { return 0 }
    return factorial(n)
}

for i in 0...8 {
    print("\(i)! = \(calculateFactorial(i))")
}
```

---

## 6.13 Function Overloading

Function Overloading คือการสร้าง Functions ที่มีชื่อเหมือนกันแต่รับ Parameters ต่างกัน

### Overloading ด้วย Parameter Types

```swift
func describe(_ value: Int) -> String {
    return "จำนวนเต็ม: \(value)"
}

func describe(_ value: Double) -> String {
    return "ทศนิยม: \(value)"
}

func describe(_ value: String) -> String {
    return "ข้อความ: \"\(value)\""
}

func describe(_ value: Bool) -> String {
    return "ตรรกะ: \(value ? "จริง" : "เท็จ")"
}

print(describe(42))         // จำนวนเต็ม: 42
print(describe(3.14))       // ทศนิยม: 3.14
print(describe("Hello"))    // ข้อความ: "Hello"
print(describe(true))       // ตรรกะ: จริง
```

### Overloading ด้วยจำนวน Parameters

```swift
func area(side: Double) -> Double {
    return side * side  // สี่เหลี่ยมจัตุรัส
}

func area(width: Double, height: Double) -> Double {
    return width * height  // สี่เหลี่ยมผืนผ้า
}

func area(base: Double, height: Double, isTriangle: Bool) -> Double {
    return isTriangle ? (base * height) / 2 : base * height
}

print("พื้นที่สี่เหลี่ยมจัตุรัส: \(area(side: 5))")
print("พื้นที่สี่เหลี่ยม: \(area(width: 4, height: 6))")
print("พื้นที่สามเหลี่ยม: \(area(base: 4, height: 6, isTriangle: true))")
```

### Overloading ด้วย Argument Labels

```swift
func convert(_ value: Int) -> Double {
    return Double(value)
}

func convert(_ value: String) -> Int? {
    return Int(value)
}

func convert(celsius: Double) -> Double {
    return celsius * 9/5 + 32
}

func convert(fahrenheit: Double) -> Double {
    return (fahrenheit - 32) * 5/9
}

print(convert(42))              // 42.0
print(convert("100") ?? -1)    // 100
print(convert(celsius: 100))   // 212.0
print(convert(fahrenheit: 212)) // 100.0
```

---

## 6.14 Throwing Functions (Error Handling)

Throwing Functions ช่วยในการจัดการข้อผิดพลาดอย่างปลอดภัย

### สร้าง Error Type

```swift
enum NetworkError: Error {
    case invalidURL
    case noData
    case timeout
    case serverError(statusCode: Int)
}

enum ValidationError: Error {
    case emptyField(fieldName: String)
    case invalidFormat(expected: String, got: String)
    case outOfRange(value: Int, min: Int, max: Int)
}
```

### Throwing Function พื้นฐาน

```swift
func validateAge(_ age: Int) throws -> Bool {
    guard age >= 0 else {
        throw ValidationError.outOfRange(value: age, min: 0, max: 150)
    }
    guard age <= 150 else {
        throw ValidationError.outOfRange(value: age, min: 0, max: 150)
    }
    return age >= 18
}

// ใช้ try
do {
    let isAdult = try validateAge(25)
    print("ผู้ใหญ่: \(isAdult)")
    
    let invalid = try validateAge(-5)  // จะ throw error
    print("ผลลัพธ์: \(invalid)")
} catch ValidationError.outOfRange(let value, let min, let max) {
    print("ข้อผิดพลาด: ค่า \(value) ไม่อยู่ในช่วง \(min)-\(max)")
} catch {
    print("ข้อผิดพลาดอื่น: \(error)")
}
```

### Multiple Throws และ Catch

```swift
func parseUserData(_ data: [String: Any]) throws -> (name: String, age: Int) {
    guard let name = data["name"] as? String, !name.isEmpty else {
        throw ValidationError.emptyField(fieldName: "name")
    }
    
    guard let ageString = data["age"] as? String else {
        throw ValidationError.invalidFormat(expected: "String", got: "other")
    }
    
    guard let age = Int(ageString) else {
        throw ValidationError.invalidFormat(expected: "number", got: ageString)
    }
    
    guard age >= 0 && age <= 150 else {
        throw ValidationError.outOfRange(value: age, min: 0, max: 150)
    }
    
    return (name: name, age: age)
}

let testData: [[String: Any]] = [
    ["name": "สมชาย", "age": "25"],
    ["name": "", "age": "30"],
    ["name": "วิชัย", "age": "abc"],
    ["name": "นภา", "age": "200"]
]

for data in testData {
    do {
        let user = try parseUserData(data)
        print("✓ ชื่อ: \(user.name), อายุ: \(user.age)")
    } catch ValidationError.emptyField(let field) {
        print("✗ ฟิลด์ \(field) ว่างเปล่า")
    } catch ValidationError.invalidFormat(let expected, let got) {
        print("✗ รูปแบบไม่ถูกต้อง: คาดหวัง \(expected) แต่ได้ '\(got)'")
    } catch ValidationError.outOfRange(let value, let min, let max) {
        print("✗ ค่า \(value) เกินช่วง \(min)-\(max)")
    }
}
```

### try? และ try!

```swift
// try? แปลง error เป็น nil
let result1 = try? validateAge(25)      // Optional(true)
let result2 = try? validateAge(-1)      // nil

// try! ใช้เมื่อมั่นใจ 100% ว่าไม่ throw
let result3 = try! validateAge(30)      // true (ระวัง: crash ถ้า throw)
```

---

## 6.15 Rethrowing Functions

Rethrowing Functions ใช้ `rethrows` เมื่อ Function จะ throw เฉพาะถ้า Closure ที่รับมา throw ด้วย

### rethrows พื้นฐาน

```swift
// rethrows: function นี้ throw เฉพาะถ้า closure ที่ส่งมา throw
func performOperation(_ operation: () throws -> Void) rethrows {
    try operation()
}

// ใช้กับ non-throwing closure: ไม่ต้องใช้ try
performOperation {
    print("ทำงานปกติ")
}

// ใช้กับ throwing closure: ต้องใช้ try
try performOperation {
    throw NetworkError.timeout
}
```

### rethrows ใน Standard Library

```swift
// map, filter, forEach ล้วนเป็น rethrows
let numbers = [1, 2, 3, 4, 5]

// map กับ non-throwing closure
let doubled = numbers.map { $0 * 2 }

// map กับ throwing closure
enum ConversionError: Error { case invalid }

let strings = ["1", "2", "abc", "4"]
let converted = try? strings.map { str -> Int in
    guard let n = Int(str) else { throw ConversionError.invalid }
    return n
}
print("แปลงได้: \(converted as Any)")
```

### Custom rethrows

```swift
func transform<T, U>(_ values: [T], using transform: (T) throws -> U) rethrows -> [U] {
    var result: [U] = []
    for value in values {
        result.append(try transform(value))
    }
    return result
}

// ใช้แบบ non-throwing
let squares = transform([1, 2, 3, 4, 5]) { $0 * $0 }
print("กำลังสอง: \(squares)")

// ใช้แบบ throwing
let parsed = try? transform(["10", "20", "30"]) { str -> Int in
    guard let n = Int(str) else { throw ConversionError.invalid }
    return n
}
print("แปลงตัวเลข: \(parsed ?? [])")
```

---

## 6.16 @discardableResult

`@discardableResult` บอก compiler ว่าไม่เป็นไรถ้าไม่ใช้ค่าที่ return กลับมา

### ปัญหาที่ @discardableResult แก้

```swift
// โดยปกติ ถ้า function คืนค่าแต่เราไม่ใช้ จะมี warning
func computeValue() -> Int {
    return 42
}

// นี้จะมี warning: Result of call to 'computeValue()' is unused
// computeValue()

// ถ้าใช้ @discardableResult จะไม่มี warning
@discardableResult
func processAndLog(_ message: String) -> Bool {
    print("[LOG] \(message)")
    return true  // return status แต่ผู้ใช้อาจไม่สนใจ
}

processAndLog("ทำงานสำเร็จ")  // ไม่มี warning แม้ไม่ใช้ return value
let success = processAndLog("ตรวจสอบข้อมูล")  // ใช้ return value ก็ได้
print("สำเร็จหรือไม่: \(success)")
```

### ใช้กับ Builder Pattern

```swift
class QueryBuilder {
    private var conditions: [String] = []
    private var tableName: String = ""
    
    @discardableResult
    func from(_ table: String) -> QueryBuilder {
        tableName = table
        return self
    }
    
    @discardableResult
    func where(_ condition: String) -> QueryBuilder {
        conditions.append(condition)
        return self
    }
    
    func build() -> String {
        var query = "SELECT * FROM \(tableName)"
        if !conditions.isEmpty {
            query += " WHERE " + conditions.joined(separator: " AND ")
        }
        return query
    }
}

let query = QueryBuilder()
    .from("users")
    .where("age > 18")
    .where("status = 'active'")
    .build()

print(query)
```

---

## 6.17 @autoclosure

`@autoclosure` ห่อ Expression เป็น Closure โดยอัตโนมัติ ทำให้โค้ดอ่านง่ายขึ้น

### @autoclosure พื้นฐาน

```swift
// ปกติต้องส่ง closure
func logIfDebug(_ message: () -> String, debug: Bool = false) {
    if debug {
        print("[DEBUG] \(message())")
    }
}

logIfDebug({ "ข้อมูล debug" }, debug: true)

// ด้วย @autoclosure ไม่ต้องใส่ { }
func logIfDebugAuto(_ message: @autoclosure () -> String, debug: Bool = false) {
    if debug {
        print("[DEBUG] \(message())")
    }
}

logIfDebugAuto("ข้อมูล debug", debug: true)  // ดูเรียบง่ายกว่า
```

### @autoclosure กับ Lazy Evaluation

```swift
// ข้อดีสำคัญ: expression จะถูก evaluate เฉพาะเมื่อจำเป็น
func expensiveCalculation() -> Int {
    print("กำลังคำนวณหนัก...")
    return (1...1000).reduce(0, +)
}

func useValueIf(_ condition: Bool, value: @autoclosure () -> Int) -> Int? {
    guard condition else { return nil }
    return value()  // คำนวณเฉพาะเมื่อ condition เป็น true
}

// expensiveCalculation จะไม่ถูกเรียกเพราะ condition เป็น false
let result1 = useValueIf(false, value: expensiveCalculation())
print("ผลลัพธ์: \(result1 as Any)")  // nil

// expensiveCalculation จะถูกเรียก
let result2 = useValueIf(true, value: expensiveCalculation())
print("ผลลัพธ์: \(result2 as Any)")  // Optional(500500)
```

### @autoclosure ใน Standard Library

```swift
// assert, precondition ใช้ @autoclosure
assert(1 + 1 == 2, "คณิตศาสตร์พัง!")

// ?? operator คือ @autoclosure
var optionalName: String? = nil
let name = optionalName ?? computeDefaultName()  // computeDefaultName เรียกเฉพาะเมื่อ nil

func computeDefaultName() -> String {
    print("คำนวณชื่อเริ่มต้น...")
    return "ผู้ใช้งานทั่วไป"
}
```

---

## 6.18 @escaping vs Non-escaping

Closure ที่ส่งเป็น Parameter มีสองประเภท: escaping และ non-escaping

### Non-escaping (ค่าเริ่มต้น)

```swift
// Non-escaping: closure ถูกเรียกภายใน function เท่านั้น
func performSync(_ action: () -> Void) {
    print("ก่อนทำงาน")
    action()  // เรียกทันที
    print("หลังทำงาน")
}

performSync {
    print("กำลังทำงาน")
}
// Output:
// ก่อนทำงาน
// กำลังทำงาน
// หลังทำงาน
```

### @escaping Closure

```swift
// @escaping: closure อาจถูกเรียกหลังจาก function return แล้ว
var storedClosures: [() -> Void] = []

func store(_ action: @escaping () -> Void) {
    storedClosures.append(action)  // เก็บไว้ใช้ทีหลัง
}

store { print("Closure 1") }
store { print("Closure 2") }
store { print("Closure 3") }

// เรียกทีหลัง
for closure in storedClosures {
    closure()
}
```

### @escaping กับ Async Operations

```swift
// Callback pattern (ก่อนมี async/await)
func fetchDataFromServer(completion: @escaping (String?, Error?) -> Void) {
    // จำลอง async operation
    DispatchQueue.main.asyncAfter(deadline: .now() + 1.0) {
        let data = "ข้อมูลจาก Server"
        completion(data, nil)
    }
}

fetchDataFromServer { data, error in
    if let data = data {
        print("ได้รับข้อมูล: \(data)")
    }
}
```

### ความแตกต่างในทางปฏิบัติ

```swift
class DataProcessor {
    var result = ""
    
    // @escaping ต้องใช้ self อย่างชัดเจน (ป้องกัน retain cycle)
    func processAsync(_ handler: @escaping (String) -> Void) {
        DispatchQueue.main.async {
            handler(self.result)  // ต้องพิมพ์ self อย่างชัดเจน
        }
    }
    
    // Non-escaping ไม่ต้องใช้ self
    func processSync(_ handler: (String) -> Void) {
        handler(result)  // ไม่ต้องพิมพ์ self
    }
}
```

---

## 6.19 Recursive Functions

Recursive Function คือ Function ที่เรียกตัวเองซ้ำๆ

### Recursive Function พื้นฐาน

```swift
// Factorial
func factorial(_ n: Int) -> Int {
    if n <= 1 { return 1 }
    return n * factorial(n - 1)
}

for i in 0...8 {
    print("\(i)! = \(factorial(i))")
}
```

### Fibonacci Recursive

```swift
// Naive Fibonacci (ช้าเพราะคำนวณซ้ำ)
func fibNaive(_ n: Int) -> Int {
    if n <= 1 { return n }
    return fibNaive(n - 1) + fibNaive(n - 2)
}

// Fibonacci with Memoization (เร็วขึ้นมาก)
var memo: [Int: Int] = [:]
func fibMemo(_ n: Int) -> Int {
    if n <= 1 { return n }
    if let cached = memo[n] { return cached }
    let result = fibMemo(n - 1) + fibMemo(n - 2)
    memo[n] = result
    return result
}

print("Fibonacci(10) = \(fibMemo(10))")  // 55
print("Fibonacci(20) = \(fibMemo(20))")  // 6765
```

### Tree Traversal

```swift
// Binary Tree
class TreeNode {
    var value: Int
    var left: TreeNode?
    var right: TreeNode?
    
    init(_ value: Int) {
        self.value = value
    }
}

// Inorder traversal (Left -> Root -> Right)
func inorderTraversal(_ node: TreeNode?) -> [Int] {
    guard let node = node else { return [] }
    return inorderTraversal(node.left) + [node.value] + inorderTraversal(node.right)
}

// สร้าง tree: 
//       5
//      / \
//     3   7
//    / \   \
//   1   4   9
let root = TreeNode(5)
root.left = TreeNode(3)
root.right = TreeNode(7)
root.left?.left = TreeNode(1)
root.left?.right = TreeNode(4)
root.right?.right = TreeNode(9)

let sorted = inorderTraversal(root)
print("Inorder: \(sorted)")  // [1, 3, 4, 5, 7, 9]
```

### Power Set

```swift
// หา Power Set ของ Array
func powerSet<T>(_ array: [T]) -> [[T]] {
    if array.isEmpty { return [[]] }
    
    let first = array[0]
    let rest = Array(array.dropFirst())
    let subsetsWithoutFirst = powerSet(rest)
    let subsetsWithFirst = subsetsWithoutFirst.map { [first] + $0 }
    
    return subsetsWithoutFirst + subsetsWithFirst
}

let ps = powerSet([1, 2, 3])
print("Power Set ของ [1,2,3]:")
for subset in ps.sorted(by: { $0.count < $1.count }) {
    print("  \(subset)")
}
```

---

## 6.20 Tail Recursion

Tail Recursion คือ Recursive Call ที่เป็น operation สุดท้ายของ Function ช่วยป้องกัน Stack Overflow

### Tail Recursive Factorial

```swift
// Non-tail recursive: ต้องรอผลจาก recursive call ก่อน
func factorialNonTail(_ n: Int) -> Int {
    if n <= 1 { return 1 }
    return n * factorialNonTail(n - 1)  // ยังต้องทำ * n อยู่
}

// Tail recursive: recursive call เป็น operation สุดท้าย
func factorialTail(_ n: Int, accumulator: Int = 1) -> Int {
    if n <= 1 { return accumulator }
    return factorialTail(n - 1, accumulator: n * accumulator)  // สุดท้ายแล้ว
}

print(factorialTail(10))  // 3628800
print(factorialTail(15))  // 1307674368000
```

### Tail Recursive Sum

```swift
func sumTo(_ n: Int, acc: Int = 0) -> Int {
    if n <= 0 { return acc }
    return sumTo(n - 1, acc: acc + n)
}

print("ผลรวม 1 ถึง 100: \(sumTo(100))")  // 5050
```

### Iterative vs Recursive Comparison

```swift
// Iterative (เร็วกว่าใน Swift เพราะ optimization)
func sumIterative(_ n: Int) -> Int {
    var total = 0
    for i in 1...n {
        total += i
    }
    return total
}

// Recursive (อ่านง่ายกว่า แต่ใช้ stack)
func sumRecursive(_ n: Int) -> Int {
    if n <= 0 { return 0 }
    return n + sumRecursive(n - 1)
}

print("Iterative: \(sumIterative(10))")   // 55
print("Recursive: \(sumRecursive(10))")  // 55
```

---

## 6.21 Pure Functions

Pure Functions คือ Functions ที่ output ขึ้นอยู่กับ input เท่านั้น และไม่มี Side Effects

### ลักษณะของ Pure Functions

```swift
// Pure Function: ผลลัพธ์เดิมเสมอสำหรับ input เดิม ไม่มี side effects
func add(_ a: Int, _ b: Int) -> Int {
    return a + b
}

func square(_ x: Double) -> Double {
    return x * x
}

func fullName(first: String, last: String) -> String {
    return "\(first) \(last)"
}

// เรียกกี่ครั้งก็ได้ผลเหมือนเดิม
print(add(3, 4))      // 7 เสมอ
print(add(3, 4))      // 7 เสมอ
print(square(5.0))    // 25.0 เสมอ
```

### Non-Pure Functions (มี Side Effects)

```swift
var globalCounter = 0  // global state

// ไม่ pure: เปลี่ยน global state
func incrementCounter() -> Int {
    globalCounter += 1
    return globalCounter
}

print(incrementCounter())  // 1
print(incrementCounter())  // 2 (ผลลัพธ์ต่างกัน!)

// ไม่ pure: ขึ้นอยู่กับ external state
func getCurrentHour() -> Int {
    return Calendar.current.component(.hour, from: Date())
}
```

### ประโยชน์ของ Pure Functions

```swift
// Pure functions ทดสอบง่าย
func calculateTax(amount: Double, rate: Double) -> Double {
    return amount * rate
}

// Test cases ง่ายมาก
assert(calculateTax(amount: 100, rate: 0.07) == 7.0)
assert(calculateTax(amount: 1000, rate: 0.10) == 100.0)

// Pure functions compose ได้ง่าย
func applyDiscount(to price: Double, discount: Double) -> Double {
    return price * (1 - discount)
}

func addTax(to price: Double, rate: Double) -> Double {
    return price + calculateTax(amount: price, rate: rate)
}

// Chain ได้ง่าย
let originalPrice = 1000.0
let finalPrice = addTax(to: applyDiscount(to: originalPrice, discount: 0.2), rate: 0.07)
print("ราคาสุดท้าย: \(String(format: "%.2f", finalPrice))")
```

---

## 6.22 Side Effects

Side Effects คือผลกระทบที่เกิดนอกเหนือจากการคืนค่า

### ประเภทของ Side Effects

```swift
// 1. การเขียน I/O
func logMessage(_ msg: String) {
    print("[LOG]: \(msg)")  // side effect: เขียน console
}

// 2. การแก้ไข Global State
var appState: [String: Any] = [:]

func updateState(key: String, value: Any) {
    appState[key] = value  // side effect: แก้ไข global state
}

// 3. การเปลี่ยนแปลง Input (In-Out)
func normalize(_ array: inout [Int]) {
    for i in array.indices {
        array[i] = max(0, min(100, array[i]))  // side effect: เปลี่ยน input
    }
}

// 4. Exception/Error Throwing
func validateAge2(_ age: Int) throws -> Int {
    guard age > 0 else { throw ValidationError.outOfRange(value: age, min: 0, max: 150) }
    return age  // throwing เป็น side effect หนึ่ง
}
```

### การจัดการ Side Effects อย่างดี

```swift
// แยก pure logic ออกจาก side effects
// Pure: คำนวณ discount
func calculateDiscount(price: Double, memberLevel: String) -> Double {
    switch memberLevel {
    case "Gold":   return price * 0.80
    case "Silver": return price * 0.90
    default:       return price
    }
}

// Impure (แต่จัดการได้): บันทึก + คำนวณ
func processOrder(price: Double, memberLevel: String) {
    let finalPrice = calculateDiscount(price: price, memberLevel: memberLevel)  // pure
    
    // Side effects อยู่ที่นี่ชัดเจน
    print("ราคาสุดท้าย: \(finalPrice)")  // I/O
    updateState(key: "lastOrder", value: finalPrice)  // state mutation
    logMessage("Order processed: \(finalPrice)")  // logging
}
```

---

## 6.23 แบบฝึกหัดพร้อมเฉลย

### แบบฝึกหัดที่ 1: Calculator Functions

```swift
// โจทย์: สร้าง Scientific Calculator อย่างง่าย

import Foundation

enum MathError: Error {
    case divisionByZero
    case negativeSqrt
    case invalidInput
}

// Basic Operations
func add(_ a: Double, _ b: Double) -> Double { a + b }
func subtract(_ a: Double, _ b: Double) -> Double { a - b }
func multiply(_ a: Double, _ b: Double) -> Double { a * b }
func divide(_ a: Double, by b: Double) throws -> Double {
    guard b != 0 else { throw MathError.divisionByZero }
    return a / b
}

// Advanced Operations
func power(_ base: Double, exponent: Int) -> Double {
    if exponent == 0 { return 1 }
    if exponent < 0 { return 1.0 / power(base, exponent: -exponent) }
    return (1...exponent).reduce(1.0) { result, _ in result * base }
}

func squareRoot(_ n: Double) throws -> Double {
    guard n >= 0 else { throw MathError.negativeSqrt }
    return sqrt(n)
}

// ทดสอบ
do {
    print("5 + 3 = \(add(5, 3))")
    print("10 - 4 = \(subtract(10, 4))")
    print("6 × 7 = \(multiply(6, 7))")
    print("15 ÷ 3 = \(try divide(15, by: 3))")
    print("2^10 = \(power(2, exponent: 10))")
    print("√144 = \(try squareRoot(144))")
    print("√(-1) = ", terminator: "")
    print(try squareRoot(-1))
} catch MathError.divisionByZero {
    print("หารด้วย 0 ไม่ได้")
} catch MathError.negativeSqrt {
    print("ไม่สามารถหาค่า sqrt ของจำนวนลบได้")
} catch {
    print("ข้อผิดพลาด: \(error)")
}
```

### แบบฝึกหัดที่ 2: String Processing Functions

```swift
// โจทย์: สร้าง String utility functions

// 1. ตรวจสอบ Palindrome
func isPalindrome(_ str: String) -> Bool {
    let cleaned = str.lowercased().filter { $0.isLetter || $0.isNumber }
    return cleaned == String(cleaned.reversed())
}

// 2. นับตัวอักษรแต่ละประเภท
func analyzeString(_ str: String) -> (letters: Int, digits: Int, spaces: Int, others: Int) {
    var letters = 0, digits = 0, spaces = 0, others = 0
    for char in str {
        if char.isLetter { letters += 1 }
        else if char.isNumber { digits += 1 }
        else if char.isWhitespace { spaces += 1 }
        else { others += 1 }
    }
    return (letters, digits, spaces, others)
}

// 3. Title Case
func titleCase(_ str: String) -> String {
    return str.split(separator: " ")
        .map { word in
            let first = word.prefix(1).uppercased()
            let rest = word.dropFirst().lowercased()
            return first + rest
        }
        .joined(separator: " ")
}

// 4. Word Frequency
func wordFrequency(_ text: String) -> [String: Int] {
    return text.lowercased()
        .split(separator: " ")
        .reduce(into: [String: Int]()) { freq, word in
            freq[String(word), default: 0] += 1
        }
}

// ทดสอบ
print("isPalindrome:")
["racecar", "hello", "A man a plan a canal Panama"].forEach {
    print("  \"\($0)\": \(isPalindrome($0))")
}

print("\nanalyzeString:")
let sample = "Hello World! 123"
let analysis = analyzeString(sample)
print("  ตัวอักษร: \(analysis.letters)")
print("  ตัวเลข: \(analysis.digits)")
print("  ช่องว่าง: \(analysis.spaces)")

print("\ntitleCase:")
print("  \(titleCase("hello world from swift"))")

print("\nwordFrequency:")
let text2 = "the quick brown fox jumps over the lazy dog the fox"
let freq = wordFrequency(text2)
let sortedFreq = freq.sorted { $0.value > $1.value }.prefix(5)
for (word, count) in sortedFreq {
    print("  \"\(word)\": \(count)")
}
```

### แบบฝึกหัดที่ 3: Higher-Order Functions

```swift
// โจทย์: สร้าง Functional Pipeline

// สร้าง function ที่ compose functions เข้าด้วยกัน
func compose<T, U, V>(_ f: @escaping (U) -> V, _ g: @escaping (T) -> U) -> (T) -> V {
    return { x in f(g(x)) }
}

// Pipeline operator
func pipe<T, U>(_ value: T, through transform: (T) -> U) -> U {
    return transform(value)
}

// ตัวอย่างการใช้งาน
let double: (Int) -> Int = { $0 * 2 }
let addOne: (Int) -> Int = { $0 + 1 }
let square2: (Int) -> Int = { $0 * $0 }

let doubleAndSquare = compose(square2, double)
print("doubleAndSquare(5) = \(doubleAndSquare(5))")  // 100

// สร้าง Pipeline สำหรับ Text Processing
let processText = compose(
    { (s: String) in s.trimmingCharacters(in: .whitespaces) },
    { (s: String) in s.lowercased() }
)

let messy = "  HELLO WORLD  "
print("Processed: '\(processText(messy))'")
```

### แบบฝึกหัดที่ 4: Recursive Data Structures

```swift
// โจทย์: JSON-like Data Structure ด้วย Recursion

indirect enum JSONValue {
    case null
    case bool(Bool)
    case number(Double)
    case string(String)
    case array([JSONValue])
    case object([String: JSONValue])
}

func prettyPrint(_ json: JSONValue, indent: Int = 0) -> String {
    let spaces = String(repeating: "  ", count: indent)
    
    switch json {
    case .null:
        return "null"
    case .bool(let b):
        return b ? "true" : "false"
    case .number(let n):
        return n.truncatingRemainder(dividingBy: 1) == 0 ? "\(Int(n))" : "\(n)"
    case .string(let s):
        return "\"\(s)\""
    case .array(let items):
        if items.isEmpty { return "[]" }
        let elements = items.map { "\(spaces)  \(prettyPrint($0, indent: indent + 1))" }
        return "[\n\(elements.joined(separator: ",\n"))\n\(spaces)]"
    case .object(let dict):
        if dict.isEmpty { return "{}" }
        let entries = dict.keys.sorted().map { key in
            "\(spaces)  \"\(key)\": \(prettyPrint(dict[key]!, indent: indent + 1))"
        }
        return "{\n\(entries.joined(separator: ",\n"))\n\(spaces)}"
    }
}

let person: JSONValue = .object([
    "name": .string("สมชาย ใจดี"),
    "age": .number(25),
    "isStudent": .bool(false),
    "courses": .array([.string("Swift"), .string("Python"), .string("SQL")]),
    "address": .object([
        "city": .string("กรุงเทพ"),
        "country": .string("ไทย")
    ])
])

print(prettyPrint(person))
```

---

## 6.24 Real-World Examples

### ตัวอย่าง 1: Data Validation Library

```swift
// Validation System
typealias Validator<T> = (T) -> Result<T, ValidationError>

func makeRequiredValidator<T>() -> Validator<T?> {
    return { value in
        guard let v = value else {
            return .failure(ValidationError.emptyField(fieldName: "value"))
        }
        return .success(v)
    }
}

func makeRangeValidator(min: Int, max: Int) -> Validator<Int> {
    return { value in
        guard value >= min && value <= max else {
            return .failure(ValidationError.outOfRange(value: value, min: min, max: max))
        }
        return .success(value)
    }
}

func makeLengthValidator(minLength: Int, maxLength: Int) -> Validator<String> {
    return { str in
        guard str.count >= minLength else {
            return .failure(ValidationError.invalidFormat(
                expected: "อย่างน้อย \(minLength) ตัวอักษร",
                got: str
            ))
        }
        guard str.count <= maxLength else {
            return .failure(ValidationError.invalidFormat(
                expected: "ไม่เกิน \(maxLength) ตัวอักษร",
                got: str
            ))
        }
        return .success(str)
    }
}

// ใช้งาน
let ageValidator = makeRangeValidator(min: 0, max: 150)
let passwordValidator = makeLengthValidator(minLength: 8, maxLength: 20)

let tests = [25, -1, 200, 30]
for age in tests {
    switch ageValidator(age) {
    case .success(let validAge):
        print("อายุ \(validAge) ถูกต้อง")
    case .failure(let error):
        print("อายุ \(age) ไม่ถูกต้อง: \(error)")
    }
}
```

### ตัวอย่าง 2: Functional Composition

```swift
// Functional utilities สำหรับ Data Processing
func memoize<Input: Hashable, Output>(_ function: @escaping (Input) -> Output) -> (Input) -> Output {
    var cache: [Input: Output] = [:]
    return { input in
        if let cached = cache[input] { return cached }
        let result = function(input)
        cache[input] = result
        return result
    }
}

// ใช้งาน memoize กับ Fibonacci
let memoFib: (Int) -> Int = memoize { n in
    if n <= 1 { return n }
    let fib: (Int) -> Int = memoize { n in n <= 1 ? n : 0 }
    return n <= 1 ? n : memoFib(n - 1) + memoFib(n - 2)
}

// Curry function
func curry<A, B, C>(_ f: @escaping (A, B) -> C) -> (A) -> (B) -> C {
    return { a in { b in f(a, b) } }
}

let curriedAdd = curry(add)
let addFive = curriedAdd(5)
print("5 + 3 = \(addFive(3))")
print("5 + 10 = \(addFive(10))")

// Partial Application
func multiply2(_ a: Int, _ b: Int) -> Int { a * b }
let triple = curry(multiply2)(3)
print("triple(7) = \(triple(7))")    // 21
print("triple(10) = \(triple(10))")  // 30
```

---

## 6.25 สรุป

ในบทนี้เราได้เรียนรู้ Functions ใน Swift อย่างครอบคลุม:

### สรุปสิ่งที่เรียนรู้

| Feature | คำอธิบาย |
|---|---|
| Function Syntax | `func name(params) -> ReturnType` |
| Argument Labels | แยก external label และ internal name |
| Default Values | parameter ที่มีค่าเริ่มต้น |
| Variadic | `...` รับหลายค่าในรูป Array |
| In-out | `inout` แก้ไขค่าภายนอก Function |
| Tuple Return | คืนหลายค่าพร้อมกัน |
| Function Types | Functions มี Type เหมือน Value |
| First-Class | Functions เก็บ, ส่ง, คืนได้ |
| Higher-Order | map, filter, reduce |
| Throwing | `throws`, `try`, `catch` |
| @discardableResult | ละการใช้ return value |
| @autoclosure | ห่อ expression เป็น Closure |
| @escaping | Closure ที่อาจเรียกทีหลัง |
| Recursive | Function เรียกตัวเอง |
| Pure Functions | ไม่มี Side Effects |

### หลักการสำคัญ

1. **Single Responsibility**: Function ควรทำหน้าที่เดียว
2. **Pure When Possible**: Function ที่ Pure Test ง่ายและ Compose ได้
3. **Meaningful Names**: ตั้งชื่อให้สื่อความหมาย อ่านออกเสียงได้
4. **Argument Labels**: ทำให้ API อ่านเป็นธรรมชาติ
5. **Error Handling**: ใช้ throws/try/catch สำหรับ operations ที่อาจล้มเหลว
6. **Function Composition**: สร้าง complex behavior จาก simple functions

### บทถัดไป

ในบทที่ 7 เราจะเรียนรู้ **Closures** ซึ่งเป็นส่วนต่อขยายของ Functions ที่ทรงพลังยิ่งขึ้น รวมถึง Closure Syntax ที่กระชับ, Capture List, และการใช้งานใน Async Programming

---

*จบบทที่ 6: Functions*
