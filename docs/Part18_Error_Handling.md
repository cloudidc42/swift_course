# ส่วนที่ 18: Error Handling (การจัดการข้อผิดพลาด)

## บทนำ

Error handling คือกระบวนการตอบสนองต่อสภาวะข้อผิดพลาดที่เกิดขึ้นระหว่างการทำงานของโปรแกรม Swift มีระบบ error handling ที่ชัดเจนและปลอดภัย ที่ช่วยให้เราเขียน code ที่จัดการข้อผิดพลาดได้อย่างเป็นระบบ

ใน Swift การ throw, catching, propagating และ manipulating errors แต่ละอย่างถูกออกแบบมาให้ชัดเจนในระดับ language ซึ่งต่างจากหลายภาษาที่ error handling เป็นเพียง convention

---

## 18.1 Error Protocol

ใน Swift error คือ value ที่ conform to `Error` protocol `Error` เป็น protocol ว่างเปล่าที่ใช้เป็น marker:

```swift
public protocol Error: Sendable {
    // ว่างเปล่า - เป็นแค่ marker protocol
}
```

เราสามารถ conform type ใดๆ ก็ได้ให้เป็น Error แต่ที่นิยมที่สุดคือ `enum`:

```swift
// Error protocol พื้นฐาน
enum SimpleError: Error {
    case somethingWentWrong
}

// สามารถใช้ struct ก็ได้
struct DatabaseError: Error {
    var message: String
    var code: Int
}

// หรือแม้แต่ class
class NetworkError: Error {
    var statusCode: Int
    var url: String
    
    init(statusCode: Int, url: String) {
        self.statusCode = statusCode
        self.url = url
    }
}
```

---

## 18.2 การกำหนด Custom Errors

### การใช้ Enum (แนะนำ)

```swift
enum ValidationError: Error {
    case emptyField(fieldName: String)
    case tooShort(fieldName: String, minimum: Int)
    case tooLong(fieldName: String, maximum: Int)
    case invalidFormat(fieldName: String, expectedFormat: String)
    case outOfRange(fieldName: String, min: Double, max: Double)
}

enum FileError: Error {
    case fileNotFound(path: String)
    case permissionDenied(path: String)
    case corruptedData(path: String)
    case diskFull
    case unsupportedFormat(format: String)
}

enum NetworkError: Error {
    case noInternet
    case timeout(seconds: Double)
    case serverError(statusCode: Int)
    case invalidURL(url: String)
    case decodingFailed(reason: String)
    case unauthorized
    case rateLimited(retryAfter: Int)
}

enum AuthError: Error {
    case invalidCredentials
    case accountLocked(unlockAt: Date)
    case tokenExpired
    case insufficientPermissions(required: String)
    case twoFactorRequired
}
```

### Error ที่มี Associated Values

```swift
enum ParseError: Error {
    case invalidJSON(line: Int, column: Int)
    case missingField(name: String, inObject: String)
    case typeMismatch(field: String, expected: String, got: String)
    case valueOutOfRange(field: String, value: Any, range: String)
}

// ใช้งาน
func parseAge(_ value: Any) throws -> Int {
    guard let ageString = value as? String else {
        throw ParseError.typeMismatch(
            field: "age",
            expected: "String",
            got: String(describing: type(of: value))
        )
    }
    
    guard let age = Int(ageString) else {
        throw ParseError.invalidJSON(line: 1, column: 1)
    }
    
    guard age >= 0 && age <= 150 else {
        throw ParseError.valueOutOfRange(
            field: "age",
            value: age,
            range: "0-150"
        )
    }
    
    return age
}
```

---

## 18.3 การ Throw Errors

ใช้ keyword `throw` เพื่อ throw error:

```swift
func divide(_ a: Double, by b: Double) throws -> Double {
    guard b != 0 else {
        throw MathError.divisionByZero
    }
    return a / b
}

enum MathError: Error {
    case divisionByZero
    case negativeSquareRoot
    case overflow
    case underflow
}

func squareRoot(_ value: Double) throws -> Double {
    guard value >= 0 else {
        throw MathError.negativeSquareRoot
    }
    return value.squareRoot()
}

// throw หลาย errors ใน function เดียว
func safeMath(a: Double, b: Double) throws -> Double {
    let divided = try divide(a, by: b)
    return try squareRoot(divided)
}
```

---

## 18.4 Throwing Functions

Function ที่สามารถ throw errors ต้องประกาศด้วย `throws`:

```swift
// Function ที่อาจ throw error
func loadFile(at path: String) throws -> String {
    guard !path.isEmpty else {
        throw FileError.fileNotFound(path: path)
    }
    
    // สมมติว่าอ่านไฟล์ได้
    return "file content"
}

// Function ที่ throws และ return optional
func parseNumber(_ string: String) throws -> Double {
    guard let number = Double(string) else {
        throw ParseError.invalidJSON(line: 0, column: 0)
    }
    return number
}

// Throwing closure
let throwingClosure: (String) throws -> Int = { str in
    guard let n = Int(str) else {
        throw ParseError.invalidJSON(line: 0, column: 0)
    }
    return n
}

// Function ที่ return throws function
func makeMultiplier(by factor: Int) -> (Int) throws -> Int {
    return { value in
        let result = value.multipliedReportingOverflow(by: factor)
        guard !result.overflow else {
            throw MathError.overflow
        }
        return result.partialValue
    }
}
```

---

## 18.5 Do-Catch Statement

ใช้ `do-catch` เพื่อจัดการ errors:

```swift
// รูปแบบพื้นฐาน
do {
    let result = try divide(10, by: 2)
    print("ผลลัพธ์: \(result)")
} catch {
    print("เกิดข้อผิดพลาด: \(error)")
}

// ตัวอย่างสมบูรณ์
do {
    let content = try loadFile(at: "/path/to/file.txt")
    let number = try parseNumber("42.5")
    let result = try divide(number, by: 0) // จะ throw error
    print("ผลลัพธ์: \(result)")
} catch FileError.fileNotFound(let path) {
    print("ไม่พบไฟล์: \(path)")
} catch MathError.divisionByZero {
    print("ไม่สามารถหารด้วยศูนย์ได้")
} catch {
    print("เกิดข้อผิดพลาดที่ไม่คาดคิด: \(error)")
}
```

### Catch ทุก Cases ของ Enum Error

```swift
do {
    try validateUserInput(name: "", email: "notanemail", age: 200)
} catch let error as ValidationError {
    switch error {
    case .emptyField(let fieldName):
        print("กรุณากรอก \(fieldName)")
    case .tooShort(let fieldName, let minimum):
        print("\(fieldName) ต้องมีความยาวอย่างน้อย \(minimum) ตัวอักษร")
    case .tooLong(let fieldName, let maximum):
        print("\(fieldName) ต้องมีความยาวไม่เกิน \(maximum) ตัวอักษร")
    case .invalidFormat(let fieldName, let format):
        print("\(fieldName) ต้องมีรูปแบบ: \(format)")
    case .outOfRange(let fieldName, let min, let max):
        print("\(fieldName) ต้องอยู่ระหว่าง \(min) ถึง \(max)")
    }
}

func validateUserInput(name: String, email: String, age: Int) throws {
    if name.isEmpty {
        throw ValidationError.emptyField(fieldName: "ชื่อ")
    }
    if name.count < 2 {
        throw ValidationError.tooShort(fieldName: "ชื่อ", minimum: 2)
    }
    if !email.contains("@") {
        throw ValidationError.invalidFormat(fieldName: "อีเมล", expectedFormat: "xxx@xxx.xxx")
    }
    if age < 0 || age > 150 {
        throw ValidationError.outOfRange(fieldName: "อายุ", min: 0, max: 150)
    }
}
```

---

## 18.6 การ Catch Errors แบบเฉพาะเจาะจง

```swift
// Catch หลาย patterns
do {
    try performNetworkRequest()
} catch NetworkError.noInternet {
    showAlert("ไม่มีการเชื่อมต่ออินเทอร์เน็ต")
} catch NetworkError.timeout(let seconds) {
    showAlert("หมดเวลาการเชื่อมต่อหลังจาก \(seconds) วินาที")
} catch NetworkError.serverError(let code) where code >= 500 {
    showAlert("เซิร์ฟเวอร์มีปัญหา (รหัส \(code))")
} catch NetworkError.serverError(let code) where code == 404 {
    showAlert("ไม่พบทรัพยากรที่ร้องขอ")
} catch NetworkError.unauthorized {
    redirectToLogin()
} catch {
    showAlert("เกิดข้อผิดพลาด: \(error.localizedDescription)")
}

func showAlert(_ message: String) {
    print("⚠️ \(message)")
}

func redirectToLogin() {
    print("🔐 กำลังนำไปหน้า Login...")
}

func performNetworkRequest() throws {
    throw NetworkError.timeout(seconds: 30.0)
}
```

### Catch หลาย Error Types ในคำสั่งเดียว

```swift
do {
    try riskyOperation()
} catch is NetworkError, is FileError {
    print("เกิดปัญหา I/O")
} catch is ValidationError {
    print("ข้อมูลไม่ถูกต้อง")
} catch {
    print("ข้อผิดพลาดที่ไม่รู้จัก: \(error)")
}

func riskyOperation() throws {
    throw FileError.diskFull
}
```

---

## 18.7 Multiple Catch Clauses

```swift
enum AppError: Error {
    case databaseError(String)
    case networkError(String)
    case parseError(String)
    case authError(String)
    case unknownError
}

func performComplexOperation() throws {
    // simulate complex operation
    let random = Int.random(in: 1...5)
    switch random {
    case 1: throw AppError.databaseError("Connection refused")
    case 2: throw AppError.networkError("Timeout")
    case 3: throw AppError.parseError("Invalid JSON")
    case 4: throw AppError.authError("Token expired")
    default: break
    }
    print("สำเร็จ!")
}

// Handle แต่ละ case
func handleOperation() {
    do {
        try performComplexOperation()
        print("การดำเนินการสำเร็จ")
    } catch AppError.databaseError(let message) {
        print("❌ Database Error: \(message)")
        // reconnect to database
    } catch AppError.networkError(let message) {
        print("🌐 Network Error: \(message)")
        // retry after delay
    } catch AppError.parseError(let message) {
        print("📄 Parse Error: \(message)")
        // return default value
    } catch AppError.authError(let message) {
        print("🔐 Auth Error: \(message)")
        // refresh token
    } catch AppError.unknownError {
        print("❓ Unknown Error")
    } catch {
        print("🚫 Unexpected: \(error)")
    }
}

handleOperation()
```

---

## 18.8 Rethrowing Errors (rethrows)

Function ที่ rethrow error คือ function ที่ throw error ก็ต่อเมื่อ closure ที่รับมาเป็น parameter throw error:

```swift
// rethrows ใน function
func performTwice(_ closure: () throws -> Void) rethrows {
    try closure()
    try closure()
}

// เรียกด้วย non-throwing closure - ไม่ต้องใช้ try
performTwice {
    print("สวัสดี!")
}

// เรียกด้วย throwing closure - ต้องใช้ try
do {
    try performTwice {
        throw SimpleError.somethingWentWrong
    }
} catch {
    print("Error: \(error)")
}

// rethrows ใน method ที่มี generic
func transform<T, U>(_ value: T, using closure: (T) throws -> U) rethrows -> U {
    return try closure(value)
}

// ใช้งานโดยไม่ต้อง try เมื่อ closure ไม่ throw
let doubled = transform(5) { $0 * 2 }
print(doubled)  // 10

// ใช้งานกับ throwing closure
do {
    let result = try transform("42") { str -> Int in
        guard let n = Int(str) else {
            throw ParseError.invalidJSON(line: 0, column: 0)
        }
        return n
    }
    print(result)  // 42
} catch {
    print("Error: \(error)")
}
```

### map, filter, forEach กับ rethrows

```swift
let strings = ["1", "2", "three", "4"]

// map throws เมื่อ closure throw
do {
    let numbers = try strings.map { str -> Int in
        guard let n = Int(str) else {
            throw ParseError.invalidJSON(line: 0, column: 0)
        }
        return n
    }
    print(numbers)
} catch {
    print("Parse error: \(error)")  // Parse error
}

// filter ใช้งานได้กับ throwing closure เช่นกัน
do {
    let validNumbers = try strings.filter { str -> Bool in
        guard Int(str) != nil else {
            throw ValidationError.invalidFormat(fieldName: "number", expectedFormat: "integer")
        }
        return true
    }
    print(validNumbers)
} catch {
    print("Error: \(error)")
}
```

---

## 18.9 Converting Errors to Optionals (try?)

ใช้ `try?` เพื่อแปลง error เป็น `nil`:

```swift
// ถ้า throw error - คืน nil
// ถ้าสำเร็จ - คืน Optional ที่ wrap value

let result1 = try? divide(10, by: 2)    // Optional(5.0)
let result2 = try? divide(10, by: 0)    // nil

if let validResult = try? divide(20, by: 4) {
    print("ผลลัพธ์: \(validResult)")  // ผลลัพธ์: 5.0
}

// ใช้กับ guard
func processValue(_ string: String) -> Double {
    guard let value = try? parseNumber(string) else {
        return 0.0  // default value
    }
    return value
}

print(processValue("3.14"))  // 3.14
print(processValue("abc"))   // 0.0

// Nil coalescing กับ try?
let number = (try? parseNumber("42")) ?? 0
print(number)  // 42.0

// ใช้ใน map/filter
let strings2 = ["1", "2", "abc", "4", "xyz", "6"]
let validNumbers = strings2.compactMap { try? parseNumber($0) }
print(validNumbers)  // [1.0, 2.0, 4.0, 6.0]
```

---

## 18.10 Forcing Try (try!)

ใช้ `try!` เมื่อมั่นใจ 100% ว่าจะไม่ throw error (ถ้า throw จะ crash):

```swift
// ใช้เมื่อมั่นใจว่าจะไม่ error
let definitelyValidNumber = try! parseNumber("42.0")
print(definitelyValidNumber)  // 42.0

// กรณีที่ใช้บ่อย: Bundle resource ที่รู้ว่ามีอยู่
// let url = Bundle.main.url(forResource: "config", withExtension: "json")!
// let data = try! Data(contentsOf: url)

// ⚠️ อย่าใช้เมื่อไม่แน่ใจ!
// let crash = try! parseNumber("not-a-number")  // CRASH!

// Pattern ที่ดีกว่า: ใช้ try! เฉพาะกับค่าที่กำหนดไว้แน่นอน
let validURLs = ["https://api.example.com", "https://cdn.example.com"]
let urls = validURLs.map { URL(string: $0)! }  // ปลอดภัยเพราะรู้ว่า valid
```

---

## 18.11 Error Propagation

Error สามารถ propagate ผ่าน call stack:

```swift
// Layer 1: Database
func fetchUserFromDB(id: Int) throws -> User {
    guard id > 0 else {
        throw AppError.databaseError("Invalid ID: \(id)")
    }
    // simulate DB fetch
    return User(id: id, name: "สมชาย", email: "somchai@example.com", age: 30, createdAt: Date())
}

struct User {
    var id: Int
    var name: String
    var email: String
    var age: Int
    var createdAt: Date
}

// Layer 2: Service
func getUserService(id: Int) throws -> User {
    // propagate error จาก fetchUserFromDB
    let user = try fetchUserFromDB(id: id)
    
    // เพิ่ม validation ของตัวเอง
    guard !user.name.isEmpty else {
        throw ValidationError.emptyField(fieldName: "name")
    }
    
    return user
}

// Layer 3: ViewModel / Controller
func loadUserProfile(id: Int) throws -> UserProfile {
    let user = try getUserService(id: id)  // propagate further
    
    return UserProfile(
        displayName: user.name,
        email: user.email,
        joinDate: user.createdAt.formatted(as: "dd/MM/yyyy")
    )
}

struct UserProfile {
    var displayName: String
    var email: String
    var joinDate: String
}

extension Date {
    func formatted(as format: String) -> String {
        let formatter = DateFormatter()
        formatter.dateFormat = format
        return formatter.string(from: self)
    }
}

// Layer 4: View / Entry point - catch ที่นี่
do {
    let profile = try loadUserProfile(id: 1)
    print("แสดงโปรไฟล์: \(profile.displayName)")
} catch AppError.databaseError(let msg) {
    print("DB Error: \(msg)")
} catch ValidationError.emptyField(let field) {
    print("Validation Error: \(field) ว่างเปล่า")
} catch {
    print("Unknown error: \(error)")
}
```

---

## 18.12 Custom Error Messages

### การ Conform to LocalizedError

```swift
enum ShoppingError: Error {
    case outOfStock(item: String)
    case insufficientFunds(required: Double, available: Double)
    case itemNotFound(id: String)
    case invalidQuantity(requested: Int, maximum: Int)
}

extension ShoppingError: LocalizedError {
    var errorDescription: String? {
        switch self {
        case .outOfStock(let item):
            return "สินค้า '\(item)' หมดสต็อก"
        case .insufficientFunds(let required, let available):
            return "ยอดเงินไม่เพียงพอ ต้องการ ฿\(required) แต่มีเพียง ฿\(available)"
        case .itemNotFound(let id):
            return "ไม่พบสินค้า ID: \(id)"
        case .invalidQuantity(let requested, let maximum):
            return "จำนวนที่ขอ (\(requested)) เกินกว่าที่มีในสต็อก (\(maximum))"
        }
    }
    
    var failureReason: String? {
        switch self {
        case .outOfStock:
            return "สินค้าหมดจากคลัง"
        case .insufficientFunds:
            return "ยอดเงินในบัญชีไม่เพียงพอ"
        case .itemNotFound:
            return "ไม่มีสินค้าในระบบ"
        case .invalidQuantity:
            return "จำนวนสินค้าในสต็อกไม่พอ"
        }
    }
    
    var recoverySuggestion: String? {
        switch self {
        case .outOfStock:
            return "กรุณารอสินค้าเข้าสต็อก หรือเลือกสินค้าอื่น"
        case .insufficientFunds:
            return "กรุณาเติมเงินในบัญชี"
        case .itemNotFound:
            return "กรุณาตรวจสอบ ID สินค้าอีกครั้ง"
        case .invalidQuantity(_, let max):
            return "กรุณาสั่งซื้อไม่เกิน \(max) ชิ้น"
        }
    }
}

func addToCart(itemId: String, quantity: Int) throws {
    guard quantity > 0 && quantity <= 10 else {
        throw ShoppingError.invalidQuantity(requested: quantity, maximum: 10)
    }
    // ...
}

do {
    try addToCart(itemId: "ITEM001", quantity: 15)
} catch let error as ShoppingError {
    print("❌ \(error.errorDescription ?? "")")
    print("เหตุผล: \(error.failureReason ?? "")")
    print("แนะนำ: \(error.recoverySuggestion ?? "")")
}
// ❌ จำนวนที่ขอ (15) เกินกว่าที่มีในสต็อก (10)
// เหตุผล: จำนวนสินค้าในสต็อกไม่พอ
// แนะนำ: กรุณาสั่งซื้อไม่เกิน 10 ชิ้น
```

---

## 18.13 Localized Errors

```swift
enum SystemError: LocalizedError {
    case fileNotFound(name: String)
    case networkUnavailable
    case permissionDenied(resource: String)
    
    var errorDescription: String? {
        // ใช้ Localizable.strings ในโปรเจคจริง
        // NSLocalizedString("error.file_not_found", comment: "")
        switch self {
        case .fileNotFound(let name):
            return String(format: "ไม่พบไฟล์ '%@'", name)
        case .networkUnavailable:
            return "ไม่สามารถเชื่อมต่อเครือข่ายได้"
        case .permissionDenied(let resource):
            return String(format: "ไม่มีสิทธิ์เข้าถึง '%@'", resource)
        }
    }
    
    var helpAnchor: String? {
        switch self {
        case .fileNotFound:
            return "help://file-not-found"
        case .networkUnavailable:
            return "help://network-issues"
        case .permissionDenied:
            return "help://permissions"
        }
    }
}

// ใช้งาน localized error
let err = SystemError.fileNotFound(name: "config.json")
print(err.localizedDescription)  // ไม่พบไฟล์ 'config.json'
```

---

## 18.14 NSError Bridging

Swift errors ทำงานร่วมกับ NSError ได้:

```swift
import Foundation

// Swift Error จะถูก bridge เป็น NSError อัตโนมัติ
enum AppErrorBridge: Error {
    case dataCorrupted
    case invalidInput(String)
}

func demonstrateBridging() {
    let swiftError: Error = AppErrorBridge.dataCorrupted
    
    // แปลงเป็น NSError
    let nsError = swiftError as NSError
    print("Domain: \(nsError.domain)")   // AppErrorBridge
    print("Code: \(nsError.code)")       // 0
    
    // แปลงกลับ
    if let appError = nsError as? AppErrorBridge {
        print("Swift Error: \(appError)")
    }
}

demonstrateBridging()

// สร้าง NSError ด้วยตัวเอง
extension NSError {
    static func appError(
        domain: String = "com.myapp.error",
        code: Int,
        description: String,
        suggestion: String? = nil
    ) -> NSError {
        var userInfo: [String: Any] = [
            NSLocalizedDescriptionKey: description
        ]
        if let suggestion = suggestion {
            userInfo[NSLocalizedRecoverySuggestionErrorKey] = suggestion
        }
        return NSError(domain: domain, code: code, userInfo: userInfo)
    }
}

let customNSError = NSError.appError(
    code: 1001,
    description: "ไม่สามารถบันทึกข้อมูลได้",
    suggestion: "กรุณาตรวจสอบพื้นที่จัดเก็บข้อมูล"
)
print(customNSError.localizedDescription)
```

---

## 18.15 การจัดการ Errors ใน Async Code

### Async/Await กับ Errors

```swift
import Foundation

// Async function ที่ throw
func fetchUser(id: Int) async throws -> User {
    let url = URL(string: "https://api.example.com/users/\(id)")!
    
    // URLSession.data(from:) throws URLError
    let (data, response) = try await URLSession.shared.data(from: url)
    
    guard let httpResponse = response as? HTTPURLResponse else {
        throw NetworkError.serverError(statusCode: 0)
    }
    
    guard httpResponse.statusCode == 200 else {
        switch httpResponse.statusCode {
        case 401: throw NetworkError.unauthorized
        case 404: throw NetworkError.serverError(statusCode: 404)
        default: throw NetworkError.serverError(statusCode: httpResponse.statusCode)
        }
    }
    
    do {
        let user = try JSONDecoder().decode(User.self, from: data)
        return user
    } catch {
        throw NetworkError.decodingFailed(reason: error.localizedDescription)
    }
}

// เรียกใช้ async throwing function
func loadProfile() async {
    do {
        let user = try await fetchUser(id: 1)
        print("โหลด User: \(user.name)")
    } catch NetworkError.unauthorized {
        print("กรุณา login ก่อน")
    } catch NetworkError.serverError(let code) {
        print("Server error: \(code)")
    } catch NetworkError.decodingFailed(let reason) {
        print("Decode error: \(reason)")
    } catch {
        print("Error: \(error)")
    }
}

// Task group กับ errors
func fetchMultipleUsers(ids: [Int]) async throws -> [User] {
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

### Structured Concurrency Error Handling

```swift
// Task ที่อาจ fail
func processData() async {
    let task = Task<String, Error> {
        // simulate work that might fail
        try await Task.sleep(nanoseconds: 1_000_000)
        if Bool.random() {
            throw AppError.databaseError("Simulated failure")
        }
        return "ข้อมูลสำเร็จ"
    }
    
    do {
        let result = try await task.value
        print("Result: \(result)")
    } catch {
        print("Task failed: \(error)")
    }
}

// ใช้ async let กับ error handling
func fetchUserData(id: Int) async throws -> (User, [String]) {
    async let user = fetchUser(id: id)
    async let permissions = fetchPermissions(userId: id)
    
    // ทั้งสองรอพร้อมกัน ถ้า error จะ throw
    return try await (user, permissions)
}

func fetchPermissions(userId: Int) async throws -> [String] {
    return ["read", "write"]
}
```

---

## 18.16 Result Type สำหรับ Errors

`Result<Success, Failure>` เป็น enum ที่แทน success หรือ failure:

```swift
// นิยามของ Result
// enum Result<Success, Failure: Error> {
//     case success(Success)
//     case failure(Failure)
// }

// สร้าง function ที่คืน Result
func divideWithResult(_ a: Double, by b: Double) -> Result<Double, MathError> {
    guard b != 0 else {
        return .failure(.divisionByZero)
    }
    return .success(a / b)
}

// ใช้งาน Result
let result = divideWithResult(10, by: 2)
switch result {
case .success(let value):
    print("ผลลัพธ์: \(value)")
case .failure(let error):
    print("Error: \(error)")
}

// map, flatMap บน Result
let result2 = divideWithResult(10, by: 2)
    .map { $0 * 2 }  // แปลง success value
    
print(result2)  // success(10.0)

// flatMap
func loadData() -> Result<Data, NetworkError> {
    // simulate
    return .success(Data())
}

func parseData(_ data: Data) -> Result<User, NetworkError> {
    // simulate
    return .success(User(id: 1, name: "Test", email: "test@test.com", age: 25, createdAt: Date()))
}

let userResult = loadData().flatMap { parseData($0) }

// get() - แปลง Result เป็น throws
do {
    let value = try result.get()
    print("Value: \(value)")
} catch {
    print("Error: \(error)")
}

// Completion handlers กับ Result
func fetchUserCompletion(id: Int, completion: @escaping (Result<User, NetworkError>) -> Void) {
    // simulate async operation
    DispatchQueue.global().async {
        if id > 0 {
            let user = User(id: id, name: "สมหมาย", email: "test@test.com", age: 25, createdAt: Date())
            completion(.success(user))
        } else {
            completion(.failure(.serverError(statusCode: 400)))
        }
    }
}

// เรียกใช้
fetchUserCompletion(id: 1) { result in
    switch result {
    case .success(let user):
        print("User: \(user.name)")
    case .failure(let error):
        print("Failed: \(error)")
    }
}
```

### Result Chaining

```swift
func validateEmail(_ email: String) -> Result<String, ValidationError> {
    guard !email.isEmpty else {
        return .failure(.emptyField(fieldName: "email"))
    }
    guard email.contains("@") else {
        return .failure(.invalidFormat(fieldName: "email", expectedFormat: "xxx@xxx.xxx"))
    }
    return .success(email)
}

func validatePassword(_ password: String) -> Result<String, ValidationError> {
    guard password.count >= 8 else {
        return .failure(.tooShort(fieldName: "password", minimum: 8))
    }
    return .success(password)
}

func registerUser(email: String, password: String) -> Result<String, ValidationError> {
    return validateEmail(email)
        .flatMap { validEmail in
            validatePassword(password)
                .map { _ in "ลงทะเบียนสำเร็จ: \(validEmail)" }
        }
}

let registrationResult = registerUser(email: "user@example.com", password: "securepass123")
switch registrationResult {
case .success(let message):
    print(message)
case .failure(let error):
    print("Error: \(error)")
}
```

---

## 18.17 Typed Throws (Swift 6)

Swift 6 เพิ่ม typed throws ที่ให้ระบุ error type ที่แน่นอน:

```swift
// Swift 6: Typed throws
enum DatabaseError: Error {
    case connectionFailed
    case queryFailed(reason: String)
    case noResults
}

// ระบุ error type ที่แน่นอน
func queryDatabase(sql: String) throws(DatabaseError) -> [String] {
    guard sql.hasPrefix("SELECT") else {
        throw DatabaseError.queryFailed(reason: "ต้องใช้ SELECT statement เท่านั้น")
    }
    
    if sql.contains("FROM empty_table") {
        throw DatabaseError.noResults
    }
    
    return ["result1", "result2"]
}

// catch โดยไม่ต้องระบุ type เพราะ compiler รู้แล้ว
do {
    let results = try queryDatabase(sql: "SELECT * FROM users")
    print(results)
} catch .connectionFailed {
    print("เชื่อมต่อ database ไม่ได้")
} catch .queryFailed(let reason) {
    print("Query ผิดพลาด: \(reason)")
} catch .noResults {
    print("ไม่มีผลลัพธ์")
}

// Typed throws กับ generic
func transform<T, E: Error>(_ value: T, using closure: (T) throws(E) -> T) throws(E) -> T {
    return try closure(value)
}

// Never throws = non-throwing
func neverThrows(_ x: Int) throws(Never) -> Int {
    return x * 2  // ไม่มี throw statement
}

// สามารถเรียกโดยไม่ต้องใช้ try
let result3 = try neverThrows(5)  // ยังต้องใช้ try แต่ compiler รู้ว่าไม่ throw
print(result3)  // 10
```

---

## 18.18 Error Handling Best Practices

### 1. ออกแบบ Error Hierarchy

```swift
// Error hierarchy ที่ดี
protocol AppErrorProtocol: Error {
    var code: Int { get }
    var message: String { get }
    var isRecoverable: Bool { get }
}

enum DataError: AppErrorProtocol {
    case notFound(id: String)
    case invalidFormat(String)
    case conflict(String)
    
    var code: Int {
        switch self {
        case .notFound: return 1001
        case .invalidFormat: return 1002
        case .conflict: return 1003
        }
    }
    
    var message: String {
        switch self {
        case .notFound(let id): return "ไม่พบข้อมูล ID: \(id)"
        case .invalidFormat(let detail): return "รูปแบบข้อมูลไม่ถูกต้อง: \(detail)"
        case .conflict(let detail): return "ข้อมูลขัดแย้ง: \(detail)"
        }
    }
    
    var isRecoverable: Bool {
        switch self {
        case .notFound: return false
        case .invalidFormat: return true  // user can correct input
        case .conflict: return true
        }
    }
}
```

### 2. ไม่ Swallow Errors

```swift
// ❌ BAD: กลืน error
func badPractice() {
    let result = try? riskyOperation()
    // ไม่รู้เลยว่า error อะไรเกิดขึ้น
}

// ✅ GOOD: จัดการ error อย่างเหมาะสม
func goodPractice() {
    do {
        try riskyOperation()
    } catch {
        // log error
        print("Error occurred: \(error)")
        // หรือ show to user
        // หรือ retry
        // หรือ use fallback
    }
}
```

### 3. Error Context

```swift
// เพิ่ม context ให้กับ error
struct ContextualError: Error {
    let underlying: Error
    let context: String
    let file: String
    let line: Int
    
    init(_ underlying: Error, context: String, file: String = #file, line: Int = #line) {
        self.underlying = underlying
        self.context = context
        self.file = file
        self.line = line
    }
}

func loadConfiguration() throws -> [String: String] {
    do {
        let data = try Data(contentsOf: URL(fileURLWithPath: "/config.json"))
        return try JSONDecoder().decode([String: String].self, from: data)
    } catch {
        throw ContextualError(error, context: "การโหลด configuration file")
    }
}
```

### 4. Fail Fast

```swift
// Validate ก่อนทำงาน
func processOrder(items: [String], userID: Int) throws {
    // Validate ทันที
    guard !items.isEmpty else {
        throw ShoppingError.outOfStock(item: "รายการสั่งซื้อ")
    }
    guard userID > 0 else {
        throw AppError.authError("Invalid user ID")
    }
    
    // ทำงานหลักที่นี่
    print("กำลัง process order สำหรับ user \(userID)")
}
```

---

## 18.19 Error Logging Strategies

### Simple Logger

```swift
import Foundation

enum LogLevel: Int, Comparable {
    case debug = 0
    case info = 1
    case warning = 2
    case error = 3
    case critical = 4
    
    static func < (lhs: LogLevel, rhs: LogLevel) -> Bool {
        return lhs.rawValue < rhs.rawValue
    }
    
    var emoji: String {
        switch self {
        case .debug: return "🔍"
        case .info: return "ℹ️"
        case .warning: return "⚠️"
        case .error: return "❌"
        case .critical: return "🚨"
        }
    }
}

class ErrorLogger {
    static let shared = ErrorLogger()
    private var minimumLevel: LogLevel = .debug
    private var logs: [(Date, LogLevel, String, Error?)] = []
    
    private init() {}
    
    func log(
        _ message: String,
        level: LogLevel = .info,
        error: Error? = nil,
        file: String = #file,
        function: String = #function,
        line: Int = #line
    ) {
        guard level >= minimumLevel else { return }
        
        let timestamp = Date()
        let fileName = URL(fileURLWithPath: file).lastPathComponent
        
        var logMessage = "\(level.emoji) [\(level)] \(timestamp.formatted(as: "HH:mm:ss")) [\(fileName):\(line)] \(message)"
        
        if let error = error {
            logMessage += "\n   Error: \(error.localizedDescription)"
        }
        
        print(logMessage)
        logs.append((timestamp, level, message, error))
    }
    
    func logError(_ error: Error, context: String = "", file: String = #file, line: Int = #line) {
        log(
            context.isEmpty ? "Error occurred" : context,
            level: .error,
            error: error,
            file: file,
            line: line
        )
    }
    
    func setMinimumLevel(_ level: LogLevel) {
        minimumLevel = level
    }
    
    func getRecentLogs(count: Int = 10) -> [(Date, LogLevel, String, Error?)] {
        return Array(logs.suffix(count))
    }
}

// Usage
let logger = ErrorLogger.shared

do {
    try riskyOperation()
} catch {
    logger.logError(error, context: "ขณะดำเนินการ risky operation")
}

logger.log("เริ่มต้นแอปพลิเคชัน", level: .info)
logger.log("กำลัง debug connection", level: .debug)
logger.log("memory ใกล้เต็ม", level: .warning)
```

### Structured Error Reporting

```swift
struct ErrorReport {
    var error: Error
    var timestamp: Date
    var userID: String?
    var sessionID: String
    var context: [String: Any]
    var stackTrace: String
    
    init(error: Error, userID: String? = nil, context: [String: Any] = [:]) {
        self.error = error
        self.timestamp = Date()
        self.userID = userID
        self.sessionID = UUID().uuidString
        self.context = context
        self.stackTrace = Thread.callStackSymbols.joined(separator: "\n")
    }
    
    func toDictionary() -> [String: Any] {
        return [
            "error": error.localizedDescription,
            "timestamp": timestamp.timeIntervalSince1970,
            "userID": userID ?? "anonymous",
            "sessionID": sessionID,
            "context": context
        ]
    }
}

class ErrorReporter {
    func report(_ error: Error, context: [String: Any] = [:]) {
        let report = ErrorReport(error: error, context: context)
        // ส่งไปยัง error tracking service เช่น Sentry, Firebase Crashlytics
        sendToAnalytics(report)
    }
    
    private func sendToAnalytics(_ report: ErrorReport) {
        print("📊 Sending error report...")
        print(report.toDictionary())
    }
}
```

---

## 18.20 แบบฝึกหัด

### แบบฝึกหัดที่ 1: File System Operations

```swift
import Foundation

enum FileSystemError: LocalizedError {
    case fileNotFound(path: String)
    case directoryNotFound(path: String)
    case insufficientPermissions(path: String)
    case diskFull(requiredBytes: Int64)
    case invalidEncoding
    case fileAlreadyExists(path: String)
    
    var errorDescription: String? {
        switch self {
        case .fileNotFound(let path):
            return "ไม่พบไฟล์: \(path)"
        case .directoryNotFound(let path):
            return "ไม่พบโฟลเดอร์: \(path)"
        case .insufficientPermissions(let path):
            return "ไม่มีสิทธิ์เข้าถึง: \(path)"
        case .diskFull(let bytes):
            return "พื้นที่ไม่เพียงพอ ต้องการอีก \(bytes) bytes"
        case .invalidEncoding:
            return "Encoding ไม่ถูกต้อง"
        case .fileAlreadyExists(let path):
            return "ไฟล์มีอยู่แล้ว: \(path)"
        }
    }
    
    var recoverySuggestion: String? {
        switch self {
        case .fileNotFound:
            return "ตรวจสอบ path ให้ถูกต้อง"
        case .directoryNotFound:
            return "สร้างโฟลเดอร์ก่อน"
        case .insufficientPermissions:
            return "ขอสิทธิ์การเข้าถึงจากผู้ดูแลระบบ"
        case .diskFull:
            return "ลบไฟล์ที่ไม่จำเป็นออก"
        case .invalidEncoding:
            return "ใช้ UTF-8 encoding"
        case .fileAlreadyExists:
            return "ลบไฟล์เดิมก่อน หรือใช้ชื่อใหม่"
        }
    }
}

class FileManager2 {
    private let basePath: String
    private var files: [String: String] = [:]  // simulate file system
    
    init(basePath: String = "/tmp") {
        self.basePath = basePath
    }
    
    func createFile(at path: String, content: String) throws {
        guard !files.keys.contains(path) else {
            throw FileSystemError.fileAlreadyExists(path: path)
        }
        files[path] = content
        print("✅ สร้างไฟล์: \(path)")
    }
    
    func readFile(at path: String) throws -> String {
        guard let content = files[path] else {
            throw FileSystemError.fileNotFound(path: path)
        }
        return content
    }
    
    func deleteFile(at path: String) throws {
        guard files[path] != nil else {
            throw FileSystemError.fileNotFound(path: path)
        }
        files.removeValue(forKey: path)
        print("🗑 ลบไฟล์: \(path)")
    }
    
    func copyFile(from source: String, to destination: String) throws {
        let content = try readFile(at: source)
        try createFile(at: destination, content: content)
        print("📋 คัดลอกจาก \(source) ไป \(destination)")
    }
    
    func moveFile(from source: String, to destination: String) throws {
        try copyFile(from: source, to: destination)
        try deleteFile(at: source)
        print("📦 ย้ายจาก \(source) ไป \(destination)")
    }
}

// ทดสอบ
let fm = FileManager2()

do {
    try fm.createFile(at: "/tmp/test.txt", content: "สวัสดีโลก")
    let content = try fm.readFile(at: "/tmp/test.txt")
    print("อ่านได้: \(content)")
    
    try fm.copyFile(from: "/tmp/test.txt", to: "/tmp/test_copy.txt")
    try fm.moveFile(from: "/tmp/test_copy.txt", to: "/tmp/test_moved.txt")
    
    // จะ throw error
    try fm.readFile(at: "/tmp/test_copy.txt")
} catch let error as FileSystemError {
    print("❌ \(error.errorDescription ?? "")")
    if let suggestion = error.recoverySuggestion {
        print("💡 \(suggestion)")
    }
} catch {
    print("Unknown error: \(error)")
}
```

### แบบฝึกหัดที่ 2: JSON Parsing

```swift
import Foundation

enum JSONParseError: Error {
    case invalidJSON
    case missingField(String)
    case invalidType(field: String, expected: String, got: String)
    case invalidValue(field: String, reason: String)
}

extension JSONParseError: LocalizedError {
    var errorDescription: String? {
        switch self {
        case .invalidJSON:
            return "JSON format ไม่ถูกต้อง"
        case .missingField(let field):
            return "ไม่พบ field '\(field)' ที่จำเป็น"
        case .invalidType(let field, let expected, let got):
            return "Field '\(field)' ต้องเป็น \(expected) แต่ได้รับ \(got)"
        case .invalidValue(let field, let reason):
            return "ค่าของ '\(field)' ไม่ถูกต้อง: \(reason)"
        }
    }
}

struct UserJSON {
    var id: Int
    var name: String
    var email: String
    var age: Int
    
    init(from json: [String: Any]) throws {
        // Parse id
        guard let id = json["id"] as? Int else {
            if json["id"] == nil {
                throw JSONParseError.missingField("id")
            }
            throw JSONParseError.invalidType(
                field: "id",
                expected: "Int",
                got: String(describing: type(of: json["id"]!))
            )
        }
        
        // Parse name
        guard let name = json["name"] as? String else {
            if json["name"] == nil {
                throw JSONParseError.missingField("name")
            }
            throw JSONParseError.invalidType(field: "name", expected: "String", got: "Unknown")
        }
        guard !name.isEmpty else {
            throw JSONParseError.invalidValue(field: "name", reason: "ชื่อต้องไม่ว่างเปล่า")
        }
        
        // Parse email
        guard let email = json["email"] as? String else {
            if json["email"] == nil {
                throw JSONParseError.missingField("email")
            }
            throw JSONParseError.invalidType(field: "email", expected: "String", got: "Unknown")
        }
        guard email.contains("@") else {
            throw JSONParseError.invalidValue(field: "email", reason: "รูปแบบอีเมลไม่ถูกต้อง")
        }
        
        // Parse age
        guard let age = json["age"] as? Int else {
            if json["age"] == nil {
                throw JSONParseError.missingField("age")
            }
            throw JSONParseError.invalidType(field: "age", expected: "Int", got: "Unknown")
        }
        guard age >= 0 && age <= 150 else {
            throw JSONParseError.invalidValue(field: "age", reason: "อายุต้องอยู่ระหว่าง 0-150")
        }
        
        self.id = id
        self.name = name
        self.email = email
        self.age = age
    }
}

// ทดสอบ
let validJSON: [String: Any] = [
    "id": 1,
    "name": "สมชาย",
    "email": "somchai@example.com",
    "age": 30
]

let invalidJSON1: [String: Any] = [
    "id": 1,
    "name": "",  // empty name
    "email": "somchai@example.com",
    "age": 30
]

let invalidJSON2: [String: Any] = [
    "id": 1,
    "name": "สมชาย",
    // missing email
    "age": 30
]

func parseUserJSON(_ json: [String: Any]) {
    do {
        let user = try UserJSON(from: json)
        print("✅ Parse สำเร็จ: \(user.name) (\(user.email))")
    } catch let error as JSONParseError {
        print("❌ \(error.errorDescription ?? "")")
    } catch {
        print("Unknown error: \(error)")
    }
}

parseUserJSON(validJSON)   // ✅ Parse สำเร็จ
parseUserJSON(invalidJSON1) // ❌ ชื่อต้องไม่ว่างเปล่า
parseUserJSON(invalidJSON2) // ❌ ไม่พบ field 'email'
```

### แบบฝึกหัดที่ 3: Network Error Handling

```swift
import Foundation

// Simulated network layer
struct APIClient {
    let baseURL: String
    var timeout: Double = 30.0
    
    func get<T: Decodable>(endpoint: String, type: T.Type) async throws -> T {
        // Validate URL
        guard let url = URL(string: baseURL + endpoint) else {
            throw NetworkError.invalidURL(url: baseURL + endpoint)
        }
        
        var request = URLRequest(url: url)
        request.timeoutInterval = timeout
        
        do {
            let (data, response) = try await URLSession.shared.data(for: request)
            
            guard let httpResponse = response as? HTTPURLResponse else {
                throw NetworkError.serverError(statusCode: 0)
            }
            
            switch httpResponse.statusCode {
            case 200...299:
                break
            case 401:
                throw NetworkError.unauthorized
            case 429:
                let retryAfter = Int(httpResponse.value(forHTTPHeaderField: "Retry-After") ?? "60") ?? 60
                throw NetworkError.rateLimited(retryAfter: retryAfter)
            default:
                throw NetworkError.serverError(statusCode: httpResponse.statusCode)
            }
            
            do {
                return try JSONDecoder().decode(T.self, from: data)
            } catch {
                throw NetworkError.decodingFailed(reason: error.localizedDescription)
            }
        } catch let error as URLError {
            switch error.code {
            case .notConnectedToInternet, .networkConnectionLost:
                throw NetworkError.noInternet
            case .timedOut:
                throw NetworkError.timeout(seconds: timeout)
            default:
                throw error
            }
        }
    }
}

// ใช้งาน
struct UserResponse: Decodable {
    var id: Int
    var name: String
}

let client = APIClient(baseURL: "https://jsonplaceholder.typicode.com")

Task {
    do {
        let user = try await client.get(endpoint: "/users/1", type: UserResponse.self)
        print("User: \(user.name)")
    } catch NetworkError.noInternet {
        print("ไม่มีอินเทอร์เน็ต")
    } catch NetworkError.timeout(let seconds) {
        print("Timeout หลังจาก \(seconds) วินาที")
    } catch NetworkError.unauthorized {
        print("กรุณา login ก่อน")
    } catch NetworkError.decodingFailed(let reason) {
        print("Decode ล้มเหลว: \(reason)")
    } catch NetworkError.serverError(let code) {
        print("Server error: \(code)")
    } catch {
        print("Error: \(error)")
    }
}
```

---

## 18.21 ตัวอย่าง Real-World: Banking App

```swift
import Foundation

// MARK: - Banking Errors

enum BankingError: Error {
    case accountNotFound(number: String)
    case insufficientFunds(required: Double, available: Double)
    case dailyLimitExceeded(limit: Double, attempted: Double)
    case transactionFailed(reason: String)
    case accountFrozen(reason: String)
    case invalidAmount(Double)
    case recipientNotFound(number: String)
    case sameAccountTransfer
}

extension BankingError: LocalizedError {
    var errorDescription: String? {
        switch self {
        case .accountNotFound(let number):
            return "ไม่พบบัญชีหมายเลข \(number)"
        case .insufficientFunds(let required, let available):
            return "ยอดเงินไม่เพียงพอ (ต้องการ: \(required) บาท, มี: \(available) บาท)"
        case .dailyLimitExceeded(let limit, let attempted):
            return "เกินวงเงินโอนต่อวัน (วงเงิน: \(limit) บาท, ต้องการโอน: \(attempted) บาท)"
        case .transactionFailed(let reason):
            return "ธุรกรรมล้มเหลว: \(reason)"
        case .accountFrozen(let reason):
            return "บัญชีถูกระงับ: \(reason)"
        case .invalidAmount(let amount):
            return "จำนวนเงินไม่ถูกต้อง: \(amount) บาท"
        case .recipientNotFound(let number):
            return "ไม่พบบัญชีผู้รับหมายเลข \(number)"
        case .sameAccountTransfer:
            return "ไม่สามารถโอนเงินไปยังบัญชีเดียวกัน"
        }
    }
    
    var failureReason: String? {
        switch self {
        case .insufficientFunds:
            return "ยอดเงินในบัญชีไม่เพียงพอสำหรับธุรกรรมนี้"
        case .accountFrozen:
            return "บัญชีถูกระงับโดยธนาคาร"
        case .dailyLimitExceeded:
            return "เกินกว่าวงเงินโอนที่กำหนดต่อวัน"
        default:
            return nil
        }
    }
    
    var recoverySuggestion: String? {
        switch self {
        case .insufficientFunds:
            return "กรุณาเติมเงินในบัญชีก่อนทำรายการ"
        case .dailyLimitExceeded:
            return "รอถึงวันถัดไป หรือติดต่อธนาคารเพื่อเพิ่มวงเงิน"
        case .accountFrozen:
            return "ติดต่อธนาคารเพื่อแก้ไขสถานะบัญชี"
        case .invalidAmount:
            return "กรุณากรอกจำนวนเงินที่ถูกต้อง (มากกว่า 0)"
        case .sameAccountTransfer:
            return "กรุณาเลือกบัญชีปลายทางที่แตกต่างกัน"
        default:
            return nil
        }
    }
}

// MARK: - Bank Account

class BankAccount {
    let number: String
    var balance: Double
    var dailyTransferTotal: Double = 0
    let dailyTransferLimit: Double = 100_000
    var isFrozen: Bool = false
    var frozenReason: String?
    var transactions: [Transaction] = []
    
    struct Transaction {
        var id: String
        var type: TransactionType
        var amount: Double
        var timestamp: Date
        var description: String
        
        enum TransactionType {
            case deposit, withdrawal, transfer
        }
    }
    
    init(number: String, initialBalance: Double = 0) {
        self.number = number
        self.balance = initialBalance
    }
    
    func deposit(_ amount: Double) throws {
        guard amount > 0 else {
            throw BankingError.invalidAmount(amount)
        }
        guard !isFrozen else {
            throw BankingError.accountFrozen(reason: frozenReason ?? "ไม่ทราบสาเหตุ")
        }
        
        balance += amount
        
        let transaction = Transaction(
            id: UUID().uuidString,
            type: .deposit,
            amount: amount,
            timestamp: Date(),
            description: "ฝากเงิน"
        )
        transactions.append(transaction)
        
        print("✅ ฝากเงิน \(amount) บาท ยอดคงเหลือ: \(balance) บาท")
    }
    
    func withdraw(_ amount: Double) throws {
        guard amount > 0 else {
            throw BankingError.invalidAmount(amount)
        }
        guard !isFrozen else {
            throw BankingError.accountFrozen(reason: frozenReason ?? "ไม่ทราบสาเหตุ")
        }
        guard balance >= amount else {
            throw BankingError.insufficientFunds(required: amount, available: balance)
        }
        
        balance -= amount
        
        let transaction = Transaction(
            id: UUID().uuidString,
            type: .withdrawal,
            amount: amount,
            timestamp: Date(),
            description: "ถอนเงิน"
        )
        transactions.append(transaction)
        
        print("✅ ถอนเงิน \(amount) บาท ยอดคงเหลือ: \(balance) บาท")
    }
    
    func transfer(amount: Double, to recipient: BankAccount) throws {
        guard amount > 0 else {
            throw BankingError.invalidAmount(amount)
        }
        guard !isFrozen else {
            throw BankingError.accountFrozen(reason: frozenReason ?? "ไม่ทราบสาเหตุ")
        }
        guard number != recipient.number else {
            throw BankingError.sameAccountTransfer
        }
        guard balance >= amount else {
            throw BankingError.insufficientFunds(required: amount, available: balance)
        }
        guard dailyTransferTotal + amount <= dailyTransferLimit else {
            throw BankingError.dailyLimitExceeded(
                limit: dailyTransferLimit,
                attempted: dailyTransferTotal + amount
            )
        }
        
        // Atomic transfer
        balance -= amount
        recipient.balance += amount
        dailyTransferTotal += amount
        
        let transaction = Transaction(
            id: UUID().uuidString,
            type: .transfer,
            amount: amount,
            timestamp: Date(),
            description: "โอนเงินไปบัญชี \(recipient.number)"
        )
        transactions.append(transaction)
        
        print("✅ โอนเงิน \(amount) บาท ไปบัญชี \(recipient.number)")
        print("   ยอดคงเหลือของคุณ: \(balance) บาท")
    }
}

// ทดสอบ
let account1 = BankAccount(number: "123-456-7890", initialBalance: 50_000)
let account2 = BankAccount(number: "987-654-3210", initialBalance: 10_000)

do {
    try account1.deposit(20_000)
    try account1.withdraw(5_000)
    try account1.transfer(amount: 15_000, to: account2)
    
    // ทดสอบ error cases
    try account1.withdraw(1_000_000)  // จะ throw insufficientFunds
} catch let error as BankingError {
    print("\n❌ เกิดข้อผิดพลาด:")
    print("   \(error.errorDescription ?? "")")
    if let reason = error.failureReason {
        print("   เหตุผล: \(reason)")
    }
    if let suggestion = error.recoverySuggestion {
        print("   แนะนำ: \(suggestion)")
    }
}
```

---

## 18.22 Error Handling กับ Combine (Reactive)

```swift
import Combine
import Foundation

// Publisher ที่อาจ fail
struct UserService {
    func fetchUser(id: Int) -> AnyPublisher<User, NetworkError> {
        let url = URL(string: "https://api.example.com/users/\(id)")!
        
        return URLSession.shared.dataTaskPublisher(for: url)
            .tryMap { data, response -> Data in
                guard let httpResponse = response as? HTTPURLResponse,
                      httpResponse.statusCode == 200 else {
                    throw NetworkError.serverError(statusCode: 500)
                }
                return data
            }
            .decode(type: User.self, decoder: JSONDecoder())
            .mapError { error -> NetworkError in
                if let networkError = error as? NetworkError {
                    return networkError
                }
                return NetworkError.decodingFailed(reason: error.localizedDescription)
            }
            .eraseToAnyPublisher()
    }
}

// ใช้งาน
var cancellables = Set<AnyCancellable>()

let userService = UserService()
userService.fetchUser(id: 1)
    .sink(
        receiveCompletion: { completion in
            switch completion {
            case .finished:
                print("โหลดสำเร็จ")
            case .failure(let error):
                print("Error: \(error)")
            }
        },
        receiveValue: { user in
            print("User: \(user.name)")
        }
    )
    .store(in: &cancellables)
```

---

## 18.23 สรุป

Error handling ใน Swift ถูกออกแบบให้:

1. **ชัดเจน** - functions ที่อาจ throw errors ต้องประกาศ `throws`
2. **ปลอดภัย** - compiler บังคับให้จัดการ errors
3. **ยืดหยุ่น** - มีหลายวิธีในการจัดการ errors

### สรุป Patterns หลัก:

| Pattern | ใช้เมื่อ |
|---------|---------|
| `do-catch` | ต้องการจัดการ error อย่างละเอียด |
| `try?` | error ที่ไม่สำคัญ หรือ fallback ได้ |
| `try!` | มั่นใจ 100% ว่าไม่ error |
| `Result<T,E>` | ส่ง error ผ่าน closure / callback |
| `throws` | function ปกติที่อาจ throw |
| `async throws` | async function ที่อาจ throw |
| `rethrows` | function ที่ forward error จาก closure |

### Best Practices:

1. **ออกแบบ Error Hierarchy** ที่ชัดเจนและมีความหมาย
2. **ใช้ LocalizedError** เพื่อ user-friendly messages
3. **อย่า Swallow Errors** - จัดการหรือ propagate เสมอ
4. **Log Errors** อย่างเหมาะสมสำหรับการ debugging
5. **Fail Fast** - validate input ก่อนทำงาน
6. **ให้ Context** กับ errors เพื่อง่ายต่อการ debug
7. **ใช้ typed throws** ใน Swift 6 เมื่อรู้ error type แน่นอน

---

*ต่อไปใน Part 19: Generics - การเขียน Code ที่ยืดหยุ่น*
