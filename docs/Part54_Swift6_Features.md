# Part 54: Swift 6 Features

## บทนำ

Swift 6 เป็นการเปลี่ยนแปลงครั้งใหญ่ที่สุดนับตั้งแต่ Swift 3 โดยเน้นที่ **Data Race Safety** ระดับ Compile Time, **Typed Throws**, **Noncopyable Types**, และ **Swift Macros** ที่ทรงพลัง ใน Part นี้เราจะเจาะลึกทุกฟีเจอร์ใหม่พร้อมตัวอย่างที่ใช้งานได้จริง

---

## 1. Swift 6 ภาพรวม

Swift 6 มีเป้าหมายหลักสามประการ:

1. **Safety** - ตรวจจับ Data Race ตั้งแต่ Compile Time
2. **Expressiveness** - Typed Throws, Noncopyable Types, Parameter Packs
3. **Performance** - Move Semantics, Optimized Executors

### การเปิดใช้ Swift 6 Mode

```swift
// Package.swift - เปิด Swift 6 สำหรับทั้ง Package
// swift-tools-version: 6.0

let package = Package(
    name: "MyPackage",
    targets: [
        .target(
            name: "MyTarget",
            swiftSettings: [
                .swiftLanguageVersion(.v6)
            ]
        )
    ]
)

// เปิด Strict Concurrency Checking ใน Swift 5.10 (ก่อน Migrate)
// swiftSettings: [
//     .enableExperimentalFeature("StrictConcurrency")
// ]
```

---

## 2. Concurrency Changes ใน Swift 6

Swift 6 เปลี่ยน Default Concurrency Checking จาก Minimal เป็น Complete

```swift
// Swift 5 - ไม่มีการตรวจสอบ (อาจมี Data Race)
class Counter_Swift5 {
    var value = 0  // ไม่ปลอดภัย
    
    func increment() {
        value += 1  // Data Race ได้!
    }
}

// Swift 6 - ต้องแก้ไขให้ถูกต้อง
actor Counter_Swift6 {
    var value = 0  // ปลอดภัยเพราะอยู่ใน Actor
    
    func increment() {
        value += 1
    }
}

// หรือใช้ @MainActor
@MainActor
class UICounter {
    var value = 0  // ปลอดภัยเพราะรันบน Main Thread เสมอ
    
    func increment() {
        value += 1
    }
}

// Swift 6 ยังบังคับ Sendable ใน Closure ที่ข้าม Isolation
func demonstrateSendableEnforcement() {
    let counter = Counter_Swift6()
    
    Task {
        // ✅ ถูกต้อง: counter เป็น Actor ซึ่ง Sendable
        await counter.increment()
    }
    
    // ❌ Swift 6 จะ Error ถ้า counter ไม่ใช่ Sendable
    // Task {
    //     unsafeReference.doSomething()  // Error!
    // }
}
```

---

## 3. Strict Concurrency Checking

Swift 6 เพิ่มระดับการตรวจสอบใหม่

```swift
// ระดับการตรวจสอบ:
// minimal   - เฉพาะ Explicit async/await (Swift 5 Default)
// targeted  - ตรวจสอบเฉพาะที่ทำเครื่องหมายไว้
// complete  - ตรวจสอบทุกอย่าง (Swift 6 Default)

// ตัวอย่างที่ผ่านใน Swift 5 แต่ Error ใน Swift 6

// ❌ Error ใน Swift 6: Non-Sendable type ข้าม Isolation Boundary
class NonSendableClass {
    var data: [Int] = []
}

func trySendingNonSendable() {
    let obj = NonSendableClass()
    
    Task {
        // Swift 6 Error: Capture of 'obj' with non-sendable type 'NonSendableClass'
        // obj.data.append(1)  // ← Error!
        
        // ✅ แก้ไขโดยทำให้เป็น Sendable
        let dataCopy = obj.data  // Capture ค่า ไม่ใช่ Reference
        print(dataCopy)
    }
}

// ✅ แก้ไขโดยทำให้ Class เป็น Sendable
final class SendableClass: @unchecked Sendable {
    // @unchecked เพราะเราจัดการ Thread Safety เอง
    private let lock = NSLock()
    private var data: [Int] = []
    
    func append(_ value: Int) {
        lock.withLock {
            data.append(value)
        }
    }
    
    func getAll() -> [Int] {
        lock.withLock { data }
    }
}
```

---

## 4. Complete Concurrency Checking

เมื่อเปิด Complete Concurrency Checking Swift จะตรวจสอบทุก Concurrency Boundary

```swift
// Complete Checking ตรวจสอบ:
// 1. Sendable Conformance
// 2. Actor Isolation
// 3. @Sendable Closures
// 4. Global Variables

// ❌ Global Mutable State - Error ใน Swift 6
// var globalCounter = 0  // Error: Global variable 'globalCounter' is not concurrency-safe

// ✅ แก้ไขด้วย Actor หรือ nonisolated(unsafe)
actor GlobalState {
    static let shared = GlobalState()
    private var counter = 0
    
    func increment() { counter += 1 }
    var value: Int { counter }
}

// หรือ
nonisolated(unsafe) var legacyGlobal = 0  // บอก Swift ว่าเราจัดการเอง

// ❌ Error: Non-Sendable Type ข้าม Isolation
struct NonSendableData {
    var value: Int
}

actor DataProcessor {
    func process() -> NonSendableData {
        return NonSendableData(value: 42)
    }
}

// ✅ แก้ไขโดยทำให้ Struct เป็น Sendable (โดยอัตโนมัติสำหรับ Struct ที่มี Sendable Properties)
struct SendableData: Sendable {
    var value: Int
}

// ตัวอย่าง Protocol Conformance ที่ต้องปรับ
protocol DataSource: Sendable {
    func fetchData() async -> [String]
}

class NetworkDataSource: DataSource, @unchecked Sendable {
    // ต้องเพิ่ม @unchecked Sendable เพราะ Class ไม่ใช่ Sendable โดยอัตโนมัติ
    
    func fetchData() async -> [String] {
        return ["item1", "item2"]
    }
}
```

---

## 5. Data Race Detection ระดับ Compile Time

Swift 6 สามารถตรวจจับ Potential Data Races ตั้งแต่เขียนโค้ด

```swift
// Swift 6 ตรวจจับ Data Races เหล่านี้:

// ❌ Race 1: Shared Mutable State
class SharedMutable {
    var counter = 0
    
    func incrementFromMultipleThreads() {
        // Swift 6 จะ Warn/Error เกี่ยวกับ Shared Access
        DispatchQueue.concurrentPerform(iterations: 1000) { _ in
            // self.counter += 1  // ← Data Race!
        }
    }
}

// ✅ แก้ไข
actor SafeSharedState {
    private var counter = 0
    
    func increment() {
        counter += 1
    }
    
    var value: Int { counter }
}

// ❌ Race 2: Captured Variable ใน Concurrent Closures
func raceInClosure() {
    var shared = 0
    
    // ✅ Swift 6 บังคับให้ Capture เป็น [shared]
    let task1 = Task { [shared] in  // Capture by value
        print("Task 1: \(shared)")
    }
    
    let task2 = Task { [shared] in  // Capture by value
        print("Task 2: \(shared)")
    }
    
    shared = 100  // ไม่กระทบ Task ที่ Capture ไปแล้ว
    
    _ = (task1, task2)
}

// ❌ Race 3: Non-isolated Stored Properties
// Swift 6 กำหนดว่า Stored Properties ของ Non-isolated Type
// ที่เข้าถึงจาก Concurrent Context ต้องเป็น Sendable

struct Config: Sendable {
    let maxConnections: Int
    let timeout: Double
    let endpoints: [String]  // [String] เป็น Sendable
}

// ใช้ Config อย่างปลอดภัย
actor ConnectionPool {
    private let config: Config
    private var connections: [String] = []
    
    init(config: Config) {
        self.config = config
    }
    
    func addConnection(_ id: String) {
        guard connections.count < config.maxConnections else { return }
        connections.append(id)
    }
}
```

---

## 6. Sendable Enforcement

Swift 6 บังคับ Sendable อย่างเข้มงวด

```swift
// Sendable Protocol และ Rules

// ✅ Value Types เป็น Sendable โดยอัตโนมัติ (ถ้า Properties เป็น Sendable)
struct Point: Sendable {  // implicit Sendable
    var x: Double
    var y: Double
}

// ✅ Enum เป็น Sendable โดยอัตโนมัติ
enum Status: Sendable {
    case active
    case inactive
    case pending(message: String)  // String เป็น Sendable
}

// ❌ Class ไม่ได้เป็น Sendable โดยอัตโนมัติ
// final class MyClass: Sendable { ... }  // ต้องระบุเอง

// ✅ Final Class สามารถเป็น Sendable ได้ถ้า Properties เป็น Sendable
final class ImmutableConfig: Sendable {
    let name: String  // let + Sendable type = ปลอดภัย
    let maxCount: Int
    
    init(name: String, maxCount: Int) {
        self.name = name
        self.maxCount = maxCount
    }
}

// ✅ @unchecked Sendable สำหรับ Class ที่เราจัดการ Thread Safety เอง
final class ThreadSafeQueue<T: Sendable>: @unchecked Sendable {
    private var storage: [T] = []
    private let lock = NSLock()
    
    func enqueue(_ item: T) {
        lock.withLock { storage.append(item) }
    }
    
    func dequeue() -> T? {
        lock.withLock {
            storage.isEmpty ? nil : storage.removeFirst()
        }
    }
}

// Sendable Closure
func processAsync(completion: @Sendable @escaping () -> Void) {
    Task {
        // completion ถูก Capture ข้าม Task Boundary
        // @Sendable บอก Swift ว่า Closure นี้ปลอดภัยสำหรับ Concurrent Use
        completion()
    }
}

// Sending Parameter (Swift 6 ใหม่)
// บอกว่า Caller ต้องส่ง Ownership ไป
func consume(sending value: some Any) {
    // value ถูกส่งมาพร้อม Ownership
    print("Consumed: \(value)")
}
```

---

## 7. Typed Throws

Swift 6 เพิ่ม Typed Throws ที่ให้เราระบุ Error Type ที่ Function จะ Throw ได้

```swift
// Swift 5 - ไม่ระบุ Type ของ Error
func oldStyle() throws {
    throw NSError(domain: "test", code: 1)  // Throw อะไรก็ได้
}

// Swift 6 - Typed Throws
enum NetworkError: Error {
    case connectionFailed(reason: String)
    case timeout(after: Double)
    case invalidResponse(statusCode: Int)
    case unauthorized
}

// ระบุ Error Type ด้วย throws(NetworkError)
func fetchUser(id: String) throws(NetworkError) -> User {
    guard !id.isEmpty else {
        throw NetworkError.invalidResponse(statusCode: 400)
    }
    
    // จำลองการดึงข้อมูล
    if id == "banned" {
        throw NetworkError.unauthorized
    }
    
    return User(id: id, name: "ผู้ใช้ \(id)")
}

// ข้อดีของ Typed Throws:
// 1. Compiler รู้ว่า Catch Block จะรับ Error Type อะไร
// 2. ไม่ต้องทำ Type Casting ใน Catch
// 3. Exhaustive Catch ที่ Compile Time

do {
    let user = try fetchUser(id: "user123")
    print("ได้รับ User: \(user.name)")
} catch NetworkError.connectionFailed(let reason) {
    print("เชื่อมต่อไม่ได้: \(reason)")
} catch NetworkError.timeout(let seconds) {
    print("Timeout หลัง \(seconds) วินาที")
} catch NetworkError.invalidResponse(let code) {
    print("Response ไม่ถูกต้อง: HTTP \(code)")
} catch NetworkError.unauthorized {
    print("ไม่มีสิทธิ์เข้าถึง")
}
// ไม่ต้องมี catch {} สำหรับ Error อื่น เพราะ Compiler รู้ว่า Exhaustive แล้ว

// Typed Throws กับ Generic
func map<T, U, E: Error>(
    _ array: [T],
    transform: (T) throws(E) -> U
) throws(E) -> [U] {
    var result: [U] = []
    for element in array {
        result.append(try transform(element))
    }
    return result
}

// ใช้งาน
let numbers = ["1", "2", "three", "4"]

enum ParseError: Error {
    case invalidNumber(String)
}

do {
    let integers = try map(numbers) { str throws(ParseError) -> Int in
        guard let int = Int(str) else {
            throw ParseError.invalidNumber(str)
        }
        return int
    }
    print("แปลงสำเร็จ: \(integers)")
} catch ParseError.invalidNumber(let str) {
    print("แปลงไม่ได้: '\(str)'")
}

// Async Typed Throws
func fetchData(from url: String) async throws(NetworkError) -> Data {
    guard url.hasPrefix("https://") else {
        throw NetworkError.invalidResponse(statusCode: 400)
    }
    
    // จำลองการดาวน์โหลด
    try? await Task.sleep(nanoseconds: 100_000_000)
    return "Mock Data".data(using: .utf8)!
}

struct User {
    let id: String
    let name: String
}
```

---

## 8. Noncopyable Types (Move-Only Types)

Noncopyable Types ช่วยป้องกัน Accidental Copy และ จัดการ Ownership อย่างแม่นยำ

```swift
// ~Copyable บอกว่า Type นี้ไม่สามารถ Copy ได้
struct UniqueFile: ~Copyable {
    private let fileDescriptor: Int32
    private let path: String
    
    init(path: String) throws {
        self.path = path
        // เปิดไฟล์จริง
        self.fileDescriptor = open(path, O_RDONLY)
        guard fileDescriptor >= 0 else {
            throw FileError.cannotOpen(path)
        }
        print("เปิดไฟล์: \(path)")
    }
    
    // deinit จะถูกเรียกเมื่อ UniqueFile ถูก Consume
    deinit {
        close(fileDescriptor)
        print("ปิดไฟล์: \(path)")
    }
    
    consuming func readAll() -> String {
        // หลังจากเรียก consuming method นี้
        // UniqueFile จะถูก "Consumed" และ deinit จะถูกเรียก
        var content = ""
        var buffer = [CChar](repeating: 0, count: 1024)
        
        while read(fileDescriptor, &buffer, 1024) > 0 {
            content += String(cString: buffer)
        }
        
        return content
    }
    
    borrowing func fileSize() -> Int64 {
        var stat = stat()
        fstat(fileDescriptor, &stat)
        return stat.st_size
    }
}

enum FileError: Error {
    case cannotOpen(String)
}

// ✅ การใช้งาน Noncopyable Type
func processFile(path: String) throws {
    var file = try UniqueFile(path: path)
    
    // ✅ Borrowing - อ่านข้อมูลโดยไม่ Consume
    let size = file.fileSize()
    print("ขนาดไฟล์: \(size) bytes")
    
    // ✅ Consuming - อ่านทั้งหมดและ Consume ไฟล์
    let content = file.readAll()
    print("เนื้อหา: \(content.prefix(100))")
    
    // ❌ ไม่สามารถใช้ file อีกต่อไปหลังจาก consume
    // let size2 = file.fileSize()  // Error: 'file' used after consuming
}

// Noncopyable ใน Generics
func process<T: ~Copyable>(_ value: consuming T) {
    // รับ Ownership ของ value
    print("กำลังประมวลผล...")
    // value ถูก Consumed ที่นี่
}

// Optional ของ Noncopyable Type
func maybeProcess(file: consuming UniqueFile?) {
    if let file = consume file {
        // file ถูก Consumed ที่นี่
        print("ขนาด: \(file.fileSize())")
    } else {
        print("ไม่มีไฟล์")
    }
}
```

---

## 9. ~Copyable Constraint

`~Copyable` ใช้เพื่อระบุว่า Type Parameter สามารถรับ Noncopyable Types ได้

```swift
// ~Copyable ใน Protocol
protocol Resource: ~Copyable {
    consuming func release()
}

// Noncopyable Struct ที่ Conform Protocol
struct GPUBuffer: ~Copyable, Resource {
    private let id: Int
    
    init(size: Int) {
        self.id = createGPUBuffer(size: size)
        print("สร้าง GPU Buffer #\(id) ขนาด \(size) bytes")
    }
    
    consuming func release() {
        destroyGPUBuffer(id: id)
        print("ปล่อย GPU Buffer #\(id)")
    }
    
    borrowing func write(_ data: [Float]) {
        writeToGPUBuffer(id: id, data: data)
    }
    
    deinit {
        // deinit เป็น consuming operation โดยอัตโนมัติ
        print("deinit GPU Buffer #\(id)")
    }
}

// Stub functions
func createGPUBuffer(size: Int) -> Int { Int.random(in: 1...1000) }
func destroyGPUBuffer(id: Int) { }
func writeToGPUBuffer(id: Int, data: [Float]) { }

// Generic Function ที่รับ Noncopyable Types
func withResource<R: Resource & ~Copyable, T>(
    _ resource: consuming R,
    do work: (borrowing R) throws -> T
) rethrows -> T {
    defer { resource.release() }  // ✅ ปล่อย Resource เสมอ
    return try work(resource)
}

// ใช้งาน
func renderFrame() {
    let buffer = GPUBuffer(size: 1024 * 1024)
    
    withResource(buffer) { buf in
        buf.write([1.0, 0.0, 0.5, 1.0])
        print("เรนเดอร์ Frame")
    }
    // buffer ถูก release อัตโนมัติ
}

// Enum ที่เป็น Noncopyable
enum Either<Left: ~Copyable, Right: ~Copyable>: ~Copyable {
    case left(Left)
    case right(Right)
    
    consuming func map<T>(
        left: (consuming Left) -> T,
        right: (consuming Right) -> T
    ) -> T {
        switch consume self {
        case .left(let l):
            return left(l)
        case .right(let r):
            return right(r)
        }
    }
}
```

---

## 10. Consuming และ Borrowing Parameter Modifiers

Swift 6 เพิ่ม `consuming` และ `borrowing` เพื่อควบคุม Ownership อย่างชัดเจน

```swift
// consuming - Transfer Ownership ไปยัง Function
// borrowing - อ่านโดยไม่ Transfer Ownership
// inout     - Transfer และคืน Ownership (ที่มีอยู่แล้ว)

struct LargeData: ~Copyable {
    private var buffer: [Int]
    
    init(size: Int) {
        self.buffer = Array(0..<size)
    }
    
    // consuming - Function นี้ "เป็นเจ้าของ" data และจะ Destroy หลังเสร็จ
    consuming func processAndDiscard() -> Int {
        return buffer.reduce(0, +)
    }
    
    // borrowing - Function นี้แค่ "อ่าน" data ไม่ได้เป็นเจ้าของ
    borrowing func sum() -> Int {
        return buffer.reduce(0, +)
    }
    
    // mutating - เปลี่ยนแปลง data
    mutating func transform(by factor: Int) {
        buffer = buffer.map { $0 * factor }
    }
}

// ตัวอย่างการใช้งาน
func demonstrateOwnership() {
    var data = LargeData(size: 1000)
    
    // borrowing - data ยังใช้ได้หลัง call
    let sum1 = data.sum()
    print("ผลรวมครั้งที่ 1: \(sum1)")
    
    // borrowing อีกครั้ง - ยังใช้ได้
    let sum2 = data.sum()
    print("ผลรวมครั้งที่ 2: \(sum2)")
    
    // mutating - data ยังใช้ได้
    data.transform(by: 2)
    
    // consuming - data ถูก Consumed หลังจากนี้
    let finalResult = data.processAndDiscard()
    print("ผลลัพธ์สุดท้าย: \(finalResult)")
    
    // ❌ data ไม่สามารถใช้ได้อีกต่อไป
    // let sum3 = data.sum()  // Error!
}

// Borrowing ใน Function Parameters
func analyzeData(borrowing data: LargeData) -> String {
    // ฟังก์ชันนี้ไม่ได้เป็นเจ้าของ data
    let total = data.sum()
    return "ผลรวม: \(total)"
}

// Consuming ใน Function Parameters
func archiveData(consuming data: LargeData) {
    // ฟังก์ชันนี้ได้รับ Ownership และจะ Destroy data
    let result = data.processAndDiscard()
    print("เก็บถาวรด้วยผลรวม: \(result)")
    // data ถูก destroy ที่นี่
}

// copy() - บังคับ Copy สำหรับ Copyable Type
struct CopyableData {
    var values: [Int]
    
    func demonstrateCopy() {
        let original = CopyableData(values: [1, 2, 3])
        let copied = copy original  // บังคับ Copy แทน Move
        print("Original: \(original.values)")
        print("Copied: \(copied.values)")
    }
}
```

---

## 11. Pack Iteration (for-in บน Parameter Packs)

Swift 6 เพิ่ม Pack Iteration ที่ช่วยให้ Loop บน Parameter Packs ได้

```swift
// Parameter Packs (Value Generics) ใน Swift 5.9+
// Pack Iteration ใน Swift 6

// ฟังก์ชันที่รับ Parameter Pack
func printAll<each T>(_ values: repeat each T) {
    repeat print(each values)
}

// ใช้งาน
printAll(1, "สวัสดี", 3.14, true)
// Output:
// 1
// สวัสดี
// 3.14
// true

// Pack Iteration ด้วย for-in (Swift 6)
func transformAll<each Input, each Output>(
    _ inputs: repeat each Input,
    transform: repeat (each Input) -> each Output
) -> (repeat each Output) {
    return (repeat (each transform)(each inputs))
}

// Zip ด้วย Parameter Packs
func zip2<each T, each U>(
    _ firsts: repeat each T,
    _ seconds: repeat each U
) -> (repeat (each T, each U)) {
    return (repeat (each firsts, each seconds))
}

// ใช้ Parameter Packs สำหรับ Validation
protocol Validatable {
    func validate() throws
}

func validateAll<each T: Validatable>(_ items: repeat each T) throws {
    repeat try (each items).validate()
}

// ตัวอย่างการใช้งาน
struct Email: Validatable {
    let address: String
    
    func validate() throws {
        guard address.contains("@") else {
            throw ValidationError.invalidEmail(address)
        }
    }
}

struct Phone: Validatable {
    let number: String
    
    func validate() throws {
        guard number.count >= 10 else {
            throw ValidationError.invalidPhone(number)
        }
    }
}

enum ValidationError: Error {
    case invalidEmail(String)
    case invalidPhone(String)
}

// ตรวจสอบหลาย Types พร้อมกัน
let email = Email(address: "test@example.com")
let phone = Phone(number: "0812345678")

do {
    try validateAll(email, phone)
    print("ข้อมูลทั้งหมดถูกต้อง")
} catch {
    print("ข้อมูลไม่ถูกต้อง: \(error)")
}
```

---

## 12. Expression Macros

Expression Macros สร้าง Code ที่ Return ค่า ณ Compile Time

```swift
// สร้าง Expression Macro (ต้องอยู่ใน Separate Module)
import SwiftSyntax
import SwiftSyntaxMacros

// ประกาศ Macro
@freestanding(expression)
macro stringify<T>(_ value: T) -> (T, String) = #externalMacro(
    module: "MyMacros",
    type: "StringifyMacro"
)

// Implementation (ใน Macro Module)
// public struct StringifyMacro: ExpressionMacro {
//     public static func expansion(
//         of node: some FreestandingMacroExpansionSyntax,
//         in context: some MacroExpansionContext
//     ) throws -> ExprSyntax {
//         guard let argument = node.arguments.first?.expression else {
//             throw MacroError.missingArgument
//         }
//         return "(\(argument), \(literal: argument.description))"
//     }
// }

// ตัวอย่าง Expression Macros ที่มีใน Swift Standard Library
// #line, #column, #file, #function
func demonstrateBuiltinMacros() {
    print("File: \(#file)")
    print("Line: \(#line)")
    print("Function: \(#function)")
    
    // #if สำหรับ Conditional Compilation
    #if DEBUG
    print("กำลังรันใน Debug Mode")
    #else
    print("กำลังรันใน Release Mode")
    #endif
}

// สร้าง URL ที่ตรวจสอบ Compile Time
@freestanding(expression)
macro URL(_ string: String) -> URL = #externalMacro(module: "MyMacros", type: "URLMacro")

// ใช้งาน (หลังจาก Implement Macro แล้ว)
// let apiURL = #URL("https://api.example.com/v1")  // Error ถ้า URL ไม่ถูกต้อง

// Freestanding Declaration Macro
@freestanding(declaration, names: named(makeAdder))
macro makeAdder(for base: Int) = #externalMacro(module: "MyMacros", type: "MakeAdderMacro")

// ใช้งาน
// #makeAdder(for: 10)
// // ขยายเป็น:
// func makeAdder(adding value: Int) -> Int {
//     return 10 + value
// }
```

---

## 13. Attached Macros

Attached Macros แนบกับ Type, Property หรือ Function Declaration

```swift
// ประเภทของ Attached Macros:
// @attached(member)          - เพิ่ม Members ให้ Type
// @attached(memberAttribute) - เพิ่ม Attributes ให้ Members
// @attached(accessor)        - เพิ่ม get/set/willSet/didSet
// @attached(conformance)     - เพิ่ม Protocol Conformance
// @attached(peer)            - สร้าง Declaration คู่

// ตัวอย่าง: @CodingKeys Macro
@attached(member, names: named(CodingKeys))
@attached(conformance)
macro CodingKeys() = #externalMacro(module: "MyMacros", type: "CodingKeysMacro")

// @EquatableByID Macro
@attached(conformance)
@attached(member, names: named(==))
macro EquatableByID() = #externalMacro(module: "MyMacros", type: "EquatableByIDMacro")

// ใช้งาน Attached Macros ที่มีใน Swift
// @Observable (Swift 5.9+)
import Observation

@Observable
class UserViewModel {
    var name: String = ""
    var email: String = ""
    var isLoading: Bool = false
    
    // @Observable ขยายเป็น:
    // - Access tracking สำหรับแต่ละ Property
    // - _$observationRegistrar
    // - withObservationTracking
}

// เทียบกับ ObservableObject เดิม
// class OldUserViewModel: ObservableObject {
//     @Published var name: String = ""
//     @Published var email: String = ""
// }

// ตัวอย่าง Custom Attached Macro สำหรับ Logging
@attached(accessor)
macro Logged() = #externalMacro(module: "MyMacros", type: "LoggedMacro")

// ใช้งาน
class Configuration {
    @Logged
    var apiKey: String = ""
    // ขยายเป็น:
    // get { return _apiKey }
    // set {
    //     print("apiKey เปลี่ยนจาก '\(_apiKey)' เป็น '\(newValue)'")
    //     _apiKey = newValue
    // }
}
```

---

## 14. @Observable Macro ภายใน

`@Observable` เป็น Macro ที่มากับ Swift 5.9/6 แทน `ObservableObject`

```swift
import Observation
import SwiftUI  // สำหรับ View

// @Observable สร้าง Observable Model อย่างอัตโนมัติ
@Observable
class ShoppingCart {
    var items: [CartItem] = []
    var couponCode: String = ""
    var isCheckingOut: Bool = false
    
    // Computed Property ทำงานได้ตามปกติ
    var total: Double {
        items.reduce(0) { $0 + $1.price * Double($1.quantity) }
    }
    
    var itemCount: Int {
        items.reduce(0) { $0 + $1.quantity }
    }
    
    func addItem(_ item: CartItem) {
        if let index = items.firstIndex(where: { $0.id == item.id }) {
            items[index].quantity += 1
        } else {
            items.append(item)
        }
    }
    
    func removeItem(id: String) {
        items.removeAll { $0.id == id }
    }
    
    func applyCoupon(_ code: String) async throws -> Double {
        isCheckingOut = true
        defer { isCheckingOut = false }
        
        // จำลองการตรวจสอบ Coupon
        try await Task.sleep(nanoseconds: 500_000_000)
        
        return code == "SAVE10" ? 0.9 : 1.0
    }
}

struct CartItem: Identifiable {
    let id: String
    let name: String
    let price: Double
    var quantity: Int
}

// ใช้ใน SwiftUI View
// struct CartView: View {
//     @State private var cart = ShoppingCart()
//
//     var body: some View {
//         VStack {
//             Text("รายการสินค้า: \(cart.itemCount)")
//             Text("รวม: \(cart.total, format: .currency(code: "THB"))")
//         }
//     }
// }

// ความแตกต่างจาก ObservableObject
// 1. ไม่ต้องใช้ @Published
// 2. Performance ดีกว่า - Track เฉพาะ Property ที่ View ใช้จริง
// 3. Works กับ @State, @Environment โดยตรง
// 4. Non-class types รองรับ (struct, enum ใน future)

// withObservationTracking - ใช้ Listen การเปลี่ยนแปลง
func observeCart(_ cart: ShoppingCart) {
    withObservationTracking {
        // Access properties ที่ต้องการ Track
        let count = cart.itemCount
        print("จำนวนสินค้า: \(count)")
    } onChange: {
        print("Cart เปลี่ยนแปลงแล้ว!")
        // จะถูกเรียกครั้งเดียวเมื่อ Property ที่ Access เปลี่ยน
    }
}
```

---

## 15. Parameter Packs และ Variadic Generics

Parameter Packs ช่วยให้ Generic Function รับ Types ที่แตกต่างกันจำนวนมากได้

```swift
// Variadic Generic (Parameter Pack)
// each T - ประกาศ Pack ของ Types

// ฟังก์ชันที่รับ Values หลาย Types
func describe<each T>(_ values: repeat each T) -> String {
    var descriptions: [String] = []
    repeat descriptions.append("\(each values)")
    return descriptions.joined(separator: ", ")
}

// ใช้งาน
let desc = describe(42, "Swift", 3.14, true)
print(desc)  // "42, Swift, 3.14, true"

// Tuple ด้วย Parameter Packs
func makeTuple<each T>(_ values: repeat each T) -> (repeat each T) {
    return (repeat each values)
}

let tuple = makeTuple(1, "two", 3.0)
print(tuple.0, tuple.1, tuple.2)  // 1 two 3.0

// Map บน Parameter Pack
func mapPack<each T, each U>(
    _ values: repeat each T,
    transform: repeat (each T) -> each U
) -> (repeat each U) {
    return (repeat (each transform)(each values))
}

// Constraint บน Pack
func sumAll<each T: Numeric>(_ values: repeat each T) -> Double {
    var total = 0.0
    repeat (total += Double("\(each values)") ?? 0)
    return total
}

// Higher-Kinded Types Pattern
struct Validator<each Input> {
    let validators: (repeat (each Input) -> Bool)
    
    init(_ validators: repeat (each Input) -> Bool) {
        self.validators = (repeat each validators)
    }
    
    func validate(_ inputs: repeat each Input) -> Bool {
        var allValid = true
        repeat (allValid = allValid && (each validators)(each inputs))
        return allValid
    }
}

// ใช้งาน
let validator = Validator(
    { (n: Int) in n > 0 },
    { (s: String) in !s.isEmpty },
    { (b: Bool) in b }
)

let isValid = validator.validate(5, "hello", true)
print("ถูกต้อง: \(isValid)")  // ถูกต้อง: true
```

---

## 16. any กับ some: ความเปลี่ยนแปลงใน Swift 6

Swift 6 บังคับให้ใช้ `any` สำหรับ Existential Types และ `some` สำหรับ Opaque Types อย่างชัดเจน

```swift
protocol Drawable {
    func draw() -> String
    var color: String { get }
}

struct Circle: Drawable {
    var radius: Double
    var color: String
    
    func draw() -> String {
        return "วงกลมรัศมี \(radius) สี \(color)"
    }
}

struct Rectangle: Drawable {
    var width: Double
    var height: Double
    var color: String
    
    func draw() -> String {
        return "สี่เหลี่ยม \(width)x\(height) สี \(color)"
    }
}

// some - Opaque Type (Compile-time Static Type)
// ✅ Performance ดีกว่า - ไม่มี Dynamic Dispatch
func makeDefaultShape() -> some Drawable {
    return Circle(radius: 10, color: "แดง")
    // Caller รู้ว่าเป็น Drawable แต่ไม่รู้ว่าเป็น Circle
}

// any - Existential Type (Runtime Dynamic Type)
// ✅ ยืดหยุ่นกว่า - สามารถ Return Type ต่างๆ ได้
func makeShape(isCircle: Bool) -> any Drawable {
    if isCircle {
        return Circle(radius: 5, color: "น้ำเงิน")
    } else {
        return Rectangle(width: 10, height: 5, color: "เขียว")
    }
}

// Array ของ Existential Types
let shapes: [any Drawable] = [
    Circle(radius: 3, color: "แดง"),
    Rectangle(width: 4, height: 6, color: "เขียว"),
    Circle(radius: 8, color: "น้ำเงิน")
]

for shape in shapes {
    print(shape.draw())
}

// ❌ Swift 6 Error: Protocol ต้องใช้ 'any' สำหรับ Existential
// let shape: Drawable = Circle(...)  // ต้องเขียน any Drawable

// Opener บน Existential (Swift 5.7+)
func openExistential(shape: any Drawable) {
    // func openAndProcess<T: Drawable>(_ s: T) { ... }
    // ไม่สามารถเรียก Generic Function ด้วย Existential โดยตรง
    
    // ✅ ใช้ 'any' type เพื่อ Open Existential
    let description = shape.draw()
    print(description)
}

// Primary Associated Types กับ any
protocol Collection2<Element> {
    associatedtype Element
    func first() -> Element?
}

// any Collection2<String> - Constrained Existential
func processStrings(collection: any Collection2<String>) {
    if let first = collection.first() {
        print("รายการแรก: \(first)")
    }
}
```

---

## 17. การ Migrate จาก Swift 5 ไปยัง Swift 6

```swift
// ขั้นตอนการ Migrate:
// 1. เปิด Strict Concurrency Checking ใน Swift 5
// 2. แก้ไข Warning ทั้งหมด
// 3. เปลี่ยนเป็น Swift 6 Language Mode

// ========================================
// Step 1: Enable Strict Concurrency
// ========================================
// ใน Package.swift:
// .swiftSettings([.enableExperimentalFeature("StrictConcurrency")])

// ========================================
// รูปแบบ Error ที่พบบ่อยและวิธีแก้
// ========================================

// Error 1: Sendable ของ Class
// ❌ Before:
class DataModel_Before {
    var items: [String] = []
}

func processAsync_Before(model: DataModel_Before) {
    Task {
        // Error: Capture of 'model' with non-sendable type
        // model.items.append("new")
    }
}

// ✅ After Option A: ทำให้เป็น Actor
actor DataModel_After_A {
    var items: [String] = []
    
    func addItem(_ item: String) {
        items.append(item)
    }
}

// ✅ After Option B: ทำให้เป็น @MainActor
@MainActor
class DataModel_After_B {
    var items: [String] = []
    
    func addItem(_ item: String) {
        items.append(item)
    }
}

// ✅ After Option C: Sendable Struct
struct DataModel_After_C: Sendable {
    var items: [String] = []
}

// Error 2: Global Mutable State
// ❌ Before:
// var globalConfig = Config()  // Error ใน Swift 6

// ✅ After Option A:
actor GlobalConfig {
    static let shared = GlobalConfig()
    var maxConnections = 10
    var timeout = 30.0
}

// ✅ After Option B: nonisolated(unsafe)
nonisolated(unsafe) var globalConfigSafe = ImmutableConfig(name: "default", maxCount: 10)

// Error 3: Protocol Conformance
// ❌ Before:
protocol OldDelegate {
    func didFinish()
}

// ✅ After:
protocol NewDelegate: AnyObject, Sendable {
    @MainActor
    func didFinish()
}

// Error 4: Closure Sendability
// ❌ Before:
func executeAsync_Before(work: @escaping () -> Void) {
    Task {
        // Error: Converting non-sendable function value to @Sendable
        // work()
    }
}

// ✅ After:
func executeAsync_After(work: @escaping @Sendable () -> Void) {
    Task {
        work()
    }
}

// Migration Helper: CheckedContinuation สำหรับ Legacy Code
func legacyCallback(completion: @escaping (Result<String, Error>) -> Void) {
    DispatchQueue.global().async {
        completion(.success("data"))
    }
}

// แปลงเป็น async/await
func modernAsync() async throws -> String {
    try await withCheckedThrowingContinuation { continuation in
        legacyCallback { result in
            continuation.resume(with: result)
        }
    }
}
```

---

## 18. แบบฝึกหัดพร้อมเฉลย

### แบบฝึกหัดที่ 1: Typed Throws Chain

**โจทย์**: สร้าง Chain ของ Operations ที่มี Typed Throws ที่แตกต่างกัน

```swift
// เฉลย

// Error Types
enum ValidationError2: Error {
    case tooShort(minLength: Int)
    case tooLong(maxLength: Int)
    case invalidCharacter(Character)
    case empty
}

enum TransformError: Error {
    case overflow
    case underflow
    case divisionByZero
}

// Typed Throws Functions
func validateInput(_ input: String) throws(ValidationError2) -> String {
    guard !input.isEmpty else { throw ValidationError2.empty }
    guard input.count >= 3 else { throw ValidationError2.tooShort(minLength: 3) }
    guard input.count <= 100 else { throw ValidationError2.tooLong(maxLength: 100) }
    
    for char in input {
        guard char.isLetter || char.isNumber || char == "_" else {
            throw ValidationError2.invalidCharacter(char)
        }
    }
    
    return input.lowercased()
}

func parseToNumber(_ input: String) throws(TransformError) -> Int {
    guard let value = Int(input) else {
        throw TransformError.overflow
    }
    return value
}

// Combine ด้วย Result
func process(input: String) -> Result<Int, Error> {
    do {
        let validated = try validateInput(input)
        let number = try parseToNumber(validated)
        return .success(number)
    } catch let error as ValidationError2 {
        return .failure(error)
    } catch let error as TransformError {
        return .failure(error)
    } catch {
        return .failure(error)
    }
}

// ทดสอบ
let testCases = ["123", "", "ab", "123abc!", "99999999999999999999"]
for test in testCases {
    switch process(input: test) {
    case .success(let number):
        print("'\(test)' → \(number)")
    case .failure(let error):
        print("'\(test)' ผิดพลาด: \(error)")
    }
}
```

### แบบฝึกหัดที่ 2: Noncopyable Resource Manager

**โจทย์**: สร้าง Resource Manager ที่ใช้ Noncopyable Types ป้องกัน Double-Free

```swift
// เฉลย

// ระบบ ID ที่ไม่ซ้ำกัน
final class IDGenerator: @unchecked Sendable {
    static let shared = IDGenerator()
    private var counter = 0
    private let lock = NSLock()
    
    func next() -> Int {
        lock.withLock {
            counter += 1
            return counter
        }
    }
}

// Resource Handle - Noncopyable
struct ResourceHandle: ~Copyable {
    let id: Int
    private let name: String
    private var isReleased = false
    
    init(name: String) {
        self.id = IDGenerator.shared.next()
        self.name = name
        print("✅ สร้าง Resource '\(name)' #\(id)")
    }
    
    borrowing func use() {
        print("📖 ใช้ Resource '\(name)' #\(id)")
    }
    
    consuming func release() {
        print("🗑️ ปล่อย Resource '\(name)' #\(id)")
        // deinit จะไม่ถูกเรียกซ้ำ
        discard self
    }
    
    deinit {
        print("🔴 deinit Resource '\(name)' #\(id) - อาจเป็น Memory Leak!")
    }
}

// Resource Pool ด้วย Actor
actor ResourcePool {
    private var available: [String] = ["DB_Connection", "File_Handle", "Network_Socket"]
    private var borrowed: [Int: String] = [:]  // id -> name
    
    func borrow() -> ResourceHandle? {
        guard let name = available.first else {
            print("⚠️ ไม่มี Resource ว่าง")
            return nil
        }
        available.removeFirst()
        
        let handle = ResourceHandle(name: name)
        borrowed[handle.id] = name
        return handle
    }
    
    func returnResource(_ handle: consuming ResourceHandle) {
        let id = handle.id
        if let name = borrowed[id] {
            available.append(name)
            borrowed.removeValue(forKey: id)
        }
        handle.release()
    }
    
    var availableCount: Int { available.count }
    var borrowedCount: Int { borrowed.count }
}

// ใช้งาน
Task {
    let pool = ResourcePool()
    
    print("จำนวน Resource ว่าง: \(await pool.availableCount)")
    
    if let handle = await pool.borrow() {
        handle.use()
        await pool.returnResource(handle)
    }
    
    print("หลังคืน Resource ว่าง: \(await pool.availableCount)")
}
```

---

## 19. Swift Macros Deep Dive

```swift
// ประเภทของ Swift Macros
// 1. Freestanding Expression Macros: #macro()
// 2. Freestanding Declaration Macros: #macro
// 3. Attached Member Macros: @Macro บน Type
// 4. Attached Conformance Macros: เพิ่ม Protocol Conformance
// 5. Attached MemberAttribute Macros: เพิ่ม Attribute ให้ Members
// 6. Attached Accessor Macros: เพิ่ม get/set
// 7. Attached Peer Macros: สร้าง Declaration คู่

// ตัวอย่าง Macro ที่มี Implementation สมบูรณ์
// (จริงๆ ต้องอยู่ใน Separate Swift Package)

// สมมติว่ามี @AutoInit Macro
@attached(member, names: named(init))
macro AutoInit() = #externalMacro(module: "MyMacros", type: "AutoInitMacro")

// ก่อนขยาย:
// @AutoInit
// struct Person {
//     var name: String
//     var age: Int
//     var email: String
// }

// หลังขยาย:
struct PersonExpanded {
    var name: String
    var age: Int
    var email: String
    
    // สร้างโดย @AutoInit
    init(name: String, age: Int, email: String) {
        self.name = name
        self.age = age
        self.email = email
    }
}

// @Builder Macro สำหรับ Builder Pattern
@attached(member, names: arbitrary)
macro Builder() = #externalMacro(module: "MyMacros", type: "BuilderMacro")

// ก่อนขยาย:
// @Builder
// struct RequestConfig {
//     var url: String = ""
//     var method: String = "GET"
//     var timeout: Double = 30.0
// }

// หลังขยาย (จำลอง):
struct RequestConfigBuilt {
    var url: String = ""
    var method: String = "GET"
    var timeout: Double = 30.0
    
    // สร้างโดย @Builder
    func withURL(_ url: String) -> Self {
        var copy = self
        copy.url = url
        return copy
    }
    
    func withMethod(_ method: String) -> Self {
        var copy = self
        copy.method = method
        return copy
    }
    
    func withTimeout(_ timeout: Double) -> Self {
        var copy = self
        copy.timeout = timeout
        return copy
    }
}

// ใช้งาน Builder
let config = RequestConfigBuilt()
    .withURL("https://api.example.com")
    .withMethod("POST")
    .withTimeout(60.0)

print("URL: \(config.url)")
print("Method: \(config.method)")
print("Timeout: \(config.timeout)")

// Macro Expansion Debugging
// ใน Xcode: คลิกขวาที่ Macro → Expand Macro
// ใน Terminal: swift-syntax-macros --expand

// สร้าง Macro สำหรับ Enum Iteration
@attached(member, names: named(allCases))
macro EnumerateAll() = #externalMacro(module: "MyMacros", type: "EnumerateAllMacro")

// หลังขยาย (จำลอง):
enum Direction: CaseIterable {
    case north, south, east, west
    
    // สร้างโดย @EnumerateAll
    static let allDirections: [Direction] = [.north, .south, .east, .west]
    
    var opposite: Direction {
        switch self {
        case .north: return .south
        case .south: return .north
        case .east: return .west
        case .west: return .east
        }
    }
}
```

---

## 20. Concurrency ใน Swift 6 ที่เปลี่ยนแปลง

```swift
// Changes ที่สำคัญใน Swift 6 Concurrency

// 1. Global Actor Isolation สำหรับ Protocol Conformance
@MainActor
protocol UIUpdatable {
    func updateUI()
}

class MyViewController: UIUpdatable {
    // ✅ Swift 6 รู้ว่า method นี้ต้องรันบน MainActor
    func updateUI() {
        print("อัปเดต UI บน Main Thread")
    }
}

// 2. Inferred Sendable สำหรับ Function Types
typealias Handler = @Sendable () -> Void

func register(handler: Handler) {
    // handler เป็น @Sendable โดย Type
    Task {
        handler()  // ✅ ปลอดภัย
    }
}

// 3. Isolated Parameters
actor NetworkManager {
    var baseURL = "https://api.example.com"
    
    // isolated parameter - รัน Synchronously บน Actor
    func configure(isolated actor: NetworkManager = self) {
        actor.baseURL = "https://new.api.example.com"
    }
}

// 4. Preconcurrency Import
// สำหรับ Framework ที่ยังไม่ได้ Migrate ไป Swift 6
// @preconcurrency import OldFramework

// 5. nonisolated(unsafe) สำหรับ Legacy Code
class LegacyClass {
    nonisolated(unsafe) var mutableState: Int = 0
    
    nonisolated func unsafeMutate() {
        mutableState += 1  // Swift 6 จะไม่ Error เพราะเราบอกว่า unsafe
    }
}

// 6. Transferring และ Sending (Swift 6.0)
// Sendable ขั้นสูงสำหรับ Ownership Transfer
func transferData<T: Sendable>(sending value: T, to destination: @escaping @Sendable (T) -> Void) {
    Task {
        destination(value)
    }
}

// 7. Atomic Operations (Swift 6 Stdlib)
// import Synchronization (ใหม่ใน Swift 6)
// let counter = Atomic<Int>(0)
// counter.wrappingAdd(1, ordering: .relaxed)
// let value = counter.load(ordering: .sequentiallyConsistent)
```

---

## 21. ตัวอย่างโปรเจ็กต์ขนาดใหญ่ด้วย Swift 6

```swift
// ระบบจัดการ Task แบบ Actor-based ที่ใช้ Swift 6 Features

// Typed Errors
enum TaskManagerError: Error, Sendable {
    case taskNotFound(id: String)
    case taskAlreadyExists(id: String)
    case invalidStatus(current: TaskStatus, expected: TaskStatus)
    case unauthorized(userId: String)
}

// Sendable Types
enum TaskStatus: Sendable, Equatable {
    case pending
    case inProgress(startedAt: Date)
    case completed(completedAt: Date, result: String)
    case failed(error: String)
    case cancelled
}

struct TaskItem: Sendable, Identifiable {
    let id: String
    let title: String
    let assignedTo: String
    var status: TaskStatus
    let createdAt: Date
    var updatedAt: Date
    
    init(id: String = UUID().uuidString, title: String, assignedTo: String) {
        self.id = id
        self.title = title
        self.assignedTo = assignedTo
        self.status = .pending
        self.createdAt = Date()
        self.updatedAt = Date()
    }
}

// Actor-based Task Manager
actor TaskManager {
    private var tasks: [String: TaskItem] = [:]
    private var observers: [String: [AsyncStream<TaskItem>.Continuation]] = [:]
    
    // Typed Throws สำหรับ Create
    func createTask(
        title: String,
        assignedTo: String
    ) throws(TaskManagerError) -> TaskItem {
        let task = TaskItem(title: title, assignedTo: assignedTo)
        
        guard tasks[task.id] == nil else {
            throw TaskManagerError.taskAlreadyExists(id: task.id)
        }
        
        tasks[task.id] = task
        notifyObservers(task: task)
        return task
    }
    
    // Typed Throws สำหรับ Update
    func startTask(id: String) throws(TaskManagerError) -> TaskItem {
        guard var task = tasks[id] else {
            throw TaskManagerError.taskNotFound(id: id)
        }
        
        guard case .pending = task.status else {
            throw TaskManagerError.invalidStatus(current: task.status, expected: .pending)
        }
        
        task.status = .inProgress(startedAt: Date())
        task.updatedAt = Date()
        tasks[id] = task
        notifyObservers(task: task)
        return task
    }
    
    func completeTask(id: String, result: String) throws(TaskManagerError) -> TaskItem {
        guard var task = tasks[id] else {
            throw TaskManagerError.taskNotFound(id: id)
        }
        
        guard case .inProgress = task.status else {
            throw TaskManagerError.invalidStatus(
                current: task.status,
                expected: .inProgress(startedAt: Date())
            )
        }
        
        task.status = .completed(completedAt: Date(), result: result)
        task.updatedAt = Date()
        tasks[id] = task
        notifyObservers(task: task)
        return task
    }
    
    // Subscribe to Task Updates
    func subscribe(to taskId: String) -> AsyncStream<TaskItem> {
        let (stream, continuation) = AsyncStream.makeStream(of: TaskItem.self)
        
        if observers[taskId] == nil {
            observers[taskId] = []
        }
        observers[taskId]?.append(continuation)
        
        continuation.onTermination = { [taskId] _ in
            Task { [weak self] in
                await self?.removeObserver(taskId: taskId, continuation: continuation)
            }
        }
        
        return stream
    }
    
    private func notifyObservers(task: TaskItem) {
        observers[task.id]?.forEach { $0.yield(task) }
    }
    
    private func removeObserver(
        taskId: String,
        continuation: AsyncStream<TaskItem>.Continuation
    ) {
        observers[taskId]?.removeAll(where: { $0 === continuation as AnyObject })
    }
    
    func getAllTasks() -> [TaskItem] {
        Array(tasks.values)
    }
    
    func getTask(id: String) throws(TaskManagerError) -> TaskItem {
        guard let task = tasks[id] else {
            throw TaskManagerError.taskNotFound(id: id)
        }
        return task
    }
}

// ใช้งาน
Task {
    let manager = TaskManager()
    
    // สร้าง Tasks
    do {
        let task1 = try manager.createTask(title: "พัฒนา Feature A", assignedTo: "สมชาย")
        let task2 = try manager.createTask(title: "ทดสอบ Feature B", assignedTo: "สมหญิง")
        
        print("สร้าง Task สำเร็จ:")
        print("  - \(task1.title) (ID: \(task1.id))")
        print("  - \(task2.title) (ID: \(task2.id))")
        
        // Subscribe ก่อน Update
        let updates = await manager.subscribe(to: task1.id)
        
        // Update Task
        Task {
            for await update in updates {
                print("Task '\(update.title)' เปลี่ยนสถานะ: \(update.status)")
            }
        }
        
        // เริ่ม Task
        let startedTask = try await manager.startTask(id: task1.id)
        print("\nเริ่มทำงาน: \(startedTask.title)")
        
        // จบ Task
        try? await Task.sleep(nanoseconds: 100_000_000)
        let completedTask = try await manager.completeTask(
            id: task1.id,
            result: "Feature A พัฒนาสำเร็จ"
        )
        print("เสร็จสิ้น: \(completedTask.title)")
        
    } catch TaskManagerError.taskNotFound(let id) {
        print("ไม่พบ Task: \(id)")
    } catch TaskManagerError.invalidStatus(let current, let expected) {
        print("สถานะไม่ถูกต้อง: \(current) (ต้องการ \(expected))")
    } catch {
        print("เกิดข้อผิดพลาด: \(error)")
    }
}
```

---

## 22. Strict Concurrency ใน Practice

```swift
// ตัวอย่าง Real-World ที่ต้องปรับสำหรับ Swift 6

// Delegate Pattern ที่ปลอดภัย
@MainActor
protocol SafeDelegate: AnyObject {
    func didReceiveData(_ data: Data)
    func didFail(with error: Error)
}

// Callback-based Service ที่ Migrate เป็น Swift 6
@MainActor
class DataService {
    weak var delegate: (any SafeDelegate)?
    private var isLoading = false
    
    func loadData(from url: URL) {
        guard !isLoading else { return }
        isLoading = true
        
        Task { [weak self] in
            do {
                let (data, _) = try await URLSession.shared.data(from: url)
                // กลับมา MainActor เพราะ DataService เป็น @MainActor
                await MainActor.run {
                    self?.isLoading = false
                    self?.delegate?.didReceiveData(data)
                }
            } catch {
                await MainActor.run {
                    self?.isLoading = false
                    self?.delegate?.didFail(with: error)
                }
            }
        }
    }
}

// Notification Center ที่ Thread-Safe
extension NotificationCenter {
    func observe<T: Sendable>(
        _ name: Notification.Name,
        transform: @escaping @Sendable (Notification) -> T?
    ) -> AsyncStream<T> {
        AsyncStream { continuation in
            let observer = self.addObserver(
                forName: name,
                object: nil,
                queue: nil
            ) { notification in
                if let value = transform(notification) {
                    continuation.yield(value)
                }
            }
            
            continuation.onTermination = { _ in
                NotificationCenter.default.removeObserver(observer)
            }
        }
    }
}

// ใช้งาน
Task {
    // Stream ของ Keyboard Notifications
    let keyboardStream = NotificationCenter.default.observe(
        UIResponder.keyboardWillShowNotification
    ) { notification -> CGFloat? in
        let frame = notification.userInfo?[UIResponder.keyboardFrameEndUserInfoKey] as? CGRect
        return frame?.height
    }
    
    for await height in keyboardStream {
        print("Keyboard height: \(height)")
    }
}

// Stub Types
enum UIResponder {
    static let keyboardWillShowNotification = Notification.Name("keyboard_will_show")
    static let keyboardFrameEndUserInfoKey = "keyboard_frame"
}

struct CGRect {
    var height: CGFloat
}

typealias CGFloat = Double
```

---

## 23. สรุป Swift 6 Features

### ตารางสรุปฟีเจอร์ใหม่

| Feature | คำอธิบาย | ประโยชน์ |
|---------|----------|---------|
| Strict Concurrency | ตรวจ Data Race ตอน Compile | ป้องกัน Bug ยากๆ |
| Typed Throws | ระบุ Error Type ใน Signature | Code ชัดเจน, ไม่ต้อง Cast |
| Noncopyable Types | ป้องกัน Copy โดยไม่ตั้งใจ | Performance, Safety |
| ~Copyable Constraint | Generic สำหรับ Noncopyable | ยืดหยุ่น |
| consuming/borrowing | Explicit Ownership | Performance, Clarity |
| Parameter Packs | Variadic Generics | Type-Safe Variadic |
| Pack Iteration | for-in บน Packs | Easy Pack Processing |
| Expression Macros | Code Generation ตอน Compile | Less Boilerplate |
| Attached Macros | เพิ่ม Members/Conformance | Powerful Automation |
| @Observable | Modern Observable | Better than ObservableObject |
| any vs some | Explicit Existential | Code ชัดเจน |
| Global Actor | Isolation ระดับ Module | Thread Safety |

### Checklist สำหรับ Migration ไป Swift 6

1. ✅ เปิด Strict Concurrency Checking
2. ✅ แก้ไข Sendable Issues ทั้งหมด
3. ✅ เพิ่ม `any` หน้า Existential Types
4. ✅ แก้ไข Global Mutable State
5. ✅ Update Protocol Conformances
6. ✅ ใช้ Typed Throws แทน untyped
7. ✅ พิจารณาใช้ Noncopyable สำหรับ Resources
8. ✅ Update Macro Dependencies
9. ✅ Test อย่างละเอียดหลัง Migration

### Best Practices ใน Swift 6

1. **Prefer Actors** สำหรับ Shared Mutable State
2. **Use Typed Throws** เพื่อความชัดเจน
3. **Leverage @Observable** แทน ObservableObject
4. **Consider Noncopyable** สำหรับ Unique Resources
5. **Write Sendable Types** เพื่อความปลอดภัย
6. **Use Parameter Packs** แทน Overloading
7. **Create Custom Macros** เพื่อลด Boilerplate

---

## ขั้นตอนต่อไป

หลังจากเรียน Swift 6 แล้ว ขั้นตอนต่อไปที่แนะนำ:

- **Part 55**: Building Real-World Apps - นำทุก Concept มารวมกัน
- **Part 56**: Performance Optimization - Profiling และ Memory Management ขั้นสูง
- **Part 57**: Testing Strategies - Unit, Integration, UI Testing
- **Part 58**: Swift Package Manager - สร้างและแจกจ่าย Libraries

---

*จบ Part 54: Swift 6 Features*
