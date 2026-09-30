# ตอนที่ 21: SwiftUI พื้นฐาน (SwiftUI Basics)

## สารบัญ
1. [SwiftUI คืออะไร](#swiftui-คืออะไร)
2. [SwiftUI vs UIKit](#swiftui-vs-uikit)
3. [โครงสร้างโปรเจกต์ SwiftUI](#โครงสร้างโปรเจกต์-swiftui)
4. [ContentView อธิบาย](#contentview-อธิบาย)
5. [@main attribute](#main-attribute)
6. [App protocol](#app-protocol)
7. [Scene types](#scene-types)
8. [View protocol](#view-protocol)
9. [body property](#body-property)
10. [Basic Views พื้นฐาน](#basic-views-พื้นฐาน)
11. [View Modifiers](#view-modifiers)
12. [ลำดับของ Modifier สำคัญ](#ลำดับของ-modifier-สำคัญ)
13. [การ Chain Modifiers](#การ-chain-modifiers)
14. [Custom Modifiers](#custom-modifiers)
15. [ViewBuilder](#viewbuilder)
16. [Previews](#previews)
17. [Environment Values](#environment-values)
18. [@Environment Property Wrapper](#environment-property-wrapper)
19. [Layout พื้นฐาน](#layout-พื้นฐาน)
20. [Spacer และ Divider](#spacer-และ-divider)
21. [Group](#group)
22. [แบบฝึกหัดพร้อมเฉลย](#แบบฝึกหัดพร้อมเฉลย)
23. [สรุป](#สรุป)

---

## SwiftUI คืออะไร

SwiftUI เป็น framework ที่ Apple เปิดตัวในปี 2019 ในงาน WWDC สำหรับการสร้าง User Interface (UI) บนแพลตฟอร์มของ Apple ทั้งหมด ได้แก่ iOS, macOS, watchOS, tvOS และ visionOS โดยใช้โค้ด Swift เพียงชุดเดียว

### คุณสมบัติเด่นของ SwiftUI

**1. Declarative Syntax (ไวยากรณ์เชิงประกาศ)**
แทนที่จะบอกว่า "ต้องทำอะไรบ้าง" (imperative) เราบอกว่า "ต้องการให้หน้าตาเป็นอย่างไร" (declarative)

```swift
// Declarative style - บอกว่าต้องการอะไร
Text("สวัสดี SwiftUI")
    .font(.largeTitle)
    .foregroundColor(.blue)
    .padding()
```

**2. Live Preview**
Xcode แสดงผล UI แบบ real-time ในขณะที่เราเขียนโค้ด ไม่ต้อง compile และ run simulator ทุกครั้ง

**3. Cross-platform**
โค้ดเดียวกันทำงานได้บนทุกแพลตฟอร์มของ Apple พร้อมการปรับแต่งตามความเหมาะสม

**4. Data-Driven UI**
UI อัปเดตอัตโนมัติเมื่อข้อมูลเปลี่ยนแปลง ผ่าน state management system

**5. Composable**
สร้าง UI จากส่วนประกอบเล็กๆ ที่เรียกว่า View แล้วนำมาประกอบกัน

### ประวัติความเป็นมา

```
2019 - SwiftUI เปิดตัวพร้อม iOS 13, macOS 10.15
2020 - SwiftUI 2.0 เพิ่ม App protocol, Scene, LazyStacks
2021 - SwiftUI 3.0 เพิ่ม searchable, refreshable, task
2022 - SwiftUI 4.0 เพิ่ม NavigationStack, Charts
2023 - SwiftUI 5.0 เพิ่ม Observable macro, SwiftData
2024 - SwiftUI 6.0 เพิ่ม visionOS improvements
```

---

## SwiftUI vs UIKit

การเปรียบเทียบระหว่าง SwiftUI และ UIKit ซึ่งเป็น framework เก่าที่ Apple ใช้มาตั้งแต่ปี 2008

### ตารางเปรียบเทียบ

| คุณสมบัติ | SwiftUI | UIKit |
|-----------|---------|-------|
| ปีที่เปิดตัว | 2019 | 2008 |
| ภาษา | Swift เท่านั้น | Swift หรือ Objective-C |
| สไตล์การเขียน | Declarative | Imperative |
| Live Preview | ใช่ | ไม่ใช่ |
| Cross-platform | ใช่ | บางส่วน |
| Storyboard | ไม่ใช้ | ใช้ได้ |
| ความเสถียร | ยังพัฒนาต่อ | เสถียรมาก |
| Community | กำลังเติบโต | ใหญ่มาก |

### ตัวอย่างการสร้าง Button

**UIKit:**
```swift
// UIKit - Imperative style
class ViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // สร้าง button
        let button = UIButton(type: .system)
        button.setTitle("กดฉัน", for: .normal)
        button.setTitleColor(.white, for: .normal)
        button.backgroundColor = .blue
        button.layer.cornerRadius = 10
        button.translatesAutoresizingMaskIntoConstraints = false
        
        // เพิ่มลงใน view
        view.addSubview(button)
        
        // กำหนด constraints
        NSLayoutConstraint.activate([
            button.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            button.centerYAnchor.constraint(equalTo: view.centerYAnchor),
            button.widthAnchor.constraint(equalToConstant: 150),
            button.heightAnchor.constraint(equalToConstant: 50)
        ])
        
        // เพิ่ม action
        button.addTarget(self, action: #selector(buttonTapped), for: .touchUpInside)
    }
    
    @objc func buttonTapped() {
        print("Button ถูกกด!")
    }
}
```

**SwiftUI:**
```swift
// SwiftUI - Declarative style
struct ContentView: View {
    var body: some View {
        Button("กดฉัน") {
            print("Button ถูกกด!")
        }
        .foregroundColor(.white)
        .padding(.horizontal, 20)
        .padding(.vertical, 15)
        .background(Color.blue)
        .cornerRadius(10)
    }
}
```

### เมื่อไหร่ควรใช้อะไร

**ใช้ SwiftUI เมื่อ:**
- เริ่มโปรเจกต์ใหม่ที่รองรับ iOS 15+ หรือใหม่กว่า
- ต้องการ cross-platform (iOS + macOS + watchOS)
- ทีมคุ้นเคยกับ functional/reactive programming
- ต้องการ development speed ที่เร็ว

**ใช้ UIKit เมื่อ:**
- รองรับ iOS เวอร์ชันเก่า (ต่ำกว่า iOS 13)
- โปรเจกต์ที่มีอยู่แล้วใช้ UIKit
- ต้องการ UI components ที่ SwiftUI ยังไม่มี
- ต้องการควบคุม performance อย่างละเอียด

### การใช้ร่วมกัน (Interoperability)

SwiftUI และ UIKit ทำงานร่วมกันได้ผ่าน:

```swift
// ใช้ UIKit ใน SwiftUI
struct MapViewRepresentable: UIViewRepresentable {
    func makeUIView(context: Context) -> MKMapView {
        return MKMapView()
    }
    
    func updateUIView(_ uiView: MKMapView, context: Context) {
        // อัปเดต map view
    }
}

// ใช้ SwiftUI ใน UIKit
let hostingController = UIHostingController(rootView: ContentView())
```

---

## โครงสร้างโปรเจกต์ SwiftUI

เมื่อสร้างโปรเจกต์ SwiftUI ใหม่ใน Xcode จะได้ไฟล์เหล่านี้:

```
MyApp/
├── MyApp.swift          // Entry point ของแอป
├── ContentView.swift    // View หลัก
├── Assets.xcassets/     // รูปภาพ, สี, Icons
│   ├── AppIcon.appiconset/
│   └── AccentColor.colorset/
├── Preview Content/     // ข้อมูลสำหรับ Preview
│   └── Preview Assets.xcassets/
└── Info.plist           // การตั้งค่าแอป
```

### MyApp.swift (Entry Point)

```swift
import SwiftUI

@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

### ContentView.swift

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        VStack {
            Image(systemName: "globe")
                .imageScale(.large)
                .foregroundStyle(.tint)
            Text("Hello, world!")
        }
        .padding()
    }
}

#Preview {
    ContentView()
}
```

---

## ContentView อธิบาย

ContentView เป็น View หลักที่แสดงเมื่อแอปเริ่มทำงาน มาทำความเข้าใจแต่ละส่วน:

```swift
// 1. Import SwiftUI framework
import SwiftUI

// 2. ประกาศ struct ที่ conform กับ View protocol
struct ContentView: View {
    
    // 3. body property ที่จำเป็นต้องมี
    var body: some View {
        
        // 4. เนื้อหา UI ที่ต้องการแสดง
        VStack {
            Text("สวัสดี SwiftUI!")
                .font(.title)
            
            Text("ยินดีต้อนรับสู่การพัฒนา iOS")
                .font(.subheadline)
                .foregroundColor(.gray)
        }
        .padding()
    }
}

// 5. Preview สำหรับดูใน Xcode Canvas
#Preview {
    ContentView()
}
```

### การแยกส่วน View

เราสามารถแยก View ย่อยออกมาได้:

```swift
struct ContentView: View {
    var body: some View {
        VStack(spacing: 20) {
            HeaderView()
            MainContentView()
            FooterView()
        }
    }
}

struct HeaderView: View {
    var body: some View {
        VStack {
            Image(systemName: "star.fill")
                .font(.system(size: 50))
                .foregroundColor(.yellow)
            
            Text("ยินดีต้อนรับ")
                .font(.largeTitle)
                .bold()
        }
        .padding()
    }
}

struct MainContentView: View {
    var body: some View {
        Text("นี่คือเนื้อหาหลัก")
            .font(.body)
            .multilineTextAlignment(.center)
            .padding(.horizontal)
    }
}

struct FooterView: View {
    var body: some View {
        Text("© 2024 My App")
            .font(.caption)
            .foregroundColor(.gray)
    }
}
```

---

## @main attribute

`@main` attribute บอก Swift ว่า struct หรือ class นี้คือ entry point ของโปรแกรม

### กฎการใช้ @main

1. มีได้เพียงหนึ่งจุดในโปรเจกต์
2. ต้องมี static func main() หรือ conform กับ protocol ที่มี main()
3. สำหรับ SwiftUI ใช้ร่วมกับ App protocol

```swift
// SwiftUI App
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}

// ถ้าต้องการ customization เพิ่มเติม
@main
struct MyApp: App {
    
    // เรียกเมื่อแอปเริ่มต้น
    init() {
        // Setup code
        configureAppearance()
        setupFirebase()
    }
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .onAppear {
                    print("App started!")
                }
        }
    }
    
    private func configureAppearance() {
        // ตั้งค่า navigation bar appearance
    }
    
    private func setupFirebase() {
        // Initialize Firebase
    }
}
```

---

## App protocol

App protocol กำหนดโครงสร้างหลักของแอป SwiftUI

### คุณสมบัติของ App protocol

```swift
public protocol App {
    associatedtype Body: Scene
    
    @SceneBuilder
    var body: Body { get }
    
    init()
}
```

### การใช้งาน App protocol

```swift
@main
struct WeatherApp: App {
    
    // State ที่แอปต้องการ
    @StateObject private var weatherStore = WeatherStore()
    
    var body: some Scene {
        WindowGroup {
            WeatherView()
                .environmentObject(weatherStore)
        }
    }
}
```

### App Delegate ใน SwiftUI

```swift
// สร้าง App Delegate
class AppDelegate: NSObject, UIApplicationDelegate {
    func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]? = nil
    ) -> Bool {
        // Setup
        return true
    }
    
    func application(
        _ application: UIApplication,
        didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data
    ) {
        // Handle push notification registration
    }
}

// ใช้งานใน App
@main
struct MyApp: App {
    
    @UIApplicationDelegateAdaptor(AppDelegate.self) var appDelegate
    
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

---

## Scene types

Scene คือ container ที่ห่อหุ้ม View hierarchy และจัดการ lifecycle

### WindowGroup

Scene ที่ใช้บ่อยที่สุด สร้าง window สำหรับแสดง content

```swift
@main
struct MyApp: App {
    var body: some Scene {
        // WindowGroup พื้นฐาน
        WindowGroup {
            ContentView()
        }
        
        // WindowGroup พร้อม customization
        WindowGroup("My Window", id: "main") {
            ContentView()
        }
        .defaultSize(width: 800, height: 600)  // macOS
        .windowStyle(.hiddenTitleBar)            // macOS
    }
}
```

### DocumentGroup

สำหรับแอปที่จัดการกับ documents (เช่น text editor, drawing app)

```swift
@main
struct DocumentApp: App {
    var body: some Scene {
        DocumentGroup(newDocument: TextDocument()) { file in
            DocumentView(document: file.$document)
        }
    }
}

// Document model
struct TextDocument: FileDocument {
    var text: String = ""
    
    static var readableContentTypes: [UTType] { [.plainText] }
    
    init(configuration: ReadConfiguration) throws {
        if let data = configuration.file.regularFileContents {
            text = String(decoding: data, as: UTF8.self)
        }
    }
    
    func fileWrapper(configuration: WriteConfiguration) throws -> FileWrapper {
        let data = text.data(using: .utf8)!
        return FileWrapper(regularFileWithContents: data)
    }
}
```

### Settings (macOS เท่านั้น)

```swift
@main
struct MacApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        
        // Settings window สำหรับ macOS
        Settings {
            SettingsView()
        }
    }
}

struct SettingsView: View {
    @AppStorage("fontSize") private var fontSize: Double = 14
    @AppStorage("darkMode") private var darkMode: Bool = false
    
    var body: some View {
        Form {
            Section("การแสดงผล") {
                Slider(value: $fontSize, in: 10...24) {
                    Text("ขนาดตัวอักษร: \(Int(fontSize))")
                }
                Toggle("Dark Mode", isOn: $darkMode)
            }
        }
        .padding()
        .frame(width: 350, height: 200)
    }
}
```

---

## View protocol

View protocol คือ building block พื้นฐานของ SwiftUI ทุก UI element ต้อง conform กับ View

### ความต้องการขั้นต่ำ

```swift
public protocol View {
    associatedtype Body: View
    
    @ViewBuilder
    var body: Self.Body { get }
}
```

### การสร้าง Custom View

```swift
// View ง่ายๆ
struct GreetingView: View {
    let name: String
    
    var body: some View {
        Text("สวัสดี, \(name)!")
            .font(.title)
            .foregroundColor(.blue)
    }
}

// View ที่มี state
struct CounterView: View {
    @State private var count = 0
    
    var body: some View {
        VStack {
            Text("Count: \(count)")
                .font(.largeTitle)
            
            HStack {
                Button("ลด") {
                    count -= 1
                }
                .buttonStyle(.bordered)
                
                Button("เพิ่ม") {
                    count += 1
                }
                .buttonStyle(.bordered)
            }
        }
    }
}

// View ที่รับ parameter
struct ProductCard: View {
    let name: String
    let price: Double
    let image: String
    
    var body: some View {
        VStack(alignment: .leading) {
            Image(systemName: image)
                .resizable()
                .aspectRatio(contentMode: .fit)
                .frame(height: 100)
            
            Text(name)
                .font(.headline)
            
            Text("฿\(price, specifier: "%.2f")")
                .font(.subheadline)
                .foregroundColor(.green)
        }
        .padding()
        .background(Color.white)
        .cornerRadius(12)
        .shadow(radius: 4)
    }
}
```

---

## body property

`body` property คือหัวใจของ View ทุกอัน มันบอกว่า View นั้นประกอบด้วยอะไร

### กฎของ body

1. ต้องเป็น `var` ไม่ใช่ `let`
2. Return type ใช้ `some View` (opaque type)
3. ต้อง return View เพียงหนึ่ง root View
4. ใช้ `@ViewBuilder` โดยอัตโนมัติ

```swift
struct ExampleView: View {
    
    // body ต้องคืนค่า View เดียว
    var body: some View {
        // VStack คือ root view ของเรา
        VStack(spacing: 16) {
            topSection
            middleSection
            bottomSection
        }
        .padding()
    }
    
    // แยก computed properties ออกมาเพื่อความเป็นระเบียบ
    private var topSection: some View {
        Text("หัวข้อ")
            .font(.title)
            .bold()
    }
    
    private var middleSection: some View {
        Text("เนื้อหา")
            .font(.body)
    }
    
    private var bottomSection: some View {
        Button("ปุ่ม") {
            // action
        }
    }
}
```

### some View vs AnyView

```swift
struct FlexibleView: View {
    let showImage: Bool
    
    // ใช้ some View - ดีกว่า, ประสิทธิภาพดีกว่า
    var body: some View {
        Group {
            if showImage {
                Image(systemName: "star")
            } else {
                Text("No Image")
            }
        }
    }
    
    // ใช้ AnyView - ควรหลีกเลี่ยง ถ้าไม่จำเป็น
    var alternativeBody: AnyView {
        if showImage {
            return AnyView(Image(systemName: "star"))
        } else {
            return AnyView(Text("No Image"))
        }
    }
}
```

---

## Basic Views พื้นฐาน

### Text

Text view สำหรับแสดงข้อความ

```swift
struct TextExamples: View {
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            // ข้อความธรรมดา
            Text("Hello, SwiftUI!")
            
            // Font styles
            Text("Title").font(.title)
            Text("Title2").font(.title2)
            Text("Title3").font(.title3)
            Text("Headline").font(.headline)
            Text("Subheadline").font(.subheadline)
            Text("Body").font(.body)
            Text("Callout").font(.callout)
            Text("Caption").font(.caption)
            Text("Caption2").font(.caption2)
            Text("Footnote").font(.footnote)
            
            // Custom font
            Text("Custom Font")
                .font(.system(size: 20, weight: .bold, design: .rounded))
            
            // สีตัวอักษร
            Text("สีน้ำเงิน").foregroundColor(.blue)
            Text("สีแดง").foregroundStyle(.red)  // iOS 17+
            
            // Bold, Italic
            Text("หนา").bold()
            Text("เอียง").italic()
            Text("หนาและเอียง").bold().italic()
            
            // Underline, Strikethrough
            Text("ขีดเส้นใต้").underline()
            Text("ขีดทับ").strikethrough()
            
            // Multi-line
            Text("ข้อความยาวๆ ที่ต้องการให้ขึ้นบรรทัดใหม่อัตโนมัติ")
                .multilineTextAlignment(.center)
                .lineLimit(3)
            
            // Localized string
            Text("ยินดีต้อนรับ")
            
            // String interpolation
            let count = 42
            Text("มีสินค้า \(count) รายการ")
            
            // Formatted numbers
            let price = 1234.56
            Text("ราคา: \(price, format: .currency(code: "THB"))")
            
            // Attributed string (iOS 15+)
            Text("**หนา** และ *เอียง* และ ~~ขีดทับ~~")
        }
        .padding()
    }
}
```

### Image

```swift
struct ImageExamples: View {
    var body: some View {
        VStack(spacing: 20) {
            // SF Symbols
            Image(systemName: "heart.fill")
                .foregroundColor(.red)
            
            // SF Symbol ขนาดต่างๆ
            Image(systemName: "star.fill")
                .imageScale(.small)
            
            Image(systemName: "star.fill")
                .imageScale(.medium)
            
            Image(systemName: "star.fill")
                .imageScale(.large)
            
            // Font size สำหรับ SF Symbol
            Image(systemName: "bell")
                .font(.system(size: 40))
            
            // รูปจาก Assets
            Image("myPhoto")
                .resizable()
                .aspectRatio(contentMode: .fit)
                .frame(width: 200, height: 200)
            
            // รูปจาก Assets - fill mode
            Image("myPhoto")
                .resizable()
                .aspectRatio(contentMode: .fill)
                .frame(width: 100, height: 100)
                .clipShape(Circle())
            
            // รูปจาก URL (ต้องใช้ AsyncImage)
            AsyncImage(url: URL(string: "https://example.com/photo.jpg")) { image in
                image
                    .resizable()
                    .aspectRatio(contentMode: .fit)
            } placeholder: {
                ProgressView()
            }
            .frame(width: 200, height: 200)
        }
    }
}
```

### Button

```swift
struct ButtonExamples: View {
    @State private var message = ""
    
    var body: some View {
        VStack(spacing: 16) {
            // Button ธรรมดา
            Button("กดฉัน") {
                message = "ถูกกดแล้ว!"
            }
            
            // Button พร้อม label
            Button {
                message = "Custom button กด"
            } label: {
                HStack {
                    Image(systemName: "star.fill")
                    Text("Favorite")
                }
                .padding()
                .background(Color.yellow)
                .foregroundColor(.black)
                .cornerRadius(10)
            }
            
            // Button styles
            Button("Bordered") { }
                .buttonStyle(.bordered)
            
            Button("Bordered Prominent") { }
                .buttonStyle(.borderedProminent)
            
            Button("Plain") { }
                .buttonStyle(.plain)
            
            // Destructive button
            Button("ลบ", role: .destructive) {
                message = "กด Delete!"
            }
            
            // Cancel button
            Button("ยกเลิก", role: .cancel) {
                message = "กด Cancel"
            }
            
            // Disabled button
            Button("ปิดใช้งาน") { }
                .disabled(true)
            
            Text(message)
                .font(.caption)
        }
        .padding()
    }
}
```

### TextField

```swift
struct TextFieldExamples: View {
    @State private var name = ""
    @State private var email = ""
    @State private var password = ""
    @State private var age = ""
    @State private var bio = ""
    
    var body: some View {
        Form {
            Section("ข้อมูลส่วนตัว") {
                // TextField พื้นฐาน
                TextField("ชื่อ", text: $name)
                
                // TextField พร้อม label แยก
                TextField(text: $email) {
                    Text("อีเมล")
                }
                .textInputAutocapitalization(.never)
                .keyboardType(.emailAddress)
                
                // SecureField สำหรับรหัสผ่าน
                SecureField("รหัสผ่าน", text: $password)
                
                // TextField ตัวเลข
                TextField("อายุ", text: $age)
                    .keyboardType(.numberPad)
                
                // TextEditor สำหรับข้อความยาว
                TextEditor(text: $bio)
                    .frame(minHeight: 100)
            }
            
            Section {
                Button("บันทึก") {
                    saveData()
                }
            }
        }
    }
    
    private func saveData() {
        print("Name: \(name), Email: \(email)")
    }
}
```

### Toggle

```swift
struct ToggleExamples: View {
    @State private var isOn = false
    @State private var notifications = true
    @State private var darkMode = false
    
    var body: some View {
        Form {
            Section("การตั้งค่า") {
                // Toggle พื้นฐาน
                Toggle("เปิด/ปิด", isOn: $isOn)
                
                // Toggle พร้อม description
                Toggle(isOn: $notifications) {
                    VStack(alignment: .leading) {
                        Text("การแจ้งเตือน")
                        Text("รับการแจ้งเตือนจากแอป")
                            .font(.caption)
                            .foregroundColor(.gray)
                    }
                }
                
                Toggle("Dark Mode", isOn: $darkMode)
            }
            
            Section("สถานะ") {
                Text("สถานะ: \(isOn ? "เปิด" : "ปิด")")
                Text("การแจ้งเตือน: \(notifications ? "เปิด" : "ปิด")")
            }
        }
    }
}
```

### Slider

```swift
struct SliderExamples: View {
    @State private var volume: Double = 50
    @State private var brightness: Double = 0.7
    @State private var fontSize: Double = 16
    
    var body: some View {
        VStack(spacing: 24) {
            // Slider พื้นฐาน
            VStack(alignment: .leading) {
                Text("ระดับเสียง: \(Int(volume))")
                Slider(value: $volume, in: 0...100)
            }
            
            // Slider พร้อม step
            VStack(alignment: .leading) {
                Text("ขนาดตัวอักษร: \(Int(fontSize))")
                Slider(value: $fontSize, in: 10...30, step: 1)
            }
            
            // Slider พร้อม label และ onEditingChanged
            VStack(alignment: .leading) {
                Text("ความสว่าง: \(String(format: "%.0f%%", brightness * 100))")
                Slider(
                    value: $brightness,
                    in: 0...1
                ) {
                    Text("Brightness")
                } minimumValueLabel: {
                    Image(systemName: "sun.min")
                } maximumValueLabel: {
                    Image(systemName: "sun.max")
                } onEditingChanged: { editing in
                    if !editing {
                        print("Brightness set to: \(brightness)")
                    }
                }
            }
            
            // ตัวอย่างการใช้งาน
            Text("ตัวอย่างข้อความ")
                .font(.system(size: fontSize))
        }
        .padding()
    }
}
```

---

## View Modifiers

View Modifier คือ function ที่ปรับแต่ง View และคืนค่า View ใหม่

### Built-in Modifiers ที่ใช้บ่อย

```swift
struct ModifierExamples: View {
    var body: some View {
        VStack(spacing: 20) {
            
            // Styling
            Text("Styled Text")
                .font(.title)
                .bold()
                .italic()
                .foregroundColor(.purple)
                .background(Color.yellow.opacity(0.3))
                .cornerRadius(8)
                .padding()
                .shadow(color: .gray, radius: 4, x: 2, y: 2)
            
            // Layout
            Text("Layout")
                .frame(width: 200, height: 50)
                .background(Color.blue)
                .foregroundColor(.white)
                .padding(.horizontal, 20)
                .padding(.vertical, 10)
            
            // Interaction
            Text("Interaction")
                .onTapGesture {
                    print("Tapped!")
                }
                .onLongPressGesture {
                    print("Long pressed!")
                }
            
            // Accessibility
            Text("Accessible")
                .accessibilityLabel("ข้อความที่อ่านออกเสียงได้")
                .accessibilityHint("กดเพื่อดูรายละเอียด")
        }
    }
}
```

### Modifier Categories

```swift
struct ModifierCategories: View {
    var body: some View {
        Text("SwiftUI Modifiers")
        
        // 1. Text Modifiers
            .font(.title)
            .fontWeight(.semibold)
            .fontDesign(.rounded)           // iOS 16+
            .kerning(2)                      // ระยะห่างตัวอักษร
            .tracking(1)                     // ระยะห่างตัวอักษร (แตกต่างจาก kerning)
            .baselineOffset(5)               // ปรับ baseline
            .strikethrough(true, color: .red)
            .underline(true, color: .blue)
        
        // 2. Color Modifiers
            .foregroundStyle(.primary)
            .background(Color.yellow)
            .tint(.blue)
        
        // 3. Layout Modifiers
            .padding()
            .frame(maxWidth: .infinity)
            .fixedSize()
            .layoutPriority(1)
        
        // 4. Visual Effects
            .opacity(0.8)
            .blur(radius: 2)
            .brightness(0.1)
            .contrast(1.2)
            .saturation(0.5)
            .grayscale(0.3)
            .colorInvert()
        
        // 5. Shape Modifiers
            .clipShape(RoundedRectangle(cornerRadius: 10))
            .cornerRadius(10)               // deprecated ใน iOS 17
            .overlay(Color.clear)
        
        // 6. Animation
            .animation(.easeInOut, value: true)
            .transition(.slide)
    }
}
```

---

## ลำดับของ Modifier สำคัญ

ลำดับที่เราใช้ Modifier ส่งผลต่อผลลัพธ์อย่างมาก

### ตัวอย่างที่แสดงความสำคัญของลำดับ

```swift
struct ModifierOrderExample: View {
    var body: some View {
        VStack(spacing: 30) {
            
            // ตัวอย่าง 1: padding แล้วค่อย background
            // background จะครอบทั้งพื้นที่รวม padding
            Text("Padding → Background")
                .padding()
                .background(Color.blue)
                .foregroundColor(.white)
            
            // ตัวอย่าง 1 แบบสลับ: background แล้วค่อย padding
            // background จะครอบแค่ตัวข้อความ ไม่รวม padding
            Text("Background → Padding")
                .background(Color.blue)
                .foregroundColor(.white)
                .padding()
            
            Divider()
            
            // ตัวอย่าง 2: cornerRadius กับ shadow
            // ถ้า cornerRadius ก่อน shadow จะมี radius ด้วย
            Text("CorrectOrder")
                .padding()
                .background(Color.white)
                .cornerRadius(10)
                .shadow(radius: 5)
            
            // shadow ก่อน cornerRadius - shadow จะเป็นรูปสี่เหลี่ยมตรง
            Text("WrongOrder")
                .padding()
                .background(Color.white)
                .shadow(radius: 5)
                .cornerRadius(10)
            
            Divider()
            
            // ตัวอย่าง 3: frame
            // frame ควรใช้ก่อน padding เพื่อกำหนดขนาดที่แน่นอน
            Text("Frame then Padding")
                .frame(width: 150, height: 50)
                .background(Color.green)
                .padding()
        }
        .padding()
    }
}
```

### ทำไมลำดับจึงสำคัญ

```swift
// SwiftUI ทำงานแบบ wrapping
// แต่ละ modifier สร้าง View ใหม่ห่อหุ้ม View เดิม

Text("Hello")
    .padding()           // สร้าง _PaddedView(Text("Hello"))
    .background(.blue)   // สร้าง _BackgroundView(_PaddedView(Text("Hello")))
    .foregroundColor(.white)  // สร้าง _ForegroundColorView(_BackgroundView(...))

// เทียบกับ
Text("Hello")
    .background(.blue)   // สร้าง _BackgroundView(Text("Hello"))
    .padding()           // สร้าง _PaddedView(_BackgroundView(Text("Hello")))
    .foregroundColor(.white)
```

---

## การ Chain Modifiers

การต่อ modifier หลายๆ ตัวเข้าด้วยกัน

```swift
struct ChainedModifiers: View {
    var body: some View {
        VStack(spacing: 20) {
            
            // Basic chaining
            Text("Chained Modifiers")
                .font(.headline)
                .bold()
                .italic()
                .foregroundColor(.white)
                .padding(.horizontal, 20)
                .padding(.vertical, 10)
                .background(
                    LinearGradient(
                        colors: [.blue, .purple],
                        startPoint: .leading,
                        endPoint: .trailing
                    )
                )
                .cornerRadius(25)
                .shadow(color: .purple.opacity(0.3), radius: 10, x: 0, y: 5)
            
            // Button with chained modifiers
            Button("สมัครสมาชิก") {
                // action
            }
            .font(.headline)
            .foregroundColor(.white)
            .frame(maxWidth: .infinity)
            .padding()
            .background(Color.blue)
            .cornerRadius(15)
            .padding(.horizontal)
            .shadow(radius: 5)
        }
    }
}
```

---

## Custom Modifiers

เราสามารถสร้าง modifier ของเราเองได้โดย conform กับ `ViewModifier` protocol

```swift
// สร้าง Custom Modifier
struct CardStyle: ViewModifier {
    var backgroundColor: Color = .white
    var cornerRadius: CGFloat = 12
    var shadowRadius: CGFloat = 4
    
    func body(content: Content) -> some View {
        content
            .padding()
            .background(backgroundColor)
            .cornerRadius(cornerRadius)
            .shadow(color: .black.opacity(0.1), radius: shadowRadius, x: 0, y: 2)
    }
}

// Extension เพื่อให้ใช้งานง่ายขึ้น
extension View {
    func cardStyle(
        backgroundColor: Color = .white,
        cornerRadius: CGFloat = 12
    ) -> some View {
        modifier(CardStyle(
            backgroundColor: backgroundColor,
            cornerRadius: cornerRadius
        ))
    }
}

// การใช้งาน
struct CustomModifierExample: View {
    var body: some View {
        VStack(spacing: 16) {
            Text("Card 1")
                .cardStyle()
            
            Text("Card 2")
                .cardStyle(backgroundColor: .blue.opacity(0.1), cornerRadius: 20)
            
            // หรือใช้ modifier() โดยตรง
            Text("Card 3")
                .modifier(CardStyle(backgroundColor: .green.opacity(0.1)))
        }
        .padding()
        .background(Color.gray.opacity(0.1))
    }
}

// ตัวอย่างที่ซับซ้อนขึ้น
struct GlowEffect: ViewModifier {
    let color: Color
    let radius: CGFloat
    
    func body(content: Content) -> some View {
        content
            .shadow(color: color, radius: radius / 3)
            .shadow(color: color, radius: radius / 3)
            .shadow(color: color, radius: radius / 3)
    }
}

extension View {
    func glowEffect(color: Color, radius: CGFloat = 10) -> some View {
        modifier(GlowEffect(color: color, radius: radius))
    }
}

struct GlowExample: View {
    var body: some View {
        Text("✨ Glow Effect")
            .font(.title)
            .foregroundColor(.white)
            .padding()
            .background(Color.black)
            .glowEffect(color: .blue, radius: 20)
    }
}
```

---

## ViewBuilder

`@ViewBuilder` คือ result builder ที่ช่วยให้เราสร้าง View จากหลายๆ View โดยไม่ต้องใช้ `return`

### การทำงานของ ViewBuilder

```swift
// ViewBuilder อนุญาตให้เขียนแบบนี้
var body: some View {
    Text("First")
    Text("Second")
    Text("Third")
}

// แทนที่จะต้องเขียนแบบนี้
var body: some View {
    return TupleView((Text("First"), Text("Second"), Text("Third")))
}
```

### การใช้ @ViewBuilder ใน Custom Functions

```swift
struct ViewBuilderExamples: View {
    
    var body: some View {
        VStack {
            customContent(for: "iOS Developer") {
                Image(systemName: "iphone")
                    .font(.title)
                Text("สร้าง iOS Apps")
            }
            
            customContent(for: "macOS Developer") {
                Image(systemName: "desktopcomputer")
                    .font(.title)
                Text("สร้าง macOS Apps")
            }
        }
    }
    
    // Function ที่ใช้ @ViewBuilder
    func customContent<Content: View>(
        for title: String,
        @ViewBuilder content: () -> Content
    ) -> some View {
        VStack {
            Text(title)
                .font(.headline)
            content()
        }
        .padding()
        .background(Color.blue.opacity(0.1))
        .cornerRadius(10)
    }
}

// Container View ที่ใช้ @ViewBuilder
struct CustomContainer<Content: View>: View {
    let title: String
    let content: Content
    
    init(title: String, @ViewBuilder content: () -> Content) {
        self.title = title
        self.content = content()
    }
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            Text(title)
                .font(.headline)
                .padding(.bottom, 4)
            
            Divider()
            
            content
        }
        .padding()
        .background(Color.white)
        .cornerRadius(12)
        .shadow(radius: 4)
    }
}

// การใช้งาน
struct ContainerUsage: View {
    var body: some View {
        CustomContainer(title: "ข้อมูลผู้ใช้") {
            Label("ชื่อ: สมชาย", systemImage: "person")
            Label("อีเมล: somchai@example.com", systemImage: "envelope")
            Label("โทร: 081-234-5678", systemImage: "phone")
        }
        .padding()
    }
}
```

---

## Previews

Previews ช่วยให้เราดู UI ใน Xcode Canvas โดยไม่ต้อง run simulator

### #Preview (iOS 17+, Xcode 15+)

```swift
// Preview พื้นฐาน
#Preview {
    ContentView()
}

// Preview พร้อมชื่อ
#Preview("Home Screen") {
    HomeView()
}

// Preview พร้อม environment
#Preview {
    ContentView()
        .environment(\.colorScheme, .dark)
}

// Preview หลายอัน
#Preview("Light Mode") {
    ContentView()
        .environment(\.colorScheme, .light)
}

#Preview("Dark Mode") {
    ContentView()
        .environment(\.colorScheme, .dark)
}

// Preview บน device ต่างๆ
#Preview("iPhone 15") {
    ContentView()
}

// Preview Widget
#Preview(as: .systemSmall) {
    MyWidget()
} timeline: {
    SimpleEntry(date: .now)
}
```

### PreviewProvider (iOS 13-16 syntax เก่า)

```swift
struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}

// Preview หลายอัน
struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        // Light mode
        ContentView()
            .previewDisplayName("Light Mode")
        
        // Dark mode
        ContentView()
            .environment(\.colorScheme, .dark)
            .previewDisplayName("Dark Mode")
        
        // iPad
        ContentView()
            .previewDevice(PreviewDevice(rawValue: "iPad Pro (12.9-inch) (6th generation)"))
            .previewDisplayName("iPad")
        
        // iPhone SE (เล็ก)
        ContentView()
            .previewDevice(PreviewDevice(rawValue: "iPhone SE (3rd generation)"))
            .previewDisplayName("iPhone SE")
    }
}
```

### Preview พร้อม Mock Data

```swift
struct UserProfileView: View {
    let user: User
    
    var body: some View {
        VStack {
            Image(systemName: "person.circle.fill")
                .font(.system(size: 80))
            Text(user.name)
                .font(.title)
            Text(user.email)
                .font(.caption)
                .foregroundColor(.gray)
        }
    }
}

// Model
struct User {
    let name: String
    let email: String
    let isPremium: Bool
    
    // Mock data สำหรับ Preview
    static let preview = User(
        name: "สมชาย ใจดี",
        email: "somchai@example.com",
        isPremium: true
    )
    
    static let mockUsers = [
        User(name: "สมชาย ใจดี", email: "somchai@example.com", isPremium: true),
        User(name: "สมหญิง รักดี", email: "somying@example.com", isPremium: false),
        User(name: "มานะ ขยันดี", email: "mana@example.com", isPremium: true)
    ]
}

#Preview("User Profile") {
    UserProfileView(user: .preview)
}

#Preview("User List") {
    List(User.mockUsers, id: \.name) { user in
        Text(user.name)
    }
}
```

### Dark Mode Preview

```swift
// วิธีที่ 1: ใช้ environment
#Preview("Dark Mode") {
    ContentView()
        .preferredColorScheme(.dark)
}

// วิธีที่ 2: ใช้ environment value
#Preview {
    ContentView()
        .environment(\.colorScheme, .dark)
}

// วิธีที่ 3: ดู Light และ Dark พร้อมกัน
struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ForEach(ColorScheme.allCases, id: \.self) { scheme in
            ContentView()
                .preferredColorScheme(scheme)
                .previewDisplayName("\(scheme == .dark ? "Dark" : "Light") Mode")
        }
    }
}
```

---

## Environment Values

Environment คือ collection ของ values ที่ส่งผ่าน View hierarchy โดยอัตโนมัติ

### Built-in Environment Values

```swift
struct EnvironmentExamples: View {
    // ค่า environment ที่ใช้บ่อย
    @Environment(\.colorScheme) private var colorScheme
    @Environment(\.dynamicTypeSize) private var typeSize
    @Environment(\.horizontalSizeClass) private var hSizeClass
    @Environment(\.verticalSizeClass) private var vSizeClass
    @Environment(\.locale) private var locale
    @Environment(\.calendar) private var calendar
    @Environment(\.timeZone) private var timeZone
    @Environment(\.font) private var font
    @Environment(\.isEnabled) private var isEnabled
    @Environment(\.dismiss) private var dismiss
    @Environment(\.openURL) private var openURL
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            Text("Color Scheme: \(colorScheme == .dark ? "Dark" : "Light")")
            Text("Size Class: \(hSizeClass == .compact ? "Compact" : "Regular")")
            Text("Locale: \(locale.identifier)")
            Text("Time Zone: \(timeZone.identifier)")
            Text("Is Enabled: \(isEnabled)")
            
            // ปรับ UI ตาม color scheme
            Text("Adaptive Color")
                .foregroundColor(colorScheme == .dark ? .white : .black)
                .background(colorScheme == .dark ? Color.black : Color.white)
            
            // ปรับ layout ตาม size class
            if hSizeClass == .compact {
                // iPhone layout
                VStack {
                    Text("Compact layout")
                }
            } else {
                // iPad layout
                HStack {
                    Text("Regular layout")
                }
            }
        }
        .padding()
    }
}
```

---

## @Environment Property Wrapper

`@Environment` ใช้เพื่อดึงค่าจาก environment

```swift
// การใช้งานพื้นฐาน
struct MyView: View {
    @Environment(\.colorScheme) private var colorScheme
    @Environment(\.dismiss) private var dismiss
    
    var body: some View {
        VStack {
            Text("Current theme: \(colorScheme == .dark ? "Dark" : "Light")")
            
            Button("ปิด") {
                dismiss()
            }
        }
    }
}

// Custom Environment Key
struct ThemeKey: EnvironmentKey {
    static let defaultValue: AppTheme = .default
}

enum AppTheme {
    case `default`
    case blue
    case green
    
    var primaryColor: Color {
        switch self {
        case .default: return .blue
        case .blue: return .blue
        case .green: return .green
        }
    }
}

extension EnvironmentValues {
    var appTheme: AppTheme {
        get { self[ThemeKey.self] }
        set { self[ThemeKey.self] = newValue }
    }
}

// การใช้ Custom Environment
struct ThemeAwareView: View {
    @Environment(\.appTheme) private var theme
    
    var body: some View {
        Text("Themed Text")
            .foregroundColor(theme.primaryColor)
    }
}

// การส่ง Custom Environment
struct AppView: View {
    var body: some View {
        ThemeAwareView()
            .environment(\.appTheme, .green)
    }
}
```

---

## Layout พื้นฐาน

### HStack, VStack, ZStack

```swift
struct StackLayouts: View {
    var body: some View {
        VStack(spacing: 30) {
            
            // HStack - จัดเรียงแนวนอน
            HStack(alignment: .center, spacing: 20) {
                Text("ซ้าย")
                    .frame(maxWidth: .infinity)
                    .padding()
                    .background(Color.red.opacity(0.2))
                
                Text("กลาง")
                    .frame(maxWidth: .infinity)
                    .padding()
                    .background(Color.green.opacity(0.2))
                
                Text("ขวา")
                    .frame(maxWidth: .infinity)
                    .padding()
                    .background(Color.blue.opacity(0.2))
            }
            
            // VStack - จัดเรียงแนวตั้ง
            VStack(alignment: .leading, spacing: 10) {
                Text("บน")
                    .frame(maxWidth: .infinity, alignment: .leading)
                    .padding()
                    .background(Color.red.opacity(0.2))
                
                Text("กลาง")
                    .frame(maxWidth: .infinity, alignment: .leading)
                    .padding()
                    .background(Color.green.opacity(0.2))
                
                Text("ล่าง")
                    .frame(maxWidth: .infinity, alignment: .leading)
                    .padding()
                    .background(Color.blue.opacity(0.2))
            }
            
            // ZStack - ซ้อนทับกัน
            ZStack {
                // ชั้นล่างสุด
                RoundedRectangle(cornerRadius: 12)
                    .fill(Color.blue)
                    .frame(width: 200, height: 100)
                
                // ชั้นกลาง
                RoundedRectangle(cornerRadius: 12)
                    .fill(Color.green.opacity(0.5))
                    .frame(width: 180, height: 80)
                
                // ชั้นบนสุด
                Text("ZStack")
                    .font(.headline)
                    .foregroundColor(.white)
            }
        }
        .padding()
    }
}
```

### Alignment ใน Stack

```swift
struct AlignmentExamples: View {
    var body: some View {
        VStack(spacing: 30) {
            
            // HStack alignment
            Group {
                HStack(alignment: .top) {
                    Text("Top\nAlignment")
                        .padding()
                        .background(Color.blue.opacity(0.2))
                    
                    Text("Short")
                        .padding()
                        .background(Color.red.opacity(0.2))
                }
                
                HStack(alignment: .center) {
                    Text("Center\nAlignment")
                        .padding()
                        .background(Color.blue.opacity(0.2))
                    
                    Text("Short")
                        .padding()
                        .background(Color.red.opacity(0.2))
                }
                
                HStack(alignment: .bottom) {
                    Text("Bottom\nAlignment")
                        .padding()
                        .background(Color.blue.opacity(0.2))
                    
                    Text("Short")
                        .padding()
                        .background(Color.red.opacity(0.2))
                }
                
                // firstTextBaseline alignment
                HStack(alignment: .firstTextBaseline) {
                    Text("Large")
                        .font(.title)
                    Text("Small")
                        .font(.caption)
                }
            }
        }
        .padding()
    }
}
```

---

## Spacer และ Divider

### Spacer

```swift
struct SpacerExamples: View {
    var body: some View {
        VStack(spacing: 20) {
            
            // Spacer ดันให้ Views ไปชิดขอบ
            HStack {
                Text("ซ้าย")
                Spacer()
                Text("ขวา")
            }
            .padding()
            .background(Color.gray.opacity(0.2))
            
            // Spacer หลายอัน - แบ่งพื้นที่เท่าๆ กัน
            HStack {
                Text("A")
                Spacer()
                Text("B")
                Spacer()
                Text("C")
            }
            .padding()
            .background(Color.gray.opacity(0.2))
            
            // Spacer กำหนดขนาดขั้นต่ำ
            HStack {
                Text("Left")
                Spacer(minLength: 50)
                Text("Right")
            }
            .padding()
            .background(Color.gray.opacity(0.2))
            
            // VStack กับ Spacer
            VStack {
                Text("ข้อความด้านบน")
                
                Spacer()
                
                Text("ข้อความด้านล่าง")
            }
            .frame(height: 150)
            .padding()
            .background(Color.blue.opacity(0.1))
            .cornerRadius(10)
        }
        .padding()
    }
}
```

### Divider

```swift
struct DividerExamples: View {
    var body: some View {
        VStack(spacing: 0) {
            
            // Divider พื้นฐาน
            Text("ส่วนที่ 1")
                .padding()
            
            Divider()
            
            Text("ส่วนที่ 2")
                .padding()
            
            Divider()
                .background(Color.blue)  // เปลี่ยนสี
            
            Text("ส่วนที่ 3")
                .padding()
            
            // Divider ใน HStack (แนวตั้ง)
            HStack(spacing: 0) {
                Text("ซ้าย")
                    .frame(maxWidth: .infinity)
                    .padding()
                
                Divider()
                
                Text("ขวา")
                    .frame(maxWidth: .infinity)
                    .padding()
            }
            .frame(height: 50)
        }
        .background(Color.white)
        .cornerRadius(12)
        .shadow(radius: 4)
        .padding()
    }
}
```

---

## Group

Group ช่วยจัดกลุ่ม Views โดยไม่เพิ่ม visual container

```swift
struct GroupExamples: View {
    @State private var showDetails = true
    
    var body: some View {
        VStack {
            
            // Group สำหรับ apply modifier ให้หลาย Views
            Group {
                Text("ข้อความ 1")
                Text("ข้อความ 2")
                Text("ข้อความ 3")
            }
            .font(.headline)
            .foregroundColor(.blue)
            .padding(.vertical, 4)
            
            // Group สำหรับ conditional display
            if showDetails {
                Group {
                    Text("รายละเอียด 1")
                    Text("รายละเอียด 2")
                    Text("รายละเอียด 3")
                }
                .padding()
                .background(Color.gray.opacity(0.1))
                .cornerRadius(8)
            }
            
            Button("Toggle Details") {
                withAnimation {
                    showDetails.toggle()
                }
            }
            
            // Group เพื่อแก้ปัญหา 10-child limit ใน ViewBuilder
            VStack {
                Group {
                    Text("1")
                    Text("2")
                    Text("3")
                    Text("4")
                    Text("5")
                    Text("6")
                    Text("7")
                    Text("8")
                    Text("9")
                    Text("10")
                }
                Group {
                    Text("11")
                    Text("12")
                }
            }
        }
        .padding()
    }
}
```

---

## แบบฝึกหัดพร้อมเฉลย

### แบบฝึกหัดที่ 1: สร้าง Profile Card

สร้าง View ที่แสดงข้อมูลโปรไฟล์ผู้ใช้

**โจทย์:**
สร้าง ProfileCardView ที่มี:
- รูปโปรไฟล์ (ใช้ SF Symbol)
- ชื่อผู้ใช้
- ตำแหน่งงาน
- จำนวน followers และ following
- ปุ่ม Follow

**เฉลย:**

```swift
struct ProfileCardView: View {
    let username: String
    let jobTitle: String
    let followers: Int
    let following: Int
    
    @State private var isFollowing = false
    
    var body: some View {
        VStack(spacing: 16) {
            // รูปโปรไฟล์
            ZStack(alignment: .bottomTrailing) {
                Image(systemName: "person.circle.fill")
                    .resizable()
                    .aspectRatio(contentMode: .fit)
                    .frame(width: 100, height: 100)
                    .foregroundColor(.blue)
                    .background(Color.blue.opacity(0.1))
                    .clipShape(Circle())
                
                // Badge
                Circle()
                    .fill(Color.green)
                    .frame(width: 20, height: 20)
                    .overlay(
                        Circle()
                            .stroke(Color.white, lineWidth: 2)
                    )
            }
            
            // ชื่อและตำแหน่ง
            VStack(spacing: 4) {
                Text(username)
                    .font(.title2)
                    .fontWeight(.bold)
                
                Text(jobTitle)
                    .font(.subheadline)
                    .foregroundColor(.gray)
            }
            
            // Stats
            HStack(spacing: 40) {
                VStack {
                    Text("\(followers)")
                        .font(.title3)
                        .fontWeight(.bold)
                    Text("Followers")
                        .font(.caption)
                        .foregroundColor(.gray)
                }
                
                Divider()
                    .frame(height: 40)
                
                VStack {
                    Text("\(following)")
                        .font(.title3)
                        .fontWeight(.bold)
                    Text("Following")
                        .font(.caption)
                        .foregroundColor(.gray)
                }
            }
            
            // Follow Button
            Button {
                withAnimation(.spring()) {
                    isFollowing.toggle()
                }
            } label: {
                Text(isFollowing ? "Following" : "Follow")
                    .font(.headline)
                    .foregroundColor(isFollowing ? .gray : .white)
                    .frame(maxWidth: .infinity)
                    .padding(.vertical, 12)
                    .background(isFollowing ? Color.gray.opacity(0.2) : Color.blue)
                    .cornerRadius(12)
                    .overlay(
                        RoundedRectangle(cornerRadius: 12)
                            .stroke(isFollowing ? Color.gray : Color.clear, lineWidth: 1)
                    )
            }
        }
        .padding(24)
        .background(Color.white)
        .cornerRadius(20)
        .shadow(color: .black.opacity(0.1), radius: 10, x: 0, y: 5)
        .padding(.horizontal)
    }
}

#Preview("Profile Card") {
    ZStack {
        Color.gray.opacity(0.15)
            .ignoresSafeArea()
        
        ProfileCardView(
            username: "สมชาย ใจดี",
            jobTitle: "iOS Developer",
            followers: 1234,
            following: 567
        )
    }
}
```

### แบบฝึกหัดที่ 2: Weather App UI

```swift
// Weather Data Model
struct WeatherData {
    let city: String
    let temperature: Int
    let condition: String
    let icon: String
    let high: Int
    let low: Int
    let humidity: Int
    let windSpeed: Int
    
    static let preview = WeatherData(
        city: "กรุงเทพมหานคร",
        temperature: 32,
        condition: "มีเมฆบางส่วน",
        icon: "cloud.sun.fill",
        high: 35,
        low: 27,
        humidity: 75,
        windSpeed: 15
    )
}

// Weather Card View
struct WeatherCardView: View {
    let weather: WeatherData
    
    var body: some View {
        ZStack {
            // Gradient Background
            LinearGradient(
                colors: [Color(hex: "4facfe"), Color(hex: "00f2fe")],
                startPoint: .topLeading,
                endPoint: .bottomTrailing
            )
            .cornerRadius(24)
            
            VStack(alignment: .leading, spacing: 0) {
                // Header
                HStack {
                    VStack(alignment: .leading) {
                        Text(weather.city)
                            .font(.title2)
                            .fontWeight(.semibold)
                            .foregroundColor(.white)
                        
                        Text(Date.now, format: .dateTime.weekday(.wide).month().day())
                            .font(.caption)
                            .foregroundColor(.white.opacity(0.8))
                    }
                    
                    Spacer()
                    
                    Image(systemName: weather.icon)
                        .font(.system(size: 50))
                        .foregroundStyle(.white, .yellow)
                }
                .padding()
                
                // Temperature
                HStack(alignment: .top) {
                    Text("\(weather.temperature)")
                        .font(.system(size: 80, weight: .thin))
                        .foregroundColor(.white)
                    
                    Text("°C")
                        .font(.title)
                        .foregroundColor(.white.opacity(0.8))
                        .padding(.top, 16)
                }
                .padding(.horizontal)
                
                Text(weather.condition)
                    .font(.headline)
                    .foregroundColor(.white.opacity(0.9))
                    .padding(.horizontal)
                    .padding(.bottom, 8)
                
                Divider()
                    .background(Color.white.opacity(0.3))
                    .padding(.horizontal)
                
                // Stats
                HStack {
                    WeatherStatView(
                        icon: "thermometer.high",
                        value: "\(weather.high)°",
                        label: "สูงสุด"
                    )
                    
                    Spacer()
                    
                    WeatherStatView(
                        icon: "thermometer.low",
                        value: "\(weather.low)°",
                        label: "ต่ำสุด"
                    )
                    
                    Spacer()
                    
                    WeatherStatView(
                        icon: "humidity",
                        value: "\(weather.humidity)%",
                        label: "ความชื้น"
                    )
                    
                    Spacer()
                    
                    WeatherStatView(
                        icon: "wind",
                        value: "\(weather.windSpeed)",
                        label: "กม./ชม."
                    )
                }
                .padding()
            }
        }
        .padding(.horizontal)
    }
}

struct WeatherStatView: View {
    let icon: String
    let value: String
    let label: String
    
    var body: some View {
        VStack(spacing: 4) {
            Image(systemName: icon)
                .font(.title3)
                .foregroundColor(.white.opacity(0.9))
            
            Text(value)
                .font(.headline)
                .foregroundColor(.white)
            
            Text(label)
                .font(.caption2)
                .foregroundColor(.white.opacity(0.7))
        }
    }
}

// Color extension (ใช้ใน Weather Card)
extension Color {
    init(hex: String) {
        let hex = hex.trimmingCharacters(in: CharacterSet.alphanumerics.inverted)
        var int: UInt64 = 0
        Scanner(string: hex).scanHexInt64(&int)
        let a, r, g, b: UInt64
        switch hex.count {
        case 3:
            (a, r, g, b) = (255, (int >> 8) * 17, (int >> 4 & 0xF) * 17, (int & 0xF) * 17)
        case 6:
            (a, r, g, b) = (255, int >> 16, int >> 8 & 0xFF, int & 0xFF)
        case 8:
            (a, r, g, b) = (int >> 24, int >> 16 & 0xFF, int >> 8 & 0xFF, int & 0xFF)
        default:
            (a, r, g, b) = (255, 0, 0, 0)
        }
        self.init(
            .sRGB,
            red: Double(r) / 255,
            green: Double(g) / 255,
            blue: Double(b) / 255,
            opacity: Double(a) / 255
        )
    }
}

#Preview("Weather Card") {
    ZStack {
        Color.gray.opacity(0.15)
            .ignoresSafeArea()
        
        WeatherCardView(weather: .preview)
    }
}
```

### แบบฝึกหัดที่ 3: Building a Simple UI จาก Scratch

```swift
// สร้าง Simple Todo App UI
struct TodoItem: Identifiable {
    let id = UUID()
    var title: String
    var isCompleted: Bool
    var priority: Priority
    
    enum Priority: String, CaseIterable {
        case low = "ต่ำ"
        case medium = "กลาง"
        case high = "สูง"
        
        var color: Color {
            switch self {
            case .low: return .green
            case .medium: return .orange
            case .high: return .red
            }
        }
    }
}

struct SimpleUIExample: View {
    @State private var todos = [
        TodoItem(title: "เรียน SwiftUI", isCompleted: true, priority: .high),
        TodoItem(title: "ฝึก coding", isCompleted: false, priority: .high),
        TodoItem(title: "อ่านหนังสือ", isCompleted: false, priority: .medium),
        TodoItem(title: "ออกกำลังกาย", isCompleted: false, priority: .low)
    ]
    
    @State private var newTodoTitle = ""
    @State private var showingInput = false
    
    var completedCount: Int {
        todos.filter { $0.isCompleted }.count
    }
    
    var body: some View {
        NavigationStack {
            VStack(spacing: 0) {
                // Progress Section
                progressSection
                
                // Todo List
                List {
                    ForEach($todos) { $todo in
                        TodoRowView(todo: $todo)
                    }
                    .onDelete { indexSet in
                        todos.remove(atOffsets: indexSet)
                    }
                }
                .listStyle(.insetGrouped)
            }
            .navigationTitle("รายการสิ่งที่ต้องทำ")
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button {
                        showingInput = true
                    } label: {
                        Image(systemName: "plus.circle.fill")
                            .font(.title2)
                    }
                }
            }
            .sheet(isPresented: $showingInput) {
                AddTodoSheet(todos: $todos, isPresented: $showingInput)
            }
        }
    }
    
    private var progressSection: some View {
        VStack(spacing: 8) {
            HStack {
                Text("ความคืบหน้า")
                    .font(.subheadline)
                    .foregroundColor(.gray)
                
                Spacer()
                
                Text("\(completedCount)/\(todos.count)")
                    .font(.subheadline)
                    .foregroundColor(.gray)
            }
            
            ProgressView(value: Double(completedCount), total: Double(max(todos.count, 1)))
                .tint(.blue)
        }
        .padding()
        .background(Color(.systemBackground))
    }
}

struct TodoRowView: View {
    @Binding var todo: TodoItem
    
    var body: some View {
        HStack(spacing: 12) {
            // Checkbox
            Button {
                withAnimation(.spring(response: 0.3)) {
                    todo.isCompleted.toggle()
                }
            } label: {
                Image(systemName: todo.isCompleted ? "checkmark.circle.fill" : "circle")
                    .font(.title3)
                    .foregroundColor(todo.isCompleted ? .blue : .gray)
            }
            .buttonStyle(.plain)
            
            // Title
            VStack(alignment: .leading, spacing: 2) {
                Text(todo.title)
                    .strikethrough(todo.isCompleted, color: .gray)
                    .foregroundColor(todo.isCompleted ? .gray : .primary)
                
                Text("ความสำคัญ: \(todo.priority.rawValue)")
                    .font(.caption)
                    .foregroundColor(todo.priority.color)
            }
            
            Spacer()
            
            // Priority indicator
            Circle()
                .fill(todo.priority.color)
                .frame(width: 8, height: 8)
        }
        .contentShape(Rectangle())
    }
}

struct AddTodoSheet: View {
    @Binding var todos: [TodoItem]
    @Binding var isPresented: Bool
    
    @State private var title = ""
    @State private var priority: TodoItem.Priority = .medium
    
    var body: some View {
        NavigationStack {
            Form {
                Section("รายละเอียด") {
                    TextField("ชื่อรายการ", text: $title)
                    
                    Picker("ความสำคัญ", selection: $priority) {
                        ForEach(TodoItem.Priority.allCases, id: \.self) { p in
                            Text(p.rawValue).tag(p)
                        }
                    }
                }
            }
            .navigationTitle("เพิ่มรายการใหม่")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .cancellationAction) {
                    Button("ยกเลิก") {
                        isPresented = false
                    }
                }
                
                ToolbarItem(placement: .confirmationAction) {
                    Button("เพิ่ม") {
                        if !title.isEmpty {
                            todos.append(TodoItem(
                                title: title,
                                isCompleted: false,
                                priority: priority
                            ))
                            isPresented = false
                        }
                    }
                    .disabled(title.isEmpty)
                }
            }
        }
    }
}

#Preview("Todo App") {
    SimpleUIExample()
}
```

---

## สรุป

ในบทนี้เราได้เรียนรู้พื้นฐานของ SwiftUI ครอบคลุม:

### สิ่งที่เรียนรู้

1. **SwiftUI คืออะไร** - Framework สำหรับสร้าง UI แบบ Declarative บนทุกแพลตฟอร์ม Apple

2. **SwiftUI vs UIKit** - SwiftUI ใหม่กว่า, ง่ายกว่า, cross-platform แต่ UIKit เสถียรกว่าและ community ใหญ่กว่า

3. **โครงสร้างโปรเจกต์** - @main, App protocol, Scene, View protocol เชื่อมต่อกัน

4. **Basic Views**:
   - `Text` - แสดงข้อความ
   - `Image` - แสดงรูปภาพ (SF Symbols, Assets, URL)
   - `Button` - รับ interaction
   - `TextField/SecureField` - รับข้อมูลจากผู้ใช้
   - `Toggle` - สวิตช์เปิด/ปิด
   - `Slider` - เลือกค่าจากช่วง

5. **View Modifiers** - ปรับแต่ง View ด้วยการ chain modifiers, ลำดับสำคัญ

6. **Custom Modifiers** - สร้าง modifier ของตัวเองด้วย `ViewModifier` protocol

7. **ViewBuilder** - สร้าง View จากหลายๆ View

8. **Previews** - ดู UI ใน Xcode Canvas แบบ real-time

9. **Environment** - ส่งข้อมูลผ่าน View hierarchy

10. **Layout**: `HStack`, `VStack`, `ZStack`, `Spacer`, `Divider`, `Group`

### ขั้นตอนต่อไป

- เรียน SwiftUI Layout ขั้นสูง (ตอนที่ 22)
- State Management: @State, @Binding, @ObservableObject
- Navigation: NavigationStack, TabView
- Animation
- Data Persistence: SwiftData, CoreData

### Tips สำหรับ SwiftUI มือใหม่

```swift
// 1. แตก View ใหญ่ๆ ออกเป็นชิ้นเล็กๆ เสมอ
struct BigView: View {
    var body: some View {
        VStack {
            HeaderSection()  // แยกออกมา
            ContentSection() // แยกออกมา
            FooterSection()  // แยกออกมา
        }
    }
}

// 2. ใช้ computed properties สำหรับ logic ใน body
struct SmartView: View {
    let score: Int
    
    // Logic แยกออกจาก View
    private var scoreColor: Color {
        score >= 80 ? .green : score >= 60 ? .orange : .red
    }
    
    var body: some View {
        Text("\(score)")
            .foregroundColor(scoreColor)  // สะอาดกว่า
    }
}

// 3. ใช้ extension เพื่อแยก modifier groups
extension View {
    func primaryButtonStyle() -> some View {
        self
            .font(.headline)
            .foregroundColor(.white)
            .frame(maxWidth: .infinity)
            .padding()
            .background(Color.blue)
            .cornerRadius(15)
    }
}
```

---

*บทต่อไป: [ตอนที่ 22 - SwiftUI Layout ขั้นสูง](Part22_SwiftUI_Layout.md)*
