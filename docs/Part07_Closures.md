# ส่วนที่ 7: Closures ใน Swift

## บทนำ

Closure เป็นหนึ่งในแนวคิดที่สำคัญและทรงพลังที่สุดใน Swift ซึ่งช่วยให้เราเขียนโค้ดที่กระชับ อ่านง่าย และมีประสิทธิภาพ ในบทนี้เราจะเรียนรู้เกี่ยวกับ Closure อย่างละเอียดตั้งแต่พื้นฐานจนถึงการใช้งานขั้นสูง

---

## 7.1 Closure คืออะไร?

**Closure** คือบล็อกโค้ดที่สามารถเก็บและส่งผ่านได้เหมือนกับ object ธรรมดา มันสามารถ "จับ" (capture) ค่าจาก context ที่มันถูกกำหนด (defined) และใช้ค่าเหล่านั้นได้แม้ว่า scope ดั้งเดิมจะหมดอายุไปแล้ว

ใน Swift, Closure มีสามรูปแบบหลัก:

1. **Global functions** - ฟังก์ชันที่มีชื่อ และไม่ capture ค่าใดๆ
2. **Nested functions** - ฟังก์ชันที่อยู่ภายในฟังก์ชันอื่น และสามารถ capture ค่าจาก enclosing function
3. **Closure expressions** - ฟังก์ชันที่ไม่มีชื่อ เขียนในรูปแบบสั้น และสามารถ capture ค่าจาก context รอบๆ

### ความแตกต่างระหว่าง Closure และ Function

```swift
// Function ปกติ
func add(a: Int, b: Int) -> Int {
    return a + b
}

// Closure ที่ทำสิ่งเดียวกัน
let addClosure = { (a: Int, b: Int) -> Int in
    return a + b
}

// เรียกใช้งาน
print(add(a: 3, b: 4))       // 7
print(addClosure(3, 4))       // 7
```

### ทำไมต้องใช้ Closure?

Closure มีประโยชน์ในหลายสถานการณ์:

1. **Callback functions** - ใช้เป็น completion handler ใน async operations
2. **Higher-order functions** - ส่งเป็น parameter ให้ฟังก์ชันอื่น เช่น `map`, `filter`, `reduce`
3. **Event handlers** - จัดการ events ใน UI
4. **Lazy evaluation** - เลื่อนการคำนวณออกไปจนกว่าจะต้องการ
5. **Encapsulation** - ห่อหุ้มโค้ดที่เกี่ยวข้องกัน

---

## 7.2 Closure Syntax

### รูปแบบพื้นฐาน

```swift
{ (parameters) -> returnType in
    statements
}
```

### ตัวอย่างต่างๆ

```swift
// Closure ที่ไม่มี parameter และไม่คืนค่า
let greet = {
    print("สวัสดีครับ!")
}
greet() // สวัสดีครับ!

// Closure ที่มี parameter
let greetPerson = { (name: String) in
    print("สวัสดี \(name)!")
}
greetPerson("สมชาย") // สวัสดี สมชาย!

// Closure ที่คืนค่า
let square = { (number: Int) -> Int in
    return number * number
}
print(square(5)) // 25

// Closure ที่มีหลาย parameter
let multiply = { (a: Int, b: Int) -> Int in
    return a * b
}
print(multiply(4, 6)) // 24
```

### Type Annotation ใน Closure

```swift
// ระบุ type อย่างชัดเจน
var operation: (Int, Int) -> Int

// กำหนดค่าให้กับ closure variable
operation = { (a: Int, b: Int) -> Int in
    return a + b
}
print(operation(10, 20)) // 30

// เปลี่ยน closure
operation = { (a: Int, b: Int) -> Int in
    return a * b
}
print(operation(10, 20)) // 200
```

---

## 7.3 Closure Expressions

Closure expressions คือวิธีการเขียน closure ที่กระชับและอ่านง่าย Swift มีหลายวิธีในการย่อโค้ด closure

### การเปรียบเทียบ: จาก verbose ไปหา concise

```swift
let numbers = [5, 2, 8, 1, 9, 3, 7, 4, 6]

// วิธีที่ 1: ใช้ฟังก์ชันธรรมดา
func sortAscending(_ a: Int, _ b: Int) -> Bool {
    return a < b
}
let sorted1 = numbers.sorted(by: sortAscending)
print(sorted1) // [1, 2, 3, 4, 5, 6, 7, 8, 9]

// วิธีที่ 2: ใช้ closure expression แบบเต็ม
let sorted2 = numbers.sorted(by: { (a: Int, b: Int) -> Bool in
    return a < b
})
print(sorted2) // [1, 2, 3, 4, 5, 6, 7, 8, 9]

// วิธีที่ 3: Type inference - Swift รู้ type จาก context
let sorted3 = numbers.sorted(by: { a, b in
    return a < b
})
print(sorted3) // [1, 2, 3, 4, 5, 6, 7, 8, 9]

// วิธีที่ 4: Implicit return - return เดียว ไม่ต้องเขียน return
let sorted4 = numbers.sorted(by: { a, b in a < b })
print(sorted4) // [1, 2, 3, 4, 5, 6, 7, 8, 9]

// วิธีที่ 5: Shorthand argument names
let sorted5 = numbers.sorted(by: { $0 < $1 })
print(sorted5) // [1, 2, 3, 4, 5, 6, 7, 8, 9]

// วิธีที่ 6: Operator method
let sorted6 = numbers.sorted(by: <)
print(sorted6) // [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### Inferring Type from Context

Swift's type system สามารถ infer type ของ closure จาก context ได้

```swift
// Array ของ String
let fruits = ["มะม่วง", "แอปเปิ้ล", "กล้วย", "ส้ม"]

// Swift รู้ว่า element เป็น String จาก context
let upperFruits = fruits.map({ fruit in
    return fruit.uppercased()
})
print(upperFruits)

// Array ของ Int
let scores = [85, 92, 78, 95, 60, 88]

// Swift รู้ว่า element เป็น Int
let passingScores = scores.filter({ score in
    return score >= 70
})
print(passingScores) // [85, 92, 78, 95, 88]
```

### Implicit Returns

เมื่อ closure body มีเพียง expression เดียว สามารถละ `return` ได้

```swift
// มี return keyword
let doubled = [1, 2, 3, 4, 5].map({ number -> Int in
    return number * 2
})

// ไม่มี return keyword (implicit return)
let doubledShort = [1, 2, 3, 4, 5].map({ number in number * 2 })

print(doubled)      // [2, 4, 6, 8, 10]
print(doubledShort) // [2, 4, 6, 8, 10]
```

---

## 7.4 Trailing Closure Syntax

เมื่อ closure เป็น parameter สุดท้ายของฟังก์ชัน เราสามารถเขียน closure ไว้ข้างนอกวงเล็บได้

### รูปแบบพื้นฐาน

```swift
// รูปแบบปกติ
let result1 = [1, 2, 3].map({ $0 * 2 })

// Trailing closure syntax
let result2 = [1, 2, 3].map { $0 * 2 }

print(result1) // [2, 4, 6]
print(result2) // [2, 4, 6]
```

### เมื่อ closure เป็น parameter เดียว

```swift
// เมื่อ closure เป็น argument เดียว วงเล็บ () สามารถละได้ทั้งหมด
let numbers = [1, 2, 3, 4, 5]

// แบบปกติ
let evens1 = numbers.filter({ $0 % 2 == 0 })

// Trailing closure
let evens2 = numbers.filter { $0 % 2 == 0 }

print(evens1) // [2, 4]
print(evens2) // [2, 4]
```

### Multiple Trailing Closures (Swift 5.3+)

เมื่อฟังก์ชันมี closure หลายตัวเป็น parameter

```swift
// ตัวอย่าง custom function ที่รับ closure หลายตัว
func performOperations(
    onSuccess: () -> Void,
    onFailure: (Error) -> Void,
    onComplete: () -> Void
) {
    // จำลองการทำงาน
    onSuccess()
    onComplete()
}

// Multiple trailing closures syntax
performOperations {
    print("สำเร็จ!")
} onFailure: { error in
    print("ผิดพลาด: \(error)")
} onComplete: {
    print("เสร็จสิ้น!")
}
```

### Trailing Closure ใน Animation

```swift
// ตัวอย่างใน UIKit (iOS)
// UIView.animate(withDuration: 0.3) {
//     self.view.alpha = 0.0
// }

// ตัวอย่างใน SwiftUI
// withAnimation(.easeInOut(duration: 0.3)) {
//     isVisible = false
// }
```

### Trailing Closure กับ if/while/for

```swift
// ใช้ trailing closure กับ forEach
[1, 2, 3, 4, 5].forEach { number in
    print("ตัวเลข: \(number)")
}

// ใช้กับ sort
var mutableNumbers = [5, 2, 8, 1, 9]
mutableNumbers.sort { $0 > $1 }
print(mutableNumbers) // [9, 8, 5, 2, 1]
```

---

## 7.5 Shorthand Argument Names ($0, $1, $2)

Swift ให้ shorthand argument names สำหรับ inline closures เพื่อให้เขียนโค้ดได้กระชับยิ่งขึ้น

### รูปแบบ Shorthand

```swift
// $0 = argument แรก
// $1 = argument ที่สอง
// $2 = argument ที่สาม
// และต่อไปเรื่อยๆ

let numbers = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3]

// ใช้ shorthand
let doubled = numbers.map { $0 * 2 }
print(doubled) // [6, 2, 8, 2, 10, 18, 4, 12, 10, 6]

let sorted = numbers.sorted { $0 < $1 }
print(sorted) // [1, 1, 2, 3, 3, 4, 5, 5, 6, 9]

let evenNumbers = numbers.filter { $0 % 2 == 0 }
print(evenNumbers) // [4, 2, 6]
```

### Shorthand กับ String

```swift
let words = ["hello", "world", "swift", "programming"]

// ใช้ $0 กับ String methods
let capitalized = words.map { $0.capitalized }
print(capitalized) // ["Hello", "World", "Swift", "Programming"]

let longWords = words.filter { $0.count > 4 }
print(longWords) // ["hello", "world", "swift", "programming"]

// sort ตามความยาว
let sortedByLength = words.sorted { $0.count < $1.count }
print(sortedByLength) // ["hello", "world", "swift", "programming"]
```

### เมื่อควรใช้ Shorthand

```swift
// ดี: เมื่อ closure ง่ายและอ่านเข้าใจได้
let squares = [1, 2, 3, 4, 5].map { $0 * $0 }

// ไม่ดี: เมื่อ closure ซับซ้อนเกินไป
// ควรใช้ชื่อที่อ่านเข้าใจได้แทน
let result = someComplexArray.reduce(0) { accumulator, element in
    // logic ที่ซับซ้อน
    return accumulator + element.value * element.weight
}
```

---

## 7.6 Capturing Values

Closure สามารถ "capture" ค่าจาก context ที่มันถูกกำหนด ซึ่งเป็นคุณสมบัติที่ทรงพลังของ closure

### การ Capture ค่า

```swift
func makeCounter() -> () -> Int {
    var count = 0
    
    // Closure นี้ capture ตัวแปร count
    let counter = {
        count += 1
        return count
    }
    
    return counter
}

let counter1 = makeCounter()
let counter2 = makeCounter()

print(counter1()) // 1
print(counter1()) // 2
print(counter1()) // 3

print(counter2()) // 1 (แต่ละ closure มี count ของตัวเอง)
print(counter2()) // 2
```

### Capture โดยอ้างอิง

```swift
var total = 0

let addToTotal = { (amount: Int) in
    total += amount  // capture total โดยอ้างอิง
}

addToTotal(10)
addToTotal(20)
addToTotal(30)
print(total) // 60 - ค่าเปลี่ยนแปลงผ่าน closure
```

### ตัวอย่าง: Capture ใน Loop

```swift
// ข้อควรระวัง: Closure ใน loop capture ตัวแปร loop variable
var closures: [() -> Int] = []

for i in 0..<5 {
    // Closure capture 'i' โดย reference ใน Swift จะ capture value ณ เวลาที่สร้าง
    let captured = i  // capture i as constant
    closures.append { captured }
}

for closure in closures {
    print(closure(), terminator: " ") // 0 1 2 3 4
}
print()
```

### ตัวอย่างการ Capture ที่ซับซ้อน

```swift
class BankAccount {
    private var balance: Double = 0.0
    
    func makeDeposit() -> (Double) -> Void {
        // Closure capture self.balance
        return { [weak self] amount in
            guard let self = self else { return }
            self.balance += amount
            print("ฝากเงิน \(amount) บาท ยอดเงินคงเหลือ: \(self.balance) บาท")
        }
    }
    
    func makeWithdrawal() -> (Double) -> Bool {
        return { [weak self] amount in
            guard let self = self else { return false }
            if self.balance >= amount {
                self.balance -= amount
                print("ถอนเงิน \(amount) บาท ยอดเงินคงเหลือ: \(self.balance) บาท")
                return true
            } else {
                print("ยอดเงินไม่เพียงพอ!")
                return false
            }
        }
    }
}

let account = BankAccount()
let deposit = account.makeDeposit()
let withdraw = account.makeWithdrawal()

deposit(1000)
deposit(500)
_ = withdraw(200)
_ = withdraw(2000)
```

---

## 7.7 Capture Lists

Capture list ใช้เพื่อกำหนดวิธีที่ closure capture ค่าต่างๆ โดยเฉพาะเมื่อทำงานกับ reference types

### รูปแบบ Capture List

```swift
{ [captureList] (parameters) -> returnType in
    // body
}
```

### [weak self]

ใช้เมื่อต้องการให้ closure ถือ reference แบบ weak ป้องกัน retain cycle

```swift
class ViewController {
    var name = "ViewControllerหลัก"
    
    func fetchData() {
        // จำลอง async operation
        DispatchQueue.main.asyncAfter(deadline: .now() + 1.0) { [weak self] in
            // self อาจเป็น nil ถ้า ViewController ถูก deallocated
            guard let self = self else {
                print("ViewController ถูก deallocated แล้ว")
                return
            }
            print("โหลดข้อมูลสำหรับ \(self.name) เสร็จแล้ว")
        }
    }
    
    deinit {
        print("\(name) ถูก deallocated")
    }
}

var vc: ViewController? = ViewController()
vc?.fetchData()
vc = nil // ViewController ถูก deallocated
```

### [unowned self]

ใช้เมื่อมั่นใจว่า self จะยังคงมีอยู่ตลอดอายุของ closure

```swift
class NetworkManager {
    var baseURL = "https://api.example.com"
    
    func loadData(completion: @escaping (String) -> Void) {
        // จำลอง network call
        DispatchQueue.global().async {
            let data = "ข้อมูลจาก network"
            DispatchQueue.main.async {
                completion(data)
            }
        }
    }
    
    func fetchUserProfile() {
        // ใช้ unowned เมื่อมั่นใจว่า self ยังมีอยู่เมื่อ closure ทำงาน
        loadData { [unowned self] data in
            print("โหลดข้อมูลจาก \(self.baseURL): \(data)")
        }
    }
}
```

### Capture ค่าเฉพาะ

```swift
var x = 10
var y = 20

// Capture x เป็น value ณ เวลาที่สร้าง closure
let closure = { [x] in
    print("x = \(x), y = \(y)")
}

x = 100
y = 200

closure() // x = 10, y = 200 (x ถูก capture เป็น value, y ถูก capture เป็น reference)
```

### Capture หลายค่า

```swift
class DataProcessor {
    var multiplier = 2
    var offset = 10
    
    func processValues(_ values: [Int]) -> [Int] {
        // Capture self แบบ weak และ capture multiplier, offset เป็น constant
        let localMultiplier = multiplier
        let localOffset = offset
        
        return values.map { value in
            value * localMultiplier + localOffset
        }
    }
}

let processor = DataProcessor()
let processed = processor.processValues([1, 2, 3, 4, 5])
print(processed) // [12, 14, 16, 18, 20]
```

---

## 7.8 Escaping Closures

**Escaping closure** คือ closure ที่ถูกเรียกหลังจากที่ฟังก์ชันที่รับมันทำงานเสร็จแล้ว

### @escaping Annotation

```swift
// @escaping บอกว่า closure จะถูกเรียกหลังจากฟังก์ชันนี้ return
func fetchData(completion: @escaping (String) -> Void) {
    DispatchQueue.global().async {
        // ทำงาน async
        let data = "ข้อมูลจาก server"
        
        DispatchQueue.main.async {
            completion(data)  // เรียก closure หลัง fetchData() return แล้ว
        }
    }
}

// การใช้งาน
fetchData { data in
    print("ได้รับข้อมูล: \(data)")
}
```

### ตัวอย่างในชีวิตจริง

```swift
class APIClient {
    var completionHandlers: [(String) -> Void] = []
    
    // จัดเก็บ completion handler ไว้ใช้ทีหลัง
    func addCompletionHandler(_ handler: @escaping (String) -> Void) {
        completionHandlers.append(handler)
    }
    
    func executeAllHandlers(with data: String) {
        for handler in completionHandlers {
            handler(data)
        }
    }
}

let client = APIClient()
client.addCompletionHandler { data in
    print("Handler 1: \(data)")
}
client.addCompletionHandler { data in
    print("Handler 2: \(data)")
}
client.executeAllHandlers(with: "ข้อมูลที่โหลดมา")
```

### Escaping Closure กับ Property

```swift
var storedClosure: (() -> Void)?

// closure จะถูกเก็บไว้ ต้องใช้ @escaping
func storeClosure(_ closure: @escaping () -> Void) {
    storedClosure = closure
}

storeClosure {
    print("Closure ถูกเรียกจาก storedClosure")
}

storedClosure?() // Closure ถูกเรียกจาก storedClosure
```

---

## 7.9 Non-Escaping Closures

**Non-escaping closure** (ค่า default) คือ closure ที่ต้องถูกเรียกภายในฟังก์ชันที่รับมัน

### ตัวอย่าง Non-Escaping

```swift
// ไม่ต้องใส่ @escaping - นี่คือ non-escaping
func performOperation(with value: Int, using operation: (Int) -> Int) -> Int {
    // closure ถูกเรียกที่นี่ ภายใน function
    return operation(value)
}

let result = performOperation(with: 5) { number in
    return number * number
}
print(result) // 25
```

### ประโยชน์ของ Non-Escaping

```swift
// Non-escaping closure ทำให้ Swift สามารถ optimize memory ได้ดีกว่า
// เพราะรู้ว่า closure จะไม่ถูกเก็บไว้หลังฟังก์ชัน return

func apply(_ operations: [(Int) -> Int], to value: Int) -> Int {
    var result = value
    for operation in operations {
        result = operation(result)
    }
    return result
}

let value = apply([
    { $0 + 10 },
    { $0 * 2 },
    { $0 - 5 }
], to: 3)

print(value) // ((3 + 10) * 2) - 5 = 21
```

---

## 7.10 Autoclosures

**Autoclosure** คือ closure ที่ถูกสร้างโดยอัตโนมัติเพื่อห่อหุ้ม expression ที่ส่งเป็น argument

### @autoclosure Annotation

```swift
// ไม่มี autoclosure - ต้องส่ง closure อย่างชัดเจน
func evaluate(condition: () -> Bool) {
    if condition() {
        print("เงื่อนไขเป็นจริง")
    }
}
evaluate(condition: { 2 > 1 })

// มี autoclosure - สามารถส่ง expression ได้เลย
func evaluate2(condition: @autoclosure () -> Bool) {
    if condition() {
        print("เงื่อนไขเป็นจริง")
    }
}
evaluate2(condition: 2 > 1)  // ไม่ต้องมีวงเล็บ {}
```

### ตัวอย่าง: Custom assert

```swift
func myAssert(_ condition: @autoclosure () -> Bool,
              _ message: @autoclosure () -> String = "การยืนยันล้มเหลว",
              file: StaticString = #file,
              line: UInt = #line) {
    if !condition() {
        print("ข้อผิดพลาด: \(message()) ที่ \(file):\(line)")
    }
}

myAssert(1 + 1 == 2)                     // ผ่าน
myAssert(2 + 2 == 5, "คณิตศาสตร์ผิดพลาด") // แสดง error
```

### Autoclosure กับ Lazy Evaluation

```swift
var array = [1, 2, 3, 4, 5]

// removeFirst() จะถูกเรียกเฉพาะเมื่อ condition เป็น true
func firstPositive(condition: @autoclosure () -> Bool, value: @autoclosure () -> Int) -> Int? {
    if condition() {
        return value()
    }
    return nil
}

// removeFirst() ไม่ถูกเรียกเพราะ condition เป็น false
// let result = firstPositive(condition: array.isEmpty, value: array.removeFirst())
```

---

## 7.11 Closures as Function Parameters

Closure สามารถส่งเป็น parameter ให้ฟังก์ชันอื่นได้ ทำให้สามารถ customize พฤติกรรมของฟังก์ชัน

### รูปแบบพื้นฐาน

```swift
// รับ closure เป็น parameter
func transform(_ value: Int, using closure: (Int) -> Int) -> Int {
    return closure(value)
}

let doubled = transform(5, using: { $0 * 2 })
let squared = transform(5, using: { $0 * $0 })
let negated = transform(5, using: { -$0 })

print(doubled)  // 10
print(squared)  // 25
print(negated)  // -5
```

### ฟังก์ชันที่รับ Closure หลายตัว

```swift
func processData(_ data: [Int],
                 filter: (Int) -> Bool,
                 transform: (Int) -> String) -> [String] {
    return data.filter(filter).map(transform)
}

let result = processData(
    [1, 2, 3, 4, 5, 6, 7, 8, 9, 10],
    filter: { $0 % 2 == 0 },           // เลขคู่
    transform: { "ตัวเลข \($0)" }      // แปลงเป็น String
)
print(result) // ["ตัวเลข 2", "ตัวเลข 4", "ตัวเลข 6", "ตัวเลข 8", "ตัวเลข 10"]
```

### Higher-Order Functions ที่กำหนดเอง

```swift
// กำหนด typealias สำหรับ closure type ที่ใช้บ่อย
typealias Predicate<T> = (T) -> Bool
typealias Transform<T, U> = (T) -> U

// ฟังก์ชัน generic ที่ใช้ closure
func find<T>(_ array: [T], where predicate: Predicate<T>) -> T? {
    for element in array {
        if predicate(element) {
            return element
        }
    }
    return nil
}

let numbers = [1, 5, 3, 8, 2, 7]
let firstBigNumber = find(numbers) { $0 > 6 }
print(firstBigNumber ?? "ไม่พบ") // Optional(8)

let words = ["apple", "banana", "cherry"]
let longWord = find(words) { $0.count > 5 }
print(longWord ?? "ไม่พบ") // Optional("banana")
```

---

## 7.12 Closures as Return Values

ฟังก์ชันสามารถคืน closure เป็นค่า return ได้ ซึ่งทำให้สามารถสร้างฟังก์ชันที่ generate ฟังก์ชันได้

### รูปแบบพื้นฐาน

```swift
// ฟังก์ชันที่คืน closure
func makeMultiplier(by factor: Int) -> (Int) -> Int {
    return { number in
        return number * factor  // capture factor
    }
}

let double = makeMultiplier(by: 2)
let triple = makeMultiplier(by: 3)
let quadruple = makeMultiplier(by: 4)

print(double(5))    // 10
print(triple(5))    // 15
print(quadruple(5)) // 20
```

### ตัวอย่าง: Function Composition

```swift
// สร้าง function ที่รวม functions อื่นเข้าด้วยกัน
func compose<T>(_ f: @escaping (T) -> T, _ g: @escaping (T) -> T) -> (T) -> T {
    return { value in
        f(g(value))
    }
}

let addOne = { (x: Int) -> Int in x + 1 }
let double = { (x: Int) -> Int in x * 2 }

let addOneThenDouble = compose(double, addOne)
let doubleThenAddOne = compose(addOne, double)

print(addOneThenDouble(3))  // (3+1)*2 = 8
print(doubleThenAddOne(3))  // (3*2)+1 = 7
```

### ตัวอย่าง: Partial Application

```swift
// Partial application - กำหนด argument บางส่วนล่วงหน้า
func partial<A, B, C>(_ f: @escaping (A, B) -> C, _ a: A) -> (B) -> C {
    return { b in f(a, b) }
}

func add(_ x: Int, _ y: Int) -> Int { x + y }

let add5 = partial(add, 5)
let add10 = partial(add, 10)

print(add5(3))   // 8
print(add10(3))  // 13
print(add5(7))   // 12
```

---

## 7.13 Closures in Collections

Closure มักถูกใช้ร่วมกับ collections (Array, Dictionary, Set)

### Array Operations ด้วย Closure

```swift
let students = [
    ("สมชาย", 85),
    ("สมหญิง", 92),
    ("สมศักดิ์", 78),
    ("สมใจ", 95),
    ("สมาน", 60)
]

// sort ตามคะแนน (มากไปน้อย)
let sortedByScore = students.sorted { $0.1 > $1.1 }
print("เรียงตามคะแนน:")
for (name, score) in sortedByScore {
    print("  \(name): \(score)")
}

// filter เฉพาะที่ผ่าน (>= 75)
let passing = students.filter { $0.1 >= 75 }
print("\nนักเรียนที่ผ่าน:")
passing.forEach { print("  \($0.0): \($0.1)") }

// map เพื่อแยกชื่อ
let names = students.map { $0.0 }
print("\nรายชื่อนักเรียน: \(names)")
```

### Dictionary ด้วย Closure

```swift
let inventory = [
    "แอปเปิ้ล": 50,
    "กล้วย": 30,
    "ส้ม": 45,
    "มะม่วง": 20,
    "สับปะรด": 60
]

// filter สินค้าที่มีน้อยกว่า 40
let lowStock = inventory.filter { $0.value < 40 }
print("สินค้าใกล้หมด:")
lowStock.forEach { key, value in
    print("  \(key): \(value) ชิ้น")
}

// mapValues เพิ่มสต็อก 20%
let restocked = inventory.mapValues { Int(Double($0) * 1.2) }
print("\nสต็อกหลังเติม 20%:")
restocked.forEach { print("  \($0.key): \($0.value)") }
```

---

## 7.14 Closures กับ map, filter, reduce

นี่คือ higher-order functions ที่สำคัญที่สุดใน Swift

### map

แปลงแต่ละ element ใน collection

```swift
let numbers = [1, 2, 3, 4, 5]

// สร้าง array ใหม่โดยแปลงแต่ละ element
let squares = numbers.map { $0 * $0 }
print(squares) // [1, 4, 9, 16, 25]

let strings = numbers.map { "ตัวเลข \($0)" }
print(strings) // ["ตัวเลข 1", "ตัวเลข 2", ...]

// map กับ String
let words = ["Hello", "World", "Swift"]
let lengths = words.map { $0.count }
print(lengths) // [5, 5, 5]

let uppercased = words.map { $0.uppercased() }
print(uppercased) // ["HELLO", "WORLD", "SWIFT"]
```

### filter

คัดเลือก elements ที่ตรงตามเงื่อนไข

```swift
let numbers = 1...20

// คัดเลือกเลขคู่
let evens = numbers.filter { $0 % 2 == 0 }
print(Array(evens)) // [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]

// คัดเลือกเลขที่หารด้วย 3 ลงตัว
let divisibleBy3 = numbers.filter { $0 % 3 == 0 }
print(Array(divisibleBy3)) // [3, 6, 9, 12, 15, 18]

// filter กับ String
let fruits = ["apple", "apricot", "banana", "avocado", "cherry"]
let aFruits = fruits.filter { $0.hasPrefix("a") }
print(aFruits) // ["apple", "apricot", "avocado"]
```

### reduce

รวม elements ทั้งหมดเป็นค่าเดียว

```swift
let numbers = [1, 2, 3, 4, 5]

// หาผลรวม
let sum = numbers.reduce(0) { $0 + $1 }
print(sum) // 15

// หาผลคูณ
let product = numbers.reduce(1) { $0 * $1 }
print(product) // 120

// หาค่าสูงสุด
let maximum = numbers.reduce(Int.min) { max($0, $1) }
print(maximum) // 5

// รวม String
let words = ["Swift", "is", "awesome"]
let sentence = words.reduce("") { 
    $0.isEmpty ? $1 : $0 + " " + $1
}
print(sentence) // "Swift is awesome"

// สร้าง Dictionary จาก Array
let fruits = ["apple", "banana", "cherry"]
let fruitLengths = fruits.reduce(into: [:]) { dict, fruit in
    dict[fruit] = fruit.count
}
print(fruitLengths) // ["apple": 5, "banana": 6, "cherry": 6]
```

### การรวม map, filter, reduce

```swift
let grades = [72, 85, 90, 65, 78, 92, 55, 88]

// หา average ของคะแนนที่ผ่าน (>= 70) หลังเพิ่ม bonus 5 คะแนน
let result = grades
    .filter { $0 >= 70 }           // คัดเลือกที่ผ่าน
    .map { $0 + 5 }                // เพิ่ม bonus
    .reduce(0, +) / grades.filter { $0 >= 70 }.count

print("Average ของคะแนนที่ผ่านหลัง bonus: \(result)")
```

---

## 7.15 compactMap และ flatMap

### compactMap

คล้าย `map` แต่จะ filter ออก `nil` values ด้วย

```swift
let strings = ["1", "2", "three", "4", "five", "6"]

// map จะได้ Optional values
let withNils: [Int?] = strings.map { Int($0) }
print(withNils) // [Optional(1), Optional(2), nil, Optional(4), nil, Optional(6)]

// compactMap จะ filter nil ออก
let numbers: [Int] = strings.compactMap { Int($0) }
print(numbers) // [1, 2, 4, 6]
```

### ตัวอย่าง compactMap ที่ซับซ้อน

```swift
struct Person {
    let name: String
    let age: Int?
}

let people = [
    Person(name: "สมชาย", age: 25),
    Person(name: "สมหญิง", age: nil),
    Person(name: "สมศักดิ์", age: 30),
    Person(name: "สมใจ", age: nil),
    Person(name: "สมาน", age: 28)
]

// ดึงเฉพาะคนที่มีอายุ
let ages = people.compactMap { $0.age }
print(ages) // [25, 30, 28]

let adultNames = people.compactMap { person -> String? in
    guard let age = person.age, age >= 18 else { return nil }
    return person.name
}
print(adultNames) // ["สมชาย", "สมศักดิ์", "สมาน"]
```

### flatMap

รวม nested collections ให้เป็น flat array

```swift
let nested = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]

// flatMap แปลง [[Int]] เป็น [Int]
let flat = nested.flatMap { $0 }
print(flat) // [1, 2, 3, 4, 5, 6, 7, 8, 9]

// ตัวอย่าง: หาคำทุกคำในประโยค
let sentences = ["Hello World", "Swift Programming", "iOS Development"]
let words = sentences.flatMap { $0.split(separator: " ").map(String.init) }
print(words) // ["Hello", "World", "Swift", "Programming", "iOS", "Development"]
```

### ความแตกต่างระหว่าง map, compactMap, flatMap

```swift
let values: [Int?] = [1, nil, 3, nil, 5]

// map: คงรูปแบบเดิม รวม nil ด้วย
let mapped = values.map { $0.map { $0 * 2 } }
print(mapped) // [Optional(2), nil, Optional(6), nil, Optional(10)]

// compactMap: ลบ nil ออก แล้ว transform
let compactMapped = values.compactMap { $0.map { $0 * 2 } }
print(compactMapped) // [2, 6, 10]

// flatMap กับ nested arrays
let arrays = [[1, 2], [3, 4], [5, 6]]
let flattened = arrays.flatMap { $0 }
print(flattened) // [1, 2, 3, 4, 5, 6]
```

---

## 7.16 sorted(by:)

ฟังก์ชัน sort ที่รับ closure เป็น comparison function

### รูปแบบพื้นฐาน

```swift
var numbers = [5, 2, 8, 1, 9, 3, 7, 4, 6]

// เรียงน้อยไปมาก
let ascending = numbers.sorted { $0 < $1 }
print(ascending) // [1, 2, 3, 4, 5, 6, 7, 8, 9]

// เรียงมากไปน้อย
let descending = numbers.sorted { $0 > $1 }
print(descending) // [9, 8, 7, 6, 5, 4, 3, 2, 1]

// ใช้ operator function โดยตรง
let sorted = numbers.sorted(by: <)
print(sorted) // [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### Sort ด้วย Custom Criteria

```swift
struct Student {
    let name: String
    let grade: Int
    let age: Int
}

let students = [
    Student(name: "สมชาย", grade: 85, age: 20),
    Student(name: "สมหญิง", grade: 92, age: 19),
    Student(name: "สมศักดิ์", grade: 85, age: 21),
    Student(name: "สมใจ", grade: 78, age: 20),
    Student(name: "สมาน", grade: 92, age: 18)
]

// เรียงตามเกรดก่อน ถ้าเกรดเท่ากันให้เรียงตามอายุ
let sorted = students.sorted { a, b in
    if a.grade != b.grade {
        return a.grade > b.grade
    }
    return a.age < b.age
}

print("นักเรียนเรียงตามเกรด:")
for student in sorted {
    print("  \(student.name): เกรด \(student.grade), อายุ \(student.age)")
}
```

### Sort แบบ Stable (การรักษาลำดับ element ที่เท่ากัน)

```swift
// Swift ใน version ใหม่ๆ มี stable sort
let items = [(1, "B"), (2, "A"), (3, "B"), (4, "A")]

// เรียงตาม second element
let sortedItems = items.sorted { $0.1 < $1.1 }
// ผลลัพธ์คาดหวัง: [(2, "A"), (4, "A"), (1, "B"), (3, "B")]
// (การรักษาลำดับ element ที่มี key เท่ากัน)
print(sortedItems.map { "\($0.0)\($0.1)" }) // ["2A", "4A", "1B", "3B"]
```

---

## 7.17 Closure Reference Types

Closure เป็น reference type ใน Swift ซึ่งหมายความว่าตัวแปรหลายตัวสามารถอ้างถึง closure เดียวกันได้

### การแชร์ Closure

```swift
var counter = 0
let increment = { counter += 1 }

let inc1 = increment
let inc2 = increment

// ทั้งสองอ้างถึง closure เดียวกัน
inc1() // counter = 1
inc2() // counter = 2
inc1() // counter = 3

print(counter) // 3 - ค่า counter แชร์กัน
```

### ผลกระทบต่อ Behavior

```swift
func makeAccumulator() -> (Int) -> Int {
    var total = 0
    return { amount in
        total += amount
        return total
    }
}

let acc1 = makeAccumulator()
let acc2 = acc1  // แชร์ closure เดียวกัน!

print(acc1(10)) // 10
print(acc1(20)) // 30
print(acc2(5))  // 35 (acc2 ใช้ total ของ acc1!)
print(acc1(15)) // 50

// ถ้าต้องการ accumulator แยกกัน ต้องสร้างใหม่
let acc3 = makeAccumulator()
print(acc3(10)) // 10 (เริ่มต้นใหม่)
```

---

## 7.18 Memory Management กับ Closures

### Retain Cycle คืออะไร?

Retain cycle เกิดขึ้นเมื่อสองออบเจ็กต์อ้างถึงกันแบบ strong ทำให้ทั้งคู่ไม่สามารถถูก deallocate ได้

```swift
class Parent {
    var child: Child?
    
    deinit {
        print("Parent ถูก deallocated")
    }
}

class Child {
    var parent: Parent?  // Strong reference ก่อให้เกิด retain cycle
    
    deinit {
        print("Child ถูก deallocated")
    }
}

// สร้าง retain cycle
var parent: Parent? = Parent()
var child: Child? = Child()
parent?.child = child
child?.parent = parent  // Retain cycle!

parent = nil  // Parent ยัง alive เพราะ child ถือ strong ref
child = nil   // Child ยัง alive เพราะ parent ถือ strong ref
// deinit ทั้งคู่ไม่ถูกเรียก!
```

### Retain Cycle ใน Closure

```swift
class Timer {
    var callback: (() -> Void)?
    var name: String
    
    init(name: String) {
        self.name = name
    }
    
    func start() {
        // Retain cycle: Timer -> closure -> Timer
        callback = {
            print("Timer \(self.name) fired!")  // Strong capture
        }
    }
    
    deinit {
        print("\(name) ถูก deallocated")
    }
}

var timer: Timer? = Timer(name: "main")
timer?.start()
timer = nil  // Timer ไม่ถูก deallocated! Retain cycle!
```

### แก้ด้วย [weak self]

```swift
class TimerFixed {
    var callback: (() -> Void)?
    var name: String
    
    init(name: String) {
        self.name = name
    }
    
    func start() {
        // แก้ retain cycle ด้วย [weak self]
        callback = { [weak self] in
            guard let self = self else { return }
            print("Timer \(self.name) fired!")
        }
    }
    
    deinit {
        print("\(name) ถูก deallocated")
    }
}

var timerFixed: TimerFixed? = TimerFixed(name: "fixed")
timerFixed?.start()
timerFixed = nil  // TimerFixed ถูก deallocated ถูกต้อง!
```

### แก้ด้วย [unowned self]

```swift
class Request {
    var completionHandler: ((String) -> Void)?
    let url: String
    
    init(url: String) {
        self.url = url
    }
    
    func setCompletion(_ handler: @escaping (String) -> Void) {
        completionHandler = handler
    }
    
    deinit {
        print("Request \(url) ถูก deallocated")
    }
}

class ViewController2 {
    let request: Request
    
    init(url: String) {
        self.request = Request(url: url)
    }
    
    func startRequest() {
        // unowned ใช้เมื่อมั่นใจว่า ViewController2 จะยังมีอยู่เสมอ
        request.setCompletion { [unowned self] data in
            self.handleResponse(data)
        }
    }
    
    func handleResponse(_ data: String) {
        print("ได้รับ response: \(data)")
    }
    
    deinit {
        print("ViewController2 ถูก deallocated")
    }
}
```

---

## 7.19 Common Closure Patterns

### Pattern 1: Completion Handler

```swift
// รูปแบบ completion handler พื้นฐาน
func loadImage(from url: String, completion: @escaping (UIImage?, Error?) -> Void) {
    DispatchQueue.global().async {
        // จำลองการโหลด
        // let image = downloadImage(from: url)
        
        DispatchQueue.main.async {
            // completion(image, nil) // สำเร็จ
            // หรือ
            // completion(nil, error) // ผิดพลาด
        }
    }
}
```

### Pattern 2: Result Type Handler (Modern Swift)

```swift
enum NetworkError: Error {
    case invalidURL
    case noData
    case decodingError
}

func fetchUser(id: Int, completion: @escaping (Result<String, NetworkError>) -> Void) {
    // จำลอง async operation
    DispatchQueue.global().async {
        if id > 0 {
            DispatchQueue.main.async {
                completion(.success("User \(id)"))
            }
        } else {
            DispatchQueue.main.async {
                completion(.failure(.invalidURL))
            }
        }
    }
}

// การใช้งาน
fetchUser(id: 42) { result in
    switch result {
    case .success(let user):
        print("ได้รับข้อมูล user: \(user)")
    case .failure(let error):
        print("ผิดพลาด: \(error)")
    }
}
```

### Pattern 3: Builder Pattern ด้วย Closure

```swift
class AlertBuilder {
    private var title: String = ""
    private var message: String = ""
    private var actions: [(String, () -> Void)] = []
    
    func withTitle(_ title: String) -> AlertBuilder {
        self.title = title
        return self
    }
    
    func withMessage(_ message: String) -> AlertBuilder {
        self.message = message
        return self
    }
    
    func addAction(_ title: String, handler: @escaping () -> Void) -> AlertBuilder {
        actions.append((title, handler))
        return self
    }
    
    func show() {
        print("=== Alert: \(title) ===")
        print(message)
        print("Actions:", actions.map { $0.0 })
    }
}

AlertBuilder()
    .withTitle("ยืนยันการลบ")
    .withMessage("คุณต้องการลบข้อมูลนี้หรือไม่?")
    .addAction("ลบ") { print("ลบแล้ว") }
    .addAction("ยกเลิก") { print("ยกเลิก") }
    .show()
```

### Pattern 4: Memoization

```swift
// Cache ผลลัพธ์ของการคำนวณที่ใช้เวลานาน
func memoize<T: Hashable, U>(_ function: @escaping (T) -> U) -> (T) -> U {
    var cache: [T: U] = [:]
    return { input in
        if let cached = cache[input] {
            return cached
        }
        let result = function(input)
        cache[input] = result
        return result
    }
}

// Fibonacci ปกติ (ช้า)
func fibonacci(_ n: Int) -> Int {
    if n <= 1 { return n }
    return fibonacci(n - 1) + fibonacci(n - 2)
}

// Fibonacci ด้วย memoization (เร็วกว่ามาก)
var memoFib: ((Int) -> Int)!
memoFib = memoize { n in
    if n <= 1 { return n }
    return memoFib(n - 1) + memoFib(n - 2)
}

print(memoFib(10)) // 55
print(memoFib(20)) // 6765
```

---

## 7.20 Practical Exercises

### Exercise 1: Custom Array Operations

```swift
// สร้าง extension สำหรับ Array ที่ใช้ closure
extension Array {
    // partitionBy: แบ่ง array เป็นสองส่วน
    func partitionBy(_ predicate: (Element) -> Bool) -> ([Element], [Element]) {
        var passing: [Element] = []
        var failing: [Element] = []
        
        for element in self {
            if predicate(element) {
                passing.append(element)
            } else {
                failing.append(element)
            }
        }
        
        return (passing, failing)
    }
    
    // groupBy: จัดกลุ่มตาม key
    func groupBy<Key: Hashable>(_ keySelector: (Element) -> Key) -> [Key: [Element]] {
        var result: [Key: [Element]] = [:]
        for element in self {
            let key = keySelector(element)
            result[key, default: []].append(element)
        }
        return result
    }
    
    // chunk: แบ่งเป็น chunk ขนาด n
    func chunked(into size: Int) -> [[Element]] {
        stride(from: 0, to: count, by: size).map {
            Array(self[$0..<Swift.min($0 + size, count)])
        }
    }
}

// ทดสอบ
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

let (evens, odds) = numbers.partitionBy { $0 % 2 == 0 }
print("เลขคู่: \(evens)")
print("เลขคี่: \(odds)")

let grouped = numbers.groupBy { $0 % 3 }
print("จัดกลุ่มตาม mod 3: \(grouped)")

let chunks = numbers.chunked(into: 3)
print("แบ่งเป็น chunks: \(chunks)")
```

### Exercise 2: Pipeline Processing

```swift
// สร้าง pipeline สำหรับ process data
struct Pipeline<T> {
    private let value: T
    
    init(_ value: T) {
        self.value = value
    }
    
    func map<U>(_ transform: (T) -> U) -> Pipeline<U> {
        return Pipeline<U>(transform(value))
    }
    
    func flatMap<U>(_ transform: (T) -> Pipeline<U>) -> Pipeline<U> {
        return transform(value)
    }
    
    var result: T {
        return value
    }
}

// ใช้งาน
let processedValue = Pipeline(10)
    .map { $0 * 2 }      // 20
    .map { $0 + 5 }      // 25
    .map { Double($0) }  // 25.0
    .map { $0 / 2.5 }    // 10.0
    .result

print("Pipeline result: \(processedValue)") // 10.0
```

### Exercise 3: Event System

```swift
// สร้าง simple event system
class EventEmitter<T> {
    private var handlers: [(T) -> Void] = []
    
    func on(_ handler: @escaping (T) -> Void) {
        handlers.append(handler)
    }
    
    func emit(_ value: T) {
        handlers.forEach { $0(value) }
    }
    
    func removeAllHandlers() {
        handlers.removeAll()
    }
}

// ใช้งาน
let buttonTapped = EventEmitter<String>()

buttonTapped.on { button in
    print("Button '\(button)' ถูกกด - Handler 1")
}

buttonTapped.on { button in
    print("Button '\(button)' ถูกกด - Handler 2")
}

buttonTapped.emit("Submit")
// Button 'Submit' ถูกกด - Handler 1
// Button 'Submit' ถูกกด - Handler 2
```

### Exercise 4: Lazy Sequence ด้วย Closure

```swift
// สร้าง infinite sequence
func makeSequence<T>(seed: T, next: @escaping (T) -> T) -> AnySequence<T> {
    return AnySequence { () -> AnyIterator<T> in
        var current = seed
        return AnyIterator {
            let value = current
            current = next(current)
            return value
        }
    }
}

// Fibonacci sequence
let fibonacci = makeSequence(seed: (0, 1)) { ($0.1, $0.0 + $0.1) }
let firstTenFibs = fibonacci.prefix(10).map { $0.0 }
print("10 Fibonacci แรก: \(Array(firstTenFibs))")

// Powers of 2
let powersOf2 = makeSequence(seed: 1) { $0 * 2 }
let firstTenPowers = Array(powersOf2.prefix(10))
print("10 กำลังของ 2 แรก: \(firstTenPowers)")
```

---

## 7.21 Real-World Examples

### Example 1: URLSession Completion Handler

```swift
import Foundation

// รูปแบบ modern ของ URLSession callback
struct APIService {
    let baseURL = "https://jsonplaceholder.typicode.com"
    
    func fetchPost(id: Int, completion: @escaping (Result<[String: Any], Error>) -> Void) {
        guard let url = URL(string: "\(baseURL)/posts/\(id)") else {
            completion(.failure(URLError(.badURL)))
            return
        }
        
        URLSession.shared.dataTask(with: url) { data, response, error in
            if let error = error {
                completion(.failure(error))
                return
            }
            
            guard let data = data else {
                completion(.failure(URLError(.zeroByteResource)))
                return
            }
            
            do {
                let json = try JSONSerialization.jsonObject(with: data) as? [String: Any] ?? [:]
                DispatchQueue.main.async {
                    completion(.success(json))
                }
            } catch {
                completion(.failure(error))
            }
        }.resume()
    }
}

// การใช้งาน
let service = APIService()
service.fetchPost(id: 1) { result in
    switch result {
    case .success(let post):
        print("โพสต์: \(post["title"] ?? "ไม่มีหัวข้อ")")
    case .failure(let error):
        print("ผิดพลาด: \(error.localizedDescription)")
    }
}
```

### Example 2: Animation Completion

```swift
// ตัวอย่าง Animation ด้วย Closure
// (ใช้ใน iOS/macOS development)

class AnimationChain {
    private var animations: [(@escaping () -> Void) -> Void] = []
    
    func add(_ animation: @escaping (@escaping () -> Void) -> Void) -> AnimationChain {
        animations.append(animation)
        return self
    }
    
    func start() {
        executeNext(index: 0)
    }
    
    private func executeNext(index: Int) {
        guard index < animations.count else { return }
        
        animations[index] {
            self.executeNext(index: index + 1)
        }
    }
}

// การใช้งาน (จำลอง)
func animateFade(_ completion: @escaping () -> Void) {
    print("Animating fade...")
    DispatchQueue.main.asyncAfter(deadline: .now() + 0.5) {
        print("Fade complete")
        completion()
    }
}

func animateSlide(_ completion: @escaping () -> Void) {
    print("Animating slide...")
    DispatchQueue.main.asyncAfter(deadline: .now() + 0.5) {
        print("Slide complete")
        completion()
    }
}

let chain = AnimationChain()
chain
    .add(animateFade)
    .add(animateSlide)
    .add { completion in
        print("Final animation")
        completion()
    }
// chain.start()
```

### Example 3: Data Transformation Pipeline

```swift
// ตัวอย่างการ transform data แบบ pipeline
struct DataPipeline {
    typealias Transform<T, U> = (T) -> U
    
    static func process<T, U, V>(
        _ input: [T],
        step1: (T) -> U?,
        step2: (U) -> V?,
        step3: (V) -> Bool
    ) -> [V] {
        return input
            .compactMap(step1)
            .compactMap(step2)
            .filter(step3)
    }
}

// ใช้กับข้อมูลจริง
let rawData = ["25", "abc", "30", "17", "invalid", "22", "35", "15"]

let processedData = DataPipeline.process(
    rawData,
    step1: { Int($0) },           // แปลงเป็น Int (บาง string อาจ fail)
    step2: { age -> String? in
        age >= 18 ? "ผู้ใหญ่: \(age)" : nil  // filter ผู้ใหญ่
    },
    step3: { !$0.isEmpty }        // ตรวจสอบไม่ว่าง
)

print(processedData)
// ["ผู้ใหญ่: 25", "ผู้ใหญ่: 30", "ผู้ใหญ่: 22", "ผู้ใหญ่: 35"]
```

### Example 4: Notification System

```swift
// Notification system แบบ type-safe
class NotificationCenter<T> {
    private var observers: [ObjectIdentifier: (T) -> Void] = [:]
    
    func addObserver<O: AnyObject>(_ observer: O, handler: @escaping (T) -> Void) {
        let key = ObjectIdentifier(observer)
        observers[key] = handler
    }
    
    func removeObserver<O: AnyObject>(_ observer: O) {
        let key = ObjectIdentifier(observer)
        observers.removeValue(forKey: key)
    }
    
    func notify(_ value: T) {
        observers.values.forEach { $0(value) }
    }
}

// ใช้งาน
class UserManager {
    let loginEvent = NotificationCenter<String>()
    
    func login(username: String) {
        // ทำ login logic
        loginEvent.notify(username)
    }
}

class LoggingService {
    func start(observing manager: UserManager) {
        manager.loginEvent.addObserver(self) { username in
            print("Log: User '\(username)' logged in at \(Date())")
        }
    }
}

let userManager = UserManager()
let logger = LoggingService()
logger.start(observing: userManager)

userManager.login(username: "สมชาย")
// Log: User 'สมชาย' logged in at ...
```

---

## 7.22 Advanced Closure Techniques

### Currying

```swift
// Currying: แปลงฟังก์ชันหลาย parameter เป็น chain ของฟังก์ชัน parameter เดียว
func curry<A, B, C>(_ function: @escaping (A, B) -> C) -> (A) -> (B) -> C {
    return { a in
        return { b in
            return function(a, b)
        }
    }
}

func power(_ base: Int, _ exponent: Int) -> Int {
    return Int(pow(Double(base), Double(exponent)))
}

let curriedPower = curry(power)
let square = curriedPower(2)  // 2^?
let cube = curriedPower(3)    // 3^?

print(square(3))  // 2^3 = 8
print(cube(4))    // 3^4 = 81
print(curriedPower(10)(3))  // 10^3 = 1000
```

### Thunk Pattern

```swift
// Thunk: closure ที่ไม่รับ parameter และคืนค่า
typealias Thunk<T> = () -> T

func lazy<T>(_ computation: @escaping () -> T) -> Thunk<T> {
    var cached: T?
    return {
        if let value = cached {
            return value
        }
        let value = computation()
        cached = value
        return value
    }
}

// การใช้งาน
let expensiveComputation = lazy {
    print("Computing...")
    return (1...1000).reduce(0, +)
}

let result1 = expensiveComputation()  // "Computing..." แล้วได้ 500500
let result2 = expensiveComputation()  // ไม่ print อีก ได้ผลจาก cache
print(result1)
print(result2)
```

### Retry Pattern

```swift
// Retry mechanism ด้วย closure
func retry<T>(
    times: Int,
    delay: TimeInterval = 0,
    operation: @escaping () throws -> T,
    completion: @escaping (Result<T, Error>) -> Void
) {
    do {
        let result = try operation()
        completion(.success(result))
    } catch {
        if times > 1 {
            print("ลองใหม่อีก \(times - 1) ครั้ง...")
            DispatchQueue.global().asyncAfter(deadline: .now() + delay) {
                retry(times: times - 1, delay: delay, operation: operation, completion: completion)
            }
        } else {
            completion(.failure(error))
        }
    }
}

// การใช้งาน
var attemptCount = 0
retry(times: 3, delay: 0.1, operation: {
    attemptCount += 1
    print("ความพยายามที่ \(attemptCount)")
    if attemptCount < 3 {
        throw URLError(.timedOut)
    }
    return "สำเร็จ!"
}) { result in
    switch result {
    case .success(let value):
        print("ผลลัพธ์: \(value)")
    case .failure(let error):
        print("ล้มเหลว: \(error)")
    }
}
```

---

## 7.23 สรุป

### สิ่งที่ได้เรียนรู้

1. **Closure basics** - Closure คือ block ของโค้ดที่สามารถส่งผ่านและเรียกใช้ได้
2. **Syntax variations** - ตั้งแต่ verbose จนถึง shorthand `$0`, `$1`
3. **Trailing closure** - เขียน closure ไว้ข้างนอกวงเล็บเมื่อเป็น argument สุดท้าย
4. **Capturing** - Closure สามารถ capture ค่าจาก surrounding context
5. **Capture lists** - ใช้ `[weak self]` และ `[unowned self]` ป้องกัน retain cycle
6. **@escaping** - สำหรับ closure ที่ถูกเก็บไว้หลัง function return
7. **Higher-order functions** - `map`, `filter`, `reduce`, `compactMap`, `flatMap`
8. **Memory management** - เข้าใจและป้องกัน retain cycle

### Best Practices

```swift
// ✅ ดี: ใช้ trailing closure เมื่ออ่านง่ายขึ้น
let result = [1, 2, 3].map { $0 * 2 }

// ✅ ดี: ใช้ [weak self] ใน escaping closures
someAsyncCall { [weak self] data in
    self?.handleData(data)
}

// ✅ ดี: ใช้ชื่อที่สื่อความหมายเมื่อ closure ซับซ้อน
let sortedStudents = students.sorted { student1, student2 in
    student1.grade > student2.grade
}

// ❌ ไม่ดี: Retain cycle
class BadClass {
    var closure: (() -> Void)?
    func setup() {
        closure = {
            print(self)  // Strong capture - retain cycle!
        }
    }
}

// ✅ ดี: ป้องกัน retain cycle
class GoodClass {
    var closure: (() -> Void)?
    func setup() {
        closure = { [weak self] in
            print(self ?? "nil")  // Weak capture - ปลอดภัย
        }
    }
}
```

### Quick Reference

| Feature | Syntax | ใช้เมื่อ |
|---------|--------|---------|
| Basic closure | `{ (params) -> Type in body }` | ทั่วไป |
| Trailing closure | `func { body }` | Parameter สุดท้าย |
| Shorthand args | `{ $0 + $1 }` | ง่ายและอ่านเข้าใจ |
| Weak capture | `[weak self]` | ป้องกัน retain cycle |
| Unowned capture | `[unowned self]` | self ต้องยังมีอยู่ |
| Escaping | `@escaping` | Closure อยู่นานกว่า function |
| Autoclosure | `@autoclosure` | Lazy evaluation |

---

## แบบฝึกหัด

### ระดับเริ่มต้น

1. สร้าง closure ที่รับ String และคืน String ที่กลับหลัง
2. ใช้ `filter` หาคำที่มีความยาวมากกว่า 5 ตัวอักษรจาก array
3. ใช้ `map` แปลง array ของ Int เป็น array ของ String ในรูปแบบ "ตัวเลข X"

### ระดับกลาง

4. สร้างฟังก์ชันที่คืน closure สำหรับตรวจสอบว่าตัวเลขหารด้วย n ลงตัวหรือไม่
5. ใช้ `reduce` หา greatest common divisor (GCD) ของ array ของ Int
6. สร้าง memoization function สำหรับ factorial

### ระดับสูง

7. สร้าง functional pipeline ที่สามารถ chain operations ได้
8. Implement Promise/Future pattern ด้วย closure
9. สร้าง observer pattern ที่ type-safe ด้วย generic closure

---

*จบบทที่ 7: Closures ใน Swift*
