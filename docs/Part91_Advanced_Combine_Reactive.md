# Part 91: Advanced Combine Framework และ Reactive Programming Patterns

## บทนำ

ในบทนี้เราจะเจาะลึก Combine framework อย่างครอบคลุม ตั้งแต่สถาปัตยกรรมพื้นฐานไปจนถึง patterns ขั้นสูงที่ใช้ในการพัฒนาแอปพลิเคชัน iOS จริงๆ Combine เป็น reactive programming framework ของ Apple ที่ช่วยให้เราจัดการกับ asynchronous events ได้อย่างสง่างามและมีประสิทธิภาพ

---

## 1. Combine Architecture Review

### 1.1 Core Concepts: Publisher, Subscriber, Subscription, Demand

Combine framework ประกอบด้วย 4 components หลัก ที่ทำงานร่วมกัน:

```swift
// Publisher - แหล่งที่มาของข้อมูล
// Subscriber - ผู้รับข้อมูล
// Subscription - การเชื่อมต่อระหว่าง Publisher และ Subscriber
// Demand - จำนวนค่าที่ Subscriber ต้องการรับ

import Combine
import Foundation

// ตัวอย่าง Publisher พื้นฐาน
let publisher = [1, 2, 3, 4, 5].publisher

// Subscriber รับค่าผ่าน sink
let cancellable = publisher.sink { completion in
    switch completion {
    case .finished:
        print("เสร็จสิ้น")
    case .failure(let error):
        print("เกิดข้อผิดพลาด: \(error)")
    }
} receiveValue: { value in
    print("ได้รับค่า: \(value)")
}
```

### 1.2 Protocol Definitions

```swift
// Publisher Protocol
public protocol Publisher {
    associatedtype Output     // ชนิดของค่าที่ส่งออก
    associatedtype Failure: Error  // ชนิดของ error
    
    func receive<S>(subscriber: S) where S : Subscriber,
        Self.Failure == S.Failure,
        Self.Output == S.Input
}

// Subscriber Protocol  
public protocol Subscriber: CustomCombineIdentifierConvertible {
    associatedtype Input
    associatedtype Failure: Error
    
    // ถูกเรียกเมื่อ Subscription เริ่มต้น
    func receive(subscription: Subscription)
    
    // ถูกเรียกเมื่อได้รับค่าใหม่
    func receive(_ input: Self.Input) -> Subscribers.Demand
    
    // ถูกเรียกเมื่อ Publisher เสร็จสิ้นหรือเกิดข้อผิดพลาด
    func receive(completion: Subscribers.Completion<Self.Failure>)
}

// Subscription Protocol
public protocol Subscription: Cancellable, CustomCombineIdentifierConvertible {
    func request(_ demand: Subscribers.Demand)
}
```

### 1.3 Demand และ Backpressure

```swift
// Demand คือการควบคุมจำนวนค่าที่ Subscriber ต้องการ
// นี่คือกลไก backpressure ของ Combine

// ประเภทของ Demand
Subscribers.Demand.unlimited  // ต้องการทุกค่า
Subscribers.Demand.none       // ไม่ต้องการค่าตอนนี้
Subscribers.Demand.max(5)     // ต้องการสูงสุด 5 ค่า

// ตัวอย่าง Custom Subscriber ที่ใช้ backpressure
class ThrottledSubscriber: Subscriber {
    typealias Input = Int
    typealias Failure = Never
    
    private var subscription: Subscription?
    private var count = 0
    private let maxBatch = 3
    
    func receive(subscription: Subscription) {
        self.subscription = subscription
        // ขอค่าแรก batch
        subscription.request(.max(maxBatch))
        print("เริ่มต้น Subscription, ขอค่า \(maxBatch) ค่าแรก")
    }
    
    func receive(_ input: Int) -> Subscribers.Demand {
        count += 1
        print("ได้รับค่า #\(count): \(input)")
        
        // หลังจากได้รับครบ batch ขอเพิ่มอีก
        if count % maxBatch == 0 {
            print("ขอค่าเพิ่มอีก \(maxBatch) ค่า")
            return .max(maxBatch)
        }
        return .none
    }
    
    func receive(completion: Subscribers.Completion<Never>) {
        print("เสร็จสิ้น: \(completion)")
    }
}

// การใช้งาน
let numbers = (1...20).publisher
let throttledSub = ThrottledSubscriber()
numbers.subscribe(throttledSub)
```

### 1.4 Custom Publisher Implementation

```swift
// สร้าง Custom Publisher ที่ emit ค่าตาม interval
struct TimerPublisher: Publisher {
    typealias Output = Date
    typealias Failure = Never
    
    let interval: TimeInterval
    let runLoop: RunLoop
    
    init(interval: TimeInterval, runLoop: RunLoop = .main) {
        self.interval = interval
        self.runLoop = runLoop
    }
    
    func receive<S>(subscriber: S) where S : Subscriber,
        Never == S.Failure,
        Date == S.Input {
        
        let subscription = TimerSubscription(
            subscriber: subscriber,
            interval: interval,
            runLoop: runLoop
        )
        subscriber.receive(subscription: subscription)
    }
}

// Subscription class สำหรับ TimerPublisher
final class TimerSubscription<S: Subscriber>: Subscription where S.Input == Date, S.Failure == Never {
    
    private var subscriber: S?
    private var timer: Timer?
    private var demand: Subscribers.Demand = .none
    
    init(subscriber: S, interval: TimeInterval, runLoop: RunLoop) {
        self.subscriber = subscriber
        
        timer = Timer.scheduledTimer(withTimeInterval: interval, repeats: true) { [weak self] _ in
            self?.sendValue()
        }
        runLoop.add(timer!, forMode: .common)
    }
    
    func request(_ demand: Subscribers.Demand) {
        self.demand += demand
    }
    
    private func sendValue() {
        guard demand > 0 else { return }
        demand -= 1
        let newDemand = subscriber?.receive(Date()) ?? .none
        demand += newDemand
    }
    
    func cancel() {
        timer?.invalidate()
        timer = nil
        subscriber = nil
    }
}

// การใช้งาน Custom Publisher
let customTimer = TimerPublisher(interval: 1.0)
let cancellable = customTimer
    .prefix(5)  // รับแค่ 5 ค่า
    .sink { date in
        print("เวลา: \(date)")
    }
```

### 1.5 Custom Subscriber Implementation

```swift
// Custom Subscriber ที่ collect ค่าและ process เป็น batch
class BatchProcessor<T>: Subscriber {
    typealias Input = T
    typealias Failure = Never
    
    private let batchSize: Int
    private var buffer: [T] = []
    private var subscription: Subscription?
    private let processBlock: ([T]) -> Void
    
    init(batchSize: Int, process: @escaping ([T]) -> Void) {
        self.batchSize = batchSize
        self.processBlock = process
    }
    
    func receive(subscription: Subscription) {
        self.subscription = subscription
        subscription.request(.max(batchSize))
    }
    
    func receive(_ input: T) -> Subscribers.Demand {
        buffer.append(input)
        
        if buffer.count >= batchSize {
            let batch = buffer
            buffer.removeAll()
            processBlock(batch)
            return .max(batchSize)  // ขอ batch ถัดไป
        }
        return .none
    }
    
    func receive(completion: Subscribers.Completion<Never>) {
        if !buffer.isEmpty {
            processBlock(buffer)  // process ค่าที่เหลือ
            buffer.removeAll()
        }
        print("BatchProcessor เสร็จสิ้น")
    }
}

// การใช้งาน
let batchProcessor = BatchProcessor<Int>(batchSize: 5) { batch in
    print("Processing batch: \(batch)")
    let sum = batch.reduce(0, +)
    print("ผลรวม: \(sum)")
}

(1...17).publisher.subscribe(batchProcessor)
```

---

## 2. Advanced Combine Operators

### 2.1 flatMap vs switchToLatest

```swift
import Combine
import Foundation

// flatMap - subscribe กับทุก inner publisher พร้อมกัน
// switchToLatest - cancel inner publisher เก่าเมื่อได้รับ publisher ใหม่

// ตัวอย่าง: Search API ที่ debounce

// Simulated network request
func searchAPI(query: String) -> AnyPublisher<[String], Error> {
    let results = ["iPhone", "iPad", "Mac", "Apple Watch"]
        .filter { $0.lowercased().contains(query.lowercased()) }
    
    return Just(results)
        .delay(for: .milliseconds(200), scheduler: DispatchQueue.global())
        .setFailureType(to: Error.self)
        .eraseToAnyPublisher()
}

// flatMap - ทุก request ทำงานพร้อมกัน (อาจได้ response ไม่เป็นลำดับ)
let searchTerms = PassthroughSubject<String, Never>()

let flatMapResults = searchTerms
    .flatMap { term in
        searchAPI(query: term)
            .catch { _ in Just([]) }
    }
    .sink { results in
        print("flatMap results: \(results)")
    }

// switchToLatest - cancel request เก่าเมื่อมี request ใหม่ (เหมาะกับ search)
let switchResults = searchTerms
    .map { term in
        searchAPI(query: term)
            .catch { _ in Just([]) }
    }
    .switchToLatest()
    .sink { results in
        print("switchToLatest results: \(results)")
    }

// Demonstration
searchTerms.send("i")
searchTerms.send("ip")
searchTerms.send("iph")
searchTerms.send("ipho")
searchTerms.send("iphon")
searchTerms.send("iphone")
```

### 2.2 merge, combineLatest, zip ความแตกต่าง

```swift
// merge - รวม Publishers ที่มี Output ชนิดเดียวกัน emit ทุกค่าจากทั้งสอง
// combineLatest - emit เมื่อใด publishers อัปเดต ใช้ค่าล่าสุดของแต่ละ publisher
// zip - จับคู่ค่าตำแหน่งเดียวกันจากแต่ละ publisher

let publisher1 = PassthroughSubject<Int, Never>()
let publisher2 = PassthroughSubject<Int, Never>()

// merge
var cancellables = Set<AnyCancellable>()

Publishers.Merge(publisher1, publisher2)
    .sink { value in
        print("merge ได้รับ: \(value)")
    }
    .store(in: &cancellables)

// combineLatest
Publishers.CombineLatest(publisher1, publisher2)
    .sink { v1, v2 in
        print("combineLatest: (\(v1), \(v2))")
    }
    .store(in: &cancellables)

// zip
Publishers.Zip(publisher1, publisher2)
    .sink { v1, v2 in
        print("zip: (\(v1), \(v2))")
    }
    .store(in: &cancellables)

// ส่งค่าทดสอบ
print("--- ส่ง p1=1 ---")
publisher1.send(1)  // merge: 1, combineLatest: ยังไม่ emit (p2 ยังไม่มีค่า), zip: ยังไม่ emit

print("--- ส่ง p2=10 ---")
publisher2.send(10)  // merge: 10, combineLatest: (1,10), zip: (1,10)

print("--- ส่ง p1=2 ---")
publisher1.send(2)  // merge: 2, combineLatest: (2,10), zip: รอ p2 ค่าที่ 2

print("--- ส่ง p1=3 ---")
publisher1.send(3)  // merge: 3, combineLatest: (3,10), zip: รอ p2 ค่าที่ 2

print("--- ส่ง p2=20 ---")
publisher2.send(20)  // merge: 20, combineLatest: (3,20), zip: (2,20) - จับคู่ p1=2 กับ p2=20
```

### 2.3 scan, reduce, collect

```swift
// scan - เหมือน reduce แต่ emit ทุก intermediate result
// reduce - รวมค่าทั้งหมดและ emit ครั้งเดียวเมื่อ complete
// collect - รวบรวมค่าเป็น array แล้ว emit

let numbers = [1, 2, 3, 4, 5].publisher

// scan - running total
let scanResult = numbers
    .scan(0) { accumulator, value in
        accumulator + value
    }
    .sink { print("scan: \($0)") }
// Output: 1, 3, 6, 10, 15

// reduce - final sum only
let reduceResult = numbers
    .reduce(0, +)
    .sink { print("reduce: \($0)") }
// Output: 15

// collect - รวบรวมทั้งหมด
let collectAll = numbers
    .collect()
    .sink { print("collect all: \($0)") }
// Output: [1, 2, 3, 4, 5]

// collect(n) - รวบรวมทีละ n ค่า
let collectN = (1...10).publisher
    .collect(3)
    .sink { print("collect(3): \($0)") }
// Output: [1,2,3], [4,5,6], [7,8,9], [10]

// ตัวอย่างจริง: Running average
struct MetricCollector {
    static func createRunningAverage(from publisher: AnyPublisher<Double, Never>) -> AnyPublisher<Double, Never> {
        return publisher
            .scan((sum: 0.0, count: 0)) { state, value in
                (sum: state.sum + value, count: state.count + 1)
            }
            .map { state in
                state.sum / Double(state.count)
            }
            .eraseToAnyPublisher()
    }
}
```

### 2.4 debounce vs throttle

```swift
// debounce - รอ interval หลังจากค่าสุดท้าย แล้วจึง emit
// throttle - emit ค่าแรก (หรือล่าสุด) ของแต่ละ interval
// throttle(latest: true) - emit ค่าล่าสุดของแต่ละ interval

import Combine
import Foundation

let keystrokes = PassthroughSubject<String, Never>()

// debounce - เหมาะสำหรับ search field (รอให้ user หยุดพิมพ์)
let debouncedSearch = keystrokes
    .debounce(for: .milliseconds(300), scheduler: DispatchQueue.main)
    .sink { text in
        print("debounce - ค้นหา: \(text)")
    }

// throttle(latest: false) - emit ค่าแรกของแต่ละ window
let throttleFirst = keystrokes
    .throttle(for: .milliseconds(500), scheduler: DispatchQueue.main, latest: false)
    .sink { text in
        print("throttle(latest:false) - ส่ง: \(text)")
    }

// throttle(latest: true) - emit ค่าล่าสุดของแต่ละ window
let throttleLatest = keystrokes
    .throttle(for: .milliseconds(500), scheduler: DispatchQueue.main, latest: true)
    .sink { text in
        print("throttle(latest:true) - ส่ง: \(text)")
    }

// ตัวอย่างเพิ่มเติม: Rate limiting API calls
class APIRateLimiter {
    private var cancellables = Set<AnyCancellable>()
    
    func setupRateLimitedRequests(
        requestPublisher: AnyPublisher<String, Never>
    ) -> AnyPublisher<Data, Error> {
        return requestPublisher
            .throttle(for: .seconds(1), scheduler: DispatchQueue.global(), latest: true)
            .flatMap { endpoint in
                URLSession.shared.dataTaskPublisher(for: URL(string: endpoint)!)
                    .map(\.data)
                    .mapError { $0 as Error }
            }
            .eraseToAnyPublisher()
    }
}
```

### 2.5 retry with exponential backoff

```swift
// retry - ลองใหม่เมื่อเกิด error
// ตัวอย่าง: retry with exponential backoff

import Combine
import Foundation

enum NetworkError: Error {
    case requestFailed
    case serverError(Int)
    case timeout
}

// Simple retry
func fetchData(url: URL) -> AnyPublisher<Data, Error> {
    URLSession.shared.dataTaskPublisher(for: url)
        .map(\.data)
        .mapError { $0 as Error }
        .retry(3)  // ลองใหม่ 3 ครั้ง
        .eraseToAnyPublisher()
}

// Exponential backoff implementation
extension Publisher {
    func retryWithExponentialBackoff(
        maxRetries: Int,
        initialDelay: TimeInterval = 1.0,
        maxDelay: TimeInterval = 60.0,
        multiplier: Double = 2.0,
        scheduler: some Scheduler = DispatchQueue.global()
    ) -> AnyPublisher<Output, Failure> {
        
        return self.catch { error -> AnyPublisher<Output, Failure> in
            guard maxRetries > 0 else {
                return Fail(error: error).eraseToAnyPublisher()
            }
            
            let delay = min(initialDelay * pow(multiplier, Double(1)), maxDelay)
            
            return Just(())
                .delay(for: .seconds(delay), scheduler: scheduler)
                .setFailureType(to: Failure.self)
                .flatMap { _ in
                    self.retryWithExponentialBackoff(
                        maxRetries: maxRetries - 1,
                        initialDelay: delay,
                        maxDelay: maxDelay,
                        multiplier: multiplier,
                        scheduler: scheduler
                    )
                }
                .eraseToAnyPublisher()
        }
        .eraseToAnyPublisher()
    }
}

// การใช้งาน
func unreliableAPI() -> AnyPublisher<String, Error> {
    let shouldFail = Bool.random()
    if shouldFail {
        return Fail(error: NetworkError.requestFailed).eraseToAnyPublisher()
    }
    return Just("Success!").setFailureType(to: Error.self).eraseToAnyPublisher()
}

var cancellables = Set<AnyCancellable>()

unreliableAPI()
    .retryWithExponentialBackoff(maxRetries: 5, initialDelay: 0.5)
    .sink(
        receiveCompletion: { print("Completion: \($0)") },
        receiveValue: { print("Value: \($0)") }
    )
    .store(in: &cancellables)
```

### 2.6 catch และ mapError

```swift
// catch - จัดการ error และแทนที่ด้วย publisher อื่น
// mapError - แปลง error type

enum AppError: Error {
    case networkUnavailable
    case serverError(code: Int, message: String)
    case decodingFailed(Error)
    case unknown
}

enum APIError: Error {
    case statusCode(Int)
    case noData
    case invalidURL
}

// mapError - แปลง error type
func fetchUserProfile(id: String) -> AnyPublisher<UserProfile, AppError> {
    let url = URL(string: "https://api.example.com/users/\(id)")!
    
    return URLSession.shared.dataTaskPublisher(for: url)
        .tryMap { data, response in
            guard let httpResponse = response as? HTTPURLResponse else {
                throw APIError.noData
            }
            guard (200...299).contains(httpResponse.statusCode) else {
                throw APIError.statusCode(httpResponse.statusCode)
            }
            return data
        }
        .decode(type: UserProfile.self, decoder: JSONDecoder())
        .mapError { error -> AppError in
            switch error {
            case let apiError as APIError:
                switch apiError {
                case .statusCode(let code):
                    return .serverError(code: code, message: "HTTP Error")
                case .noData, .invalidURL:
                    return .networkUnavailable
                }
            case is DecodingError:
                return .decodingFailed(error)
            default:
                return .unknown
            }
        }
        .eraseToAnyPublisher()
}

// ตัวอย่าง struct (สมมติ)
struct UserProfile: Codable {
    let id: String
    let name: String
    let email: String
}

// catch - แทนที่ error ด้วย fallback publisher
func fetchWithFallback(id: String) -> AnyPublisher<UserProfile, Never> {
    let cached = UserProfile(id: id, name: "Cached User", email: "cached@example.com")
    
    return fetchUserProfile(id: id)
        .catch { error -> AnyPublisher<UserProfile, AppError> in
            print("เกิดข้อผิดพลาด: \(error), ใช้ cached data")
            return Just(cached)
                .setFailureType(to: AppError.self)
                .eraseToAnyPublisher()
        }
        .replaceError(with: cached)
        .eraseToAnyPublisher()
}

// replaceError vs catch
// replaceError - แทนที่ error ด้วยค่าเดียว
// catch - แทนที่ด้วย publisher (ยืดหยุ่นกว่า)
```

### 2.7 handleEvents สำหรับ debugging

```swift
// handleEvents - ดูเหตุการณ์ต่างๆ ใน pipeline โดยไม่แก้ไขค่า
// เหมาะสำหรับ debugging

let pipeline = (1...5).publisher
    .handleEvents(
        receiveSubscription: { subscription in
            print("🔵 ได้รับ Subscription: \(subscription)")
        },
        receiveOutput: { value in
            print("📤 Output: \(value)")
        },
        receiveCompletion: { completion in
            print("✅ Completion: \(completion)")
        },
        receiveCancel: {
            print("❌ Cancelled")
        },
        receiveRequest: { demand in
            print("📥 Request demand: \(demand)")
        }
    )
    .map { $0 * 2 }
    .sink { print("Final: \($0)") }

// Extension สำหรับ debug logging ที่สะดวกกว่า
extension Publisher {
    func debugLog(_ prefix: String = "") -> AnyPublisher<Output, Failure> {
        return self.handleEvents(
            receiveOutput: { value in
                print("[\(prefix)] Output: \(value)")
            },
            receiveCompletion: { completion in
                print("[\(prefix)] Completion: \(completion)")
            },
            receiveCancel: {
                print("[\(prefix)] Cancelled")
            }
        )
        .eraseToAnyPublisher()
    }
}

// การใช้งาน debug extension
let debugPipeline = (1...3).publisher
    .debugLog("Step 1")
    .map { $0 * 10 }
    .debugLog("Step 2")
    .filter { $0 > 15 }
    .debugLog("Step 3")
    .sink { print("Result: \($0)") }
```

---

## 3. Custom Publishers

### 3.1 Timer.publish

```swift
import Combine
import Foundation

// Timer.publish - สร้าง timer publisher
// autoconnect() - เริ่ม timer อัตโนมัติ

var cancellables = Set<AnyCancellable>()

// Timer ที่ fire ทุก 1 วินาที
Timer.publish(every: 1.0, on: .main, in: .common)
    .autoconnect()
    .sink { date in
        print("Timer fired at: \(date)")
    }
    .store(in: &cancellables)

// Timer ที่ fire ทุก 0.5 วินาที และหยุดหลัง 5 ครั้ง
Timer.publish(every: 0.5, on: .main, in: .common)
    .autoconnect()
    .prefix(5)
    .sink(
        receiveCompletion: { _ in print("Timer หยุดแล้ว") },
        receiveValue: { date in print("Tick: \(date)") }
    )
    .store(in: &cancellables)

// ความแตกต่างระหว่าง Timer.publish กับ Timer.scheduledTimer
// Timer.publish - เป็น Combine Publisher, จัดการ cancellation อัตโนมัติ
// Timer.scheduledTimer - traditional Timer API ต้อง invalidate เอง

// ConnectablePublisher - ควบคุมการเริ่มต้น timer เอง
let timerPublisher = Timer.publish(every: 1.0, on: .main, in: .common)
let connection = timerPublisher.connect()  // เริ่ม timer

// ต้องการหยุดเอง
DispatchQueue.main.asyncAfter(deadline: .now() + 5) {
    connection.cancel()
}
```

### 3.2 NotificationCenter.Publisher

```swift
// NotificationCenter.Publisher - subscribe notification ผ่าน Combine

import Combine
import Foundation

var cancellables = Set<AnyCancellable>()

// Subscribe notification เมื่อ app เข้าสู่ background
NotificationCenter.default.publisher(for: UIApplication.didEnterBackgroundNotification)
    .sink { notification in
        print("App เข้า background")
    }
    .store(in: &cancellables)

// Subscribe keyboard notifications
NotificationCenter.default.publisher(for: UIResponder.keyboardWillShowNotification)
    .compactMap { notification -> CGFloat? in
        guard let keyboardFrame = notification.userInfo?[UIResponder.keyboardFrameEndUserInfoKey] as? CGRect else {
            return nil
        }
        return keyboardFrame.height
    }
    .sink { keyboardHeight in
        print("Keyboard height: \(keyboardHeight)")
    }
    .store(in: &cancellables)

// Custom notification
extension Notification.Name {
    static let userDidLogin = Notification.Name("userDidLogin")
    static let dataUpdated = Notification.Name("dataUpdated")
}

struct UserInfo {
    let userId: String
    let username: String
}

// Post notification พร้อม userInfo
func postLoginNotification(userInfo: UserInfo) {
    NotificationCenter.default.post(
        name: .userDidLogin,
        object: nil,
        userInfo: ["userId": userInfo.userId, "username": userInfo.username]
    )
}

// Subscribe และ decode userInfo
NotificationCenter.default.publisher(for: .userDidLogin)
    .compactMap { notification -> UserInfo? in
        guard
            let userId = notification.userInfo?["userId"] as? String,
            let username = notification.userInfo?["username"] as? String
        else { return nil }
        return UserInfo(userId: userId, username: username)
    }
    .sink { userInfo in
        print("User logged in: \(userInfo.username)")
    }
    .store(in: &cancellables)
```

### 3.3 URLSession Publishers

```swift
import Combine
import Foundation

// URLSession dataTaskPublisher
struct APIClient {
    let baseURL: URL
    let session: URLSession
    
    init(baseURL: URL, session: URLSession = .shared) {
        self.baseURL = baseURL
        self.session = session
    }
    
    // Generic fetch method
    func fetch<T: Decodable>(
        endpoint: String,
        responseType: T.Type,
        decoder: JSONDecoder = JSONDecoder()
    ) -> AnyPublisher<T, Error> {
        let url = baseURL.appendingPathComponent(endpoint)
        
        return session.dataTaskPublisher(for: url)
            .tryMap { data, response in
                guard let httpResponse = response as? HTTPURLResponse else {
                    throw URLError(.badServerResponse)
                }
                guard (200...299).contains(httpResponse.statusCode) else {
                    throw URLError(.badServerResponse)
                }
                return data
            }
            .decode(type: T.self, decoder: decoder)
            .receive(on: DispatchQueue.main)
            .eraseToAnyPublisher()
    }
    
    // POST request
    func post<T: Encodable, R: Decodable>(
        endpoint: String,
        body: T,
        responseType: R.Type
    ) -> AnyPublisher<R, Error> {
        let url = baseURL.appendingPathComponent(endpoint)
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        
        do {
            request.httpBody = try JSONEncoder().encode(body)
        } catch {
            return Fail(error: error).eraseToAnyPublisher()
        }
        
        return session.dataTaskPublisher(for: request)
            .tryMap { data, response in
                guard let httpResponse = response as? HTTPURLResponse,
                      (200...299).contains(httpResponse.statusCode) else {
                    throw URLError(.badServerResponse)
                }
                return data
            }
            .decode(type: R.self, decoder: JSONDecoder())
            .receive(on: DispatchQueue.main)
            .eraseToAnyPublisher()
    }
    
    // Download with progress
    func download(url: URL) -> AnyPublisher<Double, Error> {
        let subject = PassthroughSubject<Double, Error>()
        
        let task = session.downloadTask(with: url) { location, response, error in
            if let error = error {
                subject.send(completion: .failure(error))
                return
            }
            subject.send(1.0)
            subject.send(completion: .finished)
        }
        
        // ติดตาม progress (ต้องใช้ URLSession delegate จริงๆ)
        task.resume()
        
        return subject.eraseToAnyPublisher()
    }
}

// Models
struct Post: Codable {
    let id: Int
    let title: String
    let body: String
    let userId: Int
}

// การใช้งาน
let apiClient = APIClient(baseURL: URL(string: "https://jsonplaceholder.typicode.com")!)
var cancellables = Set<AnyCancellable>()

apiClient.fetch(endpoint: "posts/1", responseType: Post.self)
    .sink(
        receiveCompletion: { completion in
            if case .failure(let error) = completion {
                print("Error: \(error)")
            }
        },
        receiveValue: { post in
            print("Post: \(post.title)")
        }
    )
    .store(in: &cancellables)
```

### 3.4 Building a Custom Publisher from Scratch

```swift
// สร้าง Fibonacci Publisher ที่ emit เลข Fibonacci
struct FibonacciPublisher: Publisher {
    typealias Output = Int
    typealias Failure = Never
    
    let count: Int
    
    func receive<S>(subscriber: S) where S : Subscriber, Never == S.Failure, Int == S.Input {
        let subscription = FibonacciSubscription(subscriber: subscriber, count: count)
        subscriber.receive(subscription: subscription)
    }
}

final class FibonacciSubscription<S: Subscriber>: Subscription
    where S.Input == Int, S.Failure == Never {
    
    private var subscriber: S?
    private var demand: Subscribers.Demand = .none
    private var count: Int
    private var current = 0
    private var next = 1
    
    init(subscriber: S, count: Int) {
        self.subscriber = subscriber
        self.count = count
    }
    
    func request(_ demand: Subscribers.Demand) {
        self.demand += demand
        fulfill()
    }
    
    private func fulfill() {
        while demand > 0, count > 0 {
            let value = current
            let newNext = current + next
            current = next
            next = newNext
            count -= 1
            
            let additionalDemand = subscriber?.receive(value) ?? .none
            demand -= 1
            demand += additionalDemand
        }
        
        if count == 0 {
            subscriber?.receive(completion: .finished)
            subscriber = nil
        }
    }
    
    func cancel() {
        subscriber = nil
    }
}

// สร้าง convenience initializer ผ่าน extension
extension Publishers {
    static func fibonacci(count: Int) -> FibonacciPublisher {
        return FibonacciPublisher(count: count)
    }
}

// การใช้งาน
let fibonacci = Publishers.fibonacci(count: 10)
fibonacci
    .sink { value in
        print("Fibonacci: \(value)")
    }
```

---

## 4. Subjects

### 4.1 PassthroughSubject vs CurrentValueSubject

```swift
import Combine

// PassthroughSubject - ไม่เก็บค่า, subscriber จะรับเฉพาะค่าที่ส่งหลังจาก subscribe
// CurrentValueSubject - เก็บค่าปัจจุบัน, subscriber จะได้รับค่าปัจจุบันทันทีเมื่อ subscribe

// PassthroughSubject
let passthrough = PassthroughSubject<String, Never>()

passthrough.send("ค่าก่อน subscribe")  // ไม่มี subscriber รับ

let sub1 = passthrough.sink { print("Sub1: \($0)") }

passthrough.send("หลัง subscribe")  // Sub1 ได้รับ

// CurrentValueSubject
let current = CurrentValueSubject<Int, Never>(0)  // ค่าเริ่มต้น = 0

print("ค่าปัจจุบัน: \(current.value)")  // สามารถอ่านค่าได้โดยตรง

current.send(1)
current.send(2)

let sub2 = current.sink { print("Sub2: \($0)") }  // จะได้รับ 2 ทันที (ค่าปัจจุบัน)

current.send(3)  // Sub2 ได้รับ 3

// Use cases
// PassthroughSubject: user actions, events ที่เกิดขึ้นครั้งเดียว
// CurrentValueSubject: state ที่ต้องการรู้ค่าปัจจุบัน

class UserSessionManager {
    // PassthroughSubject สำหรับ events
    let loginEvents = PassthroughSubject<String, Never>()
    
    // CurrentValueSubject สำหรับ state
    let isLoggedIn = CurrentValueSubject<Bool, Never>(false)
    let currentUser = CurrentValueSubject<User?, Never>(nil)
    
    func login(user: User) {
        currentUser.send(user)
        isLoggedIn.send(true)
        loginEvents.send("User \(user.name) logged in")
    }
    
    func logout() {
        currentUser.send(nil)
        isLoggedIn.send(false)
        loginEvents.send("User logged out")
    }
}

struct User {
    let id: String
    let name: String
    let email: String
}
```

### 4.2 Subject Threading

```swift
import Combine
import Foundation

// Subjects ไม่ thread-safe โดย default
// ต้องระวังการส่งค่าจาก multiple threads

class ThreadSafeSubject<Output, Failure: Error> {
    private let subject: PassthroughSubject<Output, Failure>
    private let queue: DispatchQueue
    
    var publisher: AnyPublisher<Output, Failure> {
        subject.eraseToAnyPublisher()
    }
    
    init(label: String = "com.app.threadSafeSubject") {
        subject = PassthroughSubject()
        queue = DispatchQueue(label: label, attributes: .concurrent)
    }
    
    func send(_ value: Output) {
        queue.async(flags: .barrier) {
            self.subject.send(value)
        }
    }
    
    func send(completion: Subscribers.Completion<Failure>) {
        queue.async(flags: .barrier) {
            self.subject.send(completion: completion)
        }
    }
}

// ตัวอย่างการใช้งาน thread-safe subject
let safeSubject = ThreadSafeSubject<Int, Never>()
var cancellables = Set<AnyCancellable>()

safeSubject.publisher
    .receive(on: DispatchQueue.main)
    .sink { value in
        print("ได้รับ: \(value) บน main thread")
    }
    .store(in: &cancellables)

// ส่งจาก multiple threads
DispatchQueue.concurrentPerform(iterations: 100) { i in
    safeSubject.send(i)
}
```

### 4.3 multicast และ share

```swift
import Combine

// share() - แชร์ single subscription กับหลาย subscriber
// multicast - ควบคุมการ connect เอง

// ปัญหา: หาก publisher มี side effects และมีหลาย subscriber
// แต่ละ subscriber จะทำให้ publisher ทำงานใหม่

var requestCount = 0

let expensivePublisher = Deferred {
    Future<String, Never> { promise in
        requestCount += 1
        print("Making request #\(requestCount)")
        promise(.success("Data"))
    }
}

var cancellables = Set<AnyCancellable>()

// ไม่ดี: สองครั้ง request
expensivePublisher.sink { print("Sub1: \($0)") }.store(in: &cancellables)
expensivePublisher.sink { print("Sub2: \($0)") }.store(in: &cancellables)
// "Making request #1"
// "Making request #2"

// share() - แชร์ single upstream subscription
let sharedPublisher = expensivePublisher.share()

sharedPublisher.sink { print("Sub3: \($0)") }.store(in: &cancellables)
sharedPublisher.sink { print("Sub4: \($0)") }.store(in: &cancellables)
// "Making request #3" (แค่ครั้งเดียว!)

// multicast - ควบคุมการ connect เอง
let multicastPublisher = expensivePublisher
    .multicast { PassthroughSubject<String, Never>() }

let sub5 = multicastPublisher.sink { print("Sub5: \($0)") }
let sub6 = multicastPublisher.sink { print("Sub6: \($0)") }

// connect เมื่อ subscribers พร้อมแล้ว
let connection = multicastPublisher.connect()
// "Making request #..." (ครั้งเดียว ทั้ง Sub5 และ Sub6 รับ)

// หยุดการทำงาน
connection.cancel()
```

---

## 5. Combine + SwiftUI

### 5.1 onReceive modifier

```swift
import SwiftUI
import Combine

struct ClockView: View {
    @State private var currentTime = Date()
    
    let timer = Timer.publish(every: 1, on: .main, in: .common).autoconnect()
    
    var body: some View {
        VStack {
            Text("เวลาปัจจุบัน")
                .font(.headline)
            
            Text(currentTime, style: .time)
                .font(.largeTitle)
                .monospacedDigit()
        }
        .onReceive(timer) { time in
            currentTime = time
        }
    }
}

// onReceive กับ NotificationCenter
struct AppStateView: View {
    @State private var isInBackground = false
    
    var body: some View {
        Text(isInBackground ? "Background" : "Foreground")
            .onReceive(
                NotificationCenter.default.publisher(for: UIApplication.didEnterBackgroundNotification)
            ) { _ in
                isInBackground = true
            }
            .onReceive(
                NotificationCenter.default.publisher(for: UIApplication.willEnterForegroundNotification)
            ) { _ in
                isInBackground = false
            }
    }
}
```

### 5.2 @ObservableObject กับ Combine

```swift
import SwiftUI
import Combine

class SearchViewModel: ObservableObject {
    @Published var searchText = ""
    @Published var results: [String] = []
    @Published var isLoading = false
    @Published var errorMessage: String?
    
    private var cancellables = Set<AnyCancellable>()
    
    private let searchService: SearchService
    
    init(searchService: SearchService = SearchService()) {
        self.searchService = searchService
        setupSearchPipeline()
    }
    
    private func setupSearchPipeline() {
        $searchText
            .debounce(for: .milliseconds(300), scheduler: DispatchQueue.main)
            .removeDuplicates()
            .filter { !$0.isEmpty }
            .handleEvents(receiveOutput: { [weak self] _ in
                self?.isLoading = true
                self?.errorMessage = nil
            })
            .flatMap { [weak self] text -> AnyPublisher<[String], Never> in
                guard let self = self else { return Just([]).eraseToAnyPublisher() }
                
                return self.searchService.search(query: text)
                    .catch { [weak self] error -> Just<[String]> in
                        self?.errorMessage = error.localizedDescription
                        return Just([])
                    }
                    .eraseToAnyPublisher()
            }
            .handleEvents(receiveOutput: { [weak self] _ in
                self?.isLoading = false
            })
            .assign(to: &$results)
    }
}

class SearchService {
    func search(query: String) -> AnyPublisher<[String], Error> {
        let mockResults = ["Apple", "Banana", "Cherry", "Date", "Elderberry"]
            .filter { $0.lowercased().hasPrefix(query.lowercased()) }
        
        return Just(mockResults)
            .delay(for: .milliseconds(200), scheduler: DispatchQueue.global())
            .setFailureType(to: Error.self)
            .eraseToAnyPublisher()
    }
}

struct SearchView: View {
    @StateObject var viewModel = SearchViewModel()
    
    var body: some View {
        NavigationView {
            VStack {
                SearchBar(text: $viewModel.searchText)
                
                if viewModel.isLoading {
                    ProgressView("กำลังค้นหา...")
                } else if let error = viewModel.errorMessage {
                    Text("Error: \(error)")
                        .foregroundColor(.red)
                } else {
                    List(viewModel.results, id: \.self) { result in
                        Text(result)
                    }
                }
            }
            .navigationTitle("ค้นหา")
        }
    }
}

struct SearchBar: View {
    @Binding var text: String
    
    var body: some View {
        HStack {
            Image(systemName: "magnifyingglass")
            TextField("ค้นหา...", text: $text)
                .textFieldStyle(.roundedBorder)
        }
        .padding()
    }
}
```

### 5.3 assign(to:on:) vs assign(to:)

```swift
import SwiftUI
import Combine

class CounterViewModel: ObservableObject {
    @Published var count = 0
    @Published var doubleCount = 0
    
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        setupBindings()
    }
    
    private func setupBindings() {
        // assign(to:on:) - เก่ากว่า, ต้องจัดการ memory เอง
        // ปัญหา: สร้าง retain cycle ง่าย
        $count
            .map { $0 * 2 }
            .assign(to: \.doubleCount, on: self)  // อาจสร้าง retain cycle!
            .store(in: &cancellables)  // ต้อง store ไม่งั้นถูก cancel ทันที
        
        // assign(to:) - ใหม่กว่า, ไม่สร้าง retain cycle
        // ใช้กับ @Published เท่านั้น
        $count
            .map { $0 * 3 }
            .assign(to: &$doubleCount)  // ปลอดภัยจาก retain cycle
    }
    
    func increment() {
        count += 1
    }
}

// ความแตกต่างหลัก:
// assign(to: \.property, on: object) - คืน AnyCancellable ต้อง store
// assign(to: &$property) - ไม่คืน AnyCancellable, จัดการ lifetime เอง
// assign(to:) เหมาะกว่าสำหรับ @Published properties ใน ObservableObject
```

### 5.4 Cancellable Management

```swift
import SwiftUI
import Combine

// วิธีที่ 1: Store ใน Set<AnyCancellable>
class ViewModelA: ObservableObject {
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        Just("Hello")
            .sink { print($0) }
            .store(in: &cancellables)  // ถูก cancel เมื่อ ViewModel ถูก deallocate
    }
    
    deinit {
        print("ViewModelA ถูก deallocate")
        // cancellables ถูก cancel อัตโนมัติ
    }
}

// วิธีที่ 2: Store ใน Array (สำหรับ sequential management)
class ViewModelB: ObservableObject {
    private var cancellableStorage: [AnyCancellable] = []
    
    func subscribe() {
        let c = Timer.publish(every: 1.0, on: .main, in: .common)
            .autoconnect()
            .sink { _ in print("tick") }
        
        cancellableStorage.append(c)
    }
    
    func unsubscribeAll() {
        cancellableStorage.removeAll()  // cancel ทั้งหมด
    }
}

// วิธีที่ 3: Named cancellables สำหรับ fine-grained control
class ViewModelC: ObservableObject {
    private var searchCancellable: AnyCancellable?
    private var timerCancellable: AnyCancellable?
    
    func startSearch() {
        searchCancellable?.cancel()  // cancel การค้นหาเก่า
        searchCancellable = Just("result")
            .sink { print("search: \($0)") }
    }
    
    func startTimer() {
        timerCancellable = Timer.publish(every: 1.0, on: .main, in: .common)
            .autoconnect()
            .sink { _ in print("timer") }
    }
    
    func stopTimer() {
        timerCancellable = nil  // cancel timer
    }
}

// วิธีที่ 4: Cancellable ใน SwiftUI View
struct TimerView: View {
    @State private var count = 0
    @State private var cancellable: AnyCancellable?
    
    var body: some View {
        Text("Count: \(count)")
            .onAppear {
                cancellable = Timer.publish(every: 1.0, on: .main, in: .common)
                    .autoconnect()
                    .sink { _ in count += 1 }
            }
            .onDisappear {
                cancellable?.cancel()
            }
    }
}
```

---

## 6. Combine + async/await

### 6.1 values property บน Publisher

```swift
import Combine
import Foundation

// Publisher.values - แปลง Publisher เป็น AsyncSequence
// ใช้ได้กับ Swift 5.5+

async func readFromPublisher() async {
    let publisher = [1, 2, 3, 4, 5].publisher
    
    for await value in publisher.values {
        print("async received: \(value)")
    }
    print("เสร็จสิ้น")
}

// ใช้กับ real-world publisher
async func fetchDataAsync() async throws {
    let url = URL(string: "https://api.example.com/data")!
    
    let dataPublisher = URLSession.shared.dataTaskPublisher(for: url)
        .map(\.data)
        .mapError { $0 as Error }
    
    for try await data in dataPublisher.values {
        print("Received \(data.count) bytes")
    }
}
```

### 6.2 Bridging Combine ไปยัง async

```swift
import Combine
import Foundation

// แปลง Combine Publisher เป็น async function ด้วย withCheckedContinuation
extension Publisher where Failure == Error {
    func async() async throws -> Output {
        return try await withCheckedThrowingContinuation { continuation in
            var cancellable: AnyCancellable?
            
            cancellable = self
                .first()
                .sink(
                    receiveCompletion: { completion in
                        switch completion {
                        case .finished:
                            break
                        case .failure(let error):
                            continuation.resume(throwing: error)
                        }
                        cancellable?.cancel()
                    },
                    receiveValue: { value in
                        continuation.resume(returning: value)
                        cancellable?.cancel()
                    }
                )
        }
    }
}

extension Publisher where Failure == Never {
    func async() async -> Output {
        return await withCheckedContinuation { continuation in
            var cancellable: AnyCancellable?
            
            cancellable = self
                .first()
                .sink { value in
                    continuation.resume(returning: value)
                    cancellable?.cancel()
                }
        }
    }
}

// การใช้งาน
async func example() async throws {
    let result = try await URLSession.shared
        .dataTaskPublisher(for: URL(string: "https://api.example.com")!)
        .map(\.data)
        .mapError { $0 as Error }
        .async()
    
    print("Got \(result.count) bytes")
}
```

### 6.3 Bridging async ไปยัง Publisher (AsyncPublisher)

```swift
import Combine
import Foundation

// แปลง async function เป็น Publisher ด้วย Deferred + Future
func asyncToPublisher<T>(operation: @escaping () async throws -> T) -> AnyPublisher<T, Error> {
    return Deferred {
        Future { promise in
            Task {
                do {
                    let result = try await operation()
                    promise(.success(result))
                } catch {
                    promise(.failure(error))
                }
            }
        }
    }
    .eraseToAnyPublisher()
}

// ตัวอย่างการใช้งาน
async func fetchUser(id: String) async throws -> User {
    // Simulate async operation
    try await Task.sleep(nanoseconds: 1_000_000_000)
    return User(id: id, name: "Test User", email: "test@example.com")
}

let userPublisher = asyncToPublisher {
    try await fetchUser(id: "123")
}

var cancellables = Set<AnyCancellable>()

userPublisher
    .sink(
        receiveCompletion: { print("Completion: \($0)") },
        receiveValue: { user in print("User: \(user.name)") }
    )
    .store(in: &cancellables)

// AsyncStream เป็น Publisher
func asyncStreamPublisher<T>(stream: AsyncStream<T>) -> AnyPublisher<T, Never> {
    let subject = PassthroughSubject<T, Never>()
    
    Task {
        for await value in stream {
            subject.send(value)
        }
        subject.send(completion: .finished)
    }
    
    return subject.eraseToAnyPublisher()
}

// ตัวอย่าง: WebSocket events เป็น Publisher
func webSocketPublisher(url: URL) -> AnyPublisher<String, Error> {
    let subject = PassthroughSubject<String, Error>()
    let session = URLSession.shared
    let webSocketTask = session.webSocketTask(with: url)
    
    func receive() {
        webSocketTask.receive { result in
            switch result {
            case .success(let message):
                switch message {
                case .string(let text):
                    subject.send(text)
                    receive()  // continue receiving
                case .data(let data):
                    if let text = String(data: data, encoding: .utf8) {
                        subject.send(text)
                    }
                    receive()
                @unknown default:
                    break
                }
            case .failure(let error):
                subject.send(completion: .failure(error))
            }
        }
    }
    
    webSocketTask.resume()
    receive()
    
    return subject.eraseToAnyPublisher()
}
```

---

## 7. Testing Combine

### 7.1 XCTestExpectation กับ Combine

```swift
import XCTest
import Combine

class CombineTests: XCTestCase {
    var cancellables = Set<AnyCancellable>()
    
    override func tearDown() {
        cancellables.removeAll()
        super.tearDown()
    }
    
    // ทดสอบ Publisher ที่ emit ค่าเดียว
    func testSimplePublisher() {
        let expectation = expectation(description: "Publisher ส่งค่า")
        
        Just(42)
            .sink { value in
                XCTAssertEqual(value, 42)
                expectation.fulfill()
            }
            .store(in: &cancellables)
        
        waitForExpectations(timeout: 1.0)
    }
    
    // ทดสอบ Publisher ที่ emit หลายค่า
    func testMultipleValues() {
        let expectation = expectation(description: "ได้รับค่าทั้งหมด")
        var receivedValues: [Int] = []
        
        [1, 2, 3, 4, 5].publisher
            .sink(
                receiveCompletion: { completion in
                    if case .finished = completion {
                        XCTAssertEqual(receivedValues, [1, 2, 3, 4, 5])
                        expectation.fulfill()
                    }
                },
                receiveValue: { value in
                    receivedValues.append(value)
                }
            )
            .store(in: &cancellables)
        
        waitForExpectations(timeout: 1.0)
    }
    
    // ทดสอบ error handling
    func testPublisherError() {
        let expectation = expectation(description: "ได้รับ error")
        
        enum TestError: Error {
            case mockError
        }
        
        Fail<Int, TestError>(error: .mockError)
            .sink(
                receiveCompletion: { completion in
                    if case .failure(let error) = completion {
                        XCTAssertEqual(error, TestError.mockError)
                        expectation.fulfill()
                    }
                },
                receiveValue: { _ in
                    XCTFail("ไม่ควรได้รับค่า")
                }
            )
            .store(in: &cancellables)
        
        waitForExpectations(timeout: 1.0)
    }
    
    // ทดสอบ async publisher
    func testAsyncPublisher() async {
        let values = await [1, 2, 3].publisher
            .map { $0 * 2 }
            .values
            .reduce(into: []) { $0.append($1) }
        
        XCTAssertEqual(values, [2, 4, 6])
    }
}
```

### 7.2 sink testing patterns

```swift
import XCTest
import Combine

// Helper class สำหรับ testing
class PublisherSpy<Output, Failure: Error> {
    var values: [Output] = []
    var completion: Subscribers.Completion<Failure>?
    var cancellable: AnyCancellable?
    
    func subscribe(to publisher: AnyPublisher<Output, Failure>) {
        cancellable = publisher.sink(
            receiveCompletion: { [weak self] completion in
                self?.completion = completion
            },
            receiveValue: { [weak self] value in
                self?.values.append(value)
            }
        )
    }
    
    var didFinish: Bool {
        if case .finished = completion {
            return true
        }
        return false
    }
    
    var didFail: Bool {
        if case .failure = completion {
            return true
        }
        return false
    }
    
    var failureError: Failure? {
        if case .failure(let error) = completion {
            return error
        }
        return nil
    }
}

// ตัวอย่างการใช้ PublisherSpy
class ViewModelTests: XCTestCase {
    func testSearchViewModel() {
        let viewModel = SearchViewModel()
        let spy = PublisherSpy<[String], Never>()
        spy.subscribe(to: viewModel.$results.eraseToAnyPublisher())
        
        viewModel.searchText = "apple"
        
        // รอให้ debounce ทำงาน
        let expectation = expectation(description: "Search completed")
        DispatchQueue.main.asyncAfter(deadline: .now() + 0.5) {
            expectation.fulfill()
        }
        
        waitForExpectations(timeout: 1.0)
        
        XCTAssertFalse(spy.values.isEmpty)
    }
}

// TestScheduler pattern (ไม่ต้องใช้ library)
class VirtualTimeScheduler: Scheduler {
    typealias SchedulerTimeType = DispatchQueue.SchedulerTimeType
    typealias SchedulerOptions = DispatchQueue.SchedulerOptions
    
    var now: SchedulerTimeType { DispatchQueue.main.now }
    var minimumTolerance: SchedulerTimeType.Stride { .zero }
    
    private var scheduledActions: [(date: SchedulerTimeType, action: () -> Void)] = []
    
    func schedule(options: SchedulerOptions?, _ action: @escaping () -> Void) {
        action()
    }
    
    func schedule(after date: SchedulerTimeType, tolerance: SchedulerTimeType.Stride, options: SchedulerOptions?, _ action: @escaping () -> Void) {
        scheduledActions.append((date: date, action: action))
    }
    
    func schedule(after date: SchedulerTimeType, interval: SchedulerTimeType.Stride, tolerance: SchedulerTimeType.Stride, options: SchedulerOptions?, _ action: @escaping () -> Void) -> Cancellable {
        return AnyCancellable {}
    }
    
    func advance(by seconds: TimeInterval) {
        let deadline = now.advanced(by: .seconds(seconds))
        let actionsToRun = scheduledActions.filter { $0.date <= deadline }
        scheduledActions.removeAll { $0.date <= deadline }
        actionsToRun.sorted { $0.date < $1.date }.forEach { $0.action() }
    }
}
```

---

## 8. Real-world Combine Patterns

### 8.1 Form Validation Pipeline

```swift
import Combine
import Foundation

struct ValidationError: Error, LocalizedError {
    let message: String
    var errorDescription: String? { message }
}

class RegistrationViewModel: ObservableObject {
    @Published var username = ""
    @Published var email = ""
    @Published var password = ""
    @Published var confirmPassword = ""
    
    @Published var usernameError: String?
    @Published var emailError: String?
    @Published var passwordError: String?
    @Published var confirmPasswordError: String?
    @Published var isFormValid = false
    
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        setupValidation()
    }
    
    private func validateUsername(_ username: String) -> String? {
        if username.isEmpty { return "กรุณากรอก username" }
        if username.count < 3 { return "Username ต้องมีอย่างน้อย 3 ตัวอักษร" }
        if username.count > 20 { return "Username ต้องไม่เกิน 20 ตัวอักษร" }
        let validCharacters = CharacterSet.alphanumerics.union(CharacterSet(charactersIn: "_"))
        if username.unicodeScalars.contains(where: { !validCharacters.contains($0) }) {
            return "Username ใช้ได้เฉพาะตัวอักษร ตัวเลข และ _"
        }
        return nil
    }
    
    private func validateEmail(_ email: String) -> String? {
        if email.isEmpty { return "กรุณากรอก email" }
        let emailRegex = #"^[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$"#
        let emailPredicate = NSPredicate(format: "SELF MATCHES %@", emailRegex)
        if !emailPredicate.evaluate(with: email) {
            return "รูปแบบ email ไม่ถูกต้อง"
        }
        return nil
    }
    
    private func validatePassword(_ password: String) -> String? {
        if password.isEmpty { return "กรุณากรอก password" }
        if password.count < 8 { return "Password ต้องมีอย่างน้อย 8 ตัวอักษร" }
        if !password.contains(where: { $0.isUppercase }) {
            return "Password ต้องมีตัวพิมพ์ใหญ่อย่างน้อย 1 ตัว"
        }
        if !password.contains(where: { $0.isNumber }) {
            return "Password ต้องมีตัวเลขอย่างน้อย 1 ตัว"
        }
        return nil
    }
    
    private func setupValidation() {
        $username
            .debounce(for: .milliseconds(500), scheduler: DispatchQueue.main)
            .map { [weak self] username in self?.validateUsername(username) }
            .assign(to: &$usernameError)
        
        $email
            .debounce(for: .milliseconds(500), scheduler: DispatchQueue.main)
            .map { [weak self] email in self?.validateEmail(email) }
            .assign(to: &$emailError)
        
        $password
            .debounce(for: .milliseconds(500), scheduler: DispatchQueue.main)
            .map { [weak self] password in self?.validatePassword(password) }
            .assign(to: &$passwordError)
        
        Publishers.CombineLatest($password, $confirmPassword)
            .debounce(for: .milliseconds(500), scheduler: DispatchQueue.main)
            .map { password, confirm -> String? in
                if confirm.isEmpty { return nil }
                return password == confirm ? nil : "Password ไม่ตรงกัน"
            }
            .assign(to: &$confirmPasswordError)
        
        // ตรวจสอบว่า form ถูกต้องทั้งหมด
        Publishers.CombineLatest4($username, $email, $password, $confirmPassword)
            .map { [weak self] username, email, password, confirm -> Bool in
                guard let self = self else { return false }
                return self.validateUsername(username) == nil &&
                       self.validateEmail(email) == nil &&
                       self.validatePassword(password) == nil &&
                       password == confirm && !confirm.isEmpty
            }
            .assign(to: &$isFormValid)
    }
}
```

### 8.2 Search with Debounce + Networking

```swift
import Combine
import Foundation
import SwiftUI

struct SearchResult: Identifiable, Codable {
    let id: Int
    let title: String
    let description: String
}

class AdvancedSearchViewModel: ObservableObject {
    @Published var query = ""
    @Published var results: [SearchResult] = []
    @Published var isLoading = false
    @Published var error: Error?
    @Published var hasMoreResults = false
    
    private var currentPage = 0
    private var cancellables = Set<AnyCancellable>()
    private let apiClient: APIClient
    
    init(apiClient: APIClient = APIClient(baseURL: URL(string: "https://api.example.com")!)) {
        self.apiClient = apiClient
        setupSearch()
    }
    
    private func setupSearch() {
        $query
            .debounce(for: .milliseconds(300), scheduler: DispatchQueue.main)
            .removeDuplicates()
            .handleEvents(receiveOutput: { [weak self] _ in
                self?.results = []
                self?.currentPage = 0
            })
            .filter { !$0.trimmingCharacters(in: .whitespaces).isEmpty }
            .flatMap { [weak self] query -> AnyPublisher<[SearchResult], Never> in
                guard let self = self else { return Just([]).eraseToAnyPublisher() }
                
                self.isLoading = true
                self.error = nil
                
                return self.performSearch(query: query, page: 0)
            }
            .receive(on: DispatchQueue.main)
            .handleEvents(receiveOutput: { [weak self] _ in
                self?.isLoading = false
            })
            .assign(to: &$results)
    }
    
    private func performSearch(query: String, page: Int) -> AnyPublisher<[SearchResult], Never> {
        // Simulated search
        let mockResults = (0..<10).map { index in
            SearchResult(
                id: page * 10 + index,
                title: "\(query) result #\(page * 10 + index)",
                description: "Description for result \(index)"
            )
        }
        
        return Just(mockResults)
            .delay(for: .milliseconds(300), scheduler: DispatchQueue.global())
            .catch { [weak self] error -> Just<[SearchResult]> in
                DispatchQueue.main.async {
                    self?.error = error
                    self?.isLoading = false
                }
                return Just([])
            }
            .eraseToAnyPublisher()
    }
    
    func loadMoreResults() {
        guard !isLoading, !query.isEmpty else { return }
        currentPage += 1
        
        performSearch(query: query, page: currentPage)
            .receive(on: DispatchQueue.main)
            .sink { [weak self] newResults in
                self?.results.append(contentsOf: newResults)
                self?.hasMoreResults = !newResults.isEmpty
            }
            .store(in: &cancellables)
    }
}
```

### 8.3 Pagination with State Machine

```swift
import Combine
import Foundation

// State Machine สำหรับ Pagination
enum PaginationState<T> {
    case idle
    case loading(page: Int)
    case loaded(items: [T], currentPage: Int, hasMore: Bool)
    case loadingMore(items: [T], currentPage: Int)
    case error(Error, items: [T])
    
    var items: [T] {
        switch self {
        case .loaded(let items, _, _), .loadingMore(let items, _), .error(_, let items):
            return items
        default:
            return []
        }
    }
    
    var isLoading: Bool {
        if case .loading = self { return true }
        if case .loadingMore = self { return true }
        return false
    }
    
    var canLoadMore: Bool {
        if case .loaded(_, _, let hasMore) = self { return hasMore }
        return false
    }
}

class PaginatedViewModel<T: Codable & Identifiable>: ObservableObject {
    @Published private(set) var state: PaginationState<T> = .idle
    
    private let pageSize = 20
    private var cancellables = Set<AnyCancellable>()
    private let fetchPage: (Int, Int) -> AnyPublisher<([T], Bool), Error>
    
    init(fetchPage: @escaping (Int, Int) -> AnyPublisher<([T], Bool), Error>) {
        self.fetchPage = fetchPage
    }
    
    func loadFirstPage() {
        state = .loading(page: 1)
        
        fetchPage(1, pageSize)
            .receive(on: DispatchQueue.main)
            .sink(
                receiveCompletion: { [weak self] completion in
                    if case .failure(let error) = completion {
                        self?.state = .error(error, items: self?.state.items ?? [])
                    }
                },
                receiveValue: { [weak self] items, hasMore in
                    self?.state = .loaded(items: items, currentPage: 1, hasMore: hasMore)
                }
            )
            .store(in: &cancellables)
    }
    
    func loadNextPage() {
        guard case .loaded(let items, let currentPage, _) = state, state.canLoadMore else { return }
        
        let nextPage = currentPage + 1
        state = .loadingMore(items: items, currentPage: currentPage)
        
        fetchPage(nextPage, pageSize)
            .receive(on: DispatchQueue.main)
            .sink(
                receiveCompletion: { [weak self] completion in
                    if case .failure(let error) = completion {
                        self?.state = .error(error, items: items)
                    }
                },
                receiveValue: { [weak self] newItems, hasMore in
                    let allItems = items + newItems
                    self?.state = .loaded(items: allItems, currentPage: nextPage, hasMore: hasMore)
                }
            )
            .store(in: &cancellables)
    }
}
```

---

## 9. Combine + Core Data

### 9.1 NSFetchedResultsController Publisher

```swift
import Combine
import CoreData

// สร้าง Publisher สำหรับ Core Data fetch results
class FetchedResultsPublisher<T: NSManagedObject>: NSObject, NSFetchedResultsControllerDelegate, Publisher {
    typealias Output = [T]
    typealias Failure = Error
    
    private let controller: NSFetchedResultsController<T>
    private var subject = PassthroughSubject<[T], Error>()
    
    init(
        fetchRequest: NSFetchRequest<T>,
        context: NSManagedObjectContext,
        sectionNameKeyPath: String? = nil,
        cacheName: String? = nil
    ) {
        controller = NSFetchedResultsController(
            fetchRequest: fetchRequest,
            managedObjectContext: context,
            sectionNameKeyPath: sectionNameKeyPath,
            cacheName: cacheName
        )
        
        super.init()
        controller.delegate = self
        
        do {
            try controller.performFetch()
        } catch {
            subject.send(completion: .failure(error))
        }
    }
    
    func receive<S>(subscriber: S) where S : Subscriber, Error == S.Failure, [T] == S.Input {
        subject.receive(subscriber: subscriber)
        
        // Send initial results
        if let objects = controller.fetchedObjects {
            subject.send(objects)
        }
    }
    
    // NSFetchedResultsControllerDelegate
    func controllerDidChangeContent(_ controller: NSFetchedResultsController<NSFetchRequestResult>) {
        if let objects = self.controller.fetchedObjects {
            subject.send(objects)
        }
    }
}

// ตัวอย่าง Core Data Entity
// สมมติมี TodoItem entity
class TodoListViewModel: ObservableObject {
    @Published var todos: [TodoItem] = []
    private var cancellables = Set<AnyCancellable>()
    
    init(context: NSManagedObjectContext) {
        let fetchRequest: NSFetchRequest<TodoItem> = TodoItem.fetchRequest()
        fetchRequest.sortDescriptors = [NSSortDescriptor(keyPath: \TodoItem.createdAt, ascending: false)]
        
        let publisher = FetchedResultsPublisher(fetchRequest: fetchRequest, context: context)
        
        publisher
            .receive(on: DispatchQueue.main)
            .catch { _ in Just([]) }
            .assign(to: &$todos)
    }
}

// placeholder
class TodoItem: NSManagedObject {
    @NSManaged var id: UUID
    @NSManaged var title: String
    @NSManaged var isCompleted: Bool
    @NSManaged var createdAt: Date
    
    class func fetchRequest() -> NSFetchRequest<TodoItem> {
        return NSFetchRequest<TodoItem>(entityName: "TodoItem")
    }
}
```

---

## 10. Advanced Memory Management ใน Combine

### 10.1 AnyCancellable Set Patterns

```swift
import Combine

// Pattern 1: Store ใน ViewModel
class ViewModel: ObservableObject {
    private var cancellables = Set<AnyCancellable>()
    
    // cancellables ถูก cancel เมื่อ ViewModel ถูก deallocate
    deinit {
        // ไม่จำเป็นต้อง cancel เอง - Set จัดการให้
        print("ViewModel deallocated")
    }
}

// Pattern 2: Keyed cancellables สำหรับ replacement
class SmartSubscriber {
    private var cancellables: [String: AnyCancellable] = [:]
    
    func subscribe(key: String, to publisher: AnyPublisher<Int, Never>) {
        // Cancel subscription เก่าก่อนสร้างใหม่
        cancellables[key]?.cancel()
        cancellables[key] = publisher.sink { print("\(key): \($0)") }
    }
    
    func unsubscribe(key: String) {
        cancellables.removeValue(forKey: key)
    }
}

// Pattern 3: Scope-based cancellation
class ScopeBasedSubscriber {
    private var cancellables = Set<AnyCancellable>()
    
    func withAutoCancel<T>(_ publisher: AnyPublisher<T, Never>, handler: @escaping (T) -> Void) {
        publisher.sink(receiveValue: handler).store(in: &cancellables)
    }
    
    func cancelAll() {
        cancellables.removeAll()
    }
}
```

### 10.2 Weak Captures ใน Pipelines

```swift
import Combine

class DataProcessor: ObservableObject {
    @Published var processedData: [String] = []
    private var cancellables = Set<AnyCancellable>()
    
    func processStream(_ publisher: AnyPublisher<String, Never>) {
        publisher
            // ใช้ [weak self] เพื่อป้องกัน retain cycle
            .map { [weak self] value -> String in
                guard let self = self else { return value }
                return self.transform(value)
            }
            // เมื่อ self เป็น nil ให้ filter ออก
            .compactMap { $0 }
            .receive(on: DispatchQueue.main)
            // assign(to:) ไม่สร้าง retain cycle
            .sink { [weak self] value in
                self?.processedData.append(value)
            }
            .store(in: &cancellables)
    }
    
    private func transform(_ value: String) -> String {
        return value.uppercased()
    }
}

// Retain Cycle ที่พบบ่อย
class LeakyViewModel: ObservableObject {
    @Published var data = ""
    private var cancellables = Set<AnyCancellable>()
    
    func badPattern() {
        Just("test")
            // WRONG: strong capture สร้าง retain cycle
            // .map { value in self.transform(value) }  // ❌
            
            // CORRECT: weak capture
            .map { [weak self] value in
                self?.transform(value) ?? value  // ✅
            }
            .sink { [weak self] value in
                self?.data = value  // ✅
            }
            .store(in: &cancellables)
    }
    
    func transform(_ value: String) -> String { value }
}

// Common memory leaks
class MemoryLeakExamples {
    var cancellables = Set<AnyCancellable>()
    
    // Leak 1: ไม่ store cancellable
    func leak1() {
        Just("hello").sink { print($0) }  // ❌ cancel ทันที เพราะไม่ store
    }
    
    // Leak 2: Strong capture ใน closure
    func leak2(service: SomeService) {
        service.publisher
            // .sink { self.process($0) }  // ❌ retain cycle
            .sink { [weak self] value in self?.process(value) }  // ✅
            .store(in: &cancellables)
    }
    
    func process(_ value: String) {}
}

class SomeService {
    var publisher: AnyPublisher<String, Never> {
        Just("data").eraseToAnyPublisher()
    }
}
```

---

## 11. Reactive Architecture

### 11.1 Unidirectional Data Flow กับ Combine

```swift
import Combine
import SwiftUI

// State - immutable state
struct AppState {
    var cart: CartState = CartState()
    var user: UserState = UserState()
    var products: ProductsState = ProductsState()
}

struct CartState {
    var items: [CartItem] = []
    var isLoading = false
    var total: Double {
        items.reduce(0) { $0 + $1.price * Double($1.quantity) }
    }
}

struct UserState {
    var isLoggedIn = false
    var currentUser: User? = nil
}

struct ProductsState {
    var items: [Product] = []
    var isLoading = false
    var searchQuery = ""
}

// Actions - events ที่เกิดขึ้น
enum AppAction {
    case cart(CartAction)
    case user(UserAction)
    case products(ProductsAction)
}

enum CartAction {
    case addItem(Product, quantity: Int)
    case removeItem(id: UUID)
    case updateQuantity(id: UUID, quantity: Int)
    case clearCart
}

enum UserAction {
    case login(email: String, password: String)
    case logout
    case loginSuccess(User)
    case loginFailed(Error)
}

enum ProductsAction {
    case fetchProducts
    case fetchProductsSuccess([Product])
    case fetchProductsFailed(Error)
    case updateSearch(String)
}

// Models
struct CartItem: Identifiable {
    let id: UUID
    let product: Product
    var quantity: Int
    var price: Double { product.price }
}

struct Product: Identifiable, Codable {
    let id: UUID
    let name: String
    let price: Double
    let description: String
}

// Store - single source of truth
class Store: ObservableObject {
    @Published private(set) var state: AppState
    
    private let reducer: (AppState, AppAction) -> AppState
    private var cancellables = Set<AnyCancellable>()
    
    init(
        initialState: AppState = AppState(),
        reducer: @escaping (AppState, AppAction) -> AppState
    ) {
        self.state = initialState
        self.reducer = reducer
    }
    
    func dispatch(_ action: AppAction) {
        state = reducer(state, action)
    }
}

// Reducers
func appReducer(state: AppState, action: AppAction) -> AppState {
    var newState = state
    
    switch action {
    case .cart(let cartAction):
        newState.cart = cartReducer(state: state.cart, action: cartAction)
    case .user(let userAction):
        newState.user = userReducer(state: state.user, action: userAction)
    case .products(let productsAction):
        newState.products = productsReducer(state: state.products, action: productsAction)
    }
    
    return newState
}

func cartReducer(state: CartState, action: CartAction) -> CartState {
    var newState = state
    
    switch action {
    case .addItem(let product, let quantity):
        if let index = newState.items.firstIndex(where: { $0.product.id == product.id }) {
            newState.items[index].quantity += quantity
        } else {
            newState.items.append(CartItem(id: UUID(), product: product, quantity: quantity))
        }
        
    case .removeItem(let id):
        newState.items.removeAll { $0.id == id }
        
    case .updateQuantity(let id, let quantity):
        if let index = newState.items.firstIndex(where: { $0.id == id }) {
            if quantity <= 0 {
                newState.items.remove(at: index)
            } else {
                newState.items[index].quantity = quantity
            }
        }
        
    case .clearCart:
        newState.items = []
    }
    
    return newState
}

func userReducer(state: UserState, action: UserAction) -> UserState {
    var newState = state
    switch action {
    case .loginSuccess(let user):
        newState.isLoggedIn = true
        newState.currentUser = user
    case .logout:
        newState.isLoggedIn = false
        newState.currentUser = nil
    default:
        break
    }
    return newState
}

func productsReducer(state: ProductsState, action: ProductsAction) -> ProductsState {
    var newState = state
    switch action {
    case .fetchProducts:
        newState.isLoading = true
    case .fetchProductsSuccess(let products):
        newState.items = products
        newState.isLoading = false
    case .fetchProductsFailed:
        newState.isLoading = false
    case .updateSearch(let query):
        newState.searchQuery = query
    }
    return newState
}
```

### 11.2 Full Shopping Cart Example

```swift
import SwiftUI
import Combine

struct ShoppingCartView: View {
    @EnvironmentObject var store: Store
    
    var body: some View {
        NavigationView {
            VStack {
                if store.state.cart.items.isEmpty {
                    EmptyCartView()
                } else {
                    CartItemsList()
                    CartSummary()
                }
            }
            .navigationTitle("ตะกร้าสินค้า")
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button("ล้างตะกร้า") {
                        store.dispatch(.cart(.clearCart))
                    }
                    .disabled(store.state.cart.items.isEmpty)
                }
            }
        }
    }
}

struct CartItemsList: View {
    @EnvironmentObject var store: Store
    
    var body: some View {
        List {
            ForEach(store.state.cart.items) { item in
                CartItemRow(item: item)
            }
            .onDelete { indexSet in
                indexSet.forEach { index in
                    let item = store.state.cart.items[index]
                    store.dispatch(.cart(.removeItem(id: item.id)))
                }
            }
        }
    }
}

struct CartItemRow: View {
    let item: CartItem
    @EnvironmentObject var store: Store
    
    var body: some View {
        HStack {
            VStack(alignment: .leading) {
                Text(item.product.name)
                    .font(.headline)
                Text("฿\(item.price, specifier: "%.2f") ต่อชิ้น")
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
            
            Spacer()
            
            HStack {
                Button("-") {
                    store.dispatch(.cart(.updateQuantity(id: item.id, quantity: item.quantity - 1)))
                }
                .buttonStyle(.bordered)
                
                Text("\(item.quantity)")
                    .frame(width: 30, alignment: .center)
                
                Button("+") {
                    store.dispatch(.cart(.updateQuantity(id: item.id, quantity: item.quantity + 1)))
                }
                .buttonStyle(.bordered)
            }
        }
        .padding(.vertical, 4)
    }
}

struct CartSummary: View {
    @EnvironmentObject var store: Store
    
    var body: some View {
        VStack(spacing: 8) {
            Divider()
            HStack {
                Text("รวมทั้งหมด")
                    .font(.headline)
                Spacer()
                Text("฿\(store.state.cart.total, specifier: "%.2f")")
                    .font(.title2)
                    .fontWeight(.bold)
                    .foregroundColor(.blue)
            }
            .padding()
            
            Button("สั่งซื้อ") {
                // Process order
            }
            .buttonStyle(.borderedProminent)
            .padding()
        }
    }
}

struct EmptyCartView: View {
    var body: some View {
        VStack(spacing: 16) {
            Image(systemName: "cart.badge.minus")
                .font(.system(size: 60))
                .foregroundColor(.secondary)
            Text("ตะกร้าสินค้าว่างเปล่า")
                .font(.title2)
                .foregroundColor(.secondary)
        }
        .frame(maxWidth: .infinity, maxHeight: .infinity)
    }
}
```

---

## 12. Exercises with Solutions

### Exercise 1: สร้าง Password Strength Publisher

```swift
// โจทย์: สร้าง Publisher ที่ตรวจสอบความแข็งแกร่งของ password
// และคืนค่า PasswordStrength enum

enum PasswordStrength: CaseIterable {
    case weak
    case fair
    case strong
    case veryStrong
    
    var description: String {
        switch self {
        case .weak: return "อ่อนแอ"
        case .fair: return "ปานกลาง"
        case .strong: return "แข็งแกร่ง"
        case .veryStrong: return "แข็งแกร่งมาก"
        }
    }
    
    var color: String {
        switch self {
        case .weak: return "red"
        case .fair: return "orange"
        case .strong: return "yellow"
        case .veryStrong: return "green"
        }
    }
}

// Solution
extension Publisher where Output == String, Failure == Never {
    func passwordStrength() -> AnyPublisher<PasswordStrength, Never> {
        return self.map { password -> PasswordStrength in
            var score = 0
            
            if password.count >= 8 { score += 1 }
            if password.count >= 12 { score += 1 }
            if password.contains(where: { $0.isUppercase }) { score += 1 }
            if password.contains(where: { $0.isLowercase }) { score += 1 }
            if password.contains(where: { $0.isNumber }) { score += 1 }
            let specialChars = "!@#$%^&*()_+-=[]{}|;':\",./<>?"
            if password.contains(where: { specialChars.contains($0) }) { score += 2 }
            
            switch score {
            case 0...2: return .weak
            case 3...4: return .fair
            case 5...6: return .strong
            default: return .veryStrong
            }
        }
        .eraseToAnyPublisher()
    }
}

// การใช้งาน
class PasswordViewModel: ObservableObject {
    @Published var password = ""
    @Published var strength: PasswordStrength = .weak
    
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        $password
            .passwordStrength()
            .assign(to: &$strength)
    }
}
```

### Exercise 2: Network Request Queue

```swift
// โจทย์: สร้าง request queue ที่ส่ง requests ทีละ N รายการ

class NetworkQueue {
    private let maxConcurrent: Int
    private var cancellables = Set<AnyCancellable>()
    
    init(maxConcurrent: Int = 3) {
        self.maxConcurrent = maxConcurrent
    }
    
    func execute<T>(
        requests: [AnyPublisher<T, Error>]
    ) -> AnyPublisher<[T], Error> {
        
        // แบ่ง requests เป็น batches
        let batches = requests.chunked(into: maxConcurrent)
        
        // ดำเนินการทีละ batch แล้ว collect ผลลัพธ์
        return batches.publisher
            .flatMap(maxPublishers: .max(1)) { batch in
                Publishers.MergeMany(batch)
                    .collect()
            }
            .collect()
            .map { $0.flatMap { $0 } }
            .eraseToAnyPublisher()
    }
}

extension Array {
    func chunked(into size: Int) -> [[Element]] {
        return stride(from: 0, to: count, by: size).map {
            Array(self[$0 ..< Swift.min($0 + size, count)])
        }
    }
}

// ทดสอบ
let queue = NetworkQueue(maxConcurrent: 2)
var cancellables = Set<AnyCancellable>()

let requests: [AnyPublisher<Int, Error>] = (1...10).map { n in
    Just(n)
        .setFailureType(to: Error.self)
        .delay(for: .milliseconds(100), scheduler: DispatchQueue.global())
        .eraseToAnyPublisher()
}

queue.execute(requests: requests)
    .sink(
        receiveCompletion: { print("Queue completed: \($0)") },
        receiveValue: { results in print("Results: \(results)") }
    )
    .store(in: &cancellables)
```

### Exercise 3: Real-time Stock Price Simulator

```swift
// โจทย์: สร้าง stock price simulator ที่ใช้ Combine

struct StockPrice {
    let symbol: String
    let price: Double
    let change: Double
    let changePercent: Double
    let timestamp: Date
}

class StockPriceSimulator {
    private var cancellables = Set<AnyCancellable>()
    
    func pricePublisher(
        symbol: String,
        initialPrice: Double
    ) -> AnyPublisher<StockPrice, Never> {
        
        var currentPrice = initialPrice
        
        return Timer.publish(every: 1.0, on: .main, in: .common)
            .autoconnect()
            .map { _ -> StockPrice in
                // Simulate price movement
                let change = Double.random(in: -2.0...2.0)
                let previousPrice = currentPrice
                currentPrice = max(0.01, currentPrice + change)
                let changeAmount = currentPrice - previousPrice
                let changePercent = (changeAmount / previousPrice) * 100
                
                return StockPrice(
                    symbol: symbol,
                    price: currentPrice,
                    change: changeAmount,
                    changePercent: changePercent,
                    timestamp: Date()
                )
            }
            .eraseToAnyPublisher()
    }
    
    func portfolioPublisher(
        symbols: [(symbol: String, initialPrice: Double)]
    ) -> AnyPublisher<[StockPrice], Never> {
        
        let publishers = symbols.map { symbolData in
            pricePublisher(symbol: symbolData.symbol, initialPrice: symbolData.initialPrice)
        }
        
        return Publishers.MergeMany(publishers)
            .scan([String: StockPrice]()) { dict, price in
                var newDict = dict
                newDict[price.symbol] = price
                return newDict
            }
            .map { dict in Array(dict.values).sorted { $0.symbol < $1.symbol } }
            .eraseToAnyPublisher()
    }
}

class StockPortfolioViewModel: ObservableObject {
    @Published var stocks: [StockPrice] = []
    
    private let simulator = StockPriceSimulator()
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        let portfolio: [(symbol: String, initialPrice: Double)] = [
            ("AAPL", 175.50),
            ("GOOGL", 140.20),
            ("MSFT", 380.00),
            ("AMZN", 185.75),
            ("TSLA", 240.00)
        ]
        
        simulator.portfolioPublisher(symbols: portfolio)
            .assign(to: &$stocks)
    }
    
    var totalValue: Double {
        stocks.reduce(0) { $0 + $1.price }
    }
    
    var totalChange: Double {
        stocks.reduce(0) { $0 + $1.change }
    }
}
```

---

## สรุป

ในบทนี้เราได้เรียนรู้ Combine framework อย่างครอบคลุม ตั้งแต่:

1. **Architecture พื้นฐาน**: Publisher, Subscriber, Subscription, Demand และ backpressure
2. **Custom implementations**: สร้าง Publisher และ Subscriber เอง
3. **Advanced operators**: flatMap vs switchToLatest, merge/combineLatest/zip, backpressure operators
4. **Subjects**: PassthroughSubject vs CurrentValueSubject, thread safety
5. **SwiftUI integration**: onReceive, @ObservableObject, assign patterns
6. **Async/await bridging**: values property, converting between Combine and async
7. **Testing**: XCTestExpectation, sink testing, spy patterns
8. **Real-world patterns**: Form validation, search, pagination, state machines
9. **Core Data integration**: FetchedResultsController publisher
10. **Memory management**: Cancellable patterns, retain cycle prevention
11. **Reactive architecture**: Unidirectional data flow, Redux-like store
12. **Exercises**: Password strength, network queue, stock simulator

การเรียนรู้ Combine ต้องการการฝึกฝนอย่างต่อเนื่อง แนะนำให้ลองสร้างโปรเจกต์จริงโดยใช้ patterns ที่เรียนรู้มาในบทนี้

---

*จบ Part 91: Advanced Combine Framework และ Reactive Programming Patterns*
