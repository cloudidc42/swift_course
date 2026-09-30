# Part 65: Data Structures ใน Swift

## บทนำ

Data Structures (โครงสร้างข้อมูล) เป็นรากฐานสำคัญของการเขียนโปรแกรมที่มีประสิทธิภาพ การเลือกใช้โครงสร้างข้อมูลที่เหมาะสมกับปัญหาสามารถส่งผลต่อประสิทธิภาพของโปรแกรมได้อย่างมาก ในบทนี้เราจะเรียนรู้โครงสร้างข้อมูลพื้นฐานและขั้นสูงพร้อม implementation ใน Swift

---

## 1. Big O Notation Review

### ความหมายของ Big O

Big O Notation เป็นสัญกรณ์ที่ใช้อธิบาย **Asymptotic Complexity** ของ Algorithm — คือการวัดประสิทธิภาพในแง่เวลา (Time Complexity) และพื้นที่หน่วยความจำ (Space Complexity) เมื่อ input มีขนาดใหญ่ขึ้น

```swift
// ตัวอย่างการวิเคราะห์ Big O

// O(1) - Constant Time: ไม่ขึ้นกับขนาด input
func getFirstElement(_ array: [Int]) -> Int? {
    return array.first  // เข้าถึงได้ทันที ไม่ว่า array จะใหญ่แค่ไหน
}

// O(log n) - Logarithmic Time: แบ่งครึ่งทุกรอบ
func binarySearch(_ array: [Int], target: Int) -> Int? {
    var left = 0
    var right = array.count - 1
    while left <= right {
        let mid = left + (right - left) / 2
        if array[mid] == target { return mid }
        else if array[mid] < target { left = mid + 1 }
        else { right = mid - 1 }
    }
    return nil
}

// O(n) - Linear Time: วนซ้ำแต่ละ element ครั้งเดียว
func linearSearch(_ array: [Int], target: Int) -> Int? {
    for (index, element) in array.enumerated() {
        if element == target { return index }
    }
    return nil
}

// O(n log n) - Linearithmic: Merge Sort, Quick Sort (average)
func mergeSort(_ array: [Int]) -> [Int] {
    guard array.count > 1 else { return array }
    let mid = array.count / 2
    let left = mergeSort(Array(array[..<mid]))
    let right = mergeSort(Array(array[mid...]))
    return merge(left, right)
}

func merge(_ left: [Int], _ right: [Int]) -> [Int] {
    var result: [Int] = []
    var i = 0, j = 0
    while i < left.count && j < right.count {
        if left[i] <= right[j] {
            result.append(left[i])
            i += 1
        } else {
            result.append(right[j])
            j += 1
        }
    }
    result.append(contentsOf: left[i...])
    result.append(contentsOf: right[j...])
    return result
}

// O(n²) - Quadratic Time: nested loops
func bubbleSort(_ array: inout [Int]) {
    let n = array.count
    for i in 0..<n {
        for j in 0..<(n - i - 1) {
            if array[j] > array[j + 1] {
                array.swapAt(j, j + 1)
            }
        }
    }
}

// O(2^n) - Exponential: naive recursive Fibonacci
func fibNaive(_ n: Int) -> Int {
    if n <= 1 { return n }
    return fibNaive(n - 1) + fibNaive(n - 2)
}

// O(n!) - Factorial: all permutations
func permutations(_ array: [Int]) -> [[Int]] {
    guard array.count > 1 else { return [array] }
    var result: [[Int]] = []
    for (i, element) in array.enumerated() {
        var remaining = array
        remaining.remove(at: i)
        permutations(remaining).forEach { result.append([element] + $0) }
    }
    return result
}
```

### ตารางเปรียบเทียบ Big O

| Complexity | Name | ตัวอย่าง |
|------------|------|---------|
| O(1) | Constant | Array access by index |
| O(log n) | Logarithmic | Binary search |
| O(n) | Linear | Linear search |
| O(n log n) | Linearithmic | Merge sort |
| O(n²) | Quadratic | Bubble sort |
| O(2^n) | Exponential | Recursive Fibonacci |
| O(n!) | Factorial | All permutations |

### Space Complexity

```swift
// O(1) Space - ใช้พื้นที่คงที่
func sumArray(_ array: [Int]) -> Int {
    var sum = 0  // ใช้แค่ตัวแปรเดียว
    for num in array {
        sum += num
    }
    return sum
}

// O(n) Space - ใช้พื้นที่ตามขนาด input
func copyArray(_ array: [Int]) -> [Int] {
    return array  // สร้าง array ใหม่ขนาดเท่ากัน
}

// O(n²) Space - matrix
func createMatrix(_ n: Int) -> [[Int]] {
    return Array(repeating: Array(repeating: 0, count: n), count: n)
}
```

---

## 2. Arrays (อาร์เรย์)

### Static Array vs Dynamic Array

Array ใน Swift เป็น **Dynamic Array** ที่สามารถปรับขนาดได้อัตโนมัติ

```swift
// MARK: - Static Array (ขนาดคงที่)
// ใน Swift ไม่มี static array โดยตรง แต่สามารถจำลองได้

struct StaticArray<T> {
    private var storage: [T?]
    private let capacity: Int
    private(set) var count: Int = 0
    
    init(capacity: Int) {
        self.capacity = capacity
        self.storage = Array(repeating: nil, count: capacity)
    }
    
    mutating func append(_ element: T) throws {
        guard count < capacity else {
            throw ArrayError.overflow
        }
        storage[count] = element
        count += 1
    }
    
    func get(at index: Int) throws -> T {
        guard index >= 0 && index < count else {
            throw ArrayError.indexOutOfBounds
        }
        return storage[index]!
    }
    
    mutating func set(_ element: T, at index: Int) throws {
        guard index >= 0 && index < count else {
            throw ArrayError.indexOutOfBounds
        }
        storage[index] = element
    }
    
    enum ArrayError: Error {
        case overflow
        case indexOutOfBounds
    }
}

// การใช้งาน Static Array
var staticArr = StaticArray<Int>(capacity: 5)
try? staticArr.append(1)
try? staticArr.append(2)
try? staticArr.append(3)
print("Static Array count: \(staticArr.count)")  // 3

// MARK: - Dynamic Array (ขนาดปรับได้)
// Swift Array เป็น Dynamic Array อยู่แล้ว

class DynamicArray<T> {
    private var storage: [T] = []
    private var _capacity: Int = 1
    
    var count: Int { storage.count }
    var capacity: Int { _capacity }
    var isEmpty: Bool { storage.isEmpty }
    
    // O(1) amortized - บางครั้งต้อง resize
    func append(_ element: T) {
        if storage.count >= _capacity {
            _capacity *= 2
            // Swift จัดการ resize อัตโนมัติ
        }
        storage.append(element)
    }
    
    // O(1) - เข้าถึง index โดยตรง
    subscript(index: Int) -> T {
        get { storage[index] }
        set { storage[index] = newValue }
    }
    
    // O(n) - ต้องเลื่อน elements
    func insert(_ element: T, at index: Int) {
        storage.insert(element, at: index)
    }
    
    // O(n) - ต้องเลื่อน elements
    func remove(at index: Int) -> T {
        return storage.remove(at: index)
    }
    
    // O(n) - ค้นหาแบบ linear
    func indexOf(_ element: T, comparator: (T, T) -> Bool) -> Int? {
        for (i, e) in storage.enumerated() {
            if comparator(e, element) { return i }
        }
        return nil
    }
}

// MARK: - Array Operations ใน Swift

var numbers = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3]

// Sorting - O(n log n)
let sorted = numbers.sorted()
let sortedDesc = numbers.sorted(by: >)

// Filtering - O(n)
let evenNumbers = numbers.filter { $0 % 2 == 0 }

// Mapping - O(n)
let doubled = numbers.map { $0 * 2 }

// Reducing - O(n)
let sum = numbers.reduce(0, +)

// Binary Search (requires sorted array) - O(log n)
let sortedNumbers = numbers.sorted()
if let index = sortedNumbers.firstIndex(of: 5) {
    print("Found 5 at index \(index)")
}

// Partition
var arr = [1, 2, 3, 4, 5, 6, 7, 8]
let partitionIndex = arr.partition { $0 > 4 }
print("Partition at: \(partitionIndex)")
print("Elements ≤ 4: \(arr[..<partitionIndex])")
print("Elements > 4: \(arr[partitionIndex...])")
```

### Time Complexity ของ Array Operations

| Operation | Best Case | Average Case | Worst Case |
|-----------|-----------|--------------|------------|
| Access | O(1) | O(1) | O(1) |
| Search | O(1) | O(n) | O(n) |
| Insert (end) | O(1) | O(1) amortized | O(n) |
| Insert (middle) | O(n) | O(n) | O(n) |
| Delete (end) | O(1) | O(1) | O(1) |
| Delete (middle) | O(n) | O(n) | O(n) |

---

## 3. Linked Lists (ลิสต์เชื่อมโยง)

### Singly Linked List (ลิสต์เชื่อมโยงทางเดียว)

```swift
// MARK: - Node
class SinglyNode<T> {
    var value: T
    var next: SinglyNode<T>?
    
    init(_ value: T) {
        self.value = value
        self.next = nil
    }
}

// MARK: - Singly Linked List
class SinglyLinkedList<T: Equatable> {
    private var head: SinglyNode<T>?
    private var tail: SinglyNode<T>?
    private(set) var count: Int = 0
    
    var isEmpty: Bool { head == nil }
    
    // O(1) - เพิ่มที่หัว
    func prepend(_ value: T) {
        let newNode = SinglyNode(value)
        if isEmpty {
            head = newNode
            tail = newNode
        } else {
            newNode.next = head
            head = newNode
        }
        count += 1
    }
    
    // O(1) - เพิ่มที่ท้าย (มี tail pointer)
    func append(_ value: T) {
        let newNode = SinglyNode(value)
        if isEmpty {
            head = newNode
            tail = newNode
        } else {
            tail?.next = newNode
            tail = newNode
        }
        count += 1
    }
    
    // O(n) - เพิ่มที่ตำแหน่งกำหนด
    func insert(_ value: T, at index: Int) {
        guard index >= 0 && index <= count else { return }
        if index == 0 {
            prepend(value)
            return
        }
        if index == count {
            append(value)
            return
        }
        var current = head
        for _ in 0..<(index - 1) {
            current = current?.next
        }
        let newNode = SinglyNode(value)
        newNode.next = current?.next
        current?.next = newNode
        count += 1
    }
    
    // O(1) - ลบที่หัว
    @discardableResult
    func removeFirst() -> T? {
        guard let node = head else { return nil }
        head = node.next
        if head == nil { tail = nil }
        count -= 1
        return node.value
    }
    
    // O(n) - ลบที่ท้าย (ต้องวนหาตัวก่อนหน้า)
    @discardableResult
    func removeLast() -> T? {
        guard !isEmpty else { return nil }
        if head === tail {
            let value = head?.value
            head = nil
            tail = nil
            count -= 1
            return value
        }
        var current = head
        while current?.next !== tail {
            current = current?.next
        }
        let value = tail?.value
        current?.next = nil
        tail = current
        count -= 1
        return value
    }
    
    // O(n) - ค้นหา
    func contains(_ value: T) -> Bool {
        var current = head
        while let node = current {
            if node.value == value { return true }
            current = node.next
        }
        return false
    }
    
    // O(n) - แปลงเป็น Array
    func toArray() -> [T] {
        var result: [T] = []
        var current = head
        while let node = current {
            result.append(node.value)
            current = node.next
        }
        return result
    }
    
    // O(n) - Reverse linked list (in-place)
    func reverse() {
        var prev: SinglyNode<T>? = nil
        var current = head
        tail = head
        while let node = current {
            let next = node.next
            node.next = prev
            prev = node
            current = next
        }
        head = prev
    }
    
    // O(n) - หา middle node (Floyd's algorithm)
    func middleNode() -> SinglyNode<T>? {
        var slow = head
        var fast = head
        while fast?.next != nil {
            slow = slow?.next
            fast = fast?.next?.next
        }
        return slow
    }
    
    // O(n) - ตรวจสอบ cycle
    func hasCycle() -> Bool {
        var slow = head
        var fast = head
        while fast?.next != nil {
            slow = slow?.next
            fast = fast?.next?.next
            if slow === fast { return true }
        }
        return false
    }
}

// การใช้งาน
let list = SinglyLinkedList<Int>()
list.append(1)
list.append(2)
list.append(3)
list.prepend(0)
print("List: \(list.toArray())")  // [0, 1, 2, 3]
list.reverse()
print("Reversed: \(list.toArray())")  // [3, 2, 1, 0]
print("Middle: \(list.middleNode()?.value ?? -1)")  // 2 or 1
```

### Doubly Linked List (ลิสต์เชื่อมโยงสองทาง)

```swift
// MARK: - Doubly Node
class DoublyNode<T> {
    var value: T
    var next: DoublyNode<T>?
    weak var prev: DoublyNode<T>?  // weak เพื่อป้องกัน retain cycle
    
    init(_ value: T) {
        self.value = value
    }
}

// MARK: - Doubly Linked List
class DoublyLinkedList<T: Equatable> {
    private var head: DoublyNode<T>?
    private var tail: DoublyNode<T>?
    private(set) var count: Int = 0
    
    var isEmpty: Bool { head == nil }
    
    // O(1)
    func prepend(_ value: T) {
        let newNode = DoublyNode(value)
        if isEmpty {
            head = newNode
            tail = newNode
        } else {
            newNode.next = head
            head?.prev = newNode
            head = newNode
        }
        count += 1
    }
    
    // O(1)
    func append(_ value: T) {
        let newNode = DoublyNode(value)
        if isEmpty {
            head = newNode
            tail = newNode
        } else {
            newNode.prev = tail
            tail?.next = newNode
            tail = newNode
        }
        count += 1
    }
    
    // O(1) - ลบ node ที่รู้จัก (ข้อดีของ doubly)
    func remove(node: DoublyNode<T>) -> T {
        node.prev?.next = node.next
        node.next?.prev = node.prev
        if node === head { head = node.next }
        if node === tail { tail = node.prev }
        node.next = nil
        node.prev = nil
        count -= 1
        return node.value
    }
    
    // O(1)
    @discardableResult
    func removeFirst() -> T? {
        guard let node = head else { return nil }
        return remove(node: node)
    }
    
    // O(1) - ข้อดีของ doubly vs singly
    @discardableResult
    func removeLast() -> T? {
        guard let node = tail else { return nil }
        return remove(node: node)
    }
    
    // Traverse ย้อนหลัง - ข้อดีของ doubly
    func toArrayReversed() -> [T] {
        var result: [T] = []
        var current = tail
        while let node = current {
            result.append(node.value)
            current = node.prev
        }
        return result
    }
}
```

### Circular Linked List (ลิสต์วงกลม)

```swift
// MARK: - Circular Linked List
class CircularLinkedList<T: Equatable> {
    private var tail: SinglyNode<T>?  // tail.next ชี้ไปที่ head
    private(set) var count: Int = 0
    
    var isEmpty: Bool { tail == nil }
    var head: SinglyNode<T>? { tail?.next }
    
    // O(1)
    func append(_ value: T) {
        let newNode = SinglyNode(value)
        if isEmpty {
            newNode.next = newNode  // ชี้ไปที่ตัวเอง
            tail = newNode
        } else {
            newNode.next = tail?.next  // newNode ชี้ไป head
            tail?.next = newNode       // tail ชี้ไป newNode
            tail = newNode             // อัพเดต tail
        }
        count += 1
    }
    
    // O(1)
    func prepend(_ value: T) {
        let newNode = SinglyNode(value)
        if isEmpty {
            newNode.next = newNode
            tail = newNode
        } else {
            newNode.next = tail?.next  // newNode ชี้ไป head เดิม
            tail?.next = newNode       // tail ชี้ไป newNode
        }
        count += 1
    }
    
    // O(1) - ลบ head
    @discardableResult
    func removeFirst() -> T? {
        guard !isEmpty else { return nil }
        if tail?.next === tail {  // มีแค่ node เดียว
            let value = tail?.value
            tail = nil
            count -= 1
            return value
        }
        let value = tail?.next?.value
        tail?.next = tail?.next?.next
        count -= 1
        return value
    }
    
    // ใช้งาน: Josephus Problem
    func josephus(k: Int) -> T? {
        guard !isEmpty else { return nil }
        var current = tail
        while count > 1 {
            for _ in 0..<(k - 1) {
                current = current?.next
            }
            // ลบ node ถัดไป
            let toDelete = current?.next
            current?.next = toDelete?.next
            if toDelete === tail {
                tail = current
            }
            count -= 1
        }
        return current?.value
    }
}
```

### Time Complexity ของ Linked List

| Operation | Singly | Doubly |
|-----------|--------|--------|
| Access (by index) | O(n) | O(n) |
| Search | O(n) | O(n) |
| Insert at head | O(1) | O(1) |
| Insert at tail | O(1)* | O(1) |
| Insert at middle | O(n) | O(n) |
| Delete at head | O(1) | O(1) |
| Delete at tail | O(n) | O(1) |
| Delete (known node) | O(n) | O(1) |

---

## 4. Stacks (สแตก)

Stack ใช้หลักการ **LIFO** (Last In, First Out) — element ที่ push เข้ามาหลังสุดจะ pop ออกก่อน

```swift
// MARK: - Stack Implementation

// วิธีที่ 1: ใช้ Array
struct ArrayStack<T> {
    private var storage: [T] = []
    
    var isEmpty: Bool { storage.isEmpty }
    var count: Int { storage.count }
    var top: T? { storage.last }
    
    // O(1) amortized
    mutating func push(_ element: T) {
        storage.append(element)
    }
    
    // O(1)
    @discardableResult
    mutating func pop() -> T? {
        return storage.popLast()
    }
    
    // O(1)
    func peek() -> T? {
        return storage.last
    }
}

// วิธีที่ 2: ใช้ Linked List (no amortized cost)
class LinkedStack<T> {
    private class Node {
        let value: T
        var next: Node?
        init(_ value: T) { self.value = value }
    }
    
    private var top: Node?
    private(set) var count: Int = 0
    
    var isEmpty: Bool { top == nil }
    
    // O(1) - always
    func push(_ element: T) {
        let node = Node(element)
        node.next = top
        top = node
        count += 1
    }
    
    // O(1) - always
    @discardableResult
    func pop() -> T? {
        guard let node = top else { return nil }
        top = node.next
        count -= 1
        return node.value
    }
    
    func peek() -> T? { top?.value }
}

// MARK: - Stack Applications

// 1. ตรวจสอบ Balanced Parentheses
func isBalanced(_ s: String) -> Bool {
    var stack = ArrayStack<Character>()
    let matching: [Character: Character] = [")": "(", "]": "[", "}": "{"]
    
    for char in s {
        switch char {
        case "(", "[", "{":
            stack.push(char)
        case ")", "]", "}":
            guard let top = stack.pop(), top == matching[char] else {
                return false
            }
        default:
            break
        }
    }
    return stack.isEmpty
}

print(isBalanced("({[]})"))  // true
print(isBalanced("({[})"))   // false

// 2. Evaluate Reverse Polish Notation
func evalRPN(_ tokens: [String]) -> Int {
    var stack = ArrayStack<Int>()
    for token in tokens {
        if let num = Int(token) {
            stack.push(num)
        } else {
            let b = stack.pop()!
            let a = stack.pop()!
            switch token {
            case "+": stack.push(a + b)
            case "-": stack.push(a - b)
            case "*": stack.push(a * b)
            case "/": stack.push(a / b)
            default: break
            }
        }
    }
    return stack.pop() ?? 0
}

print(evalRPN(["2", "1", "+", "3", "*"]))  // (2+1)*3 = 9

// 3. Min Stack - stack ที่รู้จัก minimum ตลอดเวลา
class MinStack {
    private var stack: [(value: Int, currentMin: Int)] = []
    
    var top: Int? { stack.last?.value }
    var minimum: Int? { stack.last?.currentMin }
    
    func push(_ val: Int) {
        let currentMin = stack.isEmpty ? val : min(val, stack.last!.currentMin)
        stack.append((val, currentMin))
    }
    
    func pop() -> Int? {
        return stack.popLast()?.value
    }
}

let minStack = MinStack()
minStack.push(5)
minStack.push(3)
minStack.push(7)
minStack.push(1)
print("Min: \(minStack.minimum!)")  // 1
minStack.pop()
print("Min after pop: \(minStack.minimum!)")  // 3

// 4. Next Greater Element
func nextGreaterElement(_ nums: [Int]) -> [Int] {
    var result = Array(repeating: -1, count: nums.count)
    var stack = ArrayStack<Int>()  // เก็บ indices
    
    for i in 0..<nums.count {
        while !stack.isEmpty && nums[stack.top!] < nums[i] {
            let idx = stack.pop()!
            result[idx] = nums[i]
        }
        stack.push(i)
    }
    return result
}

print(nextGreaterElement([4, 1, 2]))  // [-1, 2, -1]
print(nextGreaterElement([1, 3, 2])) // [3, -1, -1]
```

---

## 5. Queues (คิว)

Queue ใช้หลักการ **FIFO** (First In, First Out) — element ที่ enqueue ก่อนจะ dequeue ออกก่อน

```swift
// MARK: - Regular Queue

// วิธีที่ 1: ใช้ Array (ไม่มีประสิทธิภาพ - dequeue เป็น O(n))
struct NaiveQueue<T> {
    private var storage: [T] = []
    
    var isEmpty: Bool { storage.isEmpty }
    var count: Int { storage.count }
    var front: T? { storage.first }
    
    mutating func enqueue(_ element: T) {
        storage.append(element)
    }
    
    // O(n) - ต้อง shift elements
    @discardableResult
    mutating func dequeue() -> T? {
        return isEmpty ? nil : storage.removeFirst()
    }
}

// วิธีที่ 2: ใช้ Two Stacks (O(1) amortized)
struct TwoStackQueue<T> {
    private var inStack: [T] = []
    private var outStack: [T] = []
    
    var isEmpty: Bool { inStack.isEmpty && outStack.isEmpty }
    var count: Int { inStack.count + outStack.count }
    
    // O(1)
    mutating func enqueue(_ element: T) {
        inStack.append(element)
    }
    
    // O(1) amortized
    @discardableResult
    mutating func dequeue() -> T? {
        if outStack.isEmpty {
            outStack = inStack.reversed()
            inStack.removeAll()
        }
        return outStack.popLast()
    }
    
    var front: T? {
        return outStack.last ?? inStack.first
    }
}

// วิธีที่ 3: ใช้ Linked List (O(1) always)
class LinkedQueue<T> {
    private class Node {
        let value: T
        var next: Node?
        init(_ value: T) { self.value = value }
    }
    
    private var head: Node?
    private var tail: Node?
    private(set) var count: Int = 0
    
    var isEmpty: Bool { head == nil }
    var front: T? { head?.value }
    var rear: T? { tail?.value }
    
    // O(1)
    func enqueue(_ element: T) {
        let node = Node(element)
        if isEmpty {
            head = node
            tail = node
        } else {
            tail?.next = node
            tail = node
        }
        count += 1
    }
    
    // O(1)
    @discardableResult
    func dequeue() -> T? {
        guard let node = head else { return nil }
        head = node.next
        if head == nil { tail = nil }
        count -= 1
        return node.value
    }
}

// MARK: - Deque (Double-Ended Queue)

struct Deque<T> {
    private var storage: [T] = []
    
    var isEmpty: Bool { storage.isEmpty }
    var count: Int { storage.count }
    var front: T? { storage.first }
    var back: T? { storage.last }
    
    mutating func addFront(_ element: T) { storage.insert(element, at: 0) }
    mutating func addBack(_ element: T) { storage.append(element) }
    
    @discardableResult
    mutating func removeFront() -> T? {
        return isEmpty ? nil : storage.removeFirst()
    }
    
    @discardableResult
    mutating func removeBack() -> T? {
        return storage.popLast()
    }
}

// MARK: - Circular Queue (Ring Buffer)

struct CircularQueue<T> {
    private var storage: [T?]
    private var head: Int = 0
    private var tail: Int = 0
    private(set) var count: Int = 0
    private let capacity: Int
    
    init(capacity: Int) {
        self.capacity = capacity
        self.storage = Array(repeating: nil, count: capacity)
    }
    
    var isEmpty: Bool { count == 0 }
    var isFull: Bool { count == capacity }
    
    // O(1)
    mutating func enqueue(_ element: T) -> Bool {
        guard !isFull else { return false }
        storage[tail] = element
        tail = (tail + 1) % capacity
        count += 1
        return true
    }
    
    // O(1)
    @discardableResult
    mutating func dequeue() -> T? {
        guard !isEmpty else { return nil }
        let element = storage[head]
        storage[head] = nil
        head = (head + 1) % capacity
        count -= 1
        return element
    }
    
    var front: T? { isEmpty ? nil : storage[head] }
    var rear: T? { isEmpty ? nil : storage[(tail - 1 + capacity) % capacity] }
}

// MARK: - Priority Queue

struct PriorityQueue<T> {
    private var heap: [T]
    private let comparator: (T, T) -> Bool
    
    var isEmpty: Bool { heap.isEmpty }
    var count: Int { heap.count }
    var top: T? { heap.first }
    
    init(comparator: @escaping (T, T) -> Bool) {
        self.heap = []
        self.comparator = comparator
    }
    
    // O(log n)
    mutating func enqueue(_ element: T) {
        heap.append(element)
        siftUp(heap.count - 1)
    }
    
    // O(log n)
    @discardableResult
    mutating func dequeue() -> T? {
        guard !isEmpty else { return nil }
        if heap.count == 1 { return heap.removeLast() }
        let top = heap[0]
        heap[0] = heap.removeLast()
        siftDown(0)
        return top
    }
    
    private mutating func siftUp(_ index: Int) {
        var child = index
        while child > 0 {
            let parent = (child - 1) / 2
            if comparator(heap[child], heap[parent]) {
                heap.swapAt(child, parent)
                child = parent
            } else {
                break
            }
        }
    }
    
    private mutating func siftDown(_ index: Int) {
        let n = heap.count
        var parent = index
        while true {
            let left = 2 * parent + 1
            let right = 2 * parent + 2
            var target = parent
            if left < n && comparator(heap[left], heap[target]) { target = left }
            if right < n && comparator(heap[right], heap[target]) { target = right }
            if target == parent { break }
            heap.swapAt(parent, target)
            parent = target
        }
    }
}

// การใช้งาน Priority Queue
var maxPQ = PriorityQueue<Int>(comparator: >)  // Max heap
maxPQ.enqueue(3)
maxPQ.enqueue(1)
maxPQ.enqueue(4)
maxPQ.enqueue(1)
maxPQ.enqueue(5)
print("Max PQ top: \(maxPQ.top!)")   // 5
maxPQ.dequeue()
print("After dequeue: \(maxPQ.top!)")  // 4

// Queue Application: BFS
func bfs(graph: [Int: [Int]], start: Int) -> [Int] {
    var visited: Set<Int> = [start]
    var queue = LinkedQueue<Int>()
    var order: [Int] = []
    queue.enqueue(start)
    
    while let node = queue.dequeue() {
        order.append(node)
        for neighbor in graph[node] ?? [] {
            if !visited.contains(neighbor) {
                visited.insert(neighbor)
                queue.enqueue(neighbor)
            }
        }
    }
    return order
}
```

---

## 6. Hash Tables / Hash Maps (ตารางแฮช)

```swift
// MARK: - Hash Table Implementation

struct HashTable<Key: Hashable, Value> {
    private typealias Element = (key: Key, value: Value)
    private typealias Bucket = [Element]
    private var buckets: [Bucket]
    private(set) var count: Int = 0
    
    var isEmpty: Bool { count == 0 }
    private let loadFactorThreshold: Double = 0.75
    
    init(capacity: Int = 16) {
        buckets = Array(repeating: [], count: capacity)
    }
    
    // O(1) average
    private func index(for key: Key) -> Int {
        return abs(key.hashValue) % buckets.count
    }
    
    // O(1) average, O(n) worst case (many collisions)
    subscript(key: Key) -> Value? {
        get {
            let i = index(for: key)
            return buckets[i].first { $0.key == key }?.value
        }
        set {
            if let value = newValue {
                updateValue(value, forKey: key)
            } else {
                removeValue(forKey: key)
            }
        }
    }
    
    @discardableResult
    mutating func updateValue(_ value: Value, forKey key: Key) -> Value? {
        let i = index(for: key)
        if let idx = buckets[i].firstIndex(where: { $0.key == key }) {
            let old = buckets[i][idx].value
            buckets[i][idx].value = value
            return old
        } else {
            buckets[i].append((key, value))
            count += 1
            if Double(count) / Double(buckets.count) > loadFactorThreshold {
                resize()
            }
            return nil
        }
    }
    
    @discardableResult
    mutating func removeValue(forKey key: Key) -> Value? {
        let i = index(for: key)
        guard let idx = buckets[i].firstIndex(where: { $0.key == key }) else {
            return nil
        }
        let old = buckets[i][idx].value
        buckets[i].remove(at: idx)
        count -= 1
        return old
    }
    
    // Resize เมื่อ load factor สูงเกินไป
    private mutating func resize() {
        let newBuckets = Array(repeating: Bucket(), count: buckets.count * 2)
        var newTable = HashTable(capacity: newBuckets.count)
        for bucket in buckets {
            for element in bucket {
                newTable.updateValue(element.value, forKey: element.key)
            }
        }
        self = newTable
    }
    
    var keys: [Key] {
        buckets.flatMap { $0.map { $0.key } }
    }
    
    var values: [Value] {
        buckets.flatMap { $0.map { $0.value } }
    }
}

// MARK: - Collision Resolution Strategies

// 1. Separate Chaining (ใช้ด้านบน)
// 2. Open Addressing - Linear Probing

struct OpenAddressHashTable<Key: Hashable, Value> {
    private enum Slot {
        case empty
        case deleted
        case occupied(key: Key, value: Value)
    }
    
    private var slots: [Slot]
    private var count: Int = 0
    private let capacity: Int
    
    init(capacity: Int = 16) {
        self.capacity = capacity
        self.slots = Array(repeating: .empty, count: capacity)
    }
    
    private func probe(_ key: Key, from start: Int) -> Int {
        var i = start
        while case .occupied(let k, _) = slots[i], k != key {
            i = (i + 1) % capacity  // Linear Probing
        }
        return i
    }
    
    subscript(key: Key) -> Value? {
        get {
            let start = abs(key.hashValue) % capacity
            let i = probe(key, from: start)
            if case .occupied(_, let value) = slots[i] { return value }
            return nil
        }
        set {
            let start = abs(key.hashValue) % capacity
            let i = probe(key, from: start)
            if let value = newValue {
                slots[i] = .occupied(key: key, value: value)
                count += 1
            } else if case .occupied = slots[i] {
                slots[i] = .deleted
                count -= 1
            }
        }
    }
}

// MARK: - Hash Table Use Cases

// 1. Two Sum
func twoSum(_ nums: [Int], _ target: Int) -> [Int] {
    var seen: [Int: Int] = [:]  // value -> index
    for (i, num) in nums.enumerated() {
        let complement = target - num
        if let j = seen[complement] {
            return [j, i]
        }
        seen[num] = i
    }
    return []
}

print(twoSum([2, 7, 11, 15], 9))  // [0, 1]

// 2. Group Anagrams
func groupAnagrams(_ strs: [String]) -> [[String]] {
    var groups: [String: [String]] = [:]
    for str in strs {
        let key = String(str.sorted())  // sorted characters as key
        groups[key, default: []].append(str)
    }
    return Array(groups.values)
}

print(groupAnagrams(["eat","tea","tan","ate","nat","bat"]))

// 3. LRU Cache
class LRUCache {
    private var cache: [Int: Int] = [:]
    private var order: [Int] = []  // tail = most recent
    private let capacity: Int
    
    init(_ capacity: Int) {
        self.capacity = capacity
    }
    
    func get(_ key: Int) -> Int {
        guard let value = cache[key] else { return -1 }
        order.removeAll { $0 == key }
        order.append(key)
        return value
    }
    
    func put(_ key: Int, _ value: Int) {
        if cache[key] != nil {
            order.removeAll { $0 == key }
        } else if cache.count >= capacity {
            let lru = order.removeFirst()
            cache.removeValue(forKey: lru)
        }
        cache[key] = value
        order.append(key)
    }
}

let lru = LRUCache(3)
lru.put(1, 1); lru.put(2, 2); lru.put(3, 3)
lru.get(1)       // access 1
lru.put(4, 4)    // evicts 2
print(lru.get(2)) // -1 (evicted)
print(lru.get(3)) // 3

// Swift Dictionary - built-in hash map
var dict: [String: Int] = [:]
dict["apple"] = 5
dict["banana"] = 3
dict["cherry", default: 0] += 1

// Dictionary operations
let keys = dict.keys.sorted()
let values = dict.values
let filtered = dict.filter { $0.value > 3 }
let mapped = dict.mapValues { $0 * 2 }
```

---

## 7. Trees (ต้นไม้)

### Binary Tree (ต้นไม้ไบนารี)

```swift
// MARK: - Binary Tree Node
class TreeNode<T> {
    var value: T
    var left: TreeNode<T>?
    var right: TreeNode<T>?
    
    init(_ value: T) {
        self.value = value
    }
}

// MARK: - Tree Traversals

class BinaryTree<T> {
    var root: TreeNode<T>?
    
    // In-order: Left -> Root -> Right (gives sorted order for BST)
    func inOrder(_ node: TreeNode<T>?, _ result: inout [T]) {
        guard let node = node else { return }
        inOrder(node.left, &result)
        result.append(node.value)
        inOrder(node.right, &result)
    }
    
    // Pre-order: Root -> Left -> Right (useful for copying tree)
    func preOrder(_ node: TreeNode<T>?, _ result: inout [T]) {
        guard let node = node else { return }
        result.append(node.value)
        preOrder(node.left, &result)
        preOrder(node.right, &result)
    }
    
    // Post-order: Left -> Right -> Root (useful for deleting tree)
    func postOrder(_ node: TreeNode<T>?, _ result: inout [T]) {
        guard let node = node else { return }
        postOrder(node.left, &result)
        postOrder(node.right, &result)
        result.append(node.value)
    }
    
    // Level-order (BFS)
    func levelOrder() -> [[T]] {
        guard let root = root else { return [] }
        var result: [[T]] = []
        var queue = [root]
        
        while !queue.isEmpty {
            let levelSize = queue.count
            var level: [T] = []
            for _ in 0..<levelSize {
                let node = queue.removeFirst()
                level.append(node.value)
                if let left = node.left { queue.append(left) }
                if let right = node.right { queue.append(right) }
            }
            result.append(level)
        }
        return result
    }
    
    // Height of tree
    func height(_ node: TreeNode<T>?) -> Int {
        guard let node = node else { return -1 }
        return 1 + max(height(node.left), height(node.right))
    }
    
    // Count nodes
    func count(_ node: TreeNode<T>?) -> Int {
        guard let node = node else { return 0 }
        return 1 + count(node.left) + count(node.right)
    }
    
    // Is Balanced
    func isBalanced(_ node: TreeNode<T>?) -> Bool {
        func checkHeight(_ node: TreeNode<T>?) -> Int {
            guard let node = node else { return 0 }
            let leftH = checkHeight(node.left)
            if leftH == -1 { return -1 }
            let rightH = checkHeight(node.right)
            if rightH == -1 { return -1 }
            if abs(leftH - rightH) > 1 { return -1 }
            return max(leftH, rightH) + 1
        }
        return checkHeight(node) != -1
    }
}
```

### Binary Search Tree (BST)

```swift
// MARK: - Binary Search Tree
class BST<T: Comparable> {
    private var root: TreeNode<T>?
    
    // O(log n) average, O(n) worst case (unbalanced)
    func insert(_ value: T) {
        root = insert(root, value)
    }
    
    private func insert(_ node: TreeNode<T>?, _ value: T) -> TreeNode<T> {
        guard let node = node else { return TreeNode(value) }
        if value < node.value {
            node.left = insert(node.left, value)
        } else if value > node.value {
            node.right = insert(node.right, value)
        }
        return node  // duplicate ignored
    }
    
    // O(log n) average
    func contains(_ value: T) -> Bool {
        var current = root
        while let node = current {
            if value == node.value { return true }
            current = value < node.value ? node.left : node.right
        }
        return false
    }
    
    // O(log n) average
    func delete(_ value: T) {
        root = delete(root, value)
    }
    
    private func delete(_ node: TreeNode<T>?, _ value: T) -> TreeNode<T>? {
        guard let node = node else { return nil }
        if value < node.value {
            node.left = delete(node.left, value)
        } else if value > node.value {
            node.right = delete(node.right, value)
        } else {
            // Found node to delete
            if node.left == nil { return node.right }
            if node.right == nil { return node.left }
            // Has two children: replace with in-order successor (min of right subtree)
            let minNode = findMin(node.right!)
            node.value = minNode.value
            node.right = delete(node.right, minNode.value)
        }
        return node
    }
    
    private func findMin(_ node: TreeNode<T>) -> TreeNode<T> {
        var current = node
        while let left = current.left { current = left }
        return current
    }
    
    // Validate BST
    func isValidBST() -> Bool {
        return validate(root, min: nil, max: nil)
    }
    
    private func validate(_ node: TreeNode<T>?, min: T?, max: T?) -> Bool {
        guard let node = node else { return true }
        if let min = min, node.value <= min { return false }
        if let max = max, node.value >= max { return false }
        return validate(node.left, min: min, max: node.value) &&
               validate(node.right, min: node.value, max: max)
    }
    
    // In-order traversal gives sorted output
    func toSortedArray() -> [T] {
        var result: [T] = []
        func inOrder(_ node: TreeNode<T>?) {
            guard let node = node else { return }
            inOrder(node.left)
            result.append(node.value)
            inOrder(node.right)
        }
        inOrder(root)
        return result
    }
}

let bst = BST<Int>()
[5, 3, 7, 1, 4, 6, 8].forEach { bst.insert($0) }
print("BST sorted: \(bst.toSortedArray())")  // [1, 3, 4, 5, 6, 7, 8]
print("Contains 4: \(bst.contains(4))")       // true
bst.delete(3)
print("After delete 3: \(bst.toSortedArray())")  // [1, 4, 5, 6, 7, 8]
```

### AVL Tree (Self-Balancing BST)

```swift
// MARK: - AVL Tree Node
class AVLNode<T: Comparable> {
    var value: T
    var left: AVLNode<T>?
    var right: AVLNode<T>?
    var height: Int = 1
    
    init(_ value: T) { self.value = value }
}

class AVLTree<T: Comparable> {
    private var root: AVLNode<T>?
    
    private func height(_ node: AVLNode<T>?) -> Int {
        node?.height ?? 0
    }
    
    private func balanceFactor(_ node: AVLNode<T>?) -> Int {
        guard let node = node else { return 0 }
        return height(node.left) - height(node.right)
    }
    
    private func updateHeight(_ node: AVLNode<T>) {
        node.height = 1 + max(height(node.left), height(node.right))
    }
    
    // Right Rotation
    private func rotateRight(_ y: AVLNode<T>) -> AVLNode<T> {
        let x = y.left!
        let T2 = x.right
        x.right = y
        y.left = T2
        updateHeight(y)
        updateHeight(x)
        return x
    }
    
    // Left Rotation
    private func rotateLeft(_ x: AVLNode<T>) -> AVLNode<T> {
        let y = x.right!
        let T2 = y.left
        y.left = x
        x.right = T2
        updateHeight(x)
        updateHeight(y)
        return y
    }
    
    // Balance node
    private func balance(_ node: AVLNode<T>) -> AVLNode<T> {
        updateHeight(node)
        let bf = balanceFactor(node)
        
        // Left-Left case
        if bf > 1 && balanceFactor(node.left) >= 0 {
            return rotateRight(node)
        }
        // Left-Right case
        if bf > 1 && balanceFactor(node.left) < 0 {
            node.left = rotateLeft(node.left!)
            return rotateRight(node)
        }
        // Right-Right case
        if bf < -1 && balanceFactor(node.right) <= 0 {
            return rotateLeft(node)
        }
        // Right-Left case
        if bf < -1 && balanceFactor(node.right) > 0 {
            node.right = rotateRight(node.right!)
            return rotateLeft(node)
        }
        return node
    }
    
    // O(log n) guaranteed
    func insert(_ value: T) {
        root = insert(root, value)
    }
    
    private func insert(_ node: AVLNode<T>?, _ value: T) -> AVLNode<T> {
        guard let node = node else { return AVLNode(value) }
        if value < node.value {
            node.left = insert(node.left, value)
        } else if value > node.value {
            node.right = insert(node.right, value)
        } else {
            return node  // duplicate
        }
        return balance(node)
    }
}
```

---

## 8. Heaps (ฮีป)

```swift
// MARK: - Generic Heap

struct Heap<T> {
    private var elements: [T]
    private let priorityFunction: (T, T) -> Bool
    
    var isEmpty: Bool { elements.isEmpty }
    var count: Int { elements.count }
    var peek: T? { elements.first }
    
    init(priorityFunction: @escaping (T, T) -> Bool) {
        self.elements = []
        self.priorityFunction = priorityFunction
    }
    
    // สร้าง heap จาก array - O(n)
    init(_ array: [T], priorityFunction: @escaping (T, T) -> Bool) {
        self.elements = array
        self.priorityFunction = priorityFunction
        buildHeap()
    }
    
    private mutating func buildHeap() {
        // Start from last non-leaf node
        for i in stride(from: elements.count / 2 - 1, through: 0, by: -1) {
            siftDown(i)
        }
    }
    
    // O(log n)
    mutating func insert(_ element: T) {
        elements.append(element)
        siftUp(elements.count - 1)
    }
    
    // O(log n)
    @discardableResult
    mutating func remove() -> T? {
        guard !isEmpty else { return nil }
        if elements.count == 1 { return elements.removeLast() }
        let top = elements[0]
        elements[0] = elements.removeLast()
        siftDown(0)
        return top
    }
    
    private mutating func siftUp(_ index: Int) {
        var child = index
        while child > 0 {
            let parent = (child - 1) / 2
            if priorityFunction(elements[child], elements[parent]) {
                elements.swapAt(child, parent)
                child = parent
            } else { break }
        }
    }
    
    private mutating func siftDown(_ index: Int) {
        var parent = index
        while true {
            let left = 2 * parent + 1
            let right = 2 * parent + 2
            var candidate = parent
            if left < count && priorityFunction(elements[left], elements[candidate]) {
                candidate = left
            }
            if right < count && priorityFunction(elements[right], elements[candidate]) {
                candidate = right
            }
            if candidate == parent { break }
            elements.swapAt(parent, candidate)
            parent = candidate
        }
    }
}

// Max Heap
var maxHeap = Heap<Int>(priorityFunction: >)
[3, 1, 4, 1, 5, 9, 2, 6].forEach { maxHeap.insert($0) }
print("Max: \(maxHeap.peek!)")  // 9
maxHeap.remove()
print("Next max: \(maxHeap.peek!)")  // 6

// Min Heap
var minHeap = Heap<Int>([3, 1, 4, 1, 5, 9, 2, 6], priorityFunction: <)
print("Min: \(minHeap.peek!)")  // 1

// MARK: - Heap Applications

// Heap Sort - O(n log n), in-place
func heapSort(_ array: inout [Int]) {
    var heap = Heap(array, priorityFunction: >)
    for i in stride(from: array.count - 1, through: 0, by: -1) {
        array[i] = heap.remove()!
    }
}

var arr = [5, 3, 8, 1, 9, 2]
heapSort(&arr)
print("Heap sorted: \(arr)")  // [1, 2, 3, 5, 8, 9]

// K-th Largest Element
func kthLargest(_ nums: [Int], _ k: Int) -> Int {
    var minHeap = Heap<Int>(priorityFunction: <)
    for num in nums {
        minHeap.insert(num)
        if minHeap.count > k {
            minHeap.remove()
        }
    }
    return minHeap.peek!
}

print("3rd largest in [3,2,1,5,6,4]: \(kthLargest([3,2,1,5,6,4], 2))")  // 5

// Merge K Sorted Lists
func mergeKSortedArrays(_ arrays: [[Int]]) -> [Int] {
    // (value, arrayIndex, elementIndex)
    var minHeap = Heap<(Int, Int, Int)>(priorityFunction: { $0.0 < $1.0 })
    var result: [Int] = []
    
    // Initialize with first element of each array
    for (i, array) in arrays.enumerated() {
        if !array.isEmpty {
            minHeap.insert((array[0], i, 0))
        }
    }
    
    while !minHeap.isEmpty {
        let (val, arrayIdx, elemIdx) = minHeap.remove()!
        result.append(val)
        let nextIdx = elemIdx + 1
        if nextIdx < arrays[arrayIdx].count {
            minHeap.insert((arrays[arrayIdx][nextIdx], arrayIdx, nextIdx))
        }
    }
    return result
}
```

---

## 9. Graphs (กราฟ)

```swift
// MARK: - Graph Representations

// Adjacency List - O(V+E) space
class Graph {
    let vertices: Int
    var adjList: [Int: [(vertex: Int, weight: Int)]]
    var isDirected: Bool
    
    init(vertices: Int, directed: Bool = false) {
        self.vertices = vertices
        self.isDirected = directed
        self.adjList = Dictionary(uniqueKeysWithValues: (0..<vertices).map { ($0, []) })
    }
    
    func addEdge(from u: Int, to v: Int, weight: Int = 1) {
        adjList[u]?.append((v, weight))
        if !isDirected {
            adjList[v]?.append((u, weight))
        }
    }
    
    // BFS - O(V + E)
    func bfs(from start: Int) -> [Int] {
        var visited = Set<Int>()
        var queue: [Int] = [start]
        var order: [Int] = []
        visited.insert(start)
        
        while !queue.isEmpty {
            let vertex = queue.removeFirst()
            order.append(vertex)
            for (neighbor, _) in adjList[vertex] ?? [] {
                if !visited.contains(neighbor) {
                    visited.insert(neighbor)
                    queue.append(neighbor)
                }
            }
        }
        return order
    }
    
    // DFS - O(V + E)
    func dfs(from start: Int) -> [Int] {
        var visited = Set<Int>()
        var order: [Int] = []
        
        func dfsHelper(_ vertex: Int) {
            visited.insert(vertex)
            order.append(vertex)
            for (neighbor, _) in adjList[vertex] ?? [] {
                if !visited.contains(neighbor) {
                    dfsHelper(neighbor)
                }
            }
        }
        dfsHelper(start)
        return order
    }
    
    // Dijkstra's Algorithm - O((V + E) log V)
    func dijkstra(from start: Int) -> [Int: Int] {
        var distances: [Int: Int] = Dictionary(uniqueKeysWithValues: (0..<vertices).map { ($0, Int.max) })
        distances[start] = 0
        var priorityQueue = Heap<(Int, Int)>(priorityFunction: { $0.1 < $1.1 })  // (vertex, distance)
        priorityQueue.insert((start, 0))
        
        while !priorityQueue.isEmpty {
            let (vertex, dist) = priorityQueue.remove()!
            if dist > distances[vertex]! { continue }  // outdated entry
            
            for (neighbor, weight) in adjList[vertex] ?? [] {
                let newDist = dist + weight
                if newDist < distances[neighbor]! {
                    distances[neighbor] = newDist
                    priorityQueue.insert((neighbor, newDist))
                }
            }
        }
        return distances
    }
    
    // Detect Cycle (Undirected)
    func hasCycleUndirected() -> Bool {
        var visited = Set<Int>()
        
        func dfs(_ v: Int, _ parent: Int) -> Bool {
            visited.insert(v)
            for (neighbor, _) in adjList[v] ?? [] {
                if !visited.contains(neighbor) {
                    if dfs(neighbor, v) { return true }
                } else if neighbor != parent {
                    return true
                }
            }
            return false
        }
        
        for v in 0..<vertices {
            if !visited.contains(v) {
                if dfs(v, -1) { return true }
            }
        }
        return false
    }
    
    // Topological Sort (Directed Acyclic Graph)
    func topologicalSort() -> [Int]? {
        var inDegree = [Int: Int]()
        for v in 0..<vertices { inDegree[v] = 0 }
        for v in 0..<vertices {
            for (neighbor, _) in adjList[v] ?? [] {
                inDegree[neighbor, default: 0] += 1
            }
        }
        
        var queue = inDegree.filter { $0.value == 0 }.map { $0.key }
        var order: [Int] = []
        
        while !queue.isEmpty {
            let v = queue.removeFirst()
            order.append(v)
            for (neighbor, _) in adjList[v] ?? [] {
                inDegree[neighbor]! -= 1
                if inDegree[neighbor]! == 0 {
                    queue.append(neighbor)
                }
            }
        }
        
        return order.count == vertices ? order : nil  // nil = has cycle
    }
    
    // Union-Find for Minimum Spanning Tree
    func kruskalMST() -> [(Int, Int, Int)] {
        var edges: [(weight: Int, u: Int, v: Int)] = []
        for u in 0..<vertices {
            for (v, w) in adjList[u] ?? [] where u < v {
                edges.append((w, u, v))
            }
        }
        edges.sort { $0.weight < $1.weight }
        
        var parent = Array(0..<vertices)
        var rank = Array(repeating: 0, count: vertices)
        
        func find(_ x: Int) -> Int {
            if parent[x] != x { parent[x] = find(parent[x]) }
            return parent[x]
        }
        
        func union(_ x: Int, _ y: Int) -> Bool {
            let px = find(x), py = find(y)
            if px == py { return false }
            if rank[px] < rank[py] { parent[px] = py }
            else if rank[px] > rank[py] { parent[py] = px }
            else { parent[py] = px; rank[px] += 1 }
            return true
        }
        
        var mst: [(Int, Int, Int)] = []
        for edge in edges {
            if union(edge.u, edge.v) {
                mst.append((edge.u, edge.v, edge.weight))
                if mst.count == vertices - 1 { break }
            }
        }
        return mst
    }
}

// การใช้งาน
let g = Graph(vertices: 5, directed: false)
g.addEdge(from: 0, to: 1, weight: 4)
g.addEdge(from: 0, to: 2, weight: 1)
g.addEdge(from: 2, to: 1, weight: 2)
g.addEdge(from: 1, to: 3, weight: 1)
g.addEdge(from: 2, to: 3, weight: 5)
g.addEdge(from: 3, to: 4, weight: 3)

print("BFS: \(g.bfs(from: 0))")
print("DFS: \(g.dfs(from: 0))")
let dist = g.dijkstra(from: 0)
print("Shortest from 0: \(dist)")
```

---

## 10. Sets (เซต)

```swift
// MARK: - Set Operations ใน Swift

var setA: Set<Int> = [1, 2, 3, 4, 5]
var setB: Set<Int> = [4, 5, 6, 7, 8]

// Union - O(n+m)
let union = setA.union(setB)
print("Union: \(union)")  // {1,2,3,4,5,6,7,8}

// Intersection - O(min(n,m))
let intersection = setA.intersection(setB)
print("Intersection: \(intersection)")  // {4,5}

// Difference - O(n)
let difference = setA.subtracting(setB)
print("Difference: \(difference)")  // {1,2,3}

// Symmetric Difference - O(n+m)
let symDiff = setA.symmetricDifference(setB)
print("Symmetric Diff: \(symDiff)")  // {1,2,3,6,7,8}

// Subset/Superset
print("Is subset: \(setA.isSubset(of: [1,2,3,4,5,6]))")  // true
print("Is superset: \(setA.isSuperset(of: [1,2]))")        // true
print("Disjoint: \(setA.isDisjoint(with: [6,7,8]))")      // true

// MARK: - Custom Hash for Set

struct Point: Hashable {
    let x: Int
    let y: Int
    
    // Swift synthesizes Hashable for structs with Hashable properties
}

var points: Set<Point> = []
points.insert(Point(x: 1, y: 2))
points.insert(Point(x: 3, y: 4))
points.insert(Point(x: 1, y: 2))  // duplicate
print("Unique points: \(points.count)")  // 2

// Use Case: Find duplicates
func findDuplicates(_ nums: [Int]) -> [Int] {
    var seen: Set<Int> = []
    var duplicates: Set<Int> = []
    for num in nums {
        if seen.contains(num) {
            duplicates.insert(num)
        } else {
            seen.insert(num)
        }
    }
    return Array(duplicates)
}

print(findDuplicates([4, 3, 2, 7, 8, 2, 3, 1]))  // [2, 3]
```

---

## 11. Tries (ทรี)

```swift
// MARK: - Trie Implementation

class TrieNode {
    var children: [Character: TrieNode] = [:]
    var isEndOfWord: Bool = false
    var count: Int = 0  // จำนวน words ที่ผ่าน node นี้
}

class Trie {
    private let root = TrieNode()
    
    // O(m) - m = length of word
    func insert(_ word: String) {
        var current = root
        for char in word {
            if current.children[char] == nil {
                current.children[char] = TrieNode()
            }
            current = current.children[char]!
            current.count += 1
        }
        current.isEndOfWord = true
    }
    
    // O(m)
    func search(_ word: String) -> Bool {
        var current = root
        for char in word {
            guard let node = current.children[char] else { return false }
            current = node
        }
        return current.isEndOfWord
    }
    
    // O(m)
    func startsWith(_ prefix: String) -> Bool {
        var current = root
        for char in prefix {
            guard let node = current.children[char] else { return false }
            current = node
        }
        return true
    }
    
    // O(m) - ลบ word
    func delete(_ word: String) {
        delete(root, word, 0)
    }
    
    @discardableResult
    private func delete(_ node: TrieNode, _ word: String, _ index: Int) -> Bool {
        if index == word.count {
            node.isEndOfWord = false
            return node.children.isEmpty
        }
        let char = word[word.index(word.startIndex, offsetBy: index)]
        guard let child = node.children[char] else { return false }
        child.count -= 1
        if delete(child, word, index + 1) {
            node.children.removeValue(forKey: char)
            return !node.isEndOfWord && node.children.isEmpty
        }
        return false
    }
    
    // หา words ทั้งหมดที่ขึ้นต้นด้วย prefix
    func autocomplete(_ prefix: String) -> [String] {
        var current = root
        for char in prefix {
            guard let node = current.children[char] else { return [] }
            current = node
        }
        var results: [String] = []
        dfs(current, prefix, &results)
        return results
    }
    
    private func dfs(_ node: TrieNode, _ current: String, _ results: inout [String]) {
        if node.isEndOfWord { results.append(current) }
        for (char, child) in node.children {
            dfs(child, current + String(char), &results)
        }
    }
    
    // Word count with prefix
    func countWithPrefix(_ prefix: String) -> Int {
        var current = root
        for char in prefix {
            guard let node = current.children[char] else { return 0 }
            current = node
        }
        return current.count
    }
}

let trie = Trie()
["apple", "app", "application", "apt", "banana"].forEach { trie.insert($0) }
print("Search 'apple': \(trie.search("apple"))")   // true
print("Search 'appl': \(trie.search("appl"))")    // false
print("Starts with 'app': \(trie.startsWith("app"))")  // true
print("Autocomplete 'app': \(trie.autocomplete("app"))")  // [app, apple, application]
```

---

## 12. Thread-Safe Data Structures

```swift
import Foundation

// MARK: - Thread-Safe Stack

class ThreadSafeStack<T> {
    private var storage: [T] = []
    private let lock = NSLock()
    
    var isEmpty: Bool {
        lock.lock()
        defer { lock.unlock() }
        return storage.isEmpty
    }
    
    func push(_ element: T) {
        lock.lock()
        defer { lock.unlock() }
        storage.append(element)
    }
    
    func pop() -> T? {
        lock.lock()
        defer { lock.unlock() }
        return storage.popLast()
    }
}

// MARK: - Thread-Safe Queue ด้วย DispatchQueue

class ConcurrentQueue<T> {
    private var storage: [T] = []
    private let queue = DispatchQueue(label: "com.example.ConcurrentQueue",
                                      attributes: .concurrent)
    
    func enqueue(_ element: T) {
        queue.async(flags: .barrier) { [weak self] in
            self?.storage.append(element)
        }
    }
    
    func dequeue() -> T? {
        var result: T?
        queue.sync(flags: .barrier) { [weak self] in
            result = self?.storage.isEmpty == false ? self?.storage.removeFirst() : nil
        }
        return result
    }
    
    var count: Int {
        queue.sync { storage.count }
    }
}

// MARK: - Actor-based Thread-Safe Data Structure (Swift 5.5+)

actor SafeCache<Key: Hashable, Value> {
    private var storage: [Key: Value] = [:]
    
    func get(_ key: Key) -> Value? {
        return storage[key]
    }
    
    func set(_ key: Key, value: Value) {
        storage[key] = value
    }
    
    func remove(_ key: Key) {
        storage.removeValue(forKey: key)
    }
    
    var count: Int { storage.count }
}

// การใช้งาน Actor
Task {
    let cache = SafeCache<String, Int>()
    await cache.set("a", value: 1)
    await cache.set("b", value: 2)
    let val = await cache.get("a")
    print("Cached value: \(val ?? -1)")
}
```

---

## 13. เมื่อไหร่ควรใช้โครงสร้างข้อมูลแต่ละชนิด

| Structure | ใช้เมื่อ | หลีกเลี่ยงเมื่อ |
|-----------|---------|--------------|
| Array | ต้องการ random access, ข้อมูลมีลำดับ | ต้อง insert/delete ตรงกลางบ่อย |
| Linked List | ต้อง insert/delete ที่หัว/ท้ายบ่อย | ต้องการ random access |
| Stack | LIFO, undo/redo, call stack | ต้องเข้าถึง middle elements |
| Queue | FIFO, BFS, scheduling | ต้องเข้าถึงตรงกลาง |
| Priority Queue | ต้องการ ordered processing | ต้องการ FIFO strict |
| Hash Map | key-value lookup, counting | ต้องการ ordered data |
| BST | ordered operations, range queries | unbalanced data |
| AVL/Red-Black | balanced BST operations | simple read-heavy |
| Heap | k-th largest/smallest, priority | random access |
| Graph | relationships, paths, networks | simple linear data |
| Trie | prefix matching, autocomplete | general key-value |
| Set | uniqueness, membership testing | ordered data needed |

---

## 14. Practical Exercises

### Exercise 1: Implement LRU Cache

```swift
// LRU Cache ด้วย Doubly Linked List + Hash Map
// O(1) สำหรับ get และ put

class LRUCacheFast {
    private class Node {
        let key: Int
        var value: Int
        var prev: Node?
        var next: Node?
        init(_ key: Int, _ value: Int) {
            self.key = key
            self.value = value
        }
    }
    
    private var cache: [Int: Node] = [:]
    private let capacity: Int
    private let head: Node  // dummy head (most recent)
    private let tail: Node  // dummy tail (least recent)
    
    init(_ capacity: Int) {
        self.capacity = capacity
        head = Node(0, 0)
        tail = Node(0, 0)
        head.next = tail
        tail.prev = head
    }
    
    private func addToFront(_ node: Node) {
        node.next = head.next
        node.prev = head
        head.next?.prev = node
        head.next = node
    }
    
    private func remove(_ node: Node) {
        node.prev?.next = node.next
        node.next?.prev = node.prev
    }
    
    func get(_ key: Int) -> Int {
        guard let node = cache[key] else { return -1 }
        remove(node)
        addToFront(node)
        return node.value
    }
    
    func put(_ key: Int, _ value: Int) {
        if let node = cache[key] {
            node.value = value
            remove(node)
            addToFront(node)
        } else {
            if cache.count >= capacity {
                let lru = tail.prev!
                remove(lru)
                cache.removeValue(forKey: lru.key)
            }
            let node = Node(key, value)
            cache[key] = node
            addToFront(node)
        }
    }
}

// Test
let lruFast = LRUCacheFast(2)
lruFast.put(1, 1)
lruFast.put(2, 2)
print(lruFast.get(1))    // 1
lruFast.put(3, 3)        // evicts key 2
print(lruFast.get(2))    // -1
```

### Exercise 2: Flatten Binary Tree to Linked List

```swift
func flatten(_ root: TreeNode<Int>?) {
    var current = root
    while let node = current {
        if let leftChild = node.left {
            // หา rightmost ของ left subtree
            var rightmost = leftChild
            while let right = rightmost.right { rightmost = right }
            // ต่อ right subtree เดิมที่ปลาย left subtree
            rightmost.right = node.right
            // ย้าย left subtree ไปด้านขวา
            node.right = leftChild
            node.left = nil
        }
        current = node.right
    }
}
```

### Exercise 3: Word Ladder

```swift
func ladderLength(_ beginWord: String, _ endWord: String, _ wordList: [String]) -> Int {
    var wordSet = Set(wordList)
    guard wordSet.contains(endWord) else { return 0 }
    
    var queue: [(word: String, steps: Int)] = [(beginWord, 1)]
    var visited: Set<String> = [beginWord]
    
    while !queue.isEmpty {
        let (word, steps) = queue.removeFirst()
        
        for i in word.indices {
            var chars = Array(word)
            for c in "abcdefghijklmnopqrstuvwxyz" {
                chars[i] = c
                let newWord = String(chars)
                if newWord == endWord { return steps + 1 }
                if wordSet.contains(newWord) && !visited.contains(newWord) {
                    visited.insert(newWord)
                    queue.append((newWord, steps + 1))
                }
            }
            chars[i] = Array(word)[i]  // restore
        }
    }
    return 0
}
```

---

## 15. สรุป

### Data Structure Cheat Sheet

```
Arrays:
  - Access: O(1), Search: O(n), Insert/Delete end: O(1)amortized, Insert/Delete middle: O(n)

Linked Lists:
  - Access: O(n), Search: O(n), Insert/Delete head: O(1), Insert/Delete tail: O(1)*

Hash Map:
  - Average: O(1) for get/set/delete
  - Worst: O(n) with many collisions

BST (balanced):
  - Search/Insert/Delete: O(log n)

Heap:
  - Insert: O(log n), Remove max/min: O(log n), Peek: O(1)

Graph (V=vertices, E=edges):
  - BFS/DFS: O(V+E), Dijkstra: O((V+E) log V)

Trie:
  - Insert/Search/Delete: O(m) where m = key length
```

โครงสร้างข้อมูลที่ดีคือรากฐานของซอฟต์แวร์ที่มีประสิทธิภาพ การเลือกโครงสร้างข้อมูลที่เหมาะสมจะส่งผลโดยตรงต่อความเร็วและการใช้หน่วยความจำของโปรแกรม ควรฝึกฝนและทำความเข้าใจ trade-offs ของแต่ละโครงสร้างเพื่อนำไปประยุกต์ใช้ได้อย่างถูกต้อง

---

*จบ Part 65: Data Structures ใน Swift*
