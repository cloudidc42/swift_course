# Part 33: UserDefaults และ File System ใน Swift

## บทนำ

การจัดเก็บข้อมูลถาวร (Persistent Storage) เป็นส่วนสำคัญของแอปพลิเคชันทุกประเภท ใน iOS และ macOS มีหลายวิธีในการจัดเก็บข้อมูล ตั้งแต่การบันทึกการตั้งค่าง่ายๆ ด้วย UserDefaults ไปจนถึงการจัดการไฟล์ด้วย FileManager และการเก็บข้อมูลที่ต้องการความปลอดภัยสูงใน Keychain

ในบทนี้เราจะเรียนรู้:
- **UserDefaults**: การบันทึกการตั้งค่าและข้อมูลขนาดเล็ก
- **FileManager**: การจัดการไฟล์และโฟลเดอร์
- **Property Lists**: การทำงานกับไฟล์ plist
- **JSON Storage**: การบันทึกข้อมูล JSON
- **Keychain**: การเก็บข้อมูลที่ต้องการความปลอดภัย

---

## 33.1 UserDefaults พื้นฐาน

`UserDefaults` เป็น interface สำหรับบันทึกข้อมูลการตั้งค่าของผู้ใช้ในรูปแบบ key-value pairs ข้อมูลที่บันทึกจะคงอยู่แม้จะปิดแอปและเปิดใหม่

### การทำงานของ UserDefaults

UserDefaults บันทึกข้อมูลในรูปแบบ Property List (plist) ลงในระบบไฟล์ของอุปกรณ์ ข้อมูลจะถูกแคชไว้ใน memory เพื่อให้การเข้าถึงทำได้รวดเร็ว

```swift
import Foundation

// การเข้าถึง UserDefaults
let defaults = UserDefaults.standard

// บันทึกข้อมูลประเภทต่างๆ
defaults.set(true, forKey: "isDarkMode")
defaults.set(42, forKey: "userAge")
defaults.set(3.14, forKey: "pi")
defaults.set("John Doe", forKey: "userName")
defaults.set(Date(), forKey: "lastLoginDate")
defaults.set([1, 2, 3, 4, 5], forKey: "favoriteNumbers")
defaults.set(["name": "John", "age": 30], forKey: "userInfo")

// อ่านข้อมูลกลับมา
let isDarkMode = defaults.bool(forKey: "isDarkMode")
let userAge = defaults.integer(forKey: "userAge")
let pi = defaults.double(forKey: "pi")
let userName = defaults.string(forKey: "userName") ?? "Unknown"
let lastLogin = defaults.object(forKey: "lastLoginDate") as? Date
let numbers = defaults.array(forKey: "favoriteNumbers") as? [Int]
let info = defaults.dictionary(forKey: "userInfo")

print("Dark Mode: \(isDarkMode)")
print("User Age: \(userAge)")
print("User Name: \(userName)")
```

### ประเภทข้อมูลที่รองรับ

UserDefaults รองรับประเภทข้อมูลพื้นฐานดังนี้:

```swift
import Foundation

let defaults = UserDefaults.standard

// Primitive types
defaults.set(true, forKey: "boolValue")          // Bool
defaults.set(42, forKey: "intValue")              // Int
defaults.set(3.14, forKey: "floatValue")          // Float
defaults.set(2.718281828, forKey: "doubleValue")  // Double
defaults.set("Hello", forKey: "stringValue")      // String

// Collections
defaults.set([1, 2, 3], forKey: "arrayValue")    // Array
defaults.set(["key": "value"], forKey: "dictValue") // Dictionary

// Objects
defaults.set(Data(), forKey: "dataValue")        // Data
defaults.set(Date(), forKey: "dateValue")        // Date
defaults.set(URL(string: "https://example.com")!, forKey: "urlValue") // URL

// การอ่านข้อมูล
let boolVal = defaults.bool(forKey: "boolValue")
let intVal = defaults.integer(forKey: "intValue")
let floatVal = defaults.float(forKey: "floatValue")
let doubleVal = defaults.double(forKey: "doubleValue")
let stringVal = defaults.string(forKey: "stringValue")
let dataVal = defaults.data(forKey: "dataValue")
let dateVal = defaults.object(forKey: "dateValue") as? Date
let urlVal = defaults.url(forKey: "urlValue")
let arrayVal = defaults.array(forKey: "arrayValue")
let dictVal = defaults.dictionary(forKey: "dictValue")
```

---

## 33.2 การบันทึกและอ่านค่า

### การตรวจสอบว่ามีค่าอยู่หรือไม่

```swift
import Foundation

let defaults = UserDefaults.standard

// ตรวจสอบด้วย object(forKey:)
if defaults.object(forKey: "userName") != nil {
    let name = defaults.string(forKey: "userName")!
    print("Found user: \(name)")
} else {
    print("No user found")
}

// การใช้ค่า default
let theme = defaults.string(forKey: "theme") ?? "light"
let fontSize = defaults.integer(forKey: "fontSize") == 0 ? 14 : defaults.integer(forKey: "fontSize")

print("Theme: \(theme)")
print("Font Size: \(fontSize)")
```

### การลบข้อมูล

```swift
import Foundation

let defaults = UserDefaults.standard

// ลบค่าที่ระบุ
defaults.removeObject(forKey: "userName")

// ตรวจสอบหลังจากลบ
if defaults.object(forKey: "userName") == nil {
    print("userName deleted successfully")
}
```

### การใช้ Constants สำหรับ Keys

Best practice คือการใช้ constants เพื่อป้องกัน typos:

```swift
import Foundation

// วิธีที่ 1: struct ของ keys
struct UserDefaultsKeys {
    static let isDarkMode = "isDarkMode"
    static let userName = "userName"
    static let userAge = "userAge"
    static let lastLoginDate = "lastLoginDate"
    static let notificationsEnabled = "notificationsEnabled"
    static let language = "language"
}

let defaults = UserDefaults.standard

// ใช้งาน
defaults.set(true, forKey: UserDefaultsKeys.isDarkMode)
defaults.set("Alice", forKey: UserDefaultsKeys.userName)
defaults.set(25, forKey: UserDefaultsKeys.userAge)

let isDark = defaults.bool(forKey: UserDefaultsKeys.isDarkMode)
let name = defaults.string(forKey: UserDefaultsKeys.userName)

// วิธีที่ 2: extension บน String
extension String {
    static let isDarkMode = "isDarkMode"
    static let userName = "userName"
}

// วิธีที่ 3: enum
enum DefaultsKey: String {
    case isDarkMode
    case userName
    case userAge
    case theme
    case fontSize
}

extension UserDefaults {
    func value(for key: DefaultsKey) -> Any? {
        return object(forKey: key.rawValue)
    }
    
    func set(_ value: Any?, for key: DefaultsKey) {
        set(value, forKey: key.rawValue)
    }
    
    func bool(for key: DefaultsKey) -> Bool {
        return bool(forKey: key.rawValue)
    }
    
    func string(for key: DefaultsKey) -> String? {
        return string(forKey: key.rawValue)
    }
    
    func integer(for key: DefaultsKey) -> Int {
        return integer(forKey: key.rawValue)
    }
}

// การใช้งาน
UserDefaults.standard.set(true, for: .isDarkMode)
let dark = UserDefaults.standard.bool(for: .isDarkMode)
```

---

## 33.3 Custom Types กับ UserDefaults (Codable)

UserDefaults ไม่รองรับ custom types โดยตรง แต่เราสามารถใช้ Codable เพื่อแปลง custom types เป็น Data แล้วบันทึกได้

### การบันทึก Codable Types

```swift
import Foundation

// Custom type ที่ conform to Codable
struct UserProfile: Codable {
    var id: UUID
    var name: String
    var email: String
    var age: Int
    var isPremium: Bool
    var createdAt: Date
    
    init(name: String, email: String, age: Int) {
        self.id = UUID()
        self.name = name
        self.email = email
        self.age = age
        self.isPremium = false
        self.createdAt = Date()
    }
}

// Extension สำหรับการบันทึกและอ่าน Codable
extension UserDefaults {
    func setCodable<T: Encodable>(_ value: T, forKey key: String) {
        if let encoded = try? JSONEncoder().encode(value) {
            set(encoded, forKey: key)
        }
    }
    
    func codable<T: Decodable>(_ type: T.Type, forKey key: String) -> T? {
        guard let data = data(forKey: key) else { return nil }
        return try? JSONDecoder().decode(type, from: data)
    }
}

// การใช้งาน
let profile = UserProfile(name: "Alice", email: "alice@example.com", age: 28)
UserDefaults.standard.setCodable(profile, forKey: "userProfile")

if let savedProfile = UserDefaults.standard.codable(UserProfile.self, forKey: "userProfile") {
    print("Name: \(savedProfile.name)")
    print("Email: \(savedProfile.email)")
    print("Age: \(savedProfile.age)")
}
```

### การบันทึก Array ของ Codable Types

```swift
import Foundation

struct Task: Codable, Identifiable {
    var id: UUID
    var title: String
    var isCompleted: Bool
    var dueDate: Date?
    var priority: Priority
    
    enum Priority: String, Codable {
        case low = "low"
        case medium = "medium"
        case high = "high"
    }
    
    init(title: String, priority: Priority = .medium, dueDate: Date? = nil) {
        self.id = UUID()
        self.title = title
        self.isCompleted = false
        self.dueDate = dueDate
        self.priority = priority
    }
}

class TaskStore {
    static let shared = TaskStore()
    private let key = "savedTasks"
    
    private init() {}
    
    var tasks: [Task] {
        get {
            guard let data = UserDefaults.standard.data(forKey: key),
                  let decoded = try? JSONDecoder().decode([Task].self, from: data) else {
                return []
            }
            return decoded
        }
        set {
            if let encoded = try? JSONEncoder().encode(newValue) {
                UserDefaults.standard.set(encoded, forKey: key)
            }
        }
    }
    
    func addTask(_ task: Task) {
        var current = tasks
        current.append(task)
        tasks = current
    }
    
    func removeTask(at index: Int) {
        var current = tasks
        current.remove(at: index)
        tasks = current
    }
    
    func toggleTask(id: UUID) {
        var current = tasks
        if let index = current.firstIndex(where: { $0.id == id }) {
            current[index].isCompleted.toggle()
        }
        tasks = current
    }
    
    func clearAll() {
        tasks = []
    }
}

// การใช้งาน
let store = TaskStore.shared

store.addTask(Task(title: "Buy groceries", priority: .high))
store.addTask(Task(title: "Read a book", priority: .low))
store.addTask(Task(title: "Exercise", priority: .medium))

print("Tasks: \(store.tasks.count)")
for task in store.tasks {
    print("- \(task.title) [\(task.priority.rawValue)]")
}
```

---

## 33.4 @AppStorage กับ SwiftUI

`@AppStorage` เป็น property wrapper ใน SwiftUI ที่ใช้งาน UserDefaults ในรูปแบบที่สะดวกกว่า และรองรับการ update UI อัตโนมัติเมื่อค่าเปลี่ยน

### การใช้ @AppStorage เบื้องต้น

```swift
import SwiftUI

struct SettingsView: View {
    // @AppStorage จะอ่านและเขียนค่าใน UserDefaults อัตโนมัติ
    @AppStorage("isDarkMode") private var isDarkMode = false
    @AppStorage("userName") private var userName = ""
    @AppStorage("fontSize") private var fontSize = 14.0
    @AppStorage("language") private var language = "th"
    @AppStorage("notificationsEnabled") private var notificationsEnabled = true
    
    var body: some View {
        NavigationView {
            Form {
                Section("ธีม") {
                    Toggle("โหมดมืด", isOn: $isDarkMode)
                }
                
                Section("โปรไฟล์") {
                    TextField("ชื่อผู้ใช้", text: $userName)
                }
                
                Section("การแสดงผล") {
                    HStack {
                        Text("ขนาดตัวอักษร")
                        Spacer()
                        Text("\(Int(fontSize))")
                    }
                    Slider(value: $fontSize, in: 10...24, step: 1)
                }
                
                Section("การแจ้งเตือน") {
                    Toggle("เปิดการแจ้งเตือน", isOn: $notificationsEnabled)
                }
                
                Section("ภาษา") {
                    Picker("ภาษา", selection: $language) {
                        Text("ไทย").tag("th")
                        Text("English").tag("en")
                        Text("日本語").tag("ja")
                    }
                }
            }
            .navigationTitle("การตั้งค่า")
            .preferredColorScheme(isDarkMode ? .dark : .light)
        }
    }
}
```

### @AppStorage กับ Enum

```swift
import SwiftUI

enum AppTheme: String, CaseIterable {
    case light = "light"
    case dark = "dark"
    case system = "system"
    
    var displayName: String {
        switch self {
        case .light: return "สว่าง"
        case .dark: return "มืด"
        case .system: return "ตามระบบ"
        }
    }
    
    var colorScheme: ColorScheme? {
        switch self {
        case .light: return .light
        case .dark: return .dark
        case .system: return nil
        }
    }
}

struct ThemeSettingsView: View {
    @AppStorage("appTheme") private var themeRawValue = AppTheme.system.rawValue
    
    private var selectedTheme: AppTheme {
        AppTheme(rawValue: themeRawValue) ?? .system
    }
    
    var body: some View {
        Picker("ธีม", selection: Binding(
            get: { selectedTheme },
            set: { themeRawValue = $0.rawValue }
        )) {
            ForEach(AppTheme.allCases, id: \.rawValue) { theme in
                Text(theme.displayName).tag(theme)
            }
        }
        .pickerStyle(.segmented)
    }
}
```

### @AppStorage กับ Custom Codable Types

```swift
import SwiftUI

struct AppSettings: Codable {
    var accentColor: String = "blue"
    var showAnimations: Bool = true
    var autoSave: Bool = true
    var backupEnabled: Bool = false
    var maxHistoryItems: Int = 100
}

// Custom property wrapper สำหรับ Codable types
@propertyWrapper
struct CodableAppStorage<T: Codable>: DynamicProperty {
    private let key: String
    private let defaultValue: T
    @State private var _value: T
    
    init(wrappedValue: T, _ key: String) {
        self.key = key
        self.defaultValue = wrappedValue
        
        if let data = UserDefaults.standard.data(forKey: key),
           let decoded = try? JSONDecoder().decode(T.self, from: data) {
            self._value = State(initialValue: decoded)
        } else {
            self._value = State(initialValue: wrappedValue)
        }
    }
    
    var wrappedValue: T {
        get { _value }
        nonmutating set {
            _value = newValue
            if let encoded = try? JSONEncoder().encode(newValue) {
                UserDefaults.standard.set(encoded, forKey: key)
            }
        }
    }
    
    var projectedValue: Binding<T> {
        Binding(
            get: { wrappedValue },
            set: { wrappedValue = $0 }
        )
    }
}

struct AppSettingsView: View {
    @CodableAppStorage(wrappedValue: AppSettings(), "appSettings")
    var settings: AppSettings
    
    var body: some View {
        Form {
            Toggle("แสดง Animation", isOn: $settings.showAnimations)
            Toggle("บันทึกอัตโนมัติ", isOn: $settings.autoSave)
            Toggle("สำรองข้อมูล", isOn: $settings.backupEnabled)
            
            Stepper("ประวัติสูงสุด: \(settings.maxHistoryItems)",
                    value: $settings.maxHistoryItems,
                    in: 10...500, step: 10)
        }
    }
}
```

---

## 33.5 UserDefaults Suites

UserDefaults Suites ช่วยให้แอปในกลุ่ม App Group สามารถแชร์ข้อมูลกันได้

```swift
import Foundation

// สร้าง UserDefaults suite ที่กำหนดเอง
let suiteName = "com.example.myapp.shared"
let sharedDefaults = UserDefaults(suiteName: suiteName)

// บันทึกข้อมูลใน suite
sharedDefaults?.set("shared value", forKey: "sharedKey")
sharedDefaults?.set(42, forKey: "sharedInt")

// อ่านข้อมูลจาก suite
let sharedValue = sharedDefaults?.string(forKey: "sharedKey")
print("Shared: \(sharedValue ?? "not found")")

// App Group UserDefaults (สำหรับ sharing ระหว่าง app และ extension)
let appGroupID = "group.com.example.myapp"
if let groupDefaults = UserDefaults(suiteName: appGroupID) {
    groupDefaults.set("data from main app", forKey: "sharedData")
    groupDefaults.synchronize()
    
    // Widget หรือ Extension สามารถอ่านข้อมูลนี้ได้
    let data = groupDefaults.string(forKey: "sharedData")
    print("App Group data: \(data ?? "nil")")
}
```

### การใช้ Suite กับ Widget

```swift
import WidgetKit
import SwiftUI

// Shared UserDefaults สำหรับ Widget และ App
class SharedStore {
    static let shared = SharedStore()
    
    private let defaults: UserDefaults?
    private let appGroupID = "group.com.example.myapp"
    
    private init() {
        defaults = UserDefaults(suiteName: appGroupID)
    }
    
    var lastUpdated: Date {
        get { defaults?.object(forKey: "lastUpdated") as? Date ?? Date() }
        set {
            defaults?.set(newValue, forKey: "lastUpdated")
            // Reload widget
            WidgetCenter.shared.reloadAllTimelines()
        }
    }
    
    var widgetData: [String: Any]? {
        get { defaults?.dictionary(forKey: "widgetData") }
        set {
            defaults?.set(newValue, forKey: "widgetData")
            lastUpdated = Date()
        }
    }
}
```

---

## 33.6 NSUbiquitousKeyValueStore (iCloud)

`NSUbiquitousKeyValueStore` ช่วยให้ sync ข้อมูลขนาดเล็กผ่าน iCloud ได้

```swift
import Foundation

class iCloudStore {
    static let shared = iCloudStore()
    
    private let store = NSUbiquitousKeyValueStore.default
    
    private init() {
        // ฟัง notification เมื่อ iCloud sync
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(ubiquitousKeyValueStoreDidChange(_:)),
            name: NSUbiquitousKeyValueStore.didChangeExternallyNotification,
            object: store
        )
        
        // Sync เมื่อเริ่มต้น
        store.synchronize()
    }
    
    deinit {
        NotificationCenter.default.removeObserver(self)
    }
    
    @objc private func ubiquitousKeyValueStoreDidChange(_ notification: Notification) {
        guard let userInfo = notification.userInfo,
              let reason = userInfo[NSUbiquitousKeyValueStoreChangeReasonKey] as? Int else {
            return
        }
        
        switch reason {
        case NSUbiquitousKeyValueStoreServerChange:
            print("Server changed iCloud values")
        case NSUbiquitousKeyValueStoreInitialSyncChange:
            print("Initial sync from iCloud")
        case NSUbiquitousKeyValueStoreQuotaViolationChange:
            print("iCloud quota exceeded!")
        case NSUbiquitousKeyValueStoreAccountChange:
            print("iCloud account changed")
        default:
            break
        }
        
        // Keys ที่เปลี่ยนแปลง
        if let changedKeys = userInfo[NSUbiquitousKeyValueStoreChangedKeysKey] as? [String] {
            print("Changed keys: \(changedKeys)")
            // Update local state
            NotificationCenter.default.post(
                name: .iCloudDataChanged,
                object: nil,
                userInfo: ["keys": changedKeys]
            )
        }
    }
    
    // MARK: - Getters/Setters
    
    var userName: String {
        get { store.string(forKey: "userName") ?? "" }
        set {
            store.set(newValue, forKey: "userName")
            store.synchronize()
        }
    }
    
    var isPremium: Bool {
        get { store.bool(forKey: "isPremium") }
        set {
            store.set(newValue, forKey: "isPremium")
            store.synchronize()
        }
    }
    
    var preferences: [String: Any]? {
        get { store.dictionary(forKey: "preferences") }
        set {
            store.set(newValue, forKey: "preferences")
            store.synchronize()
        }
    }
    
    func set<T: Encodable>(_ value: T, forKey key: String) {
        if let data = try? JSONEncoder().encode(value) {
            store.set(data, forKey: key)
            store.synchronize()
        }
    }
    
    func get<T: Decodable>(_ type: T.Type, forKey key: String) -> T? {
        guard let data = store.data(forKey: key) else { return nil }
        return try? JSONDecoder().decode(type, from: data)
    }
}

extension Notification.Name {
    static let iCloudDataChanged = Notification.Name("iCloudDataChanged")
}
```

---

## 33.7 FileManager พื้นฐาน

`FileManager` เป็น class หลักสำหรับการจัดการไฟล์และโฟลเดอร์ใน iOS และ macOS

```swift
import Foundation

// การเข้าถึง FileManager
let fileManager = FileManager.default

// ข้อมูลพื้นฐานของ FileManager
print("Current directory: \(fileManager.currentDirectoryPath)")
print("Temporary directory: \(fileManager.temporaryDirectory.path)")
```

### การตรวจสอบไฟล์และโฟลเดอร์

```swift
import Foundation

let fileManager = FileManager.default

// ตรวจสอบว่าไฟล์/โฟลเดอร์มีอยู่หรือไม่
let path = "/Users/user/Desktop/test.txt"

if fileManager.fileExists(atPath: path) {
    print("File exists")
} else {
    print("File does not exist")
}

// ตรวจสอบว่าเป็นโฟลเดอร์หรือไม่
var isDirectory: ObjCBool = false
if fileManager.fileExists(atPath: path, isDirectory: &isDirectory) {
    if isDirectory.boolValue {
        print("It's a directory")
    } else {
        print("It's a file")
    }
}

// ตรวจสอบ permissions
let isReadable = fileManager.isReadableFile(atPath: path)
let isWritable = fileManager.isWritableFile(atPath: path)
let isExecutable = fileManager.isExecutableFile(atPath: path)
let isDeletable = fileManager.isDeletableFile(atPath: path)

print("Readable: \(isReadable)")
print("Writable: \(isWritable)")
```

---

## 33.8 Documents Directory

Documents Directory เป็นโฟลเดอร์หลักที่แอปใช้บันทึกข้อมูลผู้ใช้ ข้อมูลในนี้จะถูก backup โดย iCloud

```swift
import Foundation

// วิธีหา Documents Directory
func getDocumentsDirectory() -> URL {
    let paths = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)
    return paths[0]
}

let documentsURL = getDocumentsDirectory()
print("Documents: \(documentsURL.path)")

// สร้าง URL สำหรับไฟล์ใน Documents
let fileURL = documentsURL.appendingPathComponent("myFile.txt")
print("File URL: \(fileURL.path)")

// บันทึกข้อความลงไฟล์
let content = "Hello, Swift!"
do {
    try content.write(to: fileURL, atomically: true, encoding: .utf8)
    print("File saved successfully")
} catch {
    print("Error saving file: \(error)")
}

// อ่านข้อความจากไฟล์
do {
    let loaded = try String(contentsOf: fileURL, encoding: .utf8)
    print("Loaded: \(loaded)")
} catch {
    print("Error loading file: \(error)")
}
```

### การจัดการโฟลเดอร์ใน Documents

```swift
import Foundation

class DocumentsManager {
    static let shared = DocumentsManager()
    
    private let fileManager = FileManager.default
    
    var documentsURL: URL {
        fileManager.urls(for: .documentDirectory, in: .userDomainMask)[0]
    }
    
    private init() {}
    
    // สร้างโฟลเดอร์ย่อย
    func createSubdirectory(named name: String) -> URL? {
        let url = documentsURL.appendingPathComponent(name, isDirectory: true)
        
        do {
            try fileManager.createDirectory(at: url, withIntermediateDirectories: true, attributes: nil)
            return url
        } catch {
            print("Error creating directory: \(error)")
            return nil
        }
    }
    
    // รายการไฟล์ใน Documents
    func listFiles(in subdirectory: String? = nil) -> [URL] {
        let url = subdirectory != nil ?
            documentsURL.appendingPathComponent(subdirectory!) :
            documentsURL
        
        do {
            let contents = try fileManager.contentsOfDirectory(
                at: url,
                includingPropertiesForKeys: [.fileSizeKey, .creationDateKey, .isDirectoryKey],
                options: [.skipsHiddenFiles]
            )
            return contents
        } catch {
            print("Error listing files: \(error)")
            return []
        }
    }
    
    // ขนาดของโฟลเดอร์
    func sizeOfDirectory(at url: URL) -> Int64 {
        guard let enumerator = fileManager.enumerator(at: url, includingPropertiesForKeys: [.fileSizeKey]) else {
            return 0
        }
        
        var totalSize: Int64 = 0
        for case let fileURL as URL in enumerator {
            if let fileSize = try? fileURL.resourceValues(forKeys: [.fileSizeKey]).fileSize {
                totalSize += Int64(fileSize)
            }
        }
        return totalSize
    }
}

// การใช้งาน
let manager = DocumentsManager.shared

if let imagesDir = manager.createSubdirectory(named: "Images") {
    print("Images directory: \(imagesDir.path)")
}

let files = manager.listFiles()
for file in files {
    print("- \(file.lastPathComponent)")
}
```

---

## 33.9 Caches Directory

Caches Directory ใช้เก็บข้อมูลชั่วคราวที่สามารถสร้างใหม่ได้ ระบบอาจลบข้อมูลในนี้เมื่อพื้นที่เก็บข้อมูลน้อย

```swift
import Foundation

// หา Caches Directory
func getCachesDirectory() -> URL {
    let paths = FileManager.default.urls(for: .cachesDirectory, in: .userDomainMask)
    return paths[0]
}

let cachesURL = getCachesDirectory()
print("Caches: \(cachesURL.path)")

// การบันทึกข้อมูลใน Cache
class ImageCache {
    static let shared = ImageCache()
    
    private let fileManager = FileManager.default
    private let cacheURL: URL
    
    private init() {
        let caches = fileManager.urls(for: .cachesDirectory, in: .userDomainMask)[0]
        cacheURL = caches.appendingPathComponent("ImageCache", isDirectory: true)
        
        try? fileManager.createDirectory(at: cacheURL, withIntermediateDirectories: true, attributes: nil)
    }
    
    func cacheURL(for key: String) -> URL {
        let safeKey = key.replacingOccurrences(of: "/", with: "_")
            .replacingOccurrences(of: ":", with: "_")
        return cacheURL.appendingPathComponent(safeKey)
    }
    
    func save(_ data: Data, for key: String) {
        let url = cacheURL(for: key)
        try? data.write(to: url)
    }
    
    func load(for key: String) -> Data? {
        let url = cacheURL(for: key)
        return try? Data(contentsOf: url)
    }
    
    func remove(for key: String) {
        let url = cacheURL(for: key)
        try? fileManager.removeItem(at: url)
    }
    
    func clearAll() {
        try? fileManager.removeItem(at: cacheURL)
        try? fileManager.createDirectory(at: cacheURL, withIntermediateDirectories: true, attributes: nil)
    }
    
    var totalSize: Int64 {
        guard let enumerator = fileManager.enumerator(
            at: cacheURL,
            includingPropertiesForKeys: [.fileSizeKey]
        ) else { return 0 }
        
        return (enumerator.allObjects as? [URL])?.reduce(0) { total, url in
            let size = (try? url.resourceValues(forKeys: [.fileSizeKey]))?.fileSize ?? 0
            return total + Int64(size)
        } ?? 0
    }
}
```

---

## 33.10 Temporary Directory

Temporary Directory ใช้เก็บไฟล์ชั่วคราวที่ใช้แล้วทิ้ง ระบบจะลบข้อมูลในนี้ตามดุลยพินิจ

```swift
import Foundation

// หา Temporary Directory
let tmpURL = FileManager.default.temporaryDirectory
print("Temp: \(tmpURL.path)")

// สร้างไฟล์ชั่วคราว
func createTemporaryFile(extension ext: String = "tmp") -> URL {
    let fileName = UUID().uuidString + "." + ext
    return FileManager.default.temporaryDirectory.appendingPathComponent(fileName)
}

let tempFile = createTemporaryFile(extension: "txt")
try? "Temporary content".write(to: tempFile, atomically: true, encoding: .utf8)
print("Temp file: \(tempFile.lastPathComponent)")

// ลบหลังใช้งาน
defer {
    try? FileManager.default.removeItem(at: tempFile)
}

// สร้างโฟลเดอร์ชั่วคราว
func createTemporaryDirectory() -> URL? {
    let url = FileManager.default.temporaryDirectory.appendingPathComponent(UUID().uuidString)
    do {
        try FileManager.default.createDirectory(at: url, withIntermediateDirectories: true, attributes: nil)
        return url
    } catch {
        return nil
    }
}
```

---

## 33.11 การสร้างไฟล์และโฟลเดอร์

```swift
import Foundation

let fileManager = FileManager.default
let documentsURL = fileManager.urls(for: .documentDirectory, in: .userDomainMask)[0]

// สร้างโฟลเดอร์
func createDirectory(named name: String, at parentURL: URL) throws -> URL {
    let dirURL = parentURL.appendingPathComponent(name, isDirectory: true)
    
    if !fileManager.fileExists(atPath: dirURL.path) {
        try fileManager.createDirectory(
            at: dirURL,
            withIntermediateDirectories: true,
            attributes: nil
        )
        print("Created directory: \(name)")
    } else {
        print("Directory already exists: \(name)")
    }
    
    return dirURL
}

// สร้างไฟล์
func createFile(named name: String, at directoryURL: URL, content: Data = Data()) throws -> URL {
    let fileURL = directoryURL.appendingPathComponent(name)
    
    if !fileManager.fileExists(atPath: fileURL.path) {
        fileManager.createFile(atPath: fileURL.path, contents: content, attributes: nil)
        print("Created file: \(name)")
    } else {
        print("File already exists: \(name)")
    }
    
    return fileURL
}

// การใช้งาน
do {
    let projectDir = try createDirectory(named: "MyProject", at: documentsURL)
    let srcDir = try createDirectory(named: "Sources", at: projectDir)
    let _ = try createFile(named: "main.swift", at: srcDir, content: "// Main file".data(using: .utf8)!)
    
    print("Project structure created")
} catch {
    print("Error: \(error)")
}
```

---

## 33.12 การอ่านและเขียนไฟล์

### การเขียนและอ่านข้อความ

```swift
import Foundation

// เขียนข้อความ
func writeText(_ text: String, to url: URL) throws {
    try text.write(to: url, atomically: true, encoding: .utf8)
}

// อ่านข้อความ
func readText(from url: URL) throws -> String {
    try String(contentsOf: url, encoding: .utf8)
}

// ตัวอย่างการใช้งาน
let documentsURL = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)[0]
let noteURL = documentsURL.appendingPathComponent("note.txt")

do {
    let content = """
    วันนี้ฉันเรียนรู้เรื่อง:
    - UserDefaults
    - FileManager
    - การบันทึกข้อมูล
    
    พรุ่งนี้จะเรียนต่อเรื่อง Keychain
    """
    
    try writeText(content, to: noteURL)
    print("Saved note")
    
    let loaded = try readText(from: noteURL)
    print("Loaded:\n\(loaded)")
} catch {
    print("Error: \(error)")
}
```

### การเขียนและอ่าน Data

```swift
import Foundation

// เขียน Data
func writeData(_ data: Data, to url: URL) throws {
    try data.write(to: url, options: [.atomic])
}

// อ่าน Data
func readData(from url: URL) throws -> Data {
    try Data(contentsOf: url)
}

// ตัวอย่าง: บันทึก image data
let imageURL = documentsURL.appendingPathComponent("photo.jpg")

// สมมติว่ามี imageData
let imageData = Data(repeating: 0xFF, count: 1024) // Mock data

do {
    try writeData(imageData, to: imageURL)
    print("Image saved: \(imageURL.lastPathComponent)")
    
    let loadedData = try readData(from: imageURL)
    print("Image loaded: \(loadedData.count) bytes")
} catch {
    print("Error: \(error)")
}
```

### การเขียนและอ่าน Streams

```swift
import Foundation

// การเขียนด้วย OutputStream (สำหรับไฟล์ขนาดใหญ่)
func writeWithStream(text: String, to url: URL) {
    guard let stream = OutputStream(url: url, append: false) else {
        print("Could not open stream")
        return
    }
    
    stream.open()
    defer { stream.close() }
    
    if let data = text.data(using: .utf8) {
        data.withUnsafeBytes { bytes in
            stream.write(bytes.bindMemory(to: UInt8.self).baseAddress!, maxLength: data.count)
        }
    }
}

// การอ่านด้วย InputStream
func readWithStream(from url: URL) -> String? {
    guard let stream = InputStream(url: url) else {
        return nil
    }
    
    stream.open()
    defer { stream.close() }
    
    var result = Data()
    let bufferSize = 1024
    var buffer = [UInt8](repeating: 0, count: bufferSize)
    
    while stream.hasBytesAvailable {
        let count = stream.read(&buffer, maxLength: bufferSize)
        if count > 0 {
            result.append(contentsOf: buffer[0..<count])
        }
    }
    
    return String(data: result, encoding: .utf8)
}
```

---

## 33.13 File Attributes

```swift
import Foundation

let fileManager = FileManager.default
let documentsURL = fileManager.urls(for: .documentDirectory, in: .userDomainMask)[0]
let fileURL = documentsURL.appendingPathComponent("test.txt")

// สร้างไฟล์ทดสอบ
try? "Test content".write(to: fileURL, atomically: true, encoding: .utf8)

// อ่าน attributes
do {
    let attributes = try fileManager.attributesOfItem(atPath: fileURL.path)
    
    // ขนาดไฟล์
    if let size = attributes[.size] as? Int64 {
        print("Size: \(size) bytes")
    }
    
    // วันที่สร้าง
    if let created = attributes[.creationDate] as? Date {
        print("Created: \(created)")
    }
    
    // วันที่แก้ไขล่าสุด
    if let modified = attributes[.modificationDate] as? Date {
        print("Modified: \(modified)")
    }
    
    // ประเภท
    if let type = attributes[.type] as? FileAttributeType {
        switch type {
        case .typeRegular: print("Regular file")
        case .typeDirectory: print("Directory")
        case .typeSymbolicLink: print("Symbolic link")
        default: print("Other type")
        }
    }
    
    // Permissions
    if let permissions = attributes[.posixPermissions] as? NSNumber {
        print("Permissions: \(String(permissions.intValue, radix: 8))")
    }
    
} catch {
    print("Error reading attributes: \(error)")
}

// การใช้ ResourceValues (แนะนำมากกว่า)
do {
    let resourceValues = try fileURL.resourceValues(forKeys: [
        .fileSizeKey,
        .creationDateKey,
        .contentModificationDateKey,
        .isDirectoryKey,
        .isReadableKey,
        .isWritableKey,
        .localizedNameKey
    ])
    
    print("File size: \(resourceValues.fileSize ?? 0)")
    print("Created: \(resourceValues.creationDate?.description ?? "unknown")")
    print("Is directory: \(resourceValues.isDirectory ?? false)")
    print("Is readable: \(resourceValues.isReadable ?? false)")
    
} catch {
    print("Error: \(error)")
}
```

### การแก้ไข Attributes

```swift
import Foundation

let fileManager = FileManager.default
let documentsURL = fileManager.urls(for: .documentDirectory, in: .userDomainMask)[0]
let fileURL = documentsURL.appendingPathComponent("locked.txt")

// สร้างไฟล์
try? "Locked content".write(to: fileURL, atomically: true, encoding: .utf8)

// ล็อคไฟล์ (ไม่สามารถลบได้โดยตรง)
do {
    try fileManager.setAttributes(
        [.immutable: true],
        ofItemAtPath: fileURL.path
    )
    print("File locked")
    
    // ปลดล็อค
    try fileManager.setAttributes(
        [.immutable: false],
        ofItemAtPath: fileURL.path
    )
    print("File unlocked")
} catch {
    print("Error: \(error)")
}

// เปลี่ยนวันที่แก้ไข
do {
    let newDate = Calendar.current.date(byAdding: .day, value: -7, to: Date())!
    try fileManager.setAttributes(
        [.modificationDate: newDate],
        ofItemAtPath: fileURL.path
    )
} catch {
    print("Error changing date: \(error)")
}
```

---

## 33.14 การย้ายและคัดลอกไฟล์

```swift
import Foundation

let fileManager = FileManager.default
let documentsURL = fileManager.urls(for: .documentDirectory, in: .userDomainMask)[0]

// สร้างโครงสร้างไฟล์ทดสอบ
let sourceDir = documentsURL.appendingPathComponent("Source")
let destDir = documentsURL.appendingPathComponent("Destination")

try? fileManager.createDirectory(at: sourceDir, withIntermediateDirectories: true, attributes: nil)
try? fileManager.createDirectory(at: destDir, withIntermediateDirectories: true, attributes: nil)

let sourceFile = sourceDir.appendingPathComponent("document.txt")
try? "Original content".write(to: sourceFile, atomically: true, encoding: .utf8)

// คัดลอกไฟล์
let copyDest = destDir.appendingPathComponent("document_copy.txt")
do {
    try fileManager.copyItem(at: sourceFile, to: copyDest)
    print("File copied successfully")
} catch {
    print("Copy error: \(error)")
}

// ย้ายไฟล์
let moveDest = destDir.appendingPathComponent("document.txt")
do {
    try fileManager.moveItem(at: sourceFile, to: moveDest)
    print("File moved successfully")
} catch {
    print("Move error: \(error)")
}

// คัดลอกโฟลเดอร์ทั้งหมด
let sourceDirCopy = documentsURL.appendingPathComponent("SourceCopy")
do {
    try fileManager.copyItem(at: sourceDir, to: sourceDirCopy)
    print("Directory copied successfully")
} catch {
    print("Directory copy error: \(error)")
}
```

### การย้ายด้วย Progress

```swift
import Foundation

class FileOperationHelper: NSObject, FileManagerDelegate {
    let fileManager: FileManager
    
    override init() {
        self.fileManager = FileManager()
        super.init()
        self.fileManager.delegate = self
    }
    
    func fileManager(_ fileManager: FileManager, shouldMoveItemAt srcURL: URL, to dstURL: URL) -> Bool {
        print("Moving: \(srcURL.lastPathComponent) -> \(dstURL.lastPathComponent)")
        return true
    }
    
    func fileManager(_ fileManager: FileManager, shouldCopyItemAt srcURL: URL, to dstURL: URL) -> Bool {
        print("Copying: \(srcURL.lastPathComponent) -> \(dstURL.lastPathComponent)")
        return true
    }
    
    func fileManager(_ fileManager: FileManager, shouldProceedAfterError error: Error, movingItemAt srcURL: URL, to dstURL: URL) -> Bool {
        print("Error during move: \(error)")
        return false
    }
    
    func moveFile(from source: URL, to destination: URL) throws {
        if fileManager.fileExists(atPath: destination.path) {
            try fileManager.removeItem(at: destination)
        }
        try fileManager.moveItem(at: source, to: destination)
    }
}
```

---

## 33.15 การลบไฟล์

```swift
import Foundation

let fileManager = FileManager.default
let documentsURL = fileManager.urls(for: .documentDirectory, in: .userDomainMask)[0]

// ลบไฟล์เดียว
func deleteFile(at url: URL) -> Bool {
    do {
        try fileManager.removeItem(at: url)
        print("Deleted: \(url.lastPathComponent)")
        return true
    } catch {
        print("Delete error: \(error)")
        return false
    }
}

// ลบโฟลเดอร์และเนื้อหาทั้งหมด
func deleteDirectory(at url: URL) -> Bool {
    do {
        try fileManager.removeItem(at: url)
        print("Deleted directory: \(url.lastPathComponent)")
        return true
    } catch {
        print("Delete error: \(error)")
        return false
    }
}

// ลบไฟล์หลายไฟล์
func deleteFiles(at urls: [URL]) {
    for url in urls {
        _ = deleteFile(at: url)
    }
}

// ลบไฟล์เก่า (เก่ากว่า n วัน)
func deleteOldFiles(in directory: URL, olderThan days: Int) {
    let cutoffDate = Calendar.current.date(byAdding: .day, value: -days, to: Date())!
    
    guard let enumerator = fileManager.enumerator(
        at: directory,
        includingPropertiesForKeys: [.contentModificationDateKey],
        options: [.skipsSubdirectoryDescendants]
    ) else { return }
    
    for case let fileURL as URL in enumerator {
        if let modDate = try? fileURL.resourceValues(forKeys: [.contentModificationDateKey]).contentModificationDate,
           modDate < cutoffDate {
            _ = deleteFile(at: fileURL)
        }
    }
}

// ตัวอย่าง: ล้าง cache เก่ากว่า 7 วัน
let cachesURL = fileManager.urls(for: .cachesDirectory, in: .userDomainMask)[0]
deleteOldFiles(in: cachesURL, olderThan: 7)
```

---

## 33.16 Bundle Resources

Bundle Resources คือไฟล์ที่ถูกรวมอยู่ในแอป เช่น รูปภาพ, ไฟล์ข้อมูล, plist ฯลฯ

```swift
import Foundation

// หาไฟล์ใน Bundle
if let path = Bundle.main.path(forResource: "data", ofType: "json") {
    print("Found data.json at: \(path)")
    
    if let data = try? Data(contentsOf: URL(fileURLWithPath: path)) {
        print("Loaded \(data.count) bytes")
    }
}

// ใช้ URL แทน path
if let url = Bundle.main.url(forResource: "config", withExtension: "plist") {
    print("Config URL: \(url)")
}

// หาไฟล์ในโฟลเดอร์ย่อย
if let url = Bundle.main.url(forResource: "welcome", withExtension: "html", subdirectory: "Web") {
    print("Welcome HTML: \(url)")
}

// รายการทรัพยากรทั้งหมด
if let resourceURLs = Bundle.main.urls(forResourcesWithExtension: "json", subdirectory: nil) {
    print("JSON files in bundle:")
    for url in resourceURLs {
        print("- \(url.lastPathComponent)")
    }
}

// Bundle Info.plist
let appName = Bundle.main.infoDictionary?["CFBundleName"] as? String ?? "Unknown"
let version = Bundle.main.infoDictionary?["CFBundleShortVersionString"] as? String ?? "1.0"
let buildNumber = Bundle.main.infoDictionary?["CFBundleVersion"] as? String ?? "1"
let bundleID = Bundle.main.bundleIdentifier ?? "unknown"

print("App: \(appName) v\(version) (\(buildNumber))")
print("Bundle ID: \(bundleID)")
```

### การโหลดข้อมูลจาก Bundle

```swift
import Foundation

// โหลด JSON จาก Bundle
func loadJSONFromBundle<T: Decodable>(named name: String, as type: T.Type) throws -> T {
    guard let url = Bundle.main.url(forResource: name, withExtension: "json") else {
        throw NSError(domain: "BundleError", code: 404,
                     userInfo: [NSLocalizedDescriptionKey: "File not found: \(name).json"])
    }
    
    let data = try Data(contentsOf: url)
    return try JSONDecoder().decode(type, from: data)
}

// ตัวอย่างการใช้งาน
struct Country: Codable {
    let code: String
    let name: String
    let capital: String
}

// สมมติมี countries.json ใน Bundle
// do {
//     let countries = try loadJSONFromBundle(named: "countries", as: [Country].self)
//     print("Loaded \(countries.count) countries")
// } catch {
//     print("Error: \(error)")
// }
```

---

## 33.17 Property Lists (plist)

Property Lists (plist) เป็นรูปแบบข้อมูลมาตรฐานของ Apple ที่ใช้บันทึกข้อมูลแบบ structured

```swift
import Foundation

// การอ่าน plist จาก Bundle
func readPlistFromBundle(named name: String) -> [String: Any]? {
    guard let url = Bundle.main.url(forResource: name, withExtension: "plist"),
          let data = try? Data(contentsOf: url),
          let plist = try? PropertyListSerialization.propertyList(from: data, format: nil) as? [String: Any] else {
        return nil
    }
    return plist
}

// การเขียน plist
func writePlist(_ dictionary: [String: Any], to url: URL) throws {
    let data = try PropertyListSerialization.data(
        fromPropertyList: dictionary,
        format: .xml,
        options: 0
    )
    try data.write(to: url)
}

// การอ่าน plist ที่บันทึกไว้
func readPlist(from url: URL) throws -> [String: Any] {
    let data = try Data(contentsOf: url)
    guard let plist = try PropertyListSerialization.propertyList(
        from: data,
        format: nil
    ) as? [String: Any] else {
        throw NSError(domain: "PlistError", code: 1,
                     userInfo: [NSLocalizedDescriptionKey: "Invalid plist format"])
    }
    return plist
}

// ตัวอย่างการใช้งาน
let documentsURL = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)[0]
let plistURL = documentsURL.appendingPathComponent("settings.plist")

let settings: [String: Any] = [
    "theme": "dark",
    "fontSize": 16,
    "language": "th",
    "notifications": true,
    "lastSync": Date()
]

do {
    try writePlist(settings, to: plistURL)
    print("Plist saved")
    
    let loaded = try readPlist(from: plistURL)
    print("Theme: \(loaded["theme"] ?? "unknown")")
    print("Font Size: \(loaded["fontSize"] ?? 0)")
} catch {
    print("Error: \(error)")
}
```

### Codable กับ PropertyListEncoder

```swift
import Foundation

struct AppConfiguration: Codable {
    var theme: String
    var fontSize: Int
    var language: String
    var enableAnalytics: Bool
    var maxCacheSize: Int
    var supportedFeatures: [String]
    
    init() {
        theme = "system"
        fontSize = 14
        language = "th"
        enableAnalytics = true
        maxCacheSize = 50 // MB
        supportedFeatures = ["darkMode", "iCloudSync", "widgets"]
    }
}

class ConfigurationManager {
    static let shared = ConfigurationManager()
    
    private let configURL: URL
    private var config: AppConfiguration
    
    private init() {
        let docs = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)[0]
        configURL = docs.appendingPathComponent("config.plist")
        config = ConfigurationManager.loadConfig(from: configURL)
    }
    
    private static func loadConfig(from url: URL) -> AppConfiguration {
        guard let data = try? Data(contentsOf: url),
              let config = try? PropertyListDecoder().decode(AppConfiguration.self, from: data) else {
            return AppConfiguration()
        }
        return config
    }
    
    func save() {
        if let data = try? PropertyListEncoder().encode(config) {
            try? data.write(to: configURL)
        }
    }
    
    var theme: String {
        get { config.theme }
        set { config.theme = newValue; save() }
    }
    
    var fontSize: Int {
        get { config.fontSize }
        set { config.fontSize = newValue; save() }
    }
    
    var language: String {
        get { config.language }
        set { config.language = newValue; save() }
    }
}
```

---

## 33.18 JSON File Storage

```swift
import Foundation

// Generic JSON Storage
class JSONStore<T: Codable> {
    private let fileURL: URL
    private let encoder = JSONEncoder()
    private let decoder = JSONDecoder()
    
    init(fileName: String) {
        let docs = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)[0]
        fileURL = docs.appendingPathComponent("\(fileName).json")
        
        encoder.dateEncodingStrategy = .iso8601
        encoder.outputFormatting = [.prettyPrinted, .sortedKeys]
        decoder.dateDecodingStrategy = .iso8601
    }
    
    func save(_ value: T) throws {
        let data = try encoder.encode(value)
        try data.write(to: fileURL, options: [.atomic])
    }
    
    func load() throws -> T {
        let data = try Data(contentsOf: fileURL)
        return try decoder.decode(T.self, from: data)
    }
    
    func delete() throws {
        try FileManager.default.removeItem(at: fileURL)
    }
    
    var exists: Bool {
        FileManager.default.fileExists(atPath: fileURL.path)
    }
}

// การใช้งาน
struct UserData: Codable {
    var profile: UserProfile
    var settings: UserSettings
    var recentItems: [String]
}

struct UserProfile: Codable {
    var name: String
    var email: String
    var avatar: String?
}

struct UserSettings: Codable {
    var isDarkMode: Bool
    var language: String
    var notifications: Bool
}

let store = JSONStore<UserData>(fileName: "userData")

let userData = UserData(
    profile: UserProfile(name: "Alice", email: "alice@example.com", avatar: nil),
    settings: UserSettings(isDarkMode: true, language: "th", notifications: true),
    recentItems: ["item1", "item2", "item3"]
)

do {
    try store.save(userData)
    print("User data saved")
    
    let loaded = try store.load()
    print("Name: \(loaded.profile.name)")
    print("Dark mode: \(loaded.settings.isDarkMode)")
} catch {
    print("Error: \(error)")
}
```

### Async JSON Operations

```swift
import Foundation

actor AsyncJSONStore<T: Codable> {
    private let fileURL: URL
    private let encoder = JSONEncoder()
    private let decoder = JSONDecoder()
    
    init(fileName: String) {
        let docs = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)[0]
        fileURL = docs.appendingPathComponent("\(fileName).json")
        encoder.outputFormatting = .prettyPrinted
    }
    
    func save(_ value: T) async throws {
        let data = try encoder.encode(value)
        try await Task.detached {
            try data.write(to: self.fileURL, options: [.atomic])
        }.value
    }
    
    func load() async throws -> T {
        let data = try await Task.detached {
            try Data(contentsOf: self.fileURL)
        }.value
        return try decoder.decode(T.self, from: data)
    }
}
```

---

## 33.19 Keychain พื้นฐาน

Keychain เป็นระบบเก็บข้อมูลที่มีความปลอดภัยสูง เหมาะสำหรับ passwords, tokens, และข้อมูลที่ต้องการการป้องกัน

```swift
import Foundation
import Security

// Simple Keychain Wrapper
class KeychainManager {
    static let shared = KeychainManager()
    
    private init() {}
    
    // บันทึก string
    func save(_ value: String, for key: String, service: String = "com.example.myapp") -> Bool {
        guard let data = value.data(using: .utf8) else { return false }
        
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: key,
            kSecValueData as String: data
        ]
        
        // ลบของเก่าก่อน
        SecItemDelete(query as CFDictionary)
        
        // บันทึกใหม่
        let status = SecItemAdd(query as CFDictionary, nil)
        return status == errSecSuccess
    }
    
    // อ่าน string
    func load(for key: String, service: String = "com.example.myapp") -> String? {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: key,
            kSecReturnData as String: true,
            kSecMatchLimit as String: kSecMatchLimitOne
        ]
        
        var item: CFTypeRef?
        let status = SecItemCopyMatching(query as CFDictionary, &item)
        
        guard status == errSecSuccess,
              let data = item as? Data,
              let value = String(data: data, encoding: .utf8) else {
            return nil
        }
        
        return value
    }
    
    // ลบ
    func delete(for key: String, service: String = "com.example.myapp") -> Bool {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: key
        ]
        
        let status = SecItemDelete(query as CFDictionary)
        return status == errSecSuccess || status == errSecItemNotFound
    }
    
    // ตรวจสอบว่ามีอยู่หรือไม่
    func exists(for key: String, service: String = "com.example.myapp") -> Bool {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: key,
            kSecReturnData as String: false
        ]
        
        let status = SecItemCopyMatching(query as CFDictionary, nil)
        return status == errSecSuccess
    }
}

// การใช้งาน
let keychain = KeychainManager.shared

// บันทึก token
if keychain.save("eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9", for: "authToken") {
    print("Token saved")
}

// อ่าน token
if let token = keychain.load(for: "authToken") {
    print("Token: \(token.prefix(20))...")
}

// ลบ token (เมื่อ logout)
if keychain.delete(for: "authToken") {
    print("Token deleted")
}
```

---

## 33.20 SecItem API

```swift
import Foundation
import Security

// ขั้นสูง: การใช้ SecItem API โดยตรง
class AdvancedKeychainManager {
    enum KeychainError: Error {
        case duplicateItem
        case itemNotFound
        case invalidData
        case unexpectedStatus(OSStatus)
    }
    
    // บันทึก Data
    func saveData(_ data: Data, for key: String, accessibility: CFString = kSecAttrAccessibleWhenUnlocked) throws {
        var query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: key,
            kSecAttrAccessible as String: accessibility,
            kSecValueData as String: data
        ]
        
        var status = SecItemAdd(query as CFDictionary, nil)
        
        if status == errSecDuplicateItem {
            // Update แทน
            let updateQuery: [String: Any] = [
                kSecClass as String: kSecClassGenericPassword,
                kSecAttrAccount as String: key
            ]
            let updateAttrs: [String: Any] = [
                kSecValueData as String: data
            ]
            status = SecItemUpdate(updateQuery as CFDictionary, updateAttrs as CFDictionary)
        }
        
        guard status == errSecSuccess else {
            throw KeychainError.unexpectedStatus(status)
        }
    }
    
    // อ่าน Data
    func loadData(for key: String) throws -> Data {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: key,
            kSecReturnData as String: true,
            kSecMatchLimit as String: kSecMatchLimitOne
        ]
        
        var item: CFTypeRef?
        let status = SecItemCopyMatching(query as CFDictionary, &item)
        
        guard status == errSecSuccess else {
            if status == errSecItemNotFound {
                throw KeychainError.itemNotFound
            }
            throw KeychainError.unexpectedStatus(status)
        }
        
        guard let data = item as? Data else {
            throw KeychainError.invalidData
        }
        
        return data
    }
    
    // บันทึก Codable
    func save<T: Encodable>(_ value: T, for key: String) throws {
        let data = try JSONEncoder().encode(value)
        try saveData(data, for: key)
    }
    
    // อ่าน Codable
    func load<T: Decodable>(_ type: T.Type, for key: String) throws -> T {
        let data = try loadData(for: key)
        return try JSONDecoder().decode(type, from: data)
    }
    
    // รายการ keys ทั้งหมด
    func allKeys() -> [String] {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecReturnAttributes as String: true,
            kSecMatchLimit as String: kSecMatchLimitAll
        ]
        
        var items: CFTypeRef?
        guard SecItemCopyMatching(query as CFDictionary, &items) == errSecSuccess,
              let itemArray = items as? [[String: Any]] else {
            return []
        }
        
        return itemArray.compactMap { $0[kSecAttrAccount as String] as? String }
    }
    
    // ลบทุกอย่าง
    func clearAll() {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword
        ]
        SecItemDelete(query as CFDictionary)
    }
}

// Keychain กับ Biometric Authentication
class BiometricKeychain {
    func saveWithBiometrics(_ data: Data, for key: String) throws {
        // สร้าง access control ที่ต้องใช้ biometrics
        var error: Unmanaged<CFError>?
        guard let access = SecAccessControlCreateWithFlags(
            kCFAllocatorDefault,
            kSecAttrAccessibleWhenPasscodeSetThisDeviceOnly,
            .userPresence,  // ต้องผ่าน Face ID หรือ Touch ID
            &error
        ) else {
            throw error!.takeRetainedValue() as Error
        }
        
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: key,
            kSecAttrAccessControl as String: access,
            kSecValueData as String: data,
            kSecUseDataProtectionKeychain as String: true
        ]
        
        SecItemDelete(query as CFDictionary)
        let status = SecItemAdd(query as CFDictionary, nil)
        
        guard status == errSecSuccess else {
            throw NSError(domain: NSOSStatusErrorDomain, code: Int(status))
        }
    }
}
```

---

## 33.21 แบบฝึกหัดพร้อมเฉลย

### แบบฝึกหัดที่ 1: User Preferences Manager

สร้าง class สำหรับจัดการ preferences ของผู้ใช้ที่ครอบคลุม

```swift
import Foundation

// โจทย์: สร้าง PreferencesManager ที่รองรับ:
// 1. บันทึกและอ่านการตั้งค่าพื้นฐาน
// 2. รองรับ custom types ด้วย Codable
// 3. มี default values
// 4. Reset เป็นค่า default ได้
// 5. Export/Import การตั้งค่า

// เฉลย:

struct Preferences: Codable {
    var theme: Theme
    var fontSize: FontSize
    var language: String
    var notificationsEnabled: Bool
    var autoSaveInterval: Int // seconds
    var recentFiles: [String]
    var customColors: [String: String]
    
    enum Theme: String, Codable, CaseIterable {
        case light, dark, system
        
        var displayName: String {
            switch self {
            case .light: return "สว่าง"
            case .dark: return "มืด"
            case .system: return "ตามระบบ"
            }
        }
    }
    
    enum FontSize: Int, Codable, CaseIterable {
        case small = 12, medium = 14, large = 16, extraLarge = 18
        
        var displayName: String {
            switch self {
            case .small: return "เล็ก"
            case .medium: return "กลาง"
            case .large: return "ใหญ่"
            case .extraLarge: return "ใหญ่มาก"
            }
        }
    }
    
    static var defaults: Preferences {
        Preferences(
            theme: .system,
            fontSize: .medium,
            language: "th",
            notificationsEnabled: true,
            autoSaveInterval: 60,
            recentFiles: [],
            customColors: [:]
        )
    }
}

class PreferencesManager {
    static let shared = PreferencesManager()
    
    private let key = "userPreferences"
    private let encoder = JSONEncoder()
    private let decoder = JSONDecoder()
    
    private(set) var preferences: Preferences {
        didSet { save() }
    }
    
    private init() {
        if let data = UserDefaults.standard.data(forKey: key),
           let prefs = try? decoder.decode(Preferences.self, from: data) {
            preferences = prefs
        } else {
            preferences = .defaults
        }
    }
    
    private func save() {
        if let data = try? encoder.encode(preferences) {
            UserDefaults.standard.set(data, forKey: key)
        }
    }
    
    func reset() {
        preferences = .defaults
    }
    
    func export() -> Data? {
        try? encoder.encode(preferences)
    }
    
    func importPreferences(from data: Data) throws {
        preferences = try decoder.decode(Preferences.self, from: data)
    }
    
    func addRecentFile(_ path: String) {
        var prefs = preferences
        prefs.recentFiles.removeAll { $0 == path }
        prefs.recentFiles.insert(path, at: 0)
        if prefs.recentFiles.count > 10 {
            prefs.recentFiles = Array(prefs.recentFiles.prefix(10))
        }
        preferences = prefs
    }
    
    // Convenience setters
    func setTheme(_ theme: Preferences.Theme) {
        preferences.theme = theme
    }
    
    func setFontSize(_ size: Preferences.FontSize) {
        preferences.fontSize = size
    }
    
    func setLanguage(_ code: String) {
        preferences.language = code
    }
    
    func setNotifications(_ enabled: Bool) {
        preferences.notificationsEnabled = enabled
    }
}

// การใช้งาน
let prefs = PreferencesManager.shared

prefs.setTheme(.dark)
prefs.setFontSize(.large)
prefs.addRecentFile("/Users/user/Documents/project.swift")
prefs.addRecentFile("/Users/user/Documents/data.json")

print("Theme: \(prefs.preferences.theme.displayName)")
print("Font: \(prefs.preferences.fontSize.displayName)")
print("Recent files: \(prefs.preferences.recentFiles.count)")

// Export
if let exported = prefs.export() {
    print("Exported \(exported.count) bytes")
}

// Reset
prefs.reset()
print("After reset - Theme: \(prefs.preferences.theme.rawValue)")
```

### แบบฝึกหัดที่ 2: File-based Note Taking App

```swift
import Foundation

// โจทย์: สร้างระบบบันทึก Note ที่:
// 1. บันทึกแต่ละ note เป็นไฟล์ JSON แยกกัน
// 2. รองรับการสร้าง อ่าน แก้ไข ลบ
// 3. รองรับการค้นหา
// 4. มี metadata (วันที่สร้าง แก้ไข tag)

struct Note: Codable, Identifiable {
    var id: UUID
    var title: String
    var content: String
    var tags: [String]
    var createdAt: Date
    var updatedAt: Date
    var isPinned: Bool
    
    init(title: String, content: String = "", tags: [String] = []) {
        self.id = UUID()
        self.title = title
        self.content = content
        self.tags = tags
        self.createdAt = Date()
        self.updatedAt = Date()
        self.isPinned = false
    }
}

class NoteRepository {
    private let fileManager = FileManager.default
    private let encoder: JSONEncoder
    private let decoder: JSONDecoder
    private let notesDirectory: URL
    
    init() {
        let docs = fileManager.urls(for: .documentDirectory, in: .userDomainMask)[0]
        notesDirectory = docs.appendingPathComponent("Notes", isDirectory: true)
        
        encoder = JSONEncoder()
        encoder.dateEncodingStrategy = .iso8601
        encoder.outputFormatting = .prettyPrinted
        
        decoder = JSONDecoder()
        decoder.dateDecodingStrategy = .iso8601
        
        try? fileManager.createDirectory(at: notesDirectory, withIntermediateDirectories: true, attributes: nil)
    }
    
    private func fileURL(for id: UUID) -> URL {
        notesDirectory.appendingPathComponent("\(id.uuidString).json")
    }
    
    // สร้างหรืออัพเดต
    func save(_ note: Note) throws {
        var mutableNote = note
        mutableNote.updatedAt = Date()
        let data = try encoder.encode(mutableNote)
        try data.write(to: fileURL(for: note.id), options: .atomic)
    }
    
    // อ่านหนึ่ง note
    func load(id: UUID) throws -> Note {
        let data = try Data(contentsOf: fileURL(for: id))
        return try decoder.decode(Note.self, from: data)
    }
    
    // อ่านทั้งหมด
    func loadAll() -> [Note] {
        guard let files = try? fileManager.contentsOfDirectory(
            at: notesDirectory,
            includingPropertiesForKeys: nil,
            options: [.skipsHiddenFiles]
        ) else { return [] }
        
        return files.filter { $0.pathExtension == "json" }.compactMap { url in
            try? decoder.decode(Note.self, from: Data(contentsOf: url))
        }.sorted { $0.updatedAt > $1.updatedAt }
    }
    
    // ลบ
    func delete(id: UUID) throws {
        try fileManager.removeItem(at: fileURL(for: id))
    }
    
    // ค้นหา
    func search(query: String) -> [Note] {
        let lowercased = query.lowercased()
        return loadAll().filter { note in
            note.title.lowercased().contains(lowercased) ||
            note.content.lowercased().contains(lowercased) ||
            note.tags.contains { $0.lowercased().contains(lowercased) }
        }
    }
    
    // กรองด้วย tag
    func notes(withTag tag: String) -> [Note] {
        loadAll().filter { $0.tags.contains(tag) }
    }
    
    // ทั้งหมดที่ pin
    var pinnedNotes: [Note] {
        loadAll().filter { $0.isPinned }
    }
    
    // สถิติ
    var totalCount: Int {
        (try? fileManager.contentsOfDirectory(at: notesDirectory, includingPropertiesForKeys: nil).count) ?? 0
    }
}

// การใช้งาน
let repo = NoteRepository()

var note1 = Note(title: "Swift เบื้องต้น", content: "Swift เป็นภาษาที่พัฒนาโดย Apple...", tags: ["swift", "programming"])
var note2 = Note(title: "UserDefaults", content: "การบันทึกข้อมูลด้วย UserDefaults...", tags: ["swift", "storage"])
note2.isPinned = true

do {
    try repo.save(note1)
    try repo.save(note2)
    print("Saved \(repo.totalCount) notes")
    
    let results = repo.search(query: "swift")
    print("Search results: \(results.count)")
    
    let swiftNotes = repo.notes(withTag: "swift")
    print("Swift tagged: \(swiftNotes.count)")
    
    let pinned = repo.pinnedNotes
    print("Pinned: \(pinned.count)")
} catch {
    print("Error: \(error)")
}
```

---

## 33.22 การสร้าง Settings Screen พร้อม Persistence

```swift
import SwiftUI

// MARK: - Models

struct AppSettings: Codable, Equatable {
    var profile: ProfileSettings
    var appearance: AppearanceSettings
    var privacy: PrivacySettings
    var notifications: NotificationSettings
    
    struct ProfileSettings: Codable, Equatable {
        var name: String
        var email: String
        var bio: String
        var avatarData: Data?
    }
    
    struct AppearanceSettings: Codable, Equatable {
        var theme: String  // "light", "dark", "system"
        var accentColor: String  // "blue", "red", "green", etc.
        var fontSize: Double
        var useSystemFont: Bool
        var showAnimations: Bool
    }
    
    struct PrivacySettings: Codable, Equatable {
        var analyticsEnabled: Bool
        var crashReportingEnabled: Bool
        var locationEnabled: Bool
    }
    
    struct NotificationSettings: Codable, Equatable {
        var pushEnabled: Bool
        var emailEnabled: Bool
        var quietHoursEnabled: Bool
        var quietHoursStart: Int  // hour (0-23)
        var quietHoursEnd: Int
    }
    
    static var defaults: AppSettings {
        AppSettings(
            profile: ProfileSettings(name: "", email: "", bio: ""),
            appearance: AppearanceSettings(
                theme: "system",
                accentColor: "blue",
                fontSize: 14,
                useSystemFont: true,
                showAnimations: true
            ),
            privacy: PrivacySettings(
                analyticsEnabled: true,
                crashReportingEnabled: true,
                locationEnabled: false
            ),
            notifications: NotificationSettings(
                pushEnabled: true,
                emailEnabled: false,
                quietHoursEnabled: false,
                quietHoursStart: 22,
                quietHoursEnd: 8
            )
        )
    }
}

// MARK: - ViewModel

class SettingsViewModel: ObservableObject {
    @Published var settings: AppSettings {
        didSet {
            if settings != oldValue {
                saveSettings()
            }
        }
    }
    
    @Published var isSaving = false
    @Published var saveError: String?
    
    private let key = "appSettings"
    private let encoder = JSONEncoder()
    private let decoder = JSONDecoder()
    
    init() {
        if let data = UserDefaults.standard.data(forKey: "appSettings"),
           let saved = try? JSONDecoder().decode(AppSettings.self, from: data) {
            settings = saved
        } else {
            settings = .defaults
        }
    }
    
    func saveSettings() {
        if let data = try? encoder.encode(settings) {
            UserDefaults.standard.set(data, forKey: key)
        }
    }
    
    func resetToDefaults() {
        settings = .defaults
    }
    
    func exportSettings() -> URL? {
        guard let data = try? encoder.encode(settings) else { return nil }
        let url = FileManager.default.temporaryDirectory.appendingPathComponent("settings_export.json")
        try? data.write(to: url)
        return url
    }
}

// MARK: - Views

struct SettingsScreen: View {
    @StateObject private var viewModel = SettingsViewModel()
    @State private var showingResetAlert = false
    @State private var showingExportSheet = false
    @State private var exportURL: URL?
    
    var body: some View {
        NavigationView {
            List {
                // Profile Section
                Section {
                    NavigationLink("โปรไฟล์") {
                        ProfileSettingsView(settings: $viewModel.settings.profile)
                    }
                } header: {
                    Text("บัญชีผู้ใช้")
                }
                
                // Appearance Section
                Section {
                    NavigationLink("การแสดงผล") {
                        AppearanceSettingsView(settings: $viewModel.settings.appearance)
                    }
                } header: {
                    Text("การแสดงผล")
                }
                
                // Privacy Section
                Section {
                    NavigationLink("ความเป็นส่วนตัว") {
                        PrivacySettingsView(settings: $viewModel.settings.privacy)
                    }
                } header: {
                    Text("ความเป็นส่วนตัว")
                }
                
                // Notifications Section
                Section {
                    NavigationLink("การแจ้งเตือน") {
                        NotificationSettingsView(settings: $viewModel.settings.notifications)
                    }
                } header: {
                    Text("การแจ้งเตือน")
                }
                
                // Actions Section
                Section {
                    Button("Export การตั้งค่า") {
                        exportURL = viewModel.exportSettings()
                        showingExportSheet = exportURL != nil
                    }
                    
                    Button("Reset เป็นค่าเริ่มต้น", role: .destructive) {
                        showingResetAlert = true
                    }
                } header: {
                    Text("การจัดการ")
                }
            }
            .navigationTitle("การตั้งค่า")
            .alert("Reset การตั้งค่า?", isPresented: $showingResetAlert) {
                Button("Reset", role: .destructive) {
                    viewModel.resetToDefaults()
                }
                Button("ยกเลิก", role: .cancel) {}
            } message: {
                Text("การตั้งค่าทั้งหมดจะถูกรีเซ็ตเป็นค่าเริ่มต้น")
            }
        }
    }
}

struct ProfileSettingsView: View {
    @Binding var settings: AppSettings.ProfileSettings
    
    var body: some View {
        Form {
            Section("ข้อมูลส่วนตัว") {
                TextField("ชื่อ", text: $settings.name)
                TextField("อีเมล", text: $settings.email)
                    .keyboardType(.emailAddress)
                    .autocapitalization(.none)
            }
            
            Section("เกี่ยวกับฉัน") {
                TextEditor(text: $settings.bio)
                    .frame(height: 100)
            }
        }
        .navigationTitle("โปรไฟล์")
    }
}

struct AppearanceSettingsView: View {
    @Binding var settings: AppSettings.AppearanceSettings
    
    let themes = [("light", "สว่าง"), ("dark", "มืด"), ("system", "ตามระบบ")]
    let accentColors = [("blue", "น้ำเงิน"), ("red", "แดง"), ("green", "เขียว"), ("orange", "ส้ม")]
    
    var body: some View {
        Form {
            Section("ธีม") {
                Picker("ธีม", selection: $settings.theme) {
                    ForEach(themes, id: \.0) { theme in
                        Text(theme.1).tag(theme.0)
                    }
                }
                .pickerStyle(.segmented)
            }
            
            Section("สี Accent") {
                Picker("สี", selection: $settings.accentColor) {
                    ForEach(accentColors, id: \.0) { color in
                        Text(color.1).tag(color.0)
                    }
                }
            }
            
            Section("ตัวอักษร") {
                Toggle("ใช้ font ของระบบ", isOn: $settings.useSystemFont)
                
                if !settings.useSystemFont {
                    HStack {
                        Text("ขนาด: \(Int(settings.fontSize))")
                        Spacer()
                        Slider(value: $settings.fontSize, in: 10...24, step: 1)
                            .frame(width: 150)
                    }
                }
            }
            
            Section("อื่นๆ") {
                Toggle("แสดง Animation", isOn: $settings.showAnimations)
            }
        }
        .navigationTitle("การแสดงผล")
    }
}

struct PrivacySettingsView: View {
    @Binding var settings: AppSettings.PrivacySettings
    
    var body: some View {
        Form {
            Section {
                Toggle("ส่งข้อมูลการใช้งาน", isOn: $settings.analyticsEnabled)
                Toggle("ส่งรายงาน Crash", isOn: $settings.crashReportingEnabled)
                Toggle("อนุญาต Location", isOn: $settings.locationEnabled)
            } header: {
                Text("การเก็บข้อมูล")
            } footer: {
                Text("ข้อมูลเหล่านี้ช่วยให้เราพัฒนาแอปได้ดียิ่งขึ้น")
            }
        }
        .navigationTitle("ความเป็นส่วนตัว")
    }
}

struct NotificationSettingsView: View {
    @Binding var settings: AppSettings.NotificationSettings
    
    var body: some View {
        Form {
            Section("ช่องทางการแจ้งเตือน") {
                Toggle("Push Notification", isOn: $settings.pushEnabled)
                Toggle("อีเมล", isOn: $settings.emailEnabled)
            }
            
            Section("เวลาเงียบ") {
                Toggle("เปิดใช้เวลาเงียบ", isOn: $settings.quietHoursEnabled)
                
                if settings.quietHoursEnabled {
                    Picker("เริ่ม", selection: $settings.quietHoursStart) {
                        ForEach(0..<24) { hour in
                            Text("\(hour):00").tag(hour)
                        }
                    }
                    
                    Picker("สิ้นสุด", selection: $settings.quietHoursEnd) {
                        ForEach(0..<24) { hour in
                            Text("\(hour):00").tag(hour)
                        }
                    }
                }
            }
        }
        .navigationTitle("การแจ้งเตือน")
    }
}
```

---

## 33.23 สรุป

ในบทนี้เราได้เรียนรู้เกี่ยวกับการจัดเก็บข้อมูลใน iOS/macOS อย่างครอบคลุม:

### UserDefaults
- ใช้สำหรับข้อมูลขนาดเล็กและการตั้งค่า
- รองรับ primitive types และ collections
- ใช้ Codable สำหรับ custom types
- `@AppStorage` ทำให้ใช้งานง่ายใน SwiftUI
- UserDefaults Suites สำหรับ App Groups

### FileManager
- จัดการไฟล์และโฟลเดอร์อย่างสมบูรณ์
- Documents Directory สำหรับข้อมูลผู้ใช้
- Caches Directory สำหรับข้อมูลชั่วคราวที่สร้างใหม่ได้
- Temporary Directory สำหรับไฟล์ที่ใช้แล้วทิ้ง

### รูปแบบการบันทึกข้อมูล
- Property Lists (plist) สำหรับข้อมูล structured แบบ Apple
- JSON สำหรับการแลกเปลี่ยนข้อมูล
- Keychain สำหรับข้อมูลที่ต้องการความปลอดภัย

### Best Practices
- ใช้ constants สำหรับ keys ใน UserDefaults
- ใช้ `@AppStorage` แทน UserDefaults โดยตรงใน SwiftUI
- ใช้ FileManager สำหรับไฟล์ขนาดใหญ่
- เก็บ credentials ใน Keychain เสมอ
- ใช้ iCloud (NSUbiquitousKeyValueStore) สำหรับ sync ข้อมูลขนาดเล็ก

### สิ่งที่ต้องระวัง
- UserDefaults ไม่เหมาะกับข้อมูลขนาดใหญ่
- ไม่เก็บ passwords ใน UserDefaults
- ใช้ `.atomic` เมื่อเขียนไฟล์สำคัญ
- จัดการ errors อย่างเหมาะสมด้วย try-catch

---

*บทต่อไป: Part 34 - Notifications ทั้ง Local และ System*
