# Part 69: การเตรียมตัวสัมภาษณ์งาน iOS Developer

## บทนำ

การสัมภาษณ์งาน iOS Developer เป็นกระบวนการที่ต้องใช้ความรู้หลายด้านพร้อมกัน ตั้งแต่ทักษะทางเทคนิค ความสามารถในการแก้ปัญหา ไปจนถึงทักษะการสื่อสารและการทำงานเป็นทีม บทนี้จะครอบคลุมทุกแง่มุมที่จำเป็นสำหรับการเตรียมตัวสัมภาษณ์งาน iOS Developer อย่างครบถ้วน

---

## 1. ภาพรวมกระบวนการสัมภาษณ์ iOS Developer

### 1.1 ขั้นตอนทั่วไปของกระบวนการสัมภาษณ์

กระบวนการสัมภาษณ์งาน iOS Developer มักประกอบด้วยหลายขั้นตอน:

**ขั้นที่ 1: Application Screening**
- HR review resume และ portfolio
- Keyword matching กับ job description
- บางบริษัทมี automated screening ด้วย ATS (Applicant Tracking System)

**ขั้นที่ 2: Phone/Video Screening**
- สนทนากับ HR หรือ Recruiter ประมาณ 15-30 นาที
- ถามเกี่ยวกับ background, motivation, และ availability
- บางครั้งมี technical screening เบื้องต้น

**ขั้นที่ 3: Technical Phone Screen**
- สนทนากับ Engineer ประมาณ 30-60 นาที
- คำถาม Swift/iOS พื้นฐาน
- อาจมี live coding บน shared editor

**ขั้นที่ 4: Take-Home Assignment**
- โปรเจกต์ที่ต้องทำที่บ้าน ใช้เวลา 2-8 ชั่วโมง
- ทดสอบความสามารถในการเขียนโค้ดจริง
- ประเมิน code quality, architecture, และ testing

**ขั้นที่ 5: On-site/Virtual Interview**
- หลาย rounds ติดต่อกัน (3-5 sessions)
- Technical coding interviews
- System design interview
- Behavioral interviews
- Team fit conversations

**ขั้นที่ 6: Final Decision**
- Reference checks
- Offer negotiation
- Background check

### 1.2 สิ่งที่บริษัทประเมิน

บริษัทส่วนใหญ่ประเมินผู้สมัครใน 4 มิติหลัก:

1. **Technical Skills**: ความรู้ Swift, iOS APIs, และ best practices
2. **Problem Solving**: ความสามารถในการวิเคราะห์และแก้ปัญหา
3. **Communication**: การอธิบายความคิดได้ชัดเจน
4. **Culture Fit**: ค่านิยมและการทำงานสอดคล้องกับทีม

---

## 2. ประเภทของการสัมภาษณ์

### 2.1 Technical Interview

การสัมภาษณ์ด้านเทคนิคแบ่งออกเป็น:

**Coding Interview**
- เขียนโค้ดแก้ปัญหา algorithm
- มักใช้ LeetCode-style problems
- ใช้เวลา 45-60 นาทีต่อ session
- อธิบาย thought process ขณะเขียน

**iOS-Specific Technical**
- ถามเกี่ยวกับ Swift language features
- Memory management concepts
- UIKit/SwiftUI patterns
- Networking, Persistence, Concurrency

**Code Review**
- ให้ดู code ที่มีปัญหาแล้วหาข้อผิดพลาด
- เสนอการปรับปรุง
- อธิบาย trade-offs

### 2.2 System Design Interview

สำหรับ Senior positions มักมี:

**Mobile System Design**
- ออกแบบ app architecture
- API design discussion
- Caching strategies
- Offline support
- Performance considerations

**Common Topics**:
- ออกแบบ News Feed app
- ออกแบบ Chat application
- ออกแบบ Photo sharing app
- ออกแบบ Ride-sharing app

### 2.3 Behavioral Interview

ใช้ STAR method (Situation, Task, Action, Result):

- Tell me about a challenging project
- How do you handle disagreements with teammates
- Describe a time you had to learn quickly
- How do you prioritize when multiple things need attention

---

## 3. คำถาม Swift ที่พบบ่อยพร้อมคำตอบ

### 3.1 คำถามพื้นฐาน Swift

**Q: อธิบายความแตกต่างระหว่าง struct และ class**

```swift
// Struct - Value Type
struct Point {
    var x: Double
    var y: Double
    
    mutating func move(by delta: Point) {
        x += delta.x
        y += delta.y
    }
}

// Class - Reference Type
class Person {
    var name: String
    var age: Int
    
    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }
}

// ความแตกต่างหลัก:
var p1 = Point(x: 1, y: 2)
var p2 = p1  // Copy - p2 เป็น independent copy
p2.x = 10
print(p1.x)  // 1 - ไม่ถูก affect

var person1 = Person(name: "Alice", age: 30)
var person2 = person1  // Reference - ชี้ไปที่ object เดียวกัน
person2.name = "Bob"
print(person1.name)  // "Bob" - ถูก affect เพราะ same reference
```

**คำตอบที่ดี**: Struct เป็น value type - เมื่อ assign หรือ pass เป็น parameter จะ copy ค่าทั้งหมด เหมาะกับข้อมูลที่ immutable และ lightweight Class เป็น reference type - ทุกตัวแปรที่ reference ไปที่ object เดียวกันจะเห็นการเปลี่ยนแปลงร่วมกัน เหมาะกับ entities ที่มี identity และ lifecycle

**Q: อธิบาย Optional ใน Swift**

```swift
// Optional คือ type ที่อาจมีค่าหรือไม่มีก็ได้ (nil)
var name: String? = "Alice"
var age: Int? = nil

// 4 วิธีในการ unwrap Optional:

// 1. Optional Binding (if let / guard let)
if let unwrappedName = name {
    print("Name: \(unwrappedName)")
}

// guard let - ใช้ใน function เพื่อ early return
func greet(name: String?) {
    guard let name = name else {
        print("No name provided")
        return
    }
    print("Hello, \(name)!")
}

// 2. Nil Coalescing Operator (??)
let displayName = name ?? "Anonymous"

// 3. Optional Chaining (?.)
let uppercaseName = name?.uppercased()  // Returns String?

// 4. Force Unwrap (!) - ระวัง! crash ถ้า nil
let forcedName = name!  // อันตราย

// Implicit Unwrapped Optional (!= forced unwrap)
var implicitName: String! = "Bob"  // ใช้ระวัง
```

**Q: อธิบาย Closure ใน Swift**

```swift
// Closure คือ block ของโค้ดที่ capture ตัวแปรจาก surrounding scope

// Basic Closure
let greet = { (name: String) -> String in
    return "Hello, \(name)!"
}

// Trailing Closure Syntax
let numbers = [3, 1, 4, 1, 5, 9, 2, 6]
let sorted = numbers.sorted { $0 < $1 }  // Shorthand

// Capture List - จัดการ reference cycle
class NetworkManager {
    var completion: ((Data?) -> Void)?
    
    func fetch(url: URL) {
        // [weak self] ป้องกัน retain cycle
        URLSession.shared.dataTask(with: url) { [weak self] data, _, _ in
            guard let self = self else { return }
            self.completion?(data)
        }.resume()
    }
}

// Escaping vs Non-escaping Closure
func performAsync(completion: @escaping () -> Void) {
    DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
        completion()  // Closure ถูก call หลัง function return = @escaping
    }
}

func performSync(action: () -> Void) {
    action()  // เรียกทันที = non-escaping (default)
}
```

### 3.2 คำถามเกี่ยวกับ Protocol และ Generics

**Q: อธิบาย Protocol-Oriented Programming**

```swift
// Protocol กำหนด interface / contract
protocol Drawable {
    func draw()
    var color: UIColor { get }
}

protocol Resizable {
    mutating func resize(by factor: CGFloat)
}

// Protocol Composition
typealias DrawableResizable = Drawable & Resizable

// Protocol Extension - default implementation
extension Drawable {
    func draw() {
        print("Drawing with color: \(color)")
    }
    
    func drawWithBorder() {
        draw()
        print("Drawing border")
    }
}

// Concrete Types conform to protocols
struct Circle: Drawable, Resizable {
    var radius: CGFloat
    var color: UIColor
    
    mutating func resize(by factor: CGFloat) {
        radius *= factor
    }
}

struct Rectangle: DrawableResizable {
    var width: CGFloat
    var height: CGFloat
    var color: UIColor
    
    mutating func resize(by factor: CGFloat) {
        width *= factor
        height *= factor
    }
}

// Protocol as Type (Existential)
var shapes: [any Drawable] = [Circle(radius: 5, color: .red), 
                               Rectangle(width: 10, height: 5, color: .blue)]

// Protocol with Associated Type
protocol Container {
    associatedtype Item
    var items: [Item] { get }
    mutating func add(_ item: Item)
    func contains(_ item: Item) -> Bool where Item: Equatable
}

struct Stack<T>: Container {
    var items: [T] = []
    
    mutating func add(_ item: T) {
        items.append(item)
    }
    
    func contains(_ item: T) -> Bool where T: Equatable {
        return items.contains(item)
    }
}
```

**Q: อธิบาย Generics และประโยชน์**

```swift
// Generic Function
func swap<T>(_ a: inout T, _ b: inout T) {
    let temp = a
    a = b
    b = temp
}

var x = 5, y = 10
swap(&x, &y)
print(x, y)  // 10, 5

// Generic Type Constraints
func findMax<T: Comparable>(_ array: [T]) -> T? {
    guard !array.isEmpty else { return nil }
    return array.max()
}

// Generic Class
class Cache<Key: Hashable, Value> {
    private var storage: [Key: Value] = [:]
    private let maxSize: Int
    
    init(maxSize: Int) {
        self.maxSize = maxSize
    }
    
    func set(_ value: Value, forKey key: Key) {
        if storage.count >= maxSize {
            storage.removeAll()  // Simplified eviction
        }
        storage[key] = value
    }
    
    func get(forKey key: Key) -> Value? {
        return storage[key]
    }
}

// Where Clause
func allSatisfy<S: Sequence>(_ sequence: S, 
                              condition: (S.Element) -> Bool) -> Bool 
                              where S.Element: Comparable {
    return sequence.allSatisfy(condition)
}
```

---

## 4. คำถามเกี่ยวกับ Memory Management

### 4.1 ARC (Automatic Reference Counting)

**Q: อธิบาย ARC ทำงานอย่างไร**

```swift
// ARC ติดตาม reference count ของ class instances
// เมื่อ count = 0 จะ deallocate memory

class Dog {
    let name: String
    
    init(name: String) {
        self.name = name
        print("\(name) is initialized")
    }
    
    deinit {
        print("\(name) is being deinitialized")
    }
}

// ARC in action
var dog1: Dog? = Dog(name: "Rex")  // Reference count = 1
var dog2: Dog? = dog1               // Reference count = 2
var dog3: Dog? = dog1               // Reference count = 3

dog1 = nil  // Reference count = 2
dog2 = nil  // Reference count = 1
dog3 = nil  // Reference count = 0 → deinit called
```

**Q: อธิบาย Retain Cycle และวิธีแก้**

```swift
// Retain Cycle - 2 objects reference กันทำให้ไม่ถูก deallocate

// ปัญหา: Strong Reference Cycle
class Owner {
    var name: String
    var pet: Pet?  // Strong reference
    
    init(name: String) { self.name = name }
    deinit { print("\(name) deallocated") }
}

class Pet {
    var name: String
    var owner: Owner?  // Strong reference → Retain Cycle!
    
    init(name: String) { self.name = name }
    deinit { print("\(name) deallocated") }
}

var alice: Owner? = Owner(name: "Alice")
var rex: Pet? = Pet(name: "Rex")

alice?.pet = rex   // alice → rex
rex?.owner = alice // rex → alice (cycle!)

alice = nil  // ไม่ถูก deallocate เพราะ rex ยัง hold reference
rex = nil    // ไม่ถูก deallocate เพราะ alice ยัง hold reference
// Memory Leak!

// แก้ไขด้วย weak reference
class FixedPet {
    var name: String
    weak var owner: Owner?  // Weak - ไม่เพิ่ม reference count
    
    init(name: String) { self.name = name }
    deinit { print("\(name) deallocated") }
}

// แก้ไขด้วย unowned reference
class CreditCard {
    let number: String
    unowned let customer: Customer  // unowned - ไม่เพิ่ม count, ไม่ optional
    
    init(number: String, customer: Customer) {
        self.number = number
        self.customer = customer
    }
    
    deinit { print("Card \(number) deallocated") }
}

class Customer {
    var name: String
    var card: CreditCard?  // Customer มีชีวิตนานกว่า Card
    
    init(name: String) { self.name = name }
    deinit { print("\(name) deallocated") }
}
```

**Q: เมื่อไรควรใช้ weak vs unowned**

**คำตอบ:**
- **weak**: ใช้เมื่อ reference อาจเป็น nil ในช่วงชีวิตของ object เช่น delegate pattern ที่ delegate อาจถูก set เป็น nil
- **unowned**: ใช้เมื่อแน่ใจว่า object ที่ reference ไปนั้นมีชีวิตอยู่ตลอดเวลาที่ object นี้ยังอยู่ เช่น credit card กับ customer ที่ card ไม่สามารถมีอยู่โดยไม่มี customer

```swift
// Weak - ใช้ใน Delegate Pattern
protocol ViewDelegate: AnyObject {
    func viewDidLoad()
}

class ViewController {
    weak var delegate: ViewDelegate?  // Weak เพราะ delegate อาจเป็น nil
    
    func notifyDelegate() {
        delegate?.viewDidLoad()  // Safe - optional chaining
    }
}

// Unowned - ใช้เมื่อ lifetime guaranteed
class ViewModel {
    unowned let viewController: ViewController
    
    init(viewController: ViewController) {
        self.viewController = viewController
    }
}

// Closure Capture List
class ImageLoader {
    var image: UIImage?
    
    func loadImage(from url: URL) {
        URLSession.shared.dataTask(with: url) { [weak self] data, _, _ in
            // weak เพราะ ImageLoader อาจถูก deallocate ก่อน completion
            guard let self = self, let data = data else { return }
            self.image = UIImage(data: data)
        }.resume()
    }
}
```

---

## 5. คำถามเกี่ยวกับ Concurrency

### 5.1 Grand Central Dispatch (GCD)

**Q: อธิบาย GCD และ DispatchQueue**

```swift
import Foundation

// Main Queue - UI updates ต้องทำบน main thread
DispatchQueue.main.async {
    // Update UI
    self.label.text = "Updated"
}

// Global Queue - Background work
DispatchQueue.global(qos: .background).async {
    // Heavy computation
    let result = computeHeavyTask()
    
    DispatchQueue.main.async {
        // Update UI with result
        self.updateUI(with: result)
    }
}

// QoS (Quality of Service) Levels
// .userInteractive - Highest priority, for immediate UI interaction
// .userInitiated - User-initiated, expecting quick response
// .default - Normal work
// .utility - Long-running tasks with progress indicator
// .background - User-unaware background tasks
// .unspecified - Legacy code

// Custom Serial Queue
let serialQueue = DispatchQueue(label: "com.app.serial")
serialQueue.async { print("Task 1") }
serialQueue.async { print("Task 2") }  // ทำหลัง Task 1 เสมอ

// Custom Concurrent Queue
let concurrentQueue = DispatchQueue(label: "com.app.concurrent", 
                                    attributes: .concurrent)
concurrentQueue.async { print("Task A") }
concurrentQueue.async { print("Task B") }  // อาจทำพร้อมกันกับ A

// DispatchGroup - รอ multiple tasks
let group = DispatchGroup()

group.enter()
fetchUserData { 
    group.leave()
}

group.enter()
fetchProductData {
    group.leave()
}

group.notify(queue: .main) {
    print("All tasks completed")
}
```

### 5.2 Swift Concurrency (async/await)

**Q: อธิบาย async/await และข้อดีเหนือ completion handlers**

```swift
// Traditional Completion Handler Style - ยากอ่าน (Callback Hell)
func fetchUserAndPosts(userId: String, 
                       completion: @escaping (Result<UserWithPosts, Error>) -> Void) {
    fetchUser(id: userId) { userResult in
        switch userResult {
        case .success(let user):
            fetchPosts(forUser: user.id) { postsResult in
                switch postsResult {
                case .success(let posts):
                    completion(.success(UserWithPosts(user: user, posts: posts)))
                case .failure(let error):
                    completion(.failure(error))
                }
            }
        case .failure(let error):
            completion(.failure(error))
        }
    }
}

// Async/Await Style - อ่านง่ายกว่ามาก
func fetchUserAndPosts(userId: String) async throws -> UserWithPosts {
    let user = try await fetchUser(id: userId)
    let posts = try await fetchPosts(forUser: user.id)
    return UserWithPosts(user: user, posts: posts)
}

// Actor - Thread-safe shared mutable state
actor BankAccount {
    private var balance: Double = 0
    
    func deposit(_ amount: Double) {
        balance += amount
    }
    
    func withdraw(_ amount: Double) throws {
        guard balance >= amount else {
            throw BankError.insufficientFunds
        }
        balance -= amount
    }
    
    var currentBalance: Double {
        balance
    }
}

// การใช้งาน Actor
let account = BankAccount()
await account.deposit(100)
try await account.withdraw(50)

// Structured Concurrency - TaskGroup
func fetchAllUsers() async throws -> [User] {
    try await withThrowingTaskGroup(of: User.self) { group in
        let userIds = ["1", "2", "3", "4", "5"]
        
        for id in userIds {
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

// Async Sequence
func processItems() async {
    let stream = AsyncStream<Int> { continuation in
        for i in 1...10 {
            continuation.yield(i)
        }
        continuation.finish()
    }
    
    for await value in stream {
        print(value)
    }
}
```

**Q: อธิบาย MainActor และการ update UI**

```swift
// @MainActor ทำให้ code ทำงานบน main thread เสมอ
@MainActor
class ViewModel: ObservableObject {
    @Published var users: [User] = []
    @Published var isLoading = false
    @Published var errorMessage: String?
    
    func loadUsers() async {
        isLoading = true
        
        do {
            // Network call - ทำบน background thread โดยอัตโนมัติ
            let fetchedUsers = try await userService.fetchUsers()
            users = fetchedUsers  // Safe เพราะ @MainActor
        } catch {
            errorMessage = error.localizedDescription
        }
        
        isLoading = false
    }
}

// ใช้ MainActor.run สำหรับ isolated code blocks
func updateUI() async {
    let data = await fetchData()
    
    await MainActor.run {
        self.label.text = data.title
        self.imageView.image = data.image
    }
}
```

---

## 6. คำถาม SwiftUI

### 6.1 State Management

**Q: อธิบาย property wrappers ใน SwiftUI**

```swift
import SwiftUI

// @State - Local mutable state ใน View
struct CounterView: View {
    @State private var count = 0
    
    var body: some View {
        VStack {
            Text("Count: \(count)")
            Button("Increment") {
                count += 1
            }
        }
    }
}

// @Binding - Two-way binding กับ parent's state
struct ToggleButton: View {
    @Binding var isOn: Bool
    
    var body: some View {
        Button(isOn ? "Turn Off" : "Turn On") {
            isOn.toggle()
        }
    }
}

// @ObservableObject + @Published - Shared model across views
class UserViewModel: ObservableObject {
    @Published var name = ""
    @Published var email = ""
    @Published var isLoggedIn = false
    
    func login() {
        // Business logic
        isLoggedIn = true
    }
}

// @StateObject - View owns the observable object
struct LoginView: View {
    @StateObject private var viewModel = UserViewModel()
    
    var body: some View {
        TextField("Name", text: $viewModel.name)
    }
}

// @ObservedObject - View receives the observable object
struct ProfileView: View {
    @ObservedObject var viewModel: UserViewModel
    
    var body: some View {
        Text("Welcome, \(viewModel.name)")
    }
}

// @EnvironmentObject - Inject into environment
struct AppView: View {
    @StateObject private var userViewModel = UserViewModel()
    
    var body: some View {
        ContentView()
            .environmentObject(userViewModel)
    }
}

struct ContentView: View {
    @EnvironmentObject var userViewModel: UserViewModel
    
    var body: some View {
        Text(userViewModel.name)
    }
}

// @Environment - Built-in environment values
struct ThemeAwareView: View {
    @Environment(\.colorScheme) var colorScheme
    @Environment(\.sizeCategory) var sizeCategory
    
    var body: some View {
        Text("Hello")
            .foregroundColor(colorScheme == .dark ? .white : .black)
    }
}
```

**Q: อธิบาย View Lifecycle ใน SwiftUI**

```swift
struct LifecycleView: View {
    @State private var message = "Initial"
    
    var body: some View {
        Text(message)
            .onAppear {
                // เรียกเมื่อ view ปรากฏบนหน้าจอ
                message = "Appeared"
            }
            .onDisappear {
                // เรียกเมื่อ view หายไป
                print("View disappeared")
            }
            .task {
                // Async task ที่ cancel อัตโนมัติเมื่อ view disappear
                await loadData()
            }
            .onChange(of: message) { newValue in
                // เรียกเมื่อ message เปลี่ยน
                print("Message changed to: \(newValue)")
            }
    }
    
    func loadData() async {
        // Fetch data
    }
}

// เปรียบเทียบกับ UIKit
class LifecycleViewController: UIViewController {
    override func viewDidLoad() { /* View loaded into memory */ }
    override func viewWillAppear(_ animated: Bool) { /* About to appear */ }
    override func viewDidAppear(_ animated: Bool) { /* Appeared */ }
    override func viewWillDisappear(_ animated: Bool) { /* About to disappear */ }
    override func viewDidDisappear(_ animated: Bool) { /* Disappeared */ }
}
```

---

## 7. คำถาม UIKit

### 7.1 View Controller Lifecycle

**Q: อธิบาย UIViewController lifecycle อย่างละเอียด**

```swift
class DetailViewController: UIViewController {
    
    // 1. Allocation and initialization
    override init(nibName nibNameOrNil: String?, bundle nibBundleOrNil: Bundle?) {
        super.init(nibName: nibNameOrNil, bundle: nibBundleOrNil)
        // เหมาะสำหรับ setup ที่ไม่ต้องการ view
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    // 2. View loading
    override func loadView() {
        // Override ถ้าสร้าง view programmatically
        // ถ้าใช้ storyboard/xib ไม่ต้อง override
        view = UIView()
    }
    
    // 3. View loaded - ใช้สำหรับ initial setup
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
        setupConstraints()
        // ตั้ง one-time configuration
    }
    
    // 4. View about to appear
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        // Refresh data, update UI, register for notifications
        navigationController?.setNavigationBarHidden(false, animated: animated)
    }
    
    // 5. View appeared on screen
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        // Start animations, analytics tracking
        Analytics.track(screen: "Detail")
    }
    
    // 6. View about to disappear
    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        // Save data, stop ongoing tasks
    }
    
    // 7. View disappeared
    override func viewDidDisappear(_ animated: Bool) {
        super.viewDidDisappear(animated)
        // Cleanup, unregister notifications
    }
    
    // 8. Memory warning
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Release non-essential resources
        imageCache.removeAll()
    }
    
    private func setupUI() {
        title = "Detail"
        view.backgroundColor = .systemBackground
    }
    
    private func setupConstraints() {
        // Auto Layout setup
    }
}
```

**Q: อธิบาย Auto Layout และวิธี implement constraints**

```swift
class ProfileViewController: UIViewController {
    
    private let profileImageView = UIImageView()
    private let nameLabel = UILabel()
    private let bioLabel = UILabel()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupConstraints()
    }
    
    private func setupConstraints() {
        // ต้อง set translatesAutoresizingMaskIntoConstraints = false
        [profileImageView, nameLabel, bioLabel].forEach {
            $0.translatesAutoresizingMaskIntoConstraints = false
            view.addSubview($0)
        }
        
        // Method 1: NSLayoutConstraint
        NSLayoutConstraint.activate([
            profileImageView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 20),
            profileImageView.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            profileImageView.widthAnchor.constraint(equalToConstant: 100),
            profileImageView.heightAnchor.constraint(equalToConstant: 100),
            
            nameLabel.topAnchor.constraint(equalTo: profileImageView.bottomAnchor, constant: 16),
            nameLabel.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 16),
            nameLabel.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -16),
            
            bioLabel.topAnchor.constraint(equalTo: nameLabel.bottomAnchor, constant: 8),
            bioLabel.leadingAnchor.constraint(equalTo: nameLabel.leadingAnchor),
            bioLabel.trailingAnchor.constraint(equalTo: nameLabel.trailingAnchor)
        ])
    }
}

// ใช้ UIStackView ลดความซับซ้อน
class SimpleProfileView: UIView {
    
    init() {
        super.init(frame: .zero)
        setupUI()
    }
    
    required init?(coder: NSCoder) { fatalError() }
    
    private func setupUI() {
        let stackView = UIStackView(arrangedSubviews: [
            makeImageView(),
            makeLabel(text: "Name", style: .headline),
            makeLabel(text: "Bio", style: .body)
        ])
        stackView.axis = .vertical
        stackView.spacing = 8
        stackView.alignment = .center
        stackView.translatesAutoresizingMaskIntoConstraints = false
        
        addSubview(stackView)
        
        NSLayoutConstraint.activate([
            stackView.topAnchor.constraint(equalTo: topAnchor, constant: 20),
            stackView.leadingAnchor.constraint(equalTo: leadingAnchor, constant: 16),
            stackView.trailingAnchor.constraint(equalTo: trailingAnchor, constant: -16),
            stackView.bottomAnchor.constraint(lessThanOrEqualTo: bottomAnchor, constant: -20)
        ])
    }
    
    private func makeImageView() -> UIImageView {
        let iv = UIImageView()
        iv.contentMode = .scaleAspectFill
        iv.clipsToBounds = true
        iv.layer.cornerRadius = 50
        iv.widthAnchor.constraint(equalToConstant: 100).isActive = true
        iv.heightAnchor.constraint(equalToConstant: 100).isActive = true
        return iv
    }
    
    private func makeLabel(text: String, style: UIFont.TextStyle) -> UILabel {
        let label = UILabel()
        label.text = text
        label.font = .preferredFont(forTextStyle: style)
        label.numberOfLines = 0
        return label
    }
}
```

---

## 8. คำถามเกี่ยวกับ Data Structures

### 8.1 Array, Stack, Queue

**Q: Implement Stack ด้วย Swift**

```swift
// Stack - LIFO (Last In, First Out)
struct Stack<T> {
    private var elements: [T] = []
    
    var isEmpty: Bool { elements.isEmpty }
    var count: Int { elements.count }
    var top: T? { elements.last }
    
    mutating func push(_ element: T) {
        elements.append(element)
    }
    
    @discardableResult
    mutating func pop() -> T? {
        return elements.popLast()
    }
    
    func peek() -> T? {
        return elements.last
    }
}

// ตัวอย่างการใช้งาน: Valid Parentheses
func isValid(_ s: String) -> Bool {
    var stack = Stack<Character>()
    let matching: [Character: Character] = [")": "(", "]": "[", "}": "{"]
    
    for char in s {
        if "([{".contains(char) {
            stack.push(char)
        } else if let match = matching[char] {
            guard stack.pop() == match else { return false }
        }
    }
    
    return stack.isEmpty
}

// Queue - FIFO (First In, First Out)
struct Queue<T> {
    private var enqueueStack: [T] = []
    private var dequeueStack: [T] = []
    
    var isEmpty: Bool { enqueueStack.isEmpty && dequeueStack.isEmpty }
    var count: Int { enqueueStack.count + dequeueStack.count }
    
    mutating func enqueue(_ element: T) {
        enqueueStack.append(element)
    }
    
    mutating func dequeue() -> T? {
        if dequeueStack.isEmpty {
            dequeueStack = enqueueStack.reversed()
            enqueueStack.removeAll()
        }
        return dequeueStack.popLast()
    }
    
    var front: T? {
        return dequeueStack.last ?? enqueueStack.first
    }
}
```

### 8.2 Linked List

**Q: Implement Linked List ด้วย Swift**

```swift
// Node
class ListNode<T> {
    var value: T
    var next: ListNode<T>?
    
    init(_ value: T) {
        self.value = value
    }
}

// Linked List
class LinkedList<T> {
    var head: ListNode<T>?
    var tail: ListNode<T>?
    var count = 0
    
    var isEmpty: Bool { head == nil }
    
    // O(1) - Append to end
    func append(_ value: T) {
        let node = ListNode(value)
        if let tail = tail {
            tail.next = node
        } else {
            head = node
        }
        tail = node
        count += 1
    }
    
    // O(1) - Prepend to front
    func prepend(_ value: T) {
        let node = ListNode(value)
        node.next = head
        head = node
        if tail == nil { tail = node }
        count += 1
    }
    
    // O(n) - Remove at index
    func remove(at index: Int) -> T? {
        guard index >= 0 && index < count else { return nil }
        
        if index == 0 {
            let value = head?.value
            head = head?.next
            if head == nil { tail = nil }
            count -= 1
            return value
        }
        
        var current = head
        for _ in 0..<(index - 1) {
            current = current?.next
        }
        
        let node = current?.next
        current?.next = node?.next
        if node?.next == nil { tail = current }
        count -= 1
        return node?.value
    }
    
    // O(n) - Reverse
    func reverse() {
        var prev: ListNode<T>? = nil
        var current = head
        tail = head
        
        while let node = current {
            let next = node.next
            node.next = prev
            prev = node
            current = next
        }
        
        head = prev
    }
}

// Interview Problem: Detect Cycle in Linked List (Floyd's Algorithm)
func hasCycle<T>(_ head: ListNode<T>?) -> Bool {
    var slow = head
    var fast = head
    
    while fast != nil && fast?.next != nil {
        slow = slow?.next
        fast = fast?.next?.next
        
        if slow === fast { return true }
    }
    
    return false
}
```

### 8.3 Binary Tree

**Q: Implement Binary Search Tree**

```swift
class TreeNode<T: Comparable> {
    var value: T
    var left: TreeNode<T>?
    var right: TreeNode<T>?
    
    init(_ value: T) {
        self.value = value
    }
}

class BinarySearchTree<T: Comparable> {
    var root: TreeNode<T>?
    
    // O(log n) average - Insert
    func insert(_ value: T) {
        root = insertNode(root, value: value)
    }
    
    private func insertNode(_ node: TreeNode<T>?, value: T) -> TreeNode<T> {
        guard let node = node else { return TreeNode(value) }
        
        if value < node.value {
            node.left = insertNode(node.left, value: value)
        } else if value > node.value {
            node.right = insertNode(node.right, value: value)
        }
        
        return node
    }
    
    // O(n) - Inorder traversal (sorted order)
    func inorder() -> [T] {
        var result: [T] = []
        inorderTraversal(root, result: &result)
        return result
    }
    
    private func inorderTraversal(_ node: TreeNode<T>?, result: inout [T]) {
        guard let node = node else { return }
        inorderTraversal(node.left, result: &result)
        result.append(node.value)
        inorderTraversal(node.right, result: &result)
    }
    
    // Level-order traversal (BFS)
    func levelOrder() -> [[T]] {
        guard let root = root else { return [] }
        
        var result: [[T]] = []
        var queue: [TreeNode<T>] = [root]
        
        while !queue.isEmpty {
            let levelSize = queue.count
            var level: [T] = []
            
            for _ in 0..<levelSize {
                let node = queue.removeFirst()
                level.append(node.value)
                
                if let left = node.left { queue.append(left) }
                if let right = node.right { queue.append(right) }
            }
            
            result.append(level)
        }
        
        return result
    }
}
```

---

## 9. คำถาม Algorithm ที่พบบ่อย

### 9.1 Sorting Algorithms

```swift
// Quick Sort - Average O(n log n)
func quickSort(_ array: [Int]) -> [Int] {
    guard array.count > 1 else { return array }
    
    let pivot = array[array.count / 2]
    let left = array.filter { $0 < pivot }
    let middle = array.filter { $0 == pivot }
    let right = array.filter { $0 > pivot }
    
    return quickSort(left) + middle + quickSort(right)
}

// Merge Sort - Always O(n log n)
func mergeSort(_ array: [Int]) -> [Int] {
    guard array.count > 1 else { return array }
    
    let mid = array.count / 2
    let left = mergeSort(Array(array[..<mid]))
    let right = mergeSort(Array(array[mid...]))
    
    return merge(left, right)
}

private func merge(_ left: [Int], _ right: [Int]) -> [Int] {
    var result: [Int] = []
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
    
    return result + Array(left[leftIndex...]) + Array(right[rightIndex...])
}
```

### 9.2 Two Pointers Pattern

```swift
// Two Sum - O(n)
func twoSum(_ nums: [Int], _ target: Int) -> [Int] {
    var map: [Int: Int] = [:]  // value: index
    
    for (i, num) in nums.enumerated() {
        let complement = target - num
        if let j = map[complement] {
            return [j, i]
        }
        map[num] = i
    }
    
    return []
}

// Three Sum - O(n²)
func threeSum(_ nums: [Int]) -> [[Int]] {
    let sorted = nums.sorted()
    var result: [[Int]] = []
    
    for i in 0..<sorted.count - 2 {
        if i > 0 && sorted[i] == sorted[i-1] { continue }
        
        var left = i + 1
        var right = sorted.count - 1
        
        while left < right {
            let sum = sorted[i] + sorted[left] + sorted[right]
            
            if sum == 0 {
                result.append([sorted[i], sorted[left], sorted[right]])
                while left < right && sorted[left] == sorted[left+1] { left += 1 }
                while left < right && sorted[right] == sorted[right-1] { right -= 1 }
                left += 1
                right -= 1
            } else if sum < 0 {
                left += 1
            } else {
                right -= 1
            }
        }
    }
    
    return result
}
```

### 9.3 Dynamic Programming

```swift
// Fibonacci - O(n) with memoization
func fibonacci(_ n: Int) -> Int {
    if n <= 1 { return n }
    
    var dp = [Int](repeating: 0, count: n + 1)
    dp[0] = 0
    dp[1] = 1
    
    for i in 2...n {
        dp[i] = dp[i-1] + dp[i-2]
    }
    
    return dp[n]
}

// Longest Common Subsequence - O(m*n)
func longestCommonSubsequence(_ text1: String, _ text2: String) -> Int {
    let s1 = Array(text1), s2 = Array(text2)
    let m = s1.count, n = s2.count
    var dp = [[Int]](repeating: [Int](repeating: 0, count: n + 1), count: m + 1)
    
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

// Coin Change - O(amount * coins)
func coinChange(_ coins: [Int], _ amount: Int) -> Int {
    var dp = [Int](repeating: amount + 1, count: amount + 1)
    dp[0] = 0
    
    for i in 1...amount {
        for coin in coins {
            if coin <= i {
                dp[i] = min(dp[i], dp[i - coin] + 1)
            }
        }
    }
    
    return dp[amount] > amount ? -1 : dp[amount]
}
```

---

## 10. System Design สำหรับ Mobile

### 10.1 ออกแบบ News Feed App

**โครงสร้างหลัก:**

```
Feed App Architecture:
├── Presentation Layer
│   ├── FeedViewController (UIKit) / FeedView (SwiftUI)
│   ├── FeedViewModel
│   └── FeedCell / FeedItemView
├── Domain Layer
│   ├── FeedUseCase
│   ├── Post Model
│   └── FeedRepository Protocol
├── Data Layer
│   ├── FeedAPIService
│   ├── FeedCacheService
│   └── FeedRepositoryImpl
└── Infrastructure
    ├── NetworkManager
    ├── ImageCacheManager
    └── PersistenceManager
```

**คำถามที่ต้องตอบในการออกแบบ:**
1. Pagination strategy (cursor-based vs offset-based)
2. Caching strategy (memory + disk)
3. Image loading and caching
4. Optimistic updates
5. Offline support
6. Real-time updates (WebSocket/SSE)

```swift
// Feed Architecture Example

// MARK: - Domain Models
struct Post: Identifiable, Codable {
    let id: String
    let authorId: String
    let content: String
    let imageURLs: [URL]
    let createdAt: Date
    var likeCount: Int
    var isLiked: Bool
    var commentCount: Int
}

struct FeedPage {
    let posts: [Post]
    let nextCursor: String?
    let hasMore: Bool
}

// MARK: - Repository Protocol
protocol FeedRepository {
    func fetchFeed(cursor: String?) async throws -> FeedPage
    func likePost(id: String) async throws
    func saveToLocal(_ posts: [Post]) throws
    func loadFromLocal() throws -> [Post]
}

// MARK: - ViewModel
@MainActor
class FeedViewModel: ObservableObject {
    @Published var posts: [Post] = []
    @Published var isLoading = false
    @Published var isLoadingMore = false
    @Published var error: Error?
    
    private var cursor: String?
    private var hasMore = true
    private let repository: FeedRepository
    
    init(repository: FeedRepository) {
        self.repository = repository
    }
    
    func loadFeed() async {
        guard !isLoading else { return }
        isLoading = true
        cursor = nil
        
        do {
            // Load cached posts first for immediate display
            if let cached = try? repository.loadFromLocal() {
                posts = cached
            }
            
            let page = try await repository.fetchFeed(cursor: nil)
            posts = page.posts
            cursor = page.nextCursor
            hasMore = page.hasMore
            
            // Update cache
            try? repository.saveToLocal(page.posts)
        } catch {
            self.error = error
        }
        
        isLoading = false
    }
    
    func loadMore() async {
        guard hasMore && !isLoadingMore, let cursor = cursor else { return }
        isLoadingMore = true
        
        do {
            let page = try await repository.fetchFeed(cursor: cursor)
            posts.append(contentsOf: page.posts)
            self.cursor = page.nextCursor
            hasMore = page.hasMore
        } catch {
            self.error = error
        }
        
        isLoadingMore = false
    }
    
    func toggleLike(postId: String) {
        guard let index = posts.firstIndex(where: { $0.id == postId }) else { return }
        
        // Optimistic Update
        posts[index].isLiked.toggle()
        posts[index].likeCount += posts[index].isLiked ? 1 : -1
        
        Task {
            do {
                try await repository.likePost(id: postId)
            } catch {
                // Revert on failure
                posts[index].isLiked.toggle()
                posts[index].likeCount += posts[index].isLiked ? 1 : -1
            }
        }
    }
}
```

### 10.2 ออกแบบ Chat Application

**ข้อพิจารณาหลัก:**
- Real-time messaging (WebSocket)
- Message persistence (Core Data / SQLite)
- Offline queue (unsent messages)
- Read receipts
- Push notifications
- Media sharing

```swift
// Chat Message Model
struct Message: Identifiable, Codable {
    let id: String
    let conversationId: String
    let senderId: String
    let content: MessageContent
    let timestamp: Date
    var status: MessageStatus
    
    enum MessageContent: Codable {
        case text(String)
        case image(URL)
        case audio(URL, duration: TimeInterval)
        case file(URL, name: String, size: Int)
    }
    
    enum MessageStatus: String, Codable {
        case sending
        case sent
        case delivered
        case read
        case failed
    }
}

// WebSocket Manager
class ChatWebSocketManager: ObservableObject {
    private var webSocketTask: URLSessionWebSocketTask?
    private let url: URL
    @Published var isConnected = false
    
    var onMessage: ((Message) -> Void)?
    
    init(url: URL) {
        self.url = url
    }
    
    func connect() {
        let urlSession = URLSession(configuration: .default)
        webSocketTask = urlSession.webSocketTask(with: url)
        webSocketTask?.resume()
        isConnected = true
        receive()
    }
    
    func disconnect() {
        webSocketTask?.cancel(with: .goingAway, reason: nil)
        isConnected = false
    }
    
    func send(message: Message) async throws {
        let data = try JSONEncoder().encode(message)
        try await webSocketTask?.send(.data(data))
    }
    
    private func receive() {
        webSocketTask?.receive { [weak self] result in
            switch result {
            case .success(let message):
                switch message {
                case .data(let data):
                    if let msg = try? JSONDecoder().decode(Message.self, from: data) {
                        DispatchQueue.main.async {
                            self?.onMessage?(msg)
                        }
                    }
                case .string(let text):
                    print("Received text: \(text)")
                @unknown default:
                    break
                }
                self?.receive()  // Continue receiving
            case .failure(let error):
                print("WebSocket error: \(error)")
                self?.isConnected = false
            }
        }
    }
}
```

---

## 11. Behavioral Interview Tips

### 11.1 STAR Method

**S - Situation**: อธิบายบริบทและสถานการณ์
**T - Task**: อธิบาย task หรือ challenge ที่ต้องแก้
**A - Action**: อธิบายสิ่งที่คุณทำ
**R - Result**: อธิบายผลลัพธ์ที่ได้

**ตัวอย่างคำตอบ:**

**Q: "เล่าประสบการณ์ที่คุณต้องเรียนรู้เทคโนโลยีใหม่อย่างรวดเร็ว"**

**ตอบแบบ STAR:**

**Situation**: "ทีมเราได้รับ requirement ใหม่ที่ต้องการ AR features ใน app ภายใน 3 เดือน ทั้งที่ไม่มีใครในทีมมีประสบการณ์กับ ARKit มาก่อน"

**Task**: "ผมเป็นคนรับผิดชอบ research และ implement AR feature หลัก ซึ่ง overlap กับ feature อื่นที่กำลังทำอยู่"

**Action**: "ผมแบ่งการเรียนรู้เป็น sprint 1 สัปดาห์ โดยสัปดาห์แรก focus ที่ ARKit fundamentals ผ่าน Apple documentation และ WWDC sessions สัปดาห์ที่สองสร้าง prototype เล็กๆ เพื่อ validate concepts และสัปดาห์ที่สามรวม AR กับ existing app architecture"

**Result**: "เราส่ง feature ได้ทันกำหนด customer satisfaction เพิ่มขึ้น 23% และ feature นั้นกลายเป็นจุดเด่นของ app ในตลาด"

### 11.2 คำถาม Behavioral ที่พบบ่อย

**1. Technical Leadership**
- "เล่าเกี่ยวกับ technical decision ที่สำคัญที่คุณทำ"
- "คุณจัดการ technical debt อย่างไร"
- "เล่าประสบการณ์ที่ต้อง mentor junior developer"

**2. Collaboration**
- "เล่าประสบการณ์ที่ขัดแย้งกับ teammate"
- "คุณทำงานกับ designer/PM อย่างไร"
- "เล่าประสบการณ์ที่ต้อง deliver ใน tight deadline"

**3. Problem Solving**
- "เล่าเกี่ยวกับ bug ที่ยากที่สุดที่เคยแก้"
- "คุณ approach ปัญหาที่ไม่รู้วิธีแก้อย่างไร"
- "เล่าเกี่ยวกับ project ที่ล้มเหลว และเรียนรู้อะไร"

---

## 12. Coding Challenge Tips

### 12.1 วิธี Approach ปัญหา

```
Step 1: Clarify Requirements (2-3 นาที)
- ถามเกี่ยวกับ edge cases
- ตรวจสอบ input/output format
- ถามเกี่ยวกับ constraints

Step 2: Think Out Loud (3-5 นาที)
- อธิบาย brute force solution ก่อน
- วิเคราะห์ time/space complexity
- เสนอ optimization

Step 3: Code the Solution (15-25 นาที)
- เขียน clean, readable code
- อธิบายขณะเขียน
- Handle edge cases

Step 4: Test and Verify (5 นาที)
- Test กับ simple cases
- Test กับ edge cases
- Walk through ด้วย example
```

### 12.2 Template สำหรับ Coding Problems

```swift
// Template สำหรับ Array/String Problems
class Solution {
    func solve(_ input: [Int]) -> [Int] {
        // 1. Handle edge cases
        guard !input.isEmpty else { return [] }
        
        // 2. Initialize variables
        var result: [Int] = []
        
        // 3. Main logic
        // ...
        
        return result
    }
}

// Template สำหรับ Graph Problems (BFS)
class GraphSolution {
    func bfs(_ graph: [[Int]], _ start: Int) -> [Int] {
        var visited = Set<Int>()
        var queue = [start]
        var result: [Int] = []
        
        visited.insert(start)
        
        while !queue.isEmpty {
            let node = queue.removeFirst()
            result.append(node)
            
            for neighbor in graph[node] {
                if !visited.contains(neighbor) {
                    visited.insert(neighbor)
                    queue.append(neighbor)
                }
            }
        }
        
        return result
    }
}

// Template สำหรับ Sliding Window
class SlidingWindowSolution {
    func maxSubarraySum(_ nums: [Int], _ k: Int) -> Int {
        guard nums.count >= k else { return 0 }
        
        var windowSum = nums[0..<k].reduce(0, +)
        var maxSum = windowSum
        
        for i in k..<nums.count {
            windowSum += nums[i] - nums[i - k]
            maxSum = max(maxSum, windowSum)
        }
        
        return maxSum
    }
}
```

---

## 13. Take-Home Project Tips

### 13.1 สิ่งที่ควรทำ

**Architecture:**
- ใช้ Clean Architecture หรือ MVVM อย่างชัดเจน
- แยก layers ให้ดี (Presentation, Domain, Data)
- ใช้ Dependency Injection

**Code Quality:**
- เขียน Unit Tests อย่างน้อย 70% coverage
- ใช้ SwiftLint สำหรับ code style
- Documentation comments สำหรับ public APIs

**UI/UX:**
- Support Dark Mode
- Handle loading/error/empty states
- Basic accessibility support

**Error Handling:**
- Handle network errors gracefully
- Implement retry logic
- User-friendly error messages

### 13.2 README ที่ดี

```markdown
# Project Name

## Overview
Brief description of what the app does

## Architecture
- Pattern: MVVM + Clean Architecture
- Key components: ...

## How to Run
1. Clone the repo
2. Open .xcodeproj
3. Run on simulator/device

## Key Design Decisions
- Why MVVM: Clear separation of concerns...
- Networking: Custom URLSession wrapper for testability...

## Testing
- Unit tests cover ViewModels and Use Cases
- Run tests: Cmd+U

## Known Limitations / Future Improvements
- Feature X was not implemented due to time
- Could improve Y by...
```

---

## 14. Portfolio Building

### 14.1 Projects ที่ควรมีใน Portfolio

**Essential Projects:**
1. **Networking App** - แสดง API integration, async/await, error handling
2. **Core Data App** - แสดง persistence, CRUD operations
3. **Custom UI Components** - แสดง UIKit/SwiftUI skills
4. **Complex Navigation** - แสดง navigation patterns

**Impressive Projects:**
1. **Real-time App** - WebSocket, live updates
2. **AR/ML App** - ARKit, Core ML
3. **Widget/Extension** - App Extensions
4. **Open Source Library** - แสดงความสามารถเขียน reusable code

### 14.2 Project Showcase Template

```swift
// โครงสร้าง GitHub Repository ที่ดี

/*
AppName/
├── README.md (ละเอียด)
├── Screenshots/ (UI screenshots + GIFs)
├── AppName/
│   ├── App/
│   │   └── AppDelegate.swift
│   ├── Presentation/
│   │   ├── Features/
│   │   └── Common/
│   ├── Domain/
│   │   ├── Models/
│   │   ├── UseCases/
│   │   └── Repositories/
│   └── Data/
│       ├── Network/
│       ├── Persistence/
│       └── Repositories/
├── AppNameTests/
└── AppNameUITests/
*/
```

---

## 15. GitHub Profile Optimization

### 15.1 GitHub Profile Tips

**Profile README.md:**
```markdown
# สวัสดี! 👋 ผมชื่อ [ชื่อ]

## iOS Developer | Swift Enthusiast

### ทักษะ
- Swift, Objective-C
- UIKit, SwiftUI, Combine
- MVVM, Clean Architecture, TDD
- Core Data, Realm, Firebase
- CI/CD: Fastlane, GitHub Actions, Bitrise

### Featured Projects
- 🚀 [App Name](link) - Brief description (⭐ X stars)
- 📱 [App Name](link) - Brief description

### สถิติ
![GitHub Stats](https://github-readme-stats.vercel.app/api?username=USERNAME)
```

**Repository Best Practices:**
- ทุก repo มี README ที่ละเอียด
- มี screenshots หรือ demo GIF
- Code ใช้ consistent formatting
- มี License file
- Topics/tags ที่เกี่ยวข้อง

---

## 16. LinkedIn Tips สำหรับ iOS Developer

### 16.1 Profile Optimization

**Headline:**
- อย่าใช้แค่ "iOS Developer"
- ใช้: "iOS Developer | Swift | SwiftUI | 4+ Years | Fintech"

**About Section:**
```
ฉันเป็น iOS Developer ที่มีประสบการณ์ X ปีในการสร้าง 
consumer-facing apps ที่ใช้งานโดยผู้ใช้กว่า [X] คน

ความเชี่ยวชาญ:
• Swift, SwiftUI, UIKit
• Architecture: MVVM, Clean Architecture
• Domains: Fintech, E-commerce, Social

ปัจจุบันกำลังสนใจ: [Current interests]
```

**Experience Section:**
- Focus on impact: "ลด app launch time 40% โดย..."
- ใช้ numbers: "โค้ด feature X ที่มีผู้ใช้ X คน"
- Technologies used: แต่ละ role

**Skills Endorsements:**
- Request endorsements จาก colleagues
- Top 5: Swift, iOS Development, UIKit, SwiftUI, Xcode

---

## 17. Salary Negotiation

### 17.1 Research ก่อน

**แหล่ง salary data:**
- Glassdoor - Company-specific data
- Levels.fyi - Tech companies by level
- LinkedIn Salary
- Blind app - Anonymous discussions
- ถาม network ส่วนตัว

**ปัจจัยที่ส่งผลต่อ salary:**
- YOE (Years of Experience)
- Location (SF, NYC vs. remote)
- Company stage (Startup vs. FAANG)
- Level (Junior, Mid, Senior, Staff)
- Domain expertise (Fintech premium, etc.)

### 17.2 Negotiation Strategy

```
1. อย่า anchor salary ก่อน
   - "ผมสนใจ role นี้มาก แต่อยากรู้ budget range ก่อน"
   
2. เมื่อได้ offer ให้ counter
   - "ขอบคุณสำหรับ offer ครับ ผม excited มาก 
      แต่จากการ research และ experience ที่มี 
      ผมคิดว่า X จะ reflect value ที่ผมจะ bring ได้ดีกว่า"

3. Negotiate ทั้ง package
   - Base salary
   - Equity (RSU/Options)
   - Sign-on bonus
   - PTO
   - Remote flexibility
   
4. เมื่อถูกกดดัน
   - "ผมต้องการเวลา 24-48 ชั่วโมง เพื่อพิจารณาอย่างรอบคอบ"
```

---

## 18. คำถามที่ควรถามผู้สัมภาษณ์

### 18.1 คำถามเกี่ยวกับ Technical

```
1. "Tech stack ปัจจุบันเป็นอย่างไร? 
    มีแผนจะ migrate ไปที่ SwiftUI/Swift 6 อย่างไร?"

2. "ทีมใช้ CI/CD อย่างไร? 
    มี automated testing pipeline ไหม?"

3. "App มี technical debt ไหม? 
    ทีมจัดการอย่างไร?"

4. "Architecture ของ codebase เป็นอย่างไร? 
    มีการทำ documentation ไหม?"

5. "Release cycle เป็นอย่างไร? 
    เร็วที่สุดที่ feature จะ shipped คือกี่วัน?"
```

### 18.2 คำถามเกี่ยวกับ Team

```
1. "ทีม iOS มีกี่คน? 
    ทำงาน parallel tracks อย่างไร?"

2. "Onboarding process เป็นอย่างไร?"

3. "ทีมทำ code review อย่างไร? 
    มี standards อะไรบ้าง?"

4. "ความสำเร็จของ role นี้วัดอย่างไร?"

5. "สิ่งที่คุณชอบมากที่สุดในการทำงานที่นี่คืออะไร?"
```

---

## 19. Common Mistakes ที่ควรหลีกเลี่ยง

### 19.1 ก่อนสัมภาษณ์

- **ไม่ research บริษัท**: ดู app ของเขา, อ่าน tech blog, ดู job description ละเอียด
- **ไม่ practice coding**: LeetCode, HackerRank อย่างน้อย 2-3 เดือนก่อน
- **Resume ไม่ตรงกับ job**: Tailor resume สำหรับแต่ละ application

### 19.2 ระหว่างสัมภาษณ์

- **เงียบขณะ code**: อธิบาย thought process ตลอดเวลา
- **ไม่ถามคำถาม clarifying**: ถามก่อนเริ่ม code เสมอ
- **Overcomplicate solutions**: เริ่มจาก simple solution ก่อน
- **Panic เมื่อ stuck**: หยุด, วิเคราะห์, ถามความช่วยเหลือได้

### 19.3 หลังสัมภาษณ์

- **ไม่ follow up**: ส่ง thank you email ภายใน 24 ชั่วโมง
- **ไม่ reflect**: จดสิ่งที่ทำได้ดีและต้องปรับปรุง

---

## 20. Top iOS Companies และกระบวนการสัมภาษณ์

### 20.1 FAANG/Big Tech

**Apple:**
- 4-6 rounds
- Deep Swift/Objective-C knowledge
- Focus on iOS APIs และ system-level programming
- System design for iOS
- Culture: "Think different", attention to detail

**Meta (Facebook):**
- Coding interview: Data structures + algorithms
- System design: Mobile-specific
- Behavioral: STAR format
- Culture: "Move fast"

**Google:**
- Strong algorithm focus
- General coding + mobile-specific
- Culture fit assessment
- Culture: Data-driven, scale

### 20.2 Startups

**Early-stage (Series A/B):**
- Smaller team → More ownership
- Take-home project สำคัญ
- Culture fit มาก
- Expect to wear many hats

**Growth-stage (Series C+):**
- More structured process
- Focus on domain expertise
- System design ที่ scale ขึ้น

---

## 21. Mock Interview Questions พร้อม Solutions

### 21.1 Easy Level

**Q: Reverse a String in Swift**
```swift
func reverseString(_ s: String) -> String {
    return String(s.reversed())
}

// หรือ manual approach
func reverseStringManual(_ s: String) -> String {
    var chars = Array(s)
    var left = 0
    var right = chars.count - 1
    
    while left < right {
        chars.swapAt(left, right)
        left += 1
        right -= 1
    }
    
    return String(chars)
}
```

**Q: Check Palindrome**
```swift
func isPalindrome(_ s: String) -> Bool {
    let cleaned = s.lowercased().filter { $0.isLetter || $0.isNumber }
    return cleaned == String(cleaned.reversed())
}
```

### 21.2 Medium Level

**Q: LRU Cache Implementation**

```swift
class LRUCache {
    class Node {
        var key: Int
        var val: Int
        var prev: Node?
        var next: Node?
        
        init(_ key: Int, _ val: Int) {
            self.key = key
            self.val = val
        }
    }
    
    private var capacity: Int
    private var cache: [Int: Node] = [:]
    private var head: Node  // dummy head
    private var tail: Node  // dummy tail
    
    init(_ capacity: Int) {
        self.capacity = capacity
        head = Node(0, 0)
        tail = Node(0, 0)
        head.next = tail
        tail.prev = head
    }
    
    func get(_ key: Int) -> Int {
        guard let node = cache[key] else { return -1 }
        moveToFront(node)
        return node.val
    }
    
    func put(_ key: Int, _ value: Int) {
        if let node = cache[key] {
            node.val = value
            moveToFront(node)
        } else {
            if cache.count == capacity {
                let lru = tail.prev!
                remove(lru)
                cache.removeValue(forKey: lru.key)
            }
            
            let node = Node(key, value)
            cache[key] = node
            addToFront(node)
        }
    }
    
    private func remove(_ node: Node) {
        node.prev?.next = node.next
        node.next?.prev = node.prev
    }
    
    private func addToFront(_ node: Node) {
        node.next = head.next
        node.prev = head
        head.next?.prev = node
        head.next = node
    }
    
    private func moveToFront(_ node: Node) {
        remove(node)
        addToFront(node)
    }
}
```

### 21.3 Hard Level

**Q: Serialize and Deserialize Binary Tree**

```swift
class Codec {
    func serialize(_ root: TreeNode?) -> String {
        guard let root = root else { return "null" }
        
        var result: [String] = []
        var queue: [TreeNode?] = [root]
        
        while !queue.isEmpty {
            let node = queue.removeFirst()
            
            if let node = node {
                result.append("\(node.val)")
                queue.append(node.left)
                queue.append(node.right)
            } else {
                result.append("null")
            }
        }
        
        return result.joined(separator: ",")
    }
    
    func deserialize(_ data: String) -> TreeNode? {
        let vals = data.split(separator: ",").map(String.init)
        guard vals[0] != "null" else { return nil }
        
        let root = TreeNode(Int(vals[0])!)
        var queue: [TreeNode] = [root]
        var i = 1
        
        while !queue.isEmpty && i < vals.count {
            let node = queue.removeFirst()
            
            if vals[i] != "null" {
                node.left = TreeNode(Int(vals[i])!)
                queue.append(node.left!)
            }
            i += 1
            
            if i < vals.count && vals[i] != "null" {
                node.right = TreeNode(Int(vals[i])!)
                queue.append(node.right!)
            }
            i += 1
        }
        
        return root
    }
}
```

---

## 22. Resources สำหรับ Preparation

### 22.1 Online Resources

**Coding Practice:**
- LeetCode (อย่างน้อย 50-100 Easy/Medium problems)
- HackerRank iOS challenges
- Cracking the Coding Interview book

**iOS/Swift Specific:**
- Swift by Sundell (swiftbysundell.com)
- Hacking with Swift (hackingwithswift.com)
- WWDC Videos (developer.apple.com/videos)
- Ray Wenderlich tutorials

**System Design:**
- Grokking the System Design Interview
- Mobile System Design (YouTube)
- Designing Data-Intensive Applications book

### 22.2 Daily Practice Plan

```
Month 1: Foundation
- Week 1-2: Swift language features review
- Week 3-4: LeetCode Easy problems (2-3 per day)

Month 2: Intermediate
- Week 1-2: iOS-specific concepts deep dive
- Week 3-4: LeetCode Medium problems (2 per day)

Month 3: Advanced
- Week 1-2: System design concepts
- Week 3: Mock interviews (Pramp, Interviewing.io)
- Week 4: Company research + application
```

---

## สรุป

การเตรียมตัวสัมภาษณ์งาน iOS Developer ต้องใช้เวลาและความพยายาม แต่ถ้าวางแผนอย่างเป็นระบบ คุณจะสามารถสัมภาษณ์ได้อย่างมั่นใจ

**Key Takeaways:**
1. **Technical Foundation**: Master Swift, memory management, concurrency
2. **Data Structures & Algorithms**: Practice consistently
3. **iOS Knowledge**: Deep understanding of UIKit/SwiftUI, architecture patterns
4. **Communication**: Talk through your thinking process
5. **Behavioral**: Prepare STAR stories
6. **Portfolio**: Build and showcase quality projects
7. **Research**: Know the company and their tech stack
8. **Practice**: Mock interviews are invaluable

สิ่งสำคัญที่สุดคือ **consistency** - ฝึกทุกวัน เรียนรู้จากความผิดพลาด และไม่ยอมแพ้ โชคดีสำหรับการสัมภาษณ์ครั้งต่อไป!

---

*เนื้อหาในบทนี้ครอบคลุมทุกแง่มุมของการเตรียมตัวสัมภาษณ์งาน iOS Developer จากพื้นฐานภาษา Swift ไปจนถึง System Design และ Behavioral Skills*
