# Part 53: Advanced Concurrency ใน Swift

## บทนำ

ใน Part ก่อนหน้า เราได้เรียนรู้พื้นฐานของ `async/await` และ `Actor` ใน Swift แล้ว ใน Part นี้เราจะเจาะลึกลงไปในหัวข้อขั้นสูง ที่ช่วยให้คุณสามารถเขียนโปรแกรม Concurrent ที่มีประสิทธิภาพ ปลอดภัย และง่ายต่อการทดสอบ

---

## 1. Advanced Task Patterns

### 1.1 Task Group แบบ Dynamic

`TaskGroup` ช่วยให้เราสามารถรัน Task หลายตัวพร้อมกันและรวบรวมผลลัพธ์ได้

```swift
import Foundation

// ตัวอย่างการดาวน์โหลดข้อมูลจาก URLs หลายตัวพร้อมกัน
func downloadAllData(from urls: [URL]) async throws -> [Data] {
    try await withThrowingTaskGroup(of: (Int, Data).self) { group in
        // เพิ่ม Task สำหรับแต่ละ URL
        for (index, url) in urls.enumerated() {
            group.addTask {
                let (data, _) = try await URLSession.shared.data(from: url)
                return (index, data)
            }
        }
        
        // รวบรวมผลลัพธ์โดยรักษาลำดับ
        var results = Array(repeating: Data(), count: urls.count)
        for try await (index, data) in group {
            results[index] = data
        }
        return results
    }
}

// ใช้งาน
Task {
    let urls = [
        URL(string: "https://api.example.com/data/1")!,
        URL(string: "https://api.example.com/data/2")!,
        URL(string: "https://api.example.com/data/3")!
    ]
    
    do {
        let allData = try await downloadAllData(from: urls)
        print("ดาวน์โหลดสำเร็จ: \(allData.count) รายการ")
    } catch {
        print("เกิดข้อผิดพลาด: \(error)")
    }
}
```

### 1.2 Discarding Task Group

Swift 5.9 เพิ่ม `withDiscardingTaskGroup` ที่ไม่เก็บผลลัพธ์ของ Child Task — เหมาะสำหรับงานที่ไม่ต้องการ Return value

```swift
// ตัวอย่างการส่ง Notification ไปยังหลาย Endpoint พร้อมกัน
func notifyAllEndpoints(message: String, endpoints: [URL]) async {
    await withDiscardingTaskGroup { group in
        for endpoint in endpoints {
            group.addTask {
                do {
                    var request = URLRequest(url: endpoint)
                    request.httpMethod = "POST"
                    request.httpBody = message.data(using: .utf8)
                    _ = try await URLSession.shared.data(for: request)
                    print("ส่งสำเร็จไปยัง: \(endpoint)")
                } catch {
                    print("ส่งไม่สำเร็จไปยัง: \(endpoint) - \(error)")
                }
            }
        }
    }
    print("ส่งทั้งหมดเสร็จสิ้น")
}
```

### 1.3 Structured vs Unstructured Concurrency

```swift
// Structured Concurrency - ใช้ async let
func fetchUserProfile(userId: String) async throws -> UserProfile {
    // รัน 3 งานพร้อมกันแบบ Structured
    async let userInfo = fetchUserInfo(userId: userId)
    async let userPosts = fetchUserPosts(userId: userId)
    async let userFollowers = fetchFollowers(userId: userId)
    
    // รอทั้งหมดพร้อมกัน
    let (info, posts, followers) = try await (userInfo, userPosts, userFollowers)
    
    return UserProfile(info: info, posts: posts, followers: followers)
}

struct UserProfile {
    let info: UserInfo
    let posts: [Post]
    let followers: [User]
}

struct UserInfo { let name: String }
struct Post { let content: String }
struct User { let id: String }

func fetchUserInfo(userId: String) async throws -> UserInfo {
    try await Task.sleep(nanoseconds: 100_000_000)
    return UserInfo(name: "สมชาย ใจดี")
}

func fetchUserPosts(userId: String) async throws -> [Post] {
    try await Task.sleep(nanoseconds: 200_000_000)
    return [Post(content: "Hello Swift!")]
}

func fetchFollowers(userId: String) async throws -> [User] {
    try await Task.sleep(nanoseconds: 150_000_000)
    return [User(id: "user_123")]
}
```

---

## 2. Task Priority

Task Priority กำหนดว่า Task ไหนจะได้รับ CPU Time ก่อน

```swift
import Foundation

// ลำดับความสำคัญจากสูงไปต่ำ:
// .high > .medium > .low > .background > .utility

// สร้าง Task ด้วย Priority ต่างๆ
func demonstrateTaskPriority() {
    // High priority - ใช้สำหรับงานที่ต้องการ Response เร็ว
    Task(priority: .high) {
        print("High priority task เริ่มทำงาน")
        await performCriticalWork()
    }
    
    // Medium priority - ค่าเริ่มต้น
    Task(priority: .medium) {
        print("Medium priority task เริ่มทำงาน")
        await performNormalWork()
    }
    
    // Background priority - ใช้สำหรับงาน Sync ที่ไม่เร่งด่วน
    Task(priority: .background) {
        print("Background task เริ่มทำงาน")
        await performBackgroundSync()
    }
}

func performCriticalWork() async {
    // จำลองงานสำคัญ
    try? await Task.sleep(nanoseconds: 50_000_000)
    print("งานสำคัญเสร็จแล้ว")
}

func performNormalWork() async {
    try? await Task.sleep(nanoseconds: 100_000_000)
    print("งานปกติเสร็จแล้ว")
}

func performBackgroundSync() async {
    try? await Task.sleep(nanoseconds: 200_000_000)
    print("งาน Background เสร็จแล้ว")
}

// Priority Escalation - Swift จะยก Priority ของ Child Task ให้เท่า Parent
Task(priority: .high) {
    // Child task นี้จะถูก Escalate เป็น .high อัตโนมัติ
    await Task(priority: .background) {
        // แม้ว่าจะระบุ .background แต่ Swift อาจ Escalate เป็น .high
        print("Current priority: \(Task.currentPriority)")
    }.value
}
```

### 2.1 Task Priority ใน Real-World Scenario

```swift
class ImageProcessor {
    func processImages(_ images: [UIImage]) async -> [ProcessedImage] {
        await withTaskGroup(of: ProcessedImage?.self) { group in
            for (index, image) in images.enumerated() {
                // กำหนด Priority ตามตำแหน่ง - รูปแรกๆ สำคัญกว่า
                let priority: TaskPriority = index < 3 ? .high : .background
                
                group.addTask(priority: priority) {
                    await self.processImage(image)
                }
            }
            
            var processed: [ProcessedImage] = []
            for await result in group {
                if let image = result {
                    processed.append(image)
                }
            }
            return processed
        }
    }
    
    private func processImage(_ image: UIImage) async -> ProcessedImage? {
        // จำลองการประมวลผลรูปภาพ
        try? await Task.sleep(nanoseconds: 100_000_000)
        return ProcessedImage(originalSize: image.size)
    }
}

struct ProcessedImage {
    let originalSize: CGSize
}

// สร้าง Stub UIImage สำหรับตัวอย่าง
class UIImage {
    let size: CGSize
    init(size: CGSize = CGSize(width: 100, height: 100)) {
        self.size = size
    }
}

struct CGSize {
    let width: Double
    let height: Double
}
```

---

## 3. Task Local Values

`TaskLocal` ช่วยให้เราสามารถส่งข้อมูลผ่าน Task Tree โดยไม่ต้องส่งผ่าน Parameter

### 3.1 พื้นฐาน TaskLocal

```swift
import Foundation

// ประกาศ TaskLocal ด้วย Property Wrapper @TaskLocal
enum RequestContext {
    @TaskLocal static var requestId: String = "default"
    @TaskLocal static var userId: String? = nil
    @TaskLocal static var traceId: UUID = UUID()
}

// ใช้งาน TaskLocal
func handleRequest() async {
    // กำหนดค่า TaskLocal สำหรับ Task นี้และ Child Tasks ทั้งหมด
    await RequestContext.$requestId.withValue("req_12345") {
        await RequestContext.$userId.withValue("user_67890") {
            print("กำลังประมวลผล Request ID: \(RequestContext.requestId)")
            print("สำหรับ User: \(RequestContext.userId ?? "ไม่ระบุ")")
            
            // Child Task จะสืบทอดค่า TaskLocal
            await processRequest()
        }
    }
}

func processRequest() async {
    // สามารถอ่านค่า TaskLocal ได้โดยตรง
    let reqId = RequestContext.requestId
    let userId = RequestContext.userId
    
    print("กำลังประมวลผลใน processRequest:")
    print("  Request ID: \(reqId)")
    print("  User ID: \(userId ?? "ไม่ระบุ")")
    
    // Task ใหม่ที่สร้างใน Scope นี้ก็จะได้รับค่าด้วย
    async let step1 = performStep1()
    async let step2 = performStep2()
    await (step1, step2)
}

func performStep1() async {
    print("Step 1 - Request ID: \(RequestContext.requestId)")
}

func performStep2() async {
    print("Step 2 - Request ID: \(RequestContext.requestId)")
}
```

### 3.2 TaskLocal สำหรับ Logging

```swift
// ระบบ Logging ที่ใช้ TaskLocal เพื่อติดตาม Context
struct Logger {
    @TaskLocal static var context: LogContext = LogContext()
    
    static func log(_ message: String, level: LogLevel = .info) {
        let ctx = context
        let timestamp = ISO8601DateFormatter().string(from: Date())
        print("[\(timestamp)] [\(level)] [req:\(ctx.requestId)] [user:\(ctx.userId ?? "-")] \(message)")
    }
}

struct LogContext {
    var requestId: String = UUID().uuidString
    var userId: String? = nil
    var serviceName: String = "main"
}

enum LogLevel: String {
    case debug = "DEBUG"
    case info = "INFO"
    case warning = "WARN"
    case error = "ERROR"
}

// ใช้งาน
func handleAPIRequest(requestId: String, userId: String) async {
    let context = LogContext(requestId: requestId, userId: userId, serviceName: "api")
    
    await Logger.$context.withValue(context) {
        Logger.log("เริ่มประมวลผล API Request")
        
        do {
            let result = try await processAPIRequest()
            Logger.log("สำเร็จ: \(result)")
        } catch {
            Logger.log("เกิดข้อผิดพลาด: \(error)", level: .error)
        }
    }
}

func processAPIRequest() async throws -> String {
    Logger.log("กำลัง Validate ข้อมูล")
    try await Task.sleep(nanoseconds: 100_000_000)
    
    Logger.log("กำลัง Query ฐานข้อมูล")
    try await Task.sleep(nanoseconds: 200_000_000)
    
    return "ผลลัพธ์ API"
}
```

---

## 4. TaskLocal Property Wrapper

`@TaskLocal` เป็น Property Wrapper พิเศษที่ใช้กับ Static Properties เท่านั้น

```swift
// ข้อกำหนดของ @TaskLocal:
// 1. ต้องเป็น static property
// 2. ต้องมีค่า Default
// 3. ค่าจะถูก "Inherit" โดย Child Tasks
// 4. การเปลี่ยนค่าทำผ่าน withValue(_:operation:) เท่านั้น

// ตัวอย่างที่ซับซ้อนขึ้น - Feature Flags
enum FeatureFlags {
    @TaskLocal static var isExperimentalFeatureEnabled: Bool = false
    @TaskLocal static var maxRetryCount: Int = 3
    @TaskLocal static var timeoutInterval: TimeInterval = 30.0
}

class NetworkService {
    func fetchData(from url: URL) async throws -> Data {
        var lastError: Error?
        let maxRetries = FeatureFlags.maxRetryCount
        let timeout = FeatureFlags.timeoutInterval
        
        for attempt in 1...maxRetries {
            do {
                // ใช้ค่าจาก TaskLocal
                let config = URLSessionConfiguration.default
                config.timeoutIntervalForRequest = timeout
                let session = URLSession(configuration: config)
                
                let (data, _) = try await session.data(from: url)
                return data
            } catch {
                lastError = error
                print("Attempt \(attempt) ล้มเหลว: \(error)")
                
                if attempt < maxRetries {
                    try await Task.sleep(nanoseconds: UInt64(attempt) * 1_000_000_000)
                }
            }
        }
        
        throw lastError ?? NSError(domain: "NetworkError", code: -1)
    }
}

// ใช้ Feature Flag ที่แตกต่างกันสำหรับ Test
Task {
    // Production settings
    let service = NetworkService()
    
    // Override สำหรับ Testing
    await FeatureFlags.$maxRetryCount.withValue(1) {
        await FeatureFlags.$timeoutInterval.withValue(5.0) {
            // ใน Scope นี้จะ Retry แค่ 1 ครั้งและ Timeout ใน 5 วินาที
            do {
                let data = try await service.fetchData(from: URL(string: "https://api.example.com")!)
                print("ได้รับข้อมูล: \(data.count) bytes")
            } catch {
                print("ล้มเหลว: \(error)")
            }
        }
    }
}
```

---

## 5. Actor Reentrancy

Actor Reentrancy เป็นปรากฏการณ์ที่ Actor สามารถรับ Message ใหม่ได้ในขณะที่กำลัง Await อยู่

```swift
// ปัญหาของ Actor Reentrancy
actor BankAccount {
    private var balance: Double = 1000.0
    
    // ⚠️ ปัญหา: Reentrancy อาจทำให้ State ไม่ Consistent
    func transferWithReentrancyBug(amount: Double, to target: BankAccount) async throws {
        guard balance >= amount else {
            throw BankError.insufficientFunds
        }
        
        // ⚠️ ระหว่าง await นี้ Actor อาจรับ Message อื่น
        // ทำให้ balance อาจเปลี่ยนไปก่อนที่เราจะหัก
        await target.deposit(amount: amount)
        
        // ตอนนี้ balance อาจน้อยกว่า amount แล้ว!
        balance -= amount  // อาจเป็นลบได้!
    }
    
    // ✅ วิธีที่ถูกต้อง: หักเงินก่อน แล้วค่อย await
    func transferSafely(amount: Double, to target: BankAccount) async throws {
        guard balance >= amount else {
            throw BankError.insufficientFunds
        }
        
        // หักเงินก่อนที่จะ await
        balance -= amount
        
        // ตอนนี้แม้จะมี Reentrant Call, balance ก็ถูกแล้ว
        await target.deposit(amount: amount)
    }
    
    func deposit(amount: Double) async {
        balance += amount
        print("ฝากเงิน \(amount) บาท ยอดคงเหลือ: \(balance) บาท")
    }
    
    func getBalance() -> Double {
        return balance
    }
}

enum BankError: Error {
    case insufficientFunds
}

// การป้องกัน Reentrancy ด้วย Nonisolated + Task
actor Cache<Key: Hashable, Value> {
    private var storage: [Key: Value] = [:]
    private var pendingRequests: [Key: Task<Value, Error>] = [:]
    
    // ป้องกัน Duplicate Requests สำหรับ Key เดียวกัน
    func value(for key: Key, compute: @Sendable @escaping () async throws -> Value) async throws -> Value {
        // ถ้ามีค่าใน Cache แล้ว
        if let cached = storage[key] {
            return cached
        }
        
        // ถ้ากำลัง Compute อยู่แล้ว รอผลลัพธ์นั้น
        if let pending = pendingRequests[key] {
            return try await pending.value
        }
        
        // สร้าง Task ใหม่
        let task = Task {
            try await compute()
        }
        pendingRequests[key] = task
        
        do {
            let value = try await task.value
            storage[key] = value
            pendingRequests.removeValue(forKey: key)
            return value
        } catch {
            pendingRequests.removeValue(forKey: key)
            throw error
        }
    }
    
    func invalidate(key: Key) {
        storage.removeValue(forKey: key)
    }
}
```

---

## 6. Actor Hop

Actor Hop คือการ "กระโดด" ระหว่าง Actor Contexts ซึ่งมีค่าใช้จ่ายด้าน Performance

```swift
// Actor Hop เกิดขึ้นทุกครั้งที่เรา await method ของ Actor อื่น
actor Database {
    func fetchUser(id: String) async -> User? {
        // จำลองการ Query
        try? await Task.sleep(nanoseconds: 10_000_000)
        return User(id: id)
    }
    
    func updateUser(_ user: User) async {
        // จำลองการ Update
        try? await Task.sleep(nanoseconds: 5_000_000)
        print("อัปเดต User: \(user.id)")
    }
}

actor Analytics {
    func track(event: String, userId: String) async {
        // จำลองการส่ง Event
        print("Track: \(event) for \(userId)")
    }
}

struct UserData {
    let id: String
    let name: String
}

// ⚠️ หลาย Actor Hops - ไม่ดี
class UserService_Bad {
    let db = Database()
    let analytics = Analytics()
    
    func updateUserProfile(userId: String, newName: String) async {
        // Hop 1: ไป Database
        guard let user = await db.fetchUser(id: userId) else { return }
        
        // Hop 2: กลับมา
        let updatedUser = User(id: user.id, name: newName)
        
        // Hop 3: ไป Database อีกครั้ง
        await db.updateUser(updatedUser)
        
        // Hop 4: ไป Analytics
        await analytics.track(event: "profile_updated", userId: userId)
        
        // แต่ละ Hop มีค่าใช้จ่าย Context Switch!
    }
}

// ✅ ลด Actor Hops โดย Batch Operations
actor UserService_Good {
    let db = Database()
    let analytics = Analytics()
    
    // รวมงานที่เกี่ยวข้องไว้ใน Actor เดียวกัน
    func batchUpdateAndTrack(userId: String, newName: String) async {
        // Hop เดียวไป Database
        guard let user = await db.fetchUser(id: userId) else { return }
        let updatedUser = User(id: user.id, name: newName)
        await db.updateUser(updatedUser)
        // Hop เดียวไป Analytics
        await analytics.track(event: "profile_updated", userId: userId)
    }
}

struct User: Sendable {
    let id: String
    var name: String = "ผู้ใช้งาน"
    init(id: String, name: String = "ผู้ใช้งาน") {
        self.id = id
        self.name = name
    }
}
```

---

## 7. Distributed Actors

Distributed Actors (Swift 5.7+) ช่วยให้สามารถสื่อสารระหว่างกระบวนการหรือเครื่องต่างๆ ได้

```swift
import Distributed

// Distributed Actor ต้องมี ActorSystem
// ใน Production จะใช้ Framework อย่าง Swift Distributed Actors Cluster

// ตัวอย่าง Simple Distributed Actor
distributed actor GreetingService {
    // Distributed Actor ต้องมี actorSystem property
    typealias ActorSystem = LocalTestingDistributedActorSystem
    
    distributed func greet(name: String) -> String {
        return "สวัสดี, \(name)! จาก Distributed Actor"
    }
    
    distributed func processData(_ items: [Int]) async -> Int {
        return items.reduce(0, +)
    }
}

// ใช้งาน Distributed Actor
// หมายเหตุ: ต้องมี ActorSystem ที่ Configured แล้ว
func useDistributedActor() async throws {
    let system = LocalTestingDistributedActorSystem()
    let service = GreetingService(actorSystem: system)
    
    // เรียกใช้ Distributed Method - ดูเหมือน Local แต่อาจทำงาน Remote
    let greeting = try await service.greet(name: "สมชาย")
    print(greeting)
    
    let sum = try await service.processData([1, 2, 3, 4, 5])
    print("ผลรวม: \(sum)")
}
```

---

## 8. Executor Protocols

Executor เป็น Low-level Mechanism ที่กำหนดว่า Task จะรันบน Thread ไหน

```swift
// Executor Protocol ใน Swift
// public protocol Executor: AnyObject, Sendable {
//     func enqueue(_ job: consuming ExecutorJob)
// }

// public protocol SerialExecutor: Executor {
//     func asUnownedSerialExecutor() -> UnownedSerialExecutor
//     func isSameExclusiveExecutionContext(other executor: Self) -> Bool
// }

// ตัวอย่างการใช้งาน Executor
actor MyActor {
    // ระบุ Custom Executor (Swift 5.9+)
    nonisolated var unownedExecutor: UnownedSerialExecutor {
        // ใช้ MainActor's executor
        MainActor.sharedUnownedExecutor
    }
    
    func doWork() {
        // งานนี้จะรันบน Main Thread เสมอ
        print("รันบน Main Thread: \(Thread.isMainThread)")
    }
}

// Serial Executor สำหรับรันงานบน Specific Queue
final class DispatchQueueExecutor: SerialExecutor {
    private let queue: DispatchQueue
    
    init(label: String, qos: DispatchQoS = .default) {
        self.queue = DispatchQueue(label: label, qos: qos)
    }
    
    func enqueue(_ job: consuming ExecutorJob) {
        let job = UnownedJob(job)
        queue.async {
            job.runSynchronously(on: self.asUnownedSerialExecutor())
        }
    }
    
    func asUnownedSerialExecutor() -> UnownedSerialExecutor {
        UnownedSerialExecutor(ordinary: self)
    }
}
```

---

## 9. Custom Executors (Swift 5.9+)

Custom Executors ช่วยให้เรากำหนดได้ว่า Actor จะรันบน Thread หรือ Queue ไหน

```swift
import Foundation

// Custom Executor ที่ใช้ DispatchQueue
final class SerialQueueExecutor: SerialExecutor, @unchecked Sendable {
    let queue: DispatchQueue
    
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

// Actor ที่ใช้ Custom Executor
actor DatabaseActor {
    private let executor = SerialQueueExecutor(label: "com.app.database")
    
    nonisolated var unownedExecutor: UnownedSerialExecutor {
        executor.asUnownedSerialExecutor()
    }
    
    private var records: [String: String] = [:]
    
    func insert(key: String, value: String) {
        records[key] = value
        print("Insert ใน thread: \(Thread.current.name ?? "unknown")")
    }
    
    func fetch(key: String) -> String? {
        return records[key]
    }
}

// ใช้งาน
Task {
    let db = DatabaseActor()
    await db.insert(key: "name", value: "สมชาย")
    let value = await db.fetch(key: "name")
    print("ค่าที่ได้: \(value ?? "ไม่พบ")")
}

// TaskExecutor (Swift 6.0+) - รัน Task ทั้งหมดบน Executor ที่กำหนด
// Task(executorPreference: myExecutor) {
//     // Task นี้จะรันบน myExecutor
// }
```

---

## 10. SerialExecutor

`SerialExecutor` รับประกันว่างานจะรันทีละงานโดยไม่มี Concurrent Execution

```swift
// ตัวอย่าง Custom SerialExecutor สำหรับ I/O Operations
final class IOExecutor: SerialExecutor, @unchecked Sendable {
    private let thread: Thread
    private let queue: OperationQueue
    
    init() {
        queue = OperationQueue()
        queue.maxConcurrentOperationCount = 1
        queue.name = "com.app.io-executor"
        queue.qualityOfService = .utility
        thread = Thread.current  // จะ Update ใน Production
    }
    
    func enqueue(_ job: consuming ExecutorJob) {
        let unownedJob = UnownedJob(job)
        queue.addOperation {
            unownedJob.runSynchronously(on: self.asUnownedSerialExecutor())
        }
    }
    
    func asUnownedSerialExecutor() -> UnownedSerialExecutor {
        UnownedSerialExecutor(ordinary: self)
    }
}

// ใช้ SerialExecutor สำหรับ File Operations
actor FileManager_Custom {
    private let ioExecutor = IOExecutor()
    
    nonisolated var unownedExecutor: UnownedSerialExecutor {
        ioExecutor.asUnownedSerialExecutor()
    }
    
    func readFile(at path: String) async throws -> String {
        // งานนี้จะรันบน IO Thread เสมอ
        try String(contentsOfFile: path, encoding: .utf8)
    }
    
    func writeFile(content: String, to path: String) async throws {
        // งานนี้จะรันบน IO Thread เสมอ
        try content.write(toFile: path, atomically: true, encoding: .utf8)
    }
}
```

---

## 11. MainActor Optimization

`@MainActor` รับประกันว่าโค้ดจะรันบน Main Thread ซึ่งจำเป็นสำหรับ UI Updates

```swift
import Foundation

// @MainActor บน Class ทั้งหมด
@MainActor
class ViewController {
    var title: String = "หน้าหลัก"
    var isLoading: Bool = false
    
    // Method ทั้งหมดรันบน Main Thread โดยอัตโนมัติ
    func updateUI() {
        title = "กำลังโหลด..."
        isLoading = true
        print("อัปเดต UI บน Main Thread: \(Thread.isMainThread)")
    }
    
    func loadData() async {
        updateUI()  // รันบน Main Thread
        
        // ออกจาก Main Thread เพื่อทำงาน Background
        let data = await Task.detached(priority: .background) {
            await self.fetchDataFromAPI()
        }.value
        
        // กลับมาที่ Main Thread อัตโนมัติเพราะ ViewController เป็น @MainActor
        isLoading = false
        title = "โหลดเสร็จแล้ว (\(data.count) รายการ)"
    }
    
    nonisolated func fetchDataFromAPI() async -> [String] {
        // ทำงานบน Background Thread
        try? await Task.sleep(nanoseconds: 1_000_000_000)
        return ["item1", "item2", "item3"]
    }
}

// Optimization: ลด MainActor Hops
@MainActor
class OptimizedViewController {
    var items: [String] = []
    
    // ✅ ดีกว่า: รวม UI Updates เข้าด้วยกัน
    func loadAndUpdate() async {
        // เตรียม Loading State
        items = []
        print("เริ่มโหลด")
        
        // ออกไป Background ครั้งเดียว
        let newItems = await loadItemsInBackground()
        
        // กลับมา Update UI ครั้งเดียว
        items = newItems
        print("โหลดเสร็จ: \(items.count) รายการ")
    }
    
    nonisolated func loadItemsInBackground() async -> [String] {
        try? await Task.sleep(nanoseconds: 500_000_000)
        return Array(repeating: "รายการ", count: 10)
    }
}
```

---

## 12. Cooperative Cancellation

Cooperative Cancellation คือระบบที่ Task ต้องตรวจสอบและตอบสนองต่อการ Cancel เอง

```swift
import Foundation

// การตรวจสอบ Cancellation
func processLargeDataset(_ items: [Int]) async throws -> [Int] {
    var results: [Int] = []
    
    for item in items {
        // ตรวจสอบการ Cancel ก่อนทำงานแต่ละชิ้น
        try Task.checkCancellation()
        
        let processed = await processItem(item)
        results.append(processed)
    }
    
    return results
}

func processItem(_ item: Int) async -> Int {
    try? await Task.sleep(nanoseconds: 10_000_000)
    return item * 2
}

// ใช้ withTaskCancellationHandler
func downloadWithCancellation(url: URL) async throws -> Data {
    try await withTaskCancellationHandler {
        // งานหลัก
        let (data, _) = try await URLSession.shared.data(from: url)
        return data
    } onCancel: {
        // จะถูกเรียกเมื่อ Task ถูก Cancel
        URLSession.shared.invalidateAndCancel()
        print("Download ถูก Cancel")
    }
}

// การ Cancel Task จากภายนอก
func demonstrateCancellation() async {
    let task = Task {
        do {
            let items = Array(1...1000)
            let results = try await processLargeDataset(items)
            print("ประมวลผลเสร็จ: \(results.count) รายการ")
        } catch is CancellationError {
            print("Task ถูก Cancel")
        } catch {
            print("เกิดข้อผิดพลาด: \(error)")
        }
    }
    
    // Cancel หลังจาก 100ms
    try? await Task.sleep(nanoseconds: 100_000_000)
    task.cancel()
    
    // รอให้ Task จัดการการ Cancel เสร็จ
    await task.value
}

// Custom Cancellable Resource
actor CancellableDownloader {
    private var activeTask: Task<Data, Error>?
    
    func download(from url: URL) async throws -> Data {
        // Cancel Task เก่าถ้ามี
        activeTask?.cancel()
        
        let task = Task {
            try await withTaskCancellationHandler {
                var request = URLRequest(url: url)
                request.timeoutInterval = 30
                let (data, _) = try await URLSession.shared.data(for: request)
                return data
            } onCancel: {
                print("กำลัง Cancel download จาก \(url)")
            }
        }
        
        activeTask = task
        
        do {
            let data = try await task.value
            activeTask = nil
            return data
        } catch {
            activeTask = nil
            throw error
        }
    }
    
    func cancelCurrentDownload() {
        activeTask?.cancel()
        activeTask = nil
    }
}
```

---

## 13. AsyncChannel (Swift Async Algorithms)

`AsyncChannel` จาก Package `swift-async-algorithms` ช่วยให้สามารถส่งข้อมูล Async ระหว่าง Task ได้

```swift
// ต้องเพิ่ม Package: swift-async-algorithms
// import AsyncAlgorithms

// ตัวอย่าง Conceptual ของ AsyncChannel (เพราะต้องใช้ Package เพิ่มเติม)
// ในโค้ดจริงต้อง import AsyncAlgorithms

// จำลอง AsyncChannel เพื่อแสดงแนวคิด
final class SimpleAsyncChannel<Element: Sendable>: AsyncSequence, @unchecked Sendable {
    typealias AsyncIterator = AsyncStream<Element>.AsyncIterator
    
    private let stream: AsyncStream<Element>
    private let continuation: AsyncStream<Element>.Continuation
    
    init() {
        var cont: AsyncStream<Element>.Continuation!
        self.stream = AsyncStream<Element> { continuation in
            cont = continuation
        }
        self.continuation = cont
    }
    
    func send(_ element: Element) async {
        continuation.yield(element)
    }
    
    func finish() {
        continuation.finish()
    }
    
    func makeAsyncIterator() -> AsyncStream<Element>.AsyncIterator {
        stream.makeAsyncIterator()
    }
}

// ใช้งาน AsyncChannel Pattern
func demonstrateAsyncChannel() async {
    let channel = SimpleAsyncChannel<String>()
    
    // Producer Task
    Task {
        let messages = ["สวัสดี", "Hello", "こんにちは", "Bonjour"]
        for message in messages {
            await channel.send(message)
            try? await Task.sleep(nanoseconds: 500_000_000)
        }
        channel.finish()
    }
    
    // Consumer Task
    for await message in channel {
        print("ได้รับ: \(message)")
    }
    
    print("ช่อง Communication ปิดแล้ว")
}

// Real AsyncChannel จาก swift-async-algorithms
// let channel = AsyncChannel<String>()
//
// Task {
//     await channel.send("message 1")
//     await channel.send("message 2")
//     channel.finish()
// }
//
// for await value in channel {
//     print(value)
// }
```

---

## 14. async/await กับ Combine

การรวม Combine กับ async/await ช่วยให้เราใช้ประโยชน์จากทั้งสองระบบ

```swift
import Combine
import Foundation

// แปลง Publisher เป็น AsyncSequence
extension Publisher where Failure == Never {
    var values: AsyncPublisher<Self> {
        AsyncPublisher(self)
    }
}

// ใช้ Combine Publisher ใน async/await Context
class DataService {
    private let subject = PassthroughSubject<String, Never>()
    
    var updates: AnyPublisher<String, Never> {
        subject.eraseToAnyPublisher()
    }
    
    func sendUpdate(_ value: String) {
        subject.send(value)
    }
}

func useCombineWithAsync() async {
    let service = DataService()
    var cancellables = Set<AnyCancellable>()
    
    // วิธีที่ 1: ใช้ values property
    Task {
        // AsyncPublisher ช่วยให้ใช้ for await ได้
        for await value in service.updates.values {
            print("ได้รับ (async): \(value)")
        }
    }
    
    // วิธีที่ 2: ใช้ sink แบบปกติ
    service.updates
        .sink { value in
            print("ได้รับ (Combine): \(value)")
        }
        .store(in: &cancellables)
    
    // ส่งข้อมูล
    service.sendUpdate("อัปเดตแรก")
    service.sendUpdate("อัปเดตที่สอง")
}
```

---

## 15. การแปลงระหว่าง Combine และ async/await

```swift
import Combine
import Foundation

// แปลง async function เป็น Publisher
func asyncFunctionToPublisher<T>(
    priority: TaskPriority? = nil,
    operation: @escaping () async throws -> T
) -> AnyPublisher<T, Error> {
    let subject = PassthroughSubject<T, Error>()
    
    Task(priority: priority) {
        do {
            let result = try await operation()
            subject.send(result)
            subject.send(completion: .finished)
        } catch {
            subject.send(completion: .failure(error))
        }
    }
    
    return subject.eraseToAnyPublisher()
}

// ตัวอย่างการใช้งาน
class ImageLoader {
    // async function
    func loadImage(named name: String) async throws -> String {
        try await Task.sleep(nanoseconds: 500_000_000)
        return "Image:\(name)"
    }
    
    // แปลงเป็น Publisher
    func loadImagePublisher(named name: String) -> AnyPublisher<String, Error> {
        asyncFunctionToPublisher {
            try await self.loadImage(named: name)
        }
    }
}

// แปลง Combine Future เป็น async
extension Future where Failure == Error {
    var asyncValue: Output {
        get async throws {
            try await withCheckedThrowingContinuation { continuation in
                var cancellable: AnyCancellable?
                cancellable = self.sink(
                    receiveCompletion: { completion in
                        switch completion {
                        case .failure(let error):
                            continuation.resume(throwing: error)
                        case .finished:
                            break
                        }
                        cancellable?.cancel()
                    },
                    receiveValue: { value in
                        continuation.resume(returning: value)
                    }
                )
            }
        }
    }
}

// ใช้ withCheckedContinuation เพื่อ Bridge Callback-based API
func withCallback(completion: @escaping (Result<String, Error>) -> Void) {
    DispatchQueue.global().asyncAfter(deadline: .now() + 0.5) {
        completion(.success("ผลลัพธ์จาก Callback"))
    }
}

func callbackToAsync() async throws -> String {
    try await withCheckedThrowingContinuation { continuation in
        withCallback { result in
            switch result {
            case .success(let value):
                continuation.resume(returning: value)
            case .failure(let error):
                continuation.resume(throwing: error)
            }
        }
    }
}
```

---

## 16. Clocks: ContinuousClock และ SuspendingClock

Swift 5.7 เพิ่ม `Clock` Protocol พร้อม Implementation สองแบบ

```swift
import Foundation

// ContinuousClock - นับเวลาต่อเนื่องแม้ Device จะ Sleep
// SuspendingClock - หยุดนับเมื่อ Device Sleep (เป็น Default ของ Task.sleep)

// ใช้ ContinuousClock
func useContinuousClock() async throws {
    let clock = ContinuousClock()
    
    // วัดเวลา
    let elapsed = try await clock.measure {
        try await Task.sleep(for: .seconds(1))
    }
    print("ใช้เวลา: \(elapsed)")
    
    // Sleep จนถึง Instant ที่กำหนด
    let deadline = clock.now + .seconds(2)
    try await clock.sleep(until: deadline, tolerance: .milliseconds(100))
    print("ตื่นแล้ว!")
}

// ใช้ SuspendingClock
func useSuspendingClock() async throws {
    let clock = SuspendingClock()
    
    let start = clock.now
    try await Task.sleep(for: .seconds(1))
    let elapsed = clock.now - start
    
    print("ใช้เวลา (SuspendingClock): \(elapsed)")
}

// สร้าง Generic Timer ด้วย Clock Protocol
struct Timer<C: Clock> where C.Duration == Duration {
    let clock: C
    let interval: Duration
    
    init(clock: C, every interval: Duration) {
        self.clock = clock
        self.interval = interval
    }
    
    func run(action: @escaping () async -> Void) async {
        var next = clock.now.advanced(by: interval)
        
        while !Task.isCancelled {
            do {
                try await clock.sleep(until: next)
                await action()
                next = next.advanced(by: interval)
            } catch {
                break
            }
        }
    }
}

// ใช้งาน
Task {
    let timer = Timer(clock: ContinuousClock(), every: .seconds(1))
    await timer.run {
        print("Timer fired at: \(Date())")
    }
}
```

---

## 17. sleep(until:)

การ Sleep ที่ Precise กว่า `Task.sleep(nanoseconds:)`

```swift
import Foundation

// sleep(until:tolerance:clock:) - สำหรับความแม่นยำสูง
func preciseSleep() async throws {
    let clock = ContinuousClock()
    let target = clock.now + .milliseconds(500)
    
    // Sleep จนถึง Target Time
    // tolerance: อนุญาตให้ตื่นได้เร็วกว่า target ได้มากแค่ไหน
    try await clock.sleep(until: target, tolerance: .milliseconds(10))
    
    print("ตื่นขึ้น ณ: \(Date())")
}

// การใช้ sleep สำหรับ Polling
func pollUntilReady<T>(
    timeout: Duration,
    interval: Duration = .milliseconds(100),
    check: @escaping () async -> T?
) async throws -> T {
    let clock = ContinuousClock()
    let deadline = clock.now + timeout
    
    while clock.now < deadline {
        if let result = await check() {
            return result
        }
        
        // Sleep จนถึง Interval ถัดไปหรือ Deadline (แล้วแต่อะไรมาก่อน)
        let nextCheck = min(clock.now + interval, deadline)
        try await clock.sleep(until: nextCheck)
    }
    
    throw TimeoutError()
}

struct TimeoutError: Error {
    var description: String { "หมดเวลาแล้ว" }
}

// ใช้งาน
Task {
    var isReady = false
    
    // จำลอง Server ที่พร้อมหลัง 300ms
    Task {
        try await Task.sleep(for: .milliseconds(300))
        isReady = true
    }
    
    do {
        _ = try await pollUntilReady(timeout: .seconds(2)) {
            isReady ? "พร้อมแล้ว!" : nil
        }
        print("Server พร้อมแล้ว!")
    } catch {
        print("หมดเวลา: \(error)")
    }
}
```

---

## 18. withDeadline (Timeout Pattern)

Swift ยังไม่มี built-in `withDeadline` แต่เราสามารถสร้างได้

```swift
import Foundation

// Custom withTimeout function
func withTimeout<T>(
    seconds: Double,
    operation: @escaping () async throws -> T
) async throws -> T {
    try await withThrowingTaskGroup(of: T.self) { group in
        // เพิ่ม Task หลัก
        group.addTask {
            try await operation()
        }
        
        // เพิ่ม Timeout Task
        group.addTask {
            try await Task.sleep(for: .seconds(seconds))
            throw TimeoutError()
        }
        
        // รับผลลัพธ์แรกที่ได้
        guard let result = try await group.next() else {
            throw CancellationError()
        }
        
        // Cancel Task ที่เหลือ
        group.cancelAll()
        return result
    }
}

struct TimeoutError2: Error, LocalizedError {
    let timeout: Double
    var errorDescription: String? {
        "การดำเนินการหมดเวลาหลังจาก \(timeout) วินาที"
    }
}

// วิธีที่ 2: ใช้ race condition pattern
func withDeadline<T>(
    _ deadline: ContinuousClock.Instant,
    operation: @escaping () async throws -> T
) async throws -> T {
    let clock = ContinuousClock()
    let remaining = deadline - clock.now
    
    guard remaining > .zero else {
        throw TimeoutError()
    }
    
    return try await withTimeout(seconds: Double(remaining.components.seconds), operation: operation)
}

// ใช้งาน
Task {
    do {
        // ต้องเสร็จภายใน 2 วินาที
        let result = try await withTimeout(seconds: 2.0) {
            // จำลองงานที่ใช้เวลา
            try await Task.sleep(for: .seconds(1))
            return "ผลลัพธ์"
        }
        print("สำเร็จ: \(result)")
    } catch {
        print("หมดเวลา: \(error)")
    }
    
    // ตัวอย่างที่ Timeout
    do {
        let result = try await withTimeout(seconds: 0.5) {
            try await Task.sleep(for: .seconds(2))
            return "ผลลัพธ์ช้า"
        }
        print("สำเร็จ: \(result)")
    } catch {
        print("หมดเวลาตามที่คาดไว้: \(error)")
    }
}
```

---

## 19. การป้องกัน Race Conditions

Race Condition เกิดขึ้นเมื่อ Thread หลายตัวเข้าถึงข้อมูลเดียวกันพร้อมกัน

```swift
import Foundation

// ❌ Race Condition - อย่าทำแบบนี้
class UnsafeCounter {
    var value: Int = 0
    
    func increment() {
        // ไม่ Thread-Safe!
        value += 1
    }
}

// ✅ แก้ด้วย Actor
actor SafeCounter {
    private var value: Int = 0
    
    func increment() {
        value += 1
    }
    
    func decrement() {
        value -= 1
    }
    
    var current: Int {
        value
    }
}

// ✅ ตัวอย่างที่ซับซ้อน: Thread-Safe Dictionary
actor ConcurrentDictionary<Key: Hashable, Value> {
    private var storage: [Key: Value] = [:]
    
    subscript(key: Key) -> Value? {
        get { storage[key] }
        set { storage[key] = newValue }
    }
    
    func updateValue(_ value: Value, forKey key: Key) -> Value? {
        defer { storage[key] = value }
        return storage[key]
    }
    
    func removeValue(forKey key: Key) -> Value? {
        storage.removeValue(forKey: key)
    }
    
    var count: Int { storage.count }
    
    var keys: [Key] { Array(storage.keys) }
    
    func contains(key: Key) -> Bool {
        storage[key] != nil
    }
}

// ✅ ใช้ Actor สำหรับ Shared State
actor UserSession {
    private(set) var isLoggedIn: Bool = false
    private(set) var currentUser: UserData? = nil
    private var loginAttempts: Int = 0
    
    func login(username: String, password: String) async throws {
        loginAttempts += 1
        
        guard loginAttempts <= 3 else {
            throw AuthError.tooManyAttempts
        }
        
        // จำลอง Authentication
        try await Task.sleep(nanoseconds: 500_000_000)
        
        if username == "admin" && password == "password" {
            isLoggedIn = true
            currentUser = UserData(id: "1", name: username)
            loginAttempts = 0
        } else {
            throw AuthError.invalidCredentials
        }
    }
    
    func logout() {
        isLoggedIn = false
        currentUser = nil
    }
}

enum AuthError: Error {
    case invalidCredentials
    case tooManyAttempts
}

struct UserData {
    let id: String
    let name: String
}
```

---

## 20. การทดสอบ Concurrency

การทดสอบโค้ด Concurrent ต้องการเทคนิคพิเศษ

```swift
import XCTest
@testable import MyApp  // สมมติว่านี้คือ Module ของเรา

// การทดสอบ Actor
final class SafeCounterTests: XCTestCase {
    
    func testConcurrentIncrements() async {
        let counter = SafeCounter()
        
        // รัน Increment พร้อมกัน 1000 ครั้ง
        await withTaskGroup(of: Void.self) { group in
            for _ in 0..<1000 {
                group.addTask {
                    await counter.increment()
                }
            }
        }
        
        let finalValue = await counter.current
        XCTAssertEqual(finalValue, 1000, "ควรได้ 1000 แต่ได้ \(finalValue)")
    }
    
    func testConcurrentReadAndWrite() async {
        let dict = ConcurrentDictionary<String, Int>()
        
        // เขียนและอ่านพร้อมกัน
        await withTaskGroup(of: Void.self) { group in
            // Writers
            for i in 0..<100 {
                group.addTask {
                    await dict.updateValue(i, forKey: "key_\(i)")
                }
            }
            
            // Readers
            for i in 0..<100 {
                group.addTask {
                    let _ = await dict["key_\(i)"]
                }
            }
        }
        
        let count = await dict.count
        XCTAssertLessThanOrEqual(count, 100)
    }
}

// Mock สำหรับ Async Testing
class MockNetworkService {
    var shouldFail = false
    var delay: Duration = .milliseconds(0)
    
    func fetchData() async throws -> String {
        if delay > .zero {
            try await Task.sleep(for: delay)
        }
        
        if shouldFail {
            throw NSError(domain: "MockError", code: 1)
        }
        
        return "Mock Data"
    }
}

final class NetworkTests: XCTestCase {
    
    func testSuccessfulFetch() async throws {
        let service = MockNetworkService()
        let data = try await service.fetchData()
        XCTAssertEqual(data, "Mock Data")
    }
    
    func testFailedFetch() async {
        let service = MockNetworkService()
        service.shouldFail = true
        
        do {
            _ = try await service.fetchData()
            XCTFail("ควร Throw Error")
        } catch {
            XCTAssertNotNil(error)
        }
    }
    
    func testTimeout() async throws {
        let service = MockNetworkService()
        service.delay = .seconds(5)
        
        do {
            _ = try await withTimeout(seconds: 1.0) {
                try await service.fetchData()
            }
            XCTFail("ควร Timeout")
        } catch {
            // คาดหวัง Timeout Error
            XCTAssertTrue(error is TimeoutError)
        }
    }
}
```

---

## 21. Performance Profiling ของ Concurrent Code

```swift
import Foundation
import os.signpost

// ใช้ os_signpost สำหรับ Profiling
let subsystem = "com.example.app"
let category = "Concurrency"
let log = OSLog(subsystem: subsystem, category: category)

func profiledWork(name: String, work: () async -> Void) async {
    let signpostID = OSSignpostID(log: log)
    
    os_signpost(.begin, log: log, name: "Task", signpostID: signpostID, "%{public}s", name)
    
    await work()
    
    os_signpost(.end, log: log, name: "Task", signpostID: signpostID)
}

// ตัวอย่างการ Profile
Task {
    await profiledWork(name: "DataProcessing") {
        // งานที่ต้องการ Profile
        await processLargeDataset([1, 2, 3, 4, 5])
    }
}

// Instrument สำหรับวัด Throughput
actor ThroughputMonitor {
    private var operationCount: Int = 0
    private var startTime: ContinuousClock.Instant = ContinuousClock().now
    
    func recordOperation() {
        operationCount += 1
    }
    
    func getThroughput() -> Double {
        let elapsed = ContinuousClock().now - startTime
        let seconds = Double(elapsed.components.seconds) + Double(elapsed.components.attoseconds) * 1e-18
        return seconds > 0 ? Double(operationCount) / seconds : 0
    }
    
    func reset() {
        operationCount = 0
        startTime = ContinuousClock().now
    }
}

// วัด Performance ของ Concurrent Operations
func benchmarkConcurrentOperations() async {
    let monitor = ThroughputMonitor()
    let iterations = 10_000
    
    // Sequential
    let seqStart = ContinuousClock().now
    for i in 0..<iterations {
        _ = processNumber(i)
    }
    let seqTime = ContinuousClock().now - seqStart
    
    // Concurrent
    let concStart = ContinuousClock().now
    await withTaskGroup(of: Int.self) { group in
        for i in 0..<iterations {
            group.addTask {
                return processNumber(i)
            }
        }
        var results: [Int] = []
        for await result in group {
            results.append(result)
            await monitor.recordOperation()
        }
    }
    let concTime = ContinuousClock().now - concStart
    
    print("Sequential time: \(seqTime)")
    print("Concurrent time: \(concTime)")
    let throughput = await monitor.getThroughput()
    print("Throughput: \(throughput) ops/sec")
}

func processNumber(_ n: Int) -> Int {
    // จำลองการคำนวณ
    var result = n
    for _ in 0..<100 {
        result = (result &* 1103515245 &+ 12345) & 0x7fffffff
    }
    return result
}
```

---

## 22. แบบฝึกหัดพร้อมเฉลย

### แบบฝึกหัดที่ 1: Async Data Pipeline

**โจทย์**: สร้าง Data Pipeline ที่ดาวน์โหลดข้อมูล, แปลงข้อมูล และบันทึกผลลัพธ์ โดยแต่ละขั้นตอนทำงานแบบ Concurrent

```swift
// เฉลย
struct DataPipeline {
    // ขั้นที่ 1: ดาวน์โหลดข้อมูล
    static func fetchData(from urls: [URL]) async throws -> [(URL, Data)] {
        try await withThrowingTaskGroup(of: (URL, Data).self) { group in
            for url in urls {
                group.addTask {
                    let (data, _) = try await URLSession.shared.data(from: url)
                    return (url, data)
                }
            }
            
            var results: [(URL, Data)] = []
            for try await result in group {
                results.append(result)
            }
            return results
        }
    }
    
    // ขั้นที่ 2: แปลงข้อมูล
    static func transform(_ items: [(URL, Data)]) async -> [(URL, String)] {
        await withTaskGroup(of: (URL, String).self) { group in
            for (url, data) in items {
                group.addTask {
                    let transformed = String(data: data, encoding: .utf8) ?? ""
                    return (url, transformed.uppercased())  // แปลงเป็น Uppercase
                }
            }
            
            var results: [(URL, String)] = []
            for await result in group {
                results.append(result)
            }
            return results
        }
    }
    
    // ขั้นที่ 3: บันทึกผลลัพธ์
    static func save(_ items: [(URL, String)], to directory: URL) async throws {
        try await withThrowingTaskGroup(of: Void.self) { group in
            for (url, content) in items {
                group.addTask {
                    let filename = url.lastPathComponent
                    let destination = directory.appendingPathComponent(filename)
                    try content.write(to: destination, atomically: true, encoding: .utf8)
                    print("บันทึก: \(filename)")
                }
            }
            
            for try await _ in group {}
        }
    }
    
    // Pipeline รวม
    static func run(urls: [URL], outputDirectory: URL) async throws {
        print("เริ่ม Pipeline...")
        
        let rawData = try await fetchData(from: urls)
        print("ดาวน์โหลดเสร็จ: \(rawData.count) ไฟล์")
        
        let transformed = await transform(rawData)
        print("แปลงข้อมูลเสร็จ: \(transformed.count) ไฟล์")
        
        try await save(transformed, to: outputDirectory)
        print("Pipeline เสร็จสิ้น!")
    }
}
```

### แบบฝึกหัดที่ 2: Rate Limiter

**โจทย์**: สร้าง Rate Limiter ที่จำกัดจำนวน Request ที่สามารถทำได้ต่อวินาที

```swift
// เฉลย
actor RateLimiter {
    private let maxRequests: Int
    private let interval: Duration
    private var requestTimes: [ContinuousClock.Instant] = []
    private let clock = ContinuousClock()
    
    init(maxRequests: Int, per interval: Duration) {
        self.maxRequests = maxRequests
        self.interval = interval
    }
    
    func acquire() async throws {
        while true {
            let now = clock.now
            let windowStart = now - interval
            
            // ลบ Request ที่หมดอายุแล้ว
            requestTimes = requestTimes.filter { $0 > windowStart }
            
            if requestTimes.count < maxRequests {
                // มีที่ว่าง
                requestTimes.append(now)
                return
            }
            
            // ต้องรอ
            guard let oldest = requestTimes.first else { return }
            let waitUntil = oldest + interval
            
            try await clock.sleep(until: waitUntil)
        }
    }
}

// ใช้งาน
class APIClient {
    private let rateLimiter = RateLimiter(maxRequests: 10, per: .seconds(1))
    
    func makeRequest(to url: URL) async throws -> Data {
        // รอจนกว่าจะผ่าน Rate Limit
        try await rateLimiter.acquire()
        
        // ทำ Request จริง
        let (data, _) = try await URLSession.shared.data(from: url)
        return data
    }
}

// ทดสอบ Rate Limiter
Task {
    let client = APIClient()
    let url = URL(string: "https://api.example.com/data")!
    
    // พยายามทำ 20 Requests พร้อมกัน
    await withTaskGroup(of: Void.self) { group in
        for i in 0..<20 {
            group.addTask {
                do {
                    _ = try await client.makeRequest(to: url)
                    print("Request \(i) สำเร็จ")
                } catch {
                    print("Request \(i) ล้มเหลว: \(error)")
                }
            }
        }
    }
}
```

---

## 23. การสร้าง Actor-Based Data Pipeline

ตัวอย่างขนาดใหญ่ที่แสดงการใช้ Actor สร้าง Data Pipeline

```swift
import Foundation

// ============================================================
// Actor-Based Data Pipeline
// ============================================================

// Protocol สำหรับ Pipeline Stages
protocol PipelineStage<Input, Output> {
    associatedtype Input: Sendable
    associatedtype Output: Sendable
    
    func process(_ input: Input) async throws -> Output
}

// Buffer Stage - เก็บ Items ไว้ชั่วคราว
actor BufferStage<T: Sendable> {
    private var buffer: [T] = []
    private let capacity: Int
    private var consumers: [CheckedContinuation<T, Never>] = []
    private var producers: [CheckedContinuation<Void, Never>] = []
    
    init(capacity: Int) {
        self.capacity = capacity
    }
    
    func push(_ item: T) async {
        if buffer.count < capacity {
            if let consumer = consumers.first {
                consumers.removeFirst()
                consumer.resume(returning: item)
            } else {
                buffer.append(item)
            }
        } else {
            // Buffer เต็ม - รอ
            await withCheckedContinuation { continuation in
                producers.append(continuation)
            }
            buffer.append(item)
        }
    }
    
    func pop() async -> T {
        if let item = buffer.first {
            buffer.removeFirst()
            
            // ปลดปล่อย Producer ที่รออยู่
            if let producer = producers.first {
                producers.removeFirst()
                producer.resume()
            }
            
            return item
        } else {
            // Buffer ว่าง - รอ
            return await withCheckedContinuation { continuation in
                consumers.append(continuation)
            }
        }
    }
    
    var count: Int { buffer.count }
}

// Transform Stage - แปลงข้อมูล
actor TransformStage<Input: Sendable, Output: Sendable> {
    private let transform: (Input) async throws -> Output
    private let workers: Int
    
    init(workers: Int, transform: @escaping (Input) async throws -> Output) {
        self.workers = workers
        self.transform = transform
    }
    
    func processAll(
        input: BufferStage<Input>,
        output: BufferStage<Output>,
        count: Int
    ) async throws {
        try await withThrowingTaskGroup(of: Void.self) { group in
            for _ in 0..<workers {
                group.addTask {
                    for _ in 0..<(count / self.workers) {
                        let item = await input.pop()
                        let result = try await self.transform(item)
                        await output.push(result)
                    }
                }
            }
            
            try await group.waitForAll()
        }
    }
}

// Pipeline Coordinator
actor DataPipelineCoordinator<T: Sendable, U: Sendable> {
    private let inputBuffer: BufferStage<T>
    private let outputBuffer: BufferStage<U>
    private let transformer: TransformStage<T, U>
    
    private(set) var processedCount: Int = 0
    private(set) var errorCount: Int = 0
    
    init(
        bufferSize: Int = 100,
        workers: Int = 4,
        transform: @escaping (T) async throws -> U
    ) {
        self.inputBuffer = BufferStage(capacity: bufferSize)
        self.outputBuffer = BufferStage(capacity: bufferSize)
        self.transformer = TransformStage(workers: workers, transform: transform)
    }
    
    func feed(_ items: [T]) async {
        await withTaskGroup(of: Void.self) { group in
            for item in items {
                group.addTask {
                    await self.inputBuffer.push(item)
                }
            }
        }
    }
    
    func process(count: Int) async throws -> [U] {
        try await transformer.processAll(
            input: inputBuffer,
            output: outputBuffer,
            count: count
        )
        
        var results: [U] = []
        for _ in 0..<count {
            let result = await outputBuffer.pop()
            results.append(result)
            processedCount += 1
        }
        
        return results
    }
}

// ตัวอย่างการใช้งาน Pipeline
Task {
    // สร้าง Pipeline ที่แปลง Int เป็น String
    let pipeline = DataPipelineCoordinator<Int, String>(
        bufferSize: 50,
        workers: 4
    ) { number in
        // จำลองการแปลงข้อมูลที่ใช้เวลา
        try await Task.sleep(nanoseconds: UInt64.random(in: 1_000_000...10_000_000))
        return "Processed: \(number * number)"
    }
    
    // ป้อนข้อมูล
    let inputData = Array(1...100)
    await pipeline.feed(inputData)
    
    // ประมวลผล
    do {
        let results = try await pipeline.process(count: 100)
        print("ประมวลผลเสร็จ: \(results.count) รายการ")
        print("ตัวอย่างผลลัพธ์: \(results.prefix(5).joined(separator: ", "))")
        
        let processed = await pipeline.processedCount
        print("จำนวนที่ประมวลผล: \(processed)")
    } catch {
        print("เกิดข้อผิดพลาด: \(error)")
    }
}
```

---

## 24. การใช้ AsyncStream

`AsyncStream` เป็นวิธีสร้าง Async Sequence จาก Callback-based code

```swift
import Foundation

// สร้าง AsyncStream จาก Notification
func notificationStream(name: Notification.Name) -> AsyncStream<Notification> {
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

// สร้าง AsyncStream จาก Timer
func timerStream(interval: TimeInterval) -> AsyncStream<Date> {
    AsyncStream { continuation in
        let timer = Timer.scheduledTimer(withTimeInterval: interval, repeats: true) { _ in
            continuation.yield(Date())
        }
        
        continuation.onTermination = { _ in
            timer.invalidate()
        }
    }
}

// AsyncThrowingStream สำหรับงานที่อาจ Throw Error
func dataStream(from urls: [URL]) -> AsyncThrowingStream<Data, Error> {
    AsyncThrowingStream { continuation in
        Task {
            for url in urls {
                do {
                    let (data, _) = try await URLSession.shared.data(from: url)
                    continuation.yield(data)
                } catch {
                    continuation.finish(throwing: error)
                    return
                }
            }
            continuation.finish()
        }
    }
}

// ใช้งาน AsyncStream
Task {
    // อ่านจาก Timer
    let timer = timerStream(interval: 1.0)
    var count = 0
    
    for await date in timer {
        print("เวลาปัจจุบัน: \(date)")
        count += 1
        if count >= 3 { break }
    }
}

// Debounce ด้วย AsyncStream
func debounce<T: Sendable>(
    stream: some AsyncSequence<T, Never>,
    duration: Duration
) -> AsyncStream<T> {
    AsyncStream { continuation in
        Task {
            var currentTask: Task<Void, Never>?
            
            for await value in stream {
                currentTask?.cancel()
                currentTask = Task {
                    do {
                        try await Task.sleep(for: duration)
                        continuation.yield(value)
                    } catch {}
                }
            }
            continuation.finish()
        }
    }
}
```

---

## 25. สรุป

ใน Part นี้เราได้เรียนรู้หัวข้อขั้นสูงของ Concurrency ใน Swift ดังนี้:

### สิ่งที่ได้เรียนรู้

| หัวข้อ | สาระสำคัญ |
|--------|----------|
| Advanced Task Patterns | TaskGroup, Dynamic Task Creation, Structured vs Unstructured |
| Task Priority | กำหนดลำดับความสำคัญ, Priority Escalation |
| TaskLocal Values | ส่งข้อมูลผ่าน Task Tree, @TaskLocal Property Wrapper |
| Actor Reentrancy | ระวัง State ไม่ Consistent, หัก State ก่อน await |
| Actor Hop | ลด Context Switches, Batch Operations |
| Distributed Actors | Communication ข้าม Process/Machine |
| Custom Executors | กำหนด Thread ที่ Actor รันบน |
| SerialExecutor | รับประกัน Sequential Execution |
| MainActor Optimization | ลด Main Thread Hops |
| Cooperative Cancellation | ตรวจสอบ Task.isCancelled, withTaskCancellationHandler |
| AsyncChannel | Communication ระหว่าง Tasks |
| Combine + async/await | Bridge ระหว่างสองระบบ |
| Clocks | ContinuousClock, SuspendingClock, Precise Sleep |
| Timeout Pattern | withTimeout, withDeadline |
| Race Condition Prevention | Actor, @Sendable, Isolation |
| Testing Concurrency | XCTest async, Mock Services |
| Performance Profiling | os_signpost, Throughput Measurement |
| Actor-Based Pipeline | สร้าง Data Pipeline ด้วย Actors |

### Best Practices

1. **ใช้ Structured Concurrency** เมื่อเป็นไปได้ (async let, TaskGroup)
2. **ระวัง Actor Reentrancy** - เปลี่ยน State ก่อน await
3. **ลด Actor Hops** โดย Batch Operations
4. **ใช้ TaskLocal** สำหรับ Context (Request ID, Logging)
5. **Implement Cooperative Cancellation** ใน Long-running Tasks
6. **Test Concurrent Code** ด้วย High Concurrency Scenarios
7. **Profile ก่อน Optimize** - ใช้ Instruments และ os_signpost

### ขั้นตอนต่อไป

ใน Part 54 เราจะเรียนรู้เกี่ยวกับ Swift 6 Features ซึ่งรวมถึง Strict Concurrency Checking, Typed Throws, Noncopyable Types และ Swift Macros

---

*จบ Part 53: Advanced Concurrency*
