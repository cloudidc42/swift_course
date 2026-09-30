# ส่วนที่ 19: Generics ใน Swift

## บทนำ

Generics เป็นหนึ่งในฟีเจอร์ที่ทรงพลังที่สุดใน Swift ช่วยให้เราเขียนโค้ดที่ยืดหยุ่น นำกลับมาใช้ซ้ำได้ และทำงานกับ type ต่างๆ ได้โดยไม่ต้องเขียนโค้ดซ้ำหลายครั้ง Standard Library ของ Swift เองก็สร้างมาจาก Generics เกือบทั้งหมด เช่น `Array`, `Dictionary`, และ `Optional`

---

## 19.1 Generics คืออะไร และทำไมต้องใช้?

### ปัญหาที่ Generics แก้ไข

สมมติว่าเราต้องการฟังก์ชันสำหรับสลับค่าของตัวแปรสองตัว ถ้าไม่มี Generics เราต้องเขียนฟังก์ชันแยกสำหรับแต่ละ type:

```swift
// ❌ วิธีที่ไม่ดี - ต้องเขียนซ้ำสำหรับทุก type
func swapTwoInts(_ a: inout Int, _ b: inout Int) {
    let temporaryA = a
    a = b
    b = temporaryA
}

func swapTwoStrings(_ a: inout String, _ b: inout String) {
    let temporaryA = a
    a = b
    b = temporaryA
}

func swapTwoDoubles(_ a: inout Double, _ b: inout Double) {
    let temporaryA = a
    a = b
    b = temporaryA
}

// ต้องใช้ฟังก์ชันที่ต่างกันสำหรับ type ต่างกัน
var someInt = 3
var anotherInt = 107
swapTwoInts(&someInt, &anotherInt)
print("someInt: \(someInt), anotherInt: \(anotherInt)")
// someInt: 107, anotherInt: 3

var someString = "hello"
var anotherString = "world"
swapTwoStrings(&someString, &anotherString)
print("someString: \(someString), anotherString: \(anotherString)")
// someString: world, anotherString: hello
```

โค้ดด้านบนมีปัญหาเพราะ logic เหมือนกันทุกประการ แต่เราต้องเขียนแยกกันสำหรับแต่ละ type ซึ่งขัดหลัก DRY (Don't Repeat Yourself)

### วิธีแก้ด้วย Generics

```swift
// ✅ วิธีที่ดี - ใช้ Generics
func swapTwoValues<T>(_ a: inout T, _ b: inout T) {
    let temporaryA = a
    a = b
    b = temporaryA
}

// ใช้ฟังก์ชันเดียวกันกับทุก type
var someInt = 3
var anotherInt = 107
swapTwoValues(&someInt, &anotherInt)
print("someInt: \(someInt), anotherInt: \(anotherInt)")
// someInt: 107, anotherInt: 3

var someString = "hello"
var anotherString = "world"
swapTwoValues(&someString, &anotherString)
print("someString: \(someString), anotherString: \(anotherString)")
// someString: world, anotherString: hello

var someDouble = 3.14
var anotherDouble = 2.71
swapTwoValues(&someDouble, &anotherDouble)
print("someDouble: \(someDouble), anotherDouble: \(anotherDouble)")
// someDouble: 2.71, anotherDouble: 3.14
```

### ประโยชน์ของ Generics

1. **Code Reuse (นำโค้ดกลับมาใช้ซ้ำ)**: เขียนครั้งเดียว ใช้ได้หลาย type
2. **Type Safety (ความปลอดภัยของ type)**: Compiler ตรวจสอบ type ให้ตั้งแต่ compile time
3. **Performance (ประสิทธิภาพ)**: Swift สามารถ optimize โค้ดได้ดีกว่า Any หรือ AnyObject
4. **Clarity (ความชัดเจน)**: Code อ่านง่ายและเข้าใจง่ายกว่าการใช้ casting

---

## 19.2 Generic Functions

Generic function คือฟังก์ชันที่ทำงานได้กับหลาย type โดยใช้ **type parameter** เป็นตัวแทน

### ไวยากรณ์พื้นฐาน

```swift
func functionName<T>(parameter: T) -> T {
    // implementation
}
```

ตัว `<T>` คือ **type parameter list** และ `T` คือ **type parameter**

### ตัวอย่าง Generic Functions

```swift
// ฟังก์ชัน print ค่า generic
func printValue<T>(_ value: T) {
    print("Value: \(value)")
}

printValue(42)          // Value: 42
printValue("Hello")     // Value: Hello
printValue(3.14)        // Value: 3.14
printValue([1, 2, 3])   // Value: [1, 2, 3]

// ฟังก์ชันที่รับและคืน generic type
func identity<T>(_ value: T) -> T {
    return value
}

let intValue = identity(10)          // Int
let stringValue = identity("Swift")  // String
let boolValue = identity(true)       // Bool

// ฟังก์ชันที่มี multiple type parameters
func pair<T, U>(_ first: T, _ second: U) -> (T, U) {
    return (first, second)
}

let intString = pair(1, "one")   // (Int, String)
let boolDouble = pair(true, 3.14) // (Bool, Double)

print(intString)   // (1, "one")
print(boolDouble)  // (true, 3.14)

// ฟังก์ชันสำหรับค้นหาใน array
func findFirst<T: Equatable>(in array: [T], matching element: T) -> Int? {
    for (index, item) in array.enumerated() {
        if item == element {
            return index
        }
    }
    return nil
}

let numbers = [1, 5, 3, 7, 2, 8, 4]
if let index = findFirst(in: numbers, matching: 7) {
    print("Found 7 at index \(index)")  // Found 7 at index 3
}

let fruits = ["apple", "banana", "cherry", "date"]
if let index = findFirst(in: fruits, matching: "cherry") {
    print("Found cherry at index \(index)")  // Found cherry at index 2
}
```

### Generic Functions กับ Array Operations

```swift
// ฟังก์ชันแปลง array ด้วย transformation function
func transform<T, U>(_ array: [T], using transform: (T) -> U) -> [U] {
    var result: [U] = []
    for element in array {
        result.append(transform(element))
    }
    return result
}

let numbers = [1, 2, 3, 4, 5]
let doubled = transform(numbers, using: { $0 * 2 })
print(doubled)  // [2, 4, 6, 8, 10]

let strings = transform(numbers, using: { "Number \($0)" })
print(strings)  // ["Number 1", "Number 2", "Number 3", "Number 4", "Number 5"]

// ฟังก์ชัน filter generic
func filter<T>(_ array: [T], where predicate: (T) -> Bool) -> [T] {
    var result: [T] = []
    for element in array {
        if predicate(element) {
            result.append(element)
        }
    }
    return result
}

let evenNumbers = filter(numbers, where: { $0 % 2 == 0 })
print(evenNumbers)  // [2, 4]

// ฟังก์ชัน reduce generic
func reduce<T, U>(_ array: [T], initialValue: U, combining: (U, T) -> U) -> U {
    var result = initialValue
    for element in array {
        result = combining(result, element)
    }
    return result
}

let sum = reduce(numbers, initialValue: 0, combining: { $0 + $1 })
print(sum)  // 15

let product = reduce(numbers, initialValue: 1, combining: { $0 * $1 })
print(product)  // 120

let combined = reduce(numbers, initialValue: "", combining: { "\($0)\($1)" })
print(combined)  // "12345"
```

---

## 19.3 Type Parameters

Type parameter คือ placeholder สำหรับ actual type ที่จะถูกกำหนดเมื่อใช้งาน

### การตั้งชื่อ Type Parameters

```swift
// T เป็น type parameter ที่นิยมใช้ (ย่อมาจาก Type)
func single<T>(_ value: T) -> T { return value }

// K, V ใช้สำหรับ Key-Value
func keyValue<K, V>(key: K, value: V) -> [K: V] {
    return [key: value]
}

// Element ใช้สำหรับ collection
func firstElement<Element>(_ array: [Element]) -> Element? {
    return array.isEmpty ? nil : array[0]
}

// Node ใช้สำหรับ tree หรือ graph structures
struct Node<Value> {
    var value: Value
    var children: [Node<Value>] = []
}

// Result ใช้บ่อยในฟังก์ชันที่คืนค่า
func process<Input, Output>(_ input: Input, transform: (Input) -> Output) -> Output {
    return transform(input)
}
```

### Convention การตั้งชื่อ

```swift
// 1. ใช้ชื่อที่บอกความหมาย เมื่อมี relationship กับ protocol หรือ type อื่น
struct Stack<Element> {  // ✅ ดี - บอกว่าเป็น element ใน stack
    var items: [Element] = []
}

// 2. ใช้ตัวอักษรเดียว (T, U, V) เมื่อไม่มีความหมายเฉพาะ
func swap<T>(_ a: inout T, _ b: inout T) {  // ✅ ดี
    let temp = a; a = b; b = temp
}

// 3. ใช้ชื่อ descriptive สำหรับ complex generic
protocol Container {
    associatedtype Item  // ✅ ดี - ชัดเจนว่าเป็น item ใน container
    var count: Int { get }
    subscript(i: Int) -> Item { get }
}

// 4. ตัวพิมพ์ใหญ่ CamelCase สำหรับ type parameters
struct Pair<FirstType, SecondType> {  // ✅ ดี
    var first: FirstType
    var second: SecondType
}
```

---

## 19.4 Generic Types

นอกจาก functions แล้ว เรายังสามารถสร้าง generic classes, structs, และ enums ได้

### Generic Struct

```swift
// Generic Stack
struct Stack<Element> {
    private var items: [Element] = []
    
    // เพิ่ม element
    mutating func push(_ item: Element) {
        items.append(item)
    }
    
    // ดึง element ออก
    @discardableResult
    mutating func pop() -> Element? {
        return items.popLast()
    }
    
    // ดูค่าบนสุด
    var top: Element? {
        return items.last
    }
    
    // ตรวจสอบว่าว่างหรือไม่
    var isEmpty: Bool {
        return items.isEmpty
    }
    
    // จำนวน elements
    var count: Int {
        return items.count
    }
}

// การใช้งาน
var intStack = Stack<Int>()
intStack.push(1)
intStack.push(2)
intStack.push(3)
print(intStack.top ?? "empty")  // 3
intStack.pop()
print(intStack.top ?? "empty")  // 2

var stringStack = Stack<String>()
stringStack.push("Hello")
stringStack.push("World")
print(stringStack.top ?? "empty")  // World
```

### Generic Class

```swift
// Generic Box
class Box<T> {
    var value: T
    
    init(_ value: T) {
        self.value = value
    }
    
    func transform<U>(_ transform: (T) -> U) -> Box<U> {
        return Box<U>(transform(value))
    }
}

let intBox = Box(42)
print(intBox.value)  // 42

let stringBox = intBox.transform { "The answer is \($0)" }
print(stringBox.value)  // The answer is 42

// Generic Linked List Node
class ListNode<T> {
    var value: T
    var next: ListNode<T>?
    
    init(_ value: T) {
        self.value = value
    }
}

// สร้าง linked list
let node1 = ListNode(1)
let node2 = ListNode(2)
let node3 = ListNode(3)
node1.next = node2
node2.next = node3

// traverse linked list
var current: ListNode<Int>? = node1
while let node = current {
    print(node.value, terminator: " ")  // 1 2 3
    current = node.next
}
print()
```

### Generic Enum

```swift
// Generic Result type (คล้าย Swift's Result)
enum MyResult<Success, Failure: Error> {
    case success(Success)
    case failure(Failure)
    
    var isSuccess: Bool {
        if case .success = self { return true }
        return false
    }
    
    var value: Success? {
        if case .success(let value) = self { return value }
        return nil
    }
    
    var error: Failure? {
        if case .failure(let error) = self { return error }
        return nil
    }
    
    func map<NewSuccess>(_ transform: (Success) -> NewSuccess) -> MyResult<NewSuccess, Failure> {
        switch self {
        case .success(let value):
            return .success(transform(value))
        case .failure(let error):
            return .failure(error)
        }
    }
}

// ตัวอย่างการใช้งาน
enum NetworkError: Error {
    case notFound
    case serverError(Int)
    case invalidData
}

func fetchUser(id: Int) -> MyResult<String, NetworkError> {
    if id == 1 {
        return .success("Alice")
    } else if id == 999 {
        return .failure(.notFound)
    } else {
        return .failure(.serverError(500))
    }
}

let result1 = fetchUser(id: 1)
switch result1 {
case .success(let name):
    print("Found user: \(name)")  // Found user: Alice
case .failure(let error):
    print("Error: \(error)")
}

let result2 = fetchUser(id: 999)
if let error = result2.error {
    print("Error: \(error)")  // Error: notFound
}

// Generic Optional (ตัวอย่างวิธีทำงานของ Swift's Optional)
enum MyOptional<Wrapped> {
    case some(Wrapped)
    case none
    
    func map<U>(_ transform: (Wrapped) -> U) -> MyOptional<U> {
        switch self {
        case .some(let value):
            return .some(transform(value))
        case .none:
            return .none
        }
    }
    
    func flatMap<U>(_ transform: (Wrapped) -> MyOptional<U>) -> MyOptional<U> {
        switch self {
        case .some(let value):
            return transform(value)
        case .none:
            return .none
        }
    }
}
```

---

## 19.5 Type Constraints

Type constraints กำหนดข้อกำหนดว่า type parameter ต้องเป็น subclass ของ class หรือ conform ต่อ protocol

### Syntax ของ Type Constraints

```swift
// ต้องเป็น subclass ของ SomeClass
func function<T: SomeClass>(_ value: T) { }

// ต้อง conform ต่อ SomeProtocol
func function<T: SomeProtocol>(_ value: T) { }

// ต้องทั้ง conform ต่อ protocol และเป็น subclass
func function<T: SomeClass & SomeProtocol>(_ value: T) { }
```

### ตัวอย่าง Type Constraints

```swift
// ใช้ Equatable constraint สำหรับการเปรียบเทียบ
func contains<T: Equatable>(_ array: [T], element: T) -> Bool {
    for item in array {
        if item == element {
            return true
        }
    }
    return false
}

let numbers = [1, 2, 3, 4, 5]
print(contains(numbers, element: 3))  // true
print(contains(numbers, element: 6))  // false

let words = ["swift", "ios", "macos"]
print(contains(words, element: "ios"))  // true

// ใช้ Comparable constraint สำหรับการเรียงลำดับ
func findMinAndMax<T: Comparable>(_ array: [T]) -> (min: T, max: T)? {
    guard !array.isEmpty else { return nil }
    
    var min = array[0]
    var max = array[0]
    
    for element in array.dropFirst() {
        if element < min { min = element }
        if element > max { max = element }
    }
    
    return (min, max)
}

if let result = findMinAndMax([3, 1, 4, 1, 5, 9, 2, 6]) {
    print("Min: \(result.min), Max: \(result.max)")  // Min: 1, Max: 9
}

if let result = findMinAndMax(["banana", "apple", "cherry"]) {
    print("Min: \(result.min), Max: \(result.max)")  // Min: apple, Max: cherry
}

// ใช้ Hashable constraint
func removeDuplicates<T: Hashable>(_ array: [T]) -> [T] {
    var seen = Set<T>()
    var result: [T] = []
    
    for element in array {
        if seen.insert(element).inserted {
            result.append(element)
        }
    }
    
    return result
}

let duplicates = [1, 2, 3, 2, 4, 3, 5]
print(removeDuplicates(duplicates))  // [1, 2, 3, 4, 5]

let dupStrings = ["a", "b", "a", "c", "b"]
print(removeDuplicates(dupStrings))  // ["a", "b", "c"]
```

### Custom Protocol Constraints

```swift
// กำหนด protocol สำหรับ constraint
protocol Printable {
    func prettyPrint()
}

protocol Measurable {
    var measurement: Double { get }
}

// ใช้ custom protocol constraint
func printIfLarge<T: Printable & Measurable>(_ item: T, threshold: Double) {
    if item.measurement > threshold {
        item.prettyPrint()
    } else {
        print("Item is too small to display")
    }
}

struct Shape: Printable, Measurable {
    let name: String
    let area: Double
    
    var measurement: Double { return area }
    
    func prettyPrint() {
        print("Shape: \(name), Area: \(area) sq units")
    }
}

let bigShape = Shape(name: "Circle", area: 314.15)
let smallShape = Shape(name: "Triangle", area: 5.0)

printIfLarge(bigShape, threshold: 100)    // Shape: Circle, Area: 314.15 sq units
printIfLarge(smallShape, threshold: 100)  // Item is too small to display
```

---

## 19.6 Where Clauses

`where` clause ให้เราเพิ่ม constraints เพิ่มเติมที่ซับซ้อนขึ้น

### Where Clause ใน Generic Functions

```swift
// ตรวจสอบว่า arrays สองชุดมีค่าเหมือนกันหรือไม่
func allItemsMatch<C1: Collection, C2: Collection>(_ c1: C1, _ c2: C2) -> Bool
    where C1.Element: Equatable, C1.Element == C2.Element {
    guard c1.count == c2.count else { return false }
    
    for (item1, item2) in zip(c1, c2) {
        if item1 != item2 {
            return false
        }
    }
    return true
}

var stackOfStrings = ["uno", "dos", "tres"]
let arrayOfStrings = ["uno", "dos", "tres"]

if allItemsMatch(stackOfStrings, arrayOfStrings) {
    print("All items match.")  // All items match.
}

// Where clause ใน extension
extension Array where Element: Numeric {
    func sum() -> Element {
        return reduce(0, +)
    }
    
    func average() -> Double {
        guard !isEmpty else { return 0 }
        let sum = reduce(0) { $0 + ($1 as! Double) }
        return sum / Double(count)
    }
}

// ใช้ได้เฉพาะกับ Array ของ Numeric types
let intArray = [1, 2, 3, 4, 5]
print(intArray.sum())  // 15

let doubleArray = [1.5, 2.5, 3.0, 4.0]
print(doubleArray.sum())  // 11.0

// Where clause ใน protocol extension
protocol Container {
    associatedtype Item
    var items: [Item] { get }
}

extension Container where Item: Equatable {
    func contains(_ item: Item) -> Bool {
        return items.contains(item)
    }
    
    func count(of item: Item) -> Int {
        return items.filter { $0 == item }.count
    }
}

extension Container where Item: Comparable {
    func sorted() -> [Item] {
        return items.sorted()
    }
    
    var min: Item? { return items.min() }
    var max: Item? { return items.max() }
}
```

### Where Clause ใน Extensions

```swift
// เพิ่ม method เฉพาะเมื่อ type ตรงตามเงื่อนไข
extension Stack where Element: Equatable {
    func contains(_ element: Element) -> Bool {
        return items.contains(element)
    }
}

extension Stack where Element: Comparable {
    func min() -> Element? {
        return items.min()
    }
    
    func max() -> Element? {
        return items.max()
    }
}

var numberStack = Stack<Int>()
numberStack.push(3)
numberStack.push(1)
numberStack.push(4)
numberStack.push(1)
numberStack.push(5)

print(numberStack.contains(4))  // true
print(numberStack.min() ?? 0)   // 1
print(numberStack.max() ?? 0)   // 5
```

---

## 19.7 Associated Types

Associated types ใช้ใน protocols เพื่อกำหนด type placeholder ที่จะถูกระบุเมื่อ type conform ต่อ protocol

### การประกาศ Associated Types

```swift
protocol Container {
    // กำหนด associated type
    associatedtype Item
    
    mutating func append(_ item: Item)
    var count: Int { get }
    subscript(i: Int) -> Item { get }
}

// ตัวอย่าง Stack conform ต่อ Container
struct IntStack: Container {
    var items: [Int] = []
    
    mutating func push(_ item: Int) {
        items.append(item)
    }
    
    mutating func pop() -> Int? {
        return items.popLast()
    }
    
    // Conform ต่อ Container protocol
    // Swift จะ infer ว่า Item = Int
    mutating func append(_ item: Int) {
        push(item)
    }
    
    var count: Int {
        return items.count
    }
    
    subscript(i: Int) -> Int {
        return items[i]
    }
}

// Generic Stack ที่ conform ต่อ Container
struct GenericStack<Element>: Container {
    var items: [Element] = []
    
    mutating func push(_ item: Element) {
        items.append(item)
    }
    
    mutating func pop() -> Element? {
        return items.popLast()
    }
    
    // Conform ต่อ Container
    // typealias Item = Element (Swift infer ให้อัตโนมัติ)
    mutating func append(_ item: Element) {
        push(item)
    }
    
    var count: Int {
        return items.count
    }
    
    subscript(i: Int) -> Element {
        return items[i]
    }
}
```

### Associated Types กับ Constraints

```swift
protocol ComparableContainer: Container where Item: Comparable {
    func min() -> Item?
    func max() -> Item?
}

// หรือใช้ constraint ใน associatedtype
protocol SortableContainer {
    associatedtype Item: Comparable
    var items: [Item] { get }
    func sorted() -> [Item]
}

struct SortableArray<T: Comparable>: SortableContainer {
    var items: [T]
    
    func sorted() -> [T] {
        return items.sorted()
    }
    
    func min() -> T? {
        return items.min()
    }
    
    func max() -> T? {
        return items.max()
    }
}

let sortable = SortableArray(items: [5, 3, 8, 1, 9, 2])
print(sortable.sorted())        // [1, 2, 3, 5, 8, 9]
print(sortable.min() ?? 0)      // 1
print(sortable.max() ?? 0)      // 9
```

### Extending Protocol with Where Clause

```swift
// เพิ่ม default implementation เมื่อ associated type ตรงตามเงื่อนไข
extension Container where Item: Equatable {
    func contains(_ item: Item) -> Bool {
        for i in 0..<count {
            if self[i] == item {
                return true
            }
        }
        return false
    }
}

// ทดสอบ
var stack = GenericStack<String>()
stack.push("Hello")
stack.push("World")
stack.push("Swift")

print(stack.contains("World"))  // true
print(stack.contains("Python")) // false
```

---

## 19.8 Generic Subscripts

```swift
// Generic subscript ใน struct
struct SafeCollection<T> {
    private var items: [T]
    
    init(_ items: [T]) {
        self.items = items
    }
    
    // Generic subscript ที่รับ range
    subscript<Indices: Sequence>(indices: Indices) -> [T] where Indices.Element == Int {
        return indices.compactMap { index in
            guard index >= 0 && index < items.count else { return nil }
            return items[index]
        }
    }
    
    // Subscript ปกติ
    subscript(safe index: Int) -> T? {
        guard index >= 0 && index < items.count else { return nil }
        return items[index]
    }
}

let collection = SafeCollection([10, 20, 30, 40, 50])
print(collection[safe: 2] ?? -1)     // 30
print(collection[safe: 10] ?? -1)    // -1

let selectedItems = collection[[0, 2, 4]]
print(selectedItems)  // [10, 30, 50]

// Generic subscript ใน Dictionary extension
extension Dictionary {
    subscript<T>(key: Key, default defaultValue: T) -> T where Value == T {
        return self[key] ?? defaultValue
    }
}

let scores: [String: Int] = ["Alice": 95, "Bob": 87]
print(scores["Alice", default: 0])   // 95
print(scores["Charlie", default: 0]) // 0
```

---

## 19.9 Protocol ด้วย Associated Types (Generic Protocols)

```swift
// Protocol ที่ทำงานเหมือน Generic
protocol Queue {
    associatedtype Element
    
    mutating func enqueue(_ element: Element)
    mutating func dequeue() -> Element?
    var front: Element? { get }
    var isEmpty: Bool { get }
    var count: Int { get }
}

// Implementation ของ Queue
struct SimpleQueue<T>: Queue {
    private var items: [T] = []
    
    mutating func enqueue(_ element: T) {
        items.append(element)
    }
    
    mutating func dequeue() -> T? {
        guard !items.isEmpty else { return nil }
        return items.removeFirst()
    }
    
    var front: T? {
        return items.first
    }
    
    var isEmpty: Bool {
        return items.isEmpty
    }
    
    var count: Int {
        return items.count
    }
}

// Double-ended Queue (Deque)
protocol Deque: Queue {
    mutating func pushFront(_ element: Element)
    mutating func popBack() -> Element?
    var back: Element? { get }
}

struct SimpleDeque<T>: Deque {
    private var items: [T] = []
    
    mutating func enqueue(_ element: T) {
        items.append(element)
    }
    
    mutating func dequeue() -> T? {
        guard !items.isEmpty else { return nil }
        return items.removeFirst()
    }
    
    mutating func pushFront(_ element: T) {
        items.insert(element, at: 0)
    }
    
    mutating func popBack() -> T? {
        return items.popLast()
    }
    
    var front: T? { return items.first }
    var back: T? { return items.last }
    var isEmpty: Bool { return items.isEmpty }
    var count: Int { return items.count }
}

// ทดสอบ
var queue = SimpleQueue<String>()
queue.enqueue("First")
queue.enqueue("Second")
queue.enqueue("Third")
print(queue.front ?? "empty")   // First
queue.dequeue()
print(queue.front ?? "empty")   // Second
print(queue.count)               // 2
```

---

## 19.10 Extending Generic Types

```swift
// เพิ่ม functionality ให้ generic type
extension Stack {
    // เพิ่ม method ที่ใช้ได้กับทุก Element type
    func peek(at index: Int) -> Element? {
        guard index >= 0 && index < items.count else { return nil }
        return items[items.count - 1 - index]  // จากบนลงล่าง
    }
    
    var allItems: [Element] {
        return items.reversed()
    }
}

// เพิ่ม Sequence conformance
extension Stack: Sequence {
    func makeIterator() -> IndexingIterator<[Element]> {
        return items.reversed().makeIterator()
    }
}

// ตอนนี้ Stack สามารถ iterate ได้
var stack = Stack<Int>()
stack.push(1)
stack.push(2)
stack.push(3)

for item in stack {
    print(item, terminator: " ")  // 3 2 1
}
print()

// เพิ่ม CustomStringConvertible
extension Stack: CustomStringConvertible {
    var description: String {
        let items = self.items.map { "\($0)" }
        return "Stack: [\(items.joined(separator: ", "))] (top: \(self.top.map {"\($0)"} ?? "empty"))"
    }
}

print(stack)  // Stack: [1, 2, 3] (top: 3)

// Extend เฉพาะเมื่อ Element conform ต่อ Codable
extension Stack: Codable where Element: Codable {
    // Swift จัดการ encoding/decoding ให้อัตโนมัติ
}
```

---

## 19.11 Type Erasure

Type erasure เป็นเทคนิคในการซ่อน concrete type ไว้เบื้องหลัง wrapper เพื่อให้ทำงานกับ protocol ที่มี associated types ได้

### ปัญหาที่ Type Erasure แก้ไข

```swift
protocol Animal {
    associatedtype Sound
    func makeSound() -> Sound
    var name: String { get }
}

struct Dog: Animal {
    let name = "Dog"
    func makeSound() -> String { return "Woof" }
}

struct Cat: Animal {
    let name = "Cat"
    func makeSound() -> String { return "Meow" }
}

// ❌ ทำแบบนี้ไม่ได้เพราะ Animal มี associated type
// let animals: [Animal] = [Dog(), Cat()]  // Error!

// ✅ ใช้ Type Erasure
struct AnyAnimal<Sound>: Animal {
    let name: String
    private let _makeSound: () -> Sound
    
    init<A: Animal>(_ animal: A) where A.Sound == Sound {
        self.name = animal.name
        self._makeSound = animal.makeSound
    }
    
    func makeSound() -> Sound {
        return _makeSound()
    }
}

// ตอนนี้ทำได้
let animals: [AnyAnimal<String>] = [AnyAnimal(Dog()), AnyAnimal(Cat())]

for animal in animals {
    print("\(animal.name) says: \(animal.makeSound())")
}
// Dog says: Woof
// Cat says: Meow
```

### AnySequence - ตัวอย่างจาก Standard Library

```swift
// Type-erased wrapper สำหรับ Sequence
let range: AnySequence<Int> = AnySequence(1...5)
let array: AnySequence<Int> = AnySequence([1, 2, 3, 4, 5])

// ทั้งสองใช้ผ่าน AnySequence<Int> ได้เหมือนกัน
func processSequence(_ seq: AnySequence<Int>) {
    for element in seq {
        print(element, terminator: " ")
    }
    print()
}

processSequence(range)  // 1 2 3 4 5
processSequence(array)  // 1 2 3 4 5
```

### Custom Type Erasure Pattern

```swift
// Protocol ที่มี associated type
protocol Formatter {
    associatedtype Input
    associatedtype Output
    func format(_ input: Input) -> Output
}

// Type-erased wrapper
struct AnyFormatter<I, O>: Formatter {
    private let _format: (I) -> O
    
    init<F: Formatter>(_ formatter: F) where F.Input == I, F.Output == O {
        self._format = formatter.format
    }
    
    func format(_ input: I) -> O {
        return _format(input)
    }
}

// Concrete formatters
struct UppercaseFormatter: Formatter {
    func format(_ input: String) -> String {
        return input.uppercased()
    }
}

struct LengthFormatter: Formatter {
    func format(_ input: String) -> Int {
        return input.count
    }
}

// ใช้ AnyFormatter
let formatters: [AnyFormatter<String, String>] = [
    AnyFormatter(UppercaseFormatter())
]

for formatter in formatters {
    print(formatter.format("hello"))  // HELLO
}
```

---

## 19.12 Any vs Generics

```swift
// ❌ การใช้ Any - ไม่มี type safety
func printAny(_ value: Any) {
    print(value)
}

func addAny(_ a: Any, _ b: Any) -> Any {
    // ต้อง cast และอาจ crash ที่ runtime
    if let intA = a as? Int, let intB = b as? Int {
        return intA + intB
    }
    if let strA = a as? String, let strB = b as? String {
        return strA + strB
    }
    fatalError("Cannot add these types")
}

let result1 = addAny(1, 2)  // Any - ไม่รู้ว่าเป็น Int หรืออะไร
// print(result1 + 1)  // Error - ต้อง cast ก่อน
if let intResult = result1 as? Int {
    print(intResult + 1)  // 4
}

// ✅ การใช้ Generics - type safety เต็มที่
func printGeneric<T>(_ value: T) {
    print(value)
}

func addGeneric<T: AdditiveArithmetic>(_ a: T, _ b: T) -> T {
    return a + b
}

let result2 = addGeneric(1, 2)      // Int - รู้แน่นอน
print(result2 + 1)                  // 4 - ไม่ต้อง cast

let result3 = addGeneric(1.5, 2.5)  // Double
print(result3 + 0.5)                // 4.5

// Performance comparison
// Any: boxing/unboxing ทำให้ช้ากว่า
// Generics: monomorphization ทำให้เร็วเท่ากับโค้ดที่เขียนเฉพาะ
```

---

## 19.13 Opaque Types (some Keyword)

Opaque types ให้ฟังก์ชันคืน "some type ที่ conform ต่อ protocol" โดยไม่เปิดเผย concrete type

```swift
// Protocol ที่ใช้สาธิต
protocol Shape {
    func area() -> Double
    func perimeter() -> Double
    var name: String { get }
}

struct Circle: Shape {
    let radius: Double
    var name: String { "Circle" }
    func area() -> Double { Double.pi * radius * radius }
    func perimeter() -> Double { 2 * Double.pi * radius }
}

struct Rectangle: Shape {
    let width: Double
    let height: Double
    var name: String { "Rectangle" }
    func area() -> Double { width * height }
    func perimeter() -> Double { 2 * (width + height) }
}

struct Triangle: Shape {
    let base: Double
    let height: Double
    let sideA: Double
    let sideB: Double
    var name: String { "Triangle" }
    func area() -> Double { 0.5 * base * height }
    func perimeter() -> Double { base + sideA + sideB }
}

// ❌ ไม่สามารถใช้ protocol เป็น return type ถ้า protocol มี associated type
// func makeShape() -> Shape { ... }  // อาจเกิดปัญหา

// ✅ Opaque Types ด้วย some
func makeCircle(radius: Double) -> some Shape {
    return Circle(radius: radius)
}

func makeRectangle(width: Double, height: Double) -> some Shape {
    return Rectangle(width: width, height: height)
}

let circle = makeCircle(radius: 5.0)
print("Area: \(circle.area())")       // Area: 78.53981633974483
print("Name: \(circle.name)")         // Name: Circle

// Opaque return type ใน SwiftUI style
protocol View {
    associatedtype Body: View
    var body: Body { get }
}

// ใน SwiftUI จริงๆ:
// struct ContentView: View {
//     var body: some View {  // some View คือ opaque type
//         Text("Hello")
//     }
// }

// ข้อแตกต่างระหว่าง some และ any
func opaqueShape() -> some Shape {
    return Circle(radius: 1.0)
    // Compiler รู้ว่าเป็น Circle เสมอ (one specific type)
}

// some Shape:
// - Preserves type identity
// - Compiler รู้ concrete type
// - ใช้ static dispatch
// - ไม่สามารถ mix types

// any Shape:
// - Type erasure
// - Compiler ไม่รู้ concrete type ณ compile time
// - ใช้ dynamic dispatch
// - สามารถ mix types ได้
```

---

## 19.14 Existential Types (any Keyword)

ใน Swift 5.7 ขึ้นไป เราต้องเขียน `any Protocol` อย่างชัดเจนเมื่อต้องการ existential type

```swift
// ก่อน Swift 5.7 - implicit existential
// let shape: Shape = Circle(radius: 5)

// Swift 5.7+ - explicit existential ด้วย any
let shape: any Shape = Circle(radius: 5)

// Array ของ existential types
let shapes: [any Shape] = [
    Circle(radius: 5),
    Rectangle(width: 10, height: 3),
    Triangle(base: 6, height: 4, sideA: 5, sideB: 5)
]

// Iterate over existential types
for shape in shapes {
    print("\(shape.name): area = \(String(format: "%.2f", shape.area()))")
}
// Circle: area = 78.54
// Rectangle: area = 30.00
// Triangle: area = 12.00

// ข้อจำกัดของ existential types
// 1. ไม่สามารถใช้ protocol ที่มี Self requirements หรือ associated types เป็น existential ได้โดยตรง
//    (ต้องใช้ type erasure หรือ some)

// 2. Performance overhead - boxing/unboxing
// 3. ไม่สามารถใช้ == โดยตรง

// เปรียบเทียบ some vs any
func processSome(_ shape: some Shape) {
    // shape มี concrete type ที่รู้แน่นอน
    print("Processing \(shape.name)")
}

func processAny(_ shape: any Shape) {
    // shape อาจเป็น type ใดก็ได้
    print("Processing \(shape.name)")
}

processSome(Circle(radius: 3))
processAny(Rectangle(width: 5, height: 2))

// Opening existentials (Swift 5.7+)
func computeTotalArea(_ shapes: [any Shape]) -> Double {
    return shapes.reduce(0) { total, shape in
        total + shape.area()  // Swift automatically "opens" the existential
    }
}

let totalArea = computeTotalArea(shapes)
print("Total area: \(String(format: "%.2f", totalArea))")
```

---

## 19.15 Generic Algorithms

```swift
// Binary Search - Generic Implementation
func binarySearch<T: Comparable>(_ sortedArray: [T], target: T) -> Int? {
    var low = 0
    var high = sortedArray.count - 1
    
    while low <= high {
        let mid = (low + high) / 2
        let midValue = sortedArray[mid]
        
        if midValue == target {
            return mid
        } else if midValue < target {
            low = mid + 1
        } else {
            high = mid - 1
        }
    }
    
    return nil
}

let sortedNumbers = [1, 3, 5, 7, 9, 11, 13, 15, 17, 19]
if let index = binarySearch(sortedNumbers, target: 11) {
    print("Found 11 at index \(index)")  // Found 11 at index 5
}

let sortedStrings = ["apple", "banana", "cherry", "date", "elderberry"]
if let index = binarySearch(sortedStrings, target: "cherry") {
    print("Found cherry at index \(index)")  // Found cherry at index 2
}

// Merge Sort - Generic Implementation
func mergeSort<T: Comparable>(_ array: [T]) -> [T] {
    guard array.count > 1 else { return array }
    
    let middle = array.count / 2
    let left = mergeSort(Array(array[..<middle]))
    let right = mergeSort(Array(array[middle...]))
    
    return merge(left, right)
}

func merge<T: Comparable>(_ left: [T], _ right: [T]) -> [T] {
    var result: [T] = []
    var leftIndex = 0
    var rightIndex = 0
    
    while leftIndex < left.count && rightIndex < right.count {
        if left[leftIndex] <= right[rightIndex] {
            result.append(left[leftIndex])
            leftIndex += 1
        } else {
            result.append(right[rightIndex])
            rightIndex += 1
        }
    }
    
    result.append(contentsOf: left[leftIndex...])
    result.append(contentsOf: right[rightIndex...])
    
    return result
}

let unsorted = [38, 27, 43, 3, 9, 82, 10]
let sorted = mergeSort(unsorted)
print(sorted)  // [3, 9, 10, 27, 38, 43, 82]

let unsortedStrings = ["banana", "apple", "cherry", "date"]
let sortedStrings2 = mergeSort(unsortedStrings)
print(sortedStrings2)  // ["apple", "banana", "cherry", "date"]

// QuickSelect Algorithm - หา k-th smallest element
func quickSelect<T: Comparable>(_ array: inout [T], k: Int, low: Int, high: Int) -> T {
    if low == high { return array[low] }
    
    let pivot = partition(&array, low: low, high: high)
    
    if k == pivot {
        return array[pivot]
    } else if k < pivot {
        return quickSelect(&array, k: k, low: low, high: pivot - 1)
    } else {
        return quickSelect(&array, k: k, low: pivot + 1, high: high)
    }
}

func partition<T: Comparable>(_ array: inout [T], low: Int, high: Int) -> Int {
    let pivot = array[high]
    var i = low
    
    for j in low..<high {
        if array[j] <= pivot {
            array.swapAt(i, j)
            i += 1
        }
    }
    
    array.swapAt(i, high)
    return i
}

var arr = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3]
let kthSmallest = quickSelect(&arr, k: 4, low: 0, high: arr.count - 1)
print("5th smallest: \(kthSmallest)")  // 4 (0-indexed, so 5th = index 4)
```

---

## 19.16 Generic Data Structures

### Generic Stack

```swift
// ปรับปรุง Stack ให้สมบูรณ์
struct Stack<Element> {
    private var storage: [Element] = []
    
    init() {}
    
    init(_ elements: [Element]) {
        storage = elements
    }
    
    mutating func push(_ element: Element) {
        storage.append(element)
    }
    
    @discardableResult
    mutating func pop() -> Element? {
        return storage.popLast()
    }
    
    var top: Element? { storage.last }
    var bottom: Element? { storage.first }
    var isEmpty: Bool { storage.isEmpty }
    var count: Int { storage.count }
    
    func toArray() -> [Element] {
        return storage
    }
}

extension Stack: ExpressibleByArrayLiteral {
    init(arrayLiteral elements: Element...) {
        storage = elements
    }
}

extension Stack: CustomStringConvertible where Element: CustomStringConvertible {
    var description: String {
        let elements = storage.map { $0.description }.joined(separator: ", ")
        return "Stack([\(elements)])"
    }
}

extension Stack: Equatable where Element: Equatable {
    static func == (lhs: Stack<Element>, rhs: Stack<Element>) -> Bool {
        return lhs.storage == rhs.storage
    }
}

// ทดสอบ
var stack: Stack<Int> = [1, 2, 3, 4, 5]
print(stack)  // Stack([1, 2, 3, 4, 5])
stack.push(6)
print(stack.top ?? 0)    // 6
print(stack.count)        // 6
```

### Generic Queue

```swift
// Queue ด้วย Two-Stack Implementation (Amortized O(1))
struct Queue<Element> {
    private var enqueueStack: [Element] = []
    private var dequeueStack: [Element] = []
    
    init() {}
    
    mutating func enqueue(_ element: Element) {
        enqueueStack.append(element)
    }
    
    mutating func dequeue() -> Element? {
        if dequeueStack.isEmpty {
            dequeueStack = enqueueStack.reversed()
            enqueueStack.removeAll()
        }
        return dequeueStack.popLast()
    }
    
    var front: Element? {
        if dequeueStack.isEmpty {
            return enqueueStack.first
        }
        return dequeueStack.last
    }
    
    var isEmpty: Bool {
        return enqueueStack.isEmpty && dequeueStack.isEmpty
    }
    
    var count: Int {
        return enqueueStack.count + dequeueStack.count
    }
}

extension Queue: Sequence {
    func makeIterator() -> IndexingIterator<[Element]> {
        return (dequeueStack.reversed() + enqueueStack).makeIterator()
    }
}

// ทดสอบ
var queue = Queue<String>()
queue.enqueue("First")
queue.enqueue("Second")
queue.enqueue("Third")

print(queue.front ?? "empty")  // First
queue.dequeue()
print(queue.front ?? "empty")  // Second
print(queue.count)              // 2

for item in queue {
    print(item)  // Second, Third
}
```

### Generic LinkedList

```swift
// Singly Linked List
final class LinkedList<T> {
    private class Node<T> {
        var value: T
        var next: Node<T>?
        
        init(_ value: T) {
            self.value = value
        }
    }
    
    private var head: Node<T>?
    private var tail: Node<T>?
    private(set) var count: Int = 0
    
    var isEmpty: Bool { head == nil }
    var first: T? { head?.value }
    var last: T? { tail?.value }
    
    // เพิ่มที่ท้าย
    func append(_ value: T) {
        let newNode = Node(value)
        
        if tail == nil {
            head = newNode
            tail = newNode
        } else {
            tail?.next = newNode
            tail = newNode
        }
        count += 1
    }
    
    // เพิ่มที่ต้น
    func prepend(_ value: T) {
        let newNode = Node(value)
        newNode.next = head
        head = newNode
        if tail == nil { tail = newNode }
        count += 1
    }
    
    // ลบ node แรก
    @discardableResult
    func removeFirst() -> T? {
        guard let node = head else { return nil }
        
        head = node.next
        if head == nil { tail = nil }
        count -= 1
        
        return node.value
    }
    
    // แปลงเป็น Array
    func toArray() -> [T] {
        var array: [T] = []
        var current = head
        while let node = current {
            array.append(node.value)
            current = node.next
        }
        return array
    }
}

extension LinkedList: Sequence {
    func makeIterator() -> AnyIterator<T> {
        var current = head
        return AnyIterator {
            guard let node = current else { return nil }
            current = node.next
            return node.value
        }
    }
}

extension LinkedList: CustomStringConvertible {
    var description: String {
        return toArray().map { "\($0)" }.joined(separator: " -> ")
    }
}

// ทดสอบ
let list = LinkedList<Int>()
list.append(1)
list.append(2)
list.append(3)
list.prepend(0)

print(list)  // 0 -> 1 -> 2 -> 3
print("Count: \(list.count)")  // Count: 4
print("First: \(list.first ?? -1)")  // First: 0
print("Last: \(list.last ?? -1)")    // Last: 3

list.removeFirst()
print(list)  // 1 -> 2 -> 3

for item in list {
    print(item, terminator: " ")  // 1 2 3
}
```

### Generic Binary Search Tree

```swift
// Binary Search Tree
class BinarySearchTree<T: Comparable> {
    private class Node {
        var value: T
        var left: Node?
        var right: Node?
        
        init(_ value: T) {
            self.value = value
        }
    }
    
    private var root: Node?
    
    func insert(_ value: T) {
        root = insert(root, value: value)
    }
    
    private func insert(_ node: Node?, value: T) -> Node {
        guard let node = node else {
            return Node(value)
        }
        
        if value < node.value {
            node.left = insert(node.left, value: value)
        } else if value > node.value {
            node.right = insert(node.right, value: value)
        }
        
        return node
    }
    
    func contains(_ value: T) -> Bool {
        return contains(root, value: value)
    }
    
    private func contains(_ node: Node?, value: T) -> Bool {
        guard let node = node else { return false }
        
        if value == node.value { return true }
        if value < node.value { return contains(node.left, value: value) }
        return contains(node.right, value: value)
    }
    
    // In-order traversal (sorted)
    func inOrder() -> [T] {
        var result: [T] = []
        inOrder(root, result: &result)
        return result
    }
    
    private func inOrder(_ node: Node?, result: inout [T]) {
        guard let node = node else { return }
        inOrder(node.left, result: &result)
        result.append(node.value)
        inOrder(node.right, result: &result)
    }
}

// ทดสอบ
let bst = BinarySearchTree<Int>()
[5, 3, 7, 1, 4, 6, 8].forEach { bst.insert($0) }

print(bst.inOrder())  // [1, 3, 4, 5, 6, 7, 8]
print(bst.contains(4))  // true
print(bst.contains(9))  // false
```

---

## 19.17 Standard Library Generics

```swift
// Array<Element>
var numbers: Array<Int> = [1, 2, 3]  // เหมือนกับ [Int]
var strings: [String] = ["a", "b"]

// map, filter, reduce - Generic methods
let doubled = numbers.map { $0 * 2 }        // [2, 4, 6]
let evens = numbers.filter { $0 % 2 == 0 }  // [2]
let sum = numbers.reduce(0, +)               // 6

// compactMap - filter out nil values
let optionals: [Int?] = [1, nil, 3, nil, 5]
let nonNils = optionals.compactMap { $0 }  // [1, 3, 5]

// flatMap
let nested = [[1, 2], [3, 4], [5, 6]]
let flat = nested.flatMap { $0 }  // [1, 2, 3, 4, 5, 6]

// Dictionary<Key: Hashable, Value>
var dict: Dictionary<String, Int> = ["a": 1]  // เหมือนกับ [String: Int]

// mapValues
let doubledValues = dict.mapValues { $0 * 2 }
print(doubledValues)  // ["a": 2]

// filter
let largeValues = dict.filter { $0.value > 0 }
print(largeValues)  // ["a": 1]

// Optional<Wrapped>
var optional: Optional<Int> = .some(42)  // เหมือนกับ Int?

// map, flatMap บน Optional
let mapped = optional.map { $0 * 2 }    // Optional(84)
let string = optional.map { "\($0)" }   // Optional("42")

// Result<Success, Failure>
enum AppError: Error { case failed }

let result: Result<Int, AppError> = .success(42)
let doubled2 = result.map { $0 * 2 }  // .success(84)

switch doubled2 {
case .success(let value):
    print("Success: \(value)")  // Success: 84
case .failure(let error):
    print("Error: \(error)")
}
```

---

## 19.18 ตัวอย่างจริง: Generic Cache

```swift
// Generic Cache with LRU (Least Recently Used) eviction
final class LRUCache<Key: Hashable, Value> {
    private let capacity: Int
    private var cache: [Key: Value] = [:]
    private var order: [Key] = []  // LRU order
    
    init(capacity: Int) {
        self.capacity = capacity
    }
    
    func get(_ key: Key) -> Value? {
        guard let value = cache[key] else { return nil }
        
        // Move to most recently used
        order.removeAll { $0 == key }
        order.append(key)
        
        return value
    }
    
    func put(_ key: Key, value: Value) {
        if let _ = cache[key] {
            // Update existing
            cache[key] = value
            order.removeAll { $0 == key }
            order.append(key)
        } else {
            // Add new
            if cache.count >= capacity {
                // Remove least recently used
                if let lruKey = order.first {
                    cache.removeValue(forKey: lruKey)
                    order.removeFirst()
                }
            }
            cache[key] = value
            order.append(key)
        }
    }
    
    var size: Int { return cache.count }
    var keys: [Key] { return order }
}

// ทดสอบ
let cache = LRUCache<String, Int>(capacity: 3)
cache.put("a", value: 1)
cache.put("b", value: 2)
cache.put("c", value: 3)

print(cache.get("a") ?? -1)  // 1 (a ถูก access)
cache.put("d", value: 4)      // b ถูกลบ (LRU)

print(cache.get("b") ?? -1)  // -1 (b ถูกลบไปแล้ว)
print(cache.get("c") ?? -1)  // 3
print(cache.get("d") ?? -1)  // 4
print("Cache size: \(cache.size)")  // Cache size: 3
```

---

## 19.19 แบบฝึกหัด

### แบบฝึกหัดที่ 1: Generic Pair

```swift
// สร้าง generic struct Pair ที่เก็บ 2 ค่าที่ต่างกัน
struct Pair<First, Second> {
    let first: First
    let second: Second
    
    init(_ first: First, _ second: Second) {
        self.first = first
        self.second = second
    }
    
    // สลับ Pair
    func swapped() -> Pair<Second, First> {
        return Pair<Second, First>(second, first)
    }
}

// เพิ่ม Equatable เมื่อทั้ง First และ Second เป็น Equatable
extension Pair: Equatable where First: Equatable, Second: Equatable {
    static func == (lhs: Pair, rhs: Pair) -> Bool {
        return lhs.first == rhs.first && lhs.second == rhs.second
    }
}

// เพิ่ม map
extension Pair {
    func mapFirst<T>(_ transform: (First) -> T) -> Pair<T, Second> {
        return Pair<T, Second>(transform(first), second)
    }
    
    func mapSecond<T>(_ transform: (Second) -> T) -> Pair<First, T> {
        return Pair<First, T>(first, transform(second))
    }
}

// ทดสอบ
let pair = Pair(1, "one")
print(pair.first)   // 1
print(pair.second)  // one

let swapped = pair.swapped()
print(swapped.first)   // one
print(swapped.second)  // 1

let doubled = pair.mapFirst { $0 * 2 }
print(doubled.first)  // 2
```

### แบบฝึกหัดที่ 2: Generic Priority Queue

```swift
// Priority Queue ที่ element ที่มี priority สูงกว่าจะถูกดึงออกก่อน
struct PriorityQueue<Element> {
    private var elements: [Element] = []
    private let priorityFunction: (Element, Element) -> Bool
    
    init(priorityFunction: @escaping (Element, Element) -> Bool) {
        self.priorityFunction = priorityFunction
    }
    
    var isEmpty: Bool { elements.isEmpty }
    var count: Int { elements.count }
    var peek: Element? { elements.first }
    
    mutating func enqueue(_ element: Element) {
        elements.append(element)
        elements.sort(by: priorityFunction)
    }
    
    mutating func dequeue() -> Element? {
        guard !isEmpty else { return nil }
        return elements.removeFirst()
    }
}

// Max Priority Queue สำหรับ Int
var maxPQ = PriorityQueue<Int>(priorityFunction: >)
maxPQ.enqueue(3)
maxPQ.enqueue(1)
maxPQ.enqueue(4)
maxPQ.enqueue(1)
maxPQ.enqueue(5)

while !maxPQ.isEmpty {
    print(maxPQ.dequeue() ?? 0, terminator: " ")  // 5 4 3 1 1
}
print()

// Task Priority Queue
struct Task {
    let name: String
    let priority: Int
}

var taskQueue = PriorityQueue<Task>(priorityFunction: { $0.priority > $1.priority })
taskQueue.enqueue(Task(name: "Low priority task", priority: 1))
taskQueue.enqueue(Task(name: "High priority task", priority: 10))
taskQueue.enqueue(Task(name: "Medium priority task", priority: 5))

while !taskQueue.isEmpty {
    if let task = taskQueue.dequeue() {
        print("Processing: \(task.name) (priority: \(task.priority))")
    }
}
// Processing: High priority task (priority: 10)
// Processing: Medium priority task (priority: 5)
// Processing: Low priority task (priority: 1)
```

### แบบฝึกหัดที่ 3: Generic Event System

```swift
// Generic Event System
class EventEmitter<EventType: Hashable, DataType> {
    private var listeners: [EventType: [(DataType) -> Void]] = [:]
    
    func on(_ event: EventType, handler: @escaping (DataType) -> Void) {
        if listeners[event] == nil {
            listeners[event] = []
        }
        listeners[event]?.append(handler)
    }
    
    func emit(_ event: EventType, data: DataType) {
        listeners[event]?.forEach { handler in
            handler(data)
        }
    }
    
    func removeAllListeners(for event: EventType) {
        listeners.removeValue(forKey: event)
    }
}

// ทดสอบ
enum UserEvent: String, Hashable {
    case login
    case logout
    case profileUpdated
}

struct UserData {
    let id: Int
    let name: String
}

let emitter = EventEmitter<UserEvent, UserData>()

emitter.on(.login) { user in
    print("User logged in: \(user.name)")
}

emitter.on(.login) { user in
    print("Analytics: Login event for user \(user.id)")
}

emitter.on(.logout) { user in
    print("User logged out: \(user.name)")
}

emitter.emit(.login, data: UserData(id: 1, name: "Alice"))
// User logged in: Alice
// Analytics: Login event for user 1

emitter.emit(.logout, data: UserData(id: 1, name: "Alice"))
// User logged out: Alice
```

---

## 19.20 สรุป

### สิ่งที่เรียนรู้ในบทนี้

1. **Generic Functions**: ฟังก์ชันที่ทำงานได้กับหลาย type โดยใช้ type parameters
2. **Generic Types**: Classes, Structs, Enums ที่ทำงานกับ generic types
3. **Type Constraints**: การจำกัด type parameters ด้วย protocols หรือ class inheritance
4. **Where Clauses**: เงื่อนไขเพิ่มเติมสำหรับ type constraints
5. **Associated Types**: Type placeholders ใน protocols
6. **Type Erasure**: เทคนิคซ่อน concrete type เบื้องหลัง wrapper
7. **Opaque Types (some)**: Return type ที่ซ่อน concrete type แต่ยังคง type identity
8. **Existential Types (any)**: Type erasure ที่ built-in ใน Swift
9. **Generic Algorithms**: Binary search, merge sort, quickselect
10. **Generic Data Structures**: Stack, Queue, LinkedList, BST

### Best Practices

```swift
// 1. ใช้ Generics แทน Any เมื่อเป็นไปได้
func goodFunction<T: Comparable>(_ value: T) { }  // ✅
func badFunction(_ value: Any) { }                  // ❌

// 2. ตั้งชื่อ type parameters ให้มีความหมาย
struct Cache<Key: Hashable, Value> { }  // ✅
struct Cache<K, V> { }                  // ❌ (ไม่ชัดเจน)

// 3. ใช้ where clause สำหรับ complex constraints
func process<T, U>(_ pair: (T, U)) where T: Equatable, U: Comparable { }

// 4. ชอบ some ก่อน any เมื่อทำได้ (ประสิทธิภาพดีกว่า)
func makeShape() -> some Shape { Circle(radius: 1) }  // ✅ ดีกว่า
func makeShape() -> any Shape { Circle(radius: 1) }   // ✅ ใช้ได้ แต่มี overhead

// 5. Type erasure เมื่อต้องการ heterogeneous collections
let items: [AnyShape] = [AnyShape(Circle()), AnyShape(Rectangle())]
```

### Quick Reference

| Feature | ไวยากรณ์ | ใช้เมื่อ |
|---------|---------|---------|
| Generic Function | `func f<T>(_ x: T)` | ฟังก์ชันที่ทำงานหลาย types |
| Type Constraint | `<T: Protocol>` | ต้องการความสามารถเฉพาะ |
| Where Clause | `where T: P, T == U` | Constraints ที่ซับซ้อน |
| Associated Type | `associatedtype Item` | Generic protocols |
| Opaque Type | `some Protocol` | Return type ที่ซ่อน concrete type |
| Existential Type | `any Protocol` | Heterogeneous collections |

---

*จบบทที่ 19: Generics ใน Swift*

*บทถัดไป: บทที่ 20 - Memory Management และ ARC*
