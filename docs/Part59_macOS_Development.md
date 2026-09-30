# Part 59: macOS Development

## บทนำ

การพัฒนาแอปพลิเคชันสำหรับ macOS เป็นส่วนสำคัญของ Apple ecosystem ที่นักพัฒนา Swift ควรเรียนรู้ ในบทนี้เราจะสำรวจการพัฒนาแอปบน macOS ตั้งแต่พื้นฐานจนถึงความสามารถขั้นสูง รวมถึง AppKit, SwiftUI, Menu Bar Apps, Window Management, Document-based Apps และฟีเจอร์เฉพาะของ macOS

---

## 59.1 macOS Development Overview

macOS เป็นระบบปฏิบัติการที่มีประสิทธิภาพสูง เหมาะสำหรับการทำงานที่ต้องการความซับซ้อนและประสิทธิภาพ การพัฒนาแอปบน macOS มีความแตกต่างจาก iOS หลายประการ

### ความแตกต่างระหว่าง macOS และ iOS Development

```swift
// macOS มี features เพิ่มเติมที่ iOS ไม่มี:
// 1. Multiple Windows - หลายหน้าต่างในเวลาเดียวกัน
// 2. Menu Bar - แถบเมนูที่ด้านบนของจอ
// 3. Keyboard Shortcuts - ทางลัดคีย์บอร์ดที่ซับซ้อน
// 4. Drag and Drop - ลากและวางระหว่างแอป
// 5. File System Access - เข้าถึงระบบไฟล์ได้มากกว่า
// 6. Resizable Windows - ปรับขนาดหน้าต่างได้อิสระ
// 7. Touch Bar - แถบสัมผัสบน MacBook (Legacy)
// 8. Multiple Displays - หลายจอพร้อมกัน
```

### macOS Architecture

```
macOS Application
├── App Bundle (.app)
│   ├── Contents/
│   │   ├── MacOS/ (executable)
│   │   ├── Resources/ (assets)
│   │   └── Info.plist (configuration)
├── Frameworks
│   ├── AppKit (Traditional UI)
│   ├── SwiftUI (Modern UI)
│   ├── Foundation (Core services)
│   └── Cocoa (AppKit + Foundation)
└── System Services
    ├── Sandbox
    ├── Notarization
    └── Hardened Runtime
```

### เริ่มต้นสร้างโปรเจกต์ macOS

```swift
// ใน Xcode เลือก:
// File > New > Project > macOS > App

// AppDelegate.swift
import Cocoa

@main
class AppDelegate: NSObject, NSApplicationDelegate {
    
    func applicationDidFinishLaunching(_ aNotification: Notification) {
        // เริ่มต้นแอปพลิเคชัน
        print("แอปเริ่มต้นแล้ว")
    }

    func applicationWillTerminate(_ aNotification: Notification) {
        // ทำความสะอาดก่อนปิดแอป
        print("แอปกำลังปิด")
    }
    
    func applicationShouldTerminateAfterLastWindowClosed(_ sender: NSApplication) -> Bool {
        // ปิดแอปเมื่อปิดหน้าต่างสุดท้าย
        return true
    }
}
```

---

## 59.2 AppKit vs SwiftUI สำหรับ macOS

### AppKit (Traditional)

AppKit เป็น framework เดิมสำหรับการพัฒนา macOS ที่มีมาตั้งแต่ NeXT Step มีความสามารถครบครัน แต่ซับซ้อนกว่า

```swift
import Cocoa

// สร้าง Window ด้วย AppKit
class MainViewController: NSViewController {
    
    private let titleLabel = NSTextField(labelWithString: "สวัสดี macOS!")
    private let button = NSButton(title: "คลิก", target: nil, action: nil)
    private let textField = NSTextField()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
    }
    
    private func setupUI() {
        // ตั้งค่า Label
        titleLabel.translatesAutoresizingMaskIntoConstraints = false
        titleLabel.font = .systemFont(ofSize: 24, weight: .bold)
        titleLabel.alignment = .center
        view.addSubview(titleLabel)
        
        // ตั้งค่า Button
        button.translatesAutoresizingMaskIntoConstraints = false
        button.bezelStyle = .rounded
        button.target = self
        button.action = #selector(buttonClicked)
        view.addSubview(button)
        
        // ตั้งค่า TextField
        textField.translatesAutoresizingMaskIntoConstraints = false
        textField.placeholderString = "ป้อนข้อความ..."
        view.addSubview(textField)
        
        // Auto Layout Constraints
        NSLayoutConstraint.activate([
            titleLabel.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            titleLabel.topAnchor.constraint(equalTo: view.topAnchor, constant: 50),
            
            textField.topAnchor.constraint(equalTo: titleLabel.bottomAnchor, constant: 20),
            textField.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 20),
            textField.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -20),
            
            button.topAnchor.constraint(equalTo: textField.bottomAnchor, constant: 20),
            button.centerXAnchor.constraint(equalTo: view.centerXAnchor)
        ])
    }
    
    @objc private func buttonClicked() {
        let text = textField.stringValue
        titleLabel.stringValue = text.isEmpty ? "สวัสดี macOS!" : "คุณพิมพ์: \(text)"
    }
}
```

### SwiftUI สำหรับ macOS

```swift
import SwiftUI

// สร้าง View ด้วย SwiftUI
struct ContentView: View {
    @State private var message = "สวัสดี macOS!"
    @State private var inputText = ""
    
    var body: some View {
        VStack(spacing: 20) {
            Text(message)
                .font(.largeTitle)
                .fontWeight(.bold)
                .multilineTextAlignment(.center)
            
            TextField("ป้อนข้อความ...", text: $inputText)
                .textFieldStyle(.roundedBorder)
                .frame(maxWidth: 300)
            
            Button("คลิก") {
                message = inputText.isEmpty ? "สวัสดี macOS!" : "คุณพิมพ์: \(inputText)"
            }
            .buttonStyle(.borderedProminent)
        }
        .padding(40)
        .frame(minWidth: 400, minHeight: 300)
    }
}

// App Entry Point
@main
struct MyMacApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

### เปรียบเทียบ AppKit vs SwiftUI

| Feature | AppKit | SwiftUI |
|---------|--------|---------|
| อายุ | ตั้งแต่ NeXT (1988) | 2019 |
| ความยืดหยุ่น | สูงมาก | สูง (เพิ่มขึ้นเรื่อยๆ) |
| ความซับซ้อน | สูง | ต่ำกว่า |
| Platform | macOS เท่านั้น | Cross-platform |
| Performance | ดีเยี่ยม | ดี (บางกรณียังสู้ AppKit ไม่ได้) |
| Live Preview | ไม่มี | มี |
| Data Binding | Manual | Automatic |

---

## 59.3 Mac Catalyst

Mac Catalyst ช่วยให้นำแอป iPad มาใช้บน macOS ได้โดยไม่ต้องเขียนใหม่ทั้งหมด

### เปิดใช้ Mac Catalyst

```swift
// ใน Xcode > Project Settings > Targets > ติ๊ก "Mac Catalyst"

// ตรวจสอบ Platform ใน Code
#if targetEnvironment(macCatalyst)
    // Code สำหรับ Mac Catalyst
    print("กำลังรันบน Mac Catalyst")
#else
    // Code สำหรับ iOS/iPadOS
    print("กำลังรันบน iOS")
#endif
```

### ปรับแต่ง UI สำหรับ Mac Catalyst

```swift
import UIKit

class ViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        #if targetEnvironment(macCatalyst)
        setupForMac()
        #else
        setupForIOS()
        #endif
    }
    
    private func setupForMac() {
        // ปรับ UI สำหรับ Mac
        // ซ่อน navigation bar
        navigationController?.setNavigationBarHidden(true, animated: false)
        
        // ปรับ font size สำหรับ Mac
        view.backgroundColor = .windowBackgroundColor
    }
    
    private func setupForIOS() {
        // ปรับ UI สำหรับ iOS
        view.backgroundColor = .systemBackground
    }
}
```

### เพิ่ม Menu Bar Items ด้วย Mac Catalyst

```swift
import UIKit

// ใน AppDelegate หรือ SceneDelegate
class AppDelegate: UIResponder, UIApplicationDelegate {
    
    override func buildMenu(with builder: UIMenuBuilder) {
        super.buildMenu(with: builder)
        
        // เพิ่ม Custom Menu
        let customCommand = UIKeyCommand(
            title: "ทำสิ่งพิเศษ",
            action: #selector(doSomethingSpecial),
            input: "S",
            modifierFlags: [.command, .shift]
        )
        
        let customMenu = UIMenu(
            title: "เมนูพิเศษ",
            children: [customCommand]
        )
        
        builder.insertSibling(customMenu, afterMenu: .view)
    }
    
    @objc func doSomethingSpecial() {
        print("ทำสิ่งพิเศษแล้ว!")
    }
}
```

---

## 59.4 SwiftUI สำหรับ macOS โดยเฉพาะ

SwiftUI มี modifier และ component พิเศษสำหรับ macOS

### WindowGroup และ Window

```swift
import SwiftUI

@main
struct MacApp: App {
    var body: some Scene {
        // หน้าต่างหลัก
        WindowGroup {
            ContentView()
        }
        .windowStyle(.titleBar)
        .windowToolbarStyle(.unified)
        .defaultSize(width: 800, height: 600)
        .defaultPosition(.center)
        
        // หน้าต่างสำหรับการตั้งค่า
        Settings {
            SettingsView()
        }
    }
}
```

### macOS-specific Modifiers

```swift
import SwiftUI

struct MacContentView: View {
    @State private var selectedItem: String?
    
    var body: some View {
        NavigationSplitView {
            // Sidebar
            List(["รายการ 1", "รายการ 2", "รายการ 3"], id: \.self, selection: $selectedItem) { item in
                Label(item, systemImage: "doc")
            }
            .navigationTitle("Sidebar")
            .listStyle(.sidebar)
        } detail: {
            // Detail View
            if let item = selectedItem {
                Text("เลือก: \(item)")
                    .navigationTitle(item)
            } else {
                Text("เลือกรายการจาก Sidebar")
                    .foregroundColor(.secondary)
            }
        }
        .toolbar {
            ToolbarItem(placement: .primaryAction) {
                Button(action: {}) {
                    Label("เพิ่ม", systemImage: "plus")
                }
            }
        }
    }
}
```

### NSViewRepresentable - ใช้ AppKit ใน SwiftUI

```swift
import SwiftUI
import AppKit

// Wrap AppKit Component ใน SwiftUI
struct NSTextViewRepresentable: NSViewRepresentable {
    @Binding var text: String
    
    func makeNSView(context: Context) -> NSScrollView {
        let scrollView = NSTextView.scrollablePlainDocumentContentTextView()
        let textView = scrollView.documentView as! NSTextView
        textView.delegate = context.coordinator
        textView.font = .systemFont(ofSize: 14)
        textView.isRichText = false
        textView.allowsUndo = true
        return scrollView
    }
    
    func updateNSView(_ nsView: NSScrollView, context: Context) {
        let textView = nsView.documentView as! NSTextView
        if textView.string != text {
            textView.string = text
        }
    }
    
    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }
    
    class Coordinator: NSObject, NSTextViewDelegate {
        var parent: NSTextViewRepresentable
        
        init(_ parent: NSTextViewRepresentable) {
            self.parent = parent
        }
        
        func textDidChange(_ notification: Notification) {
            guard let textView = notification.object as? NSTextView else { return }
            parent.text = textView.string
        }
    }
}

// ใช้งาน
struct EditorView: View {
    @State private var text = ""
    
    var body: some View {
        VStack {
            Text("Editor")
                .font(.headline)
            
            NSTextViewRepresentable(text: $text)
                .frame(minHeight: 200)
                .border(Color.secondary, width: 1)
        }
        .padding()
    }
}
```

---

## 59.5 Menu Bar Apps (NSStatusItem)

Menu Bar Apps เป็นแอปที่อยู่ในแถบเมนูด้านบนของ macOS ไม่มี dock icon และไม่มีหน้าต่างหลัก

### สร้าง Menu Bar App พื้นฐาน

```swift
import Cocoa
import SwiftUI

class AppDelegate: NSObject, NSApplicationDelegate {
    var statusItem: NSStatusItem!
    var popover: NSPopover!
    
    func applicationDidFinishLaunching(_ notification: Notification) {
        // ซ่อน Dock Icon
        NSApp.setActivationPolicy(.accessory)
        
        // สร้าง Status Item
        statusItem = NSStatusBar.system.statusItem(withLength: NSStatusItem.squareLength)
        
        if let button = statusItem.button {
            button.image = NSImage(systemSymbolName: "star.fill", accessibilityDescription: "Menu Bar App")
            button.action = #selector(togglePopover)
            button.target = self
        }
        
        // สร้าง Popover
        popover = NSPopover()
        popover.contentSize = NSSize(width: 300, height: 400)
        popover.behavior = .transient
        popover.contentViewController = NSHostingController(rootView: MenuBarContentView())
    }
    
    @objc func togglePopover() {
        if let button = statusItem.button {
            if popover.isShown {
                popover.performClose(nil)
            } else {
                popover.show(relativeTo: button.bounds, of: button, preferredEdge: .minY)
                popover.contentViewController?.view.window?.makeKey()
            }
        }
    }
}

// SwiftUI View สำหรับ Popover
struct MenuBarContentView: View {
    @State private var message = "สวัสดี Menu Bar!"
    
    var body: some View {
        VStack(spacing: 16) {
            Text("Menu Bar App")
                .font(.headline)
            
            Text(message)
                .foregroundColor(.secondary)
            
            Divider()
            
            Button("รีเซ็ต") {
                message = "รีเซ็ตแล้ว!"
            }
            
            Button("ออกจากแอป") {
                NSApplication.shared.terminate(nil)
            }
            .foregroundColor(.red)
        }
        .padding()
        .frame(width: 280)
    }
}
```

### Menu Bar App ที่ซับซ้อนขึ้น

```swift
import Cocoa

class MenuBarManager: NSObject {
    var statusItem: NSStatusItem!
    var menu: NSMenu!
    
    override init() {
        super.init()
        setupStatusItem()
        setupMenu()
    }
    
    private func setupStatusItem() {
        statusItem = NSStatusBar.system.statusItem(withLength: NSStatusItem.variableLength)
        
        if let button = statusItem.button {
            button.title = "MyApp"
            button.image = NSImage(systemSymbolName: "cpu", accessibilityDescription: "App")
            button.imagePosition = .imageLeading
        }
        
        statusItem.menu = buildMenu()
    }
    
    private func buildMenu() -> NSMenu {
        let menu = NSMenu()
        
        // เพิ่ม Items
        menu.addItem(NSMenuItem(title: "เปิดแอป", action: #selector(openApp), keyEquivalent: "o"))
        menu.addItem(NSMenuItem.separator())
        
        // Submenu
        let submenuItem = NSMenuItem(title: "ตัวเลือก", action: nil, keyEquivalent: "")
        let submenu = NSMenu()
        submenu.addItem(NSMenuItem(title: "ตั้งค่า 1", action: #selector(option1), keyEquivalent: ""))
        submenu.addItem(NSMenuItem(title: "ตั้งค่า 2", action: #selector(option2), keyEquivalent: ""))
        submenuItem.submenu = submenu
        menu.addItem(submenuItem)
        
        menu.addItem(NSMenuItem.separator())
        menu.addItem(NSMenuItem(title: "ออก", action: #selector(quit), keyEquivalent: "q"))
        
        // ตั้งค่า Target
        for item in menu.items {
            item.target = self
        }
        
        return menu
    }
    
    private func setupMenu() {}
    
    @objc func openApp() {
        NSApp.activate(ignoringOtherApps: true)
    }
    
    @objc func option1() {
        print("เลือกตัวเลือก 1")
    }
    
    @objc func option2() {
        print("เลือกตัวเลือก 2")
    }
    
    @objc func quit() {
        NSApp.terminate(nil)
    }
}
```

---

## 59.6 Window Management

macOS รองรับการจัดการหน้าต่างที่ซับซ้อน

### สร้างและจัดการ Window

```swift
import Cocoa

class WindowManager: NSObject {
    var windows: [NSWindow] = []
    
    // สร้างหน้าต่างใหม่
    func createWindow(title: String, content: NSViewController) -> NSWindow {
        let window = NSWindow(
            contentRect: NSRect(x: 0, y: 0, width: 600, height: 400),
            styleMask: [.titled, .closable, .miniaturizable, .resizable],
            backing: .buffered,
            defer: false
        )
        
        window.title = title
        window.contentViewController = content
        window.center()
        window.setFrameAutosaveName(title) // จำตำแหน่งหน้าต่าง
        
        // กำหนดขนาดขั้นต่ำ
        window.minSize = NSSize(width: 400, height: 300)
        
        // กำหนดขนาดสูงสุด
        window.maxSize = NSSize(width: 1200, height: 900)
        
        windows.append(window)
        return window
    }
    
    // แสดงหน้าต่าง
    func showWindow(_ window: NSWindow) {
        window.makeKeyAndOrderFront(nil)
        NSApp.activate(ignoringOtherApps: true)
    }
    
    // ปิดหน้าต่างทั้งหมด
    func closeAllWindows() {
        windows.forEach { $0.close() }
        windows.removeAll()
    }
}

// NSWindowDelegate
class MyWindowController: NSWindowController, NSWindowDelegate {
    
    override func windowDidLoad() {
        super.windowDidLoad()
        window?.delegate = self
    }
    
    func windowWillClose(_ notification: Notification) {
        print("หน้าต่างกำลังปิด")
    }
    
    func windowDidResize(_ notification: Notification) {
        if let size = window?.frame.size {
            print("ขนาดหน้าต่าง: \(size.width) x \(size.height)")
        }
    }
    
    func windowDidEnterFullScreen(_ notification: Notification) {
        print("เข้าสู่ Full Screen")
    }
    
    func windowDidExitFullScreen(_ notification: Notification) {
        print("ออกจาก Full Screen")
    }
}
```

### Multiple Windows ใน SwiftUI

```swift
import SwiftUI

// ใช้ openWindow Environment
struct ContentView: View {
    @Environment(\.openWindow) var openWindow
    
    var body: some View {
        VStack {
            Button("เปิดหน้าต่างใหม่") {
                openWindow(id: "detail-window")
            }
            
            Button("เปิด Inspector") {
                openWindow(id: "inspector")
            }
        }
    }
}

@main
struct MultiWindowApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        
        // หน้าต่าง Detail
        Window("รายละเอียด", id: "detail-window") {
            DetailWindowView()
        }
        .defaultSize(width: 400, height: 300)
        
        // Inspector Panel
        Window("Inspector", id: "inspector") {
            InspectorView()
        }
        .windowStyle(.hiddenTitleBar)
        .defaultPosition(.topTrailing)
    }
}
```

---

## 59.7 macOS-specific UI Elements

### NSTableView

NSTableView เป็น component สำหรับแสดงข้อมูลในรูปแบบตาราง

```swift
import Cocoa

// Data Model
struct Person {
    let name: String
    let age: Int
    let email: String
}

class TableViewController: NSViewController {
    
    @IBOutlet weak var tableView: NSTableView!
    
    var people: [Person] = [
        Person(name: "สมชาย", age: 30, email: "somchai@example.com"),
        Person(name: "สมหญิง", age: 25, email: "somying@example.com"),
        Person(name: "มานี", age: 35, email: "manee@example.com"),
        Person(name: "มีนา", age: 28, email: "meena@example.com")
    ]
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        tableView.delegate = self
        tableView.dataSource = self
        
        // ลงทะเบียน Column
        setupColumns()
        
        // Enable sorting
        tableView.sortDescriptors = [
            NSSortDescriptor(key: "name", ascending: true)
        ]
    }
    
    private func setupColumns() {
        // ล้าง Columns เดิม
        while tableView.numberOfColumns > 0 {
            tableView.removeTableColumn(tableView.tableColumns[0])
        }
        
        // Column: ชื่อ
        let nameColumn = NSTableColumn(identifier: NSUserInterfaceItemIdentifier("name"))
        nameColumn.title = "ชื่อ"
        nameColumn.width = 150
        nameColumn.sortDescriptorPrototype = NSSortDescriptor(key: "name", ascending: true)
        tableView.addTableColumn(nameColumn)
        
        // Column: อายุ
        let ageColumn = NSTableColumn(identifier: NSUserInterfaceItemIdentifier("age"))
        ageColumn.title = "อายุ"
        ageColumn.width = 80
        tableView.addTableColumn(ageColumn)
        
        // Column: อีเมล
        let emailColumn = NSTableColumn(identifier: NSUserInterfaceItemIdentifier("email"))
        emailColumn.title = "อีเมล"
        emailColumn.width = 200
        tableView.addTableColumn(emailColumn)
    }
}

// MARK: - NSTableViewDataSource
extension TableViewController: NSTableViewDataSource {
    
    func numberOfRows(in tableView: NSTableView) -> Int {
        return people.count
    }
    
    func tableView(_ tableView: NSTableView, sortDescriptorsDidChange oldDescriptors: [NSSortDescriptor]) {
        let sortedPeople = (people as NSArray).sortedArray(using: tableView.sortDescriptors) as! [Person]
        people = sortedPeople
        tableView.reloadData()
    }
}

// MARK: - NSTableViewDelegate
extension TableViewController: NSTableViewDelegate {
    
    func tableView(_ tableView: NSTableView, viewFor tableColumn: NSTableColumn?, row: Int) -> NSView? {
        let person = people[row]
        
        let identifier = NSUserInterfaceItemIdentifier("cell")
        var cell = tableView.makeView(withIdentifier: identifier, owner: self) as? NSTableCellView
        
        if cell == nil {
            cell = NSTableCellView()
            cell?.identifier = identifier
            
            let textField = NSTextField(labelWithString: "")
            textField.translatesAutoresizingMaskIntoConstraints = false
            cell?.addSubview(textField)
            cell?.textField = textField
            
            NSLayoutConstraint.activate([
                textField.leadingAnchor.constraint(equalTo: cell!.leadingAnchor, constant: 4),
                textField.centerYAnchor.constraint(equalTo: cell!.centerYAnchor)
            ])
        }
        
        switch tableColumn?.identifier.rawValue {
        case "name":
            cell?.textField?.stringValue = person.name
        case "age":
            cell?.textField?.stringValue = "\(person.age)"
        case "email":
            cell?.textField?.stringValue = person.email
        default:
            break
        }
        
        return cell
    }
    
    func tableViewSelectionDidChange(_ notification: Notification) {
        let row = tableView.selectedRow
        if row >= 0 {
            let person = people[row]
            print("เลือก: \(person.name)")
        }
    }
}
```

### NSOutlineView

NSOutlineView ใช้สำหรับแสดงข้อมูลแบบ hierarchical (tree structure)

```swift
import Cocoa

// Tree Data Model
class TreeNode {
    let name: String
    var children: [TreeNode]
    var isExpanded: Bool = false
    
    init(name: String, children: [TreeNode] = []) {
        self.name = name
        self.children = children
    }
    
    var isLeaf: Bool { children.isEmpty }
}

class OutlineViewController: NSViewController {
    
    @IBOutlet weak var outlineView: NSOutlineView!
    
    // สร้าง Tree Data
    var rootItems: [TreeNode] = {
        let docs = TreeNode(name: "เอกสาร", children: [
            TreeNode(name: "รายงาน.docx"),
            TreeNode(name: "นำเสนอ.pptx")
        ])
        
        let images = TreeNode(name: "รูปภาพ", children: [
            TreeNode(name: "vacation", children: [
                TreeNode(name: "beach.jpg"),
                TreeNode(name: "mountain.jpg")
            ]),
            TreeNode(name: "profile.png")
        ])
        
        let downloads = TreeNode(name: "ดาวน์โหลด")
        
        return [docs, images, downloads]
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        outlineView.delegate = self
        outlineView.dataSource = self
        outlineView.reloadData()
        
        // ขยาย Items เริ่มต้น
        outlineView.expandItem(nil, expandChildren: false)
    }
}

// MARK: - NSOutlineViewDataSource
extension OutlineViewController: NSOutlineViewDataSource {
    
    func outlineView(_ outlineView: NSOutlineView, numberOfChildrenOfItem item: Any?) -> Int {
        if item == nil {
            return rootItems.count
        }
        let node = item as! TreeNode
        return node.children.count
    }
    
    func outlineView(_ outlineView: NSOutlineView, child index: Int, ofItem item: Any?) -> Any {
        if item == nil {
            return rootItems[index]
        }
        let node = item as! TreeNode
        return node.children[index]
    }
    
    func outlineView(_ outlineView: NSOutlineView, isItemExpandable item: Any) -> Bool {
        let node = item as! TreeNode
        return !node.isLeaf
    }
}

// MARK: - NSOutlineViewDelegate
extension OutlineViewController: NSOutlineViewDelegate {
    
    func outlineView(_ outlineView: NSOutlineView, viewFor tableColumn: NSTableColumn?, item: Any) -> NSView? {
        let node = item as! TreeNode
        
        let identifier = NSUserInterfaceItemIdentifier("cell")
        var cell = outlineView.makeView(withIdentifier: identifier, owner: self) as? NSTableCellView
        
        if cell == nil {
            cell = NSTableCellView()
            cell?.identifier = identifier
            
            let imageView = NSImageView()
            imageView.translatesAutoresizingMaskIntoConstraints = false
            
            let textField = NSTextField(labelWithString: "")
            textField.translatesAutoresizingMaskIntoConstraints = false
            
            cell?.addSubview(imageView)
            cell?.addSubview(textField)
            cell?.imageView = imageView
            cell?.textField = textField
            
            NSLayoutConstraint.activate([
                imageView.leadingAnchor.constraint(equalTo: cell!.leadingAnchor, constant: 2),
                imageView.centerYAnchor.constraint(equalTo: cell!.centerYAnchor),
                imageView.widthAnchor.constraint(equalToConstant: 16),
                imageView.heightAnchor.constraint(equalToConstant: 16),
                
                textField.leadingAnchor.constraint(equalTo: imageView.trailingAnchor, constant: 4),
                textField.centerYAnchor.constraint(equalTo: cell!.centerYAnchor)
            ])
        }
        
        cell?.textField?.stringValue = node.name
        
        if node.isLeaf {
            cell?.imageView?.image = NSImage(systemSymbolName: "doc", accessibilityDescription: nil)
        } else {
            cell?.imageView?.image = NSImage(systemSymbolName: "folder", accessibilityDescription: nil)
        }
        
        return cell
    }
}
```

---

## 59.8 Toolbar และ Sidebar

### NSToolbar

```swift
import Cocoa

class ToolbarViewController: NSViewController, NSToolbarDelegate {
    
    var toolbar: NSToolbar!
    
    // กำหนด Identifiers
    let newDocumentItem = NSToolbarItem.Identifier("newDocument")
    let saveItem = NSToolbarItem.Identifier("save")
    let searchItem = NSToolbarItem.Identifier("search")
    let shareItem = NSToolbarItem.Identifier("share")
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupToolbar()
    }
    
    private func setupToolbar() {
        toolbar = NSToolbar(identifier: "MainToolbar")
        toolbar.delegate = self
        toolbar.allowsUserCustomization = true
        toolbar.autosavesConfiguration = true
        toolbar.displayMode = .iconAndLabel
        
        view.window?.toolbar = toolbar
    }
    
    // MARK: - NSToolbarDelegate
    
    func toolbar(_ toolbar: NSToolbar, itemForItemIdentifier itemIdentifier: NSToolbarItem.Identifier, willBeInsertedIntoToolbar flag: Bool) -> NSToolbarItem? {
        
        switch itemIdentifier {
        case newDocumentItem:
            let item = NSToolbarItem(itemIdentifier: itemIdentifier)
            item.label = "เอกสารใหม่"
            item.paletteLabel = "สร้างเอกสารใหม่"
            item.toolTip = "สร้างเอกสารใหม่"
            item.image = NSImage(systemSymbolName: "doc.badge.plus", accessibilityDescription: nil)
            item.action = #selector(newDocument)
            item.target = self
            return item
            
        case saveItem:
            let item = NSToolbarItem(itemIdentifier: itemIdentifier)
            item.label = "บันทึก"
            item.paletteLabel = "บันทึกเอกสาร"
            item.image = NSImage(systemSymbolName: "square.and.arrow.down", accessibilityDescription: nil)
            item.action = #selector(saveDocument)
            item.target = self
            return item
            
        case searchItem:
            let searchToolbarItem = NSSearchToolbarItem(itemIdentifier: itemIdentifier)
            searchToolbarItem.searchField.placeholderString = "ค้นหา..."
            searchToolbarItem.searchField.action = #selector(searchChanged)
            searchToolbarItem.searchField.target = self
            return searchToolbarItem
            
        default:
            return nil
        }
    }
    
    func toolbarDefaultItemIdentifiers(_ toolbar: NSToolbar) -> [NSToolbarItem.Identifier] {
        return [newDocumentItem, saveItem, .flexibleSpace, searchItem]
    }
    
    func toolbarAllowedItemIdentifiers(_ toolbar: NSToolbar) -> [NSToolbarItem.Identifier] {
        return [newDocumentItem, saveItem, searchItem, shareItem, .space, .flexibleSpace, .separator]
    }
    
    @objc func newDocument() {
        print("สร้างเอกสารใหม่")
    }
    
    @objc func saveDocument() {
        print("บันทึกเอกสาร")
    }
    
    @objc func searchChanged(_ sender: NSSearchField) {
        print("ค้นหา: \(sender.stringValue)")
    }
}
```

### SwiftUI Toolbar

```swift
import SwiftUI

struct ToolbarContentView: View {
    @State private var searchText = ""
    @State private var sortOrder = SortOrder.name
    
    enum SortOrder {
        case name, date, size
    }
    
    var body: some View {
        List {
            ForEach(filteredItems, id: \.self) { item in
                Text(item)
            }
        }
        .searchable(text: $searchText, prompt: "ค้นหาไฟล์...")
        .navigationTitle("ไฟล์ทั้งหมด")
        .toolbar {
            ToolbarItemGroup(placement: .primaryAction) {
                Button(action: addNew) {
                    Label("เพิ่ม", systemImage: "plus")
                }
                
                Picker("เรียงตาม", selection: $sortOrder) {
                    Label("ชื่อ", systemImage: "textformat").tag(SortOrder.name)
                    Label("วันที่", systemImage: "calendar").tag(SortOrder.date)
                    Label("ขนาด", systemImage: "arrow.up.arrow.down").tag(SortOrder.size)
                }
                .pickerStyle(.segmented)
            }
            
            ToolbarItem(placement: .secondaryAction) {
                Button(action: share) {
                    Label("แชร์", systemImage: "square.and.arrow.up")
                }
            }
        }
    }
    
    var filteredItems: [String] {
        let items = ["เอกสาร.pdf", "รูปภาพ.jpg", "สเปรดชีต.xlsx", "นำเสนอ.pptx"]
        if searchText.isEmpty {
            return items
        }
        return items.filter { $0.contains(searchText) }
    }
    
    func addNew() { print("เพิ่มใหม่") }
    func share() { print("แชร์") }
}
```

---

## 59.9 Document-based Apps

Document-based apps ช่วยให้ผู้ใช้จัดการไฟล์เอกสารได้เหมือน TextEdit หรือ Preview

### สร้าง Document-based App

```swift
import Cocoa

// Document Model
class TextDocument: NSDocument {
    
    var text: String = ""
    
    override init() {
        super.init()
    }
    
    override class var autosavesInPlace: Bool {
        return true
    }
    
    // อ่านข้อมูลจากไฟล์
    override func read(from data: Data, ofType typeName: String) throws {
        guard let content = String(data: data, encoding: .utf8) else {
            throw NSError(domain: NSCocoaErrorDomain, code: NSFileReadCorruptFileError)
        }
        text = content
    }
    
    // เขียนข้อมูลลงไฟล์
    override func data(ofType typeName: String) throws -> Data {
        guard let data = text.data(using: .utf8) else {
            throw NSError(domain: NSCocoaErrorDomain, code: NSFileWriteInapplicableStringEncodingError)
        }
        return data
    }
    
    // สร้าง Window Controller
    override func makeWindowControllers() {
        let windowController = DocumentWindowController()
        windowController.document = self
        addWindowController(windowController)
        windowController.showWindow(nil)
    }
}

// Document Window Controller
class DocumentWindowController: NSWindowController {
    
    override func windowDidLoad() {
        super.windowDidLoad()
        
        if let doc = document as? TextDocument {
            // แสดงเนื้อหาเอกสาร
            print("โหลดเอกสาร: \(doc.text.prefix(100))")
        }
    }
}
```

### Document-based App ด้วย SwiftUI

```swift
import SwiftUI
import UniformTypeIdentifiers

// Document Definition
struct TextFile: FileDocument {
    var text: String
    
    static var readableContentTypes: [UTType] { [.plainText] }
    
    init(text: String = "") {
        self.text = text
    }
    
    init(configuration: ReadConfiguration) throws {
        guard let data = configuration.file.regularFileContents,
              let string = String(data: data, encoding: .utf8)
        else {
            throw CocoaError(.fileReadCorruptFile)
        }
        text = string
    }
    
    func fileWrapper(configuration: WriteConfiguration) throws -> FileWrapper {
        let data = text.data(using: .utf8)!
        return .init(regularFileWithContents: data)
    }
}

// App Entry Point
@main
struct DocumentApp: App {
    var body: some Scene {
        DocumentGroup(newDocument: TextFile()) { file in
            DocumentView(document: file.$document)
        }
    }
}

// Document View
struct DocumentView: View {
    @Binding var document: TextFile
    
    var body: some View {
        TextEditor(text: $document.text)
            .font(.system(.body, design: .monospaced))
            .padding()
            .toolbar {
                ToolbarItem(placement: .primaryAction) {
                    Text("\(document.text.count) ตัวอักษร")
                        .foregroundColor(.secondary)
                }
            }
    }
}
```

---

## 59.10 File Handling บน macOS

macOS มีวิธีจัดการไฟล์ที่หลากหลาย

### Open Panel และ Save Panel

```swift
import Cocoa

class FileOperationsManager {
    
    // เปิด File Open Panel
    func openFile(completion: @escaping (URL?) -> Void) {
        let panel = NSOpenPanel()
        panel.allowsMultipleSelection = false
        panel.canChooseDirectories = false
        panel.canChooseFiles = true
        panel.allowedContentTypes = [.plainText, .pdf, .image]
        panel.title = "เลือกไฟล์"
        panel.message = "กรุณาเลือกไฟล์ที่ต้องการเปิด"
        panel.prompt = "เปิด"
        
        panel.begin { response in
            if response == .OK {
                completion(panel.url)
            } else {
                completion(nil)
            }
        }
    }
    
    // เปิด File Open Panel สำหรับหลายไฟล์
    func openMultipleFiles(completion: @escaping ([URL]) -> Void) {
        let panel = NSOpenPanel()
        panel.allowsMultipleSelection = true
        panel.canChooseDirectories = false
        panel.canChooseFiles = true
        
        panel.begin { response in
            if response == .OK {
                completion(panel.urls)
            } else {
                completion([])
            }
        }
    }
    
    // Save Panel
    func saveFile(content: String, completion: @escaping (Bool) -> Void) {
        let panel = NSSavePanel()
        panel.allowedContentTypes = [.plainText]
        panel.nameFieldStringValue = "เอกสารใหม่.txt"
        panel.title = "บันทึกไฟล์"
        panel.message = "เลือกที่บันทึกไฟล์"
        panel.prompt = "บันทึก"
        
        panel.begin { [weak self] response in
            guard response == .OK, let url = panel.url else {
                completion(false)
                return
            }
            
            do {
                try content.write(to: url, atomically: true, encoding: .utf8)
                completion(true)
            } catch {
                print("ไม่สามารถบันทึกไฟล์: \(error)")
                completion(false)
            }
        }
    }
    
    // อ่านไฟล์
    func readFile(url: URL) -> String? {
        do {
            return try String(contentsOf: url, encoding: .utf8)
        } catch {
            print("ไม่สามารถอ่านไฟล์: \(error)")
            return nil
        }
    }
    
    // ลบไฟล์
    func deleteFile(url: URL) -> Bool {
        do {
            try FileManager.default.removeItem(at: url)
            return true
        } catch {
            print("ไม่สามารถลบไฟล์: \(error)")
            return false
        }
    }
}
```

### Security-Scoped Bookmarks

```swift
import Foundation

class SecurityScopedBookmarkManager {
    
    // สร้าง Bookmark สำหรับ URL ที่มีสิทธิ์เข้าถึง
    func createBookmark(for url: URL) -> Data? {
        do {
            let bookmarkData = try url.bookmarkData(
                options: .withSecurityScope,
                includingResourceValuesForKeys: nil,
                relativeTo: nil
            )
            return bookmarkData
        } catch {
            print("ไม่สามารถสร้าง bookmark: \(error)")
            return nil
        }
    }
    
    // โหลด URL จาก Bookmark
    func resolveBookmark(_ bookmarkData: Data) -> URL? {
        var isStale = false
        do {
            let url = try URL(
                resolvingBookmarkData: bookmarkData,
                options: .withSecurityScope,
                relativeTo: nil,
                bookmarkDataIsStale: &isStale
            )
            
            if isStale {
                print("Bookmark หมดอายุ ต้องสร้างใหม่")
            }
            
            return url
        } catch {
            print("ไม่สามารถโหลด bookmark: \(error)")
            return nil
        }
    }
    
    // เข้าถึงไฟล์ด้วย Security Scope
    func accessFile(url: URL, action: () -> Void) {
        let hasAccess = url.startAccessingSecurityScopedResource()
        defer {
            if hasAccess {
                url.stopAccessingSecurityScopedResource()
            }
        }
        action()
    }
}
```

---

## 59.11 Drag and Drop

### รับ Drag and Drop

```swift
import Cocoa

class DragDropView: NSView {
    
    var acceptedTypes: [NSPasteboard.PasteboardType] = [
        .fileURL,
        .string,
        .png,
        .tiff
    ]
    
    var onFilesDropped: (([URL]) -> Void)?
    var onTextDropped: ((String) -> Void)?
    var onImageDropped: ((NSImage) -> Void)?
    
    override init(frame frameRect: NSRect) {
        super.init(frame: frameRect)
        setup()
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
        setup()
    }
    
    private func setup() {
        registerForDraggedTypes(acceptedTypes)
        wantsLayer = true
        layer?.cornerRadius = 8
        layer?.borderWidth = 2
        layer?.borderColor = NSColor.separatorColor.cgColor
    }
    
    // MARK: - Drag Operations
    
    override func draggingEntered(_ sender: NSDraggingInfo) -> NSDragOperation {
        // เปลี่ยนสีเมื่อลากเข้ามา
        layer?.borderColor = NSColor.controlAccentColor.cgColor
        layer?.backgroundColor = NSColor.controlAccentColor.withAlphaComponent(0.1).cgColor
        return .copy
    }
    
    override func draggingExited(_ sender: NSDraggingInfo?) {
        // คืนค่าสี
        layer?.borderColor = NSColor.separatorColor.cgColor
        layer?.backgroundColor = nil
    }
    
    override func draggingUpdated(_ sender: NSDraggingInfo) -> NSDragOperation {
        return .copy
    }
    
    override func performDragOperation(_ sender: NSDraggingInfo) -> Bool {
        let pasteboard = sender.draggingPasteboard
        
        // ตรวจสอบประเภทข้อมูล
        if let urls = pasteboard.readObjects(forClasses: [NSURL.self], options: nil) as? [URL] {
            onFilesDropped?(urls)
            return true
        }
        
        if let string = pasteboard.string(forType: .string) {
            onTextDropped?(string)
            return true
        }
        
        if let imageData = pasteboard.data(forType: .tiff),
           let image = NSImage(data: imageData) {
            onImageDropped?(image)
            return true
        }
        
        return false
    }
    
    override func concludeDragOperation(_ sender: NSDraggingInfo?) {
        // คืนค่าสี
        layer?.borderColor = NSColor.separatorColor.cgColor
        layer?.backgroundColor = nil
    }
}

// ใช้งาน DragDropView
class DropZoneViewController: NSViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let dropView = DragDropView(frame: NSRect(x: 50, y: 50, width: 300, height: 200))
        view.addSubview(dropView)
        
        dropView.onFilesDropped = { urls in
            print("ไฟล์ที่ลาก:")
            urls.forEach { print("  - \($0.lastPathComponent)") }
        }
        
        dropView.onTextDropped = { text in
            print("ข้อความที่ลาก: \(text)")
        }
        
        dropView.onImageDropped = { image in
            print("รูปภาพขนาด: \(image.size)")
        }
    }
}
```

### Drag Source

```swift
import Cocoa

class DraggableView: NSView {
    
    var dragContent: String = "Hello, World!"
    var dragImage: NSImage?
    
    override func mouseDown(with event: NSEvent) {
        // เริ่มต้น Drag Operation
        let pasteboard = NSPasteboard()
        pasteboard.clearContents()
        
        // เพิ่มข้อมูลลง Pasteboard
        pasteboard.setString(dragContent, forType: .string)
        
        // สร้าง Drag Image
        let image = dragImage ?? createDragImage()
        
        let draggingItem = NSDraggingItem(pasteboardWriter: dragContent as NSString)
        draggingItem.setDraggingFrame(bounds, contents: image)
        
        beginDraggingSession(with: [draggingItem], event: event, source: self)
    }
    
    private func createDragImage() -> NSImage {
        let image = NSImage(size: NSSize(width: 100, height: 50))
        image.lockFocus()
        
        NSColor.blue.setFill()
        NSRect(x: 0, y: 0, width: 100, height: 50).fill()
        
        let attributes: [NSAttributedString.Key: Any] = [
            .foregroundColor: NSColor.white,
            .font: NSFont.systemFont(ofSize: 14)
        ]
        
        (dragContent as NSString).draw(
            in: NSRect(x: 10, y: 15, width: 80, height: 20),
            withAttributes: attributes
        )
        
        image.unlockFocus()
        return image
    }
}

extension DraggableView: NSDraggingSource {
    
    func draggingSession(_ session: NSDraggingSession, sourceOperationMaskFor context: NSDraggingContext) -> NSDragOperation {
        return .copy
    }
    
    func draggingSession(_ session: NSDraggingSession, endedAt screenPoint: NSPoint, operation: NSDragOperation) {
        print("Drag สิ้นสุดที่: \(screenPoint), operation: \(operation)")
    }
}
```

---

## 59.12 Copy and Paste

```swift
import Cocoa

class ClipboardManager {
    
    static let shared = ClipboardManager()
    private let pasteboard = NSPasteboard.general
    
    // คัดลอก String
    func copyString(_ string: String) {
        pasteboard.clearContents()
        pasteboard.setString(string, forType: .string)
    }
    
    // วาง String
    func pasteString() -> String? {
        return pasteboard.string(forType: .string)
    }
    
    // คัดลอก Image
    func copyImage(_ image: NSImage) {
        pasteboard.clearContents()
        pasteboard.writeObjects([image])
    }
    
    // วาง Image
    func pasteImage() -> NSImage? {
        guard let objects = pasteboard.readObjects(forClasses: [NSImage.self], options: nil) as? [NSImage] else {
            return nil
        }
        return objects.first
    }
    
    // คัดลอก URLs
    func copyURLs(_ urls: [URL]) {
        pasteboard.clearContents()
        pasteboard.writeObjects(urls as [NSURL])
    }
    
    // วาง URLs
    func pasteURLs() -> [URL]? {
        return pasteboard.readObjects(forClasses: [NSURL.self], options: nil) as? [URL]
    }
    
    // ตรวจสอบว่า Pasteboard มีข้อมูลประเภทใด
    func hasString() -> Bool {
        return pasteboard.availableType(from: [.string]) != nil
    }
    
    func hasImage() -> Bool {
        return pasteboard.canReadObject(forClasses: [NSImage.self], options: nil)
    }
    
    func hasFiles() -> Bool {
        return pasteboard.canReadObject(forClasses: [NSURL.self], options: nil)
    }
    
    // ล้าง Clipboard
    func clear() {
        pasteboard.clearContents()
    }
    
    // นับครั้งที่เปลี่ยน
    var changeCount: Int {
        return pasteboard.changeCount
    }
}

// ใช้งาน
class CopyPasteViewController: NSViewController {
    
    var lastChangeCount: Int = 0
    var clipboardTimer: Timer?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        startMonitoringClipboard()
    }
    
    override func viewWillDisappear() {
        super.viewWillDisappear()
        clipboardTimer?.invalidate()
    }
    
    private func startMonitoringClipboard() {
        lastChangeCount = ClipboardManager.shared.changeCount
        
        clipboardTimer = Timer.scheduledTimer(withTimeInterval: 0.5, repeats: true) { [weak self] _ in
            guard let self = self else { return }
            
            let currentCount = ClipboardManager.shared.changeCount
            if currentCount != self.lastChangeCount {
                self.lastChangeCount = currentCount
                self.clipboardChanged()
            }
        }
    }
    
    private func clipboardChanged() {
        if ClipboardManager.shared.hasString() {
            if let text = ClipboardManager.shared.pasteString() {
                print("Clipboard มีข้อความ: \(text.prefix(50))")
            }
        } else if ClipboardManager.shared.hasImage() {
            print("Clipboard มีรูปภาพ")
        } else if ClipboardManager.shared.hasFiles() {
            if let urls = ClipboardManager.shared.pasteURLs() {
                print("Clipboard มีไฟล์: \(urls.map { $0.lastPathComponent })")
            }
        }
    }
    
    // รองรับ Standard Edit Menu Commands
    @IBAction func copyAction(_ sender: Any) {
        // Custom Copy Logic
        ClipboardManager.shared.copyString("ข้อความที่คัดลอก")
    }
    
    @IBAction func pasteAction(_ sender: Any) {
        if let text = ClipboardManager.shared.pasteString() {
            print("วาง: \(text)")
        }
    }
    
    @IBAction func cutAction(_ sender: Any) {
        ClipboardManager.shared.copyString("ข้อความที่ตัด")
        // ลบข้อมูลต้นฉบับ
    }
}
```

---

## 59.13 Touch Bar (Legacy)

Touch Bar เป็น OLED strip บน MacBook Pro รุ่นเก่า (2016-2021)

```swift
import Cocoa

// Touch Bar ด้วย NSTouchBar
class TouchBarViewController: NSViewController {
    
    // Identifiers
    let touchBarIdentifier = NSTouchBar.CustomizationIdentifier("com.example.touchbar")
    let playButtonId = NSTouchBarItem.Identifier("playButton")
    let volumeSlider = NSTouchBarItem.Identifier("volumeSlider")
    let colorPickerItem = NSTouchBarItem.Identifier("colorPicker")
    
    override func makeTouchBar() -> NSTouchBar? {
        let touchBar = NSTouchBar()
        touchBar.delegate = self
        touchBar.customizationIdentifier = touchBarIdentifier
        touchBar.defaultItemIdentifiers = [playButtonId, .fixedSpaceLarge, volumeSlider]
        touchBar.customizationAllowedItemIdentifiers = [playButtonId, volumeSlider, colorPickerItem]
        return touchBar
    }
}

extension TouchBarViewController: NSTouchBarDelegate {
    
    func touchBar(_ touchBar: NSTouchBar, makeItemForIdentifier identifier: NSTouchBarItem.Identifier) -> NSTouchBarItem? {
        
        switch identifier {
        case playButtonId:
            let item = NSCustomTouchBarItem(identifier: identifier)
            let button = NSButton(title: "▶", target: self, action: #selector(playPause))
            button.bezelColor = .systemGreen
            item.view = button
            item.customizationLabel = "เล่น/หยุด"
            return item
            
        case volumeSlider:
            let item = NSSliderTouchBarItem(identifier: identifier)
            item.label = "ระดับเสียง"
            item.slider.minValue = 0
            item.slider.maxValue = 100
            item.slider.doubleValue = 50
            item.slider.target = self
            item.slider.action = #selector(volumeChanged)
            item.customizationLabel = "ปรับระดับเสียง"
            return item
            
        case colorPickerItem:
            let item = NSColorPickerTouchBarItem(identifier: identifier)
            item.customizationLabel = "เลือกสี"
            return item
            
        default:
            return nil
        }
    }
    
    @objc func playPause() {
        print("เล่น/หยุด")
    }
    
    @objc func volumeChanged(_ sender: NSSlider) {
        print("ระดับเสียง: \(Int(sender.doubleValue))%")
    }
}
```

---

## 59.14 Menu Items และ Keyboard Shortcuts

### สร้าง Custom Menu

```swift
import Cocoa

class MenuBuilder: NSObject {
    
    func buildApplicationMenu() {
        let mainMenu = NSMenu()
        NSApp.mainMenu = mainMenu
        
        // Application Menu
        let appMenuItem = NSMenuItem()
        mainMenu.addItem(appMenuItem)
        let appMenu = NSMenu()
        appMenuItem.submenu = appMenu
        appMenu.addItem(NSMenuItem(title: "เกี่ยวกับ MyApp", action: #selector(NSApplication.orderFrontStandardAboutPanel(_:)), keyEquivalent: ""))
        appMenu.addItem(NSMenuItem.separator())
        appMenu.addItem(NSMenuItem(title: "ตั้งค่า...", action: #selector(openSettings), keyEquivalent: ","))
        appMenu.addItem(NSMenuItem.separator())
        appMenu.addItem(NSMenuItem(title: "ออกจาก MyApp", action: #selector(NSApplication.terminate(_:)), keyEquivalent: "q"))
        
        // File Menu
        let fileMenuItem = NSMenuItem()
        mainMenu.addItem(fileMenuItem)
        let fileMenu = NSMenu(title: "ไฟล์")
        fileMenuItem.submenu = fileMenu
        
        let newItem = NSMenuItem(title: "ใหม่", action: #selector(newDocument), keyEquivalent: "n")
        fileMenu.addItem(newItem)
        
        let openItem = NSMenuItem(title: "เปิด...", action: #selector(openDocument), keyEquivalent: "o")
        fileMenu.addItem(openItem)
        
        fileMenu.addItem(NSMenuItem.separator())
        
        let saveItem = NSMenuItem(title: "บันทึก", action: #selector(saveDocument), keyEquivalent: "s")
        fileMenu.addItem(saveItem)
        
        let saveAsItem = NSMenuItem(title: "บันทึกเป็น...", action: #selector(saveDocumentAs), keyEquivalent: "S")
        saveAsItem.keyEquivalentModifierMask = [.command, .shift]
        fileMenu.addItem(saveAsItem)
        
        // Edit Menu
        let editMenuItem = NSMenuItem()
        mainMenu.addItem(editMenuItem)
        let editMenu = NSMenu(title: "แก้ไข")
        editMenuItem.submenu = editMenu
        
        editMenu.addItem(NSMenuItem(title: "เลิกทำ", action: #selector(UndoManager.undo), keyEquivalent: "z"))
        editMenu.addItem(NSMenuItem(title: "ทำซ้ำ", action: #selector(UndoManager.redo), keyEquivalent: "Z"))
        editMenu.addItem(NSMenuItem.separator())
        editMenu.addItem(NSMenuItem(title: "ตัด", action: #selector(NSText.cut(_:)), keyEquivalent: "x"))
        editMenu.addItem(NSMenuItem(title: "คัดลอก", action: #selector(NSText.copy(_:)), keyEquivalent: "c"))
        editMenu.addItem(NSMenuItem(title: "วาง", action: #selector(NSText.paste(_:)), keyEquivalent: "v"))
        editMenu.addItem(NSMenuItem(title: "เลือกทั้งหมด", action: #selector(NSText.selectAll(_:)), keyEquivalent: "a"))
        
        // Custom Menu
        let customMenuItem = NSMenuItem()
        mainMenu.addItem(customMenuItem)
        let customMenu = NSMenu(title: "เครื่องมือ")
        customMenuItem.submenu = customMenu
        
        let analyzeItem = NSMenuItem(title: "วิเคราะห์", action: #selector(analyze), keyEquivalent: "r")
        analyzeItem.keyEquivalentModifierMask = [.command, .option]
        customMenu.addItem(analyzeItem)
    }
    
    @objc func openSettings() { print("เปิดการตั้งค่า") }
    @objc func newDocument() { print("เอกสารใหม่") }
    @objc func openDocument() { print("เปิดเอกสาร") }
    @objc func saveDocument() { print("บันทึกเอกสาร") }
    @objc func saveDocumentAs() { print("บันทึกเอกสารเป็น") }
    @objc func analyze() { print("วิเคราะห์") }
}
```

### Keyboard Shortcuts ใน SwiftUI

```swift
import SwiftUI

struct ShortcutView: View {
    @State private var message = ""
    
    var body: some View {
        VStack {
            Text(message.isEmpty ? "กด Shortcut" : message)
                .font(.headline)
        }
        // Command + N
        .keyboardShortcut("n", modifiers: .command)
        // Command + S
        .onReceive(NotificationCenter.default.publisher(for: NSNotification.Name("save"))) { _ in
            message = "บันทึกแล้ว"
        }
        // Commands ใน Menu
        .commands {
            CommandGroup(replacing: .newItem) {
                Button("ไฟล์ใหม่") {
                    message = "สร้างไฟล์ใหม่"
                }
                .keyboardShortcut("n", modifiers: .command)
            }
            
            CommandMenu("เครื่องมือ") {
                Button("ประมวลผล") {
                    message = "กำลังประมวลผล..."
                }
                .keyboardShortcut("p", modifiers: [.command, .option])
                
                Divider()
                
                Button("รีเฟรช") {
                    message = "รีเฟรชแล้ว"
                }
                .keyboardShortcut("r", modifiers: .command)
            }
        }
    }
}
```

---

## 59.15 App Sandbox บน macOS

App Sandbox จำกัดสิทธิ์การเข้าถึงระบบของแอป เพื่อความปลอดภัย

### กำหนด Entitlements

```xml
<!-- MyApp.entitlements -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <!-- เปิด App Sandbox -->
    <key>com.apple.security.app-sandbox</key>
    <true/>
    
    <!-- เข้าถึง Network -->
    <key>com.apple.security.network.client</key>
    <true/>
    
    <!-- รับ Network Connections -->
    <key>com.apple.security.network.server</key>
    <true/>
    
    <!-- เข้าถึง Camera -->
    <key>com.apple.security.device.camera</key>
    <true/>
    
    <!-- เข้าถึง Microphone -->
    <key>com.apple.security.device.audio-input</key>
    <true/>
    
    <!-- เข้าถึง Location -->
    <key>com.apple.security.personal-information.location</key>
    <true/>
    
    <!-- อ่านไฟล์ที่ผู้ใช้เลือก -->
    <key>com.apple.security.files.user-selected.read-only</key>
    <true/>
    
    <!-- อ่านและเขียนไฟล์ที่ผู้ใช้เลือก -->
    <key>com.apple.security.files.user-selected.read-write</key>
    <true/>
    
    <!-- เข้าถึง Downloads folder -->
    <key>com.apple.security.files.downloads.read-write</key>
    <true/>
    
    <!-- Print -->
    <key>com.apple.security.print</key>
    <true/>
</dict>
</plist>
```

### ตรวจสอบ Sandbox Status

```swift
import Foundation

struct SandboxChecker {
    
    static var isSandboxed: Bool {
        let environment = ProcessInfo.processInfo.environment
        return environment["APP_SANDBOX_CONTAINER_ID"] != nil
    }
    
    static var containerPath: String? {
        if isSandboxed {
            return FileManager.default.homeDirectoryForCurrentUser.path
        }
        return nil
    }
    
    static func checkPermission(for resource: String) {
        let access = access(resource, F_OK)
        print("\(resource): \(access == 0 ? "เข้าถึงได้" : "ไม่สามารถเข้าถึง")")
    }
}
```

---

## 59.16 Hardened Runtime

Hardened Runtime เพิ่มความปลอดภัยให้แอปและจำเป็นสำหรับ Notarization

```xml
<!-- Entitlements สำหรับ Hardened Runtime -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <!-- อนุญาต JIT Compilation (สำหรับ JavaScript engine) -->
    <key>com.apple.security.cs.allow-jit</key>
    <true/>
    
    <!-- อนุญาต Unsigned Executable Memory -->
    <key>com.apple.security.cs.allow-unsigned-executable-memory</key>
    <true/>
    
    <!-- Disable Library Validation (สำหรับ plugins) -->
    <key>com.apple.security.cs.disable-library-validation</key>
    <true/>
    
    <!-- อนุญาต DYLD Environment Variables -->
    <key>com.apple.security.cs.allow-dyld-environment-variables</key>
    <true/>
    
    <!-- Debugging -->
    <key>com.apple.security.get-task-allow</key>
    <true/>
</dict>
</plist>
```

---

## 59.17 Notarization

Notarization คือกระบวนการส่งแอปให้ Apple ตรวจสอบก่อนแจกจ่าย

### ขั้นตอน Notarization

```bash
# 1. Archive แอป
xcodebuild archive \
    -project MyApp.xcodeproj \
    -scheme MyApp \
    -archivePath MyApp.xcarchive

# 2. Export สำหรับ Developer ID
xcodebuild -exportArchive \
    -archivePath MyApp.xcarchive \
    -exportOptionsPlist ExportOptions.plist \
    -exportPath ./export

# ExportOptions.plist
# <?xml version="1.0" encoding="UTF-8"?>
# <plist version="1.0">
# <dict>
#     <key>method</key>
#     <string>developer-id</string>
#     <key>signingStyle</key>
#     <string>automatic</string>
# </dict>
# </plist>

# 3. Submit สำหรับ Notarization
xcrun notarytool submit MyApp.zip \
    --apple-id "developer@example.com" \
    --password "app-specific-password" \
    --team-id "XXXXXXXXXX" \
    --wait

# 4. Staple Notarization
xcrun stapler staple MyApp.app

# 5. ตรวจสอบ
xcrun stapler validate MyApp.app
spctl -a -v MyApp.app
```

---

## 59.18 macOS Sequoia Features

macOS Sequoia (macOS 15) มีฟีเจอร์ใหม่หลายอย่าง

### iPhone Mirroring API

```swift
import AppKit

// ตรวจสอบ iPhone Mirroring Support
@available(macOS 15.0, *)
class iPhoneMirroringManager {
    
    func checkSupport() {
        // ตรวจสอบว่า iPhone เชื่อมต่ออยู่หรือไม่
        print("ตรวจสอบ iPhone Mirroring support")
    }
}
```

### Window Tiling

```swift
import SwiftUI

@available(macOS 15.0, *)
struct TilingContentView: View {
    var body: some View {
        ContentView()
            // รองรับ Window Tiling
            .windowResizability(.contentSize)
    }
}
```

### Improved Settings

```swift
import SwiftUI

@available(macOS 15.0, *)
struct SettingsContentView: View {
    var body: some View {
        Form {
            Section("ทั่วไป") {
                Toggle("เปิดใช้งานอัตโนมัติ", isOn: .constant(true))
                Toggle("แจ้งเตือน", isOn: .constant(false))
            }
            
            Section("ความเป็นส่วนตัว") {
                Toggle("ส่งข้อมูลวิเคราะห์", isOn: .constant(false))
            }
        }
        .formStyle(.grouped)
    }
}
```

---

## 59.19 Sharing Files ระหว่าง Mac และ iOS

### App Groups

```swift
import Foundation

// กำหนด App Group ใน Entitlements
// com.apple.security.application-groups: ["group.com.yourcompany.app"]

class AppGroupStorage {
    
    static let groupIdentifier = "group.com.yourcompany.app"
    
    static var sharedContainer: URL? {
        return FileManager.default.containerURL(
            forSecurityApplicationGroupIdentifier: groupIdentifier
        )
    }
    
    // เขียนข้อมูลลง Shared Container
    static func write(_ data: Data, to filename: String) -> Bool {
        guard let containerURL = sharedContainer else { return false }
        let fileURL = containerURL.appendingPathComponent(filename)
        
        do {
            try data.write(to: fileURL)
            return true
        } catch {
            print("เขียนไฟล์ล้มเหลว: \(error)")
            return false
        }
    }
    
    // อ่านข้อมูลจาก Shared Container
    static func read(from filename: String) -> Data? {
        guard let containerURL = sharedContainer else { return nil }
        let fileURL = containerURL.appendingPathComponent(filename)
        return try? Data(contentsOf: fileURL)
    }
    
    // แชร์ UserDefaults
    static var sharedDefaults: UserDefaults? {
        return UserDefaults(suiteName: groupIdentifier)
    }
}

// ใช้งาน
class DataSharingManager {
    
    func saveSharedData() {
        // บันทึกข้อมูลที่แชร์กับ iOS
        if let data = "ข้อมูลที่แชร์".data(using: .utf8) {
            _ = AppGroupStorage.write(data, to: "shared_data.txt")
        }
        
        // บันทึกใน UserDefaults
        AppGroupStorage.sharedDefaults?.set("สวัสดี iOS!", forKey: "greeting")
        AppGroupStorage.sharedDefaults?.synchronize()
    }
    
    func loadSharedData() -> String? {
        // อ่านข้อมูล
        if let data = AppGroupStorage.read(from: "shared_data.txt") {
            return String(data: data, encoding: .utf8)
        }
        return nil
    }
}
```

---

## 59.20 Handoff และ Continuity

Handoff ช่วยให้ผู้ใช้สลับงานระหว่างอุปกรณ์ Apple ได้อย่างราบรื่น

### NSUserActivity สำหรับ Handoff

```swift
import Foundation
import AppKit

class HandoffManager: NSObject {
    
    let activityType = "com.yourcompany.app.editing"
    var currentActivity: NSUserActivity?
    
    // เริ่มต้น Activity
    func startActivity(with documentURL: URL, content: String) {
        let activity = NSUserActivity(activityType: activityType)
        activity.title = "แก้ไขเอกสาร"
        activity.isEligibleForHandoff = true
        activity.isEligibleForSearch = true
        
        // ข้อมูลที่จะส่งไปยังอุปกรณ์อื่น
        activity.userInfo = [
            "documentURL": documentURL.absoluteString,
            "scrollPosition": 0,
            "cursorPosition": content.count
        ]
        
        activity.requiredUserInfoKeys = Set(["documentURL"])
        activity.becomeCurrent()
        
        currentActivity = activity
    }
    
    // อัพเดท Activity
    func updateActivity(scrollPosition: Int, cursorPosition: Int) {
        currentActivity?.addUserInfoEntries(from: [
            "scrollPosition": scrollPosition,
            "cursorPosition": cursorPosition
        ])
    }
    
    // หยุด Activity
    func stopActivity() {
        currentActivity?.resignCurrent()
        currentActivity?.invalidate()
        currentActivity = nil
    }
}

// รับ Handoff Activity
extension AppDelegate {
    
    func application(_ application: NSApplication, continue userActivity: NSUserActivity, restorationHandler: @escaping ([NSUserActivityRestoring]) -> Void) -> Bool {
        
        guard userActivity.activityType == "com.yourcompany.app.editing",
              let urlString = userActivity.userInfo?["documentURL"] as? String,
              let url = URL(string: urlString)
        else {
            return false
        }
        
        // เปิดเอกสารที่ค้างอยู่
        let scrollPosition = userActivity.userInfo?["scrollPosition"] as? Int ?? 0
        let cursorPosition = userActivity.userInfo?["cursorPosition"] as? Int ?? 0
        
        print("รับ Handoff: เปิด \(url.lastPathComponent) ที่ position \(cursorPosition)")
        
        // เปิดเอกสาร
        NSDocumentController.shared.openDocument(withContentsOf: url, display: true) { document, _, _ in
            if let doc = document {
                print("เปิดเอกสาร: \(doc.displayName ?? "ไม่ทราบชื่อ")")
            }
        }
        
        return true
    }
}
```

---

## 59.21 Practical Exercises

### แบบฝึกหัดที่ 1: สร้าง Simple Text Editor

```swift
import SwiftUI
import UniformTypeIdentifiers

// Text Document
struct SimpleTextFile: FileDocument {
    var text: String
    var fileName: String
    
    static var readableContentTypes: [UTType] { [.plainText] }
    
    init(text: String = "", fileName: String = "เอกสารใหม่") {
        self.text = text
        self.fileName = fileName
    }
    
    init(configuration: ReadConfiguration) throws {
        guard let data = configuration.file.regularFileContents,
              let string = String(data: data, encoding: .utf8)
        else {
            throw CocoaError(.fileReadCorruptFile)
        }
        text = string
        fileName = configuration.file.filename ?? "เอกสาร"
    }
    
    func fileWrapper(configuration: WriteConfiguration) throws -> FileWrapper {
        let data = text.data(using: .utf8)!
        return .init(regularFileWithContents: data)
    }
}

// Statistics View
struct StatisticsView: View {
    let text: String
    
    var wordCount: Int {
        text.components(separatedBy: .whitespacesAndNewlines)
            .filter { !$0.isEmpty }.count
    }
    
    var lineCount: Int {
        text.components(separatedBy: .newlines).count
    }
    
    var body: some View {
        HStack(spacing: 20) {
            Label("\(text.count) ตัวอักษร", systemImage: "character")
            Label("\(wordCount) คำ", systemImage: "text.word.spacing")
            Label("\(lineCount) บรรทัด", systemImage: "list.number")
        }
        .font(.caption)
        .foregroundColor(.secondary)
        .padding(.horizontal)
    }
}

// Editor View
struct TextEditorView: View {
    @Binding var document: SimpleTextFile
    @State private var showStats = false
    @State private var fontSize: Double = 14
    @FocusState private var isEditorFocused: Bool
    
    var body: some View {
        VStack(spacing: 0) {
            // Editor
            TextEditor(text: $document.text)
                .font(.system(size: fontSize, design: .monospaced))
                .focused($isEditorFocused)
                .onAppear { isEditorFocused = true }
            
            Divider()
            
            // Statistics Bar
            if showStats {
                StatisticsView(text: document.text)
                    .padding(.vertical, 4)
            }
        }
        .navigationTitle(document.fileName)
        .toolbar {
            ToolbarItemGroup(placement: .primaryAction) {
                // Font Size Control
                HStack {
                    Button(action: { fontSize = max(8, fontSize - 2) }) {
                        Image(systemName: "textformat.size.smaller")
                    }
                    Text("\(Int(fontSize))")
                        .frame(width: 30)
                    Button(action: { fontSize = min(72, fontSize + 2) }) {
                        Image(systemName: "textformat.size.larger")
                    }
                }
                
                // Stats Toggle
                Toggle(isOn: $showStats) {
                    Label("สถิติ", systemImage: "chart.bar")
                }
                .toggleStyle(.button)
            }
        }
    }
}

// App
@main
struct SimpleTextEditorApp: App {
    var body: some Scene {
        DocumentGroup(newDocument: SimpleTextFile()) { file in
            TextEditorView(document: file.$document)
        }
        
        Settings {
            Form {
                Section("การแสดงผล") {
                    Toggle("แสดงหมายเลขบรรทัด", isOn: .constant(false))
                    Toggle("Word Wrap", isOn: .constant(true))
                }
            }
            .formStyle(.grouped)
            .padding()
        }
    }
}
```

---

## 59.22 Building a Menu Bar App (Complete)

```swift
import SwiftUI
import AppKit

// MARK: - Data Model

struct Note: Identifiable, Codable {
    let id: UUID
    var title: String
    var content: String
    var createdAt: Date
    var color: String
    
    init(title: String = "", content: String = "", color: String = "yellow") {
        self.id = UUID()
        self.title = title
        self.content = content
        self.createdAt = Date()
        self.color = color
    }
}

// MARK: - Storage

class NotesStore: ObservableObject {
    @Published var notes: [Note] = []
    
    private let storageKey = "quick_notes"
    
    init() {
        loadNotes()
    }
    
    func addNote(_ note: Note = Note()) {
        notes.insert(note, at: 0)
        saveNotes()
    }
    
    func updateNote(_ note: Note) {
        if let index = notes.firstIndex(where: { $0.id == note.id }) {
            notes[index] = note
            saveNotes()
        }
    }
    
    func deleteNote(_ note: Note) {
        notes.removeAll { $0.id == note.id }
        saveNotes()
    }
    
    private func saveNotes() {
        if let encoded = try? JSONEncoder().encode(notes) {
            UserDefaults.standard.set(encoded, forKey: storageKey)
        }
    }
    
    private func loadNotes() {
        if let data = UserDefaults.standard.data(forKey: storageKey),
           let decoded = try? JSONDecoder().decode([Note].self, from: data) {
            notes = decoded
        }
    }
}

// MARK: - Note Row View

struct NoteRowView: View {
    let note: Note
    
    var noteColor: Color {
        switch note.color {
        case "red": return .red
        case "blue": return .blue
        case "green": return .green
        default: return .yellow
        }
    }
    
    var body: some View {
        HStack(spacing: 8) {
            // Color Indicator
            Circle()
                .fill(noteColor)
                .frame(width: 8, height: 8)
            
            VStack(alignment: .leading, spacing: 2) {
                Text(note.title.isEmpty ? "ไม่มีชื่อ" : note.title)
                    .font(.system(size: 13, weight: .medium))
                    .lineLimit(1)
                
                Text(note.content.isEmpty ? "ว่าง" : note.content)
                    .font(.system(size: 11))
                    .foregroundColor(.secondary)
                    .lineLimit(1)
            }
            
            Spacer()
            
            Text(note.createdAt, style: .relative)
                .font(.system(size: 10))
                .foregroundColor(.secondary)
        }
        .padding(.horizontal, 8)
        .padding(.vertical, 4)
    }
}

// MARK: - Quick Note Editor

struct QuickNoteEditor: View {
    @Binding var note: Note
    @FocusState private var isTitleFocused: Bool
    
    var body: some View {
        VStack(spacing: 0) {
            // Title
            TextField("หัวข้อ...", text: $note.title)
                .textFieldStyle(.plain)
                .font(.system(size: 14, weight: .semibold))
                .focused($isTitleFocused)
                .padding(.horizontal, 12)
                .padding(.top, 8)
                .padding(.bottom, 4)
            
            Divider()
                .padding(.horizontal, 12)
            
            // Content
            TextEditor(text: $note.content)
                .font(.system(size: 13))
                .frame(height: 120)
                .padding(.horizontal, 8)
        }
        .onAppear { isTitleFocused = true }
    }
}

// MARK: - Main Menu Bar View

struct MenuBarNotesView: View {
    @StateObject private var store = NotesStore()
    @State private var selectedNote: Note?
    @State private var isAddingNote = false
    @State private var newNote = Note()
    @State private var searchText = ""
    
    var filteredNotes: [Note] {
        if searchText.isEmpty { return store.notes }
        return store.notes.filter {
            $0.title.contains(searchText) || $0.content.contains(searchText)
        }
    }
    
    var body: some View {
        VStack(spacing: 0) {
            // Header
            HStack {
                Text("Quick Notes")
                    .font(.system(size: 13, weight: .semibold))
                
                Spacer()
                
                Button(action: { withAnimation { isAddingNote.toggle() } }) {
                    Image(systemName: isAddingNote ? "xmark.circle.fill" : "plus.circle.fill")
                        .foregroundColor(isAddingNote ? .secondary : .accentColor)
                }
                .buttonStyle(.plain)
            }
            .padding(.horizontal, 12)
            .padding(.vertical, 8)
            
            // Add Note Form
            if isAddingNote {
                Divider()
                
                QuickNoteEditor(note: $newNote)
                
                HStack {
                    // Color Picker
                    ForEach(["yellow", "red", "blue", "green"], id: \.self) { color in
                        Button(action: { newNote.color = color }) {
                            Circle()
                                .fill(colorFromString(color))
                                .frame(width: 16, height: 16)
                                .overlay(
                                    Circle()
                                        .stroke(Color.primary, lineWidth: newNote.color == color ? 2 : 0)
                                )
                        }
                        .buttonStyle(.plain)
                    }
                    
                    Spacer()
                    
                    Button("ยกเลิก") {
                        newNote = Note()
                        isAddingNote = false
                    }
                    .foregroundColor(.secondary)
                    
                    Button("บันทึก") {
                        store.addNote(newNote)
                        newNote = Note()
                        isAddingNote = false
                    }
                    .buttonStyle(.borderedProminent)
                    .disabled(newNote.title.isEmpty && newNote.content.isEmpty)
                }
                .padding(.horizontal, 12)
                .padding(.bottom, 8)
            }
            
            Divider()
            
            // Search
            HStack {
                Image(systemName: "magnifyingglass")
                    .foregroundColor(.secondary)
                    .font(.system(size: 12))
                
                TextField("ค้นหาโน้ต...", text: $searchText)
                    .textFieldStyle(.plain)
                    .font(.system(size: 12))
                
                if !searchText.isEmpty {
                    Button(action: { searchText = "" }) {
                        Image(systemName: "xmark.circle.fill")
                            .foregroundColor(.secondary)
                            .font(.system(size: 12))
                    }
                    .buttonStyle(.plain)
                }
            }
            .padding(.horizontal, 12)
            .padding(.vertical, 6)
            
            Divider()
            
            // Notes List
            if filteredNotes.isEmpty {
                VStack(spacing: 8) {
                    Image(systemName: "note.text")
                        .font(.system(size: 30))
                        .foregroundColor(.secondary)
                    
                    Text(searchText.isEmpty ? "ยังไม่มีโน้ต" : "ไม่พบโน้ต")
                        .foregroundColor(.secondary)
                        .font(.system(size: 12))
                }
                .frame(maxWidth: .infinity)
                .padding(.vertical, 30)
            } else {
                ScrollView {
                    LazyVStack(spacing: 0) {
                        ForEach(filteredNotes) { note in
                            NoteRowView(note: note)
                                .background(selectedNote?.id == note.id ? Color.accentColor.opacity(0.1) : Color.clear)
                                .contentShape(Rectangle())
                                .onTapGesture {
                                    selectedNote = note
                                }
                                .contextMenu {
                                    Button("ลบ") {
                                        store.deleteNote(note)
                                    }
                                }
                            
                            Divider()
                                .padding(.leading, 24)
                        }
                    }
                }
                .frame(maxHeight: 250)
            }
            
            Divider()
            
            // Footer
            HStack {
                Text("\(store.notes.count) โน้ต")
                    .font(.system(size: 11))
                    .foregroundColor(.secondary)
                
                Spacer()
                
                Button("ออก") {
                    NSApplication.shared.terminate(nil)
                }
                .font(.system(size: 11))
                .foregroundColor(.secondary)
                .buttonStyle(.plain)
            }
            .padding(.horizontal, 12)
            .padding(.vertical, 6)
        }
        .frame(width: 300)
    }
    
    func colorFromString(_ color: String) -> Color {
        switch color {
        case "red": return .red
        case "blue": return .blue
        case "green": return .green
        default: return .yellow
        }
    }
}

// MARK: - App Delegate

class MenuBarAppDelegate: NSObject, NSApplicationDelegate {
    var statusItem: NSStatusItem!
    var popover: NSPopover!
    
    func applicationDidFinishLaunching(_ notification: Notification) {
        NSApp.setActivationPolicy(.accessory)
        
        statusItem = NSStatusBar.system.statusItem(withLength: NSStatusItem.squareLength)
        
        if let button = statusItem.button {
            button.image = NSImage(systemSymbolName: "note.text", accessibilityDescription: "Quick Notes")
            button.action = #selector(togglePopover(_:))
            button.target = self
        }
        
        popover = NSPopover()
        popover.contentSize = NSSize(width: 300, height: 500)
        popover.behavior = .transient
        popover.contentViewController = NSHostingController(rootView: MenuBarNotesView())
    }
    
    @objc func togglePopover(_ sender: AnyObject?) {
        if let button = statusItem.button {
            if popover.isShown {
                popover.performClose(sender)
            } else {
                popover.show(relativeTo: button.bounds, of: button, preferredEdge: .minY)
                NSApp.activate(ignoringOtherApps: true)
            }
        }
    }
}

// MARK: - Main Entry Point

@main
struct QuickNotesApp: App {
    @NSApplicationDelegateAdaptor(MenuBarAppDelegate.self) var appDelegate
    
    var body: some Scene {
        // ไม่มี WindowGroup - แอปอยู่ใน Menu Bar เท่านั้น
        Settings {
            Form {
                Section("การแสดงผล") {
                    Toggle("เริ่มต้นพร้อม macOS", isOn: .constant(true))
                }
            }
            .formStyle(.grouped)
            .padding()
            .frame(width: 400)
        }
    }
}
```

---

## 59.23 สรุป

ในบทนี้เราได้เรียนรู้:

1. **macOS Development Overview** - ภาพรวมการพัฒนา macOS และความแตกต่างจาก iOS
2. **AppKit vs SwiftUI** - เปรียบเทียบ framework ทั้งสองและวิธีใช้งาน
3. **Mac Catalyst** - นำแอป iPad มาใช้บน macOS
4. **SwiftUI for macOS** - การใช้ SwiftUI บน macOS โดยเฉพาะ
5. **Menu Bar Apps** - สร้างแอปที่อยู่ใน Status Bar
6. **Window Management** - การจัดการหน้าต่างบน macOS
7. **NSTableView/NSOutlineView** - แสดงข้อมูลในรูปแบบตารางและ tree
8. **Toolbar และ Sidebar** - การใช้ Toolbar และ Sidebar
9. **Document-based Apps** - สร้างแอปจัดการเอกสาร
10. **File Handling** - จัดการไฟล์บน macOS
11. **Drag and Drop** - การลากและวางข้อมูล
12. **Copy and Paste** - การคัดลอกและวาง
13. **Touch Bar** - รองรับ Touch Bar (Legacy)
14. **Menu Items** - สร้าง Menu และ Keyboard Shortcuts
15. **App Sandbox** - ความปลอดภัยด้วย Sandbox
16. **Hardened Runtime** - เพิ่มความปลอดภัย
17. **Notarization** - กระบวนการ Notarize แอป
18. **macOS Sequoia** - ฟีเจอร์ใหม่ใน macOS 15
19. **App Groups** - แชร์ข้อมูลระหว่างแอป
20. **Handoff** - สลับงานระหว่างอุปกรณ์

---

## แบบฝึกหัดเพิ่มเติม

1. สร้าง Menu Bar App ที่แสดงสถานะ CPU และ Memory
2. สร้าง Document-based App สำหรับ Markdown Editor
3. สร้างแอปที่รองรับ Drag and Drop จาก Finder
4. สร้าง Settings Window ที่มีหลาย Tabs
5. ใช้ NSTableView แสดงข้อมูล JSON จาก API

---

*บทต่อไป: Part 60 - Widgets and WidgetKit*
