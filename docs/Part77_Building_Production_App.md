# Part 77: Building a Production App - การสร้างแอปพลิเคชันระดับ Production

## บทนำ

การสร้างแอปพลิเคชันระดับ Production ต่างจากการสร้างแอปสำหรับเรียนรู้หรือ Prototype อย่างมาก ต้องคำนึงถึงความเสถียร ประสิทธิภาพ ความปลอดภัย และประสบการณ์ของผู้ใช้ในทุกกรณีการใช้งาน บทนี้จะนำพาผ่านกระบวนการทั้งหมดตั้งแต่การวางแผนจนถึงการ Deploy และ Monitor

---

## 1. Project Planning สำหรับ Production App

### 1.1 การกำหนด Problem Statement

ก่อนเริ่มเขียนโค้ดแม้แต่บรรทัดเดียว ต้องเข้าใจปัญหาที่จะแก้ให้ชัดเจน:

```markdown
## Problem Statement Template

### ปัญหาที่พบ
- ผู้ใช้ไม่สามารถ... (What)
- เกิดขึ้นเมื่อ... (When)  
- ส่งผลต่อ... (Impact)

### กลุ่มเป้าหมาย
- Primary Users: นักช้อปออนไลน์ อายุ 18-45 ปี
- Secondary Users: ผู้ขายสินค้า

### ความสำเร็จที่วัดได้
- ลด Cart Abandonment Rate จาก 70% เป็น 45%
- เพิ่ม Conversion Rate เป็น 5%
- User Retention (Day 7) ≥ 40%
```

### 1.2 Competitive Analysis

```swift
// โครงสร้างข้อมูลสำหรับ Competitive Analysis
struct CompetitorAnalysis {
    let name: String
    let strengths: [String]
    let weaknesses: [String]
    let opportunities: [String]  // จุดที่เราจะแตกต่าง
    
    static let competitors: [CompetitorAnalysis] = [
        CompetitorAnalysis(
            name: "Lazada",
            strengths: ["Brand Recognition", "Large Product Catalog"],
            weaknesses: ["Complex UI", "Slow Search"],
            opportunities: ["Better UX", "Faster Search", "AR Try-On"]
        ),
        CompetitorAnalysis(
            name: "Shopee",
            strengths: ["Live Streaming", "Games/Rewards"],
            weaknesses: ["Too Cluttered", "Poor Recommendations"],
            opportunities: ["Clean UI", "AI-Powered Recommendations"]
        )
    ]
}
```

---

## 2. Feature Scope และ MVP Definition

### 2.1 MoSCoW Method

```markdown
## MVP Feature List - ShopSwift App

### Must Have (M) - ต้องมีใน MVP
- [ ] User Registration/Login (Email, Google, Apple)
- [ ] Product Listing with Search & Filter
- [ ] Product Detail Page
- [ ] Shopping Cart
- [ ] Checkout (PromptPay, Credit Card)
- [ ] Order Confirmation

### Should Have (S) - ควรมีใน Version 1.1
- [ ] Order History & Tracking
- [ ] Product Reviews & Ratings
- [ ] Wishlist
- [ ] Push Notifications
- [ ] Offline Browsing (Cached Products)

### Could Have (C) - อาจมีใน Version 2.0
- [ ] AR Product Try-On
- [ ] Live Shopping
- [ ] AI Recommendations
- [ ] Social Sharing

### Won't Have (W) - ไม่ทำในตอนนี้
- [ ] Seller Marketplace
- [ ] Auction System
- [ ] Crypto Payment
```

### 2.2 User Story Mapping

```swift
// User Journey สำหรับ Happy Path
enum UserJourney: CaseIterable {
    case discovery      // ค้นพบสินค้า
    case consideration  // พิจารณาซื้อ
    case purchase       // ซื้อสินค้า
    case postPurchase   // หลังซื้อ
    
    var stories: [String] {
        switch self {
        case .discovery:
            return [
                "ผู้ใช้ Browse สินค้า Featured",
                "ผู้ใช้ Search หาสินค้า",
                "ผู้ใช้ Filter ตาม Category/Price"
            ]
        case .consideration:
            return [
                "ผู้ใช้ดู Product Detail",
                "ผู้ใช้อ่าน Reviews",
                "ผู้ใช้เปรียบเทียบสินค้า"
            ]
        case .purchase:
            return [
                "ผู้ใช้เพิ่มสินค้าใน Cart",
                "ผู้ใช้ Checkout",
                "ผู้ใช้จ่ายเงิน",
                "ผู้ใช้รับ Order Confirmation"
            ]
        case .postPurchase:
            return [
                "ผู้ใช้ Track Order",
                "ผู้ใช้รับสินค้า",
                "ผู้ใช้เขียน Review"
            ]
        }
    }
}
```

---

## 3. Technical Requirements Document (TRD)

### 3.1 System Architecture Overview

```markdown
## Technical Requirements Document
Version: 1.0 | Date: 2024-01-15 | Author: iOS Team

### 1. Platform Requirements
- iOS 16.0+
- iPhone (Portrait Primary, Landscape Supported)
- iPad (Optional - Phase 2)
- Dark Mode Support: Required
- Dynamic Type Support: Required
- Accessibility (WCAG 2.1 AA): Required

### 2. Performance Requirements
- App Launch Time: < 2 seconds (Cold Start)
- Screen Transition: < 300ms
- API Response Time: < 500ms (P95)
- Image Load Time: < 1 second
- Offline Functionality: Core features work offline

### 3. Security Requirements
- All API calls via HTTPS with Certificate Pinning
- Sensitive data encrypted with Keychain
- Biometric Authentication support
- No sensitive data in logs
- OWASP Mobile Top 10 compliance

### 4. Scalability
- Support 100,000 concurrent users
- Handle 10,000 products in catalog
- Pagination: 20 items per page
```

### 3.2 API Contract Definition

```swift
// API Response Protocol
protocol APIResponse: Codable {
    var success: Bool { get }
    var message: String? { get }
}

// Generic API Result
struct APIResult<T: Codable>: APIResponse {
    let success: Bool
    let message: String?
    let data: T?
    let pagination: Pagination?
    
    struct Pagination: Codable {
        let currentPage: Int
        let totalPages: Int
        let totalItems: Int
        let itemsPerPage: Int
        
        enum CodingKeys: String, CodingKey {
            case currentPage = "current_page"
            case totalPages = "total_pages"
            case totalItems = "total_items"
            case itemsPerPage = "items_per_page"
        }
    }
}

// Error Response
struct APIError: APIResponse, Error {
    let success: Bool
    let message: String?
    let errorCode: String?
    let details: [String: String]?
    
    enum CodingKeys: String, CodingKey {
        case success
        case message
        case errorCode = "error_code"
        case details
    }
}
```

---

## 4. Project Setup Best Practices

### 4.1 Xcode Project Structure

```
ShopSwift/
├── App/
│   ├── ShopSwiftApp.swift
│   ├── AppDelegate.swift
│   └── SceneDelegate.swift
├── Core/
│   ├── Network/
│   │   ├── APIClient.swift
│   │   ├── Endpoint.swift
│   │   └── NetworkMonitor.swift
│   ├── Storage/
│   │   ├── CoreDataStack.swift
│   │   ├── KeychainManager.swift
│   │   └── UserDefaultsManager.swift
│   ├── DI/
│   │   └── DependencyContainer.swift
│   └── Extensions/
├── Features/
│   ├── Auth/
│   │   ├── Login/
│   │   ├── Register/
│   │   └── Profile/
│   ├── Catalog/
│   │   ├── ProductList/
│   │   ├── ProductDetail/
│   │   └── Search/
│   ├── Cart/
│   ├── Checkout/
│   └── Orders/
├── Shared/
│   ├── Components/
│   ├── Models/
│   └── Utils/
├── Resources/
│   ├── Assets.xcassets
│   ├── Localizable.strings
│   └── Info.plist
└── Tests/
    ├── UnitTests/
    └── UITests/
```

### 4.2 Build Configurations

```swift
// Configuration.swift
import Foundation

enum AppEnvironment: String {
    case development = "development"
    case staging = "staging"
    case production = "production"
    
    static var current: AppEnvironment {
        #if DEBUG
        return .development
        #elseif STAGING
        return .staging
        #else
        return .production
        #endif
    }
}

struct AppConfiguration {
    static let shared = AppConfiguration()
    
    let baseURL: URL
    let apiKey: String
    let sentryDSN: String
    let firebaseProjectID: String
    let certificatePinningKey: String
    let logLevel: LogLevel
    
    enum LogLevel: Int {
        case verbose = 0
        case debug = 1
        case info = 2
        case warning = 3
        case error = 4
        case none = 5
    }
    
    private init() {
        let env = AppEnvironment.current
        
        // โหลดค่าจาก .xcconfig
        let baseURLString = Bundle.main.infoDictionary?["BASE_URL"] as? String ?? ""
        self.baseURL = URL(string: baseURLString)!
        self.apiKey = Bundle.main.infoDictionary?["API_KEY"] as? String ?? ""
        self.sentryDSN = Bundle.main.infoDictionary?["SENTRY_DSN"] as? String ?? ""
        self.firebaseProjectID = Bundle.main.infoDictionary?["FIREBASE_PROJECT_ID"] as? String ?? ""
        self.certificatePinningKey = Bundle.main.infoDictionary?["CERT_PIN_KEY"] as? String ?? ""
        
        switch env {
        case .development:
            self.logLevel = .verbose
        case .staging:
            self.logLevel = .debug
        case .production:
            self.logLevel = .warning
        }
    }
}
```

### 4.3 xcconfig Files

```bash
# Development.xcconfig
BASE_URL = https://api-dev.shopswift.com
API_KEY = dev_key_here
SENTRY_DSN = https://sentry.io/dev/project
FIREBASE_PROJECT_ID = shopswift-dev
CERT_PIN_KEY = sha256/base64_dev_key

# Staging.xcconfig
BASE_URL = https://api-staging.shopswift.com
API_KEY = staging_key_here
SENTRY_DSN = https://sentry.io/staging/project
FIREBASE_PROJECT_ID = shopswift-staging
CERT_PIN_KEY = sha256/base64_staging_key

# Production.xcconfig
BASE_URL = https://api.shopswift.com
API_KEY = $(API_KEY_PROD)  # จาก CI/CD secrets
SENTRY_DSN = https://sentry.io/prod/project
FIREBASE_PROJECT_ID = shopswift-prod
CERT_PIN_KEY = sha256/base64_prod_key
```

---

## 5. Dependency Management Strategy

### 5.1 Swift Package Manager (SPM)

```swift
// Package.swift สำหรับ Dependencies
// swift-tools-version: 5.9

import PackageDescription

// ใน Xcode Project - เพิ่ม Dependencies ผ่าน File > Add Packages

// Dependencies ที่แนะนำสำหรับ Production App:

// 1. Networking
// Alamofire: https://github.com/Alamofire/Alamofire (5.9.0)

// 2. Image Loading
// Kingfisher: https://github.com/onevcat/Kingfisher (7.10.0)
// SDWebImage: https://github.com/SDWebImage/SDWebImage (5.18.0)

// 3. UI Components
// SnapKit: https://github.com/SnapKit/SnapKit (5.7.0) - ถ้าไม่ใช้ SwiftUI

// 4. Logging
// CocoaLumberjack: https://github.com/CocoaLumberjack/CocoaLumberjack (3.8.0)

// 5. Error Tracking
// Sentry: https://github.com/getsentry/sentry-cocoa (8.20.0)

// 6. Analytics & Remote Config
// Firebase: https://github.com/firebase/firebase-ios-sdk (10.19.0)

// 7. Keychain
// KeychainAccess: https://github.com/kishikawakatsumi/KeychainAccess (4.2.0)

// 8. JSON
// Swift Codable (Built-in) - ใช้ Built-in แทน SwiftyJSON

// 9. Testing
// Quick + Nimble: https://github.com/Quick/Quick (7.4.0)
// OHHTTPStubs: https://github.com/AliSoftware/OHHTTPStubs (9.1.0)
```

### 5.2 Dependency Injection Container

```swift
// DependencyContainer.swift
import Foundation

// Protocol สำหรับ Services
protocol NetworkServiceProtocol {
    func request<T: Decodable>(_ endpoint: Endpoint) async throws -> T
}

protocol StorageServiceProtocol {
    func save<T: Encodable>(_ value: T, forKey key: String) throws
    func load<T: Decodable>(forKey key: String) throws -> T
}

protocol AuthServiceProtocol {
    var isLoggedIn: Bool { get }
    func login(email: String, password: String) async throws -> User
    func logout() async throws
}

// DI Container
@MainActor
final class DependencyContainer: ObservableObject {
    static let shared = DependencyContainer()
    
    // Singleton Services
    lazy var networkService: NetworkServiceProtocol = {
        NetworkService(configuration: AppConfiguration.shared)
    }()
    
    lazy var storageService: StorageServiceProtocol = {
        StorageService()
    }()
    
    lazy var authService: AuthServiceProtocol = {
        AuthService(networkService: networkService, storageService: storageService)
    }()
    
    lazy var analyticsService: AnalyticsServiceProtocol = {
        AnalyticsService()
    }()
    
    // Feature-specific factories
    func makeProductListViewModel() -> ProductListViewModel {
        ProductListViewModel(
            productRepository: ProductRepository(networkService: networkService),
            analyticsService: analyticsService
        )
    }
    
    func makeCartViewModel() -> CartViewModel {
        CartViewModel(
            cartRepository: CartRepository(storageService: storageService),
            analyticsService: analyticsService
        )
    }
    
    private init() {}
}
```

---

## 6. Git Branching Strategy

### 6.1 Git Flow สำหรับ iOS Team

```bash
# Branch Structure
main          # Production-ready code เท่านั้น
├── develop   # Integration branch
├── release/1.0.0   # Release preparation
├── feature/SHOP-123-product-list
├── feature/SHOP-124-cart
├── fix/SHOP-200-payment-crash
└── hotfix/SHOP-201-critical-auth-bug

# Commit Convention
# feat: เพิ่มฟีเจอร์ใหม่
# fix: แก้ Bug
# refactor: Refactor โค้ด
# test: เพิ่ม/แก้ไข Tests
# docs: อัปเดต Documentation
# chore: งาน Maintenance

# ตัวอย่าง Commit Messages
git commit -m "feat(SHOP-123): add product list with infinite scroll"
git commit -m "fix(SHOP-200): resolve crash when payment fails"
git commit -m "refactor(cart): extract cart total calculation"
```

### 6.2 Pull Request Template

```markdown
# Pull Request: [TICKET-NUMBER] Title

## Description
สรุปการเปลี่ยนแปลงที่ทำ และเหตุผล

## Type of Change
- [ ] Bug fix (non-breaking change)
- [ ] New feature (non-breaking change)
- [ ] Breaking change
- [ ] Refactoring
- [ ] Documentation update

## How to Test
1. เปิด app
2. ไปที่หน้า Product List
3. ตรวจสอบว่า...

## Screenshots (ถ้ามี UI changes)
| Before | After |
|--------|-------|
| img    | img   |

## Checklist
- [ ] Tests ผ่านทั้งหมด
- [ ] Code ผ่าน SwiftLint
- [ ] เพิ่ม/อัปเดต Unit Tests
- [ ] ไม่มี print statements ในโค้ด Production
- [ ] Localization strings อัปเดตแล้ว
```

---

## 7. Environment Configuration

### 7.1 Multi-Environment Setup

```swift
// EnvironmentManager.swift
import Foundation

struct EnvironmentConfig {
    // Network
    let apiBaseURL: URL
    let apiVersion: String
    let requestTimeout: TimeInterval
    let maxRetries: Int
    
    // Feature Flags
    let enableCrashReporting: Bool
    let enableAnalytics: Bool
    let enablePushNotifications: Bool
    let enableARFeatures: Bool
    
    // Cache
    let diskCacheSize: Int  // bytes
    let memoryCacheSize: Int  // bytes
    let cacheExpirationTime: TimeInterval
    
    // Debug
    let showNetworkLogs: Bool
    let showPerformanceLogs: Bool
    
    static let development = EnvironmentConfig(
        apiBaseURL: URL(string: "https://api-dev.shopswift.com/v1")!,
        apiVersion: "v1",
        requestTimeout: 60,
        maxRetries: 3,
        enableCrashReporting: false,
        enableAnalytics: false,
        enablePushNotifications: true,
        enableARFeatures: true,
        diskCacheSize: 500 * 1024 * 1024,  // 500 MB
        memoryCacheSize: 100 * 1024 * 1024,  // 100 MB
        cacheExpirationTime: 3600,  // 1 hour
        showNetworkLogs: true,
        showPerformanceLogs: true
    )
    
    static let staging = EnvironmentConfig(
        apiBaseURL: URL(string: "https://api-staging.shopswift.com/v1")!,
        apiVersion: "v1",
        requestTimeout: 30,
        maxRetries: 2,
        enableCrashReporting: true,
        enableAnalytics: false,
        enablePushNotifications: true,
        enableARFeatures: true,
        diskCacheSize: 200 * 1024 * 1024,
        memoryCacheSize: 50 * 1024 * 1024,
        cacheExpirationTime: 1800,
        showNetworkLogs: true,
        showPerformanceLogs: false
    )
    
    static let production = EnvironmentConfig(
        apiBaseURL: URL(string: "https://api.shopswift.com/v1")!,
        apiVersion: "v1",
        requestTimeout: 30,
        maxRetries: 2,
        enableCrashReporting: true,
        enableAnalytics: true,
        enablePushNotifications: true,
        enableARFeatures: false,  // ยังไม่พร้อม Production
        diskCacheSize: 200 * 1024 * 1024,
        memoryCacheSize: 50 * 1024 * 1024,
        cacheExpirationTime: 1800,
        showNetworkLogs: false,
        showPerformanceLogs: false
    )
}
```

---

## 8. Deep Linking Setup

### 8.1 Universal Links Setup

Universal Links ช่วยให้ link ปกติบนเว็บเปิด App ได้โดยตรง

```json
// apple-app-site-association (AASA) file
// วางไว้ที่ https://shopswift.com/.well-known/apple-app-site-association
{
    "applinks": {
        "details": [
            {
                "appIDs": ["TEAMID123.com.shopswift.app"],
                "components": [
                    {
                        "/": "/products/*",
                        "comment": "Product detail pages"
                    },
                    {
                        "/": "/categories/*",
                        "comment": "Category pages"
                    },
                    {
                        "/": "/orders/*",
                        "comment": "Order tracking"
                    },
                    {
                        "/": "/promotions/*",
                        "comment": "Promotion pages"
                    }
                ]
            }
        ]
    },
    "webcredentials": {
        "apps": ["TEAMID123.com.shopswift.app"]
    }
}
```

```swift
// DeepLinkManager.swift
import UIKit
import SwiftUI

enum DeepLink: Equatable {
    case product(id: String)
    case category(slug: String)
    case order(id: String)
    case promotion(code: String)
    case profile
    case cart
    case unknown
    
    init(url: URL) {
        let components = URLComponents(url: url, resolvingAgainstBaseURL: true)
        let host = url.host
        let pathComponents = url.pathComponents.filter { $0 != "/" }
        
        // Universal Links: https://shopswift.com/products/123
        if host == "shopswift.com" || host == "www.shopswift.com" {
            self = DeepLink.parse(path: pathComponents, queryItems: components?.queryItems)
            return
        }
        
        // Custom URL Scheme: shopswift://products/123
        if url.scheme == "shopswift" {
            let host = url.host ?? ""
            let path = url.pathComponents.filter { $0 != "/" }
            self = DeepLink.parseScheme(host: host, path: path, queryItems: components?.queryItems)
            return
        }
        
        self = .unknown
    }
    
    private static func parse(path: [String], queryItems: [URLQueryItem]?) -> DeepLink {
        guard !path.isEmpty else { return .unknown }
        
        switch path[0] {
        case "products" where path.count > 1:
            return .product(id: path[1])
        case "categories" where path.count > 1:
            return .category(slug: path[1])
        case "orders" where path.count > 1:
            return .order(id: path[1])
        case "promotions":
            let code = queryItems?.first(where: { $0.name == "code" })?.value ?? ""
            return .promotion(code: code)
        default:
            return .unknown
        }
    }
    
    private static func parseScheme(host: String, path: [String], queryItems: [URLQueryItem]?) -> DeepLink {
        switch host {
        case "product" where !path.isEmpty:
            return .product(id: path[0])
        case "cart":
            return .cart
        case "profile":
            return .profile
        default:
            return .unknown
        }
    }
}

// DeepLink Router
@MainActor
class DeepLinkRouter: ObservableObject {
    @Published var activeDeepLink: DeepLink?
    
    static let shared = DeepLinkRouter()
    
    func handle(_ url: URL) {
        let link = DeepLink(url: url)
        guard link != .unknown else { return }
        
        AnalyticsService.shared.track(.deepLinkOpened(link))
        activeDeepLink = link
    }
}

// SwiftUI Integration
struct ShopSwiftApp: App {
    @StateObject private var deepLinkRouter = DeepLinkRouter.shared
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(deepLinkRouter)
                .onOpenURL { url in
                    deepLinkRouter.handle(url)
                }
        }
    }
}
```

### 8.2 Custom URL Scheme

```swift
// Info.plist เพิ่ม URL Types
// CFBundleURLTypes > CFBundleURLSchemes > shopswift

// ใน AppDelegate
func application(_ app: UIApplication, 
                 open url: URL, 
                 options: [UIApplication.OpenURLOptionsKey: Any] = [:]) -> Bool {
    // จัดการ URL Scheme
    if url.scheme == "shopswift" {
        DeepLinkRouter.shared.handle(url)
        return true
    }
    return false
}

// Scene-based handling
func scene(_ scene: UIScene, 
           openURLContexts URLContexts: Set<UIOpenURLContext>) {
    guard let url = URLContexts.first?.url else { return }
    DeepLinkRouter.shared.handle(url)
}

func scene(_ scene: UIScene, 
           continue userActivity: NSUserActivity) {
    guard userActivity.activityType == NSUserActivityTypeBrowsingWeb,
          let url = userActivity.webpageURL else { return }
    DeepLinkRouter.shared.handle(url)
}
```

---

## 9. App Lifecycle Management

### 9.1 AppDelegate Setup

```swift
// AppDelegate.swift
import UIKit
import Firebase
import UserNotifications

@main
class AppDelegate: UIResponder, UIApplicationDelegate {
    
    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        
        // ลำดับการ Initialize สำคัญมาก
        setupLogging()
        setupCrashReporting()
        setupAnalytics()
        setupPushNotifications(application)
        setupAppearance()
        
        // จัดการ Launch Options
        if let notification = launchOptions?[.remoteNotification] as? [String: Any] {
            PushNotificationManager.shared.handleLaunchNotification(notification)
        }
        
        return true
    }
    
    private func setupLogging() {
        Logger.shared.configure(level: AppConfiguration.shared.logLevel)
        Logger.info("App launched - Version: \(AppInfo.version) Build: \(AppInfo.build)")
    }
    
    private func setupCrashReporting() {
        guard AppConfiguration.shared.enableCrashReporting else { return }
        SentrySDK.start { options in
            options.dsn = AppConfiguration.shared.sentryDSN
            options.environment = AppEnvironment.current.rawValue
            options.release = "\(AppInfo.bundleId)@\(AppInfo.version)+\(AppInfo.build)"
            options.enableAutoPerformanceTracing = true
            options.tracesSampleRate = 0.2
            options.profilesSampleRate = 0.1
        }
    }
    
    private func setupAnalytics() {
        guard AppConfiguration.shared.enableAnalytics else { return }
        FirebaseApp.configure()
        Analytics.setAnalyticsCollectionEnabled(true)
        
        // Set User Properties
        if let userId = AuthManager.shared.currentUser?.id {
            Analytics.setUserID(userId)
        }
    }
    
    private func setupPushNotifications(_ application: UIApplication) {
        UNUserNotificationCenter.current().delegate = self
        application.registerForRemoteNotifications()
    }
    
    private func setupAppearance() {
        // Global UI Appearance
        let navBarAppearance = UINavigationBarAppearance()
        navBarAppearance.configureWithOpaqueBackground()
        navBarAppearance.backgroundColor = .systemBackground
        navBarAppearance.titleTextAttributes = [
            .font: UIFont.systemFont(ofSize: 17, weight: .semibold)
        ]
        UINavigationBar.appearance().standardAppearance = navBarAppearance
        UINavigationBar.appearance().scrollEdgeAppearance = navBarAppearance
        
        let tabBarAppearance = UITabBarAppearance()
        tabBarAppearance.configureWithOpaqueBackground()
        UITabBar.appearance().standardAppearance = tabBarAppearance
        UITabBar.appearance().scrollEdgeAppearance = tabBarAppearance
    }
}

// AppDelegate + UNUserNotificationCenterDelegate
extension AppDelegate: UNUserNotificationCenterDelegate {
    func userNotificationCenter(_ center: UNUserNotificationCenter,
                                willPresent notification: UNNotification,
                                withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
        completionHandler([.banner, .sound, .badge])
    }
    
    func userNotificationCenter(_ center: UNUserNotificationCenter,
                                didReceive response: UNNotificationResponse,
                                withCompletionHandler completionHandler: @escaping () -> Void) {
        PushNotificationManager.shared.handleNotificationResponse(response)
        completionHandler()
    }
    
    func application(_ application: UIApplication,
                     didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
        let token = deviceToken.map { String(format: "%02.2hhx", $0) }.joined()
        PushNotificationManager.shared.registerDeviceToken(token)
    }
}
```

### 9.2 Scene Lifecycle

```swift
// SceneDelegate.swift
import UIKit
import SwiftUI

class SceneDelegate: UIResponder, UIWindowSceneDelegate {
    var window: UIWindow?
    
    func scene(_ scene: UIScene,
               willConnectTo session: UISceneSession,
               options connectionOptions: UIScene.ConnectionOptions) {
        guard let windowScene = scene as? UIWindowScene else { return }
        
        let window = UIWindow(windowScene: windowScene)
        let container = DependencyContainer.shared
        
        let contentView = RootView()
            .environmentObject(container)
        
        window.rootViewController = UIHostingController(rootView: contentView)
        self.window = window
        window.makeKeyAndVisible()
        
        // จัดการ URL จาก Launch
        if let url = connectionOptions.urlContexts.first?.url {
            DeepLinkRouter.shared.handle(url)
        }
        
        // จัดการ User Activity (Universal Links)
        if let userActivity = connectionOptions.userActivities.first {
            self.scene(scene, continue: userActivity)
        }
    }
    
    func sceneWillResignActive(_ scene: UIScene) {
        // App จะหยุดรับ Events - บันทึก State
        StateRestorationManager.shared.saveCurrentState()
    }
    
    func sceneDidEnterBackground(_ scene: UIScene) {
        // App เข้า Background - Schedule Background Tasks
        BackgroundTaskManager.shared.scheduleBackgroundRefresh()
        BackgroundTaskManager.shared.scheduleDatabaseCleanup()
        
        // Clear Sensitive UI (e.g. Credit card numbers)
        NotificationCenter.default.post(name: .appDidEnterBackground, object: nil)
    }
    
    func sceneWillEnterForeground(_ scene: UIScene) {
        // App จะกลับมา Foreground
        NotificationCenter.default.post(name: .appWillEnterForeground, object: nil)
    }
    
    func sceneDidBecomeActive(_ scene: UIScene) {
        // App Active แล้ว
        UIApplication.shared.applicationIconBadgeNumber = 0
        
        // Refresh Token ถ้าจำเป็น
        Task {
            try? await AuthManager.shared.refreshTokenIfNeeded()
        }
        
        // Sync data
        Task {
            await SyncManager.shared.syncIfNeeded()
        }
    }
}
```

---

## 10. Background Task Management (BGTaskScheduler)

### 10.1 การ Setup BGTaskScheduler

```swift
// BackgroundTaskManager.swift
import BackgroundTasks
import UIKit

class BackgroundTaskManager {
    static let shared = BackgroundTaskManager()
    
    // Task Identifiers - ต้องเพิ่มใน Info.plist ด้วย
    enum TaskIdentifier {
        static let appRefresh = "com.shopswift.app.refresh"
        static let dbCleanup = "com.shopswift.db.cleanup"
        static let syncOrders = "com.shopswift.sync.orders"
    }
    
    func registerBackgroundTasks() {
        // ลงทะเบียน App Refresh Task (สำหรับ Update Content)
        BGTaskScheduler.shared.register(
            forTaskWithIdentifier: TaskIdentifier.appRefresh,
            using: nil
        ) { [weak self] task in
            self?.handleAppRefresh(task: task as! BGAppRefreshTask)
        }
        
        // ลงทะเบียน Processing Task (สำหรับงานหนัก)
        BGTaskScheduler.shared.register(
            forTaskWithIdentifier: TaskIdentifier.dbCleanup,
            using: nil
        ) { [weak self] task in
            self?.handleDatabaseCleanup(task: task as! BGProcessingTask)
        }
        
        BGTaskScheduler.shared.register(
            forTaskWithIdentifier: TaskIdentifier.syncOrders,
            using: nil
        ) { [weak self] task in
            self?.handleOrderSync(task: task as! BGProcessingTask)
        }
    }
    
    // Schedule App Refresh (ทุก 15 นาที)
    func scheduleBackgroundRefresh() {
        let request = BGAppRefreshTaskRequest(identifier: TaskIdentifier.appRefresh)
        request.earliestBeginDate = Date(timeIntervalSinceNow: 15 * 60)
        
        do {
            try BGTaskScheduler.shared.submit(request)
            Logger.debug("Background refresh scheduled")
        } catch BGTaskScheduler.Error.notPermitted {
            Logger.warning("Background refresh not permitted")
        } catch {
            Logger.error("Failed to schedule background refresh: \(error)")
        }
    }
    
    // Schedule Database Cleanup (ทุกคืน)
    func scheduleDatabaseCleanup() {
        let request = BGProcessingTaskRequest(identifier: TaskIdentifier.dbCleanup)
        request.earliestBeginDate = Date(timeIntervalSinceNow: 60 * 60)  // 1 hour from now
        request.requiresNetworkConnectivity = false
        request.requiresExternalPower = false
        
        do {
            try BGTaskScheduler.shared.submit(request)
        } catch {
            Logger.error("Failed to schedule DB cleanup: \(error)")
        }
    }
    
    // Handle App Refresh
    private func handleAppRefresh(task: BGAppRefreshTask) {
        // Schedule next refresh ก่อนเสมอ
        scheduleBackgroundRefresh()
        
        let operation = RefreshOperation()
        
        task.expirationHandler = {
            operation.cancel()
        }
        
        operation.completionBlock = {
            task.setTaskCompleted(success: !operation.isCancelled)
        }
        
        OperationQueue.main.addOperation(operation)
    }
    
    // Handle Database Cleanup
    private func handleDatabaseCleanup(task: BGProcessingTask) {
        Task {
            do {
                // ลบ Orders เก่ากว่า 30 วัน
                try await CoreDataStack.shared.deleteOldOrders(olderThan: 30)
                // ลบ Cache ที่หมดอายุ
                try await CacheManager.shared.clearExpiredCache()
                task.setTaskCompleted(success: true)
            } catch {
                Logger.error("DB Cleanup failed: \(error)")
                task.setTaskCompleted(success: false)
            }
        }
    }
    
    // Handle Order Sync
    private func handleOrderSync(task: BGProcessingTask) {
        task.expirationHandler = {
            task.setTaskCompleted(success: false)
        }
        
        Task {
            do {
                try await OrderSyncService.shared.syncPendingOrders()
                task.setTaskCompleted(success: true)
            } catch {
                task.setTaskCompleted(success: false)
            }
        }
    }
}

// Info.plist - BGTaskSchedulerPermittedIdentifiers
// <array>
//     <string>com.shopswift.app.refresh</string>
//     <string>com.shopswift.db.cleanup</string>
//     <string>com.shopswift.sync.orders</string>
// </array>
```

---

## 11. State Restoration

### 11.1 StateRestorationManager

```swift
// StateRestorationManager.swift
import Foundation

struct AppState: Codable {
    var selectedTab: Int
    var navigationPath: [String]  // Encoded navigation state
    var lastVisitedProductID: String?
    var cartItemCount: Int
    var searchQuery: String?
    var timestamp: Date
    
    static var `default`: AppState {
        AppState(
            selectedTab: 0,
            navigationPath: [],
            lastVisitedProductID: nil,
            cartItemCount: 0,
            searchQuery: nil,
            timestamp: Date()
        )
    }
}

class StateRestorationManager {
    static let shared = StateRestorationManager()
    
    private let stateKey = "app_state"
    private let maxStateAge: TimeInterval = 24 * 60 * 60  // 24 hours
    
    func saveCurrentState() {
        // รวบรวม State จาก App
        let state = AppState(
            selectedTab: TabBarManager.shared.selectedIndex,
            navigationPath: NavigationManager.shared.encodedPath,
            lastVisitedProductID: NavigationManager.shared.lastProductID,
            cartItemCount: CartManager.shared.itemCount,
            searchQuery: SearchManager.shared.lastQuery,
            timestamp: Date()
        )
        
        if let encoded = try? JSONEncoder().encode(state) {
            UserDefaults.standard.set(encoded, forKey: stateKey)
        }
    }
    
    func restoreState() -> AppState? {
        guard let data = UserDefaults.standard.data(forKey: stateKey),
              let state = try? JSONDecoder().decode(AppState.self, from: data) else {
            return nil
        }
        
        // ตรวจสอบว่า State ไม่เก่าเกินไป
        guard Date().timeIntervalSince(state.timestamp) < maxStateAge else {
            clearState()
            return nil
        }
        
        return state
    }
    
    func clearState() {
        UserDefaults.standard.removeObject(forKey: stateKey)
    }
}

// SwiftUI State Restoration
struct ContentView: View {
    @StateObject private var restorationManager = StateRestorationManager.shared
    @State private var selectedTab = 0
    
    var body: some View {
        TabView(selection: $selectedTab) {
            HomeView()
                .tabItem { Label("หน้าแรก", systemImage: "house.fill") }
                .tag(0)
            
            CatalogView()
                .tabItem { Label("สินค้า", systemImage: "square.grid.2x2.fill") }
                .tag(1)
            
            CartView()
                .tabItem { Label("ตะกร้า", systemImage: "cart.fill") }
                .tag(2)
            
            ProfileView()
                .tabItem { Label("โปรไฟล์", systemImage: "person.fill") }
                .tag(3)
        }
        .onAppear {
            if let state = StateRestorationManager.shared.restoreState() {
                selectedTab = state.selectedTab
            }
        }
        .onChange(of: selectedTab) { _ in
            StateRestorationManager.shared.saveCurrentState()
        }
    }
}
```

---

## 12. Error Reporting Integration (Sentry)

### 12.1 Custom Error Types และ Sentry Integration

```swift
// ErrorReporting.swift
import Sentry

// Custom Error Types
enum AppError: LocalizedError {
    case networkError(underlying: Error, endpoint: String)
    case authenticationFailed(reason: String)
    case paymentFailed(code: String, message: String)
    case invalidData(field: String, expected: String)
    case serverError(statusCode: Int, message: String)
    case unexpectedError(message: String)
    
    var errorDescription: String? {
        switch self {
        case .networkError(_, let endpoint):
            return "เกิดปัญหาการเชื่อมต่อที่ \(endpoint)"
        case .authenticationFailed(let reason):
            return "การเข้าสู่ระบบล้มเหลว: \(reason)"
        case .paymentFailed(_, let message):
            return "การชำระเงินล้มเหลว: \(message)"
        case .invalidData(let field, _):
            return "ข้อมูลไม่ถูกต้อง: \(field)"
        case .serverError(let code, let message):
            return "เซิร์ฟเวอร์ผิดพลาด (\(code)): \(message)"
        case .unexpectedError(let message):
            return "เกิดข้อผิดพลาดที่ไม่คาดคิด: \(message)"
        }
    }
    
    var sentryLevel: SentryLevel {
        switch self {
        case .networkError: return .warning
        case .authenticationFailed: return .error
        case .paymentFailed: return .error
        case .invalidData: return .warning
        case .serverError: return .error
        case .unexpectedError: return .fatal
        }
    }
}

// Error Reporter
class ErrorReporter {
    static let shared = ErrorReporter()
    
    func report(_ error: AppError, context: [String: Any] = [:]) {
        // Log locally
        Logger.error("[\(type(of: error))] \(error.localizedDescription)")
        
        guard AppConfiguration.shared.enableCrashReporting else { return }
        
        // Report to Sentry
        SentrySDK.capture(error: error) { scope in
            scope.setLevel(error.sentryLevel)
            
            // เพิ่ม Context
            context.forEach { key, value in
                scope.setExtra(value: value, key: key)
            }
            
            // เพิ่ม User Info
            if let user = AuthManager.shared.currentUser {
                let sentryUser = Sentry.User(userId: user.id)
                sentryUser.email = user.email
                scope.setUser(sentryUser)
            }
            
            // เพิ่ม Breadcrumbs
            let breadcrumb = Breadcrumb()
            breadcrumb.category = "error"
            breadcrumb.message = error.localizedDescription
            breadcrumb.level = error.sentryLevel
            scope.addBreadcrumb(breadcrumb)
        }
    }
    
    func addBreadcrumb(message: String, category: String, data: [String: Any]? = nil) {
        let breadcrumb = Breadcrumb()
        breadcrumb.category = category
        breadcrumb.message = message
        breadcrumb.data = data
        SentrySDK.addBreadcrumb(breadcrumb)
    }
    
    func setUserContext(userId: String, email: String?) {
        let user = Sentry.User(userId: userId)
        user.email = email
        SentrySDK.setUser(user)
    }
}

// Usage
class PaymentViewModel: ObservableObject {
    @Published var isLoading = false
    @Published var errorMessage: String?
    
    func processPayment(amount: Double, method: PaymentMethod) async {
        isLoading = true
        defer { isLoading = false }
        
        ErrorReporter.shared.addBreadcrumb(
            message: "Starting payment",
            category: "payment",
            data: ["amount": amount, "method": method.rawValue]
        )
        
        do {
            let result = try await PaymentService.shared.charge(amount: amount, method: method)
            ErrorReporter.shared.addBreadcrumb(
                message: "Payment successful",
                category: "payment",
                data: ["transaction_id": result.transactionId]
            )
        } catch let error as AppError {
            ErrorReporter.shared.report(error, context: [
                "amount": amount,
                "payment_method": method.rawValue
            ])
            errorMessage = error.localizedDescription
        }
    }
}
```

---

## 13. Analytics Setup

### 13.1 Analytics Service Architecture

```swift
// AnalyticsService.swift
import Foundation
import FirebaseAnalytics

// Analytics Events
enum AnalyticsEvent {
    // Screen Events
    case screenViewed(name: String, parameters: [String: Any]?)
    
    // Auth Events
    case loginStarted(method: String)
    case loginSuccess(method: String, userId: String)
    case loginFailed(method: String, error: String)
    case logoutSuccess
    
    // Product Events
    case productViewed(id: String, name: String, price: Double, category: String)
    case productAddedToCart(id: String, name: String, price: Double, quantity: Int)
    case productRemovedFromCart(id: String)
    case productSearched(query: String, resultsCount: Int)
    
    // Cart Events
    case cartViewed(itemCount: Int, totalValue: Double)
    case checkoutStarted(itemCount: Int, totalValue: Double)
    case checkoutStepCompleted(step: Int, stepName: String)
    
    // Purchase Events
    case purchaseCompleted(orderId: String, revenue: Double, items: [[String: Any]])
    case purchaseFailed(reason: String)
    
    // Deep Link Events
    case deepLinkOpened(_ link: DeepLink)
    
    var name: String {
        switch self {
        case .screenViewed: return "screen_view"
        case .loginStarted: return "login_started"
        case .loginSuccess: return "login"
        case .loginFailed: return "login_failed"
        case .logoutSuccess: return "logout"
        case .productViewed: return "view_item"
        case .productAddedToCart: return "add_to_cart"
        case .productRemovedFromCart: return "remove_from_cart"
        case .productSearched: return "search"
        case .cartViewed: return "view_cart"
        case .checkoutStarted: return "begin_checkout"
        case .checkoutStepCompleted: return "checkout_progress"
        case .purchaseCompleted: return "purchase"
        case .purchaseFailed: return "purchase_failed"
        case .deepLinkOpened: return "deep_link_opened"
        }
    }
    
    var parameters: [String: Any] {
        switch self {
        case .screenViewed(let name, let params):
            var p: [String: Any] = ["screen_name": name]
            params?.forEach { p[$0.key] = $0.value }
            return p
        case .loginStarted(let method):
            return ["method": method]
        case .loginSuccess(let method, let userId):
            return ["method": method, "user_id": userId]
        case .productViewed(let id, let name, let price, let category):
            return [
                "item_id": id,
                "item_name": name,
                "price": price,
                "item_category": category
            ]
        case .productAddedToCart(let id, let name, let price, let qty):
            return [
                "item_id": id,
                "item_name": name,
                "price": price,
                "quantity": qty,
                "value": price * Double(qty)
            ]
        case .purchaseCompleted(let orderId, let revenue, let items):
            return [
                "transaction_id": orderId,
                "value": revenue,
                "currency": "THB",
                "items": items
            ]
        default:
            return [:]
        }
    }
}

// Analytics Service Protocol
protocol AnalyticsServiceProtocol {
    func track(_ event: AnalyticsEvent)
    func setUserProperty(_ value: String?, forName name: String)
    func identify(userId: String, traits: [String: Any])
}

// Firebase Analytics Implementation
class AnalyticsService: AnalyticsServiceProtocol {
    static let shared = AnalyticsService()
    
    private var isEnabled: Bool {
        AppConfiguration.shared.enableAnalytics
    }
    
    func track(_ event: AnalyticsEvent) {
        guard isEnabled else {
            Logger.debug("[Analytics] \(event.name): \(event.parameters)")
            return
        }
        
        Analytics.logEvent(event.name, parameters: event.parameters)
    }
    
    func setUserProperty(_ value: String?, forName name: String) {
        guard isEnabled else { return }
        Analytics.setUserProperty(value, forName: name)
    }
    
    func identify(userId: String, traits: [String: Any]) {
        guard isEnabled else { return }
        Analytics.setUserID(userId)
        traits.forEach { key, value in
            Analytics.setUserProperty("\(value)", forName: key)
        }
    }
}
```

---

## 14. A/B Testing Setup (Firebase Remote Config)

### 14.1 Remote Config Manager

```swift
// RemoteConfigManager.swift
import FirebaseRemoteConfig
import FirebaseABTesting

// Feature Flags / A/B Test Keys
enum RemoteConfigKey: String, CaseIterable {
    // UI Experiments
    case newCheckoutFlow = "new_checkout_flow_enabled"
    case productGridColumns = "product_grid_columns"
    case showARButton = "show_ar_button"
    
    // Business Logic
    case freeShippingThreshold = "free_shipping_threshold"
    case maxCartItems = "max_cart_items"
    case recommendationAlgorithm = "recommendation_algorithm"
    
    // Feature Flags
    case enableLiveChat = "enable_live_chat"
    case enablePriceAlerts = "enable_price_alerts"
    case enableGroupBuy = "enable_group_buy"
    
    var defaultValue: Any {
        switch self {
        case .newCheckoutFlow: return false
        case .productGridColumns: return 2
        case .showARButton: return false
        case .freeShippingThreshold: return 500.0
        case .maxCartItems: return 99
        case .recommendationAlgorithm: return "collaborative"
        case .enableLiveChat: return false
        case .enablePriceAlerts: return true
        case .enableGroupBuy: return false
        }
    }
}

class RemoteConfigManager: ObservableObject {
    static let shared = RemoteConfigManager()
    
    private let remoteConfig: RemoteConfig
    @Published private(set) var isLoaded = false
    
    private init() {
        self.remoteConfig = RemoteConfig.remoteConfig()
        
        let settings = RemoteConfigSettings()
        settings.minimumFetchInterval = AppEnvironment.current == .production ? 3600 : 0
        remoteConfig.configSettings = settings
        
        // ตั้งค่า Default Values
        var defaults: [String: NSObject] = [:]
        RemoteConfigKey.allCases.forEach { key in
            defaults[key.rawValue] = key.defaultValue as? NSObject
        }
        remoteConfig.setDefaults(defaults)
    }
    
    // Fetch and Activate
    func fetchAndActivate() async {
        do {
            let status = try await remoteConfig.fetchAndActivate()
            Logger.info("Remote Config: \(status == .successFetchedFromRemote ? "Updated" : "Using cache")")
            isLoaded = true
        } catch {
            Logger.error("Remote Config fetch failed: \(error)")
            isLoaded = true  // ใช้ defaults ได้
        }
    }
    
    // Type-safe getters
    func bool(for key: RemoteConfigKey) -> Bool {
        remoteConfig[key.rawValue].boolValue
    }
    
    func int(for key: RemoteConfigKey) -> Int {
        Int(remoteConfig[key.rawValue].numberValue)
    }
    
    func double(for key: RemoteConfigKey) -> Double {
        remoteConfig[key.rawValue].numberValue.doubleValue
    }
    
    func string(for key: RemoteConfigKey) -> String {
        remoteConfig[key.rawValue].stringValue ?? ""
    }
    
    // Experiment tracking
    func isInExperiment(_ key: RemoteConfigKey) -> Bool {
        remoteConfig[key.rawValue].source == .remote
    }
}

// Usage in Views
struct CheckoutView: View {
    @StateObject private var config = RemoteConfigManager.shared
    
    var body: some View {
        Group {
            if config.bool(for: .newCheckoutFlow) {
                NewCheckoutFlowView()
                    .onAppear {
                        AnalyticsService.shared.track(.screenViewed(
                            name: "checkout_new",
                            parameters: ["variant": "new_flow"]
                        ))
                    }
            } else {
                LegacyCheckoutView()
                    .onAppear {
                        AnalyticsService.shared.track(.screenViewed(
                            name: "checkout_legacy",
                            parameters: ["variant": "control"]
                        ))
                    }
            }
        }
    }
}
```

---

## 15. App Rating Prompt (SKStoreReviewController)

### 15.1 Smart Rating Request Strategy

```swift
// RatingManager.swift
import StoreKit

class RatingManager {
    static let shared = RatingManager()
    
    // Trigger Conditions
    private struct Conditions {
        var completedOrders: Int
        var appLaunchCount: Int
        var daysSinceFirstLaunch: Int
        var lastRatingRequestDate: Date?
        var hasRated: Bool
        var appVersion: String
        
        init() {
            let defaults = UserDefaults.standard
            self.completedOrders = defaults.integer(forKey: "rating_completed_orders")
            self.appLaunchCount = defaults.integer(forKey: "rating_launch_count")
            self.daysSinceFirstLaunch = defaults.integer(forKey: "rating_days_since_install")
            self.lastRatingRequestDate = defaults.object(forKey: "rating_last_request") as? Date
            self.hasRated = defaults.bool(forKey: "rating_has_rated")
            self.appVersion = defaults.string(forKey: "rating_app_version") ?? ""
        }
    }
    
    private var conditions: Conditions
    private let minOrdersRequired = 2
    private let minLaunchCountRequired = 5
    private let minDaysRequired = 3
    private let minDaysBetweenRequests = 60
    
    private init() {
        self.conditions = Conditions()
        incrementLaunchCount()
    }
    
    private func incrementLaunchCount() {
        let defaults = UserDefaults.standard
        var count = defaults.integer(forKey: "rating_launch_count")
        count += 1
        defaults.set(count, forKey: "rating_launch_count")
        conditions.appLaunchCount = count
        
        // Update days since install
        if defaults.object(forKey: "rating_install_date") == nil {
            defaults.set(Date(), forKey: "rating_install_date")
        }
        if let installDate = defaults.object(forKey: "rating_install_date") as? Date {
            let days = Calendar.current.dateComponents([.day], from: installDate, to: Date()).day ?? 0
            defaults.set(days, forKey: "rating_days_since_install")
            conditions.daysSinceFirstLaunch = days
        }
    }
    
    func orderCompleted() {
        let defaults = UserDefaults.standard
        var count = defaults.integer(forKey: "rating_completed_orders")
        count += 1
        defaults.set(count, forKey: "rating_completed_orders")
        conditions.completedOrders = count
        
        // ลองแสดง Rating หลังสั่งซื้อสำเร็จ
        requestReviewIfAppropriate()
    }
    
    private func shouldRequestReview() -> Bool {
        // ไม่แสดงถ้า User เคย Rate แล้วใน Version นี้
        let currentVersion = Bundle.main.infoDictionary?["CFBundleShortVersionString"] as? String ?? ""
        if conditions.hasRated && conditions.appVersion == currentVersion {
            return false
        }
        
        // ตรวจสอบ Minimum Requirements
        guard conditions.completedOrders >= minOrdersRequired else { return false }
        guard conditions.appLaunchCount >= minLaunchCountRequired else { return false }
        guard conditions.daysSinceFirstLaunch >= minDaysRequired else { return false }
        
        // ตรวจสอบว่าไม่ได้ถามเร็วเกินไป
        if let lastRequest = conditions.lastRatingRequestDate {
            let daysSinceLastRequest = Calendar.current.dateComponents(
                [.day], from: lastRequest, to: Date()
            ).day ?? 0
            if daysSinceLastRequest < minDaysBetweenRequests {
                return false
            }
        }
        
        return true
    }
    
    func requestReviewIfAppropriate() {
        guard shouldRequestReview() else { return }
        
        // รอสักครู่ก่อนแสดง เพื่อ UX ที่ดี
        DispatchQueue.main.asyncAfter(deadline: .now() + 2.0) { [weak self] in
            self?.requestReview()
        }
    }
    
    private func requestReview() {
        guard let scene = UIApplication.shared.connectedScenes
            .first(where: { $0.activationState == .foregroundActive }) as? UIWindowScene else {
            return
        }
        
        // บันทึกวันที่ขอ
        let defaults = UserDefaults.standard
        defaults.set(Date(), forKey: "rating_last_request")
        defaults.set(
            Bundle.main.infoDictionary?["CFBundleShortVersionString"] as? String ?? "",
            forKey: "rating_app_version"
        )
        
        // Track analytics
        AnalyticsService.shared.track(.screenViewed(name: "rating_prompt", parameters: nil))
        
        // แสดง Rating Dialog
        SKStoreReviewController.requestReview(in: scene)
    }
}
```

---

## 16. Building a Complete E-Commerce App Architecture

### 16.1 Domain Models

```swift
// Models/Product.swift
import Foundation

struct Product: Identifiable, Codable, Hashable {
    let id: String
    let name: String
    let description: String
    let price: Double
    let compareAtPrice: Double?  // ราคาก่อนลด
    let images: [ProductImage]
    let category: Category
    let tags: [String]
    let variants: [ProductVariant]
    let inventory: Inventory
    let rating: Rating
    let isNew: Bool
    let isFeatured: Bool
    
    var isOnSale: Bool {
        guard let compareAtPrice = compareAtPrice else { return false }
        return compareAtPrice > price
    }
    
    var discountPercentage: Int? {
        guard let compareAtPrice = compareAtPrice, compareAtPrice > price else { return nil }
        return Int(((compareAtPrice - price) / compareAtPrice) * 100)
    }
    
    struct ProductImage: Codable, Hashable {
        let id: String
        let url: URL
        let altText: String?
        let position: Int
    }
    
    struct Category: Codable, Hashable {
        let id: String
        let name: String
        let slug: String
        let imageURL: URL?
    }
    
    struct ProductVariant: Identifiable, Codable, Hashable {
        let id: String
        let name: String  // "สีแดง - Size M"
        let options: [String: String]  // ["color": "red", "size": "M"]
        let price: Double
        let sku: String
        let inventoryQuantity: Int
        let imageURL: URL?
    }
    
    struct Inventory: Codable, Hashable {
        let quantity: Int
        let isAvailable: Bool
        let trackQuantity: Bool
    }
    
    struct Rating: Codable, Hashable {
        let average: Double  // 0-5
        let count: Int
        
        var stars: Int { Int(average.rounded()) }
    }
}
```

### 16.2 Product Catalog

```swift
// Features/Catalog/ProductListViewModel.swift
import Foundation
import Combine

@MainActor
class ProductListViewModel: ObservableObject {
    @Published var products: [Product] = []
    @Published var isLoading = false
    @Published var hasMore = true
    @Published var error: String?
    @Published var searchQuery = ""
    @Published var selectedCategory: Product.Category?
    @Published var sortOption: SortOption = .relevance
    @Published var priceRange: ClosedRange<Double> = 0...10000
    
    enum SortOption: String, CaseIterable {
        case relevance = "ความเกี่ยวข้อง"
        case priceLowToHigh = "ราคา: ต่ำ-สูง"
        case priceHighToLow = "ราคา: สูง-ต่ำ"
        case newest = "ใหม่ล่าสุด"
        case bestseller = "ขายดีที่สุด"
        case rating = "คะแนนสูงสุด"
    }
    
    private let productRepository: ProductRepositoryProtocol
    private let analyticsService: AnalyticsServiceProtocol
    private var currentPage = 0
    private var searchTask: Task<Void, Never>?
    private var cancellables = Set<AnyCancellable>()
    
    init(productRepository: ProductRepositoryProtocol,
         analyticsService: AnalyticsServiceProtocol) {
        self.productRepository = productRepository
        self.analyticsService = analyticsService
        setupSearchDebounce()
    }
    
    private func setupSearchDebounce() {
        $searchQuery
            .debounce(for: .milliseconds(500), scheduler: DispatchQueue.main)
            .removeDuplicates()
            .sink { [weak self] query in
                guard let self = self else { return }
                self.resetAndFetch()
                if !query.isEmpty {
                    self.analyticsService.track(.productSearched(
                        query: query,
                        resultsCount: self.products.count
                    ))
                }
            }
            .store(in: &cancellables)
    }
    
    func loadInitial() {
        resetAndFetch()
    }
    
    func loadMore() {
        guard !isLoading, hasMore else { return }
        fetch(page: currentPage + 1)
    }
    
    private func resetAndFetch() {
        products = []
        currentPage = 0
        hasMore = true
        fetch(page: 1)
    }
    
    private func fetch(page: Int) {
        isLoading = true
        error = nil
        
        Task {
            do {
                let result = try await productRepository.fetchProducts(
                    query: searchQuery.isEmpty ? nil : searchQuery,
                    category: selectedCategory?.id,
                    sort: sortOption,
                    priceMin: priceRange.lowerBound,
                    priceMax: priceRange.upperBound,
                    page: page
                )
                
                if page == 1 {
                    products = result.items
                } else {
                    products.append(contentsOf: result.items)
                }
                
                currentPage = page
                hasMore = result.hasMore
                isLoading = false
            } catch {
                self.error = error.localizedDescription
                isLoading = false
                ErrorReporter.shared.report(
                    AppError.networkError(underlying: error, endpoint: "products"),
                    context: ["page": page, "query": searchQuery]
                )
            }
        }
    }
}

// ProductListView.swift
struct ProductListView: View {
    @StateObject private var viewModel: ProductListViewModel
    @State private var showFilters = false
    let columns = [GridItem(.flexible()), GridItem(.flexible())]
    
    init(viewModel: ProductListViewModel) {
        _viewModel = StateObject(wrappedValue: viewModel)
    }
    
    var body: some View {
        NavigationStack {
            VStack(spacing: 0) {
                SearchBar(text: $viewModel.searchQuery)
                    .padding()
                
                CategoryFilterBar(
                    selected: $viewModel.selectedCategory
                )
                
                ScrollView {
                    LazyVGrid(columns: columns, spacing: 16) {
                        ForEach(viewModel.products) { product in
                            NavigationLink(destination: ProductDetailView(product: product)) {
                                ProductCard(product: product)
                            }
                            .onAppear {
                                if product.id == viewModel.products.last?.id {
                                    viewModel.loadMore()
                                }
                            }
                        }
                        
                        if viewModel.isLoading {
                            ForEach(0..<4, id: \.self) { _ in
                                ProductCardSkeleton()
                            }
                        }
                    }
                    .padding()
                }
            }
            .navigationTitle("สินค้าทั้งหมด")
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button(action: { showFilters = true }) {
                        Image(systemName: "slider.horizontal.3")
                    }
                }
            }
            .sheet(isPresented: $showFilters) {
                FilterView(viewModel: viewModel)
            }
        }
        .onAppear {
            viewModel.loadInitial()
        }
    }
}
```

### 16.3 Shopping Cart

```swift
// Features/Cart/CartManager.swift
import Foundation
import Combine

struct CartItem: Identifiable, Codable {
    let id: String
    let product: Product
    let variant: Product.ProductVariant?
    var quantity: Int
    let addedAt: Date
    
    var subtotal: Double {
        (variant?.price ?? product.price) * Double(quantity)
    }
}

@MainActor
class CartManager: ObservableObject {
    static let shared = CartManager()
    
    @Published private(set) var items: [CartItem] = []
    @Published private(set) var isLoading = false
    
    var itemCount: Int { items.reduce(0) { $0 + $1.quantity } }
    var subtotal: Double { items.reduce(0) { $0 + $1.subtotal } }
    var shippingFee: Double {
        let threshold = RemoteConfigManager.shared.double(for: .freeShippingThreshold)
        return subtotal >= threshold ? 0 : 50
    }
    var total: Double { subtotal + shippingFee }
    
    private let storage: CartStorageProtocol
    private let syncService: CartSyncServiceProtocol
    
    private init() {
        self.storage = CartStorage()
        self.syncService = CartSyncService()
        loadFromStorage()
    }
    
    func addItem(_ product: Product, variant: Product.ProductVariant? = nil, quantity: Int = 1) {
        let variantId = variant?.id
        
        if let index = items.firstIndex(where: {
            $0.product.id == product.id && $0.variant?.id == variantId
        }) {
            items[index].quantity += quantity
        } else {
            let item = CartItem(
                id: UUID().uuidString,
                product: product,
                variant: variant,
                quantity: quantity,
                addedAt: Date()
            )
            items.append(item)
        }
        
        saveToStorage()
        
        AnalyticsService.shared.track(.productAddedToCart(
            id: product.id,
            name: product.name,
            price: variant?.price ?? product.price,
            quantity: quantity
        ))
        
        // Haptic feedback
        HapticManager.shared.impact(.medium)
    }
    
    func removeItem(_ item: CartItem) {
        items.removeAll { $0.id == item.id }
        saveToStorage()
        AnalyticsService.shared.track(.productRemovedFromCart(id: item.product.id))
    }
    
    func updateQuantity(for item: CartItem, quantity: Int) {
        guard let index = items.firstIndex(where: { $0.id == item.id }) else { return }
        if quantity <= 0 {
            removeItem(item)
        } else {
            items[index].quantity = quantity
            saveToStorage()
        }
    }
    
    func clear() {
        items = []
        saveToStorage()
    }
    
    private func loadFromStorage() {
        items = (try? storage.loadCart()) ?? []
    }
    
    private func saveToStorage() {
        try? storage.saveCart(items)
        
        // Sync กับ Server ถ้า Login แล้ว
        if AuthManager.shared.isLoggedIn {
            Task {
                try? await syncService.syncCart(items)
            }
        }
    }
}

// CartView.swift
struct CartView: View {
    @ObservedObject var cart = CartManager.shared
    @State private var showCheckout = false
    
    var body: some View {
        NavigationStack {
            if cart.items.isEmpty {
                EmptyCartView()
            } else {
                List {
                    ForEach(cart.items) { item in
                        CartItemRow(item: item)
                    }
                    .onDelete { indexSet in
                        indexSet.forEach { cart.removeItem(cart.items[$0]) }
                    }
                    
                    Section {
                        OrderSummaryView(
                            subtotal: cart.subtotal,
                            shipping: cart.shippingFee,
                            total: cart.total
                        )
                    }
                }
                .safeAreaInset(edge: .bottom) {
                    Button(action: { showCheckout = true }) {
                        HStack {
                            Text("ดำเนินการชำระเงิน")
                                .font(.headline)
                            Spacer()
                            Text("฿\(cart.total, specifier: "%.2f")")
                                .font(.headline)
                        }
                        .padding()
                        .frame(maxWidth: .infinity)
                        .background(Color.accentColor)
                        .foregroundColor(.white)
                        .cornerRadius(12)
                        .padding()
                    }
                }
            }
        }
        .navigationTitle("ตะกร้าสินค้า (\(cart.itemCount))")
        .sheet(isPresented: $showCheckout) {
            CheckoutView()
        }
    }
}
```

### 16.4 User Authentication

```swift
// Features/Auth/AuthManager.swift
import Foundation
import AuthenticationServices

@MainActor
class AuthManager: ObservableObject {
    static let shared = AuthManager()
    
    @Published private(set) var currentUser: User?
    @Published private(set) var authState: AuthState = .loading
    
    enum AuthState {
        case loading
        case authenticated(User)
        case unauthenticated
    }
    
    var isLoggedIn: Bool {
        if case .authenticated = authState { return true }
        return false
    }
    
    private let authService: AuthServiceProtocol
    private let tokenStorage: TokenStorageProtocol
    
    private init() {
        self.authService = AuthService()
        self.tokenStorage = KeychainTokenStorage()
        Task { await checkAuthState() }
    }
    
    // Check existing auth state
    func checkAuthState() async {
        guard let token = tokenStorage.loadAccessToken() else {
            authState = .unauthenticated
            return
        }
        
        do {
            let user = try await authService.validateToken(token)
            currentUser = user
            authState = .authenticated(user)
            ErrorReporter.shared.setUserContext(userId: user.id, email: user.email)
        } catch {
            tokenStorage.clearTokens()
            authState = .unauthenticated
        }
    }
    
    // Email/Password Login
    func login(email: String, password: String) async throws {
        AnalyticsService.shared.track(.loginStarted(method: "email"))
        
        do {
            let response = try await authService.login(email: email, password: password)
            tokenStorage.saveTokens(
                access: response.accessToken,
                refresh: response.refreshToken
            )
            currentUser = response.user
            authState = .authenticated(response.user)
            
            AnalyticsService.shared.track(.loginSuccess(
                method: "email",
                userId: response.user.id
            ))
        } catch {
            AnalyticsService.shared.track(.loginFailed(
                method: "email",
                error: error.localizedDescription
            ))
            throw error
        }
    }
    
    // Sign in with Apple
    func signInWithApple(_ authorization: ASAuthorization) async throws {
        guard let credential = authorization.credential as? ASAuthorizationAppleIDCredential,
              let identityToken = credential.identityToken,
              let tokenString = String(data: identityToken, encoding: .utf8) else {
            throw AppError.authenticationFailed(reason: "Invalid Apple credentials")
        }
        
        AnalyticsService.shared.track(.loginStarted(method: "apple"))
        
        let response = try await authService.loginWithApple(
            identityToken: tokenString,
            fullName: credential.fullName?.formatted(),
            email: credential.email
        )
        
        tokenStorage.saveTokens(access: response.accessToken, refresh: response.refreshToken)
        currentUser = response.user
        authState = .authenticated(response.user)
        
        AnalyticsService.shared.track(.loginSuccess(method: "apple", userId: response.user.id))
    }
    
    // Refresh Token
    func refreshTokenIfNeeded() async throws {
        guard let refreshToken = tokenStorage.loadRefreshToken() else { return }
        
        let response = try await authService.refreshToken(refreshToken)
        tokenStorage.saveTokens(
            access: response.accessToken,
            refresh: response.refreshToken
        )
    }
    
    // Logout
    func logout() async {
        do {
            try await authService.logout()
        } catch {
            // Log but continue
            Logger.warning("Logout API call failed: \(error)")
        }
        
        tokenStorage.clearTokens()
        currentUser = nil
        authState = .unauthenticated
        CartManager.shared.clear()
        
        AnalyticsService.shared.track(.logoutSuccess)
        SentrySDK.setUser(nil)
    }
}
```

### 16.5 Checkout Flow

```swift
// Features/Checkout/CheckoutViewModel.swift
import Foundation

@MainActor
class CheckoutViewModel: ObservableObject {
    enum CheckoutStep: Int, CaseIterable {
        case address = 0
        case shipping = 1
        case payment = 2
        case review = 3
        case confirmation = 4
        
        var title: String {
            switch self {
            case .address: return "ที่อยู่จัดส่ง"
            case .shipping: return "วิธีจัดส่ง"
            case .payment: return "ชำระเงิน"
            case .review: return "ตรวจสอบ"
            case .confirmation: return "ยืนยันการสั่งซื้อ"
            }
        }
    }
    
    @Published var currentStep: CheckoutStep = .address
    @Published var selectedAddress: DeliveryAddress?
    @Published var selectedShipping: ShippingOption?
    @Published var selectedPayment: PaymentMethod?
    @Published var promoCode: String = ""
    @Published var promoDiscount: Double = 0
    @Published var isLoading = false
    @Published var error: String?
    @Published var completedOrder: Order?
    
    private let orderService: OrderServiceProtocol
    private let paymentService: PaymentServiceProtocol
    private let cart = CartManager.shared
    
    var canProceed: Bool {
        switch currentStep {
        case .address: return selectedAddress != nil
        case .shipping: return selectedShipping != nil
        case .payment: return selectedPayment != nil
        case .review: return true
        case .confirmation: return false
        }
    }
    
    var orderTotal: Double {
        cart.total - promoDiscount
    }
    
    init(orderService: OrderServiceProtocol, paymentService: PaymentServiceProtocol) {
        self.orderService = orderService
        self.paymentService = paymentService
    }
    
    func nextStep() {
        guard let next = CheckoutStep(rawValue: currentStep.rawValue + 1) else { return }
        
        AnalyticsService.shared.track(.checkoutStepCompleted(
            step: currentStep.rawValue + 1,
            stepName: currentStep.title
        ))
        
        withAnimation {
            currentStep = next
        }
    }
    
    func applyPromoCode() async {
        isLoading = true
        defer { isLoading = false }
        
        do {
            let discount = try await orderService.validatePromoCode(promoCode)
            promoDiscount = discount
        } catch {
            self.error = "โค้ดส่วนลดไม่ถูกต้อง"
        }
    }
    
    func placeOrder() async {
        guard let address = selectedAddress,
              let shipping = selectedShipping,
              let payment = selectedPayment else { return }
        
        isLoading = true
        defer { isLoading = false }
        
        do {
            // 1. Create Order
            let orderRequest = CreateOrderRequest(
                items: cart.items.map { OrderItem(cartItem: $0) },
                deliveryAddress: address,
                shippingOption: shipping,
                paymentMethod: payment,
                promoCode: promoCode.isEmpty ? nil : promoCode
            )
            
            let order = try await orderService.createOrder(orderRequest)
            
            // 2. Process Payment
            let paymentResult = try await paymentService.processPayment(
                orderId: order.id,
                amount: order.total,
                method: payment
            )
            
            // 3. Confirm Order
            try await orderService.confirmOrder(
                id: order.id,
                paymentId: paymentResult.transactionId
            )
            
            // 4. Success
            completedOrder = order
            cart.clear()
            currentStep = .confirmation
            
            AnalyticsService.shared.track(.purchaseCompleted(
                orderId: order.id,
                revenue: order.total,
                items: cart.items.map { item in
                    ["item_id": item.product.id,
                     "item_name": item.product.name,
                     "price": item.product.price,
                     "quantity": item.quantity]
                }
            ))
            
            // Schedule delivery notification
            PushNotificationManager.shared.scheduleOrderConfirmation(order)
            
            RatingManager.shared.orderCompleted()
            
        } catch let error as AppError {
            ErrorReporter.shared.report(error)
            self.error = error.localizedDescription
            AnalyticsService.shared.track(.purchaseFailed(reason: error.localizedDescription))
        }
    }
}
```

### 16.6 Push Notifications

```swift
// Features/Notifications/PushNotificationManager.swift
import UserNotifications
import FirebaseMessaging

class PushNotificationManager: NSObject, ObservableObject {
    static let shared = PushNotificationManager()
    
    @Published var permissionStatus: UNAuthorizationStatus = .notDetermined
    
    enum NotificationCategory: String {
        case orderUpdate = "ORDER_UPDATE"
        case promotion = "PROMOTION"
        case priceAlert = "PRICE_ALERT"
        case generalMessage = "GENERAL"
    }
    
    func requestPermission() async -> Bool {
        do {
            let granted = try await UNUserNotificationCenter.current().requestAuthorization(
                options: [.alert, .badge, .sound]
            )
            await updatePermissionStatus()
            
            AnalyticsService.shared.setUserProperty(
                granted ? "enabled" : "disabled",
                forName: "push_notifications"
            )
            
            return granted
        } catch {
            return false
        }
    }
    
    func registerDeviceToken(_ token: String) {
        Messaging.messaging().apnsToken = Data(token.hexToBytes())
        
        // Register with backend
        Task {
            if AuthManager.shared.isLoggedIn {
                try? await NotificationService.shared.registerToken(token)
            }
        }
    }
    
    // Schedule local notification สำหรับ Order Confirmation
    func scheduleOrderConfirmation(_ order: Order) {
        let content = UNMutableNotificationContent()
        content.title = "ยืนยันการสั่งซื้อ!"
        content.body = "คำสั่งซื้อ #\(order.id) ได้รับการยืนยันแล้ว"
        content.sound = .default
        content.badge = 1
        content.userInfo = [
            "order_id": order.id,
            "deep_link": "shopswift://order/\(order.id)"
        ]
        content.categoryIdentifier = NotificationCategory.orderUpdate.rawValue
        
        let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 1, repeats: false)
        let request = UNNotificationRequest(
            identifier: "order_\(order.id)",
            content: content,
            trigger: trigger
        )
        
        UNUserNotificationCenter.current().add(request)
    }
    
    // Handle notification response
    func handleNotificationResponse(_ response: UNNotificationResponse) {
        let userInfo = response.notification.request.content.userInfo
        
        if let deepLink = userInfo["deep_link"] as? String,
           let url = URL(string: deepLink) {
            DeepLinkRouter.shared.handle(url)
        }
        
        // Track engagement
        AnalyticsService.shared.track(.screenViewed(
            name: "notification_opened",
            parameters: [
                "category": response.notification.request.content.categoryIdentifier,
                "action": response.actionIdentifier
            ]
        ))
    }
    
    private func updatePermissionStatus() async {
        let settings = await UNUserNotificationCenter.current().notificationSettings()
        permissionStatus = settings.authorizationStatus
    }
}
```

### 16.7 Offline Support

```swift
// Core/Network/NetworkMonitor.swift
import Network
import Combine

class NetworkMonitor: ObservableObject {
    static let shared = NetworkMonitor()
    
    @Published private(set) var isConnected = true
    @Published private(set) var connectionType: ConnectionType = .wifi
    
    enum ConnectionType {
        case wifi, cellular, wiredEthernet, unknown, none
    }
    
    private let monitor = NWPathMonitor()
    private let queue = DispatchQueue(label: "NetworkMonitor")
    
    private init() {
        monitor.pathUpdateHandler = { [weak self] path in
            DispatchQueue.main.async {
                self?.isConnected = path.status == .satisfied
                self?.connectionType = self?.getConnectionType(path) ?? .none
                
                if path.status == .satisfied {
                    NotificationCenter.default.post(name: .networkConnected, object: nil)
                } else {
                    NotificationCenter.default.post(name: .networkDisconnected, object: nil)
                }
            }
        }
        monitor.start(queue: queue)
    }
    
    private func getConnectionType(_ path: NWPath) -> ConnectionType {
        if path.usesInterfaceType(.wifi) { return .wifi }
        if path.usesInterfaceType(.cellular) { return .cellular }
        if path.usesInterfaceType(.wiredEthernet) { return .wiredEthernet }
        return .unknown
    }
}

// OfflineCacheManager.swift
class OfflineCacheManager {
    static let shared = OfflineCacheManager()
    
    private let coreDataStack = CoreDataStack.shared
    
    // Cache Products สำหรับ Offline
    func cacheProducts(_ products: [Product]) async throws {
        let context = coreDataStack.backgroundContext
        
        try await context.perform {
            for product in products {
                let cached = CachedProductEntity.findOrCreate(
                    id: product.id,
                    in: context
                )
                cached.update(from: product)
            }
            try context.save()
        }
    }
    
    // Load Products จาก Cache
    func loadCachedProducts(
        query: String? = nil,
        category: String? = nil
    ) async throws -> [Product] {
        let context = coreDataStack.viewContext
        
        return try await context.perform {
            let request = CachedProductEntity.fetchRequest()
            
            var predicates: [NSPredicate] = []
            if let query = query, !query.isEmpty {
                predicates.append(NSPredicate(format: "name CONTAINS[cd] %@", query))
            }
            if let category = category {
                predicates.append(NSPredicate(format: "categoryId == %@", category))
            }
            
            if !predicates.isEmpty {
                request.predicate = NSCompoundPredicate(andPredicateWithSubpredicates: predicates)
            }
            
            request.sortDescriptors = [NSSortDescriptor(key: "isFeatured", ascending: false)]
            
            let entities = try context.fetch(request)
            return entities.compactMap { $0.toProduct() }
        }
    }
    
    // Offline Banner View
    var offlineBanner: some View {
        Group {
            if !NetworkMonitor.shared.isConnected {
                HStack {
                    Image(systemName: "wifi.slash")
                    Text("ไม่มีการเชื่อมต่ออินเทอร์เน็ต - แสดงข้อมูล Offline")
                        .font(.caption)
                }
                .frame(maxWidth: .infinity)
                .padding(.vertical, 8)
                .background(Color.orange)
                .foregroundColor(.white)
                .transition(.move(edge: .top))
            }
        }
        .animation(.easeInOut, value: NetworkMonitor.shared.isConnected)
    }
}
```

---

## 17. Deployment Pipeline

### 17.1 Fastlane Configuration

```ruby
# Fastfile
default_platform(:ios)

platform :ios do
  
  desc "Run Unit Tests"
  lane :test do
    run_tests(
      scheme: "ShopSwift",
      devices: ["iPhone 15 Pro"],
      code_coverage: true,
      output_directory: "./test_output",
      output_types: "html,junit"
    )
    
    # Upload coverage to Codecov
    codecov(
      project_name: "ShopSwift",
      token: ENV["CODECOV_TOKEN"]
    )
  end
  
  desc "Build and Deploy to TestFlight (Beta)"
  lane :beta do
    ensure_git_branch(branch: "develop")
    
    # Bump build number
    increment_build_number(
      build_number: ENV["BUILD_NUMBER"] || latest_testflight_build_number + 1
    )
    
    # Match Signing (App Store)
    match(
      type: "appstore",
      app_identifier: "com.shopswift.app",
      readonly: true
    )
    
    # Build
    build_app(
      scheme: "ShopSwift-Staging",
      configuration: "Staging",
      export_method: "app-store",
      export_options: {
        provisioningProfiles: {
          "com.shopswift.app" => "match AppStore com.shopswift.app"
        }
      }
    )
    
    # Upload to TestFlight
    upload_to_testflight(
      skip_waiting_for_build_processing: true,
      changelog: changelog_from_git_commits(
        merge_commit_filtering: "exclude_merges"
      )
    )
    
    # Notify Slack
    slack(
      message: "ShopSwift Beta uploaded to TestFlight!",
      channel: "#ios-releases",
      slack_url: ENV["SLACK_WEBHOOK"]
    )
  end
  
  desc "Deploy to App Store"
  lane :release do
    ensure_git_branch(branch: "main")
    
    # Verify all tests pass
    test
    
    # Match Production signing
    match(
      type: "appstore",
      app_identifier: "com.shopswift.app",
      readonly: true
    )
    
    # Build Production
    build_app(
      scheme: "ShopSwift",
      configuration: "Release",
      export_method: "app-store"
    )
    
    # Upload to App Store
    upload_to_app_store(
      force: true,
      skip_metadata: false,
      skip_screenshots: false,
      submit_for_review: false,  # Manual review submission
      precheck_include_in_app_purchases: false
    )
    
    # Tag release
    add_git_tag(tag: "v\(get_version_number)")
    push_git_tags
    
    # Create GitHub Release
    set_github_release(
      repository_name: "company/shopswift-ios",
      api_token: ENV["GITHUB_TOKEN"],
      name: "v\(get_version_number)",
      tag_name: "v\(get_version_number)",
      description: changelog_from_git_commits
    )
  end
end
```

### 17.2 GitHub Actions CI/CD

```yaml
# .github/workflows/ios.yml
name: iOS CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    name: Run Tests
    runs-on: macos-14
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Select Xcode
        run: sudo xcode-select -s /Applications/Xcode_15.2.app
      
      - name: Cache SPM
        uses: actions/cache@v4
        with:
          path: .build
          key: ${{ runner.os }}-spm-${{ hashFiles('**/Package.resolved') }}
      
      - name: Run Tests
        run: |
          xcodebuild test \
            -project ShopSwift.xcodeproj \
            -scheme ShopSwift \
            -destination 'platform=iOS Simulator,name=iPhone 15 Pro' \
            -enableCodeCoverage YES \
            | xcpretty --color
      
      - name: Upload Coverage
        uses: codecov/codecov-action@v3
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
  
  beta:
    name: Deploy to TestFlight
    runs-on: macos-14
    needs: test
    if: github.ref == 'refs/heads/develop'
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.2'
          bundler-cache: true
      
      - name: Import Certificates
        env:
          CERTIFICATES_P12: ${{ secrets.CERTIFICATES_P12 }}
          CERTIFICATES_P12_PASSWORD: ${{ secrets.CERTIFICATES_P12_PASSWORD }}
          KEYCHAIN_PASSWORD: ${{ secrets.KEYCHAIN_PASSWORD }}
        run: |
          echo $CERTIFICATES_P12 | base64 --decode > certificates.p12
          security create-keychain -p "$KEYCHAIN_PASSWORD" build.keychain
          security import certificates.p12 -P "$CERTIFICATES_P12_PASSWORD" -A -k build.keychain
      
      - name: Deploy Beta
        env:
          APP_STORE_CONNECT_API_KEY: ${{ secrets.APP_STORE_CONNECT_API_KEY }}
          BUILD_NUMBER: ${{ github.run_number }}
        run: bundle exec fastlane beta
```

---

## 18. App Monitoring in Production

### 18.1 Performance Monitoring

```swift
// PerformanceMonitor.swift
import MetricKit
import FirebasePerformance

class PerformanceMonitor: NSObject {
    static let shared = PerformanceMonitor()
    
    private var traces: [String: Trace] = [:]
    
    func setupMetricKit() {
        MXMetricManager.shared.add(self)
    }
    
    // Custom Performance Traces
    func startTrace(_ name: String) {
        let trace = Performance.startTrace(name: name)
        traces[name] = trace
    }
    
    func stopTrace(_ name: String, success: Bool = true) {
        guard let trace = traces[name] else { return }
        trace.setValue(success ? 1 : 0, forAttribute: "success")
        trace.stop()
        traces.removeValue(forKey: name)
    }
    
    // Measure Block Execution
    func measure<T>(_ name: String, block: () async throws -> T) async rethrows -> T {
        startTrace(name)
        defer { stopTrace(name) }
        return try await block()
    }
    
    // Network Performance
    func trackNetworkRequest(
        url: String,
        method: String,
        responseCode: Int,
        requestSize: Int64,
        responseSize: Int64,
        duration: TimeInterval
    ) {
        let metric = HTTPMetric(url: URL(string: url)!, httpMethod: HTTPMethod(rawValue: method)!)
        metric?.requestPayloadSize = requestSize
        metric?.responsePayloadSize = responseSize
        metric?.responseCode = responseCode
        metric?.stop()
    }
}

extension PerformanceMonitor: MXMetricManagerSubscriber {
    func didReceive(_ payloads: [MXMetricPayload]) {
        for payload in payloads {
            // App Launch Time
            if let launchMetrics = payload.applicationLaunchMetrics {
                let coldLaunch = launchMetrics.histogrammedTimeToFirstDraw.bucketHistogram
                Logger.info("Cold launch metrics: \(coldLaunch)")
                
                // Report to custom dashboard
                MetricsDashboard.shared.report(.appLaunch, value: coldLaunch.totalBucketCount)
            }
            
            // Memory Usage
            if let memoryMetrics = payload.memoryMetrics {
                Logger.info("Peak memory: \(memoryMetrics.peakMemoryUsage)")
            }
            
            // CPU Time
            if let cpuMetrics = payload.cpuMetrics {
                Logger.info("CPU time: \(cpuMetrics.cumulativeCPUTime)")
            }
        }
    }
    
    func didReceive(_ payloads: [MXDiagnosticPayload]) {
        for payload in payloads {
            // Crash Diagnostics
            if let crashes = payload.crashDiagnostics {
                for crash in crashes {
                    Logger.error("Crash: \(crash.callStackTree)")
                }
            }
            
            // Hang Diagnostics
            if let hangs = payload.hangDiagnostics {
                for hang in hangs {
                    Logger.warning("Hang: \(hang.callStackTree)")
                }
            }
        }
    }
}
```

### 18.2 Custom Dashboard

```swift
// MonitoringDashboard.swift - In-App Debug View (Debug builds only)
#if DEBUG
struct MonitoringDashboard: View {
    @State private var metrics = AppMetrics.shared
    
    var body: some View {
        NavigationStack {
            List {
                Section("Network") {
                    MetricRow(title: "Total Requests", value: "\(metrics.totalRequests)")
                    MetricRow(title: "Failed Requests", value: "\(metrics.failedRequests)")
                    MetricRow(title: "Avg Response Time", value: "\(Int(metrics.avgResponseTime))ms")
                    MetricRow(title: "Cache Hit Rate", value: "\(Int(metrics.cacheHitRate * 100))%")
                }
                
                Section("Memory") {
                    MetricRow(title: "Current Usage", value: "\(metrics.currentMemoryMB) MB")
                    MetricRow(title: "Peak Usage", value: "\(metrics.peakMemoryMB) MB")
                }
                
                Section("Performance") {
                    MetricRow(title: "FPS", value: "\(Int(metrics.currentFPS))")
                    MetricRow(title: "Dropped Frames", value: "\(metrics.droppedFrames)")
                }
                
                Section("App") {
                    MetricRow(title: "Launch Count", value: "\(metrics.launchCount)")
                    MetricRow(title: "Session Duration", value: "\(Int(metrics.sessionDuration))s")
                    MetricRow(title: "Errors Count", value: "\(metrics.errorsCount)")
                }
            }
            .navigationTitle("Performance Dashboard")
            .toolbar {
                Button("Reset") {
                    AppMetrics.shared.reset()
                }
            }
        }
    }
}
#endif
```

---

## 19. Order History

```swift
// Features/Orders/OrderHistoryViewModel.swift
@MainActor
class OrderHistoryViewModel: ObservableObject {
    @Published var orders: [Order] = []
    @Published var isLoading = false
    @Published var selectedStatus: Order.Status?
    
    enum Order.Status: String, CaseIterable {
        case pending = "รอดำเนินการ"
        case confirmed = "ยืนยันแล้ว"
        case processing = "กำลังจัดเตรียม"
        case shipped = "จัดส่งแล้ว"
        case delivered = "ส่งสำเร็จ"
        case cancelled = "ยกเลิกแล้ว"
        
        var color: Color {
            switch self {
            case .pending: return .orange
            case .confirmed: return .blue
            case .processing: return .purple
            case .shipped: return .green
            case .delivered: return .teal
            case .cancelled: return .red
            }
        }
    }
    
    private let orderRepository: OrderRepositoryProtocol
    
    func loadOrders() async {
        isLoading = true
        defer { isLoading = false }
        
        do {
            orders = try await orderRepository.fetchOrders(
                status: selectedStatus?.rawValue
            )
        } catch {
            ErrorReporter.shared.report(
                AppError.networkError(underlying: error, endpoint: "orders")
            )
        }
    }
}
```

---

## 20. Summary

ในบทนี้เราได้เรียนรู้การสร้างแอปพลิเคชันระดับ Production ครบถ้วน:

### สิ่งที่ได้เรียนรู้

1. **Project Planning**: การวางแผนที่ดีตั้งแต่ต้น ช่วยลดปัญหาในภายหลัง
2. **MVP Definition**: ใช้ MoSCoW Method เพื่อจัดลำดับความสำคัญ
3. **Technical Architecture**: Clean Architecture ทำให้โค้ด Maintainable
4. **Environment Configuration**: xcconfig files แยก Dev/Staging/Production
5. **Deep Linking**: Universal Links + Custom URL Scheme ให้ UX ที่ดี
6. **Background Tasks**: BGTaskScheduler สำหรับงานที่ทำใน Background
7. **Error Reporting**: Sentry ช่วย Track และ Debug ปัญหาใน Production
8. **Analytics**: Firebase Analytics วัดผลและทำ A/B Testing ได้
9. **App Rating**: Smart Rating Strategy ไม่รบกวนผู้ใช้
10. **E-Commerce Architecture**: โครงสร้างที่ Scale ได้สำหรับ Shopping App
11. **CI/CD**: Fastlane + GitHub Actions อัตโนมัติทั้ง Build และ Deploy
12. **Monitoring**: MetricKit + Firebase Performance วัด Performance จริง

### Best Practices ที่สำคัญ

- **Never hardcode secrets** - ใช้ xcconfig + CI/CD secrets เสมอ
- **Error handling first** - ออกแบบ Error Cases ก่อน Happy Path
- **Offline first** - แอปต้องทำงานได้แม้ไม่มี Internet
- **Analytics from day 1** - เก็บข้อมูลตั้งแต่เริ่มต้น
- **Automate everything** - CI/CD ลด Human Error

---

*บทต่อไป: Part 78 - Advanced Core Data*
