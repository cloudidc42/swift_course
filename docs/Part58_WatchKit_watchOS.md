# Part 58: WatchKit และ watchOS Development

## ภาพรวม watchOS Development

watchOS เป็นระบบปฏิบัติการของ Apple Watch ที่ช่วยให้นักพัฒนาสร้างแอปที่ทำงานบนข้อมือของผู้ใช้ได้ การพัฒนา watchOS มีความแตกต่างจาก iOS ที่ต้องคำนึงถึงหน้าจอขนาดเล็ก การประมวลผลที่จำกัด และแบตเตอรี่ที่มีข้อจำกัด

### ประวัติการพัฒนา

- **watchOS 1-2**: ต้องใช้ iPhone ในการประมวลผล
- **watchOS 3**: แอปทำงานบน Watch โดยตรง แต่ยังต้องใช้ iPhone
- **watchOS 6**: รองรับ Independent Watch Apps (แอปทำงานได้ไม่ต้องมี iPhone)
- **watchOS 7+**: SwiftUI เป็น first-class citizen บน watchOS

### ข้อจำกัดของ watchOS

1. **หน้าจอเล็ก** - Apple Watch มีขนาด 40mm, 41mm, 44mm, 45mm, 49mm
2. **การโต้ตอบสั้น** - ผู้ใช้ต้องการดูข้อมูลอย่างรวดเร็ว
3. **แบตเตอรี่** - ต้องใช้พลังงานอย่างมีประสิทธิภาพ
4. **ไม่มี Keyboard** - ต้องใช้ Scribble, Dictation, หรือ Digital Crown

---

## Watch App Targets

### การสร้าง Watch App Target

ใน Xcode:
1. File → New → Target
2. เลือก "watchOS" → "App"
3. กำหนดชื่อและ Bundle Identifier

### โครงสร้างโปรเจกต์

```
MyApp/
├── iOS App Target/
│   ├── ContentView.swift
│   └── ...
└── Watch App Target/
    ├── MyApp Watch App/
    │   ├── ContentView.swift
    │   ├── Assets.xcassets
    │   └── Info.plist
    └── MyApp Watch App Extension/ (watchOS 6 และก่อนหน้า)
```

### watchOS 7+ Architecture (Single App Bundle)

```
MyApp Watch App/
├── @main
│   └── MyWatchApp.swift
├── Views/
│   └── ContentView.swift
├── Models/
│   └── DataModel.swift
└── Info.plist
```

### App Entry Point

```swift
import SwiftUI

@main
struct MyWatchApp: App {
    // WKExtensionDelegate (ถ้าต้องการ)
    @WKApplicationDelegateAdaptor(AppDelegate.self) var appDelegate
    
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}

class AppDelegate: NSObject, WKApplicationDelegate {
    func applicationDidFinishLaunching() {
        // Setup code
        print("Watch app launched")
    }
    
    func applicationDidBecomeActive() {
        print("Watch app active")
    }
    
    func applicationWillResignActive() {
        print("Watch app going inactive")
    }
}
```

---

## WatchKit vs SwiftUI สำหรับ watchOS

### WatchKit (Legacy)

```swift
// WKInterfaceController - วิธีเก่า
import WatchKit

class MainInterfaceController: WKInterfaceController {
    @IBOutlet var label: WKInterfaceLabel!
    @IBOutlet var button: WKInterfaceButton!
    @IBOutlet var table: WKInterfaceTable!
    
    override func awake(withContext context: Any?) {
        super.awake(withContext: context)
        label.setText("Hello Watch!")
        button.setTitle("กดที่นี่")
    }
    
    @IBAction func buttonTapped() {
        label.setText("กดแล้ว!")
    }
    
    override func willActivate() {
        super.willActivate()
        // จะ activate
    }
    
    override func didDeactivate() {
        super.didDeactivate()
        // ถูก deactivate
    }
}
```

### SwiftUI (Modern - แนะนำ)

```swift
import SwiftUI

struct ContentView: View {
    @State private var message = "Hello Watch!"
    @State private var tapCount = 0
    
    var body: some View {
        VStack(spacing: 8) {
            Text(message)
                .font(.headline)
                .multilineTextAlignment(.center)
            
            Text("กด: \(tapCount)")
                .font(.caption)
                .foregroundColor(.secondary)
            
            Button("กดที่นี่") {
                tapCount += 1
                message = "กดแล้ว \(tapCount) ครั้ง!"
            }
            .buttonStyle(.borderedProminent)
        }
        .padding()
    }
}
```

---

## SwiftUI บน watchOS

### Differences จาก iOS SwiftUI

watchOS SwiftUI มีข้อจำกัดบางอย่าง:

1. ไม่รองรับ `UIKit` bridging
2. ไม่มี `NavigationView` บาง features
3. ไม่มี `List` sections บางแบบ
4. ไม่มี `MapView` (ใน watchOS เก่าบางรุ่น)

### Layout Basics

```swift
struct WatchLayoutExample: View {
    var body: some View {
        // ScrollView สำหรับ content ยาว
        ScrollView {
            VStack(spacing: 12) {
                // Header
                HStack {
                    Image(systemName: "heart.fill")
                        .foregroundColor(.red)
                    Text("สุขภาพ")
                        .font(.headline)
                }
                
                // Data Cards
                ForEach(0..<5) { index in
                    DataCard(value: "\(index * 100)", label: "ข้อมูล \(index)")
                }
            }
            .padding(.horizontal)
        }
    }
}

struct DataCard: View {
    let value: String
    let label: String
    
    var body: some View {
        HStack {
            Text(label)
                .font(.caption)
                .foregroundColor(.secondary)
            Spacer()
            Text(value)
                .font(.body)
                .fontWeight(.semibold)
        }
        .padding(.vertical, 8)
        .padding(.horizontal, 12)
        .background(Color.gray.opacity(0.2))
        .cornerRadius(8)
    }
}
```

### watchOS-Specific Views

```swift
struct WatchSpecificViews: View {
    @State private var progress: Double = 0.7
    
    var body: some View {
        ScrollView {
            VStack(spacing: 16) {
                // Gauge (watchOS 8+)
                Gauge(value: progress) {
                    Image(systemName: "heart.fill")
                        .foregroundColor(.red)
                } currentValueLabel: {
                    Text("\(Int(progress * 100))%")
                }
                .gaugeStyle(.accessoryCircular)
                .tint(.red)
                
                // ProgressView
                ProgressView(value: progress)
                    .progressViewStyle(.linear)
                    .tint(.green)
                
                // TimelineView (watchOS 8+)
                TimelineView(.periodic(from: Date(), by: 1)) { context in
                    Text(context.date, style: .time)
                        .font(.title2)
                        .fontWeight(.bold)
                }
                
                // Confirmation Dialog
                Button("ยืนยัน") {}
                    .buttonStyle(.borderedProminent)
                    .tint(.green)
            }
            .padding()
        }
    }
}
```

### Navigation บน watchOS

```swift
struct WatchNavigationExample: View {
    var body: some View {
        NavigationStack {
            List {
                NavigationLink("ก้าว") {
                    StepsDetailView()
                }
                NavigationLink("หัวใจ") {
                    HeartRateDetailView()
                }
                NavigationLink("การนอน") {
                    SleepDetailView()
                }
            }
            .navigationTitle("สุขภาพ")
        }
    }
}

struct StepsDetailView: View {
    var body: some View {
        VStack {
            Text("8,432")
                .font(.title)
                .fontWeight(.bold)
                .foregroundColor(.green)
            Text("ก้าววันนี้")
                .font(.caption)
                .foregroundColor(.secondary)
        }
        .navigationTitle("ก้าว")
    }
}

struct HeartRateDetailView: View {
    var body: some View {
        VStack {
            Text("72")
                .font(.title)
                .fontWeight(.bold)
                .foregroundColor(.red)
            Text("BPM")
                .font(.caption)
                .foregroundColor(.secondary)
        }
        .navigationTitle("หัวใจ")
    }
}

struct SleepDetailView: View {
    var body: some View {
        VStack {
            Text("7h 23m")
                .font(.title)
                .fontWeight(.bold)
                .foregroundColor(.blue)
            Text("การนอนเมื่อคืน")
                .font(.caption)
                .foregroundColor(.secondary)
        }
        .navigationTitle("การนอน")
    }
}
```

---

## WKExtensionDelegate

WKExtensionDelegate จัดการ lifecycle ของ Watch App extension

```swift
import WatchKit

class ExtensionDelegate: NSObject, WKExtensionDelegate {
    
    func applicationDidFinishLaunching() {
        // แอปเปิดตัวสำเร็จ
        setupBackgroundTasks()
    }
    
    func applicationDidBecomeActive() {
        // แอป active
        print("Watch app became active")
    }
    
    func applicationWillResignActive() {
        // แอปกำลังจะ inactive
        print("Watch app will resign active")
    }
    
    func applicationWillEnterForeground() {
        // แอปกลับมา foreground
    }
    
    func applicationDidEnterBackground() {
        // แอปไปอยู่ background
        scheduleBackgroundRefresh()
    }
    
    func handleBackgroundTasks(_ backgroundTasks: Set<WKBackgroundTask>) {
        for task in backgroundTasks {
            switch task {
            case let refreshTask as WKApplicationRefreshBackgroundTask:
                // ดึงข้อมูลใหม่
                performBackgroundRefresh {
                    refreshTask.setTaskCompletedWithSnapshot(false)
                }
                
            case let snapshotTask as WKSnapshotRefreshBackgroundTask:
                // อัปเดต snapshot
                snapshotTask.setTaskCompleted(
                    restoredDefaultState: true,
                    estimatedSnapshotExpiration: Date(timeIntervalSinceNow: 3600),
                    userInfo: nil
                )
                
            case let connectivityTask as WKWatchConnectivityRefreshBackgroundTask:
                // จัดการ Watch Connectivity
                connectivityTask.setTaskCompletedWithSnapshot(false)
                
            default:
                task.setTaskCompletedWithSnapshot(false)
            }
        }
    }
    
    private func setupBackgroundTasks() {
        // ตั้งค่า background tasks
    }
    
    private func scheduleBackgroundRefresh() {
        let nextRefreshDate = Date(timeIntervalSinceNow: 3600) // 1 ชั่วโมง
        WKExtension.shared().scheduleBackgroundRefresh(
            withPreferredDate: nextRefreshDate,
            userInfo: nil
        ) { error in
            if let error = error {
                print("Error scheduling background refresh: \(error)")
            }
        }
    }
    
    private func performBackgroundRefresh(completion: @escaping () -> Void) {
        // ดึงข้อมูลใหม่จาก server หรือ HealthKit
        Task {
            // await fetchData()
            completion()
        }
    }
}
```

---

## Watch Connectivity (WCSession)

WCSession ใช้สำหรับการส่งข้อมูลระหว่าง iPhone และ Apple Watch

### ประเภทการส่งข้อมูล

1. **sendMessage** - ส่ง dictionary แบบ real-time (ต้องทั้งคู่ active)
2. **transferUserInfo** - ส่ง dictionary แบบ background (FIFO queue)
3. **updateApplicationContext** - อัปเดต context (เก็บค่าล่าสุดเท่านั้น)
4. **transferFile** - ส่งไฟล์
5. **transferComplicationUserInfo** - ส่งข้อมูลสำหรับ Complication

### iOS Side (Phone App)

```swift
import WatchConnectivity
import SwiftUI

class PhoneConnectivityManager: NSObject, ObservableObject, WCSessionDelegate {
    static let shared = PhoneConnectivityManager()
    
    @Published var isWatchReachable = false
    @Published var lastReceivedMessage: [String: Any] = [:]
    
    private var session: WCSession?
    
    private override init() {
        super.init()
        setupSession()
    }
    
    private func setupSession() {
        guard WCSession.isSupported() else {
            print("WCSession ไม่รองรับบนอุปกรณ์นี้")
            return
        }
        
        session = WCSession.default
        session?.delegate = self
        session?.activate()
    }
    
    // MARK: - ส่งข้อมูลไปยัง Watch
    
    func sendMessageToWatch(_ message: [String: Any]) {
        guard let session = session, session.isReachable else {
            print("Watch ไม่พร้อมรับข้อมูล")
            return
        }
        
        session.sendMessage(message, replyHandler: { reply in
            print("Watch ตอบกลับ: \(reply)")
        }, errorHandler: { error in
            print("Error sending message: \(error)")
        })
    }
    
    func updateApplicationContext(_ context: [String: Any]) {
        guard let session = session else { return }
        
        do {
            try session.updateApplicationContext(context)
            print("อัปเดต application context สำเร็จ")
        } catch {
            print("Error updating context: \(error)")
        }
    }
    
    func transferUserInfo(_ info: [String: Any]) {
        guard let session = session else { return }
        session.transferUserInfo(info)
        print("ส่ง user info สำเร็จ")
    }
    
    func transferFileToWatch(at url: URL, metadata: [String: Any]? = nil) {
        guard let session = session else { return }
        session.transferFile(url, metadata: metadata)
        print("ส่งไฟล์สำเร็จ")
    }
    
    // MARK: - WCSessionDelegate
    
    func session(_ session: WCSession, activationDidCompleteWith activationState: WCSessionActivationState, error: Error?) {
        DispatchQueue.main.async {
            self.isWatchReachable = session.isReachable
        }
        
        if let error = error {
            print("Session activation error: \(error)")
        } else {
            print("Session activated: \(activationState.rawValue)")
        }
    }
    
    func sessionReachabilityDidChange(_ session: WCSession) {
        DispatchQueue.main.async {
            self.isWatchReachable = session.isReachable
        }
    }
    
    func session(_ session: WCSession, didReceiveMessage message: [String: Any]) {
        DispatchQueue.main.async {
            self.lastReceivedMessage = message
            print("รับข้อความจาก Watch: \(message)")
        }
    }
    
    func session(_ session: WCSession, didReceiveMessage message: [String: Any], replyHandler: @escaping ([String: Any]) -> Void) {
        DispatchQueue.main.async {
            self.lastReceivedMessage = message
            // ตอบกลับ
            replyHandler(["status": "received", "timestamp": Date().timeIntervalSince1970])
        }
    }
    
    func session(_ session: WCSession, didReceiveUserInfo userInfo: [String: Any]) {
        print("รับ user info: \(userInfo)")
    }
    
    func session(_ session: WCSession, didReceiveApplicationContext applicationContext: [String: Any]) {
        print("รับ application context: \(applicationContext)")
    }
    
    func session(_ session: WCSession, didFinish fileTransfer: WCSessionFileTransfer, error: Error?) {
        if let error = error {
            print("File transfer error: \(error)")
        } else {
            print("File transfer completed")
        }
    }
    
    // iOS Only
    func sessionDidBecomeInactive(_ session: WCSession) {
        print("Session became inactive")
    }
    
    func sessionDidDeactivate(_ session: WCSession) {
        session.activate()
    }
}
```

### watchOS Side (Watch App)

```swift
import WatchConnectivity
import SwiftUI

class WatchConnectivityManager: NSObject, ObservableObject, WCSessionDelegate {
    static let shared = WatchConnectivityManager()
    
    @Published var isPhoneReachable = false
    @Published var receivedData: [String: Any] = [:]
    @Published var stepCount = 0
    
    private override init() {
        super.init()
        setupSession()
    }
    
    private func setupSession() {
        guard WCSession.isSupported() else { return }
        
        WCSession.default.delegate = self
        WCSession.default.activate()
    }
    
    // MARK: - ส่งข้อมูลไปยัง iPhone
    
    func sendMessageToPhone(_ message: [String: Any], replyHandler: (([String: Any]) -> Void)? = nil) {
        guard WCSession.default.isReachable else {
            print("iPhone ไม่พร้อมรับข้อมูล")
            return
        }
        
        WCSession.default.sendMessage(message, replyHandler: replyHandler) { error in
            print("Error sending to phone: \(error)")
        }
    }
    
    func requestDataFromPhone() {
        sendMessageToPhone(["request": "healthData"]) { reply in
            DispatchQueue.main.async {
                self.receivedData = reply
                if let steps = reply["steps"] as? Int {
                    self.stepCount = steps
                }
            }
        }
    }
    
    func updateWatchContext(steps: Int, heartRate: Double) {
        let context: [String: Any] = [
            "steps": steps,
            "heartRate": heartRate,
            "timestamp": Date().timeIntervalSince1970
        ]
        
        do {
            try WCSession.default.updateApplicationContext(context)
        } catch {
            print("Error updating context: \(error)")
        }
    }
    
    // MARK: - WCSessionDelegate
    
    func session(_ session: WCSession, activationDidCompleteWith activationState: WCSessionActivationState, error: Error?) {
        DispatchQueue.main.async {
            self.isPhoneReachable = session.isReachable
        }
    }
    
    func sessionReachabilityDidChange(_ session: WCSession) {
        DispatchQueue.main.async {
            self.isPhoneReachable = WCSession.default.isReachable
        }
    }
    
    func session(_ session: WCSession, didReceiveMessage message: [String: Any]) {
        DispatchQueue.main.async {
            self.receivedData = message
            self.processReceivedData(message)
        }
    }
    
    func session(_ session: WCSession, didReceiveMessage message: [String: Any], replyHandler: @escaping ([String: Any]) -> Void) {
        processReceivedData(message)
        replyHandler(["watchReceived": true, "timestamp": Date().timeIntervalSince1970])
    }
    
    func session(_ session: WCSession, didReceiveApplicationContext applicationContext: [String: Any]) {
        DispatchQueue.main.async {
            self.processReceivedData(applicationContext)
        }
    }
    
    func session(_ session: WCSession, didReceiveUserInfo userInfo: [String: Any]) {
        DispatchQueue.main.async {
            self.processReceivedData(userInfo)
        }
    }
    
    private func processReceivedData(_ data: [String: Any]) {
        if let steps = data["steps"] as? Int {
            stepCount = steps
        }
    }
}
```

### SwiftUI Integration

```swift
// iPhone App View
struct PhoneMainView: View {
    @StateObject private var connectivity = PhoneConnectivityManager.shared
    @State private var stepData = 8432
    @State private var heartRateData = 72.0
    
    var body: some View {
        NavigationStack {
            VStack(spacing: 20) {
                // Watch Status
                HStack {
                    Circle()
                        .fill(connectivity.isWatchReachable ? Color.green : Color.red)
                        .frame(width: 12, height: 12)
                    Text(connectivity.isWatchReachable ? "Watch เชื่อมต่อแล้ว" : "Watch ไม่ได้เชื่อมต่อ")
                        .font(.caption)
                }
                
                // Stats
                VStack(spacing: 12) {
                    StatRow(label: "ก้าว", value: "\(stepData)")
                    StatRow(label: "หัวใจ", value: "\(Int(heartRateData)) BPM")
                }
                
                // Send to Watch
                Button("ส่งข้อมูลไปยัง Watch") {
                    connectivity.updateApplicationContext([
                        "steps": stepData,
                        "heartRate": heartRateData,
                        "timestamp": Date().timeIntervalSince1970
                    ])
                }
                .buttonStyle(.borderedProminent)
                
                Button("ส่ง Message ไปยัง Watch") {
                    connectivity.sendMessageToWatch([
                        "action": "update",
                        "steps": stepData,
                        "heartRate": heartRateData
                    ])
                }
                .buttonStyle(.bordered)
            }
            .padding()
            .navigationTitle("iPhone App")
        }
    }
}

struct StatRow: View {
    let label: String
    let value: String
    
    var body: some View {
        HStack {
            Text(label)
                .foregroundColor(.secondary)
            Spacer()
            Text(value)
                .fontWeight(.semibold)
        }
        .padding()
        .background(Color(.systemBackground))
        .cornerRadius(8)
    }
}

// Watch App View
struct WatchMainView: View {
    @StateObject private var connectivity = WatchConnectivityManager.shared
    
    var body: some View {
        VStack(spacing: 10) {
            // Connection Status
            HStack(spacing: 4) {
                Circle()
                    .fill(connectivity.isPhoneReachable ? Color.green : Color.gray)
                    .frame(width: 8, height: 8)
                Text(connectivity.isPhoneReachable ? "เชื่อมต่อ" : "ไม่ได้เชื่อมต่อ")
                    .font(.caption2)
                    .foregroundColor(.secondary)
            }
            
            // Steps
            VStack(spacing: 2) {
                Text("\(connectivity.stepCount)")
                    .font(.title2)
                    .fontWeight(.bold)
                    .foregroundColor(.green)
                Text("ก้าว")
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
            
            // Request Data Button
            Button("ดึงข้อมูล") {
                connectivity.requestDataFromPhone()
            }
            .buttonStyle(.borderedProminent)
            .font(.caption)
        }
        .padding()
    }
}
```

---

## Complications

Complications คือ widgets ที่แสดงบนหน้าปัดนาฬิกา Apple Watch

### ประเภท Complications

```swift
// CLKComplicationFamily types:
// .modularSmall      - เล็ก
// .modularLarge      - ใหญ่
// .utilitarianSmall  - Utility เล็ก
// .utilitarianLarge  - Utility ใหญ่
// .circularSmall     - วงกลมเล็ก
// .extraLarge        - ใหญ่มาก (Apple Watch 2+)
// .graphicCorner     - มุม (Series 4+)
// .graphicBezel      - ขอบ (Series 4+)
// .graphicCircular   - วงกลม (Series 4+)
// .graphicRectangular - สี่เหลี่ยม (Series 4+)
// .graphicExtraLarge - ใหญ่มาก Graphic (Series 7+)
```

### Complication Data Source

```swift
import ClockKit

class ComplicationController: NSObject, CLKComplicationDataSource {
    
    // MARK: - Complication Configuration
    
    func getComplicationDescriptors(handler: @escaping ([CLKComplicationDescriptor]) -> Void) {
        let descriptors = [
            CLKComplicationDescriptor(
                identifier: "StepCount",
                displayName: "จำนวนก้าว",
                supportedFamilies: [
                    .modularSmall,
                    .circularSmall,
                    .graphicCircular,
                    .graphicCorner,
                    .utilitarianSmall
                ]
            )
        ]
        handler(descriptors)
    }
    
    // MARK: - Timeline Configuration
    
    func getTimelineEndDate(for complication: CLKComplication, withHandler handler: @escaping (Date?) -> Void) {
        handler(nil) // ไม่มีวันหมดอายุ
    }
    
    func getPrivacyBehavior(for complication: CLKComplication, withHandler handler: @escaping (CLKComplicationPrivacyBehavior) -> Void) {
        handler(.showOnLockScreen)
    }
    
    // MARK: - Timeline Population
    
    func getCurrentTimelineEntry(for complication: CLKComplication, withHandler handler: @escaping (CLKComplicationTimelineEntry?) -> Void) {
        let template = createTemplate(for: complication.family, steps: 8432)
        
        if let template = template {
            let entry = CLKComplicationTimelineEntry(date: Date(), complicationTemplate: template)
            handler(entry)
        } else {
            handler(nil)
        }
    }
    
    func getTimelineEntries(for complication: CLKComplication, before date: Date, limit: Int, withHandler handler: @escaping ([CLKComplicationTimelineEntry]?) -> Void) {
        handler(nil)
    }
    
    func getTimelineEntries(for complication: CLKComplication, after date: Date, limit: Int, withHandler handler: @escaping ([CLKComplicationTimelineEntry]?) -> Void) {
        handler(nil)
    }
    
    // MARK: - Placeholder Templates
    
    func getLocalizableSampleTemplate(for complication: CLKComplication, withHandler handler: @escaping (CLKComplicationTemplate?) -> Void) {
        let template = createTemplate(for: complication.family, steps: 1234)
        handler(template)
    }
    
    // MARK: - Template Creation
    
    private func createTemplate(for family: CLKComplicationFamily, steps: Int) -> CLKComplicationTemplate? {
        let stepsText = "\(steps)"
        let labelText = "ก้าว"
        
        switch family {
        case .modularSmall:
            let template = CLKComplicationTemplateModularSmallStackText()
            template.line1TextProvider = CLKSimpleTextProvider(text: stepsText)
            template.line2TextProvider = CLKSimpleTextProvider(text: labelText)
            return template
            
        case .circularSmall:
            let template = CLKComplicationTemplateCircularSmallStackText()
            template.line1TextProvider = CLKSimpleTextProvider(text: stepsText)
            template.line2TextProvider = CLKSimpleTextProvider(text: labelText)
            return template
            
        case .graphicCircular:
            let template = CLKComplicationTemplateGraphicCircularStackText()
            template.line1TextProvider = CLKSimpleTextProvider(text: stepsText)
            template.line2TextProvider = CLKSimpleTextProvider(text: labelText)
            return template
            
        case .graphicCorner:
            let template = CLKComplicationTemplateGraphicCornerStackText()
            template.innerTextProvider = CLKSimpleTextProvider(text: stepsText)
            template.outerTextProvider = CLKSimpleTextProvider(text: labelText)
            return template
            
        case .utilitarianSmall:
            let template = CLKComplicationTemplateUtilitarianSmallFlat()
            template.textProvider = CLKSimpleTextProvider(text: "\(stepsText) \(labelText)")
            return template
            
        default:
            return nil
        }
    }
    
    // MARK: - อัปเดต Complication จาก App
    
    static func reloadComplications() {
        let server = CLKComplicationServer.sharedInstance()
        server.activeComplications?.forEach { complication in
            server.reloadTimeline(for: complication)
        }
    }
}
```

### WidgetKit สำหรับ watchOS (watchOS 9+)

```swift
import WidgetKit
import SwiftUI

struct StepCountEntry: TimelineEntry {
    let date: Date
    let steps: Int
    let goal: Int
}

struct StepCountProvider: TimelineProvider {
    func placeholder(in context: Context) -> StepCountEntry {
        StepCountEntry(date: Date(), steps: 0, goal: 10000)
    }
    
    func getSnapshot(in context: Context, completion: @escaping (StepCountEntry) -> Void) {
        let entry = StepCountEntry(date: Date(), steps: 5000, goal: 10000)
        completion(entry)
    }
    
    func getTimeline(in context: Context, completion: @escaping (Timeline<StepCountEntry>) -> Void) {
        // ดึงข้อมูลจริงและสร้าง timeline
        Task {
            let steps = await fetchSteps()
            let entry = StepCountEntry(date: Date(), steps: steps, goal: 10000)
            let nextUpdate = Calendar.current.date(byAdding: .minute, value: 30, to: Date())!
            let timeline = Timeline(entries: [entry], policy: .after(nextUpdate))
            completion(timeline)
        }
    }
    
    private func fetchSteps() async -> Int {
        // ดึงข้อมูลจาก HealthKit
        return 8432 // placeholder
    }
}

struct StepCountWidgetView: View {
    var entry: StepCountProvider.Entry
    
    @Environment(\.widgetFamily) var family
    
    var body: some View {
        switch family {
        case .accessoryCircular:
            CircularView(steps: entry.steps, goal: entry.goal)
        case .accessoryCorner:
            CornerView(steps: entry.steps)
        case .accessoryRectangular:
            RectangularView(steps: entry.steps, goal: entry.goal)
        default:
            CircularView(steps: entry.steps, goal: entry.goal)
        }
    }
}

struct CircularView: View {
    let steps: Int
    let goal: Int
    
    var progress: Double {
        Double(steps) / Double(goal)
    }
    
    var body: some View {
        ZStack {
            Circle()
                .stroke(Color.green.opacity(0.3), lineWidth: 4)
            Circle()
                .trim(from: 0, to: min(progress, 1.0))
                .stroke(Color.green, style: StrokeStyle(lineWidth: 4, lineCap: .round))
                .rotationEffect(.degrees(-90))
            
            VStack(spacing: 0) {
                Text("\(steps / 1000)k")
                    .font(.system(size: 14, weight: .bold))
                Text("ก้าว")
                    .font(.system(size: 8))
                    .foregroundColor(.secondary)
            }
        }
    }
}

struct CornerView: View {
    let steps: Int
    
    var body: some View {
        Label("\(steps)", systemImage: "figure.walk")
            .foregroundColor(.green)
    }
}

struct RectangularView: View {
    let steps: Int
    let goal: Int
    
    var body: some View {
        VStack(alignment: .leading, spacing: 4) {
            Label("ก้าววันนี้", systemImage: "figure.walk")
                .font(.caption)
                .foregroundColor(.secondary)
            
            Text("\(steps)")
                .font(.title3)
                .fontWeight(.bold)
                .foregroundColor(.green)
            
            ProgressView(value: Double(steps), total: Double(goal))
                .tint(.green)
        }
    }
}

@main
struct StepCountWidget: Widget {
    let kind = "StepCount"
    
    var body: some WidgetConfiguration {
        StaticConfiguration(kind: kind, provider: StepCountProvider()) { entry in
            StepCountWidgetView(entry: entry)
        }
        .configurationDisplayName("ก้าว")
        .description("แสดงจำนวนก้าวของวันนี้")
        .supportedFamilies([.accessoryCircular, .accessoryCorner, .accessoryRectangular])
    }
}
```

---

## Background Tasks บน watchOS

watchOS มีการจำกัด background execution อย่างเข้มงวด

### ประเภท Background Tasks

```swift
import WatchKit

// ใน WKExtensionDelegate
func handleBackgroundTasks(_ backgroundTasks: Set<WKBackgroundTask>) {
    for task in backgroundTasks {
        switch task {
        // 1. Application Refresh - อัปเดตข้อมูล
        case let refreshTask as WKApplicationRefreshBackgroundTask:
            handleRefreshTask(refreshTask)
            
        // 2. Snapshot Refresh - อัปเดต UI snapshot
        case let snapshotTask as WKSnapshotRefreshBackgroundTask:
            handleSnapshotTask(snapshotTask)
            
        // 3. Watch Connectivity - รับข้อมูลจาก iPhone
        case let connectivityTask as WKWatchConnectivityRefreshBackgroundTask:
            handleConnectivityTask(connectivityTask)
            
        // 4. URLSession - ดาวน์โหลดข้อมูล
        case let urlSessionTask as WKURLSessionRefreshBackgroundTask:
            handleURLSessionTask(urlSessionTask)
            
        // 5. Relevant Shortcut Refresh
        case let relevantShortcutTask as WKRelevantShortcutRefreshBackgroundTask:
            relevantShortcutTask.setTaskCompletedWithSnapshot(false)
            
        // 6. Intent Did Run Shortcut
        case let intentDidRunTask as WKIntentDidRunRefreshBackgroundTask:
            intentDidRunTask.setTaskCompletedWithSnapshot(false)
            
        default:
            task.setTaskCompletedWithSnapshot(false)
        }
    }
}

func handleRefreshTask(_ task: WKApplicationRefreshBackgroundTask) {
    // ดึงข้อมูลใหม่
    fetchLatestData { completed in
        if completed {
            // กำหนดเวลา refresh ครั้งต่อไป
            self.scheduleNextRefresh()
        }
        task.setTaskCompletedWithSnapshot(false)
    }
}

func handleSnapshotTask(_ task: WKSnapshotRefreshBackgroundTask) {
    // อัปเดต UI สำหรับ snapshot
    task.setTaskCompleted(
        restoredDefaultState: true,
        estimatedSnapshotExpiration: .distantFuture,
        userInfo: nil
    )
}

func handleConnectivityTask(_ task: WKWatchConnectivityRefreshBackgroundTask) {
    // จัดการข้อมูลที่ได้รับจาก iPhone
    task.setTaskCompletedWithSnapshot(false)
}

func handleURLSessionTask(_ task: WKURLSessionRefreshBackgroundTask) {
    let backgroundSession = URLSession(
        configuration: .background(withIdentifier: task.sessionIdentifier),
        delegate: nil,
        delegateQueue: nil
    )
    
    // ดาวน์โหลดข้อมูล
    backgroundSession.getAllTasks { tasks in
        task.setTaskCompletedWithSnapshot(false)
    }
}

func scheduleNextRefresh() {
    let nextRefresh = Date(timeIntervalSinceNow: 3600) // 1 ชั่วโมง
    WKExtension.shared().scheduleBackgroundRefresh(
        withPreferredDate: nextRefresh,
        userInfo: nil
    ) { error in
        if let error = error {
            print("Schedule error: \(error)")
        }
    }
}

func fetchLatestData(completion: @escaping (Bool) -> Void) {
    // ดึงข้อมูลจาก HealthKit หรือ server
    completion(true)
}
```

---

## Workout Sessions บน watchOS

```swift
#if os(watchOS)
import HealthKit
import WatchKit
import SwiftUI

@MainActor
class WatchWorkoutManager: NSObject, ObservableObject {
    private let healthStore = HKHealthStore()
    
    var session: HKWorkoutSession?
    var builder: HKLiveWorkoutBuilder?
    
    @Published var isActive = false
    @Published var isPaused = false
    @Published var heartRate: Double = 0
    @Published var activeCalories: Double = 0
    @Published var distance: Double = 0
    @Published var elapsedTime: TimeInterval = 0
    
    private var timer: Timer?
    private var workoutStartDate: Date?
    
    override init() {
        super.init()
    }
    
    func requestAuthorization() async throws {
        let typesToShare: Set<HKSampleType> = [HKObjectType.workoutType()]
        let typesToRead: Set<HKObjectType> = [
            HKQuantityType.quantityType(forIdentifier: .heartRate)!,
            HKQuantityType.quantityType(forIdentifier: .activeEnergyBurned)!,
            HKQuantityType.quantityType(forIdentifier: .distanceWalkingRunning)!
        ]
        
        try await healthStore.requestAuthorization(toShare: typesToShare, read: typesToRead)
    }
    
    func startWorkout(activityType: HKWorkoutActivityType, locationType: HKWorkoutSessionLocationType = .outdoor) async throws {
        let configuration = HKWorkoutConfiguration()
        configuration.activityType = activityType
        configuration.locationType = locationType
        
        session = try HKWorkoutSession(healthStore: healthStore, configuration: configuration)
        builder = session?.associatedWorkoutBuilder()
        
        session?.delegate = self
        builder?.delegate = self
        
        builder?.dataSource = HKLiveWorkoutDataSource(
            healthStore: healthStore,
            workoutConfiguration: configuration
        )
        
        let startDate = Date()
        workoutStartDate = startDate
        
        session?.startActivity(with: startDate)
        try await builder?.beginCollection(at: startDate)
        
        isActive = true
        startTimer()
        
        // Haptic feedback
        WKInterfaceDevice.current().play(.start)
    }
    
    func pauseWorkout() {
        session?.pause()
        isPaused = true
        timer?.invalidate()
        
        WKInterfaceDevice.current().play(.stop)
    }
    
    func resumeWorkout() {
        session?.resume()
        isPaused = false
        startTimer()
        
        WKInterfaceDevice.current().play(.start)
    }
    
    func stopWorkout() async {
        session?.end()
        
        do {
            try await builder?.endCollection(at: Date())
            let workout = try await builder?.finishWorkout()
            print("Workout saved: \(String(describing: workout))")
        } catch {
            print("Error finishing workout: \(error)")
        }
        
        isActive = false
        isPaused = false
        timer?.invalidate()
        timer = nil
        
        WKInterfaceDevice.current().play(.success)
    }
    
    private func startTimer() {
        timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { [weak self] _ in
            guard let self = self, let startDate = self.workoutStartDate else { return }
            self.elapsedTime = Date().timeIntervalSince(startDate)
        }
    }
    
    var formattedElapsedTime: String {
        let hours = Int(elapsedTime) / 3600
        let minutes = (Int(elapsedTime) % 3600) / 60
        let seconds = Int(elapsedTime) % 60
        
        if hours > 0 {
            return String(format: "%02d:%02d:%02d", hours, minutes, seconds)
        } else {
            return String(format: "%02d:%02d", minutes, seconds)
        }
    }
}

extension WatchWorkoutManager: HKWorkoutSessionDelegate {
    nonisolated func workoutSession(_ workoutSession: HKWorkoutSession,
                        didChangeTo toState: HKWorkoutSessionState,
                        from fromState: HKWorkoutSessionState,
                        date: Date) {
        print("Workout state changed: \(fromState.rawValue) -> \(toState.rawValue)")
    }
    
    nonisolated func workoutSession(_ workoutSession: HKWorkoutSession, didFailWithError error: Error) {
        print("Workout session failed: \(error)")
    }
}

extension WatchWorkoutManager: HKLiveWorkoutBuilderDelegate {
    nonisolated func workoutBuilder(_ workoutBuilder: HKLiveWorkoutBuilder, didCollectDataOf collectedTypes: Set<HKSampleType>) {
        for type in collectedTypes {
            guard let quantityType = type as? HKQuantityType else { continue }
            
            let statistics = workoutBuilder.statistics(for: quantityType)
            
            Task { @MainActor in
                switch quantityType {
                case HKQuantityType.quantityType(forIdentifier: .heartRate)!:
                    self.heartRate = statistics?.mostRecentQuantity()?.doubleValue(for: HKUnit(from: "count/min")) ?? 0
                    
                case HKQuantityType.quantityType(forIdentifier: .activeEnergyBurned)!:
                    self.activeCalories = statistics?.sumQuantity()?.doubleValue(for: .kilocalorie()) ?? 0
                    
                case HKQuantityType.quantityType(forIdentifier: .distanceWalkingRunning)!:
                    self.distance = statistics?.sumQuantity()?.doubleValue(for: .meter()) ?? 0
                    
                default:
                    break
                }
            }
        }
    }
    
    nonisolated func workoutBuilderDidCollectEvent(_ workoutBuilder: HKLiveWorkoutBuilder) {}
}

// MARK: - Watch Workout View
struct WorkoutSessionView: View {
    @StateObject private var manager = WatchWorkoutManager()
    @State private var selectedActivity: HKWorkoutActivityType = .running
    
    var body: some View {
        if manager.isActive {
            ActiveWorkoutView(manager: manager)
        } else {
            WorkoutSelectionView(manager: manager, selectedActivity: $selectedActivity)
        }
    }
}

struct WorkoutSelectionView: View {
    @ObservedObject var manager: WatchWorkoutManager
    @Binding var selectedActivity: HKWorkoutActivityType
    
    let activities: [(String, HKWorkoutActivityType, String)] = [
        ("วิ่ง", .running, "figure.run"),
        ("เดิน", .walking, "figure.walk"),
        ("ปั่นจักรยาน", .cycling, "figure.outdoor.cycle"),
        ("ว่ายน้ำ", .swimming, "figure.pool.swim"),
        ("โยคะ", .yoga, "figure.mind.and.body")
    ]
    
    var body: some View {
        ScrollView {
            VStack(spacing: 8) {
                Text("เลือกการออกกำลังกาย")
                    .font(.headline)
                    .padding(.top)
                
                ForEach(activities, id: \.1.rawValue) { name, type, icon in
                    Button(action: {
                        Task {
                            try? await manager.startWorkout(activityType: type)
                        }
                    }) {
                        HStack {
                            Image(systemName: icon)
                                .foregroundColor(.green)
                                .frame(width: 24)
                            Text(name)
                            Spacer()
                            Image(systemName: "chevron.right")
                                .font(.caption)
                                .foregroundColor(.secondary)
                        }
                        .padding(.vertical, 8)
                        .padding(.horizontal, 12)
                    }
                    .buttonStyle(.plain)
                    .background(Color.gray.opacity(0.2))
                    .cornerRadius(8)
                }
            }
            .padding(.horizontal)
        }
    }
}

struct ActiveWorkoutView: View {
    @ObservedObject var manager: WatchWorkoutManager
    
    var body: some View {
        TabView {
            // Main Stats Tab
            VStack(spacing: 8) {
                Text(manager.formattedElapsedTime)
                    .font(.title2)
                    .fontWeight(.bold)
                    .monospacedDigit()
                
                Divider()
                
                HStack(spacing: 16) {
                    VStack {
                        Image(systemName: "heart.fill")
                            .foregroundColor(.red)
                        Text("\(Int(manager.heartRate))")
                            .font(.headline)
                        Text("BPM")
                            .font(.caption2)
                            .foregroundColor(.secondary)
                    }
                    
                    VStack {
                        Image(systemName: "flame.fill")
                            .foregroundColor(.orange)
                        Text("\(Int(manager.activeCalories))")
                            .font(.headline)
                        Text("kcal")
                            .font(.caption2)
                            .foregroundColor(.secondary)
                    }
                }
                
                if manager.distance > 0 {
                    VStack {
                        Text(String(format: "%.2f", manager.distance / 1000))
                            .font(.headline)
                        Text("กม.")
                            .font(.caption2)
                            .foregroundColor(.secondary)
                    }
                }
            }
            .padding()
            
            // Controls Tab
            VStack(spacing: 12) {
                Button(manager.isPaused ? "ดำเนินต่อ" : "หยุดชั่วคราว") {
                    if manager.isPaused {
                        manager.resumeWorkout()
                    } else {
                        manager.pauseWorkout()
                    }
                }
                .buttonStyle(.borderedProminent)
                .tint(manager.isPaused ? .green : .yellow)
                
                Button("จบการออกกำลังกาย") {
                    Task {
                        await manager.stopWorkout()
                    }
                }
                .buttonStyle(.bordered)
                .tint(.red)
            }
            .padding()
        }
        .tabViewStyle(.page)
    }
}
#endif
```

---

## Digital Crown

Digital Crown คือปุ่มหมุนที่ Apple Watch ใช้สำหรับการ scroll และ input

```swift
import SwiftUI

struct DigitalCrownExample: View {
    @State private var scrollAmount = 0.0
    @State private var selectedHour = 8
    @State private var selectedMinute = 30
    
    var body: some View {
        VStack(spacing: 12) {
            // ตัวอย่างที่ 1: Scrollable List
            Text("เลื่อนด้วย Digital Crown")
                .font(.caption)
                .foregroundColor(.secondary)
            
            Text("\(Int(scrollAmount))")
                .font(.title)
                .fontWeight(.bold)
                .foregroundColor(.blue)
                .focusable()
                .digitalCrownRotation($scrollAmount, from: 0, through: 100, by: 1, sensitivity: .medium)
            
            Divider()
            
            // ตัวอย่างที่ 2: Time Picker
            Text("เลือกเวลา: \(selectedHour):\(String(format: "%02d", selectedMinute))")
                .font(.headline)
        }
        .padding()
    }
}

// Picker ที่ใช้ Digital Crown
struct CrownPickerView: View {
    @State private var selectedValue: Double = 50
    let range: ClosedRange<Double> = 0...100
    
    var body: some View {
        VStack {
            Gauge(value: selectedValue, in: range) {
                Text("ค่า")
            } currentValueLabel: {
                Text("\(Int(selectedValue))")
            }
            .gaugeStyle(.accessoryCircular)
            .tint(.blue)
            
            Text("\(Int(selectedValue))")
                .font(.largeTitle)
                .fontWeight(.bold)
                .focusable()
                .digitalCrownRotation(
                    $selectedValue,
                    from: range.lowerBound,
                    through: range.upperBound,
                    by: 1,
                    sensitivity: .medium,
                    isContinuous: false,
                    isHapticFeedbackEnabled: true
                )
        }
    }
}

// List navigation ด้วย Digital Crown
struct CrownListView: View {
    let items = Array(1...20)
    @State private var selectedItem = 1.0
    
    var body: some View {
        VStack {
            Text("Item #\(Int(selectedItem))")
                .font(.title2)
                .fontWeight(.bold)
            
            Text("หมุน Digital Crown เพื่อเปลี่ยน")
                .font(.caption)
                .foregroundColor(.secondary)
        }
        .focusable()
        .digitalCrownRotation(
            $selectedItem,
            from: 1,
            through: Double(items.count),
            by: 1,
            sensitivity: .low,
            isHapticFeedbackEnabled: true
        )
    }
}
```

---

## Haptic Feedback

Apple Watch รองรับ haptic feedback ที่หลากหลาย

```swift
import WatchKit
import SwiftUI

struct HapticFeedbackExample: View {
    var body: some View {
        ScrollView {
            VStack(spacing: 12) {
                Text("Haptic Feedback")
                    .font(.headline)
                
                Group {
                    HapticButton("Notification", haptic: .notification)
                    HapticButton("Direction Up", haptic: .directionUp)
                    HapticButton("Direction Down", haptic: .directionDown)
                    HapticButton("Success", haptic: .success)
                    HapticButton("Failure", haptic: .failure)
                    HapticButton("Retry", haptic: .retry)
                    HapticButton("Start", haptic: .start)
                    HapticButton("Stop", haptic: .stop)
                    HapticButton("Click", haptic: .click)
                }
            }
            .padding()
        }
    }
}

struct HapticButton: View {
    let label: String
    let haptic: WKHapticType
    
    init(_ label: String, haptic: WKHapticType) {
        self.label = label
        self.haptic = haptic
    }
    
    var body: some View {
        Button(label) {
            WKInterfaceDevice.current().play(haptic)
        }
        .buttonStyle(.borderedProminent)
        .frame(maxWidth: .infinity)
    }
}

// Haptic patterns สำหรับ Workout
class WorkoutHaptics {
    static func playWorkoutStart() {
        WKInterfaceDevice.current().play(.start)
    }
    
    static func playWorkoutPause() {
        WKInterfaceDevice.current().play(.stop)
    }
    
    static func playWorkoutComplete() {
        // ลำดับ haptic สำหรับ completion
        WKInterfaceDevice.current().play(.success)
        
        DispatchQueue.main.asyncAfter(deadline: .now() + 0.3) {
            WKInterfaceDevice.current().play(.notification)
        }
    }
    
    static func playGoalReached() {
        WKInterfaceDevice.current().play(.directionUp)
    }
    
    static func playHeartRateAlert() {
        // สั่น 3 ครั้ง
        for i in 0..<3 {
            DispatchQueue.main.asyncAfter(deadline: .now() + Double(i) * 0.5) {
                WKInterfaceDevice.current().play(.notification)
            }
        }
    }
}
```

---

## Notifications บน watchOS

```swift
import WatchKit
import UserNotifications
import SwiftUI

// ขอ notification permission บน watch
func requestNotificationPermission() {
    UNUserNotificationCenter.current().requestAuthorization(
        options: [.alert, .sound, .badge]
    ) { granted, error in
        if granted {
            print("Notification permission granted")
        }
    }
}

// สร้าง Local Notification
func scheduleLocalNotification(title: String, body: String, delay: TimeInterval) {
    let content = UNMutableNotificationContent()
    content.title = title
    content.body = body
    content.sound = .default
    
    let trigger = UNTimeIntervalNotificationTrigger(timeInterval: delay, repeats: false)
    
    let request = UNNotificationRequest(
        identifier: UUID().uuidString,
        content: content,
        trigger: trigger
    )
    
    UNUserNotificationCenter.current().add(request) { error in
        if let error = error {
            print("Notification error: \(error)")
        }
    }
}

// Notification Controller สำหรับ watchOS
class NotificationController: WKUserNotificationHostingController<NotificationView> {
    var notificationTitle: String = ""
    var notificationMessage: String = ""
    
    override var body: NotificationView {
        return NotificationView(
            title: notificationTitle,
            message: notificationMessage
        )
    }
    
    override func didReceive(_ notification: UNNotification) {
        notificationTitle = notification.request.content.title
        notificationMessage = notification.request.content.body
    }
}

struct NotificationView: View {
    let title: String
    let message: String
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            Text(title)
                .font(.headline)
                .foregroundColor(.white)
            
            Text(message)
                .font(.caption)
                .foregroundColor(.secondary)
        }
        .padding()
        .background(Color.blue.opacity(0.8))
        .cornerRadius(12)
    }
}

// Workout Reminder Notification
func scheduleWorkoutReminder() {
    let content = UNMutableNotificationContent()
    content.title = "ถึงเวลาออกกำลังกาย!"
    content.body = "คุณยังไม่ได้ออกกำลังกายวันนี้ มาออกกำลังกาย 30 นาทีกันเถอะ"
    content.sound = .default
    content.categoryIdentifier = "WORKOUT_REMINDER"
    
    // กำหนด action buttons
    let startAction = UNNotificationAction(
        identifier: "START_WORKOUT",
        title: "เริ่มเลย",
        options: [.foreground]
    )
    
    let laterAction = UNNotificationAction(
        identifier: "REMIND_LATER",
        title: "เตือนอีกครั้ง",
        options: []
    )
    
    let category = UNNotificationCategory(
        identifier: "WORKOUT_REMINDER",
        actions: [startAction, laterAction],
        intentIdentifiers: [],
        options: []
    )
    
    UNUserNotificationCenter.current().setNotificationCategories([category])
    
    // เตือนตอน 18:00
    var dateComponents = DateComponents()
    dateComponents.hour = 18
    dateComponents.minute = 0
    
    let trigger = UNCalendarNotificationTrigger(dateMatching: dateComponents, repeats: true)
    
    let request = UNNotificationRequest(
        identifier: "workout-reminder",
        content: content,
        trigger: trigger
    )
    
    UNUserNotificationCenter.current().add(request)
}
```

---

## Independent watchOS Apps

ตั้งแต่ watchOS 6 แอป Watch สามารถทำงานได้โดยไม่ต้องมี iPhone

### การตั้งค่า Independent App

```swift
// ใน Info.plist ของ Watch target
// WKWatchOnly = YES (ต้องการ watch เท่านั้น)
// หรือ
// WKRunsIndependentlyOfCompanionApp = YES (ทำงานได้โดยไม่มี companion app)
```

### Networking จาก Watch โดยตรง

```swift
import Foundation
import SwiftUI

class WatchNetworkManager: ObservableObject {
    @Published var data: [String: Any] = [:]
    @Published var isLoading = false
    @Published var error: String?
    
    func fetchData(from url: URL) async {
        isLoading = true
        error = nil
        
        do {
            let (data, response) = try await URLSession.shared.data(from: url)
            
            guard let httpResponse = response as? HTTPURLResponse,
                  httpResponse.statusCode == 200 else {
                error = "Server error"
                return
            }
            
            if let json = try JSONSerialization.jsonObject(with: data) as? [String: Any] {
                await MainActor.run {
                    self.data = json
                }
            }
        } catch {
            await MainActor.run {
                self.error = error.localizedDescription
            }
        }
        
        await MainActor.run {
            isLoading = false
        }
    }
}

// Watch App ที่ดึงข้อมูลเองจาก Network
struct IndependentWatchView: View {
    @StateObject private var networkManager = WatchNetworkManager()
    
    var body: some View {
        Group {
            if networkManager.isLoading {
                ProgressView("กำลังโหลด...")
            } else if let error = networkManager.error {
                VStack {
                    Text("เกิดข้อผิดพลาด")
                        .font(.headline)
                    Text(error)
                        .font(.caption)
                        .foregroundColor(.secondary)
                    Button("ลองอีกครั้ง") {
                        Task {
                            await networkManager.fetchData(from: URL(string: "https://api.example.com/health")!)
                        }
                    }
                }
            } else {
                Text("โหลดข้อมูลสำเร็จ")
                    .font(.headline)
            }
        }
        .task {
            await networkManager.fetchData(from: URL(string: "https://api.example.com/health")!)
        }
    }
}
```

---

## watchOS App Design Guidelines

### หลักการออกแบบ

1. **ย่อให้กระชับ** - แสดงข้อมูลที่จำเป็นเท่านั้น
2. **ใช้สีอย่างมีจุดประสงค์** - สีช่วยบ่งบอกสถานะและความสำคัญ
3. **Typography** - ใช้ขนาดตัวอักษรที่อ่านง่ายบนหน้าจอเล็ก
4. **Interaction** - ออกแบบสำหรับการโต้ตอบสั้น (3-5 วินาที)

```swift
// Design System สำหรับ watchOS
struct WatchDesignSystem {
    // Typography
    static let titleFont = Font.system(.headline, weight: .semibold)
    static let bodyFont = Font.system(.body)
    static let captionFont = Font.system(.caption)
    static let microFont = Font.system(.caption2)
    
    // Colors
    static let primary = Color.white
    static let secondary = Color.gray
    static let accent = Color.blue
    static let success = Color.green
    static let warning = Color.yellow
    static let error = Color.red
    
    // Spacing
    static let spacing = CGFloat(8)
    static let largeSpacing = CGFloat(16)
    
    // Corner Radius
    static let cornerRadius = CGFloat(8)
}

// Reusable Watch Components
struct WatchCard<Content: View>: View {
    let content: Content
    
    init(@ViewBuilder content: () -> Content) {
        self.content = content()
    }
    
    var body: some View {
        content
            .padding(WatchDesignSystem.spacing)
            .background(Color.gray.opacity(0.2))
            .cornerRadius(WatchDesignSystem.cornerRadius)
    }
}

struct WatchStatView: View {
    let value: String
    let label: String
    let color: Color
    let icon: String
    
    var body: some View {
        VStack(spacing: 4) {
            Image(systemName: icon)
                .foregroundColor(color)
                .font(.system(size: 20))
            
            Text(value)
                .font(WatchDesignSystem.titleFont)
                .foregroundColor(WatchDesignSystem.primary)
            
            Text(label)
                .font(WatchDesignSystem.microFont)
                .foregroundColor(WatchDesignSystem.secondary)
        }
    }
}

// Glanceable UI Example
struct GlanceableHealthView: View {
    let steps: Int
    let heartRate: Int
    let calories: Int
    
    var body: some View {
        VStack(spacing: 4) {
            // วันที่และเวลา
            Text(Date(), style: .time)
                .font(.caption2)
                .foregroundColor(.secondary)
            
            Divider()
            
            // ข้อมูลสำคัญ
            HStack(spacing: 12) {
                WatchStatView(value: "\(steps/1000)k", label: "ก้าว", color: .green, icon: "figure.walk")
                Divider().frame(height: 40)
                WatchStatView(value: "\(heartRate)", label: "BPM", color: .red, icon: "heart.fill")
                Divider().frame(height: 40)
                WatchStatView(value: "\(calories)", label: "kcal", color: .orange, icon: "flame.fill")
            }
        }
        .padding(.horizontal, 8)
    }
}
```

---

## การทดสอบบน Apple Watch Simulator

### การตั้งค่า Simulator

1. เปิด Xcode → Simulator
2. เลือก Device: Apple Watch Series 9 45mm (หรือขนาดอื่น)
3. เลือก Pair กับ iPhone Simulator

### Tips การทดสอบ

```swift
// 1. ใช้ Preview สำหรับ UI ที่รวดเร็ว
#Preview("Watch 45mm") {
    ContentView()
        .previewDevice("Apple Watch Series 9 (45mm)")
}

#Preview("Watch 41mm") {
    ContentView()
        .previewDevice("Apple Watch Series 9 (41mm)")
}

// 2. ทดสอบ Digital Crown ใน Simulator
// - ใช้ scroll wheel ของ mouse
// - หรือ Option + Scroll

// 3. ทดสอบ Watch Face Complications
// - ใช้ Edit Face ใน Simulator
// - เลือก Complication ที่ต้องการทดสอบ

// 4. Mock Data สำหรับ HealthKit ใน Simulator
// HealthKit ทำงานได้ใน Simulator แต่ต้องเพิ่มข้อมูลด้วยตนเอง
// ผ่าน Health App ใน iPhone Simulator ที่ pair กัน
```

---

## แบบฝึกหัดปฏิบัติ

### แบบฝึกหัดที่ 1: Watch Health Dashboard

```swift
// โจทย์: สร้าง Watch app ที่แสดงข้อมูลสุขภาพแบบ glanceable
// Solution:

import SwiftUI
import HealthKit

@main
struct HealthWatchApp: App {
    var body: some Scene {
        WindowGroup {
            HealthWatchRootView()
        }
    }
}

struct HealthWatchRootView: View {
    @StateObject private var vm = HealthWatchViewModel()
    
    var body: some View {
        Group {
            if vm.isAuthorized {
                HealthWatchDashboard(vm: vm)
            } else {
                AuthRequestView(vm: vm)
            }
        }
        .task {
            await vm.requestAuth()
        }
    }
}

@MainActor
class HealthWatchViewModel: ObservableObject {
    private let store = HKHealthStore()
    
    @Published var steps = 0
    @Published var heartRate = 0.0
    @Published var calories = 0.0
    @Published var isAuthorized = false
    @Published var isLoading = false
    
    func requestAuth() async {
        guard HKHealthStore.isHealthDataAvailable() else { return }
        
        let types: Set<HKObjectType> = [
            HKQuantityType.quantityType(forIdentifier: .stepCount)!,
            HKQuantityType.quantityType(forIdentifier: .heartRate)!,
            HKQuantityType.quantityType(forIdentifier: .activeEnergyBurned)!
        ]
        
        do {
            try await store.requestAuthorization(toShare: [], read: types)
            isAuthorized = true
            await loadData()
        } catch {
            print("Auth error: \(error)")
        }
    }
    
    func loadData() async {
        isLoading = true
        await withTaskGroup(of: Void.self) { g in
            g.addTask { await self.loadSteps() }
            g.addTask { await self.loadHeartRate() }
            g.addTask { await self.loadCalories() }
        }
        isLoading = false
    }
    
    private func loadSteps() async {
        guard let type = HKQuantityType.quantityType(forIdentifier: .stepCount) else { return }
        let start = Calendar.current.startOfDay(for: Date())
        let pred = HKQuery.predicateForSamples(withStart: start, end: Date())
        
        steps = await withCheckedContinuation { c in
            let q = HKStatisticsQuery(quantityType: type, quantitySamplePredicate: pred, options: .cumulativeSum) { _, r, _ in
                c.resume(returning: Int(r?.sumQuantity()?.doubleValue(for: .count()) ?? 0))
            }
            store.execute(q)
        }
    }
    
    private func loadHeartRate() async {
        guard let type = HKQuantityType.quantityType(forIdentifier: .heartRate) else { return }
        let sort = NSSortDescriptor(key: HKSampleSortIdentifierStartDate, ascending: false)
        
        heartRate = await withCheckedContinuation { c in
            let q = HKSampleQuery(sampleType: type, predicate: nil, limit: 1, sortDescriptors: [sort]) { _, s, _ in
                c.resume(returning: (s?.first as? HKQuantitySample)?.quantity.doubleValue(for: HKUnit(from: "count/min")) ?? 0)
            }
            store.execute(q)
        }
    }
    
    private func loadCalories() async {
        guard let type = HKQuantityType.quantityType(forIdentifier: .activeEnergyBurned) else { return }
        let start = Calendar.current.startOfDay(for: Date())
        let pred = HKQuery.predicateForSamples(withStart: start, end: Date())
        
        calories = await withCheckedContinuation { c in
            let q = HKStatisticsQuery(quantityType: type, quantitySamplePredicate: pred, options: .cumulativeSum) { _, r, _ in
                c.resume(returning: r?.sumQuantity()?.doubleValue(for: .kilocalorie()) ?? 0)
            }
            store.execute(q)
        }
    }
}

struct AuthRequestView: View {
    @ObservedObject var vm: HealthWatchViewModel
    
    var body: some View {
        VStack(spacing: 8) {
            Image(systemName: "heart.fill")
                .font(.title2)
                .foregroundColor(.red)
            Text("ต้องการข้อมูล\nสุขภาพ")
                .font(.caption)
                .multilineTextAlignment(.center)
            Button("อนุญาต") {
                Task { await vm.requestAuth() }
            }
            .buttonStyle(.borderedProminent)
            .font(.caption)
        }
        .padding()
    }
}

struct HealthWatchDashboard: View {
    @ObservedObject var vm: HealthWatchViewModel
    
    var body: some View {
        ScrollView {
            VStack(spacing: 10) {
                // Time
                Text(Date(), style: .time)
                    .font(.caption2)
                    .foregroundColor(.secondary)
                
                // Step Ring
                ZStack {
                    Circle()
                        .stroke(Color.green.opacity(0.3), lineWidth: 6)
                        .frame(width: 80, height: 80)
                    Circle()
                        .trim(from: 0, to: min(Double(vm.steps) / 10000, 1.0))
                        .stroke(Color.green, style: StrokeStyle(lineWidth: 6, lineCap: .round))
                        .frame(width: 80, height: 80)
                        .rotationEffect(.degrees(-90))
                    VStack(spacing: 0) {
                        Text("\(vm.steps)")
                            .font(.system(size: 16, weight: .bold))
                        Text("ก้าว")
                            .font(.system(size: 8))
                            .foregroundColor(.secondary)
                    }
                }
                
                // Stats Row
                HStack(spacing: 16) {
                    VStack {
                        Text("\(Int(vm.heartRate))")
                            .font(.headline).foregroundColor(.red)
                        Text("BPM").font(.caption2).foregroundColor(.secondary)
                    }
                    VStack {
                        Text("\(Int(vm.calories))")
                            .font(.headline).foregroundColor(.orange)
                        Text("kcal").font(.caption2).foregroundColor(.secondary)
                    }
                }
                
                // Refresh
                Button("รีเฟรช") {
                    Task { await vm.loadData() }
                }
                .buttonStyle(.bordered)
                .font(.caption2)
            }
            .padding(.horizontal, 8)
        }
    }
}
```

### แบบฝึกหัดที่ 2: Watch Connectivity Demo

```swift
// โจทย์: สร้าง app ที่ส่งข้อมูลระหว่าง iPhone และ Apple Watch
// Solution: ดูโค้ด WatchConnectivityManager และ PhoneConnectivityManager ด้านบน

// ส่วนเพิ่มเติม - Watch View สำหรับ connectivity demo
struct WatchConnectivityDemoView: View {
    @StateObject private var wc = WatchConnectivityManager.shared
    @State private var messageToSend = ""
    @State private var receivedMessages: [String] = []
    
    var body: some View {
        ScrollView {
            VStack(spacing: 8) {
                // Status
                HStack {
                    Circle()
                        .fill(wc.isPhoneReachable ? Color.green : Color.red)
                        .frame(width: 8, height: 8)
                    Text(wc.isPhoneReachable ? "iPhone เชื่อมต่อ" : "ไม่ได้เชื่อมต่อ")
                        .font(.caption2)
                }
                
                // Current Steps
                if wc.stepCount > 0 {
                    Text("\(wc.stepCount) ก้าว")
                        .font(.title3)
                        .fontWeight(.bold)
                        .foregroundColor(.green)
                }
                
                // Actions
                Button("ขอข้อมูลจาก iPhone") {
                    wc.requestDataFromPhone()
                }
                .buttonStyle(.borderedProminent)
                .font(.caption)
                
                Button("อัปเดต Context") {
                    wc.updateWatchContext(steps: 5000, heartRate: 72)
                }
                .buttonStyle(.bordered)
                .font(.caption)
            }
            .padding(8)
        }
    }
}
```

---

## การสร้าง watchOS Companion App

### App Architecture

```
FitnessApp/
├── Shared/
│   ├── Models/
│   │   ├── HealthData.swift      (shared between iOS & watchOS)
│   │   └── WorkoutData.swift
│   └── Services/
│       └── HealthKitService.swift
├── iOS App/
│   ├── ContentView.swift
│   ├── PhoneConnectivityManager.swift
│   └── Views/
│       └── ...
└── Watch App/
    ├── ContentView.swift
    ├── WatchConnectivityManager.swift
    └── Views/
        ├── DashboardView.swift
        ├── WorkoutView.swift
        └── ...
```

### Shared Models

```swift
// ไฟล์นี้ใช้ร่วมกันระหว่าง iOS และ watchOS
import Foundation

struct HealthSummary: Codable {
    var steps: Int
    var heartRate: Double
    var calories: Double
    var distance: Double
    var date: Date
    
    init(steps: Int = 0, heartRate: Double = 0, calories: Double = 0, distance: Double = 0, date: Date = Date()) {
        self.steps = steps
        self.heartRate = heartRate
        self.calories = calories
        self.distance = distance
        self.date = date
    }
    
    var dictionary: [String: Any] {
        return [
            "steps": steps,
            "heartRate": heartRate,
            "calories": calories,
            "distance": distance,
            "timestamp": date.timeIntervalSince1970
        ]
    }
    
    init?(from dictionary: [String: Any]) {
        guard let steps = dictionary["steps"] as? Int,
              let heartRate = dictionary["heartRate"] as? Double,
              let calories = dictionary["calories"] as? Double,
              let distance = dictionary["distance"] as? Double,
              let timestamp = dictionary["timestamp"] as? Double else {
            return nil
        }
        
        self.steps = steps
        self.heartRate = heartRate
        self.calories = calories
        self.distance = distance
        self.date = Date(timeIntervalSince1970: timestamp)
    }
}

struct WorkoutSession: Codable, Identifiable {
    let id: UUID
    var activityType: String
    var startDate: Date
    var endDate: Date?
    var calories: Double
    var distance: Double?
    var heartRateAverage: Double?
    
    var duration: TimeInterval? {
        guard let end = endDate else { return nil }
        return end.timeIntervalSince(startDate)
    }
    
    var isActive: Bool {
        return endDate == nil
    }
}
```

### Complete Watch App

```swift
import SwiftUI
import WatchConnectivity
import HealthKit

@main
struct FitnessWatchApp: App {
    @WKApplicationDelegateAdaptor(WatchAppDelegate.self) var appDelegate
    
    var body: some Scene {
        WindowGroup {
            WatchRootView()
        }
    }
}

class WatchAppDelegate: NSObject, WKApplicationDelegate {
    func applicationDidFinishLaunching() {
        // Initialize services
        _ = WatchConnectivityManager.shared
    }
}

struct WatchRootView: View {
    @StateObject private var healthVM = WatchHealthViewModel()
    
    var body: some View {
        TabView {
            WatchDashboardView(vm: healthVM)
            WatchWorkoutListView()
            WatchSettingsView()
        }
        .task {
            await healthVM.initialize()
        }
    }
}

@MainActor
class WatchHealthViewModel: ObservableObject {
    @Published var summary = HealthSummary()
    @Published var isLoading = false
    
    private let healthStore = HKHealthStore()
    private let connectivity = WatchConnectivityManager.shared
    
    func initialize() async {
        guard HKHealthStore.isHealthDataAvailable() else { return }
        
        do {
            let types: Set<HKObjectType> = [
                HKQuantityType.quantityType(forIdentifier: .stepCount)!,
                HKQuantityType.quantityType(forIdentifier: .heartRate)!,
                HKQuantityType.quantityType(forIdentifier: .activeEnergyBurned)!,
                HKQuantityType.quantityType(forIdentifier: .distanceWalkingRunning)!
            ]
            try await healthStore.requestAuthorization(toShare: [], read: types)
            await refresh()
        } catch {
            print("Error: \(error)")
        }
    }
    
    func refresh() async {
        isLoading = true
        
        async let steps = fetchSteps()
        async let hr = fetchHeartRate()
        async let cal = fetchCalories()
        async let dist = fetchDistance()
        
        summary = HealthSummary(
            steps: await steps,
            heartRate: await hr,
            calories: await cal,
            distance: await dist
        )
        
        // ส่งข้อมูลไปยัง iPhone
        try? WCSession.default.updateApplicationContext(summary.dictionary)
        
        isLoading = false
    }
    
    private func fetchSteps() async -> Int {
        guard let type = HKQuantityType.quantityType(forIdentifier: .stepCount) else { return 0 }
        let start = Calendar.current.startOfDay(for: Date())
        let pred = HKQuery.predicateForSamples(withStart: start, end: Date())
        
        return await withCheckedContinuation { c in
            let q = HKStatisticsQuery(quantityType: type, quantitySamplePredicate: pred, options: .cumulativeSum) { _, r, _ in
                c.resume(returning: Int(r?.sumQuantity()?.doubleValue(for: .count()) ?? 0))
            }
            healthStore.execute(q)
        }
    }
    
    private func fetchHeartRate() async -> Double {
        guard let type = HKQuantityType.quantityType(forIdentifier: .heartRate) else { return 0 }
        let sort = NSSortDescriptor(key: HKSampleSortIdentifierStartDate, ascending: false)
        
        return await withCheckedContinuation { c in
            let q = HKSampleQuery(sampleType: type, predicate: nil, limit: 1, sortDescriptors: [sort]) { _, s, _ in
                c.resume(returning: (s?.first as? HKQuantitySample)?.quantity.doubleValue(for: HKUnit(from: "count/min")) ?? 0)
            }
            healthStore.execute(q)
        }
    }
    
    private func fetchCalories() async -> Double {
        guard let type = HKQuantityType.quantityType(forIdentifier: .activeEnergyBurned) else { return 0 }
        let start = Calendar.current.startOfDay(for: Date())
        let pred = HKQuery.predicateForSamples(withStart: start, end: Date())
        
        return await withCheckedContinuation { c in
            let q = HKStatisticsQuery(quantityType: type, quantitySamplePredicate: pred, options: .cumulativeSum) { _, r, _ in
                c.resume(returning: r?.sumQuantity()?.doubleValue(for: .kilocalorie()) ?? 0)
            }
            healthStore.execute(q)
        }
    }
    
    private func fetchDistance() async -> Double {
        guard let type = HKQuantityType.quantityType(forIdentifier: .distanceWalkingRunning) else { return 0 }
        let start = Calendar.current.startOfDay(for: Date())
        let pred = HKQuery.predicateForSamples(withStart: start, end: Date())
        
        return await withCheckedContinuation { c in
            let q = HKStatisticsQuery(quantityType: type, quantitySamplePredicate: pred, options: .cumulativeSum) { _, r, _ in
                c.resume(returning: r?.sumQuantity()?.doubleValue(for: .meter()) ?? 0)
            }
            healthStore.execute(q)
        }
    }
}

struct WatchDashboardView: View {
    @ObservedObject var vm: WatchHealthViewModel
    
    var stepProgress: Double {
        min(Double(vm.summary.steps) / 10000, 1.0)
    }
    
    var body: some View {
        ScrollView {
            VStack(spacing: 10) {
                // Time
                Text(Date(), style: .time)
                    .font(.caption2)
                    .foregroundColor(.secondary)
                
                // Step Progress Ring
                ZStack {
                    Circle()
                        .stroke(Color.green.opacity(0.25), lineWidth: 8)
                    Circle()
                        .trim(from: 0, to: stepProgress)
                        .stroke(
                            AngularGradient(colors: [.green.opacity(0.7), .green], center: .center),
                            style: StrokeStyle(lineWidth: 8, lineCap: .round)
                        )
                        .rotationEffect(.degrees(-90))
                    
                    VStack(spacing: 1) {
                        Text("\(vm.summary.steps)")
                            .font(.system(size: 18, weight: .bold, design: .rounded))
                            .foregroundColor(.green)
                        Text("ก้าว")
                            .font(.system(size: 9))
                            .foregroundColor(.secondary)
                    }
                }
                .frame(width: 90, height: 90)
                
                // Stats
                HStack(spacing: 0) {
                    Spacer()
                    VStack(spacing: 2) {
                        Image(systemName: "heart.fill").foregroundColor(.red).font(.caption)
                        Text("\(Int(vm.summary.heartRate))").font(.callout).fontWeight(.semibold)
                        Text("BPM").font(.system(size: 8)).foregroundColor(.secondary)
                    }
                    Spacer()
                    VStack(spacing: 2) {
                        Image(systemName: "flame.fill").foregroundColor(.orange).font(.caption)
                        Text("\(Int(vm.summary.calories))").font(.callout).fontWeight(.semibold)
                        Text("kcal").font(.system(size: 8)).foregroundColor(.secondary)
                    }
                    Spacer()
                    VStack(spacing: 2) {
                        Image(systemName: "map.fill").foregroundColor(.blue).font(.caption)
                        Text(String(format: "%.1f", vm.summary.distance / 1000)).font(.callout).fontWeight(.semibold)
                        Text("กม.").font(.system(size: 8)).foregroundColor(.secondary)
                    }
                    Spacer()
                }
            }
            .padding(.horizontal, 6)
        }
        .navigationTitle("สุขภาพ")
        .toolbar {
            ToolbarItem(placement: .confirmationAction) {
                Button(action: { Task { await vm.refresh() } }) {
                    Image(systemName: "arrow.clockwise")
                        .font(.caption)
                }
            }
        }
    }
}

struct WatchWorkoutListView: View {
    let workoutTypes: [(String, HKWorkoutActivityType, String, Color)] = [
        ("วิ่ง", .running, "figure.run", .green),
        ("เดิน", .walking, "figure.walk", .blue),
        ("ปั่นจักรยาน", .cycling, "figure.outdoor.cycle", .orange),
        ("ว่ายน้ำ", .swimming, "figure.pool.swim", .cyan),
        ("โยคะ", .yoga, "figure.mind.and.body", .purple)
    ]
    
    var body: some View {
        NavigationStack {
            List(workoutTypes, id: \.1.rawValue) { name, type, icon, color in
                NavigationLink {
                    // WorkoutActiveView(activityType: type)
                    Text("เริ่ม \(name)")
                } label: {
                    HStack {
                        Image(systemName: icon)
                            .foregroundColor(color)
                            .frame(width: 24)
                        Text(name)
                    }
                }
            }
            .navigationTitle("ออกกำลังกาย")
        }
    }
}

struct WatchSettingsView: View {
    @State private var stepGoal = 10000.0
    @State private var notifications = true
    
    var body: some View {
        NavigationStack {
            ScrollView {
                VStack(alignment: .leading, spacing: 12) {
                    Text("เป้าหมายก้าว")
                        .font(.caption)
                        .foregroundColor(.secondary)
                    
                    Text("\(Int(stepGoal))")
                        .font(.title3)
                        .fontWeight(.bold)
                    
                    Slider(value: $stepGoal, in: 5000...20000, step: 1000)
                        .tint(.green)
                    
                    Toggle("การแจ้งเตือน", isOn: $notifications)
                        .font(.caption)
                }
                .padding()
            }
            .navigationTitle("ตั้งค่า")
        }
    }
}
```

---

## สรุป

watchOS development เป็นทักษะที่สำคัญสำหรับนักพัฒนา iOS ที่ต้องการสร้างประสบการณ์บน Apple Watch

### Key Takeaways

1. **SwiftUI เป็น preferred framework** สำหรับ watchOS ใหม่ๆ
2. **WCSession** ช่วยส่งข้อมูลระหว่าง iPhone และ Watch
3. **Complications** แสดงข้อมูลบนหน้าปัดนาฬิกา
4. **Background Tasks** มีข้อจำกัด ต้องใช้อย่างมีประสิทธิภาพ
5. **Digital Crown** เป็น unique input สำหรับ Watch
6. **Haptic Feedback** ช่วยสื่อสารกับผู้ใช้
7. **Independent Apps** (watchOS 6+) ทำงานได้โดยไม่ต้องมี iPhone

### Best Practices

```swift
// 1. ออกแบบสำหรับ glanceable interaction (3-5 วินาที)
// 2. ใช้พลังงานอย่างมีประสิทธิภาพ
// 3. ทดสอบบน hardware จริงเพื่อ performance
// 4. ใช้ async/await สำหรับ modern code
// 5. Handle ทุก error case

// ตัวอย่าง efficient data loading
func loadWatchData() async {
    // โหลดแบบ parallel เพื่อความรวดเร็ว
    async let steps = fetchSteps()
    async let heartRate = fetchHeartRate()
    
    // รอทั้งคู่พร้อมกัน
    let (s, hr) = await (steps, heartRate)
    
    await MainActor.run {
        self.steps = s
        self.heartRate = hr
    }
}

// Efficient memory usage
struct WatchViewModel: ObservableObject {
    // เก็บเฉพาะข้อมูลที่จำเป็น
    @Published var currentStats = HealthSummary()
    
    // ไม่เก็บ history ยาวๆ บน Watch
    // ส่งไปยัง iPhone แทน
}
```

### Watch App Checklist

- [ ] รองรับทุกขนาด watch (38/40/41/42/44/45/49mm)
- [ ] UI แสดงผลได้ใน landscape และ portrait
- [ ] ทดสอบ battery usage
- [ ] ใช้ Background Tasks อย่างมีประสิทธิภาพ
- [ ] Complications อัปเดตอย่างถูกต้อง
- [ ] Watch Connectivity ทำงานได้ทั้ง foreground และ background
- [ ] Haptic feedback ใช้อย่างเหมาะสม
- [ ] Text ขนาดใหญ่พอที่จะอ่านบนหน้าจอเล็ก

---

*จบ Part 58: WatchKit และ watchOS Development*
