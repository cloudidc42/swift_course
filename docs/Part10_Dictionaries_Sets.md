# ส่วนที่ 10: Dictionaries และ Sets ใน Swift

## บทนำ (Introduction)

ใน Swift มี collection types หลักสามประเภท:
- **Array**: เก็บข้อมูลแบบมีลำดับ
- **Dictionary**: เก็บข้อมูลแบบ key-value pairs
- **Set**: เก็บข้อมูลแบบไม่มีลำดับ ไม่มีค่าซ้ำ

ทั้ง Dictionary และ Set เป็น **value types** เช่นเดียวกับ Array และใช้ **Copy-on-Write** optimization

---

## 10.1 Dictionary Declaration and Initialization

Dictionary คือ collection ที่เก็บข้อมูลในรูปแบบ **key-value pairs** โดย key ต้องเป็น unique (ไม่ซ้ำกัน) และต้องเป็น type ที่ implement `Hashable`

### การประกาศพื้นฐาน

```swift
// วิธีที่ 1: ระบุ type อย่างชัดเจน
var dict1: Dictionary<String, Int> = Dictionary<String, Int>()

// วิธีที่ 2: ใช้ shorthand syntax (แนะนำ)
var dict2: [String: Int] = [String: Int]()

// วิธีที่ 3: ใช้ type inference
var dict3 = [String: Int]()

// วิธีที่ 4: สร้าง immutable dictionary
let fixedDict: [String: Double] = ["pi": 3.14159, "e": 2.71828]

// ตรวจสอบ type
print(type(of: dict2)) // Dictionary<String, Int>
```

### ประเภทของ Key และ Value

```swift
// Key ต้องเป็น Hashable
var intToString: [Int: String] = [:]
var stringToArray: [String: [Int]] = [:]
var enumToValue: [Character: Int] = [:]

// Nested Dictionary
var nestedDict: [String: [String: Int]] = [:]

// Dictionary ที่มี Optional value
var optionalValues: [String: Int?] = [:]
```

---

## 10.2 Dictionary Literals

```swift
// Dictionary literal พื้นฐาน
let scores: [String: Int] = [
    "Alice": 95,
    "Bob": 87,
    "Charlie": 92,
    "Diana": 88
]

// Type inference
let capitals = [
    "Thailand": "Bangkok",
    "Japan": "Tokyo",
    "France": "Paris",
    "Germany": "Berlin",
    "Australia": "Canberra"
]

// Dictionary ที่มีค่า Array
let courseGrades = [
    "Mathematics": [85, 90, 78, 92],
    "Science": [88, 72, 95, 83],
    "English": [90, 88, 91, 87]
]

// Empty Dictionary Literals
var emptyDict: [String: Int] = [:]
var anotherEmpty = [Int: String]()
```

---

## 10.3 การเข้าถึงค่าใน Dictionary (Accessing Dictionary Values)

### การเข้าถึงด้วย Subscript

```swift
let scores = ["Alice": 95, "Bob": 87, "Charlie": 92]

// การเข้าถึงด้วย key (คืนค่า Optional)
let aliceScore = scores["Alice"]
print(aliceScore)  // Optional(95)
print(type(of: aliceScore)) // Optional<Int>

// Unwrap ด้วย if let
if let score = scores["Alice"] {
    print("Alice's score: \(score)") // Alice's score: 95
}

// Unwrap ด้วย nil coalescing
let bobScore = scores["Bob"] ?? 0
print("Bob's score: \(bobScore)") // Bob's score: 87

// Key ที่ไม่มีอยู่จะได้ nil
let unknownScore = scores["Unknown"]
print(unknownScore) // nil
```

### ค่า Default

```swift
let wordCount = ["apple": 3, "banana": 2, "cherry": 1]

// ใช้ default value ถ้า key ไม่มีอยู่
let count = wordCount["mango", default: 0]
print(count) // 0

// ประโยชน์สำหรับการนับ
var counter = [String: Int]()
let words = ["apple", "banana", "apple", "cherry", "banana", "apple"]

for word in words {
    counter[word, default: 0] += 1
}
print(counter) // ["apple": 3, "banana": 2, "cherry": 1]
```

---

## 10.4 การแก้ไข Dictionary (Modifying Dictionaries)

### การเพิ่มและอัปเดต

```swift
var population: [String: Int] = ["Bangkok": 10_000_000]

// เพิ่ม key-value pair ใหม่
population["Chiang Mai"] = 1_800_000
population["Phuket"] = 400_000
print(population)

// อัปเดตค่าที่มีอยู่
population["Bangkok"] = 10_500_000

// updateValue(_:forKey:) - คืนค่าเก่า (Optional)
let oldValue = population.updateValue(2_000_000, forKey: "Chiang Mai")
print("Old value: \(oldValue ?? 0)") // Old value: 1800000

// เพิ่มค่าจาก Dictionary อื่น
let moreCities: [String: Int] = ["Khon Kaen": 300_000, "Nakhon Ratchasima": 500_000]
population.merge(moreCities) { current, new in new }
print(population.count) // 5
```

### การลบ

```swift
var inventory = ["iPhone": 50, "iPad": 30, "MacBook": 20, "AirPods": 100]

// ลบด้วย key (subscript = nil)
inventory["AirPods"] = nil
print(inventory.count) // 3

// removeValue(forKey:) - คืนค่าที่ถูกลบ (Optional)
if let removed = inventory.removeValue(forKey: "iPad") {
    print("Removed iPad with quantity: \(removed)") // 30
}

// ลบทั้งหมด
inventory.removeAll()
print(inventory.isEmpty) // true

// ลบตามเงื่อนไข
var prices = ["Apple": 100.0, "Banana": 25.0, "Cherry": 200.0, "Durian": 300.0]
prices = prices.filter { $0.value < 200 }
print(prices) // ["Apple": 100.0, "Banana": 25.0]
```

---

## 10.5 Dictionary Properties

```swift
let data = ["name": "Alice", "city": "Bangkok", "country": "Thailand"]

// count - จำนวน key-value pairs
print(data.count) // 3

// isEmpty - ตรวจสอบว่าว่างหรือไม่
print(data.isEmpty) // false
print([:].isEmpty)  // true

// keys - collection ของ keys ทั้งหมด
print(data.keys)  // ["name", "city", "country"] (ลำดับไม่แน่นอน)

// values - collection ของ values ทั้งหมด
print(data.values) // ["Alice", "Bangkok", "Thailand"] (ลำดับไม่แน่นอน)

// แปลง keys/values เป็น Array
let keyArray = Array(data.keys).sorted()
let valueArray = Array(data.values)
print(keyArray) // ["city", "country", "name"]

// ตรวจสอบว่ามี key หรือไม่
print(data.keys.contains("name"))    // true
print(data.keys.contains("email"))   // false
```

---

## 10.6 การวนซ้ำผ่าน Dictionary (Iterating Over Dictionaries)

```swift
let scores = ["Alice": 95, "Bob": 87, "Charlie": 92, "Diana": 88]

// วนผ่าน key-value pairs
for (name, score) in scores {
    print("\(name): \(score)")
}

// วนผ่านเฉพาะ keys
for name in scores.keys {
    print(name)
}

// วนผ่านเฉพาะ values
for score in scores.values {
    print(score)
}

// วนผ่านแบบเรียงลำดับ
for name in scores.keys.sorted() {
    print("\(name): \(scores[name]!)")
}
// Alice: 95
// Bob: 87
// Charlie: 92
// Diana: 88

// forEach
scores.forEach { name, score in
    print("\(name) got \(score) points")
}

// enumerated() (เพิ่ม index)
for (index, pair) in scores.enumerated() {
    print("\(index + 1). \(pair.key): \(pair.value)")
}
```

---

## 10.7 Dictionary Methods (filter, map, mapValues)

### filter

```swift
let scores = ["Alice": 95, "Bob": 67, "Charlie": 82, "Diana": 45, "Eve": 91]

// กรอง key-value pairs ตามเงื่อนไข
let passing = scores.filter { $0.value >= 75 }
print(passing) // ["Alice": 95, "Charlie": 82, "Eve": 91]

// กรองตาม key
let aNames = scores.filter { $0.key.hasPrefix("A") }
print(aNames) // ["Alice": 95]
```

### map และ mapValues

```swift
let prices: [String: Double] = ["Apple": 100.0, "Banana": 25.0, "Cherry": 200.0]

// map คืนค่าเป็น Array ของ tuples
let mappedArray = prices.map { key, value in
    return "\(key): ฿\(value)"
}
print(mappedArray)

// mapValues - แปลงเฉพาะ values (คืนค่าเป็น Dictionary)
let discounted = prices.mapValues { $0 * 0.9 }
print(discounted) // ["Apple": 90.0, "Banana": 22.5, "Cherry": 180.0]

// แปลง values เป็น String
let stringPrices = prices.mapValues { "฿\(String(format: "%.2f", $0))" }
print(stringPrices) // ["Apple": "฿100.00", "Banana": "฿25.00", "Cherry": "฿200.00"]

// mapKeys (ไม่มี built-in แต่สร้างเองได้)
let uppercasedKeys = Dictionary(uniqueKeysWithValues: prices.map { ($0.key.uppercased(), $0.value) })
print(uppercasedKeys)
```

### compactMapValues

```swift
let data = ["a": "1", "b": "two", "c": "3", "d": "four", "e": "5"]

// กรองค่าที่แปลงไม่สำเร็จออก
let validInts = data.compactMapValues { Int($0) }
print(validInts) // ["a": 1, "c": 3, "e": 5]
```

### reduce

```swift
let itemCounts = ["Apples": 10, "Bananas": 5, "Cherries": 20, "Dates": 8]

// หาผลรวม values
let totalItems = itemCounts.values.reduce(0, +)
print(totalItems) // 43

// หาค่าสูงสุด
let maxItems = itemCounts.values.max() ?? 0
print(maxItems) // 20

// สร้าง Dictionary ใหม่
let doubled = itemCounts.reduce(into: [:]) { result, pair in
    result[pair.key] = pair.value * 2
}
print(doubled)
```

---

## 10.8 Nested Dictionaries (Dictionary ซ้อนกัน)

```swift
// ตัวอย่าง: ข้อมูลนักเรียนในหลายห้อง
var schoolData: [String: [String: Int]] = [
    "ม.4/1": ["สมชาย": 85, "สมหญิง": 90, "สมศรี": 78],
    "ม.4/2": ["สมหมาย": 92, "สมใจ": 88, "สมศักดิ์": 75],
    "ม.4/3": ["สมปอง": 70, "สมบุญ": 95, "สมพร": 83]
]

// เข้าถึง nested value
if let classData = schoolData["ม.4/1"],
   let score = classData["สมชาย"] {
    print("สมชาย's score: \(score)") // 85
}

// เพิ่มนักเรียนใหม่
schoolData["ม.4/1"]?["สมวงค์"] = 88

// วนซ้ำผ่าน nested dictionary
for (className, students) in schoolData.sorted(by: { $0.key < $1.key }) {
    print("\n\(className):")
    for (name, score) in students.sorted(by: { $0.key < $1.key }) {
        print("  \(name): \(score)")
    }
}

// ตัวอย่างที่ซับซ้อนขึ้น: JSON-like structure
var config: [String: Any] = [
    "server": [
        "host": "localhost",
        "port": 8080
    ] as [String: Any],
    "database": [
        "name": "mydb",
        "maxConnections": 10
    ] as [String: Any],
    "debug": true
]

if let serverConfig = config["server"] as? [String: Any],
   let host = serverConfig["host"] as? String,
   let port = serverConfig["port"] as? Int {
    print("Server: \(host):\(port)")
}
```

---

## 10.9 Dictionary Merging

```swift
var dict1 = ["a": 1, "b": 2, "c": 3]
let dict2 = ["b": 20, "c": 30, "d": 40]

// merge(_:uniquingKeysWith:) - แก้ไข in-place
dict1.merge(dict2) { current, new in
    current + new  // เมื่อ key ซ้ำให้บวกค่ากัน
}
print(dict1) // ["a": 1, "b": 22, "c": 33, "d": 40]

// merging(_:uniquingKeysWith:) - คืนค่า Dictionary ใหม่
var base = ["x": 1, "y": 2]
let extra = ["y": 20, "z": 30]
let merged = base.merging(extra) { current, _ in current } // เก็บค่าเดิม
print(merged) // ["x": 1, "y": 2, "z": 30]

let overwritten = base.merging(extra) { _, new in new } // ใช้ค่าใหม่
print(overwritten) // ["x": 1, "y": 20, "z": 30]

// รวมหลาย dictionaries
func mergeDictionaries<K: Hashable, V>(_ dicts: [[K: V]],
                                       with strategy: (V, V) -> V) -> [K: V] {
    var result = [K: V]()
    for dict in dicts {
        result.merge(dict, uniquingKeysWith: strategy)
    }
    return result
}

let d1 = ["a": 1, "b": 2]
let d2 = ["b": 3, "c": 4]
let d3 = ["c": 5, "d": 6]
let combined = mergeDictionaries([d1, d2, d3]) { $0 + $1 }
print(combined) // ["a": 1, "b": 5, "c": 9, "d": 6]
```

---

## 10.10 Set Declaration and Initialization

Set คือ collection ที่เก็บค่า **unique** (ไม่ซ้ำ) และ**ไม่มีลำดับ** (unordered) ต้องการ element ที่ implement `Hashable`

### การสร้าง Set

```swift
// วิธีที่ 1: ระบุ type อย่างชัดเจน
var set1: Set<Int> = Set<Int>()
var set2: Set<String> = []

// วิธีที่ 2: สร้างจาก array literal
var fruits: Set<String> = ["Apple", "Banana", "Cherry"]
var numbers: Set = [1, 2, 3, 4, 5] // type inference

// สำคัญ: ต้องระบุ type เพราะไม่งั้น Swift จะสร้างเป็น Array
let ambiguous = ["Apple", "Banana"] // นี่คือ Array!
let asSet: Set = ["Apple", "Banana"] // นี่คือ Set

// สร้างจาก Array
let array = [1, 2, 3, 2, 4, 3, 5]
let fromArray = Set(array)
print(fromArray) // {1, 2, 3, 4, 5} (ลำดับไม่แน่นอน)

// ตรวจสอบ
print(type(of: fromArray)) // Set<Int>
```

### Set กับ Custom Types

```swift
struct Point: Hashable {
    let x: Int
    let y: Int
}

var points: Set<Point> = []
points.insert(Point(x: 1, y: 2))
points.insert(Point(x: 3, y: 4))
points.insert(Point(x: 1, y: 2)) // ซ้ำ จะไม่ถูกเพิ่ม

print(points.count) // 2
```

---

## 10.11 Set Operations

### Union (การรวม)

```swift
let setA: Set<Int> = [1, 2, 3, 4, 5]
let setB: Set<Int> = [3, 4, 5, 6, 7]

// union - รวม elements ทั้งสอง Set (ไม่ซ้ำ)
let unionSet = setA.union(setB)
print(unionSet) // {1, 2, 3, 4, 5, 6, 7} (ลำดับไม่แน่นอน)

// formUnion - แก้ไข in-place
var mutableSet: Set<Int> = [1, 2, 3]
mutableSet.formUnion([3, 4, 5])
print(mutableSet) // {1, 2, 3, 4, 5}
```

### Intersection (ส่วนที่ตัดกัน)

```swift
let setA: Set<Int> = [1, 2, 3, 4, 5]
let setB: Set<Int> = [3, 4, 5, 6, 7]

// intersection - elements ที่มีอยู่ใน set ทั้งสอง
let common = setA.intersection(setB)
print(common) // {3, 4, 5}

// formIntersection - แก้ไข in-place
var set1: Set<String> = ["Alice", "Bob", "Charlie"]
set1.formIntersection(["Bob", "Charlie", "Diana"])
print(set1) // {"Bob", "Charlie"}
```

### Subtract (การลบ)

```swift
let setA: Set<Int> = [1, 2, 3, 4, 5]
let setB: Set<Int> = [3, 4, 5, 6, 7]

// subtracting - elements ใน A แต่ไม่ใน B
let onlyInA = setA.subtracting(setB)
print(onlyInA) // {1, 2}

// subtract - แก้ไข in-place
var mySet: Set<Int> = [1, 2, 3, 4, 5]
mySet.subtract([3, 4, 5])
print(mySet) // {1, 2}
```

### Symmetric Difference (ส่วนต่าง)

```swift
let setA: Set<Int> = [1, 2, 3, 4, 5]
let setB: Set<Int> = [3, 4, 5, 6, 7]

// symmetricDifference - elements ที่อยู่ใน set ใด set หนึ่งแต่ไม่อยู่ในทั้งสอง
let exclusive = setA.symmetricDifference(setB)
print(exclusive) // {1, 2, 6, 7}

// formSymmetricDifference - แก้ไข in-place
var set1: Set<Int> = [1, 2, 3, 4, 5]
set1.formSymmetricDifference([3, 4, 5, 6, 7])
print(set1) // {1, 2, 6, 7}
```

### ตัวอย่างการใช้ Set Operations

```swift
// Venn Diagram: นักเรียนที่เรียนวิชาต่างๆ
let mathStudents: Set<String> = ["Alice", "Bob", "Charlie", "Diana", "Eve"]
let scienceStudents: Set<String> = ["Bob", "Charlie", "Frank", "Grace"]
let artStudents: Set<String> = ["Alice", "Diana", "Frank", "Henry"]

// นักเรียนที่เรียนทั้ง Math และ Science
let mathAndScience = mathStudents.intersection(scienceStudents)
print("Math & Science: \(mathAndScience)")

// นักเรียนทั้งหมด
let allStudents = mathStudents.union(scienceStudents).union(artStudents)
print("All students: \(allStudents.count)")

// นักเรียนที่เรียนเฉพาะ Math ไม่เรียน Science
let onlyMath = mathStudents.subtracting(scienceStudents)
print("Only Math: \(onlyMath)")

// นักเรียนที่เรียนทั้งสามวิชา
let allThree = mathStudents.intersection(scienceStudents).intersection(artStudents)
print("All three subjects: \(allThree)")
```

---

## 10.12 Set Membership Testing

```swift
let primes: Set<Int> = [2, 3, 5, 7, 11, 13, 17, 19, 23, 29]

// contains
print(primes.contains(7))   // true
print(primes.contains(10))  // false

// isSubset(of:) - ตรวจสอบว่าเป็น subset หรือไม่
let small: Set<Int> = [2, 3, 5]
print(small.isSubset(of: primes))     // true
print(primes.isSubset(of: small))     // false

// isStrictSubset(of:) - subset แต่ไม่เท่ากัน
print(small.isStrictSubset(of: primes)) // true
print(primes.isStrictSubset(of: primes)) // false

// isSuperset(of:) - ตรวจสอบว่าเป็น superset หรือไม่
print(primes.isSuperset(of: small))  // true
print(small.isSuperset(of: primes))  // false

// isDisjoint(with:) - ไม่มี element ร่วมกัน
let evens: Set<Int> = [2, 4, 6, 8, 10]
let odds: Set<Int> = [1, 3, 5, 7, 9]
print(evens.isDisjoint(with: odds))      // true
print(evens.isDisjoint(with: primes))    // false (มี 2 ร่วมกัน)

// ความเท่ากัน
let set1: Set<Int> = [1, 2, 3]
let set2: Set<Int> = [3, 1, 2]
print(set1 == set2) // true (ลำดับไม่สำคัญ)
```

---

## 10.13 การแก้ไข Set (Modifying Sets)

```swift
var techSkills: Set<String> = ["Swift", "Python", "JavaScript"]

// insert - เพิ่ม element (คืน tuple: (inserted: Bool, memberAfterInsert: Element))
let result1 = techSkills.insert("Kotlin")
print(result1.inserted)           // true
print(result1.memberAfterInsert)  // Kotlin

let result2 = techSkills.insert("Swift") // ซ้ำ
print(result2.inserted)           // false (ไม่ถูกเพิ่ม)

// update(with:) - เพิ่มหรืออัปเดต
let old = techSkills.update(with: "Swift")
print(old) // Optional("Swift") (ค่าเก่า)

// remove - ลบ element (คืนค่าที่ถูกลบ หรือ nil)
if let removed = techSkills.remove("Python") {
    print("Removed: \(removed)")
}

// removeAll
techSkills.removeAll()
print(techSkills.isEmpty) // true

// filter ใช้ได้กับ Set
var numbers: Set<Int> = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
let evenNumbers = numbers.filter { $0 % 2 == 0 }
print(evenNumbers) // {2, 4, 6, 8, 10}
```

---

## 10.14 การวนซ้ำผ่าน Set (Iterating Over Sets)

```swift
let colors: Set<String> = ["Red", "Green", "Blue", "Yellow", "Purple"]

// for-in loop (ลำดับไม่แน่นอน)
for color in colors {
    print(color)
}

// วนแบบมีลำดับ (sort ก่อน)
for color in colors.sorted() {
    print(color)
}
// Blue, Green, Purple, Red, Yellow

// forEach
colors.forEach { print($0) }

// map, filter, reduce ทำงานได้เหมือน Array
let lengths = colors.map { $0.count }
print(lengths) // [3, 5, 4, 6, 6] (ลำดับไม่แน่นอน)

let longColors = colors.filter { $0.count > 4 }
print(longColors) // {"Green", "Yellow", "Purple"}
```

---

## 10.15 การแปลงระหว่าง Set และ Array

```swift
// Array → Set (ลบซ้ำ)
let arrayWithDups = [1, 2, 3, 2, 4, 3, 5, 1]
let uniqueSet = Set(arrayWithDups)
print(uniqueSet) // {1, 2, 3, 4, 5}

// Set → Array
let set: Set<String> = ["Apple", "Banana", "Cherry"]
let array = Array(set)
print(array) // ลำดับไม่แน่นอน

// Set → Sorted Array
let sortedArray = set.sorted()
print(sortedArray) // ["Apple", "Banana", "Cherry"]

// ประโยชน์: ลบซ้ำแล้วรักษาลำดับ
let original = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3]
var seen = Set<Int>()
let deduplicated = original.filter { seen.insert($0).inserted }
print(deduplicated) // [3, 1, 4, 5, 9, 2, 6]

// แปลง String เป็น Set ของ Characters
let text = "hello world"
let uniqueChars = Set(text)
print(uniqueChars.sorted()) // [" ", "d", "e", "h", "l", "o", "r", "w"]
print(uniqueChars.count) // 8 (ไม่นับซ้ำ)
```

---

## 10.16 OrderedSet และ OrderedDictionary (Swift Collections)

Swift Collections package มี `OrderedSet` และ `OrderedDictionary` ที่รักษาลำดับการแทรก

```swift
// ติดตั้ง: Swift Package Manager
// https://github.com/apple/swift-collections
// import OrderedCollections

// OrderedSet - Set ที่รักษาลำดับ
// var orderedSet = OrderedSet<Int>()
// orderedSet.append(3)
// orderedSet.append(1)
// orderedSet.append(4)
// orderedSet.append(1) // ซ้ำ ไม่ถูกเพิ่ม
// print(orderedSet) // [3, 1, 4]

// ใช้ workaround กับ Swift standard library
struct OrderedSetSim<T: Hashable> {
    private var set: Set<T> = []
    private var array: [T] = []
    
    mutating func insert(_ element: T) {
        if set.insert(element).inserted {
            array.append(element)
        }
    }
    
    mutating func remove(_ element: T) {
        if set.remove(element) != nil {
            array.removeAll { $0 == element }
        }
    }
    
    var count: Int { array.count }
    var isEmpty: Bool { array.isEmpty }
    
    func contains(_ element: T) -> Bool { set.contains(element) }
    
    var elements: [T] { array }
}

var orderedColors = OrderedSetSim<String>()
orderedColors.insert("Red")
orderedColors.insert("Green")
orderedColors.insert("Blue")
orderedColors.insert("Red") // ซ้ำ ไม่ถูกเพิ่ม

print(orderedColors.elements) // ["Red", "Green", "Blue"] (รักษาลำดับ)
print(orderedColors.contains("Green")) // true
```

---

## 10.17 เมื่อไหรควรใช้อะไร (When to Use Dictionary vs Array vs Set)

### Array ควรใช้เมื่อ:

```swift
// 1. ลำดับสำคัญ
let playlist = ["Song A", "Song B", "Song C"]

// 2. อนุญาตให้มีค่าซ้ำ
let scores = [85, 90, 85, 78, 90] // นักเรียนได้คะแนนเท่ากันได้

// 3. ต้องการเข้าถึงด้วย index
let grid = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]

// 4. ต้องการประสิทธิภาพดีสำหรับการเข้าถึงตามลำดับ
let fibonacci = [0, 1, 1, 2, 3, 5, 8, 13]
```

### Dictionary ควรใช้เมื่อ:

```swift
// 1. ต้องการค้นหาด้วย key ที่มีความหมาย
let capitals = ["Thailand": "Bangkok", "Japan": "Tokyo"]

// 2. ต้องการ fast lookup (O(1))
let phoneBook = ["Alice": "02-111-1111", "Bob": "02-222-2222"]

// 3. ข้อมูลมีลักษณะ key-value
let userProfile = ["name": "Alice", "age": "25", "city": "Bangkok"]

// 4. การนับหรือ grouping
let wordFrequency = ["swift": 10, "programming": 5, "apple": 8]
```

### Set ควรใช้เมื่อ:

```swift
// 1. ต้องการตรวจสอบสมาชิกอย่างรวดเร็ว (O(1))
let allowedCountries: Set<String> = ["Thailand", "Japan", "Korea"]
if allowedCountries.contains(userCountry) { /* ... */ }

// 2. ต้องการลบค่าซ้ำ
let uniqueVisitors = Set(allVisitors)

// 3. ต้องการ set operations (union, intersection, etc.)
let premiumUsers: Set<String> = ["Alice", "Charlie"]
let activeUsers: Set<String> = ["Alice", "Bob", "Diana"]
let premiumAndActive = premiumUsers.intersection(activeUsers)
```

### สรุปการเปรียบเทียบ

| Feature | Array | Dictionary | Set |
|---------|-------|-----------|-----|
| ลำดับ | มี | ไม่มี (Swift 5.7+) | ไม่มี |
| ค่าซ้ำ | ได้ | Keys ไม่ได้ | ไม่ได้ |
| Access | O(1) by index | O(1) by key | N/A |
| Search | O(n) | O(1) | O(1) |
| Insert | O(1) amortized | O(1) | O(1) |
| Memory | น้อยกว่า | มากกว่า | ปานกลาง |

---

## 10.18 Performance Comparison (การเปรียบเทียบประสิทธิภาพ)

```swift
import Foundation

let testSize = 100_000

// สร้างข้อมูลทดสอบ
let testArray = Array(0..<testSize)
let testSet = Set(0..<testSize)
let testDict = Dictionary(uniqueKeysWithValues: (0..<testSize).map { ($0, $0) })

// ค้นหาใน Array: O(n)
let searchValue = testSize - 1

// ค้นหาใน Set: O(1)
let setContains = testSet.contains(searchValue) // เร็วมาก!

// ค้นหาใน Dictionary: O(1)
let dictValue = testDict[searchValue] // เร็วมาก!

// สรุปความซับซ้อน
/*
Operation        | Array | Dictionary | Set
----------------|-------|-----------|----
Contains/Lookup | O(n)  | O(1)      | O(1)
Insert          | O(1)* | O(1)      | O(1)
Delete          | O(n)  | O(1)      | O(1)
Sorted iteration| O(1)  | O(n log n)| O(n log n)

*) amortized, worst case O(n) when resizing
*/
```

---

## 10.19 Common Patterns (รูปแบบที่ใช้บ่อย)

### Grouping

```swift
struct Employee {
    let name: String
    let department: String
    let salary: Double
}

let employees = [
    Employee(name: "Alice", department: "Engineering", salary: 80000),
    Employee(name: "Bob", department: "Marketing", salary: 60000),
    Employee(name: "Charlie", department: "Engineering", salary: 90000),
    Employee(name: "Diana", department: "HR", salary: 55000),
    Employee(name: "Eve", department: "Marketing", salary: 65000)
]

// Grouping ด้วย Dictionary
let byDepartment = Dictionary(grouping: employees) { $0.department }

for (dept, emps) in byDepartment.sorted(by: { $0.key < $1.key }) {
    print("\(dept):")
    emps.forEach { print("  \($0.name): ฿\(String(format: "%.0f", $0.salary))") }
}
```

### Caching

```swift
// Simple Memoization Cache
class MemoCache<Key: Hashable, Value> {
    private var cache: [Key: Value] = [:]
    private let maxSize: Int
    
    init(maxSize: Int = 100) {
        self.maxSize = maxSize
    }
    
    func get(_ key: Key) -> Value? {
        return cache[key]
    }
    
    func set(_ key: Key, value: Value) {
        if cache.count >= maxSize {
            cache.removeValue(forKey: cache.keys.first!)
        }
        cache[key] = value
    }
}

// Fibonacci ด้วย memoization
var memo = [Int: Int]()

func fib(_ n: Int) -> Int {
    if n <= 1 { return n }
    if let cached = memo[n] { return cached }
    let result = fib(n - 1) + fib(n - 2)
    memo[n] = result
    return result
}

print(fib(40)) // 102334155 (รวดเร็วมาก!)
```

### Word Frequency Counter

```swift
func wordFrequency(_ text: String) -> [(word: String, count: Int)] {
    let words = text.lowercased()
        .components(separatedBy: .whitespacesAndNewlines)
        .map { $0.trimmingCharacters(in: .punctuationCharacters) }
        .filter { !$0.isEmpty }
    
    var frequency = [String: Int]()
    words.forEach { frequency[$0, default: 0] += 1 }
    
    return frequency.map { (word: $0.key, count: $0.value) }
        .sorted { $0.count > $1.count }
}

let text = "Swift is great. Swift is fast. Swift is safe. I love Swift programming."
let freq = wordFrequency(text)
freq.prefix(5).forEach { print("\($0.word): \($0.count)") }
// swift: 4
// is: 3
// great: 1
// fast: 1
// safe: 1
```

### Index Building

```swift
// สร้าง index สำหรับการค้นหาเร็ว
struct Article {
    let id: Int
    let title: String
    let tags: [String]
}

class ArticleIndex {
    private var articles: [Article] = []
    private var tagIndex: [String: Set<Int>] = [:]  // tag -> article IDs
    private var idIndex: [Int: Int] = [:]            // articleID -> array index
    
    func addArticle(_ article: Article) {
        let arrayIndex = articles.count
        articles.append(article)
        idIndex[article.id] = arrayIndex
        
        for tag in article.tags {
            tagIndex[tag, default: []].insert(article.id)
        }
    }
    
    func articles(withTag tag: String) -> [Article] {
        guard let ids = tagIndex[tag] else { return [] }
        return ids.compactMap { id in
            guard let index = idIndex[id] else { return nil }
            return articles[index]
        }
    }
    
    func article(withId id: Int) -> Article? {
        guard let index = idIndex[id] else { return nil }
        return articles[index]
    }
}

let index = ArticleIndex()
index.addArticle(Article(id: 1, title: "Swift Basics", tags: ["swift", "programming", "ios"]))
index.addArticle(Article(id: 2, title: "Python Tutorial", tags: ["python", "programming"]))
index.addArticle(Article(id: 3, title: "iOS Development", tags: ["swift", "ios", "mobile"]))
index.addArticle(Article(id: 4, title: "SwiftUI Guide", tags: ["swift", "ios", "ui"]))

let swiftArticles = index.articles(withTag: "swift")
print("Swift articles: \(swiftArticles.count)") // 3
swiftArticles.forEach { print("  \($0.title)") }
```

---

## 10.20 แบบฝึกหัด (Practical Exercises)

### แบบฝึกหัดที่ 1: Word Count

**โจทย์:** นับจำนวนคำในประโยค และแสดงคำที่พบบ่อยที่สุด 3 อันดับ

```swift
// เฉลย
func topWords(_ text: String, count: Int = 3) -> [(String, Int)] {
    let stopWords: Set<String> = ["a", "an", "the", "is", "are", "was", "were", "i", "it"]
    
    var frequency = [String: Int]()
    
    text.lowercased()
        .components(separatedBy: .whitespacesAndNewlines)
        .map { $0.trimmingCharacters(in: .punctuationCharacters) }
        .filter { !$0.isEmpty && !stopWords.contains($0) }
        .forEach { frequency[$0, default: 0] += 1 }
    
    return frequency
        .sorted { $0.value > $1.value }
        .prefix(count)
        .map { ($0.key, $0.value) }
}

let article = "Swift programming is great. Swift is fast and Swift is safe. I love Swift."
let top = topWords(article, count: 3)
top.forEach { print("\($0.0): \($0.1)") }
// swift: 4
// programming: 1
// great: 1
```

### แบบฝึกหัดที่ 2: Anagram Detection

**โจทย์:** ตรวจสอบว่าสอง String เป็น Anagram กันหรือไม่

```swift
// เฉลย
func isAnagram(_ s1: String, _ s2: String) -> Bool {
    let clean1 = s1.lowercased().filter { $0.isLetter }
    let clean2 = s2.lowercased().filter { $0.isLetter }
    
    guard clean1.count == clean2.count else { return false }
    
    var frequency = [Character: Int]()
    clean1.forEach { frequency[$0, default: 0] += 1 }
    clean2.forEach { frequency[$0, default: 0] -= 1 }
    
    return frequency.values.allSatisfy { $0 == 0 }
}

print(isAnagram("listen", "silent"))     // true
print(isAnagram("hello", "world"))      // false
print(isAnagram("Astronomer", "Moon starer")) // true
```

### แบบฝึกหัดที่ 3: Graph Adjacency List

**โจทย์:** สร้าง Graph ด้วย Dictionary และ Set แล้ว BFS

```swift
// เฉลย
struct Graph {
    var adjacencyList: [Int: Set<Int>] = [:]
    
    mutating func addEdge(from: Int, to: Int) {
        adjacencyList[from, default: []].insert(to)
        adjacencyList[to, default: []].insert(from) // undirected
    }
    
    func bfs(from start: Int) -> [Int] {
        var visited: Set<Int> = [start]
        var queue = [start]
        var result: [Int] = []
        
        while !queue.isEmpty {
            let node = queue.removeFirst()
            result.append(node)
            
            if let neighbors = adjacencyList[node] {
                for neighbor in neighbors.sorted() {
                    if !visited.contains(neighbor) {
                        visited.insert(neighbor)
                        queue.append(neighbor)
                    }
                }
            }
        }
        return result
    }
    
    func isConnected(from: Int, to: Int) -> Bool {
        guard adjacencyList[from] != nil else { return false }
        var visited: Set<Int> = []
        var stack = [from]
        
        while !stack.isEmpty {
            let node = stack.removeLast()
            if node == to { return true }
            if visited.contains(node) { continue }
            visited.insert(node)
            stack.append(contentsOf: adjacencyList[node] ?? [])
        }
        return false
    }
}

var graph = Graph()
graph.addEdge(from: 1, to: 2)
graph.addEdge(from: 1, to: 3)
graph.addEdge(from: 2, to: 4)
graph.addEdge(from: 3, to: 5)
graph.addEdge(from: 4, to: 5)

print("BFS from 1: \(graph.bfs(from: 1))")
print("Connected 1→5: \(graph.isConnected(from: 1, to: 5))")
print("Connected 2→3: \(graph.isConnected(from: 2, to: 3))")
```

### แบบฝึกหัดที่ 4: Inventory with Dictionary

**โจทย์:** ระบบจัดการสินค้าที่ใช้ Dictionary

```swift
// เฉลย
struct InventorySystem {
    private var stock: [String: (quantity: Int, price: Double)] = [:]
    
    mutating func addItem(name: String, quantity: Int, price: Double) {
        stock[name] = (quantity, price)
    }
    
    mutating func sell(name: String, quantity: Int) -> Bool {
        guard let item = stock[name], item.quantity >= quantity else {
            print("ไม่มีสินค้าเพียงพอ: \(name)")
            return false
        }
        stock[name]?.quantity -= quantity
        if stock[name]!.quantity == 0 {
            stock.removeValue(forKey: name)
        }
        return true
    }
    
    mutating func restock(name: String, quantity: Int) {
        stock[name]?.quantity += quantity
    }
    
    func totalValue() -> Double {
        stock.values.reduce(0) { $0 + Double($1.quantity) * $1.price }
    }
    
    func lowStockItems(threshold: Int) -> [String] {
        stock.filter { $0.value.quantity <= threshold }
            .map { $0.key }
            .sorted()
    }
    
    func printReport() {
        print("=== Inventory Report ===")
        stock.sorted { $0.key < $1.key }.forEach { name, info in
            print("  \(name): \(info.quantity) units @ ฿\(String(format: "%.2f", info.price))")
        }
        print("Total Value: ฿\(String(format: "%.2f", totalValue()))")
    }
}

var inv = InventorySystem()
inv.addItem(name: "iPhone 15", quantity: 50, price: 35000)
inv.addItem(name: "MacBook Pro", quantity: 20, price: 65000)
inv.addItem(name: "AirPods", quantity: 100, price: 7900)
inv.addItem(name: "iPad", quantity: 3, price: 25000)

print("Sold iPhone: \(inv.sell(name: "iPhone 15", quantity: 5))")
print("Low stock (≤5): \(inv.lowStockItems(threshold: 5))")
inv.printReport()
```

### แบบฝึกหัดที่ 5: Unique Items ด้วย Set

**โจทย์:** ใช้ Set ในการจัดการสิทธิ์ผู้ใช้

```swift
// เฉลย
enum Permission: String, Hashable {
    case read, write, delete, admin, export
}

struct User {
    let name: String
    var permissions: Set<Permission>
    
    mutating func grant(_ permission: Permission) {
        permissions.insert(permission)
    }
    
    mutating func revoke(_ permission: Permission) {
        permissions.remove(permission)
    }
    
    func canDo(_ permission: Permission) -> Bool {
        return permissions.contains(permission) || permissions.contains(.admin)
    }
    
    func sharedPermissions(with other: User) -> Set<Permission> {
        return permissions.intersection(other.permissions)
    }
    
    func hasAllPermissions(of role: Set<Permission>) -> Bool {
        return role.isSubset(of: permissions)
    }
}

var alice = User(name: "Alice", permissions: [.read, .write, .delete])
var bob = User(name: "Bob", permissions: [.read])
var admin = User(name: "Admin", permissions: [.admin])

alice.grant(.export)
bob.grant(.write)

print("Alice can delete: \(alice.canDo(.delete))")    // true
print("Bob can delete: \(bob.canDo(.delete))")        // false
print("Admin can delete: \(admin.canDo(.delete))")    // true (has .admin)

let shared = alice.sharedPermissions(with: bob)
print("Shared: \(shared.map { $0.rawValue }.sorted())")

let editorRole: Set<Permission> = [.read, .write]
print("Alice has editor permissions: \(alice.hasAllPermissions(of: editorRole))") // true
print("Bob has editor permissions: \(bob.hasAllPermissions(of: editorRole))")   // true
```

---

## 10.21 ตัวอย่างโปรแกรมจริง (Real-World Examples)

### ตัวอย่างที่ 1: Phone Book ด้วย Dictionary

```swift
class PhoneBook {
    private var contacts: [String: [String]] = [:] // name: [phone numbers]
    private var phoneToName: [String: String] = [:] // reverse lookup
    
    func addContact(name: String, phone: String) {
        contacts[name, default: []].append(phone)
        phoneToName[phone] = name
    }
    
    func removeContact(name: String) {
        if let phones = contacts[name] {
            phones.forEach { phoneToName.removeValue(forKey: $0) }
        }
        contacts.removeValue(forKey: name)
    }
    
    func lookup(name: String) -> [String]? {
        return contacts[name]
    }
    
    func lookupByPhone(_ phone: String) -> String? {
        return phoneToName[phone]
    }
    
    func search(prefix: String) -> [String] {
        return contacts.keys
            .filter { $0.lowercased().hasPrefix(prefix.lowercased()) }
            .sorted()
    }
    
    func printAll() {
        print("=== Phone Book ===")
        contacts.sorted { $0.key < $1.key }.forEach { name, phones in
            print("\(name): \(phones.joined(separator: ", "))")
        }
    }
}

let book = PhoneBook()
book.addContact(name: "สมชาย", phone: "081-111-1111")
book.addContact(name: "สมหญิง", phone: "082-222-2222")
book.addContact(name: "สมศรี", phone: "083-333-3333")
book.addContact(name: "สมชาย", phone: "090-111-2222") // เบอร์ที่ 2

book.printAll()
print("สมชาย phones: \(book.lookup(name: "สมชาย") ?? [])")
print("Phone owner: \(book.lookupByPhone("082-222-2222") ?? "Not found")")
print("Names starting with สม: \(book.search(prefix: "สม"))")
```

### ตัวอย่างที่ 2: Cache System

```swift
class LRUCache<Key: Hashable, Value> {
    private var cache: [Key: Value] = [:]
    private var accessOrder: [Key] = []
    private let capacity: Int
    
    init(capacity: Int) {
        self.capacity = capacity
    }
    
    func get(_ key: Key) -> Value? {
        guard let value = cache[key] else { return nil }
        
        // Move to end (most recently used)
        accessOrder.removeAll { $0 == key }
        accessOrder.append(key)
        
        return value
    }
    
    func put(_ key: Key, value: Value) {
        if cache[key] != nil {
            accessOrder.removeAll { $0 == key }
        } else if cache.count >= capacity {
            // Remove least recently used
            let lruKey = accessOrder.removeFirst()
            cache.removeValue(forKey: lruKey)
        }
        
        cache[key] = value
        accessOrder.append(key)
    }
    
    var size: Int { cache.count }
}

let lru = LRUCache<String, Int>(capacity: 3)
lru.put("a", value: 1)
lru.put("b", value: 2)
lru.put("c", value: 3)
print(lru.get("a")) // Optional(1)
lru.put("d", value: 4) // ขับ "b" ออก (LRU)
print(lru.get("b")) // nil (ถูกขับออก)
print(lru.get("c")) // Optional(3)
print("Cache size: \(lru.size)") // 3
```

### ตัวอย่างที่ 3: Tag System ด้วย Set

```swift
class TagSystem {
    private var itemTags: [String: Set<String>] = [:]  // item -> tags
    private var tagItems: [String: Set<String>] = [:]  // tag -> items
    
    func addTag(_ tag: String, to item: String) {
        itemTags[item, default: []].insert(tag)
        tagItems[tag, default: []].insert(item)
    }
    
    func removeTag(_ tag: String, from item: String) {
        itemTags[item]?.remove(tag)
        tagItems[tag]?.remove(item)
    }
    
    func tags(for item: String) -> Set<String> {
        return itemTags[item] ?? []
    }
    
    func items(withTag tag: String) -> Set<String> {
        return tagItems[tag] ?? []
    }
    
    func items(withAllTags tags: [String]) -> Set<String> {
        guard let first = tags.first else { return Set(itemTags.keys) }
        var result = items(withTag: first)
        for tag in tags.dropFirst() {
            result = result.intersection(items(withTag: tag))
        }
        return result
    }
    
    func items(withAnyTag tags: [String]) -> Set<String> {
        return tags.reduce(into: Set<String>()) { $0.formUnion(items(withTag: $1)) }
    }
    
    func relatedItems(to item: String) -> Set<String> {
        let myTags = tags(for: item)
        return myTags.reduce(into: Set<String>()) { result, tag in
            result.formUnion(items(withTag: tag))
        }.subtracting([item])
    }
}

let ts = TagSystem()
ts.addTag("swift", to: "article1")
ts.addTag("ios", to: "article1")
ts.addTag("programming", to: "article1")
ts.addTag("swift", to: "article2")
ts.addTag("macos", to: "article2")
ts.addTag("ios", to: "article3")
ts.addTag("swiftui", to: "article3")
ts.addTag("programming", to: "article4")
ts.addTag("python", to: "article4")

print("Swift articles: \(ts.items(withTag: "swift"))")
print("Swift AND iOS: \(ts.items(withAllTags: ["swift", "ios"]))")
print("Related to article1: \(ts.relatedItems(to: "article1"))")
```

### ตัวอย่างที่ 4: Student Grade System

```swift
class GradeSystem {
    private var grades: [String: [String: Int]] = [:] // student -> subject -> grade
    
    func addGrade(student: String, subject: String, grade: Int) {
        grades[student, default: [:]][subject] = grade
    }
    
    func getGrade(student: String, subject: String) -> Int? {
        return grades[student]?[subject]
    }
    
    func average(for student: String) -> Double? {
        guard let studentGrades = grades[student], !studentGrades.isEmpty else { return nil }
        let total = studentGrades.values.reduce(0, +)
        return Double(total) / Double(studentGrades.count)
    }
    
    func topStudents(in subject: String, count: Int) -> [(String, Int)] {
        var result: [(String, Int)] = []
        for (student, subjects) in grades {
            if let grade = subjects[subject] {
                result.append((student, grade))
            }
        }
        return result.sorted { $0.1 > $1.1 }.prefix(count).map { $0 }
    }
    
    func subjectsNeedingImprovement(for student: String, threshold: Int = 70) -> [String] {
        return grades[student]?.filter { $0.value < threshold }.map { $0.key }.sorted() ?? []
    }
    
    func classReport(subject: String) -> [String: Int] {
        var report: [String: Int] = ["A": 0, "B": 0, "C": 0, "D": 0, "F": 0]
        for studentGrades in grades.values {
            guard let grade = studentGrades[subject] else { continue }
            switch grade {
            case 90...100: report["A", default: 0] += 1
            case 80..<90:  report["B", default: 0] += 1
            case 70..<80:  report["C", default: 0] += 1
            case 60..<70:  report["D", default: 0] += 1
            default:       report["F", default: 0] += 1
            }
        }
        return report
    }
}

let gradeSystem = GradeSystem()
let subjects = ["Mathematics", "Science", "English", "Thai"]
let studentData: [(String, [Int])] = [
    ("สมชาย",  [85, 78, 90, 88]),
    ("สมหญิง", [92, 95, 88, 91]),
    ("สมศรี",  [65, 70, 72, 68]),
    ("สมหมาย", [75, 80, 85, 78]),
    ("สมใจ",   [55, 60, 58, 52])
]

for (student, grades) in studentData {
    for (index, subject) in subjects.enumerated() {
        gradeSystem.addGrade(student: student, subject: subject, grade: grades[index])
    }
}

print("Top 3 in Mathematics:")
gradeSystem.topStudents(in: "Mathematics", count: 3).forEach {
    print("  \($0.0): \($0.1)")
}

print("\nNeeds improvement (สมใจ):")
print("  \(gradeSystem.subjectsNeedingImprovement(for: "สมใจ"))")

print("\nMathematics grade distribution:")
gradeSystem.classReport(subject: "Mathematics").sorted { $0.key < $1.key }.forEach {
    print("  Grade \($0.key): \($0.value) students")
}
```

---

## 10.22 สรุป (Summary)

### Dictionary - สิ่งที่ได้เรียนรู้

1. **การสร้าง**: ใช้ `[Key: Value]()` หรือ Dictionary literal
2. **การเข้าถึง**: subscript คืน Optional, ใช้ `default:` เพื่อกำหนดค่าเริ่มต้น
3. **การแก้ไข**: set via subscript, `updateValue`, `merge`, `removeValue`
4. **Higher-Order Functions**: `filter`, `map`, `mapValues`, `compactMapValues`
5. **Grouping**: `Dictionary(grouping:by:)` สำหรับจัดกลุ่มข้อมูล
6. **Performance**: O(1) สำหรับ lookup, insert, delete

### Set - สิ่งที่ได้เรียนรู้

1. **การสร้าง**: ต้องระบุ type อย่างชัดเจน ต้องเป็น Hashable
2. **Set Operations**: union, intersection, subtracting, symmetricDifference
3. **Membership**: contains O(1), isSubset, isSuperset, isDisjoint
4. **Performance**: O(1) สำหรับ insert, remove, contains

### Best Practices

```swift
// 1. ใช้ default value เมื่อ access Dictionary
let count = dict["key", default: 0]

// 2. ใช้ compactMapValues แทน map + filter nil
let valid = dict.compactMapValues { transform($0) }

// 3. ใช้ Set สำหรับ fast membership testing
let allowed: Set<String> = ["read", "write", "admin"]
if allowed.contains(userPermission) { ... }

// 4. ใช้ Dictionary(grouping:) สำหรับการจัดกลุ่ม
let grouped = Dictionary(grouping: items) { $0.category }

// 5. ใช้ Set operations แทน nested loops
let common = setA.intersection(setB) // แทน O(n²) loop

// 6. เรียง keys เมื่อต้องการลำดับ
dict.sorted { $0.key < $1.key }.forEach { ... }
```

### Quick Reference

| Operation | Dictionary | Set |
|-----------|-----------|-----|
| Create empty | `[K:V]()` | `Set<T>()` |
| Add | `dict[k] = v` | `set.insert(x)` |
| Remove | `dict[k] = nil` | `set.remove(x)` |
| Contains | `dict[k] != nil` | `set.contains(x)` |
| Count | `.count` | `.count` |
| Iterate | `for (k,v) in dict` | `for x in set` |
| Filter | `.filter { }` | `.filter { }` |
| Keys | `.keys` | N/A |
| Values | `.values` | N/A |
| Set ops | N/A | `.union()`, `.intersection()` |

---

*จบบทที่ 10: Dictionaries และ Sets ใน Swift*
