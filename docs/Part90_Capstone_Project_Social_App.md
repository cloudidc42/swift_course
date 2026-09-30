# Part 90: Capstone Project — SwiftConnect Social App

## บทนำ

บทนี้เป็น Capstone Project ที่รวมทุกความรู้จาก Course นี้เข้าด้วยกัน เราจะสร้าง **SwiftConnect** — แอปพลิเคชัน Professional Networking ที่ทำงานได้จริงในระดับ production โดยใช้ Architecture, Patterns, และ Frameworks ที่ดีที่สุด

---

## 1. Project Overview: SwiftConnect

### 1.1 Feature Set

**SwiftConnect** เป็น professional networking app คล้าย LinkedIn โดยมี features หลัก:

- **Profiles**: สร้างและแก้ไข professional profile พร้อมรูปภาพ
- **Posts**: สร้าง, อ่าน, Like, และ Comment บน posts
- **Connections**: ส่ง/รับ connection requests, ดู network
- **Messaging**: Real-time direct messaging
- **Notifications**: Push notifications และ in-app notifications
- **Search**: ค้นหา users และ posts

### 1.2 Tech Stack

| Layer | Technology | Rationale |
|-------|-----------|-----------|
| UI | SwiftUI | Modern declarative UI, iOS 16+ |
| Architecture | Clean + MVVM + Modular SPM | Testability, scalability |
| Networking | async/await + URLSession | Native, no dependencies |
| Real-time | WebSocket (URLSessionWebSocketTask) | Native WebSocket support |
| Local DB | Core Data | Offline support, proven |
| Auth | Sign in with Apple | Privacy, App Store requirement |
| Cache | NSCache + FileManager | Performance |
| CI/CD | GitHub Actions + Fastlane | Automation |

### 1.3 Architecture: Clean + MVVM + Modular SPM

```
SwiftConnect/
├── App/                          # Main app target
│   ├── SwiftConnectApp.swift
│   └── ContentView.swift
├── Packages/                     # SPM local packages
│   ├── Core/                     # Shared models, protocols
│   ├── Networking/               # API client layer
│   ├── Auth/                     # Authentication module
│   ├── Features/
│   │   ├── Feed/                 # Feed feature
│   │   ├── Profile/              # Profile feature
│   │   ├── Messaging/            # Messaging feature
│   │   ├── Notifications/        # Notifications feature
│   │   └── Search/               # Search feature
│   └── DesignSystem/             # Shared UI components
└── Resources/                    # Assets, configs
```

---

## 2. Complete Architecture Setup

### 2.1 Root Package.swift

```swift
// Package.swift (root)
import PackageDescription

let package = Package(
    name: "SwiftConnect",
    platforms: [
        .iOS(.v16),
        .macOS(.v13)
    ],
    products: [
        .library(name: "Core", targets: ["Core"]),
        .library(name: "Networking", targets: ["Networking"]),
        .library(name: "Auth", targets: ["Auth"]),
        .library(name: "FeedFeature", targets: ["FeedFeature"]),
        .library(name: "ProfileFeature", targets: ["ProfileFeature"]),
        .library(name: "MessagingFeature", targets: ["MessagingFeature"]),
        .library(name: "SearchFeature", targets: ["SearchFeature"]),
        .library(name: "DesignSystem", targets: ["DesignSystem"]),
    ],
    dependencies: [
        // ไม่มี external dependencies หลัก - ใช้ Apple frameworks ทั้งหมด
    ],
    targets: [
        // Core - ไม่ depend on อะไรเลย
        .target(name: "Core", path: "Packages/Core/Sources"),
        .testTarget(name: "CoreTests", dependencies: ["Core"], path: "Packages/Core/Tests"),
        
        // Networking - depend on Core
        .target(name: "Networking", dependencies: ["Core"], path: "Packages/Networking/Sources"),
        .testTarget(name: "NetworkingTests", dependencies: ["Networking"], path: "Packages/Networking/Tests"),
        
        // Auth - depend on Core, Networking
        .target(name: "Auth", dependencies: ["Core", "Networking"], path: "Packages/Auth/Sources"),
        .testTarget(name: "AuthTests", dependencies: ["Auth"], path: "Packages/Auth/Tests"),
        
        // DesignSystem - depend on Core
        .target(name: "DesignSystem", dependencies: ["Core"], path: "Packages/DesignSystem/Sources"),
        
        // Features - depend on Core, Networking, DesignSystem
        .target(
            name: "FeedFeature",
            dependencies: ["Core", "Networking", "DesignSystem"],
            path: "Packages/Features/Feed/Sources"
        ),
        .testTarget(
            name: "FeedFeatureTests",
            dependencies: ["FeedFeature"],
            path: "Packages/Features/Feed/Tests"
        ),
        
        .target(
            name: "ProfileFeature",
            dependencies: ["Core", "Networking", "DesignSystem"],
            path: "Packages/Features/Profile/Sources"
        ),
        
        .target(
            name: "MessagingFeature",
            dependencies: ["Core", "Networking", "DesignSystem"],
            path: "Packages/Features/Messaging/Sources"
        ),
        
        .target(
            name: "SearchFeature",
            dependencies: ["Core", "Networking", "DesignSystem"],
            path: "Packages/Features/Search/Sources"
        ),
    ]
)
```

### 2.2 Core Models

```swift
// Packages/Core/Sources/Models/User.swift
import Foundation

public struct User: Identifiable, Codable, Hashable, Sendable {
    public let id: String
    public var firstName: String
    public var lastName: String
    public var email: String
    public var headline: String
    public var profileImageURL: URL?
    public var connectionCount: Int
    public var isConnected: Bool
    public var createdAt: Date
    
    public var fullName: String {
        "\(firstName) \(lastName)"
    }
    
    public init(
        id: String = UUID().uuidString,
        firstName: String,
        lastName: String,
        email: String,
        headline: String = "",
        profileImageURL: URL? = nil,
        connectionCount: Int = 0,
        isConnected: Bool = false,
        createdAt: Date = Date()
    ) {
        self.id = id
        self.firstName = firstName
        self.lastName = lastName
        self.email = email
        self.headline = headline
        self.profileImageURL = profileImageURL
        self.connectionCount = connectionCount
        self.isConnected = isConnected
        self.createdAt = createdAt
    }
}

// Packages/Core/Sources/Models/Post.swift
public struct Post: Identifiable, Codable, Hashable, Sendable {
    public let id: String
    public let authorId: String
    public var author: User?
    public var content: String
    public var imageURL: URL?
    public var likeCount: Int
    public var commentCount: Int
    public var isLiked: Bool
    public let createdAt: Date
    
    public init(
        id: String = UUID().uuidString,
        authorId: String,
        author: User? = nil,
        content: String,
        imageURL: URL? = nil,
        likeCount: Int = 0,
        commentCount: Int = 0,
        isLiked: Bool = false,
        createdAt: Date = Date()
    ) {
        self.id = id
        self.authorId = authorId
        self.author = author
        self.content = content
        self.imageURL = imageURL
        self.likeCount = likeCount
        self.commentCount = commentCount
        self.isLiked = isLiked
        self.createdAt = createdAt
    }
}

// Packages/Core/Sources/Models/Message.swift
public struct Message: Identifiable, Codable, Hashable, Sendable {
    public let id: String
    public let conversationId: String
    public let senderId: String
    public var content: String
    public var isRead: Bool
    public var isTyping: Bool
    public let sentAt: Date
    
    public var isFromCurrentUser: Bool = false
    
    public init(
        id: String = UUID().uuidString,
        conversationId: String,
        senderId: String,
        content: String,
        isRead: Bool = false,
        isTyping: Bool = false,
        sentAt: Date = Date()
    ) {
        self.id = id
        self.conversationId = conversationId
        self.senderId = senderId
        self.content = content
        self.isRead = isRead
        self.isTyping = isTyping
        self.sentAt = sentAt
    }
}

// Packages/Core/Sources/Protocols/Repository.swift
public protocol Repository<Entity> {
    associatedtype Entity
    
    func fetchAll() async throws -> [Entity]
    func fetch(id: String) async throws -> Entity?
    func save(_ entity: Entity) async throws
    func delete(id: String) async throws
}

// Packages/Core/Sources/Protocols/UseCase.swift
public protocol UseCase {
    associatedtype Input
    associatedtype Output
    
    func execute(_ input: Input) async throws -> Output
}
```

---

## 3. Authentication Module

### 3.1 Sign in with Apple

```swift
// Packages/Auth/Sources/SignInWithApple/AppleSignInManager.swift
import AuthenticationServices
import Foundation

@MainActor
public final class AppleSignInManager: NSObject, ObservableObject {
    @Published public private(set) var isSignedIn: Bool = false
    @Published public private(set) var currentUser: AuthUser?
    @Published public private(set) var error: AuthError?
    
    private let tokenManager: TokenManager
    private let keychainService: KeychainService
    
    public init(
        tokenManager: TokenManager = .shared,
        keychainService: KeychainService = .shared
    ) {
        self.tokenManager = tokenManager
        self.keychainService = keychainService
        super.init()
        
        // ตรวจสอบ existing session
        Task { await checkExistingSession() }
    }
    
    public func signIn() async throws -> AuthUser {
        return try await withCheckedThrowingContinuation { continuation in
            let request = ASAuthorizationAppleIDProvider().createRequest()
            request.requestedScopes = [.fullName, .email]
            
            let controller = ASAuthorizationController(authorizationRequests: [request])
            controller.delegate = self
            controller.presentationContextProvider = self
            
            // Store continuation สำหรับใช้ใน delegate
            self.signInContinuation = continuation
            controller.performRequests()
        }
    }
    
    public func signOut() async throws {
        try keychainService.deleteTokens()
        currentUser = nil
        isSignedIn = false
    }
    
    private func checkExistingSession() async {
        guard let tokens = keychainService.loadTokens() else { return }
        
        // ตรวจสอบ token expiry
        if tokens.isAccessTokenValid {
            isSignedIn = true
            currentUser = keychainService.loadUser()
        } else if tokens.isRefreshTokenValid {
            // ลอง refresh
            do {
                let newTokens = try await tokenManager.refresh(tokens.refreshToken)
                try keychainService.saveTokens(newTokens)
                isSignedIn = true
                currentUser = keychainService.loadUser()
            } catch {
                // Refresh failed - logout
                try? keychainService.deleteTokens()
            }
        }
    }
    
    private var signInContinuation: CheckedContinuation<AuthUser, Error>?
}

extension AppleSignInManager: ASAuthorizationControllerDelegate {
    public nonisolated func authorizationController(
        controller: ASAuthorizationController,
        didCompleteWithAuthorization authorization: ASAuthorization
    ) {
        Task { @MainActor in
            guard let appleIDCredential = authorization.credential as? ASAuthorizationAppleIDCredential else {
                signInContinuation?.resume(throwing: AuthError.invalidCredential)
                signInContinuation = nil
                return
            }
            
            do {
                // Exchange Apple credential กับ backend token
                guard let identityToken = appleIDCredential.identityToken,
                      let tokenString = String(data: identityToken, encoding: .utf8) else {
                    throw AuthError.invalidCredential
                }
                
                let authResult = try await tokenManager.exchangeAppleToken(
                    identityToken: tokenString,
                    fullName: appleIDCredential.fullName,
                    email: appleIDCredential.email
                )
                
                // บันทึก tokens
                try keychainService.saveTokens(authResult.tokens)
                try keychainService.saveUser(authResult.user)
                
                currentUser = authResult.user
                isSignedIn = true
                
                signInContinuation?.resume(returning: authResult.user)
            } catch {
                signInContinuation?.resume(throwing: error)
            }
            
            signInContinuation = nil
        }
    }
    
    public nonisolated func authorizationController(
        controller: ASAuthorizationController,
        didCompleteWithError error: Error
    ) {
        Task { @MainActor in
            if let authError = error as? ASAuthorizationError,
               authError.code == .canceled {
                signInContinuation?.resume(throwing: AuthError.cancelled)
            } else {
                signInContinuation?.resume(throwing: AuthError.underlying(error))
            }
            signInContinuation = nil
        }
    }
}

extension AppleSignInManager: ASAuthorizationControllerPresentationContextProviding {
    public nonisolated func presentationAnchor(
        for controller: ASAuthorizationController
    ) -> ASPresentationAnchor {
        // Return key window
        return UIApplication.shared.connectedScenes
            .compactMap { $0 as? UIWindowScene }
            .flatMap { $0.windows }
            .first { $0.isKeyWindow } ?? UIWindow()
    }
}

public enum AuthError: LocalizedError {
    case invalidCredential
    case cancelled
    case tokenExpired
    case refreshFailed
    case underlying(Error)
    
    public var errorDescription: String? {
        switch self {
        case .invalidCredential: return "Credential ไม่ถูกต้อง"
        case .cancelled: return "ยกเลิกการเข้าสู่ระบบ"
        case .tokenExpired: return "Session หมดอายุ"
        case .refreshFailed: return "ไม่สามารถ refresh token ได้"
        case .underlying(let error): return error.localizedDescription
        }
    }
}
```

### 3.2 Token Management

```swift
// Packages/Auth/Sources/Tokens/TokenManager.swift
import Foundation

public struct AuthTokens: Codable {
    public let accessToken: String
    public let refreshToken: String
    public let accessTokenExpiresAt: Date
    public let refreshTokenExpiresAt: Date
    
    public var isAccessTokenValid: Bool {
        accessTokenExpiresAt > Date().addingTimeInterval(60) // 1 minute buffer
    }
    
    public var isRefreshTokenValid: Bool {
        refreshTokenExpiresAt > Date().addingTimeInterval(300) // 5 minute buffer
    }
}

public struct AuthUser: Codable, Sendable {
    public let id: String
    public let email: String?
    public var firstName: String?
    public var lastName: String?
    public var profileImageURL: URL?
}

public struct AuthResult {
    public let user: AuthUser
    public let tokens: AuthTokens
}

public actor TokenManager {
    public static let shared = TokenManager()
    
    private let apiClient: AuthAPIClient
    private var refreshTask: Task<AuthTokens, Error>?
    
    init(apiClient: AuthAPIClient = AuthAPIClient()) {
        self.apiClient = apiClient
    }
    
    public func exchangeAppleToken(
        identityToken: String,
        fullName: PersonNameComponents?,
        email: String?
    ) async throws -> AuthResult {
        let request = ExchangeAppleTokenRequest(
            identityToken: identityToken,
            firstName: fullName?.givenName,
            lastName: fullName?.familyName,
            email: email
        )
        return try await apiClient.exchangeAppleToken(request)
    }
    
    // Refresh token พร้อม deduplication
    // ถ้ามีการ refresh อยู่แล้ว ให้รอผลลัพธ์จาก task เดิม
    public func refresh(_ refreshToken: String) async throws -> AuthTokens {
        if let existingTask = refreshTask {
            return try await existingTask.value
        }
        
        let task = Task<AuthTokens, Error> {
            defer { refreshTask = nil }
            return try await apiClient.refreshTokens(refreshToken: refreshToken)
        }
        
        refreshTask = task
        return try await task.value
    }
}

// Packages/Auth/Sources/Keychain/KeychainService.swift
import Security
import Foundation

public final class KeychainService {
    public static let shared = KeychainService()
    
    private let tokenKey = "com.swiftconnect.tokens"
    private let userKey = "com.swiftconnect.user"
    
    public func saveTokens(_ tokens: AuthTokens) throws {
        let data = try JSONEncoder().encode(tokens)
        try save(data, forKey: tokenKey)
    }
    
    public func loadTokens() -> AuthTokens? {
        guard let data = load(forKey: tokenKey) else { return nil }
        return try? JSONDecoder().decode(AuthTokens.self, from: data)
    }
    
    public func saveUser(_ user: AuthUser) throws {
        let data = try JSONEncoder().encode(user)
        try save(data, forKey: userKey)
    }
    
    public func loadUser() -> AuthUser? {
        guard let data = load(forKey: userKey) else { return nil }
        return try? JSONDecoder().decode(AuthUser.self, from: data)
    }
    
    public func deleteTokens() throws {
        try delete(forKey: tokenKey)
        try delete(forKey: userKey)
    }
    
    private func save(_ data: Data, forKey key: String) throws {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: key,
            kSecValueData as String: data,
            kSecAttrAccessible as String: kSecAttrAccessibleAfterFirstUnlock
        ]
        
        SecItemDelete(query as CFDictionary)
        
        let status = SecItemAdd(query as CFDictionary, nil)
        guard status == errSecSuccess else {
            throw KeychainError.saveFailed(status)
        }
    }
    
    private func load(forKey key: String) -> Data? {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: key,
            kSecReturnData as String: true,
            kSecMatchLimit as String: kSecMatchLimitOne
        ]
        
        var result: AnyObject?
        let status = SecItemCopyMatching(query as CFDictionary, &result)
        
        guard status == errSecSuccess else { return nil }
        return result as? Data
    }
    
    private func delete(forKey key: String) throws {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: key
        ]
        
        let status = SecItemDelete(query as CFDictionary)
        guard status == errSecSuccess || status == errSecItemNotFound else {
            throw KeychainError.deleteFailed(status)
        }
    }
    
    enum KeychainError: LocalizedError {
        case saveFailed(OSStatus)
        case deleteFailed(OSStatus)
        
        var errorDescription: String? {
            switch self {
            case .saveFailed(let status): return "Keychain save failed: \(status)"
            case .deleteFailed(let status): return "Keychain delete failed: \(status)"
            }
        }
    }
}
```

---

## 4. Networking Layer

### 4.1 Generic API Client

```swift
// Packages/Networking/Sources/APIClient/APIClient.swift
import Foundation

// HTTP Method
public enum HTTPMethod: String {
    case GET, POST, PUT, PATCH, DELETE
}

// API Request Protocol
public protocol APIRequest {
    associatedtype Response: Decodable
    
    var path: String { get }
    var method: HTTPMethod { get }
    var headers: [String: String] { get }
    var queryParameters: [String: String] { get }
    var body: Encodable? { get }
}

extension APIRequest {
    public var headers: [String: String] { [:] }
    public var queryParameters: [String: String] { [:] }
    public var body: Encodable? { nil }
}

// API Error
public enum APIError: LocalizedError {
    case invalidURL
    case networkError(Error)
    case httpError(statusCode: Int, data: Data)
    case decodingError(Error)
    case unauthorized
    case notFound
    case serverError(String)
    case offline
    
    public var errorDescription: String? {
        switch self {
        case .invalidURL: return "URL ไม่ถูกต้อง"
        case .networkError(let error): return "Network error: \(error.localizedDescription)"
        case .httpError(let code, _): return "HTTP Error: \(code)"
        case .decodingError(let error): return "Decoding error: \(error.localizedDescription)"
        case .unauthorized: return "ไม่มีสิทธิ์เข้าถึง กรุณาเข้าสู่ระบบใหม่"
        case .notFound: return "ไม่พบข้อมูลที่ต้องการ"
        case .serverError(let msg): return "Server error: \(msg)"
        case .offline: return "ไม่มีการเชื่อมต่ออินเทอร์เน็ต"
        }
    }
}

// Main API Client
public actor APIClient {
    public static let shared = APIClient(
        baseURL: URL(string: "https://api.swiftconnect.com/v1")!
    )
    
    private let baseURL: URL
    private let session: URLSession
    private let decoder: JSONDecoder
    private let encoder: JSONEncoder
    private let keychainService: KeychainService
    private let tokenManager: TokenManager
    
    // Retry configuration
    private let maxRetryCount = 3
    private let retryDelay: TimeInterval = 1.0
    
    public init(
        baseURL: URL,
        session: URLSession = .shared,
        keychainService: KeychainService = .shared,
        tokenManager: TokenManager = .shared
    ) {
        self.baseURL = baseURL
        self.session = session
        self.keychainService = keychainService
        self.tokenManager = tokenManager
        
        self.decoder = JSONDecoder()
        self.decoder.dateDecodingStrategy = .iso8601
        self.decoder.keyDecodingStrategy = .convertFromSnakeCase
        
        self.encoder = JSONEncoder()
        self.encoder.dateEncodingStrategy = .iso8601
        self.encoder.keyEncodingStrategy = .convertToSnakeCase
    }
    
    public func send<Request: APIRequest>(_ request: Request) async throws -> Request.Response {
        return try await sendWithRetry(request, retryCount: 0)
    }
    
    private func sendWithRetry<Request: APIRequest>(
        _ request: Request,
        retryCount: Int
    ) async throws -> Request.Response {
        do {
            return try await performRequest(request)
        } catch APIError.unauthorized where retryCount == 0 {
            // ลอง refresh token
            guard let tokens = keychainService.loadTokens() else {
                throw APIError.unauthorized
            }
            
            let newTokens = try await tokenManager.refresh(tokens.refreshToken)
            try keychainService.saveTokens(newTokens)
            
            return try await sendWithRetry(request, retryCount: retryCount + 1)
        } catch APIError.networkError where retryCount < maxRetryCount {
            // Retry on network errors
            try await Task.sleep(nanoseconds: UInt64(retryDelay * 1_000_000_000))
            return try await sendWithRetry(request, retryCount: retryCount + 1)
        }
    }
    
    private func performRequest<Request: APIRequest>(_ request: Request) async throws -> Request.Response {
        // Build URL
        var urlComponents = URLComponents(url: baseURL.appendingPathComponent(request.path), resolvingAgainstBaseURL: true)!
        
        if !request.queryParameters.isEmpty {
            urlComponents.queryItems = request.queryParameters.map {
                URLQueryItem(name: $0.key, value: $0.value)
            }
        }
        
        guard let url = urlComponents.url else {
            throw APIError.invalidURL
        }
        
        // Build URLRequest
        var urlRequest = URLRequest(url: url)
        urlRequest.httpMethod = request.method.rawValue
        urlRequest.timeoutInterval = 30
        
        // Headers
        urlRequest.setValue("application/json", forHTTPHeaderField: "Content-Type")
        urlRequest.setValue("application/json", forHTTPHeaderField: "Accept")
        urlRequest.setValue("SwiftConnect-iOS/1.0", forHTTPHeaderField: "User-Agent")
        
        // Auth header
        if let tokens = keychainService.loadTokens(), tokens.isAccessTokenValid {
            urlRequest.setValue("Bearer \(tokens.accessToken)", forHTTPHeaderField: "Authorization")
        }
        
        // Custom headers
        for (key, value) in request.headers {
            urlRequest.setValue(value, forHTTPHeaderField: key)
        }
        
        // Body
        if let body = request.body {
            urlRequest.httpBody = try encoder.encode(AnyEncodable(body))
        }
        
        // Execute request
        let (data, response): (Data, URLResponse)
        do {
            (data, response) = try await session.data(for: urlRequest)
        } catch let urlError as URLError {
            if urlError.code == .notConnectedToInternet || urlError.code == .networkConnectionLost {
                throw APIError.offline
            }
            throw APIError.networkError(urlError)
        }
        
        // Validate response
        guard let httpResponse = response as? HTTPURLResponse else {
            throw APIError.networkError(URLError(.badServerResponse))
        }
        
        switch httpResponse.statusCode {
        case 200...299:
            break
        case 401:
            throw APIError.unauthorized
        case 404:
            throw APIError.notFound
        case 400...499:
            throw APIError.httpError(statusCode: httpResponse.statusCode, data: data)
        case 500...599:
            let message = String(data: data, encoding: .utf8) ?? "Unknown server error"
            throw APIError.serverError(message)
        default:
            throw APIError.httpError(statusCode: httpResponse.statusCode, data: data)
        }
        
        // Decode response
        do {
            return try decoder.decode(Request.Response.self, from: data)
        } catch {
            throw APIError.decodingError(error)
        }
    }
}

// Helper สำหรับ encode any Encodable
private struct AnyEncodable: Encodable {
    private let _encode: (Encoder) throws -> Void
    
    init<T: Encodable>(_ value: T) {
        _encode = value.encode
    }
    
    func encode(to encoder: Encoder) throws {
        try _encode(encoder)
    }
}
```

### 4.2 Concrete API Requests

```swift
// Packages/Networking/Sources/Requests/FeedRequests.swift
import Foundation
import Core

public struct GetFeedRequest: APIRequest {
    public typealias Response = PaginatedResponse<Post>
    
    public let path = "/feed"
    public let method: HTTPMethod = .GET
    public let queryParameters: [String: String]
    
    public init(page: Int = 1, limit: Int = 20) {
        queryParameters = [
            "page": "\(page)",
            "limit": "\(limit)"
        ]
    }
}

public struct CreatePostRequest: APIRequest {
    public typealias Response = Post
    
    public let path = "/posts"
    public let method: HTTPMethod = .POST
    public let body: Encodable?
    
    struct Body: Encodable {
        let content: String
        let imageData: Data?
    }
    
    public init(content: String, imageData: Data? = nil) {
        body = Body(content: content, imageData: imageData)
    }
}

public struct LikePostRequest: APIRequest {
    public typealias Response = LikeResponse
    
    public let path: String
    public let method: HTTPMethod = .POST
    
    public init(postId: String) {
        path = "/posts/\(postId)/like"
    }
}

public struct LikeResponse: Decodable {
    public let liked: Bool
    public let likeCount: Int
}

public struct PaginatedResponse<T: Decodable>: Decodable {
    public let data: [T]
    public let totalCount: Int
    public let hasNextPage: Bool
    public let nextPage: Int?
}
```

### 4.3 Offline Detection

```swift
// Packages/Networking/Sources/Connectivity/NetworkMonitor.swift
import Network
import Foundation
import Combine

@MainActor
public final class NetworkMonitor: ObservableObject {
    public static let shared = NetworkMonitor()
    
    @Published public private(set) var isConnected: Bool = true
    @Published public private(set) var connectionType: ConnectionType = .unknown
    
    private let monitor: NWPathMonitor
    private let queue = DispatchQueue(label: "NetworkMonitor")
    
    public enum ConnectionType {
        case wifi, cellular, ethernet, unknown
    }
    
    private init() {
        monitor = NWPathMonitor()
        monitor.pathUpdateHandler = { [weak self] path in
            Task { @MainActor in
                self?.isConnected = path.status == .satisfied
                self?.connectionType = self?.getConnectionType(path) ?? .unknown
            }
        }
        monitor.start(queue: queue)
    }
    
    deinit {
        monitor.cancel()
    }
    
    private func getConnectionType(_ path: NWPath) -> ConnectionType {
        if path.usesInterfaceType(.wifi) { return .wifi }
        if path.usesInterfaceType(.cellular) { return .cellular }
        if path.usesInterfaceType(.wiredEthernet) { return .ethernet }
        return .unknown
    }
}
```

---

## 5. User Profile Feature

### 5.1 Profile ViewModel

```swift
// Packages/Features/Profile/Sources/ProfileViewModel.swift
import Foundation
import Core
import Networking
import SwiftUI

@MainActor
public final class ProfileViewModel: ObservableObject {
    // MARK: - Published State
    @Published public private(set) var user: User?
    @Published public private(set) var posts: [Post] = []
    @Published public private(set) var isLoading = false
    @Published public private(set) var isSaving = false
    @Published public private(set) var error: Error?
    @Published public var isEditMode = false
    
    // Edit state
    @Published public var editFirstName = ""
    @Published public var editLastName = ""
    @Published public var editHeadline = ""
    @Published public var selectedImage: UIImage?
    
    // MARK: - Dependencies
    private let apiClient: APIClient
    private let imageCache: ImageCache
    private let userId: String
    
    public init(
        userId: String,
        apiClient: APIClient = .shared,
        imageCache: ImageCache = .shared
    ) {
        self.userId = userId
        self.apiClient = apiClient
        self.imageCache = imageCache
    }
    
    // MARK: - Methods
    public func loadProfile() async {
        isLoading = true
        defer { isLoading = false }
        
        do {
            async let userRequest = apiClient.send(GetUserRequest(userId: userId))
            async let postsRequest = apiClient.send(GetUserPostsRequest(userId: userId))
            
            let (fetchedUser, fetchedPosts) = try await (userRequest, postsRequest)
            
            user = fetchedUser
            posts = fetchedPosts.data
            
            // Populate edit fields
            editFirstName = fetchedUser.firstName
            editLastName = fetchedUser.lastName
            editHeadline = fetchedUser.headline
        } catch {
            self.error = error
        }
    }
    
    public func saveProfile() async {
        guard var updatedUser = user else { return }
        
        isSaving = true
        defer { isSaving = false }
        
        do {
            // Upload image ถ้ามีการเลือกใหม่
            var imageURL: URL? = updatedUser.profileImageURL
            if let image = selectedImage {
                imageURL = try await uploadProfileImage(image)
            }
            
            updatedUser.firstName = editFirstName
            updatedUser.lastName = editLastName
            updatedUser.headline = editHeadline
            updatedUser.profileImageURL = imageURL
            
            let savedUser = try await apiClient.send(
                UpdateUserRequest(user: updatedUser)
            )
            
            user = savedUser
            isEditMode = false
            selectedImage = nil
        } catch {
            self.error = error
        }
    }
    
    private func uploadProfileImage(_ image: UIImage) async throws -> URL {
        // Compress image
        guard let compressed = image.jpegData(compressionQuality: 0.8) else {
            throw ProfileError.imageCompressionFailed
        }
        
        // Upload
        let response = try await apiClient.send(
            UploadProfileImageRequest(imageData: compressed)
        )
        
        return response.imageURL
    }
    
    enum ProfileError: LocalizedError {
        case imageCompressionFailed
        
        var errorDescription: String? {
            "ไม่สามารถ compress รูปภาพได้"
        }
    }
}
```

### 5.2 Profile View

```swift
// Packages/Features/Profile/Sources/ProfileView.swift
import SwiftUI
import Core
import DesignSystem
import PhotosUI

public struct ProfileView: View {
    @StateObject private var viewModel: ProfileViewModel
    @State private var photosPickerItem: PhotosPickerItem?
    
    public init(userId: String) {
        _viewModel = StateObject(
            wrappedValue: ProfileViewModel(userId: userId)
        )
    }
    
    public var body: some View {
        ScrollView {
            VStack(spacing: 0) {
                // Header
                profileHeader
                
                // Stats
                statsSection
                
                Divider()
                
                // Posts
                postsSection
            }
        }
        .navigationBarTitleDisplayMode(.inline)
        .toolbar {
            if !viewModel.isEditMode {
                Button("Edit") {
                    viewModel.isEditMode = true
                }
            } else {
                HStack {
                    Button("Cancel") {
                        viewModel.isEditMode = false
                    }
                    Button("Save") {
                        Task { await viewModel.saveProfile() }
                    }
                    .fontWeight(.semibold)
                    .disabled(viewModel.isSaving)
                }
            }
        }
        .task { await viewModel.loadProfile() }
        .overlay {
            if viewModel.isLoading {
                ProgressView()
                    .frame(maxWidth: .infinity, maxHeight: .infinity)
                    .background(.ultraThinMaterial)
            }
        }
        .alert("Error", isPresented: Binding(
            get: { viewModel.error != nil },
            set: { if !$0 { viewModel.error = nil } }
        )) {
            Button("OK") { viewModel.error = nil }
        } message: {
            Text(viewModel.error?.localizedDescription ?? "")
        }
        .onChange(of: photosPickerItem) { item in
            Task {
                if let data = try? await item?.loadTransferable(type: Data.self),
                   let image = UIImage(data: data) {
                    viewModel.selectedImage = image
                }
            }
        }
    }
    
    private var profileHeader: some View {
        VStack(spacing: 16) {
            // Profile Image
            PhotosPicker(selection: $photosPickerItem, matching: .images) {
                ZStack(alignment: .bottomTrailing) {
                    Group {
                        if let selectedImage = viewModel.selectedImage {
                            Image(uiImage: selectedImage)
                                .resizable()
                                .scaledToFill()
                        } else if let url = viewModel.user?.profileImageURL {
                            AsyncImage(url: url) { phase in
                                switch phase {
                                case .success(let image):
                                    image.resizable().scaledToFill()
                                default:
                                    Color.secondary
                                }
                            }
                        } else {
                            Color.secondary
                                .overlay(
                                    Image(systemName: "person.fill")
                                        .font(.system(size: 40))
                                        .foregroundStyle(.white)
                                )
                        }
                    }
                    .frame(width: 100, height: 100)
                    .clipShape(Circle())
                    
                    if viewModel.isEditMode {
                        Circle()
                            .fill(Color.accentColor)
                            .frame(width: 30, height: 30)
                            .overlay(
                                Image(systemName: "camera.fill")
                                    .font(.system(size: 14))
                                    .foregroundStyle(.white)
                            )
                    }
                }
            }
            .disabled(!viewModel.isEditMode)
            
            // Name
            if viewModel.isEditMode {
                VStack(spacing: 8) {
                    HStack {
                        TextField("First name", text: $viewModel.editFirstName)
                            .textFieldStyle(.roundedBorder)
                        TextField("Last name", text: $viewModel.editLastName)
                            .textFieldStyle(.roundedBorder)
                    }
                    TextField("Headline", text: $viewModel.editHeadline)
                        .textFieldStyle(.roundedBorder)
                }
                .padding(.horizontal)
            } else if let user = viewModel.user {
                Text(user.fullName)
                    .font(.title2)
                    .fontWeight(.bold)
                
                Text(user.headline)
                    .font(.subheadline)
                    .foregroundStyle(.secondary)
                    .multilineTextAlignment(.center)
                    .padding(.horizontal)
            }
        }
        .padding(.vertical, 20)
    }
    
    private var statsSection: some View {
        HStack(spacing: 30) {
            statItem(value: viewModel.user?.connectionCount ?? 0, label: "Connections")
            statItem(value: viewModel.posts.count, label: "Posts")
        }
        .padding(.vertical, 16)
    }
    
    private func statItem(value: Int, label: String) -> some View {
        VStack(spacing: 4) {
            Text("\(value)")
                .font(.title3)
                .fontWeight(.bold)
            Text(label)
                .font(.caption)
                .foregroundStyle(.secondary)
        }
    }
    
    private var postsSection: some View {
        LazyVStack(spacing: 0) {
            ForEach(viewModel.posts) { post in
                PostCardView(post: post)
                Divider()
            }
        }
    }
}
```

---

## 6. Feed Feature

### 6.1 Infinite Scroll Implementation

```swift
// Packages/Features/Feed/Sources/FeedViewModel.swift
import Foundation
import Core
import Networking
import Combine

@MainActor
public final class FeedViewModel: ObservableObject {
    // MARK: - Published State
    @Published public private(set) var posts: [Post] = []
    @Published public private(set) var isLoadingInitial = false
    @Published public private(set) var isLoadingMore = false
    @Published public private(set) var hasNextPage = true
    @Published public private(set) var error: Error?
    
    // MARK: - Pagination
    private var currentPage = 1
    private let pageSize = 20
    private var isPerformingRequest = false
    
    // MARK: - Dependencies
    private let apiClient: APIClient
    
    public init(apiClient: APIClient = .shared) {
        self.apiClient = apiClient
    }
    
    // MARK: - Methods
    
    public func loadInitialFeed() async {
        guard !isLoadingInitial else { return }
        
        isLoadingInitial = true
        currentPage = 1
        defer { isLoadingInitial = false }
        
        do {
            let response = try await apiClient.send(
                GetFeedRequest(page: 1, limit: pageSize)
            )
            posts = response.data
            hasNextPage = response.hasNextPage
        } catch {
            self.error = error
        }
    }
    
    public func loadMoreIfNeeded(currentPost: Post) async {
        // ตรวจสอบว่าถึงท้าย list หรือยัง
        guard let lastPost = posts.last,
              lastPost.id == currentPost.id,
              hasNextPage,
              !isLoadingMore,
              !isPerformingRequest else { return }
        
        await loadNextPage()
    }
    
    private func loadNextPage() async {
        isLoadingMore = true
        isPerformingRequest = true
        defer {
            isLoadingMore = false
            isPerformingRequest = false
        }
        
        do {
            currentPage += 1
            let response = try await apiClient.send(
                GetFeedRequest(page: currentPage, limit: pageSize)
            )
            posts.append(contentsOf: response.data)
            hasNextPage = response.hasNextPage
        } catch {
            currentPage -= 1 // Rollback
            self.error = error
        }
    }
    
    public func refresh() async {
        await loadInitialFeed()
    }
    
    // MARK: - Post Actions
    
    public func toggleLike(post: Post) async {
        // Optimistic update
        guard let index = posts.firstIndex(where: { $0.id == post.id }) else { return }
        
        let wasLiked = posts[index].isLiked
        posts[index].isLiked = !wasLiked
        posts[index].likeCount += wasLiked ? -1 : 1
        
        do {
            let response = try await apiClient.send(LikePostRequest(postId: post.id))
            
            // Sync with server response
            posts[index].isLiked = response.liked
            posts[index].likeCount = response.likeCount
        } catch {
            // Rollback optimistic update
            posts[index].isLiked = wasLiked
            posts[index].likeCount += wasLiked ? 1 : -1
            self.error = error
        }
    }
    
    public func createPost(content: String, image: UIImage?) async -> Bool {
        var imageData: Data? = nil
        if let image = image {
            imageData = image.jpegData(compressionQuality: 0.8)
        }
        
        do {
            let newPost = try await apiClient.send(
                CreatePostRequest(content: content, imageData: imageData)
            )
            posts.insert(newPost, at: 0)
            return true
        } catch {
            self.error = error
            return false
        }
    }
}
```

### 6.2 Feed View

```swift
// Packages/Features/Feed/Sources/FeedView.swift
import SwiftUI
import Core
import DesignSystem

public struct FeedView: View {
    @StateObject private var viewModel = FeedViewModel()
    @State private var showCreatePost = false
    
    public init() {}
    
    public var body: some View {
        NavigationStack {
            Group {
                if viewModel.isLoadingInitial {
                    feedSkeleton
                } else if viewModel.posts.isEmpty {
                    emptyState
                } else {
                    feedList
                }
            }
            .navigationTitle("SwiftConnect")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                Button {
                    showCreatePost = true
                } label: {
                    Image(systemName: "square.and.pencil")
                }
            }
            .refreshable {
                await viewModel.refresh()
            }
            .sheet(isPresented: $showCreatePost) {
                CreatePostView { content, image in
                    Task {
                        let success = await viewModel.createPost(content: content, image: image)
                        if success { showCreatePost = false }
                    }
                }
            }
        }
        .task {
            await viewModel.loadInitialFeed()
        }
    }
    
    private var feedList: some View {
        ScrollView {
            LazyVStack(spacing: 0) {
                ForEach(viewModel.posts) { post in
                    PostCardView(
                        post: post,
                        onLike: {
                            Task { await viewModel.toggleLike(post: post) }
                        }
                    )
                    .task {
                        // Trigger pagination
                        await viewModel.loadMoreIfNeeded(currentPost: post)
                    }
                    
                    Divider()
                }
                
                if viewModel.isLoadingMore {
                    ProgressView()
                        .padding()
                }
            }
        }
    }
    
    private var feedSkeleton: some View {
        ScrollView {
            LazyVStack(spacing: 0) {
                ForEach(0..<5) { _ in
                    PostCardSkeleton()
                    Divider()
                }
            }
        }
    }
    
    private var emptyState: some View {
        VStack(spacing: 16) {
            Image(systemName: "newspaper")
                .font(.system(size: 60))
                .foregroundStyle(.secondary)
            
            Text("ยังไม่มีโพสต์")
                .font(.title3)
                .fontWeight(.semibold)
            
            Text("ติดตามคนอื่นหรือสร้างโพสต์แรกของคุณ!")
                .font(.subheadline)
                .foregroundStyle(.secondary)
                .multilineTextAlignment(.center)
            
            Button("สร้างโพสต์") {
                showCreatePost = true
            }
            .buttonStyle(.borderedProminent)
        }
        .padding()
    }
}

// Post Card View
public struct PostCardView: View {
    let post: Post
    var onLike: (() -> Void)?
    
    public var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            // Author row
            if let author = post.author {
                HStack(spacing: 10) {
                    AsyncImage(url: author.profileImageURL) { image in
                        image.resizable().scaledToFill()
                    } placeholder: {
                        Color.secondary
                    }
                    .frame(width: 44, height: 44)
                    .clipShape(Circle())
                    
                    VStack(alignment: .leading, spacing: 2) {
                        Text(author.fullName)
                            .font(.subheadline)
                            .fontWeight(.semibold)
                        Text(author.headline)
                            .font(.caption)
                            .foregroundStyle(.secondary)
                            .lineLimit(1)
                    }
                    
                    Spacer()
                    
                    Text(post.createdAt, style: .relative)
                        .font(.caption)
                        .foregroundStyle(.secondary)
                }
            }
            
            // Content
            Text(post.content)
                .font(.body)
                .lineLimit(5)
            
            // Image
            if let imageURL = post.imageURL {
                AsyncImage(url: imageURL) { image in
                    image.resizable().scaledToFit()
                } placeholder: {
                    Color.secondary
                        .frame(height: 200)
                }
                .cornerRadius(8)
            }
            
            // Actions
            HStack(spacing: 20) {
                Button {
                    onLike?()
                } label: {
                    Label("\(post.likeCount)", systemImage: post.isLiked ? "heart.fill" : "heart")
                        .foregroundStyle(post.isLiked ? .red : .secondary)
                }
                
                Label("\(post.commentCount)", systemImage: "bubble.right")
                    .foregroundStyle(.secondary)
                
                Spacer()
                
                Button {
                    // Share
                } label: {
                    Image(systemName: "square.and.arrow.up")
                        .foregroundStyle(.secondary)
                }
            }
            .font(.subheadline)
        }
        .padding()
    }
}

public struct PostCardSkeleton: View {
    @State private var animating = false
    
    public var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            HStack(spacing: 10) {
                Circle()
                    .frame(width: 44, height: 44)
                VStack(alignment: .leading, spacing: 4) {
                    RoundedRectangle(cornerRadius: 4)
                        .frame(width: 120, height: 14)
                    RoundedRectangle(cornerRadius: 4)
                        .frame(width: 200, height: 12)
                }
            }
            
            VStack(alignment: .leading, spacing: 4) {
                ForEach(0..<3) { _ in
                    RoundedRectangle(cornerRadius: 4)
                        .frame(maxWidth: .infinity)
                        .frame(height: 12)
                }
            }
        }
        .padding()
        .redacted(reason: .placeholder)
        .shimmering(active: animating)
        .onAppear { animating = true }
    }
}
```

---

## 7. Real-time Messaging

### 7.1 WebSocket Manager

```swift
// Packages/Features/Messaging/Sources/WebSocket/WebSocketManager.swift
import Foundation
import Combine
import Core

public enum WebSocketEvent {
    case connected
    case disconnected(Error?)
    case messageReceived(Message)
    case typingIndicator(userId: String, conversationId: String, isTyping: Bool)
    case readReceipt(messageId: String, userId: String)
    case error(Error)
}

public actor WebSocketManager {
    private var webSocketTask: URLSessionWebSocketTask?
    private let session: URLSession
    private let baseURL: URL
    
    private let eventSubject = PassthroughSubject<WebSocketEvent, Never>()
    public nonisolated var events: AnyPublisher<WebSocketEvent, Never> {
        eventSubject.eraseToAnyPublisher()
    }
    
    private var reconnectTask: Task<Void, Never>?
    private var isConnected = false
    private var reconnectDelay: TimeInterval = 1.0
    private let maxReconnectDelay: TimeInterval = 60.0
    
    public init(
        baseURL: URL = URL(string: "wss://api.swiftconnect.com/ws")!,
        session: URLSession = .shared
    ) {
        self.baseURL = baseURL
        self.session = session
    }
    
    public func connect(token: String) async {
        var request = URLRequest(url: baseURL)
        request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        
        webSocketTask = session.webSocketTask(with: request)
        webSocketTask?.resume()
        
        isConnected = true
        reconnectDelay = 1.0
        eventSubject.send(.connected)
        
        // Start receiving
        await receiveMessages()
    }
    
    public func disconnect() {
        reconnectTask?.cancel()
        webSocketTask?.cancel(with: .goingAway, reason: nil)
        webSocketTask = nil
        isConnected = false
        eventSubject.send(.disconnected(nil))
    }
    
    public func sendMessage(_ message: Message) async throws {
        let encoder = JSONEncoder()
        encoder.dateEncodingStrategy = .iso8601
        
        let payload = WebSocketPayload(type: "message", data: message)
        let data = try encoder.encode(payload)
        let wsMessage = URLSessionWebSocketTask.Message.data(data)
        
        try await webSocketTask?.send(wsMessage)
    }
    
    public func sendTypingIndicator(conversationId: String, isTyping: Bool) async throws {
        let payload = TypingPayload(
            type: "typing",
            conversationId: conversationId,
            isTyping: isTyping
        )
        
        let data = try JSONEncoder().encode(payload)
        try await webSocketTask?.send(.data(data))
    }
    
    public func sendReadReceipt(messageId: String) async throws {
        let payload = ReadReceiptPayload(type: "read", messageId: messageId)
        let data = try JSONEncoder().encode(payload)
        try await webSocketTask?.send(.data(data))
    }
    
    private func receiveMessages() async {
        guard let task = webSocketTask else { return }
        
        do {
            while isConnected {
                let message = try await task.receive()
                
                switch message {
                case .data(let data):
                    handleIncomingData(data)
                case .string(let text):
                    if let data = text.data(using: .utf8) {
                        handleIncomingData(data)
                    }
                @unknown default:
                    break
                }
            }
        } catch {
            if isConnected {
                eventSubject.send(.disconnected(error))
                await scheduleReconnect()
            }
        }
    }
    
    private func handleIncomingData(_ data: Data) {
        let decoder = JSONDecoder()
        decoder.dateDecodingStrategy = .iso8601
        
        guard let payload = try? decoder.decode(IncomingPayload.self, from: data) else {
            return
        }
        
        switch payload.type {
        case "message":
            if let message = try? decoder.decode(Message.self, from: data) {
                eventSubject.send(.messageReceived(message))
            }
        case "typing":
            if let typing = try? decoder.decode(TypingPayload.self, from: data) {
                eventSubject.send(.typingIndicator(
                    userId: typing.userId ?? "",
                    conversationId: typing.conversationId,
                    isTyping: typing.isTyping
                ))
            }
        case "read":
            if let receipt = try? decoder.decode(ReadReceiptPayload.self, from: data) {
                eventSubject.send(.readReceipt(
                    messageId: receipt.messageId,
                    userId: receipt.userId ?? ""
                ))
            }
        default:
            break
        }
    }
    
    private func scheduleReconnect() async {
        reconnectTask?.cancel()
        reconnectTask = Task {
            try? await Task.sleep(nanoseconds: UInt64(reconnectDelay * 1_000_000_000))
            guard !Task.isCancelled else { return }
            
            reconnectDelay = min(reconnectDelay * 2, maxReconnectDelay)
            
            // ดึง fresh token
            if let tokens = KeychainService.shared.loadTokens(),
               tokens.isAccessTokenValid {
                await connect(token: tokens.accessToken)
            }
        }
    }
}

// Codable Payloads
struct WebSocketPayload<T: Encodable>: Encodable {
    let type: String
    let data: T
}

struct IncomingPayload: Decodable {
    let type: String
}

struct TypingPayload: Codable {
    let type: String
    let conversationId: String
    let isTyping: Bool
    var userId: String?
}

struct ReadReceiptPayload: Codable {
    let type: String
    let messageId: String
    var userId: String?
}
```

### 7.2 Message Persistence (Core Data)

```swift
// Packages/Features/Messaging/Sources/Persistence/MessagePersistence.swift
import CoreData
import Core

public final class MessagePersistenceController {
    public static let shared = MessagePersistenceController()
    
    private let container: NSPersistentContainer
    
    private init() {
        container = NSPersistentContainer(name: "SwiftConnect")
        container.loadPersistentStores { _, error in
            if let error = error {
                fatalError("Core Data failed: \(error)")
            }
        }
        container.viewContext.automaticallyMergesChangesFromParent = true
    }
    
    public var viewContext: NSManagedObjectContext {
        container.viewContext
    }
    
    // Save messages
    public func saveMessages(_ messages: [Message]) async throws {
        let context = container.newBackgroundContext()
        
        try await context.perform {
            for message in messages {
                let entity = MessageEntity(context: context)
                entity.id = message.id
                entity.conversationId = message.conversationId
                entity.senderId = message.senderId
                entity.content = message.content
                entity.isRead = message.isRead
                entity.sentAt = message.sentAt
            }
            
            try context.save()
        }
    }
    
    // Fetch messages for conversation
    public func fetchMessages(
        conversationId: String,
        limit: Int = 50,
        before date: Date? = nil
    ) async throws -> [Message] {
        let context = container.newBackgroundContext()
        
        return try await context.perform {
            let request = MessageEntity.fetchRequest()
            
            var predicates = [NSPredicate(
                format: "conversationId == %@", conversationId
            )]
            
            if let before = date {
                predicates.append(NSPredicate(
                    format: "sentAt < %@", before as NSDate
                ))
            }
            
            request.predicate = NSCompoundPredicate(
                andPredicateWithSubpredicates: predicates
            )
            request.sortDescriptors = [
                NSSortDescriptor(keyPath: \MessageEntity.sentAt, ascending: false)
            ]
            request.fetchLimit = limit
            
            let entities = try context.fetch(request)
            return entities.map { entity in
                Message(
                    id: entity.id ?? "",
                    conversationId: entity.conversationId ?? "",
                    senderId: entity.senderId ?? "",
                    content: entity.content ?? "",
                    isRead: entity.isRead,
                    sentAt: entity.sentAt ?? Date()
                )
            }
        }
    }
    
    // Mark message as read
    public func markAsRead(messageId: String) async throws {
        let context = container.newBackgroundContext()
        
        try await context.perform {
            let request = MessageEntity.fetchRequest()
            request.predicate = NSPredicate(format: "id == %@", messageId)
            
            guard let entity = try context.fetch(request).first else { return }
            entity.isRead = true
            try context.save()
        }
    }
}
```

### 7.3 Messaging ViewModel

```swift
// Packages/Features/Messaging/Sources/MessagingViewModel.swift
import Foundation
import Combine
import Core
import Networking

@MainActor
public final class MessagingViewModel: ObservableObject {
    @Published public private(set) var messages: [Message] = []
    @Published public private(set) var isLoading = false
    @Published public private(set) var typingUsers: Set<String> = []
    @Published public var draftMessage = ""
    
    private let conversationId: String
    private let currentUserId: String
    private let webSocketManager: WebSocketManager
    private let persistence: MessagePersistenceController
    private let apiClient: APIClient
    private var cancellables = Set<AnyCancellable>()
    private var typingTimer: Timer?
    private var isTyping = false
    
    public init(
        conversationId: String,
        currentUserId: String,
        webSocketManager: WebSocketManager,
        persistence: MessagePersistenceController = .shared,
        apiClient: APIClient = .shared
    ) {
        self.conversationId = conversationId
        self.currentUserId = currentUserId
        self.webSocketManager = webSocketManager
        self.persistence = persistence
        self.apiClient = apiClient
        
        setupWebSocketSubscription()
    }
    
    public func loadMessages() async {
        isLoading = true
        defer { isLoading = false }
        
        // Load from local cache first
        if let cached = try? await persistence.fetchMessages(
            conversationId: conversationId
        ) {
            messages = cached.reversed()
        }
        
        // Then fetch from server
        do {
            let response = try await apiClient.send(
                GetMessagesRequest(conversationId: conversationId)
            )
            messages = response.data.reversed()
            try? await persistence.saveMessages(response.data)
        } catch {
            // Keep cached data on error
        }
    }
    
    public func sendMessage() async {
        let content = draftMessage.trimmingCharacters(in: .whitespacesAndNewlines)
        guard !content.isEmpty else { return }
        
        let message = Message(
            conversationId: conversationId,
            senderId: currentUserId,
            content: content
        )
        
        // Optimistic insert
        messages.append(message)
        draftMessage = ""
        
        // Stop typing indicator
        stopTyping()
        
        do {
            try await webSocketManager.sendMessage(message)
            try? await persistence.saveMessages([message])
        } catch {
            // Rollback
            messages.removeLast()
            draftMessage = content
        }
    }
    
    public func handleTyping() {
        if !isTyping {
            isTyping = true
            Task {
                try? await webSocketManager.sendTypingIndicator(
                    conversationId: conversationId,
                    isTyping: true
                )
            }
        }
        
        // Reset timer
        typingTimer?.invalidate()
        typingTimer = Timer.scheduledTimer(
            withTimeInterval: 2.0,
            repeats: false
        ) { [weak self] _ in
            self?.stopTyping()
        }
    }
    
    private func stopTyping() {
        guard isTyping else { return }
        isTyping = false
        typingTimer?.invalidate()
        typingTimer = nil
        
        Task {
            try? await webSocketManager.sendTypingIndicator(
                conversationId: conversationId,
                isTyping: false
            )
        }
    }
    
    private func setupWebSocketSubscription() {
        webSocketManager.events
            .receive(on: DispatchQueue.main)
            .sink { [weak self] event in
                Task { @MainActor [weak self] in
                    self?.handleWebSocketEvent(event)
                }
            }
            .store(in: &cancellables)
    }
    
    private func handleWebSocketEvent(_ event: WebSocketEvent) {
        switch event {
        case .messageReceived(let message):
            guard message.conversationId == conversationId,
                  message.senderId != currentUserId else { return }
            
            messages.append(message)
            
            // Mark as read
            Task {
                try? await webSocketManager.sendReadReceipt(messageId: message.id)
                try? await persistence.markAsRead(messageId: message.id)
            }
            
        case .typingIndicator(let userId, let convId, let isTyping):
            guard convId == conversationId, userId != currentUserId else { return }
            
            if isTyping {
                typingUsers.insert(userId)
            } else {
                typingUsers.remove(userId)
            }
            
        case .readReceipt(let messageId, _):
            if let index = messages.firstIndex(where: { $0.id == messageId }) {
                messages[index].isRead = true
            }
            
        default:
            break
        }
    }
}
```

---

## 8. Push Notifications

### 8.1 APNs Registration

```swift
// App/Notifications/NotificationManager.swift
import UserNotifications
import UIKit
import Foundation
import Networking

@MainActor
public final class NotificationManager: NSObject, ObservableObject {
    public static let shared = NotificationManager()
    
    @Published public private(set) var authorizationStatus: UNAuthorizationStatus = .notDetermined
    
    private let center = UNUserNotificationCenter.current()
    private let apiClient: APIClient
    
    private override init() {
        apiClient = .shared
        super.init()
        center.delegate = self
    }
    
    public func requestAuthorization() async -> Bool {
        do {
            let granted = try await center.requestAuthorization(
                options: [.alert, .badge, .sound]
            )
            
            if granted {
                await UIApplication.shared.registerForRemoteNotifications()
            }
            
            await updateAuthorizationStatus()
            return granted
        } catch {
            return false
        }
    }
    
    public func registerDeviceToken(_ tokenData: Data) async {
        let token = tokenData.map { String(format: "%02.2hhx", $0) }.joined()
        
        do {
            try await apiClient.send(RegisterDeviceTokenRequest(
                token: token,
                platform: "ios",
                bundleId: Bundle.main.bundleIdentifier ?? ""
            ))
        } catch {
            print("Failed to register device token: \(error)")
        }
    }
    
    public func handleNotification(
        _ response: UNNotificationResponse
    ) -> NotificationAction {
        let userInfo = response.notification.request.content.userInfo
        return parseNotificationAction(from: userInfo)
    }
    
    private func parseNotificationAction(
        from userInfo: [AnyHashable: Any]
    ) -> NotificationAction {
        guard let type = userInfo["type"] as? String else {
            return .unknown
        }
        
        switch type {
        case "new_message":
            let conversationId = userInfo["conversation_id"] as? String ?? ""
            return .openConversation(id: conversationId)
        case "new_connection":
            let userId = userInfo["user_id"] as? String ?? ""
            return .openProfile(id: userId)
        case "post_liked", "post_commented":
            let postId = userInfo["post_id"] as? String ?? ""
            return .openPost(id: postId)
        default:
            return .unknown
        }
    }
    
    private func updateAuthorizationStatus() async {
        let settings = await center.notificationSettings()
        authorizationStatus = settings.authorizationStatus
    }
}

extension NotificationManager: UNUserNotificationCenterDelegate {
    public nonisolated func userNotificationCenter(
        _ center: UNUserNotificationCenter,
        willPresent notification: UNNotification,
        withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void
    ) {
        // แสดง notification แม้ app จะ foreground อยู่
        completionHandler([.banner, .badge, .sound])
    }
    
    public nonisolated func userNotificationCenter(
        _ center: UNUserNotificationCenter,
        didReceive response: UNNotificationResponse,
        withCompletionHandler completionHandler: @escaping () -> Void
    ) {
        Task { @MainActor in
            let action = NotificationManager.shared.handleNotification(response)
            NotificationCenter.default.post(
                name: .notificationActionReceived,
                object: action
            )
        }
        completionHandler()
    }
}

public enum NotificationAction {
    case openConversation(id: String)
    case openProfile(id: String)
    case openPost(id: String)
    case unknown
}

extension Notification.Name {
    public static let notificationActionReceived = Notification.Name("notificationActionReceived")
}
```

### 8.2 Deep Links

```swift
// App/DeepLink/DeepLinkHandler.swift
import Foundation
import SwiftUI

public enum DeepLink: Equatable {
    case profile(id: String)
    case post(id: String)
    case conversation(id: String)
    case search(query: String)
}

@MainActor
public final class DeepLinkHandler: ObservableObject {
    @Published public var pendingDeepLink: DeepLink?
    
    public init() {
        setupNotificationObserver()
    }
    
    public func handle(url: URL) {
        guard let components = URLComponents(url: url, resolvingAgainstBaseURL: true) else {
            return
        }
        
        // swiftconnect://profile/123
        // swiftconnect://post/456
        // swiftconnect://conversation/789
        
        switch components.host {
        case "profile":
            let id = components.path.trimmingCharacters(in: .init(charactersIn: "/"))
            pendingDeepLink = .profile(id: id)
            
        case "post":
            let id = components.path.trimmingCharacters(in: .init(charactersIn: "/"))
            pendingDeepLink = .post(id: id)
            
        case "conversation":
            let id = components.path.trimmingCharacters(in: .init(charactersIn: "/"))
            pendingDeepLink = .conversation(id: id)
            
        case "search":
            let query = components.queryItems?.first(where: { $0.name == "q" })?.value ?? ""
            pendingDeepLink = .search(query: query)
            
        default:
            break
        }
    }
    
    private func setupNotificationObserver() {
        NotificationCenter.default.addObserver(
            forName: .notificationActionReceived,
            object: nil,
            queue: .main
        ) { [weak self] notification in
            guard let action = notification.object as? NotificationAction else { return }
            Task { @MainActor in
                self?.handleNotificationAction(action)
            }
        }
    }
    
    private func handleNotificationAction(_ action: NotificationAction) {
        switch action {
        case .openConversation(let id):
            pendingDeepLink = .conversation(id: id)
        case .openProfile(let id):
            pendingDeepLink = .profile(id: id)
        case .openPost(let id):
            pendingDeepLink = .post(id: id)
        case .unknown:
            break
        }
    }
}
```

---

## 9. Search Feature

### 9.1 Search ViewModel พร้อม Debounce

```swift
// Packages/Features/Search/Sources/SearchViewModel.swift
import Foundation
import Combine
import Core
import Networking

@MainActor
public final class SearchViewModel: ObservableObject {
    // MARK: - Published State
    @Published public var searchText = ""
    @Published public private(set) var searchResults: SearchResults = .empty
    @Published public private(set) var suggestions: [String] = []
    @Published public private(set) var isSearching = false
    @Published public private(set) var searchHistory: [String] = []
    
    // MARK: - Dependencies
    private let apiClient: APIClient
    private var cancellables = Set<AnyCancellable>()
    private let debounceInterval: TimeInterval = 0.3
    
    // MARK: - UserDefaults key
    private let historyKey = "search_history"
    private let maxHistoryItems = 10
    
    public init(apiClient: APIClient = .shared) {
        self.apiClient = apiClient
        loadSearchHistory()
        setupSearchDebounce()
    }
    
    private func setupSearchDebounce() {
        $searchText
            .debounce(for: .seconds(debounceInterval), scheduler: DispatchQueue.main)
            .removeDuplicates()
            .sink { [weak self] query in
                Task { @MainActor [weak self] in
                    await self?.performSearch(query: query)
                }
            }
            .store(in: &cancellables)
    }
    
    public func performSearch(query: String) async {
        let trimmed = query.trimmingCharacters(in: .whitespaces)
        
        guard !trimmed.isEmpty else {
            searchResults = .empty
            isSearching = false
            return
        }
        
        isSearching = true
        defer { isSearching = false }
        
        do {
            async let usersTask = apiClient.send(SearchUsersRequest(query: trimmed))
            async let postsTask = apiClient.send(SearchPostsRequest(query: trimmed))
            
            let (users, posts) = try await (usersTask, postsTask)
            
            searchResults = SearchResults(
                users: users.data,
                posts: posts.data
            )
        } catch {
            // ไม่ต้องแสดง error สำหรับ search
        }
    }
    
    public func selectResult(query: String) {
        addToHistory(query)
    }
    
    public func clearHistory() {
        searchHistory = []
        UserDefaults.standard.removeObject(forKey: historyKey)
    }
    
    public func removeHistoryItem(_ item: String) {
        searchHistory.removeAll { $0 == item }
        saveSearchHistory()
    }
    
    private func addToHistory(_ query: String) {
        guard !query.isEmpty else { return }
        
        searchHistory.removeAll { $0 == query }
        searchHistory.insert(query, at: 0)
        
        if searchHistory.count > maxHistoryItems {
            searchHistory = Array(searchHistory.prefix(maxHistoryItems))
        }
        
        saveSearchHistory()
    }
    
    private func loadSearchHistory() {
        searchHistory = UserDefaults.standard.stringArray(forKey: historyKey) ?? []
    }
    
    private func saveSearchHistory() {
        UserDefaults.standard.set(searchHistory, forKey: historyKey)
    }
}

public struct SearchResults {
    public let users: [User]
    public let posts: [Post]
    
    public static let empty = SearchResults(users: [], posts: [])
    
    public var isEmpty: Bool {
        users.isEmpty && posts.isEmpty
    }
}
```

---

## 10. SwiftUI App Implementation

### 10.1 App Entry Point

```swift
// App/SwiftConnectApp.swift
import SwiftUI
import Auth
import FeedFeature
import ProfileFeature
import MessagingFeature
import SearchFeature

@main
struct SwiftConnectApp: App {
    @StateObject private var authManager = AppleSignInManager()
    @StateObject private var deepLinkHandler = DeepLinkHandler()
    @StateObject private var notificationManager = NotificationManager.shared
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(authManager)
                .environmentObject(deepLinkHandler)
                .environmentObject(notificationManager)
                .onOpenURL { url in
                    deepLinkHandler.handle(url: url)
                }
        }
    }
}

// App/ContentView.swift
struct ContentView: View {
    @EnvironmentObject var authManager: AppleSignInManager
    
    var body: some View {
        Group {
            if authManager.isSignedIn {
                MainTabView()
            } else {
                SignInView()
            }
        }
        .animation(.easeInOut, value: authManager.isSignedIn)
    }
}

// App/MainTabView.swift
struct MainTabView: View {
    @EnvironmentObject var deepLinkHandler: DeepLinkHandler
    @State private var selectedTab: Tab = .feed
    
    enum Tab: Hashable {
        case feed, search, notifications, messaging, profile
    }
    
    var body: some View {
        TabView(selection: $selectedTab) {
            NavigationStack {
                FeedView()
            }
            .tabItem {
                Label("Feed", systemImage: "house.fill")
            }
            .tag(Tab.feed)
            
            NavigationStack {
                SearchView()
            }
            .tabItem {
                Label("Search", systemImage: "magnifyingglass")
            }
            .tag(Tab.search)
            
            NavigationStack {
                NotificationsView()
            }
            .tabItem {
                Label("Notifications", systemImage: "bell.fill")
            }
            .badge(3)
            .tag(Tab.notifications)
            
            NavigationStack {
                ConversationListView()
            }
            .tabItem {
                Label("Messages", systemImage: "message.fill")
            }
            .tag(Tab.messaging)
            
            NavigationStack {
                ProfileView(userId: "current")
            }
            .tabItem {
                Label("Profile", systemImage: "person.fill")
            }
            .tag(Tab.profile)
        }
        .onChange(of: deepLinkHandler.pendingDeepLink) { deepLink in
            handleDeepLink(deepLink)
        }
    }
    
    private func handleDeepLink(_ deepLink: DeepLink?) {
        guard let deepLink = deepLink else { return }
        
        switch deepLink {
        case .profile:
            selectedTab = .profile
        case .post:
            selectedTab = .feed
        case .conversation:
            selectedTab = .messaging
        case .search:
            selectedTab = .search
        }
    }
}
```

### 10.2 Sign In View

```swift
// App/Auth/SignInView.swift
import SwiftUI
import AuthenticationServices
import Auth

struct SignInView: View {
    @EnvironmentObject var authManager: AppleSignInManager
    @State private var isLoading = false
    @State private var error: AuthError?
    
    var body: some View {
        VStack(spacing: 40) {
            Spacer()
            
            // Logo
            VStack(spacing: 16) {
                Image(systemName: "network")
                    .font(.system(size: 80))
                    .foregroundStyle(
                        LinearGradient(
                            colors: [.blue, .purple],
                            startPoint: .topLeading,
                            endPoint: .bottomTrailing
                        )
                    )
                
                Text("SwiftConnect")
                    .font(.largeTitle)
                    .fontWeight(.bold)
                
                Text("เชื่อมต่อกับ Professional Network ของคุณ")
                    .font(.subheadline)
                    .foregroundStyle(.secondary)
                    .multilineTextAlignment(.center)
            }
            
            Spacer()
            
            // Sign in button
            VStack(spacing: 16) {
                if isLoading {
                    ProgressView()
                        .frame(height: 50)
                } else {
                    SignInWithAppleButton { request in
                        request.requestedScopes = [.fullName, .email]
                    } onCompletion: { _ in
                        // ถูกจัดการโดย AppleSignInManager
                    }
                    .frame(height: 50)
                    .cornerRadius(10)
                    
                    Button("ดำเนินการต่อโดยไม่ Login") {
                        // Guest mode
                    }
                    .foregroundStyle(.secondary)
                    .font(.footnote)
                }
            }
            .padding(.horizontal, 24)
            
            Text("การสมัครสมาชิกถือว่าคุณยอมรับ Terms of Service และ Privacy Policy ของเรา")
                .font(.caption2)
                .foregroundStyle(.tertiary)
                .multilineTextAlignment(.center)
                .padding(.horizontal)
                .padding(.bottom, 20)
        }
        .alert("เข้าสู่ระบบล้มเหลว", isPresented: Binding(
            get: { error != nil },
            set: { if !$0 { error = nil } }
        )) {
            Button("ตกลง") { error = nil }
        } message: {
            Text(error?.localizedDescription ?? "")
        }
        .task {
            // ลอง Sign in
            do {
                isLoading = true
                _ = try await authManager.signIn()
            } catch let authError as AuthError {
                if authError != .cancelled {
                    error = authError
                }
            } catch {
                self.error = .underlying(error)
            }
            isLoading = false
        }
    }
}

extension AuthError: Equatable {
    public static func == (lhs: AuthError, rhs: AuthError) -> Bool {
        switch (lhs, rhs) {
        case (.invalidCredential, .invalidCredential): return true
        case (.cancelled, .cancelled): return true
        case (.tokenExpired, .tokenExpired): return true
        case (.refreshFailed, .refreshFailed): return true
        default: return false
        }
    }
}
```

---

## 11. Testing Strategy

### 11.1 Unit Tests

```swift
// Packages/Features/Feed/Tests/FeedViewModelTests.swift
import XCTest
import Combine
@testable import FeedFeature
@testable import Core

final class FeedViewModelTests: XCTestCase {
    var viewModel: FeedViewModel!
    var mockAPIClient: MockAPIClient!
    var cancellables = Set<AnyCancellable>()
    
    override func setUp() {
        super.setUp()
        mockAPIClient = MockAPIClient()
        viewModel = FeedViewModel(apiClient: mockAPIClient)
    }
    
    override func tearDown() {
        cancellables.removeAll()
        super.tearDown()
    }
    
    @MainActor
    func testLoadInitialFeed_Success() async throws {
        // Arrange
        let mockPosts = (1...5).map { i in
            Post(
                id: "\(i)",
                authorId: "user1",
                content: "Post \(i)"
            )
        }
        mockAPIClient.mockFeedResponse = PaginatedResponse(
            data: mockPosts,
            totalCount: 5,
            hasNextPage: false,
            nextPage: nil
        )
        
        // Act
        await viewModel.loadInitialFeed()
        
        // Assert
        XCTAssertEqual(viewModel.posts.count, 5)
        XCTAssertFalse(viewModel.isLoadingInitial)
        XCTAssertNil(viewModel.error)
    }
    
    @MainActor
    func testLoadInitialFeed_NetworkError() async {
        // Arrange
        mockAPIClient.shouldFail = true
        mockAPIClient.mockError = APIError.offline
        
        // Act
        await viewModel.loadInitialFeed()
        
        // Assert
        XCTAssertTrue(viewModel.posts.isEmpty)
        XCTAssertNotNil(viewModel.error)
        XCTAssertFalse(viewModel.isLoadingInitial)
    }
    
    @MainActor
    func testToggleLike_OptimisticUpdate() async {
        // Arrange
        let post = Post(id: "1", authorId: "user1", content: "Test", likeCount: 10, isLiked: false)
        viewModel.posts = [post]
        
        mockAPIClient.mockLikeResponse = LikeResponse(liked: true, likeCount: 11)
        
        // Act
        await viewModel.toggleLike(post: post)
        
        // Assert - ตรวจสอบ state หลัง like
        XCTAssertTrue(viewModel.posts[0].isLiked)
        XCTAssertEqual(viewModel.posts[0].likeCount, 11)
    }
    
    @MainActor
    func testToggleLike_Rollback_OnError() async {
        // Arrange
        let post = Post(id: "1", authorId: "user1", content: "Test", likeCount: 10, isLiked: false)
        viewModel.posts = [post]
        
        mockAPIClient.shouldFail = true
        mockAPIClient.mockError = APIError.networkError(URLError(.notConnectedToInternet))
        
        // Act
        await viewModel.toggleLike(post: post)
        
        // Assert - ควร rollback กลับ
        XCTAssertFalse(viewModel.posts[0].isLiked)
        XCTAssertEqual(viewModel.posts[0].likeCount, 10)
    }
    
    @MainActor
    func testLoadMoreIfNeeded_TriggersNextPage() async {
        // Arrange - load first page
        let firstPagePosts = (1...20).map { i in
            Post(id: "\(i)", authorId: "user1", content: "Post \(i)")
        }
        mockAPIClient.mockFeedResponse = PaginatedResponse(
            data: firstPagePosts,
            totalCount: 40,
            hasNextPage: true,
            nextPage: 2
        )
        
        await viewModel.loadInitialFeed()
        
        // Setup second page
        let secondPagePosts = (21...40).map { i in
            Post(id: "\(i)", authorId: "user1", content: "Post \(i)")
        }
        mockAPIClient.mockFeedResponse = PaginatedResponse(
            data: secondPagePosts,
            totalCount: 40,
            hasNextPage: false,
            nextPage: nil
        )
        
        // Act - trigger load more กับ post สุดท้าย
        let lastPost = viewModel.posts.last!
        await viewModel.loadMoreIfNeeded(currentPost: lastPost)
        
        // Assert
        XCTAssertEqual(viewModel.posts.count, 40)
        XCTAssertFalse(viewModel.hasNextPage)
    }
}

// Mock API Client
class MockAPIClient: APIClient {
    var shouldFail = false
    var mockError: Error = APIError.networkError(URLError(.notConnectedToInternet))
    var mockFeedResponse: PaginatedResponse<Post>?
    var mockLikeResponse: LikeResponse?
    
    override func send<Request: APIRequest>(_ request: Request) async throws -> Request.Response {
        if shouldFail { throw mockError }
        
        if let feed = mockFeedResponse as? Request.Response {
            return feed
        }
        if let like = mockLikeResponse as? Request.Response {
            return like
        }
        
        throw APIError.notFound
    }
}
```

### 11.2 Integration Tests สำหรับ Networking

```swift
// Packages/Networking/Tests/APIClientIntegrationTests.swift
import XCTest
@testable import Networking

final class APIClientIntegrationTests: XCTestCase {
    var apiClient: APIClient!
    var mockSession: MockURLSession!
    
    override func setUp() {
        super.setUp()
        mockSession = MockURLSession()
        apiClient = APIClient(
            baseURL: URL(string: "https://api.test.swiftconnect.com")!,
            session: mockSession
        )
    }
    
    func testSuccessfulRequest() async throws {
        // Arrange
        let json = """
        {"data": [], "totalCount": 0, "hasNextPage": false, "nextPage": null}
        """.data(using: .utf8)!
        
        mockSession.mockData = json
        mockSession.mockResponse = HTTPURLResponse(
            url: URL(string: "https://api.test.swiftconnect.com/feed")!,
            statusCode: 200,
            httpVersion: nil,
            headerFields: nil
        )!
        
        // Act
        let result = try await apiClient.send(GetFeedRequest())
        
        // Assert
        XCTAssertEqual(result.totalCount, 0)
        XCTAssertFalse(result.hasNextPage)
    }
    
    func testUnauthorizedRetry() async {
        // First call returns 401, second returns 200
        var callCount = 0
        mockSession.customHandler = { request in
            callCount += 1
            if callCount == 1 {
                return (Data(), HTTPURLResponse(
                    url: request.url!,
                    statusCode: 401,
                    httpVersion: nil,
                    headerFields: nil
                )!)
            } else {
                let data = """
                {"data": [], "totalCount": 0, "hasNextPage": false, "nextPage": null}
                """.data(using: .utf8)!
                return (data, HTTPURLResponse(
                    url: request.url!,
                    statusCode: 200,
                    httpVersion: nil,
                    headerFields: nil
                )!)
            }
        }
        
        // Act
        do {
            _ = try await apiClient.send(GetFeedRequest())
            // Should retry and succeed
            XCTAssertEqual(callCount, 2)
        } catch {
            XCTFail("ควร retry สำเร็จ: \(error)")
        }
    }
}

class MockURLSession: URLSession {
    var mockData: Data?
    var mockResponse: URLResponse?
    var mockError: Error?
    var customHandler: ((URLRequest) -> (Data, URLResponse))?
    
    override func data(for request: URLRequest) async throws -> (Data, URLResponse) {
        if let error = mockError { throw error }
        
        if let handler = customHandler {
            return handler(request)
        }
        
        return (
            mockData ?? Data(),
            mockResponse ?? URLResponse()
        )
    }
}
```

### 11.3 Snapshot Tests

```swift
// Tests/SnapshotTests/FeedSnapshotTests.swift
import XCTest
import SwiftUI
import SnapshotTesting
@testable import FeedFeature
@testable import Core

final class FeedSnapshotTests: XCTestCase {
    func testPostCardView_WithContent() {
        let post = Post(
            id: "1",
            authorId: "user1",
            author: User(
                id: "user1",
                firstName: "John",
                lastName: "Doe",
                email: "john@example.com",
                headline: "iOS Developer at Apple"
            ),
            content: "นี่คือโพสต์ทดสอบ! 🚀",
            likeCount: 42,
            commentCount: 7,
            isLiked: true
        )
        
        let view = PostCardView(post: post)
            .frame(width: 390)
        
        assertSnapshot(
            matching: view,
            as: .image(layout: .fixed(width: 390, height: 200))
        )
    }
    
    func testPostCardSkeleton() {
        let view = PostCardSkeleton()
            .frame(width: 390)
        
        assertSnapshot(
            matching: view,
            as: .image(layout: .fixed(width: 390, height: 150))
        )
    }
}
```

---

## 12. CI/CD Setup

### 12.1 GitHub Actions Workflow

```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  XCODE_VERSION: '15.2'
  IOS_SIMULATOR: 'iPhone 15 Pro'
  IOS_VERSION: '17.2'

jobs:
  test:
    name: Run Tests
    runs-on: macos-14
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Select Xcode
        run: sudo xcode-select -s /Applications/Xcode_${{ env.XCODE_VERSION }}.app
      
      - name: Cache SPM packages
        uses: actions/cache@v3
        with:
          path: .build
          key: ${{ runner.os }}-spm-${{ hashFiles('**/Package.resolved') }}
          restore-keys: |
            ${{ runner.os }}-spm-
      
      - name: Run Unit Tests
        run: |
          xcodebuild test \
            -project SwiftConnect.xcodeproj \
            -scheme SwiftConnect \
            -destination "platform=iOS Simulator,name=${{ env.IOS_SIMULATOR }},OS=${{ env.IOS_VERSION }}" \
            -resultBundlePath TestResults.xcresult \
            | xcpretty
      
      - name: Upload Test Results
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: test-results
          path: TestResults.xcresult
      
      - name: Code Coverage
        run: |
          xcrun xccov view --report --json TestResults.xcresult > coverage.json
          cat coverage.json | python3 -c "
          import json, sys
          data = json.load(sys.stdin)
          coverage = data.get('lineCoverage', 0) * 100
          print(f'Code Coverage: {coverage:.1f}%')
          if coverage < 70:
              sys.exit(1)
          "
  
  lint:
    name: SwiftLint
    runs-on: macos-14
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Run SwiftLint
        run: |
          brew install swiftlint
          swiftlint lint --reporter github-actions-logging
  
  build:
    name: Build Release
    needs: [test, lint]
    runs-on: macos-14
    if: github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Select Xcode
        run: sudo xcode-select -s /Applications/Xcode_${{ env.XCODE_VERSION }}.app
      
      - name: Import Certificates
        env:
          P12_CERTIFICATE: ${{ secrets.P12_CERTIFICATE }}
          P12_PASSWORD: ${{ secrets.P12_PASSWORD }}
          PROVISION_PROFILE: ${{ secrets.PROVISION_PROFILE }}
          KEYCHAIN_PASSWORD: ${{ secrets.KEYCHAIN_PASSWORD }}
        run: |
          # สร้าง temporary keychain
          security create-keychain -p "$KEYCHAIN_PASSWORD" build.keychain
          security default-keychain -s build.keychain
          security unlock-keychain -p "$KEYCHAIN_PASSWORD" build.keychain
          
          # Import certificate
          echo "$P12_CERTIFICATE" | base64 --decode > certificate.p12
          security import certificate.p12 -k build.keychain -P "$P12_PASSWORD" -T /usr/bin/codesign
          
          # Import provisioning profile
          echo "$PROVISION_PROFILE" | base64 --decode > profile.mobileprovision
          mkdir -p ~/Library/MobileDevice/Provisioning\ Profiles
          cp profile.mobileprovision ~/Library/MobileDevice/Provisioning\ Profiles/
      
      - name: Build Archive
        run: |
          xcodebuild archive \
            -project SwiftConnect.xcodeproj \
            -scheme SwiftConnect \
            -configuration Release \
            -archivePath SwiftConnect.xcarchive \
            CODE_SIGNING_REQUIRED=YES \
            | xcpretty
      
      - name: Export IPA
        run: |
          xcodebuild -exportArchive \
            -archivePath SwiftConnect.xcarchive \
            -exportPath export \
            -exportOptionsPlist ExportOptions.plist \
            | xcpretty
      
      - name: Upload to TestFlight
        env:
          APP_STORE_CONNECT_KEY_ID: ${{ secrets.APP_STORE_CONNECT_KEY_ID }}
          APP_STORE_CONNECT_ISSUER_ID: ${{ secrets.APP_STORE_CONNECT_ISSUER_ID }}
          APP_STORE_CONNECT_KEY: ${{ secrets.APP_STORE_CONNECT_KEY }}
        run: |
          xcrun altool --upload-app \
            --type ios \
            --file export/SwiftConnect.ipa \
            --apiKey "$APP_STORE_CONNECT_KEY_ID" \
            --apiIssuer "$APP_STORE_CONNECT_ISSUER_ID"
```

### 12.2 Fastlane Setup

```ruby
# fastlane/Fastfile
default_platform(:ios)

platform :ios do
  desc "Run tests"
  lane :test do
    run_tests(
      project: "SwiftConnect.xcodeproj",
      scheme: "SwiftConnect",
      devices: ["iPhone 15 Pro"],
      reset_simulator: true,
      code_coverage: true
    )
  end
  
  desc "Upload to TestFlight"
  lane :beta do
    ensure_git_status_clean
    
    increment_build_number(
      build_number: latest_testflight_build_number + 1
    )
    
    build_app(
      project: "SwiftConnect.xcodeproj",
      scheme: "SwiftConnect",
      configuration: "Release",
      export_method: "app-store"
    )
    
    upload_to_testflight(
      skip_waiting_for_build_processing: true
    )
    
    clean_build_artifacts
    
    slack(
      message: "SwiftConnect beta upload successful! 🚀",
      channel: "#ios-deploys"
    )
  end
  
  desc "Release to App Store"
  lane :release do
    test
    beta
    
    deliver(
      submit_for_review: true,
      automatic_release: false,
      phased_release: true
    )
  end
end
```

---

## 13. Performance Optimizations

### 13.1 Image Loading Cache

```swift
// Packages/Core/Sources/Cache/ImageCache.swift
import UIKit
import Foundation

public final class ImageCache: @unchecked Sendable {
    public static let shared = ImageCache()
    
    private let memoryCache = NSCache<NSURL, UIImage>()
    private let diskCacheURL: URL
    private let queue = DispatchQueue(
        label: "ImageCache",
        qos: .utility,
        attributes: .concurrent
    )
    
    private init() {
        // ตั้งค่า memory cache
        memoryCache.countLimit = 100
        memoryCache.totalCostLimit = 50 * 1024 * 1024 // 50MB
        
        // ตั้งค่า disk cache
        diskCacheURL = FileManager.default.urls(
            for: .cachesDirectory,
            in: .userDomainMask
        )[0].appendingPathComponent("ImageCache")
        
        try? FileManager.default.createDirectory(
            at: diskCacheURL,
            withIntermediateDirectories: true
        )
    }
    
    public func image(for url: URL) async -> UIImage? {
        // 1. ตรวจสอบ memory cache
        if let cached = memoryCache.object(forKey: url as NSURL) {
            return cached
        }
        
        // 2. ตรวจสอบ disk cache
        let diskPath = diskCacheURL.appendingPathComponent(
            url.absoluteString.data(using: .utf8)!
                .base64EncodedString()
                .replacingOccurrences(of: "/", with: "_")
        )
        
        if let data = try? Data(contentsOf: diskPath),
           let image = UIImage(data: data) {
            // บันทึกลง memory cache
            memoryCache.setObject(image, forKey: url as NSURL)
            return image
        }
        
        // 3. Download จาก network
        do {
            let (data, _) = try await URLSession.shared.data(from: url)
            guard let image = UIImage(data: data) else { return nil }
            
            // Cache in memory
            memoryCache.setObject(image, forKey: url as NSURL)
            
            // Cache on disk
            queue.async(flags: .barrier) {
                try? data.write(to: diskPath)
            }
            
            return image
        } catch {
            return nil
        }
    }
    
    public func prefetch(urls: [URL]) {
        for url in urls {
            Task {
                _ = await image(for: url)
            }
        }
    }
    
    public func clearMemoryCache() {
        memoryCache.removeAllObjects()
    }
    
    public func clearDiskCache() throws {
        try FileManager.default.removeItem(at: diskCacheURL)
        try FileManager.default.createDirectory(
            at: diskCacheURL,
            withIntermediateDirectories: true
        )
    }
}

// Cached Async Image View
public struct CachedAsyncImage<Content: View, Placeholder: View>: View {
    let url: URL?
    let cache: ImageCache
    let content: (Image) -> Content
    let placeholder: () -> Placeholder
    
    @State private var image: UIImage?
    @State private var isLoading = false
    
    public init(
        url: URL?,
        cache: ImageCache = .shared,
        @ViewBuilder content: @escaping (Image) -> Content,
        @ViewBuilder placeholder: @escaping () -> Placeholder
    ) {
        self.url = url
        self.cache = cache
        self.content = content
        self.placeholder = placeholder
    }
    
    public var body: some View {
        Group {
            if let image = image {
                content(Image(uiImage: image))
            } else {
                placeholder()
                    .onAppear { loadImage() }
            }
        }
    }
    
    private func loadImage() {
        guard let url = url, !isLoading else { return }
        
        isLoading = true
        Task {
            let loadedImage = await cache.image(for: url)
            await MainActor.run {
                self.image = loadedImage
                self.isLoading = false
            }
        }
    }
}
```

### 13.2 Background Refresh

```swift
// App/BackgroundRefresh/BackgroundRefreshManager.swift
import BackgroundTasks
import Foundation
import Networking

final class BackgroundRefreshManager {
    static let shared = BackgroundRefreshManager()
    
    private let refreshTaskIdentifier = "com.swiftconnect.refresh"
    
    func register() {
        BGTaskScheduler.shared.register(
            forTaskWithIdentifier: refreshTaskIdentifier,
            using: nil
        ) { task in
            self.handleRefreshTask(task as! BGAppRefreshTask)
        }
    }
    
    func scheduleRefresh() {
        let request = BGAppRefreshTaskRequest(identifier: refreshTaskIdentifier)
        request.earliestBeginDate = Date(timeIntervalSinceNow: 15 * 60) // 15 minutes
        
        try? BGTaskScheduler.shared.submit(request)
    }
    
    private func handleRefreshTask(_ task: BGAppRefreshTask) {
        // Schedule ครั้งถัดไป
        scheduleRefresh()
        
        let refreshTask = Task {
            do {
                // Fetch new data
                async let feedTask = refreshFeed()
                async let notificationsTask = refreshNotifications()
                
                let _ = try await (feedTask, notificationsTask)
                task.setTaskCompleted(success: true)
            } catch {
                task.setTaskCompleted(success: false)
            }
        }
        
        task.expirationHandler = {
            refreshTask.cancel()
        }
    }
    
    private func refreshFeed() async throws {
        _ = try await APIClient.shared.send(GetFeedRequest(page: 1, limit: 10))
    }
    
    private func refreshNotifications() async throws {
        _ = try await APIClient.shared.send(GetNotificationsRequest())
    }
}
```

---

## 14. App Store Preparation

### 14.1 Privacy Manifest

```xml
<!-- PrivacyInfo.xcprivacy -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
    "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>NSPrivacyTracking</key>
    <false/>
    
    <key>NSPrivacyTrackingDomains</key>
    <array/>
    
    <key>NSPrivacyCollectedDataTypes</key>
    <array>
        <!-- Name -->
        <dict>
            <key>NSPrivacyCollectedDataType</key>
            <string>NSPrivacyCollectedDataTypeName</string>
            <key>NSPrivacyCollectedDataTypeLinked</key>
            <true/>
            <key>NSPrivacyCollectedDataTypeTracking</key>
            <false/>
            <key>NSPrivacyCollectedDataTypePurposes</key>
            <array>
                <string>NSPrivacyCollectedDataTypePurposeAppFunctionality</string>
            </array>
        </dict>
        
        <!-- Email Address -->
        <dict>
            <key>NSPrivacyCollectedDataType</key>
            <string>NSPrivacyCollectedDataTypeEmailAddress</string>
            <key>NSPrivacyCollectedDataTypeLinked</key>
            <true/>
            <key>NSPrivacyCollectedDataTypeTracking</key>
            <false/>
            <key>NSPrivacyCollectedDataTypePurposes</key>
            <array>
                <string>NSPrivacyCollectedDataTypePurposeAppFunctionality</string>
                <string>NSPrivacyCollectedDataTypePurposeAnalytics</string>
            </array>
        </dict>
        
        <!-- Photos or Videos -->
        <dict>
            <key>NSPrivacyCollectedDataType</key>
            <string>NSPrivacyCollectedDataTypePhotosOrVideos</string>
            <key>NSPrivacyCollectedDataTypeLinked</key>
            <false/>
            <key>NSPrivacyCollectedDataTypeTracking</key>
            <false/>
            <key>NSPrivacyCollectedDataTypePurposes</key>
            <array>
                <string>NSPrivacyCollectedDataTypePurposeAppFunctionality</string>
            </array>
        </dict>
    </array>
    
    <key>NSPrivacyAccessedAPITypes</key>
    <array>
        <!-- UserDefaults -->
        <dict>
            <key>NSPrivacyAccessedAPIType</key>
            <string>NSPrivacyAccessedAPICategoryUserDefaults</string>
            <key>NSPrivacyAccessedAPITypeReasons</key>
            <array>
                <string>CA92.1</string>
            </array>
        </dict>
        
        <!-- File Timestamp -->
        <dict>
            <key>NSPrivacyAccessedAPIType</key>
            <string>NSPrivacyAccessedAPICategoryFileTimestamp</string>
            <key>NSPrivacyAccessedAPITypeReasons</key>
            <array>
                <string>C617.1</string>
            </array>
        </dict>
    </array>
</dict>
</plist>
```

### 14.2 App Store Connect Metadata

```swift
// fastlane/metadata/th/description.txt
SwiftConnect — แอปพลิเคชัน Professional Networking สำหรับนักพัฒนา iOS

เชื่อมต่อกับ Professional Network ของคุณ

FEATURES:
• สร้าง Professional Profile พร้อมรูปภาพและ headline
• โพสต์ข่าวสาร บทความ และโค้ด
• Like และ Comment บน posts
• ส่ง Connection Request และสร้าง Network
• Real-time Messaging
• Push Notifications
• ค้นหา Professionals และ Posts

PRIVACY:
• Sign in with Apple — ไม่มีการเก็บ password
• ไม่มีการ Track ข้ามแอพ
• ข้อมูลเข้ารหัสด้วย Keychain

ดาวน์โหลดฟรี — ไม่มี In-App Purchase
```

### 14.3 App Privacy Labels Configuration

```swift
// App Store Connect Privacy Labels (App Privacy):

// Data Used to Track You: None

// Data Linked to You:
// - Name (App Functionality)
// - Email Address (App Functionality)
// - Photos or Videos (App Functionality, optional)
// - Messages (App Functionality)
// - User Content - Other User Content (App Functionality)

// Data Not Linked to You:
// - Crash Data (App Functionality)
// - Performance Data (App Functionality)
```

---

## สรุป Capstone Project

### สิ่งที่ได้เรียนรู้จาก Capstone

1. **Modular Architecture** — แยก code เป็น independent modules ด้วย SPM ทำให้ maintainable และ testable

2. **Authentication Flow** — Sign in with Apple + JWT + Keychain storage สำหรับ secure session management

3. **Generic Networking Layer** — Type-safe API client ที่จัดการ retry, token refresh, และ error handling

4. **Real-time Features** — WebSocket management พร้อม reconnection logic

5. **Optimistic UI Updates** — ทำให้ app รู้สึก responsive โดย update UI ก่อน server response

6. **Infinite Scroll** — Pagination ที่ efficient พร้อม deduplication logic

7. **Offline Support** — Core Data persistence สำหรับ messages, NetworkMonitor สำหรับ detect connectivity

8. **Performance** — NSCache + disk cache สำหรับ images, background refresh

9. **Testing Strategy** — Unit tests, integration tests, snapshot tests ครบ

10. **CI/CD** — GitHub Actions + Fastlane สำหรับ automated deployment ไป TestFlight

### Production Checklist

```
✅ Sign in with Apple implementation
✅ Keychain storage (ไม่ใช้ UserDefaults สำหรับ tokens)
✅ Certificate Pinning (ป้องกัน MITM)
✅ Privacy Manifest (.xcprivacy)
✅ App Privacy Labels บน App Store Connect
✅ Crash reporting (Crashlytics/Sentry)
✅ Analytics (ไม่ Track โดยไม่ consent)
✅ Accessibility (VoiceOver, Dynamic Type)
✅ Localization
✅ Dark Mode support
✅ iPad support (Universal app)
✅ Performance profiling (Instruments)
✅ Memory leak checks (Leaks tool)
✅ TestFlight beta testing
✅ App Store review guidelines compliance
```

---

## แบบฝึกหัด Capstone

### แบบฝึกหัดที่ 1: เพิ่ม Connections Feature

สร้าง `ConnectionsFeature` module ที่มี:
- ดู connections ของตัวเอง
- ส่ง connection request
- รับ/ปฏิเสธ requests
- ดู mutual connections

```swift
// เฉลยโครงสร้าง
// Packages/Features/Connections/Sources/

// ConnectionsViewModel.swift
@MainActor
final class ConnectionsViewModel: ObservableObject {
    @Published private(set) var connections: [User] = []
    @Published private(set) var pendingRequests: [ConnectionRequest] = []
    @Published private(set) var isLoading = false
    
    private let apiClient: APIClient
    
    init(apiClient: APIClient = .shared) {
        self.apiClient = apiClient
    }
    
    func loadConnections() async {
        isLoading = true
        defer { isLoading = false }
        
        do {
            async let connectionsTask = apiClient.send(GetConnectionsRequest())
            async let pendingTask = apiClient.send(GetPendingRequestsRequest())
            let (c, p) = try await (connectionsTask, pendingTask)
            connections = c.data
            pendingRequests = p.data
        } catch {
            // handle error
        }
    }
    
    func sendConnectionRequest(to userId: String) async -> Bool {
        do {
            _ = try await apiClient.send(SendConnectionRequest(userId: userId))
            return true
        } catch {
            return false
        }
    }
    
    func acceptRequest(_ requestId: String) async {
        guard let index = pendingRequests.firstIndex(where: { $0.id == requestId }) else { return }
        
        // Optimistic update
        let request = pendingRequests.remove(at: index)
        
        do {
            _ = try await apiClient.send(AcceptConnectionRequest(requestId: requestId))
            // Add to connections
            if let user = request.sender {
                connections.append(user)
            }
        } catch {
            // Rollback
            pendingRequests.insert(request, at: index)
        }
    }
}

struct ConnectionRequest: Identifiable, Codable {
    let id: String
    let sender: User?
    let createdAt: Date
}
```

### แบบฝึกหัดที่ 2: เพิ่ม Story Feature

สร้าง Stories ที่หายไปใน 24 ชั่วโมง คล้าย Instagram Stories:

```swift
// Story Model
struct Story: Identifiable, Codable {
    let id: String
    let userId: String
    var user: User?
    let mediaURL: URL
    let type: MediaType
    let expiresAt: Date
    let createdAt: Date
    
    var isExpired: Bool {
        expiresAt < Date()
    }
    
    enum MediaType: String, Codable {
        case image, video
    }
}

// StoryViewModel
@MainActor
final class StoryViewModel: ObservableObject {
    @Published private(set) var storyGroups: [StoryGroup] = []
    
    struct StoryGroup: Identifiable {
        let id: String
        let user: User
        var stories: [Story]
        var hasUnviewed: Bool
    }
    
    func loadStories() async {
        // TODO: Implement
    }
    
    func markAsViewed(_ story: Story) async {
        // TODO: Implement
    }
    
    func createStory(image: UIImage) async -> Bool {
        // TODO: Implement
        return false
    }
}
```

### แบบฝึกหัดที่ 3: เพิ่ม Analytics Module

สร้าง analytics module ที่ track user behavior สำหรับ improve UX:

```swift
// Analytics Event
enum AnalyticsEvent {
    case screenViewed(name: String)
    case buttonTapped(name: String, context: [String: String])
    case postLiked(postId: String)
    case searchPerformed(query: String, resultsCount: Int)
    case sessionStarted
    case sessionEnded(duration: TimeInterval)
}

// Analytics Protocol
protocol AnalyticsProvider {
    func track(_ event: AnalyticsEvent)
    func setUserId(_ id: String)
    func reset()
}

// Composite Analytics (หลาย providers)
final class AnalyticsManager: AnalyticsProvider {
    private let providers: [AnalyticsProvider]
    
    init(providers: [AnalyticsProvider]) {
        self.providers = providers
    }
    
    func track(_ event: AnalyticsEvent) {
        providers.forEach { $0.track(event) }
    }
    
    func setUserId(_ id: String) {
        providers.forEach { $0.setUserId(id) }
    }
    
    func reset() {
        providers.forEach { $0.reset() }
    }
}

// Console Analytics (สำหรับ development)
final class ConsoleAnalyticsProvider: AnalyticsProvider {
    func track(_ event: AnalyticsEvent) {
        #if DEBUG
        print("[Analytics] \(event)")
        #endif
    }
    
    func setUserId(_ id: String) {
        #if DEBUG
        print("[Analytics] User: \(id)")
        #endif
    }
    
    func reset() {
        #if DEBUG
        print("[Analytics] Reset")
        #endif
    }
}
```

---

## สรุปท้ายบท

บทนี้เป็น Capstone Project ที่รวบรวมทุกทักษะที่เรียนมาตลอด course โดยสร้าง SwiftConnect — production-quality social networking app ที่ครอบคลุม:

1. **Clean Architecture + MVVM + Modular SPM** — โครงสร้างที่ scalable และ maintainable
2. **Authentication** — Sign in with Apple + JWT + Keychain
3. **Networking** — Generic async/await API client พร้อม retry, caching
4. **Real-time** — WebSocket messaging พร้อม reconnection
5. **Persistence** — Core Data สำหรับ offline support
6. **Push Notifications + Deep Links** — Complete notification routing
7. **Performance** — Image caching, background refresh, optimistic updates
8. **Testing** — Unit, integration, snapshot tests
9. **CI/CD** — GitHub Actions + Fastlane + TestFlight
10. **App Store Preparation** — Privacy manifest, App privacy labels

ขอให้โชคดีในการสร้าง iOS apps ที่ยอดเยี่ยม! 🚀

---

*จบ Part 90: Capstone Project — SwiftConnect Social App*

*จบ Swift Programming Course — ขอบคุณที่เรียนจนจบ!*
