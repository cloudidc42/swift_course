# Part 34: Notifications - Local และ System ใน Swift

## บทนำ

Notifications ใน iOS/macOS มีสองรูปแบบหลัก:

1. **NotificationCenter**: ระบบ messaging ภายในแอป สำหรับการสื่อสารระหว่าง components
2. **UNUserNotificationCenter**: ระบบแจ้งเตือนให้ผู้ใช้ (Local/Push Notifications)

ในบทนี้เราจะครอบคลุม:
- NotificationCenter และการใช้งาน
- Combine กับ NotificationCenter
- async/await กับ NotificationCenter
- Local Notifications อย่างครบถ้วน
- Notification Actions และ Categories
- การสร้าง Reminder App

---

## 34.1 NotificationCenter

`NotificationCenter` เป็น design pattern แบบ Observer ที่ช่วยให้ objects สื่อสารกันโดยไม่ต้องรู้จักกันโดยตรง (loose coupling)

### หลักการทำงาน

```
Publisher ─── post(notification) ──► NotificationCenter ──► Observer 1
                                                         ──► Observer 2
                                                         ──► Observer 3
```

### การใช้งานพื้นฐาน

```swift
import Foundation

// NotificationCenter.default คือ instance มาตรฐาน
let center = NotificationCenter.default

// กำหนด Notification Name
let myNotificationName = Notification.Name("MyCustomNotification")

// หรือใช้ extension เพื่อ type safety
extension Notification.Name {
    static let userLoggedIn = Notification.Name("userLoggedIn")
    static let dataRefreshed = Notification.Name("dataRefreshed")
    static let themeChanged = Notification.Name("themeChanged")
    static let settingsUpdated = Notification.Name("settingsUpdated")
}
```

---

## 34.2 การ Post Notifications

```swift
import Foundation

// Post notification แบบง่าย
NotificationCenter.default.post(name: .userLoggedIn, object: nil)

// Post พร้อม object (sender)
class UserManager {
    static let shared = UserManager()
    var currentUser: String = ""
    
    func login(username: String) {
        currentUser = username
        
        // Post notification พร้อม sender
        NotificationCenter.default.post(
            name: .userLoggedIn,
            object: self
        )
    }
    
    func logout() {
        currentUser = ""
        
        // Post พร้อม userInfo
        NotificationCenter.default.post(
            name: Notification.Name("userLoggedOut"),
            object: self,
            userInfo: ["reason": "manual", "timestamp": Date()]
        )
    }
}

// Post พร้อม userInfo (dictionary ของข้อมูลเพิ่มเติม)
NotificationCenter.default.post(
    name: .dataRefreshed,
    object: nil,
    userInfo: [
        "source": "network",
        "itemCount": 42,
        "timestamp": Date()
    ]
)

// Post บน Main Thread
DispatchQueue.main.async {
    NotificationCenter.default.post(name: .themeChanged, object: nil)
}
```

---

## 34.3 การ Observe Notifications

```swift
import Foundation

class AppCoordinator {
    private var observers: [NSObjectProtocol] = []
    
    init() {
        setupObservers()
    }
    
    deinit {
        // สำคัญมาก: ต้อง remove observers เสมอ
        observers.forEach { NotificationCenter.default.removeObserver($0) }
    }
    
    private func setupObservers() {
        // วิธีที่ 1: addObserver(forName:object:queue:using:)
        let loginObserver = NotificationCenter.default.addObserver(
            forName: .userLoggedIn,
            object: nil,          // nil = รับจากทุก object
            queue: .main          // ประมวลผลบน main queue
        ) { notification in
            print("User logged in!")
            print("From: \(notification.object ?? "unknown")")
        }
        observers.append(loginObserver)
        
        // วิธีที่ 2: addObserver(_:selector:name:object:) - เหมาะกับ Objective-C style
        // ใช้ใน class ที่ inherit จาก NSObject
    }
}

// ใน UIViewController หรือ class ที่ inherit NSObject
class MyViewController: NSObject {
    
    override init() {
        super.init()
        
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(handleUserLogin(_:)),
            name: .userLoggedIn,
            object: nil
        )
    }
    
    deinit {
        NotificationCenter.default.removeObserver(self)
    }
    
    @objc private func handleUserLogin(_ notification: Notification) {
        print("User logged in via selector")
        if let userInfo = notification.userInfo {
            print("UserInfo: \(userInfo)")
        }
    }
}
```

### Observer Pattern ด้วย Closure

```swift
import Foundation

class EventBus {
    static let shared = EventBus()
    
    private var subscriptions: [UUID: NSObjectProtocol] = [:]
    
    private init() {}
    
    @discardableResult
    func subscribe(to name: Notification.Name, handler: @escaping (Notification) -> Void) -> UUID {
        let id = UUID()
        let observer = NotificationCenter.default.addObserver(
            forName: name,
            object: nil,
            queue: .main,
            using: handler
        )
        subscriptions[id] = observer
        return id
    }
    
    func unsubscribe(id: UUID) {
        if let observer = subscriptions.removeValue(forKey: id) {
            NotificationCenter.default.removeObserver(observer)
        }
    }
    
    func unsubscribeAll() {
        subscriptions.values.forEach { NotificationCenter.default.removeObserver($0) }
        subscriptions.removeAll()
    }
}

// การใช้งาน
let subscriptionID = EventBus.shared.subscribe(to: .dataRefreshed) { notification in
    print("Data refreshed!")
    if let count = notification.userInfo?["itemCount"] as? Int {
        print("Item count: \(count)")
    }
}

// ยกเลิก subscription
EventBus.shared.unsubscribe(id: subscriptionID)
```

---

## 34.4 การ Remove Observers

```swift
import Foundation

class SafeObserver {
    private var tokens: [NSObjectProtocol] = []
    
    func observe(_ name: Notification.Name, handler: @escaping (Notification) -> Void) {
        let token = NotificationCenter.default.addObserver(
            forName: name,
            object: nil,
            queue: .main,
            using: handler
        )
        tokens.append(token)
    }
    
    // ลบทั้งหมด
    func removeAll() {
        tokens.forEach { NotificationCenter.default.removeObserver($0) }
        tokens.removeAll()
    }
    
    deinit {
        removeAll()
    }
}

// วิธีที่ดีที่สุดด้วย Swift 5.5+
class ModernObserver {
    private var cancellables: [AnyObject] = []
    
    func startObserving() {
        // เก็บ token ใน property เพื่อ lifetime management
        let token = NotificationCenter.default.addObserver(
            forName: UIApplication.didBecomeActiveNotification,
            object: nil,
            queue: .main
        ) { [weak self] _ in
            self?.handleAppActive()
        }
        cancellables.append(token as AnyObject)
    }
    
    private func handleAppActive() {
        print("App became active")
    }
    
    deinit {
        cancellables.forEach { NotificationCenter.default.removeObserver($0) }
    }
}
```

---

## 34.5 Notification userInfo

```swift
import Foundation

// กำหนด keys สำหรับ userInfo
struct NotificationKeys {
    struct UserLogin {
        static let userId = "userId"
        static let userName = "userName"
        static let loginTime = "loginTime"
    }
    
    struct DataUpdate {
        static let source = "source"
        static let count = "count"
        static let error = "error"
    }
}

// Post พร้อม typed userInfo
class DataService {
    func fetchData() {
        // simulate fetch
        DispatchQueue.global().asyncAfter(deadline: .now() + 1) {
            let results = ["item1", "item2", "item3"]
            
            DispatchQueue.main.async {
                NotificationCenter.default.post(
                    name: .dataRefreshed,
                    object: self,
                    userInfo: [
                        NotificationKeys.DataUpdate.source: "API",
                        NotificationKeys.DataUpdate.count: results.count
                    ]
                )
            }
        }
    }
    
    func handleError(_ error: Error) {
        NotificationCenter.default.post(
            name: Notification.Name("dataFetchError"),
            object: self,
            userInfo: [
                NotificationKeys.DataUpdate.error: error,
                NotificationKeys.DataUpdate.source: "API"
            ]
        )
    }
}

// ดึงข้อมูลจาก userInfo อย่างปลอดภัย
extension Notification {
    var dataCount: Int? {
        return userInfo?[NotificationKeys.DataUpdate.count] as? Int
    }
    
    var dataSource: String? {
        return userInfo?[NotificationKeys.DataUpdate.source] as? String
    }
    
    var dataError: Error? {
        return userInfo?[NotificationKeys.DataUpdate.error] as? Error
    }
}

// Observer ที่ใช้ extension
NotificationCenter.default.addObserver(
    forName: .dataRefreshed,
    object: nil,
    queue: .main
) { notification in
    if let count = notification.dataCount {
        print("Received \(count) items from \(notification.dataSource ?? "unknown")")
    }
}
```

---

## 34.6 System Notifications

iOS/macOS มี system notifications ที่ built-in หลายอย่าง:

```swift
import UIKit

class SystemNotificationObserver {
    private var tokens: [NSObjectProtocol] = []
    
    func setupSystemObservers() {
        // App Lifecycle
        observe(UIApplication.didFinishLaunchingNotification) { _ in
            print("App launched")
        }
        
        observe(UIApplication.willResignActiveNotification) { _ in
            print("App will resign active - save data!")
        }
        
        observe(UIApplication.didBecomeActiveNotification) { _ in
            print("App became active - refresh data!")
        }
        
        observe(UIApplication.didEnterBackgroundNotification) { _ in
            print("App entered background")
        }
        
        observe(UIApplication.willEnterForegroundNotification) { _ in
            print("App will enter foreground")
        }
        
        observe(UIApplication.willTerminateNotification) { _ in
            print("App will terminate - final save!")
        }
        
        // Memory
        observe(UIApplication.didReceiveMemoryWarningNotification) { _ in
            print("Memory warning - clear caches!")
        }
        
        // Keyboard
        observe(UIResponder.keyboardWillShowNotification) { [weak self] notification in
            self?.handleKeyboardWillShow(notification)
        }
        
        observe(UIResponder.keyboardWillHideNotification) { [weak self] notification in
            self?.handleKeyboardWillHide(notification)
        }
        
        // Network
        observe(Notification.Name("kCFStreamErrorDomainNetServices")) { _ in
            print("Network change")
        }
        
        // User Interface
        observe(UIDevice.orientationDidChangeNotification) { _ in
            print("Orientation changed")
        }
        
        // Battery
        UIDevice.current.isBatteryMonitoringEnabled = true
        observe(UIDevice.batteryLevelDidChangeNotification) { _ in
            print("Battery level: \(UIDevice.current.batteryLevel)")
        }
        
        observe(UIDevice.batteryStateDidChangeNotification) { _ in
            let state = UIDevice.current.batteryState
            switch state {
            case .charging: print("Charging")
            case .full: print("Full")
            case .unplugged: print("Unplugged")
            case .unknown: print("Unknown")
            @unknown default: break
            }
        }
    }
    
    private func observe(_ name: Notification.Name, handler: @escaping (Notification) -> Void) {
        let token = NotificationCenter.default.addObserver(
            forName: name,
            object: nil,
            queue: .main,
            using: handler
        )
        tokens.append(token)
    }
    
    private func handleKeyboardWillShow(_ notification: Notification) {
        if let keyboardFrame = notification.userInfo?[UIResponder.keyboardFrameEndUserInfoKey] as? CGRect {
            print("Keyboard height: \(keyboardFrame.height)")
        }
        if let duration = notification.userInfo?[UIResponder.keyboardAnimationDurationUserInfoKey] as? Double {
            print("Animation duration: \(duration)")
        }
    }
    
    private func handleKeyboardWillHide(_ notification: Notification) {
        print("Keyboard hidden")
    }
    
    deinit {
        tokens.forEach { NotificationCenter.default.removeObserver($0) }
    }
}
```

---

## 34.7 Combine กับ NotificationCenter

Combine ทำให้การทำงานกับ NotificationCenter สะดวกและ functional มากขึ้น

```swift
import Foundation
import Combine

class CombineNotificationExample {
    private var cancellables = Set<AnyCancellable>()
    
    func setupCombineObservers() {
        // พื้นฐาน: แปลง notification เป็น Publisher
        NotificationCenter.default
            .publisher(for: .dataRefreshed)
            .sink { notification in
                print("Data refreshed via Combine")
            }
            .store(in: &cancellables)
        
        // กรองด้วย filter
        NotificationCenter.default
            .publisher(for: .dataRefreshed)
            .filter { notification in
                (notification.userInfo?["count"] as? Int ?? 0) > 0
            }
            .sink { notification in
                let count = notification.userInfo?["count"] as? Int ?? 0
                print("Got \(count) items")
            }
            .store(in: &cancellables)
        
        // แปลงข้อมูลด้วย map
        NotificationCenter.default
            .publisher(for: .userLoggedIn)
            .compactMap { notification in
                notification.userInfo?["userName"] as? String
            }
            .sink { userName in
                print("User logged in: \(userName)")
            }
            .store(in: &cancellables)
        
        // Debounce สำหรับ notifications ที่เกิดบ่อย
        NotificationCenter.default
            .publisher(for: UITextField.textDidChangeNotification)
            .debounce(for: .milliseconds(300), scheduler: RunLoop.main)
            .compactMap { notification in
                (notification.object as? UITextField)?.text
            }
            .sink { text in
                print("Search: \(text)")
            }
            .store(in: &cancellables)
        
        // Merge หลาย notifications
        let loginPublisher = NotificationCenter.default.publisher(for: .userLoggedIn)
        let logoutPublisher = NotificationCenter.default.publisher(for: Notification.Name("userLoggedOut"))
        
        loginPublisher.merge(with: logoutPublisher)
            .sink { notification in
                if notification.name == .userLoggedIn {
                    print("User logged in")
                } else {
                    print("User logged out")
                }
            }
            .store(in: &cancellables)
        
        // รับ notification ครั้งแรกครั้งเดียว
        NotificationCenter.default
            .publisher(for: UIApplication.didBecomeActiveNotification)
            .first()
            .sink { _ in
                print("First time becoming active")
            }
            .store(in: &cancellables)
    }
}

// ตัวอย่าง: Keyboard height binding ด้วย Combine
class KeyboardHandler: ObservableObject {
    @Published var keyboardHeight: CGFloat = 0
    
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        NotificationCenter.default
            .publisher(for: UIResponder.keyboardWillShowNotification)
            .compactMap { notification in
                notification.userInfo?[UIResponder.keyboardFrameEndUserInfoKey] as? CGRect
            }
            .map { $0.height }
            .assign(to: \.keyboardHeight, on: self)
            .store(in: &cancellables)
        
        NotificationCenter.default
            .publisher(for: UIResponder.keyboardWillHideNotification)
            .map { _ in CGFloat(0) }
            .assign(to: \.keyboardHeight, on: self)
            .store(in: &cancellables)
    }
}
```

---

## 34.8 async/await กับ NotificationCenter

Swift 5.5+ รองรับ async/await กับ NotificationCenter

```swift
import Foundation
import UIKit

// AsyncSequence จาก NotificationCenter
class AsyncNotificationExample {
    
    func waitForAppActive() async {
        // รอ notification เดียว
        let center = NotificationCenter.default
        
        // Swift 5.5+ AsyncSequence
        for await notification in center.notifications(named: UIApplication.didBecomeActiveNotification) {
            print("App became active: \(notification)")
            break  // รับครั้งเดียวแล้วหยุด
        }
    }
    
    func observeDataUpdates() async {
        let center = NotificationCenter.default
        
        // วน loop รับ notifications
        for await notification in center.notifications(named: .dataRefreshed) {
            if let count = notification.userInfo?["count"] as? Int {
                print("Received \(count) items")
            }
        }
    }
    
    // รอ notification พร้อม timeout
    func waitForLogin(timeout: TimeInterval = 30) async throws -> String {
        try await withThrowingTaskGroup(of: String.self) { group in
            // Task 1: รอ notification
            group.addTask {
                let center = NotificationCenter.default
                for await notification in center.notifications(named: .userLoggedIn) {
                    if let userName = notification.userInfo?["userName"] as? String {
                        return userName
                    }
                }
                throw CancellationError()
            }
            
            // Task 2: Timeout
            group.addTask {
                try await Task.sleep(nanoseconds: UInt64(timeout * 1_000_000_000))
                throw NSError(domain: "Timeout", code: 408, userInfo: [
                    NSLocalizedDescriptionKey: "Login timeout"
                ])
            }
            
            // รับผลลัพธ์แรก
            let result = try await group.next()!
            group.cancelAll()
            return result
        }
    }
}

// ใช้งานใน async context
Task {
    let example = AsyncNotificationExample()
    await example.waitForAppActive()
    
    do {
        let userName = try await example.waitForLogin(timeout: 10)
        print("Logged in as: \(userName)")
    } catch {
        print("Login failed: \(error)")
    }
}
```

---

## 34.9 UNUserNotificationCenter

`UNUserNotificationCenter` จัดการ local และ push notifications ให้กับผู้ใช้

```swift
import UserNotifications
import UIKit

// Setup ใน AppDelegate
class AppDelegate: UIResponder, UIApplicationDelegate, UNUserNotificationCenterDelegate {
    
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        
        // ตั้ง delegate
        UNUserNotificationCenter.current().delegate = self
        
        return true
    }
    
    // รับ notification ขณะแอปเปิดอยู่ (Foreground)
    func userNotificationCenter(
        _ center: UNUserNotificationCenter,
        willPresent notification: UNNotification,
        withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void
    ) {
        // กำหนดว่าจะแสดง notification อย่างไรเมื่อแอปอยู่ foreground
        completionHandler([.banner, .badge, .sound, .list])
    }
    
    // จัดการเมื่อผู้ใช้กด notification
    func userNotificationCenter(
        _ center: UNUserNotificationCenter,
        didReceive response: UNNotificationResponse,
        withCompletionHandler completionHandler: @escaping () -> Void
    ) {
        let notification = response.notification
        let actionIdentifier = response.actionIdentifier
        let userInfo = notification.request.content.userInfo
        
        print("Notification tapped: \(notification.request.identifier)")
        print("Action: \(actionIdentifier)")
        print("UserInfo: \(userInfo)")
        
        // จัดการ action ต่างๆ
        switch actionIdentifier {
        case "ACCEPT_ACTION":
            print("User accepted")
        case "DECLINE_ACTION":
            print("User declined")
        case UNNotificationDefaultActionIdentifier:
            print("Default tap")
        case UNNotificationDismissActionIdentifier:
            print("Notification dismissed")
        default:
            break
        }
        
        completionHandler()
    }
}
```

---

## 34.10 การขอ Permission

```swift
import UserNotifications

class NotificationPermissionManager {
    static let shared = NotificationPermissionManager()
    
    private init() {}
    
    // ขอ permission
    func requestPermission() async throws -> Bool {
        let options: UNAuthorizationOptions = [.alert, .badge, .sound, .provisional]
        
        let granted = try await UNUserNotificationCenter.current().requestAuthorization(options: options)
        return granted
    }
    
    // ตรวจสอบ status ปัจจุบัน
    func checkStatus() async -> UNAuthorizationStatus {
        let settings = await UNUserNotificationCenter.current().notificationSettings()
        return settings.authorizationStatus
    }
    
    // ขอ permission พร้อม handle ผลลัพธ์
    func requestPermissionWithFeedback() async {
        do {
            let granted = try await requestPermission()
            
            if granted {
                print("Permission granted - notifications enabled")
                await registerForRemoteNotifications()
            } else {
                print("Permission denied - notifications disabled")
            }
        } catch {
            print("Permission request error: \(error)")
        }
    }
    
    // ลงทะเบียนสำหรับ push notifications
    @MainActor
    private func registerForRemoteNotifications() async {
        UIApplication.shared.registerForRemoteNotifications()
    }
    
    // เปิดการตั้งค่า notifications
    func openNotificationSettings() {
        if let url = URL(string: UIApplication.openNotificationSettingsURLString) {
            UIApplication.shared.open(url)
        }
    }
    
    // ตรวจสอบและแสดง dialog ถ้าจำเป็น
    func ensurePermission() async -> Bool {
        let status = await checkStatus()
        
        switch status {
        case .authorized, .provisional, .ephemeral:
            return true
        case .notDetermined:
            return (try? await requestPermission()) ?? false
        case .denied:
            // ไม่สามารถขออีกได้ ต้องให้ผู้ใช้ไปตั้งค่าเอง
            await MainActor.run {
                openNotificationSettings()
            }
            return false
        @unknown default:
            return false
        }
    }
}

// SwiftUI View สำหรับขอ permission
import SwiftUI

struct NotificationPermissionView: View {
    @State private var permissionStatus: UNAuthorizationStatus = .notDetermined
    @State private var isLoading = false
    
    var body: some View {
        VStack(spacing: 20) {
            Image(systemName: "bell.circle.fill")
                .font(.system(size: 80))
                .foregroundColor(.blue)
            
            Text("เปิดการแจ้งเตือน")
                .font(.title)
                .fontWeight(.bold)
            
            Text("เพื่อไม่ให้พลาดข่าวสารสำคัญ กรุณาอนุญาตการแจ้งเตือน")
                .multilineTextAlignment(.center)
                .foregroundColor(.secondary)
            
            statusView
            
            Button(action: handlePermission) {
                if isLoading {
                    ProgressView()
                        .frame(maxWidth: .infinity)
                } else {
                    Text(buttonTitle)
                        .frame(maxWidth: .infinity)
                }
            }
            .buttonStyle(.borderedProminent)
            .disabled(isLoading)
        }
        .padding()
        .task {
            permissionStatus = await NotificationPermissionManager.shared.checkStatus()
        }
    }
    
    private var statusView: some View {
        HStack {
            Image(systemName: statusIcon)
            Text(statusText)
        }
        .foregroundColor(statusColor)
    }
    
    private var statusIcon: String {
        switch permissionStatus {
        case .authorized, .provisional: return "checkmark.circle.fill"
        case .denied: return "xmark.circle.fill"
        default: return "questionmark.circle.fill"
        }
    }
    
    private var statusText: String {
        switch permissionStatus {
        case .authorized: return "อนุญาตแล้ว"
        case .provisional: return "อนุญาตชั่วคราว"
        case .denied: return "ปฏิเสธ"
        case .notDetermined: return "ยังไม่ได้ตัดสินใจ"
        default: return "ไม่ทราบ"
        }
    }
    
    private var statusColor: Color {
        switch permissionStatus {
        case .authorized, .provisional: return .green
        case .denied: return .red
        default: return .orange
        }
    }
    
    private var buttonTitle: String {
        switch permissionStatus {
        case .authorized: return "เปิดการตั้งค่า"
        case .denied: return "ไปที่การตั้งค่า"
        default: return "อนุญาตการแจ้งเตือน"
        }
    }
    
    private func handlePermission() {
        isLoading = true
        Task {
            _ = await NotificationPermissionManager.shared.ensurePermission()
            permissionStatus = await NotificationPermissionManager.shared.checkStatus()
            isLoading = false
        }
    }
}
```

---

## 34.11 UNNotificationRequest

```swift
import UserNotifications

// การสร้าง Notification Request
class NotificationBuilder {
    
    // สร้าง basic notification
    static func basicNotification(
        id: String = UUID().uuidString,
        title: String,
        body: String,
        trigger: UNNotificationTrigger? = nil
    ) -> UNNotificationRequest {
        let content = UNMutableNotificationContent()
        content.title = title
        content.body = body
        content.sound = .default
        
        return UNNotificationRequest(
            identifier: id,
            content: content,
            trigger: trigger
        )
    }
    
    // สร้าง notification พร้อมทุก property
    static func richNotification(
        id: String,
        title: String,
        subtitle: String? = nil,
        body: String,
        categoryIdentifier: String? = nil,
        userInfo: [AnyHashable: Any] = [:],
        badge: Int? = nil,
        sound: UNNotificationSound = .default,
        attachments: [UNNotificationAttachment] = [],
        trigger: UNNotificationTrigger? = nil
    ) -> UNNotificationRequest {
        let content = UNMutableNotificationContent()
        content.title = title
        content.body = body
        content.sound = sound
        content.userInfo = userInfo
        content.attachments = attachments
        
        if let subtitle = subtitle {
            content.subtitle = subtitle
        }
        if let categoryIdentifier = categoryIdentifier {
            content.categoryIdentifier = categoryIdentifier
        }
        if let badge = badge {
            content.badge = NSNumber(value: badge)
        }
        
        return UNNotificationRequest(
            identifier: id,
            content: content,
            trigger: trigger
        )
    }
}
```

---

## 34.12 UNNotificationContent

```swift
import UserNotifications

// การสร้าง content แบบต่างๆ
class NotificationContentFactory {
    
    // Text notification
    static func textContent(title: String, body: String) -> UNMutableNotificationContent {
        let content = UNMutableNotificationContent()
        content.title = title
        content.body = body
        content.sound = .default
        return content
    }
    
    // Notification พร้อมรูป (Attachment)
    static func imageContent(title: String, body: String, imageURL: URL) -> UNMutableNotificationContent {
        let content = UNMutableNotificationContent()
        content.title = title
        content.body = body
        
        if let attachment = try? UNNotificationAttachment(
            identifier: "image",
            url: imageURL,
            options: [
                UNNotificationAttachmentOptionsThumbnailClippingRectKey: CGRect(x: 0, y: 0, width: 1, height: 0.5).dictionaryRepresentation
            ]
        ) {
            content.attachments = [attachment]
        }
        
        return content
    }
    
    // Notification พร้อม action buttons
    static func actionContent(
        title: String,
        body: String,
        categoryIdentifier: String
    ) -> UNMutableNotificationContent {
        let content = UNMutableNotificationContent()
        content.title = title
        content.body = body
        content.categoryIdentifier = categoryIdentifier
        content.sound = .default
        return content
    }
    
    // Notification พร้อมข้อมูล
    static func dataContent(
        title: String,
        body: String,
        data: [String: Any]
    ) -> UNMutableNotificationContent {
        let content = UNMutableNotificationContent()
        content.title = title
        content.body = body
        content.userInfo = data
        content.sound = .default
        return content
    }
    
    // Notification สำหรับ reminder
    static func reminderContent(
        taskTitle: String,
        dueDate: Date,
        notes: String? = nil
    ) -> UNMutableNotificationContent {
        let content = UNMutableNotificationContent()
        content.title = "ครบกำหนด: \(taskTitle)"
        content.body = notes ?? "งานนี้ครบกำหนดแล้ว"
        content.categoryIdentifier = "REMINDER_CATEGORY"
        content.sound = .default
        content.userInfo = [
            "taskTitle": taskTitle,
            "dueDate": dueDate.timeIntervalSince1970
        ]
        return content
    }
}
```

---

## 34.13 UNNotificationTrigger

### Time Interval Trigger

```swift
import UserNotifications

// Trigger ตาม time interval
func scheduleDelayedNotification(after seconds: TimeInterval) {
    let content = UNMutableNotificationContent()
    content.title = "แจ้งเตือน"
    content.body = "เวลาผ่านไป \(Int(seconds)) วินาทีแล้ว"
    content.sound = .default
    
    // repeats: true = วนซ้ำทุก interval
    // repeats: false = แจ้งเตือนครั้งเดียว
    let trigger = UNTimeIntervalNotificationTrigger(
        timeInterval: seconds,
        repeats: false
    )
    
    let request = UNNotificationRequest(
        identifier: "delayed-\(UUID().uuidString)",
        content: content,
        trigger: trigger
    )
    
    UNUserNotificationCenter.current().add(request) { error in
        if let error = error {
            print("Error scheduling: \(error)")
        } else {
            print("Scheduled for \(seconds)s from now")
        }
    }
}

// ตัวอย่าง: แจ้งเตือนอีก 5 นาที
scheduleDelayedNotification(after: 300)
```

### Calendar Trigger

```swift
import UserNotifications

// Trigger ตามเวลาในปฏิทิน
func scheduleAtTime(hour: Int, minute: Int, repeats: Bool = false) {
    let content = UNMutableNotificationContent()
    content.title = "เตือนความจำ"
    content.body = "ถึงเวลา \(hour):\(String(format: "%02d", minute)) แล้ว"
    content.sound = .default
    
    var dateComponents = DateComponents()
    dateComponents.hour = hour
    dateComponents.minute = minute
    
    let trigger = UNCalendarNotificationTrigger(
        dateMatching: dateComponents,
        repeats: repeats
    )
    
    let request = UNNotificationRequest(
        identifier: "daily-\(hour)-\(minute)",
        content: content,
        trigger: trigger
    )
    
    UNUserNotificationCenter.current().add(request) { error in
        if let error = error {
            print("Error: \(error)")
        } else {
            print("Scheduled at \(hour):\(minute) - repeats: \(repeats)")
        }
    }
}

// แจ้งเตือนทุกวันเวลา 9:00 น.
scheduleAtTime(hour: 9, minute: 0, repeats: true)

// แจ้งเตือนวันจันทร์เวลา 10:30
func scheduleWeeklyNotification() {
    let content = UNMutableNotificationContent()
    content.title = "Weekly Review"
    content.body = "ถึงเวลา review งานสัปดาห์นี้แล้ว"
    
    var dateComponents = DateComponents()
    dateComponents.weekday = 2  // 1=Sunday, 2=Monday
    dateComponents.hour = 10
    dateComponents.minute = 30
    
    let trigger = UNCalendarNotificationTrigger(
        dateMatching: dateComponents,
        repeats: true
    )
    
    let request = UNNotificationRequest(
        identifier: "weekly-review",
        content: content,
        trigger: trigger
    )
    
    UNUserNotificationCenter.current().add(request, withCompletionHandler: nil)
}

// แจ้งเตือนตาม Date
func scheduleAtDate(_ date: Date, id: String = UUID().uuidString) {
    let content = UNMutableNotificationContent()
    content.title = "แจ้งเตือน"
    content.body = "ถึงเวลาที่กำหนดแล้ว"
    
    let components = Calendar.current.dateComponents(
        [.year, .month, .day, .hour, .minute, .second],
        from: date
    )
    
    let trigger = UNCalendarNotificationTrigger(
        dateMatching: components,
        repeats: false
    )
    
    let request = UNNotificationRequest(
        identifier: id,
        content: content,
        trigger: trigger
    )
    
    UNUserNotificationCenter.current().add(request, withCompletionHandler: nil)
}
```

### Location Trigger

```swift
import UserNotifications
import CoreLocation

// Trigger ตาม location (ต้องมี location permission)
func scheduleLocationNotification(coordinate: CLLocationCoordinate2D, radius: CLLocationDistance, identifier: String) {
    let content = UNMutableNotificationContent()
    content.title = "คุณมาถึงแล้ว"
    content.body = "ยินดีต้อนรับสู่สถานที่นี้"
    content.sound = .default
    
    let region = CLCircularRegion(
        center: coordinate,
        radius: radius,
        identifier: identifier
    )
    region.notifyOnEntry = true   // แจ้งเตือนเมื่อเข้า
    region.notifyOnExit = false   // ไม่แจ้งเตือนเมื่อออก
    
    let trigger = UNLocationNotificationTrigger(
        region: region,
        repeats: false
    )
    
    let request = UNNotificationRequest(
        identifier: "location-\(identifier)",
        content: content,
        trigger: trigger
    )
    
    UNUserNotificationCenter.current().add(request) { error in
        if let error = error {
            print("Location notification error: \(error)")
        } else {
            print("Location notification scheduled")
        }
    }
}

// ตัวอย่าง: แจ้งเตือนเมื่อถึงบ้าน
let homeCoordinate = CLLocationCoordinate2D(latitude: 13.7563, longitude: 100.5018)
scheduleLocationNotification(
    coordinate: homeCoordinate,
    radius: 100,  // เมตร
    identifier: "home"
)
```

---

## 34.14 การ Schedule Local Notifications

```swift
import UserNotifications

class NotificationScheduler {
    static let shared = NotificationScheduler()
    private let center = UNUserNotificationCenter.current()
    
    private init() {}
    
    // Schedule notification แบบ async
    func schedule(_ request: UNNotificationRequest) async throws {
        try await center.add(request)
    }
    
    // Schedule หลาย notifications
    func scheduleMultiple(_ requests: [UNNotificationRequest]) async {
        await withTaskGroup(of: Void.self) { group in
            for request in requests {
                group.addTask {
                    try? await self.center.add(request)
                }
            }
        }
    }
    
    // Schedule reminder
    func scheduleReminder(
        id: String,
        title: String,
        message: String,
        at date: Date,
        userInfo: [AnyHashable: Any] = [:]
    ) async throws {
        let content = UNMutableNotificationContent()
        content.title = title
        content.body = message
        content.sound = .default
        content.categoryIdentifier = "REMINDER"
        content.userInfo = userInfo
        
        let components = Calendar.current.dateComponents(
            [.year, .month, .day, .hour, .minute],
            from: date
        )
        
        let trigger = UNCalendarNotificationTrigger(
            dateMatching: components,
            repeats: false
        )
        
        let request = UNNotificationRequest(
            identifier: id,
            content: content,
            trigger: trigger
        )
        
        try await schedule(request)
        print("Reminder scheduled: \(title) at \(date)")
    }
    
    // Schedule daily reminder
    func scheduleDailyReminder(
        id: String,
        title: String,
        message: String,
        hour: Int,
        minute: Int
    ) async throws {
        let content = UNMutableNotificationContent()
        content.title = title
        content.body = message
        content.sound = .default
        
        var components = DateComponents()
        components.hour = hour
        components.minute = minute
        
        let trigger = UNCalendarNotificationTrigger(
            dateMatching: components,
            repeats: true
        )
        
        let request = UNNotificationRequest(
            identifier: id,
            content: content,
            trigger: trigger
        )
        
        try await schedule(request)
    }
    
    // Cancel notification
    func cancel(id: String) {
        center.removePendingNotificationRequests(withIdentifiers: [id])
        center.removeDeliveredNotifications(withIdentifiers: [id])
    }
    
    // Cancel หลาย notifications
    func cancelMultiple(ids: [String]) {
        center.removePendingNotificationRequests(withIdentifiers: ids)
        center.removeDeliveredNotifications(withIdentifiers: ids)
    }
    
    // Cancel ทั้งหมด
    func cancelAll() {
        center.removeAllPendingNotificationRequests()
        center.removeAllDeliveredNotifications()
    }
}
```

---

## 34.15 Pending Notifications

```swift
import UserNotifications

class PendingNotificationsManager {
    let center = UNUserNotificationCenter.current()
    
    // รายการ pending notifications
    func getPendingNotifications() async -> [UNNotificationRequest] {
        await center.pendingNotificationRequests()
    }
    
    // ตรวจสอบว่า notification มีอยู่หรือไม่
    func hasPendingNotification(id: String) async -> Bool {
        let pending = await getPendingNotifications()
        return pending.contains { $0.identifier == id }
    }
    
    // แสดงรายการ pending
    func printPendingNotifications() async {
        let requests = await getPendingNotifications()
        
        print("Pending notifications: \(requests.count)")
        for request in requests {
            print("ID: \(request.identifier)")
            print("  Title: \(request.content.title)")
            if let trigger = request.trigger as? UNCalendarNotificationTrigger {
                print("  Trigger: \(trigger.dateComponents)")
            } else if let trigger = request.trigger as? UNTimeIntervalNotificationTrigger {
                print("  In: \(trigger.timeInterval)s")
            }
        }
    }
    
    // อัพเดต notification (ต้อง reschedule)
    func updateNotification(id: String, newContent: UNMutableNotificationContent, newTrigger: UNNotificationTrigger?) async throws {
        // ลบอันเก่า
        center.removePendingNotificationRequests(withIdentifiers: [id])
        
        // สร้างอันใหม่
        let request = UNNotificationRequest(
            identifier: id,
            content: newContent,
            trigger: newTrigger
        )
        
        try await center.add(request)
    }
}
```

---

## 34.16 Delivered Notifications

```swift
import UserNotifications

class DeliveredNotificationsManager {
    let center = UNUserNotificationCenter.current()
    
    // รายการ delivered notifications
    func getDeliveredNotifications() async -> [UNNotification] {
        await center.deliveredNotifications()
    }
    
    // ลบ notification ที่ส่งไปแล้ว
    func removeDelivered(ids: [String]) {
        center.removeDeliveredNotifications(withIdentifiers: ids)
    }
    
    // ลบทั้งหมด
    func removeAllDelivered() {
        center.removeAllDeliveredNotifications()
    }
    
    // นับ delivered notifications
    func deliveredCount() async -> Int {
        await getDeliveredNotifications().count
    }
    
    // แสดงสรุป
    func printDeliveredSummary() async {
        let notifications = await getDeliveredNotifications()
        print("Delivered notifications: \(notifications.count)")
        for notification in notifications {
            let date = notification.date
            let title = notification.request.content.title
            print("[\(date)] \(title)")
        }
    }
}
```

---

## 34.17 Notification Actions

```swift
import UserNotifications

class NotificationActionsSetup {
    
    static func setupCategories() {
        // Reminder category
        let reminderCategory = createReminderCategory()
        
        // Message category
        let messageCategory = createMessageCategory()
        
        // Download category
        let downloadCategory = createDownloadCategory()
        
        // ลงทะเบียน categories
        UNUserNotificationCenter.current().setNotificationCategories([
            reminderCategory,
            messageCategory,
            downloadCategory
        ])
    }
    
    private static func createReminderCategory() -> UNNotificationCategory {
        // Action 1: เสร็จแล้ว
        let doneAction = UNNotificationAction(
            identifier: "MARK_DONE",
            title: "เสร็จแล้ว",
            options: [.foreground]
        )
        
        // Action 2: เลื่อนออกไป
        let snoozeAction = UNNotificationAction(
            identifier: "SNOOZE",
            title: "เลื่อน 15 นาที",
            options: []
        )
        
        // Action 3: ลบ (destructive)
        let deleteAction = UNNotificationAction(
            identifier: "DELETE_REMINDER",
            title: "ลบ",
            options: [.destructive]
        )
        
        return UNNotificationCategory(
            identifier: "REMINDER",
            actions: [doneAction, snoozeAction, deleteAction],
            intentIdentifiers: [],
            options: [.customDismissAction]
        )
    }
    
    private static func createMessageCategory() -> UNNotificationCategory {
        // Text input action
        let replyAction = UNTextInputNotificationAction(
            identifier: "REPLY",
            title: "ตอบกลับ",
            options: [.foreground],
            textInputButtonTitle: "ส่ง",
            textInputPlaceholder: "พิมพ์ข้อความ..."
        )
        
        let likeAction = UNNotificationAction(
            identifier: "LIKE",
            title: "ถูกใจ ❤️",
            options: []
        )
        
        return UNNotificationCategory(
            identifier: "MESSAGE",
            actions: [replyAction, likeAction],
            intentIdentifiers: [],
            options: []
        )
    }
    
    private static func createDownloadCategory() -> UNNotificationCategory {
        let viewAction = UNNotificationAction(
            identifier: "VIEW_FILE",
            title: "ดูไฟล์",
            options: [.foreground]
        )
        
        let shareAction = UNNotificationAction(
            identifier: "SHARE_FILE",
            title: "แชร์",
            options: [.foreground]
        )
        
        return UNNotificationCategory(
            identifier: "DOWNLOAD_COMPLETE",
            actions: [viewAction, shareAction],
            intentIdentifiers: [],
            options: []
        )
    }
}

// การจัดการ action responses
class NotificationResponseHandler {
    
    func handle(_ response: UNNotificationResponse) {
        let identifier = response.notification.request.identifier
        let userInfo = response.notification.request.content.userInfo
        
        switch response.actionIdentifier {
        case "MARK_DONE":
            handleMarkDone(notificationID: identifier, userInfo: userInfo)
            
        case "SNOOZE":
            handleSnooze(notificationID: identifier, userInfo: userInfo)
            
        case "DELETE_REMINDER":
            handleDelete(notificationID: identifier, userInfo: userInfo)
            
        case "REPLY":
            if let textResponse = response as? UNTextInputNotificationResponse {
                handleReply(text: textResponse.userText, userInfo: userInfo)
            }
            
        case "LIKE":
            handleLike(userInfo: userInfo)
            
        case UNNotificationDefaultActionIdentifier:
            // ผู้ใช้กด notification
            openApp(with: userInfo)
            
        case UNNotificationDismissActionIdentifier:
            // ผู้ใช้ dismiss
            logDismiss(identifier: identifier)
            
        default:
            break
        }
    }
    
    private func handleMarkDone(notificationID: String, userInfo: [AnyHashable: Any]) {
        if let taskID = userInfo["taskID"] as? String {
            print("Mark task done: \(taskID)")
            // TaskManager.shared.complete(id: taskID)
        }
        UNUserNotificationCenter.current().removeDeliveredNotifications(withIdentifiers: [notificationID])
    }
    
    private func handleSnooze(notificationID: String, userInfo: [AnyHashable: Any]) {
        print("Snooze for 15 minutes")
        
        // สร้าง notification ใหม่อีก 15 นาที
        let content = UNMutableNotificationContent()
        content.title = "เตือนอีกครั้ง"
        content.body = userInfo["taskTitle"] as? String ?? "งานที่ต้องทำ"
        content.sound = .default
        content.categoryIdentifier = "REMINDER"
        content.userInfo = userInfo
        
        let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 900, repeats: false)
        let request = UNNotificationRequest(
            identifier: "\(notificationID)-snooze",
            content: content,
            trigger: trigger
        )
        
        UNUserNotificationCenter.current().add(request, withCompletionHandler: nil)
    }
    
    private func handleDelete(notificationID: String, userInfo: [AnyHashable: Any]) {
        if let taskID = userInfo["taskID"] as? String {
            print("Delete task: \(taskID)")
        }
    }
    
    private func handleReply(text: String, userInfo: [AnyHashable: Any]) {
        print("Reply: \(text)")
    }
    
    private func handleLike(userInfo: [AnyHashable: Any]) {
        print("Liked")
    }
    
    private func openApp(with userInfo: [AnyHashable: Any]) {
        print("Open app with: \(userInfo)")
    }
    
    private func logDismiss(identifier: String) {
        print("Dismissed: \(identifier)")
    }
}
```

---

## 34.18 Notification Categories

```swift
import UserNotifications

// ระบบ Category ที่สมบูรณ์
class NotificationCategoryManager {
    static let shared = NotificationCategoryManager()
    
    enum CategoryID: String {
        case reminder = "REMINDER"
        case message = "MESSAGE"
        case alert = "ALERT"
        case download = "DOWNLOAD"
        case social = "SOCIAL"
    }
    
    private init() {}
    
    func setup() {
        var categories: Set<UNNotificationCategory> = []
        
        categories.insert(reminderCategory)
        categories.insert(messageCategory)
        categories.insert(alertCategory)
        categories.insert(downloadCategory)
        categories.insert(socialCategory)
        
        UNUserNotificationCenter.current().setNotificationCategories(categories)
    }
    
    private var reminderCategory: UNNotificationCategory {
        UNNotificationCategory(
            identifier: CategoryID.reminder.rawValue,
            actions: [
                UNNotificationAction(identifier: "COMPLETE", title: "เสร็จแล้ว ✓", options: []),
                UNNotificationAction(identifier: "SNOOZE", title: "เลื่อน", options: []),
                UNNotificationAction(identifier: "DELETE", title: "ลบ", options: [.destructive])
            ],
            intentIdentifiers: [],
            options: [.customDismissAction]
        )
    }
    
    private var messageCategory: UNNotificationCategory {
        UNNotificationCategory(
            identifier: CategoryID.message.rawValue,
            actions: [
                UNTextInputNotificationAction(
                    identifier: "REPLY",
                    title: "ตอบกลับ",
                    options: [.foreground],
                    textInputButtonTitle: "ส่ง",
                    textInputPlaceholder: "ข้อความ..."
                ),
                UNNotificationAction(identifier: "MARK_READ", title: "อ่านแล้ว", options: [])
            ],
            intentIdentifiers: [],
            options: []
        )
    }
    
    private var alertCategory: UNNotificationCategory {
        UNNotificationCategory(
            identifier: CategoryID.alert.rawValue,
            actions: [
                UNNotificationAction(identifier: "VIEW", title: "ดูรายละเอียด", options: [.foreground]),
                UNNotificationAction(identifier: "DISMISS", title: "ปิด", options: [])
            ],
            intentIdentifiers: [],
            options: []
        )
    }
    
    private var downloadCategory: UNNotificationCategory {
        UNNotificationCategory(
            identifier: CategoryID.download.rawValue,
            actions: [
                UNNotificationAction(identifier: "OPEN", title: "เปิดไฟล์", options: [.foreground]),
                UNNotificationAction(identifier: "SHARE", title: "แชร์", options: [.foreground])
            ],
            intentIdentifiers: [],
            options: []
        )
    }
    
    private var socialCategory: UNNotificationCategory {
        UNNotificationCategory(
            identifier: CategoryID.social.rawValue,
            actions: [
                UNNotificationAction(identifier: "LIKE", title: "❤️ ถูกใจ", options: []),
                UNNotificationAction(identifier: "COMMENT", title: "💬 คอมเมนต์", options: [.foreground]),
                UNNotificationAction(identifier: "SHARE", title: "📤 แชร์", options: [.foreground])
            ],
            intentIdentifiers: [],
            options: []
        )
    }
}
```

---

## 34.19 Notification Extensions

### Notification Service Extension

Service Extension ใช้แก้ไข notification content ก่อนแสดง (สำหรับ push notifications)

```swift
// NotificationServiceExtension.swift - ใน Notification Service Extension target
import UserNotifications

class NotificationService: UNNotificationServiceExtension {
    var contentHandler: ((UNNotificationContent) -> Void)?
    var bestAttemptContent: UNMutableNotificationContent?
    
    override func didReceive(
        _ request: UNNotificationRequest,
        withContentHandler contentHandler: @escaping (UNNotificationContent) -> Void
    ) {
        self.contentHandler = contentHandler
        bestAttemptContent = (request.content.mutableCopy() as? UNMutableNotificationContent)
        
        guard let bestAttemptContent = bestAttemptContent else {
            contentHandler(request.content)
            return
        }
        
        // แก้ไข content
        bestAttemptContent.title = "[Modified] " + bestAttemptContent.title
        
        // ดาวน์โหลด attachment ถ้ามี URL
        if let attachmentURLString = request.content.userInfo["imageURL"] as? String,
           let attachmentURL = URL(string: attachmentURLString) {
            downloadAndAttach(url: attachmentURL, to: bestAttemptContent) {
                contentHandler(bestAttemptContent)
            }
        } else {
            contentHandler(bestAttemptContent)
        }
    }
    
    override func serviceExtensionTimeWillExpire() {
        // เวลาหมด - ส่ง content ที่มีอยู่ไปก่อน
        if let contentHandler = contentHandler,
           let bestAttemptContent = bestAttemptContent {
            contentHandler(bestAttemptContent)
        }
    }
    
    private func downloadAndAttach(url: URL, to content: UNMutableNotificationContent, completion: @escaping () -> Void) {
        URLSession.shared.downloadTask(with: url) { tempURL, _, error in
            guard let tempURL = tempURL, error == nil else {
                completion()
                return
            }
            
            // สร้างไฟล์ชั่วคราว
            let tempDir = FileManager.default.temporaryDirectory
            let fileName = url.lastPathComponent
            let destinationURL = tempDir.appendingPathComponent(fileName)
            
            try? FileManager.default.moveItem(at: tempURL, to: destinationURL)
            
            if let attachment = try? UNNotificationAttachment(
                identifier: "image",
                url: destinationURL,
                options: nil
            ) {
                content.attachments = [attachment]
            }
            
            completion()
        }.resume()
    }
}
```

### Notification Content Extension

Content Extension ใช้แสดง custom UI สำหรับ notification

```swift
// NotificationViewController.swift - ใน Notification Content Extension target
import UIKit
import UserNotifications
import UserNotificationsUI

class NotificationViewController: UIViewController, UNNotificationContentExtension {
    
    @IBOutlet var titleLabel: UILabel!
    @IBOutlet var bodyLabel: UILabel!
    @IBOutlet var progressView: UIProgressView!
    @IBOutlet var imageView: UIImageView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
    }
    
    private func setupUI() {
        view.backgroundColor = .systemBackground
        // Setup custom UI
    }
    
    func didReceive(_ notification: UNNotification) {
        let content = notification.request.content
        
        titleLabel.text = content.title
        bodyLabel.text = content.body
        
        if let progress = content.userInfo["progress"] as? Float {
            progressView.progress = progress
        }
        
        // โหลดรูปจาก attachment
        if let attachment = content.attachments.first {
            if attachment.url.startAccessingSecurityScopedResource() {
                if let imageData = try? Data(contentsOf: attachment.url) {
                    imageView.image = UIImage(data: imageData)
                }
                attachment.url.stopAccessingSecurityScopedResource()
            }
        }
    }
    
    func didReceive(_ response: UNNotificationResponse,
                   completionHandler completion: @escaping (UNNotificationContentExtensionResponseOption) -> Void) {
        switch response.actionIdentifier {
        case "COMPLETE":
            // จัดการ complete action
            progressView.progress = 1.0
            completion(.doNotDismiss)
            
        case "VIEW":
            // เปิดแอป
            completion(.dismissAndForwardAction)
            
        default:
            completion(.dismiss)
        }
    }
}
```

---

## 34.20 Badge Count

```swift
import UIKit
import UserNotifications

class BadgeManager {
    static let shared = BadgeManager()
    
    private init() {}
    
    // ตั้งค่า badge
    var badgeCount: Int {
        get { UIApplication.shared.applicationIconBadgeNumber }
        set {
            DispatchQueue.main.async {
                UIApplication.shared.applicationIconBadgeNumber = max(0, newValue)
            }
        }
    }
    
    // เพิ่ม badge
    func increment(by count: Int = 1) {
        badgeCount += count
    }
    
    // ลด badge
    func decrement(by count: Int = 1) {
        badgeCount = max(0, badgeCount - count)
    }
    
    // ล้าง badge
    func clear() {
        badgeCount = 0
    }
    
    // ตั้ง badge ใน notification content
    func setBadgeInContent(_ content: UNMutableNotificationContent, count: Int) {
        content.badge = NSNumber(value: count)
    }
    
    // อัพเดต badge อัตโนมัติจาก notifications
    func updateBadgeFromPendingCount() async {
        let pending = await UNUserNotificationCenter.current().pendingNotificationRequests()
        badgeCount = pending.count
    }
}

// SwiftUI Badge Modifier
import SwiftUI

struct BadgeModifier: ViewModifier {
    @ObservedObject var badgeManager: BadgeObserver
    
    func body(content: Content) -> some View {
        content
            .badge(badgeManager.count)
    }
}

class BadgeObserver: ObservableObject {
    @Published var count: Int = 0
    
    init() {
        NotificationCenter.default.addObserver(
            forName: Notification.Name("badgeCountChanged"),
            object: nil,
            queue: .main
        ) { [weak self] notification in
            self?.count = notification.userInfo?["count"] as? Int ?? 0
        }
    }
}
```

---

## 34.21 Notification Sound

```swift
import UserNotifications
import AVFoundation

// เสียง notifications
class NotificationSoundManager {
    
    // เสียงเริ่มต้น
    static let defaultSound = UNNotificationSound.default
    
    // เสียงวิกฤต (ดังเสมอ แม้ silent mode)
    static let criticalSound = UNNotificationSound.defaultCritical
    
    // เสียงที่กำหนดเอง (ต้องมีในบันเดิล)
    static func customSound(named name: String) -> UNNotificationSound {
        UNNotificationSound(named: UNNotificationSoundName(rawValue: name))
    }
    
    // เสียงที่กำหนดเองแบบวิกฤต
    static func criticalCustomSound(named name: String, volume: Float = 1.0) -> UNNotificationSound {
        UNNotificationSound.criticalSoundNamed(
            UNNotificationSoundName(rawValue: name),
            withAudioVolume: volume
        )
    }
    
    // ตัวอย่างการใช้งาน
    static func scheduleNotificationWithCustomSound() {
        let content = UNMutableNotificationContent()
        content.title = "แจ้งเตือน"
        content.body = "มีข้อความใหม่"
        
        // ใช้เสียงที่กำหนดเอง (ไฟล์ alert.aiff ในบันเดิล)
        content.sound = customSound(named: "alert.aiff")
        
        let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 5, repeats: false)
        let request = UNNotificationRequest(identifier: UUID().uuidString, content: content, trigger: trigger)
        
        UNUserNotificationCenter.current().add(request, withCompletionHandler: nil)
    }
}
```

---

## 34.22 แบบฝึกหัดพร้อมเฉลย

### แบบฝึกหัดที่ 1: NotificationCenter Message Bus

```swift
import Foundation

// โจทย์: สร้าง type-safe message bus ด้วย NotificationCenter

// เฉลย:

// Protocol สำหรับทุก message
protocol AppMessage {
    static var notificationName: Notification.Name { get }
}

// Messages ต่างๆ
struct UserLoggedInMessage: AppMessage {
    static let notificationName = Notification.Name("UserLoggedIn")
    let userId: String
    let userName: String
    let loginTime: Date
}

struct DataUpdatedMessage: AppMessage {
    static let notificationName = Notification.Name("DataUpdated")
    let source: String
    let itemCount: Int
    let timestamp: Date
}

struct ErrorOccurredMessage: AppMessage {
    static let notificationName = Notification.Name("ErrorOccurred")
    let error: Error
    let context: String
}

// Message Bus
class MessageBus {
    static let shared = MessageBus()
    private let encoder = JSONEncoder()
    private let decoder = JSONDecoder()
    private let messageKey = "message_payload"
    
    private init() {
        encoder.dateEncodingStrategy = .iso8601
        decoder.dateDecodingStrategy = .iso8601
    }
    
    func post<T: AppMessage & Encodable>(_ message: T) {
        var userInfo: [AnyHashable: Any] = [:]
        if let data = try? encoder.encode(message) {
            userInfo[messageKey] = data
        }
        
        NotificationCenter.default.post(
            name: T.notificationName,
            object: nil,
            userInfo: userInfo
        )
    }
    
    @discardableResult
    func observe<T: AppMessage & Decodable>(
        _ type: T.Type,
        handler: @escaping (T) -> Void
    ) -> NSObjectProtocol {
        NotificationCenter.default.addObserver(
            forName: T.notificationName,
            object: nil,
            queue: .main
        ) { [weak self] notification in
            guard let self = self,
                  let data = notification.userInfo?[self.messageKey] as? Data,
                  let message = try? self.decoder.decode(T.self, from: data) else {
                return
            }
            handler(message)
        }
    }
}

// การใช้งาน
let bus = MessageBus.shared

// Observer
let token = bus.observe(UserLoggedInMessage.self) { message in
    print("User logged in: \(message.userName) at \(message.loginTime)")
}

// Post
bus.post(UserLoggedInMessage(
    userId: "123",
    userName: "Alice",
    loginTime: Date()
))

bus.post(DataUpdatedMessage(
    source: "API",
    itemCount: 42,
    timestamp: Date()
))
```

### แบบฝึกหัดที่ 2: Local Notification System

```swift
import UserNotifications

// โจทย์: สร้าง Notification System ที่สมบูรณ์

struct NotificationItem: Codable, Identifiable {
    var id: String
    var title: String
    var message: String
    var scheduledDate: Date
    var isRecurring: Bool
    var recurringInterval: RecurringInterval
    var category: String
    var isActive: Bool
    
    enum RecurringInterval: String, Codable, CaseIterable {
        case none = "none"
        case daily = "daily"
        case weekly = "weekly"
        case monthly = "monthly"
        
        var displayName: String {
            switch self {
            case .none: return "ไม่วนซ้ำ"
            case .daily: return "ทุกวัน"
            case .weekly: return "ทุกสัปดาห์"
            case .monthly: return "ทุกเดือน"
            }
        }
    }
    
    init(title: String, message: String, scheduledDate: Date, recurringInterval: RecurringInterval = .none, category: String = "DEFAULT") {
        self.id = UUID().uuidString
        self.title = title
        self.message = message
        self.scheduledDate = scheduledDate
        self.isRecurring = recurringInterval != .none
        self.recurringInterval = recurringInterval
        self.category = category
        self.isActive = true
    }
}

class NotificationSystem {
    static let shared = NotificationSystem()
    
    private let center = UNUserNotificationCenter.current()
    private var notifications: [NotificationItem] = []
    private let storageKey = "scheduledNotifications"
    
    private init() {
        loadNotifications()
    }
    
    // บันทึกลง UserDefaults
    private func saveNotifications() {
        if let data = try? JSONEncoder().encode(notifications) {
            UserDefaults.standard.set(data, forKey: storageKey)
        }
    }
    
    private func loadNotifications() {
        guard let data = UserDefaults.standard.data(forKey: storageKey),
              let items = try? JSONDecoder().decode([NotificationItem].self, from: data) else {
            return
        }
        notifications = items
    }
    
    // เพิ่มและ schedule
    func add(_ item: NotificationItem) async throws {
        let content = UNMutableNotificationContent()
        content.title = item.title
        content.body = item.message
        content.sound = .default
        content.categoryIdentifier = item.category
        content.userInfo = ["notificationID": item.id]
        
        let trigger: UNNotificationTrigger
        
        switch item.recurringInterval {
        case .none:
            let components = Calendar.current.dateComponents(
                [.year, .month, .day, .hour, .minute],
                from: item.scheduledDate
            )
            trigger = UNCalendarNotificationTrigger(dateMatching: components, repeats: false)
            
        case .daily:
            let components = Calendar.current.dateComponents(
                [.hour, .minute],
                from: item.scheduledDate
            )
            trigger = UNCalendarNotificationTrigger(dateMatching: components, repeats: true)
            
        case .weekly:
            let components = Calendar.current.dateComponents(
                [.weekday, .hour, .minute],
                from: item.scheduledDate
            )
            trigger = UNCalendarNotificationTrigger(dateMatching: components, repeats: true)
            
        case .monthly:
            let components = Calendar.current.dateComponents(
                [.day, .hour, .minute],
                from: item.scheduledDate
            )
            trigger = UNCalendarNotificationTrigger(dateMatching: components, repeats: true)
        }
        
        let request = UNNotificationRequest(
            identifier: item.id,
            content: content,
            trigger: trigger
        )
        
        try await center.add(request)
        
        var mutableItem = item
        mutableItem.isActive = true
        notifications.append(mutableItem)
        saveNotifications()
    }
    
    // ยกเลิก
    func cancel(id: String) {
        center.removePendingNotificationRequests(withIdentifiers: [id])
        center.removeDeliveredNotifications(withIdentifiers: [id])
        
        if let index = notifications.firstIndex(where: { $0.id == id }) {
            notifications[index].isActive = false
            saveNotifications()
        }
    }
    
    // ลบ
    func delete(id: String) {
        cancel(id: id)
        notifications.removeAll { $0.id == id }
        saveNotifications()
    }
    
    // รายการทั้งหมด
    var allNotifications: [NotificationItem] { notifications }
    
    // รายการที่ active
    var activeNotifications: [NotificationItem] {
        notifications.filter { $0.isActive }
    }
    
    // Sync กับ system
    func syncWithSystem() async {
        let pending = await center.pendingNotificationRequests()
        let pendingIDs = Set(pending.map { $0.identifier })
        
        for i in 0..<notifications.count {
            notifications[i].isActive = pendingIDs.contains(notifications[i].id)
        }
        saveNotifications()
    }
}
```

---

## 34.23 การสร้าง Reminder App

```swift
import SwiftUI
import UserNotifications

// MARK: - Models

struct Reminder: Identifiable, Codable {
    var id: String
    var title: String
    var notes: String
    var dueDate: Date
    var isCompleted: Bool
    var priority: Priority
    var notificationID: String?
    
    enum Priority: Int, Codable, CaseIterable {
        case low = 0, medium = 1, high = 2
        
        var displayName: String {
            switch self {
            case .low: return "ต่ำ"
            case .medium: return "กลาง"
            case .high: return "สูง"
            }
        }
        
        var color: Color {
            switch self {
            case .low: return .green
            case .medium: return .orange
            case .high: return .red
            }
        }
    }
    
    init(title: String, notes: String = "", dueDate: Date = Date(), priority: Priority = .medium) {
        self.id = UUID().uuidString
        self.title = title
        self.notes = notes
        self.dueDate = dueDate
        self.isCompleted = false
        self.priority = priority
    }
}

// MARK: - Repository

class ReminderRepository: ObservableObject {
    @Published var reminders: [Reminder] = []
    
    private let key = "reminders"
    private let center = UNUserNotificationCenter.current()
    
    init() {
        load()
        Task { await requestPermission() }
    }
    
    private func load() {
        if let data = UserDefaults.standard.data(forKey: key),
           let saved = try? JSONDecoder().decode([Reminder].self, from: data) {
            reminders = saved
        }
    }
    
    private func save() {
        if let data = try? JSONEncoder().encode(reminders) {
            UserDefaults.standard.set(data, forKey: key)
        }
    }
    
    private func requestPermission() async {
        _ = try? await center.requestAuthorization(options: [.alert, .badge, .sound])
    }
    
    func add(_ reminder: Reminder) async {
        var mutableReminder = reminder
        
        // Schedule notification
        let content = UNMutableNotificationContent()
        content.title = reminder.title
        content.body = reminder.notes.isEmpty ? "ถึงเวลาแล้ว" : reminder.notes
        content.sound = .default
        content.categoryIdentifier = "REMINDER"
        content.userInfo = ["reminderID": reminder.id]
        
        if reminder.priority == .high {
            content.interruptionLevel = .timeSensitive
        }
        
        let components = Calendar.current.dateComponents(
            [.year, .month, .day, .hour, .minute],
            from: reminder.dueDate
        )
        let trigger = UNCalendarNotificationTrigger(dateMatching: components, repeats: false)
        
        let notifID = "reminder-\(reminder.id)"
        let request = UNNotificationRequest(identifier: notifID, content: content, trigger: trigger)
        
        if let _ = try? await center.add(request) {
            mutableReminder.notificationID = notifID
        }
        
        await MainActor.run {
            reminders.append(mutableReminder)
            reminders.sort { $0.dueDate < $1.dueDate }
            save()
        }
    }
    
    func toggle(_ reminder: Reminder) {
        if let index = reminders.firstIndex(where: { $0.id == reminder.id }) {
            reminders[index].isCompleted.toggle()
            
            if reminders[index].isCompleted, let notifID = reminders[index].notificationID {
                center.removePendingNotificationRequests(withIdentifiers: [notifID])
            }
            save()
        }
    }
    
    func delete(_ reminder: Reminder) {
        if let notifID = reminder.notificationID {
            center.removePendingNotificationRequests(withIdentifiers: [notifID])
        }
        reminders.removeAll { $0.id == reminder.id }
        save()
    }
    
    func deleteCompleted() {
        let completedIDs = reminders.filter { $0.isCompleted }.compactMap { $0.notificationID }
        center.removePendingNotificationRequests(withIdentifiers: completedIDs)
        reminders.removeAll { $0.isCompleted }
        save()
    }
    
    var pendingReminders: [Reminder] {
        reminders.filter { !$0.isCompleted }
    }
    
    var completedReminders: [Reminder] {
        reminders.filter { $0.isCompleted }
    }
    
    var overdueReminders: [Reminder] {
        reminders.filter { !$0.isCompleted && $0.dueDate < Date() }
    }
}

// MARK: - Views

struct ReminderAppView: View {
    @StateObject private var repo = ReminderRepository()
    @State private var showingAddSheet = false
    @State private var selectedFilter: ReminderFilter = .all
    
    enum ReminderFilter: String, CaseIterable {
        case all = "ทั้งหมด"
        case pending = "รอดำเนินการ"
        case completed = "เสร็จแล้ว"
        case overdue = "เกินกำหนด"
    }
    
    var filteredReminders: [Reminder] {
        switch selectedFilter {
        case .all: return repo.reminders
        case .pending: return repo.pendingReminders
        case .completed: return repo.completedReminders
        case .overdue: return repo.overdueReminders
        }
    }
    
    var body: some View {
        NavigationView {
            VStack(spacing: 0) {
                // Filter Picker
                Picker("Filter", selection: $selectedFilter) {
                    ForEach(ReminderFilter.allCases, id: \.self) { filter in
                        Text(filter.rawValue).tag(filter)
                    }
                }
                .pickerStyle(.segmented)
                .padding()
                
                // Statistics
                statsRow
                    .padding(.horizontal)
                
                // List
                if filteredReminders.isEmpty {
                    emptyState
                } else {
                    List {
                        ForEach(filteredReminders) { reminder in
                            ReminderRow(reminder: reminder) {
                                repo.toggle(reminder)
                            }
                        }
                        .onDelete { indexSet in
                            for index in indexSet {
                                repo.delete(filteredReminders[index])
                            }
                        }
                    }
                }
            }
            .navigationTitle("Reminders")
            .toolbar {
                ToolbarItem(placement: .navigationBarLeading) {
                    if !repo.completedReminders.isEmpty {
                        Button("ล้างที่เสร็จ") {
                            repo.deleteCompleted()
                        }
                        .foregroundColor(.red)
                    }
                }
                
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button {
                        showingAddSheet = true
                    } label: {
                        Image(systemName: "plus")
                    }
                }
            }
            .sheet(isPresented: $showingAddSheet) {
                AddReminderView { reminder in
                    Task { await repo.add(reminder) }
                }
            }
        }
    }
    
    private var statsRow: some View {
        HStack(spacing: 16) {
            StatBadge(
                count: repo.pendingReminders.count,
                label: "รอดำเนินการ",
                color: .blue
            )
            StatBadge(
                count: repo.overdueReminders.count,
                label: "เกินกำหนด",
                color: .red
            )
            StatBadge(
                count: repo.completedReminders.count,
                label: "เสร็จแล้ว",
                color: .green
            )
        }
        .padding(.vertical, 8)
    }
    
    private var emptyState: some View {
        VStack(spacing: 16) {
            Spacer()
            Image(systemName: "checkmark.circle")
                .font(.system(size: 60))
                .foregroundColor(.secondary)
            Text("ไม่มีรายการ")
                .font(.title2)
                .foregroundColor(.secondary)
            Spacer()
        }
    }
}

struct ReminderRow: View {
    let reminder: Reminder
    let onToggle: () -> Void
    
    var isOverdue: Bool {
        !reminder.isCompleted && reminder.dueDate < Date()
    }
    
    var body: some View {
        HStack(spacing: 12) {
            Button(action: onToggle) {
                Image(systemName: reminder.isCompleted ? "checkmark.circle.fill" : "circle")
                    .font(.title2)
                    .foregroundColor(reminder.isCompleted ? .green : .secondary)
            }
            .buttonStyle(.plain)
            
            VStack(alignment: .leading, spacing: 4) {
                Text(reminder.title)
                    .strikethrough(reminder.isCompleted)
                    .foregroundColor(reminder.isCompleted ? .secondary : .primary)
                
                if !reminder.notes.isEmpty {
                    Text(reminder.notes)
                        .font(.caption)
                        .foregroundColor(.secondary)
                        .lineLimit(1)
                }
                
                HStack {
                    Image(systemName: "clock")
                        .font(.caption2)
                    Text(reminder.dueDate.formatted(date: .abbreviated, time: .shortened))
                        .font(.caption)
                }
                .foregroundColor(isOverdue ? .red : .secondary)
            }
            
            Spacer()
            
            Circle()
                .fill(reminder.priority.color)
                .frame(width: 8, height: 8)
        }
        .padding(.vertical, 4)
        .background(
            RoundedRectangle(cornerRadius: 8)
                .fill(isOverdue ? Color.red.opacity(0.05) : Color.clear)
        )
    }
}

struct StatBadge: View {
    let count: Int
    let label: String
    let color: Color
    
    var body: some View {
        VStack(spacing: 4) {
            Text("\(count)")
                .font(.title2)
                .fontWeight(.bold)
                .foregroundColor(color)
            Text(label)
                .font(.caption2)
                .foregroundColor(.secondary)
        }
        .frame(maxWidth: .infinity)
        .padding(.vertical, 8)
        .background(color.opacity(0.1))
        .cornerRadius(8)
    }
}

struct AddReminderView: View {
    @Environment(\.dismiss) private var dismiss
    let onAdd: (Reminder) -> Void
    
    @State private var title = ""
    @State private var notes = ""
    @State private var dueDate = Date().addingTimeInterval(3600)
    @State private var priority: Reminder.Priority = .medium
    
    var isValid: Bool { !title.trimmingCharacters(in: .whitespaces).isEmpty }
    
    var body: some View {
        NavigationView {
            Form {
                Section("รายละเอียด") {
                    TextField("ชื่องาน", text: $title)
                    TextField("หมายเหตุ (ไม่บังคับ)", text: $notes, axis: .vertical)
                        .lineLimit(3)
                }
                
                Section("กำหนดส่ง") {
                    DatePicker(
                        "วันและเวลา",
                        selection: $dueDate,
                        in: Date()...,
                        displayedComponents: [.date, .hourAndMinute]
                    )
                }
                
                Section("ความสำคัญ") {
                    Picker("ระดับ", selection: $priority) {
                        ForEach(Reminder.Priority.allCases, id: \.self) { p in
                            HStack {
                                Circle()
                                    .fill(p.color)
                                    .frame(width: 10, height: 10)
                                Text(p.displayName)
                            }.tag(p)
                        }
                    }
                    .pickerStyle(.segmented)
                }
            }
            .navigationTitle("เพิ่ม Reminder")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .cancellationAction) {
                    Button("ยกเลิก") { dismiss() }
                }
                ToolbarItem(placement: .confirmationAction) {
                    Button("เพิ่ม") {
                        let reminder = Reminder(
                            title: title,
                            notes: notes,
                            dueDate: dueDate,
                            priority: priority
                        )
                        onAdd(reminder)
                        dismiss()
                    }
                    .disabled(!isValid)
                }
            }
        }
    }
}
```

---

## 34.24 สรุป

ในบทนี้เราได้เรียนรู้เกี่ยวกับ Notifications อย่างครอบคลุม:

### NotificationCenter
- ระบบ messaging ภายในแอปแบบ Observer pattern
- ใช้ `post(name:object:userInfo:)` ส่ง notifications
- ใช้ `addObserver(forName:object:queue:using:)` รับ notifications
- ต้อง remove observer เสมอเพื่อป้องกัน memory leak
- รองรับ Combine และ async/await

### System Notifications
- App lifecycle notifications (didBecomeActive, willResignActive ฯลฯ)
- Keyboard notifications
- Memory warnings
- Device state changes

### UNUserNotificationCenter
- ต้องขอ permission ก่อนใช้งาน
- รองรับ 3 trigger types: Time Interval, Calendar, Location
- สามารถกำหนด Actions และ Categories
- รองรับ rich content ด้วย attachments
- สามารถ extend ด้วย Service Extension และ Content Extension

### Best Practices
- ตรวจสอบ permission status ก่อนใช้งานเสมอ
- ใช้ unique identifiers สำหรับ notifications
- จัดการ notification responses ใน AppDelegate
- ระวัง battery impact จาก location triggers
- ใช้ `.timeSensitive` สำหรับ notifications สำคัญ

### สิ่งที่ต้องระวัง
- ไม่สามารถ schedule notifications ที่เกิน 64 รายการพร้อมกัน
- Location notifications ต้องการ always location permission
- ไม่ overload ผู้ใช้ด้วย notifications มากเกินไป
- Test บน real device สำหรับ push notifications

---

*บทต่อไป: Part 35 - CoreData และ SwiftData*
