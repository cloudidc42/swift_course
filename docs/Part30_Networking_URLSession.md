# Part 30: Networking กับ URLSession

## บทนำ

Networking เป็นส่วนสำคัญของแอปพลิเคชันสมัยใหม่ iOS ใช้ `URLSession` เป็น framework หลักสำหรับการสื่อสารกับเครือข่าย ในบทนี้เราจะเรียนรู้การใช้งาน URLSession ตั้งแต่พื้นฐานจนถึงการใช้งานขั้นสูง

---

## 1. URLSession Basics

### ความรู้เบื้องต้น

`URLSession` เป็น class ที่จัดการ HTTP/HTTPS requests ทั้งหมด มีลักษณะสำคัญ:

```swift
import Foundation

// URLSession shared instance - ใช้สำหรับ simple requests
let sharedSession = URLSession.shared

// สร้าง custom session
let configuration = URLSessionConfiguration.default
let customSession = URLSession(configuration: configuration)

// Session พร้อม delegate
let delegateSession = URLSession(
    configuration: .default,
    delegate: myDelegate,
    delegateQueue: .main
)
```

### Request แบบง่ายที่สุด

```swift
// GET request พื้นฐาน
func fetchSimpleData() async throws -> Data {
    let url = URL(string: "https://api.example.com/data")!
    let (data, _) = try await URLSession.shared.data(from: url)
    return data
}

// พร้อม error handling
func fetchData(from urlString: String) async throws -> Data {
    guard let url = URL(string: urlString) else {
        throw URLError(.badURL)
    }
    
    let (data, response) = try await URLSession.shared.data(from: url)
    
    guard let httpResponse = response as? HTTPURLResponse else {
        throw URLError(.badServerResponse)
    }
    
    guard (200...299).contains(httpResponse.statusCode) else {
        throw URLError(.init(rawValue: httpResponse.statusCode))
    }
    
    return data
}
```

---

## 2. URLRequest

`URLRequest` ใช้สำหรับกำหนดรายละเอียดของ HTTP request

### สร้าง URLRequest

```swift
// URLRequest พื้นฐาน
var request = URLRequest(url: URL(string: "https://api.example.com/users")!)

// กำหนด HTTP method
request.httpMethod = "POST"

// กำหนด timeout
request.timeoutInterval = 30.0

// กำหนด cache policy
request.cachePolicy = .reloadIgnoringLocalCacheData
```

### Cache Policies

```swift
// Cache policies ต่างๆ
// .useProtocolCachePolicy - ใช้ policy จาก HTTP headers (ค่า default)
request.cachePolicy = .useProtocolCachePolicy

// .reloadIgnoringLocalCacheData - ดาวน์โหลดจาก server เสมอ ไม่ใช้ cache
request.cachePolicy = .reloadIgnoringLocalCacheData

// .returnCacheDataElseLoad - ใช้ cache ถ้ามี ถ้าไม่มีค่อย load
request.cachePolicy = .returnCacheDataElseLoad

// .returnCacheDataDontLoad - ใช้ cache เท่านั้น ถ้าไม่มี cache = error
request.cachePolicy = .returnCacheDataDontLoad
```

---

## 3. HTTP Methods

### GET Request

```swift
// GET - ดึงข้อมูล
func getUser(id: Int) async throws -> User {
    let url = URL(string: "https://api.example.com/users/\(id)")!
    var request = URLRequest(url: url)
    request.httpMethod = "GET"
    
    let (data, response) = try await URLSession.shared.data(for: request)
    try validateResponse(response)
    
    return try JSONDecoder().decode(User.self, from: data)
}

// GET กับ Query Parameters
func searchUsers(query: String, page: Int = 1) async throws -> [User] {
    var components = URLComponents(string: "https://api.example.com/users/search")!
    components.queryItems = [
        URLQueryItem(name: "q", value: query),
        URLQueryItem(name: "page", value: String(page)),
        URLQueryItem(name: "limit", value: "20")
    ]
    
    guard let url = components.url else {
        throw URLError(.badURL)
    }
    
    let (data, response) = try await URLSession.shared.data(from: url)
    try validateResponse(response)
    
    return try JSONDecoder().decode([User].self, from: data)
}
```

### POST Request

```swift
// POST - สร้างข้อมูลใหม่
struct CreateUserRequest: Encodable {
    let name: String
    let email: String
    let age: Int
}

func createUser(_ userData: CreateUserRequest) async throws -> User {
    let url = URL(string: "https://api.example.com/users")!
    var request = URLRequest(url: url)
    request.httpMethod = "POST"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    
    // Encode body
    request.httpBody = try JSONEncoder().encode(userData)
    
    let (data, response) = try await URLSession.shared.data(for: request)
    try validateResponse(response)
    
    return try JSONDecoder().decode(User.self, from: data)
}
```

### PUT Request

```swift
// PUT - อัปเดตข้อมูลทั้งหมด
struct UpdateUserRequest: Encodable {
    let name: String
    let email: String
    let age: Int
}

func updateUser(id: Int, with data: UpdateUserRequest) async throws -> User {
    let url = URL(string: "https://api.example.com/users/\(id)")!
    var request = URLRequest(url: url)
    request.httpMethod = "PUT"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    request.httpBody = try JSONEncoder().encode(data)
    
    let (responseData, response) = try await URLSession.shared.data(for: request)
    try validateResponse(response)
    
    return try JSONDecoder().decode(User.self, from: responseData)
}
```

### PATCH Request

```swift
// PATCH - อัปเดตข้อมูลบางส่วน
struct PatchUserRequest: Encodable {
    var name: String?
    var email: String?
    var age: Int?
    
    // เข้ารหัสเฉพาะ field ที่ไม่ nil
    private enum CodingKeys: String, CodingKey {
        case name, email, age
    }
    
    func encode(to encoder: Encoder) throws {
        var container = encoder.container(keyedBy: CodingKeys.self)
        if let name = name { try container.encode(name, forKey: .name) }
        if let email = email { try container.encode(email, forKey: .email) }
        if let age = age { try container.encode(age, forKey: .age) }
    }
}

func patchUser(id: Int, with data: PatchUserRequest) async throws -> User {
    let url = URL(string: "https://api.example.com/users/\(id)")!
    var request = URLRequest(url: url)
    request.httpMethod = "PATCH"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    request.httpBody = try JSONEncoder().encode(data)
    
    let (responseData, response) = try await URLSession.shared.data(for: request)
    try validateResponse(response)
    
    return try JSONDecoder().decode(User.self, from: responseData)
}
```

### DELETE Request

```swift
// DELETE - ลบข้อมูล
func deleteUser(id: Int) async throws {
    let url = URL(string: "https://api.example.com/users/\(id)")!
    var request = URLRequest(url: url)
    request.httpMethod = "DELETE"
    
    let (_, response) = try await URLSession.shared.data(for: request)
    try validateResponse(response)
    // 204 No Content - ไม่มี body กลับมา
}

// Helper function สำหรับ validate response
func validateResponse(_ response: URLResponse) throws {
    guard let httpResponse = response as? HTTPURLResponse else {
        throw NetworkError.invalidResponse
    }
    
    switch httpResponse.statusCode {
    case 200...299:
        return // success
    case 400:
        throw NetworkError.badRequest
    case 401:
        throw NetworkError.unauthorized
    case 403:
        throw NetworkError.forbidden
    case 404:
        throw NetworkError.notFound
    case 429:
        throw NetworkError.rateLimited
    case 500...599:
        throw NetworkError.serverError(httpResponse.statusCode)
    default:
        throw NetworkError.httpError(httpResponse.statusCode)
    }
}
```

---

## 4. Request Headers

### การตั้งค่า Headers

```swift
// กำหนด headers ต่างๆ
var request = URLRequest(url: url)

// Content-Type
request.setValue("application/json", forHTTPHeaderField: "Content-Type")

// Accept
request.setValue("application/json", forHTTPHeaderField: "Accept")

// Authorization
request.setValue("Bearer eyJhbGciOiJIUzI1NiJ9...", forHTTPHeaderField: "Authorization")

// Custom headers
request.setValue("my-app/1.0", forHTTPHeaderField: "User-Agent")
request.setValue("th", forHTTPHeaderField: "Accept-Language")
request.setValue("gzip", forHTTPHeaderField: "Accept-Encoding")

// addValue เพิ่ม value แทนที่จะแทนที่
request.addValue("application/json", forHTTPHeaderField: "Accept")
request.addValue("text/plain", forHTTPHeaderField: "Accept")
// ผลลัพธ์: Accept: application/json, text/plain
```

### Header Manager

```swift
// จัดการ headers แบบ reusable
struct HTTPHeaders {
    private var headers: [String: String] = [:]
    
    mutating func set(_ value: String, forKey key: String) {
        headers[key] = value
    }
    
    mutating func add(_ value: String, forKey key: String) {
        if let existing = headers[key] {
            headers[key] = "\(existing), \(value)"
        } else {
            headers[key] = value
        }
    }
    
    func apply(to request: inout URLRequest) {
        for (key, value) in headers {
            request.setValue(value, forHTTPHeaderField: key)
        }
    }
}

// การใช้งาน
var headers = HTTPHeaders()
headers.set("application/json", forKey: "Content-Type")
headers.set("Bearer token123", forKey: "Authorization")

var request = URLRequest(url: url)
headers.apply(to: &request)
```

---

## 5. Request Body

### JSON Body

```swift
// ส่ง JSON body
struct LoginRequest: Encodable {
    let email: String
    let password: String
}

func login(email: String, password: String) async throws -> AuthToken {
    let url = URL(string: "https://api.example.com/auth/login")!
    var request = URLRequest(url: url)
    request.httpMethod = "POST"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    
    let body = LoginRequest(email: email, password: password)
    request.httpBody = try JSONEncoder().encode(body)
    
    let (data, response) = try await URLSession.shared.data(for: request)
    try validateResponse(response)
    
    return try JSONDecoder().decode(AuthToken.self, from: data)
}
```

### Form URL Encoded Body

```swift
// ส่ง application/x-www-form-urlencoded
func submitForm(name: String, email: String) async throws {
    let url = URL(string: "https://example.com/contact")!
    var request = URLRequest(url: url)
    request.httpMethod = "POST"
    request.setValue("application/x-www-form-urlencoded", forHTTPHeaderField: "Content-Type")
    
    let params = ["name": name, "email": email]
    let bodyString = params.map { key, value in
        let encodedKey = key.addingPercentEncoding(withAllowedCharacters: .urlQueryAllowed) ?? key
        let encodedValue = value.addingPercentEncoding(withAllowedCharacters: .urlQueryAllowed) ?? value
        return "\(encodedKey)=\(encodedValue)"
    }.joined(separator: "&")
    
    request.httpBody = bodyString.data(using: .utf8)
    
    let (_, response) = try await URLSession.shared.data(for: request)
    try validateResponse(response)
}
```

---

## 6. URLSessionConfiguration

```swift
// default - ใช้ disk-based cache และ cookies
let defaultConfig = URLSessionConfiguration.default

// ephemeral - ไม่มี cache, cookies, หรือ credentials บน disk
let ephemeralConfig = URLSessionConfiguration.ephemeral

// background - สำหรับ background transfers
let backgroundConfig = URLSessionConfiguration.background(withIdentifier: "com.myapp.background")

// Custom configuration
let customConfig = URLSessionConfiguration.default
customConfig.timeoutIntervalForRequest = 30       // timeout สำหรับแต่ละ request
customConfig.timeoutIntervalForResource = 300     // timeout สำหรับทั้งหมด
customConfig.httpMaximumConnectionsPerHost = 6    // max connections ต่อ host
customConfig.requestCachePolicy = .reloadIgnoringLocalCacheData
customConfig.allowsCellularAccess = true          // อนุญาต cellular
customConfig.waitsForConnectivity = true          // รอ connectivity

// HTTP Additional Headers
customConfig.httpAdditionalHeaders = [
    "User-Agent": "MyApp/1.0 iOS/\(UIDevice.current.systemVersion)",
    "Accept": "application/json"
]

let session = URLSession(configuration: customConfig)
```

---

## 7. dataTask (Completion Handler Style)

แม้ว่า async/await จะเป็นวิธีที่แนะนำ แต่บางครั้งต้องทำงานกับ completion handler style เดิม

```swift
// dataTask แบบเดิม
class LegacyNetworkService {
    private let session = URLSession.shared
    
    func fetchUser(id: Int, completion: @escaping (Result<User, Error>) -> Void) {
        let url = URL(string: "https://api.example.com/users/\(id)")!
        
        session.dataTask(with: url) { data, response, error in
            // ต้องกลับไปยัง main thread เอง
            DispatchQueue.main.async {
                if let error = error {
                    completion(.failure(error))
                    return
                }
                
                guard let data = data else {
                    completion(.failure(NetworkError.noData))
                    return
                }
                
                guard let httpResponse = response as? HTTPURLResponse,
                      (200...299).contains(httpResponse.statusCode) else {
                    completion(.failure(NetworkError.badResponse))
                    return
                }
                
                do {
                    let user = try JSONDecoder().decode(User.self, from: data)
                    completion(.success(user))
                } catch {
                    completion(.failure(error))
                }
            }
        }.resume()
    }
}

// แปลง dataTask เป็น async
extension URLSession {
    func fetchJSON<T: Decodable>(_ type: T.Type, from url: URL) async throws -> T {
        let (data, response) = try await data(from: url)
        
        guard let httpResponse = response as? HTTPURLResponse,
              (200...299).contains(httpResponse.statusCode) else {
            throw NetworkError.badResponse
        }
        
        return try JSONDecoder().decode(T.self, from: data)
    }
}
```

---

## 8. URLSession กับ async/await

```swift
// Modern async/await approach
class NetworkService {
    private let session: URLSession
    private let decoder: JSONDecoder
    
    init(session: URLSession = .shared) {
        self.session = session
        self.decoder = JSONDecoder()
        self.decoder.keyDecodingStrategy = .convertFromSnakeCase
        self.decoder.dateDecodingStrategy = .iso8601
    }
    
    func request<T: Decodable>(_ type: T.Type, endpoint: Endpoint) async throws -> T {
        var request = try endpoint.makeRequest()
        
        let (data, response) = try await session.data(for: request)
        
        guard let httpResponse = response as? HTTPURLResponse else {
            throw NetworkError.invalidResponse
        }
        
        guard (200...299).contains(httpResponse.statusCode) else {
            throw NetworkError.httpError(httpResponse.statusCode)
        }
        
        return try decoder.decode(T.self, from: data)
    }
}

// Endpoint definition
struct Endpoint {
    let path: String
    let method: HTTPMethod
    var queryItems: [URLQueryItem]?
    var headers: [String: String]?
    var body: Encodable?
    
    static let baseURL = "https://api.example.com"
    
    func makeRequest() throws -> URLRequest {
        var components = URLComponents(string: Self.baseURL + path)!
        
        if let queryItems = queryItems {
            components.queryItems = queryItems
        }
        
        guard let url = components.url else {
            throw URLError(.badURL)
        }
        
        var request = URLRequest(url: url)
        request.httpMethod = method.rawValue
        
        // Set default headers
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        request.setValue("application/json", forHTTPHeaderField: "Accept")
        
        // Set custom headers
        headers?.forEach { key, value in
            request.setValue(value, forHTTPHeaderField: key)
        }
        
        // Set body
        if let body = body {
            request.httpBody = try JSONEncoder().encode(body)
        }
        
        return request
    }
}

enum HTTPMethod: String {
    case get = "GET"
    case post = "POST"
    case put = "PUT"
    case patch = "PATCH"
    case delete = "DELETE"
}
```

---

## 9. Uploading Data

### อัปโหลดข้อมูล JSON

```swift
func uploadJSON<T: Encodable>(_ data: T, to url: URL) async throws -> Data {
    var request = URLRequest(url: url)
    request.httpMethod = "POST"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    
    let body = try JSONEncoder().encode(data)
    
    let (responseData, response) = try await URLSession.shared.upload(for: request, from: body)
    try validateResponse(response)
    
    return responseData
}
```

### อัปโหลดไฟล์

```swift
func uploadFile(fileURL: URL, to uploadURL: URL) async throws -> UploadResponse {
    var request = URLRequest(url: uploadURL)
    request.httpMethod = "POST"
    
    let (responseData, response) = try await URLSession.shared.upload(for: request, fromFile: fileURL)
    try validateResponse(response)
    
    return try JSONDecoder().decode(UploadResponse.self, from: responseData)
}
```

### อัปโหลดพร้อม Progress

```swift
class UploadManager: NSObject {
    private var session: URLSession!
    private var progressHandlers: [URLSessionTask: (Double) -> Void] = [:]
    private var completionHandlers: [URLSessionTask: (Result<Data, Error>) -> Void] = [:]
    
    override init() {
        super.init()
        let config = URLSessionConfiguration.default
        session = URLSession(configuration: config, delegate: self, delegateQueue: nil)
    }
    
    func upload(
        data: Data,
        to url: URL,
        progress: @escaping (Double) -> Void,
        completion: @escaping (Result<Data, Error>) -> Void
    ) {
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        
        let task = session.uploadTask(with: request, from: data)
        progressHandlers[task] = progress
        completionHandlers[task] = completion
        task.resume()
    }
}

extension UploadManager: URLSessionTaskDelegate {
    func urlSession(
        _ session: URLSession,
        task: URLSessionTask,
        didSendBodyData bytesSent: Int64,
        totalBytesSent: Int64,
        totalBytesExpectedToSend: Int64
    ) {
        let percentage = Double(totalBytesSent) / Double(totalBytesExpectedToSend)
        DispatchQueue.main.async {
            self.progressHandlers[task]?(percentage)
        }
    }
}
```

---

## 10. Downloading Files

### Download พื้นฐาน

```swift
func downloadFile(from url: URL, to destinationURL: URL) async throws {
    let (tempURL, response) = try await URLSession.shared.download(from: url)
    try validateResponse(response)
    
    // ย้ายไฟล์จาก temp location ไปยัง destination
    try FileManager.default.moveItem(at: tempURL, to: destinationURL)
}
```

### Download พร้อม Progress Tracking

```swift
func downloadWithProgress(from url: URL) -> (task: URLSessionDownloadTask, stream: AsyncStream<Double>) {
    var continuation: AsyncStream<Double>.Continuation?
    
    let stream = AsyncStream<Double> { cont in
        continuation = cont
    }
    
    let progressHandler: (Double) -> Void = { progress in
        continuation?.yield(progress)
    }
    
    let task = URLSession.shared.downloadTask(with: url) { tempURL, response, error in
        guard let tempURL = tempURL else { return }
        
        let documentsURL = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first!
        let destinationURL = documentsURL.appendingPathComponent(url.lastPathComponent)
        
        try? FileManager.default.moveItem(at: tempURL, to: destinationURL)
        continuation?.finish()
    }
    
    return (task, stream)
}
```

---

## 11. Background Downloads

### Setup Background Session

```swift
// AppDelegate.swift
class AppDelegate: UIApplicationDelegate {
    func application(
        _ application: UIApplication,
        handleEventsForBackgroundURLSession identifier: String,
        completionHandler: @escaping () -> Void
    ) {
        BackgroundDownloadManager.shared.handleEventsForBackgroundURLSession(
            identifier: identifier,
            completionHandler: completionHandler
        )
    }
}

// BackgroundDownloadManager.swift
class BackgroundDownloadManager: NSObject {
    static let shared = BackgroundDownloadManager()
    
    private let backgroundSessionIdentifier = "com.myapp.background.download"
    private var backgroundCompletionHandler: (() -> Void)?
    private lazy var backgroundSession: URLSession = {
        let config = URLSessionConfiguration.background(withIdentifier: backgroundSessionIdentifier)
        config.isDiscretionary = false        // ดาวน์โหลดทันที ไม่รอเวลาที่เหมาะสม
        config.sessionSendsLaunchEvents = true // เปิดแอปเมื่อ download เสร็จ
        return URLSession(configuration: config, delegate: self, delegateQueue: nil)
    }()
    
    func startBackgroundDownload(from url: URL) {
        let task = backgroundSession.downloadTask(with: url)
        task.resume()
        print("เริ่ม background download: \(url)")
    }
    
    func handleEventsForBackgroundURLSession(identifier: String, completionHandler: @escaping () -> Void) {
        if identifier == backgroundSessionIdentifier {
            backgroundCompletionHandler = completionHandler
        }
    }
}

extension BackgroundDownloadManager: URLSessionDownloadDelegate {
    func urlSession(
        _ session: URLSession,
        downloadTask: URLSessionDownloadTask,
        didFinishDownloadingTo location: URL
    ) {
        let documentsURL = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first!
        let fileName = downloadTask.originalRequest?.url?.lastPathComponent ?? "downloaded_file"
        let destinationURL = documentsURL.appendingPathComponent(fileName)
        
        do {
            try FileManager.default.moveItem(at: location, to: destinationURL)
            print("ดาวน์โหลดเสร็จ: \(destinationURL)")
        } catch {
            print("Error saving file: \(error)")
        }
    }
    
    func urlSessionDidFinishEvents(forBackgroundURLSession session: URLSession) {
        DispatchQueue.main.async {
            self.backgroundCompletionHandler?()
            self.backgroundCompletionHandler = nil
        }
    }
}
```

---

## 12. URLSessionDelegate

### Complete Delegate Implementation

```swift
class NetworkSessionManager: NSObject {
    private var session: URLSession!
    
    override init() {
        super.init()
        let config = URLSessionConfiguration.default
        session = URLSession(configuration: config, delegate: self, delegateQueue: nil)
    }
}

// MARK: - URLSessionDelegate
extension NetworkSessionManager: URLSessionDelegate {
    // Session ระดับ events
    func urlSession(_ session: URLSession, didBecomeInvalidWithError error: Error?) {
        print("Session invalid: \(error?.localizedDescription ?? "no error")")
    }
    
    func urlSessionDidFinishEvents(forBackgroundURLSession session: URLSession) {
        print("Background session finished")
    }
}

// MARK: - URLSessionTaskDelegate
extension NetworkSessionManager: URLSessionTaskDelegate {
    func urlSession(
        _ session: URLSession,
        task: URLSessionTask,
        didCompleteWithError error: Error?
    ) {
        if let error = error {
            print("Task failed: \(error)")
        } else {
            print("Task completed successfully")
        }
    }
    
    func urlSession(
        _ session: URLSession,
        task: URLSessionTask,
        willPerformHTTPRedirection response: HTTPURLResponse,
        newRequest request: URLRequest,
        completionHandler: @escaping (URLRequest?) -> Void
    ) {
        // อนุญาต redirect
        completionHandler(request)
        
        // หรือบล็อก redirect
        // completionHandler(nil)
    }
    
    func urlSession(
        _ session: URLSession,
        task: URLSessionTask,
        didSendBodyData bytesSent: Int64,
        totalBytesSent: Int64,
        totalBytesExpectedToSend: Int64
    ) {
        let progress = Double(totalBytesSent) / Double(totalBytesExpectedToSend)
        print("Upload progress: \(Int(progress * 100))%")
    }
}

// MARK: - URLSessionDataDelegate
extension NetworkSessionManager: URLSessionDataDelegate {
    func urlSession(
        _ session: URLSession,
        dataTask: URLSessionDataTask,
        didReceive response: URLResponse,
        completionHandler: @escaping (URLSession.ResponseDisposition) -> Void
    ) {
        guard let httpResponse = response as? HTTPURLResponse,
              (200...299).contains(httpResponse.statusCode) else {
            completionHandler(.cancel)
            return
        }
        completionHandler(.allow)
    }
    
    func urlSession(
        _ session: URLSession,
        dataTask: URLSessionDataTask,
        didReceive data: Data
    ) {
        // รับข้อมูล incrementally
        print("Received \(data.count) bytes")
    }
}
```

---

## 13. HTTP Authentication

### Basic Authentication

```swift
// Basic Auth - ส่ง username:password encoded เป็น Base64
func makeBasicAuthRequest(username: String, password: String, url: URL) -> URLRequest {
    var request = URLRequest(url: url)
    
    let credentials = "\(username):\(password)"
    let encodedCredentials = Data(credentials.utf8).base64EncodedString()
    request.setValue("Basic \(encodedCredentials)", forHTTPHeaderField: "Authorization")
    
    return request
}
```

### Bearer Token Authentication

```swift
// Token-based authentication
class AuthenticatedNetworkService {
    private var authToken: String?
    private let session: URLSession
    
    init() {
        self.session = URLSession.shared
    }
    
    func setToken(_ token: String) {
        authToken = token
    }
    
    func makeAuthenticatedRequest(url: URL) -> URLRequest {
        var request = URLRequest(url: url)
        
        if let token = authToken {
            request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        }
        
        return request
    }
    
    func fetchProtectedResource() async throws -> Data {
        guard authToken != nil else {
            throw AuthError.notAuthenticated
        }
        
        let request = makeAuthenticatedRequest(url: URL(string: "https://api.example.com/protected")!)
        let (data, response) = try await session.data(for: request)
        try validateResponse(response)
        
        return data
    }
}

// Token Refresh Pattern
actor TokenManager {
    private var accessToken: String?
    private var refreshToken: String?
    private var isRefreshing = false
    private var refreshWaiters: [CheckedContinuation<String, Error>] = []
    
    func getValidToken() async throws -> String {
        if let token = accessToken, !isTokenExpired(token) {
            return token
        }
        
        return try await refreshAccessToken()
    }
    
    private func refreshAccessToken() async throws -> String {
        if isRefreshing {
            // รอ refresh ที่กำลังทำอยู่
            return try await withCheckedThrowingContinuation { continuation in
                refreshWaiters.append(continuation)
            }
        }
        
        isRefreshing = true
        
        do {
            let newToken = try await performTokenRefresh()
            accessToken = newToken
            isRefreshing = false
            
            // แจ้ง waiters ทั้งหมด
            refreshWaiters.forEach { $0.resume(returning: newToken) }
            refreshWaiters.removeAll()
            
            return newToken
        } catch {
            isRefreshing = false
            refreshWaiters.forEach { $0.resume(throwing: error) }
            refreshWaiters.removeAll()
            throw error
        }
    }
    
    private func performTokenRefresh() async throws -> String {
        guard let refreshToken = refreshToken else {
            throw AuthError.noRefreshToken
        }
        
        // เรียก refresh endpoint
        let url = URL(string: "https://api.example.com/auth/refresh")!
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        request.httpBody = try JSONEncoder().encode(["refresh_token": refreshToken])
        
        let (data, _) = try await URLSession.shared.data(for: request)
        let response = try JSONDecoder().decode(TokenResponse.self, from: data)
        
        return response.accessToken
    }
    
    private func isTokenExpired(_ token: String) -> Bool {
        // ตรวจสอบ expiry จาก JWT หรือ stored expiry time
        return false // Simplified
    }
}
```

### Challenge-based Authentication (Digest, NTLM)

```swift
extension NetworkSessionManager {
    func urlSession(
        _ session: URLSession,
        didReceive challenge: URLAuthenticationChallenge,
        completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void
    ) {
        switch challenge.protectionSpace.authenticationMethod {
        case NSURLAuthenticationMethodHTTPBasic, NSURLAuthenticationMethodHTTPDigest:
            let credential = URLCredential(
                user: "username",
                password: "password",
                persistence: .forSession
            )
            completionHandler(.useCredential, credential)
            
        case NSURLAuthenticationMethodServerTrust:
            // Server certificate validation (see certificate pinning section)
            if let serverTrust = challenge.protectionSpace.serverTrust {
                let credential = URLCredential(trust: serverTrust)
                completionHandler(.useCredential, credential)
            } else {
                completionHandler(.cancelAuthenticationChallenge, nil)
            }
            
        default:
            completionHandler(.performDefaultHandling, nil)
        }
    }
}
```

---

## 14. Certificate Pinning Basics

Certificate pinning เป็นการ verify ว่า server certificate ที่ได้รับตรงกับที่เราคาดหวัง

```swift
class PinnedNetworkManager: NSObject {
    private var session: URLSession!
    private let pinnedCertificateData: Data
    
    init(pinnedCertificateName: String) {
        // โหลด certificate จาก bundle
        guard let certURL = Bundle.main.url(forResource: pinnedCertificateName, withExtension: "cer"),
              let certData = try? Data(contentsOf: certURL) else {
            fatalError("Certificate not found: \(pinnedCertificateName)")
        }
        
        self.pinnedCertificateData = certData
        
        super.init()
        
        let config = URLSessionConfiguration.default
        session = URLSession(configuration: config, delegate: self, delegateQueue: nil)
    }
}

extension PinnedNetworkManager: URLSessionDelegate {
    func urlSession(
        _ session: URLSession,
        didReceive challenge: URLAuthenticationChallenge,
        completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void
    ) {
        guard challenge.protectionSpace.authenticationMethod == NSURLAuthenticationMethodServerTrust,
              let serverTrust = challenge.protectionSpace.serverTrust else {
            completionHandler(.cancelAuthenticationChallenge, nil)
            return
        }
        
        // ดึง certificate จาก server
        guard let serverCertificate = SecTrustGetCertificateAtIndex(serverTrust, 0) else {
            completionHandler(.cancelAuthenticationChallenge, nil)
            return
        }
        
        let serverCertData = SecCertificateCopyData(serverCertificate) as Data
        
        // เปรียบเทียบกับ pinned certificate
        if serverCertData == pinnedCertificateData {
            // Certificate ตรงกัน - อนุญาต
            let credential = URLCredential(trust: serverTrust)
            completionHandler(.useCredential, credential)
        } else {
            // Certificate ไม่ตรง - ปฏิเสธ
            print("⚠️ Certificate pinning failed!")
            completionHandler(.cancelAuthenticationChallenge, nil)
        }
    }
}

// Public Key Pinning (แนะนำกว่า Certificate Pinning)
class PublicKeyPinnedManager: NSObject {
    private let pinnedPublicKeyHashes: Set<String>
    private var session: URLSession!
    
    init(pinnedHashes: Set<String>) {
        self.pinnedPublicKeyHashes = pinnedHashes
        super.init()
        session = URLSession(configuration: .default, delegate: self, delegateQueue: nil)
    }
    
    private func publicKeyHash(for certificate: SecCertificate) -> String? {
        guard let publicKey = SecCertificateCopyKey(certificate),
              let publicKeyData = SecKeyCopyExternalRepresentation(publicKey, nil) as Data? else {
            return nil
        }
        
        var hash = [UInt8](repeating: 0, count: Int(CC_SHA256_DIGEST_LENGTH))
        publicKeyData.withUnsafeBytes { buffer in
            _ = CC_SHA256(buffer.baseAddress, CC_LONG(buffer.count), &hash)
        }
        
        return Data(hash).base64EncodedString()
    }
}
```

---

## 15. URLCache

### การตั้งค่า URLCache

```swift
// ตั้งค่า shared cache
let memoryCapacity = 50 * 1024 * 1024    // 50 MB
let diskCapacity = 200 * 1024 * 1024     // 200 MB
let cache = URLCache(memoryCapacity: memoryCapacity, diskCapacity: diskCapacity)
URLCache.shared = cache

// Custom cache ใน session configuration
let config = URLSessionConfiguration.default
config.urlCache = URLCache(
    memoryCapacity: 20 * 1024 * 1024,
    diskCapacity: 100 * 1024 * 1024
)
config.requestCachePolicy = .useProtocolCachePolicy

// ตรวจสอบ cached response
func getCachedResponse(for url: URL) -> CachedURLResponse? {
    let request = URLRequest(url: url)
    return URLCache.shared.cachedResponse(for: request)
}

// บันทึก custom cache
func cacheResponse(_ data: Data, for url: URL) {
    let request = URLRequest(url: url)
    let response = HTTPURLResponse(
        url: url,
        statusCode: 200,
        httpVersion: "HTTP/1.1",
        headerFields: ["Cache-Control": "max-age=3600"]
    )!
    
    let cachedResponse = CachedURLResponse(response: response, data: data)
    URLCache.shared.storeCachedResponse(cachedResponse, for: request)
}

// ลบ cache
func clearCache(for url: URL) {
    let request = URLRequest(url: url)
    URLCache.shared.removeCachedResponse(for: request)
}

func clearAllCache() {
    URLCache.shared.removeAllCachedResponses()
}
```

---

## 16. Network Reachability

```swift
import Network

class NetworkReachability: ObservableObject {
    @Published var isConnected = false
    @Published var connectionType: ConnectionType = .unknown
    
    private let monitor = NWPathMonitor()
    private let queue = DispatchQueue(label: "NetworkMonitor")
    
    enum ConnectionType {
        case wifi, cellular, ethernet, unknown
    }
    
    init() {
        startMonitoring()
    }
    
    private func startMonitoring() {
        monitor.pathUpdateHandler = { [weak self] path in
            DispatchQueue.main.async {
                self?.isConnected = path.status == .satisfied
                self?.connectionType = self?.getConnectionType(from: path) ?? .unknown
                
                if path.status == .satisfied {
                    print("Network connected via \(self?.connectionType ?? .unknown)")
                } else {
                    print("Network disconnected")
                }
            }
        }
        
        monitor.start(queue: queue)
    }
    
    private func getConnectionType(from path: NWPath) -> ConnectionType {
        if path.usesInterfaceType(.wifi) {
            return .wifi
        } else if path.usesInterfaceType(.cellular) {
            return .cellular
        } else if path.usesInterfaceType(.wiredEthernet) {
            return .ethernet
        }
        return .unknown
    }
    
    func stopMonitoring() {
        monitor.cancel()
    }
    
    deinit {
        stopMonitoring()
    }
}

// การใช้งาน
class NetworkAwareViewController: UIViewController {
    private let reachability = NetworkReachability()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Observe network changes
        reachability.$isConnected
            .sink { [weak self] isConnected in
                self?.handleNetworkChange(isConnected: isConnected)
            }
            .store(in: &cancellables)
    }
    
    private func handleNetworkChange(isConnected: Bool) {
        if isConnected {
            fetchData()
        } else {
            showOfflineMessage()
        }
    }
}
```

---

## 17. Timeout Configuration

```swift
// Timeout ในระดับต่างๆ
struct TimeoutConfiguration {
    // Request timeout: เวลา maximum สำหรับ server ที่จะ respond
    static let requestTimeout: TimeInterval = 30
    
    // Resource timeout: เวลา maximum สำหรับ download/upload ทั้งหมด
    static let resourceTimeout: TimeInterval = 300
}

// Custom session กับ timeout
func createSessionWithTimeout(
    request: TimeInterval = 30,
    resource: TimeInterval = 300
) -> URLSession {
    let config = URLSessionConfiguration.default
    config.timeoutIntervalForRequest = request
    config.timeoutIntervalForResource = resource
    return URLSession(configuration: config)
}

// Timeout ใน URLRequest
func makeTimedRequest(url: URL, timeout: TimeInterval = 15) -> URLRequest {
    var request = URLRequest(url: url)
    request.timeoutInterval = timeout
    return request
}

// Cancellable Request กับ timeout
func fetchWithTimeout<T: Decodable>(
    _ type: T.Type,
    from url: URL,
    timeout: TimeInterval = 30
) async throws -> T {
    let task = Task {
        let (data, response) = try await URLSession.shared.data(from: url)
        try validateResponse(response)
        return try JSONDecoder().decode(type, from: data)
    }
    
    let timeoutTask = Task {
        try await Task.sleep(nanoseconds: UInt64(timeout * 1_000_000_000))
        task.cancel()
    }
    
    do {
        let result = try await task.value
        timeoutTask.cancel()
        return result
    } catch {
        timeoutTask.cancel()
        throw error
    }
}
```

---

## 18. Multipart Form Data

```swift
// Multipart Form Data สำหรับอัปโหลดไฟล์และข้อมูล
struct MultipartFormData {
    private let boundary: String
    private var body = Data()
    
    init() {
        boundary = "Boundary-\(UUID().uuidString)"
    }
    
    var contentType: String {
        "multipart/form-data; boundary=\(boundary)"
    }
    
    mutating func addField(name: String, value: String) {
        body.append("--\(boundary)\r\n".data(using: .utf8)!)
        body.append("Content-Disposition: form-data; name=\"\(name)\"\r\n\r\n".data(using: .utf8)!)
        body.append("\(value)\r\n".data(using: .utf8)!)
    }
    
    mutating func addFile(
        name: String,
        filename: String,
        data: Data,
        mimeType: String
    ) {
        body.append("--\(boundary)\r\n".data(using: .utf8)!)
        body.append("Content-Disposition: form-data; name=\"\(name)\"; filename=\"\(filename)\"\r\n".data(using: .utf8)!)
        body.append("Content-Type: \(mimeType)\r\n\r\n".data(using: .utf8)!)
        body.append(data)
        body.append("\r\n".data(using: .utf8)!)
    }
    
    func build() -> Data {
        var result = body
        result.append("--\(boundary)--\r\n".data(using: .utf8)!)
        return result
    }
}

// การใช้งาน
func uploadProfileImage(image: UIImage, username: String) async throws -> UserProfile {
    var formData = MultipartFormData()
    
    // เพิ่มข้อมูล text
    formData.addField(name: "username", value: username)
    
    // เพิ่มรูปภาพ
    guard let imageData = image.jpegData(compressionQuality: 0.8) else {
        throw UploadError.invalidImageData
    }
    
    formData.addFile(
        name: "avatar",
        filename: "profile.jpg",
        data: imageData,
        mimeType: "image/jpeg"
    )
    
    let url = URL(string: "https://api.example.com/users/profile")!
    var request = URLRequest(url: url)
    request.httpMethod = "POST"
    request.setValue(formData.contentType, forHTTPHeaderField: "Content-Type")
    
    let bodyData = formData.build()
    
    let (responseData, response) = try await URLSession.shared.upload(for: request, from: bodyData)
    try validateResponse(response)
    
    return try JSONDecoder().decode(UserProfile.self, from: responseData)
}
```

---

## 19. HTTP Cookies

```swift
// จัดการ cookies
class CookieManager {
    static let shared = CookieManager()
    private let cookieStorage = HTTPCookieStorage.shared
    
    // ตั้งค่า cookie
    func setCookie(name: String, value: String, domain: String) {
        let properties: [HTTPCookiePropertyKey: Any] = [
            .name: name,
            .value: value,
            .domain: domain,
            .path: "/",
            .expires: Date().addingTimeInterval(7 * 24 * 60 * 60)  // 7 วัน
        ]
        
        if let cookie = HTTPCookie(properties: properties) {
            cookieStorage.setCookie(cookie)
        }
    }
    
    // อ่าน cookies สำหรับ URL
    func cookies(for url: URL) -> [HTTPCookie] {
        return cookieStorage.cookies(for: url) ?? []
    }
    
    // ลบ cookie
    func deleteCookie(named name: String, for url: URL) {
        let cookies = cookieStorage.cookies(for: url) ?? []
        cookies.filter { $0.name == name }.forEach { cookieStorage.deleteCookie($0) }
    }
    
    // ลบ cookies ทั้งหมด
    func clearAllCookies() {
        cookieStorage.cookies?.forEach { cookieStorage.deleteCookie($0) }
    }
}

// Session กับ cookie handling
let config = URLSessionConfiguration.default
config.httpCookieAcceptPolicy = .always      // รับ cookies เสมอ
// หรือ
config.httpCookieAcceptPolicy = .never       // ไม่รับ cookies
// หรือ
config.httpCookieAcceptPolicy = .onlyFromMainDocumentDomain  // รับจาก main domain เท่านั้น

config.httpShouldSetCookies = true           // ส่ง cookies โดยอัตโนมัติ
```

---

## 20. REST API Patterns

### API Client

```swift
// Protocol-based API Client
protocol APIClientProtocol {
    func request<T: Decodable>(_ endpoint: APIEndpoint) async throws -> T
    func request(_ endpoint: APIEndpoint) async throws
}

// Endpoint Protocol
protocol APIEndpoint {
    var path: String { get }
    var method: HTTPMethod { get }
    var queryItems: [URLQueryItem]? { get }
    var headers: [String: String]? { get }
    var body: Encodable? { get }
}

// Concrete API Client
class APIClient: APIClientProtocol {
    private let baseURL: URL
    private let session: URLSession
    private let decoder: JSONDecoder
    private let encoder: JSONEncoder
    
    init(baseURL: URL, session: URLSession = .shared) {
        self.baseURL = baseURL
        self.session = session
        
        decoder = JSONDecoder()
        decoder.keyDecodingStrategy = .convertFromSnakeCase
        decoder.dateDecodingStrategy = .iso8601
        
        encoder = JSONEncoder()
        encoder.keyEncodingStrategy = .convertToSnakeCase
    }
    
    func request<T: Decodable>(_ endpoint: APIEndpoint) async throws -> T {
        let request = try buildRequest(for: endpoint)
        let (data, response) = try await session.data(for: request)
        
        try handleResponse(response, data: data)
        
        return try decoder.decode(T.self, from: data)
    }
    
    func request(_ endpoint: APIEndpoint) async throws {
        let request = try buildRequest(for: endpoint)
        let (data, response) = try await session.data(for: request)
        
        try handleResponse(response, data: data)
    }
    
    private func buildRequest(for endpoint: APIEndpoint) throws -> URLRequest {
        var components = URLComponents(url: baseURL.appendingPathComponent(endpoint.path), resolvingAgainstBaseURL: true)!
        
        if let queryItems = endpoint.queryItems {
            components.queryItems = queryItems
        }
        
        guard let url = components.url else {
            throw NetworkError.invalidURL
        }
        
        var request = URLRequest(url: url)
        request.httpMethod = endpoint.method.rawValue
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        request.setValue("application/json", forHTTPHeaderField: "Accept")
        
        endpoint.headers?.forEach { key, value in
            request.setValue(value, forHTTPHeaderField: key)
        }
        
        if let body = endpoint.body {
            request.httpBody = try encoder.encode(body)
        }
        
        return request
    }
    
    private func handleResponse(_ response: URLResponse, data: Data) throws {
        guard let httpResponse = response as? HTTPURLResponse else {
            throw NetworkError.invalidResponse
        }
        
        switch httpResponse.statusCode {
        case 200...299:
            return
        case 400:
            if let apiError = try? decoder.decode(APIError.self, from: data) {
                throw NetworkError.apiError(apiError)
            }
            throw NetworkError.badRequest
        case 401:
            throw NetworkError.unauthorized
        case 403:
            throw NetworkError.forbidden
        case 404:
            throw NetworkError.notFound
        case 422:
            if let validationErrors = try? decoder.decode(ValidationError.self, from: data) {
                throw NetworkError.validationError(validationErrors)
            }
            throw NetworkError.badRequest
        case 429:
            throw NetworkError.rateLimited
        case 500...599:
            throw NetworkError.serverError(httpResponse.statusCode)
        default:
            throw NetworkError.httpError(httpResponse.statusCode)
        }
    }
}
```

### Endpoints

```swift
// User Endpoints
enum UserEndpoints: APIEndpoint {
    case list(page: Int, limit: Int)
    case get(id: Int)
    case create(CreateUserRequest)
    case update(id: Int, UpdateUserRequest)
    case delete(id: Int)
    case search(query: String)
    
    var path: String {
        switch self {
        case .list: return "/users"
        case .get(let id): return "/users/\(id)"
        case .create: return "/users"
        case .update(let id, _): return "/users/\(id)"
        case .delete(let id): return "/users/\(id)"
        case .search: return "/users/search"
        }
    }
    
    var method: HTTPMethod {
        switch self {
        case .list, .get, .search: return .get
        case .create: return .post
        case .update: return .put
        case .delete: return .delete
        }
    }
    
    var queryItems: [URLQueryItem]? {
        switch self {
        case .list(let page, let limit):
            return [
                URLQueryItem(name: "page", value: String(page)),
                URLQueryItem(name: "limit", value: String(limit))
            ]
        case .search(let query):
            return [URLQueryItem(name: "q", value: query)]
        default:
            return nil
        }
    }
    
    var headers: [String: String]? { return nil }
    
    var body: Encodable? {
        switch self {
        case .create(let request): return request
        case .update(_, let request): return request
        default: return nil
        }
    }
}

// การใช้งาน
class UserRepository {
    private let client: APIClientProtocol
    
    init(client: APIClientProtocol) {
        self.client = client
    }
    
    func getAllUsers(page: Int = 1) async throws -> [User] {
        return try await client.request(UserEndpoints.list(page: page, limit: 20))
    }
    
    func getUser(id: Int) async throws -> User {
        return try await client.request(UserEndpoints.get(id: id))
    }
    
    func createUser(_ request: CreateUserRequest) async throws -> User {
        return try await client.request(UserEndpoints.create(request))
    }
    
    func updateUser(id: Int, with request: UpdateUserRequest) async throws -> User {
        return try await client.request(UserEndpoints.update(id: id, request))
    }
    
    func deleteUser(id: Int) async throws {
        try await client.request(UserEndpoints.delete(id: id))
    }
    
    func searchUsers(query: String) async throws -> [User] {
        return try await client.request(UserEndpoints.search(query: query))
    }
}
```

---

## 21. Request Interception

```swift
// Interceptor Pattern
protocol RequestInterceptor {
    func intercept(_ request: URLRequest) async throws -> URLRequest
    func intercept(_ response: HTTPURLResponse, data: Data) async throws -> Data
}

// Auth Interceptor
class AuthInterceptor: RequestInterceptor {
    private let tokenManager: TokenManager
    
    init(tokenManager: TokenManager) {
        self.tokenManager = tokenManager
    }
    
    func intercept(_ request: URLRequest) async throws -> URLRequest {
        let token = try await tokenManager.getValidToken()
        
        var modifiedRequest = request
        modifiedRequest.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        
        return modifiedRequest
    }
    
    func intercept(_ response: HTTPURLResponse, data: Data) async throws -> Data {
        if response.statusCode == 401 {
            // Token expired - refresh and retry
            throw NetworkError.unauthorized
        }
        return data
    }
}

// Logging Interceptor
class LoggingInterceptor: RequestInterceptor {
    func intercept(_ request: URLRequest) async throws -> URLRequest {
        print("→ \(request.httpMethod ?? "GET") \(request.url?.absoluteString ?? "")")
        if let body = request.httpBody, let bodyString = String(data: body, encoding: .utf8) {
            print("  Body: \(bodyString)")
        }
        return request
    }
    
    func intercept(_ response: HTTPURLResponse, data: Data) async throws -> Data {
        print("← \(response.statusCode) \(response.url?.absoluteString ?? "")")
        return data
    }
}

// Interceptor Chain
class InterceptorChain {
    private let interceptors: [RequestInterceptor]
    
    init(interceptors: [RequestInterceptor]) {
        self.interceptors = interceptors
    }
    
    func process(request: URLRequest) async throws -> URLRequest {
        var currentRequest = request
        
        for interceptor in interceptors {
            currentRequest = try await interceptor.intercept(currentRequest)
        }
        
        return currentRequest
    }
    
    func process(response: HTTPURLResponse, data: Data) async throws -> Data {
        var currentData = data
        
        for interceptor in interceptors.reversed() {
            currentData = try await interceptor.intercept(response, data: currentData)
        }
        
        return currentData
    }
}
```

---

## 22. Practical Exercises

### Exercise 1: Weather App

```swift
import Foundation

// MARK: - Models
struct WeatherResponse: Decodable {
    let main: MainWeather
    let weather: [WeatherDescription]
    let wind: Wind
    let name: String
    
    struct MainWeather: Decodable {
        let temp: Double
        let feelsLike: Double
        let tempMin: Double
        let tempMax: Double
        let humidity: Int
        let pressure: Int
        
        enum CodingKeys: String, CodingKey {
            case temp
            case feelsLike = "feels_like"
            case tempMin = "temp_min"
            case tempMax = "temp_max"
            case humidity
            case pressure
        }
    }
    
    struct WeatherDescription: Decodable {
        let id: Int
        let main: String
        let description: String
        let icon: String
    }
    
    struct Wind: Decodable {
        let speed: Double
        let deg: Int
    }
}

struct ForecastResponse: Decodable {
    let list: [ForecastItem]
    let city: City
    
    struct ForecastItem: Decodable {
        let dt: TimeInterval
        let main: WeatherResponse.MainWeather
        let weather: [WeatherResponse.WeatherDescription]
        let wind: WeatherResponse.Wind
        
        var date: Date { Date(timeIntervalSince1970: dt) }
    }
    
    struct City: Decodable {
        let name: String
        let country: String
    }
}

// MARK: - Weather Service
actor WeatherService {
    private let apiKey: String
    private let session: URLSession
    private let decoder: JSONDecoder
    private let cache = NSCache<NSString, CachedWeather>()
    
    class CachedWeather {
        let data: WeatherResponse
        let timestamp: Date
        
        init(data: WeatherResponse) {
            self.data = data
            self.timestamp = Date()
        }
        
        var isExpired: Bool {
            Date().timeIntervalSince(timestamp) > 600  // 10 นาที
        }
    }
    
    init(apiKey: String) {
        self.apiKey = apiKey
        
        let config = URLSessionConfiguration.default
        config.timeoutIntervalForRequest = 30
        self.session = URLSession(configuration: config)
        
        self.decoder = JSONDecoder()
    }
    
    func getCurrentWeather(for city: String) async throws -> WeatherResponse {
        let cacheKey = city as NSString
        
        // ตรวจสอบ cache
        if let cached = cache.object(forKey: cacheKey), !cached.isExpired {
            return cached.data
        }
        
        var components = URLComponents(string: "https://api.openweathermap.org/data/2.5/weather")!
        components.queryItems = [
            URLQueryItem(name: "q", value: city),
            URLQueryItem(name: "appid", value: apiKey),
            URLQueryItem(name: "units", value: "metric"),
            URLQueryItem(name: "lang", value: "th")
        ]
        
        guard let url = components.url else {
            throw URLError(.badURL)
        }
        
        let (data, response) = try await session.data(from: url)
        
        guard let httpResponse = response as? HTTPURLResponse,
              (200...299).contains(httpResponse.statusCode) else {
            if let httpResponse = response as? HTTPURLResponse, httpResponse.statusCode == 404 {
                throw WeatherError.cityNotFound(city)
            }
            throw WeatherError.fetchFailed
        }
        
        let weather = try decoder.decode(WeatherResponse.self, from: data)
        cache.setObject(CachedWeather(data: weather), forKey: cacheKey)
        
        return weather
    }
    
    func getForecast(for city: String, days: Int = 5) async throws -> ForecastResponse {
        var components = URLComponents(string: "https://api.openweathermap.org/data/2.5/forecast")!
        components.queryItems = [
            URLQueryItem(name: "q", value: city),
            URLQueryItem(name: "appid", value: apiKey),
            URLQueryItem(name: "units", value: "metric"),
            URLQueryItem(name: "cnt", value: String(days * 8)),  // 8 readings per day
            URLQueryItem(name: "lang", value: "th")
        ]
        
        guard let url = components.url else { throw URLError(.badURL) }
        
        let (data, response) = try await session.data(from: url)
        
        guard let httpResponse = response as? HTTPURLResponse,
              (200...299).contains(httpResponse.statusCode) else {
            throw WeatherError.fetchFailed
        }
        
        return try decoder.decode(ForecastResponse.self, from: data)
    }
    
    func getWeatherForMultipleCities(_ cities: [String]) async -> [String: Result<WeatherResponse, Error>] {
        await withTaskGroup(of: (String, Result<WeatherResponse, Error>).self) { group in
            for city in cities {
                group.addTask {
                    do {
                        let weather = try await self.getCurrentWeather(for: city)
                        return (city, .success(weather))
                    } catch {
                        return (city, .failure(error))
                    }
                }
            }
            
            var results: [String: Result<WeatherResponse, Error>] = [:]
            for await (city, result) in group {
                results[city] = result
            }
            return results
        }
    }
}

enum WeatherError: LocalizedError {
    case cityNotFound(String)
    case fetchFailed
    case invalidAPIKey
    
    var errorDescription: String? {
        switch self {
        case .cityNotFound(let city): return "ไม่พบเมือง: \(city)"
        case .fetchFailed: return "ไม่สามารถดึงข้อมูลสภาพอากาศได้"
        case .invalidAPIKey: return "API Key ไม่ถูกต้อง"
        }
    }
}

// MARK: - Weather ViewModel
@MainActor
class WeatherViewModel: ObservableObject {
    @Published var currentWeather: WeatherResponse?
    @Published var forecast: ForecastResponse?
    @Published var isLoading = false
    @Published var errorMessage: String?
    
    private let service: WeatherService
    
    init(apiKey: String) {
        self.service = WeatherService(apiKey: apiKey)
    }
    
    func loadWeather(for city: String) async {
        isLoading = true
        errorMessage = nil
        
        async let current = service.getCurrentWeather(for: city)
        async let forecast = service.getForecast(for: city)
        
        do {
            let (weatherData, forecastData) = try await (current, forecast)
            self.currentWeather = weatherData
            self.forecast = forecastData
        } catch {
            errorMessage = error.localizedDescription
        }
        
        isLoading = false
    }
    
    func loadMultipleCities(_ cities: [String]) async {
        isLoading = true
        let results = await service.getWeatherForMultipleCities(cities)
        
        for (city, result) in results {
            switch result {
            case .success(let weather):
                print("\(city): \(weather.main.temp)°C")
            case .failure(let error):
                print("\(city): Error - \(error.localizedDescription)")
            }
        }
        
        isLoading = false
    }
}
```

### Exercise 2: News Reader App

```swift
// MARK: - News Models
struct NewsResponse: Decodable {
    let status: String
    let totalResults: Int
    let articles: [Article]
}

struct Article: Decodable, Identifiable {
    let source: Source
    let author: String?
    let title: String
    let description: String?
    let url: String
    let urlToImage: String?
    let publishedAt: Date
    let content: String?
    
    var id: String { url }
    
    struct Source: Decodable {
        let id: String?
        let name: String
    }
    
    enum CodingKeys: String, CodingKey {
        case source, author, title, description, url, urlToImage, publishedAt, content
    }
    
    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        source = try container.decode(Source.self, forKey: .source)
        author = try container.decodeIfPresent(String.self, forKey: .author)
        title = try container.decode(String.self, forKey: .title)
        description = try container.decodeIfPresent(String.self, forKey: .description)
        url = try container.decode(String.self, forKey: .url)
        urlToImage = try container.decodeIfPresent(String.self, forKey: .urlToImage)
        content = try container.decodeIfPresent(String.self, forKey: .content)
        
        let dateString = try container.decode(String.self, forKey: .publishedAt)
        let formatter = ISO8601DateFormatter()
        publishedAt = formatter.date(from: dateString) ?? Date()
    }
}

// MARK: - News Service
actor NewsService {
    private let apiKey: String
    private let baseURL = "https://newsapi.org/v2"
    private let session: URLSession
    private let decoder: JSONDecoder
    
    enum Category: String {
        case business, entertainment, general, health, science, sports, technology
    }
    
    init(apiKey: String) {
        self.apiKey = apiKey
        
        let config = URLSessionConfiguration.default
        config.timeoutIntervalForRequest = 30
        config.requestCachePolicy = .useProtocolCachePolicy
        self.session = URLSession(configuration: config)
        
        decoder = JSONDecoder()
    }
    
    func fetchTopHeadlines(
        country: String = "th",
        category: Category? = nil,
        page: Int = 1,
        pageSize: Int = 20
    ) async throws -> NewsResponse {
        var components = URLComponents(string: "\(baseURL)/top-headlines")!
        var queryItems = [
            URLQueryItem(name: "country", value: country),
            URLQueryItem(name: "page", value: String(page)),
            URLQueryItem(name: "pageSize", value: String(pageSize)),
            URLQueryItem(name: "apiKey", value: apiKey)
        ]
        
        if let category = category {
            queryItems.append(URLQueryItem(name: "category", value: category.rawValue))
        }
        
        components.queryItems = queryItems
        
        guard let url = components.url else { throw URLError(.badURL) }
        
        let (data, response) = try await session.data(from: url)
        
        guard let httpResponse = response as? HTTPURLResponse else {
            throw NetworkError.invalidResponse
        }
        
        if httpResponse.statusCode == 401 {
            throw NetworkError.unauthorized
        }
        
        guard (200...299).contains(httpResponse.statusCode) else {
            throw NetworkError.httpError(httpResponse.statusCode)
        }
        
        return try decoder.decode(NewsResponse.self, from: data)
    }
    
    func searchArticles(
        query: String,
        language: String = "th",
        sortBy: String = "publishedAt",
        page: Int = 1
    ) async throws -> NewsResponse {
        var components = URLComponents(string: "\(baseURL)/everything")!
        components.queryItems = [
            URLQueryItem(name: "q", value: query),
            URLQueryItem(name: "language", value: language),
            URLQueryItem(name: "sortBy", value: sortBy),
            URLQueryItem(name: "page", value: String(page)),
            URLQueryItem(name: "apiKey", value: apiKey)
        ]
        
        guard let url = components.url else { throw URLError(.badURL) }
        
        let (data, response) = try await session.data(from: url)
        
        guard let httpResponse = response as? HTTPURLResponse,
              (200...299).contains(httpResponse.statusCode) else {
            throw NetworkError.invalidResponse
        }
        
        return try decoder.decode(NewsResponse.self, from: data)
    }
    
    func fetchAllCategories(country: String = "th") async throws -> [Category: [Article]] {
        let categories: [Category] = [.business, .technology, .sports, .health, .entertainment]
        
        return try await withThrowingTaskGroup(of: (Category, [Article]).self) { group in
            for category in categories {
                group.addTask {
                    let response = try await self.fetchTopHeadlines(country: country, category: category)
                    return (category, response.articles)
                }
            }
            
            var result: [Category: [Article]] = [:]
            for try await (category, articles) in group {
                result[category] = articles
            }
            return result
        }
    }
}

// MARK: - News ViewModel
@MainActor
class NewsViewModel: ObservableObject {
    @Published var articles: [Article] = []
    @Published var isLoading = false
    @Published var errorMessage: String?
    @Published var currentPage = 1
    @Published var hasMorePages = true
    
    private let service: NewsService
    private var currentCategory: NewsService.Category?
    private var currentSearchQuery: String?
    
    init(apiKey: String) {
        self.service = NewsService(apiKey: apiKey)
    }
    
    func loadTopHeadlines(category: NewsService.Category? = nil) async {
        currentPage = 1
        currentCategory = category
        currentSearchQuery = nil
        articles = []
        
        await fetchNextPage()
    }
    
    func search(query: String) async {
        currentPage = 1
        currentSearchQuery = query
        currentCategory = nil
        articles = []
        
        await fetchNextPage()
    }
    
    func loadMore() async {
        guard hasMorePages, !isLoading else { return }
        currentPage += 1
        await fetchNextPage()
    }
    
    private func fetchNextPage() async {
        guard !isLoading else { return }
        
        isLoading = true
        errorMessage = nil
        
        do {
            let response: NewsResponse
            
            if let query = currentSearchQuery {
                response = try await service.searchArticles(query: query, page: currentPage)
            } else {
                response = try await service.fetchTopHeadlines(
                    category: currentCategory,
                    page: currentPage
                )
            }
            
            if currentPage == 1 {
                articles = response.articles
            } else {
                articles.append(contentsOf: response.articles)
            }
            
            hasMorePages = articles.count < response.totalResults
            
        } catch {
            errorMessage = error.localizedDescription
            if currentPage > 1 { currentPage -= 1 }  // Roll back
        }
        
        isLoading = false
    }
    
    func refreshContent() async {
        if let query = currentSearchQuery {
            await search(query: query)
        } else {
            await loadTopHeadlines(category: currentCategory)
        }
    }
}
```

### Exercise 3: Network Monitor กับ Auto-Retry

```swift
// MARK: - Network-Aware Request Manager
class ResilienceNetworkManager {
    private let networkMonitor = NetworkReachability()
    private let maxRetries = 3
    private let baseRetryDelay: TimeInterval = 1.0
    
    // Stream ของ network status
    var isConnected: Bool { networkMonitor.isConnected }
    
    func request<T: Decodable>(
        _ type: T.Type,
        url: URL,
        method: String = "GET",
        headers: [String: String] = [:],
        body: Data? = nil
    ) async throws -> T {
        // รอ network ถ้าไม่มี connection
        if !isConnected {
            try await waitForConnection()
        }
        
        return try await withRetry(maxAttempts: maxRetries) {
            var request = URLRequest(url: url)
            request.httpMethod = method
            request.httpBody = body
            
            for (key, value) in headers {
                request.setValue(value, forHTTPHeaderField: key)
            }
            
            let (data, response) = try await URLSession.shared.data(for: request)
            
            guard let httpResponse = response as? HTTPURLResponse,
                  (200...299).contains(httpResponse.statusCode) else {
                throw NetworkError.badResponse
            }
            
            return try JSONDecoder().decode(type, from: data)
        }
    }
    
    private func waitForConnection(timeout: TimeInterval = 30) async throws {
        let deadline = Date().addingTimeInterval(timeout)
        
        while !isConnected {
            guard Date() < deadline else {
                throw NetworkError.timeout
            }
            try await Task.sleep(nanoseconds: 500_000_000)  // 0.5 วินาที
        }
    }
    
    private func withRetry<T>(
        maxAttempts: Int,
        operation: () async throws -> T
    ) async throws -> T {
        var lastError: Error?
        
        for attempt in 1...maxAttempts {
            do {
                return try await operation()
            } catch {
                lastError = error
                
                if attempt < maxAttempts {
                    let delay = baseRetryDelay * pow(2.0, Double(attempt - 1))
                    print("Attempt \(attempt) failed, retrying in \(delay)s...")
                    try await Task.sleep(nanoseconds: UInt64(delay * 1_000_000_000))
                }
            }
        }
        
        throw lastError!
    }
}
```

---

## 23. Error Handling Best Practices

```swift
// Comprehensive Error Types
enum NetworkError: Error, LocalizedError {
    case invalidURL
    case invalidResponse
    case noData
    case badRequest
    case unauthorized
    case forbidden
    case notFound
    case rateLimited
    case serverError(Int)
    case httpError(Int)
    case timeout
    case noNetwork
    case decodingError(Error)
    case apiError(APIError)
    case validationError(ValidationError)
    case unknown(Error)
    
    var errorDescription: String? {
        switch self {
        case .invalidURL: return "URL ไม่ถูกต้อง"
        case .invalidResponse: return "Response ไม่ถูกต้อง"
        case .noData: return "ไม่มีข้อมูล"
        case .badRequest: return "Request ไม่ถูกต้อง (400)"
        case .unauthorized: return "ไม่ได้รับอนุญาต - กรุณาล็อกอินใหม่"
        case .forbidden: return "ไม่มีสิทธิ์เข้าถึง"
        case .notFound: return "ไม่พบข้อมูลที่ต้องการ"
        case .rateLimited: return "ส่ง Request มากเกินไป กรุณารอสักครู่"
        case .serverError(let code): return "Server error: \(code)"
        case .httpError(let code): return "HTTP error: \(code)"
        case .timeout: return "หมดเวลา กรุณาลองใหม่"
        case .noNetwork: return "ไม่มีการเชื่อมต่ออินเทอร์เน็ต"
        case .decodingError: return "ไม่สามารถอ่านข้อมูลได้"
        case .apiError(let error): return error.message
        case .validationError(let error): return error.message
        case .unknown(let error): return error.localizedDescription
        }
    }
    
    var isRetryable: Bool {
        switch self {
        case .serverError, .timeout, .noNetwork, .rateLimited:
            return true
        default:
            return false
        }
    }
}

struct APIError: Decodable {
    let code: String
    let message: String
}

struct ValidationError: Decodable {
    let fields: [String: [String]]
    
    var message: String {
        fields.map { field, errors in
            "\(field): \(errors.joined(separator: ", "))"
        }.joined(separator: "\n")
    }
}
```

---

## 24. สรุป

URLSession เป็น framework ที่ทรงพลังสำหรับ Networking ใน iOS:

| Feature | การใช้งาน |
|---------|---------|
| `URLSession.shared` | Simple requests ทั่วไป |
| `URLSessionConfiguration` | Custom session settings |
| `async/await` | Modern way ของการทำ network calls |
| `URLSessionDelegate` | Fine-grained control |
| Background Session | Downloads/uploads ขณะ app ปิด |
| Certificate Pinning | Security สำหรับ sensitive apps |
| URLCache | Cache responses เพื่อ performance |
| NWPathMonitor | ตรวจสอบ network status |

### Architecture Recommendations

```
NetworkLayer/
├── APIClient.swift          // Core HTTP client
├── Endpoints/
│   ├── UserEndpoints.swift
│   └── ProductEndpoints.swift
├── Models/
│   ├── Request/
│   └── Response/
├── Interceptors/
│   ├── AuthInterceptor.swift
│   └── LoggingInterceptor.swift
├── Services/
│   ├── WeatherService.swift
│   └── NewsService.swift
└── Errors/
    └── NetworkError.swift
```

### Checklist สำหรับ Networking

- [ ] ใช้ `async/await` แทน completion handlers
- [ ] Handle errors อย่างครบถ้วน (network, HTTP, decoding)
- [ ] ใช้ `URLSessionConfiguration` ที่เหมาะสม
- [ ] Implement retry logic สำหรับ transient errors
- [ ] ตรวจสอบ network reachability ก่อน request
- [ ] ใช้ `URLCache` อย่างเหมาะสม
- [ ] Implement certificate pinning สำหรับ sensitive data
- [ ] Log requests/responses ใน development
- [ ] Cancel requests เมื่อไม่จำเป็น
- [ ] Handle background downloads สำหรับไฟล์ใหญ่

---

## แบบฝึกหัดเพิ่มเติม

1. สร้าง `APIClient` ที่รองรับ request signing ด้วย HMAC-SHA256
2. สร้าง Pagination helper ที่โหลดข้อมูลอัตโนมัติเมื่อ scroll ถึงท้าย list
3. สร้าง Mock URLSession สำหรับ Unit Testing ของ NetworkService
4. สร้าง Request Queue ที่จำกัด concurrent requests ไม่เกิน N requests
5. สร้าง Network Logger ที่บันทึก request/response ลงไฟล์สำหรับ debugging

---

*จบ Part 30: Networking กับ URLSession*
