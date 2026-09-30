# ส่วนที่ 8: Optionals ใน Swift

## บทนำ

Optionals เป็นหนึ่งในคุณสมบัติที่โดดเด่นที่สุดของ Swift และเป็นสิ่งที่ทำให้ Swift แตกต่างจากภาษาโปรแกรมอื่นๆ อย่างมาก Optionals ช่วยให้เราจัดการกับการไม่มีค่า (absence of value) ได้อย่างปลอดภัยและชัดเจน

---

## 8.1 Optionals คืออะไรและทำไมต้องมี?

### ปัญหาใน Null Safety

ในภาษาโปรแกรมเก่าๆ เช่น Objective-C, C++, Java การส่งค่า `null` หรือ `nil` ไปที่ตัวแปรที่ไม่คาดหวัง `null` มักเกิด runtime error ที่เรียกว่า "Null Pointer Exception" หรือ "NullReferenceException"

```swift
// ใน Objective-C (ไม่ใช่ Swift)
// NSString *name = nil;
// NSUInteger length = [name length]; // ไม่ crash ใน ObjC แต่อาจก่อปัญหาทาง logic

// ใน Java
// String name = null;
// int length = name.length(); // NullPointerException!
```

### Swift's Solution: Optionals

Swift แก้ปัญหานี้ด้วย type system ที่ force ให้ programmer จัดการกับ `nil` อย่างชัดเจน

```swift
// ใน Swift ตัวแปรปกติไม่สามารถเป็น nil ได้
var name: String = "สมชาย"
// name = nil  // Error: 'nil' cannot be assigned to type 'String'

// ต้องใช้ Optional เมื่อต้องการให้ค่าเป็น nil ได้
var optionalName: String? = "สมชาย"
optionalName = nil  // ได้!
```

### ทำไม Optionals ถึงสำคัญ?

1. **Compile-time safety** - Swift ตรวจสอบ optional handling ตั้งแต่ compile time
2. **Explicit intent** - โค้ดชัดเจนว่าค่าไหนสามารถเป็น nil ได้
3. **Prevents crashes** - ลด runtime crashes จาก null dereference
4. **Better documentation** - Type signature บอกว่าค่าอาจเป็น nil หรือไม่

---

## 8.2 Optional Declaration

### วิธีประกาศ Optional

```swift
// รูปแบบที่ 1: ใช้ ? หลัง type (syntactic sugar)
var age: Int? = 25
var name: String? = nil
var isLoggedIn: Bool? = true

// รูปแบบที่ 2: ใช้ Optional<Type> (รูปแบบเต็ม)
var age2: Optional<Int> = 25
var name2: Optional<String> = nil

// ทั้งสองรูปแบบเหมือนกัน
print(age)     // Optional(25)
print(age2)    // Optional(25)
```

### Optional Types ต่างๆ

```swift
// Optional ของ types ต่างๆ
var intOpt: Int? = 42
var doubleOpt: Double? = 3.14
var stringOpt: String? = "Hello"
var boolOpt: Bool? = false
var arrayOpt: [Int]? = [1, 2, 3]
var dictOpt: [String: Int]? = ["a": 1]

// Optional ของ custom types
struct Point {
    let x: Double
    let y: Double
}

var pointOpt: Point? = Point(x: 1.0, y: 2.0)
var nullPoint: Point? = nil

print(pointOpt)  // Optional(Point(x: 1.0, y: 2.0))
print(nullPoint) // nil
```

### ค่า default ของ Optional

```swift
// Optional มีค่า default เป็น nil
var optionalString: String?
print(optionalString) // nil (ไม่ต้องกำหนดค่าเริ่มต้น)

// ต่างจาก non-optional ที่ต้องกำหนดค่าก่อนใช้
// var nonOptional: String  // Error: Variable 'nonOptional' used before being initialized
var nonOptional: String = "ต้องกำหนดค่าเสมอ"
```

---

## 8.3 nil ใน Swift

### ความหมายของ nil

ใน Swift, `nil` หมายถึง "ไม่มีค่า" (absence of value) และสามารถใช้กับ Optional เท่านั้น

```swift
// nil ใช้กับ Optional ได้
var optInt: Int? = nil
var optStr: String? = nil
var optArr: [Int]? = nil

print(optInt == nil)  // true
print(optStr == nil)  // true

// ตรวจสอบ nil
if optInt == nil {
    print("optInt ไม่มีค่า")
}

if optStr != nil {
    print("optStr มีค่า")
} else {
    print("optStr ไม่มีค่า")
}
```

### nil กับ Types ต่างๆ

```swift
// nil ของ Optional Int vs 0
var noValue: Int? = nil
var zero: Int? = 0
var positiveZero: Int = 0

print(noValue)     // nil
print(zero)        // Optional(0)
print(positiveZero) // 0

// ความแตกต่าง
print(noValue == nil)  // true
print(zero == nil)     // false
print(zero == 0)       // true (ด้วย automatic unwrapping ใน comparison)
```

### nil ใน Collections

```swift
// Array ที่มี Optional elements
var mixedArray: [Int?] = [1, nil, 3, nil, 5]
print(mixedArray) // [Optional(1), nil, Optional(3), nil, Optional(5)]

// Dictionary ที่มี Optional values
var dict: [String: Int?] = [
    "มีค่า": 42,
    "ไม่มีค่า": nil
]
print(dict) // ["มีค่า": Optional(42), "ไม่มีค่า": nil]
```

---

## 8.4 Optional Binding (if let)

**Optional binding** คือวิธีการ "unwrap" Optional อย่างปลอดภัยด้วย `if let`

### รูปแบบพื้นฐาน

```swift
var optionalName: String? = "สมชาย"

// if let unwrap Optional อย่างปลอดภัย
if let name = optionalName {
    // name เป็น String (ไม่ใช่ String?) ใน scope นี้
    print("สวัสดี \(name)!")
} else {
    print("ไม่มีชื่อ")
}

// ลอง nil
optionalName = nil

if let name = optionalName {
    print("สวัสดี \(name)!")
} else {
    print("ไม่มีชื่อ")  // จะ print นี้แทน
}
```

### Shadowing ใน if let

```swift
var optionalAge: Int? = 25

// สามารถใช้ชื่อเดิมได้ (shadowing)
if let optionalAge = optionalAge {
    // optionalAge เป็น Int ไม่ใช่ Int? แล้ว
    print("อายุ: \(optionalAge)")
}

// Swift 5.7+: Short syntax
if let optionalAge {
    // ใช้ชื่อเดิม ไม่ต้องเขียน = optionalAge
    print("อายุ: \(optionalAge)")
}
```

### Multiple if let

```swift
var firstName: String? = "สม"
var lastName: String? = "ชาย"
var age: Int? = 25

// unwrap หลาย Optional พร้อมกัน
if let firstName = firstName,
   let lastName = lastName,
   let age = age {
    print("ชื่อ: \(firstName)\(lastName), อายุ: \(age)")
}

// ถ้า Optional ตัวใดตัวหนึ่งเป็น nil จะ skip ทั้งหมด
var middleName: String? = nil

if let first = firstName,
   let middle = middleName,  // nil ตัวนี้
   let last = lastName {
    print("ชื่อเต็ม: \(first) \(middle) \(last)")
} else {
    print("ข้อมูลไม่ครบ")  // จะ print นี้
}
```

### if let กับ Conditions

```swift
var score: Int? = 85

// ผสม optional binding กับ condition
if let score = score, score >= 70 {
    print("ผ่าน! คะแนน: \(score)")
} else {
    print("ไม่ผ่านหรือไม่มีคะแนน")
}

// ใช้กับ String
var userInput: String? = "  hello  "
if let input = userInput, !input.trimmingCharacters(in: .whitespaces).isEmpty {
    print("มี input: '\(input.trimmingCharacters(in: .whitespaces))'")
}
```

---

## 8.5 Optional Binding (guard let)

`guard let` ใช้ unwrap Optional แต่แตกต่างจาก `if let` คือถ้า Optional เป็น nil จะต้อง exit จาก scope ปัจจุบัน

### รูปแบบพื้นฐาน

```swift
func greet(name: String?) {
    // guard let ต้องมี else clause ที่ exit จาก scope
    guard let name = name else {
        print("ไม่มีชื่อที่จะทักทาย")
        return  // ต้อง exit!
    }
    
    // name เป็น String ที่ unwrapped แล้ว
    // และสามารถใช้ได้ตลอด scope ที่เหลือของฟังก์ชัน
    print("สวัสดี \(name)!")
}

greet(name: "สมชาย")  // สวัสดี สมชาย!
greet(name: nil)       // ไม่มีชื่อที่จะทักทาย
```

### ความแตกต่างระหว่าง if let และ guard let

```swift
func processWithIfLet(value: Int?) {
    if let value = value {
        // value ใช้ได้แค่ใน block นี้
        print("ค่า: \(value)")
        // โค้ดอื่นๆ ต้องอยู่ใน block
    }
    // value ไม่สามารถใช้ที่นี่
}

func processWithGuardLet(value: Int?) {
    guard let value = value else {
        print("ไม่มีค่า")
        return  // Exit early
    }
    
    // value ใช้ได้ตรงนี้และตลอดไป
    print("ค่า: \(value)")
    print("สองเท่า: \(value * 2)")
    print("สามเท่า: \(value * 3)")
    // ง่ายกว่าการ nest if let
}
```

### Guard กับ Early Return Pattern

```swift
struct User {
    var name: String?
    var email: String?
    var age: Int?
}

func createUserProfile(_ user: User) -> String {
    // Guard ช่วยลด nesting และทำให้โค้ดอ่านง่ายขึ้น
    guard let name = user.name else {
        return "ข้อผิดพลาด: ต้องมีชื่อ"
    }
    
    guard let email = user.email else {
        return "ข้อผิดพลาด: ต้องมี email"
    }
    
    guard let age = user.age, age >= 18 else {
        return "ข้อผิดพลาด: ต้องอายุ 18 ปีขึ้นไป"
    }
    
    // โค้ดหลักอยู่ที่นี่ ไม่มี nesting
    return "สร้าง profile สำเร็จ: \(name) (\(email)), อายุ \(age)"
}

let validUser = User(name: "สมชาย", email: "somchai@example.com", age: 25)
let invalidUser = User(name: "สมหญิง", email: nil, age: 20)
let youngUser = User(name: "เด็กน้อย", email: "kid@example.com", age: 15)

print(createUserProfile(validUser))   // สร้าง profile สำเร็จ
print(createUserProfile(invalidUser)) // ข้อผิดพลาด: ต้องมี email
print(createUserProfile(youngUser))   // ข้อผิดพลาด: ต้องอายุ 18 ปีขึ้นไป
```

### Guard ใน Loop

```swift
let rawData = ["25", "abc", "30", nil, "17", "invalid", "22"]

var validAges: [Int] = []

for item in rawData {
    guard let item = item else {
        print("ข้ามค่า nil")
        continue  // ข้ามไปยัง iteration ถัดไป
    }
    
    guard let age = Int(item) else {
        print("'\(item)' ไม่ใช่ตัวเลข")
        continue
    }
    
    guard age > 0 && age < 150 else {
        print("อายุ \(age) ไม่สมเหตุสมผล")
        continue
    }
    
    validAges.append(age)
}

print("อายุที่ valid: \(validAges)")
```

---

## 8.6 Optional Chaining

**Optional chaining** ช่วยให้เราเรียก properties, methods, และ subscripts บน Optional ได้อย่างปลอดภัย โดยถ้า Optional เป็น nil จะคืนค่า nil ทันที

### รูปแบบพื้นฐาน

```swift
class Address {
    var street: String?
    var city: String?
    var zipCode: String?
}

class Person {
    var name: String
    var address: Address?
    
    init(name: String) {
        self.name = name
    }
}

let person = Person(name: "สมชาย")

// Optional chaining ด้วย ?
// ถ้า person.address เป็น nil จะคืน nil ทันที ไม่ crash
let city = person.address?.city
print(city ?? "ไม่มีที่อยู่") // ไม่มีที่อยู่

// เพิ่มที่อยู่
person.address = Address()
person.address?.city = "กรุงเทพมหานคร"

let cityAfterSet = person.address?.city
print(cityAfterSet ?? "ไม่มีที่อยู่") // กรุงเทพมหานคร
```

### Chaining หลายระดับ

```swift
class Company {
    var name: String
    var ceo: Person?
    
    init(name: String) {
        self.name = name
    }
}

let company = Company(name: "บริษัท Swift จำกัด")
let ceo = Person(name: "คุณสมชาย")
ceo.address = Address()
ceo.address?.city = "กรุงเทพมหานคร"
ceo.address?.street = "ถนนสุขุมวิท"

company.ceo = ceo

// Chain ยาวๆ - ถ้าตัวใดตัวหนึ่งเป็น nil จะคืน nil
let ceoCity = company.ceo?.address?.city
print(ceoCity ?? "ไม่ทราบ") // กรุงเทพมหานคร

// ถ้า ceo ไม่มีที่อยู่
company.ceo?.address = nil
let ceoStreet = company.ceo?.address?.street
print(ceoStreet ?? "ไม่ทราบ") // ไม่ทราบ
```

### Method Calls ผ่าน Optional Chaining

```swift
class TextProcessor {
    var text: String?
    
    func uppercase() -> TextProcessor {
        text = text?.uppercased()
        return self
    }
    
    func trim() -> TextProcessor {
        text = text?.trimmingCharacters(in: .whitespaces)
        return self
    }
    
    func wordCount() -> Int {
        return text?.split(separator: " ").count ?? 0
    }
}

let processor = TextProcessor()
processor.text = "  hello world swift  "

// Optional chaining กับ method calls
let result = processor.trim().uppercase().wordCount()
print("จำนวนคำ: \(result)") // 3

// เรียก method บน optional object
var optProcessor: TextProcessor? = TextProcessor()
optProcessor?.text = "test"
let uppercased = optProcessor?.uppercase().text
print(uppercased ?? "nil") // TEST
```

### Subscript ผ่าน Optional Chaining

```swift
var optArray: [Int]? = [1, 2, 3, 4, 5]
var optDict: [String: Int]? = ["apple": 1, "banana": 2]

// ใช้ subscript ผ่าน optional chaining
let firstElement = optArray?[0]  // Optional(1)
let appleCount = optDict?["apple"]  // Optional(1)

print(firstElement ?? 0)   // 1
print(appleCount ?? 0)     // 1

optArray = nil
let nilElement = optArray?[0]  // nil (ไม่ crash!)
print(nilElement ?? -1)  // -1
```

---

## 8.7 Nil-Coalescing Operator (??)

**Nil-coalescing operator** `??` ให้ค่า default เมื่อ Optional เป็น nil

### รูปแบบพื้นฐาน

```swift
// รูปแบบ: optional ?? defaultValue
var optName: String? = nil
let name = optName ?? "ผู้ใช้ไม่ระบุชื่อ"
print(name) // ผู้ใช้ไม่ระบุชื่อ

optName = "สมชาย"
let nameWithValue = optName ?? "ผู้ใช้ไม่ระบุชื่อ"
print(nameWithValue) // สมชาย
```

### เปรียบเทียบ ?? กับ if let

```swift
var optScore: Int? = 85

// ด้วย if let
let score1: Int
if let s = optScore {
    score1 = s
} else {
    score1 = 0
}

// ด้วย ?? (กระชับกว่ามาก)
let score2 = optScore ?? 0

print(score1) // 85
print(score2) // 85
```

### Chaining ??

```swift
var first: String? = nil
var second: String? = nil
var third: String? = "พบแล้ว!"

// Chain nil-coalescing operators
let result = first ?? second ?? third ?? "ไม่มีค่าเลย"
print(result) // พบแล้ว!

// ตัวอย่างในชีวิตจริง: user preferences
var userTheme: String? = nil
var systemTheme: String? = nil
var defaultTheme = "light"

let currentTheme = userTheme ?? systemTheme ?? defaultTheme
print(currentTheme) // light
```

### ?? กับ Function Calls

```swift
func getUsername() -> String? {
    return nil  // จำลองว่าไม่มี username
}

func getGuestName() -> String? {
    return "Guest User"
}

// เรียกฟังก์ชันที่สองเฉพาะเมื่อตัวแรกคืน nil (lazy evaluation)
let displayName = getUsername() ?? getGuestName() ?? "Unknown"
print(displayName) // Guest User
```

### ?? กับ Assignment

```swift
var count: Int? = nil

// ไม่ทำงานอย่างที่คิด - ?? จะ unwrap เท่านั้น
// let result = count ?? 0  // result เป็น 0 แต่ count ยังเป็น nil

// ถ้าต้องการ assign ค่า default ให้ตัวแปร
if count == nil {
    count = 0
}

// หรือใช้ Optional chaining กับ assignment (Swift 5.7+)
count = count ?? 0
print(count ?? -1) // 0
```

---

## 8.8 Force Unwrapping (!)

**Force unwrapping** ด้วย `!` บังคับ unwrap Optional โดยไม่ตรวจสอบ nil

### รูปแบบพื้นฐาน

```swift
var optionalAge: Int? = 25

// Force unwrap - อันตราย! ถ้า nil จะ crash
let age = optionalAge!
print(age) // 25

// ถ้าเป็น nil จะ crash!
optionalAge = nil
// let crashAge = optionalAge!  // Fatal error: Unexpectedly found nil while unwrapping
```

### เมื่อควรใช้ Force Unwrapping?

```swift
// ✅ ใช้ได้: เมื่อมั่นใจ 100% ว่าไม่เป็น nil
let url = URL(string: "https://www.google.com")!  // URL นี้ valid แน่นอน

// ✅ ใช้ได้: หลัง check nil แล้ว
var optNumber: Int? = 42
if optNumber != nil {
    print(optNumber!)  // ปลอดภัยเพราะ check แล้ว
}

// ❌ ไม่ดี: Force unwrap โดยไม่ check
var unsafeOptional: String? = nil
// print(unsafeOptional!)  // อันตราย!
```

### ทำไมควรหลีกเลี่ยง?

```swift
// ตัวอย่างโค้ดที่อันตราย
func parseAge(from string: String) -> Int {
    return Int(string)!  // Crash ถ้า string ไม่ใช่ตัวเลข!
}

// ดีกว่า:
func parseAgeSafe(from string: String) -> Int? {
    return Int(string)
}

// หรือ:
func parseAgeWithDefault(from string: String) -> Int {
    return Int(string) ?? 0
}

print(parseAgeWithDefault(from: "25"))    // 25
print(parseAgeWithDefault(from: "abc"))   // 0 (ไม่ crash)
```

---

## 8.9 Implicitly Unwrapped Optionals

**Implicitly unwrapped optional** (IUO) ประกาศด้วย `!` หลัง type ซึ่งจะถูก unwrap อัตโนมัติเมื่อใช้

### รูปแบบพื้นฐาน

```swift
// ประกาศ implicitly unwrapped optional
var implicitName: String! = "สมชาย"

// ใช้งานโดยไม่ต้อง unwrap
print(implicitName)       // Optional("สมชาย") เมื่อ print
print(implicitName + " !")  // สมชาย ! (auto-unwrapped)

// ยังสามารถ assign nil ได้
implicitName = nil
// print(implicitName + " !")  // Crash! nil ถูก auto-unwrap
```

### เมื่อควรใช้ IUO?

```swift
// 1. IBOutlets ใน iOS development
// @IBOutlet var label: UILabel!  // UI จะถูกสร้างก่อนใช้

// 2. Properties ที่ initialize ใน setup แต่ไม่ใช่ใน init
class Database {
    var connection: DatabaseConnection!
    
    func setup() {
        connection = DatabaseConnection(host: "localhost")
    }
    
    func query(_ sql: String) -> [Row] {
        // connection ถูก setup แล้วแน่นอนก่อนใช้
        return connection.execute(sql)
    }
}

// 3. Unit testing - setup ก่อน test
class MyTests {
    var sut: SystemUnderTest!
    
    func setUp() {
        sut = SystemUnderTest()
    }
    
    func testSomething() {
        // sut มีค่าแน่นอนหลัง setUp
        // sut.doSomething()
    }
}
```

### ข้อควรระวัง

```swift
// IUO ยังสามารถ check nil ได้
var iuo: String! = nil

// ตรวจสอบก่อนใช้ - ดีกว่า crash
if iuo != nil {
    print(iuo)
}

// หรือใช้ optional binding
if let value = iuo {
    print(value)
}
```

---

## 8.10 Optional Pattern Matching

Swift มีวิธีหลายแบบในการ match Optional ด้วย patterns

### Pattern Matching ใน switch

```swift
var optScore: Int? = 85

switch optScore {
case .some(let score) where score >= 90:
    print("ดีเยี่ยม: \(score)")
case .some(let score) where score >= 70:
    print("ผ่าน: \(score)")
case .some(let score):
    print("ไม่ผ่าน: \(score)")
case .none:
    print("ไม่มีคะแนน")
}

// รูปแบบสั้นกว่า
switch optScore {
case let score? where score >= 90:
    print("ดีเยี่ยม: \(score)")
case let score?:
    print("คะแนน: \(score)")
case nil:
    print("ไม่มีคะแนน")
}
```

### if case

```swift
var optValue: Int? = 42

// ใช้ if case สำหรับ pattern matching แบบง่าย
if case let value? = optValue {
    print("ค่าคือ: \(value)")
}

if case .some(let value) = optValue {
    print("some value: \(value)")
}
```

### Pattern ใน for loop

```swift
let optionalNumbers: [Int?] = [1, nil, 3, nil, 5, 6, nil, 8]

// ข้าม nil ใน for loop ด้วย pattern matching
for case let number? in optionalNumbers {
    print(number, terminator: " ")
}
print() // 1 3 5 6 8

// เทียบกับ compactMap
let nonNilNumbers = optionalNumbers.compactMap { $0 }
print(nonNilNumbers) // [1, 3, 5, 6, 8]
```

---

## 8.11 Multiple Optional Binding

การ unwrap Optional หลายตัวพร้อมกัน

### if let หลายตัว

```swift
var firstName: String? = "สม"
var lastName: String? = "ชาย"
var email: String? = "somchai@example.com"
var phone: String? = nil

// Unwrap ทั้งหมดพร้อมกัน
if let first = firstName,
   let last = lastName,
   let email = email {
    print("ชื่อ: \(first)\(last), Email: \(email)")
}

// ถ้าตัวใดตัวหนึ่งเป็น nil จะไม่ทำงาน
if let first = firstName,
   let phone = phone {  // phone เป็น nil
    print("จะไม่ถูก print")
} else {
    print("ข้อมูลไม่ครบ: ไม่มีเบอร์โทร")
}
```

### guard let หลายตัว

```swift
func sendMessage(to recipient: String?, with text: String?, at time: Date?) -> Bool {
    guard let recipient = recipient,
          let text = text,
          let time = time else {
        print("ข้อมูลไม่ครบถ้วน")
        return false
    }
    
    // ใช้ recipient, text, time ได้ตรงนี้ทั้งหมด
    print("ส่งข้อความ '\(text)' ไปหา \(recipient) เวลา \(time)")
    return true
}

_ = sendMessage(to: "สมชาย", with: "สวัสดี", at: Date())
_ = sendMessage(to: nil, with: "ข้อความ", at: Date())
```

### Mixed Conditions

```swift
var age: Int? = 25
var name: String? = "สมชาย"

// ผสม optional binding กับ boolean condition
if let age = age, age >= 18,
   let name = name, !name.isEmpty {
    print("\(name) อายุ \(age) ปี ผ่านเงื่อนไข")
}
```

---

## 8.12 Optional map และ flatMap

Optionals มี `map` และ `flatMap` ที่ใช้แปลงค่าอย่างปลอดภัย

### Optional map

```swift
var optNumber: Int? = 5

// ถ้า optNumber มีค่า จะ apply transform
// ถ้า nil จะคืน nil
let doubled = optNumber.map { $0 * 2 }
print(doubled)  // Optional(10)

optNumber = nil
let nilResult = optNumber.map { $0 * 2 }
print(nilResult)  // nil

// เปรียบเทียบกับ if let
var score: Int? = 85
let grade1: String?
if let s = score {
    grade1 = s >= 70 ? "ผ่าน" : "ไม่ผ่าน"
} else {
    grade1 = nil
}

// ด้วย map (กระชับกว่า)
let grade2 = score.map { $0 >= 70 ? "ผ่าน" : "ไม่ผ่าน" }

print(grade1 ?? "nil")  // ผ่าน
print(grade2 ?? "nil")  // ผ่าน
```

### Optional flatMap

ใช้เมื่อ transform function คืน Optional

```swift
var optString: String? = "42"

// map จะได้ Optional<Optional<Int>>
let mapResult: Int?? = optString.map { Int($0) }
print(mapResult as Any)  // Optional(Optional(42))

// flatMap จะ flatten เป็น Optional<Int>
let flatMapResult: Int? = optString.flatMap { Int($0) }
print(flatMapResult as Any)  // Optional(42)

// ตัวอย่างชัดขึ้น
var phoneString: String? = "0812345678"

// ดึงเฉพาะตัวเลข
let phoneNumbers = phoneString.flatMap { str -> String? in
    let digits = str.filter { $0.isNumber }
    return digits.isEmpty ? nil : digits
}
print(phoneNumbers ?? "ไม่มีหมายเลข")  // 0812345678
```

### ตัวอย่างที่ซับซ้อน

```swift
struct Config {
    var serverURL: String?
    var port: String?
    var timeout: String?
}

func createEndpoint(from config: Config) -> URL? {
    guard let serverURL = config.serverURL else { return nil }
    guard let port = config.port,
          let portInt = Int(port) else {
        return URL(string: serverURL)
    }
    return URL(string: "\(serverURL):\(portInt)")
}

let config = Config(
    serverURL: "https://api.example.com",
    port: "8080",
    timeout: "30"
)

let endpoint = createEndpoint(from: config)
print(endpoint?.absoluteString ?? "Invalid config")
```

---

## 8.13 Optionals ใน switch Statements

```swift
var optValue: String? = "hello"

switch optValue {
case "hello":
    print("พบ hello")
case "world":
    print("พบ world")
case let value?:
    print("พบค่าอื่น: \(value)")
case nil:
    print("ไม่มีค่า")
}
```

### Pattern Matching แบบซับซ้อน

```swift
enum Status {
    case active
    case inactive
    case pending
}

var userStatus: Status? = .active

switch userStatus {
case .some(.active):
    print("ผู้ใช้ active")
case .some(.inactive):
    print("ผู้ใช้ inactive")
case .some(.pending):
    print("รอการอนุมัติ")
case .none:
    print("ไม่มีข้อมูลสถานะ")
}
```

### switch กับ Tuple ที่มี Optional

```swift
var username: String? = "สมชาย"
var isAdmin: Bool? = true

switch (username, isAdmin) {
case let (name?, admin?) where admin:
    print("Admin: \(name)")
case let (name?, _):
    print("User: \(name)")
case (nil, _):
    print("ไม่ทราบชื่อ")
}
```

---

## 8.14 Optional Casting (as?, as!)

การแปลง type ด้วย optional casting

### as? (Safe Casting)

```swift
class Animal {
    var name: String
    init(name: String) { self.name = name }
}

class Dog: Animal {
    func bark() { print("\(name) says: Woof!") }
}

class Cat: Animal {
    func meow() { print("\(name) says: Meow!") }
}

let animals: [Animal] = [Dog(name: "บัดดี้"), Cat(name: "วิสกี้"), Dog(name: "แม็กซ์")]

for animal in animals {
    if let dog = animal as? Dog {
        dog.bark()
    } else if let cat = animal as? Cat {
        cat.meow()
    }
}
```

### as! (Forced Casting)

```swift
let animal: Animal = Dog(name: "เร็กซ์")

// Force cast - crash ถ้า animal ไม่ใช่ Dog
let dog = animal as! Dog
dog.bark()

// as? ปลอดภัยกว่า
if let dog = animal as? Dog {
    dog.bark()
} else {
    print("ไม่ใช่สุนัข")
}
```

### Downcasting กับ Collections

```swift
let mixed: [Any] = [1, "hello", 3.14, true, "world", 42]

// ดึงเฉพาะ String
let strings = mixed.compactMap { $0 as? String }
print(strings) // ["hello", "world"]

// ดึงเฉพาะ Int
let ints = mixed.compactMap { $0 as? Int }
print(ints) // [1, 42]
```

---

## 8.15 Nested Optionals

Optional ของ Optional (ซ้อนกัน)

### ทำความเข้าใจ Nested Optionals

```swift
var regularInt: Int = 5                    // Int
var optInt: Int? = 5                       // Optional<Int>
var nestedOptInt: Int?? = 5                // Optional<Optional<Int>>

print(type(of: regularInt))               // Int
print(type(of: optInt))                   // Optional<Int>
print(type(of: nestedOptInt))             // Optional<Optional<Int>>

// การ unwrap nested optional
if let outer = nestedOptInt {              // outer เป็น Int?
    if let inner = outer {                 // inner เป็น Int
        print("ค่าคือ: \(inner)")
    }
}
```

### เมื่อ Nested Optionals เกิดขึ้น

```swift
// Dictionary ที่มี Optional values
var scores: [String: Int?] = [
    "สมชาย": 85,
    "สมหญิง": nil,   // มี key แต่ value เป็น nil
]

// การเข้าถึง: dict[key] คืน Optional<Optional<Int>>
let somchaiScore = scores["สมชาย"]   // Optional(Optional(85))
let somyingScore = scores["สมหญิง"]  // Optional(nil)
let unknownScore = scores["ไม่มีคน"]  // nil (key ไม่มี)

print(somchaiScore as Any)   // Optional(Optional(85))
print(somyingScore as Any)   // Optional(nil)
print(unknownScore as Any)   // nil
```

### ลดความซับซ้อนด้วย flatMap

```swift
// Flatten nested optional
var nestedOpt: Int?? = 42
let flattened: Int? = nestedOpt.flatMap { $0 }
print(flattened as Any)  // Optional(42)

// หรือ unwrap สองชั้น
if case let value?? = nestedOpt {
    print("ค่า: \(value)")  // 42
}
```

---

## 8.16 Optional vs Optional<Optional>

### ความแตกต่างที่สำคัญ

```swift
// สามกรณีที่แตกต่างกัน
var a: Int?? = nil         // nil เองก็เป็น Optional<Optional<Int>>
var b: Int?? = .some(nil)  // outer มีค่า แต่ inner เป็น nil
var c: Int?? = .some(.some(42))  // มีค่า 42

// การตรวจสอบ
print(a == nil)  // true
print(b == nil)  // false! outer ไม่ nil
print(c == nil)  // false

// Unwrap แต่ละชั้น
if let outer = b {     // outer เป็น Int? (nil)
    if let inner = outer {
        print(inner)
    } else {
        print("outer มีค่าแต่ inner เป็น nil")  // พิมพ์นี้
    }
} else {
    print("outer เป็น nil")
}
```

### ใช้ในทางปฏิบัติ

```swift
// ตัวอย่าง: แยกความแตกต่างระหว่าง "ไม่ได้ส่งมา" กับ "ส่งมาเป็น nil"
struct UpdateRequest {
    var name: String??          // nil = ไม่ได้อัพเดท, .some(nil) = ลบออก, .some("value") = กำหนดค่า
    var email: String??
}

func processUpdate(_ request: UpdateRequest) {
    if let newName = request.name {
        if let name = newName {
            print("เปลี่ยนชื่อเป็น: \(name)")
        } else {
            print("ลบชื่อออก")
        }
    } else {
        print("ไม่เปลี่ยนแปลงชื่อ")
    }
}

let req1 = UpdateRequest(name: "สมหญิง", email: nil)
let req2 = UpdateRequest(name: .some(nil), email: nil)
let req3 = UpdateRequest(name: nil, email: nil)

processUpdate(req1)  // เปลี่ยนชื่อเป็น: สมหญิง
processUpdate(req2)  // ลบชื่อออก
processUpdate(req3)  // ไม่เปลี่ยนแปลงชื่อ
```

---

## 8.17 Common Optional Patterns

### Pattern 1: Default Value Pattern

```swift
struct UserPreferences {
    var theme: String?
    var language: String?
    var fontSize: Int?
}

class AppSettings {
    static let defaults = UserPreferences(
        theme: "light",
        language: "th",
        fontSize: 16
    )
    
    var userPrefs: UserPreferences = UserPreferences()
    
    var currentTheme: String {
        return userPrefs.theme ?? AppSettings.defaults.theme!
    }
    
    var currentLanguage: String {
        return userPrefs.language ?? AppSettings.defaults.language!
    }
    
    var currentFontSize: Int {
        return userPrefs.fontSize ?? AppSettings.defaults.fontSize!
    }
}

let settings = AppSettings()
print(settings.currentTheme)     // light
print(settings.currentFontSize)  // 16

settings.userPrefs.theme = "dark"
print(settings.currentTheme)     // dark
```

### Pattern 2: Optional Chaining กับ Default

```swift
class User {
    var profile: Profile?
    var name: String
    
    init(name: String) {
        self.name = name
    }
}

class Profile {
    var bio: String?
    var avatarURL: String?
    var website: String?
}

let user = User(name: "สมชาย")
user.profile = Profile()
user.profile?.bio = "นักพัฒนา iOS"

// Optional chaining + nil-coalescing
let bio = user.profile?.bio ?? "ยังไม่มีคำแนะนำตัว"
let avatarURL = user.profile?.avatarURL ?? "https://default-avatar.com/avatar.png"
let website = user.profile?.website ?? "ไม่มี website"

print(bio)        // นักพัฒนา iOS
print(avatarURL)  // https://default-avatar.com/avatar.png
print(website)    // ไม่มี website
```

### Pattern 3: Try-Catch กับ Optional

```swift
enum ParseError: Error {
    case invalidFormat
    case outOfRange
}

func parseAge(_ string: String) throws -> Int {
    guard let age = Int(string) else {
        throw ParseError.invalidFormat
    }
    guard age >= 0 && age <= 150 else {
        throw ParseError.outOfRange
    }
    return age
}

// try? คืน Optional (nil ถ้า throw)
let age1 = try? parseAge("25")   // Optional(25)
let age2 = try? parseAge("abc")  // nil
let age3 = try? parseAge("200")  // nil

print(age1 ?? "invalid")  // 25
print(age2 ?? "invalid")  // invalid
print(age3 ?? "invalid")  // invalid

// try! (อันตราย - crash ถ้า throw)
// let age4 = try! parseAge("abc")  // Fatal error!
```

---

## 8.18 Best Practices สำหรับ Optionals

### หลักการทั่วไป

```swift
// ✅ ดี: ใช้ guard let สำหรับ early exit
func processUser(user: User?) {
    guard let user = user else {
        print("ไม่มีข้อมูล user")
        return
    }
    // ทำงานกับ user ที่ unwrap แล้ว
    print("ประมวลผล user: \(user.name)")
}

// ✅ ดี: ใช้ ?? สำหรับ default values
let displayName = user?.name ?? "ผู้ใช้ไม่ระบุชื่อ"

// ✅ ดี: ใช้ optional chaining
let profileBio = user?.profile?.bio

// ❌ ไม่ดี: Force unwrap โดยไม่จำเป็น
// let unsafeName = user!.name

// ❌ ไม่ดี: Pyramid of doom
if let a = optA {
    if let b = optB {
        if let c = optC {
            // โค้ดที่ดี
        }
    }
}

// ✅ ดี: ใช้ multiple binding แทน
if let a = optA, let b = optB, let c = optC {
    // โค้ดที่ดี
}
```

### เมื่อควรใช้แต่ละวิธี

```swift
// if let: เมื่อต้องทำงานทั้งกรณี nil และไม่ nil
var optData: Data? = Data()
if let data = optData {
    print("มีข้อมูล \(data.count) bytes")
} else {
    print("ไม่มีข้อมูล")
}

// guard let: เมื่อ nil คือ error condition
func loadData(from url: URL?) throws {
    guard let url = url else {
        throw URLError(.badURL)
    }
    // ทำงานต่อกับ url
}

// ??: เมื่อต้องการ default value
let count = optData?.count ?? 0

// optional chaining: เมื่อเข้าถึง properties แบบ chain
let firstByte = optData?.first  // Optional<UInt8>
```

---

## 8.19 Common Mistakes to Avoid

### Mistake 1: ใช้ Force Unwrap โดยไม่จำเป็น

```swift
// ❌ อันตราย
var name: String? = getUserName()
print(name!)  // Crash ถ้า nil

// ✅ ปลอดภัย
if let name = getUserName() {
    print(name)
}

// หรือ
print(getUserName() ?? "ไม่ทราบชื่อ")

func getUserName() -> String? { return nil }
```

### Mistake 2: การ Compare Optional กับ Value

```swift
var optInt: Int? = 5

// ✅ ถูกต้อง
if optInt == 5 {
    print("ค่าเป็น 5")
}

// ✅ ถูกต้องเช่นกัน
if let value = optInt, value == 5 {
    print("ค่าเป็น 5")
}

// ⚠️ ระวัง: การเปรียบเทียบ Optional กับ Optional
var a: Int? = 5
var b: Int? = 5
var c: Int? = nil

print(a == b)    // true
print(a == c)    // false
print(b == c)    // false
print(c == nil)  // true
```

### Mistake 3: Pyramid of Doom

```swift
// ❌ ยากอ่าน: Pyramid of doom
func processRequest(data: Data?) {
    if let data = data {
        if let json = try? JSONSerialization.jsonObject(with: data) as? [String: Any] {
            if let name = json["name"] as? String {
                if let age = json["age"] as? Int {
                    print("ชื่อ: \(name), อายุ: \(age)")
                }
            }
        }
    }
}

// ✅ ดีกว่า: ใช้ guard
func processRequestClean(data: Data?) {
    guard let data = data else { return }
    guard let json = try? JSONSerialization.jsonObject(with: data) as? [String: Any] else { return }
    guard let name = json["name"] as? String,
          let age = json["age"] as? Int else { return }
    
    print("ชื่อ: \(name), อายุ: \(age)")
}
```

### Mistake 4: Ignoring Optional Return Values

```swift
var array = [1, 2, 3, 4, 5]

// ⚠️ ระวัง: first/last/min/max คืน Optional
let first = array.first  // Optional(1)
let last = array.last    // Optional(5)

// ถ้า array ว่าง จะได้ nil
let emptyArray: [Int] = []
let emptyFirst = emptyArray.first  // nil

// ✅ จัดการ Optional
if let firstElement = array.first {
    print("Element แรก: \(firstElement)")
}

// หรือ
let safeFirst = array.first ?? 0
```

---

## 8.20 Practical Exercises

### Exercise 1: Safe JSON Parser

```swift
// สร้าง JSON parser ที่ใช้ Optional อย่างถูกต้อง
typealias JSON = [String: Any]

extension Dictionary where Key == String {
    func string(for key: String) -> String? {
        return self[key] as? String
    }
    
    func int(for key: String) -> Int? {
        return self[key] as? Int
    }
    
    func bool(for key: String) -> Bool? {
        return self[key] as? Bool
    }
    
    func array<T>(for key: String) -> [T]? {
        return self[key] as? [T]
    }
    
    func dict(for key: String) -> [String: Any]? {
        return self[key] as? [String: Any]
    }
}

// ใช้งาน
let userJSON: JSON = [
    "name": "สมชาย",
    "age": 25,
    "isVerified": true,
    "email": "somchai@example.com",
    "scores": [85, 90, 78],
    "address": [
        "city": "กรุงเทพมหานคร",
        "zip": "10110"
    ]
]

let name = userJSON.string(for: "name") ?? "ไม่ทราบ"
let age = userJSON.int(for: "age") ?? 0
let isVerified = userJSON.bool(for: "isVerified") ?? false
let scores = userJSON.array(for: "scores") as [Int]? ?? []
let address = userJSON.dict(for: "address")
let city = address?.string(for: "city") ?? "ไม่ทราบ"

print("ชื่อ: \(name)")
print("อายุ: \(age)")
print("ยืนยันแล้ว: \(isVerified)")
print("คะแนน: \(scores)")
print("เมือง: \(city)")
```

### Exercise 2: Optional Chaining กับ Complex Objects

```swift
// สร้างโมเดลซับซ้อนและใช้ optional chaining
struct Product {
    var id: Int
    var name: String
    var price: Double?
    var category: Category?
    var reviews: [Review]?
}

struct Category {
    var name: String
    var parentCategory: Category?
}

struct Review {
    var rating: Int
    var comment: String?
    var reviewer: String?
}

// สร้างข้อมูล
let electronics = Category(name: "อิเล็กทรอนิกส์", parentCategory: nil)
let phones = Category(name: "โทรศัพท์", parentCategory: electronics)

let review1 = Review(rating: 5, comment: "ดีมาก!", reviewer: "สมชาย")
let review2 = Review(rating: 4, comment: nil, reviewer: nil)

let iphone = Product(
    id: 1,
    name: "iPhone 15",
    price: 29900.0,
    category: phones,
    reviews: [review1, review2]
)

// Optional chaining
let parentCategoryName = iphone.category?.parentCategory?.name
print("หมวดหมู่หลัก: \(parentCategoryName ?? "ไม่มี")")

let avgRating = iphone.reviews.map { reviews in
    Double(reviews.reduce(0) { $0 + $1.rating }) / Double(reviews.count)
}
print("คะแนนเฉลี่ย: \(avgRating.map { String(format: "%.1f", $0) } ?? "ไม่มีรีวิว")")

let firstReviewerName = iphone.reviews?.first?.reviewer
print("ผู้รีวิวแรก: \(firstReviewerName ?? "ไม่ระบุ")")
```

### Exercise 3: Optional ใน Network Requests

```swift
// จำลอง network response handling ด้วย Optional
struct APIResponse<T> {
    var data: T?
    var error: String?
    var statusCode: Int
}

struct UserData {
    var id: Int
    var name: String
    var email: String
}

func parseUserResponse(_ json: [String: Any]) -> UserData? {
    guard let id = json["id"] as? Int,
          let name = json["name"] as? String,
          let email = json["email"] as? String else {
        return nil
    }
    return UserData(id: id, name: name, email: email)
}

func handleResponse(_ response: APIResponse<[String: Any]>) {
    guard response.statusCode == 200 else {
        let errorMsg = response.error ?? "Unknown error"
        print("ผิดพลาด (\(response.statusCode)): \(errorMsg)")
        return
    }
    
    guard let json = response.data else {
        print("ไม่มีข้อมูล")
        return
    }
    
    guard let user = parseUserResponse(json) else {
        print("ไม่สามารถแปลงข้อมูล")
        return
    }
    
    print("ผู้ใช้: \(user.name) (ID: \(user.id), Email: \(user.email))")
}

// ทดสอบ
let successResponse = APIResponse<[String: Any]>(
    data: ["id": 1, "name": "สมชาย", "email": "somchai@example.com"],
    error: nil,
    statusCode: 200
)

let errorResponse = APIResponse<[String: Any]>(
    data: nil,
    error: "Not Found",
    statusCode: 404
)

handleResponse(successResponse)  // ผู้ใช้: สมชาย
handleResponse(errorResponse)     // ผิดพลาด (404): Not Found
```

### Exercise 4: Optional ใน User Input Validation

```swift
// Form validation ด้วย Optional
struct RegistrationForm {
    var username: String?
    var password: String?
    var confirmPassword: String?
    var email: String?
    var age: String?
}

enum ValidationError: Error, CustomStringConvertible {
    case emptyUsername
    case shortPassword
    case passwordMismatch
    case invalidEmail
    case invalidAge
    case underage
    
    var description: String {
        switch self {
        case .emptyUsername: return "กรุณากรอก username"
        case .shortPassword: return "Password ต้องมีอย่างน้อย 8 ตัวอักษร"
        case .passwordMismatch: return "Password ไม่ตรงกัน"
        case .invalidEmail: return "Email ไม่ถูกต้อง"
        case .invalidAge: return "กรุณากรอกอายุที่ถูกต้อง"
        case .underage: return "ต้องอายุ 18 ปีขึ้นไป"
        }
    }
}

func validateForm(_ form: RegistrationForm) -> Result<Void, [ValidationError]> {
    var errors: [ValidationError] = []
    
    // ตรวจสอบ username
    if let username = form.username, !username.isEmpty {
        // OK
    } else {
        errors.append(.emptyUsername)
    }
    
    // ตรวจสอบ password
    if let password = form.password, password.count >= 8 {
        // ตรวจสอบ confirm password
        if let confirm = form.confirmPassword, password != confirm {
            errors.append(.passwordMismatch)
        }
    } else {
        errors.append(.shortPassword)
    }
    
    // ตรวจสอบ email (แบบง่าย)
    if let email = form.email, email.contains("@") && email.contains(".") {
        // OK
    } else {
        errors.append(.invalidEmail)
    }
    
    // ตรวจสอบ age
    if let ageString = form.age, let age = Int(ageString) {
        if age < 18 {
            errors.append(.underage)
        }
    } else {
        errors.append(.invalidAge)
    }
    
    return errors.isEmpty ? .success(()) : .failure(errors)
}

// ทดสอบ
let validForm = RegistrationForm(
    username: "somchai",
    password: "password123",
    confirmPassword: "password123",
    email: "somchai@example.com",
    age: "25"
)

let invalidForm = RegistrationForm(
    username: "",
    password: "short",
    confirmPassword: "notmatch",
    email: "invalid",
    age: "abc"
)

switch validateForm(validForm) {
case .success:
    print("ลงทะเบียนสำเร็จ!")
case .failure(let errors):
    print("ข้อผิดพลาด:")
    errors.forEach { print("  - \($0)") }
}

print()

switch validateForm(invalidForm) {
case .success:
    print("ลงทะเบียนสำเร็จ!")
case .failure(let errors):
    print("ข้อผิดพลาด:")
    errors.forEach { print("  - \($0)") }
}
```

---

## 8.21 Real-World Examples

### Example 1: การจัดการ API Response

```swift
import Foundation

// Model
struct Post: Codable {
    let id: Int
    let title: String
    let body: String
    let userId: Int
}

// API Service
class PostService {
    func fetchPost(id: Int, completion: @escaping (Post?, Error?) -> Void) {
        guard let url = URL(string: "https://jsonplaceholder.typicode.com/posts/\(id)") else {
            completion(nil, URLError(.badURL))
            return
        }
        
        URLSession.shared.dataTask(with: url) { data, response, error in
            // จัดการ error
            if let error = error {
                completion(nil, error)
                return
            }
            
            // ตรวจสอบ HTTP status
            guard let httpResponse = response as? HTTPURLResponse,
                  (200...299).contains(httpResponse.statusCode) else {
                completion(nil, URLError(.badServerResponse))
                return
            }
            
            // Decode JSON
            guard let data = data else {
                completion(nil, URLError(.zeroByteResource))
                return
            }
            
            do {
                let post = try JSONDecoder().decode(Post.self, from: data)
                DispatchQueue.main.async {
                    completion(post, nil)
                }
            } catch {
                completion(nil, error)
            }
        }.resume()
    }
}
```

### Example 2: UserDefaults กับ Optional

```swift
// Extension สำหรับ UserDefaults ที่ใช้ Optional อย่างถูกต้อง
extension UserDefaults {
    func optionalInt(forKey key: String) -> Int? {
        return object(forKey: key) as? Int
    }
    
    func optionalBool(forKey key: String) -> Bool? {
        return object(forKey: key) as? Bool
    }
    
    func optionalString(forKey key: String) -> String? {
        return string(forKey: key)
    }
    
    func setOptional<T>(_ value: T?, forKey key: String) {
        if let value = value {
            set(value, forKey: key)
        } else {
            removeObject(forKey: key)
        }
    }
}

// ใช้งาน
class UserSession {
    static let shared = UserSession()
    private let defaults = UserDefaults.standard
    
    var userID: Int? {
        get { defaults.optionalInt(forKey: "userID") }
        set { defaults.setOptional(newValue, forKey: "userID") }
    }
    
    var username: String? {
        get { defaults.optionalString(forKey: "username") }
        set { defaults.setOptional(newValue, forKey: "username") }
    }
    
    var isLoggedIn: Bool {
        return userID != nil && username != nil
    }
    
    func login(userID: Int, username: String) {
        self.userID = userID
        self.username = username
    }
    
    func logout() {
        userID = nil
        username = nil
    }
}

let session = UserSession.shared
print("Logged in: \(session.isLoggedIn)")  // false

session.login(userID: 42, username: "สมชาย")
print("Logged in: \(session.isLoggedIn)")  // true
print("User: \(session.username ?? "unknown")")  // สมชาย

session.logout()
print("Logged in: \(session.isLoggedIn)")  // false
```

### Example 3: Search Functionality

```swift
// ระบบค้นหาที่ใช้ Optional อย่างเหมาะสม
struct SearchResult {
    let items: [String]
    let totalCount: Int
    let nextPageToken: String?  // nil ถ้าไม่มีหน้าถัดไป
}

class SearchEngine {
    private var database: [String] = [
        "Swift Programming", "iOS Development", "SwiftUI",
        "UIKit", "Core Data", "Combine", "async/await",
        "Protocol-Oriented Programming", "Generics",
        "Closures", "Optionals", "Error Handling"
    ]
    
    func search(
        query: String?,
        page: Int = 1,
        pageSize: Int = 3
    ) -> SearchResult? {
        // Return nil ถ้า query เป็น nil หรือว่าง
        guard let query = query, !query.isEmpty else {
            return nil
        }
        
        let lowercasedQuery = query.lowercased()
        let allResults = database.filter {
            $0.lowercased().contains(lowercasedQuery)
        }
        
        let startIndex = (page - 1) * pageSize
        guard startIndex < allResults.count else {
            return SearchResult(items: [], totalCount: allResults.count, nextPageToken: nil)
        }
        
        let endIndex = min(startIndex + pageSize, allResults.count)
        let pageItems = Array(allResults[startIndex..<endIndex])
        
        let hasNextPage = endIndex < allResults.count
        let nextToken = hasNextPage ? "page_\(page + 1)" : nil
        
        return SearchResult(items: pageItems, totalCount: allResults.count, nextPageToken: nextToken)
    }
}

// ใช้งาน
let engine = SearchEngine()

// ค้นหาปกติ
if let results = engine.search(query: "swift") {
    print("พบ \(results.totalCount) ผลลัพธ์:")
    results.items.forEach { print("  - \($0)") }
    
    if let nextPage = results.nextPageToken {
        print("หน้าถัดไป: \(nextPage)")
    } else {
        print("ไม่มีหน้าถัดไป")
    }
}

// ค้นหาด้วย nil query
let nilResult = engine.search(query: nil)
print("\nค้นหาด้วย nil: \(nilResult == nil ? "คืน nil" : "มีผลลัพธ์")")

// ค้นหาด้วย empty string
let emptyResult = engine.search(query: "")
print("ค้นหาด้วยข้อความว่าง: \(emptyResult == nil ? "คืน nil" : "มีผลลัพธ์")")
```

---

## 8.22 สรุป

### สิ่งที่ได้เรียนรู้

1. **Optionals** - วิธีการแสดง "ไม่มีค่า" ใน Swift อย่างปลอดภัย
2. **Optional declaration** - ใช้ `?` หลัง type
3. **nil** - ค่าที่แสดงการไม่มีค่า
4. **if let** - Unwrap Optional อย่างปลอดภัยพร้อม else clause
5. **guard let** - Early exit เมื่อ Optional เป็น nil
6. **Optional chaining** - เข้าถึง properties/methods ด้วย `?`
7. **??** - ให้ค่า default เมื่อ Optional เป็น nil
8. **Force unwrapping** - `!` ใช้ด้วยความระมัดระวัง
9. **IUO** - Implicitly Unwrapped Optionals สำหรับกรณีพิเศษ
10. **Optional map/flatMap** - แปลงค่าใน Optional อย่างปลอดภัย

### Quick Reference

| Operation | Syntax | ผลลัพธ์ |
|-----------|--------|---------|
| ประกาศ Optional | `var x: Int?` | Optional ที่ nil |
| Check nil | `x == nil` | Bool |
| if let unwrap | `if let v = x { }` | Unwrapped ใน scope |
| guard let | `guard let v = x else { return }` | Unwrapped ตลอดไป |
| Optional chaining | `x?.property` | Optional |
| Nil-coalescing | `x ?? default` | Non-optional |
| Force unwrap | `x!` | Non-optional (อันตราย) |
| map | `x.map { $0 + 1 }` | Optional |
| flatMap | `x.flatMap { fn($0) }` | Optional |

### Best Practices สรุป

```swift
// ✅ ใช้ optional binding
if let value = optionalValue {
    use(value)
}

// ✅ ใช้ guard สำหรับ early exit
guard let value = optionalValue else { return }

// ✅ ใช้ ?? สำหรับ defaults
let result = optionalValue ?? defaultValue

// ✅ ใช้ optional chaining
let property = object?.property?.subProperty

// ❌ หลีกเลี่ยง force unwrap
// let dangerous = optionalValue!

// ❌ หลีกเลี่ยง nested if let
// if let a = optA { if let b = optB { ... } }
```

---

## แบบฝึกหัด

### ระดับเริ่มต้น

1. ประกาศตัวแปร Optional ของแต่ละ type (String, Int, Double, Bool) และลองกำหนดค่า nil
2. เขียนฟังก์ชันที่รับ `String?` และพิมพ์ความยาวของ string หรือ "ไม่มีข้อมูล" ถ้าเป็น nil
3. สร้าง dictionary `[String: Int?]` และแสดงเฉพาะ values ที่ไม่ใช่ nil

### ระดับกลาง

4. สร้าง model สำหรับ order ที่มี optional fields และเขียนฟังก์ชัน validate
5. ใช้ optional chaining กับ nested objects (User -> Profile -> Settings)
6. เขียน extension สำหรับ Optional ที่มี `orThrow` method

### ระดับสูง

7. สร้าง type-safe key-value store ที่ใช้ Optional อย่างถูกต้อง
8. Implement `Optional<T>` ของตัวเองด้วย enum
9. สร้าง validator chain ที่ใช้ `Result<T, Error>` และจัดการ Optional อย่างครบถ้วน

---

*จบบทที่ 8: Optionals ใน Swift*
