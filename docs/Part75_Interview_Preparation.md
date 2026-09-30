# Part 75: การเตรียมตัวสัมภาษณ์งาน iOS Developer

## บทนำ

การสัมภาษณ์งานสำหรับตำแหน่ง iOS Developer นั้นต้องการการเตรียมตัวอย่างรอบด้าน ทั้งความรู้ด้านเทคนิค Swift, iOS frameworks, system design, และทักษะการสื่อสาร ในบทนี้เราจะครอบคลุมทุกด้านที่จำเป็นสำหรับการสัมภาษณ์งานให้ประสบความสำเร็จ

---

## 1. ภาพรวมกระบวนการสัมภาษณ์ iOS Developer

### ขั้นตอนทั่วไปของการสัมภาษณ์

```
1. การยื่นใบสมัคร (Application)
   ↓
2. การคัดกรองเบื้องต้น (Initial Screening) - HR/Recruiter
   ↓
3. การสัมภาษณ์ทางโทรศัพท์/วิดีโอ (Phone/Video Screen)
   ↓
4. การทดสอบทักษะเทคนิค (Technical Assessment)
   - Take-home project หรือ
   - Coding challenge
   ↓
5. การสัมภาษณ์เชิงเทคนิค (Technical Interview)
   - Swift/iOS fundamentals
   - Data structures & algorithms
   - System design
   ↓
6. การสัมภาษณ์พฤติกรรม (Behavioral Interview)
   ↓
7. การสัมภาษณ์กับทีม (Team/Cultural Fit)
   ↓
8. ข้อเสนอ (Offer)
```

### ระยะเวลาที่คาดหวัง

- **Startup**: 1-2 สัปดาห์, 2-3 รอบ
- **บริษัทขนาดกลาง**: 2-4 สัปดาห์, 3-4 รอบ
- **บริษัทใหญ่/FAANG**: 4-8 สัปดาห์, 5-7 รอบ

---

## 2. ประเภทของการสัมภาษณ์

### 2.1 Initial Screening (HR Screening)

สิ่งที่ HR มักถาม:
- ประสบการณ์การทำงานโดยรวม
- เหตุผลที่ต้องการเปลี่ยนงาน
- ความคาดหวังด้านเงินเดือน
- ความพร้อมในการเริ่มงาน
- ตำแหน่งที่สนใจและเป้าหมายอาชีพ

**เคล็ดลับ**: เตรียม elevator pitch ประมาณ 2-3 นาทีเกี่ยวกับตัวเองและประสบการณ์

### 2.2 Technical Phone Screen

มักใช้เวลา 30-60 นาที ครอบคลุม:
- คำถามพื้นฐาน Swift
- ปัญหา coding แบบง่าย (LeetCode Easy/Medium)
- การอธิบายโปรเจกต์ที่เคยทำ

### 2.3 Technical Onsite Interview

มักใช้เวลา 4-6 ชั่วโมง แบ่งเป็นหลายรอบ:
- **Coding Round**: Data structures, algorithms
- **iOS Fundamentals**: Swift, frameworks, architecture
- **System Design**: Mobile architecture, API design
- **Behavioral**: STAR method questions

### 2.4 Take-home Project

- มักใช้เวลา 3-7 วัน
- ต้องแสดงทักษะการออกแบบและเขียนโค้ดที่ดี
- ควรมี README, unit tests, และ clean code

---

## 3. คำถามสัมภาษณ์ Swift และ iOS ที่พบบ่อย (50+ ข้อ)

### หมวด 3.1: Memory Management และ ARC

#### คำถามที่ 1: ARC คืออะไร และทำงานอย่างไร?

**คำตอบ**:
ARC (Automatic Reference Counting) คือระบบจัดการหน่วยความจำของ Swift ที่นับจำนวน reference ไปยัง object แต่ละตัว เมื่อ reference count เป็น 0 object จะถูกลบออกจากหน่วยความจำโดยอัตโนมัติ

```swift
class Person {
    let name: String
    
    init(name: String) {
        self.name = name
        print("\(name) ถูกสร้าง")
    }
    
    deinit {
        print("\(name) ถูกลบออกจากหน่วยความจำ")
    }
}

// Reference count = 1
var person1: Person? = Person(name: "สมชาย")

// Reference count = 2
var person2 = person1

// Reference count = 1
person1 = nil

// Reference count = 0 → deinit ถูกเรียก
person2 = nil
// Output: "สมชาย ถูกลบออกจากหน่วยความจำ"
```

**ข้อดีของ ARC**:
- ไม่ต้องจัดการหน่วยความจำด้วยตนเอง
- ไม่มี Garbage Collection (ไม่มี pause)
- Performance คาดเดาได้

#### คำถามที่ 2: อธิบายความแตกต่างระหว่าง strong, weak, และ unowned

**คำตอบ**:

```swift
class Owner {
    var name: String
    var pet: Pet?
    
    init(name: String) {
        self.name = name
    }
    
    deinit {
        print("\(name) ถูกลบ")
    }
}

class Pet {
    var name: String
    weak var owner: Owner?  // weak เพื่อป้องกัน retain cycle
    
    init(name: String) {
        self.name = name
    }
    
    deinit {
        print("\(name) ถูกลบ")
    }
}

var owner: Owner? = Owner(name: "สมชาย")
var pet: Pet? = Pet(name: "บิงโก")

owner?.pet = pet
pet?.owner = owner  // weak reference ไม่เพิ่ม retain count

owner = nil  // owner ถูกลบ เพราะ pet ไม่ได้ถือ strong reference
pet = nil    // pet ถูกลบ
```

**Strong** (ค่าเริ่มต้น):
- เพิ่ม reference count
- Object จะไม่ถูกลบตราบใดที่มี strong reference

**Weak**:
- ไม่เพิ่ม reference count
- ต้องเป็น Optional
- กลายเป็น nil โดยอัตโนมัติเมื่อ object ถูกลบ
- ใช้เมื่อ reference อาจเป็น nil

**Unowned**:
- ไม่เพิ่ม reference count
- ไม่เป็น Optional (ไม่กลายเป็น nil อัตโนมัติ)
- ใช้เมื่อ reference จะไม่เป็น nil ตลอดอายุการใช้งาน
- ถ้า access หลัง object ถูกลบ → crash

```swift
class BankAccount {
    let customer: Customer
    
    init(customer: Customer) {
        self.customer = customer
    }
}

class Customer {
    var name: String
    var account: BankAccount?
    
    init(name: String) {
        self.name = name
    }
}

// unowned: Customer จะมีชีวิตอยู่นานกว่า BankAccount เสมอ
class BankCard {
    let cardNumber: String
    unowned let customer: Customer  // ไม่เพิ่ม retain count
    
    init(cardNumber: String, customer: Customer) {
        self.cardNumber = cardNumber
        self.customer = customer
    }
}
```

#### คำถามที่ 3: Retain Cycle คืออะไร และวิธีแก้ไข?

**คำตอบ**:
Retain Cycle เกิดขึ้นเมื่อ objects สองตัวหรือมากกว่าถือ strong reference ซึ่งกันและกัน ทำให้ ARC ไม่สามารถลบพวกมันได้

```swift
// ปัญหา: Retain Cycle
class Parent {
    var child: Child?
    
    deinit {
        print("Parent ถูกลบ")
    }
}

class Child {
    var parent: Parent?  // strong reference → retain cycle!
    
    deinit {
        print("Child ถูกลบ")
    }
}

var parent: Parent? = Parent()
var child: Child? = Child()

parent?.child = child
child?.parent = parent  // สร้าง retain cycle

parent = nil  // deinit ไม่ถูกเรียก!
child = nil   // deinit ไม่ถูกเรียก! → Memory leak

// วิธีแก้ไข: ใช้ weak
class Child {
    weak var parent: Parent?  // แก้ไข retain cycle
    
    deinit {
        print("Child ถูกลบ")
    }
}
```

**Retain Cycle ใน Closure**:

```swift
class ViewController: UIViewController {
    var name = "iOS Developer"
    
    func badExample() {
        // ปัญหา: self ถูก capture แบบ strong
        let closure = {
            print(self.name)  // VC ไม่ถูกลบเพราะ closure ถือ self
        }
        // ... use closure
    }
    
    func goodExample() {
        // แก้ไข: ใช้ [weak self]
        let closure = { [weak self] in
            guard let self = self else { return }
            print(self.name)
        }
        // ... use closure
    }
    
    func alsoGoodExample() {
        // หรือใช้ [unowned self] ถ้าแน่ใจว่า self จะไม่เป็น nil
        let closure = { [unowned self] in
            print(self.name)
        }
    }
}
```

#### คำถามที่ 4: Memory leak ตรวจสอบและแก้ไขอย่างไร?

**คำตอบ**:
```swift
// การใช้ Instruments > Leaks
// หรือใช้ Xcode Memory Debugger

// วิธีตรวจสอบ retain cycle ด้วย deinit
class MyViewController: UIViewController {
    deinit {
        print("MyViewController ถูก deinit")
        // ถ้าข้อความนี้ไม่ปรากฏหลังจาก pop/dismiss = มี memory leak
    }
}

// ตรวจสอบ weak references ใน delegate pattern
protocol DataDelegate: AnyObject {
    func didReceiveData(_ data: String)
}

class DataManager {
    weak var delegate: DataDelegate?  // ต้องเป็น weak!
    
    func fetchData() {
        // ... fetch data
        delegate?.didReceiveData("some data")
    }
}
```

---

### หมวด 3.2: Optionals และ Nil

#### คำถามที่ 5: Optional คืออะไร และทำไมถึงสำคัญ?

**คำตอบ**:
Optional คือ type ที่สามารถมีค่าหรือไม่มีค่า (nil) ก็ได้ เป็นวิธีที่ Swift บังคับให้ developer จัดการกับกรณีที่ค่าอาจไม่มีอย่างชัดเจน

```swift
// Optional declaration
var name: String? = "สมชาย"
var age: Int? = nil

// วิธี unwrap Optional

// 1. Optional Binding (if let)
if let unwrappedName = name {
    print("ชื่อคือ: \(unwrappedName)")
} else {
    print("ไม่มีชื่อ")
}

// 2. Guard let (แนะนำในฟังก์ชัน)
func greet(name: String?) {
    guard let name = name else {
        print("ไม่มีชื่อ")
        return
    }
    print("สวัสดี \(name)")
}

// 3. Nil coalescing operator
let displayName = name ?? "ผู้ใช้นิรนาม"

// 4. Optional chaining
struct User {
    var address: Address?
}

struct Address {
    var city: String?
}

let user: User? = User(address: Address(city: "กรุงเทพฯ"))
let city = user?.address?.city  // ถ้าใดก็ตามเป็น nil ผลลัพธ์ = nil

// 5. Force unwrap (ใช้ระวัง!)
let forcedName = name!  // crash ถ้า name = nil
```

#### คำถามที่ 6: ImplicitlyUnwrappedOptional คืออะไร?

**คำตอบ**:
```swift
// Implicitly Unwrapped Optional ใช้ ! แทน ?
var label: UILabel!  // ใช้บ่อยใน IBOutlet

// ไม่ต้อง unwrap เมื่อใช้งาน
label.text = "Hello"  // ถ้า label = nil → crash

// ใช้กรณีไหน?
// 1. IBOutlets (สร้างโดย Interface Builder)
@IBOutlet weak var titleLabel: UILabel!

// 2. Properties ที่รู้ว่าจะมีค่าหลัง init
class MyClass {
    var value: Int!
    
    init() {
        setup()
    }
    
    func setup() {
        value = 42
    }
}
```

---

### หมวด 3.3: Value Types vs Reference Types

#### คำถามที่ 7: อธิบายความแตกต่างระหว่าง struct และ class

**คำตอบ**:

```swift
// STRUCT - Value Type
struct Point {
    var x: Double
    var y: Double
}

var point1 = Point(x: 1, y: 2)
var point2 = point1  // copy!

point2.x = 10
print(point1.x)  // 1 (ไม่เปลี่ยน)
print(point2.x)  // 10

// CLASS - Reference Type
class Location {
    var latitude: Double
    var longitude: Double
    
    init(lat: Double, lon: Double) {
        self.latitude = lat
        self.longitude = lon
    }
}

var loc1 = Location(lat: 13.7, lon: 100.5)
var loc2 = loc1  // reference! ชี้ไป object เดียวกัน

loc2.latitude = 0
print(loc1.latitude)  // 0 (เปลี่ยน!)
print(loc2.latitude)  // 0
```

**ความแตกต่างหลัก**:

| ลักษณะ | Struct | Class |
|--------|--------|-------|
| Type | Value | Reference |
| Inheritance | ไม่รองรับ | รองรับ |
| ARC | ไม่มี | มี |
| Mutability | ต้องมาร์ก mutating | ไม่ต้อง |
| deinit | ไม่มี | มี |
| Identity (===) | ไม่มี | มี |

**เมื่อไหร่ใช้อะไร**:
```swift
// ใช้ Struct เมื่อ:
// - ข้อมูลง่าย ๆ (Coordinate, Size, Color)
// - ต้องการ copy semantics
// - ใช้ใน SwiftUI (ส่วนใหญ่)
struct UserProfile {
    var name: String
    var email: String
    var age: Int
}

// ใช้ Class เมื่อ:
// - ต้องการ shared state
// - ต้องการ inheritance
// - ต้องการ deinit
// - ทำงานกับ Objective-C APIs
class NetworkManager {
    static let shared = NetworkManager()
    private init() {}
    
    func fetch(url: URL) {
        // ...
    }
}
```

#### คำถามที่ 8: Copy-on-Write คืออะไร?

**คำตอบ**:
```swift
// Copy-on-Write (CoW) - Swift collections ใช้เทคนิคนี้
// Copy จะเกิดขึ้นจริงเมื่อมีการแก้ไขเท่านั้น

var array1 = [1, 2, 3, 4, 5]
var array2 = array1  // ยังใช้ buffer เดียวกัน (ไม่ copy จริง)

array2.append(6)  // ตอนนี้ copy จริง
print(array1)  // [1, 2, 3, 4, 5]
print(array2)  // [1, 2, 3, 4, 5, 6]

// Custom CoW
final class Storage<T> {
    var items: [T]
    
    init(_ items: [T] = []) {
        self.items = items
    }
    
    init(copying other: Storage<T>) {
        self.items = other.items
    }
}

struct MyCollection<T> {
    private var storage = Storage<T>()
    
    mutating func append(_ item: T) {
        if !isKnownUniquelyReferenced(&storage) {
            storage = Storage(copying: storage)  // copy เมื่อจำเป็น
        }
        storage.items.append(item)
    }
}
```

---

### หมวด 3.4: Protocols และ Generics

#### คำถามที่ 9: Protocol-Oriented Programming คืออะไร?

**คำตอบ**:
```swift
// Protocol เป็น blueprint ของ methods, properties
protocol Drawable {
    func draw()
    var color: String { get }
}

protocol Resizable {
    mutating func resize(by factor: Double)
}

// Protocol Composition
protocol Shape: Drawable & Resizable {
    var area: Double { get }
}

struct Circle: Shape {
    var radius: Double
    var color: String
    
    var area: Double {
        return .pi * radius * radius
    }
    
    func draw() {
        print("วาดวงกลมสี \(color) รัศมี \(radius)")
    }
    
    mutating func resize(by factor: Double) {
        radius *= factor
    }
}

struct Rectangle: Shape {
    var width: Double
    var height: Double
    var color: String
    
    var area: Double {
        return width * height
    }
    
    func draw() {
        print("วาดสี่เหลี่ยม \(width)x\(height) สี \(color)")
    }
    
    mutating func resize(by factor: Double) {
        width *= factor
        height *= factor
    }
}

// Protocol Extension - default implementation
extension Drawable {
    func drawWithBorder() {
        print("วาดเส้นขอบ")
        draw()
    }
}

// Polymorphism ผ่าน protocol
func drawAll(shapes: [any Shape]) {
    for shape in shapes {
        shape.draw()
        print("พื้นที่: \(shape.area)")
    }
}

let shapes: [any Shape] = [
    Circle(radius: 5, color: "แดง"),
    Rectangle(width: 4, height: 6, color: "น้ำเงิน")
]
drawAll(shapes: shapes)
```

#### คำถามที่ 10: Generics ทำงานอย่างไร?

**คำตอบ**:
```swift
// Generic Function
func swapValues<T>(_ a: inout T, _ b: inout T) {
    let temp = a
    a = b
    b = temp
}

var x = 5, y = 10
swapValues(&x, &y)
print(x, y)  // 10 5

// Generic Type
struct Stack<Element> {
    private var items: [Element] = []
    
    mutating func push(_ item: Element) {
        items.append(item)
    }
    
    mutating func pop() -> Element? {
        return items.popLast()
    }
    
    var top: Element? {
        return items.last
    }
    
    var isEmpty: Bool {
        return items.isEmpty
    }
}

var stack = Stack<Int>()
stack.push(1)
stack.push(2)
stack.push(3)
print(stack.pop()!)  // 3

// Generic Constraints
func findMax<T: Comparable>(_ array: [T]) -> T? {
    guard !array.isEmpty else { return nil }
    return array.max()
}

// Protocol with Associated Type
protocol Container {
    associatedtype Item
    mutating func add(_ item: Item)
    func get(at index: Int) -> Item?
    var count: Int { get }
}

struct SimpleContainer<T>: Container {
    private var items: [T] = []
    
    mutating func add(_ item: T) {
        items.append(item)
    }
    
    func get(at index: Int) -> T? {
        guard index < items.count else { return nil }
        return items[index]
    }
    
    var count: Int { items.count }
}
```

---

### หมวด 3.5: Closures และ Capture Lists

#### คำถามที่ 11: Closure คืออะไร และมีกี่ประเภท?

**คำตอบ**:
```swift
// 1. Global Function - ชื่อ, ไม่ capture values
func add(_ a: Int, _ b: Int) -> Int {
    return a + b
}

// 2. Nested Function - ชื่อ, capture values จาก enclosing function
func makeAdder(for value: Int) -> (Int) -> Int {
    func adder(_ n: Int) -> Int {
        return n + value  // capture value จากภายนอก
    }
    return adder
}

// 3. Closure Expression - ไม่มีชื่อ
let multiply = { (a: Int, b: Int) -> Int in
    return a * b
}

// Closure Syntax Shorthand
let numbers = [3, 1, 4, 1, 5, 9, 2, 6]

// แบบยาว
let sorted1 = numbers.sorted(by: { (a: Int, b: Int) -> Bool in
    return a < b
})

// Inferring types
let sorted2 = numbers.sorted(by: { a, b in a < b })

// Shorthand arguments
let sorted3 = numbers.sorted(by: { $0 < $1 })

// Operator function
let sorted4 = numbers.sorted(by: <)

// Trailing Closure
let sorted5 = numbers.sorted { $0 < $1 }
```

#### คำถามที่ 12: Capture List คืออะไร?

**คำตอบ**:
```swift
// Closure capture values โดย reference ตามค่าเริ่มต้น
var counter = 0
let increment = {
    counter += 1
}
increment()
increment()
print(counter)  // 2 (capture by reference)

// Capture by value ด้วย capture list
var value = 10
let capturedClosure = { [value] in  // copy value ตอนสร้าง closure
    print(value)
}
value = 20
capturedClosure()  // print 10 (ค่าเดิมที่ capture ไว้)

// [weak self] เพื่อป้องกัน retain cycle
class ViewController: UIViewController {
    var data = "iOS"
    
    func loadData() {
        // Simulate async operation
        DispatchQueue.global().async { [weak self] in
            // self เป็น optional
            guard let self = self else { return }
            
            DispatchQueue.main.async {
                self.updateUI(with: self.data)
            }
        }
    }
    
    func updateUI(with data: String) {
        // Update UI
    }
}

// Escaping vs Non-escaping
func performNow(operation: () -> Void) {
    operation()  // non-escaping: ทำงานทันที
}

var completionHandlers: [() -> Void] = []
func storeCompletion(handler: @escaping () -> Void) {
    completionHandlers.append(handler)  // escaping: เก็บไว้ใช้ทีหลัง
}
```

---

### หมวด 3.6: Async/Await และ Concurrency

#### คำถามที่ 13: Swift Concurrency คืออะไร?

**คำตอบ**:
```swift
import Foundation

// เก่า: Completion Handler (Callback Hell)
func fetchUser(id: Int, completion: @escaping (Result<User, Error>) -> Void) {
    URLSession.shared.dataTask(with: URL(string: "https://api.example.com/users/\(id)")!) { data, response, error in
        if let error = error {
            completion(.failure(error))
            return
        }
        // ... parse data
        completion(.success(user))
    }.resume()
}

// ใหม่: async/await (Swift 5.5+)
func fetchUser(id: Int) async throws -> User {
    let url = URL(string: "https://api.example.com/users/\(id)")!
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode(User.self, from: data)
}

// การใช้งาน
Task {
    do {
        let user = try await fetchUser(id: 1)
        print("ได้ user: \(user.name)")
    } catch {
        print("เกิดข้อผิดพลาด: \(error)")
    }
}

// Concurrent tasks
async let user = fetchUser(id: 1)
async let posts = fetchPosts(userId: 1)
let (fetchedUser, fetchedPosts) = try await (user, posts)

// Task Group
func fetchMultipleUsers(ids: [Int]) async throws -> [User] {
    try await withThrowingTaskGroup(of: User.self) { group in
        for id in ids {
            group.addTask {
                try await fetchUser(id: id)
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

#### คำถามที่ 14: Actor คืออะไร?

**คำตอบ**:
```swift
// Actor ป้องกัน data race
actor BankAccount {
    private var balance: Double = 0
    
    func deposit(amount: Double) {
        balance += amount
    }
    
    func withdraw(amount: Double) throws {
        guard balance >= amount else {
            throw BankError.insufficientFunds
        }
        balance -= amount
    }
    
    func getBalance() -> Double {
        return balance
    }
}

enum BankError: Error {
    case insufficientFunds
}

// การใช้งาน
let account = BankAccount()

Task {
    await account.deposit(amount: 1000)
    try await account.withdraw(amount: 500)
    let balance = await account.getBalance()
    print("ยอดคงเหลือ: \(balance)")
}

// MainActor - ทำงานบน main thread
@MainActor
class ViewModel: ObservableObject {
    @Published var items: [Item] = []
    
    func loadData() async {
        let data = try? await fetchData()
        // อัปเดต UI บน main thread โดยอัตโนมัติ
        self.items = data ?? []
    }
}
```

#### คำถามที่ 15: GCD vs Swift Concurrency

**คำตอบ**:
```swift
// GCD (Grand Central Dispatch) - เก่า
DispatchQueue.global(qos: .background).async {
    let data = heavyComputation()
    
    DispatchQueue.main.async {
        self.updateUI(with: data)
    }
}

// Swift Concurrency - ใหม่ (Swift 5.5+)
Task {
    let data = await heavyComputation()  // runs in background
    await MainActor.run {
        self.updateUI(with: data)
    }
}

// หรือใช้ @MainActor annotation
@MainActor
func updateUI(with data: Data) {
    // guaranteed to run on main thread
}

// Structured Concurrency
func processImages(_ urls: [URL]) async throws -> [UIImage] {
    try await withThrowingTaskGroup(of: UIImage.self) { group in
        for url in urls {
            group.addTask {
                let (data, _) = try await URLSession.shared.data(from: url)
                guard let image = UIImage(data: data) else {
                    throw ImageError.invalidData
                }
                return image
            }
        }
        
        var images: [UIImage] = []
        for try await image in group {
            images.append(image)
        }
        return images
    }
}

enum ImageError: Error {
    case invalidData
}
```

---

### หมวด 3.7: SwiftUI vs UIKit

#### คำถามที่ 16: เปรียบเทียบ SwiftUI และ UIKit

**คำตอบ**:

```swift
// UIKit - Imperative (บอกวิธีทำ)
class ProfileViewController: UIViewController {
    private let nameLabel = UILabel()
    private let avatarImageView = UIImageView()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
    }
    
    private func setupUI() {
        nameLabel.font = .systemFont(ofSize: 20, weight: .bold)
        nameLabel.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(nameLabel)
        
        NSLayoutConstraint.activate([
            nameLabel.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            nameLabel.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 20)
        ])
    }
    
    func updateName(_ name: String) {
        nameLabel.text = name  // update manually
    }
}

// SwiftUI - Declarative (บอกว่าต้องการอะไร)
struct ProfileView: View {
    @State private var name: String = ""
    
    var body: some View {
        VStack {
            Text(name)
                .font(.title)
                .bold()
            
            TextField("ชื่อ", text: $name)
                .textFieldStyle(.roundedBorder)
        }
        .padding()
    }
}
```

**การเลือกใช้**:

| SwiftUI | UIKit |
|---------|-------|
| iOS 13+ | iOS 2+ |
| Declarative | Imperative |
| Less code | More control |
| Preview | Simulator |
| Data-driven | Manual updates |

#### คำถามที่ 17: App Lifecycle ใน SwiftUI vs UIKit

**คำตอบ**:
```swift
// UIKit App Lifecycle
@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
    
    func application(_ application: UIApplication, 
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        // แอปเริ่มต้น
        return true
    }
}

class SceneDelegate: UIResponder, UIWindowSceneDelegate {
    
    func sceneDidBecomeActive(_ scene: UIScene) {
        // Scene กลายเป็น active
    }
    
    func sceneWillResignActive(_ scene: UIScene) {
        // Scene กำลังจะหยุด active
    }
    
    func sceneDidEnterBackground(_ scene: UIScene) {
        // Scene เข้า background
    }
}

// SwiftUI App Lifecycle
@main
struct MyApp: App {
    @Environment(\.scenePhase) var scenePhase
    
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .onChange(of: scenePhase) { newPhase in
            switch newPhase {
            case .active:
                print("แอปกำลัง active")
            case .inactive:
                print("แอปกำลัง inactive")
            case .background:
                print("แอปอยู่ใน background")
            @unknown default:
                break
            }
        }
    }
}
```

---

### หมวด 3.8: Core Data

#### คำถามที่ 18: Core Data คืออะไร และใช้งานอย่างไร?

**คำตอบ**:
```swift
import CoreData

// Core Data Stack
class CoreDataStack {
    static let shared = CoreDataStack()
    
    lazy var persistentContainer: NSPersistentContainer = {
        let container = NSPersistentContainer(name: "MyApp")
        container.loadPersistentStores { _, error in
            if let error = error {
                fatalError("Core Data error: \(error)")
            }
        }
        return container
    }()
    
    var context: NSManagedObjectContext {
        return persistentContainer.viewContext
    }
    
    func saveContext() {
        let context = persistentContainer.viewContext
        if context.hasChanges {
            do {
                try context.save()
            } catch {
                print("Save error: \(error)")
            }
        }
    }
}

// Entity และ CRUD
// สมมติว่ามี Entity ชื่อ "Person" ใน .xcdatamodeld

// Create
func createPerson(name: String, age: Int16) {
    let context = CoreDataStack.shared.context
    let person = NSEntityDescription.insertNewObject(
        forEntityName: "Person",
        into: context
    )
    person.setValue(name, forKey: "name")
    person.setValue(age, forKey: "age")
    CoreDataStack.shared.saveContext()
}

// Read
func fetchPersons() -> [NSManagedObject] {
    let context = CoreDataStack.shared.context
    let fetchRequest = NSFetchRequest<NSManagedObject>(entityName: "Person")
    
    // เพิ่ม sort
    fetchRequest.sortDescriptors = [
        NSSortDescriptor(key: "name", ascending: true)
    ]
    
    // เพิ่ม filter
    fetchRequest.predicate = NSPredicate(format: "age > %d", 18)
    
    do {
        return try context.fetch(fetchRequest)
    } catch {
        print("Fetch error: \(error)")
        return []
    }
}

// SwiftUI + Core Data
struct PersonListView: View {
    @FetchRequest(
        sortDescriptors: [NSSortDescriptor(keyPath: \Person.name, ascending: true)]
    ) var persons: FetchedResults<Person>
    
    @Environment(\.managedObjectContext) var viewContext
    
    var body: some View {
        List(persons) { person in
            Text(person.name ?? "Unknown")
        }
        .toolbar {
            Button("เพิ่ม") {
                let person = Person(context: viewContext)
                person.name = "คนใหม่"
                try? viewContext.save()
            }
        }
    }
}
```

---

### หมวด 3.9: Networking

#### คำถามที่ 19: การทำ Networking ใน iOS

**คำตอบ**:
```swift
import Foundation

// Basic URLSession
struct NetworkManager {
    static let shared = NetworkManager()
    
    func fetch<T: Decodable>(from url: URL) async throws -> T {
        let (data, response) = try await URLSession.shared.data(from: url)
        
        guard let httpResponse = response as? HTTPURLResponse,
              (200...299).contains(httpResponse.statusCode) else {
            throw NetworkError.invalidResponse
        }
        
        return try JSONDecoder().decode(T.self, from: data)
    }
    
    func post<T: Codable>(to url: URL, body: T) async throws -> Data {
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        request.httpBody = try JSONEncoder().encode(body)
        
        let (data, _) = try await URLSession.shared.data(for: request)
        return data
    }
}

enum NetworkError: Error, LocalizedError {
    case invalidURL
    case invalidResponse
    case noData
    
    var errorDescription: String? {
        switch self {
        case .invalidURL: return "URL ไม่ถูกต้อง"
        case .invalidResponse: return "Response ไม่ถูกต้อง"
        case .noData: return "ไม่มีข้อมูล"
        }
    }
}

// Model
struct Post: Codable, Identifiable {
    let id: Int
    let title: String
    let body: String
}

// ViewModel
@MainActor
class PostsViewModel: ObservableObject {
    @Published var posts: [Post] = []
    @Published var isLoading = false
    @Published var error: String?
    
    func loadPosts() async {
        isLoading = true
        defer { isLoading = false }
        
        do {
            let url = URL(string: "https://jsonplaceholder.typicode.com/posts")!
            posts = try await NetworkManager.shared.fetch(from: url)
        } catch {
            self.error = error.localizedDescription
        }
    }
}
```

---

### หมวด 3.10: Testing

#### คำถามที่ 20: Unit Testing ใน Swift

**คำตอบ**:
```swift
import XCTest
@testable import MyApp

// Unit Test
class CalculatorTests: XCTestCase {
    
    var sut: Calculator!  // System Under Test
    
    override func setUp() {
        super.setUp()
        sut = Calculator()
    }
    
    override func tearDown() {
        sut = nil
        super.tearDown()
    }
    
    func testAddition() {
        let result = sut.add(2, 3)
        XCTAssertEqual(result, 5)
    }
    
    func testDivisionByZero() {
        XCTAssertThrowsError(try sut.divide(10, by: 0)) { error in
            XCTAssertEqual(error as? CalculatorError, .divisionByZero)
        }
    }
    
    // Async test
    func testFetchData() async throws {
        let viewModel = PostsViewModel()
        await viewModel.loadPosts()
        XCTAssertFalse(viewModel.posts.isEmpty)
    }
    
    // Performance test
    func testSortPerformance() {
        let numbers = Array(1...10000).shuffled()
        
        measure {
            _ = numbers.sorted()
        }
    }
}

// Mock
protocol NetworkServiceProtocol {
    func fetch(from url: URL) async throws -> Data
}

class MockNetworkService: NetworkServiceProtocol {
    var mockData: Data?
    var shouldThrow = false
    
    func fetch(from url: URL) async throws -> Data {
        if shouldThrow {
            throw NetworkError.invalidResponse
        }
        return mockData ?? Data()
    }
}

class ViewModelTests: XCTestCase {
    
    func testLoadPostsSuccess() async throws {
        let mockService = MockNetworkService()
        let mockPosts = [Post(id: 1, title: "Test", body: "Body")]
        mockService.mockData = try JSONEncoder().encode(mockPosts)
        
        let viewModel = PostsViewModel(networkService: mockService)
        await viewModel.loadPosts()
        
        XCTAssertEqual(viewModel.posts.count, 1)
        XCTAssertFalse(viewModel.isLoading)
    }
}
```

---

## 4. Data Structure Questions ใน Swift

### 4.1 Array

```swift
// คำถาม: Two Sum - หาคู่ตัวเลขที่รวมกันได้ตามเป้าหมาย
func twoSum(_ nums: [Int], _ target: Int) -> [Int] {
    var seen = [Int: Int]()  // value: index
    
    for (i, num) in nums.enumerated() {
        let complement = target - num
        if let j = seen[complement] {
            return [j, i]
        }
        seen[num] = i
    }
    
    return []
}

// Test
print(twoSum([2, 7, 11, 15], 9))  // [0, 1]
```

### 4.2 Linked List

```swift
// Linked List Implementation
class ListNode {
    var val: Int
    var next: ListNode?
    
    init(_ val: Int) {
        self.val = val
    }
}

class LinkedList {
    var head: ListNode?
    
    func append(_ val: Int) {
        let node = ListNode(val)
        if head == nil {
            head = node
            return
        }
        var current = head
        while current?.next != nil {
            current = current?.next
        }
        current?.next = node
    }
    
    // คำถาม: Reverse Linked List
    func reverse() -> ListNode? {
        var prev: ListNode? = nil
        var current = head
        
        while current != nil {
            let next = current?.next
            current?.next = prev
            prev = current
            current = next
        }
        
        return prev
    }
    
    // คำถาม: Detect Cycle
    func hasCycle() -> Bool {
        var slow = head
        var fast = head
        
        while fast != nil && fast?.next != nil {
            slow = slow?.next
            fast = fast?.next?.next
            
            if slow === fast {
                return true
            }
        }
        
        return false
    }
}
```

### 4.3 Stack และ Queue

```swift
// Stack
struct Stack<T> {
    private var elements: [T] = []
    
    mutating func push(_ element: T) {
        elements.append(element)
    }
    
    @discardableResult
    mutating func pop() -> T? {
        return elements.popLast()
    }
    
    var peek: T? {
        return elements.last
    }
    
    var isEmpty: Bool {
        return elements.isEmpty
    }
}

// Queue (ด้วย 2 stacks)
struct Queue<T> {
    private var inbox: [T] = []
    private var outbox: [T] = []
    
    mutating func enqueue(_ element: T) {
        inbox.append(element)
    }
    
    mutating func dequeue() -> T? {
        if outbox.isEmpty {
            outbox = inbox.reversed()
            inbox.removeAll()
        }
        return outbox.popLast()
    }
    
    var isEmpty: Bool {
        return inbox.isEmpty && outbox.isEmpty
    }
}

// Valid Parentheses
func isValid(_ s: String) -> Bool {
    var stack: [Character] = []
    let matching: [Character: Character] = [")": "(", "]": "[", "}": "{"]
    
    for char in s {
        if "([{".contains(char) {
            stack.append(char)
        } else if let open = matching[char] {
            if stack.last != open {
                return false
            }
            stack.removeLast()
        }
    }
    
    return stack.isEmpty
}

print(isValid("()[]{}"))  // true
print(isValid("(]"))       // false
```

### 4.4 Binary Tree

```swift
class TreeNode {
    var val: Int
    var left: TreeNode?
    var right: TreeNode?
    
    init(_ val: Int) {
        self.val = val
    }
}

// คำถาม: Maximum Depth of Binary Tree
func maxDepth(_ root: TreeNode?) -> Int {
    guard let root = root else { return 0 }
    return 1 + max(maxDepth(root.left), maxDepth(root.right))
}

// คำถาม: Inorder Traversal
func inorderTraversal(_ root: TreeNode?) -> [Int] {
    var result: [Int] = []
    
    func inorder(_ node: TreeNode?) {
        guard let node = node else { return }
        inorder(node.left)
        result.append(node.val)
        inorder(node.right)
    }
    
    inorder(root)
    return result
}

// คำถาม: Level Order Traversal (BFS)
func levelOrder(_ root: TreeNode?) -> [[Int]] {
    guard let root = root else { return [] }
    
    var result: [[Int]] = []
    var queue: [TreeNode] = [root]
    
    while !queue.isEmpty {
        let levelSize = queue.count
        var level: [Int] = []
        
        for _ in 0..<levelSize {
            let node = queue.removeFirst()
            level.append(node.val)
            
            if let left = node.left { queue.append(left) }
            if let right = node.right { queue.append(right) }
        }
        
        result.append(level)
    }
    
    return result
}
```

### 4.5 Hash Map

```swift
// คำถาม: Group Anagrams
func groupAnagrams(_ strs: [String]) -> [[String]] {
    var groups = [String: [String]]()
    
    for str in strs {
        let key = String(str.sorted())
        groups[key, default: []].append(str)
    }
    
    return Array(groups.values)
}

print(groupAnagrams(["eat", "tea", "tan", "ate", "nat", "bat"]))
// [["eat", "tea", "ate"], ["tan", "nat"], ["bat"]]

// คำถาม: Top K Frequent Elements
func topKFrequent(_ nums: [Int], _ k: Int) -> [Int] {
    var freq = [Int: Int]()
    for num in nums {
        freq[num, default: 0] += 1
    }
    
    return freq.sorted { $0.value > $1.value }
               .prefix(k)
               .map { $0.key }
}
```

---

## 5. Algorithm Questions ใน Swift

### 5.1 Sorting

```swift
// Quick Sort
func quickSort(_ arr: inout [Int], _ low: Int, _ high: Int) {
    if low < high {
        let pivot = partition(&arr, low, high)
        quickSort(&arr, low, pivot - 1)
        quickSort(&arr, pivot + 1, high)
    }
}

func partition(_ arr: inout [Int], _ low: Int, _ high: Int) -> Int {
    let pivot = arr[high]
    var i = low - 1
    
    for j in low..<high {
        if arr[j] <= pivot {
            i += 1
            arr.swapAt(i, j)
        }
    }
    arr.swapAt(i + 1, high)
    return i + 1
}

// Merge Sort
func mergeSort(_ arr: [Int]) -> [Int] {
    if arr.count <= 1 { return arr }
    
    let mid = arr.count / 2
    let left = mergeSort(Array(arr[..<mid]))
    let right = mergeSort(Array(arr[mid...]))
    
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
```

### 5.2 Binary Search

```swift
// Binary Search
func binarySearch(_ arr: [Int], _ target: Int) -> Int {
    var left = 0, right = arr.count - 1
    
    while left <= right {
        let mid = left + (right - left) / 2
        
        if arr[mid] == target {
            return mid
        } else if arr[mid] < target {
            left = mid + 1
        } else {
            right = mid - 1
        }
    }
    
    return -1
}

// Search in Rotated Sorted Array
func searchRotated(_ nums: [Int], _ target: Int) -> Int {
    var left = 0, right = nums.count - 1
    
    while left <= right {
        let mid = left + (right - left) / 2
        
        if nums[mid] == target { return mid }
        
        if nums[left] <= nums[mid] {
            if nums[left] <= target && target < nums[mid] {
                right = mid - 1
            } else {
                left = mid + 1
            }
        } else {
            if nums[mid] < target && target <= nums[right] {
                left = mid + 1
            } else {
                right = mid - 1
            }
        }
    }
    
    return -1
}
```

### 5.3 Dynamic Programming

```swift
// Fibonacci ด้วย memoization
func fibonacci(_ n: Int, memo: inout [Int: Int]) -> Int {
    if n <= 1 { return n }
    
    if let cached = memo[n] {
        return cached
    }
    
    let result = fibonacci(n - 1, memo: &memo) + fibonacci(n - 2, memo: &memo)
    memo[n] = result
    return result
}

// Climbing Stairs
func climbStairs(_ n: Int) -> Int {
    if n <= 2 { return n }
    
    var dp = Array(repeating: 0, count: n + 1)
    dp[1] = 1
    dp[2] = 2
    
    for i in 3...n {
        dp[i] = dp[i-1] + dp[i-2]
    }
    
    return dp[n]
}

// Longest Common Subsequence
func longestCommonSubsequence(_ text1: String, _ text2: String) -> Int {
    let s1 = Array(text1), s2 = Array(text2)
    let m = s1.count, n = s2.count
    
    var dp = Array(repeating: Array(repeating: 0, count: n + 1), count: m + 1)
    
    for i in 1...m {
        for j in 1...n {
            if s1[i-1] == s2[j-1] {
                dp[i][j] = dp[i-1][j-1] + 1
            } else {
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
            }
        }
    }
    
    return dp[m][n]
}
```

### 5.4 Graph Algorithms

```swift
// DFS
func dfs(_ graph: [[Int]], _ start: Int, visited: inout Set<Int>) {
    visited.insert(start)
    print(start, terminator: " ")
    
    for neighbor in graph[start] {
        if !visited.contains(neighbor) {
            dfs(graph, neighbor, visited: &visited)
        }
    }
}

// BFS
func bfs(_ graph: [[Int]], _ start: Int) {
    var visited = Set<Int>()
    var queue: [Int] = [start]
    visited.insert(start)
    
    while !queue.isEmpty {
        let node = queue.removeFirst()
        print(node, terminator: " ")
        
        for neighbor in graph[node] {
            if !visited.contains(neighbor) {
                visited.insert(neighbor)
                queue.append(neighbor)
            }
        }
    }
}

// Number of Islands
func numIslands(_ grid: [[Character]]) -> Int {
    var grid = grid
    var count = 0
    
    func dfs(_ r: Int, _ c: Int) {
        guard r >= 0 && r < grid.count && 
              c >= 0 && c < grid[0].count && 
              grid[r][c] == "1" else { return }
        
        grid[r][c] = "0"
        dfs(r+1, c); dfs(r-1, c)
        dfs(r, c+1); dfs(r, c-1)
    }
    
    for r in 0..<grid.count {
        for c in 0..<grid[0].count {
            if grid[r][c] == "1" {
                count += 1
                dfs(r, c)
            }
        }
    }
    
    return count
}
```

---

## 6. System Design สำหรับ Mobile

### 6.1 Design Instagram Feed

```
ระบบ Feed ของ Instagram

Components:
┌─────────────────────────────────────────────┐
│                Mobile Client                 │
│  - FeedViewController/FeedView               │
│  - Pagination (infinite scroll)              │
│  - Image caching (NSCache/disk)              │
│  - Optimistic updates                        │
└─────────────────┬───────────────────────────┘
                  │ HTTPS/REST/GraphQL
┌─────────────────▼───────────────────────────┐
│               API Gateway                    │
│  - Authentication (JWT/OAuth)                │
│  - Rate limiting                             │
│  - Load balancing                            │
└─────────────────┬───────────────────────────┘
                  │
┌─────────────────▼───────────────────────────┐
│              Feed Service                    │
│  - Ranked feed algorithm                     │
│  - Cursor-based pagination                   │
│  - Cache (Redis)                             │
└─────────────────────────────────────────────┘

ประเด็นสำคัญที่ต้องพูดถึง:

1. Pagination: cursor-based vs offset-based
2. Caching: CDN สำหรับรูปภาพ, Redis สำหรับ feed
3. Offline support: Core Data/local cache
4. Image loading: NSCache + disk cache
5. Real-time updates: WebSocket หรือ long polling
```

```swift
// Feed ViewModel
@MainActor
class FeedViewModel: ObservableObject {
    @Published var posts: [Post] = []
    @Published var isLoadingMore = false
    @Published var hasMore = true
    
    private var cursor: String?
    
    func loadInitial() async {
        do {
            let response = try await FeedAPI.fetch(cursor: nil, limit: 20)
            posts = response.posts
            cursor = response.nextCursor
            hasMore = response.hasMore
        } catch {
            // handle error
        }
    }
    
    func loadMore() async {
        guard !isLoadingMore && hasMore else { return }
        isLoadingMore = true
        defer { isLoadingMore = false }
        
        do {
            let response = try await FeedAPI.fetch(cursor: cursor, limit: 20)
            posts.append(contentsOf: response.posts)
            cursor = response.nextCursor
            hasMore = response.hasMore
        } catch {
            // handle error
        }
    }
}
```

### 6.2 Design Chat Application

```
ระบบ Chat

ประเด็นสำคัญ:

1. Real-time messaging: WebSocket
2. Message delivery states: sent → delivered → read
3. Offline support: local DB (Core Data/SQLite)
4. Media sharing: presigned URLs สำหรับ upload
5. Push notifications: APNs
6. End-to-end encryption

Architecture:
┌────────────────────────────────────────┐
│              iOS App                    │
│  ┌────────────┐  ┌──────────────────┐  │
│  │ ChatService │  │  LocalDB         │  │
│  │ WebSocket  │  │  (Core Data)     │  │
│  └────────────┘  └──────────────────┘  │
└────────────────────────────────────────┘
```

```swift
// WebSocket Manager
class ChatWebSocketManager: NSObject {
    private var webSocketTask: URLSessionWebSocketTask?
    private var pingTimer: Timer?
    
    func connect(to url: URL, token: String) {
        var request = URLRequest(url: url)
        request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        
        let session = URLSession(configuration: .default, delegate: self, delegateQueue: nil)
        webSocketTask = session.webSocketTask(with: request)
        webSocketTask?.resume()
        
        startListening()
        schedulePing()
    }
    
    func send(message: ChatMessage) async throws {
        let data = try JSONEncoder().encode(message)
        let string = String(data: data, encoding: .utf8)!
        try await webSocketTask?.send(.string(string))
    }
    
    private func startListening() {
        webSocketTask?.receive { [weak self] result in
            switch result {
            case .success(let message):
                self?.handleMessage(message)
                self?.startListening()  // continue listening
            case .failure(let error):
                self?.handleDisconnect(error: error)
            }
        }
    }
    
    private func handleMessage(_ message: URLSessionWebSocketTask.Message) {
        // decode and process message
    }
    
    private func schedulePing() {
        pingTimer = Timer.scheduledTimer(withTimeInterval: 30, repeats: true) { [weak self] _ in
            self?.webSocketTask?.sendPing { _ in }
        }
    }
    
    func disconnect() {
        pingTimer?.invalidate()
        webSocketTask?.cancel(with: .normalClosure, reason: nil)
    }
}
```

---

## 7. Behavioral Interview Tips (STAR Method)

### STAR Method

**S**ituation - อธิบายสถานการณ์
**T**ask - หน้าที่ความรับผิดชอบ
**A**ction - สิ่งที่คุณทำ
**R**esult - ผลลัพธ์

### ตัวอย่างคำถาม Behavioral

#### "บอกเล่าเกี่ยวกับเวลาที่คุณต้องแก้ปัญหา bug ที่ยาก"

**ตัวอย่างคำตอบ STAR**:

> **Situation**: ผมทำงานใน e-commerce app ที่มีผู้ใช้หลายล้านคน และเราพบว่าแอปจะ crash ในบางกรณีที่ไม่แน่นอน โดยเฉพาะในช่วง peak traffic
>
> **Task**: ผมรับผิดชอบในการหาสาเหตุและแก้ไข crash นี้ภายใน 48 ชั่วโมง เพราะมันกระทบต่อยอดขาย
>
> **Action**: ผมวิเคราะห์ crash logs ใน Firebase Crashlytics พบว่า crash เกิดจาก race condition ในโค้ดที่ update UI จาก background thread ผมใช้ Thread Sanitizer เพื่อยืนยัน จากนั้นแก้ไขด้วยการ dispatch ทุก UI update ไปที่ main thread และเพิ่ม unit tests
>
> **Result**: Crash rate ลดลง 99.8% crash ภายในวันเดียวหลัง deploy fix app rating เพิ่มจาก 3.2 เป็น 4.1 stars

#### คำถาม Behavioral ที่พบบ่อย

1. บอกเล่าเกี่ยวกับโปรเจกต์ที่คุณภูมิใจมากที่สุด
2. อธิบายสถานการณ์ที่คุณไม่เห็นด้วยกับเพื่อนร่วมทีม
3. เล่าถึงเวลาที่คุณต้องเรียนรู้เทคโนโลยีใหม่อย่างรวดเร็ว
4. บอกเกี่ยวกับเวลาที่ deadline กระชั้นชิด คุณจัดการอย่างไร?
5. อธิบายเวลาที่คุณต้องทำงานกับคนที่มีวิสัยทัศน์ต่างกัน

---

## 8. Coding Challenge Best Practices

### ขั้นตอนการแก้ปัญหา

```
1. ทำความเข้าใจปัญหา (2-5 นาที)
   - ถามคำถามเพื่อชี้แจง edge cases
   - ยืนยัน input/output format
   - ถามเรื่อง constraints (ขนาด input, time complexity ที่ยอมรับได้)

2. วางแผน approach (3-5 นาที)
   - คิด brute force ก่อน
   - optimize ถ้าจำเป็น
   - อธิบาย approach ให้ผู้สัมภาษณ์ฟัง

3. เขียนโค้ด (15-20 นาที)
   - เขียนสะอาด อ่านง่าย
   - พูดคุยระหว่างเขียน
   - จัดการ edge cases

4. Test ด้วยตัวอย่าง (5 นาที)
   - ทดสอบด้วย examples ที่ให้มา
   - ทดสอบ edge cases
   - Dry run โค้ด

5. วิเคราะห์ Complexity
   - Time complexity
   - Space complexity
```

### ตัวอย่าง: อธิบายระหว่างเขียนโค้ด

```swift
// ผู้สัมภาษณ์: "หา maximum subarray sum"

// ผม: "ผมจะใช้ Kadane's Algorithm ครับ
// แนวคิดคือ ณ ทุก element เราจะเลือกว่าจะ:
// 1. ต่อ subarray ที่มีอยู่
// 2. เริ่ม subarray ใหม่จาก element นี้
// โดยเลือกตัวที่ใหญ่กว่า"

func maxSubArray(_ nums: [Int]) -> Int {
    // เริ่มจาก element แรก
    var maxSum = nums[0]
    var currentSum = nums[0]
    
    // วน loop จาก element ที่ 2
    for i in 1..<nums.count {
        // เลือกระหว่าง เริ่มใหม่ หรือ ต่อของเดิม
        currentSum = max(nums[i], currentSum + nums[i])
        maxSum = max(maxSum, currentSum)
    }
    
    return maxSum
}

// "Time complexity O(n), Space complexity O(1) ครับ"
// Test: [-2, 1, -3, 4, -1, 2, 1, -5, 4] → 6 (subarray [4,-1,2,1])
print(maxSubArray([-2, 1, -3, 4, -1, 2, 1, -5, 4]))  // 6
```

---

## 9. Take-home Project Tips

### สิ่งที่ควรทำ

```swift
// 1. Project Structure ที่ดี
MyApp/
├── Models/
│   ├── User.swift
│   └── Post.swift
├── Views/
│   ├── Components/
│   └── Screens/
├── ViewModels/
│   └── PostViewModel.swift
├── Services/
│   ├── NetworkService.swift
│   └── StorageService.swift
├── Extensions/
└── Tests/
    ├── Unit/
    └── Integration/

// 2. Architecture Pattern ที่ชัดเจน (MVVM)
// 3. Error handling ที่ครอบคลุม
// 4. Unit tests สำหรับ business logic
// 5. README ที่อธิบาย setup และ decisions
```

### README Template

```markdown
# Project Name

## Setup
1. Clone repo
2. Open .xcodeproj
3. Build & Run

## Architecture
- MVVM pattern
- Combine สำหรับ reactive bindings
- URLSession สำหรับ networking

## Key Decisions
- ใช้ SwiftUI เพราะ...
- ไม่ใช้ third-party libraries เพราะ...
- Core Data สำหรับ local storage เพราะ...

## Testing
- Unit tests: ViewModels
- Integration tests: Network layer
- Test coverage: ~80%

## What I'd improve with more time
- Add pagination
- Implement caching
- Add more error states
```

---

## 10. Portfolio Advice และ GitHub Profile Optimization

### GitHub Profile README

```markdown
# สมชาย พัฒนาซอฟต์แวร์

## 🍎 iOS Developer | Swift | SwiftUI

### Featured Projects

| Project | Description | Tech |
|---------|-------------|------|
| [AppName](link) | A weather app with beautiful animations | SwiftUI, CoreLocation |
| [AppName2](link) | E-commerce app with AR features | UIKit, ARKit, Core Data |

### Skills
- Swift, SwiftUI, UIKit
- MVVM, Clean Architecture
- CoreData, Realm
- REST APIs, GraphQL
- Unit Testing, UI Testing

### Contributions
- 50+ contributions this year
- 3 open source projects
```

### สิ่งที่ควรมีใน Portfolio

1. **App Store App**: มีแอปใน App Store อย่างน้อย 1 ตัว
2. **GitHub repos**: 5-10 repos ที่มีคุณภาพสูง
3. **Personal website/blog**: แสดงความเชี่ยวชาญ
4. **LinkedIn**: อัพเดทสม่ำเสมอ
5. **Stack Overflow**: ตอบคำถาม build reputation

---

## 11. Salary Negotiation สำหรับ iOS Developer

### ช่วงเงินเดือน (ประมาณการ 2024-2025)

**Thailand (กรุงเทพฯ)**:
- Junior (0-2 ปี): 35,000 - 60,000 บาท/เดือน
- Mid-level (2-5 ปี): 60,000 - 100,000 บาท/เดือน
- Senior (5+ ปี): 100,000 - 180,000 บาท/เดือน
- Lead/Principal: 150,000 - 250,000+ บาท/เดือน

**Remote (USD)**:
- Junior: $50,000 - $90,000/year
- Mid: $90,000 - $150,000/year
- Senior: $150,000 - $250,000/year
- FAANG Senior: $250,000 - $500,000+/year (total comp)

### เทคนิค Negotiation

```
1. รู้ market rate: ค้นคว้าจาก Glassdoor, Levels.fyi, LinkedIn Salary
2. อย่าบอกตัวเลขก่อน: "ขอทราบ budget ของบริษัทก่อนได้ไหมครับ"
3. ให้ range ไม่ใช่ตัวเลขเดียว: "ผมมองหาที่ 80,000-95,000 บาท/เดือน"
4. Negotiate total package: เงินเดือน, หุ้น (RSU), signing bonus, วันลา
5. ขอเวลาพิจารณา: "ขอเวลา 2-3 วันเพื่อพิจารณาได้ไหมครับ"
```

---

## 12. คำถามที่ควรถามผู้สัมภาษณ์ (20 ข้อ)

### เกี่ยวกับงาน
1. ในวันแรกที่เข้างาน ผมจะทำงานอะไรบ้าง?
2. Team มีกี่คน และมี iOS developer กี่คน?
3. Release cycle ของแอปเป็นอย่างไร?
4. บริษัทใช้ process อะไรในการ code review?
5. Tech stack ปัจจุบันเป็นอะไรบ้าง และมีแผนจะเปลี่ยนไหม?

### เกี่ยวกับทีม
6. ทีมใช้ methodology อะไร (Agile, Scrum, Kanban)?
7. Engineering culture เป็นอย่างไร?
8. ทีมให้ความสำคัญกับ testing มากแค่ไหน?
9. ทีมทำงาน remote หรือ on-site?
10. Onboarding process เป็นอย่างไร?

### เกี่ยวกับการเติบโต
11. Junior developer เติบโตไปเป็น Senior ใช้เวลาเฉลี่ยเท่าไหร่?
12. บริษัทสนับสนุนการเรียนรู้อย่างไร (conferences, courses, books)?
13. มี mentorship program ไหม?
14. Performance review ทำบ่อยแค่ไหน?
15. โอกาสในการนำเสนอ technical talks ภายในทีม?

### เกี่ยวกับโปรดักต์
16. User base ของแอปเป็นอย่างไร?
17. Business model ของบริษัทเป็นอย่างไร?
18. แผน roadmap สำหรับ 6 เดือนข้างหน้าเป็นอย่างไร?
19. ความท้าทายทางเทคนิคที่ใหญ่ที่สุดในตอนนี้คืออะไร?
20. ทำไมตำแหน่งนี้ถึงว่างอยู่?

---

## 13. Common Mistakes ที่ควรหลีกเลี่ยง

### ระหว่าง Coding Interview

```
❌ ข้อผิดพลาดที่พบบ่อย:

1. เขียนโค้ดทันทีโดยไม่ถาม clarifying questions
   ✅ แก้ไข: ถามก่อน "ค่า input จะเป็น null ได้ไหม? ขนาด array max เท่าไหร่?"

2. ไม่พูดอธิบาย แค่เขียนเงียบ ๆ
   ✅ แก้ไข: อธิบาย thinking process ตลอดเวลา

3. panic เมื่อไม่รู้จัก problem
   ✅ แก้ไข: เริ่มจาก brute force แล้ว optimize ทีละขั้น

4. ไม่ test โค้ดหลังเขียนเสร็จ
   ✅ แก้ไข: walk through ด้วย example เสมอ

5. ใช้ force unwrap (!)
   ✅ แก้ไข: ใช้ if let หรือ guard let

6. ไม่จัดการ edge cases (empty array, nil, single element)
   ✅ แก้ไข: ถาม/สังเกต edge cases ตั้งแต่ต้น
```

### ระหว่าง Behavioral Interview

```
❌ ข้อผิดพลาด:

1. ตอบแบบ generic ไม่มี specific examples
   ✅ แก้ไข: ใช้ STAR method ทุกครั้ง

2. พูดไม่ดีเกี่ยวกับนายจ้างเก่า
   ✅ แก้ไข: "ผมต้องการ challenge ใหม่" แทน "บริษัทเก่าแย่"

3. ไม่รู้จักบริษัทที่สัมภาษณ์
   ✅ แก้ไข: ค้นคว้าบริษัท แอป ข่าวล่าสุดก่อนสัมภาษณ์

4. ตอบสั้นเกินไปหรือยาวเกินไป
   ✅ แก้ไข: target 2-3 นาทีต่อคำถาม
```

---

## 14. กระบวนการสัมภาษณ์ของบริษัท Top ในไทยและต่างประเทศ

### บริษัทไทย

**LINEMAN Wongnai**
- Technical screen (45 นาที)
- Coding test online
- Technical interview (iOS-specific + algorithms)
- Culture fit interview
- Focus: Swift, SwiftUI, performance optimization

**Agoda**
- Online coding test
- Technical interview (2-3 rounds)
- System design
- Focus: Scalability, data structures, algorithms

**SCB Tech X (ธนาคารไทยพาณิชย์)**
- HR screen
- Technical test
- Technical interview
- Focus: FinTech, security, clean code

**True Corporation / DTAC**
- HR interview
- Technical interview
- Product discussion
- Focus: Telecom domain, networking, UX

### บริษัทระดับสากล

**Apple**
- Recruiter screen
- Phone technical interviews (2-3)
- Onsite (5-6 rounds)
  - Coding (4 rounds)
  - Behavioral (1-2 rounds)
- Focus: Deep Swift knowledge, performance, Apple frameworks
- Timeline: 4-8 weeks

**Shopee (Sea Group)**
- Online assessment
- Phone/video screen
- Virtual onsite (4-5 rounds)
  - Algorithms (2 rounds)
  - System design
  - Culture fit
- Focus: Scalability, product thinking

**Grab**
- Screen test
- Technical interview
- System design
- Focus: Mobile architecture, maps/location

**Airbnb, Uber, Meta**
- Similar process: 1 screen + 5-6 onsite rounds
- Focus: MVVM/Clean arch, testing, system design

---

## 15. Practical Mock Interview: 10 Problems และ Solutions

### Problem 1: Reverse a String

```swift
// คำถาม: Reverse string "Hello" → "olleH"

func reverseString(_ s: String) -> String {
    return String(s.reversed())
}

// หรือ manual
func reverseStringManual(_ s: String) -> String {
    var chars = Array(s)
    var left = 0, right = chars.count - 1
    
    while left < right {
        chars.swapAt(left, right)
        left += 1
        right -= 1
    }
    
    return String(chars)
}

print(reverseString("สวัสดี"))  // "ีดัสวส"
```

### Problem 2: Check Palindrome

```swift
func isPalindrome(_ s: String) -> Bool {
    let cleaned = s.lowercased().filter { $0.isLetter || $0.isNumber }
    return cleaned == String(cleaned.reversed())
}

print(isPalindrome("A man a plan a canal Panama"))  // true
print(isPalindrome("race a car"))  // false
```

### Problem 3: FizzBuzz

```swift
func fizzBuzz(_ n: Int) -> [String] {
    return (1...n).map { i in
        switch (i % 3 == 0, i % 5 == 0) {
        case (true, true):  return "FizzBuzz"
        case (true, false): return "Fizz"
        case (false, true): return "Buzz"
        default:            return "\(i)"
        }
    }
}

print(fizzBuzz(15))
```

### Problem 4: Find Missing Number

```swift
// คำถาม: [0,1,3] → 2 (missing in range 0...n)
func missingNumber(_ nums: [Int]) -> Int {
    let n = nums.count
    let expected = n * (n + 1) / 2
    return expected - nums.reduce(0, +)
}

print(missingNumber([3, 0, 1]))  // 2
```

### Problem 5: Merge Two Sorted Arrays

```swift
func mergeSortedArrays(_ nums1: [Int], _ nums2: [Int]) -> [Int] {
    var result: [Int] = []
    var i = 0, j = 0
    
    while i < nums1.count && j < nums2.count {
        if nums1[i] <= nums2[j] {
            result.append(nums1[i])
            i += 1
        } else {
            result.append(nums2[j])
            j += 1
        }
    }
    
    result.append(contentsOf: nums1[i...])
    result.append(contentsOf: nums2[j...])
    
    return result
}

print(mergeSortedArrays([1, 3, 5], [2, 4, 6]))  // [1, 2, 3, 4, 5, 6]
```

### Problem 6: Anagram Check

```swift
func isAnagram(_ s: String, _ t: String) -> Bool {
    guard s.count == t.count else { return false }
    
    var freq = [Character: Int]()
    
    for char in s {
        freq[char, default: 0] += 1
    }
    
    for char in t {
        freq[char, default: 0] -= 1
        if freq[char]! < 0 { return false }
    }
    
    return true
}

print(isAnagram("anagram", "nagaram"))  // true
print(isAnagram("rat", "car"))           // false
```

### Problem 7: Maximum Product Subarray

```swift
func maxProduct(_ nums: [Int]) -> Int {
    var maxProd = nums[0]
    var minProd = nums[0]
    var result = nums[0]
    
    for i in 1..<nums.count {
        let candidates = [nums[i], maxProd * nums[i], minProd * nums[i]]
        maxProd = candidates.max()!
        minProd = candidates.min()!
        result = max(result, maxProd)
    }
    
    return result
}

print(maxProduct([2, 3, -2, 4]))   // 6
print(maxProduct([-2, 0, -1]))     // 0
```

### Problem 8: Valid BST

```swift
func isValidBST(_ root: TreeNode?) -> Bool {
    func validate(_ node: TreeNode?, min: Int?, max: Int?) -> Bool {
        guard let node = node else { return true }
        
        if let min = min, node.val <= min { return false }
        if let max = max, node.val >= max { return false }
        
        return validate(node.left, min: min, max: node.val) &&
               validate(node.right, min: node.val, max: max)
    }
    
    return validate(root, min: nil, max: nil)
}
```

### Problem 9: LRU Cache

```swift
class LRUCache {
    private let capacity: Int
    private var cache: [Int: Int] = [:]
    private var order: [Int] = []
    
    init(_ capacity: Int) {
        self.capacity = capacity
    }
    
    func get(_ key: Int) -> Int {
        guard let value = cache[key] else { return -1 }
        
        // move to most recently used
        order.removeAll { $0 == key }
        order.append(key)
        
        return value
    }
    
    func put(_ key: Int, _ value: Int) {
        if cache[key] != nil {
            order.removeAll { $0 == key }
        } else if cache.count >= capacity {
            // remove least recently used
            let lru = order.removeFirst()
            cache.removeValue(forKey: lru)
        }
        
        cache[key] = value
        order.append(key)
    }
}

let lru = LRUCache(2)
lru.put(1, 1)
lru.put(2, 2)
print(lru.get(1))  // 1
lru.put(3, 3)      // evicts key 2
print(lru.get(2))  // -1 (evicted)
```

### Problem 10: Word Search (Backtracking)

```swift
func exist(_ board: [[Character]], _ word: String) -> Bool {
    let rows = board.count, cols = board[0].count
    let chars = Array(word)
    var board = board
    
    func dfs(_ r: Int, _ c: Int, _ i: Int) -> Bool {
        if i == chars.count { return true }
        if r < 0 || r >= rows || c < 0 || c >= cols { return false }
        if board[r][c] != chars[i] { return false }
        
        let temp = board[r][c]
        board[r][c] = "#"  // mark as visited
        
        let found = dfs(r+1, c, i+1) || dfs(r-1, c, i+1) ||
                    dfs(r, c+1, i+1) || dfs(r, c-1, i+1)
        
        board[r][c] = temp  // restore
        return found
    }
    
    for r in 0..<rows {
        for c in 0..<cols {
            if dfs(r, c, 0) { return true }
        }
    }
    
    return false
}
```

---

## 16. สรุป

การเตรียมตัวสัมภาษณ์งาน iOS Developer ต้องการการฝึกฝนอย่างสม่ำเสมอในหลายด้าน:

### แผนการเตรียมตัว 30 วัน

**สัปดาห์ที่ 1: พื้นฐาน Swift**
- Day 1-2: Memory management (ARC, retain cycles)
- Day 3-4: Value vs Reference types
- Day 5-6: Protocols, Generics
- Day 7: Closures, Functional programming

**สัปดาห์ที่ 2: iOS Frameworks**
- Day 8-9: UIKit fundamentals
- Day 10-11: SwiftUI basics
- Day 12-13: Core Data, Networking
- Day 14: Testing

**สัปดาห์ที่ 3: Algorithms & Data Structures**
- Day 15-16: Arrays, Strings
- Day 17-18: Linked Lists, Stacks, Queues
- Day 19-20: Trees, Graphs
- Day 21: Dynamic Programming

**สัปดาห์ที่ 4: System Design & Soft Skills**
- Day 22-24: Mobile system design
- Day 25-26: Behavioral questions
- Day 27-28: Mock interviews
- Day 29-30: Review weak areas

### Resources ที่แนะนำ

- **LeetCode**: ฝึก coding (target 100+ problems)
- **Swift.org**: เอกสาร official
- **WWDC videos**: iOS engineering deep dives
- **Hacking with Swift**: practical tutorials
- **Ray Wenderlich**: iOS tutorials
- **iOS Interview Guide**: เตรียม iOS specific

---

*"การเตรียมตัวที่ดีคือกุญแจสู่ความสำเร็จ ฝึกฝนทุกวัน แม้แต่ 30 นาที จะสร้างความแตกต่างอย่างมากในระยะยาว"*

---

**ต่อไป**: [Part 76 - เส้นทางอาชีพ iOS Developer](Part76_Career_iOS_Developer.md)
