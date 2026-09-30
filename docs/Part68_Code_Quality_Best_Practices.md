# ตอนที่ 68: Code Quality และ Best Practices

## บทนำ

Code Quality ไม่ใช่แค่เรื่องของ "โค้ดสวย" แต่เป็นเรื่องของความสามารถในการบำรุงรักษา (maintainability), การขยายระบบ (scalability), และความร่วมมือในทีม (collaboration) บทนี้จะครอบคลุมทุกแง่มุมของการเขียนโค้ด Swift ที่มีคุณภาพสูงในระดับ professional

---

## 1. Clean Code Principles

### 1.1 Meaningful Names

```swift
// ❌ BAD: ชื่อที่ไม่สื่อความหมาย
func calc(_ a: Double, _ b: Double, _ c: Int) -> Double {
    var r = 0.0
    for i in 0..<c {
        r += a * b
    }
    return r
}

// ✅ GOOD: ชื่อที่บอกเจตนาชัดเจน
func calculateTotalRevenue(unitPrice: Double, quantity: Double, months: Int) -> Double {
    var totalRevenue = 0.0
    for _ in 0..<months {
        totalRevenue += unitPrice * quantity
    }
    return totalRevenue
}

// ❌ BAD: ชื่อ variable ที่คลุมเครือ
let d = 86400
let data: [Any] = []
let lst = [String]()
let mgr = Manager()

// ✅ GOOD: ชื่อที่บอกความหมายชัดเจน
let secondsPerDay = 86400
let userProfiles: [UserProfile] = []
let recentSearchHistory = [String]()
let userAccountManager = UserAccountManager()

// ❌ BAD: Boolean names ที่สับสน
var flag = false
var check = true
var status = false

// ✅ GOOD: Boolean names ที่อ่านแล้วเข้าใจทันที
var isUserLoggedIn = false
var hasCompletedOnboarding = true
var isLoadingData = false
var canSubmitForm = true
```

### 1.2 Functions ที่ดี

```swift
// ❌ BAD: Function ที่ทำหลายอย่างเกินไป
func processUserData(user: User) {
    // Validate
    guard !user.name.isEmpty else { return }
    guard user.age >= 18 else { return }
    guard isValidEmail(user.email) else { return }
    
    // Save to database
    let userRecord = UserRecord(
        name: user.name,
        email: user.email,
        createdAt: Date()
    )
    database.save(userRecord)
    
    // Send welcome email
    let emailBody = "สวัสดี \(user.name), ขอบคุณที่สมัครสมาชิก!"
    emailService.send(to: user.email, subject: "ยินดีต้อนรับ", body: emailBody)
    
    // Update analytics
    analytics.track(event: "user_registered", properties: ["name": user.name])
    
    // Show success notification
    notificationManager.show(message: "สมัครสมาชิกสำเร็จ!")
}

// ✅ GOOD: แบ่ง function ออกเป็น responsibility เล็กๆ
struct UserRegistrationService {
    
    private let userRepository: UserRepository
    private let emailService: EmailService
    private let analyticsService: AnalyticsService
    
    func registerUser(_ user: User) throws {
        try validateUser(user)
        let savedUser = try saveUser(user)
        sendWelcomeEmail(to: savedUser)
        trackRegistration(for: savedUser)
    }
    
    private func validateUser(_ user: User) throws {
        guard !user.name.isEmpty else {
            throw ValidationError.emptyName
        }
        guard user.age >= 18 else {
            throw ValidationError.underAge
        }
        guard isValidEmail(user.email) else {
            throw ValidationError.invalidEmail
        }
    }
    
    private func saveUser(_ user: User) throws -> User {
        return try userRepository.save(user)
    }
    
    private func sendWelcomeEmail(to user: User) {
        emailService.sendWelcome(to: user)
    }
    
    private func trackRegistration(for user: User) {
        analyticsService.track(.userRegistered(userID: user.id))
    }
    
    private func isValidEmail(_ email: String) -> Bool {
        let pattern = "[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}"
        return email.range(of: pattern, options: .regularExpression) != nil
    }
}

enum ValidationError: Error {
    case emptyName
    case underAge
    case invalidEmail
}

struct User {
    let id: String
    let name: String
    let email: String
    let age: Int
}

// Stub types
class UserRepository {
    func save(_ user: User) throws -> User { return user }
}
class EmailService {
    func sendWelcome(to user: User) {}
}
class AnalyticsService {
    enum Event { case userRegistered(userID: String) }
    func track(_ event: Event) {}
}
```

### 1.3 Comments ที่มีคุณค่า

```swift
// ❌ BAD: Comments ที่ไม่เพิ่มค่า
// เพิ่ม 1 ให้กับ count
count += 1

// ตรวจสอบว่า array ว่างหรือไม่
if items.isEmpty {
    return
}

// ✅ GOOD: Comments ที่อธิบาย WHY ไม่ใช่ WHAT
// Increment count before processing to handle the
// edge case where the first item triggers an early return
count += 1

// ✅ GOOD: อธิบาย business logic ที่ซับซ้อน
/// คำนวณราคาหลังส่วนลดตาม loyalty tier
/// 
/// ระบบ tier:
/// - Bronze (0-999 points): ส่วนลด 5%
/// - Silver (1000-4999 points): ส่วนลด 10%
/// - Gold (5000+ points): ส่วนลด 15%
///
/// หมายเหตุ: ส่วนลดไม่ถูก compound กับโปรโมชั่นอื่น
func calculateDiscountedPrice(_ originalPrice: Double, loyaltyPoints: Int) -> Double {
    let discountRate: Double
    switch loyaltyPoints {
    case 0..<1000:
        discountRate = 0.05
    case 1000..<5000:
        discountRate = 0.10
    default:
        discountRate = 0.15
    }
    return originalPrice * (1 - discountRate)
}

// ✅ GOOD: TODO/FIXME with tracking
// TODO(john@team.com, JIRA-1234): Implement caching for better performance
// FIXME: Race condition possible when multiple threads access this - needs mutex
// MARK: - Network Layer (section markers)
```

---

## 2. SOLID Principles ด้วย Swift

### 2.1 Single Responsibility Principle (SRP)

```swift
// ❌ BAD: Class ที่รับผิดชอบหลายอย่าง
class UserManager {
    
    func login(email: String, password: String) -> Bool {
        // Authenticate user
        return true
    }
    
    func saveToDatabase(user: User) {
        // Save to CoreData
    }
    
    func sendWelcomeEmail(to email: String) {
        // Send email via SMTP
    }
    
    func generateReport() -> String {
        // Generate user activity report
        return ""
    }
    
    func formatName(_ name: String) -> String {
        // Format user name
        return name
    }
}

// ✅ GOOD: แยก responsibility ออกชัดเจน
class AuthenticationService {
    func login(email: String, password: String) async throws -> AuthToken {
        // Only handles authentication
        return AuthToken(value: "token")
    }
    
    func logout(token: AuthToken) async throws {
        // Only handles logout
    }
}

class UserRepository {
    func save(_ user: User) async throws {
        // Only handles data persistence
    }
    
    func find(byID id: String) async throws -> User? {
        // Only handles data retrieval
        return nil
    }
}

class EmailNotificationService {
    func sendWelcomeEmail(to email: String, name: String) async throws {
        // Only handles email sending
    }
}

class ReportGenerator {
    func generateUserActivityReport(userID: String) async -> String {
        // Only handles report generation
        return ""
    }
}

struct AuthToken {
    let value: String
}
```

### 2.2 Open/Closed Principle (OCP)

```swift
// ❌ BAD: ต้องแก้ class เดิมเมื่อเพิ่ม payment method
class PaymentProcessor {
    func process(amount: Double, method: String) throws {
        if method == "credit_card" {
            // process credit card
        } else if method == "paypal" {
            // process paypal
        } else if method == "bitcoin" {
            // process bitcoin - ต้องแก้ class ทุกครั้งที่เพิ่ม method
        }
    }
}

// ✅ GOOD: Open for extension, closed for modification
protocol PaymentMethod {
    var name: String { get }
    func processPayment(amount: Double) async throws -> PaymentResult
    func validatePayment(amount: Double) throws
}

struct PaymentResult {
    let transactionID: String
    let status: Status
    enum Status { case success, pending, failed }
}

struct CreditCardPayment: PaymentMethod {
    var name: String { "credit_card" }
    let cardNumber: String
    let expiryDate: String
    let cvv: String
    
    func processPayment(amount: Double) async throws -> PaymentResult {
        // Credit card specific logic
        return PaymentResult(transactionID: UUID().uuidString, status: .success)
    }
    
    func validatePayment(amount: Double) throws {
        guard amount > 0 else { throw PaymentError.invalidAmount }
        guard amount <= 100000 else { throw PaymentError.exceedsLimit }
    }
}

struct PayPalPayment: PaymentMethod {
    var name: String { "paypal" }
    let email: String
    let accessToken: String
    
    func processPayment(amount: Double) async throws -> PaymentResult {
        // PayPal specific logic
        return PaymentResult(transactionID: UUID().uuidString, status: .success)
    }
    
    func validatePayment(amount: Double) throws {
        guard amount > 0 else { throw PaymentError.invalidAmount }
    }
}

// เพิ่ม payment method ใหม่โดยไม่ต้องแก้ existing code
struct PromptPayPayment: PaymentMethod {
    var name: String { "promptpay" }
    let phoneNumber: String
    
    func processPayment(amount: Double) async throws -> PaymentResult {
        // PromptPay specific logic
        return PaymentResult(transactionID: UUID().uuidString, status: .pending)
    }
    
    func validatePayment(amount: Double) throws {
        guard amount > 0 && amount <= 50000 else {
            throw PaymentError.invalidAmount
        }
    }
}

// Processor ไม่ต้องเปลี่ยนเมื่อเพิ่ม payment method
class PaymentService {
    func process(payment: PaymentMethod, amount: Double) async throws -> PaymentResult {
        try payment.validatePayment(amount: amount)
        return try await payment.processPayment(amount: amount)
    }
}

enum PaymentError: Error {
    case invalidAmount
    case exceedsLimit
    case processingFailed(String)
}
```

### 2.3 Liskov Substitution Principle (LSP)

```swift
// ❌ BAD: Subclass ที่ violate behavior ของ parent
class Rectangle {
    var width: Double
    var height: Double
    
    init(width: Double, height: Double) {
        self.width = width
        self.height = height
    }
    
    var area: Double { width * height }
}

class Square: Rectangle {
    // Square override behavior - violates LSP
    override var width: Double {
        didSet { height = width }  // Square must have equal sides
    }
    override var height: Double {
        didSet { width = height }
    }
}

// Bug: ใช้ Square แทน Rectangle ได้ผลต่างกัน
func resizeRectangle(_ rect: Rectangle) {
    rect.width = 10
    rect.height = 5
    // expect: area = 50
    // with Square: area = 25 (unexpected!)
}

// ✅ GOOD: ใช้ Protocol แทน Inheritance
protocol Shape {
    var area: Double { get }
    var perimeter: Double { get }
}

struct Rectangle: Shape {
    let width: Double
    let height: Double
    
    var area: Double { width * height }
    var perimeter: Double { 2 * (width + height) }
}

struct Square: Shape {
    let side: Double
    
    var area: Double { side * side }
    var perimeter: Double { 4 * side }
}

struct Circle: Shape {
    let radius: Double
    
    var area: Double { Double.pi * radius * radius }
    var perimeter: Double { 2 * Double.pi * radius }
}

// ใช้ได้กับ Shape ทุก type อย่าง consistent
func printShapeInfo(_ shape: Shape) {
    print("Area: \(shape.area), Perimeter: \(shape.perimeter)")
}
```

### 2.4 Interface Segregation Principle (ISP)

```swift
// ❌ BAD: Interface ที่ใหญ่เกินไป
protocol Worker {
    func work()
    func eat()
    func sleep()
    func attend(meeting: Meeting)
    func writeCode()
    func designUI()
    func manageTeam()
}

// RobotWorker ต้อง implement eat() และ sleep() ทั้งที่ไม่ต้องการ
class RobotWorker: Worker {
    func work() { print("Working...") }
    func eat() { fatalError("Robots don't eat!") }  // Problem!
    func sleep() { fatalError("Robots don't sleep!") }  // Problem!
    func attend(meeting: Meeting) { }
    func writeCode() { }
    func designUI() { }
    func manageTeam() { }
}

// ✅ GOOD: แบ่ง interface ย่อยๆ ตาม capability
protocol Workable {
    func work()
}

protocol Eatable {
    func eat()
}

protocol Sleepable {
    func sleep()
}

protocol MeetingAttendable {
    func attend(meeting: Meeting)
}

protocol CodeWritable {
    func writeCode()
}

protocol UIDesignable {
    func designUI()
}

protocol TeamManageable {
    func manageTeam()
}

// Human worker ใช้ทุก capabilities
class HumanDeveloper: Workable, Eatable, Sleepable, MeetingAttendable, CodeWritable {
    func work() { print("Human working") }
    func eat() { print("Eating lunch") }
    func sleep() { print("Sleeping 8 hours") }
    func attend(meeting: Meeting) { print("In meeting") }
    func writeCode() { print("Writing Swift code") }
}

// Robot ใช้แค่ที่จำเป็น
class RobotDeveloper: Workable, MeetingAttendable, CodeWritable {
    func work() { print("Robot processing") }
    func attend(meeting: Meeting) { print("Video call") }
    func writeCode() { print("AI generating code") }
}

struct Meeting {
    let title: String
    let startTime: Date
}
```

### 2.5 Dependency Inversion Principle (DIP)

```swift
// ❌ BAD: High-level module ขึ้นกับ low-level module โดยตรง
class UserService {
    // Directly depends on concrete implementation
    private let mysqlDatabase = MySQLDatabase()
    private let smtpEmailSender = SMTPEmailSender()
    
    func createUser(name: String, email: String) {
        let user = User(id: UUID().uuidString, name: name, email: email, age: 0)
        mysqlDatabase.save(user)
        smtpEmailSender.send(to: email, message: "Welcome!")
    }
}

// ✅ GOOD: Depend on abstractions
protocol Database {
    func save<T: Encodable>(_ object: T, collection: String) throws
    func fetch<T: Decodable>(_ type: T.Type, id: String, collection: String) throws -> T?
}

protocol EmailSender {
    func send(to email: String, subject: String, body: String) async throws
}

// Concrete implementations (low-level)
class MySQLDatabase: Database {
    func save<T: Encodable>(_ object: T, collection: String) throws {
        // MySQL implementation
    }
    func fetch<T: Decodable>(_ type: T.Type, id: String, collection: String) throws -> T? {
        return nil
    }
}

class FirestoreDatabase: Database {
    func save<T: Encodable>(_ object: T, collection: String) throws {
        // Firestore implementation
    }
    func fetch<T: Decodable>(_ type: T.Type, id: String, collection: String) throws -> T? {
        return nil
    }
}

class SendGridEmailSender: EmailSender {
    func send(to email: String, subject: String, body: String) async throws {
        // SendGrid API implementation
    }
}

// High-level module depends on abstractions
class UserServiceDIP {
    private let database: Database
    private let emailSender: EmailSender
    
    // Inject dependencies
    init(database: Database, emailSender: EmailSender) {
        self.database = database
        self.emailSender = emailSender
    }
    
    func createUser(name: String, email: String) async throws {
        let user = User(id: UUID().uuidString, name: name, email: email, age: 0)
        try database.save(user, collection: "users")
        try await emailSender.send(
            to: email,
            subject: "ยินดีต้อนรับ",
            body: "สวัสดี \(name)!"
        )
    }
}

// Easy to test - inject mock
class MockDatabase: Database {
    var savedObjects: [String: Any] = [:]
    
    func save<T: Encodable>(_ object: T, collection: String) throws {
        // In-memory storage for testing
    }
    
    func fetch<T: Decodable>(_ type: T.Type, id: String, collection: String) throws -> T? {
        return nil
    }
}

class MockEmailSender: EmailSender {
    var sentEmails: [(to: String, subject: String, body: String)] = []
    
    func send(to email: String, subject: String, body: String) async throws {
        sentEmails.append((to: email, subject: subject, body: body))
    }
}
```

---

## 3. DRY และ KISS Principles

### 3.1 DRY (Don't Repeat Yourself)

```swift
// ❌ BAD: Code ซ้ำในหลายที่
class ProductViewController: UIViewController {
    func showLoadingIndicator() {
        let indicator = UIActivityIndicatorView(style: .large)
        indicator.center = view.center
        indicator.startAnimating()
        view.addSubview(indicator)
    }
}

class OrderViewController: UIViewController {
    func showLoadingIndicator() {
        let indicator = UIActivityIndicatorView(style: .large)
        indicator.center = view.center
        indicator.startAnimating()
        view.addSubview(indicator)
    }
}

// ✅ GOOD: Reusable Extension
extension UIViewController {
    func showLoadingIndicator(tag: Int = 999) -> UIActivityIndicatorView {
        let indicator = UIActivityIndicatorView(style: .large)
        indicator.tag = tag
        indicator.center = view.center
        indicator.startAnimating()
        view.addSubview(indicator)
        return indicator
    }
    
    func hideLoadingIndicator(tag: Int = 999) {
        view.viewWithTag(tag)?.removeFromSuperview()
    }
}

// DRY ด้วย Generic Functions
// ❌ BAD: Functions แยกสำหรับแต่ละ type
func filterProducts(_ products: [Product], where predicate: (Product) -> Bool) -> [Product] {
    return products.filter(predicate)
}

func filterOrders(_ orders: [Order], where predicate: (Order) -> Bool) -> [Order] {
    return orders.filter(predicate)
}

// ✅ GOOD: Generic function
func filterItems<T>(_ items: [T], where predicate: (T) -> Bool) -> [T] {
    return items.filter(predicate)
}

// DRY ด้วย Protocol Extensions
protocol Identifiable {
    var id: String { get }
}

protocol Timestampable {
    var createdAt: Date { get }
    var updatedAt: Date { get }
}

extension Array where Element: Identifiable {
    func find(byID id: String) -> Element? {
        return first { $0.id == id }
    }
    
    func remove(byID id: String) -> [Element] {
        return filter { $0.id != id }
    }
}

struct Order: Identifiable, Timestampable {
    let id: String
    let createdAt: Date
    let updatedAt: Date
    let items: [String]
}

// ใช้ find(byID:) กับ type ใดก็ได้ที่ implement Identifiable
let orders: [Order] = []
let specificOrder = orders.find(byID: "order-123")
```

### 3.2 KISS (Keep It Simple, Stupid)

```swift
// ❌ BAD: Over-engineered solution
class UserStatusCalculator {
    
    struct StatusContext {
        let user: User
        let loginHistory: [Date]
        let purchaseHistory: [Double]
        let preferences: [String: Any]
    }
    
    enum StatusStrategy {
        case basic
        case advanced
        case ml
    }
    
    private let strategy: StatusStrategy
    
    init(strategy: StatusStrategy = .basic) {
        self.strategy = strategy
    }
    
    func calculateStatus(context: StatusContext) -> String {
        switch strategy {
        case .basic:
            return basicCalculation(context: context)
        case .advanced:
            return advancedCalculation(context: context)
        case .ml:
            return mlCalculation(context: context)
        }
    }
    
    private func basicCalculation(context: StatusContext) -> String {
        // Complex logic for something simple
        return "active"
    }
    
    private func advancedCalculation(context: StatusContext) -> String {
        return "active"
    }
    
    private func mlCalculation(context: StatusContext) -> String {
        return "active"
    }
}

// ✅ GOOD: Simple, clear solution
extension User {
    var status: String {
        guard !name.isEmpty else { return "incomplete" }
        return "active"
    }
}

// ✅ GOOD: KISS ในการ implement features
// ❌ BAD: Premature optimization
func findUser(id: String) -> User? {
    // Complex binary search, caching, etc. สำหรับ array ขนาดเล็ก
    let cache = NSCache<NSString, AnyObject>()
    // ... 50 lines of code
    return nil
}

// ✅ GOOD: Simple until you actually need optimization
func findUser(id: String, in users: [User]) -> User? {
    return users.first { $0.id == id }
    // เพิ่ม optimization เมื่อ profiling บอกว่าช้าจริงๆ
}
```

---

## 4. Code Smells และ Refactoring

### 4.1 Long Method

```swift
// ❌ BAD: Method ที่ยาวเกินไป (Long Method)
func processOrder(order: Order) {
    // Validate order
    guard !order.items.isEmpty else {
        print("Empty order")
        return
    }
    guard order.items.count <= 100 else {
        print("Too many items")
        return
    }
    
    // Calculate prices
    var subtotal = 0.0
    for item in order.items {
        // subtotal += item.price * item.quantity
    }
    
    // Apply discounts
    var discount = 0.0
    if subtotal > 1000 {
        discount = subtotal * 0.1
    } else if subtotal > 500 {
        discount = subtotal * 0.05
    }
    
    // Calculate tax
    let taxRate = 0.07
    let taxableAmount = subtotal - discount
    let tax = taxableAmount * taxRate
    
    // Calculate shipping
    var shippingCost = 50.0
    if subtotal > 500 {
        shippingCost = 0
    }
    
    // Final total
    let total = subtotal - discount + tax + shippingCost
    
    // Save to database
    // database.save(order, total: total)
    
    // Send confirmation email
    // emailService.sendConfirmation(order: order, total: total)
    
    // Update inventory
    for item in order.items {
        // inventory.decrease(item.id, by: item.count)
    }
    
    print("Order processed: \(total)")
}

// ✅ GOOD: Extract Method Refactoring
struct OrderProcessor {
    
    func processOrder(_ order: Order) throws {
        try validateOrder(order)
        let pricing = calculatePricing(for: order)
        try saveOrder(order, with: pricing)
        notifyCustomer(order: order, pricing: pricing)
        updateInventory(for: order)
    }
    
    private func validateOrder(_ order: Order) throws {
        guard !order.items.isEmpty else {
            throw OrderError.emptyOrder
        }
        guard order.items.count <= 100 else {
            throw OrderError.tooManyItems
        }
    }
    
    private func calculatePricing(for order: Order) -> OrderPricing {
        let subtotal = calculateSubtotal(order.items)
        let discount = calculateDiscount(subtotal: subtotal)
        let tax = calculateTax(subtotal: subtotal, discount: discount)
        let shipping = calculateShipping(subtotal: subtotal)
        
        return OrderPricing(
            subtotal: subtotal,
            discount: discount,
            tax: tax,
            shipping: shipping,
            total: subtotal - discount + tax + shipping
        )
    }
    
    private func calculateSubtotal(_ items: [String]) -> Double {
        // items.reduce(0) { $0 + $1.price * Double($1.quantity) }
        return 0
    }
    
    private func calculateDiscount(subtotal: Double) -> Double {
        switch subtotal {
        case 1000...: return subtotal * 0.10
        case 500..<1000: return subtotal * 0.05
        default: return 0
        }
    }
    
    private func calculateTax(subtotal: Double, discount: Double) -> Double {
        let taxRate = 0.07
        return (subtotal - discount) * taxRate
    }
    
    private func calculateShipping(subtotal: Double) -> Double {
        return subtotal >= 500 ? 0 : 50
    }
    
    private func saveOrder(_ order: Order, with pricing: OrderPricing) throws {
        // database.save(order, pricing: pricing)
    }
    
    private func notifyCustomer(order: Order, pricing: OrderPricing) {
        // emailService.sendConfirmation(order: order, pricing: pricing)
    }
    
    private func updateInventory(for order: Order) {
        // inventory.update(for: order)
    }
}

struct OrderPricing {
    let subtotal: Double
    let discount: Double
    let tax: Double
    let shipping: Double
    let total: Double
}

enum OrderError: Error {
    case emptyOrder
    case tooManyItems
}
```

### 4.2 God Class

```swift
// ❌ BAD: God Class ที่รู้เรื่องทุกอย่าง
class AppManager {
    var currentUser: User?
    var cart: [String] = []
    var notifications: [String] = []
    var settings: [String: Any] = [:]
    var networkStatus: Bool = true
    
    func login() {}
    func logout() {}
    func addToCart(item: String) {}
    func removeFromCart(item: String) {}
    func checkout() {}
    func sendNotification() {}
    func updateSettings(key: String, value: Any) {}
    func fetchData() {}
    func cacheData() {}
    func parseJSON(_ data: Data) {}
}

// ✅ GOOD: แบ่ง responsibility ออก
class AuthManager {
    var currentUser: User?
    func login(email: String, password: String) async throws {}
    func logout() {}
    func refreshToken() async throws {}
}

class ShoppingCartManager {
    private var items: [CartItem] = []
    func add(item: CartItem) {}
    func remove(itemID: String) {}
    func clear() {}
    var total: Double { 0 }
}

class NotificationManager {
    func schedule(_ notification: LocalNotification) {}
    func cancel(id: String) {}
    func requestPermission() async -> Bool { return true }
}

class SettingsManager {
    func get<T>(_ key: String, default defaultValue: T) -> T { return defaultValue }
    func set(_ value: Any, for key: String) {}
    func reset() {}
}

struct CartItem {
    let id: String
    let name: String
    let price: Double
    let quantity: Int
}

struct LocalNotification {
    let id: String
    let title: String
    let body: String
    let triggerDate: Date
}
```

---

## 5. SwiftLint Configuration

```yaml
# .swiftlint.yml

# เปิดใช้ rules เพิ่มเติม
opt_in_rules:
  - array_init
  - closure_spacing
  - collection_alignment
  - contains_over_filter_count
  - contains_over_filter_is_empty
  - contains_over_first_not_nil
  - contains_over_range_nil_comparison
  - empty_collection_literal
  - empty_count
  - empty_string
  - enum_case_associated_values_count
  - explicit_init
  - extension_access_modifier
  - fallthrough
  - fatal_error_message
  - file_header
  - first_where
  - force_unwrapping
  - identical_operands
  - last_where
  - legacy_multiple
  - literal_expression_end_indentation
  - lower_acl_than_parent
  - modifier_order
  - multiline_arguments
  - multiline_function_chains
  - multiline_parameters
  - nimble_operator
  - nslocalizedstring_key
  - number_separator
  - object_literal
  - operator_usage_whitespace
  - overridden_super_call
  - pattern_matching_keywords
  - prefer_self_type_over_type_of_self
  - private_action
  - private_outlet
  - prohibited_super_call
  - quick_discouraged_call
  - redundant_nil_coalescing
  - redundant_type_annotation
  - single_test_class
  - sorted_first_last
  - static_operator
  - strong_iboutlet
  - toggle_bool
  - trailing_closure
  - unneeded_parentheses_in_closure_argument
  - vertical_parameter_alignment_on_call
  - yoda_condition

# ปิด rules ที่ไม่ต้องการ
disabled_rules:
  - todo
  - nesting

# Custom rules
custom_rules:
  no_print_in_production:
    included: ".*\\.swift"
    excluded: ".*Tests\\.swift"
    name: "No print statements"
    regex: "print\\("
    message: "ใช้ logger แทน print() ใน production code"
    severity: warning

  no_force_cast:
    name: "Avoid force cast"
    regex: "as!"
    message: "หลีกเลี่ยงการใช้ force cast"
    severity: error

# Rule configuration
line_length:
  warning: 120
  error: 200
  ignores_comments: true
  ignores_urls: true

function_body_length:
  warning: 50
  error: 100

type_body_length:
  warning: 300
  error: 500

file_length:
  warning: 500
  error: 1000

cyclomatic_complexity:
  warning: 10
  error: 20

nesting:
  type_level:
    warning: 3
  function_level:
    warning: 5

identifier_name:
  min_length:
    error: 2
  max_length:
    warning: 40
    error: 60
  excluded:
    - id
    - i
    - j
    - x
    - y
    - z

type_name:
  min_length:
    error: 3
  max_length:
    warning: 50
    error: 60

# Included/Excluded paths
included:
  - Sources
  - App

excluded:
  - Pods
  - .build
  - DerivedData
  - fastlane
  - vendor
```

```swift
// SwiftLint Custom Rules ใน Swift
// สร้าง custom rule ด้วย Swift
// swiftlint:disable:next force_cast
let value = someValue as! String  // เว้นในกรณีจำเป็น

// swiftlint:disable:this line_length
let veryLongString = "This is a very long string that exceeds the line length limit but is necessary"

// Disable for a block
// swiftlint:disable force_unwrapping
let required = optionalValue!
let alsoRequired = anotherOptional!
// swiftlint:enable force_unwrapping
```

---

## 6. SwiftFormat

```swift
// .swiftformat configuration file

// Indentation
--indent 4
--indentcase false
--trimwhitespace always
--insertlines enabled
--removelines enabled
--allman false  // K&R style braces

// Spacing
--operatorfunc spaced
--ranges spaced
--closingparen balanced
--commas always
--semicolons never

// Wrapping
--wraparguments before-first
--wrapparameters before-first
--wrapcollections before-first
--maxwidth 120

// Syntax
--self insert  // explicit self
--importgrouping testable-last
--redundanttype infer-locals-only
--voidtype void

// Rules
--enable isEmpty
--enable braces
--enable indent
--enable linebreaks
--enable redundantSelf
--enable sortImports
--enable spaceAroundOperators
--enable trailingClosures
--disable unusedArguments
```

```swift
// SwiftFormat ตัวอย่างการจัดรูปแบบ

// ❌ Before SwiftFormat
import Foundation
import UIKit
import SwiftUI
struct MyView:View{
var title:String
var count:Int=0
var body:some View{
VStack{
Text(title).font(.headline)
Button(action:{count+=1}){
Text("Tap me")
}
}
}
}

// ✅ After SwiftFormat
import Foundation
import SwiftUI
import UIKit

struct MyView: View {
    var title: String
    var count: Int = 0

    var body: some View {
        VStack {
            Text(title)
                .font(.headline)
            Button {
                count += 1
            } label: {
                Text("Tap me")
            }
        }
    }
}
```

---

## 7. Code Review Best Practices

### 7.1 Pull Request Template

```markdown
## สรุปการเปลี่ยนแปลง
<!-- อธิบายสั้นๆ ว่าทำอะไรและทำไม -->

## ประเภทของการเปลี่ยนแปลง
- [ ] Bug fix (แก้ bug ที่มีอยู่)
- [ ] New feature (เพิ่มฟีเจอร์ใหม่)
- [ ] Breaking change (เปลี่ยนแปลงที่ส่งผลกระทบต่อ functionality เดิม)
- [ ] Documentation update
- [ ] Refactoring
- [ ] Performance improvement

## การทดสอบ
- [ ] Unit tests ผ่าน
- [ ] Integration tests ผ่าน
- [ ] Manual testing ทำแล้ว

## Screenshots (ถ้ามี UI changes)
<!-- แนบ before/after screenshots -->

## Checklist
- [ ] โค้ดตาม style guide
- [ ] Self-reviewed แล้ว
- [ ] Comments เพิ่มในส่วนที่ซับซ้อน
- [ ] Documentation update แล้ว (ถ้าจำเป็น)
- [ ] ไม่มี breaking changes หรือ documented แล้ว

## Related Issues
Closes #123
```

### 7.2 Code Review Guidelines สำหรับ Reviewer

```swift
// สิ่งที่ต้องตรวจสอบใน Code Review

// 1. Correctness
// ❓ โค้ดทำงานถูกต้องตาม requirement หรือไม่?
// ❓ Edge cases ได้รับการจัดการหรือไม่?
// ❓ Error handling ครบถ้วนหรือไม่?

// ตัวอย่าง: ตรวจสอบ nil handling
// ❌ Reviewable issue
func getUser(id: String) -> User {
    return users[id]!  // Force unwrap - อาจ crash
}

// ✅ Better approach
func getUser(id: String) -> User? {
    return users[id]
}

// 2. Performance
// ❓ มี unnecessary computation หรือไม่?
// ❓ Memory leaks?

// ❌ Reviewable issue
class MyViewController: UIViewController {
    var timer: Timer?
    
    func startTimer() {
        // Timer ที่ไม่ถูก invalidate - memory leak
        Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { _ in
            self.updateUI()  // Strong reference cycle!
        }
    }
    
    func updateUI() { }
}

// ✅ Fixed version
class MyViewControllerFixed: UIViewController {
    var timer: Timer?
    
    func startTimer() {
        timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { [weak self] _ in
            self?.updateUI()
        }
    }
    
    override func viewDidDisappear(_ animated: Bool) {
        super.viewDidDisappear(animated)
        timer?.invalidate()
        timer = nil
    }
    
    func updateUI() { }
}

// 3. Security
// ❓ Sensitive data ถูก log หรือไม่?
// ❌
print("User password: \(user.password)")

// ✅
print("User logged in: \(user.id)")

// 4. Testability
// ❓ Code สามารถ test ได้ง่ายหรือไม่?
var users: [String: User] = [:]
```

---

## 8. Documentation ด้วย DocC

### 8.1 DocC Comments

```swift
/// # Networking Module
///
/// จัดการ HTTP requests ทั้งหมดของ Application
///
/// ## Overview
///
/// `NetworkClient` เป็น centralized HTTP client ที่รองรับ:
/// - Async/Await
/// - Automatic retry
/// - Request/Response logging
/// - Error handling
///
/// ## Topics
///
/// ### Getting Started
/// - ``NetworkClient/shared``
/// - ``NetworkClient/request(_:)``
///
/// ### Configuration
/// - ``NetworkConfiguration``
/// - ``NetworkClient/init(configuration:)``
///
/// ## Usage
///
/// ```swift
/// let client = NetworkClient.shared
/// let user: User = try await client.request(.getUser(id: "123"))
/// ```

/// HTTP Client สำหรับ Application
///
/// ใช้ `shared` instance สำหรับทั้ง application เว้นแต่ต้องการ configuration พิเศษ
///
/// - Note: Thread-safe สำหรับ concurrent requests
/// - Important: ต้อง configure base URL ก่อนใช้งาน
/// - Warning: ไม่ควรใช้ใน background thread โดยตรง ใช้ async/await แทน
public class NetworkClient {
    
    /// Shared singleton instance
    ///
    /// ใช้ instance นี้ใน production code เสมอ
    public static let shared = NetworkClient()
    
    /// สร้าง NetworkClient ด้วย custom configuration
    ///
    /// - Parameter configuration: การตั้งค่า network behavior
    ///
    /// ## Example
    /// ```swift
    /// let config = NetworkConfiguration(
    ///     baseURL: URL(string: "https://api.example.com")!,
    ///     timeout: 30
    /// )
    /// let client = NetworkClient(configuration: config)
    /// ```
    public init(configuration: NetworkConfiguration = .default) {
        self.configuration = configuration
    }
    
    private let configuration: NetworkConfiguration
    
    /// ส่ง HTTP request และ decode response
    ///
    /// - Parameter endpoint: Endpoint ที่ต้องการเรียก
    /// - Returns: Decoded response ประเภท T
    /// - Throws:
    ///   - `NetworkError.noConnection` ถ้าไม่มีอินเทอร์เน็ต
    ///   - `NetworkError.timeout` ถ้า request หมดเวลา
    ///   - `NetworkError.serverError(Int)` ถ้า server ตอบกลับ error status
    ///   - `DecodingError` ถ้า response format ไม่ตรง
    ///
    /// ## Example
    ///
    /// ```swift
    /// do {
    ///     let users: [User] = try await client.request(.getUsers)
    ///     print("Got \(users.count) users")
    /// } catch NetworkError.noConnection {
    ///     print("ไม่มีอินเทอร์เน็ต")
    /// } catch {
    ///     print("Error: \(error)")
    /// }
    /// ```
    public func request<T: Decodable>(_ endpoint: Endpoint) async throws -> T {
        // Implementation
        fatalError("Not implemented")
    }
}

/// Configuration สำหรับ NetworkClient
public struct NetworkConfiguration {
    /// Base URL ของ API
    public let baseURL: URL
    
    /// Timeout ในหน่วยวินาที (default: 30)
    public let timeout: TimeInterval
    
    /// จำนวนครั้งที่ retry เมื่อ request ล้มเหลว (default: 3)
    public let maxRetries: Int
    
    /// Default configuration
    public static let `default` = NetworkConfiguration(
        baseURL: URL(string: "https://api.example.com")!,
        timeout: 30,
        maxRetries: 3
    )
    
    public init(baseURL: URL, timeout: TimeInterval = 30, maxRetries: Int = 3) {
        self.baseURL = baseURL
        self.timeout = timeout
        self.maxRetries = maxRetries
    }
}

struct Endpoint {
    let path: String
    let method: String
    static func getUser(id: String) -> Endpoint {
        Endpoint(path: "/users/\(id)", method: "GET")
    }
    static let getUsers = Endpoint(path: "/users", method: "GET")
}

enum NetworkError: Error {
    case noConnection
    case timeout
    case serverError(Int)
}
```

### 8.2 Documentation Catalog

```
// Structure ของ DocC Documentation Catalog
MyApp.docc/
├── MyApp.md          // Landing page
├── GettingStarted.md
├── Resources/
│   ├── networking-flow.png
│   └── architecture.png
└── Tutorials/
    └── BuildYourFirstFeature.tutorial
```

```markdown
# ``MyApp``

เครื่องมือ productivity สำหรับทีมพัฒนา Swift

## Overview

MyApp ช่วยให้ทีมของคุณ:
- จัดการ tasks อย่างมีประสิทธิภาพ
- Collaborate แบบ real-time
- Track progress ด้วย analytics

## Topics

### Essentials

- <doc:GettingStarted>
- ``NetworkClient``
- ``AuthenticationService``

### Data Management

- ``UserRepository``
- ``CacheManager``

### UI Components

- ``ComponentLibrary``
```

---

## 9. Code Style Guide

### 9.1 Naming Conventions

```swift
// Apple Swift API Design Guidelines

// Types: UpperCamelCase
struct UserProfile {}
class NetworkManager {}
enum ConnectionStatus {}
protocol Authenticatable {}

// Variables, Functions: lowerCamelCase
var userCount = 0
let maxRetries = 3
func fetchUserData() {}
var isLoading = false

// Constants: lowerCamelCase (ไม่ใช้ k prefix แบบ Objective-C)
let defaultTimeout = 30.0
let maximumItemCount = 100

// Acronyms: UpperCase เมื่อเป็น prefix ทั้งหมด
var url: URL
var htmlContent: String
let userID: String  // ไม่ใช่ userId
var httpStatusCode: Int
let apiKey: String

// Boolean: ควรขึ้นต้นด้วย is, has, can, should, will
var isEnabled = true
var hasLoadedData = false
var canSubmit = true
var shouldRefresh = false
var willAnimateTransition = true

// Protocol methods ที่ return Bool
protocol Validatable {
    func isValid() -> Bool  // ✅
    func validate() -> Bool  // ❌ ไม่ชัดเจน
}

// Closures: สื่อความหมาย
let sortByName: (User, User) -> Bool = { $0.name < $1.name }
let filterActive = { (user: User) in user.isActive }

// Type aliases: ชัดเจน
typealias CompletionHandler = (Result<Data, Error>) -> Void
typealias UserID = String
typealias Price = Double
```

### 9.2 File Organization

```swift
// File organization ที่ดี

// MARK: - Imports (เรียง alphabetically)
import Combine
import Foundation
import SwiftUI
import UIKit

// MARK: - Type Declaration
class UserProfileViewModel: ObservableObject {
    
    // MARK: - Types (nested types ก่อน)
    enum State {
        case idle
        case loading
        case loaded(UserProfile)
        case error(Error)
    }
    
    // MARK: - Published Properties
    @Published var state: State = .idle
    @Published var profile: UserProfile?
    
    // MARK: - Private Properties
    private let userService: UserService
    private var cancellables = Set<AnyCancellable>()
    
    // MARK: - Computed Properties
    var isLoading: Bool {
        if case .loading = state { return true }
        return false
    }
    
    // MARK: - Initialization
    init(userService: UserService = UserService()) {
        self.userService = userService
    }
    
    // MARK: - Public Methods
    func loadProfile(userID: String) {
        state = .loading
        Task {
            await fetchProfile(userID: userID)
        }
    }
    
    func refresh() {
        guard let profile = profile else { return }
        loadProfile(userID: profile.id)
    }
    
    // MARK: - Private Methods
    private func fetchProfile(userID: String) async {
        do {
            let profile = try await userService.getProfile(userID: userID)
            await MainActor.run {
                self.profile = profile
                self.state = .loaded(profile)
            }
        } catch {
            await MainActor.run {
                self.state = .error(error)
            }
        }
    }
}

// MARK: - Extensions
extension UserProfileViewModel: Equatable {
    static func == (lhs: UserProfileViewModel, rhs: UserProfileViewModel) -> Bool {
        return lhs.profile?.id == rhs.profile?.id
    }
}

struct UserProfile {
    let id: String
    let name: String
    let email: String
}

class UserService {
    func getProfile(userID: String) async throws -> UserProfile {
        return UserProfile(id: userID, name: "Test", email: "test@test.com")
    }
}
```

---

## 10. Module Organization

```swift
// Package.swift สำหรับ Multi-module Architecture

// swift-tools-version:5.9
import PackageDescription

let package = Package(
    name: "MyApp",
    platforms: [
        .iOS(.v16),
        .macOS(.v13)
    ],
    products: [
        .library(name: "Core", targets: ["Core"]),
        .library(name: "UI", targets: ["UI"]),
        .library(name: "Network", targets: ["Network"]),
        .library(name: "Domain", targets: ["Domain"])
    ],
    dependencies: [
        .package(url: "https://github.com/pointfreeco/swift-composable-architecture", from: "1.0.0"),
        .package(url: "https://github.com/realm/SwiftLint", from: "0.54.0")
    ],
    targets: [
        // Core - Foundation layer
        .target(
            name: "Core",
            dependencies: [],
            path: "Sources/Core"
        ),
        
        // Domain - Business logic
        .target(
            name: "Domain",
            dependencies: ["Core"],
            path: "Sources/Domain"
        ),
        
        // Network - Data layer
        .target(
            name: "Network",
            dependencies: ["Core", "Domain"],
            path: "Sources/Network"
        ),
        
        // UI - Presentation layer
        .target(
            name: "UI",
            dependencies: ["Core", "Domain"],
            path: "Sources/UI"
        ),
        
        // Tests
        .testTarget(
            name: "CoreTests",
            dependencies: ["Core"],
            path: "Tests/CoreTests"
        ),
        .testTarget(
            name: "DomainTests",
            dependencies: ["Domain"],
            path: "Tests/DomainTests"
        )
    ]
)

/*
Directory Structure:
Sources/
├── Core/           # Foundation utilities, extensions
│   ├── Extensions/
│   ├── Utilities/
│   └── Models/
├── Domain/         # Business logic
│   ├── UseCases/
│   ├── Repositories/
│   └── Models/
├── Network/        # API, data fetching
│   ├── API/
│   ├── Models/
│   └── Repositories/
└── UI/             # Views, ViewModels
    ├── Components/
    ├── Screens/
    └── ViewModels/
*/
```

---

## 11. Git Workflow

### 11.1 GitFlow

```bash
# GitFlow branches:
# main        - Production-ready code
# develop     - Integration branch
# feature/*   - New features
# release/*   - Release preparation
# hotfix/*    - Production bug fixes

# เริ่ม feature ใหม่
git checkout develop
git pull origin develop
git checkout -b feature/add-payment-feature

# ทำงานและ commit
git add -A
git commit -m "feat: add PromptPay payment option"

# Push และสร้าง PR
git push origin feature/add-payment-feature

# เมื่อ feature เสร็จ merge กลับ develop
git checkout develop
git merge --no-ff feature/add-payment-feature
git push origin develop

# สร้าง release branch
git checkout -b release/1.2.0 develop

# แก้ bug เล็กน้อยและ update version
git commit -m "chore: bump version to 1.2.0"

# Merge to main และ tag
git checkout main
git merge --no-ff release/1.2.0
git tag -a v1.2.0 -m "Release version 1.2.0"
git push origin main --tags

# Merge back to develop
git checkout develop
git merge --no-ff release/1.2.0
```

### 11.2 Trunk-Based Development

```bash
# Trunk-Based Development:
# main  - Single integration branch (trunk)
# Short-lived feature branches (< 2 days)

# Feature flag สำหรับ incomplete features
# git checkout -b feat/new-ui-experiment
# ทำงาน, commit บ่อยๆ (ทุกชั่วโมง)
# สร้าง PR เล็กๆ บ่อยๆ แทน PR ใหญ่นานๆ ครั้ง
# Continuous Integration ทุก commit

# Workflow
git checkout main
git pull origin main
git checkout -b feat/quick-fix-123

# ทำงาน
git add specific-file.swift
git commit -m "fix: resolve nil crash in user profile loading

Fixes crash when user.address is nil on first load.
Added nil coalescing to safely handle missing address data.

Closes #123"

git push origin feat/quick-fix-123
# สร้าง PR → Review → Merge ภายใน 1-2 วัน
```

### 11.3 Commit Message Best Practices

```bash
# Format: <type>(<scope>): <subject>
# 
# <body>
#
# <footer>

# Types:
# feat     - ฟีเจอร์ใหม่
# fix      - Bug fix
# docs     - Documentation
# style    - Formatting, missing semi colons, etc
# refactor - Code restructuring
# test     - Adding tests
# chore    - Maintenance tasks
# perf     - Performance improvements
# ci       - CI/CD changes
# build    - Build system changes
# revert   - Revert previous commit

# ตัวอย่างที่ดี
git commit -m "feat(auth): add biometric authentication support

Added Face ID and Touch ID support for login screen.
Falls back to PIN if biometric is unavailable.

- Implements LocalAuthentication framework
- Added proper error handling for biometric failures  
- Updated privacy description in Info.plist

Closes #456"

git commit -m "fix(network): resolve race condition in concurrent requests

Multiple concurrent requests to same endpoint could result in
duplicate data being saved to cache. Added serial queue for
cache writes to prevent data corruption.

Fixes #789"

# ตัวอย่างที่ไม่ดี
git commit -m "fix bug"
git commit -m "update stuff"
git commit -m "wip"
git commit -m "changes"
```

---

## 12. Tech Debt Management

```swift
// เครื่องมือ track tech debt

// 1. TODO/FIXME Comments with metadata
// TODO(JIRA-1234, @owner, priority: high): Implement proper caching
// FIXME(JIRA-5678, @john): Memory leak in delegate pattern
// HACK(JIRA-9012): Workaround for iOS 15 bug, remove after iOS 17 is minimum

// 2. Technical Debt Register (ใน notion/confluence/jira)
/*
| ID    | Description          | File              | Priority | Effort | Owner |
|-------|---------------------|-------------------|----------|--------|-------|
| TD-1  | Replace UserDefaults with proper DB | UserSettings.swift | High | M | John |
| TD-2  | Remove legacy API v1 | APIv1Client.swift | Medium   | L | Jane |
| TD-3  | Add proper pagination | ProductList.swift | Low      | S | Bob  |
*/

// 3. Deprecation marking
extension UserDefaults {
    /// ⚠️ Deprecated: ใช้ `SecureStorage.shared` แทน
    /// จะถูกลบใน version 3.0.0
    @available(*, deprecated, message: "Use SecureStorage.shared instead")
    static func legacySaveUser(_ user: User) {
        // Legacy implementation
    }
}

// 4. Architecture Decision Records (ADR)
/*
ADR-001: Use SwiftUI over UIKit for new screens
Date: 2024-01-15
Status: Accepted

Context:
ทีมต้องตัดสินใจว่าจะใช้ SwiftUI หรือ UIKit สำหรับ features ใหม่

Decision:
ใช้ SwiftUI สำหรับ screens ใหม่ทั้งหมด แต่คงเดิม UIKit screens ที่มีอยู่

Consequences:
+ Faster development
+ Better previews
+ Less boilerplate
- ต้องเรียนรู้ใหม่
- บาง features ยังไม่รองรับใน SwiftUI
*/
```

---

## 13. Refactoring Techniques

### 13.1 Rename Refactoring

```swift
// ❌ Before: ชื่อไม่ชัดเจน
class DataHandler {
    func proc(_ x: [String: Any]) -> Bool {
        return validate(x)
    }
    
    private func validate(_ data: [String: Any]) -> Bool {
        return data["name"] != nil
    }
}

// ✅ After: ชื่อชัดเจน
class UserRegistrationHandler {
    func processRegistration(_ formData: [String: Any]) -> Bool {
        return isFormDataValid(formData)
    }
    
    private func isFormDataValid(_ formData: [String: Any]) -> Bool {
        return formData["name"] != nil
    }
}
```

### 13.2 Extract Class Refactoring

```swift
// ❌ Before: Class ที่ใหญ่เกินไป
class Order {
    var customerName: String = ""
    var customerEmail: String = ""
    var customerAddress: String = ""
    var customerPhone: String = ""
    
    var items: [String] = []
    var totalAmount: Double = 0
    var discountCode: String = ""
    
    var shippingCarrier: String = ""
    var shippingMethod: String = ""
    var estimatedDelivery: Date = Date()
    var trackingNumber: String = ""
    
    func formatCustomerAddress() -> String { "" }
    func validateCustomerEmail() -> Bool { true }
    func calculateDiscount() -> Double { 0 }
    func calculateShipping() -> Double { 0 }
    func estimateDeliveryDate() -> Date { Date() }
}

// ✅ After: Extract separate classes
struct CustomerInfo {
    var name: String
    var email: String
    var address: String
    var phone: String
    
    func formattedAddress() -> String { "" }
    func isEmailValid() -> Bool { true }
}

struct OrderItems {
    var items: [String]
    var discountCode: String?
    
    var subtotal: Double { 0 }
    func calculateDiscount() -> Double { 0 }
    var total: Double { subtotal - calculateDiscount() }
}

struct ShippingInfo {
    var carrier: String
    var method: String
    var trackingNumber: String?
    
    func estimateDeliveryDate() -> Date { Date() }
    func calculateCost() -> Double { 0 }
}

struct RefactoredOrder {
    let id: String
    var customer: CustomerInfo
    var items: OrderItems
    var shipping: ShippingInfo
    
    var grandTotal: Double {
        items.total + shipping.calculateCost()
    }
}
```

### 13.3 Introduce Parameter Object

```swift
// ❌ Before: Function ที่มี parameters มากเกินไป
func searchProducts(
    query: String,
    minPrice: Double,
    maxPrice: Double,
    category: String?,
    brand: String?,
    rating: Int?,
    inStock: Bool,
    sortBy: String,
    ascending: Bool,
    page: Int,
    pageSize: Int
) -> [Product] {
    return []
}

// ✅ After: Parameter Object
struct ProductSearchCriteria {
    let query: String
    let priceRange: ClosedRange<Double>?
    let category: String?
    let brand: String?
    let minimumRating: Int?
    let inStockOnly: Bool
    let sortOrder: SortOrder
    let pagination: Pagination
    
    enum SortOrder {
        case priceAscending
        case priceDescending
        case rating
        case relevance
    }
    
    struct Pagination {
        let page: Int
        let pageSize: Int
        
        static let `default` = Pagination(page: 1, pageSize: 20)
    }
    
    static func builder() -> Builder {
        return Builder()
    }
    
    class Builder {
        private var query = ""
        private var priceRange: ClosedRange<Double>?
        private var category: String?
        private var brand: String?
        private var minimumRating: Int?
        private var inStockOnly = false
        private var sortOrder: SortOrder = .relevance
        private var pagination: Pagination = .default
        
        func query(_ q: String) -> Builder {
            query = q; return self
        }
        
        func priceRange(_ range: ClosedRange<Double>) -> Builder {
            priceRange = range; return self
        }
        
        func inStockOnly(_ only: Bool) -> Builder {
            inStockOnly = only; return self
        }
        
        func sortBy(_ order: SortOrder) -> Builder {
            sortOrder = order; return self
        }
        
        func build() -> ProductSearchCriteria {
            return ProductSearchCriteria(
                query: query,
                priceRange: priceRange,
                category: category,
                brand: brand,
                minimumRating: minimumRating,
                inStockOnly: inStockOnly,
                sortOrder: sortOrder,
                pagination: pagination
            )
        }
    }
}

// ใช้งาน
func searchProducts(_ criteria: ProductSearchCriteria) -> [Product] {
    return []
}

let criteria = ProductSearchCriteria.builder()
    .query("iPhone case")
    .priceRange(100...500)
    .inStockOnly(true)
    .sortBy(.priceAscending)
    .build()

let results = searchProducts(criteria)

struct Product { let id: String }
```

---

## 14. Pair Programming

```swift
// Pair Programming Patterns

// 1. Driver-Navigator Pattern
// Driver: เขียนโค้ด
// Navigator: review, suggest improvements, think ahead

// 2. Ping-Pong TDD
// Person A เขียน failing test
// Person B เขียน implementation ให้ผ่าน
// Person A refactor และเขียน test ต่อไป
// สลับกันไปเรื่อยๆ

// ตัวอย่าง Ping-Pong TDD:

// Round 1 - Person A เขียน failing test:
// func test_emptyCart_returnsZeroTotal() {
//     let cart = ShoppingCart()
//     XCTAssertEqual(cart.total, 0)
// }

// Round 1 - Person B implement:
struct ShoppingCart {
    private var items: [CartItem2] = []
    
    var total: Double {
        items.reduce(0) { $0 + $1.subtotal }
    }
    
    mutating func add(_ item: CartItem2) {
        if let index = items.firstIndex(where: { $0.productID == item.productID }) {
            items[index] = CartItem2(
                productID: item.productID,
                price: item.price,
                quantity: items[index].quantity + item.quantity
            )
        } else {
            items.append(item)
        }
    }
    
    mutating func remove(productID: String) {
        items.removeAll { $0.productID == productID }
    }
    
    var itemCount: Int { items.count }
}

struct CartItem2 {
    let productID: String
    let price: Double
    let quantity: Int
    
    var subtotal: Double { price * Double(quantity) }
}

// Round 2 - Person A เขียน test ต่อ:
// func test_addItem_increasesTotal() {
//     var cart = ShoppingCart()
//     let item = CartItem2(productID: "1", price: 100, quantity: 2)
//     cart.add(item)
//     XCTAssertEqual(cart.total, 200)
// }
```

---

## 15. Code Mentorship

```swift
// Mentorship through Code Review

// สิ่งที่ Mentor ควรทำ:

// 1. ให้ feedback ที่ constructive
// ❌ BAD: "โค้ดนี้แย่มาก"
// ✅ GOOD: "เราสามารถ simplify ส่วนนี้ได้ด้วยการใช้ map() แทน for loop:
//
// จาก:
// var names = [String]()
// for user in users {
//     names.append(user.name)
// }
//
// เป็น:
// let names = users.map { $0.name }
"

// 2. อธิบาย WHY ไม่ใช่แค่ WHAT
// ❌: "ใช้ [weak self] ที่นี่"
// ✅: "ใช้ [weak self] ที่นี่เพื่อป้องกัน retain cycle
//     เมื่อ closure capture self, ARC จะเพิ่ม reference count ของ self
//     ถ้า self ก็ hold reference ถึง closure (ผ่าน timer/notification)
//     จะเกิด retain cycle ทำให้ทั้งคู่ไม่ถูก deallocate"

// 3. Pair บน code ที่ยาก
// แทนที่จะให้ mentee แก้คนเดียว ทำ pair programming session

// 4. Review ด้วยคำถาม ไม่ใช่คำสั่ง
// ❌: "เปลี่ยนเป็น optional"
// ✅: "ถ้า user เป็น nil จะเกิดอะไรขึ้น? เราควร handle ยังไง?"

// 5. Celebrate improvements
// "โค้ดนี้ดีขึ้นมากจาก PR ที่แล้ว!"
// "เห็นว่าใช้ Result type ได้อย่างถูกต้องแล้ว 👍"
```

---

## 16. Open Source Contribution

```swift
// การ Contribute ให้ Open Source Projects

// 1. เริ่มจาก small issues (good first issue)
// 2. อ่าน CONTRIBUTING.md ก่อนเสมอ
// 3. สร้าง fork และทำงานใน branch ของตัวเอง

// ตัวอย่าง: Contribute bug fix
// - Fork repo
// git clone https://github.com/yourusername/open-source-lib.git
// git checkout -b fix/nil-crash-in-parser

// เขียน test ก่อน reproduce bug:
import XCTest

class ParserTests: XCTestCase {
    
    // Reproduce the bug first
    func test_parseEmptyString_shouldNotCrash() {
        let parser = JSONParser()
        // ก่อนหน้านี้ crash ตรงนี้
        XCTAssertNil(parser.parse(""))
    }
    
    func test_parseValidJSON_returnsObject() {
        let parser = JSONParser()
        let json = "{\"name\": \"John\"}"
        let result = parser.parse(json)
        XCTAssertNotNil(result)
        XCTAssertEqual(result?["name"] as? String, "John")
    }
}

// Fix the bug:
struct JSONParser {
    func parse(_ string: String) -> [String: Any]? {
        guard !string.isEmpty,
              let data = string.data(using: .utf8) else {
            return nil
        }
        
        return try? JSONSerialization.jsonObject(
            with: data,
            options: []
        ) as? [String: Any]
    }
}

// PR Description:
/*
## Bug Fix: Nil crash when parsing empty string

### Problem
Parsing an empty string caused an NSException crash instead of returning nil.

### Solution  
Added guard to check for empty string before attempting to parse.

### Tests
Added test cases for:
- Empty string (should return nil, not crash)
- Valid JSON (should return dictionary)
- Invalid JSON (should return nil)

Fixes #1234
*/
```

---

## 17. แบบฝึกหัดพร้อมเฉลย

### แบบฝึกหัดที่ 1: Refactor God Class

**โจทย์:** Refactor class นี้ให้ตาม Single Responsibility Principle

```swift
// Original Code (มี issues หลายอย่าง)
class ProductManager {
    var products: [ProductItem] = []
    var cart: [ProductItem] = []
    var currentUser: UserAccount?
    
    func fetchProducts() async {}
    func addToCart(_ product: ProductItem) {}
    func removeFromCart(_ product: ProductItem) {}
    func checkout() {}
    func login(email: String, password: String) {}
    func logout() {}
    func formatPrice(_ price: Double) -> String { "" }
    func sendReceiptEmail() {}
}

// เฉลย: แยกเป็น separate classes ตาม responsibility
struct ProductItem {
    let id: String
    let name: String
    let price: Double
}

struct UserAccount {
    let id: String
    let email: String
}

// Responsibility 1: Product Catalog
class ProductCatalog: ObservableObject {
    @Published private(set) var products: [ProductItem] = []
    
    func fetchProducts() async {
        // Fetch from API
    }
    
    func search(query: String) -> [ProductItem] {
        products.filter { $0.name.localizedCaseInsensitiveContains(query) }
    }
}

// Responsibility 2: Shopping Cart
class CartManager: ObservableObject {
    @Published private(set) var items: [ProductItem] = []
    
    var total: Double {
        items.reduce(0) { $0 + $1.price }
    }
    
    func add(_ product: ProductItem) {
        items.append(product)
    }
    
    func remove(_ product: ProductItem) {
        items.removeAll { $0.id == product.id }
    }
    
    func clear() {
        items.removeAll()
    }
}

// Responsibility 3: Checkout
class CheckoutService {
    func checkout(items: [ProductItem], user: UserAccount) async throws -> String {
        // Process payment, return order ID
        return UUID().uuidString
    }
}

// Responsibility 4: Authentication
class AuthenticationManager: ObservableObject {
    @Published private(set) var currentUser: UserAccount?
    
    func login(email: String, password: String) async throws {
        // Authenticate
        currentUser = UserAccount(id: UUID().uuidString, email: email)
    }
    
    func logout() {
        currentUser = nil
    }
}

// Responsibility 5: Formatting (utility)
struct PriceFormatter {
    static func format(_ price: Double, currency: String = "THB") -> String {
        let formatter = NumberFormatter()
        formatter.numberStyle = .currency
        formatter.currencyCode = currency
        return formatter.string(from: NSNumber(value: price)) ?? "\(price)"
    }
}

// Responsibility 6: Notifications
class OrderNotificationService {
    func sendReceiptEmail(to email: String, orderID: String) async throws {
        // Send email
    }
}
```

### แบบฝึกหัดที่ 2: SOLID Principles

**โจทย์:** เขียน notification system ที่ตาม SOLID principles

```swift
// เฉลย: SOLID Notification System

// Interface Segregation
protocol NotificationSendable {
    func send(_ notification: Notification2) async throws
}

protocol NotificationSchedulable {
    func schedule(_ notification: Notification2, at date: Date) throws
}

protocol NotificationCancellable {
    func cancel(id: String)
}

// Open/Closed - Notification types
struct Notification2 {
    let id: String
    let title: String
    let body: String
    let data: [String: String]
    
    init(id: String = UUID().uuidString, title: String, body: String, data: [String: String] = [:]) {
        self.id = id
        self.title = title
        self.body = body
        self.data = data
    }
}

// Concrete implementations
class PushNotificationSender: NotificationSendable {
    func send(_ notification: Notification2) async throws {
        print("Sending push: \(notification.title)")
    }
}

class EmailNotificationSender: NotificationSendable {
    private let recipientEmail: String
    
    init(email: String) {
        self.recipientEmail = email
    }
    
    func send(_ notification: Notification2) async throws {
        print("Sending email to \(recipientEmail): \(notification.title)")
    }
}

class SMSNotificationSender: NotificationSendable {
    private let phoneNumber: String
    
    init(phone: String) {
        self.phoneNumber = phone
    }
    
    func send(_ notification: Notification2) async throws {
        print("Sending SMS to \(phoneNumber): \(notification.body)")
    }
}

// Dependency Inversion - High-level coordinator depends on abstraction
class NotificationCoordinator {
    private let senders: [NotificationSendable]
    
    init(senders: [NotificationSendable]) {
        self.senders = senders
    }
    
    func broadcast(_ notification: Notification2) async {
        await withTaskGroup(of: Void.self) { group in
            for sender in senders {
                group.addTask {
                    try? await sender.send(notification)
                }
            }
        }
    }
}

// Usage
let coordinator = NotificationCoordinator(senders: [
    PushNotificationSender(),
    EmailNotificationSender(email: "user@example.com"),
    SMSNotificationSender(phone: "+66891234567")
])

Task {
    await coordinator.broadcast(Notification2(
        title: "คำสั่งซื้อของคุณพร้อมแล้ว",
        body: "คำสั่งซื้อ #12345 จัดส่งแล้ว"
    ))
}
```

### แบบฝึกหัดที่ 3: SwiftLint Configuration

**โจทย์:** สร้าง SwiftLint configuration สำหรับ team ที่มี strict quality standards

```yaml
# เฉลย: .swiftlint.yml สำหรับ team

opt_in_rules:
  - force_unwrapping
  - empty_count
  - empty_string
  - fatal_error_message
  - first_where
  - last_where
  - modifier_order
  - redundant_type_annotation
  - toggle_bool
  - yoda_condition
  - contains_over_filter_count

disabled_rules:
  - identifier_name  # ตั้งค่าเองใน custom section

custom_rules:
  no_direct_print:
    name: "No print()"
    regex: "(?<!//\\s*)\\bprint\\("
    message: "ใช้ AppLogger แทน print()"
    severity: warning
    excluded: ".*Tests\\.swift"
  
  force_unwrap_optional:
    name: "Avoid force unwrap"
    regex: "\\w+!"
    message: "ใช้ optional binding แทน force unwrap"
    severity: error
    excluded: ".*Tests\\.swift"

line_length:
  warning: 120
  error: 200

function_body_length:
  warning: 40
  error: 80

type_body_length:
  warning: 250
  error: 400

file_length:
  warning: 400
  error: 800

cyclomatic_complexity:
  warning: 8
  error: 15

nesting:
  type_level:
    warning: 2
  function_level:
    warning: 3

identifier_name:
  min_length:
    error: 2
  max_length:
    warning: 35
    error: 50
  excluded:
    - id
    - i
    - j
    - x
    - y

excluded:
  - Pods
  - .build
  - DerivedData
  - Generated
```

---

## สรุป

ในบทนี้เราได้เรียนรู้ทุกด้านของ Code Quality:

1. **Clean Code Principles** - ชื่อที่ดี, function ที่สั้น, comments ที่มีคุณค่า
2. **SOLID Principles** - SRP, OCP, LSP, ISP, DIP พร้อม Swift examples
3. **DRY และ KISS** - หลีกเลี่ยง repetition, ความเรียบง่าย
4. **Code Smells** - Long Method, God Class, Duplicate Code
5. **SwiftLint** - Automated code quality enforcement
6. **SwiftFormat** - Consistent code formatting
7. **Code Review** - Best practices สำหรับ reviewer และ author
8. **DocC** - Professional documentation
9. **Code Style Guide** - Naming, organization conventions
10. **Module Organization** - Multi-module architecture
11. **Git Workflow** - GitFlow และ Trunk-Based Development
12. **Commit Messages** - Conventional commits
13. **Tech Debt** - การจัดการและลด technical debt
14. **Refactoring** - Extract Method, Extract Class, Parameter Object
15. **Pair Programming** - Driver-Navigator, Ping-Pong TDD
16. **Code Mentorship** - ให้ feedback อย่าง constructive
17. **Open Source** - การ contribute ให้ community

การ master ทักษะเหล่านี้จะทำให้คุณเป็นนักพัฒนาที่ทีมต้องการ ไม่ใช่แค่คนที่เขียนโค้ดได้ แต่คนที่เขียนโค้ดที่คนอื่นอ่านและดูแลรักษาได้อย่างมีความสุข
