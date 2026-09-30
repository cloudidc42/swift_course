# ตอนที่ 83: Cross-Platform Swift Development

## การพัฒนา Swift แบบหลายแพลตฟอร์ม

---

## 1. ภาพรวม Swift Multiplatform

Swift ได้รับการออกแบบมาให้ทำงานได้บนหลายแพลตฟอร์ม ตั้งแต่อุปกรณ์ Apple ไปจนถึง Linux และ Windows การเข้าใจความสามารถและข้อจำกัดของแต่ละแพลตฟอร์มเป็นสิ่งสำคัญสำหรับนักพัฒนาที่ต้องการสร้างแอปพลิเคชันที่ทำงานได้บนหลายระบบ

### 1.1 แพลตฟอร์มที่ Swift รองรับ

```swift
// แผนภูมิแพลตฟอร์มที่ Swift รองรับ:
//
// Apple Platforms:
// ├── iOS (iPhone, iPad)
// ├── macOS (Mac, Mac mini, MacBook)
// ├── watchOS (Apple Watch)
// ├── tvOS (Apple TV)
// └── visionOS (Apple Vision Pro)
//
// Non-Apple Platforms:
// ├── Linux (Ubuntu, Fedora, Amazon Linux, etc.)
// └── Windows (ผ่าน swift.org Windows toolchain)
```

### 1.2 Foundation Framework บนแต่ละแพลตฟอร์ม

Foundation เป็น framework หลักที่ Swift ใช้ แต่มีความแตกต่างกันบนแต่ละแพลตฟอร์ม:

```swift
// Foundation บน Apple Platforms - ครบสมบูรณ์
import Foundation

let url = URL(string: "https://api.example.com")!
let date = Date()
let calendar = Calendar.current

// Foundation บน Linux - swift-corelibs-foundation
// ส่วนใหญ่เหมือนกัน แต่บางส่วนยังขาด:
// - NSAppleScript
// - NSUserNotification (บางส่วน)
// - RunLoop บางฟังก์ชัน
```

### 1.3 Conditional Compilation - #if os()

การเขียนโค้ดที่ต่างกันบนแต่ละแพลตฟอร์มทำได้ด้วย `#if os()`:

```swift
import Foundation

class PlatformInfo {
    
    static var platformName: String {
        #if os(iOS)
        return "iOS"
        #elseif os(macOS)
        return "macOS"
        #elseif os(watchOS)
        return "watchOS"
        #elseif os(tvOS)
        return "tvOS"
        #elseif os(visionOS)
        return "visionOS"
        #elseif os(Linux)
        return "Linux"
        #elseif os(Windows)
        return "Windows"
        #else
        return "Unknown"
        #endif
    }
    
    static var isApplePlatform: Bool {
        #if os(iOS) || os(macOS) || os(watchOS) || os(tvOS) || os(visionOS)
        return true
        #else
        return false
        #endif
    }
    
    static var supportsUIKit: Bool {
        #if canImport(UIKit)
        return true
        #else
        return false
        #endif
    }
}

// ใช้งาน
print("Running on: \(PlatformInfo.platformName)")
print("Is Apple Platform: \(PlatformInfo.isApplePlatform)")
```

### 1.4 #if canImport() - ตรวจสอบ Framework

`canImport` ใช้ตรวจสอบว่า framework มีอยู่หรือไม่ก่อน import:

```swift
#if canImport(UIKit)
import UIKit
typealias PlatformColor = UIColor
typealias PlatformView = UIView
typealias PlatformImage = UIImage

extension UIColor {
    static var platformBackground: UIColor {
        return .systemBackground
    }
}

#elseif canImport(AppKit)
import AppKit
typealias PlatformColor = NSColor
typealias PlatformView = NSView
typealias PlatformImage = NSImage

extension NSColor {
    static var platformBackground: NSColor {
        return .windowBackgroundColor
    }
}

#else
// Linux หรือแพลตฟอร์มอื่น - ไม่มี UI framework
struct PlatformColor {
    let red: Double
    let green: Double  
    let blue: Double
    let alpha: Double
}
#endif

// โค้ดนี้ทำงานได้ทุกแพลตฟอร์ม
func createThemeColor() -> PlatformColor {
    #if canImport(UIKit)
    return UIColor.platformBackground
    #elseif canImport(AppKit)
    return NSColor.platformBackground
    #else
    return PlatformColor(red: 1.0, green: 1.0, blue: 1.0, alpha: 1.0)
    #endif
}
```

### 1.5 Architecture Conditions

```swift
// ตรวจสอบ CPU Architecture
#if arch(arm64)
print("Running on ARM64 (Apple Silicon or iOS device)")
#elseif arch(x86_64)
print("Running on Intel x86_64")
#elseif arch(arm)
print("Running on 32-bit ARM (older iOS devices)")
#endif

// ตรวจสอบ Environment
#if targetEnvironment(simulator)
print("Running in iOS Simulator")
#elseif targetEnvironment(macCatalyst)
print("Running as Mac Catalyst app")
#else
print("Running on real device")
#endif
```

### 1.6 Swift Version Conditions

```swift
// ตรวจสอบ Swift version
#if swift(>=5.9)
// ใช้ Macros ได้
import Observation

@Observable
class UserProfile {
    var name: String = ""
    var age: Int = 0
}
#else
// ใช้ ObservableObject แทน
import Combine

class UserProfile: ObservableObject {
    @Published var name: String = ""
    @Published var age: Int = 0
}
#endif
```

### 1.7 Platform-Specific Code Isolation Strategies

กลยุทธ์ที่ดีในการแยกโค้ดเฉพาะแพลตฟอร์ม:

```swift
// กลยุทธ์ที่ 1: Protocol-based abstraction
protocol PlatformServices {
    func openURL(_ url: URL)
    func copyToClipboard(_ text: String)
    func pasteFromClipboard() -> String?
    func showAlert(title: String, message: String)
}

// การ implement บน iOS/macOS
#if canImport(UIKit)
import UIKit

class iOSPlatformServices: PlatformServices {
    func openURL(_ url: URL) {
        UIApplication.shared.open(url)
    }
    
    func copyToClipboard(_ text: String) {
        UIPasteboard.general.string = text
    }
    
    func pasteFromClipboard() -> String? {
        return UIPasteboard.general.string
    }
    
    func showAlert(title: String, message: String) {
        guard let windowScene = UIApplication.shared.connectedScenes.first as? UIWindowScene,
              let rootVC = windowScene.windows.first?.rootViewController else { return }
        
        let alert = UIAlertController(title: title, message: message, preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "OK", style: .default))
        rootVC.present(alert, animated: true)
    }
}

#elseif canImport(AppKit)
import AppKit

class macOSPlatformServices: PlatformServices {
    func openURL(_ url: URL) {
        NSWorkspace.shared.open(url)
    }
    
    func copyToClipboard(_ text: String) {
        NSPasteboard.general.clearContents()
        NSPasteboard.general.setString(text, forType: .string)
    }
    
    func pasteFromClipboard() -> String? {
        return NSPasteboard.general.string(forType: .string)
    }
    
    func showAlert(title: String, message: String) {
        let alert = NSAlert()
        alert.messageText = title
        alert.informativeText = message
        alert.runModal()
    }
}
#endif

// กลยุทธ์ที่ 2: Separate files per platform
// PlatformUtils_iOS.swift  - #if os(iOS)
// PlatformUtils_macOS.swift - #if os(macOS)
// PlatformUtils_common.swift - โค้ดร่วมกัน
```

---

## 2. Mac Catalyst Deep Dive

Mac Catalyst ช่วยให้นักพัฒนาสามารถนำ iPad app ของตนไปรันบน macOS ได้โดยใช้โค้ดเดิมเป็นส่วนใหญ่

### 2.1 การ Convert iOS App ไปเป็น Mac App

```swift
// ขั้นตอนการเปิดใช้งาน Mac Catalyst ใน Xcode:
// 1. เปิด Project settings
// 2. เลือก Target ของ iOS app
// 3. ไปที่ General tab
// 4. ติ๊ก "Mac" ใต้ Destination
// 5. เลือก "Optimize Interface for Mac" หรือ "Scale Interface to Match iPad"

// ตัวอย่างโค้ดที่ต้องปรับสำหรับ Mac Catalyst:

#if targetEnvironment(macCatalyst)
import AppKit
#else
import UIKit
#endif

class ViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
    }
    
    private func setupUI() {
        #if targetEnvironment(macCatalyst)
        // ปรับ UI สำหรับ Mac
        view.backgroundColor = UIColor(named: "MacBackground") ?? .white
        navigationController?.navigationBar.prefersLargeTitles = false
        #else
        // UI ปกติสำหรับ iOS
        view.backgroundColor = .systemBackground
        navigationController?.navigationBar.prefersLargeTitles = true
        #endif
    }
}
```

### 2.2 UIKit vs AppKit Differences

```swift
// ความแตกต่างหลักระหว่าง UIKit และ AppKit:

// 1. Color System
#if targetEnvironment(macCatalyst)
// ใน Mac Catalyst ใช้ UIColor แต่ได้รับ mapping จาก NSColor
let backgroundColor = UIColor.windowBackground  // Mac-specific color
#else
let backgroundColor = UIColor.systemBackground  // iOS color
#endif

// 2. Font Handling
import UIKit

func createBodyFont() -> UIFont {
    #if targetEnvironment(macCatalyst)
    // Mac ใช้ font size ที่เล็กกว่า iPad
    return UIFont.systemFont(ofSize: 13)
    #else
    return UIFont.preferredFont(forTextStyle: .body)
    #endif
}

// 3. Event Handling - Mouse vs Touch
class MultiInputView: UIView {
    
    // UIKit รองรับทั้ง touch และ mouse/trackpad ผ่าน UIHoverGestureRecognizer
    override func didMoveToWindow() {
        super.didMoveToWindow()
        
        #if targetEnvironment(macCatalyst)
        let hoverGesture = UIHoverGestureRecognizer(target: self, action: #selector(handleHover))
        addGestureRecognizer(hoverGesture)
        #endif
    }
    
    #if targetEnvironment(macCatalyst)
    @objc private func handleHover(_ gesture: UIHoverGestureRecognizer) {
        switch gesture.state {
        case .began:
            layer.borderColor = UIColor.systemBlue.cgColor
            layer.borderWidth = 2
        case .ended, .cancelled:
            layer.borderWidth = 0
        default:
            break
        }
    }
    #endif
}
```

### 2.3 Mac-Specific UI Adaptations

```swift
import UIKit

// การสร้าง Mac-style toolbar
class MacAdaptedViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        #if targetEnvironment(macCatalyst)
        setupMacToolbar()
        #endif
    }
    
    #if targetEnvironment(macCatalyst)
    private func setupMacToolbar() {
        // ซ่อน navigation bar ปกติ
        navigationController?.setNavigationBarHidden(true, animated: false)
        
        // ตั้งค่า NSToolbar ผ่าน UIWindowScene
        if let windowScene = view.window?.windowScene {
            windowScene.titlebar?.titleVisibility = .visible
            windowScene.title = "My Mac App"
        }
    }
    
    // Override เพื่อรองรับ NSToolbar
    override func buildMenu(with builder: UIMenuBuilder) {
        super.buildMenu(with: builder)
        
        // เพิ่ม custom menu items
        let refreshCommand = UIKeyCommand(
            title: "Refresh",
            action: #selector(refresh),
            input: "R",
            modifierFlags: .command
        )
        
        let refreshMenu = UIMenu(
            title: "",
            image: nil,
            identifier: UIMenu.Identifier("com.myapp.view"),
            options: .displayInline,
            children: [refreshCommand]
        )
        
        builder.insertChild(refreshMenu, atStartOfMenu: .view)
    }
    
    @objc func refresh() {
        // Refresh content
        print("Refreshing...")
    }
    #endif
}
```

### 2.4 Keyboard Shortcuts และ Menu Bar

```swift
import UIKit

// การเพิ่ม keyboard shortcuts สำหรับ Mac Catalyst
class DocumentViewController: UIViewController {
    
    // UIKeyCommand ใน responder chain
    override var keyCommands: [UIKeyCommand]? {
        return [
            UIKeyCommand(
                title: "Save",
                action: #selector(saveDocument),
                input: "S",
                modifierFlags: .command,
                discoverabilityTitle: "Save Document"
            ),
            UIKeyCommand(
                title: "Save As...",
                action: #selector(saveDocumentAs),
                input: "S",
                modifierFlags: [.command, .shift],
                discoverabilityTitle: "Save Document As"
            ),
            UIKeyCommand(
                title: "Find",
                action: #selector(openFind),
                input: "F",
                modifierFlags: .command,
                discoverabilityTitle: "Find in Document"
            )
        ]
    }
    
    @objc func saveDocument() {
        print("Saving document...")
    }
    
    @objc func saveDocumentAs() {
        print("Save as dialog...")
    }
    
    @objc func openFind() {
        print("Opening find...")
    }
    
    // การสร้าง Menu Bar items
    override func buildMenu(with builder: UIMenuBuilder) {
        super.buildMenu(with: builder)
        
        // สร้าง File menu custom items
        let newDocCommand = UICommand(
            title: "New Document",
            action: #selector(newDocument)
        )
        
        let fileMenu = UIMenu(
            title: "Document",
            children: [newDocCommand]
        )
        
        builder.insertChild(fileMenu, atStartOfMenu: .file)
    }
    
    @objc func newDocument() {
        print("Creating new document...")
    }
}
```

### 2.5 Touch Bar Support (สำหรับ MacBook Pro รุ่นเก่า)

```swift
#if targetEnvironment(macCatalyst)
import UIKit

// Touch Bar ใน Mac Catalyst ผ่าน NSTouchBar
// ต้องใช้ Objective-C bridging หรือ Swift interop กับ AppKit

// วิธีง่ายกว่าคือใช้ UIKeyCommand แทน Touch Bar
// เพราะ Touch Bar กำลังถูก deprecate แล้ว

extension DocumentViewController: NSTouchBarDelegate {
    func makeTouchBar() -> NSTouchBar? {
        let touchBar = NSTouchBar()
        touchBar.delegate = self
        touchBar.customizationIdentifier = NSTouchBar.CustomizationIdentifier("com.myapp.main")
        touchBar.defaultItemIdentifiers = [
            NSTouchBarItem.Identifier("save"),
            NSTouchBarItem.Identifier("share"),
            .flexibleSpace
        ]
        return touchBar
    }
    
    func touchBar(_ touchBar: NSTouchBar, makeItemForIdentifier identifier: NSTouchBarItem.Identifier) -> NSTouchBarItem? {
        switch identifier.rawValue {
        case "save":
            let item = NSButtonTouchBarItem(identifier: identifier, title: "Save", target: self, action: #selector(saveDocument))
            return item
        case "share":
            let item = NSButtonTouchBarItem(identifier: identifier, title: "Share", target: self, action: #selector(shareDocument))
            return item
        default:
            return nil
        }
    }
    
    @objc func shareDocument() {
        print("Sharing...")
    }
}
#endif
```

---

## 3. SwiftUI Multiplatform Apps

SwiftUI เป็นเครื่องมือที่ทรงพลังสำหรับการสร้าง multiplatform apps เพราะโค้ดส่วนใหญ่ใช้ร่วมกันได้

### 3.1 Shared UI Components Across Platforms

```swift
import SwiftUI

// Component ที่ทำงานได้ทุกแพลตฟอร์ม
struct UserCardView: View {
    let user: User
    
    var body: some View {
        HStack(spacing: 16) {
            AsyncImage(url: user.avatarURL) { image in
                image
                    .resizable()
                    .aspectRatio(contentMode: .fill)
            } placeholder: {
                Circle()
                    .fill(Color.gray.opacity(0.3))
            }
            .frame(width: avatarSize, height: avatarSize)
            .clipShape(Circle())
            
            VStack(alignment: .leading, spacing: 4) {
                Text(user.name)
                    .font(.headline)
                
                Text(user.email)
                    .font(.subheadline)
                    .foregroundColor(.secondary)
            }
            
            Spacer()
            
            statusBadge
        }
        .padding()
        .background(cardBackground)
        .cornerRadius(cornerRadius)
    }
    
    // Platform-adaptive sizing
    private var avatarSize: CGFloat {
        #if os(watchOS)
        return 32
        #elseif os(tvOS)
        return 80
        #else
        return 48
        #endif
    }
    
    private var cornerRadius: CGFloat {
        #if os(macOS)
        return 8
        #else
        return 12
        #endif
    }
    
    private var cardBackground: some View {
        #if os(macOS)
        return Color(NSColor.controlBackgroundColor)
        #elseif os(iOS) || os(visionOS)
        return Color(UIColor.secondarySystemBackground)
        #else
        return Color.black.opacity(0.3)
        #endif
    }
    
    private var statusBadge: some View {
        Circle()
            .fill(user.isOnline ? Color.green : Color.gray)
            .frame(width: 12, height: 12)
    }
}

struct User: Identifiable {
    let id: UUID
    let name: String
    let email: String
    let avatarURL: URL?
    var isOnline: Bool
}
```

### 3.2 Platform-Specific Views ด้วย #if os(macOS)

```swift
import SwiftUI

struct ContentView: View {
    @State private var selectedItem: String?
    @State private var items = ["Item 1", "Item 2", "Item 3", "Item 4"]
    
    var body: some View {
        #if os(macOS)
        macOSLayout
        #elseif os(iOS)
        iOSLayout
        #elseif os(watchOS)
        watchOSLayout
        #elseif os(tvOS)
        tvOSLayout
        #else
        defaultLayout
        #endif
    }
    
    #if os(macOS)
    private var macOSLayout: some View {
        NavigationSplitView {
            // Sidebar
            List(items, id: \.self, selection: $selectedItem) { item in
                Text(item)
            }
            .listStyle(.sidebar)
            .frame(minWidth: 200)
        } detail: {
            if let selected = selectedItem {
                DetailView(item: selected)
            } else {
                Text("Select an item")
                    .foregroundColor(.secondary)
            }
        }
        .frame(minWidth: 800, minHeight: 600)
    }
    #endif
    
    #if os(iOS)
    private var iOSLayout: some View {
        NavigationStack {
            List(items, id: \.self) { item in
                NavigationLink(item) {
                    DetailView(item: item)
                }
            }
            .navigationTitle("Items")
        }
    }
    #endif
    
    #if os(watchOS)
    private var watchOSLayout: some View {
        NavigationStack {
            List(items, id: \.self) { item in
                NavigationLink(item) {
                    Text(item)
                        .font(.caption)
                }
            }
        }
    }
    #endif
    
    #if os(tvOS)
    private var tvOSLayout: some View {
        HStack {
            List(items, id: \.self) { item in
                Button(item) {
                    selectedItem = item
                }
            }
            .frame(width: 400)
            
            if let selected = selectedItem {
                DetailView(item: selected)
            }
        }
    }
    #endif
    
    private var defaultLayout: some View {
        List(items, id: \.self) { item in
            Text(item)
        }
    }
}

struct DetailView: View {
    let item: String
    
    var body: some View {
        VStack {
            Text(item)
                .font(.largeTitle)
            Text("Detail content for \(item)")
                .foregroundColor(.secondary)
        }
        .padding()
        #if os(macOS)
        .frame(maxWidth: .infinity, maxHeight: .infinity)
        #endif
    }
}
```

### 3.3 NavigationSplitView สำหรับ iPad/Mac

```swift
import SwiftUI

struct ThreeColumnApp: View {
    @State private var selectedCategory: Category?
    @State private var selectedItem: Item?
    @State private var columnVisibility: NavigationSplitViewVisibility = .all
    
    var body: some View {
        NavigationSplitView(columnVisibility: $columnVisibility) {
            // Column 1: Categories (Sidebar)
            CategoryListView(selectedCategory: $selectedCategory)
                .navigationTitle("Categories")
                #if os(macOS)
                .navigationSplitViewColumnWidth(min: 180, ideal: 220, max: 280)
                #endif
            
        } content: {
            // Column 2: Items in Category
            if let category = selectedCategory {
                ItemListView(category: category, selectedItem: $selectedItem)
                    .navigationTitle(category.name)
                    #if os(macOS)
                    .navigationSplitViewColumnWidth(min: 250, ideal: 300, max: 400)
                    #endif
            } else {
                Text("Select a category")
                    .foregroundColor(.secondary)
            }
            
        } detail: {
            // Column 3: Item Detail
            if let item = selectedItem {
                ItemDetailView(item: item)
            } else {
                EmptyDetailView()
            }
        }
        .navigationSplitViewStyle(.balanced)
    }
}

struct Category: Identifiable {
    let id: UUID
    let name: String
    let icon: String
}

struct Item: Identifiable {
    let id: UUID
    let title: String
    let categoryId: UUID
    let content: String
}

struct CategoryListView: View {
    @Binding var selectedCategory: Category?
    
    let categories = [
        Category(id: UUID(), name: "Work", icon: "briefcase"),
        Category(id: UUID(), name: "Personal", icon: "person"),
        Category(id: UUID(), name: "Shopping", icon: "cart")
    ]
    
    var body: some View {
        List(categories, selection: $selectedCategory) { category in
            Label(category.name, systemImage: category.icon)
                .tag(category)
        }
    }
}

struct ItemListView: View {
    let category: Category
    @Binding var selectedItem: Item?
    
    var body: some View {
        List(sampleItems(for: category), selection: $selectedItem) { item in
            VStack(alignment: .leading) {
                Text(item.title)
                    .font(.headline)
                Text(item.content.prefix(50) + "...")
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
            .tag(item)
        }
    }
    
    private func sampleItems(for category: Category) -> [Item] {
        return [
            Item(id: UUID(), title: "Task 1", categoryId: category.id, content: "This is the content of task 1"),
            Item(id: UUID(), title: "Task 2", categoryId: category.id, content: "This is the content of task 2")
        ]
    }
}

struct ItemDetailView: View {
    let item: Item
    
    var body: some View {
        ScrollView {
            VStack(alignment: .leading, spacing: 16) {
                Text(item.title)
                    .font(.largeTitle)
                    .bold()
                
                Text(item.content)
                    .font(.body)
            }
            .padding()
            .frame(maxWidth: .infinity, alignment: .leading)
        }
        .navigationTitle(item.title)
        #if os(macOS)
        .navigationSubtitle("Item Detail")
        #endif
    }
}

struct EmptyDetailView: View {
    var body: some View {
        VStack(spacing: 16) {
            Image(systemName: "square.dashed")
                .font(.system(size: 60))
                .foregroundColor(.secondary)
            
            Text("Select an item")
                .font(.title2)
                .foregroundColor(.secondary)
        }
    }
}
```

### 3.4 NavigationStyle Differences

```swift
import SwiftUI

struct NavigationStyleDemo: View {
    var body: some View {
        Group {
            #if os(iOS)
            // iOS - NavigationStack (iOS 16+) หรือ NavigationView
            NavigationStack {
                ContentListView()
                    .navigationTitle("My App")
            }
            .navigationViewStyle(.stack) // สำหรับ iPhone
            
            #elseif os(macOS)
            // macOS - NavigationSplitView
            NavigationSplitView {
                SidebarView()
            } detail: {
                ContentListView()
            }
            
            #elseif os(tvOS)
            // tvOS - ใช้ NavigationStack
            NavigationStack {
                ContentListView()
            }
            
            #elseif os(watchOS)
            // watchOS - NavigationStack
            NavigationStack {
                WatchContentView()
            }
            #endif
        }
    }
}

struct ContentListView: View {
    var body: some View {
        List {
            Text("Item 1")
            Text("Item 2")
        }
    }
}

struct SidebarView: View {
    var body: some View {
        List {
            Label("Inbox", systemImage: "tray")
            Label("Drafts", systemImage: "doc")
            Label("Sent", systemImage: "paperplane")
        }
    }
}

struct WatchContentView: View {
    var body: some View {
        List {
            Text("Item 1").font(.caption2)
            Text("Item 2").font(.caption2)
        }
    }
}
```

---

## 4. tvOS Development

### 4.1 Focus Engine

tvOS ใช้ Focus Engine ในการควบคุมว่า element ใดกำลัง "focused" อยู่ ผู้ใช้นำทางด้วย Siri Remote หรือ game controller

```swift
import SwiftUI

// ใน SwiftUI focus ทำงานโดยอัตโนมัติ แต่เราสามารถ customize ได้
struct TVFocusableCard: View {
    let title: String
    let imageName: String
    @FocusState private var isFocused: Bool
    
    var body: some View {
        VStack {
            Image(systemName: imageName)
                .resizable()
                .aspectRatio(contentMode: .fit)
                .frame(height: 120)
                .foregroundColor(isFocused ? .white : .gray)
            
            Text(title)
                .font(.headline)
                .foregroundColor(isFocused ? .white : .secondary)
        }
        .frame(width: 200, height: 200)
        .background(isFocused ? Color.blue : Color.black.opacity(0.5))
        .cornerRadius(16)
        .scaleEffect(isFocused ? 1.1 : 1.0)
        .shadow(radius: isFocused ? 20 : 5)
        .animation(.easeInOut(duration: 0.2), value: isFocused)
        .focusable(true)
        .focused($isFocused)
        .onMoveCommand { direction in
            // จัดการการกด directional ด้วยตัวเอง (ถ้าต้องการ)
            print("Moved: \(direction)")
        }
    }
}

struct TVHomeScreen: View {
    let categories = ["Action", "Comedy", "Drama", "Documentary"]
    let items = (1...10).map { "Movie \($0)" }
    
    var body: some View {
        ScrollView {
            VStack(alignment: .leading, spacing: 40) {
                ForEach(categories, id: \.self) { category in
                    VStack(alignment: .leading) {
                        Text(category)
                            .font(.title2)
                            .bold()
                            .padding(.horizontal, 60)
                        
                        ScrollView(.horizontal, showsIndicators: false) {
                            HStack(spacing: 20) {
                                ForEach(items, id: \.self) { item in
                                    TVFocusableCard(
                                        title: "\(category): \(item)",
                                        imageName: "film"
                                    )
                                }
                            }
                            .padding(.horizontal, 60)
                        }
                    }
                }
            }
            .padding(.vertical, 40)
        }
    }
}
```

### 4.2 UIKit Focus Engine สำหรับ tvOS

```swift
#if os(tvOS)
import UIKit

class TVFocusableCollectionCell: UICollectionViewCell {
    
    private let imageView = UIImageView()
    private let titleLabel = UILabel()
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        setupUI()
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    private func setupUI() {
        // Setup image view
        imageView.contentMode = .scaleAspectFill
        imageView.clipsToBounds = true
        imageView.layer.cornerRadius = 8
        contentView.addSubview(imageView)
        
        // Setup label
        titleLabel.textAlignment = .center
        titleLabel.font = UIFont.systemFont(ofSize: 16, weight: .medium)
        titleLabel.textColor = .white
        contentView.addSubview(titleLabel)
        
        // Layout
        imageView.translatesAutoresizingMaskIntoConstraints = false
        titleLabel.translatesAutoresizingMaskIntoConstraints = false
        
        NSLayoutConstraint.activate([
            imageView.topAnchor.constraint(equalTo: contentView.topAnchor),
            imageView.leadingAnchor.constraint(equalTo: contentView.leadingAnchor),
            imageView.trailingAnchor.constraint(equalTo: contentView.trailingAnchor),
            imageView.heightAnchor.constraint(equalTo: contentView.heightAnchor, multiplier: 0.8),
            
            titleLabel.topAnchor.constraint(equalTo: imageView.bottomAnchor, constant: 8),
            titleLabel.leadingAnchor.constraint(equalTo: contentView.leadingAnchor),
            titleLabel.trailingAnchor.constraint(equalTo: contentView.trailingAnchor)
        ])
    }
    
    // tvOS Focus พิเศษ
    override func didUpdateFocus(in context: UIFocusUpdateContext, with coordinator: UIFocusAnimationCoordinator) {
        super.didUpdateFocus(in: context, with: coordinator)
        
        coordinator.addCoordinatedAnimations {
            if self.isFocused {
                self.transform = CGAffineTransform(scaleX: 1.1, y: 1.1)
                self.layer.shadowRadius = 20
                self.layer.shadowOpacity = 0.8
                self.titleLabel.textColor = .white
            } else {
                self.transform = .identity
                self.layer.shadowRadius = 0
                self.layer.shadowOpacity = 0
                self.titleLabel.textColor = UIColor.lightGray
            }
        }
    }
    
    // รองรับ parallax effect บน tvOS
    override var motionEffects: [UIMotionEffect] {
        get {
            if isFocused {
                let motionGroup = UIMotionEffectGroup()
                
                let xMotion = UIInterpolatingMotionEffect(keyPath: "layer.transform.translation.x", type: .tiltAlongHorizontalAxis)
                xMotion.minimumRelativeValue = -10
                xMotion.maximumRelativeValue = 10
                
                let yMotion = UIInterpolatingMotionEffect(keyPath: "layer.transform.translation.y", type: .tiltAlongVerticalAxis)
                yMotion.minimumRelativeValue = -10
                yMotion.maximumRelativeValue = 10
                
                motionGroup.motionEffects = [xMotion, yMotion]
                return [motionGroup]
            }
            return []
        }
        set { }
    }
}
#endif
```

### 4.3 Remote Control Handling

```swift
import SwiftUI

#if os(tvOS)
struct TVRemoteHandlingView: View {
    @State private var selectedIndex = 0
    @State private var isPlaying = false
    @State private var volume: Double = 0.5
    let items = ["Movie 1", "Movie 2", "Movie 3"]
    
    var body: some View {
        VStack {
            Text(items[selectedIndex])
                .font(.largeTitle)
            
            HStack {
                Button(action: { isPlaying.toggle() }) {
                    Image(systemName: isPlaying ? "pause.fill" : "play.fill")
                        .font(.system(size: 48))
                }
                
                Slider(value: $volume)
                    .frame(width: 300)
            }
        }
        .onPlayPauseCommand {
            // Siri Remote play/pause button
            isPlaying.toggle()
        }
        .onMoveCommand { direction in
            // D-pad / Siri Remote swipe
            switch direction {
            case .left:
                selectedIndex = max(0, selectedIndex - 1)
            case .right:
                selectedIndex = min(items.count - 1, selectedIndex + 1)
            default:
                break
            }
        }
        .onExitCommand {
            // Menu button / Back
            print("Exit pressed")
        }
    }
}
#endif
```

### 4.4 TVUIKit

```swift
#if os(tvOS)
import TVUIKit
import UIKit

// TVPosterView - แสดง poster-style content
class TVContentViewController: UIViewController {
    
    private let posterView = TVPosterView()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupPosterView()
    }
    
    private func setupPosterView() {
        posterView.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(posterView)
        
        NSLayoutConstraint.activate([
            posterView.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            posterView.centerYAnchor.constraint(equalTo: view.centerYAnchor),
            posterView.widthAnchor.constraint(equalToConstant: 500),
            posterView.heightAnchor.constraint(equalToConstant: 750)
        ])
        
        // ตั้งค่า poster
        posterView.image = UIImage(systemName: "film")
        posterView.title = "Epic Movie"
        posterView.subtitle = "2024 • Action • 2h 30m"
        
        // Focus overlay
        posterView.imageView.clipsToBounds = true
        posterView.imageView.layer.cornerRadius = 12
    }
}

// TVLockupView - layout for showing content with labels
class TVLockupViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        createLockupViews()
    }
    
    private func createLockupViews() {
        let stackView = UIStackView()
        stackView.axis = .horizontal
        stackView.spacing = 24
        stackView.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(stackView)
        
        NSLayoutConstraint.activate([
            stackView.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            stackView.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
        
        let movies = ["Movie A", "Movie B", "Movie C"]
        movies.forEach { movie in
            let lockup = TVLockupView()
            lockup.widthAnchor.constraint(equalToConstant: 250).isActive = true
            lockup.heightAnchor.constraint(equalToConstant: 400).isActive = true
            stackView.addArrangedSubview(lockup)
        }
    }
}
#endif
```

---

## 5. visionOS / Apple Vision Pro

### 5.1 WindowGroup สำหรับ visionOS

```swift
import SwiftUI
import RealityKit

#if os(visionOS)
@main
struct VisionProApp: App {
    var body: some Scene {
        // Window ปกติ - 2D content
        WindowGroup {
            ContentView()
        }
        .windowStyle(.plain)
        .defaultSize(width: 800, height: 600)
        
        // Volumetric Window - 3D content
        WindowGroup(id: "3d-model") {
            ModelView()
        }
        .windowStyle(.volumetric)
        .defaultSize(width: 0.6, height: 0.6, depth: 0.6, in: .meters)
        
        // ImmersiveSpace - fully immersive experience
        ImmersiveSpace(id: "immersive") {
            ImmersiveView()
        }
        .immersionStyle(selection: .constant(.mixed), in: .mixed)
    }
}

struct ContentView: View {
    @Environment(\.openImmersiveSpace) var openImmersiveSpace
    @Environment(\.dismissImmersiveSpace) var dismissImmersiveSpace
    @State private var isImmersive = false
    
    var body: some View {
        VStack(spacing: 20) {
            Text("Vision Pro App")
                .font(.extraLargeTitle)
            
            Button(isImmersive ? "Exit Immersive" : "Enter Immersive") {
                Task {
                    if isImmersive {
                        await dismissImmersiveSpace()
                    } else {
                        await openImmersiveSpace(id: "immersive")
                    }
                    isImmersive.toggle()
                }
            }
            .buttonStyle(.borderedProminent)
        }
        .padding(40)
    }
}
#endif
```

### 5.2 RealityKit สำหรับ visionOS

```swift
#if os(visionOS)
import SwiftUI
import RealityKit
import RealityKitContent

struct ImmersiveView: View {
    var body: some View {
        RealityView { content in
            // โหลด 3D scene จาก Reality Composer Pro
            if let scene = try? await Entity(named: "Scene", in: realityKitContentBundle) {
                content.add(scene)
            }
            
            // สร้าง entity ด้วยโค้ด
            let boxMesh = MeshResource.generateBox(size: 0.1)
            let boxMaterial = SimpleMaterial(color: .blue, roughness: 0.5, isMetallic: true)
            let boxEntity = ModelEntity(mesh: boxMesh, materials: [boxMaterial])
            
            boxEntity.position = SIMD3<Float>(0, 1.5, -1)
            
            // เพิ่ม collision สำหรับ interaction
            boxEntity.generateCollisionShapes(recursive: true)
            boxEntity.components.set(InputTargetComponent())
            
            content.add(boxEntity)
        } update: { content in
            // Update เมื่อ state เปลี่ยน
        }
        .gesture(
            TapGesture()
                .targetedToAnyEntity()
                .onEnded { value in
                    // เมื่อ tap entity
                    value.entity.model?.materials = [SimpleMaterial(color: .red, roughness: 0.5, isMetallic: false)]
                }
        )
    }
}

struct ModelView: View {
    @State private var rotation: Double = 0
    
    var body: some View {
        RealityView { content in
            // โหลด USDZ model
            if let model = try? await Entity(named: "toy_car") {
                model.scale = SIMD3<Float>(repeating: 0.5)
                content.add(model)
            }
        }
        .rotation3DEffect(.degrees(rotation), axis: (0, 1, 0))
        .gesture(
            DragGesture()
                .onChanged { value in
                    rotation = value.translation.width
                }
        )
    }
}
#endif
```

### 5.3 Hand Tracking

```swift
#if os(visionOS)
import SwiftUI
import RealityKit
import ARKit

struct HandTrackingView: View {
    @State private var handTrackingSession = HandTrackingSession()
    
    var body: some View {
        RealityView { content in
            // Setup hand tracking
        }
        .task {
            await handTrackingSession.start()
        }
    }
}

@MainActor
class HandTrackingSession: ObservableObject {
    private let session = ARKitSession()
    private let handTracking = HandTrackingProvider()
    
    @Published var leftHandAnchor: HandAnchor?
    @Published var rightHandAnchor: HandAnchor?
    
    func start() async {
        do {
            try await session.run([handTracking])
            
            for await update in handTracking.anchorUpdates {
                switch update.event {
                case .added, .updated:
                    let anchor = update.anchor
                    if anchor.chirality == .left {
                        leftHandAnchor = anchor
                    } else {
                        rightHandAnchor = anchor
                    }
                case .removed:
                    let anchor = update.anchor
                    if anchor.chirality == .left {
                        leftHandAnchor = nil
                    } else {
                        rightHandAnchor = nil
                    }
                }
            }
        } catch {
            print("Hand tracking error: \(error)")
        }
    }
    
    func getJointPosition(_ joint: HandSkeleton.JointName, from anchor: HandAnchor) -> simd_float4x4? {
        guard let skeleton = anchor.handSkeleton else { return nil }
        let jointTransform = skeleton.joint(joint).anchorFromJointTransform
        return anchor.originFromAnchorTransform * jointTransform
    }
}
#endif
```

### 5.4 Spatial Audio

```swift
#if os(visionOS)
import SwiftUI
import RealityKit
import AVFoundation

struct SpatialAudioView: View {
    var body: some View {
        RealityView { content in
            // สร้าง entity สำหรับ spatial audio
            let audioEntity = Entity()
            audioEntity.position = SIMD3<Float>(1, 1.5, -1) // วางตำแหน่งในอวกาศ
            
            // เพิ่ม SpatialAudioComponent
            var spatialAudio = SpatialAudioComponent()
            spatialAudio.gain = 0 // dB
            spatialAudio.directivity = .beam(focus: 0.5) // ทิศทางของเสียง
            audioEntity.components.set(spatialAudio)
            
            // โหลดและเล่นเสียง
            if let audioResource = try? await AudioFileResource(named: "ambience.mp3") {
                audioEntity.playAudio(audioResource)
            }
            
            content.add(audioEntity)
        }
    }
}

// AmbientAudio - เสียงรอบทิศทาง
class AmbientAudioManager {
    private var audioController: AudioPlaybackController?
    
    func playAmbientSound(named name: String) async {
        guard let resource = try? await AudioFileResource(named: name) else { return }
        
        var ambientAudio = AmbientAudioComponent()
        ambientAudio.gain = -10
        
        let entity = Entity()
        entity.components.set(ambientAudio)
        audioController = entity.playAudio(resource)
    }
    
    func stopAmbientSound() {
        audioController?.stop()
    }
}
#endif
```

---

## 6. Shared Business Logic

### 6.1 Platform-Agnostic Swift Packages

```swift
// Package.swift สำหรับ shared library
// swift-tools-version: 5.9

import PackageDescription

let package = Package(
    name: "SharedBusinessLogic",
    platforms: [
        .iOS(.v17),
        .macOS(.v14),
        .watchOS(.v10),
        .tvOS(.v17),
        .visionOS(.v1)
    ],
    products: [
        .library(
            name: "SharedBusinessLogic",
            targets: ["SharedBusinessLogic"]
        )
    ],
    dependencies: [
        .package(url: "https://github.com/apple/swift-log.git", from: "1.5.0"),
        .package(url: "https://github.com/apple/swift-collections.git", from: "1.0.0")
    ],
    targets: [
        .target(
            name: "SharedBusinessLogic",
            dependencies: [
                .product(name: "Logging", package: "swift-log"),
                .product(name: "Collections", package: "swift-collections")
            ],
            path: "Sources/SharedBusinessLogic"
        ),
        .testTarget(
            name: "SharedBusinessLogicTests",
            dependencies: ["SharedBusinessLogic"],
            path: "Tests/SharedBusinessLogicTests"
        )
    ]
)
```

### 6.2 Model Layer Sharing

```swift
// Sources/SharedBusinessLogic/Models/Note.swift
import Foundation

// Model ที่ใช้ร่วมกันทุกแพลตฟอร์ม
public struct Note: Codable, Identifiable, Hashable, Sendable {
    public let id: UUID
    public var title: String
    public var content: String
    public var tags: [String]
    public var createdAt: Date
    public var updatedAt: Date
    public var isFavorite: Bool
    
    public init(
        id: UUID = UUID(),
        title: String,
        content: String,
        tags: [String] = [],
        createdAt: Date = Date(),
        updatedAt: Date = Date(),
        isFavorite: Bool = false
    ) {
        self.id = id
        self.title = title
        self.content = content
        self.tags = tags
        self.createdAt = createdAt
        self.updatedAt = updatedAt
        self.isFavorite = isFavorite
    }
    
    public mutating func update(title: String? = nil, content: String? = nil) {
        if let title = title { self.title = title }
        if let content = content { self.content = content }
        self.updatedAt = Date()
    }
}

// Repository Protocol - ใช้ได้ทุกแพลตฟอร์ม
public protocol NoteRepository {
    func fetchAll() async throws -> [Note]
    func fetch(by id: UUID) async throws -> Note?
    func save(_ note: Note) async throws
    func delete(by id: UUID) async throws
    func search(query: String) async throws -> [Note]
}

// ViewModel - ใช้ร่วมกัน
import Observation

@Observable
public final class NotesViewModel {
    public var notes: [Note] = []
    public var isLoading = false
    public var error: Error?
    public var searchQuery = ""
    
    private let repository: NoteRepository
    
    public init(repository: NoteRepository) {
        self.repository = repository
    }
    
    public var filteredNotes: [Note] {
        if searchQuery.isEmpty {
            return notes
        }
        return notes.filter { note in
            note.title.localizedCaseInsensitiveContains(searchQuery) ||
            note.content.localizedCaseInsensitiveContains(searchQuery)
        }
    }
    
    @MainActor
    public func loadNotes() async {
        isLoading = true
        defer { isLoading = false }
        
        do {
            notes = try await repository.fetchAll()
        } catch {
            self.error = error
        }
    }
    
    @MainActor
    public func saveNote(_ note: Note) async {
        do {
            try await repository.save(note)
            await loadNotes()
        } catch {
            self.error = error
        }
    }
    
    @MainActor
    public func deleteNote(id: UUID) async {
        do {
            try await repository.delete(by: id)
            notes.removeAll { $0.id == id }
        } catch {
            self.error = error
        }
    }
}
```

### 6.3 Networking Layer Sharing

```swift
// Sources/SharedBusinessLogic/Networking/APIClient.swift
import Foundation

// Shared networking layer
public protocol APIClientProtocol {
    func fetch<T: Decodable>(_ endpoint: Endpoint) async throws -> T
    func upload<T: Decodable>(data: Data, to endpoint: Endpoint) async throws -> T
}

public struct Endpoint {
    public let path: String
    public let method: HTTPMethod
    public let queryItems: [URLQueryItem]?
    public let body: Encodable?
    
    public enum HTTPMethod: String {
        case get = "GET"
        case post = "POST"
        case put = "PUT"
        case delete = "DELETE"
        case patch = "PATCH"
    }
    
    public init(
        path: String,
        method: HTTPMethod = .get,
        queryItems: [URLQueryItem]? = nil,
        body: Encodable? = nil
    ) {
        self.path = path
        self.method = method
        self.queryItems = queryItems
        self.body = body
    }
}

public enum APIError: Error, LocalizedError {
    case invalidURL
    case invalidResponse
    case httpError(statusCode: Int, data: Data)
    case decodingError(Error)
    case networkError(Error)
    
    public var errorDescription: String? {
        switch self {
        case .invalidURL: return "Invalid URL"
        case .invalidResponse: return "Invalid response from server"
        case .httpError(let code, _): return "HTTP error: \(code)"
        case .decodingError(let error): return "Decoding error: \(error.localizedDescription)"
        case .networkError(let error): return "Network error: \(error.localizedDescription)"
        }
    }
}

public final class APIClient: APIClientProtocol {
    private let baseURL: URL
    private let session: URLSession
    private let decoder: JSONDecoder
    private let encoder: JSONEncoder
    
    public init(baseURL: URL, session: URLSession = .shared) {
        self.baseURL = baseURL
        self.session = session
        
        self.decoder = JSONDecoder()
        self.decoder.keyDecodingStrategy = .convertFromSnakeCase
        self.decoder.dateDecodingStrategy = .iso8601
        
        self.encoder = JSONEncoder()
        self.encoder.keyEncodingStrategy = .convertToSnakeCase
        self.encoder.dateEncodingStrategy = .iso8601
    }
    
    public func fetch<T: Decodable>(_ endpoint: Endpoint) async throws -> T {
        let request = try buildRequest(for: endpoint)
        
        do {
            let (data, response) = try await session.data(for: request)
            try validateResponse(response, data: data)
            return try decoder.decode(T.self, from: data)
        } catch let error as APIError {
            throw error
        } catch let decodingError as DecodingError {
            throw APIError.decodingError(decodingError)
        } catch {
            throw APIError.networkError(error)
        }
    }
    
    public func upload<T: Decodable>(data: Data, to endpoint: Endpoint) async throws -> T {
        var request = try buildRequest(for: endpoint)
        request.httpBody = data
        
        let (responseData, response) = try await session.data(for: request)
        try validateResponse(response, data: responseData)
        return try decoder.decode(T.self, from: responseData)
    }
    
    private func buildRequest(for endpoint: Endpoint) throws -> URLRequest {
        var components = URLComponents(url: baseURL.appendingPathComponent(endpoint.path), resolvingAgainstBaseURL: true)
        components?.queryItems = endpoint.queryItems
        
        guard let url = components?.url else {
            throw APIError.invalidURL
        }
        
        var request = URLRequest(url: url)
        request.httpMethod = endpoint.method.rawValue
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        
        if let body = endpoint.body {
            request.httpBody = try encoder.encode(body)
        }
        
        return request
    }
    
    private func validateResponse(_ response: URLResponse, data: Data) throws {
        guard let httpResponse = response as? HTTPURLResponse else {
            throw APIError.invalidResponse
        }
        
        guard 200...299 ~= httpResponse.statusCode else {
            throw APIError.httpError(statusCode: httpResponse.statusCode, data: data)
        }
    }
}
```

### 6.4 Testing Shared Code

```swift
// Tests/SharedBusinessLogicTests/NotesViewModelTests.swift
import XCTest
@testable import SharedBusinessLogic

// Mock Repository สำหรับ testing
class MockNoteRepository: NoteRepository {
    var notes: [Note] = []
    var shouldThrow = false
    
    func fetchAll() async throws -> [Note] {
        if shouldThrow { throw TestError.mockError }
        return notes
    }
    
    func fetch(by id: UUID) async throws -> Note? {
        if shouldThrow { throw TestError.mockError }
        return notes.first { $0.id == id }
    }
    
    func save(_ note: Note) async throws {
        if shouldThrow { throw TestError.mockError }
        if let index = notes.firstIndex(where: { $0.id == note.id }) {
            notes[index] = note
        } else {
            notes.append(note)
        }
    }
    
    func delete(by id: UUID) async throws {
        if shouldThrow { throw TestError.mockError }
        notes.removeAll { $0.id == id }
    }
    
    func search(query: String) async throws -> [Note] {
        if shouldThrow { throw TestError.mockError }
        return notes.filter { $0.title.contains(query) }
    }
}

enum TestError: Error {
    case mockError
}

@MainActor
class NotesViewModelTests: XCTestCase {
    var mockRepository: MockNoteRepository!
    var viewModel: NotesViewModel!
    
    override func setUp() {
        super.setUp()
        mockRepository = MockNoteRepository()
        viewModel = NotesViewModel(repository: mockRepository)
    }
    
    func testLoadNotes() async {
        // Arrange
        let note1 = Note(title: "Test 1", content: "Content 1")
        let note2 = Note(title: "Test 2", content: "Content 2")
        mockRepository.notes = [note1, note2]
        
        // Act
        await viewModel.loadNotes()
        
        // Assert
        XCTAssertEqual(viewModel.notes.count, 2)
        XCTAssertFalse(viewModel.isLoading)
        XCTAssertNil(viewModel.error)
    }
    
    func testLoadNotesFailure() async {
        // Arrange
        mockRepository.shouldThrow = true
        
        // Act
        await viewModel.loadNotes()
        
        // Assert
        XCTAssertNotNil(viewModel.error)
        XCTAssertTrue(viewModel.notes.isEmpty)
    }
    
    func testSearchFiltering() async {
        // Arrange
        mockRepository.notes = [
            Note(title: "Swift Guide", content: "Swift programming"),
            Note(title: "Python Tips", content: "Python guide"),
            Note(title: "SwiftUI Layout", content: "Layout guide")
        ]
        await viewModel.loadNotes()
        
        // Act
        viewModel.searchQuery = "Swift"
        
        // Assert
        XCTAssertEqual(viewModel.filteredNotes.count, 2)
    }
    
    func testSaveNote() async {
        // Arrange
        let note = Note(title: "New Note", content: "Content")
        
        // Act
        await viewModel.saveNote(note)
        
        // Assert
        XCTAssertEqual(viewModel.notes.count, 1)
        XCTAssertEqual(viewModel.notes.first?.title, "New Note")
    }
}
```

---

## 7. SwiftUI Adaptive Layouts

### 7.1 horizontalSizeClass และ verticalSizeClass

```swift
import SwiftUI

struct AdaptiveContentView: View {
    @Environment(\.horizontalSizeClass) var horizontalSizeClass
    @Environment(\.verticalSizeClass) var verticalSizeClass
    
    var body: some View {
        Group {
            if horizontalSizeClass == .compact {
                // iPhone portrait หรือ iPad split view ที่แคบ
                compactLayout
            } else if horizontalSizeClass == .regular && verticalSizeClass == .regular {
                // iPad portrait/landscape หรือ Mac
                regularLayout
            } else {
                // iPhone landscape
                landscapeLayout
            }
        }
    }
    
    private var compactLayout: some View {
        VStack {
            headerView
            contentList
        }
    }
    
    private var regularLayout: some View {
        HStack(spacing: 0) {
            sidebarView
                .frame(width: 300)
            
            Divider()
            
            contentList
        }
    }
    
    private var landscapeLayout: some View {
        HStack {
            contentList
        }
    }
    
    private var headerView: some View {
        Text("My App")
            .font(.largeTitle)
            .padding()
    }
    
    private var sidebarView: some View {
        List {
            Label("Home", systemImage: "house")
            Label("Search", systemImage: "magnifyingglass")
            Label("Profile", systemImage: "person")
        }
    }
    
    private var contentList: some View {
        List {
            ForEach(1...20, id: \.self) { item in
                Text("Item \(item)")
            }
        }
    }
}
```

### 7.2 @ScaledMetric

```swift
import SwiftUI

struct ScaledMetricDemo: View {
    // ScaledMetric ปรับขนาดตาม Dynamic Type
    @ScaledMetric(relativeTo: .body) var iconSize: CGFloat = 24
    @ScaledMetric(relativeTo: .headline) var spacing: CGFloat = 16
    @ScaledMetric var customSize: CGFloat = 32 // ใช้ .body เป็น base
    
    var body: some View {
        VStack(spacing: spacing) {
            HStack {
                Image(systemName: "star.fill")
                    .font(.system(size: iconSize))
                    .foregroundColor(.yellow)
                
                Text("Rated Item")
                    .font(.headline)
            }
            
            // Grid ที่ปรับขนาดตาม text size
            LazyVGrid(
                columns: [
                    GridItem(.adaptive(minimum: iconSize * 4))
                ]
            ) {
                ForEach(1...12, id: \.self) { item in
                    RoundedRectangle(cornerRadius: 8)
                        .fill(Color.blue.opacity(0.3))
                        .frame(height: customSize * 2)
                        .overlay(
                            Text("\(item)")
                                .font(.caption)
                        )
                }
            }
        }
        .padding(spacing)
    }
}
```

### 7.3 Adaptive Navigation Patterns

```swift
import SwiftUI

// Adaptive Navigation ที่ทำงานได้ดีทั้ง iPhone, iPad, Mac
struct AdaptiveNavigationApp: View {
    @Environment(\.horizontalSizeClass) var sizeClass
    @State private var selectedSection: AppSection? = .home
    @State private var path = NavigationPath()
    
    enum AppSection: String, CaseIterable, Identifiable {
        case home = "Home"
        case explore = "Explore"
        case library = "Library"
        case profile = "Profile"
        
        var id: String { rawValue }
        
        var icon: String {
            switch self {
            case .home: return "house"
            case .explore: return "magnifyingglass"
            case .library: return "books.vertical"
            case .profile: return "person.circle"
            }
        }
    }
    
    var body: some View {
        if sizeClass == .regular {
            // iPad/Mac: NavigationSplitView
            NavigationSplitView {
                List(AppSection.allCases, selection: $selectedSection) { section in
                    Label(section.rawValue, systemImage: section.icon)
                        .tag(section)
                }
                .listStyle(.sidebar)
                .navigationTitle("My App")
            } detail: {
                if let section = selectedSection {
                    sectionView(for: section)
                }
            }
        } else {
            // iPhone: TabView
            TabView {
                ForEach(AppSection.allCases) { section in
                    NavigationStack(path: $path) {
                        sectionView(for: section)
                    }
                    .tabItem {
                        Label(section.rawValue, systemImage: section.icon)
                    }
                    .tag(section)
                }
            }
        }
    }
    
    @ViewBuilder
    func sectionView(for section: AppSection) -> some View {
        switch section {
        case .home:
            HomeView()
        case .explore:
            ExploreView()
        case .library:
            LibraryView()
        case .profile:
            ProfileView()
        }
    }
}

struct HomeView: View {
    var body: some View {
        Text("Home")
            .navigationTitle("Home")
    }
}

struct ExploreView: View {
    var body: some View {
        Text("Explore")
            .navigationTitle("Explore")
    }
}

struct LibraryView: View {
    var body: some View {
        Text("Library")
            .navigationTitle("Library")
    }
}

struct ProfileView: View {
    var body: some View {
        Text("Profile")
            .navigationTitle("Profile")
    }
}
```

---

## 8. Multiplatform Project Setup ใน Xcode

### 8.1 Single Target, Multiple Destinations

```swift
// ในการตั้งค่า Xcode สำหรับ multiplatform:
// 1. File > New > Project
// 2. เลือก "Multiplatform" > "App"
// 3. Xcode จะสร้าง single target ที่รองรับ iOS, macOS, watchOS

// โครงสร้างไฟล์แบบ multiplatform:
// MyApp/
// ├── Shared/
// │   ├── ContentView.swift      (ใช้ร่วมกัน)
// │   ├── Models/
// │   └── ViewModels/
// ├── iOS/
// │   ├── AppDelegate.swift
// │   └── iOS_specific_views/
// ├── macOS/
// │   ├── AppDelegate.swift
// │   └── macOS_specific_views/
// └── Shared.xcassets/           (shared assets)

// MyApp.swift - Entry point สำหรับทุกแพลตฟอร์ม
import SwiftUI

@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        
        #if os(macOS)
        Settings {
            SettingsView()
        }
        #endif
    }
}

struct SettingsView: View {
    var body: some View {
        Form {
            Section("Appearance") {
                Toggle("Dark Mode", isOn: .constant(false))
            }
        }
        .padding()
        .frame(width: 400, height: 300)
    }
}
```

### 8.2 Build Configurations

```swift
// ใน Build Settings ของ Xcode:
// SWIFT_ACTIVE_COMPILATION_CONDITIONS = DEBUG
// สำหรับ Release: ไม่มี DEBUG

// ใช้ใน code:
#if DEBUG
let apiBaseURL = "https://api-dev.example.com"
let loggingEnabled = true
#else
let apiBaseURL = "https://api.example.com"
let loggingEnabled = false
#endif

// Custom build conditions:
// SWIFT_ACTIVE_COMPILATION_CONDITIONS = $(inherited) BETA_FEATURES
#if BETA_FEATURES
struct BetaFeatureBanner: View {
    var body: some View {
        Text("BETA")
            .font(.caption)
            .foregroundColor(.white)
            .padding(.horizontal, 8)
            .background(Color.orange)
            .cornerRadius(4)
    }
}
#endif
```

### 8.3 Xcode Schemes สำหรับ Multiplatform

```swift
// Scheme configuration สำหรับแต่ละแพลตฟอร์ม:
// 1. "MyApp (iOS)" - รัน/test บน iOS Simulator หรือ device
// 2. "MyApp (macOS)" - รัน/test บน Mac  
// 3. "MyApp (tvOS)" - รัน/test บน tvOS Simulator
// 4. "MyApp (watchOS)" - รัน/test บน watchOS Simulator

// xcconfig files สำหรับแต่ละ configuration:
// Debug.xcconfig
// Release.xcconfig
// Shared.xcconfig

// ตัวอย่าง xcconfig:
// PRODUCT_BUNDLE_IDENTIFIER = com.mycompany.myapp
// MARKETING_VERSION = 1.0.0
// CURRENT_PROJECT_VERSION = 1
```

---

## 9. Platform-Specific App Store Distribution

### 9.1 iOS App Store

```swift
// ข้อกำหนดสำหรับ iOS App Store:
// - Minimum deployment target: iOS 16 (ปัจจุบัน)
// - Required: Privacy manifest (PrivacyInfo.xcprivacy)
// - Required: App icons (ขนาดต่างๆ)
// - Optional: App Preview videos

// Privacy Info สำหรับ iOS:
// PrivacyInfo.xcprivacy
/*
{
    "NSPrivacyTracking": false,
    "NSPrivacyTrackingDomains": [],
    "NSPrivacyCollectedDataTypes": [
        {
            "NSPrivacyCollectedDataType": "NSPrivacyCollectedDataTypeEmailAddress",
            "NSPrivacyCollectedDataTypeLinked": true,
            "NSPrivacyCollectedDataTypeTracking": false,
            "NSPrivacyCollectedDataTypePurposes": ["NSPrivacyCollectedDataTypePurposeAppFunctionality"]
        }
    ],
    "NSPrivacyAccessedAPITypes": [
        {
            "NSPrivacyAccessedAPIType": "NSPrivacyAccessedAPICategoryFileTimestamp",
            "NSPrivacyAccessedAPITypeReasons": ["C617.1"]
        }
    ]
}
*/
```

### 9.2 Mac App Store vs Direct Distribution

```swift
// Mac App Store:
// - ต้องผ่าน sandboxing
// - ใช้ App Sandbox entitlement
// - จำกัดการเข้าถึง file system

// Direct Distribution (Developer ID):
// - ต้อง notarize app
// - ไม่ต้อง sandboxing (แต่แนะนำ)
// - เข้าถึง system resources ได้มากกว่า

// Entitlements สำหรับ Mac:
// MyApp.entitlements
/*
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" ...>
<plist version="1.0">
<dict>
    <key>com.apple.security.app-sandbox</key>
    <true/>
    <key>com.apple.security.network.client</key>
    <true/>
    <key>com.apple.security.files.user-selected.read-write</key>
    <true/>
</dict>
</plist>
*/

// Notarization script:
// xcrun notarytool submit MyApp.zip \
//   --apple-id "developer@example.com" \
//   --password "app-specific-password" \
//   --team-id "TEAMID123" \
//   --wait
```

---

## 10. Complete Cross-Platform App: Note-Taking App

### 10.1 App Architecture

```swift
// โปรเจกต์: CrossPlatformNotes
// รองรับ: iOS, macOS (Mac Catalyst และ native)

// Package.swift
// swift-tools-version: 5.9
import PackageDescription

let package = Package(
    name: "CrossPlatformNotes",
    platforms: [
        .iOS(.v17),
        .macOS(.v14)
    ],
    products: [
        .library(name: "NotesCore", targets: ["NotesCore"])
    ],
    targets: [
        .target(name: "NotesCore", path: "Sources/NotesCore"),
        .testTarget(name: "NotesCoreTests", dependencies: ["NotesCore"])
    ]
)
```

### 10.2 Core Data Model

```swift
// NotesDataModel.xcdatamodeld
// Entity: NoteEntity
// Attributes:
//   id: UUID
//   title: String
//   content: String
//   createdAt: Date
//   updatedAt: Date
//   isFavorite: Boolean
//   tags: Transformable (Array<String>)

import CoreData
import Foundation

// NoteEntity+CoreDataProperties.swift
extension NoteEntity {
    @nonobjc public class func fetchRequest() -> NSFetchRequest<NoteEntity> {
        return NSFetchRequest<NoteEntity>(entityName: "NoteEntity")
    }
    
    @NSManaged public var id: UUID
    @NSManaged public var title: String
    @NSManaged public var content: String
    @NSManaged public var createdAt: Date
    @NSManaged public var updatedAt: Date
    @NSManaged public var isFavorite: Bool
    @NSManaged public var tagsData: Data?
    
    var tags: [String] {
        get {
            guard let data = tagsData else { return [] }
            return (try? JSONDecoder().decode([String].self, from: data)) ?? []
        }
        set {
            tagsData = try? JSONEncoder().encode(newValue)
        }
    }
    
    func toNote() -> Note {
        Note(
            id: id,
            title: title,
            content: content,
            tags: tags,
            createdAt: createdAt,
            updatedAt: updatedAt,
            isFavorite: isFavorite
        )
    }
}
```

### 10.3 Core Data Repository

```swift
import CoreData
import Foundation

class CoreDataNoteRepository: NoteRepository {
    private let context: NSManagedObjectContext
    
    init(context: NSManagedObjectContext) {
        self.context = context
    }
    
    func fetchAll() async throws -> [Note] {
        let request = NoteEntity.fetchRequest()
        request.sortDescriptors = [NSSortDescriptor(key: "updatedAt", ascending: false)]
        
        return try await context.perform {
            let entities = try self.context.fetch(request)
            return entities.map { $0.toNote() }
        }
    }
    
    func fetch(by id: UUID) async throws -> Note? {
        let request = NoteEntity.fetchRequest()
        request.predicate = NSPredicate(format: "id == %@", id as CVarArg)
        request.fetchLimit = 1
        
        return try await context.perform {
            let entities = try self.context.fetch(request)
            return entities.first?.toNote()
        }
    }
    
    func save(_ note: Note) async throws {
        try await context.perform {
            let request = NoteEntity.fetchRequest()
            request.predicate = NSPredicate(format: "id == %@", note.id as CVarArg)
            
            let entity: NoteEntity
            if let existing = try self.context.fetch(request).first {
                entity = existing
            } else {
                entity = NoteEntity(context: self.context)
                entity.id = note.id
                entity.createdAt = note.createdAt
            }
            
            entity.title = note.title
            entity.content = note.content
            entity.tags = note.tags
            entity.updatedAt = note.updatedAt
            entity.isFavorite = note.isFavorite
            
            try self.context.save()
        }
    }
    
    func delete(by id: UUID) async throws {
        try await context.perform {
            let request = NoteEntity.fetchRequest()
            request.predicate = NSPredicate(format: "id == %@", id as CVarArg)
            
            let entities = try self.context.fetch(request)
            entities.forEach { self.context.delete($0) }
            
            try self.context.save()
        }
    }
    
    func search(query: String) async throws -> [Note] {
        let request = NoteEntity.fetchRequest()
        request.predicate = NSPredicate(
            format: "title CONTAINS[cd] %@ OR content CONTAINS[cd] %@",
            query, query
        )
        request.sortDescriptors = [NSSortDescriptor(key: "updatedAt", ascending: false)]
        
        return try await context.perform {
            let entities = try self.context.fetch(request)
            return entities.map { $0.toNote() }
        }
    }
}
```

### 10.4 Main App View (Multiplatform)

```swift
import SwiftUI

struct NotesAppView: View {
    @StateObject private var viewModel: NotesViewModel
    @State private var selectedNote: Note?
    @State private var showingNewNote = false
    
    init(repository: NoteRepository) {
        _viewModel = StateObject(wrappedValue: NotesViewModel(repository: repository))
    }
    
    var body: some View {
        #if os(macOS)
        macOSLayout
        #else
        iOSLayout
        #endif
    }
    
    #if os(macOS)
    private var macOSLayout: some View {
        NavigationSplitView {
            notesList
                .toolbar {
                    ToolbarItem(placement: .primaryAction) {
                        Button(action: { createNewNote() }) {
                            Image(systemName: "square.and.pencil")
                        }
                    }
                }
        } detail: {
            if let note = selectedNote {
                NoteEditorView(note: note, onSave: { updatedNote in
                    Task { await viewModel.saveNote(updatedNote) }
                })
            } else {
                emptyState
            }
        }
        .navigationTitle("Notes")
        .frame(minWidth: 900, minHeight: 600)
        .task { await viewModel.loadNotes() }
    }
    #endif
    
    private var iOSLayout: some View {
        NavigationStack {
            notesList
                .navigationTitle("Notes")
                .toolbar {
                    ToolbarItem(placement: .primaryAction) {
                        Button(action: { createNewNote() }) {
                            Image(systemName: "square.and.pencil")
                        }
                    }
                }
                .sheet(isPresented: $showingNewNote) {
                    NavigationStack {
                        NoteEditorView(note: Note(title: "", content: ""), onSave: { note in
                            Task { await viewModel.saveNote(note) }
                            showingNewNote = false
                        })
                        .navigationTitle("New Note")
                        .toolbar {
                            ToolbarItem(placement: .cancellationAction) {
                                Button("Cancel") { showingNewNote = false }
                            }
                        }
                    }
                }
        }
        .task { await viewModel.loadNotes() }
    }
    
    private var notesList: some View {
        Group {
            if viewModel.isLoading {
                ProgressView()
            } else if viewModel.filteredNotes.isEmpty {
                emptyState
            } else {
                List(viewModel.filteredNotes, selection: $selectedNote) { note in
                    NoteRowView(note: note)
                        .tag(note)
                        .swipeActions(edge: .trailing) {
                            Button(role: .destructive) {
                                Task { await viewModel.deleteNote(id: note.id) }
                            } label: {
                                Label("Delete", systemImage: "trash")
                            }
                        }
                }
                .searchable(text: $viewModel.searchQuery)
            }
        }
    }
    
    private var emptyState: some View {
        VStack(spacing: 16) {
            Image(systemName: "note.text")
                .font(.system(size: 60))
                .foregroundColor(.secondary)
            Text("No Notes")
                .font(.title2)
            Text("Create your first note")
                .foregroundColor(.secondary)
        }
    }
    
    private func createNewNote() {
        #if os(macOS)
        let newNote = Note(title: "Untitled", content: "")
        Task {
            await viewModel.saveNote(newNote)
            selectedNote = newNote
        }
        #else
        showingNewNote = true
        #endif
    }
}

struct NoteRowView: View {
    let note: Note
    
    var body: some View {
        VStack(alignment: .leading, spacing: 4) {
            HStack {
                Text(note.title.isEmpty ? "Untitled" : note.title)
                    .font(.headline)
                    .lineLimit(1)
                
                if note.isFavorite {
                    Image(systemName: "star.fill")
                        .foregroundColor(.yellow)
                        .font(.caption)
                }
            }
            
            Text(note.content)
                .font(.caption)
                .foregroundColor(.secondary)
                .lineLimit(2)
            
            Text(note.updatedAt, style: .relative)
                .font(.caption2)
                .foregroundColor(.tertiaryLabel)
        }
        .padding(.vertical, 2)
    }
}

struct NoteEditorView: View {
    @State private var note: Note
    let onSave: (Note) -> Void
    
    @FocusState private var focusedField: Field?
    
    enum Field {
        case title, content
    }
    
    init(note: Note, onSave: @escaping (Note) -> Void) {
        _note = State(initialValue: note)
        self.onSave = onSave
    }
    
    var body: some View {
        VStack(spacing: 0) {
            TextField("Title", text: $note.title)
                .font(.largeTitle)
                .bold()
                .focused($focusedField, equals: .title)
                .textFieldStyle(.plain)
                .padding()
            
            Divider()
            
            TextEditor(text: $note.content)
                .focused($focusedField, equals: .content)
                .font(.body)
                .padding()
        }
        .toolbar {
            ToolbarItem(placement: .primaryAction) {
                Button("Save") {
                    note.updatedAt = Date()
                    onSave(note)
                }
                .keyboardShortcut("S", modifiers: .command)
            }
        }
        .onAppear {
            if note.title.isEmpty {
                focusedField = .title
            } else {
                focusedField = .content
            }
        }
    }
}
```

---

## 11. แบบฝึกหัดพร้อมเฉลย

### แบบฝึกหัดที่ 1: Conditional Compilation

**โจทย์:** สร้าง `PlatformUtils` class ที่มีฟังก์ชัน `openURL(_:)` และ `hapticFeedback()` ที่ทำงานได้ทั้งบน iOS และ macOS

**เฉลย:**

```swift
import Foundation

#if canImport(UIKit)
import UIKit
#elseif canImport(AppKit)
import AppKit
#endif

class PlatformUtils {
    static let shared = PlatformUtils()
    
    private init() {}
    
    // เปิด URL บนทั้ง iOS และ macOS
    func openURL(_ url: URL) {
        #if os(iOS) || os(visionOS)
        UIApplication.shared.open(url)
        #elseif os(macOS)
        NSWorkspace.shared.open(url)
        #endif
    }
    
    // Haptic feedback (iOS เท่านั้น)
    func hapticFeedback(style: HapticStyle = .medium) {
        #if os(iOS)
        let generator: UIImpactFeedbackGenerator
        switch style {
        case .light:
            generator = UIImpactFeedbackGenerator(style: .light)
        case .medium:
            generator = UIImpactFeedbackGenerator(style: .medium)
        case .heavy:
            generator = UIImpactFeedbackGenerator(style: .heavy)
        }
        generator.prepare()
        generator.impactOccurred()
        #else
        // macOS, tvOS ไม่มี haptic (หรือจัดการต่างกัน)
        print("[PlatformUtils] Haptic feedback not supported on this platform")
        #endif
    }
    
    // Share content
    func share(text: String, from view: Any? = nil) {
        #if os(iOS)
        let activityVC = UIActivityViewController(activityItems: [text], applicationActivities: nil)
        
        if let viewController = view as? UIViewController {
            viewController.present(activityVC, animated: true)
        }
        #elseif os(macOS)
        let sharingPicker = NSSharingServicePicker(items: [text])
        if let button = view as? NSButton {
            sharingPicker.show(relativeTo: button.bounds, of: button, preferredEdge: .minY)
        }
        #endif
    }
    
    enum HapticStyle {
        case light, medium, heavy
    }
}
```

### แบบฝึกหัดที่ 2: Adaptive SwiftUI Layout

**โจทย์:** สร้าง `ProductGridView` ที่แสดงสินค้าเป็น grid บน iPad/Mac และ list บน iPhone

**เฉลย:**

```swift
import SwiftUI

struct Product: Identifiable {
    let id = UUID()
    let name: String
    let price: Double
    let imageName: String
}

struct ProductGridView: View {
    @Environment(\.horizontalSizeClass) var sizeClass
    
    let products = [
        Product(name: "Swift Book", price: 29.99, imageName: "book"),
        Product(name: "iOS Course", price: 49.99, imageName: "laptopcomputer"),
        Product(name: "Mac Mini", price: 599.99, imageName: "desktopcomputer"),
        Product(name: "iPhone", price: 999.99, imageName: "iphone"),
        Product(name: "iPad", price: 799.99, imageName: "ipad"),
        Product(name: "Watch", price: 399.99, imageName: "applewatch")
    ]
    
    var columns: [GridItem] {
        if sizeClass == .regular {
            // iPad/Mac: 3-4 คอลัมน์
            return [
                GridItem(.adaptive(minimum: 200, maximum: 250))
            ]
        } else {
            // iPhone: 2 คอลัมน์
            return [
                GridItem(.flexible()),
                GridItem(.flexible())
            ]
        }
    }
    
    var body: some View {
        NavigationStack {
            ScrollView {
                if sizeClass == .compact {
                    // iPhone: List-style
                    LazyVStack(spacing: 12) {
                        ForEach(products) { product in
                            ProductListRow(product: product)
                        }
                    }
                    .padding()
                } else {
                    // iPad/Mac: Grid-style
                    LazyVGrid(columns: columns, spacing: 20) {
                        ForEach(products) { product in
                            ProductCard(product: product)
                        }
                    }
                    .padding()
                }
            }
            .navigationTitle("Products")
        }
    }
}

struct ProductCard: View {
    let product: Product
    
    var body: some View {
        VStack(alignment: .leading) {
            Image(systemName: product.imageName)
                .resizable()
                .aspectRatio(contentMode: .fit)
                .frame(height: 100)
                .frame(maxWidth: .infinity)
                .background(Color.blue.opacity(0.1))
                .cornerRadius(8)
            
            VStack(alignment: .leading, spacing: 4) {
                Text(product.name)
                    .font(.headline)
                Text("$\(product.price, specifier: "%.2f")")
                    .font(.subheadline)
                    .foregroundColor(.secondary)
            }
            .padding(.horizontal, 8)
            .padding(.bottom, 8)
        }
        .background(Color(uiColor: .secondarySystemBackground))
        .cornerRadius(12)
    }
}

struct ProductListRow: View {
    let product: Product
    
    var body: some View {
        HStack(spacing: 16) {
            Image(systemName: product.imageName)
                .font(.system(size: 32))
                .frame(width: 60, height: 60)
                .background(Color.blue.opacity(0.1))
                .cornerRadius(8)
            
            VStack(alignment: .leading) {
                Text(product.name)
                    .font(.headline)
                Text("$\(product.price, specifier: "%.2f")")
                    .foregroundColor(.secondary)
            }
            
            Spacer()
            
            Image(systemName: "chevron.right")
                .foregroundColor(.secondary)
        }
        .padding()
        .background(Color(uiColor: .secondarySystemBackground))
        .cornerRadius(12)
    }
}

// iOS specific extension
extension Color {
    init(uiColor: UIColor) {
        self.init(uiColor)
    }
}
```

### แบบฝึกหัดที่ 3: Shared Business Logic

**โจทย์:** สร้าง Swift Package ที่มี `WeatherService` protocol และ `WeatherViewModel` ที่ใช้ได้ทุกแพลตฟอร์ม

**เฉลย:**

```swift
// Package.swift
// swift-tools-version: 5.9
import PackageDescription

let package = Package(
    name: "WeatherKit",
    platforms: [
        .iOS(.v17), .macOS(.v14), .watchOS(.v10), .tvOS(.v17)
    ],
    products: [
        .library(name: "WeatherKit", targets: ["WeatherKit"])
    ],
    targets: [
        .target(name: "WeatherKit", path: "Sources"),
        .testTarget(name: "WeatherKitTests", dependencies: ["WeatherKit"])
    ]
)

// Sources/WeatherModels.swift
import Foundation

public struct Weather: Codable, Sendable {
    public let cityName: String
    public let temperature: Double
    public let humidity: Double
    public let description: String
    public let icon: String
    public let fetchedAt: Date
    
    public var temperatureCelsius: Double { temperature }
    public var temperatureFahrenheit: Double { temperature * 9/5 + 32 }
    
    public init(cityName: String, temperature: Double, humidity: Double, description: String, icon: String) {
        self.cityName = cityName
        self.temperature = temperature
        self.humidity = humidity
        self.description = description
        self.icon = icon
        self.fetchedAt = Date()
    }
}

// Sources/WeatherServiceProtocol.swift
public protocol WeatherServiceProtocol: Sendable {
    func fetchWeather(for city: String) async throws -> Weather
    func fetchForecast(for city: String, days: Int) async throws -> [Weather]
}

// Sources/OpenWeatherService.swift
import Foundation

public final class OpenWeatherService: WeatherServiceProtocol {
    private let apiKey: String
    private let baseURL = URL(string: "https://api.openweathermap.org/data/2.5")!
    private let session: URLSession
    
    public init(apiKey: String, session: URLSession = .shared) {
        self.apiKey = apiKey
        self.session = session
    }
    
    public func fetchWeather(for city: String) async throws -> Weather {
        let url = baseURL
            .appendingPathComponent("weather")
            .appending(queryItems: [
                URLQueryItem(name: "q", value: city),
                URLQueryItem(name: "appid", value: apiKey),
                URLQueryItem(name: "units", value: "metric")
            ])
        
        let (data, response) = try await session.data(from: url)
        
        guard let httpResponse = response as? HTTPURLResponse,
              200...299 ~= httpResponse.statusCode else {
            throw WeatherError.invalidResponse
        }
        
        let apiResponse = try JSONDecoder().decode(WeatherAPIResponse.self, from: data)
        return apiResponse.toWeather()
    }
    
    public func fetchForecast(for city: String, days: Int) async throws -> [Weather] {
        // ใช้ forecast API
        return []
    }
}

enum WeatherError: Error {
    case invalidResponse
    case cityNotFound
    case networkError
}

struct WeatherAPIResponse: Decodable {
    let name: String
    let main: MainData
    let weather: [WeatherData]
    
    struct MainData: Decodable {
        let temp: Double
        let humidity: Double
    }
    
    struct WeatherData: Decodable {
        let description: String
        let icon: String
    }
    
    func toWeather() -> Weather {
        Weather(
            cityName: name,
            temperature: main.temp,
            humidity: main.humidity,
            description: weather.first?.description ?? "",
            icon: weather.first?.icon ?? ""
        )
    }
}

// Sources/WeatherViewModel.swift
import Observation

@Observable
public final class WeatherViewModel {
    public var currentWeather: Weather?
    public var isLoading = false
    public var errorMessage: String?
    public var cityName = "Bangkok"
    
    private let service: WeatherServiceProtocol
    
    public init(service: WeatherServiceProtocol) {
        self.service = service
    }
    
    @MainActor
    public func fetchWeather() async {
        guard !cityName.isEmpty else { return }
        
        isLoading = true
        errorMessage = nil
        
        do {
            currentWeather = try await service.fetchWeather(for: cityName)
        } catch {
            errorMessage = error.localizedDescription
        }
        
        isLoading = false
    }
}
```

### แบบฝึกหัดที่ 4: visionOS Basic App

**โจทย์:** สร้าง visionOS app ที่แสดง 3D sphere ที่ผู้ใช้สามารถ tap เพื่อเปลี่ยนสีได้

**เฉลย:**

```swift
#if os(visionOS)
import SwiftUI
import RealityKit

struct ColorChangingSphereView: View {
    @State private var sphereColor: UIColor = .blue
    private let colors: [UIColor] = [.blue, .red, .green, .yellow, .purple, .orange]
    @State private var colorIndex = 0
    
    var body: some View {
        RealityView { content in
            // สร้าง sphere
            let sphereMesh = MeshResource.generateSphere(radius: 0.15)
            let sphereMaterial = SimpleMaterial(color: sphereColor, roughness: 0.3, isMetallic: true)
            let sphereEntity = ModelEntity(mesh: sphereMesh, materials: [sphereMaterial])
            
            // ตั้งตำแหน่ง
            sphereEntity.position = SIMD3<Float>(0, 1.5, -1.5)
            
            // เพิ่ม interaction
            sphereEntity.generateCollisionShapes(recursive: false)
            sphereEntity.components.set(InputTargetComponent())
            
            // ชื่อ entity สำหรับอ้างอิง
            sphereEntity.name = "colorSphere"
            
            content.add(sphereEntity)
            
        } update: { content in
            // Update สีเมื่อ state เปลี่ยน
            if let sphere = content.entities.first(where: { $0.name == "colorSphere" }) as? ModelEntity {
                let newMaterial = SimpleMaterial(color: sphereColor, roughness: 0.3, isMetallic: true)
                sphere.model?.materials = [newMaterial]
            }
        }
        .gesture(
            TapGesture()
                .targetedToAnyEntity()
                .onEnded { _ in
                    colorIndex = (colorIndex + 1) % colors.count
                    sphereColor = colors[colorIndex]
                }
        )
        .overlay(alignment: .bottom) {
            VStack {
                Text("Tap the sphere to change color")
                    .font(.title3)
                    .padding()
                    .background(.regularMaterial, in: RoundedRectangle(cornerRadius: 12))
            }
            .padding(.bottom, 40)
        }
    }
}

@main
struct ColorSphereApp: App {
    var body: some Scene {
        WindowGroup {
            ColorChangingSphereView()
        }
        .windowStyle(.volumetric)
        .defaultSize(width: 1.0, height: 1.0, depth: 1.0, in: .meters)
    }
}
#endif
```

### แบบฝึกหัดที่ 5: Mac Catalyst Keyboard Shortcuts

**โจทย์:** สร้าง text editor ที่มี keyboard shortcuts สำหรับ bold, italic, underline บน Mac Catalyst

**เฉลย:**

```swift
import SwiftUI
import UIKit

struct RichTextEditor: View {
    @State private var text = NSMutableAttributedString(string: "")
    @State private var isBold = false
    @State private var isItalic = false
    
    var body: some View {
        VStack(spacing: 0) {
            // Toolbar
            HStack(spacing: 12) {
                Button(action: toggleBold) {
                    Image(systemName: "bold")
                        .foregroundColor(isBold ? .blue : .primary)
                }
                .keyboardShortcut("B", modifiers: .command)
                
                Button(action: toggleItalic) {
                    Image(systemName: "italic")
                        .foregroundColor(isItalic ? .blue : .primary)
                }
                .keyboardShortcut("I", modifiers: .command)
                
                Divider()
                    .frame(height: 24)
                
                Button(action: { print("Format") }) {
                    Image(systemName: "textformat")
                }
                
                Spacer()
            }
            .padding()
            .background(Color.gray.opacity(0.1))
            
            Divider()
            
            // Text area
            TextEditor(text: .constant("Type here..."))
                .font(.body)
                .padding()
        }
    }
    
    func toggleBold() {
        isBold.toggle()
        print("Bold: \(isBold)")
    }
    
    func toggleItalic() {
        isItalic.toggle()
        print("Italic: \(isItalic)")
    }
}

// UIKit version สำหรับ Mac Catalyst
#if targetEnvironment(macCatalyst)
class RichTextEditorViewController: UIViewController {
    
    private var textView = UITextView()
    
    override var keyCommands: [UIKeyCommand]? {
        return [
            UIKeyCommand(title: "Bold", action: #selector(toggleBold), input: "B", modifierFlags: .command),
            UIKeyCommand(title: "Italic", action: #selector(toggleItalic), input: "I", modifierFlags: .command),
            UIKeyCommand(title: "Underline", action: #selector(toggleUnderline), input: "U", modifierFlags: .command),
            UIKeyCommand(title: "Find", action: #selector(findText), input: "F", modifierFlags: .command)
        ]
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupTextView()
    }
    
    private func setupTextView() {
        textView.font = UIFont.preferredFont(forTextStyle: .body)
        textView.allowsEditingTextAttributes = true
        textView.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(textView)
        
        NSLayoutConstraint.activate([
            textView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            textView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            textView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            textView.bottomAnchor.constraint(equalTo: view.safeAreaLayoutGuide.bottomAnchor)
        ])
    }
    
    @objc private func toggleBold() {
        guard let selectedRange = textView.selectedTextRange else { return }
        let range = textView.selectedRange
        
        let attrs = textView.textStorage.attributes(at: range.location, effectiveRange: nil)
        let currentFont = attrs[.font] as? UIFont ?? UIFont.preferredFont(forTextStyle: .body)
        
        let newFont: UIFont
        if currentFont.fontDescriptor.symbolicTraits.contains(.traitBold) {
            newFont = UIFont(descriptor: currentFont.fontDescriptor.withSymbolicTraits(.init())!, size: currentFont.pointSize)
        } else {
            let boldDescriptor = currentFont.fontDescriptor.withSymbolicTraits(.traitBold)!
            newFont = UIFont(descriptor: boldDescriptor, size: currentFont.pointSize)
        }
        
        textView.textStorage.addAttribute(.font, value: newFont, range: range)
    }
    
    @objc private func toggleItalic() {
        guard textView.selectedRange.length > 0 else { return }
        let range = textView.selectedRange
        
        let attrs = textView.textStorage.attributes(at: range.location, effectiveRange: nil)
        let currentFont = attrs[.font] as? UIFont ?? UIFont.preferredFont(forTextStyle: .body)
        
        let newFont: UIFont
        if currentFont.fontDescriptor.symbolicTraits.contains(.traitItalic) {
            newFont = UIFont(descriptor: currentFont.fontDescriptor.withSymbolicTraits(.init())!, size: currentFont.pointSize)
        } else {
            let italicDescriptor = currentFont.fontDescriptor.withSymbolicTraits(.traitItalic)!
            newFont = UIFont(descriptor: italicDescriptor, size: currentFont.pointSize)
        }
        
        textView.textStorage.addAttribute(.font, value: newFont, range: range)
    }
    
    @objc private func toggleUnderline() {
        guard textView.selectedRange.length > 0 else { return }
        let range = textView.selectedRange
        
        let attrs = textView.textStorage.attributes(at: range.location, effectiveRange: nil)
        let isUnderlined = (attrs[.underlineStyle] as? Int) == NSUnderlineStyle.single.rawValue
        
        if isUnderlined {
            textView.textStorage.removeAttribute(.underlineStyle, range: range)
        } else {
            textView.textStorage.addAttribute(.underlineStyle, value: NSUnderlineStyle.single.rawValue, range: range)
        }
    }
    
    @objc private func findText() {
        let alertController = UIAlertController(title: "Find", message: nil, preferredStyle: .alert)
        alertController.addTextField { textField in
            textField.placeholder = "Search text..."
        }
        
        let findAction = UIAlertAction(title: "Find", style: .default) { _ in
            if let searchText = alertController.textFields?.first?.text {
                self.highlight(text: searchText)
            }
        }
        
        alertController.addAction(findAction)
        alertController.addAction(UIAlertAction(title: "Cancel", style: .cancel))
        present(alertController, animated: true)
    }
    
    private func highlight(text: String) {
        let fullText = textView.text ?? ""
        let attributedText = NSMutableAttributedString(attributedString: textView.attributedText)
        
        // Reset highlights
        attributedText.removeAttribute(.backgroundColor, range: NSRange(location: 0, length: attributedText.length))
        
        // Find and highlight
        var searchRange = fullText.startIndex..<fullText.endIndex
        while let range = fullText.range(of: text, options: .caseInsensitive, range: searchRange) {
            let nsRange = NSRange(range, in: fullText)
            attributedText.addAttribute(.backgroundColor, value: UIColor.yellow, range: nsRange)
            searchRange = range.upperBound..<fullText.endIndex
        }
        
        textView.attributedText = attributedText
    }
}
#endif
```

---

## สรุป

การพัฒนา Swift แบบ cross-platform ต้องการความเข้าใจในหลายด้าน:

1. **Conditional Compilation** - `#if os()`, `#if canImport()` ช่วยแยกโค้ดสำหรับแต่ละแพลตฟอร์ม
2. **Mac Catalyst** - วิธีที่ง่ายที่สุดในการนำ iOS app ไปบน macOS
3. **SwiftUI** - framework ที่แชร์โค้ดได้มากที่สุดระหว่างแพลตฟอร์ม
4. **tvOS** - ใช้ Focus Engine เป็นหัวใจหลักของ UX
5. **visionOS** - แพลตฟอร์มใหม่ที่ใช้ RealityKit และ spatial computing
6. **Shared Business Logic** - แยก business logic ออกจาก UI ใน Swift Package
7. **Adaptive Layouts** - ใช้ `horizontalSizeClass`, `@ScaledMetric` สำหรับ responsive UI

การทำความเข้าใจและนำหลักการเหล่านี้ไปใช้จะช่วยให้สามารถสร้างแอปที่มีคุณภาพสูงบนทุกแพลตฟอร์มของ Apple ได้อย่างมีประสิทธิภาพ
