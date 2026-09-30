# Part 94: App Performance Advanced (การเพิ่มประสิทธิภาพแอปขั้นสูง)

## บทนำ

Performance optimization คือกระบวนการที่ต้องอาศัยทั้งศาสตร์และศิลป์ บทนี้จะครอบคลุมเทคนิคขั้นสูงในการวัดผล วิเคราะห์ และแก้ไขปัญหา performance ในแอป iOS ตั้งแต่ระดับ CPU จนถึง UI rendering

---

## 1. Performance Measurement Science (วิทยาศาสตร์การวัด Performance)

### 1.1 Amdahl's Law กับ Mobile Apps

Amdahl's Law บอกว่า speedup สูงสุดที่ได้จากการ parallelize คือ 1/(1-p) เมื่อ p = สัดส่วนของ code ที่ parallelize ได้

```swift
import Foundation

// Amdahl's Law calculator
struct AmdahlsLaw {
    /// คำนวณ speedup สูงสุดจาก parallelization
    /// - Parameters:
    ///   - parallelFraction: สัดส่วนของ code ที่ parallelize ได้ (0.0 - 1.0)
    ///   - processorCount: จำนวน processor/core
    /// - Returns: Speedup factor
    static func maxSpeedup(parallelFraction p: Double, processorCount n: Int) -> Double {
        let serial = 1.0 - p
        let parallel = p / Double(n)
        return 1.0 / (serial + parallel)
    }
    
    /// คำนวณ speedup จริงบน iPhone
    static func analyzeIOSApp() {
        let cores = ProcessInfo.processInfo.processorCount
        print("Active processor cores: \(cores)")
        
        // Scenario: Image processing pipeline
        // 20% serial (load, decode), 80% parallel (filters)
        let imageSpeedup = maxSpeedup(parallelFraction: 0.80, processorCount: cores)
        print("Image pipeline speedup: \(String(format: "%.2f", imageSpeedup))x")
        
        // Scenario: Network + parsing
        // 40% serial (parsing), 60% parallel (concurrent requests)
        let networkSpeedup = maxSpeedup(parallelFraction: 0.60, processorCount: cores)
        print("Network pipeline speedup: \(String(format: "%.2f", networkSpeedup))x")
        
        // ข้อสรุป: serial bottleneck ที่ 20% จำกัด speedup ไว้ที่ ~5x
        // แม้จะมี core มากเท่าไหร่ก็ตาม
    }
}

// การวัด actual speedup
func measureSpeedup() async {
    let items = Array(0..<1000)
    
    // Serial execution
    let serialStart = ContinuousClock.now
    var serialResults: [Int] = []
    for item in items {
        serialResults.append(heavyWork(item))
    }
    let serialTime = ContinuousClock.now - serialStart
    
    // Parallel execution
    let parallelStart = ContinuousClock.now
    let parallelResults = await withTaskGroup(of: Int.self) { group in
        for item in items {
            group.addTask { heavyWork(item) }
        }
        var results: [Int] = []
        for await r in group { results.append(r) }
        return results
    }
    let parallelTime = ContinuousClock.now - parallelStart
    
    let speedup = Double(serialTime.components.seconds * 1_000_000_000 + serialTime.components.attoseconds / 1_000_000_000) /
                  Double(parallelTime.components.seconds * 1_000_000_000 + parallelTime.components.attoseconds / 1_000_000_000)
    
    print("Actual speedup: \(String(format: "%.2f", speedup))x")
    print("Serial: \(serialResults.count) | Parallel: \(parallelResults.count)")
}

func heavyWork(_ n: Int) -> Int {
    var result = n
    for _ in 0..<10000 { result = (result &* 1664525 &+ 1013904223) }
    return result
}
```

### 1.2 Profiling Methodology

```
กระบวนการ: Measure → Identify → Fix → Verify (วนซ้ำ)

1. MEASURE: วัด baseline ก่อนทำอะไร
2. IDENTIFY: หา bottleneck จริงๆ ไม่ใช่แค่เดา
3. FIX: แก้ปัญหาทีละอย่าง
4. VERIFY: วัดผลอีกครั้งเพื่อยืนยัน improvement
```

```swift
// Benchmark framework สำหรับ methodology
struct PerformanceBenchmark {
    struct Result {
        let name: String
        let median: Duration
        let mean: Duration
        let stdDev: Duration
        let min: Duration
        let max: Duration
        let samples: Int
        
        var isSignificant: Bool {
            // CV (Coefficient of Variation) < 5% ถือว่าผล stable
            let cvPercent = (stdDev.toSeconds() / mean.toSeconds()) * 100
            return cvPercent < 5.0
        }
    }
    
    /// รัน benchmark และ return statistical results
    static func measure(
        name: String,
        iterations: Int = 100,
        warmupIterations: Int = 10,
        operation: () throws -> Void
    ) rethrows -> Result {
        
        // Warmup - ให้ JIT/cache warm up
        for _ in 0..<warmupIterations {
            try operation()
        }
        
        // Actual measurements
        var samples: [Duration] = []
        let clock = ContinuousClock()
        
        for _ in 0..<iterations {
            let start = clock.now
            try operation()
            let elapsed = clock.now - start
            samples.append(elapsed)
        }
        
        samples.sort { $0 < $1 }
        
        let median = samples[samples.count / 2]
        let sum = samples.reduce(Duration.zero) { $0 + $1 }
        let meanSeconds = sum.toSeconds() / Double(samples.count)
        let mean = Duration.seconds(meanSeconds)
        
        let variance = samples.map { pow($0.toSeconds() - meanSeconds, 2) }.reduce(0, +) / Double(samples.count)
        let stdDev = Duration.seconds(sqrt(variance))
        
        return Result(
            name: name,
            median: median,
            mean: mean,
            stdDev: stdDev,
            min: samples.first!,
            max: samples.last!,
            samples: iterations
        )
    }
    
    static func compare(baseline: Result, optimized: Result) {
        let speedup = baseline.median.toSeconds() / optimized.median.toSeconds()
        let improvement = (1 - optimized.median.toSeconds() / baseline.median.toSeconds()) * 100
        
        print("=== Benchmark Comparison ===")
        print("Baseline '\(baseline.name)': \(baseline.median.formatted())")
        print("Optimized '\(optimized.name)': \(optimized.median.formatted())")
        print("Speedup: \(String(format: "%.2f", speedup))x")
        print("Improvement: \(String(format: "%.1f", improvement))%")
        print("Statistically significant: \(baseline.isSignificant && optimized.isSignificant)")
    }
}

extension Duration {
    func toSeconds() -> Double {
        let (seconds, attoseconds) = components
        return Double(seconds) + Double(attoseconds) * 1e-18
    }
    
    func formatted() -> String {
        let s = toSeconds()
        if s < 0.000001 { return String(format: "%.2f ns", s * 1e9) }
        if s < 0.001 { return String(format: "%.2f µs", s * 1e6) }
        if s < 1 { return String(format: "%.2f ms", s * 1000) }
        return String(format: "%.3f s", s)
    }
}
```

### 1.3 Statistical Significance ใน Benchmarks

```swift
// T-test สำหรับ benchmark comparison
struct StatisticalTest {
    /// Welch's t-test - เปรียบเทียบ two samples
    static func tTest(sample1: [Double], sample2: [Double]) -> (tStatistic: Double, significant: Bool) {
        let n1 = Double(sample1.count)
        let n2 = Double(sample2.count)
        
        let mean1 = sample1.reduce(0, +) / n1
        let mean2 = sample2.reduce(0, +) / n2
        
        let var1 = sample1.map { pow($0 - mean1, 2) }.reduce(0, +) / (n1 - 1)
        let var2 = sample2.map { pow($0 - mean2, 2) }.reduce(0, +) / (n2 - 1)
        
        let se = sqrt(var1/n1 + var2/n2)
        let t = abs(mean1 - mean2) / se
        
        // Critical value t=2.0 สำหรับ p<0.05 (approximation)
        return (t, t > 2.0)
    }
    
    static func runComparison(
        iterations: Int = 50,
        baseline: () -> Void,
        optimized: () -> Void
    ) {
        var baselineSamples: [Double] = []
        var optimizedSamples: [Double] = []
        let clock = ContinuousClock()
        
        // Warmup
        for _ in 0..<10 { baseline(); optimized() }
        
        for _ in 0..<iterations {
            let s = clock.now; baseline()
            baselineSamples.append((clock.now - s).toSeconds())
            
            let s2 = clock.now; optimized()
            optimizedSamples.append((clock.now - s2).toSeconds())
        }
        
        let (t, significant) = tTest(sample1: baselineSamples, sample2: optimizedSamples)
        let speedup = baselineSamples.reduce(0,+)/Double(iterations) /
                      (optimizedSamples.reduce(0,+)/Double(iterations))
        
        print("t-statistic: \(String(format: "%.2f", t))")
        print("Statistically significant: \(significant)")
        print("Mean speedup: \(String(format: "%.2f", speedup))x")
    }
}
```

### 1.4 swift-benchmark Framework

```swift
// Package.swift
// .package(url: "https://github.com/google/swift-benchmark", from: "0.1.0")

import Benchmark

// ตัวอย่าง benchmark suite
let suite = BenchmarkSuite(name: "StringOperations") { suite in
    let data = Array(0..<1000).map { "item_\($0)" }
    
    suite.benchmark("joined(separator:)") {
        _ = data.joined(separator: ", ")
    }
    
    suite.benchmark("reduce with +") {
        _ = data.reduce("") { $0 + ($0.isEmpty ? "" : ", ") + $1 }
    }
    
    suite.benchmark("NSMutableString") {
        let result = NSMutableString()
        for (i, s) in data.enumerated() {
            if i > 0 { result.append(", ") }
            result.append(s)
        }
        _ = result as String
    }
}

// ผลลัพธ์ที่คาดหวัง:
// joined(separator:)  : 5.2 µs (fastest - built-in optimization)
// NSMutableString     : 12.1 µs
// reduce with +       : 245.3 µs (slowest - O(n²) string copying)
```

---

## 2. Memory Performance

### 2.1 Stack vs Heap Allocation Profiling

```swift
import Foundation

// Stack allocation - เร็วมาก (เพียงแค่เลื่อน stack pointer)
struct StackAllocated {
    var x: Double = 0
    var y: Double = 0
    var z: Double = 0
    // ถูก allocate บน stack ถ้าขนาดไม่ใหญ่เกิน
}

// Heap allocation - ช้ากว่า (ต้องหา free block, lock, bookkeeping)
class HeapAllocated {
    var x: Double = 0
    var y: Double = 0
    var z: Double = 0
    // ถูก allocate บน heap เสมอ
}

// Benchmark
func benchmarkStackVsHeap() {
    let iterations = 1_000_000
    let clock = ContinuousClock()
    
    // Stack
    let stackStart = clock.now
    for _ in 0..<iterations {
        var p = StackAllocated()
        p.x = 1.0; p.y = 2.0; p.z = 3.0
        _ = p.x + p.y + p.z
    }
    let stackTime = clock.now - stackStart
    
    // Heap
    let heapStart = clock.now
    for _ in 0..<iterations {
        let p = HeapAllocated()
        p.x = 1.0; p.y = 2.0; p.z = 3.0
        _ = p.x + p.y + p.z
    }
    let heapTime = clock.now - heapStart
    
    print("Stack: \(stackTime.formatted())")
    print("Heap: \(heapTime.formatted())")
    print("Heap overhead: \(String(format: "%.1f", heapTime.toSeconds() / stackTime.toSeconds()))x")
}

// Measuring allocation with malloc_size
func measureHeapAllocation() {
    let obj = HeapAllocated()
    let ptr = Unmanaged.passUnretained(obj).toOpaque()
    let allocSize = malloc_size(ptr)
    print("HeapAllocated size on heap: \(allocSize) bytes")
    // ปกติ > sizeof(HeapAllocated) เพราะมี ARC overhead, alignment padding
}
```

### 2.2 Heap Allocation Reduction Techniques

```swift
// Technique 1: ใช้ struct แทน class เมื่อทำได้
// Before: class (heap allocation per instance)
class PersonClass {
    let name: String
    let age: Int
    init(name: String, age: Int) { self.name = name; self.age = age }
}

// After: struct (stack allocation, copy semantics)
struct PersonStruct {
    let name: String
    let age: Int
}

// Technique 2: Value Buffer Optimization (Existential container)
// Protocol types สร้าง existential container บน heap ถ้า type ใหญ่กว่า 3 words

protocol Shape {
    func area() -> Double
}

// ❌ Large struct ใน existential - ต้อง heap allocate
struct LargeShape: Shape {
    var points: [CGPoint] = Array(repeating: .zero, count: 100)
    func area() -> Double { 0 }
}

// ✅ Small struct - fits in existential inline (no heap)
struct SmallCircle: Shape {
    let radius: Double
    func area() -> Double { .pi * radius * radius }
}

// Technique 3: withUnsafeTemporaryAllocation
func processBatchEfficiently(count: Int) {
    // Allocate บน stack สำหรับ temporary buffer
    withUnsafeTemporaryAllocation(of: Int.self, capacity: count) { buffer in
        for i in 0..<count {
            buffer[i] = i * i
        }
        let sum = buffer.reduce(0, +)
        print("Sum: \(sum)")
    }
    // buffer ถูก deallocate อัตโนมัติ - ไม่มี heap allocation
}

// Technique 4: ContiguousArray สำหรับ value types
// Array<SomeClass> เก็บ ARC-managed references
// ContiguousArray<SomeStruct> เก็บ values โดยตรง

func compareArrayTypes() {
    // Array ของ struct - ดีอยู่แล้ว
    var structArray: [PersonStruct] = []
    
    // ContiguousArray - เร็วกว่า Array เล็กน้อยสำหรับ bridged types
    var contiguousArray: ContiguousArray<PersonStruct> = []
    
    for i in 0..<1000 {
        structArray.append(PersonStruct(name: "Person \(i)", age: i))
        contiguousArray.append(PersonStruct(name: "Person \(i)", age: i))
    }
}
```

### 2.3 Small Struct/Enum Optimization

```swift
// Swift compiler มี optimization สำหรับ small structs

// Enum กับ raw values - ขนาดเล็กมาก
enum Color: UInt8 {  // เพียง 1 byte
    case red = 0, green = 1, blue = 2
}

// Optional enum - ยังคงเป็น 1 byte (special case)
let optColor: Color? = .red  // 1 byte เพราะ nil ใช้ bit pattern พิเศษ

// ตรวจสอบ memory layout
func checkSizes() {
    print("Bool: \(MemoryLayout<Bool>.size) bytes")
    print("Int: \(MemoryLayout<Int>.size) bytes")
    print("Color: \(MemoryLayout<Color>.size) bytes")
    print("Color?: \(MemoryLayout<Color?>.size) bytes")  // ยัง 1 byte!
    
    // Class reference - เป็น pointer เสมอ (8 bytes บน 64-bit)
    print("PersonClass ref: \(MemoryLayout<PersonClass>.size) bytes")
    
    // Struct - เท่ากับผลรวมของ fields (+ padding)
    print("PersonStruct: \(MemoryLayout<PersonStruct>.size) bytes")
}

// Copy-on-Write (COW) สำหรับ large structs
struct LargeBuffer {
    private var storage: Storage
    
    private class Storage {
        var data: [UInt8]
        init(data: [UInt8]) { self.data = data }
        
        func copy() -> Storage { Storage(data: data) }
    }
    
    init(size: Int) {
        self.storage = Storage(data: Array(repeating: 0, count: size))
    }
    
    var count: Int { storage.data.count }
    
    // COW: copy เฉพาะเมื่อมีการ mutate และมี reference อื่นอยู่
    subscript(index: Int) -> UInt8 {
        get { storage.data[index] }
        set {
            // isKnownUniquelyReferenced ตรวจสอบว่า reference นี้เป็น unique หรือไม่
            if !isKnownUniquelyReferenced(&storage) {
                storage = storage.copy()
            }
            storage.data[index] = newValue
        }
    }
}
```

### 2.4 Buffer Reuse Patterns

```swift
// Object Pool pattern - reuse expensive objects
final class ObjectPool<T: AnyObject> {
    private var available: [T] = []
    private var inUse: Set<ObjectIdentifier> = []
    private let lock = NSLock()
    private let factory: () -> T
    private let reset: (T) -> Void
    private let maxSize: Int
    
    init(maxSize: Int, factory: @escaping () -> T, reset: @escaping (T) -> Void = { _ in }) {
        self.maxSize = maxSize
        self.factory = factory
        self.reset = reset
        
        // Pre-warm pool
        for _ in 0..<min(maxSize / 4, 10) {
            available.append(factory())
        }
    }
    
    func acquire() -> T {
        lock.lock()
        defer { lock.unlock() }
        
        if let obj = available.popLast() {
            inUse.insert(ObjectIdentifier(obj))
            return obj
        }
        
        let obj = factory()
        inUse.insert(ObjectIdentifier(obj))
        return obj
    }
    
    func release(_ obj: T) {
        lock.lock()
        defer { lock.unlock() }
        
        inUse.remove(ObjectIdentifier(obj))
        
        if available.count < maxSize {
            reset(obj)
            available.append(obj)
        }
    }
}

// ตัวอย่าง: Reuse image rendering contexts
let rendererPool = ObjectPool<UIGraphicsImageRenderer>(
    maxSize: 10,
    factory: { UIGraphicsImageRenderer(size: CGSize(width: 100, height: 100)) }
)

// Buffer reuse สำหรับ network data
class NetworkBufferManager {
    private var buffers: [Data] = []
    private let lock = NSLock()
    private let bufferSize = 65536 // 64KB
    
    func acquireBuffer() -> Data {
        lock.lock()
        defer { lock.unlock() }
        return buffers.popLast() ?? Data(capacity: bufferSize)
    }
    
    func releaseBuffer(_ data: inout Data) {
        lock.lock()
        defer { lock.unlock() }
        data.removeAll(keepingCapacity: true) // reset ไม่ deallocate
        if buffers.count < 20 {
            buffers.append(data)
        }
    }
}
```

### 2.5 NSCache vs Custom LRU Cache

```swift
// NSCache - มี built-in LRU + memory pressure handling
class ImageCacheWithNSCache {
    private let cache = NSCache<NSString, UIImage>()
    
    init() {
        cache.countLimit = 100          // สูงสุด 100 images
        cache.totalCostLimit = 50 * 1024 * 1024  // 50MB
    }
    
    func set(_ image: UIImage, forKey key: String) {
        let cost = Int(image.size.width * image.size.height * 4) // RGBA
        cache.setObject(image, forKey: key as NSString, cost: cost)
    }
    
    func get(_ key: String) -> UIImage? {
        cache.object(forKey: key as NSString)
    }
}

// Custom LRU Cache - ควบคุมได้มากกว่า
class LRUCache<Key: Hashable, Value> {
    private class Node {
        var key: Key
        var value: Value
        var prev: Node?
        var next: Node?
        
        init(key: Key, value: Value) {
            self.key = key
            self.value = value
        }
    }
    
    private var cache: [Key: Node] = [:]
    private let head = Node(key: 0 as! Key, value: 0 as! Value)  // dummy
    private let tail = Node(key: 0 as! Key, value: 0 as! Value)  // dummy
    private let capacity: Int
    
    init(capacity: Int) {
        self.capacity = capacity
        head.next = tail
        tail.prev = head
    }
    
    func get(_ key: Key) -> Value? {
        guard let node = cache[key] else { return nil }
        moveToFront(node)
        return node.value
    }
    
    func set(_ key: Key, value: Value) {
        if let node = cache[key] {
            node.value = value
            moveToFront(node)
        } else {
            let node = Node(key: key, value: value)
            cache[key] = node
            addToFront(node)
            
            if cache.count > capacity {
                if let lru = removeLast() {
                    cache.removeValue(forKey: lru.key)
                }
            }
        }
    }
    
    private func moveToFront(_ node: Node) {
        removeNode(node)
        addToFront(node)
    }
    
    private func addToFront(_ node: Node) {
        node.next = head.next
        node.prev = head
        head.next?.prev = node
        head.next = node
    }
    
    private func removeNode(_ node: Node) {
        node.prev?.next = node.next
        node.next?.prev = node.prev
    }
    
    private func removeLast() -> Node? {
        guard let last = tail.prev, last !== head else { return nil }
        removeNode(last)
        return last
    }
}
```

---

## 3. CPU Performance

### 3.1 Algorithmic Complexity vs Constant Factors

```swift
// O(n log n) ที่มี constant factor ใหญ่อาจช้ากว่า O(n²) ที่ n เล็ก
// ต้องวัดจริงเสมอ!

func compareAlgorithms() {
    let smallData = Array(0..<100).shuffled()
    let largeData = Array(0..<100_000).shuffled()
    let clock = ContinuousClock()
    
    // Small data comparison
    print("=== Small Data (n=100) ===")
    
    var data1 = smallData
    let t1 = clock.now
    data1.sort()  // O(n log n) - Tim Sort
    print("Built-in sort: \((clock.now - t1).formatted())")
    
    var data2 = smallData
    let t2 = clock.now
    insertionSort(&data2)  // O(n²) but fast for small n
    print("Insertion sort: \((clock.now - t2).formatted())")
    
    // Large data
    print("\n=== Large Data (n=100,000) ===")
    
    var data3 = largeData
    let t3 = clock.now
    data3.sort()
    print("Built-in sort: \((clock.now - t3).formatted())")
    
    var data4 = largeData
    let t4 = clock.now
    insertionSort(&data4)
    print("Insertion sort: \((clock.now - t4).formatted())")
}

func insertionSort<T: Comparable>(_ array: inout [T]) {
    for i in 1..<array.count {
        let key = array[i]
        var j = i - 1
        while j >= 0 && array[j] > key {
            array[j + 1] = array[j]
            j -= 1
        }
        array[j + 1] = key
    }
}
```

### 3.2 Branch Prediction และ Code Layout

```swift
// Branch prediction: CPU เดา branch ล่วงหน้า
// Unpredictable branches ทำให้ pipeline stall

// ❌ Random branches ทำให้ prediction miss
func processRandomData(_ data: [Int]) -> Int {
    var sum = 0
    for value in data {
        if value > 128 {  // 50% chance - hard to predict
            sum += value
        }
    }
    return sum
}

// ✅ Branchless - ไม่มี branch = ไม่มี misprediction
func processDataBranchless(_ data: [Int]) -> Int {
    var sum = 0
    for value in data {
        // ไม่มี if - ใช้ arithmetic แทน
        let mask = (value - 129) >> 63  // -1 ถ้า value <= 128, 0 ถ้า > 128
        sum += value & ~mask  // value ถ้า > 128, 0 ถ้าไม่
    }
    return sum
}

// ✅ SIMD branchless (ดียิ่งกว่า)
import simd

func processDataSIMD(_ data: [Int32]) -> Int32 {
    var sum: Int32 = 0
    let threshold = SIMD8<Int32>(repeating: 128)
    
    // Process 8 elements at a time
    let vectorCount = data.count / 8
    data.withUnsafeBufferPointer { buffer in
        var ptr = buffer.baseAddress!
        for _ in 0..<vectorCount {
            let vec = SIMD8<Int32>(ptr[0], ptr[1], ptr[2], ptr[3],
                                    ptr[4], ptr[5], ptr[6], ptr[7])
            let mask = vec .> threshold  // vector comparison
            let masked = vec & SIMD8<Int32>(mask)  // zero where <= threshold
            sum += masked.wrappedSum()
            ptr += 8
        }
    }
    return sum
}

// Sorting สำหรับ better branch prediction
func processWithSort(_ data: [Int]) -> Int {
    // Sort ก่อน - branch prediction จะดีขึ้นมาก
    let sorted = data.sorted()
    var sum = 0
    for value in sorted {
        if value > 128 {
            sum += value
        }
    }
    return sum
}
```

### 3.3 SIMD กับ Swift's simd Module

```swift
import simd
import Accelerate

// SIMD basics: ทำงานกับ vectors ของ numbers พร้อมกัน
func simdBasics() {
    // Float vectors
    let v1 = SIMD4<Float>(1, 2, 3, 4)
    let v2 = SIMD4<Float>(5, 6, 7, 8)
    
    let sum = v1 + v2          // (6, 8, 10, 12) - 4 additions at once
    let product = v1 * v2      // (5, 12, 21, 32)
    let dotProduct = (v1 * v2).sum()  // 5+12+21+32 = 70
    
    print("Sum: \(sum)")
    print("Dot product: \(dotProduct)")
    
    // Distance calculation
    let a = SIMD3<Float>(1, 2, 3)
    let b = SIMD3<Float>(4, 6, 8)
    let diff = b - a
    let distance = sqrt((diff * diff).sum())
    print("Distance: \(distance)")
}

// Color processing กับ SIMD
struct ColorProcessor {
    // Process pixel array กับ SIMD
    static func applyBrightnessFilter(
        pixels: inout [UInt8],
        factor: Float,
        width: Int,
        height: Int
    ) {
        let pixelCount = width * height * 4 // RGBA
        let floatFactor = factor
        
        // Accelerate framework สำหรับ vectorized operations
        pixels.withUnsafeMutableBufferPointer { buffer in
            // แปลง UInt8 เป็น Float
            var floatPixels = [Float](repeating: 0, count: pixelCount)
            vDSP_vfltu8(buffer.baseAddress!, 1, &floatPixels, 1, vDSP_Length(pixelCount))
            
            // คูณด้วย factor (vectorized)
            vDSP_vsmul(floatPixels, 1, [floatFactor], &floatPixels, 1, vDSP_Length(pixelCount))
            
            // Clamp ให้อยู่ใน 0-255
            var minVal: Float = 0, maxVal: Float = 255
            vDSP_vclip(floatPixels, 1, &minVal, &maxVal, &floatPixels, 1, vDSP_Length(pixelCount))
            
            // แปลงกลับเป็น UInt8
            vDSP_vfixu8(floatPixels, 1, buffer.baseAddress!, 1, vDSP_Length(pixelCount))
        }
    }
    
    // Matrix operations กับ simd
    static func transformPoints(_ points: [SIMD3<Float>], matrix: float4x4) -> [SIMD3<Float>] {
        return points.map { point in
            let homogeneous = SIMD4<Float>(point.x, point.y, point.z, 1.0)
            let transformed = matrix * homogeneous
            return SIMD3<Float>(transformed.x, transformed.y, transformed.z) / transformed.w
        }
    }
}
```

### 3.4 withUnsafe Buffers สำหรับ Critical Paths

```swift
// withUnsafeBufferPointer - ไม่มี bounds checking = เร็วกว่า
// ใช้เฉพาะใน hot paths ที่ verified แล้ว

func fastSumWithUnsafe(_ array: [Int]) -> Int {
    var sum = 0
    array.withUnsafeBufferPointer { buffer in
        // bounds check ถูก disable บน release builds กับ UnsafeBufferPointer
        let ptr = buffer.baseAddress!
        let count = buffer.count
        
        for i in 0..<count {
            sum &+= ptr[i]  // &+ เพื่อ overflow wrapping (ไม่มี overflow check)
        }
    }
    return sum
}

// Unsafe string operations
func fastStringSearch(_ haystack: String, _ needle: String) -> Bool {
    return haystack.withCString { haystackPtr in
        needle.withCString { needlePtr in
            strstr(haystackPtr, needlePtr) != nil
        }
    }
}

// Memory copying กับ UnsafeRawBufferPointer
func fastMemoryCopy(from source: [UInt8], to destination: inout [UInt8]) {
    precondition(destination.count >= source.count)
    
    source.withUnsafeBytes { srcBytes in
        destination.withUnsafeMutableBytes { dstBytes in
            dstBytes.copyMemory(from: srcBytes)
        }
    }
}

// Struct serialization ด้วย unsafe pointers
func serialize<T>(_ value: T) -> Data {
    var copy = value
    return withUnsafeBytes(of: &copy) { Data($0) }
}

func deserialize<T>(_ data: Data, as type: T.Type) -> T? {
    guard data.count == MemoryLayout<T>.size else { return nil }
    return data.withUnsafeBytes { $0.load(as: T.self) }
}
```

---

## 4. Swift Compiler Optimizations

### 4.1 Whole Module Optimization (WMO)

```
Build Settings:
- Swift Compilation Mode: Whole Module (Release)
- Optimization Level: Optimize for Speed [-O]

WMO ทำให้ compiler:
1. Inline functions ข้าม files
2. Specialize generic functions
3. Devirtualize protocol/class method calls
4. Dead code elimination
5. Global analysis และ optimization
```

```swift
// ตัวอย่าง optimization ที่ WMO ทำได้

// File A.swift
public func computeValue(_ x: Int) -> Int {
    return heavyTransform(x)
}

private func heavyTransform(_ x: Int) -> Int {
    return x * x + x + 1
}

// File B.swift - กับ WMO, compiler เห็น both files
// และสามารถ inline heavyTransform ได้
func processArray(_ values: [Int]) -> [Int] {
    return values.map { computeValue($0) }
    // WMO: inlines computeValue และ heavyTransform
    // = values.map { $0 * $0 + $0 + 1 }
}
```

### 4.2 @_optimize Attribute

```swift
// @_optimize(none) - disable optimization สำหรับ debugging
@_optimize(none)
func debugFunction(_ x: Int) -> Int {
    let result = x * 42  // ง่ายต่อการ breakpoint
    return result
}

// @_optimize(speed) - optimize สำหรับ speed (เทียบเท่า -O)
@_optimize(speed)
func criticalFunction(_ data: [Double]) -> Double {
    return data.reduce(0, +)
}

// @_optimize(size) - optimize สำหรับ binary size
@_optimize(size)
func rarePath(_ config: [String: Any]) -> String {
    // ... complex logic ที่ไม่ต้องการ speed
    return ""
}

// Note: @_optimize เป็น underscored = unofficial
// ใช้ด้วยความระมัดระวัง
```

### 4.3 @inline Attributes

```swift
// @inline(__always) - force inline ทุกครั้ง
// เหมาะกับ hot path functions ที่เล็กมาก
@inline(__always)
func fastAdd(_ a: Int, _ b: Int) -> Int {
    return a &+ b
}

// @inline(never) - ห้าม inline
// เหมาะกับ error paths, rarely called code
@inline(never)
func handleCriticalError(_ error: Error) {
    // code นี้จะไม่ถูก inline
    // ทำให้ main path compact และ cache-friendly
    print("Critical error: \(error)")
    // ... logging, reporting
}

// ตัวอย่างการใช้ใน collection operations
extension Array where Element: Numeric {
    @inline(__always)
    func sum() -> Element {
        // Force inline เพื่อ maximize optimization opportunity
        reduce(.zero, +)
    }
    
    @inline(__always)
    var isEmpty_fast: Bool {
        count == 0  // Inline เพื่อให้ compiler optimize เป็น single comparison
    }
}

// @_transparent - เหนือกว่า @inline(__always)
// ทำให้ function "transparent" - เหมือน body ถูก copy มา
// ใช้สำหรับ operator definitions
@_transparent
func vectorLength(_ v: SIMD3<Float>) -> Float {
    sqrt((v * v).sum())
}
```

### 4.4 Specialization ของ Generic Code

```swift
// Generic functions อาจช้ากว่า non-generic เพราะ type erasure
// Specialization แก้ปัญหานี้

// Generic function
func genericMax<T: Comparable>(_ a: T, _ b: T) -> T {
    return a > b ? a : b
}

// Compiler specializes: genericMax<Int>, genericMax<Double>, etc.
// แต่ละ specialization เหมือน non-generic function

// @_specialize attribute - force specialization
extension Array {
    @_specialize(where Element == Int)
    @_specialize(where Element == Double)
    @_specialize(where Element == String)
    func customSort() -> [Element] where Element: Comparable {
        return sorted()
    }
}

// Protocol witness table vs direct dispatch
protocol Processable {
    func process() -> Int
}

struct FastProcessor: Processable {
    var value: Int
    
    func process() -> Int { value * 2 }
}

// ❌ Protocol type - dynamic dispatch via witness table
func processViaProtocol(_ items: [any Processable]) -> [Int] {
    items.map { $0.process() }
}

// ✅ Generic - compiler can specialize and devirtualize
func processGeneric<T: Processable>(_ items: [T]) -> [Int] {
    items.map { $0.process() }
}

// ✅ ✅ Concrete type - best performance
func processConcrete(_ items: [FastProcessor]) -> [Int] {
    items.map { $0.process() }
}
```

---

## 5. SwiftUI Performance Deep Dive

### 5.1 View Identity และ State

```swift
import SwiftUI

// View identity: SwiftUI ใช้ identity เพื่อตัดสินใจว่า view เดิมหรือใหม่

// Structural identity - ตำแหน่งใน view tree
struct StructuralIdentityExample: View {
    var showDetail: Bool
    
    var body: some View {
        VStack {
            // View เดิมตลอด - ใช้ transition แทน ถ้าต้องการ animate
            if showDetail {
                DetailView()  // identity ผูกกับ if branch
            } else {
                SummaryView()  // คนละ identity กับ DetailView!
            }
        }
    }
}

// Explicit identity - ด้วย .id() modifier
struct ExplicitIdentityExample: View {
    var userId: String
    
    var body: some View {
        UserProfileView(userId: userId)
            .id(userId)  // Force recreate เมื่อ userId เปลี่ยน
    }
}

// ⚠️ อย่าใช้ .id() กับ random values
struct BadIdentityExample: View {
    var body: some View {
        ForEach(0..<5) { i in
            Text("Item \(i)")
                .id(UUID())  // ❌ recreate ทุกครั้ง - very bad!
        }
    }
}
```

### 5.2 Equatable Conformance สำหรับ Performance

```swift
// SwiftUI skip re-render ถ้า view เป็น Equatable และ value ไม่เปลี่ยน
struct UserCard: View, Equatable {
    let user: User
    let isSelected: Bool
    
    // Custom equality - เฉพาะ fields ที่ affect rendering
    static func == (lhs: UserCard, rhs: UserCard) -> Bool {
        lhs.user.id == rhs.user.id &&
        lhs.user.name == rhs.user.name &&
        lhs.isSelected == rhs.isSelected
        // ไม่เช็ค user.lastLoginDate ถ้ามันไม่ affect UI
    }
    
    var body: some View {
        HStack {
            Circle()
                .fill(isSelected ? Color.blue : Color.gray)
                .frame(width: 40, height: 40)
            Text(user.name)
                .font(.headline)
        }
    }
}

// EquatableView wrapper สำหรับ existing views
struct ExpensiveView: View {
    let config: ViewConfig
    
    var body: some View {
        // expensive rendering
        Color.blue
    }
}

struct ViewConfig: Equatable {
    let title: String
    let color: Color
    let fontSize: CGFloat
}

// ใช้ .equatable() modifier
struct ParentView: View {
    @State var config = ViewConfig(title: "Hello", color: .blue, fontSize: 17)
    @State var counter = 0  // Changes ที่ไม่ affect ExpensiveView
    
    var body: some View {
        VStack {
            ExpensiveView(config: config)
                .equatable()  // Skip re-render ถ้า config ไม่เปลี่ยน
            
            Text("Counter: \(counter)")
            
            Button("Increment") { counter += 1 }
            // ExpensiveView จะไม่ re-render เมื่อกด!
        }
    }
}
```

### 5.3 @State Granularity

```swift
// ❌ State เดียวใหญ่เกินไป - ทำให้ re-render ทั้งหมดเมื่อเปลี่ยน field ใดก็ตาม
struct BigStateView: View {
    @State var viewState = AppViewState(
        users: [],
        selectedUser: nil,
        isLoading: false,
        searchText: "",
        sortOrder: .ascending,
        filterCategory: "all"
    )
    
    var body: some View {
        // ทุก field เปลี่ยน = ทุก subview re-render
        UserList(users: viewState.users,
                 isLoading: viewState.isLoading)
    }
}

// ✅ แยก State เพื่อ minimize re-renders
struct GranularStateView: View {
    @State var users: [User] = []
    @State var selectedUserId: String? = nil
    @State var isLoading = false
    @State var searchText = ""
    
    var body: some View {
        // isLoading เปลี่ยน = แค่ LoadingView re-render
        // searchText เปลี่ยน = แค่ SearchBar re-render
        VStack {
            SearchBar(text: $searchText)
            UserList(users: filteredUsers, selectedId: $selectedUserId)
            if isLoading { LoadingView() }
        }
    }
    
    var filteredUsers: [User] {
        guard !searchText.isEmpty else { return users }
        return users.filter { $0.name.localizedCaseInsensitiveContains(searchText) }
    }
}

struct AppViewState {
    var users: [User]
    var selectedUser: User?
    var isLoading: Bool
    var searchText: String
    var sortOrder: SortOrder
    var filterCategory: String
}

enum SortOrder { case ascending, descending }
struct User { let id: String; let name: String }
```

### 5.4 LazyVStack vs VStack Profiling

```swift
// LazyVStack: สร้าง views เฉพาะเมื่อ visible
// VStack: สร้างทุก views ทันที

struct ListComparisonView: View {
    let items = Array(0..<10_000).map { "Item \($0)" }
    
    var body: some View {
        ScrollView {
            // ✅ LazyVStack - เหมาะกับ large lists
            LazyVStack(spacing: 0, pinnedViews: .sectionHeaders) {
                Section {
                    ForEach(items, id: \.self) { item in
                        ExpensiveRow(text: item)
                    }
                } header: {
                    Text("Section Header")
                        .padding()
                        .background(Color(.systemBackground))
                }
            }
        }
    }
}

// ❌ VStack กับ 10,000 items - สร้างทั้งหมดพร้อมกัน
struct NaiveListView: View {
    let items = Array(0..<10_000).map { "Item \($0)" }
    
    var body: some View {
        ScrollView {
            VStack {  // สร้าง 10,000 views ทันที - ช้ามาก!
                ForEach(items, id: \.self) { item in
                    Text(item)
                }
            }
        }
    }
}

// ExpensiveRow - simulate complex cell
struct ExpensiveRow: View {
    let text: String
    
    var body: some View {
        HStack {
            Circle().fill(Color.blue).frame(width: 40, height: 40)
            VStack(alignment: .leading) {
                Text(text).font(.headline)
                Text("Subtitle for \(text)").font(.caption).foregroundColor(.secondary)
            }
            Spacer()
            Image(systemName: "chevron.right").foregroundColor(.secondary)
        }
        .padding(.horizontal)
        .padding(.vertical, 8)
    }
}
```

### 5.5 List vs LazyVStack Comparison

```swift
// List: ใช้ UITableView underneath - มี built-in optimizations
// LazyVStack ใน ScrollView: pure SwiftUI - flexible แต่ overhead มากกว่า

struct ListPerformanceComparison: View {
    let items = Array(0..<1000).map { ItemModel(id: $0, title: "Item \($0)") }
    @State var useList = true
    
    var body: some View {
        Group {
            if useList {
                // List - เร็วกว่าสำหรับ simple cells
                List(items) { item in
                    ItemRow(item: item)
                }
            } else {
                // LazyVStack - flexible มากกว่า แต่ overhead เพิ่มขึ้น
                ScrollView {
                    LazyVStack(spacing: 0) {
                        ForEach(items) { item in
                            ItemRow(item: item)
                            Divider()
                        }
                    }
                }
            }
        }
        .toolbar {
            Toggle("Use List", isOn: $useList)
        }
    }
}

struct ItemModel: Identifiable {
    let id: Int
    let title: String
}

struct ItemRow: View {
    let item: ItemModel
    
    var body: some View {
        Text(item.title)
            .padding(.vertical, 8)
    }
}

// เมื่อไหร่ควรใช้อะไร:
// List: static data, simple cells, ต้องการ swipe actions/editing
// LazyVStack: custom layout, sticky headers ที่ซับซ้อน, mixed content
// VStack: < 50 items หรือ items ที่ต้องการ size ทั้งหมด
```

---

## 6. Core Data Performance

### 6.1 NSFetchRequest Tuning

```swift
import CoreData

// Fetch request optimization
func optimizedFetch(context: NSManagedObjectContext) throws -> [NSManagedObject] {
    let request = NSFetchRequest<NSManagedObject>(entityName: "Article")
    
    // 1. Fetch batch - หยิบ objects มา N ตัวพร้อมกัน (ดีกว่า all at once)
    request.fetchBatchSize = 20  // 20 objects ต่อ batch
    
    // 2. Fetch limit - จำกัดจำนวนผลลัพธ์
    request.fetchLimit = 100
    
    // 3. Fetch offset - สำหรับ pagination
    request.fetchOffset = 0
    
    // 4. ดึงแค่ properties ที่ต้องการ
    request.propertiesToFetch = ["title", "publishDate", "isRead"]
    request.resultType = .dictionaryResultType  // Dictionary แทน managed object
    
    // 5. ไม่ return property faults
    request.returnsObjectsAsFaults = false  // pre-fill properties
    
    // 6. Include subentities
    request.includesSubentities = false  // ถ้าไม่ต้องการ
    
    // 7. Predicate indexing
    request.predicate = NSPredicate(format: "isRead == NO AND publishDate > %@", 
                                    NSDate(timeIntervalSinceNow: -7*24*3600))
    
    // 8. Sort descriptor
    request.sortDescriptors = [
        NSSortDescriptor(key: "publishDate", ascending: false)
    ]
    
    return try context.fetch(request)
}

// Count query - เร็วกว่า fetch ทั้งหมด
func countArticles(context: NSManagedObjectContext, isRead: Bool) throws -> Int {
    let request = NSFetchRequest<NSManagedObject>(entityName: "Article")
    request.predicate = NSPredicate(format: "isRead == %@", NSNumber(value: isRead))
    return try context.count(for: request)
}
```

### 6.2 Faulting และ Prefetching

```swift
// NSManagedObject Faults - lightweight placeholder ที่โหลด data เมื่อ access
// Default behavior ช่วยประหยัด memory แต่อาจเกิด N+1 query problem

// ❌ N+1 Problem
func fetchArticlesWithAuthors_BAD(context: NSManagedObjectContext) throws {
    let articles = try context.fetch(NSFetchRequest<Article>(entityName: "Article"))
    
    for article in articles {
        // แต่ละ article access author = 1 fault = 1 query
        // 100 articles = 100 extra queries!
        print(article.author?.name ?? "Unknown")
    }
}

// ✅ Prefetch relationships
func fetchArticlesWithAuthors_GOOD(context: NSManagedObjectContext) throws {
    let request = NSFetchRequest<Article>(entityName: "Article")
    
    // Prefetch authors ใน query เดียว
    request.relationshipKeyPathsForPrefetching = ["author"]
    
    // Prefetch nested relationships
    // request.relationshipKeyPathsForPrefetching = ["author", "author.profile"]
    
    let articles = try context.fetch(request)
    
    for article in articles {
        // author ถูก prefetch แล้ว - ไม่มี extra query
        print(article.author?.name ?? "Unknown")
    }
}

class Article: NSManagedObject {
    @NSManaged var title: String?
    @NSManaged var publishDate: Date?
    @NSManaged var isRead: Bool
    @NSManaged var author: Author?
}

class Author: NSManagedObject {
    @NSManaged var name: String?
}
```

### 6.3 Compound Predicates Optimization

```swift
// Index ใน Core Data model ช่วย query performance
// ต้องสร้าง index ใน .xcdatamodeld สำหรับ fields ที่ query บ่อย

// ✅ ใช้ indexed fields ใน predicate
let efficientPredicate = NSPredicate(
    format: "publishDate > %@ AND isRead == NO",  // ทั้งคู่ควร indexed
    NSDate(timeIntervalSinceNow: -86400)
)

// ❌ เปรียบเทียบ string โดยไม่มี index - full scan
let inefficientPredicate = NSPredicate(
    format: "title CONTAINS[cd] %@",  // CONTAINS ไม่ใช้ index
    "swift"
)

// ✅ ใช้ BEGINSWITH แทน CONTAINS ถ้าทำได้ (ใช้ index ได้)
let betterStringPredicate = NSPredicate(
    format: "title BEGINSWITH[cd] %@",
    "swift"
)

// Compound predicate optimization
func buildEfficientPredicate(
    minDate: Date,
    categories: [String],
    isRead: Bool
) -> NSPredicate {
    var subpredicates: [NSPredicate] = []
    
    // 1. Most selective predicate ก่อน (ลด result set ได้มากที่สุด)
    subpredicates.append(NSPredicate(format: "publishDate > %@", minDate as NSDate))
    
    // 2. Boolean check - เร็วมาก
    subpredicates.append(NSPredicate(format: "isRead == %@", NSNumber(value: isRead)))
    
    // 3. IN predicate - ใช้ index ถ้ามี
    if !categories.isEmpty {
        subpredicates.append(NSPredicate(format: "category IN %@", categories))
    }
    
    return NSCompoundPredicate(andPredicateWithSubpredicates: subpredicates)
}
```

---

## 7. Network Performance

### 7.1 HTTP/2 Multiplexing

```swift
import Foundation

// URLSession รองรับ HTTP/2 โดยอัตโนมัติ
// HTTP/2 ให้ประโยชน์:
// - Multiplexing: หลาย requests ผ่าน connection เดียว
// - Header compression (HPACK)
// - Server push
// - Binary framing

// ตั้งค่า URLSession สำหรับ performance
func createOptimizedSession() -> URLSession {
    let config = URLSessionConfiguration.default
    
    // Connection pooling
    config.httpMaximumConnectionsPerHost = 6  // HTTP/1.1 default
    // HTTP/2: 1 connection เพียงพอ เพราะ multiplexing
    
    // Timeout settings
    config.timeoutIntervalForRequest = 30  // per-request timeout
    config.timeoutIntervalForResource = 300  // total resource timeout
    
    // Cache policy
    config.requestCachePolicy = .returnCacheDataElseLoad
    config.urlCache = URLCache(
        memoryCapacity: 10 * 1024 * 1024,  // 10MB memory
        diskCapacity: 100 * 1024 * 1024    // 100MB disk
    )
    
    // HTTP pipelining (HTTP/1.1 only - ไม่จำเป็นกับ HTTP/2)
    config.httpShouldUsePipelining = true
    
    return URLSession(configuration: config)
}

// ส่ง multiple requests พร้อมกัน กับ HTTP/2
func parallelRequests(session: URLSession) async throws -> [Data] {
    let urls = [
        URL(string: "https://api.example.com/users")!,
        URL(string: "https://api.example.com/posts")!,
        URL(string: "https://api.example.com/comments")!,
    ]
    
    return try await withThrowingTaskGroup(of: Data.self) { group in
        for url in urls {
            group.addTask {
                let (data, _) = try await session.data(from: url)
                return data
            }
        }
        
        var results: [Data] = []
        for try await data in group {
            results.append(data)
        }
        return results
    }
}
```

### 7.2 Request Deduplication

```swift
// ป้องกัน duplicate requests สำหรับ resource เดียวกัน
actor RequestDeduplicator<Response: Sendable> {
    private var inFlight: [URL: Task<Response, Error>] = [:]
    
    func fetch(
        url: URL,
        fetcher: @Sendable (URL) async throws -> Response
    ) async throws -> Response {
        // ถ้ามี request นี้อยู่แล้ว รอ result เดิม
        if let existingTask = inFlight[url] {
            return try await existingTask.value
        }
        
        // สร้าง task ใหม่
        let task = Task<Response, Error> {
            defer { Task { await self.complete(url: url) } }
            return try await fetcher(url)
        }
        
        inFlight[url] = task
        return try await task.value
    }
    
    private func complete(url: URL) {
        inFlight.removeValue(forKey: url)
    }
}

// ใช้งาน
class ImageLoader: ObservableObject {
    private let deduplicator = RequestDeduplicator<UIImage>()
    private let session = URLSession.shared
    
    func loadImage(url: URL) async throws -> UIImage {
        return try await deduplicator.fetch(url: url) { [session] url in
            let (data, _) = try await session.data(from: url)
            guard let image = UIImage(data: data) else {
                throw URLError(.cannotDecodeContentData)
            }
            return image
        }
    }
}
```

### 7.3 Binary Protocols (Protobuf vs JSON)

```swift
// JSON parsing - สะดวกแต่ช้ากว่า binary
struct JSONBenchmark {
    struct UserJSON: Codable {
        let id: Int
        let name: String
        let email: String
        let age: Int
    }
    
    static func benchmark() {
        let jsonString = """
        {"id":1,"name":"Alice","email":"alice@example.com","age":30}
        """
        let jsonData = jsonString.data(using: .utf8)!
        
        let clock = ContinuousClock()
        let iterations = 100_000
        
        // JSON decoding
        let jsonStart = clock.now
        for _ in 0..<iterations {
            _ = try? JSONDecoder().decode(UserJSON.self, from: jsonData)
        }
        print("JSON decode: \((clock.now - jsonStart).formatted())")
        
        // JSON encoding
        let user = UserJSON(id: 1, name: "Alice", email: "alice@example.com", age: 30)
        let jsonEncStart = clock.now
        for _ in 0..<iterations {
            _ = try? JSONEncoder().encode(user)
        }
        print("JSON encode: \((clock.now - jsonEncStart).formatted())")
    }
}

// MessagePack - binary alternative ที่ compact กว่า JSON
// ต้องใช้ library เช่น MessagePack.swift
// ข้อดี: เล็กกว่า ~30%, parse เร็วกว่า ~2-3x

// Custom binary format สำหรับ critical paths
struct BinaryRecord {
    var id: UInt32
    var timestamp: Double
    var value: Float
    var flags: UInt8
    
    var binaryRepresentation: Data {
        var data = Data(capacity: 17) // 4+8+4+1 bytes
        withUnsafeBytes(of: id.littleEndian) { data.append(contentsOf: $0) }
        withUnsafeBytes(of: timestamp) { data.append(contentsOf: $0) }
        withUnsafeBytes(of: value) { data.append(contentsOf: $0) }
        data.append(flags)
        return data
    }
    
    static func from(data: Data) -> BinaryRecord? {
        guard data.count == 17 else { return nil }
        return data.withUnsafeBytes { ptr in
            BinaryRecord(
                id: UInt32(littleEndian: ptr.load(fromByteOffset: 0, as: UInt32.self)),
                timestamp: ptr.load(fromByteOffset: 4, as: Double.self),
                value: ptr.load(fromByteOffset: 12, as: Float.self),
                flags: ptr.load(fromByteOffset: 16, as: UInt8.self)
            )
        }
    }
}
```

---

## 8. Image Performance

### 8.1 Downsampling vs Resizing

```swift
import UIKit
import ImageIO

// ❌ Naive resizing - decode full resolution แล้วค่อย resize
// Memory spike: 4000x3000 image = 46MB+ ก่อน resize
func naiveResize(imageData: Data, targetSize: CGSize) -> UIImage? {
    guard let image = UIImage(data: imageData) else { return nil }
    
    let renderer = UIGraphicsImageRenderer(size: targetSize)
    return renderer.image { _ in
        image.draw(in: CGRect(origin: .zero, size: targetSize))
    }
}

// ✅ Downsampling - decode เฉพาะขนาดที่ต้องการ
// Memory: เฉพาะ target size = ประหยัดมาก
func downsample(imageData: Data, to pointSize: CGSize, scale: CGFloat = UIScreen.main.scale) -> UIImage? {
    let imageSourceOptions = [kCGImageSourceShouldCache: false] as CFDictionary
    guard let imageSource = CGImageSourceCreateWithData(imageData as CFData, imageSourceOptions) else {
        return nil
    }
    
    let maxDimensionInPixels = max(pointSize.width, pointSize.height) * scale
    let downsampleOptions = [
        kCGImageSourceCreateThumbnailFromImageAlways: true,
        kCGImageSourceShouldCacheImmediately: true,
        kCGImageSourceCreateThumbnailWithTransform: true,
        kCGImageSourceThumbnailMaxPixelSize: maxDimensionInPixels
    ] as CFDictionary
    
    guard let downsampledImage = CGImageSourceCreateThumbnailAtIndex(imageSource, 0, downsampleOptions) else {
        return nil
    }
    
    return UIImage(cgImage: downsampledImage)
}

// Downsampling จาก URL
func downsampleFromURL(_ url: URL, to size: CGSize) -> UIImage? {
    let imageSourceOptions = [kCGImageSourceShouldCache: false] as CFDictionary
    guard let imageSource = CGImageSourceCreateWithURL(url as CFURL, imageSourceOptions) else {
        return nil
    }
    
    let maxPixels = max(size.width, size.height) * UIScreen.main.scale
    let options = [
        kCGImageSourceCreateThumbnailFromImageAlways: true,
        kCGImageSourceShouldCacheImmediately: true,
        kCGImageSourceCreateThumbnailWithTransform: true,
        kCGImageSourceThumbnailMaxPixelSize: maxPixels
    ] as CFDictionary
    
    guard let cgImage = CGImageSourceCreateThumbnailAtIndex(imageSource, 0, options) else {
        return nil
    }
    
    return UIImage(cgImage: cgImage)
}
```

### 8.2 UIGraphicsImageRenderer

```swift
// UIGraphicsImageRenderer - modern, efficient image rendering
// รองรับ wide color, PDFs, และมี better memory management

class ImageCompositor {
    // Compose หลาย images เป็นหนึ่ง
    static func compose(images: [UIImage], size: CGSize) -> UIImage {
        let renderer = UIGraphicsImageRenderer(size: size)
        
        return renderer.image { context in
            let cgContext = context.cgContext
            
            // Draw background
            UIColor.white.setFill()
            context.fill(CGRect(origin: .zero, size: size))
            
            // Draw images in grid
            let cols = Int(ceil(sqrt(Double(images.count))))
            let rows = Int(ceil(Double(images.count) / Double(cols)))
            let cellWidth = size.width / CGFloat(cols)
            let cellHeight = size.height / CGFloat(rows)
            
            for (index, image) in images.enumerated() {
                let col = index % cols
                let row = index / cols
                let rect = CGRect(
                    x: CGFloat(col) * cellWidth,
                    y: CGFloat(row) * cellHeight,
                    width: cellWidth,
                    height: cellHeight
                )
                image.draw(in: rect)
            }
        }
    }
    
    // สร้าง PDF
    static func createPDF(from images: [UIImage], pageSize: CGSize) -> Data {
        let renderer = UIGraphicsPDFRenderer(bounds: CGRect(origin: .zero, size: pageSize))
        
        return renderer.pdfData { context in
            for image in images {
                context.beginPage()
                image.draw(in: CGRect(origin: .zero, size: pageSize))
            }
        }
    }
    
    // Watermark image
    static func addWatermark(to image: UIImage, text: String) -> UIImage {
        let renderer = UIGraphicsImageRenderer(size: image.size)
        
        return renderer.image { context in
            // Draw original
            image.draw(at: .zero)
            
            // Draw watermark
            let attributes: [NSAttributedString.Key: Any] = [
                .font: UIFont.systemFont(ofSize: image.size.width / 10),
                .foregroundColor: UIColor.white.withAlphaComponent(0.5)
            ]
            
            let attributedText = NSAttributedString(string: text, attributes: attributes)
            let textSize = attributedText.size()
            let textRect = CGRect(
                x: (image.size.width - textSize.width) / 2,
                y: (image.size.height - textSize.height) / 2,
                width: textSize.width,
                height: textSize.height
            )
            
            attributedText.draw(in: textRect)
        }
    }
}
```

### 8.3 Image Pipeline Profiling

```swift
// Image pipeline: Load → Decode → Process → Cache → Display
// แต่ละขั้นตอนมี cost ต่างกัน

class InstrumentedImagePipeline {
    private let cache = NSCache<NSURL, UIImage>()
    private let session = URLSession.shared
    
    struct PipelineMetrics {
        var fetchTime: Duration = .zero
        var decodeTime: Duration = .zero
        var processTime: Duration = .zero
        var cacheHit: Bool = false
    }
    
    func loadImage(url: URL, targetSize: CGSize) async throws -> (UIImage, PipelineMetrics) {
        var metrics = PipelineMetrics()
        let clock = ContinuousClock()
        
        // Check cache
        if let cached = cache.object(forKey: url as NSURL) {
            metrics.cacheHit = true
            return (cached, metrics)
        }
        
        // Fetch
        let fetchStart = clock.now
        let (data, _) = try await session.data(from: url)
        metrics.fetchTime = clock.now - fetchStart
        
        // Decode with downsampling
        let decodeStart = clock.now
        guard let image = downsample(imageData: data, to: targetSize) else {
            throw URLError(.cannotDecodeContentData)
        }
        metrics.decodeTime = clock.now - decodeStart
        
        // Process (e.g., apply filters)
        let processStart = clock.now
        let processed = await Task.detached(priority: .userInitiated) {
            self.applyFilters(to: image)
        }.value
        metrics.processTime = clock.now - processStart
        
        // Cache
        let cost = Int(processed.size.width * processed.size.height * 4)
        cache.setObject(processed, forKey: url as NSURL, cost: cost)
        
        return (processed, metrics)
    }
    
    private func applyFilters(to image: UIImage) -> UIImage {
        // ตัวอย่าง: apply slight sharpening
        guard let cgImage = image.cgImage else { return image }
        
        let filter = CIFilter(name: "CISharpenLuminance")!
        filter.setValue(CIImage(cgImage: cgImage), forKey: kCIInputImageKey)
        filter.setValue(0.3, forKey: kCIInputSharpnessKey)
        
        guard let outputImage = filter.outputImage,
              let resultCGImage = CIContext().createCGImage(outputImage, from: outputImage.extent) else {
            return image
        }
        
        return UIImage(cgImage: resultCGImage)
    }
}
```

---

## 9. Launch Time Optimization

### 9.1 Static vs Dynamic Initialization

```swift
// Static initialization - happens before main()
// ❌ ช้า: complex static initialization
class ExpensiveStaticInit {
    static let shared = ExpensiveStaticInit()  // init ตอน first access
    
    private init() {
        // Expensive setup
        Thread.sleep(forTimeInterval: 0.1)  // ❌ blocking!
    }
}

// ✅ Lazy initialization
class LazyInit {
    static let shared = LazyInit()
    
    private var isSetup = false
    
    func ensureSetup() async {
        guard !isSetup else { return }
        isSetup = true
        await performExpensiveSetup()
    }
    
    private func performExpensiveSetup() async {
        try? await Task.sleep(for: .milliseconds(100))
    }
}

// ✅ Lazy stored property
class AppConfiguration {
    lazy var expensiveData: [String: Any] = {
        // คำนวณเมื่อ access ครั้งแรกเท่านั้น
        return loadConfigFromDisk()
    }()
    
    private func loadConfigFromDisk() -> [String: Any] {
        // load from bundle or disk
        return [:]
    }
}

// ตรวจสอบ launch time
// ใน main.swift หรือ AppDelegate:
// DYLD_PRINT_STATISTICS=1 ใน scheme environment variables
// แสดงเวลาของ:
// - Total pre-main: เวลาทั้งหมดก่อน main()
// - dylib loading: โหลด dynamic libraries
// - rebase/binding: fix up addresses
// - ObjC setup: register ObjC classes
// - initializer time: static initializers
```

### 9.2 Pre-main Timing

```swift
// วัด launch time ด้วย os_signpost
import os

let launchLog = OSLog(subsystem: "com.myapp", category: .pointsOfInterest)

@main
struct MyApp: App {
    init() {
        os_signpost(.begin, log: launchLog, name: "App Init")
        // App initialization
        os_signpost(.end, log: launchLog, name: "App Init")
    }
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .onAppear {
                    os_signpost(.event, log: launchLog, name: "First Frame")
                }
        }
    }
}

// เทคนิค: วัด cold vs warm launch
class LaunchTimeMonitor {
    static let shared = LaunchTimeMonitor()
    private var launchDate: Date?
    
    func recordLaunchStart() {
        launchDate = Date()
    }
    
    func recordFirstMeaningfulPaint() {
        guard let start = launchDate else { return }
        let elapsed = Date().timeIntervalSince(start) * 1000
        print("Launch to First Meaningful Paint: \(String(format: "%.0f", elapsed))ms")
        
        // Apple guidelines:
        // Cold launch: < 400ms (warm-up) + < 20s total before watchdog kills
        // Warm launch: < 400ms
    }
}
```

### 9.3 dyld Closures

```swift
// dyld closure: pre-computed launch data ที่ OS cache ไว้
// สร้างได้ง่ายขึ้นเมื่อ:
// 1. ใช้ static linking แทน dynamic (ลด dylibs)
// 2. ลด ObjC classes
// 3. ลด category usage
// 4. ใช้ Swift structs/enums แทน ObjC classes

// ตรวจสอบ dynamic library usage
// ใน terminal: otool -L MyApp.app/MyApp

// Reduce dynamic framework count
// ❌ หลาย frameworks
// import AnalyticsFramework     // separate dylib
// import NetworkFramework       // separate dylib
// import DatabaseFramework      // separate dylib

// ✅ รวมเป็น single framework หรือ static library
// ใน Package.swift: type: .static
// .library(name: "CoreLibrary", type: .static, targets: ["CoreLibrary"])
```

### 9.4 Startup Task Prioritization

```swift
// แบ่ง startup tasks เป็น:
// 1. Critical path (ต้องทำก่อน first render)
// 2. Near-critical (ทำระหว่าง first render)
// 3. Background (ทำหลัง first render เสร็จ)

@MainActor
class AppStartup {
    func initialize() async {
        // 1. Critical: ต้องมีก่อน show UI
        await configureCritical()
        
        // Show first UI
        // ...
        
        // 2. Near-critical: ต้องมีก่อน user interaction
        Task(priority: .high) {
            await configureNearCritical()
        }
        
        // 3. Background: ไม่ urgent
        Task(priority: .background) {
            await configureBackground()
        }
    }
    
    private func configureCritical() async {
        // Authentication state check
        // Core Data stack setup
        // Essential user preferences
    }
    
    private func configureNearCritical() async {
        // Analytics setup
        // Push notification registration
        // Remote config fetch
    }
    
    private func configureBackground() async {
        // Cache warming
        // Prefetch data
        // Cleanup old files
        // Update app badges
    }
}
```

---

## 10. Energy Efficiency

### 10.1 Background Fetch Optimization

```swift
import BackgroundTasks

class BackgroundTaskManager {
    static let fetchIdentifier = "com.myapp.refresh"
    
    static func registerTasks() {
        BGTaskScheduler.shared.register(
            forTaskWithIdentifier: fetchIdentifier,
            using: nil
        ) { task in
            handleBackgroundFetch(task: task as! BGAppRefreshTask)
        }
    }
    
    static func scheduleBackgroundFetch() {
        let request = BGAppRefreshTaskRequest(identifier: fetchIdentifier)
        
        // ขอ refresh ไม่เร็วกว่านี้
        // iOS อาจ delay ตาม battery/usage pattern
        request.earliestBeginDate = Date(timeIntervalSinceNow: 15 * 60)  // 15 นาที
        
        try? BGTaskScheduler.shared.submit(request)
    }
    
    private static func handleBackgroundFetch(task: BGAppRefreshTask) {
        scheduleBackgroundFetch()  // Schedule next
        
        let fetchTask = Task {
            do {
                // ทำงานที่จำเป็นจริงๆ เท่านั้น
                let newData = try await fetchMinimalUpdates()
                await saveUpdates(newData)
                task.setTaskCompleted(success: true)
            } catch {
                task.setTaskCompleted(success: false)
            }
        }
        
        // Handle expiration
        task.expirationHandler = {
            fetchTask.cancel()
        }
    }
    
    private static func fetchMinimalUpdates() async throws -> [String: Any] {
        // เฉพาะ delta updates ไม่ใช่ full refresh
        let url = URL(string: "https://api.example.com/updates?since=\(lastUpdateTimestamp())")!
        let (data, _) = try await URLSession.shared.data(from: url)
        return try JSONSerialization.jsonObject(with: data) as! [String: Any]
    }
    
    private static func saveUpdates(_ data: [String: Any]) async { }
    private static func lastUpdateTimestamp() -> Int { 0 }
}
```

### 10.2 GPS Accuracy vs Battery

```swift
import CoreLocation

class EfficientLocationManager: NSObject, CLLocationManagerDelegate {
    private let manager = CLLocationManager()
    private var purpose: LocationPurpose = .background
    
    enum LocationPurpose {
        case navigation    // ต้องการ accuracy สูง
        case mapping       // ต้องการ accuracy ปานกลาง
        case background    // ต้องการ battery savings
        case geofencing    // ไม่ต้องการ GPS เลย
    }
    
    func configure(for purpose: LocationPurpose) {
        self.purpose = purpose
        
        switch purpose {
        case .navigation:
            manager.desiredAccuracy = kCLLocationAccuracyBestForNavigation
            manager.distanceFilter = 1  // update ทุก 1 เมตร
            manager.activityType = .automotiveNavigation
            
        case .mapping:
            manager.desiredAccuracy = kCLLocationAccuracyNearestTenMeters
            manager.distanceFilter = 10
            manager.activityType = .fitness
            
        case .background:
            // ✅ ประหยัดพลังงาน: accuracy ต่ำกว่า, filter สูงกว่า
            manager.desiredAccuracy = kCLLocationAccuracyHundredMeters
            manager.distanceFilter = 50
            manager.activityType = .other
            manager.pausesLocationUpdatesAutomatically = true  // iOS pause เมื่อ stationary
            
        case .geofencing:
            // ✅ ประหยัดพลังงานที่สุด: ไม่ใช้ GPS เลย!
            manager.stopUpdatingLocation()
            // ใช้ region monitoring แทน
        }
    }
}
```

### 10.3 Significant Location Change API

```swift
// Significant Location Change: update เฉพาะเมื่อเปลี่ยน cell tower
// ใช้พลังงานน้อยกว่า GPS มาก แต่ accuracy ต่ำกว่า (~500m-1km)

class SignificantLocationManager: NSObject, CLLocationManagerDelegate {
    private let manager = CLLocationManager()
    
    func startMonitoring() {
        manager.delegate = self
        
        // ✅ Significant location changes - battery efficient
        manager.startMonitoringSignificantLocationChanges()
        
        // ✅ Region monitoring - zero GPS usage
        let region = CLCircularRegion(
            center: CLLocationCoordinate2D(latitude: 13.7563, longitude: 100.5018),
            radius: 500,
            identifier: "Bangkok_Center"
        )
        region.notifyOnEntry = true
        region.notifyOnExit = true
        manager.startMonitoring(for: region)
    }
    
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        guard let location = locations.last else { return }
        print("Significant change: \(location.coordinate)")
        // Update user's location in database
        // Fetch location-relevant content
    }
    
    func locationManager(_ manager: CLLocationManager, didEnterRegion region: CLRegion) {
        print("Entered region: \(region.identifier)")
        // Show location-specific notification
    }
}
```

### 10.4 Timer Coalescing

```swift
// Timer coalescing: iOS groups timers เพื่อ wake CPU น้อยลง
// ใช้ tolerance ที่สูงกว่าเพื่อ enable coalescing

class EfficientTimer {
    private var timer: Timer?
    
    // ❌ Precise timer - wakes CPU ทุก interval พอดี
    func startPreciseTimer() {
        timer = Timer.scheduledTimer(withTimeInterval: 60, repeats: true) { _ in
            self.performPeriodicWork()
        }
        // tolerance = 0 (default) - ไม่มี coalescing
    }
    
    // ✅ Tolerant timer - iOS coalesce กับ timers อื่น
    func startEfficientTimer() {
        timer = Timer.scheduledTimer(withTimeInterval: 60, repeats: true) { _ in
            self.performPeriodicWork()
        }
        timer?.tolerance = 10  // ±10 วินาที - iOS สามารถ group timer นี้ได้
    }
    
    // ✅ ดีที่สุด: ใช้ GCD DispatchSourceTimer กับ leeway
    func startGCDTimer() {
        let source = DispatchSource.makeTimerSource(queue: .global(qos: .utility))
        source.schedule(
            deadline: .now() + 60,
            repeating: 60,
            leeway: .seconds(10)  // 10 วินาที leeway สำหรับ coalescing
        )
        source.setEventHandler {
            self.performPeriodicWork()
        }
        source.resume()
    }
    
    // ✅ ดีที่สุดสำหรับ async: Task.sleep กับ tolerance
    func startAsyncTimer() {
        Task(priority: .utility) {
            while !Task.isCancelled {
                try await Task.sleep(for: .seconds(60), tolerance: .seconds(10))
                await performPeriodicWorkAsync()
            }
        }
    }
    
    private func performPeriodicWork() {
        print("Periodic work at \(Date())")
    }
    
    private func performPeriodicWorkAsync() async {
        print("Async periodic work at \(Date())")
    }
}
```

---

## 11. Instruments Deep Dive

### 11.1 Time Profiler

```
การใช้ Time Profiler:
1. Product → Profile (Cmd+I)
2. เลือก Time Profiler template
3. Record การใช้งาน app
4. ดู Call Tree:
   - Hide System Libraries: ซ่อน system frames
   - Invert Call Tree: แสดง hottest functions ก่อน
   - Separate by Thread: ดู bottleneck ต่อ thread
5. หา "Heavy Stack Trace":
   - Functions ที่มี % สูงสุด = bottleneck
   - Double-click เพื่อ jump ไปที่ source code
```

```swift
// Code สำหรับ Time Profiler demo
// ก่อน optimization - O(n²) algorithm
class SlowProcessor {
    func findDuplicates_Slow(in array: [Int]) -> [Int] {
        var duplicates: [Int] = []
        for i in 0..<array.count {           // O(n)
            for j in (i+1)..<array.count {   // O(n)
                if array[i] == array[j] && !duplicates.contains(array[i]) {  // O(n)
                    duplicates.append(array[i])
                }
            }
        }
        return duplicates  // O(n³) total!
    }
}

// หลัง optimization - O(n) algorithm
class FastProcessor {
    func findDuplicates_Fast(in array: [Int]) -> [Int] {
        var seen = Set<Int>()
        var duplicates = Set<Int>()
        
        for element in array {   // O(n) เท่านั้น
            if !seen.insert(element).inserted {
                duplicates.insert(element)
            }
        }
        
        return Array(duplicates)  // O(n) total
    }
}

// os_signpost สำหรับ custom intervals ใน Instruments
import os

let signpostLog = OSLog(subsystem: "com.myapp", category: .pointsOfInterest)

func processWithSignpost(_ data: [Int]) -> [Int] {
    os_signpost(.begin, log: signpostLog, name: "findDuplicates")
    defer { os_signpost(.end, log: signpostLog, name: "findDuplicates") }
    
    return FastProcessor().findDuplicates_Fast(in: data)
}
```

### 11.2 Allocations Instrument

```swift
// Track memory allocations

// ❌ Pattern ที่ทำให้ Allocations spike
func badAllocationPattern() {
    for _ in 0..<10000 {
        // สร้าง string ใหม่ทุก iteration - 10,000 allocations
        let _ = String(repeating: "a", count: 1000)
    }
}

// ✅ Reuse buffer
func goodAllocationPattern() {
    var buffer = String()
    buffer.reserveCapacity(1000)
    
    for _ in 0..<10000 {
        buffer.removeAll(keepingCapacity: true)  // reuse allocation
        buffer.append(String(repeating: "a", count: 1000))
        _ = buffer
    }
}

// ❌ Closure ที่ capture ทำให้ retain cycle
class BadViewController: UIViewController {
    var data: [String] = []
    
    func loadData() {
        fetchDataFromNetwork { [self] result in  // ❌ strong reference cycle!
            self.data = result
        }
    }
    
    func fetchDataFromNetwork(completion: @escaping ([String]) -> Void) { }
}

// ✅ Weak capture
class GoodViewController: UIViewController {
    var data: [String] = []
    
    func loadData() {
        fetchDataFromNetwork { [weak self] result in  // ✅
            self?.data = result
        }
    }
    
    func fetchDataFromNetwork(completion: @escaping ([String]) -> Void) { }
}
```

### 11.3 Leaks Instrument

```swift
// Retain cycles - สาเหตุหลักของ memory leaks

// ❌ Classic retain cycle
class Node {
    var value: Int
    var next: Node?  // strong reference
    
    init(_ value: Int) { self.value = value }
}

// สร้าง cycle:
// let a = Node(1)
// let b = Node(2)
// a.next = b
// b.next = a  // cycle! ทั้งคู่ไม่ถูก deallocate

// ✅ Break cycle ด้วย weak
class TreeNode {
    var value: Int
    var children: [TreeNode] = []
    weak var parent: TreeNode?  // weak เพื่อ break cycle
    
    init(_ value: Int) { self.value = value }
    
    func addChild(_ child: TreeNode) {
        children.append(child)
        child.parent = self  // safe เพราะ weak
    }
}

// ❌ Delegate retain cycle
class DataManager {
    var delegate: DataManagerDelegate?  // ❌ strong
}

// ✅ Weak delegate
class DataManagerFixed {
    weak var delegate: DataManagerDelegate?  // ✅ weak
}

protocol DataManagerDelegate: AnyObject { }
```

---

## 12. Performance Testing ใน CI

### 12.1 XCTest Performance Metrics

```swift
import XCTest

class PerformanceTests: XCTestCase {
    // Basic performance test
    func testSortingPerformance() {
        let data = Array(0..<10000).shuffled()
        
        measure {
            var copy = data
            copy.sort()
        }
        // XCTest จะ run 10 iterations และ report mean/stddev
    }
    
    // ระบุ metrics เฉพาะ
    func testImageProcessingPerformance() throws {
        let image = UIImage(systemName: "photo")!
        let imageData = image.pngData()!
        
        let metrics: [XCTMetric] = [
            XCTClockMetric(),           // Wall clock time
            XCTMemoryMetric(),          // Memory usage
            XCTCPUMetric(),             // CPU usage
            XCTStorageMetric()          // I/O
        ]
        
        let options = XCTMeasureOptions()
        options.iterationCount = 5
        
        measure(metrics: metrics, options: options) {
            _ = downsample(imageData: imageData, to: CGSize(width: 100, height: 100))
        }
    }
    
    // Async performance test
    func testAsyncFetchPerformance() async throws {
        let client = MockNetworkClient()
        
        let options = XCTMeasureOptions()
        options.iterationCount = 10
        
        try await measure(metrics: [XCTClockMetric()], options: options) {
            _ = try await client.fetchData()
        }
    }
}

class MockNetworkClient {
    func fetchData() async throws -> [String] {
        try await Task.sleep(for: .milliseconds(50))
        return ["data1", "data2"]
    }
}
```

### 12.2 Baseline Management

```swift
// Baseline: ผล performance ที่ "acceptable"
// เมื่อ test รันครั้งแรก ต้อง set baseline:
// Product → Test → Edit Scheme → Set Baseline

// ใน CI:
// xcodebuild test -scheme MyApp -testPlan PerformanceTests
//                 -destination "platform=iOS Simulator,name=iPhone 16"

// ตั้ง performance threshold ด้วย code
func testWithCustomThreshold() {
    let options = XCTMeasureOptions()
    options.iterationCount = 20
    options.invocationOptions = .manuallyStop  // control เองว่าเมื่อไหร่หยุด
    
    measure(options: options) {
        performWork()
        stopMeasuring()  // หยุดวัดหลังจาก critical section
        performCleanup() // ไม่นับใน measurement
    }
}

func performWork() { /* ... */ }
func performCleanup() { /* ... */ }
```

### 12.3 Performance Regression Detection

```swift
// Custom performance tracker ใน CI pipeline
struct PerformanceReport {
    struct Measurement {
        let name: String
        let value: Double
        let unit: String
        let timestamp: Date
        let buildNumber: String
    }
    
    var measurements: [Measurement] = []
    
    mutating func record(name: String, value: Double, unit: String) {
        measurements.append(Measurement(
            name: name,
            value: value,
            unit: unit,
            timestamp: Date(),
            buildNumber: ProcessInfo.processInfo.environment["BUILD_NUMBER"] ?? "unknown"
        ))
    }
    
    func checkRegressions(against baseline: PerformanceReport, threshold: Double = 0.10) -> [String] {
        var regressions: [String] = []
        
        for current in measurements {
            guard let baselineMeasurement = baseline.measurements.first(where: { $0.name == current.name }) else { continue }
            
            let changePercent = (current.value - baselineMeasurement.value) / baselineMeasurement.value
            
            if changePercent > threshold {
                regressions.append(
                    "\(current.name): \(String(format: "+%.1f%%", changePercent * 100)) " +
                    "(baseline: \(baselineMeasurement.value)\(baselineMeasurement.unit), " +
                    "current: \(current.value)\(current.unit))"
                )
            }
        }
        
        return regressions
    }
}
```

---

## 13. Complete Optimization Case Study: Slow Table View → 60fps

### ปัญหา: UITableView ที่กระตุก

```swift
import UIKit

// ❌ BEFORE: Slow implementation ที่ทำงานบน main thread
class SlowTableViewController: UITableViewController {
    var items: [ArticleItem] = []
    
    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
        let item = items[indexPath.row]
        
        // ❌ 1. Load image synchronously บน main thread
        if let url = URL(string: item.imageURL) {
            let data = try? Data(contentsOf: url)  // blocking!
            cell.imageView?.image = data.flatMap { UIImage(data: $0) }
        }
        
        // ❌ 2. คำนวณ attributed string ทุกครั้ง
        let attributes: [NSAttributedString.Key: Any] = [
            .font: UIFont.systemFont(ofSize: 16),
            .foregroundColor: UIColor.black
        ]
        cell.textLabel?.attributedText = NSAttributedString(string: item.title, attributes: attributes)
        
        // ❌ 3. Date formatting ทุกครั้ง
        let formatter = DateFormatter()  // expensive to create!
        formatter.dateFormat = "dd MMM yyyy"
        cell.detailTextLabel?.text = formatter.string(from: item.date)
        
        return cell
    }
    
    override func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
        // ❌ 4. คำนวณ height ทุกครั้ง โดยไม่ cache
        let item = items[indexPath.row]
        return item.title.isEmpty ? 44 : 88
    }
}

struct ArticleItem {
    let title: String
    let imageURL: String
    let date: Date
}
```

```swift
// ✅ AFTER: Optimized implementation

// Pre-computed cell data
struct CellViewModel {
    let title: NSAttributedString
    let formattedDate: String
    let imageURL: URL?
    let rowHeight: CGFloat
}

class FastTableViewController: UITableViewController {
    var items: [ArticleItem] = []
    private var cellViewModels: [CellViewModel] = []
    private var heightCache: [IndexPath: CGFloat] = [:]
    
    // Shared, reusable formatter
    private static let dateFormatter: DateFormatter = {
        let f = DateFormatter()
        f.dateFormat = "dd MMM yyyy"
        return f
    }()
    
    // Shared text attributes
    private static let titleAttributes: [NSAttributedString.Key: Any] = [
        .font: UIFont.systemFont(ofSize: 16, weight: .medium),
        .foregroundColor: UIColor.label
    ]
    
    // Image loader กับ cache
    private let imageLoader = AsyncImageLoader()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        prepareViewModels()
    }
    
    // Pre-compute ทุกอย่างใน background
    private func prepareViewModels() {
        Task.detached(priority: .userInitiated) { [items] in
            let vms = items.map { item -> CellViewModel in
                let attributed = NSAttributedString(
                    string: item.title,
                    attributes: Self.titleAttributes
                )
                let date = Self.dateFormatter.string(from: item.date)
                let url = URL(string: item.imageURL)
                let height: CGFloat = item.title.count > 50 ? 88 : 60
                
                return CellViewModel(
                    title: attributed,
                    formattedDate: date,
                    imageURL: url,
                    rowHeight: height
                )
            }
            
            await MainActor.run {
                self.cellViewModels = vms
                self.tableView.reloadData()
            }
        }
    }
    
    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "FastCell", for: indexPath) as! FastCell
        
        if indexPath.row < cellViewModels.count {
            let vm = cellViewModels[indexPath.row]
            
            // ✅ ใช้ pre-computed data
            cell.titleLabel.attributedText = vm.title
            cell.dateLabel.text = vm.formattedDate
            
            // ✅ Load image async
            cell.articleImageView.image = nil  // reset
            if let url = vm.imageURL {
                imageLoader.loadImage(url: url) { [weak cell] image in
                    cell?.articleImageView.image = image
                }
            }
        }
        
        return cell
    }
    
    override func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
        // ✅ ใช้ pre-computed height
        guard indexPath.row < cellViewModels.count else { return 60 }
        return cellViewModels[indexPath.row].rowHeight
    }
    
    override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        cellViewModels.count
    }
}

class FastCell: UITableViewCell {
    var titleLabel = UILabel()
    var dateLabel = UILabel()
    var articleImageView = UIImageView()
}

// Async image loader กับ cache
class AsyncImageLoader {
    private let cache = NSCache<NSURL, UIImage>()
    private let queue = DispatchQueue(label: "imageLoader", qos: .userInitiated, attributes: .concurrent)
    
    func loadImage(url: URL, completion: @escaping (UIImage?) -> Void) {
        // Check cache
        if let cached = cache.object(forKey: url as NSURL) {
            completion(cached)
            return
        }
        
        // Load in background
        queue.async { [weak self] in
            guard let data = try? Data(contentsOf: url),
                  let image = UIImage(data: data) else {
                DispatchQueue.main.async { completion(nil) }
                return
            }
            
            self?.cache.setObject(image, forKey: url as NSURL)
            DispatchQueue.main.async { completion(image) }
        }
    }
}
```

### ผลลัพธ์การ optimize:

```
Before → After:
- Frame time: 32ms → 5ms (6.4x ดีขึ้น)
- Scrolling FPS: 30fps → 60fps
- Memory usage: เพิ่ม 50MB ลดลงเหลือ spike 5MB
- CPU usage ขณะ scroll: 85% → 12%
- Cold scroll (first-time): ลดการ stutter จาก 500ms → 50ms
```

---

## 14. Exercises กับ Solutions

### Exercise 1: Profile และ Optimize String Builder

```swift
// โจทย์: Function นี้ช้ามาก หาว่าทำไมและ optimize

// ❌ Original - O(n²)
func buildReport_Slow(items: [(name: String, value: Double)]) -> String {
    var result = ""
    result += "=== Report ===\n"
    
    for item in items {
        result += "\(item.name): \(String(format: "%.2f", item.value))\n"  // String concatenation O(n)!
    }
    
    result += "Total items: \(items.count)\n"
    result += "Average: \(String(format: "%.2f", items.map(\.value).reduce(0,+) / Double(items.count)))\n"
    
    return result
}

// Solution: ✅ O(n) กับ string interpolation ที่ efficient
func buildReport_Fast(items: [(name: String, value: Double)]) -> String {
    var parts: [String] = []
    parts.reserveCapacity(items.count + 3)
    
    parts.append("=== Report ===")
    
    let total = items.reduce(0.0) { $0 + $1.value }
    let average = items.isEmpty ? 0 : total / Double(items.count)
    
    for item in items {
        parts.append("\(item.name): \(String(format: "%.2f", item.value))")
    }
    
    parts.append("Total items: \(items.count)")
    parts.append("Average: \(String(format: "%.2f", average))")
    
    return parts.joined(separator: "\n")
}

// ✅ ดีที่สุด: ใช้ StringBuilder pattern
func buildReport_Optimal(items: [(name: String, value: Double)]) -> String {
    let total = items.reduce(0.0) { $0 + $1.value }
    let average = items.isEmpty ? 0 : total / Double(items.count)
    
    return """
    === Report ===
    \(items.map { "\($0.name): \(String(format: "%.2f", $0.value))" }.joined(separator: "\n"))
    Total items: \(items.count)
    Average: \(String(format: "%.2f", average))
    """
}

// Benchmark
func benchmarkStringBuilders() {
    let items = (1...10000).map { i in (name: "Item \(i)", value: Double(i) * 1.5) }
    let clock = ContinuousClock()
    
    let t1 = clock.now
    _ = buildReport_Slow(items: Array(items.prefix(100)))  // ใช้แค่ 100 เพราะ slow มาก
    print("Slow (100 items): \((clock.now - t1).formatted())")
    
    let t2 = clock.now
    _ = buildReport_Fast(items: items)
    print("Fast (10000 items): \((clock.now - t2).formatted())")
    
    let t3 = clock.now
    _ = buildReport_Optimal(items: items)
    print("Optimal (10000 items): \((clock.now - t3).formatted())")
}
```

### Exercise 2: Memory Leak ใน Closure Chain

```swift
// โจทย์: Code นี้มี memory leak หาและแก้ไข

// ❌ Leak version
class DataPipeline {
    var transformers: [(String) -> String] = []
    var completion: ((String) -> Void)?
    
    func addTransformer(_ t: @escaping (String) -> String) {
        transformers.append(t)
    }
    
    func execute(input: String) {
        var current = input
        for transformer in transformers {
            current = transformer(current)
        }
        completion?(current)
    }
}

class LeakyViewController: UIViewController {
    let pipeline = DataPipeline()
    var result = ""
    
    func setup() {
        pipeline.addTransformer { text in
            return text.uppercased()
        }
        
        // ❌ LEAK: strong reference cycle
        // pipeline.completion → closure → self → pipeline
        pipeline.completion = { [self] result in  // ❌ strong capture
            self.result = result
            self.updateUI()
        }
    }
    
    func updateUI() {
        print("Result: \(result)")
    }
}

// ✅ Fixed version
class FixedViewController: UIViewController {
    let pipeline = DataPipeline()
    var result = ""
    
    func setup() {
        pipeline.addTransformer { text in
            text.uppercased()
        }
        
        // ✅ Weak capture breaks cycle
        pipeline.completion = { [weak self] result in
            self?.result = result
            self?.updateUI()
        }
    }
    
    func updateUI() {
        print("Result: \(result)")
    }
    
    deinit {
        // ควรเห็น deinit ถ้าไม่มี leak
        print("FixedViewController deallocated")
    }
}
```

### Exercise 3: Optimize JSON Decoding Pipeline

```swift
// โจทย์: JSON decoding pipeline ที่ decode items ซ้ำๆ และ decode ทั้ง array ทุกครั้ง

struct ArticleDTO: Codable {
    let id: Int
    let title: String
    let content: String
    let authorId: Int
    let tags: [String]
    let createdAt: String
}

// ❌ Slow: Decode ทั้งหมดทุกครั้ง ไม่มี cache
class SlowArticleService {
    func getArticles(from jsonData: Data) throws -> [ArticleDTO] {
        return try JSONDecoder().decode([ArticleDTO].self, from: jsonData)
        // สร้าง JSONDecoder ใหม่ทุกครั้ง = expensive
    }
}

// ✅ Optimized version
class FastArticleService {
    // Reuse decoder
    private let decoder: JSONDecoder = {
        let d = JSONDecoder()
        d.keyDecodingStrategy = .convertFromSnakeCase
        // Configure once
        return d
    }()
    
    // Cache decoded results
    private var cache: [Int: ArticleDTO] = [:]
    private var lastData: Data?
    private var lastResult: [ArticleDTO]?
    
    func getArticles(from jsonData: Data) throws -> [ArticleDTO] {
        // Return cached result ถ้า data ไม่เปลี่ยน
        if jsonData == lastData, let cached = lastResult {
            return cached
        }
        
        let articles = try decoder.decode([ArticleDTO].self, from: jsonData)
        
        // Cache
        lastData = jsonData
        lastResult = articles
        
        return articles
    }
    
    // Decode concurrent กับ task group
    func decodeMultipleBatches(batches: [Data]) async throws -> [[ArticleDTO]] {
        return try await withThrowingTaskGroup(of: (Int, [ArticleDTO]).self) { group in
            for (index, batch) in batches.enumerated() {
                let decoderCopy = self.decoder  // สร้าง local reference
                group.addTask {
                    let articles = try decoderCopy.decode([ArticleDTO].self, from: batch)
                    return (index, articles)
                }
            }
            
            var results = Array(repeating: [ArticleDTO](), count: batches.count)
            for try await (index, articles) in group {
                results[index] = articles
            }
            return results
        }
    }
}
```

---

## สรุป

ใน Part 94 นี้เราได้เรียนรู้:

1. **Performance Measurement Science**: Amdahl's Law, profiling methodology, statistical significance, swift-benchmark
2. **Memory Performance**: Stack vs heap, allocation reduction, COW, buffer reuse, NSCache vs LRU
3. **CPU Performance**: Algorithmic complexity, branch prediction, SIMD, withUnsafe buffers
4. **Compiler Optimizations**: WMO, @_optimize, @inline, generic specialization
5. **SwiftUI Performance**: View identity, Equatable, @State granularity, LazyVStack vs List
6. **Core Data Performance**: Fetch request tuning, faulting, prefetching, compound predicates
7. **Network Performance**: HTTP/2, request deduplication, binary protocols
8. **Image Performance**: Downsampling, UIGraphicsImageRenderer, image pipeline
9. **Launch Time**: Static vs dynamic init, pre-main timing, dyld closures, task prioritization
10. **Energy Efficiency**: Background fetch, GPS accuracy, significant location, timer coalescing
11. **Instruments Deep Dive**: Time Profiler, Allocations, Leaks, System Trace
12. **Performance Testing in CI**: XCTest metrics, baseline management, regression detection
13. **Complete Case Study**: UITableView จาก 30fps เป็น 60fps
14. **Exercises**: String builders, memory leaks, JSON decoding pipeline

Performance optimization คือกระบวนการต่อเนื่อง - วัด, วิเคราะห์, แก้ไข, วัดใหม่ อย่าเดาและอย่า optimize ก่อนวัด!
