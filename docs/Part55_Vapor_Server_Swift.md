# Part 55: Vapor - Server-Side Swift

## บทนำ

Server-Side Swift เป็นการนำภาษา Swift ซึ่งเป็นที่รู้จักในฐานะภาษาสำหรับพัฒนา iOS/macOS มาใช้ในการพัฒนา Backend Server ซึ่งเปิดโอกาสให้นักพัฒนา Swift สามารถเขียนทั้ง Frontend (iOS App) และ Backend ด้วยภาษาเดียวกัน

Vapor เป็น Web Framework ที่ได้รับความนิยมมากที่สุดสำหรับ Server-Side Swift มีระบบ Routing, Middleware, ORM (Object-Relational Mapping), และ Template Engine ที่ครบครัน

---

## 1. What is Vapor?

Vapor คือ Web Framework สำหรับ Swift ที่ทำงานบน Server มีคุณสมบัติหลักดังนี้:

- **High Performance**: ใช้ SwiftNIO เป็น async I/O networking layer
- **Type Safe**: ใช้ระบบ Type System ของ Swift อย่างเต็มที่
- **Modern Swift**: รองรับ async/await, Concurrency
- **Fluent ORM**: จัดการ Database ด้วย Object-Relational Mapping
- **Leaf Templating**: สร้าง Dynamic HTML pages
- **Authentication**: ระบบ Authentication ในตัว

### ประวัติความเป็นมา

Vapor ถูกสร้างโดย Tanner Nelson ในปี 2016 และปัจจุบันอยู่ที่ Version 4 ซึ่งมีการปรับปรุงครั้งใหญ่รองรับ Swift Concurrency

```swift
// ตัวอย่าง Vapor Application ง่ายๆ
import Vapor

@main
struct App {
    static func main() async throws {
        var env = try Environment.detect()
        try LoggingSystem.bootstrap(from: &env)
        let app = Application(env)
        defer { app.shutdown() }
        try configure(app)
        try await app.run()
    }
}
```

---

## 2. Why Server-Side Swift?

### ข้อดีของการใช้ Swift สำหรับ Backend

**1. Code Sharing**
```swift
// โค้ดนี้ใช้ได้ทั้ง iOS และ Server
struct User: Codable {
    let id: UUID
    let name: String
    let email: String
    let createdAt: Date
}

// iOS App ใช้โมเดลนี้รับ data จาก API
// Server ใช้โมเดลนี้ส่ง data ให้ Client
```

**2. Type Safety**
```swift
// ข้อผิดพลาดถูกตรวจจับตั้งแต่ Compile Time
func getUser(id: UUID) async throws -> User {
    // ถ้าไม่คืน User จะ Error ตอน Compile
    guard let user = try await User.find(id, on: database) else {
        throw Abort(.notFound)
    }
    return user
}
```

**3. Performance**
- Swift ถูก Compile เป็น Native Code
- ไม่มี Virtual Machine (ต่างจาก Java/Python)
- Memory Management ด้วย ARC ที่มีประสิทธิภาพ

**4. Async/Await**
```swift
// Modern concurrency ทำให้เขียนโค้ด async ได้ง่าย
app.get("users") { req async throws -> [User] in
    let users = try await User.query(on: req.db).all()
    return users
}
```

### เปรียบเทียบกับ Backend Technologies อื่น

| Technology | Language | Performance | Type Safety |
|-----------|---------|------------|------------|
| Vapor | Swift | สูง | สูงมาก |
| Express.js | JavaScript | ปานกลาง | ต่ำ (ต้องใช้ TypeScript) |
| Django | Python | ปานกลาง | ต่ำ |
| Spring | Java | สูง | สูง |
| Go Fiber | Go | สูงมาก | ปานกลาง |

---

## 3. Installing Vapor

### ความต้องการของระบบ

- macOS 12+ หรือ Ubuntu 20.04+
- Swift 5.8+
- Xcode 14+ (สำหรับ macOS)

### ติดตั้ง Vapor Toolbox

```bash
# macOS - ใช้ Homebrew
brew install vapor

# ตรวจสอบการติดตั้ง
vapor --version
```

### ติดตั้งบน Ubuntu

```bash
# ติดตั้ง Swift ก่อน
curl -fsSL https://swift.org/install.sh | bash

# ติดตั้ง Vapor Toolbox
git clone https://github.com/vapor/toolbox.git
cd toolbox
swift build -c release
sudo mv .build/release/vapor /usr/local/bin
```

### ตรวจสอบ Swift Version

```bash
swift --version
# Swift version 5.9.0 (swift-5.9-RELEASE)
# Target: x86_64-unknown-linux-gnu

vapor --version
# vapor/4.x.x
```

---

## 4. Creating a Vapor Project

### สร้าง Project ใหม่

```bash
# สร้าง Project ชื่อ MyAPI
vapor new MyAPI

# เลือก Options:
# Would you like to use Fluent? Yes
# Which database driver? SQLite (สำหรับ development)
# Would you like to use Leaf? No

cd MyAPI
```

### เปิดใน Xcode

```bash
# เปิด Package.swift
open Package.swift
```

### โครงสร้าง Project

```
MyAPI/
├── Package.swift          # Swift Package configuration
├── Sources/
│   └── App/
│       ├── Controllers/   # Route Controllers
│       ├── Migrations/    # Database Migrations
│       ├── Models/        # Data Models
│       ├── configure.swift # App configuration
│       └── routes.swift   # Route definitions
├── Tests/
│   └── AppTests/
│       └── AppTests.swift
└── Resources/
    └── Views/             # Leaf templates (ถ้าใช้ Leaf)
```

### Package.swift

```swift
// swift-tools-version:5.9
import PackageDescription

let package = Package(
    name: "MyAPI",
    platforms: [
        .macOS(.v13)
    ],
    dependencies: [
        // Vapor framework
        .package(url: "https://github.com/vapor/vapor.git", from: "4.89.0"),
        // Fluent ORM
        .package(url: "https://github.com/vapor/fluent.git", from: "4.8.0"),
        // SQLite driver
        .package(url: "https://github.com/vapor/fluent-sqlite-driver.git", from: "4.3.0"),
    ],
    targets: [
        .executableTarget(
            name: "App",
            dependencies: [
                .product(name: "Vapor", package: "vapor"),
                .product(name: "Fluent", package: "fluent"),
                .product(name: "FluentSQLiteDriver", package: "fluent-sqlite-driver"),
            ]
        ),
        .testTarget(
            name: "AppTests",
            dependencies: [
                .target(name: "App"),
                .product(name: "XCTVapor", package: "vapor"),
            ]
        )
    ]
)
```

---

## 5. Application Structure

### configure.swift

```swift
import Fluent
import FluentSQLiteDriver
import Vapor

// configures your application
public func configure(_ app: Application) async throws {
    // Configure encoder/decoder
    let encoder = JSONEncoder()
    encoder.dateEncodingStrategy = .iso8601
    
    let decoder = JSONDecoder()
    decoder.dateDecodingStrategy = .iso8601
    
    ContentConfiguration.global.use(encoder: encoder, for: .json)
    ContentConfiguration.global.use(decoder: decoder, for: .json)
    
    // Configure database
    app.databases.use(.sqlite(.file("db.sqlite")), as: .sqlite)
    
    // Run migrations
    app.migrations.add(CreateUser())
    app.migrations.add(CreatePost())
    
    try await app.autoMigrate()
    
    // Configure middleware
    app.middleware.use(FileMiddleware(publicDirectory: app.directory.publicDirectory))
    app.middleware.use(ErrorMiddleware.default(environment: app.environment))
    
    // Register routes
    try routes(app)
}
```

### routes.swift

```swift
import Vapor

func routes(_ app: Application) throws {
    // Health check endpoint
    app.get { req async in
        "It works!"
    }
    
    app.get("hello") { req async -> String in
        "Hello, world!"
    }
    
    // Register controllers
    try app.register(collection: UserController())
    try app.register(collection: PostController())
}
```

### entrypoint.swift (หรือ main.swift)

```swift
import Vapor

@main
struct Entrypoint {
    static func main() async throws {
        var env = try Environment.detect()
        try LoggingSystem.bootstrap(from: &env)
        
        let app = Application(env)
        defer { app.shutdown() }
        
        do {
            try await configure(app)
        } catch {
            app.logger.report(error: error)
            throw error
        }
        
        try await app.run()
    }
}
```

---

## 6. Routing

### Basic Routes

```swift
import Vapor

func routes(_ app: Application) throws {
    // GET /
    app.get { req async -> String in
        "Homepage"
    }
    
    // GET /hello
    app.get("hello") { req async -> String in
        "Hello!"
    }
    
    // POST /login
    app.post("login") { req async throws -> String in
        // ประมวลผล login
        return "Logged in"
    }
    
    // PUT /users/:id
    app.put("users", ":id") { req async throws -> String in
        let id = req.parameters.get("id")!
        return "Updated user \(id)"
    }
    
    // DELETE /users/:id
    app.delete("users", ":id") { req async throws -> HTTPStatus in
        let id = req.parameters.get("id")!
        // ลบ user
        return .noContent
    }
    
    // PATCH /users/:id
    app.patch("users", ":id") { req async throws -> String in
        "Patched"
    }
}
```

### Route Groups

```swift
// สร้าง Route Group สำหรับ /api
let api = app.grouped("api")

// GET /api/users
api.get("users") { req async throws -> [User] in
    try await User.query(on: req.db).all()
}

// POST /api/users
api.post("users") { req async throws -> User in
    let user = try req.content.decode(User.self)
    try await user.save(on: req.db)
    return user
}

// Nested groups
let v1 = app.grouped("v1")
let v1Api = v1.grouped("api")

// GET /v1/api/users
v1Api.get("users") { req async throws -> [User] in
    try await User.query(on: req.db).all()
}
```

### Route Collections

```swift
// สร้าง Controller ที่ implement RouteCollection
struct UserController: RouteCollection {
    func boot(routes: RoutesBuilder) throws {
        let users = routes.grouped("users")
        
        users.get(use: index)
        users.post(use: create)
        users.group(":userID") { user in
            user.get(use: show)
            user.put(use: update)
            user.delete(use: delete)
        }
    }
    
    // GET /users
    func index(req: Request) async throws -> [User] {
        try await User.query(on: req.db).all()
    }
    
    // POST /users
    func create(req: Request) async throws -> User {
        let user = try req.content.decode(User.self)
        try await user.save(on: req.db)
        return user
    }
    
    // GET /users/:userID
    func show(req: Request) async throws -> User {
        guard let user = try await User.find(req.parameters.get("userID"), on: req.db) else {
            throw Abort(.notFound)
        }
        return user
    }
    
    // PUT /users/:userID
    func update(req: Request) async throws -> User {
        guard let user = try await User.find(req.parameters.get("userID"), on: req.db) else {
            throw Abort(.notFound)
        }
        let updatedUser = try req.content.decode(User.self)
        user.name = updatedUser.name
        user.email = updatedUser.email
        try await user.save(on: req.db)
        return user
    }
    
    // DELETE /users/:userID
    func delete(req: Request) async throws -> HTTPStatus {
        guard let user = try await User.find(req.parameters.get("userID"), on: req.db) else {
            throw Abort(.notFound)
        }
        try await user.delete(on: req.db)
        return .noContent
    }
}
```

---

## 7. Route Handlers

### Response Types

Route Handlers สามารถคืนค่าหลายรูปแบบ:

```swift
// คืน String
app.get("text") { req async -> String in
    "Hello, World!"
}

// คืน Int
app.get("number") { req async -> Int in
    42
}

// คืน JSON (ผ่าน Codable)
app.get("user") { req async -> User in
    User(id: 1, name: "John", email: "john@example.com")
}

// คืน Array
app.get("users") { req async throws -> [User] in
    try await User.query(on: req.db).all()
}

// คืน Response โดยตรง
app.get("custom") { req async -> Response in
    let response = Response(status: .ok)
    response.headers.contentType = .json
    try response.content.encode(["message": "Hello"])
    return response
}

// คืน HTTPStatus
app.delete("resource", ":id") { req async throws -> HTTPStatus in
    // ลบ resource
    return .noContent  // 204
}
```

### Async Handlers

```swift
// ใช้ async/await
app.get("data") { req async throws -> SomeData in
    // เรียก database
    let items = try await Item.query(on: req.db).all()
    // เรียก external API
    let result = try await req.client.get("https://api.example.com/data")
    // ประมวลผล
    return SomeData(items: items)
}
```

### Error Handling ใน Handlers

```swift
app.get("users", ":id") { req async throws -> User in
    // Validate parameter
    guard let id = req.parameters.get("id", as: UUID.self) else {
        throw Abort(.badRequest, reason: "Invalid UUID format")
    }
    
    // Find user
    guard let user = try await User.find(id, on: req.db) else {
        throw Abort(.notFound, reason: "User not found")
    }
    
    return user
}

// Custom Error
enum AppError: AbortError {
    case userNotFound
    case invalidCredentials
    case insufficientPermissions
    
    var status: HTTPResponseStatus {
        switch self {
        case .userNotFound: return .notFound
        case .invalidCredentials: return .unauthorized
        case .insufficientPermissions: return .forbidden
        }
    }
    
    var reason: String {
        switch self {
        case .userNotFound: return "ไม่พบผู้ใช้งาน"
        case .invalidCredentials: return "ข้อมูลการเข้าสู่ระบบไม่ถูกต้อง"
        case .insufficientPermissions: return "ไม่มีสิทธิ์เข้าถึง"
        }
    }
}
```

---

## 8. Request and Response

### Request Object

```swift
app.post("process") { req async throws -> String in
    // Headers
    let contentType = req.headers.contentType
    let authorization = req.headers.bearerAuthorization
    let customHeader = req.headers["X-Custom-Header"].first
    
    // URL
    let url = req.url
    let path = req.url.path
    let query = req.url.query
    
    // Body
    let body = req.body
    
    // Logger
    req.logger.info("Processing request")
    
    // Database
    let db = req.db
    
    // Auth
    let user = try req.auth.require(User.self)
    
    return "Processed"
}
```

### Response Object

```swift
// สร้าง Custom Response
app.get("custom-response") { req async throws -> Response in
    var headers = HTTPHeaders()
    headers.add(name: .contentType, value: "application/json")
    headers.add(name: "X-Custom-Header", value: "MyValue")
    
    let body = Response.Body(string: """
    {
        "message": "Hello",
        "timestamp": "\(Date())"
    }
    """)
    
    return Response(
        status: .ok,
        headers: headers,
        body: body
    )
}

// Response พร้อม Cookie
app.post("login") { req async throws -> Response in
    // authenticate user...
    
    let response = Response(status: .ok)
    response.cookies["session"] = HTTPCookies.Value(
        string: "session-token-here",
        expires: Date().addingTimeInterval(86400),
        isHTTPOnly: true,
        isSecure: true
    )
    
    try response.content.encode(["message": "Logged in"])
    return response
}
```

---

## 9. Content (JSON Decoding/Encoding)

### Encoding และ Decoding

```swift
// Model ที่ใช้ Codable
struct CreateUserDTO: Content {
    let name: String
    let email: String
    let password: String
}

struct UserResponse: Content {
    let id: UUID
    let name: String
    let email: String
    let createdAt: Date
}

// Decode จาก Request Body
app.post("users") { req async throws -> UserResponse in
    let createDTO = try req.content.decode(CreateUserDTO.self)
    
    // Validate
    guard createDTO.name.count >= 2 else {
        throw Abort(.badRequest, reason: "ชื่อต้องมีอย่างน้อย 2 ตัวอักษร")
    }
    
    guard createDTO.email.contains("@") else {
        throw Abort(.badRequest, reason: "Email ไม่ถูกต้อง")
    }
    
    // สร้าง User
    let user = User(
        name: createDTO.name,
        email: createDTO.email,
        passwordHash: try Bcrypt.hash(createDTO.password)
    )
    try await user.save(on: req.db)
    
    return UserResponse(
        id: user.id!,
        name: user.name,
        email: user.email,
        createdAt: user.createdAt ?? Date()
    )
}
```

### Validation

```swift
import Vapor

struct CreateUserDTO: Content, Validatable {
    let name: String
    let email: String
    let password: String
    let age: Int
    
    static func validations(_ validations: inout Validations) {
        validations.add("name", as: String.self, is: .count(2...50))
        validations.add("email", as: String.self, is: .email)
        validations.add("password", as: String.self, is: .count(8...))
        validations.add("age", as: Int.self, is: .range(18...))
    }
}

app.post("register") { req async throws -> UserResponse in
    // Validate ก่อน decode
    try CreateUserDTO.validate(content: req)
    let dto = try req.content.decode(CreateUserDTO.self)
    
    // ต่อจากนี้แน่ใจได้ว่าข้อมูลถูกต้อง
    let user = User(name: dto.name, email: dto.email, age: dto.age)
    try await user.save(on: req.db)
    
    return UserResponse(id: user.id!, name: user.name, email: user.email)
}
```

### JSON Configuration

```swift
// ใน configure.swift
let encoder = JSONEncoder()
encoder.dateEncodingStrategy = .iso8601
encoder.keyEncodingStrategy = .convertToSnakeCase
encoder.outputFormatting = [.prettyPrinted, .sortedKeys]

let decoder = JSONDecoder()
decoder.dateDecodingStrategy = .iso8601
decoder.keyDecodingStrategy = .convertFromSnakeCase

ContentConfiguration.global.use(encoder: encoder, for: .json)
ContentConfiguration.global.use(decoder: decoder, for: .json)
```

---

## 10. Query Parameters

### อ่าน Query Parameters

```swift
// GET /search?q=swift&page=1&limit=20
app.get("search") { req async throws -> SearchResult in
    // อ่านค่าแบบ Optional
    let query = req.query[String.self, at: "q"]
    let page = req.query[Int.self, at: "page"] ?? 1
    let limit = req.query[Int.self, at: "limit"] ?? 20
    
    // หรือใช้ decode
    struct SearchParams: Content {
        let q: String
        let page: Int?
        let limit: Int?
        let category: String?
    }
    
    let params = try req.query.decode(SearchParams.self)
    
    // ค้นหา
    let results = try await Item.query(on: req.db)
        .filter(\.$name ~~ (params.q))
        .range((params.page ?? 1 - 1) * (params.limit ?? 20)..<(params.page ?? 1) * (params.limit ?? 20))
        .all()
    
    return SearchResult(
        items: results,
        page: params.page ?? 1,
        limit: params.limit ?? 20,
        total: results.count
    )
}
```

### Query Parameter Validation

```swift
struct PaginationParams: Content, Validatable {
    let page: Int?
    let limit: Int?
    let sortBy: String?
    let order: String?
    
    static func validations(_ validations: inout Validations) {
        validations.add("page", as: Int?.self, is: .nil || .range(1...))
        validations.add("limit", as: Int?.self, is: .nil || .range(1...100))
        validations.add("order", as: String?.self, is: .nil || .in("asc", "desc"))
    }
}

app.get("items") { req async throws -> Page<Item> in
    try PaginationParams.validate(query: req)
    let params = try req.query.decode(PaginationParams.self)
    
    let page = params.page ?? 1
    let limit = params.limit ?? 20
    
    return try await Item.query(on: req.db)
        .paginate(PageRequest(page: page, per: limit))
}
```

---

## 11. Path Parameters

### Basic Path Parameters

```swift
// :id เป็น Path Parameter
app.get("users", ":id") { req async throws -> User in
    // อ่านเป็น String
    let idString = req.parameters.get("id")!
    
    // อ่านเป็น specific type
    guard let id = req.parameters.get("id", as: UUID.self) else {
        throw Abort(.badRequest, reason: "Invalid ID format")
    }
    
    guard let user = try await User.find(id, on: req.db) else {
        throw Abort(.notFound)
    }
    
    return user
}

// หลาย Path Parameters
app.get("categories", ":categoryId", "products", ":productId") { req async throws -> Product in
    guard let categoryId = req.parameters.get("categoryId", as: UUID.self),
          let productId = req.parameters.get("productId", as: UUID.self) else {
        throw Abort(.badRequest)
    }
    
    guard let product = try await Product.query(on: req.db)
        .filter(\.$id == productId)
        .filter(\.$categoryId == categoryId)
        .first() else {
        throw Abort(.notFound)
    }
    
    return product
}
```

### Catchall Parameters

```swift
// * จับ path segment เดียว
app.get("files", "*") { req -> String in
    "Single segment"
}

// ** จับ path segments ทั้งหมด
app.get("files", "**") { req -> String in
    let path = req.parameters.getCatchall().joined(separator: "/")
    return "Path: \(path)"
}
```

---

## 12. Request Body

### รับ JSON Body

```swift
struct CreatePost: Content {
    let title: String
    let content: String
    let tags: [String]
    let isPublished: Bool
}

app.post("posts") { req async throws -> Post in
    let createPost = try req.content.decode(CreatePost.self)
    
    let post = Post(
        title: createPost.title,
        content: createPost.content,
        tags: createPost.tags,
        isPublished: createPost.isPublished
    )
    try await post.save(on: req.db)
    return post
}
```

### รับ Form Data

```swift
struct LoginForm: Content {
    let email: String
    let password: String
    let rememberMe: Bool?
}

// HTML Form POST
app.post("login") { req async throws -> Response in
    let form = try req.content.decode(LoginForm.self)
    // ตรวจสอบ credentials...
    return req.redirect(to: "/dashboard")
}
```

### รับ File Upload

```swift
struct UploadForm: Content {
    let file: File
    let description: String?
}

app.post("upload") { req async throws -> String in
    let form = try req.content.decode(UploadForm.self)
    let file = form.file
    
    // ตรวจสอบ file type
    guard file.contentType?.type == "image" else {
        throw Abort(.badRequest, reason: "ต้องเป็นไฟล์รูปภาพเท่านั้น")
    }
    
    // บันทึกไฟล์
    let filename = "\(UUID().uuidString).\(file.extension ?? "bin")"
    let path = app.directory.publicDirectory + "uploads/" + filename
    
    try await req.fileio.writeFile(file.data, at: path)
    
    return "/uploads/\(filename)"
}
```

### Body Streaming

```swift
// สำหรับไฟล์ขนาดใหญ่
app.on(.POST, "large-upload", body: .stream) { req -> EventLoopFuture<HTTPStatus> in
    req.body.drain { chunk in
        switch chunk {
        case .buffer(let buffer):
            // ประมวลผล chunk
            print("Received \(buffer.readableBytes) bytes")
            return req.eventLoop.makeSucceededFuture(())
        case .error(let error):
            return req.eventLoop.makeFailedFuture(error)
        case .end:
            return req.eventLoop.makeSucceededFuture(())
        }
    }
}
```

---

## 13. Middleware

Middleware คือ code ที่ทำงานก่อนและ/หรือหลัง Route Handler เหมาะสำหรับ:
- Logging
- Authentication
- CORS
- Rate Limiting
- Error Handling
- Request/Response transformation

### ลำดับการทำงาน

```
Request → Middleware 1 → Middleware 2 → Route Handler → Middleware 2 → Middleware 1 → Response
```

### การเพิ่ม Middleware

```swift
// เพิ่มใน configure.swift
app.middleware.use(FileMiddleware(publicDirectory: app.directory.publicDirectory))
app.middleware.use(ErrorMiddleware.default(environment: app.environment))

// เพิ่มกับ Route Group เฉพาะ
let protected = app.grouped(UserAuthenticator())
protected.get("profile") { req async throws -> Profile in
    let user = try req.auth.require(User.self)
    return user.profile
}
```

---

## 14. Custom Middleware

### สร้าง Middleware เอง

```swift
// Logging Middleware
struct RequestLoggerMiddleware: AsyncMiddleware {
    func respond(to request: Request, chainingTo next: AsyncResponder) async throws -> Response {
        let start = Date()
        
        request.logger.info("→ \(request.method) \(request.url.path)")
        
        let response = try await next.respond(to: request)
        
        let elapsed = Date().timeIntervalSince(start) * 1000
        request.logger.info("← \(response.status.code) [\(String(format: "%.2f", elapsed))ms]")
        
        return response
    }
}

// ใช้งาน
app.middleware.use(RequestLoggerMiddleware())
```

### Rate Limiting Middleware

```swift
actor RateLimiter {
    private var requestCounts: [String: (count: Int, resetAt: Date)] = [:]
    private let maxRequests: Int
    private let windowSeconds: TimeInterval
    
    init(maxRequests: Int = 100, windowSeconds: TimeInterval = 60) {
        self.maxRequests = maxRequests
        self.windowSeconds = windowSeconds
    }
    
    func shouldAllow(ip: String) -> Bool {
        let now = Date()
        
        if let record = requestCounts[ip] {
            if now > record.resetAt {
                requestCounts[ip] = (1, now.addingTimeInterval(windowSeconds))
                return true
            }
            if record.count >= maxRequests {
                return false
            }
            requestCounts[ip] = (record.count + 1, record.resetAt)
            return true
        }
        
        requestCounts[ip] = (1, now.addingTimeInterval(windowSeconds))
        return true
    }
}

struct RateLimitMiddleware: AsyncMiddleware {
    let limiter: RateLimiter
    
    func respond(to request: Request, chainingTo next: AsyncResponder) async throws -> Response {
        let ip = request.peerAddress?.description ?? "unknown"
        
        guard await limiter.shouldAllow(ip: ip) else {
            throw Abort(.tooManyRequests, reason: "เกินจำนวน Request ที่กำหนด กรุณารอสักครู่")
        }
        
        return try await next.respond(to: request)
    }
}

// ใช้งาน
let rateLimiter = RateLimiter(maxRequests: 100, windowSeconds: 60)
app.middleware.use(RateLimitMiddleware(limiter: rateLimiter))
```

### Response Transformer Middleware

```swift
struct APIVersionMiddleware: AsyncMiddleware {
    func respond(to request: Request, chainingTo next: AsyncResponder) async throws -> Response {
        var response = try await next.respond(to: request)
        response.headers.add(name: "X-API-Version", value: "1.0")
        response.headers.add(name: "X-Powered-By", value: "Vapor/4.0")
        return response
    }
}
```

---

## 15. CORS Middleware

### ตั้งค่า CORS

```swift
// ใน configure.swift
let corsConfiguration = CORSMiddleware.Configuration(
    allowedOrigin: .custom("https://myapp.com"),  // หรือ .all สำหรับ development
    allowedMethods: [.GET, .POST, .PUT, .DELETE, .PATCH, .OPTIONS],
    allowedHeaders: [
        .accept,
        .authorization,
        .contentType,
        .origin,
        .xRequestedWith,
        HTTPHeaders.Name("X-Custom-Header")
    ],
    allowCredentials: true,
    cacheExpiration: 600  // 10 minutes
)

let cors = CORSMiddleware(configuration: corsConfiguration)

// ต้องเพิ่ม CORS ก่อน middleware อื่น
app.middleware.use(cors, at: .beginning)
```

### CORS สำหรับหลาย Origins

```swift
struct MultiOriginCORSMiddleware: AsyncMiddleware {
    let allowedOrigins: Set<String>
    
    func respond(to request: Request, chainingTo next: AsyncResponder) async throws -> Response {
        let response = try await next.respond(to: request)
        
        if let origin = request.headers[.origin].first,
           allowedOrigins.contains(origin) {
            response.headers.replaceOrAdd(name: .accessControlAllowOrigin, value: origin)
            response.headers.replaceOrAdd(name: .accessControlAllowCredentials, value: "true")
            response.headers.replaceOrAdd(
                name: .accessControlAllowHeaders,
                value: "Content-Type, Authorization"
            )
        }
        
        return response
    }
}

// ใช้งาน
let corsMiddleware = MultiOriginCORSMiddleware(allowedOrigins: [
    "https://app.example.com",
    "https://admin.example.com",
    "http://localhost:3000"  // development
])
app.middleware.use(corsMiddleware)
```

---

## 16. Authentication Middleware

### Basic Authentication

```swift
import Vapor

// User Model ที่ implement Authenticatable
final class User: Model, Content {
    static let schema = "users"
    
    @ID(key: .id) var id: UUID?
    @Field(key: "email") var email: String
    @Field(key: "password_hash") var passwordHash: String
    
    init() {}
    
    init(id: UUID? = nil, email: String, passwordHash: String) {
        self.id = id
        self.email = email
        self.passwordHash = passwordHash
    }
}

extension User: ModelAuthenticatable {
    static let usernameKey = \User.$email
    static let passwordHashKey = \User.$passwordHash
    
    func verify(password: String) throws -> Bool {
        try Bcrypt.verify(password, created: self.passwordHash)
    }
}

// ใช้งาน Basic Auth
let basic = app.grouped(User.authenticator())
basic.post("login") { req async throws -> UserToken in
    let user = try req.auth.require(User.self)
    let token = try await UserToken.generate(for: user, on: req.db)
    return token
}
```

### Token Authentication

```swift
// Token Model
final class UserToken: Model, Content {
    static let schema = "user_tokens"
    
    @ID(key: .id) var id: UUID?
    @Field(key: "value") var value: String
    @Field(key: "expires_at") var expiresAt: Date?
    @Parent(key: "user_id") var user: User
    
    init() {}
    
    init(id: UUID? = nil, value: String, expiresAt: Date?, userID: User.IDValue) {
        self.id = id
        self.value = value
        self.expiresAt = expiresAt
        self.$user.id = userID
    }
}

extension UserToken: ModelTokenAuthenticatable {
    static let valueKey = \UserToken.$value
    static let userKey = \UserToken.$user
    
    var isValid: Bool {
        guard let expiresAt = expiresAt else { return true }
        return expiresAt > Date()
    }
}

// สร้าง Token
extension UserToken {
    static func generate(for user: User, on db: Database) async throws -> UserToken {
        let token = UserToken(
            value: [UInt8].random(count: 32).base64,
            expiresAt: Date().addingTimeInterval(60 * 60 * 24 * 7),  // 7 days
            userID: try user.requireID()
        )
        try await token.save(on: db)
        return token
    }
}

// Protected routes
let tokenAuth = app.grouped(UserToken.authenticator())
let protected = tokenAuth.grouped(User.guardMiddleware())

protected.get("me") { req async throws -> User in
    try req.auth.require(User.self)
}

protected.get("profile") { req async throws -> Profile in
    let user = try req.auth.require(User.self)
    return try await user.$profile.get(on: req.db)
}
```

---

## 17. Fluent ORM

Fluent คือ ORM (Object-Relational Mapper) ของ Vapor ช่วยให้ทำงานกับ Database โดยไม่ต้องเขียน SQL โดยตรง

### ตั้งค่า Fluent

```swift
// ใน Package.swift - เพิ่ม dependencies
.package(url: "https://github.com/vapor/fluent.git", from: "4.8.0"),
.package(url: "https://github.com/vapor/fluent-postgres-driver.git", from: "2.7.0"),
```

```swift
// ใน configure.swift
import Fluent
import FluentPostgresDriver

app.databases.use(.postgres(
    hostname: Environment.get("DATABASE_HOST") ?? "localhost",
    username: Environment.get("DATABASE_USERNAME") ?? "postgres",
    password: Environment.get("DATABASE_PASSWORD") ?? "",
    database: Environment.get("DATABASE_NAME") ?? "myapp"
), as: .psql)
```

---

## 18. Database Drivers

### PostgreSQL

```swift
// Package.swift
.package(url: "https://github.com/vapor/fluent-postgres-driver.git", from: "2.7.0"),

// configure.swift
import FluentPostgresDriver

app.databases.use(.postgres(
    hostname: "localhost",
    port: 5432,
    username: "postgres",
    password: "password",
    database: "mydb",
    tlsConfiguration: .none
), as: .psql)
```

### MySQL

```swift
// Package.swift
.package(url: "https://github.com/vapor/fluent-mysql-driver.git", from: "4.3.0"),

// configure.swift
import FluentMySQLDriver

app.databases.use(.mysql(
    hostname: "localhost",
    port: 3306,
    username: "root",
    password: "password",
    database: "mydb"
), as: .mysql)
```

### SQLite (สำหรับ Development)

```swift
// Package.swift
.package(url: "https://github.com/vapor/fluent-sqlite-driver.git", from: "4.3.0"),

// configure.swift
import FluentSQLiteDriver

// บันทึกลงไฟล์
app.databases.use(.sqlite(.file("db.sqlite")), as: .sqlite)

// หรือ In-Memory (ล้างข้อมูลเมื่อ restart)
app.databases.use(.sqlite(.memory), as: .sqlite)
```

### ใช้หลาย Databases พร้อมกัน

```swift
// Primary database
app.databases.use(.postgres(...), as: .psql)

// Analytics database  
app.databases.use(.postgres(...), as: DatabaseID(string: "analytics"))

// Cache database
app.databases.use(.sqlite(.memory), as: DatabaseID(string: "cache"))

// ระบุ database ที่ต้องการใช้
let users = try await User.query(on: req.db(.psql)).all()
let events = try await Event.query(on: req.db(DatabaseID(string: "analytics"))).all()
```

---

## 19. Models and Migrations

### สร้าง Model

```swift
import Fluent
import Vapor

final class Product: Model, Content {
    // ชื่อ Table ใน Database
    static let schema = "products"
    
    @ID(key: .id)
    var id: UUID?
    
    @Field(key: "name")
    var name: String
    
    @Field(key: "description")
    var description: String
    
    @Field(key: "price")
    var price: Double
    
    @Field(key: "stock")
    var stock: Int
    
    @OptionalField(key: "image_url")
    var imageURL: String?
    
    @Timestamp(key: "created_at", on: .create)
    var createdAt: Date?
    
    @Timestamp(key: "updated_at", on: .update)
    var updatedAt: Date?
    
    @Timestamp(key: "deleted_at", on: .delete)
    var deletedAt: Date?  // Soft delete
    
    // Required by Fluent
    init() {}
    
    init(id: UUID? = nil, name: String, description: String, price: Double, stock: Int, imageURL: String? = nil) {
        self.id = id
        self.name = name
        self.description = description
        self.price = price
        self.stock = stock
        self.imageURL = imageURL
    }
}
```

### สร้าง Migration

```swift
import Fluent

struct CreateProduct: AsyncMigration {
    func prepare(on database: Database) async throws {
        try await database.schema("products")
            .id()
            .field("name", .string, .required)
            .field("description", .string, .required)
            .field("price", .double, .required)
            .field("stock", .int, .required)
            .field("image_url", .string)
            .field("created_at", .datetime)
            .field("updated_at", .datetime)
            .field("deleted_at", .datetime)
            .create()
    }
    
    func revert(on database: Database) async throws {
        try await database.schema("products").delete()
    }
}

// Migration ที่ซับซ้อนขึ้น - เพิ่ม column
struct AddCategoryToProduct: AsyncMigration {
    func prepare(on database: Database) async throws {
        try await database.schema("products")
            .field("category_id", .uuid, .references("categories", "id"))
            .update()
    }
    
    func revert(on database: Database) async throws {
        try await database.schema("products")
            .deleteField("category_id")
            .update()
    }
}

// เพิ่ม Migration ใน configure.swift
app.migrations.add(CreateProduct())
app.migrations.add(AddCategoryToProduct())

// Run migrations
try await app.autoMigrate()
```

---

## 20. CRUD Operations with Fluent

### Create

```swift
// สร้าง Product เดียว
app.post("products") { req async throws -> Product in
    let product = try req.content.decode(Product.self)
    try await product.save(on: req.db)
    return product
}

// สร้างหลาย Products
app.post("products", "bulk") { req async throws -> [Product] in
    let products = try req.content.decode([Product].self)
    try await products.create(on: req.db)
    return products
}
```

### Read

```swift
// ดึงทั้งหมด
app.get("products") { req async throws -> [Product] in
    try await Product.query(on: req.db).all()
}

// ดึงด้วย ID
app.get("products", ":id") { req async throws -> Product in
    guard let product = try await Product.find(req.parameters.get("id"), on: req.db) else {
        throw Abort(.notFound)
    }
    return product
}

// ดึงด้วย Query
app.get("products", "search") { req async throws -> [Product] in
    let name = req.query[String.self, at: "name"] ?? ""
    let minPrice = req.query[Double.self, at: "min_price"] ?? 0
    let maxPrice = req.query[Double.self, at: "max_price"] ?? Double.infinity
    
    return try await Product.query(on: req.db)
        .filter(\.$name ~~ name)           // LIKE search
        .filter(\.$price >= minPrice)
        .filter(\.$price <= maxPrice)
        .sort(\.$name)
        .all()
}

// Pagination
app.get("products", "page") { req async throws -> Page<Product> in
    let page = req.query[Int.self, at: "page"] ?? 1
    let per = req.query[Int.self, at: "per"] ?? 20
    
    return try await Product.query(on: req.db)
        .paginate(PageRequest(page: page, per: per))
}
```

### Update

```swift
// Full Update (PUT)
app.put("products", ":id") { req async throws -> Product in
    guard let product = try await Product.find(req.parameters.get("id"), on: req.db) else {
        throw Abort(.notFound)
    }
    
    let updatedProduct = try req.content.decode(Product.self)
    product.name = updatedProduct.name
    product.description = updatedProduct.description
    product.price = updatedProduct.price
    product.stock = updatedProduct.stock
    product.imageURL = updatedProduct.imageURL
    
    try await product.save(on: req.db)
    return product
}

// Partial Update (PATCH)
struct UpdateProductDTO: Content {
    let name: String?
    let price: Double?
    let stock: Int?
}

app.patch("products", ":id") { req async throws -> Product in
    guard let product = try await Product.find(req.parameters.get("id"), on: req.db) else {
        throw Abort(.notFound)
    }
    
    let update = try req.content.decode(UpdateProductDTO.self)
    
    if let name = update.name { product.name = name }
    if let price = update.price { product.price = price }
    if let stock = update.stock { product.stock = stock }
    
    try await product.save(on: req.db)
    return product
}
```

### Delete

```swift
// Hard Delete
app.delete("products", ":id") { req async throws -> HTTPStatus in
    guard let product = try await Product.find(req.parameters.get("id"), on: req.db) else {
        throw Abort(.notFound)
    }
    try await product.delete(on: req.db)
    return .noContent
}

// Soft Delete (ต้องมี @Timestamp deleted_at ใน Model)
app.delete("products", ":id", "soft") { req async throws -> HTTPStatus in
    guard let product = try await Product.find(req.parameters.get("id"), on: req.db) else {
        throw Abort(.notFound)
    }
    // จะ set deleted_at แทนการลบจริง
    try await product.delete(on: req.db)
    return .noContent
}

// Restore soft deleted
app.post("products", ":id", "restore") { req async throws -> Product in
    guard let product = try await Product.query(on: req.db)
        .withDeleted()
        .filter(\.$id == req.parameters.get("id", as: UUID.self)!)
        .first() else {
        throw Abort(.notFound)
    }
    try await product.restore(on: req.db)
    return product
}
```

---

## 21. Relations in Fluent

### One-to-Many

```swift
// Category มีหลาย Products
final class Category: Model, Content {
    static let schema = "categories"
    
    @ID(key: .id) var id: UUID?
    @Field(key: "name") var name: String
    
    // Relation ไปหา Products
    @Children(for: \.$category)
    var products: [Product]
    
    init() {}
    init(id: UUID? = nil, name: String) {
        self.id = id
        self.name = name
    }
}

// Product อยู่ใน Category เดียว
final class Product: Model, Content {
    static let schema = "products"
    
    @ID(key: .id) var id: UUID?
    @Field(key: "name") var name: String
    @Field(key: "price") var price: Double
    
    // Foreign Key ไปหา Category
    @Parent(key: "category_id")
    var category: Category
    
    init() {}
    init(id: UUID? = nil, name: String, price: Double, categoryID: Category.IDValue) {
        self.id = id
        self.name = name
        self.price = price
        self.$category.id = categoryID
    }
}

// ใช้งาน
app.get("categories", ":id", "products") { req async throws -> [Product] in
    guard let category = try await Category.find(req.parameters.get("id"), on: req.db) else {
        throw Abort(.notFound)
    }
    return try await category.$products.get(on: req.db)
}

// Eager Loading - โหลด relation พร้อมกัน
app.get("categories") { req async throws -> [Category] in
    try await Category.query(on: req.db)
        .with(\.$products)  // Eager load products
        .all()
}
```

### Many-to-Many

```swift
// Product มีหลาย Tags, Tag อยู่ใน Products หลายตัว
final class Tag: Model, Content {
    static let schema = "tags"
    
    @ID(key: .id) var id: UUID?
    @Field(key: "name") var name: String
    
    @Siblings(through: ProductTag.self, from: \.$tag, to: \.$product)
    var products: [Product]
    
    init() {}
    init(id: UUID? = nil, name: String) {
        self.id = id
        self.name = name
    }
}

// Pivot Table
final class ProductTag: Model {
    static let schema = "product_tags"
    
    @ID(key: .id) var id: UUID?
    
    @Parent(key: "product_id") var product: Product
    @Parent(key: "tag_id") var tag: Tag
    
    init() {}
    init(productID: Product.IDValue, tagID: Tag.IDValue) {
        self.$product.id = productID
        self.$tag.id = tagID
    }
}

// เพิ่ม Siblings ใน Product
extension Product {
    @Siblings(through: ProductTag.self, from: \.$product, to: \.$tag)
    var tags: [Tag]
}

// Migration สำหรับ Pivot Table
struct CreateProductTag: AsyncMigration {
    func prepare(on database: Database) async throws {
        try await database.schema("product_tags")
            .id()
            .field("product_id", .uuid, .required, .references("products", "id", onDelete: .cascade))
            .field("tag_id", .uuid, .required, .references("tags", "id", onDelete: .cascade))
            .unique(on: "product_id", "tag_id")
            .create()
    }
    
    func revert(on database: Database) async throws {
        try await database.schema("product_tags").delete()
    }
}

// ใช้งาน
app.post("products", ":id", "tags", ":tagId") { req async throws -> HTTPStatus in
    guard let product = try await Product.find(req.parameters.get("id"), on: req.db),
          let tag = try await Tag.find(req.parameters.get("tagId"), on: req.db) else {
        throw Abort(.notFound)
    }
    
    try await product.$tags.attach(tag, on: req.db)
    return .ok
}

app.get("products", ":id") { req async throws -> ProductResponse in
    guard let product = try await Product.query(on: req.db)
        .with(\.$tags)
        .filter(\.$id == req.parameters.get("id", as: UUID.self)!)
        .first() else {
        throw Abort(.notFound)
    }
    return ProductResponse(product: product, tags: product.tags)
}
```

### One-to-One

```swift
// User มี Profile หนึ่งอัน
final class Profile: Model, Content {
    static let schema = "profiles"
    
    @ID(key: .id) var id: UUID?
    @Field(key: "bio") var bio: String
    @Field(key: "avatar_url") var avatarURL: String?
    
    @Parent(key: "user_id")
    var user: User
    
    init() {}
    init(id: UUID? = nil, bio: String, avatarURL: String? = nil, userID: User.IDValue) {
        self.id = id
        self.bio = bio
        self.avatarURL = avatarURL
        self.$user.id = userID
    }
}

extension User {
    @OptionalChild(for: \.$user)
    var profile: Profile?
}
```

---

## 22. Async/Await in Vapor

### การใช้ async/await

```swift
// ก่อน Vapor 4 (EventLoopFuture style)
func getUserOld(req: Request) -> EventLoopFuture<User> {
    User.find(req.parameters.get("id"), on: req.db)
        .unwrap(or: Abort(.notFound))
}

// Vapor 4 ด้วย async/await (แนะนำ)
func getUserNew(req: Request) async throws -> User {
    guard let user = try await User.find(req.parameters.get("id"), on: req.db) else {
        throw Abort(.notFound)
    }
    return user
}
```

### Parallel Execution

```swift
app.get("dashboard") { req async throws -> Dashboard in
    // ทำงานพร้อมกัน ไม่รอกัน
    async let users = User.query(on: req.db).count()
    async let products = Product.query(on: req.db).count()
    async let orders = Order.query(on: req.db).count()
    async let revenue = Order.query(on: req.db).sum(\.$total)
    
    // รอทุกอย่างพร้อมกัน
    let (userCount, productCount, orderCount, totalRevenue) = try await (
        users, products, orders, revenue
    )
    
    return Dashboard(
        users: userCount,
        products: productCount,
        orders: orderCount,
        revenue: totalRevenue ?? 0
    )
}
```

### Task Groups

```swift
app.get("process") { req async throws -> ProcessResult in
    let items = try await Item.query(on: req.db).all()
    
    let results = try await withThrowingTaskGroup(of: ItemResult.self) { group in
        for item in items {
            group.addTask {
                // ประมวลผล item แต่ละตัวพร้อมกัน
                return try await processItem(item, on: req.db)
            }
        }
        
        var allResults: [ItemResult] = []
        for try await result in group {
            allResults.append(result)
        }
        return allResults
    }
    
    return ProcessResult(results: results)
}
```

---

## 23. WebSockets in Vapor

### Basic WebSocket

```swift
// สร้าง WebSocket endpoint
app.webSocket("chat") { req, ws in
    // เมื่อ client เชื่อมต่อ
    ws.send("Welcome to chat!")
    
    // รับ message
    ws.onText { ws, text in
        print("Received: \(text)")
        // ส่งกลับ
        ws.send("Echo: \(text)")
    }
    
    ws.onBinary { ws, buffer in
        print("Received binary data: \(buffer.readableBytes) bytes")
    }
    
    // เมื่อ client ตัดการเชื่อมต่อ
    ws.onClose.whenComplete { _ in
        print("Client disconnected")
    }
}
```

### Chat Room ด้วย WebSocket

```swift
// จัดการ WebSocket connections
actor ChatRoom {
    private var connections: [UUID: WebSocket] = [:]
    
    func connect(id: UUID, ws: WebSocket) {
        connections[id] = ws
    }
    
    func disconnect(id: UUID) {
        connections.removeValue(forKey: id)
    }
    
    func broadcast(message: String, except excludeID: UUID? = nil) async {
        for (id, ws) in connections {
            if id != excludeID && !ws.isClosed {
                ws.send(message)
            }
        }
    }
    
    var count: Int {
        connections.count
    }
}

// Setup
let chatRoom = ChatRoom()

app.webSocket("chat", ":roomId") { req, ws in
    let userId = UUID()
    let roomId = req.parameters.get("roomId")!
    
    // เชื่อมต่อ
    await chatRoom.connect(id: userId, ws: ws)
    await chatRoom.broadcast(message: "User \(userId) joined room \(roomId)")
    
    ws.onText { ws, message in
        Task {
            let chatMessage = """
            {"userId": "\(userId)", "message": "\(message)", "timestamp": "\(Date())"}
            """
            await chatRoom.broadcast(message: chatMessage, except: userId)
        }
    }
    
    ws.onClose.whenComplete { _ in
        Task {
            await chatRoom.disconnect(id: userId)
            await chatRoom.broadcast(message: "User \(userId) left")
        }
    }
}
```

---

## 24. File Serving

### Static Files

```swift
// ใน configure.swift
// เสิร์ฟไฟล์จาก Public directory
app.middleware.use(FileMiddleware(publicDirectory: app.directory.publicDirectory))
```

### Stream File

```swift
// ส่งไฟล์เป็น Response
app.get("files", "**") { req async throws -> Response in
    let path = req.parameters.getCatchall().joined(separator: "/")
    let filePath = app.directory.publicDirectory + path
    
    return req.fileio.streamFile(at: filePath)
}
```

### File Download

```swift
app.get("download", ":filename") { req async throws -> Response in
    let filename = req.parameters.get("filename")!
    let filePath = app.directory.publicDirectory + "downloads/" + filename
    
    // ตรวจสอบว่าไฟล์มีอยู่
    guard FileManager.default.fileExists(atPath: filePath) else {
        throw Abort(.notFound)
    }
    
    let response = req.fileio.streamFile(at: filePath)
    response.headers.contentDisposition = .init(.attachment, filename: filename)
    return response
}
```

---

## 25. JWT Authentication

### ตั้งค่า JWT

```swift
// Package.swift
.package(url: "https://github.com/vapor/jwt.git", from: "4.2.2"),

// configure.swift
import JWT

// กำหนด Secret Key
let jwks = try JWKSigner(jwks: JWKS(keys: [
    JWK.rsa(
        .RS256,
        identifier: "mykey",
        modulus: "...",
        exponent: "..."
    )
]))

// หรือใช้ HMAC (ง่ายกว่า)
app.jwt.signers.use(.hs256(key: Environment.get("JWT_SECRET") ?? "secret"))
```

### สร้างและตรวจสอบ JWT

```swift
// JWT Payload
struct UserPayload: JWTPayload {
    let subject: SubjectClaim
    let expiration: ExpirationClaim
    let userId: UUID
    let role: String
    
    enum CodingKeys: String, CodingKey {
        case subject = "sub"
        case expiration = "exp"
        case userId = "user_id"
        case role
    }
    
    func verify(using signer: JWTSigner) throws {
        try expiration.verifyNotExpired()
    }
}

// สร้าง Token
func generateToken(for user: User, req: Request) throws -> String {
    let payload = UserPayload(
        subject: .init(value: user.email),
        expiration: .init(value: Date().addingTimeInterval(60 * 60 * 24)),  // 24 hours
        userId: try user.requireID(),
        role: user.role
    )
    return try req.jwt.sign(payload)
}

// Login Endpoint
app.post("auth", "login") { req async throws -> LoginResponse in
    let credentials = try req.content.decode(LoginCredentials.self)
    
    guard let user = try await User.query(on: req.db)
        .filter(\.$email == credentials.email)
        .first() else {
        throw Abort(.unauthorized, reason: "Invalid credentials")
    }
    
    guard try user.verify(password: credentials.password) else {
        throw Abort(.unauthorized, reason: "Invalid credentials")
    }
    
    let token = try generateToken(for: user, req: req)
    return LoginResponse(token: token, user: UserDTO(user: user))
}

// Protected endpoint ที่ต้องการ JWT
struct JWTAuthenticator: AsyncBearerAuthenticator {
    func authenticate(bearer: BearerAuthorization, for request: Request) async throws {
        let payload = try request.jwt.verify(bearer.token, as: UserPayload.self)
        
        guard let user = try await User.find(payload.userId, on: request.db) else {
            throw Abort(.unauthorized)
        }
        
        request.auth.login(user)
    }
}

let jwtProtected = app.grouped(JWTAuthenticator())
jwtProtected.get("profile") { req async throws -> UserDTO in
    let user = try req.auth.require(User.self)
    return UserDTO(user: user)
}
```

---

## 26. Leaf Templating

### ตั้งค่า Leaf

```swift
// Package.swift
.package(url: "https://github.com/vapor/leaf.git", from: "4.2.4"),

// configure.swift
import Leaf

app.views.use(.leaf)
app.leaf.cache.isEnabled = app.environment.isRelease
```

### สร้าง Template

```html
<!-- Resources/Views/index.leaf -->
<!DOCTYPE html>
<html>
<head>
    <title>#(title)</title>
</head>
<body>
    <h1>#(title)</h1>
    
    #if(users.count > 0):
    <ul>
        #for(user in users):
        <li>
            <strong>#(user.name)</strong> - #(user.email)
        </li>
        #endfor
    </ul>
    #else:
    <p>No users found</p>
    #endif
    
    <!-- Include partial template -->
    #extend("base"):
        #export("content"):
            <p>Main content here</p>
        #endexport
    #endextend
</body>
</html>
```

### Render Template

```swift
struct IndexContext: Encodable {
    let title: String
    let users: [User]
}

app.get { req async throws -> View in
    let users = try await User.query(on: req.db).all()
    let context = IndexContext(title: "User List", users: users)
    return try await req.view.render("index", context)
}
```

---

## 27. Testing Vapor Apps

### XCTVapor

```swift
import XCTVapor
@testable import App

final class UserControllerTests: XCTestCase {
    var app: Application!
    
    override func setUp() async throws {
        app = try await Application.make(.testing)
        try await configure(app)
        // ใช้ in-memory database สำหรับ testing
        app.databases.use(.sqlite(.memory), as: .sqlite)
        try await app.autoMigrate()
    }
    
    override func tearDown() async throws {
        try await app.autoRevert()
        await app.asyncShutdown()
        app = nil
    }
    
    func testCreateUser() async throws {
        let createDTO = CreateUserDTO(
            name: "John Doe",
            email: "john@example.com",
            password: "password123"
        )
        
        try await app.test(.POST, "users", beforeRequest: { req in
            try req.content.encode(createDTO)
        }, afterResponse: { res in
            XCTAssertEqual(res.status, .created)
            
            let user = try res.content.decode(UserResponse.self)
            XCTAssertEqual(user.name, "John Doe")
            XCTAssertEqual(user.email, "john@example.com")
        })
    }
    
    func testGetUser() async throws {
        // สร้าง user ก่อน
        let user = User(name: "Jane", email: "jane@example.com", passwordHash: "hash")
        try await user.save(on: app.db)
        
        try await app.test(.GET, "users/\(user.id!)") { res in
            XCTAssertEqual(res.status, .ok)
            
            let fetchedUser = try res.content.decode(UserResponse.self)
            XCTAssertEqual(fetchedUser.id, user.id)
            XCTAssertEqual(fetchedUser.name, "Jane")
        }
    }
    
    func testGetNonExistentUser() async throws {
        try await app.test(.GET, "users/\(UUID())") { res in
            XCTAssertEqual(res.status, .notFound)
        }
    }
    
    func testDeleteUser() async throws {
        let user = User(name: "Bob", email: "bob@example.com", passwordHash: "hash")
        try await user.save(on: app.db)
        
        // Authenticate
        let token = try await UserToken.generate(for: user, on: app.db)
        
        try await app.test(
            .DELETE,
            "users/\(user.id!)",
            headers: ["Authorization": "Bearer \(token.value)"]
        ) { res in
            XCTAssertEqual(res.status, .noContent)
        }
        
        // ตรวจสอบว่าถูกลบแล้ว
        let deletedUser = try await User.find(user.id, on: app.db)
        XCTAssertNil(deletedUser)
    }
}
```

---

## 28. Deployment to Heroku

### ขั้นตอนการ Deploy

```bash
# 1. ติดตั้ง Heroku CLI
brew install heroku/brew/heroku

# 2. Login
heroku login

# 3. สร้าง App
heroku create my-vapor-app

# 4. เพิ่ม PostgreSQL
heroku addons:create heroku-postgresql:mini

# 5. Set Buildpack
heroku buildpacks:set vapor/vapor

# 6. Set Environment Variables
heroku config:set JWT_SECRET=your-secret-key
heroku config:set LOG_LEVEL=info

# 7. Deploy
git push heroku main
```

### Procfile

```
# Procfile
web: App serve --env production --port $PORT --hostname 0.0.0.0
```

### heroku.yml (ใช้ Docker)

```yaml
build:
  docker:
    web: Dockerfile

run:
  web: App serve --env production --port $PORT --hostname 0.0.0.0
```

---

## 29. Docker for Vapor

### Dockerfile

```dockerfile
# Build stage
FROM swift:5.9 as build

WORKDIR /build

# Copy Package files first (for caching)
COPY ./Package.* ./
RUN swift package resolve

# Copy source code
COPY ./Sources ./Sources
COPY ./Tests ./Tests

# Build release
RUN swift build -c release --disable-sandbox

# Production stage
FROM ubuntu:22.04

# Install runtime dependencies
RUN apt-get -q update && apt-get -q dist-upgrade -y && \
    apt-get -q install -y libssl-dev libcurl4 && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Copy built binary
COPY --from=build /build/.build/release/App .

# Copy resources
COPY ./Public ./Public
COPY ./Resources ./Resources

EXPOSE 8080

ENTRYPOINT ["./App"]
CMD ["serve", "--env", "production", "--hostname", "0.0.0.0", "--port", "8080"]
```

### docker-compose.yml

```yaml
version: '3.9'

services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      DATABASE_HOST: db
      DATABASE_NAME: vapor_app
      DATABASE_USERNAME: vapor
      DATABASE_PASSWORD: password
      LOG_LEVEL: info
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:15
    environment:
      POSTGRES_DB: vapor_app
      POSTGRES_USER: vapor
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    volumes:
      - db_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U vapor"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  db_data:
```

```bash
# รัน ด้วย Docker Compose
docker-compose up --build

# Run migrations
docker-compose exec app ./App migrate

# Logs
docker-compose logs -f app
```

---

## 30. Building a REST API with Vapor

### Project: Todo List API

เราจะสร้าง REST API สำหรับ Todo List ที่ครบถ้วน

#### Models

```swift
// Sources/App/Models/Todo.swift
import Fluent
import Vapor

final class Todo: Model, Content {
    static let schema = "todos"
    
    @ID(key: .id)
    var id: UUID?
    
    @Field(key: "title")
    var title: String
    
    @Field(key: "description")
    var description: String?
    
    @Field(key: "is_completed")
    var isCompleted: Bool
    
    @Field(key: "priority")
    var priority: Priority
    
    @OptionalField(key: "due_date")
    var dueDate: Date?
    
    @Parent(key: "user_id")
    var user: User
    
    @Timestamp(key: "created_at", on: .create)
    var createdAt: Date?
    
    @Timestamp(key: "updated_at", on: .update)
    var updatedAt: Date?
    
    init() {}
    
    init(id: UUID? = nil, title: String, description: String? = nil,
         isCompleted: Bool = false, priority: Priority = .medium,
         dueDate: Date? = nil, userID: User.IDValue) {
        self.id = id
        self.title = title
        self.description = description
        self.isCompleted = isCompleted
        self.priority = priority
        self.dueDate = dueDate
        self.$user.id = userID
    }
}

extension Todo {
    enum Priority: String, Codable {
        case low, medium, high
    }
}
```

#### Migrations

```swift
// Sources/App/Migrations/CreateTodo.swift
import Fluent

struct CreateTodo: AsyncMigration {
    func prepare(on database: Database) async throws {
        try await database.enum("priority")
            .case("low")
            .case("medium")
            .case("high")
            .create()
        
        let priorityType = try await database.enum("priority").read()
        
        try await database.schema("todos")
            .id()
            .field("title", .string, .required)
            .field("description", .string)
            .field("is_completed", .bool, .required, .sql(.default(false)))
            .field("priority", priorityType, .required, .sql(.default("medium")))
            .field("due_date", .datetime)
            .field("user_id", .uuid, .required, .references("users", "id", onDelete: .cascade))
            .field("created_at", .datetime)
            .field("updated_at", .datetime)
            .create()
    }
    
    func revert(on database: Database) async throws {
        try await database.schema("todos").delete()
        try await database.enum("priority").delete()
    }
}
```

#### DTOs

```swift
// Sources/App/DTOs/TodoDTO.swift
import Vapor

struct CreateTodoDTO: Content, Validatable {
    let title: String
    let description: String?
    let priority: Todo.Priority?
    let dueDate: Date?
    
    static func validations(_ validations: inout Validations) {
        validations.add("title", as: String.self, is: .count(1...200))
    }
}

struct UpdateTodoDTO: Content {
    let title: String?
    let description: String?
    let isCompleted: Bool?
    let priority: Todo.Priority?
    let dueDate: Date?
}

struct TodoResponse: Content {
    let id: UUID
    let title: String
    let description: String?
    let isCompleted: Bool
    let priority: Todo.Priority
    let dueDate: Date?
    let createdAt: Date?
    let updatedAt: Date?
    
    init(todo: Todo) throws {
        self.id = try todo.requireID()
        self.title = todo.title
        self.description = todo.description
        self.isCompleted = todo.isCompleted
        self.priority = todo.priority
        self.dueDate = todo.dueDate
        self.createdAt = todo.createdAt
        self.updatedAt = todo.updatedAt
    }
}
```

#### Controller

```swift
// Sources/App/Controllers/TodoController.swift
import Fluent
import Vapor

struct TodoController: RouteCollection {
    func boot(routes: RoutesBuilder) throws {
        let todos = routes.grouped("todos")
        let protected = todos.grouped(UserToken.authenticator(), User.guardMiddleware())
        
        protected.get(use: index)
        protected.post(use: create)
        protected.group(":todoId") { todo in
            todo.get(use: show)
            todo.put(use: update)
            todo.delete(use: delete)
            todo.post("complete", use: markComplete)
        }
    }
    
    // GET /todos - ดึง todos ของ user ที่ login
    func index(req: Request) async throws -> [TodoResponse] {
        let user = try req.auth.require(User.self)
        
        var query = Todo.query(on: req.db)
            .filter(\.$user.$id == (try user.requireID()))
        
        // Filter by completion status
        if let isCompleted = req.query[Bool.self, at: "completed"] {
            query = query.filter(\.$isCompleted == isCompleted)
        }
        
        // Filter by priority
        if let priority = req.query[String.self, at: "priority"],
           let priorityEnum = Todo.Priority(rawValue: priority) {
            query = query.filter(\.$priority == priorityEnum)
        }
        
        // Sort
        let sortField = req.query[String.self, at: "sort"] ?? "created_at"
        let sortOrder = req.query[String.self, at: "order"] ?? "desc"
        
        switch sortField {
        case "title":
            query = sortOrder == "asc" ? query.sort(\.$title, .ascending) : query.sort(\.$title, .descending)
        case "due_date":
            query = sortOrder == "asc" ? query.sort(\.$dueDate, .ascending) : query.sort(\.$dueDate, .descending)
        default:
            query = sortOrder == "asc" ? query.sort(\.$createdAt, .ascending) : query.sort(\.$createdAt, .descending)
        }
        
        let todos = try await query.all()
        return try todos.map { try TodoResponse(todo: $0) }
    }
    
    // POST /todos
    func create(req: Request) async throws -> Response {
        let user = try req.auth.require(User.self)
        try CreateTodoDTO.validate(content: req)
        let dto = try req.content.decode(CreateTodoDTO.self)
        
        let todo = Todo(
            title: dto.title,
            description: dto.description,
            priority: dto.priority ?? .medium,
            dueDate: dto.dueDate,
            userID: try user.requireID()
        )
        try await todo.save(on: req.db)
        
        let response = Response(status: .created)
        try response.content.encode(TodoResponse(todo: todo))
        return response
    }
    
    // GET /todos/:todoId
    func show(req: Request) async throws -> TodoResponse {
        let user = try req.auth.require(User.self)
        let todo = try await findTodo(req: req, for: user)
        return try TodoResponse(todo: todo)
    }
    
    // PUT /todos/:todoId
    func update(req: Request) async throws -> TodoResponse {
        let user = try req.auth.require(User.self)
        let todo = try await findTodo(req: req, for: user)
        let dto = try req.content.decode(UpdateTodoDTO.self)
        
        if let title = dto.title { todo.title = title }
        if let description = dto.description { todo.description = description }
        if let isCompleted = dto.isCompleted { todo.isCompleted = isCompleted }
        if let priority = dto.priority { todo.priority = priority }
        if let dueDate = dto.dueDate { todo.dueDate = dueDate }
        
        try await todo.save(on: req.db)
        return try TodoResponse(todo: todo)
    }
    
    // DELETE /todos/:todoId
    func delete(req: Request) async throws -> HTTPStatus {
        let user = try req.auth.require(User.self)
        let todo = try await findTodo(req: req, for: user)
        try await todo.delete(on: req.db)
        return .noContent
    }
    
    // POST /todos/:todoId/complete
    func markComplete(req: Request) async throws -> TodoResponse {
        let user = try req.auth.require(User.self)
        let todo = try await findTodo(req: req, for: user)
        todo.isCompleted = true
        try await todo.save(on: req.db)
        return try TodoResponse(todo: todo)
    }
    
    // Helper
    private func findTodo(req: Request, for user: User) async throws -> Todo {
        guard let todoId = req.parameters.get("todoId", as: UUID.self) else {
            throw Abort(.badRequest, reason: "Invalid todo ID")
        }
        
        guard let todo = try await Todo.query(on: req.db)
            .filter(\.$id == todoId)
            .filter(\.$user.$id == (try user.requireID()))
            .first() else {
            throw Abort(.notFound, reason: "Todo not found")
        }
        
        return todo
    }
}
```

---

## 31. Practical Exercises

### แบบฝึกหัดที่ 1: Blog API

สร้าง API สำหรับ Blog ที่มีฟีเจอร์:
- User registration และ login
- สร้าง/แก้ไข/ลบ Posts
- Comment system
- Tag system
- Search posts

```swift
// แนวทาง Solution

// Models
final class BlogPost: Model, Content {
    static let schema = "blog_posts"
    
    @ID(key: .id) var id: UUID?
    @Field(key: "title") var title: String
    @Field(key: "slug") var slug: String
    @Field(key: "content") var content: String
    @Field(key: "excerpt") var excerpt: String?
    @Field(key: "is_published") var isPublished: Bool
    @Field(key: "views") var views: Int
    
    @Parent(key: "author_id") var author: User
    @Children(for: \.$post) var comments: [Comment]
    @Siblings(through: PostTag.self, from: \.$post, to: \.$tag) var tags: [Tag]
    
    @Timestamp(key: "created_at", on: .create) var createdAt: Date?
    @Timestamp(key: "updated_at", on: .update) var updatedAt: Date?
    @Timestamp(key: "published_at", on: .none) var publishedAt: Date?
    
    init() {}
}

final class Comment: Model, Content {
    static let schema = "comments"
    
    @ID(key: .id) var id: UUID?
    @Field(key: "content") var content: String
    @Field(key: "is_approved") var isApproved: Bool
    
    @Parent(key: "post_id") var post: BlogPost
    @Parent(key: "author_id") var author: User
    
    @Timestamp(key: "created_at", on: .create) var createdAt: Date?
    
    init() {}
}

// Controller
struct BlogController: RouteCollection {
    func boot(routes: RoutesBuilder) throws {
        let posts = routes.grouped("posts")
        
        // Public routes
        posts.get(use: list)
        posts.get(":slug", use: show)
        
        // Protected routes
        let protected = posts.grouped(UserToken.authenticator(), User.guardMiddleware())
        protected.post(use: create)
        protected.group(":postId") { post in
            post.put(use: update)
            post.delete(use: delete)
            post.post("publish", use: publish)
            post.post("comments", use: addComment)
        }
    }
    
    func list(req: Request) async throws -> Page<BlogPostResponse> {
        return try await BlogPost.query(on: req.db)
            .filter(\.$isPublished == true)
            .with(\.$author)
            .with(\.$tags)
            .sort(\.$publishedAt, .descending)
            .paginate(for: req)
    }
    
    func show(req: Request) async throws -> BlogPostResponse {
        let slug = req.parameters.get("slug")!
        
        guard let post = try await BlogPost.query(on: req.db)
            .filter(\.$slug == slug)
            .filter(\.$isPublished == true)
            .with(\.$author)
            .with(\.$tags)
            .with(\.$comments) { comment in
                comment.with(\.$author)
            }
            .first() else {
            throw Abort(.notFound)
        }
        
        // เพิ่ม view count
        post.views += 1
        try await post.save(on: req.db)
        
        return BlogPostResponse(post: post)
    }
    
    func create(req: Request) async throws -> BlogPostResponse {
        let user = try req.auth.require(User.self)
        let dto = try req.content.decode(CreateBlogPostDTO.self)
        
        let slug = dto.title
            .lowercased()
            .replacingOccurrences(of: " ", with: "-")
            .replacingOccurrences(of: "[^a-z0-9-]", with: "", options: .regularExpression)
        
        let post = BlogPost()
        post.title = dto.title
        post.slug = "\(slug)-\(UUID().uuidString.prefix(8))"
        post.content = dto.content
        post.excerpt = dto.excerpt
        post.isPublished = false
        post.views = 0
        post.$author.id = try user.requireID()
        
        try await post.save(on: req.db)
        
        // เพิ่ม tags
        if let tagNames = dto.tags {
            for tagName in tagNames {
                let tag = try await Tag.query(on: req.db)
                    .filter(\.$name == tagName)
                    .first() ?? {
                        let newTag = Tag(name: tagName)
                        try await newTag.save(on: req.db)
                        return newTag
                    }()
                try await post.$tags.attach(tag, on: req.db)
            }
        }
        
        return BlogPostResponse(post: post)
    }
    
    func publish(req: Request) async throws -> BlogPostResponse {
        let user = try req.auth.require(User.self)
        
        guard let postId = req.parameters.get("postId", as: UUID.self),
              let post = try await BlogPost.query(on: req.db)
                .filter(\.$id == postId)
                .filter(\.$author.$id == (try user.requireID()))
                .first() else {
            throw Abort(.notFound)
        }
        
        post.isPublished = true
        post.publishedAt = Date()
        try await post.save(on: req.db)
        
        return BlogPostResponse(post: post)
    }
}
```

### แบบฝึกหัดที่ 2: E-Commerce API

```swift
// ระบบ Shopping Cart
struct CartController: RouteCollection {
    func boot(routes: RoutesBuilder) throws {
        let cart = routes.grouped("cart")
        let protected = cart.grouped(UserToken.authenticator(), User.guardMiddleware())
        
        protected.get(use: getCart)
        protected.post("items", use: addItem)
        protected.put("items", ":itemId", use: updateQuantity)
        protected.delete("items", ":itemId", use: removeItem)
        protected.post("checkout", use: checkout)
    }
    
    func getCart(req: Request) async throws -> CartResponse {
        let user = try req.auth.require(User.self)
        let userId = try user.requireID()
        
        let cartItems = try await CartItem.query(on: req.db)
            .filter(\.$user.$id == userId)
            .with(\.$product)
            .all()
        
        let total = cartItems.reduce(0.0) { $0 + ($1.product.price * Double($1.quantity)) }
        
        return CartResponse(
            items: cartItems.map { CartItemResponse(item: $0) },
            total: total,
            itemCount: cartItems.reduce(0) { $0 + $1.quantity }
        )
    }
    
    func addItem(req: Request) async throws -> CartResponse {
        let user = try req.auth.require(User.self)
        let dto = try req.content.decode(AddToCartDTO.self)
        
        // ตรวจสอบ product มีอยู่
        guard let product = try await Product.find(dto.productId, on: req.db) else {
            throw Abort(.notFound, reason: "Product not found")
        }
        
        // ตรวจสอบ stock
        guard product.stock >= dto.quantity else {
            throw Abort(.badRequest, reason: "Insufficient stock")
        }
        
        let userId = try user.requireID()
        
        // เช็คว่ามี cart item อยู่แล้วหรือเปล่า
        if let existingItem = try await CartItem.query(on: req.db)
            .filter(\.$user.$id == userId)
            .filter(\.$product.$id == dto.productId)
            .first() {
            existingItem.quantity += dto.quantity
            try await existingItem.save(on: req.db)
        } else {
            let cartItem = CartItem(
                quantity: dto.quantity,
                userID: userId,
                productID: dto.productId
            )
            try await cartItem.save(on: req.db)
        }
        
        return try await getCart(req: req)
    }
    
    func checkout(req: Request) async throws -> OrderResponse {
        let user = try req.auth.require(User.self)
        let userId = try user.requireID()
        
        let cartItems = try await CartItem.query(on: req.db)
            .filter(\.$user.$id == userId)
            .with(\.$product)
            .all()
        
        guard !cartItems.isEmpty else {
            throw Abort(.badRequest, reason: "Cart is empty")
        }
        
        // ตรวจสอบ stock ทั้งหมดก่อน
        for item in cartItems {
            guard item.product.stock >= item.quantity else {
                throw Abort(.conflict, reason: "\(item.product.name) มีจำนวนไม่เพียงพอ")
            }
        }
        
        // สร้าง Order
        let order = Order(
            status: .pending,
            total: cartItems.reduce(0.0) { $0 + ($1.product.price * Double($1.quantity)) },
            userID: userId
        )
        try await order.save(on: req.db)
        
        // สร้าง Order Items และ ลด Stock
        for item in cartItems {
            let orderItem = OrderItem(
                quantity: item.quantity,
                price: item.product.price,
                orderID: try order.requireID(),
                productID: try item.product.requireID()
            )
            try await orderItem.save(on: req.db)
            
            // ลด stock
            item.product.stock -= item.quantity
            try await item.product.save(on: req.db)
            
            // ลบออกจาก cart
            try await item.delete(on: req.db)
        }
        
        return OrderResponse(order: order)
    }
}
```

---

## 32. Summary

ในบทนี้เราได้เรียนรู้:

1. **Vapor Framework** - Web framework สำหรับ Server-Side Swift ที่มีประสิทธิภาพสูง
2. **Routing** - กำหนด URL paths และ HTTP methods ด้วย Route Groups และ Collections
3. **Middleware** - เพิ่ม logic ที่ทำงานก่อน/หลัง Route Handlers
4. **Fluent ORM** - จัดการ Database ด้วย Models, Migrations, และ Relations
5. **Authentication** - ระบบ Basic Auth, Token Auth, และ JWT
6. **Async/Await** - ใช้ Modern Swift Concurrency
7. **WebSockets** - Real-time communication
8. **Testing** - เขียน Test ด้วย XCTVapor
9. **Deployment** - Deploy ไปยัง Heroku และใช้ Docker

### Key Concepts

```swift
// สรุป Pattern ที่สำคัญ

// 1. Route Collection
struct MyController: RouteCollection {
    func boot(routes: RoutesBuilder) throws {
        routes.get("endpoint", use: handler)
    }
    
    func handler(req: Request) async throws -> Response {
        // logic
    }
}

// 2. Fluent Model
final class MyModel: Model, Content {
    static let schema = "table_name"
    @ID(key: .id) var id: UUID?
    @Field(key: "field") var field: String
    init() {}
}

// 3. Protected Routes
let protected = app.grouped(
    UserToken.authenticator(),
    User.guardMiddleware()
)
protected.get("protected") { req async throws -> Response in
    let user = try req.auth.require(User.self)
    // ...
}

// 4. Error Handling
enum AppError: AbortError {
    case notFound
    var status: HTTPResponseStatus { .notFound }
    var reason: String { "Not found" }
}
```

### ขั้นตอนถัดไป

- ศึกษา Vapor Advanced Topics (Job Queues, Caching)
- เรียนรู้ OpenAPI/Swagger integration
- ลองสร้าง Microservices Architecture
- ศึกษา GraphQL ด้วย Vapor

---

*จบ Part 55 - Vapor Server-Side Swift*
