# Part 48: Advanced Networking

## บทนำ

การทำ networking ใน iOS/macOS applications มีความซับซ้อนมากกว่าการส่ง HTTP request ธรรมดา ในระบบ production จำเป็นต้องคำนึงถึง authentication, error handling, retry logic, offline support และอื่นๆ อีกมากมาย บทนี้จะครอบคลุม patterns และ best practices สำหรับการสร้าง network layer ที่แข็งแกร่งและพร้อมใช้งานจริง

---

## 1. REST API Best Practices

### หลักการพื้นฐาน REST

REST (Representational State Transfer) เป็น architectural style สำหรับ distributed systems โดยมีหลักการสำคัญ:

1. **Stateless** - แต่ละ request มีข้อมูลครบในตัวเอง
2. **Client-Server** - แยก concerns ระหว่าง client และ server
3. **Cacheable** - Responses บอกได้ว่า cache ได้หรือไม่
4. **Uniform Interface** - ใช้ HTTP methods อย่างถูกต้อง
5. **Layered System** - Client ไม่รู้ว่าคุยกับ server จริงหรือ proxy

### HTTP Methods ที่ถูกต้อง

```swift
// Endpoint definitions
enum UserEndpoint {
    case getAllUsers                          // GET /users
    case getUser(id: Int)                    // GET /users/{id}
    case createUser(user: CreateUserRequest) // POST /users
    case updateUser(id: Int, user: UpdateUserRequest) // PUT /users/{id}
    case patchUser(id: Int, fields: [String: Any]) // PATCH /users/{id}
    case deleteUser(id: Int)                 // DELETE /users/{id}
}

// HTTP Method enum
enum HTTPMethod: String {
    case get    = "GET"
    case post   = "POST"
    case put    = "PUT"
    case patch  = "PATCH"
    case delete = "DELETE"
    case head   = "HEAD"
    case options = "OPTIONS"
}
```

### Status Codes ที่ควรรู้จัก

```swift
enum HTTPStatusCode: Int {
    // 2xx Success
    case ok = 200
    case created = 201
    case accepted = 202
    case noContent = 204
    
    // 3xx Redirection
    case notModified = 304
    
    // 4xx Client Errors
    case badRequest = 400
    case unauthorized = 401
    case forbidden = 403
    case notFound = 404
    case methodNotAllowed = 405
    case conflict = 409
    case unprocessableEntity = 422
    case tooManyRequests = 429
    
    // 5xx Server Errors
    case internalServerError = 500
    case badGateway = 502
    case serviceUnavailable = 503
    case gatewayTimeout = 504
    
    var isSuccess: Bool {
        (200...299).contains(rawValue)
    }
    
    var isClientError: Bool {
        (400...499).contains(rawValue)
    }
    
    var isServerError: Bool {
        (500...599).contains(rawValue)
    }
    
    var shouldRetry: Bool {
        switch self {
        case .tooManyRequests, .serviceUnavailable, .gatewayTimeout:
            return true
        default:
            return false
        }
    }
}
```

### Request/Response Headers

```swift
struct HTTPHeaders {
    // Common request headers
    static let contentType = "Content-Type"
    static let accept = "Accept"
    static let authorization = "Authorization"
    static let userAgent = "User-Agent"
    static let acceptLanguage = "Accept-Language"
    static let acceptEncoding = "Accept-Encoding"
    
    // Common values
    struct ContentType {
        static let json = "application/json"
        static let formURLEncoded = "application/x-www-form-urlencoded"
        static let multipartForm = "multipart/form-data"
        static let plainText = "text/plain"
    }
    
    // Response headers
    static let retryAfter = "Retry-After"
    static let rateLimit = "X-RateLimit-Limit"
    static let rateLimitRemaining = "X-RateLimit-Remaining"
    static let rateLimitReset = "X-RateLimit-Reset"
}
```

---

## 2. API Versioning

### รูปแบบการทำ API Versioning

```swift
// 1. URL Path Versioning (แนะนำ)
// https://api.example.com/v1/users
// https://api.example.com/v2/users

// 2. Query Parameter Versioning
// https://api.example.com/users?version=1
// https://api.example.com/users?api-version=2

// 3. Header Versioning
// Accept: application/vnd.example.v1+json
// API-Version: 2

// 4. Subdomain Versioning
// https://v1.api.example.com/users
// https://v2.api.example.com/users
```

### Implementation สำหรับ URL Path Versioning

```swift
// APIConfiguration.swift

struct APIConfiguration {
    let baseURL: URL
    let version: APIVersion
    let environment: Environment
    
    enum APIVersion: String {
        case v1 = "v1"
        case v2 = "v2"
        case v3 = "v3"
        
        var pathComponent: String { rawValue }
    }
    
    enum Environment {
        case development
        case staging
        case production
        
        var baseURL: URL {
            switch self {
            case .development:
                return URL(string: "https://dev-api.example.com")!
            case .staging:
                return URL(string: "https://staging-api.example.com")!
            case .production:
                return URL(string: "https://api.example.com")!
            }
        }
    }
    
    var versionedBaseURL: URL {
        environment.baseURL.appendingPathComponent(version.pathComponent)
    }
    
    static let current = APIConfiguration(
        baseURL: Environment.production.baseURL,
        version: .v2,
        environment: .production
    )
}

// ตัวอย่างการใช้งาน
let config = APIConfiguration.current
let usersURL = config.versionedBaseURL.appendingPathComponent("users")
// Result: https://api.example.com/v2/users
```

### Feature Flags สำหรับ API Version Migration

```swift
// APIFeatureFlags.swift

struct APIFeatureFlags {
    
    static var useV2GraphQL: Bool {
        return APIConfiguration.current.version >= .v2
    }
    
    static var supportsWebSockets: Bool {
        return APIConfiguration.current.version >= .v3
    }
    
    static var useNewAuthFlow: Bool {
        // Feature flag จาก remote config
        return RemoteConfig.shared.bool(forKey: "use_new_auth_flow")
    }
}
```

---

## 3. Authentication Patterns

### JWT (JSON Web Token)

```swift
// JWTManager.swift

import Foundation
import CryptoKit

struct JWT {
    let header: Header
    let payload: Payload
    let signature: String
    
    struct Header: Codable {
        let alg: String  // Algorithm (e.g., "HS256", "RS256")
        let typ: String  // Type = "JWT"
    }
    
    struct Payload: Codable {
        let sub: String     // Subject (user ID)
        let iat: TimeInterval  // Issued at
        let exp: TimeInterval  // Expiration
        let iss: String?    // Issuer
        let aud: String?    // Audience
        
        // Custom claims
        let role: String?
        let permissions: [String]?
        
        var isExpired: Bool {
            Date().timeIntervalSince1970 >= exp
        }
        
        var expiresIn: TimeInterval {
            exp - Date().timeIntervalSince1970
        }
    }
    
    var rawToken: String {
        "\(encodeHeader()).\(encodePayload()).\(signature)"
    }
    
    private func encodeHeader() -> String {
        let data = try? JSONEncoder().encode(header)
        return data?.base64URLEncoded ?? ""
    }
    
    private func encodePayload() -> String {
        let data = try? JSONEncoder().encode(payload)
        return data?.base64URLEncoded ?? ""
    }
    
    /// Parse JWT token จาก string
    static func parse(_ token: String) throws -> JWT {
        let parts = token.components(separatedBy: ".")
        guard parts.count == 3 else {
            throw JWTError.invalidFormat
        }
        
        let headerData = Data(base64URLEncoded: parts[0]) ?? Data()
        let payloadData = Data(base64URLEncoded: parts[1]) ?? Data()
        
        let decoder = JSONDecoder()
        let header = try decoder.decode(Header.self, from: headerData)
        let payload = try decoder.decode(Payload.self, from: payloadData)
        
        return JWT(header: header, payload: payload, signature: parts[2])
    }
}

enum JWTError: LocalizedError {
    case invalidFormat
    case expired
    case invalidSignature
    
    var errorDescription: String? {
        switch self {
        case .invalidFormat: return "JWT format ไม่ถูกต้อง"
        case .expired: return "Token หมดอายุแล้ว"
        case .invalidSignature: return "Signature ไม่ถูกต้อง"
        }
    }
}

// Extension สำหรับ base64 URL encoding
extension Data {
    var base64URLEncoded: String {
        base64EncodedString()
            .replacingOccurrences(of: "+", with: "-")
            .replacingOccurrences(of: "/", with: "_")
            .replacingOccurrences(of: "=", with: "")
    }
    
    init?(base64URLEncoded string: String) {
        var base64 = string
            .replacingOccurrences(of: "-", with: "+")
            .replacingOccurrences(of: "_", with: "/")
        
        // Add padding
        while base64.count % 4 != 0 {
            base64 += "="
        }
        
        self.init(base64Encoded: base64)
    }
}
```

### OAuth2 Flow

```swift
// OAuth2Manager.swift

import Foundation
import AuthenticationServices

class OAuth2Manager: NSObject {
    
    private let clientID: String
    private let clientSecret: String
    private let redirectURI: String
    private let authorizationURL: URL
    private let tokenURL: URL
    private let scopes: [String]
    
    private var webAuthSession: ASWebAuthenticationSession?
    
    init(
        clientID: String,
        clientSecret: String,
        redirectURI: String,
        authorizationURL: URL,
        tokenURL: URL,
        scopes: [String]
    ) {
        self.clientID = clientID
        self.clientSecret = clientSecret
        self.redirectURI = redirectURI
        self.authorizationURL = authorizationURL
        self.tokenURL = tokenURL
        self.scopes = scopes
    }
    
    /// สร้าง Authorization URL พร้อม PKCE
    func buildAuthorizationURL(state: String, codeVerifier: String) -> URL {
        let codeChallenge = PKCE.generateChallenge(from: codeVerifier)
        
        var components = URLComponents(url: authorizationURL, resolvingAgainstBaseURL: false)!
        components.queryItems = [
            URLQueryItem(name: "client_id", value: clientID),
            URLQueryItem(name: "redirect_uri", value: redirectURI),
            URLQueryItem(name: "response_type", value: "code"),
            URLQueryItem(name: "scope", value: scopes.joined(separator: " ")),
            URLQueryItem(name: "state", value: state),
            URLQueryItem(name: "code_challenge", value: codeChallenge),
            URLQueryItem(name: "code_challenge_method", value: "S256")
        ]
        
        return components.url!
    }
    
    /// เริ่ม OAuth2 flow บน iOS
    @MainActor
    func authorize() async throws -> String {
        let state = UUID().uuidString
        let codeVerifier = PKCE.generateVerifier()
        let authURL = buildAuthorizationURL(state: state, codeVerifier: codeVerifier)
        
        let callbackURL = try await withCheckedThrowingContinuation { continuation in
            let session = ASWebAuthenticationSession(
                url: authURL,
                callbackURLScheme: URL(string: redirectURI)?.scheme
            ) { callbackURL, error in
                if let error = error {
                    continuation.resume(throwing: error)
                } else if let callbackURL = callbackURL {
                    continuation.resume(returning: callbackURL)
                } else {
                    continuation.resume(throwing: OAuth2Error.cancelled)
                }
            }
            session.presentationContextProvider = self
            session.prefersEphemeralWebBrowserSession = true
            session.start()
            self.webAuthSession = session
        }
        
        // Extract authorization code
        guard let components = URLComponents(url: callbackURL, resolvingAgainstBaseURL: false),
              let code = components.queryItems?.first(where: { $0.name == "code" })?.value,
              let returnedState = components.queryItems?.first(where: { $0.name == "state" })?.value,
              returnedState == state else {
            throw OAuth2Error.invalidCallback
        }
        
        // Exchange code for token
        return try await exchangeCode(code, codeVerifier: codeVerifier)
    }
    
    /// แลก authorization code เป็น access token
    private func exchangeCode(_ code: String, codeVerifier: String) async throws -> String {
        var request = URLRequest(url: tokenURL)
        request.httpMethod = "POST"
        request.setValue("application/x-www-form-urlencoded", forHTTPHeaderField: "Content-Type")
        
        let body = [
            "grant_type": "authorization_code",
            "code": code,
            "redirect_uri": redirectURI,
            "client_id": clientID,
            "client_secret": clientSecret,
            "code_verifier": codeVerifier
        ].map { "\($0.key)=\($0.value)" }.joined(separator: "&")
        
        request.httpBody = body.data(using: .utf8)
        
        let (data, _) = try await URLSession.shared.data(for: request)
        let tokenResponse = try JSONDecoder().decode(TokenResponse.self, from: data)
        
        return tokenResponse.accessToken
    }
}

// MARK: - ASWebAuthenticationPresentationContextProviding

extension OAuth2Manager: ASWebAuthenticationPresentationContextProviding {
    func presentationAnchor(for session: ASWebAuthenticationSession) -> ASPresentationAnchor {
        UIApplication.shared.connectedScenes
            .compactMap { $0 as? UIWindowScene }
            .first?.windows.first ?? ASPresentationAnchor()
    }
}

// MARK: - PKCE (Proof Key for Code Exchange)

enum PKCE {
    static func generateVerifier() -> String {
        var bytes = [UInt8](repeating: 0, count: 32)
        _ = SecRandomCopyBytes(kSecRandomDefault, bytes.count, &bytes)
        return Data(bytes).base64URLEncoded
    }
    
    static func generateChallenge(from verifier: String) -> String {
        let data = Data(verifier.utf8)
        let hash = SHA256.hash(data: data)
        return Data(hash).base64URLEncoded
    }
}

// MARK: - Models

struct TokenResponse: Codable {
    let accessToken: String
    let refreshToken: String?
    let expiresIn: Int
    let tokenType: String
    
    enum CodingKeys: String, CodingKey {
        case accessToken = "access_token"
        case refreshToken = "refresh_token"
        case expiresIn = "expires_in"
        case tokenType = "token_type"
    }
}

enum OAuth2Error: LocalizedError {
    case cancelled
    case invalidCallback
    case tokenExchangeFailed
    
    var errorDescription: String? {
        switch self {
        case .cancelled: return "ผู้ใช้ยกเลิก"
        case .invalidCallback: return "Callback URL ไม่ถูกต้อง"
        case .tokenExchangeFailed: return "แลก token ไม่สำเร็จ"
        }
    }
}
```

### API Key Authentication

```swift
// APIKeyAuthenticator.swift

struct APIKeyAuthenticator {
    
    enum Placement {
        case header(name: String)
        case queryParam(name: String)
        case bearerToken
    }
    
    let apiKey: String
    let placement: Placement
    
    func authenticate(request: inout URLRequest) {
        switch placement {
        case .header(let name):
            request.setValue(apiKey, forHTTPHeaderField: name)
            
        case .queryParam(let name):
            guard var components = URLComponents(
                url: request.url!,
                resolvingAgainstBaseURL: false
            ) else { return }
            
            var queryItems = components.queryItems ?? []
            queryItems.append(URLQueryItem(name: name, value: apiKey))
            components.queryItems = queryItems
            request.url = components.url
            
        case .bearerToken:
            request.setValue(
                "Bearer \(apiKey)",
                forHTTPHeaderField: "Authorization"
            )
        }
    }
}

// การใช้งาน
let authenticator = APIKeyAuthenticator(
    apiKey: "my-secret-api-key",
    placement: .header(name: "X-API-Key")
)

var request = URLRequest(url: URL(string: "https://api.example.com/data")!)
authenticator.authenticate(request: &request)
```

---

## 4. Refresh Token Handling

### Token Storage และ Management

```swift
// TokenStorage.swift

import Foundation
import Security

class TokenStorage {
    
    static let shared = TokenStorage()
    
    private let keychainService = "com.example.app"
    
    private enum Keys {
        static let accessToken = "access_token"
        static let refreshToken = "refresh_token"
        static let tokenExpiry = "token_expiry"
    }
    
    // MARK: - Access Token
    
    var accessToken: String? {
        get { getFromKeychain(key: Keys.accessToken) }
        set {
            if let token = newValue {
                saveToKeychain(key: Keys.accessToken, value: token)
            } else {
                deleteFromKeychain(key: Keys.accessToken)
            }
        }
    }
    
    var refreshToken: String? {
        get { getFromKeychain(key: Keys.refreshToken) }
        set {
            if let token = newValue {
                saveToKeychain(key: Keys.refreshToken, value: token)
            } else {
                deleteFromKeychain(key: Keys.refreshToken)
            }
        }
    }
    
    var tokenExpiry: Date? {
        get {
            guard let data = getDataFromKeychain(key: Keys.tokenExpiry) else { return nil }
            return try? JSONDecoder().decode(Date.self, from: data)
        }
        set {
            if let date = newValue,
               let data = try? JSONEncoder().encode(date) {
                saveDataToKeychain(key: Keys.tokenExpiry, data: data)
            } else {
                deleteFromKeychain(key: Keys.tokenExpiry)
            }
        }
    }
    
    var isAccessTokenExpired: Bool {
        guard let expiry = tokenExpiry else { return true }
        // ถือว่า expired ก่อน 5 นาทีเพื่อป้องกัน race condition
        return Date() >= expiry.addingTimeInterval(-300)
    }
    
    func saveTokens(
        accessToken: String,
        refreshToken: String?,
        expiresIn: TimeInterval
    ) {
        self.accessToken = accessToken
        self.refreshToken = refreshToken
        self.tokenExpiry = Date().addingTimeInterval(expiresIn)
    }
    
    func clearTokens() {
        accessToken = nil
        refreshToken = nil
        tokenExpiry = nil
    }
    
    // MARK: - Keychain Operations
    
    private func saveToKeychain(key: String, value: String) {
        saveDataToKeychain(key: key, data: Data(value.utf8))
    }
    
    private func saveDataToKeychain(key: String, data: Data) {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: keychainService,
            kSecAttrAccount as String: key,
            kSecValueData as String: data,
            kSecAttrAccessible as String: kSecAttrAccessibleWhenUnlockedThisDeviceOnly
        ]
        
        SecItemDelete(query as CFDictionary)
        SecItemAdd(query as CFDictionary, nil)
    }
    
    private func getFromKeychain(key: String) -> String? {
        guard let data = getDataFromKeychain(key: key) else { return nil }
        return String(data: data, encoding: .utf8)
    }
    
    private func getDataFromKeychain(key: String) -> Data? {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: keychainService,
            kSecAttrAccount as String: key,
            kSecReturnData as String: true,
            kSecMatchLimit as String: kSecMatchLimitOne
        ]
        
        var result: AnyObject?
        SecItemCopyMatching(query as CFDictionary, &result)
        return result as? Data
    }
    
    private func deleteFromKeychain(key: String) {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: keychainService,
            kSecAttrAccount as String: key
        ]
        SecItemDelete(query as CFDictionary)
    }
}
```

### Token Refresh Manager

```swift
// TokenRefreshManager.swift

actor TokenRefreshManager {
    
    private let tokenStorage: TokenStorage
    private let authService: AuthServiceProtocol
    private var refreshTask: Task<String, Error>?
    
    init(tokenStorage: TokenStorage, authService: AuthServiceProtocol) {
        self.tokenStorage = tokenStorage
        self.authService = authService
    }
    
    /// รับ valid access token (refresh ถ้าจำเป็น)
    func validAccessToken() async throws -> String {
        // ถ้า token ยังไม่ expired ส่งคืนทันที
        if !tokenStorage.isAccessTokenExpired,
           let token = tokenStorage.accessToken {
            return token
        }
        
        // ถ้ากำลัง refresh อยู่ รอผล
        if let existingTask = refreshTask {
            return try await existingTask.value
        }
        
        // เริ่ม refresh
        let task = Task<String, Error> {
            defer { refreshTask = nil }
            
            guard let refreshToken = tokenStorage.refreshToken else {
                throw AuthError.noRefreshToken
            }
            
            let newTokens = try await authService.refreshToken(refreshToken)
            
            tokenStorage.saveTokens(
                accessToken: newTokens.accessToken,
                refreshToken: newTokens.refreshToken,
                expiresIn: TimeInterval(newTokens.expiresIn)
            )
            
            return newTokens.accessToken
        }
        
        refreshTask = task
        return try await task.value
    }
}

enum AuthError: LocalizedError {
    case noRefreshToken
    case refreshFailed
    case sessionExpired
    
    var errorDescription: String? {
        switch self {
        case .noRefreshToken: return "ไม่มี refresh token"
        case .refreshFailed: return "ไม่สามารถ refresh token ได้"
        case .sessionExpired: return "Session หมดอายุ กรุณา login ใหม่"
        }
    }
}
```

---

## 5. Request Interceptors

### Interceptor Pattern

```swift
// RequestInterceptor.swift

protocol RequestInterceptor {
    func intercept(_ request: URLRequest) async throws -> URLRequest
}

protocol ResponseInterceptor {
    func intercept(
        _ response: HTTPURLResponse,
        data: Data,
        for request: URLRequest
    ) async throws -> (HTTPURLResponse, Data)
}

// Chain of interceptors
class InterceptorChain {
    private var requestInterceptors: [RequestInterceptor] = []
    private var responseInterceptors: [ResponseInterceptor] = []
    
    func addRequestInterceptor(_ interceptor: RequestInterceptor) {
        requestInterceptors.append(interceptor)
    }
    
    func addResponseInterceptor(_ interceptor: ResponseInterceptor) {
        responseInterceptors.append(interceptor)
    }
    
    func processRequest(_ request: URLRequest) async throws -> URLRequest {
        var processed = request
        for interceptor in requestInterceptors {
            processed = try await interceptor.intercept(processed)
        }
        return processed
    }
    
    func processResponse(
        _ response: HTTPURLResponse,
        data: Data,
        for request: URLRequest
    ) async throws -> (HTTPURLResponse, Data) {
        var processedResponse = response
        var processedData = data
        
        for interceptor in responseInterceptors {
            (processedResponse, processedData) = try await interceptor.intercept(
                processedResponse,
                data: processedData,
                for: request
            )
        }
        
        return (processedResponse, processedData)
    }
}

// Authentication Interceptor
class AuthInterceptor: RequestInterceptor {
    
    private let tokenManager: TokenRefreshManager
    
    init(tokenManager: TokenRefreshManager) {
        self.tokenManager = tokenManager
    }
    
    func intercept(_ request: URLRequest) async throws -> URLRequest {
        var modified = request
        let token = try await tokenManager.validAccessToken()
        modified.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        return modified
    }
}

// Logging Interceptor
class LoggingInterceptor: RequestInterceptor, ResponseInterceptor {
    
    private let logger: Logger
    
    init(logger: Logger = Logger()) {
        self.logger = logger
    }
    
    func intercept(_ request: URLRequest) async throws -> URLRequest {
        logger.log("""
        ➡️ REQUEST
        URL: \(request.url?.absoluteString ?? "unknown")
        Method: \(request.httpMethod ?? "unknown")
        Headers: \(request.allHTTPHeaderFields ?? [:])
        """)
        return request
    }
    
    func intercept(
        _ response: HTTPURLResponse,
        data: Data,
        for request: URLRequest
    ) async throws -> (HTTPURLResponse, Data) {
        logger.log("""
        ⬅️ RESPONSE
        URL: \(request.url?.absoluteString ?? "unknown")
        Status: \(response.statusCode)
        Body: \(String(data: data, encoding: .utf8) ?? "binary data")
        """)
        return (response, data)
    }
}

// User-Agent Interceptor
class UserAgentInterceptor: RequestInterceptor {
    
    private let userAgent: String
    
    init() {
        let appName = Bundle.main.infoDictionary?["CFBundleName"] as? String ?? "Unknown"
        let appVersion = Bundle.main.infoDictionary?["CFBundleShortVersionString"] as? String ?? "0.0"
        let osVersion = ProcessInfo.processInfo.operatingSystemVersionString
        self.userAgent = "\(appName)/\(appVersion) iOS/\(osVersion)"
    }
    
    func intercept(_ request: URLRequest) async throws -> URLRequest {
        var modified = request
        modified.setValue(userAgent, forHTTPHeaderField: "User-Agent")
        return modified
    }
}
```

---

## 6. Response Parsing Strategy

### Generic Response Parsing

```swift
// ResponseParser.swift

protocol ResponseParser {
    associatedtype Output
    func parse(_ data: Data) throws -> Output
}

// JSON Response Parser
struct JSONResponseParser<T: Decodable>: ResponseParser {
    
    let decoder: JSONDecoder
    
    init(decoder: JSONDecoder = .init()) {
        self.decoder = decoder
    }
    
    func parse(_ data: Data) throws -> T {
        do {
            return try decoder.decode(T.self, from: data)
        } catch let decodingError as DecodingError {
            throw NetworkError.decodingFailed(decodingError)
        }
    }
}

// API Response Wrapper
struct APIResponse<T: Decodable>: Decodable {
    let data: T?
    let error: APIErrorResponse?
    let meta: MetaData?
    
    struct MetaData: Decodable {
        let page: Int?
        let perPage: Int?
        let total: Int?
        let totalPages: Int?
        
        enum CodingKeys: String, CodingKey {
            case page
            case perPage = "per_page"
            case total
            case totalPages = "total_pages"
        }
    }
}

struct APIErrorResponse: Decodable {
    let code: String
    let message: String
    let details: [String: String]?
}

// Paginated Response
struct PaginatedResponse<T: Decodable>: Decodable {
    let items: [T]
    let pagination: Pagination
    
    struct Pagination: Decodable {
        let currentPage: Int
        let totalPages: Int
        let totalItems: Int
        let itemsPerPage: Int
        let hasNextPage: Bool
        let hasPreviousPage: Bool
        
        enum CodingKeys: String, CodingKey {
            case currentPage = "current_page"
            case totalPages = "total_pages"
            case totalItems = "total_items"
            case itemsPerPage = "items_per_page"
            case hasNextPage = "has_next_page"
            case hasPreviousPage = "has_previous_page"
        }
    }
}

// Custom Date Decoding
extension JSONDecoder {
    static var apiDecoder: JSONDecoder {
        let decoder = JSONDecoder()
        decoder.keyDecodingStrategy = .convertFromSnakeCase
        
        let formatter = DateFormatter()
        formatter.dateFormat = "yyyy-MM-dd'T'HH:mm:ssZ"
        formatter.locale = Locale(identifier: "en_US_POSIX")
        
        decoder.dateDecodingStrategy = .formatted(formatter)
        return decoder
    }
}
```

---

## 7. API Error Handling

### Comprehensive Error Types

```swift
// NetworkError.swift

enum NetworkError: LocalizedError, Equatable {
    
    // Connection errors
    case noInternetConnection
    case timeout
    case connectionLost
    
    // Request errors
    case invalidURL
    case encodingFailed(Error)
    
    // Response errors
    case invalidResponse
    case decodingFailed(Error)
    case unexpectedStatusCode(Int)
    
    // Server errors
    case serverError(statusCode: Int, message: String?)
    case rateLimitExceeded(retryAfter: TimeInterval?)
    
    // Auth errors
    case unauthorized
    case forbidden
    case sessionExpired
    
    // Business errors
    case apiError(code: String, message: String)
    
    // Generic
    case unknown(Error)
    
    var errorDescription: String? {
        switch self {
        case .noInternetConnection:
            return "ไม่มีการเชื่อมต่ออินเทอร์เน็ต"
        case .timeout:
            return "Request หมดเวลา กรุณาลองใหม่"
        case .connectionLost:
            return "การเชื่อมต่อขาดหาย กรุณาลองใหม่"
        case .invalidURL:
            return "URL ไม่ถูกต้อง"
        case .encodingFailed(let error):
            return "ไม่สามารถ encode ข้อมูลได้: \(error.localizedDescription)"
        case .invalidResponse:
            return "ได้รับ response ที่ไม่ถูกต้อง"
        case .decodingFailed(let error):
            return "ไม่สามารถอ่านข้อมูลได้: \(error.localizedDescription)"
        case .unexpectedStatusCode(let code):
            return "Status code ไม่คาดคิด: \(code)"
        case .serverError(let code, let message):
            return message ?? "Server error (\(code))"
        case .rateLimitExceeded(let retryAfter):
            if let retryAfter = retryAfter {
                return "เกิน rate limit กรุณารออีก \(Int(retryAfter)) วินาที"
            }
            return "เกิน rate limit กรุณาลองใหม่ภายหลัง"
        case .unauthorized:
            return "ไม่ได้รับอนุญาต กรุณา login ใหม่"
        case .forbidden:
            return "ไม่มีสิทธิ์เข้าถึง"
        case .sessionExpired:
            return "Session หมดอายุ กรุณา login ใหม่"
        case .apiError(_, let message):
            return message
        case .unknown(let error):
            return "เกิดข้อผิดพลาดที่ไม่คาดคิด: \(error.localizedDescription)"
        }
    }
    
    var isRetryable: Bool {
        switch self {
        case .timeout, .connectionLost, .noInternetConnection:
            return true
        case .serverError(let code, _):
            return [503, 504].contains(code)
        case .rateLimitExceeded:
            return true
        default:
            return false
        }
    }
    
    static func == (lhs: NetworkError, rhs: NetworkError) -> Bool {
        switch (lhs, rhs) {
        case (.noInternetConnection, .noInternetConnection): return true
        case (.timeout, .timeout): return true
        case (.unauthorized, .unauthorized): return true
        case (.forbidden, .forbidden): return true
        default: return false
        }
    }
}

// Error Handler
class NetworkErrorHandler {
    
    static func handle(_ error: Error) -> NetworkError {
        if let networkError = error as? NetworkError {
            return networkError
        }
        
        if let urlError = error as? URLError {
            return mapURLError(urlError)
        }
        
        return .unknown(error)
    }
    
    private static func mapURLError(_ error: URLError) -> NetworkError {
        switch error.code {
        case .notConnectedToInternet, .networkConnectionLost:
            return .noInternetConnection
        case .timedOut:
            return .timeout
        case .cancelled:
            return .connectionLost
        default:
            return .unknown(error)
        }
    }
    
    static func handle(response: HTTPURLResponse, data: Data) throws {
        switch response.statusCode {
        case 200...299:
            return // Success
            
        case 401:
            throw NetworkError.unauthorized
            
        case 403:
            throw NetworkError.forbidden
            
        case 429:
            let retryAfter = response.value(forHTTPHeaderField: "Retry-After")
                .flatMap { TimeInterval($0) }
            throw NetworkError.rateLimitExceeded(retryAfter: retryAfter)
            
        case 500...599:
            let message = parseErrorMessage(from: data)
            throw NetworkError.serverError(
                statusCode: response.statusCode,
                message: message
            )
            
        default:
            throw NetworkError.unexpectedStatusCode(response.statusCode)
        }
    }
    
    private static func parseErrorMessage(from data: Data) -> String? {
        let errorResponse = try? JSONDecoder().decode(APIErrorResponse.self, from: data)
        return errorResponse?.message
    }
}
```

---

## 8. Retry Logic

### Retry Policy

```swift
// RetryPolicy.swift

struct RetryPolicy {
    let maxAttempts: Int
    let backoffStrategy: BackoffStrategy
    let retryableErrors: (NetworkError) -> Bool
    
    enum BackoffStrategy {
        case fixed(TimeInterval)           // รอเวลาเดิมทุกครั้ง
        case linear(initial: TimeInterval) // รอเพิ่มขึ้นเรื่อยๆ
        case exponential(                  // รอแบบ exponential
            initial: TimeInterval,
            multiplier: Double,
            maxDelay: TimeInterval
        )
        case custom((Int) -> TimeInterval) // กำหนดเอง
        
        func delay(forAttempt attempt: Int) -> TimeInterval {
            switch self {
            case .fixed(let delay):
                return delay
                
            case .linear(let initial):
                return initial * Double(attempt)
                
            case .exponential(let initial, let multiplier, let max):
                let delay = initial * pow(multiplier, Double(attempt - 1))
                return min(delay, max)
                
            case .custom(let calculator):
                return calculator(attempt)
            }
        }
    }
    
    static let `default` = RetryPolicy(
        maxAttempts: 3,
        backoffStrategy: .exponential(
            initial: 1.0,
            multiplier: 2.0,
            maxDelay: 60.0
        ),
        retryableErrors: { $0.isRetryable }
    )
    
    static let aggressive = RetryPolicy(
        maxAttempts: 5,
        backoffStrategy: .exponential(
            initial: 0.5,
            multiplier: 1.5,
            maxDelay: 30.0
        ),
        retryableErrors: { $0.isRetryable }
    )
    
    static let none = RetryPolicy(
        maxAttempts: 1,
        backoffStrategy: .fixed(0),
        retryableErrors: { _ in false }
    )
}

// Retry Executor
struct RetryExecutor {
    
    let policy: RetryPolicy
    
    func execute<T>(
        _ operation: () async throws -> T
    ) async throws -> T {
        var lastError: Error?
        
        for attempt in 1...policy.maxAttempts {
            do {
                return try await operation()
            } catch let networkError as NetworkError {
                lastError = networkError
                
                // ตรวจสอบว่าควร retry หรือไม่
                guard policy.retryableErrors(networkError),
                      attempt < policy.maxAttempts else {
                    throw networkError
                }
                
                // คำนวณ delay
                let delay = policy.backoffStrategy.delay(forAttempt: attempt)
                
                // สำหรับ rate limit ให้ใช้ Retry-After
                let actualDelay: TimeInterval
                if case .rateLimitExceeded(let retryAfter) = networkError,
                   let retryAfter = retryAfter {
                    actualDelay = retryAfter
                } else {
                    actualDelay = delay
                }
                
                // เพิ่ม jitter เพื่อป้องกัน thundering herd
                let jitter = Double.random(in: 0...0.1) * actualDelay
                
                try await Task.sleep(
                    nanoseconds: UInt64((actualDelay + jitter) * 1_000_000_000)
                )
                
            } catch {
                lastError = error
                throw error  // Non-network errors ไม่ retry
            }
        }
        
        throw lastError ?? NetworkError.unknown(NSError())
    }
}
```

---

## 9. Request Deduplication

### Deduplication Manager

```swift
// RequestDeduplicator.swift

actor RequestDeduplicator<Response> {
    
    private var pendingRequests: [String: Task<Response, Error>] = [:]
    
    /// Execute request หรือ wait สำหรับ existing request ที่เหมือนกัน
    func execute(
        key: String,
        operation: () async throws -> Response
    ) async throws -> Response {
        
        // ถ้ามี request นี้อยู่แล้ว รอผล
        if let existingTask = pendingRequests[key] {
            return try await existingTask.value
        }
        
        // สร้าง task ใหม่
        let task = Task<Response, Error> {
            defer { pendingRequests.removeValue(forKey: key) }
            return try await operation()
        }
        
        pendingRequests[key] = task
        return try await task.value
    }
}

// การใช้งาน
class UserService {
    
    private let client: NetworkClient
    private let deduplicator = RequestDeduplicator<User>()
    
    init(client: NetworkClient) {
        self.client = client
    }
    
    func getUser(id: Int) async throws -> User {
        let key = "user_\(id)"
        return try await deduplicator.execute(key: key) {
            try await self.client.get("/users/\(id)")
        }
    }
}
```

---

## 10. Network Layer Architecture

### Protocol-Based Network Layer

```swift
// NetworkClient.swift

// MARK: - Protocol

protocol NetworkClientProtocol {
    func request<T: Decodable>(
        _ endpoint: EndpointProtocol,
        responseType: T.Type
    ) async throws -> T
    
    func requestData(_ endpoint: EndpointProtocol) async throws -> Data
    
    func requestWithResponse<T: Decodable>(
        _ endpoint: EndpointProtocol,
        responseType: T.Type
    ) async throws -> (T, HTTPURLResponse)
}

// MARK: - Endpoint Protocol

protocol EndpointProtocol {
    var baseURL: URL { get }
    var path: String { get }
    var method: HTTPMethod { get }
    var headers: [String: String] { get }
    var queryParameters: [String: String]? { get }
    var body: Data? { get }
    var timeout: TimeInterval { get }
    var requiresAuth: Bool { get }
}

extension EndpointProtocol {
    var headers: [String: String] {
        ["Content-Type": "application/json",
         "Accept": "application/json"]
    }
    var queryParameters: [String: String]? { nil }
    var body: Data? { nil }
    var timeout: TimeInterval { 30 }
    var requiresAuth: Bool { true }
    
    func buildURLRequest() throws -> URLRequest {
        var components = URLComponents(
            url: baseURL.appendingPathComponent(path),
            resolvingAgainstBaseURL: false
        )
        
        if let params = queryParameters {
            components?.queryItems = params.map {
                URLQueryItem(name: $0.key, value: $0.value)
            }
        }
        
        guard let url = components?.url else {
            throw NetworkError.invalidURL
        }
        
        var request = URLRequest(url: url, timeoutInterval: timeout)
        request.httpMethod = method.rawValue
        request.httpBody = body
        
        for (key, value) in headers {
            request.setValue(value, forHTTPHeaderField: key)
        }
        
        return request
    }
}

// MARK: - Network Client Implementation

class NetworkClient: NetworkClientProtocol {
    
    private let session: URLSession
    private let interceptorChain: InterceptorChain
    private let retryExecutor: RetryExecutor
    
    init(
        configuration: URLSessionConfiguration = .default,
        interceptorChain: InterceptorChain = InterceptorChain(),
        retryPolicy: RetryPolicy = .default
    ) {
        self.session = URLSession(configuration: configuration)
        self.interceptorChain = interceptorChain
        self.retryExecutor = RetryExecutor(policy: retryPolicy)
    }
    
    func request<T: Decodable>(
        _ endpoint: EndpointProtocol,
        responseType: T.Type
    ) async throws -> T {
        let data = try await requestData(endpoint)
        return try JSONDecoder.apiDecoder.decode(T.self, from: data)
    }
    
    func requestData(_ endpoint: EndpointProtocol) async throws -> Data {
        let (data, _) = try await requestWithRawResponse(endpoint)
        return data
    }
    
    func requestWithResponse<T: Decodable>(
        _ endpoint: EndpointProtocol,
        responseType: T.Type
    ) async throws -> (T, HTTPURLResponse) {
        let (data, response) = try await requestWithRawResponse(endpoint)
        let decoded = try JSONDecoder.apiDecoder.decode(T.self, from: data)
        return (decoded, response)
    }
    
    private func requestWithRawResponse(
        _ endpoint: EndpointProtocol
    ) async throws -> (Data, HTTPURLResponse) {
        
        return try await retryExecutor.execute {
            // Build request
            var request = try endpoint.buildURLRequest()
            
            // Apply interceptors
            request = try await self.interceptorChain.processRequest(request)
            
            // Execute request
            let (data, response) = try await self.session.data(for: request)
            
            guard let httpResponse = response as? HTTPURLResponse else {
                throw NetworkError.invalidResponse
            }
            
            // Handle HTTP errors
            try NetworkErrorHandler.handle(response: httpResponse, data: data)
            
            // Apply response interceptors
            let (processedResponse, processedData) = try await self.interceptorChain.processResponse(
                httpResponse,
                data: data,
                for: request
            )
            
            return (processedData, processedResponse)
        }
    }
}
```

---

## 11. Alamofire Overview

### การใช้งาน Alamofire พื้นฐาน

```swift
// AlamofireNetworkClient.swift

import Alamofire

class AlamofireNetworkClient {
    
    private let session: Session
    
    init(configuration: URLSessionConfiguration = .default) {
        let interceptor = AuthInterceptorAdapter()
        self.session = Session(
            configuration: configuration,
            interceptor: interceptor
        )
    }
    
    // GET request
    func get<T: Decodable>(
        _ url: URLConvertible,
        parameters: Parameters? = nil
    ) async throws -> T {
        try await withCheckedThrowingContinuation { continuation in
            session.request(
                url,
                method: .get,
                parameters: parameters,
                encoding: URLEncoding.default
            )
            .validate()
            .responseDecodable(of: T.self, decoder: JSONDecoder.apiDecoder) { response in
                switch response.result {
                case .success(let value):
                    continuation.resume(returning: value)
                case .failure(let error):
                    continuation.resume(throwing: error)
                }
            }
        }
    }
    
    // POST request
    func post<T: Decodable, E: Encodable>(
        _ url: URLConvertible,
        body: E
    ) async throws -> T {
        try await withCheckedThrowingContinuation { continuation in
            session.request(
                url,
                method: .post,
                parameters: body,
                encoder: JSONParameterEncoder.default
            )
            .validate()
            .responseDecodable(of: T.self) { response in
                switch response.result {
                case .success(let value):
                    continuation.resume(returning: value)
                case .failure(let error):
                    continuation.resume(throwing: error)
                }
            }
        }
    }
    
    // Upload file
    func upload(
        _ url: URLConvertible,
        fileData: Data,
        mimeType: String,
        progressHandler: ((Double) -> Void)? = nil
    ) async throws -> UploadResponse {
        try await withCheckedThrowingContinuation { continuation in
            session.upload(
                multipartFormData: { formData in
                    formData.append(
                        fileData,
                        withName: "file",
                        fileName: "upload.\(mimeType.split(separator: "/").last ?? "bin")",
                        mimeType: mimeType
                    )
                },
                to: url
            )
            .uploadProgress { progress in
                progressHandler?(progress.fractionCompleted)
            }
            .validate()
            .responseDecodable(of: UploadResponse.self) { response in
                switch response.result {
                case .success(let value):
                    continuation.resume(returning: value)
                case .failure(let error):
                    continuation.resume(throwing: error)
                }
            }
        }
    }
}

// Alamofire RequestInterceptor adapter
class AuthInterceptorAdapter: RequestInterceptor {
    
    func adapt(
        _ urlRequest: URLRequest,
        for session: Session,
        completion: @escaping (Result<URLRequest, Error>) -> Void
    ) {
        var request = urlRequest
        if let token = TokenStorage.shared.accessToken {
            request.headers.add(.authorization(bearerToken: token))
        }
        completion(.success(request))
    }
    
    func retry(
        _ request: Request,
        for session: Session,
        dueTo error: Error,
        completion: @escaping (RetryResult) -> Void
    ) {
        guard let response = request.task?.response as? HTTPURLResponse,
              response.statusCode == 401,
              request.retryCount < 1 else {
            completion(.doNotRetry)
            return
        }
        
        // Refresh token
        Task {
            do {
                _ = try await TokenRefreshManager.shared.validAccessToken()
                completion(.retry)
            } catch {
                completion(.doNotRetryWithError(error))
            }
        }
    }
}
```

---

## 12. Moya Overview

### การสร้าง API Layer ด้วย Moya

```swift
// UserAPI.swift - Moya TargetType

import Moya

enum UserAPI {
    case getUsers(page: Int, perPage: Int)
    case getUser(id: Int)
    case createUser(name: String, email: String)
    case updateUser(id: Int, name: String)
    case deleteUser(id: Int)
}

extension UserAPI: TargetType {
    
    var baseURL: URL {
        URL(string: "https://api.example.com/v2")!
    }
    
    var path: String {
        switch self {
        case .getUsers:
            return "/users"
        case .getUser(let id):
            return "/users/\(id)"
        case .createUser:
            return "/users"
        case .updateUser(let id, _):
            return "/users/\(id)"
        case .deleteUser(let id):
            return "/users/\(id)"
        }
    }
    
    var method: Moya.Method {
        switch self {
        case .getUsers, .getUser:
            return .get
        case .createUser:
            return .post
        case .updateUser:
            return .put
        case .deleteUser:
            return .delete
        }
    }
    
    var task: Task {
        switch self {
        case .getUsers(let page, let perPage):
            return .requestParameters(
                parameters: ["page": page, "per_page": perPage],
                encoding: URLEncoding.queryString
            )
            
        case .getUser, .deleteUser:
            return .requestPlain
            
        case .createUser(let name, let email):
            return .requestJSONEncodable(CreateUserRequest(name: name, email: email))
            
        case .updateUser(_, let name):
            return .requestJSONEncodable(UpdateUserRequest(name: name))
        }
    }
    
    var headers: [String: String]? {
        ["Content-Type": "application/json",
         "Accept": "application/json"]
    }
    
    var sampleData: Data {
        switch self {
        case .getUsers:
            return """
            {
                "users": [
                    {"id": 1, "name": "John Doe", "email": "john@example.com"},
                    {"id": 2, "name": "Jane Smith", "email": "jane@example.com"}
                ]
            }
            """.data(using: .utf8)!
            
        case .getUser(let id):
            return """
            {"id": \(id), "name": "John Doe", "email": "john@example.com"}
            """.data(using: .utf8)!
            
        default:
            return Data()
        }
    }
}

// Moya Service
class UserService {
    
    private let provider: MoyaProvider<UserAPI>
    
    init(stubbing: Bool = false) {
        if stubbing {
            provider = MoyaProvider<UserAPI>(stubClosure: MoyaProvider.immediatelyStub)
        } else {
            let authPlugin = AccessTokenPlugin { _ in
                TokenStorage.shared.accessToken ?? ""
            }
            provider = MoyaProvider<UserAPI>(plugins: [authPlugin, NetworkLoggerPlugin()])
        }
    }
    
    func getUsers(page: Int = 1, perPage: Int = 20) async throws -> [User] {
        try await provider.async.request(.getUsers(page: page, perPage: perPage))
            .map([User].self, atKeyPath: "users")
    }
    
    func getUser(id: Int) async throws -> User {
        try await provider.async.request(.getUser(id: id))
            .map(User.self)
    }
}
```

---

## 13. GraphQL กับ Apollo iOS

### Apollo iOS Setup

```swift
// Package.swift - เพิ่ม Apollo dependency
.package(
    url: "https://github.com/apollographql/apollo-ios.git",
    from: "1.7.0"
)
```

### GraphQL Queries

```graphql
# GetUser.graphql
query GetUser($id: ID!) {
    user(id: $id) {
        id
        name
        email
        avatar {
            url
            width
            height
        }
        posts(first: 5) {
            edges {
                node {
                    id
                    title
                    createdAt
                }
            }
        }
    }
}

# CreateUser.graphql
mutation CreateUser($input: CreateUserInput!) {
    createUser(input: $input) {
        user {
            id
            name
            email
        }
        errors {
            field
            message
        }
    }
}
```

```swift
// GraphQLClient.swift

import Apollo

class GraphQLClient {
    
    static let shared = GraphQLClient()
    
    private let apollo: ApolloClient
    
    private init() {
        let store = ApolloStore()
        let interceptorProvider = DefaultInterceptorProvider(store: store)
        let networkTransport = RequestChainNetworkTransport(
            interceptorProvider: interceptorProvider,
            endpointURL: URL(string: "https://api.example.com/graphql")!
        )
        apollo = ApolloClient(networkTransport: networkTransport, store: store)
    }
    
    // Query
    func fetch<Query: GraphQLQuery>(
        _ query: Query
    ) async throws -> Query.Data {
        try await withCheckedThrowingContinuation { continuation in
            apollo.fetch(query: query) { result in
                switch result {
                case .success(let graphQLResult):
                    if let data = graphQLResult.data {
                        continuation.resume(returning: data)
                    } else if let errors = graphQLResult.errors {
                        continuation.resume(throwing: GraphQLError.errors(errors))
                    } else {
                        continuation.resume(throwing: GraphQLError.noData)
                    }
                case .failure(let error):
                    continuation.resume(throwing: error)
                }
            }
        }
    }
    
    // Mutation
    func perform<Mutation: GraphQLMutation>(
        _ mutation: Mutation
    ) async throws -> Mutation.Data {
        try await withCheckedThrowingContinuation { continuation in
            apollo.perform(mutation: mutation) { result in
                switch result {
                case .success(let graphQLResult):
                    if let data = graphQLResult.data {
                        continuation.resume(returning: data)
                    } else if let errors = graphQLResult.errors {
                        continuation.resume(throwing: GraphQLError.errors(errors))
                    } else {
                        continuation.resume(throwing: GraphQLError.noData)
                    }
                case .failure(let error):
                    continuation.resume(throwing: error)
                }
            }
        }
    }
    
    // Subscription
    func subscribe<Subscription: GraphQLSubscription>(
        _ subscription: Subscription
    ) -> AsyncThrowingStream<Subscription.Data, Error> {
        AsyncThrowingStream { continuation in
            let cancellable = apollo.subscribe(subscription: subscription) { result in
                switch result {
                case .success(let graphQLResult):
                    if let data = graphQLResult.data {
                        continuation.yield(data)
                    }
                case .failure(let error):
                    continuation.finish(throwing: error)
                }
            }
            
            continuation.onTermination = { _ in
                cancellable.cancel()
            }
        }
    }
}

enum GraphQLError: LocalizedError {
    case noData
    case errors([GraphQLError])
    
    var errorDescription: String? {
        switch self {
        case .noData: return "ไม่ได้รับข้อมูล"
        case .errors(let errors): return errors.map { $0.message }.joined(separator: "\n")
        }
    }
}
```

---

## 14. WebSocket กับ URLSessionWebSocketTask

### WebSocket Manager

```swift
// WebSocketManager.swift

import Foundation

class WebSocketManager: NSObject {
    
    private var webSocketTask: URLSessionWebSocketTask?
    private var session: URLSession?
    private var pingTimer: Timer?
    
    // Message handler
    var onMessage: ((WebSocketMessage) -> Void)?
    var onConnect: (() -> Void)?
    var onDisconnect: ((Error?) -> Void)?
    
    enum WebSocketMessage {
        case text(String)
        case data(Data)
    }
    
    // MARK: - Connection
    
    func connect(to url: URL, headers: [String: String] = [:]) {
        var request = URLRequest(url: url)
        for (key, value) in headers {
            request.setValue(value, forHTTPHeaderField: key)
        }
        
        let configuration = URLSessionConfiguration.default
        session = URLSession(
            configuration: configuration,
            delegate: self,
            delegateQueue: .main
        )
        
        webSocketTask = session?.webSocketTask(with: request)
        webSocketTask?.resume()
        
        startReceiving()
        startPinging()
    }
    
    func disconnect(code: URLSessionWebSocketTask.CloseCode = .normalClosure) {
        stopPinging()
        webSocketTask?.cancel(with: code, reason: nil)
        webSocketTask = nil
    }
    
    // MARK: - Sending Messages
    
    func send(text: String) async throws {
        let message = URLSessionWebSocketTask.Message.string(text)
        try await webSocketTask?.send(message)
    }
    
    func send(data: Data) async throws {
        let message = URLSessionWebSocketTask.Message.data(data)
        try await webSocketTask?.send(message)
    }
    
    func sendJSON<T: Encodable>(_ value: T) async throws {
        let data = try JSONEncoder().encode(value)
        try await send(data: data)
    }
    
    // MARK: - Receiving Messages
    
    private func startReceiving() {
        webSocketTask?.receive { [weak self] result in
            switch result {
            case .success(let message):
                switch message {
                case .string(let text):
                    self?.onMessage?(.text(text))
                case .data(let data):
                    self?.onMessage?(.data(data))
                @unknown default:
                    break
                }
                // ต้อง receive อีกครั้งเพื่อรับ message ต่อไป
                self?.startReceiving()
                
            case .failure(let error):
                self?.onDisconnect?(error)
            }
        }
    }
    
    // MARK: - Ping/Pong
    
    private func startPinging() {
        pingTimer = Timer.scheduledTimer(withTimeInterval: 30, repeats: true) { [weak self] _ in
            self?.webSocketTask?.sendPing { error in
                if let error = error {
                    print("Ping failed: \(error.localizedDescription)")
                }
            }
        }
    }
    
    private func stopPinging() {
        pingTimer?.invalidate()
        pingTimer = nil
    }
}

// MARK: - URLSessionWebSocketDelegate

extension WebSocketManager: URLSessionWebSocketDelegate {
    
    func urlSession(
        _ session: URLSession,
        webSocketTask: URLSessionWebSocketTask,
        didOpenWithProtocol protocol: String?
    ) {
        onConnect?()
    }
    
    func urlSession(
        _ session: URLSession,
        webSocketTask: URLSessionWebSocketTask,
        didCloseWith closeCode: URLSessionWebSocketTask.CloseCode,
        reason: Data?
    ) {
        onDisconnect?(nil)
    }
}

// MARK: - Chat Example

struct ChatMessage: Codable {
    let type: String
    let userId: String
    let content: String
    let timestamp: Date
}

class ChatService {
    
    private let webSocketManager = WebSocketManager()
    
    var onMessageReceived: ((ChatMessage) -> Void)?
    
    func connect(roomId: String, token: String) {
        let url = URL(string: "wss://chat.example.com/rooms/\(roomId)")!
        webSocketManager.connect(
            to: url,
            headers: ["Authorization": "Bearer \(token)"]
        )
        
        webSocketManager.onMessage = { [weak self] message in
            if case .text(let text) = message,
               let data = text.data(using: .utf8),
               let chatMessage = try? JSONDecoder().decode(ChatMessage.self, from: data) {
                self?.onMessageReceived?(chatMessage)
            }
        }
    }
    
    func sendMessage(_ content: String, userId: String) async throws {
        let message = ChatMessage(
            type: "message",
            userId: userId,
            content: content,
            timestamp: Date()
        )
        try await webSocketManager.sendJSON(message)
    }
}
```

---

## 15. Server-Sent Events (SSE)

### SSE Client

```swift
// SSEClient.swift

import Foundation

class SSEClient {
    
    private let url: URL
    private let headers: [String: String]
    private var task: URLSessionDataTask?
    private var session: URLSession?
    
    var onEvent: ((SSEEvent) -> Void)?
    var onError: ((Error) -> Void)?
    var onConnect: (() -> Void)?
    var onDisconnect: (() -> Void)?
    
    struct SSEEvent {
        let id: String?
        let event: String?
        let data: String
        let retry: TimeInterval?
    }
    
    init(url: URL, headers: [String: String] = [:]) {
        self.url = url
        self.headers = headers
    }
    
    func connect() {
        var request = URLRequest(url: url)
        request.setValue("text/event-stream", forHTTPHeaderField: "Accept")
        request.setValue("no-cache", forHTTPHeaderField: "Cache-Control")
        
        for (key, value) in headers {
            request.setValue(value, forHTTPHeaderField: key)
        }
        
        let configuration = URLSessionConfiguration.default
        configuration.timeoutIntervalForRequest = TimeInterval(INT_MAX)
        configuration.timeoutIntervalForResource = TimeInterval(INT_MAX)
        
        session = URLSession(
            configuration: configuration,
            delegate: SSESessionDelegate(client: self),
            delegateQueue: .main
        )
        
        task = session?.dataTask(with: request)
        task?.resume()
        onConnect?()
    }
    
    func disconnect() {
        task?.cancel()
        task = nil
        session?.invalidateAndCancel()
        session = nil
        onDisconnect?()
    }
    
    fileprivate func processData(_ data: Data) {
        guard let text = String(data: data, encoding: .utf8) else { return }
        
        // Parse SSE format
        let lines = text.components(separatedBy: "\n")
        var eventId: String?
        var eventType: String?
        var eventData: [String] = []
        var retryTime: TimeInterval?
        
        for line in lines {
            if line.isEmpty {
                // Empty line = dispatch event
                if !eventData.isEmpty {
                    let event = SSEEvent(
                        id: eventId,
                        event: eventType,
                        data: eventData.joined(separator: "\n"),
                        retry: retryTime
                    )
                    onEvent?(event)
                    eventData = []
                    eventId = nil
                    eventType = nil
                }
            } else if line.hasPrefix("id:") {
                eventId = line.dropFirst(3).trimmingCharacters(in: .whitespaces)
            } else if line.hasPrefix("event:") {
                eventType = line.dropFirst(6).trimmingCharacters(in: .whitespaces)
            } else if line.hasPrefix("data:") {
                let data = line.dropFirst(5).trimmingCharacters(in: .whitespaces)
                eventData.append(data)
            } else if line.hasPrefix("retry:") {
                let retryStr = line.dropFirst(6).trimmingCharacters(in: .whitespaces)
                retryTime = TimeInterval(retryStr)
            }
        }
    }
}

private class SSESessionDelegate: NSObject, URLSessionDataDelegate {
    
    weak var client: SSEClient?
    
    init(client: SSEClient) {
        self.client = client
    }
    
    func urlSession(_ session: URLSession, dataTask: URLSessionDataTask, didReceive data: Data) {
        client?.processData(data)
    }
    
    func urlSession(_ session: URLSession, task: URLSessionTask, didCompleteWithError error: Error?) {
        if let error = error {
            client?.onError?(error)
        } else {
            client?.onDisconnect?()
        }
    }
}

// AsyncStream wrapper สำหรับ SSE
func sseEvents(url: URL, headers: [String: String] = [:]) -> AsyncThrowingStream<SSEClient.SSEEvent, Error> {
    AsyncThrowingStream { continuation in
        let client = SSEClient(url: url, headers: headers)
        
        client.onEvent = { event in
            continuation.yield(event)
        }
        
        client.onError = { error in
            continuation.finish(throwing: error)
        }
        
        client.onDisconnect = {
            continuation.finish()
        }
        
        client.connect()
        
        continuation.onTermination = { _ in
            client.disconnect()
        }
    }
}

// การใช้งาน
func streamUpdates() async {
    let url = URL(string: "https://api.example.com/stream")!
    
    do {
        for try await event in sseEvents(url: url) {
            print("Event: \(event.event ?? "message")")
            print("Data: \(event.data)")
        }
    } catch {
        print("Stream error: \(error)")
    }
}
```

---

## 16. Network Monitoring (NWPathMonitor)

### Network Monitor

```swift
// NetworkMonitor.swift

import Network
import Combine

class NetworkMonitor: ObservableObject {
    
    static let shared = NetworkMonitor()
    
    @Published private(set) var isConnected: Bool = false
    @Published private(set) var connectionType: ConnectionType = .unknown
    @Published private(set) var isExpensive: Bool = false  // Cellular
    @Published private(set) var isConstrained: Bool = false  // Low Data Mode
    
    private let monitor = NWPathMonitor()
    private let queue = DispatchQueue(label: "NetworkMonitor")
    
    enum ConnectionType {
        case wifi
        case cellular
        case ethernet
        case unknown
        
        var description: String {
            switch self {
            case .wifi: return "Wi-Fi"
            case .cellular: return "Cellular"
            case .ethernet: return "Ethernet"
            case .unknown: return "Unknown"
            }
        }
    }
    
    private init() {
        startMonitoring()
    }
    
    func startMonitoring() {
        monitor.pathUpdateHandler = { [weak self] path in
            DispatchQueue.main.async {
                self?.isConnected = path.status == .satisfied
                self?.isExpensive = path.isExpensive
                self?.isConstrained = path.isConstrained
                self?.connectionType = self?.getConnectionType(path) ?? .unknown
            }
        }
        monitor.start(queue: queue)
    }
    
    func stopMonitoring() {
        monitor.cancel()
    }
    
    private func getConnectionType(_ path: NWPath) -> ConnectionType {
        if path.usesInterfaceType(.wifi) {
            return .wifi
        } else if path.usesInterfaceType(.cellular) {
            return .cellular
        } else if path.usesInterfaceType(.wiredEthernet) {
            return .ethernet
        }
        return .unknown
    }
    
    /// รอจนกว่าจะมีการเชื่อมต่อ
    func waitForConnection() async {
        guard !isConnected else { return }
        
        await withCheckedContinuation { continuation in
            var cancellable: AnyCancellable?
            cancellable = $isConnected
                .filter { $0 }
                .first()
                .sink { _ in
                    continuation.resume()
                    cancellable?.cancel()
                }
        }
    }
}

// SwiftUI Integration
import SwiftUI

struct NetworkStatusView: View {
    @ObservedObject private var monitor = NetworkMonitor.shared
    
    var body: some View {
        HStack(spacing: 8) {
            Circle()
                .fill(monitor.isConnected ? Color.green : Color.red)
                .frame(width: 10, height: 10)
            
            Text(monitor.isConnected ? monitor.connectionType.description : "ไม่มีสัญญาณ")
                .font(.caption)
                .foregroundColor(.secondary)
            
            if monitor.isExpensive {
                Image(systemName: "antenna.radiowaves.left.and.right")
                    .font(.caption)
                    .foregroundColor(.orange)
            }
        }
    }
}
```

---

## 17. Offline Support Patterns

### Offline Queue

```swift
// OfflineRequestQueue.swift

import Foundation

class OfflineRequestQueue {
    
    static let shared = OfflineRequestQueue()
    
    private let storage: UserDefaults
    private let key = "offline_request_queue"
    
    struct QueuedRequest: Codable {
        let id: UUID
        let url: String
        let method: String
        let headers: [String: String]
        let body: Data?
        let createdAt: Date
        var retryCount: Int
    }
    
    private init(storage: UserDefaults = .standard) {
        self.storage = storage
    }
    
    var pendingRequests: [QueuedRequest] {
        get {
            guard let data = storage.data(forKey: key),
                  let requests = try? JSONDecoder().decode([QueuedRequest].self, from: data) else {
                return []
            }
            return requests
        }
        set {
            let data = try? JSONEncoder().encode(newValue)
            storage.set(data, forKey: key)
        }
    }
    
    func enqueue(_ request: URLRequest) {
        var queued = pendingRequests
        
        let queuedRequest = QueuedRequest(
            id: UUID(),
            url: request.url?.absoluteString ?? "",
            method: request.httpMethod ?? "GET",
            headers: request.allHTTPHeaderFields ?? [:],
            body: request.httpBody,
            createdAt: Date(),
            retryCount: 0
        )
        
        queued.append(queuedRequest)
        pendingRequests = queued
    }
    
    func processQueue() async {
        let requests = pendingRequests
        
        for request in requests {
            do {
                try await executeQueuedRequest(request)
                removeRequest(id: request.id)
            } catch {
                incrementRetryCount(id: request.id)
                
                // ลบ requests ที่ retry เกิน 5 ครั้ง
                if let current = pendingRequests.first(where: { $0.id == request.id }),
                   current.retryCount >= 5 {
                    removeRequest(id: request.id)
                }
            }
        }
    }
    
    private func executeQueuedRequest(_ queuedRequest: QueuedRequest) async throws {
        guard let url = URL(string: queuedRequest.url) else { return }
        
        var request = URLRequest(url: url)
        request.httpMethod = queuedRequest.method
        request.httpBody = queuedRequest.body
        
        for (key, value) in queuedRequest.headers {
            request.setValue(value, forHTTPHeaderField: key)
        }
        
        let (_, response) = try await URLSession.shared.data(for: request)
        
        guard let httpResponse = response as? HTTPURLResponse,
              (200...299).contains(httpResponse.statusCode) else {
            throw NetworkError.invalidResponse
        }
    }
    
    private func removeRequest(id: UUID) {
        pendingRequests = pendingRequests.filter { $0.id != id }
    }
    
    private func incrementRetryCount(id: UUID) {
        pendingRequests = pendingRequests.map { request in
            if request.id == id {
                var updated = request
                updated.retryCount += 1
                return updated
            }
            return request
        }
    }
}

// Auto-process queue เมื่อมีการเชื่อมต่อ
class NetworkAwareService {
    
    private let networkMonitor = NetworkMonitor.shared
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        // เฝ้าดูการเปลี่ยนแปลงของ network status
        networkMonitor.$isConnected
            .removeDuplicates()
            .filter { $0 }  // เฉพาะเมื่อ connected
            .sink { [weak self] _ in
                Task {
                    await OfflineRequestQueue.shared.processQueue()
                }
            }
            .store(in: &cancellables)
    }
}
```

---

## 18. Mock Networking สำหรับ Tests

### Protocol-Based Mocking

```swift
// MockNetworkClient.swift

import Foundation

// Mock URL Protocol
class MockURLProtocol: URLProtocol {
    
    static var requestHandler: ((URLRequest) throws -> (HTTPURLResponse, Data))?
    
    override class func canInit(with request: URLRequest) -> Bool {
        return true
    }
    
    override class func canonicalRequest(for request: URLRequest) -> URLRequest {
        return request
    }
    
    override func startLoading() {
        guard let handler = MockURLProtocol.requestHandler else {
            client?.urlProtocol(self, didFailWithError: MockError.noHandler)
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
    
    enum MockError: Error {
        case noHandler
    }
}

// Mock Response Builder
struct MockResponse {
    
    static func success<T: Encodable>(
        _ data: T,
        statusCode: Int = 200
    ) -> (URLRequest) throws -> (HTTPURLResponse, Data) {
        return { request in
            let responseData = try JSONEncoder().encode(data)
            let response = HTTPURLResponse(
                url: request.url!,
                statusCode: statusCode,
                httpVersion: nil,
                headerFields: ["Content-Type": "application/json"]
            )!
            return (response, responseData)
        }
    }
    
    static func failure(
        statusCode: Int,
        message: String? = nil
    ) -> (URLRequest) throws -> (HTTPURLResponse, Data) {
        return { request in
            let errorBody: [String: Any] = [
                "error": ["code": "ERROR", "message": message ?? "Error occurred"]
            ]
            let data = try JSONSerialization.data(withJSONObject: errorBody)
            let response = HTTPURLResponse(
                url: request.url!,
                statusCode: statusCode,
                httpVersion: nil,
                headerFields: nil
            )!
            return (response, data)
        }
    }
    
    static func networkError(_ error: URLError) -> (URLRequest) throws -> (HTTPURLResponse, Data) {
        return { _ in throw error }
    }
}

// Test example
class UserServiceTests: XCTestCase {
    
    var sut: UserService!
    var session: URLSession!
    
    override func setUpWithError() throws {
        let config = URLSessionConfiguration.ephemeral
        config.protocolClasses = [MockURLProtocol.self]
        session = URLSession(configuration: config)
        sut = UserService(session: session)
    }
    
    override func tearDownWithError() throws {
        MockURLProtocol.requestHandler = nil
    }
    
    func testGetUserSuccess() async throws {
        // Given
        let expectedUser = User(id: 1, name: "John", email: "john@example.com")
        MockURLProtocol.requestHandler = MockResponse.success(expectedUser)
        
        // When
        let user = try await sut.getUser(id: 1)
        
        // Then
        XCTAssertEqual(user.id, expectedUser.id)
        XCTAssertEqual(user.name, expectedUser.name)
    }
    
    func testGetUserNotFound() async throws {
        // Given
        MockURLProtocol.requestHandler = MockResponse.failure(statusCode: 404)
        
        // When/Then
        do {
            _ = try await sut.getUser(id: 999)
            XCTFail("Expected error to be thrown")
        } catch let error as NetworkError {
            XCTAssertEqual(error, .unexpectedStatusCode(404))
        }
    }
    
    func testGetUserNetworkError() async throws {
        // Given
        let networkError = URLError(.notConnectedToInternet)
        MockURLProtocol.requestHandler = MockResponse.networkError(networkError)
        
        // When/Then
        do {
            _ = try await sut.getUser(id: 1)
            XCTFail("Expected error to be thrown")
        } catch let error as NetworkError {
            XCTAssertEqual(error, .noInternetConnection)
        }
    }
}
```

---

## 19. บทฝึกหัดปฏิบัติ

### Exercise 1: สร้าง Production-Ready Network Layer

```swift
// ProductionNetworkLayer.swift
// Complete implementation

import Foundation
import Network

// MARK: - Configuration

struct NetworkConfiguration {
    let baseURL: URL
    let defaultTimeout: TimeInterval
    let maxRetryAttempts: Int
    let enableLogging: Bool
    
    static let production = NetworkConfiguration(
        baseURL: URL(string: "https://api.example.com/v2")!,
        defaultTimeout: 30,
        maxRetryAttempts: 3,
        enableLogging: false
    )
    
    static let development = NetworkConfiguration(
        baseURL: URL(string: "https://dev-api.example.com/v2")!,
        defaultTimeout: 60,
        maxRetryAttempts: 1,
        enableLogging: true
    )
}

// MARK: - Endpoint

struct Endpoint {
    let path: String
    let method: HTTPMethod
    var queryParameters: [String: String]?
    var body: Encodable?
    var customHeaders: [String: String]?
    var requiresAuth: Bool
    var timeout: TimeInterval?
    
    init(
        path: String,
        method: HTTPMethod = .get,
        queryParameters: [String: String]? = nil,
        body: Encodable? = nil,
        customHeaders: [String: String]? = nil,
        requiresAuth: Bool = true,
        timeout: TimeInterval? = nil
    ) {
        self.path = path
        self.method = method
        self.queryParameters = queryParameters
        self.body = body
        self.customHeaders = customHeaders
        self.requiresAuth = requiresAuth
        self.timeout = timeout
    }
}

// MARK: - Network Client

class ProductionNetworkClient {
    
    private let configuration: NetworkConfiguration
    private let session: URLSession
    private let tokenStorage: TokenStorage
    private let tokenRefreshManager: TokenRefreshManager
    private let networkMonitor: NetworkMonitor
    private let retryPolicy: RetryPolicy
    
    init(
        configuration: NetworkConfiguration = .production,
        tokenStorage: TokenStorage = .shared,
        networkMonitor: NetworkMonitor = .shared
    ) {
        self.configuration = configuration
        self.tokenStorage = tokenStorage
        self.networkMonitor = networkMonitor
        
        let sessionConfig = URLSessionConfiguration.default
        sessionConfig.timeoutIntervalForRequest = configuration.defaultTimeout
        self.session = URLSession(configuration: sessionConfig)
        
        let authService = AuthService()
        self.tokenRefreshManager = TokenRefreshManager(
            tokenStorage: tokenStorage,
            authService: authService
        )
        
        self.retryPolicy = RetryPolicy(
            maxAttempts: configuration.maxRetryAttempts,
            backoffStrategy: .exponential(initial: 1.0, multiplier: 2.0, maxDelay: 30.0),
            retryableErrors: { $0.isRetryable }
        )
    }
    
    func request<T: Decodable>(
        _ endpoint: Endpoint,
        responseType: T.Type = T.self
    ) async throws -> T {
        
        // ตรวจสอบ network connection
        guard networkMonitor.isConnected else {
            throw NetworkError.noInternetConnection
        }
        
        return try await RetryExecutor(policy: retryPolicy).execute {
            // Build request
            var urlRequest = try self.buildRequest(from: endpoint)
            
            // Add auth header ถ้าจำเป็น
            if endpoint.requiresAuth {
                let token = try await self.tokenRefreshManager.validAccessToken()
                urlRequest.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
            }
            
            // Log request
            if self.configuration.enableLogging {
                self.logRequest(urlRequest)
            }
            
            // Execute
            let (data, response) = try await self.session.data(for: urlRequest)
            
            guard let httpResponse = response as? HTTPURLResponse else {
                throw NetworkError.invalidResponse
            }
            
            // Log response
            if self.configuration.enableLogging {
                self.logResponse(httpResponse, data: data)
            }
            
            // Handle errors
            try NetworkErrorHandler.handle(response: httpResponse, data: data)
            
            // Decode
            do {
                return try JSONDecoder.apiDecoder.decode(T.self, from: data)
            } catch {
                throw NetworkError.decodingFailed(error)
            }
        }
    }
    
    private func buildRequest(from endpoint: Endpoint) throws -> URLRequest {
        var components = URLComponents(
            url: configuration.baseURL.appendingPathComponent(endpoint.path),
            resolvingAgainstBaseURL: false
        )
        
        if let params = endpoint.queryParameters {
            components?.queryItems = params.map {
                URLQueryItem(name: $0.key, value: $0.value)
            }
        }
        
        guard let url = components?.url else {
            throw NetworkError.invalidURL
        }
        
        var request = URLRequest(
            url: url,
            timeoutInterval: endpoint.timeout ?? configuration.defaultTimeout
        )
        request.httpMethod = endpoint.method.rawValue
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        request.setValue("application/json", forHTTPHeaderField: "Accept")
        
        if let body = endpoint.body {
            request.httpBody = try JSONEncoder().encode(body)
        }
        
        if let customHeaders = endpoint.customHeaders {
            for (key, value) in customHeaders {
                request.setValue(value, forHTTPHeaderField: key)
            }
        }
        
        return request
    }
    
    private func logRequest(_ request: URLRequest) {
        print("""
        ───────────────────────────────────
        ➡️ REQUEST
        URL: \(request.url?.absoluteString ?? "")
        Method: \(request.httpMethod ?? "")
        ───────────────────────────────────
        """)
    }
    
    private func logResponse(_ response: HTTPURLResponse, data: Data) {
        print("""
        ───────────────────────────────────
        ⬅️ RESPONSE
        Status: \(response.statusCode)
        ───────────────────────────────────
        """)
    }
}

// MARK: - Service Layer

class UserAPIService {
    
    private let client: ProductionNetworkClient
    
    init(client: ProductionNetworkClient = ProductionNetworkClient()) {
        self.client = client
    }
    
    func getUsers(page: Int = 1, perPage: Int = 20) async throws -> PaginatedResponse<User> {
        let endpoint = Endpoint(
            path: "/users",
            method: .get,
            queryParameters: [
                "page": "\(page)",
                "per_page": "\(perPage)"
            ]
        )
        return try await client.request(endpoint)
    }
    
    func getUser(id: Int) async throws -> User {
        let endpoint = Endpoint(path: "/users/\(id)")
        return try await client.request(endpoint)
    }
    
    func createUser(_ request: CreateUserRequest) async throws -> User {
        let endpoint = Endpoint(
            path: "/users",
            method: .post,
            body: request
        )
        return try await client.request(endpoint)
    }
    
    func updateUser(id: Int, request: UpdateUserRequest) async throws -> User {
        let endpoint = Endpoint(
            path: "/users/\(id)",
            method: .put,
            body: request
        )
        return try await client.request(endpoint)
    }
    
    func deleteUser(id: Int) async throws {
        let endpoint = Endpoint(
            path: "/users/\(id)",
            method: .delete
        )
        let _: EmptyResponse = try await client.request(endpoint)
    }
}

struct EmptyResponse: Decodable {}

// MARK: - Models

struct User: Codable, Identifiable {
    let id: Int
    let name: String
    let email: String
    let avatar: String?
    let createdAt: Date?
    
    enum CodingKeys: String, CodingKey {
        case id, name, email, avatar
        case createdAt = "created_at"
    }
}

struct CreateUserRequest: Encodable {
    let name: String
    let email: String
    let password: String
}

struct UpdateUserRequest: Encodable {
    let name: String?
    let email: String?
}
```

### Exercise 2: WebSocket Chat App

```swift
// ChatApp.swift

import SwiftUI

@MainActor
class ChatViewModel: ObservableObject {
    
    @Published var messages: [Message] = []
    @Published var inputText: String = ""
    @Published var connectionStatus: ConnectionStatus = .disconnected
    @Published var error: String?
    
    private let chatService: ChatService
    private let userId: String
    
    enum ConnectionStatus {
        case connected, disconnected, connecting
        
        var displayText: String {
            switch self {
            case .connected: return "เชื่อมต่อแล้ว"
            case .disconnected: return "ไม่ได้เชื่อมต่อ"
            case .connecting: return "กำลังเชื่อมต่อ..."
            }
        }
        
        var color: Color {
            switch self {
            case .connected: return .green
            case .disconnected: return .red
            case .connecting: return .orange
            }
        }
    }
    
    struct Message: Identifiable {
        let id = UUID()
        let userId: String
        let content: String
        let timestamp: Date
        let isFromCurrentUser: Bool
    }
    
    init(userId: String = "user123") {
        self.userId = userId
        self.chatService = ChatService()
        
        setupChatService()
    }
    
    private func setupChatService() {
        chatService.onMessageReceived = { [weak self] chatMessage in
            guard let self = self else { return }
            let message = Message(
                userId: chatMessage.userId,
                content: chatMessage.content,
                timestamp: chatMessage.timestamp,
                isFromCurrentUser: chatMessage.userId == self.userId
            )
            self.messages.append(message)
        }
    }
    
    func connect(roomId: String) {
        connectionStatus = .connecting
        chatService.connect(roomId: roomId, token: "user_token")
        connectionStatus = .connected
    }
    
    func sendMessage() async {
        guard !inputText.isEmpty else { return }
        
        let text = inputText
        inputText = ""
        
        do {
            try await chatService.sendMessage(text, userId: userId)
        } catch {
            self.error = error.localizedDescription
        }
    }
}

struct ChatView: View {
    
    @StateObject private var viewModel = ChatViewModel()
    
    var body: some View {
        VStack(spacing: 0) {
            // Status bar
            HStack {
                Circle()
                    .fill(viewModel.connectionStatus.color)
                    .frame(width: 8, height: 8)
                Text(viewModel.connectionStatus.displayText)
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
            .padding(.vertical, 8)
            
            // Messages
            ScrollViewReader { proxy in
                ScrollView {
                    LazyVStack(spacing: 8) {
                        ForEach(viewModel.messages) { message in
                            MessageBubble(message: message)
                                .id(message.id)
                        }
                    }
                    .padding()
                }
                .onChange(of: viewModel.messages.count) { _ in
                    if let lastMessage = viewModel.messages.last {
                        withAnimation {
                            proxy.scrollTo(lastMessage.id, anchor: .bottom)
                        }
                    }
                }
            }
            
            // Input
            HStack(spacing: 12) {
                TextField("พิมพ์ข้อความ...", text: $viewModel.inputText)
                    .textFieldStyle(.roundedBorder)
                
                Button {
                    Task { await viewModel.sendMessage() }
                } label: {
                    Image(systemName: "paperplane.fill")
                        .foregroundColor(.blue)
                }
                .disabled(viewModel.inputText.isEmpty)
            }
            .padding()
        }
        .onAppear {
            viewModel.connect(roomId: "room123")
        }
    }
}

struct MessageBubble: View {
    let message: ChatViewModel.Message
    
    var body: some View {
        HStack {
            if message.isFromCurrentUser { Spacer() }
            
            VStack(alignment: message.isFromCurrentUser ? .trailing : .leading, spacing: 4) {
                if !message.isFromCurrentUser {
                    Text(message.userId)
                        .font(.caption)
                        .foregroundColor(.secondary)
                }
                
                Text(message.content)
                    .padding(10)
                    .background(message.isFromCurrentUser ? Color.blue : Color.gray.opacity(0.2))
                    .foregroundColor(message.isFromCurrentUser ? .white : .primary)
                    .cornerRadius(16)
                
                Text(message.timestamp, style: .time)
                    .font(.caption2)
                    .foregroundColor(.secondary)
            }
            
            if !message.isFromCurrentUser { Spacer() }
        }
    }
}
```

---

## สรุป

การสร้าง network layer ที่ดีสำหรับ production application จำเป็นต้องคำนึงถึงหลายปัจจัย:

### Checklist สำหรับ Production Network Layer

```swift
// ✅ Authentication
// - JWT token management
// - OAuth2 with PKCE
// - Automatic token refresh
// - Secure storage ใน Keychain

// ✅ Error Handling
// - Comprehensive error types
// - User-friendly messages
// - Proper error propagation

// ✅ Resilience
// - Retry logic with exponential backoff
// - Jitter เพื่อป้องกัน thundering herd
// - Circuit breaker pattern

// ✅ Performance
// - Request deduplication
// - Caching strategy
// - Connection pooling

// ✅ Offline Support
// - Network monitoring
// - Request queuing
// - Sync when reconnected

// ✅ Testing
// - Mock URL Protocol
// - Protocol-based design
// - Unit tests ครบถ้วน

// ✅ Observability
// - Request/Response logging
// - Error tracking
// - Performance monitoring
```

### แนะนำ Libraries

```
Networking:
- Alamofire: https://github.com/Alamofire/Alamofire
- Moya: https://github.com/Moya/Moya

GraphQL:
- Apollo iOS: https://github.com/apollographql/apollo-ios

WebSocket:
- Starscream: https://github.com/daltoniam/Starscream

Certificate Pinning:
- TrustKit: https://github.com/datatheorem/TrustKit
```

### แหล่งข้อมูลเพิ่มเติม

- [Apple URLSession Documentation](https://developer.apple.com/documentation/foundation/urlsession)
- [WWDC: Advances in Networking](https://developer.apple.com/videos/networking/)
- [RFC 7617 - HTTP Basic Authentication](https://tools.ietf.org/html/rfc7617)
- [RFC 6750 - Bearer Token](https://tools.ietf.org/html/rfc6750)
- [OAuth 2.0 with PKCE](https://oauth.net/2/pkce/)

---

*จบ Part 48: Advanced Networking*
