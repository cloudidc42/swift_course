# ส่วนที่ 20: Memory Management และ ARC ใน Swift

## บทนำ

การจัดการหน่วยความจำ (Memory Management) เป็นหนึ่งในแง่มุมที่สำคัญที่สุดของการพัฒนาแอปพลิเคชัน Swift ใช้กลไกที่เรียกว่า **ARC (Automatic Reference Counting)** เพื่อจัดการ lifecycle ของ objects โดยอัตโนมัติ การเข้าใจ ARC อย่างลึกซึ้งช่วยป้องกัน memory leaks, retain cycles และปัญหาด้านประสิทธิภาพ

---

## 20.1 พื้นฐาน Memory Management

### หน่วยความจำในโปรแกรม

เมื่อโปรแกรมทำงาน หน่วยความจำถูกแบ่งออกเป็นส่วนต่างๆ:

```
┌─────────────────────────────────────┐
│           Stack Memory              │  ← เร็ว, จัดการอัตโนมัติ
│  (Local variables, function calls)  │
├─────────────────────────────────────┤
│                                     │
│           Heap Memory               │  ← ยืดหยุ่น, ต้องจัดการ
│    (Objects, reference types)       │
│                                     │
├─────────────────────────────────────┤
│       Static/Global Memory          │  ← ตลอดชีวิตโปรแกรม
│   (Global variables, constants)     │
├─────────────────────────────────────┤
│          Code Segment               │  ← คำสั่งโปรแกรม
└─────────────────────────────────────┘
```

### Stack vs Heap

```swift
// Stack Memory - Value Types
func stackExample() {
    let x = 10              // อยู่บน stack
    var y = 20              // อยู่บน stack
    let point = (x: 1.0, y: 2.0)  // tuple อยู่บน stack
    
    // เมื่อ function จบ ทุกอย่างบน stack ถูก deallocate ทันที
}

// Heap Memory - Reference Types (Classes)
class Person {
    var name: String
    var age: Int
    
    init(name: String, age: Int) {
        self.name = name
        self.age = age
        print("Person \(name) created")
    }
    
    deinit {
        print("Person \(name) deallocated")
    }
}

func heapExample() {
    let person = Person(name: "Alice", age: 30)
    // person เป็น reference ที่อยู่บน stack
    // แต่ Person object อยู่บน heap
    print(person.name)
    
    // เมื่อ function จบ reference ถูก release
    // ARC จะ deallocate Person object ถ้าไม่มี reference อื่น
}

heapExample()
// Person Alice created
// Person Alice deallocated
```

### Memory Allocation

```swift
// Value types - Stack allocation
struct Point {
    var x: Double
    var y: Double
}

var point1 = Point(x: 1.0, y: 2.0)
var point2 = point1  // Copy - แต่ละตัวมีหน่วยความจำของตัวเอง
point2.x = 10.0

print(point1.x)  // 1.0 - ไม่ถูกกระทบ
print(point2.x)  // 10.0

// Reference types - Heap allocation
class Circle {
    var radius: Double
    
    init(radius: Double) {
        self.radius = radius
    }
}

var circle1 = Circle(radius: 5.0)
var circle2 = circle1  // Reference copy - ชี้ไปที่ object เดียวกัน
circle2.radius = 10.0

print(circle1.radius)  // 10.0 - ถูกกระทบเพราะ share reference เดียวกัน
print(circle2.radius)  // 10.0
```

---

## 20.2 ARC (Automatic Reference Counting)

### ARC ทำงานอย่างไร

```swift
// ARC นับ reference count ของแต่ละ object
// Object ถูก deallocate เมื่อ reference count = 0

class Dog {
    let name: String
    
    init(name: String) {
        self.name = name
        print("\(name) is being initialized")
    }
    
    deinit {
        print("\(name) is being deinitialized")
    }
}

// ตัวอย่างการทำงานของ ARC
var reference1: Dog?
var reference2: Dog?
var reference3: Dog?

reference1 = Dog(name: "Buddy")
// reference count = 1
// "Buddy is being initialized"

reference2 = reference1
// reference count = 2

reference3 = reference1
// reference count = 3

reference1 = nil
// reference count = 2

reference2 = nil
// reference count = 1

reference3 = nil
// reference count = 0 → Deallocated!
// "Buddy is being deinitialized"
```

### ARC ใน Action

```swift
class Customer {
    let name: String
    var card: CreditCard?
    
    init(name: String) {
        self.name = name
        print("\(name): Customer initialized")
    }
    
    deinit {
        print("\(name): Customer deinitialized")
    }
}

class CreditCard {
    let number: UInt64
    var customer: Customer?
    
    init(number: UInt64) {
        self.number = number
        print("Card #\(number): CreditCard initialized")
    }
    
    deinit {
        print("Card #\(number): CreditCard deinitialized")
    }
}

// สถานการณ์ปกติ (ไม่มี cycle)
var john: Customer? = Customer(name: "John")
var card: CreditCard? = CreditCard(number: 1234567890123456)

john?.card = card     // Customer ถือ reference ไปที่ CreditCard
// card?.customer = john  // ถ้าบรรทัดนี้เปิด จะเกิด retain cycle!

card = nil
// CreditCard deinitialized (reference count = 0)

john = nil
// Customer deinitialized (reference count = 0)
```

---

## 20.3 Strong References

Strong reference คือ reference แบบค่าเริ่มต้นใน Swift ที่เพิ่ม reference count

```swift
class VideoPlayer {
    var title: String
    var duration: Double
    
    init(title: String, duration: Double) {
        self.title = title
        self.duration = duration
        print("VideoPlayer '\(title)' initialized")
    }
    
    deinit {
        print("VideoPlayer '\(title)' deinitialized")
    }
}

// Strong reference - เพิ่ม reference count
var player1: VideoPlayer? = VideoPlayer(title: "Swift Tutorial", duration: 45.0)
// Reference count: 1

var player2 = player1  // Strong reference
// Reference count: 2

var player3 = player1  // Strong reference
// Reference count: 3

// แต่ละ variable ถือ strong reference
print("Players: \(player1!.title), \(player2!.title)")

player1 = nil  // Reference count: 2
player2 = nil  // Reference count: 1
player3 = nil  // Reference count: 0 → Dealloc!
// "VideoPlayer 'Swift Tutorial' deinitialized"

// Strong reference ใน Collections
class Task {
    let id: Int
    let name: String
    
    init(id: Int, name: String) {
        self.id = id
        self.name = name
    }
    
    deinit {
        print("Task \(id) '\(name)' deinitialized")
    }
}

var tasks: [Task] = []

tasks.append(Task(id: 1, name: "Download"))
tasks.append(Task(id: 2, name: "Process"))
tasks.append(Task(id: 3, name: "Upload"))

// Array ถือ strong reference ไปที่ Task objects
print("Tasks count: \(tasks.count)")  // 3

tasks.removeAll()
// Tasks ถูก deallocate ทั้งหมด (ถ้าไม่มี reference อื่น)
```

---

## 20.4 Reference Cycles และปัญหา

Reference cycle (หรือ Retain cycle) เกิดขึ้นเมื่อ objects สองตัวหรือมากกว่า ต่างถือ strong reference ถึงกันและกัน ทำให้ ARC ไม่สามารถ deallocate ได้

```swift
// ตัวอย่าง Strong Reference Cycle
class Person {
    let name: String
    var apartment: Apartment?
    
    init(name: String) {
        self.name = name
        print("\(name) is being initialized")
    }
    
    deinit {
        print("\(name) is being deinitialized")
    }
}

class Apartment {
    let unit: String
    var tenant: Person?  // Strong reference → Cycle!
    
    init(unit: String) {
        self.unit = unit
        print("Apartment \(unit) is being initialized")
    }
    
    deinit {
        print("Apartment \(unit) is being deinitialized")
    }
}

// สร้าง Reference Cycle
var john: Person? = Person(name: "John")
var unit4A: Apartment? = Apartment(unit: "4A")

john?.apartment = unit4A  // john → unit4A (strong)
unit4A?.tenant = john     // unit4A → john (strong)

// ตอนนี้เกิด cycle: john ↔ unit4A

john = nil     // ลบ external reference แต่ object ยังอยู่!
unit4A = nil   // ลบ external reference แต่ object ยังอยู่!

// deinit ไม่ถูกเรียก! = Memory Leak
print("This shows there's a memory leak (no deinit messages above)")
```

### Visualizing Reference Cycles

```
Strong Reference Cycle:
     john variable
          ↓ (strong)
     [Person "John"] ←──────────────────┐
          │                             │
          │ apartment (strong)          │ tenant (strong)
          ↓                             │
     [Apartment "4A"] ──────────────────┘

เมื่อ john = nil และ unit4A = nil:
     john variable → nil
     unit4A variable → nil
     
     แต่ [Person "John"] และ [Apartment "4A"] ยังถือกันอยู่!
     Reference count ไม่เป็น 0 → Memory Leak
```

---

## 20.5 Weak References

`weak` reference ไม่เพิ่ม reference count และจะเป็น `nil` อัตโนมัติเมื่อ object ถูก deallocate

```swift
// แก้ไข Cycle ด้วย weak reference
class Person {
    let name: String
    var apartment: Apartment?
    
    init(name: String) { self.name = name }
    deinit { print("\(name) is being deinitialized") }
}

class Apartment {
    let unit: String
    weak var tenant: Person?  // weak reference - ไม่เพิ่ม reference count
    
    init(unit: String) { self.unit = unit }
    deinit { print("Apartment \(unit) is being deinitialized") }
}

var john: Person? = Person(name: "John")
var unit4A: Apartment? = Apartment(unit: "4A")

john?.apartment = unit4A   // Person → Apartment (strong)
unit4A?.tenant = john      // Apartment → Person (weak - ไม่นับ)

// Reference counts:
// john: 1 (john variable เท่านั้น)
// unit4A: 2 (unit4A variable + john.apartment)

john = nil
// john reference count: 0 → Person deallocated!
// "John is being deinitialized"
// unit4A?.tenant จะเป็น nil อัตโนมัติ

print(unit4A?.tenant)  // nil

unit4A = nil
// unit4A reference count: 0 → Apartment deallocated!
// "Apartment 4A is being deinitialized"
```

### กฎการใช้ weak

```swift
// weak ต้องเป็น var (ไม่ใช่ let) และ Optional
class Node {
    var value: Int
    weak var parent: Node?  // ✅ weak + Optional
    var children: [Node] = []
    
    init(value: Int) {
        self.value = value
    }
    
    func addChild(_ child: Node) {
        children.append(child)
        child.parent = self
    }
}

let root = Node(value: 1)
let child1 = Node(value: 2)
let child2 = Node(value: 3)

root.addChild(child1)
root.addChild(child2)

print(child1.parent?.value ?? -1)  // 1 (parent คือ root)
print(root.children.count)          // 2

// weak reference ใน Delegate Pattern
protocol DataSourceDelegate: AnyObject {
    func dataDidLoad(_ data: [String])
    func dataLoadFailed(with error: Error)
}

class DataSource {
    weak var delegate: DataSourceDelegate?  // ✅ weak เพื่อป้องกัน cycle
    
    func loadData() {
        // Simulated async loading
        let data = ["Item 1", "Item 2", "Item 3"]
        delegate?.dataDidLoad(data)
    }
}

class ViewController: DataSourceDelegate {
    let dataSource = DataSource()
    
    init() {
        dataSource.delegate = self  // strong reference ไปที่ ViewController
        // ถ้า DataSource ถือ strong reference กลับ → cycle
        // การใช้ weak ป้องกัน cycle
    }
    
    func dataDidLoad(_ data: [String]) {
        print("Loaded: \(data)")
    }
    
    func dataLoadFailed(with error: Error) {
        print("Error: \(error)")
    }
}
```

---

## 20.6 Unowned References

`unowned` reference ไม่เพิ่ม reference count แต่ต่างจาก `weak` ตรงที่ถือว่า object จะไม่เป็น nil ตลอดช่วงชีวิตของ reference

```swift
// ใช้ unowned เมื่อ lifetime ของ object ที่ถูกอ้างถึง ยาวกว่า object ที่อ้าง
class Customer {
    let name: String
    var card: CreditCard?
    
    init(name: String) {
        self.name = name
        print("\(name): Customer initialized")
    }
    
    deinit {
        print("\(name): Customer deinitialized")
    }
}

class CreditCard {
    let number: UInt64
    unowned let customer: Customer  // unowned - ไม่เพิ่ม reference count
    // CreditCard ไม่มีทางอยู่โดยไม่มี Customer → ใช้ unowned ได้
    
    init(number: UInt64, customer: Customer) {
        self.number = number
        self.customer = customer
        print("Card #\(number): CreditCard initialized")
    }
    
    deinit {
        print("Card #\(number): CreditCard deinitialized")
    }
}

var john: Customer? = Customer(name: "John")
john?.card = CreditCard(number: 1234567890123456, customer: john!)

print(john?.card?.customer.name ?? "none")  // John

john = nil
// john reference count: 0 → Customer deallocated
// "John: Customer deinitialized"
// Card ถูก deallocate ด้วย (เพราะ john?.card ถูก release พร้อมกัน)
// "Card #1234567890123456: CreditCard deinitialized"
```

### weak vs unowned: เมื่อไหรใช้อะไร

```swift
// ใช้ weak เมื่อ:
// - Reference อาจเป็น nil ในระหว่าง lifetime
// - Lifetime ของ object ที่ถูกอ้างถึงสั้นกว่า object ที่อ้าง

class Timer {
    weak var delegate: TimerDelegate?  // delegate อาจหายไปก่อน Timer
}

// ใช้ unowned เมื่อ:
// - Reference จะไม่เป็น nil ตลอด lifetime ของ reference
// - Lifetime ของ object ที่ถูกอ้างถึงยาวกว่าหรือเท่ากับ object ที่อ้าง

class Order {
    let customer: Customer  // Customer ต้องมีก่อน Order
    init(customer: Customer) { self.customer = customer }
}

class OrderItem {
    unowned let order: Order  // ✅ Order จะมีอยู่ตลอดที่ OrderItem มีอยู่
    let product: String
    
    init(order: Order, product: String) {
        self.order = order
        self.product = product
    }
}

// Unsafe unowned - ถ้าเข้าถึงหลัง deallocate จะ crash!
// unowned(unsafe) - เร็วกว่าแต่ไม่ปลอดภัย, ใช้ในกรณีที่ต้องการ performance สูงมาก
```

### Unowned Optional

```swift
// Swift 5.3+ รองรับ unowned Optional
class Department {
    let name: String
    var courses: [Course] = []
    var head: Employee?
    
    init(name: String) { self.name = name }
}

class Course {
    let name: String
    unowned var department: Department  // ไม่เป็น nil
    unowned var instructor: Employee?   // อาจเป็น nil (Swift 5.3+)
    
    init(name: String, in department: Department) {
        self.name = name
        self.department = department
    }
}

class Employee {
    let name: String
    var courses: [Course] = []
    
    init(name: String) { self.name = name }
}

let cs = Department(name: "Computer Science")
let swiftCourse = Course(name: "Swift Programming", in: cs)
let alice = Employee(name: "Alice")

swiftCourse.instructor = alice
cs.courses.append(swiftCourse)
alice.courses.append(swiftCourse)
```

---

## 20.7 Closure Capture และ Memory

Closures สามารถสร้าง retain cycles เมื่อ capture `self` ด้วย strong reference

```swift
// ปัญหา: Retain Cycle ใน Closure
class HTMLElement {
    let name: String
    let text: String?
    
    // ❌ Strong capture of self → Retain Cycle!
    lazy var asHTML: () -> String = {
        if let text = self.text {
            return "<\(self.name)>\(text)</\(self.name)>"
        } else {
            return "<\(self.name) />"
        }
    }
    
    init(name: String, text: String? = nil) {
        self.name = name
        self.text = text
    }
    
    deinit {
        print("\(name) is being deinitialized")
    }
}

var paragraph: HTMLElement? = HTMLElement(name: "p", text: "Hello, world!")
print(paragraph!.asHTML())
// <p>Hello, world!</p>

paragraph = nil
// deinit ไม่ถูกเรียก! → Memory Leak
// HTMLElement → closure (strong)
// closure → HTMLElement (strong capture of self)
```

### Capture Lists

```swift
// ✅ แก้ด้วย [weak self]
class HTMLElementFixed {
    let name: String
    let text: String?
    
    lazy var asHTML: () -> String = { [weak self] in
        guard let self = self else { return "" }
        if let text = self.text {
            return "<\(self.name)>\(text)</\(self.name)>"
        } else {
            return "<\(self.name) />"
        }
    }
    
    init(name: String, text: String? = nil) {
        self.name = name
        self.text = text
    }
    
    deinit {
        print("\(name) is being deinitialized")
    }
}

var paragraph: HTMLElementFixed? = HTMLElementFixed(name: "p", text: "Hello!")
print(paragraph!.asHTML())
paragraph = nil
// "p is being deinitialized" ✅
```

---

## 20.8 [weak self] Pattern

`[weak self]` เป็น pattern ที่ใช้บ่อยที่สุดสำหรับ closures ที่ capture `self`

```swift
// Pattern ที่ 1: guard let self = self
class NetworkManager {
    var isBusy = false
    
    func fetchData(completion: @escaping (String) -> Void) {
        // Simulated async operation
        DispatchQueue.main.asyncAfter(deadline: .now() + 1.0) {
            completion("Data loaded")
        }
    }
}

class ViewController {
    let networkManager = NetworkManager()
    var data: String?
    
    // ✅ วิธีที่ 1: guard let self = self (แนะนำ)
    func loadData() {
        networkManager.fetchData { [weak self] result in
            guard let self = self else {
                print("ViewController was deallocated")
                return
            }
            self.data = result
            self.updateUI()
        }
    }
    
    // ✅ วิธีที่ 2: Optional chaining
    func loadDataAlternative() {
        networkManager.fetchData { [weak self] result in
            self?.data = result
            self?.updateUI()
        }
    }
    
    func updateUI() {
        print("UI Updated with: \(data ?? "no data")")
    }
    
    deinit {
        print("ViewController deinitialized")
    }
}

// ทดสอบ
var vc: ViewController? = ViewController()
vc?.loadData()

DispatchQueue.main.asyncAfter(deadline: .now() + 0.5) {
    vc = nil  // VC ถูก deallocate ก่อน callback
}
// หลังจาก 1 วินาที: "ViewController was deallocated"
```

### [weak self] ใน Timer

```swift
class Counter {
    var count = 0
    var timer: Timer?
    
    func start() {
        // ❌ Strong capture → timer ถือ Counter, Counter ถือ timer
        // timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { _ in
        //     self.count += 1
        // }
        
        // ✅ Weak capture
        timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { [weak self] _ in
            guard let self = self else { return }
            self.count += 1
            print("Count: \(self.count)")
        }
    }
    
    func stop() {
        timer?.invalidate()
        timer = nil
    }
    
    deinit {
        timer?.invalidate()
        print("Counter deinitialized")
    }
}

var counter: Counter? = Counter()
counter?.start()

// หลังจาก 3 วินาที ลบ counter
DispatchQueue.main.asyncAfter(deadline: .now() + 3.0) {
    counter?.stop()
    counter = nil
    // "Counter deinitialized"
}
```

### [weak self] ใน async/await

```swift
class DataService {
    func fetchUser() async throws -> String {
        // Simulated network call
        try await Task.sleep(nanoseconds: 1_000_000_000)
        return "Alice"
    }
}

class ProfileViewModel {
    let service = DataService()
    var userName: String?
    
    func loadProfile() {
        Task { [weak self] in
            guard let self = self else { return }
            do {
                let name = try await self.service.fetchUser()
                // Back to main actor for UI updates
                await MainActor.run { [weak self] in
                    self?.userName = name
                    self?.updateUI()
                }
            } catch {
                print("Error: \(error)")
            }
        }
    }
    
    func updateUI() {
        print("Profile loaded: \(userName ?? "unknown")")
    }
    
    deinit {
        print("ProfileViewModel deinitialized")
    }
}
```

---

## 20.9 [unowned self] Pattern

```swift
// ใช้ [unowned self] เมื่อ self จะมีชีวิตอยู่ตลอดที่ closure ทำงาน
class Animation {
    var isRunning = false
    
    // ✅ unowned เพราะ Animation จะมีอยู่ตลอดที่ closure ทำงาน
    lazy var start: () -> Void = { [unowned self] in
        self.isRunning = true
        print("Animation started")
    }
    
    lazy var stop: () -> Void = { [unowned self] in
        self.isRunning = false
        print("Animation stopped")
    }
    
    deinit {
        print("Animation deinitialized")
    }
}

var animation: Animation? = Animation()
animation?.start()   // Animation started
animation?.stop()    // Animation stopped
animation = nil      // Animation deinitialized

// ⚠️ ระวัง: [unowned self] ที่ไม่ปลอดภัย
class DangerousClass {
    var timer: Timer?
    
    // ❌ อันตราย! ถ้า self ถูก deallocate ก่อน timer fires
    func startDangerously() {
        timer = Timer.scheduledTimer(withTimeInterval: 5.0, repeats: false) { [unowned self] _ in
            // CRASH! ถ้า self ถูก deallocate ก่อนหน้านี้
            print("Timer fired: \(self)")
        }
    }
    
    // ✅ ปลอดภัยกว่า - ใช้ weak แทน
    func startSafely() {
        timer = Timer.scheduledTimer(withTimeInterval: 5.0, repeats: false) { [weak self] _ in
            guard let self = self else { return }
            print("Timer fired safely")
        }
    }
}
```

### เปรียบเทียบ weak vs unowned

```swift
// Scenario 1: Parent-Child ที่ child ไม่อยู่ได้โดยไม่มี parent
class BankAccount {
    let accountNumber: String
    
    lazy var overdraftHandler: () -> Void = { [unowned self] in
        // self (BankAccount) จะมีอยู่ตลอดที่ closure นี้ถูกเรียก
        print("Overdraft detected for account: \(self.accountNumber)")
    }
    
    init(accountNumber: String) {
        self.accountNumber = accountNumber
    }
}

// Scenario 2: Delegate ที่อาจหายไปก่อน
class APIClient {
    weak var delegate: APIClientDelegate?  // ✅ weak เพราะ delegate อาจหายไป
    
    func performRequest() {
        // ถ้า delegate เป็น nil ก็ไม่ทำอะไร
        delegate?.requestDidSucceed(data: "Response")
    }
}

protocol APIClientDelegate: AnyObject {
    func requestDidSucceed(data: String)
}

// Rule of thumb:
// - ใช้ weak ถ้าไม่แน่ใจ (ปลอดภัยกว่า แม้จะช้ากว่าเล็กน้อย)
// - ใช้ unowned เฉพาะเมื่อมั่นใจ 100% ว่า object ไม่ถูก deallocate ก่อน
```

---

## 20.10 Value Types vs Reference Types Memory

```swift
// Value Types - Copy Semantics
struct Temperature {
    var celsius: Double
    
    var fahrenheit: Double {
        get { celsius * 9/5 + 32 }
        set { celsius = (newValue - 32) * 5/9 }
    }
}

var temp1 = Temperature(celsius: 100.0)
var temp2 = temp1  // Full copy

temp2.celsius = 0.0  // ไม่กระทบ temp1

print(temp1.celsius)  // 100.0
print(temp2.celsius)  // 0.0

// Reference Types - Reference Semantics
class MutableTemperature {
    var celsius: Double
    init(celsius: Double) { self.celsius = celsius }
}

var mTemp1 = MutableTemperature(celsius: 100.0)
var mTemp2 = mTemp1  // Reference copy

mTemp2.celsius = 0.0  // กระทบ mTemp1!

print(mTemp1.celsius)  // 0.0 !! (ถูกเปลี่ยน)
print(mTemp2.celsius)  // 0.0

// เมื่อไหรควรใช้ Value vs Reference
// Value Types (struct, enum):
// - เมื่อต้องการ copy semantics
// - เมื่อ data ถูก share ใน multiple contexts
// - Thread-safe by default
// - เหมาะกับ: coordinates, colors, sizes, configurations

// Reference Types (class):
// - เมื่อต้องการ shared mutable state
// - เมื่อมี identity (ใช้ === เปรียบเทียบ)
// - Subclassing
// - เหมาะกับ: view controllers, data managers, services
```

---

## 20.11 Copy-on-Write (COW)

Copy-on-Write เป็นการ optimize สำหรับ value types ขนาดใหญ่ ให้ copy ก็ต่อเมื่อจำเป็น

```swift
// Swift Standard Library ใช้ COW สำหรับ Array, Dictionary, Set, String
var array1 = [1, 2, 3, 4, 5]
var array2 = array1  // ยังไม่ copy จริงๆ! แค่ share storage

// ณ จุดนี้ array1 และ array2 share underlying storage
print(array1)  // [1, 2, 3, 4, 5]
print(array2)  // [1, 2, 3, 4, 5]

// เมื่อ modify ตัวใดตัวหนึ่ง จึง copy
array2.append(6)  // Copy happens here!
print(array1)  // [1, 2, 3, 4, 5] - ไม่ถูกกระทบ
print(array2)  // [1, 2, 3, 4, 5, 6]

// การ implement COW เอง
struct COWData {
    // Internal storage class
    private class Storage {
        var data: [Int]
        init(_ data: [Int]) { self.data = data }
        init(copying other: Storage) { self.data = other.data }
    }
    
    private var storage: Storage
    
    init(_ data: [Int] = []) {
        storage = Storage(data)
    }
    
    // Copy-on-write: copy เฉพาะเมื่อจำเป็น
    private mutating func ensureUnique() {
        if !isKnownUniquelyReferenced(&storage) {
            print("Copying storage...")
            storage = Storage(copying: storage)
        }
    }
    
    var data: [Int] { storage.data }
    
    mutating func append(_ element: Int) {
        ensureUnique()
        storage.data.append(element)
    }
    
    mutating func remove(at index: Int) {
        ensureUnique()
        storage.data.remove(at: index)
    }
}

var cow1 = COWData([1, 2, 3])
var cow2 = cow1  // Share storage (ไม่ copy จริงๆ)

print(cow1.data)  // [1, 2, 3]
print(cow2.data)  // [1, 2, 3]

cow2.append(4)   // "Copying storage..." ← copy เกิดที่นี่
print(cow1.data)  // [1, 2, 3]
print(cow2.data)  // [1, 2, 3, 4]

cow2.append(5)  // ไม่มี "Copying storage..." เพราะ cow2 unique แล้ว
print(cow2.data)  // [1, 2, 3, 4, 5]
```

---

## 20.12 Memory Layout

```swift
import Swift

// ตรวจสอบ memory layout
print("Int size: \(MemoryLayout<Int>.size) bytes")        // 8 bytes (64-bit)
print("Int stride: \(MemoryLayout<Int>.stride) bytes")    // 8 bytes
print("Int alignment: \(MemoryLayout<Int>.alignment)")    // 8

print("Bool size: \(MemoryLayout<Bool>.size) byte")       // 1 byte
print("Double size: \(MemoryLayout<Double>.size) bytes")  // 8 bytes

struct SmallStruct {
    var a: Int8   // 1 byte
    var b: Int16  // 2 bytes
    var c: Int32  // 4 bytes
}

print("SmallStruct size: \(MemoryLayout<SmallStruct>.size)")      // 8
print("SmallStruct stride: \(MemoryLayout<SmallStruct>.stride)")  // 8
print("SmallStruct alignment: \(MemoryLayout<SmallStruct>.alignment)")  // 4

// Memory Alignment
struct AlignedStruct {
    var a: Int8    // 1 byte, offset 0
    // padding 7 bytes
    var b: Int64   // 8 bytes, offset 8 (ต้องการ 8-byte alignment)
    var c: Int32   // 4 bytes, offset 16
    // padding 4 bytes
}

print("AlignedStruct size: \(MemoryLayout<AlignedStruct>.size)")    // 24
print("AlignedStruct stride: \(MemoryLayout<AlignedStruct>.stride)")  // 24

// Reorder fields เพื่อลด padding
struct OptimizedStruct {
    var b: Int64   // 8 bytes, offset 0
    var c: Int32   // 4 bytes, offset 8
    var a: Int8    // 1 byte, offset 12
    // padding 3 bytes
}

print("OptimizedStruct size: \(MemoryLayout<OptimizedStruct>.size)")   // 16
// ประหยัดได้ 8 bytes โดยเรียง fields ใหม่!
```

### Enum Memory Layout

```swift
// Enum memory layout
enum SimpleEnum {
    case a
    case b
    case c
}
print("SimpleEnum size: \(MemoryLayout<SimpleEnum>.size)")  // 1 byte

enum WithAssociated {
    case integer(Int)     // 8 bytes + 1 tag
    case double(Double)   // 8 bytes + 1 tag
    case text(String)     // variable
}
print("WithAssociated size: \(MemoryLayout<WithAssociated>.size)")  // 17+

// Optional bool optimization
// Optional<Bool> ใช้ 3 states ใน 1 byte!
print("Bool size: \(MemoryLayout<Bool>.size)")          // 1
print("Bool? size: \(MemoryLayout<Bool?>.size)")        // 1 (!)
print("Int? size: \(MemoryLayout<Int?>.size)")          // 9 (8 + 1 tag)
```

---

## 20.13 Stack vs Heap Memory

```swift
// Stack Allocation - เร็วมาก, จัดการอัตโนมัติ
func stackAllocation() {
    // ทุกอย่างนี้อยู่บน stack
    var x: Int = 10
    var y: Double = 3.14
    var flag: Bool = true
    var array: (Int, Int, Int) = (1, 2, 3)  // Fixed-size tuple
    
    print(x, y, flag, array)
    // เมื่อ function จบ ทุกอย่างถูก deallocate ทันที
    // ไม่ต้องการ ARC
}

// Heap Allocation - ยืดหยุ่น แต่ช้ากว่า
func heapAllocation() {
    // Classes อยู่บน heap
    class MyClass {
        var value: Int
        init(_ v: Int) { value = v }
    }
    
    let obj1 = MyClass(1)  // Heap allocation
    let obj2 = MyClass(2)  // Heap allocation
    
    // ARC จัดการ lifetime
}

// Small Value Optimization
// Swift optimizes small structs to live on stack
struct TinyStruct {
    var x: Int = 0
}

// ถ้า struct เล็กพอ มันอยู่บน stack โดยอัตโนมัติ
var tiny = TinyStruct()
tiny.x = 42

// Classes always live on heap
// Structs may be optimized to stack (compiler decides)
// Large structs/arrays: heap via COW
```

---

## 20.14 Unsafe Pointers (บทนำ)

```swift
import Foundation

// Unsafe Pointer - ใช้สำหรับ interop กับ C code หรือ low-level optimization
// ⚠️ ใช้ด้วยความระมัดระวัง!

// UnsafePointer - read-only pointer ไปที่ memory
var value: Int = 42
withUnsafePointer(to: &value) { pointer in
    print("Value at \(pointer): \(pointer.pointee)")  // Value at ...: 42
}

// UnsafeMutablePointer - read-write pointer
var mutableValue: Int = 10
withUnsafeMutablePointer(to: &mutableValue) { pointer in
    pointer.pointee = 20  // แก้ไขผ่าน pointer
}
print(mutableValue)  // 20

// UnsafeBufferPointer - pointer to contiguous memory (array-like)
let numbers = [1, 2, 3, 4, 5]
numbers.withUnsafeBufferPointer { buffer in
    print("Count: \(buffer.count)")
    print("First: \(buffer[0])")
    
    // Iterate
    for element in buffer {
        print(element, terminator: " ")  // 1 2 3 4 5
    }
    print()
}

// ตัวอย่างการใช้กับ C API
// (ใน production code จริงๆ)
import Darwin

var randomNumbers = [Int32](repeating: 0, count: 5)
randomNumbers.withUnsafeMutableBufferPointer { buffer in
    // สร้างเลขสุ่มผ่าน C function
    for i in 0..<buffer.count {
        buffer[i] = Int32.random(in: 1...100)
    }
}
print("Random numbers: \(randomNumbers)")

// Manual Memory Management (ไม่แนะนำถ้าไม่จำเป็น)
let pointer = UnsafeMutablePointer<Int>.allocate(capacity: 1)
pointer.initialize(to: 100)
print(pointer.pointee)  // 100
pointer.deallocate()    // ต้อง deallocate เอง!
```

---

## 20.15 autoreleasepool

```swift
import Foundation

// autoreleasepool ช่วยจัดการ memory ในลูปที่สร้าง objects จำนวนมาก
// โดยเฉพาะเมื่อทำงานกับ Objective-C APIs

// ❌ ปัญหา: memory เพิ่มขึ้นเรื่อยๆ ในลูป
func processWithoutPool() {
    for i in 0..<10000 {
        // Objects ที่สร้างใน loop อาจสะสมใน memory
        let data = NSData(contentsOfFile: "/dev/null")
        _ = data
    }
    // Objects ถูก release เมื่อ function จบ (หรือ autorelease pool drain)
}

// ✅ ดีกว่า: ใช้ autoreleasepool
func processWithPool() {
    for i in 0..<10000 {
        autoreleasepool {
            // Objects ในนี้จะถูก release เมื่อ block จบ
            let data = NSData(contentsOfFile: "/dev/null")
            // data ถูก release ที่นี่ ไม่สะสม
            _ = data
        }
    }
}

// ตัวอย่างจริง: การประมวลผล images
func processImages(urls: [URL]) {
    for url in urls {
        autoreleasepool {
            // Image data อาจใหญ่มาก
            // autoreleasepool ช่วยปล่อย memory หลังแต่ละ iteration
            if let data = try? Data(contentsOf: url) {
                // process data...
                print("Processing \(data.count) bytes")
            }
        }
    }
}

// autoreleasepool ใน background threads
func backgroundProcessing() {
    DispatchQueue.global().async {
        autoreleasepool {
            // Objects ที่สร้างใน background thread
            // ถ้าไม่มี autoreleasepool พวกมันอาจไม่ถูก release ทันที
            let results = (0..<1000).map { NSString(format: "Item %d", $0) }
            print("Created \(results.count) items")
        }
        // Objects ถูก release เมื่อ autoreleasepool drain
    }
}
```

---

## 20.16 Detecting Memory Leaks

### การใช้ deinit เพื่อตรวจสอบ

```swift
// เพิ่ม deinit เพื่อตรวจสอบ deallocation
class MemoryTracker {
    let name: String
    
    init(name: String) {
        self.name = name
        print("✅ \(name): allocated")
    }
    
    deinit {
        print("🗑️ \(name): deallocated")
    }
}

func testDeallocation() {
    let tracker = MemoryTracker(name: "TestObject")
    print("Using \(tracker.name)")
}
// ✅ TestObject: allocated
// Using TestObject
// 🗑️ TestObject: deallocated ← ถ้าไม่เห็นบรรทัดนี้ = Memory Leak!

// ตัวอย่างการตรวจหา Leak
class LeakyClass {
    let name: String
    var reference: LeakyClass?
    
    init(name: String) {
        self.name = name
        print("✅ \(name) allocated")
    }
    
    deinit {
        print("🗑️ \(name) deallocated")
    }
}

func testLeak() {
    var a: LeakyClass? = LeakyClass(name: "A")
    var b: LeakyClass? = LeakyClass(name: "B")
    
    a?.reference = b  // A → B (strong)
    b?.reference = a  // B → A (strong) ← สร้าง cycle!
    
    a = nil
    b = nil
    // deinit ไม่ถูกเรียก = Memory Leak!
}

testLeak()
print("After testLeak() - no deinit messages = Memory Leak!")
```

### Memory Debugging Techniques

```swift
// 1. ใช้ assert เพื่อตรวจสอบ deallocation ใน tests
class TrackableObject {
    private static var instanceCount = 0
    
    init() {
        TrackableObject.instanceCount += 1
        print("Instance count: \(TrackableObject.instanceCount)")
    }
    
    deinit {
        TrackableObject.instanceCount -= 1
        print("Instance count: \(TrackableObject.instanceCount)")
    }
    
    static var count: Int { instanceCount }
}

func testNoLeak() {
    var objects: [TrackableObject] = []
    
    for _ in 0..<5 {
        objects.append(TrackableObject())
    }
    
    print("During: \(TrackableObject.count)")  // 5
    objects.removeAll()
    print("After: \(TrackableObject.count)")   // 0 (ถ้าไม่มี leak)
}

// 2. Weak reference เป็น "probe" สำหรับตรวจสอบ lifecycle
func testWithWeakProbe() {
    weak var probe: TrackableObject?
    
    do {
        let obj = TrackableObject()
        probe = obj
        print("Inside scope: \(probe != nil)")  // true
    }
    
    print("Outside scope: \(probe != nil)")  // false (ถ้า deallocated)
    assert(probe == nil, "Memory leak detected!")
}

// 3. Instruments ใน Xcode (กล่าวถึงเพื่อความสมบูรณ์)
// - Product → Profile → Instruments
// - เลือก "Leaks" template
// - Run app และดู leak reports
// - ใช้ "Allocations" เพื่อดู memory usage
// - ใช้ "Zombies" เพื่อหา use-after-free bugs
```

---

## 20.17 Memory Management ใน View Controllers

```swift
import UIKit  // (conceptual - ไม่ run ใน playground ทั่วไป)

// Real-world pattern: View Controller memory management
class MyViewController {  // แทน UIViewController
    var dataSource: DataManager?
    
    // ✅ ใช้ weak สำหรับ delegate
    weak var delegate: MyViewControllerDelegate?
    
    // ✅ ใช้ weak สำหรับ references ที่ไม่ควร own
    weak var parentCoordinator: AppCoordinator?
    
    func setupNotifications() {
        // ✅ ใช้ [weak self] ใน notification handlers
        NotificationCenter.default.addObserver(
            forName: NSNotification.Name("DataUpdated"),
            object: nil,
            queue: .main
        ) { [weak self] notification in
            guard let self = self else { return }
            self.handleDataUpdate(notification)
        }
    }
    
    func handleDataUpdate(_ notification: Notification) {
        print("Data updated")
    }
    
    func loadDataAsync() {
        // ✅ ใช้ [weak self] ใน async callbacks
        dataSource?.fetchAsync { [weak self] result in
            DispatchQueue.main.async { [weak self] in
                guard let self = self else { return }
                switch result {
                case .success(let data):
                    self.updateUI(with: data)
                case .failure(let error):
                    self.showError(error)
                }
            }
        }
    }
    
    func updateUI(with data: Any) {
        print("Updating UI")
    }
    
    func showError(_ error: Error) {
        print("Error: \(error)")
    }
    
    deinit {
        // ✅ ล้าง notifications ใน deinit
        NotificationCenter.default.removeObserver(self)
        print("ViewController deallocated")
    }
}

protocol MyViewControllerDelegate: AnyObject {
    func viewControllerDidFinish(_ vc: MyViewController)
}

class AppCoordinator {}

class DataManager {
    func fetchAsync(completion: @escaping (Result<Any, Error>) -> Void) {
        DispatchQueue.global().async {
            // Simulated async fetch
            completion(.success("data"))
        }
    }
}
```

---

## 20.18 Delegate Pattern กับ Memory Management

```swift
// ❌ ปัญหาที่พบบ่อย: Delegate Retain Cycle
protocol ButtonDelegate: AnyObject {
    func buttonTapped()
}

class Button {
    // ✅ ต้องเป็น weak เสมอสำหรับ delegate!
    weak var delegate: ButtonDelegate?
    
    func tap() {
        delegate?.buttonTapped()
    }
}

class MyScreen: ButtonDelegate {
    let button = Button()
    
    init() {
        button.delegate = self
        // MyScreen → Button (strong via property)
        // Button → MyScreen (weak via delegate) ✅ ไม่มี cycle
    }
    
    func buttonTapped() {
        print("Button was tapped!")
    }
    
    deinit {
        print("MyScreen deallocated")
    }
}

var screen: MyScreen? = MyScreen()
screen?.button.tap()  // Button was tapped!
screen = nil          // MyScreen deallocated ✅

// Multiple Delegates Pattern
protocol ViewDelegate: AnyObject {
    func viewDidAppear()
}

class WeakWrapper<T: AnyObject> {
    weak var value: T?
    init(_ value: T) { self.value = value }
}

class MultiDelegateView {
    private var delegates: [WeakWrapper<AnyObject>] = []
    
    func addDelegate<T: ViewDelegate>(_ delegate: T) {
        delegates.append(WeakWrapper(delegate))
    }
    
    func notifyDelegates() {
        delegates = delegates.filter { $0.value != nil }  // Clean up nil refs
        delegates.forEach { wrapper in
            (wrapper.value as? ViewDelegate)?.viewDidAppear()
        }
    }
}
```

---

## 20.19 Best Practices

```swift
// 1. เสมอใช้ weak สำหรับ delegate properties
protocol NetworkDelegate: AnyObject { }
class NetworkService {
    weak var delegate: NetworkDelegate?  // ✅
}

// 2. ใช้ [weak self] ใน closures ที่อาจ outlive self
class ViewModel {
    func fetch() {
        URLSession.shared.dataTask(with: URL(string: "https://api.example.com")!) {
            [weak self] data, response, error in
            guard let self = self else { return }
            // ปลอดภัย: ถ้า ViewModel ถูก deallocate closure จะ return
            self.processData(data)
        }.resume()
    }
    
    func processData(_ data: Data?) { }
}

// 3. ตรวจสอบ deinit เสมอในช่วง development
class ImportantObject {
    deinit {
        #if DEBUG
        print("⚠️ \(type(of: self)) deallocated")
        #endif
    }
}

// 4. หลีกเลี่ยง force unwrap บน weak references
class SafeExample {
    weak var target: AnyObject?
    
    func doSomething() {
        // ❌ อันตราย
        // let t = target!  // Crash ถ้า target เป็น nil
        
        // ✅ ปลอดภัย
        guard let t = target else {
            print("Target was deallocated")
            return
        }
        print("Target: \(t)")
    }
}

// 5. ใช้ autoreleasepool ใน tight loops
func processLargeDataset() {
    for i in 0..<100000 {
        autoreleasepool {
            let data = NSMutableData(length: 1024)!
            // Process data
            _ = data
        }
    }
}

// 6. Prefer structs เมื่อทำได้ (ไม่ต้องการ ARC)
struct Configuration {  // ✅ struct แทน class
    var timeout: TimeInterval = 30.0
    var maxRetries: Int = 3
    var baseURL: String = "https://api.example.com"
}

// 7. ทำให้ closures ที่ escape ปลอดภัย
class RequestManager {
    private var pendingRequests: [(Data) -> Void] = []
    
    func addRequest(completion: @escaping (Data) -> Void) {
        pendingRequests.append(completion)
    }
    
    func cancelAll() {
        pendingRequests.removeAll()
        // closures ถูก release → weak self references เป็น nil
    }
}
```

---

## 20.20 Common Memory Mistakes

```swift
// Mistake 1: Retain Cycle ใน Block Properties
class BadViewController {
    var onTap: (() -> Void)?
    
    init() {
        // ❌ Strong capture → cycle
        onTap = {
            print("Tapped: \(self)")  // self ถูก capture strong
        }
    }
}

class GoodViewController {
    var onTap: (() -> Void)?
    
    init() {
        // ✅ Weak capture
        onTap = { [weak self] in
            guard let self = self else { return }
            print("Tapped: \(self)")
        }
    }
}

// Mistake 2: Timer Retain Cycle
class BadTimer {
    var timer: Timer?
    var count = 0
    
    func start() {
        // ❌ Timer ถือ strong reference ไป BadTimer
        // BadTimer ถือ strong reference ไป Timer
        timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { _ in
            self.count += 1  // Strong capture!
        }
    }
    
    deinit {
        timer?.invalidate()
        print("BadTimer deinitialized")  // จะไม่ถูกเรียกเพราะ cycle!
    }
}

class GoodTimer {
    var timer: Timer?
    var count = 0
    
    func start() {
        // ✅ Weak capture
        timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) {
            [weak self] _ in
            self?.count += 1
        }
    }
    
    deinit {
        timer?.invalidate()
        print("GoodTimer deinitialized")  // ✅ จะถูกเรียก
    }
}

// Mistake 3: Notification Observer ที่ไม่ remove
class BadObserver {
    init() {
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(handleNotification),
            name: .NSCalendarDayChanged,
            object: nil
        )
        // ❌ ไม่ remove → MemoryLeak และ multiple callbacks!
    }
    
    @objc func handleNotification() { }
}

class GoodObserver {
    init() {
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(handleNotification),
            name: .NSCalendarDayChanged,
            object: nil
        )
    }
    
    @objc func handleNotification() { }
    
    deinit {
        // ✅ Remove observer ใน deinit
        NotificationCenter.default.removeObserver(self)
    }
}

// Mistake 4: Closure ที่ Capture ตัวแปรใหญ่โดยไม่จำเป็น
let largeData = Array(repeating: 0, count: 1_000_000)

// ❌ ดัก largeData ทั้งหมดเข้า closure
let badClosure: () -> Int = {
    return largeData.count  // Closure ถือ reference ไป largeData ทั้ง Array
}

// ✅ ดักเฉพาะค่าที่ต้องการ
let count = largeData.count
let goodClosure: () -> Int = {
    return count  // ดักเฉพาะ Int ไม่ใช่ Array ทั้งหมด
}
```

---

## 20.21 แบบฝึกหัด

### แบบฝึกหัดที่ 1: ตรวจหา Memory Leak

```swift
// โค้ดต่อไปนี้มี memory leak ที่ไหน? แก้ไขอย่างไร?

class Library {
    var name: String
    var books: [Book] = []
    
    init(name: String) {
        self.name = name
        print("Library '\(name)' created")
    }
    
    deinit {
        print("Library '\(name)' destroyed")
    }
}

class Book {
    var title: String
    var library: Library?  // ← ปัญหาอยู่ที่นี่
    
    init(title: String) {
        self.title = title
        print("Book '\(title)' created")
    }
    
    deinit {
        print("Book '\(title)' destroyed")
    }
}

// ทดสอบ
func testLibraryLeak() {
    var lib: Library? = Library(name: "Central Library")
    var book: Book? = Book(title: "Swift Programming")
    
    lib?.books.append(book!)  // Library → Book (strong)
    book?.library = lib        // Book → Library (strong) ← CYCLE!
    
    lib = nil   // Library ยังอยู่เพราะ Book ถือไว้
    book = nil  // Book ยังอยู่เพราะ Library ถือไว้
    // Memory Leak!
}

testLibraryLeak()

// ✅ วิธีแก้:
class BookFixed {
    var title: String
    weak var library: Library?  // ← เปลี่ยนเป็น weak
    
    init(title: String) {
        self.title = title
        print("BookFixed '\(title)' created")
    }
    
    deinit {
        print("BookFixed '\(title)' destroyed")
    }
}

func testLibraryFixed() {
    var lib: Library? = Library(name: "Fixed Library")
    var book: BookFixed? = BookFixed(title: "Fixed Book")
    
    lib?.books.append(book! as! Book)  // ต้องปรับ type
    book?.library = lib
    
    lib = nil
    book = nil
    // ทั้ง Library และ Book ถูก deallocate ✅
}
```

### แบบฝึกหัดที่ 2: Observer Pattern ที่ปลอดภัย

```swift
// Observer Pattern ที่ memory-safe
protocol Observer: AnyObject {
    func update(with value: Int)
}

class Observable {
    private var observers: [WeakObserver] = []
    private var value: Int = 0 {
        didSet { notifyObservers() }
    }
    
    private struct WeakObserver {
        weak var observer: Observer?
    }
    
    func addObserver(_ observer: Observer) {
        observers.append(WeakObserver(observer: observer))
    }
    
    func removeObserver(_ observer: Observer) {
        observers.removeAll { $0.observer === observer }
    }
    
    private func notifyObservers() {
        // ลบ observers ที่ถูก deallocate แล้ว
        observers = observers.filter { $0.observer != nil }
        observers.forEach { $0.observer?.update(with: value) }
    }
    
    func setValue(_ newValue: Int) {
        value = newValue
    }
}

class ConcreteObserver: Observer {
    let name: String
    
    init(name: String) {
        self.name = name
        print("\(name): Observer created")
    }
    
    func update(with value: Int) {
        print("\(name): Received value \(value)")
    }
    
    deinit {
        print("\(name): Observer deallocated")
    }
}

// ทดสอบ
let observable = Observable()
var observer1: ConcreteObserver? = ConcreteObserver(name: "Observer1")
var observer2: ConcreteObserver? = ConcreteObserver(name: "Observer2")

observable.addObserver(observer1!)
observable.addObserver(observer2!)

observable.setValue(10)
// Observer1: Received value 10
// Observer2: Received value 10

observer1 = nil  // Observer1 deallocated
// Observer1: Observer deallocated

observable.setValue(20)
// Observer2: Received value 20 (observer1 ถูกลบอัตโนมัติ)

observer2 = nil
// Observer2: Observer deallocated
```

### แบบฝึกหัดที่ 3: Resource Manager

```swift
// Resource Manager ที่จัดการ memory อย่างถูกต้อง
class Resource {
    let id: Int
    let name: String
    
    init(id: Int, name: String) {
        self.id = id
        self.name = name
        print("Resource \(id) '\(name)' acquired")
    }
    
    deinit {
        print("Resource \(id) '\(name)' released")
    }
}

class ResourceManager {
    private var resources: [Int: Resource] = [:]
    private var cache: [Int: WeakRef<Resource>] = [:]
    
    private class WeakRef<T: AnyObject> {
        weak var value: T?
        init(_ value: T) { self.value = value }
    }
    
    // Acquire resource (strong reference)
    func acquire(id: Int, name: String) -> Resource {
        if let resource = resources[id] {
            return resource
        }
        
        let resource = Resource(id: id, name: name)
        resources[id] = resource
        return resource
    }
    
    // Release resource
    func release(id: Int) {
        resources.removeValue(forKey: id)
        // Resource จะถูก deallocate ถ้าไม่มีคนอื่นถือไว้
    }
    
    // Cached access (weak reference)
    func cachedResource(id: Int) -> Resource? {
        return cache[id]?.value
    }
    
    var activeCount: Int { return resources.count }
}

// ทดสอบ
let manager = ResourceManager()

let r1 = manager.acquire(id: 1, name: "Database Connection")
let r2 = manager.acquire(id: 2, name: "File Handle")

print("Active resources: \(manager.activeCount)")  // 2

manager.release(id: 1)
// Resource 1 'Database Connection' released ← ถ้าไม่มีคนอื่นถือ

print("Active resources: \(manager.activeCount)")  // 1

// r1 ยังอยู่เพราะ local variable ถือไว้
print("r1 still accessible: \(r1.name)")  // r1 still accessible: Database Connection

manager.release(id: 2)
print("Active resources: \(manager.activeCount)")  // 0
// r2 ยังอยู่เพราะ local variable ถือไว้
```

---

## 20.22 Memory Debugging Tools

### Instruments (Xcode)

```
การใช้ Instruments สำหรับ Memory Analysis:

1. เปิด Instruments:
   - Xcode → Product → Profile (⌘I)
   - หรือ Instruments.app โดยตรง

2. Templates ที่ใช้บ่อย:

   📊 Allocations:
   - ดู memory allocations ทั้งหมด
   - Track lifetime ของ objects
   - ระบุ memory spike

   🔍 Leaks:
   - ตรวจหา memory leaks อัตโนมัติ
   - แสดง reference graph
   - ระบุ retain cycles

   💀 Zombies:
   - ตรวจหา use-after-free bugs
   - เปิด NSZombie
   - แสดง message ที่ส่งถึง deallocated objects

3. Address Sanitizer (ASan):
   - Xcode → Scheme → Diagnostics → Address Sanitizer
   - ตรวจหา buffer overflows, use-after-free
   
4. Memory Graph Debugger:
   - Debug → Debug Memory Graph ใน Xcode
   - แสดง reference graph ขณะ debug
   - ระบุ retain cycles ได้ง่าย
```

### Memory Debugging ใน Code

```swift
// Debug Helper สำหรับตรวจสอบ memory
class MemoryDebugHelper {
    
    // ดู reference count (approximate)
    static func referenceCount<T: AnyObject>(of object: T) -> Int {
        // ใช้ Swift runtime function (ไม่แนะนำสำหรับ production)
        return _getRetainCount(object)
    }
    
    // ตรวจสอบว่า object ถูก uniquely referenced
    static func isUnique<T: AnyObject>(_ object: inout T) -> Bool {
        return isKnownUniquelyReferenced(&object)
    }
}

// สร้าง Leak Detector อย่างง่าย
class LeakDetector {
    private static var tracked: [ObjectIdentifier: WeakBox] = [:]
    
    private class WeakBox {
        weak var object: AnyObject?
        let name: String
        
        init(_ object: AnyObject, name: String) {
            self.object = object
            self.name = name
        }
    }
    
    static func track(_ object: AnyObject, name: String = "\(type(of: object))") {
        let id = ObjectIdentifier(object)
        tracked[id] = WeakBox(object, name: name)
    }
    
    static func checkLeaks() {
        print("\n=== Leak Detection Report ===")
        var leakCount = 0
        
        for (_, box) in tracked {
            if box.object != nil {
                print("⚠️ LEAK: \(box.name) is still alive!")
                leakCount += 1
            }
        }
        
        if leakCount == 0 {
            print("✅ No leaks detected!")
        } else {
            print("❌ Found \(leakCount) leak(s)!")
        }
        
        tracked.removeAll()
        print("=============================\n")
    }
}

// ทดสอบ LeakDetector
class TestObject {
    let name: String
    init(name: String) { self.name = name }
    deinit { print("\(name) deallocated") }
}

func testLeakDetection() {
    var obj1: TestObject? = TestObject(name: "Object1")
    var obj2: TestObject? = TestObject(name: "Object2")
    
    LeakDetector.track(obj1!, name: "Object1")
    LeakDetector.track(obj2!, name: "Object2")
    
    obj1 = nil  // ปล่อย Object1
    // obj2 ยังอยู่
    
    LeakDetector.checkLeaks()
    // ⚠️ LEAK: Object2 is still alive!
    
    obj2 = nil
}

testLeakDetection()
```

---

## 20.23 ตัวอย่างจริง: Async Image Loader

```swift
import Foundation

// Async Image Loader ที่จัดการ memory อย่างถูกต้อง
class AsyncImageLoader {
    private var cache = [URL: Data]()
    private var loadingTasks = [URL: URLSessionDataTask]()
    
    // ✅ Singleton ที่ไม่มี memory issue
    static let shared = AsyncImageLoader()
    private init() {}
    
    func loadImage(
        from url: URL,
        completion: @escaping (Result<Data, Error>) -> Void
    ) {
        // ตรวจสอบ cache
        if let cachedData = cache[url] {
            completion(.success(cachedData))
            return
        }
        
        // ตรวจสอบ loading ที่กำลังทำอยู่
        if loadingTasks[url] != nil {
            return
        }
        
        // เริ่ม download
        let task = URLSession.shared.dataTask(with: url) { [weak self] data, _, error in
            guard let self = self else { return }
            
            if let error = error {
                DispatchQueue.main.async {
                    completion(.failure(error))
                }
                return
            }
            
            guard let data = data else { return }
            
            // Cache ผลลัพธ์
            self.cache[url] = data
            self.loadingTasks.removeValue(forKey: url)
            
            DispatchQueue.main.async {
                completion(.success(data))
            }
        }
        
        loadingTasks[url] = task
        task.resume()
    }
    
    func clearCache() {
        cache.removeAll()
        loadingTasks.values.forEach { $0.cancel() }
        loadingTasks.removeAll()
    }
    
    deinit {
        clearCache()
        print("AsyncImageLoader deallocated")
    }
}

// ตัวอย่างการใช้ใน ViewController
class ImageViewController {
    var imageData: Data?
    let loader = AsyncImageLoader.shared
    
    func loadImage() {
        guard let url = URL(string: "https://example.com/image.jpg") else { return }
        
        // ✅ [weak self] เพราะ closure อาจ execute หลัง VC ถูก dismiss
        loader.loadImage(from: url) { [weak self] result in
            guard let self = self else { return }
            
            switch result {
            case .success(let data):
                self.imageData = data
                self.displayImage()
            case .failure(let error):
                print("Failed to load image: \(error)")
            }
        }
    }
    
    func displayImage() {
        print("Displaying image (\(imageData?.count ?? 0) bytes)")
    }
    
    deinit {
        print("ImageViewController deallocated")
    }
}
```

---

## 20.24 สรุป

### สิ่งที่เรียนรู้ในบทนี้

1. **Memory Basics**: Stack (เร็ว, อัตโนมัติ) vs Heap (ยืดหยุ่น, ต้องจัดการ)
2. **ARC**: นับ reference count, deallocate เมื่อ count = 0
3. **Strong References**: ค่าเริ่มต้น, เพิ่ม reference count
4. **Weak References**: ไม่เพิ่ม count, เป็น nil อัตโนมัติ
5. **Unowned References**: ไม่เพิ่ม count, ต้องมั่นใจว่าไม่เป็น nil
6. **Reference Cycles**: ป้องกันด้วย weak หรือ unowned
7. **Closure Capture**: [weak self], [unowned self] ใน capture lists
8. **Copy-on-Write**: Optimization สำหรับ value types
9. **Memory Layout**: Size, stride, alignment
10. **autoreleasepool**: จัดการ Objective-C objects ใน loops
11. **Debugging**: Instruments, Memory Graph, deinit tracking

### Quick Reference

```swift
// เมื่อไหรใช้อะไร
// Strong (default): ownership ที่ชัดเจน
var child: Node

// Weak: optional reference, may become nil
weak var delegate: SomeDelegate?

// Unowned: non-optional reference, must not outlive owner
unowned var parent: Node

// Closure capture rules:
// [weak self]: self might be nil (async callbacks, timers)
// [unowned self]: self will definitely exist (synchronous, computed properties)
// Nothing: synchronous, short-lived closures that don't escape
```

### Checklist สำหรับ Memory Safety

```
✅ ใช้ weak สำหรับ delegate properties ทุกครั้ง
✅ ใช้ [weak self] ใน escaping closures
✅ เพิ่ม deinit ระหว่าง debug เพื่อยืนยัน deallocation
✅ ลบ NotificationCenter observers ใน deinit
✅ invalidate Timers ใน deinit
✅ ใช้ autoreleasepool ใน tight loops กับ Objective-C APIs
✅ ใช้ Instruments Leaks เพื่อตรวจสอบก่อน release
✅ ชอบ struct มากกว่า class เมื่อทำได้
```

---

*จบบทที่ 20: Memory Management และ ARC ใน Swift*

*บทถัดไป: บทที่ 21 - Concurrency และ async/await*
