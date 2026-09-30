# Part 56: Modular Architecture in iOS

## บทนำ

เมื่อ iOS Application เติบโตขึ้น การจัดการโค้ดในโปรเจกต์เดียวกันทั้งหมดเริ่มกลายเป็นปัญหา ทีมขยายใหญ่ขึ้น Build Time นานขึ้น การเพิ่ม Feature ใหม่ส่งผลต่อส่วนอื่น และการทดสอบทำได้ยากขึ้น

Modular Architecture คือแนวทางการแบ่ง Application ออกเป็น Modules อิสระที่มีขอบเขตชัดเจน แต่ละ Module รับผิดชอบหน้าที่เฉพาะของตัวเอง ทำงานร่วมกันผ่าน Interface ที่กำหนดไว้

---

## 1. Why Modular Architecture?

### ปัญหาของ Monolithic Architecture

```
MyApp/
├── Views/
│   ├── LoginView.swift
│   ├── HomeView.swift
│   ├── ProductListView.swift
│   ├── CartView.swift
│   └── ProfileView.swift
├── ViewModels/
│   ├── LoginViewModel.swift
│   ├── HomeViewModel.swift
│   └── ... (100+ files)
├── Models/
│   └── ... (50+ files)
├── Services/
│   └── ... (30+ files)
└── Utilities/
    └── ... (20+ files)
```

ปัญหา:
- **Build Time**: เปลี่ยนไฟล์เดียว ต้อง recompile ทั้งหมด
- **Coupling**: Code ทุกส่วนรู้จักกัน เปลี่ยนอะไรกระทบทุกที่
- **Team Conflicts**: หลายคนแก้ไฟล์เดียวกัน เกิด Merge Conflicts บ่อย
- **Testing**: ยากที่จะ Test แต่ละส่วนแยกกัน
- **Reuse**: ไม่สามารถนำ Code ไปใช้ใน App อื่นได้

### ประโยชน์ของ Modular Architecture

```
MyApp/
├── App/                    # Main App target
├── Features/
│   ├── Login/              # Feature Module
│   ├── Home/
│   ├── Products/
│   ├── Cart/
│   └── Profile/
├── Core/
│   ├── Networking/         # Core Module
│   ├── Storage/
│   └── Analytics/
└── UI/
    ├── DesignSystem/       # Shared UI Module
    └── Components/
```

ประโยชน์:
1. **Faster Build Times**: แต่ละ Module compile แยกกัน
2. **Clear Boundaries**: รู้ชัดเจนว่า Code นั้นอยู่ที่ไหน
3. **Team Independence**: ทีมต่างๆ ทำงานบน Module ของตัวเองโดยไม่กวนกัน
4. **Reusability**: นำ Module ไปใช้ซ้ำในโปรเจกต์อื่น
5. **Testability**: Test แต่ละ Module แยกกันได้ง่าย
6. **Parallel Development**: หลายทีมทำงานพร้อมกัน

### Metrics ที่ดีขึ้น

| ตัวชี้วัด | Before Modular | After Modular |
|---------|--------------|--------------|
| Clean Build Time | 8 นาที | 2 นาที |
| Incremental Build | 3 นาที | 15 วินาที |
| Unit Test Coverage | 30% | 75% |
| Merge Conflicts/week | 15 ครั้ง | 3 ครั้ง |

---

## 2. Framework Targets in Xcode

### สร้าง Framework Target

```
1. File → New → Target...
2. เลือก "Framework"
3. ตั้งชื่อ เช่น "NetworkingModule"
4. กำหนด Embed settings
```

### ตั้งค่า Framework

```swift
// NetworkingModule.swift - Public Interface
import Foundation

// @_exported ทำให้ผู้ใช้ framework ไม่ต้อง import dependency เอง
@_exported import Alamofire

public protocol NetworkClientProtocol {
    func request<T: Decodable>(_ endpoint: Endpoint) async throws -> T
}

public class NetworkClient: NetworkClientProtocol {
    public static let shared = NetworkClient()
    
    private let session: Session
    
    public init(configuration: NetworkConfiguration = .default) {
        self.session = Session(configuration: configuration.urlSessionConfiguration)
    }
    
    public func request<T: Decodable>(_ endpoint: Endpoint) async throws -> T {
        let response = await session.request(endpoint.url,
                                              method: endpoint.method,
                                              parameters: endpoint.parameters)
            .serializingDecodable(T.self)
            .response
        
        switch response.result {
        case .success(let value):
            return value
        case .failure(let error):
            throw NetworkError.underlying(error)
        }
    }
}
```

### การใช้ Framework ในโปรเจกต์

```swift
// ใน Main App
import NetworkingModule

class ProductService {
    private let client: NetworkClientProtocol
    
    init(client: NetworkClientProtocol = NetworkClient.shared) {
        self.client = client
    }
    
    func fetchProducts() async throws -> [Product] {
        try await client.request(ProductEndpoint.list)
    }
}
```

### Access Control ใน Framework

```swift
// public - เข้าถึงได้จากนอก Module
public struct Product {
    public let id: String
    public let name: String
    // ...
}

// internal (default) - เข้าถึงได้เฉพาะใน Module
struct ProductCache {
    var products: [Product] = []
}

// private/fileprivate - เข้าถึงได้เฉพาะใน file/type
private var _lastFetchDate: Date?
```

---

## 3. Swift Packages as Modules

### Swift Package Manager (SPM) สำหรับ Modules

SPM เป็นวิธีที่แนะนำในการสร้าง Modules ใน Swift เพราะ:
- ไม่ต้องพึ่ง Xcode
- Version Control ง่าย
- Cross-platform ได้
- Open Source friendly

### สร้าง Swift Package

```bash
# สร้าง Package ใหม่
mkdir NetworkingModule
cd NetworkingModule
swift package init --type library

# โครงสร้าง
NetworkingModule/
├── Package.swift
├── Sources/
│   └── NetworkingModule/
│       ├── NetworkClient.swift
│       ├── Endpoint.swift
│       └── NetworkError.swift
└── Tests/
    └── NetworkingModuleTests/
        └── NetworkClientTests.swift
```

### Package.swift สำหรับ Module

```swift
// swift-tools-version: 5.9
import PackageDescription

let package = Package(
    name: "NetworkingModule",
    platforms: [
        .iOS(.v16),
        .macOS(.v13)
    ],
    products: [
        // Library ที่ผู้อื่นสามารถ import ได้
        .library(
            name: "NetworkingModule",
            targets: ["NetworkingModule"]
        ),
    ],
    dependencies: [
        // External dependencies
    ],
    targets: [
        .target(
            name: "NetworkingModule",
            dependencies: [],
            swiftSettings: [
                .enableExperimentalFeature("StrictConcurrency")
            ]
        ),
        .testTarget(
            name: "NetworkingModuleTests",
            dependencies: ["NetworkingModule"]
        ),
    ]
)
```

### Local Package ใน Xcode Project

```swift
// Package.swift ของ Main App
let package = Package(
    name: "MyApp",
    dependencies: [
        // Local packages
        .package(path: "../NetworkingModule"),
        .package(path: "../DesignSystem"),
        .package(path: "../Analytics"),
        
        // Remote packages
        .package(url: "https://github.com/some/package", from: "1.0.0"),
    ],
    targets: [
        .target(
            name: "MyApp",
            dependencies: [
                "NetworkingModule",
                "DesignSystem",
                "Analytics",
            ]
        )
    ]
)
```

---

## 4. Dependency Graph

### วางแผน Dependency Graph

กฎสำคัญของ Dependency Graph:
1. **No Circular Dependencies**: A ขึ้นอยู่กับ B, B ขึ้นอยู่กับ A = ไม่ได้!
2. **Direction**: Layers ที่สูงกว่าขึ้นอยู่กับ Layers ที่ต่ำกว่า
3. **Feature → Core → Foundation**: ทิศทางการ depend ชัดเจน

```
App Layer (Top)
     │
     ▼
Feature Modules (Login, Home, Products, Cart)
     │
     ▼
Domain Layer (UseCases, Repositories)
     │
     ▼
Data Layer (Network, Storage, Database)
     │
     ▼
Foundation/Core (Extensions, Utilities, Design System)
     │
     ▼
Third-party Dependencies (Bottom)
```

### ตัวอย่าง Dependency Graph ที่ถูกต้อง

```
LoginFeature ──depends on──► AuthDomain
                              │
                              ├──► NetworkingModule
                              └──► StorageModule

ProductsFeature ──depends on──► ProductsDomain
                                │
                                ├──► NetworkingModule
                                └──► DesignSystem

CartFeature ──depends on──► CartDomain
                            │
                            ├──► ProductsDomain (shared models)
                            └──► NetworkingModule
```

### ตัวอย่าง Dependency Graph ที่ผิด (Circular)

```swift
// ❌ WRONG - Circular Dependency
// Module A
import ModuleB
public func doSomething() { ModuleB.helper() }

// Module B  
import ModuleA   // ← ทำให้เกิด Circular!
public func helper() { ModuleA.doSomething() }
```

### แก้ Circular Dependency

```swift
// ✅ CORRECT - Extract shared interface to Core
// CoreModule/Protocol.swift
public protocol SharedProtocol {
    func sharedOperation()
}

// Module A - depends on CoreModule only
import CoreModule
public func doSomething(handler: SharedProtocol) {
    handler.sharedOperation()
}

// Module B - depends on CoreModule only
import CoreModule
public class Implementation: SharedProtocol {
    public func sharedOperation() { /* ... */ }
}
```

---

## 5. Feature Modules

Feature Module คือ Module ที่รับผิดชอบ User Feature หนึ่งๆ อย่างสมบูรณ์

### โครงสร้าง Feature Module

```
LoginFeature/
├── Package.swift
├── Sources/
│   └── LoginFeature/
│       ├── LoginFeature.swift     # Public interface
│       ├── Presentation/
│       │   ├── LoginView.swift
│       │   └── LoginViewModel.swift
│       ├── Domain/
│       │   ├── LoginUseCase.swift
│       │   └── AuthRepository.swift
│       └── Data/
│           └── AuthService.swift
└── Tests/
    └── LoginFeatureTests/
```

### Public Interface ของ Feature

```swift
// LoginFeature.swift - Public API
import SwiftUI

// ✅ เปิดเผยเฉพาะสิ่งที่จำเป็น
public struct LoginFeature {
    
    // Factory method สำหรับสร้าง View
    public static func makeView(
        coordinator: LoginCoordinatorProtocol
    ) -> some View {
        LoginView(
            viewModel: LoginViewModel(
                coordinator: coordinator,
                useCase: LoginUseCase(
                    repository: AuthRepository()
                )
            )
        )
    }
    
    // Configuration
    public struct Configuration {
        public let allowBiometrics: Bool
        public let allowSocialLogin: Bool
        
        public init(allowBiometrics: Bool = true, allowSocialLogin: Bool = true) {
            self.allowBiometrics = allowBiometrics
            self.allowSocialLogin = allowSocialLogin
        }
    }
}

// Protocol สำหรับ Coordinator
public protocol LoginCoordinatorProtocol: AnyObject {
    func loginDidSucceed(user: AuthenticatedUser)
    func loginDidCancel()
    func showForgotPassword()
    func showRegistration()
}
```

### Feature Module ภายใน (Internal)

```swift
// LoginViewModel.swift - internal, ไม่เปิดเผยออกไป
import SwiftUI
import AuthDomain

@MainActor
final class LoginViewModel: ObservableObject {
    @Published var email: String = ""
    @Published var password: String = ""
    @Published var isLoading: Bool = false
    @Published var errorMessage: String?
    
    private let useCase: LoginUseCaseProtocol
    private weak var coordinator: LoginCoordinatorProtocol?
    
    init(coordinator: LoginCoordinatorProtocol, useCase: LoginUseCaseProtocol) {
        self.coordinator = coordinator
        self.useCase = useCase
    }
    
    func login() async {
        guard validateInput() else { return }
        
        isLoading = true
        errorMessage = nil
        
        do {
            let user = try await useCase.login(email: email, password: password)
            coordinator?.loginDidSucceed(user: user)
        } catch {
            errorMessage = error.localizedDescription
        }
        
        isLoading = false
    }
    
    private func validateInput() -> Bool {
        guard !email.isEmpty, !password.isEmpty else {
            errorMessage = "กรุณากรอกอีเมลและรหัสผ่าน"
            return false
        }
        return true
    }
}
```

---

## 6. Core Modules

Core Modules คือ Modules ที่ให้บริการพื้นฐานที่ Feature Modules ทุกตัวต้องการ

### Analytics Module

```swift
// Sources/AnalyticsModule/Analytics.swift
import Foundation

// Protocol-first approach
public protocol AnalyticsTracker {
    func track(event: AnalyticsEvent)
    func identify(userId: String, properties: [String: Any])
    func reset()
}

public struct AnalyticsEvent {
    public let name: String
    public let properties: [String: Any]
    public let timestamp: Date
    
    public init(name: String, properties: [String: Any] = [:]) {
        self.name = name
        self.properties = properties
        self.timestamp = Date()
    }
}

// Composite tracker ที่รวมหลาย trackers
public final class CompositeAnalyticsTracker: AnalyticsTracker {
    private var trackers: [AnalyticsTracker]
    
    public init(trackers: [AnalyticsTracker] = []) {
        self.trackers = trackers
    }
    
    public func addTracker(_ tracker: AnalyticsTracker) {
        trackers.append(tracker)
    }
    
    public func track(event: AnalyticsEvent) {
        trackers.forEach { $0.track(event: event) }
    }
    
    public func identify(userId: String, properties: [String: Any]) {
        trackers.forEach { $0.identify(userId: userId, properties: properties) }
    }
    
    public func reset() {
        trackers.forEach { $0.reset() }
    }
}

// Concrete implementations
public final class FirebaseAnalyticsTracker: AnalyticsTracker {
    public init() {}
    
    public func track(event: AnalyticsEvent) {
        // Firebase.logEvent(event.name, parameters: event.properties)
        print("[Firebase] Track: \(event.name)")
    }
    
    public func identify(userId: String, properties: [String: Any]) {
        // Firebase.setUserID(userId)
    }
    
    public func reset() {
        // Firebase.setUserID(nil)
    }
}
```

### Logging Module

```swift
// Sources/LoggingModule/Logger.swift
import OSLog

public protocol LoggerProtocol {
    func debug(_ message: String, file: String, line: Int)
    func info(_ message: String, file: String, line: Int)
    func warning(_ message: String, file: String, line: Int)
    func error(_ message: String, error: Error?, file: String, line: Int)
}

public final class SystemLogger: LoggerProtocol {
    private let logger: os.Logger
    
    public init(subsystem: String = Bundle.main.bundleIdentifier ?? "app",
                category: String = "general") {
        self.logger = os.Logger(subsystem: subsystem, category: category)
    }
    
    public func debug(_ message: String, file: String = #file, line: Int = #line) {
        logger.debug("[\(file.components(separatedBy: "/").last ?? ""):\(line)] \(message)")
    }
    
    public func info(_ message: String, file: String = #file, line: Int = #line) {
        logger.info("[\(file.components(separatedBy: "/").last ?? ""):\(line)] \(message)")
    }
    
    public func warning(_ message: String, file: String = #file, line: Int = #line) {
        logger.warning("[\(file.components(separatedBy: "/").last ?? ""):\(line)] ⚠️ \(message)")
    }
    
    public func error(_ message: String, error: Error? = nil, file: String = #file, line: Int = #line) {
        if let error = error {
            logger.error("[\(file.components(separatedBy: "/").last ?? ""):\(line)] ❌ \(message): \(error.localizedDescription)")
        } else {
            logger.error("[\(file.components(separatedBy: "/").last ?? ""):\(line)] ❌ \(message)")
        }
    }
}
```

---

## 7. UI Modules

### Design System Module

```swift
// Sources/DesignSystem/DesignSystem.swift
import SwiftUI

// Design Tokens
public enum DesignSystem {
    
    // MARK: - Colors
    public enum Colors {
        public static let primary = Color("Primary", bundle: .module)
        public static let secondary = Color("Secondary", bundle: .module)
        public static let background = Color("Background", bundle: .module)
        public static let surface = Color("Surface", bundle: .module)
        public static let onPrimary = Color("OnPrimary", bundle: .module)
        public static let error = Color("Error", bundle: .module)
        public static let success = Color("Success", bundle: .module)
        public static let warning = Color("Warning", bundle: .module)
    }
    
    // MARK: - Typography
    public enum Typography {
        public static let headline1 = Font.system(size: 32, weight: .bold, design: .default)
        public static let headline2 = Font.system(size: 24, weight: .semibold, design: .default)
        public static let headline3 = Font.system(size: 20, weight: .semibold, design: .default)
        public static let body1 = Font.system(size: 16, weight: .regular, design: .default)
        public static let body2 = Font.system(size: 14, weight: .regular, design: .default)
        public static let caption = Font.system(size: 12, weight: .regular, design: .default)
        public static let button = Font.system(size: 16, weight: .semibold, design: .default)
    }
    
    // MARK: - Spacing
    public enum Spacing {
        public static let xs: CGFloat = 4
        public static let sm: CGFloat = 8
        public static let md: CGFloat = 16
        public static let lg: CGFloat = 24
        public static let xl: CGFloat = 32
        public static let xxl: CGFloat = 48
    }
    
    // MARK: - Corner Radius
    public enum CornerRadius {
        public static let small: CGFloat = 4
        public static let medium: CGFloat = 8
        public static let large: CGFloat = 16
        public static let full: CGFloat = 999
    }
    
    // MARK: - Shadows
    public enum Shadows {
        public static let small = ShadowStyle(
            color: .black.opacity(0.1),
            radius: 4,
            x: 0,
            y: 2
        )
        public static let medium = ShadowStyle(
            color: .black.opacity(0.15),
            radius: 8,
            x: 0,
            y: 4
        )
    }
}

public struct ShadowStyle {
    public let color: Color
    public let radius: CGFloat
    public let x: CGFloat
    public let y: CGFloat
}
```

### Shared UI Components

```swift
// Sources/DesignSystem/Components/DSButton.swift
import SwiftUI

public struct DSButton: View {
    
    public enum Style {
        case primary
        case secondary
        case outline
        case ghost
        case destructive
    }
    
    public enum Size {
        case small
        case medium
        case large
    }
    
    private let title: String
    private let style: Style
    private let size: Size
    private let isLoading: Bool
    private let isDisabled: Bool
    private let action: () -> Void
    
    public init(
        _ title: String,
        style: Style = .primary,
        size: Size = .medium,
        isLoading: Bool = false,
        isDisabled: Bool = false,
        action: @escaping () -> Void
    ) {
        self.title = title
        self.style = style
        self.size = size
        self.isLoading = isLoading
        self.isDisabled = isDisabled
        self.action = action
    }
    
    public var body: some View {
        Button(action: action) {
            HStack(spacing: DesignSystem.Spacing.sm) {
                if isLoading {
                    ProgressView()
                        .progressViewStyle(.circular)
                        .tint(textColor)
                        .scaleEffect(0.8)
                }
                
                Text(title)
                    .font(buttonFont)
                    .foregroundColor(textColor)
            }
            .frame(maxWidth: .infinity)
            .frame(height: buttonHeight)
            .background(backgroundColor)
            .cornerRadius(DesignSystem.CornerRadius.medium)
            .overlay(
                RoundedRectangle(cornerRadius: DesignSystem.CornerRadius.medium)
                    .stroke(borderColor, lineWidth: style == .outline ? 1.5 : 0)
            )
            .opacity(isDisabled || isLoading ? 0.6 : 1)
        }
        .disabled(isDisabled || isLoading)
    }
    
    private var buttonHeight: CGFloat {
        switch size {
        case .small: return 36
        case .medium: return 48
        case .large: return 56
        }
    }
    
    private var buttonFont: Font {
        switch size {
        case .small: return DesignSystem.Typography.body2
        case .medium: return DesignSystem.Typography.button
        case .large: return DesignSystem.Typography.headline3
        }
    }
    
    private var backgroundColor: Color {
        switch style {
        case .primary: return DesignSystem.Colors.primary
        case .secondary: return DesignSystem.Colors.secondary
        case .outline, .ghost: return .clear
        case .destructive: return DesignSystem.Colors.error
        }
    }
    
    private var textColor: Color {
        switch style {
        case .primary, .secondary, .destructive: return DesignSystem.Colors.onPrimary
        case .outline: return DesignSystem.Colors.primary
        case .ghost: return DesignSystem.Colors.primary
        }
    }
    
    private var borderColor: Color {
        switch style {
        case .outline: return DesignSystem.Colors.primary
        default: return .clear
        }
    }
}

// TextField Component
public struct DSTextField: View {
    private let title: String
    private let placeholder: String
    @Binding private var text: String
    private let errorMessage: String?
    private let isSecure: Bool
    
    public init(
        _ title: String,
        placeholder: String = "",
        text: Binding<String>,
        errorMessage: String? = nil,
        isSecure: Bool = false
    ) {
        self.title = title
        self.placeholder = placeholder
        self._text = text
        self.errorMessage = errorMessage
        self.isSecure = isSecure
    }
    
    public var body: some View {
        VStack(alignment: .leading, spacing: DesignSystem.Spacing.xs) {
            Text(title)
                .font(DesignSystem.Typography.body2)
                .foregroundColor(.secondary)
            
            Group {
                if isSecure {
                    SecureField(placeholder, text: $text)
                } else {
                    TextField(placeholder, text: $text)
                }
            }
            .padding(DesignSystem.Spacing.md)
            .background(DesignSystem.Colors.surface)
            .cornerRadius(DesignSystem.CornerRadius.medium)
            .overlay(
                RoundedRectangle(cornerRadius: DesignSystem.CornerRadius.medium)
                    .stroke(errorMessage != nil ? DesignSystem.Colors.error : Color.gray.opacity(0.3), lineWidth: 1)
            )
            
            if let error = errorMessage {
                Text(error)
                    .font(DesignSystem.Typography.caption)
                    .foregroundColor(DesignSystem.Colors.error)
            }
        }
    }
}

// Preview
#Preview {
    VStack(spacing: 16) {
        DSButton("Primary Button") {}
        DSButton("Secondary", style: .secondary) {}
        DSButton("Outline", style: .outline) {}
        DSButton("Loading", isLoading: true) {}
        DSButton("Disabled", isDisabled: true) {}
    }
    .padding()
}
```

---

## 8. Network Modules

### การสร้าง Network Layer Module

```swift
// Sources/NetworkingModule/Endpoint.swift
import Foundation

public protocol Endpoint {
    var baseURL: URL { get }
    var path: String { get }
    var method: HTTPMethod { get }
    var headers: [String: String] { get }
    var parameters: Encodable? { get }
    var requiresAuthentication: Bool { get }
}

public extension Endpoint {
    var url: URL { baseURL.appendingPathComponent(path) }
    var headers: [String: String] { [:] }
    var parameters: Encodable? { nil }
    var requiresAuthentication: Bool { true }
}

public enum HTTPMethod: String {
    case get = "GET"
    case post = "POST"
    case put = "PUT"
    case patch = "PATCH"
    case delete = "DELETE"
}

// Sources/NetworkingModule/NetworkClient.swift
import Foundation

public protocol NetworkClientProtocol {
    func request<T: Decodable>(_ endpoint: Endpoint) async throws -> T
    func requestWithoutResponse(_ endpoint: Endpoint) async throws
}

public actor NetworkClient: NetworkClientProtocol {
    
    private let session: URLSession
    private let decoder: JSONDecoder
    private let tokenProvider: TokenProviderProtocol?
    
    public init(
        session: URLSession = .shared,
        decoder: JSONDecoder = .init(),
        tokenProvider: TokenProviderProtocol? = nil
    ) {
        self.session = session
        self.decoder = decoder
        self.tokenProvider = tokenProvider
        
        self.decoder.dateDecodingStrategy = .iso8601
        self.decoder.keyDecodingStrategy = .convertFromSnakeCase
    }
    
    public func request<T: Decodable>(_ endpoint: Endpoint) async throws -> T {
        let request = try await buildRequest(for: endpoint)
        
        let (data, response) = try await session.data(for: request)
        
        guard let httpResponse = response as? HTTPURLResponse else {
            throw NetworkError.invalidResponse
        }
        
        guard (200...299).contains(httpResponse.statusCode) else {
            throw NetworkError.httpError(statusCode: httpResponse.statusCode, data: data)
        }
        
        do {
            return try decoder.decode(T.self, from: data)
        } catch {
            throw NetworkError.decodingError(error)
        }
    }
    
    public func requestWithoutResponse(_ endpoint: Endpoint) async throws {
        let request = try await buildRequest(for: endpoint)
        let (_, response) = try await session.data(for: request)
        
        guard let httpResponse = response as? HTTPURLResponse,
              (200...299).contains(httpResponse.statusCode) else {
            throw NetworkError.invalidResponse
        }
    }
    
    private func buildRequest(for endpoint: Endpoint) async throws -> URLRequest {
        var request = URLRequest(url: endpoint.url)
        request.httpMethod = endpoint.method.rawValue
        request.timeoutInterval = 30
        
        // Default headers
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        request.setValue("application/json", forHTTPHeaderField: "Accept")
        
        // Custom headers
        endpoint.headers.forEach { key, value in
            request.setValue(value, forHTTPHeaderField: key)
        }
        
        // Authentication
        if endpoint.requiresAuthentication,
           let token = await tokenProvider?.accessToken {
            request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        }
        
        // Parameters
        if let parameters = endpoint.parameters {
            let encoder = JSONEncoder()
            encoder.dateEncodingStrategy = .iso8601
            request.httpBody = try encoder.encode(parameters)
        }
        
        return request
    }
}

// NetworkError
public enum NetworkError: LocalizedError {
    case invalidURL
    case invalidResponse
    case httpError(statusCode: Int, data: Data)
    case decodingError(Error)
    case noInternetConnection
    case timeout
    case unauthorized
    case serverError(String)
    
    public var errorDescription: String? {
        switch self {
        case .invalidURL: return "URL ไม่ถูกต้อง"
        case .invalidResponse: return "Response ไม่ถูกต้อง"
        case .httpError(let code, _): return "HTTP Error: \(code)"
        case .decodingError: return "ไม่สามารถอ่านข้อมูลได้"
        case .noInternetConnection: return "ไม่มีการเชื่อมต่ออินเทอร์เน็ต"
        case .timeout: return "หมดเวลาการเชื่อมต่อ"
        case .unauthorized: return "ไม่ได้รับอนุญาต"
        case .serverError(let message): return "Server Error: \(message)"
        }
    }
}
```

---

## 9. Data Modules

### Storage Module

```swift
// Sources/StorageModule/KeyValueStore.swift
import Foundation

public protocol KeyValueStoreProtocol {
    func set<T: Encodable>(_ value: T, forKey key: String) throws
    func get<T: Decodable>(_ type: T.Type, forKey key: String) -> T?
    func remove(forKey key: String)
    func removeAll()
    func contains(key: String) -> Bool
}

// UserDefaults Implementation
public final class UserDefaultsStore: KeyValueStoreProtocol {
    private let userDefaults: UserDefaults
    private let encoder = JSONEncoder()
    private let decoder = JSONDecoder()
    
    public init(suiteName: String? = nil) {
        if let suiteName = suiteName {
            self.userDefaults = UserDefaults(suiteName: suiteName) ?? .standard
        } else {
            self.userDefaults = .standard
        }
    }
    
    public func set<T: Encodable>(_ value: T, forKey key: String) throws {
        let data = try encoder.encode(value)
        userDefaults.set(data, forKey: key)
    }
    
    public func get<T: Decodable>(_ type: T.Type, forKey key: String) -> T? {
        guard let data = userDefaults.data(forKey: key) else { return nil }
        return try? decoder.decode(type, from: data)
    }
    
    public func remove(forKey key: String) {
        userDefaults.removeObject(forKey: key)
    }
    
    public func removeAll() {
        userDefaults.dictionaryRepresentation().keys.forEach {
            userDefaults.removeObject(forKey: $0)
        }
    }
    
    public func contains(key: String) -> Bool {
        userDefaults.object(forKey: key) != nil
    }
}

// Secure Storage (Keychain)
public final class KeychainStore: KeyValueStoreProtocol {
    private let serviceIdentifier: String
    
    public init(serviceIdentifier: String = Bundle.main.bundleIdentifier ?? "app") {
        self.serviceIdentifier = serviceIdentifier
    }
    
    public func set<T: Encodable>(_ value: T, forKey key: String) throws {
        let encoder = JSONEncoder()
        let data = try encoder.encode(value)
        
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: serviceIdentifier,
            kSecAttrAccount as String: key,
            kSecValueData as String: data
        ]
        
        // Delete existing item first
        SecItemDelete(query as CFDictionary)
        
        // Add new item
        let status = SecItemAdd(query as CFDictionary, nil)
        guard status == errSecSuccess else {
            throw StorageError.keychainError(status)
        }
    }
    
    public func get<T: Decodable>(_ type: T.Type, forKey key: String) -> T? {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: serviceIdentifier,
            kSecAttrAccount as String: key,
            kSecReturnData as String: true,
            kSecMatchLimit as String: kSecMatchLimitOne
        ]
        
        var result: AnyObject?
        let status = SecItemCopyMatching(query as CFDictionary, &result)
        
        guard status == errSecSuccess,
              let data = result as? Data else { return nil }
        
        return try? JSONDecoder().decode(type, from: data)
    }
    
    public func remove(forKey key: String) {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: serviceIdentifier,
            kSecAttrAccount as String: key
        ]
        SecItemDelete(query as CFDictionary)
    }
    
    public func removeAll() {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: serviceIdentifier
        ]
        SecItemDelete(query as CFDictionary)
    }
    
    public func contains(key: String) -> Bool {
        get(String.self, forKey: key) != nil
    }
}

public enum StorageError: LocalizedError {
    case keychainError(OSStatus)
    case encodingError
    case decodingError
    
    public var errorDescription: String? {
        switch self {
        case .keychainError(let status): return "Keychain error: \(status)"
        case .encodingError: return "Failed to encode data"
        case .decodingError: return "Failed to decode data"
        }
    }
}
```

---

## 10. Domain Modules

### Use Cases และ Repository Pattern

```swift
// Sources/AuthDomain/AuthDomain.swift

// MARK: - Models
public struct AuthenticatedUser {
    public let id: String
    public let email: String
    public let name: String
    public let role: UserRole
    public let accessToken: String
    public let refreshToken: String
    
    public init(id: String, email: String, name: String, role: UserRole,
                accessToken: String, refreshToken: String) {
        self.id = id
        self.email = email
        self.name = name
        self.role = role
        self.accessToken = accessToken
        self.refreshToken = refreshToken
    }
}

public enum UserRole: String, Codable {
    case user, admin, moderator
}

// MARK: - Repository Protocol
public protocol AuthRepositoryProtocol {
    func login(email: String, password: String) async throws -> AuthenticatedUser
    func logout() async throws
    func refreshToken() async throws -> String
    func getCurrentUser() -> AuthenticatedUser?
}

// MARK: - Use Cases
public protocol LoginUseCaseProtocol {
    func execute(email: String, password: String) async throws -> AuthenticatedUser
}

public final class LoginUseCase: LoginUseCaseProtocol {
    private let repository: AuthRepositoryProtocol
    private let validator: InputValidatorProtocol
    
    public init(repository: AuthRepositoryProtocol,
                validator: InputValidatorProtocol = InputValidator()) {
        self.repository = repository
        self.validator = validator
    }
    
    public func execute(email: String, password: String) async throws -> AuthenticatedUser {
        // Business logic validation
        try validator.validateEmail(email)
        try validator.validatePassword(password)
        
        // Call repository
        let user = try await repository.login(email: email, password: password)
        
        // Post-login business logic
        await trackLoginEvent(user: user)
        
        return user
    }
    
    private func trackLoginEvent(user: AuthenticatedUser) async {
        // Track analytics
    }
}

// Input Validator
public protocol InputValidatorProtocol {
    func validateEmail(_ email: String) throws
    func validatePassword(_ password: String) throws
}

public struct InputValidator: InputValidatorProtocol {
    public init() {}
    
    public func validateEmail(_ email: String) throws {
        guard !email.isEmpty else {
            throw ValidationError.emptyEmail
        }
        
        let emailRegex = #"^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$"#
        guard email.range(of: emailRegex, options: .regularExpression) != nil else {
            throw ValidationError.invalidEmailFormat
        }
    }
    
    public func validatePassword(_ password: String) throws {
        guard !password.isEmpty else {
            throw ValidationError.emptyPassword
        }
        
        guard password.count >= 8 else {
            throw ValidationError.passwordTooShort
        }
    }
}

public enum ValidationError: LocalizedError {
    case emptyEmail
    case invalidEmailFormat
    case emptyPassword
    case passwordTooShort
    
    public var errorDescription: String? {
        switch self {
        case .emptyEmail: return "กรุณากรอกอีเมล"
        case .invalidEmailFormat: return "รูปแบบอีเมลไม่ถูกต้อง"
        case .emptyPassword: return "กรุณากรอกรหัสผ่าน"
        case .passwordTooShort: return "รหัสผ่านต้องมีอย่างน้อย 8 ตัวอักษร"
        }
    }
}
```

---

## 11. Inter-Module Communication

### Event Bus Pattern

```swift
// Sources/CoreModule/EventBus.swift
import Foundation
import Combine

public final class EventBus {
    public static let shared = EventBus()
    
    private var subjects: [ObjectIdentifier: Any] = [:]
    private let lock = NSLock()
    
    private init() {}
    
    public func publisher<Event>(for eventType: Event.Type) -> AnyPublisher<Event, Never> {
        let key = ObjectIdentifier(eventType)
        
        lock.lock()
        defer { lock.unlock() }
        
        if let existing = subjects[key] as? PassthroughSubject<Event, Never> {
            return existing.eraseToAnyPublisher()
        }
        
        let subject = PassthroughSubject<Event, Never>()
        subjects[key] = subject
        return subject.eraseToAnyPublisher()
    }
    
    public func publish<Event>(_ event: Event) {
        let key = ObjectIdentifier(type(of: event))
        
        lock.lock()
        defer { lock.unlock() }
        
        (subjects[key] as? PassthroughSubject<Event, Never>)?.send(event)
    }
}

// Events
public struct UserLoggedInEvent {
    public let user: AuthenticatedUser
    public init(user: AuthenticatedUser) {
        self.user = user
    }
}

public struct UserLoggedOutEvent {
    public init() {}
}

public struct CartUpdatedEvent {
    public let itemCount: Int
    public init(itemCount: Int) {
        self.itemCount = itemCount
    }
}

// ใช้งานใน Feature Module
class LoginViewModel: ObservableObject {
    private var cancellables = Set<AnyCancellable>()
    
    func loginSuccess(user: AuthenticatedUser) {
        // แจ้ง Modules อื่นๆ ว่า login สำเร็จแล้ว
        EventBus.shared.publish(UserLoggedInEvent(user: user))
    }
}

class AppCoordinator {
    init() {
        // Subscribe to events
        EventBus.shared
            .publisher(for: UserLoggedInEvent.self)
            .sink { [weak self] event in
                self?.handleUserLoggedIn(event.user)
            }
            .store(in: &cancellables)
        
        EventBus.shared
            .publisher(for: UserLoggedOutEvent.self)
            .sink { [weak self] _ in
                self?.handleUserLoggedOut()
            }
            .store(in: &cancellables)
    }
    
    private var cancellables = Set<AnyCancellable>()
    
    private func handleUserLoggedIn(_ user: AuthenticatedUser) {
        // Navigate to main screen
    }
    
    private func handleUserLoggedOut() {
        // Navigate to login screen
    }
}
```

---

## 12. Protocol-Based Boundaries

### กำหนด Boundary ด้วย Protocols

```swift
// Public interface สำหรับ Products Feature
public protocol ProductsFeatureInterface {
    func makeProductListView() -> AnyView
    func makeProductDetailView(productId: String) -> AnyView
    func makeCartView() -> AnyView
}

// Implementation (ซ่อน internals)
public final class ProductsFeature: ProductsFeatureInterface {
    private let dependencies: ProductsDependencies
    
    public init(dependencies: ProductsDependencies) {
        self.dependencies = dependencies
    }
    
    public func makeProductListView() -> AnyView {
        AnyView(
            ProductListView(
                viewModel: ProductListViewModel(
                    repository: ProductsRepository(
                        networkClient: dependencies.networkClient
                    )
                )
            )
        )
    }
    
    public func makeProductDetailView(productId: String) -> AnyView {
        AnyView(
            ProductDetailView(
                viewModel: ProductDetailViewModel(
                    productId: productId,
                    repository: ProductsRepository(
                        networkClient: dependencies.networkClient
                    )
                )
            )
        )
    }
    
    public func makeCartView() -> AnyView {
        AnyView(CartView())
    }
}

// Dependencies
public struct ProductsDependencies {
    public let networkClient: NetworkClientProtocol
    public let analytics: AnalyticsTracker
    public let storage: KeyValueStoreProtocol
    
    public init(
        networkClient: NetworkClientProtocol,
        analytics: AnalyticsTracker,
        storage: KeyValueStoreProtocol
    ) {
        self.networkClient = networkClient
        self.analytics = analytics
        self.storage = storage
    }
}
```

---

## 13. Dependency Injection Across Modules

### Service Locator Pattern

```swift
// Sources/CoreModule/ServiceLocator.swift
public final class ServiceLocator {
    public static let shared = ServiceLocator()
    
    private var services: [ObjectIdentifier: Any] = [:]
    private let lock = NSLock()
    
    private init() {}
    
    public func register<Service>(_ service: Service, for type: Service.Type) {
        lock.lock()
        defer { lock.unlock() }
        services[ObjectIdentifier(type)] = service
    }
    
    public func resolve<Service>(_ type: Service.Type) -> Service? {
        lock.lock()
        defer { lock.unlock() }
        return services[ObjectIdentifier(type)] as? Service
    }
    
    public func resolve<Service>(_ type: Service.Type) throws -> Service {
        guard let service: Service = resolve(type) else {
            throw ServiceLocatorError.serviceNotFound(String(describing: type))
        }
        return service
    }
}

enum ServiceLocatorError: LocalizedError {
    case serviceNotFound(String)
    
    var errorDescription: String? {
        switch self {
        case .serviceNotFound(let type):
            return "Service not found: \(type)"
        }
    }
}

// ลงทะเบียน services ตอนเริ่ม app
func setupServices() {
    let networkClient = NetworkClient()
    ServiceLocator.shared.register(networkClient as NetworkClientProtocol, for: NetworkClientProtocol.self)
    
    let analytics = CompositeAnalyticsTracker()
    ServiceLocator.shared.register(analytics as AnalyticsTracker, for: AnalyticsTracker.self)
    
    let storage = UserDefaultsStore()
    ServiceLocator.shared.register(storage as KeyValueStoreProtocol, for: KeyValueStoreProtocol.self)
}
```

### Property Wrapper สำหรับ DI

```swift
@propertyWrapper
public struct Injected<Dependency> {
    private var dependency: Dependency?
    
    public var wrappedValue: Dependency {
        get {
            if let dependency = dependency {
                return dependency
            }
            guard let resolved: Dependency = ServiceLocator.shared.resolve(Dependency.self) else {
                fatalError("Dependency \(Dependency.self) not registered")
            }
            return resolved
        }
    }
    
    public init() {}
    
    public init(_ dependency: Dependency) {
        self.dependency = dependency
    }
}

// ใช้งาน
class ProductViewModel: ObservableObject {
    @Injected private var networkClient: NetworkClientProtocol
    @Injected private var analytics: AnalyticsTracker
    
    func loadProducts() async throws {
        // ใช้ dependencies โดยตรง
        let products: [Product] = try await networkClient.request(ProductEndpoint.list)
        analytics.track(event: AnalyticsEvent(name: "products_loaded"))
    }
}
```

---

## 14. TCA (The Composable Architecture) Overview

TCA คือ Architecture Pattern ที่สร้างโดย Point-Free ออกแบบมาสำหรับ SwiftUI โดยเฉพาะ

### หลักการของ TCA

```
State → View → User Action → Reducer → New State → View (loop)
```

### ส่วนประกอบหลัก

```swift
import ComposableArchitecture

// 1. State - ข้อมูลทั้งหมดของ Feature
struct ProductsFeature: Reducer {
    
    struct State: Equatable {
        var products: [Product] = []
        var isLoading: Bool = false
        var errorMessage: String?
        var searchQuery: String = ""
        var selectedProduct: Product?
    }
    
    // 2. Action - สิ่งที่ผู้ใช้หรือระบบทำ
    enum Action {
        case onAppear
        case searchQueryChanged(String)
        case productTapped(Product)
        case refreshButtonTapped
        
        // Results
        case productsLoaded([Product])
        case loadingFailed(String)
    }
    
    // 3. Dependencies
    @Dependency(\.apiClient) var apiClient
    
    // 4. Reducer - logic สำหรับ state transitions
    var body: some ReducerOf<Self> {
        Reduce { state, action in
            switch action {
            case .onAppear:
                state.isLoading = true
                return .run { send in
                    do {
                        let products = try await apiClient.fetchProducts()
                        await send(.productsLoaded(products))
                    } catch {
                        await send(.loadingFailed(error.localizedDescription))
                    }
                }
                
            case .searchQueryChanged(let query):
                state.searchQuery = query
                return .none
                
            case .productTapped(let product):
                state.selectedProduct = product
                return .none
                
            case .refreshButtonTapped:
                state.isLoading = true
                state.errorMessage = nil
                return .run { send in
                    do {
                        let products = try await apiClient.fetchProducts()
                        await send(.productsLoaded(products))
                    } catch {
                        await send(.loadingFailed(error.localizedDescription))
                    }
                }
                
            case .productsLoaded(let products):
                state.products = products
                state.isLoading = false
                return .none
                
            case .loadingFailed(let message):
                state.errorMessage = message
                state.isLoading = false
                return .none
            }
        }
    }
}

// 5. View
struct ProductsView: View {
    let store: StoreOf<ProductsFeature>
    
    var body: some View {
        WithViewStore(store, observe: { $0 }) { viewStore in
            VStack {
                TextField("ค้นหา...", text: viewStore.binding(
                    get: \.searchQuery,
                    send: ProductsFeature.Action.searchQueryChanged
                ))
                
                if viewStore.isLoading {
                    ProgressView()
                } else if let error = viewStore.errorMessage {
                    Text(error)
                        .foregroundColor(.red)
                } else {
                    List(viewStore.products) { product in
                        Button(product.name) {
                            viewStore.send(.productTapped(product))
                        }
                    }
                }
            }
            .onAppear {
                viewStore.send(.onAppear)
            }
            .refreshable {
                viewStore.send(.refreshButtonTapped)
            }
        }
    }
}
```

### Composing Reducers

```swift
// Parent Feature ที่รวม child features
struct AppFeature: Reducer {
    struct State {
        var home: HomeFeature.State = .init()
        var products: ProductsFeature.State = .init()
        var cart: CartFeature.State = .init()
        var profile: ProfileFeature.State = .init()
        var selectedTab: Tab = .home
    }
    
    enum Tab {
        case home, products, cart, profile
    }
    
    enum Action {
        case home(HomeFeature.Action)
        case products(ProductsFeature.Action)
        case cart(CartFeature.Action)
        case profile(ProfileFeature.Action)
        case tabSelected(Tab)
    }
    
    var body: some ReducerOf<Self> {
        Scope(state: \.home, action: /Action.home) {
            HomeFeature()
        }
        Scope(state: \.products, action: /Action.products) {
            ProductsFeature()
        }
        Scope(state: \.cart, action: /Action.cart) {
            CartFeature()
        }
        Scope(state: \.profile, action: /Action.profile) {
            ProfileFeature()
        }
        
        Reduce { state, action in
            switch action {
            case .tabSelected(let tab):
                state.selectedTab = tab
                return .none
            default:
                return .none
            }
        }
    }
}
```

---

## 15. Benefits and Tradeoffs

### ประโยชน์ของ Modular Architecture

```
ด้าน Build Time:
├── Incremental Compilation: เปลี่ยน Module A ไม่ต้อง recompile Module B
├── Parallel Compilation: หลาย Modules compile พร้อมกัน
└── Caching: Module ที่ไม่เปลี่ยน ใช้ cached version

ด้าน Code Quality:
├── Clear Ownership: รู้ว่า Code นี้ของทีมไหน
├── Enforced Boundaries: ไม่สามารถ access internals ของ Module อื่น
└── Better Testing: Test แต่ละ Module แยกกัน

ด้าน Team Productivity:
├── Parallel Development: หลายทีมทำงานพร้อมกัน
├── Fewer Merge Conflicts: แต่ละทีมแก้ไขคนละ Module
└── Feature Flags: Enable/Disable features ง่ายๆ
```

### Tradeoffs

```
ความซับซ้อนที่เพิ่มขึ้น:
├── Initial Setup: ใช้เวลานานกว่าในการ setup
├── Boilerplate: ต้องเขียน Public interfaces
└── Coordination: ต้องประสานงานระหว่าง Module owners

Over-engineering Risk:
├── Small apps: อาจไม่คุ้มค่า
├── Single developer: ไม่ได้ประโยชน์เรื่อง parallel work
└── Premature abstraction: แบ่ง Module เร็วเกินไป
```

---

## 16. Module Testing

### Unit Testing Modules

```swift
// Tests/AuthDomainTests/LoginUseCaseTests.swift
import XCTest
@testable import AuthDomain

final class LoginUseCaseTests: XCTestCase {
    
    var sut: LoginUseCase!
    var mockRepository: MockAuthRepository!
    
    override func setUp() {
        super.setUp()
        mockRepository = MockAuthRepository()
        sut = LoginUseCase(
            repository: mockRepository,
            validator: InputValidator()
        )
    }
    
    override func tearDown() {
        sut = nil
        mockRepository = nil
        super.tearDown()
    }
    
    func testLoginSuccess() async throws {
        // Arrange
        let expectedUser = AuthenticatedUser(
            id: "123",
            email: "test@example.com",
            name: "Test User",
            role: .user,
            accessToken: "token",
            refreshToken: "refresh"
        )
        mockRepository.stubbedUser = expectedUser
        
        // Act
        let result = try await sut.execute(
            email: "test@example.com",
            password: "password123"
        )
        
        // Assert
        XCTAssertEqual(result.id, expectedUser.id)
        XCTAssertEqual(result.email, expectedUser.email)
        XCTAssertTrue(mockRepository.loginCalled)
    }
    
    func testLoginWithInvalidEmail() async {
        // Arrange
        let invalidEmail = "not-an-email"
        
        // Act & Assert
        do {
            _ = try await sut.execute(email: invalidEmail, password: "password123")
            XCTFail("Should have thrown ValidationError.invalidEmailFormat")
        } catch let error as ValidationError {
            XCTAssertEqual(error, .invalidEmailFormat)
        } catch {
            XCTFail("Unexpected error: \(error)")
        }
    }
    
    func testLoginWithShortPassword() async {
        // Act & Assert
        do {
            _ = try await sut.execute(email: "test@example.com", password: "short")
            XCTFail("Should have thrown ValidationError.passwordTooShort")
        } catch let error as ValidationError {
            XCTAssertEqual(error, .passwordTooShort)
        } catch {
            XCTFail("Unexpected error: \(error)")
        }
    }
}

// Mock Objects
class MockAuthRepository: AuthRepositoryProtocol {
    var stubbedUser: AuthenticatedUser?
    var shouldThrowError: Error?
    var loginCalled = false
    
    func login(email: String, password: String) async throws -> AuthenticatedUser {
        loginCalled = true
        
        if let error = shouldThrowError {
            throw error
        }
        
        guard let user = stubbedUser else {
            throw NSError(domain: "test", code: -1)
        }
        
        return user
    }
    
    func logout() async throws {}
    
    func refreshToken() async throws -> String {
        return "new-token"
    }
    
    func getCurrentUser() -> AuthenticatedUser? {
        return stubbedUser
    }
}
```

### Integration Testing

```swift
// Tests/AppTests/IntegrationTests.swift
import XCTest
import AuthDomain
import NetworkingModule

final class AuthIntegrationTests: XCTestCase {
    
    func testLoginFlow() async throws {
        // ใช้ Mock Server
        let mockServer = MockHTTPServer()
        mockServer.stub(.post, "/auth/login") { _ in
            return HTTPResponse(
                statusCode: 200,
                body: """
                {
                    "id": "123",
                    "email": "test@example.com",
                    "name": "Test User",
                    "access_token": "token123",
                    "refresh_token": "refresh123"
                }
                """
            )
        }
        
        let networkClient = NetworkClient(baseURL: mockServer.baseURL)
        let repository = AuthRepository(networkClient: networkClient)
        let useCase = LoginUseCase(repository: repository)
        
        let user = try await useCase.execute(
            email: "test@example.com",
            password: "password123"
        )
        
        XCTAssertEqual(user.email, "test@example.com")
        XCTAssertEqual(user.accessToken, "token123")
    }
}
```

---

## 17. Build Time Improvements with Modules

### วิเคราะห์ Build Time

```bash
# ใช้ -Xfrontend -debug-time-function-bodies เพื่อดูว่า function ไหน compile นาน
# เพิ่มใน Build Settings > Swift Compiler - Custom Flags > Other Swift Flags

# ใช้ xcode-build-times เพื่อวิเคราะห์
brew install xcbeautify
xcodebuild clean build | xcbeautify
```

### เทคนิคเพิ่มความเร็ว

```swift
// 1. ลด Type Inference
// ❌ Slow - compiler ต้อง infer type
let result = items.filter { $0.price > 100 }
    .map { $0.name }
    .sorted()

// ✅ Fast - ระบุ type ชัดเจน
let result: [String] = items
    .filter { (item: Product) -> Bool in item.price > 100 }
    .map { (item: Product) -> String in item.name }
    .sorted()

// 2. หลีกเลี่ยง Complex Expressions
// ❌ ช้า
let complexValue = foo.bar?.baz ?? fooBar + bazFoo * 10 - (bar?.count ?? 0)

// ✅ เร็วกว่า
let barBaz = foo.bar?.baz
let fooBarResult = fooBar + bazFoo * 10
let barCount = bar?.count ?? 0
let complexValue = barBaz ?? fooBarResult - barCount

// 3. ใช้ @inlinable สำหรับ performance-critical code ใน Module
@inlinable
public func fastPath(value: Int) -> Int {
    // Inlined at call site
    return value * 2
}
```

### Module Caching Strategy

```swift
// Package.swift ที่ optimize สำหรับ build speed
let package = Package(
    name: "MyApp",
    products: [
        // แยก products ชัดเจน
        .library(name: "DesignSystem", targets: ["DesignSystem"]),
        .library(name: "NetworkingModule", targets: ["NetworkingModule"]),
        .library(name: "LoginFeature", targets: ["LoginFeature"]),
    ],
    targets: [
        // DesignSystem ไม่ depend on อะไรเลย = compile เร็ว
        .target(name: "DesignSystem"),
        
        // NetworkingModule ไม่ depend on DesignSystem = compile parallel
        .target(name: "NetworkingModule"),
        
        // LoginFeature depend on ทั้งคู่
        .target(
            name: "LoginFeature",
            dependencies: ["DesignSystem", "NetworkingModule"]
        ),
    ]
)
```

---

## 18. Module Boundaries and Encapsulation

### กำหนด Boundaries ชัดเจน

```swift
// ✅ GOOD: เปิดเผยเฉพาะ public interface
public struct ProductsModule {
    
    // Public Models
    public struct Product: Identifiable, Sendable {
        public let id: String
        public let name: String
        public let price: Double
        public let imageURL: URL?
        
        public init(id: String, name: String, price: Double, imageURL: URL?) {
            self.id = id
            self.name = name
            self.price = price
            self.imageURL = imageURL
        }
    }
    
    // Public Factory
    @MainActor
    public static func makeProductListView(
        onProductSelected: @escaping (Product) -> Void
    ) -> some View {
        ProductListView(
            viewModel: ProductListViewModel(),
            onProductSelected: onProductSelected
        )
    }
}

// ❌ BAD: ไม่ได้ซ่อน internals
public class ProductListViewModel: ObservableObject {  // ควรเป็น internal
    @Published public var products: [Product] = []  // ควรเป็น internal
    var networkClient: NetworkClientProtocol!  // ควรเป็น private
    
    public func loadProducts() async {
        // ทุกคนสามารถเรียกได้ = ยากที่จะ control
    }
}
```

### Dependency Inversion

```swift
// ตาม Dependency Inversion Principle
// High-level modules ขึ้นอยู่กับ abstractions, ไม่ใช่ concrete implementations

// ❌ BAD: depend on concrete class
class LoginViewModel {
    private let service = AuthService()  // Concrete!
    
    func login() async throws {
        try await service.login(email: email, password: password)
    }
}

// ✅ GOOD: depend on protocol
class LoginViewModel {
    private let service: AuthServiceProtocol  // Abstract!
    
    init(service: AuthServiceProtocol = AuthService()) {
        self.service = service
    }
    
    func login() async throws {
        try await service.login(email: email, password: password)
    }
}

// Testing ง่าย
class MockAuthService: AuthServiceProtocol {
    var shouldSucceed = true
    
    func login(email: String, password: String) async throws -> User {
        if shouldSucceed { return User.mock }
        else { throw AuthError.invalidCredentials }
    }
}

let viewModel = LoginViewModel(service: MockAuthService())
```

---

## 19. Micro-Features Architecture

Micro-Features Architecture แบ่ง Feature ออกเป็น 5 Targets:

```
Feature/
├── Example/           # Demo App สำหรับ develop feature แยก
├── Interface/         # Public Protocol definitions
├── Implementation/    # Concrete implementations
├── Testing/           # Mock objects, test helpers
└── Tests/             # Unit tests
```

### ตัวอย่าง LoginFeature

```swift
// LoginFeatureInterface - ไม่มี implementation detail
public protocol LoginFeatureBuilding {
    func makeLoginView(coordinator: LoginCoordinating) -> AnyView
}

public protocol LoginCoordinating: AnyObject {
    func didLogin(user: User)
    func didTapForgotPassword()
    func didTapSignUp()
}

// LoginFeature (Implementation)
import LoginFeatureInterface

public final class LoginFeatureBuilder: LoginFeatureBuilding {
    private let dependencies: Dependencies
    
    public struct Dependencies {
        let authService: AuthServiceProtocol
        let analytics: AnalyticsTracking
    }
    
    public init(dependencies: Dependencies) {
        self.dependencies = dependencies
    }
    
    public func makeLoginView(coordinator: LoginCoordinating) -> AnyView {
        let viewModel = LoginViewModel(
            authService: dependencies.authService,
            analytics: dependencies.analytics,
            coordinator: coordinator
        )
        return AnyView(LoginView(viewModel: viewModel))
    }
}

// LoginFeatureTesting - Mock builder สำหรับ test
import LoginFeatureInterface

public final class MockLoginFeatureBuilder: LoginFeatureBuilding {
    public var makeLoginViewCalled = false
    public var lastCoordinator: LoginCoordinating?
    public var viewToReturn: AnyView = AnyView(EmptyView())
    
    public init() {}
    
    public func makeLoginView(coordinator: LoginCoordinating) -> AnyView {
        makeLoginViewCalled = true
        lastCoordinator = coordinator
        return viewToReturn
    }
}
```

---

## 20. Practical Exercise: Building a Modular iOS App

### โปรเจกต์: Modular E-Commerce App

#### Package Structure

```
ECommerceApp/
├── App/                              # Main App
├── Packages/
│   ├── CorePackage/
│   │   ├── Sources/
│   │   │   ├── DesignSystem/
│   │   │   ├── NetworkingModule/
│   │   │   ├── StorageModule/
│   │   │   └── AnalyticsModule/
│   │   └── Tests/
│   ├── DomainPackage/
│   │   ├── Sources/
│   │   │   ├── AuthDomain/
│   │   │   ├── ProductsDomain/
│   │   │   └── CartDomain/
│   │   └── Tests/
│   └── FeaturesPackage/
│       ├── Sources/
│       │   ├── LoginFeature/
│       │   ├── HomeFeature/
│       │   ├── ProductsFeature/
│       │   └── CartFeature/
│       └── Tests/
└── Package.swift
```

#### App Coordinator

```swift
// App/Coordinator/AppCoordinator.swift
import SwiftUI
import LoginFeature
import HomeFeature
import ProductsFeature
import CartFeature
import AuthDomain

@MainActor
class AppCoordinator: ObservableObject {
    
    @Published var appState: AppState = .launch
    
    enum AppState {
        case launch
        case onboarding
        case authenticated(User)
        case unauthenticated
    }
    
    private let authService: AuthServiceProtocol
    
    init(authService: AuthServiceProtocol = AuthService()) {
        self.authService = authService
    }
    
    func start() async {
        // ตรวจสอบ session
        if let user = authService.currentUser {
            appState = .authenticated(user)
        } else {
            appState = .unauthenticated
        }
    }
    
    // Create feature builders with injected dependencies
    func makeLoginFeature() -> LoginFeatureBuilding {
        LoginFeatureBuilder(dependencies: .init(
            authService: authService,
            analytics: AnalyticsService.shared
        ))
    }
    
    func makeHomeFeature() -> HomeFeatureBuilding {
        HomeFeatureBuilder(dependencies: .init(
            productService: ProductService(),
            analytics: AnalyticsService.shared
        ))
    }
    
    func makeCartFeature() -> CartFeatureBuilding {
        CartFeatureBuilder(dependencies: .init(
            cartService: CartService(),
            analytics: AnalyticsService.shared
        ))
    }
}

// Root View
struct AppRootView: View {
    @StateObject private var coordinator = AppCoordinator()
    
    var body: some View {
        Group {
            switch coordinator.appState {
            case .launch:
                LaunchView()
            case .onboarding:
                OnboardingView(coordinator: coordinator)
            case .authenticated:
                MainTabView(coordinator: coordinator)
            case .unauthenticated:
                coordinator.makeLoginFeature()
                    .makeLoginView(coordinator: coordinator)
            }
        }
        .task {
            await coordinator.start()
        }
    }
}

extension AppCoordinator: LoginCoordinating {
    func didLogin(user: User) {
        appState = .authenticated(user)
    }
    
    func didTapForgotPassword() {
        // Navigate to forgot password
    }
    
    func didTapSignUp() {
        // Navigate to sign up
    }
}
```

#### Main Tab View

```swift
struct MainTabView: View {
    let coordinator: AppCoordinator
    @State private var selectedTab: Tab = .home
    
    enum Tab: CaseIterable {
        case home, products, cart, profile
        
        var icon: String {
            switch self {
            case .home: return "house.fill"
            case .products: return "square.grid.2x2.fill"
            case .cart: return "cart.fill"
            case .profile: return "person.fill"
            }
        }
        
        var title: String {
            switch self {
            case .home: return "หน้าหลัก"
            case .products: return "สินค้า"
            case .cart: return "ตะกร้า"
            case .profile: return "โปรไฟล์"
            }
        }
    }
    
    var body: some View {
        TabView(selection: $selectedTab) {
            ForEach(Tab.allCases, id: \.self) { tab in
                tabView(for: tab)
                    .tabItem {
                        Label(tab.title, systemImage: tab.icon)
                    }
                    .tag(tab)
            }
        }
    }
    
    @ViewBuilder
    private func tabView(for tab: Tab) -> some View {
        switch tab {
        case .home:
            coordinator.makeHomeFeature().makeHomeView()
        case .products:
            coordinator.makeProductsFeature().makeProductListView(
                onProductSelected: { product in
                    // Navigate to product detail
                }
            )
        case .cart:
            coordinator.makeCartFeature().makeCartView()
        case .profile:
            ProfileView()
        }
    }
}
```

---

## 21. Module Testing Strategy

### Test Pyramid สำหรับ Modular Apps

```
         /\
        /  \
       / E2E \     ← น้อยที่สุด (ช้า, แพง)
      /------\
     /        \
    / Integration\  ← ปานกลาง
   /------------\
  /              \
 /   Unit Tests   \  ← มากที่สุด (เร็ว, ถูก)
/________________\
```

### Test ที่ดีสำหรับ Modules

```swift
// 1. Unit Test แต่ละ Module แยกกัน
class ProductsRepositoryTests: XCTestCase {
    func testFetchProducts() async throws {
        let mockClient = MockNetworkClient()
        mockClient.stub(ProductEndpoint.list, with: [Product.mock])
        
        let repository = ProductsRepository(networkClient: mockClient)
        let products = try await repository.fetchProducts()
        
        XCTAssertEqual(products.count, 1)
        XCTAssertEqual(products.first?.name, Product.mock.name)
    }
}

// 2. Integration Test ระหว่าง Modules
class LoginFlowTests: XCTestCase {
    func testCompleteLoginFlow() async throws {
        // Setup real modules but with mock external dependencies
        let mockServer = MockHTTPServer()
        setupLoginEndpoint(on: mockServer)
        
        let networkClient = NetworkClient(baseURL: mockServer.baseURL)
        let authRepository = AuthRepository(networkClient: networkClient)
        let loginUseCase = LoginUseCase(repository: authRepository)
        let viewModel = LoginViewModel(useCase: loginUseCase)
        
        // Simulate user interaction
        viewModel.email = "test@example.com"
        viewModel.password = "password123"
        await viewModel.login()
        
        // Assert
        XCTAssertNotNil(viewModel.loggedInUser)
        XCTAssertFalse(viewModel.isLoading)
        XCTAssertNil(viewModel.errorMessage)
    }
}
```

---

## 22. Summary

ในบทนี้เราได้เรียนรู้:

1. **Why Modular Architecture** - แก้ปัญหา Build Time, Coupling, Team Collaboration
2. **Framework Targets** - สร้าง Module ด้วย Xcode Framework Targets
3. **Swift Packages** - ใช้ SPM เป็น Module system ที่แนะนำ
4. **Dependency Graph** - ออกแบบ Module dependencies ที่ถูกต้อง
5. **Feature Modules** - แบ่ง App ตาม User Features
6. **Core Modules** - Networking, Storage, Analytics
7. **UI Modules** - Design System, Shared Components
8. **Protocol Boundaries** - ใช้ Protocols เป็น Module boundaries
9. **Dependency Injection** - Service Locator, Property Wrappers
10. **TCA Overview** - Architecture pattern สำหรับ SwiftUI
11. **Testing Strategy** - Unit + Integration testing ของ Modules
12. **Micro-Features** - แบ่ง Feature เป็น 5 Targets

### Key Principles

```swift
// 1. Depend on Abstractions, not Concretions
protocol NetworkClientProtocol { /* ... */ }
class ViewModel {
    let client: NetworkClientProtocol  // ✅ Protocol
    // NOT: let client: NetworkClient  // ❌ Concrete
}

// 2. One-way dependencies
// Features → Domain → Data → Foundation
// ห้าม: Domain → Features

// 3. Small, focused modules
// แต่ละ Module มี Single Responsibility ชัดเจน

// 4. Public interface ที่ minimal
// เปิดเผยเฉพาะสิ่งที่จำเป็น, ซ่อน implementation details

// 5. Test-friendly design
// แต่ละ Module ต้อง testable แยกกันได้
```

### เมื่อไหรควรใช้ Modular Architecture

**ควรใช้เมื่อ:**
- ทีมมีมากกว่า 3-4 คน
- App มี Features มากกว่า 10 features
- Build Time เริ่มช้า (> 2 นาที)
- ต้องการ reuse code ใน App อื่น

**ยังไม่จำเป็นเมื่อ:**
- App เล็กๆ (< 5 features)
- ทีมเล็ก (1-2 คน)
- Prototype หรือ MVP

### ขั้นตอนถัดไป

- ศึกษา The Composable Architecture (TCA) เพิ่มเติม
- เรียนรู้ Feature Flagging ใน Modular Apps
- ลอง Extract Module จาก Existing App
- ศึกษา Tuist สำหรับ Large-scale project management

---

*จบ Part 56 - Modular Architecture in iOS*
