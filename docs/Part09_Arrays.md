# ส่วนที่ 9: Arrays ใน Swift

## บทนำ (Introduction)

Array คือโครงสร้างข้อมูลพื้นฐานที่สำคัญที่สุดในการเขียนโปรแกรม Swift เป็น **ordered collection** (คอลเลกชันที่มีลำดับ) ที่เก็บข้อมูลหลายค่าในตัวแปรเดียว โดยค่าทั้งหมดต้องเป็นประเภท (type) เดียวกัน

ใน Swift, Array เป็น **value type** ซึ่งหมายความว่าเมื่อคุณกำหนด Array ให้กับตัวแปรอื่น หรือส่งผ่านไปยัง function จะมีการ **copy** ข้อมูลทั้งหมด ไม่ใช่การอ้างอิงไปยังที่เดิม

---

## 9.1 การประกาศและการสร้าง Array (Array Declaration and Initialization)

### การประกาศแบบพื้นฐาน

```swift
// วิธีที่ 1: ระบุ type อย่างชัดเจน
var numbers: Array<Int> = Array<Int>()

// วิธีที่ 2: ใช้ shorthand syntax (แนะนำ)
var names: [String] = [String]()

// วิธีที่ 3: ใช้ type inference
var scores = [Int]()

// วิธีที่ 4: ประกาศเป็น constant ด้วย let
let fixedArray: [Double] = [1.0, 2.0, 3.0]
```

### ความแตกต่างระหว่าง var และ let สำหรับ Array

```swift
// var = mutable array (แก้ไขได้)
var mutableArray = [1, 2, 3]
mutableArray.append(4)          // ทำได้
mutableArray[0] = 10            // ทำได้

// let = immutable array (แก้ไขไม่ได้)
let immutableArray = [1, 2, 3]
// immutableArray.append(4)     // Error! ทำไม่ได้
// immutableArray[0] = 10       // Error! ทำไม่ได้
```

---

## 9.2 Array Literals

Array literal คือการสร้าง Array โดยระบุค่าโดยตรงภายในวงเล็บเหลี่ยม `[ ]`

```swift
// Array ของตัวเลขจำนวนเต็ม
let integers: [Int] = [1, 2, 3, 4, 5]

// Array ของ String
let fruits = ["Apple", "Banana", "Cherry", "Durian", "Elderberry"]

// Array ของ Double
let temperatures = [36.5, 37.0, 36.8, 37.2, 36.9]

// Array ของ Bool
let flags = [true, false, true, true, false]

// Array ของ Character
let vowels: [Character] = ["a", "e", "i", "o", "u"]

// Array ผสมประเภท (ใช้ Any)
let mixed: [Any] = [1, "hello", 3.14, true]

// Array ของ Optional
let optionals: [Int?] = [1, nil, 3, nil, 5]
```

### Type Inference กับ Array Literals

```swift
// Swift สามารถอนุมาน type ได้จาก literal
let numbers = [10, 20, 30, 40, 50]     // อนุมานเป็น [Int]
let words = ["Swift", "is", "awesome"] // อนุมานเป็น [String]
let values = [1.5, 2.5, 3.5]           // อนุมานเป็น [Double]

// ตรวจสอบ type
print(type(of: numbers)) // Array<Int>
print(type(of: words))   // Array<String>
```

---

## 9.3 Empty Arrays (Array ว่างเปล่า)

มีหลายวิธีในการสร้าง Array ว่างเปล่า:

```swift
// วิธีที่ 1: ใช้ initializer
var emptyInts = [Int]()
var emptyStrings = [String]()
var emptyDoubles = Array<Double>()

// วิธีที่ 2: ระบุ type แล้วกำหนดค่า empty array literal
var emptyArray1: [Int] = []
var emptyArray2: [String] = []

// วิธีที่ 3: ใช้ Array() initializer
var emptyArray3 = Array<Int>()

// ตรวจสอบว่าว่างหรือไม่
print(emptyInts.isEmpty)    // true
print(emptyInts.count)      // 0

// เพิ่มข้อมูลทีหลัง
emptyInts.append(1)
emptyInts.append(2)
emptyInts.append(3)
print(emptyInts) // [1, 2, 3]
```

---

## 9.4 Array with Default Values (Array ที่มีค่าเริ่มต้น)

Swift มี initializer พิเศษสำหรับสร้าง Array ที่มีค่าเริ่มต้นเหมือนกันทุกตำแหน่ง:

```swift
// สร้าง Array ที่มีค่า 0 จำนวน 5 ตำแหน่ง
var zeros = Array(repeating: 0, count: 5)
print(zeros) // [0, 0, 0, 0, 0]

// สร้าง Array ที่มีค่า "Hello" จำนวน 3 ตำแหน่ง
var greetings = Array(repeating: "Hello", count: 3)
print(greetings) // ["Hello", "Hello", "Hello"]

// สร้าง Array ที่มีค่า false จำนวน 4 ตำแหน่ง
var boolArray = Array(repeating: false, count: 4)
print(boolArray) // [false, false, false, false]

// ตัวอย่างการใช้งาน: สร้าง score board
var scores = Array(repeating: 0, count: 10)
print("Initial scores: \(scores)")
// Initial scores: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]

// สร้าง 2D array (matrix) ที่มีค่าเริ่มต้น
var matrix = Array(repeating: Array(repeating: 0, count: 3), count: 3)
print(matrix)
// [[0, 0, 0], [0, 0, 0], [0, 0, 0]]
```

---

## 9.5 การเข้าถึงองค์ประกอบของ Array (Accessing Array Elements)

### การใช้ Index

```swift
let fruits = ["Apple", "Banana", "Cherry", "Durian", "Elderberry"]

// การเข้าถึงด้วย index (เริ่มต้นที่ 0)
print(fruits[0]) // Apple
print(fruits[1]) // Banana
print(fruits[4]) // Elderberry

// การเข้าถึง element สุดท้าย
print(fruits[fruits.count - 1]) // Elderberry

// ใช้ property first และ last (คืนค่า Optional)
print(fruits.first)  // Optional("Apple")
print(fruits.last)   // Optional("Elderberry")

// Unwrap ค่า Optional
if let firstFruit = fruits.first {
    print("First fruit: \(firstFruit)") // First fruit: Apple
}

if let lastFruit = fruits.last {
    print("Last fruit: \(lastFruit)") // Last fruit: Elderberry
}
```

### การป้องกัน Index Out of Bounds

```swift
let numbers = [10, 20, 30, 40, 50]

// ตรวจสอบ index ก่อนเข้าถึง
func safeGet(_ array: [Int], at index: Int) -> Int? {
    guard index >= 0 && index < array.count else {
        return nil
    }
    return array[index]
}

print(safeGet(numbers, at: 2))  // Optional(30)
print(safeGet(numbers, at: 10)) // nil

// ใช้ indices property
print(numbers.indices) // 0..<5

if numbers.indices.contains(3) {
    print(numbers[3]) // 40
}
```

### การใช้ Range ในการเข้าถึง

```swift
let letters = ["A", "B", "C", "D", "E", "F", "G"]

// เข้าถึงหลาย elements ด้วย range
let slice1 = letters[1...3]   // ["B", "C", "D"]
let slice2 = letters[2..<5]   // ["C", "D", "E"]
let slice3 = letters[...2]    // ["A", "B", "C"]
let slice4 = letters[4...]    // ["E", "F", "G"]

print(Array(slice1)) // ["B", "C", "D"]
print(Array(slice2)) // ["C", "D", "E"]
```

---

## 9.6 การแก้ไข Array (Modifying Arrays)

### การเพิ่ม Elements

```swift
var shoppingCart = ["Milk", "Bread"]

// append - เพิ่มที่ท้าย
shoppingCart.append("Eggs")
print(shoppingCart) // ["Milk", "Bread", "Eggs"]

// append(contentsOf:) - เพิ่มหลาย elements
shoppingCart.append(contentsOf: ["Butter", "Cheese"])
print(shoppingCart) // ["Milk", "Bread", "Eggs", "Butter", "Cheese"]

// += operator - เพิ่มหลาย elements (เหมือน append(contentsOf:))
shoppingCart += ["Yogurt", "Cream"]
print(shoppingCart) // ["Milk", "Bread", "Eggs", "Butter", "Cheese", "Yogurt", "Cream"]

// insert(at:) - แทรกที่ตำแหน่งที่ระบุ
shoppingCart.insert("Water", at: 0)
print(shoppingCart[0]) // Water

shoppingCart.insert("Juice", at: 2)
print(shoppingCart)
```

### การลบ Elements

```swift
var fruits = ["Apple", "Banana", "Cherry", "Durian", "Elderberry"]

// remove(at:) - ลบที่ index ที่ระบุ (และคืนค่าที่ถูกลบ)
let removed = fruits.remove(at: 1)
print("Removed: \(removed)") // Removed: Banana
print(fruits) // ["Apple", "Cherry", "Durian", "Elderberry"]

// removeFirst() - ลบ element แรก
fruits.removeFirst()
print(fruits) // ["Cherry", "Durian", "Elderberry"]

// removeLast() - ลบ element สุดท้าย
fruits.removeLast()
print(fruits) // ["Cherry", "Durian"]

// removeFirst(_:) - ลบ n elements แรก
var numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
numbers.removeFirst(3)
print(numbers) // [4, 5, 6, 7, 8, 9, 10]

// removeLast(_:) - ลบ n elements สุดท้าย
numbers.removeLast(2)
print(numbers) // [4, 5, 6, 7, 8]

// removeAll() - ลบทั้งหมด
numbers.removeAll()
print(numbers) // []
print(numbers.isEmpty) // true

// removeSubrange - ลบช่วงของ elements
var colors = ["Red", "Green", "Blue", "Yellow", "Purple"]
colors.removeSubrange(1...3)
print(colors) // ["Red", "Purple"]
```

### การแก้ไข Elements

```swift
var grades = [85, 90, 78, 92, 88]

// แก้ไขด้วย index
grades[0] = 95
print(grades) // [95, 90, 78, 92, 88]

// แก้ไขหลาย elements ด้วย range
grades[1...2] = [87, 83]
print(grades) // [95, 87, 83, 92, 88]

// replaceSubrange - แทนที่ช่วงด้วย elements ใหม่
var letters = ["A", "B", "C", "D", "E"]
letters.replaceSubrange(1...3, with: ["X", "Y"])
print(letters) // ["A", "X", "Y", "E"]
```

---

## 9.7 Array Properties

```swift
let numbers = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3]

// count - จำนวน elements
print(numbers.count) // 10

// isEmpty - ตรวจสอบว่าว่างหรือไม่
print(numbers.isEmpty) // false
print([].isEmpty)      // true

// first - element แรก (Optional)
print(numbers.first!)  // 3

// last - element สุดท้าย (Optional)
print(numbers.last!)   // 3

// indices - range ของ valid indices
print(numbers.indices) // 0..<10

// startIndex และ endIndex
print(numbers.startIndex) // 0
print(numbers.endIndex)   // 10

// capacity - ความจุปัจจุบัน (อาจมากกว่า count)
var dynamicArray = [Int]()
print(dynamicArray.capacity) // 0

dynamicArray.reserveCapacity(100)
print(dynamicArray.capacity) // 100 (หรือมากกว่า)
```

---

## 9.8 การวนซ้ำผ่าน Array (Iterating Over Arrays)

### for-in Loop พื้นฐาน

```swift
let fruits = ["Apple", "Banana", "Cherry", "Durian"]

// วนผ่านแต่ละ element
for fruit in fruits {
    print(fruit)
}
// Apple
// Banana
// Cherry
// Durian

// วนพร้อม index ด้วย enumerated()
for (index, fruit) in fruits.enumerated() {
    print("\(index + 1). \(fruit)")
}
// 1. Apple
// 2. Banana
// 3. Cherry
// 4. Durian
```

### การวนแบบต่างๆ

```swift
let numbers = [10, 20, 30, 40, 50]

// ใช้ indices
for index in numbers.indices {
    print("numbers[\(index)] = \(numbers[index])")
}

// ใช้ stride สำหรับการก้าวข้าม
for i in stride(from: 0, to: numbers.count, by: 2) {
    print(numbers[i]) // 10, 30, 50
}

// วนย้อนหลัง
for number in numbers.reversed() {
    print(number) // 50, 40, 30, 20, 10
}

// ใช้ forEach
numbers.forEach { number in
    print(number)
}

// ใช้ forEach กับ closure แบบสั้น
numbers.forEach { print($0) }
```

### while Loop กับ Array

```swift
var queue = [1, 2, 3, 4, 5]

// ประมวลผลจนกว่าจะว่าง
while !queue.isEmpty {
    let first = queue.removeFirst()
    print("Processing: \(first)")
}

// ใช้ index
var items = ["Task1", "Task2", "Task3", "Task4", "Task5"]
var index = 0
while index < items.count {
    print(items[index])
    index += 1
}
```

---

## 9.9 Array Slices

ArraySlice คือ "มุมมอง" (view) ของ Array ส่วนหนึ่ง ไม่ได้ copy ข้อมูล

```swift
let numbers = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

// สร้าง slice
let slice = numbers[3...6]
print(slice)        // [3, 4, 5, 6]
print(type(of: slice)) // ArraySlice<Int>

// สำคัญ: index ของ slice ยังเป็น index ของ original array
print(slice.startIndex) // 3
print(slice.endIndex)   // 7
print(slice[3])         // 3 (ใช้ index เดิม!)
print(slice[4])         // 4

// ระวัง! การใช้ index ที่ผิด
// print(slice[0])  // Runtime error! Index out of bounds

// แปลง slice กลับเป็น Array (มี index เริ่มต้นใหม่)
let arrayFromSlice = Array(slice)
print(arrayFromSlice)             // [3, 4, 5, 6]
print(arrayFromSlice.startIndex)  // 0

// prefix และ suffix
let prefix3 = numbers.prefix(3)
print(Array(prefix3)) // [0, 1, 2]

let suffix3 = numbers.suffix(3)
print(Array(suffix3)) // [7, 8, 9]

let prefix = numbers.prefix(while: { $0 < 5 })
print(Array(prefix)) // [0, 1, 2, 3, 4]

let drop = numbers.drop(while: { $0 < 5 })
print(Array(drop)) // [5, 6, 7, 8, 9]
```

---

## 9.10 Multidimensional Arrays (Array หลายมิติ)

```swift
// 2D Array (Matrix)
var matrix: [[Int]] = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

// เข้าถึงด้วย double index
print(matrix[0][0]) // 1
print(matrix[1][2]) // 6
print(matrix[2][1]) // 8

// แก้ไขค่า
matrix[1][1] = 55
print(matrix[1]) // [4, 55, 6]

// วนซ้ำผ่าน 2D array
for row in matrix {
    for element in row {
        print(element, terminator: " ")
    }
    print() // ขึ้นบรรทัดใหม่
}

// สร้าง 2D array ด้วย Array(repeating:count:)
var grid = Array(repeating: Array(repeating: 0, count: 4), count: 3)
print(grid)
// [[0, 0, 0, 0], [0, 0, 0, 0], [0, 0, 0, 0]]

// 3D Array
var cube: [[[Int]]] = [
    [[1, 2], [3, 4]],
    [[5, 6], [7, 8]]
]
print(cube[0][1][0]) // 3

// ตัวอย่าง: Tic-Tac-Toe Board
var board: [[Character]] = [
    [" ", " ", " "],
    [" ", " ", " "],
    [" ", " ", " "]
]

board[0][0] = "X"
board[1][1] = "O"
board[2][2] = "X"

for row in board {
    print(row.map { String($0) }.joined(separator: "|"))
    print("-+-+-")
}
```

---

## 9.11 Array Operations

### contains

```swift
let fruits = ["Apple", "Banana", "Cherry", "Durian"]

// ตรวจสอบว่ามี element นั้นหรือไม่
print(fruits.contains("Banana")) // true
print(fruits.contains("Mango"))  // false

// ใช้เงื่อนไขที่ซับซ้อน
let numbers = [1, 5, 8, 12, 17, 25]
let hasEven = numbers.contains(where: { $0 % 2 == 0 })
print(hasEven) // true

let hasNegative = numbers.contains(where: { $0 < 0 })
print(hasNegative) // false
```

### firstIndex และ lastIndex

```swift
let letters = ["A", "B", "C", "B", "A", "D"]

// หา index แรกที่พบ
if let firstB = letters.firstIndex(of: "B") {
    print("First B at index: \(firstB)") // First B at index: 1
}

// หา index สุดท้ายที่พบ
if let lastB = letters.lastIndex(of: "B") {
    print("Last B at index: \(lastB)") // Last B at index: 3
}

// ใช้เงื่อนไข
let scores = [75, 82, 90, 65, 88, 92]
if let firstHighScore = scores.firstIndex(where: { $0 >= 90 }) {
    print("First score >= 90 at index: \(firstHighScore)") // 2
}
```

### filter

```swift
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

// กรองเฉพาะเลขคู่
let evenNumbers = numbers.filter { $0 % 2 == 0 }
print(evenNumbers) // [2, 4, 6, 8, 10]

// กรองเฉพาะเลขที่มากกว่า 5
let greaterThan5 = numbers.filter { $0 > 5 }
print(greaterThan5) // [6, 7, 8, 9, 10]

// filter กับ String
let names = ["Alice", "Bob", "Charlie", "David", "Emma"]
let longNames = names.filter { $0.count > 4 }
print(longNames) // ["Alice", "Charlie", "David", "Emma"]

// filter กับ custom type
struct Student {
    let name: String
    let grade: Int
}

let students = [
    Student(name: "Alice", grade: 85),
    Student(name: "Bob", grade: 72),
    Student(name: "Charlie", grade: 91),
    Student(name: "David", grade: 68)
]

let passingStudents = students.filter { $0.grade >= 75 }
print(passingStudents.map { $0.name }) // ["Alice", "Charlie"]
```

### map

```swift
let numbers = [1, 2, 3, 4, 5]

// แปลงค่าทุกตัว
let doubled = numbers.map { $0 * 2 }
print(doubled) // [2, 4, 6, 8, 10]

let squares = numbers.map { $0 * $0 }
print(squares) // [1, 4, 9, 16, 25]

// แปลง type
let strings = numbers.map { String($0) }
print(strings) // ["1", "2", "3", "4", "5"]

// แปลง String เป็น uppercase
let fruits = ["apple", "banana", "cherry"]
let uppercased = fruits.map { $0.uppercased() }
print(uppercased) // ["APPLE", "BANANA", "CHERRY"]

// map กับ struct
let prices = [100.0, 250.0, 75.5, 300.0]
let discountedPrices = prices.map { $0 * 0.9 } // ลด 10%
print(discountedPrices) // [90.0, 225.0, 67.95, 270.0]

// compactMap - map แล้วกำจัด nil
let stringNumbers = ["1", "two", "3", "four", "5"]
let validNumbers = stringNumbers.compactMap { Int($0) }
print(validNumbers) // [1, 3, 5]

// flatMap - map แล้วกำจัด nested arrays
let nestedArrays = [[1, 2, 3], [4, 5], [6, 7, 8, 9]]
let flattened = nestedArrays.flatMap { $0 }
print(flattened) // [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### reduce

```swift
let numbers = [1, 2, 3, 4, 5]

// หาผลรวม
let sum = numbers.reduce(0) { $0 + $1 }
print(sum) // 15

// หาผลคูณ
let product = numbers.reduce(1) { $0 * $1 }
print(product) // 120

// หาค่าสูงสุด (แต่ควรใช้ max() แทน)
let maximum = numbers.reduce(Int.min) { max($0, $1) }
print(maximum) // 5

// สร้าง String จาก Array
let words = ["Swift", "is", "awesome"]
let sentence = words.reduce("") { $0.isEmpty ? $1 : $0 + " " + $1 }
print(sentence) // Swift is awesome

// ใช้ operator สั้นกว่า
let total = numbers.reduce(0, +)
print(total) // 15

// reduce(into:) - มีประสิทธิภาพมากกว่าสำหรับ mutable result
let wordLengths = words.reduce(into: [:]) { dict, word in
    dict[word] = word.count
}
print(wordLengths) // ["Swift": 5, "is": 2, "awesome": 7]
```

---

## 9.12 การเรียงลำดับ Array (Sorting Arrays)

### sorted() และ sort()

```swift
var numbers = [5, 3, 8, 1, 9, 2, 7, 4, 6]

// sorted() - คืนค่า array ใหม่ (ไม่แก้ไขต้นฉบับ)
let sortedAsc = numbers.sorted()
print(sortedAsc) // [1, 2, 3, 4, 5, 6, 7, 8, 9]
print(numbers)   // [5, 3, 8, 1, 9, 2, 7, 4, 6] (ไม่เปลี่ยน)

// sort() - แก้ไข array ในที่
numbers.sort()
print(numbers) // [1, 2, 3, 4, 5, 6, 7, 8, 9]

// เรียงจากมากไปน้อย
let descending = numbers.sorted(by: >)
print(descending) // [9, 8, 7, 6, 5, 4, 3, 2, 1]

// เรียง String
var fruits = ["Cherry", "Apple", "Elderberry", "Banana", "Durian"]
fruits.sort()
print(fruits) // ["Apple", "Banana", "Cherry", "Durian", "Elderberry"]

// เรียงแบบ reverse
fruits.sort(by: >)
print(fruits) // ["Elderberry", "Durian", "Cherry", "Banana", "Apple"]
```

### การเรียงแบบ Custom

```swift
struct Person {
    let name: String
    let age: Int
}

let people = [
    Person(name: "Charlie", age: 30),
    Person(name: "Alice", age: 25),
    Person(name: "Bob", age: 35),
    Person(name: "Diana", age: 28)
]

// เรียงตามอายุ
let byAge = people.sorted { $0.age < $1.age }
byAge.forEach { print("\($0.name): \($0.age)") }
// Alice: 25
// Diana: 28
// Charlie: 30
// Bob: 35

// เรียงตามชื่อ
let byName = people.sorted { $0.name < $1.name }
byName.forEach { print($0.name) }
// Alice, Bob, Charlie, Diana

// เรียงหลาย criteria (ชื่อก่อน ถ้าชื่อเหมือนกันเรียงตามอายุ)
let mixedPeople = [
    Person(name: "Alice", age: 30),
    Person(name: "Bob", age: 25),
    Person(name: "Alice", age: 20),
    Person(name: "Charlie", age: 35)
]

let sorted = mixedPeople.sorted {
    if $0.name != $1.name {
        return $0.name < $1.name
    }
    return $0.age < $1.age
}

sorted.forEach { print("\($0.name): \($0.age)") }
// Alice: 20
// Alice: 30
// Bob: 25
// Charlie: 35
```

### reversed()

```swift
let numbers = [1, 2, 3, 4, 5]

// reversed() คืนค่า ReversedCollection (lazy, ไม่ copy ข้อมูล)
let reversed = numbers.reversed()
print(Array(reversed)) // [5, 4, 3, 2, 1]

// แปลงเป็น array ถ้าต้องการ
var mutableNumbers = [1, 2, 3, 4, 5]
mutableNumbers.reverse() // แก้ไขในที่
print(mutableNumbers) // [5, 4, 3, 2, 1]
```

---

## 9.13 การค้นหาใน Array (Searching in Arrays)

```swift
let numbers = [15, 28, 7, 42, 13, 65, 9, 31]

// Linear search ด้วย contains
print(numbers.contains(42)) // true
print(numbers.contains(100)) // false

// หา index ด้วย firstIndex(of:)
if let index = numbers.firstIndex(of: 42) {
    print("Found 42 at index: \(index)") // Found 42 at index: 3
}

// ค้นหาด้วยเงื่อนไข
if let first = numbers.first(where: { $0 > 30 }) {
    print("First number > 30: \(first)") // First number > 30: 42
}

// allSatisfy - ตรวจสอบว่าทุกค่าตรงเงื่อนไข
let allPositive = numbers.allSatisfy { $0 > 0 }
print(allPositive) // true

// Binary Search (ต้อง sort ก่อน)
let sortedNumbers = numbers.sorted()
print(sortedNumbers) // [7, 9, 13, 15, 28, 31, 42, 65]

// Partition - แบ่ง array ตามเงื่อนไข
var partitionNumbers = [1, 5, 8, 2, 9, 3, 7, 4, 6]
let pivot = partitionNumbers.partition(by: { $0 > 5 })
print(partitionNumbers[..<pivot]) // elements <= 5
print(partitionNumbers[pivot...]) // elements > 5
```

---

## 9.14 Array Concatenation (การต่อ Array)

```swift
let array1 = [1, 2, 3]
let array2 = [4, 5, 6]
let array3 = [7, 8, 9]

// ใช้ + operator
let combined = array1 + array2
print(combined) // [1, 2, 3, 4, 5, 6]

// ต่อหลาย arrays
let allNumbers = array1 + array2 + array3
print(allNumbers) // [1, 2, 3, 4, 5, 6, 7, 8, 9]

// ใช้ += operator
var mutableArray = [1, 2, 3]
mutableArray += [4, 5, 6]
print(mutableArray) // [1, 2, 3, 4, 5, 6]

// append(contentsOf:)
mutableArray.append(contentsOf: [7, 8, 9])
print(mutableArray) // [1, 2, 3, 4, 5, 6, 7, 8, 9]

// ต่อด้วย sequence
let range = 10...15
mutableArray.append(contentsOf: range)
print(mutableArray) // [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15]
```

---

## 9.15 Array Copying (Value Semantics)

Swift Array เป็น value type ใช้ **Copy-on-Write** optimization

```swift
var original = [1, 2, 3, 4, 5]
var copy = original // copy ข้อมูล (แต่ Swift ใช้ CoW ทำให้มีประสิทธิภาพ)

// แก้ไข copy ไม่กระทบ original
copy[0] = 100
print(original) // [1, 2, 3, 4, 5]
print(copy)     // [100, 2, 3, 4, 5]

// แก้ไข original ไม่กระทบ copy
original.append(6)
print(original) // [1, 2, 3, 4, 5, 6]
print(copy)     // [100, 2, 3, 4, 5]

// Copy-on-Write (CoW)
// Swift ไม่ copy จนกว่าจะมีการแก้ไข
var a = [1, 2, 3]
var b = a // ยังไม่มีการ copy จริง (shared buffer)

b.append(4) // ตอนนี้ถึงจะ copy
print(a) // [1, 2, 3] (ไม่เปลี่ยน)
print(b) // [1, 2, 3, 4]

// ตัวอย่างการแสดงผล value semantics
func modifyArray(_ arr: [Int]) -> [Int] {
    var modified = arr
    modified.append(999)
    return modified
}

let original2 = [1, 2, 3]
let modified = modifyArray(original2)
print(original2) // [1, 2, 3] (ไม่เปลี่ยน)
print(modified)  // [1, 2, 3, 999]
```

---

## 9.16 Array as Function Parameters

```swift
// Pass array by value (default)
func printArray(_ arr: [Int]) {
    for item in arr {
        print(item)
    }
}

let numbers = [1, 2, 3, 4, 5]
printArray(numbers) // ส่ง copy ไปให้ function

// inout - ส่ง reference เพื่อแก้ไขได้
func doubleElements(_ arr: inout [Int]) {
    for i in arr.indices {
        arr[i] *= 2
    }
}

var myNumbers = [1, 2, 3, 4, 5]
doubleElements(&myNumbers)
print(myNumbers) // [2, 4, 6, 8, 10]

// Return array จาก function
func generateFibonacci(count: Int) -> [Int] {
    guard count > 0 else { return [] }
    var result = [0]
    if count == 1 { return result }
    result.append(1)
    for i in 2..<count {
        result.append(result[i-1] + result[i-2])
    }
    return result
}

let fibonacci = generateFibonacci(count: 10)
print(fibonacci) // [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

// Variadic parameters (รับหลายค่าเป็น array)
func sum(_ numbers: Int...) -> Int {
    return numbers.reduce(0, +)
}

print(sum(1, 2, 3))           // 6
print(sum(1, 2, 3, 4, 5))    // 15
print(sum(10, 20, 30, 40))   // 100
```

---

## 9.17 ContiguousArray

`ContiguousArray` คล้ายกับ `Array` แต่รับประกันว่าข้อมูลถูกเก็บใน contiguous memory block ทำให้มีประสิทธิภาพสูงกว่าเล็กน้อยสำหรับ element ที่เป็น value type

```swift
import Foundation

// สร้าง ContiguousArray
var contiguous: ContiguousArray<Int> = [1, 2, 3, 4, 5]

// API เหมือนกับ Array ทุกอย่าง
contiguous.append(6)
print(contiguous) // [1, 2, 3, 4, 5, 6]

contiguous.sort()
print(contiguous)

// แปลงระหว่าง Array และ ContiguousArray
let regularArray = Array(contiguous)
let backToContiguous = ContiguousArray(regularArray)

// เมื่อใช้ ContiguousArray?
// - เมื่อ element เป็น value type (Int, Double, struct)
// - ต้องการประสิทธิภาพสูงสุดในการเข้าถึง
// - ทำงานกับ C code หรือ low-level APIs

// ตัวอย่างการ benchmark (แนวคิด)
var regularPerf = [Int](repeating: 0, count: 1000000)
var contiguousPerf = ContiguousArray<Int>(repeating: 0, count: 1000000)

// ContiguousArray มักเร็วกว่าเล็กน้อยสำหรับ numeric operations
```

---

## 9.18 ArraySlice

```swift
let numbers = Array(0...9) // [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

// สร้าง ArraySlice
let middle: ArraySlice<Int> = numbers[3...6]
print(middle)        // [3, 4, 5, 6]
print(middle.count)  // 4

// สำคัญ: ArraySlice ใช้ index จาก original array
print(middle.startIndex) // 3 (ไม่ใช่ 0!)
print(middle.endIndex)   // 7

// วนซ้ำผ่าน slice (ใช้ for-in ปลอดภัย)
for element in middle {
    print(element) // 3, 4, 5, 6
}

// แปลงเป็น Array ใหม่
let newArray = Array(middle) // index เริ่มต้นที่ 0

// ใช้ slice กับ algorithms
let evens = numbers[...].filter { $0 % 2 == 0 }
print(evens) // [0, 2, 4, 6, 8]

// ข้อควรระวัง: ArraySlice ยึดถือ original array ในหน่วยความจำ
// ไม่ควรเก็บ ArraySlice ไว้นานเกินความจำเป็น
func processSlice(_ slice: ArraySlice<Int>) {
    // ควรแปลงเป็น Array ก่อนเก็บไว้ใช้นานๆ
    let localArray = Array(slice)
    print(localArray)
}
```

---

## 9.19 Common Array Algorithms (อัลกอริทึมที่ใช้บ่อย)

### การนับ (Counting)

```swift
let items = ["apple", "banana", "apple", "cherry", "banana", "apple"]

// นับด้วย reduce
let appleCount = items.reduce(0) { $0 + ($1 == "apple" ? 1 : 0) }
print(appleCount) // 3

// นับทุก element ด้วย Dictionary
let countDict = items.reduce(into: [:]) { dict, item in
    dict[item, default: 0] += 1
}
print(countDict) // ["apple": 3, "banana": 2, "cherry": 1]

// NSCountedSet (เฉพาะ macOS/iOS)
// import Foundation
// let counted = NSCountedSet(array: items)
```

### การกำจัดซ้ำ (Deduplication)

```swift
let duplicates = [1, 2, 3, 2, 4, 3, 5, 1, 6]

// วิธีที่ 1: ใช้ Set (ไม่รักษาลำดับ)
let unique1 = Array(Set(duplicates))
print(unique1.sorted()) // [1, 2, 3, 4, 5, 6]

// วิธีที่ 2: รักษาลำดับ
var seen = Set<Int>()
let unique2 = duplicates.filter { seen.insert($0).inserted }
print(unique2) // [1, 2, 3, 4, 5, 6]
```

### Chunking (แบ่งกลุ่ม)

```swift
extension Array {
    func chunks(of size: Int) -> [[Element]] {
        return stride(from: 0, to: count, by: size).map {
            Array(self[$0..<Swift.min($0 + size, count)])
        }
    }
}

let numbers = Array(1...10)
let chunks = numbers.chunks(of: 3)
print(chunks) // [[1, 2, 3], [4, 5, 6], [7, 8, 9], [10]]
```

### Zip

```swift
let names = ["Alice", "Bob", "Charlie"]
let scores = [85, 92, 78]

// จับคู่สอง arrays
let combined = zip(names, scores)
for (name, score) in combined {
    print("\(name): \(score)")
}

// สร้าง Dictionary จาก zip
let scoreDict = Dictionary(uniqueKeysWithValues: zip(names, scores))
print(scoreDict) // ["Alice": 85, "Bob": 92, "Charlie": 78]
```

### Prefix Sum และ Running Total

```swift
let values = [3, 1, 4, 1, 5, 9, 2, 6]

// คำนวณ running total
var runningTotal = 0
let prefixSums = values.map { value -> Int in
    runningTotal += value
    return runningTotal
}
print(prefixSums) // [3, 4, 8, 9, 14, 23, 25, 31]

// หรือใช้ scan (ถ้ามี)
// ใน Swift ไม่มี scan built-in แต่สร้างเองได้
extension Array {
    func scan<T>(_ initial: T, _ combine: (T, Element) -> T) -> [T] {
        var result: [T] = []
        var current = initial
        for element in self {
            current = combine(current, element)
            result.append(current)
        }
        return result
    }
}

let sums = values.scan(0, +)
print(sums) // [3, 4, 8, 9, 14, 23, 25, 31]
```

---

## 9.20 Performance Considerations (ประสิทธิภาพ)

### reserveCapacity

```swift
// ถ้ารู้จำนวน element ล่วงหน้า ควร reserve ก่อน
var optimized = [Int]()
optimized.reserveCapacity(1000)

for i in 0..<1000 {
    optimized.append(i)
}
// ลดการ resize หลายครั้ง → เร็วกว่า

// เปรียบเทียบ
var withoutReserve = [Int]()
for i in 0..<1000 {
    withoutReserve.append(i) // อาจ resize หลายครั้ง
}
```

### ความซับซ้อน (Complexity)

```
Operation           | Time Complexity
--------------------|----------------
Access by index     | O(1)
append              | O(1) amortized
insert at front     | O(n)
insert at middle    | O(n)
remove at index     | O(n)
contains            | O(n)
sort                | O(n log n)
filter/map          | O(n)
```

### Lazy Evaluation

```swift
let largeArray = Array(1...1_000_000)

// ไม่ใช้ lazy: สร้าง array ชั่วคราวขนาดใหญ่
let filtered = largeArray.filter { $0 % 2 == 0 }.prefix(10)

// ใช้ lazy: คำนวณแบบขี้เกียจ ประหยัดหน่วยความจำ
let lazyFiltered = largeArray.lazy.filter { $0 % 2 == 0 }.prefix(10)
print(Array(lazyFiltered)) // [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]
```

---

## 9.21 ตัวอย่างโปรแกรมจริง (Real-World Examples)

### ตัวอย่างที่ 1: ระบบจัดการรายชื่อนักเรียน

```swift
struct Student {
    let id: Int
    let name: String
    var scores: [Int]
    
    var average: Double {
        guard !scores.isEmpty else { return 0 }
        return Double(scores.reduce(0, +)) / Double(scores.count)
    }
    
    var grade: String {
        switch average {
        case 90...100: return "A"
        case 80..<90:  return "B"
        case 70..<80:  return "C"
        case 60..<70:  return "D"
        default:       return "F"
        }
    }
}

class StudentManager {
    private var students: [Student] = []
    
    func addStudent(_ student: Student) {
        students.append(student)
    }
    
    func removeStudent(withId id: Int) {
        students.removeAll { $0.id == id }
    }
    
    func findStudent(withId id: Int) -> Student? {
        return students.first { $0.id == id }
    }
    
    func topStudents(count: Int) -> [Student] {
        return students.sorted { $0.average > $1.average }.prefix(count).map { $0 }
    }
    
    func studentsByGrade(_ grade: String) -> [Student] {
        return students.filter { $0.grade == grade }
    }
    
    func classAverage() -> Double {
        guard !students.isEmpty else { return 0 }
        let totalAvg = students.map { $0.average }.reduce(0, +)
        return totalAvg / Double(students.count)
    }
    
    func printReport() {
        print("=== Student Report ===")
        let sorted = students.sorted { $0.average > $1.average }
        for (rank, student) in sorted.enumerated() {
            print("\(rank + 1). \(student.name) - Average: \(String(format: "%.1f", student.average)) Grade: \(student.grade)")
        }
        print("Class Average: \(String(format: "%.1f", classAverage()))")
    }
}

// การใช้งาน
let manager = StudentManager()
manager.addStudent(Student(id: 1, name: "สมชาย", scores: [85, 90, 78, 92]))
manager.addStudent(Student(id: 2, name: "สมหญิง", scores: [92, 88, 95, 91]))
manager.addStudent(Student(id: 3, name: "สมศรี", scores: [70, 75, 68, 72]))
manager.addStudent(Student(id: 4, name: "สมหมาย", scores: [55, 60, 58, 62]))
manager.addStudent(Student(id: 5, name: "สมใจ", scores: [88, 82, 85, 90]))

manager.printReport()
print("\nTop 3 Students:")
manager.topStudents(count: 3).forEach { print("  \($0.name): \(String(format: "%.1f", $0.average))") }
```

### ตัวอย่างที่ 2: ระบบจัดการสินค้าคงคลัง

```swift
struct Product {
    let sku: String
    let name: String
    var price: Double
    var quantity: Int
    let category: String
    
    var totalValue: Double { price * Double(quantity) }
}

class Inventory {
    private var products: [Product] = []
    
    func addProduct(_ product: Product) {
        products.append(product)
    }
    
    func updateQuantity(sku: String, by amount: Int) {
        if let index = products.firstIndex(where: { $0.sku == sku }) {
            products[index].quantity += amount
        }
    }
    
    func lowStockItems(threshold: Int) -> [Product] {
        return products.filter { $0.quantity < threshold }
    }
    
    func productsByCategory() -> [String: [Product]] {
        return Dictionary(grouping: products) { $0.category }
    }
    
    func totalInventoryValue() -> Double {
        return products.reduce(0) { $0 + $1.totalValue }
    }
    
    func mostValuableProducts(count: Int) -> [Product] {
        return products.sorted { $0.totalValue > $1.totalValue }.prefix(count).map { $0 }
    }
    
    func searchProducts(containing keyword: String) -> [Product] {
        return products.filter {
            $0.name.lowercased().contains(keyword.lowercased()) ||
            $0.sku.lowercased().contains(keyword.lowercased())
        }
    }
}

// การใช้งาน
let inventory = Inventory()
inventory.addProduct(Product(sku: "A001", name: "Apple iPhone", price: 35000, quantity: 50, category: "Electronics"))
inventory.addProduct(Product(sku: "A002", name: "Samsung Galaxy", price: 28000, quantity: 30, category: "Electronics"))
inventory.addProduct(Product(sku: "B001", name: "Nike Shoes", price: 3500, quantity: 100, category: "Clothing"))
inventory.addProduct(Product(sku: "B002", name: "Adidas T-Shirt", price: 890, quantity: 5, category: "Clothing"))
inventory.addProduct(Product(sku: "C001", name: "Programming Book", price: 650, quantity: 200, category: "Books"))

print("=== Inventory Report ===")
print("Total Value: ฿\(String(format: "%.0f", inventory.totalInventoryValue()))")

print("\nLow Stock (< 10 units):")
inventory.lowStockItems(threshold: 10).forEach {
    print("  \($0.name): \($0.quantity) units")
}

print("\nProducts by Category:")
for (category, items) in inventory.productsByCategory() {
    print("  \(category): \(items.count) items")
}
```

### ตัวอย่างที่ 3: การวิเคราะห์ข้อมูลตัวเลข

```swift
struct Statistics {
    static func mean(_ data: [Double]) -> Double {
        guard !data.isEmpty else { return 0 }
        return data.reduce(0, +) / Double(data.count)
    }
    
    static func median(_ data: [Double]) -> Double {
        guard !data.isEmpty else { return 0 }
        let sorted = data.sorted()
        let mid = sorted.count / 2
        return sorted.count % 2 == 0
            ? (sorted[mid - 1] + sorted[mid]) / 2
            : sorted[mid]
    }
    
    static func mode(_ data: [Double]) -> [Double] {
        let counts = data.reduce(into: [:]) { dict, value in
            dict[value, default: 0] += 1
        }
        let maxCount = counts.values.max() ?? 0
        return counts.filter { $0.value == maxCount }.map { $0.key }.sorted()
    }
    
    static func variance(_ data: [Double]) -> Double {
        let m = mean(data)
        return data.map { pow($0 - m, 2) }.reduce(0, +) / Double(data.count)
    }
    
    static func standardDeviation(_ data: [Double]) -> Double {
        return sqrt(variance(data))
    }
    
    static func range(_ data: [Double]) -> Double {
        guard let min = data.min(), let max = data.max() else { return 0 }
        return max - min
    }
}

let temperatures = [25.0, 28.0, 30.0, 27.0, 29.0, 31.0, 26.0, 28.0, 30.0, 29.0]

print("=== Temperature Statistics ===")
print("Mean: \(String(format: "%.2f", Statistics.mean(temperatures)))°C")
print("Median: \(String(format: "%.2f", Statistics.median(temperatures)))°C")
print("Mode: \(Statistics.mode(temperatures))°C")
print("Std Dev: \(String(format: "%.2f", Statistics.standardDeviation(temperatures)))°C")
print("Range: \(String(format: "%.2f", Statistics.range(temperatures)))°C")
print("Min: \(temperatures.min()!)°C")
print("Max: \(temperatures.max()!)°C")
```

---

## 9.22 แบบฝึกหัด (Practical Exercises)

### แบบฝึกหัดที่ 1: การดำเนินการพื้นฐาน

**โจทย์:** สร้าง function ที่รับ Array ของตัวเลข แล้วคืนค่า:
- ผลรวม, ค่าเฉลี่ย, ค่าสูงสุด, ค่าต่ำสุด

```swift
// เฉลย
func analyzeNumbers(_ numbers: [Int]) -> (sum: Int, average: Double, max: Int?, min: Int?) {
    let sum = numbers.reduce(0, +)
    let average = numbers.isEmpty ? 0.0 : Double(sum) / Double(numbers.count)
    return (sum, average, numbers.max(), numbers.min())
}

let testNumbers = [5, 3, 8, 1, 9, 2, 7, 4, 6]
let result = analyzeNumbers(testNumbers)
print("Sum: \(result.sum)")         // 45
print("Average: \(result.average)") // 5.0
print("Max: \(result.max!)")        // 9
print("Min: \(result.min!)")        // 1
```

### แบบฝึกหัดที่ 2: การจัดการข้อมูล

**โจทย์:** สร้าง function `removeDuplicates` ที่ลบค่าซ้ำออกโดยรักษาลำดับ

```swift
// เฉลย
func removeDuplicates<T: Hashable>(_ array: [T]) -> [T] {
    var seen = Set<T>()
    return array.filter { seen.insert($0).inserted }
}

let withDups = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5]
let noDups = removeDuplicates(withDups)
print(noDups) // [3, 1, 4, 5, 9, 2, 6]
```

### แบบฝึกหัดที่ 3: Two Sum Problem

**โจทย์:** หา index ของ 2 ตัวเลขที่บวกกันแล้วได้ค่าที่กำหนด

```swift
// เฉลย
func twoSum(_ nums: [Int], target: Int) -> (Int, Int)? {
    var seen: [Int: Int] = [:]
    for (index, num) in nums.enumerated() {
        let complement = target - num
        if let prevIndex = seen[complement] {
            return (prevIndex, index)
        }
        seen[num] = index
    }
    return nil
}

let numbers = [2, 7, 11, 15]
if let (i, j) = twoSum(numbers, target: 9) {
    print("Indices: \(i), \(j)") // Indices: 0, 1
    print("Values: \(numbers[i]), \(numbers[j])") // Values: 2, 7
}
```

### แบบฝึกหัดที่ 4: Matrix Operations

**โจทย์:** คำนวณผลคูณเมทริกซ์ 2x2

```swift
// เฉลย
func matrixMultiply(_ a: [[Int]], _ b: [[Int]]) -> [[Int]]? {
    let rows = a.count
    let cols = b[0].count
    let inner = b.count
    
    guard a[0].count == inner else { return nil }
    
    var result = Array(repeating: Array(repeating: 0, count: cols), count: rows)
    
    for i in 0..<rows {
        for j in 0..<cols {
            for k in 0..<inner {
                result[i][j] += a[i][k] * b[k][j]
            }
        }
    }
    return result
}

let matA = [[1, 2], [3, 4]]
let matB = [[5, 6], [7, 8]]
if let product = matrixMultiply(matA, matB) {
    print(product) // [[19, 22], [43, 50]]
}
```

### แบบฝึกหัดที่ 5: Sliding Window

**โจทย์:** หาผลรวมสูงสุดของ subarray ที่มีความยาว k

```swift
// เฉลย
func maxSubarraySum(_ arr: [Int], k: Int) -> Int {
    guard arr.count >= k else { return 0 }
    
    var windowSum = arr.prefix(k).reduce(0, +)
    var maxSum = windowSum
    
    for i in k..<arr.count {
        windowSum += arr[i] - arr[i - k]
        maxSum = max(maxSum, windowSum)
    }
    
    return maxSum
}

let arr = [2, 1, 5, 1, 3, 2]
print(maxSubarraySum(arr, k: 3)) // 9 (5+1+3)
```

### แบบฝึกหัดที่ 6: Merge Sort

**โจทย์:** Implement Merge Sort บน Array

```swift
// เฉลย
func mergeSort(_ array: [Int]) -> [Int] {
    guard array.count > 1 else { return array }
    
    let mid = array.count / 2
    let left = mergeSort(Array(array[..<mid]))
    let right = mergeSort(Array(array[mid...]))
    
    return merge(left, right)
}

func merge(_ left: [Int], _ right: [Int]) -> [Int] {
    var result: [Int] = []
    var l = 0, r = 0
    
    while l < left.count && r < right.count {
        if left[l] <= right[r] {
            result.append(left[l])
            l += 1
        } else {
            result.append(right[r])
            r += 1
        }
    }
    
    result += left[l...]
    result += right[r...]
    return result
}

let unsorted = [38, 27, 43, 3, 9, 82, 10]
let sorted = mergeSort(unsorted)
print(sorted) // [3, 9, 10, 27, 38, 43, 82]
```

---

## 9.23 สรุป (Summary)

### สิ่งที่ได้เรียนรู้

1. **Array Basics**: การสร้าง Array ด้วยหลายวิธี ทั้ง literal, initializer, และ Array(repeating:count:)

2. **การเข้าถึงข้อมูล**: ใช้ index, first, last, และ range subscript

3. **การแก้ไข Array**: append, insert, remove พร้อมวิธีการต่างๆ

4. **การวนซ้ำ**: for-in, forEach, enumerated(), reversed()

5. **Higher-Order Functions**: filter, map, reduce, compactMap, flatMap

6. **การเรียงลำดับ**: sorted(), sort() พร้อม custom comparator

7. **Value Semantics**: Array เป็น value type ใช้ Copy-on-Write

8. **Performance**: reserveCapacity, lazy evaluation, ContiguousArray

9. **ArraySlice**: มุมมองของ array ที่ไม่ copy ข้อมูล

### Best Practices

```swift
// 1. ใช้ let สำหรับ array ที่ไม่ต้องแก้ไข
let immutable = [1, 2, 3]

// 2. ใช้ isEmpty แทนการตรวจ count == 0
if myArray.isEmpty { print("Empty") }

// 3. ใช้ first/last แทนการ access ด้วย index
if let first = myArray.first { print(first) }

// 4. reserveCapacity เมื่อรู้จำนวนล่วงหน้า
var big = [Int]()
big.reserveCapacity(10000)

// 5. ใช้ lazy สำหรับ chain operations บน array ขนาดใหญ่
let result = bigArray.lazy.filter { ... }.map { ... }.first

// 6. ใช้ compactMap แทน map + filter(nil)
let valid = strings.compactMap { Int($0) }
```

### Quick Reference

| Operation | Code | Complexity |
|-----------|------|-----------|
| Create | `[Int]()` | O(1) |
| Append | `.append(x)` | O(1) amortized |
| Insert | `.insert(x, at:)` | O(n) |
| Access | `arr[i]` | O(1) |
| Search | `.firstIndex(of:)` | O(n) |
| Sort | `.sorted()` | O(n log n) |
| Filter | `.filter { }` | O(n) |
| Map | `.map { }` | O(n) |
| Reduce | `.reduce(0, +)` | O(n) |

---

*จบบทที่ 9: Arrays ใน Swift*
