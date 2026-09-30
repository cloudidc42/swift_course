# ตอนที่ 5: Loops และ Iteration ใน Swift

## บทนำ

การวนซ้ำ (Looping) เป็นหนึ่งในแนวคิดพื้นฐานที่สำคัญที่สุดในการเขียนโปรแกรม การวนซ้ำช่วยให้เราสามารถทำงานซ้ำๆ ได้โดยไม่ต้องเขียนโค้ดซ้ำหลายครั้ง Swift มี Loop หลายประเภทที่ช่วยให้การเขียนโปรแกรมมีประสิทธิภาพและอ่านง่ายขึ้น

ในบทนี้เราจะเรียนรู้:
- `for-in` loop สำหรับการวนซ้ำผ่าน Collection
- `while` loop สำหรับการวนซ้ำตามเงื่อนไข
- `repeat-while` loop ที่รับประกันการทำงานอย่างน้อยหนึ่งครั้ง
- การควบคุม Loop ด้วย `break` และ `continue`
- Labeled loops สำหรับ Nested loops
- Method ที่ช่วยในการ Iteration เช่น `forEach`, `enumerated()`, `zip()`
- และอื่นๆ อีกมากมาย

---

## 5.1 For-In Loop พื้นฐาน

`for-in` loop เป็น Loop ที่ใช้บ่อยที่สุดใน Swift ใช้สำหรับวนซ้ำผ่าน Sequence ต่างๆ

### Syntax พื้นฐาน

```swift
for item in collection {
    // โค้ดที่ต้องการทำซ้ำ
}
```

### ตัวอย่างง่ายๆ

```swift
// วนซ้ำผ่านตัวเลข
for number in 1...5 {
    print("จำนวน: \(number)")
}
// Output:
// จำนวน: 1
// จำนวน: 2
// จำนวน: 3
// จำนวน: 4
// จำนวน: 5

// วนซ้ำผ่าน Array ของ String
let fruits = ["แอปเปิ้ล", "กล้วย", "ส้ม", "มะม่วง"]
for fruit in fruits {
    print("ผลไม้: \(fruit)")
}
// Output:
// ผลไม้: แอปเปิ้ล
// ผลไม้: กล้วย
// ผลไม้: ส้ม
// ผลไม้: มะม่วง
```

### การใช้ Wildcard `_` เมื่อไม่ต้องการค่า

บางครั้งเราต้องการทำซ้ำตามจำนวนครั้งโดยไม่สนใจค่าใน Loop

```swift
// แสดงข้อความ 3 ครั้งโดยไม่ใช้ค่า
for _ in 1...3 {
    print("Hello, World!")
}
// Output:
// Hello, World!
// Hello, World!
// Hello, World!

// นับเวลาถอยหลัง
var countdown = 5
for _ in 1...5 {
    print("เหลือ \(countdown) วินาที")
    countdown -= 1
}
```

---

## 5.2 For-In กับ Ranges

Swift มี Range หลายประเภทที่ใช้ใน for-in loop ได้

### Closed Range (`...`)

```swift
// รวมทั้งต้นและปลาย Range
for i in 1...10 {
    print(i, terminator: " ")
}
// Output: 1 2 3 4 5 6 7 8 9 10
```

### Half-Open Range (`..<`)

```swift
// รวมต้น Range แต่ไม่รวมปลาย
for i in 0..<5 {
    print(i, terminator: " ")
}
// Output: 0 1 2 3 4

// ประโยชน์ในการวนผ่าน Array index
let colors = ["แดง", "เขียว", "น้ำเงิน"]
for i in 0..<colors.count {
    print("สี[\(i)] = \(colors[i])")
}
```

### One-Sided Range

```swift
let numbers = [10, 20, 30, 40, 50]

// ตั้งแต่ index 2 เป็นต้นไป
for num in numbers[2...] {
    print(num, terminator: " ")
}
// Output: 30 40 50

// จนถึง index 2
for num in numbers[...2] {
    print(num, terminator: " ")
}
// Output: 10 20 30

// จนถึงแต่ไม่รวม index 3
for num in numbers[..<3] {
    print(num, terminator: " ")
}
// Output: 10 20 30
```

### การคำนวณผลรวม

```swift
// คำนวณผลรวมของตัวเลข 1 ถึง 100
var sum = 0
for i in 1...100 {
    sum += i
}
print("ผลรวมของ 1 ถึง 100 = \(sum)")
// Output: ผลรวมของ 1 ถึง 100 = 5050
```

---

## 5.3 For-In กับ Arrays

### การวนผ่าน Array ธรรมดา

```swift
let temperatures = [32.5, 28.3, 35.1, 29.8, 31.2]

var totalTemp = 0.0
for temp in temperatures {
    totalTemp += temp
}
let average = totalTemp / Double(temperatures.count)
print("อุณหภูมิเฉลี่ย: \(average) องศา")

// ค้นหาค่าสูงสุด
var maxTemp = temperatures[0]
for temp in temperatures {
    if temp > maxTemp {
        maxTemp = temp
    }
}
print("อุณหภูมิสูงสุด: \(maxTemp) องศา")
```

### การวนผ่าน Array of Structs

```swift
struct Student {
    let name: String
    let score: Int
}

let students = [
    Student(name: "สมชาย", score: 85),
    Student(name: "สมหญิง", score: 92),
    Student(name: "วิชัย", score: 78),
    Student(name: "นภา", score: 95)
]

// แสดงนักเรียนที่ได้คะแนน >= 90
print("นักเรียนที่ได้ A:")
for student in students {
    if student.score >= 90 {
        print("  \(student.name): \(student.score) คะแนน")
    }
}

// คำนวณคะแนนเฉลี่ย
var totalScore = 0
for student in students {
    totalScore += student.score
}
let averageScore = Double(totalScore) / Double(students.count)
print("คะแนนเฉลี่ย: \(averageScore)")
```

### การวนผ่าน Multidimensional Array

```swift
let matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

print("Matrix:")
for row in matrix {
    for element in row {
        print(element, terminator: "\t")
    }
    print()  // ขึ้นบรรทัดใหม่
}
// Output:
// 1    2    3
// 4    5    6
// 7    8    9
```

---

## 5.4 For-In กับ Dictionaries

Dictionary ใน Swift ไม่มีลำดับที่แน่นอน เมื่อวนซ้ำจะได้ `(key, value)` tuple

### การวนผ่าน Dictionary

```swift
let capitals = [
    "ไทย": "กรุงเทพมหานคร",
    "ญี่ปุ่น": "โตเกียว",
    "ฝรั่งเศส": "ปารีส",
    "เยอรมนี": "เบอร์ลิน"
]

for (country, capital) in capitals {
    print("\(country) มีเมืองหลวงคือ \(capital)")
}
// Output (ลำดับอาจแตกต่าง):
// ไทย มีเมืองหลวงคือ กรุงเทพมหานคร
// ญี่ปุ่น มีเมืองหลวงคือ โตเกียว
// ...
```

### การวนผ่านเฉพาะ Keys หรือ Values

```swift
let scores = ["คณิต": 85, "วิทย์": 92, "ภาษาไทย": 78, "อังกฤษ": 88]

// วนผ่านเฉพาะ Keys
print("วิชาที่มี:")
for subject in scores.keys.sorted() {
    print("  \(subject)")
}

// วนผ่านเฉพาะ Values
var total = 0
for score in scores.values {
    total += score
}
print("คะแนนรวม: \(total)")
print("คะแนนเฉลี่ย: \(Double(total) / Double(scores.count))")
```

### Dictionary กับ Array of Structs

```swift
var studentGrades: [String: [Int]] = [
    "สมชาย": [85, 90, 78, 92],
    "สมหญิง": [95, 88, 91, 87],
    "วิชัย": [72, 68, 75, 80]
]

for (name, grades) in studentGrades {
    let average = Double(grades.reduce(0, +)) / Double(grades.count)
    print("\(name) คะแนนเฉลี่ย: \(String(format: "%.1f", average))")
}
```

---

## 5.5 For-In กับ Stride

`stride` ช่วยให้เราวนซ้ำด้วย step ที่กำหนดเอง

### stride(from:to:by:) - Half-Open

```swift
// นับทีละ 2
for i in stride(from: 0, to: 10, by: 2) {
    print(i, terminator: " ")
}
// Output: 0 2 4 6 8

// นับถอยหลัง
for i in stride(from: 10, to: 0, by: -1) {
    print(i, terminator: " ")
}
// Output: 10 9 8 7 6 5 4 3 2 1

// นับทีละ 0.5
for x in stride(from: 0.0, to: 1.1, by: 0.5) {
    print(x, terminator: " ")
}
// Output: 0.0 0.5 1.0
```

### stride(from:through:by:) - Closed

```swift
// นับทีละ 5 รวมถึงค่าสุดท้าย
for i in stride(from: 0, through: 20, by: 5) {
    print(i, terminator: " ")
}
// Output: 0 5 10 15 20

// สร้างตาราง
print("ตาราง 3:")
for i in stride(from: 3, through: 30, by: 3) {
    print("3 × \(i/3) = \(i)")
}
```

### ประยุกต์ใช้ Stride

```swift
// คำนวณค่าไซน์
import Foundation

print("ค่าไซน์ตั้งแต่ 0 ถึง 360 องศา:")
for angle in stride(from: 0, through: 360, by: 45) {
    let radians = Double(angle) * .pi / 180.0
    let sineValue = sin(radians)
    print("sin(\(angle)°) = \(String(format: "%.4f", sineValue))")
}
```

---

## 5.6 While Loop

`while` loop ทำซ้ำตราบใดที่เงื่อนไขเป็น `true`

### Syntax พื้นฐาน

```swift
while condition {
    // โค้ดที่ทำซ้ำ
}
```

### ตัวอย่างพื้นฐาน

```swift
// นับจาก 1 ถึง 5
var count = 1
while count <= 5 {
    print("นับ: \(count)")
    count += 1
}

// ค้นหาตัวเลขแรกที่มากกว่า 100 เมื่อยกกำลัง 2
var number = 1
while number <= 100 {
    number *= 2
}
print("ตัวเลขแรกที่มากกว่า 100 เมื่อยกกำลัง 2: \(number)")
// Output: 128
```

### While Loop กับ Collection

```swift
var items = [5, 3, 8, 1, 9, 2, 7, 4, 6]
var index = 0

// ค้นหาตัวเลขที่มากกว่า 7
while index < items.count {
    if items[index] > 7 {
        print("พบตัวเลขที่มากกว่า 7: \(items[index]) ที่ index \(index)")
    }
    index += 1
}
```

### Algorithm: Binary Search

```swift
func binarySearch(in sortedArray: [Int], target: Int) -> Int? {
    var left = 0
    var right = sortedArray.count - 1
    
    while left <= right {
        let mid = (left + right) / 2
        
        if sortedArray[mid] == target {
            return mid
        } else if sortedArray[mid] < target {
            left = mid + 1
        } else {
            right = mid - 1
        }
    }
    
    return nil  // ไม่พบ
}

let sortedNumbers = [1, 3, 5, 7, 9, 11, 13, 15, 17, 19]
if let position = binarySearch(in: sortedNumbers, target: 11) {
    print("พบ 11 ที่ index: \(position)")
} else {
    print("ไม่พบตัวเลขที่ค้นหา")
}
```

### While Loop สำหรับ Game Logic

```swift
// จำลองเกมเดิน
var position = 0
var steps = 0
let targetPosition = 20

print("เริ่มที่ตำแหน่ง \(position)")
while position < targetPosition {
    let diceRoll = Int.random(in: 1...6)
    position += diceRoll
    steps += 1
    print("ก้าวที่ \(steps): ทอดลูกเต๋าได้ \(diceRoll), อยู่ที่ตำแหน่ง \(min(position, targetPosition))")
}
print("ถึงเป้าหมายใน \(steps) ก้าว!")
```

---

## 5.7 Repeat-While Loop

`repeat-while` ทำโค้ดก่อนแล้วค่อยตรวจสอบเงื่อนไข รับประกันว่าจะทำงานอย่างน้อย 1 ครั้ง

### Syntax พื้นฐาน

```swift
repeat {
    // โค้ดที่ทำซ้ำ
} while condition
```

### ตัวอย่างพื้นฐาน

```swift
// ขอรหัสผ่านจนกว่าจะถูกต้อง (จำลอง)
let correctPassword = "swift2024"
var attempts = 0
var enteredPassword = ""

repeat {
    attempts += 1
    // จำลองการกรอกรหัสผ่าน
    enteredPassword = attempts <= 2 ? "wrongpass" : "swift2024"
    print("ครั้งที่ \(attempts): กรอกรหัสผ่าน...")
} while enteredPassword != correctPassword

print("เข้าสู่ระบบสำเร็จใน \(attempts) ครั้ง")
```

### ความแตกต่างระหว่าง while และ repeat-while

```swift
// while อาจไม่ทำงานเลย
var x = 10
while x < 5 {
    print("while: \(x)")  // ไม่ถูกเรียกเลย
    x += 1
}
print("while loop จบ, x = \(x)")

// repeat-while ทำงานอย่างน้อย 1 ครั้ง
var y = 10
repeat {
    print("repeat-while: \(y)")  // ถูกเรียก 1 ครั้ง
    y += 1
} while y < 5
print("repeat-while loop จบ, y = \(y)")
```

### ประยุกต์ใช้: เมนูโปรแกรม

```swift
// จำลองระบบเมนู
func showMenu() {
    print("\n=== เมนูหลัก ===")
    print("1. ดูข้อมูล")
    print("2. เพิ่มข้อมูล")
    print("3. ลบข้อมูล")
    print("0. ออกจากโปรแกรม")
    print("เลือก: ", terminator: "")
}

var choice = -1
var runCount = 0

repeat {
    showMenu()
    // จำลองการเลือก
    let choices = [1, 2, 3, 1, 0]
    choice = runCount < choices.count ? choices[runCount] : 0
    runCount += 1
    
    switch choice {
    case 1:
        print("กำลังดูข้อมูล...")
    case 2:
        print("กำลังเพิ่มข้อมูล...")
    case 3:
        print("กำลังลบข้อมูล...")
    case 0:
        print("ออกจากโปรแกรม")
    default:
        print("เลือกใหม่")
    }
} while choice != 0
```

---

## 5.8 Break Statement

`break` ใช้สำหรับหยุด Loop ทันที

### Break ใน For Loop

```swift
// ค้นหาตัวเลขแรกที่หารด้วย 7 ลงตัว
let numbers = [15, 22, 49, 14, 35, 7, 21]

for number in numbers {
    if number % 7 == 0 {
        print("พบตัวเลขแรกที่หารด้วย 7 ลงตัว: \(number)")
        break  // หยุด Loop ทันทีเมื่อพบ
    }
}
```

### Break ใน While Loop

```swift
// ค้นหาตัวเลขที่ยกกำลัง 2 แล้วมากกว่า 1000
var base = 1
while true {
    if base * base > 1000 {
        print("จำนวนแรกที่ยกกำลัง 2 แล้วมากกว่า 1000: \(base)")
        print("\(base)² = \(base * base)")
        break
    }
    base += 1
}
```

### Break ใน Switch Statement

```swift
let grade = "B"

switch grade {
case "A":
    print("ดีเยี่ยม")
    // break เป็น default ใน Swift switch
case "B":
    print("ดี")
case "C":
    print("พอใช้")
default:
    print("ควรปรับปรุง")
}
```

---

## 5.9 Continue Statement

`continue` ใช้สำหรับข้ามการทำงานในรอบปัจจุบันและไปยังรอบถัดไป

### Continue ใน For Loop

```swift
// แสดงเฉพาะเลขคู่
for i in 1...10 {
    if i % 2 != 0 {
        continue  // ข้ามเลขคี่
    }
    print(i, terminator: " ")
}
// Output: 2 4 6 8 10

// กรองสตริงที่ไม่ต้องการ
let words = ["apple", "", "banana", "  ", "cherry", "date"]
var validWords: [String] = []

for word in words {
    if word.trimmingCharacters(in: .whitespaces).isEmpty {
        continue  // ข้ามสตริงว่างหรือมีแค่ช่องว่าง
    }
    validWords.append(word)
}
print("คำที่ถูกต้อง: \(validWords)")
```

### Continue ใน While Loop

```swift
var index = 0
let data = [1, -2, 3, -4, 5, -6, 7, -8, 9, -10]

var positiveSum = 0
while index < data.count {
    let value = data[index]
    index += 1
    
    if value < 0 {
        continue  // ข้ามค่าลบ
    }
    positiveSum += value
}
print("ผลรวมของค่าบวก: \(positiveSum)")
```

### ความแตกต่างระหว่าง break และ continue

```swift
// break หยุด Loop ทันที
print("ตัวอย่าง break:")
for i in 1...10 {
    if i == 5 {
        break
    }
    print(i, terminator: " ")
}
print("\nหลัง loop")
// Output: 1 2 3 4

// continue ข้ามรอบนั้นและไปรอบถัดไป
print("\nตัวอย่าง continue:")
for i in 1...10 {
    if i == 5 {
        continue
    }
    print(i, terminator: " ")
}
print("\nหลัง loop")
// Output: 1 2 3 4 6 7 8 9 10
```

---

## 5.10 Labeled Loops

Labeled loops ช่วยให้เราระบุได้ว่าต้องการ `break` หรือ `continue` ที่ Loop ไหนใน Nested loops

### Syntax ของ Labeled Loops

```swift
outerLabel: for i in range1 {
    innerLabel: for j in range2 {
        // ใช้ break outerLabel เพื่อออกจาก outer loop
        // ใช้ continue outerLabel เพื่อข้ามไปยังรอบถัดไปของ outer loop
    }
}
```

### ตัวอย่าง: ค้นหาใน Matrix

```swift
let searchMatrix = [
    [1,  2,  3,  4],
    [5,  6,  7,  8],
    [9,  10, 11, 12],
    [13, 14, 15, 16]
]

let target = 7
var found = false

outerLoop: for row in 0..<searchMatrix.count {
    for col in 0..<searchMatrix[row].count {
        if searchMatrix[row][col] == target {
            print("พบ \(target) ที่ row=\(row), col=\(col)")
            found = true
            break outerLoop  // ออกจาก outer loop ทันที
        }
    }
}

if !found {
    print("ไม่พบ \(target)")
}
```

### ตัวอย่าง: Continue กับ Labeled Loop

```swift
// ข้ามทั้ง row เมื่อพบค่าลบ
let grid = [
    [1, 2, 3],
    [4, -5, 6],  // row นี้มีค่าลบ
    [7, 8, 9]
]

print("ผลรวมของแต่ละ row (ข้าม row ที่มีค่าลบ):")
rowLoop: for (rowIndex, row) in grid.enumerated() {
    for element in row {
        if element < 0 {
            print("Row \(rowIndex): ข้ามเพราะมีค่าลบ")
            continue rowLoop  // ข้ามไปยัง row ถัดไป
        }
    }
    let rowSum = row.reduce(0, +)
    print("Row \(rowIndex): ผลรวม = \(rowSum)")
}
```

---

## 5.11 Nested Loops

Nested loops คือการวน Loop ซ้อน Loop ใช้สำหรับงานที่ต้องการมิติหลายชั้น

### ตัวอย่างพื้นฐาน: ตารางสูตรคูณ

```swift
// ตารางสูตรคูณ 1-5
print("ตารางสูตรคูณ:")
print("   ", terminator: "")
for j in 1...5 {
    print(String(format: "%4d", j), terminator: "")
}
print()
print("  " + String(repeating: "-", count: 22))

for i in 1...5 {
    print(String(format: "%2d |", i), terminator: "")
    for j in 1...5 {
        print(String(format: "%4d", i * j), terminator: "")
    }
    print()
}
```

### รูปแบบดาว (Pattern Printing)

```swift
// รูปสามเหลี่ยมขึ้น
print("รูปสามเหลี่ยมขึ้น:")
for i in 1...5 {
    for j in 1...i {
        print("*", terminator: "")
    }
    print()
}
// Output:
// *
// **
// ***
// ****
// *****

// รูปสามเหลี่ยมลง
print("\nรูปสามเหลี่ยมลง:")
for i in stride(from: 5, through: 1, by: -1) {
    for _ in 1...i {
        print("*", terminator: "")
    }
    print()
}

// รูปพีระมิด
print("\nรูปพีระมิด:")
let height = 5
for i in 1...height {
    let spaces = height - i
    let stars = 2 * i - 1
    print(String(repeating: " ", count: spaces) + String(repeating: "*", count: stars))
}
```

### Bubble Sort Algorithm

```swift
// Bubble Sort - ใช้ Nested loops
func bubbleSort(_ array: inout [Int]) {
    let n = array.count
    for i in 0..<n-1 {
        for j in 0..<n-1-i {
            if array[j] > array[j+1] {
                array.swapAt(j, j+1)
            }
        }
    }
}

var unsorted = [64, 34, 25, 12, 22, 11, 90]
print("ก่อนเรียงลำดับ: \(unsorted)")
bubbleSort(&unsorted)
print("หลังเรียงลำดับ: \(unsorted)")
```

---

## 5.12 forEach Method

`forEach` เป็น method ที่ทำงานคล้าย `for-in` แต่ใช้ Closure

### Syntax พื้นฐาน

```swift
collection.forEach { item in
    // ทำงานกับ item
}

// หรือแบบสั้น
collection.forEach { print($0) }
```

### ตัวอย่างการใช้ forEach

```swift
let cities = ["กรุงเทพ", "เชียงใหม่", "ขอนแก่น", "ภูเก็ต", "หาดใหญ่"]

// แบบ for-in
print("for-in:")
for city in cities {
    print("  \(city)")
}

// แบบ forEach
print("\nforEach:")
cities.forEach { city in
    print("  \(city)")
}

// แบบสั้นที่สุด
print("\nforEach สั้น:")
cities.forEach { print("  \($0)") }
```

### forEach กับ Dictionary

```swift
let population = [
    "กรุงเทพ": 10_539_000,
    "นนทบุรี": 1_440_000,
    "เชียงใหม่": 1_762_000
]

population.forEach { (city, pop) in
    print("\(city): \(pop.formatted()) คน")
}
```

### ข้อจำกัดของ forEach

```swift
// forEach ไม่รองรับ break และ continue โดยตรง
let numbers = [1, 2, 3, 4, 5]

// นี้จะ compile error:
// numbers.forEach { n in
//     if n == 3 { break }  // Error!
// }

// ถ้าต้องการ break ให้ใช้ for-in แทน
for n in numbers {
    if n == 3 { break }
    print(n)
}
```

---

## 5.13 map, filter, reduce (Preview)

เป็นการปูพื้นฐานก่อนเรียน Higher-Order Functions อย่างเต็มรูปแบบในบทถัดๆ ไป

### map - แปลงทุก Element

```swift
let scores = [70, 85, 60, 92, 78]

// เพิ่มคะแนนทุกคน 5 คะแนน
let boostedScores = scores.map { $0 + 5 }
print("คะแนนเดิม: \(scores)")
print("คะแนนหลังบวก 5: \(boostedScores)")

// แปลงเป็น Grade
let grades = scores.map { score -> String in
    switch score {
    case 90...100: return "A"
    case 80..<90:  return "B"
    case 70..<80:  return "C"
    case 60..<70:  return "D"
    default:       return "F"
    }
}
print("เกรด: \(grades)")
```

### filter - คัดกรอง Element

```swift
let allNumbers = Array(1...20)

// กรองเฉพาะเลขคู่
let evenNumbers = allNumbers.filter { $0 % 2 == 0 }
print("เลขคู่: \(evenNumbers)")

// กรองเฉพาะเลขที่มากกว่า 10
let bigNumbers = allNumbers.filter { $0 > 10 }
print("เลขมากกว่า 10: \(bigNumbers)")
```

### reduce - รวมค่าทั้งหมด

```swift
let prices = [150.0, 299.0, 450.0, 89.0, 199.0]

// คำนวณยอดรวม
let total = prices.reduce(0, +)
print("ยอดรวม: \(total)")

// คำนวณโดย Closure
let totalWithTax = prices.reduce(0.0) { result, price in
    result + price * 1.07  // บวก VAT 7%
}
print("ยอดรวมรวม VAT: \(String(format: "%.2f", totalWithTax))")
```

### ใช้ร่วมกัน (Method Chaining)

```swift
let studentScores = [55, 70, 85, 45, 92, 60, 88, 75]

// หาคะแนนเฉลี่ยของนักเรียนที่ผ่าน (>= 60)
let averagePassingScore = studentScores
    .filter { $0 >= 60 }
    .map { Double($0) }
    .reduce(0.0, +) / Double(studentScores.filter { $0 >= 60 }.count)

print("คะแนนเฉลี่ยของผู้ที่ผ่าน: \(String(format: "%.2f", averagePassingScore))")
```

---

## 5.14 enumerated()

`enumerated()` ช่วยให้ได้ทั้ง index และค่าพร้อมกัน

### Syntax และตัวอย่างพื้นฐาน

```swift
let fruits = ["แอปเปิ้ล", "กล้วย", "ส้ม", "มะม่วง"]

// แบบเก่า (ไม่ค่อยดี)
for i in 0..<fruits.count {
    print("\(i): \(fruits[i])")
}

// แบบใช้ enumerated() (ดีกว่า)
for (index, fruit) in fruits.enumerated() {
    print("\(index): \(fruit)")
}

// หรือเริ่ม index จาก 1
for (index, fruit) in fruits.enumerated() {
    print("\(index + 1). \(fruit)")
}
```

### ประยุกต์ใช้ enumerated()

```swift
// หาตำแหน่งของค่าที่ต้องการ
let temperatures = [23.5, 28.1, 19.8, 32.3, 25.6, 17.2, 30.1]

var highTempDays: [(day: Int, temp: Double)] = []
for (day, temp) in temperatures.enumerated() {
    if temp > 28.0 {
        highTempDays.append((day: day + 1, temp: temp))
    }
}

print("วันที่อุณหภูมิสูงกว่า 28°C:")
for info in highTempDays {
    print("  วันที่ \(info.day): \(info.temp)°C")
}

// เปรียบเทียบกับค่าก่อนหน้า
print("\nการเปลี่ยนแปลงอุณหภูมิ:")
for (index, temp) in temperatures.enumerated() {
    if index > 0 {
        let change = temp - temperatures[index - 1]
        let direction = change > 0 ? "↑" : "↓"
        print("วัน \(index + 1): \(temp)°C (\(direction)\(abs(change)))")
    } else {
        print("วัน 1: \(temp)°C")
    }
}
```

---

## 5.15 zip()

`zip()` ช่วยรวม Sequence สองตัวเข้าด้วยกัน

### Syntax พื้นฐาน

```swift
let names = ["อลิซ", "บ็อบ", "ชาร์ลี"]
let ages = [25, 30, 28]

for (name, age) in zip(names, ages) {
    print("\(name) อายุ \(age) ปี")
}
```

### ตัวอย่างประยุกต์

```swift
// สร้าง Dictionary จากสอง Array
let keys = ["name", "age", "city"]
let values: [Any] = ["สมชาย", 25, "กรุงเทพ"]

var person: [String: Any] = [:]
for (key, value) in zip(keys, values) {
    person[key] = value
}
print("ข้อมูลบุคคล: \(person)")

// เปรียบเทียบสองอาร์เรย์
let expected = [1, 2, 3, 4, 5]
let actual   = [1, 2, 0, 4, 5]

var differences: [(index: Int, expected: Int, actual: Int)] = []
for (index, (exp, act)) in zip(expected, actual).enumerated() {
    if exp != act {
        differences.append((index: index, expected: exp, actual: act))
    }
}

if differences.isEmpty {
    print("ค่าทุกตัวตรงกัน")
} else {
    print("พบความแตกต่าง:")
    for diff in differences {
        print("  Index \(diff.index): คาดหวัง \(diff.expected) แต่ได้ \(diff.actual)")
    }
}
```

### zip กับ Sequence ที่มีขนาดต่างกัน

```swift
// zip จะหยุดที่ Sequence ที่สั้นกว่า
let longArray = [1, 2, 3, 4, 5, 6, 7]
let shortArray = ["a", "b", "c"]

for (num, letter) in zip(longArray, shortArray) {
    print("\(num) - \(letter)")
}
// Output:
// 1 - a
// 2 - b
// 3 - c
// (หยุดที่ 3 เพราะ shortArray มีแค่ 3 ตัว)
```

---

## 5.16 Infinite Loops และการหยุด

Infinite Loop คือ Loop ที่ไม่มีวันสิ้นสุดโดยธรรมชาติ ต้องใช้ `break` เพื่อหยุด

### สร้าง Infinite Loop

```swift
// วิธีสร้าง Infinite Loop
while true {
    // ทำงานไปเรื่อยๆ
}

for _ in 0... {
    // ทำงานไปเรื่อยๆ
}

repeat {
    // ทำงานไปเรื่อยๆ
} while true
```

### ตัวอย่างที่ดีของ Infinite Loop

```swift
// Server-like loop
var requestCount = 0
let maxRequests = 5  // จำลอง limit

while true {
    requestCount += 1
    
    // จำลองการรับ request
    let request = "Request #\(requestCount)"
    print("กำลังประมวลผล: \(request)")
    
    if requestCount >= maxRequests {
        print("ถึงจำนวน request สูงสุดแล้ว")
        break
    }
}

// Event Loop (จำลอง)
var shouldStop = false
var eventCount = 0

while true {
    eventCount += 1
    
    // จำลอง event
    if eventCount == 3 {
        shouldStop = true
    }
    
    print("Event \(eventCount) ถูกประมวลผล")
    
    if shouldStop {
        print("หยุด Event Loop")
        break
    }
}
```

### Infinite Loop กับ Timer (แนวคิด)

```swift
// แนวคิดของ Game Loop
struct GameState {
    var score: Int = 0
    var lives: Int = 3
    var level: Int = 1
    var isRunning: Bool = true
}

var gameState = GameState()

// Game Loop หลัก
while gameState.isRunning {
    // จำลองการเล่นเกม
    gameState.score += Int.random(in: 1...10)
    
    // เช็คเงื่อนไขจบเกม
    if gameState.score >= 50 {
        print("ผ่าน Level \(gameState.level)! Score: \(gameState.score)")
        gameState.level += 1
        gameState.score = 0
        
        if gameState.level > 3 {
            print("คุณชนะเกม!")
            gameState.isRunning = false
        }
    }
    
    if gameState.lives <= 0 {
        print("Game Over!")
        gameState.isRunning = false
    }
}
```

---

## 5.17 ประสิทธิภาพของ Loop

### ควรหลีกเลี่ยงการคำนวณซ้ำใน Loop

```swift
// ไม่ดี: คำนวณ count ทุกรอบ
let largeArray = Array(1...1000)
for i in 0..<largeArray.count {  // .count ถูกเรียกทุกรอบ
    // ...
}

// ดีกว่า: เก็บ count ไว้ก่อน
let count = largeArray.count
for i in 0..<count {
    // ...
}

// ดีที่สุด: ใช้ for-in โดยตรง
for element in largeArray {
    // ...
}
```

### ลดการสร้าง Object ใน Loop

```swift
import Foundation

// ไม่ดี: สร้าง DateFormatter ใหม่ทุกรอบ (ช้ามาก)
let dateStrings = ["2024-01-15", "2024-02-20", "2024-03-10"]

// ไม่ดี:
var dates1: [Date] = []
for str in dateStrings {
    let formatter = DateFormatter()  // สร้างใหม่ทุกรอบ - ช้า!
    formatter.dateFormat = "yyyy-MM-dd"
    if let date = formatter.date(from: str) {
        dates1.append(date)
    }
}

// ดีกว่า:
var dates2: [Date] = []
let formatter = DateFormatter()  // สร้างครั้งเดียว
formatter.dateFormat = "yyyy-MM-dd"
for str in dateStrings {
    if let date = formatter.date(from: str) {
        dates2.append(date)
    }
}
```

### Early Exit เพื่อประสิทธิภาพ

```swift
// ใช้ break เพื่อหยุดทันทีเมื่อพบสิ่งที่ต้องการ
func findFirst(in array: [Int], where predicate: (Int) -> Bool) -> Int? {
    for element in array {
        if predicate(element) {
            return element  // หยุดทันที ไม่วนต่อ
        }
    }
    return nil
}

let bigArray = Array(1...1_000_000)
if let found = findFirst(in: bigArray, where: { $0 > 999_990 }) {
    print("พบ: \(found)")
}
```

---

## 5.18 การวนซ้ำผ่าน String

String ใน Swift เป็น Collection ของ Character สามารถวนซ้ำได้

### วนผ่าน Character

```swift
let message = "สวัสดี Swift"

// วนผ่านทุก Character
for char in message {
    print(char, terminator: " ")
}
print()

// นับตัวอักษรบางประเภท
var letterCount = 0
var spaceCount = 0
for char in message {
    if char.isLetter {
        letterCount += 1
    } else if char.isWhitespace {
        spaceCount += 1
    }
}
print("ตัวอักษร: \(letterCount), ช่องว่าง: \(spaceCount)")
```

### วนผ่าน Unicode Scalars

```swift
let emoji = "Hello! 🌍🚀"

print("Unicode Scalars:")
for scalar in emoji.unicodeScalars {
    print("U+\(String(scalar.value, radix: 16, uppercase: true)): \(scalar)")
}
```

### String Processing

```swift
// หาคำใน String
func countWords(in text: String) -> Int {
    var count = 0
    var inWord = false
    
    for char in text {
        if char.isWhitespace {
            inWord = false
        } else if !inWord {
            inWord = true
            count += 1
        }
    }
    return count
}

let text = "Swift เป็นภาษาโปรแกรมที่ทันสมัยและทรงพลัง"
print("จำนวนคำ: \(countWords(in: text))")

// แปลง String
func reverseWords(_ sentence: String) -> String {
    return sentence
        .split(separator: " ")
        .reversed()
        .joined(separator: " ")
}

let original = "Hello World from Swift"
let reversed = reverseWords(original)
print("กลับคำ: \(reversed)")
```

---

## 5.19 การวนซ้ำผ่าน Files (Preview)

แนวคิดพื้นฐานของการ Iterate ผ่าน Files (จะเรียนรายละเอียดในบทต่อไป)

```swift
import Foundation

// อ่านและประมวลผลบรรทัด (แนวคิด)
func processLines(from text: String) -> [String] {
    var result: [String] = []
    
    // แยก text เป็นบรรทัดๆ
    let lines = text.components(separatedBy: "\n")
    
    for (lineNumber, line) in lines.enumerated() {
        let trimmed = line.trimmingCharacters(in: .whitespaces)
        if !trimmed.isEmpty {
            result.append("บรรทัด \(lineNumber + 1): \(trimmed)")
        }
    }
    
    return result
}

let multilineText = """
บรรทัดที่หนึ่ง
บรรทัดที่สอง

บรรทัดที่สี่
"""

let processed = processLines(from: multilineText)
for line in processed {
    print(line)
}
```

---

## 5.20 Common Patterns (รูปแบบที่พบบ่อย)

### Pattern 1: Accumulator (สะสมค่า)

```swift
// Sum Accumulator
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
var sum = 0
for n in numbers {
    sum += n
}
print("ผลรวม: \(sum)")

// Product Accumulator
var product = 1
for n in numbers where n > 0 {
    product *= n
}
print("ผลคูณ: \(product)")

// String Accumulator
let words = ["Hello", "World", "from", "Swift"]
var sentence = ""
for (index, word) in words.enumerated() {
    if index > 0 { sentence += " " }
    sentence += word
}
print("ประโยค: \(sentence)")

// Array Accumulator
var evenNumbers: [Int] = []
for n in 1...20 {
    if n % 2 == 0 {
        evenNumbers.append(n)
    }
}
print("เลขคู่: \(evenNumbers)")
```

### Pattern 2: Search (ค้นหา)

```swift
// Linear Search
func linearSearch<T: Equatable>(in array: [T], for target: T) -> Int? {
    for (index, element) in array.enumerated() {
        if element == target {
            return index
        }
    }
    return nil
}

let data = [10, 25, 7, 43, 18, 55, 32]
if let index = linearSearch(in: data, for: 43) {
    print("พบ 43 ที่ index: \(index)")
}

// Find All Matching
func findAll<T: Equatable>(in array: [T], matching target: T) -> [Int] {
    var indices: [Int] = []
    for (index, element) in array.enumerated() {
        if element == target {
            indices.append(index)
        }
    }
    return indices
}

let repeating = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5]
let indices5 = findAll(in: repeating, matching: 5)
print("พบ 5 ที่ index: \(indices5)")
```

### Pattern 3: Transform (แปลงค่า)

```swift
// แปลงทุก Element
func transformAll<T, U>(_ array: [T], using transform: (T) -> U) -> [U] {
    var result: [U] = []
    for element in array {
        result.append(transform(element))
    }
    return result
}

let celsius = [0.0, 20.0, 37.0, 100.0]
let fahrenheit = transformAll(celsius) { c in c * 9/5 + 32 }
print("°F: \(fahrenheit)")

// แปลงแบบมีเงื่อนไข
func transformWhere<T>(_ array: [T], 
                       condition: (T) -> Bool,
                       transform: (T) -> T) -> [T] {
    var result: [T] = []
    for element in array {
        if condition(element) {
            result.append(transform(element))
        } else {
            result.append(element)
        }
    }
    return result
}

let mixed = [-3, 5, -1, 8, -7, 2]
let absValues = transformWhere(mixed, condition: { $0 < 0 }, transform: { -$0 })
print("ค่าสัมบูรณ์: \(absValues)")
```

---

## 5.21 แบบฝึกหัดพร้อมเฉลย

### แบบฝึกหัดที่ 1: FizzBuzz

```swift
// โจทย์: พิมพ์ตัวเลข 1-100
// - ถ้าหารด้วย 3 ลงตัว พิมพ์ "Fizz"
// - ถ้าหารด้วย 5 ลงตัว พิมพ์ "Buzz"
// - ถ้าหารด้วยทั้ง 3 และ 5 ลงตัว พิมพ์ "FizzBuzz"
// - นอกนั้น พิมพ์ตัวเลข

func fizzBuzz() {
    for i in 1...100 {
        if i % 15 == 0 {
            print("FizzBuzz")
        } else if i % 3 == 0 {
            print("Fizz")
        } else if i % 5 == 0 {
            print("Buzz")
        } else {
            print(i)
        }
    }
}

// ทดสอบ 1-20
for i in 1...20 {
    let result: String
    if i % 15 == 0 { result = "FizzBuzz" }
    else if i % 3 == 0 { result = "Fizz" }
    else if i % 5 == 0 { result = "Buzz" }
    else { result = "\(i)" }
    print(result)
}
```

### แบบฝึกหัดที่ 2: จำนวนเฉพาะ

```swift
// โจทย์: หาจำนวนเฉพาะทั้งหมดที่น้อยกว่าหรือเท่ากับ n

func findPrimes(upTo n: Int) -> [Int] {
    guard n >= 2 else { return [] }
    
    var primes: [Int] = []
    
    for candidate in 2...n {
        var isPrime = true
        
        for divisor in 2..<candidate {
            if divisor * divisor > candidate { break }
            if candidate % divisor == 0 {
                isPrime = false
                break
            }
        }
        
        if isPrime {
            primes.append(candidate)
        }
    }
    
    return primes
}

let primesUpTo50 = findPrimes(upTo: 50)
print("จำนวนเฉพาะถึง 50: \(primesUpTo50)")
print("จำนวนทั้งหมด: \(primesUpTo50.count)")
```

### แบบฝึกหัดที่ 3: Fibonacci Sequence

```swift
// โจทย์: สร้างลำดับ Fibonacci n ตัวแรก

func fibonacci(count: Int) -> [Int] {
    guard count > 0 else { return [] }
    guard count > 1 else { return [0] }
    
    var sequence = [0, 1]
    
    while sequence.count < count {
        let last = sequence[sequence.count - 1]
        let secondLast = sequence[sequence.count - 2]
        sequence.append(last + secondLast)
    }
    
    return sequence
}

let fib15 = fibonacci(count: 15)
print("Fibonacci 15 ตัวแรก:")
for (index, num) in fib15.enumerated() {
    print("F(\(index)) = \(num)")
}
```

### แบบฝึกหัดที่ 4: ตรวจสอบ Palindrome

```swift
// โจทย์: ตรวจสอบว่า String เป็น Palindrome หรือไม่

func isPalindrome(_ str: String) -> Bool {
    let chars = Array(str.lowercased().filter { $0.isLetter })
    var left = 0
    var right = chars.count - 1
    
    while left < right {
        if chars[left] != chars[right] {
            return false
        }
        left += 1
        right -= 1
    }
    
    return true
}

let testStrings = ["radar", "hello", "level", "swift", "madam", "racecar"]
for str in testStrings {
    print("\"\(str)\" เป็น Palindrome: \(isPalindrome(str))")
}
```

### แบบฝึกหัดที่ 5: Triangle Numbers

```swift
// โจทย์: หา Triangle Number ที่ n
// Triangle Numbers: 1, 3, 6, 10, 15, 21, ...
// T(n) = 1 + 2 + 3 + ... + n

func triangleNumber(_ n: Int) -> Int {
    var total = 0
    for i in 1...n {
        total += i
    }
    return total
}

// แสดง 10 Triangle Numbers แรก
print("Triangle Numbers:")
for n in 1...10 {
    print("T(\(n)) = \(triangleNumber(n))")
}

// หา Triangle Number แรกที่มากกว่า 1000
var n = 1
while triangleNumber(n) <= 1000 {
    n += 1
}
print("\nTriangle Number แรกที่มากกว่า 1000: T(\(n)) = \(triangleNumber(n))")
```

---

## 5.22 ตัวอย่างเกมและอัลกอริทึม

### เกม: Guess the Number

```swift
// เกมทายตัวเลข (จำลอง)
func playGuessingGame() {
    let secretNumber = Int.random(in: 1...100)
    var attempts = 0
    var guesses = [25, 75, 50, 60, 55, 58, 56, 57]  // จำลองการทาย
    var guessIndex = 0
    var hasWon = false
    
    print("เกมทายตัวเลข 1-100")
    
    repeat {
        let guess = guessIndex < guesses.count ? guesses[guessIndex] : secretNumber
        guessIndex += 1
        attempts += 1
        
        print("ทาย: \(guess)")
        
        if guess < secretNumber {
            print("น้อยเกินไป!")
        } else if guess > secretNumber {
            print("มากเกินไป!")
        } else {
            print("ถูกต้อง! ใช้ \(attempts) ครั้ง")
            hasWon = true
        }
    } while !hasWon && attempts < 10
    
    if !hasWon {
        print("หมดจำนวนครั้ง ตัวเลขที่ถูกคือ \(secretNumber)")
    }
}

playGuessingGame()
```

### อัลกอริทึม: Pattern Number

```swift
// รูปแบบเลข: Pascal's Triangle
func pascalTriangle(rows: Int) {
    var triangle: [[Int]] = []
    
    for row in 0..<rows {
        var currentRow: [Int] = []
        
        if row == 0 {
            currentRow = [1]
        } else {
            let prevRow = triangle[row - 1]
            currentRow.append(1)
            
            for i in 1..<row {
                currentRow.append(prevRow[i-1] + prevRow[i])
            }
            
            currentRow.append(1)
        }
        
        triangle.append(currentRow)
        
        // แสดงผล
        let spaces = String(repeating: " ", count: (rows - row) * 2)
        let numbers = currentRow.map { "\($0)" }.joined(separator: "   ")
        print(spaces + numbers)
    }
}

print("Pascal's Triangle (5 แถว):")
pascalTriangle(rows: 5)
```

### อัลกอริทึม: Selection Sort

```swift
func selectionSort(_ array: inout [Int]) {
    let n = array.count
    
    for i in 0..<n-1 {
        var minIndex = i
        
        for j in (i+1)..<n {
            if array[j] < array[minIndex] {
                minIndex = j
            }
        }
        
        if minIndex != i {
            array.swapAt(i, minIndex)
        }
    }
}

var toSort = [64, 25, 12, 22, 11]
print("ก่อน: \(toSort)")
selectionSort(&toSort)
print("หลัง: \(toSort)")
```

---

## 5.23 สรุป

ในบทนี้เราได้เรียนรู้ Loop และ Iteration ใน Swift ทั้งหมด:

### สิ่งที่เรียนรู้

| Loop/Feature | ใช้เมื่อ |
|---|---|
| `for-in` | วนผ่าน Collection ที่รู้จำนวนรอบ |
| `while` | วนตามเงื่อนไข ไม่รู้จำนวนรอบ |
| `repeat-while` | ต้องทำอย่างน้อยหนึ่งครั้งก่อนเช็คเงื่อนไข |
| `break` | หยุด Loop ทันที |
| `continue` | ข้ามรอบปัจจุบัน |
| Labeled loops | ควบคุม Nested loops |
| `forEach` | วนผ่าน Collection แบบ Functional |
| `enumerated()` | ได้ทั้ง index และค่า |
| `zip()` | รวม Sequence สองตัว |
| `stride` | วนด้วย step ที่กำหนด |

### หลักการสำคัญ

1. **เลือก Loop ให้เหมาะสม**: `for-in` สำหรับ Collection, `while` สำหรับเงื่อนไข
2. **Early Exit**: ใช้ `break` เพื่อหยุดเมื่อพบสิ่งที่ต้องการ ประหยัดเวลา
3. **Avoid Redundant Calculation**: ไม่คำนวณซ้ำใน Loop
4. **Prefer High-Level**: ใช้ `map`, `filter`, `reduce` แทน Loop ที่ซับซ้อนเมื่อเป็นไปได้
5. **Infinite Loop Safety**: มีเงื่อนไข break เสมอใน Infinite loops

### บทถัดไป

ในบทที่ 6 เราจะเรียนรู้ **Functions** ซึ่งเป็นการจัดกลุ่มโค้ดให้นำไปใช้ซ้ำได้ รวมถึงการส่งผ่านค่า (Parameters) และรับค่ากลับ (Return Values) รวมถึงแนวคิดขั้นสูงเช่น Closures และ Higher-Order Functions

---

*จบบทที่ 5: Loops และ Iteration*
