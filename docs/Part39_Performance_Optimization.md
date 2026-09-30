# ตอนที่ 39: Performance Optimization ใน Swift

## บทนำ

การเพิ่มประสิทธิภาพ (Performance Optimization) เป็นหนึ่งในทักษะสำคัญที่นักพัฒนา iOS จำเป็นต้องมี แอปพลิเคชันที่ทำงานได้อย่างราบรื่นไม่เพียงแต่ให้ประสบการณ์ที่ดีแก่ผู้ใช้ แต่ยังช่วยประหยัดแบตเตอรี่และทรัพยากรของอุปกรณ์อีกด้วย

ในบทเรียนนี้เราจะเรียนรู้:
- หลักการพื้นฐานของ Performance
- เครื่องมือสำหรับวัดและวิเคราะห์ประสิทธิภาพ
- เทคนิคการเพิ่มประสิทธิภาพใน Swift
- การจัดการ Memory อย่างมีประสิทธิภาพ
- การใช้ Concurrency อย่างถูกต้อง

---

## 39.1 หลักการพื้นฐานของ Performance

### ความหมายของ Performance

Performance หมายถึงความสามารถของแอปพลิเคชันในการตอบสนองต่อผู้ใช้ได้อย่างรวดเร็วและใช้ทรัพยากรอย่างมีประสิทธิภาพ โดยมีมิติหลักดังนี้:

1. **Responsiveness** - ความเร็วในการตอบสนองต่อ User Interaction
2. **Throughput** - ปริมาณงานที่ทำได้ในช่วงเวลาหนึ่ง
3. **Memory Usage** - การใช้หน่วยความจำ
4. **Battery Life** - ผลกระทบต่อแบตเตอรี่
5. **App Size** - ขนาดของแอปพลิเคชัน

### กฎทองของการ Optimize

```swift
// กฎสำคัญ: อย่า Optimize ก่อนวัดผล
// "Premature optimization is the root of all evil" - Donald Knuth

// ขั้นตอนที่ถูกต้อง:
// 1. ทำให้มันทำงานได้ถูกต้องก่อน (Make it work)
// 2. ทำให้มันทำงานได้สวยงาม (Make it right)  
// 3. วัดผลเพื่อหา Bottleneck (Measure)
// 4. เพิ่มประสิทธิภาพเฉพาะส่วนที่จำเป็น (Optimize)

class PerformancePhilosophy {
    // ตัวอย่าง: โค้ดที่ถูกต้องก่อน แล้วค่อย optimize
    
    // Version 1: ทำให้ถูกต้องก่อน (Correct but may be slow)
    func findDuplicates_v1(_ array: [Int]) -> [Int] {
        var seen = [Int]()
        var duplicates = [Int]()
        
        for item in array {
            if seen.contains(item) {
                if !duplicates.contains(item) {
                    duplicates.append(item)
                }
            } else {
                seen.append(item)
            }
        }
        return duplicates
    }
    
    // Version 2: หลังจากวัดพบว่าช้า จึง optimize
    func findDuplicates_v2(_ array: [Int]) -> [Int] {
        var seen = Set<Int>()
        var duplicates = Set<Int>()
        
        for item in array {
            if seen.contains(item) {
                duplicates.insert(item)
            } else {
                seen.insert(item)
            }
        }
        return Array(duplicates)
    }
}
```

---

## 39.2 Big O Notation

### ทบทวน Big O Notation

Big O Notation เป็นวิธีการอธิบายความซับซ้อนของ Algorithm ในแง่ของเวลาและ Memory

```swift
// O(1) - Constant Time: ใช้เวลาคงที่ไม่ว่า Input จะใหญ่แค่ไหน
func getFirstElement(_ array: [Int]) -> Int? {
    return array.first  // O(1)
}

// O(log n) - Logarithmic Time: Binary Search
func binarySearch(_ sortedArray: [Int], target: Int) -> Int? {
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
    return nil
}  // O(log n)

// O(n) - Linear Time: ค้นหาแบบ Linear
func linearSearch(_ array: [Int], target: Int) -> Int? {
    for (index, item) in array.enumerated() {
        if item == target {
            return index
        }
    }
    return nil
}  // O(n)

// O(n log n) - ส่วนใหญ่เป็น Sorting Algorithms ที่ดี
func mergeSort(_ array: [Int]) -> [Int] {
    guard array.count > 1 else { return array }
    
    let mid = array.count / 2
    let left = mergeSort(Array(array[..<mid]))
    let right = mergeSort(Array(array[mid...]))
    
    return merge(left, right)
}

func merge(_ left: [Int], _ right: [Int]) -> [Int] {
    var result = [Int]()
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
}  // O(n log n)

// O(n²) - Quadratic Time: Nested Loops
func bubbleSort(_ array: inout [Int]) {
    let n = array.count
    for i in 0..<n {
        for j in 0..<(n - i - 1) {
            if array[j] > array[j + 1] {
                array.swapAt(j, j + 1)
            }
        }
    }
}  // O(n²)

// O(2^n) - Exponential Time: หลีกเลี่ยงถ้าทำได้
func fibonacci_naive(_ n: Int) -> Int {
    if n <= 1 { return n }
    return fibonacci_naive(n - 1) + fibonacci_naive(n - 2)
}  // O(2^n) - ช้ามาก!

// ปรับปรุงด้วย Dynamic Programming
func fibonacci_optimized(_ n: Int) -> Int {
    if n <= 1 { return n }
    var memo = [Int: Int]()
    
    func fib(_ n: Int) -> Int {
        if let cached = memo[n] { return cached }
        if n <= 1 { return n }
        let result = fib(n - 1) + fib(n - 2)
        memo[n] = result
        return result
    }
    
    return fib(n)
}  // O(n) - เร็วกว่ามาก
```

### Space Complexity

```swift
// Space O(1): ไม่ใช้ Memory เพิ่มขึ้นตาม Input
func sumArray(_ array: [Int]) -> Int {
    var sum = 0
    for item in array {
        sum += item
    }
    return sum
}

// Space O(n): ใช้ Memory เพิ่มขึ้นตาม Input
func reverseArray(_ array: [Int]) -> [Int] {
    return array.reversed()  // สร้าง Array ใหม่ขนาด n
}

// Space O(1) alternative: ใช้ In-place
func reverseArrayInPlace(_ array: inout [Int]) {
    var left = 0
    var right = array.count - 1
    
    while left < right {
        array.swapAt(left, right)
        left += 1
        right -= 1
    }
}
```

---

## 39.3 Value Types vs Reference Types Performance

### ความแตกต่างด้าน Performance

```swift
import Foundation

// Value Types (Struct): Copy on assignment
struct PointValue {
    var x: Double
    var y: Double
    
    func distance(to other: PointValue) -> Double {
        let dx = x - other.x
        let dy = y - other.y
        return sqrt(dx * dx + dy * dy)
    }
}

// Reference Types (Class): Share reference
class PointReference {
    var x: Double
    var y: Double
    
    init(x: Double, y: Double) {
        self.x = x
        self.y = y
    }
    
    func distance(to other: PointReference) -> Double {
        let dx = x - other.x
        let dy = y - other.y
        return sqrt(dx * dx + dy * dy)
    }
}

// การทดสอบประสิทธิภาพ
func testValueTypePerformance() {
    var points = [PointValue]()
    
    // Value types ถูก store บน Stack (เร็วกว่า)
    for i in 0..<1000 {
        let point = PointValue(x: Double(i), y: Double(i))
        points.append(point)
    }
    
    // การ modify ไม่มี reference counting overhead
    for i in 0..<points.count {
        points[i].x += 1.0
    }
}

func testReferenceTypePerformance() {
    var points = [PointReference]()
    
    // Reference types ถูก allocate บน Heap (ช้ากว่า)
    for i in 0..<1000 {
        let point = PointReference(x: Double(i), y: Double(i))
        points.append(point)
    }
    
    // การ modify มี reference counting overhead
    for point in points {
        point.x += 1.0
    }
}

// เมื่อไหร่ควรใช้ Class แทน Struct
class DataManager {
    // ใช้ Class เมื่อ:
    // 1. ต้องการ Shared State
    // 2. ต้องการ Identity
    // 3. ต้องการ Inheritance
    // 4. ต้อง interface กับ Objective-C
    
    var sharedData: [String: Any] = [:]
    
    func updateData(key: String, value: Any) {
        sharedData[key] = value
        // การเปลี่ยนแปลงนี้จะเห็นได้จากทุกที่ที่ reference object นี้
    }
}
```

### Protocol-Oriented Performance

```swift
// Protocol ช่วยให้ใช้ Value Types ได้อย่างยืดหยุ่น
protocol Shape {
    var area: Double { get }
    var perimeter: Double { get }
}

struct Circle: Shape {
    let radius: Double
    
    var area: Double {
        return Double.pi * radius * radius
    }
    
    var perimeter: Double {
        return 2 * Double.pi * radius
    }
}

struct Rectangle: Shape {
    let width: Double
    let height: Double
    
    var area: Double {
        return width * height
    }
    
    var perimeter: Double {
        return 2 * (width + height)
    }
}

// Protocol Composition สำหรับ Performance-critical Code
protocol Drawable: Shape {
    func draw()
}

struct OptimizedCircle: Drawable {
    let radius: Double
    
    var area: Double { Double.pi * radius * radius }
    var perimeter: Double { 2 * Double.pi * radius }
    
    func draw() {
        // Drawing implementation
        print("Drawing circle with radius \(radius)")
    }
}

// การใช้ some keyword สำหรับ Opaque Types (Swift 5.1+)
func createBestShape(isCircle: Bool) -> some Shape {
    if isCircle {
        return Circle(radius: 5.0)
    } else {
        return Circle(radius: 3.0)  // ต้อง return ประเภทเดียวกัน
    }
}
```

---

## 39.4 Copy-on-Write Optimization

### หลักการ Copy-on-Write (CoW)

```swift
// Swift Collections ใช้ Copy-on-Write อยู่แล้ว
var array1 = [1, 2, 3, 4, 5]
var array2 = array1  // ยังไม่มีการ copy จริง - share memory

array2.append(6)  // ตอนนี้ต่างหากจึง copy - "copy on write"

print(array1)  // [1, 2, 3, 4, 5]
print(array2)  // [1, 2, 3, 4, 5, 6]

// การ implement CoW เอง
final class StorageBuffer<T> {
    var elements: [T]
    
    init(_ elements: [T] = []) {
        self.elements = elements
    }
    
    init(copying other: StorageBuffer<T>) {
        self.elements = other.elements
    }
}

struct CowCollection<T> {
    private var buffer: StorageBuffer<T>
    
    init(_ elements: [T] = []) {
        buffer = StorageBuffer(elements)
    }
    
    // ตรวจสอบว่ามีการ share reference หรือไม่
    private mutating func ensureUniquelyReferenced() {
        if !isKnownUniquelyReferenced(&buffer) {
            buffer = StorageBuffer(copying: buffer)
        }
    }
    
    mutating func append(_ element: T) {
        ensureUniquelyReferenced()  // copy เฉพาะเมื่อจำเป็น
        buffer.elements.append(element)
    }
    
    var count: Int { buffer.elements.count }
    
    subscript(index: Int) -> T {
        get { buffer.elements[index] }
        set {
            ensureUniquelyReferenced()
            buffer.elements[index] = newValue
        }
    }
}

// การทดสอบ CoW
func demonstrateCow() {
    var col1 = CowCollection<Int>([1, 2, 3])
    var col2 = col1  // ยังไม่มี copy จริง
    
    col2.append(4)  // copy เกิดขึ้นตอนนี้เท่านั้น
    
    print("col1 count:", col1.count)  // 3
    print("col2 count:", col2.count)  // 4
}
```

### Optimizing with isKnownUniquelyReferenced

```swift
// ตัวอย่าง Real-world CoW Implementation
final class ImageData {
    var pixels: [UInt8]
    var width: Int
    var height: Int
    
    init(width: Int, height: Int) {
        self.width = width
        self.height = height
        self.pixels = Array(repeating: 0, count: width * height * 4)
    }
    
    init(copying other: ImageData) {
        self.width = other.width
        self.height = other.height
        self.pixels = other.pixels
    }
}

struct Image {
    private var data: ImageData
    
    init(width: Int, height: Int) {
        data = ImageData(width: width, height: height)
    }
    
    private mutating func ensureUniqueStorage() {
        if !isKnownUniquelyReferenced(&data) {
            print("Copying image data...")  // จะเห็นว่า copy เฉพาะเมื่อจำเป็น
            data = ImageData(copying: data)
        }
    }
    
    mutating func setPixel(x: Int, y: Int, r: UInt8, g: UInt8, b: UInt8, a: UInt8) {
        ensureUniqueStorage()
        let index = (y * data.width + x) * 4
        data.pixels[index] = r
        data.pixels[index + 1] = g
        data.pixels[index + 2] = b
        data.pixels[index + 3] = a
    }
    
    var width: Int { data.width }
    var height: Int { data.height }
}
```

---

## 39.5 Lazy Evaluation

### Lazy Properties

```swift
class ExpensiveComputation {
    // Computed property: คำนวณทุกครั้งที่เข้าถึง
    var expensiveComputed: [Int] {
        print("Computing...")
        return (0..<1000).map { $0 * $0 }  // O(n) ทุกครั้ง
    }
    
    // Lazy property: คำนวณครั้งเดียว ครั้งแรกที่เข้าถึง
    lazy var expensiveLazy: [Int] = {
        print("Computing (lazy)...")
        return (0..<1000).map { $0 * $0 }  // คำนวณครั้งเดียว
    }()
    
    var name: String
    
    init(name: String) {
        self.name = name
        // expensiveLazy ยังไม่ถูกคำนวณ ณ จุดนี้
    }
}

// การใช้งาน
let obj = ExpensiveComputation(name: "Test")
// ยังไม่มีการคำนวณ

let result1 = obj.expensiveLazy  // คำนวณตอนนี้
let result2 = obj.expensiveLazy  // ใช้ค่าที่ cache ไว้แล้ว
```

### Lazy Sequences

```swift
// Lazy Collections ใน Swift
let numbers = Array(1...1_000_000)

// ไม่ใช้ Lazy: สร้าง intermediate arrays ทุกขั้น (ช้าและใช้ Memory มาก)
let result_eager = numbers
    .filter { $0 % 2 == 0 }  // สร้าง Array ใหม่ 500,000 elements
    .map { $0 * $0 }          // สร้าง Array ใหม่ 500,000 elements
    .prefix(10)               // เลือก 10 elements

// ใช้ Lazy: ประมวลผลเฉพาะที่จำเป็น (เร็วกว่ามากเมื่อต้องการแค่ส่วนหนึ่ง)
let result_lazy = numbers.lazy
    .filter { $0 % 2 == 0 }  // ยังไม่ประมวลผล
    .map { $0 * $0 }          // ยังไม่ประมวลผล
    .prefix(10)               // ประมวลผลแค่ที่จำเป็นเพื่อได้ 10 items
    
print(Array(result_lazy))  // [4, 16, 36, 64, 100, 144, 196, 256, 324, 400]

// Lazy Sequence สำหรับ Infinite Sequences
struct InfiniteSequence: Sequence, IteratorProtocol {
    var current = 0
    
    mutating func next() -> Int? {
        defer { current += 1 }
        return current
    }
}

let infinite = InfiniteSequence()
let first10Even = infinite.lazy
    .filter { $0 % 2 == 0 }
    .prefix(10)

print(Array(first10Even))  // [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
```

---

## 39.6 Avoiding Premature Optimization

### หลักการสำคัญ

```swift
// ตัวอย่าง: Premature Optimization ที่ทำให้โค้ดอ่านยาก
// ก่อน Optimize (อ่านง่ายกว่า):
func calculateAverage(_ numbers: [Double]) -> Double {
    guard !numbers.isEmpty else { return 0 }
    return numbers.reduce(0, +) / Double(numbers.count)
}

// หลัง Premature Optimize (อ่านยากขึ้น แต่เร็วกว่าแค่นิดหน่อย):
func calculateAverage_premature(_ numbers: [Double]) -> Double {
    let count = numbers.count
    guard count > 0 else { return 0 }
    
    var sum: Double = 0
    var i = 0
    while i < count {  // ใช้ while แทน for...in เพราะคิดว่าเร็วกว่า
        sum += numbers[i]
        i += 1
    }
    return sum / Double(count)
}

// ความจริง: Swift Compiler จะ optimize ให้อยู่แล้ว
// ควรใช้ version แรกที่อ่านง่ายกว่า

// เมื่อไหร่ควร Optimize จริงๆ:
// 1. วัดแล้วพบว่าเป็น Bottleneck จริง
// 2. Optimization ไม่ทำให้โค้ดอ่านยากเกินไป
// 3. มี Test ที่ครอบคลุมก่อน Optimize

class PerformanceProfiler {
    static func measure(label: String, block: () -> Void) {
        let start = CFAbsoluteTimeGetCurrent()
        block()
        let elapsed = CFAbsoluteTimeGetCurrent() - start
        print("\(label): \(elapsed * 1000)ms")
    }
}

// การใช้งาน
PerformanceProfiler.measure(label: "Average calculation") {
    let numbers = (0..<10000).map { Double($0) }
    let _ = calculateAverage(numbers)
}
```

---

## 39.7 Instruments Usage

### การใช้ Instruments เพื่อ Profile

```swift
// Instruments เป็น Profiling Tool ที่มาพร้อม Xcode
// วิธีเปิด: Xcode > Product > Profile (⌘+I)

// Main Instruments ที่ควรรู้จัก:

// 1. Time Profiler - วัดเวลาที่ใช้ใน CPU
// 2. Allocations - วัดการ Allocate Memory
// 3. Leaks - หา Memory Leaks
// 4. Core Data - วิเคราะห์ Database Operations
// 5. Network - วิเคราะห์ Network Calls
// 6. Energy Log - วิเคราะห์การใช้พลังงาน

// การเพิ่ม Signpost เพื่อ Mark ใน Instruments
import os.signpost

class AppPerformanceMarker {
    static let log = OSLog(subsystem: "com.example.app", category: "performance")
    
    static func markStart(name: StaticString) {
        os_signpost(.begin, log: log, name: name)
    }
    
    static func markEnd(name: StaticString) {
        os_signpost(.end, log: log, name: name)
    }
    
    static func measure<T>(name: StaticString, block: () throws -> T) rethrows -> T {
        os_signpost(.begin, log: log, name: name)
        defer { os_signpost(.end, log: log, name: name) }
        return try block()
    }
}

// การใช้งาน
func loadData() {
    AppPerformanceMarker.measure(name: "Data Loading") {
        // โค้ดที่ต้องการวัดเวลา
        Thread.sleep(forTimeInterval: 0.1)  // จำลอง I/O operation
    }
}

// Custom Instruments Template
// สร้างใน .instrpkg file สำหรับ Custom Profiling Tool
```

---

## 39.8 Time Profiler Analysis

### การอ่านผล Time Profiler

```swift
// สิ่งที่ต้องดูใน Time Profiler:
// 1. Heavy Thread - Thread ที่ใช้ CPU มากสุด
// 2. Call Tree - ลำดับการเรียก Functions
// 3. Self vs Total Time
//    - Self: เวลาที่ function นี้ใช้โดยตรง
//    - Total: รวมเวลาของ functions ที่ถูกเรียกด้วย

// ตัวอย่าง โค้ดที่ทำให้ Time Profiler แสดงผลที่น่าสนใจ
class HeavyWorkExample {
    
    // Function ที่ใช้เวลาใน Main Thread มาก (ควรย้ายไป Background)
    func processImages(_ images: [Data]) -> [Data] {
        return images.map { imageData in
            // การประมวลผล Image ที่ใช้เวลานาน
            return processImage(imageData)
        }
    }
    
    private func processImage(_ data: Data) -> Data {
        // จำลอง Heavy Processing
        var result = data
        for _ in 0..<1000 {
            result = result  // Placeholder
        }
        return result
    }
    
    // หลัง Profile และพบ Bottleneck: ย้ายไป Background Thread
    func processImagesAsync(_ images: [Data], completion: @escaping ([Data]) -> Void) {
        DispatchQueue.global(qos: .userInitiated).async {
            let processed = images.map { self.processImage($0) }
            DispatchQueue.main.async {
                completion(processed)
            }
        }
    }
}

// การ Measure Performance ด้วย XCTest
import XCTest

class PerformanceTests: XCTestCase {
    
    func testSortingPerformance() {
        let numbers = (0..<10000).shuffled()
        
        measure {
            let _ = numbers.sorted()
        }
    }
    
    func testFilteringPerformance() {
        let numbers = Array(0..<100000)
        
        measure {
            let _ = numbers.filter { $0 % 2 == 0 }
        }
    }
    
    // การตั้ง Performance Baseline
    func testBaselinePerformance() {
        measure(metrics: [XCTCPUMetric(), XCTMemoryMetric()]) {
            // โค้ดที่ต้องการ Measure
        }
    }
}
```

---

## 39.9 Memory Optimization

### หลักการ Memory Management

```swift
// ARC (Automatic Reference Counting) จัดการ Memory อัตโนมัติ
// แต่เราต้องระวัง Memory Leaks และ Retain Cycles

// Retain Cycle ที่พบบ่อย:
class Parent {
    var child: Child?
    var name: String
    
    init(name: String) {
        self.name = name
        print("Parent \(name) created")
    }
    
    deinit {
        print("Parent \(name) deallocated")
    }
}

class Child {
    weak var parent: Parent?  // ใช้ weak เพื่อหลีกเลี่ยง Retain Cycle
    var name: String
    
    init(name: String) {
        self.name = name
        print("Child \(name) created")
    }
    
    deinit {
        print("Child \(name) deallocated")
    }
}

func demonstrateMemoryManagement() {
    var parent: Parent? = Parent(name: "Dad")
    var child: Child? = Child(name: "Son")
    
    parent?.child = child
    child?.parent = parent  // weak reference - ไม่เพิ่ม retain count
    
    parent = nil  // Parent deallocated
    child = nil   // Child deallocated
    // ไม่มี Memory Leak เพราะใช้ weak
}

// Closure Retain Cycle
class ViewController_Memory {
    var name = "MyViewController"
    var completion: (() -> Void)?
    
    func setupBadClosure() {
        // Bad: Retain Cycle
        completion = {
            print(self.name)  // self ถูก capture strongly
        }
    }
    
    func setupGoodClosure() {
        // Good: ใช้ [weak self]
        completion = { [weak self] in
            guard let self = self else { return }
            print(self.name)
        }
    }
    
    func setupUnownedClosure() {
        // Good ถ้า self จะยังมีอยู่เสมอเมื่อ closure ถูกเรียก
        completion = { [unowned self] in
            print(self.name)
        }
    }
    
    deinit {
        print("\(name) deallocated")
    }
}
```

### Memory Optimization Techniques

```swift
// 1. ใช้ struct แทน class เมื่อทำได้
struct LightweightData {
    let id: Int
    let value: String
    // อยู่บน Stack - ไม่มี Heap Allocation overhead
}

// 2. Object Pooling สำหรับ Objects ที่สร้างบ่อย
class ObjectPool<T: AnyObject> {
    private var pool: [T] = []
    private let factory: () -> T
    private let reset: (T) -> Void
    private let maxSize: Int
    
    init(maxSize: Int = 10, factory: @escaping () -> T, reset: @escaping (T) -> Void) {
        self.maxSize = maxSize
        self.factory = factory
        self.reset = reset
    }
    
    func acquire() -> T {
        if let object = pool.popLast() {
            return object
        }
        return factory()
    }
    
    func release(_ object: T) {
        guard pool.count < maxSize else { return }
        reset(object)
        pool.append(object)
    }
}

// 3. Autoreleasepool สำหรับ Loops ที่สร้าง Objects จำนวนมาก
func processLargeDataset(_ items: [String]) {
    for batch in items.chunked(into: 100) {
        autoreleasepool {
            // Objects ที่สร้างในนี้จะถูก release หลังจาก block นี้จบ
            for item in batch {
                let processed = item.uppercased()
                // ... processing
                _ = processed
            }
        }
    }
}

extension Array {
    func chunked(into size: Int) -> [[Element]] {
        return stride(from: 0, to: count, by: size).map {
            Array(self[$0 ..< Swift.min($0 + size, count)])
        }
    }
}
```

---

## 39.10 Reducing Allocations

### การลด Memory Allocations

```swift
// Stack vs Heap Allocation
// Stack: เร็วกว่า, ขนาดจำกัด, เหมาะสำหรับ Value Types
// Heap: ช้ากว่า, ขนาดใหญ่กว่า, ใช้สำหรับ Reference Types

// เทคนิค 1: Pre-allocate Arrays
func buildArray_bad() -> [Int] {
    var result = [Int]()  // เริ่มต้น empty, จะ resize หลายครั้ง
    for i in 0..<1000 {
        result.append(i)
    }
    return result
}

func buildArray_good() -> [Int] {
    var result = [Int]()
    result.reserveCapacity(1000)  // Pre-allocate ป้องกันการ resize
    for i in 0..<1000 {
        result.append(i)
    }
    return result
}

// เทคนิค 2: Reuse Buffers
class DataProcessor {
    private var buffer = [UInt8]()  // Reuse buffer แทนการสร้างใหม่ทุกครั้ง
    
    func process(_ data: Data) -> Data {
        buffer.removeAll(keepingCapacity: true)  // Clear แต่ยัง keep memory
        
        for byte in data {
            buffer.append(byte ^ 0xFF)  // XOR operation
        }
        
        return Data(buffer)
    }
}

// เทคนิค 3: Avoid Boxing Value Types
// Boxing คือการ wrap value type ใน reference type container

// Bad: Boxing เกิดขึ้นเมื่อใส่ Int ใน Array<Any>
var anyArray: [Any] = []
for i in 0..<1000 {
    anyArray.append(i)  // Each Int gets boxed
}

// Good: ใช้ Typed Array
var intArray: [Int] = []
for i in 0..<1000 {
    intArray.append(i)  // ไม่มี Boxing
}

// เทคนิค 4: ContiguousArray สำหรับ Performance-critical Code
var contiguous = ContiguousArray<Int>()  // Guaranteed contiguous memory
contiguous.reserveCapacity(1000)
for i in 0..<1000 {
    contiguous.append(i)
}
// เร็วกว่า [Int] เล็กน้อย โดยเฉพาะสำหรับ loops
```

---

## 39.11 String Performance

### การ Optimize String Operations

```swift
// Strings ใน Swift เป็น Unicode-aware ซึ่งอาจช้ากว่า
// เนื่องจาก Character อาจมีหลาย Code Points

// เทคนิค 1: ใช้ hasPrefix/hasSuffix แทนการ substring
let url = "https://api.example.com/users/123"

// Bad: สร้าง substring
let prefix = String(url.prefix(5))
if prefix == "https" {
    print("Secure")
}

// Good: ใช้ hasPrefix (ไม่สร้าง substring)
if url.hasPrefix("https") {
    print("Secure")
}

// เทคนิค 2: StringBuilder Pattern
func buildLargeString_bad(_ items: [String]) -> String {
    var result = ""
    for item in items {
        result += item + ", "  // String concatenation สร้าง String ใหม่ทุกครั้ง
    }
    return result
}

func buildLargeString_good(_ items: [String]) -> String {
    return items.joined(separator: ", ")  // ดีกว่า
}

// หรือใช้ InterpolationBuilder
func buildLargeString_better(_ items: [String]) -> String {
    var result = ""
    result.reserveCapacity(items.reduce(0) { $0 + $1.count + 2 })
    for item in items {
        result += item
        result += ", "
    }
    return result
}

// เทคนิค 3: ใช้ String.Index แทน Int Index
let text = "Hello, World!"
let startIndex = text.startIndex
let endIndex = text.index(startIndex, offsetBy: 5)
let substring = text[startIndex..<endIndex]  // "Hello"

// เทคนิค 4: ใช้ utf8 หรือ utf16 เมื่อเหมาะสม
let data = "Hello".utf8
// เร็วกว่าการ iterate ผ่าน Character

// เทคนิค 5: String Interning สำหรับ Repeated Strings
struct InternedString {
    private static var pool: [String: String] = [:]
    
    static func intern(_ string: String) -> String {
        if let existing = pool[string] {
            return existing
        }
        pool[string] = string
        return string
    }
}
```

---

## 39.12 Collection Performance

### การเลือก Collection ที่เหมาะสม

```swift
// Array: เหมาะสำหรับ Sequential Access และ Random Access by Index
// - O(1) random access
// - O(n) search (unsorted)
// - O(1) amortized append
// - O(n) insert/remove ตรงกลาง

// Dictionary: เหมาะสำหรับ Key-Value Lookup
// - O(1) average lookup, insert, delete

// Set: เหมาะสำหรับ Unique Elements และ Membership Testing
// - O(1) average lookup, insert, delete
// - ไม่มี order

// การเปรียบเทียบ
let largeArray = Array(0..<100000)
let largeSet = Set(0..<100000)
let largeDictionary = Dictionary(uniqueKeysWithValues: (0..<100000).map { ($0, $0 * 2) })

// ค้นหาใน Array: O(n)
func searchArray() {
    let _ = largeArray.contains(99999)  // ต้อง scan ทั้ง array ในกรณีเลวร้ายสุด
}

// ค้นหาใน Set: O(1)
func searchSet() {
    let _ = largeSet.contains(99999)  // Hash lookup - เร็วกว่ามาก
}

// Array Operations ที่มีประสิทธิภาพ
var numbers = [3, 1, 4, 1, 5, 9, 2, 6]

// ดี: sort() สำหรับ in-place
numbers.sort()  // O(n log n)

// ดี: sorted() สำหรับ new array
let sorted = numbers.sorted()  // O(n log n) แต่ allocate memory ใหม่

// หลีกเลี่ยง: การ insert ตรงกลาง Array ใหญ่ๆ
numbers.insert(0, at: numbers.count / 2)  // O(n) ต้อง shift elements

// การ Filter + Map ด้วย compactMap
let strings = ["1", "2", "abc", "4", "5"]
let ints = strings.compactMap { Int($0) }  // Filter + Map ในขั้นเดียว

// Reduce สำหรับ Aggregation
let sum = numbers.reduce(0, +)
let product = numbers.reduce(1, *)
```

---

## 39.13 Algorithm Choice

### การเลือก Algorithm ที่เหมาะสม

```swift
// ตัวอย่าง: การหา Element ที่ซ้ำกัน

// Algorithm 1: Brute Force O(n²)
func findDuplicatesBrute(_ arr: [Int]) -> [Int] {
    var duplicates = [Int]()
    for i in 0..<arr.count {
        for j in (i+1)..<arr.count {
            if arr[i] == arr[j] && !duplicates.contains(arr[i]) {
                duplicates.append(arr[i])
            }
        }
    }
    return duplicates
}

// Algorithm 2: HashSet O(n)
func findDuplicatesHash(_ arr: [Int]) -> [Int] {
    var seen = Set<Int>()
    var duplicates = Set<Int>()
    
    for num in arr {
        if seen.contains(num) {
            duplicates.insert(num)
        } else {
            seen.insert(num)
        }
    }
    return Array(duplicates)
}

// Algorithm 3: Sort first O(n log n)
func findDuplicatesSort(_ arr: [Int]) -> [Int] {
    let sorted = arr.sorted()
    var duplicates = [Int]()
    
    for i in 1..<sorted.count {
        if sorted[i] == sorted[i-1] && (duplicates.isEmpty || duplicates.last != sorted[i]) {
            duplicates.append(sorted[i])
        }
    }
    return duplicates
}

// สำหรับ Array ขนาดเล็ก Algorithm 3 อาจเร็วกว่าเพราะ Memory Locality ดีกว่า
// สำหรับ Array ขนาดใหญ่ Algorithm 2 (O(n)) จะเร็วกว่าอย่างชัดเจน

// Sorting: เลือก Algorithm ตาม Use Case
// - sorted(): General purpose, O(n log n)
// - insertionSort: ดีสำหรับ nearly sorted arrays
// - radixSort: ดีสำหรับ integers ใน range ที่ทราบ

extension Array where Element == Int {
    // Radix Sort O(d * n) เมื่อ d คือจำนวน digits
    func radixSort() -> [Int] {
        guard !isEmpty else { return self }
        
        var output = self
        let maxVal = output.max()!
        var exp = 1
        
        while maxVal / exp > 0 {
            var count = Array(repeating: 0, count: 10)
            
            for num in output {
                count[(num / exp) % 10] += 1
            }
            
            for i in 1..<10 {
                count[i] += count[i-1]
            }
            
            var temp = Array(repeating: 0, count: output.count)
            for i in stride(from: output.count - 1, through: 0, by: -1) {
                let digit = (output[i] / exp) % 10
                temp[count[digit] - 1] = output[i]
                count[digit] -= 1
            }
            
            output = temp
            exp *= 10
        }
        
        return output
    }
}
```

---

## 39.14 Caching Strategies

### การ Implement Cache

```swift
import Foundation

// Simple Cache ด้วย Dictionary
class SimpleCache<Key: Hashable, Value> {
    private var cache: [Key: Value] = [:]
    private let maxSize: Int
    private var accessOrder: [Key] = []  // สำหรับ LRU
    
    init(maxSize: Int = 100) {
        self.maxSize = maxSize
    }
    
    func get(_ key: Key) -> Value? {
        if let value = cache[key] {
            // Update access order (LRU)
            if let index = accessOrder.firstIndex(of: key) {
                accessOrder.remove(at: index)
            }
            accessOrder.append(key)
            return value
        }
        return nil
    }
    
    func set(_ key: Key, value: Value) {
        if cache[key] == nil {
            if cache.count >= maxSize {
                // Evict least recently used
                if let lru = accessOrder.first {
                    cache.removeValue(forKey: lru)
                    accessOrder.removeFirst()
                }
            }
        }
        cache[key] = value
        
        if let index = accessOrder.firstIndex(of: key) {
            accessOrder.remove(at: index)
        }
        accessOrder.append(key)
    }
    
    func invalidate(_ key: Key) {
        cache.removeValue(forKey: key)
        if let index = accessOrder.firstIndex(of: key) {
            accessOrder.remove(at: index)
        }
    }
    
    func clear() {
        cache.removeAll()
        accessOrder.removeAll()
    }
}

// NSCache - Thread-safe cache ที่มาพร้อม iOS
class ImageCache {
    static let shared = ImageCache()
    private let cache = NSCache<NSString, UIImage>()
    
    private init() {
        cache.countLimit = 100
        cache.totalCostLimit = 50 * 1024 * 1024  // 50 MB
    }
    
    func image(for key: String) -> UIImage? {
        return cache.object(forKey: key as NSString)
    }
    
    func store(_ image: UIImage, for key: String) {
        let cost = image.size.width * image.size.height * 4  // bytes estimate
        cache.setObject(image, forKey: key as NSString, cost: Int(cost))
    }
    
    func remove(for key: String) {
        cache.removeObject(forKey: key as NSString)
    }
}

// Memoization สำหรับ Pure Functions
class Memoize<Input: Hashable, Output> {
    private var cache: [Input: Output] = [:]
    private let function: (Input) -> Output
    
    init(_ function: @escaping (Input) -> Output) {
        self.function = function
    }
    
    func call(_ input: Input) -> Output {
        if let cached = cache[input] {
            return cached
        }
        let result = function(input)
        cache[input] = result
        return result
    }
}

// การใช้งาน
let expensiveFunction = Memoize<Int, Int> { n in
    // จำลอง expensive computation
    Thread.sleep(forTimeInterval: 0.01)
    return n * n
}

print(expensiveFunction.call(5))   // ช้า - คำนวณ
print(expensiveFunction.call(5))   // เร็ว - จาก cache
print(expensiveFunction.call(10))  // ช้า - คำนวณใหม่
```

---

## 39.15 Lazy Loading

### การ Implement Lazy Loading

```swift
import UIKit

// Lazy Loading Images ใน UITableView
class LazyImageCell: UITableViewCell {
    var imageURL: URL?
    private var currentTask: URLSessionDataTask?
    
    func configure(with url: URL) {
        imageURL = url
        
        // ตรวจสอบ Cache ก่อน
        if let cachedImage = ImageCache.shared.image(for: url.absoluteString) {
            imageView?.image = cachedImage
            return
        }
        
        // Placeholder ระหว่างโหลด
        imageView?.image = UIImage(systemName: "photo")
        
        // โหลดแบบ Async
        currentTask?.cancel()
        currentTask = URLSession.shared.dataTask(with: url) { [weak self] data, _, _ in
            guard let self = self,
                  self.imageURL == url,  // ตรวจสอบว่า cell ยังแสดง URL เดิม
                  let data = data,
                  let image = UIImage(data: data) else { return }
            
            ImageCache.shared.store(image, for: url.absoluteString)
            
            DispatchQueue.main.async {
                self.imageView?.image = image
            }
        }
        currentTask?.resume()
    }
    
    override func prepareForReuse() {
        super.prepareForReuse()
        currentTask?.cancel()
        currentTask = nil
        imageView?.image = nil
        imageURL = nil
    }
}

// Lazy Loading Data ใน List
class PaginatedDataLoader<T> {
    private var items: [T] = []
    private var currentPage = 0
    private var isLoading = false
    private var hasMoreData = true
    
    let pageSize: Int
    let loadPage: (Int, Int, @escaping ([T]) -> Void) -> Void
    
    init(pageSize: Int = 20, loadPage: @escaping (Int, Int, @escaping ([T]) -> Void) -> Void) {
        self.pageSize = pageSize
        self.loadPage = loadPage
    }
    
    func loadNextPageIfNeeded(currentIndex: Int, completion: @escaping ([T]) -> Void) {
        guard !isLoading && hasMoreData else { return }
        
        let threshold = items.count - 5  // โหลดเมื่อเหลืออีก 5 items
        guard currentIndex >= threshold else { return }
        
        isLoading = true
        loadPage(currentPage, pageSize) { [weak self] newItems in
            guard let self = self else { return }
            
            self.items.append(contentsOf: newItems)
            self.currentPage += 1
            self.hasMoreData = newItems.count == self.pageSize
            self.isLoading = false
            
            completion(self.items)
        }
    }
    
    var allItems: [T] { items }
}
```

---

## 39.16 Image Optimization

### การ Optimize Images ใน iOS

```swift
import UIKit
import ImageIO

class ImageOptimizer {
    
    // 1. ย่อขนาด Image ให้ตรงกับ Display Size
    static func resize(_ image: UIImage, to targetSize: CGSize) -> UIImage {
        let renderer = UIGraphicsImageRenderer(size: targetSize)
        return renderer.image { _ in
            image.draw(in: CGRect(origin: .zero, size: targetSize))
        }
    }
    
    // 2. Downscale Image ขณะโหลด (ประหยัด Memory มาก)
    static func loadThumbnail(from url: URL, maxDimension: CGFloat) -> UIImage? {
        let options: [CFString: Any] = [
            kCGImageSourceShouldCache: false,
            kCGImageSourceCreateThumbnailFromImageAlways: true,
            kCGImageSourceThumbnailMaxPixelSize: maxDimension,
            kCGImageSourceCreateThumbnailWithTransform: true
        ]
        
        guard let source = CGImageSourceCreateWithURL(url as CFURL, nil),
              let thumbnail = CGImageSourceCreateThumbnailAtIndex(source, 0, options as CFDictionary) else {
            return nil
        }
        
        return UIImage(cgImage: thumbnail)
    }
    
    // 3. Prefetch Images ล่วงหน้า
    static func prefetch(urls: [URL]) {
        let config = URLSessionConfiguration.background(withIdentifier: "image-prefetch")
        let session = URLSession(configuration: config)
        
        for url in urls {
            guard ImageCache.shared.image(for: url.absoluteString) == nil else { continue }
            
            session.dataTask(with: url) { data, _, _ in
                if let data = data, let image = UIImage(data: data) {
                    ImageCache.shared.store(image, for: url.absoluteString)
                }
            }.resume()
        }
    }
    
    // 4. Compress Image ก่อน Upload
    static func compress(_ image: UIImage, maxBytes: Int) -> Data? {
        var compression: CGFloat = 1.0
        var data = image.jpegData(compressionQuality: compression)
        
        while let currentData = data, currentData.count > maxBytes && compression > 0.1 {
            compression -= 0.1
            data = image.jpegData(compressionQuality: compression)
        }
        
        return data
    }
}

// การใช้งานใน UITableView
extension UIImageView {
    func loadImage(from url: URL, placeholder: UIImage? = nil) {
        image = placeholder
        
        DispatchQueue.global(qos: .userInitiated).async {
            if let cached = ImageCache.shared.image(for: url.absoluteString) {
                DispatchQueue.main.async { [weak self] in
                    self?.image = cached
                }
                return
            }
            
            if let image = ImageOptimizer.loadThumbnail(from: url, maxDimension: 200) {
                ImageCache.shared.store(image, for: url.absoluteString)
                DispatchQueue.main.async { [weak self] in
                    self?.image = image
                }
            }
        }
    }
}
```

---

## 39.17 Network Performance

### Caching และ Batching

```swift
import Foundation

// URL Cache สำหรับ HTTP Responses
class NetworkOptimizer {
    
    // 1. Configure URLCache
    static func setupURLCache() {
        let memoryCapacity = 50 * 1024 * 1024   // 50 MB Memory
        let diskCapacity = 200 * 1024 * 1024    // 200 MB Disk
        let cache = URLCache(memoryCapacity: memoryCapacity,
                            diskCapacity: diskCapacity)
        URLCache.shared = cache
    }
    
    // 2. Request Batching
    class RequestBatcher<Request, Response> {
        private var pendingRequests: [(Request, (Result<Response, Error>) -> Void)] = []
        private var batchTimer: Timer?
        private let batchDelay: TimeInterval
        private let maxBatchSize: Int
        private let executeBatch: ([Request], @escaping ([Result<Response, Error>]) -> Void) -> Void
        
        init(
            batchDelay: TimeInterval = 0.1,
            maxBatchSize: Int = 50,
            executeBatch: @escaping ([Request], @escaping ([Result<Response, Error>]) -> Void) -> Void
        ) {
            self.batchDelay = batchDelay
            self.maxBatchSize = maxBatchSize
            self.executeBatch = executeBatch
        }
        
        func add(request: Request, completion: @escaping (Result<Response, Error>) -> Void) {
            pendingRequests.append((request, completion))
            
            if pendingRequests.count >= maxBatchSize {
                flush()
            } else {
                batchTimer?.invalidate()
                batchTimer = Timer.scheduledTimer(withTimeInterval: batchDelay, repeats: false) { [weak self] _ in
                    self?.flush()
                }
            }
        }
        
        private func flush() {
            batchTimer?.invalidate()
            guard !pendingRequests.isEmpty else { return }
            
            let batch = pendingRequests
            pendingRequests.removeAll()
            
            let requests = batch.map { $0.0 }
            let completions = batch.map { $0.1 }
            
            executeBatch(requests) { results in
                for (index, result) in results.enumerated() {
                    if index < completions.count {
                        completions[index](result)
                    }
                }
            }
        }
    }
}

// 3. Conditional Requests (ETag, Last-Modified)
class ConditionalRequestManager {
    private var etags: [URL: String] = [:]
    private var lastModified: [URL: String] = [:]
    
    func request(url: URL, completion: @escaping (Data?, Bool) -> Void) {
        var request = URLRequest(url: url)
        
        if let etag = etags[url] {
            request.addValue(etag, forHTTPHeaderField: "If-None-Match")
        }
        
        if let modified = lastModified[url] {
            request.addValue(modified, forHTTPHeaderField: "If-Modified-Since")
        }
        
        URLSession.shared.dataTask(with: request) { [weak self] data, response, _ in
            guard let httpResponse = response as? HTTPURLResponse else {
                completion(nil, false)
                return
            }
            
            if httpResponse.statusCode == 304 {
                // Not Modified - ใช้ Cache
                completion(nil, true)
                return
            }
            
            if let etag = httpResponse.allHeaderFields["ETag"] as? String {
                self?.etags[url] = etag
            }
            
            if let modified = httpResponse.allHeaderFields["Last-Modified"] as? String {
                self?.lastModified[url] = modified
            }
            
            completion(data, false)
        }.resume()
    }
}
```

---

## 39.18 Background Tasks

### การจัดการ Background Tasks

```swift
import BackgroundTasks
import Foundation

// Background Task Registration
class BackgroundTaskManager {
    
    // Register Background Tasks
    static func registerTasks() {
        BGTaskScheduler.shared.register(
            forTaskWithIdentifier: "com.example.app.refresh",
            using: nil
        ) { task in
            handleAppRefresh(task: task as! BGAppRefreshTask)
        }
        
        BGTaskScheduler.shared.register(
            forTaskWithIdentifier: "com.example.app.processing",
            using: nil
        ) { task in
            handleProcessing(task: task as! BGProcessingTask)
        }
    }
    
    // Schedule App Refresh
    static func scheduleAppRefresh() {
        let request = BGAppRefreshTaskRequest(identifier: "com.example.app.refresh")
        request.earliestBeginDate = Date(timeIntervalSinceNow: 15 * 60)  // 15 minutes
        
        do {
            try BGTaskScheduler.shared.submit(request)
            print("Background refresh scheduled")
        } catch {
            print("Failed to schedule: \(error)")
        }
    }
    
    // Handle App Refresh
    static func handleAppRefresh(task: BGAppRefreshTask) {
        scheduleAppRefresh()  // Schedule next refresh
        
        let refreshOperation = DataRefreshOperation()
        
        task.expirationHandler = {
            refreshOperation.cancel()
        }
        
        refreshOperation.completionBlock = {
            task.setTaskCompleted(success: !refreshOperation.isCancelled)
        }
        
        OperationQueue().addOperation(refreshOperation)
    }
    
    // Handle Processing Task
    static func handleProcessing(task: BGProcessingTask) {
        let operation = HeavyProcessingOperation()
        
        task.expirationHandler = {
            operation.cancel()
        }
        
        operation.completionBlock = {
            task.setTaskCompleted(success: !operation.isCancelled)
        }
        
        OperationQueue().addOperation(operation)
    }
}

class DataRefreshOperation: Operation {
    override func main() {
        guard !isCancelled else { return }
        // Fetch new data
        print("Refreshing data in background...")
        Thread.sleep(forTimeInterval: 2)
    }
}

class HeavyProcessingOperation: Operation {
    override func main() {
        guard !isCancelled else { return }
        print("Heavy processing in background...")
        Thread.sleep(forTimeInterval: 5)
    }
}

// UIBackgroundTask สำหรับ Short Tasks
class ShortBackgroundTask {
    private var backgroundTaskID: UIBackgroundTaskIdentifier = .invalid
    
    func beginTask(name: String, completion: @escaping () -> Void) {
        backgroundTaskID = UIApplication.shared.beginBackgroundTask(withName: name) {
            // Expiration handler
            self.endTask()
        }
        
        DispatchQueue.global(qos: .userInitiated).async {
            completion()
            self.endTask()
        }
    }
    
    private func endTask() {
        if backgroundTaskID != .invalid {
            UIApplication.shared.endBackgroundTask(backgroundTaskID)
            backgroundTaskID = .invalid
        }
    }
}
```

---

## 39.19 Main Thread Rules

### กฎการใช้ Main Thread

```swift
import UIKit

// กฎสำคัญ: UI Updates ต้องทำบน Main Thread เสมอ

class MainThreadSafeViewController: UIViewController {
    
    @IBOutlet weak var label: UILabel!
    @IBOutlet weak var tableView: UITableView!
    
    func loadData() {
        // 1. โหลดข้อมูลบน Background Thread
        DispatchQueue.global(qos: .userInitiated).async {
            let data = self.fetchData()
            
            // 2. อัปเดต UI บน Main Thread เสมอ
            DispatchQueue.main.async { [weak self] in
                self?.label.text = "Loaded \(data.count) items"
                self?.tableView.reloadData()
            }
        }
    }
    
    func fetchData() -> [String] {
        Thread.sleep(forTimeInterval: 1)  // จำลอง Network Call
        return ["Item 1", "Item 2", "Item 3"]
    }
    
    // Thread-safe UI Update Helper
    func safeMainThread(_ block: @escaping () -> Void) {
        if Thread.isMainThread {
            block()
        } else {
            DispatchQueue.main.async(execute: block)
        }
    }
    
    // การตรวจสอบ Main Thread ใน Debug
    func updateUI() {
        assert(Thread.isMainThread, "UI updates must be on main thread!")
        label.text = "Updated"
    }
}

// Main Actor ใน Swift Concurrency (Swift 5.5+)
@MainActor
class ModernViewController {
    var labelText: String = ""
    
    func loadAndUpdateUI() async {
        // ทำงานบน Background Thread อัตโนมัติ
        let data = await fetchData()
        
        // กลับมา Main Thread อัตโนมัติ (เพราะ @MainActor)
        labelText = "Loaded \(data.count) items"
    }
    
    func fetchData() async -> [String] {
        // Async operation
        try? await Task.sleep(nanoseconds: 1_000_000_000)
        return ["Item 1", "Item 2", "Item 3"]
    }
}
```

---

## 39.20 Concurrent Data Access

### Thread Safety และ Data Race Prevention

```swift
import Foundation

// Thread-Unsafe Example (อย่าทำแบบนี้!)
class UnsafeCounter {
    var count = 0  // Data Race! หลาย Thread เขียนพร้อมกัน
    
    func increment() {
        count += 1  // Not Atomic!
    }
}

// ทางแก้ 1: DispatchQueue (Serial Queue)
class SafeCounterWithQueue {
    private var count = 0
    private let queue = DispatchQueue(label: "com.example.counter")
    
    func increment() {
        queue.async {
            self.count += 1
        }
    }
    
    func getCount() -> Int {
        return queue.sync { count }
    }
}

// ทางแก้ 2: NSLock
class SafeCounterWithLock {
    private var count = 0
    private let lock = NSLock()
    
    func increment() {
        lock.lock()
        count += 1
        lock.unlock()
    }
    
    // Better: ใช้ withLock pattern
    func incrementSafe() {
        lock.withLock {
            count += 1
        }
    }
    
    var value: Int {
        return lock.withLock { count }
    }
}

extension NSLock {
    @discardableResult
    func withLock<T>(_ block: () throws -> T) rethrows -> T {
        lock()
        defer { unlock() }
        return try block()
    }
}

// ทางแก้ 3: Actor (Swift 5.5+)
actor SafeCounterActor {
    private var count = 0
    
    func increment() {
        count += 1  // Safe - Actor ensures single access
    }
    
    func getCount() -> Int {
        return count
    }
}

// การใช้งาน Actor
func useActor() async {
    let counter = SafeCounterActor()
    
    await withTaskGroup(of: Void.self) { group in
        for _ in 0..<100 {
            group.addTask {
                await counter.increment()
            }
        }
    }
    
    print("Final count:", await counter.getCount())  // จะได้ 100 เสมอ
}

// ทางแก้ 4: Concurrent Queue + Barrier (Reader-Writer Pattern)
class ThreadSafeCache<Key: Hashable, Value> {
    private var cache: [Key: Value] = [:]
    private let queue = DispatchQueue(
        label: "com.example.cache",
        attributes: .concurrent  // Concurrent สำหรับการอ่าน
    )
    
    func get(_ key: Key) -> Value? {
        return queue.sync { cache[key] }  // Concurrent read
    }
    
    func set(_ key: Key, value: Value) {
        queue.async(flags: .barrier) {  // Exclusive write
            self.cache[key] = value
        }
    }
    
    func remove(_ key: Key) {
        queue.async(flags: .barrier) {
            self.cache.removeValue(forKey: key)
        }
    }
}
```

---

## 39.21 Grand Central Dispatch (GCD)

### GCD Fundamentals

```swift
import Foundation

class GCDExamples {
    
    // 1. Serial Queue: ทำงานทีละอย่าง
    let serialQueue = DispatchQueue(label: "com.example.serial")
    
    // 2. Concurrent Queue: ทำงานพร้อมกันได้
    let concurrentQueue = DispatchQueue(
        label: "com.example.concurrent",
        attributes: .concurrent
    )
    
    // 3. Global Queue: System-provided Concurrent Queues
    func useGlobalQueues() {
        // QoS (Quality of Service) levels
        DispatchQueue.global(qos: .userInteractive).async {
            // สำหรับงานที่ User กำลังรอ (highest priority)
        }
        
        DispatchQueue.global(qos: .userInitiated).async {
            // สำหรับงานที่ User เริ่มต้น เช่น tap button
        }
        
        DispatchQueue.global(qos: .utility).async {
            // สำหรับงาน long-running เช่น download
        }
        
        DispatchQueue.global(qos: .background).async {
            // สำหรับงาน invisible to user เช่น backup
        }
    }
    
    // 4. Main Queue
    func useMainQueue() {
        DispatchQueue.main.async {
            // UI Updates
        }
        
        // DispatchQueue.main.sync จาก Main Thread จะ Deadlock!
        // อย่าทำแบบนี้ถ้าอยู่บน Main Thread:
        // DispatchQueue.main.sync { ... }  // DEADLOCK!
    }
    
    // 5. Async/Sync Operations
    func demonstrateAsyncSync() {
        print("1 - Main Thread")
        
        // async: return ทันที, block ทำงานทีหลัง
        serialQueue.async {
            print("3 - Async Block")
        }
        
        print("2 - Main Thread continues")
        
        // sync: block จนกว่า block จะเสร็จ
        serialQueue.sync {
            print("4 - Sync Block")
        }
        
        print("5 - After sync")
    }
    
    // 6. DispatchWorkItem
    func useWorkItem() {
        let workItem = DispatchWorkItem {
            print("Work item executing")
        }
        
        DispatchQueue.global().async(execute: workItem)
        
        // รอผล
        workItem.notify(queue: .main) {
            print("Work item completed")
        }
        
        // Cancel ถ้าจำเป็น
        workItem.cancel()
    }
    
    // 7. Delayed Execution
    func delayedExecution() {
        DispatchQueue.main.asyncAfter(deadline: .now() + 2.0) {
            print("Executed after 2 seconds")
        }
    }
}
```

---

## 39.22 DispatchQueue

### Advanced DispatchQueue Usage

```swift
import Foundation

// DispatchQueue สำหรับ Thread-Safe Operations
class DataStore {
    private var data: [String: Any] = [:]
    private let accessQueue = DispatchQueue(
        label: "com.example.datastore",
        qos: .userInitiated,
        attributes: .concurrent
    )
    
    // Thread-safe Read
    func read(key: String) -> Any? {
        return accessQueue.sync {
            data[key]
        }
    }
    
    // Thread-safe Write
    func write(key: String, value: Any) {
        accessQueue.async(flags: .barrier) {
            self.data[key] = value
        }
    }
    
    // Thread-safe Multiple Reads (การอ่านพร้อมกันได้)
    func readMultiple(keys: [String]) -> [String: Any] {
        return accessQueue.sync {
            keys.reduce(into: [:]) { result, key in
                result[key] = data[key]
            }
        }
    }
}

// DispatchQueue Priority Inheritance
class PriorityAwareTask {
    
    func performTask(priority: DispatchQoS.QoSClass) {
        let queue = DispatchQueue(
            label: "com.example.task",
            qos: DispatchQoS(qosClass: priority, relativePriority: 0)
        )
        
        queue.async {
            // งานนี้จะได้ priority ตามที่กำหนด
            print("Working at priority: \(priority)")
        }
    }
    
    // Target Queue สำหรับ Priority Propagation
    func setupQueueHierarchy() {
        let rootQueue = DispatchQueue(
            label: "com.example.root",
            qos: .userInitiated,
            attributes: .concurrent
        )
        
        let childQueue = DispatchQueue(
            label: "com.example.child",
            target: rootQueue  // inherit priority from root
        )
        
        childQueue.async {
            print("Child queue task")
        }
    }
}

// DispatchSpecificKey สำหรับ Queue Detection
let specificKey = DispatchSpecificKey<String>()

class QueueAwareService {
    private let queue = DispatchQueue(label: "com.example.service")
    
    init() {
        queue.setSpecific(key: specificKey, value: "service-queue")
    }
    
    func performWork() {
        if DispatchQueue.getSpecific(key: specificKey) == "service-queue" {
            // เราอยู่บน queue ของ service นี้แล้ว
            doWork()
        } else {
            queue.async { self.doWork() }
        }
    }
    
    private func doWork() {
        print("Doing work on service queue")
    }
}
```

---

## 39.23 DispatchGroup

### DispatchGroup สำหรับ Coordinating Multiple Tasks

```swift
import Foundation

class NetworkManager {
    
    // รอหลาย Network Calls พร้อมกัน
    func fetchAllData(completion: @escaping ([String: Any]) -> Void) {
        let group = DispatchGroup()
        var results: [String: Any] = [:]
        let lock = NSLock()
        
        // Fetch User Data
        group.enter()
        fetchUser { user in
            lock.withLock { results["user"] = user }
            group.leave()
        }
        
        // Fetch Product Data
        group.enter()
        fetchProducts { products in
            lock.withLock { results["products"] = products }
            group.leave()
        }
        
        // Fetch Settings
        group.enter()
        fetchSettings { settings in
            lock.withLock { results["settings"] = settings }
            group.leave()
        }
        
        // รอทุก fetch เสร็จ
        group.notify(queue: .main) {
            completion(results)
        }
    }
    
    // รอด้วย Timeout
    func fetchWithTimeout(completion: @escaping (Bool) -> Void) {
        let group = DispatchGroup()
        
        group.enter()
        DispatchQueue.global().async {
            Thread.sleep(forTimeInterval: 2)
            group.leave()
        }
        
        // รอสูงสุด 3 วินาที
        let result = group.wait(timeout: .now() + 3)
        
        switch result {
        case .success:
            completion(true)
        case .timedOut:
            completion(false)
        }
    }
    
    private func fetchUser(completion: @escaping (String) -> Void) {
        DispatchQueue.global().asyncAfter(deadline: .now() + 0.5) {
            completion("John Doe")
        }
    }
    
    private func fetchProducts(completion: @escaping ([String]) -> Void) {
        DispatchQueue.global().asyncAfter(deadline: .now() + 0.8) {
            completion(["Product A", "Product B"])
        }
    }
    
    private func fetchSettings(completion: @escaping ([String: Bool]) -> Void) {
        DispatchQueue.global().asyncAfter(deadline: .now() + 0.3) {
            completion(["darkMode": true, "notifications": false])
        }
    }
}

// DispatchGroup กับ Async/Await
func fetchAllDataModern() async -> [String: Any] {
    async let user = fetchUserAsync()
    async let products = fetchProductsAsync()
    async let settings = fetchSettingsAsync()
    
    let (u, p, s) = await (user, products, settings)
    return ["user": u, "products": p, "settings": s]
}

func fetchUserAsync() async -> String {
    try? await Task.sleep(nanoseconds: 500_000_000)
    return "John Doe"
}

func fetchProductsAsync() async -> [String] {
    try? await Task.sleep(nanoseconds: 800_000_000)
    return ["Product A", "Product B"]
}

func fetchSettingsAsync() async -> [String: Bool] {
    try? await Task.sleep(nanoseconds: 300_000_000)
    return ["darkMode": true]
}
```

---

## 39.24 DispatchSemaphore

### DispatchSemaphore สำหรับ Resource Control

```swift
import Foundation

class ResourceLimiter {
    // จำกัดการเข้าถึง Resource พร้อมกัน
    private let semaphore: DispatchSemaphore
    
    init(maxConcurrent: Int) {
        semaphore = DispatchSemaphore(value: maxConcurrent)
    }
    
    func withResource<T>(_ work: () throws -> T) rethrows -> T {
        semaphore.wait()      // ขอ permit
        defer { semaphore.signal() }  // คืน permit เมื่อเสร็จ
        return try work()
    }
}

// จำกัด Concurrent Network Requests
class LimitedNetworkManager {
    private let semaphore = DispatchSemaphore(value: 5)  // Max 5 concurrent
    
    func fetch(url: URL, completion: @escaping (Data?) -> Void) {
        DispatchQueue.global().async {
            self.semaphore.wait()  // รอถ้า 5 requests กำลังทำงานอยู่
            
            URLSession.shared.dataTask(with: url) { data, _, _ in
                self.semaphore.signal()  // อนุญาตให้ request ถัดไปเริ่มได้
                completion(data)
            }.resume()
        }
    }
    
    func fetchBatch(urls: [URL], completion: @escaping ([Data?]) -> Void) {
        let group = DispatchGroup()
        var results = Array<Data?>(repeating: nil, count: urls.count)
        let lock = NSLock()
        
        for (index, url) in urls.enumerated() {
            group.enter()
            fetch(url: url) { data in
                lock.withLock { results[index] = data }
                group.leave()
            }
        }
        
        group.notify(queue: .main) {
            completion(results)
        }
    }
}

// Semaphore ใน Async Context
class AsyncSemaphore {
    private var permits: Int
    private var waiters: [CheckedContinuation<Void, Never>] = []
    private let lock = NSLock()
    
    init(value: Int) {
        permits = value
    }
    
    func wait() async {
        await withCheckedContinuation { continuation in
            lock.withLock {
                if permits > 0 {
                    permits -= 1
                    continuation.resume()
                } else {
                    waiters.append(continuation)
                }
            }
        }
    }
    
    func signal() {
        lock.withLock {
            if let waiter = waiters.first {
                waiters.removeFirst()
                waiter.resume()
            } else {
                permits += 1
            }
        }
    }
}
```

---

## 39.25 OperationQueue

### OperationQueue สำหรับ Complex Dependencies

```swift
import Foundation

// Custom Operation
class DataProcessingOperation: Operation {
    private let inputData: Data
    private(set) var result: ProcessedData?
    
    struct ProcessedData {
        let count: Int
        let checksum: Int
    }
    
    init(data: Data) {
        self.inputData = data
    }
    
    override func main() {
        guard !isCancelled else { return }
        
        // จำลอง Heavy Processing
        var checksum = 0
        for byte in inputData {
            guard !isCancelled else { return }
            checksum ^= Int(byte)
        }
        
        result = ProcessedData(count: inputData.count, checksum: checksum)
    }
}

// Download Operation
class DownloadOperation: Operation {
    let url: URL
    private(set) var downloadedData: Data?
    
    init(url: URL) {
        self.url = url
    }
    
    override var isAsynchronous: Bool { true }
    
    private var _isExecuting = false
    private var _isFinished = false
    
    override var isExecuting: Bool {
        get { _isExecuting }
        set {
            willChangeValue(forKey: "isExecuting")
            _isExecuting = newValue
            didChangeValue(forKey: "isExecuting")
        }
    }
    
    override var isFinished: Bool {
        get { _isFinished }
        set {
            willChangeValue(forKey: "isFinished")
            _isFinished = newValue
            didChangeValue(forKey: "isFinished")
        }
    }
    
    override func start() {
        guard !isCancelled else {
            isFinished = true
            return
        }
        
        isExecuting = true
        
        URLSession.shared.dataTask(with: url) { [weak self] data, _, _ in
            self?.downloadedData = data
            self?.isExecuting = false
            self?.isFinished = true
        }.resume()
    }
}

// OperationQueue พร้อม Dependencies
class PipelineManager {
    let queue = OperationQueue()
    
    func runPipeline(urls: [URL]) {
        queue.maxConcurrentOperationCount = 3
        
        var processingOps: [DataProcessingOperation] = []
        
        for url in urls {
            let downloadOp = DownloadOperation(url: url)
            
            let processOp = BlockOperation { [weak downloadOp] in
                guard let data = downloadOp?.downloadedData else { return }
                // Process downloaded data
                print("Processing \(data.count) bytes from \(url)")
            }
            
            // processOp ต้องรอ downloadOp เสร็จก่อน
            processOp.addDependency(downloadOp)
            
            queue.addOperation(downloadOp)
            queue.addOperation(processOp)
        }
        
        // Completion Operation รอทุกอย่างเสร็จ
        let completionOp = BlockOperation {
            print("All downloads and processing complete!")
        }
        
        queue.operations.forEach { completionOp.addDependency($0) }
        queue.addOperation(completionOp)
    }
    
    func cancelAll() {
        queue.cancelAllOperations()
    }
}
```

---

## 39.26 Practical Exercises

### แบบฝึกหัดที่ 1: Optimizing a Slow Function

```swift
// โจทย์: ปรับปรุงฟังก์ชันนี้ให้เร็วขึ้น
// โค้ดเดิม (ช้ามาก)
func findCommonElements_slow(_ arr1: [Int], _ arr2: [Int]) -> [Int] {
    var result = [Int]()
    for item1 in arr1 {
        for item2 in arr2 {
            if item1 == item2 && !result.contains(item1) {
                result.append(item1)
            }
        }
    }
    return result
}
// O(n³) - ช้ามาก!

// เฉลย: ใช้ Set Operations
func findCommonElements_fast(_ arr1: [Int], _ arr2: [Int]) -> [Int] {
    let set1 = Set(arr1)
    let set2 = Set(arr2)
    return Array(set1.intersection(set2))
}
// O(n) - เร็วกว่ามาก!

// ทดสอบ
func exercise1() {
    let arr1 = Array(0..<1000).shuffled()
    let arr2 = Array(500..<1500).shuffled()
    
    let start1 = CFAbsoluteTimeGetCurrent()
    // let _ = findCommonElements_slow(arr1, arr2)  // ช้าเกินไปสำหรับทดสอบ
    let time1 = CFAbsoluteTimeGetCurrent() - start1
    
    let start2 = CFAbsoluteTimeGetCurrent()
    let _ = findCommonElements_fast(arr1, arr2)
    let time2 = CFAbsoluteTimeGetCurrent() - start2
    
    print("Fast: \(time2 * 1000)ms")
}
```

### แบบฝึกหัดที่ 2: Thread-Safe Singleton

```swift
// โจทย์: สร้าง Thread-Safe Singleton ที่สามารถ initialize แบบ Lazy ได้

// เฉลย
final class ThreadSafeSingleton {
    // Swift Singleton Pattern - Thread-safe โดย default
    static let shared = ThreadSafeSingleton()
    
    private var data: [String: Any] = [:]
    private let queue = DispatchQueue(
        label: "com.example.singleton",
        attributes: .concurrent
    )
    
    private init() {
        // Private init ป้องกันการสร้าง instance เพิ่ม
        setupInitialData()
    }
    
    private func setupInitialData() {
        data["version"] = "1.0"
        data["initialized"] = Date()
    }
    
    func getValue(for key: String) -> Any? {
        return queue.sync { data[key] }
    }
    
    func setValue(_ value: Any, for key: String) {
        queue.async(flags: .barrier) {
            self.data[key] = value
        }
    }
}

// การใช้งาน
func exercise2() {
    let singleton = ThreadSafeSingleton.shared
    
    // Concurrent reads
    DispatchQueue.concurrentPerform(iterations: 100) { _ in
        let _ = singleton.getValue(for: "version")
    }
    
    // Concurrent writes
    DispatchQueue.concurrentPerform(iterations: 10) { i in
        singleton.setValue("value\(i)", for: "key\(i)")
    }
    
    print("Singleton working correctly")
}
```

### แบบฝึกหัดที่ 3: Async Image Loader

```swift
import UIKit

// โจทย์: สร้าง Image Loader ที่มีประสิทธิภาพ พร้อม Cache และ Cancel support

class EfficientImageLoader {
    private let cache = NSCache<NSString, UIImage>()
    private var activeTasks: [URL: URLSessionDataTask] = [:]
    private let taskLock = NSLock()
    
    // โหลด Image พร้อม Cache
    func loadImage(
        from url: URL,
        completion: @escaping (UIImage?) -> Void
    ) -> CancellationToken {
        let token = CancellationToken()
        
        // ตรวจสอบ Cache
        if let cached = cache.object(forKey: url.absoluteString as NSString) {
            completion(cached)
            return token
        }
        
        // สร้าง Task
        let task = URLSession.shared.dataTask(with: url) { [weak self] data, _, _ in
            guard !token.isCancelled else { return }
            
            if let data = data, let image = UIImage(data: data) {
                self?.cache.setObject(image, forKey: url.absoluteString as NSString)
                DispatchQueue.main.async {
                    completion(image)
                }
            } else {
                DispatchQueue.main.async {
                    completion(nil)
                }
            }
            
            self?.taskLock.withLock {
                self?.activeTasks.removeValue(forKey: url)
            }
        }
        
        taskLock.withLock {
            activeTasks[url] = task
        }
        
        token.onCancel = { [weak self] in
            self?.taskLock.withLock {
                self?.activeTasks[url]?.cancel()
                self?.activeTasks.removeValue(forKey: url)
            }
        }
        
        task.resume()
        return token
    }
}

class CancellationToken {
    private(set) var isCancelled = false
    var onCancel: (() -> Void)?
    private let lock = NSLock()
    
    func cancel() {
        lock.withLock {
            guard !isCancelled else { return }
            isCancelled = true
            onCancel?()
        }
    }
}

// การใช้งาน
func exercise3() {
    let loader = EfficientImageLoader()
    guard let url = URL(string: "https://example.com/image.jpg") else { return }
    
    let token = loader.loadImage(from: url) { image in
        if let image = image {
            print("Image loaded: \(image.size)")
        }
    }
    
    // Cancel ถ้าต้องการ
    // token.cancel()
    
    _ = token  // Suppress warning
}
```

### แบบฝึกหัดที่ 4: Performance Benchmark

```swift
import Foundation

// โจทย์: สร้าง Benchmark Tool เพื่อเปรียบเทียบ Algorithm ต่างๆ

struct Benchmark {
    let name: String
    let iterations: Int
    
    func run(_ block: () -> Void) -> TimeInterval {
        // Warmup
        for _ in 0..<5 {
            block()
        }
        
        // Measure
        let start = CFAbsoluteTimeGetCurrent()
        for _ in 0..<iterations {
            block()
        }
        let elapsed = CFAbsoluteTimeGetCurrent() - start
        
        return elapsed / Double(iterations)
    }
    
    func compare(_ algorithms: [(name: String, block: () -> Void)]) {
        print("\n=== Benchmark: \(name) ===")
        print("Iterations: \(iterations)")
        
        var times: [(String, TimeInterval)] = []
        
        for algorithm in algorithms {
            let time = run(algorithm.block)
            times.append((algorithm.name, time))
            print("\(algorithm.name): \(String(format: "%.4f", time * 1000))ms per iteration")
        }
        
        if let fastest = times.min(by: { $0.1 < $1.1 }) {
            print("Fastest: \(fastest.0)")
        }
    }
}

// การใช้งาน
func exercise4() {
    let numbers = (0..<10000).map { _ in Int.random(in: 0..<100000) }
    
    let benchmark = Benchmark(name: "Sorting Comparison", iterations: 100)
    
    benchmark.compare([
        ("sorted()", { let _ = numbers.sorted() }),
        ("sort()", { var copy = numbers; copy.sort() }),
        ("mergeSort", { let _ = mergeSort(numbers) })
    ])
    
    let findBenchmark = Benchmark(name: "Search Comparison", iterations: 1000)
    let sortedNumbers = numbers.sorted()
    let target = numbers.randomElement()!
    
    findBenchmark.compare([
        ("contains (Array)", { let _ = numbers.contains(target) }),
        ("contains (Set)", { let s = Set(numbers); let _ = s.contains(target) }),
        ("binarySearch", { let _ = binarySearch(sortedNumbers, target: target) })
    ])
}

exercise4()
```

---

## 39.27 สรุป

### สิ่งที่เรียนรู้ในบทนี้

1. **Performance Fundamentals**: ความเข้าใจ Responsiveness, Throughput, Memory, Battery
2. **Big O Notation**: การวิเคราะห์ Time และ Space Complexity
3. **Value vs Reference Types**: การเลือกใช้ Struct/Class อย่างเหมาะสม
4. **Copy-on-Write**: การใช้ CoW เพื่อประหยัด Memory
5. **Lazy Evaluation**: การชะลอการคำนวณจนกว่าจะจำเป็น
6. **Instruments**: เครื่องมือ Profiling อย่าง Time Profiler
7. **Memory Optimization**: การลด Allocations, Retain Cycles
8. **String Performance**: การ Optimize String Operations
9. **Collection Choice**: การเลือก Array/Set/Dictionary อย่างถูกต้อง
10. **Caching**: NSCache, Custom Cache, Memoization
11. **Lazy Loading**: Image Loading, Pagination
12. **Network Optimization**: Caching, Batching, Conditional Requests
13. **Background Tasks**: BGTaskScheduler, UIBackgroundTask
14. **Thread Safety**: Lock, Semaphore, Actor
15. **GCD**: DispatchQueue, DispatchGroup, DispatchSemaphore
16. **OperationQueue**: Dependencies, Async Operations

### หลักการสำคัญที่ต้องจำ

```swift
// 1. วัดก่อน Optimize เสมอ
// 2. ใช้ Value Types เมื่อทำได้
// 3. UI Updates บน Main Thread เสมอ
// 4. หลีกเลี่ยง Retain Cycles ด้วย [weak self]
// 5. ใช้ Cache อย่างชาญฉลาด
// 6. Lazy Load เมื่อ Resources มีราคาแพง
// 7. ใช้ Concurrent Queue อย่างระมัดระวัง
// 8. เลือก Algorithm ที่เหมาะสมกับ Use Case
```

### แนวทางการ Profile ก่อน Optimize

```
1. เปิด Instruments (⌘+I ใน Xcode)
2. เลือก Time Profiler
3. Record แอป
4. ดู Heavy Thread ใน Call Tree
5. ระบุ Bottleneck (Self Time สูง)
6. Optimize เฉพาะส่วนนั้น
7. วัดผลอีกครั้งเพื่อยืนยัน
```

---

*ในบทถัดไป เราจะเรียนรู้เรื่อง Security Best Practices ใน iOS ซึ่งเป็นสิ่งสำคัญไม่แพ้ Performance*
