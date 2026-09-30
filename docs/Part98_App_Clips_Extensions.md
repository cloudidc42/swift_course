# Part 98: App Clips และ Extensions ใน Swift

## ภาพรวม

ในบทนี้เราจะเรียนรู้เกี่ยวกับ **App Clips** และ **Extensions** ซึ่งเป็นสองเทคโนโลยีสำคัญที่ช่วยให้แอปพลิเคชัน iOS ทำงานได้อย่างยืดหยุ่นมากขึ้น App Clips ช่วยให้ผู้ใช้เข้าถึงฟีเจอร์หลักของแอปโดยไม่ต้องติดตั้งแอปเต็มรูปแบบ ส่วน Extensions ช่วยให้แอปของเราทำงานร่วมกับระบบและแอปอื่นๆ ได้

---

## 1. App Clips Overview

### App Clips คืออะไร?

**App Clips** คือส่วนเล็กๆ ของแอปพลิเคชันที่ผู้ใช้สามารถเปิดใช้งานได้ทันทีโดยไม่ต้องดาวน์โหลดแอปเต็มรูปแบบ ผู้ใช้สามารถสัมผัสประสบการณ์หลักของแอปได้อย่างรวดเร็ว แล้วค่อยตัดสินใจว่าจะติดตั้งแอปเต็มรูปแบบหรือไม่

### Use Cases หลักของ App Clips

- **การชำระเงิน (Payment)**: สแกน QR Code ที่ร้านอาหารเพื่อชำระเงินทันที
- **เมนูอาหาร (Menu)**: เปิดดูเมนูร้านอาหารจาก NFC tag โดยไม่ต้องติดตั้งแอป
- **จอดรถ (Parking)**: สแกน QR Code ที่ที่จอดรถเพื่อชำระค่าจอดรถ
- **ตั๋ว (Ticketing)**: เข้างาน event โดยแสดง App Clip แทนตั๋วกระดาษ
- **การจองบริการ**: นัดหมาย, จองโต๊ะร้านอาหาร, จองห้องพัก

### ข้อจำกัดด้านขนาด

App Clips มีขนาดไม่เกิน **50MB** (ไฟล์ที่ดาวน์โหลดได้จริง) ซึ่งทำให้โหลดได้รวดเร็ว Apple กำหนดขนาดนี้เพื่อให้ผู้ใช้ไม่ต้องรอนาน

### App Clip Card Metadata

เมื่อผู้ใช้ trigger App Clip ระบบจะแสดง **App Clip Card** ซึ่งประกอบด้วยข้อมูล metadata ที่กำหนดใน App Store Connect โดยข้อมูลเหล่านี้รวมถึง:

- **Title**: ชื่อของ App Clip experience
- **Subtitle**: คำอธิบายสั้นๆ
- **Call to Action**: ข้อความบนปุ่ม (เช่น "Open", "Play", "View")
- **Header Image**: รูปภาพหัว card

ตัวอย่าง YAML สำหรับ App Clip metadata (ใช้ใน App Store Connect API):

```yaml
appClipDefaultExperience:
  action: OPEN
  headerImage: /path/to/header.png
  localizations:
    - locale: "th"
      title: "เมนูร้านอาหาร"
      subtitle: "ดูเมนูและสั่งอาหารได้เลย"
      callToAction: "เปิด"
    - locale: "en-US"
      title: "Restaurant Menu"
      subtitle: "View menu and order food"
      callToAction: "Open"
  advancedExperiences:
    - url: "https://example.com/menu/table/5"
      action: ORDER
      title: "สั่งอาหารโต๊ะ 5"
```

### Invocation Types

App Clips สามารถเปิดได้จากหลายช่องทาง:

| ประเภท | คำอธิบาย |
|--------|----------|
| QR Code | สแกน QR Code ที่มี URL ของ App Clip |
| NFC Tag | แตะ NFC tag ที่ร้านค้าหรือสถานที่ต่างๆ |
| Safari Smart App Banner | Banner ใน Safari web page |
| Messages | ลิงก์ที่แชร์ใน iMessage |
| Maps Place Card | การ์ดสถานที่ใน Apple Maps |
| Siri App Suggestions | คำแนะนำจาก Siri |
| Recently Used | App Clip ที่เพิ่งใช้งาน |

---

## 2. Creating an App Clip Target

### การตั้งค่าใน Xcode

ขั้นตอนการสร้าง App Clip target:

1. เปิด Xcode project
2. ไปที่ **File → New → Target**
3. เลือก **App Clip** template
4. ตั้งชื่อ target (ควรใช้ชื่อแอปหลัก + "Clip" เช่น "RestaurantAppClip")
5. ตรวจสอบว่า **Embed in Application** ชี้ไปที่ main app target

### การแชร์โค้ดด้วย Framework Target

เพื่อแชร์โค้ดระหว่าง App Clip และแอปหลัก ควรสร้าง **shared framework**:

```
MyRestaurantApp (Main App)
    └── Embeds: SharedCore.framework
MyRestaurantAppClip (App Clip)
    └── Embeds: SharedCore.framework
SharedCore.framework
    └── Models, NetworkService, UIComponents
```

การตั้งค่าใน `Package.swift` หากใช้ Swift Package:

```swift
// Package.swift
let package = Package(
    name: "SharedCore",
    platforms: [.iOS(.v16)],
    products: [
        .library(name: "SharedCore", targets: ["SharedCore"])
    ],
    targets: [
        .target(
            name: "SharedCore",
            path: "Sources/SharedCore"
        )
    ]
)
```

### การรับ URL จาก NSUserActivity

เมื่อ App Clip ถูกเปิด ระบบจะส่ง `NSUserActivity` ที่มี URL ของ invocation มาให้

```swift
// RestaurantMenuAppClip.swift
import SwiftUI

@main
struct RestaurantMenuAppClip: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
                .onContinueUserActivity(
                    NSUserActivityTypes.browsingWeb,
                    perform: handleUserActivity
                )
        }
    }
    
    func handleUserActivity(_ activity: NSUserActivity) {
        // ดึง URL จาก NSUserActivity
        guard let incomingURL = activity.webpageURL else {
            print("ไม่พบ URL ใน UserActivity")
            return
        }
        
        // parse URL เพื่อดึง parameters
        let components = URLComponents(
            url: incomingURL,
            resolvingAgainstBaseURL: true
        )
        
        // ตัวอย่าง URL: https://restaurant.example.com/menu?table=5
        if let tableParam = components?.queryItems?.first(where: {
            $0.name == "table"
        })?.value {
            print("โต๊ะหมายเลข: \(tableParam)")
            // นำทางไปยังหน้าสั่งอาหารของโต๊ะนั้น
            AppState.shared.selectedTable = Int(tableParam)
        }
    }
}

// ContentView.swift - Entry point ของ Restaurant Menu App Clip
struct ContentView: View {
    @StateObject private var appState = AppState.shared
    
    var body: some View {
        NavigationView {
            if let table = appState.selectedTable {
                MenuView(tableNumber: table)
            } else {
                MenuView(tableNumber: nil)
            }
        }
    }
}

// AppState.swift - Shared state สำหรับ App Clip
class AppState: ObservableObject {
    static let shared = AppState()
    @Published var selectedTable: Int?
}

// MenuView.swift - หน้าแสดงเมนูอาหาร
struct MenuView: View {
    let tableNumber: Int?
    
    var body: some View {
        List {
            Section("เมนูอาหารจานหลัก") {
                MenuItemRow(name: "ข้าวผัดกุ้ง", price: 120)
                MenuItemRow(name: "ต้มยำกุ้ง", price: 180)
                MenuItemRow(name: "ผัดไทย", price: 100)
            }
        }
        .navigationTitle(tableNumber.map { "โต๊ะ \($0)" } ?? "เมนู")
    }
}

struct MenuItemRow: View {
    let name: String
    let price: Int
    
    var body: some View {
        HStack {
            Text(name)
            Spacer()
            Text("฿\(price)")
                .foregroundColor(.secondary)
        }
    }
}
```

---

## 3. App Clip Invocations

### ประเภทของการเปิด App Clips

#### QR Code
ผู้ใช้สแกน QR Code ที่มี URL เช่น `https://restaurant.example.com/menu?table=5` ระบบจะตรวจสอบว่า URL ตรงกับ App Clip ที่ลงทะเบียนไว้หรือไม่

#### NFC Tag
ร้านค้าสามารถวาง NFC tag ที่โต๊ะหรือสินค้า เมื่อผู้ใช้แตะโทรศัพท์กับ tag ระบบจะเปิด App Clip โดยอัตโนมัติ

#### Safari Smart App Banner
เพิ่ม `<meta>` tag ใน HTML เพื่อแสดง Smart App Banner ที่ด้านบนของ Safari

```html
<!-- เพิ่มใน <head> ของ HTML page -->
<!-- Smart App Banner สำหรับ App Clip -->
<meta 
    name="apple-itunes-app" 
    content="app-id=1234567890, 
             app-clip-bundle-id=com.example.restaurant.clip,
             app-clip-display=card,
             affiliate-data=at=1l3v8L&ct=website_banner"
>

<!-- ตัวอย่างการใช้งานจริงในหน้าเมนูร้านอาหาร -->
<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <!-- Smart App Banner - แสดงบน Safari iOS -->
    <meta name="apple-itunes-app" 
          content="app-id=1234567890,
                   app-clip-bundle-id=com.example.restaurant.clip">
    <title>เมนูร้านอาหาร</title>
</head>
<body>
    <h1>ยินดีต้อนรับสู่ร้านอาหาร</h1>
    <p>สแกน QR Code หรือคลิก banner ด้านบนเพื่อสั่งอาหาร</p>
</body>
</html>
```

#### Messages (iMessage)
เมื่อผู้ใช้แชร์ URL ที่ลงทะเบียนเป็น App Clip ใน iMessage ระบบจะแสดง App Clip card ในบทสนทนา

#### Place Cards ใน Maps
ธุรกิจสามารถลงทะเบียน App Clip กับ Apple Maps ผ่าน **App Store Connect** เพื่อให้ App Clip card ปรากฏใน Place Card ของสถานที่

### การกำหนด Associated Domains

ต้องเพิ่ม **Associated Domains** ใน Entitlements ของ App Clip:

```xml
<!-- MyRestaurantAppClip.entitlements -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" 
    "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>com.apple.developer.associated-domains</key>
    <array>
        <!-- appclips: prefix สำหรับ App Clips -->
        <string>appclips:restaurant.example.com</string>
    </array>
</dict>
</plist>
```

และต้องมีไฟล์ `apple-app-site-association` บน server:

```json
{
    "appclips": {
        "apps": ["TEAMID1234.com.example.restaurant.clip"]
    }
}
```

---

## 4. App Clip Limitations & Ephemeral Notifications

### ข้อจำกัดของ App Clips

App Clips มีข้อจำกัดหลายอย่างเมื่อเทียบกับแอปเต็มรูปแบบ:

#### ไม่มี Background Refresh
App Clips ไม่สามารถรับ `BGAppRefreshTask` หรือ `BGProcessingTask` ได้ ต้องทำงานเฉพาะเมื่อผู้ใช้เปิดใช้งานเท่านั้น

#### การเข้าถึง Keychain ที่จำกัด
App Clips ไม่สามารถอ่านข้อมูลจาก Keychain ของแอปหลักได้ (และในทางกลับกัน) แต่สามารถแชร์ข้อมูลได้ผ่าน App Groups หลังจากที่ผู้ใช้ติดตั้งแอปเต็มรูปแบบ

#### ข้อมูลชั่วคราว
ข้อมูลใน App Clip จะถูกลบหลังจาก 30 วันที่ไม่ได้ใช้งาน

#### ไม่มี iCloud Sync
App Clips ไม่รองรับ CloudKit หรือ iCloud Documents

### Ephemeral Notifications

App Clips สามารถขอ **Ephemeral Notifications** ได้โดยไม่ต้องขอ permission จากผู้ใช้อย่างเป็นทางการ แต่การแจ้งเตือนนี้จะทำงานได้เพียง 8 ชั่วโมงหลังจากใช้ App Clip ครั้งแรก

```swift
// การขอ Ephemeral Notifications
import UserNotifications

func requestEphemeralNotifications() async {
    let center = UNUserNotificationCenter.current()
    
    do {
        // สำหรับ App Clips ใช้ .ephemeral แทน .alert, .badge, .sound
        let granted = try await center.requestAuthorization(
            options: [.alert, .sound, .ephemeral]
        )
        
        if granted {
            print("ได้รับอนุญาต Ephemeral Notifications")
        }
    } catch {
        print("เกิดข้อผิดพลาด: \(error)")
    }
}
```

### SKOverlay เพื่อแนะนำให้ติดตั้งแอปเต็มรูปแบบ

`SKOverlay` แสดง banner ที่ด้านล่างของหน้าจอเพื่อให้ผู้ใช้ดาวน์โหลดแอปเต็มรูปแบบหลังจากทำ task เสร็จแล้ว

```swift
// OrderCompletionView.swift
import SwiftUI
import StoreKit

struct OrderCompletionView: View {
    @State private var showOverlay = false
    let orderId: String
    
    var body: some View {
        VStack(spacing: 20) {
            Image(systemName: "checkmark.circle.fill")
                .font(.system(size: 80))
                .foregroundColor(.green)
            
            Text("สั่งอาหารสำเร็จแล้ว!")
                .font(.title)
                .fontWeight(.bold)
            
            Text("หมายเลขออร์เดอร์: \(orderId)")
                .foregroundColor(.secondary)
            
            Text("ดาวน์โหลดแอปเพื่อติดตามสถานะออร์เดอร์")
                .multilineTextAlignment(.center)
                .padding()
        }
        .padding()
        .onAppear {
            // รอสักครู่แล้วแสดง SKOverlay
            DispatchQueue.main.asyncAfter(deadline: .now() + 2) {
                showOverlay = true
            }
        }
        // SKOverlay modifier
        .appStoreOverlay(isPresented: $showOverlay) {
            // กำหนด App Store ID ของแอปเต็มรูปแบบ
            SKOverlay.AppClipConfiguration(position: .bottom)
        }
    }
}

// SKOverlay แบบ programmatic (UIKit)
class OrderCompleteViewController: UIViewController, SKOverlayDelegate {
    func showAppInstallBanner() {
        guard let windowScene = view.window?.windowScene else { return }
        let overlay = SKOverlay(configuration: SKOverlay.AppClipConfiguration(position: .bottom))
        overlay.delegate = self
        overlay.present(in: windowScene)
    }
    func storeOverlayDidFinishDismissal(_ overlay: SKOverlay, transitionContext: SKOverlay.TransitionContext) {
        print("ผู้ใช้ปิด overlay")
    }
}
```

---

## 5. Testing App Clips

### การทดสอบ App Clips ใน Xcode (Local Experience)

ก่อนที่จะ submit ขึ้น App Store เราสามารถทดสอบ App Clip ได้บนเครื่องจริง ผ่าน **Local Experience** โดยไม่ต้อง configure บน App Store Connect

ขั้นตอน:
1. ไปที่ **Settings → Developer** บนอุปกรณ์ iOS
2. เลื่อนลงมาที่ **Local Experiences**
3. กด **Register Local Experience**
4. กรอก URL Prefix ที่ต้องการทดสอบ
5. เลือก App Clip Bundle ID
6. กำหนด title, subtitle, call to action
7. สแกน QR Code ที่มี URL นั้น หรือพิมพ์ URL ใน Safari

### การใช้ Environment Variable ใน Scheme

ใน Xcode สามารถตั้งค่า `_XCAppClipURL` เพื่อจำลอง URL ที่ถูกส่งมาเมื่อ App Clip ถูกเปิด:

```swift
// AppClipApp.swift
import SwiftUI

@main
struct AppClipApp: App {
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .onContinueUserActivity(
                    NSUserActivityTypes.browsingWeb,
                    perform: handleActivity
                )
                .onAppear {
                    // ใน Debug mode ใช้ environment variable
                    #if DEBUG
                    checkDebugURL()
                    #endif
                }
        }
    }
    
    func handleActivity(_ activity: NSUserActivity) {
        guard let url = activity.webpageURL else { return }
        processInvocationURL(url)
    }
    
    #if DEBUG
    func checkDebugURL() {
        // _XCAppClipURL ถูกตั้งค่าผ่าน Xcode Scheme
        // Edit Scheme → Run → Environment Variables
        // Name: _XCAppClipURL
        // Value: https://restaurant.example.com/menu?table=3
        if let urlString = ProcessInfo.processInfo
            .environment["_XCAppClipURL"],
           let url = URL(string: urlString) {
            print("Debug App Clip URL: \(url)")
            processInvocationURL(url)
        }
    }
    #endif
    
    func processInvocationURL(_ url: URL) {
        let components = URLComponents(
            url: url,
            resolvingAgainstBaseURL: true
        )
        print("App Clip เปิดด้วย URL: \(url.absoluteString)")
        // จัดการ URL parameters ตามต้องการ
    }
}
```

การตั้งค่า Scheme สำหรับ testing:

```
Xcode Menu:
Product → Scheme → Edit Scheme
→ Run → Environment Variables
→ เพิ่ม: _XCAppClipURL = https://restaurant.example.com/menu?table=5
```

---

## 6. Share Extension

### Share Extension คืออะไร?

**Share Extension** ช่วยให้ผู้ใช้แชร์ content (URL, ข้อความ, รูปภาพ) จากแอปอื่นมายังแอปของเราได้ โดยปุ่ม Share Sheet ใน iOS จะแสดงแอปของเราเป็นตัวเลือก

### NSExtensionItem และ Attachments

ข้อมูลที่แชร์มาจะอยู่ใน `NSExtensionItem` และ `NSItemProvider` ซึ่งเราต้องดึงออกมาแบบ async

```swift
// ShareViewController.swift
import UIKit
import Social
import UniformTypeIdentifiers

class ShareViewController: UIViewController {
    
    // UI Elements
    private let titleLabel = UILabel()
    private let textView = UITextView()
    private let saveButton = UIButton(type: .system)
    
    private var sharedURL: URL?
    private var sharedText: String?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
        loadSharedContent()
    }
    
    private func setupUI() {
        view.backgroundColor = .systemBackground
        title = "บันทึกลิงก์"
        
        navigationItem.leftBarButtonItem = UIBarButtonItem(
            barButtonSystemItem: .cancel,
            target: self,
            action: #selector(cancelTapped)
        )
        
        navigationItem.rightBarButtonItem = UIBarButtonItem(
            title: "บันทึก",
            style: .done,
            target: self,
            action: #selector(saveTapped)
        )
    }
    
    private func loadSharedContent() {
        // ดึงข้อมูลจาก NSExtensionContext
        guard let inputItems = extensionContext?.inputItems as? [NSExtensionItem] else {
            return
        }
        
        for item in inputItems {
            guard let attachments = item.attachments else { continue }
            
            for provider in attachments {
                // โหลด URL
                if provider.hasItemConformingToTypeIdentifier(UTType.url.identifier) {
                    provider.loadItem(forTypeIdentifier: UTType.url.identifier, options: nil) {
                        [weak self] item, _ in
                        if let url = item as? URL {
                            DispatchQueue.main.async {
                                self?.sharedURL = url
                                self?.textView.text = url.absoluteString
                            }
                        }
                    }
                }
                // โหลด plain text
                if provider.hasItemConformingToTypeIdentifier(UTType.plainText.identifier) {
                    provider.loadItem(forTypeIdentifier: UTType.plainText.identifier, options: nil) {
                        [weak self] item, _ in
                        if let text = item as? String {
                            DispatchQueue.main.async {
                                let existing = self?.textView.text ?? ""
                                self?.sharedText = text
                                self?.textView.text = existing.isEmpty ? text : "\(existing)\n\(text)"
                            }
                        }
                    }
                }
            }
        }
    }
    
    @objc private func saveTapped() {
        let defaults = UserDefaults(suiteName: "group.com.example.myapp")
        var savedLinks = defaults?.array(forKey: "savedLinks") as? [[String: String]] ?? []
        var newLink: [String: String] = ["timestamp": ISO8601DateFormatter().string(from: Date())]
        if let url = sharedURL { newLink["url"] = url.absoluteString }
        if let text = sharedText { newLink["text"] = text }
        savedLinks.append(newLink)
        defaults?.set(savedLinks, forKey: "savedLinks")
        extensionContext?.completeRequest(returningItems: nil, completionHandler: nil)
    }
    
    @objc private func cancelTapped() {
        extensionContext?.cancelRequest(
            withError: NSError(domain: "UserCancelled", code: 0, userInfo: nil)
        )
    }
}
```

### Info.plist สำหรับ Share Extension

```xml
<!-- ShareExtension/Info.plist -->
<key>NSExtension</key>
<dict>
    <key>NSExtensionAttributes</key>
    <dict>
        <!-- ประเภทของ content ที่รับได้ -->
        <key>NSExtensionActivationRule</key>
        <dict>
            <key>NSExtensionActivationSupportsWebURLWithMaxCount</key>
            <integer>1</integer>
            <key>NSExtensionActivationSupportsText</key>
            <true/>
        </dict>
    </dict>
    <key>NSExtensionMainStoryboard</key>
    <string>MainInterface</string>
    <key>NSExtensionPointIdentifier</key>
    <string>com.apple.share-services</string>
</dict>
```

---

## 7. Action Extension

### Action Extension คืออะไร?

**Action Extension** ช่วยให้แอปของเราประมวลผล content จากแอปอื่น เช่น แก้ไขรูปภาพ, แปลงไฟล์, หรือเพิ่ม watermark ความแตกต่างจาก Share Extension คือ Action Extension ส่ง content กลับไปยังแอปต้นทางได้

### การสร้าง Image Watermark Action Extension

```swift
// WatermarkActionViewController.swift
import UIKit
import MobileCoreServices
import UniformTypeIdentifiers

class WatermarkActionViewController: UIViewController {
    
    @IBOutlet weak var imageView: UIImageView!
    @IBOutlet weak var watermarkTextField: UITextField!
    
    private var originalImage: UIImage?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupNavigation()
        loadInputImage()
    }
    
    private func setupNavigation() {
        title = "เพิ่ม Watermark"
        
        navigationItem.leftBarButtonItem = UIBarButtonItem(
            barButtonSystemItem: .cancel,
            target: self,
            action: #selector(cancelTapped)
        )
        
        navigationItem.rightBarButtonItem = UIBarButtonItem(
            title: "ใช้งาน",
            style: .done,
            target: self,
            action: #selector(applyTapped)
        )
    }
    
    private func loadInputImage() {
        guard let providers = (extensionContext?.inputItems as? [NSExtensionItem])?.first?.attachments
        else { return }
        
        for provider in providers {
            guard provider.hasItemConformingToTypeIdentifier(UTType.image.identifier) else { continue }
            provider.loadItem(forTypeIdentifier: UTType.image.identifier, options: nil) {
                [weak self] item, _ in
                let image: UIImage?
                if let url = item as? URL { image = UIImage(contentsOfFile: url.path) }
                else if let img = item as? UIImage { image = img }
                else if let data = item as? Data { image = UIImage(data: data) }
                else { image = nil }
                DispatchQueue.main.async { self?.originalImage = image; self?.imageView.image = image }
            }
            break
        }
    }
    
    @objc private func applyTapped() {
        guard let image = originalImage else { return }
        let watermarkedImage = addWatermark(to: image, text: watermarkTextField.text ?? "© My App")
        let output = NSExtensionItem()
        output.attachments = [NSItemProvider(object: watermarkedImage)]
        extensionContext?.completeRequest(returningItems: [output], completionHandler: nil)
    }
    
    private func addWatermark(to image: UIImage, text: String) -> UIImage {
        UIGraphicsImageRenderer(size: image.size).image { _ in
            image.draw(in: CGRect(origin: .zero, size: image.size))
            let attrs: [NSAttributedString.Key: Any] = [
                .font: UIFont.boldSystemFont(ofSize: image.size.width * 0.05),
                .foregroundColor: UIColor.white.withAlphaComponent(0.7),
                .strokeColor: UIColor.black.withAlphaComponent(0.5),
                .strokeWidth: -2.0
            ]
            let size = text.size(withAttributes: attrs)
            let margin: CGFloat = 20
            text.draw(
                in: CGRect(x: image.size.width - size.width - margin,
                           y: image.size.height - size.height - margin,
                           width: size.width, height: size.height),
                withAttributes: attrs
            )
        }
    }
    
    @objc private func cancelTapped() {
        extensionContext?.cancelRequest(
            withError: NSError(domain: "com.example.WatermarkAction", code: NSUserCancelledError)
        )
    }
}
```

---

## 8. Custom Keyboard Extension

### UIInputViewController

**Custom Keyboard Extension** ช่วยให้เราสร้าง keyboard แบบกำหนดเองได้ โดย subclass `UIInputViewController`

### ข้อจำกัดของ Custom Keyboard

- ไม่สามารถเข้าถึงกล้องหรือ microphone โดยตรง
- ไม่สามารถแสดง alert ที่เด่นกว่า keyboard ได้
- ต้องมีปุ่ม "Globe" เพื่อให้ผู้ใช้เปลี่ยน keyboard ได้
- หาก `RequestsOpenAccess` เปิดอยู่ จะเข้าถึง network ได้

```swift
// EmojiKeyboardViewController.swift
import UIKit

class EmojiKeyboardViewController: UIInputViewController {
    
    // รายการ emoji ที่แสดงใน keyboard
    private let emojis: [[String]] = [
        ["😀", "😂", "🥰", "😎", "🤔", "😴", "🥺", "😡"],
        ["👍", "👎", "🙌", "👏", "🤝", "💪", "🖐️", "✌️"],
        ["❤️", "🧡", "💛", "💚", "💙", "💜", "🖤", "🤍"],
        ["🍕", "🍔", "🍜", "🍣", "🍱", "🍰", "☕", "🍺"],
        ["⚽", "🏀", "🎮", "🎵", "📚", "💻", "📱", "🎨"]
    ]
    
    private var collectionView: UICollectionView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupCollectionView()
        setupDeleteButton()
        setupNextKeyboardButton()
    }
    
    private func setupCollectionView() {
        let layout = UICollectionViewFlowLayout()
        layout.itemSize = CGSize(width: 44, height: 44)
        layout.minimumInteritemSpacing = 4
        layout.minimumLineSpacing = 4
        layout.sectionInset = UIEdgeInsets(
            top: 8, left: 8, bottom: 8, right: 8
        )
        
        collectionView = UICollectionView(
            frame: .zero,
            collectionViewLayout: layout
        )
        collectionView.backgroundColor = .systemGroupedBackground
        collectionView.register(
            EmojiCell.self,
            forCellWithReuseIdentifier: "EmojiCell"
        )
        collectionView.dataSource = self
        collectionView.delegate = self
        collectionView.translatesAutoresizingMaskIntoConstraints = false
        
        view.addSubview(collectionView)
        
        NSLayoutConstraint.activate([
            collectionView.topAnchor.constraint(equalTo: view.topAnchor),
            collectionView.leadingAnchor.constraint(
                equalTo: view.leadingAnchor
            ),
            collectionView.trailingAnchor.constraint(
                equalTo: view.trailingAnchor
            ),
            collectionView.heightAnchor.constraint(equalToConstant: 280)
        ])
        
        view.heightAnchor.constraint(equalToConstant: 320).isActive = true
    }
    
    private func setupDeleteButton() {
        let deleteButton = UIButton(type: .system)
        deleteButton.setTitle("⌫", for: .normal)
        deleteButton.titleLabel?.font = .systemFont(ofSize: 20)
        deleteButton.addTarget(
            self,
            action: #selector(deleteTapped),
            for: .touchUpInside
        )
        deleteButton.translatesAutoresizingMaskIntoConstraints = false
        
        view.addSubview(deleteButton)
        
        NSLayoutConstraint.activate([
            deleteButton.trailingAnchor.constraint(
                equalTo: view.trailingAnchor, constant: -16
            ),
            deleteButton.bottomAnchor.constraint(
                equalTo: view.bottomAnchor, constant: -8
            ),
            deleteButton.widthAnchor.constraint(equalToConstant: 44),
            deleteButton.heightAnchor.constraint(equalToConstant: 44)
        ])
    }
    
    private func setupNextKeyboardButton() {
        let nextButton = UIButton(type: .system)
        nextButton.setTitle("🌐", for: .normal)
        nextButton.titleLabel?.font = .systemFont(ofSize: 20)
        nextButton.addTarget(
            self,
            action: #selector(handleInputModeList(from:with:)),
            for: .allTouchEvents
        )
        nextButton.translatesAutoresizingMaskIntoConstraints = false
        
        view.addSubview(nextButton)
        
        NSLayoutConstraint.activate([
            nextButton.leadingAnchor.constraint(
                equalTo: view.leadingAnchor, constant: 16
            ),
            nextButton.bottomAnchor.constraint(
                equalTo: view.bottomAnchor, constant: -8
            ),
            nextButton.widthAnchor.constraint(equalToConstant: 44),
            nextButton.heightAnchor.constraint(equalToConstant: 44)
        ])
    }
    
    @objc private func deleteTapped() {
        textDocumentProxy.deleteBackward()
    }
    
    private func insertEmoji(_ emoji: String) {
        textDocumentProxy.insertText(emoji)
    }
}

// MARK: - UICollectionViewDataSource
extension EmojiKeyboardViewController: UICollectionViewDataSource {
    
    func numberOfSections(in collectionView: UICollectionView) -> Int {
        return emojis.count
    }
    
    func collectionView(
        _ collectionView: UICollectionView,
        numberOfItemsInSection section: Int
    ) -> Int {
        return emojis[section].count
    }
    
    func collectionView(
        _ collectionView: UICollectionView,
        cellForItemAt indexPath: IndexPath
    ) -> UICollectionViewCell {
        let cell = collectionView.dequeueReusableCell(
            withReuseIdentifier: "EmojiCell",
            for: indexPath
        ) as! EmojiCell
        
        cell.configure(with: emojis[indexPath.section][indexPath.item])
        return cell
    }
}

// MARK: - UICollectionViewDelegate
extension EmojiKeyboardViewController: UICollectionViewDelegate {
    
    func collectionView(
        _ collectionView: UICollectionView,
        didSelectItemAt indexPath: IndexPath
    ) {
        let emoji = emojis[indexPath.section][indexPath.item]
        insertEmoji(emoji)
    }
}

// MARK: - EmojiCell (UICollectionViewCell subclass)
class EmojiCell: UICollectionViewCell {
    private let label = UILabel()
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        label.font = .systemFont(ofSize: 28)
        label.textAlignment = .center
        label.translatesAutoresizingMaskIntoConstraints = false
        contentView.addSubview(label)
        NSLayoutConstraint.activate([
            label.centerXAnchor.constraint(equalTo: contentView.centerXAnchor),
            label.centerYAnchor.constraint(equalTo: contentView.centerYAnchor)
        ])
        contentView.layer.cornerRadius = 8
        contentView.backgroundColor = .systemBackground
    }
    required init?(coder: NSCoder) { fatalError("init(coder:) has not been implemented") }
    func configure(with emoji: String) { label.text = emoji }
    override var isHighlighted: Bool {
        didSet { contentView.backgroundColor = isHighlighted ? .systemGray4 : .systemBackground }
    }
}
```

---

## 9. Notification Service Extension

### UNNotificationServiceExtension

**Notification Service Extension** ช่วยให้เราประมวลผล push notification ก่อนที่จะแสดงให้ผู้ใช้เห็น ใช้สำหรับ:

- ดาวน์โหลดและแนบรูปภาพ/วิดีโอ (**Rich Push Notifications**)
- ถอดรหัส payload ที่เข้ารหัสไว้
- แก้ไข title/body ของ notification
- บันทึก notification ลงฐานข้อมูล

### Rich Push Notification ด้วยการดาวน์โหลดรูปภาพ

```swift
// NotificationService.swift
import UserNotifications

class NotificationService: UNNotificationServiceExtension {
    
    // contentHandler และ bestAttemptContent เก็บไว้
    // เพื่อให้ใช้ใน serviceExtensionTimeWillExpire()
    var contentHandler: ((UNNotificationContent) -> Void)?
    var bestAttemptContent: UNMutableNotificationContent?
    
    override func didReceive(
        _ request: UNNotificationRequest,
        withContentHandler contentHandler: @escaping (UNNotificationContent) -> Void
    ) {
        self.contentHandler = contentHandler
        
        // สร้าง mutable copy ของ content
        guard let mutableContent = request.content.mutableCopy()
            as? UNMutableNotificationContent else {
            contentHandler(request.content)
            return
        }
        
        self.bestAttemptContent = mutableContent
        
        // ดึง URL ของรูปภาพจาก payload
        // payload ควรมี key "image-url" ที่ระดับ root
        guard let imageURLString = mutableContent
            .userInfo["image-url"] as? String,
              let imageURL = URL(string: imageURLString) else {
            // ไม่มีรูปภาพ ส่ง content ปกติ
            contentHandler(mutableContent)
            return
        }
        
        // ดาวน์โหลดรูปภาพ
        downloadAndAttachImage(
            from: imageURL,
            to: mutableContent,
            contentHandler: contentHandler
        )
    }
    
    private func downloadAndAttachImage(
        from url: URL,
        to content: UNMutableNotificationContent,
        contentHandler: @escaping (UNNotificationContent) -> Void
    ) {
        let task = URLSession.shared.downloadTask(with: url) { 
            tempURL, response, error in
            
            defer {
                // เรียก contentHandler เสมอ ไม่ว่าจะสำเร็จหรือไม่
                contentHandler(content)
            }
            
            guard error == nil, let tempURL = tempURL else {
                print("ดาวน์โหลดรูปภาพล้มเหลว: \(error?.localizedDescription ?? "unknown")")
                return
            }
            
            // ย้ายไฟล์ชั่วคราวไปยัง location ถาวร
            let fileManager = FileManager.default
            
            // กำหนด extension จาก MIME type
            let ext = self.fileExtension(from: response)
            let permanentURL = tempURL.deletingPathExtension()
                .appendingPathExtension(ext)
            
            do {
                if fileManager.fileExists(atPath: permanentURL.path) {
                    try fileManager.removeItem(at: permanentURL)
                }
                try fileManager.moveItem(at: tempURL, to: permanentURL)
                
                // สร้าง UNNotificationAttachment
                let attachment = try UNNotificationAttachment(
                    identifier: "image-attachment",
                    url: permanentURL,
                    options: [
                        // thumbnail clipping rect (optional)
                        UNNotificationAttachmentOptionsThumbnailClippingRectKey:
                            CGRect(x: 0, y: 0, width: 1, height: 0.5).dictionaryRepresentation
                    ]
                )
                
                content.attachments = [attachment]
                print("แนบรูปภาพสำเร็จ")
                
            } catch {
                print("เกิดข้อผิดพลาดในการแนบรูปภาพ: \(error)")
            }
        }
        
        task.resume()
    }
    
    private func fileExtension(from response: URLResponse?) -> String {
        guard let mimeType = response?.mimeType else { return "jpg" }
        
        switch mimeType {
        case "image/jpeg": return "jpg"
        case "image/png": return "png"
        case "image/gif": return "gif"
        case "image/webp": return "webp"
        case "video/mp4": return "mp4"
        default: return "jpg"
        }
    }
    
    // เรียกเมื่อเวลาหมด (ประมาณ 30 วินาที)
    // ต้องส่ง content ทันที
    override func serviceExtensionTimeWillExpire() {
        if let contentHandler = contentHandler,
           let content = bestAttemptContent {
            // ส่ง content ที่ดีที่สุดที่มีอยู่ในตอนนี้
            contentHandler(content)
        }
    }
}
```

### APNs Payload สำหรับ Rich Push

Push notification payload ที่ server ต้องส่ง:

```json
{
    "aps": {
        "alert": {
            "title": "โปรโมชั่นพิเศษ!",
            "body": "ลด 50% สำหรับเมนูพิเศษวันนี้"
        },
        "mutable-content": 1,
        "sound": "default",
        "badge": 1
    },
    "image-url": "https://example.com/promo-image.jpg",
    "action-url": "https://restaurant.example.com/promo/daily"
}
```

---

## 10. App Intents (iOS 16+)

### App Intents Framework

**App Intents** เป็น framework ใหม่ที่ใช้แทน **Intents Extension** แบบเก่า ช่วยให้แอปทำงานร่วมกับ **Siri**, **Shortcuts**, **Spotlight** และ **Widget** ได้

### AppIntent Protocol

```swift
// ShoppingListIntents.swift
import AppIntents
import SwiftUI

// MARK: - Model
struct ShoppingItem: Identifiable, Equatable {
    let id: UUID
    var name: String
    var quantity: Int
    var category: String
    
    init(id: UUID = UUID(), name: String, quantity: Int = 1, category: String = "ทั่วไป") {
        self.id = id
        self.name = name
        self.quantity = quantity
        self.category = category
    }
}

// MARK: - AddToShoppingList Intent
struct AddToShoppingListIntent: AppIntent {
    
    // ชื่อที่แสดงใน Shortcuts app
    static var title: LocalizedStringResource = "เพิ่มสินค้าในรายการ"
    
    // คำอธิบาย
    static var description = IntentDescription(
        "เพิ่มสินค้าเข้าในรายการช้อปปิ้ง",
        categoryName: "รายการช้อปปิ้ง"
    )
    
    // Parameters ที่ Siri/Shortcuts จะถาม
    @Parameter(title: "ชื่อสินค้า", description: "ระบุชื่อสินค้าที่ต้องการซื้อ")
    var itemName: String
    
    @Parameter(
        title: "จำนวน",
        description: "ระบุจำนวนที่ต้องการ",
        default: 1,
        inclusiveRange: (1, 99)
    )
    var quantity: Int
    
    @Parameter(
        title: "หมวดหมู่",
        description: "เลือกหมวดหมู่ของสินค้า",
        default: "ทั่วไป"
    )
    var category: String
    
    // ฟังก์ชันหลักที่ทำงานเมื่อ intent ถูกเรียก
    func perform() async throws -> some IntentResult & ProvidesDialog {
        // เพิ่มสินค้าลงใน shopping list
        let newItem = ShoppingItem(
            name: itemName,
            quantity: quantity,
            category: category
        )
        
        await ShoppingListStore.shared.add(newItem)
        
        // ส่ง feedback กลับให้ Siri พูด
        return .result(
            dialog: IntentDialog(
                "เพิ่ม \(itemName) จำนวน \(quantity) ชิ้น ลงในรายการแล้ว"
            )
        )
    }
}

// MARK: - ClearShoppingList Intent
struct ClearShoppingListIntent: AppIntent {
    
    static var title: LocalizedStringResource = "ล้างรายการช้อปปิ้ง"
    
    static var description = IntentDescription(
        "ลบสินค้าทั้งหมดออกจากรายการช้อปปิ้ง"
    )
    
    // ยืนยันก่อนดำเนินการ
    static var authenticationPolicy = IntentAuthenticationPolicy.requiresAuthentication
    
    func perform() async throws -> some IntentResult & ProvidesDialog {
        await ShoppingListStore.shared.clearAll()
        return .result(dialog: "ล้างรายการช้อปปิ้งทั้งหมดแล้ว")
    }
}

// MARK: - App Shortcuts Provider
struct ShoppingAppShortcuts: AppShortcutsProvider {
    
    // กำหนด App Shortcuts ที่ Siri รู้จักโดยอัตโนมัติ
    static var appShortcuts: [AppShortcut] {
        AppShortcut(
            intent: AddToShoppingListIntent(),
            phrases: [
                // วลีที่ Siri ฟัง
                "เพิ่ม \(\.$itemName) ในรายการช้อปปิ้งใน \(.applicationName)",
                "บันทึก \(\.$itemName) ใน \(.applicationName)",
                "ซื้อ \(\.$itemName) ใน \(.applicationName)"
            ],
            shortTitle: "เพิ่มสินค้า",
            systemImageName: "cart.badge.plus"
        )
    }
}

// MARK: - Store (actor สำหรับ thread-safe access)
actor ShoppingListStore {
    static let shared = ShoppingListStore()
    private var items: [ShoppingItem] = []
    
    func add(_ item: ShoppingItem) {
        items.append(item)
        // บันทึกลง App Group UserDefaults เพื่อแชร์กับ Widget
        UserDefaults(suiteName: "group.com.example.shoppingapp")?
            .set(items.map { ["id": $0.id.uuidString, "name": $0.name, "qty": $0.quantity] },
                 forKey: "shoppingItems")
    }
    
    func clearAll() { items.removeAll() }
    func getAll() -> [ShoppingItem] { items }
}
```

---

## 11. Background Tasks

### BGTaskScheduler

**Background Tasks** ช่วยให้แอปทำงานเบื้องหลังได้ในเวลาที่เหมาะสม iOS จะจัดสรรเวลาให้ตามการใช้งานของผู้ใช้และสถานะของอุปกรณ์

### ประเภทของ Background Tasks

| ประเภท | คำอธิบาย | เวลาที่ได้ |
|--------|----------|-----------|
| `BGAppRefreshTask` | อัปเดตข้อมูลเบื้องหลัง | ~30 วินาที |
| `BGProcessingTask` | งานหนักที่ต้องการเวลา | หลายนาที |

### การลงทะเบียนและกำหนดเวลา Background Tasks

```swift
// AppDelegate.swift
import UIKit
import BackgroundTasks

@main
class AppDelegate: UIResponder, UIApplicationDelegate {
    
    // Background Task Identifiers (ต้องลงทะเบียนใน Info.plist ด้วย)
    static let refreshTaskIdentifier = "com.example.app.refresh"
    static let processingTaskIdentifier = "com.example.app.processing"
    
    func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
    ) -> Bool {
        
        // ลงทะเบียน task handlers
        BGTaskScheduler.shared.register(
            forTaskWithIdentifier: AppDelegate.refreshTaskIdentifier,
            using: nil
        ) { task in
            self.handleAppRefresh(task: task as! BGAppRefreshTask)
        }
        
        BGTaskScheduler.shared.register(
            forTaskWithIdentifier: AppDelegate.processingTaskIdentifier,
            using: nil
        ) { task in
            self.handleDataProcessing(task: task as! BGProcessingTask)
        }
        
        return true
    }
    
    // MARK: - Handle BGAppRefreshTask
    private func handleAppRefresh(task: BGAppRefreshTask) {
        // กำหนด task ต่อไปให้ทำงาน
        scheduleAppRefresh()
        
        // สร้าง operation สำหรับ sync ข้อมูล
        let syncOperation = DataSyncOperation()
        
        // กำหนด expiration handler
        task.expirationHandler = {
            syncOperation.cancel()
        }
        
        syncOperation.completionBlock = {
            let success = !syncOperation.isCancelled
            task.setTaskCompleted(success: success)
            print("App Refresh Task เสร็จสิ้น: \(success ? "สำเร็จ" : "ยกเลิก")")
        }
        
        OperationQueue.main.addOperation(syncOperation)
    }
    
    // MARK: - Handle BGProcessingTask  
    private func handleDataProcessing(task: BGProcessingTask) {
        let heavyOperation = HeavyDataProcessingOperation()
        
        task.expirationHandler = {
            heavyOperation.cancel()
        }
        
        heavyOperation.completionBlock = {
            task.setTaskCompleted(success: !heavyOperation.isCancelled)
        }
        
        let queue = OperationQueue()
        queue.maxConcurrentOperationCount = 1
        queue.addOperation(heavyOperation)
    }
    
    // MARK: - Scheduling
    func scheduleAppRefresh() {
        let request = BGAppRefreshTaskRequest(
            identifier: AppDelegate.refreshTaskIdentifier
        )
        // กำหนดเวลาเร็วที่สุดที่ task จะทำงาน (ไม่ได้การันตีว่าจะทำงานตรงเวลา)
        request.earliestBeginDate = Date(timeIntervalSinceNow: 15 * 60) // 15 นาที
        
        do {
            try BGTaskScheduler.shared.submit(request)
            print("กำหนด App Refresh Task สำเร็จ")
        } catch {
            print("ไม่สามารถกำหนด task: \(error)")
        }
    }
    
    func scheduleBackgroundProcessing() {
        let request = BGProcessingTaskRequest(
            identifier: AppDelegate.processingTaskIdentifier
        )
        request.earliestBeginDate = Date(timeIntervalSinceNow: 60 * 60) // 1 ชั่วโมง
        request.requiresNetworkConnectivity = true  // ต้องการ network
        request.requiresExternalPower = false       // ไม่ต้องชาร์จอยู่
        
        do {
            try BGTaskScheduler.shared.submit(request)
            print("กำหนด Processing Task สำเร็จ")
        } catch {
            print("ไม่สามารถกำหนด processing task: \(error)")
        }
    }
    
    // เรียกเมื่อแอปเข้า background
    func applicationDidEnterBackground(_ application: UIApplication) {
        scheduleAppRefresh()
        scheduleBackgroundProcessing()
    }
}

// MARK: - Operations
class DataSyncOperation: Operation {
    
    override func main() {
        guard !isCancelled else { return }
        
        print("เริ่ม sync ข้อมูล...")
        
        // จำลองการ sync ข้อมูล
        let semaphore = DispatchSemaphore(value: 0)
        
        URLSession.shared.dataTask(
            with: URL(string: "https://api.example.com/sync")!
        ) { data, response, error in
            if let data = data {
                // บันทึกข้อมูลลง local storage
                UserDefaults.standard.set(data, forKey: "lastSyncData")
                UserDefaults.standard.set(Date(), forKey: "lastSyncDate")
                print("Sync สำเร็จ: \(data.count) bytes")
            }
            semaphore.signal()
        }.resume()
        
        semaphore.wait()
    }
}

class HeavyDataProcessingOperation: Operation {
    override func main() {
        guard !isCancelled else { return }
        print("เริ่มประมวลผลข้อมูลหนัก...")
        cleanOldCacheFiles()
    }
    
    private func cleanOldCacheFiles() {
        let cacheDir = FileManager.default.urls(for: .cachesDirectory, in: .userDomainMask).first!
        let cutoffDate = Date(timeIntervalSinceNow: -7 * 24 * 60 * 60) // 7 วัน
        
        guard let files = try? FileManager.default.contentsOfDirectory(
            at: cacheDir, includingPropertiesForKeys: [.creationDateKey]
        ) else { return }
        
        for file in files {
            guard !isCancelled else { return }
            if let created = try? file.resourceValues(forKeys: [.creationDateKey]).creationDate,
               created < cutoffDate {
                try? FileManager.default.removeItem(at: file)
            }
        }
    }
}
```

### Info.plist สำหรับ Background Tasks

```xml
<!-- Info.plist -->
<key>BGTaskSchedulerPermittedIdentifiers</key>
<array>
    <string>com.example.app.refresh</string>
    <string>com.example.app.processing</string>
</array>
```

---

## 12. Exercises

### แบบฝึกหัดที่ 1: App Clip สำหรับ Event Check-in ด้วย QR Code

**โจทย์**: สร้าง App Clip สำหรับระบบ check-in งาน event โดย:
- รับ URL ในรูปแบบ `https://event.example.com/checkin?eventId=EVT001&ticketId=TKT123`
- แสดงข้อมูล event (ชื่องาน, วันที่, สถานที่)
- ปุ่ม Check-in ที่เรียก API เพื่อยืนยันการเข้างาน
- แสดงผล check-in สำเร็จและแสดง SKOverlay

**Solution:**

```swift
// EventCheckInAppClip - main entry point
import SwiftUI
import StoreKit

@main
struct EventCheckInAppClip: App {
    @StateObject private var checkInState = CheckInState()
    
    var body: some Scene {
        WindowGroup {
            CheckInRootView()
                .environmentObject(checkInState)
                .onContinueUserActivity(
                    NSUserActivityTypes.browsingWeb
                ) { activity in
                    if let url = activity.webpageURL {
                        checkInState.processURL(url)
                    }
                }
                // ใน DEBUG ใช้ environment variable
                .onAppear {
                    #if DEBUG
                    if let urlString = ProcessInfo.processInfo
                        .environment["_XCAppClipURL"],
                       let url = URL(string: urlString) {
                        checkInState.processURL(url)
                    }
                    #endif
                }
        }
    }
}

// MARK: - Model
struct EventInfo {
    let eventId: String
    let ticketId: String
    var eventName: String = ""
    var date: String = ""
    var venue: String = ""
    var attendeeName: String = ""
}

// MARK: - State
class CheckInState: ObservableObject {
    @Published var eventInfo: EventInfo?
    @Published var checkInStatus: CheckInStatus = .idle
    @Published var showAppOverlay = false
    
    enum CheckInStatus {
        case idle
        case loading
        case success(message: String)
        case failed(error: String)
    }
    
    func processURL(_ url: URL) {
        let items = URLComponents(url: url, resolvingAgainstBaseURL: true)?.queryItems
        guard let eventId = items?.first(where: { $0.name == "eventId" })?.value,
              let ticketId = items?.first(where: { $0.name == "ticketId" })?.value else {
            checkInStatus = .failed(error: "URL ไม่ถูกต้อง"); return
        }
        eventInfo = EventInfo(eventId: eventId, ticketId: ticketId)
        checkInStatus = .loading
        // จำลอง API call เพื่อดึงข้อมูล event
        DispatchQueue.main.asyncAfter(deadline: .now() + 1) { [weak self] in
            self?.eventInfo?.eventName = "Swift Developer Conference 2026"
            self?.eventInfo?.date = "1 ตุลาคม 2026 เวลา 09:00 น."
            self?.eventInfo?.venue = "ศูนย์ประชุมแห่งชาติสิริกิตต์"
            self?.eventInfo?.attendeeName = "คุณวิชัย พัฒนาโค้ด"
            self?.checkInStatus = .idle
        }
    }
    
    func performCheckIn() {
        guard let info = eventInfo else { return }
        checkInStatus = .loading
        // POST /api/checkin { eventId, ticketId }
        var request = URLRequest(url: URL(string: "https://event.example.com/api/checkin")!)
        request.httpMethod = "POST"
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        request.httpBody = try? JSONEncoder().encode(["eventId": info.eventId, "ticketId": info.ticketId])
        
        // จำลอง success response
        DispatchQueue.main.asyncAfter(deadline: .now() + 1.5) { [weak self] in
            self?.checkInStatus = .success(message: "เช็คอินสำเร็จ! ยินดีต้อนรับ")
            DispatchQueue.main.asyncAfter(deadline: .now() + 2) { self?.showAppOverlay = true }
        }
    }
}

// MARK: - Views
struct CheckInRootView: View {
    @EnvironmentObject var state: CheckInState
    
    var body: some View {
        NavigationView {
            switch state.checkInStatus {
            case .idle:
                if let eventInfo = state.eventInfo {
                    EventDetailView(eventInfo: eventInfo)
                } else {
                    ProgressView("กำลังโหลด...")
                }
                
            case .loading:
                ProgressView("กำลังดำเนินการ...")
                    .padding()
                
            case .success(let message):
                CheckInSuccessView(message: message)
                    .appStoreOverlay(isPresented: $state.showAppOverlay) {
                        SKOverlay.AppClipConfiguration(position: .bottom)
                    }
                
            case .failed(let error):
                ErrorView(message: error)
            }
        }
    }
}

struct EventDetailView: View {
    let eventInfo: EventInfo
    @EnvironmentObject var state: CheckInState
    
    var body: some View {
        VStack(spacing: 20) {
            Image(systemName: "ticket.fill").font(.system(size: 60)).foregroundColor(.blue)
            
            VStack(alignment: .leading, spacing: 12) {
                InfoRow(icon: "calendar", label: "งาน", value: eventInfo.eventName)
                InfoRow(icon: "clock", label: "วันที่", value: eventInfo.date)
                InfoRow(icon: "mappin", label: "สถานที่", value: eventInfo.venue)
                InfoRow(icon: "person", label: "ชื่อ", value: eventInfo.attendeeName)
            }
            .padding().background(Color(.systemGroupedBackground)).cornerRadius(12)
            
            Button(action: { state.performCheckIn() }) {
                Label("เช็คอิน", systemImage: "checkmark.circle.fill")
                    .font(.headline).frame(maxWidth: .infinity).padding()
                    .background(Color.blue).foregroundColor(.white).cornerRadius(12)
            }
        }
        .padding().navigationTitle("เช็คอินงาน Event")
    }
}

// Helper Views
struct InfoRow: View {
    let icon: String; let label: String; let value: String
    var body: some View {
        HStack {
            Image(systemName: icon).foregroundColor(.secondary).frame(width: 24)
            VStack(alignment: .leading) {
                Text(label).font(.caption).foregroundColor(.secondary)
                Text(value).font(.body)
            }
        }
    }
}

struct CheckInSuccessView: View {
    let message: String
    var body: some View {
        VStack(spacing: 20) {
            Image(systemName: "checkmark.seal.fill").font(.system(size: 80)).foregroundColor(.green)
            Text(message).font(.title2).fontWeight(.semibold).multilineTextAlignment(.center)
            Text("ดาวน์โหลดแอปเพื่อดู schedule และอัปเดตข่าวสาร")
                .foregroundColor(.secondary).multilineTextAlignment(.center)
        }.padding()
    }
}

struct ErrorView: View {
    let message: String
    var body: some View {
        VStack(spacing: 16) {
            Image(systemName: "xmark.octagon.fill").font(.system(size: 60)).foregroundColor(.red)
            Text("เกิดข้อผิดพลาด").font(.title2).fontWeight(.semibold)
            Text(message).foregroundColor(.secondary)
        }.padding()
    }
}
```

---

### แบบฝึกหัดที่ 2: Notification Service Extension พร้อม Custom Payload Handling

**โจทย์**: สร้าง Notification Service Extension ที่:
- ดาวน์โหลดรูปภาพจาก URL ใน payload
- ถอดรหัส encrypted message ใน payload
- เพิ่ม deep link action ให้กับ notification
- จัดการ edge cases (network error, timeout)

**Solution:**

```swift
// AdvancedNotificationService.swift
import UserNotifications
import CryptoKit

class AdvancedNotificationService: UNNotificationServiceExtension {
    
    var contentHandler: ((UNNotificationContent) -> Void)?
    var bestAttemptContent: UNMutableNotificationContent?
    private var downloadTask: URLSessionDataTask?
    
    override func didReceive(
        _ request: UNNotificationRequest,
        withContentHandler contentHandler: @escaping (UNNotificationContent) -> Void
    ) {
        self.contentHandler = contentHandler
        
        guard let mutableContent = request.content.mutableCopy()
            as? UNMutableNotificationContent else {
            contentHandler(request.content)
            return
        }
        
        self.bestAttemptContent = mutableContent
        
        // ดึง custom payload
        let userInfo = mutableContent.userInfo
        
        // ขั้นตอนที่ 1: ถอดรหัส encrypted message (ถ้ามี)
        if let encryptedMessage = userInfo["encrypted_body"] as? String {
            mutableContent.body = decryptMessage(encryptedMessage)
        }
        
        // ขั้นตอนที่ 2: เพิ่ม category สำหรับ action buttons
        if let actionType = userInfo["action_type"] as? String {
            mutableContent.categoryIdentifier = actionCategoryIdentifier(
                for: actionType
            )
        }
        
        // ขั้นตอนที่ 3: ดาวน์โหลดรูปภาพ (ถ้ามี URL)
        if let imageURLString = userInfo["image_url"] as? String,
           let imageURL = URL(string: imageURLString) {
            
            downloadAndAttach(
                imageURL: imageURL,
                to: mutableContent
            ) { finalContent in
                contentHandler(finalContent)
            }
        } else {
            // ไม่มีรูปภาพ ส่ง content ทันที
            contentHandler(mutableContent)
        }
    }
    
    // MARK: - Decrypt Message
    private func decryptMessage(_ encrypted: String) -> String {
        // จำลองการถอดรหัส (ในระบบจริงใช้ CryptoKit หรือ encryption library)
        // ตัวอย่างนี้ใช้ Base64 decode อย่างง่าย
        guard let data = Data(base64Encoded: encrypted),
              let decoded = String(data: data, encoding: .utf8) else {
            return "ไม่สามารถถอดรหัสข้อความได้"
        }
        return decoded
    }
    
    // MARK: - Action Category
    private func actionCategoryIdentifier(for type: String) -> String {
        switch type {
        case "order": return "ORDER_NOTIFICATION"
        case "promotion": return "PROMO_NOTIFICATION"
        case "message": return "MESSAGE_NOTIFICATION"
        default: return ""
        }
    }
    
    // MARK: - Download & Attach Image
    private func downloadAndAttach(
        imageURL: URL,
        to content: UNMutableNotificationContent,
        completion: @escaping (UNNotificationContent) -> Void
    ) {
        let config = URLSessionConfiguration.default
        config.timeoutIntervalForRequest = 25 // timeout 25 วินาที
        let session = URLSession(configuration: config)
        
        let task = session.downloadTask(with: imageURL) { 
            tempURL, response, error in
            
            // จัดการ error cases
            if let error = error {
                print("Download error: \(error.localizedDescription)")
                completion(content) // ส่ง content โดยไม่มีรูปภาพ
                return
            }
            
            guard let tempURL = tempURL else {
                completion(content)
                return
            }
            
            // กำหนด file extension จาก Content-Type header
            let ext: String
            if let httpResponse = response as? HTTPURLResponse,
               let contentType = httpResponse.allHeaderFields["Content-Type"] as? String {
                ext = self.extension(forContentType: contentType)
            } else {
                ext = "jpg"
            }
            
            // ย้ายไฟล์ไปยัง permanent location
            let permanentURL = tempURL
                .deletingPathExtension()
                .appendingPathExtension(ext)
            
            do {
                if FileManager.default.fileExists(atPath: permanentURL.path) {
                    try FileManager.default.removeItem(at: permanentURL)
                }
                try FileManager.default.moveItem(at: tempURL, to: permanentURL)
                
                // Validate image file size (ไม่เกิน 10MB สำหรับ notification)
                let fileSize = try permanentURL.resourceValues(
                    forKeys: [.fileSizeKey]
                ).fileSize ?? 0
                
                guard fileSize < 10 * 1024 * 1024 else {
                    print("รูปภาพใหญ่เกินไป: \(fileSize) bytes")
                    completion(content)
                    return
                }
                
                // สร้าง attachment
                let attachment = try UNNotificationAttachment(
                    identifier: UUID().uuidString,
                    url: permanentURL,
                    options: nil
                )
                
                content.attachments = [attachment]
                completion(content)
                
            } catch {
                print("Attachment error: \(error)")
                completion(content)
            }
        }
        
        self.downloadTask = task
        task.resume()
    }
    
    private func `extension`(forContentType contentType: String) -> String {
        if contentType.contains("jpeg") || contentType.contains("jpg") {
            return "jpg"
        } else if contentType.contains("png") {
            return "png"
        } else if contentType.contains("gif") {
            return "gif"
        }
        return "jpg"
    }
    
    // เรียกเมื่อใกล้หมดเวลา
    override func serviceExtensionTimeWillExpire() {
        downloadTask?.cancel()
        
        if let handler = contentHandler,
           let content = bestAttemptContent {
            handler(content)
        }
    }
}

// APNs test payload สำหรับทดสอบ Notification Service Extension:
// {
//   "aps": { "alert": {"title":"ออร์เดอร์ใหม่!","body":"placeholder"},
//            "mutable-content":1, "category":"ORDER_NOTIFICATION", "sound":"default" },
//   "encrypted_body": "6L+95ZiJ5LqG5aSx5pWX5paH5Lu25LiL5LiA5q615ZCN",
//   "image_url": "https://example.com/order-image.jpg",
//   "action_type": "order",
//   "order_id": "ORD-2026-001234"
// }
```

---

## สรุป

ในบทนี้เราได้เรียนรู้:

1. **App Clips**: การสร้าง lightweight experience สำหรับ use cases เฉพาะ เช่น payment, menu, parking
2. **App Clip Target Setup**: การตั้งค่า Xcode, แชร์โค้ด, รับ URL จาก NSUserActivity
3. **Invocation Types**: QR Code, NFC, Smart App Banner, Messages, Maps
4. **App Clip Limitations**: ไม่มี background refresh, จำกัด Keychain, ใช้ SKOverlay
5. **Testing**: Local Experience, `_XCAppClipURL` environment variable
6. **Share Extension**: รับและประมวลผล content จากแอปอื่น
7. **Action Extension**: ส่ง processed content กลับไปยังแอปต้นทาง
8. **Custom Keyboard Extension**: UIInputViewController, UICollectionView
9. **Notification Service Extension**: Rich push notification ด้วยรูปภาพ
10. **App Intents**: AppIntent protocol, @Parameter, Siri integration
11. **Background Tasks**: BGAppRefreshTask, BGProcessingTask

เทคโนโลยีเหล่านี้ช่วยให้แอปของเราทำงานได้อย่างหลากหลายและมอบประสบการณ์ที่ดีให้กับผู้ใช้ได้ทั้งในระหว่างใช้งานและนอกเวลาใช้งานแอปพลิเคชัน

---

*หมายเหตุ: โค้ดตัวอย่างในบทนี้ต้องการ iOS 16.0+ สำหรับ App Intents framework และ iOS 14.0+ สำหรับ App Clips*
