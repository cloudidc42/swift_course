# Part 70: การสร้าง Framework ของตัวเอง

## บทนำ

Framework คือชุดโค้ดที่ reusable ซึ่งออกแบบมาเพื่อแก้ปัญหาเฉพาะด้านและสามารถแชร์ให้กับนักพัฒนาคนอื่นได้ การสร้าง Framework ที่ดีนั้นต้องการทักษะพิเศษที่ต่างจากการสร้าง app ทั่วไป เพราะต้องคำนึงถึง API design, backward compatibility, documentation, และการ distribution บทนี้จะสอนทุกอย่างที่จำเป็นในการสร้าง professional-grade Swift framework

---

## 1. การวางแผน Framework

### 1.1 ตั้งคำถามก่อนเริ่ม

ก่อนเริ่มเขียนโค้ด ต้องตอบคำถามสำคัญ:

**Who is your audience?**
- นักพัฒนาภายในบริษัทเดียวกัน (Internal SDK)
- Open source community
- Third-party developers ที่ต้องซื้อ license

**What problem does it solve?**
- อธิบายปัญหาได้ใน 1-2 ประโยค
- มี alternatives อยู่แล้วไหม? ทำไมของเราดีกว่า?

**What are the constraints?**
- Minimum iOS version ที่รองรับ
- Dependencies ที่ยอมให้มีได้
- License (MIT, Apache, Commercial)

### 1.2 Framework Charter

สร้าง document กำหนดขอบเขต:

```markdown
# Framework Name - Charter

## Problem Statement
อธิบายปัญหาที่ framework นี้แก้

## Goals
- Goal 1: ...
- Goal 2: ...

## Non-Goals
- ไม่รองรับ X
- ไม่จัดการ Y

## Target Audience
นักพัฒนา iOS ที่ต้องการ Z

## Success Metrics
- Easy to integrate (< 5 minutes setup)
- Minimal API surface
- Comprehensive documentation
```

### 1.3 API Design Principles

**Principle 1: Minimal Surface Area**
เปิดเผยเฉพาะสิ่งที่จำเป็น

```swift
// ไม่ดี - เปิดเผยมากเกินไป
public class NetworkManager {
    public var session: URLSession  // ไม่ควร expose
    public var queue: DispatchQueue  // ไม่ควร expose
    public var decoder: JSONDecoder  // ไม่ควร expose
    
    public func fetch<T: Decodable>(from url: URL, 
                                    completion: @escaping (Result<T, Error>) -> Void) { }
}

// ดีกว่า - hide implementation details
public class NetworkManager {
    private var session: URLSession
    private var queue: DispatchQueue
    private var decoder: JSONDecoder
    
    public init(configuration: NetworkConfiguration = .default) { 
        // ...
    }
    
    public func fetch<T: Decodable>(from url: URL) async throws -> T { }
}
```

**Principle 2: Progressive Disclosure**
API ง่ายสำหรับ common cases, flexible สำหรับ advanced

```swift
// Level 1: Simple (90% of use cases)
let result = try await Networking.get("https://api.example.com/users")

// Level 2: With options (9% of use cases)
let client = NetworkClient(baseURL: "https://api.example.com")
let result: [User] = try await client.get("/users", headers: ["Auth": "token"])

// Level 3: Full control (1% of use cases)
var request = URLRequest(url: url)
request.httpMethod = "GET"
let result: [User] = try await client.perform(request)
```

**Principle 3: Fail Fast**

```swift
// Framework ควร detect misuse ตั้งแต่ compile time
public struct Configuration {
    public let apiKey: String
    
    public init(apiKey: String) {
        precondition(!apiKey.isEmpty, "API key cannot be empty")
        self.apiKey = apiKey
    }
}

// ใช้ @available เพื่อ guide users
@available(*, deprecated, renamed: "fetchAsync")
public func fetch(completion: @escaping (Result<Data, Error>) -> Void) { }

public func fetchAsync() async throws -> Data { }
```

---

## 2. Public API Design

### 2.1 Naming Conventions

```swift
// ✅ ดี - ชื่อ descriptive และ consistent
public protocol Authenticatable {
    func authenticate(with credentials: Credentials) async throws -> AuthToken
    func signOut() async throws
}

public struct Credentials {
    public let username: String
    public let password: String
    
    public init(username: String, password: String) {
        self.username = username
        self.password = password
    }
}

// ❌ ไม่ดี - ชื่อ vague หรือ inconsistent
public protocol Auth {
    func login(u: String, p: String, cb: @escaping (Any?, Any?) -> Void)
    func doSignOut()
}
```

### 2.2 Error Design

```swift
// สร้าง Error types ที่ specific และ informative
public enum NetworkError: Error, LocalizedError {
    case invalidURL(String)
    case noInternetConnection
    case serverError(statusCode: Int, message: String?)
    case decodingFailed(underlyingError: Error)
    case timeout(after: TimeInterval)
    case unauthorized
    case rateLimited(retryAfter: TimeInterval?)
    
    public var errorDescription: String? {
        switch self {
        case .invalidURL(let url):
            return "Invalid URL: \(url)"
        case .noInternetConnection:
            return "No internet connection. Please check your network settings."
        case .serverError(let code, let message):
            return "Server error \(code): \(message ?? "Unknown error")"
        case .decodingFailed(let error):
            return "Failed to decode response: \(error.localizedDescription)"
        case .timeout(let interval):
            return "Request timed out after \(interval) seconds"
        case .unauthorized:
            return "Authentication required. Please sign in."
        case .rateLimited(let retryAfter):
            if let retry = retryAfter {
                return "Rate limited. Please retry after \(retry) seconds."
            }
            return "Rate limited. Please try again later."
        }
    }
    
    public var recoverySuggestion: String? {
        switch self {
        case .noInternetConnection:
            return "Check Wi-Fi or cellular connection"
        case .unauthorized:
            return "Sign in to continue"
        case .rateLimited(let retryAfter):
            if let retry = retryAfter {
                return "Wait \(Int(retry)) seconds before retrying"
            }
            return "Wait before retrying"
        default:
            return nil
        }
    }
}
```

### 2.3 Result Types

```swift
// ใช้ Result type สำหรับ synchronous operations ที่อาจ fail
public func parseJSON<T: Decodable>(_ data: Data) -> Result<T, ParseError> {
    do {
        let value = try JSONDecoder().decode(T.self, from: data)
        return .success(value)
    } catch {
        return .failure(.invalidJSON(error))
    }
}

// ใช้ throws สำหรับ async/await style
public func parseJSONAsync<T: Decodable>(_ data: Data) throws -> T {
    return try JSONDecoder().decode(T.self, from: data)
}
```

---

## 3. API Stability และ Semantic Versioning

### 3.1 Semantic Versioning (SemVer)

Format: **MAJOR.MINOR.PATCH**

```
1.0.0 → Initial release
1.0.1 → Bug fix (PATCH)
1.1.0 → New feature, backward compatible (MINOR)  
2.0.0 → Breaking changes (MAJOR)
```

**ตัวอย่างการตัดสินใจ version:**

```swift
// Version 1.0.0 - Initial API
public func fetchUsers() async throws -> [User] { }

// Version 1.1.0 - Added parameter with default (Non-breaking)
public func fetchUsers(limit: Int = 20) async throws -> [User] { }

// Version 1.2.0 - Added new method (Non-breaking)  
public func fetchUser(id: String) async throws -> User { }

// Version 2.0.0 - Changed return type (BREAKING)
public func fetchUsers(limit: Int = 20) async throws -> PaginatedResult<User> { }
```

### 3.2 @available Attribute

```swift
// Mark deprecated APIs
@available(iOS 13, *)  // Minimum OS requirement
public func modernAPI() async throws -> Result { }

// Deprecation with message
@available(*, deprecated, message: "Use modernAPI() instead")
public func oldAPI(completion: @escaping (Result<Data, Error>) -> Void) {
    Task {
        do {
            let result = try await modernAPI()
            completion(.success(result.data))
        } catch {
            completion(.failure(error))
        }
    }
}

// Renamed API
@available(*, deprecated, renamed: "fetchAsync(from:)")
public func fetch(url: URL, completion: @escaping (Data?) -> Void) { }

@available(iOS 15, *)
public func fetchAsync(from url: URL) async throws -> Data { }

// Unavailable
@available(*, unavailable, message: "This feature is not supported")
public func unsupportedFeature() { }

// Platform-specific
#if os(iOS)
@available(iOS 14, *)
public func iOSOnlyFeature() { }
#endif

// Swift version conditional
#if swift(>=5.9)
public func modernSwiftFeature() { }
#endif
```

### 3.3 Version Availability Checks

```swift
// ตรวจสอบ OS version ก่อนใช้ feature
public func performOperation() {
    if #available(iOS 16, *) {
        // Use iOS 16+ API
        useModernAPI()
    } else {
        // Fallback for older OS
        useLegacyAPI()
    }
}

// ใช้ @backDeployed สำหรับ Swift 5.8+
@backDeployed(before: iOS 16)
public func backDeployedFunction() -> String {
    return "Implemented for older OS"
}
```

---

## 4. Breaking Changes Management

### 4.1 Evolution Strategy

```swift
// Stage 1: Introduce new API alongside old
// Version 1.2.0

// Old API - ยังคงอยู่
public func fetch(url: URL, completion: @escaping (Data?, Error?) -> Void) {
    // Implementation
}

// New API - เพิ่มมาใหม่
public func fetch(from url: URL) async throws -> Data {
    // Implementation
}

// Stage 2: Deprecate old API
// Version 1.3.0

@available(*, deprecated, message: "Use async fetch(from:) instead")
public func fetch(url: URL, completion: @escaping (Data?, Error?) -> Void) {
    Task {
        do {
            let data = try await fetch(from: url)
            completion(data, nil)
        } catch {
            completion(nil, error)
        }
    }
}

// Stage 3: Remove old API
// Version 2.0.0 (Major version bump)
// Old completion-based API removed
```

### 4.2 CHANGELOG Best Practices

```markdown
# Changelog

All notable changes to this project will be documented in this file.
Format based on [Keep a Changelog](https://keepachangelog.com).

## [Unreleased]

## [2.0.0] - 2024-01-15

### Breaking Changes
- `NetworkManager.fetch(url:completion:)` has been removed. 
  Use `fetch(from:)` async version instead.
- Minimum iOS version bumped to iOS 15

### Added
- New `StreamingSupport` for real-time data
- `NetworkManager.batch(_:)` for parallel requests

### Changed
- `Configuration` now requires `apiKey` parameter
- Error types reorganized under `NetworkError` namespace

### Fixed
- Memory leak in `ImageLoader` when cancelling requests

### Migration Guide
```swift
// Before (1.x)
manager.fetch(url: url) { data, error in
    // handle
}

// After (2.0)
let data = try await manager.fetch(from: url)
```

## [1.3.0] - 2023-12-01

### Deprecated
- `fetch(url:completion:)` - Use `fetch(from:)` async instead

### Added
- `fetch(from:)` async/await version
```

---

## 5. Framework Documentation

### 5.1 DocC Documentation

DocC เป็น documentation system ของ Apple สำหรับ Swift:

```swift
/// Network client สำหรับทำ HTTP requests
///
/// `NetworkClient` จัดการ HTTP requests พร้อม automatic retry,
/// caching, และ authentication.
///
/// ## Getting Started
/// สร้าง client instance:
/// ```swift
/// let client = NetworkClient(configuration: .default)
/// ```
///
/// ## Making Requests
/// ใช้ `fetch(_:)` สำหรับ GET requests:
/// ```swift
/// let users: [User] = try await client.fetch("https://api.example.com/users")
/// ```
///
/// ## Topics
/// ### Configuration
/// - ``NetworkConfiguration``
/// - ``NetworkConfiguration/default``
///
/// ### Making Requests  
/// - ``fetch(_:)``
/// - ``post(_:body:)``
/// - ``delete(_:)``
///
/// ### Error Handling
/// - ``NetworkError``
public class NetworkClient {
    
    /// Configuration สำหรับ network client
    public let configuration: NetworkConfiguration
    
    /// สร้าง NetworkClient ใหม่
    /// - Parameter configuration: Configuration ที่จะใช้ (default: `.default`)
    public init(configuration: NetworkConfiguration = .default) {
        self.configuration = configuration
    }
    
    /// ดึงข้อมูลจาก URL และ decode เป็น Decodable type
    ///
    /// - Parameter url: URL ที่จะ fetch
    /// - Returns: Decoded value ของ type T
    /// - Throws: ``NetworkError`` ถ้า request ล้มเหลว
    ///
    /// ## Example
    /// ```swift
    /// struct User: Decodable {
    ///     let id: Int
    ///     let name: String
    /// }
    ///
    /// let user: User = try await client.fetch("https://api.example.com/users/1")
    /// print(user.name)  // "Alice"
    /// ```
    public func fetch<T: Decodable>(_ url: String) async throws -> T {
        // Implementation
        fatalError("Not implemented")
    }
}
```

### 5.2 สร้าง Documentation Catalog

โครงสร้าง DocC:

```
Sources/
└── MyFramework/
    └── Documentation.docc/
        ├── MyFramework.md          (Main page)
        ├── GettingStarted.md       (Tutorial)
        ├── Resources/
        │   └── icon.png
        └── Tutorials/
            └── BuildYourFirst.tutorial
```

**MyFramework.md:**
```markdown
# ``MyFramework``

A powerful networking library for iOS.

## Overview

MyFramework makes it easy to perform HTTP requests, handle errors,
and decode JSON responses.

## Topics

### Essentials
- ``NetworkClient``
- ``NetworkConfiguration``

### Authentication
- ``AuthManager``
- ``AuthToken``

### Error Handling
- ``NetworkError``
```

### 5.3 Hosting Documentation

**GitHub Pages:**
```yaml
# .github/workflows/deploy-docs.yml
name: Deploy Documentation

on:
  push:
    branches: [main]

jobs:
  deploy-docs:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build Documentation
        run: |
          xcodebuild docbuild \
            -scheme MyFramework \
            -destination "platform=iOS Simulator,name=iPhone 14" \
            OTHER_DOCC_FLAGS="--transform-for-static-hosting \
              --hosting-base-path MyFramework \
              --output-path ./docs"
      
      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./docs
```

---

## 6. Unit Testing สำหรับ Framework

### 6.1 Test Structure

```swift
// Tests/MyFrameworkTests/NetworkClientTests.swift
import XCTest
@testable import MyFramework

final class NetworkClientTests: XCTestCase {
    
    var sut: NetworkClient!  // System Under Test
    var mockSession: MockURLSession!
    
    override func setUp() {
        super.setUp()
        mockSession = MockURLSession()
        sut = NetworkClient(session: mockSession)
    }
    
    override func tearDown() {
        sut = nil
        mockSession = nil
        super.tearDown()
    }
    
    // MARK: - Fetch Tests
    
    func testFetch_Success_ReturnsDecodedData() async throws {
        // Arrange
        let expectedUser = User(id: 1, name: "Alice")
        let data = try JSONEncoder().encode(expectedUser)
        mockSession.stub(data: data, statusCode: 200)
        
        // Act
        let result: User = try await sut.fetch("https://example.com/users/1")
        
        // Assert
        XCTAssertEqual(result.id, expectedUser.id)
        XCTAssertEqual(result.name, expectedUser.name)
    }
    
    func testFetch_ServerError_ThrowsNetworkError() async {
        // Arrange
        mockSession.stub(data: nil, statusCode: 500)
        
        // Act & Assert
        do {
            let _: User = try await sut.fetch("https://example.com/users/1")
            XCTFail("Expected error to be thrown")
        } catch NetworkError.serverError(let code, _) {
            XCTAssertEqual(code, 500)
        } catch {
            XCTFail("Unexpected error: \(error)")
        }
    }
    
    func testFetch_InvalidURL_ThrowsInvalidURLError() async {
        // Act & Assert
        do {
            let _: User = try await sut.fetch("not a valid url")
            XCTFail("Expected error")
        } catch NetworkError.invalidURL(let url) {
            XCTAssertEqual(url, "not a valid url")
        } catch {
            XCTFail("Wrong error type: \(error)")
        }
    }
    
    func testFetch_NetworkTimeout_ThrowsTimeoutError() async {
        // Arrange
        mockSession.stubError(URLError(.timedOut))
        
        // Act & Assert
        do {
            let _: User = try await sut.fetch("https://example.com/users/1")
            XCTFail("Expected timeout error")
        } catch NetworkError.timeout {
            // Expected
        } catch {
            XCTFail("Unexpected error: \(error)")
        }
    }
}

// MARK: - Mock URLSession
class MockURLSession: URLSessionProtocol {
    private var stubbedData: Data?
    private var stubbedStatusCode: Int = 200
    private var stubbedError: Error?
    
    func stub(data: Data?, statusCode: Int) {
        self.stubbedData = data
        self.stubbedStatusCode = statusCode
        self.stubbedError = nil
    }
    
    func stubError(_ error: Error) {
        self.stubbedError = error
        self.stubbedData = nil
    }
    
    func data(for request: URLRequest) async throws -> (Data, URLResponse) {
        if let error = stubbedError {
            throw error
        }
        
        let response = HTTPURLResponse(
            url: request.url!,
            statusCode: stubbedStatusCode,
            httpVersion: nil,
            headerFields: nil
        )!
        
        return (stubbedData ?? Data(), response)
    }
}
```

### 6.2 Protocol-Based Testability

```swift
// ใช้ Protocol เพื่อให้ test ง่าย
public protocol URLSessionProtocol {
    func data(for request: URLRequest) async throws -> (Data, URLResponse)
}

// URLSession conform to protocol
extension URLSession: URLSessionProtocol {}

// NetworkClient ใช้ protocol ไม่ใช่ concrete type
public class NetworkClient {
    private let session: URLSessionProtocol
    
    public init(session: URLSessionProtocol = URLSession.shared) {
        self.session = session
    }
}

// Test สามารถ inject mock ได้
let mockSession = MockURLSession()
let testClient = NetworkClient(session: mockSession)
```

### 6.3 Performance Tests

```swift
func testFetchPerformance() throws {
    let expectation = expectation(description: "Fetch completed")
    
    measure {
        Task {
            do {
                let _: [User] = try await sut.fetch("https://example.com/users")
                expectation.fulfill()
            } catch {
                XCTFail(error.localizedDescription)
            }
        }
    }
    
    wait(for: [expectation], timeout: 5.0)
}

// ใช้ XCTest metrics
func testCachingPerformance() throws {
    measure(metrics: [XCTClockMetric(), XCTMemoryMetric()]) {
        for _ in 0..<1000 {
            _ = cache.get(forKey: "testKey")
        }
    }
}
```

---

## 7. การ Distribute ผ่าน Swift Package Manager

### 7.1 Package.swift

```swift
// Package.swift
// swift-tools-version: 5.9

import PackageDescription

let package = Package(
    name: "MyNetworkKit",
    platforms: [
        .iOS(.v15),
        .macOS(.v12),
        .tvOS(.v15),
        .watchOS(.v8)
    ],
    products: [
        // Library products สำหรับ users
        .library(
            name: "MyNetworkKit",
            targets: ["MyNetworkKit"]
        ),
        // Optional: executable สำหรับ CLI tools
        .executable(
            name: "networking-cli",
            targets: ["NetworkingCLI"]
        )
    ],
    dependencies: [
        // External dependencies
        .package(
            url: "https://github.com/apple/swift-log.git",
            from: "1.5.0"
        ),
        .package(
            url: "https://github.com/apple/swift-crypto.git",
            from: "2.5.0"
        )
    ],
    targets: [
        // Main library target
        .target(
            name: "MyNetworkKit",
            dependencies: [
                .product(name: "Logging", package: "swift-log"),
                .product(name: "Crypto", package: "swift-crypto")
            ],
            path: "Sources/MyNetworkKit",
            swiftSettings: [
                .enableUpcomingFeature("BareSlashRegexLiterals"),
                .enableExperimentalFeature("StrictConcurrency")
            ]
        ),
        
        // Test target
        .testTarget(
            name: "MyNetworkKitTests",
            dependencies: ["MyNetworkKit"],
            path: "Tests/MyNetworkKitTests"
        ),
        
        // CLI tool target
        .executableTarget(
            name: "NetworkingCLI",
            dependencies: ["MyNetworkKit"],
            path: "Sources/CLI"
        )
    ]
)
```

### 7.2 Directory Structure

```
MyNetworkKit/
├── Package.swift
├── README.md
├── CHANGELOG.md
├── LICENSE
├── Sources/
│   ├── MyNetworkKit/
│   │   ├── Documentation.docc/
│   │   ├── Core/
│   │   │   ├── NetworkClient.swift
│   │   │   ├── NetworkConfiguration.swift
│   │   │   └── NetworkError.swift
│   │   ├── Auth/
│   │   │   ├── AuthManager.swift
│   │   │   └── AuthToken.swift
│   │   ├── Cache/
│   │   │   └── ResponseCache.swift
│   │   └── Extensions/
│   │       └── URLRequest+Extensions.swift
│   └── CLI/
│       └── main.swift
└── Tests/
    └── MyNetworkKitTests/
        ├── NetworkClientTests.swift
        ├── AuthManagerTests.swift
        └── Mocks/
            └── MockURLSession.swift
```

### 7.3 Publishing ไปยัง GitHub

```bash
# Tag version
git tag -a 1.0.0 -m "Release version 1.0.0"
git push origin 1.0.0

# สร้าง GitHub Release
# ไปที่ GitHub → Releases → Create new release
# เลือก tag 1.0.0
# เพิ่ม release notes จาก CHANGELOG
```

**ผู้ใช้ install ผ่าน SPM:**
```swift
// Package.swift ของ user
dependencies: [
    .package(url: "https://github.com/username/MyNetworkKit", from: "1.0.0")
]
```

---

## 8. การ Distribute ผ่าน CocoaPods

### 8.1 สร้าง Podspec

```ruby
# MyNetworkKit.podspec
Pod::Spec.new do |spec|
  spec.name          = "MyNetworkKit"
  spec.version       = "1.0.0"
  spec.summary       = "A powerful networking library for iOS"
  spec.description   = <<-DESC
    MyNetworkKit provides a simple and powerful API for making 
    HTTP requests in Swift with async/await support.
  DESC
  
  spec.homepage      = "https://github.com/username/MyNetworkKit"
  spec.license       = { :type => "MIT", :file => "LICENSE" }
  spec.author        = { "Your Name" => "email@example.com" }
  
  spec.ios.deployment_target  = "15.0"
  spec.osx.deployment_target  = "12.0"
  spec.swift_versions         = ["5.7", "5.8", "5.9"]
  
  spec.source        = { 
    :git => "https://github.com/username/MyNetworkKit.git", 
    :tag => spec.version 
  }
  
  spec.source_files  = "Sources/MyNetworkKit/**/*.swift"
  spec.exclude_files = "Sources/CLI/**"
  
  # Dependencies
  spec.dependency "SwiftyJSON", "~> 5.0"
  
  # Subspecs สำหรับ optional features
  spec.subspec "Auth" do |auth|
    auth.source_files = "Sources/MyNetworkKit/Auth/**/*.swift"
  end
  
  spec.subspec "Cache" do |cache|
    cache.source_files = "Sources/MyNetworkKit/Cache/**/*.swift"
  end
  
  # Build settings
  spec.pod_target_xcconfig = { 
    "SWIFT_VERSION" => "5.9",
    "OTHER_SWIFT_FLAGS" => "-enable-upcoming-feature BareSlashRegexLiterals"
  }
end
```

### 8.2 Validate และ Push

```bash
# Validate podspec ก่อน publish
pod spec lint MyNetworkKit.podspec --verbose

# Push ไปยัง CocoaPods trunk
pod trunk push MyNetworkKit.podspec

# ตรวจสอบสถานะ
pod search MyNetworkKit
```

---

## 9. การ Distribute ผ่าน Carthage

### 9.1 Cartfile

```ruby
# Cartfile ของ user
github "username/MyNetworkKit" ~> 1.0
```

### 9.2 ทำให้ Carthage Compatible

```yaml
# .github/workflows/carthage-build.yml
name: Build for Carthage

on:
  release:
    types: [published]

jobs:
  build:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build XCFramework
        run: |
          # Build for all platforms
          xcodebuild archive \
            -scheme MyNetworkKit \
            -destination "generic/platform=iOS" \
            -archivePath archives/ios \
            SKIP_INSTALL=NO BUILD_LIBRARY_FOR_DISTRIBUTION=YES
          
          xcodebuild archive \
            -scheme MyNetworkKit \
            -destination "generic/platform=iOS Simulator" \
            -archivePath archives/ios-sim \
            SKIP_INSTALL=NO BUILD_LIBRARY_FOR_DISTRIBUTION=YES
          
          # Create XCFramework
          xcodebuild -create-xcframework \
            -framework archives/ios.xcarchive/Products/Library/Frameworks/MyNetworkKit.framework \
            -framework archives/ios-sim.xcarchive/Products/Library/Frameworks/MyNetworkKit.framework \
            -output MyNetworkKit.xcframework
          
          # Zip and attach to release
          zip -r MyNetworkKit.xcframework.zip MyNetworkKit.xcframework
      
      - name: Upload to Release
        uses: softprops/action-gh-release@v1
        with:
          files: MyNetworkKit.xcframework.zip
```

---

## 10. Binary Framework (XCFramework)

### 10.1 สร้าง XCFramework ด้วยตนเอง

```bash
#!/bin/bash
# build-xcframework.sh

FRAMEWORK_NAME="MyNetworkKit"
SCHEME="MyNetworkKit"

# ลบ build เก่า
rm -rf archives
rm -rf "${FRAMEWORK_NAME}.xcframework"

# Build iOS Device
xcodebuild archive \
  -scheme ${SCHEME} \
  -destination "generic/platform=iOS" \
  -archivePath "archives/ios" \
  SKIP_INSTALL=NO \
  BUILD_LIBRARY_FOR_DISTRIBUTION=YES \
  SWIFT_SERIALIZE_DEBUGGING_OPTIONS=NO

# Build iOS Simulator
xcodebuild archive \
  -scheme ${SCHEME} \
  -destination "generic/platform=iOS Simulator" \
  -archivePath "archives/ios-simulator" \
  SKIP_INSTALL=NO \
  BUILD_LIBRARY_FOR_DISTRIBUTION=YES

# Build macOS
xcodebuild archive \
  -scheme ${SCHEME} \
  -destination "generic/platform=macOS" \
  -archivePath "archives/macos" \
  SKIP_INSTALL=NO \
  BUILD_LIBRARY_FOR_DISTRIBUTION=YES

# Build tvOS
xcodebuild archive \
  -scheme ${SCHEME} \
  -destination "generic/platform=tvOS" \
  -archivePath "archives/tvos" \
  SKIP_INSTALL=NO \
  BUILD_LIBRARY_FOR_DISTRIBUTION=YES

# Create XCFramework
xcodebuild -create-xcframework \
  -framework "archives/ios.xcarchive/Products/Library/Frameworks/${FRAMEWORK_NAME}.framework" \
  -debug-symbols "archives/ios.xcarchive/dSYMs/${FRAMEWORK_NAME}.framework.dSYM" \
  -framework "archives/ios-simulator.xcarchive/Products/Library/Frameworks/${FRAMEWORK_NAME}.framework" \
  -framework "archives/macos.xcarchive/Products/Library/Frameworks/${FRAMEWORK_NAME}.framework" \
  -framework "archives/tvos.xcarchive/Products/Library/Frameworks/${FRAMEWORK_NAME}.framework" \
  -output "${FRAMEWORK_NAME}.xcframework"

echo "✅ XCFramework created: ${FRAMEWORK_NAME}.xcframework"
```

### 10.2 Module Stability

```swift
// Enable Module Stability ใน Build Settings:
// BUILD_LIBRARY_FOR_DISTRIBUTION = YES

// สร้าง .swiftinterface file อัตโนมัติ
// ทำให้ framework ทำงานได้กับ Swift versions ต่างๆ

// สำคัญ: ต้องใช้ @frozen สำหรับ enums และ structs
// ที่ต้องการ ABI stability

@frozen
public enum Status {
    case active
    case inactive
    case pending
}

// @frozen ทำให้ compiler optimize ได้ดีกว่า
// แต่ไม่สามารถเพิ่ม cases ได้โดยไม่ break ABI

// Non-frozen enum (default) - สามารถเพิ่ม cases ได้
public enum NetworkState {
    case connected
    case disconnected
    case connecting
    // Future cases can be added
}

// User ต้อง handle @unknown default:
switch networkState {
case .connected: break
case .disconnected: break
case .connecting: break
@unknown default: break  // Required for non-frozen enums
}
```

---

## 11. Dynamic vs Static Frameworks

### 11.1 ความแตกต่าง

**Static Framework (.a + headers):**
- Code ถูก compile รวมเข้ากับ app binary
- App launch time เร็วกว่า (ไม่ต้อง load)
- App binary size ใหญ่ขึ้น
- หลาย targets สามารถ link ได้ แต่จะ duplicate code

**Dynamic Framework (.framework):**
- Load ตอน runtime
- App launch time ช้ากว่าเล็กน้อย
- สามารถ share กันระหว่าง app กับ extensions
- Apple ต้องการ dynamic framework สำหรับ extensions

```swift
// Package.swift - กำหนด library type
.library(
    name: "MyNetworkKit",
    type: .static,    // Static library
    targets: ["MyNetworkKit"]
)

// หรือ
.library(
    name: "MyNetworkKit",
    type: .dynamic,   // Dynamic library
    targets: ["MyNetworkKit"]
)

// ถ้าไม่ระบุ type = automatic (SPM เลือกให้)
```

### 11.2 เมื่อไรใช้อะไร

```
Static Framework ดีกว่าเมื่อ:
✅ Framework ขนาดเล็ก
✅ ต้องการ app launch performance
✅ ไม่มี app extensions

Dynamic Framework ดีกว่าเมื่อ:
✅ Framework ขนาดใหญ่
✅ มีหลาย targets/extensions ที่ share framework
✅ ต้องการ hot-swap/update framework โดยไม่ rebuild app
```

---

## 12. Open Source Community Management

### 12.1 CONTRIBUTING.md

```markdown
# Contributing to MyNetworkKit

## Ways to Contribute
- 🐛 Report bugs
- 💡 Suggest features
- 📖 Improve documentation
- 🔧 Submit code changes

## Development Setup
1. Fork and clone the repository
2. Open `Package.swift` in Xcode
3. Create a branch: `git checkout -b feature/my-feature`

## Code Standards
- Follow Swift API Design Guidelines
- Write tests for new code
- Update documentation
- Keep commits atomic and well-described

## Pull Request Process
1. Update CHANGELOG.md
2. Ensure tests pass: `swift test`
3. Check SwiftLint: `swiftlint`
4. Open PR with clear description

## Commit Messages
Format: `type(scope): subject`

Types: feat, fix, docs, style, refactor, test, chore

Examples:
- `feat(auth): add OAuth 2.0 support`
- `fix(cache): fix memory leak in ResponseCache`
- `docs: update getting started guide`

## Code of Conduct
Be respectful, inclusive, and professional.
```

### 12.2 Issue Templates

```yaml
# .github/ISSUE_TEMPLATE/bug_report.yml
name: Bug Report
description: File a bug report
title: "[Bug]: "
labels: ["bug", "needs-triage"]
body:
  - type: markdown
    attributes:
      value: "Thank you for reporting a bug!"
  
  - type: textarea
    id: description
    attributes:
      label: Bug Description
      description: A clear description of the bug
      placeholder: Describe the bug...
    validations:
      required: true
  
  - type: textarea
    id: reproduction
    attributes:
      label: Reproduction Steps
      description: Steps to reproduce the behavior
      value: |
        1. Create a NetworkClient
        2. Call fetch(...)
        3. See error
    validations:
      required: true
  
  - type: textarea
    id: expected
    attributes:
      label: Expected Behavior
      placeholder: What should happen?
    validations:
      required: true
  
  - type: textarea
    id: code-sample
    attributes:
      label: Code Sample
      description: Minimal code to reproduce
      render: swift
  
  - type: dropdown
    id: ios-version
    attributes:
      label: iOS Version
      options:
        - iOS 17
        - iOS 16
        - iOS 15
    validations:
      required: true
  
  - type: input
    id: framework-version
    attributes:
      label: Framework Version
      placeholder: "1.2.3"
    validations:
      required: true
```

```yaml
# .github/ISSUE_TEMPLATE/feature_request.yml
name: Feature Request
description: Suggest a new feature
title: "[Feature]: "
labels: ["enhancement"]
body:
  - type: textarea
    id: problem
    attributes:
      label: Problem Statement
      description: What problem does this solve?
    validations:
      required: true
  
  - type: textarea
    id: solution
    attributes:
      label: Proposed Solution
      description: How would you like this to work?
    validations:
      required: true
  
  - type: textarea
    id: api-sketch
    attributes:
      label: API Sketch (optional)
      render: swift
      description: What would the API look like?
```

---

## 13. ตัวอย่าง Framework: Analytics SDK

### 13.1 Analytics Framework Design

```swift
// Sources/AnalyticsKit/AnalyticsKit.swift

/// Analytics SDK สำหรับติดตาม user events
public final class Analytics {
    
    // Singleton
    public static let shared = Analytics()
    
    private var providers: [AnalyticsProvider] = []
    private var queue: DispatchQueue
    private var eventQueue: [Event] = []
    private var isBatchingEnabled: Bool
    private let batchSize: Int
    
    // MARK: - Configuration
    
    public struct Configuration {
        public var providers: [AnalyticsProvider]
        public var enableBatching: Bool
        public var batchSize: Int
        public var batchInterval: TimeInterval
        public var enableLogging: Bool
        
        public static var `default`: Configuration {
            Configuration(
                providers: [],
                enableBatching: true,
                batchSize: 10,
                batchInterval: 30,
                enableLogging: false
            )
        }
        
        public init(
            providers: [AnalyticsProvider],
            enableBatching: Bool = true,
            batchSize: Int = 10,
            batchInterval: TimeInterval = 30,
            enableLogging: Bool = false
        ) {
            self.providers = providers
            self.enableBatching = enableBatching
            self.batchSize = batchSize
            self.batchInterval = batchInterval
            self.enableLogging = enableLogging
        }
    }
    
    private init() {
        self.queue = DispatchQueue(label: "com.analyticskit.queue", qos: .background)
        self.isBatchingEnabled = true
        self.batchSize = 10
    }
    
    /// Configure Analytics SDK
    /// - Parameter configuration: SDK configuration
    public func configure(with configuration: Configuration) {
        self.providers = configuration.providers
        self.isBatchingEnabled = configuration.enableBatching
    }
    
    // MARK: - Event Tracking
    
    /// Track a custom event
    /// - Parameters:
    ///   - name: Event name
    ///   - properties: Optional event properties
    public func track(_ name: String, properties: [String: Any]? = nil) {
        let event = Event(
            name: name,
            properties: properties ?? [:],
            timestamp: Date(),
            userId: currentUserId,
            sessionId: currentSessionId
        )
        
        queue.async { [weak self] in
            self?.processEvent(event)
        }
    }
    
    /// Track screen view
    /// - Parameters:
    ///   - name: Screen name
    ///   - properties: Optional properties
    public func screen(_ name: String, properties: [String: Any]? = nil) {
        var props = properties ?? [:]
        props["screen_name"] = name
        track("Screen View", properties: props)
    }
    
    /// Identify user
    /// - Parameters:
    ///   - userId: User identifier
    ///   - traits: User traits/properties
    public func identify(userId: String, traits: [String: Any]? = nil) {
        self.currentUserId = userId
        
        providers.forEach { provider in
            provider.identify(userId: userId, traits: traits ?? [:])
        }
    }
    
    // MARK: - Private
    
    private var currentUserId: String?
    private var currentSessionId = UUID().uuidString
    
    private func processEvent(_ event: Event) {
        if isBatchingEnabled {
            eventQueue.append(event)
            if eventQueue.count >= batchSize {
                flushEvents()
            }
        } else {
            sendEvent(event)
        }
    }
    
    private func flushEvents() {
        let batch = eventQueue
        eventQueue.removeAll()
        
        providers.forEach { provider in
            batch.forEach { event in
                provider.track(event: event)
            }
        }
    }
    
    private func sendEvent(_ event: Event) {
        providers.forEach { provider in
            provider.track(event: event)
        }
    }
}

// MARK: - Models

public struct Event {
    public let id: String
    public let name: String
    public let properties: [String: Any]
    public let timestamp: Date
    public let userId: String?
    public let sessionId: String
    
    init(name: String, properties: [String: Any], timestamp: Date, 
         userId: String?, sessionId: String) {
        self.id = UUID().uuidString
        self.name = name
        self.properties = properties
        self.timestamp = timestamp
        self.userId = userId
        self.sessionId = sessionId
    }
}

// MARK: - Provider Protocol

public protocol AnalyticsProvider {
    var name: String { get }
    func track(event: Event)
    func identify(userId: String, traits: [String: Any])
    func flush()
}

// MARK: - Built-in Providers

public class ConsoleProvider: AnalyticsProvider {
    public let name = "Console"
    
    public init() {}
    
    public func track(event: Event) {
        print("[Analytics] Event: \(event.name)")
        if !event.properties.isEmpty {
            print("  Properties: \(event.properties)")
        }
    }
    
    public func identify(userId: String, traits: [String: Any]) {
        print("[Analytics] Identify: \(userId)")
    }
    
    public func flush() {}
}
```

---

## 14. ตัวอย่าง Framework: Network Layer

### 14.1 Comprehensive Network Framework

```swift
// Sources/NetworkKit/Core/NetworkClient.swift

import Foundation

/// HTTP client with async/await support
public actor NetworkClient {
    
    private let session: URLSession
    private let baseURL: URL?
    private let interceptors: [RequestInterceptor]
    private let decoder: JSONDecoder
    private let encoder: JSONEncoder
    
    // MARK: - Initialization
    
    public init(
        baseURL: URL? = nil,
        session: URLSession = .shared,
        interceptors: [RequestInterceptor] = [],
        decoder: JSONDecoder = .init(),
        encoder: JSONEncoder = .init()
    ) {
        self.baseURL = baseURL
        self.session = session
        self.interceptors = interceptors
        self.decoder = decoder
        self.encoder = encoder
    }
    
    // MARK: - Request Building
    
    private func buildRequest(for endpoint: Endpoint) throws -> URLRequest {
        var urlComponents: URLComponents
        
        if let base = baseURL {
            let fullPath = base.appendingPathComponent(endpoint.path)
            guard let components = URLComponents(url: fullPath, resolvingAgainstBaseURL: true) else {
                throw NetworkError.invalidURL(endpoint.path)
            }
            urlComponents = components
        } else {
            guard let components = URLComponents(string: endpoint.path) else {
                throw NetworkError.invalidURL(endpoint.path)
            }
            urlComponents = components
        }
        
        // Add query parameters
        if let params = endpoint.queryParameters, !params.isEmpty {
            urlComponents.queryItems = params.map { URLQueryItem(name: $0.key, value: "\($0.value)") }
        }
        
        guard let url = urlComponents.url else {
            throw NetworkError.invalidURL(endpoint.path)
        }
        
        var request = URLRequest(url: url)
        request.httpMethod = endpoint.method.rawValue
        request.timeoutInterval = endpoint.timeout
        
        // Set headers
        endpoint.headers?.forEach { request.setValue($0.value, forHTTPHeaderField: $0.key) }
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        request.setValue("application/json", forHTTPHeaderField: "Accept")
        
        // Set body
        if let body = endpoint.body {
            request.httpBody = try encoder.encode(body)
        }
        
        return request
    }
    
    // MARK: - Performing Requests
    
    public func perform<T: Decodable>(_ endpoint: Endpoint) async throws -> T {
        var request = try buildRequest(for: endpoint)
        
        // Apply interceptors
        for interceptor in interceptors {
            request = try await interceptor.adapt(request)
        }
        
        do {
            let (data, response) = try await session.data(for: request)
            
            guard let httpResponse = response as? HTTPURLResponse else {
                throw NetworkError.invalidResponse
            }
            
            // Handle HTTP errors
            switch httpResponse.statusCode {
            case 200...299:
                break
            case 401:
                throw NetworkError.unauthorized
            case 429:
                let retryAfter = httpResponse.value(forHTTPHeaderField: "Retry-After")
                    .flatMap(Double.init)
                throw NetworkError.rateLimited(retryAfter: retryAfter)
            case 500...599:
                throw NetworkError.serverError(
                    statusCode: httpResponse.statusCode,
                    message: String(data: data, encoding: .utf8)
                )
            default:
                throw NetworkError.serverError(
                    statusCode: httpResponse.statusCode,
                    message: nil
                )
            }
            
            // Decode response
            do {
                return try decoder.decode(T.self, from: data)
            } catch {
                throw NetworkError.decodingFailed(underlyingError: error)
            }
            
        } catch let error as NetworkError {
            throw error
        } catch let error as URLError {
            switch error.code {
            case .timedOut:
                throw NetworkError.timeout(after: request.timeoutInterval)
            case .notConnectedToInternet, .networkConnectionLost:
                throw NetworkError.noInternetConnection
            default:
                throw NetworkError.unknown(error)
            }
        }
    }
}

// MARK: - Endpoint Definition

public struct Endpoint {
    public let path: String
    public let method: HTTPMethod
    public let queryParameters: [String: Any]?
    public let headers: [String: String]?
    public let body: (any Encodable)?
    public let timeout: TimeInterval
    
    public init(
        path: String,
        method: HTTPMethod = .get,
        queryParameters: [String: Any]? = nil,
        headers: [String: String]? = nil,
        body: (any Encodable)? = nil,
        timeout: TimeInterval = 30
    ) {
        self.path = path
        self.method = method
        self.queryParameters = queryParameters
        self.headers = headers
        self.body = body
        self.timeout = timeout
    }
}

public enum HTTPMethod: String {
    case get = "GET"
    case post = "POST"
    case put = "PUT"
    case patch = "PATCH"
    case delete = "DELETE"
}

// MARK: - Request Interceptor

public protocol RequestInterceptor {
    func adapt(_ request: URLRequest) async throws -> URLRequest
    func retry(_ request: URLRequest, after error: Error) async throws -> Bool
}

// Auth Interceptor Example
public class AuthInterceptor: RequestInterceptor {
    private let tokenProvider: () async throws -> String
    
    public init(tokenProvider: @escaping () async throws -> String) {
        self.tokenProvider = tokenProvider
    }
    
    public func adapt(_ request: URLRequest) async throws -> URLRequest {
        var request = request
        let token = try await tokenProvider()
        request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        return request
    }
    
    public func retry(_ request: URLRequest, after error: Error) async throws -> Bool {
        // Retry on auth error by refreshing token
        if case NetworkError.unauthorized = error {
            return true
        }
        return false
    }
}
```

---

## 15. ตัวอย่าง Framework: UI Component Library

### 15.1 SwiftUI Component Library

```swift
// Sources/DesignKit/Components/Button/PrimaryButton.swift

import SwiftUI

/// Primary action button with consistent styling
public struct PrimaryButton: View {
    
    private let title: String
    private let icon: Image?
    private let isLoading: Bool
    private let isDisabled: Bool
    private let size: ButtonSize
    private let action: () -> Void
    
    // MARK: - Initialization
    
    /// สร้าง PrimaryButton
    /// - Parameters:
    ///   - title: Button label
    ///   - icon: Optional leading icon
    ///   - isLoading: Show loading indicator
    ///   - isDisabled: Disable the button
    ///   - size: Button size (.small, .medium, .large)
    ///   - action: Tap action handler
    public init(
        _ title: String,
        icon: Image? = nil,
        isLoading: Bool = false,
        isDisabled: Bool = false,
        size: ButtonSize = .medium,
        action: @escaping () -> Void
    ) {
        self.title = title
        self.icon = icon
        self.isLoading = isLoading
        self.isDisabled = isDisabled
        self.size = size
        self.action = action
    }
    
    public var body: some View {
        Button(action: action) {
            HStack(spacing: 8) {
                if isLoading {
                    ProgressView()
                        .progressViewStyle(CircularProgressViewStyle(tint: .white))
                        .frame(width: 16, height: 16)
                } else if let icon = icon {
                    icon
                        .resizable()
                        .frame(width: size.iconSize, height: size.iconSize)
                }
                
                Text(title)
                    .font(size.font)
                    .fontWeight(.semibold)
            }
            .frame(maxWidth: .infinity)
            .frame(height: size.height)
            .padding(.horizontal, size.horizontalPadding)
        }
        .background(isDisabled ? Color.gray : Color.blue)
        .foregroundColor(.white)
        .cornerRadius(size.cornerRadius)
        .disabled(isDisabled || isLoading)
        .animation(.easeInOut(duration: 0.2), value: isLoading)
    }
    
    // MARK: - Button Size
    
    public enum ButtonSize {
        case small, medium, large
        
        var height: CGFloat {
            switch self {
            case .small: return 36
            case .medium: return 48
            case .large: return 56
            }
        }
        
        var font: Font {
            switch self {
            case .small: return .subheadline
            case .medium: return .body
            case .large: return .title3
            }
        }
        
        var iconSize: CGFloat {
            switch self {
            case .small: return 14
            case .medium: return 18
            case .large: return 22
            }
        }
        
        var horizontalPadding: CGFloat {
            switch self {
            case .small: return 16
            case .medium: return 24
            case .large: return 32
            }
        }
        
        var cornerRadius: CGFloat {
            switch self {
            case .small: return 8
            case .medium: return 12
            case .large: return 14
            }
        }
    }
}

// MARK: - Preview Provider
struct PrimaryButton_Previews: PreviewProvider {
    static var previews: some View {
        VStack(spacing: 16) {
            PrimaryButton("Continue") { }
            PrimaryButton("Loading", isLoading: true) { }
            PrimaryButton("Disabled", isDisabled: true) { }
            PrimaryButton("With Icon", icon: Image(systemName: "arrow.right")) { }
            PrimaryButton("Small", size: .small) { }
            PrimaryButton("Large", size: .large) { }
        }
        .padding()
    }
}
```

---

## 16. การสร้าง Complete Utility Framework

### 16.1 SwiftUtils Framework

```swift
// Package.swift
// swift-tools-version: 5.9

import PackageDescription

let package = Package(
    name: "SwiftUtils",
    platforms: [.iOS(.v15), .macOS(.v12)],
    products: [
        .library(name: "SwiftUtils", targets: ["SwiftUtils"])
    ],
    targets: [
        .target(
            name: "SwiftUtils",
            path: "Sources/SwiftUtils"
        ),
        .testTarget(
            name: "SwiftUtilsTests",
            dependencies: ["SwiftUtils"]
        )
    ]
)
```

```swift
// Sources/SwiftUtils/Extensions/Collection+Extensions.swift

public extension Collection {
    
    /// Safe subscript - returns nil instead of crashing
    subscript(safe index: Index) -> Element? {
        return indices.contains(index) ? self[index] : nil
    }
    
    /// Returns chunks of specified size
    func chunked(into size: Int) -> [[Element]] {
        return stride(from: 0, to: count, by: size).map {
            Array(self[index(startIndex, offsetBy: $0)..<index(startIndex, offsetBy: Swift.min($0 + size, count))])
        }
    }
    
    /// Returns true if collection has elements
    var isNotEmpty: Bool { !isEmpty }
}

public extension Array {
    
    /// Remove duplicates while preserving order (requires Equatable)
    func removingDuplicates<T: Hashable>(by keyPath: KeyPath<Element, T>) -> [Element] {
        var seen = Set<T>()
        return filter { seen.insert($0[keyPath: keyPath]).inserted }
    }
}

// Sources/SwiftUtils/Extensions/String+Extensions.swift

public extension String {
    
    /// Trim whitespace and newlines
    var trimmed: String {
        trimmingCharacters(in: .whitespacesAndNewlines)
    }
    
    /// Check if string is a valid email
    var isValidEmail: Bool {
        let emailRegex = #"^[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$"#
        return range(of: emailRegex, options: .regularExpression) != nil
    }
    
    /// Check if string contains only numbers
    var isNumeric: Bool {
        !isEmpty && allSatisfy { $0.isNumber }
    }
    
    /// Convert to URL if valid
    var asURL: URL? {
        URL(string: self)
    }
    
    /// Localized string helper
    var localized: String {
        NSLocalizedString(self, comment: "")
    }
    
    /// Truncate to max length with ellipsis
    func truncated(to maxLength: Int, trailing: String = "...") -> String {
        guard count > maxLength else { return self }
        return String(prefix(maxLength - trailing.count)) + trailing
    }
}

// Sources/SwiftUtils/UserDefaults/UserDefaultsWrapper.swift

/// Property wrapper สำหรับ UserDefaults
@propertyWrapper
public struct UserDefault<T> {
    
    public let key: String
    public let defaultValue: T
    private let store: UserDefaults
    
    public init(
        _ key: String,
        defaultValue: T,
        store: UserDefaults = .standard
    ) {
        self.key = key
        self.defaultValue = defaultValue
        self.store = store
    }
    
    public var wrappedValue: T {
        get {
            return store.object(forKey: key) as? T ?? defaultValue
        }
        set {
            if let optional = newValue as? AnyOptional, optional.isNil {
                store.removeObject(forKey: key)
            } else {
                store.set(newValue, forKey: key)
            }
        }
    }
    
    public var projectedValue: UserDefault<T> { self }
}

private protocol AnyOptional {
    var isNil: Bool { get }
}

extension Optional: AnyOptional {
    var isNil: Bool { self == nil }
}

// ตัวอย่างการใช้งาน
struct AppSettings {
    @UserDefault("hasOnboarded", defaultValue: false)
    static var hasOnboarded: Bool
    
    @UserDefault("userName", defaultValue: "")
    static var userName: String
    
    @UserDefault("themeMode", defaultValue: 0)
    static var themeMode: Int
}

// Sources/SwiftUtils/Networking/NetworkReachability.swift

import Network

/// Monitor network connectivity
public final class NetworkReachability: ObservableObject {
    
    public static let shared = NetworkReachability()
    
    @Published public private(set) var isConnected = false
    @Published public private(set) var connectionType: ConnectionType = .unknown
    
    private let monitor: NWPathMonitor
    private let queue = DispatchQueue(label: "NetworkReachability")
    
    public enum ConnectionType {
        case wifi, cellular, wiredEthernet, unknown
    }
    
    private init() {
        monitor = NWPathMonitor()
        startMonitoring()
    }
    
    deinit {
        stopMonitoring()
    }
    
    private func startMonitoring() {
        monitor.pathUpdateHandler = { [weak self] path in
            DispatchQueue.main.async {
                self?.isConnected = path.status == .satisfied
                self?.connectionType = self?.getConnectionType(path) ?? .unknown
            }
        }
        monitor.start(queue: queue)
    }
    
    private func stopMonitoring() {
        monitor.cancel()
    }
    
    private func getConnectionType(_ path: NWPath) -> ConnectionType {
        if path.usesInterfaceType(.wifi) { return .wifi }
        if path.usesInterfaceType(.cellular) { return .cellular }
        if path.usesInterfaceType(.wiredEthernet) { return .wiredEthernet }
        return .unknown
    }
}
```

---

## 17. แบบฝึกหัดพร้อม Solutions

### แบบฝึกหัดที่ 1: สร้าง Logger Framework

**โจทย์**: สร้าง framework ชื่อ `SwiftLogger` ที่มีความสามารถ:
- Log levels: debug, info, warning, error
- Multiple destinations (console, file, network)
- Structured logging
- Thread-safe

```swift
// Solution

import Foundation

// MARK: - Log Level
public enum LogLevel: Int, Comparable, CustomStringConvertible {
    case debug = 0
    case info = 1
    case warning = 2
    case error = 3
    
    public static func < (lhs: LogLevel, rhs: LogLevel) -> Bool {
        lhs.rawValue < rhs.rawValue
    }
    
    public var description: String {
        switch self {
        case .debug: return "DEBUG"
        case .info: return "INFO"
        case .warning: return "WARNING"
        case .error: return "ERROR"
        }
    }
    
    var emoji: String {
        switch self {
        case .debug: return "🔍"
        case .info: return "ℹ️"
        case .warning: return "⚠️"
        case .error: return "❌"
        }
    }
}

// MARK: - Log Entry
public struct LogEntry {
    public let level: LogLevel
    public let message: String
    public let metadata: [String: Any]?
    public let timestamp: Date
    public let file: String
    public let function: String
    public let line: Int
    
    public var formatted: String {
        let dateStr = DateFormatter.logFormatter.string(from: timestamp)
        let location = "\(URL(fileURLWithPath: file).lastPathComponent):\(line)"
        return "\(level.emoji) [\(dateStr)] [\(level)] \(location) \(function) - \(message)"
    }
}

extension DateFormatter {
    static let logFormatter: DateFormatter = {
        let formatter = DateFormatter()
        formatter.dateFormat = "yyyy-MM-dd HH:mm:ss.SSS"
        return formatter
    }()
}

// MARK: - Log Destination Protocol
public protocol LogDestination {
    var minimumLevel: LogLevel { get }
    func write(_ entry: LogEntry)
}

// MARK: - Console Destination
public class ConsoleDestination: LogDestination {
    public let minimumLevel: LogLevel
    
    public init(minimumLevel: LogLevel = .debug) {
        self.minimumLevel = minimumLevel
    }
    
    public func write(_ entry: LogEntry) {
        print(entry.formatted)
    }
}

// MARK: - File Destination
public class FileDestination: LogDestination {
    public let minimumLevel: LogLevel
    private let fileURL: URL
    private let queue: DispatchQueue
    private var fileHandle: FileHandle?
    
    public init(fileURL: URL, minimumLevel: LogLevel = .info) throws {
        self.minimumLevel = minimumLevel
        self.fileURL = fileURL
        self.queue = DispatchQueue(label: "SwiftLogger.File", qos: .background)
        
        // Create file if doesn't exist
        if !FileManager.default.fileExists(atPath: fileURL.path) {
            FileManager.default.createFile(atPath: fileURL.path, contents: nil)
        }
        
        self.fileHandle = try FileHandle(forWritingTo: fileURL)
        self.fileHandle?.seekToEndOfFile()
    }
    
    public func write(_ entry: LogEntry) {
        queue.async { [weak self] in
            guard let self = self,
                  let data = (entry.formatted + "\n").data(using: .utf8) else { return }
            self.fileHandle?.write(data)
        }
    }
    
    deinit {
        fileHandle?.closeFile()
    }
}

// MARK: - Logger
public final class Logger {
    
    public static let shared = Logger()
    
    private var destinations: [LogDestination] = []
    private let queue = DispatchQueue(label: "SwiftLogger", attributes: .concurrent)
    
    private init() {
        // Default console destination
        destinations.append(ConsoleDestination())
    }
    
    public func addDestination(_ destination: LogDestination) {
        queue.async(flags: .barrier) { [weak self] in
            self?.destinations.append(destination)
        }
    }
    
    public func removeAllDestinations() {
        queue.async(flags: .barrier) { [weak self] in
            self?.destinations.removeAll()
        }
    }
    
    // MARK: - Logging Methods
    
    public func debug(
        _ message: String,
        metadata: [String: Any]? = nil,
        file: String = #file,
        function: String = #function,
        line: Int = #line
    ) {
        log(.debug, message: message, metadata: metadata, file: file, function: function, line: line)
    }
    
    public func info(
        _ message: String,
        metadata: [String: Any]? = nil,
        file: String = #file,
        function: String = #function,
        line: Int = #line
    ) {
        log(.info, message: message, metadata: metadata, file: file, function: function, line: line)
    }
    
    public func warning(
        _ message: String,
        metadata: [String: Any]? = nil,
        file: String = #file,
        function: String = #function,
        line: Int = #line
    ) {
        log(.warning, message: message, metadata: metadata, file: file, function: function, line: line)
    }
    
    public func error(
        _ message: String,
        metadata: [String: Any]? = nil,
        file: String = #file,
        function: String = #function,
        line: Int = #line
    ) {
        log(.error, message: message, metadata: metadata, file: file, function: function, line: line)
    }
    
    private func log(
        _ level: LogLevel,
        message: String,
        metadata: [String: Any]?,
        file: String,
        function: String,
        line: Int
    ) {
        let entry = LogEntry(
            level: level,
            message: message,
            metadata: metadata,
            timestamp: Date(),
            file: file,
            function: function,
            line: line
        )
        
        queue.async { [weak self] in
            self?.destinations
                .filter { $0.minimumLevel <= level }
                .forEach { $0.write(entry) }
        }
    }
}

// MARK: - Usage Example
/*
// Configure
let logFile = FileManager.default
    .urls(for: .documentDirectory, in: .userDomainMask)[0]
    .appendingPathComponent("app.log")

if let fileDestination = try? FileDestination(fileURL: logFile, minimumLevel: .info) {
    Logger.shared.addDestination(fileDestination)
}

// Use
Logger.shared.debug("Starting app")
Logger.shared.info("User logged in", metadata: ["userId": "123"])
Logger.shared.warning("Cache miss", metadata: ["key": "users"])
Logger.shared.error("Network failed", metadata: ["url": "https://api.example.com"])
*/
```

---

## 18. Versioning Strategy

### 18.1 แผนการ Version ระยะยาว

```
v0.x.x - Development/Pre-release
  v0.1.0 - First prototype
  v0.5.0 - Beta with core features
  v0.9.0 - Release candidate
  
v1.0.0 - Stable release
  v1.0.x - Bug fixes only
  v1.1.0 - New features (backward compatible)
  v1.2.0 - More features
  
v2.0.0 - Major version (breaking changes allowed)
  - Removed deprecated APIs
  - New architecture
  - Updated minimum OS version
```

### 18.2 Git Tagging Strategy

```bash
# Release workflow
git checkout main
git pull origin main

# Update version in Package.swift and CHANGELOG
# ...

# Tag release
git tag -a "1.2.0" -m "Release 1.2.0

New Features:
- Added batch request support
- Added response caching

Bug Fixes:
- Fixed memory leak in ImageLoader
- Fixed crash on retry"

# Push tag
git push origin "1.2.0"

# Create GitHub Release (via gh CLI)
gh release create "1.2.0" \
  --title "Version 1.2.0" \
  --notes-file CHANGELOG.md \
  MyNetworkKit.xcframework.zip
```

---

## สรุป

การสร้าง framework ที่ดีต้องอาศัยความเข้าใจในหลายด้าน:

**Key Takeaways:**

1. **API Design**: ออกแบบ API ให้ simple สำหรับ common cases และ flexible สำหรับ advanced use
2. **Stability**: ใช้ semantic versioning และ @available อย่างถูกต้อง
3. **Documentation**: DocC documentation ทำให้ users ใช้งาน framework ได้ง่าย
4. **Testing**: Unit tests ที่ครอบคลุมทำให้มั่นใจใน quality
5. **Distribution**: เลือก distribution method ที่เหมาะกับ audience
6. **Community**: README, CONTRIBUTING, และ issue templates ดีๆ ช่วยสร้าง community
7. **Versioning**: มีกลยุทธ์การ version ที่ชัดเจนและสื่อสารกับ users

Framework ที่ดีไม่ได้วัดแค่ functionality แต่วัดที่ developer experience ด้วย - ถ้า users เปิด documentation แล้วสามารถ integrate ได้ภายใน 5 นาที นั่นคือ framework ที่ประสบความสำเร็จ!

---

*เนื้อหาในบทนี้ครอบคลุมทุกขั้นตอนของการสร้าง Swift Framework จาก API design ไปจนถึง open source community management*
