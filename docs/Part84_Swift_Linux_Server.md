# ตอนที่ 84: Swift บน Linux และ Server-Side Development

## การพัฒนา Swift สำหรับ Server และ Linux

---

## 1. Swift บน Linux Setup

### 1.1 การติดตั้ง Swift Toolchain บน Ubuntu/Debian

```bash
# Ubuntu 22.04 LTS
# ดาวน์โหลด Swift toolchain จาก swift.org

# วิธีที่ 1: ผ่าน apt (Ubuntu 24.04+)
sudo apt-get install swift

# วิธีที่ 2: ติดตั้งแบบ manual
# 1. ดาวน์โหลด toolchain
wget https://download.swift.org/swift-5.10-release/ubuntu2204/swift-5.10-RELEASE/swift-5.10-RELEASE-ubuntu22.04.tar.gz

# 2. ตรวจสอบ checksum
wget https://download.swift.org/swift-5.10-release/ubuntu2204/swift-5.10-RELEASE/swift-5.10-RELEASE-ubuntu22.04.tar.gz.sig

# 3. แตกไฟล์
tar xzf swift-5.10-RELEASE-ubuntu22.04.tar.gz

# 4. ย้ายไปยัง /opt
sudo mv swift-5.10-RELEASE-ubuntu22.04 /opt/swift

# 5. เพิ่มใน PATH
echo 'export PATH="/opt/swift/usr/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# ติดตั้ง dependencies
sudo apt-get install -y \
    binutils \
    git \
    gnupg2 \
    libc6-dev \
    libcurl4-openssl-dev \
    libedit2 \
    libgcc-11-dev \
    libpython3-dev \
    libsqlite3-0 \
    libstdc++-11-dev \
    libxml2-dev \
    libz3-dev \
    pkg-config \
    python3-lldb-13 \
    tzdata \
    unzip \
    zlib1g-dev

# ตรวจสอบการติดตั้ง
swift --version
# Swift version 5.10 (swift-5.10-RELEASE)
```

### 1.2 Docker กับ Swift

```dockerfile
# Dockerfile สำหรับ Swift app

# Multi-stage build
# Stage 1: Build
FROM swift:5.10-jammy AS builder

WORKDIR /app

# Copy Package.swift ก่อนเพื่อ cache dependencies
COPY Package.swift Package.resolved ./
RUN swift package resolve

# Copy source code
COPY Sources ./Sources
COPY Tests ./Tests

# Build สำหรับ production
RUN swift build -c release --static-swift-stdlib

# Stage 2: Runtime
FROM ubuntu:22.04

# Install runtime dependencies
RUN apt-get update && apt-get install -y \
    libcurl4 \
    libxml2 \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Copy binary จาก builder
COPY --from=builder /app/.build/release/MyServer .

# Expose port
EXPOSE 8080

# Run
CMD ["./MyServer"]
```

```yaml
# docker-compose.yml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - DATABASE_URL=postgresql://user:password@db:5432/mydb
      - LOG_LEVEL=info
    depends_on:
      db:
        condition: service_healthy
    
  db:
    image: postgres:16-alpine
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=mydb
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d mydb"]
      interval: 5s
      timeout: 5s
      retries: 5

volumes:
  pgdata:
```

### 1.3 Swift Package Manager บน Linux

```swift
// Package.swift
// swift-tools-version: 5.10

import PackageDescription

let package = Package(
    name: "MyLinuxServer",
    platforms: [
        .macOS(.v13)
        // Linux ไม่ต้องระบุ platform
    ],
    products: [
        .executable(name: "MyServer", targets: ["MyServer"]),
        .library(name: "MyServerLib", targets: ["MyServerLib"])
    ],
    dependencies: [
        .package(url: "https://github.com/hummingbird-project/hummingbird.git", from: "2.0.0"),
        .package(url: "https://github.com/apple/swift-log.git", from: "1.5.0"),
        .package(url: "https://github.com/apple/swift-argument-parser.git", from: "1.3.0")
    ],
    targets: [
        .executableTarget(
            name: "MyServer",
            dependencies: [
                "MyServerLib",
                .product(name: "ArgumentParser", package: "swift-argument-parser")
            ]
        ),
        .target(
            name: "MyServerLib",
            dependencies: [
                .product(name: "Hummingbird", package: "hummingbird"),
                .product(name: "Logging", package: "swift-log")
            ]
        ),
        .testTarget(
            name: "MyServerLibTests",
            dependencies: ["MyServerLib"]
        )
    ]
)
```

### 1.4 Foundation บน Linux vs Apple Platforms

```swift
// Foundation บน Linux ใช้ swift-corelibs-foundation
// ความแตกต่างที่สำคัญ:

import Foundation

// 1. FileManager - ส่วนใหญ่เหมือนกัน
let fm = FileManager.default
let currentDir = fm.currentDirectoryPath

// 2. Date/Calendar - เหมือนกัน
let date = Date()
let calendar = Calendar.current
let components = calendar.dateComponents([.year, .month, .day], from: date)

// 3. URLSession - เหมือนกัน แต่บน Linux ต้องใช้ async/await
Task {
    let url = URL(string: "https://api.example.com/data")!
    let (data, response) = try await URLSession.shared.data(from: url)
    print("Received \(data.count) bytes")
}

// 4. JSON Encoding/Decoding - เหมือนกัน
struct Config: Codable {
    let host: String
    let port: Int
    let debug: Bool
}

let config = Config(host: "localhost", port: 8080, debug: true)
let encoder = JSONEncoder()
encoder.outputFormatting = .prettyPrinted
let jsonData = try encoder.encode(config)
print(String(data: jsonData, encoding: .utf8)!)

// 5. บน Linux ไม่มี:
// - NSAppleScript
// - NSUserActivity บางส่วน
// - AppKit/UIKit
// - CoreData (ใช้ SQLite หรือ third-party แทน)

// 6. การอ่าน Environment Variables
func getEnvVar(_ name: String, default defaultValue: String = "") -> String {
    return ProcessInfo.processInfo.environment[name] ?? defaultValue
}

let databaseURL = getEnvVar("DATABASE_URL", default: "postgresql://localhost/mydb")
let port = Int(getEnvVar("PORT", default: "8080")) ?? 8080
let logLevel = getEnvVar("LOG_LEVEL", default: "info")
```

---

## 2. Server-Side Swift Ecosystem

### 2.1 Vapor (ทบทวนสั้น)

```swift
// Vapor เป็น full-featured web framework
// ครอบคลุมใน Part55 แล้ว - ทบทวนโดยย่อ

import Vapor

// เพิ่มใน Package.swift:
// .package(url: "https://github.com/vapor/vapor.git", from: "4.92.0")

// entrypoint
@main
struct VaporApp {
    static func main() async throws {
        var env = try Environment.detect()
        try LoggingSystem.bootstrap(from: &env)
        
        let app = Application(env)
        defer { app.shutdown() }
        
        try configure(app)
        try await app.runFromAsyncMainEntrypoint()
    }
}

func configure(_ app: Application) throws {
    // Database
    app.databases.use(.postgres(
        hostname: Environment.get("DB_HOST") ?? "localhost",
        username: Environment.get("DB_USER") ?? "postgres",
        password: Environment.get("DB_PASS") ?? "",
        database: Environment.get("DB_NAME") ?? "vapor_db"
    ), as: .psql)
    
    // Routes
    try routes(app)
}

func routes(_ app: Application) throws {
    app.get("hello") { req async throws -> String in
        "Hello, World!"
    }
}
```

### 2.2 Hummingbird - Lightweight Alternative

Hummingbird เป็น framework ที่เบากว่า Vapor เหมาะสำหรับ microservices:

```swift
// Package.swift
// .package(url: "https://github.com/hummingbird-project/hummingbird.git", from: "2.0.0")

import Hummingbird
import Logging

@main
struct HummingbirdApp {
    static func main() async throws {
        // สร้าง router
        let router = Router()
        
        router.get("/") { request, context in
            "Hello, Hummingbird!"
        }
        
        router.get("/health") { request, context in
            return Response(
                status: .ok,
                headers: HTTPFields(),
                body: .init(byteBuffer: ByteBuffer(string: #"{"status":"ok"}"#))
            )
        }
        
        // สร้าง app
        let app = Application(
            router: router,
            configuration: .init(
                address: .hostname("0.0.0.0", port: 8080)
            )
        )
        
        // รัน server
        try await app.runService()
    }
}
```

### 2.3 Swift OpenAPI Generator

```swift
// Package.swift
// .package(url: "https://github.com/apple/swift-openapi-generator.git", from: "1.2.0")
// .package(url: "https://github.com/apple/swift-openapi-runtime.git", from: "1.3.0")
// .package(url: "https://github.com/swift-server/swift-openapi-hummingbird.git", from: "2.0.0")

// openapi.yaml
/*
openapi: "3.1.0"
info:
  title: My API
  version: 1.0.0
paths:
  /users:
    get:
      operationId: listUsers
      summary: List all users
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/User'
  /users/{id}:
    get:
      operationId: getUser
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          description: Not found
components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: string
        name:
          type: string
        email:
          type: string
      required: [id, name, email]
*/

// openapi-generator-config.yaml
/*
generate:
  - types
  - server
  - client
*/

// Generated code usage:
import OpenAPIRuntime
import OpenAPIHummingbird

// Implement server protocol
struct MyAPIHandler: APIProtocol {
    func listUsers(_ input: Operations.listUsers.Input) async throws -> Operations.listUsers.Output {
        let users = [
            Components.Schemas.User(id: "1", name: "Alice", email: "alice@example.com"),
            Components.Schemas.User(id: "2", name: "Bob", email: "bob@example.com")
        ]
        return .ok(.init(body: .json(users)))
    }
    
    func getUser(_ input: Operations.getUser.Input) async throws -> Operations.getUser.Output {
        let id = input.path.id
        
        // ค้นหา user
        if id == "1" {
            let user = Components.Schemas.User(id: "1", name: "Alice", email: "alice@example.com")
            return .ok(.init(body: .json(user)))
        } else {
            return .undocumented(statusCode: 404, .init())
        }
    }
}
```

---

## 3. Hummingbird Framework เชิงลึก

### 3.1 Router, Middleware, Request/Response

```swift
import Hummingbird
import Logging

// Router พื้นฐาน
func buildRouter() -> Router<BasicRequestContext> {
    let router = Router(context: BasicRequestContext.self)
    
    // Middleware
    router.middlewares.add(LogRequestsMiddleware(.info))
    router.middlewares.add(CORSMiddleware())
    
    // Routes
    router.get("/") { request, context -> String in
        "Welcome to Hummingbird!"
    }
    
    // Route Groups
    let api = router.group("api/v1")
    api.get("users", use: listUsers)
    api.post("users", use: createUser)
    api.get("users/:id", use: getUser)
    api.put("users/:id", use: updateUser)
    api.delete("users/:id", use: deleteUser)
    
    return router
}

// Handler Functions
func listUsers(request: Request, context: BasicRequestContext) async throws -> [UserResponse] {
    // จำลองข้อมูลจาก database
    return [
        UserResponse(id: "1", name: "Alice", email: "alice@example.com"),
        UserResponse(id: "2", name: "Bob", email: "bob@example.com")
    ]
}

func getUser(request: Request, context: BasicRequestContext) async throws -> UserResponse {
    let id = try context.parameters.require("id")
    
    // ค้นหา user
    guard let user = await UserStore.shared.find(id: id) else {
        throw HTTPError(.notFound, message: "User not found")
    }
    
    return UserResponse(id: user.id, name: user.name, email: user.email)
}

func createUser(request: Request, context: BasicRequestContext) async throws -> Response {
    let body = try await request.decode(as: CreateUserRequest.self, context: context)
    
    // Validate
    guard !body.name.isEmpty else {
        throw HTTPError(.badRequest, message: "Name is required")
    }
    
    guard body.email.contains("@") else {
        throw HTTPError(.badRequest, message: "Invalid email")
    }
    
    let user = User(id: UUID().uuidString, name: body.name, email: body.email)
    await UserStore.shared.save(user)
    
    let response = UserResponse(id: user.id, name: user.name, email: user.email)
    return try Response.created(response, context: context)
}

func updateUser(request: Request, context: BasicRequestContext) async throws -> UserResponse {
    let id = try context.parameters.require("id")
    let body = try await request.decode(as: UpdateUserRequest.self, context: context)
    
    guard var user = await UserStore.shared.find(id: id) else {
        throw HTTPError(.notFound)
    }
    
    if let name = body.name { user.name = name }
    if let email = body.email { user.email = email }
    
    await UserStore.shared.save(user)
    return UserResponse(id: user.id, name: user.name, email: user.email)
}

func deleteUser(request: Request, context: BasicRequestContext) async throws -> HTTPResponse.Status {
    let id = try context.parameters.require("id")
    await UserStore.shared.delete(id: id)
    return .noContent
}
```

### 3.2 Custom Middleware

```swift
import Hummingbird
import Logging

// Authentication Middleware
struct BearerAuthMiddleware<Context: RequestContext>: RouterMiddleware {
    let jwtSecret: String
    
    func handle(
        _ request: Request,
        context: Context,
        next: (Request, Context) async throws -> Response
    ) async throws -> Response {
        // ข้ามการตรวจสอบสำหรับ public routes
        let publicPaths = ["/", "/health", "/api/v1/auth/login", "/api/v1/auth/register"]
        if publicPaths.contains(request.uri.path) {
            return try await next(request, context)
        }
        
        // ตรวจสอบ Authorization header
        guard let authHeader = request.headers[.authorization],
              authHeader.hasPrefix("Bearer ") else {
            throw HTTPError(.unauthorized, message: "Missing or invalid token")
        }
        
        let token = String(authHeader.dropFirst("Bearer ".count))
        
        // Verify JWT token
        guard let userId = try? verifyJWT(token: token, secret: jwtSecret) else {
            throw HTTPError(.unauthorized, message: "Invalid token")
        }
        
        // เก็บ userId ไว้ใน context สำหรับ handler ที่ตามมา
        // context.userId = userId  // ต้องสร้าง custom context
        
        return try await next(request, context)
    }
    
    private func verifyJWT(token: String, secret: String) throws -> String {
        // Simplified JWT verification
        // ใน production ใช้ JWTKit หรือ library ที่เหมาะสม
        return "user-id-123"
    }
}

// Rate Limiting Middleware
actor RateLimiter {
    private var requestCounts: [String: (count: Int, resetAt: Date)] = [:]
    private let limit: Int
    private let window: TimeInterval
    
    init(limit: Int = 100, window: TimeInterval = 60) {
        self.limit = limit
        self.window = window
    }
    
    func checkLimit(for ip: String) -> Bool {
        let now = Date()
        
        if let existing = requestCounts[ip] {
            if now > existing.resetAt {
                requestCounts[ip] = (count: 1, resetAt: now.addingTimeInterval(window))
                return true
            }
            
            if existing.count >= limit {
                return false
            }
            
            requestCounts[ip] = (count: existing.count + 1, resetAt: existing.resetAt)
            return true
        } else {
            requestCounts[ip] = (count: 1, resetAt: now.addingTimeInterval(window))
            return true
        }
    }
}

struct RateLimitMiddleware<Context: RequestContext>: RouterMiddleware {
    let rateLimiter: RateLimiter
    
    func handle(
        _ request: Request,
        context: Context,
        next: (Request, Context) async throws -> Response
    ) async throws -> Response {
        let ip = request.headers[.xForwardedFor] ?? "unknown"
        
        guard await rateLimiter.checkLimit(for: ip) else {
            throw HTTPError(.tooManyRequests, message: "Rate limit exceeded")
        }
        
        return try await next(request, context)
    }
}

// Logging Middleware
struct RequestLoggingMiddleware<Context: RequestContext>: RouterMiddleware {
    let logger: Logger
    
    func handle(
        _ request: Request,
        context: Context,
        next: (Request, Context) async throws -> Response
    ) async throws -> Response {
        let start = Date()
        let method = request.method.rawValue
        let path = request.uri.path
        
        do {
            let response = try await next(request, context)
            let duration = Date().timeIntervalSince(start) * 1000
            
            logger.info("\(method) \(path) → \(response.status.code) (\(String(format: "%.1f", duration))ms)")
            
            return response
        } catch {
            let duration = Date().timeIntervalSince(start) * 1000
            logger.error("\(method) \(path) → ERROR: \(error) (\(String(format: "%.1f", duration))ms)")
            throw error
        }
    }
}
```

### 3.3 JSON Encoding/Decoding

```swift
import Hummingbird
import Foundation

// Model structs
struct UserResponse: Codable, ResponseCodable {
    let id: String
    let name: String
    let email: String
    let createdAt: Date?
    
    enum CodingKeys: String, CodingKey {
        case id
        case name
        case email
        case createdAt = "created_at"
    }
}

struct CreateUserRequest: Codable, DecodableWithConfiguration {
    let name: String
    let email: String
    let password: String
    
    // Custom validation
    func validate() throws {
        guard name.count >= 2 else {
            throw ValidationError.nameTooShort
        }
        guard email.contains("@") && email.contains(".") else {
            throw ValidationError.invalidEmail
        }
        guard password.count >= 8 else {
            throw ValidationError.passwordTooShort
        }
    }
}

enum ValidationError: Error, CustomStringConvertible {
    case nameTooShort
    case invalidEmail
    case passwordTooShort
    
    var description: String {
        switch self {
        case .nameTooShort: return "Name must be at least 2 characters"
        case .invalidEmail: return "Invalid email format"
        case .passwordTooShort: return "Password must be at least 8 characters"
        }
    }
}

struct UpdateUserRequest: Codable, DecodableWithConfiguration {
    let name: String?
    let email: String?
}

// Error Response
struct ErrorResponse: Codable, ResponseCodable {
    let error: String
    let message: String
    let statusCode: Int
    
    enum CodingKeys: String, CodingKey {
        case error
        case message
        case statusCode = "status_code"
    }
}

// Custom error handling
struct GlobalErrorMiddleware<Context: RequestContext>: RouterMiddleware {
    func handle(
        _ request: Request,
        context: Context,
        next: (Request, Context) async throws -> Response
    ) async throws -> Response {
        do {
            return try await next(request, context)
        } catch let httpError as HTTPError {
            let errorResponse = ErrorResponse(
                error: httpError.status.reasonPhrase,
                message: httpError.message ?? httpError.status.reasonPhrase,
                statusCode: Int(httpError.status.code)
            )
            return try Response.json(errorResponse, status: httpError.status)
        } catch let validationError as ValidationError {
            let errorResponse = ErrorResponse(
                error: "Validation Error",
                message: validationError.description,
                statusCode: 400
            )
            return try Response.json(errorResponse, status: .badRequest)
        } catch {
            let errorResponse = ErrorResponse(
                error: "Internal Server Error",
                message: "An unexpected error occurred",
                statusCode: 500
            )
            return try Response.json(errorResponse, status: .internalServerError)
        }
    }
}
```

### 3.4 Testing Hummingbird Apps

```swift
import XCTest
import HummingbirdTesting
@testable import MyServerLib

final class UserAPITests: XCTestCase {
    
    var app: Application<BasicRequestContext>!
    
    override func setUp() async throws {
        // สร้าง app สำหรับ testing
        let router = buildRouter()
        app = Application(router: router, configuration: .init(address: .hostname("127.0.0.1", port: 0)))
    }
    
    override func tearDown() async throws {
        await app.shutdown()
    }
    
    func testListUsers() async throws {
        try await app.test(.live) { client in
            let response = try await client.execute(uri: "/api/v1/users", method: .get)
            
            XCTAssertEqual(response.status, .ok)
            
            let users = try JSONDecoder().decode([UserResponse].self, from: response.body)
            XCTAssertGreaterThan(users.count, 0)
        }
    }
    
    func testCreateUser() async throws {
        try await app.test(.live) { client in
            let body = CreateUserRequest(
                name: "Charlie",
                email: "charlie@example.com",
                password: "securepass123"
            )
            let bodyData = try JSONEncoder().encode(body)
            
            let response = try await client.execute(
                uri: "/api/v1/users",
                method: .post,
                headers: [.contentType: "application/json"],
                body: ByteBuffer(data: bodyData)
            )
            
            XCTAssertEqual(response.status, .created)
            
            let createdUser = try JSONDecoder().decode(UserResponse.self, from: response.body)
            XCTAssertEqual(createdUser.name, "Charlie")
            XCTAssertEqual(createdUser.email, "charlie@example.com")
        }
    }
    
    func testCreateUserValidation() async throws {
        try await app.test(.live) { client in
            // ทดสอบ invalid email
            let body = CreateUserRequest(
                name: "Dave",
                email: "not-an-email",
                password: "securepass123"
            )
            let bodyData = try JSONEncoder().encode(body)
            
            let response = try await client.execute(
                uri: "/api/v1/users",
                method: .post,
                headers: [.contentType: "application/json"],
                body: ByteBuffer(data: bodyData)
            )
            
            XCTAssertEqual(response.status, .badRequest)
        }
    }
    
    func testGetNonexistentUser() async throws {
        try await app.test(.live) { client in
            let response = try await client.execute(
                uri: "/api/v1/users/nonexistent-id",
                method: .get
            )
            
            XCTAssertEqual(response.status, .notFound)
        }
    }
    
    func testHealthCheck() async throws {
        try await app.test(.live) { client in
            let response = try await client.execute(uri: "/health", method: .get)
            XCTAssertEqual(response.status, .ok)
        }
    }
}
```

---

## 4. Database Access บน Linux

### 4.1 PostgresNIO - Direct PostgreSQL Client

```swift
// Package.swift
// .package(url: "https://github.com/vapor/postgres-nio.git", from: "1.21.0")

import PostgresNIO
import NIOCore
import NIOPosix
import Logging

// สร้าง connection pool
class DatabasePool {
    private let pool: PostgresConnectionPool
    
    init() async throws {
        let config = PostgresConnection.Configuration(
            host: ProcessInfo.processInfo.environment["DB_HOST"] ?? "localhost",
            port: 5432,
            username: ProcessInfo.processInfo.environment["DB_USER"] ?? "postgres",
            password: ProcessInfo.processInfo.environment["DB_PASS"] ?? "",
            database: ProcessInfo.processInfo.environment["DB_NAME"] ?? "mydb",
            tls: .disable
        )
        
        var poolConfig = PostgresConnectionPool.Configuration(
            maxConnections: 10
        )
        
        self.pool = try await PostgresConnectionPool(
            configuration: config,
            logger: Logger(label: "postgres")
        )
    }
    
    // Execute query
    func query(_ sql: String, _ binds: PostgresBindings = PostgresBindings()) async throws -> PostgresRowSequence {
        try await pool.withConnection { connection in
            try await connection.query(PostgresQuery(unsafeSQL: sql, binds: binds), logger: Logger(label: "db"))
        }
    }
}

// Repository ที่ใช้ PostgresNIO
struct UserRepository {
    private let db: DatabasePool
    
    init(db: DatabasePool) {
        self.db = db
    }
    
    func createTable() async throws {
        let sql = """
            CREATE TABLE IF NOT EXISTS users (
                id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
                name TEXT NOT NULL,
                email TEXT UNIQUE NOT NULL,
                password_hash TEXT NOT NULL,
                created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
                updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
            )
        """
        _ = try await db.query(sql)
    }
    
    func findAll() async throws -> [User] {
        var users: [User] = []
        
        let rows = try await db.query("SELECT id, name, email, created_at FROM users ORDER BY created_at DESC")
        
        for try await row in rows {
            let id = try row.decode(UUID.self, context: .default)
            let name = try row.decode(String.self, context: .default)
            let email = try row.decode(String.self, context: .default)
            
            users.append(User(id: id.uuidString, name: name, email: email))
        }
        
        return users
    }
    
    func find(id: String) async throws -> User? {
        guard let uuid = UUID(uuidString: id) else { return nil }
        
        var binds = PostgresBindings()
        binds.append(uuid, context: .default)
        
        let rows = try await db.query(
            "SELECT id, name, email FROM users WHERE id = $1",
            binds
        )
        
        for try await row in rows {
            let userId = try row.decode(UUID.self, context: .default)
            let name = try row.decode(String.self, context: .default)
            let email = try row.decode(String.self, context: .default)
            return User(id: userId.uuidString, name: name, email: email)
        }
        
        return nil
    }
    
    func create(name: String, email: String, passwordHash: String) async throws -> User {
        var binds = PostgresBindings()
        binds.append(name, context: .default)
        binds.append(email, context: .default)
        binds.append(passwordHash, context: .default)
        
        let rows = try await db.query(
            "INSERT INTO users (name, email, password_hash) VALUES ($1, $2, $3) RETURNING id, name, email",
            binds
        )
        
        for try await row in rows {
            let id = try row.decode(UUID.self, context: .default)
            let returnedName = try row.decode(String.self, context: .default)
            let returnedEmail = try row.decode(String.self, context: .default)
            return User(id: id.uuidString, name: returnedName, email: returnedEmail)
        }
        
        throw DatabaseError.insertFailed
    }
    
    func delete(id: String) async throws {
        guard let uuid = UUID(uuidString: id) else { return }
        
        var binds = PostgresBindings()
        binds.append(uuid, context: .default)
        
        _ = try await db.query("DELETE FROM users WHERE id = $1", binds)
    }
}

enum DatabaseError: Error {
    case insertFailed
    case updateFailed
    case connectionFailed
}

struct User {
    let id: String
    var name: String
    var email: String
}
```

### 4.2 Redis ด้วย RediStack

```swift
// Package.swift
// .package(url: "https://github.com/swift-server/RediStack.git", from: "1.4.0")

import RediStack
import NIOCore
import NIOPosix
import Foundation

class RedisCache {
    private let connection: RedisConnection
    
    init() async throws {
        let eventLoopGroup = MultiThreadedEventLoopGroup(numberOfThreads: 1)
        
        let config = try RedisConnection.Configuration(
            hostname: ProcessInfo.processInfo.environment["REDIS_HOST"] ?? "localhost",
            port: 6379,
            password: ProcessInfo.processInfo.environment["REDIS_PASS"]
        )
        
        self.connection = try await RedisConnection.make(
            configuration: config,
            boundEventLoop: eventLoopGroup.next()
        ).get()
    }
    
    // Set value with TTL
    func set<T: Codable>(_ key: String, value: T, ttl: Int = 3600) async throws {
        let data = try JSONEncoder().encode(value)
        let string = String(data: data, encoding: .utf8)!
        
        _ = try await connection.set(RedisKey(key), to: string).get()
        
        if ttl > 0 {
            _ = try await connection.expire(RedisKey(key), after: .seconds(ttl)).get()
        }
    }
    
    // Get value
    func get<T: Codable>(_ key: String, as type: T.Type) async throws -> T? {
        guard let string = try await connection.get(RedisKey(key)).get(),
              let data = string.string?.data(using: .utf8) else {
            return nil
        }
        
        return try JSONDecoder().decode(T.self, from: data)
    }
    
    // Delete
    func delete(_ key: String) async throws {
        _ = try await connection.delete(RedisKey(key)).get()
    }
    
    // Increment counter
    func increment(_ key: String, by amount: Int = 1) async throws -> Int {
        return try await connection.increment(RedisKey(key), by: amount).get()
    }
    
    // Check existence
    func exists(_ key: String) async throws -> Bool {
        return try await connection.exists(RedisKey(key)).get() > 0
    }
    
    // List operations
    func pushToList(_ key: String, value: String) async throws {
        _ = try await connection.lpush(value, into: RedisKey(key)).get()
    }
    
    func getListRange(_ key: String, start: Int = 0, end: Int = -1) async throws -> [String] {
        let values = try await connection.lrange(from: RedisKey(key), indices: start...end).get()
        return values.compactMap { $0.string }
    }
}

// Cache-aside pattern
class CachedUserRepository {
    private let db: UserRepository
    private let cache: RedisCache
    
    init(db: UserRepository, cache: RedisCache) {
        self.db = db
        self.cache = cache
    }
    
    func find(id: String) async throws -> User? {
        let cacheKey = "user:\(id)"
        
        // ลองอ่านจาก cache ก่อน
        if let cached = try await cache.get(cacheKey, as: User.self) {
            return cached
        }
        
        // ถ้าไม่มีใน cache อ่านจาก database
        guard let user = try await db.find(id: id) else {
            return nil
        }
        
        // บันทึกลง cache
        try await cache.set(cacheKey, value: user, ttl: 300) // 5 นาที
        
        return user
    }
    
    func invalidate(id: String) async throws {
        try await cache.delete("user:\(id)")
    }
}
```

### 4.3 MongoDB ด้วย MongoKitten

```swift
// Package.swift
// .package(url: "https://github.com/orlandos-nl/MongoKitten.git", from: "7.7.0")

import MongoKitten
import Foundation

struct Article: Codable {
    var _id: ObjectId
    var title: String
    var content: String
    var author: String
    var tags: [String]
    var createdAt: Date
    var views: Int
    
    init(title: String, content: String, author: String, tags: [String] = []) {
        self._id = ObjectId()
        self.title = title
        self.content = content
        self.author = author
        self.tags = tags
        self.createdAt = Date()
        self.views = 0
    }
}

class ArticleRepository {
    private let collection: MongoCollection
    
    init(database: MongoDatabase) {
        self.collection = database["articles"]
    }
    
    func create(_ article: Article) async throws {
        try await collection.insertEncoded(article)
    }
    
    func findAll(limit: Int = 20) async throws -> [Article] {
        return try await collection
            .find()
            .sort(["createdAt": .descending])
            .limit(limit)
            .decode(Article.self)
            .drain()
    }
    
    func findByTag(_ tag: String) async throws -> [Article] {
        let query: Document = ["tags": tag]
        
        return try await collection
            .find(query)
            .decode(Article.self)
            .drain()
    }
    
    func findById(_ id: ObjectId) async throws -> Article? {
        let query: Document = ["_id": id]
        return try await collection.findOne(query, as: Article.self)
    }
    
    func incrementViews(id: ObjectId) async throws {
        let query: Document = ["_id": id]
        let update: Document = ["$inc": ["views": 1] as Document]
        try await collection.updateOne(where: query, to: update)
    }
    
    func search(query: String) async throws -> [Article] {
        // Text search (ต้องสร้าง text index ก่อน)
        let textQuery: Document = ["$text": ["$search": query] as Document]
        
        return try await collection
            .find(textQuery)
            .decode(Article.self)
            .drain()
    }
    
    // Aggregation
    func getTopAuthors(limit: Int = 5) async throws -> [(author: String, articleCount: Int)] {
        let pipeline: [Document] = [
            ["$group": ["_id": "$author", "count": ["$sum": 1]] as Document],
            ["$sort": ["count": -1] as Document],
            ["$limit": limit]
        ]
        
        let results = try await collection.aggregate(pipeline).drain()
        
        return results.compactMap { doc in
            guard let author = doc["_id"] as? String,
                  let count = doc["count"] as? Int else { return nil }
            return (author: author, articleCount: count)
        }
    }
}
```

---

## 5. Swift Concurrency บน Server

### 5.1 ServiceLifecycle

```swift
// Package.swift
// .package(url: "https://github.com/swift-server/swift-service-lifecycle.git", from: "2.3.0")

import ServiceLifecycle
import Logging
import Hummingbird

// สร้าง services ที่ implement Service protocol
struct HTTPServerService: Service {
    let app: Application<BasicRequestContext>
    let logger: Logger
    
    func run() async throws {
        logger.info("Starting HTTP server on port 8080")
        try await app.runService()
    }
}

struct BackgroundWorkerService: Service {
    let logger: Logger
    
    func run() async throws {
        logger.info("Starting background worker")
        
        // ทำงาน background ต่อเนื่องจนกว่าจะถูก cancel
        while !Task.isCancelled {
            await processQueue()
            try await Task.sleep(for: .seconds(5))
        }
        
        logger.info("Background worker stopped")
    }
    
    private func processQueue() async {
        // ประมวลผล queue items
        logger.debug("Processing queue...")
    }
}

struct DatabaseMigrationService: Service {
    let db: DatabasePool
    let logger: Logger
    
    func run() async throws {
        logger.info("Running database migrations")
        
        // รัน migrations
        try await runMigrations()
        
        // Migration service เสร็จแล้ว - จบการทำงาน
        logger.info("Migrations completed")
    }
    
    private func runMigrations() async throws {
        // Migration logic
    }
}

// Main entry point
@main
struct ServerMain {
    static func main() async throws {
        var logger = Logger(label: "server")
        logger.logLevel = .info
        
        // สร้าง services
        let router = buildRouter()
        let app = Application(
            router: router,
            configuration: .init(address: .hostname("0.0.0.0", port: 8080))
        )
        
        let httpService = HTTPServerService(app: app, logger: logger)
        let workerService = BackgroundWorkerService(logger: logger)
        
        // ServiceGroup จัดการ lifecycle
        let serviceGroup = ServiceGroup(
            services: [httpService, workerService],
            gracefulShutdownSignals: [.sigterm, .sigint],
            logger: logger
        )
        
        try await serviceGroup.run()
    }
}
```

### 5.2 Graceful Shutdown

```swift
import ServiceLifecycle
import NIOCore

// Custom Service ที่รองรับ graceful shutdown
struct GracefulHTTPService: Service {
    private let host: String
    private let port: Int
    private let logger: Logger
    
    init(host: String = "0.0.0.0", port: Int = 8080, logger: Logger) {
        self.host = host
        self.port = port
        self.logger = logger
    }
    
    func run() async throws {
        logger.info("Server starting on \(host):\(port)")
        
        // รัน server
        try await withGracefulShutdownHandler {
            // Main server loop
            try await startServer()
        } onGracefulShutdown: {
            // Graceful shutdown handler
            logger.info("Graceful shutdown initiated")
            await self.drainConnections()
        }
        
        logger.info("Server stopped")
    }
    
    private func startServer() async throws {
        // Server implementation
    }
    
    private func drainConnections() async {
        logger.info("Draining existing connections...")
        // รอให้ connections ที่มีอยู่เสร็จก่อน
        try? await Task.sleep(for: .seconds(5))
        logger.info("All connections drained")
    }
}
```

### 5.3 Structured Concurrency สำหรับ Services

```swift
import Foundation

// Task Groups สำหรับ parallel processing
func processItems(_ items: [String]) async throws -> [ProcessedItem] {
    return try await withThrowingTaskGroup(of: ProcessedItem.self) { group in
        for item in items {
            group.addTask {
                try await processItem(item)
            }
        }
        
        var results: [ProcessedItem] = []
        for try await result in group {
            results.append(result)
        }
        return results
    }
}

func processItem(_ item: String) async throws -> ProcessedItem {
    // Simulate processing
    try await Task.sleep(for: .milliseconds(100))
    return ProcessedItem(original: item, processed: item.uppercased())
}

struct ProcessedItem {
    let original: String
    let processed: String
}

// Actor สำหรับ shared state
actor ConnectionPool {
    private var connections: [DatabaseConnection] = []
    private let maxSize: Int
    private var waiters: [CheckedContinuation<DatabaseConnection, Error>] = []
    
    init(maxSize: Int = 10) {
        self.maxSize = maxSize
    }
    
    func acquire() async throws -> DatabaseConnection {
        if let connection = connections.popLast() {
            return connection
        }
        
        if connections.count < maxSize {
            return try await createNewConnection()
        }
        
        // รอจนกว่าจะมี connection ว่าง
        return try await withCheckedThrowingContinuation { continuation in
            waiters.append(continuation)
        }
    }
    
    func release(_ connection: DatabaseConnection) {
        if let waiter = waiters.first {
            waiters.removeFirst()
            waiter.resume(returning: connection)
        } else {
            connections.append(connection)
        }
    }
    
    private func createNewConnection() async throws -> DatabaseConnection {
        // สร้าง connection ใหม่
        return DatabaseConnection()
    }
}

class DatabaseConnection {
    func query(_ sql: String) async throws -> [[String: Any]] {
        return []
    }
}
```

---

## 6. gRPC กับ Swift

### 6.1 Protocol Buffers

```protobuf
// user.proto
syntax = "proto3";

package user.v1;

option swift_prefix = "User_";

// User service
service UserService {
    rpc GetUser (GetUserRequest) returns (GetUserResponse);
    rpc ListUsers (ListUsersRequest) returns (ListUsersResponse);
    rpc CreateUser (CreateUserRequest) returns (CreateUserResponse);
    rpc DeleteUser (DeleteUserRequest) returns (DeleteUserResponse);
    
    // Server-side streaming
    rpc WatchUser (WatchUserRequest) returns (stream UserEvent);
    
    // Bidirectional streaming
    rpc Chat (stream ChatMessage) returns (stream ChatMessage);
}

message User {
    string id = 1;
    string name = 2;
    string email = 3;
    int64 created_at = 4;
}

message GetUserRequest {
    string id = 1;
}

message GetUserResponse {
    User user = 1;
}

message ListUsersRequest {
    int32 page = 1;
    int32 page_size = 2;
}

message ListUsersResponse {
    repeated User users = 1;
    int32 total = 2;
    bool has_more = 3;
}

message CreateUserRequest {
    string name = 1;
    string email = 2;
}

message CreateUserResponse {
    User user = 1;
}

message DeleteUserRequest {
    string id = 1;
}

message DeleteUserResponse {
    bool success = 1;
}

message WatchUserRequest {
    string user_id = 1;
}

message UserEvent {
    string user_id = 1;
    string event_type = 2;
    bytes data = 3;
}

message ChatMessage {
    string sender_id = 1;
    string content = 2;
    int64 timestamp = 3;
}
```

### 6.2 SwiftProtobuf Code Generation

```bash
# ติดตั้ง protoc และ plugin
brew install protobuf
brew install swift-protobuf
brew install grpc-swift

# Generate Swift code
protoc user.proto \
    --swift_out=Sources/Generated \
    --grpc-swift_out=Sources/Generated \
    --swift_opt=Visibility=Public \
    --grpc-swift_opt=Visibility=Public
```

### 6.3 GRPC-Swift Server

```swift
// Package.swift dependencies:
// .package(url: "https://github.com/grpc/grpc-swift.git", from: "1.21.0")
// .package(url: "https://github.com/apple/swift-protobuf.git", from: "1.26.0")

import GRPC
import NIOCore
import NIOPosix
import Foundation

// Implement gRPC service
final class UserServiceProvider: User_UserServiceProvider {
    var interceptors: User_UserServiceServerInterceptorFactoryProtocol?
    
    private let repository: UserRepository
    
    init(repository: UserRepository) {
        self.repository = repository
    }
    
    // Unary RPC
    func getUser(
        request: User_GetUserRequest,
        context: StatusOnlyCallContext
    ) -> EventLoopFuture<User_GetUserResponse> {
        let promise = context.eventLoop.makePromise(of: User_GetUserResponse.self)
        
        Task {
            do {
                guard let user = try await repository.find(id: request.id) else {
                    promise.fail(GRPCStatus(code: .notFound, message: "User not found"))
                    return
                }
                
                var protoUser = User_User()
                protoUser.id = user.id
                protoUser.name = user.name
                protoUser.email = user.email
                
                var response = User_GetUserResponse()
                response.user = protoUser
                
                promise.succeed(response)
            } catch {
                promise.fail(error)
            }
        }
        
        return promise.futureResult
    }
    
    // Unary - List Users
    func listUsers(
        request: User_ListUsersRequest,
        context: StatusOnlyCallContext
    ) -> EventLoopFuture<User_ListUsersResponse> {
        let promise = context.eventLoop.makePromise(of: User_ListUsersResponse.self)
        
        Task {
            do {
                let users = try await repository.findAll()
                
                var response = User_ListUsersResponse()
                response.users = users.map { user in
                    var u = User_User()
                    u.id = user.id
                    u.name = user.name
                    u.email = user.email
                    return u
                }
                response.total = Int32(users.count)
                response.hasMore = false
                
                promise.succeed(response)
            } catch {
                promise.fail(error)
            }
        }
        
        return promise.futureResult
    }
    
    // Create User
    func createUser(
        request: User_CreateUserRequest,
        context: StatusOnlyCallContext
    ) -> EventLoopFuture<User_CreateUserResponse> {
        let promise = context.eventLoop.makePromise(of: User_CreateUserResponse.self)
        
        Task {
            do {
                let user = try await repository.create(
                    name: request.name,
                    email: request.email,
                    passwordHash: "hashed_password"
                )
                
                var protoUser = User_User()
                protoUser.id = user.id
                protoUser.name = user.name
                protoUser.email = user.email
                
                var response = User_CreateUserResponse()
                response.user = protoUser
                
                promise.succeed(response)
            } catch {
                promise.fail(error)
            }
        }
        
        return promise.futureResult
    }
    
    // Delete User
    func deleteUser(
        request: User_DeleteUserRequest,
        context: StatusOnlyCallContext
    ) -> EventLoopFuture<User_DeleteUserResponse> {
        let promise = context.eventLoop.makePromise(of: User_DeleteUserResponse.self)
        
        Task {
            do {
                try await repository.delete(id: request.id)
                
                var response = User_DeleteUserResponse()
                response.success = true
                
                promise.succeed(response)
            } catch {
                promise.fail(error)
            }
        }
        
        return promise.futureResult
    }
    
    // Server Streaming RPC
    func watchUser(
        request: User_WatchUserRequest,
        context: StreamingResponseCallContext<User_UserEvent>
    ) -> EventLoopFuture<GRPCStatus> {
        let promise = context.eventLoop.makePromise(of: GRPCStatus.self)
        
        Task {
            // ส่ง events ต่อเนื่อง
            for i in 0...4 {
                var event = User_UserEvent()
                event.userID = request.userID
                event.eventType = "update"
                event.data = "Event \(i)".data(using: .utf8) ?? Data()
                
                _ = context.sendResponse(event)
                try await Task.sleep(for: .seconds(1))
            }
            
            promise.succeed(.ok)
        }
        
        return promise.futureResult
    }
    
    // Bidirectional Streaming
    func chat(context: StreamingResponseCallContext<User_ChatMessage>) -> EventLoopFuture<(StreamEvent<User_ChatMessage>) -> Void> {
        var handler: (StreamEvent<User_ChatMessage>) -> Void = { _ in }
        
        handler = { event in
            switch event {
            case .message(let message):
                // Echo back with prefix
                var response = User_ChatMessage()
                response.senderID = "server"
                response.content = "Echo: \(message.content)"
                response.timestamp = Int64(Date().timeIntervalSince1970)
                
                _ = context.sendResponse(response)
                
            case .end:
                context.statusPromise.succeed(.ok)
            }
        }
        
        return context.eventLoop.makeSucceededFuture(handler)
    }
}

// gRPC Server Setup
func startGRPCServer(repository: UserRepository) async throws {
    let group = MultiThreadedEventLoopGroup(numberOfThreads: System.coreCount)
    defer {
        try? group.syncShutdownGracefully()
    }
    
    let provider = UserServiceProvider(repository: repository)
    
    let server = try Server.insecure(group: group)
        .withServiceProviders([provider])
        .bind(host: "0.0.0.0", port: 50051)
        .wait()
    
    print("gRPC server started on port 50051")
    
    try server.onClose.wait()
}
```

### 6.4 gRPC Client

```swift
import GRPC
import NIOCore
import NIOPosix

class UserGRPCClient {
    private let client: User_UserServiceNIOClient
    
    init(host: String = "localhost", port: Int = 50051) throws {
        let group = MultiThreadedEventLoopGroup(numberOfThreads: 1)
        
        let channel = try GRPCChannelPool.with(
            target: .host(host, port: port),
            transportSecurity: .plaintext,
            eventLoopGroup: group
        )
        
        self.client = User_UserServiceNIOClient(channel: channel)
    }
    
    func getUser(id: String) async throws -> User_User {
        var request = User_GetUserRequest()
        request.id = id
        
        let response = try await client.getUser(request).response.get()
        return response.user
    }
    
    func listUsers() async throws -> [User_User] {
        var request = User_ListUsersRequest()
        request.page = 1
        request.pageSize = 20
        
        let response = try await client.listUsers(request).response.get()
        return response.users
    }
    
    func watchUser(id: String) -> AsyncThrowingStream<User_UserEvent, Error> {
        var request = User_WatchUserRequest()
        request.userID = id
        
        return AsyncThrowingStream { continuation in
            let call = client.watchUser(request) { event in
                continuation.yield(event)
            }
            
            Task {
                do {
                    let status = try await call.status.get()
                    if status.code == .ok {
                        continuation.finish()
                    } else {
                        continuation.finish(throwing: GRPCStatus(code: status.code, message: status.message))
                    }
                } catch {
                    continuation.finish(throwing: error)
                }
            }
        }
    }
}
```

---

## 7. Kafka/Message Queues กับ Swift

### 7.1 swift-kafka-client

```swift
// Package.swift
// .package(url: "https://github.com/swift-server/swift-kafka-client.git", from: "0.5.0")

import Kafka
import ServiceLifecycle
import Logging

// Producer
struct OrderEventProducer: Service {
    private let logger: Logger
    
    func run() async throws {
        var config = KafkaProducerConfiguration()
        config.bootstrapBrokerAddresses = ["localhost:9092"]
        config.messageMaxBytes = 1_000_000
        
        let (producer, events) = try KafkaProducer.makeProducerWithEvents(
            configuration: config,
            logger: logger
        )
        
        // Process acknowledgement events
        async let eventProcessing: () = {
            for await event in events {
                switch event {
                case .deliveryReport(let report):
                    switch report.status {
                    case .acknowledged(let message):
                        logger.debug("Message delivered: \(message.topic) partition \(message.partition)")
                    case .failure(let message, let error):
                        logger.error("Message delivery failed: \(error) for message in \(message.topic)")
                    }
                }
            }
        }()
        
        // Send events
        for i in 0...99 {
            let order = OrderEvent(
                orderId: UUID().uuidString,
                userId: "user-\(i % 10)",
                amount: Double.random(in: 10...1000),
                timestamp: Date()
            )
            
            let data = try JSONEncoder().encode(order)
            
            try producer.send(
                KafkaProducerMessage(
                    topic: "orders",
                    key: KafkaByteBuffer(bytes: Array(order.orderId.utf8)),
                    value: KafkaByteBuffer(bytes: Array(data))
                )
            )
            
            logger.info("Sent order: \(order.orderId)")
            try await Task.sleep(for: .milliseconds(100))
        }
        
        await eventProcessing
    }
}

struct OrderEvent: Codable {
    let orderId: String
    let userId: String
    let amount: Double
    let timestamp: Date
    
    enum CodingKeys: String, CodingKey {
        case orderId = "order_id"
        case userId = "user_id"
        case amount
        case timestamp
    }
}
```

### 7.2 Kafka Consumer

```swift
import Kafka
import ServiceLifecycle
import Logging

// Consumer
struct OrderEventConsumer: Service {
    private let logger: Logger
    private let orderProcessor: OrderProcessor
    
    init(logger: Logger, orderProcessor: OrderProcessor) {
        self.logger = logger
        self.orderProcessor = orderProcessor
    }
    
    func run() async throws {
        var config = KafkaConsumerConfiguration(
            consumptionStrategy: .group(
                id: "order-processing-service",
                topics: ["orders"]
            ),
            bootstrapBrokerAddresses: ["localhost:9092"]
        )
        config.autoOffsetReset = .earliest
        
        let consumer = try KafkaConsumer(
            configuration: config,
            logger: logger
        )
        
        logger.info("Starting order consumer")
        
        // Consume messages
        for await messageResult in consumer.messages {
            switch messageResult {
            case .success(let message):
                await handleMessage(message)
                
                // Commit offset
                try consumer.scheduleCommit(message)
                
            case .failure(let error):
                logger.error("Consumer error: \(error)")
            }
        }
    }
    
    private func handleMessage(_ message: KafkaConsumerMessage) async {
        guard let data = message.value.flatMap({ Data($0.readableBytesView) }) else {
            logger.warning("Empty message received")
            return
        }
        
        do {
            let order = try JSONDecoder().decode(OrderEvent.self, from: data)
            logger.info("Processing order: \(order.orderId)")
            await orderProcessor.process(order)
        } catch {
            logger.error("Failed to decode order: \(error)")
        }
    }
}

// Event-driven microservice
actor OrderProcessor {
    private var processedCount = 0
    private let logger: Logger
    
    init(logger: Logger) {
        self.logger = logger
    }
    
    func process(_ order: OrderEvent) async {
        // ประมวลผล order
        processedCount += 1
        logger.info("Order \(order.orderId) processed (total: \(processedCount))")
        
        // อาจจะส่ง event ต่อ
        // await notificationProducer.send(orderConfirmation)
    }
}
```

---

## 8. Swift Microservices

### 8.1 REST Service Architecture

```swift
import Hummingbird
import Logging
import Foundation

// Complete microservice structure

// Configuration
struct ServiceConfiguration {
    let host: String
    let port: Int
    let databaseURL: String
    let redisURL: String
    let logLevel: Logger.Level
    let serviceName: String
    
    init() {
        let env = ProcessInfo.processInfo.environment
        self.host = env["HOST"] ?? "0.0.0.0"
        self.port = Int(env["PORT"] ?? "8080") ?? 8080
        self.databaseURL = env["DATABASE_URL"] ?? "postgresql://localhost/mydb"
        self.redisURL = env["REDIS_URL"] ?? "redis://localhost:6379"
        self.logLevel = Logger.Level(rawValue: env["LOG_LEVEL"] ?? "info") ?? .info
        self.serviceName = env["SERVICE_NAME"] ?? "user-service"
    }
}

// Health Check
struct HealthStatus: Codable {
    let status: String
    let version: String
    let uptime: Double
    let checks: [String: String]
    
    static let version = "1.0.0"
    static let startTime = Date()
    
    static func current(dbOk: Bool, redisOk: Bool) -> HealthStatus {
        HealthStatus(
            status: dbOk && redisOk ? "healthy" : "degraded",
            version: version,
            uptime: -startTime.timeIntervalSinceNow,
            checks: [
                "database": dbOk ? "ok" : "error",
                "redis": redisOk ? "ok" : "error"
            ]
        )
    }
}

// Dependency Container
final class ServiceContainer {
    let config: ServiceConfiguration
    let logger: Logger
    let db: DatabasePool
    let cache: RedisCache
    let userRepository: UserRepository
    
    init() async throws {
        self.config = ServiceConfiguration()
        
        var logger = Logger(label: config.serviceName)
        logger.logLevel = config.logLevel
        self.logger = logger
        
        self.db = try await DatabasePool()
        self.cache = try await RedisCache()
        self.userRepository = UserRepository(db: db)
    }
}

// Route setup
func configureRoutes(_ router: Router<BasicRequestContext>, container: ServiceContainer) {
    let logger = container.logger
    
    // Health check
    router.get("/health") { request, context -> Response in
        let health = HealthStatus.current(dbOk: true, redisOk: true)
        return try Response.json(health)
    }
    
    router.get("/ready") { request, context -> Response in
        // Readiness check - ตรวจสอบว่าพร้อมรับ traffic
        return Response(status: .ok)
    }
    
    // API routes
    let api = router.group("api/v1")
    api.middlewares.add(RequestLoggingMiddleware(logger: logger))
    
    let users = api.group("users")
    users.get(use: listUsersHandler(container: container))
    users.post(use: createUserHandler(container: container))
    users.get(":id", use: getUserHandler(container: container))
    users.put(":id", use: updateUserHandler(container: container))
    users.delete(":id", use: deleteUserHandler(container: container))
}

// Handlers as free functions
func listUsersHandler(container: ServiceContainer) -> @Sendable (Request, BasicRequestContext) async throws -> Response {
    return { request, context in
        let users = try await container.userRepository.findAll()
        let responses = users.map { UserResponse(id: $0.id, name: $0.name, email: $0.email) }
        return try Response.json(responses)
    }
}

func getUserHandler(container: ServiceContainer) -> @Sendable (Request, BasicRequestContext) async throws -> Response {
    return { request, context in
        let id = try context.parameters.require("id")
        
        guard let user = try await container.userRepository.find(id: id) else {
            throw HTTPError(.notFound, message: "User not found")
        }
        
        let response = UserResponse(id: user.id, name: user.name, email: user.email)
        return try Response.json(response)
    }
}

func createUserHandler(container: ServiceContainer) -> @Sendable (Request, BasicRequestContext) async throws -> Response {
    return { request, context in
        let body = try await request.decode(as: CreateUserRequest.self, context: context)
        try body.validate()
        
        let user = try await container.userRepository.create(
            name: body.name,
            email: body.email,
            passwordHash: "hashed_\(body.password)"
        )
        
        let response = UserResponse(id: user.id, name: user.name, email: user.email)
        return try Response.created(response)
    }
}

func updateUserHandler(container: ServiceContainer) -> @Sendable (Request, BasicRequestContext) async throws -> Response {
    return { request, context in
        let id = try context.parameters.require("id")
        let _ = try await request.decode(as: UpdateUserRequest.self, context: context)
        
        guard let user = try await container.userRepository.find(id: id) else {
            throw HTTPError(.notFound)
        }
        
        let response = UserResponse(id: user.id, name: user.name, email: user.email)
        return try Response.json(response)
    }
}

func deleteUserHandler(container: ServiceContainer) -> @Sendable (Request, BasicRequestContext) async throws -> Response {
    return { request, context in
        let id = try context.parameters.require("id")
        try await container.userRepository.delete(id: id)
        return Response(status: .noContent)
    }
}
```

### 8.2 Kubernetes Deployment

```yaml
# kubernetes/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
  labels:
    app: user-service
    version: "1.0.0"
spec:
  replicas: 3
  selector:
    matchLabels:
      app: user-service
  template:
    metadata:
      labels:
        app: user-service
    spec:
      containers:
      - name: user-service
        image: myregistry/user-service:1.0.0
        ports:
        - containerPort: 8080
        env:
        - name: PORT
          value: "8080"
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: url
        - name: LOG_LEVEL
          value: "info"
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 15
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 10
---
apiVersion: v1
kind: Service
metadata:
  name: user-service
spec:
  selector:
    app: user-service
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP
```

---

## 9. Performance และ Benchmarking

### 9.1 swift-benchmark

```swift
// Package.swift
// .package(url: "https://github.com/google/swift-benchmark.git", from: "0.1.1")

import Benchmark
import Foundation

// Benchmark suite
benchmark("JSON Encoding") {
    let users = (0..<100).map { i in
        UserResponse(id: "\(i)", name: "User \(i)", email: "user\(i)@example.com")
    }
    
    let encoder = JSONEncoder()
    blackHole(try! encoder.encode(users))
}

benchmark("JSON Decoding") {
    let data = """
    [{"id":"1","name":"Alice","email":"alice@example.com"},
     {"id":"2","name":"Bob","email":"bob@example.com"}]
    """.data(using: .utf8)!
    
    let decoder = JSONDecoder()
    blackHole(try! decoder.decode([UserResponse].self, from: data))
}

benchmark("String Concatenation - Loop") {
    var result = ""
    for i in 0..<1000 {
        result += "\(i)"
    }
    blackHole(result)
}

benchmark("String Concatenation - Join") {
    let result = (0..<1000).map { "\($0)" }.joined()
    blackHole(result)
}

benchmark("Dictionary Lookup") {
    let dict = Dictionary(uniqueKeysWithValues: (0..<10000).map { ($0, "value-\($0)") })
    
    for key in 0..<1000 {
        blackHole(dict[key])
    }
}

Benchmark.main()
```

```bash
# รัน benchmarks
swift run -c release MyBenchmarks

# Output ตัวอย่าง:
# name                  time        std        iterations
# -----------------------------------------------------------
# JSON Encoding         1.2 ms      ±5%        100
# JSON Decoding         0.8 ms      ±3%        100
# String Concat Loop    0.5 ms      ±2%        1000
# String Concat Join    0.2 ms      ±1%        1000
```

### 9.2 Memory Profiling บน Linux

```bash
# ใช้ Valgrind สำหรับ memory profiling
valgrind --tool=massif --pages-as-heap=yes ./MyServer

# ใช้ perf สำหรับ CPU profiling
perf record -g ./MyServer
perf report

# ใช้ heaptrack
heaptrack ./MyServer
heaptrack_gui heaptrack.MyServer.12345.gz

# Swift built-in memory debugging
SWIFT_DETERMINISTIC_HASHING=1 ./MyServer
```

```swift
// Memory tracking ใน Swift
class MemoryMonitor {
    static func currentMemoryUsage() -> UInt64 {
        var info = mach_task_basic_info()
        var count = mach_msg_type_number_t(MemoryLayout<mach_task_basic_info>.size / MemoryLayout<natural_t>.size)
        
        let result = withUnsafeMutablePointer(to: &info) {
            $0.withMemoryRebound(to: integer_t.self, capacity: Int(count)) {
                task_info(mach_task_self_, task_flavor_t(MACH_TASK_BASIC_INFO), $0, &count)
            }
        }
        
        return result == KERN_SUCCESS ? info.resident_size : 0
    }
    
    static func logMemoryUsage(label: String = "") {
        let bytes = currentMemoryUsage()
        let megabytes = Double(bytes) / 1_048_576
        print("[\(label)] Memory: \(String(format: "%.2f", megabytes)) MB")
    }
}
```

---

## 10. Swift AWS Lambda

### 10.1 swift-aws-lambda-runtime

```swift
// Package.swift
// .package(url: "https://github.com/swift-server/swift-aws-lambda-runtime.git", from: "1.0.0-alpha")
// .package(url: "https://github.com/swift-server/swift-aws-lambda-events.git", from: "0.4.0")

import AWSLambdaRuntime
import AWSLambdaEvents
import Foundation

// Simple Lambda Handler - String in, String out
@main
struct HelloLambda: SimpleLambdaHandler {
    func handle(_ event: String, context: LambdaContext) async throws -> String {
        context.logger.info("Received event: \(event)")
        return "Hello, \(event)!"
    }
}

// API Gateway Handler
struct APIGatewayHandler: LambdaHandler {
    typealias Event = APIGatewayV2Request
    typealias Output = APIGatewayV2Response
    
    func handle(_ request: APIGatewayV2Request, context: LambdaContext) async throws -> APIGatewayV2Response {
        context.logger.info("Handling request: \(request.requestContext.http.method) \(request.requestContext.http.path)")
        
        let path = request.requestContext.http.path
        let method = request.requestContext.http.method
        
        switch (method, path) {
        case ("GET", "/hello"):
            return APIGatewayV2Response(
                statusCode: .ok,
                headers: ["Content-Type": "application/json"],
                body: #"{"message": "Hello from Lambda!"}"#
            )
            
        case ("POST", "/users"):
            guard let body = request.body,
                  let data = body.data(using: .utf8) else {
                return APIGatewayV2Response(statusCode: .badRequest)
            }
            
            let user = try JSONDecoder().decode(CreateUserRequest.self, from: data)
            
            // บันทึก user
            let response = UserResponse(id: UUID().uuidString, name: user.name, email: user.email)
            let responseData = try JSONEncoder().encode(response)
            
            return APIGatewayV2Response(
                statusCode: .created,
                headers: ["Content-Type": "application/json"],
                body: String(data: responseData, encoding: .utf8)
            )
            
        default:
            return APIGatewayV2Response(statusCode: .notFound)
        }
    }
}

// SQS Event Handler
struct SQSEventHandler: LambdaHandler {
    typealias Event = SQSEvent
    typealias Output = SQSBatchResponse
    
    func handle(_ event: SQSEvent, context: LambdaContext) async throws -> SQSBatchResponse {
        var failures: [SQSBatchResponse.BatchItemFailure] = []
        
        for record in event.records {
            do {
                try await processMessage(record.body, logger: context.logger)
            } catch {
                context.logger.error("Failed to process message \(record.messageId): \(error)")
                failures.append(SQSBatchResponse.BatchItemFailure(itemIdentifier: record.messageId))
            }
        }
        
        return SQSBatchResponse(batchItemFailures: failures)
    }
    
    private func processMessage(_ body: String, logger: Logger) async throws {
        guard let data = body.data(using: .utf8) else {
            throw LambdaError.invalidMessage
        }
        
        let order = try JSONDecoder().decode(OrderEvent.self, from: data)
        logger.info("Processing order: \(order.orderId)")
        
        // ประมวลผล order
    }
}

enum LambdaError: Error {
    case invalidMessage
    case processingFailed(String)
}

// DynamoDB Event Handler
struct DynamoDBStreamHandler: LambdaHandler {
    typealias Event = DynamoDBEvent
    typealias Output = Void
    
    func handle(_ event: DynamoDBEvent, context: LambdaContext) async throws {
        for record in event.records {
            switch record.eventName {
            case "INSERT":
                if let newImage = record.dynamodb?.newImage {
                    context.logger.info("New item inserted: \(newImage)")
                }
            case "MODIFY":
                context.logger.info("Item modified")
            case "REMOVE":
                context.logger.info("Item removed")
            default:
                break
            }
        }
    }
}
```

### 10.2 Lambda Deployment ด้วย SAM

```yaml
# template.yaml (AWS SAM)
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31

Description: Swift Lambda Functions

Globals:
  Function:
    Runtime: provided.al2
    Architectures:
      - arm64
    Timeout: 30
    MemorySize: 256
    Environment:
      Variables:
        LOG_LEVEL: info

Resources:
  HelloFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: .build/release/HelloLambda
      Handler: bootstrap
      Events:
        ApiEvent:
          Type: HttpApi
          Properties:
            Path: /hello
            Method: GET

  UserAPIFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: .build/release/APIGatewayHandler
      Handler: bootstrap
      Policies:
        - DynamoDBCrudPolicy:
            TableName: !Ref UsersTable
      Events:
        GetUsers:
          Type: HttpApi
          Properties:
            Path: /users
            Method: GET
        CreateUser:
          Type: HttpApi
          Properties:
            Path: /users
            Method: POST

  OrderProcessorFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: .build/release/SQSEventHandler
      Handler: bootstrap
      Events:
        SQSEvent:
          Type: SQS
          Properties:
            Queue: !GetAtt OrderQueue.Arn
            BatchSize: 10
            FunctionResponseTypes:
              - ReportBatchItemFailures

  OrderQueue:
    Type: AWS::SQS::Queue
    Properties:
      VisibilityTimeout: 60

  UsersTable:
    Type: AWS::DynamoDB::Table
    Properties:
      AttributeDefinitions:
        - AttributeName: id
          AttributeType: S
      KeySchema:
        - AttributeName: id
          KeyType: HASH
      BillingMode: PAY_PER_REQUEST
```

```bash
# Build และ deploy
# ติดตั้ง SAM CLI
brew install aws/tap/aws-sam-cli

# Build สำหรับ Lambda (Linux ARM64)
sam build

# Test locally
sam local invoke HelloFunction --event events/hello.json

# Deploy
sam deploy --guided

# Build script
#!/bin/bash
docker run --rm \
  --volume "$(pwd):/src" \
  --workdir "/src" \
  swift:5.10-amazonlinux2 \
  swift build -c release --product HelloLambda --static-swift-stdlib
```

---

## 11. CI/CD สำหรับ Swift บน Linux

### 11.1 GitHub Actions สำหรับ Linux

```yaml
# .github/workflows/swift-linux.yml
name: Swift Linux CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test-linux:
    name: Test on Linux
    runs-on: ubuntu-22.04
    
    strategy:
      matrix:
        swift-version: ['5.9', '5.10']
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Swift
      uses: swift-actions/setup-swift@v2
      with:
        swift-version: ${{ matrix.swift-version }}
    
    - name: Cache SPM dependencies
      uses: actions/cache@v3
      with:
        path: .build
        key: ${{ runner.os }}-spm-${{ hashFiles('**/Package.resolved') }}
        restore-keys: |
          ${{ runner.os }}-spm-
    
    - name: Build
      run: swift build -v
    
    - name: Run tests
      run: swift test -v --enable-test-discovery
    
    - name: Run tests with sanitizers
      run: |
        swift test -Xswiftc -sanitize=thread
        swift test -Xswiftc -sanitize=address

  build-docker:
    name: Build Docker Image
    runs-on: ubuntu-22.04
    needs: test-linux
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3
    
    - name: Login to Container Registry
      uses: docker/login-action@v3
      with:
        registry: ghcr.io
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
    
    - name: Build and push
      uses: docker/build-push-action@v5
      with:
        context: .
        push: ${{ github.ref == 'refs/heads/main' }}
        tags: ghcr.io/${{ github.repository }}:latest
        cache-from: type=gha
        cache-to: type=gha,mode=max
    
  deploy:
    name: Deploy to Kubernetes
    runs-on: ubuntu-22.04
    needs: build-docker
    if: github.ref == 'refs/heads/main'
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Configure kubectl
      uses: azure/k8s-set-context@v3
      with:
        method: kubeconfig
        kubeconfig: ${{ secrets.KUBE_CONFIG }}
    
    - name: Deploy
      run: |
        kubectl set image deployment/user-service \
          user-service=ghcr.io/${{ github.repository }}:latest
        kubectl rollout status deployment/user-service
```

### 11.2 Optimized Multi-Stage Dockerfile

```dockerfile
# Dockerfile.optimized

# Stage 1: Dependency resolution (cached separately)
FROM swift:5.10-jammy AS deps-resolver

WORKDIR /app
COPY Package.swift Package.resolved ./
RUN swift package resolve

# Stage 2: Build
FROM swift:5.10-jammy AS builder

WORKDIR /app

# Copy resolved dependencies
COPY --from=deps-resolver /app/.build /app/.build
COPY --from=deps-resolver /app/Package.swift ./
COPY --from=deps-resolver /app/Package.resolved ./

# Copy source
COPY Sources ./Sources
COPY Tests ./Tests

# Build release binary with optimizations
RUN swift build -c release \
    -Xswiftc -O \
    -Xswiftc -whole-module-optimization \
    --static-swift-stdlib

# Run tests
RUN swift test -c release

# Stage 3: Minimal runtime image
FROM ubuntu:22.04 AS runtime

# Install only necessary runtime libs
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      ca-certificates \
      libcurl4 \
      libxml2 \
      tzdata && \
    rm -rf /var/lib/apt/lists/* && \
    update-ca-certificates

# Create non-root user
RUN useradd --create-home --shell /bin/bash appuser

WORKDIR /home/appuser/app

# Copy binary
COPY --from=builder /app/.build/release/MyServer .
COPY --chown=appuser:appuser --from=builder /app/.build/release/MyServer .

USER appuser

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8080/health || exit 1

CMD ["./MyServer", "--hostname", "0.0.0.0", "--port", "8080"]
```

---

## 12. Complete Project: REST API Microservice กับ PostgreSQL

### 12.1 Project Structure

```
UserService/
├── Package.swift
├── Sources/
│   └── UserService/
│       ├── main.swift
│       ├── App/
│       │   ├── Application.swift
│       │   ├── Configuration.swift
│       │   └── Routes.swift
│       ├── Models/
│       │   ├── User.swift
│       │   ├── Requests.swift
│       │   └── Responses.swift
│       ├── Repositories/
│       │   ├── UserRepository.swift
│       │   └── DatabasePool.swift
│       ├── Services/
│       │   └── UserService.swift
│       └── Middleware/
│           ├── AuthMiddleware.swift
│           └── LoggingMiddleware.swift
├── Tests/
│   └── UserServiceTests/
│       ├── UserRepositoryTests.swift
│       └── UserAPITests.swift
├── Dockerfile
├── docker-compose.yml
└── kubernetes/
    ├── deployment.yaml
    └── service.yaml
```

### 12.2 Complete Application Code

```swift
// Sources/UserService/main.swift
import ServiceLifecycle
import Logging
import Foundation

@main
struct UserServiceMain {
    static func main() async throws {
        // Setup logging
        LoggingSystem.bootstrap { label in
            var handler = StreamLogHandler.standardOutput(label: label)
            handler.logLevel = .info
            return handler
        }
        
        let logger = Logger(label: "user-service")
        logger.info("Starting User Service...")
        
        do {
            let app = try await UserServiceApp()
            try await app.run()
        } catch {
            logger.error("Failed to start: \(error)")
            exit(1)
        }
    }
}

// Sources/UserService/App/Application.swift
import Hummingbird
import ServiceLifecycle
import Logging

final class UserServiceApp {
    private let container: ServiceContainer
    private let logger: Logger
    
    init() async throws {
        self.container = try await ServiceContainer()
        self.logger = container.logger
    }
    
    func run() async throws {
        // Setup database
        let userRepo = container.userRepository
        try await userRepo.createTable()
        logger.info("Database tables ready")
        
        // Setup router
        let router = Router<BasicRequestContext>()
        
        // Middlewares
        router.middlewares.add(CORSMiddleware(
            allowedOrigin: .all,
            allowedMethods: [.get, .post, .put, .delete, .options],
            allowedHeaders: [.contentType, .authorization]
        ))
        router.middlewares.add(RequestLoggingMiddleware(logger: logger))
        
        // Routes
        setupRoutes(router)
        
        // Create application
        let app = Application(
            router: router,
            configuration: .init(
                address: .hostname(container.config.host, port: container.config.port)
            ),
            logger: logger
        )
        
        // Service group
        let serviceGroup = ServiceGroup(
            services: [app],
            gracefulShutdownSignals: [.sigterm, .sigint],
            logger: logger
        )
        
        logger.info("Server listening on \(container.config.host):\(container.config.port)")
        try await serviceGroup.run()
    }
    
    private func setupRoutes(_ router: Router<BasicRequestContext>) {
        // Health
        router.get("/health") { _, _ in
            HealthStatus.current(dbOk: true, redisOk: true)
        }
        
        router.get("/ready") { _, _ in HTTPResponse.Status.ok }
        
        // API
        let api = router.group("api/v1")
        
        // Users
        let users = api.group("users")
        users.get(use: UserHandlers.list(container: container))
        users.post(use: UserHandlers.create(container: container))
        users.get(":id", use: UserHandlers.getById(container: container))
        users.put(":id", use: UserHandlers.update(container: container))
        users.delete(":id", use: UserHandlers.delete(container: container))
    }
}

// Sources/UserService/App/Routes.swift
import Hummingbird

struct UserHandlers {
    static func list(container: ServiceContainer) -> @Sendable (Request, BasicRequestContext) async throws -> [UserResponse] {
        return { _, _ in
            let users = try await container.userRepository.findAll()
            return users.map { UserResponse(from: $0) }
        }
    }
    
    static func getById(container: ServiceContainer) -> @Sendable (Request, BasicRequestContext) async throws -> UserResponse {
        return { request, context in
            let id = try context.parameters.require("id")
            
            guard let user = try await container.userRepository.find(id: id) else {
                throw HTTPError(.notFound, message: "User with id '\(id)' not found")
            }
            
            return UserResponse(from: user)
        }
    }
    
    static func create(container: ServiceContainer) -> @Sendable (Request, BasicRequestContext) async throws -> Response {
        return { request, context in
            let body = try await request.decode(as: CreateUserRequest.self, context: context)
            try body.validate()
            
            // Check email uniqueness
            let existing = try await container.userRepository.findByEmail(body.email)
            if existing != nil {
                throw HTTPError(.conflict, message: "Email already exists")
            }
            
            let user = try await container.userRepository.create(
                name: body.name,
                email: body.email,
                passwordHash: hashPassword(body.password)
            )
            
            return try Response.created(UserResponse(from: user))
        }
    }
    
    static func update(container: ServiceContainer) -> @Sendable (Request, BasicRequestContext) async throws -> UserResponse {
        return { request, context in
            let id = try context.parameters.require("id")
            let body = try await request.decode(as: UpdateUserRequest.self, context: context)
            
            guard var user = try await container.userRepository.find(id: id) else {
                throw HTTPError(.notFound)
            }
            
            if let name = body.name { user.name = name }
            if let email = body.email { user.email = email }
            
            try await container.userRepository.save(user)
            return UserResponse(from: user)
        }
    }
    
    static func delete(container: ServiceContainer) -> @Sendable (Request, BasicRequestContext) async throws -> HTTPResponse.Status {
        return { request, context in
            let id = try context.parameters.require("id")
            
            guard try await container.userRepository.find(id: id) != nil else {
                throw HTTPError(.notFound)
            }
            
            try await container.userRepository.delete(id: id)
            return .noContent
        }
    }
    
    private static func hashPassword(_ password: String) -> String {
        // ใน production ใช้ bcrypt หรือ Argon2
        return "hashed_\(password)"
    }
}

// Sources/UserService/Models/User.swift
import Foundation

struct User: Codable, Identifiable {
    let id: String
    var name: String
    var email: String
    let createdAt: Date
    var updatedAt: Date
}

// Sources/UserService/Models/Responses.swift
import Foundation

struct UserResponse: Codable, ResponseCodable {
    let id: String
    let name: String
    let email: String
    let createdAt: String
    
    init(from user: User) {
        self.id = user.id
        self.name = user.name
        self.email = user.email
        self.createdAt = ISO8601DateFormatter().string(from: user.createdAt)
    }
}

// Sources/UserService/Models/Requests.swift
struct CreateUserRequest: Codable, DecodableWithConfiguration {
    let name: String
    let email: String
    let password: String
    
    func validate() throws {
        guard name.count >= 2 else {
            throw HTTPError(.badRequest, message: "Name must be at least 2 characters")
        }
        guard email.contains("@") else {
            throw HTTPError(.badRequest, message: "Invalid email address")
        }
        guard password.count >= 8 else {
            throw HTTPError(.badRequest, message: "Password must be at least 8 characters")
        }
    }
}

struct UpdateUserRequest: Codable, DecodableWithConfiguration {
    let name: String?
    let email: String?
}
```

---

## 13. แบบฝึกหัดพร้อมเฉลย

### แบบฝึกหัดที่ 1: Hummingbird Todo API

**โจทย์:** สร้าง REST API สำหรับ Todo list ด้วย Hummingbird ที่มี endpoints: GET /todos, POST /todos, PUT /todos/:id, DELETE /todos/:id

**เฉลย:**

```swift
import Hummingbird
import Foundation

struct Todo: Codable, Sendable {
    let id: String
    var title: String
    var completed: Bool
    let createdAt: Date
}

// In-memory store (ใน production ใช้ database)
actor TodoStore {
    private var todos: [String: Todo] = [:]
    
    func getAll() -> [Todo] {
        Array(todos.values).sorted { $0.createdAt < $1.createdAt }
    }
    
    func get(id: String) -> Todo? {
        todos[id]
    }
    
    func create(title: String) -> Todo {
        let todo = Todo(id: UUID().uuidString, title: title, completed: false, createdAt: Date())
        todos[todo.id] = todo
        return todo
    }
    
    func update(id: String, title: String?, completed: Bool?) -> Todo? {
        guard var todo = todos[id] else { return nil }
        if let title = title { todo.title = title }
        if let completed = completed { todo.completed = completed }
        todos[id] = todo
        return todo
    }
    
    func delete(id: String) -> Bool {
        return todos.removeValue(forKey: id) != nil
    }
}

struct CreateTodoRequest: Codable, DecodableWithConfiguration {
    let title: String
}

struct UpdateTodoRequest: Codable, DecodableWithConfiguration {
    let title: String?
    let completed: Bool?
}

let store = TodoStore()

let router = Router<BasicRequestContext>()

// List todos
router.get("/todos") { _, _ in
    await store.getAll()
}

// Get single todo
router.get("/todos/:id") { _, context in
    let id = try context.parameters.require("id")
    guard let todo = await store.get(id: id) else {
        throw HTTPError(.notFound)
    }
    return todo
}

// Create todo
router.post("/todos") { request, context -> Response in
    let body = try await request.decode(as: CreateTodoRequest.self, context: context)
    
    guard !body.title.trimmingCharacters(in: .whitespaces).isEmpty else {
        throw HTTPError(.badRequest, message: "Title cannot be empty")
    }
    
    let todo = await store.create(title: body.title)
    return try Response.created(todo)
}

// Update todo
router.put("/todos/:id") { request, context -> Todo in
    let id = try context.parameters.require("id")
    let body = try await request.decode(as: UpdateTodoRequest.self, context: context)
    
    guard let todo = await store.update(id: id, title: body.title, completed: body.completed) else {
        throw HTTPError(.notFound)
    }
    return todo
}

// Delete todo
router.delete("/todos/:id") { _, context -> HTTPResponse.Status in
    let id = try context.parameters.require("id")
    
    guard await store.delete(id: id) else {
        throw HTTPError(.notFound)
    }
    
    return .noContent
}

// Run
let app = Application(router: router)
try await app.runService()
```

### แบบฝึกหัดที่ 2: PostgreSQL Repository

**โจทย์:** สร้าง `ProductRepository` ที่ใช้ PostgresNIO ในการ CRUD สินค้า

**เฉลย:**

```swift
import PostgresNIO
import Foundation

struct Product: Codable {
    let id: UUID
    var name: String
    var price: Double
    var stock: Int
    let createdAt: Date
}

class ProductRepository {
    private let pool: PostgresConnectionPool
    
    init(pool: PostgresConnectionPool) {
        self.pool = pool
    }
    
    func createTable() async throws {
        _ = try await pool.withConnection { conn in
            try await conn.query("""
                CREATE TABLE IF NOT EXISTS products (
                    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
                    name TEXT NOT NULL,
                    price NUMERIC(10,2) NOT NULL,
                    stock INTEGER NOT NULL DEFAULT 0,
                    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
                )
            """, logger: .init(label: "db"))
        }
    }
    
    func findAll() async throws -> [Product] {
        try await pool.withConnection { conn in
            let rows = try await conn.query(
                "SELECT id, name, price, stock, created_at FROM products ORDER BY created_at DESC",
                logger: .init(label: "db")
            )
            
            var products: [Product] = []
            for try await row in rows {
                let id = try row.decode(UUID.self, context: .default)
                let name = try row.decode(String.self, context: .default)
                let price = try row.decode(Double.self, context: .default)
                let stock = try row.decode(Int.self, context: .default)
                let createdAt = try row.decode(Date.self, context: .default)
                
                products.append(Product(id: id, name: name, price: price, stock: stock, createdAt: createdAt))
            }
            return products
        }
    }
    
    func find(id: UUID) async throws -> Product? {
        try await pool.withConnection { conn in
            var binds = PostgresBindings()
            binds.append(id, context: .default)
            
            let rows = try await conn.query(
                "SELECT id, name, price, stock, created_at FROM products WHERE id = $1",
                binds,
                logger: .init(label: "db")
            )
            
            for try await row in rows {
                let id = try row.decode(UUID.self, context: .default)
                let name = try row.decode(String.self, context: .default)
                let price = try row.decode(Double.self, context: .default)
                let stock = try row.decode(Int.self, context: .default)
                let createdAt = try row.decode(Date.self, context: .default)
                
                return Product(id: id, name: name, price: price, stock: stock, createdAt: createdAt)
            }
            return nil
        }
    }
    
    func create(name: String, price: Double, stock: Int) async throws -> Product {
        try await pool.withConnection { conn in
            var binds = PostgresBindings()
            binds.append(name, context: .default)
            binds.append(price, context: .default)
            binds.append(stock, context: .default)
            
            let rows = try await conn.query(
                "INSERT INTO products (name, price, stock) VALUES ($1, $2, $3) RETURNING id, name, price, stock, created_at",
                binds,
                logger: .init(label: "db")
            )
            
            for try await row in rows {
                let id = try row.decode(UUID.self, context: .default)
                let name = try row.decode(String.self, context: .default)
                let price = try row.decode(Double.self, context: .default)
                let stock = try row.decode(Int.self, context: .default)
                let createdAt = try row.decode(Date.self, context: .default)
                
                return Product(id: id, name: name, price: price, stock: stock, createdAt: createdAt)
            }
            
            throw DatabaseError.insertFailed
        }
    }
    
    func updateStock(id: UUID, delta: Int) async throws -> Product? {
        try await pool.withConnection { conn in
            var binds = PostgresBindings()
            binds.append(delta, context: .default)
            binds.append(id, context: .default)
            
            let rows = try await conn.query(
                """
                UPDATE products 
                SET stock = stock + $1 
                WHERE id = $2 AND stock + $1 >= 0
                RETURNING id, name, price, stock, created_at
                """,
                binds,
                logger: .init(label: "db")
            )
            
            for try await row in rows {
                let id = try row.decode(UUID.self, context: .default)
                let name = try row.decode(String.self, context: .default)
                let price = try row.decode(Double.self, context: .default)
                let stock = try row.decode(Int.self, context: .default)
                let createdAt = try row.decode(Date.self, context: .default)
                
                return Product(id: id, name: name, price: price, stock: stock, createdAt: createdAt)
            }
            
            return nil  // Stock ไม่พอ
        }
    }
    
    func delete(id: UUID) async throws {
        try await pool.withConnection { conn in
            var binds = PostgresBindings()
            binds.append(id, context: .default)
            
            _ = try await conn.query(
                "DELETE FROM products WHERE id = $1",
                binds,
                logger: .init(label: "db")
            )
        }
    }
}
```

### แบบฝึกหัดที่ 3: gRPC Calculator Service

**โจทย์:** สร้าง gRPC service สำหรับ Calculator ที่มี operations: Add, Subtract, Multiply, Divide และ streaming CalculateMany

**เฉลย:**

```protobuf
// calculator.proto
syntax = "proto3";
package calculator.v1;

service CalculatorService {
    rpc Calculate(CalculateRequest) returns (CalculateResponse);
    rpc CalculateMany(stream CalculateRequest) returns (stream CalculateResponse);
}

message CalculateRequest {
    double a = 1;
    double b = 2;
    Operation operation = 3;
}

enum Operation {
    ADD = 0;
    SUBTRACT = 1;
    MULTIPLY = 2;
    DIVIDE = 3;
}

message CalculateResponse {
    double result = 1;
    string expression = 2;
    bool success = 3;
    string error = 4;
}
```

```swift
// CalculatorServiceProvider.swift
import GRPC
import NIOCore

final class CalculatorServiceProvider: Calculator_CalculatorServiceProvider {
    var interceptors: Calculator_CalculatorServiceServerInterceptorFactoryProtocol?
    
    func calculate(
        request: Calculator_CalculateRequest,
        context: StatusOnlyCallContext
    ) -> EventLoopFuture<Calculator_CalculateResponse> {
        
        let result = performCalculation(a: request.a, b: request.b, operation: request.operation)
        
        var response = Calculator_CalculateResponse()
        
        switch result {
        case .success(let value):
            response.result = value
            response.success = true
            response.expression = formatExpression(a: request.a, b: request.b, 
                                                    operation: request.operation, result: value)
        case .failure(let error):
            response.success = false
            response.error = error.localizedDescription
        }
        
        return context.eventLoop.makeSucceededFuture(response)
    }
    
    func calculateMany(context: StreamingResponseCallContext<Calculator_CalculateResponse>) 
        -> EventLoopFuture<(StreamEvent<Calculator_CalculateRequest>) -> Void> {
        
        var handler: (StreamEvent<Calculator_CalculateRequest>) -> Void = { _ in }
        
        handler = { event in
            switch event {
            case .message(let request):
                let result = self.performCalculation(a: request.a, b: request.b, operation: request.operation)
                
                var response = Calculator_CalculateResponse()
                switch result {
                case .success(let value):
                    response.result = value
                    response.success = true
                case .failure(let error):
                    response.success = false
                    response.error = error.localizedDescription
                }
                
                _ = context.sendResponse(response)
                
            case .end:
                context.statusPromise.succeed(.ok)
            }
        }
        
        return context.eventLoop.makeSucceededFuture(handler)
    }
    
    private func performCalculation(a: Double, b: Double, operation: Calculator_Operation) -> Result<Double, Error> {
        switch operation {
        case .add:
            return .success(a + b)
        case .subtract:
            return .success(a - b)
        case .multiply:
            return .success(a * b)
        case .divide:
            guard b != 0 else {
                return .failure(CalculatorError.divisionByZero)
            }
            return .success(a / b)
        default:
            return .failure(CalculatorError.unknownOperation)
        }
    }
    
    private func formatExpression(a: Double, b: Double, operation: Calculator_Operation, result: Double) -> String {
        let op: String
        switch operation {
        case .add: op = "+"
        case .subtract: op = "-"
        case .multiply: op = "×"
        case .divide: op = "÷"
        default: op = "?"
        }
        return "\(a) \(op) \(b) = \(result)"
    }
}

enum CalculatorError: Error, LocalizedError {
    case divisionByZero
    case unknownOperation
    
    var errorDescription: String? {
        switch self {
        case .divisionByZero: return "Division by zero"
        case .unknownOperation: return "Unknown operation"
        }
    }
}
```

### แบบฝึกหัดที่ 4: Redis Caching Layer

**โจทย์:** สร้าง caching decorator สำหรับ repository ที่ใช้ Redis ในการ cache results

**เฉลย:**

```swift
import Foundation

// Generic cache protocol
protocol CacheProtocol {
    func get<T: Codable>(_ key: String, as type: T.Type) async throws -> T?
    func set<T: Codable>(_ key: String, value: T, ttl: Int) async throws
    func invalidate(_ key: String) async throws
    func invalidatePattern(_ pattern: String) async throws
}

// Cache-aware repository wrapper
final class CachedRepository<T: Codable & Identifiable> where T.ID == String {
    private let underlying: AnyRepository<T>
    private let cache: CacheProtocol
    private let cachePrefix: String
    private let cacheTTL: Int
    
    init<R: RepositoryProtocol>(
        repository: R,
        cache: CacheProtocol,
        cachePrefix: String,
        ttl: Int = 300
    ) where R.Model == T {
        self.underlying = AnyRepository(repository)
        self.cache = cache
        self.cachePrefix = cachePrefix
        self.cacheTTL = ttl
    }
    
    private func cacheKey(for id: String) -> String {
        "\(cachePrefix):\(id)"
    }
    
    private var allKey: String {
        "\(cachePrefix):all"
    }
    
    func findAll() async throws -> [T] {
        // ลองอ่านจาก cache
        if let cached = try await cache.get(allKey, as: [T].self) {
            return cached
        }
        
        // อ่านจาก database
        let items = try await underlying.findAll()
        
        // บันทึกลง cache
        try await cache.set(allKey, value: items, ttl: cacheTTL)
        
        return items
    }
    
    func find(id: String) async throws -> T? {
        let key = cacheKey(for: id)
        
        // ลองอ่านจาก cache
        if let cached = try await cache.get(key, as: T.self) {
            return cached
        }
        
        // อ่านจาก database
        guard let item = try await underlying.find(id: id) else {
            return nil
        }
        
        // บันทึกลง cache
        try await cache.set(key, value: item, ttl: cacheTTL)
        
        return item
    }
    
    func save(_ item: T) async throws {
        try await underlying.save(item)
        
        // Invalidate cache
        try await cache.invalidate(cacheKey(for: item.id))
        try await cache.invalidate(allKey)
    }
    
    func delete(id: String) async throws {
        try await underlying.delete(id: id)
        
        // Invalidate cache
        try await cache.invalidate(cacheKey(for: id))
        try await cache.invalidate(allKey)
    }
}

// Repository protocol
protocol RepositoryProtocol {
    associatedtype Model: Codable & Identifiable where Model.ID == String
    
    func findAll() async throws -> [Model]
    func find(id: String) async throws -> Model?
    func save(_ item: Model) async throws
    func delete(id: String) async throws
}

// Type-erased repository
struct AnyRepository<T: Codable & Identifiable> where T.ID == String {
    private let _findAll: () async throws -> [T]
    private let _find: (String) async throws -> T?
    private let _save: (T) async throws -> Void
    private let _delete: (String) async throws -> Void
    
    init<R: RepositoryProtocol>(_ repository: R) where R.Model == T {
        self._findAll = repository.findAll
        self._find = repository.find(id:)
        self._save = repository.save
        self._delete = repository.delete(id:)
    }
    
    func findAll() async throws -> [T] { try await _findAll() }
    func find(id: String) async throws -> T? { try await _find(id) }
    func save(_ item: T) async throws { try await _save(item) }
    func delete(id: String) async throws { try await _delete(id) }
}
```

### แบบฝึกหัดที่ 5: AWS Lambda ด้วย DynamoDB

**โจทย์:** สร้าง Lambda function ที่รับ SQS event และบันทึกข้อมูลลง DynamoDB

**เฉลย:**

```swift
import AWSLambdaRuntime
import AWSLambdaEvents
import AWSDynamoDB
import Foundation

struct OrderRecord: Codable {
    let orderId: String
    let userId: String
    let amount: Double
    let status: String
    let processedAt: String
    
    func toDynamoItem() -> [String: AttributeValue] {
        return [
            "orderId": .s(orderId),
            "userId": .s(userId),
            "amount": .n(String(amount)),
            "status": .s(status),
            "processedAt": .s(processedAt)
        ]
    }
}

struct OrderProcessorLambda: LambdaHandler {
    typealias Event = SQSEvent
    typealias Output = SQSBatchResponse
    
    let dynamoDB: DynamoDBClient
    let tableName: String
    
    init(context: LambdaInitializationContext) async throws {
        self.dynamoDB = try DynamoDBClient()
        self.tableName = ProcessInfo.processInfo.environment["TABLE_NAME"] ?? "orders"
    }
    
    func handle(_ event: SQSEvent, context: LambdaContext) async throws -> SQSBatchResponse {
        var failures: [SQSBatchResponse.BatchItemFailure] = []
        
        await withTaskGroup(of: (String, Error?).self) { group in
            for record in event.records {
                group.addTask {
                    do {
                        try await self.processRecord(record, logger: context.logger)
                        return (record.messageId, nil)
                    } catch {
                        return (record.messageId, error)
                    }
                }
            }
            
            for await (messageId, error) in group {
                if let error = error {
                    context.logger.error("Failed to process \(messageId): \(error)")
                    failures.append(.init(itemIdentifier: messageId))
                }
            }
        }
        
        return SQSBatchResponse(batchItemFailures: failures)
    }
    
    private func processRecord(_ record: SQSEvent.Message, logger: Logger) async throws {
        guard let data = record.body.data(using: .utf8) else {
            throw ProcessingError.invalidData
        }
        
        let order = try JSONDecoder().decode(OrderEvent.self, from: data)
        
        let record = OrderRecord(
            orderId: order.orderId,
            userId: order.userId,
            amount: order.amount,
            status: "processed",
            processedAt: ISO8601DateFormatter().string(from: Date())
        )
        
        let input = PutItemInput(
            item: record.toDynamoItem(),
            tableName: tableName
        )
        
        _ = try await dynamoDB.putItem(input: input)
        
        logger.info("Saved order \(order.orderId) to DynamoDB")
    }
}

enum ProcessingError: Error {
    case invalidData
    case saveFailed
}

@main
struct Main {
    static func main() async throws {
        let runtime = LambdaRuntime.init { context in
            try await OrderProcessorLambda(context: context)
        }
        try await runtime.run()
    }
}
```

---

## สรุป

การพัฒนา Swift บน Linux และ server-side development เปิดโลกใหม่สำหรับ Swift developers:

1. **Swift บน Linux** - ทำงานได้เต็มประสิทธิภาพด้วย swift-corelibs-foundation ครอบคลุมส่วนใหญ่ของ Foundation API

2. **Hummingbird** - เป็น framework ที่เบา ทันสมัย และรองรับ Swift Concurrency อย่างสมบูรณ์ เหมาะสำหรับ microservices

3. **Database Access** - PostgresNIO สำหรับ PostgreSQL, RediStack สำหรับ Redis, MongoKitten สำหรับ MongoDB ล้วนรองรับ async/await

4. **gRPC** - GRPC-Swift ช่วยสร้าง high-performance APIs พร้อม streaming support

5. **Kafka** - swift-kafka-client สำหรับ event-driven architecture

6. **ServiceLifecycle** - จัดการ lifecycle ของ services อย่างเป็นระบบ พร้อม graceful shutdown

7. **AWS Lambda** - swift-aws-lambda-runtime ช่วยสร้าง serverless functions บน AWS

8. **CI/CD** - GitHub Actions + Docker multi-stage builds สำหรับ production deployments

Swift บน server มีประสิทธิภาพสูง type-safe และมี ecosystem ที่เติบโตอย่างต่อเนื่อง ทำให้เป็นตัวเลือกที่น่าสนใจสำหรับ backend development โดยเฉพาะทีมที่มีประสบการณ์ iOS/macOS development อยู่แล้ว
