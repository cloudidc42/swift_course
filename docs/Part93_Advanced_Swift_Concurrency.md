# Part 93: Advanced Swift Concurrency (การทำงานแบบ Concurrent ขั้นสูงใน Swift)

## บทนำ

Swift Concurrency ถูกแนะนำใน Swift 5.5 และพัฒนาอย่างต่อเนื่องจนถึง Swift 6 ซึ่งเพิ่ม strict concurrency checking บทนี้จะเจาะลึกถึงกลไกภายในของ Swift Concurrency, Custom Executors, Distributed Actors, และ pattern ขั้นสูงต่างๆ ที่นักพัฒนา iOS ระดับ Senior ควรรู้

---

## 1. Swift Concurrency Model Deep Dive (โมเดล Concurrency ของ Swift เชิงลึก)

### 1.1 Cooperative Thread Pool (Thread Pool แบบ Cooperative)

Swift Concurrency ไม่ใช้ thread pool แบบเดิมที่แต่ละ task จะครอบครอง thread ตลอดเวลา แต่ใช้ **Cooperative Thread Pool** ซึ่งมีจำนวน thread เท่ากับจำนวน CPU cores (logical cores)

```swift
// ตัวอย่างการสังเกต cooperative thread pool
import Foundation

// Swift runtime จัดการ thread pool อัตโนมัติ
// จำนวน threads = จำนวน CPU cores (ปกติ)
// ไม่เหมือน GCD ที่อาจสร้าง thread ได้มากกว่า cores

func demonstrateCooperativePool() async {
    // Task เหล่านี้จะถูก schedule บน cooperative pool
    async let task1 = heavyComputation(id: 1)
    async let task2 = heavyComputation(id: 2)
    async let task3 = heavyComputation(id: 3)
    
    let results = await [task1, task2, task3]
    print("Results: \(results)")
}

func heavyComputation(id: Int) async -> Int {
    // Suspension point ทำให้ thread ว่างสำหรับ task อื่น
    await Task.yield()
    var sum = 0
    for i in 0..<1_000_000 {
        sum += i
    }
    return sum + id
}
```

### หลักการสำคัญของ Cooperative Thread Pool:

1. **Thread Count Limit**: จำนวน thread จำกัดที่ CPU cores เพื่อป้องกัน thread explosion
2. **Cooperative Scheduling**: Tasks ยอมสละ thread ที่ suspension points
3. **No Blocking**: ไม่ควร block thread ด้วย synchronous operations
4. **Work Stealing**: Threads ที่ว่างจะ "steal" work จาก queues ของ thread อื่น

```swift
// ❌ อย่าทำ: Blocking thread ใน async context
func badPractice() async {
    // Thread.sleep blocks the thread completely!
    Thread.sleep(forTimeInterval: 1.0)
}

// ✅ ทำแบบนี้: ใช้ async sleep ที่ยอมสละ thread
func goodPractice() async {
    // Task.sleep suspends the task, freeing the thread
    try? await Task.sleep(for: .seconds(1))
}

// ❌ อย่าทำ: Semaphore blocking
func badSemaphore() async {
    let sem = DispatchSemaphore(value: 0)
    DispatchQueue.global().async {
        // some work
        sem.signal()
    }
    sem.wait() // BLOCKS THE THREAD - อันตราย!
}
```

### 1.2 Thread Hopping และ Suspension Points

**Thread hopping** คือปรากฏการณ์ที่ async function อาจกลับมา resume บน thread ที่ต่างออกไปหลังจาก await

```swift
import Foundation

class ThreadHoppingDemo {
    func demonstrateHopping() async {
        let thread1 = Thread.current
        print("Before await: \(thread1.name ?? "unknown")")
        
        // Suspension point - thread อาจเปลี่ยน
        await Task.yield()
        
        let thread2 = Thread.current
        print("After await: \(thread2.name ?? "unknown")")
        
        // thread1 และ thread2 อาจเป็น thread ต่างกัน!
        // นี่คือเหตุผลที่ Actor isolation สำคัญมาก
    }
    
    // Actor จะทำให้แน่ใจว่า code ทำงานบน executor ของมัน
    actor SafeCounter {
        var count = 0
        
        func increment() {
            count += 1
            // code ทั้งหมดใน actor method ทำงานใน actor's serial executor
            // ไม่ว่าจะ await กี่ครั้ง
        }
    }
}
```

### Suspension Points ที่ควรรู้:

```swift
// 1. await expression
let result = await someAsyncFunction()

// 2. async let binding
async let value = computeAsync()

// 3. for await loop
for await item in asyncSequence { }

// 4. Task.yield() - explicit yield
await Task.yield()

// 5. Task.sleep
try await Task.sleep(for: .seconds(1))

// 6. withTaskCancellationHandler
await withTaskCancellationHandler {
    // work
} onCancel: {
    // cleanup
}
```

### 1.3 Continuations และ Runtime

Continuations เป็นกลไกที่ใช้เชื่อม callback-based API กับ async/await

```swift
import Foundation

// Unsafe Continuation - ต้องเรียก resume ครั้งเดียวเสมอ
func fetchDataWithCompletion() async throws -> Data {
    return try await withUnsafeThrowingContinuation { continuation in
        URLSession.shared.dataTask(with: URL(string: "https://api.example.com/data")!) { data, response, error in
            if let error = error {
                continuation.resume(throwing: error)
            } else if let data = data {
                continuation.resume(returning: data)
            } else {
                continuation.resume(throwing: URLError(.badServerResponse))
            }
        }.resume()
    }
}

// Checked Continuation - มี runtime checking (ปลอดภัยกว่า)
func fetchDataChecked() async throws -> Data {
    return try await withCheckedThrowingContinuation { continuation in
        // ถ้าเรียก resume มากกว่าหนึ่งครั้ง หรือไม่เรียกเลย
        // จะมี runtime warning/crash
        URLSession.shared.dataTask(with: URL(string: "https://api.example.com/data")!) { data, response, error in
            if let error = error {
                continuation.resume(throwing: error)
            } else if let data = data {
                continuation.resume(returning: data)
            } else {
                continuation.resume(throwing: URLError(.badServerResponse))
            }
        }.resume()
    }
}

// ตัวอย่างการ wrap delegate-based API
class LocationFetcher: NSObject, CLLocationManagerDelegate {
    private var continuation: CheckedContinuation<CLLocation, Error>?
    private let manager = CLLocationManager()
    
    func getCurrentLocation() async throws -> CLLocation {
        return try await withCheckedThrowingContinuation { continuation in
            self.continuation = continuation
            manager.delegate = self
            manager.requestLocation()
        }
    }
    
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        continuation?.resume(returning: locations.first!)
        continuation = nil
    }
    
    func locationManager(_ manager: CLLocationManager, didFailWithError error: Error) {
        continuation?.resume(throwing: error)
        continuation = nil
    }
}
```

### 1.4 Executor Protocol

Executor เป็น protocol ที่กำหนดวิธีการ schedule และ run jobs

```swift
// Executor protocol (Swift standard library)
// public protocol Executor: AnyObject, Sendable {
//     func enqueue(_ job: consuming ExecutorJob)
// }

// SerialExecutor - runs jobs one at a time
// public protocol SerialExecutor: Executor {
//     func asUnownedSerialExecutor() -> UnownedSerialExecutor
// }

// การใช้งาน MainActor executor
@MainActor
func updateUI() {
    // ทำงานบน main thread เสมอ
    // MainActor ใช้ DispatchQueue.main เป็น executor
}

// Custom executor example
final class MySerialExecutor: SerialExecutor {
    private let queue: DispatchQueue
    
    init(label: String) {
        self.queue = DispatchQueue(label: label)
    }
    
    func enqueue(_ job: consuming ExecutorJob) {
        let unownedJob = UnownedJob(job)
        queue.async {
            unownedJob.runSynchronously(on: self.asUnownedSerialExecutor())
        }
    }
    
    func asUnownedSerialExecutor() -> UnownedSerialExecutor {
        UnownedSerialExecutor(ordinary: self)
    }
}
```

---

## 2. Actor Isolation Rules (กฎของ Actor Isolation)

### 2.1 Actor Reentrancy ในเชิงลึก

Actor ใน Swift เป็น **reentrant** หมายความว่า actor สามารถรับ message ใหม่ได้ในระหว่างที่กำลัง await อยู่

```swift
actor BankAccount {
    var balance: Double = 1000.0
    
    // ❌ Bug ที่เกิดจาก reentrancy
    func withdraw(amount: Double) async throws {
        guard balance >= amount else {
            throw BankError.insufficientFunds
        }
        
        // Suspension point! Actor สามารถรับ message อื่นได้ที่นี่
        await processWithdrawal(amount: amount)
        
        // balance อาจเปลี่ยนแปลงไปแล้วจาก withdrawal อื่น!
        balance -= amount // อาจทำให้ balance เป็นลบได้
    }
    
    // ✅ วิธีที่ถูกต้อง: ตรวจสอบ state อีกครั้งหลัง await
    func withdrawSafe(amount: Double) async throws {
        guard balance >= amount else {
            throw BankError.insufficientFunds
        }
        
        // จอง amount ก่อน
        let reserved = amount
        balance -= reserved // หัก balance ก่อน suspend
        
        do {
            await processWithdrawal(amount: reserved)
        } catch {
            // Rollback ถ้าเกิด error
            balance += reserved
            throw error
        }
    }
    
    private func processWithdrawal(amount: Double) async {
        // simulate network call
        try? await Task.sleep(for: .milliseconds(100))
    }
}

enum BankError: Error {
    case insufficientFunds
}

// ตัวอย่างที่แสดง reentrancy ชัดเจน
actor Counter {
    var value = 0
    
    func incrementWithDelay() async {
        let currentValue = value
        print("Read value: \(currentValue)")
        
        // Suspension point - value อาจเปลี่ยนระหว่างนี้
        try? await Task.sleep(for: .milliseconds(10))
        
        // value อาจไม่ใช่ currentValue + 1 ถ้ามี concurrent increments
        value = currentValue + 1
        print("Set value to: \(value)")
    }
    
    // ✅ Atomic increment
    func incrementAtomic() {
        value += 1 // ไม่มี suspension - ปลอดภัย
    }
}
```

### 2.2 nonisolated Keyword

`nonisolated` ใช้เพื่อบอกว่า method นั้นไม่ต้องการ actor isolation

```swift
actor DatabaseManager {
    var connectionString: String
    var isConnected: Bool = false
    
    init(connectionString: String) {
        self.connectionString = connectionString
    }
    
    // nonisolated method - เรียกได้จาก non-async context
    nonisolated var description: String {
        // ไม่สามารถเข้าถึง mutable state ของ actor ได้
        // connectionString เป็น let จึงเข้าถึงได้
        return "DatabaseManager(\(connectionString))"
    }
    
    // nonisolated computed property
    nonisolated var debugInfo: String {
        return "DB Manager v1.0"
    }
    
    // nonisolated func
    nonisolated func logMessage(_ message: String) {
        print("[DB] \(message)")
        // ไม่สามารถเข้าถึง isConnected ได้เพราะมัน mutable
    }
    
    func connect() async {
        isConnected = true
        logMessage("Connected to \(connectionString)")
    }
}

// Protocol conformance กับ nonisolated
protocol Describable {
    var description: String { get }
}

actor ServiceActor: Describable {
    let name: String
    var status: String = "idle"
    
    init(name: String) {
        self.name = name
    }
    
    // Protocol requirement ต้องเป็น nonisolated เพื่อให้ conform ได้
    // โดยไม่ต้องเป็น async
    nonisolated var description: String {
        return "Service: \(name)"
    }
}
```

### 2.3 @preconcurrency Attribute

`@preconcurrency` ใช้สำหรับ code ที่เขียนก่อนที่ Swift Concurrency จะมีการตรวจสอบ strict

```swift
import Foundation

// @preconcurrency import - suppress warnings สำหรับ module เก่า
@preconcurrency import SomeLegacyFramework

// @preconcurrency protocol - สำหรับ protocol ที่เขียนก่อน concurrency era
@preconcurrency protocol LegacyDelegate: AnyObject {
    func didComplete(result: String)
}

// การใช้ @preconcurrency กับ class
@preconcurrency
class LegacyService: NSObject {
    var delegate: LegacyDelegate?
    
    func start() {
        DispatchQueue.global().async {
            // งานที่ทำใน background
            let result = "completed"
            
            DispatchQueue.main.async {
                // @preconcurrency จะ suppress Sendable warning ที่นี่
                self.delegate?.didComplete(result: result)
            }
        }
    }
}

// การ migrate จาก legacy code
// ขั้นตอนที่ 1: ใช้ @preconcurrency เพื่อ suppress warnings
// ขั้นตอนที่ 2: ค่อยๆ migrate ไปใช้ actor/Sendable
// ขั้นตอนที่ 3: ลบ @preconcurrency ออก

actor ModernService {
    @preconcurrency
    var legacyCallback: ((String) -> Void)?
    
    func performWork() async {
        let result = await computeResult()
        // @preconcurrency ช่วย suppress Sendable check สำหรับ callback
        legacyCallback?(result)
    }
    
    private func computeResult() async -> String {
        try? await Task.sleep(for: .seconds(1))
        return "result"
    }
}
```

### 2.4 Sendable Protocol และการตรวจสอบ

`Sendable` เป็น protocol ที่ระบุว่า type นั้นปลอดภัยในการส่งระหว่าง concurrency domains

```swift
// Types ที่เป็น Sendable โดยอัตโนมัติ:
// - Value types (struct, enum) ที่ properties ทั้งหมดเป็น Sendable
// - Actor types
// - @MainActor classes
// - Classes ที่ final และ immutable

// ✅ Auto-Sendable struct
struct Point: Sendable {
    let x: Double
    let y: Double
}

// ✅ Auto-Sendable enum
enum Status: Sendable {
    case active
    case inactive
    case pending(String)
}

// ❌ ไม่ใช่ Sendable - มี mutable state
class MutableObject {
    var value = 0
}

// ✅ Sendable class - ต้อง final และ immutable
final class ImmutableRecord: Sendable {
    let id: Int
    let name: String
    
    init(id: Int, name: String) {
        self.id = id
        self.name = name
    }
}

// @unchecked Sendable - บอกว่าเรา handle thread safety เอง
final class ThreadSafeCache: @unchecked Sendable {
    private var storage: [String: Any] = [:]
    private let lock = NSLock()
    
    func set(_ value: Any, forKey key: String) {
        lock.lock()
        defer { lock.unlock() }
        storage[key] = value
    }
    
    func get(forKey key: String) -> Any? {
        lock.lock()
        defer { lock.unlock() }
        return storage[key]
    }
}

// Swift 6 Sendable checking
// ใน Swift 6 mode การส่ง non-Sendable types ข้าม actor boundaries จะเป็น error

@MainActor
class ViewModel {
    var items: [String] = []
    
    func loadData() async {
        // ✅ String เป็น Sendable - ปลอดภัย
        let data = await fetchStrings()
        items = data
    }
    
    private func fetchStrings() async -> [String] {
        // simulate fetch
        try? await Task.sleep(for: .seconds(1))
        return ["item1", "item2"]
    }
}

// Sendable closure
func performAsync(operation: @Sendable () async -> Void) async {
    await operation()
}

// ตัวอย่างการใช้งาน
func example() async {
    let value = 42 // Int เป็น Sendable
    
    await performAsync {
        // ✅ value ถูก capture และส่งผ่าน Sendable closure
        print("Value: \(value)")
    }
}
```

---

## 3. Custom Executors (Swift 5.9+)

### 3.1 SerialExecutor Protocol

```swift
import Foundation

// Custom Serial Executor บน DispatchQueue
final class DispatchQueueExecutor: SerialExecutor {
    let queue: DispatchQueue
    
    init(label: String, qos: DispatchQoS = .default) {
        self.queue = DispatchQueue(label: label, qos: qos)
    }
    
    func enqueue(_ job: consuming ExecutorJob) {
        let unownedJob = UnownedJob(job)
        queue.async {
            unownedJob.runSynchronously(on: self.asUnownedSerialExecutor())
        }
    }
    
    func asUnownedSerialExecutor() -> UnownedSerialExecutor {
        return UnownedSerialExecutor(ordinary: self)
    }
}

// ทดสอบ custom executor
actor DataProcessor {
    private let executor = DispatchQueueExecutor(
        label: "com.app.dataprocessor",
        qos: .userInitiated
    )
    
    nonisolated var unownedExecutor: UnownedSerialExecutor {
        executor.asUnownedSerialExecutor()
    }
    
    var processedCount = 0
    
    func process(data: [Int]) async -> [Int] {
        processedCount += data.count
        return data.map { $0 * 2 }
    }
}

// ทดสอบ
func testCustomExecutor() async {
    let processor = DataProcessor()
    let result = await processor.process(data: [1, 2, 3, 4, 5])
    let count = await processor.processedCount
    print("Processed \(count) items: \(result)")
}
```

### 3.2 Actor กับ Custom Executor

```swift
import Foundation

// Priority Executor - รัน high priority jobs ก่อน
final class PriorityExecutor: SerialExecutor {
    private let highPriorityQueue = DispatchQueue(
        label: "com.app.high",
        qos: .userInteractive
    )
    private let normalQueue = DispatchQueue(
        label: "com.app.normal",
        qos: .default
    )
    
    func enqueue(_ job: consuming ExecutorJob) {
        let unownedJob = UnownedJob(job)
        
        // ตรวจสอบ priority ของ job
        if job.priority >= TaskPriority.high {
            highPriorityQueue.async {
                unownedJob.runSynchronously(on: self.asUnownedSerialExecutor())
            }
        } else {
            normalQueue.async {
                unownedJob.runSynchronously(on: self.asUnownedSerialExecutor())
            }
        }
    }
    
    func asUnownedSerialExecutor() -> UnownedSerialExecutor {
        UnownedSerialExecutor(ordinary: self)
    }
}

// Actor ที่ใช้ PriorityExecutor
actor UIUpdateActor {
    private static let executor = PriorityExecutor()
    
    nonisolated var unownedExecutor: UnownedSerialExecutor {
        Self.executor.asUnownedSerialExecutor()
    }
    
    var pendingUpdates: [String] = []
    
    func queueUpdate(_ update: String, isUrgent: Bool) async {
        if isUrgent {
            // สร้าง task ด้วย high priority
            Task(priority: .high) {
                await self.applyUpdate(update)
            }
        } else {
            pendingUpdates.append(update)
        }
    }
    
    private func applyUpdate(_ update: String) {
        print("Applying update: \(update)")
        pendingUpdates.removeAll { $0 == update }
    }
}
```

### 3.3 MainActor Internals

```swift
// MainActor คือ global actor ที่ใช้ DispatchQueue.main
// @MainActor เทียบเท่ากับ @UIActor ใน UIKit context

@MainActor
class ViewController: UIViewController {
    var label: UILabel!
    
    // ทุก method ใน @MainActor class ทำงานบน main thread
    func updateLabel(text: String) {
        label.text = text
    }
    
    // nonisolated method สามารถเรียกจาก background ได้
    nonisolated func formatText(_ text: String) -> String {
        return text.uppercased()
    }
}

// การใช้ @MainActor กับ closure
func fetchAndDisplay() async {
    let data = await fetchData()
    
    // ต้องใช้ @MainActor annotation หรือ MainActor.run
    await MainActor.run {
        // ทำงานบน main thread
        print("Updating UI with: \(data)")
    }
}

// หรือใช้ @MainActor annotation
@MainActor
func updateUI(with data: String) {
    print("On main thread: \(data)")
}

// ตัวอย่างการ hop ไป MainActor
actor BackgroundProcessor {
    func processAndUpdateUI(items: [String]) async {
        // ทำงานบน background (actor's executor)
        let processed = items.map { $0.uppercased() }
        
        // Hop ไป MainActor เพื่ออัพเดต UI
        await MainActor.run {
            // ทำงานบน main thread
            print("Updating UI: \(processed)")
        }
        
        // กลับมาทำงานบน actor's executor
        print("Back on actor")
    }
}

func fetchData() async -> String {
    try? await Task.sleep(for: .seconds(1))
    return "fetched data"
}
```

### 3.4 Hooking เข้า Existing Dispatch Queues

```swift
import Foundation

// Wrapper สำหรับ DispatchQueue เป็น SerialExecutor
final class DispatchQueueSerialExecutor: SerialExecutor {
    private let queue: DispatchQueue
    
    // Wrap existing dispatch queue
    init(queue: DispatchQueue) {
        self.queue = queue
    }
    
    func enqueue(_ job: consuming ExecutorJob) {
        let unownedJob = UnownedJob(job)
        queue.async {
            unownedJob.runSynchronously(on: self.asUnownedSerialExecutor())
        }
    }
    
    func asUnownedSerialExecutor() -> UnownedSerialExecutor {
        UnownedSerialExecutor(ordinary: self)
    }
}

// Actor ที่ใช้ existing queue (เช่น queue จาก legacy code)
actor LegacyBridgeActor {
    private let legacyQueue: DispatchQueue
    private let executor: DispatchQueueSerialExecutor
    
    init(legacyQueue: DispatchQueue) {
        self.legacyQueue = legacyQueue
        self.executor = DispatchQueueSerialExecutor(queue: legacyQueue)
    }
    
    nonisolated var unownedExecutor: UnownedSerialExecutor {
        executor.asUnownedSerialExecutor()
    }
    
    var data: [String] = []
    
    func append(_ item: String) {
        data.append(item)
    }
}

// ตัวอย่างการใช้งาน
func bridgeLegacyCode() async {
    // ใช้ queue เดิมที่มีอยู่
    let existingQueue = DispatchQueue(label: "com.legacy.queue")
    
    let actor = LegacyBridgeActor(legacyQueue: existingQueue)
    
    await actor.append("item1")
    await actor.append("item2")
    
    let data = await actor.data
    print("Data: \(data)")
    
    // Legacy code ยังสามารถใช้ queue ได้ตามปกติ
    existingQueue.async {
        print("Still running on legacy queue")
    }
}
```

---

## 4. Distributed Actors (Swift 5.7+)

### 4.1 Distributed Actor Isolation

Distributed Actors ขยาย Actor model ไปยัง distributed systems ที่ actors อาจอยู่ใน process ต่างกันหรือบน machine ต่างกัน

```swift
import Distributed

// Distributed actor ต้องระบุ ActorSystem
distributed actor DistributedCounter {
    var count: Int = 0
    
    // distributed func - สามารถเรียกได้จาก remote
    distributed func increment() {
        count += 1
    }
    
    distributed func getCount() -> Int {
        return count
    }
    
    // ฟังก์ชัน non-distributed - local only
    func resetLocally() {
        count = 0
    }
}

// การใช้งาน distributed actor
// (ต้องมี concrete ActorSystem implementation)
```

### 4.2 ActorSystem Protocol

```swift
import Distributed

// ActorSystem protocol กำหนด infrastructure สำหรับ distributed actors
// public protocol DistributedActorSystem: Sendable {
//     associatedtype ActorID: Sendable & Hashable & Codable
//     associatedtype InvocationEncoder: DistributedTargetInvocationEncoder
//     associatedtype InvocationDecoder: DistributedTargetInvocationDecoder
//     associatedtype ResultHandler: DistributedTargetInvocationResultHandler
//     associatedtype SerializationRequirement
//     
//     func resolve<Act>(id: ActorID, as actorType: Act.Type) throws -> Act?
//     func assignID<Act>(_ actorType: Act.Type) -> ActorID
//     func actorReady<Act>(_ actor: Act)
//     func resignID(_ id: ActorID)
//     func makeInvocationEncoder() -> InvocationEncoder
// }

// ตัวอย่าง simple local ActorSystem สำหรับทดสอบ
// (ใน production จะใช้ framework เช่น swift-distributed-actors)
struct LocalActorSystem: DistributedActorSystem {
    typealias ActorID = Int
    typealias InvocationEncoder = LocalInvocationEncoder
    typealias InvocationDecoder = LocalInvocationDecoder
    typealias ResultHandler = LocalResultHandler
    typealias SerializationRequirement = Codable
    
    static var idCounter = 0
    
    func resolve<Act>(id: ActorID, as actorType: Act.Type) throws -> Act? 
    where Act: DistributedActor {
        return nil // local system ไม่ต้อง resolve remote
    }
    
    func assignID<Act>(_ actorType: Act.Type) -> ActorID 
    where Act: DistributedActor {
        Self.idCounter += 1
        return Self.idCounter
    }
    
    func actorReady<Act>(_ actor: Act) where Act: DistributedActor {}
    
    func resignID(_ id: ActorID) {}
    
    func makeInvocationEncoder() -> LocalInvocationEncoder {
        LocalInvocationEncoder()
    }
}
```

### 4.3 Sample Distributed Counter

```swift
import Distributed

// Distributed counter protocol
protocol CounterProtocol: DistributedActor {
    distributed func increment(by amount: Int) async throws
    distributed func decrement(by amount: Int) async throws
    distributed func getValue() async throws -> Int
    distributed func reset() async throws
}

// Implementation
distributed actor RemoteCounter: CounterProtocol {
    private var value: Int = 0
    
    distributed func increment(by amount: Int) {
        value += amount
        print("Incremented by \(amount), new value: \(value)")
    }
    
    distributed func decrement(by amount: Int) {
        value -= amount
        print("Decremented by \(amount), new value: \(value)")
    }
    
    distributed func getValue() -> Int {
        return value
    }
    
    distributed func reset() {
        value = 0
        print("Counter reset")
    }
}

// Distributed actor กับ error handling
distributed actor ReliableService {
    enum ServiceError: Error {
        case unavailable
        case timeout
        case invalidInput(String)
    }
    
    var isAvailable: Bool = true
    
    distributed func processRequest(_ input: String) async throws -> String {
        guard isAvailable else {
            throw ServiceError.unavailable
        }
        
        guard !input.isEmpty else {
            throw ServiceError.invalidInput("Input cannot be empty")
        }
        
        // simulate processing
        try await Task.sleep(for: .milliseconds(100))
        
        return "Processed: \(input.uppercased())"
    }
}
```

### 4.4 Use Cases สำหรับ Distributed Actors

```swift
import Distributed

// Use Case 1: Microservices Architecture
distributed actor UserService {
    distributed func getUser(id: String) async throws -> UserData
    distributed func updateUser(_ user: UserData) async throws
    distributed func deleteUser(id: String) async throws
}

struct UserData: Codable, Sendable {
    let id: String
    let name: String
    let email: String
}

// Use Case 2: Game Server
distributed actor GameRoom {
    distributed func join(player: String) async throws -> Bool
    distributed func leave(player: String) async throws
    distributed func sendMove(from player: String, move: GameMove) async throws
    distributed func getState() async throws -> GameState
}

struct GameMove: Codable, Sendable {
    let position: (Int, Int)
    let timestamp: Date
}

struct GameState: Codable, Sendable {
    let players: [String]
    let board: [[Int]]
    let currentTurn: String
}

// Use Case 3: Distributed Computation
distributed actor ComputeNode {
    distributed func compute(chunk: DataChunk) async throws -> ComputeResult
    distributed func getCapacity() async throws -> Int
    distributed func ping() async throws -> Bool
}

struct DataChunk: Codable, Sendable {
    let id: Int
    let data: [Double]
}

struct ComputeResult: Codable, Sendable {
    let chunkId: Int
    let result: Double
    let processingTime: Double
}
```

---

## 5. Structured vs Unstructured Concurrency

### 5.1 Task Hierarchy และ Cancellation Propagation

```swift
// Structured Concurrency - Task hierarchy
func structuredExample() async throws {
    // Parent task
    print("Parent task started")
    
    // Child tasks (structured)
    async let child1 = fetchData(id: 1)
    async let child2 = fetchData(id: 2)
    
    // Parent รอ children ทั้งหมด
    let results = try await [child1, child2]
    print("All children completed: \(results)")
}
// ถ้า parent ถูก cancel, children ทั้งหมดจะถูก cancel ด้วย

// Task cancellation propagation
func cancellationExample() async throws {
    let parentTask = Task {
        do {
            // นี่คือ parent task
            async let subtask1 = longRunningTask(name: "Task 1")
            async let subtask2 = longRunningTask(name: "Task 2")
            
            let results = try await [subtask1, subtask2]
            print("Results: \(results)")
        } catch is CancellationError {
            print("Parent task was cancelled")
        }
    }
    
    // Cancel หลังจาก 1 วินาที
    try await Task.sleep(for: .seconds(1))
    parentTask.cancel()
    
    // รอให้ task cleanup เสร็จ
    await parentTask.value
}

func longRunningTask(name: String) async throws -> String {
    print("\(name) started")
    defer { print("\(name) ended") }
    
    for i in 0..<10 {
        // ตรวจสอบ cancellation ทุก iteration
        try Task.checkCancellation()
        
        print("\(name) step \(i)")
        try await Task.sleep(for: .milliseconds(200))
    }
    
    return "\(name) completed"
}

func fetchData(id: Int) async throws -> String {
    try await Task.sleep(for: .milliseconds(100))
    return "data-\(id)"
}
```

### 5.2 Detached Tasks

```swift
// Detached tasks ไม่มี parent - ไม่ได้รับ cancellation/priority จาก parent
func detachedTaskExample() async {
    let currentPriority = Task.currentPriority
    print("Current priority: \(currentPriority)")
    
    // Structured task - inherit parent priority และ cancellation
    let structuredTask = Task {
        print("Structured task priority: \(Task.currentPriority)")
        // Priority เดียวกับ parent
    }
    
    // Detached task - ไม่ inherit priority หรือ cancellation
    let detachedTask = Task.detached(priority: .background) {
        print("Detached task priority: \(Task.currentPriority)")
        // Priority ที่ระบุ (.background) ไม่ใช่ parent priority
        
        // ไม่สามารถเข้าถึง task-local values จาก parent
        // ไม่รับ cancellation จาก parent
    }
    
    await structuredTask.value
    await detachedTask.value
}

// Use case สำหรับ Detached Tasks
class AnalyticsService {
    static func logEvent(_ event: String) {
        // Fire-and-forget analytics logging
        // ไม่ต้องรอผล และไม่ต้องการ cancel เมื่อ caller ถูก cancel
        Task.detached(priority: .background) {
            await Self.sendToServer(event: event)
        }
    }
    
    private static func sendToServer(event: String) async {
        // simulate network call
        try? await Task.sleep(for: .seconds(1))
        print("Event logged: \(event)")
    }
}
```

### 5.3 Task Groups vs async let

```swift
// async let - เหมาะเมื่อรู้จำนวน tasks ล่วงหน้า
func parallelFetchWithAsyncLet() async throws -> (users: [User], posts: [Post]) {
    async let users = fetchUsers()
    async let posts = fetchPosts()
    
    // ทั้งสองทำงานพร้อมกัน, await พร้อมกัน
    return try await (users, posts)
}

// TaskGroup - เหมาะเมื่อจำนวน tasks ไม่แน่นอน หรือต้องการ dynamic
func parallelFetchWithTaskGroup(ids: [Int]) async throws -> [User] {
    return try await withThrowingTaskGroup(of: User.self) { group in
        for id in ids {
            group.addTask {
                return try await fetchUser(id: id)
            }
        }
        
        var users: [User] = []
        for try await user in group {
            users.append(user)
        }
        return users
    }
}

// DiscardingTaskGroup (Swift 5.9+) - สำหรับ fire-and-forget
func processItemsConcurrently(items: [String]) async throws {
    try await withThrowingDiscardingTaskGroup { group in
        for item in items {
            group.addTask {
                try await processItem(item)
            }
        }
        // ไม่ต้อง collect results
    }
}

// TaskGroup กับ rate limiting
func rateLimitedFetch(urls: [URL], maxConcurrent: Int) async throws -> [Data] {
    var results: [Data] = Array(repeating: Data(), count: urls.count)
    
    try await withThrowingTaskGroup(of: (Int, Data).self) { group in
        var inFlight = 0
        var nextIndex = 0
        
        // เริ่ม tasks แรก
        while inFlight < maxConcurrent && nextIndex < urls.count {
            let index = nextIndex
            let url = urls[index]
            group.addTask {
                let (data, _) = try await URLSession.shared.data(from: url)
                return (index, data)
            }
            inFlight += 1
            nextIndex += 1
        }
        
        // Collect results และ add tasks ใหม่
        for try await (index, data) in group {
            results[index] = data
            inFlight -= 1
            
            if nextIndex < urls.count {
                let newIndex = nextIndex
                let url = urls[newIndex]
                group.addTask {
                    let (data, _) = try await URLSession.shared.data(from: url)
                    return (newIndex, data)
                }
                inFlight += 1
                nextIndex += 1
            }
        }
    }
    
    return results
}

struct User: Codable { let id: Int; let name: String }
struct Post: Codable { let id: Int; let title: String }

func fetchUsers() async throws -> [User] {
    try await Task.sleep(for: .milliseconds(100))
    return [User(id: 1, name: "Alice"), User(id: 2, name: "Bob")]
}

func fetchPosts() async throws -> [Post] {
    try await Task.sleep(for: .milliseconds(150))
    return [Post(id: 1, title: "Hello"), Post(id: 2, title: "World")]
}

func fetchUser(id: Int) async throws -> User {
    try await Task.sleep(for: .milliseconds(50))
    return User(id: id, name: "User \(id)")
}

func processItem(_ item: String) async throws {
    try await Task.sleep(for: .milliseconds(100))
    print("Processed: \(item)")
}
```

### 5.4 Task Priorities

```swift
// Task priorities ใน Swift
// .high = 25
// .medium = 21 (default)
// .low = 17
// .background = 9
// .userInitiated = 25
// .utility = 17

func taskPriorityExample() async {
    // สร้าง tasks ด้วย priorities ต่างกัน
    let tasks = await withTaskGroup(of: Void.self) { group in
        group.addTask(priority: .high) {
            print("High priority task")
            await Task.yield()
        }
        
        group.addTask(priority: .medium) {
            print("Medium priority task")
            await Task.yield()
        }
        
        group.addTask(priority: .background) {
            print("Background priority task")
            await Task.yield()
        }
        
        for await _ in group {}
    }
    
    // Priority escalation
    // ถ้า high priority task รอ medium priority task
    // medium priority จะถูก escalate เป็น high priority
    let highPriorityTask = Task(priority: .high) {
        // รอ result จาก lower priority task
        let result = await Task(priority: .background) {
            return "background result"
        }.value
        // Priority ของ background task จะถูก escalate เป็น high
        print("Got: \(result)")
    }
    
    await highPriorityTask.value
}

// Task priority inheritance
func priorityInheritance() async {
    // Current task priority
    let myPriority = Task.currentPriority
    print("My priority: \(myPriority)")
    
    // Child task inherits parent priority
    await Task {
        print("Child priority: \(Task.currentPriority)")
        // เท่ากับ myPriority
    }.value
    
    // Detached task ใช้ priority ที่ระบุ
    await Task.detached(priority: .low) {
        print("Detached priority: \(Task.currentPriority)")
        // .low ไม่ใช่ myPriority
    }.value
}
```

---

## 6. Advanced AsyncSequence

### 6.1 Building Custom AsyncSequence

```swift
// สร้าง custom AsyncSequence
struct CountdownSequence: AsyncSequence {
    typealias Element = Int
    
    let from: Int
    let delay: Duration
    
    struct AsyncIterator: AsyncIteratorProtocol {
        var current: Int
        let delay: Duration
        
        mutating func next() async throws -> Int? {
            guard current > 0 else { return nil }
            
            // ตรวจสอบ cancellation
            try Task.checkCancellation()
            
            // Delay ระหว่าง elements
            try await Task.sleep(for: delay)
            
            let value = current
            current -= 1
            return value
        }
    }
    
    func makeAsyncIterator() -> AsyncIterator {
        AsyncIterator(current: from, delay: delay)
    }
}

// ใช้งาน
func testCountdown() async throws {
    let countdown = CountdownSequence(from: 5, delay: .seconds(1))
    
    for try await count in countdown {
        print(count)
    }
    print("Launch!")
}

// Custom AsyncSequence สำหรับ file reading
struct FileLineSequence: AsyncSequence {
    typealias Element = String
    
    let url: URL
    
    struct AsyncIterator: AsyncIteratorProtocol {
        var lines: [String]
        var index = 0
        
        mutating func next() async throws -> String? {
            try Task.checkCancellation()
            
            guard index < lines.count else { return nil }
            let line = lines[index]
            index += 1
            
            // simulate async reading
            try await Task.sleep(for: .microseconds(100))
            
            return line
        }
    }
    
    func makeAsyncIterator() -> AsyncIterator {
        let content = (try? String(contentsOf: url)) ?? ""
        let lines = content.components(separatedBy: .newlines)
        return AsyncIterator(lines: lines)
    }
}
```

### 6.2 AsyncStream กับ Continuation

```swift
// AsyncStream - สร้าง AsyncSequence จาก callback/delegate patterns

// ตัวอย่าง 1: Timer เป็น AsyncStream
func makeTimerStream(interval: Duration) -> AsyncStream<Date> {
    AsyncStream { continuation in
        let timer = Timer.scheduledTimer(
            withTimeInterval: interval.components.seconds,
            repeats: true
        ) { _ in
            continuation.yield(Date())
        }
        
        // Cleanup เมื่อ stream จบ
        continuation.onTermination = { _ in
            timer.invalidate()
        }
    }
}

// ตัวอย่าง 2: NotificationCenter เป็น AsyncStream
func makeNotificationStream(name: Notification.Name) -> AsyncStream<Notification> {
    AsyncStream { continuation in
        let observer = NotificationCenter.default.addObserver(
            forName: name,
            object: nil,
            queue: nil
        ) { notification in
            continuation.yield(notification)
        }
        
        continuation.onTermination = { _ in
            NotificationCenter.default.removeObserver(observer)
        }
    }
}

// ตัวอย่าง 3: CLLocationManager เป็น AsyncStream
class LocationStreamProvider: NSObject, CLLocationManagerDelegate {
    private var continuation: AsyncStream<CLLocation>.Continuation?
    private let manager = CLLocationManager()
    
    func makeLocationStream() -> AsyncStream<CLLocation> {
        AsyncStream { continuation in
            self.continuation = continuation
            self.manager.delegate = self
            self.manager.startUpdatingLocation()
            
            continuation.onTermination = { [weak self] _ in
                self?.manager.stopUpdatingLocation()
            }
        }
    }
    
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        for location in locations {
            continuation?.yield(location)
        }
    }
}

// การใช้งาน
func processLocations() async {
    let provider = LocationStreamProvider()
    let locationStream = provider.makeLocationStream()
    
    for await location in locationStream {
        print("New location: \(location.coordinate)")
        
        // หยุดหลังจาก 10 updates
        if location.horizontalAccuracy < 10 {
            break
        }
    }
}
```

### 6.3 Back-pressure ใน AsyncStream

```swift
// AsyncStream.BufferingPolicy ควบคุม back-pressure

// ไม่มี buffer limit - เก็บทุก value
let unbuffered = AsyncStream<Int>(bufferingPolicy: .unbounded) { continuation in
    // ...
}

// Buffer แบบจำกัดจำนวน - ทิ้ง oldest ถ้าเต็ม
let bufferedOldest = AsyncStream<Int>(bufferingPolicy: .bufferingOldest(10)) { continuation in
    // ถ้า buffer เต็ม (10 items), drop newest item ที่เข้ามา
}

// Buffer แบบจำกัดจำนวน - ทิ้ง newest ถ้าเต็ม (default)
let bufferedNewest = AsyncStream<Int>(bufferingPolicy: .bufferingNewest(10)) { continuation in
    // ถ้า buffer เต็ม (10 items), drop oldest item
}

// ตัวอย่าง sensor data stream กับ back-pressure
actor SensorDataProcessor {
    let rawDataStream: AsyncStream<SensorReading>
    private let continuation: AsyncStream<SensorReading>.Continuation
    
    init() {
        var cont: AsyncStream<SensorReading>.Continuation!
        rawDataStream = AsyncStream(
            bufferingPolicy: .bufferingNewest(100) // เก็บ 100 readings ล่าสุด
        ) { continuation in
            cont = continuation
        }
        self.continuation = cont
    }
    
    func receiveSensorReading(_ reading: SensorReading) {
        let result = continuation.yield(reading)
        
        switch result {
        case .enqueued:
            // Normal case
            break
        case .dropped(let dropped):
            print("⚠️ Dropped sensor reading due to back-pressure: \(dropped)")
        case .terminated:
            print("Stream terminated")
        }
    }
    
    func processReadings() async {
        for await reading in rawDataStream {
            await process(reading)
        }
    }
    
    private func process(_ reading: SensorReading) async {
        // process...
        try? await Task.sleep(for: .milliseconds(50))
    }
}

struct SensorReading: Sendable {
    let timestamp: Date
    let value: Double
    let sensorId: String
}
```

### 6.4 AsyncAlgorithms Library

```swift
// swift-async-algorithms package
// https://github.com/apple/swift-async-algorithms

import AsyncAlgorithms

// ตัวอย่าง algorithm ต่างๆ
func asyncAlgorithmsExamples() async throws {
    // Zip - รวม sequences พร้อมกัน
    let numbers = AsyncStream<Int> { continuation in
        for i in 1...5 { continuation.yield(i) }
        continuation.finish()
    }
    
    let letters = AsyncStream<String> { continuation in
        for s in ["a", "b", "c", "d", "e"] { continuation.yield(s) }
        continuation.finish()
    }
    
    for await (number, letter) in zip(numbers, letters) {
        print("\(number): \(letter)")
    }
    
    // Chain - ต่อ sequences
    let first = [1, 2, 3].async
    let second = [4, 5, 6].async
    
    for await value in chain(first, second) {
        print(value) // 1, 2, 3, 4, 5, 6
    }
    
    // Merge - รวม sequences แบบ concurrent
    let stream1 = makeNumberStream(from: 1, to: 3, delay: .seconds(1))
    let stream2 = makeNumberStream(from: 10, to: 12, delay: .milliseconds(500))
    
    for await value in merge(stream1, stream2) {
        print(value) // ลำดับขึ้นกับ timing
    }
}

func makeNumberStream(from start: Int, to end: Int, delay: Duration) -> AsyncStream<Int> {
    AsyncStream { continuation in
        Task {
            for i in start...end {
                try? await Task.sleep(for: delay)
                continuation.yield(i)
            }
            continuation.finish()
        }
    }
}
```

---

## 7. TaskLocal Values

### 7.1 @TaskLocal Property Wrapper

```swift
// TaskLocal - ค่าที่ propagate ไปยัง child tasks

enum RequestContext {
    @TaskLocal static var requestId: String = "unknown"
    @TaskLocal static var userId: String? = nil
    @TaskLocal static var isAdmin: Bool = false
}

// การใช้งาน
func handleRequest() async {
    // Set task-local values สำหรับ request นี้
    await RequestContext.$requestId.withValue("req-12345") {
        await RequestContext.$userId.withValue("user-456") {
            await processRequest()
        }
    }
}

func processRequest() async {
    // อ่าน task-local values
    let requestId = RequestContext.requestId
    let userId = RequestContext.userId ?? "anonymous"
    
    print("Processing request \(requestId) for user \(userId)")
    
    // Child tasks จะ inherit task-local values
    await withTaskGroup(of: Void.self) { group in
        group.addTask {
            // requestId และ userId พร้อมใช้งานที่นี่ด้วย
            let id = RequestContext.requestId
            print("Child task has requestId: \(id)")
        }
    }
}
```

### 7.2 Context Propagation

```swift
// Task-local values propagate ลงไปยัง child tasks
// แต่ไม่ขึ้นไปยัง parent หรือ sibling tasks

enum TraceContext {
    @TaskLocal static var traceId: String = ""
    @TaskLocal static var spanId: String = ""
    @TaskLocal static var baggage: [String: String] = [:]
}

// Tracing middleware
func withTracing<T>(
    operation: String,
    execute: () async throws -> T
) async rethrows -> T {
    let traceId = TraceContext.traceId.isEmpty 
        ? UUID().uuidString 
        : TraceContext.traceId
    let spanId = UUID().uuidString
    
    return try await TraceContext.$traceId.withValue(traceId) {
        try await TraceContext.$spanId.withValue(spanId) {
            let start = Date()
            defer {
                let elapsed = Date().timeIntervalSince(start)
                print("[\(traceId)/\(spanId)] \(operation) completed in \(String(format: "%.3f", elapsed))s")
            }
            
            print("[\(traceId)/\(spanId)] Starting \(operation)")
            return try await execute()
        }
    }
}

// ใช้งาน
func apiHandler() async throws {
    try await withTracing(operation: "api.handler") {
        let data = try await withTracing(operation: "db.query") {
            try await fetchFromDatabase()
        }
        
        let processed = try await withTracing(operation: "data.process") {
            try await processData(data)
        }
        
        print("Result: \(processed)")
    }
}

func fetchFromDatabase() async throws -> [String] {
    try await Task.sleep(for: .milliseconds(50))
    return ["item1", "item2"]
}

func processData(_ data: [String]) async throws -> String {
    try await Task.sleep(for: .milliseconds(20))
    return data.joined(separator: ", ")
}
```

### 7.3 Tracing กับ TaskLocal

```swift
// Complete distributed tracing implementation
struct Span {
    let traceId: String
    let spanId: String
    let parentSpanId: String?
    let operation: String
    let startTime: Date
    var endTime: Date?
    var attributes: [String: String] = [:]
    var events: [(Date, String)] = []
    
    mutating func addAttribute(_ key: String, _ value: String) {
        attributes[key] = value
    }
    
    mutating func addEvent(_ message: String) {
        events.append((Date(), message))
    }
}

actor SpanCollector {
    var spans: [Span] = []
    
    func record(_ span: Span) {
        spans.append(span)
    }
    
    func printTrace() {
        let sorted = spans.sorted { $0.startTime < $1.startTime }
        for span in sorted {
            let duration = span.endTime.map { 
                $0.timeIntervalSince(span.startTime) * 1000 
            }.map { String(format: "%.2fms", $0) } ?? "ongoing"
            
            let indent = span.parentSpanId != nil ? "  " : ""
            print("\(indent)[\(span.traceId)] \(span.operation): \(duration)")
        }
    }
}

enum Tracer {
    @TaskLocal static var currentSpan: Span? = nil
    static let collector = SpanCollector()
    
    static func startSpan(operation: String) -> Span {
        let traceId = currentSpan?.traceId ?? UUID().uuidString
        return Span(
            traceId: traceId,
            spanId: UUID().uuidString,
            parentSpanId: currentSpan?.spanId,
            operation: operation,
            startTime: Date()
        )
    }
    
    static func withSpan<T>(
        operation: String,
        execute: () async throws -> T
    ) async rethrows -> T {
        var span = startSpan(operation: operation)
        
        let result = try await $currentSpan.withValue(span) {
            try await execute()
        }
        
        span.endTime = Date()
        await collector.record(span)
        
        return result
    }
}
```

---

## 8. Clock และ Time

### 8.1 Clock Protocol

```swift
// Clock protocol - abstract over time measurement
// public protocol Clock<Duration>: Sendable {
//     associatedtype Duration
//     associatedtype Instant: InstantProtocol where Instant.Duration == Duration
//     
//     var now: Instant { get }
//     var minimumResolution: Duration { get }
//     func sleep(until deadline: Instant, tolerance: Duration?) async throws
// }

// Built-in clocks:
// ContinuousClock - เดินต่อเนื่อง แม้ system sleep
// SuspendingClock - หยุดเมื่อ system sleep

// ใช้งาน Clock
func measureTime() async {
    let clock = ContinuousClock()
    
    let elapsed = await clock.measure {
        try? await Task.sleep(for: .seconds(1))
    }
    
    print("Elapsed: \(elapsed)")
}
```

### 8.2 ContinuousClock vs SuspendingClock

```swift
// ContinuousClock - นับเวลาแบบต่อเนื่อง
// เหมาะสำหรับ: measuring elapsed time, animations, UI timing
let continuousClock = ContinuousClock()
let startContinuous = continuousClock.now

// SuspendingClock - หยุดเมื่อ device sleep
// เหมาะสำหรับ: alarms, reminders (user-facing time)
let suspendingClock = SuspendingClock()
let startSuspending = suspendingClock.now

// ความแตกต่าง:
// ถ้า device sleep 1 ชั่วโมง:
// ContinuousClock.now - startContinuous = 1 hour + เวลาก่อน sleep
// SuspendingClock.now - startSuspending = เฉพาะเวลาที่ไม่ได้ sleep

func demonstrateClocks() async throws {
    let continuous = ContinuousClock()
    let suspending = SuspendingClock()
    
    let start1 = continuous.now
    let start2 = suspending.now
    
    // Sleep for 2 seconds
    try await Task.sleep(until: continuous.now.advanced(by: .seconds(2)), clock: continuous)
    
    let elapsed1 = continuous.now - start1
    let elapsed2 = suspending.now - start2
    
    print("Continuous elapsed: \(elapsed1)")
    print("Suspending elapsed: \(elapsed2)")
    // ทั้งสองควรใกล้เคียงกันถ้า device ไม่ได้ sleep
}
```

### 8.3 sleep(for:) กับ Clock

```swift
// Task.sleep ใช้ SuspendingClock โดย default
try await Task.sleep(for: .seconds(1))

// ระบุ clock เองได้
try await Task.sleep(until: ContinuousClock.now.advanced(by: .seconds(1)), 
                     clock: ContinuousClock())

// ตัวอย่าง retry กับ exponential backoff
func retryWithBackoff<T>(
    maxAttempts: Int,
    initialDelay: Duration = .seconds(1),
    maxDelay: Duration = .seconds(60),
    clock: some Clock<Duration> = ContinuousClock(),
    operation: () async throws -> T
) async throws -> T {
    var delay = initialDelay
    
    for attempt in 1...maxAttempts {
        do {
            return try await operation()
        } catch {
            guard attempt < maxAttempts else { throw error }
            
            print("Attempt \(attempt) failed, retrying in \(delay)...")
            try await clock.sleep(for: delay)
            
            delay = min(delay * 2, maxDelay) // exponential backoff
        }
    }
    
    throw RetryError.maxAttemptsReached
}

enum RetryError: Error {
    case maxAttemptsReached
}
```

### 8.4 Testing กับ Custom Clocks

```swift
// Test clock สำหรับ testing โดยไม่ต้องรอเวลาจริง
// ใช้ swift-clocks package หรือสร้างเอง

// Manual Clock สำหรับ testing
final class ManualClock: Clock, @unchecked Sendable {
    typealias Duration = Swift.Duration
    
    struct Instant: InstantProtocol {
        var offset: Duration = .zero
        
        func advanced(by duration: Duration) -> Self {
            Instant(offset: offset + duration)
        }
        
        func duration(to other: Self) -> Duration {
            other.offset - offset
        }
        
        static func < (lhs: Self, rhs: Self) -> Bool {
            lhs.offset < rhs.offset
        }
    }
    
    private(set) var now = Instant()
    private var sleepers: [(Instant, CheckedContinuation<Void, Error>)] = []
    private let lock = NSLock()
    
    var minimumResolution: Duration { .nanoseconds(1) }
    
    func sleep(until deadline: Instant, tolerance: Duration? = nil) async throws {
        try await withCheckedThrowingContinuation { continuation in
            lock.lock()
            defer { lock.unlock() }
            sleepers.append((deadline, continuation))
            sleepers.sort { $0.0 < $1.0 }
        }
    }
    
    func advance(by duration: Duration) {
        lock.lock()
        now = now.advanced(by: duration)
        let toWake = sleepers.filter { $0.0 <= now }
        sleepers.removeAll { $0.0 <= now }
        lock.unlock()
        
        for (_, continuation) in toWake {
            continuation.resume()
        }
    }
}

// ใช้ ManualClock ใน tests
func testWithManualClock() async throws {
    let clock = ManualClock()
    var eventLog: [String] = []
    
    // Start countdown ที่ใช้ clock
    let task = Task {
        for countdown in stride(from: 5, through: 0, by: -1) {
            eventLog.append("Tick: \(countdown)")
            try await clock.sleep(until: clock.now.advanced(by: .seconds(1)))
        }
        eventLog.append("Done!")
    }
    
    // Advance time manually
    for _ in 0...5 {
        clock.advance(by: .seconds(1))
        await Task.yield() // ให้ task ทำงาน
    }
    
    await task.value
    print(eventLog)
    // ["Tick: 5", "Tick: 4", "Tick: 3", "Tick: 2", "Tick: 1", "Tick: 0", "Done!"]
}
```

---

## 9. Async Algorithms

### 9.1 swift-async-algorithms Overview

```swift
// Package.swift
// .package(url: "https://github.com/apple/swift-async-algorithms", from: "1.0.0")
// .product(name: "AsyncAlgorithms", package: "swift-async-algorithms")

import AsyncAlgorithms

// สร้าง async sequences สำหรับ demo
extension Array {
    var async: AsyncLazySequence<[Element]> {
        AsyncLazySequence(self)
    }
}
```

### 9.2 Debounce และ Throttle

```swift
import AsyncAlgorithms

// Debounce - emit หลังจากไม่มี events มาสักระยะ
func searchDebounce() async throws {
    let searchQueries = AsyncStream<String> { continuation in
        // simulate user typing
        Task {
            let keystrokes = ["h", "he", "hel", "hell", "hello"]
            for k in keystrokes {
                continuation.yield(k)
                try? await Task.sleep(for: .milliseconds(100))
            }
            try? await Task.sleep(for: .seconds(1)) // pause
            continuation.yield("hello world")
            continuation.finish()
        }
    }
    
    // รอ 300ms ของความเงียบก่อน emit
    let debouncedQueries = searchQueries.debounce(for: .milliseconds(300), clock: ContinuousClock())
    
    for await query in debouncedQueries {
        print("Search for: \(query)")
        // จะเห็นแค่ "hello" และ "hello world"
    }
}

// Throttle - emit ไม่เกิน 1 ครั้งต่อ interval
func locationThrottle() async throws {
    let locationUpdates = AsyncStream<CLLocationCoordinate2D> { continuation in
        // simulate rapid location updates
        Task {
            for i in 0..<20 {
                continuation.yield(CLLocationCoordinate2D(
                    latitude: 13.7 + Double(i) * 0.001,
                    longitude: 100.5
                ))
                try? await Task.sleep(for: .milliseconds(100))
            }
            continuation.finish()
        }
    }
    
    // Emit ไม่เกิน 1 ครั้งต่อวินาที
    let throttled = locationUpdates.throttle(for: .seconds(1), clock: ContinuousClock())
    
    for await coordinate in throttled {
        print("Location: \(coordinate.latitude), \(coordinate.longitude)")
    }
}
```

### 9.3 Merge, Zip, Chain

```swift
import AsyncAlgorithms

// Merge - รวม sequences แบบ concurrent (emit เมื่อ element พร้อม)
func mergeExample() async {
    let evens = AsyncStream<Int> { continuation in
        Task {
            for i in stride(from: 0, to: 10, by: 2) {
                try? await Task.sleep(for: .milliseconds(300))
                continuation.yield(i)
            }
            continuation.finish()
        }
    }
    
    let odds = AsyncStream<Int> { continuation in
        Task {
            for i in stride(from: 1, to: 10, by: 2) {
                try? await Task.sleep(for: .milliseconds(200))
                continuation.yield(i)
            }
            continuation.finish()
        }
    }
    
    // รวมทั้งสอง streams
    for await value in merge(evens, odds) {
        print(value) // mixed order based on timing
    }
}

// Zip - combine elements pairwise (รอให้ทั้งคู่มี element)
func zipExample() async {
    let names = ["Alice", "Bob", "Charlie"].async
    let scores = [95, 87, 92].async
    
    for await (name, score) in zip(names, scores) {
        print("\(name): \(score)")
    }
}

// Chain - ต่อ sequences ตามลำดับ
func chainExample() async {
    let first = [1, 2, 3].async
    let second = [4, 5, 6].async
    let third = [7, 8, 9].async
    
    for await value in chain(first, second, third) {
        print(value) // 1, 2, 3, 4, 5, 6, 7, 8, 9
    }
}
```

### 9.4 Buffer และ Window

```swift
import AsyncAlgorithms

// Buffer - สะสม elements ก่อนส่งเป็น batch
func bufferExample() async {
    let dataStream = makeHighFrequencyStream()
    
    // สะสม 10 elements แล้วส่งเป็น array
    for await batch in dataStream.chunks(ofCount: 10) {
        print("Processing batch of \(batch.count): \(batch)")
        await processBatch(Array(batch))
    }
}

// Chunks โดย time
func timeWindowExample() async throws {
    let events = makeEventStream()
    
    // รวม events ใน 1 วินาที
    for await window in events.chunked(
        by: .repeating(every: .seconds(1), clock: ContinuousClock())
    ) {
        print("Events in window: \(Array(window))")
    }
}

// Sliding window
func slidingWindowExample() async {
    let values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10].async
    
    // Window ขนาด 3 แบบ sliding
    for await window in values.windows(ofCount: 3) {
        print("Window: \(Array(window))")
        // [1,2,3], [2,3,4], [3,4,5], ...
    }
}

func makeHighFrequencyStream() -> AsyncStream<Int> {
    AsyncStream { continuation in
        Task {
            for i in 0..<100 {
                continuation.yield(i)
            }
            continuation.finish()
        }
    }
}

func makeEventStream() -> AsyncStream<String> {
    AsyncStream { continuation in
        Task {
            for i in 0..<20 {
                try? await Task.sleep(for: .milliseconds(150))
                continuation.yield("event-\(i)")
            }
            continuation.finish()
        }
    }
}

func processBatch(_ batch: [Int]) async {
    // process...
}
```

---

## 10. Concurrency กับ Core Data

### 10.1 NSManagedObjectContext Isolation

```swift
import CoreData

// Core Data context ต้องใช้ใน thread/queue ที่ถูกต้อง
// Main context: main queue
// Background context: private queue

class CoreDataStack {
    lazy var persistentContainer: NSPersistentContainer = {
        let container = NSPersistentContainer(name: "Model")
        container.loadPersistentStores { _, error in
            if let error = error {
                fatalError("Failed to load: \(error)")
            }
        }
        return container
    }()
    
    var viewContext: NSManagedObjectContext {
        persistentContainer.viewContext
    }
    
    func newBackgroundContext() -> NSManagedObjectContext {
        persistentContainer.newBackgroundContext()
    }
}

// ❌ อย่าทำ: เข้าถึง managed objects บน wrong thread
func badCoreData(context: NSManagedObjectContext) async {
    // บาง context ต้องทำงานบน specific queue
    let objects = try? context.fetch(NSFetchRequest<NSManagedObject>(entityName: "Item"))
    // อาจ crash หรือ data corruption!
}
```

### 10.2 perform async/await (Swift 5.7+)

```swift
import CoreData

extension NSManagedObjectContext {
    // perform กับ async/await (iOS 15+)
    func fetchAsync<T: NSManagedObject>(request: NSFetchRequest<T>) async throws -> [T] {
        try await perform {
            try self.fetch(request)
        }
    }
}

// ตัวอย่างการใช้งาน
class ItemRepository {
    let stack: CoreDataStack
    
    init(stack: CoreDataStack) {
        self.stack = stack
    }
    
    func fetchAllItems() async throws -> [Item] {
        let context = stack.viewContext
        let request = Item.fetchRequest()
        request.sortDescriptors = [NSSortDescriptor(key: "name", ascending: true)]
        
        return try await context.perform {
            try context.fetch(request)
        }
    }
    
    func createItem(name: String, value: Int) async throws {
        let context = stack.newBackgroundContext()
        
        try await context.perform {
            let item = Item(context: context)
            item.name = name
            item.value = Int32(value)
            item.createdAt = Date()
            
            try context.save()
        }
    }
    
    func updateItem(id: NSManagedObjectID, newName: String) async throws {
        let context = stack.newBackgroundContext()
        
        try await context.perform {
            guard let item = try context.existingObject(with: id) as? Item else {
                throw NSError(domain: "NotFound", code: 404)
            }
            item.name = newName
            try context.save()
        }
    }
    
    func deleteItems(matching predicate: NSPredicate) async throws -> Int {
        let context = stack.newBackgroundContext()
        
        return try await context.perform {
            let request = Item.fetchRequest()
            request.predicate = predicate
            
            let items = try context.fetch(request)
            items.forEach { context.delete($0) }
            try context.save()
            
            return items.count
        }
    }
}

// Mock Item entity
class Item: NSManagedObject {
    @NSManaged var name: String?
    @NSManaged var value: Int32
    @NSManaged var createdAt: Date?
    
    static func fetchRequest() -> NSFetchRequest<Item> {
        NSFetchRequest<Item>(entityName: "Item")
    }
}
```

### 10.3 Background Context Patterns

```swift
import CoreData

// Pattern 1: Batch import กับ background context
class DataImporter {
    let stack: CoreDataStack
    
    init(stack: CoreDataStack) {
        self.stack = stack
    }
    
    func importItems(_ rawData: [[String: Any]]) async throws {
        let context = stack.newBackgroundContext()
        context.mergePolicy = NSMergeByPropertyObjectTrumpMergePolicy
        
        try await context.perform {
            // Process ใน batches เพื่อประหยัด memory
            let batchSize = 100
            var processed = 0
            
            for batchStart in stride(from: 0, to: rawData.count, by: batchSize) {
                let batchEnd = min(batchStart + batchSize, rawData.count)
                let batch = rawData[batchStart..<batchEnd]
                
                for data in batch {
                    let item = Item(context: context)
                    item.name = data["name"] as? String
                    item.value = Int32(data["value"] as? Int ?? 0)
                    item.createdAt = Date()
                }
                
                // Save แต่ละ batch
                if context.hasChanges {
                    try context.save()
                    context.reset() // Reset context เพื่อ free memory
                }
                
                processed += batch.count
                print("Imported \(processed)/\(rawData.count)")
            }
        }
    }
}

// Pattern 2: Child context สำหรับ editing
class EditingSession {
    let parentContext: NSManagedObjectContext
    let editContext: NSManagedObjectContext
    
    init(parentContext: NSManagedObjectContext) {
        self.parentContext = parentContext
        self.editContext = NSManagedObjectContext(concurrencyType: .privateQueueConcurrencyType)
        self.editContext.parent = parentContext
    }
    
    func editItem(id: NSManagedObjectID) async -> Item? {
        await editContext.perform {
            self.editContext.object(with: id) as? Item
        }
    }
    
    func saveEdits() async throws {
        // Save child context ขึ้น parent
        try await editContext.perform {
            if self.editContext.hasChanges {
                try self.editContext.save()
            }
        }
        
        // Save parent ไปยัง persistent store
        try await parentContext.perform {
            if self.parentContext.hasChanges {
                try self.parentContext.save()
            }
        }
    }
    
    func discardEdits() async {
        await editContext.perform {
            self.editContext.rollback()
        }
    }
}
```

---

## 11. Concurrency กับ URLSession

### 11.1 Concurrent Requests กับ TaskGroup

```swift
import Foundation

struct APIClient {
    let baseURL: URL
    let session: URLSession
    
    init(baseURL: URL) {
        self.baseURL = baseURL
        
        let config = URLSessionConfiguration.default
        config.httpMaximumConnectionsPerHost = 6 // HTTP/1.1 limit
        self.session = URLSession(configuration: config)
    }
    
    // ดึงข้อมูลหลาย endpoints พร้อมกัน
    func fetchMultipleEndpoints(paths: [String]) async throws -> [String: Data] {
        var results: [String: Data] = [:]
        
        try await withThrowingTaskGroup(of: (String, Data).self) { group in
            for path in paths {
                group.addTask {
                    let url = self.baseURL.appendingPathComponent(path)
                    let (data, _) = try await self.session.data(from: url)
                    return (path, data)
                }
            }
            
            for try await (path, data) in group {
                results[path] = data
            }
        }
        
        return results
    }
}
```

### 11.2 Parallel Image Downloads

```swift
import UIKit

class ImageDownloader {
    private let session: URLSession
    private var cache: [URL: UIImage] = [:]
    private let cacheActor = CacheActor()
    
    init() {
        let config = URLSessionConfiguration.default
        config.urlCache = URLCache(
            memoryCapacity: 50 * 1024 * 1024,  // 50MB
            diskCapacity: 200 * 1024 * 1024     // 200MB
        )
        self.session = URLSession(configuration: config)
    }
    
    func downloadImages(urls: [URL]) async throws -> [URL: UIImage] {
        var results: [URL: UIImage] = [:]
        
        try await withThrowingTaskGroup(of: (URL, UIImage).self) { group in
            for url in urls {
                group.addTask { [self] in
                    // ตรวจสอบ cache ก่อน
                    if let cached = await cacheActor.get(url) {
                        return (url, cached)
                    }
                    
                    // Download
                    let (data, response) = try await session.data(from: url)
                    
                    guard let httpResponse = response as? HTTPURLResponse,
                          (200...299).contains(httpResponse.statusCode) else {
                        throw URLError(.badServerResponse)
                    }
                    
                    guard let image = UIImage(data: data) else {
                        throw URLError(.cannotDecodeContentData)
                    }
                    
                    // Cache result
                    await cacheActor.set(image, for: url)
                    
                    return (url, image)
                }
            }
            
            for try await (url, image) in group {
                results[url] = image
            }
        }
        
        return results
    }
    
    // Thumbnail generation กับ concurrent processing
    func generateThumbnails(from urls: [URL], size: CGSize) async throws -> [UIImage] {
        let images = try await downloadImages(urls: urls)
        
        return try await withThrowingTaskGroup(of: UIImage.self) { group in
            for url in urls {
                guard let image = images[url] else { continue }
                
                group.addTask {
                    return await self.resize(image: image, to: size)
                }
            }
            
            var thumbnails: [UIImage] = []
            for try await thumbnail in group {
                thumbnails.append(thumbnail)
            }
            return thumbnails
        }
    }
    
    private func resize(image: UIImage, to size: CGSize) async -> UIImage {
        // Image resizing ใน background
        return await Task.detached(priority: .userInitiated) {
            let renderer = UIGraphicsImageRenderer(size: size)
            return renderer.image { _ in
                image.draw(in: CGRect(origin: .zero, size: size))
            }
        }.value
    }
}

actor CacheActor {
    private var cache: [URL: UIImage] = [:]
    
    func get(_ url: URL) -> UIImage? {
        cache[url]
    }
    
    func set(_ image: UIImage, for url: URL) {
        cache[url] = image
    }
}
```

### 11.3 Rate Limiting Concurrent Requests

```swift
// Rate Limiter ด้วย Actor
actor RateLimiter {
    private let maxRequests: Int
    private let window: Duration
    private var requestTimestamps: [ContinuousClock.Instant] = []
    private let clock = ContinuousClock()
    
    init(maxRequests: Int, per window: Duration) {
        self.maxRequests = maxRequests
        self.window = window
    }
    
    func waitForPermit() async {
        while !canProceed() {
            // ลบ timestamps เก่าออก
            cleanOldTimestamps()
            
            if !canProceed() {
                // รอจนกว่า oldest request จะหมดอายุ
                let oldest = requestTimestamps.first!
                let expireTime = oldest.advanced(by: window)
                let now = clock.now
                
                if expireTime > now {
                    try? await Task.sleep(until: expireTime, clock: clock)
                }
            }
        }
        
        requestTimestamps.append(clock.now)
    }
    
    private func canProceed() -> Bool {
        cleanOldTimestamps()
        return requestTimestamps.count < maxRequests
    }
    
    private func cleanOldTimestamps() {
        let cutoff = clock.now.advanced(by: -window)
        requestTimestamps.removeAll { $0 < cutoff }
    }
}

// ใช้ rate limiter
class RateLimitedAPIClient {
    private let session = URLSession.shared
    private let rateLimiter = RateLimiter(maxRequests: 10, per: .seconds(1))
    
    func fetch(url: URL) async throws -> Data {
        await rateLimiter.waitForPermit()
        let (data, _) = try await session.data(from: url)
        return data
    }
    
    func fetchAll(urls: [URL]) async throws -> [Data] {
        try await withThrowingTaskGroup(of: (Int, Data).self) { group in
            for (index, url) in urls.enumerated() {
                group.addTask {
                    let data = try await self.fetch(url: url)
                    return (index, data)
                }
            }
            
            var results = Array(repeating: Data(), count: urls.count)
            for try await (index, data) in group {
                results[index] = data
            }
            return results
        }
    }
}
```

---

## 12. Testing Async Code

### 12.1 Swift Testing Async Support

```swift
import Testing

// Swift Testing รองรับ async tests โดยตรง
@Test
func testAsyncFetch() async throws {
    let client = MockAPIClient()
    let result = try await client.fetch()
    
    #expect(result == "expected data")
}

@Test
func testConcurrentOperations() async throws {
    let counter = Counter()
    
    await withTaskGroup(of: Void.self) { group in
        for _ in 0..<100 {
            group.addTask {
                await counter.increment()
            }
        }
    }
    
    let count = await counter.value
    #expect(count == 100)
}

actor Counter {
    var value = 0
    
    func increment() {
        value += 1
    }
}

class MockAPIClient {
    func fetch() async throws -> String {
        try await Task.sleep(for: .milliseconds(10))
        return "expected data"
    }
}
```

### 12.2 XCTest Async Expectations

```swift
import XCTest

class AsyncTests: XCTestCase {
    // XCTest รองรับ async test functions
    func testAsyncOperation() async throws {
        let service = AsyncService()
        let result = try await service.performOperation()
        XCTAssertEqual(result, "success")
    }
    
    // Test กับ timeout
    func testWithTimeout() async throws {
        let service = SlowService()
        
        // ถ้าใช้ expectation แบบเก่า
        let expectation = expectation(description: "operation completes")
        
        Task {
            do {
                let result = try await service.slowOperation()
                XCTAssertFalse(result.isEmpty)
                expectation.fulfill()
            } catch {
                XCTFail("Unexpected error: \(error)")
            }
        }
        
        await fulfillment(of: [expectation], timeout: 10.0)
    }
    
    // Test concurrent operations
    func testConcurrentWrites() async throws {
        let db = AsyncDatabase()
        
        try await withThrowingTaskGroup(of: Void.self) { group in
            for i in 0..<50 {
                group.addTask {
                    try await db.write("key\(i)", value: "value\(i)")
                }
            }
        }
        
        let count = await db.count
        XCTAssertEqual(count, 50)
    }
}

class AsyncService {
    func performOperation() async throws -> String {
        try await Task.sleep(for: .milliseconds(50))
        return "success"
    }
}

class SlowService {
    func slowOperation() async throws -> String {
        try await Task.sleep(for: .seconds(2))
        return "done"
    }
}

actor AsyncDatabase {
    var storage: [String: String] = [:]
    
    var count: Int { storage.count }
    
    func write(_ key: String, value: String) async throws {
        storage[key] = value
    }
}
```

### 12.3 Testing กับ Custom Clocks

```swift
import XCTest

// Test scheduler ที่ใช้ manual clock
class TimedOperationTests: XCTestCase {
    func testRetryWithMockClock() async throws {
        let clock = ManualClock()
        var attemptCount = 0
        
        let task = Task {
            // Operation ที่ fail 2 ครั้งแล้วสำเร็จ
            try await retryWithBackoff(
                maxAttempts: 3,
                initialDelay: .seconds(1),
                clock: clock
            ) {
                attemptCount += 1
                if attemptCount < 3 {
                    throw TestError.transientFailure
                }
                return "success"
            }
        }
        
        // Advance time ให้ครบ retry delays
        await Task.yield()
        clock.advance(by: .seconds(1)) // retry 1
        await Task.yield()
        clock.advance(by: .seconds(2)) // retry 2 (exponential backoff)
        await Task.yield()
        
        let result = try await task.value
        XCTAssertEqual(result, "success")
        XCTAssertEqual(attemptCount, 3)
    }
}

enum TestError: Error {
    case transientFailure
}
```

### 12.4 Testing Cancellation Behavior

```swift
import XCTest

class CancellationTests: XCTestCase {
    func testCancellationPropagates() async {
        let task = Task {
            do {
                try await longOperation()
                XCTFail("Should have been cancelled")
            } catch is CancellationError {
                // Expected
                print("Task was properly cancelled")
            }
        }
        
        // Cancel หลังจาก short delay
        try? await Task.sleep(for: .milliseconds(100))
        task.cancel()
        
        await task.value
    }
    
    func testTaskGroupCancellation() async {
        var completedTasks: [Int] = []
        let completionActor = CompletionTracker()
        
        let task = Task {
            await withTaskGroup(of: Void.self) { group in
                for i in 0..<5 {
                    group.addTask {
                        do {
                            try await Task.sleep(for: .seconds(Double(i) + 1))
                            await completionActor.complete(i)
                        } catch is CancellationError {
                            // Task cancelled
                        }
                    }
                }
            }
        }
        
        // Cancel หลังจาก 1.5 วินาที
        try? await Task.sleep(for: .milliseconds(1500))
        task.cancel()
        
        await task.value
        
        let completed = await completionActor.completed
        // เฉพาะ task 0 (sleep 1s) ที่ควรจะ complete
        XCTAssertEqual(completed, [0])
    }
}

actor CompletionTracker {
    var completed: [Int] = []
    
    func complete(_ id: Int) {
        completed.append(id)
    }
}

func longOperation() async throws {
    for _ in 0..<100 {
        try Task.checkCancellation()
        try await Task.sleep(for: .milliseconds(100))
    }
}
```

---

## 13. Common Concurrency Bugs

### 13.1 Data Races (Swift 6 Examples)

```swift
// Swift 6 strict concurrency checking จะ catch bugs เหล่านี้

// ❌ Data race - accessing shared mutable state
class SharedCounter_Unsafe {
    var count = 0
    
    func increment() {
        count += 1 // Swift 6: error - mutation of captured var in concurrently-executing code
    }
}

// ❌ การส่ง non-Sendable type ข้าม concurrency boundary
class NonSendableData {
    var value: String = ""
}

func unsafeTransfer() async {
    let data = NonSendableData()
    
    Task {
        // Swift 6: error - capture of non-Sendable type
        print(data.value)
    }
}

// ✅ Fix: ใช้ Actor
actor SafeCounter {
    var count = 0
    
    func increment() {
        count += 1 // Safe - protected by actor
    }
}

// ✅ Fix: ใช้ Sendable type
struct SendableData: Sendable {
    let value: String
}

func safeTransfer() async {
    let data = SendableData(value: "hello")
    
    Task {
        print(data.value) // Safe - Sendable
    }
}

// ❌ Race condition กับ actor reentrancy
actor DownloadManager_Buggy {
    var downloads: Set<URL> = []
    
    func download(url: URL) async throws -> Data {
        guard !downloads.contains(url) else {
            throw DownloadError.alreadyInProgress
        }
        downloads.insert(url) // ✅ ก่อน await
        
        // Suspension point!
        let data = try await URLSession.shared.data(from: url).0
        
        // downloads.contains(url) ยังคงเป็น true เพราะเราตั้งค่าก่อน await
        downloads.remove(url)
        return data
    }
}

enum DownloadError: Error {
    case alreadyInProgress
}
```

### 13.2 Deadlocks ใน Actor Code

```swift
// ❌ Potential deadlock: actor calling another actor synchronously
// (จริงๆ Swift actors ไม่ deadlock แบบ GCD แต่มี performance issue)

actor ServiceA {
    let serviceB: ServiceB
    
    init(serviceB: ServiceB) {
        self.serviceB = serviceB
    }
    
    // ✅ Safe เพราะ Swift actors ใช้ cooperative scheduling
    func doWork() async -> String {
        // await จะ suspend ไม่ block
        let result = await serviceB.doWork()
        return "A: \(result)"
    }
}

actor ServiceB {
    func doWork() async -> String {
        return "B: done"
    }
}

// ❌ อันตราย: ใช้ semaphore กับ async code
func deadlockExample() async {
    let sem = DispatchSemaphore(value: 0)
    
    Task {
        // ถ้า thread pool เต็ม task นี้อาจไม่ทำงาน
        sem.signal()
    }
    
    // DEADLOCK! ถ้า task ข้างบนไม่ได้รับ thread
    sem.wait() // blocks the current thread
}

// ✅ ใช้ async/await แทน
func nonDeadlockExample() async {
    await withCheckedContinuation { continuation in
        Task {
            continuation.resume()
        }
    }
}
```

### 13.3 Task Leak Detection

```swift
// Task leaks เกิดเมื่อ tasks ไม่ถูก cancel และ deallocate

class TaskLeaker {
    var tasks: [Task<Void, Never>] = []
    
    // ❌ Task leak - ไม่เก็บ reference หรือ cancel
    func leak() {
        Task {
            // task นี้จะทำงานไปเรื่อยๆ แม้ object ถูก deallocate
            while true {
                try? await Task.sleep(for: .seconds(1))
                print("Still running!")
            }
        }
        // task ถูก forget - no way to cancel
    }
    
    // ✅ เก็บ task reference เพื่อ cancel ได้
    func noLeak() {
        let task = Task {
            while !Task.isCancelled {
                try? await Task.sleep(for: .seconds(1))
                print("Running...")
            }
        }
        tasks.append(task)
    }
    
    deinit {
        // Cancel tasks เมื่อ object deallocate
        for task in tasks {
            task.cancel()
        }
    }
}

// Task scope management
class TaskManager {
    private var activeTasks: Set<Task<Void, Never>> = []
    private let taskActor = TaskTrackingActor()
    
    func addTask(_ task: Task<Void, Never>) async {
        await taskActor.add(task)
        
        // Auto-remove เมื่อ task complete
        Task {
            await task.value
            await self.taskActor.remove(task)
        }
    }
    
    func cancelAll() async {
        await taskActor.cancelAll()
    }
}

actor TaskTrackingActor {
    var tasks: Set<Task<Void, Never>> = []
    
    func add(_ task: Task<Void, Never>) {
        tasks.insert(task)
    }
    
    func remove(_ task: Task<Void, Never>) {
        tasks.remove(task)
    }
    
    func cancelAll() {
        tasks.forEach { $0.cancel() }
        tasks.removeAll()
    }
}
```

---

## 14. Complete Concurrent App: Real-Time Data Dashboard

```swift
import SwiftUI
import Foundation

// MARK: - Data Models

struct MetricData: Sendable, Identifiable {
    let id: UUID
    let name: String
    var value: Double
    var history: [Double]
    let unit: String
    let threshold: Double
    
    init(name: String, unit: String, threshold: Double) {
        self.id = UUID()
        self.name = name
        self.value = 0
        self.history = []
        self.unit = unit
        self.threshold = threshold
    }
    
    var isOverThreshold: Bool { value > threshold }
    var trend: Double {
        guard history.count >= 2 else { return 0 }
        return history.last! - history[history.count - 2]
    }
}

struct SystemAlert: Sendable, Identifiable {
    let id = UUID()
    let message: String
    let severity: Severity
    let timestamp: Date
    
    enum Severity: String, Sendable {
        case info, warning, critical
    }
}

// MARK: - Data Sources

actor MetricDataSource {
    private var isRunning = false
    
    func streamCPUUsage() -> AsyncStream<Double> {
        AsyncStream { continuation in
            Task {
                while !Task.isCancelled {
                    let usage = Double.random(in: 20...80)
                    continuation.yield(usage)
                    try? await Task.sleep(for: .seconds(1))
                }
                continuation.finish()
            }
        }
    }
    
    func streamMemoryUsage() -> AsyncStream<Double> {
        AsyncStream { continuation in
            Task {
                var baseMemory = 50.0
                while !Task.isCancelled {
                    baseMemory += Double.random(in: -2...3)
                    baseMemory = max(10, min(90, baseMemory))
                    continuation.yield(baseMemory)
                    try? await Task.sleep(for: .milliseconds(1500))
                }
                continuation.finish()
            }
        }
    }
    
    func streamNetworkLatency() -> AsyncStream<Double> {
        AsyncStream { continuation in
            Task {
                while !Task.isCancelled {
                    let latency = Double.random(in: 10...200)
                    continuation.yield(latency)
                    try? await Task.sleep(for: .milliseconds(500))
                }
                continuation.finish()
            }
        }
    }
    
    func streamRequestsPerSecond() -> AsyncStream<Double> {
        AsyncStream { continuation in
            Task {
                while !Task.isCancelled {
                    let rps = Double.random(in: 100...1000)
                    continuation.yield(rps)
                    try? await Task.sleep(for: .milliseconds(800))
                }
                continuation.finish()
            }
        }
    }
}

// MARK: - Dashboard ViewModel

@MainActor
class DashboardViewModel: ObservableObject {
    @Published var metrics: [MetricData] = []
    @Published var alerts: [SystemAlert] = []
    @Published var isLoading = false
    @Published var lastUpdated: Date = Date()
    
    private let dataSource = MetricDataSource()
    private var streamTasks: [Task<Void, Never>] = []
    
    // TaskLocal สำหรับ tracking
    @TaskLocal static var sessionId: String = "unknown"
    
    init() {
        setupMetrics()
    }
    
    private func setupMetrics() {
        metrics = [
            MetricData(name: "CPU Usage", unit: "%", threshold: 80),
            MetricData(name: "Memory", unit: "%", threshold: 85),
            MetricData(name: "Network Latency", unit: "ms", threshold: 150),
            MetricData(name: "Requests/sec", unit: "req/s", threshold: 800)
        ]
    }
    
    func startMonitoring() {
        isLoading = true
        
        // Start concurrent metric streams
        let cpuTask = Task {
            let stream = await dataSource.streamCPUUsage()
            for await value in stream {
                updateMetric(name: "CPU Usage", value: value)
                checkThreshold(metricName: "CPU Usage", value: value, threshold: 80)
            }
        }
        
        let memTask = Task {
            let stream = await dataSource.streamMemoryUsage()
            for await value in stream {
                updateMetric(name: "Memory", value: value)
                checkThreshold(metricName: "Memory", value: value, threshold: 85)
            }
        }
        
        let netTask = Task {
            let stream = await dataSource.streamNetworkLatency()
            for await value in stream {
                updateMetric(name: "Network Latency", value: value)
                checkThreshold(metricName: "Network Latency", value: value, threshold: 150)
            }
        }
        
        let reqTask = Task {
            let stream = await dataSource.streamRequestsPerSecond()
            for await value in stream {
                updateMetric(name: "Requests/sec", value: value)
            }
        }
        
        streamTasks = [cpuTask, memTask, netTask, reqTask]
        isLoading = false
    }
    
    private func updateMetric(name: String, value: Double) {
        guard let index = metrics.firstIndex(where: { $0.name == name }) else { return }
        
        metrics[index].value = value
        metrics[index].history.append(value)
        
        // เก็บแค่ 60 ค่าล่าสุด
        if metrics[index].history.count > 60 {
            metrics[index].history.removeFirst()
        }
        
        lastUpdated = Date()
    }
    
    private func checkThreshold(metricName: String, value: Double, threshold: Double) {
        if value > threshold {
            let alert = SystemAlert(
                message: "\(metricName) exceeded threshold: \(String(format: "%.1f", value))",
                severity: value > threshold * 1.2 ? .critical : .warning,
                timestamp: Date()
            )
            
            alerts.insert(alert, at: 0)
            
            // เก็บแค่ 50 alerts ล่าสุด
            if alerts.count > 50 {
                alerts.removeLast()
            }
        }
    }
    
    func stopMonitoring() {
        streamTasks.forEach { $0.cancel() }
        streamTasks.removeAll()
    }
    
    deinit {
        streamTasks.forEach { $0.cancel() }
    }
}

// MARK: - SwiftUI Views

struct DashboardView: View {
    @StateObject private var viewModel = DashboardViewModel()
    
    var body: some View {
        NavigationStack {
            ScrollView {
                VStack(spacing: 16) {
                    // Status header
                    HStack {
                        Circle()
                            .fill(Color.green)
                            .frame(width: 12, height: 12)
                        Text("System Online")
                            .font(.subheadline)
                        Spacer()
                        Text("Updated: \(viewModel.lastUpdated.formatted(.dateTime.hour().minute().second()))")
                            .font(.caption)
                            .foregroundColor(.secondary)
                    }
                    .padding(.horizontal)
                    
                    // Metrics Grid
                    LazyVGrid(columns: [GridItem(.flexible()), GridItem(.flexible())], spacing: 12) {
                        ForEach(viewModel.metrics) { metric in
                            MetricCard(metric: metric)
                        }
                    }
                    .padding(.horizontal)
                    
                    // Alerts
                    if !viewModel.alerts.isEmpty {
                        VStack(alignment: .leading, spacing: 8) {
                            Text("Recent Alerts")
                                .font(.headline)
                                .padding(.horizontal)
                            
                            ForEach(viewModel.alerts.prefix(5)) { alert in
                                AlertRow(alert: alert)
                            }
                        }
                    }
                }
                .padding(.vertical)
            }
            .navigationTitle("System Dashboard")
            .onAppear {
                viewModel.startMonitoring()
            }
            .onDisappear {
                viewModel.stopMonitoring()
            }
        }
    }
}

struct MetricCard: View {
    let metric: MetricData
    
    var color: Color {
        if metric.isOverThreshold {
            return .red
        } else if metric.value > metric.threshold * 0.8 {
            return .orange
        }
        return .green
    }
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            HStack {
                Text(metric.name)
                    .font(.caption)
                    .foregroundColor(.secondary)
                Spacer()
                Image(systemName: metric.trend > 0 ? "arrow.up.right" : "arrow.down.right")
                    .foregroundColor(metric.trend > 0 ? .red : .green)
                    .font(.caption)
            }
            
            HStack(alignment: .bottom) {
                Text(String(format: "%.1f", metric.value))
                    .font(.title2)
                    .fontWeight(.bold)
                    .foregroundColor(color)
                Text(metric.unit)
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
            
            // Mini chart
            GeometryReader { geo in
                HStack(alignment: .bottom, spacing: 1) {
                    ForEach(Array(metric.history.suffix(20).enumerated()), id: \.0) { _, val in
                        let height = max(2, geo.size.height * (val / 100.0))
                        Rectangle()
                            .fill(color.opacity(0.7))
                            .frame(width: max(1, (geo.size.width - 19) / 20), height: height)
                    }
                }
                .frame(maxWidth: .infinity, maxHeight: .infinity, alignment: .bottom)
            }
            .frame(height: 30)
        }
        .padding(12)
        .background(Color(.systemBackground))
        .cornerRadius(12)
        .shadow(color: .black.opacity(0.1), radius: 4, x: 0, y: 2)
    }
}

struct AlertRow: View {
    let alert: SystemAlert
    
    var iconName: String {
        switch alert.severity {
        case .info: return "info.circle.fill"
        case .warning: return "exclamationmark.triangle.fill"
        case .critical: return "xmark.octagon.fill"
        }
    }
    
    var iconColor: Color {
        switch alert.severity {
        case .info: return .blue
        case .warning: return .orange
        case .critical: return .red
        }
    }
    
    var body: some View {
        HStack(spacing: 12) {
            Image(systemName: iconName)
                .foregroundColor(iconColor)
            
            VStack(alignment: .leading, spacing: 2) {
                Text(alert.message)
                    .font(.subheadline)
                Text(alert.timestamp.formatted(.relative(presentation: .named)))
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
            
            Spacer()
        }
        .padding(.horizontal)
        .padding(.vertical, 8)
        .background(iconColor.opacity(0.05))
    }
}
```

---

## 15. Exercises กับ Solutions

### Exercise 1: Thread-Safe Cache

**โจทย์**: สร้าง thread-safe cache ที่รองรับ:
- Set/get values โดย key
- TTL (Time-To-Live) สำหรับแต่ละ entry
- Auto-eviction เมื่อ TTL หมด
- Maximum size พร้อม LRU eviction

```swift
// Solution
actor TTLCache<Key: Hashable, Value: Sendable> {
    private struct CacheEntry {
        let value: Value
        let expiresAt: ContinuousClock.Instant
        var accessedAt: ContinuousClock.Instant
    }
    
    private var storage: [Key: CacheEntry] = [:]
    private let maxSize: Int
    private let clock = ContinuousClock()
    
    init(maxSize: Int) {
        self.maxSize = maxSize
    }
    
    func set(_ value: Value, forKey key: Key, ttl: Duration) {
        evictExpired()
        
        let entry = CacheEntry(
            value: value,
            expiresAt: clock.now.advanced(by: ttl),
            accessedAt: clock.now
        )
        
        if storage.count >= maxSize && storage[key] == nil {
            evictLRU()
        }
        
        storage[key] = entry
    }
    
    func get(_ key: Key) -> Value? {
        guard var entry = storage[key] else { return nil }
        
        if entry.expiresAt <= clock.now {
            storage.removeValue(forKey: key)
            return nil
        }
        
        entry.accessedAt = clock.now
        storage[key] = entry
        return entry.value
    }
    
    private func evictExpired() {
        let now = clock.now
        storage = storage.filter { $0.value.expiresAt > now }
    }
    
    private func evictLRU() {
        guard let lruKey = storage.min(by: { $0.value.accessedAt < $1.value.accessedAt })?.key else { return }
        storage.removeValue(forKey: lruKey)
    }
    
    var count: Int { storage.count }
}

// Test
func testTTLCache() async throws {
    let cache = TTLCache<String, String>(maxSize: 5)
    
    await cache.set("hello", forKey: "greeting", ttl: .seconds(2))
    
    let value1 = await cache.get("greeting")
    assert(value1 == "hello", "Should find value")
    
    try await Task.sleep(for: .seconds(3))
    
    let value2 = await cache.get("greeting")
    assert(value2 == nil, "Should be expired")
    
    print("✅ TTL Cache test passed")
}
```

### Exercise 2: Concurrent File Processor

**โจทย์**: ประมวลผล files หลายไฟล์พร้อมกัน พร้อม progress reporting

```swift
// Solution
actor FileProcessor {
    private var processedCount = 0
    private var totalCount = 0
    private var errors: [String: Error] = [:]
    
    let progressStream: AsyncStream<Double>
    private let progressContinuation: AsyncStream<Double>.Continuation
    
    init() {
        var cont: AsyncStream<Double>.Continuation!
        progressStream = AsyncStream { cont = $0 }
        progressContinuation = cont!
    }
    
    func processFiles(_ urls: [URL], maxConcurrent: Int = 4) async throws -> [URL: String] {
        totalCount = urls.count
        processedCount = 0
        
        var results: [URL: String] = [:]
        
        try await withThrowingTaskGroup(of: (URL, String)?.self) { group in
            var inFlight = 0
            var iterator = urls.makeIterator()
            
            // เติม initial batch
            while inFlight < maxConcurrent, let url = iterator.next() {
                group.addTask { [weak self] in
                    return try await self?.processFile(url)
                }
                inFlight += 1
            }
            
            // Process results และ add more tasks
            for try await result in group {
                inFlight -= 1
                
                if let (url, content) = result {
                    results[url] = content
                    await updateProgress()
                }
                
                // Add next task
                if let url = iterator.next() {
                    group.addTask { [weak self] in
                        return try await self?.processFile(url)
                    }
                    inFlight += 1
                }
            }
        }
        
        progressContinuation.finish()
        return results
    }
    
    private func processFile(_ url: URL) async throws -> (URL, String) {
        // simulate file processing
        try await Task.sleep(for: .milliseconds(Double.random(in: 50...200)))
        
        let content = "Processed: \(url.lastPathComponent)"
        return (url, content)
    }
    
    private func updateProgress() {
        processedCount += 1
        let progress = Double(processedCount) / Double(totalCount)
        progressContinuation.yield(progress)
    }
}

// ใช้งาน
func demonstrateFileProcessor() async throws {
    let urls = (1...20).map { URL(fileURLWithPath: "/tmp/file\($0).txt") }
    let processor = FileProcessor()
    
    // Monitor progress
    Task {
        for await progress in await processor.progressStream {
            print("Progress: \(Int(progress * 100))%")
        }
    }
    
    let results = try await processor.processFiles(urls, maxConcurrent: 4)
    print("Processed \(results.count) files")
}
```

### Exercise 3: Async Event Bus

**โจทย์**: สร้าง event bus ที่ subscribers สามารถ subscribe/unsubscribe ได้ และ events ถูก deliver แบบ async

```swift
// Solution
actor AsyncEventBus<Event: Sendable> {
    private var subscribers: [UUID: AsyncStream<Event>.Continuation] = [:]
    
    func subscribe() -> (id: UUID, stream: AsyncStream<Event>) {
        let id = UUID()
        var continuation: AsyncStream<Event>.Continuation!
        
        let stream = AsyncStream<Event>(bufferingPolicy: .bufferingNewest(100)) { cont in
            continuation = cont
        }
        
        subscribers[id] = continuation
        
        return (id, stream)
    }
    
    func unsubscribe(id: UUID) {
        subscribers[id]?.finish()
        subscribers.removeValue(forKey: id)
    }
    
    func publish(_ event: Event) {
        for (_, continuation) in subscribers {
            continuation.yield(event)
        }
    }
    
    func publishToAll(_ events: [Event]) {
        for event in events {
            publish(event)
        }
    }
    
    var subscriberCount: Int { subscribers.count }
}

// ตัวอย่างการใช้งาน
enum AppEvent: Sendable {
    case userLoggedIn(String)
    case userLoggedOut
    case dataUpdated(String)
    case errorOccurred(String)
}

func eventBusExample() async {
    let bus = AsyncEventBus<AppEvent>()
    
    // Subscribe
    let (id1, stream1) = await bus.subscribe()
    let (id2, stream2) = await bus.subscribe()
    
    // Listen in background
    Task {
        for await event in stream1 {
            print("Subscriber 1 received: \(event)")
        }
    }
    
    Task {
        for await event in stream2 {
            print("Subscriber 2 received: \(event)")
        }
        // จะหยุดเมื่อ unsubscribe
    }
    
    // Publish events
    await bus.publish(.userLoggedIn("Alice"))
    await bus.publish(.dataUpdated("items"))
    
    // Unsubscribe
    await bus.unsubscribe(id: id2)
    
    await bus.publish(.userLoggedOut)
    // Subscriber 2 จะไม่ได้รับ event นี้
    
    // Cleanup
    await bus.unsubscribe(id: id1)
}
```

---

## สรุป

ใน Part 93 นี้เราได้เรียนรู้:

1. **Swift Concurrency Model**: Cooperative thread pool, suspension points, continuations, executors
2. **Actor Isolation**: Reentrancy, nonisolated, @preconcurrency, Sendable
3. **Custom Executors**: SerialExecutor, actor integration, MainActor internals
4. **Distributed Actors**: ActorSystem, remote invocation, use cases
5. **Structured/Unstructured Concurrency**: Task hierarchy, detached tasks, task groups
6. **Advanced AsyncSequence**: Custom sequences, AsyncStream, back-pressure
7. **TaskLocal**: Context propagation, distributed tracing
8. **Clock Protocol**: ContinuousClock, SuspendingClock, testing with manual clocks
9. **Async Algorithms**: debounce, throttle, merge, zip, buffer
10. **Core Data Concurrency**: perform async/await, background context patterns
11. **URLSession Concurrency**: Task groups, parallel downloads, rate limiting
12. **Testing Async Code**: Swift Testing, XCTest, clock testing, cancellation testing
13. **Common Bugs**: Data races, deadlocks, task leaks
14. **Real-World App**: Complete dashboard with concurrent data streams
15. **Exercises**: Practical problems with complete solutions

ใน Part 94 เราจะเรียนรู้เกี่ยวกับ App Performance Optimization ขั้นสูง
