# Part 61: Push Notifications ขั้นสูง (Advanced Push Notifications)

## บทนำ

Push Notifications เป็นหนึ่งในฟีเจอร์ที่สำคัญที่สุดสำหรับแอปพลิเคชัน iOS ที่ช่วยให้แอปสามารถสื่อสารกับผู้ใช้ได้แม้กระทั่งเมื่อแอปไม่ได้ทำงานอยู่เบื้องหน้า ในบทนี้เราจะเรียนรู้ทุกแง่มุมของ Push Notifications ตั้งแต่พื้นฐานไปจนถึงเทคนิคขั้นสูง

---

## 61.1 APNs (Apple Push Notification service) ภาพรวม

### APNs คืออะไร?

APNs (Apple Push Notification service) เป็นบริการของ Apple ที่ทำหน้าที่เป็น Gateway สำหรับการส่ง Push Notifications จาก Server ของนักพัฒนาไปยังอุปกรณ์ Apple

### สถาปัตยกรรมของ APNs

```
┌─────────────┐    ┌──────────┐    ┌─────────────────┐    ┌──────────────┐
│  Your       │    │  APNs    │    │  Apple Device   │    │  Your App    │
│  Server     │───▶│  Gateway │───▶│  (iPhone/iPad)  │───▶│  (Receives   │
│             │    │          │    │                 │    │   Notification)│
└─────────────┘    └──────────┘    └─────────────────┘    └──────────────┘
```

การทำงานของ APNs มีขั้นตอนดังนี้:

1. **App Registration**: แอปลงทะเบียนกับ APNs เพื่อรับ Device Token
2. **Token Storage**: Device Token ถูกส่งไปยัง Server ของแอป
3. **Push Request**: Server ส่ง Notification Request ไปยัง APNs พร้อม Device Token
4. **Delivery**: APNs ส่ง Notification ไปยังอุปกรณ์ที่ระบุ
5. **Display**: ระบบแสดง Notification ให้ผู้ใช้เห็น

### APNs Endpoints

APNs มี Endpoint สองแบบ:

```
Production: api.push.apple.com
Development: api.sandbox.push.apple.com
```

### ข้อกำหนดของ APNs

- ต้องใช้ HTTP/2 protocol
- ต้องมี Certificate หรือ Token-based authentication
- มีขนาดสูงสุดของ Payload 4KB
- รองรับ Priority levels: 10 (immediate) และ 5 (power-saving)

---

## 61.2 การลงทะเบียนสำหรับ Remote Notifications

### การขอ Permission

ก่อนที่จะสามารถรับ Push Notifications ได้ แอปต้องขอ Permission จากผู้ใช้

```swift
import UserNotifications
import UIKit

class AppDelegate: UIResponder, UIApplicationDelegate {
    
    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        
        // ขอ Permission สำหรับ Notifications
        requestNotificationPermission()
        
        return true
    }
    
    func requestNotificationPermission() {
        let center = UNUserNotificationCenter.current()
        
        // กำหนด Options ที่ต้องการ
        let options: UNAuthorizationOptions = [.alert, .sound, .badge]
        
        center.requestAuthorization(options: options) { granted, error in
            DispatchQueue.main.async {
                if granted {
                    print("✅ Permission granted for notifications")
                    // ลงทะเบียนสำหรับ Remote Notifications
                    UIApplication.shared.registerForRemoteNotifications()
                } else if let error = error {
                    print("❌ Error requesting permission: \(error.localizedDescription)")
                } else {
                    print("⚠️ Permission denied by user")
                }
            }
        }
    }
}
```

### การตรวจสอบสถานะ Authorization

```swift
func checkNotificationAuthorizationStatus() {
    UNUserNotificationCenter.current().getNotificationSettings { settings in
        switch settings.authorizationStatus {
        case .authorized:
            print("✅ Authorized")
        case .denied:
            print("❌ Denied")
        case .notDetermined:
            print("❓ Not Determined")
        case .provisional:
            print("🔔 Provisional")
        case .ephemeral:
            print("⏱️ Ephemeral")
        @unknown default:
            print("Unknown status")
        }
        
        // ตรวจสอบ settings เพิ่มเติม
        print("Alert: \(settings.alertSetting.rawValue)")
        print("Sound: \(settings.soundSetting.rawValue)")
        print("Badge: \(settings.badgeSetting.rawValue)")
        print("Lock Screen: \(settings.lockScreenSetting.rawValue)")
        print("Notification Center: \(settings.notificationCenterSetting.rawValue)")
    }
}
```

### การจัดการ Registration ใน AppDelegate

```swift
// AppDelegate.swift
extension AppDelegate {
    
    // เรียกเมื่อลงทะเบียนสำเร็จ
    func application(_ application: UIApplication,
                     didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
        
        // แปลง Token เป็น String
        let tokenString = deviceToken.map { String(format: "%02.2hhx", $0) }.joined()
        print("📱 Device Token: \(tokenString)")
        
        // ส่ง Token ไปยัง Server
        sendDeviceTokenToServer(token: tokenString)
    }
    
    // เรียกเมื่อลงทะเบียนล้มเหลว
    func application(_ application: UIApplication,
                     didFailToRegisterForRemoteNotificationsWithError error: Error) {
        print("❌ Failed to register for remote notifications: \(error)")
    }
    
    private func sendDeviceTokenToServer(token: String) {
        // ส่ง Token ไปยัง Backend Server
        guard let url = URL(string: "https://yourserver.com/api/register-device") else { return }
        
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        
        let body: [String: Any] = [
            "device_token": token,
            "platform": "ios",
            "user_id": getCurrentUserId()
        ]
        
        request.httpBody = try? JSONSerialization.data(withJSONObject: body)
        
        URLSession.shared.dataTask(with: request) { data, response, error in
            if let error = error {
                print("❌ Error sending token: \(error)")
                return
            }
            print("✅ Token sent successfully")
        }.resume()
    }
    
    private func getCurrentUserId() -> String {
        // ดึง User ID จาก Storage
        return UserDefaults.standard.string(forKey: "userId") ?? ""
    }
}
```

---

## 61.3 Device Token

### Device Token คืออะไร?

Device Token เป็น Unique Identifier ที่ APNs สร้างขึ้นสำหรับการระบุอุปกรณ์และแอป Token มีลักษณะเป็น 32 bytes (256-bit) และอาจเปลี่ยนแปลงได้ในบางสถานการณ์

### เมื่อไหร่ที่ Token เปลี่ยนแปลง?

1. เมื่อ User ติดตั้งแอปบนอุปกรณ์ใหม่
2. เมื่อ User Restore อุปกรณ์จาก Backup
3. เมื่อ User ติดตั้ง OS ใหม่
4. เมื่อ User ลบและติดตั้งแอปใหม่

### การจัดการ Device Token อย่างถูกต้อง

```swift
class DeviceTokenManager {
    static let shared = DeviceTokenManager()
    private let userDefaults = UserDefaults.standard
    private let tokenKey = "deviceToken"
    
    // บันทึก Token ใหม่
    func saveToken(_ tokenData: Data) {
        let tokenString = tokenData.hexString
        let savedToken = userDefaults.string(forKey: tokenKey)
        
        if tokenString != savedToken {
            userDefaults.set(tokenString, forKey: tokenKey)
            // Token เปลี่ยนแปลง - อัปเดต Server
            updateServerWithNewToken(tokenString)
        }
    }
    
    // ดึง Token ที่บันทึกไว้
    var currentToken: String? {
        return userDefaults.string(forKey: tokenKey)
    }
    
    private func updateServerWithNewToken(_ token: String) {
        print("🔄 Updating server with new token: \(token)")
        // ส่ง API Call ไปยัง Server
    }
}

// Extension สำหรับแปลง Data เป็น Hex String
extension Data {
    var hexString: String {
        return map { String(format: "%02.2hhx", $0) }.joined()
    }
}
```

### การจัดการ Token ใน SwiftUI

```swift
// SwiftUI App
@main
struct PushNotificationApp: App {
    @UIApplicationDelegateAdaptor(AppDelegate.self) var appDelegate
    
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}

// ViewModel สำหรับจัดการ Notification
class NotificationViewModel: ObservableObject {
    @Published var deviceToken: String = ""
    @Published var authorizationStatus: UNAuthorizationStatus = .notDetermined
    
    init() {
        checkAuthorizationStatus()
    }
    
    func requestPermission() {
        UNUserNotificationCenter.current()
            .requestAuthorization(options: [.alert, .sound, .badge]) { [weak self] granted, _ in
                DispatchQueue.main.async {
                    if granted {
                        UIApplication.shared.registerForRemoteNotifications()
                        self?.checkAuthorizationStatus()
                    }
                }
            }
    }
    
    func checkAuthorizationStatus() {
        UNUserNotificationCenter.current().getNotificationSettings { [weak self] settings in
            DispatchQueue.main.async {
                self?.authorizationStatus = settings.authorizationStatus
            }
        }
    }
}
```

---

## 61.4 การส่ง Test Notifications

### การส่ง Notification ผ่าน Terminal

สามารถใช้ `curl` เพื่อส่ง Test Notification โดยตรงไปยัง APNs:

```bash
# ใช้ Token-based Auth (APNs Auth Key)
curl -v \
  --header "apns-topic: com.yourcompany.yourapp" \
  --header "apns-push-type: alert" \
  --header "authorization: bearer $JWT_TOKEN" \
  --data '{"aps":{"alert":{"title":"Test","body":"Hello from curl!"},"sound":"default"}}' \
  --http2 \
  https://api.sandbox.push.apple.com/3/device/$DEVICE_TOKEN
```

### การสร้าง JWT Token สำหรับ APNs

```swift
import CryptoKit
import Foundation

class APNsJWTGenerator {
    
    struct JWTHeader: Codable {
        let alg: String
        let kid: String
    }
    
    struct JWTClaims: Codable {
        let iss: String  // Team ID
        let iat: Int     // Issued at time
    }
    
    static func generateJWT(
        teamID: String,
        keyID: String,
        privateKeyString: String
    ) throws -> String {
        
        let header = JWTHeader(alg: "ES256", kid: keyID)
        let claims = JWTClaims(iss: teamID, iat: Int(Date().timeIntervalSince1970))
        
        let encoder = JSONEncoder()
        let headerData = try encoder.encode(header)
        let claimsData = try encoder.encode(claims)
        
        let headerBase64 = headerData.base64URLEncoded
        let claimsBase64 = claimsData.base64URLEncoded
        
        let signingInput = "\(headerBase64).\(claimsBase64)"
        
        // Sign with P256 Private Key
        let privateKey = try P256.Signing.PrivateKey(pemRepresentation: privateKeyString)
        let signature = try privateKey.signature(for: signingInput.data(using: .utf8)!)
        let signatureBase64 = signature.derRepresentation.base64URLEncoded
        
        return "\(signingInput).\(signatureBase64)"
    }
}

extension Data {
    var base64URLEncoded: String {
        return base64EncodedString()
            .replacingOccurrences(of: "+", with: "-")
            .replacingOccurrences(of: "/", with: "_")
            .replacingOccurrences(of: "=", with: "")
    }
}
```

### Xcode Simulator Testing

ตั้งแต่ Xcode 11.4 สามารถส่ง Notification ไปยัง Simulator ได้:

```bash
# ส่ง Notification ไปยัง Simulator
xcrun simctl push booted com.yourapp.bundle payload.json

# สร้างไฟล์ payload.json
cat > payload.json << EOF
{
    "aps": {
        "alert": {
            "title": "Test Notification",
            "body": "This is a test from Simulator"
        },
        "sound": "default",
        "badge": 1
    }
}
EOF
```

---

## 61.5 Notification Payload Structure

### โครงสร้าง Payload พื้นฐาน

```json
{
    "aps": {
        "alert": {
            "title": "หัวข้อ Notification",
            "subtitle": "หัวข้อย่อย",
            "body": "เนื้อหาของ Notification",
            "launch-image": "Default.png",
            "title-loc-key": "TITLE_KEY",
            "title-loc-args": ["arg1", "arg2"],
            "loc-key": "BODY_KEY",
            "loc-args": ["arg1"]
        },
        "sound": "default",
        "badge": 5,
        "thread-id": "conversation-123",
        "category": "MESSAGE_CATEGORY",
        "content-available": 1,
        "mutable-content": 1,
        "target-content-id": "widget-123",
        "interruption-level": "active",
        "relevance-score": 0.8,
        "filter-criteria": "filter-value"
    },
    "custom_key": "custom_value",
    "data": {
        "screen": "chat",
        "conversation_id": "abc123"
    }
}
```

### Payload สำหรับ Silent Notification

```json
{
    "aps": {
        "content-available": 1
    },
    "type": "data_sync",
    "sync_timestamp": "2024-01-01T00:00:00Z"
}
```

### Payload สำหรับ Rich Notification

```json
{
    "aps": {
        "alert": {
            "title": "New Message",
            "body": "You have a new message"
        },
        "mutable-content": 1,
        "sound": "message.caf"
    },
    "media_url": "https://example.com/image.jpg",
    "media_type": "image",
    "action_url": "yourapp://message/123"
}
```

### Swift Model สำหรับ Payload

```swift
struct NotificationPayload: Codable {
    let aps: APSPayload
    let customData: [String: AnyCodable]?
    
    struct APSPayload: Codable {
        let alert: AlertPayload?
        let sound: String?
        let badge: Int?
        let category: String?
        let threadId: String?
        let contentAvailable: Int?
        let mutableContent: Int?
        let interruptionLevel: String?
        let relevanceScore: Double?
        
        enum CodingKeys: String, CodingKey {
            case alert, sound, badge, category
            case threadId = "thread-id"
            case contentAvailable = "content-available"
            case mutableContent = "mutable-content"
            case interruptionLevel = "interruption-level"
            case relevanceScore = "relevance-score"
        }
    }
    
    struct AlertPayload: Codable {
        let title: String?
        let subtitle: String?
        let body: String?
    }
}

// AnyCodable สำหรับ Dynamic JSON Values
struct AnyCodable: Codable {
    let value: Any
    
    init(_ value: Any) {
        self.value = value
    }
    
    func encode(to encoder: Encoder) throws {
        var container = encoder.singleValueContainer()
        
        if let intValue = value as? Int {
            try container.encode(intValue)
        } else if let doubleValue = value as? Double {
            try container.encode(doubleValue)
        } else if let stringValue = value as? String {
            try container.encode(stringValue)
        } else if let boolValue = value as? Bool {
            try container.encode(boolValue)
        }
    }
    
    init(from decoder: Decoder) throws {
        let container = try decoder.singleValueContainer()
        
        if let intValue = try? container.decode(Int.self) {
            value = intValue
        } else if let doubleValue = try? container.decode(Double.self) {
            value = doubleValue
        } else if let stringValue = try? container.decode(String.self) {
            value = stringValue
        } else if let boolValue = try? container.decode(Bool.self) {
            value = boolValue
        } else {
            value = ""
        }
    }
}
```

---

## 61.6 Notification Content Types

### Alert Notifications

Notification ที่แสดงข้อความแจ้งเตือน:

```swift
// การสร้าง Local Notification แบบ Alert
func scheduleAlertNotification() {
    let content = UNMutableNotificationContent()
    content.title = "ข้อความใหม่"
    content.subtitle = "จาก John Doe"
    content.body = "สวัสดีครับ! คุณว่างไหม?"
    content.sound = .default
    content.badge = 1
    
    // เพิ่ม Thread Identifier เพื่อจัดกลุ่ม
    content.threadIdentifier = "chat-john-doe"
    
    // กำหนด Target สำหรับ iPad Split View
    content.targetContentIdentifier = "chat-screen"
    
    let trigger = UNTimeIntervalNotificationTrigger(
        timeInterval: 5,
        repeats: false
    )
    
    let request = UNNotificationRequest(
        identifier: UUID().uuidString,
        content: content,
        trigger: trigger
    )
    
    UNUserNotificationCenter.current().add(request) { error in
        if let error = error {
            print("Error: \(error)")
        }
    }
}
```

### Badge Notifications

การอัปเดต Badge บน App Icon:

```swift
// อัปเดต Badge
func updateBadge(count: Int) {
    if #available(iOS 16.0, *) {
        UNUserNotificationCenter.current().setBadgeCount(count) { error in
            if let error = error {
                print("Error setting badge: \(error)")
            }
        }
    } else {
        UIApplication.shared.applicationIconBadgeNumber = count
    }
}

// ล้าง Badge
func clearBadge() {
    updateBadge(count: 0)
}
```

### Sound Notifications

การใช้ Custom Sound:

```swift
func scheduleNotificationWithCustomSound() {
    let content = UNMutableNotificationContent()
    content.title = "Notification พร้อม Custom Sound"
    content.body = "มี Sound พิเศษ!"
    
    // ใช้ Custom Sound (ต้องมีไฟล์ในแอป)
    content.sound = UNNotificationSound(named: UNNotificationSoundName("custom_sound.aiff"))
    
    // หรือใช้ Sound สำหรับ Critical Alert
    if #available(iOS 15.0, *) {
        content.sound = UNNotificationSound.criticalSoundNamed(
            UNNotificationSoundName("critical_sound.aiff"),
            withAudioVolume: 1.0
        )
    }
    
    let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 1, repeats: false)
    let request = UNNotificationRequest(
        identifier: "custom-sound",
        content: content,
        trigger: trigger
    )
    
    UNUserNotificationCenter.current().add(request)
}
```

---

## 61.7 Notification Service Extension

### การสร้าง Notification Service Extension

Notification Service Extension ช่วยให้สามารถแก้ไข Notification Content ก่อนที่จะแสดงให้ผู้ใช้เห็น

**วิธีสร้าง:**
1. ใน Xcode ไปที่ File → New → Target
2. เลือก "Notification Service Extension"
3. ตั้งชื่อ Extension เช่น "NotificationService"

```swift
// NotificationService.swift
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
        
        // แก้ไข Notification Content
        modifyNotificationContent(bestAttemptContent) { modifiedContent in
            contentHandler(modifiedContent)
        }
    }
    
    override func serviceExtensionTimeWillExpire() {
        // เรียกก่อน Time Limit จะหมด (30 วินาที)
        if let contentHandler = contentHandler,
           let bestAttemptContent = bestAttemptContent {
            // ส่ง Content ที่แก้ไขแล้ว (หรือ Content เดิมถ้าไม่ทัน)
            contentHandler(bestAttemptContent)
        }
    }
    
    private func modifyNotificationContent(
        _ content: UNMutableNotificationContent,
        completion: @escaping (UNNotificationContent) -> Void
    ) {
        // ดึง Media URL จาก UserInfo
        guard let mediaURLString = content.userInfo["media_url"] as? String,
              let mediaURL = URL(string: mediaURLString) else {
            completion(content)
            return
        }
        
        // ดาวน์โหลด Media และเพิ่มเป็น Attachment
        downloadMedia(from: mediaURL) { attachment in
            if let attachment = attachment {
                content.attachments = [attachment]
            }
            
            // เพิ่มข้อมูลเพิ่มเติม
            content.title = "[Modified] " + content.title
            
            completion(content)
        }
    }
    
    private func downloadMedia(
        from url: URL,
        completion: @escaping (UNNotificationAttachment?) -> Void
    ) {
        URLSession.shared.downloadTask(with: url) { tempURL, response, error in
            guard let tempURL = tempURL else {
                completion(nil)
                return
            }
            
            // สร้างชื่อไฟล์ที่มี Extension ที่ถูกต้อง
            let fileManager = FileManager.default
            let mimeType = response?.mimeType ?? "image/jpeg"
            let fileExtension = self.fileExtension(for: mimeType)
            let targetURL = tempURL.appendingPathExtension(fileExtension)
            
            do {
                try fileManager.moveItem(at: tempURL, to: targetURL)
                
                let attachment = try UNNotificationAttachment(
                    identifier: "media",
                    url: targetURL,
                    options: nil
                )
                completion(attachment)
            } catch {
                print("Error creating attachment: \(error)")
                completion(nil)
            }
        }.resume()
    }
    
    private func fileExtension(for mimeType: String) -> String {
        switch mimeType {
        case "image/jpeg": return "jpg"
        case "image/png": return "png"
        case "image/gif": return "gif"
        case "video/mp4": return "mp4"
        case "audio/mpeg": return "mp3"
        default: return "jpg"
        }
    }
}
```

---

## 61.8 การแก้ไข Notification Content

### การเพิ่ม Rich Media

```swift
// เพิ่ม Image Attachment
func addImageAttachment(to content: UNMutableNotificationContent, imageURL: URL) async {
    do {
        let (tempURL, _) = try await URLSession.shared.download(from: imageURL)
        
        let attachment = try UNNotificationAttachment(
            identifier: UUID().uuidString,
            url: tempURL,
            options: [
                UNNotificationAttachmentOptionsThumbnailHiddenKey: false,
                UNNotificationAttachmentOptionsThumbnailClippingRectKey: CGRect(x: 0, y: 0, width: 1, height: 1)
            ]
        )
        
        content.attachments.append(attachment)
    } catch {
        print("Failed to add attachment: \(error)")
    }
}

// เพิ่ม Local Image
func addLocalImageAttachment(to content: UNMutableNotificationContent) {
    guard let imageURL = Bundle.main.url(forResource: "notification_image", withExtension: "png") else {
        return
    }
    
    do {
        let attachment = try UNNotificationAttachment(
            identifier: "local-image",
            url: imageURL,
            options: nil
        )
        content.attachments = [attachment]
    } catch {
        print("Error: \(error)")
    }
}
```

### การ Decrypt Notification Content

```swift
// ถอดรหัส Encrypted Notification
class EncryptedNotificationService: UNNotificationServiceExtension {
    
    override func didReceive(
        _ request: UNNotificationRequest,
        withContentHandler contentHandler: @escaping (UNNotificationContent) -> Void
    ) {
        guard let content = request.content.mutableCopy() as? UNMutableNotificationContent,
              let encryptedBody = content.userInfo["encrypted_body"] as? String else {
            contentHandler(request.content)
            return
        }
        
        // ถอดรหัส
        if let decryptedBody = decrypt(encryptedBody) {
            content.body = decryptedBody
        }
        
        contentHandler(content)
    }
    
    private func decrypt(_ encrypted: String) -> String? {
        // ใส่ Logic การถอดรหัสที่นี่
        // ตัวอย่างเช่น AES decryption
        return "Decrypted: \(encrypted)"
    }
}
```

---

## 61.9 Notification Content Extension

### การสร้าง Notification Content Extension

Notification Content Extension ช่วยสร้าง Custom UI สำหรับ Notification

**วิธีสร้าง:**
1. File → New → Target
2. เลือก "Notification Content Extension"
3. ตั้งชื่อ เช่น "NotificationContentExtension"

```swift
// NotificationViewController.swift
import UIKit
import UserNotifications
import UserNotificationsUI

class NotificationViewController: UIViewController, UNNotificationContentExtension {
    
    @IBOutlet weak var titleLabel: UILabel!
    @IBOutlet weak var bodyLabel: UILabel!
    @IBOutlet weak var imageView: UIImageView!
    @IBOutlet weak var progressView: UIProgressView!
    @IBOutlet weak var likeButton: UIButton!
    @IBOutlet weak var shareButton: UIButton!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
    }
    
    func didReceive(_ notification: UNNotification) {
        let content = notification.request.content
        
        titleLabel.text = content.title
        bodyLabel.text = content.body
        
        // โหลด Attachment ถ้ามี
        if let attachment = content.attachments.first {
            loadAttachment(attachment)
        }
        
        // อ่านข้อมูล Custom
        if let progressValue = content.userInfo["progress"] as? Float {
            progressView.progress = progressValue
            progressView.isHidden = false
        }
    }
    
    func didReceive(_ response: UNNotificationResponse,
                    completionHandler completion: @escaping (UNNotificationContentExtensionResponseOption) -> Void) {
        
        switch response.actionIdentifier {
        case "LIKE_ACTION":
            animateLike()
            completion(.doNotDismiss)
            
        case "SHARE_ACTION":
            completion(.dismissAndForwardAction)
            
        default:
            completion(.dismiss)
        }
    }
    
    private func setupUI() {
        view.backgroundColor = .systemBackground
        imageView.contentMode = .scaleAspectFill
        imageView.clipsToBounds = true
        imageView.layer.cornerRadius = 12
        progressView.isHidden = true
    }
    
    private func loadAttachment(_ attachment: UNNotificationAttachment) {
        guard attachment.url.startAccessingSecurityScopedResource() else { return }
        defer { attachment.url.stopAccessingSecurityScopedResource() }
        
        if let imageData = try? Data(contentsOf: attachment.url),
           let image = UIImage(data: imageData) {
            DispatchQueue.main.async {
                self.imageView.image = image
                self.imageView.isHidden = false
            }
        }
    }
    
    private func animateLike() {
        UIView.animate(withDuration: 0.3, animations: {
            self.likeButton.transform = CGAffineTransform(scaleX: 1.3, y: 1.3)
        }) { _ in
            UIView.animate(withDuration: 0.2) {
                self.likeButton.transform = .identity
            }
        }
        
        likeButton.setTitle("❤️ Liked!", for: .normal)
        likeButton.isEnabled = false
    }
}
```

### Info.plist สำหรับ Content Extension

```xml
<!-- NotificationContentExtension/Info.plist -->
<key>NSExtension</key>
<dict>
    <key>NSExtensionAttributes</key>
    <dict>
        <key>UNNotificationExtensionCategory</key>
        <array>
            <string>MESSAGE_CATEGORY</string>
            <string>MEDIA_CATEGORY</string>
        </array>
        <key>UNNotificationExtensionInitialContentSizeRatio</key>
        <real>0.6</real>
        <key>UNNotificationExtensionDefaultContentHidden</key>
        <true/>
        <key>UNNotificationExtensionUserInteractionEnabled</key>
        <true/>
    </dict>
    <key>NSExtensionPrincipalClass</key>
    <string>$(PRODUCT_MODULE_NAME).NotificationViewController</string>
    <key>NSExtensionPointIdentifier</key>
    <string>com.apple.usernotifications.content-extension</string>
</dict>
```

---

## 61.10 Notification Categories and Actions

### การสร้าง Notification Category

```swift
class NotificationCategoryManager {
    
    static func setupCategories() {
        // Actions สำหรับ Message Category
        let replyAction = UNTextInputNotificationAction(
            identifier: "REPLY_ACTION",
            title: "ตอบกลับ",
            options: [],
            textInputButtonTitle: "ส่ง",
            textInputPlaceholder: "พิมพ์ข้อความ..."
        )
        
        let markReadAction = UNNotificationAction(
            identifier: "MARK_READ_ACTION",
            title: "อ่านแล้ว",
            options: []
        )
        
        let deleteAction = UNNotificationAction(
            identifier: "DELETE_ACTION",
            title: "ลบ",
            options: [.destructive]
        )
        
        let messageCategory = UNNotificationCategory(
            identifier: "MESSAGE_CATEGORY",
            actions: [replyAction, markReadAction, deleteAction],
            intentIdentifiers: [],
            options: [.customDismissAction, .hiddenPreviewsShowTitle]
        )
        
        // Actions สำหรับ Order Category
        let trackOrderAction = UNNotificationAction(
            identifier: "TRACK_ORDER",
            title: "ติดตามพัสดุ",
            options: [.foreground]
        )
        
        let cancelOrderAction = UNNotificationAction(
            identifier: "CANCEL_ORDER",
            title: "ยกเลิก",
            options: [.destructive, .authenticationRequired]
        )
        
        let orderCategory = UNNotificationCategory(
            identifier: "ORDER_CATEGORY",
            actions: [trackOrderAction, cancelOrderAction],
            intentIdentifiers: [],
            options: []
        )
        
        // ลงทะเบียน Categories
        UNUserNotificationCenter.current().setNotificationCategories([
            messageCategory,
            orderCategory
        ])
    }
}
```

### การจัดการ Action Response

```swift
extension AppDelegate: UNUserNotificationCenterDelegate {
    
    func userNotificationCenter(
        _ center: UNUserNotificationCenter,
        didReceive response: UNNotificationResponse,
        withCompletionHandler completionHandler: @escaping () -> Void
    ) {
        let userInfo = response.notification.request.content.userInfo
        
        switch response.actionIdentifier {
        case "REPLY_ACTION":
            if let textResponse = response as? UNTextInputNotificationResponse {
                let replyText = textResponse.userText
                handleReply(text: replyText, userInfo: userInfo)
            }
            
        case "MARK_READ_ACTION":
            handleMarkAsRead(userInfo: userInfo)
            
        case "DELETE_ACTION":
            handleDelete(userInfo: userInfo)
            
        case "TRACK_ORDER":
            openOrderTracking(userInfo: userInfo)
            
        case "CANCEL_ORDER":
            showCancelConfirmation(userInfo: userInfo)
            
        case UNNotificationDefaultActionIdentifier:
            // ผู้ใช้แตะ Notification
            handleNotificationTap(userInfo: userInfo)
            
        case UNNotificationDismissActionIdentifier:
            // ผู้ใช้ Dismiss Notification
            handleNotificationDismiss(userInfo: userInfo)
            
        default:
            break
        }
        
        completionHandler()
    }
    
    // แสดง Notification เมื่อแอปทำงานอยู่
    func userNotificationCenter(
        _ center: UNUserNotificationCenter,
        willPresent notification: UNNotification,
        withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void
    ) {
        if #available(iOS 14.0, *) {
            completionHandler([.banner, .sound, .badge, .list])
        } else {
            completionHandler([.alert, .sound, .badge])
        }
    }
    
    private func handleReply(text: String, userInfo: [AnyHashable: Any]) {
        guard let messageId = userInfo["message_id"] as? String else { return }
        print("Replying to message \(messageId): \(text)")
        // ส่ง Reply ผ่าน API
    }
    
    private func handleMarkAsRead(userInfo: [AnyHashable: Any]) {
        guard let messageId = userInfo["message_id"] as? String else { return }
        print("Marking message \(messageId) as read")
    }
    
    private func handleDelete(userInfo: [AnyHashable: Any]) {
        print("Deleting notification item")
    }
    
    private func openOrderTracking(userInfo: [AnyHashable: Any]) {
        guard let orderId = userInfo["order_id"] as? String else { return }
        NotificationCenter.default.post(
            name: NSNotification.Name("OpenOrderTracking"),
            object: orderId
        )
    }
    
    private func showCancelConfirmation(userInfo: [AnyHashable: Any]) {
        print("Show cancel confirmation")
    }
    
    private func handleNotificationTap(userInfo: [AnyHashable: Any]) {
        // เปิดหน้าจอที่เกี่ยวข้อง
        if let screen = userInfo["screen"] as? String {
            navigateToScreen(screen, userInfo: userInfo)
        }
    }
    
    private func handleNotificationDismiss(userInfo: [AnyHashable: Any]) {
        print("Notification dismissed")
    }
    
    private func navigateToScreen(_ screen: String, userInfo: [AnyHashable: Any]) {
        NotificationCenter.default.post(
            name: NSNotification.Name("NavigateToScreen"),
            object: screen,
            userInfo: userInfo as? [String: Any]
        )
    }
}
```

---

## 61.11 Background Notifications (Content-Available)

### การส่ง Silent Push

Silent Push Notifications ให้แอปทำงาน Background โดยไม่แสดง UI:

```json
{
    "aps": {
        "content-available": 1
    },
    "type": "sync_data",
    "sync_type": "messages",
    "last_sync": "2024-01-01T10:00:00Z"
}
```

### การจัดการ Background Notification

```swift
// AppDelegate.swift
func application(_ application: UIApplication,
                 didReceiveRemoteNotification userInfo: [AnyHashable: Any],
                 fetchCompletionHandler completionHandler: @escaping (UIBackgroundFetchResult) -> Void) {
    
    guard let type = userInfo["type"] as? String else {
        completionHandler(.noData)
        return
    }
    
    switch type {
    case "sync_data":
        handleDataSync(userInfo: userInfo, completion: completionHandler)
        
    case "refresh_content":
        handleContentRefresh(completion: completionHandler)
        
    default:
        completionHandler(.noData)
    }
}

private func handleDataSync(
    userInfo: [AnyHashable: Any],
    completion: @escaping (UIBackgroundFetchResult) -> Void
) {
    let syncType = userInfo["sync_type"] as? String ?? "all"
    
    DataSyncService.shared.sync(type: syncType) { result in
        switch result {
        case .success(let hasNewData):
            completion(hasNewData ? .newData : .noData)
        case .failure:
            completion(.failed)
        }
    }
}

private func handleContentRefresh(
    completion: @escaping (UIBackgroundFetchResult) -> Void
) {
    ContentService.shared.refresh { result in
        switch result {
        case .success:
            completion(.newData)
        case .failure:
            completion(.failed)
        }
    }
}
```

### Background App Refresh

เปิดใช้ Background App Refresh ใน Info.plist:

```xml
<key>UIBackgroundModes</key>
<array>
    <string>remote-notification</string>
    <string>fetch</string>
    <string>background-processing</string>
</array>
```

---

## 61.12 Scheduled Push Notifications

### Local Notification Triggers

```swift
class LocalNotificationScheduler {
    
    // กำหนดเวลา
    func scheduleAtTime() {
        let content = UNMutableNotificationContent()
        content.title = "ถึงเวลา!"
        content.body = "อย่าลืมดื่มน้ำนะ"
        content.sound = .default
        
        // ตั้งเวลาทุกวันเวลา 10:00 น.
        var dateComponents = DateComponents()
        dateComponents.hour = 10
        dateComponents.minute = 0
        
        let trigger = UNCalendarNotificationTrigger(
            dateMatching: dateComponents,
            repeats: true
        )
        
        let request = UNNotificationRequest(
            identifier: "daily-water-reminder",
            content: content,
            trigger: trigger
        )
        
        UNUserNotificationCenter.current().add(request)
    }
    
    // กำหนดตามตำแหน่ง
    func scheduleAtLocation() {
        let content = UNMutableNotificationContent()
        content.title = "ยินดีต้อนรับสู่สาขา"
        content.body = "มีโปรโมชั่นพิเศษสำหรับคุณ!"
        content.sound = .default
        
        let coordinates = CLLocationCoordinate2D(
            latitude: 13.7563,
            longitude: 100.5018
        )
        let region = CLCircularRegion(
            center: coordinates,
            radius: 100,
            identifier: "branch-area"
        )
        region.notifyOnEntry = true
        region.notifyOnExit = false
        
        let trigger = UNLocationNotificationTrigger(
            region: region,
            repeats: true
        )
        
        let request = UNNotificationRequest(
            identifier: "branch-welcome",
            content: content,
            trigger: trigger
        )
        
        UNUserNotificationCenter.current().add(request)
    }
    
    // กำหนดหลังจาก X วินาที
    func scheduleAfterDelay(seconds: TimeInterval) {
        let content = UNMutableNotificationContent()
        content.title = "เตือนความจำ"
        content.body = "คุณทิ้งของไว้ในรถ!"
        content.sound = .default
        
        let trigger = UNTimeIntervalNotificationTrigger(
            timeInterval: seconds,
            repeats: false
        )
        
        let request = UNNotificationRequest(
            identifier: UUID().uuidString,
            content: content,
            trigger: trigger
        )
        
        UNUserNotificationCenter.current().add(request)
    }
    
    // ยกเลิก Scheduled Notifications
    func cancelNotification(identifier: String) {
        UNUserNotificationCenter.current().removePendingNotificationRequests(
            withIdentifiers: [identifier]
        )
    }
    
    // ยกเลิกทั้งหมด
    func cancelAllNotifications() {
        UNUserNotificationCenter.current().removeAllPendingNotificationRequests()
    }
    
    // ดูรายการ Pending Notifications
    func getPendingNotifications(completion: @escaping ([UNNotificationRequest]) -> Void) {
        UNUserNotificationCenter.current().getPendingNotificationRequests { requests in
            completion(requests)
        }
    }
}
```

---

## 61.13 Firebase Cloud Messaging (FCM)

### การตั้งค่า Firebase

1. ไปที่ [Firebase Console](https://console.firebase.google.com)
2. สร้าง Project ใหม่
3. เพิ่ม iOS App
4. ดาวน์โหลด `GoogleService-Info.plist`
5. เพิ่มไฟล์ลงในโปรเจกต์

### การติดตั้ง Firebase SDK

```ruby
# Podfile
pod 'Firebase/Messaging'
pod 'Firebase/Analytics'
```

หรือใช้ Swift Package Manager:
- URL: `https://github.com/firebase/firebase-ios-sdk`
- Package: `FirebaseMessaging`

### การตั้งค่า FCM

```swift
// AppDelegate.swift
import Firebase
import FirebaseMessaging

class AppDelegate: UIResponder, UIApplicationDelegate {
    
    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        
        // Configure Firebase
        FirebaseApp.configure()
        
        // ตั้งค่า Messaging Delegate
        Messaging.messaging().delegate = self
        
        // ขอ Permission
        UNUserNotificationCenter.current().delegate = self
        UNUserNotificationCenter.current().requestAuthorization(
            options: [.alert, .sound, .badge]
        ) { granted, _ in
            if granted {
                DispatchQueue.main.async {
                    application.registerForRemoteNotifications()
                }
            }
        }
        
        return true
    }
    
    func application(_ application: UIApplication,
                     didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
        // ส่ง APNs Token ไปยัง Firebase
        Messaging.messaging().apnsToken = deviceToken
    }
}

// FCM Token Delegate
extension AppDelegate: MessagingDelegate {
    
    func messaging(_ messaging: Messaging, didReceiveRegistrationToken fcmToken: String?) {
        guard let token = fcmToken else { return }
        
        print("🔥 FCM Token: \(token)")
        
        // บันทึก Token
        UserDefaults.standard.set(token, forKey: "fcmToken")
        
        // ส่งไปยัง Server
        sendFCMTokenToServer(token)
    }
    
    private func sendFCMTokenToServer(_ token: String) {
        // อัปเดต Token บน Server
    }
}
```

### การสมัครสมาชิก Topic

```swift
class FCMTopicManager {
    
    // Subscribe ไปยัง Topic
    func subscribe(to topic: String) {
        Messaging.messaging().subscribe(toTopic: topic) { error in
            if let error = error {
                print("Error subscribing to \(topic): \(error)")
            } else {
                print("✅ Subscribed to \(topic)")
            }
        }
    }
    
    // Unsubscribe จาก Topic
    func unsubscribe(from topic: String) {
        Messaging.messaging().unsubscribe(fromTopic: topic) { error in
            if let error = error {
                print("Error unsubscribing from \(topic): \(error)")
            } else {
                print("✅ Unsubscribed from \(topic)")
            }
        }
    }
    
    // Subscribe ตามความสนใจของผู้ใช้
    func subscribeUserInterests(_ interests: [String]) {
        for interest in interests {
            subscribe(to: interest)
        }
    }
}

// ตัวอย่างการใช้งาน
let topicManager = FCMTopicManager()
topicManager.subscribe(to: "news")
topicManager.subscribe(to: "sports")
topicManager.subscribeUserInterests(["technology", "cooking", "travel"])
```

### การส่ง FCM ผ่าน REST API

```swift
class FCMSender {
    
    private let serverKey = "YOUR_SERVER_KEY"
    private let fcmURL = "https://fcm.googleapis.com/fcm/send"
    
    // ส่งไปยัง Device Token เดียว
    func sendToDevice(token: String, title: String, body: String, data: [String: Any]? = nil) async throws {
        var payload: [String: Any] = [
            "to": token,
            "notification": [
                "title": title,
                "body": body,
                "sound": "default"
            ]
        ]
        
        if let data = data {
            payload["data"] = data
        }
        
        try await sendRequest(payload: payload)
    }
    
    // ส่งไปยัง Topic
    func sendToTopic(topic: String, title: String, body: String) async throws {
        let payload: [String: Any] = [
            "to": "/topics/\(topic)",
            "notification": [
                "title": title,
                "body": body
            ]
        ]
        
        try await sendRequest(payload: payload)
    }
    
    private func sendRequest(payload: [String: Any]) async throws {
        guard let url = URL(string: fcmURL) else {
            throw URLError(.badURL)
        }
        
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        request.setValue("key=\(serverKey)", forHTTPHeaderField: "Authorization")
        request.httpBody = try JSONSerialization.data(withJSONObject: payload)
        
        let (_, response) = try await URLSession.shared.data(for: request)
        
        guard let httpResponse = response as? HTTPURLResponse,
              (200...299).contains(httpResponse.statusCode) else {
            throw URLError(.badServerResponse)
        }
    }
}
```

---

## 61.14 OneSignal Overview

### การติดตั้ง OneSignal

```ruby
# Podfile
pod 'OneSignalXCFramework', '>= 5.0.0', '< 6.0'
```

### การตั้งค่า OneSignal

```swift
// AppDelegate.swift
import OneSignalFramework

class AppDelegate: UIResponder, UIApplicationDelegate {
    
    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        
        // ลบ Initialize code นี้เมื่อทำ Production
        OneSignal.Debug.setLogLevel(.LL_VERBOSE)
        
        // Initialize OneSignal
        OneSignal.initialize("YOUR_ONESIGNAL_APP_ID",
                           withLaunchOptions: launchOptions)
        
        // Request Notification Permission
        OneSignal.Notifications.requestPermission({ accepted in
            print("User accepted notifications: \(accepted)")
        }, fallbackToSettings: true)
        
        return true
    }
}
```

### การส่ง Notification ผ่าน OneSignal REST API

```swift
class OneSignalSender {
    
    private let appId = "YOUR_ONESIGNAL_APP_ID"
    private let apiKey = "YOUR_REST_API_KEY"
    
    func sendNotification(
        playerIds: [String],
        title: String,
        message: String,
        data: [String: Any]? = nil
    ) async throws {
        
        let url = URL(string: "https://onesignal.com/api/v1/notifications")!
        
        var payload: [String: Any] = [
            "app_id": appId,
            "include_player_ids": playerIds,
            "headings": ["en": title, "th": title],
            "contents": ["en": message, "th": message]
        ]
        
        if let data = data {
            payload["data"] = data
        }
        
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        request.setValue("Basic \(apiKey)", forHTTPHeaderField: "Authorization")
        request.httpBody = try JSONSerialization.data(withJSONObject: payload)
        
        let (responseData, _) = try await URLSession.shared.data(for: request)
        
        if let response = try? JSONDecoder().decode(OneSignalResponse.self, from: responseData) {
            print("Notification sent - ID: \(response.id)")
        }
    }
    
    struct OneSignalResponse: Codable {
        let id: String
        let recipients: Int
    }
}
```

---

## 61.15 Server-Side Push Notification Sending

### Node.js APNs Server

```javascript
// server.js (Node.js)
const apn = require('@parse/node-apn');
const path = require('path');

// ใช้ Certificate
const optionsCert = {
    cert: path.join(__dirname, 'cert.pem'),
    key: path.join(__dirname, 'key.pem'),
    production: false // true สำหรับ Production
};

// ใช้ Token (ควรใช้วิธีนี้)
const optionsToken = {
    token: {
        key: path.join(__dirname, 'AuthKey_XXXXXXXX.p8'),
        keyId: 'YOUR_KEY_ID',
        teamId: 'YOUR_TEAM_ID'
    },
    production: false
};

const provider = new apn.Provider(optionsToken);

async function sendNotification(deviceToken, title, body, data = {}) {
    const notification = new apn.Notification();
    
    notification.expiry = Math.floor(Date.now() / 1000) + 3600; // 1 ชั่วโมง
    notification.badge = 1;
    notification.sound = 'ping.aiff';
    notification.alert = { title, body };
    notification.topic = 'com.yourcompany.yourapp';
    notification.payload = { data };
    
    try {
        const result = await provider.send(notification, deviceToken);
        console.log('Sent:', result.sent.length);
        console.log('Failed:', result.failed.length);
        
        if (result.failed.length > 0) {
            console.error('Failures:', result.failed);
        }
    } catch (error) {
        console.error('Error:', error);
    }
}

// ส่ง Notification
sendNotification(
    'DEVICE_TOKEN',
    'Hello!',
    'This is a test notification',
    { screen: 'home', userId: '123' }
);
```

### Python APNs Server

```python
# server.py (Python)
import jwt
import time
import httpx
import json

class APNsSender:
    def __init__(self, key_id: str, team_id: str, private_key: str, bundle_id: str):
        self.key_id = key_id
        self.team_id = team_id
        self.private_key = private_key
        self.bundle_id = bundle_id
        self.base_url = "https://api.sandbox.push.apple.com"
    
    def generate_jwt(self) -> str:
        """สร้าง JWT Token"""
        payload = {
            "iss": self.team_id,
            "iat": int(time.time())
        }
        
        headers = {
            "alg": "ES256",
            "kid": self.key_id
        }
        
        token = jwt.encode(
            payload,
            self.private_key,
            algorithm="ES256",
            headers=headers
        )
        
        return token
    
    async def send_notification(
        self,
        device_token: str,
        title: str,
        body: str,
        data: dict = None
    ):
        """ส่ง Notification ไปยังอุปกรณ์"""
        
        jwt_token = self.generate_jwt()
        
        payload = {
            "aps": {
                "alert": {
                    "title": title,
                    "body": body
                },
                "sound": "default",
                "badge": 1
            }
        }
        
        if data:
            payload.update(data)
        
        headers = {
            "authorization": f"bearer {jwt_token}",
            "apns-topic": self.bundle_id,
            "apns-push-type": "alert",
            "apns-priority": "10"
        }
        
        url = f"{self.base_url}/3/device/{device_token}"
        
        async with httpx.AsyncClient(http2=True) as client:
            response = await client.post(
                url,
                json=payload,
                headers=headers
            )
            
            if response.status_code == 200:
                print("✅ Notification sent successfully!")
            else:
                print(f"❌ Error: {response.status_code} - {response.text}")
```

---

## 61.16 Certificate vs Token-based Auth (APNs Auth Key)

### Certificate-based Authentication (.p12)

**ข้อดี:**
- ใช้ได้กับ APNs เวอร์ชันเก่า
- ง่ายต่อการตั้งค่าเบื้องต้น

**ข้อเสีย:**
- ต้องต่ออายุทุกปี
- แยก Certificate สำหรับ Development และ Production
- จัดการยากสำหรับหลาย Apps

### Token-based Authentication (.p8 Auth Key)

**ข้อดี:**
- ไม่หมดอายุ
- ใช้ Key เดียวสำหรับทุก Apps
- ใช้ได้ทั้ง Development และ Production
- ปลอดภัยกว่า

**ข้อเสีย:**
- ต้องสร้าง JWT Token ทุกครั้งที่ส่ง

### วิธีสร้าง APNs Auth Key

1. ไปที่ Apple Developer Portal
2. เลือก Certificates, Identifiers & Profiles
3. เลือก Keys
4. คลิก "+" เพื่อสร้าง Key ใหม่
5. เลือก "Apple Push Notifications service (APNs)"
6. ดาวน์โหลด `.p8` file

### การใช้ Auth Key ใน Swift

```swift
import CryptoKit

class APNsTokenGenerator {
    
    private let teamID: String
    private let keyID: String
    private let privateKey: P256.Signing.PrivateKey
    
    init(teamID: String, keyID: String, p8FileContent: String) throws {
        self.teamID = teamID
        self.keyID = keyID
        
        // โหลด Private Key จาก P8 file
        let cleanedKey = p8FileContent
            .replacingOccurrences(of: "-----BEGIN PRIVATE KEY-----", with: "")
            .replacingOccurrences(of: "-----END PRIVATE KEY-----", with: "")
            .replacingOccurrences(of: "\n", with: "")
            .trimmingCharacters(in: .whitespaces)
        
        guard let keyData = Data(base64Encoded: cleanedKey) else {
            throw APNsError.invalidKeyFormat
        }
        
        self.privateKey = try P256.Signing.PrivateKey(derRepresentation: keyData)
    }
    
    func generateToken() throws -> String {
        let header = ["alg": "ES256", "kid": keyID]
        let claims = ["iss": teamID, "iat": Int(Date().timeIntervalSince1970)] as [String: Any]
        
        let headerJSON = try JSONSerialization.data(withJSONObject: header)
        let claimsJSON = try JSONSerialization.data(withJSONObject: claims)
        
        let headerBase64 = headerJSON.base64URLEncoded
        let claimsBase64 = claimsJSON.base64URLEncoded
        
        let signingInput = Data("\(headerBase64).\(claimsBase64)".utf8)
        let signature = try privateKey.signature(for: signingInput)
        
        return "\(headerBase64).\(claimsBase64).\(signature.rawRepresentation.base64URLEncoded)"
    }
    
    enum APNsError: Error {
        case invalidKeyFormat
        case tokenGenerationFailed
    }
}
```

---

## 61.17 Notification Analytics

### การติดตาม Notification Events

```swift
class NotificationAnalytics {
    
    enum NotificationEvent: String {
        case received = "notification_received"
        case opened = "notification_opened"
        case dismissed = "notification_dismissed"
        case actionTaken = "notification_action_taken"
    }
    
    static func track(
        event: NotificationEvent,
        notificationId: String,
        category: String? = nil,
        action: String? = nil
    ) {
        var parameters: [String: Any] = [
            "notification_id": notificationId,
            "timestamp": ISO8601DateFormatter().string(from: Date())
        ]
        
        if let category = category {
            parameters["category"] = category
        }
        
        if let action = action {
            parameters["action"] = action
        }
        
        // ส่งไปยัง Analytics Service
        AnalyticsService.shared.log(event: event.rawValue, parameters: parameters)
    }
    
    static func trackDeliveryRate(sent: Int, delivered: Int) {
        let rate = delivered > 0 ? Double(delivered) / Double(sent) * 100 : 0
        
        AnalyticsService.shared.log(
            event: "notification_delivery_rate",
            parameters: [
                "sent": sent,
                "delivered": delivered,
                "rate": rate
            ]
        )
    }
    
    static func trackOpenRate(delivered: Int, opened: Int) {
        let rate = delivered > 0 ? Double(opened) / Double(delivered) * 100 : 0
        
        AnalyticsService.shared.log(
            event: "notification_open_rate",
            parameters: [
                "delivered": delivered,
                "opened": opened,
                "rate": rate
            ]
        )
    }
}

// Mock Analytics Service
class AnalyticsService {
    static let shared = AnalyticsService()
    
    func log(event: String, parameters: [String: Any]) {
        print("📊 Analytics: \(event) - \(parameters)")
    }
}
```

---

## 61.18 Provisional Authorization

Provisional Authorization ช่วยให้ผู้ใช้ได้รับ Notification แบบเงียบก่อน ก่อนที่จะตัดสินใจ Allow หรือ Deny

```swift
class ProvisionalNotificationManager {
    
    func requestProvisionalAuthorization() {
        UNUserNotificationCenter.current().requestAuthorization(
            options: [.alert, .sound, .badge, .provisional]
        ) { granted, error in
            if granted {
                print("✅ Provisional authorization granted")
                // Notification จะส่งไปยัง Notification Center แบบเงียบ
                // ไม่มี Banner หรือ Sound
                DispatchQueue.main.async {
                    UIApplication.shared.registerForRemoteNotifications()
                }
            }
        }
    }
    
    // Upgrade จาก Provisional เป็น Full Authorization
    func upgradeToFullAuthorization() {
        UNUserNotificationCenter.current().requestAuthorization(
            options: [.alert, .sound, .badge]
        ) { granted, error in
            DispatchQueue.main.async {
                if granted {
                    print("✅ Full authorization granted")
                } else {
                    print("❌ User denied full authorization")
                    // แนะนำให้เปิด Settings
                    self.showSettingsAlert()
                }
            }
        }
    }
    
    private func showSettingsAlert() {
        guard let windowScene = UIApplication.shared.connectedScenes.first as? UIWindowScene,
              let rootVC = windowScene.windows.first?.rootViewController else { return }
        
        let alert = UIAlertController(
            title: "เปิดการแจ้งเตือน",
            message: "กรุณาเปิดการแจ้งเตือนใน Settings เพื่อรับข้อมูลสำคัญ",
            preferredStyle: .alert
        )
        
        alert.addAction(UIAlertAction(title: "ไปที่ Settings", style: .default) { _ in
            if let settingsURL = URL(string: UIApplication.openSettingsURLString) {
                UIApplication.shared.open(settingsURL)
            }
        })
        
        alert.addAction(UIAlertAction(title: "ไม่ใช่ตอนนี้", style: .cancel))
        
        rootVC.present(alert, animated: true)
    }
}
```

---

## 61.19 Critical Alerts

Critical Alerts เป็น Notification พิเศษที่แสดงแม้ว่า Do Not Disturb จะเปิดอยู่ และมีเสียงดัง ต้องได้รับ Entitlement จาก Apple ก่อน

```swift
class CriticalAlertManager {
    
    func requestCriticalAlertPermission() {
        // ต้องมี com.apple.developer.usernotifications.critical-alerts entitlement
        UNUserNotificationCenter.current().requestAuthorization(
            options: [.alert, .sound, .badge, .criticalAlert]
        ) { granted, error in
            print("Critical alerts granted: \(granted)")
        }
    }
    
    func sendCriticalAlert(title: String, body: String) {
        let content = UNMutableNotificationContent()
        content.title = title
        content.body = body
        
        // Critical Sound
        content.sound = UNNotificationSound.defaultCritical
        
        // หรือกำหนด Volume
        content.sound = UNNotificationSound.criticalSoundNamed(
            UNNotificationSoundName("alert.aiff"),
            withAudioVolume: 1.0
        )
        
        // กำหนด Interruption Level
        if #available(iOS 15.0, *) {
            content.interruptionLevel = .critical
        }
        
        let trigger = UNTimeIntervalNotificationTrigger(
            timeInterval: 1,
            repeats: false
        )
        
        let request = UNNotificationRequest(
            identifier: "critical-alert-\(UUID())",
            content: content,
            trigger: trigger
        )
        
        UNUserNotificationCenter.current().add(request) { error in
            if let error = error {
                print("Error: \(error)")
            }
        }
    }
}
```

---

## 61.20 Time-Sensitive Notifications

Time-Sensitive Notifications เป็น Notifications ที่สำคัญเวลา เช่น การนัดหมาย การเตือนยา:

```swift
@available(iOS 15.0, *)
class TimeSensitiveNotificationManager {
    
    func scheduleTimeSensitiveNotification() {
        let content = UNMutableNotificationContent()
        content.title = "การนัดหมายในอีก 15 นาที"
        content.body = "การประชุมกับทีม Marketing"
        content.sound = .default
        
        // กำหนด Interruption Level
        content.interruptionLevel = .timeSensitive
        
        // กำหนด Relevance Score (0.0 - 1.0)
        content.relevanceScore = 0.9
        
        let trigger = UNTimeIntervalNotificationTrigger(
            timeInterval: 5,
            repeats: false
        )
        
        let request = UNNotificationRequest(
            identifier: UUID().uuidString,
            content: content,
            trigger: trigger
        )
        
        UNUserNotificationCenter.current().add(request)
    }
    
    // Interruption Levels ทั้งหมด
    func demonstrateInterruptionLevels() {
        let levels: [(UNNotificationInterruptionLevel, String)] = [
            (.passive, "Passive - ไม่รบกวน"),
            (.active, "Active - ปกติ"),
            (.timeSensitive, "Time Sensitive - สำคัญ"),
            (.critical, "Critical - ฉุกเฉิน")
        ]
        
        for (index, (level, description)) in levels.enumerated() {
            let content = UNMutableNotificationContent()
            content.title = "Interruption Level"
            content.body = description
            content.interruptionLevel = level
            content.sound = level == .passive ? nil : .default
            
            let trigger = UNTimeIntervalNotificationTrigger(
                timeInterval: Double(index + 1) * 5,
                repeats: false
            )
            
            let request = UNNotificationRequest(
                identifier: "level-\(index)",
                content: content,
                trigger: trigger
            )
            
            UNUserNotificationCenter.current().add(request)
        }
    }
}
```

---

## 61.21 Focus Filters

Focus Filters ช่วยให้แอปสามารถปรับตัวตาม Focus Mode ของผู้ใช้:

```swift
import AppIntents

// สร้าง Focus Filter
@available(iOS 16.0, *)
struct MyAppFocusFilter: SetFocusFilterIntent {
    
    static var title: LocalizedStringResource = "กรอง Notifications ตาม Focus"
    
    static var description: IntentDescription? = """
    ปรับแต่งการรับ Notifications ตาม Focus Mode ที่เลือก
    """
    
    // Parameters
    @Parameter(title: "แสดงข้อความ")
    var showMessages: Bool
    
    @Parameter(title: "แสดงการแจ้งเตือนจากงาน")
    var showWorkNotifications: Bool
    
    @Parameter(title: "เสียง Notification")
    var notificationSound: NotificationSoundOption
    
    enum NotificationSoundOption: String, AppEnum {
        case on = "เปิด"
        case off = "ปิด"
        case vibrationOnly = "สั่นเท่านั้น"
        
        static var typeDisplayRepresentation = TypeDisplayRepresentation(name: "เสียง")
        static var caseDisplayRepresentations: [Self: DisplayRepresentation] = [
            .on: "เปิดเสียง",
            .off: "ปิดเสียง",
            .vibrationOnly: "สั่นเท่านั้น"
        ]
    }
    
    func perform() async throws -> some IntentResult {
        // บันทึก Focus Filter Settings
        let settings = FocusFilterSettings(
            showMessages: showMessages,
            showWorkNotifications: showWorkNotifications,
            notificationSound: notificationSound.rawValue
        )
        
        FocusFilterManager.shared.applySettings(settings)
        
        return .result()
    }
}

struct FocusFilterSettings: Codable {
    let showMessages: Bool
    let showWorkNotifications: Bool
    let notificationSound: String
}

class FocusFilterManager {
    static let shared = FocusFilterManager()
    private let settingsKey = "focusFilterSettings"
    
    func applySettings(_ settings: FocusFilterSettings) {
        if let data = try? JSONEncoder().encode(settings) {
            UserDefaults.standard.set(data, forKey: settingsKey)
        }
        
        // Apply settings to notification handling
        NotificationCenter.default.post(
            name: NSNotification.Name("FocusFilterChanged"),
            object: settings
        )
    }
    
    var currentSettings: FocusFilterSettings? {
        guard let data = UserDefaults.standard.data(forKey: settingsKey) else { return nil }
        return try? JSONDecoder().decode(FocusFilterSettings.self, from: data)
    }
}
```

---

## 61.22 แบบฝึกหัดพร้อมเฉลยสมบูรณ์

### แบบฝึกหัดที่ 1: ระบบ Notification พื้นฐาน

**โจทย์:** สร้างระบบ Notification สำหรับแอป Todo List ที่แสดง Reminder เมื่อถึงเวลาที่กำหนด

```swift
// เฉลย: TodoNotificationSystem.swift
import UserNotifications
import Foundation

struct TodoItem: Identifiable, Codable {
    let id: UUID
    var title: String
    var dueDate: Date
    var isCompleted: Bool
    var notificationId: String?
    
    init(title: String, dueDate: Date) {
        self.id = UUID()
        self.title = title
        self.dueDate = dueDate
        self.isCompleted = false
    }
}

class TodoNotificationManager {
    
    static let shared = TodoNotificationManager()
    private let notificationCenter = UNUserNotificationCenter.current()
    
    // Request Permission
    func requestPermission(completion: @escaping (Bool) -> Void) {
        notificationCenter.requestAuthorization(
            options: [.alert, .sound, .badge]
        ) { granted, _ in
            DispatchQueue.main.async {
                completion(granted)
            }
        }
    }
    
    // Schedule Notification สำหรับ Todo
    @discardableResult
    func scheduleNotification(for todo: TodoItem) -> String {
        let notificationId = "todo-\(todo.id.uuidString)"
        
        let content = UNMutableNotificationContent()
        content.title = "⏰ ครบกำหนดแล้ว!"
        content.body = todo.title
        content.sound = .default
        content.badge = 1
        content.userInfo = ["todo_id": todo.id.uuidString]
        content.categoryIdentifier = "TODO_CATEGORY"
        
        // สร้าง DateComponents จาก dueDate
        let components = Calendar.current.dateComponents(
            [.year, .month, .day, .hour, .minute],
            from: todo.dueDate
        )
        
        let trigger = UNCalendarNotificationTrigger(
            dateMatching: components,
            repeats: false
        )
        
        let request = UNNotificationRequest(
            identifier: notificationId,
            content: content,
            trigger: trigger
        )
        
        notificationCenter.add(request) { error in
            if let error = error {
                print("Error scheduling notification: \(error)")
            }
        }
        
        return notificationId
    }
    
    // ยกเลิก Notification
    func cancelNotification(id: String) {
        notificationCenter.removePendingNotificationRequests(withIdentifiers: [id])
    }
    
    // Setup Categories
    func setupCategories() {
        let doneAction = UNNotificationAction(
            identifier: "MARK_DONE",
            title: "✅ ทำเสร็จแล้ว",
            options: []
        )
        
        let snoozeAction = UNNotificationAction(
            identifier: "SNOOZE",
            title: "⏰ เลื่อนออกไป 30 นาที",
            options: []
        )
        
        let deleteAction = UNNotificationAction(
            identifier: "DELETE_TODO",
            title: "🗑️ ลบ",
            options: [.destructive]
        )
        
        let category = UNNotificationCategory(
            identifier: "TODO_CATEGORY",
            actions: [doneAction, snoozeAction, deleteAction],
            intentIdentifiers: [],
            options: []
        )
        
        notificationCenter.setNotificationCategories([category])
    }
}

// ViewModel
class TodoListViewModel: ObservableObject {
    @Published var todos: [TodoItem] = []
    private let notificationManager = TodoNotificationManager.shared
    
    init() {
        notificationManager.setupCategories()
        notificationManager.requestPermission { granted in
            if granted {
                print("Notification permission granted")
            }
        }
        loadTodos()
    }
    
    func addTodo(title: String, dueDate: Date) {
        var todo = TodoItem(title: title, dueDate: dueDate)
        let notificationId = notificationManager.scheduleNotification(for: todo)
        todo.notificationId = notificationId
        todos.append(todo)
        saveTodos()
    }
    
    func completeTodo(id: UUID) {
        if let index = todos.firstIndex(where: { $0.id == id }) {
            todos[index].isCompleted = true
            if let notificationId = todos[index].notificationId {
                notificationManager.cancelNotification(id: notificationId)
            }
            saveTodos()
        }
    }
    
    func deleteTodo(id: UUID) {
        if let index = todos.firstIndex(where: { $0.id == id }) {
            if let notificationId = todos[index].notificationId {
                notificationManager.cancelNotification(id: notificationId)
            }
            todos.remove(at: index)
            saveTodos()
        }
    }
    
    private func saveTodos() {
        if let data = try? JSONEncoder().encode(todos) {
            UserDefaults.standard.set(data, forKey: "todos")
        }
    }
    
    private func loadTodos() {
        guard let data = UserDefaults.standard.data(forKey: "todos"),
              let saved = try? JSONDecoder().decode([TodoItem].self, from: data) else { return }
        todos = saved
    }
}
```

### แบบฝึกหัดที่ 2: Rich Notification ด้วย Notification Service Extension

**โจทย์:** สร้าง Notification Service Extension ที่ดาวน์โหลดรูปภาพและเพิ่มเป็น Attachment

```swift
// เฉลย: RichNotificationService.swift
import UserNotifications
import UIKit

class RichNotificationService: UNNotificationServiceExtension {
    
    var contentHandler: ((UNNotificationContent) -> Void)?
    var modifiedContent: UNMutableNotificationContent?
    
    override func didReceive(
        _ request: UNNotificationRequest,
        withContentHandler contentHandler: @escaping (UNNotificationContent) -> Void
    ) {
        self.contentHandler = contentHandler
        
        guard let content = request.content.mutableCopy() as? UNMutableNotificationContent else {
            contentHandler(request.content)
            return
        }
        
        self.modifiedContent = content
        
        // ดาวน์โหลดรูปภาพแบบ Async
        Task {
            await processNotification(content: content, originalContent: request.content)
            contentHandler(content)
        }
    }
    
    override func serviceExtensionTimeWillExpire() {
        guard let contentHandler = contentHandler,
              let content = modifiedContent else { return }
        contentHandler(content)
    }
    
    private func processNotification(
        content: UNMutableNotificationContent,
        originalContent: UNNotificationContent
    ) async {
        // 1. ดาวน์โหลดรูปภาพ
        if let imageURLString = content.userInfo["image_url"] as? String,
           let imageURL = URL(string: imageURLString) {
            if let attachment = await downloadImage(from: imageURL) {
                content.attachments = [attachment]
            }
        }
        
        // 2. เพิ่ม Emoji ตาม Category
        if let category = content.userInfo["category"] as? String {
            let emoji = categoryEmoji(for: category)
            content.title = "\(emoji) \(content.title)"
        }
        
        // 3. เพิ่ม Badge Count จาก Server
        if let badgeCount = content.userInfo["badge_count"] as? Int {
            content.badge = NSNumber(value: badgeCount)
        }
    }
    
    private func downloadImage(from url: URL) async -> UNNotificationAttachment? {
        do {
            let (tempURL, response) = try await URLSession.shared.download(from: url)
            
            let mimeType = (response as? HTTPURLResponse)?.value(forHTTPHeaderField: "Content-Type") ?? "image/jpeg"
            let ext = fileExtension(for: mimeType)
            
            let permanentURL = tempURL.deletingLastPathComponent()
                .appendingPathComponent(UUID().uuidString)
                .appendingPathExtension(ext)
            
            try FileManager.default.moveItem(at: tempURL, to: permanentURL)
            
            return try UNNotificationAttachment(
                identifier: UUID().uuidString,
                url: permanentURL,
                options: nil
            )
        } catch {
            print("Failed to download image: \(error)")
            return nil
        }
    }
    
    private func categoryEmoji(for category: String) -> String {
        switch category.lowercased() {
        case "news": return "📰"
        case "message": return "💬"
        case "order": return "📦"
        case "payment": return "💳"
        case "promo": return "🎉"
        default: return "🔔"
        }
    }
    
    private func fileExtension(for mimeType: String) -> String {
        let mapping: [String: String] = [
            "image/jpeg": "jpg",
            "image/png": "png",
            "image/gif": "gif",
            "image/webp": "webp",
            "video/mp4": "mp4",
            "audio/mpeg": "mp3"
        ]
        return mapping[mimeType] ?? "jpg"
    }
}
```

### แบบฝึกหัดที่ 3: Notification Analytics Dashboard

**โจทย์:** สร้าง Analytics System ที่ติดตาม Notification Open Rate และ Conversion

```swift
// เฉลย: NotificationAnalyticsDashboard.swift
import Foundation
import Combine

struct NotificationMetrics: Codable {
    var sent: Int = 0
    var delivered: Int = 0
    var opened: Int = 0
    var dismissed: Int = 0
    var actions: [String: Int] = [:]
    var revenue: Double = 0.0
    
    var deliveryRate: Double {
        guard sent > 0 else { return 0 }
        return Double(delivered) / Double(sent) * 100
    }
    
    var openRate: Double {
        guard delivered > 0 else { return 0 }
        return Double(opened) / Double(delivered) * 100
    }
    
    var conversionRate: Double {
        guard opened > 0 else { return 0 }
        let conversions = actions["PURCHASE"] ?? 0
        return Double(conversions) / Double(opened) * 100
    }
}

class NotificationAnalyticsDashboard: ObservableObject {
    
    @Published var metrics: NotificationMetrics = NotificationMetrics()
    @Published var dailyMetrics: [Date: NotificationMetrics] = [:]
    
    private let metricsKey = "notificationMetrics"
    
    init() {
        loadMetrics()
    }
    
    // บันทึก Event
    func trackEvent(_ event: String, notificationId: String, metadata: [String: Any] = [:]) {
        let calendar = Calendar.current
        let today = calendar.startOfDay(for: Date())
        
        switch event {
        case "sent":
            metrics.sent += 1
            dailyMetrics[today, default: NotificationMetrics()].sent += 1
            
        case "delivered":
            metrics.delivered += 1
            dailyMetrics[today, default: NotificationMetrics()].delivered += 1
            
        case "opened":
            metrics.opened += 1
            dailyMetrics[today, default: NotificationMetrics()].opened += 1
            
        case "dismissed":
            metrics.dismissed += 1
            dailyMetrics[today, default: NotificationMetrics()].dismissed += 1
            
        default:
            // Action Events
            metrics.actions[event, default: 0] += 1
            dailyMetrics[today, default: NotificationMetrics()].actions[event, default: 0] += 1
            
            if event == "PURCHASE",
               let amount = metadata["amount"] as? Double {
                metrics.revenue += amount
                dailyMetrics[today, default: NotificationMetrics()].revenue += amount
            }
        }
        
        saveMetrics()
    }
    
    // คืนค่า Metrics ช่วง 7 วัน
    func weeklyMetrics() -> [NotificationMetrics] {
        let calendar = Calendar.current
        return (0..<7).compactMap { daysAgo in
            let date = calendar.date(byAdding: .day, value: -daysAgo, to: Date())!
            let day = calendar.startOfDay(for: date)
            return dailyMetrics[day]
        }.reversed()
    }
    
    func generateReport() -> String {
        """
        📊 Notification Analytics Report
        ================================
        Total Sent: \(metrics.sent)
        Total Delivered: \(metrics.delivered)
        Total Opened: \(metrics.opened)
        Total Dismissed: \(metrics.dismissed)
        
        📈 Rates
        Delivery Rate: \(String(format: "%.1f", metrics.deliveryRate))%
        Open Rate: \(String(format: "%.1f", metrics.openRate))%
        Conversion Rate: \(String(format: "%.1f", metrics.conversionRate))%
        
        💰 Revenue from Notifications: ฿\(String(format: "%.2f", metrics.revenue))
        
        🎯 Actions Taken:
        \(metrics.actions.map { "  \($0.key): \($0.value)" }.joined(separator: "\n"))
        """
    }
    
    private func saveMetrics() {
        if let data = try? JSONEncoder().encode(metrics) {
            UserDefaults.standard.set(data, forKey: metricsKey)
        }
    }
    
    private func loadMetrics() {
        guard let data = UserDefaults.standard.data(forKey: metricsKey),
              let saved = try? JSONDecoder().decode(NotificationMetrics.self, from: data) else { return }
        metrics = saved
    }
}
```

---

## 61.23 การสร้างระบบ Push Notification ครบวงจร

### สถาปัตยกรรมของระบบ

```
┌─────────────────────────────────────────────────────────┐
│                     Push Notification System             │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  iOS App                                               │
│  ┌──────────────────┐   ┌──────────────────┐           │
│  │ Notification     │   │ Notification     │           │
│  │ Registration     │   │ Handler          │           │
│  └────────┬─────────┘   └────────▲─────────┘           │
│           │                      │                     │
│  ┌──────────────────────────────────────────────────┐  │
│  │              UNUserNotificationCenter            │  │
│  └──────────────────────────────────────────────────┘  │
│                                                         │
├─────────────────────────────────────────────────────────┤
│  Extensions                                             │
│  ┌──────────────────┐   ┌──────────────────┐           │
│  │ Service          │   │ Content          │           │
│  │ Extension        │   │ Extension        │           │
│  └──────────────────┘   └──────────────────┘           │
├─────────────────────────────────────────────────────────┤
│  Backend                                                │
│  ┌──────────┐ ┌──────────┐ ┌──────────────────────┐   │
│  │ APNs     │ │  FCM     │ │   OneSignal          │   │
│  │ Direct   │ │          │ │                      │   │
│  └──────────┘ └──────────┘ └──────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

### Complete Push Notification System

```swift
// PushNotificationSystem.swift
import UserNotifications
import UIKit
import FirebaseMessaging

// MARK: - Push Notification System
class PushNotificationSystem: NSObject {
    
    static let shared = PushNotificationSystem()
    
    private var handlers: [String: (UNNotificationResponse) -> Void] = [:]
    private let analytics = NotificationAnalyticsDashboard()
    
    override init() {
        super.init()
        setupSystem()
    }
    
    // MARK: - Setup
    
    func setupSystem() {
        UNUserNotificationCenter.current().delegate = self
        setupCategories()
    }
    
    func requestPermissions(completion: @escaping (Bool) -> Void) {
        UNUserNotificationCenter.current().requestAuthorization(
            options: [.alert, .sound, .badge, .provisional]
        ) { [weak self] granted, error in
            DispatchQueue.main.async {
                if granted {
                    UIApplication.shared.registerForRemoteNotifications()
                    self?.analytics.trackEvent("permission_granted", notificationId: "system")
                }
                completion(granted)
            }
        }
    }
    
    // MARK: - Token Management
    
    func handleDeviceToken(_ tokenData: Data) {
        let token = tokenData.map { String(format: "%02.2hhx", $0) }.joined()
        
        // Update Firebase
        Messaging.messaging().apnsToken = tokenData
        
        // Save and sync with backend
        TokenManager.shared.updateToken(token)
    }
    
    // MARK: - Category Setup
    
    private func setupCategories() {
        let categories: [UNNotificationCategory] = [
            createMessageCategory(),
            createOrderCategory(),
            createPromoCategory()
        ]
        UNUserNotificationCenter.current().setNotificationCategories(Set(categories))
    }
    
    private func createMessageCategory() -> UNNotificationCategory {
        let replyAction = UNTextInputNotificationAction(
            identifier: "REPLY",
            title: "ตอบกลับ",
            options: [],
            textInputButtonTitle: "ส่ง",
            textInputPlaceholder: "พิมพ์ข้อความ..."
        )
        
        return UNNotificationCategory(
            identifier: "MESSAGE",
            actions: [replyAction],
            intentIdentifiers: [],
            options: [.hiddenPreviewsShowTitle]
        )
    }
    
    private func createOrderCategory() -> UNNotificationCategory {
        let trackAction = UNNotificationAction(
            identifier: "TRACK",
            title: "ติดตามสินค้า",
            options: [.foreground]
        )
        
        return UNNotificationCategory(
            identifier: "ORDER",
            actions: [trackAction],
            intentIdentifiers: [],
            options: []
        )
    }
    
    private func createPromoCategory() -> UNNotificationCategory {
        let shopAction = UNNotificationAction(
            identifier: "SHOP",
            title: "ช้อปเลย",
            options: [.foreground]
        )
        
        let dismissAction = UNNotificationAction(
            identifier: "DISMISS_PROMO",
            title: "ไม่สนใจ",
            options: [.destructive]
        )
        
        return UNNotificationCategory(
            identifier: "PROMO",
            actions: [shopAction, dismissAction],
            intentIdentifiers: [],
            options: []
        )
    }
    
    // MARK: - Handler Registration
    
    func registerHandler(for category: String, handler: @escaping (UNNotificationResponse) -> Void) {
        handlers[category] = handler
    }
}

// MARK: - UNUserNotificationCenterDelegate
extension PushNotificationSystem: UNUserNotificationCenterDelegate {
    
    func userNotificationCenter(
        _ center: UNUserNotificationCenter,
        willPresent notification: UNNotification,
        withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void
    ) {
        let category = notification.request.content.categoryIdentifier
        analytics.trackEvent("delivered", notificationId: notification.request.identifier)
        
        if #available(iOS 14.0, *) {
            completionHandler([.banner, .sound, .badge])
        } else {
            completionHandler([.alert, .sound, .badge])
        }
    }
    
    func userNotificationCenter(
        _ center: UNUserNotificationCenter,
        didReceive response: UNNotificationResponse,
        withCompletionHandler completionHandler: @escaping () -> Void
    ) {
        let notificationId = response.notification.request.identifier
        let category = response.notification.request.content.categoryIdentifier
        
        analytics.trackEvent("opened", notificationId: notificationId)
        
        if response.actionIdentifier == UNNotificationDefaultActionIdentifier {
            handlers[category]?(response)
        } else {
            analytics.trackEvent(response.actionIdentifier, notificationId: notificationId)
            handlers[category]?(response)
        }
        
        completionHandler()
    }
}

// MARK: - Token Manager
class TokenManager {
    static let shared = TokenManager()
    
    private let tokenKey = "pushToken"
    private let fcmTokenKey = "fcmToken"
    
    var pushToken: String? {
        get { UserDefaults.standard.string(forKey: tokenKey) }
    }
    
    var fcmToken: String? {
        get { UserDefaults.standard.string(forKey: fcmTokenKey) }
    }
    
    func updateToken(_ token: String) {
        let existing = UserDefaults.standard.string(forKey: tokenKey)
        
        if token != existing {
            UserDefaults.standard.set(token, forKey: tokenKey)
            syncWithServer(apnsToken: token)
        }
    }
    
    func updateFCMToken(_ token: String) {
        UserDefaults.standard.set(token, forKey: fcmTokenKey)
        syncWithServer(fcmToken: token)
    }
    
    private func syncWithServer(apnsToken: String? = nil, fcmToken: String? = nil) {
        var params: [String: String] = [:]
        
        if let apns = apnsToken { params["apns_token"] = apns }
        if let fcm = fcmToken { params["fcm_token"] = fcm }
        
        guard !params.isEmpty else { return }
        
        // API Call
        Task {
            do {
                try await APIClient.shared.updateDeviceTokens(params)
                print("✅ Tokens synced with server")
            } catch {
                print("❌ Failed to sync tokens: \(error)")
            }
        }
    }
}

// MARK: - API Client (Mock)
class APIClient {
    static let shared = APIClient()
    
    func updateDeviceTokens(_ tokens: [String: String]) async throws {
        // Mock implementation
        print("Syncing tokens: \(tokens)")
    }
}
```

---

## 61.24 สรุป

ในบทนี้เราได้เรียนรู้เกี่ยวกับ Push Notifications ครบทุกด้าน:

1. **APNs Architecture** - เข้าใจการทำงานของระบบ Push Notification ของ Apple
2. **Registration & Device Token** - การลงทะเบียนและจัดการ Device Token
3. **Notification Types** - Alert, Badge, Sound, Silent Notifications
4. **Service & Content Extensions** - การแก้ไข Content และสร้าง Custom UI
5. **Categories & Actions** - การตอบสนองต่อ Notification โดยตรง
6. **FCM & OneSignal** - การใช้ Third-party Services
7. **Authentication Methods** - Certificate vs Token-based Auth
8. **Advanced Features** - Critical Alerts, Time-Sensitive, Focus Filters
9. **Analytics** - การวัดและวิเคราะห์ประสิทธิภาพ
10. **Complete System** - การสร้างระบบ Push Notification ครบวงจร

### สิ่งที่ควรจำ:

- ใช้ Token-based Authentication (.p8) แทน Certificate (.p12)
- จัดการ Device Token อย่างระมัดระวัง Token อาจเปลี่ยนแปลงได้
- เคารพ Privacy ของผู้ใช้ ขอ Permission เมื่อจำเป็นเท่านั้น
- ใช้ Provisional Authorization เพื่อลด Friction ในการ Onboarding
- วิเคราะห์ Data เพื่อปรับปรุง Notification Strategy
- ทดสอบบน Real Device เสมอ ไม่ใช่แค่ Simulator

### แหล่งข้อมูลเพิ่มเติม:

- [Apple Developer: UserNotifications](https://developer.apple.com/documentation/usernotifications)
- [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging/ios/client)
- [OneSignal iOS SDK](https://documentation.onesignal.com/docs/ios-sdk-setup)
- [APNs Overview](https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server)

---

*บทถัดไป: Part 62 - App Store Distribution*
