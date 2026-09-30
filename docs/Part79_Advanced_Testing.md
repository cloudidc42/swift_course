# Part 79: Advanced Testing in Swift

## บทนำ

การทดสอบซอฟต์แวร์เป็นส่วนสำคัญของการพัฒนาแอปพลิเคชันที่มีคุณภาพ ในบทนี้เราจะเจาะลึกเทคนิคการทดสอบขั้นสูงใน Swift ตั้งแต่ XCTest พื้นฐานไปจนถึง Swift Testing framework ใหม่ รวมถึงการทดสอบ async/await, Combine, Core Data, และการทดสอบ UI ด้วย Page Object Model

---

## 1. Advanced XCTest Techniques

### 1.1 XCTest Lifecycle

```swift
import XCTest

class AdvancedXCTestExample: XCTestCase {
    
    // เรียกครั้งเดียวก่อน test methods ทั้งหมดใน class
    override class func setUp() {
        super.setUp()
        // ตั้งค่า static resources
        UserDefaults.standard.set("test", forKey: "environment")
    }
    
    // เรียกก่อนแต่ละ test method
    override func setUp() {
        super.setUp()
        // ตั้งค่า instance resources
        continueAfterFailure = false
    }
    
    // เรียกหลังแต่ละ test method
    override func tearDown() {
        // ล้าง instance resources
        super.tearDown()
    }
    
    // เรียกครั้งเดียวหลัง test methods ทั้งหมดใน class
    override class func tearDown() {
        UserDefaults.standard.removeObject(forKey: "environment")
        super.tearDown()
    }
    
    func testExample() {
        // การทดสอบ
        XCTAssertTrue(true)
    }
}
```

### 1.2 XCTest Assertions ขั้นสูง

```swift
class AssertionExamples: XCTestCase {
    
    func testBasicAssertions() {
        // Assert ค่าเท่ากัน
        XCTAssertEqual(2 + 2, 4)
        XCTAssertNotEqual(2 + 2, 5)
        
        // Assert boolean
        XCTAssertTrue(1 < 2)
        XCTAssertFalse(1 > 2)
        
        // Assert nil / not nil
        let optional: String? = "hello"
        XCTAssertNotNil(optional)
        
        let nilValue: String? = nil
        XCTAssertNil(nilValue)
    }
    
    func testFloatingPointAssertions() {
        // ใช้ accuracy สำหรับ floating point
        XCTAssertEqual(0.1 + 0.2, 0.3, accuracy: 0.001)
        
        let pi = Double.pi
        XCTAssertEqual(pi, 3.14159, accuracy: 0.00001)
    }
    
    func testThrowAssertions() {
        enum TestError: Error {
            case someError
        }
        
        func throwingFunction() throws {
            throw TestError.someError
        }
        
        // Assert ว่า throws
        XCTAssertThrowsError(try throwingFunction()) { error in
            XCTAssertEqual(error as? TestError, TestError.someError)
        }
        
        // Assert ว่าไม่ throws
        func nonThrowingFunction() throws -> Int {
            return 42
        }
        XCTAssertNoThrow(try nonThrowingFunction())
    }
    
    func testCustomFailureMessages() {
        let value = 42
        XCTAssertEqual(value, 42, "ค่าควรเป็น 42 แต่ได้ \(value)")
    }
    
    func testUnwrappingOptionals() throws {
        let optional: String? = "hello"
        
        // ใช้ XCTUnwrap เพื่อ unwrap optional (throws ถ้า nil)
        let value = try XCTUnwrap(optional)
        XCTAssertEqual(value, "hello")
    }
}
```

### 1.3 Asynchronous Testing ด้วย XCTestExpectation

```swift
class AsyncTestingExamples: XCTestCase {
    
    func testWithExpectation() {
        let expectation = expectation(description: "Async operation completed")
        
        DispatchQueue.global().asyncAfter(deadline: .now() + 0.5) {
            // ทำงาน async
            expectation.fulfill()
        }
        
        waitForExpectations(timeout: 2.0) { error in
            if let error = error {
                XCTFail("Expectation failed: \(error)")
            }
        }
    }
    
    func testMultipleExpectations() {
        let firstExpectation = expectation(description: "First operation")
        let secondExpectation = expectation(description: "Second operation")
        
        DispatchQueue.global().async {
            firstExpectation.fulfill()
        }
        
        DispatchQueue.global().async {
            secondExpectation.fulfill()
        }
        
        wait(for: [firstExpectation, secondExpectation], timeout: 2.0)
    }
    
    func testOrderedExpectations() {
        let firstExpectation = expectation(description: "First")
        let secondExpectation = expectation(description: "Second")
        
        // enforceOrder: true - ต้อง fulfill ตามลำดับ
        wait(for: [firstExpectation, secondExpectation], timeout: 2.0, enforceOrder: true)
    }
    
    func testInvertedExpectation() {
        let expectation = expectation(description: "Should NOT be called")
        expectation.isInverted = true
        
        // expectation ที่ inverted จะ fail ถ้าถูก fulfill
        // ใช้สำหรับ test ว่าบางอย่างไม่เกิดขึ้น
        
        wait(for: [expectation], timeout: 0.5)
    }
    
    // iOS 15+ / Swift 5.5+ async test
    func testAsyncAwait() async throws {
        let result = try await fetchData()
        XCTAssertEqual(result, "expected data")
    }
    
    private func fetchData() async throws -> String {
        try await Task.sleep(nanoseconds: 100_000_000)
        return "expected data"
    }
}
```

### 1.4 Performance Testing

```swift
class PerformanceTests: XCTestCase {
    
    func testSortingPerformance() {
        let array = (0..<10000).shuffled()
        
        measure {
            let _ = array.sorted()
        }
    }
    
    func testWithMetrics() {
        let metrics: [XCTMetric] = [
            XCTClockMetric(),
            XCTMemoryMetric(),
            XCTCPUMetric()
        ]
        
        let options = XCTMeasureOptions()
        options.iterationCount = 5
        
        measure(metrics: metrics, options: options) {
            var result = 0
            for i in 0..<100000 {
                result += i
            }
        }
    }
}
```

---

## 2. Test Doubles: Fakes, Stubs, Mocks, Spies, Dummies

Test doubles คือ objects ที่ใช้แทน dependencies จริงในการทดสอบ แต่ละประเภทมีวัตถุประสงค์ต่างกัน

### 2.1 Dummy

Dummy คือ object ที่ถูกส่งผ่านไปแต่ไม่ถูกใช้จริง

```swift
// Protocol ที่ต้องการ
protocol Logger {
    func log(_ message: String)
}

// Dummy Logger - ไม่ทำอะไรเลย
class DummyLogger: Logger {
    func log(_ message: String) {
        // ไม่ทำอะไร - dummy ไม่สนใจการเรียก
    }
}

// Service ที่ต้องการ Logger
class UserService {
    private let logger: Logger
    
    init(logger: Logger) {
        self.logger = logger
    }
    
    func createUser(name: String) -> Bool {
        guard !name.isEmpty else { return false }
        logger.log("Created user: \(name)")
        return true
    }
}

// Test ที่ใช้ Dummy
class UserServiceTests: XCTestCase {
    func testCreateUserWithValidName() {
        let dummy = DummyLogger()
        let service = UserService(logger: dummy)
        
        // เราไม่สนใจว่า logger จะถูกเรียกหรือไม่
        XCTAssertTrue(service.createUser(name: "John"))
    }
}
```

### 2.2 Stub

Stub ให้ค่าที่กำหนดไว้ล่วงหน้าเมื่อถูกเรียก

```swift
protocol NetworkService {
    func fetchUser(id: Int) async throws -> User
}

struct User: Equatable {
    let id: Int
    let name: String
    let email: String
}

// Stub ที่ return ค่าที่กำหนดไว้
class StubNetworkService: NetworkService {
    var stubbedUser: User?
    var stubbedError: Error?
    
    func fetchUser(id: Int) async throws -> User {
        if let error = stubbedError {
            throw error
        }
        guard let user = stubbedUser else {
            throw NSError(domain: "Test", code: 0)
        }
        return user
    }
}

// ViewModel ที่ใช้ NetworkService
class UserViewModel {
    private let networkService: NetworkService
    var user: User?
    var errorMessage: String?
    
    init(networkService: NetworkService) {
        self.networkService = networkService
    }
    
    func loadUser(id: Int) async {
        do {
            user = try await networkService.fetchUser(id: id)
        } catch {
            errorMessage = error.localizedDescription
        }
    }
}

// Test ที่ใช้ Stub
class UserViewModelTests: XCTestCase {
    func testLoadUserSuccess() async {
        let stub = StubNetworkService()
        stub.stubbedUser = User(id: 1, name: "Alice", email: "alice@example.com")
        
        let viewModel = UserViewModel(networkService: stub)
        await viewModel.loadUser(id: 1)
        
        XCTAssertEqual(viewModel.user?.name, "Alice")
        XCTAssertNil(viewModel.errorMessage)
    }
    
    func testLoadUserFailure() async {
        let stub = StubNetworkService()
        stub.stubbedError = NSError(domain: "Test", code: 404, 
                                    userInfo: [NSLocalizedDescriptionKey: "Not found"])
        
        let viewModel = UserViewModel(networkService: stub)
        await viewModel.loadUser(id: 999)
        
        XCTAssertNil(viewModel.user)
        XCTAssertNotNil(viewModel.errorMessage)
    }
}
```

### 2.3 Mock

Mock ตรวจสอบว่า method ถูกเรียกถูกต้องหรือไม่

```swift
protocol AnalyticsService {
    func track(event: String, parameters: [String: Any])
}

// Mock ที่บันทึกการเรียก
class MockAnalyticsService: AnalyticsService {
    var trackedEvents: [(event: String, parameters: [String: Any])] = []
    var trackCallCount = 0
    
    func track(event: String, parameters: [String: Any]) {
        trackCallCount += 1
        trackedEvents.append((event: event, parameters: parameters))
    }
    
    // Helper methods สำหรับ assertion
    func verifyTracked(event: String) -> Bool {
        return trackedEvents.contains { $0.event == event }
    }
    
    func verifyTracked(event: String, times: Int) -> Bool {
        return trackedEvents.filter { $0.event == event }.count == times
    }
}

class CheckoutService {
    private let analytics: AnalyticsService
    
    init(analytics: AnalyticsService) {
        self.analytics = analytics
    }
    
    func processPayment(amount: Double) {
        analytics.track(event: "payment_initiated", parameters: ["amount": amount])
        // ... process payment ...
        analytics.track(event: "payment_completed", parameters: ["amount": amount])
    }
}

class CheckoutServiceTests: XCTestCase {
    func testPaymentTracksEvents() {
        let mock = MockAnalyticsService()
        let service = CheckoutService(analytics: mock)
        
        service.processPayment(amount: 99.99)
        
        XCTAssertTrue(mock.verifyTracked(event: "payment_initiated"))
        XCTAssertTrue(mock.verifyTracked(event: "payment_completed"))
        XCTAssertEqual(mock.trackCallCount, 2)
    }
}
```

### 2.4 Spy

Spy บันทึกการเรียกและยังเรียก implementation จริงด้วย

```swift
class SpyLogger: Logger {
    private let realLogger: Logger
    var loggedMessages: [String] = []
    
    init(wrapping realLogger: Logger) {
        self.realLogger = realLogger
    }
    
    func log(_ message: String) {
        loggedMessages.append(message)
        realLogger.log(message) // เรียก implementation จริง
    }
}

// ConsoleLogger จริง
class ConsoleLogger: Logger {
    func log(_ message: String) {
        print("[LOG] \(message)")
    }
}

class SpyTests: XCTestCase {
    func testSpyRecordsAndForwards() {
        let realLogger = ConsoleLogger()
        let spy = SpyLogger(wrapping: realLogger)
        let service = UserService(logger: spy)
        
        _ = service.createUser(name: "Bob")
        
        // ตรวจสอบว่า log ถูกบันทึก
        XCTAssertEqual(spy.loggedMessages.count, 1)
        XCTAssertTrue(spy.loggedMessages[0].contains("Bob"))
    }
}
```

### 2.5 Fake

Fake มี implementation จริงแต่ simplified สำหรับการทดสอบ

```swift
protocol UserRepository {
    func save(user: User) throws
    func findById(_ id: Int) -> User?
    func findAll() -> [User]
    func delete(id: Int) throws
}

// Fake ที่ใช้ in-memory storage แทน database จริง
class FakeUserRepository: UserRepository {
    private var storage: [Int: User] = [:]
    
    func save(user: User) throws {
        storage[user.id] = user
    }
    
    func findById(_ id: Int) -> User? {
        return storage[id]
    }
    
    func findAll() -> [User] {
        return Array(storage.values)
    }
    
    func delete(id: Int) throws {
        guard storage[id] != nil else {
            throw NSError(domain: "NotFound", code: 404)
        }
        storage.removeValue(forKey: id)
    }
}

class UserRepositoryTests: XCTestCase {
    var repository: FakeUserRepository!
    
    override func setUp() {
        super.setUp()
        repository = FakeUserRepository()
    }
    
    func testSaveAndFind() throws {
        let user = User(id: 1, name: "Charlie", email: "charlie@example.com")
        
        try repository.save(user: user)
        let found = repository.findById(1)
        
        XCTAssertEqual(found?.name, "Charlie")
    }
    
    func testDeleteNonExistentUserThrows() {
        XCTAssertThrowsError(try repository.delete(id: 999))
    }
}
```

---

## 3. Protocol-Based Mocking

Protocol-based mocking เป็นเทคนิคที่ใช้ Swift protocols เพื่อสร้าง mock objects

### 3.1 การออกแบบ Protocol สำหรับ Testability

```swift
// Protocol ที่ครอบ URLSession
protocol URLSessionProtocol {
    func data(from url: URL) async throws -> (Data, URLResponse)
    func data(for request: URLRequest) async throws -> (Data, URLResponse)
}

// URLSession conform protocol
extension URLSession: URLSessionProtocol {}

// Mock URLSession
class MockURLSession: URLSessionProtocol {
    var mockData: Data?
    var mockResponse: URLResponse?
    var mockError: Error?
    var requestsMade: [URLRequest] = []
    
    func data(from url: URL) async throws -> (Data, URLResponse) {
        if let error = mockError { throw error }
        let data = mockData ?? Data()
        let response = mockResponse ?? HTTPURLResponse(url: url, statusCode: 200, 
                                                        httpVersion: nil, 
                                                        headerFields: nil)!
        return (data, response)
    }
    
    func data(for request: URLRequest) async throws -> (Data, URLResponse) {
        requestsMade.append(request)
        return try await data(from: request.url!)
    }
}

// API Client ที่ใช้ protocol
class APIClient {
    private let session: URLSessionProtocol
    private let baseURL: URL
    
    init(session: URLSessionProtocol = URLSession.shared, 
         baseURL: URL = URL(string: "https://api.example.com")!) {
        self.session = session
        self.baseURL = baseURL
    }
    
    func fetchUsers() async throws -> [User] {
        let url = baseURL.appendingPathComponent("users")
        let (data, _) = try await session.data(from: url)
        return try JSONDecoder().decode([User].self, from: data)
    }
}

// Test
class APIClientTests: XCTestCase {
    func testFetchUsers() async throws {
        let mockSession = MockURLSession()
        
        let users = [User(id: 1, name: "Alice", email: "alice@example.com")]
        mockSession.mockData = try JSONEncoder().encode(users)
        
        let client = APIClient(session: mockSession)
        let fetchedUsers = try await client.fetchUsers()
        
        XCTAssertEqual(fetchedUsers.count, 1)
        XCTAssertEqual(fetchedUsers[0].name, "Alice")
    }
}
```

### 3.2 Protocol Composition สำหรับ Mock

```swift
protocol Readable {
    func read(key: String) -> String?
}

protocol Writable {
    func write(key: String, value: String)
}

protocol Storage: Readable & Writable {}

class MockStorage: Storage {
    var store: [String: String] = [:]
    var readCalls: [String] = []
    var writeCalls: [(key: String, value: String)] = []
    
    func read(key: String) -> String? {
        readCalls.append(key)
        return store[key]
    }
    
    func write(key: String, value: String) {
        writeCalls.append((key: key, value: value))
        store[key] = value
    }
}
```

---

## 4. Property-Based Testing กับ SwiftCheck

Property-based testing เป็นเทคนิคที่สร้าง test inputs แบบ random เพื่อหา edge cases

### 4.1 แนวคิด Property-Based Testing

```swift
// แทนที่จะเขียน example-based tests:
func testReverse_ExampleBased() {
    XCTAssertEqual([1, 2, 3].reversed(), [3, 2, 1])
}

// เราเขียน property-based tests:
// Property: reversing twice returns original
// Property: reversed array has same count
// Property: first element of reversed = last element of original
```

### 4.2 SwiftCheck Integration

```swift
// Package.swift dependency:
// .package(url: "https://github.com/typelift/SwiftCheck.git", from: "0.12.0")

import XCTest
import SwiftCheck

class PropertyBasedTests: XCTestCase {
    
    func testReverseProperty() {
        // Property: reversing an array twice gives original array
        property("reversing twice is identity") <- forAll { (array: [Int]) in
            return array.reversed().reversed() == array
        }
    }
    
    func testSortProperty() {
        // Property: sorted array should be ordered
        property("sorted array is ordered") <- forAll { (array: [Int]) in
            let sorted = array.sorted()
            return zip(sorted, sorted.dropFirst()).allSatisfy { $0 <= $1 }
        }
    }
    
    func testStringConcatenation() {
        property("string concatenation length") <- forAll { (s1: String, s2: String) in
            return (s1 + s2).count == s1.count + s2.count
        }
    }
    
    // Custom Generator
    struct Email {
        let value: String
    }
    
    func testEmailValidation() {
        let emailGen = String.arbitrary.map { name in
            Email(value: "\(name)@example.com")
        }
        
        property("valid emails always pass") <- forAll(emailGen) { email in
            return isValidEmail(email.value)
        }
    }
    
    func isValidEmail(_ email: String) -> Bool {
        let regex = "[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,64}"
        let predicate = NSPredicate(format: "SELF MATCHES %@", regex)
        return predicate.evaluate(with: email)
    }
}
```

---

## 5. Snapshot Testing กับ swift-snapshot-testing

Snapshot testing บันทึกภาพของ UI และเปรียบเทียบกับ snapshots ที่บันทึกไว้

### 5.1 Setup

```swift
// Package.swift:
// .package(url: "https://github.com/pointfreeco/swift-snapshot-testing", from: "1.15.0")

import XCTest
import SnapshotTesting
import UIKit

class SnapshotTests: XCTestCase {
    
    func testButtonSnapshot() {
        let button = UIButton(type: .system)
        button.setTitle("Click Me", for: .normal)
        button.backgroundColor = .systemBlue
        button.tintColor = .white
        button.layer.cornerRadius = 8
        button.frame = CGRect(x: 0, y: 0, width: 200, height: 44)
        
        // ครั้งแรก: บันทึก snapshot
        // ครั้งต่อไป: เปรียบเทียบกับที่บันทึกไว้
        assertSnapshot(matching: button, as: .image)
    }
    
    func testViewControllerSnapshot() {
        let vc = MyViewController()
        vc.loadViewIfNeeded()
        
        // Test หลาย device sizes
        assertSnapshot(matching: vc, as: .image(on: .iPhone13))
        assertSnapshot(matching: vc, as: .image(on: .iPadPro11))
    }
    
    func testSnapshotWithRecord() {
        // record: true จะบันทึก snapshot ใหม่
        let view = MyCustomView()
        assertSnapshot(matching: view, as: .image, record: true)
    }
}
```

### 5.2 SwiftUI Snapshot Testing

```swift
import SwiftUI
import SnapshotTesting

class SwiftUISnapshotTests: XCTestCase {
    
    func testSwiftUIView() {
        let view = ContentView()
        
        assertSnapshot(matching: view, as: .image(layout: .device(config: .iPhone13)))
    }
    
    func testDarkModeSnapshot() {
        let view = ContentView()
            .environment(\.colorScheme, .dark)
        
        assertSnapshot(matching: view, as: .image(layout: .device(config: .iPhone13)))
    }
}

struct ContentView: View {
    var body: some View {
        VStack {
            Text("Hello, World!")
                .font(.title)
            Button("Tap Me") { }
                .buttonStyle(.borderedProminent)
        }
        .padding()
    }
}
```

---

## 6. Network Testing กับ MockURLProtocol

### 6.1 MockURLProtocol Implementation

```swift
import Foundation

class MockURLProtocol: URLProtocol {
    // เก็บ mock responses
    static var requestHandler: ((URLRequest) throws -> (HTTPURLResponse, Data))?
    
    override class func canInit(with request: URLRequest) -> Bool {
        return true // Handle requests ทั้งหมด
    }
    
    override class func canonicalRequest(for request: URLRequest) -> URLRequest {
        return request
    }
    
    override func startLoading() {
        guard let handler = MockURLProtocol.requestHandler else {
            XCTFail("No request handler set")
            return
        }
        
        do {
            let (response, data) = try handler(request)
            client?.urlProtocol(self, didReceive: response, cacheStoragePolicy: .notAllowed)
            client?.urlProtocol(self, didLoad: data)
            client?.urlProtocolDidFinishLoading(self)
        } catch {
            client?.urlProtocol(self, didFailWithError: error)
        }
    }
    
    override func stopLoading() {}
}

// การใช้ MockURLProtocol
class NetworkTestingExamples: XCTestCase {
    
    var session: URLSession!
    
    override func setUp() {
        super.setUp()
        
        let config = URLSessionConfiguration.ephemeral
        config.protocolClasses = [MockURLProtocol.self]
        session = URLSession(configuration: config)
    }
    
    func testSuccessfulRequest() async throws {
        let expectedData = """
        {"id": 1, "name": "Alice", "email": "alice@example.com"}
        """.data(using: .utf8)!
        
        MockURLProtocol.requestHandler = { request in
            let response = HTTPURLResponse(
                url: request.url!,
                statusCode: 200,
                httpVersion: nil,
                headerFields: ["Content-Type": "application/json"]
            )!
            return (response, expectedData)
        }
        
        let url = URL(string: "https://api.example.com/users/1")!
        let (data, response) = try await session.data(from: url)
        
        let httpResponse = response as? HTTPURLResponse
        XCTAssertEqual(httpResponse?.statusCode, 200)
        
        let user = try JSONDecoder().decode(User.self, from: data)
        XCTAssertEqual(user.name, "Alice")
    }
    
    func testFailedRequest() async {
        MockURLProtocol.requestHandler = { _ in
            throw URLError(.notConnectedToInternet)
        }
        
        let url = URL(string: "https://api.example.com/users/1")!
        
        do {
            _ = try await session.data(from: url)
            XCTFail("Expected error to be thrown")
        } catch {
            XCTAssertNotNil(error)
        }
    }
    
    func testRequestHeaders() async throws {
        var capturedRequest: URLRequest?
        
        MockURLProtocol.requestHandler = { request in
            capturedRequest = request
            let response = HTTPURLResponse(url: request.url!, statusCode: 200, 
                                           httpVersion: nil, headerFields: nil)!
            return (response, Data())
        }
        
        var request = URLRequest(url: URL(string: "https://api.example.com/users")!)
        request.addValue("Bearer token123", forHTTPHeaderField: "Authorization")
        
        _ = try await session.data(for: request)
        
        XCTAssertEqual(capturedRequest?.value(forHTTPHeaderField: "Authorization"), 
                       "Bearer token123")
    }
}
```

---

## 7. Core Data Testing กับ In-Memory Store

### 7.1 การตั้งค่า In-Memory Store

```swift
import CoreData
import XCTest

// Core Data Stack สำหรับ Testing
class TestCoreDataStack {
    
    lazy var persistentContainer: NSPersistentContainer = {
        let container = NSPersistentContainer(name: "MyApp")
        
        // ใช้ in-memory store สำหรับ testing
        let description = NSPersistentStoreDescription()
        description.type = NSInMemoryStoreType
        description.shouldAddStoreAsynchronously = false
        
        container.persistentStoreDescriptions = [description]
        
        container.loadPersistentStores { _, error in
            if let error = error {
                fatalError("Failed to load test store: \(error)")
            }
        }
        
        return container
    }()
    
    var context: NSManagedObjectContext {
        return persistentContainer.viewContext
    }
}

// Core Data Model (สมมติ)
// Entity: PersonEntity
// Attributes: id (Int64), name (String), email (String)

class CoreDataTests: XCTestCase {
    
    var stack: TestCoreDataStack!
    var context: NSManagedObjectContext!
    
    override func setUp() {
        super.setUp()
        stack = TestCoreDataStack()
        context = stack.context
    }
    
    override func tearDown() {
        stack = nil
        context = nil
        super.tearDown()
    }
    
    func testCreatePerson() throws {
        // สร้าง entity
        let person = NSEntityDescription.insertNewObject(
            forEntityName: "PersonEntity",
            into: context
        )
        person.setValue(1, forKey: "id")
        person.setValue("Alice", forKey: "name")
        person.setValue("alice@example.com", forKey: "email")
        
        try context.save()
        
        // Fetch และตรวจสอบ
        let fetchRequest = NSFetchRequest<NSManagedObject>(entityName: "PersonEntity")
        let results = try context.fetch(fetchRequest)
        
        XCTAssertEqual(results.count, 1)
        XCTAssertEqual(results[0].value(forKey: "name") as? String, "Alice")
    }
    
    func testFetchWithPredicate() throws {
        // เพิ่มข้อมูล
        for i in 1...5 {
            let person = NSEntityDescription.insertNewObject(
                forEntityName: "PersonEntity",
                into: context
            )
            person.setValue(Int64(i), forKey: "id")
            person.setValue("Person \(i)", forKey: "name")
            person.setValue("person\(i)@example.com", forKey: "email")
        }
        try context.save()
        
        // Fetch ด้วย predicate
        let fetchRequest = NSFetchRequest<NSManagedObject>(entityName: "PersonEntity")
        fetchRequest.predicate = NSPredicate(format: "id > 3")
        
        let results = try context.fetch(fetchRequest)
        XCTAssertEqual(results.count, 2)
    }
}
```

---

## 8. Combine Publisher Testing

### 8.1 Testing Combine Pipelines

```swift
import Combine
import XCTest

class CombineTestingExamples: XCTestCase {
    
    var cancellables = Set<AnyCancellable>()
    
    override func tearDown() {
        cancellables.removeAll()
        super.tearDown()
    }
    
    func testPublisherWithExpectation() {
        let expectation = expectation(description: "Publisher completes")
        var receivedValues: [Int] = []
        
        [1, 2, 3, 4, 5].publisher
            .filter { $0 % 2 == 0 }
            .sink(
                receiveCompletion: { _ in expectation.fulfill() },
                receiveValue: { receivedValues.append($0) }
            )
            .store(in: &cancellables)
        
        wait(for: [expectation], timeout: 1.0)
        XCTAssertEqual(receivedValues, [2, 4])
    }
    
    func testViewModelWithCombine() async {
        class SearchViewModel: ObservableObject {
            @Published var searchText = ""
            @Published var results: [String] = []
            private var cancellables = Set<AnyCancellable>()
            
            init() {
                $searchText
                    .debounce(for: .milliseconds(300), scheduler: RunLoop.main)
                    .removeDuplicates()
                    .map { query -> [String] in
                        guard !query.isEmpty else { return [] }
                        return ["Result for: \(query)"]
                    }
                    .assign(to: &$results)
            }
        }
        
        let viewModel = SearchViewModel()
        let expectation = expectation(description: "Results updated")
        
        viewModel.$results
            .dropFirst()
            .sink { results in
                if !results.isEmpty {
                    expectation.fulfill()
                }
            }
            .store(in: &cancellables)
        
        viewModel.searchText = "Swift"
        
        await fulfillment(of: [expectation], timeout: 1.0)
        XCTAssertFalse(viewModel.results.isEmpty)
    }
    
    func testErrorHandling() {
        let expectation = expectation(description: "Error received")
        
        enum TestError: Error {
            case networkError
        }
        
        let subject = PassthroughSubject<Int, TestError>()
        var receivedError: TestError?
        
        subject
            .sink(
                receiveCompletion: { completion in
                    if case .failure(let error) = completion {
                        receivedError = error
                        expectation.fulfill()
                    }
                },
                receiveValue: { _ in }
            )
            .store(in: &cancellables)
        
        subject.send(completion: .failure(.networkError))
        
        wait(for: [expectation], timeout: 1.0)
        XCTAssertEqual(receivedError, .networkError)
    }
    
    // ใช้ collect() เพื่อรับค่าทั้งหมด
    func testCollectAllValues() {
        let expectation = expectation(description: "All values collected")
        var allValues: [Int] = []
        
        (1...5).publisher
            .collect()
            .sink(
                receiveCompletion: { _ in expectation.fulfill() },
                receiveValue: { allValues = $0 }
            )
            .store(in: &cancellables)
        
        wait(for: [expectation], timeout: 1.0)
        XCTAssertEqual(allValues, [1, 2, 3, 4, 5])
    }
}
```

---

## 9. Swift Concurrency Testing (Async/Await)

### 9.1 Testing Async Functions

```swift
class AsyncConcurrencyTests: XCTestCase {
    
    // ใช้ async throws ใน test function
    func testAsyncOperation() async throws {
        let result = try await performAsyncOperation()
        XCTAssertEqual(result, 42)
    }
    
    private func performAsyncOperation() async throws -> Int {
        try await Task.sleep(nanoseconds: 100_000_000) // 0.1 วินาที
        return 42
    }
    
    // Testing Task cancellation
    func testTaskCancellation() async {
        let task = Task {
            do {
                try await Task.sleep(nanoseconds: 10_000_000_000) // 10 วินาที
                return "Completed"
            } catch {
                return "Cancelled"
            }
        }
        
        task.cancel()
        
        let result = await task.value
        XCTAssertEqual(result, "Cancelled")
    }
    
    // Testing async sequence
    func testAsyncSequence() async throws {
        var received: [Int] = []
        
        for try await value in makeAsyncSequence() {
            received.append(value)
        }
        
        XCTAssertEqual(received, [1, 2, 3, 4, 5])
    }
    
    private func makeAsyncSequence() -> AsyncStream<Int> {
        AsyncStream { continuation in
            Task {
                for i in 1...5 {
                    continuation.yield(i)
                    try await Task.sleep(nanoseconds: 10_000_000)
                }
                continuation.finish()
            }
        }
    }
    
    // Testing structured concurrency
    func testStructuredConcurrency() async throws {
        async let first = fetchValue(1)
        async let second = fetchValue(2)
        
        let results = try await [first, second]
        XCTAssertEqual(results.sorted(), [1, 2])
    }
    
    private func fetchValue(_ value: Int) async throws -> Int {
        try await Task.sleep(nanoseconds: 50_000_000)
        return value
    }
    
    // Testing withTaskGroup
    func testTaskGroup() async throws {
        let results = try await withThrowingTaskGroup(of: Int.self) { group in
            for i in 1...5 {
                group.addTask {
                    try await self.fetchValue(i)
                }
            }
            
            var collected: [Int] = []
            for try await result in group {
                collected.append(result)
            }
            return collected
        }
        
        XCTAssertEqual(results.sorted(), [1, 2, 3, 4, 5])
    }
}
```

---

## 10. Actor Testing

### 10.1 Testing Swift Actors

```swift
// Actor สำหรับ thread-safe counter
actor Counter {
    private var value: Int = 0
    
    func increment() {
        value += 1
    }
    
    func decrement() {
        value -= 1
    }
    
    func getValue() -> Int {
        return value
    }
    
    func reset() {
        value = 0
    }
}

class ActorTests: XCTestCase {
    
    func testActorIncrement() async {
        let counter = Counter()
        
        await counter.increment()
        await counter.increment()
        await counter.increment()
        
        let value = await counter.getValue()
        XCTAssertEqual(value, 3)
    }
    
    func testActorThreadSafety() async {
        let counter = Counter()
        
        // Test concurrent access
        await withTaskGroup(of: Void.self) { group in
            for _ in 0..<1000 {
                group.addTask {
                    await counter.increment()
                }
            }
        }
        
        let value = await counter.getValue()
        XCTAssertEqual(value, 1000)
    }
    
    func testActorIsolation() async {
        let counter = Counter()
        
        // Actor methods ต้องถูกเรียก async
        await counter.increment()
        let value = await counter.getValue()
        
        XCTAssertEqual(value, 1)
    }
}

// Global Actor Testing
@globalActor
actor DatabaseActor {
    static let shared = DatabaseActor()
}

@DatabaseActor
class DatabaseManager {
    var records: [String] = []
    
    func addRecord(_ record: String) {
        records.append(record)
    }
    
    func getRecords() -> [String] {
        return records
    }
}

class GlobalActorTests: XCTestCase {
    func testGlobalActor() async {
        let manager = DatabaseManager()
        
        await manager.addRecord("Record 1")
        await manager.addRecord("Record 2")
        
        let records = await manager.getRecords()
        XCTAssertEqual(records.count, 2)
    }
}
```

---

## 11. UI Testing Best Practices

### 11.1 Basic UI Testing

```swift
import XCTest

class UITestingBestPractices: XCTestCase {
    
    var app: XCUIApplication!
    
    override func setUp() {
        super.setUp()
        continueAfterFailure = false
        
        app = XCUIApplication()
        app.launchArguments = ["--uitesting"]
        app.launchEnvironment = ["ENV": "test"]
        app.launch()
    }
    
    override func tearDown() {
        app.terminate()
        super.tearDown()
    }
    
    func testLoginFlow() {
        // Navigate to login screen
        app.buttons["loginButton"].tap()
        
        // Enter credentials
        let emailField = app.textFields["emailTextField"]
        emailField.tap()
        emailField.typeText("user@example.com")
        
        let passwordField = app.secureTextFields["passwordTextField"]
        passwordField.tap()
        passwordField.typeText("password123")
        
        // Submit
        app.buttons["submitButton"].tap()
        
        // Verify success
        XCTAssertTrue(app.staticTexts["Welcome"].exists)
    }
    
    func testNavigationFlow() {
        // ตรวจสอบว่า navigation ทำงานถูกต้อง
        XCTAssertTrue(app.navigationBars["Home"].exists)
        
        app.tabBars.buttons["Profile"].tap()
        XCTAssertTrue(app.navigationBars["Profile"].exists)
    }
    
    func testScrolling() {
        let table = app.tables.firstMatch
        
        // Scroll down
        table.swipeUp()
        
        // ตรวจสอบ cell ที่ควรปรากฏหลัง scroll
        XCTAssertTrue(app.cells["cell_10"].exists)
    }
}
```

---

## 12. Page Object Model สำหรับ UI Tests

Page Object Model (POM) เป็น design pattern ที่แยก UI interaction logic ออกจาก test logic

### 12.1 Base Page Object

```swift
import XCTest

// Base Page Protocol
protocol PageObject {
    var app: XCUIApplication { get }
    func waitForPageToLoad(timeout: TimeInterval) -> Bool
}

extension PageObject {
    func waitForElement(_ element: XCUIElement, timeout: TimeInterval = 5.0) -> Bool {
        return element.waitForExistence(timeout: timeout)
    }
    
    func tapIfExists(_ element: XCUIElement) {
        if element.exists {
            element.tap()
        }
    }
}

// Login Page Object
class LoginPage: PageObject {
    let app: XCUIApplication
    
    // Elements
    var emailTextField: XCUIElement { app.textFields["emailTextField"] }
    var passwordTextField: XCUIElement { app.secureTextFields["passwordTextField"] }
    var loginButton: XCUIElement { app.buttons["loginButton"] }
    var errorLabel: XCUIElement { app.staticTexts["errorLabel"] }
    var registerLink: XCUIElement { app.buttons["registerLink"] }
    
    init(app: XCUIApplication) {
        self.app = app
    }
    
    func waitForPageToLoad(timeout: TimeInterval = 5.0) -> Bool {
        return loginButton.waitForExistence(timeout: timeout)
    }
    
    @discardableResult
    func enterEmail(_ email: String) -> LoginPage {
        emailTextField.tap()
        emailTextField.clearAndTypeText(email)
        return self
    }
    
    @discardableResult
    func enterPassword(_ password: String) -> LoginPage {
        passwordTextField.tap()
        passwordTextField.clearAndTypeText(password)
        return self
    }
    
    func tapLoginButton() -> HomePage {
        loginButton.tap()
        return HomePage(app: app)
    }
    
    func tapLoginButtonExpectingError() -> LoginPage {
        loginButton.tap()
        return self
    }
    
    func tapRegisterLink() -> RegisterPage {
        registerLink.tap()
        return RegisterPage(app: app)
    }
}

// Home Page Object
class HomePage: PageObject {
    let app: XCUIApplication
    
    var welcomeLabel: XCUIElement { app.staticTexts["welcomeLabel"] }
    var profileButton: XCUIElement { app.tabBars.buttons["Profile"] }
    var logoutButton: XCUIElement { app.buttons["logoutButton"] }
    
    init(app: XCUIApplication) {
        self.app = app
    }
    
    func waitForPageToLoad(timeout: TimeInterval = 5.0) -> Bool {
        return welcomeLabel.waitForExistence(timeout: timeout)
    }
    
    func tapProfile() -> ProfilePage {
        profileButton.tap()
        return ProfilePage(app: app)
    }
    
    func logout() -> LoginPage {
        logoutButton.tap()
        return LoginPage(app: app)
    }
}

// Register Page Object
class RegisterPage: PageObject {
    let app: XCUIApplication
    
    var nameTextField: XCUIElement { app.textFields["nameTextField"] }
    var emailTextField: XCUIElement { app.textFields["registerEmailTextField"] }
    var passwordTextField: XCUIElement { app.secureTextFields["registerPasswordTextField"] }
    var registerButton: XCUIElement { app.buttons["registerButton"] }
    
    init(app: XCUIApplication) {
        self.app = app
    }
    
    func waitForPageToLoad(timeout: TimeInterval = 5.0) -> Bool {
        return registerButton.waitForExistence(timeout: timeout)
    }
}

// Profile Page Object
class ProfilePage: PageObject {
    let app: XCUIApplication
    
    var nameLabel: XCUIElement { app.staticTexts["nameLabel"] }
    var editButton: XCUIElement { app.buttons["editButton"] }
    
    init(app: XCUIApplication) {
        self.app = app
    }
    
    func waitForPageToLoad(timeout: TimeInterval = 5.0) -> Bool {
        return nameLabel.waitForExistence(timeout: timeout)
    }
}

// Extension สำหรับ clear text
extension XCUIElement {
    func clearAndTypeText(_ text: String) {
        guard let currentValue = value as? String, !currentValue.isEmpty else {
            typeText(text)
            return
        }
        
        // Select all and delete
        tap()
        let deleteString = String(repeating: XCUIKeyboardKey.delete.rawValue, 
                                   count: currentValue.count)
        typeText(deleteString)
        typeText(text)
    }
}

// Tests ที่ใช้ Page Object Model
class LoginFlowTests: XCTestCase {
    
    var app: XCUIApplication!
    
    override func setUp() {
        super.setUp()
        continueAfterFailure = false
        app = XCUIApplication()
        app.launchArguments = ["--uitesting", "--reset-state"]
        app.launch()
    }
    
    func testSuccessfulLogin() {
        let loginPage = LoginPage(app: app)
        XCTAssertTrue(loginPage.waitForPageToLoad())
        
        let homePage = loginPage
            .enterEmail("user@example.com")
            .enterPassword("password123")
            .tapLoginButton()
        
        XCTAssertTrue(homePage.waitForPageToLoad())
        XCTAssertTrue(homePage.welcomeLabel.exists)
    }
    
    func testFailedLogin() {
        let loginPage = LoginPage(app: app)
        XCTAssertTrue(loginPage.waitForPageToLoad())
        
        loginPage
            .enterEmail("wrong@example.com")
            .enterPassword("wrongpassword")
            .tapLoginButtonExpectingError()
        
        XCTAssertTrue(loginPage.errorLabel.exists)
        XCTAssertTrue(loginPage.errorLabel.label.contains("Invalid"))
    }
    
    func testNavigateToRegister() {
        let loginPage = LoginPage(app: app)
        let registerPage = loginPage.tapRegisterLink()
        
        XCTAssertTrue(registerPage.waitForPageToLoad())
    }
}
```

---

## 13. Accessibility Identifier Strategy

### 13.1 Centralized Accessibility Identifiers

```swift
// AccessibilityIdentifier.swift - Centralized identifiers
enum AccessibilityIdentifier {
    enum Login {
        static let emailTextField = "login.emailTextField"
        static let passwordTextField = "login.passwordTextField"
        static let loginButton = "login.loginButton"
        static let errorLabel = "login.errorLabel"
        static let registerLink = "login.registerLink"
    }
    
    enum Home {
        static let welcomeLabel = "home.welcomeLabel"
        static let profileTab = "home.profileTab"
        static let logoutButton = "home.logoutButton"
    }
    
    enum UserList {
        static let tableView = "userList.tableView"
        static func cell(at index: Int) -> String {
            return "userList.cell.\(index)"
        }
        static let loadingIndicator = "userList.loadingIndicator"
        static let errorView = "userList.errorView"
    }
}

// การใช้ใน SwiftUI
struct LoginView: View {
    @State private var email = ""
    @State private var password = ""
    
    var body: some View {
        VStack {
            TextField("Email", text: $email)
                .accessibilityIdentifier(AccessibilityIdentifier.Login.emailTextField)
            
            SecureField("Password", text: $password)
                .accessibilityIdentifier(AccessibilityIdentifier.Login.passwordTextField)
            
            Button("Login") { }
                .accessibilityIdentifier(AccessibilityIdentifier.Login.loginButton)
        }
    }
}

// การใช้ใน UIKit
class LoginViewController: UIViewController {
    let emailTextField: UITextField = {
        let textField = UITextField()
        textField.accessibilityIdentifier = AccessibilityIdentifier.Login.emailTextField
        return textField
    }()
    
    let loginButton: UIButton = {
        let button = UIButton()
        button.accessibilityIdentifier = AccessibilityIdentifier.Login.loginButton
        return button
    }()
}

// Updated Page Objects ที่ใช้ Centralized Identifiers
class UpdatedLoginPage: PageObject {
    let app: XCUIApplication
    
    var emailTextField: XCUIElement { 
        app.textFields[AccessibilityIdentifier.Login.emailTextField] 
    }
    var passwordTextField: XCUIElement { 
        app.secureTextFields[AccessibilityIdentifier.Login.passwordTextField] 
    }
    var loginButton: XCUIElement { 
        app.buttons[AccessibilityIdentifier.Login.loginButton] 
    }
    
    init(app: XCUIApplication) {
        self.app = app
    }
    
    func waitForPageToLoad(timeout: TimeInterval = 5.0) -> Bool {
        return loginButton.waitForExistence(timeout: timeout)
    }
}
```

---

## 14. Test Parallelization

### 14.1 การเปิดใช้งาน Parallel Testing

```swift
// ใน Xcode: Test Plan > Options > Execution > Parallel
// หรือใน command line:
// xcodebuild test -scheme MyApp -parallelizeTargets -parallel-testing-enabled YES

// Test ที่ parallelizable ต้องไม่มี shared state
class ParallelizableTests: XCTestCase {
    
    // ดี: ใช้ local state เท่านั้น
    func testCalculation() {
        let result = 2 + 2
        XCTAssertEqual(result, 4)
    }
    
    // ดี: สร้าง instance ใหม่แต่ละ test
    func testUserCreation() {
        let service = UserService(logger: DummyLogger())
        XCTAssertTrue(service.createUser(name: "Test"))
    }
}

// สำหรับ tests ที่ต้องการ serial execution
class SerialTests: XCTestCase {
    
    // บน class นี้จะ run serially
    override class var defaultTestSuite: XCTestSuite {
        return XCTestSuite(name: "SerialTests")
    }
}
```

### 14.2 Test Sharding

```bash
# แบ่ง tests ออกเป็น shards สำหรับ CI
# Shard 1 of 3:
xcodebuild test -scheme MyApp \
  -only-testing:MyAppTests \
  -parallel-testing-enabled YES \
  -parallel-testing-worker-count 4 \
  -test-iterations 1

# ใน GitHub Actions:
# jobs:
#   test:
#     strategy:
#       matrix:
#         shard: [1, 2, 3]
#     steps:
#       - run: xcodebuild test -scheme MyApp -test-shard ${{ matrix.shard }}/3
```

---

## 15. Flaky Tests Detection and Fixing

### 15.1 Common Causes of Flaky Tests

```swift
// ปัญหา 1: Race conditions
class FlakyTestExample: XCTestCase {
    
    // BAD: Race condition
    func testFlakyRaceCondition() {
        var result = 0
        
        DispatchQueue.global().async {
            result = 42
        }
        
        // อาจ fail เพราะ async operation อาจยังไม่เสร็จ
        XCTAssertEqual(result, 42)
    }
    
    // GOOD: ใช้ expectation
    func testFixedRaceCondition() {
        let expectation = expectation(description: "Value set")
        var result = 0
        
        DispatchQueue.global().async {
            result = 42
            expectation.fulfill()
        }
        
        waitForExpectations(timeout: 1.0)
        XCTAssertEqual(result, 42)
    }
}

// ปัญหา 2: Order dependency
class OrderDependentTest: XCTestCase {
    static var sharedState = 0
    
    // BAD: พึ่ง shared state
    func testFirst() {
        OrderDependentTest.sharedState = 10
    }
    
    func testSecond() {
        // อาจ fail ถ้า testFirst ไม่ run ก่อน
        XCTAssertEqual(OrderDependentTest.sharedState, 10)
    }
    
    // GOOD: ไม่พึ่ง shared state
    func testIndependent() {
        var localState = 0
        localState = 10
        XCTAssertEqual(localState, 10)
    }
}

// ปัญหา 3: Time-dependent tests
class TimeDependentTest: XCTestCase {
    
    // BAD: พึ่ง actual time
    func testTimeout_Flaky() {
        let start = Date()
        // ... some operation ...
        let elapsed = Date().timeIntervalSince(start)
        XCTAssertLessThan(elapsed, 0.1) // อาจ fail ถ้า CI ช้า
    }
    
    // GOOD: ใช้ mock clock
    protocol Clock {
        func now() -> Date
        func sleep(for duration: TimeInterval) async throws
    }
    
    class MockClock: Clock {
        var currentTime = Date()
        
        func now() -> Date { currentTime }
        
        func sleep(for duration: TimeInterval) async throws {
            currentTime.addTimeInterval(duration)
        }
    }
}
```

### 15.2 Detecting Flaky Tests

```bash
# Run tests หลายๆ ครั้งเพื่อหา flaky tests
# xcodebuild test -scheme MyApp -test-iterations 10 -retry-tests-on-failure

# ดู test results ใน Xcode:
# Window > Organizer > Tests

# Script สำหรับหา flaky tests:
#!/bin/bash
FAILURES=0
for i in {1..10}; do
    xcodebuild test -scheme MyApp -quiet
    if [ $? -ne 0 ]; then
        FAILURES=$((FAILURES + 1))
    fi
done
echo "Failed $FAILURES out of 10 runs"
```

---

## 16. Code Coverage Analysis

### 16.1 การวัด Code Coverage

```swift
// เปิด Code Coverage ใน Xcode:
// Product > Scheme > Edit Scheme > Test > Code Coverage > Gather coverage

// ดู coverage ใน command line:
// xcodebuild test -scheme MyApp -enableCodeCoverage YES

// Coverage report ด้วย xcov:
// xcov --scheme MyApp --output_directory coverage_report

// ตั้งค่า minimum coverage threshold:
class CoverageCheckScript {
    // ในสคริปต์ CI:
    // COVERAGE=$(xcrun xccov view --report TestResults.xcresult | grep "MyApp.swift" | awk '{print $4}')
    // MINIMUM=80
    // if [ "$COVERAGE" -lt "$MINIMUM" ]; then echo "Coverage too low"; exit 1; fi
}
```

---

## 17. Mutation Testing Concepts

Mutation testing วัดคุณภาพของ test suite โดยการ "mutate" code และตรวจสอบว่า tests สามารถจับได้

```swift
// Original code:
func isAdult(age: Int) -> Bool {
    return age >= 18
}

// Mutations ที่ mutation testing จะสร้าง:
// Mutation 1: เปลี่ยน >= เป็น >
func isAdult_mutation1(age: Int) -> Bool {
    return age > 18  // ควรจะ fail test
}

// Mutation 2: เปลี่ยน 18 เป็น 19
func isAdult_mutation2(age: Int) -> Bool {
    return age >= 19  // ควรจะ fail test
}

// Tests ที่ดีต้องจับ mutations ได้ทั้งหมด
class AdultCheckTests: XCTestCase {
    func testExactlyAge18IsAdult() {
        XCTAssertTrue(isAdult(age: 18))  // จับ mutation 1 และ 2 ได้
    }
    
    func testAge17IsNotAdult() {
        XCTAssertFalse(isAdult(age: 17))  // จับ mutations เพิ่มเติม
    }
    
    func testAge0IsNotAdult() {
        XCTAssertFalse(isAdult(age: 0))
    }
}

// Tools: muter (https://github.com/muter-mutation-testing/muter)
// muter run --filesToMutate Sources/
```

---

## 18. Swift Testing Framework Deep Dive

Swift Testing เป็น framework ใหม่ที่แนะนำใน Swift 5.10 / Xcode 16

### 18.1 Basic Swift Testing

```swift
import Testing

// @Test แทน func test...()
@Test
func basicAddition() {
    let result = 2 + 2
    #expect(result == 4)
}

// #expect แทน XCTAssert
@Test
func stringManipulation() {
    let greeting = "Hello, World!"
    
    #expect(greeting.hasPrefix("Hello"))
    #expect(greeting.count == 13)
    #expect(!greeting.isEmpty)
}

// #require แทน XCTUnwrap (throw ถ้า nil)
@Test
func unwrappingOptional() throws {
    let optional: String? = "test"
    let value = try #require(optional)
    #expect(value == "test")
}

// Test ที่ throws
@Test
func throwingTest() throws {
    func divide(_ a: Int, by b: Int) throws -> Int {
        guard b != 0 else {
            throw DivisionError.divideByZero
        }
        return a / b
    }
    
    let result = try divide(10, by: 2)
    #expect(result == 5)
    
    #expect(throws: DivisionError.divideByZero) {
        try divide(10, by: 0)
    }
}

enum DivisionError: Error, Equatable {
    case divideByZero
}
```

### 18.2 @Suite

```swift
// @Suite จัดกลุ่ม tests
@Suite("User Management Tests")
struct UserManagementTests {
    
    let repository = FakeUserRepository()
    
    @Test("Creating a new user")
    func createUser() throws {
        let user = User(id: 1, name: "Alice", email: "alice@example.com")
        try repository.save(user: user)
        
        let found = repository.findById(1)
        #expect(found?.name == "Alice")
    }
    
    @Test("Finding non-existent user returns nil")
    func findNonExistentUser() {
        let found = repository.findById(999)
        #expect(found == nil)
    }
    
    @Suite("Deletion Tests")
    struct DeletionTests {
        let repository = FakeUserRepository()
        
        @Test
        func deleteExistingUser() throws {
            let user = User(id: 1, name: "Bob", email: "bob@example.com")
            try repository.save(user: user)
            try repository.delete(id: 1)
            
            #expect(repository.findById(1) == nil)
        }
    }
}
```

### 18.3 Parameterized Tests

```swift
import Testing

// Parameterized tests กับ @Test arguments
@Test("Multiplication table", arguments: [
    (2, 3, 6),
    (4, 5, 20),
    (7, 8, 56),
    (0, 100, 0)
])
func testMultiplication(a: Int, b: Int, expected: Int) {
    #expect(a * b == expected)
}

// Parameterized กับ custom types
enum Platform {
    case iOS, macOS, watchOS
}

@Test("Platform support", arguments: Platform.allCases)
func testPlatformSupport(platform: Platform) {
    // Test สำหรับแต่ละ platform
    #expect(isSupported(platform))
}

extension Platform: CaseIterable {}

func isSupported(_ platform: Platform) -> Bool {
    return true // Simplified
}

// Parameterized กับ zip
@Test("String formatting", arguments: zip(
    ["hello", "world", "swift"],
    ["Hello", "World", "Swift"]
))
func testStringCapitalization(input: String, expected: String) {
    #expect(input.capitalized == expected)
}
```

### 18.4 Tags และ Conditions

```swift
import Testing

// Tags สำหรับจัดกลุ่ม tests
extension Tag {
    @Tag static var networking: Self
    @Tag static var database: Self
    @Tag static var ui: Self
    @Tag static var slow: Self
}

@Suite
struct TaggedTests {
    
    @Test(.tags(.networking))
    func testNetworkRequest() async throws {
        // Network test
        #expect(true)
    }
    
    @Test(.tags(.database, .slow))
    func testDatabaseOperation() throws {
        // Database test
        #expect(true)
    }
}

// Conditions
@Suite
struct ConditionalTests {
    
    // เฉพาะ macOS
    @Test(.enabled(if: ProcessInfo.processInfo.operatingSystemVersion.majorVersion >= 14))
    func testModernFeature() {
        #expect(true)
    }
    
    // Skip test
    @Test("Temporarily disabled", .disabled("Bug #123 not fixed yet"))
    func testBrokenFeature() {
        // จะถูก skip
    }
}
```

---

## 19. TDD Workflow กับ Examples

### 19.1 Red-Green-Refactor Cycle

```swift
// Step 1: RED - เขียน test ที่ fail ก่อน
// Step 2: GREEN - เขียน code ให้ test ผ่าน (minimal implementation)
// Step 3: REFACTOR - ปรับปรุง code โดยไม่ให้ tests fail

// Example: สร้าง ShoppingCart ด้วย TDD

// RED: Test fail
import Testing

@Suite("Shopping Cart TDD")
struct ShoppingCartTests {
    
    @Test("Empty cart has zero total")
    func emptyCartTotal() {
        let cart = ShoppingCart()
        #expect(cart.total == 0.0)
    }
}

// GREEN: Minimal implementation
struct ShoppingCart {
    var total: Double = 0.0
}

// RED: Test fail
// @Test("Adding item increases total")
// func addingItemIncreasesTotal() {
//     var cart = ShoppingCart()
//     cart.addItem(name: "Apple", price: 1.50)
//     #expect(cart.total == 1.50)
// }

// GREEN: Implementation
struct CartItem {
    let name: String
    let price: Double
    let quantity: Int
}

struct ShoppingCartV2 {
    private(set) var items: [CartItem] = []
    
    var total: Double {
        items.reduce(0) { $0 + ($1.price * Double($1.quantity)) }
    }
    
    mutating func addItem(name: String, price: Double, quantity: Int = 1) {
        if let index = items.firstIndex(where: { $0.name == name }) {
            let existing = items[index]
            items[index] = CartItem(name: name, price: price, 
                                    quantity: existing.quantity + quantity)
        } else {
            items.append(CartItem(name: name, price: price, quantity: quantity))
        }
    }
    
    mutating func removeItem(name: String) {
        items.removeAll { $0.name == name }
    }
    
    mutating func clear() {
        items.removeAll()
    }
}

@Suite("Shopping Cart V2 TDD")
struct ShoppingCartV2Tests {
    
    @Test("Empty cart total is zero")
    func emptyCartTotal() {
        let cart = ShoppingCartV2()
        #expect(cart.total == 0.0)
    }
    
    @Test("Adding single item")
    func addSingleItem() {
        var cart = ShoppingCartV2()
        cart.addItem(name: "Apple", price: 1.50)
        #expect(cart.total == 1.50)
    }
    
    @Test("Adding multiple items")
    func addMultipleItems() {
        var cart = ShoppingCartV2()
        cart.addItem(name: "Apple", price: 1.50)
        cart.addItem(name: "Banana", price: 0.75)
        #expect(cart.total == 2.25, "Total should be sum of all items")
    }
    
    @Test("Adding duplicate item increases quantity")
    func addDuplicateItem() {
        var cart = ShoppingCartV2()
        cart.addItem(name: "Apple", price: 1.50)
        cart.addItem(name: "Apple", price: 1.50)
        #expect(cart.items.count == 1, "Should have only one unique item")
        #expect(cart.total == 3.00, "Total should be doubled")
    }
    
    @Test("Removing item", arguments: ["Apple", "Banana"])
    func removeItem(itemName: String) {
        var cart = ShoppingCartV2()
        cart.addItem(name: "Apple", price: 1.50)
        cart.addItem(name: "Banana", price: 0.75)
        
        cart.removeItem(name: itemName)
        
        #expect(!cart.items.contains { $0.name == itemName })
    }
    
    @Test("Clearing cart")
    func clearCart() {
        var cart = ShoppingCartV2()
        cart.addItem(name: "Apple", price: 1.50)
        cart.addItem(name: "Banana", price: 0.75)
        
        cart.clear()
        
        #expect(cart.items.isEmpty)
        #expect(cart.total == 0.0)
    }
}
```

---

## 20. BDD กับ Quick/Nimble

### 20.1 Quick/Nimble Setup และ Usage

```swift
// Package.swift dependencies:
// .package(url: "https://github.com/Quick/Quick.git", from: "7.0.0"),
// .package(url: "https://github.com/Quick/Nimble.git", from: "13.0.0")

import Quick
import Nimble
import Foundation

class UserServiceSpec: QuickSpec {
    override class func spec() {
        describe("UserService") {
            var service: UserService!
            var mockLogger: MockLogger!
            
            beforeEach {
                mockLogger = MockLogger()
                service = UserService(logger: mockLogger)
            }
            
            context("when creating a user with valid name") {
                it("returns true") {
                    expect(service.createUser(name: "Alice")).to(beTrue())
                }
                
                it("logs the creation") {
                    _ = service.createUser(name: "Alice")
                    expect(mockLogger.loggedMessages).to(containElementSatisfying { 
                        $0.contains("Alice") 
                    })
                }
            }
            
            context("when creating a user with empty name") {
                it("returns false") {
                    expect(service.createUser(name: "")).to(beFalse())
                }
                
                it("does not log anything") {
                    _ = service.createUser(name: "")
                    expect(mockLogger.loggedMessages).to(beEmpty())
                }
            }
        }
    }
}

// Mock สำหรับ Quick/Nimble tests
class MockLogger: Logger {
    var loggedMessages: [String] = []
    
    func log(_ message: String) {
        loggedMessages.append(message)
    }
}

// Nimble Matchers
class NimbleMatchersExample: QuickSpec {
    override class func spec() {
        describe("Nimble matchers") {
            it("equality") {
                expect(2 + 2).to(equal(4))
                expect(2 + 2).toNot(equal(5))
            }
            
            it("comparison") {
                expect(10).to(beGreaterThan(5))
                expect(1).to(beLessThan(2))
                expect(5).to(beGreaterThanOrEqualTo(5))
            }
            
            it("collection") {
                let array = [1, 2, 3]
                expect(array).to(contain(2))
                expect(array).to(haveCount(3))
                expect(array).toNot(beEmpty())
            }
            
            it("string") {
                let str = "Hello, World!"
                expect(str).to(beginWith("Hello"))
                expect(str).to(endWith("!"))
                expect(str).to(contain("World"))
            }
            
            it("async") {
                waitUntil(timeout: .seconds(2)) { done in
                    DispatchQueue.global().asyncAfter(deadline: .now() + 0.5) {
                        done()
                    }
                }
            }
        }
    }
}
```

---

## 21. Practical Exercise: Building a Fully Tested MVVM Feature

### 21.1 Feature: Todo List กับ MVVM

```swift
// MARK: - Models

struct Todo: Identifiable, Equatable, Codable {
    let id: UUID
    var title: String
    var isCompleted: Bool
    var createdAt: Date
    
    init(id: UUID = UUID(), title: String, isCompleted: Bool = false, 
         createdAt: Date = Date()) {
        self.id = id
        self.title = title
        self.isCompleted = isCompleted
        self.createdAt = createdAt
    }
}

// MARK: - Repository Protocol

protocol TodoRepository {
    func fetchAll() async throws -> [Todo]
    func save(_ todo: Todo) async throws
    func update(_ todo: Todo) async throws
    func delete(id: UUID) async throws
}

// MARK: - ViewModel

@MainActor
class TodoListViewModel: ObservableObject {
    @Published var todos: [Todo] = []
    @Published var isLoading = false
    @Published var errorMessage: String?
    @Published var filter: Filter = .all
    
    enum Filter: String, CaseIterable {
        case all = "All"
        case active = "Active"
        case completed = "Completed"
    }
    
    private let repository: TodoRepository
    
    init(repository: TodoRepository) {
        self.repository = repository
    }
    
    var filteredTodos: [Todo] {
        switch filter {
        case .all: return todos
        case .active: return todos.filter { !$0.isCompleted }
        case .completed: return todos.filter { $0.isCompleted }
        }
    }
    
    var completionCount: Int {
        todos.filter { $0.isCompleted }.count
    }
    
    func loadTodos() async {
        isLoading = true
        errorMessage = nil
        
        do {
            todos = try await repository.fetchAll()
        } catch {
            errorMessage = "Failed to load todos: \(error.localizedDescription)"
        }
        
        isLoading = false
    }
    
    func addTodo(title: String) async {
        guard !title.trimmingCharacters(in: .whitespaces).isEmpty else {
            errorMessage = "Todo title cannot be empty"
            return
        }
        
        let todo = Todo(title: title.trimmingCharacters(in: .whitespaces))
        
        do {
            try await repository.save(todo)
            todos.append(todo)
        } catch {
            errorMessage = "Failed to save todo: \(error.localizedDescription)"
        }
    }
    
    func toggleTodo(_ todo: Todo) async {
        var updated = todo
        updated.isCompleted.toggle()
        
        do {
            try await repository.update(updated)
            if let index = todos.firstIndex(where: { $0.id == todo.id }) {
                todos[index] = updated
            }
        } catch {
            errorMessage = "Failed to update todo"
        }
    }
    
    func deleteTodo(_ todo: Todo) async {
        do {
            try await repository.delete(id: todo.id)
            todos.removeAll { $0.id == todo.id }
        } catch {
            errorMessage = "Failed to delete todo"
        }
    }
}

// MARK: - Fake Repository

class FakeTodoRepository: TodoRepository {
    var todos: [Todo] = []
    var shouldFail = false
    
    func fetchAll() async throws -> [Todo] {
        if shouldFail { throw NSError(domain: "Test", code: 500) }
        return todos
    }
    
    func save(_ todo: Todo) async throws {
        if shouldFail { throw NSError(domain: "Test", code: 500) }
        todos.append(todo)
    }
    
    func update(_ todo: Todo) async throws {
        if shouldFail { throw NSError(domain: "Test", code: 500) }
        if let index = todos.firstIndex(where: { $0.id == todo.id }) {
            todos[index] = todo
        }
    }
    
    func delete(id: UUID) async throws {
        if shouldFail { throw NSError(domain: "Test", code: 500) }
        todos.removeAll { $0.id == id }
    }
}

// MARK: - Tests

import Testing

@Suite("TodoListViewModel Tests")
@MainActor
struct TodoListViewModelTests {
    
    var repository: FakeTodoRepository
    var viewModel: TodoListViewModel
    
    init() {
        repository = FakeTodoRepository()
        viewModel = TodoListViewModel(repository: repository)
    }
    
    @Test("Initial state")
    func initialState() {
        #expect(viewModel.todos.isEmpty)
        #expect(!viewModel.isLoading)
        #expect(viewModel.errorMessage == nil)
        #expect(viewModel.filter == .all)
    }
    
    @Test("Load todos successfully")
    func loadTodosSuccess() async {
        repository.todos = [
            Todo(title: "Todo 1"),
            Todo(title: "Todo 2"),
            Todo(title: "Todo 3")
        ]
        
        await viewModel.loadTodos()
        
        #expect(viewModel.todos.count == 3)
        #expect(!viewModel.isLoading)
        #expect(viewModel.errorMessage == nil)
    }
    
    @Test("Load todos failure shows error")
    func loadTodosFailure() async {
        repository.shouldFail = true
        
        await viewModel.loadTodos()
        
        #expect(viewModel.todos.isEmpty)
        #expect(viewModel.errorMessage != nil)
    }
    
    @Test("Add valid todo")
    func addValidTodo() async {
        await viewModel.addTodo(title: "New Todo")
        
        #expect(viewModel.todos.count == 1)
        #expect(viewModel.todos.first?.title == "New Todo")
        #expect(repository.todos.count == 1)
    }
    
    @Test("Add empty todo shows error")
    func addEmptyTodo() async {
        await viewModel.addTodo(title: "   ")
        
        #expect(viewModel.todos.isEmpty)
        #expect(viewModel.errorMessage != nil)
    }
    
    @Test("Toggle todo completion")
    func toggleTodo() async {
        repository.todos = [Todo(title: "Test Todo")]
        await viewModel.loadTodos()
        
        let todo = viewModel.todos[0]
        #expect(!todo.isCompleted)
        
        await viewModel.toggleTodo(todo)
        
        #expect(viewModel.todos[0].isCompleted)
    }
    
    @Test("Delete todo")
    func deleteTodo() async {
        repository.todos = [Todo(title: "To Delete")]
        await viewModel.loadTodos()
        
        let todo = viewModel.todos[0]
        await viewModel.deleteTodo(todo)
        
        #expect(viewModel.todos.isEmpty)
        #expect(repository.todos.isEmpty)
    }
    
    @Test("Filter todos", arguments: [
        (TodoListViewModel.Filter.all, 3),
        (TodoListViewModel.Filter.active, 2),
        (TodoListViewModel.Filter.completed, 1)
    ])
    func filterTodos(filter: TodoListViewModel.Filter, expectedCount: Int) async {
        repository.todos = [
            Todo(title: "Active 1"),
            Todo(title: "Active 2"),
            Todo(title: "Completed", isCompleted: true)
        ]
        await viewModel.loadTodos()
        
        viewModel.filter = filter
        
        #expect(viewModel.filteredTodos.count == expectedCount)
    }
    
    @Test("Completion count")
    func completionCount() async {
        repository.todos = [
            Todo(title: "Active 1"),
            Todo(title: "Active 2"),
            Todo(title: "Completed 1", isCompleted: true),
            Todo(title: "Completed 2", isCompleted: true)
        ]
        await viewModel.loadTodos()
        
        #expect(viewModel.completionCount == 2)
    }
}
```

---

## 22. สรุป

ในบทนี้เราได้เรียนรู้เทคนิคการทดสอบขั้นสูงที่ครอบคลุม:

1. **Advanced XCTest** - Lifecycle, assertions, async testing, performance
2. **Test Doubles** - Dummy, Stub, Mock, Spy, Fake และความแตกต่าง
3. **Protocol-Based Mocking** - การออกแบบ protocols เพื่อ testability
4. **Property-Based Testing** - SwiftCheck สำหรับหา edge cases
5. **Snapshot Testing** - swift-snapshot-testing สำหรับ UI regression
6. **Network Testing** - MockURLProtocol สำหรับ mock HTTP requests
7. **Core Data Testing** - In-memory store สำหรับ database tests
8. **Combine Testing** - Testing publishers และ subscribers
9. **Swift Concurrency** - Testing async/await, Tasks, TaskGroups
10. **Actor Testing** - Thread-safe testing ด้วย actors
11. **UI Testing** - Best practices และ Page Object Model
12. **Accessibility Identifiers** - Centralized strategy
13. **Test Parallelization** - เพิ่มความเร็วของ test suite
14. **Flaky Tests** - Detection และ fixing
15. **Code Coverage** - การวัดและ threshold
16. **Mutation Testing** - วัดคุณภาพของ test suite
17. **Swift Testing Framework** - #expect, #require, @Suite, parameterized tests
18. **TDD Workflow** - Red-Green-Refactor cycle
19. **BDD กับ Quick/Nimble** - Behavior-driven development
20. **Fully Tested MVVM Feature** - Todo List ตัวอย่างสมบูรณ์

### Best Practices สรุป

```swift
// 1. Test พฤติกรรม ไม่ใช่ implementation
// 2. ใช้ ARRANGE-ACT-ASSERT pattern
// 3. ชื่อ test ควรอธิบาย what/when/then
// 4. Test หนึ่งควร verify สิ่งเดียว
// 5. Fast, Isolated, Repeatable, Self-verifying, Timely (FIRST)
// 6. ใช้ test doubles แทน dependencies จริง
// 7. หลีกเลี่ยง shared state ระหว่าง tests
// 8. เขียน test ก่อน code (TDD)
// 9. รักษา test code ให้อ่านง่ายเหมือน production code
// 10. ตั้ง coverage target ที่สมเหตุสมผล (80%+)
```

---

*จบบทที่ 79: Advanced Testing in Swift*
