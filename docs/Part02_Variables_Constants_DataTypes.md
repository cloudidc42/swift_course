# Part 02: Variables, Constants & Data Types — ตัวแปร ค่าคงที่ และชนิดข้อมูล

[![Swift](https://img.shields.io/badge/Swift-5.9+-orange?logo=swift)](https://swift.org)
[![Level](https://img.shields.io/badge/Level-Beginner-green)](https://swift.org)
[![Part](https://img.shields.io/badge/Part-02%20of%20100-blue)](../README.md)

---

## 📋 สิ่งที่จะได้เรียนในตอนนี้

- `var` vs `let` — ตัวแปรและค่าคงที่
- Type Inference — การอนุมานชนิดข้อมูล
- Explicit Type Annotations — การระบุชนิดข้อมูลอย่างชัดเจน
- Integer Types — Int, Int8, Int16, Int32, Int64, UInt
- Floating Point Types — Float, Double, Float80
- Boolean Type — Bool
- Character Type — Character
- String Type — String
- Type Aliases — typealias
- Numeric Literals — ทศนิยม, ไบนารี, ออกทัล, เลขฐานสิบหก
- Numeric Type Conversion — การแปลงชนิดตัวเลข
- Overflow Operators — ตัวดำเนินการ Overflow
- Type Safety ใน Swift
- Naming Conventions — หลักการตั้งชื่อ
- Multiple Variable Declarations
- Constant Expressions
- Lazy Variables
- Computed Properties เบื้องต้น
- แบบฝึกหัดพร้อมเฉลย
- ตัวอย่างโลกจริง
- สรุป

---

## 1. ตัวแปร (Variables) vs ค่าคงที่ (Constants)

การเก็บข้อมูลในโปรแกรม Swift ทำผ่านสิ่งที่เรียกว่า **ตัวแปร (Variable)** และ **ค่าคงที่ (Constant)**

### 1.1 ตัวแปร (Variable) — `var`

**ตัวแปร** คือพื้นที่เก็บข้อมูลที่ **เปลี่ยนแปลงค่าได้** หลังจากกำหนดค่าแล้ว

```swift
// ประกาศตัวแปรด้วย var
var score = 0
print("คะแนนเริ่มต้น: \(score)")  // 0

// เปลี่ยนค่าได้
score = 10
print("หลังได้แต้ม: \(score)")    // 10

score = score + 5
print("หลังบวกเพิ่ม: \(score)")   // 15

// ตัวอย่างอื่นๆ
var playerName = "ผู้เล่น 1"
var health = 100
var isAlive = true

// เปลี่ยนค่า
playerName = "สมชาย"
health -= 30  // เหลือ 70
isAlive = health > 0  // true
```

### 1.2 ค่าคงที่ (Constant) — `let`

**ค่าคงที่** คือพื้นที่เก็บข้อมูลที่ **เปลี่ยนแปลงค่าไม่ได้** หลังจากกำหนดค่าแล้ว (Immutable)

```swift
// ประกาศค่าคงที่ด้วย let
let pi = 3.14159265358979
let appName = "Swift Learning"
let maxPlayers = 4

// ❌ ไม่สามารถเปลี่ยนค่าได้!
// pi = 3.14  // Error: cannot assign to value: 'pi' is a 'let' constant
// appName = "New Name"  // Error!

print("Pi = \(pi)")
print("App: \(appName)")
print("ผู้เล่นสูงสุด: \(maxPlayers) คน")
```

### 1.3 ทำไมต้องมีทั้ง var และ let?

นี่คือหนึ่งในหลักการสำคัญของ Swift: **Prefer Constants Over Variables**

```swift
// ❌ ไม่ดี — ใช้ var ทั้งหมด
var name = "สมหญิง"      // ถ้าไม่เปลี่ยน ไม่ควรใช้ var
var birthYear = 2000     // ปีเกิดไม่เปลี่ยน!
var PI = 3.14159         // Pi ไม่เปลี่ยน!

// ✅ ดีกว่า — ใช้ let สำหรับสิ่งที่ไม่เปลี่ยน
let personName = "สมหญิง"
let birthYear2 = 2000
let piValue = 3.14159

var currentAge = 2024 - birthYear2  // อายุเปลี่ยนได้
```

**เหตุผลที่ใช้ `let` แทน `var` เมื่อทำได้:**
1. **ความปลอดภัย** — ป้องกันการเปลี่ยนค่าโดยไม่ตั้งใจ
2. **ความชัดเจน** — บอกผู้อ่านโค้ดว่าค่านี้ไม่เปลี่ยน
3. **ประสิทธิภาพ** — Compiler สามารถ Optimize ได้ดีกว่า
4. **Thread Safety** — ค่าที่ไม่เปลี่ยนปลอดภัยสำหรับ Multi-threading

```swift
// ตัวอย่างจริง: ข้อมูลผู้ใช้
struct UserProfile {
    let userId: String      // ไม่ควรเปลี่ยน
    let createdDate: Date   // วันที่สร้างไม่เปลี่ยน
    var username: String    // เปลี่ยนได้
    var email: String       // เปลี่ยนได้
    var followersCount: Int // เปลี่ยนได้
}
```

---

## 2. Type Inference — การอนุมานชนิดข้อมูล

Swift เป็นภาษา **Strongly Typed** ทุกตัวแปรมีชนิดข้อมูล แต่เราไม่ต้องระบุเสมอไป Swift จะ **อนุมาน (Infer)** ชนิดข้อมูลให้อัตโนมัติ

### 2.1 ตัวอย่าง Type Inference

```swift
// Swift อนุมาน Type ให้อัตโนมัติ
let message = "Hello"      // String
let count = 42             // Int
let price = 99.99          // Double
let isActive = true        // Bool

// ดู Type ด้วย type(of:)
print(type(of: message))   // String
print(type(of: count))     // Int
print(type(of: price))     // Double
print(type(of: isActive))  // Bool
```

### 2.2 วิธีที่ Swift อนุมาน Type

```swift
// ตัวเลขจำนวนเต็ม → Int
let apples = 5        // Int
let bigNum = 1_000_000  // Int

// ตัวเลขทศนิยม → Double (ไม่ใช่ Float!)
let weight = 75.5     // Double
let height = 1.75     // Double

// ข้อความ → String
let greeting = "สวัสดี"  // String

// ตรรกะ → Bool
let done = false        // Bool

// สังเกต: Swift เลือก Double แทน Float เป็น Default
let x = 3.14  // Double (ไม่ใช่ Float)
print(type(of: x))  // Double
```

### 2.3 ข้อจำกัดของ Type Inference

```swift
// ❌ ไม่ได้: เปลี่ยน Type ไม่ได้!
var number = 42      // Swift อนุมานว่าเป็น Int
// number = 3.14     // Error: Cannot assign value of type 'Double' to type 'Int'

var text = "Hello"   // String
// text = 100        // Error: Cannot assign value of type 'Int' to type 'String'

// ✅ ถูก: ใช้ Type ที่สอดคล้องกัน
number = 100         // Int ← Int ✅
text = "World"       // String ← String ✅
```

---

## 3. Explicit Type Annotations — ระบุชนิดข้อมูลอย่างชัดเจน

บางครั้งเราต้องการระบุ Type ชัดๆ โดยใช้ `: TypeName` หลังชื่อตัวแปร

### 3.1 รูปแบบพื้นฐาน

```swift
// รูปแบบ: var/let ชื่อตัวแปร: ชนิดข้อมูล = ค่า
var age: Int = 25
let name: String = "สมชาย"
var temperature: Double = 36.5
var isStudent: Bool = true

// ประกาศโดยไม่กำหนดค่าทันที (ต้องระบุ Type)
var score: Int    // ยังไม่มีค่า
// print(score)  // Error: variable 'score' used before being initialized

score = 95        // กำหนดค่าก่อนใช้
print(score)      // 95
```

### 3.2 เมื่อควรระบุ Type ชัดๆ

```swift
// 1. ต้องการ Float แทน Double (default)
let temperature: Float = 36.5   // Float
let exact: Double = 36.5        // Double (ค่าเหมือนกัน แต่ precision ต่าง)

// 2. ต้องการ Int ขนาดเฉพาะ
let small: Int8 = 100           // ใช้ Int8 แทน Int
let big: Int64 = 1_000_000_000_000  // ใช้ Int64

// 3. ประกาศตัวแปรก่อนกำหนดค่า
var result: String
if true {
    result = "สำเร็จ"
} else {
    result = "ล้มเหลว"
}
print(result)

// 4. ค่าที่ Ambiguous
let x: Double = 10   // ถ้าไม่ระบุ จะเป็น Int
print(type(of: x))   // Double

// 5. เพื่อความชัดเจนในโค้ด
var userAge: Int = 30  // ชัดเจนกว่าแม้ Swift จะ infer ได้
```

### 3.3 Type Annotation กับ Collection Types

```swift
// Array
var numbers: [Int] = [1, 2, 3, 4, 5]
var names: [String] = ["สมชาย", "สมหญิง", "สมศรี"]
var emptyArray: [Double] = []  // ต้องระบุ Type สำหรับ array ว่าง

// Dictionary
var scores: [String: Int] = ["คณิต": 90, "วิทย์": 85]
var emptyDict: [String: Any] = [:]

// Optional
var optionalName: String? = nil  // อาจเป็น String หรือ nil
var optionalAge: Int? = 25
```

---

## 4. Integer Types — ชนิดข้อมูลจำนวนเต็ม

Swift มีชนิดข้อมูลจำนวนเต็มหลายแบบ สำหรับขนาดและ Sign ที่แตกต่างกัน

### 4.1 Signed Integers (มีเครื่องหมาย)

```swift
// Int — ขนาดขึ้นกับ Platform (32 หรือ 64-bit)
let defaultInt: Int = 42

// Int8 — 8 บิต (-128 ถึง 127)
let smallInt: Int8 = 100
print("Int8 min: \(Int8.min)")    // -128
print("Int8 max: \(Int8.max)")    // 127

// Int16 — 16 บิต (-32,768 ถึง 32,767)
let mediumInt: Int16 = 10000
print("Int16 min: \(Int16.min)")  // -32768
print("Int16 max: \(Int16.max)")  // 32767

// Int32 — 32 บิต (-2,147,483,648 ถึง 2,147,483,647)
let bigInt: Int32 = 2_000_000
print("Int32 min: \(Int32.min)")  // -2147483648
print("Int32 max: \(Int32.max)")  // 2147483647

// Int64 — 64 บิต (ใหญ่มาก)
let hugeInt: Int64 = 9_223_372_036_854_775_807
print("Int64 max: \(Int64.max)")  // 9223372036854775807
```

### 4.2 Unsigned Integers (ไม่มีเครื่องหมาย — บวกเท่านั้น)

```swift
// UInt — ขนาดขึ้นกับ Platform
let unsignedDefault: UInt = 42

// UInt8 — 8 บิต (0 ถึง 255)
let byte: UInt8 = 255
print("UInt8 min: \(UInt8.min)")  // 0
print("UInt8 max: \(UInt8.max)")  // 255

// UInt16 — 16 บิต (0 ถึง 65,535)
let port: UInt16 = 8080
print("UInt16 max: \(UInt16.max)")  // 65535

// UInt32 — 32 บิต
let rgbColor: UInt32 = 0xFF5733    // สีส้ม
print("UInt32 max: \(UInt32.max)")  // 4294967295

// UInt64 — 64 บิต
print("UInt64 max: \(UInt64.max)")  // 18446744073709551615
```

### 4.3 ตาราง Integer Types

| Type | Size | Minimum | Maximum |
|------|------|---------|---------|
| `Int8` | 8 bits | -128 | 127 |
| `Int16` | 16 bits | -32,768 | 32,767 |
| `Int32` | 32 bits | -2,147,483,648 | 2,147,483,647 |
| `Int64` | 64 bits | -9.2 × 10¹⁸ | 9.2 × 10¹⁸ |
| `Int` | Platform | same as Int32/64 | same as Int32/64 |
| `UInt8` | 8 bits | 0 | 255 |
| `UInt16` | 16 bits | 0 | 65,535 |
| `UInt32` | 32 bits | 0 | 4,294,967,295 |
| `UInt64` | 64 bits | 0 | 1.8 × 10¹⁹ |
| `UInt` | Platform | 0 | same as UInt32/64 |

### 4.4 Int บน Platform ต่างๆ

```swift
// ตรวจสอบขนาดของ Int ตาม Platform
print("Int size: \(MemoryLayout<Int>.size) bytes")
// macOS/iOS 64-bit: 8 bytes (64 bits)
// 32-bit: 4 bytes (32 bits)

// Int เท่ากับ Int64 บน Platform ส่วนใหญ่
print("Int.max = \(Int.max)")    // 9223372036854775807 (64-bit)
print("Int.min = \(Int.min)")    // -9223372036854775808 (64-bit)

// แนะนำให้ใช้ Int เสมอ ยกเว้นมีเหตุผลพิเศษ
let score: Int = 100     // ✅ ดี
// let score: Int32 = 100  // ไม่จำเป็น ใช้ Int ได้
```

### 4.5 Operations บน Integers

```swift
let a = 17
let b = 5

print("บวก: \(a + b)")        // 22
print("ลบ: \(a - b)")         // 12
print("คูณ: \(a * b)")        // 85
print("หาร: \(a / b)")        // 3  ← Integer Division!
print("เศษ: \(a % b)")        // 2
print("ลบ Unary: \(-a)")      // -17

// Integer Division ตัดทศนิยมทิ้ง!
print("7 / 2 = \(7 / 2)")     // 3 (ไม่ใช่ 3.5!)
print("7 % 2 = \(7 % 2)")     // 1

// Bitwise Operations
let x: UInt8 = 0b1100_1010    // 202
let y: UInt8 = 0b0011_0101    // 53

print("AND: \(x & y)")         // Bitwise AND
print("OR:  \(x | y)")         // Bitwise OR
print("XOR: \(x ^ y)")         // Bitwise XOR
print("NOT: \(~x)")            // Bitwise NOT
print("Shift Left: \(x << 1)") // Left Shift
print("Shift Right: \(x >> 1)")// Right Shift
```

---

## 5. Floating Point Types — ชนิดข้อมูลทศนิยม

### 5.1 Float — 32-bit Floating Point

```swift
// Float — 32 บิต ทศนิยม ~7 หลักนัยสำคัญ
let temperature: Float = 36.5
let pi_float: Float = 3.14159265358979  // จะถูกตัดทอน!

print("Float pi: \(pi_float)")           // 3.1415927 (ตัดทอน)
print("Float size: \(MemoryLayout<Float>.size) bytes")  // 4 bytes
```

### 5.2 Double — 64-bit Floating Point

```swift
// Double — 64 บิต ทศนิยม ~15-17 หลักนัยสำคัญ
let piDouble: Double = 3.14159265358979323846
print("Double pi: \(piDouble)")  // 3.141592653589793

// Double เป็น Default สำหรับทศนิยมใน Swift
let x = 3.14     // Double (ไม่ใช่ Float)
print(type(of: x))  // Double

print("Double size: \(MemoryLayout<Double>.size) bytes")  // 8 bytes
```

### 5.3 Float80 — 80-bit Extended Precision (macOS/Linux เท่านั้น)

```swift
// Float80 — มีความแม่นยำสูงมาก (ไม่รองรับบน iOS/watchOS)
#if os(macOS) || os(Linux)
let highPrecision: Float80 = 3.14159265358979323846264338327950288
print("Float80 pi: \(highPrecision)")
print("Float80 size: \(MemoryLayout<Float80>.size) bytes")  // 16 bytes
#endif
```

### 5.4 เปรียบเทียบ Float Types

| Type | Size | Precision | Range |
|------|------|-----------|-------|
| `Float` | 4 bytes | ~7 digits | ±3.4 × 10³⁸ |
| `Double` | 8 bytes | ~15-17 digits | ±1.8 × 10³⁰⁸ |
| `Float80` | 16 bytes | ~18-19 digits | ±1.1 × 10⁴⁹³² |

### 5.5 Operations บน Floating Point

```swift
import Foundation

let a = 10.0
let b = 3.0

print("บวก: \(a + b)")                          // 13.0
print("ลบ: \(a - b)")                           // 7.0
print("คูณ: \(a * b)")                          // 30.0
print("หาร: \(a / b)")                          // 3.3333333333333335
print("sqrt: \(sqrt(a))")                        // 3.1622776601683795
print("pow: \(pow(a, b))")                       // 1000.0
print("floor: \(floor(a / b))")                  // 3.0
print("ceil: \(ceil(a / b))")                    // 4.0
print("round: \(round(a / b))")                  // 3.0
print("abs: \(abs(-5.5))")                       // 5.5

// Special Values
print("Infinity: \(Double.infinity)")
print("-Infinity: \(-Double.infinity)")
print("NaN: \(Double.nan)")
print("5.0/0.0: \(5.0/0.0)")                    // inf
print("isNaN: \(Double.nan.isNaN)")              // true
print("isInfinite: \(Double.infinity.isInfinite)")  // true

// Floating Point Precision ปัญหาที่พบบ่อย!
let f1 = 0.1
let f2 = 0.2
let sum = f1 + f2
print("0.1 + 0.2 = \(sum)")          // 0.30000000000000004 !!
print("0.1 + 0.2 == 0.3: \(sum == 0.3)")  // false!!

// วิธีเปรียบเทียบ Float ที่ถูกต้อง
let epsilon = 1e-10
print("เท่ากันโดยประมาณ: \(abs(sum - 0.3) < epsilon)")  // true
```

---

## 6. Boolean Type — ชนิดข้อมูลตรรกะ

### 6.1 Bool Basics

```swift
// Bool มีค่าได้แค่ true หรือ false
var isSwiftFun: Bool = true
var isBoring: Bool = false
let alwaysTrue = true    // Type inference → Bool

print(isSwiftFun)   // true
print(isBoring)     // false
print(!isSwiftFun)  // false (Logical NOT)

// Toggle — สลับค่า
isSwiftFun.toggle()
print(isSwiftFun)   // false

isSwiftFun.toggle()
print(isSwiftFun)   // true
```

### 6.2 Logical Operators

```swift
let a = true
let b = false

// AND (&&) — จริงทั้งคู่ถึงจะเป็น true
print("a && b: \(a && b)")  // false
print("a && a: \(a && a)")  // true

// OR (||) — จริงอย่างน้อยหนึ่งตัวถึงจะเป็น true
print("a || b: \(a || b)")  // true
print("b || b: \(b || b)")  // false

// NOT (!) — กลับค่า
print("!a: \(!a)")  // false
print("!b: \(!b)")  // true

// Compound
let canVote = true
let hasID = false
let canEnter = canVote && hasID
print("เข้าได้: \(canEnter)")  // false

// Short-circuit Evaluation
var counter = 0

func increment() -> Bool {
    counter += 1
    return true
}

// ถ้าฝั่งซ้ายเป็น false ฝั่งขวาจะไม่ถูก evaluate!
let result = false && increment()
print("counter: \(counter)")  // 0 — increment ไม่ถูกเรียก!

let result2 = true || increment()
print("counter: \(counter)")  // 0 — increment ไม่ถูกเรียก!
```

### 6.3 Comparison Operators ที่คืนค่า Bool

```swift
let x = 10
let y = 20

print(x == y)   // false — เท่ากัน?
print(x != y)   // true  — ไม่เท่ากัน?
print(x < y)    // true  — น้อยกว่า?
print(x > y)    // false — มากกว่า?
print(x <= y)   // true  — น้อยกว่าหรือเท่ากัน?
print(x >= y)   // false — มากกว่าหรือเท่ากัน?

// Ternary Operator
let max = x > y ? x : y
print("Max: \(max)")  // 20
```

---

## 7. Character Type — ชนิดข้อมูลอักขระ

### 7.1 Character Basics

```swift
// Character คืออักขระเดียว ใช้เครื่องหมายคำพูดเดี่ยว... ไม่ใช่!
// Swift ใช้ Double Quotes สำหรับทั้ง String และ Character
// ต้องระบุ Type ชัดๆ

let letterA: Character = "A"
let emoji: Character = "🐦"         // Swift รองรับ Emoji!
let thaiChar: Character = "ก"       // ภาษาไทย
let chineseChar: Character = "中"    // ภาษาจีน
let space: Character = " "

print(letterA)    // A
print(emoji)      // 🐦
print(thaiChar)   // ก
```

### 7.2 Character Properties

```swift
let c1: Character = "A"
let c2: Character = "a"
let c3: Character = "5"
let c4: Character = " "
let c5: Character = "!"

// ตรวจสอบ Properties
print(c1.isLetter)          // true — เป็นตัวอักษร?
print(c1.isUppercase)       // true — ตัวพิมพ์ใหญ่?
print(c2.isLowercase)       // true — ตัวพิมพ์เล็ก?
print(c3.isNumber)          // true — เป็นตัวเลข?
print(c3.isWholeNumber)     // true — เป็นเลขจำนวนเต็ม?
print(c3.wholeNumberValue!) // 5   — ค่าตัวเลข
print(c4.isWhitespace)      // true — เป็น whitespace?
print(c5.isPunctuation)     // true — เป็นเครื่องหมายวรรคตอน?

// แปลงตัวพิมพ์
let upper = c2.uppercased()  // "A" (คืนเป็น String)
let lower = c1.lowercased()  // "a"
print(upper, lower)
```

### 7.3 Extended Grapheme Clusters

```swift
// Swift's Character รองรับ Unicode Extended Grapheme Clusters
// ซึ่งหมายถึง "อักขระที่มองเห็นได้" ซึ่งอาจประกอบจากหลาย Unicode scalar

let e1: Character = "\u{65}"           // e
let e2: Character = "\u{65}\u{301}"    // é (e + combining accent)

// ทั้งคู่เป็น Character เดียว แม้มี Unicode scalar ต่างกัน
print(e1)   // e
print(e2)   // é

// ภาษาไทยมี Combining Characters ด้วย!
let thai: Character = "\u{0E01}\u{0E49}"  // ก้ (ก + ไม้โท)
print(thai)  // ก้
```

---

## 8. String Type — ชนิดข้อมูลข้อความ

String ใน Swift เป็น Collection ของ Character และเป็น Value Type (copy-on-write)

### 8.1 String Basics

```swift
// ประกาศ String
let empty = ""               // String ว่าง
let hello = "Hello, Swift!"  // String ปกติ
var greeting = "สวัสดี"      // String ภาษาไทย

// String ว่าง
print(empty.isEmpty)        // true
print(hello.isEmpty)        // false
print(hello.count)          // 13 — จำนวนอักขระ

// String Concatenation (+)
let firstName = "สมชาย"
let lastName = "ใจดี"
let fullName = firstName + " " + lastName
print(fullName)  // สมชาย ใจดี

// Append
var message = "Hello"
message += ", World"
message.append("!")
print(message)  // Hello, World!
```

### 8.2 String Interpolation

```swift
let name = "สมหญิง"
let age = 25
let score = 98.5

// แทรกค่าตัวแปรใน String ด้วย \()
let intro = "ฉันชื่อ \(name) อายุ \(age) ปี"
print(intro)  // ฉันชื่อ สมหญิง อายุ 25 ปี

// แทรก Expression ได้
let bmi = 70.0 / (1.75 * 1.75)
print("BMI ของฉัน = \(String(format: "%.1f", bmi))")

// String Interpolation ซับซ้อน
let info = """
    ชื่อ: \(name)
    อายุ: \(age) ปี
    คะแนน: \(score)
    เกรด: \(score >= 80 ? "A" : "B")
    """
print(info)
```

### 8.3 Multi-line Strings

```swift
// Multi-line String ใช้ """ """
let poem = """
    กลางดึกดื่นฉันนั่งเขียนโค้ด
    Swift คือภาษาแห่งความฝัน
    Type Safety พิทักษ์ทุกหน
    Compiler ช่วยเตือนก่อนพัง
    """

print(poem)

// Indentation — ช่องว่างหน้า """ ปิดกำหนด indentation
let html = """
    <html>
        <body>
            <h1>Hello, Swift!</h1>
        </body>
    </html>
    """
print(html)
```

### 8.4 String Methods ที่สำคัญ

```swift
var str = "  Hello, Swift World!  "

// Trimming
print(str.trimmingCharacters(in: .whitespaces))  // "Hello, Swift World!"

// Upper/Lower Case
print(str.uppercased())  // "  HELLO, SWIFT WORLD!  "
print(str.lowercased())  // "  hello, swift world!  "

// Contains
print(str.contains("Swift"))   // true
print(str.contains("Python"))  // false

// hasPrefix / hasSuffix
let url = "https://swift.org/learn"
print(url.hasPrefix("https"))  // true
print(url.hasSuffix("learn"))  // true

// Split
let csv = "สมชาย,25,กรุงเทพ"
let parts = csv.split(separator: ",")
print(parts)            // ["สมชาย", "25", "กรุงเทพ"]
print(parts[0])         // สมชาย

// Replace
var text = "Hello World"
text = text.replacingOccurrences(of: "World", with: "Swift")
print(text)  // Hello Swift

// Substring
let greeting2 = "Hello, World!"
let start = greeting2.index(greeting2.startIndex, offsetBy: 7)
let end = greeting2.index(greeting2.endIndex, offsetBy: -1)
let middle = greeting2[start..<end]
print(middle)  // World
```

### 8.5 String Comparison

```swift
let s1 = "Apple"
let s2 = "apple"
let s3 = "Apple"

// Case Sensitive (ค่าเริ่มต้น)
print(s1 == s2)  // false — A ≠ a
print(s1 == s3)  // true

// Case Insensitive
print(s1.lowercased() == s2.lowercased())  // true

// Lexicographic comparison
print("Apple" < "Banana")  // true
print("Swift" > "Python")  // true

// Localized Comparison (คำนึงถึงภาษา)
let th1 = "ก"
let th2 = "ข"
print(th1 < th2)  // true (ตาม Unicode)
```

---

## 9. Type Aliases — ชื่อเรียกแทนชนิดข้อมูล

`typealias` ช่วยให้เราสร้างชื่อเรียกแทนชนิดข้อมูลที่มีอยู่แล้ว เพื่อความอ่านง่ายขึ้น

### 9.1 พื้นฐาน typealias

```swift
// สร้างชื่อเรียกแทน
typealias AudioSample = UInt16
typealias Meters = Double
typealias Kilograms = Double
typealias Seconds = Double
typealias StudentID = String
typealias Score = Int

// ใช้งาน
var maxAmplitude: AudioSample = AudioSample.max
print("Max amplitude: \(maxAmplitude)")  // 65535

let distance: Meters = 100.0
let time: Seconds = 9.58  // Usain Bolt world record
let speed = distance / time
print("ความเร็ว: \(String(format: "%.2f", speed)) m/s")

let weight: Kilograms = 70.5
let studentId: StudentID = "ST2024001"
let mathScore: Score = 95
```

### 9.2 typealias กับ Complex Types

```swift
// Closure Type ที่ยาว → สั้นลงด้วย typealias
typealias CompletionHandler = (Bool, Error?) -> Void
typealias JSONDictionary = [String: Any]
typealias Coordinate = (lat: Double, lon: Double)

// ใช้งาน
func fetchData(completion: CompletionHandler) {
    // ... fetch data
    completion(true, nil)
}

var userInfo: JSONDictionary = [
    "name": "สมชาย",
    "age": 25,
    "active": true
]

let bangkokCoord: Coordinate = (lat: 13.7563, lon: 100.5018)
print("กรุงเทพ: \(bangkokCoord.lat)°N, \(bangkokCoord.lon)°E")
```

---

## 10. Numeric Literals — การเขียนตัวเลข

Swift รองรับการเขียนตัวเลขในหลายรูปแบบ

### 10.1 Integer Literals

```swift
// Decimal (ฐาน 10) — ค่าเริ่มต้น
let decimal = 17
let million = 1_000_000     // ใช้ _ เพื่อความอ่านง่าย

// Binary (ฐาน 2) — ขึ้นต้นด้วย 0b
let binary = 0b10001        // = 17 ในทศนิยม
let binaryByte = 0b1111_1111  // = 255

// Octal (ฐาน 8) — ขึ้นต้นด้วย 0o
let octal = 0o21            // = 17 ในทศนิยม

// Hexadecimal (ฐาน 16) — ขึ้นต้นด้วย 0x
let hex = 0x11              // = 17 ในทศนิยม
let hexColor = 0xFF5733     // = สีส้ม (RGB)
let hexBig = 0xDEAD_BEEF    // = 3735928559

// ตรวจสอบ
print("Decimal: \(decimal)")
print("Binary 0b10001: \(binary)")
print("Octal 0o21: \(octal)")
print("Hex 0x11: \(hex)")
print("ทั้งหมดเท่ากับ 17: \(decimal == binary && binary == octal && octal == hex)")
```

### 10.2 Floating Point Literals

```swift
// Decimal Float
let f1 = 1.25
let f2 = 1.25e2   // = 125.0  (1.25 × 10²)
let f3 = 1.25e-2  // = 0.0125 (1.25 × 10⁻²)

// Hexadecimal Float
let f4 = 0xFp0    // = 15.0   (F หรือ 15 × 2⁰)
let f5 = 0xFp2    // = 60.0   (15 × 2²)
let f6 = 0xFp-2   // = 3.75   (15 × 2⁻²)

print("1.25e2 = \(f2)")   // 125.0
print("1.25e-2 = \(f3)")  // 0.0125
print("0xFp2 = \(f5)")    // 60.0

// Readable Format ด้วย _
let bigFloat = 1_000_000.123_456
print("bigFloat = \(bigFloat)")
```

### 10.3 ตัวอย่างใช้งานจริง

```swift
// Color ในรูป Hex
let red: UInt32 = 0xFF0000
let green: UInt32 = 0x00FF00
let blue: UInt32 = 0x0000FF
let orange: UInt32 = 0xFF5733

// Bit Flags
let readPermission: UInt8   = 0b00000001  // 1
let writePermission: UInt8  = 0b00000010  // 2
let executePermission: UInt8 = 0b00000100  // 4

var userPermissions: UInt8 = readPermission | writePermission
print("Permissions: \(String(userPermissions, radix: 2))")  // 11

let canRead = (userPermissions & readPermission) != 0
let canWrite = (userPermissions & writePermission) != 0
let canExecute = (userPermissions & executePermission) != 0

print("อ่านได้: \(canRead)")      // true
print("เขียนได้: \(canWrite)")    // true
print("รันได้: \(canExecute)")    // false

// Scientific Notation สำหรับค่าทางวิทยาศาสตร์
let speedOfLight = 2.998e8   // 299,800,000 m/s
let electronMass = 9.109e-31  // 0.0000000000000000000000000000009109 kg
let avogadro = 6.022e23       // 602,200,000,000,000,000,000,000

print("ความเร็วแสง: \(speedOfLight) m/s")
print("มวลอิเล็กตรอน: \(electronMass) kg")
print("Avogadro: \(avogadro)")
```

---

## 11. Numeric Type Conversion — การแปลงชนิดตัวเลข

Swift **ไม่ทำ Implicit Conversion** ต้องแปลงชนิดข้อมูลตัวเลขด้วยตัวเอง

### 11.1 ทำไม Swift ไม่แปลงอัตโนมัติ?

```swift
let a: Int = 10
let b: Double = 3.14

// ❌ Error: ไม่สามารถบวก Int กับ Double ได้โดยตรง!
// let sum = a + b  // error: binary operator '+' cannot be applied to operands of type 'Int' and 'Double'

// ✅ ต้องแปลงก่อน
let sum = Double(a) + b     // แปลง a เป็น Double
let sum2 = a + Int(b)       // แปลง b เป็น Int (ตัดทศนิยมทิ้ง)

print("Double + Double: \(sum)")   // 13.14
print("Int + Int: \(sum2)")        // 13 (Int division)
```

### 11.2 วิธีแปลงชนิดตัวเลข

```swift
// Int → Double
let intVal: Int = 42
let doubleVal = Double(intVal)   // 42.0
print(type(of: doubleVal))       // Double

// Double → Int (ตัดทศนิยม ไม่ปัดเศษ)
let pi = 3.99999
let intPi = Int(pi)              // 3 (ตัดทศนิยมทิ้ง!)
print(intPi)                     // 3

// Double → Int (ปัดเศษ)
import Foundation
let rounded = Int(pi.rounded())  // 4
print(rounded)                   // 4

// Int → String
let num = 42
let str = String(num)            // "42"
print(str)
print(type(of: str))             // String

// String → Int (Optional เพราะอาจล้มเหลว)
let numStr = "123"
if let parsed = Int(numStr) {
    print("แปลงสำเร็จ: \(parsed)")  // 123
}

let invalidStr = "abc"
if let parsed = Int(invalidStr) {
    print("สำเร็จ: \(parsed)")
} else {
    print("ไม่สามารถแปลงได้: \(invalidStr)")  // แสดงข้อความนี้
}

// Float → Double
let floatVal: Float = 3.14
let doubleVal2 = Double(floatVal)
print(doubleVal2)  // 3.140000104904175 (ความผิดพลาดจาก Float precision)

// Int ขนาดต่างๆ
let big: Int64 = 1_000_000_000
let small = Int32(big)     // อาจ Overflow ถ้าค่ามากเกินไป!
print(small)               // 1000000000

let tooBig: Int64 = 3_000_000_000  // เกิน Int32.max
// let overflow = Int32(tooBig)  // Runtime Error: Integer overflow!
```

### 11.3 Safe Conversion ด้วย Overflow Check

```swift
let value: Int64 = 3_000_000_000

// ตรวจสอบก่อนแปลง
if value >= Int64(Int32.min) && value <= Int64(Int32.max) {
    let converted = Int32(value)
    print("แปลงได้: \(converted)")
} else {
    print("ค่าเกินขอบเขต Int32!")  // แสดงอันนี้
}

// ใช้ exactly initializer
if let safe = Int32(exactly: value) {
    print("Safe: \(safe)")
} else {
    print("Overflow!")  // แสดงอันนี้
}
```

---

## 12. Overflow Operators — ตัวดำเนินการ Overflow

ปกติ Swift จะ **crash** ถ้า Integer overflow แต่มี Overflow Operators พิเศษสำหรับกรณีที่ต้องการให้ wrap around

### 12.1 Overflow คืออะไร?

```swift
// ปกติ Swift จะ Crash ถ้า Overflow!
var maxInt = Int8.max   // 127
// maxInt += 1  // ❌ Runtime crash: arithmetic overflow

// ✅ ใช้ Overflow Operators: &+, &-, &*
var overflow1 = Int8.max    // 127
overflow1 = overflow1 &+ 1  // Wrap around → -128!
print("127 &+ 1 = \(overflow1)")  // -128

var overflow2 = Int8.min    // -128
overflow2 = overflow2 &- 1  // Wrap around → 127!
print("-128 &- 1 = \(overflow2)") // 127

var overflow3: UInt8 = 0
overflow3 = overflow3 &- 1  // Wrap around → 255!
print("0 &- 1 = \(overflow3)")    // 255
```

### 12.2 Overflow Operators

| Operator | ชื่อ |
|----------|------|
| `&+` | Overflow Addition |
| `&-` | Overflow Subtraction |
| `&*` | Overflow Multiplication |

```swift
// ตัวอย่างใช้งานจริง: checksum, hash, wrapping counter
var counter: UInt8 = 255
counter = counter &+ 1   // 0 — วนรอบ!
print("Counter: \(counter)")  // 0

// Overflow Multiplication
let a: Int8 = 100
let b: Int8 = 4
let product = a &* b  // 100 * 4 = 400, แต่ Int8 max คือ 127
print("100 &* 4 = \(product)")  // 112 (ตัดส่วนที่เกิน)
```

---

## 13. Type Safety ใน Swift

### 13.1 Type Safety คืออะไร?

Swift เป็น **Type-Safe Language** หมายความว่า:
- ทุกตัวแปรมี Type ที่ชัดเจน
- ไม่สามารถใส่ค่า Type ผิดได้โดยอัตโนมัติ
- Compiler ตรวจสอบ Type ก่อน Compile

```swift
// ❌ Type Safety ป้องกันข้อผิดพลาดแบบนี้
var name = "Swift"
// name = 42     // Error: Cannot assign value of type 'Int' to type 'String'
// name = true   // Error: Cannot assign value of type 'Bool' to type 'String'

// ต้องแปลง Type ชัดๆ
var message = name + " is \(5 + 5) years old"
print(message)  // Swift is 10 years old
```

### 13.2 เปรียบเทียบกับภาษาอื่น

```javascript
// JavaScript (ไม่ Type Safe)
var x = "5"
var y = 10
console.log(x + y)   // "510" (String concat, ไม่ใช่บวกตัวเลข!)
console.log(x * y)   // 50   (แปลงอัตโนมัติ — สับสน!)
```

```python
# Python (Type Error เกิดตอน Runtime)
x = "5"
y = 10
# print(x + y)  # TypeError: can only concatenate str (not "int") to str
```

```swift
// Swift (Type Safe — Error ตอน Compile Time)
let x = "5"
let y = 10
// print(x + y)  // Error: binary operator '+' cannot be applied...
print(Int(x)! + y)  // 15 — ต้องแปลงชัดๆ
```

### 13.3 Type Safety กับ Optionals

```swift
// Swift ใช้ Optional เพื่อ Type-Safe กับค่าที่อาจเป็น nil
var optionalName: String? = "สมชาย"

// ❌ ไม่สามารถใช้ Optional ตรงๆ โดยไม่ Unwrap
// let upper = optionalName.uppercased()  // Error!

// ✅ ต้อง Unwrap ก่อน
if let name = optionalName {
    print(name.uppercased())  // สมชาย → สมชาย
}

// Optional Chaining
print(optionalName?.uppercased() ?? "ไม่มีชื่อ")

optionalName = nil
print(optionalName?.uppercased() ?? "ไม่มีชื่อ")  // ไม่มีชื่อ
```

---

## 14. Naming Conventions — หลักการตั้งชื่อ

### 14.1 camelCase สำหรับตัวแปรและฟังก์ชัน

```swift
// ✅ camelCase — เริ่มด้วยตัวพิมพ์เล็ก
var firstName = "สมชาย"
var lastName = "ใจดี"
var totalScore = 0
var isAuthenticated = false
var maxRetryCount = 3

func calculateBMI(weight: Double, height: Double) -> Double {
    return weight / (height * height)
}
```

### 14.2 PascalCase สำหรับ Types

```swift
// ✅ PascalCase — เริ่มด้วยตัวพิมพ์ใหญ่
struct UserProfile {}
class DatabaseManager {}
enum NetworkError {}
protocol Printable {}
typealias UserID = String
```

### 14.3 ตัวอย่าง Naming ที่ดีและไม่ดี

```swift
// ❌ ไม่ดี
var n = "สมชาย"          // ชื่อสั้นเกิน ไม่รู้ว่า n คืออะไร
var x1 = true            // ไม่สื่อความหมาย
let MAXVALUE = 100       // ไม่ใช่ Swift Style (ใช้ SCREAMING_SNAKE_CASE ใน C)
var user_name = "test"   // snake_case ไม่ใช่ Swift convention

// ✅ ดี
var playerName = "สมชาย"
var isLoggedIn = true
let maximumValue = 100   // หรือ maxValue
var userName = "test"    // camelCase

// ✅ ชื่อที่อธิบายตัวเองได้
var numberOfRetries = 3         // ชัดเจน
var isEmailVerified = false     // ชัดเจน
let databaseConnectionTimeout = 30.0  // ชัดเจน

// Bool ควรเริ่มด้วย is/has/can/should/will/did
var isActive = true
var hasPermission = false
var canEdit = true
var shouldAutoSave = false
var willReload = false
var didFinishLoading = false
```

### 14.4 Unicode Characters ในชื่อตัวแปร

```swift
// Swift รองรับ Unicode ในชื่อตัวแปร (แต่ไม่แนะนำ!)
let π = 3.14159   // Pi symbol
let 名前 = "สมชาย"  // ภาษาญี่ปุ่น
let 🐦 = "Swift"  // Emoji (ไม่แนะนำ!)

// ใช้ในทาง Mathematical ได้บ้าง
let α = 0.001    // Learning rate ใน ML
let Δx = 0.0001  // Delta x

print(π)   // 3.14159
print(名前)  // สมชาย
```

---

## 15. Multiple Variable Declarations

### 15.1 ประกาศหลายตัวแปรพร้อมกัน

```swift
// ประกาศหลายตัวในบรรทัดเดียว (ไม่แนะนำ — อ่านยาก)
var x = 1, y = 2, z = 3

// ดีกว่า — แยกบรรทัด
var a = 1
var b = 2
var c = 3

// ประกาศหลาย let
let width = 1920, height = 1080
print("\(width)×\(height)")  // 1920×1080

// ประกาศ Type เดียวกัน
var red: Double, green: Double, blue: Double
red = 1.0
green = 0.5
blue = 0.0
```

### 15.2 Tuple Assignment

```swift
// Tuple — เก็บหลายค่าในตัวแปรเดียว
let point = (x: 10, y: 20)
print("x: \(point.x), y: \(point.y)")

// Destructuring
let (px, py) = point
print("px=\(px), py=\(py)")

// Swap ค่าด้วย Tuple (ไม่ต้องใช้ temp variable!)
var first = "Hello"
var second = "World"
(first, second) = (second, first)
print("\(first) \(second)")  // World Hello

// Ignore บางค่าด้วย _
let (name, _, score) = ("สมชาย", 25, 95)
print("ชื่อ: \(name), คะแนน: \(score)")
```

---

## 16. Constant Expressions

### 16.1 let กับ Value Types

```swift
// Value Types (Int, String, Bool, struct, enum)
// let ทำให้ immutable ทั้ง object
let number = 42
// number = 43  // Error!

let greeting = "Hello"
// greeting = "Hi"  // Error!

// Struct กับ let
struct Point {
    var x: Int
    var y: Int
}

let p = Point(x: 10, y: 20)
// p.x = 30  // Error! ไม่สามารถเปลี่ยน property ของ let struct

var p2 = Point(x: 10, y: 20)
p2.x = 30  // ✅ var struct ให้เปลี่ยน property ได้
```

### 16.2 let กับ Reference Types (Class)

```swift
// Reference Types (class)
// let ทำให้ reference ไม่เปลี่ยน แต่ object ยังเปลี่ยนได้!
class Person {
    var name: String
    var age: Int
    
    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }
}

let alice = Person(name: "Alice", age: 25)
alice.name = "Alicia"   // ✅ เปลี่ยน property ได้!
alice.age = 26          // ✅

// alice = Person(name: "Bob", age: 30)  // ❌ Error! ไม่สามารถเปลี่ยน reference

print("\(alice.name), \(alice.age)")  // Alicia, 26
```

---

## 17. Lazy Variables

### 17.1 lazy var คืออะไร?

**Lazy Variable** คือตัวแปรที่จะถูก Initialize ก็ต่อเมื่อถูกเรียกใช้ครั้งแรก (Lazy Initialization) ช่วยประหยัดทรัพยากร

```swift
// ตัวอย่าง: Property ที่ cost สูงในการสร้าง
class DataProcessor {
    // ✅ lazy — จะสร้างก็ต่อเมื่อถูกเรียกใช้
    lazy var expensiveData: [Int] = {
        print("กำลังสร้าง expensiveData...")
        return (1...1_000_000).map { $0 * 2 }  // สร้างอาร์เรย์ใหญ่
    }()
    
    var name: String
    
    init(name: String) {
        self.name = name
        print("สร้าง DataProcessor: \(name)")
    }
}

let processor = DataProcessor(name: "MyProcessor")
print("สร้าง processor แล้ว")  // expensiveData ยังไม่ถูกสร้าง!

// เรียกใช้ครั้งแรก → สร้างข้อมูล
let count = processor.expensiveData.count
print("จำนวน: \(count)")  // กำลังสร้าง... แล้วแสดง 1000000
```

### 17.2 ใช้ lazy ที่ไหนบ้าง?

```swift
class ViewController {
    // UIView ที่ cost สูงในการสร้าง
    lazy var tableView: UITableView = {
        let tv = UITableView()
        tv.backgroundColor = .white
        // ... configure
        return tv
    }()
    
    // Database connection
    lazy var database: DatabaseConnection = {
        return DatabaseConnection(url: "sqlite://app.db")
    }()
    
    // Large computation
    lazy var fibonacci: [Int] = {
        var seq = [0, 1]
        for i in 2..<100 {
            seq.append(seq[i-1] + seq[i-2])
        }
        return seq
    }()
}
```

### 17.3 ข้อควรระวัง lazy

```swift
// lazy ใช้ได้กับ var เท่านั้น (ไม่ใช่ let)
// lazy let — ❌ Error!

// lazy ไม่ Thread-Safe โดยค่าเริ่มต้น
// ถ้าใช้ใน Multi-thread ต้องเพิ่ม synchronization เอง

// lazy ใช้ได้ใน stored properties เท่านั้น
// ไม่สามารถใช้กับ computed properties
```

---

## 18. Computed Properties — เกริ่นนำ

**Computed Properties** คือ Properties ที่คำนวณค่าทุกครั้งที่ถูกเข้าถึง (ไม่เก็บค่าจริง)

```swift
struct Rectangle {
    var width: Double
    var height: Double
    
    // Computed Property — คำนวณทุกครั้งที่ถูกเรียก
    var area: Double {
        return width * height
    }
    
    var perimeter: Double {
        return 2 * (width + height)
    }
    
    var isSquare: Bool {
        return width == height
    }
}

let rect = Rectangle(width: 10, height: 5)
print("พื้นที่: \(rect.area)")        // 50.0
print("เส้นรอบวง: \(rect.perimeter)") // 30.0
print("เป็นสี่เหลี่ยมจตุรัส: \(rect.isSquare)") // false

// เปลี่ยน width
var rect2 = Rectangle(width: 5, height: 5)
print("area: \(rect2.area)")      // 25.0
print("isSquare: \(rect2.isSquare)")  // true
```

---

## 19. แบบฝึกหัดพร้อมเฉลย

### แบบฝึกหัดที่ 1: Type Inference Detective

บอก Type ของตัวแปรต่อไปนี้:

```swift
let a = 42          // ?
var b = 3.14        // ?
let c = "Hello"     // ?
var d = true        // ?
let e = a + 10      // ?
var f = b * 2       // ?
let g = [1, 2, 3]  // ?
var h: Float = 1.5  // ?
```

**เฉลย:**

```swift
let a = 42          // Int
var b = 3.14        // Double
let c = "Hello"     // String
var d = true        // Bool
let e = a + 10      // Int (Int + Int = Int)
var f = b * 2       // Double (Double * Int literal = Double)
let g = [1, 2, 3]  // [Int]
var h: Float = 1.5  // Float (ระบุ Type ชัดๆ)

// ตรวจสอบ
print(type(of: a))  // Int
print(type(of: b))  // Double
print(type(of: c))  // String
print(type(of: d))  // Bool
print(type(of: e))  // Int
print(type(of: f))  // Double
print(type(of: g))  // Array<Int>
print(type(of: h))  // Float
```

---

### แบบฝึกหัดที่ 2: Unit Converter

เขียน Unit Converter ที่แปลงหน่วยต่างๆ:

```swift
// exercise_unit_converter.swift
import Foundation

// กำหนดค่าคงที่
let kmPerMile: Double = 1.60934
let kgPerPound: Double = 0.453592
let cmPerInch: Double = 2.54
let celsiusToFahrenheitFactor: Double = 9.0/5.0

// ฟังก์ชันแปลง
func milesToKm(_ miles: Double) -> Double { miles * kmPerMile }
func kmToMiles(_ km: Double) -> Double { km / kmPerMile }
func poundsToKg(_ pounds: Double) -> Double { pounds * kgPerPound }
func kgToPounds(_ kg: Double) -> Double { kg / kgPerPound }
func inchesToCm(_ inches: Double) -> Double { inches * cmPerInch }
func cmToInches(_ cm: Double) -> Double { cm / cmPerInch }
func celsiusToF(_ c: Double) -> Double { (c * celsiusToFahrenheitFactor) + 32 }
func fahrenheitToC(_ f: Double) -> Double { (f - 32) / celsiusToFahrenheitFactor }

// แสดงผล
print("╔══════════════════════════════════════╗")
print("║         Unit Converter               ║")
print("╠══════════════════════════════════════╣")

let miles = 26.2  // Marathon!
print(String(format: "║ %.1f miles = %.2f km", miles, milesToKm(miles)))

let weight: Double = 70  // kg
print(String(format: "║ %.1f kg = %.1f lbs", weight, kgToPounds(weight)))

let height: Double = 175  // cm
print(String(format: "║ %.0f cm = %.1f inches", height, cmToInches(height)))

let bodyTemp: Double = 37
print(String(format: "║ %.1f°C = %.1f°F", bodyTemp, celsiusToF(bodyTemp)))

print("╚══════════════════════════════════════╝")
```

---

### แบบฝึกหัดที่ 3: Student Grade System

สร้างระบบคิดเกรดนักเรียน:

```swift
// exercise_grade.swift

typealias StudentName = String
typealias Score = Double

// ข้อมูลนักเรียน
let studentName: StudentName = "สมศักดิ์ เรียนดี"
let mathScore: Score = 85.5
let scienceScore: Score = 92.0
let englishScore: Score = 78.5
let thaiScore: Score = 88.0
let socialScore: Score = 81.0

// คำนวณ
let totalScore = mathScore + scienceScore + englishScore + thaiScore + socialScore
let averageScore = totalScore / 5.0

// คิดเกรด
let grade: String
let gpa: Double

if averageScore >= 80 {
    grade = "A"
    gpa = 4.0
} else if averageScore >= 75 {
    grade = "B+"
    gpa = 3.5
} else if averageScore >= 70 {
    grade = "B"
    gpa = 3.0
} else if averageScore >= 65 {
    grade = "C+"
    gpa = 2.5
} else if averageScore >= 60 {
    grade = "C"
    gpa = 2.0
} else if averageScore >= 55 {
    grade = "D+"
    gpa = 1.5
} else if averageScore >= 50 {
    grade = "D"
    gpa = 1.0
} else {
    grade = "F"
    gpa = 0.0
}

// แสดงผล
print("╔══════════════════════════════════════╗")
print("║         ใบรายงานผลการเรียน          ║")
print("╠══════════════════════════════════════╣")
print("║ ชื่อ: \(studentName)")
print("╠══════════════════════════════════════╣")
print(String(format: "║ คณิตศาสตร์:  %5.1f", mathScore))
print(String(format: "║ วิทยาศาสตร์: %5.1f", scienceScore))
print(String(format: "║ ภาษาอังกฤษ: %5.1f", englishScore))
print(String(format: "║ ภาษาไทย:    %5.1f", thaiScore))
print(String(format: "║ สังคมศึกษา: %5.1f", socialScore))
print("╠══════════════════════════════════════╣")
print(String(format: "║ คะแนนรวม:   %5.1f / 500", totalScore))
print(String(format: "║ เฉลี่ย:     %5.1f", averageScore))
print("║ เกรด:       \(grade)")
print(String(format: "║ GPA:        %.1f", gpa))
print("╚══════════════════════════════════════╝")
```

---

### แบบฝึกหัดที่ 4: Integer Exploration

สำรวจ Integer Types:

```swift
// exercise_integers.swift

print("=== Integer Type Sizes ===\n")

let types: [(String, Int)] = [
    ("Int8",  MemoryLayout<Int8>.size),
    ("Int16", MemoryLayout<Int16>.size),
    ("Int32", MemoryLayout<Int32>.size),
    ("Int64", MemoryLayout<Int64>.size),
    ("Int",   MemoryLayout<Int>.size),
    ("UInt8",  MemoryLayout<UInt8>.size),
    ("UInt16", MemoryLayout<UInt16>.size),
    ("UInt32", MemoryLayout<UInt32>.size),
    ("UInt64", MemoryLayout<UInt64>.size),
]

print(String(format: "%-8s %8s %25s %25s", "Type", "Bytes", "Min", "Max"))
print(String(repeating: "-", count: 70))

// แสดง Int8 และ UInt8 เป็นตัวอย่าง
print(String(format: "%-8s %8d %25d %25d", "Int8",  1, Int8.min,  Int8.max))
print(String(format: "%-8s %8d %25d %25d", "Int16", 2, Int16.min, Int16.max))
print(String(format: "%-8s %8d %25d %25d", "Int32", 4, Int32.min, Int32.max))
print(String(format: "%-8s %8d %25lld %25lld", "Int64", 8, Int64.min, Int64.max))
print(String(format: "%-8s %8d %25d %25d", "UInt8",  1, UInt8.min,  UInt8.max))
print(String(format: "%-8s %8d %25d %25d", "UInt16", 2, UInt16.min, UInt16.max))
```

---

### แบบฝึกหัดที่ 5: String Manipulation

เขียนโปรแกรมจัดการ String:

```swift
// exercise_string.swift

var sentence = "  swift is awesome programming language  "

// 1. ลบ whitespace รอบข้าง
let trimmed = sentence.trimmingCharacters(in: .whitespaces)
print("Trimmed: '\(trimmed)'")

// 2. Capitalize ตัวแรกของแต่ละคำ
let titled = trimmed.capitalized
print("Titled: '\(titled)'")

// 3. แทนที่คำ
let modified = trimmed.replacingOccurrences(of: "awesome", with: "fantastic")
print("Modified: '\(modified)'")

// 4. นับจำนวนคำ
let words = trimmed.split(separator: " ")
print("จำนวนคำ: \(words.count)")

// 5. ตรวจสอบ palindrome
func isPalindrome(_ str: String) -> Bool {
    let clean = str.lowercased().filter { $0.isLetter }
    return clean == String(clean.reversed())
}

let testStrings = ["racecar", "hello", "level", "swift", "civic"]
for s in testStrings {
    print("'\(s)' เป็น palindrome: \(isPalindrome(s))")
}
```

---

## 20. ตัวอย่างโลกจริง

### 20.1 User Profile System

```swift
// real_world_example.swift

// Type Aliases สำหรับความชัดเจน
typealias UserID = String
typealias Email = String
typealias PhoneNumber = String
typealias Age = Int

// Constants สำหรับ Validation
let minAge: Age = 18
let maxAge: Age = 120
let maxUsernameLength = 30
let minPasswordLength = 8

// ข้อมูลผู้ใช้
let userId: UserID = "USR-2024-001"
let username = "swiftlearner"
let email: Email = "swift@example.com"
let phone: PhoneNumber = "+66812345678"
var userAge: Age = 25
var isEmailVerified = false
var isPhoneVerified = true
var loginCount: UInt = 0
var profileScore: Double = 0.0

// Computed Values
let isAdult = userAge >= minAge
let isValidUsername = username.count <= maxUsernameLength && !username.isEmpty
let canLogin = isEmailVerified || isPhoneVerified

// Login
loginCount += 1
profileScore = Double(loginCount) * 10.5

print("=== User Profile ===")
print("ID: \(userId)")
print("Username: \(username) (\(isValidUsername ? "valid" : "invalid"))")
print("Email: \(email) (\(isEmailVerified ? "verified" : "unverified"))")
print("Phone: \(phone) (\(isPhoneVerified ? "verified" : "unverified"))")
print("Age: \(userAge) (\(isAdult ? "Adult" : "Minor"))")
print("Can Login: \(canLogin)")
print("Login Count: \(loginCount)")
print(String(format: "Profile Score: %.1f", profileScore))
```

### 20.2 Shopping Cart

```swift
// shopping_cart.swift

// Types
typealias Price = Double
typealias Quantity = Int
typealias ItemName = String

// Constants
let taxRate: Double = 0.07          // 7% VAT
let freeShippingThreshold: Price = 500.0
let shippingCost: Price = 50.0

// Cart Items (ชั่วคราว ก่อนเรียน Array)
let item1Name: ItemName = "Swift Programming Book"
let item1Price: Price = 450.0
let item1Qty: Quantity = 2

let item2Name: ItemName = "Mechanical Keyboard"
let item2Price: Price = 2990.0
let item2Qty: Quantity = 1

let item3Name: ItemName = "iPhone Case"
let item3Price: Price = 299.0
let item3Qty: Quantity = 3

// คำนวณ
let item1Total = item1Price * Double(item1Qty)
let item2Total = item2Price * Double(item2Qty)
let item3Total = item3Price * Double(item3Qty)

let subtotal = item1Total + item2Total + item3Total
let tax = subtotal * taxRate
let shipping = subtotal >= freeShippingThreshold ? 0.0 : shippingCost
let total = subtotal + tax + shipping

// แสดงใบเสร็จ
print("╔══════════════════════════════════════════╗")
print("║              ใบเสร็จรับเงิน              ║")
print("╠══════════════════════════════════════════╣")
print(String(format: "║ %-25s x%d  %7.2f", item1Name, item1Qty, item1Total))
print(String(format: "║ %-25s x%d  %7.2f", item2Name, item2Qty, item2Total))
print(String(format: "║ %-25s x%d  %7.2f", item3Name, item3Qty, item3Total))
print("╠══════════════════════════════════════════╣")
print(String(format: "║ ราคารวม (Subtotal):          %9.2f ║", subtotal))
print(String(format: "║ ภาษีมูลค่าเพิ่ม 7%%:          %9.2f ║", tax))
print(String(format: "║ ค่าจัดส่ง:                  %9.2f ║", shipping))
if shipping == 0 {
    print("║ (ฟรีค่าจัดส่ง! ซื้อเกิน 500 บาท)       ║")
}
print("╠══════════════════════════════════════════╣")
print(String(format: "║ ยอดชำระ (Total):             %9.2f ║", total))
print("╚══════════════════════════════════════════╝")
```

---

## 21. สรุป

ในตอนนี้เราได้เรียนรู้พื้นฐานที่สำคัญที่สุดของ Swift:

### สิ่งที่เรียนมาแล้ว

| หัวข้อ | สาระสำคัญ |
|--------|---------|
| `var` vs `let` | `var` เปลี่ยนได้, `let` ไม่เปลี่ยน — ใช้ `let` เป็น Default |
| Type Inference | Swift อนุมาน Type ให้อัตโนมัติ |
| Type Annotation | `:TypeName` ระบุ Type ชัดๆ เมื่อจำเป็น |
| Integer Types | `Int` ปกติ, `Int8/16/32/64` สำหรับขนาดเฉพาะ |
| Float Types | `Double` เป็น Default, `Float` สำหรับ 32-bit |
| `Bool` | `true`/`false` เท่านั้น ไม่มี implicit conversion |
| `Character` | อักขระเดี่ยว รองรับ Unicode ครบ |
| `String` | Immutable Value Type, String Interpolation ด้วย `\()` |
| `typealias` | ชื่อแทน Type เพื่อความอ่านง่าย |
| Numeric Literals | `0b` Binary, `0o` Octal, `0x` Hex |
| Type Conversion | ต้องแปลงชัดๆ — Swift ไม่แปลงอัตโนมัติ |
| Overflow Operators | `&+`, `&-`, `&*` สำหรับ Wrap-around |
| Type Safety | Compiler ตรวจสอบ Type ก่อน Compile |
| Naming Convention | camelCase ตัวแปร/ฟังก์ชัน, PascalCase Types |
| `lazy var` | Initialize เมื่อถูกใช้ครั้งแรก |
| Computed Properties | คำนวณค่าทุกครั้งที่ถูกเรียก |

### กฎทอง (Golden Rules)

```
1. ใช้ let เสมอ เปลี่ยนเป็น var เมื่อจำเป็น
2. ปล่อยให้ Swift Infer Type เมื่อทำได้ชัดเจน
3. ตั้งชื่อให้ "อธิบายตัวเอง" (Self-documenting)
4. ไม่มี Implicit Type Conversion — ต้องแปลงชัดๆ
5. Bool ควรชื่อขึ้นต้นด้วย is/has/can/should/will/did
```

---

### ก้าวต่อไป

➡️ **[Part 03: Operators — ตัวดำเนินการทุกชนิด](Part03_Operators.md)**
- Arithmetic, Comparison, Logical Operators
- Assignment Operators
- Range Operators
- Nil-Coalescing Operator
- Custom Operators

---

## 📚 อ่านเพิ่มเติม

| แหล่งข้อมูล | หัวข้อ |
|------------|-------|
| [Swift Book — The Basics](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/thebasics/) | ตัวแปร, ชนิดข้อมูล |
| [Swift Book — Strings](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/stringsandcharacters/) | String อย่างละเอียด |
| [Swift Book — Collection Types](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/collectiontypes/) | Array, Dictionary, Set |
| [Hacking with Swift](https://www.hackingwithswift.com/quick-start/beginners) | ตัวอย่างสนุกๆ |

---

*Part 02 จาก 100 | หลักสูตร Swift ฉบับสมบูรณ์*
