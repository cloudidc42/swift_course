# ส่วนที่ 4: การควบคุมการไหลของโปรแกรม - If และ Switch (Control Flow: If and Switch)

## บทนำ (Introduction)

การควบคุมการไหลของโปรแกรม (Control Flow) คือความสามารถของโปรแกรมในการตัดสินใจว่าจะทำงานใดต่อไป โดยขึ้นอยู่กับเงื่อนไขที่กำหนด ใน Swift มีโครงสร้างหลักในการควบคุมการไหล ได้แก่:

- `if`, `if-else`, `if-else if-else` - ตรวจสอบเงื่อนไขและทำงานตามผลลัพธ์
- `guard` - ออกจากขอบเขตปัจจุบันเมื่อเงื่อนไขไม่เป็นจริง
- `switch` - เปรียบเทียบค่ากับหลายรูปแบบ
- Pattern Matching - จับคู่รูปแบบข้อมูล
- Labeled Statements - ควบคุมการวนซ้ำซ้อน

บทนี้จะเรียนรู้อย่างละเอียดพร้อมตัวอย่างการใช้งานจริง

---

## 4.1 คำสั่ง if พื้นฐาน (if Statement Basics)

### รูปแบบพื้นฐาน

```swift
// รูปแบบ:
// if เงื่อนไข {
//     โค้ดที่ทำงานเมื่อเงื่อนไขเป็นจริง
// }

let temperature = 30

if temperature > 25 {
    print("อากาศร้อน")
}

// เงื่อนไขต้องเป็น Bool เสมอ (ต่างจาก C/C++/JavaScript)
// ใน Swift นี้จะเกิดข้อผิดพลาด:
// if temperature {  // Error! ต้องเป็น Bool
//     print("ข้อผิดพลาด")
// }
```

### การใช้ if กับชนิดข้อมูลต่าง ๆ

```swift
// ตรวจสอบ Int
let score = 85
if score >= 60 {
    print("ผ่านการทดสอบ")
}

// ตรวจสอบ String
let name = "Swift"
if name.isEmpty {
    print("ไม่มีชื่อ")
}

if name.count > 3 {
    print("ชื่อยาว: \(name)")
}

// ตรวจสอบ Optional
let optionalValue: Int? = 42
if optionalValue != nil {
    print("มีค่า: \(optionalValue!)")
}

// ดีกว่า: ใช้ if let (Optional Binding)
if let value = optionalValue {
    print("มีค่า: \(value)")  // ไม่ต้องใช้ ! แล้ว
}
```

### ตัวอย่างจริง: ตรวจสอบอายุ

```swift
func checkAge(age: Int) {
    if age < 0 {
        print("อายุไม่ถูกต้อง")
    }
    
    if age >= 18 {
        print("บรรลุนิติภาวะ - สามารถทำนิติกรรมได้")
    }
    
    if age >= 60 {
        print("ผู้สูงอายุ - ได้รับสิทธิพิเศษ")
    }
}

checkAge(age: 25)
// บรรลุนิติภาวะ - สามารถทำนิติกรรมได้

checkAge(age: 65)
// บรรลุนิติภาวะ - สามารถทำนิติกรรมได้
// ผู้สูงอายุ - ได้รับสิทธิพิเศษ
```

---

## 4.2 คำสั่ง if-else

### รูปแบบ if-else

```swift
// รูปแบบ:
// if เงื่อนไข {
//     โค้ดเมื่อเป็นจริง
// } else {
//     โค้ดเมื่อเป็นเท็จ
// }

let isRaining = true

if isRaining {
    print("เอาร่มไปด้วย")
} else {
    print("ไม่ต้องเอาร่ม")
}

// ตัวอย่างกับตัวเลข
let number = 7
if number % 2 == 0 {
    print("\(number) เป็นเลขคู่")
} else {
    print("\(number) เป็นเลขคี่")
}
```

### if-else กับ Optional Binding

```swift
// การใช้ Optional Binding อย่างมีประสิทธิภาพ
func greet(name: String?) {
    if let actualName = name {
        print("สวัสดี, \(actualName)!")
    } else {
        print("สวัสดี, ผู้เยี่ยมชม!")
    }
}

greet(name: "Alice")   // สวัสดี, Alice!
greet(name: nil)        // สวัสดี, ผู้เยี่ยมชม!

// Swift 5.7+: ใช้ shorthand if let
func greet2(name: String?) {
    if let name {  // ชื่อเดิมโดยไม่ต้องพิมพ์ชื่อซ้ำ
        print("สวัสดี, \(name)!")
    } else {
        print("สวัสดี, ผู้เยี่ยมชม!")
    }
}
```

### ตัวอย่างจริง: ตรวจสอบสิทธิ์เข้าใช้งาน

```swift
struct User {
    let username: String
    let password: String
    var isActive: Bool
    var loginAttempts: Int = 0
}

func authenticate(user: User, inputPassword: String) -> String {
    if !user.isActive {
        return "บัญชีถูกระงับการใช้งาน"
    }
    
    if user.loginAttempts >= 3 {
        return "บัญชีถูกล็อกเนื่องจากล็อกอินผิดเกินกำหนด"
    }
    
    if user.password == inputPassword {
        return "ล็อกอินสำเร็จ! ยินดีต้อนรับ \(user.username)"
    } else {
        return "รหัสผ่านไม่ถูกต้อง (ครั้งที่ \(user.loginAttempts + 1))"
    }
}

let user1 = User(username: "alice", password: "P@ss123", isActive: true)
let user2 = User(username: "bob", password: "secret", isActive: false)

print(authenticate(user: user1, inputPassword: "P@ss123"))  // สำเร็จ
print(authenticate(user: user1, inputPassword: "wrongpass")) // ผิด
print(authenticate(user: user2, inputPassword: "secret"))    // ระงับ
```

---

## 4.3 คำสั่ง if-else if-else

### รูปแบบหลายเงื่อนไข

```swift
// รูปแบบ:
// if เงื่อนไข1 {
//     ...
// } else if เงื่อนไข2 {
//     ...
// } else if เงื่อนไข3 {
//     ...
// } else {
//     ...
// }

let score = 78

if score >= 90 {
    print("เกรด A")
} else if score >= 80 {
    print("เกรด B")
} else if score >= 70 {
    print("เกรด C")
} else if score >= 60 {
    print("เกรด D")
} else {
    print("เกรด F")
}
// เกรด C
```

### ข้อควรระวัง: ลำดับของเงื่อนไข

```swift
// ตัวอย่างที่ผิด: เงื่อนไขแรกจับทุกอย่าง
let temperature2 = 35
if temperature2 > 0 {  // จะเป็น true เสมอสำหรับค่าบวก
    print("ไม่หนาว")
} else if temperature2 > 30 {  // โค้ดนี้จะไม่มีวันทำงาน!
    print("ร้อนมาก")
}

// ตัวอย่างที่ถูกต้อง: เรียงจากเฉพาะเจาะจงไปกว้าง
if temperature2 > 35 {
    print("ร้อนมากผิดปกติ")
} else if temperature2 > 30 {
    print("ร้อน")
} else if temperature2 > 20 {
    print("อุ่น")
} else if temperature2 > 10 {
    print("เย็น")
} else {
    print("หนาว")
}
```

### ตัวอย่างจริง: เครื่องคิดค่าไฟ

```swift
func calculateElectricBill(units: Double) -> Double {
    // อัตราค่าไฟแบบขั้นบันได
    var bill = 0.0
    var remainingUnits = units
    
    if remainingUnits > 400 {
        bill += (remainingUnits - 400) * 4.18
        remainingUnits = 400
    }
    
    if remainingUnits > 150 {
        bill += (remainingUnits - 150) * 3.60
        remainingUnits = 150
    }
    
    if remainingUnits > 0 {
        bill += remainingUnits * 2.35
    }
    
    // ค่าบริการ
    let serviceCharge = 38.22
    return bill + serviceCharge
}

let usageUnits = [50.0, 150.0, 300.0, 500.0]
for units in usageUnits {
    let bill = calculateElectricBill(units: units)
    print(String(format: "ใช้ %.0f หน่วย = %.2f บาท", units, bill))
}
```

---

## 4.4 คำสั่ง if ซ้อนกัน (Nested if Statements)

### การซ้อน if

```swift
// if ซ้อนกัน
let isLoggedIn = true
let hasPermission = true
let isFileExists = false

if isLoggedIn {
    print("ผู้ใช้ล็อกอินแล้ว")
    
    if hasPermission {
        print("มีสิทธิ์เข้าถึง")
        
        if isFileExists {
            print("เปิดไฟล์สำเร็จ")
        } else {
            print("ไม่พบไฟล์ที่ต้องการ")
        }
    } else {
        print("ไม่มีสิทธิ์เข้าถึง")
    }
} else {
    print("กรุณาล็อกอินก่อน")
}
```

### หลีกเลี่ยง Pyramid of Doom

```swift
// ❌ แบบที่ไม่ดี: ซ้อนกันมากเกินไป (Pyramid of Doom)
func processOrderBad(userId: Int?, orderId: Int?, amount: Double?) {
    if let userId = userId {
        if let orderId = orderId {
            if let amount = amount {
                if amount > 0 {
                    if userId > 0 {
                        print("ประมวลผลออเดอร์ \(orderId) สำหรับผู้ใช้ \(userId) จำนวน \(amount) บาท")
                    }
                }
            }
        }
    }
}

// ✅ แบบที่ดี: ใช้ guard let และ comma ใน if let
func processOrderGood(userId: Int?, orderId: Int?, amount: Double?) {
    guard let userId = userId, userId > 0 else {
        print("userId ไม่ถูกต้อง")
        return
    }
    
    guard let orderId = orderId else {
        print("orderId ไม่ถูกต้อง")
        return
    }
    
    guard let amount = amount, amount > 0 else {
        print("จำนวนเงินไม่ถูกต้อง")
        return
    }
    
    print("ประมวลผลออเดอร์ \(orderId) สำหรับผู้ใช้ \(userId) จำนวน \(amount) บาท")
}

// หรือใช้ if let หลายค่าพร้อมกัน
func processOrderCompact(userId: Int?, orderId: Int?, amount: Double?) {
    if let userId = userId,
       let orderId = orderId,
       let amount = amount,
       userId > 0,
       amount > 0 {
        print("ประมวลผลออเดอร์ \(orderId) สำหรับผู้ใช้ \(userId) จำนวน \(amount) บาท")
    } else {
        print("ข้อมูลไม่ครบถ้วนหรือไม่ถูกต้อง")
    }
}

processOrderGood(userId: 1, orderId: 100, amount: 500.0)
processOrderCompact(userId: nil, orderId: 100, amount: 500.0)
```

---

## 4.5 คำสั่ง guard

`guard` ใช้เพื่อออกจากขอบเขตปัจจุบัน (function, loop, block) เมื่อเงื่อนไขไม่เป็นจริง

### รูปแบบพื้นฐาน

```swift
// รูปแบบ:
// guard เงื่อนไข else {
//     // โค้ดเมื่อเงื่อนไขไม่จริง
//     // ต้องออกจากขอบเขต: return, throw, break, continue
// }
// โค้ดที่ทำงานต่อเมื่อเงื่อนไขเป็นจริง

func processAge(age: Int) {
    guard age >= 0 else {
        print("อายุต้องไม่ติดลบ")
        return
    }
    
    guard age <= 150 else {
        print("อายุไม่สมเหตุสมผล")
        return
    }
    
    // ถ้าถึงบรรทัดนี้ age อยู่ในช่วง 0-150
    print("อายุที่ถูกต้อง: \(age)")
    
    if age >= 18 {
        print("บรรลุนิติภาวะ")
    }
}

processAge(age: -5)   // อายุต้องไม่ติดลบ
processAge(age: 200)  // อายุไม่สมเหตุสมผล
processAge(age: 25)   // อายุที่ถูกต้อง: 25, บรรลุนิติภาวะ
```

### guard let สำหรับ Optional

```swift
// guard let เผยแพร่ค่าออกไปยังขอบเขตภายนอก (ต่างจาก if let)
func getUserInfo(from data: [String: Any]) {
    guard let name = data["name"] as? String else {
        print("ไม่มีชื่อผู้ใช้")
        return
    }
    
    guard let age = data["age"] as? Int else {
        print("ไม่มีข้อมูลอายุ")
        return
    }
    
    guard let email = data["email"] as? String, email.contains("@") else {
        print("อีเมลไม่ถูกต้อง")
        return
    }
    
    // name, age, email ใช้งานได้ที่นี่
    print("ชื่อ: \(name), อายุ: \(age), อีเมล: \(email)")
}

getUserInfo(from: ["name": "Alice", "age": 25, "email": "alice@example.com"])
getUserInfo(from: ["name": "Bob", "age": 30])
getUserInfo(from: [:])
```

### guard vs if: เมื่อใดใช้อะไร

```swift
// if let: เมื่อต้องการใช้ค่าเฉพาะในบล็อก if
func exampleIfLet(value: String?) {
    if let v = value {
        print("ค่าใน if: \(v)")
    }
    // v ใช้งานไม่ได้ที่นี่แล้ว
    print("ทำงานต่อ...")
}

// guard let: เมื่อต้องการออกเร็วถ้าไม่มีค่า
func exampleGuardLet(value: String?) {
    guard let v = value else {
        print("ไม่มีค่า ออกจากฟังก์ชัน")
        return
    }
    // v ใช้งานได้ตลอดส่วนที่เหลือ
    print("ค่าหลัง guard: \(v)")
    print("ยังคงใช้ค่าได้: \(v.uppercased())")
}

exampleIfLet(value: "hello")
exampleIfLet(value: nil)
exampleGuardLet(value: "world")
exampleGuardLet(value: nil)
```

---

## 4.6 รูปแบบ Guard-Else

### Early Exit Pattern

```swift
// รูปแบบการออกเร็ว (Early Exit) ด้วย guard
func transferMoney(from sender: String, to receiver: String, amount: Double) -> String {
    
    // ตรวจสอบผู้ส่ง
    guard !sender.isEmpty else {
        return "ข้อผิดพลาด: ไม่ระบุผู้ส่ง"
    }
    
    // ตรวจสอบผู้รับ
    guard !receiver.isEmpty else {
        return "ข้อผิดพลาด: ไม่ระบุผู้รับ"
    }
    
    // ตรวจสอบไม่ส่งให้ตัวเอง
    guard sender != receiver else {
        return "ข้อผิดพลาด: ไม่สามารถโอนให้ตัวเองได้"
    }
    
    // ตรวจสอบจำนวนเงิน
    guard amount > 0 else {
        return "ข้อผิดพลาด: จำนวนเงินต้องมากกว่า 0"
    }
    
    guard amount <= 1_000_000 else {
        return "ข้อผิดพลาด: จำนวนเงินเกินขีดจำกัด"
    }
    
    // ถ้าผ่านทุก guard แล้ว
    return "โอนเงิน \(amount) บาท จาก \(sender) ไป \(receiver) สำเร็จ"
}

print(transferMoney(from: "Alice", to: "Bob", amount: 500.0))
print(transferMoney(from: "", to: "Bob", amount: 500.0))
print(transferMoney(from: "Alice", to: "Alice", amount: 500.0))
print(transferMoney(from: "Alice", to: "Bob", amount: -100.0))
```

### guard ใน loop

```swift
let data = ["Alice:25", "Bob:invalid", ":30", "Charlie:28", ""]

for item in data {
    // ข้ามรายการที่ว่างเปล่า
    guard !item.isEmpty else {
        print("ข้ามรายการว่างเปล่า")
        continue  // ใน loop ใช้ continue แทน return
    }
    
    let parts = item.split(separator: ":")
    guard parts.count == 2 else {
        print("รูปแบบไม่ถูกต้อง: \(item)")
        continue
    }
    
    let name = String(parts[0])
    guard !name.isEmpty else {
        print("ชื่อว่างเปล่า")
        continue
    }
    
    guard let age = Int(parts[1]) else {
        print("อายุไม่ใช่ตัวเลข: \(parts[1])")
        continue
    }
    
    print("ชื่อ: \(name), อายุ: \(age)")
}
```

---

## 4.7 คำสั่ง switch พื้นฐาน (Switch Statement Basics)

`switch` ใน Swift มีความสามารถมากกว่า C/Java มาก ไม่จำเป็นต้องมี `break` และรองรับ Pattern Matching หลากหลายรูปแบบ

### รูปแบบพื้นฐาน

```swift
// ทุก case ต้อง exhaustive (ครอบคลุมทุกค่า)
let day = 3

switch day {
case 1:
    print("วันจันทร์")
case 2:
    print("วันอังคาร")
case 3:
    print("วันพุธ")
case 4:
    print("วันพฤหัสบดี")
case 5:
    print("วันศุกร์")
case 6:
    print("วันเสาร์")
case 7:
    print("วันอาทิตย์")
default:
    print("ไม่ใช่วันในสัปดาห์")
}
```

### การรวม Case

```swift
let day2 = 6

switch day2 {
case 1, 2, 3, 4, 5:
    print("วันทำงาน (จันทร์-ศุกร์)")
case 6, 7:
    print("วันหยุดสุดสัปดาห์")
default:
    print("ค่าไม่ถูกต้อง")
}
```

### switch กับ String

```swift
let season = "ฤดูฝน"

switch season {
case "ฤดูร้อน":
    print("อากาศร้อน ควรดื่มน้ำมาก ๆ")
case "ฤดูฝน":
    print("อย่าลืมพกร่ม")
case "ฤดูหนาว":
    print("ใส่เสื้อหนาว ๆ")
default:
    print("ฤดูกาลที่ไม่รู้จัก")
}
```

### switch ต้อง Exhaustive

```swift
enum Direction {
    case north, south, east, west
}

let direction = Direction.north

switch direction {
case .north:
    print("ไปทางเหนือ")
case .south:
    print("ไปทางใต้")
case .east:
    print("ไปทางตะวันออก")
case .west:
    print("ไปทางตะวันตก")
// ไม่ต้องมี default เพราะครอบคลุมทุก case แล้ว
}
```

---

## 4.8 Switch กับ Ranges

```swift
// ใช้ช่วงค่า (Range) ใน switch
let score2 = 78

switch score2 {
case 90...100:
    print("A - ยอดเยี่ยม")
case 80..<90:
    print("B - ดีมาก")
case 70..<80:
    print("C - ดี")
case 60..<70:
    print("D - พอใช้")
case 0..<60:
    print("F - ไม่ผ่าน")
default:
    print("คะแนนไม่ถูกต้อง")
}

// ตัวอย่างกับ Double
let bmi = 22.5
let bmiCategory: String

switch bmi {
case ..<18.5:
    bmiCategory = "น้ำหนักน้อย"
case 18.5..<25:
    bmiCategory = "น้ำหนักปกติ"
case 25..<30:
    bmiCategory = "น้ำหนักเกิน"
case 30...:
    bmiCategory = "โรคอ้วน"
default:
    bmiCategory = "ไม่ทราบ"
}
print("BMI \(bmi): \(bmiCategory)")
```

### ตัวอย่างจริง: ระบบจัดส่งสินค้า

```swift
func shippingCost(weight: Double) -> Double {
    switch weight {
    case 0..<0.5:
        return 25.0  // บาท
    case 0.5..<1:
        return 35.0
    case 1..<3:
        return 50.0
    case 3..<5:
        return 75.0
    case 5..<10:
        return 100.0
    case 10...:
        return 150.0 + (weight - 10) * 10.0
    default:
        return 0.0  // น้ำหนักไม่ถูกต้อง
    }
}

let weights = [0.3, 0.8, 2.5, 7.0, 15.0]
for w in weights {
    print(String(format: "น้ำหนัก %.1f กก. = %.2f บาท", w, shippingCost(weight: w)))
}
```

---

## 4.9 Switch กับ Tuple

```swift
// switch กับ tuple
let coordinates = (0, 1)

switch coordinates {
case (0, 0):
    print("จุดกำเนิด (Origin)")
case (_, 0):
    print("อยู่บนแกน X")
case (0, _):
    print("อยู่บนแกน Y")
case (-5...5, -5...5):
    print("อยู่ในพื้นที่ใกล้กับจุดกำเนิด")
default:
    print("อยู่ที่ (\(coordinates.0), \(coordinates.1))")
}
```

### ตัวอย่างจริง: ระบบไฟจราจร

```swift
enum TrafficLight {
    case red, yellow, green
}

enum TimeOfDay {
    case day, night
}

func trafficInstruction(light: TrafficLight, time: TimeOfDay) -> String {
    switch (light, time) {
    case (.red, _):
        return "หยุด!"
    case (.yellow, .day):
        return "ระวัง เตรียมหยุด"
    case (.yellow, .night):
        return "ระวังเป็นพิเศษ (กลางคืน)"
    case (.green, .day):
        return "ไปได้"
    case (.green, .night):
        return "ไปได้ แต่ระวัง"
    }
}

print(trafficInstruction(light: .red, time: .night))     // หยุด!
print(trafficInstruction(light: .green, time: .day))     // ไปได้
print(trafficInstruction(light: .yellow, time: .night))  // ระวังเป็นพิเศษ
```

---

## 4.10 Switch กับ Value Binding

```swift
// Value Binding: กำหนดชื่อให้ค่าที่จับได้
let point = (3, -2)

switch point {
case (let x, 0):
    print("อยู่บนแกน X ที่ตำแหน่ง \(x)")
case (0, let y):
    print("อยู่บนแกน Y ที่ตำแหน่ง \(y)")
case let (x, y):
    print("อยู่ที่ (\(x), \(y))")
}

// Value Binding กับ enum ที่มี associated values
enum Shape {
    case circle(radius: Double)
    case rectangle(width: Double, height: Double)
    case triangle(base: Double, height: Double)
}

func calculateArea(shape: Shape) -> Double {
    switch shape {
    case .circle(let radius):
        return Double.pi * radius * radius
    case .rectangle(let width, let height):
        return width * height
    case .triangle(let base, let height):
        return 0.5 * base * height
    }
}

let shapes: [Shape] = [
    .circle(radius: 5),
    .rectangle(width: 4, height: 6),
    .triangle(base: 8, height: 3)
]

for shape in shapes {
    print(String(format: "พื้นที่: %.2f", calculateArea(shape: shape)))
}
```

---

## 4.11 Switch Fallthrough

ปกติ Swift จะไม่ fallthrough แต่สามารถใช้ `fallthrough` keyword ได้ถ้าต้องการ

```swift
// ปกติ: ไม่ fallthrough
let number = 3
switch number {
case 3:
    print("สาม")  // จะหยุดที่นี่ ไม่ไป case 4
case 4:
    print("สี่")
default:
    break
}
// Output: สาม

// ใช้ fallthrough เมื่อต้องการ
let number2 = 3
switch number2 {
case 3:
    print("สาม")
    fallthrough  // ไปทำ case ถัดไป
case 4:
    print("สี่หรือมากกว่า")
case 5:
    print("ห้า")
default:
    break
}
// Output: สาม
//         สี่หรือมากกว่า
```

### ตัวอย่างการใช้ fallthrough จริง ๆ

```swift
// Fallthrough ในกรณีที่ต้องการสะสม actions
func describeNumber(_ n: Int) {
    print("จำนวน \(n):")
    
    switch n {
    case 1...3:
        print("- เป็นจำนวนน้อย")
        fallthrough
    case 1...10:
        print("- เป็นหลักหน่วย")
        fallthrough
    case 1...100:
        print("- น้อยกว่า 101")
    default:
        print("- เป็นจำนวนใหญ่")
    }
}

describeNumber(2)
// จำนวน 2:
// - เป็นจำนวนน้อย
// - เป็นหลักหน่วย
// - น้อยกว่า 101

describeNumber(7)
// จำนวน 7:
// - เป็นหลักหน่วย
// - น้อยกว่า 101
```

---

## 4.12 Where Clause ใน Switch

`where` ใช้เพิ่มเงื่อนไขพิเศษให้กับ case

```swift
// where clause ใน switch
let point2 = (1, -1)

switch point2 {
case let (x, y) where x == y:
    print("(\(x), \(y)) อยู่บนเส้น y = x")
case let (x, y) where x == -y:
    print("(\(x), \(y)) อยู่บนเส้น y = -x")
case let (x, y) where x > 0 && y > 0:
    print("(\(x), \(y)) อยู่ใน Quadrant I")
case let (x, y) where x < 0 && y > 0:
    print("(\(x), \(y)) อยู่ใน Quadrant II")
case let (x, y) where x < 0 && y < 0:
    print("(\(x), \(y)) อยู่ใน Quadrant III")
case let (x, y) where x > 0 && y < 0:
    print("(\(x), \(y)) อยู่ใน Quadrant IV")
default:
    print("อยู่บนแกน")
}

// where กับ for loop
let numbers = [1, -2, 3, -4, 5, -6, 7, -8]
for number in numbers where number > 0 {
    print("จำนวนบวก: \(number)")
}
```

### ตัวอย่างจริง: การประมวลผลข้อมูลนักเรียน

```swift
struct Student {
    let name: String
    let score: Int
    let attendance: Double  // เปอร์เซ็นต์การเข้าเรียน
}

func evaluateStudent(_ student: Student) -> String {
    switch student.score {
    case 0..<60 where student.attendance < 0.75:
        return "\(student.name): ไม่ผ่าน (คะแนนต่ำและเข้าเรียนน้อย)"
    case 0..<60:
        return "\(student.name): ไม่ผ่าน (คะแนนต่ำ)"
    case 60..<80 where student.attendance < 0.75:
        return "\(student.name): เสี่ยงไม่ผ่าน (เข้าเรียนน้อย)"
    case 60..<80:
        return "\(student.name): ผ่าน ระดับปานกลาง"
    case 80...100 where student.attendance >= 0.9:
        return "\(student.name): ยอดเยี่ยม! (คะแนนดีและเข้าเรียนสม่ำเสมอ)"
    case 80...100:
        return "\(student.name): ผ่าน ระดับดี"
    default:
        return "\(student.name): ข้อมูลไม่ถูกต้อง"
    }
}

let students = [
    Student(name: "Alice", score: 92, attendance: 0.95),
    Student(name: "Bob", score: 75, attendance: 0.68),
    Student(name: "Charlie", score: 45, attendance: 0.5),
    Student(name: "Diana", score: 88, attendance: 0.82)
]

for student in students {
    print(evaluateStudent(student))
}
```

---

## 4.13 Pattern Matching พื้นฐาน

### ~= Operator

```swift
// Pattern Matching ใช้ ~= operator
let value = 7
let isInRange = 1...10 ~= value
print("7 อยู่ใน 1...10: \(isInRange)")  // true

// Custom pattern matching
struct Category {
    let minAge: Int
    let maxAge: Int
    let name: String
}

// ใช้ ~= สำหรับ custom type
extension Category {
    static func ~= (pattern: Category, value: Int) -> Bool {
        return value >= pattern.minAge && value <= pattern.maxAge
    }
}

let child = Category(minAge: 0, maxAge: 12, name: "เด็ก")
let teen = Category(minAge: 13, maxAge: 17, name: "วัยรุ่น")
let adult = Category(minAge: 18, maxAge: 59, name: "ผู้ใหญ่")
let senior = Category(minAge: 60, maxAge: 120, name: "ผู้สูงอายุ")

let personAge = 15
switch personAge {
case child:
    print("เป็น\(child.name)")
case teen:
    print("เป็น\(teen.name)")
case adult:
    print("เป็น\(adult.name)")
case senior:
    print("เป็น\(senior.name)")
default:
    print("ไม่ทราบหมวดหมู่")
}
```

### Pattern Matching กับ Optional

```swift
// Optional pattern
let scores: [Int?] = [85, nil, 92, nil, 78]

for case let score? in scores {  // จับเฉพาะค่าที่ไม่ใช่ nil
    print("คะแนน: \(score)")
}

// ตรวจสอบ nil
for score in scores {
    switch score {
    case .none:
        print("ไม่มีคะแนน")
    case .some(let s) where s >= 80:
        print("คะแนนดี: \(s)")
    case .some(let s):
        print("คะแนน: \(s)")
    }
}
```

---

## 4.14 @unknown default

ใช้เมื่อ enum อาจมี case ใหม่ในอนาคต (สำหรับ enum จากไลบรารีภายนอก)

```swift
// สมมติว่า enum นี้อาจมี case ใหม่ในอนาคต
enum NetworkStatus {
    case connected
    case disconnected
    case connecting
    // อาจมี case ใหม่เพิ่มในอนาคต
}

let status = NetworkStatus.connected

// @unknown default จะเตือน compiler warning
// เมื่อ enum มี case ใหม่ที่ยังไม่ได้จัดการ
switch status {
case .connected:
    print("เชื่อมต่อแล้ว")
case .disconnected:
    print("ไม่ได้เชื่อมต่อ")
case .connecting:
    print("กำลังเชื่อมต่อ...")
@unknown default:
    print("สถานะไม่ทราบ (อาจเป็น case ใหม่)")
}
```

### เหตุผลที่ใช้ @unknown default

```swift
// ตัวอย่างเชิงปฏิบัติ: จัดการ API response status
// สมมติว่า OrderStatus มาจาก external library
enum OrderStatus: Int {
    case pending = 1
    case processing = 2
    case shipped = 3
    case delivered = 4
    case cancelled = 5
    // อาจมี case ใหม่ในอนาคต เช่น refunded, disputed
}

func handleOrderStatus(_ status: OrderStatus) -> String {
    switch status {
    case .pending:
        return "รอดำเนินการ"
    case .processing:
        return "กำลังเตรียมสินค้า"
    case .shipped:
        return "จัดส่งแล้ว"
    case .delivered:
        return "ส่งถึงผู้รับแล้ว"
    case .cancelled:
        return "ยกเลิกแล้ว"
    @unknown default:
        return "สถานะใหม่ที่ยังไม่รองรับ"
    }
}

print(handleOrderStatus(.processing))
```

---

## 4.15 Labeled Statements

Labeled statements ช่วยควบคุมการวนซ้ำที่ซ้อนกัน

### การใช้งาน Label

```swift
// ค้นหาใน matrix 2 มิติ
let matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]
let target = 5
var found = false

// หา target ใน matrix
outerLoop: for row in matrix {
    for element in row {
        if element == target {
            print("พบ \(target)!")
            found = true
            break outerLoop  // ออกจาก loop นอกด้วย
        }
    }
}

if !found {
    print("ไม่พบ \(target)")
}
```

### continue กับ Label

```swift
// ข้ามการวนซ้ำใน outer loop
let numbers2 = [1, 2, 3, 4, 5]
let divisors = [2, 3]

outerLoop2: for number in numbers2 {
    for divisor in divisors {
        if number % divisor == 0 && divisor != number {
            print("\(number) หารด้วย \(divisor) ลงตัว - ข้ามไปตัวถัดไป")
            continue outerLoop2  // ข้ามไป number ถัดไป
        }
    }
    print("\(number) ผ่านการตรวจสอบทั้งหมด")
}
```

### ตัวอย่างจริง: ตรวจสอบตาราง

```swift
// ตรวจสอบว่ามีช่องว่างใน schedule หรือไม่
let schedule = [
    ["08:00", "09:00", "10:00"],  // วันจันทร์
    ["", "09:00", "10:00"],       // วันอังคาร (ไม่มี 08:00)
    ["08:00", "", "10:00"],       // วันพุธ (ไม่มี 09:00)
]

let days = ["จันทร์", "อังคาร", "พุธ"]
let times = ["08:00", "09:00", "10:00"]
var hasEmpty = false

scheduleCheck: for (dayIndex, daySchedule) in schedule.enumerated() {
    for (timeIndex, slot) in daySchedule.enumerated() {
        if slot.isEmpty {
            print("พบช่องว่าง: วัน\(days[dayIndex]) เวลา \(times[timeIndex])")
            hasEmpty = true
            // ไม่ break ออก - ต้องการหาช่องว่างทั้งหมด
        }
    }
}

print(hasEmpty ? "มีช่องว่างในตาราง" : "ตารางสมบูรณ์")
```

---

## 4.16 รูปแบบ Early Exit

### Early Exit Pattern ที่ดี

```swift
// ❌ แบบที่ไม่ดี: ซ้อนกันลึก
func processDataBad(data: Data?) -> String {
    if let data = data {
        if data.count > 0 {
            if let text = String(data: data, encoding: .utf8) {
                if !text.isEmpty {
                    return "ประมวลผล: \(text)"
                } else {
                    return "ข้อความว่างเปล่า"
                }
            } else {
                return "ไม่สามารถแปลงเป็น UTF-8"
            }
        } else {
            return "ข้อมูลว่างเปล่า"
        }
    } else {
        return "ไม่มีข้อมูล"
    }
}

// ✅ แบบที่ดี: Early Exit
func processDataGood(data: Data?) -> String {
    guard let data = data else { return "ไม่มีข้อมูล" }
    guard data.count > 0 else { return "ข้อมูลว่างเปล่า" }
    guard let text = String(data: data, encoding: .utf8) else { 
        return "ไม่สามารถแปลงเป็น UTF-8" 
    }
    guard !text.isEmpty else { return "ข้อความว่างเปล่า" }
    
    return "ประมวลผล: \(text)"
}
```

### ตัวอย่างจริง: Parsing JSON อย่างปลอดภัย

```swift
import Foundation

func parseUserJSON(_ jsonString: String) -> (id: Int, name: String, email: String)? {
    // แปลง string เป็น Data
    guard let data = jsonString.data(using: .utf8) else {
        print("ไม่สามารถแปลง string เป็น data")
        return nil
    }
    
    // Parse JSON
    guard let json = try? JSONSerialization.jsonObject(with: data) as? [String: Any] else {
        print("ไม่สามารถ parse JSON")
        return nil
    }
    
    // ดึงค่าแต่ละฟิลด์
    guard let id = json["id"] as? Int else {
        print("ไม่มีหรือ id ไม่ใช่ Int")
        return nil
    }
    
    guard let name = json["name"] as? String, !name.isEmpty else {
        print("ไม่มีชื่อหรือชื่อว่างเปล่า")
        return nil
    }
    
    guard let email = json["email"] as? String, email.contains("@") else {
        print("อีเมลไม่ถูกต้อง")
        return nil
    }
    
    return (id, name, email)
}

let validJSON = """
{"id": 1, "name": "Alice", "email": "alice@example.com"}
"""

let invalidJSON = """
{"id": 2, "name": "", "email": "notanemail"}
"""

if let user = parseUserJSON(validJSON) {
    print("ผู้ใช้: #\(user.id) \(user.name) (\(user.email))")
}

if parseUserJSON(invalidJSON) == nil {
    print("ข้อมูลไม่ผ่านการตรวจสอบ")
}
```

---

## 4.17 Conditional Compilation (#if, #else, #endif)

### การใช้งาน Compilation Directives

```swift
// ตรวจสอบ platform
#if os(iOS)
    print("กำลังทำงานบน iOS")
#elseif os(macOS)
    print("กำลังทำงานบน macOS")
#elseif os(watchOS)
    print("กำลังทำงานบน watchOS")
#elseif os(tvOS)
    print("กำลังทำงานบน tvOS")
#else
    print("กำลังทำงานบน platform อื่น")
#endif
```

### DEBUG vs Release

```swift
// ตรวจสอบ build configuration
#if DEBUG
    print("โหมด Debug - แสดง log ทั้งหมด")
    let logLevel = "verbose"
#else
    let logLevel = "error"
#endif

// ใช้ canImport ตรวจสอบว่า import ได้หรือไม่
#if canImport(UIKit)
    import UIKit
    let screenWidth = UIScreen.main.bounds.width
#elseif canImport(AppKit)
    import AppKit
    let screenWidth = NSScreen.main?.frame.width ?? 0
#endif
```

### Custom Compilation Flags

```swift
// กำหนด flag ใน Build Settings: OTHER_SWIFT_FLAGS = -D FEATURE_ENABLED

// ใช้ในโค้ด:
#if FEATURE_ENABLED
    print("ฟีเจอร์พิเศษเปิดใช้งาน")
    // แสดง UI พิเศษ
#else
    print("ฟีเจอร์พิเศษปิดใช้งาน")
#endif

// ตรวจสอบ Swift version
#if swift(>=5.7)
    print("Swift 5.7 ขึ้นไป - รองรับ shorthand if let")
#elseif swift(>=5.5)
    print("Swift 5.5 ขึ้นไป - รองรับ async/await")
#else
    print("Swift เวอร์ชันเก่า")
#endif
```

---

## 4.18 Best Practices

### 1. ใช้ guard สำหรับ Early Exit

```swift
// ✅ ดี: ใช้ guard ทำให้อ่านง่าย
func calculateDiscount(price: Double?, percentage: Double?) -> Double {
    guard let price = price, price > 0 else { return 0 }
    guard let percentage = percentage, 0...100 ~= percentage else { return 0 }
    
    return price * (percentage / 100)
}

// ❌ ไม่ดี: ซ้อนกันมาก
func calculateDiscountBad(price: Double?, percentage: Double?) -> Double {
    if let price = price {
        if price > 0 {
            if let percentage = percentage {
                if percentage >= 0 && percentage <= 100 {
                    return price * (percentage / 100)
                }
            }
        }
    }
    return 0
}
```

### 2. ใช้ switch แทน if-else if ที่ยาว

```swift
// ✅ ดี: ใช้ switch ชัดเจนกว่า
func classify(value: Int) -> String {
    switch value {
    case ..<0:        return "ลบ"
    case 0:           return "ศูนย์"
    case 1...9:       return "หลักหน่วย"
    case 10...99:     return "หลักสิบ"
    case 100...999:   return "หลักร้อย"
    default:          return "มากกว่าร้อย"
    }
}

// ❌ ไม่ดี: if-else ยาวเกินไป
func classifyBad(value: Int) -> String {
    if value < 0 {
        return "ลบ"
    } else if value == 0 {
        return "ศูนย์"
    } else if value >= 1 && value <= 9 {
        return "หลักหน่วย"
    } else if value >= 10 && value <= 99 {
        return "หลักสิบ"
    } else if value >= 100 && value <= 999 {
        return "หลักร้อย"
    } else {
        return "มากกว่าร้อย"
    }
}
```

### 3. ตั้งชื่อเงื่อนไขให้สื่อความหมาย

```swift
// ✅ ดี: ตั้งชื่อเงื่อนไขที่ซับซ้อน
let user_age = 20
let hasParentalConsent = false
let isStudentID = true

let canRegister = user_age >= 18 || (user_age >= 13 && hasParentalConsent)
let hasValidID = isStudentID || user_age >= 18

if canRegister && hasValidID {
    print("สามารถลงทะเบียนได้")
}

// ❌ ไม่ดี: เงื่อนไขยาวและอ่านยาก
if (user_age >= 18 || (user_age >= 13 && hasParentalConsent)) && (isStudentID || user_age >= 18) {
    print("สามารถลงทะเบียนได้")
}
```

### 4. หลีกเลี่ยง fallthrough ที่ไม่จำเป็น

```swift
// ✅ ดี: ใช้ comma แทน fallthrough
let day3 = "เสาร์"
switch day3 {
case "เสาร์", "อาทิตย์":
    print("วันหยุดสุดสัปดาห์")
default:
    print("วันทำงาน")
}

// ❌ ไม่ดี: ใช้ fallthrough ที่ไม่ชัดเจน
switch day3 {
case "เสาร์":
    fallthrough
case "อาทิตย์":
    print("วันหยุดสุดสัปดาห์")
default:
    print("วันทำงาน")
}
```

---

## 4.19 ข้อผิดพลาดที่พบบ่อย (Common Mistakes)

### 1. ลืม break ใน switch (ใน Swift ไม่ต้องใส่)

```swift
// Swift ไม่ต้องมี break (ต่างจาก C/Java)
let x = 2
switch x {
case 1:
    print("หนึ่ง")  // จะหยุดที่นี่โดยอัตโนมัติ
    // ไม่ต้องมี break!
case 2:
    print("สอง")
case 3:
    print("สาม")
default:
    break  // ใช้ break เมื่อไม่ต้องทำอะไรใน default
}
```

### 2. ใช้ = แทน == ใน เงื่อนไข

```swift
var count = 5

// ✅ ถูกต้อง
if count == 5 {
    print("count เท่ากับ 5")
}

// ❌ ใน Swift นี้จะ error (ต่างจาก C ที่อนุญาต)
// if count = 5 {  // Error: = ไม่คืนค่า Bool
// }
```

### 3. guard ต้อง transfer control

```swift
func example(value: Int?) {
    // ✅ ถูกต้อง: guard ต้องมี return, throw, break, หรือ continue
    guard let v = value else {
        return  // บังคับต้องออกจาก scope
    }
    print(v)
    
    // ❌ ผิด: guard ที่ไม่ออกจาก scope
    // guard let v2 = value else {
    //     print("no value")  // Error! ต้องมี return/throw/break/continue
    // }
}
```

### 4. Switch ต้อง Exhaustive

```swift
enum Status {
    case active, inactive, pending
}

let status2 = Status.active

// ✅ ถูกต้อง: ครอบคลุมทุก case
switch status2 {
case .active:
    print("ใช้งาน")
case .inactive:
    print("ไม่ใช้งาน")
case .pending:
    print("รอดำเนินการ")
}

// ❌ ผิด: ไม่ครอบคลุมทุก case (ถ้าไม่มี default)
// switch status2 {
// case .active:
//     print("ใช้งาน")
// // Error! ขาด inactive และ pending
// }
```

### 5. Optional ที่ไม่ได้ Unwrap

```swift
let name2: String? = "Alice"

// ❌ ไม่ดี: Force unwrap อาจ crash
// print(name2!)  // crash ถ้า nil

// ✅ ดี: Optional Binding
if let n = name2 {
    print(n)
}

// ✅ ดี: nil coalescing
print(name2 ?? "ไม่มีชื่อ")

// ✅ ดี: guard let
func printName(_ name: String?) {
    guard let name = name else {
        print("ไม่มีชื่อ")
        return
    }
    print(name)
}
```

---

## 4.20 แบบฝึกหัดพร้อมเฉลย (Practical Exercises with Solutions)

### แบบฝึกหัดที่ 1: เครื่องคิดเกรด

```swift
// โจทย์: สร้างระบบคำนวณเกรดแบบ GPA
// A=4.0, B+=3.5, B=3.0, C+=2.5, C=2.0, D+=1.5, D=1.0, F=0

struct CourseGrade {
    let courseName: String
    let score: Double
    let credits: Int
}

func letterGrade(score: Double) -> String {
    switch score {
    case 80...100: return "A"
    case 75..<80:  return "B+"
    case 70..<75:  return "B"
    case 65..<70:  return "C+"
    case 60..<65:  return "C"
    case 55..<60:  return "D+"
    case 50..<55:  return "D"
    default:       return "F"
    }
}

func gradePoint(letter: String) -> Double {
    switch letter {
    case "A":  return 4.0
    case "B+": return 3.5
    case "B":  return 3.0
    case "C+": return 2.5
    case "C":  return 2.0
    case "D+": return 1.5
    case "D":  return 1.0
    default:   return 0.0
    }
}

func calculateGPA(courses: [CourseGrade]) -> Double {
    guard !courses.isEmpty else { return 0.0 }
    
    var totalWeightedPoints = 0.0
    var totalCredits = 0
    
    for course in courses {
        let letter = letterGrade(score: course.score)
        let point = gradePoint(letter: letter)
        totalWeightedPoints += point * Double(course.credits)
        totalCredits += course.credits
    }
    
    guard totalCredits > 0 else { return 0.0 }
    return totalWeightedPoints / Double(totalCredits)
}

let myCourses = [
    CourseGrade(courseName: "คณิตศาสตร์", score: 85, credits: 3),
    CourseGrade(courseName: "ฟิสิกส์", score: 72, credits: 3),
    CourseGrade(courseName: "ภาษาอังกฤษ", score: 91, credits: 2),
    CourseGrade(courseName: "โปรแกรมมิ่ง", score: 78, credits: 3),
    CourseGrade(courseName: "ประวัติศาสตร์", score: 63, credits: 2)
]

print("=== ผลการเรียน ===")
for course in myCourses {
    let grade = letterGrade(score: course.score)
    print(String(format: "%-20s %.1f -> %s", 
                (course.courseName as NSString).utf8String!, 
                course.score, grade))
}

let gpa = calculateGPA(courses: myCourses)
print(String(format: "\nGPA: %.2f", gpa))

switch gpa {
case 3.5...4.0:
    print("เกียรตินิยมอันดับ 1")
case 3.25..<3.5:
    print("เกียรตินิยมอันดับ 2")
case 2.0..<3.25:
    print("ผ่านการศึกษา")
default:
    print("ผลการเรียนต่ำกว่าเกณฑ์")
}
```

### แบบฝึกหัดที่ 2: ระบบไฟจราจร (Traffic Light)

```swift
// โจทย์: สร้างระบบจำลองไฟจราจร
enum TrafficSignal {
    case red(duration: Int)
    case yellow(duration: Int)
    case green(duration: Int)
    case blinking(color: String)
    
    var instruction: String {
        switch self {
        case .red:
            return "หยุดรถ!"
        case .yellow:
            return "ระวัง! เตรียมหยุด"
        case .green:
            return "ไปได้"
        case .blinking(let color) where color == "red":
            return "หยุดแล้วไป (ทางหลัก)"
        case .blinking(let color) where color == "yellow":
            return "ระวังเป็นพิเศษ"
        case .blinking:
            return "ระวัง"
        }
    }
    
    var duration: Int {
        switch self {
        case .red(let d), .yellow(let d), .green(let d):
            return d
        case .blinking:
            return 0  // ไม่จำกัดเวลา
        }
    }
}

func simulateTraffic(signals: [TrafficSignal]) {
    print("=== จำลองการจราจร ===")
    for signal in signals {
        print("สัญญาณ: \(signal.instruction)", terminator: "")
        if signal.duration > 0 {
            print(" (เป็นเวลา \(signal.duration) วินาที)")
        } else {
            print()
        }
    }
}

let trafficSequence: [TrafficSignal] = [
    .red(duration: 30),
    .green(duration: 25),
    .yellow(duration: 5),
    .red(duration: 30)
]

simulateTraffic(signals: trafficSequence)
```

### แบบฝึกหัดที่ 3: เครื่องคิดเลขอย่างง่าย

```swift
// โจทย์: สร้างเครื่องคิดเลขที่รับ input เป็น String
enum CalculatorError: Error {
    case invalidInput(String)
    case divisionByZero
    case unsupportedOperation(String)
}

func evaluate(expression: String) throws -> Double {
    // แยกส่วน "10 + 5" หรือ "3.5 * 2"
    let parts = expression.split(separator: " ").map { String($0) }
    
    guard parts.count == 3 else {
        throw CalculatorError.invalidInput("รูปแบบต้องเป็น 'ตัวเลข ตัวดำเนินการ ตัวเลข'")
    }
    
    guard let num1 = Double(parts[0]) else {
        throw CalculatorError.invalidInput("'\(parts[0])' ไม่ใช่ตัวเลข")
    }
    
    guard let num2 = Double(parts[2]) else {
        throw CalculatorError.invalidInput("'\(parts[2])' ไม่ใช่ตัวเลข")
    }
    
    let op = parts[1]
    
    switch op {
    case "+":
        return num1 + num2
    case "-":
        return num1 - num2
    case "*":
        return num1 * num2
    case "/":
        guard num2 != 0 else {
            throw CalculatorError.divisionByZero
        }
        return num1 / num2
    case "%":
        guard num2 != 0 else {
            throw CalculatorError.divisionByZero
        }
        return num1.truncatingRemainder(dividingBy: num2)
    default:
        throw CalculatorError.unsupportedOperation("ไม่รองรับตัวดำเนินการ '\(op)'")
    }
}

// ทดสอบ
let expressions = ["10 + 5", "3.5 * 2", "100 / 4", "10 / 0", "abc + 5", "7 ^ 2"]
for expr in expressions {
    do {
        let result = try evaluate(expression: expr)
        print("\(expr) = \(result)")
    } catch CalculatorError.invalidInput(let msg) {
        print("Input ไม่ถูกต้อง: \(msg)")
    } catch CalculatorError.divisionByZero {
        print("ข้อผิดพลาด: หารด้วยศูนย์!")
    } catch CalculatorError.unsupportedOperation(let msg) {
        print("ข้อผิดพลาด: \(msg)")
    } catch {
        print("ข้อผิดพลาดไม่ทราบสาเหตุ: \(error)")
    }
}
```

### แบบฝึกหัดที่ 4: ระบบจัดการลำดับ Priority

```swift
// โจทย์: สร้างระบบ Task ที่มี Priority
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
    
    var emoji: String {
        switch self {
        case .low:      return "🟢"
        case .medium:   return "🟡"
        case .high:     return "🟠"
        case .critical: return "🔴"
        }
    }
}

struct Task {
    let id: Int
    let title: String
    let priority: Priority
    var isCompleted: Bool = false
    
    var statusLabel: String {
        return isCompleted ? "เสร็จแล้ว" : "รอดำเนินการ"
    }
}

class TaskManager {
    var tasks: [Task] = []
    
    func addTask(_ task: Task) {
        tasks.append(task)
        print("เพิ่มงาน: [\(task.priority.emoji) \(task.priority.label)] \(task.title)")
    }
    
    func getTasksByPriority(_ priority: Priority) -> [Task] {
        return tasks.filter { $0.priority == priority && !$0.isCompleted }
    }
    
    func getNextTask() -> Task? {
        // ดึงงานที่ยังไม่เสร็จและ priority สูงที่สุด
        return tasks
            .filter { !$0.isCompleted }
            .max { $0.priority < $1.priority }
    }
    
    func printSummary() {
        print("\n=== สรุปงานทั้งหมด ===")
        
        let priorities: [Priority] = [.critical, .high, .medium, .low]
        for priority in priorities {
            let pending = getTasksByPriority(priority)
            if !pending.isEmpty {
                print("\n\(priority.emoji) ระดับ\(priority.label):")
                for task in pending {
                    print("  [\(task.id)] \(task.title)")
                }
            }
        }
    }
}

let manager = TaskManager()
manager.addTask(Task(id: 1, title: "แก้ bug การ login", priority: .critical))
manager.addTask(Task(id: 2, title: "อัปเดต dependencies", priority: .medium))
manager.addTask(Task(id: 3, title: "เพิ่มฟีเจอร์ search", priority: .high))
manager.addTask(Task(id: 4, title: "เขียน unit test", priority: .low))
manager.addTask(Task(id: 5, title: "แก้ไข security vulnerability", priority: .critical))

manager.printSummary()

if let nextTask = manager.getNextTask() {
    print("\n⚡ งานที่ควรทำต่อไป: \(nextTask.title)")
}
```

### แบบฝึกหัดที่ 5: Parser สำหรับ Data

```swift
// โจทย์: Parse ข้อมูล CSV อย่างง่าย
struct Record {
    let id: Int
    let name: String
    let score: Double
    let grade: String
}

func parseCSV(_ csv: String) -> [Record] {
    var records: [Record] = []
    let lines = csv.components(separatedBy: "\n")
    
    // ข้ามบรรทัดแรก (header)
    for (index, line) in lines.enumerated() {
        guard index > 0 else { continue }
        guard !line.trimmingCharacters(in: .whitespaces).isEmpty else { continue }
        
        let fields = line.split(separator: ",").map { $0.trimmingCharacters(in: .whitespaces) }
        guard fields.count == 3 else {
            print("บรรทัด \(index + 1): รูปแบบไม่ถูกต้อง (ต้องมี 3 คอลัมน์)")
            continue
        }
        
        guard let id = Int(fields[0]) else {
            print("บรรทัด \(index + 1): ID ไม่ใช่ตัวเลข")
            continue
        }
        
        let name = fields[1]
        guard !name.isEmpty else {
            print("บรรทัด \(index + 1): ชื่อว่างเปล่า")
            continue
        }
        
        guard let score = Double(fields[2]), 0...100 ~= score else {
            print("บรรทัด \(index + 1): คะแนนไม่ถูกต้อง")
            continue
        }
        
        // คำนวณเกรด
        let grade: String
        switch score {
        case 80...100: grade = "A"
        case 70..<80:  grade = "B"
        case 60..<70:  grade = "C"
        case 50..<60:  grade = "D"
        default:       grade = "F"
        }
        
        records.append(Record(id: id, name: name, score: score, grade: grade))
    }
    
    return records
}

let csvData = """
ID,Name,Score
1,Alice,92.5
2,Bob,78.0
3,Charlie,invalid
4,,85.0
5,Diana,63.5
6,Eve,45.0
"""

let parsed = parseCSV(csvData)
print("\n=== ผลการ Parse ===")
for record in parsed {
    print(String(format: "[%d] %-10s %.1f -> %s",
                record.id,
                (record.name as NSString).utf8String!,
                record.score,
                record.grade))
}
print("จำนวนที่ parse สำเร็จ: \(parsed.count) รายการ")
```

---

## 4.21 ตัวอย่าง Real-World จาก iOS Development

### ตัวอย่างที่ 1: URL Validation

```swift
enum URLValidationError {
    case empty
    case missingScheme
    case invalidScheme(String)
    case missingHost
    case tooLong
}

func validateURL(_ urlString: String) -> Result<URL, URLValidationError> {
    // ตรวจสอบว่าว่างเปล่าหรือไม่
    guard !urlString.isEmpty else {
        return .failure(.empty)
    }
    
    // ตรวจสอบความยาว
    guard urlString.count <= 2048 else {
        return .failure(.tooLong)
    }
    
    // แปลงเป็น URL
    guard let url = URL(string: urlString) else {
        return .failure(.missingScheme)
    }
    
    // ตรวจสอบ scheme
    guard let scheme = url.scheme else {
        return .failure(.missingScheme)
    }
    
    guard scheme == "https" || scheme == "http" else {
        return .failure(.invalidScheme(scheme))
    }
    
    // ตรวจสอบ host
    guard let host = url.host, !host.isEmpty else {
        return .failure(.missingHost)
    }
    
    return .success(url)
}

let testURLs = [
    "",
    "not-a-url",
    "ftp://example.com",
    "https://",
    "https://www.example.com/path?query=1"
]

for urlString in testURLs {
    switch validateURL(urlString) {
    case .success(let url):
        print("✓ ถูกต้อง: \(url.absoluteString)")
    case .failure(let error):
        switch error {
        case .empty:
            print("✗ '\(urlString)': URL ว่างเปล่า")
        case .missingScheme:
            print("✗ '\(urlString)': ไม่มี scheme (https/http)")
        case .invalidScheme(let scheme):
            print("✗ '\(urlString)': scheme '\(scheme)' ไม่รองรับ")
        case .missingHost:
            print("✗ '\(urlString)': ไม่มี host")
        case .tooLong:
            print("✗ '\(urlString)': URL ยาวเกินไป")
        }
    }
}
```

### ตัวอย่างที่ 2: Form Validation

```swift
// ระบบตรวจสอบฟอร์มลงทะเบียน
struct RegistrationForm {
    var username: String
    var email: String
    var password: String
    var confirmPassword: String
    var age: Int
    var agreedToTerms: Bool
}

struct ValidationResult {
    var errors: [String] = []
    var isValid: Bool { errors.isEmpty }
    
    mutating func addError(_ message: String) {
        errors.append(message)
    }
}

func validateRegistration(_ form: RegistrationForm) -> ValidationResult {
    var result = ValidationResult()
    
    // ตรวจสอบ username
    switch form.username.count {
    case 0:
        result.addError("กรุณากรอก Username")
    case 1..<3:
        result.addError("Username ต้องมีอย่างน้อย 3 ตัวอักษร")
    case 21...:
        result.addError("Username ต้องไม่เกิน 20 ตัวอักษร")
    default:
        if !form.username.allSatisfy({ $0.isLetter || $0.isNumber || $0 == "_" }) {
            result.addError("Username ใช้ได้เฉพาะตัวอักษร ตัวเลข และ _")
        }
    }
    
    // ตรวจสอบ email
    if form.email.isEmpty {
        result.addError("กรุณากรอก Email")
    } else if !form.email.contains("@") || !form.email.contains(".") {
        result.addError("รูปแบบ Email ไม่ถูกต้อง")
    }
    
    // ตรวจสอบ password
    let passwordChecks: [(condition: Bool, message: String)] = [
        (form.password.count < 8, "รหัสผ่านต้องมีอย่างน้อย 8 ตัวอักษร"),
        (!form.password.contains { $0.isUppercase }, "รหัสผ่านต้องมีตัวพิมพ์ใหญ่"),
        (!form.password.contains { $0.isLowercase }, "รหัสผ่านต้องมีตัวพิมพ์เล็ก"),
        (!form.password.contains { $0.isNumber }, "รหัสผ่านต้องมีตัวเลข")
    ]
    
    for check in passwordChecks where check.condition {
        result.addError(check.message)
    }
    
    // ตรวจสอบว่ารหัสผ่านตรงกัน
    if form.password != form.confirmPassword {
        result.addError("รหัสผ่านไม่ตรงกัน")
    }
    
    // ตรวจสอบอายุ
    switch form.age {
    case ..<13:
        result.addError("ต้องมีอายุอย่างน้อย 13 ปี")
    case 13..<18:
        result.addError("ผู้ใช้อายุต่ำกว่า 18 ปีต้องมีผู้ปกครองยินยอม")
    default:
        break
    }
    
    // ตรวจสอบการยอมรับเงื่อนไข
    guard form.agreedToTerms else {
        result.addError("กรุณายอมรับข้อตกลงการใช้งาน")
        return result
    }
    
    return result
}

let form1 = RegistrationForm(
    username: "alice_2024",
    email: "alice@example.com",
    password: "P@ssw0rd!",
    confirmPassword: "P@ssw0rd!",
    age: 25,
    agreedToTerms: true
)

let form2 = RegistrationForm(
    username: "ab",
    email: "notanemail",
    password: "weak",
    confirmPassword: "different",
    age: 12,
    agreedToTerms: false
)

for (index, form) in [form1, form2].enumerated() {
    let validation = validateRegistration(form)
    print("\n=== ฟอร์มที่ \(index + 1) ===")
    if validation.isValid {
        print("✓ ผ่านการตรวจสอบทั้งหมด!")
    } else {
        print("✗ พบข้อผิดพลาด \(validation.errors.count) รายการ:")
        for error in validation.errors {
            print("  • \(error)")
        }
    }
}
```

---

## 4.22 สรุป (Summary)

ในบทนี้เราได้เรียนรู้การควบคุมการไหลของโปรแกรมใน Swift อย่างละเอียด:

### คำสั่งที่เรียนรู้

| คำสั่ง | การใช้งาน |
|---|---|
| `if` | ทำงานเมื่อเงื่อนไขเป็นจริง |
| `if-else` | เลือกระหว่างสองทาง |
| `if-else if-else` | เลือกระหว่างหลายทาง |
| `guard` | ออกเร็วเมื่อเงื่อนไขไม่เป็นจริง |
| `switch` | จับคู่กับหลาย pattern |
| `fallthrough` | ให้ switch ทำงานต่อในase ถัดไป |
| Labeled statements | ควบคุม loop ซ้อนกัน |
| `#if`, `#else`, `#endif` | Conditional compilation |

### รูปแบบที่ดีใน Swift

1. **Early Exit** - ใช้ `guard` ออกเร็วเมื่อเงื่อนไขไม่ผ่าน
2. **Pattern Matching** - ใช้ `switch` กับ ranges, tuples, value binding
3. **Optional Binding** - ใช้ `if let` หรือ `guard let` แทนการ force unwrap
4. **Exhaustive Switch** - cover ทุก case หรือใช้ `default`
5. **Named Conditions** - ตั้งชื่อเงื่อนไขที่ซับซ้อนให้อ่านง่าย

### ข้อแตกต่างสำคัญจากภาษาอื่น

- `switch` ใน Swift ไม่ต้องมี `break`
- `if` ต้องมีเงื่อนไขเป็น `Bool` เสมอ (ไม่ใช้ 0/non-zero)
- `guard` เผยแพร่ค่าออกนอกบล็อก (ต่างจาก `if let`)
- `switch` ต้อง exhaustive (ครอบคลุมทุกกรณี)
- รองรับ Pattern Matching ที่หลากหลายมาก

### คำถามทบทวน

1. ความแตกต่างระหว่าง `if let` กับ `guard let` คืออะไร?
2. เมื่อใดควรใช้ `switch` แทน `if-else if-else`?
3. `fallthrough` ใน Swift ทำงานอย่างไร ต่างจาก Java อย่างไร?
4. `where` clause ใน `switch` ใช้ทำอะไร?
5. `@unknown default` ต่างจาก `default` อย่างไร และเมื่อใดควรใช้?
6. Labeled statements มีประโยชน์อย่างไร?
7. ทำไม Swift จึงบังคับให้ `switch` เป็น exhaustive?

---

*บทต่อไป: ส่วนที่ 5 - การวนซ้ำ (Loops: for-in, while, repeat-while)*
