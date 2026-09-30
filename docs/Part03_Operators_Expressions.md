# ส่วนที่ 3: ตัวดำเนินการและนิพจน์ใน Swift (Operators and Expressions)

## บทนำ (Introduction)

ตัวดำเนินการ (Operators) เป็นสัญลักษณ์พิเศษที่ใช้ในการทำงานกับค่าต่าง ๆ ใน Swift ตัวอย่างเช่น `+` ใช้บวกตัวเลข, `==` ใช้เปรียบเทียบค่าสองค่าว่าเท่ากันหรือไม่ Swift มีตัวดำเนินการหลายประเภทที่ครอบคลุมการใช้งานทุกด้าน ตั้งแต่การคำนวณทางคณิตศาสตร์ไปจนถึงการจัดการบิต

ในบทนี้เราจะเรียนรู้:
- ตัวดำเนินการทางคณิตศาสตร์ (Arithmetic Operators)
- ตัวดำเนินการกำหนดค่า (Assignment Operators)
- ตัวดำเนินการเปรียบเทียบ (Comparison Operators)
- ตัวดำเนินการตรรกะ (Logical Operators)
- ตัวดำเนินการระดับบิต (Bitwise Operators)
- ตัวดำเนินการช่วง (Range Operators)
- ตัวดำเนินการ Nil-Coalescing
- ตัวดำเนินการเงื่อนไขสามส่วน (Ternary Operator)
- ตัวดำเนินการอัตลักษณ์ (Identity Operators)
- ลำดับความสำคัญและการเชื่อมโยงของตัวดำเนินการ
- ตัวดำเนินการที่กำหนดเอง (Custom Operators)
- ตัวดำเนินการ Overflow
- การเชื่อมต่อ String และ String Interpolation

---

## 3.1 ตัวดำเนินการทางคณิตศาสตร์ (Arithmetic Operators)

Swift รองรับตัวดำเนินการทางคณิตศาสตร์มาตรฐานสำหรับชนิดข้อมูลตัวเลขทุกชนิด

### ตัวดำเนินการพื้นฐาน

| ตัวดำเนินการ | ชื่อ | ตัวอย่าง |
|---|---|---|
| `+` | บวก (Addition) | `5 + 3 = 8` |
| `-` | ลบ (Subtraction) | `5 - 3 = 2` |
| `*` | คูณ (Multiplication) | `5 * 3 = 15` |
| `/` | หาร (Division) | `10 / 2 = 5` |
| `%` | หารเอาเศษ (Remainder) | `10 % 3 = 1` |

### ตัวอย่างการใช้งาน

```swift
// ตัวดำเนินการทางคณิตศาสตร์พื้นฐาน
let a = 10
let b = 3

let sum = a + b         // 13 (การบวก)
let difference = a - b  // 7  (การลบ)
let product = a * b     // 30 (การคูณ)
let quotient = a / b    // 3  (การหาร - ผลลัพธ์เป็น Int จึงตัดทศนิยม)
let remainder = a % b   // 1  (หารเอาเศษ)

print("การบวก: \(a) + \(b) = \(sum)")
print("การลบ: \(a) - \(b) = \(difference)")
print("การคูณ: \(a) * \(b) = \(product)")
print("การหาร: \(a) / \(b) = \(quotient)")
print("หารเอาเศษ: \(a) % \(b) = \(remainder)")
```

### การหารทศนิยม

```swift
// การหารกับ Double
let x: Double = 10.0
let y: Double = 3.0

let floatDivision = x / y  // 3.3333...
print("การหาร Double: \(x) / \(y) = \(floatDivision)")

// ระวัง! การหาร Int จะตัดทศนิยม
let intA: Int = 7
let intB: Int = 2
let intResult = intA / intB  // 3 ไม่ใช่ 3.5!
print("การหาร Int: \(intA) / \(intB) = \(intResult)")

// แปลงเป็น Double ก่อนหาร
let doubleResult = Double(intA) / Double(intB)  // 3.5
print("การหาร (แปลงก่อน): \(intA) / \(intB) = \(doubleResult)")
```

### ตัวดำเนินการ Unary (ตัวดำเนินการเดี่ยว)

```swift
// Unary minus (-) - เปลี่ยนเครื่องหมาย
let positive = 5
let negative = -positive  // -5
let backToPositive = -negative  // 5

print("Unary minus: \(-positive)")

// Unary plus (+) - ไม่เปลี่ยนค่า (มีไว้เพื่อความสมมาตร)
let three = 3
let alsoThree = +three  // 3

print("Unary plus: \(+three)")
```

### ตัวดำเนินการ Remainder (%) กับตัวเลขทศนิยม

```swift
// % ใช้ได้กับ Double ด้วย
let doubleRemainder = 8.5 % 2.5  // 1.0
print("ทศนิยม Remainder: 8.5 % 2.5 = \(doubleRemainder)")

// ประโยชน์ของ remainder
let seconds = 125
let minutes = seconds / 60      // 2 นาที
let remainingSeconds = seconds % 60  // 5 วินาที
print("\(seconds) วินาที = \(minutes) นาที \(remainingSeconds) วินาที")
```

### ตัวอย่างจริง: เครื่องคิดเลขพื้นฐาน

```swift
import Foundation

func basicCalculator(num1: Double, num2: Double, operation: String) -> Double? {
    switch operation {
    case "+":
        return num1 + num2
    case "-":
        return num1 - num2
    case "*":
        return num1 * num2
    case "/":
        guard num2 != 0 else {
            print("ข้อผิดพลาด: ไม่สามารถหารด้วยศูนย์ได้!")
            return nil
        }
        return num1 / num2
    case "%":
        guard num2 != 0 else {
            print("ข้อผิดพลาด: ไม่สามารถหารด้วยศูนย์ได้!")
            return nil
        }
        return num1.truncatingRemainder(dividingBy: num2)
    default:
        print("ข้อผิดพลาด: ตัวดำเนินการไม่ถูกต้อง")
        return nil
    }
}

// ทดสอบเครื่องคิดเลข
if let result = basicCalculator(num1: 15.0, num2: 4.0, operation: "/") {
    print("15 / 4 = \(result)")  // 3.75
}

if let result = basicCalculator(num1: 17.0, num2: 5.0, operation: "%") {
    print("17 % 5 = \(result)")  // 2.0
}
```

---

## 3.2 ตัวดำเนินการกำหนดค่า (Assignment Operators)

### ตัวดำเนินการกำหนดค่าพื้นฐาน (=)

```swift
// การกำหนดค่าพื้นฐาน
var score = 100
var playerName = "Alice"
let maxScore = 1000

// กำหนดค่าจาก tuple
let (x, y) = (10, 20)
print("x = \(x), y = \(y)")

// ข้อสังเกต: = ใน Swift ไม่คืนค่า (ต่างจาก C/C++)
// var a = 5
// if a = 10 { ... }  // ข้อผิดพลาด! ใน Swift
```

### ตัวดำเนินการกำหนดค่าประกอบ (Compound Assignment Operators)

```swift
var number = 10

// += (บวกแล้วกำหนดค่า)
number += 5   // number = number + 5 = 15
print("หลัง +=5: \(number)")

// -= (ลบแล้วกำหนดค่า)
number -= 3   // number = number - 3 = 12
print("หลัง -=3: \(number)")

// *= (คูณแล้วกำหนดค่า)
number *= 2   // number = number * 2 = 24
print("หลัง *=2: \(number)")

// /= (หารแล้วกำหนดค่า)
number /= 4   // number = number / 4 = 6
print("หลัง /=4: \(number)")

// %= (หารเอาเศษแล้วกำหนดค่า)
number %= 4   // number = number % 4 = 2
print("หลัง %=4: \(number)")
```

### ตัวอย่างจริง: การสะสมคะแนน

```swift
// ระบบสะสมคะแนน
var totalScore = 0
var multiplier = 1

// รอบที่ 1
let round1Score = 150
totalScore += round1Score
print("หลังรอบ 1: \(totalScore)")

// รอบที่ 2 (คะแนนคูณ 2)
let round2Score = 200
multiplier *= 2
totalScore += round2Score * multiplier
print("หลังรอบ 2: \(totalScore)")

// รอบที่ 3 (ลดคะแนน)
let penalty = 50
totalScore -= penalty
print("หลังหักคะแนน: \(totalScore)")

// ตรวจสอบขีดสูงสุด
let maxAllowed = 1000
if totalScore > maxAllowed {
    totalScore = maxAllowed
    print("คะแนนเกินขีดสูงสุด ปรับเป็น: \(totalScore)")
}

print("คะแนนสุดท้าย: \(totalScore)")
```

---

## 3.3 ตัวดำเนินการเปรียบเทียบ (Comparison Operators)

ตัวดำเนินการเปรียบเทียบใน Swift คืนค่า `Bool` (`true` หรือ `false`)

| ตัวดำเนินการ | ความหมาย |
|---|---|
| `==` | เท่ากับ (Equal to) |
| `!=` | ไม่เท่ากับ (Not equal to) |
| `>` | มากกว่า (Greater than) |
| `<` | น้อยกว่า (Less than) |
| `>=` | มากกว่าหรือเท่ากับ (Greater than or equal to) |
| `<=` | น้อยกว่าหรือเท่ากับ (Less than or equal to) |

```swift
let age = 25

print(age == 25)    // true  - เท่ากับ
print(age != 30)    // true  - ไม่เท่ากับ
print(age > 18)     // true  - มากกว่า
print(age < 30)     // true  - น้อยกว่า
print(age >= 25)    // true  - มากกว่าหรือเท่ากับ
print(age <= 25)    // true  - น้อยกว่าหรือเท่ากับ
```

### การเปรียบเทียบ String

```swift
// เปรียบเทียบ String (case-sensitive)
let name1 = "Alice"
let name2 = "alice"
let name3 = "Alice"

print(name1 == name2)  // false (A ≠ a)
print(name1 == name3)  // true
print(name1 < name2)   // true (A มาก่อน a ใน Unicode)

// เปรียบเทียบโดยไม่สนใจตัวพิมพ์เล็ก/ใหญ่
print(name1.lowercased() == name2.lowercased())  // true
```

### การเปรียบเทียบ Tuple

```swift
// Tuple เปรียบเทียบจากซ้ายไปขวา
let point1 = (1, 2)
let point2 = (1, 3)
let point3 = (2, 1)

print(point1 < point2)  // true (1==1, ดังนั้นเปรียบ 2<3)
print(point1 < point3)  // true (1<2)
print(point2 > point3)  // false (1<2)

// หมายเหตุ: Tuple เปรียบเทียบได้สูงสุด 6 elements
let comparison = (1, "apple") < (1, "banana")  // true
print(comparison)
```

### ตัวอย่างจริง: ระบบให้คะแนนนักเรียน

```swift
func gradeEvaluation(score: Int) -> String {
    if score >= 90 {
        return "A - ยอดเยี่ยม!"
    } else if score >= 80 {
        return "B - ดีมาก"
    } else if score >= 70 {
        return "C - ดี"
    } else if score >= 60 {
        return "D - พอใช้"
    } else {
        return "F - ไม่ผ่าน"
    }
}

let scores = [95, 82, 73, 65, 45]
for score in scores {
    print("คะแนน \(score): \(gradeEvaluation(score: score))")
}
```

---

## 3.4 ตัวดำเนินการตรรกะ (Logical Operators)

ตัวดำเนินการตรรกะใช้รวมหรือกลับค่า Boolean

| ตัวดำเนินการ | ชื่อ | คำอธิบาย |
|---|---|---|
| `!` | NOT | กลับค่า Boolean |
| `&&` | AND | true เมื่อทั้งสองเป็น true |
| `\|\|` | OR | true เมื่ออย่างน้อยหนึ่งเป็น true |

### Logical NOT (!)

```swift
var isLoggedIn = false

print(!isLoggedIn)   // true
print(!true)         // false
print(!false)        // true

// ตัวอย่างการใช้งาน
if !isLoggedIn {
    print("กรุณาเข้าสู่ระบบ")
}
```

### Logical AND (&&)

```swift
// && เป็น true เมื่อทั้งสองด้านเป็น true
let hasAccount = true
let hasPassword = true
let isVerified = false

print(hasAccount && hasPassword)    // true
print(hasAccount && isVerified)     // false
print(hasPassword && isVerified)    // false

// Short-circuit evaluation
// ถ้าด้านซ้ายเป็น false ด้านขวาจะไม่ถูกประเมิน
func checkCondition() -> Bool {
    print("ตรวจสอบเงื่อนไขที่สอง...")
    return true
}

let falseValue = false
let result = falseValue && checkCondition()  // checkCondition() จะไม่ถูกเรียก!
print("ผลลัพธ์: \(result)")
```

### Logical OR (||)

```swift
// || เป็น true เมื่ออย่างน้อยหนึ่งด้านเป็น true
let isAdmin = false
let isModerator = true
let isOwner = false

print(isAdmin || isModerator)   // true
print(isAdmin || isOwner)       // false
print(isAdmin || isModerator || isOwner)  // true

// Short-circuit evaluation สำหรับ ||
// ถ้าด้านซ้ายเป็น true ด้านขวาจะไม่ถูกประเมิน
let trueValue = true
let result2 = trueValue || checkCondition()  // checkCondition() จะไม่ถูกเรียก!
```

### การรวมตัวดำเนินการตรรกะ

```swift
// ตัวอย่าง: ตรวจสอบสิทธิ์การเข้าถึง
let age2 = 20
let hasID = true
let isVIP = false

// เข้าได้ถ้า: อายุ >= 18 AND มี ID, หรือ เป็น VIP
let canEnter = (age2 >= 18 && hasID) || isVIP
print("เข้าได้: \(canEnter)")  // true

// ตัวอย่างที่ซับซ้อนขึ้น
let temperature = 25.0
let isRaining = false
let hasUmbrella = true

let isGoodWeather = temperature > 20 && temperature < 35 && !isRaining
let isPrepared = isRaining && hasUmbrella

print("อากาศดี: \(isGoodWeather)")      // true
print("พร้อมรับฝน: \(isPrepared)")      // false
```

### ตัวอย่างจริง: ระบบตรวจสอบรหัสผ่าน

```swift
func validatePassword(_ password: String) -> (isValid: Bool, message: String) {
    let hasMinLength = password.count >= 8
    let hasUppercase = password.contains { $0.isUppercase }
    let hasLowercase = password.contains { $0.isLowercase }
    let hasDigit = password.contains { $0.isNumber }
    let hasSpecialChar = password.contains { "!@#$%^&*".contains($0) }
    
    if !hasMinLength {
        return (false, "รหัสผ่านต้องมีความยาวอย่างน้อย 8 ตัวอักษร")
    }
    
    if !hasUppercase || !hasLowercase {
        return (false, "รหัสผ่านต้องมีทั้งตัวพิมพ์ใหญ่และตัวพิมพ์เล็ก")
    }
    
    if !hasDigit {
        return (false, "รหัสผ่านต้องมีตัวเลขอย่างน้อย 1 ตัว")
    }
    
    if hasMinLength && hasUppercase && hasLowercase && hasDigit && hasSpecialChar {
        return (true, "รหัสผ่านแข็งแกร่งมาก!")
    }
    
    return (true, "รหัสผ่านผ่านเกณฑ์ขั้นต่ำ")
}

let passwords = ["abc", "Password1", "P@ssw0rd!"]
for pwd in passwords {
    let result3 = validatePassword(pwd)
    print("'\(pwd)': \(result3.isValid ? "✓" : "✗") - \(result3.message)")
}
```

---

## 3.5 ตัวดำเนินการระดับบิต (Bitwise Operators)

ตัวดำเนินการระดับบิตใช้จัดการข้อมูลในระดับ Binary

### Bitwise NOT (~)

```swift
// ~ กลับค่าทุกบิต
let bits: UInt8 = 0b00001111  // 15
let invertedBits = ~bits       // 0b11110000 = 240

print("ค่าเดิม: \(bits) (binary: \(String(bits, radix: 2, uppercase: false)))")
print("กลับบิต: \(invertedBits)")
```

### Bitwise AND (&)

```swift
// & ผลลัพธ์เป็น 1 เมื่อทั้งสองบิตเป็น 1
let firstBits: UInt8  = 0b11111100  // 252
let secondBits: UInt8 = 0b00111111  // 63
let andResult = firstBits & secondBits  // 0b00111100 = 60

print("AND Result: \(andResult)")

// ประโยชน์: ตรวจสอบว่าบิตหนึ่งถูก set หรือไม่
let permissions: UInt8 = 0b00000111  // 7 (read=1, write=2, execute=4)
let readPermission: UInt8 = 0b00000001
let writePermission: UInt8 = 0b00000010

let canRead = (permissions & readPermission) != 0    // true
let canWrite = (permissions & writePermission) != 0  // true
print("สามารถอ่าน: \(canRead)")
print("สามารถเขียน: \(canWrite)")
```

### Bitwise OR (|)

```swift
// | ผลลัพธ์เป็น 1 เมื่ออย่างน้อยหนึ่งบิตเป็น 1
let someBits: UInt8 = 0b10110010  // 178
let moreBits: UInt8 = 0b01011110  // 94
let orResult = someBits | moreBits  // 0b11111110 = 254

print("OR Result: \(orResult)")

// ประโยชน์: เพิ่ม permission
var userPermissions: UInt8 = 0b00000001  // read only
let addWrite: UInt8 = 0b00000010
userPermissions |= addWrite  // เพิ่มสิทธิ์เขียน
print("Permissions หลังเพิ่ม: \(userPermissions)")  // 3
```

### Bitwise XOR (^)

```swift
// ^ ผลลัพธ์เป็น 1 เมื่อบิตสองตัวต่างกัน
let firstBits2: UInt8 = 0b00010100  // 20
let otherBits: UInt8  = 0b00000101  // 5
let xorResult = firstBits2 ^ otherBits  // 0b00010001 = 17

print("XOR Result: \(xorResult)")

// ประโยชน์: Toggle bit
var flag: UInt8 = 0b00000001  // on
let toggleBit: UInt8 = 0b00000001
flag ^= toggleBit  // toggle (off)
print("หลัง toggle: \(flag)")  // 0
flag ^= toggleBit  // toggle (on)
print("หลัง toggle อีกครั้ง: \(flag)")  // 1
```

### Bitwise Shift Operators (<< และ >>)

```swift
// << เลื่อนบิตไปซ้าย (คูณด้วย 2^n)
let shiftLeft: UInt8 = 0b00001111  // 15
let leftShifted = shiftLeft << 1   // 0b00011110 = 30

print("Shift left 1: \(leftShifted)")  // 30

// >> เลื่อนบิตไปขวา (หารด้วย 2^n)
let shiftRight: UInt8 = 0b11110000  // 240
let rightShifted = shiftRight >> 1  // 0b01111000 = 120

print("Shift right 1: \(rightShifted)")  // 120

// ตัวอย่างจริง: แปลงสี RGB จากค่า hex
let color: UInt32 = 0xFF5733  // สีส้มแดง
let red   = UInt8((color >> 16) & 0xFF)   // 255
let green = UInt8((color >> 8) & 0xFF)    // 87
let blue  = UInt8(color & 0xFF)           // 51

print("RGB: (\(red), \(green), \(blue))")
```

---

## 3.6 ตัวดำเนินการช่วง (Range Operators)

### Closed Range Operator (...)

```swift
// a...b รวม a และ b
let closedRange = 1...5  // 1, 2, 3, 4, 5

for i in closedRange {
    print(i, terminator: " ")
}
print()  // output: 1 2 3 4 5

// ใช้กับ Array
let names = ["Anna", "Ben", "Clara", "David", "Eve"]
let firstThree = names[0...2]  // Anna, Ben, Clara
print(firstThree)

// One-sided range
let suffix = names[2...]  // ตั้งแต่ index 2 ถึงท้าย
let prefix = names[...2]  // ตั้งแต่ต้นถึง index 2
print(Array(suffix))
print(Array(prefix))
```

### Half-Open Range Operator (..<)

```swift
// a..<b รวม a แต่ไม่รวม b
let halfOpenRange = 0..<5  // 0, 1, 2, 3, 4

for i in halfOpenRange {
    print(i, terminator: " ")
}
print()  // output: 0 1 2 3 4

// มักใช้กับ Array index (เพราะ Array เริ่มที่ 0)
let colors = ["แดง", "เขียว", "น้ำเงิน"]
for i in 0..<colors.count {
    print("สี[\(i)]: \(colors[i])")
}
```

### ตัวอย่างจริง: การจัดกลุ่มช่วงอายุ

```swift
func ageGroup(age: Int) -> String {
    switch age {
    case 0..<13:
        return "เด็ก (0-12 ปี)"
    case 13..<18:
        return "วัยรุ่น (13-17 ปี)"
    case 18..<60:
        return "ผู้ใหญ่ (18-59 ปี)"
    case 60...:
        return "ผู้สูงอายุ (60+ ปี)"
    default:
        return "ไม่ทราบ"
    }
}

let ages = [5, 15, 30, 65]
for age in ages {
    print("อายุ \(age): \(ageGroup(age: age))")
}
```

---

## 3.7 Nil-Coalescing Operator (??)

`??` ให้ค่า default เมื่อ Optional มีค่าเป็น `nil`

```swift
// รูปแบบ: a ?? b
// ถ้า a ไม่ใช่ nil ให้ใช้ค่าของ a
// ถ้า a เป็น nil ให้ใช้ค่า b

var username: String? = "Alice"
var guestName: String? = nil

let displayName1 = username ?? "ผู้เยี่ยมชม"    // "Alice"
let displayName2 = guestName ?? "ผู้เยี่ยมชม"   // "ผู้เยี่ยมชม"

print(displayName1)
print(displayName2)
```

### การซ้อน ?? หลายชั้น

```swift
var primaryName: String? = nil
var secondaryName: String? = nil
var fallbackName: String? = "ไม่ระบุชื่อ"

let finalName = primaryName ?? secondaryName ?? fallbackName ?? "Anonymous"
print(finalName)  // "ไม่ระบุชื่อ"
```

### ตัวอย่างจริง: การตั้งค่าเริ่มต้น

```swift
struct UserSettings {
    var theme: String?
    var fontSize: Int?
    var language: String?
}

let defaultTheme = "Light"
let defaultFontSize = 16
let defaultLanguage = "ไทย"

let userSettings = UserSettings(theme: nil, fontSize: 18, language: nil)

let theme = userSettings.theme ?? defaultTheme
let fontSize = userSettings.fontSize ?? defaultFontSize
let language = userSettings.language ?? defaultLanguage

print("ธีม: \(theme)")         // "Light"
print("ขนาดตัวอักษร: \(fontSize)")  // 18 (ค่าที่ user ตั้ง)
print("ภาษา: \(language)")     // "ไทย"
```

---

## 3.8 ตัวดำเนินการเงื่อนไขสามส่วน (Ternary Conditional Operator)

รูปแบบ: `condition ? valueIfTrue : valueIfFalse`

```swift
// รูปแบบพื้นฐาน
let temperature2 = 30
let weather = temperature2 > 25 ? "ร้อน" : "เย็น"
print("อากาศ: \(weather)")  // "ร้อน"

// เทียบกับ if-else
let score2 = 75
var result4: String
if score2 >= 60 {
    result4 = "ผ่าน"
} else {
    result4 = "ไม่ผ่าน"
}
// เขียนแบบ ternary สั้นกว่า:
let result5 = score2 >= 60 ? "ผ่าน" : "ไม่ผ่าน"

print(result4)
print(result5)
```

### ตัวอย่างการใช้ใน String Interpolation

```swift
let count = 5
let itemLabel = count == 1 ? "รายการ" : "รายการ"  // ภาษาไทยไม่ต้องเปลี่ยน
let englishLabel = count == 1 ? "item" : "items"

print("มี \(count) \(englishLabel)")  // "มี 5 items"

// การซ้อน ternary (ควรหลีกเลี่ยงเพื่อความชัดเจน)
let value = 15
let category = value < 0 ? "ลบ" : value == 0 ? "ศูนย์" : "บวก"
print("หมวดหมู่: \(category)")  // "บวก"
```

### ตัวอย่างจริง

```swift
struct Product {
    let name: String
    let price: Double
    let discount: Double  // เปอร์เซ็นต์ส่วนลด (0-100)
    
    var finalPrice: Double {
        let discountAmount = price * (discount / 100)
        return discount > 0 ? price - discountAmount : price
    }
    
    var priceLabel: String {
        return discount > 0 ? "ราคาลด: \(finalPrice) บาท" : "ราคา: \(finalPrice) บาท"
    }
}

let products = [
    Product(name: "เสื้อ", price: 500, discount: 20),
    Product(name: "กางเกง", price: 800, discount: 0),
    Product(name: "รองเท้า", price: 1200, discount: 15)
]

for product in products {
    print("\(product.name): \(product.priceLabel)")
}
```

---

## 3.9 ตัวดำเนินการอัตลักษณ์ (Identity Operators)

ใช้เปรียบเทียบว่าตัวแปรสองตัวชี้ไปยัง object เดียวกันหรือไม่ (สำหรับ class เท่านั้น)

```swift
class Person {
    let name: String
    let age: Int
    
    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }
}

let person1 = Person(name: "Alice", age: 25)
let person2 = person1       // ชี้ไปยัง object เดียวกัน
let person3 = Person(name: "Alice", age: 25)  // object ใหม่

// === ตรวจสอบว่าเป็น instance เดียวกัน
print(person1 === person2)  // true (same instance)
print(person1 === person3)  // false (different instances)

// !== ตรงข้ามกับ ===
print(person1 !== person3)  // true
```

### ความแตกต่างระหว่าง == และ ===

```swift
// == เปรียบเทียบค่า (ต้องใช้ protocol Equatable)
// === เปรียบเทียบตำแหน่งใน memory

class Car: Equatable {
    let model: String
    let year: Int
    
    init(model: String, year: Int) {
        self.model = model
        self.year = year
    }
    
    static func == (lhs: Car, rhs: Car) -> Bool {
        return lhs.model == rhs.model && lhs.year == rhs.year
    }
}

let car1 = Car(model: "Tesla", year: 2023)
let car2 = Car(model: "Tesla", year: 2023)
let car3 = car1

print(car1 == car2)   // true  (ค่าเหมือนกัน)
print(car1 === car2)  // false (คนละ object)
print(car1 === car3)  // true  (object เดียวกัน)
```

---

## 3.10 ลำดับความสำคัญของตัวดำเนินการ (Operator Precedence)

```swift
// ตัวอย่างที่แสดงให้เห็นลำดับความสำคัญ
let result6 = 2 + 3 * 4    // 14 ไม่ใช่ 20 (* มีความสำคัญกว่า +)
let result7 = (2 + 3) * 4  // 20 (วงเล็บบังคับให้คำนวณ + ก่อน)

print(result6)  // 14
print(result7)  // 20

// ลำดับตัวดำเนินการ (สูงไปต่ำ):
// 1. ตัวดำเนินการ Unary: - ! ~ ++ --
// 2. การคูณ/หาร: * / %
// 3. การบวก/ลบ: + -
// 4. Bitwise Shift: << >>
// 5. การเปรียบเทียบ: < <= > >=
// 6. ความเท่ากัน: == !=
// 7. Bitwise AND: &
// 8. Bitwise XOR: ^
// 9. Bitwise OR: |
// 10. Logical AND: &&
// 11. Logical OR: ||
// 12. Ternary: ? :
// 13. การกำหนดค่า: = += -= *= /= %=
```

### ตัวอย่างลำดับที่ซับซ้อน

```swift
// ตัวอย่างที่ต้องระวัง
let a2 = 5
let b2 = 3
let c2 = 2

let expr1 = a2 + b2 * c2         // 11 (คูณก่อน บวกทีหลัง)
let expr2 = (a2 + b2) * c2       // 16
let expr3 = a2 * b2 + c2 * a2    // 25 (15 + 10)
let expr4 = a2 * (b2 + c2 * a2)  // 65 (5 * 13)

print("a + b * c = \(expr1)")
print("(a + b) * c = \(expr2)")
print("a * b + c * a = \(expr3)")
print("a * (b + c * a) = \(expr4)")

// คำแนะนำ: ใช้วงเล็บเมื่อไม่แน่ใจ เพื่อความชัดเจน
```

---

## 3.11 ตัวดำเนินการที่กำหนดเอง (Custom Operators)

Swift อนุญาตให้สร้างตัวดำเนินการใหม่ได้

### การประกาศ Custom Operator

```swift
// ประกาศ prefix operator
prefix operator **

// กำหนด implementation
prefix func ** (value: Int) -> Int {
    return value * value
}

let squared = **5
print("5^2 = \(squared)")  // 25

// ประกาศ infix operator
infix operator **: MultiplicationPrecedence

func ** (lhs: Double, rhs: Double) -> Double {
    return pow(lhs, rhs)
}

let powerResult = 2.0 ** 10.0
print("2^10 = \(powerResult)")  // 1024.0
```

### Custom Operator กับ Struct

```swift
struct Vector2D {
    var x: Double
    var y: Double
}

// เพิ่มตัวดำเนินการ + สำหรับ Vector2D
func + (left: Vector2D, right: Vector2D) -> Vector2D {
    return Vector2D(x: left.x + right.x, y: left.y + right.y)
}

// เพิ่มตัวดำเนินการ - สำหรับ Vector2D
func - (left: Vector2D, right: Vector2D) -> Vector2D {
    return Vector2D(x: left.x - right.x, y: left.y - right.y)
}

// เพิ่มตัวดำเนินการ * (scalar multiplication)
func * (vector: Vector2D, scalar: Double) -> Vector2D {
    return Vector2D(x: vector.x * scalar, y: vector.y * scalar)
}

let v1 = Vector2D(x: 1.0, y: 2.0)
let v2 = Vector2D(x: 3.0, y: 4.0)

let sum3 = v1 + v2    // Vector2D(x: 4.0, y: 6.0)
let diff = v1 - v2    // Vector2D(x: -2.0, y: -2.0)
let scaled = v1 * 3.0 // Vector2D(x: 3.0, y: 6.0)

print("v1 + v2 = (\(sum3.x), \(sum3.y))")
print("v1 - v2 = (\(diff.x), \(diff.y))")
print("v1 * 3 = (\(scaled.x), \(scaled.y))")
```

---

## 3.12 Overflow Operators

ใน Swift การ overflow ปกติจะเกิด runtime error แต่เราสามารถใช้ overflow operators เพื่อให้ wrap around แทน

```swift
// ปกติ: ค่าเกิน UInt8 จะ crash
// let overflow: UInt8 = 255 + 1  // Error!

// ใช้ overflow operators
var maxUInt8: UInt8 = 255
let overflowResult = maxUInt8 &+ 1   // wrap around เป็น 0
print("255 &+ 1 = \(overflowResult)")  // 0

var minUInt8: UInt8 = 0
let underflowResult = minUInt8 &- 1  // wrap around เป็น 255
print("0 &- 1 = \(underflowResult)")  // 255

// &* overflow multiply
var bigNumber: UInt8 = 200
let multiplyResult = bigNumber &* 2  // 400 % 256 = 144
print("200 &* 2 = \(multiplyResult)")  // 144
```

### ตัวอย่างการใช้งาน Overflow Operators

```swift
// ใช้ใน game score ที่อาจ wrap around
struct GameScore {
    var score: UInt32 = 0
    
    mutating func addPoints(_ points: UInt32) {
        score = score &+ points  // ป้องกัน overflow
    }
    
    mutating func subtractPoints(_ points: UInt32) {
        score = score &- points  // อาจ wrap to very large number
    }
}

var game = GameScore()
game.addPoints(UInt32.max)  // max value
print("คะแนน: \(game.score)")
game.addPoints(1)  // wrap around
print("หลัง overflow: \(game.score)")  // 0
```

---

## 3.13 String Concatenation และ String Interpolation

### การเชื่อมต่อ String ด้วย +

```swift
let firstName = "สมชาย"
let lastName = "ใจดี"

// การใช้ +
let fullName = firstName + " " + lastName
print(fullName)  // "สมชาย ใจดี"

// การใช้ +=
var greeting = "สวัสดี"
greeting += " "
greeting += firstName
print(greeting)  // "สวัสดี สมชาย"

// เชื่อมต่อหลายส่วน
let parts = ["มีนาคม", " ", "2024"]
let date = parts.joined()  // วิธีที่ดีกว่าสำหรับหลายส่วน
print(date)
```

### String Interpolation (\(...))

```swift
let name = "Alice"
let age3 = 25
let score3 = 95.5

// รูปแบบพื้นฐาน
let message = "ชื่อ: \(name), อายุ: \(age3), คะแนน: \(score3)"
print(message)

// นิพจน์ใน interpolation
let area = "\(3 * 4) ตารางเมตร"  // "12 ตารางเมตร"
print(area)

// การเรียกใช้ method
let uppercaseName = "ชื่อของฉัน: \(name.uppercased())"
print(uppercaseName)  // "ชื่อของฉัน: ALICE"

// การจัดรูปแบบตัวเลข
let pi = 3.14159
let formattedPi = String(format: "π ≈ %.2f", pi)  // "π ≈ 3.14"
print(formattedPi)
```

### Custom String Interpolation

```swift
// Swift 5+ รองรับ custom interpolation
extension String.StringInterpolation {
    mutating func appendInterpolation(_ value: Double, format: String) {
        let formatted = String(format: format, value)
        appendLiteral(formatted)
    }
}

let price = 1234.5678
let formattedMessage = "ราคา: \(price, format: "%.2f") บาท"
print(formattedMessage)  // "ราคา: 1234.57 บาท"
```

---

## 3.14 Operator Overloading

```swift
struct Money {
    var amount: Double
    var currency: String
    
    init(_ amount: Double, currency: String = "THB") {
        self.amount = amount
        self.currency = currency
    }
}

// Operator overloading สำหรับ Money
extension Money: CustomStringConvertible {
    var description: String {
        return String(format: "%.2f \(currency)", amount)
    }
}

// + operator
func + (lhs: Money, rhs: Money) -> Money {
    assert(lhs.currency == rhs.currency, "ไม่สามารถรวมสกุลเงินต่างกัน")
    return Money(lhs.amount + rhs.amount, currency: lhs.currency)
}

// - operator
func - (lhs: Money, rhs: Money) -> Money {
    assert(lhs.currency == rhs.currency, "ไม่สามารถลบสกุลเงินต่างกัน")
    return Money(lhs.amount - rhs.amount, currency: lhs.currency)
}

// * operator (คูณด้วย factor)
func * (money: Money, factor: Double) -> Money {
    return Money(money.amount * factor, currency: money.currency)
}

// == operator
extension Money: Equatable {
    static func == (lhs: Money, rhs: Money) -> Bool {
        return lhs.amount == rhs.amount && lhs.currency == rhs.currency
    }
}

// < operator
extension Money: Comparable {
    static func < (lhs: Money, rhs: Money) -> Bool {
        return lhs.amount < rhs.amount
    }
}

// ทดสอบ
let price1 = Money(100.0)
let price2 = Money(250.0)

let total = price1 + price2
print("รวม: \(total)")    // 350.00 THB

let discount = total * 0.9
print("ลด 10%: \(discount)")  // 315.00 THB

print("price1 < price2: \(price1 < price2)")  // true
print("price1 == price2: \(price1 == price2)")  // false
```

---

## 3.15 แบบฝึกหัดพร้อมเฉลย (Exercises with Solutions)

### แบบฝึกหัดที่ 1: เครื่องคิดเลข BMI

```swift
// โจทย์: สร้าง function คำนวณ BMI และระบุหมวดหมู่
// BMI = น้ำหนัก(kg) / ส่วนสูง(m)^2

func calculateBMI(weight: Double, height: Double) -> (bmi: Double, category: String) {
    let bmi = weight / (height * height)
    
    let category: String
    switch bmi {
    case ..<18.5:
        category = "น้ำหนักน้อย (Underweight)"
    case 18.5..<25.0:
        category = "น้ำหนักปกติ (Normal)"
    case 25.0..<30.0:
        category = "น้ำหนักเกิน (Overweight)"
    default:
        category = "โรคอ้วน (Obese)"
    }
    
    return (bmi, category)
}

let result8 = calculateBMI(weight: 70.0, height: 1.75)
print(String(format: "BMI: %.2f - %@", result8.bmi, result8.category))
```

### แบบฝึกหัดที่ 2: การคำนวณดอกเบี้ย

```swift
// โจทย์: คำนวณดอกเบี้ยแบบทบต้น
// A = P * (1 + r/n)^(n*t)

func compoundInterest(principal: Double, rate: Double, timesPerYear: Int, years: Int) -> Double {
    let r = rate / 100.0
    let n = Double(timesPerYear)
    let t = Double(years)
    return principal * pow(1 + r/n, n * t)
}

let principal = 10000.0  // เงินต้น 10,000 บาท
let rate = 5.0           // อัตราดอกเบี้ย 5%
let timesPerYear = 12    // ทบต้นทุกเดือน
let years = 5            // เป็นเวลา 5 ปี

let totalAmount = compoundInterest(principal: principal, rate: rate, 
                                   timesPerYear: timesPerYear, years: years)
let interest = totalAmount - principal

print(String(format: "เงินต้น: %.2f บาท", principal))
print(String(format: "ดอกเบี้ยรวม: %.2f บาท", interest))
print(String(format: "ยอดรวม: %.2f บาท", totalAmount))
```

### แบบฝึกหัดที่ 3: ตรวจสอบปีอธิกสุรทิน

```swift
// โจทย์: ตรวจสอบว่าปีที่กำหนดเป็นปีอธิกสุรทินหรือไม่
// ปีอธิกสุรทิน: หาร 4 ลงตัว แต่ถ้าหาร 100 ลงตัวต้องหาร 400 ลงตัวด้วย

func isLeapYear(_ year: Int) -> Bool {
    return (year % 4 == 0 && year % 100 != 0) || (year % 400 == 0)
}

let yearsToCheck = [2000, 1900, 2024, 2023, 1600]
for year in yearsToCheck {
    let status = isLeapYear(year) ? "ปีอธิกสุรทิน" : "ปีปกติ"
    print("\(year): \(status)")
}
```

### แบบฝึกหัดที่ 4: การแปลงอุณหภูมิ

```swift
// โจทย์: สร้าง struct Temperature ที่รองรับ operator overloading

struct Temperature {
    var celsius: Double
    
    var fahrenheit: Double { celsius * 9/5 + 32 }
    var kelvin: Double { celsius + 273.15 }
    
    init(celsius: Double) {
        self.celsius = celsius
    }
    
    static func fromFahrenheit(_ f: Double) -> Temperature {
        return Temperature(celsius: (f - 32) * 5/9)
    }
}

extension Temperature: CustomStringConvertible {
    var description: String {
        return String(format: "%.1f°C / %.1f°F / %.2f K", celsius, fahrenheit, kelvin)
    }
}

extension Temperature {
    static func + (lhs: Temperature, rhs: Temperature) -> Temperature {
        return Temperature(celsius: lhs.celsius + rhs.celsius)
    }
    
    static func - (lhs: Temperature, rhs: Temperature) -> Temperature {
        return Temperature(celsius: lhs.celsius - rhs.celsius)
    }
}

extension Temperature: Comparable {
    static func < (lhs: Temperature, rhs: Temperature) -> Bool {
        return lhs.celsius < rhs.celsius
    }
    
    static func == (lhs: Temperature, rhs: Temperature) -> Bool {
        return lhs.celsius == rhs.celsius
    }
}

let bodyTemp = Temperature(celsius: 37.0)
let roomTemp = Temperature(celsius: 22.0)
let boilingPoint = Temperature(celsius: 100.0)

print("อุณหภูมิร่างกาย: \(bodyTemp)")
print("อุณหภูมิห้อง: \(roomTemp)")
print("จุดเดือด: \(boilingPoint)")

print("ร่างกาย > ห้อง: \(bodyTemp > roomTemp)")

let tempFromFahrenheit = Temperature.fromFahrenheit(98.6)
print("98.6°F = \(tempFromFahrenheit)")
```

### แบบฝึกหัดที่ 5: ระบบบิต Permission

```swift
// โจทย์: สร้างระบบจัดการสิทธิ์ด้วย Bitwise Operations

struct FilePermission: OptionSet {
    let rawValue: Int
    
    static let read    = FilePermission(rawValue: 1 << 0)  // 1
    static let write   = FilePermission(rawValue: 1 << 1)  // 2
    static let execute = FilePermission(rawValue: 1 << 2)  // 4
    
    static let readOnly: FilePermission = [.read]
    static let readWrite: FilePermission = [.read, .write]
    static let all: FilePermission = [.read, .write, .execute]
}

var permissions = FilePermission.readOnly
print("สิทธิ์เริ่มต้น: อ่านได้ = \(permissions.contains(.read)), เขียนได้ = \(permissions.contains(.write))")

permissions.insert(.write)
print("หลังเพิ่มสิทธิ์เขียน: เขียนได้ = \(permissions.contains(.write))")

permissions.remove(.read)
print("หลังลบสิทธิ์อ่าน: อ่านได้ = \(permissions.contains(.read))")

// ตรวจสอบหลายสิทธิ์พร้อมกัน
let testPerm = FilePermission.all
print("All permissions: read=\(testPerm.contains(.read)), write=\(testPerm.contains(.write)), execute=\(testPerm.contains(.execute))")
```

---

## 3.16 กรณีการใช้งานจริง (Real-World Use Cases)

### กรณีที่ 1: ระบบตะกร้าสินค้า

```swift
struct CartItem {
    let name: String
    let price: Double
    let quantity: Int
    
    var subtotal: Double { price * Double(quantity) }
}

struct ShoppingCart {
    var items: [CartItem] = []
    let taxRate: Double = 0.07  // VAT 7%
    let discountThreshold: Double = 1000.0
    let discountRate: Double = 0.10
    
    var subtotal: Double {
        items.reduce(0) { $0 + $1.subtotal }
    }
    
    var discount: Double {
        subtotal >= discountThreshold ? subtotal * discountRate : 0
    }
    
    var tax: Double {
        (subtotal - discount) * taxRate
    }
    
    var total: Double {
        subtotal - discount + tax
    }
    
    func printReceipt() {
        print("=== ใบเสร็จ ===")
        for item in items {
            print(String(format: "%-15s %3d x %7.2f = %8.2f", 
                        (item.name as NSString).utf8String!, 
                        item.quantity, item.price, item.subtotal))
        }
        print(String(repeating: "-", count: 40))
        print(String(format: "ราคารวม:  %29.2f", subtotal))
        if discount > 0 {
            print(String(format: "ส่วนลด:  -%28.2f", discount))
        }
        print(String(format: "ภาษี:     %29.2f", tax))
        print(String(repeating: "=", count: 40))
        print(String(format: "รวมทั้งสิ้น: %26.2f", total))
    }
}

var cart = ShoppingCart()
cart.items = [
    CartItem(name: "เสื้อ", price: 299.0, quantity: 2),
    CartItem(name: "กางเกง", price: 499.0, quantity: 1),
    CartItem(name: "รองเท้า", price: 799.0, quantity: 1)
]

cart.printReceipt()
```

### กรณีที่ 2: การประมวลผลสีใน Graphic Design

```swift
struct Color {
    var red: UInt8
    var green: UInt8
    var blue: UInt8
    var alpha: UInt8 = 255
    
    // สร้างจาก hex string เช่น "#FF5733"
    init?(hex: String) {
        var hexValue = hex
        if hexValue.hasPrefix("#") {
            hexValue = String(hexValue.dropFirst())
        }
        
        guard hexValue.count == 6,
              let rgb = UInt32(hexValue, radix: 16) else {
            return nil
        }
        
        red   = UInt8((rgb >> 16) & 0xFF)
        green = UInt8((rgb >> 8) & 0xFF)
        blue  = UInt8(rgb & 0xFF)
    }
    
    init(red: UInt8, green: UInt8, blue: UInt8, alpha: UInt8 = 255) {
        self.red = red
        self.green = green
        self.blue = blue
        self.alpha = alpha
    }
    
    var hexString: String {
        return String(format: "#%02X%02X%02X", red, green, blue)
    }
    
    // ผสมสีสองสี
    func blend(with other: Color, ratio: Double = 0.5) -> Color {
        let r = Double(red) * (1 - ratio) + Double(other.red) * ratio
        let g = Double(green) * (1 - ratio) + Double(other.green) * ratio
        let b = Double(blue) * (1 - ratio) + Double(other.blue) * ratio
        return Color(red: UInt8(r), green: UInt8(g), blue: UInt8(b))
    }
    
    // ทำให้สว่างขึ้น
    func lighten(by factor: Double) -> Color {
        let r = min(255.0, Double(red) * (1 + factor))
        let g = min(255.0, Double(green) * (1 + factor))
        let b = min(255.0, Double(blue) * (1 + factor))
        return Color(red: UInt8(r), green: UInt8(g), blue: UInt8(b))
    }
}

if let orange = Color(hex: "#FF5733") {
    print("สีส้ม: R=\(orange.red), G=\(orange.green), B=\(orange.blue)")
    
    let blue = Color(red: 0, green: 0, blue: 255)
    let blended = orange.blend(with: blue, ratio: 0.3)
    print("สีผสม: \(blended.hexString)")
    
    let lighter = orange.lighten(by: 0.2)
    print("สีสว่างขึ้น: \(lighter.hexString)")
}
```

---

## สรุป (Summary)

ในบทนี้เราได้เรียนรู้ตัวดำเนินการต่าง ๆ ใน Swift:

| หมวดหมู่ | ตัวดำเนินการ | การใช้งาน |
|---|---|---|
| คณิตศาสตร์ | `+`, `-`, `*`, `/`, `%` | การคำนวณพื้นฐาน |
| กำหนดค่า | `=`, `+=`, `-=`, `*=`, `/=`, `%=` | กำหนดและอัปเดตค่า |
| เปรียบเทียบ | `==`, `!=`, `>`, `<`, `>=`, `<=` | เปรียบเทียบค่า |
| ตรรกะ | `&&`, `\|\|`, `!` | เงื่อนไข Boolean |
| บิต | `&`, `\|`, `^`, `~`, `<<`, `>>` | จัดการระดับ bit |
| ช่วง | `...`, `..<` | ระบุช่วงของค่า |
| Nil-Coalescing | `??` | ค่า default สำหรับ Optional |
| Ternary | `? :` | เงื่อนไขสั้น |
| อัตลักษณ์ | `===`, `!==` | เปรียบเทียบ reference |
| Overflow | `&+`, `&-`, `&*` | การคำนวณที่ wrap around |

### จุดสำคัญที่ต้องจำ

1. **การหาร Int** จะตัดทศนิยมทิ้ง ใช้ `Double` ถ้าต้องการทศนิยม
2. **Short-circuit evaluation** ใน `&&` และ `||` ช่วยประหยัด performance
3. **`??`** เป็นวิธีที่สวยงามในการจัดการ Optional
4. **`===`** ใช้เฉพาะกับ class (reference type) ไม่ใช่ struct
5. **Custom operators** ควรใช้เมื่อทำให้โค้ดอ่านง่ายขึ้นจริง ๆ
6. **Operator overloading** ทำให้ custom types ใช้งานได้เป็นธรรมชาติ

### คำถามทบทวน

1. ทำไม `7 / 2` ใน Swift ถึงให้ผลเป็น `3` ไม่ใช่ `3.5`?
2. อะไรคือความแตกต่างระหว่าง `==` และ `===`?
3. เมื่อใดควรใช้ `??` แทน `if let`?
4. `&&` และ `||` มีคุณสมบัติ short-circuit evaluation คืออะไร?
5. ทำไมการใช้ overflow operators ถึงสำคัญในบางกรณี?

---

*บทต่อไป: ส่วนที่ 4 - การควบคุมการไหลของโปรแกรม (Control Flow: if, switch)*
