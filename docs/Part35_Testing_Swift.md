# Part 35: การทดสอบใน Swift (Testing in Swift)

## บทนำ

การทดสอบซอฟต์แวร์เป็นส่วนสำคัญของการพัฒนาแอปพลิเคชันที่มีคุณภาพ ในบทนี้เราจะเรียนรู้เกี่ยวกับการเขียน test ใน Swift โดยใช้ XCTest framework และ Swift Testing framework ใหม่ที่มาพร้อมกับ Swift 6

---

## 35.1 ทำไมการทดสอบจึงสำคัญ (Why Testing Matters)

การทดสอบซอฟต์แวร์มีความสำคัญอย่างยิ่งในการพัฒนาซอฟต์แวร์ระดับมืออาชีพ เหตุผลหลักๆ มีดังนี้

### ประโยชน์ของการทดสอบ

1. **ตรวจจับ bug ได้เร็ว** - การทดสอบช่วยให้พบปัญหาตั้งแต่ขั้นตอนการพัฒนา ก่อนที่จะถึงมือผู้ใช้งาน
2. **ลด regression** - เมื่อเพิ่มฟีเจอร์ใหม่หรือแก้ไขโค้ด test จะช่วยให้แน่ใจว่าโค้ดเดิมยังทำงานได้ถูกต้อง
3. **เอกสารประกอบโค้ด** - Test cases ทำหน้าที่เป็น living documentation ที่แสดงว่าโค้ดควรทำงานอย่างไร
4. **ออกแบบโค้ดได้ดีขึ้น** - การเขียน test บังคับให้เราออกแบบโค้ดให้ testable ซึ่งมักหมายถึงโค้ดที่มี modular และ loosely coupled
5. **เพิ่มความมั่นใจ** - นักพัฒนาสามารถ refactor โค้ดได้อย่างมั่นใจเมื่อมี test ครอบคลุม
6. **ประหยัดเวลาในระยะยาว** - แม้การเขียน test ใช้เวลาเพิ่ม แต่ช่วยลดเวลาในการ debug และแก้ไข bug ในภายหลัง

### ประเภทของการทดสอบ

```
Testing Pyramid
        /\
       /  \
      / E2E\        End-to-End Tests (น้อย, ช้า, แพง)
     /------\
    /        \
   /Integration\    Integration Tests (ปานกลาง)
  /------------\
 /              \
/   Unit Tests   \   Unit Tests (มาก, เร็ว, ถูก)
/----------------\
```

- **Unit Tests** - ทดสอบหน่วยเล็กๆ ของโค้ด เช่น function หรือ method เดียว
- **Integration Tests** - ทดสอบการทำงานร่วมกันของหลายๆ component
- **UI Tests** - ทดสอบการทำงานจาก perspective ของผู้ใช้งาน
- **Performance Tests** - ทดสอบประสิทธิภาพของโค้ด

---

## 35.2 XCTest Framework

XCTest เป็น testing framework มาตรฐานของ Apple สำหรับการทดสอบ Swift และ Objective-C

### การตั้งค่า Test Target

เมื่อสร้าง Xcode project ใหม่ สามารถเลือก "Include Tests" เพื่อให้ Xcode สร้าง Test target ให้อัตโนมัติ หรือเพิ่มทีหลังโดย:
1. File → New → Target
2. เลือก "Unit Testing Bundle" หรือ "UI Testing Bundle"

### โครงสร้างพื้นฐาน

```swift
import XCTest
@testable import MyApp  // import module ที่ต้องการทดสอบ

class MyTests: XCTestCase {
    
    func testExample() {
        // เขียน test ที่นี่
        XCTAssertTrue(true)
    }
}
```

### การ import @testable

`@testable import` ช่วยให้สามารถเข้าถึง internal members ของ module ที่ต้องการทดสอบ ซึ่งปกติแล้วจะไม่สามารถเข้าถึงได้จากภายนอก

```swift
// ไม่ต้องใช้ @testable ถ้า members เป็น public
import Foundation

// ใช้ @testable เพื่อเข้าถึง internal members
@testable import MyApp

class CalculatorTests: XCTestCase {
    
    func testAddition() {
        let calculator = Calculator()
        // สามารถเรียกใช้ internal methods ได้
        let result = calculator.add(2, 3)
        XCTAssertEqual(result, 5)
    }
}
```

---

## 35.3 Unit Tests

Unit test คือการทดสอบหน่วยเล็กที่สุดของโค้ดแบบแยกส่วน

### ตัวอย่าง Unit Test พื้นฐาน

สมมติว่าเรามี Calculator class:

```swift
// Calculator.swift
struct Calculator {
    func add(_ a: Double, _ b: Double) -> Double {
        return a + b
    }
    
    func subtract(_ a: Double, _ b: Double) -> Double {
        return a - b
    }
    
    func multiply(_ a: Double, _ b: Double) -> Double {
        return a * b
    }
    
    func divide(_ a: Double, _ b: Double) throws -> Double {
        guard b != 0 else {
            throw CalculatorError.divisionByZero
        }
        return a / b
    }
}

enum CalculatorError: Error {
    case divisionByZero
}
```

และ test ที่สอดคล้อง:

```swift
// CalculatorTests.swift
import XCTest
@testable import MyApp

class CalculatorTests: XCTestCase {
    
    var calculator: Calculator!
    
    override func setUp() {
        super.setUp()
        calculator = Calculator()
    }
    
    override func tearDown() {
        calculator = nil
        super.tearDown()
    }
    
    // MARK: - Addition Tests
    
    func testAddPositiveNumbers() {
        let result = calculator.add(2, 3)
        XCTAssertEqual(result, 5, "การบวกเลขบวกควรได้ผลถูกต้อง")
    }
    
    func testAddNegativeNumbers() {
        let result = calculator.add(-2, -3)
        XCTAssertEqual(result, -5)
    }
    
    func testAddZero() {
        let result = calculator.add(5, 0)
        XCTAssertEqual(result, 5)
    }
    
    // MARK: - Division Tests
    
    func testDivideByZeroThrowsError() {
        XCTAssertThrowsError(try calculator.divide(10, 0)) { error in
            XCTAssertEqual(error as? CalculatorError, CalculatorError.divisionByZero)
        }
    }
    
    func testDivideNormalNumbers() throws {
        let result = try calculator.divide(10, 2)
        XCTAssertEqual(result, 5)
    }
}
```

---

## 35.4 XCTestCase

XCTestCase เป็น base class สำหรับการสร้าง test classes ทั้งหมด

### คุณสมบัติของ XCTestCase

```swift
class MyTests: XCTestCase {
    
    // Property ที่ใช้ได้ใน test
    // self.name - ชื่อของ test ปัจจุบัน
    // self.testRun - ข้อมูลเกี่ยวกับ test run ปัจจุบัน
    
    func testCheckTestName() {
        print("กำลังรัน test: \(self.name)")
    }
    
    // สามารถ skip test ได้
    func testSkippable() throws {
        let isFeatureEnabled = false
        try XCTSkipUnless(isFeatureEnabled, "Feature ยังไม่ได้เปิดใช้งาน")
        
        // โค้ดด้านล่างจะไม่รันถ้า feature ไม่ได้เปิดใช้งาน
        XCTAssertTrue(true)
    }
    
    // XCTSkipIf - skip ถ้าเงื่อนไขเป็นจริง
    func testConditionalSkip() throws {
        try XCTSkipIf(ProcessInfo.processInfo.environment["CI"] != nil,
                      "ข้าม test นี้บน CI environment")
        
        // โค้ดสำหรับ local testing เท่านั้น
    }
}
```

---

## 35.5 การตั้งชื่อ Test Methods (Test Methods Naming Convention)

การตั้งชื่อ test method ที่ดีช่วยให้เข้าใจว่า test ทดสอบอะไรและคาดหวังผลลัพธ์อะไร

### รูปแบบการตั้งชื่อที่นิยม

**รูปแบบ 1: test_[unitOfWork]_[stateUnderTest]_[expectedBehavior]**

```swift
class UserManagerTests: XCTestCase {
    
    // test_[method]_[condition]_[expectedResult]
    func test_login_withValidCredentials_returnsUser() {
        // ...
    }
    
    func test_login_withInvalidPassword_throwsAuthError() {
        // ...
    }
    
    func test_login_withEmptyUsername_returnsFalse() {
        // ...
    }
}
```

**รูปแบบ 2: testMethod_WhenCondition_ShouldResult**

```swift
class ShoppingCartTests: XCTestCase {
    
    func testAddItem_WhenCartIsEmpty_ShouldHaveOneItem() {
        // ...
    }
    
    func testRemoveItem_WhenItemExists_ShouldDecreaseCount() {
        // ...
    }
    
    func testCalculateTotal_WhenApplyingDiscount_ShouldReducePrice() {
        // ...
    }
}
```

**รูปแบบ 3: Given-When-Then (ตาม BDD)**

```swift
class BankAccountTests: XCTestCase {
    
    func testGivenSufficientBalance_WhenWithdrawing_ThenBalanceDecreases() {
        // Given
        var account = BankAccount(balance: 1000)
        
        // When
        try? account.withdraw(500)
        
        // Then
        XCTAssertEqual(account.balance, 500)
    }
}
```

### สิ่งที่ควรหลีกเลี่ยง

```swift
// ❌ ชื่อที่ไม่ดี
func test1() { }
func testSomething() { }
func testIt() { }

// ✅ ชื่อที่ดี
func testLoginWithValidCredentialsReturnsUser() { }
func testDivisionByZeroThrowsError() { }
func testEmptyArrayReturnsZeroCount() { }
```

---

## 35.6 setUp และ tearDown

`setUp` และ `tearDown` เป็น lifecycle methods ที่รันก่อนและหลัง test method แต่ละตัว

### การใช้งาน setUp และ tearDown

```swift
class DatabaseTests: XCTestCase {
    
    var database: Database!
    var testUser: User!
    
    // รันก่อน test method แต่ละตัว
    override func setUp() {
        super.setUp()
        
        // เตรียม test environment
        database = Database(inMemory: true)
        database.connect()
        
        testUser = User(id: "test-1", name: "ทดสอบ", email: "test@example.com")
        database.insert(testUser)
        
        print("✅ setUp เสร็จแล้ว")
    }
    
    // รันหลัง test method แต่ละตัว
    override func tearDown() {
        // ทำความสะอาด
        database.clearAll()
        database.disconnect()
        database = nil
        testUser = nil
        
        print("🧹 tearDown เสร็จแล้ว")
        
        super.tearDown()
    }
    
    func testFindUserById() {
        let found = database.findUser(byId: "test-1")
        XCTAssertNotNil(found)
        XCTAssertEqual(found?.name, "ทดสอบ")
    }
    
    func testDeleteUser() {
        database.deleteUser(byId: "test-1")
        let found = database.findUser(byId: "test-1")
        XCTAssertNil(found)
    }
}
```

### Class-level Setup

```swift
class ExpensiveSetupTests: XCTestCase {
    
    // รันครั้งเดียวก่อน test ทั้งหมดใน class
    override class func setUp() {
        super.setUp()
        print("🚀 Class setUp - รันครั้งเดียว")
        // ตั้งค่าที่ใช้เวลานาน เช่น database connection
    }
    
    // รันครั้งเดียวหลัง test ทั้งหมดใน class
    override class func tearDown() {
        print("🏁 Class tearDown - รันครั้งเดียว")
        super.tearDown()
    }
    
    // รันก่อน test แต่ละตัว
    override func setUp() {
        super.setUp()
        print("  → Instance setUp")
    }
    
    override func tearDown() {
        print("  ← Instance tearDown")
        super.tearDown()
    }
    
    func testFirst() {
        print("  🧪 testFirst")
        XCTAssertTrue(true)
    }
    
    func testSecond() {
        print("  🧪 testSecond")
        XCTAssertTrue(true)
    }
}

// Output:
// 🚀 Class setUp - รันครั้งเดียว
//   → Instance setUp
//   🧪 testFirst
//   ← Instance tearDown
//   → Instance setUp
//   🧪 testSecond
//   ← Instance tearDown
// 🏁 Class tearDown - รันครั้งเดียว
```

---

## 35.7 setUpWithError และ tearDownWithError

ใน Swift 5.2 ขึ้นไป มี `setUpWithError()` และ `tearDownWithError()` ที่รองรับการ throw error

```swift
class NetworkTests: XCTestCase {
    
    var networkClient: NetworkClient!
    var mockServer: MockServer!
    
    // setUpWithError สามารถ throw error ได้
    override func setUpWithError() throws {
        try super.setUpWithError()
        
        // ถ้า setup ล้มเหลว test จะ fail ทันที
        mockServer = try MockServer.start(port: 8080)
        networkClient = NetworkClient(baseURL: mockServer.url)
        
        // ตรวจสอบว่า server พร้อมใช้งาน
        let isReady = try mockServer.waitForReady(timeout: 5)
        XCTAssertTrue(isReady, "Mock server ไม่พร้อมใช้งาน")
    }
    
    // tearDownWithError ก็รองรับ error เช่นกัน
    override func tearDownWithError() throws {
        try mockServer?.stop()
        networkClient = nil
        mockServer = nil
        
        try super.tearDownWithError()
    }
    
    func testFetchData() async throws {
        let data = try await networkClient.fetch("/api/users")
        XCTAssertFalse(data.isEmpty)
    }
}
```

### ข้อแตกต่างระหว่าง setUp และ setUpWithError

```swift
class ComparisonExample: XCTestCase {
    
    // setUp แบบเก่า - ไม่รองรับ throws
    override func setUp() {
        super.setUp()
        // ถ้าเกิด error ต้องจัดการเองหรือ fatalError
        // ไม่สามารถ propagate error ได้
    }
    
    // setUpWithError แบบใหม่ - รองรับ throws
    override func setUpWithError() throws {
        try super.setUpWithError()
        // สามารถ throw error ได้
        // ถ้า throw แสดงว่า setup ล้มเหลว test จะ fail
        let config = try loadTestConfiguration()
        // ใช้ config ต่อได้
    }
    
    private func loadTestConfiguration() throws -> Configuration {
        // อาจ throw error ถ้าไม่พบ config file
        guard let path = Bundle(for: type(of: self)).path(forResource: "TestConfig", ofType: "json") else {
            throw TestError.missingConfiguration
        }
        let data = try Data(contentsOf: URL(fileURLWithPath: path))
        return try JSONDecoder().decode(Configuration.self, from: data)
    }
}

enum TestError: Error {
    case missingConfiguration
}
```

---

## 35.8 Assertions

XCTest มี assertion functions มากมายสำหรับการตรวจสอบผลลัพธ์

### XCTAssert (การ assert พื้นฐาน)

```swift
class AssertionExamples: XCTestCase {
    
    // XCTAssert - เหมือน XCTAssertTrue
    func testXCTAssert() {
        let value = 5
        XCTAssert(value > 0, "ค่าควรเป็นบวก")
        XCTAssert(value == 5)
    }
    
    // XCTAssertTrue - ตรวจสอบว่าเป็น true
    func testXCTAssertTrue() {
        let isLoggedIn = true
        XCTAssertTrue(isLoggedIn, "ควร logged in")
        
        let isEmpty = [].isEmpty
        XCTAssertTrue(isEmpty)
    }
    
    // XCTAssertFalse - ตรวจสอบว่าเป็น false
    func testXCTAssertFalse() {
        let isLoading = false
        XCTAssertFalse(isLoading, "ไม่ควรกำลัง loading")
        
        let hasErrors = [String]().isEmpty == false
        XCTAssertFalse(hasErrors)
    }
}
```

### XCTAssertEqual และ XCTAssertNotEqual

```swift
class EqualityTests: XCTestCase {
    
    // XCTAssertEqual - ตรวจสอบว่าเท่ากัน
    func testXCTAssertEqual() {
        // ตัวเลข
        XCTAssertEqual(2 + 3, 5)
        XCTAssertEqual(3.14, Double.pi, accuracy: 0.01)  // ใช้ accuracy สำหรับ floating point
        
        // String
        let name = "สมชาย"
        XCTAssertEqual(name, "สมชาย")
        
        // Array
        let numbers = [1, 2, 3]
        XCTAssertEqual(numbers, [1, 2, 3])
        
        // Optional
        let optValue: Int? = 42
        XCTAssertEqual(optValue, 42)
    }
    
    // XCTAssertNotEqual - ตรวจสอบว่าไม่เท่ากัน
    func testXCTAssertNotEqual() {
        XCTAssertNotEqual(2 + 2, 5)
        
        let result = "hello"
        XCTAssertNotEqual(result, "world")
    }
    
    // Floating point comparison
    func testFloatingPoint() {
        let calculated = 0.1 + 0.2
        // ❌ อาจ fail เพราะ floating point precision
        // XCTAssertEqual(calculated, 0.3)
        
        // ✅ ใช้ accuracy
        XCTAssertEqual(calculated, 0.3, accuracy: 0.0001, "Floating point ควรใกล้เคียง 0.3")
    }
}
```

### XCTAssertNil และ XCTAssertNotNil

```swift
class NilTests: XCTestCase {
    
    // XCTAssertNil - ตรวจสอบว่าเป็น nil
    func testXCTAssertNil() {
        let optionalValue: String? = nil
        XCTAssertNil(optionalValue)
        
        // ใช้กับ function ที่ return optional
        let result = findUser(byId: "nonexistent")
        XCTAssertNil(result, "ไม่ควรพบ user ที่ไม่มีอยู่")
    }
    
    // XCTAssertNotNil - ตรวจสอบว่าไม่เป็น nil
    func testXCTAssertNotNil() {
        let value: String? = "Hello"
        XCTAssertNotNil(value)
        
        // ใช้กับ object ที่ควรมีค่า
        let user = createUser(name: "ทดสอบ")
        XCTAssertNotNil(user, "ควรสร้าง user ได้สำเร็จ")
    }
    
    // XCTUnwrap - unwrap optional และ fail ถ้าเป็น nil
    func testXCTUnwrap() throws {
        let optionalValue: String? = "Hello"
        
        // XCTUnwrap จะ throw ถ้าค่าเป็น nil
        let value = try XCTUnwrap(optionalValue, "ค่าไม่ควรเป็น nil")
        XCTAssertEqual(value, "Hello")
        
        // ใช้แทน guard let ใน test
        let user: User? = User(name: "ทดสอบ")
        let unwrappedUser = try XCTUnwrap(user)
        XCTAssertEqual(unwrappedUser.name, "ทดสอบ")
    }
    
    private func findUser(byId id: String) -> User? {
        return nil  // stub
    }
    
    private func createUser(name: String) -> User? {
        return User(name: name)
    }
}
```

### XCTAssertThrowsError

```swift
class ErrorTests: XCTestCase {
    
    // XCTAssertThrowsError - ตรวจสอบว่า throw error
    func testThrowsError() {
        // Basic - แค่ตรวจสอบว่า throw
        XCTAssertThrowsError(try riskyOperation()) { error in
            // ตรวจสอบ error ที่ได้
            print("ได้รับ error: \(error)")
        }
    }
    
    // ตรวจสอบ error type และ message
    func testSpecificError() {
        XCTAssertThrowsError(try divide(10, by: 0)) { error in
            // ตรวจสอบว่า error เป็น type ที่ถูกต้อง
            guard let calcError = error as? CalculatorError else {
                XCTFail("ต้องเป็น CalculatorError")
                return
            }
            XCTAssertEqual(calcError, .divisionByZero)
        }
    }
    
    // XCTAssertNoThrow - ตรวจสอบว่าไม่ throw error
    func testNoThrow() {
        XCTAssertNoThrow(try divide(10, by: 2))
        XCTAssertNoThrow(try safeOperation())
    }
    
    private func riskyOperation() throws {
        throw NSError(domain: "test", code: 1)
    }
    
    private func divide(_ a: Double, by b: Double) throws -> Double {
        guard b != 0 else { throw CalculatorError.divisionByZero }
        return a / b
    }
    
    private func safeOperation() throws { }
}
```

### XCTAssertGreaterThan, XCTAssertLessThan

```swift
class ComparisonAssertionTests: XCTestCase {
    
    func testComparisonAssertions() {
        let score = 85
        
        // XCTAssertGreaterThan
        XCTAssertGreaterThan(score, 60, "คะแนนควรมากกว่า 60")
        
        // XCTAssertGreaterThanOrEqual
        XCTAssertGreaterThanOrEqual(score, 85)
        
        // XCTAssertLessThan
        XCTAssertLessThan(score, 100)
        
        // XCTAssertLessThanOrEqual
        XCTAssertLessThanOrEqual(score, 100)
        
        // ใช้กับ Array count
        let items = [1, 2, 3, 4, 5]
        XCTAssertGreaterThan(items.count, 0)
        XCTAssertLessThanOrEqual(items.count, 10)
    }
}
```

---

## 35.9 Test Failure Messages

การเขียน failure message ที่ดีช่วยให้เข้าใจว่าทำไม test ถึง fail

```swift
class FailureMessageTests: XCTestCase {
    
    // การใช้ message parameter
    func testWithGoodMessages() {
        let user = UserService().createUser(name: "")
        
        // ❌ message ไม่ชัดเจน
        XCTAssertNotNil(user)
        
        // ✅ message ที่ชัดเจน
        XCTAssertNotNil(user, "ควรสร้าง user ได้แม้ชื่อจะว่าง")
        
        // บอก context ให้ชัดเจน
        let items = fetchItems()
        XCTAssertEqual(items.count, 3,
                      "ควรพบ 3 items แต่พบ \(items.count) items")
    }
    
    // XCTFail - fail test ด้วย message
    func testXCTFail() {
        let paymentMethod = "Unknown"
        
        switch paymentMethod {
        case "Credit Card":
            // handle credit card
            break
        case "PayPal":
            // handle paypal
            break
        default:
            XCTFail("ไม่รู้จัก payment method: \(paymentMethod)")
        }
    }
    
    // การใช้ context ใน message
    func testWithContext() {
        let orders = [
            Order(id: "1", status: .pending),
            Order(id: "2", status: .completed),
            Order(id: "3", status: .pending)
        ]
        
        for order in orders {
            if order.status == .pending {
                XCTAssertNotNil(order.createdAt,
                               "Order ID: \(order.id) ที่ pending ควรมี createdAt")
            }
        }
    }
    
    private func fetchItems() -> [String] {
        return ["a", "b", "c"]
    }
}
```

---

## 35.10 Async Testing (async/await)

Swift 5.5+ รองรับการทดสอบ async code โดยตรง

### การทดสอบ async functions

```swift
class AsyncTests: XCTestCase {
    
    // ทดสอบ async function โดยตรง
    func testAsyncFetch() async throws {
        let service = UserService()
        
        // ใช้ async/await โดยตรงใน test
        let users = try await service.fetchUsers()
        
        XCTAssertFalse(users.isEmpty, "ควรพบ users อย่างน้อย 1 คน")
        XCTAssertEqual(users.count, 3)
    }
    
    // ทดสอบ async function ที่ throw error
    func testAsyncWithError() async {
        let service = NetworkService()
        
        do {
            let result = try await service.fetch(from: URL(string: "invalid-url")!)
            XCTFail("ควร throw error แต่ได้ result: \(result)")
        } catch {
            XCTAssertNotNil(error)
        }
    }
    
    // ทดสอบ concurrent operations
    func testConcurrentOperations() async throws {
        let service = DataService()
        
        // รัน parallel operations
        async let users = service.fetchUsers()
        async let products = service.fetchProducts()
        async let orders = service.fetchOrders()
        
        let (fetchedUsers, fetchedProducts, fetchedOrders) = try await (users, products, orders)
        
        XCTAssertFalse(fetchedUsers.isEmpty)
        XCTAssertFalse(fetchedProducts.isEmpty)
        XCTAssertFalse(fetchedOrders.isEmpty)
    }
    
    // ทดสอบ MainActor
    func testMainActorOperation() async {
        let viewModel = await MainActor.run {
            ViewModel()
        }
        
        await viewModel.loadData()
        
        let state = await MainActor.run {
            viewModel.state
        }
        
        XCTAssertEqual(state, .loaded)
    }
}
```

### ตัวอย่าง Service สำหรับทดสอบ

```swift
// UserService.swift
actor UserService {
    private var cachedUsers: [User] = []
    
    func fetchUsers() async throws -> [User] {
        try await Task.sleep(nanoseconds: 100_000_000)  // จำลอง network delay
        return [
            User(id: "1", name: "สมชาย"),
            User(id: "2", name: "สมหญิง"),
            User(id: "3", name: "สมศักดิ์")
        ]
    }
}

// UserServiceTests.swift
class UserServiceTests: XCTestCase {
    
    var sut: UserService!  // System Under Test
    
    override func setUp() {
        super.setUp()
        sut = UserService()
    }
    
    func testFetchUsers_ReturnsCorrectCount() async throws {
        let users = try await sut.fetchUsers()
        XCTAssertEqual(users.count, 3)
    }
    
    func testFetchUsers_ContainsExpectedNames() async throws {
        let users = try await sut.fetchUsers()
        let names = users.map { $0.name }
        
        XCTAssertTrue(names.contains("สมชาย"))
        XCTAssertTrue(names.contains("สมหญิง"))
    }
}
```

---

## 35.11 Expectations (XCTestExpectation)

XCTestExpectation ใช้สำหรับทดสอบ asynchronous code แบบ callback-based

### การใช้งาน XCTestExpectation พื้นฐาน

```swift
class ExpectationTests: XCTestCase {
    
    // ทดสอบ callback-based async code
    func testCallbackBasedAsync() {
        let expectation = expectation(description: "ข้อมูลถูกโหลดแล้ว")
        
        let service = LegacyService()
        service.fetchData { result in
            switch result {
            case .success(let data):
                XCTAssertFalse(data.isEmpty)
                expectation.fulfill()  // บอกว่า expectation สำเร็จแล้ว
            case .failure(let error):
                XCTFail("ไม่ควรเกิด error: \(error)")
            }
        }
        
        // รอให้ expectation fulfill ภายใน 5 วินาที
        waitForExpectations(timeout: 5) { error in
            if let error = error {
                XCTFail("Timeout: \(error)")
            }
        }
    }
    
    // Multiple expectations
    func testMultipleAsyncOperations() {
        let firstExpectation = expectation(description: "Operation 1")
        let secondExpectation = expectation(description: "Operation 2")
        
        DispatchQueue.global().async {
            sleep(1)
            firstExpectation.fulfill()
        }
        
        DispatchQueue.global().async {
            sleep(2)
            secondExpectation.fulfill()
        }
        
        // รอให้ทั้งสอง expectations fulfill
        wait(for: [firstExpectation, secondExpectation], timeout: 5)
    }
    
    // Expectation กับ notification
    func testNotificationPosted() {
        let notificationName = Notification.Name("DataUpdated")
        let expectation = XCTNSNotificationExpectation(
            name: notificationName,
            object: nil
        )
        
        // trigger ที่ส่ง notification
        DispatchQueue.main.asyncAfter(deadline: .now() + 0.5) {
            NotificationCenter.default.post(name: notificationName, object: nil)
        }
        
        wait(for: [expectation], timeout: 2)
    }
    
    // Expectation พร้อม expectedFulfillmentCount
    func testMultipleFulfillments() {
        let expectation = expectation(description: "รับข้อมูลหลายครั้ง")
        expectation.expectedFulfillmentCount = 3
        
        var count = 0
        let timer = Timer.scheduledTimer(withTimeInterval: 0.1, repeats: true) { timer in
            count += 1
            expectation.fulfill()
            if count >= 3 {
                timer.invalidate()
            }
        }
        
        wait(for: [expectation], timeout: 2)
        XCTAssertEqual(count, 3)
    }
    
    // Inverted expectation - ตรวจสอบว่าบางอย่าง ไม่ เกิดขึ้น
    func testSomethingDoesNotHappen() {
        let expectation = expectation(description: "ไม่ควรเกิด callback")
        expectation.isInverted = true  // ถ้า fulfill จะ fail
        
        let service = ConditionalService(shouldCallback: false)
        service.perform { _ in
            expectation.fulfill()  // ถ้าถูกเรียก = fail
        }
        
        wait(for: [expectation], timeout: 1)
    }
}
```

---

## 35.12 Performance Tests

XCTest รองรับการวัดประสิทธิภาพของโค้ด

### การใช้งาน measure

```swift
class PerformanceTests: XCTestCase {
    
    // วัดเวลาของ synchronous code
    func testSortingPerformance() {
        let numbers = (1...10000).shuffled()
        
        measure {
            // โค้ดที่ต้องการวัดประสิทธิภาพ
            let _ = numbers.sorted()
        }
    }
    
    // วัดเวลาของ async code
    func testAsyncPerformance() {
        measure {
            let expectation = expectation(description: "async operation")
            
            Task {
                let _ = await heavyComputation()
                expectation.fulfill()
            }
            
            wait(for: [expectation], timeout: 30)
        }
    }
    
    // กำหนด baseline สำหรับ performance
    func testWithBaseline() {
        let options = XCTMeasureOptions.default
        options.iterationCount = 10  // รัน 10 ครั้ง
        
        measure(options: options) {
            let data = generateTestData(count: 1000)
            let _ = processData(data)
        }
    }
    
    // วัด memory footprint
    func testMemoryPerformance() {
        let options = XCTMeasureOptions()
        // ใช้ metrics ที่ต้องการ
        
        measureMetrics([.wallClockTime], automaticallyStartMeasuring: false) {
            let data = generateTestData(count: 10000)
            
            startMeasuring()
            let _ = processData(data)
            stopMeasuring()
        }
    }
    
    private func heavyComputation() async -> [Int] {
        return (1...1000).map { $0 * $0 }
    }
    
    private func generateTestData(count: Int) -> [Int] {
        return (1...count).map { _ in Int.random(in: 1...1000) }
    }
    
    private func processData(_ data: [Int]) -> [Int] {
        return data.filter { $0 % 2 == 0 }.sorted()
    }
}
```

---

## 35.13 UI Testing (XCUITest)

XCUITest ช่วยให้ทดสอบ User Interface ของแอปได้

### การตั้งค่า UI Test

```swift
import XCTest

class MyAppUITests: XCTestCase {
    
    var app: XCUIApplication!
    
    override func setUpWithError() throws {
        continueAfterFailure = false  // หยุดทดสอบทันทีเมื่อ fail
        
        app = XCUIApplication()
        app.launchArguments = ["--uitesting"]  // ส่ง arguments ไปยัง app
        app.launchEnvironment = ["UITEST_MODE": "1"]
        app.launch()
    }
    
    override func tearDownWithError() throws {
        app = nil
    }
    
    // ทดสอบการ login
    func testLoginFlow() {
        // หา elements
        let usernameField = app.textFields["usernameField"]
        let passwordField = app.secureTextFields["passwordField"]
        let loginButton = app.buttons["loginButton"]
        
        // ตรวจสอบว่า elements มีอยู่
        XCTAssertTrue(usernameField.exists)
        XCTAssertTrue(passwordField.exists)
        XCTAssertTrue(loginButton.exists)
        
        // กรอกข้อมูล
        usernameField.tap()
        usernameField.typeText("testuser@example.com")
        
        passwordField.tap()
        passwordField.typeText("password123")
        
        // กดปุ่ม login
        loginButton.tap()
        
        // ตรวจสอบผลลัพธ์
        let welcomeLabel = app.staticTexts["welcomeLabel"]
        XCTAssertTrue(welcomeLabel.waitForExistence(timeout: 3))
        XCTAssertEqual(welcomeLabel.label, "ยินดีต้อนรับ!")
    }
    
    // ทดสอบ navigation
    func testNavigationToSettings() {
        // กดปุ่ม Settings
        app.tabBars.buttons["Settings"].tap()
        
        // ตรวจสอบว่า navigate ไปยัง Settings page
        let settingsTitle = app.navigationBars["การตั้งค่า"]
        XCTAssertTrue(settingsTitle.exists)
    }
    
    // ทดสอบ scroll
    func testScrollTableView() {
        let tableView = app.tables.firstMatch
        
        // Scroll ลงไป
        tableView.swipeUp()
        
        // ตรวจสอบว่ามี cell ที่ต้องการ
        let cell = tableView.cells.element(boundBy: 5)
        XCTAssertTrue(cell.exists)
    }
}
```

---

## 35.14 UI Element Queries

XCUITest มี API สำหรับ query หา UI elements

```swift
class UIElementQueryTests: XCTestCase {
    
    var app: XCUIApplication!
    
    override func setUp() {
        super.setUp()
        app = XCUIApplication()
        app.launch()
    }
    
    func testElementQueries() {
        // Query ด้วย Accessibility Identifier
        let loginButton = app.buttons["loginButton"]
        
        // Query ด้วย label text
        let saveButton = app.buttons["บันทึก"]
        
        // Query ด้วย type
        let firstTextField = app.textFields.firstMatch
        let allButtons = app.buttons.allElements
        
        // Query แบบ nested
        let tableView = app.tables["mainTable"]
        let firstCell = tableView.cells.firstMatch
        let labelInCell = firstCell.staticTexts.firstMatch
        
        // ตรวจสอบ element exists
        XCTAssertTrue(loginButton.exists)
        
        // รอให้ element ปรากฏ
        XCTAssertTrue(loginButton.waitForExistence(timeout: 5))
        
        // ตรวจสอบ properties
        XCTAssertTrue(loginButton.isEnabled)
        XCTAssertTrue(loginButton.isHittable)
        
        // Query ด้วย predicate
        let enabledButtons = app.buttons.matching(
            NSPredicate(format: "enabled == true")
        )
        XCTAssertGreaterThan(enabledButtons.count, 0)
    }
    
    // ทดสอบ alert
    func testHandleAlert() {
        // trigger ที่ทำให้เกิด alert
        app.buttons["showAlertButton"].tap()
        
        // ตรวจสอบ alert
        let alert = app.alerts.firstMatch
        XCTAssertTrue(alert.waitForExistence(timeout: 2))
        XCTAssertEqual(alert.label, "ยืนยันการลบ")
        
        // กด OK
        alert.buttons["ตกลง"].tap()
        
        // ตรวจสอบว่า alert หายไป
        XCTAssertFalse(alert.exists)
    }
    
    // ทดสอบ screenshot
    func testTakeScreenshot() {
        let screenshot = app.screenshot()
        let attachment = XCTAttachment(screenshot: screenshot)
        attachment.name = "หน้าหลัก"
        attachment.lifetime = .keepAlways
        add(attachment)
    }
    
    // ทดสอบ gesture
    func testGestures() {
        let element = app.views["scrollableView"]
        
        // Swipe
        element.swipeUp()
        element.swipeDown()
        element.swipeLeft()
        element.swipeRight()
        
        // Pinch (zoom)
        element.pinch(withScale: 2.0, velocity: 1.0)  // zoom in
        element.pinch(withScale: 0.5, velocity: -1.0)  // zoom out
        
        // Rotate
        element.rotate(.pi / 4, withVelocity: 1.0)
        
        // Long press
        element.press(forDuration: 1.0)
        
        // Double tap
        element.doubleTap()
    }
}
```

---

## 35.15 Test Plans

Test Plan ช่วยจัดการ test configuration สำหรับ scenarios ต่างๆ

### การสร้าง Test Plan ใน Xcode

1. File → New → File → Test Plan
2. หรือ Product → Scheme → Edit Scheme → Test → Plans → Add Test Plan

### โครงสร้าง Test Plan (.xctestplan)

```json
{
    "configurations": [
        {
            "id": "01234567-89AB-CDEF-0123-456789ABCDEF",
            "name": "Thai Language",
            "options": {
                "language": "th",
                "region": "TH",
                "commandLineArgumentEntries": [
                    {"argument": "--reset-database"}
                ],
                "environmentVariableEntries": [
                    {"key": "TEST_ENV", "value": "staging"}
                ]
            }
        },
        {
            "id": "FEDCBA98-7654-3210-FEDC-BA9876543210",
            "name": "English Language",
            "options": {
                "language": "en",
                "region": "US"
            }
        }
    ],
    "defaultOptions": {
        "codeCoverage": "enabled",
        "targetForVariableExpansion": {
            "containerPath": "container:MyApp.xcodeproj",
            "identifier": "MyApp",
            "name": "MyApp"
        }
    },
    "testTargets": [
        {
            "target": {
                "containerPath": "container:MyApp.xcodeproj",
                "identifier": "MyAppTests",
                "name": "MyAppTests"
            }
        }
    ],
    "version": 1
}
```

---

## 35.16 Code Coverage

Code coverage บอกว่าโค้ดส่วนไหนถูก test ครอบคลุมบ้าง

### การเปิดใช้ Code Coverage

1. Product → Scheme → Edit Scheme
2. Tab "Test" → Options
3. เปิด "Gather coverage for..."

### การดู Coverage Report

หลังจากรัน test สามารถดู coverage ได้จาก:
- Report Navigator (Cmd+9) → Test Report → Coverage tab

### เป้าหมาย Coverage ที่ดี

```
Coverage Levels:
- 80%+ = ดีมาก
- 60-80% = ดีพอใช้
- < 60% = ต้องเพิ่ม test

หมายเหตุ: 100% coverage ไม่ได้แปลว่า bug-free!
```

### การวิเคราะห์ Coverage

```swift
// ตัวอย่างโค้ดที่ test coverage ครอบคลุมน้อย
class PaymentProcessor {
    func processPayment(amount: Double, method: PaymentMethod) throws -> Receipt {
        // ✅ ส่วนนี้ถูก test
        guard amount > 0 else {
            throw PaymentError.invalidAmount
        }
        
        // ✅ ส่วนนี้ถูก test
        switch method {
        case .creditCard:
            return try processCreditCard(amount: amount)
        case .paypal:
            return try processPayPal(amount: amount)
        case .bankTransfer:
            // ❌ ส่วนนี้ยังไม่ถูก test!
            return try processBankTransfer(amount: amount)
        case .cryptocurrency:
            // ❌ ส่วนนี้ยังไม่ถูก test!
            return try processCrypto(amount: amount)
        }
    }
}
```

---

## 35.17 Mocking และ Stubbing

Mocking ช่วยให้แยกการทดสอบจาก dependencies ภายนอก

### การสร้าง Mock Objects

```swift
// Protocol สำหรับ dependency
protocol NetworkServiceProtocol {
    func fetch(from url: URL) async throws -> Data
}

// Mock implementation
class MockNetworkService: NetworkServiceProtocol {
    var shouldFail = false
    var mockData: Data = Data()
    var fetchCallCount = 0
    var lastURL: URL?
    
    func fetch(from url: URL) async throws -> Data {
        fetchCallCount += 1
        lastURL = url
        
        if shouldFail {
            throw NetworkError.connectionFailed
        }
        
        return mockData
    }
}

// การใช้งาน Mock ใน test
class UserRepositoryTests: XCTestCase {
    
    var mockNetwork: MockNetworkService!
    var repository: UserRepository!
    
    override func setUp() {
        super.setUp()
        mockNetwork = MockNetworkService()
        repository = UserRepository(networkService: mockNetwork)
    }
    
    func testFetchUsers_Success() async throws {
        // Arrange
        let expectedUsers = [User(id: "1", name: "สมชาย")]
        let jsonData = try JSONEncoder().encode(expectedUsers)
        mockNetwork.mockData = jsonData
        
        // Act
        let users = try await repository.fetchUsers()
        
        // Assert
        XCTAssertEqual(users.count, 1)
        XCTAssertEqual(users.first?.name, "สมชาย")
        XCTAssertEqual(mockNetwork.fetchCallCount, 1)
    }
    
    func testFetchUsers_NetworkFailure() async {
        // Arrange
        mockNetwork.shouldFail = true
        
        // Act & Assert
        do {
            let _ = try await repository.fetchUsers()
            XCTFail("ควร throw error")
        } catch {
            XCTAssertNotNil(error)
        }
    }
}
```

### Stub

```swift
// Stub คือ object ที่ return ค่าที่กำหนดไว้ล่วงหน้า
class StubUserService: UserServiceProtocol {
    
    // Predefined responses
    var usersToReturn: [User] = []
    var errorToThrow: Error? = nil
    
    func getUsers() async throws -> [User] {
        if let error = errorToThrow {
            throw error
        }
        return usersToReturn
    }
}

// การใช้งาน
func testUserList_ShowsCorrectCount() async throws {
    let stub = StubUserService()
    stub.usersToReturn = [
        User(id: "1", name: "A"),
        User(id: "2", name: "B")
    ]
    
    let viewModel = UserListViewModel(userService: stub)
    await viewModel.loadUsers()
    
    XCTAssertEqual(viewModel.users.count, 2)
}
```

---

## 35.18 Dependency Injection for Testability

Dependency Injection (DI) ทำให้โค้ด testable ได้ง่ายขึ้น

### รูปแบบ Dependency Injection

```swift
// ❌ แบบที่ test ยาก - Hard-coded dependency
class UserViewModel {
    private let service = UserService()  // สร้างเอง ไม่สามารถ mock ได้
    
    func loadUsers() async {
        let users = try? await service.fetchUsers()
        // ...
    }
}

// ✅ แบบที่ test ง่าย - Dependency Injection
class UserViewModel {
    private let service: UserServiceProtocol
    
    // Constructor Injection
    init(service: UserServiceProtocol = UserService()) {
        self.service = service
    }
    
    func loadUsers() async throws {
        let users = try await service.fetchUsers()
        // ...
    }
}

// ใน production code
let viewModel = UserViewModel()  // ใช้ default

// ใน test code
let mockService = MockUserService()
let viewModel = UserViewModel(service: mockService)  // inject mock
```

### Protocol-based DI

```swift
// Define protocol
protocol DateProviding {
    var now: Date { get }
}

// Production implementation
struct SystemDateProvider: DateProviding {
    var now: Date { Date() }
}

// Test implementation
struct MockDateProvider: DateProviding {
    let fixedDate: Date
    var now: Date { fixedDate }
}

// Class ที่ใช้ date
class SubscriptionManager {
    private let dateProvider: DateProviding
    
    init(dateProvider: DateProviding = SystemDateProvider()) {
        self.dateProvider = dateProvider
    }
    
    func isSubscriptionActive(expiryDate: Date) -> Bool {
        return dateProvider.now < expiryDate
    }
}

// Test
class SubscriptionTests: XCTestCase {
    
    func testActiveSubscription() {
        // กำหนดวันที่คงที่สำหรับ test
        let fixedDate = Calendar.current.date(
            from: DateComponents(year: 2024, month: 1, day: 15)
        )!
        let mockDateProvider = MockDateProvider(fixedDate: fixedDate)
        
        let manager = SubscriptionManager(dateProvider: mockDateProvider)
        
        let futureExpiry = Calendar.current.date(
            from: DateComponents(year: 2024, month: 12, day: 31)
        )!
        
        XCTAssertTrue(manager.isSubscriptionActive(expiryDate: futureExpiry))
    }
    
    func testExpiredSubscription() {
        let fixedDate = Calendar.current.date(
            from: DateComponents(year: 2025, month: 1, day: 15)
        )!
        let mockDateProvider = MockDateProvider(fixedDate: fixedDate)
        
        let manager = SubscriptionManager(dateProvider: mockDateProvider)
        
        let pastExpiry = Calendar.current.date(
            from: DateComponents(year: 2024, month: 12, day: 31)
        )!
        
        XCTAssertFalse(manager.isSubscriptionActive(expiryDate: pastExpiry))
    }
}
```

---

## 35.19 TDD (Test-Driven Development)

TDD คือแนวทางการพัฒนาที่เขียน test ก่อนเขียน implementation

### วงจร TDD: Red → Green → Refactor

```
Red: เขียน test ที่ fail ก่อน
  ↓
Green: เขียน code น้อยที่สุดให้ test pass
  ↓
Refactor: ปรับปรุงโค้ดให้ดีขึ้น
  ↓
(วนซ้ำ)
```

### ตัวอย่าง TDD: สร้าง ShoppingCart

**Step 1: Red - เขียน test ก่อน**

```swift
// ShoppingCartTests.swift - เขียนก่อนที่จะมี implementation
import XCTest
@testable import MyShop

class ShoppingCartTests: XCTestCase {
    
    func test_emptyCart_hasZeroItems() {
        let cart = ShoppingCart()
        XCTAssertEqual(cart.itemCount, 0)
    }
    
    func test_addItem_increasesItemCount() {
        var cart = ShoppingCart()
        let item = CartItem(name: "สินค้า A", price: 100)
        
        cart.add(item)
        
        XCTAssertEqual(cart.itemCount, 1)
    }
    
    func test_addMultipleItems_countMatchesAdded() {
        var cart = ShoppingCart()
        
        cart.add(CartItem(name: "A", price: 100))
        cart.add(CartItem(name: "B", price: 200))
        cart.add(CartItem(name: "C", price: 150))
        
        XCTAssertEqual(cart.itemCount, 3)
    }
    
    func test_totalPrice_sumOfItemPrices() {
        var cart = ShoppingCart()
        
        cart.add(CartItem(name: "A", price: 100))
        cart.add(CartItem(name: "B", price: 200))
        
        XCTAssertEqual(cart.totalPrice, 300)
    }
    
    func test_removeItem_decreasesCount() {
        var cart = ShoppingCart()
        let item = CartItem(name: "A", price: 100)
        
        cart.add(item)
        cart.remove(item)
        
        XCTAssertEqual(cart.itemCount, 0)
    }
    
    func test_applyDiscount_reducesTotal() {
        var cart = ShoppingCart()
        cart.add(CartItem(name: "A", price: 1000))
        
        cart.applyDiscount(percentage: 10)
        
        XCTAssertEqual(cart.totalPrice, 900)
    }
}
```

**Step 2: Green - เขียน implementation**

```swift
// ShoppingCart.swift
struct CartItem: Equatable {
    let id: UUID
    let name: String
    let price: Double
    
    init(name: String, price: Double) {
        self.id = UUID()
        self.name = name
        self.price = price
    }
}

struct ShoppingCart {
    private var items: [CartItem] = []
    private var discountPercentage: Double = 0
    
    var itemCount: Int {
        items.count
    }
    
    var totalPrice: Double {
        let subtotal = items.reduce(0) { $0 + $1.price }
        let discount = subtotal * (discountPercentage / 100)
        return subtotal - discount
    }
    
    mutating func add(_ item: CartItem) {
        items.append(item)
    }
    
    mutating func remove(_ item: CartItem) {
        items.removeAll { $0.id == item.id }
    }
    
    mutating func applyDiscount(percentage: Double) {
        discountPercentage = percentage
    }
}
```

**Step 3: Refactor**

```swift
// ปรับปรุง implementation หลังจาก test pass แล้ว
extension ShoppingCart {
    var isEmpty: Bool { items.isEmpty }
    var hasDiscount: Bool { discountPercentage > 0 }
    var discountAmount: Double { 
        items.reduce(0) { $0 + $1.price } * (discountPercentage / 100)
    }
}
```

---

## 35.20 BDD Approach

BDD (Behavior-Driven Development) เน้นการเขียน test ในรูปแบบภาษาธรรมชาติ

### โครงสร้าง Given-When-Then

```swift
class OrderProcessingTests: XCTestCase {
    
    // Format: test_given[Precondition]_when[Action]_then[ExpectedResult]
    
    func test_givenEmptyCart_whenCheckingOut_thenThrowsEmptyCartError() {
        // Given - สภาพแวดล้อมเริ่มต้น
        let cart = ShoppingCart()
        let checkout = CheckoutService()
        
        // When - การกระทำที่ทดสอบ
        // Then - ผลลัพธ์ที่คาดหวัง
        XCTAssertThrowsError(try checkout.process(cart)) { error in
            XCTAssertEqual(error as? CheckoutError, .emptyCart)
        }
    }
    
    func test_givenItemsInCart_whenApplyingValidCoupon_thenDiscountApplied() {
        // Given
        var cart = ShoppingCart()
        cart.add(CartItem(name: "สินค้า", price: 1000))
        let couponService = CouponService()
        
        // When
        let coupon = Coupon(code: "SAVE20", discount: 20)
        couponService.apply(coupon, to: &cart)
        
        // Then
        XCTAssertEqual(cart.totalPrice, 800)
        XCTAssertTrue(cart.hasDiscount)
    }
    
    func test_givenOrderPlaced_whenPaymentSucceeds_thenOrderConfirmed() async throws {
        // Given
        let order = Order(items: [CartItem(name: "A", price: 500)])
        let mockPayment = MockPaymentGateway(shouldSucceed: true)
        let orderService = OrderService(paymentGateway: mockPayment)
        
        // When
        let result = try await orderService.place(order)
        
        // Then
        XCTAssertEqual(result.status, .confirmed)
        XCTAssertNotNil(result.confirmationNumber)
    }
}
```

---

## 35.21 Testing Combine Publishers

Combine framework ต้องการวิธีการทดสอบที่เฉพาะเจาะจง

```swift
import XCTest
import Combine
@testable import MyApp

class CombineTests: XCTestCase {
    
    var cancellables = Set<AnyCancellable>()
    
    override func tearDown() {
        cancellables.removeAll()
        super.tearDown()
    }
    
    // ทดสอบ Publisher ที่ emit ค่าเดียว
    func testPublisherEmitsValue() {
        let expectation = expectation(description: "ได้รับค่า")
        var receivedValue: String?
        
        Just("Hello")
            .sink { value in
                receivedValue = value
                expectation.fulfill()
            }
            .store(in: &cancellables)
        
        wait(for: [expectation], timeout: 1)
        XCTAssertEqual(receivedValue, "Hello")
    }
    
    // ทดสอบ Publisher ที่ emit หลายค่า
    func testPublisherEmitsMultipleValues() {
        let expectation = expectation(description: "ได้รับทุกค่า")
        expectation.expectedFulfillmentCount = 3
        var receivedValues: [Int] = []
        
        [1, 2, 3].publisher
            .sink { value in
                receivedValues.append(value)
                expectation.fulfill()
            }
            .store(in: &cancellables)
        
        wait(for: [expectation], timeout: 1)
        XCTAssertEqual(receivedValues, [1, 2, 3])
    }
    
    // ทดสอบ Publisher ที่ fail
    func testPublisherFailure() {
        let expectation = expectation(description: "ได้รับ error")
        var receivedError: Error?
        
        Fail<String, NetworkError>(error: .connectionFailed)
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
        
        wait(for: [expectation], timeout: 1)
        XCTAssertNotNil(receivedError)
    }
    
    // ทดสอบ ViewModel ที่ใช้ Combine
    func testViewModelPublishesState() {
        let expectation = expectation(description: "state เปลี่ยน")
        var states: [LoadingState] = []
        
        let viewModel = UserListViewModel()
        viewModel.$loadingState
            .sink { state in
                states.append(state)
                if case .loaded = state {
                    expectation.fulfill()
                }
            }
            .store(in: &cancellables)
        
        viewModel.loadData()
        
        wait(for: [expectation], timeout: 5)
        XCTAssertTrue(states.contains(.loading))
        XCTAssertTrue(states.last == .loaded)
    }
    
    // ทดสอบ Subject
    func testPassthroughSubject() {
        let subject = PassthroughSubject<Int, Never>()
        var received: [Int] = []
        
        subject
            .sink { received.append($0) }
            .store(in: &cancellables)
        
        subject.send(1)
        subject.send(2)
        subject.send(3)
        
        XCTAssertEqual(received, [1, 2, 3])
    }
    
    // ทดสอบ CurrentValueSubject
    func testCurrentValueSubject() {
        let subject = CurrentValueSubject<Int, Never>(0)
        var received: [Int] = []
        
        subject
            .sink { received.append($0) }
            .store(in: &cancellables)
        
        subject.value = 10
        subject.value = 20
        
        XCTAssertEqual(received, [0, 10, 20])  // รวม initial value
        XCTAssertEqual(subject.value, 20)
    }
}
```

---

## 35.22 Swift Testing Framework (Swift 6)

Swift 6 มาพร้อมกับ Swift Testing framework ใหม่ที่มี syntax สะอาดกว่า XCTest

### การ import

```swift
import Testing  // import Swift Testing
```

### @Test attribute

```swift
import Testing

// ใช้ @Test แทน test prefix
struct CalculatorTests {
    
    @Test func addition() {
        let result = Calculator().add(2, 3)
        #expect(result == 5)
    }
    
    @Test("การลบเลขบวก")  // Custom test name
    func subtraction() {
        let result = Calculator().subtract(10, 3)
        #expect(result == 7)
    }
    
    // Test ใน extension
    @Test func multiplicationByZero() {
        let result = Calculator().multiply(5, 0)
        #expect(result == 0)
    }
}
```

### #expect macro

```swift
import Testing

struct ExpectTests {
    
    @Test func basicExpectations() {
        // #expect แทน XCTAssert
        let value = 42
        #expect(value == 42)
        #expect(value > 0)
        #expect(value != 0)
        
        // Optional checking
        let optional: String? = "Hello"
        #expect(optional != nil)
        
        // Collection
        let array = [1, 2, 3]
        #expect(array.count == 3)
        #expect(array.contains(2))
    }
    
    @Test func withCustomMessage() {
        let score = 75
        #expect(score >= 60, "คะแนนควรผ่านเกณฑ์")
    }
    
    // #require - throw ถ้าไม่ตรงตามเงื่อนไข (หยุด test ทันที)
    @Test func requireUnwrap() throws {
        let optional: String? = "Hello"
        let value = try #require(optional)  // throw ถ้า nil
        #expect(value == "Hello")
    }
    
    // Testing throws
    @Test func throwingFunction() throws {
        #expect(throws: CalculatorError.divisionByZero) {
            try Calculator().divide(10, 0)
        }
    }
    
    // ไม่ควร throw
    @Test func noThrow() throws {
        let result = try Calculator().divide(10, 2)
        #expect(result == 5)
    }
}
```

### Parameterized Tests

```swift
import Testing

struct ParameterizedTests {
    
    // ทดสอบด้วย multiple inputs
    @Test(arguments: [2, 3, 4, 5, 6])
    func isEven(number: Int) {
        // test รันสำหรับแต่ละค่าใน arguments
        if number % 2 == 0 {
            #expect(number.isMultiple(of: 2))
        }
    }
    
    // ทดสอบด้วย pairs ของค่า
    @Test(arguments: zip([1, 2, 3], [2, 4, 6]))
    func doubleValue(input: Int, expected: Int) {
        let result = input * 2
        #expect(result == expected)
    }
    
    // ทดสอบด้วย enum cases
    @Test(arguments: Direction.allCases)
    func directionHasOpposite(direction: Direction) {
        let opposite = direction.opposite
        #expect(opposite != direction)
    }
}

enum Direction: CaseIterable {
    case north, south, east, west
    
    var opposite: Direction {
        switch self {
        case .north: return .south
        case .south: return .north
        case .east: return .west
        case .west: return .east
        }
    }
}
```

### Test Suites และ Tags

```swift
import Testing

// Test Suite ด้วย struct
@Suite("Calculator Tests")
struct CalculatorTestSuite {
    
    let calculator = Calculator()
    
    @Test func addition() {
        #expect(calculator.add(1, 2) == 3)
    }
    
    @Test func subtraction() {
        #expect(calculator.subtract(5, 3) == 2)
    }
    
    // Nested suite
    @Suite("Division Tests")
    struct DivisionTests {
        @Test func normalDivision() throws {
            let result = try Calculator().divide(10, 2)
            #expect(result == 5)
        }
        
        @Test func divisionByZero() {
            #expect(throws: CalculatorError.divisionByZero) {
                try Calculator().divide(10, 0)
            }
        }
    }
}

// Tags สำหรับ categorize tests
extension Tag {
    @Tag static var critical: Self
    @Tag static var integration: Self
    @Tag static var performance: Self
}

struct TaggedTests {
    
    @Test(.tags(.critical))
    func criticalTest() {
        #expect(true)
    }
    
    @Test(.tags(.integration, .critical))
    func integrationTest() {
        #expect(true)
    }
}
```

### Async Tests ใน Swift Testing

```swift
import Testing

struct AsyncSwiftTests {
    
    @Test func asyncFetch() async throws {
        let service = UserService()
        let users = try await service.fetchUsers()
        #expect(users.count > 0)
    }
    
    @Test func concurrentOperations() async throws {
        async let users = UserService().fetchUsers()
        async let products = ProductService().fetchProducts()
        
        let (u, p) = try await (users, products)
        #expect(!u.isEmpty)
        #expect(!p.isEmpty)
    }
}
```

---

## 35.23 แบบฝึกหัดพร้อมเฉลย (Practical Exercises)

### แบบฝึกหัดที่ 1: ทดสอบ StringProcessor

**โจทย์:** สร้าง test สำหรับ StringProcessor

```swift
// StringProcessor.swift
struct StringProcessor {
    func reverse(_ string: String) -> String {
        String(string.reversed())
    }
    
    func isPalindrome(_ string: String) -> Bool {
        let cleaned = string.lowercased().filter { $0.isLetter }
        return cleaned == String(cleaned.reversed())
    }
    
    func wordCount(_ string: String) -> Int {
        string.split(separator: " ").count
    }
    
    func capitalize(_ string: String) -> String {
        string.split(separator: " ")
            .map { $0.capitalized }
            .joined(separator: " ")
    }
}
```

**เฉลย:**

```swift
// StringProcessorTests.swift
import XCTest
@testable import MyApp

class StringProcessorTests: XCTestCase {
    
    var processor: StringProcessor!
    
    override func setUp() {
        super.setUp()
        processor = StringProcessor()
    }
    
    // MARK: - Reverse Tests
    
    func testReverseNormalString() {
        XCTAssertEqual(processor.reverse("hello"), "olleh")
    }
    
    func testReverseThaiString() {
        XCTAssertEqual(processor.reverse("สวัสดี"), "ีดัสวั")
    }
    
    func testReverseEmptyString() {
        XCTAssertEqual(processor.reverse(""), "")
    }
    
    func testReverseSingleChar() {
        XCTAssertEqual(processor.reverse("A"), "A")
    }
    
    // MARK: - Palindrome Tests
    
    func testPalindrome_racecar() {
        XCTAssertTrue(processor.isPalindrome("racecar"))
    }
    
    func testPalindrome_caseInsensitive() {
        XCTAssertTrue(processor.isPalindrome("Racecar"))
    }
    
    func testPalindrome_withSpaces() {
        XCTAssertTrue(processor.isPalindrome("A man a plan a canal Panama"))
    }
    
    func testNotPalindrome() {
        XCTAssertFalse(processor.isPalindrome("hello"))
    }
    
    // MARK: - Word Count Tests
    
    func testWordCount_singleWord() {
        XCTAssertEqual(processor.wordCount("Hello"), 1)
    }
    
    func testWordCount_multipleWords() {
        XCTAssertEqual(processor.wordCount("Hello World Swift"), 3)
    }
    
    // MARK: - Capitalize Tests
    
    func testCapitalize() {
        XCTAssertEqual(processor.capitalize("hello world"), "Hello World")
    }
}
```

### แบบฝึกหัดที่ 2: การสร้าง Testable Code

**โจทย์:** สร้าง WeatherService ที่ testable

```swift
// WeatherServiceProtocol.swift
protocol WeatherAPIProtocol {
    func fetchWeather(city: String) async throws -> WeatherData
}

// WeatherData.swift
struct WeatherData: Codable {
    let city: String
    let temperature: Double
    let condition: String
}

// WeatherService.swift
class WeatherService {
    private let api: WeatherAPIProtocol
    
    init(api: WeatherAPIProtocol = RealWeatherAPI()) {
        self.api = api
    }
    
    func getTemperatureDescription(city: String) async throws -> String {
        let data = try await api.fetchWeather(city: city)
        
        switch data.temperature {
        case ..<0:
            return "หนาวมาก (\(data.temperature)°C)"
        case 0..<15:
            return "หนาว (\(data.temperature)°C)"
        case 15..<25:
            return "เย็นสบาย (\(data.temperature)°C)"
        case 25..<35:
            return "ร้อน (\(data.temperature)°C)"
        default:
            return "ร้อนมาก (\(data.temperature)°C)"
        }
    }
}

// MockWeatherAPI.swift (สำหรับ test)
class MockWeatherAPI: WeatherAPIProtocol {
    var mockWeather: WeatherData?
    var shouldThrow = false
    
    func fetchWeather(city: String) async throws -> WeatherData {
        if shouldThrow { throw WeatherError.networkFailed }
        return mockWeather ?? WeatherData(city: city, temperature: 25, condition: "Sunny")
    }
}

// WeatherServiceTests.swift
class WeatherServiceTests: XCTestCase {
    
    var mockAPI: MockWeatherAPI!
    var service: WeatherService!
    
    override func setUp() {
        super.setUp()
        mockAPI = MockWeatherAPI()
        service = WeatherService(api: mockAPI)
    }
    
    func testVeryHotTemperature() async throws {
        mockAPI.mockWeather = WeatherData(city: "Bangkok", temperature: 40, condition: "Hot")
        
        let description = try await service.getTemperatureDescription(city: "Bangkok")
        XCTAssertTrue(description.contains("ร้อนมาก"))
    }
    
    func testColdTemperature() async throws {
        mockAPI.mockWeather = WeatherData(city: "Chiang Mai", temperature: 10, condition: "Cool")
        
        let description = try await service.getTemperatureDescription(city: "Chiang Mai")
        XCTAssertTrue(description.contains("หนาว"))
    }
    
    func testNetworkError() async {
        mockAPI.shouldThrow = true
        
        do {
            let _ = try await service.getTemperatureDescription(city: "Any")
            XCTFail("ควร throw error")
        } catch {
            XCTAssertNotNil(error)
        }
    }
}
```

---

## 35.24 การสร้าง Testable Code ตั้งแต่ต้น

### หลักการออกแบบโค้ดที่ testable

```swift
// SOLID Principles ช่วยให้โค้ด testable

// 1. Single Responsibility Principle
// ❌ ทำหลายอย่างในที่เดียว
class UserManager {
    func createUser(name: String, email: String) -> User { ... }
    func saveToDatabase(_ user: User) { ... }
    func sendWelcomeEmail(_ user: User) { ... }
    func logActivity(_ message: String) { ... }
}

// ✅ แยก responsibility
class UserCreator {
    func createUser(name: String, email: String) -> User { ... }
}

class UserRepository {
    func save(_ user: User) { ... }
}

class EmailService {
    func sendWelcome(to user: User) { ... }
}

// 2. Dependency Inversion Principle
protocol UserRepositoryProtocol {
    func save(_ user: User) throws
    func findById(_ id: String) -> User?
}

class InMemoryUserRepository: UserRepositoryProtocol {
    private var users: [String: User] = [:]
    
    func save(_ user: User) throws {
        users[user.id] = user
    }
    
    func findById(_ id: String) -> User? {
        users[id]
    }
}

// 3. ใช้ Protocol สำหรับ external dependencies
protocol NetworkClient {
    func request(_ url: URL) async throws -> Data
}

class URLSessionClient: NetworkClient {
    func request(_ url: URL) async throws -> Data {
        let (data, _) = try await URLSession.shared.data(from: url)
        return data
    }
}

// Mock สำหรับ test
class MockNetworkClient: NetworkClient {
    var mockData = Data()
    var shouldFail = false
    
    func request(_ url: URL) async throws -> Data {
        if shouldFail { throw URLError(.notConnectedToInternet) }
        return mockData
    }
}
```

### Complete Testable Architecture Example

```swift
// MARK: - Domain Layer

struct Product: Equatable {
    let id: String
    let name: String
    let price: Double
    var stockCount: Int
}

// MARK: - Repository Protocol

protocol ProductRepositoryProtocol {
    func getAll() async throws -> [Product]
    func getById(_ id: String) async throws -> Product?
    func save(_ product: Product) async throws
    func delete(id: String) async throws
}

// MARK: - Use Case

class GetProductsUseCase {
    private let repository: ProductRepositoryProtocol
    
    init(repository: ProductRepositoryProtocol) {
        self.repository = repository
    }
    
    func execute() async throws -> [Product] {
        let products = try await repository.getAll()
        return products.filter { $0.stockCount > 0 }
            .sorted { $0.price < $1.price }
    }
}

// MARK: - Mock Repository

class MockProductRepository: ProductRepositoryProtocol {
    var products: [Product] = []
    var shouldFail = false
    
    func getAll() async throws -> [Product] {
        if shouldFail { throw RepositoryError.fetchFailed }
        return products
    }
    
    func getById(_ id: String) async throws -> Product? {
        if shouldFail { throw RepositoryError.fetchFailed }
        return products.first { $0.id == id }
    }
    
    func save(_ product: Product) async throws {
        if shouldFail { throw RepositoryError.saveFailed }
        if let index = products.firstIndex(where: { $0.id == product.id }) {
            products[index] = product
        } else {
            products.append(product)
        }
    }
    
    func delete(id: String) async throws {
        products.removeAll { $0.id == id }
    }
}

// MARK: - Tests

class GetProductsUseCaseTests: XCTestCase {
    
    var mockRepo: MockProductRepository!
    var useCase: GetProductsUseCase!
    
    override func setUp() {
        super.setUp()
        mockRepo = MockProductRepository()
        useCase = GetProductsUseCase(repository: mockRepo)
    }
    
    func testReturnsOnlyInStockProducts() async throws {
        // Given
        mockRepo.products = [
            Product(id: "1", name: "A", price: 100, stockCount: 5),
            Product(id: "2", name: "B", price: 200, stockCount: 0),  // out of stock
            Product(id: "3", name: "C", price: 150, stockCount: 3)
        ]
        
        // When
        let products = try await useCase.execute()
        
        // Then
        XCTAssertEqual(products.count, 2)
        XCTAssertFalse(products.contains { $0.stockCount == 0 })
    }
    
    func testReturnsSortedByPrice() async throws {
        // Given
        mockRepo.products = [
            Product(id: "1", name: "A", price: 300, stockCount: 1),
            Product(id: "2", name: "B", price: 100, stockCount: 1),
            Product(id: "3", name: "C", price: 200, stockCount: 1)
        ]
        
        // When
        let products = try await useCase.execute()
        
        // Then
        XCTAssertEqual(products.map { $0.price }, [100, 200, 300])
    }
    
    func testThrowsErrorWhenRepositoryFails() async {
        // Given
        mockRepo.shouldFail = true
        
        // When & Then
        do {
            let _ = try await useCase.execute()
            XCTFail("ควร throw error")
        } catch {
            XCTAssertNotNil(error)
        }
    }
    
    func testEmptyRepositoryReturnsEmptyArray() async throws {
        // Given - mockRepo.products = [] (default)
        
        // When
        let products = try await useCase.execute()
        
        // Then
        XCTAssertTrue(products.isEmpty)
    }
}
```

---

## 35.25 สรุป (Summary)

ในบทนี้เราได้เรียนรู้เกี่ยวกับการทดสอบใน Swift อย่างครอบคลุม:

### สิ่งที่ได้เรียนรู้

| หัวข้อ | สรุป |
|--------|------|
| XCTest | Testing framework มาตรฐานของ Apple |
| XCTestCase | Base class สำหรับสร้าง test classes |
| setUp/tearDown | Lifecycle methods สำหรับเตรียมและทำความสะอาด |
| Assertions | XCTAssert, XCTAssertEqual, XCTAssertNil, etc. |
| Async Testing | ใช้ async/await โดยตรงใน test functions |
| Expectations | สำหรับทดสอบ callback-based async code |
| Performance Tests | วัดประสิทธิภาพด้วย measure() |
| UI Tests | ทดสอบ UI ด้วย XCUITest |
| Mocking | สร้าง mock objects เพื่อแยก dependencies |
| DI | Dependency Injection เพื่อทำให้โค้ด testable |
| TDD | เขียน test ก่อน แล้วค่อยเขียน implementation |
| BDD | เขียน test ในรูปแบบ Given-When-Then |
| Combine Testing | ทดสอบ publishers และ subscribers |
| Swift Testing | Framework ใหม่ใน Swift 6 |

### Best Practices

1. **เขียน test ที่ชัดเจน** - ชื่อ test ควรบอกว่าทดสอบอะไร
2. **AAA Pattern** - Arrange, Act, Assert
3. **One assertion per test** (ถ้าเป็นไปได้)
4. **ใช้ Dependency Injection** - ทำให้ mock ได้ง่าย
5. **Avoid testing implementation details** - ทดสอบ behavior ไม่ใช่ implementation
6. **Keep tests fast** - test ที่ช้าจะไม่ถูกรัน
7. **F.I.R.S.T principles**:
   - **F**ast - รันเร็ว
   - **I**solated - แยกจากกัน
   - **R**epeatable - ผลลัพธ์เหมือนกันทุกครั้ง
   - **S**elf-validating - pass/fail อัตโนมัติ
   - **T**imely - เขียนตรงเวลา

### Quick Reference

```swift
// XCTest Basic Assertions
XCTAssert(expression)
XCTAssertTrue(expression)
XCTAssertFalse(expression)
XCTAssertEqual(a, b)
XCTAssertNotEqual(a, b)
XCTAssertNil(expression)
XCTAssertNotNil(expression)
XCTAssertThrowsError(expression)
XCTAssertNoThrow(expression)
XCTFail("message")

// Swift Testing
@Test func testName() { }
#expect(expression)
try #require(expression)

// Async
func testAsync() async throws { }
let exp = expectation(description: "")
wait(for: [exp], timeout: 5)
```

---

*บทต่อไป: Part 36 - Debugging และ Profiling*
