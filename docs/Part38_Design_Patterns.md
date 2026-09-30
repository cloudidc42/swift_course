# Part 38: Design Patterns ใน Swift

## สารบัญ

1. [บทนำสู่ Design Patterns](#1-บทนำสู่-design-patterns)
2. [Creational Patterns](#2-creational-patterns)
   - Singleton
   - Factory Method
   - Abstract Factory
   - Builder
   - Prototype
3. [Structural Patterns](#3-structural-patterns)
   - Adapter
   - Bridge
   - Composite
   - Decorator
   - Facade
   - Flyweight
   - Proxy
4. [Behavioral Patterns](#4-behavioral-patterns)
   - Chain of Responsibility
   - Command
   - Iterator
   - Mediator
   - Memento
   - Observer
   - State
   - Strategy
   - Template Method
   - Visitor
5. [Anti-Patterns](#5-anti-patterns)
6. [Swift-Specific Patterns](#6-swift-specific-patterns)
7. [Functional Patterns](#7-functional-patterns)
8. [แบบฝึกหัด](#8-แบบฝึกหัด)
9. [Real-world iOS Examples](#9-real-world-ios-examples)
10. [สรุป](#10-สรุป)

---

## 1. บทนำสู่ Design Patterns

Design Patterns คือแนวทางในการแก้ปัญหาที่พบบ่อยในการออกแบบซอฟต์แวร์ ซึ่งได้รับการพิสูจน์แล้วว่าใช้งานได้ดี โดย Gang of Four (GoF) ได้รวบรวม 23 Patterns ไว้ในหนังสือ "Design Patterns: Elements of Reusable Object-Oriented Software"

### ประเภทของ Design Patterns

| ประเภท | จำนวน | วัตถุประสงค์ |
|--------|-------|------------|
| Creational | 5 | การสร้าง Objects |
| Structural | 7 | การจัดโครงสร้าง Classes |
| Behavioral | 10 | การสื่อสารระหว่าง Objects |

### ทำไมต้องเรียน Design Patterns?

```swift
// ตัวอย่างปัญหาที่ Design Patterns แก้ได้

// ปัญหา: ต้องการ Object เดียวตลอด App (Singleton Problem)
// ปัญหา: สร้าง Object แตกต่างกันตาม Type (Factory Problem)
// ปัญหา: เพิ่ม Behavior โดยไม่แก้ Original Class (Decorator Problem)
// ปัญหา: แจ้งเตือนหลาย Objects เมื่อ State เปลี่ยน (Observer Problem)

// Design Patterns ให้:
// 1. Common Vocabulary - ทีมสื่อสารได้ตรงกัน
// 2. Proven Solutions - วิธีแก้ปัญหาที่ผ่านการทดสอบแล้ว
// 3. Code Quality - โค้ดสะอาดและ Maintainable
// 4. Reusability - นำกลับมาใช้ใหม่ได้
```

---

## 2. Creational Patterns

Creational Patterns เกี่ยวข้องกับกระบวนการสร้าง Objects

### 2.1 Singleton Pattern

Singleton รับประกันว่า Class จะมีเพียง Instance เดียว และให้ Global Access Point

```swift
// ✅ Singleton ที่ดี
final class AnalyticsManager {
    static let shared = AnalyticsManager()
    
    private var events: [String] = []
    private var userId: String?
    
    private init() {
        // Setup analytics SDK
    }
    
    func identify(userId: String) {
        self.userId = userId
    }
    
    func track(_ event: String, properties: [String: Any] = [:]) {
        events.append(event)
        var props = properties
        if let userId = userId {
            props["user_id"] = userId
        }
        // Send to analytics server
        print("Tracked: \(event) with \(props)")
    }
    
    func reset() {
        userId = nil
        events = []
    }
}

// การใช้งาน
AnalyticsManager.shared.identify(userId: "user_123")
AnalyticsManager.shared.track("button_tapped", properties: ["screen": "home"])

// ✅ Thread-safe Singleton
final class DatabaseManager {
    static let shared = DatabaseManager()
    
    private let queue = DispatchQueue(label: "com.app.database", attributes: .concurrent)
    private var _records: [String: Any] = [:]
    
    private init() {}
    
    // Thread-safe read
    func get(_ key: String) -> Any? {
        queue.sync { _records[key] }
    }
    
    // Thread-safe write (barrier)
    func set(_ value: Any, for key: String) {
        queue.async(flags: .barrier) { [weak self] in
            self?._records[key] = value
        }
    }
}

// ❌ ปัญหาของ Singleton - ทดสอบยาก
// เพราะทุก test share state เดียวกัน
class PaymentService {
    func processPayment(amount: Double) {
        // ปัญหา: ใช้ AnalyticsManager.shared โดยตรง
        AnalyticsManager.shared.track("payment_started", properties: ["amount": amount])
    }
}

// ✅ แก้ไขด้วย Protocol + DI
protocol Analytics {
    func track(_ event: String, properties: [String: Any])
}

extension AnalyticsManager: Analytics {}

class BetterPaymentService {
    private let analytics: Analytics
    
    init(analytics: Analytics = AnalyticsManager.shared) {
        self.analytics = analytics
    }
    
    func processPayment(amount: Double) {
        analytics.track("payment_started", properties: ["amount": amount])
    }
}
```

**เมื่อไหร่ควรใช้ Singleton:**
- Configuration หรือ Settings ที่ต้องการ Global Access
- Logging และ Analytics
- Cache ที่ share ทั้ง App
- Hardware Resources (Camera, Location)

**ข้อควรระวัง:**
- Global State ทำให้ Testing ยาก
- Hidden Dependencies
- Thread Safety

---

### 2.2 Factory Method Pattern

Factory Method กำหนด Interface สำหรับสร้าง Object แต่ให้ Subclass เป็นผู้ตัดสินใจว่าจะสร้าง Class ไหน

```swift
// Product Protocol
protocol Button {
    func render()
    func handleTap()
}

// Concrete Products
class IOSButton: Button {
    private let title: String
    
    init(title: String) {
        self.title = title
    }
    
    func render() {
        print("Rendering iOS button: \(title)")
    }
    
    func handleTap() {
        print("iOS button '\(title)' tapped")
    }
}

class MacOSButton: Button {
    private let title: String
    
    init(title: String) {
        self.title = title
    }
    
    func render() {
        print("Rendering macOS button: \(title)")
    }
    
    func handleTap() {
        print("macOS button '\(title)' clicked")
    }
}

// Creator (Abstract)
protocol UIFactory {
    func createButton(title: String) -> Button
    func createAlert(message: String) -> Alert
}

// Concrete Creators
class IOSUIFactory: UIFactory {
    func createButton(title: String) -> Button {
        IOSButton(title: title)
    }
    
    func createAlert(message: String) -> Alert {
        IOSAlert(message: message)
    }
}

class MacOSUIFactory: UIFactory {
    func createButton(title: String) -> Button {
        MacOSButton(title: title)
    }
    
    func createAlert(message: String) -> Alert {
        MacOSAlert(message: message)
    }
}

// Swift Static Factory Method
class NetworkRequest {
    let url: URL
    let method: String
    let headers: [String: String]
    let body: Data?
    
    private init(url: URL, method: String, headers: [String: String], body: Data?) {
        self.url = url
        self.method = method
        self.headers = headers
        self.body = body
    }
    
    // Static factory methods
    static func get(url: URL, headers: [String: String] = [:]) -> NetworkRequest {
        NetworkRequest(url: url, method: "GET", headers: headers, body: nil)
    }
    
    static func post(url: URL, body: Data, headers: [String: String] = [:]) -> NetworkRequest {
        var defaultHeaders = ["Content-Type": "application/json"]
        headers.forEach { defaultHeaders[$0.key] = $0.value }
        return NetworkRequest(url: url, method: "POST", headers: defaultHeaders, body: body)
    }
    
    static func delete(url: URL, headers: [String: String] = [:]) -> NetworkRequest {
        NetworkRequest(url: url, method: "DELETE", headers: headers, body: nil)
    }
}

// การใช้งาน
let getRequest = NetworkRequest.get(url: URL(string: "https://api.example.com/users")!)
let postRequest = NetworkRequest.post(
    url: URL(string: "https://api.example.com/users")!,
    body: Data()
)

// Factory เพื่อสร้าง Parsers
protocol ResponseParser {
    func parse<T: Decodable>(_ data: Data, as type: T.Type) throws -> T
}

class JSONResponseParser: ResponseParser {
    private let decoder: JSONDecoder
    
    init(decoder: JSONDecoder = JSONDecoder()) {
        self.decoder = decoder
    }
    
    func parse<T: Decodable>(_ data: Data, as type: T.Type) throws -> T {
        try decoder.decode(type, from: data)
    }
}

class XMLResponseParser: ResponseParser {
    func parse<T: Decodable>(_ data: Data, as type: T.Type) throws -> T {
        // XML parsing implementation
        fatalError("Not implemented")
    }
}

class ParserFactory {
    enum ContentType: String {
        case json = "application/json"
        case xml = "application/xml"
    }
    
    static func make(for contentType: ContentType) -> ResponseParser {
        switch contentType {
        case .json: return JSONResponseParser()
        case .xml: return XMLResponseParser()
        }
    }
}
```

---

### 2.3 Abstract Factory Pattern

Abstract Factory สร้าง Families ของ Related Objects โดยไม่ระบุ Concrete Classes

```swift
// Abstract Products
protocol TextInput {
    var value: String { get set }
    func validate() -> Bool
}

protocol PasswordInput {
    var value: String { get set }
    func validate() -> Bool
    var isVisible: Bool { get set }
}

protocol LoginButton {
    var isEnabled: Bool { get }
    func tap()
}

// Concrete Products - Light Theme
class LightTextInput: TextInput {
    var value: String = ""
    func validate() -> Bool { !value.isEmpty }
}

class LightPasswordInput: PasswordInput {
    var value: String = ""
    var isVisible: Bool = false
    func validate() -> Bool { value.count >= 8 }
}

class LightLoginButton: LoginButton {
    var isEnabled: Bool = false
    func tap() { print("Light button tapped") }
}

// Concrete Products - Dark Theme
class DarkTextInput: TextInput {
    var value: String = ""
    func validate() -> Bool { !value.isEmpty }
}

class DarkPasswordInput: PasswordInput {
    var value: String = ""
    var isVisible: Bool = false
    func validate() -> Bool { value.count >= 8 }
}

class DarkLoginButton: LoginButton {
    var isEnabled: Bool = false
    func tap() { print("Dark button tapped") }
}

// Abstract Factory
protocol LoginFormFactory {
    func createEmailInput() -> TextInput
    func createPasswordInput() -> PasswordInput
    func createLoginButton() -> LoginButton
}

// Concrete Factories
class LightThemeLoginFactory: LoginFormFactory {
    func createEmailInput() -> TextInput { LightTextInput() }
    func createPasswordInput() -> PasswordInput { LightPasswordInput() }
    func createLoginButton() -> LoginButton { LightLoginButton() }
}

class DarkThemeLoginFactory: LoginFormFactory {
    func createEmailInput() -> TextInput { DarkTextInput() }
    func createPasswordInput() -> PasswordInput { DarkPasswordInput() }
    func createLoginButton() -> LoginButton { DarkLoginButton() }
}

// Client
class LoginForm {
    private let emailInput: TextInput
    private let passwordInput: PasswordInput
    private let loginButton: LoginButton
    
    init(factory: LoginFormFactory) {
        emailInput = factory.createEmailInput()
        passwordInput = factory.createPasswordInput()
        loginButton = factory.createLoginButton()
    }
    
    func render() {
        print("Rendering login form")
    }
}

// การใช้งาน
let isDarkMode = true
let factory: LoginFormFactory = isDarkMode ? DarkThemeLoginFactory() : LightThemeLoginFactory()
let form = LoginForm(factory: factory)
form.render()
```

---

### 2.4 Builder Pattern

Builder แยกการสร้าง Complex Object ออกจาก Representation

```swift
// Builder สำหรับ API Request
struct APIRequest {
    let url: URL
    let method: HTTPMethod
    let headers: [String: String]
    let queryParameters: [String: String]
    let body: Data?
    let timeout: TimeInterval
    let cachePolicy: URLRequest.CachePolicy
    
    enum HTTPMethod: String {
        case get = "GET"
        case post = "POST"
        case put = "PUT"
        case patch = "PATCH"
        case delete = "DELETE"
    }
}

// Builder
class APIRequestBuilder {
    private var urlString: String
    private var method: APIRequest.HTTPMethod = .get
    private var headers: [String: String] = [:]
    private var queryParameters: [String: String] = [:]
    private var body: Data?
    private var timeout: TimeInterval = 30
    private var cachePolicy: URLRequest.CachePolicy = .useProtocolCachePolicy
    
    init(urlString: String) {
        self.urlString = urlString
    }
    
    @discardableResult
    func method(_ method: APIRequest.HTTPMethod) -> Self {
        self.method = method
        return self
    }
    
    @discardableResult
    func header(_ key: String, _ value: String) -> Self {
        headers[key] = value
        return self
    }
    
    @discardableResult
    func headers(_ headers: [String: String]) -> Self {
        headers.forEach { self.headers[$0.key] = $0.value }
        return self
    }
    
    @discardableResult
    func queryParameter(_ key: String, _ value: String) -> Self {
        queryParameters[key] = value
        return self
    }
    
    @discardableResult
    func body<T: Encodable>(_ object: T) -> Self {
        body = try? JSONEncoder().encode(object)
        return self
    }
    
    @discardableResult
    func timeout(_ timeout: TimeInterval) -> Self {
        self.timeout = timeout
        return self
    }
    
    @discardableResult
    func cachePolicy(_ policy: URLRequest.CachePolicy) -> Self {
        self.cachePolicy = policy
        return self
    }
    
    @discardableResult
    func bearerToken(_ token: String) -> Self {
        headers["Authorization"] = "Bearer \(token)"
        return self
    }
    
    @discardableResult
    func contentType(_ type: String) -> Self {
        headers["Content-Type"] = type
        return self
    }
    
    @discardableResult
    func accept(_ type: String) -> Self {
        headers["Accept"] = type
        return self
    }
    
    func build() throws -> APIRequest {
        var components = URLComponents(string: urlString)
        if !queryParameters.isEmpty {
            components?.queryItems = queryParameters.map {
                URLQueryItem(name: $0.key, value: $0.value)
            }
        }
        
        guard let url = components?.url else {
            throw BuilderError.invalidURL
        }
        
        return APIRequest(
            url: url,
            method: method,
            headers: headers,
            queryParameters: queryParameters,
            body: body,
            timeout: timeout,
            cachePolicy: cachePolicy
        )
    }
    
    enum BuilderError: Error {
        case invalidURL
    }
}

// การใช้งาน - Method Chaining
let request = try? APIRequestBuilder("https://api.github.com/search/repositories")
    .method(.get)
    .header("User-Agent", "MyApp/1.0")
    .bearerToken("ghp_xxxx")
    .accept("application/vnd.github.v3+json")
    .queryParameter("q", "swift language:swift")
    .queryParameter("sort", "stars")
    .queryParameter("per_page", "20")
    .timeout(60)
    .build()

// SwiftUI @resultBuilder
@resultBuilder
struct ViewBuilder2 {
    static func buildBlock(_ components: String...) -> String {
        components.joined(separator: "\n")
    }
}

// Builder สำหรับ Alert
class AlertBuilder {
    private var title: String
    private var message: String?
    private var actions: [(title: String, style: String, handler: (() -> Void)?)] = []
    
    init(title: String) {
        self.title = title
    }
    
    func message(_ message: String) -> Self {
        self.message = message
        return self
    }
    
    func addDefaultAction(title: String, handler: (() -> Void)? = nil) -> Self {
        actions.append((title: title, style: "default", handler: handler))
        return self
    }
    
    func addDestructiveAction(title: String, handler: (() -> Void)? = nil) -> Self {
        actions.append((title: title, style: "destructive", handler: handler))
        return self
    }
    
    func addCancelAction(title: String = "Cancel", handler: (() -> Void)? = nil) -> Self {
        actions.append((title: title, style: "cancel", handler: handler))
        return self
    }
    
    func build() -> String {
        // Build alert description for demonstration
        var desc = "Alert: \(title)"
        if let message = message { desc += "\nMessage: \(message)" }
        desc += "\nActions: \(actions.map { $0.title }.joined(separator: ", "))"
        return desc
    }
}

let deleteAlert = AlertBuilder(title: "Delete Item")
    .message("Are you sure you want to delete this item?")
    .addDestructiveAction(title: "Delete") { print("Deleted!") }
    .addCancelAction()
    .build()
```

---

### 2.5 Prototype Pattern

Prototype สร้าง Object ใหม่โดยการ Clone Object ที่มีอยู่

```swift
// Prototype Protocol
protocol Prototype: AnyObject {
    func clone() -> Self
}

// Concrete Prototype
class UserSettings: Prototype {
    var theme: Theme
    var language: String
    var notifications: NotificationSettings
    var privacy: PrivacySettings
    
    init(theme: Theme, language: String, notifications: NotificationSettings, privacy: PrivacySettings) {
        self.theme = theme
        self.language = language
        self.notifications = notifications
        self.privacy = privacy
    }
    
    // Deep copy
    func clone() -> UserSettings {
        UserSettings(
            theme: theme,
            language: language,
            notifications: notifications.clone(),
            privacy: privacy.clone()
        )
    }
}

class NotificationSettings: Prototype {
    var pushEnabled: Bool
    var emailEnabled: Bool
    var frequency: String
    
    init(pushEnabled: Bool, emailEnabled: Bool, frequency: String) {
        self.pushEnabled = pushEnabled
        self.emailEnabled = emailEnabled
        self.frequency = frequency
    }
    
    func clone() -> NotificationSettings {
        NotificationSettings(
            pushEnabled: pushEnabled,
            emailEnabled: emailEnabled,
            frequency: frequency
        )
    }
}

class PrivacySettings: Prototype {
    var profileVisible: Bool
    var shareData: Bool
    
    init(profileVisible: Bool, shareData: Bool) {
        self.profileVisible = profileVisible
        self.shareData = shareData
    }
    
    func clone() -> PrivacySettings {
        PrivacySettings(profileVisible: profileVisible, shareData: shareData)
    }
}

enum Theme: String {
    case light
    case dark
    case system
}

// Swift Struct - Automatic Value Semantics (Built-in Prototype)
struct Configuration {
    var serverURL: String
    var timeout: TimeInterval
    var retryCount: Int
    var features: Set<String>
    
    // Struct is already a prototype via copy-on-write
    func withTimeout(_ timeout: TimeInterval) -> Configuration {
        var copy = self
        copy.timeout = timeout
        return copy
    }
    
    func withFeature(_ feature: String) -> Configuration {
        var copy = self
        copy.features.insert(feature)
        return copy
    }
}

// การใช้งาน
let baseConfig = Configuration(
    serverURL: "https://api.example.com",
    timeout: 30,
    retryCount: 3,
    features: ["basic", "premium"]
)

// Struct copy ง่ายมาก!
let testConfig = baseConfig.withTimeout(5).withFeature("test_mode")
```

---

## 3. Structural Patterns

Structural Patterns เกี่ยวข้องกับการจัดโครงสร้างของ Class และ Object

### 3.1 Adapter Pattern

Adapter แปลง Interface ของ Class หนึ่งให้เข้ากันกับอีก Interface

```swift
// ปัญหา: Library เก่าที่ใช้ API ต่างกัน
// Legacy Payment System
class LegacyPaymentSystem {
    func makePayment(cardNumber: String, amount: Int, currencyCode: String) -> String {
        // Old implementation
        return "LEGACY_TXN_\(Int.random(in: 10000...99999))"
    }
    
    func checkBalance(cardNumber: String) -> Int {
        return 50000
    }
}

// New Payment Protocol ที่ App ใช้
protocol ModernPaymentProtocol {
    func pay(amount: Double, currency: Currency) async throws -> Transaction
    func getBalance() async throws -> Money
}

struct Transaction {
    let id: String
    let amount: Double
    let currency: Currency
    let timestamp: Date
}

struct Money {
    let amount: Double
    let currency: Currency
}

enum Currency: String {
    case thb = "THB"
    case usd = "USD"
    case eur = "EUR"
}

enum PaymentError: Error {
    case invalidCard
    case insufficientFunds
    case transactionFailed
}

// Adapter
class LegacyPaymentAdapter: ModernPaymentProtocol {
    private let legacy: LegacyPaymentSystem
    private let cardNumber: String
    
    init(legacy: LegacyPaymentSystem, cardNumber: String) {
        self.legacy = legacy
        self.cardNumber = cardNumber
    }
    
    func pay(amount: Double, currency: Currency) async throws -> Transaction {
        // แปลง Double เป็น Int (สตางค์)
        let amountInCents = Int(amount * 100)
        let txnId = legacy.makePayment(
            cardNumber: cardNumber,
            amount: amountInCents,
            currencyCode: currency.rawValue
        )
        
        return Transaction(
            id: txnId,
            amount: amount,
            currency: currency,
            timestamp: Date()
        )
    }
    
    func getBalance() async throws -> Money {
        let balanceInCents = legacy.checkBalance(cardNumber: cardNumber)
        return Money(
            amount: Double(balanceInCents) / 100.0,
            currency: .thb
        )
    }
}

// การใช้งาน
let legacySystem = LegacyPaymentSystem()
let payment: ModernPaymentProtocol = LegacyPaymentAdapter(
    legacy: legacySystem,
    cardNumber: "4111111111111111"
)

Task {
    if let transaction = try? await payment.pay(amount: 99.99, currency: .thb) {
        print("Transaction: \(transaction.id)")
    }
}

// Protocol Extension เป็น Adapter
// ทำให้ Third-party Type conform กับ Protocol ของเรา
import Foundation

// สมมุติว่า URLSessionDataTask ไม่มี method cancel()
// เราต้องการ CancellableTask protocol
protocol CancellableTask {
    func cancel()
    var isCancelled: Bool { get }
}

// URLSessionDataTask มี cancel() อยู่แล้ว แต่ไม่มี isCancelled
extension URLSessionDataTask: CancellableTask {
    var isCancelled: Bool {
        return state == .canceling
    }
}
```

---

### 3.2 Bridge Pattern

Bridge แยก Abstraction ออกจาก Implementation

```swift
// Implementation Protocol
protocol MessageSender {
    func send(_ content: String, to recipient: String) async throws
}

// Concrete Implementations
class EmailSender: MessageSender {
    private let smtpServer: String
    
    init(smtpServer: String = "smtp.example.com") {
        self.smtpServer = smtpServer
    }
    
    func send(_ content: String, to recipient: String) async throws {
        print("Sending email to \(recipient): \(content)")
    }
}

class SMSSender: MessageSender {
    private let provider: String
    
    init(provider: String = "twilio") {
        self.provider = provider
    }
    
    func send(_ content: String, to recipient: String) async throws {
        print("Sending SMS to \(recipient) via \(provider): \(content)")
    }
}

class PushNotificationSender: MessageSender {
    func send(_ content: String, to recipient: String) async throws {
        print("Sending push notification to \(recipient): \(content)")
    }
}

// Abstraction
class Message {
    protected let sender: MessageSender
    
    init(sender: MessageSender) {
        self.sender = sender
    }
    
    func send(to recipient: String) async throws {
        fatalError("Subclass must implement")
    }
}

// Refined Abstractions
class UrgentMessage: Message {
    private let content: String
    
    init(content: String, sender: MessageSender) {
        self.content = content
        super.init(sender: sender)
    }
    
    override func send(to recipient: String) async throws {
        let urgentContent = "🚨 URGENT: \(content)"
        try await sender.send(urgentContent, to: recipient)
    }
}

class ScheduledMessage: Message {
    private let content: String
    private let scheduledDate: Date
    
    init(content: String, sender: MessageSender, scheduledDate: Date) {
        self.content = content
        self.scheduledDate = scheduledDate
        super.init(sender: sender)
    }
    
    override func send(to recipient: String) async throws {
        // ตรวจสอบเวลาก่อน
        guard Date() >= scheduledDate else {
            print("Not yet time to send - scheduled for \(scheduledDate)")
            return
        }
        try await sender.send(content, to: recipient)
    }
}

// การใช้งาน - เปลี่ยน Sender ได้โดยไม่ต้องแก้ Message
Task {
    let urgent = UrgentMessage(content: "Server is down!", sender: SMSSender())
    try? await urgent.send(to: "+66812345678")
    
    // เปลี่ยนเป็น Email โดยไม่แก้ UrgentMessage
    let urgentEmail = UrgentMessage(content: "Server is down!", sender: EmailSender())
    try? await urgentEmail.send(to: "admin@example.com")
}
```

---

### 3.3 Composite Pattern

Composite ให้ Client จัดการ Individual Objects และ Compositions ในรูปแบบเดียวกัน

```swift
// Component Protocol
protocol FileSystemItem {
    var name: String { get }
    var size: Int { get }
    func display(indent: String)
}

// Leaf
class File: FileSystemItem {
    let name: String
    let size: Int
    let type: String
    
    init(name: String, size: Int, type: String = "file") {
        self.name = name
        self.size = size
        self.type = type
    }
    
    func display(indent: String = "") {
        print("\(indent)📄 \(name) (\(size) bytes)")
    }
}

// Composite
class Folder: FileSystemItem {
    let name: String
    private var children: [FileSystemItem] = []
    
    var size: Int { children.reduce(0) { $0 + $1.size } }
    
    init(name: String) {
        self.name = name
    }
    
    func add(_ item: FileSystemItem) {
        children.append(item)
    }
    
    func remove(_ name: String) {
        children.removeAll { $0.name == name }
    }
    
    func find(_ name: String) -> FileSystemItem? {
        if self.name == name { return self }
        for child in children {
            if let found = (child as? Folder)?.find(name) {
                return found
            } else if child.name == name {
                return child
            }
        }
        return nil
    }
    
    func display(indent: String = "") {
        print("\(indent)📁 \(name) (\(size) bytes)")
        children.forEach { $0.display(indent: indent + "  ") }
    }
}

// การใช้งาน
let root = Folder(name: "Project")

let src = Folder(name: "src")
src.add(File(name: "main.swift", size: 1024))
src.add(File(name: "AppDelegate.swift", size: 512))

let views = Folder(name: "Views")
views.add(File(name: "HomeView.swift", size: 2048))
views.add(File(name: "ProfileView.swift", size: 1536))
src.add(views)

let resources = Folder(name: "Resources")
resources.add(File(name: "Assets.xcassets", size: 10240))
resources.add(File(name: "Info.plist", size: 256))

root.add(src)
root.add(resources)
root.add(File(name: "Package.swift", size: 512))

root.display()
// 📁 Project (16128 bytes)
//   📁 src (5120 bytes)
//     📄 main.swift (1024 bytes)
//     📄 AppDelegate.swift (512 bytes)
//     📁 Views (3584 bytes)
//       📄 HomeView.swift (2048 bytes)
//       📄 ProfileView.swift (1536 bytes)
//   📁 Resources (10496 bytes)
//     📄 Assets.xcassets (10240 bytes)
//     📄 Info.plist (256 bytes)
//   📄 Package.swift (512 bytes)
```

---

### 3.4 Decorator Pattern

Decorator เพิ่ม Behavior ให้ Object โดยไม่แก้ Original Class

```swift
// Component Protocol
protocol TextTransformer {
    func transform(_ text: String) -> String
}

// Concrete Component
class PlainTextTransformer: TextTransformer {
    func transform(_ text: String) -> String {
        return text
    }
}

// Base Decorator
class TextDecorator: TextTransformer {
    private let transformer: TextTransformer
    
    init(_ transformer: TextTransformer) {
        self.transformer = transformer
    }
    
    func transform(_ text: String) -> String {
        return transformer.transform(text)
    }
}

// Concrete Decorators
class UppercaseDecorator: TextDecorator {
    override func transform(_ text: String) -> String {
        return super.transform(text).uppercased()
    }
}

class TrimDecorator: TextDecorator {
    override func transform(_ text: String) -> String {
        return super.transform(text).trimmingCharacters(in: .whitespaces)
    }
}

class PrefixDecorator: TextDecorator {
    private let prefix: String
    
    init(_ transformer: TextTransformer, prefix: String) {
        self.prefix = prefix
        super.init(transformer)
    }
    
    override func transform(_ text: String) -> String {
        return "\(prefix)\(super.transform(text))"
    }
}

class SuffixDecorator: TextDecorator {
    private let suffix: String
    
    init(_ transformer: TextTransformer, suffix: String) {
        self.suffix = suffix
        super.init(transformer)
    }
    
    override func transform(_ text: String) -> String {
        return "\(super.transform(text))\(suffix)"
    }
}

class EmojiDecorator: TextDecorator {
    private let emoji: String
    
    init(_ transformer: TextTransformer, emoji: String) {
        self.emoji = emoji
        super.init(transformer)
    }
    
    override func transform(_ text: String) -> String {
        return "\(emoji) \(super.transform(text)) \(emoji)"
    }
}

// การใช้งาน
let base = PlainTextTransformer()

// เพิ่ม decorators ตามต้องการ
let fancy = EmojiDecorator(
    UppercaseDecorator(
        TrimDecorator(base)
    ),
    emoji: "🎉"
)

print(fancy.transform("  hello world  "))
// Output: 🎉 HELLO WORLD 🎉

// Swift Extension Decorator (Functional Style)
extension String {
    func trimmed() -> String {
        trimmingCharacters(in: .whitespaces)
    }
    
    func capitalizingFirstLetter() -> String {
        guard !isEmpty else { return self }
        return prefix(1).uppercased() + dropFirst()
    }
    
    func truncated(to maxLength: Int, trailing: String = "...") -> String {
        guard count > maxLength else { return self }
        return String(prefix(maxLength)) + trailing
    }
}

// การใช้งาน
let text = "  hello world this is a long text  "
let result = text.trimmed().capitalizingFirstLetter().truncated(to: 15)
print(result) // "Hello world thi..."

// Decorator สำหรับ Logging Network Requests
class LoggingAPIClient: APIClientProtocol {
    private let wrapped: APIClientProtocol
    private let logger: LoggerProtocol
    
    init(wrapped: APIClientProtocol, logger: LoggerProtocol) {
        self.wrapped = wrapped
        self.logger = logger
    }
    
    func request<T: Decodable>(_ endpoint: Endpoint) async throws -> T {
        let start = Date()
        logger.log("→ \(endpoint.method) \(endpoint.path)")
        
        do {
            let result: T = try await wrapped.request(endpoint)
            let elapsed = Date().timeIntervalSince(start)
            logger.log("← \(endpoint.path) [\(String(format: "%.3f", elapsed))s]")
            return result
        } catch {
            logger.log("✗ \(endpoint.path): \(error)")
            throw error
        }
    }
}
```

---

### 3.5 Facade Pattern

Facade ให้ Simplified Interface สำหรับ Complex Subsystem

```swift
// Complex Subsystems
class VideoEncoder {
    func encode(file: String, format: String) -> String {
        print("Encoding video \(file) to \(format)")
        return "encoded_\(file)"
    }
}

class AudioProcessor {
    func extractAudio(from video: String) -> String {
        print("Extracting audio from \(video)")
        return "audio_\(video)"
    }
    
    func normalizeAudio(_ audio: String) -> String {
        print("Normalizing audio \(audio)")
        return "normalized_\(audio)"
    }
}

class ThumbnailGenerator {
    func generate(from video: String, at time: TimeInterval) -> String {
        print("Generating thumbnail at \(time)s from \(video)")
        return "thumb_\(video).jpg"
    }
}

class CDNUploader {
    func upload(_ file: String, to bucket: String) async throws -> String {
        print("Uploading \(file) to \(bucket)")
        return "https://cdn.example.com/\(bucket)/\(file)"
    }
}

class MetadataStorage {
    func save(videoId: String, metadata: [String: Any]) {
        print("Saving metadata for \(videoId)")
    }
}

// Facade
class VideoProcessingFacade {
    private let encoder = VideoEncoder()
    private let audioProcessor = AudioProcessor()
    private let thumbnailGenerator = ThumbnailGenerator()
    private let cdnUploader = CDNUploader()
    private let metadataStorage = MetadataStorage()
    
    struct ProcessingResult {
        let videoURL: String
        let thumbnailURL: String
        let duration: TimeInterval
    }
    
    func processVideo(
        file: String,
        title: String,
        format: String = "mp4"
    ) async throws -> ProcessingResult {
        // 1. Encode
        let encoded = encoder.encode(file: file, format: format)
        
        // 2. Process audio
        let audio = audioProcessor.extractAudio(from: encoded)
        _ = audioProcessor.normalizeAudio(audio)
        
        // 3. Generate thumbnail
        let thumbnail = thumbnailGenerator.generate(from: encoded, at: 5.0)
        
        // 4. Upload to CDN
        let videoURL = try await cdnUploader.upload(encoded, to: "videos")
        let thumbURL = try await cdnUploader.upload(thumbnail, to: "thumbnails")
        
        // 5. Save metadata
        let videoId = UUID().uuidString
        metadataStorage.save(videoId: videoId, metadata: [
            "title": title,
            "url": videoURL,
            "thumbnail": thumbURL
        ])
        
        return ProcessingResult(
            videoURL: videoURL,
            thumbnailURL: thumbURL,
            duration: 120.0
        )
    }
}

// Client ใช้แค่ Facade - ไม่ต้องรู้ Subsystem
Task {
    let processor = VideoProcessingFacade()
    if let result = try? await processor.processVideo(file: "my_video.mov", title: "My Video") {
        print("Video URL: \(result.videoURL)")
        print("Thumbnail: \(result.thumbnailURL)")
    }
}
```

---

### 3.6 Flyweight Pattern

Flyweight ลด Memory Usage โดย Share State ระหว่าง Objects

```swift
// Flyweight สำหรับ Icons
class Icon {
    let name: String
    let data: Data // Intrinsic state - shared
    
    init(name: String, data: Data) {
        self.name = name
        self.data = data
        print("Loading icon '\(name)' (\(data.count) bytes)")
    }
}

// Flyweight Factory
class IconFactory {
    private var icons: [String: Icon] = [:]
    
    func getIcon(named name: String) -> Icon {
        if let cached = icons[name] {
            return cached  // Reuse existing
        }
        
        // Load icon data (expensive operation)
        let data = loadIconData(name: name)
        let icon = Icon(name: name, data: data)
        icons[name] = icon
        return icon
    }
    
    private func loadIconData(name: String) -> Data {
        // Simulate loading from disk
        return Data(name.utf8)
    }
    
    var loadedCount: Int { icons.count }
}

// Client ที่ใช้ Flyweight
struct TableCell {
    let iconFactory: IconFactory
    let iconName: String  // Extrinsic state
    let position: Int     // Extrinsic state
    
    var icon: Icon {
        iconFactory.getIcon(named: iconName)
    }
    
    func render() {
        print("Cell at \(position): icon '\(icon.name)'")
    }
}

// การใช้งาน
let factory = IconFactory()

// สร้าง 1000 cells แต่ icon ถูก load เพียงครั้งเดียว
let cells = (0..<1000).map { i in
    TableCell(
        iconFactory: factory,
        iconName: ["home", "search", "profile", "settings"][i % 4],
        position: i
    )
}

cells[0].render()
cells[1].render()
print("Icons loaded: \(factory.loadedCount)") // Only 4!
```

---

### 3.7 Proxy Pattern

Proxy เป็น Surrogate หรือ Placeholder สำหรับ Object อื่น

```swift
// Real Subject
protocol ImageLoader {
    func loadImage(url: URL) async throws -> UIImage
}

// Real Implementation
class NetworkImageLoader: ImageLoader {
    func loadImage(url: URL) async throws -> UIImage {
        let (data, _) = try await URLSession.shared.data(from: url)
        guard let image = UIImage(data: data) else {
            throw ImageError.invalidData
        }
        return image
    }
}

// Cache Proxy
class CachedImageLoader: ImageLoader {
    private let loader: ImageLoader
    private var cache: [URL: UIImage] = [:]
    private let queue = DispatchQueue(label: "imageCache", attributes: .concurrent)
    
    init(loader: ImageLoader) {
        self.loader = loader
    }
    
    func loadImage(url: URL) async throws -> UIImage {
        // Check cache (thread-safe read)
        if let cached = queue.sync(execute: { cache[url] }) {
            print("Cache hit for \(url.lastPathComponent)")
            return cached
        }
        
        print("Cache miss - loading \(url.lastPathComponent)")
        let image = try await loader.loadImage(url: url)
        
        // Store in cache (thread-safe write)
        queue.async(flags: .barrier) { [weak self] in
            self?.cache[url] = image
        }
        
        return image
    }
    
    func clearCache() {
        queue.async(flags: .barrier) { [weak self] in
            self?.cache = [:]
        }
    }
}

// Logging Proxy
class LoggingImageLoader: ImageLoader {
    private let loader: ImageLoader
    
    init(loader: ImageLoader) {
        self.loader = loader
    }
    
    func loadImage(url: URL) async throws -> UIImage {
        print("⬇️ Loading: \(url)")
        let start = Date()
        do {
            let image = try await loader.loadImage(url: url)
            let elapsed = Date().timeIntervalSince(start)
            print("✅ Loaded in \(String(format: "%.2f", elapsed))s")
            return image
        } catch {
            print("❌ Failed: \(error)")
            throw error
        }
    }
}

enum ImageError: Error {
    case invalidData
}

// การใช้งาน - Stack proxies
let imageLoader: ImageLoader = LoggingImageLoader(
    loader: CachedImageLoader(
        loader: NetworkImageLoader()
    )
)
```

---

## 4. Behavioral Patterns

### 4.1 Chain of Responsibility

```swift
// Handler Protocol
protocol RequestHandler: AnyObject {
    var next: RequestHandler? { get set }
    func handle(_ request: AuthRequest) -> AuthResponse?
}

struct AuthRequest {
    let token: String
    let userId: String
    let resource: String
    let action: String
}

struct AuthResponse {
    let isAuthorized: Bool
    let reason: String
}

// Base Handler
class BaseAuthHandler: RequestHandler {
    var next: RequestHandler?
    
    func handle(_ request: AuthRequest) -> AuthResponse? {
        return next?.handle(request)
    }
    
    func setNext(_ handler: RequestHandler) -> RequestHandler {
        self.next = handler
        return handler
    }
}

// Concrete Handlers
class TokenValidationHandler: BaseAuthHandler {
    override func handle(_ request: AuthRequest) -> AuthResponse? {
        guard !request.token.isEmpty else {
            return AuthResponse(isAuthorized: false, reason: "Missing token")
        }
        guard request.token.hasPrefix("Bearer ") else {
            return AuthResponse(isAuthorized: false, reason: "Invalid token format")
        }
        print("✅ Token validated")
        return next?.handle(request)
    }
}

class RateLimitHandler: BaseAuthHandler {
    private var requestCounts: [String: Int] = [:]
    private let limit = 100
    
    override func handle(_ request: AuthRequest) -> AuthResponse? {
        let count = requestCounts[request.userId, default: 0]
        guard count < limit else {
            return AuthResponse(isAuthorized: false, reason: "Rate limit exceeded")
        }
        requestCounts[request.userId] = count + 1
        print("✅ Rate limit OK (\(count + 1)/\(limit))")
        return next?.handle(request)
    }
}

class PermissionHandler: BaseAuthHandler {
    private let permissions: [String: Set<String>] = [
        "admin": ["read", "write", "delete"],
        "user": ["read", "write"],
        "guest": ["read"]
    ]
    
    override func handle(_ request: AuthRequest) -> AuthResponse? {
        let userRole = getUserRole(userId: request.userId)
        let allowed = permissions[userRole]?.contains(request.action) ?? false
        
        guard allowed else {
            return AuthResponse(isAuthorized: false, reason: "Insufficient permissions")
        }
        print("✅ Permission granted")
        return AuthResponse(isAuthorized: true, reason: "All checks passed")
    }
    
    private func getUserRole(userId: String) -> String {
        return userId.hasPrefix("admin") ? "admin" : "user"
    }
}

// การใช้งาน - สร้าง Chain
let tokenHandler = TokenValidationHandler()
let rateLimitHandler = RateLimitHandler()
let permissionHandler = PermissionHandler()

tokenHandler.setNext(rateLimitHandler).setNext(permissionHandler)
// ทำได้เพราะ setNext return next handler

let request = AuthRequest(
    token: "Bearer abc123",
    userId: "user_456",
    resource: "posts",
    action: "write"
)

let response = tokenHandler.handle(request)
print("Authorized: \(response?.isAuthorized ?? false)")
```

---

### 4.2 Command Pattern

```swift
// Command Protocol
protocol Command {
    func execute()
    func undo()
}

// Receiver
class TextEditor {
    private var text = ""
    
    var currentText: String { text }
    
    func insertText(_ newText: String, at position: Int) {
        let index = text.index(text.startIndex, offsetBy: min(position, text.count))
        text.insert(contentsOf: newText, at: index)
    }
    
    func deleteText(at position: Int, count: Int) {
        guard position < text.count else { return }
        let start = text.index(text.startIndex, offsetBy: position)
        let end = text.index(start, offsetBy: min(count, text.count - position))
        text.removeSubrange(start..<end)
    }
    
    func replaceText(in range: Range<Int>, with newText: String) {
        let start = text.index(text.startIndex, offsetBy: range.lowerBound)
        let end = text.index(text.startIndex, offsetBy: min(range.upperBound, text.count))
        text.replaceSubrange(start..<end, with: newText)
    }
}

// Concrete Commands
class InsertCommand: Command {
    private let editor: TextEditor
    private let text: String
    private let position: Int
    
    init(editor: TextEditor, text: String, at position: Int) {
        self.editor = editor
        self.text = text
        self.position = position
    }
    
    func execute() {
        editor.insertText(text, at: position)
    }
    
    func undo() {
        editor.deleteText(at: position, count: text.count)
    }
}

class DeleteCommand: Command {
    private let editor: TextEditor
    private let position: Int
    private let count: Int
    private var deletedText = ""
    
    init(editor: TextEditor, at position: Int, count: Int) {
        self.editor = editor
        self.position = position
        self.count = count
    }
    
    func execute() {
        let text = editor.currentText
        let start = text.index(text.startIndex, offsetBy: min(position, text.count))
        let end = text.index(start, offsetBy: min(count, text.count - position))
        deletedText = String(text[start..<end])
        editor.deleteText(at: position, count: count)
    }
    
    func undo() {
        editor.insertText(deletedText, at: position)
    }
}

// Invoker (History Manager)
class CommandHistory {
    private var undoStack: [Command] = []
    private var redoStack: [Command] = []
    
    func execute(_ command: Command) {
        command.execute()
        undoStack.append(command)
        redoStack.removeAll() // Clear redo when new command executed
    }
    
    func undo() {
        guard let command = undoStack.popLast() else {
            print("Nothing to undo")
            return
        }
        command.undo()
        redoStack.append(command)
    }
    
    func redo() {
        guard let command = redoStack.popLast() else {
            print("Nothing to redo")
            return
        }
        command.execute()
        undoStack.append(command)
    }
    
    var canUndo: Bool { !undoStack.isEmpty }
    var canRedo: Bool { !redoStack.isEmpty }
}

// การใช้งาน
let editor = TextEditor()
let history = CommandHistory()

history.execute(InsertCommand(editor: editor, text: "Hello", at: 0))
print(editor.currentText) // Hello

history.execute(InsertCommand(editor: editor, text: " World", at: 5))
print(editor.currentText) // Hello World

history.execute(DeleteCommand(editor: editor, at: 5, count: 6))
print(editor.currentText) // Hello

history.undo()
print(editor.currentText) // Hello World

history.undo()
print(editor.currentText) // Hello

history.redo()
print(editor.currentText) // Hello World
```

---

### 4.3 Observer Pattern

```swift
// Swift's built-in Observer mechanism

// 1. NotificationCenter
class UserManager {
    static let userDidLoginNotification = Notification.Name("userDidLogin")
    static let userDidLogoutNotification = Notification.Name("userDidLogout")
    
    func login(user: User) {
        // Login logic...
        NotificationCenter.default.post(
            name: UserManager.userDidLoginNotification,
            object: nil,
            userInfo: ["user": user]
        )
    }
    
    func logout() {
        NotificationCenter.default.post(name: UserManager.userDidLogoutNotification, object: nil)
    }
}

// Observer
class ProfileViewController {
    init() {
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(handleLogin(_:)),
            name: UserManager.userDidLoginNotification,
            object: nil
        )
    }
    
    @objc func handleLogin(_ notification: Notification) {
        if let user = notification.userInfo?["user"] as? User {
            print("User logged in: \(user.name)")
        }
    }
    
    deinit {
        NotificationCenter.default.removeObserver(self)
    }
}

// 2. Combine Publisher/Subscriber
import Combine

class StockPricePublisher {
    private let subject = PassthroughSubject<StockQuote, Never>()
    
    var publisher: AnyPublisher<StockQuote, Never> {
        subject.eraseToAnyPublisher()
    }
    
    func updatePrice(symbol: String, price: Double) {
        let quote = StockQuote(symbol: symbol, price: price, timestamp: Date())
        subject.send(quote)
    }
}

struct StockQuote {
    let symbol: String
    let price: Double
    let timestamp: Date
}

class StockTracker {
    private var cancellables = Set<AnyCancellable>()
    
    func watch(_ publisher: AnyPublisher<StockQuote, Never>, for symbol: String) {
        publisher
            .filter { $0.symbol == symbol }
            .sink { quote in
                print("\(quote.symbol): $\(quote.price) at \(quote.timestamp)")
            }
            .store(in: &cancellables)
    }
}

// 3. Custom Observable
protocol Observer: AnyObject {
    func update<T>(with value: T)
}

class Observable<T> {
    private var observers: [WeakObserver] = []
    
    var value: T {
        didSet { notifyObservers() }
    }
    
    init(_ value: T) {
        self.value = value
    }
    
    func addObserver(_ observer: Observer) {
        observers.append(WeakObserver(observer))
    }
    
    func removeObserver(_ observer: Observer) {
        observers.removeAll { $0.observer === observer }
    }
    
    private func notifyObservers() {
        observers.forEach { $0.observer?.update(with: value) }
        observers.removeAll { $0.observer == nil } // Cleanup deallocated
    }
    
    private struct WeakObserver {
        weak var observer: Observer?
        
        init(_ observer: Observer) {
            self.observer = observer
        }
    }
}
```

---

### 4.4 Strategy Pattern

```swift
// Strategy Pattern สำหรับ Sorting
protocol SortStrategy {
    func sort<T: Comparable>(_ array: [T]) -> [T]
}

class BubbleSortStrategy: SortStrategy {
    func sort<T: Comparable>(_ array: [T]) -> [T] {
        var arr = array
        let n = arr.count
        for i in 0..<n {
            for j in 0..<(n - i - 1) {
                if arr[j] > arr[j + 1] {
                    arr.swapAt(j, j + 1)
                }
            }
        }
        return arr
    }
}

class QuickSortStrategy: SortStrategy {
    func sort<T: Comparable>(_ array: [T]) -> [T] {
        guard array.count > 1 else { return array }
        let pivot = array[array.count / 2]
        let left = array.filter { $0 < pivot }
        let middle = array.filter { $0 == pivot }
        let right = array.filter { $0 > pivot }
        return sort(left) + middle + sort(right)
    }
}

class MergeSortStrategy: SortStrategy {
    func sort<T: Comparable>(_ array: [T]) -> [T] {
        guard array.count > 1 else { return array }
        let mid = array.count / 2
        let left = sort(Array(array[..<mid]))
        let right = sort(Array(array[mid...]))
        return merge(left, right)
    }
    
    private func merge<T: Comparable>(_ left: [T], _ right: [T]) -> [T] {
        var result: [T] = []
        var l = 0, r = 0
        while l < left.count && r < right.count {
            if left[l] <= right[r] {
                result.append(left[l]); l += 1
            } else {
                result.append(right[r]); r += 1
            }
        }
        return result + Array(left[l...]) + Array(right[r...])
    }
}

// Context
class Sorter<T: Comparable> {
    private var strategy: SortStrategy
    
    init(strategy: SortStrategy) {
        self.strategy = strategy
    }
    
    func changeStrategy(_ strategy: SortStrategy) {
        self.strategy = strategy
    }
    
    func sort(_ array: [T]) -> [T] {
        strategy.sort(array)
    }
}

// การใช้งาน
let sorter = Sorter<Int>(strategy: QuickSortStrategy())
let numbers = [64, 34, 25, 12, 22, 11, 90]
print(sorter.sort(numbers)) // [11, 12, 22, 25, 34, 64, 90]

// เปลี่ยน Strategy
sorter.changeStrategy(MergeSortStrategy())
print(sorter.sort(numbers)) // [11, 12, 22, 25, 34, 64, 90]

// Swift Functional Strategy ด้วย Closure
class DataExporter {
    typealias ExportStrategy = (Any) -> Data
    
    private var strategy: ExportStrategy
    
    init(strategy: @escaping ExportStrategy) {
        self.strategy = strategy
    }
    
    func setStrategy(_ strategy: @escaping ExportStrategy) {
        self.strategy = strategy
    }
    
    func export(_ data: Any) -> Data {
        strategy(data)
    }
}

// Strategies
let jsonExport: DataExporter.ExportStrategy = { data in
    (try? JSONSerialization.data(withJSONObject: data)) ?? Data()
}

let csvExport: DataExporter.ExportStrategy = { data in
    guard let dict = data as? [String: Any] else { return Data() }
    let csv = dict.map { "\($0.key),\($0.value)" }.joined(separator: "\n")
    return Data(csv.utf8)
}
```

---

### 4.5 State Pattern

```swift
// State Pattern สำหรับ Order Status
protocol OrderState {
    func confirm(order: Order) throws
    func ship(order: Order) throws
    func deliver(order: Order) throws
    func cancel(order: Order) throws
    var displayName: String { get }
}

class Order {
    var state: OrderState = PendingState()
    var id: String
    var items: [String]
    
    init(id: String, items: [String]) {
        self.id = id
        self.items = items
    }
    
    func confirm() throws { try state.confirm(order: self) }
    func ship() throws { try state.ship(order: self) }
    func deliver() throws { try state.deliver(order: self) }
    func cancel() throws { try state.cancel(order: self) }
    
    var statusText: String { state.displayName }
}

enum OrderError: Error {
    case invalidTransition(from: String, action: String)
}

// Concrete States
class PendingState: OrderState {
    var displayName = "Pending"
    
    func confirm(order: Order) throws {
        print("Order confirmed!")
        order.state = ConfirmedState()
    }
    
    func ship(order: Order) throws {
        throw OrderError.invalidTransition(from: displayName, action: "ship")
    }
    
    func deliver(order: Order) throws {
        throw OrderError.invalidTransition(from: displayName, action: "deliver")
    }
    
    func cancel(order: Order) throws {
        print("Order cancelled from pending")
        order.state = CancelledState()
    }
}

class ConfirmedState: OrderState {
    var displayName = "Confirmed"
    
    func confirm(order: Order) throws {
        throw OrderError.invalidTransition(from: displayName, action: "confirm")
    }
    
    func ship(order: Order) throws {
        print("Order shipped!")
        order.state = ShippedState()
    }
    
    func deliver(order: Order) throws {
        throw OrderError.invalidTransition(from: displayName, action: "deliver")
    }
    
    func cancel(order: Order) throws {
        print("Order cancelled from confirmed - processing refund")
        order.state = CancelledState()
    }
}

class ShippedState: OrderState {
    var displayName = "Shipped"
    
    func confirm(order: Order) throws {
        throw OrderError.invalidTransition(from: displayName, action: "confirm")
    }
    
    func ship(order: Order) throws {
        throw OrderError.invalidTransition(from: displayName, action: "ship")
    }
    
    func deliver(order: Order) throws {
        print("Order delivered!")
        order.state = DeliveredState()
    }
    
    func cancel(order: Order) throws {
        throw OrderError.invalidTransition(from: displayName, action: "cancel - already shipped")
    }
}

class DeliveredState: OrderState {
    var displayName = "Delivered"
    
    func confirm(order: Order) throws {
        throw OrderError.invalidTransition(from: displayName, action: "confirm")
    }
    func ship(order: Order) throws {
        throw OrderError.invalidTransition(from: displayName, action: "ship")
    }
    func deliver(order: Order) throws {
        throw OrderError.invalidTransition(from: displayName, action: "deliver")
    }
    func cancel(order: Order) throws {
        throw OrderError.invalidTransition(from: displayName, action: "cancel")
    }
}

class CancelledState: OrderState {
    var displayName = "Cancelled"
    
    func confirm(order: Order) throws {
        throw OrderError.invalidTransition(from: displayName, action: "confirm")
    }
    func ship(order: Order) throws {
        throw OrderError.invalidTransition(from: displayName, action: "ship")
    }
    func deliver(order: Order) throws {
        throw OrderError.invalidTransition(from: displayName, action: "deliver")
    }
    func cancel(order: Order) throws {
        throw OrderError.invalidTransition(from: displayName, action: "cancel")
    }
}

// การใช้งาน
let order = Order(id: "ORD-001", items: ["iPhone", "Case"])
print("Status: \(order.statusText)") // Pending

try? order.confirm()
print("Status: \(order.statusText)") // Confirmed

try? order.ship()
print("Status: \(order.statusText)") // Shipped

// ลองทำ action ที่ไม่ถูกต้อง
do {
    try order.cancel()
} catch OrderError.invalidTransition(let from, let action) {
    print("Cannot \(action) when order is \(from)")
}

try? order.deliver()
print("Status: \(order.statusText)") // Delivered
```

---

### 4.6 Template Method Pattern

```swift
// Template Method ใน Swift
class DataProcessor {
    // Template Method - defines the skeleton
    final func process(data: Data) throws -> ProcessedResult {
        let validated = try validate(data)
        let parsed = try parse(validated)
        let transformed = transform(parsed)
        let result = compile(transformed)
        log(result)
        return result
    }
    
    // Abstract Steps (to be overridden)
    func validate(_ data: Data) throws -> Data {
        guard !data.isEmpty else { throw ProcessingError.emptyData }
        return data
    }
    
    func parse(_ data: Data) throws -> [String: Any] {
        fatalError("Subclass must implement parse")
    }
    
    func transform(_ parsed: [String: Any]) -> [String: Any] {
        return parsed // Default: no transformation
    }
    
    func compile(_ data: [String: Any]) -> ProcessedResult {
        return ProcessedResult(data: data)
    }
    
    func log(_ result: ProcessedResult) {
        print("Processing complete: \(result.data.count) fields")
    }
}

struct ProcessedResult {
    let data: [String: Any]
}

enum ProcessingError: Error {
    case emptyData
    case invalidFormat
}

// Concrete Subclasses
class JSONProcessor: DataProcessor {
    override func parse(_ data: Data) throws -> [String: Any] {
        guard let json = try JSONSerialization.jsonObject(with: data) as? [String: Any] else {
            throw ProcessingError.invalidFormat
        }
        return json
    }
    
    override func transform(_ parsed: [String: Any]) -> [String: Any] {
        // Convert all string values to uppercase
        return parsed.mapValues { value in
            (value as? String)?.uppercased() ?? value
        }
    }
}

class CSVProcessor: DataProcessor {
    override func parse(_ data: Data) throws -> [String: Any] {
        guard let csv = String(data: data, encoding: .utf8) else {
            throw ProcessingError.invalidFormat
        }
        let lines = csv.components(separatedBy: "\n")
        guard let header = lines.first?.components(separatedBy: ",") else {
            throw ProcessingError.invalidFormat
        }
        
        var result: [String: Any] = [:]
        if lines.count > 1 {
            let values = lines[1].components(separatedBy: ",")
            for (key, value) in zip(header, values) {
                result[key] = value
            }
        }
        return result
    }
}
```

---

### 4.7 Memento Pattern

```swift
// Memento Pattern สำหรับ Undo/Redo
struct DrawingState {
    var paths: [[CGPoint]]
    var currentColor: String
    var brushSize: Float
    var backgroundColor: String
}

// Memento
struct DrawingMemento {
    private let state: DrawingState
    private let timestamp: Date
    
    fileprivate init(state: DrawingState) {
        self.state = state
        self.timestamp = Date()
    }
    
    fileprivate var savedState: DrawingState { state }
    var savedAt: Date { timestamp }
}

// Originator
class DrawingCanvas {
    private(set) var state: DrawingState
    
    init() {
        state = DrawingState(
            paths: [],
            currentColor: "#000000",
            brushSize: 2.0,
            backgroundColor: "#FFFFFF"
        )
    }
    
    func addPath(_ path: [CGPoint]) {
        state.paths.append(path)
    }
    
    func setColor(_ color: String) {
        state.currentColor = color
    }
    
    func setBrushSize(_ size: Float) {
        state.brushSize = size
    }
    
    func save() -> DrawingMemento {
        return DrawingMemento(state: state)
    }
    
    func restore(from memento: DrawingMemento) {
        state = memento.savedState
    }
}

// Caretaker
class DrawingHistory {
    private var history: [DrawingMemento] = []
    private var currentIndex = -1
    
    func push(_ memento: DrawingMemento) {
        // Remove forward history on new action
        if currentIndex < history.count - 1 {
            history.removeSubrange((currentIndex + 1)...)
        }
        history.append(memento)
        currentIndex = history.count - 1
    }
    
    func undo() -> DrawingMemento? {
        guard currentIndex > 0 else { return nil }
        currentIndex -= 1
        return history[currentIndex]
    }
    
    func redo() -> DrawingMemento? {
        guard currentIndex < history.count - 1 else { return nil }
        currentIndex += 1
        return history[currentIndex]
    }
    
    var canUndo: Bool { currentIndex > 0 }
    var canRedo: Bool { currentIndex < history.count - 1 }
}
```

---

## 5. Anti-Patterns

```swift
// ❌ Anti-Pattern 1: God Object
class GodController {
    // ทำทุกอย่าง - Network, DB, UI, Analytics, Auth...
    // นี่คือ Anti-Pattern!
}

// ❌ Anti-Pattern 2: Singleton Abuse
class GlobalState {
    static let shared = GlobalState()
    var user: User?
    var cart: [Product] = []
    var settings: AppSettings = AppSettings()
    var analytics: Analytics = Analytics()
    // ทุกอย่างอยู่ใน Singleton - ยากต่อการ Test!
}

// ❌ Anti-Pattern 3: Magic Numbers
class PriceCalculator {
    func calculateDiscount(price: Double) -> Double {
        if price > 1000 {
            return price * 0.85 // Magic number!
        }
        return price * 0.95 // Magic number!
    }
}

// ✅ แก้ไข Magic Numbers
class BetterPriceCalculator {
    private let highValueThreshold = 1000.0
    private let premiumDiscountRate = 0.85
    private let standardDiscountRate = 0.95
    
    func calculateDiscount(price: Double) -> Double {
        if price > highValueThreshold {
            return price * premiumDiscountRate
        }
        return price * standardDiscountRate
    }
}

// ❌ Anti-Pattern 4: Primitive Obsession
class UserFormValidator {
    func validate(
        firstName: String,    // ควรเป็น PersonName type
        lastName: String,
        email: String,        // ควรเป็น EmailAddress type
        phone: String,        // ควรเป็น PhoneNumber type
        age: Int              // ควรเป็น Age type
    ) -> Bool {
        return !firstName.isEmpty && email.contains("@")
    }
}

// ✅ Value Objects แทน Primitives
struct EmailAddress {
    let value: String
    
    init?(_ value: String) {
        guard value.contains("@") && value.contains(".") else { return nil }
        self.value = value
    }
}

struct PhoneNumber {
    let value: String
    
    init?(_ value: String) {
        let digits = value.filter { $0.isNumber }
        guard digits.count >= 9 else { return nil }
        self.value = value
    }
}

struct PersonName {
    let first: String
    let last: String
    var full: String { "\(first) \(last)" }
}
```

---

## 6. Swift-Specific Patterns

### Property Wrappers

```swift
// Custom Property Wrappers เป็น Pattern ที่ Swift-specific
@propertyWrapper
struct Clamped<Value: Comparable> {
    private var value: Value
    private let range: ClosedRange<Value>
    
    var wrappedValue: Value {
        get { value }
        set { value = min(max(newValue, range.lowerBound), range.upperBound) }
    }
    
    init(wrappedValue: Value, _ range: ClosedRange<Value>) {
        self.range = range
        self.value = min(max(wrappedValue, range.lowerBound), range.upperBound)
    }
}

@propertyWrapper
struct Trimmed {
    private var value = ""
    
    var wrappedValue: String {
        get { value }
        set { value = newValue.trimmingCharacters(in: .whitespacesAndNewlines) }
    }
}

@propertyWrapper
struct UserDefault<T> {
    let key: String
    let defaultValue: T
    
    var wrappedValue: T {
        get { UserDefaults.standard.object(forKey: key) as? T ?? defaultValue }
        set { UserDefaults.standard.set(newValue, forKey: key) }
    }
}

// การใช้งาน
class VolumeControl {
    @Clamped(0...100) var volume = 50
    @Clamped(-10...10) var balance = 0
}

class ProfileModel {
    @Trimmed var username: String = ""
    @Trimmed var email: String = ""
}

class AppSettings {
    @UserDefault(key: "isDarkMode", defaultValue: false)
    var isDarkMode: Bool
    
    @UserDefault(key: "fontSize", defaultValue: 16.0)
    var fontSize: Double
    
    @UserDefault(key: "language", defaultValue: "en")
    var language: String
}

// Result Builder Pattern
@resultBuilder
struct HTMLBuilder {
    static func buildBlock(_ components: String...) -> String {
        components.joined(separator: "\n")
    }
    
    static func buildOptional(_ component: String?) -> String {
        component ?? ""
    }
    
    static func buildEither(first component: String) -> String {
        component
    }
    
    static func buildEither(second component: String) -> String {
        component
    }
    
    static func buildArray(_ components: [String]) -> String {
        components.joined(separator: "\n")
    }
}

func tag(_ name: String, @HTMLBuilder content: () -> String) -> String {
    "<\(name)>\(content())</\(name)>"
}

let html = tag("div") {
    tag("h1") { "Hello World" }
    tag("p") { "This is built with ResultBuilder" }
    tag("ul") {
        for item in ["Swift", "iOS", "macOS"] {
            tag("li") { item }
        }
    }
}
```

---

## 7. Functional Patterns

```swift
// Functional Programming Patterns ใน Swift

// 1. Functor (map)
extension Optional {
    // Optional เป็น Functor อยู่แล้ว
    // map applies function only if value exists
}

let maybeName: String? = "Alice"
let maybeUpperName = maybeName.map { $0.uppercased() }
// "ALICE"

// 2. Monad (flatMap)
func findUser(id: Int) -> User? { /* ... */ nil }
func getEmail(user: User) -> String? { user.email }

// flatMap chains optional operations
let email = findUser(id: 1).flatMap { getEmail(user: $0) }

// 3. Pipe Pattern
infix operator |>: AdditionPrecedence

func |> <A, B>(value: A, function: (A) -> B) -> B {
    function(value)
}

let result = "  Hello, World!  "
    |> { $0.trimmingCharacters(in: .whitespaces) }
    |> { $0.lowercased() }
    |> { $0.replacingOccurrences(of: " ", with: "_") }

print(result) // "hello,_world!"

// 4. Memoization
func memoize<Input: Hashable, Output>(
    _ function: @escaping (Input) -> Output
) -> (Input) -> Output {
    var cache: [Input: Output] = [:]
    
    return { input in
        if let cached = cache[input] {
            return cached
        }
        let result = function(input)
        cache[input] = result
        return result
    }
}

// Fibonacci ที่ช้า
func fibonacci(_ n: Int) -> Int {
    if n <= 1 { return n }
    return fibonacci(n - 1) + fibonacci(n - 2)
}

// Fibonacci ที่เร็วด้วย Memoization
let memoFib = memoize { (n: Int) -> Int in
    if n <= 1 { return n }
    return memoFib(n - 1) + memoFib(n - 2)
}

print(memoFib(40)) // เร็วมาก!

// 5. Curry
func curry<A, B, C>(_ f: @escaping (A, B) -> C) -> (A) -> (B) -> C {
    return { a in { b in f(a, b) } }
}

let add: (Int, Int) -> Int = { $0 + $1 }
let curriedAdd = curry(add)
let add5 = curriedAdd(5)

print(add5(3)) // 8
print(add5(10)) // 15

let numbers = [1, 2, 3, 4, 5]
let result2 = numbers.map(add5)
print(result2) // [6, 7, 8, 9, 10]

// 6. Partial Application
func multiply(_ a: Int, by b: Int) -> Int { a * b }

let double: (Int) -> Int = { multiply($0, by: 2) }
let triple: (Int) -> Int = { multiply($0, by: 3) }

print(numbers.map(double)) // [2, 4, 6, 8, 10]
print(numbers.map(triple)) // [3, 6, 9, 12, 15]

// 7. Either / Result
enum Either<Left, Right> {
    case left(Left)
    case right(Right)
    
    func map<T>(_ transform: (Right) -> T) -> Either<Left, T> {
        switch self {
        case .left(let l): return .left(l)
        case .right(let r): return .right(transform(r))
        }
    }
    
    func flatMap<T>(_ transform: (Right) -> Either<Left, T>) -> Either<Left, T> {
        switch self {
        case .left(let l): return .left(l)
        case .right(let r): return transform(r)
        }
    }
}

// 8. Lens Pattern สำหรับ Immutable Updates
struct Lens<Whole, Part> {
    let get: (Whole) -> Part
    let set: (Part, Whole) -> Whole
    
    func modify(_ transform: @escaping (Part) -> Part) -> (Whole) -> Whole {
        return { whole in
            set(transform(get(whole)), whole)
        }
    }
    
    func compose<Subpart>(_ other: Lens<Part, Subpart>) -> Lens<Whole, Subpart> {
        Lens<Whole, Subpart>(
            get: { other.get(self.get($0)) },
            set: { sub, whole in
                self.set(other.set(sub, self.get(whole)), whole)
            }
        )
    }
}

struct Address {
    var street: String
    var city: String
    var country: String
}

struct PersonRecord {
    var name: String
    var age: Int
    var address: Address
}

// Lenses
let nameLens = Lens<PersonRecord, String>(
    get: { $0.name },
    set: { name, person in PersonRecord(name: name, age: person.age, address: person.address) }
)

let addressLens = Lens<PersonRecord, Address>(
    get: { $0.address },
    set: { addr, person in PersonRecord(name: person.name, age: person.age, address: addr) }
)

let cityLens = Lens<Address, String>(
    get: { $0.city },
    set: { city, addr in Address(street: addr.street, city: city, country: addr.country) }
)

let personCityLens = addressLens.compose(cityLens)

let person = PersonRecord(
    name: "Alice",
    age: 30,
    address: Address(street: "123 Main St", city: "Bangkok", country: "Thailand")
)

// Immutable update
let updatedPerson = personCityLens.set("Chiang Mai", person)
print(updatedPerson.address.city) // Chiang Mai
print(person.address.city) // Bangkok (unchanged)
```

---

## 8. แบบฝึกหัด

### แบบฝึกหัดที่ 1: Shopping Cart ด้วย Command + Observer

```swift
// Solution

// Product
struct Product: Identifiable {
    let id: UUID
    let name: String
    let price: Double
    let stock: Int
}

// Cart Item
struct CartItem {
    let product: Product
    var quantity: Int
    var subtotal: Double { product.price * Double(quantity) }
}

// Command
protocol CartCommand {
    func execute(on cart: ShoppingCart) throws
    func undo(on cart: ShoppingCart)
}

class AddToCartCommand: CartCommand {
    private let product: Product
    private let quantity: Int
    
    init(product: Product, quantity: Int = 1) {
        self.product = product
        self.quantity = quantity
    }
    
    func execute(on cart: ShoppingCart) throws {
        try cart.add(product, quantity: quantity)
    }
    
    func undo(on cart: ShoppingCart) {
        cart.remove(productId: product.id, quantity: quantity)
    }
}

class RemoveFromCartCommand: CartCommand {
    private let productId: UUID
    private let quantity: Int
    
    init(productId: UUID, quantity: Int = 1) {
        self.productId = productId
        self.quantity = quantity
    }
    
    func execute(on cart: ShoppingCart) throws {
        cart.remove(productId: productId, quantity: quantity)
    }
    
    func undo(on cart: ShoppingCart) {
        // Undo remove is complex - simplified here
        print("Undo remove not fully implemented")
    }
}

// Observer
protocol CartObserver: AnyObject {
    func cartDidUpdate(_ cart: ShoppingCart)
}

// Shopping Cart (Receiver + Observable)
class ShoppingCart {
    private(set) var items: [CartItem] = []
    private var observers: [WeakCartObserver] = []
    private var commandHistory: [CartCommand] = []
    
    var total: Double { items.reduce(0) { $0 + $1.subtotal } }
    var itemCount: Int { items.reduce(0) { $0 + $1.quantity } }
    
    enum CartError: Error {
        case outOfStock
        case productNotFound
    }
    
    func add(_ product: Product, quantity: Int = 1) throws {
        guard product.stock >= quantity else { throw CartError.outOfStock }
        
        if let index = items.firstIndex(where: { $0.product.id == product.id }) {
            items[index].quantity += quantity
        } else {
            items.append(CartItem(product: product, quantity: quantity))
        }
        notifyObservers()
    }
    
    func remove(productId: UUID, quantity: Int = 1) {
        guard let index = items.firstIndex(where: { $0.product.id == productId }) else { return }
        
        if items[index].quantity <= quantity {
            items.remove(at: index)
        } else {
            items[index].quantity -= quantity
        }
        notifyObservers()
    }
    
    func execute(_ command: CartCommand) throws {
        try command.execute(on: self)
        commandHistory.append(command)
    }
    
    func undoLast() {
        guard let command = commandHistory.popLast() else { return }
        command.undo(on: self)
    }
    
    func addObserver(_ observer: CartObserver) {
        observers.append(WeakCartObserver(observer))
    }
    
    private func notifyObservers() {
        observers.forEach { $0.observer?.cartDidUpdate(self) }
    }
    
    private struct WeakCartObserver {
        weak var observer: CartObserver?
        init(_ observer: CartObserver) { self.observer = observer }
    }
}

// UI Observer
class CartBadgeView: CartObserver {
    func cartDidUpdate(_ cart: ShoppingCart) {
        print("Badge: \(cart.itemCount) items, Total: ฿\(String(format: "%.2f", cart.total))")
    }
}

// การใช้งาน
let cart = ShoppingCart()
let badge = CartBadgeView()
cart.addObserver(badge)

let phone = Product(id: UUID(), name: "iPhone 15", price: 35900, stock: 5)
let case1 = Product(id: UUID(), name: "iPhone Case", price: 490, stock: 20)

try? cart.execute(AddToCartCommand(product: phone))
// Badge: 1 items, Total: ฿35900.00

try? cart.execute(AddToCartCommand(product: case1, quantity: 2))
// Badge: 3 items, Total: ฿36880.00

cart.undoLast()
// Badge: 1 items, Total: ฿35900.00
```

---

## 9. Real-world iOS Examples

### URLSession + Delegate Pattern (Observer)

```swift
// Apple ใช้ Delegate Pattern อย่างกว้างขวาง
class DownloadManager: NSObject {
    private lazy var session: URLSession = {
        let config = URLSessionConfiguration.background(withIdentifier: "com.app.download")
        return URLSession(configuration: config, delegate: self, delegateQueue: nil)
    }()
    
    private var completionHandlers: [Int: (Result<URL, Error>) -> Void] = [:]
    private var progressHandlers: [Int: (Double) -> Void] = [:]
    
    func download(
        url: URL,
        progress: @escaping (Double) -> Void,
        completion: @escaping (Result<URL, Error>) -> Void
    ) {
        let task = session.downloadTask(with: url)
        completionHandlers[task.taskIdentifier] = completion
        progressHandlers[task.taskIdentifier] = progress
        task.resume()
    }
}

extension DownloadManager: URLSessionDownloadDelegate {
    func urlSession(
        _ session: URLSession,
        downloadTask: URLSessionDownloadTask,
        didFinishDownloadingTo location: URL
    ) {
        let handler = completionHandlers.removeValue(forKey: downloadTask.taskIdentifier)
        handler?(.success(location))
    }
    
    func urlSession(
        _ session: URLSession,
        task: URLSessionTask,
        didCompleteWithError error: Error?
    ) {
        if let error = error {
            let handler = completionHandlers.removeValue(forKey: task.taskIdentifier)
            handler?(.failure(error))
        }
        progressHandlers.removeValue(forKey: task.taskIdentifier)
    }
    
    func urlSession(
        _ session: URLSession,
        downloadTask: URLSessionDownloadTask,
        didWriteData bytesWritten: Int64,
        totalBytesWritten: Int64,
        totalBytesExpectedToWrite: Int64
    ) {
        guard totalBytesExpectedToWrite > 0 else { return }
        let progress = Double(totalBytesWritten) / Double(totalBytesExpectedToWrite)
        progressHandlers[downloadTask.taskIdentifier]?(progress)
    }
}
```

### SwiftUI + Combine (Observer + Strategy)

```swift
// Real-world SwiftUI app using multiple patterns
import SwiftUI
import Combine

// Strategy Pattern สำหรับ Image Loading
protocol ImageFetcher {
    func fetchImage(from url: URL) -> AnyPublisher<UIImage, Error>
}

class NetworkImageFetcher: ImageFetcher {
    func fetchImage(from url: URL) -> AnyPublisher<UIImage, Error> {
        URLSession.shared.dataTaskPublisher(for: url)
            .map(\.data)
            .tryMap { data -> UIImage in
                guard let image = UIImage(data: data) else {
                    throw URLError(.cannotDecodeContentData)
                }
                return image
            }
            .eraseToAnyPublisher()
    }
}

// Decorator Pattern สำหรับ Caching
class CachedImageFetcher: ImageFetcher {
    private let wrapped: ImageFetcher
    private let cache = NSCache<NSURL, UIImage>()
    
    init(wrapped: ImageFetcher) {
        self.wrapped = wrapped
    }
    
    func fetchImage(from url: URL) -> AnyPublisher<UIImage, Error> {
        if let cached = cache.object(forKey: url as NSURL) {
            return Just(cached)
                .setFailureType(to: Error.self)
                .eraseToAnyPublisher()
        }
        
        return wrapped.fetchImage(from: url)
            .handleEvents(receiveOutput: { [weak self] image in
                self?.cache.setObject(image, forKey: url as NSURL)
            })
            .eraseToAnyPublisher()
    }
}

// ViewModel
class AsyncImageViewModel: ObservableObject {
    @Published var image: UIImage?
    @Published var isLoading = false
    @Published var error: Error?
    
    private let fetcher: ImageFetcher
    private var cancellable: AnyCancellable?
    
    init(fetcher: ImageFetcher = CachedImageFetcher(wrapped: NetworkImageFetcher())) {
        self.fetcher = fetcher
    }
    
    func load(from url: URL) {
        isLoading = true
        error = nil
        
        cancellable = fetcher.fetchImage(from: url)
            .receive(on: DispatchQueue.main)
            .sink(
                receiveCompletion: { [weak self] completion in
                    self?.isLoading = false
                    if case .failure(let error) = completion {
                        self?.error = error
                    }
                },
                receiveValue: { [weak self] image in
                    self?.image = image
                }
            )
    }
    
    func cancel() {
        cancellable?.cancel()
        isLoading = false
    }
}

// SwiftUI View
struct AsyncImageView: View {
    let url: URL
    @StateObject private var viewModel = AsyncImageViewModel()
    
    var body: some View {
        Group {
            if viewModel.isLoading {
                ProgressView()
            } else if let image = viewModel.image {
                Image(uiImage: image)
                    .resizable()
                    .scaledToFit()
            } else if viewModel.error != nil {
                Image(systemName: "photo")
                    .foregroundColor(.gray)
            } else {
                Color.clear
            }
        }
        .onAppear { viewModel.load(from: url) }
        .onDisappear { viewModel.cancel() }
    }
}
```

---

## 10. สรุป

### ตารางสรุป Design Patterns

| Pattern | ประเภท | ใช้เมื่อ |
|---------|--------|---------|
| Singleton | Creational | ต้องการ Single Instance |
| Factory Method | Creational | Subclass กำหนดว่าจะสร้าง Object ไหน |
| Abstract Factory | Creational | สร้าง Family ของ Objects |
| Builder | Creational | สร้าง Complex Object ทีละขั้น |
| Prototype | Creational | Clone Object ที่มีอยู่ |
| Adapter | Structural | ทำให้ Interface เข้ากันได้ |
| Bridge | Structural | แยก Abstraction จาก Implementation |
| Composite | Structural | จัดการ Tree Structure |
| Decorator | Structural | เพิ่ม Behavior โดยไม่แก้ Class |
| Facade | Structural | ทำ Simple Interface |
| Flyweight | Structural | Share State เพื่อประหยัด Memory |
| Proxy | Structural | Control Access to Object |
| Chain of Responsibility | Behavioral | Pass Request ตาม Chain |
| Command | Behavioral | Encapsulate Request เป็น Object |
| Iterator | Behavioral | Traverse Collection |
| Mediator | Behavioral | ลด Direct Dependencies |
| Memento | Behavioral | Snapshot และ Restore State |
| Observer | Behavioral | แจ้งเตือนเมื่อ State เปลี่ยน |
| State | Behavioral | เปลี่ยน Behavior ตาม State |
| Strategy | Behavioral | เปลี่ยน Algorithm ได้ Runtime |
| Template Method | Behavioral | กำหนด Algorithm Skeleton |
| Visitor | Behavioral | เพิ่ม Operation โดยไม่แก้ Class |

### หลักการใช้ Design Patterns

1. **ใช้เมื่อจำเป็น** - ไม่ต้องใช้ทุก Pattern ในทุก Project
2. **เข้าใจก่อนใช้** - รู้ว่า Pattern แก้ปัญหาอะไร
3. **Keep It Simple** - Simple Solution ดีกว่า Overcomplicated Pattern
4. **Combine Patterns** - Patterns มักใช้ร่วมกัน
5. **Swift Idioms** - Swift มี Features เช่น Protocol, Extension ที่ช่วยได้

### Swift Patterns ที่ใช้บ่อย

```swift
// 1. Protocol + Extension (Interface Segregation)
protocol Printable { func print() }
protocol Saveable { func save() }

extension Printable where Self: Saveable {
    func printAndSave() {
        self.print()
        self.save()
    }
}

// 2. Associated Types (Generic Patterns)
protocol Container {
    associatedtype Item
    var items: [Item] { get }
    mutating func add(_ item: Item)
}

// 3. KeyPath (Lens Pattern)
struct Config {
    var timeout: TimeInterval = 30
    var maxRetries: Int = 3
}

var config = Config()
config[keyPath: \.timeout] = 60
config[keyPath: \.maxRetries] = 5

// 4. Result Type (Railway Pattern)
func validateAge(_ age: Int) -> Result<Int, ValidationError> {
    guard age >= 0 else { return .failure(.negative) }
    guard age <= 150 else { return .failure(.tooLarge) }
    return .success(age)
}
```

### สิ่งสำคัญที่ควรจำ

- **Creational Patterns**: เกี่ยวกับ Object Creation
- **Structural Patterns**: เกี่ยวกับ Class Composition
- **Behavioral Patterns**: เกี่ยวกับ Object Communication
- Swift's Protocol, Generics, Closures ทำให้หลาย Pattern ใช้ง่ายขึ้น
- Anti-patterns เช่น God Object, Singleton Abuse ควรหลีกเลี่ยง
- ทดสอบโค้ดเสมอเพื่อยืนยันว่า Pattern ทำงานถูกต้อง

---

*จบ Part 38: Design Patterns ใน Swift*

**ต่อไป**: Part 39 - Testing ใน Swift (Unit Testing, UI Testing)
