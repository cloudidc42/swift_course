# ตอนที่ 67: Professional App Development

## บทนำ

การพัฒนา App แบบมืออาชีพไม่ได้หมายถึงแค่การเขียนโค้ดให้ทำงานได้ แต่ครอบคลุมทุกด้านตั้งแต่ Performance, Privacy, Analytics ไปจนถึงการ Integration กับระบบนิเวศของ Apple อย่างครบถ้วน บทนี้จะพาคุณผ่านทุกแง่มุมที่จำเป็นสำหรับการส่ง App ขึ้น App Store อย่างมืออาชีพ

---

## 1. Production App Checklist

### 1.1 Checklist ก่อน Submit App

```swift
// Production Readiness Checklist
// ✅ = พร้อม, ❌ = ยังไม่พร้อม, 🔄 = กำลังดำเนินการ

/*
=== FUNCTIONALITY ===
✅ ฟีเจอร์หลักทั้งหมดทำงานได้ถูกต้อง
✅ Edge cases ได้รับการจัดการ
✅ Error handling ครบถ้วน
✅ Unit tests ผ่านทั้งหมด
✅ UI tests ผ่านทั้งหมด
✅ Integration tests ผ่านทั้งหมด

=== PERFORMANCE ===
✅ Launch time < 400ms (cold start)
✅ Memory usage อยู่ในขอบเขตที่ยอมรับได้
✅ CPU usage ไม่สูงเกินไป
✅ Battery drain ผ่านการทดสอบ
✅ Network requests ถูก optimize

=== UI/UX ===
✅ รองรับ Dark Mode
✅ รองรับ Dynamic Type (ขนาดตัวอักษร)
✅ Accessibility (VoiceOver, Switch Control)
✅ Localization ครบถ้วน
✅ Landscape/Portrait orientation
✅ iPad support (ถ้าจำเป็น)
✅ Different screen sizes

=== SECURITY ===
✅ Data encryption สำหรับข้อมูลสำคัญ
✅ Keychain ใช้เก็บ credentials
✅ Certificate pinning (ถ้าจำเป็น)
✅ Jailbreak detection (ถ้าจำเป็น)
✅ Input validation

=== PRIVACY ===
✅ Privacy Policy URL
✅ Permission strings ใน Info.plist
✅ Privacy Manifest
✅ ATT implementation
✅ GDPR/CCPA/PDPA compliance

=== APP STORE ===
✅ App icons ครบทุกขนาด
✅ Screenshots ครบทุก device
✅ App description เขียนครบ
✅ Keywords optimized
✅ Version number ถูกต้อง
✅ Bundle ID ถูกต้อง

=== CRASH & ANALYTICS ===
✅ Crashlytics/Sentry integrated
✅ Analytics events defined
✅ A/B testing setup
✅ Feature flags configured
✅ Remote config setup
*/
```

### 1.2 Pre-release Testing Matrix

```swift
import XCTest

// การสร้าง Pre-release Test Suite
class ProductionReadinessTests: XCTestCase {
    
    // Test App Launch Performance
    func testAppLaunchPerformance() {
        measure(metrics: [XCTApplicationLaunchMetric()]) {
            XCUIApplication().launch()
        }
    }
    
    // Test Memory Usage
    func testMemoryUsage() {
        measure(metrics: [XCTMemoryMetric()]) {
            // ทำ operation หลักๆ
            let app = XCUIApplication()
            app.launch()
            // navigate through main screens
        }
    }
    
    // Test CPU Usage
    func testCPUUsage() {
        measure(metrics: [XCTCPUMetric()]) {
            let app = XCUIApplication()
            app.launch()
            // perform heavy operations
        }
    }
}
```

---

## 2. App Size Optimization

### 2.1 On-Demand Resources

```swift
// On-Demand Resources - โหลด asset เมื่อต้องการ
import Foundation

class ResourceManager {
    
    // Request on-demand resources
    func loadHighQualityAssets() {
        let resourceRequest = NSBundleResourceRequest(tags: ["HighQualityAssets"])
        
        resourceRequest.beginAccessingResources { error in
            if let error = error {
                print("Error loading resources: \(error)")
                return
            }
            
            DispatchQueue.main.async {
                // Resources are now available
                print("High quality assets loaded successfully")
            }
        }
    }
    
    // Conditional download
    func loadLevelAssets(level: Int) {
        let tag = "Level\(level)"
        let request = NSBundleResourceRequest(tags: [tag])
        
        // ตรวจสอบว่า download แล้วหรือยัง
        request.conditionallyBeginAccessingResources { resourcesAvailable in
            if resourcesAvailable {
                // ใช้ได้เลย
                self.startLevel(level)
            } else {
                // ต้อง download
                request.beginAccessingResources { error in
                    guard error == nil else { return }
                    DispatchQueue.main.async {
                        self.startLevel(level)
                    }
                }
            }
        }
    }
    
    private func startLevel(_ level: Int) {
        print("Starting level \(level)")
    }
}
```

### 2.2 Asset Catalog Optimization

```swift
// Image Optimization Strategy
import SwiftUI

struct OptimizedImageView: View {
    let imageName: String
    
    var body: some View {
        // ใช้ SF Symbols แทน custom images เมื่อทำได้
        Image(systemName: "star.fill")
            .resizable()
            .scaledToFit()
    }
}

// Asset Compression Configuration (ใน build settings)
/*
COMPRESS_PNG_FILES = YES
STRIP_PNG_TEXT = YES
ASSETCATALOG_COMPILER_OPTIMIZATION = space  // หรือ time
*/

// Dead Code Elimination
// ใช้ Link-Time Optimization
/*
LLVM_LTO = YES
DEAD_CODE_STRIPPING = YES
*/
```

### 2.3 Binary Size Reduction

```swift
// Swift Compiler Optimization
/*
Build Settings:
- Swift Optimization Level: Optimize for Speed (-O) สำหรับ Release
- Compilation Mode: Whole Module Optimization
- Debug Information Format: DWARF with dSYM File

Linker Flags:
- Dead code stripping: YES
- Strip Debug Symbols: YES (Release)
- Strip Linked Product: YES
*/

// ตัวอย่าง: ลด binary size ด้วย @_optimize
@_optimize(none)  // ไม่ optimize (สำหรับ debug)
func debugFunction() {
    print("Debug only")
}

// ใช้ conditional compilation
#if DEBUG
func debugHelper() {
    // Code นี้จะถูก strip ใน Release build
}
#endif
```

### 2.4 เครื่องมือวัด App Size

```bash
# ใช้ Xcode Archive แล้วดู App Thinning Size Report
# Product > Archive > Distribute App > App Store Connect
# จะได้ไฟล์ App Thinning Size Report.txt

# ตัวอย่าง output:
# Variant: Universal
# Supported variant descriptors: [device: iPhone12,1, os-version: 13.0]
# App + On Demand Resources size: 15.7 MB compressed, 38.7 MB uncompressed
```

---

## 3. Launch Time Optimization

### 3.1 Cold Start Optimization

```swift
// AppDelegate - ลด work ใน application(_:didFinishLaunchingWithOptions:)
import UIKit

@main
class AppDelegate: UIResponder, UIApplicationDelegate {
    
    func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
    ) -> Bool {
        
        // ✅ DO: ทำเฉพาะสิ่งที่จำเป็นสุดๆ
        setupCrashReporting()  // ต้องเป็นอันแรก
        
        // ❌ DON'T: อย่า initialize ทุกอย่างที่นี่
        // setupAnalytics()  // defer ไป
        // prefetchData()    // defer ไป
        // loadHeavyLibrary() // defer ไป
        
        return true
    }
    
    private func setupCrashReporting() {
        // Minimal setup เท่านั้น
        CrashReporter.shared.initialize()
    }
}

// Scene Delegate - defer non-critical setup
class SceneDelegate: UIResponder, UIWindowSceneDelegate {
    
    func scene(
        _ scene: UIScene,
        willConnectTo session: UISceneSession,
        options connectionOptions: UIScene.ConnectionOptions
    ) {
        guard let windowScene = scene as? UIWindowScene else { return }
        
        // Setup UI เท่านั้น
        let window = UIWindow(windowScene: windowScene)
        window.rootViewController = SplashViewController()
        window.makeKeyAndVisible()
        
        // Defer heavy initialization
        DispatchQueue.main.async {
            self.deferredInitialization()
        }
    }
    
    private func deferredInitialization() {
        // Setup analytics, remote config, etc.
        AnalyticsManager.shared.initialize()
        RemoteConfigManager.shared.fetch()
        FeatureFlagManager.shared.initialize()
    }
}
```

### 3.2 Measuring Launch Time

```swift
import UIKit
import os.signpost

// ใช้ os.signpost สำหรับ measurement ที่แม่นยำ
let launchLog = OSLog(subsystem: "com.yourapp", category: "Launch")

class LaunchTimeTracker {
    
    static let shared = LaunchTimeTracker()
    private var launchStart: CFTimeInterval = 0
    
    func markLaunchStart() {
        launchStart = CACurrentMediaTime()
        os_signpost(.begin, log: launchLog, name: "App Launch")
    }
    
    func markFirstFrameRendered() {
        let elapsed = CACurrentMediaTime() - launchStart
        os_signpost(.end, log: launchLog, name: "App Launch")
        
        print("Launch time: \(elapsed * 1000)ms")
        
        // Log to analytics
        AnalyticsManager.shared.log(event: "app_launch_time", 
                                     properties: ["duration_ms": elapsed * 1000])
    }
}

// SwiftUI Version
struct ContentView: View {
    @State private var isLaunching = true
    
    var body: some View {
        Group {
            if isLaunching {
                LaunchScreen()
            } else {
                MainView()
            }
        }
        .onAppear {
            LaunchTimeTracker.shared.markFirstFrameRendered()
            
            // Transition to main content
            DispatchQueue.main.asyncAfter(deadline: .now() + 0.1) {
                withAnimation {
                    isLaunching = false
                }
            }
        }
    }
}
```

### 3.3 Dylib Optimization

```swift
// ลด dynamic libraries ที่โหลดตอน launch
// ใน Build Settings:
/*
OTHER_LDFLAGS = -all_load  // หรือ -ObjC สำหรับ Objective-C categories

// ใช้ Static Libraries แทน Dynamic เมื่อทำได้
// SPM packages: .library(name: "MyLib", type: .static, ...)
*/

// Pre-warm optimization (iOS 15+)
// ระบบ pre-warm app ก่อน user เปิด
// ต้องระวัง side effects ใน initializers
class SafeInitializer {
    
    static let shared = SafeInitializer()
    
    // ❌ อย่าทำ side effects ที่นี่ - อาจถูกเรียกตอน pre-warm
    private init() {
        // Minimal setup only
    }
    
    // ✅ ทำใน explicit setup method แทน
    func setup() {
        // Heavy initialization here
    }
}
```

---

## 4. Crash Reporting

### 4.1 Firebase Crashlytics

```swift
// Firebase Crashlytics Integration
import Firebase
import FirebaseCrashlytics

// Setup ใน AppDelegate
func application(_ application: UIApplication, 
                 didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    FirebaseApp.configure()
    return true
}

// Custom Crash Reporting
class CrashReportingManager {
    
    // Log custom key-value pairs
    static func setCustomKey(_ key: String, value: String) {
        Crashlytics.crashlytics().setCustomValue(value, forKey: key)
    }
    
    // Log user identifier (anonymous)
    static func setUserID(_ id: String) {
        Crashlytics.crashlytics().setUserID(id)
    }
    
    // Log non-fatal errors
    static func recordError(_ error: Error, context: String? = nil) {
        var userInfo: [String: Any] = [:]
        if let context = context {
            userInfo["context"] = context
        }
        
        let nsError = error as NSError
        Crashlytics.crashlytics().record(
            error: NSError(
                domain: nsError.domain,
                code: nsError.code,
                userInfo: nsError.userInfo.merging(userInfo) { _, new in new }
            )
        )
    }
    
    // Log custom event (breadcrumb)
    static func log(_ message: String) {
        Crashlytics.crashlytics().log(message)
    }
    
    // Force test crash (สำหรับ testing เท่านั้น)
    #if DEBUG
    static func testCrash() {
        Crashlytics.crashlytics().setCrashlyticsCollectionEnabled(true)
        fatalError("Test crash")
    }
    #endif
}

// Usage
class NetworkManager {
    
    func fetchData() async throws -> Data {
        CrashReportingManager.log("Starting data fetch")
        CrashReportingManager.setCustomKey("last_api_call", value: "fetchData")
        
        do {
            let data = try await performRequest()
            CrashReportingManager.log("Data fetch successful")
            return data
        } catch {
            CrashReportingManager.recordError(error, context: "fetchData")
            throw error
        }
    }
    
    private func performRequest() async throws -> Data {
        // Network request implementation
        return Data()
    }
}
```

### 4.2 Sentry Integration

```swift
// Sentry Integration
import Sentry

// Setup
SentrySDK.start { options in
    options.dsn = "https://your-dsn@sentry.io/project-id"
    options.debug = false  // ปิดใน production
    options.tracesSampleRate = 0.1  // Sample 10% ของ transactions
    options.profilesSampleRate = 0.1
    options.enableAutoSessionTracking = true
    options.sessionTrackingIntervalMillis = 30000  // 30 seconds
    
    // Environment
    options.environment = "production"
    options.releaseName = Bundle.main.bundleIdentifier! + "@" +
        (Bundle.main.infoDictionary?["CFBundleShortVersionString"] as? String ?? "")
    
    // User privacy
    options.sendDefaultPii = false  // ไม่ส่ง PII
    
    // Filtering
    options.beforeSend = { event in
        // กรอง events ที่ไม่ต้องการ
        if event.message?.message?.contains("NetworkError") == true {
            return nil  // ไม่ส่ง
        }
        return event
    }
}

// Custom error tracking
class SentryManager {
    
    // Capture error with context
    static func captureError(_ error: Error, context: [String: Any]? = nil) {
        SentrySDK.capture(error: error) { scope in
            if let context = context {
                scope.setContext(value: context, key: "additional_info")
            }
        }
    }
    
    // Capture message
    static func captureMessage(_ message: String, level: SentryLevel = .info) {
        SentrySDK.capture(message: message) { scope in
            scope.setLevel(level)
        }
    }
    
    // Add breadcrumb
    static func addBreadcrumb(message: String, category: String, level: SentryLevel = .info) {
        let crumb = Breadcrumb()
        crumb.message = message
        crumb.category = category
        crumb.level = level
        crumb.timestamp = Date()
        SentrySDK.addBreadcrumb(crumb)
    }
    
    // Performance monitoring
    static func startTransaction(name: String, operation: String) -> Span {
        return SentrySDK.startTransaction(name: name, operation: operation)
    }
}

// Usage example
func loadUserProfile(userID: String) async {
    SentryManager.addBreadcrumb(
        message: "Loading user profile",
        category: "user",
        level: .info
    )
    
    let transaction = SentryManager.startTransaction(
        name: "Load User Profile",
        operation: "db.query"
    )
    
    defer { transaction.finish() }
    
    do {
        // Load profile
        let profile = try await fetchProfile(userID: userID)
        print("Profile loaded: \(profile)")
    } catch {
        SentryManager.captureError(error, context: ["user_id": userID])
    }
}
```

---

## 5. Analytics Integration

### 5.1 Firebase Analytics

```swift
import FirebaseAnalytics

// Analytics Manager - Single point of truth
class AnalyticsManager {
    
    static let shared = AnalyticsManager()
    private init() {}
    
    // MARK: - User Properties
    
    func setUserProperty(_ value: String?, forName name: String) {
        Analytics.setUserProperty(value, forName: name)
    }
    
    func setUserID(_ id: String?) {
        Analytics.setUserID(id)
    }
    
    // MARK: - Screen Tracking
    
    func trackScreen(_ screenName: String, screenClass: String? = nil) {
        Analytics.logEvent(
            AnalyticsEventScreenView,
            parameters: [
                AnalyticsParameterScreenName: screenName,
                AnalyticsParameterScreenClass: screenClass ?? screenName
            ]
        )
    }
    
    // MARK: - Custom Events
    
    func log(event: AnalyticsEvent) {
        Analytics.logEvent(event.name, parameters: event.parameters)
    }
}

// Analytics Event Type Safety
enum AnalyticsEvent {
    case appOpen
    case login(method: String)
    case purchase(productID: String, amount: Double, currency: String)
    case buttonTap(buttonName: String, screenName: String)
    case search(query: String, resultCount: Int)
    case error(code: String, message: String)
    
    var name: String {
        switch self {
        case .appOpen: return "app_open"
        case .login: return "login"
        case .purchase: return "purchase"
        case .buttonTap: return "button_tap"
        case .search: return "search"
        case .error: return "app_error"
        }
    }
    
    var parameters: [String: Any] {
        switch self {
        case .appOpen:
            return [:]
        case .login(let method):
            return ["method": method]
        case .purchase(let productID, let amount, let currency):
            return [
                AnalyticsParameterItemID: productID,
                AnalyticsParameterValue: amount,
                AnalyticsParameterCurrency: currency
            ]
        case .buttonTap(let buttonName, let screenName):
            return ["button_name": buttonName, "screen_name": screenName]
        case .search(let query, let resultCount):
            return ["search_term": query, "result_count": resultCount]
        case .error(let code, let message):
            return ["error_code": code, "error_message": message]
        }
    }
}

// SwiftUI View modifier สำหรับ screen tracking
struct AnalyticsScreenModifier: ViewModifier {
    let screenName: String
    
    func body(content: Content) -> some View {
        content
            .onAppear {
                AnalyticsManager.shared.trackScreen(screenName)
            }
    }
}

extension View {
    func trackScreen(_ name: String) -> some View {
        modifier(AnalyticsScreenModifier(screenName: name))
    }
}

// Usage
struct HomeView: View {
    var body: some View {
        VStack {
            Button("Buy Now") {
                AnalyticsManager.shared.log(event: .buttonTap(
                    buttonName: "buy_now",
                    screenName: "home"
                ))
                // Handle purchase
            }
        }
        .trackScreen("Home")
    }
}
```

### 5.2 Mixpanel Integration

```swift
import Mixpanel

class MixpanelManager {
    
    static let shared = MixpanelManager()
    
    func initialize(token: String) {
        Mixpanel.initialize(token: token, trackAutomaticEvents: true)
        Mixpanel.mainInstance().loggingEnabled = false
    }
    
    func track(event: String, properties: [String: MixpanelType]? = nil) {
        Mixpanel.mainInstance().track(event: event, properties: properties)
    }
    
    func identify(userID: String) {
        Mixpanel.mainInstance().identify(distinctId: userID)
    }
    
    func setUserProperties(_ properties: [String: MixpanelType]) {
        Mixpanel.mainInstance().people.set(properties: properties)
    }
    
    func incrementProperty(_ property: String, by value: Double = 1) {
        Mixpanel.mainInstance().people.increment(property: property, by: value)
    }
    
    func reset() {
        Mixpanel.mainInstance().reset()
    }
}
```

---

## 6. A/B Testing

### 6.1 Firebase Remote Config A/B Testing

```swift
import FirebaseRemoteConfig
import FirebaseABTesting

class ABTestingManager {
    
    static let shared = ABTestingManager()
    private let remoteConfig = RemoteConfig.remoteConfig()
    
    // Experiment keys
    enum ExperimentKey: String {
        case onboardingFlow = "onboarding_flow_variant"
        case checkoutButton = "checkout_button_color"
        case homeLayout = "home_layout_variant"
    }
    
    // Variant values
    enum OnboardingVariant: String {
        case control = "control"      // variant เดิม
        case variantA = "variant_a"   // version ใหม่ A
        case variantB = "variant_b"   // version ใหม่ B
    }
    
    func fetchConfig() async {
        do {
            try await remoteConfig.fetchAndActivate()
        } catch {
            print("Remote config fetch failed: \(error)")
        }
    }
    
    func getOnboardingVariant() -> OnboardingVariant {
        let value = remoteConfig.configValue(forKey: ExperimentKey.onboardingFlow.rawValue)
        let stringValue = value.stringValue ?? "control"
        return OnboardingVariant(rawValue: stringValue) ?? .control
    }
    
    func getCheckoutButtonColor() -> String {
        return remoteConfig.configValue(forKey: ExperimentKey.checkoutButton.rawValue)
            .stringValue ?? "blue"
    }
    
    // Track experiment exposure
    func trackExposure(key: ExperimentKey, variant: String) {
        AnalyticsManager.shared.log(event: .buttonTap(
            buttonName: "experiment_exposure",
            screenName: "\(key.rawValue)_\(variant)"
        ))
    }
}

// Usage ใน SwiftUI
struct OnboardingView: View {
    
    @State private var variant: ABTestingManager.OnboardingVariant = .control
    
    var body: some View {
        Group {
            switch variant {
            case .control:
                OriginalOnboardingView()
            case .variantA:
                NewOnboardingViewA()
            case .variantB:
                NewOnboardingViewB()
            }
        }
        .onAppear {
            variant = ABTestingManager.shared.getOnboardingVariant()
            ABTestingManager.shared.trackExposure(
                key: .onboardingFlow, 
                variant: variant.rawValue
            )
        }
    }
}
```

### 6.2 Custom A/B Testing Framework

```swift
// Simple Local A/B Testing
import Foundation
import CryptoKit

class LocalABTesting {
    
    static let shared = LocalABTesting()
    
    // Deterministic assignment based on user ID
    func getVariant(
        experimentID: String,
        userID: String,
        variants: [String],
        weights: [Double]? = nil
    ) -> String {
        guard !variants.isEmpty else { return "" }
        
        // สร้าง deterministic hash จาก experimentID + userID
        let input = "\(experimentID):\(userID)"
        let hashValue = stableHash(input)
        
        // Weighted distribution
        let normalizedWeights = weights ?? Array(repeating: 1.0 / Double(variants.count), 
                                                count: variants.count)
        
        let bucket = Double(hashValue % 100) / 100.0
        var cumulative = 0.0
        
        for (index, weight) in normalizedWeights.enumerated() {
            cumulative += weight
            if bucket < cumulative {
                return variants[index]
            }
        }
        
        return variants.last!
    }
    
    private func stableHash(_ input: String) -> UInt64 {
        let data = Data(input.utf8)
        let hash = SHA256.hash(data: data)
        let bytes = Array(hash)
        return UInt64(bytes[0]) << 56 | UInt64(bytes[1]) << 48 |
               UInt64(bytes[2]) << 40 | UInt64(bytes[3]) << 32 |
               UInt64(bytes[4]) << 24 | UInt64(bytes[5]) << 16 |
               UInt64(bytes[6]) << 8  | UInt64(bytes[7])
    }
}

// Usage
let userID = "user123"
let variant = LocalABTesting.shared.getVariant(
    experimentID: "checkout_button_test",
    userID: userID,
    variants: ["blue", "green", "red"],
    weights: [0.5, 0.25, 0.25]  // 50% blue, 25% green, 25% red
)
print("User gets variant: \(variant)")
```

---

## 7. Feature Flags

### 7.1 Local Feature Flags

```swift
// Feature Flag System
import Foundation

protocol FeatureFlagProvider {
    func isEnabled(_ flag: FeatureFlag) -> Bool
    func value(for flag: FeatureFlag) -> Any?
}

// Feature flag definitions
enum FeatureFlag: String, CaseIterable {
    // UI Features
    case newHomeScreen = "new_home_screen"
    case darkModeV2 = "dark_mode_v2"
    case enhancedSearch = "enhanced_search"
    
    // Business Features
    case premiumSubscription = "premium_subscription"
    case socialSharing = "social_sharing"
    case pushNotifications = "push_notifications"
    
    // Experimental
    case aiRecommendations = "ai_recommendations"
    case videoStreaming = "video_streaming"
    
    var defaultValue: Bool {
        switch self {
        case .newHomeScreen: return false
        case .darkModeV2: return true
        case .enhancedSearch: return true
        case .premiumSubscription: return true
        case .socialSharing: return false
        case .pushNotifications: return true
        case .aiRecommendations: return false
        case .videoStreaming: return false
        }
    }
}

// Feature Flag Manager
@MainActor
class FeatureFlagManager: ObservableObject {
    
    static let shared = FeatureFlagManager()
    
    private var localOverrides: [String: Bool] = [:]
    private var remoteFlags: [String: Bool] = [:]
    
    private init() {
        loadLocalOverrides()
    }
    
    func isEnabled(_ flag: FeatureFlag) -> Bool {
        // Priority: Local Override > Remote > Default
        if let localOverride = localOverrides[flag.rawValue] {
            return localOverride
        }
        if let remoteValue = remoteFlags[flag.rawValue] {
            return remoteValue
        }
        return flag.defaultValue
    }
    
    // สำหรับ debugging ใน development
    func setLocalOverride(_ flag: FeatureFlag, enabled: Bool) {
        localOverrides[flag.rawValue] = enabled
        saveLocalOverrides()
    }
    
    func clearLocalOverride(_ flag: FeatureFlag) {
        localOverrides.removeValue(forKey: flag.rawValue)
        saveLocalOverrides()
    }
    
    func updateFromRemote(_ flags: [String: Bool]) {
        remoteFlags = flags
    }
    
    private func loadLocalOverrides() {
        #if DEBUG
        if let saved = UserDefaults.standard.dictionary(forKey: "feature_flag_overrides") as? [String: Bool] {
            localOverrides = saved
        }
        #endif
    }
    
    private func saveLocalOverrides() {
        #if DEBUG
        UserDefaults.standard.set(localOverrides, forKey: "feature_flag_overrides")
        #endif
    }
}

// Property Wrapper สำหรับ convenience
@propertyWrapper
struct FeatureEnabled {
    let flag: FeatureFlag
    
    var wrappedValue: Bool {
        FeatureFlagManager.shared.isEnabled(flag)
    }
}

// Usage
struct HomeView: View {
    @FeatureEnabled(flag: .newHomeScreen) var showNewHome: Bool
    
    var body: some View {
        if showNewHome {
            NewHomeView()
        } else {
            LegacyHomeView()
        }
    }
}
```

### 7.2 Remote Feature Flags ด้วย Firebase

```swift
import FirebaseRemoteConfig

class RemoteFeatureFlagProvider: FeatureFlagProvider {
    
    private let remoteConfig = RemoteConfig.remoteConfig()
    
    func isEnabled(_ flag: FeatureFlag) -> Bool {
        remoteConfig.configValue(forKey: flag.rawValue).boolValue
    }
    
    func value(for flag: FeatureFlag) -> Any? {
        let configValue = remoteConfig.configValue(forKey: flag.rawValue)
        return configValue.stringValue
    }
    
    func fetchAndActivate() async throws {
        try await remoteConfig.fetchAndActivate()
    }
    
    // Setup default values
    func configureDefaults() {
        var defaults: [String: NSObject] = [:]
        for flag in FeatureFlag.allCases {
            defaults[flag.rawValue] = NSNumber(value: flag.defaultValue)
        }
        remoteConfig.setDefaults(defaults)
    }
}
```

---

## 8. Remote Configuration

### 8.1 Firebase Remote Config

```swift
import FirebaseRemoteConfig

class RemoteConfigManager {
    
    static let shared = RemoteConfigManager()
    
    private let config = RemoteConfig.remoteConfig()
    
    // Config keys
    enum ConfigKey: String {
        case minimumAppVersion = "minimum_app_version"
        case apiBaseURL = "api_base_url"
        case maintenanceMode = "maintenance_mode"
        case maxRetryCount = "max_retry_count"
        case sessionTimeout = "session_timeout_seconds"
        case forcedUpdate = "forced_update"
        case welcomeMessage = "welcome_message"
    }
    
    func initialize() {
        setupDefaults()
        
        let settings = RemoteConfigSettings()
        settings.minimumFetchInterval = 3600  // 1 hour ใน production
        
        #if DEBUG
        settings.minimumFetchInterval = 0  // ใน dev fetch ได้เลย
        #endif
        
        config.configSettings = settings
    }
    
    private func setupDefaults() {
        config.setDefaults([
            ConfigKey.minimumAppVersion.rawValue: "1.0.0" as NSObject,
            ConfigKey.apiBaseURL.rawValue: "https://api.production.com" as NSObject,
            ConfigKey.maintenanceMode.rawValue: false as NSObject,
            ConfigKey.maxRetryCount.rawValue: 3 as NSObject,
            ConfigKey.sessionTimeout.rawValue: 1800 as NSObject,
            ConfigKey.forcedUpdate.rawValue: false as NSObject,
            ConfigKey.welcomeMessage.rawValue: "ยินดีต้อนรับ!" as NSObject
        ])
    }
    
    func fetch() async {
        do {
            try await config.fetchAndActivate()
            print("Remote config fetched and activated")
            checkForForceUpdate()
        } catch {
            print("Remote config fetch failed: \(error)")
        }
    }
    
    // Typed accessors
    var minimumAppVersion: String {
        config[ConfigKey.minimumAppVersion.rawValue].stringValue ?? "1.0.0"
    }
    
    var apiBaseURL: String {
        config[ConfigKey.apiBaseURL.rawValue].stringValue ?? "https://api.production.com"
    }
    
    var isMaintenanceMode: Bool {
        config[ConfigKey.maintenanceMode.rawValue].boolValue
    }
    
    var maxRetryCount: Int {
        Int(config[ConfigKey.maxRetryCount.rawValue].numberValue) ?? 3
    }
    
    var welcomeMessage: String {
        config[ConfigKey.welcomeMessage.rawValue].stringValue ?? "ยินดีต้อนรับ!"
    }
    
    // Force update check
    private func checkForForceUpdate() {
        guard config[ConfigKey.forcedUpdate.rawValue].boolValue else { return }
        
        let minimumVersion = minimumAppVersion
        let currentVersion = Bundle.main.infoDictionary?["CFBundleShortVersionString"] as? String ?? "0"
        
        if currentVersion.compare(minimumVersion, options: .numeric) == .orderedAscending {
            DispatchQueue.main.async {
                NotificationCenter.default.post(
                    name: .forceUpdateRequired,
                    object: nil,
                    userInfo: ["minimum_version": minimumVersion]
                )
            }
        }
    }
}

extension Notification.Name {
    static let forceUpdateRequired = Notification.Name("ForceUpdateRequired")
}
```

---

## 9. Error Tracking

### 9.1 Centralized Error Management

```swift
import Foundation

// Error categories
enum AppError: LocalizedError {
    // Network errors
    case networkUnavailable
    case requestTimeout
    case serverError(statusCode: Int, message: String?)
    case invalidResponse
    
    // Authentication errors
    case unauthorized
    case tokenExpired
    case invalidCredentials
    
    // Data errors
    case decodingFailed(type: String)
    case encodingFailed
    case dataNotFound(resource: String)
    case dataCorrrupted
    
    // Business logic errors
    case insufficientPermission
    case featureUnavailable(feature: String)
    case validationFailed([String: String])
    
    var errorDescription: String? {
        switch self {
        case .networkUnavailable:
            return "ไม่มีการเชื่อมต่ออินเทอร์เน็ต กรุณาตรวจสอบการเชื่อมต่อ"
        case .requestTimeout:
            return "การเชื่อมต่อหมดเวลา กรุณาลองใหม่อีกครั้ง"
        case .serverError(let code, let message):
            return message ?? "เกิดข้อผิดพลาดจากเซิร์ฟเวอร์ (รหัส \(code))"
        case .unauthorized:
            return "กรุณาเข้าสู่ระบบใหม่"
        case .tokenExpired:
            return "Session หมดอายุ กรุณาเข้าสู่ระบบใหม่"
        case .invalidCredentials:
            return "ชื่อผู้ใช้หรือรหัสผ่านไม่ถูกต้อง"
        case .decodingFailed(let type):
            return "ไม่สามารถประมวลผลข้อมูล \(type) ได้"
        case .dataNotFound(let resource):
            return "ไม่พบ \(resource)"
        case .insufficientPermission:
            return "คุณไม่มีสิทธิ์ดำเนินการนี้"
        case .featureUnavailable(let feature):
            return "\(feature) ยังไม่พร้อมให้บริการในขณะนี้"
        case .validationFailed(let errors):
            return errors.values.joined(separator: "\n")
        default:
            return "เกิดข้อผิดพลาดที่ไม่คาดคิด"
        }
    }
    
    var isRecoverable: Bool {
        switch self {
        case .networkUnavailable, .requestTimeout:
            return true
        case .tokenExpired:
            return true
        default:
            return false
        }
    }
    
    var shouldReport: Bool {
        switch self {
        case .networkUnavailable, .requestTimeout, .unauthorized,
             .tokenExpired, .invalidCredentials:
            return false  // User errors - ไม่ต้อง report
        default:
            return true  // Developer errors - ต้อง report
        }
    }
}

// Error Handler
@MainActor
class ErrorHandler {
    
    static let shared = ErrorHandler()
    
    func handle(_ error: Error, context: String? = nil) {
        let appError = classify(error)
        
        // Log to crash reporter
        if appError.shouldReport {
            CrashReportingManager.recordError(error, context: context)
        }
        
        // Show user-facing message
        showAlert(for: appError)
        
        // Handle specific cases
        switch appError {
        case .tokenExpired, .unauthorized:
            handleAuthExpiry()
        case .networkUnavailable:
            showOfflineMode()
        default:
            break
        }
    }
    
    private func classify(_ error: Error) -> AppError {
        if let appError = error as? AppError {
            return appError
        }
        
        // Map system errors
        let nsError = error as NSError
        switch nsError.domain {
        case NSURLErrorDomain:
            switch nsError.code {
            case NSURLErrorNotConnectedToInternet:
                return .networkUnavailable
            case NSURLErrorTimedOut:
                return .requestTimeout
            default:
                return .invalidResponse
            }
        default:
            return .serverError(statusCode: nsError.code, message: nsError.localizedDescription)
        }
    }
    
    private func showAlert(for error: AppError) {
        // Post notification สำหรับ UI layer
        NotificationCenter.default.post(
            name: .showErrorAlert,
            object: nil,
            userInfo: ["error": error]
        )
    }
    
    private func handleAuthExpiry() {
        NotificationCenter.default.post(name: .userSessionExpired, object: nil)
    }
    
    private func showOfflineMode() {
        NotificationCenter.default.post(name: .appWentOffline, object: nil)
    }
}

extension Notification.Name {
    static let showErrorAlert = Notification.Name("ShowErrorAlert")
    static let userSessionExpired = Notification.Name("UserSessionExpired")
    static let appWentOffline = Notification.Name("AppWentOffline")
}
```

---

## 10. Logging Strategy ใน Production

### 10.1 Structured Logging

```swift
import os.log
import Foundation

// Log levels
enum LogLevel: String {
    case debug = "DEBUG"
    case info = "INFO"
    case warning = "WARNING"
    case error = "ERROR"
    case critical = "CRITICAL"
}

// Log categories
enum LogCategory: String {
    case app = "App"
    case network = "Network"
    case auth = "Auth"
    case database = "Database"
    case analytics = "Analytics"
    case ui = "UI"
    case performance = "Performance"
}

// Logger
struct AppLogger {
    
    private let subsystem = Bundle.main.bundleIdentifier ?? "com.app"
    
    // OS Log instances per category
    private func osLog(for category: LogCategory) -> OSLog {
        OSLog(subsystem: subsystem, category: category.rawValue)
    }
    
    func log(
        _ message: String,
        level: LogLevel = .info,
        category: LogCategory = .app,
        metadata: [String: Any]? = nil,
        file: String = #file,
        function: String = #function,
        line: Int = #line
    ) {
        let log = osLog(for: category)
        
        // ใช้ os_log สำหรับ system logging
        let osLogType: OSLogType
        switch level {
        case .debug: osLogType = .debug
        case .info: osLogType = .info
        case .warning: osLogType = .default
        case .error: osLogType = .error
        case .critical: osLogType = .fault
        }
        
        #if DEBUG
        // ใน debug แสดง metadata ด้วย
        let filename = (file as NSString).lastPathComponent
        let metadataStr = metadata.map { "\($0)" } ?? ""
        os_log("%{public}@ [%{public}@:%d] %{public}@", 
               log: log, type: osLogType,
               message, filename, line, metadataStr)
        #else
        // ใน production ส่งแค่ message (ไม่มี file/line สำหรับ security)
        os_log("%{public}@", log: log, type: osLogType, message)
        #endif
        
        // ส่งไป crash reporter สำหรับ level ที่สำคัญ
        if level == .error || level == .critical {
            CrashReportingManager.log("[\(level.rawValue)] \(message)")
        }
    }
}

// Convenience
let logger = AppLogger()

// Global log functions
func logDebug(_ message: String, category: LogCategory = .app) {
    #if DEBUG
    logger.log(message, level: .debug, category: category)
    #endif
}

func logInfo(_ message: String, category: LogCategory = .app) {
    logger.log(message, level: .info, category: category)
}

func logWarning(_ message: String, category: LogCategory = .app) {
    logger.log(message, level: .warning, category: category)
}

func logError(_ message: String, category: LogCategory = .app) {
    logger.log(message, level: .error, category: category)
}

// Usage
class AuthService {
    
    func login(email: String, password: String) async throws {
        logInfo("Login attempt", category: .auth)
        
        do {
            let token = try await performLogin(email: email, password: password)
            logInfo("Login successful", category: .auth)
            saveToken(token)
        } catch {
            logError("Login failed: \(error.localizedDescription)", category: .auth)
            throw error
        }
    }
    
    private func performLogin(email: String, password: String) async throws -> String {
        return "token"
    }
    
    private func saveToken(_ token: String) {}
}
```

---

## 11. Privacy Compliance

### 11.1 GDPR Compliance

```swift
// GDPR Consent Manager
import Foundation
import UIKit

class GDPRConsentManager {
    
    static let shared = GDPRConsentManager()
    
    // Consent types
    struct ConsentState {
        var analyticsEnabled: Bool = false
        var marketingEnabled: Bool = false
        var functionalEnabled: Bool = true  // จำเป็นสำหรับ app
        var thirdPartyEnabled: Bool = false
        var consentDate: Date?
        var consentVersion: String?
    }
    
    private(set) var currentConsent: ConsentState = ConsentState()
    
    // Keys สำหรับ UserDefaults
    private enum Keys {
        static let consentState = "gdpr_consent_state"
        static let consentVersion = "gdpr_consent_version"
    }
    
    // Current consent policy version
    private let currentVersion = "2024.1"
    
    func loadSavedConsent() {
        if let data = UserDefaults.standard.data(forKey: Keys.consentState),
           let saved = try? JSONDecoder().decode(ConsentState.self, from: data) {
            currentConsent = saved
        }
    }
    
    func needsConsentUpdate() -> Bool {
        return currentConsent.consentVersion != currentVersion ||
               currentConsent.consentDate == nil
    }
    
    func updateConsent(
        analytics: Bool,
        marketing: Bool,
        thirdParty: Bool
    ) {
        currentConsent.analyticsEnabled = analytics
        currentConsent.marketingEnabled = marketing
        currentConsent.thirdPartyEnabled = thirdParty
        currentConsent.consentDate = Date()
        currentConsent.consentVersion = currentVersion
        
        saveConsent()
        applyConsent()
    }
    
    func withdrawAllConsent() {
        currentConsent.analyticsEnabled = false
        currentConsent.marketingEnabled = false
        currentConsent.thirdPartyEnabled = false
        currentConsent.consentDate = Date()
        
        saveConsent()
        applyConsent()
    }
    
    // Delete user data (Right to Erasure)
    func deleteUserData(completion: @escaping (Bool) -> Void) {
        // 1. Delete from local storage
        clearLocalData()
        
        // 2. Request deletion from server
        Task {
            do {
                try await requestServerDataDeletion()
                DispatchQueue.main.async {
                    completion(true)
                }
            } catch {
                DispatchQueue.main.async {
                    completion(false)
                }
            }
        }
    }
    
    private func saveConsent() {
        if let data = try? JSONEncoder().encode(currentConsent) {
            UserDefaults.standard.set(data, forKey: Keys.consentState)
        }
    }
    
    private func applyConsent() {
        // Enable/disable analytics
        if currentConsent.analyticsEnabled {
            AnalyticsManager.shared.enable()
        } else {
            AnalyticsManager.shared.disable()
        }
        
        // Notify other services
        NotificationCenter.default.post(
            name: .consentUpdated,
            object: nil,
            userInfo: ["consent": currentConsent]
        )
    }
    
    private func clearLocalData() {
        UserDefaults.standard.removeObject(forKey: "user_id")
        // Clear other PII data
    }
    
    private func requestServerDataDeletion() async throws {
        // API call to request data deletion
    }
}

// GDPR Consent View
struct ConsentView: View {
    @State private var analyticsEnabled = false
    @State private var marketingEnabled = false
    @State private var thirdPartyEnabled = false
    let onComplete: () -> Void
    
    var body: some View {
        NavigationView {
            Form {
                Section("เราใช้ข้อมูลของคุณอย่างไร") {
                    Text("เราเคารพความเป็นส่วนตัวของคุณ กรุณาเลือกว่าคุณยินยอมให้เราใช้ข้อมูลของคุณในรูปแบบใด")
                        .font(.body)
                        .foregroundColor(.secondary)
                }
                
                Section("การตั้งค่าความเป็นส่วนตัว") {
                    Toggle("Analytics", isOn: $analyticsEnabled)
                    Text("ช่วยให้เราเข้าใจวิธีที่คุณใช้ App เพื่อปรับปรุงประสบการณ์")
                        .font(.caption)
                        .foregroundColor(.secondary)
                    
                    Toggle("การตลาด", isOn: $marketingEnabled)
                    Text("รับข้อเสนอและโปรโมชั่นที่เกี่ยวข้องกับคุณ")
                        .font(.caption)
                        .foregroundColor(.secondary)
                    
                    Toggle("บริษัทภายนอก", isOn: $thirdPartyEnabled)
                    Text("อนุญาตให้พันธมิตรของเราใช้ข้อมูลเพื่อปรับแต่งโฆษณา")
                        .font(.caption)
                        .foregroundColor(.secondary)
                }
                
                Section {
                    Button("ยอมรับที่เลือก") {
                        GDPRConsentManager.shared.updateConsent(
                            analytics: analyticsEnabled,
                            marketing: marketingEnabled,
                            thirdParty: thirdPartyEnabled
                        )
                        onComplete()
                    }
                    .frame(maxWidth: .infinity)
                    .buttonStyle(.borderedProminent)
                    
                    Button("ปฏิเสธทั้งหมด") {
                        GDPRConsentManager.shared.withdrawAllConsent()
                        onComplete()
                    }
                    .frame(maxWidth: .infinity)
                    .foregroundColor(.secondary)
                }
            }
            .navigationTitle("ความเป็นส่วนตัว")
        }
    }
}

extension Notification.Name {
    static let consentUpdated = Notification.Name("ConsentUpdated")
}

// Extend ConsentState to be Codable
extension GDPRConsentManager.ConsentState: Codable {}
```

### 11.2 CCPA/PDPA Compliance

```swift
// CCPA (California Consumer Privacy Act) Manager
class CCPAManager {
    
    static let shared = CCPAManager()
    
    private let defaults = UserDefaults.standard
    private let optOutKey = "ccpa_sale_opt_out"
    
    // ผู้ใช้ opt out จากการขายข้อมูล
    var hasSaleOptOut: Bool {
        get { defaults.bool(forKey: optOutKey) }
        set { defaults.set(newValue, forKey: optOutKey) }
    }
    
    // "Do Not Sell My Personal Information" link
    func handleDoNotSell() {
        hasSaleOptOut = true
        
        // Notify all data brokers/partners
        notifyDataPartners(optOut: true)
        
        AnalyticsManager.shared.log(event: .buttonTap(
            buttonName: "do_not_sell",
            screenName: "privacy_settings"
        ))
    }
    
    // California residents rights
    func requestDataAccess() async throws -> [String: Any] {
        // Return all data held about user
        return [
            "account": [:],
            "activity": [],
            "purchases": []
        ]
    }
    
    func requestDataDeletion() async throws {
        // Delete all user data
    }
    
    func requestDataPortability() async throws -> Data {
        // Export user data in machine-readable format
        return Data()
    }
    
    private func notifyDataPartners(optOut: Bool) {
        // Notify advertising partners
    }
}

// PDPA (Thailand Personal Data Protection Act) Manager
class PDPAManager {
    
    static let shared = PDPAManager()
    
    struct PDPAConsent: Codable {
        var consentGiven: Bool
        var consentDate: Date?
        var purpose: [String]
        var dataCategories: [String]
        var retentionPeriod: String
        
        static var empty: PDPAConsent {
            PDPAConsent(
                consentGiven: false,
                consentDate: nil,
                purpose: [],
                dataCategories: [],
                retentionPeriod: ""
            )
        }
    }
    
    func showConsentForm(for purposes: [String], completion: @escaping (Bool) -> Void) {
        // แสดง consent form ตาม PDPA requirements
        // ต้องมี: วัตถุประสงค์การเก็บข้อมูล, ประเภทข้อมูล, ระยะเวลาเก็บ
    }
    
    func recordConsent(purposes: [String], dataCategories: [String]) {
        let consent = PDPAConsent(
            consentGiven: true,
            consentDate: Date(),
            purpose: purposes,
            dataCategories: dataCategories,
            retentionPeriod: "2 ปี"
        )
        
        if let data = try? JSONEncoder().encode(consent) {
            UserDefaults.standard.set(data, forKey: "pdpa_consent")
        }
    }
}
```

---

## 12. App Privacy Report

### 12.1 Privacy Manifest

```xml
<!-- PrivacyInfo.xcprivacy -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <!-- Privacy Nutrition Labels -->
    <key>NSPrivacyTracking</key>
    <false/>
    
    <key>NSPrivacyTrackingDomains</key>
    <array/>
    
    <!-- Accessed API types -->
    <key>NSPrivacyAccessedAPITypes</key>
    <array>
        <dict>
            <key>NSPrivacyAccessedAPIType</key>
            <string>NSPrivacyAccessedAPICategoryUserDefaults</string>
            <key>NSPrivacyAccessedAPITypeReasons</key>
            <array>
                <string>CA92.1</string>
            </array>
        </dict>
        <dict>
            <key>NSPrivacyAccessedAPIType</key>
            <string>NSPrivacyAccessedAPICategoryFileTimestamp</string>
            <key>NSPrivacyAccessedAPITypeReasons</key>
            <array>
                <string>C617.1</string>
            </array>
        </dict>
    </array>
    
    <!-- Collected data types -->
    <key>NSPrivacyCollectedDataTypes</key>
    <array>
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
            </array>
        </dict>
    </array>
</dict>
</plist>
```

```swift
// Privacy Report Viewer
class AppPrivacyReportReader {
    
    // อ่าน App Privacy Report (iOS 15.2+)
    func getPrivacyReport() async -> [PrivacyReportEntry]? {
        guard let reportURL = getPrivacyReportURL() else {
            return nil
        }
        
        do {
            let data = try Data(contentsOf: reportURL)
            let report = try JSONDecoder().decode([PrivacyReportEntry].self, from: data)
            return report
        } catch {
            print("Failed to read privacy report: \(error)")
            return nil
        }
    }
    
    private func getPrivacyReportURL() -> URL? {
        // Privacy report location
        let documentsPath = FileManager.default.urls(
            for: .documentDirectory,
            in: .userDomainMask
        ).first
        return documentsPath?.appendingPathComponent("app-privacy-report.ndjson")
    }
}

struct PrivacyReportEntry: Codable {
    let type: String
    let timestamp: String
    let bundleID: String?
    let domain: String?
    let accessor: AccessorInfo?
    
    struct AccessorInfo: Codable {
        let identifier: String
        let category: String
    }
}
```

---

## 13. ATT (App Tracking Transparency)

```swift
import AppTrackingTransparency
import AdSupport

class ATTManager {
    
    static let shared = ATTManager()
    
    // Request tracking permission
    func requestTrackingPermission(completion: @escaping (ATTrackingManager.AuthorizationStatus) -> Void) {
        
        // ต้องรอให้ App โหลดเสร็จก่อน
        if #available(iOS 14, *) {
            ATTrackingManager.requestTrackingAuthorization { status in
                DispatchQueue.main.async {
                    self.handleTrackingStatus(status)
                    completion(status)
                }
            }
        } else {
            completion(.authorized)  // iOS 13 ไม่ต้องขอ
        }
    }
    
    var trackingStatus: ATTrackingManager.AuthorizationStatus {
        if #available(iOS 14, *) {
            return ATTrackingManager.trackingAuthorizationStatus
        }
        return .authorized
    }
    
    var advertisingID: String? {
        guard trackingStatus == .authorized else { return nil }
        return ASIdentifierManager.shared().advertisingIdentifier.uuidString
    }
    
    var isTrackingEnabled: Bool {
        trackingStatus == .authorized
    }
    
    private func handleTrackingStatus(_ status: ATTrackingManager.AuthorizationStatus) {
        switch status {
        case .authorized:
            print("Tracking authorized - enable personalized ads")
            enablePersonalizedTracking()
        case .denied:
            print("Tracking denied - use contextual ads only")
            disableTracking()
        case .restricted:
            print("Tracking restricted by parental controls")
            disableTracking()
        case .notDetermined:
            print("Not yet asked")
        @unknown default:
            break
        }
    }
    
    private func enablePersonalizedTracking() {
        // Configure advertising SDK with IDFA
        if let idfa = advertisingID {
            AnalyticsManager.shared.setUserProperty(idfa, forName: "advertising_id")
        }
    }
    
    private func disableTracking() {
        // ใช้ anonymous tracking แทน
        AnalyticsManager.shared.setUserProperty(nil, forName: "advertising_id")
    }
}

// Info.plist entry ที่จำเป็น:
// NSUserTrackingUsageDescription
// "เราใช้ข้อมูลนี้เพื่อแสดงโฆษณาที่เกี่ยวข้องกับคุณและวัดผลประสิทธิภาพโฆษณา"

// SwiftUI Usage
struct ATTPermissionView: View {
    @State private var showingATTDialog = false
    @State private var attStatus: ATTrackingManager.AuthorizationStatus = .notDetermined
    
    var body: some View {
        VStack(spacing: 20) {
            Image(systemName: "hand.raised.fill")
                .font(.system(size: 60))
                .foregroundColor(.blue)
            
            Text("ช่วยให้เราปรับปรุง App ได้")
                .font(.title2)
                .bold()
            
            Text("เราจะขออนุญาตติดตามเพื่อแสดงโฆษณาที่เกี่ยวข้องกับคุณ คุณสามารถเปลี่ยนแปลงได้ในการตั้งค่า")
                .multilineTextAlignment(.center)
                .foregroundColor(.secondary)
            
            Button("ดำเนินการต่อ") {
                ATTManager.shared.requestTrackingPermission { status in
                    attStatus = status
                }
            }
            .buttonStyle(.borderedProminent)
        }
        .padding()
    }
}
```

---

## 14. App Clips

### 14.1 สร้าง App Clip

```swift
// App Clip Target - AppClipApp.swift
import SwiftUI
import AppClip

@main
struct MyAppClip: App {
    
    var body: some Scene {
        WindowGroup {
            AppClipContentView()
                .onContinueUserActivity(NSUserActivityTypeBrowsingWeb) { activity in
                    handleInvocation(activity)
                }
        }
    }
    
    func handleInvocation(_ activity: NSUserActivity) {
        guard let incomingURL = activity.webpageURL else { return }
        
        // Parse URL และแสดง content ที่เกี่ยวข้อง
        let components = URLComponents(url: incomingURL, resolvingAgainstBaseURL: true)
        
        if let productID = components?.queryItems?.first(where: { $0.name == "product_id" })?.value {
            print("Show product: \(productID)")
        }
    }
}

// App Clip Content View
struct AppClipContentView: View {
    @State private var showingInstallPrompt = false
    
    var body: some View {
        VStack {
            // Content ที่ต้องการแสดง
            ProductPreviewView()
            
            // ชักชวนให้ download full app
            Button("Download Full App") {
                showingInstallPrompt = true
            }
            .buttonStyle(.borderedProminent)
        }
        .appStoreOverlay(isPresented: $showingInstallPrompt) {
            SKOverlay.AppClipConfiguration(position: .bottom)
        }
    }
}

// Verification - ตรวจสอบ URL
class AppClipVerifier {
    
    static func verify(url: URL) -> Bool {
        // ตรวจสอบว่า URL valid
        let validDomains = ["example.com", "www.example.com"]
        return validDomains.contains(url.host ?? "")
    }
    
    static func parseProductID(from url: URL) -> String? {
        URLComponents(url: url, resolvingAgainstBaseURL: true)?
            .queryItems?
            .first(where: { $0.name == "id" })?
            .value
    }
}
```

---

## 15. Universal Links และ App Links

### 15.1 Universal Links Setup

```swift
// apple-app-site-association file (วางบน server)
/*
{
    "applinks": {
        "apps": [],
        "details": [
            {
                "appIDs": ["TEAMID.com.yourcompany.app"],
                "components": [
                    {
                        "/": "/products/*",
                        "comment": "Product pages"
                    },
                    {
                        "/": "/users/*/profile",
                        "comment": "User profiles"
                    },
                    {
                        "/": "/articles/*",
                        "comment": "Article pages",
                        "exclude": false
                    }
                ]
            }
        ]
    }
}
*/

// Info.plist - Associated Domains
// com.apple.developer.associated-domains = ["applinks:example.com"]

// Handle Universal Links
class UniversalLinkHandler {
    
    enum DeepLink {
        case product(id: String)
        case userProfile(username: String)
        case article(slug: String)
        case unknown
    }
    
    static func handle(_ url: URL) -> DeepLink {
        let pathComponents = url.pathComponents
        
        // /products/{id}
        if pathComponents.count >= 3 && pathComponents[1] == "products" {
            return .product(id: pathComponents[2])
        }
        
        // /users/{username}/profile
        if pathComponents.count >= 4 && pathComponents[1] == "users" && pathComponents[3] == "profile" {
            return .userProfile(username: pathComponents[2])
        }
        
        // /articles/{slug}
        if pathComponents.count >= 3 && pathComponents[1] == "articles" {
            return .article(slug: pathComponents[2])
        }
        
        return .unknown
    }
}

// SwiftUI App - Handle Universal Links
@main
struct MyApp: App {
    
    var body: some Scene {
        WindowGroup {
            RootView()
                .onOpenURL { url in
                    handleURL(url)
                }
        }
    }
    
    func handleURL(_ url: URL) {
        let deepLink = UniversalLinkHandler.handle(url)
        
        switch deepLink {
        case .product(let id):
            NavigationManager.shared.navigate(to: .product(id: id))
        case .userProfile(let username):
            NavigationManager.shared.navigate(to: .userProfile(username: username))
        case .article(let slug):
            NavigationManager.shared.navigate(to: .article(slug: slug))
        case .unknown:
            print("Unknown deep link: \(url)")
        }
    }
}

// Navigation Manager
@MainActor
class NavigationManager: ObservableObject {
    
    static let shared = NavigationManager()
    
    enum Destination {
        case product(id: String)
        case userProfile(username: String)
        case article(slug: String)
    }
    
    @Published var pendingNavigation: Destination?
    
    func navigate(to destination: Destination) {
        pendingNavigation = destination
    }
}
```

---

## 16. Handoff ระหว่างอุปกรณ์

```swift
import UIKit

// Handoff - ส่งต่อ activity ระหว่าง iPhone, iPad, Mac
class HandoffManager {
    
    // Activity types
    enum ActivityType: String {
        case viewingArticle = "com.yourapp.viewingArticle"
        case editingDocument = "com.yourapp.editingDocument"
        case browsing = "com.yourapp.browsing"
    }
    
    // สร้าง activity สำหรับ handoff
    func createActivity(type: ActivityType, userInfo: [String: Any]) -> NSUserActivity {
        let activity = NSUserActivity(activityType: type.rawValue)
        activity.title = titleFor(type: type)
        activity.isEligibleForHandoff = true
        activity.isEligibleForSearch = true
        activity.isEligibleForPublicIndexing = false
        activity.userInfo = userInfo
        activity.becomeCurrent()
        return activity
    }
    
    // Receive handoff
    func handleHandoff(_ activity: NSUserActivity) -> Bool {
        guard let typeString = ActivityType(rawValue: activity.activityType),
              let userInfo = activity.userInfo else {
            return false
        }
        
        switch typeString {
        case .viewingArticle:
            if let articleID = userInfo["articleID"] as? String {
                navigateToArticle(id: articleID)
                return true
            }
        case .editingDocument:
            if let documentID = userInfo["documentID"] as? String {
                openDocument(id: documentID)
                return true
            }
        case .browsing:
            if let urlString = userInfo["url"] as? String,
               let url = URL(string: urlString) {
                openURL(url)
                return true
            }
        }
        
        return false
    }
    
    private func titleFor(type: ActivityType) -> String {
        switch type {
        case .viewingArticle: return "กำลังอ่านบทความ"
        case .editingDocument: return "กำลังแก้ไขเอกสาร"
        case .browsing: return "กำลังท่องเว็บ"
        }
    }
    
    private func navigateToArticle(id: String) { }
    private func openDocument(id: String) { }
    private func openURL(_ url: URL) { }
}

// SwiftUI Implementation
struct ArticleView: View {
    let article: Article
    @State private var userActivity: NSUserActivity?
    
    var body: some View {
        ScrollView {
            Text(article.content)
        }
        .navigationTitle(article.title)
        .userActivity("com.yourapp.viewingArticle") { activity in
            activity.title = article.title
            activity.userInfo = ["articleID": article.id]
            activity.isEligibleForHandoff = true
        }
    }
}

struct Article {
    let id: String
    let title: String
    let content: String
}
```

---

## 17. Siri Integration (App Intents)

```swift
import AppIntents
import Foundation

// App Intent สำหรับ Siri
struct OrderCoffeeIntent: AppIntent {
    
    static var title: LocalizedStringResource = "สั่งกาแฟ"
    static var description = IntentDescription("สั่งกาแฟโปรดของคุณ")
    
    // Parameters
    @Parameter(title: "ประเภทกาแฟ")
    var coffeeType: CoffeeTypeEntity
    
    @Parameter(title: "ขนาด", default: .medium)
    var size: CoffeeSize
    
    @Parameter(title: "จำนวน", default: 1)
    var quantity: Int
    
    // Perform the intent
    func perform() async throws -> some IntentResult & ProvidesDialog {
        // สั่งกาแฟ
        let order = try await CoffeeService.shared.placeOrder(
            type: coffeeType.id,
            size: size,
            quantity: quantity
        )
        
        return .result(dialog: "สั่ง\(coffeeType.displayName)ขนาด\(size.rawValue) \(quantity)แก้วเรียบร้อยแล้ว หมายเลขคำสั่ง \(order.id)")
    }
}

// Coffee Type Entity
struct CoffeeTypeEntity: AppEntity {
    static var typeDisplayRepresentation = TypeDisplayRepresentation(name: "ประเภทกาแฟ")
    static var defaultQuery = CoffeeTypeQuery()
    
    var id: String
    var displayName: String
    
    var displayRepresentation: DisplayRepresentation {
        DisplayRepresentation(title: LocalizedStringResource(stringLiteral: displayName))
    }
}

struct CoffeeTypeQuery: EntityQuery {
    func entities(for identifiers: [String]) async throws -> [CoffeeTypeEntity] {
        return identifiers.compactMap { id in
            allCoffeeTypes.first { $0.id == id }
        }
    }
    
    func suggestedEntities() async throws -> [CoffeeTypeEntity] {
        return allCoffeeTypes
    }
    
    private var allCoffeeTypes: [CoffeeTypeEntity] {
        [
            CoffeeTypeEntity(id: "americano", displayName: "อเมริกาโน่"),
            CoffeeTypeEntity(id: "latte", displayName: "ลาเต้"),
            CoffeeTypeEntity(id: "cappuccino", displayName: "คาปูชิโน่"),
            CoffeeTypeEntity(id: "espresso", displayName: "เอสเปรสโซ่")
        ]
    }
}

enum CoffeeSize: String, AppEnum {
    case small = "เล็ก"
    case medium = "กลาง"
    case large = "ใหญ่"
    
    static var typeDisplayRepresentation = TypeDisplayRepresentation(name: "ขนาด")
    static var caseDisplayRepresentations: [CoffeeSize: DisplayRepresentation] = [
        .small: "เล็ก (S)",
        .medium: "กลาง (M)",
        .large: "ใหญ่ (L)"
    ]
}

// App Shortcuts Provider
struct CoffeeAppShortcuts: AppShortcutsProvider {
    
    static var appShortcuts: [AppShortcut] {
        AppShortcut(
            intent: OrderCoffeeIntent(),
            phrases: [
                "สั่งกาแฟผ่าน \(.applicationName)",
                "สั่ง\(.applicationName)",
                "เปิด\(.applicationName)สั่งกาแฟ"
            ],
            shortTitle: "สั่งกาแฟ",
            systemImageName: "cup.and.saucer.fill"
        )
    }
}

// Service stub
class CoffeeService {
    static let shared = CoffeeService()
    
    struct Order {
        let id: String
    }
    
    func placeOrder(type: String, size: CoffeeSize, quantity: Int) async throws -> Order {
        return Order(id: "ORD\(Int.random(in: 1000...9999))")
    }
}
```

---

## 18. Shortcuts Integration

```swift
import AppIntents

// Shortcut Actions
struct GetWeatherIntent: AppIntent {
    
    static var title: LocalizedStringResource = "ดูสภาพอากาศ"
    
    @Parameter(title: "เมือง")
    var city: String
    
    func perform() async throws -> some IntentResult & ReturnsValue<String> {
        let weather = try await WeatherService.fetch(for: city)
        return .result(value: "\(city): \(weather.temperature)°C, \(weather.condition)")
    }
}

// Widget Extension Integration
import WidgetKit

struct WeatherEntry: TimelineEntry {
    let date: Date
    let temperature: Double
    let condition: String
    let city: String
}

struct WeatherTimelineProvider: TimelineProvider {
    
    func placeholder(in context: Context) -> WeatherEntry {
        WeatherEntry(date: Date(), temperature: 30, condition: "แดด", city: "กรุงเทพ")
    }
    
    func getSnapshot(in context: Context, completion: @escaping (WeatherEntry) -> Void) {
        let entry = WeatherEntry(date: Date(), temperature: 30, condition: "แดด", city: "กรุงเทพ")
        completion(entry)
    }
    
    func getTimeline(in context: Context, completion: @escaping (Timeline<WeatherEntry>) -> Void) {
        Task {
            let entry = WeatherEntry(date: Date(), temperature: 30, condition: "แดด", city: "กรุงเทพ")
            let nextUpdate = Calendar.current.date(byAdding: .minute, value: 30, to: Date())!
            let timeline = Timeline(entries: [entry], policy: .after(nextUpdate))
            completion(timeline)
        }
    }
}

struct WeatherWidget: Widget {
    let kind = "WeatherWidget"
    
    var body: some WidgetConfiguration {
        StaticConfiguration(kind: kind, provider: WeatherTimelineProvider()) { entry in
            WeatherWidgetView(entry: entry)
        }
        .configurationDisplayName("สภาพอากาศ")
        .description("แสดงสภาพอากาศปัจจุบัน")
        .supportedFamilies([.systemSmall, .systemMedium])
    }
}

struct WeatherWidgetView: View {
    let entry: WeatherEntry
    
    var body: some View {
        VStack {
            Text(entry.city)
                .font(.caption)
            Text("\(Int(entry.temperature))°C")
                .font(.title)
                .bold()
            Text(entry.condition)
                .font(.caption2)
        }
        .containerBackground(for: .widget) {
            Color.blue.opacity(0.8)
        }
    }
}

// WeatherService stub
class WeatherService {
    struct WeatherData {
        let temperature: Double
        let condition: String
    }
    
    static func fetch(for city: String) async throws -> WeatherData {
        return WeatherData(temperature: 30, condition: "แดด")
    }
}
```

---

## 19. Spotlight Search Integration

```swift
import CoreSpotlight
import MobileCoreServices

class SpotlightManager {
    
    static let shared = SpotlightManager()
    
    // Index item
    func indexItem(_ item: SearchableItem) {
        let attributeSet = CSSearchableItemAttributeSet(contentType: .text)
        attributeSet.title = item.title
        attributeSet.contentDescription = item.description
        attributeSet.keywords = item.keywords
        
        if let thumbnailData = item.thumbnailData {
            attributeSet.thumbnailData = thumbnailData
        }
        
        let searchableItem = CSSearchableItem(
            uniqueIdentifier: item.id,
            domainIdentifier: "com.yourapp.\(item.type)",
            attributeSet: attributeSet
        )
        
        searchableItem.expirationDate = Calendar.current.date(
            byAdding: .month, value: 1, to: Date()
        )
        
        CSSearchableIndex.default().indexSearchableItems([searchableItem]) { error in
            if let error = error {
                print("Indexing error: \(error)")
            }
        }
    }
    
    // Index multiple items
    func indexItems(_ items: [SearchableItem]) {
        let searchableItems = items.map { item -> CSSearchableItem in
            let attributeSet = CSSearchableItemAttributeSet(contentType: .text)
            attributeSet.title = item.title
            attributeSet.contentDescription = item.description
            attributeSet.keywords = item.keywords
            
            return CSSearchableItem(
                uniqueIdentifier: item.id,
                domainIdentifier: "com.yourapp.\(item.type)",
                attributeSet: attributeSet
            )
        }
        
        CSSearchableIndex.default().indexSearchableItems(searchableItems) { error in
            if let error = error {
                print("Batch indexing error: \(error)")
            }
        }
    }
    
    // Delete specific items
    func deleteItems(withIDs ids: [String]) {
        CSSearchableIndex.default().deleteSearchableItems(withIdentifiers: ids) { error in
            if let error = error {
                print("Delete error: \(error)")
            }
        }
    }
    
    // Delete all items
    func deleteAllItems() {
        CSSearchableIndex.default().deleteAllSearchableItems { error in
            if let error = error {
                print("Delete all error: \(error)")
            }
        }
    }
    
    // Handle spotlight search result
    func handleActivity(_ activity: NSUserActivity) -> String? {
        guard activity.activityType == CSSearchableItemActionType,
              let identifier = activity.userInfo?[CSSearchableItemActivityIdentifier] as? String
        else { return nil }
        
        return identifier
    }
}

struct SearchableItem {
    let id: String
    let title: String
    let description: String
    let type: String
    let keywords: [String]
    let thumbnailData: Data?
}
```

---

## 20. Share Sheet Integration

```swift
import SwiftUI
import UIKit

// Share Sheet
struct ShareSheet: UIViewControllerRepresentable {
    
    let items: [Any]
    let activities: [UIActivity]?
    
    func makeUIViewController(context: Context) -> UIActivityViewController {
        UIActivityViewController(activityItems: items, applicationActivities: activities)
    }
    
    func updateUIViewController(_ uiViewController: UIActivityViewController, context: Context) {}
}

// Custom Activity
class SaveToFavoritesActivity: UIActivity {
    
    private var item: Any?
    
    override var activityTitle: String? { "บันทึกในรายการโปรด" }
    
    override var activityImage: UIImage? {
        UIImage(systemName: "heart.fill")
    }
    
    override class var activityCategory: UIActivity.Category { .action }
    
    override func canPerform(withActivityItems activityItems: [Any]) -> Bool {
        return activityItems.contains { $0 is String || $0 is URL }
    }
    
    override func prepare(withActivityItems activityItems: [Any]) {
        item = activityItems.first
    }
    
    override func perform() {
        defer { activityDidFinish(true) }
        
        if let item = item {
            FavoritesManager.shared.add(item)
        }
    }
}

// SwiftUI Usage
struct ProductView: View {
    let product: Product
    @State private var showingShare = false
    
    var body: some View {
        VStack {
            Text(product.name)
                .font(.title)
            
            Button {
                showingShare = true
            } label: {
                Label("แชร์", systemImage: "square.and.arrow.up")
            }
        }
        .sheet(isPresented: $showingShare) {
            ShareSheet(
                items: [
                    product.name,
                    URL(string: "https://example.com/products/\(product.id)")!
                ],
                activities: [SaveToFavoritesActivity()]
            )
        }
    }
}

struct Product {
    let id: String
    let name: String
}

class FavoritesManager {
    static let shared = FavoritesManager()
    func add(_ item: Any) {}
}
```

---

## 21. iCloud Drive Integration

```swift
import Foundation
import UniformTypeIdentifiers

class iCloudDriveManager {
    
    static let shared = iCloudDriveManager()
    
    // ตรวจสอบว่า iCloud พร้อมใช้งาน
    var isAvailable: Bool {
        FileManager.default.ubiquityIdentityToken != nil
    }
    
    // iCloud Documents directory
    var iCloudDocumentsURL: URL? {
        FileManager.default.url(forUbiquityContainerIdentifier: nil)?
            .appendingPathComponent("Documents")
    }
    
    // Save file to iCloud
    func saveFile(data: Data, fileName: String) async throws -> URL {
        guard let baseURL = iCloudDocumentsURL else {
            throw AppError.featureUnavailable(feature: "iCloud Drive")
        }
        
        // สร้าง Documents directory ถ้ายังไม่มี
        try FileManager.default.createDirectory(
            at: baseURL,
            withIntermediateDirectories: true
        )
        
        let fileURL = baseURL.appendingPathComponent(fileName)
        try data.write(to: fileURL, options: .atomic)
        
        return fileURL
    }
    
    // Read file from iCloud
    func readFile(named fileName: String) async throws -> Data {
        guard let fileURL = iCloudDocumentsURL?.appendingPathComponent(fileName) else {
            throw AppError.dataNotFound(resource: fileName)
        }
        
        // Download ถ้ายังไม่มีใน device
        if !FileManager.default.fileExists(atPath: fileURL.path) {
            try await downloadFile(at: fileURL)
        }
        
        return try Data(contentsOf: fileURL)
    }
    
    // List files
    func listFiles() throws -> [URL] {
        guard let baseURL = iCloudDocumentsURL else { return [] }
        
        return try FileManager.default.contentsOfDirectory(
            at: baseURL,
            includingPropertiesForKeys: [.nameKey, .fileSizeKey, .creationDateKey],
            options: .skipsHiddenFiles
        )
    }
    
    // Delete file
    func deleteFile(named fileName: String) async throws {
        guard let fileURL = iCloudDocumentsURL?.appendingPathComponent(fileName) else {
            return
        }
        
        try FileManager.default.removeItem(at: fileURL)
    }
    
    // Monitor file changes
    func startMonitoring(directory: URL, onChange: @escaping ([URL]) -> Void) {
        let query = NSMetadataQuery()
        query.searchScopes = [NSMetadataQueryUbiquitousDocumentsScope]
        query.predicate = NSPredicate(format: "%K LIKE '*.json'",
                                      NSMetadataItemFSNameKey)
        
        NotificationCenter.default.addObserver(
            forName: .NSMetadataQueryDidUpdate,
            object: query,
            queue: .main
        ) { notification in
            query.disableUpdates()
            defer { query.enableUpdates() }
            
            var changedURLs: [URL] = []
            for item in query.results {
                if let metadataItem = item as? NSMetadataItem,
                   let url = metadataItem.value(forAttribute: NSMetadataItemURLKey) as? URL {
                    changedURLs.append(url)
                }
            }
            
            onChange(changedURLs)
        }
        
        query.start()
    }
    
    private func downloadFile(at url: URL) async throws {
        try FileManager.default.startDownloadingUbiquitousItem(at: url)
        
        // รอให้ download เสร็จ
        while true {
            let values = try url.resourceValues(forKeys: [.ubiquitousItemDownloadingStatusKey])
            if values.ubiquitousItemDownloadingStatus == .current {
                break
            }
            try await Task.sleep(nanoseconds: 100_000_000)  // 0.1 seconds
        }
    }
}
```

---

## 22. แบบฝึกหัดพร้อมเฉลย

### แบบฝึกหัดที่ 1: สร้าง Production-Ready App

**โจทย์:** สร้าง App ที่มีระบบ crash reporting, analytics, และ feature flags ครบถ้วน

```swift
// เฉลย: Production App Setup

// 1. App Configuration
struct AppConfiguration {
    let environment: Environment
    let apiBaseURL: String
    let analyticsEnabled: Bool
    let crashReportingEnabled: Bool
    
    enum Environment: String {
        case development = "dev"
        case staging = "staging"
        case production = "prod"
        
        static var current: Environment {
            #if DEBUG
            return .development
            #else
            return ProcessInfo.processInfo.environment["APP_ENV"].flatMap(Environment.init) ?? .production
            #endif
        }
    }
    
    static var current: AppConfiguration {
        switch Environment.current {
        case .development:
            return AppConfiguration(
                environment: .development,
                apiBaseURL: "https://dev-api.example.com",
                analyticsEnabled: false,
                crashReportingEnabled: false
            )
        case .staging:
            return AppConfiguration(
                environment: .staging,
                apiBaseURL: "https://staging-api.example.com",
                analyticsEnabled: true,
                crashReportingEnabled: true
            )
        case .production:
            return AppConfiguration(
                environment: .production,
                apiBaseURL: "https://api.example.com",
                analyticsEnabled: true,
                crashReportingEnabled: true
            )
        }
    }
}

// 2. App Bootstrap
class AppBootstrap {
    
    static func initialize() {
        let config = AppConfiguration.current
        
        setupCrashReporting(enabled: config.crashReportingEnabled)
        setupAnalytics(enabled: config.analyticsEnabled)
        setupFeatureFlags()
        setupRemoteConfig()
    }
    
    private static func setupCrashReporting(enabled: Bool) {
        guard enabled else { return }
        // Crashlytics.crashlytics().setCrashlyticsCollectionEnabled(true)
        print("Crash reporting initialized")
    }
    
    private static func setupAnalytics(enabled: Bool) {
        guard enabled else { return }
        // Analytics.setAnalyticsCollectionEnabled(true)
        print("Analytics initialized")
    }
    
    private static func setupFeatureFlags() {
        Task {
            await FeatureFlagManager.shared.fetch()
        }
    }
    
    private static func setupRemoteConfig() {
        RemoteConfigManager.shared.initialize()
        Task {
            await RemoteConfigManager.shared.fetch()
        }
    }
}

extension FeatureFlagManager {
    func fetch() async {
        // Fetch from remote
    }
}
```

### แบบฝึกหัดที่ 2: Universal Links Deep Link Handler

**โจทย์:** สร้าง deep link handler ที่รองรับ URL patterns หลากหลาย

```swift
// เฉลย: Comprehensive Deep Link Handler

// Route definitions
enum DeepLinkRoute {
    case home
    case profile(userID: String)
    case product(id: String, category: String?)
    case search(query: String, filters: [String: String])
    case settings(section: String?)
    case notification(id: String)
    
    init?(url: URL) {
        guard let components = URLComponents(url: url, resolvingAgainstBaseURL: true) else {
            return nil
        }
        
        let pathComponents = url.pathComponents.filter { $0 != "/" }
        
        switch pathComponents.first {
        case "profile" where pathComponents.count >= 2:
            self = .profile(userID: pathComponents[1])
            
        case "products" where pathComponents.count >= 2:
            let category = components.queryItems?.first(where: { $0.name == "category" })?.value
            self = .product(id: pathComponents[1], category: category)
            
        case "search":
            let query = components.queryItems?.first(where: { $0.name == "q" })?.value ?? ""
            var filters: [String: String] = [:]
            components.queryItems?.forEach { item in
                if item.name != "q" {
                    filters[item.name] = item.value
                }
            }
            self = .search(query: query, filters: filters)
            
        case "settings":
            self = .settings(section: pathComponents.count >= 2 ? pathComponents[1] : nil)
            
        case "notifications" where pathComponents.count >= 2:
            self = .notification(id: pathComponents[1])
            
        case nil, "home":
            self = .home
            
        default:
            return nil
        }
    }
}

// Deep Link Coordinator
@MainActor
class DeepLinkCoordinator: ObservableObject {
    
    static let shared = DeepLinkCoordinator()
    
    @Published var currentRoute: DeepLinkRoute?
    
    func handle(url: URL) {
        guard let route = DeepLinkRoute(url: url) else {
            logWarning("Unhandled deep link: \(url)")
            return
        }
        
        logInfo("Handling deep link: \(url)")
        currentRoute = route
        
        // Track in analytics
        AnalyticsManager.shared.log(event: .buttonTap(
            buttonName: "deep_link",
            screenName: "\(route)"
        ))
    }
}
```

### แบบฝึกหัดที่ 3: Comprehensive Privacy Setup

**โจทย์:** สร้างระบบ privacy management ที่รองรับ GDPR, CCPA และ ATT

```swift
// เฉลย: Privacy Orchestrator

class PrivacyOrchestrator {
    
    static let shared = PrivacyOrchestrator()
    
    enum ComplianceRegion {
        case eu        // GDPR
        case california // CCPA
        case thailand  // PDPA
        case other     // Basic privacy
        
        static func detect(locale: Locale = .current) -> ComplianceRegion {
            let regionCode = locale.region?.identifier ?? ""
            switch regionCode {
            case "DE", "FR", "IT", "ES", "NL", "SE", "NO", "DK", "FI", "PL",
                 "AT", "BE", "CZ", "PT", "RO", "HU", "GR", "BG", "HR", "CY",
                 "EE", "IE", "LV", "LT", "LU", "MT", "SK", "SI":
                return .eu
            case "US":
                return .california  // สมมติว่าทุก US state ต้องการ CCPA (simplified)
            case "TH":
                return .thailand
            default:
                return .other
            }
        }
    }
    
    // Initialize privacy for user's region
    func initializeForRegion(_ region: ComplianceRegion) {
        switch region {
        case .eu:
            checkGDPRConsent()
        case .california:
            setupCCPANotice()
        case .thailand:
            checkPDPAConsent()
        case .other:
            setupBasicPrivacy()
        }
        
        // ATT เป็น requirement ของ Apple - ทุก region
        requestATTIfNeeded()
    }
    
    private func checkGDPRConsent() {
        if GDPRConsentManager.shared.needsConsentUpdate() {
            NotificationCenter.default.post(name: .showGDPRConsent, object: nil)
        }
    }
    
    private func setupCCPANotice() {
        // แสดง "Do Not Sell My Personal Information" ใน settings
        UserDefaults.standard.set(true, forKey: "show_ccpa_option")
    }
    
    private func checkPDPAConsent() {
        // Similar to GDPR
        let hasConsent = UserDefaults.standard.bool(forKey: "pdpa_consent_given")
        if !hasConsent {
            NotificationCenter.default.post(name: .showPDPAConsent, object: nil)
        }
    }
    
    private func setupBasicPrivacy() {
        // Minimal privacy setup
    }
    
    private func requestATTIfNeeded() {
        if ATTManager.shared.trackingStatus == .notDetermined {
            // รอให้ onboarding เสร็จก่อน
            DispatchQueue.main.asyncAfter(deadline: .now() + 2.0) {
                ATTManager.shared.requestTrackingPermission { _ in }
            }
        }
    }
}

extension Notification.Name {
    static let showGDPRConsent = Notification.Name("ShowGDPRConsent")
    static let showPDPAConsent = Notification.Name("ShowPDPAConsent")
}
```

---

## สรุป

ในบทนี้เราได้เรียนรู้การพัฒนา App แบบมืออาชีพครอบคลุม:

1. **Production Checklist** - รายการตรวจสอบก่อน submit
2. **App Size Optimization** - ลดขนาด App ด้วย on-demand resources และ binary optimization
3. **Launch Time** - เทคนิค optimize เวลา launch
4. **Crash Reporting** - Crashlytics และ Sentry integration
5. **Analytics** - Firebase Analytics และ Mixpanel
6. **A/B Testing** - Remote config และ local A/B testing
7. **Feature Flags** - Systematic feature management
8. **Remote Config** - Dynamic configuration
9. **Error Tracking** - Centralized error handling
10. **Logging Strategy** - Structured production logging
11. **Privacy Compliance** - GDPR, CCPA, PDPA
12. **Privacy Manifest** - Apple privacy requirements
13. **ATT** - App Tracking Transparency
14. **App Clips** - Mini app experience
15. **Universal Links** - Deep linking
16. **Handoff** - Cross-device continuity
17. **Siri/App Intents** - Voice interface
18. **Shortcuts** - Workflow automation
19. **Spotlight** - Search integration
20. **Share Sheet** - Content sharing
21. **iCloud Drive** - File synchronization

การ master ทักษะเหล่านี้จะทำให้ App ของคุณมีคุณภาพระดับ production ที่แท้จริง พร้อมสำหรับผู้ใช้จริงในทุก region และทุกสถานการณ์
