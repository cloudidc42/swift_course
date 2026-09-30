# Part 29: Swift Concurrency

## บทนำ

Swift Concurrency เป็นระบบการเขียนโปรแกรมแบบ concurrent ที่ Apple แนะนำใน Swift 5.5 ซึ่งทำให้การเขียนโค้ดที่ทำงานพร้อมกันหลายๆ อย่างในเวลาเดียวกันนั้นง่ายขึ้น ปลอดภัยขึ้น และอ่านเข้าใจได้ง่ายขึ้นมาก

---

## 1. ทำไมต้องใช้ Concurrency?

### ปัญหาของการทำงานแบบ Synchronous

ในการพัฒนาแอปพลิเคชัน เรามักจะต้องจัดการกับงานที่ใช้เวลานาน เช่น:
- การดาวน์โหลดข้อมูลจาก Network
- การอ่าน/เขียนไฟล์
- การประมวลผลภาพ
- การ Query ฐานข้อมูล

หากทำงานเหล่านี้แบบ Synchronous (รอให้เสร็จก่อนทำขั้นต่อไป) UI จะค้างทำให้ผู้ใช้รู้สึกว่าแอปไม่ตอบสนอง

```swift
// ตัวอย่างที่ไม่ดี - Synchronous บน Main Thread
func loadData() {
    // โค้ดนี้จะทำให้ UI ค้างระหว่างรอ!
    let data = URLSession.shared.data(from: URL(string: "https://api.example.com")!)
    // ผู้ใช้ไม่สามารถโต้ตอบกับ UI ได้จนกว่าจะโหลดเสร็จ
    processData(data)
}
```

### วิธีแก้ปัญหาในอดีต: Completion Handlers

ก่อน Swift Concurrency นักพัฒนาใช้ completion handlers หรือ callbacks:

```swift
// วิธีเดิม - Completion Handler
func loadUser(id: Int, completion: @escaping (Result<User, Error>) -> Void) {
    URLSession.shared.dataTask(with: userURL(id: id)) { data, response, error in
        if let error = error {
            completion(.failure(error))
            return
        }
        guard let data = data else {
            completion(.failure(NetworkError.noData))
            return
        }
        do {
            let user = try JSONDecoder().decode(User.self, from: data)
            completion(.success(user))
        } catch {
            completion(.failure(error))
        }
    }.resume()
}

// การใช้งาน - Pyramid of Doom
func loadUserAndPosts(userId: Int) {
    loadUser(id: userId) { result in
        switch result {
        case .success(let user):
            loadPosts(for: user.id) { postsResult in
                switch postsResult {
                case .success(let posts):
                    loadComments(for: posts.first!) { commentsResult in
                        // ยิ่งลึกยิ่งอ่านยาก!
                        switch commentsResult {
                        case .success(let comments):
                            DispatchQueue.main.async {
                                self.updateUI(user: user, posts: posts, comments: comments)
                            }
                        case .failure(let error):
                            print("Error: \(error)")
                        }
                    }
                case .failure(let error):
                    print("Error: \(error)")
                }
            }
        case .failure(let error):
            print("Error: \(error)")
        }
    }
}
```

ปัญหาของ Completion Handlers:
1. **Pyramid of Doom** - โค้ดซ้อนกันลึกมากอ่านยาก
2. **Error Handling** - ต้องจัดการ error แยกในแต่ละ level
3. **Thread Safety** - ต้องระวังเรื่อง race conditions เอง
4. **Callback Hell** - ยากต่อการ debug และ maintain

### Swift Concurrency แก้ปัญหาอย่างไร?

```swift
// วิธีใหม่ - async/await
func loadUserAndPosts(userId: Int) async throws {
    let user = try await loadUser(id: userId)
    let posts = try await loadPosts(for: user.id)
    let comments = try await loadComments(for: posts.first!)
    
    await MainActor.run {
        updateUI(user: user, posts: posts, comments: comments)
    }
}
```

โค้ดอ่านง่ายขึ้นมาก เหมือนเขียน Synchronous code แต่ทำงานแบบ Asynchronous!

---

## 2. async/await พื้นฐาน

### Async Functions

ฟังก์ชันที่มี `async` ในลายเซ็นสามารถทำงานแบบ asynchronous ได้:

```swift
// การประกาศ async function
func fetchUserData() async -> User {
    // โค้ดที่ทำงานแบบ async
    return User(name: "John", age: 30)
}

// async function ที่อาจ throw error
func fetchDataFromServer() async throws -> Data {
    let url = URL(string: "https://api.example.com/data")!
    let (data, response) = try await URLSession.shared.data(from: url)
    
    guard let httpResponse = response as? HTTPURLResponse,
          httpResponse.statusCode == 200 else {
        throw NetworkError.badResponse
    }
    
    return data
}
```

### await Keyword

`await` ใช้เพื่อ "รอ" ผลลัพธ์จาก async function โดยไม่บล็อก thread:

```swift
// ต้องเรียกใช้ async function ด้วย await
func displayUser() async {
    let user = await fetchUserData()
    print("User: \(user.name)")
}

// await กับ throws
func loadAndDisplay() async {
    do {
        let data = try await fetchDataFromServer()
        let user = try JSONDecoder().decode(User.self, from: data)
        print("Loaded: \(user.name)")
    } catch {
        print("Error: \(error)")
    }
}
```

### Suspension Points

เมื่อ Swift เจอ `await` มันจะ:
1. **Suspend** (หยุดชั่วคราว) การทำงานของ function นั้น
2. ปล่อยให้ thread ทำงานอื่นได้
3. **Resume** (กลับมาทำงาน) เมื่อ async operation เสร็จ

```swift
func processMultipleTasks() async {
    print("เริ่มต้น")
    
    // Suspension point 1 - thread ว่างระหว่างรอ
    let result1 = await fetchData(from: "url1")
    print("ได้ result1 แล้ว")
    
    // Suspension point 2
    let result2 = await fetchData(from: "url2")
    print("ได้ result2 แล้ว")
    
    print("เสร็จสิ้น")
}
```

### เรียกใช้ async จาก Synchronous Context

ใช้ `Task` เพื่อเรียก async code จาก synchronous context:

```swift
// ใน ViewController
override func viewDidLoad() {
    super.viewDidLoad()
    
    // ไม่สามารถใช้ await ตรงๆ ใน viewDidLoad ได้เพราะไม่ใช่ async
    Task {
        do {
            let data = try await fetchDataFromServer()
            await MainActor.run {
                self.updateUI(with: data)
            }
        } catch {
            print("Error: \(error)")
        }
    }
}
```

---

## 3. Async Functions เชิงลึก

### Return Types

```swift
// ส่งค่ากลับแบบต่างๆ
func fetchNumber() async -> Int {
    return 42
}

func fetchOptionalData() async -> Data? {
    // อาจคืนค่า nil ได้
    return nil
}

func fetchMultipleValues() async -> (name: String, age: Int) {
    return ("Alice", 25)
}

// ใช้กับ Generic types
func fetchArray<T: Decodable>(_ type: T.Type, from url: URL) async throws -> [T] {
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode([T].self, from: data)
}
```

### Async Properties

```swift
struct DataProvider {
    // Async computed property
    var currentUser: User {
        get async throws {
            let data = try await fetchCurrentUserData()
            return try JSONDecoder().decode(User.self, from: data)
        }
    }
    
    var userCount: Int {
        get async {
            return await database.countUsers()
        }
    }
}

// การใช้งาน
let provider = DataProvider()
let user = try await provider.currentUser
let count = await provider.userCount
```

### Async Subscripts

```swift
struct UserCache {
    subscript(id: Int) -> User? {
        get async {
            return await fetchUserFromCache(id: id)
        }
    }
}

// การใช้งาน
let cache = UserCache()
if let user = await cache[1] {
    print("Found: \(user.name)")
}
```

### Async Initializers

```swift
struct Configuration {
    let apiKey: String
    let baseURL: URL
    
    init() async throws {
        // โหลด config จาก server ระหว่าง init
        let data = try await fetchConfigData()
        let config = try JSONDecoder().decode(ConfigResponse.self, from: data)
        self.apiKey = config.apiKey
        self.baseURL = config.baseURL
    }
}

// การใช้งาน
let config = try await Configuration()
```

---

## 4. Task

`Task` คือหน่วยงาน (unit of work) ที่ทำงานแบบ asynchronous ใน Swift Concurrency

### สร้าง Task พื้นฐาน

```swift
// Task พื้นฐาน
let task = Task {
    let result = await someAsyncFunction()
    print("Result: \(result)")
}

// Task ที่ throw error ได้
let throwingTask = Task {
    let data = try await fetchData()
    return data
}

// รอผลลัพธ์จาก task
let value = try await throwingTask.value
```

### Task Priority

```swift
// กำหนด Priority ของ Task
let highPriorityTask = Task(priority: .high) {
    await doImportantWork()
}

let lowPriorityTask = Task(priority: .low) {
    await doBackgroundWork()
}

let utilityTask = Task(priority: .utility) {
    await doUtilityWork()
}

// Priority levels:
// .high       - งานที่สำคัญมาก (เช่น user-initiated)
// .medium     - งานปกติ (ค่า default)
// .low        - งานที่ไม่ด่วน
// .utility    - งาน background ทั่วไป
// .background - งาน background ที่ไม่ด่วนมาก
// .userInitiated - งานที่ user เป็นคนสั่ง
```

### Task Value

```swift
// รับค่าจาก Task
func fetchAndProcess() async throws -> ProcessedData {
    let task = Task<ProcessedData, Error> {
        let rawData = try await fetchRawData()
        return try processData(rawData)
    }
    
    // รอและรับค่า
    return try await task.value
}

// Task ที่ไม่ Throw
let simpleTask = Task<String, Never> {
    return "Hello, World!"
}
let result: String = await simpleTask.value
```

### Task Cancellation

```swift
// ยกเลิก Task
let task = Task {
    for i in 1...100 {
        // ตรวจสอบว่าถูกยกเลิกหรือยัง
        try Task.checkCancellation()
        await processItem(i)
    }
}

// ยกเลิก task
task.cancel()
print("Task is cancelled: \(task.isCancelled)")
```

---

## 5. Task.detached

`Task.detached` สร้าง task ที่ไม่ inherit context จาก parent task

### ความแตกต่างระหว่าง Task และ Task.detached

```swift
class UserManager {
    var priority = TaskPriority.medium
    
    func doWork() async {
        // Task ปกติ - inherit priority และ task-local values จาก parent
        Task {
            // priority = parent's priority
            // task-local values = parent's values
            await performWork()
        }
        
        // Task.detached - ไม่ inherit อะไรเลย
        Task.detached {
            // priority = .medium (default)
            // task-local values = ไม่มี
            await performWork()
        }
    }
}
```

### เมื่อไหร่ควรใช้ Task.detached?

```swift
// ใช้ Task.detached เมื่อต้องการทำงานแบบ independent โดยสมบูรณ์
class DocumentProcessor {
    func saveDocument(_ doc: Document) async throws {
        // บันทึกข้อมูลหลัก
        try await database.save(doc)
        
        // สร้าง thumbnail แบบ detached - ไม่จำเป็นต้องรอ parent
        Task.detached(priority: .background) {
            // ทำงานแบบ background โดยไม่กระทบ parent
            let thumbnail = await generateThumbnail(for: doc)
            await imageCache.store(thumbnail, for: doc.id)
        }
        
        // ไม่ต้องรอ thumbnail - return ทันที
    }
}
```

### ตัวอย่างการใช้งาน Task.detached

```swift
struct Analytics {
    static func logEvent(_ event: String) {
        // Log analytics แบบ fire-and-forget
        Task.detached(priority: .background) {
            do {
                try await AnalyticsService.send(event: event)
            } catch {
                // ไม่ต้องสนใจ error สำหรับ analytics
                print("Analytics failed: \(error)")
            }
        }
    }
}

// การใช้งาน
class ProductViewController {
    func viewProduct(_ product: Product) {
        displayProduct(product)
        Analytics.logEvent("product_viewed_\(product.id)")  // fire and forget
    }
}
```

---

## 6. async let

`async let` ใช้สำหรับการทำงานแบบ concurrent หลายอย่างพร้อมกัน

### การใช้งานพื้นฐาน

```swift
// โดยไม่ใช้ async let - ทำงานตามลำดับ (Sequential)
func loadProfileSequential() async throws -> Profile {
    let user = try await fetchUser()          // รอ...
    let posts = try await fetchPosts()        // รอ...
    let followers = try await fetchFollowers() // รอ...
    // เวลารวม = เวลา user + เวลา posts + เวลา followers
    return Profile(user: user, posts: posts, followers: followers)
}

// โดยใช้ async let - ทำงานพร้อมกัน (Concurrent)
func loadProfileConcurrent() async throws -> Profile {
    async let user = fetchUser()
    async let posts = fetchPosts()
    async let followers = fetchFollowers()
    
    // ทุกอย่างเริ่มทำงานพร้อมกัน รอจนกว่าทั้งหมดเสร็จ
    // เวลารวม = เวลาของงานที่ช้าที่สุด
    return try await Profile(user: user, posts: posts, followers: followers)
}
```

### async let กับ Error Handling

```swift
func loadDashboard() async throws -> Dashboard {
    async let userStats = fetchUserStats()     // อาจ throw
    async let salesData = fetchSalesData()     // อาจ throw
    async let notifications = fetchNotifications() // อาจ throw
    
    // await ทั้งหมดพร้อมกัน
    // ถ้าอย่างใดอย่างหนึ่ง throw ก็จะ propagate ขึ้นมา
    return try await Dashboard(
        userStats: userStats,
        salesData: salesData,
        notifications: notifications
    )
}
```

### async let กับ Optional Results

```swift
func loadOptionalData() async -> (primary: Data, secondary: Data?) {
    async let primary = fetchPrimaryData()
    async let secondary = fetchSecondaryData()  // อาจ fail
    
    do {
        return try await (primary: primary, secondary: secondary)
    } catch {
        // secondary ล้มเหลว แต่เรายังต้องการ primary
        return (primary: try! await primary, secondary: nil)
    }
}
```

### ตัวอย่าง Real-World: Loading App Data

```swift
struct AppData {
    let user: User
    let settings: Settings
    let featuredContent: [Content]
}

class AppDataLoader {
    func loadInitialData() async throws -> AppData {
        print("เริ่มโหลดข้อมูล...")
        
        // โหลดทั้งหมดพร้อมกัน
        async let user = fetchCurrentUser()
        async let settings = fetchUserSettings()
        async let featured = fetchFeaturedContent()
        
        // รอและรวมผลลัพธ์
        let appData = try await AppData(
            user: user,
            settings: settings,
            featuredContent: featured
        )
        
        print("โหลดข้อมูลเสร็จแล้ว")
        return appData
    }
}
```

---

## 7. TaskGroup

`TaskGroup` ใช้สำหรับการจัดการ concurrent tasks จำนวนมากที่ไม่รู้จำนวนล่วงหน้า

### withTaskGroup

```swift
// withTaskGroup - สำหรับงานที่ไม่ throw error
func downloadAllImages(urls: [URL]) async -> [UIImage] {
    await withTaskGroup(of: UIImage?.self) { group in
        // เพิ่ม task สำหรับแต่ละ URL
        for url in urls {
            group.addTask {
                return await downloadImage(from: url)
            }
        }
        
        // รวบรวมผลลัพธ์
        var images: [UIImage] = []
        for await image in group {
            if let image = image {
                images.append(image)
            }
        }
        return images
    }
}
```

### withThrowingTaskGroup

```swift
// withThrowingTaskGroup - สำหรับงานที่อาจ throw error
func fetchAllUsers(ids: [Int]) async throws -> [User] {
    try await withThrowingTaskGroup(of: User.self) { group in
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
```

### TaskGroup กับ Results ที่มี Index

```swift
// เก็บ index พร้อมกับผลลัพธ์
func downloadImagesInOrder(urls: [URL]) async -> [UIImage?] {
    await withTaskGroup(of: (Int, UIImage?).self) { group in
        for (index, url) in urls.enumerated() {
            group.addTask {
                let image = await downloadImage(from: url)
                return (index, image)
            }
        }
        
        var results = [UIImage?](repeating: nil, count: urls.count)
        for await (index, image) in group {
            results[index] = image
        }
        return results
    }
}
```

### TaskGroup กับ Cancellation

```swift
// หยุดทำงานทันทีเมื่อ error แรกเกิดขึ้น
func fetchFirstSuccessfulData(from urls: [URL]) async throws -> Data {
    try await withThrowingTaskGroup(of: Data.self) { group in
        for url in urls {
            group.addTask {
                return try await URLSession.shared.data(from: url).0
            }
        }
        
        // รับผลแรกที่สำเร็จ แล้วยกเลิกที่เหลือ
        guard let firstResult = try await group.next() else {
            throw NetworkError.allFailed
        }
        
        // ยกเลิก tasks ที่เหลือ
        group.cancelAll()
        return firstResult
    }
}
```

### ตัวอย่าง: Batch Processing

```swift
struct ImageProcessor {
    func processImages(_ images: [UIImage], maxConcurrent: Int = 5) async -> [ProcessedImage] {
        // แบ่งงานเป็น batches
        let batches = images.chunked(into: maxConcurrent)
        var processedImages: [ProcessedImage] = []
        
        for batch in batches {
            let batchResults = await withTaskGroup(of: ProcessedImage?.self) { group in
                for image in batch {
                    group.addTask {
                        return await processImage(image)
                    }
                }
                
                var results: [ProcessedImage] = []
                for await result in group {
                    if let result = result {
                        results.append(result)
                    }
                }
                return results
            }
            processedImages.append(contentsOf: batchResults)
        }
        
        return processedImages
    }
}
```

---

## 8. Structured vs Unstructured Concurrency

### Structured Concurrency

ใน Structured Concurrency, tasks มีชีวิตอยู่ภายใน scope ที่กำหนดไว้

```swift
// Structured - tasks ถูก scope โดย withTaskGroup หรือ async let
func processDataStructured() async throws -> [Result] {
    // ทุก task ใน group จะเสร็จก่อนที่ function จะ return
    return try await withThrowingTaskGroup(of: Result.self) { group in
        for item in items {
            group.addTask {
                return try await process(item)
            }
        }
        
        var results: [Result] = []
        for try await result in group {
            results.append(result)
        }
        return results
    }
    // ออกจาก withTaskGroup = ทุก task เสร็จหรือถูกยกเลิกแล้ว
}
```

ข้อดีของ Structured Concurrency:
1. **Automatic Cancellation** - เมื่อ parent ถูกยกเลิก child ก็ถูกยกเลิกด้วย
2. **Error Propagation** - errors propagate ขึ้นไปยัง parent
3. **Resource Management** - tasks ถูก clean up อัตโนมัติ
4. **Scope Control** - รู้ว่า task มีชีวิตอยู่นานแค่ไหน

```swift
// ตัวอย่าง Automatic Cancellation
func downloadAndProcess() async throws {
    try await withThrowingTaskGroup(of: Void.self) { group in
        group.addTask { try await downloadData() }
        group.addTask { try await processInBackground() }
        group.addTask { try await updateCache() }
        
        // ถ้า downloadData() throw error:
        // - tasks อื่นๆ จะถูกยกเลิกอัตโนมัติ
        // - error จะ propagate ออกมา
        for try await _ in group {}
    }
}
```

### Unstructured Concurrency

`Task` และ `Task.detached` สร้าง unstructured tasks:

```swift
class NewsViewController: UIViewController {
    var refreshTask: Task<Void, Error>?
    
    // Unstructured - task ไม่ผูกกับ scope ใดๆ
    func refreshNews() {
        // ยกเลิก task เก่าก่อน
        refreshTask?.cancel()
        
        // สร้าง task ใหม่
        refreshTask = Task {
            do {
                let articles = try await newsService.fetchLatest()
                await MainActor.run {
                    self.articles = articles
                    self.tableView.reloadData()
                }
            } catch {
                await MainActor.run {
                    self.showError(error)
                }
            }
        }
    }
    
    deinit {
        // ต้องยกเลิกเอง!
        refreshTask?.cancel()
    }
}
```

### เปรียบเทียบ Structured vs Unstructured

| ลักษณะ | Structured | Unstructured |
|--------|-----------|--------------|
| Scope | ผูกกับ scope | อิสระ |
| Cancellation | อัตโนมัติ | ต้องจัดการเอง |
| Lifetime | จำกัดด้วย scope | ขึ้นอยู่กับ reference |
| เหมาะสำหรับ | งาน parallel ที่รู้จำนวน | งาน background ที่ต้องควบคุมเอง |

---

## 9. Actor

Actor เป็น reference type ใหม่ที่ออกแบบมาเพื่อป้องกัน data races โดยอัตโนมัติ

### Actor พื้นฐาน

```swift
// Actor ประกาศด้วย keyword 'actor'
actor BankAccount {
    private var balance: Double = 0
    private var transactionHistory: [Transaction] = []
    
    // Methods ใน actor ทำงาน on actor's serialized executor
    func deposit(_ amount: Double) {
        balance += amount
        transactionHistory.append(Transaction(type: .deposit, amount: amount))
    }
    
    func withdraw(_ amount: Double) throws {
        guard balance >= amount else {
            throw BankError.insufficientFunds
        }
        balance -= amount
        transactionHistory.append(Transaction(type: .withdrawal, amount: amount))
    }
    
    func getBalance() -> Double {
        return balance
    }
}

// การใช้งาน - ต้องใช้ await เมื่อเข้าถึงจากภายนอก
func performTransaction() async throws {
    let account = BankAccount()
    
    await account.deposit(1000)
    try await account.withdraw(500)
    
    let balance = await account.getBalance()
    print("ยอดคงเหลือ: \(balance)")
}
```

### Actor Isolation

Actor มี concept ของ isolation - โค้ดใน actor ทำงานได้เพียงทีละ task เดียว:

```swift
actor Counter {
    private var count = 0
    
    // nonisolated - เข้าถึงได้โดยไม่ต้อง await
    nonisolated let id = UUID()
    
    // Isolated - ต้อง await เมื่อเรียกจากนอก actor
    func increment() {
        count += 1  // ปลอดภัย ไม่มี race condition
    }
    
    func getCount() -> Int {
        return count
    }
    
    // nonisolated method - ไม่เข้าถึง mutable state
    nonisolated func description() -> String {
        return "Counter \(id)"
    }
}

// การใช้งาน
func countItems() async {
    let counter = Counter()
    
    // เรียก isolated methods ต้อง await
    await counter.increment()
    await counter.increment()
    let count = await counter.getCount()
    print("Count: \(count)")  // 2
    
    // เรียก nonisolated method - ไม่ต้อง await
    print(counter.description())
}
```

### ปัญหาที่ Actor แก้ได้

```swift
// โดยไม่ใช้ Actor - Data Race!
class UnsafeCounter {
    var count = 0
    
    func increment() {
        count += 1  // ไม่ปลอดภัย! หลาย threads อาจอ่าน/เขียนพร้อมกัน
    }
}

// ใช้ Actor - ปลอดภัย
actor SafeCounter {
    var count = 0
    
    func increment() {
        count += 1  // Actor รับประกัน serial access
    }
    
    func incrementMultiple(_ times: Int) {
        for _ in 0..<times {
            count += 1  // ทำงานใน actor's isolated context
        }
    }
}

// ทดสอบ
func testConcurrentIncrement() async {
    let counter = SafeCounter()
    
    // สร้าง 1000 tasks ที่ increment พร้อมกัน
    await withTaskGroup(of: Void.self) { group in
        for _ in 0..<1000 {
            group.addTask {
                await counter.increment()
            }
        }
    }
    
    let finalCount = await counter.count
    print("Final count: \(finalCount)")  // รับประกันว่าจะได้ 1000 เสมอ
}
```

### Actor ที่ซับซ้อนขึ้น

```swift
actor ImageCache {
    private var cache: [URL: UIImage] = [:]
    private var inProgressDownloads: [URL: Task<UIImage, Error>] = [:]
    
    func image(for url: URL) async throws -> UIImage {
        // ถ้ามีใน cache แล้ว return ทันที
        if let cached = cache[url] {
            return cached
        }
        
        // ถ้ากำลัง download อยู่ รอผลลัพธ์จาก task นั้น
        if let existingTask = inProgressDownloads[url] {
            return try await existingTask.value
        }
        
        // เริ่ม download ใหม่
        let downloadTask = Task<UIImage, Error> {
            let (data, _) = try await URLSession.shared.data(from: url)
            guard let image = UIImage(data: data) else {
                throw ImageError.invalidData
            }
            return image
        }
        
        inProgressDownloads[url] = downloadTask
        
        do {
            let image = try await downloadTask.value
            cache[url] = image
            inProgressDownloads.removeValue(forKey: url)
            return image
        } catch {
            inProgressDownloads.removeValue(forKey: url)
            throw error
        }
    }
    
    func clearCache() {
        cache.removeAll()
    }
}
```

---

## 10. MainActor

`MainActor` เป็น actor พิเศษที่ทำงานบน Main Thread ซึ่งสำคัญสำหรับการอัปเดต UI

### ใช้งาน MainActor

```swift
// รัน code บน Main Thread
func updateUI(with data: Data) async {
    let processedData = await processInBackground(data)
    
    // อัปเดต UI ต้องทำบน Main Thread
    await MainActor.run {
        self.tableView.reloadData()
        self.loadingIndicator.stopAnimating()
        self.titleLabel.text = "โหลดเสร็จแล้ว"
    }
}

// หรือใช้ MainActor.run แบบ async
func fetchAndDisplay() async throws {
    let data = try await fetchData()
    
    await MainActor.run {
        self.data = data
        self.collectionView.reloadData()
    }
}
```

### @MainActor Attribute

```swift
// ประกาศ class/struct/function ทั้งหมดให้ทำงานบน Main Actor
@MainActor
class ViewController: UIViewController {
    var items: [Item] = [] {
        didSet {
            tableView.reloadData()  // ปลอดภัย เพราะอยู่บน Main Thread
        }
    }
    
    func loadData() async throws {
        // ออกจาก Main Thread เพื่อทำงาน async
        let data = try await fetchDataInBackground()
        
        // กลับมา Main Thread อัตโนมัติเพราะ @MainActor
        items = processData(data)
    }
}
```

### @MainActor กับ Functions

```swift
// ทำให้ function เฉพาะทำงานบน Main Thread
class DataManager {
    @MainActor
    func updateDisplay(with data: [Item]) {
        // Function นี้รับประกันว่าทำงานบน Main Thread
        viewController.items = data
        viewController.tableView.reloadData()
    }
    
    func processAndUpdate() async throws {
        let data = try await fetchData()
        let processed = await processData(data)
        
        // เรียก @MainActor function - จะ switch to main thread อัตโนมัติ
        await updateDisplay(with: processed)
    }
}
```

### ตัวอย่าง Complete: ViewModel กับ @MainActor

```swift
@MainActor
class ArticleViewModel: ObservableObject {
    @Published var articles: [Article] = []
    @Published var isLoading = false
    @Published var errorMessage: String?
    
    private let service: ArticleService
    
    init(service: ArticleService = .shared) {
        self.service = service
    }
    
    func loadArticles() async {
        isLoading = true
        errorMessage = nil
        
        do {
            // nonisolated context - ออกจาก MainActor ชั่วคราว
            let fetchedArticles = try await service.fetchArticles()
            
            // กลับมา MainActor อัตโนมัติ
            articles = fetchedArticles
        } catch {
            errorMessage = error.localizedDescription
        }
        
        isLoading = false
    }
    
    func deleteArticle(_ article: Article) async {
        guard let index = articles.firstIndex(of: article) else { return }
        articles.remove(at: index)
        
        do {
            try await service.delete(article: article)
        } catch {
            // Roll back
            articles.insert(article, at: index)
            errorMessage = "ลบไม่สำเร็จ: \(error.localizedDescription)"
        }
    }
}
```

---

## 11. Sendable Protocol

`Sendable` protocol ระบุว่า type นั้นปลอดภัยที่จะส่งข้ามไปยัง concurrency domain ต่างๆ

### ทำไมต้องมี Sendable?

```swift
// ปัญหา: ส่ง reference type ข้าม actors อาจเกิด data race
actor DataProcessor {
    func process(_ mutableObject: NSMutableArray) {
        // ถ้า caller ยังถือ reference ด้วย อาจเกิด race condition!
        mutableObject.add("processed")
    }
}
```

### Types ที่เป็น Sendable โดยอัตโนมัติ

```swift
// Value types ทั่วไปเป็น Sendable
let number: Int = 42         // Sendable
let text: String = "hello"   // Sendable
let flag: Bool = true        // Sendable
let data: Data = Data()      // Sendable

// Struct ที่มีแต่ Sendable properties
struct Point: Sendable {
    let x: Double
    let y: Double
}

// Enum ที่มีแต่ Sendable associated values
enum Status: Sendable {
    case active(String)
    case inactive
}
```

### การ Conform ให้กับ Sendable

```swift
// Struct ปกติ - conform อัตโนมัติถ้าทุก property เป็น Sendable
struct UserProfile: Sendable {
    let id: Int
    let name: String
    let email: String
    // ทุก property เป็น Sendable ดังนั้น UserProfile เป็น Sendable
}

// Class ที่ต้องการ Sendable
final class ImmutableConfig: Sendable {
    let apiKey: String
    let baseURL: URL
    
    init(apiKey: String, baseURL: URL) {
        self.apiKey = apiKey
        self.baseURL = baseURL
    }
    // final + ไม่มี mutable state = Sendable
}

// @unchecked Sendable - บอก compiler ว่าเราจัดการ thread safety เอง
class ThreadSafeCache: @unchecked Sendable {
    private var cache: [String: Data] = [:]
    private let lock = NSLock()
    
    func get(_ key: String) -> Data? {
        lock.lock()
        defer { lock.unlock() }
        return cache[key]
    }
    
    func set(_ key: String, value: Data) {
        lock.lock()
        defer { lock.unlock() }
        cache[key] = value
    }
}
```

### @Sendable Closures

```swift
// @Sendable ระบุว่า closure ปลอดภัยที่จะใช้ใน concurrent context
func performConcurrently(_ work: @Sendable () async -> Void) async {
    await withTaskGroup(of: Void.self) { group in
        for _ in 0..<10 {
            group.addTask {
                await work()
            }
        }
    }
}

// ข้อจำกัดของ @Sendable closure:
// ไม่สามารถ capture mutable values โดย reference
var counter = 0
// Error: ไม่สามารถ capture mutable variable ใน @Sendable closure
// Task { counter += 1 }  // Compile error

// ต้องใช้ actor หรือ atomic operations แทน
actor AtomicCounter {
    var value = 0
    func increment() { value += 1 }
}

let atomicCounter = AtomicCounter()
Task {
    await atomicCounter.increment()  // ปลอดภัย
}
```

---

## 12. Continuation

Continuation ใช้สำหรับการแปลง callback-based APIs เป็น async/await

### CheckedContinuation

```swift
// withCheckedContinuation - สำหรับ non-throwing
func fetchDataWithCallback() async -> Data {
    await withCheckedContinuation { continuation in
        // เรียก API แบบเดิมที่ใช้ callback
        legacyAPI.fetchData { data in
            continuation.resume(returning: data)
        }
    }
}

// withCheckedThrowingContinuation - สำหรับ throwing
func fetchUserWithCallback(id: Int) async throws -> User {
    try await withCheckedThrowingContinuation { continuation in
        legacyAPI.fetchUser(id: id) { result in
            switch result {
            case .success(let user):
                continuation.resume(returning: user)
            case .failure(let error):
                continuation.resume(throwing: error)
            }
        }
    }
}
```

### UnsafeContinuation

```swift
// UnsafeContinuation - เร็วกว่า แต่ไม่มี runtime checks
func fetchDataFast() async -> Data {
    await withUnsafeContinuation { continuation in
        legacyAPI.fetchData { data in
            // ต้องระวัง: resume ต้องถูกเรียกแน่นอนและครั้งเดียว
            continuation.resume(returning: data)
        }
    }
}
```

### กฎสำคัญของ Continuation

```swift
// กฎ: resume ต้องถูกเรียกแน่นอนและเพียงครั้งเดียว
func correctUsage() async -> String {
    return await withCheckedContinuation { continuation in
        someAsyncOperation { result, error in
            if let error = error {
                // ❌ ไม่ดี: resume ถูกเรียก แต่ error ถูกทิ้ง
                continuation.resume(returning: "error occurred")
            } else {
                continuation.resume(returning: result ?? "")
            }
        }
    }
}

// ✅ ดีกว่า: ใช้ withCheckedThrowingContinuation
func betterUsage() async throws -> String {
    return try await withCheckedThrowingContinuation { continuation in
        someAsyncOperation { result, error in
            if let error = error {
                continuation.resume(throwing: error)
            } else if let result = result {
                continuation.resume(returning: result)
            } else {
                continuation.resume(throwing: APIError.unknownError)
            }
        }
    }
}
```

### Converting Delegate-based APIs

```swift
// Delegate-based API เดิม
protocol LocationManagerDelegate {
    func didUpdateLocation(_ location: CLLocation)
    func didFailWithError(_ error: Error)
}

class LegacyLocationManager: NSObject {
    var delegate: LocationManagerDelegate?
    
    func requestLocation() {
        // ขอ location...
    }
}

// แปลงเป็น async
class AsyncLocationManager: NSObject, LocationManagerDelegate {
    private var continuation: CheckedContinuation<CLLocation, Error>?
    private let manager = LegacyLocationManager()
    
    override init() {
        super.init()
        manager.delegate = self
    }
    
    func getCurrentLocation() async throws -> CLLocation {
        try await withCheckedThrowingContinuation { continuation in
            self.continuation = continuation
            manager.requestLocation()
        }
    }
    
    // MARK: - LocationManagerDelegate
    func didUpdateLocation(_ location: CLLocation) {
        continuation?.resume(returning: location)
        continuation = nil
    }
    
    func didFailWithError(_ error: Error) {
        continuation?.resume(throwing: error)
        continuation = nil
    }
}

// การใช้งาน
func showCurrentLocation() async {
    let locationManager = AsyncLocationManager()
    
    do {
        let location = try await locationManager.getCurrentLocation()
        print("Location: \(location.coordinate.latitude), \(location.coordinate.longitude)")
    } catch {
        print("Error: \(error)")
    }
}
```

---

## 13. AsyncSequence และ AsyncStream

### AsyncSequence

`AsyncSequence` เป็น protocol สำหรับ sequences ที่ produce elements แบบ asynchronous

```swift
// การใช้ AsyncSequence กับ for await
func processAsyncSequence() async {
    // URLSession bytes เป็น AsyncSequence
    let url = URL(string: "https://api.example.com/events")!
    let (bytes, _) = try! await URLSession.shared.bytes(from: url)
    
    for try await line in bytes.lines {
        print("Received: \(line)")
    }
}

// Custom AsyncSequence
struct NumberSequence: AsyncSequence {
    typealias Element = Int
    
    let range: ClosedRange<Int>
    let delay: Duration
    
    func makeAsyncIterator() -> AsyncIterator {
        return AsyncIterator(range: range, delay: delay)
    }
    
    struct AsyncIterator: AsyncIteratorProtocol {
        let range: ClosedRange<Int>
        let delay: Duration
        var current: Int
        
        init(range: ClosedRange<Int>, delay: Duration) {
            self.range = range
            self.delay = delay
            self.current = range.lowerBound
        }
        
        mutating func next() async throws -> Int? {
            guard current <= range.upperBound else { return nil }
            
            try await Task.sleep(for: delay)
            let value = current
            current += 1
            return value
        }
    }
}

// การใช้งาน
func countdown() async throws {
    let sequence = NumberSequence(range: 1...10, delay: .seconds(1))
    
    for try await number in sequence {
        print("Number: \(number)")
    }
    
    print("Done!")
}
```

### AsyncStream

`AsyncStream` ใช้สำหรับสร้าง async sequences จาก callback-based APIs หรือ event streams

```swift
// สร้าง AsyncStream พื้นฐาน
func makeCounterStream(from start: Int, to end: Int) -> AsyncStream<Int> {
    AsyncStream { continuation in
        Task {
            for i in start...end {
                continuation.yield(i)
                try await Task.sleep(for: .milliseconds(500))
            }
            continuation.finish()
        }
    }
}

// การใช้งาน
func consumeStream() async {
    let stream = makeCounterStream(from: 1, to: 5)
    
    for await value in stream {
        print("Value: \(value)")
    }
    print("Stream ended")
}
```

### AsyncStream จาก Delegate/Callback API

```swift
// แปลง NotificationCenter เป็น AsyncStream
extension NotificationCenter {
    func notifications(named name: NSNotification.Name) -> AsyncStream<Notification> {
        AsyncStream { continuation in
            let observer = addObserver(forName: name, object: nil, queue: nil) { notification in
                continuation.yield(notification)
            }
            
            continuation.onTermination = { _ in
                self.removeObserver(observer)
            }
        }
    }
}

// การใช้งาน
func observeKeyboardNotifications() async {
    for await notification in NotificationCenter.default.notifications(named: UIResponder.keyboardWillShowNotification) {
        guard let userInfo = notification.userInfo,
              let keyboardFrame = userInfo[UIResponder.keyboardFrameEndUserInfoKey] as? CGRect else {
            continue
        }
        print("Keyboard height: \(keyboardFrame.height)")
    }
}
```

### AsyncThrowingStream

```swift
// AsyncThrowingStream - สำหรับ stream ที่อาจ throw error
func createEventStream() -> AsyncThrowingStream<Event, Error> {
    AsyncThrowingStream { continuation in
        let connection = WebSocketConnection()
        
        connection.onReceive = { data in
            do {
                let event = try JSONDecoder().decode(Event.self, from: data)
                continuation.yield(event)
            } catch {
                continuation.finish(throwing: error)
            }
        }
        
        connection.onDisconnect = { error in
            if let error = error {
                continuation.finish(throwing: error)
            } else {
                continuation.finish()
            }
        }
        
        continuation.onTermination = { _ in
            connection.disconnect()
        }
        
        connection.connect()
    }
}

// การใช้งาน
func processEvents() async throws {
    for try await event in createEventStream() {
        await handleEvent(event)
    }
}
```

### for await in loop

```swift
// for await in ใช้กับ AsyncSequence ใดก็ได้
func processAllData() async throws {
    // กับ array ที่เป็น AsyncSequence
    let numbers = AsyncStream<Int> { continuation in
        for i in 1...10 {
            continuation.yield(i)
        }
        continuation.finish()
    }
    
    // iterating
    for await number in numbers {
        print(number)
    }
    
    // กับ filter และ map
    let evenNumbers = numbers.filter { $0 % 2 == 0 }
    for await even in evenNumbers {
        print("Even: \(even)")
    }
    
    // break และ continue ทำงานปกติ
    for await number in numbers {
        if number > 5 { break }
        if number % 2 != 0 { continue }
        print("Even up to 5: \(number)")
    }
}
```

---

## 14. Task Cancellation

### การยกเลิก Task

```swift
// สร้างและยกเลิก task
let task = Task {
    try await longRunningWork()
}

// ยกเลิกหลังจาก 5 วินาที
try await Task.sleep(for: .seconds(5))
task.cancel()
```

### ตรวจสอบ Task.isCancelled

```swift
func downloadLargeFile(url: URL) async throws -> Data {
    var downloadedData = Data()
    
    let (bytes, _) = try await URLSession.shared.bytes(from: url)
    
    for try await byte in bytes {
        // ตรวจสอบทุก iteration
        if Task.isCancelled {
            throw CancellationError()
        }
        downloadedData.append(byte)
    }
    
    return downloadedData
}

// Task.checkCancellation() - throw CancellationError ถ้าถูกยกเลิก
func processItems(_ items: [Item]) async throws -> [ProcessedItem] {
    var results: [ProcessedItem] = []
    
    for item in items {
        try Task.checkCancellation()  // throw ถ้าถูกยกเลิก
        let processed = try await process(item)
        results.append(processed)
    }
    
    return results
}
```

### withTaskCancellationHandler

```swift
// จัดการ cleanup เมื่อถูกยกเลิก
func downloadWithCleanup(url: URL) async throws -> Data {
    let connection = NetworkConnection()
    
    return try await withTaskCancellationHandler {
        // งานหลัก
        return try await connection.download(from: url)
    } onCancel: {
        // ถูกเรียกเมื่อ task ถูกยกเลิก
        connection.cancel()
    }
}
```

### Cooperative Cancellation

```swift
// Cancellation ใน Swift เป็นแบบ Cooperative
// Task ที่ถูกยกเลิกต้องตรวจสอบและหยุดการทำงานเอง
func cooperativeCancellation() async throws {
    for i in 1...Int.max {
        // ตรวจสอบทุก iteration
        try Task.checkCancellation()
        
        // หรือ
        guard !Task.isCancelled else {
            print("Task cancelled at iteration \(i)")
            throw CancellationError()
        }
        
        await doSomeWork(i)
    }
}
```

### Timeout Pattern

```swift
// สร้าง timeout ด้วย Task
func fetchWithTimeout<T>(
    operation: @escaping () async throws -> T,
    timeout: Duration
) async throws -> T {
    try await withThrowingTaskGroup(of: T.self) { group in
        // เพิ่ม task หลัก
        group.addTask {
            return try await operation()
        }
        
        // เพิ่ม timeout task
        group.addTask {
            try await Task.sleep(for: timeout)
            throw TimeoutError()
        }
        
        // รับผลแรกที่ได้ (หลักหรือ timeout)
        let result = try await group.next()!
        group.cancelAll()  // ยกเลิก task ที่เหลือ
        return result
    }
}

// การใช้งาน
func loadDataWithTimeout() async throws -> Data {
    return try await fetchWithTimeout(
        operation: { try await fetchData() },
        timeout: .seconds(30)
    )
}
```

---

## 15. Converting Callback APIs to async

### Pattern พื้นฐาน

```swift
// API เดิม
class LegacyNetworkClient {
    func request(
        url: URL,
        completion: @escaping (Data?, URLResponse?, Error?) -> Void
    ) {
        URLSession.shared.dataTask(with: url, completionHandler: completion).resume()
    }
}

// แปลงเป็น async
extension LegacyNetworkClient {
    func request(url: URL) async throws -> (Data, URLResponse) {
        try await withCheckedThrowingContinuation { continuation in
            self.request(url: url) { data, response, error in
                if let error = error {
                    continuation.resume(throwing: error)
                } else if let data = data, let response = response {
                    continuation.resume(returning: (data, response))
                } else {
                    continuation.resume(throwing: NetworkError.unknown)
                }
            }
        }
    }
}
```

### Converting CLLocationManager

```swift
import CoreLocation

class LocationService: NSObject {
    private var locationContinuation: CheckedContinuation<CLLocation, Error>?
    private let manager = CLLocationManager()
    
    override init() {
        super.init()
        manager.delegate = self
    }
    
    func getCurrentLocation() async throws -> CLLocation {
        try await withCheckedThrowingContinuation { continuation in
            locationContinuation = continuation
            manager.requestWhenInUseAuthorization()
            manager.requestLocation()
        }
    }
}

extension LocationService: CLLocationManagerDelegate {
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        guard let location = locations.first else { return }
        locationContinuation?.resume(returning: location)
        locationContinuation = nil
    }
    
    func locationManager(_ manager: CLLocationManager, didFailWithError error: Error) {
        locationContinuation?.resume(throwing: error)
        locationContinuation = nil
    }
}
```

### Converting Notification-based APIs

```swift
// รอ notification เพียงครั้งเดียว
func waitForKeyboardShow() async -> CGFloat {
    await withCheckedContinuation { continuation in
        var observer: NSObjectProtocol?
        observer = NotificationCenter.default.addObserver(
            forName: UIResponder.keyboardWillShowNotification,
            object: nil,
            queue: .main
        ) { notification in
            let height = (notification.userInfo?[UIResponder.keyboardFrameEndUserInfoKey] as? CGRect)?.height ?? 0
            
            continuation.resume(returning: height)
            
            // Remove observer หลังจากรับ notification แล้ว
            if let observer = observer {
                NotificationCenter.default.removeObserver(observer)
            }
        }
    }
}
```

---

## 16. Practical Exercises

### Exercise 1: Async Image Loader

```swift
import UIKit

// MARK: - Models
struct ImageRequest {
    let url: URL
    let priority: TaskPriority
}

// MARK: - Errors
enum ImageLoaderError: Error {
    case invalidURL
    case downloadFailed
    case invalidImageData
}

// MARK: - ImageLoader Actor
actor AsyncImageLoader {
    private var cache: [URL: UIImage] = [:]
    private var activeTasks: [URL: Task<UIImage, Error>] = [:]
    private let session: URLSession
    
    init(configuration: URLSessionConfiguration = .default) {
        self.session = URLSession(configuration: configuration)
    }
    
    func loadImage(from url: URL, priority: TaskPriority = .medium) async throws -> UIImage {
        // Check cache first
        if let cached = cache[url] {
            return cached
        }
        
        // If already downloading, wait for that task
        if let existingTask = activeTasks[url] {
            return try await existingTask.value
        }
        
        // Start new download
        let task = Task(priority: priority) {
            return try await self.downloadImage(from: url)
        }
        
        activeTasks[url] = task
        
        do {
            let image = try await task.value
            cache[url] = image
            activeTasks.removeValue(forKey: url)
            return image
        } catch {
            activeTasks.removeValue(forKey: url)
            throw error
        }
    }
    
    private func downloadImage(from url: URL) async throws -> UIImage {
        let (data, response) = try await session.data(from: url)
        
        guard let httpResponse = response as? HTTPURLResponse,
              (200...299).contains(httpResponse.statusCode) else {
            throw ImageLoaderError.downloadFailed
        }
        
        guard let image = UIImage(data: data) else {
            throw ImageLoaderError.invalidImageData
        }
        
        return image
    }
    
    func prefetch(urls: [URL]) async {
        await withTaskGroup(of: Void.self) { group in
            for url in urls {
                group.addTask {
                    _ = try? await self.loadImage(from: url, priority: .low)
                }
            }
        }
    }
    
    func clearCache() {
        cache.removeAll()
        activeTasks.values.forEach { $0.cancel() }
        activeTasks.removeAll()
    }
    
    var cacheSize: Int {
        cache.count
    }
}

// MARK: - UIImageView Extension
extension UIImageView {
    private static let imageLoader = AsyncImageLoader()
    private static var taskKey: UInt8 = 0
    
    func loadImage(from urlString: String, placeholder: UIImage? = nil) {
        guard let url = URL(string: urlString) else {
            image = placeholder
            return
        }
        
        // ยกเลิก task เก่า
        currentLoadTask?.cancel()
        image = placeholder
        
        currentLoadTask = Task { @MainActor in
            do {
                let loadedImage = try await UIImageView.imageLoader.loadImage(from: url)
                if !Task.isCancelled {
                    self.image = loadedImage
                }
            } catch {
                if !Task.isCancelled {
                    self.image = placeholder
                }
            }
        }
    }
    
    private var currentLoadTask: Task<Void, Never>? {
        get { objc_getAssociatedObject(self, &UIImageView.taskKey) as? Task<Void, Never> }
        set { objc_setAssociatedObject(self, &UIImageView.taskKey, newValue, .OBJC_ASSOCIATION_RETAIN_NONATOMIC) }
    }
}

// MARK: - Test
struct ImageLoaderTest {
    static func runTest() async {
        let loader = AsyncImageLoader()
        
        let testURLs = [
            URL(string: "https://picsum.photos/200/300")!,
            URL(string: "https://picsum.photos/200/300?random=1")!,
            URL(string: "https://picsum.photos/200/300?random=2")!
        ]
        
        print("เริ่มโหลดรูป \(testURLs.count) รูปพร้อมกัน...")
        
        let startTime = Date()
        
        do {
            let images = try await withThrowingTaskGroup(of: UIImage.self) { group in
                for url in testURLs {
                    group.addTask {
                        return try await loader.loadImage(from: url)
                    }
                }
                
                var results: [UIImage] = []
                for try await image in group {
                    results.append(image)
                }
                return results
            }
            
            let elapsed = Date().timeIntervalSince(startTime)
            print("โหลดสำเร็จ \(images.count) รูปใน \(String(format: "%.2f", elapsed)) วินาที")
        } catch {
            print("เกิดข้อผิดพลาด: \(error)")
        }
    }
}
```

### Exercise 2: Async Data Fetcher with Retry

```swift
// MARK: - Retry Configuration
struct RetryConfig {
    let maxAttempts: Int
    let delay: Duration
    let backoffMultiplier: Double
    
    static let `default` = RetryConfig(
        maxAttempts: 3,
        delay: .seconds(1),
        backoffMultiplier: 2.0
    )
}

// MARK: - Generic Data Fetcher
actor AsyncDataFetcher {
    private let session: URLSession
    private let decoder: JSONDecoder
    
    init() {
        let config = URLSessionConfiguration.default
        config.timeoutIntervalForRequest = 30
        self.session = URLSession(configuration: config)
        
        self.decoder = JSONDecoder()
        self.decoder.keyDecodingStrategy = .convertFromSnakeCase
        self.decoder.dateDecodingStrategy = .iso8601
    }
    
    func fetch<T: Decodable>(
        _ type: T.Type,
        from url: URL,
        retry: RetryConfig = .default
    ) async throws -> T {
        var lastError: Error?
        var currentDelay = retry.delay
        
        for attempt in 1...retry.maxAttempts {
            do {
                let (data, response) = try await session.data(from: url)
                
                guard let httpResponse = response as? HTTPURLResponse else {
                    throw NetworkError.invalidResponse
                }
                
                switch httpResponse.statusCode {
                case 200...299:
                    return try decoder.decode(T.self, from: data)
                case 429:
                    // Rate limited - รอแล้วลองใหม่
                    try await Task.sleep(for: currentDelay)
                    currentDelay = .seconds(currentDelay.components.seconds * Int64(retry.backoffMultiplier))
                    lastError = NetworkError.rateLimited
                case 500...599:
                    // Server error - retry
                    lastError = NetworkError.serverError(httpResponse.statusCode)
                    if attempt < retry.maxAttempts {
                        try await Task.sleep(for: currentDelay)
                        currentDelay = .seconds(currentDelay.components.seconds * Int64(retry.backoffMultiplier))
                    }
                default:
                    throw NetworkError.httpError(httpResponse.statusCode)
                }
            } catch is CancellationError {
                throw CancellationError()
            } catch {
                lastError = error
                if attempt < retry.maxAttempts {
                    try await Task.sleep(for: currentDelay)
                    currentDelay = .seconds(currentDelay.components.seconds * Int64(retry.backoffMultiplier))
                }
            }
            
            print("Attempt \(attempt) failed, retrying...")
        }
        
        throw lastError ?? NetworkError.unknown
    }
    
    func fetchMultiple<T: Decodable>(
        _ type: T.Type,
        from urls: [URL],
        maxConcurrent: Int = 5
    ) async throws -> [T] {
        try await withThrowingTaskGroup(of: T.self) { group in
            var results: [T] = []
            var pendingURLs = urls
            
            // เพิ่ม tasks เริ่มต้น
            let initial = min(maxConcurrent, pendingURLs.count)
            for _ in 0..<initial {
                let url = pendingURLs.removeFirst()
                group.addTask {
                    return try await self.fetch(type, from: url)
                }
            }
            
            // รับผลและเพิ่ม tasks ใหม่
            for try await result in group {
                results.append(result)
                
                if let nextURL = pendingURLs.first {
                    pendingURLs.removeFirst()
                    group.addTask {
                        return try await self.fetch(type, from: nextURL)
                    }
                }
            }
            
            return results
        }
    }
}

// MARK: - Models
struct Post: Decodable {
    let id: Int
    let title: String
    let body: String
    let userId: Int
}

struct User: Decodable {
    let id: Int
    let name: String
    let email: String
}

// MARK: - Network Errors
enum NetworkError: Error, LocalizedError {
    case invalidResponse
    case rateLimited
    case serverError(Int)
    case httpError(Int)
    case unknown
    
    var errorDescription: String? {
        switch self {
        case .invalidResponse: return "Invalid response from server"
        case .rateLimited: return "Rate limited - too many requests"
        case .serverError(let code): return "Server error: \(code)"
        case .httpError(let code): return "HTTP error: \(code)"
        case .unknown: return "Unknown error"
        }
    }
}

// MARK: - Usage Example
struct DataFetcherExample {
    static func runDemo() async {
        let fetcher = AsyncDataFetcher()
        
        print("กำลังดึงข้อมูล posts...")
        
        do {
            // ดึงข้อมูลเดี่ยว
            let post = try await fetcher.fetch(
                Post.self,
                from: URL(string: "https://jsonplaceholder.typicode.com/posts/1")!
            )
            print("Post: \(post.title)")
            
            // ดึงข้อมูลหลายรายการพร้อมกัน
            let urls = (1...10).map { id in
                URL(string: "https://jsonplaceholder.typicode.com/posts/\(id)")!
            }
            
            let posts = try await fetcher.fetchMultiple(Post.self, from: urls, maxConcurrent: 3)
            print("โหลดได้ \(posts.count) posts")
            
        } catch {
            print("Error: \(error)")
        }
    }
}
```

### Exercise 3: Real-time Progress Tracking

```swift
// MARK: - Progress Tracking with AsyncStream
struct DownloadProgress {
    let totalBytes: Int64
    let downloadedBytes: Int64
    
    var percentage: Double {
        guard totalBytes > 0 else { return 0 }
        return Double(downloadedBytes) / Double(totalBytes) * 100
    }
}

class ProgressiveDownloader: NSObject {
    private var continuation: AsyncStream<DownloadProgress>.Continuation?
    private var session: URLSession!
    
    override init() {
        super.init()
        session = URLSession(configuration: .default, delegate: self, delegateQueue: nil)
    }
    
    func download(from url: URL) -> AsyncStream<DownloadProgress> {
        AsyncStream { continuation in
            self.continuation = continuation
            session.downloadTask(with: url).resume()
            
            continuation.onTermination = { _ in
                // cleanup if needed
            }
        }
    }
}

extension ProgressiveDownloader: URLSessionDownloadDelegate {
    func urlSession(
        _ session: URLSession,
        downloadTask: URLSessionDownloadTask,
        didWriteData bytesWritten: Int64,
        totalBytesWritten: Int64,
        totalBytesExpectedToWrite: Int64
    ) {
        let progress = DownloadProgress(
            totalBytes: totalBytesExpectedToWrite,
            downloadedBytes: totalBytesWritten
        )
        continuation?.yield(progress)
    }
    
    func urlSession(
        _ session: URLSession,
        downloadTask: URLSessionDownloadTask,
        didFinishDownloadingTo location: URL
    ) {
        continuation?.finish()
    }
}

// การใช้งาน
func downloadWithProgress() async {
    let downloader = ProgressiveDownloader()
    let url = URL(string: "https://example.com/largefile.zip")!
    
    for await progress in downloader.download(from: url) {
        print(String(format: "Progress: %.1f%%", progress.percentage))
    }
    
    print("Download complete!")
}
```

---

## 17. Best Practices และ Tips

### Tip 1: อย่าบล็อก Main Thread

```swift
// ❌ ไม่ดี
class ViewController: UIViewController {
    func viewDidLoad() {
        super.viewDidLoad()
        // นี่จะบล็อก main thread!
        let data = try! await fetchData()
    }
}

// ✅ ดี
class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        Task {
            do {
                let data = try await fetchData()
                await MainActor.run {
                    updateUI(with: data)
                }
            } catch {
                await MainActor.run {
                    showError(error)
                }
            }
        }
    }
}
```

### Tip 2: ใช้ Structured Concurrency เมื่อเป็นไปได้

```swift
// ✅ Preferred: Structured
func loadAll() async throws -> AppData {
    async let users = fetchUsers()
    async let products = fetchProducts()
    
    return try await AppData(users: users, products: products)
}

// ❌ Less preferred: Unstructured (เมื่อไม่จำเป็น)
func loadAllUnstructured() async throws -> AppData {
    let usersTask = Task { try await fetchUsers() }
    let productsTask = Task { try await fetchProducts() }
    
    let users = try await usersTask.value
    let products = try await productsTask.value
    
    return AppData(users: users, products: products)
}
```

### Tip 3: ระวัง Actor Reentrancy

```swift
actor DataStore {
    var items: [Item] = []
    var isLoading = false
    
    // ⚠️ Potential issue: reentrancy
    func loadItems() async {
        guard !isLoading else { return }
        isLoading = true  // Set flag
        
        // Suspension point! อาจมี tasks อื่นเข้ามาระหว่างนี้
        let newItems = await fetchFromServer()
        
        // หลัง suspension อาจมี task อื่นเปลี่ยน state แล้ว
        items = newItems
        isLoading = false
    }
}
```

### Tip 4: Memory Management

```swift
// ✅ ระวัง retain cycles ใน Task
class DataViewModel {
    var loadTask: Task<Void, Never>?
    
    func load() {
        loadTask = Task { [weak self] in
            guard let self = self else { return }
            let data = await self.fetchData()
            // ใช้ weak reference เพื่อป้องกัน retain cycle
            await MainActor.run { [weak self] in
                self?.updateDisplay(data)
            }
        }
    }
    
    deinit {
        loadTask?.cancel()
    }
}
```

---

## 18. สรุป

Swift Concurrency ให้เราเขียนโค้ดแบบ concurrent ได้อย่างปลอดภัยและอ่านง่าย:

| Feature | ใช้เมื่อ |
|---------|---------|
| `async/await` | ต้องการทำงาน async แบบง่ายๆ |
| `async let` | ต้องการทำงานหลายอย่างพร้อมกัน (รู้จำนวนล่วงหน้า) |
| `TaskGroup` | ต้องการทำงานหลายอย่างพร้อมกัน (ไม่รู้จำนวนล่วงหน้า) |
| `Actor` | ต้องการป้องกัน data races ใน shared mutable state |
| `MainActor` | ต้องการทำงานบน Main Thread |
| `AsyncStream` | ต้องการ stream of values แบบ async |
| `Continuation` | ต้องการแปลง callback API เป็น async |

### Checklist สำหรับ Swift Concurrency

- [ ] ใช้ `async/await` แทน completion handlers
- [ ] ใช้ `async let` หรือ `TaskGroup` สำหรับ parallel work
- [ ] ใช้ `Actor` เพื่อป้องกัน data races
- [ ] ใช้ `@MainActor` สำหรับ UI updates
- [ ] ตรวจสอบ `Task.isCancelled` ใน long-running tasks
- [ ] ใช้ `withTaskCancellationHandler` สำหรับ cleanup
- [ ] ประกาศ types เป็น `Sendable` เมื่อเหมาะสม
- [ ] ใช้ Structured Concurrency เมื่อเป็นไปได้

---

## แบบฝึกหัดเพิ่มเติม

1. สร้าง `AsyncImageGallery` ที่โหลดรูปภาพ 20 รูปพร้อมกันและแสดงผลเมื่อแต่ละรูปโหลดเสร็จ
2. แปลง `CLLocationManager` ที่ใช้ delegate เป็น `AsyncSequence` ที่ emit location updates
3. สร้าง `RateLimiter` actor ที่จำกัดจำนวน API calls ต่อวินาที
4. สร้าง `CacheActor` ที่มี LRU eviction policy สำหรับ cached responses
5. สร้าง streaming text parser ที่ใช้ `AsyncThrowingStream` เพื่อประมวลผลข้อมูลทีละ chunk

---

*จบ Part 29: Swift Concurrency*
