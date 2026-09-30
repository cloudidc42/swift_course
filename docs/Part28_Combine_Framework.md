# Part 28: Combine Framework

## บทนำ (Introduction)

Combine เป็น Framework ของ Apple ที่ใช้สำหรับการจัดการ asynchronous events และ data streams ด้วย declarative Swift API มันถูกแนะนำใน iOS 13 และ macOS 10.15 และกลายเป็นส่วนสำคัญในการพัฒนาแอป Apple โดยเฉพาะเมื่อใช้ร่วมกับ SwiftUI

---

## 1. What is Combine (Combine คืออะไร)

Combine Framework ใช้ paradigm ของ Reactive Programming โดยอนุญาตให้เขียนโค้ดที่ตอบสนองต่อการเปลี่ยนแปลงของข้อมูลโดยอัตโนมัติ

### แนวคิดหลัก

```
[Publisher] --> [Operators] --> [Subscriber]
   ข้อมูล    -->   แปลงข้อมูล  -->   รับข้อมูล
```

### ทำไมต้องใช้ Combine

```swift
// แบบเดิม (Callback Hell)
func fetchUser(id: Int, completion: @escaping (Result<User, Error>) -> Void) {
    URLSession.shared.dataTask(with: URL(string: "api/users/\(id)")!) { data, _, error in
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
            self.fetchPosts(userId: user.id) { result in
                switch result {
                case .success(let posts):
                    completion(.success(user))
                case .failure(let error):
                    completion(.failure(error))
                }
            }
        } catch {
            completion(.failure(error))
        }
    }.resume()
}

// แบบ Combine (Declarative)
import Combine

func fetchUserWithCombine(id: Int) -> AnyPublisher<User, Error> {
    URLSession.shared.dataTaskPublisher(for: URL(string: "api/users/\(id)")!)
        .map(\.data)
        .decode(type: User.self, decoder: JSONDecoder())
        .flatMap { user in
            self.fetchPostsPublisher(userId: user.id)
                .map { _ in user }
        }
        .receive(on: DispatchQueue.main)
        .eraseToAnyPublisher()
}
```

---

## 2. Publisher และ Subscriber

### Publisher Protocol

Publisher คือตัวส่งข้อมูล มี 2 สิ่งที่ต้องกำหนด:
- `Output`: ประเภทของข้อมูลที่ส่ง
- `Failure`: ประเภท Error ที่อาจเกิดขึ้น (ใช้ `Never` ถ้าไม่มี error)

```swift
import Combine

// สร้าง Custom Publisher
struct CountdownPublisher: Publisher {
    typealias Output = Int
    typealias Failure = Never
    
    let start: Int
    
    func receive<S>(subscriber: S) where S: Subscriber,
        S.Input == Int, S.Failure == Never {
        let subscription = CountdownSubscription(subscriber: subscriber, start: start)
        subscriber.receive(subscription: subscription)
    }
}

class CountdownSubscription<S: Subscriber>: Subscription where S.Input == Int, S.Failure == Never {
    var subscriber: S?
    var current: Int
    
    init(subscriber: S, start: Int) {
        self.subscriber = subscriber
        self.current = start
    }
    
    func request(_ demand: Subscribers.Demand) {
        while current >= 0 {
            _ = subscriber?.receive(current)
            current -= 1
        }
        subscriber?.receive(completion: .finished)
    }
    
    func cancel() {
        subscriber = nil
    }
}

// การใช้งาน
let countdown = CountdownPublisher(start: 5)
let cancellable = countdown.sink { value in
    print("นับถอยหลัง: \(value)")
}
```

### Subscriber Protocol

```swift
// สร้าง Custom Subscriber
class PrintSubscriber: Subscriber {
    typealias Input = String
    typealias Failure = Never
    
    func receive(subscription: Subscription) {
        print("เริ่ม Subscription")
        subscription.request(.unlimited)
    }
    
    func receive(_ input: String) -> Subscribers.Demand {
        print("ได้รับ: \(input)")
        return .none
    }
    
    func receive(completion: Subscribers.Completion<Never>) {
        print("สิ้นสุด: \(completion)")
    }
}

// การใช้งาน
let publisher = ["สวัสดี", "โลก", "จาก", "Combine"].publisher
let subscriber = PrintSubscriber()
publisher.subscribe(subscriber)
```

---

## 3. Subscription

Subscription เป็นตัวเชื่อมระหว่าง Publisher และ Subscriber

```swift
import Combine

var cancellables: Set<AnyCancellable> = []

// Subscription พื้นฐาน
let numbers = [1, 2, 3, 4, 5].publisher

let subscription = numbers
    .filter { $0 % 2 == 0 }
    .map { $0 * $0 }
    .sink { completion in
        print("เสร็จแล้ว: \(completion)")
    } receiveValue: { value in
        print("ได้รับ: \(value)")
    }

// เก็บ subscription เพื่อไม่ให้ถูก deallocated
subscription.store(in: &cancellables)
```

---

## 4. Built-in Publishers

### Just

ส่งค่าเดียวแล้วสิ้นสุด

```swift
import Combine

// Just ส่งค่าเดียวและ complete
let justPublisher = Just("สวัสดี Combine!")

justPublisher
    .sink(
        receiveCompletion: { print("Just เสร็จแล้ว: \($0)") },
        receiveValue: { print("Just ได้รับ: \($0)") }
    )
// Output: Just ได้รับ: สวัสดี Combine!
//         Just เสร็จแล้ว: finished

// Just กับ Error type
let justWithError: AnyPublisher<Int, Error> = Just(42)
    .setFailureType(to: Error.self)
    .eraseToAnyPublisher()
```

### Future

จัดการ asynchronous operation ที่ให้ผลลัพธ์เดียว

```swift
import Combine
import Foundation

// Future สำหรับการทำงาน async
func fetchData(id: Int) -> Future<String, Error> {
    Future { promise in
        // จำลองการเรียก API
        DispatchQueue.global().asyncAfter(deadline: .now() + 1) {
            if id > 0 {
                promise(.success("ข้อมูลสำหรับ ID: \(id)"))
            } else {
                promise(.failure(NSError(domain: "app", code: -1, userInfo: [NSLocalizedDescriptionKey: "ID ไม่ถูกต้อง"])))
            }
        }
    }
}

var cancellables: Set<AnyCancellable> = []

fetchData(id: 1)
    .sink(
        receiveCompletion: { completion in
            if case .failure(let error) = completion {
                print("Error: \(error.localizedDescription)")
            }
        },
        receiveValue: { data in
            print("ได้รับข้อมูล: \(data)")
        }
    )
    .store(in: &cancellables)

// Future กับ async/await
func modernFetch(id: Int) -> Future<String, Error> {
    Future { promise in
        Task {
            do {
                let result = try await someAsyncOperation(id: id)
                promise(.success(result))
            } catch {
                promise(.failure(error))
            }
        }
    }
}

func someAsyncOperation(id: Int) async throws -> String {
    try await Task.sleep(nanoseconds: 1_000_000_000)
    return "ผลลัพธ์จาก async: \(id)"
}
```

### Fail

ส่ง error ทันที

```swift
import Combine

enum AppError: Error, LocalizedError {
    case unauthorized
    case notFound
    case serverError(Int)
    
    var errorDescription: String? {
        switch self {
        case .unauthorized: return "ไม่มีสิทธิ์เข้าถึง"
        case .notFound: return "ไม่พบข้อมูล"
        case .serverError(let code): return "Server Error: \(code)"
        }
    }
}

let failPublisher = Fail<String, AppError>(error: .unauthorized)

failPublisher
    .catch { error in
        Just("ข้อมูล default เนื่องจาก: \(error.localizedDescription)")
    }
    .sink { value in
        print(value)
    }
    .store(in: &cancellables)
```

### Empty

ไม่ส่งค่าใด แต่ complete ทันที

```swift
import Combine

// Empty เสมือน Publisher ที่ว่างเปล่า
let emptyPublisher = Empty<Int, Never>()

emptyPublisher
    .sink(
        receiveCompletion: { print("Empty เสร็จ: \($0)") },
        receiveValue: { print("ได้รับ: \($0)") }
    )
    .store(in: &cancellables)
// Output: Empty เสร็จ: finished

// ใช้ Empty เพื่อทดแทน Publisher เมื่อไม่มีข้อมูล
func fetchOptionalData() -> AnyPublisher<String, Never> {
    guard let cachedData = UserDefaults.standard.string(forKey: "cache") else {
        return Empty().eraseToAnyPublisher()
    }
    return Just(cachedData).eraseToAnyPublisher()
}
```

### Deferred

สร้าง Publisher ใหม่ทุกครั้งที่มี Subscriber

```swift
import Combine

// Deferred สร้าง Publisher เมื่อมี subscription เท่านั้น
var counter = 0

let deferredPublisher = Deferred {
    counter += 1
    print("สร้าง Publisher ครั้งที่: \(counter)")
    return Just(counter)
}

// สร้าง Publisher ทุกครั้งที่ subscribe
deferredPublisher.sink { print("Subscriber 1: \($0)") }.store(in: &cancellables)
deferredPublisher.sink { print("Subscriber 2: \($0)") }.store(in: &cancellables)
// Output:
// สร้าง Publisher ครั้งที่: 1
// Subscriber 1: 1
// สร้าง Publisher ครั้งที่: 2
// Subscriber 2: 2
```

---

## 5. PassthroughSubject

Subject ที่ไม่เก็บค่า ส่งค่าให้ผู้ subscribe ที่ active เท่านั้น

```swift
import Combine

class EventBus {
    static let shared = EventBus()
    
    let userLoggedIn = PassthroughSubject<String, Never>()
    let errorOccurred = PassthroughSubject<AppError, Never>()
    
    private init() {}
}

// การส่ง event
EventBus.shared.userLoggedIn.send("user123")

// การรับ event
EventBus.shared.userLoggedIn
    .sink { userId in
        print("User เข้าสู่ระบบ: \(userId)")
    }
    .store(in: &cancellables)

// ตัวอย่างที่สมบูรณ์
class LoginViewModel: ObservableObject {
    let loginSubject = PassthroughSubject<LoginResult, Never>()
    private var cancellables: Set<AnyCancellable> = []
    
    @Published var isLoggedIn = false
    @Published var errorMessage: String?
    
    init() {
        loginSubject
            .sink { result in
                switch result {
                case .success(let user):
                    self.isLoggedIn = true
                    print("เข้าสู่ระบบสำเร็จ: \(user)")
                case .failure(let error):
                    self.errorMessage = error.localizedDescription
                }
            }
            .store(in: &cancellables)
    }
    
    func login(email: String, password: String) {
        // จำลองการ login
        if email == "test@test.com" && password == "password" {
            loginSubject.send(.success("test_user"))
        } else {
            loginSubject.send(.failure(AppError.unauthorized))
        }
    }
}

enum LoginResult {
    case success(String)
    case failure(AppError)
}
```

---

## 6. CurrentValueSubject

Subject ที่เก็บค่าปัจจุบัน และส่งค่านั้นให้ subscriber ใหม่ทันที

```swift
import Combine

class TemperatureMonitor {
    // CurrentValueSubject เก็บและส่งค่าปัจจุบัน
    let temperature = CurrentValueSubject<Double, Never>(20.0)
    private var cancellables: Set<AnyCancellable> = []
    
    init() {
        // จำลองการเปลี่ยนแปลงอุณหภูมิ
        Timer.publish(every: 1, on: .main, in: .common)
            .autoconnect()
            .sink { _ in
                let newTemp = Double.random(in: 15...35)
                self.temperature.send(newTemp)
            }
            .store(in: &cancellables)
    }
    
    var currentTemp: Double {
        temperature.value // เข้าถึงค่าปัจจุบัน
    }
}

// การใช้งาน
let monitor = TemperatureMonitor()

// Subscriber ใหม่จะได้รับค่าปัจจุบัน 20.0 ทันที
monitor.temperature
    .sink { temp in
        print("อุณหภูมิ: \(String(format: "%.1f", temp))°C")
    }
    .store(in: &cancellables)

print("อุณหภูมิตอนนี้: \(monitor.currentTemp)°C")
```

---

## 7. Operators

### map

แปลง Output เป็นอีกประเภทหนึ่ง

```swift
import Combine

// map พื้นฐาน
let numbers = [1, 2, 3, 4, 5].publisher

numbers
    .map { $0 * 2 }
    .sink { print($0) }
    .store(in: &cancellables)
// Output: 2, 4, 6, 8, 10

// map กับ String
let strings = ["hello", "world", "combine"].publisher

strings
    .map { $0.uppercased() }
    .map { "🔹 \($0)" }
    .sink { print($0) }
    .store(in: &cancellables)

// tryMap - map ที่อาจ throw error
let jsonStrings = ["[1,2,3]", "invalid", "[4,5,6]"].publisher

jsonStrings
    .tryMap { string -> [Int] in
        guard let data = string.data(using: .utf8) else {
            throw AppError.notFound
        }
        return try JSONDecoder().decode([Int].self, from: data)
    }
    .replaceError(with: [])
    .sink { print($0) }
    .store(in: &cancellables)
```

### filter

กรองเฉพาะค่าที่ตรงเงื่อนไข

```swift
import Combine

let numbers = (1...20).publisher

numbers
    .filter { $0 % 3 == 0 }  // เฉพาะหารด้วย 3 ลงตัว
    .filter { $0 > 5 }        // และมากกว่า 5
    .sink { print($0) }
    .store(in: &cancellables)
// Output: 6, 9, 12, 15, 18

// compactMap - filter nil ออกอัตโนมัติ
let optionals: [Int?] = [1, nil, 3, nil, 5, nil, 7]

optionals.publisher
    .compactMap { $0 }
    .sink { print($0) }
    .store(in: &cancellables)
// Output: 1, 3, 5, 7

// tryFilter - filter ที่อาจ throw error
let values = [1, -2, 3, -4, 5].publisher

values
    .tryFilter { value in
        guard value != 0 else { throw AppError.notFound }
        return value > 0
    }
    .replaceError(with: 0)
    .sink { print($0) }
    .store(in: &cancellables)
```

### flatMap

แปลง Publisher เป็น Publisher อื่น (เหมือน flatMap ทั่วไป)

```swift
import Combine

// flatMap สำหรับ nested Publishers
let userIds = [1, 2, 3].publisher

func fetchUser(id: Int) -> AnyPublisher<String, Never> {
    Just("ผู้ใช้ #\(id)").delay(for: 0.1, scheduler: DispatchQueue.main).eraseToAnyPublisher()
}

userIds
    .flatMap { id in
        fetchUser(id: id)
    }
    .sink { user in
        print("ดึงข้อมูล: \(user)")
    }
    .store(in: &cancellables)

// flatMap กับ maxPublishers เพื่อจำกัดจำนวน concurrent requests
userIds
    .flatMap(maxPublishers: .max(2)) { id in
        fetchUser(id: id)
    }
    .sink { print($0) }
    .store(in: &cancellables)
```

### merge, zip, combineLatest

```swift
import Combine

// merge - รวม Publishers หลายตัวเป็นหนึ่ง
let publisher1 = [1, 2, 3].publisher
let publisher2 = [4, 5, 6].publisher

Publishers.Merge(publisher1, publisher2)
    .sink { print("merged: \($0)") }
    .store(in: &cancellables)

// zip - รวมค่าจาก Publishers ที่ตรงคู่กัน
let names = ["อลิซ", "บ็อบ", "ชาลี"].publisher
let scores = [85, 92, 78].publisher

Publishers.Zip(names, scores)
    .sink { name, score in
        print("\(name): \(score) คะแนน")
    }
    .store(in: &cancellables)

// combineLatest - รวมค่าล่าสุดจากทุก Publisher
let subject1 = CurrentValueSubject<Int, Never>(0)
let subject2 = CurrentValueSubject<String, Never>("ว่าง")

Publishers.CombineLatest(subject1, subject2)
    .sink { number, text in
        print("ตัวเลข: \(number), ข้อความ: \(text)")
    }
    .store(in: &cancellables)

subject1.send(1)         // Output: ตัวเลข: 1, ข้อความ: ว่าง
subject2.send("มีค่า")   // Output: ตัวเลข: 1, ข้อความ: มีค่า
subject1.send(2)         // Output: ตัวเลข: 2, ข้อความ: มีค่า
```

### debounce และ throttle

```swift
import Combine

let searchText = PassthroughSubject<String, Never>()

// debounce - รอจนกว่าจะหยุดส่งค่าในช่วงเวลาที่กำหนด
searchText
    .debounce(for: .milliseconds(500), scheduler: DispatchQueue.main)
    .sink { text in
        print("ค้นหา: \(text)")
    }
    .store(in: &cancellables)

// throttle - ส่งค่าไม่เกินครั้งต่อช่วงเวลาที่กำหนด
searchText
    .throttle(for: .seconds(1), scheduler: DispatchQueue.main, latest: true)
    .sink { text in
        print("throttled: \(text)")
    }
    .store(in: &cancellables)

// จำลองการพิมพ์
searchText.send("s")
searchText.send("sw")
searchText.send("swi")
searchText.send("swif")
searchText.send("swift") // debounce จะส่งแค่นี้
```

### delay

```swift
import Combine

Just("ข้อความล่าช้า")
    .delay(for: .seconds(2), scheduler: DispatchQueue.main)
    .sink { message in
        print("ได้รับหลัง 2 วินาที: \(message)")
    }
    .store(in: &cancellables)
```

### retry

```swift
import Combine
import Foundation

var attemptCount = 0

func unreliableOperation() -> AnyPublisher<String, Error> {
    Future { promise in
        attemptCount += 1
        print("ลองครั้งที่: \(attemptCount)")
        if attemptCount < 3 {
            promise(.failure(AppError.serverError(500)))
        } else {
            promise(.success("สำเร็จ!"))
        }
    }.eraseToAnyPublisher()
}

unreliableOperation()
    .retry(3)
    .sink(
        receiveCompletion: { print("เสร็จ: \($0)") },
        receiveValue: { print("ผลลัพธ์: \($0)") }
    )
    .store(in: &cancellables)
// Output:
// ลองครั้งที่: 1
// ลองครั้งที่: 2
// ลองครั้งที่: 3
// ผลลัพธ์: สำเร็จ!
```

### catch และ replaceError

```swift
import Combine

// catch - จัดการ error และแทนที่ด้วย Publisher ใหม่
let failingPublisher = Fail<String, AppError>(error: .notFound)

failingPublisher
    .catch { error -> AnyPublisher<String, Never> in
        print("เกิด error: \(error.localizedDescription)")
        return Just("ข้อมูล fallback").eraseToAnyPublisher()
    }
    .sink { value in
        print("ได้รับ: \(value)")
    }
    .store(in: &cancellables)

// replaceError - แทน error ด้วยค่าที่กำหนด
let anotherFailing = Fail<Int, Error>(error: NSError(domain: "test", code: 1))

anotherFailing
    .replaceError(with: -1)
    .sink { print($0) }
    .store(in: &cancellables)
// Output: -1
```

### reduce และ scan

```swift
import Combine

// reduce - รวมค่าทั้งหมดเป็นค่าเดียวตอนสิ้นสุด
let numbers = [1, 2, 3, 4, 5].publisher

numbers
    .reduce(0) { total, current in total + current }
    .sink { total in
        print("ผลรวม: \(total)")
    }
    .store(in: &cancellables)
// Output: ผลรวม: 15

// scan - เหมือน reduce แต่ส่งค่าสะสมทุกขั้นตอน
numbers
    .scan(0) { total, current in total + current }
    .sink { print("สะสม: \($0)") }
    .store(in: &cancellables)
// Output: สะสม: 1, 2, 6, 10, 15
```

### collect

```swift
import Combine

// collect - รวมค่าทั้งหมดเป็น array
let values = [1, 2, 3, 4, 5].publisher

values
    .collect()
    .sink { allValues in
        print("ค่าทั้งหมด: \(allValues)")
    }
    .store(in: &cancellables)
// Output: ค่าทั้งหมด: [1, 2, 3, 4, 5]

// collect กับจำนวนที่กำหนด
values
    .collect(2)
    .sink { batch in
        print("batch: \(batch)")
    }
    .store(in: &cancellables)
// Output: batch: [1, 2]
//         batch: [3, 4]
//         batch: [5]
```

### Operators อื่นๆ ที่มีประโยชน์

```swift
import Combine

let numbers = [1, 2, 3, 4, 5].publisher

// first - รับเฉพาะค่าแรก
numbers.first().sink { print("first: \($0)") }.store(in: &cancellables)

// last - รับเฉพาะค่าสุดท้าย
numbers.last().sink { print("last: \($0)") }.store(in: &cancellables)

// drop - ข้ามค่าแรก N ค่า
numbers.dropFirst(2).sink { print("dropped: \($0)") }.store(in: &cancellables)
// Output: 3, 4, 5

// prefix - รับแค่ N ค่าแรก
numbers.prefix(3).sink { print("prefix: \($0)") }.store(in: &cancellables)
// Output: 1, 2, 3

// removeDuplicates - ลบค่าซ้ำที่ติดกัน
[1, 1, 2, 2, 3, 1, 1].publisher
    .removeDuplicates()
    .sink { print($0) }
    .store(in: &cancellables)
// Output: 1, 2, 3, 1

// handleEvents - ดักจับ events ต่างๆ โดยไม่แก้ไข
numbers
    .handleEvents(
        receiveSubscription: { _ in print("เริ่ม subscription") },
        receiveOutput: { print("output: \($0)") },
        receiveCompletion: { print("completion: \($0)") },
        receiveCancel: { print("cancelled") }
    )
    .sink { _ in }
    .store(in: &cancellables)

// print - debug operator
numbers
    .print("DEBUG")
    .sink { _ in }
    .store(in: &cancellables)
```

---

## 8. Subscriber: sink และ assign

### sink

```swift
import Combine

let publisher = [1, 2, 3].publisher

// sink กับ completion
publisher
    .sink(
        receiveCompletion: { completion in
            switch completion {
            case .finished:
                print("สิ้นสุดแล้ว")
            case .failure(let error):
                print("Error: \(error)")
            }
        },
        receiveValue: { value in
            print("ค่า: \(value)")
        }
    )
    .store(in: &cancellables)

// sink สั้นๆ (ไม่ดักจับ completion)
publisher
    .sink { print($0) }
    .store(in: &cancellables)
```

### assign

```swift
import Combine
import SwiftUI

class TemperatureModel: ObservableObject {
    @Published var temperature: Double = 0
    @Published var displayText: String = ""
    private var cancellables: Set<AnyCancellable> = []
    
    init() {
        let tempSubject = PassthroughSubject<Double, Never>()
        
        // assign ผูก publisher กับ property โดยตรง
        tempSubject
            .assign(to: &$temperature)
        
        // assign(to:on:) สำหรับ KeyPath
        tempSubject
            .map { "อุณหภูมิ: \(String(format: "%.1f", $0))°C" }
            .assign(to: \.displayText, on: self)
            .store(in: &cancellables)
        
        // จำลองการอ่านค่า
        Timer.publish(every: 1, on: .main, in: .common)
            .autoconnect()
            .map { _ in Double.random(in: 20...30) }
            .sink { [weak self] temp in
                tempSubject.send(temp)
            }
            .store(in: &cancellables)
    }
}
```

---

## 9. AnyCancellable และ Cancellable

```swift
import Combine

var cancellables: Set<AnyCancellable> = []

// AnyCancellable เป็น type-erased wrapper สำหรับ Cancellable
let subscription = Just(42)
    .sink { print($0) }

// เก็บไว้ใน Set
subscription.store(in: &cancellables)

// หรือ assign ให้กับตัวแปร
var myCancellable: AnyCancellable? = Just("test").sink { print($0) }

// ยกเลิก subscription ด้วยตนเอง
myCancellable?.cancel()
myCancellable = nil // AnyCancellable จะ cancel เมื่อ deallocate

// Cancellable protocol
class MyCancellable: Cancellable {
    func cancel() {
        print("ยกเลิกแล้ว")
    }
}

// ตัวอย่างการจัดการ lifecycle
class ViewModel: ObservableObject {
    private var cancellables: Set<AnyCancellable> = []
    
    init() {
        setupSubscriptions()
    }
    
    func setupSubscriptions() {
        // subscriptions จะถูกยกเลิกอัตโนมัติเมื่อ ViewModel ถูก deallocate
        Timer.publish(every: 1, on: .main, in: .common)
            .autoconnect()
            .sink { _ in
                print("tick")
            }
            .store(in: &cancellables) // เก็บไว้ใน ViewModel
    }
    
    deinit {
        // cancellables จะถูก deallocate พร้อม ViewModel
        print("ViewModel deallocated")
    }
}
```

---

## 10. Combine กับ URLSession

```swift
import Combine
import Foundation

struct Article: Codable {
    let id: Int
    let title: String
    let body: String
}

class APIService {
    private var cancellables: Set<AnyCancellable> = []
    
    func fetchArticles() -> AnyPublisher<[Article], Error> {
        let url = URL(string: "https://jsonplaceholder.typicode.com/posts")!
        
        return URLSession.shared.dataTaskPublisher(for: url)
            .tryMap { output -> Data in
                guard let response = output.response as? HTTPURLResponse,
                      (200...299).contains(response.statusCode) else {
                    throw URLError(.badServerResponse)
                }
                return output.data
            }
            .decode(type: [Article].self, decoder: JSONDecoder())
            .receive(on: DispatchQueue.main)
            .eraseToAnyPublisher()
    }
    
    func fetchArticle(id: Int) -> AnyPublisher<Article, Error> {
        let url = URL(string: "https://jsonplaceholder.typicode.com/posts/\(id)")!
        
        return URLSession.shared.dataTaskPublisher(for: url)
            .map(\.data)
            .decode(type: Article.self, decoder: JSONDecoder())
            .receive(on: DispatchQueue.main)
            .eraseToAnyPublisher()
    }
}

// ViewModel ที่ใช้ APIService
class ArticleViewModel: ObservableObject {
    @Published var articles: [Article] = []
    @Published var isLoading = false
    @Published var errorMessage: String?
    
    private let apiService = APIService()
    private var cancellables: Set<AnyCancellable> = []
    
    func loadArticles() {
        isLoading = true
        errorMessage = nil
        
        apiService.fetchArticles()
            .sink(
                receiveCompletion: { [weak self] completion in
                    self?.isLoading = false
                    if case .failure(let error) = completion {
                        self?.errorMessage = error.localizedDescription
                    }
                },
                receiveValue: { [weak self] articles in
                    self?.articles = articles
                }
            )
            .store(in: &cancellables)
    }
}

// การทำ request หลาย requests พร้อมกัน
class MultiRequestService {
    private var cancellables: Set<AnyCancellable> = []
    
    func fetchMultipleArticles(ids: [Int]) -> AnyPublisher<[Article], Error> {
        let publishers = ids.map { id in
            URLSession.shared.dataTaskPublisher(
                for: URL(string: "https://jsonplaceholder.typicode.com/posts/\(id)")!
            )
            .map(\.data)
            .decode(type: Article.self, decoder: JSONDecoder())
            .eraseToAnyPublisher()
        }
        
        return Publishers.MergeMany(publishers)
            .collect()
            .receive(on: DispatchQueue.main)
            .eraseToAnyPublisher()
    }
}
```

---

## 11. Combine กับ NotificationCenter

```swift
import Combine
import UIKit

class NotificationViewModel: ObservableObject {
    @Published var keyboardHeight: CGFloat = 0
    @Published var orientation: UIDeviceOrientation = .portrait
    private var cancellables: Set<AnyCancellable> = []
    
    init() {
        setupKeyboardObserver()
        setupOrientationObserver()
    }
    
    func setupKeyboardObserver() {
        NotificationCenter.default.publisher(for: UIResponder.keyboardWillShowNotification)
            .compactMap { notification in
                notification.userInfo?[UIResponder.keyboardFrameEndUserInfoKey] as? CGRect
            }
            .map { $0.height }
            .receive(on: DispatchQueue.main)
            .assign(to: &$keyboardHeight)
        
        NotificationCenter.default.publisher(for: UIResponder.keyboardWillHideNotification)
            .map { _ in CGFloat(0) }
            .receive(on: DispatchQueue.main)
            .assign(to: &$keyboardHeight)
    }
    
    func setupOrientationObserver() {
        NotificationCenter.default.publisher(for: UIDevice.orientationDidChangeNotification)
            .map { _ in UIDevice.current.orientation }
            .receive(on: DispatchQueue.main)
            .assign(to: &$orientation)
    }
}

// Custom Notification
extension Notification.Name {
    static let dataUpdated = Notification.Name("dataUpdated")
    static let userLoggedOut = Notification.Name("userLoggedOut")
}

class DataManager {
    private var cancellables: Set<AnyCancellable> = []
    
    init() {
        NotificationCenter.default.publisher(for: .dataUpdated)
            .compactMap { $0.userInfo?["data"] as? [String: Any] }
            .sink { data in
                print("ข้อมูลอัพเดต: \(data)")
            }
            .store(in: &cancellables)
    }
    
    func updateData(_ data: [String: Any]) {
        NotificationCenter.default.post(
            name: .dataUpdated,
            object: nil,
            userInfo: ["data": data]
        )
    }
}
```

---

## 12. Combine กับ KVO

```swift
import Combine
import Foundation

// KVO (Key-Value Observing) กับ Combine
class DownloadTask: NSObject {
    @objc dynamic var progress: Double = 0
    @objc dynamic var isCompleted: Bool = false
    
    private var cancellables: Set<AnyCancellable> = []
    
    func start() {
        // จำลองการ download
        Timer.scheduledTimer(withTimeInterval: 0.1, repeats: true) { [weak self] timer in
            guard let self = self else {
                timer.invalidate()
                return
            }
            self.progress += 0.1
            if self.progress >= 1.0 {
                self.isCompleted = true
                timer.invalidate()
            }
        }
    }
}

class DownloadViewModel: ObservableObject {
    @Published var progressValue: Double = 0
    @Published var isCompleted: Bool = false
    
    private let task = DownloadTask()
    private var cancellables: Set<AnyCancellable> = []
    
    init() {
        // ใช้ KVO Publisher
        task.publisher(for: \.progress)
            .receive(on: DispatchQueue.main)
            .assign(to: &$progressValue)
        
        task.publisher(for: \.isCompleted)
            .receive(on: DispatchQueue.main)
            .assign(to: &$isCompleted)
    }
    
    func startDownload() {
        task.start()
    }
}
```

---

## 13. Combine กับ SwiftUI

```swift
import SwiftUI
import Combine

// ObservableObject กับ @Published
class UserSettings: ObservableObject {
    @Published var username = ""
    @Published var fontSize: Double = 16
    @Published var isDarkMode = false
    
    private var cancellables: Set<AnyCancellable> = []
    
    init() {
        // บันทึกการตั้งค่าทุกครั้งที่เปลี่ยน
        $username
            .debounce(for: .seconds(0.5), scheduler: DispatchQueue.main)
            .sink { username in
                UserDefaults.standard.set(username, forKey: "username")
            }
            .store(in: &cancellables)
        
        $isDarkMode
            .sink { isDark in
                UserDefaults.standard.set(isDark, forKey: "isDarkMode")
            }
            .store(in: &cancellables)
    }
}

struct SettingsView: View {
    @StateObject private var settings = UserSettings()
    
    var body: some View {
        Form {
            Section("โปรไฟล์") {
                TextField("ชื่อผู้ใช้", text: $settings.username)
            }
            
            Section("การแสดงผล") {
                Slider(value: $settings.fontSize, in: 10...30) {
                    Text("ขนาดตัวอักษร: \(Int(settings.fontSize))")
                }
                
                Toggle("โหมดมืด", isOn: $settings.isDarkMode)
            }
        }
        .navigationTitle("การตั้งค่า")
    }
}

// การใช้ @StateObject กับ Combine
class SearchViewModel: ObservableObject {
    @Published var searchText = ""
    @Published var results: [String] = []
    @Published var isSearching = false
    
    private var cancellables: Set<AnyCancellable> = []
    
    let sampleData = ["Swift", "SwiftUI", "Combine", "Xcode", "iOS", "macOS", "watchOS", "tvOS"]
    
    init() {
        $searchText
            .debounce(for: .milliseconds(300), scheduler: DispatchQueue.main)
            .removeDuplicates()
            .handleEvents(receiveOutput: { [weak self] _ in
                self?.isSearching = true
            })
            .map { [weak self] query -> [String] in
                guard let self = self else { return [] }
                guard !query.isEmpty else { return self.sampleData }
                return self.sampleData.filter { $0.localizedCaseInsensitiveContains(query) }
            }
            .handleEvents(receiveOutput: { [weak self] _ in
                self?.isSearching = false
            })
            .assign(to: &$results)
    }
}

struct SearchView: View {
    @StateObject private var viewModel = SearchViewModel()
    
    var body: some View {
        NavigationView {
            VStack {
                if viewModel.isSearching {
                    ProgressView()
                }
                
                List(viewModel.results, id: \.self) { result in
                    Text(result)
                }
            }
            .searchable(text: $viewModel.searchText, prompt: "ค้นหา...")
            .navigationTitle("ค้นหา")
        }
    }
}
```

---

## 14. Schedulers

Scheduler กำหนดว่า Pipeline จะทำงานบน thread ไหน

```swift
import Combine
import Foundation

var cancellables: Set<AnyCancellable> = []

// DispatchQueue.main - main thread (สำหรับ UI)
Just("Hello")
    .receive(on: DispatchQueue.main)
    .sink { print(Thread.isMainThread) } // true
    .store(in: &cancellables)

// DispatchQueue.global() - background thread
Just("Hello")
    .subscribe(on: DispatchQueue.global(qos: .background))
    .receive(on: DispatchQueue.main)
    .sink { _ in print("อยู่บน main: \(Thread.isMainThread)") }
    .store(in: &cancellables)

// RunLoop.main - สำหรับ timer-based work
Timer.publish(every: 1, on: .main, in: .common)
    .autoconnect()
    .sink { date in
        print("Timer: \(date)")
    }
    .store(in: &cancellables)

// OperationQueue
let queue = OperationQueue()
queue.maxConcurrentOperationCount = 1

[1, 2, 3].publisher
    .subscribe(on: queue)
    .receive(on: DispatchQueue.main)
    .sink { print($0) }
    .store(in: &cancellables)

// ImmediateScheduler - ทำงานทันที synchronously
Just("immediate")
    .receive(on: ImmediateScheduler.shared)
    .sink { print($0) }
    .store(in: &cancellables)

// Custom Scheduler Pattern
class NetworkService {
    private let backgroundQueue = DispatchQueue(
        label: "com.app.network",
        qos: .userInitiated,
        attributes: .concurrent
    )
    
    func fetch() -> AnyPublisher<Data, Error> {
        URLSession.shared
            .dataTaskPublisher(for: URL(string: "https://api.example.com")!)
            .subscribe(on: backgroundQueue)    // ทำงานบน background
            .receive(on: DispatchQueue.main)    // ส่งผลลัพธ์บน main
            .map(\.data)
            .mapError { $0 as Error }
            .eraseToAnyPublisher()
    }
}
```

---

## 15. Error Handling ใน Combine

```swift
import Combine

enum NetworkError: Error, LocalizedError {
    case invalidURL
    case noData
    case decodingError
    case serverError(Int)
    case unknown(Error)
    
    var errorDescription: String? {
        switch self {
        case .invalidURL: return "URL ไม่ถูกต้อง"
        case .noData: return "ไม่มีข้อมูล"
        case .decodingError: return "ไม่สามารถถอดรหัสข้อมูล"
        case .serverError(let code): return "Server Error \(code)"
        case .unknown(let error): return "ข้อผิดพลาดที่ไม่รู้จัก: \(error.localizedDescription)"
        }
    }
}

var cancellables: Set<AnyCancellable> = []

// mapError - แปลง Error เป็น Error ชนิดอื่น
URLSession.shared.dataTaskPublisher(for: URL(string: "https://api.example.com")!)
    .mapError { NetworkError.unknown($0) }
    .sink(
        receiveCompletion: { completion in
            if case .failure(let error) = completion {
                print("Error: \(error.localizedDescription)")
            }
        },
        receiveValue: { _ in }
    )
    .store(in: &cancellables)

// catch - แทน error ด้วย Publisher ใหม่
let subject = PassthroughSubject<Int, NetworkError>()

subject
    .catch { error -> AnyPublisher<Int, Never> in
        print("จัดการ error: \(error)")
        return Just(-1).eraseToAnyPublisher()
    }
    .sink { print($0) }
    .store(in: &cancellables)

subject.send(1)
subject.send(completion: .failure(.noData))
// -1 จะถูกส่งแทน

// tryCatch - catch ที่อาจ throw error ใหม่
subject
    .tryCatch { error -> AnyPublisher<Int, Error> in
        if case .serverError(let code) = error, code == 503 {
            throw NetworkError.serverError(503)
        }
        return Just(0).setFailureType(to: Error.self).eraseToAnyPublisher()
    }
    .sink(
        receiveCompletion: { _ in },
        receiveValue: { print($0) }
    )
    .store(in: &cancellables)

// assertNoFailure - crash ถ้ามี error (ใช้สำหรับ testing)
Just(42)
    .setFailureType(to: Error.self)
    .assertNoFailure("ไม่ควรมี error ตรงนี้")
    .sink { print($0) }
    .store(in: &cancellables)
```

---

## 16. Memory Management ใน Combine

```swift
import Combine
import Foundation

// ปัญหา Retain Cycle
class BadViewModel {
    var cancellables: Set<AnyCancellable> = []
    var name = "Bad"
    
    init() {
        Timer.publish(every: 1, on: .main, in: .common)
            .autoconnect()
            .sink { [self] _ in  // ⚠️ Retain cycle: self retain cancellables, cancellables retain self
                print(self.name)
            }
            .store(in: &cancellables)
    }
}

// วิธีถูกต้อง: ใช้ [weak self]
class GoodViewModel {
    var cancellables: Set<AnyCancellable> = []
    var name = "Good"
    
    init() {
        Timer.publish(every: 1, on: .main, in: .common)
            .autoconnect()
            .sink { [weak self] _ in  // ✅ ไม่มี retain cycle
                guard let self = self else { return }
                print(self.name)
            }
            .store(in: &cancellables)
    }
    
    deinit {
        print("GoodViewModel deallocated")
    }
}

// การจัดการ lifecycle
class ViewModelWithLifecycle: ObservableObject {
    private var cancellables: Set<AnyCancellable> = []
    
    @Published var data: [String] = []
    
    func startListening() {
        // เริ่ม subscription เมื่อ view appear
        someDataPublisher
            .sink { [weak self] newData in
                self?.data = newData
            }
            .store(in: &cancellables)
    }
    
    func stopListening() {
        // หยุด subscription เมื่อ view disappear
        cancellables.removeAll()
    }
    
    private var someDataPublisher: AnyPublisher<[String], Never> {
        Just(["ข้อมูล 1", "ข้อมูล 2"]).eraseToAnyPublisher()
    }
}

// เก็บ subscription แบบต่างๆ
class SubscriptionManagement {
    // 1. เก็บใน Set
    private var cancellables: Set<AnyCancellable> = []
    
    // 2. เก็บแบบ optional สำหรับยกเลิกง่าย
    private var timerCancellable: AnyCancellable?
    
    func startTimer() {
        timerCancellable = Timer.publish(every: 1, on: .main, in: .common)
            .autoconnect()
            .sink { _ in print("tick") }
    }
    
    func stopTimer() {
        timerCancellable?.cancel()
        timerCancellable = nil
    }
    
    // 3. เก็บใน array
    private var subscriptions: [AnyCancellable] = []
    
    func addSubscription() {
        Just("test")
            .sink { print($0) }
            .store(in: &cancellables)
    }
    
    func cancelAll() {
        cancellables.removeAll()
        subscriptions.removeAll()
    }
}
```

---

## 17. Testing Combine Publishers

```swift
import XCTest
import Combine

// Testing synchronous publishers
class SynchronousPublisherTests: XCTestCase {
    var cancellables: Set<AnyCancellable> = []
    
    func testMapOperator() {
        var receivedValues: [Int] = []
        
        [1, 2, 3].publisher
            .map { $0 * 2 }
            .sink { receivedValues.append($0) }
            .store(in: &cancellables)
        
        XCTAssertEqual(receivedValues, [2, 4, 6])
    }
    
    func testFilterOperator() {
        var receivedValues: [Int] = []
        
        (1...10).publisher
            .filter { $0.isMultiple(of: 2) }
            .sink { receivedValues.append($0) }
            .store(in: &cancellables)
        
        XCTAssertEqual(receivedValues, [2, 4, 6, 8, 10])
    }
}

// Testing asynchronous publishers
class AsyncPublisherTests: XCTestCase {
    var cancellables: Set<AnyCancellable> = []
    
    func testFuturePublisher() {
        let expectation = expectation(description: "Future should complete")
        var result: String?
        
        let future = Future<String, Never> { promise in
            DispatchQueue.global().asyncAfter(deadline: .now() + 0.1) {
                promise(.success("สำเร็จ"))
            }
        }
        
        future
            .sink { value in
                result = value
                expectation.fulfill()
            }
            .store(in: &cancellables)
        
        waitForExpectations(timeout: 1.0)
        XCTAssertEqual(result, "สำเร็จ")
    }
    
    func testDebounce() {
        let expectation = expectation(description: "Debounce should fire once")
        expectation.expectedFulfillmentCount = 1
        
        let subject = PassthroughSubject<String, Never>()
        var count = 0
        
        subject
            .debounce(for: .milliseconds(100), scheduler: DispatchQueue.main)
            .sink { _ in
                count += 1
                expectation.fulfill()
            }
            .store(in: &cancellables)
        
        // ส่งหลายค่าอย่างรวดเร็ว
        subject.send("a")
        subject.send("ab")
        subject.send("abc")
        
        waitForExpectations(timeout: 1.0)
        XCTAssertEqual(count, 1) // ควรได้รับแค่ครั้งเดียว
    }
}

// Testing with PassthroughSubject
class SubjectTests: XCTestCase {
    var cancellables: Set<AnyCancellable> = []
    
    func testPassthroughSubject() {
        let subject = PassthroughSubject<Int, Never>()
        var values: [Int] = []
        var completed = false
        
        subject
            .sink(
                receiveCompletion: { _ in completed = true },
                receiveValue: { values.append($0) }
            )
            .store(in: &cancellables)
        
        subject.send(1)
        subject.send(2)
        subject.send(3)
        subject.send(completion: .finished)
        
        XCTAssertEqual(values, [1, 2, 3])
        XCTAssertTrue(completed)
    }
}
```

---

## 18. แบบฝึกหัดเชิงปฏิบัติ

### แบบฝึกหัดที่ 1: Search กับ Debounce

```swift
import SwiftUI
import Combine

struct Product: Identifiable, Codable {
    let id: Int
    let title: String
    let price: Double
    let category: String
    let description: String
}

class ProductSearchViewModel: ObservableObject {
    @Published var searchText = ""
    @Published var products: [Product] = []
    @Published var isLoading = false
    @Published var errorMessage: String?
    @Published var selectedCategory: String = "ทั้งหมด"
    
    private var allProducts: [Product] = []
    private var cancellables: Set<AnyCancellable> = []
    
    let categories = ["ทั้งหมด", "electronics", "jewelery", "men's clothing", "women's clothing"]
    
    init() {
        loadProducts()
        setupSearch()
    }
    
    private func loadProducts() {
        isLoading = true
        
        URLSession.shared.dataTaskPublisher(for: URL(string: "https://fakestoreapi.com/products")!)
            .map(\.data)
            .decode(type: [Product].self, decoder: JSONDecoder())
            .receive(on: DispatchQueue.main)
            .sink(
                receiveCompletion: { [weak self] completion in
                    self?.isLoading = false
                    if case .failure(let error) = completion {
                        self?.errorMessage = error.localizedDescription
                    }
                },
                receiveValue: { [weak self] products in
                    self?.allProducts = products
                    self?.products = products
                }
            )
            .store(in: &cancellables)
    }
    
    private func setupSearch() {
        // รวม searchText และ selectedCategory ด้วย combineLatest
        Publishers.CombineLatest($searchText, $selectedCategory)
            .debounce(for: .milliseconds(300), scheduler: DispatchQueue.main)
            .map { [weak self] query, category -> [Product] in
                guard let self = self else { return [] }
                
                var filtered = self.allProducts
                
                // กรองตาม category
                if category != "ทั้งหมด" {
                    filtered = filtered.filter { $0.category == category }
                }
                
                // กรองตาม search text
                if !query.isEmpty {
                    filtered = filtered.filter { product in
                        product.title.localizedCaseInsensitiveContains(query) ||
                        product.description.localizedCaseInsensitiveContains(query)
                    }
                }
                
                return filtered
            }
            .assign(to: &$products)
    }
}

struct ProductSearchView: View {
    @StateObject private var viewModel = ProductSearchViewModel()
    
    var body: some View {
        NavigationView {
            VStack {
                // Category Picker
                ScrollView(.horizontal, showsIndicators: false) {
                    HStack {
                        ForEach(viewModel.categories, id: \.self) { category in
                            Button(category) {
                                viewModel.selectedCategory = category
                            }
                            .buttonStyle(.bordered)
                            .tint(viewModel.selectedCategory == category ? .blue : .gray)
                        }
                    }
                    .padding(.horizontal)
                }
                
                if viewModel.isLoading {
                    ProgressView("กำลังโหลด...")
                        .frame(maxWidth: .infinity, maxHeight: .infinity)
                } else if let error = viewModel.errorMessage {
                    Text("Error: \(error)")
                        .foregroundColor(.red)
                        .padding()
                } else {
                    List(viewModel.products) { product in
                        VStack(alignment: .leading) {
                            Text(product.title)
                                .font(.headline)
                                .lineLimit(2)
                            Text("$\(String(format: "%.2f", product.price))")
                                .foregroundColor(.green)
                            Text(product.category)
                                .font(.caption)
                                .foregroundColor(.secondary)
                        }
                    }
                }
            }
            .searchable(text: $viewModel.searchText, prompt: "ค้นหาสินค้า...")
            .navigationTitle("สินค้า (\(viewModel.products.count))")
        }
    }
}
```

### แบบฝึกหัดที่ 2: Form Validation

```swift
import SwiftUI
import Combine

class RegistrationViewModel: ObservableObject {
    @Published var name = ""
    @Published var email = ""
    @Published var password = ""
    @Published var confirmPassword = ""
    @Published var phoneNumber = ""
    
    @Published var nameError: String?
    @Published var emailError: String?
    @Published var passwordError: String?
    @Published var confirmPasswordError: String?
    @Published var phoneError: String?
    @Published var isFormValid = false
    
    @Published var isSubmitting = false
    @Published var submitSuccess = false
    
    private var cancellables: Set<AnyCancellable> = []
    
    init() {
        setupValidation()
    }
    
    private func setupValidation() {
        // Validate name
        $name
            .debounce(for: .milliseconds(500), scheduler: DispatchQueue.main)
            .map { name -> String? in
                if name.isEmpty { return nil }
                if name.count < 2 { return "ชื่อต้องมีอย่างน้อย 2 ตัวอักษร" }
                if name.count > 50 { return "ชื่อต้องไม่เกิน 50 ตัวอักษร" }
                return nil
            }
            .assign(to: &$nameError)
        
        // Validate email
        $email
            .debounce(for: .milliseconds(500), scheduler: DispatchQueue.main)
            .map { email -> String? in
                if email.isEmpty { return nil }
                let emailRegex = "[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}"
                let predicate = NSPredicate(format: "SELF MATCHES %@", emailRegex)
                return predicate.evaluate(with: email) ? nil : "รูปแบบอีเมลไม่ถูกต้อง"
            }
            .assign(to: &$emailError)
        
        // Validate password
        $password
            .debounce(for: .milliseconds(500), scheduler: DispatchQueue.main)
            .map { password -> String? in
                if password.isEmpty { return nil }
                if password.count < 8 { return "รหัสผ่านต้องมีอย่างน้อย 8 ตัวอักษร" }
                let hasUppercase = password.range(of: "[A-Z]", options: .regularExpression) != nil
                let hasNumber = password.range(of: "[0-9]", options: .regularExpression) != nil
                if !hasUppercase { return "รหัสผ่านต้องมีตัวพิมพ์ใหญ่อย่างน้อย 1 ตัว" }
                if !hasNumber { return "รหัสผ่านต้องมีตัวเลขอย่างน้อย 1 ตัว" }
                return nil
            }
            .assign(to: &$passwordError)
        
        // Validate confirm password
        Publishers.CombineLatest($password, $confirmPassword)
            .debounce(for: .milliseconds(500), scheduler: DispatchQueue.main)
            .map { password, confirm -> String? in
                if confirm.isEmpty { return nil }
                return password == confirm ? nil : "รหัสผ่านไม่ตรงกัน"
            }
            .assign(to: &$confirmPasswordError)
        
        // Validate phone
        $phoneNumber
            .debounce(for: .milliseconds(500), scheduler: DispatchQueue.main)
            .map { phone -> String? in
                if phone.isEmpty { return nil }
                let cleaned = phone.replacingOccurrences(of: "[^0-9]", with: "", options: .regularExpression)
                if cleaned.count != 10 { return "เบอร์โทรต้องมี 10 หลัก" }
                return nil
            }
            .assign(to: &$phoneError)
        
        // Check if form is valid
        Publishers.CombineLatest4($name, $email, $password, $confirmPassword)
            .combineLatest($phoneNumber)
            .map { [weak self] combined, phone in
                let (name, email, password, confirm) = combined
                guard let self = self else { return false }
                
                return !name.isEmpty && !email.isEmpty && !password.isEmpty &&
                       !confirm.isEmpty && !phone.isEmpty &&
                       self.nameError == nil && self.emailError == nil &&
                       self.passwordError == nil && self.confirmPasswordError == nil &&
                       self.phoneError == nil
            }
            .assign(to: &$isFormValid)
    }
    
    func submit() {
        guard isFormValid else { return }
        
        isSubmitting = true
        
        // จำลองการส่งข้อมูล
        Just(())
            .delay(for: .seconds(2), scheduler: DispatchQueue.main)
            .sink { [weak self] _ in
                self?.isSubmitting = false
                self?.submitSuccess = true
            }
            .store(in: &cancellables)
    }
}

struct RegistrationView: View {
    @StateObject private var viewModel = RegistrationViewModel()
    
    var body: some View {
        NavigationView {
            Form {
                Section("ข้อมูลส่วนตัว") {
                    ValidatedTextField(
                        title: "ชื่อ-นามสกุล",
                        text: $viewModel.name,
                        error: viewModel.nameError
                    )
                    
                    ValidatedTextField(
                        title: "อีเมล",
                        text: $viewModel.email,
                        error: viewModel.emailError,
                        keyboardType: .emailAddress
                    )
                    
                    ValidatedTextField(
                        title: "เบอร์โทรศัพท์",
                        text: $viewModel.phoneNumber,
                        error: viewModel.phoneError,
                        keyboardType: .phonePad
                    )
                }
                
                Section("รหัสผ่าน") {
                    ValidatedTextField(
                        title: "รหัสผ่าน",
                        text: $viewModel.password,
                        error: viewModel.passwordError,
                        isSecure: true
                    )
                    
                    ValidatedTextField(
                        title: "ยืนยันรหัสผ่าน",
                        text: $viewModel.confirmPassword,
                        error: viewModel.confirmPasswordError,
                        isSecure: true
                    )
                }
                
                Section {
                    Button(action: viewModel.submit) {
                        HStack {
                            if viewModel.isSubmitting {
                                ProgressView()
                                    .progressViewStyle(CircularProgressViewStyle(tint: .white))
                            }
                            Text(viewModel.isSubmitting ? "กำลังสมัคร..." : "สมัครสมาชิก")
                        }
                        .frame(maxWidth: .infinity)
                    }
                    .buttonStyle(.borderedProminent)
                    .disabled(!viewModel.isFormValid || viewModel.isSubmitting)
                }
            }
            .navigationTitle("สมัครสมาชิก")
            .alert("สมัครสมาชิกสำเร็จ!", isPresented: $viewModel.submitSuccess) {
                Button("ตกลง") {}
            } message: {
                Text("ยินดีต้อนรับสู่ระบบ")
            }
        }
    }
}

struct ValidatedTextField: View {
    let title: String
    @Binding var text: String
    var error: String?
    var keyboardType: UIKeyboardType = .default
    var isSecure: Bool = false
    
    var body: some View {
        VStack(alignment: .leading, spacing: 4) {
            if isSecure {
                SecureField(title, text: $text)
            } else {
                TextField(title, text: $text)
                    .keyboardType(keyboardType)
                    .autocapitalization(.none)
            }
            
            if let error = error {
                Text(error)
                    .font(.caption)
                    .foregroundColor(.red)
            }
        }
    }
}
```

---

## 19. สรุป (Summary)

ในบทนี้เราได้เรียนรู้ Combine Framework อย่างครบถ้วน:

### Concepts หลัก

| Concept | คำอธิบาย |
|---------|-----------|
| **Publisher** | ส่งข้อมูลเมื่อมีการเปลี่ยนแปลง |
| **Subscriber** | รับและประมวลผลข้อมูล |
| **Subscription** | การเชื่อมระหว่าง Publisher และ Subscriber |
| **Operator** | แปลงหรือกรองข้อมูลระหว่างทาง |
| **Scheduler** | กำหนด thread สำหรับ pipeline |

### Built-in Publishers

| Publisher | การใช้งาน |
|-----------|-----------|
| `Just` | ส่งค่าเดียว |
| `Future` | ผลลัพธ์ async เดียว |
| `Fail` | ส่ง error ทันที |
| `Empty` | ไม่มีค่า |
| `Deferred` | สร้าง Publisher เมื่อ subscribe |
| `PassthroughSubject` | ส่งค่าต่อไปโดยไม่เก็บ |
| `CurrentValueSubject` | เก็บและส่งค่าปัจจุบัน |

### Operators สำคัญ

- **แปลงข้อมูล**: `map`, `flatMap`, `compactMap`, `tryMap`
- **กรองข้อมูล**: `filter`, `removeDuplicates`, `first`, `last`
- **รวม Publisher**: `merge`, `zip`, `combineLatest`
- **จัดการเวลา**: `debounce`, `throttle`, `delay`
- **จัดการ Error**: `catch`, `replaceError`, `retry`
- **รวบรวมข้อมูล**: `reduce`, `scan`, `collect`

### Best Practices

1. ใช้ `[weak self]` ใน closure เพื่อป้องกัน retain cycle
2. เก็บ `AnyCancellable` ใน `Set<AnyCancellable>` เสมอ
3. ใช้ `receive(on: DispatchQueue.main)` ก่อนอัพเดต UI
4. ใช้ `debounce` สำหรับ search หรือ input ที่เปลี่ยนบ่อย
5. ใช้ `eraseToAnyPublisher()` สำหรับ API ที่เปิดเผยต่อภายนอก

### Combine กับ SwiftUI

```
@Published + ObservableObject + Combine =
    Reactive UI ที่ทรงพลัง
```

---

## แบบฝึกหัดเพิ่มเติม

### แบบฝึกหัดท้าทาย 1: Real-time Chat

สร้างระบบ chat ด้วย Combine:
- ใช้ PassthroughSubject รับข้อความ
- แสดงข้อความใน list
- จำกัดจำนวนข้อความที่แสดง (50 ล่าสุด)
- เพิ่ม typing indicator ด้วย debounce

### แบบฝึกหัดท้าทาย 2: Stock Price Monitor

สร้าง monitor ราคาหุ้น:
- จำลองราคาหุ้นด้วย Timer publisher
- แสดงกราฟราคา
- Alert เมื่อราคาเกินหรือต่ำกว่า threshold
- ใช้ combineLatest รวมข้อมูลหลายหุ้น

### แบบฝึกหัดท้าทาย 3: Infinite Scroll

สร้าง list แบบ infinite scroll:
- โหลดข้อมูลหน้าแรกตอนเริ่ม
- โหลดหน้าถัดไปเมื่อ scroll ถึงล่าง
- จัดการ loading state
- จัดการ error และ retry

---

*จบบท Part 28: Combine Framework*
