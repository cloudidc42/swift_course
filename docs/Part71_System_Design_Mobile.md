# Part 71: System Design สำหรับ Mobile Applications

## บทนำ

System Design เป็นทักษะที่สำคัญมากสำหรับ Mobile Developer ระดับ Senior ไม่ใช่แค่การเขียนโค้ดได้ แต่ต้องสามารถออกแบบระบบที่ scalable, maintainable, และ performant ได้ด้วย บทนี้จะครอบคลุมหลักการและ pattern ต่างๆ ที่ใช้ในการออกแบบ Mobile Application จริงในระดับ Production

---

## 1. System Design Basics สำหรับ Mobile

### 1.1 ทำไม Mobile Developer ต้องรู้ System Design?

ในการสัมภาษณ์งานระดับ Senior หรือ Staff Engineer บริษัทชั้นนำอย่าง Google, Meta, Apple, Grab, LINE มักจะถาม System Design เสมอ นอกจากนี้ในการทำงานจริง เราต้องตัดสินใจเรื่อง:

- จะ cache data อย่างไร
- จะ handle offline mode อย่างไร
- จะ sync data กับ server อย่างไร
- จะออกแบบ API communication อย่างไร
- จะ handle real-time updates อย่างไร

### 1.2 หลักการพื้นฐาน

**Scalability** - ระบบต้องรองรับผู้ใช้ที่เพิ่มขึ้นได้

**Performance** - ต้อง load เร็ว, smooth animations, responsive UI

**Reliability** - ทำงานได้แม้ network ไม่เสถียร

**Maintainability** - โค้ดต้องดูแลรักษาได้ง่าย

**Security** - ปกป้องข้อมูลผู้ใช้

### 1.3 Constraints ของ Mobile

Mobile มี constraints พิเศษที่ Backend ไม่มี:

```swift
// Mobile-specific constraints:
// 1. Battery life - ต้องระวังการใช้ CPU/network
// 2. Memory limit - iOS/Android จำกัด RAM
// 3. Network variability - จาก WiFi ไป 3G ไป offline
// 4. Storage limit - user มี storage จำกัด
// 5. Screen size - ต้อง design ให้เหมาะสม
// 6. Background restrictions - iOS จำกัดการทำงาน background

struct MobileConstraints {
    let maxMemoryUsage: Int = 150 // MB สำหรับ app ปกติ
    let networkVariability: [String] = ["WiFi", "5G", "4G", "3G", "2G", "Offline"]
    let storageQuota: Int = 500 // MB สำหรับ cache
}
```

---

## 2. Client-Server Architecture

### 2.1 Overview

```
┌─────────────────────────────────────────────────────────┐
│                    Mobile Client                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐              │
│  │   UI     │  │Business  │  │  Data    │              │
│  │  Layer   │  │  Logic   │  │  Layer   │              │
│  └──────────┘  └──────────┘  └──────────┘              │
└─────────────────────────────┬───────────────────────────┘
                              │ HTTPS / WebSocket
┌─────────────────────────────▼───────────────────────────┐
│                    API Gateway                           │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐              │
│  │  Auth    │  │  Rate    │  │  Load    │              │
│  │  Check   │  │ Limiter  │  │ Balancer │              │
│  └──────────┘  └──────────┘  └──────────┘              │
└─────────────────────────────┬───────────────────────────┘
                              │
    ┌─────────────────────────┼─────────────────────────┐
    │                         │                         │
    ▼                         ▼                         ▼
┌──────────┐           ┌──────────┐           ┌──────────┐
│  User    │           │  Feed    │           │  Chat    │
│ Service  │           │ Service  │           │ Service  │
└──────────┘           └──────────┘           └──────────┘
```

### 2.2 Layered Architecture สำหรับ iOS

```swift
// Presentation Layer
struct FeedView: View {
    @StateObject private var viewModel = FeedViewModel()
    
    var body: some View {
        List(viewModel.posts) { post in
            PostCell(post: post)
        }
        .task { await viewModel.loadFeed() }
    }
}

// Domain Layer (Business Logic)
@MainActor
class FeedViewModel: ObservableObject {
    @Published var posts: [Post] = []
    private let useCase: FeedUseCase
    
    init(useCase: FeedUseCase = FeedUseCaseImpl()) {
        self.useCase = useCase
    }
    
    func loadFeed() async {
        do {
            posts = try await useCase.fetchFeed()
        } catch {
            // Handle error
        }
    }
}

// Use Case
protocol FeedUseCase {
    func fetchFeed() async throws -> [Post]
}

class FeedUseCaseImpl: FeedUseCase {
    private let repository: FeedRepository
    
    init(repository: FeedRepository = FeedRepositoryImpl()) {
        self.repository = repository
    }
    
    func fetchFeed() async throws -> [Post] {
        return try await repository.getFeed()
    }
}

// Data Layer (Repository)
protocol FeedRepository {
    func getFeed() async throws -> [Post]
}

class FeedRepositoryImpl: FeedRepository {
    private let remoteDataSource: FeedRemoteDataSource
    private let localDataSource: FeedLocalDataSource
    
    init(remote: FeedRemoteDataSource = FeedAPIDataSource(),
         local: FeedLocalDataSource = FeedCoreDataSource()) {
        self.remoteDataSource = remote
        self.localDataSource = local
    }
    
    func getFeed() async throws -> [Post] {
        // 1. ลองดึงจาก cache ก่อน
        if let cachedPosts = try? localDataSource.getCachedFeed(),
           !cachedPosts.isEmpty {
            // Load from cache, then refresh in background
            Task { try? await refreshFeed() }
            return cachedPosts
        }
        
        // 2. ถ้าไม่มี cache ดึงจาก remote
        let posts = try await remoteDataSource.fetchFeed()
        try? localDataSource.saveFeed(posts)
        return posts
    }
    
    private func refreshFeed() async throws {
        let posts = try await remoteDataSource.fetchFeed()
        try? localDataSource.saveFeed(posts)
    }
}
```

### 2.3 Network Layer

```swift
// Network Layer ที่ดีควร handle:
// 1. Authentication headers
// 2. Request/Response interceptors
// 3. Retry logic
// 4. Error mapping

class NetworkClient {
    private let session: URLSession
    private let baseURL: URL
    private var authToken: String?
    
    init(baseURL: URL, configuration: URLSessionConfiguration = .default) {
        self.baseURL = baseURL
        self.session = URLSession(configuration: configuration)
    }
    
    func request<T: Decodable>(
        endpoint: Endpoint,
        responseType: T.Type
    ) async throws -> T {
        let request = try buildRequest(for: endpoint)
        
        let (data, response) = try await session.data(for: request)
        
        guard let httpResponse = response as? HTTPURLResponse else {
            throw NetworkError.invalidResponse
        }
        
        switch httpResponse.statusCode {
        case 200...299:
            return try JSONDecoder().decode(T.self, from: data)
        case 401:
            throw NetworkError.unauthorized
        case 429:
            throw NetworkError.rateLimited
        case 500...599:
            throw NetworkError.serverError(httpResponse.statusCode)
        default:
            throw NetworkError.unknown(httpResponse.statusCode)
        }
    }
    
    private func buildRequest(for endpoint: Endpoint) throws -> URLRequest {
        let url = baseURL.appendingPathComponent(endpoint.path)
        var components = URLComponents(url: url, resolvingAgainstBaseURL: true)!
        components.queryItems = endpoint.queryItems
        
        var request = URLRequest(url: components.url!)
        request.httpMethod = endpoint.method.rawValue
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        
        if let token = authToken {
            request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        }
        
        if let body = endpoint.body {
            request.httpBody = try JSONEncoder().encode(body)
        }
        
        return request
    }
}

enum NetworkError: LocalizedError {
    case invalidResponse
    case unauthorized
    case rateLimited
    case serverError(Int)
    case unknown(Int)
    
    var errorDescription: String? {
        switch self {
        case .unauthorized: return "กรุณาเข้าสู่ระบบใหม่"
        case .rateLimited: return "คำขอมากเกินไป กรุณารอสักครู่"
        case .serverError(let code): return "เซิร์ฟเวอร์มีปัญหา (Error: \(code))"
        default: return "เกิดข้อผิดพลาด กรุณาลองใหม่"
        }
    }
}
```

---

## 3. REST vs GraphQL vs gRPC

### 3.1 REST API

REST (Representational State Transfer) เป็น standard ที่ใช้กันมากที่สุด

**ข้อดี:**
- Simple และเข้าใจง่าย
- Cacheable ได้ดี (HTTP caching)
- Stateless
- Wide tooling support

**ข้อเสีย:**
- Over-fetching (ได้ข้อมูลมากกว่าที่ต้องการ)
- Under-fetching (ต้องเรียก API หลายครั้ง)
- Versioning ยาก

```swift
// REST API Example
// GET /api/v1/users/{id}
// GET /api/v1/users/{id}/posts
// GET /api/v1/users/{id}/followers

struct RESTClient {
    func fetchUserProfile(id: String) async throws -> User {
        // ต้องเรียก 3 endpoints เพื่อแสดง profile ครบ
        async let user = fetchUser(id: id)
        async let posts = fetchUserPosts(userId: id)
        async let followers = fetchUserFollowers(userId: id)
        
        return try await UserProfile(
            user: user,
            posts: posts,
            followers: followers
        )
    }
}
```

### 3.2 GraphQL

GraphQL ให้ client กำหนดเองว่าต้องการข้อมูลอะไร

**ข้อดี:**
- Exact data fetching (ไม่ over/under fetch)
- Single endpoint
- Strongly typed schema
- Real-time subscriptions

**ข้อเสีย:**
- Complexity สูงกว่า REST
- Caching ยากกว่า
- Performance อาจแย่กว่าถ้าออกแบบ query ไม่ดี

```swift
// GraphQL Query
let query = """
query GetUserProfile($id: ID!) {
    user(id: $id) {
        id
        name
        avatar
        posts(first: 10) {
            edges {
                node {
                    id
                    title
                    createdAt
                }
            }
        }
        followersCount
    }
}
"""

// Swift GraphQL Client (ใช้ Apollo iOS)
class GraphQLClient {
    let apollo = ApolloClient(url: URL(string: "https://api.example.com/graphql")!)
    
    func fetchUserProfile(id: String) async throws -> UserProfile {
        return try await withCheckedThrowingContinuation { continuation in
            apollo.fetch(query: GetUserProfileQuery(id: id)) { result in
                switch result {
                case .success(let data):
                    if let user = data.data?.user {
                        continuation.resume(returning: UserProfile(from: user))
                    }
                case .failure(let error):
                    continuation.resume(throwing: error)
                }
            }
        }
    }
}
```

### 3.3 gRPC

gRPC ใช้ Protocol Buffers และ HTTP/2 เหมาะกับ microservices communication

**ข้อดี:**
- Performance สูงมาก (binary format)
- Strongly typed
- Bidirectional streaming
- Code generation

**ข้อเสีย:**
- Setup ซับซ้อน
- ไม่ human-readable
- Browser support จำกัด (ต้องใช้ grpc-web)

```protobuf
// user.proto
syntax = "proto3";

service UserService {
    rpc GetUser (GetUserRequest) returns (User);
    rpc StreamUserUpdates (GetUserRequest) returns (stream User);
}

message GetUserRequest {
    string user_id = 1;
}

message User {
    string id = 1;
    string name = 2;
    string avatar_url = 3;
    int64 created_at = 4;
}
```

```swift
// Swift gRPC Client (ใช้ grpc-swift)
import GRPC

class GRPCUserClient {
    private let client: UserServiceNIOClient
    
    init(host: String, port: Int) {
        let group = MultiThreadedEventLoopGroup(numberOfThreads: 1)
        let channel = try! GRPCChannelPool.with(
            target: .host(host, port: port),
            transportSecurity: .tls(GRPCTLSConfiguration.makeClientDefault()),
            eventLoopGroup: group
        )
        client = UserServiceNIOClient(channel: channel)
    }
    
    func getUser(id: String) async throws -> User {
        var request = GetUserRequest()
        request.userID = id
        return try await client.getUser(request).response.get()
    }
}
```

### 3.4 เปรียบเทียบและเลือกใช้

| Feature | REST | GraphQL | gRPC |
|---------|------|---------|------|
| Performance | ดี | ดี | ดีมาก |
| Flexibility | ปานกลาง | สูง | ต่ำ |
| Learning Curve | ต่ำ | ปานกลาง | สูง |
| Caching | ง่าย | ยาก | ยาก |
| Type Safety | ต่ำ | สูง | สูงมาก |
| Mobile Friendly | ดี | ดีมาก | ดี |

**เมื่อไหรควรใช้อะไร:**
- REST: Public API, Simple CRUD, Web + Mobile
- GraphQL: Complex UI ที่ต้องการ data หลาย source, Mobile ที่ต้องการ minimize data transfer
- gRPC: Internal microservices, Low-latency requirements, Streaming data

---

## 4. Caching Strategies

### 4.1 Memory Cache (NSCache)

```swift
// NSCache สำหรับ in-memory caching
class ImageCache {
    static let shared = ImageCache()
    
    private let cache: NSCache<NSString, UIImage> = {
        let cache = NSCache<NSString, UIImage>()
        cache.countLimit = 100        // max 100 items
        cache.totalCostLimit = 50 * 1024 * 1024 // 50 MB
        return cache
    }()
    
    func setImage(_ image: UIImage, forKey key: String) {
        let cost = Int(image.size.width * image.size.height * 4) // 4 bytes per pixel
        cache.setObject(image, forKey: key as NSString, cost: cost)
    }
    
    func getImage(forKey key: String) -> UIImage? {
        return cache.object(forKey: key as NSString)
    }
    
    func removeImage(forKey key: String) {
        cache.removeObject(forKey: key as NSString)
    }
    
    func clearAll() {
        cache.removeAllObjects()
    }
}

// Generic Cache
class GenericCache<Key: Hashable, Value> {
    private var cache: [Key: CacheEntry<Value>] = [:]
    private let ttl: TimeInterval
    private let lock = NSLock()
    
    init(ttl: TimeInterval = 300) { // 5 minutes default
        self.ttl = ttl
    }
    
    struct CacheEntry<V> {
        let value: V
        let expiresAt: Date
        
        var isExpired: Bool {
            return Date() > expiresAt
        }
    }
    
    func set(_ value: Value, forKey key: Key) {
        lock.lock()
        defer { lock.unlock() }
        
        cache[key] = CacheEntry(
            value: value,
            expiresAt: Date().addingTimeInterval(ttl)
        )
    }
    
    func get(forKey key: Key) -> Value? {
        lock.lock()
        defer { lock.unlock() }
        
        guard let entry = cache[key] else { return nil }
        
        if entry.isExpired {
            cache.removeValue(forKey: key)
            return nil
        }
        
        return entry.value
    }
}
```

### 4.2 Disk Cache

```swift
// Disk Cache สำหรับ persistent storage
class DiskCache {
    private let cacheDirectory: URL
    private let fileManager = FileManager.default
    private let maxDiskSize: Int64 = 200 * 1024 * 1024 // 200 MB
    
    init(name: String = "AppCache") {
        let caches = fileManager.urls(for: .cachesDirectory, in: .userDomainMask).first!
        cacheDirectory = caches.appendingPathComponent(name)
        
        try? fileManager.createDirectory(at: cacheDirectory, withIntermediateDirectories: true)
    }
    
    func save(data: Data, forKey key: String) throws {
        let url = cacheURL(for: key)
        try data.write(to: url, options: .atomic)
        
        // Trim cache ถ้าใหญ่เกินไป
        trimCacheIfNeeded()
    }
    
    func load(forKey key: String) -> Data? {
        let url = cacheURL(for: key)
        return try? Data(contentsOf: url)
    }
    
    func remove(forKey key: String) throws {
        let url = cacheURL(for: key)
        try fileManager.removeItem(at: url)
    }
    
    private func cacheURL(for key: String) -> URL {
        // Hash key เพื่อหลีกเลี่ยง invalid filename characters
        let hashedKey = key.data(using: .utf8)!
            .base64EncodedString()
            .replacingOccurrences(of: "/", with: "_")
        return cacheDirectory.appendingPathComponent(hashedKey)
    }
    
    private func trimCacheIfNeeded() {
        guard let contents = try? fileManager.contentsOfDirectory(
            at: cacheDirectory,
            includingPropertiesForKeys: [.fileSizeKey, .contentModificationDateKey]
        ) else { return }
        
        var totalSize: Int64 = 0
        var files: [(URL, Int64, Date)] = []
        
        for url in contents {
            let values = try? url.resourceValues(forKeys: [.fileSizeKey, .contentModificationDateKey])
            let size = Int64(values?.fileSize ?? 0)
            let date = values?.contentModificationDate ?? Date.distantPast
            totalSize += size
            files.append((url, size, date))
        }
        
        if totalSize > maxDiskSize {
            // Sort by oldest first
            let sorted = files.sorted { $0.2 < $1.2 }
            var removedSize: Int64 = 0
            let targetRemoval = totalSize - (maxDiskSize / 2)
            
            for (url, size, _) in sorted {
                if removedSize >= targetRemoval { break }
                try? fileManager.removeItem(at: url)
                removedSize += size
            }
        }
    }
}
```

### 4.3 HTTP Cache

```swift
// HTTP Cache ใช้ URLCache built-in
class HTTPCacheManager {
    static func configure() {
        let memoryCapacity = 50 * 1024 * 1024    // 50 MB memory
        let diskCapacity = 200 * 1024 * 1024      // 200 MB disk
        
        URLCache.shared = URLCache(
            memoryCapacity: memoryCapacity,
            diskCapacity: diskCapacity,
            diskPath: "http_cache"
        )
    }
    
    // Force cache (ใช้ cache แม้ expired)
    static func requestWithCacheFirst(url: URL) async throws -> Data {
        let request = URLRequest(url: url, cachePolicy: .returnCacheDataElseLoad)
        let (data, _) = try await URLSession.shared.data(for: request)
        return data
    }
    
    // Network first แล้ว fallback ไป cache
    static func requestWithNetworkFirst(url: URL) async throws -> Data {
        do {
            let request = URLRequest(url: url, cachePolicy: .reloadIgnoringLocalCacheData)
            let (data, response) = try await URLSession.shared.data(for: request)
            return data
        } catch {
            // Network failed, ลอง cache
            let cachedRequest = URLRequest(url: url, cachePolicy: .returnCacheDataDontLoad)
            if let cached = URLCache.shared.cachedResponse(for: cachedRequest) {
                return cached.data
            }
            throw error
        }
    }
}

// Cache Control Headers
// Cache-Control: max-age=3600 (cache 1 ชั่วโมง)
// Cache-Control: no-cache (ต้อง validate กับ server ก่อน)
// Cache-Control: no-store (ห้าม cache เลย)
// ETag: "abc123" (สำหรับ conditional requests)
// Last-Modified: Wed, 21 Oct 2015 07:28:00 GMT
```

### 4.4 Multi-level Cache Strategy

```swift
// Cache Manager ที่รวม Memory + Disk + HTTP
class MultiLevelCacheManager {
    private let memoryCache = GenericCache<String, Data>(ttl: 300)   // 5 min
    private let diskCache = DiskCache()
    
    func fetch(key: String, fetcher: () async throws -> Data) async throws -> Data {
        // Level 1: Memory Cache
        if let data = memoryCache.get(forKey: key) {
            print("Cache HIT (Memory): \(key)")
            return data
        }
        
        // Level 2: Disk Cache
        if let data = diskCache.load(forKey: key) {
            print("Cache HIT (Disk): \(key)")
            memoryCache.set(data, forKey: key) // Promote to memory
            return data
        }
        
        // Level 3: Network
        print("Cache MISS: \(key)")
        let data = try await fetcher()
        
        // Save to both caches
        memoryCache.set(data, forKey: key)
        try? diskCache.save(data: data, forKey: key)
        
        return data
    }
    
    func invalidate(key: String) {
        memoryCache.remove(forKey: key)  // ถ้า cache มี remove method
        try? diskCache.remove(forKey: key)
    }
}
```

---

## 5. Offline-First Architecture

### 5.1 หลักการ Offline-First

Offline-first หมายความว่าออกแบบ app ให้ทำงานได้แม้ไม่มี internet จากนั้นค่อย sync เมื่อ online

```swift
// Network Monitor
class NetworkMonitor: ObservableObject {
    static let shared = NetworkMonitor()
    
    @Published var isConnected = true
    @Published var connectionType: ConnectionType = .wifi
    
    private let monitor = NWPathMonitor()
    
    enum ConnectionType {
        case wifi, cellular, unknown
    }
    
    init() {
        monitor.pathUpdateHandler = { [weak self] path in
            DispatchQueue.main.async {
                self?.isConnected = path.status == .satisfied
                
                if path.usesInterfaceType(.wifi) {
                    self?.connectionType = .wifi
                } else if path.usesInterfaceType(.cellular) {
                    self?.connectionType = .cellular
                } else {
                    self?.connectionType = .unknown
                }
            }
        }
        
        monitor.start(queue: DispatchQueue.global())
    }
    
    deinit {
        monitor.cancel()
    }
}
```

### 5.2 Local-First Database

```swift
// ใช้ Core Data หรือ SwiftData เป็น source of truth
@Model
class Post {
    @Attribute(.unique) var id: String
    var title: String
    var content: String
    var authorId: String
    var createdAt: Date
    var syncStatus: SyncStatus
    
    enum SyncStatus: String, Codable {
        case synced
        case pending    // รอ sync ขึ้น server
        case failed     // sync ล้มเหลว
        case deleted    // ถูกลบ local แต่ยังไม่ sync
    }
    
    init(id: String, title: String, content: String, authorId: String) {
        self.id = id
        self.title = title
        self.content = content
        self.authorId = authorId
        self.createdAt = Date()
        self.syncStatus = .pending
    }
}

// Offline Queue สำหรับ pending operations
class OfflineOperationQueue {
    @AppStorage("offline_operations") private var encodedOperations: Data = Data()
    
    struct Operation: Codable {
        let id: UUID
        let type: OperationType
        let payload: Data
        let createdAt: Date
        var retryCount: Int
        
        enum OperationType: String, Codable {
            case create, update, delete
        }
    }
    
    private var operations: [Operation] {
        get {
            return (try? JSONDecoder().decode([Operation].self, from: encodedOperations)) ?? []
        }
        set {
            encodedOperations = (try? JSONEncoder().encode(newValue)) ?? Data()
        }
    }
    
    func enqueue(_ operation: Operation) {
        var ops = operations
        ops.append(operation)
        operations = ops
    }
    
    func processAll() async {
        for operation in operations {
            do {
                try await process(operation)
                dequeue(operation.id)
            } catch {
                handleFailure(operation, error: error)
            }
        }
    }
    
    private func process(_ operation: Operation) async throws {
        // ส่ง operation ไปยัง server
    }
    
    private func dequeue(_ id: UUID) {
        operations = operations.filter { $0.id != id }
    }
    
    private func handleFailure(_ operation: Operation, error: Error) {
        var ops = operations
        if let index = ops.firstIndex(where: { $0.id == operation.id }) {
            ops[index].retryCount += 1
            if ops[index].retryCount > 3 {
                // Remove failed operation after 3 retries
                ops.remove(at: index)
                // Notify user about failure
            }
        }
        operations = ops
    }
}
```

---

## 6. Sync Strategies

### 6.1 Pessimistic Locking

ล็อค resource ก่อนที่จะ edit เพื่อป้องกัน conflict

```swift
// Pessimistic Lock - ขอ lock จาก server ก่อน edit
class PessimisticLockManager {
    private var activeLocks: [String: LockInfo] = [:]
    
    struct LockInfo {
        let resourceId: String
        let lockedBy: String
        let expiresAt: Date
    }
    
    func acquireLock(resourceId: String, userId: String) async throws -> LockInfo {
        // ขอ lock จาก server
        let response = try await APIClient.shared.acquireLock(
            resourceId: resourceId,
            userId: userId,
            ttl: 300 // 5 minutes
        )
        
        let lock = LockInfo(
            resourceId: resourceId,
            lockedBy: userId,
            expiresAt: Date().addingTimeInterval(300)
        )
        
        activeLocks[resourceId] = lock
        return lock
    }
    
    func releaseLock(resourceId: String) async throws {
        try await APIClient.shared.releaseLock(resourceId: resourceId)
        activeLocks.removeValue(forKey: resourceId)
    }
    
    // ใช้ใน UI
    func editDocument(_ documentId: String) async throws {
        let lock = try await acquireLock(resourceId: documentId, userId: currentUserId)
        
        defer {
            Task { try? await releaseLock(resourceId: documentId) }
        }
        
        // Edit document while holding lock
        // Lock จะ expire ใน 5 นาที
    }
}
```

### 6.2 Optimistic Locking

สมมติว่าไม่มี conflict แล้วค่อย detect เมื่อ save

```swift
// Optimistic Locking ใช้ version number หรือ timestamp
struct DocumentUpdate {
    let documentId: String
    let newContent: String
    let expectedVersion: Int  // version ที่เราเห็นตอน read
}

class OptimisticLockingRepository {
    func updateDocument(_ update: DocumentUpdate) async throws -> Document {
        do {
            return try await APIClient.shared.updateDocument(
                id: update.documentId,
                content: update.newContent,
                version: update.expectedVersion
            )
        } catch let error as ConflictError {
            // Document ถูก update โดยคนอื่นแล้ว
            // ต้อง merge changes หรือแจ้ง user
            throw SyncError.conflict(latestVersion: error.serverVersion)
        }
    }
}

// Conflict Resolution
enum ConflictResolutionStrategy {
    case serverWins          // เอาของ server เสมอ
    case clientWins          // เอาของ client เสมอ
    case lastWriteWins       // เอาของที่เขียนล่าสุด
    case mergeFields         // merge field by field
    case askUser             // ถาม user ว่าจะเอาอะไร
}

class ConflictResolver {
    func resolve<T: Mergeable>(
        localVersion: T,
        serverVersion: T,
        baseVersion: T?,
        strategy: ConflictResolutionStrategy
    ) -> T {
        switch strategy {
        case .serverWins:
            return serverVersion
        case .clientWins:
            return localVersion
        case .lastWriteWins:
            return localVersion.updatedAt > serverVersion.updatedAt ? localVersion : serverVersion
        case .mergeFields:
            return localVersion.merge(with: serverVersion, base: baseVersion)
        case .askUser:
            // This would present UI to user
            fatalError("Must handle in UI layer")
        }
    }
}
```

### 6.3 CRDT (Conflict-free Replicated Data Types)

สำหรับ real-time collaborative editing เช่น document editors

```swift
// Last-Write-Wins Register (LWW-Register)
struct LWWRegister<T: Codable> {
    var value: T
    var timestamp: Date
    var nodeId: String
    
    mutating func set(_ newValue: T, at time: Date, by node: String) {
        if time > timestamp || (time == timestamp && node > nodeId) {
            value = newValue
            timestamp = time
            nodeId = node
        }
    }
    
    func merge(with other: LWWRegister<T>) -> LWWRegister<T> {
        if other.timestamp > timestamp {
            return other
        } else if other.timestamp == timestamp && other.nodeId > nodeId {
            return other
        }
        return self
    }
}
```

---

## 7. Push vs Pull Data Model

### 7.1 Pull Model

Client ดึงข้อมูลจาก server ตามที่ต้องการ

```swift
// Pull Model - Polling
class PollingDataService {
    private var timer: Timer?
    private let interval: TimeInterval
    
    init(interval: TimeInterval = 30) {
        self.interval = interval
    }
    
    func startPolling(handler: @escaping () async throws -> Void) {
        timer = Timer.scheduledTimer(withTimeInterval: interval, repeats: true) { _ in
            Task {
                try? await handler()
            }
        }
    }
    
    func stopPolling() {
        timer?.invalidate()
        timer = nil
    }
}
```

### 7.2 Push Model

Server ส่งข้อมูลมายัง client เมื่อมีการเปลี่ยนแปลง

```swift
// Push Model - APNs (Apple Push Notification service)
class PushNotificationHandler {
    func handleBackgroundPush(payload: [AnyHashable: Any]) async {
        guard let aps = payload["aps"] as? [String: Any],
              let contentAvailable = aps["content-available"] as? Int,
              contentAvailable == 1 else { return }
        
        // Background push - อัพเดทข้อมูลใน background
        if let dataType = payload["data-type"] as? String {
            switch dataType {
            case "new_message":
                await syncMessages()
            case "feed_update":
                await syncFeed()
            default:
                break
            }
        }
    }
    
    private func syncMessages() async {
        // ดึง messages ใหม่จาก server
    }
}
```

---

## 8. Real-time Updates

### 8.1 WebSocket

Connection แบบ bi-directional ที่ persistent

```swift
import Foundation

class WebSocketManager: NSObject, ObservableObject {
    private var webSocketTask: URLSessionWebSocketTask?
    private var urlSession: URLSession!
    @Published var isConnected = false
    @Published var messages: [ChatMessage] = []
    
    override init() {
        super.init()
        urlSession = URLSession(configuration: .default, delegate: self, delegateQueue: nil)
    }
    
    func connect(url: URL, token: String) {
        var request = URLRequest(url: url)
        request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        
        webSocketTask = urlSession.webSocketTask(with: request)
        webSocketTask?.resume()
        isConnected = true
        
        receiveMessage()
    }
    
    private func receiveMessage() {
        webSocketTask?.receive { [weak self] result in
            switch result {
            case .success(let message):
                switch message {
                case .string(let text):
                    self?.handleMessage(text: text)
                case .data(let data):
                    self?.handleMessage(data: data)
                @unknown default:
                    break
                }
                // Continue receiving
                self?.receiveMessage()
                
            case .failure(let error):
                print("WebSocket error: \(error)")
                self?.reconnect()
            }
        }
    }
    
    func send(message: String) {
        let message = URLSessionWebSocketTask.Message.string(message)
        webSocketTask?.send(message) { error in
            if let error = error {
                print("Send error: \(error)")
            }
        }
    }
    
    private func handleMessage(text: String) {
        guard let data = text.data(using: .utf8),
              let message = try? JSONDecoder().decode(ChatMessage.self, from: data) else { return }
        
        DispatchQueue.main.async {
            self.messages.append(message)
        }
    }
    
    private func reconnect() {
        isConnected = false
        
        // Exponential backoff
        let delay: TimeInterval = 5
        DispatchQueue.global().asyncAfter(deadline: .now() + delay) { [weak self] in
            guard let self = self else { return }
            // Reconnect logic
        }
    }
    
    func disconnect() {
        webSocketTask?.cancel(with: .goingAway, reason: nil)
        isConnected = false
    }
    
    private func handleMessage(data: Data) {
        // Handle binary data
    }
}

extension WebSocketManager: URLSessionWebSocketDelegate {
    func urlSession(_ session: URLSession, webSocketTask: URLSessionWebSocketTask, 
                    didOpenWithProtocol protocol: String?) {
        DispatchQueue.main.async {
            self.isConnected = true
        }
    }
    
    func urlSession(_ session: URLSession, webSocketTask: URLSessionWebSocketTask, 
                    didCloseWith closeCode: URLSessionWebSocketTask.CloseCode, reason: Data?) {
        DispatchQueue.main.async {
            self.isConnected = false
        }
    }
}
```

### 8.2 Server-Sent Events (SSE)

Server ส่งข้อมูลมายัง client แบบ one-way streaming

```swift
// SSE Client
class SSEClient: NSObject, URLSessionDataDelegate {
    private var urlSession: URLSession!
    private var dataTask: URLSessionDataTask?
    private var buffer = ""
    
    var onEvent: ((SSEEvent) -> Void)?
    var onError: ((Error) -> Void)?
    
    struct SSEEvent {
        let id: String?
        let event: String?
        let data: String
    }
    
    override init() {
        super.init()
        let configuration = URLSessionConfiguration.default
        configuration.timeoutIntervalForRequest = TimeInterval(INT_MAX)
        configuration.timeoutIntervalForResource = TimeInterval(INT_MAX)
        urlSession = URLSession(configuration: configuration, delegate: self, delegateQueue: nil)
    }
    
    func connect(url: URL, headers: [String: String] = [:]) {
        var request = URLRequest(url: url)
        request.setValue("text/event-stream", forHTTPHeaderField: "Accept")
        request.setValue("no-cache", forHTTPHeaderField: "Cache-Control")
        
        headers.forEach { request.setValue($1, forHTTPHeaderField: $0) }
        
        dataTask = urlSession.dataTask(with: request)
        dataTask?.resume()
    }
    
    func disconnect() {
        dataTask?.cancel()
    }
    
    func urlSession(_ session: URLSession, dataTask: URLSessionDataTask, didReceive data: Data) {
        guard let text = String(data: data, encoding: .utf8) else { return }
        buffer += text
        parseEvents()
    }
    
    private func parseEvents() {
        let events = buffer.components(separatedBy: "\n\n")
        buffer = events.last ?? ""
        
        for eventText in events.dropLast() {
            let lines = eventText.components(separatedBy: "\n")
            
            var id: String?
            var event: String?
            var data = ""
            
            for line in lines {
                if line.hasPrefix("id:") {
                    id = String(line.dropFirst(3)).trimmingCharacters(in: .whitespaces)
                } else if line.hasPrefix("event:") {
                    event = String(line.dropFirst(6)).trimmingCharacters(in: .whitespaces)
                } else if line.hasPrefix("data:") {
                    data += String(line.dropFirst(5)).trimmingCharacters(in: .whitespaces)
                }
            }
            
            if !data.isEmpty {
                let sseEvent = SSEEvent(id: id, event: event, data: data)
                DispatchQueue.main.async {
                    self.onEvent?(sseEvent)
                }
            }
        }
    }
}
```

### 8.3 Long Polling

Client ส่ง request แล้ว server hold ไว้จนมีข้อมูลใหม่หรือ timeout

```swift
// Long Polling
class LongPollingService {
    private var isPolling = false
    private let timeout: TimeInterval = 30
    
    func startPolling(url: URL, handler: @escaping (Data) -> Void) {
        isPolling = true
        Task {
            await poll(url: url, handler: handler)
        }
    }
    
    private func poll(url: URL, handler: @escaping (Data) -> Void) async {
        while isPolling {
            do {
                var request = URLRequest(url: url, timeoutInterval: timeout)
                let (data, _) = try await URLSession.shared.data(for: request)
                handler(data)
            } catch {
                // Timeout หรือ error - รอแล้ว poll ใหม่
                try? await Task.sleep(nanoseconds: 1_000_000_000) // 1 second
            }
        }
    }
    
    func stopPolling() {
        isPolling = false
    }
}
```

### 8.4 เปรียบเทียบ Real-time Approaches

| | WebSocket | SSE | Long Polling |
|---|-----------|-----|--------------|
| Direction | Bi-directional | Server → Client | Client pull |
| Overhead | Low | Low | High |
| Complexity | High | Medium | Low |
| Use Case | Chat, Games | Notifications, Feed | Simple updates |
| iOS Support | Native | Native | Native |

---

## 9. Image Loading และ Caching Architecture

### 9.1 Image Pipeline

```
URL → Disk Cache Check → Memory Cache Check → Network Download → Decode → Transform → Display
```

```swift
// Custom Image Loader (ตาม Kingfisher pattern)
actor ImageLoader {
    private var cache: [URL: UIImage] = [:]
    private var inFlight: [URL: Task<UIImage, Error>] = [:]
    private let diskCache: DiskCache
    
    init(diskCache: DiskCache = DiskCache(name: "images")) {
        self.diskCache = diskCache
    }
    
    func loadImage(from url: URL) async throws -> UIImage {
        // 1. Memory Cache
        if let cached = cache[url] {
            return cached
        }
        
        // 2. Dedup concurrent requests
        if let existing = inFlight[url] {
            return try await existing.value
        }
        
        // 3. Create new task
        let task = Task<UIImage, Error> {
            // Disk Cache
            if let data = diskCache.load(forKey: url.absoluteString),
               let image = UIImage(data: data) {
                cache[url] = image
                return image
            }
            
            // Network
            let (data, _) = try await URLSession.shared.data(from: url)
            guard let image = UIImage(data: data) else {
                throw ImageError.invalidData
            }
            
            // Save to caches
            try? diskCache.save(data: data, forKey: url.absoluteString)
            cache[url] = image
            
            return image
        }
        
        inFlight[url] = task
        
        defer {
            inFlight.removeValue(forKey: url)
        }
        
        return try await task.value
    }
    
    func prefetch(urls: [URL]) {
        for url in urls {
            Task {
                _ = try? await loadImage(from: url)
            }
        }
    }
    
    func clearCache() {
        cache.removeAll()
    }
}

// SwiftUI Image View
struct RemoteImage: View {
    let url: URL?
    var placeholder: Image = Image(systemName: "photo")
    
    @State private var image: UIImage?
    @State private var isLoading = false
    
    private static let loader = ImageLoader()
    
    var body: some View {
        Group {
            if let image = image {
                Image(uiImage: image)
                    .resizable()
            } else if isLoading {
                ProgressView()
            } else {
                placeholder
            }
        }
        .task(id: url) {
            await loadImage()
        }
    }
    
    private func loadImage() async {
        guard let url = url else { return }
        isLoading = true
        
        do {
            image = try await Self.loader.loadImage(from: url)
        } catch {
            // Show placeholder
        }
        
        isLoading = false
    }
}
```

### 9.2 Image Processing Pipeline

```swift
// Image Transform Pipeline
struct ImageProcessor {
    static func process(
        _ image: UIImage,
        options: ProcessingOptions
    ) -> UIImage {
        var result = image
        
        if let size = options.targetSize {
            result = resize(result, to: size)
        }
        
        if let radius = options.cornerRadius {
            result = applyCornerRadius(result, radius: radius)
        }
        
        if options.isCircle {
            result = cropToCircle(result)
        }
        
        return result
    }
    
    private static func resize(_ image: UIImage, to size: CGSize) -> UIImage {
        let renderer = UIGraphicsImageRenderer(size: size)
        return renderer.image { _ in
            image.draw(in: CGRect(origin: .zero, size: size))
        }
    }
    
    private static func cropToCircle(_ image: UIImage) -> UIImage {
        let size = image.size
        let renderer = UIGraphicsImageRenderer(size: size)
        
        return renderer.image { context in
            let rect = CGRect(origin: .zero, size: size)
            context.cgContext.addEllipse(in: rect)
            context.cgContext.clip()
            image.draw(in: rect)
        }
    }
    
    private static func applyCornerRadius(_ image: UIImage, radius: CGFloat) -> UIImage {
        let renderer = UIGraphicsImageRenderer(size: image.size)
        return renderer.image { context in
            let rect = CGRect(origin: .zero, size: image.size)
            UIBezierPath(roundedRect: rect, cornerRadius: radius).addClip()
            image.draw(in: rect)
        }
    }
    
    struct ProcessingOptions {
        var targetSize: CGSize?
        var cornerRadius: CGFloat?
        var isCircle: Bool = false
    }
}
```

---

## 10. Feed Design

### 10.1 Timeline Feed Architecture

```swift
// Feed ต้องรองรับ: infinite scroll, pull-to-refresh, optimistic updates
@MainActor
class FeedViewModel: ObservableObject {
    @Published var items: [FeedItem] = []
    @Published var isLoading = false
    @Published var hasMore = true
    
    private var cursor: String?
    private let pageSize = 20
    private let repository: FeedRepository
    
    init(repository: FeedRepository = FeedRepositoryImpl()) {
        self.repository = repository
    }
    
    func loadInitial() async {
        guard !isLoading else { return }
        isLoading = true
        cursor = nil
        
        do {
            let result = try await repository.getFeed(cursor: nil, limit: pageSize)
            items = result.items
            cursor = result.nextCursor
            hasMore = result.nextCursor != nil
        } catch {
            // Handle error
        }
        
        isLoading = false
    }
    
    func loadMore() async {
        guard !isLoading, hasMore, let cursor = cursor else { return }
        isLoading = true
        
        do {
            let result = try await repository.getFeed(cursor: cursor, limit: pageSize)
            items.append(contentsOf: result.items)
            self.cursor = result.nextCursor
            hasMore = result.nextCursor != nil
        } catch {
            // Handle error
        }
        
        isLoading = false
    }
    
    func refresh() async {
        cursor = nil
        await loadInitial()
    }
}

// Feed View with Infinite Scroll
struct FeedView: View {
    @StateObject var viewModel = FeedViewModel()
    
    var body: some View {
        List {
            ForEach(viewModel.items) { item in
                FeedItemView(item: item)
                    .onAppear {
                        if item.id == viewModel.items.last?.id {
                            Task { await viewModel.loadMore() }
                        }
                    }
            }
            
            if viewModel.isLoading {
                ProgressView()
                    .frame(maxWidth: .infinity)
            }
        }
        .refreshable {
            await viewModel.refresh()
        }
        .task {
            await viewModel.loadInitial()
        }
    }
}
```

### 10.2 Cursor-based Pagination

```swift
// Cursor-based vs Offset-based Pagination
// Offset: ?page=2&per_page=20 (ปัญหา: ถ้ามี item ใหม่ก็จะเกิด duplicate)
// Cursor: ?after=eyJpZCI6MTIzfQ&limit=20 (ดีกว่า stable)

struct PaginatedResponse<T: Decodable>: Decodable {
    let items: [T]
    let nextCursor: String?
    let previousCursor: String?
    let totalCount: Int?
}

// Cursor encoding (base64 ของ JSON)
struct Cursor {
    let id: String
    let createdAt: Date
    
    func encoded() -> String {
        let dict: [String: Any] = ["id": id, "created_at": createdAt.timeIntervalSince1970]
        let data = try! JSONSerialization.data(withJSONObject: dict)
        return data.base64EncodedString()
    }
    
    static func decode(_ string: String) -> Cursor? {
        guard let data = Data(base64Encoded: string),
              let dict = try? JSONSerialization.jsonObject(with: data) as? [String: Any],
              let id = dict["id"] as? String,
              let timestamp = dict["created_at"] as? TimeInterval else { return nil }
        
        return Cursor(id: id, createdAt: Date(timeIntervalSince1970: timestamp))
    }
}
```

---

## 11. Search Architecture

### 11.1 Local + Remote Search

```swift
// Search ที่ดีต้องมี:
// 1. Debounce - ไม่ search ทุก keystroke
// 2. Cancel previous request - เมื่อ query เปลี่ยน
// 3. Local cache - สำหรับ recent searches
// 4. Fallback - ถ้า remote ล้มเหลวใช้ local

@MainActor
class SearchViewModel: ObservableObject {
    @Published var query = ""
    @Published var results: [SearchResult] = []
    @Published var recentSearches: [String] = []
    @Published var isSearching = false
    
    private var searchTask: Task<Void, Never>?
    private let debounceDelay: Duration = .milliseconds(300)
    
    init() {
        loadRecentSearches()
        
        // Watch query changes
        $query
            .removeDuplicates()
            .sink { [weak self] query in
                self?.handleQueryChange(query)
            }
            .store(in: &cancellables)
    }
    
    private var cancellables = Set<AnyCancellable>()
    
    private func handleQueryChange(_ query: String) {
        searchTask?.cancel()
        
        if query.isEmpty {
            results = []
            return
        }
        
        searchTask = Task {
            try? await Task.sleep(for: debounceDelay)
            
            guard !Task.isCancelled else { return }
            
            await performSearch(query: query)
        }
    }
    
    private func performSearch(query: String) async {
        isSearching = true
        
        // 1. Local Search ก่อน (instant)
        let localResults = await searchLocally(query: query)
        if !localResults.isEmpty {
            results = localResults
        }
        
        // 2. Remote Search
        do {
            let remoteResults = try await APIClient.shared.search(query: query)
            results = remoteResults
        } catch {
            // ถ้า remote ล้มเหลว ใช้ local results
        }
        
        isSearching = false
    }
    
    private func searchLocally(query: String) async -> [SearchResult] {
        // Search ใน Core Data / SwiftData
        return []
    }
    
    func selectResult(_ result: SearchResult) {
        // Save to recent searches
        saveToRecentSearches(result.query)
    }
    
    private func saveToRecentSearches(_ query: String) {
        var recent = recentSearches
        recent.removeAll { $0 == query }
        recent.insert(query, at: 0)
        recentSearches = Array(recent.prefix(10))
        UserDefaults.standard.set(recentSearches, forKey: "recent_searches")
    }
    
    private func loadRecentSearches() {
        recentSearches = UserDefaults.standard.stringArray(forKey: "recent_searches") ?? []
    }
}
```

---

## 12. Chat Architecture

### 12.1 Chat System Design

```
Client → WebSocket → Message Queue → Message Service → Push Notification
                                   ↓
                              Message Store (Database)
                                   ↓
                         Other Clients (via WebSocket)
```

```swift
// Chat Message Model
struct ChatMessage: Codable, Identifiable {
    let id: String
    let conversationId: String
    let senderId: String
    let content: MessageContent
    let createdAt: Date
    var status: DeliveryStatus
    
    enum MessageContent: Codable {
        case text(String)
        case image(URL, thumbnail: URL?)
        case video(URL, duration: TimeInterval)
        case audio(URL, duration: TimeInterval)
        case location(latitude: Double, longitude: Double)
    }
    
    enum DeliveryStatus: String, Codable {
        case sending    // กำลังส่ง
        case sent       // ส่งถึง server แล้ว
        case delivered  // ส่งถึง recipient แล้ว
        case read       // recipient อ่านแล้ว
        case failed     // ส่งล้มเหลว
    }
}

// Chat View Model
@MainActor
class ChatViewModel: ObservableObject {
    @Published var messages: [ChatMessage] = []
    @Published var typingUsers: Set<String> = []
    
    private let conversationId: String
    private let webSocketManager: WebSocketManager
    private let messageRepository: MessageRepository
    
    init(conversationId: String) {
        self.conversationId = conversationId
        self.webSocketManager = WebSocketManager()
        self.messageRepository = MessageRepositoryImpl()
        
        setupWebSocket()
    }
    
    func sendMessage(_ content: ChatMessage.MessageContent) async {
        let message = ChatMessage(
            id: UUID().uuidString,
            conversationId: conversationId,
            senderId: currentUserId,
            content: content,
            createdAt: Date(),
            status: .sending
        )
        
        // Optimistic update
        messages.append(message)
        
        do {
            let sent = try await messageRepository.sendMessage(message)
            
            // Update status
            if let index = messages.firstIndex(where: { $0.id == message.id }) {
                messages[index] = sent
            }
        } catch {
            // Mark as failed
            if let index = messages.firstIndex(where: { $0.id == message.id }) {
                messages[index].status = .failed
            }
        }
    }
    
    private func setupWebSocket() {
        webSocketManager.onMessage = { [weak self] event in
            self?.handleWebSocketEvent(event)
        }
        
        webSocketManager.connect(
            url: URL(string: "wss://api.example.com/chat/\(conversationId)")!,
            token: authToken
        )
    }
    
    private func handleWebSocketEvent(_ event: WebSocketEvent) {
        switch event {
        case .newMessage(let message):
            if !messages.contains(where: { $0.id == message.id }) {
                messages.append(message)
                sendDeliveryReceipt(for: message)
            }
            
        case .messageStatusUpdate(let messageId, let status):
            if let index = messages.firstIndex(where: { $0.id == messageId }) {
                messages[index].status = status
            }
            
        case .typing(let userId):
            typingUsers.insert(userId)
            
        case .stopTyping(let userId):
            typingUsers.remove(userId)
        }
    }
    
    private func sendDeliveryReceipt(for message: ChatMessage) {
        webSocketManager.send(event: .delivered(messageId: message.id))
    }
    
    var currentUserId: String { "current_user" }
    var authToken: String { "token" }
}
```

---

## 13. Video Streaming Architecture

### 13.1 Adaptive Bitrate Streaming

```swift
import AVKit
import AVFoundation

class VideoPlayerManager: ObservableObject {
    @Published var player: AVPlayer?
    @Published var isBuffering = false
    @Published var progress: Double = 0
    
    private var timeObserver: Any?
    private var statusObserver: NSKeyValueObservation?
    
    func loadVideo(url: URL) {
        // สร้าง AVAsset สำหรับ HLS (HTTP Live Streaming)
        let asset = AVURLAsset(url: url, options: [
            AVURLAssetPreferPreciseDurationAndTimingKey: true
        ])
        
        let playerItem = AVPlayerItem(asset: asset)
        
        // Monitor buffering
        statusObserver = playerItem.observe(\.status) { [weak self] item, _ in
            DispatchQueue.main.async {
                switch item.status {
                case .readyToPlay:
                    self?.isBuffering = false
                case .failed:
                    print("Video failed: \(item.error?.localizedDescription ?? "")")
                default:
                    self?.isBuffering = true
                }
            }
        }
        
        player = AVPlayer(playerItem: playerItem)
        
        // Track progress
        let interval = CMTime(seconds: 0.5, preferredTimescale: CMTimeScale(NSEC_PER_SEC))
        timeObserver = player?.addPeriodicTimeObserver(forInterval: interval, queue: .main) { [weak self] time in
            guard let duration = self?.player?.currentItem?.duration.seconds,
                  duration > 0 else { return }
            self?.progress = time.seconds / duration
        }
    }
    
    func prefetchVideo(url: URL) {
        // Prefetch video ล่วงหน้าสำหรับ video ถัดไปใน feed
        let asset = AVURLAsset(url: url)
        asset.loadValuesAsynchronously(forKeys: ["playable"]) {
            // Asset is ready to play
        }
    }
    
    deinit {
        if let observer = timeObserver {
            player?.removeTimeObserver(observer)
        }
    }
}

// Video Feed Architecture (TikTok-style)
struct VideoFeedView: View {
    @StateObject var viewModel = VideoFeedViewModel()
    
    var body: some View {
        TabView(selection: $viewModel.currentIndex) {
            ForEach(Array(viewModel.videos.enumerated()), id: \.offset) { index, video in
                VideoPlayerView(video: video)
                    .tag(index)
                    .onAppear {
                        viewModel.onVideoAppear(at: index)
                    }
                    .onDisappear {
                        viewModel.onVideoDisappear(at: index)
                    }
            }
        }
        .tabViewStyle(.page(indexDisplayMode: .never))
        .ignoresSafeArea()
    }
}
```

---

## 14. E-Commerce App Architecture

### 14.1 High-level Architecture

```
┌────────────────────────────────────────────────┐
│                  iOS App                        │
│  ┌──────────┐ ┌──────────┐ ┌──────────────┐   │
│  │ Product  │ │  Cart    │ │   Checkout   │   │
│  │ Catalog  │ │ Manager  │ │   Flow       │   │
│  └──────────┘ └──────────┘ └──────────────┘   │
│  ┌──────────┐ ┌──────────┐ ┌──────────────┐   │
│  │  Order   │ │  User    │ │   Search     │   │
│  │ History  │ │ Profile  │ │   Filter     │   │
│  └──────────┘ └──────────┘ └──────────────┘   │
└────────────────────────────────────────────────┘
```

```swift
// Cart Manager - Singleton ที่จัดการ shopping cart
@MainActor
class CartManager: ObservableObject {
    static let shared = CartManager()
    
    @Published private(set) var items: [CartItem] = []
    @Published var isCheckingOut = false
    
    private let persistenceManager: CartPersistenceManager
    
    var totalPrice: Decimal {
        items.reduce(0) { $0 + ($1.product.price * Decimal($1.quantity)) }
    }
    
    var itemCount: Int {
        items.reduce(0) { $0 + $1.quantity }
    }
    
    func addItem(_ product: Product, quantity: Int = 1) {
        if let index = items.firstIndex(where: { $0.product.id == product.id }) {
            items[index].quantity += quantity
        } else {
            items.append(CartItem(product: product, quantity: quantity))
        }
        saveCart()
    }
    
    func removeItem(_ product: Product) {
        items.removeAll { $0.product.id == product.id }
        saveCart()
    }
    
    func updateQuantity(for product: Product, quantity: Int) {
        if quantity <= 0 {
            removeItem(product)
            return
        }
        
        if let index = items.firstIndex(where: { $0.product.id == product.id }) {
            items[index].quantity = quantity
            saveCart()
        }
    }
    
    func checkout() async throws -> Order {
        isCheckingOut = true
        defer { isCheckingOut = false }
        
        let order = try await OrderService.shared.createOrder(from: items)
        clearCart()
        return order
    }
    
    func clearCart() {
        items = []
        saveCart()
    }
    
    private func saveCart() {
        persistenceManager.save(items)
    }
    
    private init() {
        persistenceManager = CartPersistenceManager()
        items = persistenceManager.load()
    }
}

// Payment Processing
class PaymentProcessor {
    enum PaymentMethod {
        case creditCard(token: String)
        case promptPay(phoneNumber: String)
        case trueMoney(token: String)
        case linePay(token: String)
    }
    
    func processPayment(
        amount: Decimal,
        method: PaymentMethod,
        orderId: String
    ) async throws -> PaymentResult {
        // 1. Tokenize payment info (never store raw card data)
        // 2. Send to payment gateway
        // 3. Handle 3DS if required
        // 4. Return result
        
        return try await withThrowingTaskGroup(of: PaymentResult.self) { group in
            group.addTask {
                try await self.initiatePayment(amount: amount, method: method, orderId: orderId)
            }
            
            return try await group.next()!
        }
    }
    
    private func initiatePayment(amount: Decimal, method: PaymentMethod, orderId: String) async throws -> PaymentResult {
        fatalError("Implement with actual payment gateway SDK")
    }
}
```

---

## 15. Social Media App Architecture

### 15.1 Core Features Architecture

```swift
// Social Graph Manager
class SocialGraphManager {
    // Following/Followers management
    func follow(userId: String) async throws {
        // Optimistic update
        updateLocalFollowState(userId: userId, isFollowing: true)
        
        do {
            try await APIClient.shared.follow(userId: userId)
        } catch {
            // Revert optimistic update
            updateLocalFollowState(userId: userId, isFollowing: false)
            throw error
        }
    }
    
    func unfollow(userId: String) async throws {
        updateLocalFollowState(userId: userId, isFollowing: false)
        
        do {
            try await APIClient.shared.unfollow(userId: userId)
        } catch {
            updateLocalFollowState(userId: userId, isFollowing: true)
            throw error
        }
    }
    
    private func updateLocalFollowState(userId: String, isFollowing: Bool) {
        // Update SwiftData/Core Data
    }
}

// Like/React System
class ReactionManager {
    func toggleLike(postId: String) async throws {
        let currentState = getLikeState(for: postId)
        
        // Optimistic update
        updateLikeState(postId: postId, isLiked: !currentState.isLiked,
                        count: currentState.count + (currentState.isLiked ? -1 : 1))
        
        do {
            if currentState.isLiked {
                try await APIClient.shared.unlike(postId: postId)
            } else {
                try await APIClient.shared.like(postId: postId)
            }
        } catch {
            // Revert
            updateLikeState(postId: postId, isLiked: currentState.isLiked,
                           count: currentState.count)
            throw error
        }
    }
    
    private func getLikeState(for postId: String) -> LikeState {
        fatalError("Get from local database")
    }
    
    private func updateLikeState(postId: String, isLiked: Bool, count: Int) {
        // Update local database
    }
}

struct LikeState {
    let isLiked: Bool
    let count: Int
}
```

---

## 16. Maps/Location-based App Architecture

### 16.1 Location Service

```swift
import CoreLocation
import MapKit

class LocationManager: NSObject, ObservableObject, CLLocationManagerDelegate {
    @Published var currentLocation: CLLocation?
    @Published var authorizationStatus: CLAuthorizationStatus = .notDetermined
    @Published var isUpdatingLocation = false
    
    private let manager = CLLocationManager()
    
    override init() {
        super.init()
        manager.delegate = self
        manager.desiredAccuracy = kCLLocationAccuracyBest
        manager.distanceFilter = 10 // update ทุก 10 เมตร
    }
    
    func requestPermission() {
        manager.requestWhenInUseAuthorization()
    }
    
    func startUpdating() {
        manager.startUpdatingLocation()
        isUpdatingLocation = true
    }
    
    func stopUpdating() {
        manager.stopUpdatingLocation()
        isUpdatingLocation = false
    }
    
    // Single location request
    func requestOneTimeLocation() async throws -> CLLocation {
        return try await withCheckedThrowingContinuation { continuation in
            self.manager.requestLocation()
            self.locationContinuation = continuation
        }
    }
    
    private var locationContinuation: CheckedContinuation<CLLocation, Error>?
    
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        guard let location = locations.last else { return }
        currentLocation = location
        
        locationContinuation?.resume(returning: location)
        locationContinuation = nil
    }
    
    func locationManager(_ manager: CLLocationManager, didFailWithError error: Error) {
        locationContinuation?.resume(throwing: error)
        locationContinuation = nil
    }
    
    func locationManagerDidChangeAuthorization(_ manager: CLLocationManager) {
        authorizationStatus = manager.authorizationStatus
    }
}

// Geofencing
class GeofenceManager: NSObject, CLLocationManagerDelegate {
    private let manager = CLLocationManager()
    var onEnterRegion: ((CLCircularRegion) -> Void)?
    var onExitRegion: ((CLCircularRegion) -> Void)?
    
    func monitorRegion(center: CLLocationCoordinate2D, radius: CLLocationDistance, identifier: String) {
        let region = CLCircularRegion(center: center, radius: radius, identifier: identifier)
        region.notifyOnEntry = true
        region.notifyOnExit = true
        
        manager.startMonitoring(for: region)
    }
    
    func locationManager(_ manager: CLLocationManager, didEnterRegion region: CLRegion) {
        if let circular = region as? CLCircularRegion {
            onEnterRegion?(circular)
        }
    }
    
    func locationManager(_ manager: CLLocationManager, didExitRegion region: CLRegion) {
        if let circular = region as? CLCircularRegion {
            onExitRegion?(circular)
        }
    }
}
```

---

## 17. Authentication Architecture (OAuth2, PKCE)

### 17.1 OAuth2 with PKCE Flow

```swift
import CryptoKit
import AuthenticationServices

class OAuth2Manager: NSObject, ASWebAuthenticationPresentationContextProviding {
    
    private let clientId: String
    private let redirectUri: URL
    private let authEndpoint: URL
    private let tokenEndpoint: URL
    
    private var codeVerifier: String?
    
    init(clientId: String, redirectUri: URL, authEndpoint: URL, tokenEndpoint: URL) {
        self.clientId = clientId
        self.redirectUri = redirectUri
        self.authEndpoint = authEndpoint
        self.tokenEndpoint = tokenEndpoint
    }
    
    // Step 1: Generate PKCE parameters
    private func generateCodeVerifier() -> String {
        var buffer = [UInt8](repeating: 0, count: 32)
        _ = SecRandomCopyBytes(kSecRandomDefault, buffer.count, &buffer)
        return Data(buffer).base64URLEncoded()
    }
    
    private func generateCodeChallenge(from verifier: String) -> String {
        let data = verifier.data(using: .utf8)!
        let hashed = SHA256.hash(data: data)
        return Data(hashed).base64URLEncoded()
    }
    
    // Step 2: Build authorization URL
    private func buildAuthURL(codeChallenge: String, state: String) -> URL {
        var components = URLComponents(url: authEndpoint, resolvingAgainstBaseURL: false)!
        components.queryItems = [
            URLQueryItem(name: "response_type", value: "code"),
            URLQueryItem(name: "client_id", value: clientId),
            URLQueryItem(name: "redirect_uri", value: redirectUri.absoluteString),
            URLQueryItem(name: "scope", value: "openid profile email"),
            URLQueryItem(name: "state", value: state),
            URLQueryItem(name: "code_challenge", value: codeChallenge),
            URLQueryItem(name: "code_challenge_method", value: "S256")
        ]
        return components.url!
    }
    
    // Step 3: Start OAuth flow
    func startLogin() async throws -> TokenResponse {
        let verifier = generateCodeVerifier()
        let challenge = generateCodeChallenge(from: verifier)
        let state = UUID().uuidString
        self.codeVerifier = verifier
        
        let authURL = buildAuthURL(codeChallenge: challenge, state: state)
        
        // Open browser for login
        let callbackURL = try await withCheckedThrowingContinuation { continuation in
            let session = ASWebAuthenticationSession(
                url: authURL,
                callbackURLScheme: redirectUri.scheme
            ) { callbackURL, error in
                if let error = error {
                    continuation.resume(throwing: error)
                } else if let url = callbackURL {
                    continuation.resume(returning: url)
                }
            }
            
            session.presentationContextProvider = self
            session.prefersEphemeralWebBrowserSession = true
            session.start()
        }
        
        // Step 4: Extract authorization code
        let components = URLComponents(url: callbackURL, resolvingAgainstBaseURL: false)!
        guard let code = components.queryItems?.first(where: { $0.name == "code" })?.value,
              let returnedState = components.queryItems?.first(where: { $0.name == "state" })?.value,
              returnedState == state else {
            throw AuthError.invalidCallback
        }
        
        // Step 5: Exchange code for tokens
        return try await exchangeCode(code, verifier: verifier)
    }
    
    // Step 5: Token exchange
    private func exchangeCode(_ code: String, verifier: String) async throws -> TokenResponse {
        var request = URLRequest(url: tokenEndpoint)
        request.httpMethod = "POST"
        request.setValue("application/x-www-form-urlencoded", forHTTPHeaderField: "Content-Type")
        
        let params = [
            "grant_type": "authorization_code",
            "code": code,
            "redirect_uri": redirectUri.absoluteString,
            "client_id": clientId,
            "code_verifier": verifier
        ]
        
        request.httpBody = params
            .map { "\($0.key)=\($0.value)" }
            .joined(separator: "&")
            .data(using: .utf8)
        
        let (data, _) = try await URLSession.shared.data(for: request)
        return try JSONDecoder().decode(TokenResponse.self, from: data)
    }
    
    func presentationAnchor(for session: ASWebAuthenticationSession) -> ASPresentationAnchor {
        return UIApplication.shared.connectedScenes
            .compactMap { $0 as? UIWindowScene }
            .first?.windows.first ?? ASPresentationAnchor()
    }
}

struct TokenResponse: Codable {
    let accessToken: String
    let refreshToken: String?
    let expiresIn: Int
    let tokenType: String
    let idToken: String?
    
    enum CodingKeys: String, CodingKey {
        case accessToken = "access_token"
        case refreshToken = "refresh_token"
        case expiresIn = "expires_in"
        case tokenType = "token_type"
        case idToken = "id_token"
    }
}
```

---

## 18. Notification System Design

### 18.1 Push Notification Architecture

```swift
import UserNotifications
import FirebaseMessaging

class NotificationManager: NSObject, ObservableObject, UNUserNotificationCenterDelegate {
    @Published var pendingNotifications: [UNNotificationRequest] = []
    
    static let shared = NotificationManager()
    
    override init() {
        super.init()
        UNUserNotificationCenter.current().delegate = self
    }
    
    func requestPermission() async -> Bool {
        do {
            let granted = try await UNUserNotificationCenter.current().requestAuthorization(
                options: [.alert, .sound, .badge, .criticalAlert]
            )
            return granted
        } catch {
            return false
        }
    }
    
    // Register for remote notifications
    func registerForRemoteNotifications() {
        UIApplication.shared.registerForRemoteNotifications()
    }
    
    // Handle APNs token
    func handleAPNSToken(_ token: Data) {
        let tokenString = token.map { String(format: "%02.2hhx", $0) }.joined()
        print("APNs Token: \(tokenString)")
        
        // Send to server
        Task {
            try? await APIClient.shared.registerPushToken(tokenString)
        }
    }
    
    // Schedule local notification
    func scheduleLocalNotification(
        title: String,
        body: String,
        userInfo: [AnyHashable: Any] = [:],
        trigger: UNNotificationTrigger? = nil
    ) async throws {
        let content = UNMutableNotificationContent()
        content.title = title
        content.body = body
        content.sound = .default
        content.userInfo = userInfo
        
        let request = UNNotificationRequest(
            identifier: UUID().uuidString,
            content: content,
            trigger: trigger
        )
        
        try await UNUserNotificationCenter.current().add(request)
    }
    
    // Rich notification with image
    func scheduleRichNotification(imageURL: URL, title: String, body: String) async throws {
        let content = UNMutableNotificationContent()
        content.title = title
        content.body = body
        
        // Download and attach image
        let (imageData, _) = try await URLSession.shared.data(from: imageURL)
        let tempURL = FileManager.default.temporaryDirectory.appendingPathComponent("notification.jpg")
        try imageData.write(to: tempURL)
        
        let attachment = try UNNotificationAttachment(identifier: "image", url: tempURL)
        content.attachments = [attachment]
        
        let request = UNNotificationRequest(
            identifier: UUID().uuidString,
            content: content,
            trigger: nil
        )
        
        try await UNUserNotificationCenter.current().add(request)
    }
    
    // Foreground handling
    func userNotificationCenter(
        _ center: UNUserNotificationCenter,
        willPresent notification: UNNotification,
        withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void
    ) {
        completionHandler([.banner, .sound, .badge])
    }
    
    // Tap handling
    func userNotificationCenter(
        _ center: UNUserNotificationCenter,
        didReceive response: UNNotificationResponse,
        withCompletionHandler completionHandler: @escaping () -> Void
    ) {
        let userInfo = response.notification.request.content.userInfo
        
        // Deep link handling
        if let deepLink = userInfo["deep_link"] as? String {
            handleDeepLink(deepLink)
        }
        
        completionHandler()
    }
    
    private func handleDeepLink(_ link: String) {
        // Navigate to appropriate screen
        NotificationCenter.default.post(name: .handleDeepLink, object: link)
    }
}

extension Notification.Name {
    static let handleDeepLink = Notification.Name("handleDeepLink")
}
```

---

## 19. Analytics Event Pipeline Design

### 19.1 Analytics Architecture

```swift
// Analytics Manager ที่ batch events และส่งแบบ efficient
class AnalyticsManager {
    static let shared = AnalyticsManager()
    
    private var eventQueue: [AnalyticsEvent] = []
    private let batchSize = 50
    private let flushInterval: TimeInterval = 30
    private var flushTimer: Timer?
    private let lock = NSLock()
    
    struct AnalyticsEvent: Codable {
        let id: UUID
        let name: String
        let properties: [String: AnyCodable]
        let timestamp: Date
        let sessionId: String
        let userId: String?
        let deviceInfo: DeviceInfo
    }
    
    struct DeviceInfo: Codable {
        let os: String
        let osVersion: String
        let appVersion: String
        let deviceModel: String
        let locale: String
    }
    
    private init() {
        startFlushTimer()
        observeAppLifecycle()
    }
    
    func track(_ event: String, properties: [String: Any] = [:]) {
        let analyticsEvent = AnalyticsEvent(
            id: UUID(),
            name: event,
            properties: properties.mapValues { AnyCodable($0) },
            timestamp: Date(),
            sessionId: sessionId,
            userId: currentUserId,
            deviceInfo: currentDeviceInfo
        )
        
        lock.lock()
        eventQueue.append(analyticsEvent)
        let shouldFlush = eventQueue.count >= batchSize
        lock.unlock()
        
        if shouldFlush {
            Task { await flush() }
        }
    }
    
    func flush() async {
        lock.lock()
        let events = eventQueue
        eventQueue = []
        lock.unlock()
        
        guard !events.isEmpty else { return }
        
        do {
            try await APIClient.shared.sendAnalytics(events: events)
        } catch {
            // Re-queue events on failure
            lock.lock()
            eventQueue.insert(contentsOf: events, at: 0)
            lock.unlock()
        }
    }
    
    private func startFlushTimer() {
        flushTimer = Timer.scheduledTimer(withTimeInterval: flushInterval, repeats: true) { _ in
            Task { await AnalyticsManager.shared.flush() }
        }
    }
    
    private func observeAppLifecycle() {
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(appWillBackground),
            name: UIApplication.willResignActiveNotification,
            object: nil
        )
    }
    
    @objc private func appWillBackground() {
        // Flush before going background
        Task { await flush() }
    }
    
    var sessionId: String = UUID().uuidString
    var currentUserId: String? = nil
    var currentDeviceInfo: DeviceInfo {
        DeviceInfo(
            os: "iOS",
            osVersion: UIDevice.current.systemVersion,
            appVersion: Bundle.main.infoDictionary?["CFBundleShortVersionString"] as? String ?? "Unknown",
            deviceModel: UIDevice.current.model,
            locale: Locale.current.identifier
        )
    }
}

// Common Analytics Events
extension AnalyticsManager {
    func trackScreenView(_ screen: String) {
        track("screen_view", properties: ["screen_name": screen])
    }
    
    func trackButtonTap(_ button: String, screen: String) {
        track("button_tap", properties: ["button_name": button, "screen": screen])
    }
    
    func trackPurchase(orderId: String, amount: Decimal, items: [String]) {
        track("purchase", properties: [
            "order_id": orderId,
            "amount": amount,
            "items": items,
            "currency": "THB"
        ])
    }
    
    func trackError(_ error: Error, context: String) {
        track("error", properties: [
            "error_type": String(describing: type(of: error)),
            "error_message": error.localizedDescription,
            "context": context
        ])
    }
}
```

---

## 20. A/B Testing Architecture

### 20.1 Feature Flags และ A/B Testing

```swift
// Remote Config สำหรับ Feature Flags และ A/B Tests
class FeatureFlagManager {
    static let shared = FeatureFlagManager()
    
    private var flags: [String: FeatureFlag] = [:]
    private var userVariants: [String: String] = [:]
    
    struct FeatureFlag {
        let key: String
        let isEnabled: Bool
        let variants: [Variant]?
        
        struct Variant {
            let id: String
            let weight: Double  // 0.0 - 1.0
            let config: [String: Any]
        }
    }
    
    func fetchFlags() async throws {
        let response = try await APIClient.shared.getFeatureFlags(
            userId: currentUserId,
            deviceId: deviceId
        )
        
        // Store flags
        for flag in response.flags {
            flags[flag.key] = flag
        }
        
        // Assign variants
        userVariants = response.userVariants
    }
    
    func isEnabled(_ key: String) -> Bool {
        return flags[key]?.isEnabled ?? false
    }
    
    func variant(for key: String) -> String? {
        return userVariants[key]
    }
    
    func config(for key: String) -> [String: Any]? {
        guard let variantId = userVariants[key],
              let flag = flags[key],
              let variants = flag.variants,
              let variant = variants.first(where: { $0.id == variantId }) else { return nil }
        
        return variant.config
    }
    
    var currentUserId: String { "user_123" }
    var deviceId: String { UIDevice.current.identifierForVendor?.uuidString ?? "unknown" }
    
    private init() {}
}

// ใช้งานใน View
struct ProductView: View {
    @State private var showNewCheckoutFlow = FeatureFlagManager.shared.isEnabled("new_checkout_v2")
    
    var body: some View {
        VStack {
            if showNewCheckoutFlow {
                NewCheckoutButton()
            } else {
                LegacyCheckoutButton()
            }
        }
        .onAppear {
            let variant = FeatureFlagManager.shared.variant(for: "checkout_cta_text")
            AnalyticsManager.shared.track("checkout_cta_view", properties: ["variant": variant ?? "control"])
        }
    }
}
```

---

## 21. Error Monitoring and Reporting

### 21.1 Crash Reporting และ Error Monitoring

```swift
// Custom Error Reporter (ก่อนจะใช้ Sentry/Crashlytics)
class ErrorReporter {
    static let shared = ErrorReporter()
    
    private var breadcrumbs: [Breadcrumb] = []
    private let maxBreadcrumbs = 100
    
    struct Breadcrumb {
        let timestamp: Date
        let message: String
        let category: String
        let level: Level
        
        enum Level: String {
            case info, warning, error
        }
    }
    
    func addBreadcrumb(_ message: String, category: String = "app", level: Breadcrumb.Level = .info) {
        let crumb = Breadcrumb(
            timestamp: Date(),
            message: message,
            category: category,
            level: level
        )
        
        breadcrumbs.append(crumb)
        if breadcrumbs.count > maxBreadcrumbs {
            breadcrumbs.removeFirst()
        }
    }
    
    func captureError(_ error: Error, context: [String: Any] = [:]) {
        let report = ErrorReport(
            error: error,
            breadcrumbs: breadcrumbs,
            context: context,
            deviceInfo: DeviceInfoCollector.collect(),
            appState: AppStateCollector.collect()
        )
        
        Task {
            try? await sendReport(report)
        }
    }
    
    func captureMessage(_ message: String, level: String = "info") {
        addBreadcrumb(message, level: level == "error" ? .error : .info)
        
        Task {
            try? await APIClient.shared.logMessage(message: message, level: level)
        }
    }
    
    private func sendReport(_ report: ErrorReport) async throws {
        try await APIClient.shared.reportError(report)
    }
    
    private init() {}
}

struct ErrorReport {
    let error: Error
    let breadcrumbs: [ErrorReporter.Breadcrumb]
    let context: [String: Any]
    let deviceInfo: DeviceInfo
    let appState: AppState
}

struct DeviceInfo {
    static func collect() -> DeviceInfo { DeviceInfo() }
}

struct AppState {
    static func collect() -> AppState { AppState() }
}

// Global exception handler
class CrashHandler {
    static func setup() {
        // Handle uncaught exceptions
        NSSetUncaughtExceptionHandler { exception in
            let error = NSError(domain: "CrashDomain", code: 0, userInfo: [
                NSLocalizedDescriptionKey: exception.reason ?? "Unknown crash",
                "callStack": exception.callStackSymbols.joined(separator: "\n")
            ])
            ErrorReporter.shared.captureError(error)
        }
        
        // Handle signals
        signal(SIGABRT) { _ in
            ErrorReporter.shared.captureMessage("SIGABRT received", level: "fatal")
        }
    }
}
```

---

## 22. Practical Exercise: ออกแบบ Food Delivery App (Grab-like)

### 22.1 Requirements

**Functional Requirements:**
- User สามารถ browse ร้านอาหารได้
- User สามารถ search ร้านอาหารและ menu ได้
- User สามารถ add items ลง cart
- User สามารถ checkout และชำระเงินได้
- User สามารถ track order แบบ real-time ได้
- Driver รับ notification เมื่อมี order ใหม่

**Non-functional Requirements:**
- Response time < 200ms สำหรับ search
- Real-time order tracking
- รองรับ 1M concurrent users
- Offline support สำหรับ menu browsing

### 22.2 Architecture Design

```swift
// Core Models
struct Restaurant: Identifiable, Codable {
    let id: String
    let name: String
    let cuisine: [String]
    let rating: Double
    let deliveryTime: Int  // minutes
    let deliveryFee: Decimal
    let minimumOrder: Decimal
    let location: Coordinate
    let distance: Double?  // km from user
    var isOpen: Bool
    var promotions: [Promotion]
}

struct MenuItem: Identifiable, Codable {
    let id: String
    let restaurantId: String
    let name: String
    let description: String
    let price: Decimal
    let imageURL: URL?
    let category: String
    let isAvailable: Bool
    let options: [MenuOption]
}

struct Order: Identifiable, Codable {
    let id: String
    let restaurantId: String
    let userId: String
    var items: [OrderItem]
    var status: OrderStatus
    var driverLocation: Coordinate?
    let estimatedDelivery: Date
    
    enum OrderStatus: String, Codable {
        case pending
        case confirmed
        case preparing
        case readyForPickup
        case driverAssigned
        case driverPickedUp
        case delivering
        case delivered
        case cancelled
    }
}

// Real-time Order Tracking
class OrderTracker {
    private var webSocket: WebSocketManager?
    @Published var order: Order?
    @Published var driverLocation: CLLocationCoordinate2D?
    
    func startTracking(orderId: String) {
        webSocket = WebSocketManager()
        webSocket?.connect(
            url: URL(string: "wss://api.grab.com/tracking/\(orderId)")!,
            token: authToken
        )
        webSocket?.onMessage = { [weak self] event in
            self?.handleTrackingUpdate(event)
        }
    }
    
    private func handleTrackingUpdate(_ event: WebSocketEvent) {
        // Update UI based on event type
    }
    
    var authToken: String { "token" }
}

// Restaurant Discovery
class RestaurantDiscoveryService {
    func getNearbyRestaurants(
        location: CLLocationCoordinate2D,
        filters: RestaurantFilter = .default
    ) async throws -> [Restaurant] {
        let cacheKey = "restaurants_\(location.latitude)_\(location.longitude)"
        
        // Check cache (valid for 5 minutes)
        if let cached = await cacheManager.get(key: cacheKey, maxAge: 300) as? [Restaurant] {
            return applyFilters(cached, filters: filters)
        }
        
        let restaurants = try await apiClient.getNearbyRestaurants(
            lat: location.latitude,
            lng: location.longitude,
            radius: filters.radius
        )
        
        await cacheManager.set(restaurants, key: cacheKey)
        return applyFilters(restaurants, filters: filters)
    }
    
    private func applyFilters(_ restaurants: [Restaurant], filters: RestaurantFilter) -> [Restaurant] {
        var result = restaurants.filter { $0.isOpen }
        
        if let cuisines = filters.cuisines {
            result = result.filter { restaurant in
                !restaurant.cuisine.filter { cuisines.contains($0) }.isEmpty
            }
        }
        
        if let maxDeliveryFee = filters.maxDeliveryFee {
            result = result.filter { $0.deliveryFee <= maxDeliveryFee }
        }
        
        switch filters.sortBy {
        case .distance:
            result.sort { ($0.distance ?? .infinity) < ($1.distance ?? .infinity) }
        case .rating:
            result.sort { $0.rating > $1.rating }
        case .deliveryTime:
            result.sort { $0.deliveryTime < $1.deliveryTime }
        }
        
        return result
    }
    
    var cacheManager: MultiLevelCacheManager { .init() }
    var apiClient: RestaurantAPIClient { .init() }
}

struct RestaurantFilter {
    var cuisines: [String]?
    var maxDeliveryFee: Decimal?
    var sortBy: SortOption = .distance
    var radius: Double = 5.0  // km
    
    enum SortOption {
        case distance, rating, deliveryTime
    }
    
    static let `default` = RestaurantFilter()
}
```

---

## 23. Practical Exercise: ออกแบบ Messaging App

### 23.1 Chat Architecture Design

```swift
// Conversation Management
class ConversationManager: ObservableObject {
    @Published var conversations: [Conversation] = []
    
    struct Conversation: Identifiable {
        let id: String
        let participants: [User]
        var lastMessage: Message?
        var unreadCount: Int
        var isGroup: Bool
        var groupName: String?
        var groupAvatar: URL?
    }
    
    struct Message: Identifiable {
        let id: String
        let conversationId: String
        let senderId: String
        let content: MessageContent
        let timestamp: Date
        var reactions: [Reaction]
        var replyTo: Message?
        var isForwarded: Bool
        var deliveryStatus: DeliveryStatus
        
        enum MessageContent {
            case text(String)
            case image(URL)
            case video(URL, thumbnail: URL?, duration: TimeInterval)
            case audio(URL, duration: TimeInterval)
            case document(URL, filename: String, size: Int)
            case location(lat: Double, lng: Double)
            case sticker(URL)
        }
        
        struct Reaction {
            let emoji: String
            let count: Int
            let userIds: [String]
        }
    }
    
    // End-to-End Encryption
    struct E2EEncryption {
        // Key Exchange using X25519
        func generateKeyPair() -> (publicKey: Data, privateKey: Data) {
            // Generate Curve25519 key pair
            // This is simplified - use actual CryptoKit
            return (Data(), Data())
        }
        
        func encryptMessage(_ message: String, recipientPublicKey: Data, senderPrivateKey: Data) throws -> Data {
            // Use AES-256-GCM with shared secret
            fatalError("Implement with CryptoKit")
        }
        
        func decryptMessage(_ encryptedData: Data, senderPublicKey: Data, recipientPrivateKey: Data) throws -> String {
            fatalError("Implement with CryptoKit")
        }
    }
    
    // Message Delivery System
    func sendMessage(_ message: Message, to conversationId: String) async throws {
        // 1. Save locally (optimistic)
        addMessageLocally(message)
        
        // 2. Encrypt if E2E is enabled
        let payload = try preparePayload(for: message, conversationId: conversationId)
        
        // 3. Send via WebSocket (fast path)
        if webSocketManager.isConnected {
            webSocketManager.send(payload: payload)
        } else {
            // 4. Fallback to HTTP (reliable path)
            try await apiClient.sendMessage(payload)
        }
    }
    
    private func addMessageLocally(_ message: Message) {
        // Save to Core Data
    }
    
    private func preparePayload(for message: Message, conversationId: String) throws -> MessagePayload {
        fatalError("Implement encryption")
    }
    
    var webSocketManager: WebSocketManager { .init() }
    var apiClient: MessageAPIClient { .init() }
}
```

---

## 24. Summary

ในบทนี้เราได้เรียนรู้:

1. **System Design Basics** - หลักการพื้นฐานและ constraints ของ mobile

2. **Client-Server Architecture** - Layered architecture, Network layer

3. **REST vs GraphQL vs gRPC** - ข้อดีข้อเสียและเมื่อไหรควรใช้อะไร

4. **Caching Strategies** - Memory cache (NSCache), Disk cache, HTTP cache, Multi-level cache

5. **Offline-First Architecture** - Local-first database, Offline operation queue

6. **Sync Strategies** - Pessimistic locking, Optimistic locking, CRDT

7. **Real-time Updates** - WebSocket, SSE, Long polling เปรียบเทียบ

8. **Image Loading Architecture** - Image pipeline, caching, processing

9. **Feed Design** - Cursor-based pagination, Infinite scroll

10. **Search Architecture** - Debounce, Local + Remote search

11. **Chat Architecture** - WebSocket, Delivery receipts, E2E encryption

12. **Video Streaming** - AVPlayer, HLS, Adaptive bitrate

13. **E-Commerce Architecture** - Cart management, Payment processing

14. **Social Media Architecture** - Follow system, Optimistic updates

15. **Location-based Architecture** - CoreLocation, Geofencing

16. **Authentication** - OAuth2, PKCE flow

17. **Notifications** - APNs, Rich notifications, Deep links

18. **Analytics** - Event batching, Flush strategies

19. **A/B Testing** - Feature flags, Variant assignment

20. **Error Monitoring** - Crash reporting, Breadcrumbs

### Key Takeaways

- **ออกแบบ offline first** - สมมติว่า network จะหยุดทำงาน
- **Optimistic updates** - อัพเดท UI ก่อน แล้ว sync ทีหลัง เพื่อให้รู้สึก responsive
- **Cache aggressively** - แต่ต้องมี invalidation strategy ที่ดี
- **Event-driven architecture** - ใช้ WebSocket สำหรับ real-time เมื่อจำเป็น
- **Security first** - ใช้ PKCE, E2E encryption, และ secure storage

### แหล่งเรียนรู้เพิ่มเติม

- [System Design Interview Book](https://www.amazon.com/System-Design-Interview-insiders-Second/dp/B08CMF2CQF)
- [iOS App Architecture](https://www.objc.io/books/app-architecture/)
- [Designing Data-Intensive Applications](https://dataintensive.net/)
- [Mobile System Design (GitHub)](https://github.com/weeeBox/mobile-system-design)

---

*บทถัดไป: Part 72 - The Composable Architecture (TCA)*
