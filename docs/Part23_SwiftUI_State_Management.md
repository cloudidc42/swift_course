# ตอนที่ 23: SwiftUI State Management

## บทนำ

การจัดการ State (State Management) เป็นหัวใจสำคัญของการพัฒนาแอปด้วย SwiftUI เพราะ SwiftUI ใช้แนวคิด **declarative programming** ซึ่งหมายความว่า UI จะอัปเดตตัวเองอัตโนมัติเมื่อข้อมูล (State) เปลี่ยนแปลง ในบทนี้เราจะเรียนรู้ Property Wrappers ต่างๆ ที่ SwiftUI มีให้สำหรับจัดการ State รวมถึง Observation Framework ใหม่ใน Swift 5.9+

---

## 1. State คืออะไรใน SwiftUI?

**State** ในบริบทของ SwiftUI หมายถึงข้อมูลที่กำหนดว่า UI ควรแสดงผลอย่างไรในช่วงเวลาหนึ่งๆ เมื่อ State เปลี่ยนแปลง SwiftUI จะ re-render ส่วน View ที่ขึ้นอยู่กับ State นั้นโดยอัตโนมัติ

### หลักการสำคัญ: Single Source of Truth

```
Data → View
   ↑      |
   |      ↓
   ← Action
```

ใน SwiftUI เราต้องการ **Single Source of Truth** หมายความว่าข้อมูลชิ้นหนึ่งควรมีแหล่งที่มาเดียว ไม่ควรเก็บข้อมูลเดิมซ้ำซ้อนในหลายที่ เพราะจะทำให้ข้อมูลไม่สอดคล้องกัน

### ประเภทของ State ใน SwiftUI

1. **Local State** - State ที่ใช้ภายใน View เดียว (`@State`)
2. **Shared State** - State ที่แชร์ระหว่างหลาย View (`@Binding`, `@StateObject`, `@ObservedObject`)
3. **Global State** - State ที่ใช้ทั่วทั้งแอป (`@EnvironmentObject`)
4. **Persistent State** - State ที่บันทึกถาวร (`@AppStorage`, `@SceneStorage`)

---

## 2. @State Property Wrapper

`@State` เป็น Property Wrapper พื้นฐานที่สุดใน SwiftUI ใช้สำหรับเก็บค่าที่เปลี่ยนแปลงได้ภายใน View เดียว

### การทำงานของ @State

เมื่อเราประกาศ property ด้วย `@State` SwiftUI จะ:
1. เก็บค่าของ property นั้นใน **Heap** (ไม่ใช่ใน View struct ตรงๆ)
2. สร้าง binding กับ View นั้นๆ
3. เมื่อค่าเปลี่ยน จะ trigger การ re-render View

### ตัวอย่างพื้นฐาน: Counter

```swift
import SwiftUI

struct CounterView: View {
    // ประกาศ @State เป็น private เสมอ เพราะเป็น local state
    @State private var count = 0
    
    var body: some View {
        VStack(spacing: 20) {
            Text("จำนวน: \(count)")
                .font(.largeTitle)
                .fontWeight(.bold)
            
            HStack(spacing: 20) {
                Button("ลด") {
                    if count > 0 {
                        count -= 1
                    }
                }
                .buttonStyle(.bordered)
                
                Button("เพิ่ม") {
                    count += 1
                }
                .buttonStyle(.borderedProminent)
            }
        }
        .padding()
    }
}

#Preview {
    CounterView()
}
```

### @State กับประเภทข้อมูลต่างๆ

```swift
struct StateTypesView: View {
    // ตัวเลข
    @State private var number = 0
    @State private var decimal = 3.14
    
    // ข้อความ
    @State private var name = ""
    
    // Boolean
    @State private var isEnabled = false
    @State private var showSheet = false
    
    // Array
    @State private var items: [String] = []
    
    // Optional
    @State private var selectedItem: String? = nil
    
    // Custom Struct
    @State private var position = CGPoint(x: 0, y: 0)
    
    var body: some View {
        VStack {
            TextField("ชื่อ", text: $name)
                .textFieldStyle(.roundedBorder)
                .padding()
            
            Toggle("เปิดใช้งาน", isOn: $isEnabled)
                .padding()
            
            Text("ชื่อ: \(name)")
            Text("สถานะ: \(isEnabled ? "เปิด" : "ปิด")")
        }
    }
}
```

### ข้อควรระวังกับ @State

```swift
struct StateWarningsView: View {
    @State private var text = "สวัสดี"
    
    var body: some View {
        VStack {
            // ✅ ถูกต้อง: เข้าถึงค่าโดยตรง
            Text(text)
            
            // ✅ ถูกต้อง: ใช้ $ เพื่อสร้าง Binding
            TextField("พิมพ์ข้อความ", text: $text)
            
            // ✅ ถูกต้อง: แก้ไขใน closure
            Button("เคลียร์") {
                text = ""
            }
            
            // ❌ ผิด: ไม่สามารถแก้ไขนอก body
            // text = "ค่าใหม่" // error
        }
    }
}
```

---

## 3. @Binding Property Wrapper

`@Binding` ใช้สำหรับสร้าง **two-way connection** ระหว่าง View ลูกกับข้อมูลที่มาจาก View แม่ ทำให้ View ลูกสามารถอ่านและแก้ไขข้อมูลของ View แม่ได้

### การทำงานของ @Binding

```
View แม่ (@State) ←→ View ลูก (@Binding)
```

View ลูกไม่ได้เป็นเจ้าของข้อมูล แต่ได้รับ reference ไปยังข้อมูลของ View แม่

### ตัวอย่างพื้นฐาน

```swift
// View ลูกที่รับ @Binding
struct ToggleButton: View {
    @Binding var isOn: Bool  // รับ binding จาก parent
    let label: String
    
    var body: some View {
        Button(action: {
            isOn.toggle()  // แก้ไขค่าใน parent ได้เลย
        }) {
            HStack {
                Image(systemName: isOn ? "checkmark.circle.fill" : "circle")
                    .foregroundColor(isOn ? .green : .gray)
                Text(label)
            }
        }
        .buttonStyle(.plain)
    }
}

// View แม่ที่ส่ง @Binding ให้ลูก
struct ParentView: View {
    @State private var isDarkMode = false
    @State private var notificationsOn = true
    @State private var locationOn = false
    
    var body: some View {
        VStack(alignment: .leading, spacing: 15) {
            Text("การตั้งค่า")
                .font(.title)
                .fontWeight(.bold)
            
            // ส่ง binding ด้วย $ prefix
            ToggleButton(isOn: $isDarkMode, label: "โหมดมืด")
            ToggleButton(isOn: $notificationsOn, label: "การแจ้งเตือน")
            ToggleButton(isOn: $locationOn, label: "ตำแหน่งที่ตั้ง")
        }
        .padding()
    }
}
```

### @Binding กับ Form

```swift
struct ProfileFormView: View {
    @State private var firstName = ""
    @State private var lastName = ""
    @State private var age = 18
    @State private var gender = "ไม่ระบุ"
    
    var body: some View {
        Form {
            Section("ข้อมูลส่วนตัว") {
                NameField(title: "ชื่อ", text: $firstName)
                NameField(title: "นามสกุล", text: $lastName)
                
                Stepper("อายุ: \(age)", value: $age, in: 1...120)
                
                Picker("เพศ", selection: $gender) {
                    Text("ชาย").tag("ชาย")
                    Text("หญิง").tag("หญิง")
                    Text("ไม่ระบุ").tag("ไม่ระบุ")
                }
            }
        }
    }
}

struct NameField: View {
    let title: String
    @Binding var text: String
    
    var body: some View {
        HStack {
            Text(title)
                .foregroundColor(.secondary)
                .frame(width: 80, alignment: .leading)
            TextField(title, text: $text)
        }
    }
}
```

### Binding.constant สำหรับ Preview

```swift
struct MyToggle: View {
    @Binding var value: Bool
    
    var body: some View {
        Toggle("ตัวเลือก", isOn: $value)
    }
}

#Preview {
    // ใช้ Binding.constant สำหรับ preview
    MyToggle(value: .constant(true))
}
```

---

## 4. ObservableObject Protocol

`ObservableObject` เป็น protocol ที่ทำให้ class สามารถแจ้งให้ SwiftUI รู้ว่ามีการเปลี่ยนแปลงเกิดขึ้น โดยใช้ร่วมกับ `@Published`

### โครงสร้างพื้นฐาน

```swift
import SwiftUI
import Combine

class UserSettings: ObservableObject {
    @Published var username = "ผู้ใช้งาน"
    @Published var email = ""
    @Published var isPremium = false
    
    // Property ที่ไม่ใช่ @Published จะไม่ trigger การ update
    var lastLoginDate = Date()
    
    func upgrade() {
        isPremium = true
    }
    
    func logout() {
        username = "ผู้ใช้งาน"
        email = ""
        isPremium = false
    }
}
```

### @Published Property Wrapper

`@Published` เป็น Property Wrapper ที่ใช้ภายใน `ObservableObject` เมื่อค่าของ property ที่มี `@Published` เปลี่ยน จะส่งสัญญาณให้ View ที่ subscribe อยู่ทำการ re-render

```swift
class ShoppingCart: ObservableObject {
    @Published var items: [CartItem] = []
    @Published var isLoading = false
    @Published var errorMessage: String? = nil
    
    // Computed property จาก @Published properties
    var totalPrice: Double {
        items.reduce(0) { $0 + $1.price * Double($1.quantity) }
    }
    
    var itemCount: Int {
        items.reduce(0) { $0 + $1.quantity }
    }
    
    func addItem(_ item: CartItem) {
        if let index = items.firstIndex(where: { $0.id == item.id }) {
            items[index].quantity += 1
        } else {
            items.append(item)
        }
    }
    
    func removeItem(_ item: CartItem) {
        items.removeAll { $0.id == item.id }
    }
    
    func clearCart() {
        items = []
    }
}

struct CartItem: Identifiable {
    let id = UUID()
    let name: String
    let price: Double
    var quantity: Int
}
```

---

## 5. @StateObject Property Wrapper

`@StateObject` ใช้สำหรับสร้างและเป็นเจ้าของ instance ของ `ObservableObject` ภายใน View SwiftUI จะสร้าง object เพียงครั้งเดียวและรักษาไว้ตลอดชีวิตของ View

### เมื่อไรควรใช้ @StateObject

- เมื่อ View นั้นเป็นผู้สร้าง (owner) ของ object
- เมื่อต้องการให้ object มีชีวิตอยู่ตลอดช่วงที่ View แสดงอยู่

```swift
class TimerViewModel: ObservableObject {
    @Published var seconds = 0
    @Published var isRunning = false
    
    private var timer: Timer?
    
    func start() {
        isRunning = true
        timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { _ in
            self.seconds += 1
        }
    }
    
    func stop() {
        isRunning = false
        timer?.invalidate()
        timer = nil
    }
    
    func reset() {
        stop()
        seconds = 0
    }
    
    deinit {
        timer?.invalidate()
    }
}

struct TimerView: View {
    // ✅ @StateObject: View นี้เป็นเจ้าของ TimerViewModel
    @StateObject private var viewModel = TimerViewModel()
    
    var body: some View {
        VStack(spacing: 30) {
            Text(formatTime(viewModel.seconds))
                .font(.system(size: 60, weight: .thin, design: .monospaced))
            
            HStack(spacing: 20) {
                Button(viewModel.isRunning ? "หยุด" : "เริ่ม") {
                    if viewModel.isRunning {
                        viewModel.stop()
                    } else {
                        viewModel.start()
                    }
                }
                .buttonStyle(.bordered)
                .tint(viewModel.isRunning ? .red : .green)
                
                Button("รีเซ็ต") {
                    viewModel.reset()
                }
                .buttonStyle(.bordered)
            }
        }
        .padding()
    }
    
    func formatTime(_ seconds: Int) -> String {
        let mins = seconds / 60
        let secs = seconds % 60
        return String(format: "%02d:%02d", mins, secs)
    }
}
```

### ปัญหาที่พบบ่อย: @State vs @StateObject

```swift
// ❌ ผิด: ใช้ @State กับ class
struct WrongView: View {
    @State private var viewModel = MyViewModel()
    // SwiftUI จะสร้าง viewModel ใหม่ทุกครั้งที่ re-render!
    
    var body: some View { Text("") }
}

// ✅ ถูกต้อง: ใช้ @StateObject กับ class
struct CorrectView: View {
    @StateObject private var viewModel = MyViewModel()
    // SwiftUI จะสร้าง viewModel เพียงครั้งเดียว
    
    var body: some View { Text("") }
}
```

---

## 6. @ObservedObject Property Wrapper

`@ObservedObject` คล้ายกับ `@StateObject` แต่ต่างกันตรงที่ View ไม่ได้เป็นเจ้าของ object — แต่รับ object มาจากภายนอก

### ความแตกต่างระหว่าง @StateObject และ @ObservedObject

| | @StateObject | @ObservedObject |
|---|---|---|
| เจ้าของ | View เป็นเจ้าของ | รับมาจากภายนอก |
| การสร้าง | สร้างใน View | สร้างก่อนแล้วส่งมา |
| ชีวิต | อยู่นานเท่า View | ขึ้นอยู่กับผู้สร้าง |

```swift
class ProductViewModel: ObservableObject {
    @Published var name = ""
    @Published var price = 0.0
    @Published var quantity = 0
    @Published var isInStock = true
    
    var total: Double {
        price * Double(quantity)
    }
}

// View แม่: เป็นเจ้าของ ViewModel
struct ProductListView: View {
    @StateObject private var viewModel = ProductViewModel()
    
    var body: some View {
        VStack {
            // ส่ง viewModel ให้ View ลูก
            ProductDetailView(viewModel: viewModel)
            ProductCartView(viewModel: viewModel)
        }
    }
}

// View ลูก 1: รับ ViewModel มา
struct ProductDetailView: View {
    @ObservedObject var viewModel: ProductViewModel
    
    var body: some View {
        VStack(alignment: .leading) {
            TextField("ชื่อสินค้า", text: $viewModel.name)
            TextField("ราคา", value: $viewModel.price, format: .number)
            Toggle("มีในสต็อก", isOn: $viewModel.isInStock)
        }
        .padding()
    }
}

// View ลูก 2: รับ ViewModel มา
struct ProductCartView: View {
    @ObservedObject var viewModel: ProductViewModel
    
    var body: some View {
        HStack {
            Stepper("จำนวน: \(viewModel.quantity)", 
                    value: $viewModel.quantity, 
                    in: 0...100)
            Text("รวม: \(viewModel.total, format: .currency(code: "THB"))")
        }
        .padding()
    }
}
```

---

## 7. @EnvironmentObject Property Wrapper

`@EnvironmentObject` ใช้สำหรับแชร์ object กับ View ลูกทุกตัวในลำดับชั้น View โดยไม่ต้องส่งผ่านทุก View

### การทำงาน

```
App
└── ContentView
    ├── HomeView (@EnvironmentObject ใช้ได้)
    │   └── ProfileView (@EnvironmentObject ใช้ได้)
    └── SettingsView (@EnvironmentObject ใช้ได้)
```

```swift
// สร้าง AppState สำหรับเก็บข้อมูลทั่วทั้งแอป
class AppState: ObservableObject {
    @Published var isLoggedIn = false
    @Published var currentUser: User? = nil
    @Published var theme: AppTheme = .light
    @Published var language = "th"
    
    func login(user: User) {
        currentUser = user
        isLoggedIn = true
    }
    
    func logout() {
        currentUser = nil
        isLoggedIn = false
    }
}

struct User: Identifiable {
    let id = UUID()
    var name: String
    var email: String
    var avatarURL: URL?
}

enum AppTheme {
    case light, dark, system
}

// ใส่ @EnvironmentObject ที่ App หรือ View ระดับสูงสุด
@main
struct MyApp: App {
    @StateObject private var appState = AppState()
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(appState)  // inject ที่นี่
        }
    }
}

// ContentView ไม่จำเป็นต้องรับ appState มาเอง
struct ContentView: View {
    @EnvironmentObject var appState: AppState
    
    var body: some View {
        if appState.isLoggedIn {
            MainTabView()
        } else {
            LoginView()
        }
    }
}

// View ลูกสามารถใช้ได้เลยโดยไม่ต้องรับ parameter
struct HomeView: View {
    @EnvironmentObject var appState: AppState
    
    var body: some View {
        VStack {
            if let user = appState.currentUser {
                Text("สวัสดี, \(user.name)!")
                    .font(.title)
            }
            
            Button("ออกจากระบบ") {
                appState.logout()
            }
            .foregroundColor(.red)
        }
    }
}

struct ProfileView: View {
    @EnvironmentObject var appState: AppState
    
    var body: some View {
        Form {
            if let user = appState.currentUser {
                Section("ข้อมูลส่วนตัว") {
                    Text("ชื่อ: \(user.name)")
                    Text("อีเมล: \(user.email)")
                }
            }
            
            Section("การตั้งค่า") {
                Picker("ธีม", selection: $appState.theme) {
                    Text("สว่าง").tag(AppTheme.light)
                    Text("มืด").tag(AppTheme.dark)
                    Text("ตามระบบ").tag(AppTheme.system)
                }
                
                Picker("ภาษา", selection: $appState.language) {
                    Text("ไทย").tag("th")
                    Text("English").tag("en")
                }
            }
        }
    }
}
```

### ข้อควรระวัง: Missing EnvironmentObject

```swift
// ❌ ถ้าลืม inject .environmentObject() จะ crash!
struct BadApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
            // ลืม .environmentObject(AppState())
        }
    }
}

// ✅ แก้ไข: ต้อง inject เสมอ
struct GoodApp: App {
    @StateObject private var appState = AppState()
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(appState)
        }
    }
}
```

---

## 8. Observation Framework (Swift 5.9+)

ใน Swift 5.9 (iOS 17+) มี Observation Framework ใหม่ที่ทำให้การจัดการ State ง่ายขึ้นและมีประสิทธิภาพดีขึ้น โดยใช้ `@Observable` macro

### ปัญหาของ ObservableObject แบบเดิม

```swift
// แบบเดิม: ต้องประกาศ @Published ทุก property
class OldUserModel: ObservableObject {
    @Published var name = ""
    @Published var email = ""
    @Published var age = 0
    // ต้องเขียน @Published ทุกครั้ง
    // ถ้าลืม property นั้นจะไม่ trigger update
}
```

### @Observable Macro

```swift
import Observation

// แบบใหม่: ใช้ @Observable macro
@Observable
class UserModel {
    var name = ""       // ไม่ต้องใส่ @Published
    var email = ""      // ทุก property จะ observable อัตโนมัติ
    var age = 0
    
    // ถ้าไม่ต้องการให้ observable ใช้ @ObservationIgnored
    @ObservationIgnored
    var internalCache: [String: Any] = [:]
}
```

### การใช้งาน @Observable

```swift
@Observable
class BookStore {
    var books: [Book] = []
    var searchQuery = ""
    var isLoading = false
    var selectedCategory: BookCategory = .all
    
    var filteredBooks: [Book] {
        if searchQuery.isEmpty && selectedCategory == .all {
            return books
        }
        return books.filter { book in
            let matchesSearch = searchQuery.isEmpty || 
                book.title.localizedCaseInsensitiveContains(searchQuery)
            let matchesCategory = selectedCategory == .all || 
                book.category == selectedCategory
            return matchesSearch && matchesCategory
        }
    }
    
    func loadBooks() async {
        isLoading = true
        // simulate network call
        try? await Task.sleep(nanoseconds: 1_000_000_000)
        books = Book.sampleData
        isLoading = false
    }
    
    func addBook(_ book: Book) {
        books.append(book)
    }
    
    func deleteBook(at indexSet: IndexSet) {
        books.remove(atOffsets: indexSet)
    }
}

struct Book: Identifiable {
    let id = UUID()
    var title: String
    var author: String
    var category: BookCategory
    var rating: Int
    
    static var sampleData: [Book] = [
        Book(title: "การเขียน Swift", author: "สมชาย", category: .programming, rating: 5),
        Book(title: "SwiftUI สำหรับมือใหม่", author: "สมหญิง", category: .programming, rating: 4),
    ]
}

enum BookCategory: String, CaseIterable {
    case all = "ทั้งหมด"
    case programming = "โปรแกรมมิ่ง"
    case design = "ดีไซน์"
    case business = "ธุรกิจ"
}

struct BookListView: View {
    @State private var store = BookStore()  // ใช้ @State แทน @StateObject
    
    var body: some View {
        NavigationStack {
            List {
                ForEach(store.filteredBooks) { book in
                    BookRow(book: book)
                }
                .onDelete { store.deleteBook(at: $0) }
            }
            .searchable(text: $store.searchQuery)
            .overlay {
                if store.isLoading {
                    ProgressView("กำลังโหลด...")
                }
            }
            .task {
                await store.loadBooks()
            }
            .navigationTitle("ร้านหนังสือ")
            .toolbar {
                Picker("หมวดหมู่", selection: $store.selectedCategory) {
                    ForEach(BookCategory.allCases, id: \.self) { category in
                        Text(category.rawValue).tag(category)
                    }
                }
            }
        }
    }
}

struct BookRow: View {
    let book: Book
    
    var body: some View {
        VStack(alignment: .leading, spacing: 4) {
            Text(book.title)
                .font(.headline)
            Text(book.author)
                .font(.subheadline)
                .foregroundColor(.secondary)
            HStack {
                ForEach(1...5, id: \.self) { star in
                    Image(systemName: star <= book.rating ? "star.fill" : "star")
                        .foregroundColor(.yellow)
                }
            }
        }
    }
}
```

### ความแตกต่างระหว่าง ObservableObject และ @Observable

```swift
// แบบเดิม (ObservableObject)
class OldModel: ObservableObject {
    @Published var value1 = 0
    @Published var value2 = ""
    
    // ปัญหา: update ทุก property แม้มีแค่ค่าเดียวที่เปลี่ยน
}

struct OldView: View {
    @StateObject private var model = OldModel()
    // Re-render ทั้ง view เมื่อ model เปลี่ยน
    
    var body: some View {
        Text("\(model.value1)")  // render ใหม่แม้ value2 เปลี่ยน
    }
}

// แบบใหม่ (@Observable)
@Observable
class NewModel {
    var value1 = 0
    var value2 = ""
    
    // ดีกว่า: track การใช้งาน property แต่ละตัว
}

struct NewView: View {
    @State private var model = NewModel()
    // Re-render เฉพาะเมื่อ property ที่ view ใช้อยู่เปลี่ยน
    
    var body: some View {
        Text("\(model.value1)")  // render ใหม่เฉพาะเมื่อ value1 เปลี่ยน
    }
}
```

---

## 9. @Bindable (Swift 5.9+)

`@Bindable` ใช้กับ `@Observable` objects เพื่อสร้าง `Binding` ไปยัง properties ของ object

```swift
@Observable
class FormData {
    var firstName = ""
    var lastName = ""
    var birthDate = Date()
    var agreedToTerms = false
}

struct RegistrationView: View {
    @State private var formData = FormData()
    
    var body: some View {
        Form {
            // ใช้ @Bindable เพื่อสร้าง binding
            FormFields(formData: formData)
            
            Toggle("ยอมรับเงื่อนไข", isOn: $formData.agreedToTerms)
            
            Button("ลงทะเบียน") {
                submitForm()
            }
            .disabled(!formData.agreedToTerms)
        }
    }
    
    func submitForm() {
        print("ลงทะเบียน: \(formData.firstName) \(formData.lastName)")
    }
}

struct FormFields: View {
    @Bindable var formData: FormData  // ใช้ @Bindable แทน @ObservedObject
    
    var body: some View {
        Section("ข้อมูลส่วนตัว") {
            TextField("ชื่อ", text: $formData.firstName)
            TextField("นามสกุล", text: $formData.lastName)
            DatePicker("วันเกิด", selection: $formData.birthDate, 
                      displayedComponents: .date)
        }
    }
}
```

---

## 10. Data Flow ใน SwiftUI

### ทิศทางการไหลของข้อมูล

```
Source of Truth
     │
     ▼
Parent View
 ├── @State ─────────────────────── (เก็บใน parent)
 │         │
 │         ▼
 ├── Child View A         ←── @Binding (อ่าน/เขียน parent state)
 │
 └── Child View B         ←── @EnvironmentObject (global access)
```

### Pattern: Unidirectional Data Flow

```swift
// Model
struct Task: Identifiable {
    let id = UUID()
    var title: String
    var isCompleted: Bool
    var priority: Priority
    
    enum Priority: String, CaseIterable {
        case low = "ต่ำ"
        case medium = "กลาง"
        case high = "สูง"
    }
}

// ViewModel
@Observable
class TaskStore {
    var tasks: [Task] = []
    var filter: TaskFilter = .all
    
    enum TaskFilter {
        case all, active, completed
    }
    
    var filteredTasks: [Task] {
        switch filter {
        case .all:
            return tasks
        case .active:
            return tasks.filter { !$0.isCompleted }
        case .completed:
            return tasks.filter { $0.isCompleted }
        }
    }
    
    // Actions
    func addTask(title: String, priority: Task.Priority = .medium) {
        tasks.append(Task(title: title, isCompleted: false, priority: priority))
    }
    
    func toggleTask(_ task: Task) {
        if let index = tasks.firstIndex(where: { $0.id == task.id }) {
            tasks[index].isCompleted.toggle()
        }
    }
    
    func deleteTask(_ task: Task) {
        tasks.removeAll { $0.id == task.id }
    }
    
    func clearCompleted() {
        tasks.removeAll { $0.isCompleted }
    }
}

// Views
struct TaskListView: View {
    @State private var store = TaskStore()
    @State private var newTaskTitle = ""
    
    var body: some View {
        NavigationStack {
            VStack {
                // Input
                HStack {
                    TextField("เพิ่มงานใหม่...", text: $newTaskTitle)
                        .textFieldStyle(.roundedBorder)
                    
                    Button("เพิ่ม") {
                        guard !newTaskTitle.trimmingCharacters(in: .whitespaces).isEmpty else { return }
                        store.addTask(title: newTaskTitle)
                        newTaskTitle = ""
                    }
                    .disabled(newTaskTitle.isEmpty)
                }
                .padding()
                
                // Filter
                Picker("กรอง", selection: $store.filter) {
                    Text("ทั้งหมด").tag(TaskStore.TaskFilter.all)
                    Text("ยังไม่เสร็จ").tag(TaskStore.TaskFilter.active)
                    Text("เสร็จแล้ว").tag(TaskStore.TaskFilter.completed)
                }
                .pickerStyle(.segmented)
                .padding(.horizontal)
                
                // List
                List {
                    ForEach(store.filteredTasks) { task in
                        TaskRow(task: task, store: store)
                    }
                }
                
                // Footer
                HStack {
                    Text("\(store.tasks.filter { !$0.isCompleted }.count) งานที่เหลือ")
                        .foregroundColor(.secondary)
                    
                    Spacer()
                    
                    Button("ลบที่เสร็จแล้ว") {
                        store.clearCompleted()
                    }
                    .foregroundColor(.red)
                }
                .padding()
            }
            .navigationTitle("รายการงาน")
        }
    }
}

struct TaskRow: View {
    let task: Task
    let store: TaskStore
    
    var body: some View {
        HStack {
            Button {
                store.toggleTask(task)
            } label: {
                Image(systemName: task.isCompleted ? "checkmark.circle.fill" : "circle")
                    .foregroundColor(task.isCompleted ? .green : .gray)
            }
            .buttonStyle(.plain)
            
            Text(task.title)
                .strikethrough(task.isCompleted)
                .foregroundColor(task.isCompleted ? .secondary : .primary)
            
            Spacer()
            
            PriorityBadge(priority: task.priority)
        }
        .swipeActions(edge: .trailing) {
            Button("ลบ", role: .destructive) {
                store.deleteTask(task)
            }
        }
    }
}

struct PriorityBadge: View {
    let priority: Task.Priority
    
    var color: Color {
        switch priority {
        case .low: return .blue
        case .medium: return .orange
        case .high: return .red
        }
    }
    
    var body: some View {
        Text(priority.rawValue)
            .font(.caption)
            .padding(.horizontal, 8)
            .padding(.vertical, 4)
            .background(color.opacity(0.2))
            .foregroundColor(color)
            .clipShape(Capsule())
    }
}
```

---

## 11. Parent-Child Communication

### การส่งข้อมูลจาก Parent ไป Child

```swift
// 1. ส่งผ่าน Initializer (ค่าธรรมดา)
struct ChildView: View {
    let message: String  // รับค่าธรรมดา
    
    var body: some View {
        Text(message)
    }
}

// 2. ส่งผ่าน @Binding (สองทาง)
struct EditableChild: View {
    @Binding var text: String  // สองทาง
    
    var body: some View {
        TextField("แก้ไข", text: $text)
    }
}

// 3. ส่ง Closure (callback)
struct ActionChild: View {
    let onTap: () -> Void
    let onDelete: (Int) -> Void
    
    var body: some View {
        VStack {
            Button("แตะ", action: onTap)
            Button("ลบ 1") { onDelete(1) }
        }
    }
}
```

### การส่งข้อมูลจาก Child ไป Parent ผ่าน Callback

```swift
struct ParentWithCallback: View {
    @State private var selectedColor = Color.blue
    @State private var history: [Color] = []
    
    var body: some View {
        VStack {
            // แสดงประวัติ
            ScrollView(.horizontal) {
                HStack {
                    ForEach(history.indices, id: \.self) { index in
                        RoundedRectangle(cornerRadius: 8)
                            .fill(history[index])
                            .frame(width: 40, height: 40)
                    }
                }
                .padding()
            }
            
            // ColorPicker ลูกส่งข้อมูลกลับผ่าน closure
            ColorPickerView(
                selectedColor: $selectedColor,
                onColorSelected: { color in
                    history.append(color)
                }
            )
        }
    }
}

struct ColorPickerView: View {
    @Binding var selectedColor: Color
    let onColorSelected: (Color) -> Void  // callback
    
    let colors: [Color] = [.red, .orange, .yellow, .green, .blue, .purple]
    
    var body: some View {
        HStack {
            ForEach(colors, id: \.self) { color in
                Circle()
                    .fill(color)
                    .frame(width: 44, height: 44)
                    .overlay(
                        Circle()
                            .stroke(selectedColor == color ? .white : .clear, lineWidth: 3)
                    )
                    .onTapGesture {
                        selectedColor = color
                        onColorSelected(color)  // แจ้ง parent
                    }
            }
        }
    }
}
```

---

## 12. @AppStorage

`@AppStorage` ใช้สำหรับบันทึกค่าลงใน `UserDefaults` ทำให้ข้อมูลคงอยู่แม้ปิดแอปแล้ว

```swift
struct SettingsView: View {
    // ค่าจะถูกบันทึกใน UserDefaults อัตโนมัติ
    @AppStorage("isDarkMode") private var isDarkMode = false
    @AppStorage("fontSize") private var fontSize = 16.0
    @AppStorage("language") private var language = "th"
    @AppStorage("notificationsEnabled") private var notificationsEnabled = true
    @AppStorage("username") private var username = ""
    
    var body: some View {
        Form {
            Section("การแสดงผล") {
                Toggle("โหมดมืด", isOn: $isDarkMode)
                
                VStack(alignment: .leading) {
                    Text("ขนาดตัวอักษร: \(Int(fontSize))")
                    Slider(value: $fontSize, in: 12...24, step: 1)
                }
                
                Picker("ภาษา", selection: $language) {
                    Text("ไทย").tag("th")
                    Text("English").tag("en")
                    Text("日本語").tag("ja")
                }
            }
            
            Section("การแจ้งเตือน") {
                Toggle("เปิดการแจ้งเตือน", isOn: $notificationsEnabled)
            }
            
            Section("บัญชี") {
                TextField("ชื่อผู้ใช้", text: $username)
            }
            
            Button("รีเซ็ตการตั้งค่า") {
                // รีเซ็ตค่าทั้งหมด
                isDarkMode = false
                fontSize = 16.0
                language = "th"
                notificationsEnabled = true
                username = ""
            }
            .foregroundColor(.red)
        }
    }
}
```

### @AppStorage กับ Custom Types

```swift
// ประเภทที่รองรับโดยตรง: Bool, Int, Double, String, URL, Data
// สำหรับ enum ต้องทำให้ conform RawRepresentable

enum Theme: String {
    case light, dark, system
}

struct ThemeSettingsView: View {
    @AppStorage("appTheme") private var theme: Theme = .system
    
    var body: some View {
        Picker("ธีม", selection: $theme) {
            Text("สว่าง").tag(Theme.light)
            Text("มืด").tag(Theme.dark)
            Text("ตามระบบ").tag(Theme.system)
        }
    }
}

// ขยาย @AppStorage ด้วย Codable
extension AppStorage {
    // ใช้งานกับ Codable object ผ่าน JSON encoding
}

struct UserProfile: Codable {
    var name: String
    var age: Int
    var preferences: [String: Bool]
}

class ProfileStorage {
    static let key = "userProfile"
    
    static func save(_ profile: UserProfile) {
        if let data = try? JSONEncoder().encode(profile) {
            UserDefaults.standard.set(data, forKey: key)
        }
    }
    
    static func load() -> UserProfile? {
        guard let data = UserDefaults.standard.data(forKey: key),
              let profile = try? JSONDecoder().decode(UserProfile.self, from: data) else {
            return nil
        }
        return profile
    }
}
```

---

## 13. @SceneStorage

`@SceneStorage` คล้ายกับ `@AppStorage` แต่ข้อมูลถูกบันทึกตาม Scene (หน้าต่าง) ไม่ใช่ทั้งแอป เหมาะสำหรับ multi-window apps บน iPad และ Mac

```swift
struct DocumentEditor: View {
    @SceneStorage("documentText") private var documentText = ""
    @SceneStorage("cursorPosition") private var cursorPosition = 0
    @SceneStorage("isEditing") private var isEditing = false
    
    var body: some View {
        VStack {
            HStack {
                Text("เอกสาร")
                    .font(.title)
                Spacer()
                Toggle("แก้ไข", isOn: $isEditing)
            }
            .padding()
            
            if isEditing {
                TextEditor(text: $documentText)
                    .border(Color.secondary.opacity(0.5))
            } else {
                ScrollView {
                    Text(documentText.isEmpty ? "ไม่มีเนื้อหา" : documentText)
                        .frame(maxWidth: .infinity, alignment: .leading)
                        .padding()
                }
            }
        }
    }
}
```

---

## 14. Preference Keys

`PreferenceKey` ใช้สำหรับส่งข้อมูลจาก View ลูก **ขึ้นไป** หา View แม่ (ทิศทางตรงข้ามกับปกติ)

### ใช้งานจริง: วัดขนาด View

```swift
// ประกาศ PreferenceKey
struct ViewSizeKey: PreferenceKey {
    static var defaultValue: CGSize = .zero
    
    static func reduce(value: inout CGSize, nextValue: () -> CGSize) {
        value = nextValue()
    }
}

struct SizeReaderView: View {
    @State private var viewSize: CGSize = .zero
    
    var body: some View {
        VStack {
            Text("ขนาดกล่องสีฟ้า: \(Int(viewSize.width)) x \(Int(viewSize.height))")
                .padding()
            
            Rectangle()
                .fill(Color.blue.opacity(0.3))
                .frame(width: 200, height: 150)
                .background(
                    GeometryReader { geometry in
                        Color.clear
                            .preference(
                                key: ViewSizeKey.self,
                                value: geometry.size
                            )
                    }
                )
                .onPreferenceChange(ViewSizeKey.self) { size in
                    viewSize = size
                }
        }
    }
}
```

### ตัวอย่างขั้นสูง: Equal Width Buttons

```swift
struct MaxWidthKey: PreferenceKey {
    static var defaultValue: CGFloat = 0
    
    static func reduce(value: inout CGFloat, nextValue: () -> CGFloat) {
        value = max(value, nextValue())
    }
}

struct EqualWidthButtonsView: View {
    @State private var maxButtonWidth: CGFloat = 0
    
    var body: some View {
        HStack(spacing: 10) {
            ForEach(["สั้น", "ปุ่มยาวกว่า", "กลาง"], id: \.self) { title in
                Button(title) { }
                    .buttonStyle(.bordered)
                    .background(
                        GeometryReader { geo in
                            Color.clear
                                .preference(key: MaxWidthKey.self, value: geo.size.width)
                        }
                    )
                    .frame(width: maxButtonWidth == 0 ? nil : maxButtonWidth)
            }
        }
        .onPreferenceChange(MaxWidthKey.self) { width in
            maxButtonWidth = width
        }
        .padding()
    }
}
```

---

## 15. Environment Values

SwiftUI มี built-in environment values หลายตัวที่เราสามารถอ่านได้ใน View

```swift
struct EnvironmentDemoView: View {
    // อ่าน environment values
    @Environment(\.colorScheme) var colorScheme
    @Environment(\.dynamicTypeSize) var dynamicTypeSize
    @Environment(\.isEnabled) var isEnabled
    @Environment(\.dismiss) var dismiss
    @Environment(\.openURL) var openURL
    @Environment(\.locale) var locale
    @Environment(\.timeZone) var timeZone
    @Environment(\.calendar) var calendar
    @Environment(\.horizontalSizeClass) var horizontalSizeClass
    @Environment(\.verticalSizeClass) var verticalSizeClass
    
    var body: some View {
        VStack(alignment: .leading, spacing: 10) {
            Text("Color Scheme: \(colorScheme == .dark ? "มืด" : "สว่าง")")
            Text("Dynamic Type: \(dynamicTypeSize.description)")
            Text("Locale: \(locale.identifier)")
            Text("Timezone: \(timeZone.identifier)")
            
            if horizontalSizeClass == .compact {
                Text("หน้าจอเล็ก (iPhone)")
            } else {
                Text("หน้าจอใหญ่ (iPad/Mac)")
            }
            
            Button("เปิดเว็บไซต์") {
                openURL(URL(string: "https://swift.org")!)
            }
        }
        .padding()
    }
}
```

---

## 16. Custom Environment Values

เราสามารถสร้าง Environment Values เองได้

```swift
// 1. ประกาศ EnvironmentKey
struct ThemeColorKey: EnvironmentKey {
    static let defaultValue: Color = .blue
}

struct FontSizeKey: EnvironmentKey {
    static let defaultValue: CGFloat = 16
}

// 2. ขยาย EnvironmentValues
extension EnvironmentValues {
    var themeColor: Color {
        get { self[ThemeColorKey.self] }
        set { self[ThemeColorKey.self] = newValue }
    }
    
    var appFontSize: CGFloat {
        get { self[FontSizeKey.self] }
        set { self[FontSizeKey.self] = newValue }
    }
}

// 3. ใช้งาน
struct ThemedButton: View {
    @Environment(\.themeColor) var themeColor
    @Environment(\.appFontSize) var fontSize
    
    let title: String
    let action: () -> Void
    
    var body: some View {
        Button(title, action: action)
            .font(.system(size: fontSize))
            .foregroundColor(.white)
            .padding()
            .background(themeColor)
            .clipShape(RoundedRectangle(cornerRadius: 10))
    }
}

struct CustomEnvironmentDemo: View {
    var body: some View {
        VStack(spacing: 20) {
            // ใช้ค่า default
            ThemedButton(title: "ปุ่มธรรมดา") { }
            
            // กำหนดค่าเอง
            ThemedButton(title: "ปุ่มสีแดง") { }
                .environment(\.themeColor, .red)
                .environment(\.appFontSize, 20)
            
            // ส่งต่อให้ View ลูก
            VStack {
                ThemedButton(title: "ปุ่มในกลุ่มสีเขียว") { }
                ThemedButton(title: "ปุ่มอีกอัน") { }
            }
            .environment(\.themeColor, .green)
        }
        .padding()
    }
}
```

---

## 17. เมื่อไรควรใช้อะไร?

### แผนผังการตัดสินใจ

```
ต้องการเก็บ State ประเภทไหน?
│
├── ข้อมูลชั่วคราวใน View เดียว (ตัวเลข, Bool, String เล็กๆ)
│   └── @State
│
├── แชร์ข้อมูลกับ View ลูก (สองทาง)
│   └── @Binding
│
├── Object ที่ซับซ้อน, View เป็นเจ้าของ
│   ├── Swift 5.9+ (iOS 17+): @State + @Observable
│   └── ก่อนหน้า: @StateObject + ObservableObject
│
├── รับ Object มาจาก View แม่
│   ├── Swift 5.9+ (iOS 17+): Parameter ธรรมดา + @Bindable
│   └── ก่อนหน้า: @ObservedObject
│
├── แชร์ Object กับ View ทั้งแอป
│   └── @EnvironmentObject
│
├── บันทึกค่าถาวร (UserDefaults)
│   └── @AppStorage
│
└── บันทึกค่าตาม Scene
    └── @SceneStorage
```

### ตารางสรุป

| Property Wrapper | เจ้าของ | ขอบเขต | การบันทึก |
|---|---|---|---|
| `@State` | View | View เดียว | ไม่บันทึก |
| `@Binding` | View แม่ | View แม่+ลูก | ไม่บันทึก |
| `@StateObject` | View | View ลูกๆ | ไม่บันทึก |
| `@ObservedObject` | ภายนอก | View ที่รับมา | ไม่บันทึก |
| `@EnvironmentObject` | View แม่ | ทั้งลำดับชั้น | ไม่บันทึก |
| `@AppStorage` | App | ทั้งแอป | UserDefaults |
| `@SceneStorage` | Scene | Scene เดียว | Scene Storage |
| `@Observable` | - | กำหนดเอง | ไม่บันทึก |

---

## 18. Best Practices

### 1. ใช้ @State กับข้อมูลง่ายๆ เท่านั้น

```swift
// ✅ ดี: ข้อมูลง่ายๆ
struct GoodView: View {
    @State private var isExpanded = false
    @State private var searchText = ""
    @State private var selectedTab = 0
    
    var body: some View { Text("") }
}

// ⚠️ ระวัง: ข้อมูลซับซ้อนควรใช้ @Observable หรือ ObservableObject
struct AvoidThis: View {
    @State private var users: [User] = []  // ควรย้ายไปใน ViewModel
    @State private var isLoading = false
    @State private var errorMessage = ""
    // ... logic เยอะมาก
    
    var body: some View { Text("") }
}
```

### 2. Separation of Concerns

```swift
// ✅ ดี: แยก Logic ออกจาก View
@Observable
class LoginViewModel {
    var email = ""
    var password = ""
    var isLoading = false
    var errorMessage: String? = nil
    var isLoggedIn = false
    
    var isValid: Bool {
        !email.isEmpty && password.count >= 8
    }
    
    func login() async {
        guard isValid else {
            errorMessage = "กรุณากรอกข้อมูลให้ครบ"
            return
        }
        
        isLoading = true
        errorMessage = nil
        
        do {
            // เรียก API
            try await AuthService.shared.login(email: email, password: password)
            isLoggedIn = true
        } catch {
            errorMessage = error.localizedDescription
        }
        
        isLoading = false
    }
}

struct LoginView: View {
    @State private var viewModel = LoginViewModel()
    
    var body: some View {
        VStack(spacing: 20) {
            TextField("อีเมล", text: $viewModel.email)
                .textFieldStyle(.roundedBorder)
                .keyboardType(.emailAddress)
            
            SecureField("รหัสผ่าน", text: $viewModel.password)
                .textFieldStyle(.roundedBorder)
            
            if let error = viewModel.errorMessage {
                Text(error)
                    .foregroundColor(.red)
                    .font(.caption)
            }
            
            Button("เข้าสู่ระบบ") {
                Task {
                    await viewModel.login()
                }
            }
            .buttonStyle(.borderedProminent)
            .disabled(!viewModel.isValid || viewModel.isLoading)
            
            if viewModel.isLoading {
                ProgressView()
            }
        }
        .padding()
    }
}
```

### 3. ประกาศ @State เป็น private เสมอ

```swift
struct MyView: View {
    // ✅ ถูกต้อง: private
    @State private var count = 0
    
    // ❌ ผิด: public (ไม่มีประโยชน์ เพราะ View เป็น struct)
    @State var count2 = 0
    
    var body: some View { Text("\(count)") }
}
```

### 4. ใช้ Computed Properties แทนการ derive state

```swift
@Observable
class CartViewModel {
    var items: [CartItem] = []
    
    // ✅ ดี: computed property
    var totalPrice: Double {
        items.reduce(0) { $0 + $1.price * Double($1.quantity) }
    }
    
    var isCartEmpty: Bool {
        items.isEmpty
    }
    
    var itemCount: Int {
        items.reduce(0) { $0 + $1.quantity }
    }
    
    // ❌ ควรหลีกเลี่ยง: stored derived state
    // @Published var totalPrice = 0.0  // ต้อง update ทุกครั้ง
}
```

### 5. ใช้ Task สำหรับ async operations

```swift
struct AsyncView: View {
    @State private var data: [String] = []
    @State private var isLoading = false
    
    var body: some View {
        List(data, id: \.self) { item in
            Text(item)
        }
        .overlay {
            if isLoading {
                ProgressView()
            }
        }
        .task {
            // ✅ ดี: ใช้ .task modifier
            isLoading = true
            data = await loadData()
            isLoading = false
        }
    }
    
    func loadData() async -> [String] {
        try? await Task.sleep(nanoseconds: 1_000_000_000)
        return ["ข้อมูล 1", "ข้อมูล 2", "ข้อมูล 3"]
    }
}
```

---

## 19. Common Mistakes (ข้อผิดพลาดที่พบบ่อย)

### ข้อผิดพลาดที่ 1: แก้ไข State นอก body

```swift
struct WrongView: View {
    @State private var value = 0
    
    // ❌ ผิด: ไม่สามารถแก้ไข state นอก body ได้
    func setup() {
        // value = 10  // compile error ถ้าไม่ใช่ mutating
    }
    
    // ✅ ถูกต้อง: แก้ไขใน body หรือ action
    var body: some View {
        Button("ตั้งค่า") {
            value = 10  // ✅ OK ใน closure
        }
        .onAppear {
            value = 10  // ✅ OK ใน modifier
        }
    }
}
```

### ข้อผิดพลาดที่ 2: ลืม inject @EnvironmentObject

```swift
// ❌ ผิด: ลืม inject
struct BadApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()  // ContentView ใช้ @EnvironmentObject แต่ไม่ได้ inject
        }
    }
}

// ✅ ถูกต้อง
struct GoodApp: App {
    @StateObject private var appState = AppState()
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(appState)
        }
    }
}
```

### ข้อผิดพลาดที่ 3: ใช้ @ObservedObject แทน @StateObject

```swift
// ❌ ผิด: ใช้ @ObservedObject ตอน View เป็นเจ้าของ
struct WrongOwnershipView: View {
    @ObservedObject private var viewModel = MyViewModel()
    // viewModel อาจถูก re-create ทุกครั้ง!
    
    var body: some View { Text("") }
}

// ✅ ถูกต้อง
struct CorrectOwnershipView: View {
    @StateObject private var viewModel = MyViewModel()
    // viewModel ถูกสร้างครั้งเดียวและอยู่ตลอด
    
    var body: some View { Text("") }
}
```

### ข้อผิดพลาดที่ 4: ไม่ใช้ main thread สำหรับ UI update

```swift
class NetworkViewModel: ObservableObject {
    @Published var data: [String] = []
    
    // ❌ ผิด: อัปเดต UI จาก background thread
    func loadDataWrong() {
        URLSession.shared.dataTask(with: URL(string: "https://example.com")!) { data, _, _ in
            self.data = ["item1", "item2"]  // ❌ อาจ crash!
        }.resume()
    }
    
    // ✅ ถูกต้อง: ใช้ MainActor
    @MainActor
    func loadDataCorrect() async {
        let result = await fetchFromNetwork()
        data = result  // ✅ ทำงานบน main thread
    }
    
    func fetchFromNetwork() async -> [String] {
        return ["item1", "item2"]
    }
}
```

### ข้อผิดพลาดที่ 5: ทำซ้ำ Source of Truth

```swift
// ❌ ผิด: เก็บข้อมูลเดิมในหลายที่
struct WrongDoubleState: View {
    @State private var name = ""
    @State private var nameUppercased = ""  // derived state!
    
    var body: some View {
        TextField("ชื่อ", text: $name)
            .onChange(of: name) { _, new in
                nameUppercased = new.uppercased()  // ต้อง sync เอง
            }
    }
}

// ✅ ถูกต้อง: ใช้ computed property
struct CorrectSingleSource: View {
    @State private var name = ""
    
    var nameUppercased: String { name.uppercased() }  // derive จาก source
    
    var body: some View {
        TextField("ชื่อ", text: $name)
    }
}
```

---

## 20. แบบฝึกหัดพร้อมเฉลย

### แบบฝึกหัด 1: Counter App

สร้าง Counter App ที่มีฟีเจอร์:
- แสดงตัวเลข
- ปุ่ม + และ -
- ไม่ให้ลดต่ำกว่า 0
- ไม่ให้เพิ่มเกิน 100
- ปุ่มรีเซ็ต
- แสดงสีต่างกันตามค่า (0: แดง, 1-50: เขียว, 51-100: น้ำเงิน)

```swift
// เฉลย: Counter App
import SwiftUI

struct CounterApp: View {
    @State private var count = 0
    
    var counterColor: Color {
        switch count {
        case 0: return .red
        case 1...50: return .green
        default: return .blue
        }
    }
    
    var body: some View {
        VStack(spacing: 30) {
            Text("\(count)")
                .font(.system(size: 80, weight: .bold, design: .rounded))
                .foregroundColor(counterColor)
                .animation(.easeInOut, value: count)
                .frame(width: 200, height: 200)
                .background(
                    Circle()
                        .fill(counterColor.opacity(0.1))
                )
            
            HStack(spacing: 40) {
                Button {
                    if count > 0 {
                        withAnimation { count -= 1 }
                    }
                } label: {
                    Image(systemName: "minus.circle.fill")
                        .font(.system(size: 50))
                        .foregroundColor(count == 0 ? .gray : .red)
                }
                .disabled(count == 0)
                
                Button {
                    if count < 100 {
                        withAnimation { count += 1 }
                    }
                } label: {
                    Image(systemName: "plus.circle.fill")
                        .font(.system(size: 50))
                        .foregroundColor(count == 100 ? .gray : .green)
                }
                .disabled(count == 100)
            }
            
            Button("รีเซ็ต") {
                withAnimation { count = 0 }
            }
            .font(.headline)
            .foregroundColor(.secondary)
            
            Text(statusText)
                .font(.caption)
                .foregroundColor(.secondary)
        }
        .padding()
        .navigationTitle("Counter")
    }
    
    var statusText: String {
        switch count {
        case 0: return "เริ่มต้น"
        case 100: return "ถึงขีดสูงสุดแล้ว!"
        default: return "เหลืออีก \(100 - count) จาก 100"
        }
    }
}

#Preview {
    NavigationStack {
        CounterApp()
    }
}
```

### แบบฝึกหัด 2: Todo List App

สร้าง Todo List App ที่มีฟีเจอร์:
- เพิ่ม/ลบงาน
- Mark งานว่าเสร็จ/ไม่เสร็จ
- กรองแสดงตาม status
- นับจำนวนงานที่เหลือ
- บันทึกค่าใน UserDefaults

```swift
// เฉลย: Todo List App

struct TodoItem: Identifiable, Codable {
    let id: UUID
    var title: String
    var isCompleted: Bool
    var createdAt: Date
    var priority: Priority
    
    init(title: String, priority: Priority = .medium) {
        self.id = UUID()
        self.title = title
        self.isCompleted = false
        self.createdAt = Date()
        self.priority = priority
    }
    
    enum Priority: String, Codable, CaseIterable {
        case low = "ต่ำ"
        case medium = "กลาง"
        case high = "สูง"
        
        var color: Color {
            switch self {
            case .low: return .blue
            case .medium: return .orange
            case .high: return .red
            }
        }
        
        var icon: String {
            switch self {
            case .low: return "arrow.down.circle"
            case .medium: return "minus.circle"
            case .high: return "arrow.up.circle"
            }
        }
    }
}

@Observable
class TodoStore {
    var items: [TodoItem] = [] {
        didSet { save() }
    }
    var filter: Filter = .all
    var newItemTitle = ""
    var selectedPriority: TodoItem.Priority = .medium
    
    enum Filter: String, CaseIterable {
        case all = "ทั้งหมด"
        case active = "ยังไม่เสร็จ"
        case completed = "เสร็จแล้ว"
    }
    
    init() {
        load()
    }
    
    var filteredItems: [TodoItem] {
        switch filter {
        case .all: return items
        case .active: return items.filter { !$0.isCompleted }
        case .completed: return items.filter { $0.isCompleted }
        }
    }
    
    var activeCount: Int {
        items.filter { !$0.isCompleted }.count
    }
    
    func addItem() {
        let title = newItemTitle.trimmingCharacters(in: .whitespaces)
        guard !title.isEmpty else { return }
        
        let item = TodoItem(title: title, priority: selectedPriority)
        items.insert(item, at: 0)
        newItemTitle = ""
        selectedPriority = .medium
    }
    
    func toggleItem(_ item: TodoItem) {
        if let index = items.firstIndex(where: { $0.id == item.id }) {
            items[index].isCompleted.toggle()
        }
    }
    
    func deleteItems(at indexSet: IndexSet) {
        let filteredItems = self.filteredItems
        let itemsToDelete = indexSet.map { filteredItems[$0] }
        items.removeAll { item in
            itemsToDelete.contains { $0.id == item.id }
        }
    }
    
    func clearCompleted() {
        items.removeAll { $0.isCompleted }
    }
    
    private func save() {
        if let data = try? JSONEncoder().encode(items) {
            UserDefaults.standard.set(data, forKey: "todoItems")
        }
    }
    
    private func load() {
        if let data = UserDefaults.standard.data(forKey: "todoItems"),
           let savedItems = try? JSONDecoder().decode([TodoItem].self, from: data) {
            items = savedItems
        }
    }
}

struct TodoListApp: View {
    @State private var store = TodoStore()
    
    var body: some View {
        NavigationStack {
            VStack(spacing: 0) {
                // Input Area
                VStack(spacing: 12) {
                    HStack {
                        TextField("เพิ่มงานใหม่...", text: $store.newItemTitle)
                            .textFieldStyle(.roundedBorder)
                            .onSubmit { store.addItem() }
                        
                        Button("เพิ่ม", action: store.addItem)
                            .buttonStyle(.borderedProminent)
                            .disabled(store.newItemTitle.trimmingCharacters(in: .whitespaces).isEmpty)
                    }
                    
                    HStack {
                        Text("ความสำคัญ:")
                            .font(.caption)
                            .foregroundColor(.secondary)
                        
                        ForEach(TodoItem.Priority.allCases, id: \.self) { priority in
                            Button {
                                store.selectedPriority = priority
                            } label: {
                                Label(priority.rawValue, systemImage: priority.icon)
                                    .font(.caption)
                                    .padding(.horizontal, 8)
                                    .padding(.vertical, 4)
                                    .background(
                                        store.selectedPriority == priority ?
                                        priority.color.opacity(0.2) : Color.clear
                                    )
                                    .foregroundColor(priority.color)
                                    .clipShape(Capsule())
                                    .overlay(
                                        Capsule().stroke(
                                            store.selectedPriority == priority ? priority.color : Color.clear,
                                            lineWidth: 1
                                        )
                                    )
                            }
                        }
                    }
                }
                .padding()
                .background(Color(.systemGroupedBackground))
                
                // Filter
                Picker("กรอง", selection: $store.filter) {
                    ForEach(TodoStore.Filter.allCases, id: \.self) { filter in
                        Text(filter.rawValue).tag(filter)
                    }
                }
                .pickerStyle(.segmented)
                .padding()
                
                // List
                List {
                    ForEach(store.filteredItems) { item in
                        TodoItemRow(item: item, onToggle: { store.toggleItem(item) })
                    }
                    .onDelete { store.deleteItems(at: $0) }
                }
                .listStyle(.plain)
                
                // Footer
                if !store.items.isEmpty {
                    HStack {
                        Text("\(store.activeCount) งานที่เหลือ")
                            .font(.caption)
                            .foregroundColor(.secondary)
                        
                        Spacer()
                        
                        if store.items.contains(where: { $0.isCompleted }) {
                            Button("ลบที่เสร็จแล้ว") {
                                withAnimation {
                                    store.clearCompleted()
                                }
                            }
                            .font(.caption)
                            .foregroundColor(.red)
                        }
                    }
                    .padding()
                    .background(Color(.systemGroupedBackground))
                }
            }
            .navigationTitle("รายการงาน")
        }
    }
}

struct TodoItemRow: View {
    let item: TodoItem
    let onToggle: () -> Void
    
    var body: some View {
        HStack(spacing: 12) {
            Button(action: onToggle) {
                Image(systemName: item.isCompleted ? "checkmark.circle.fill" : "circle")
                    .font(.title2)
                    .foregroundColor(item.isCompleted ? .green : .gray)
            }
            .buttonStyle(.plain)
            
            VStack(alignment: .leading, spacing: 2) {
                Text(item.title)
                    .strikethrough(item.isCompleted)
                    .foregroundColor(item.isCompleted ? .secondary : .primary)
                
                Text(item.createdAt.formatted(date: .abbreviated, time: .shortened))
                    .font(.caption2)
                    .foregroundColor(.secondary)
            }
            
            Spacer()
            
            Image(systemName: item.priority.icon)
                .foregroundColor(item.priority.color)
                .font(.caption)
        }
        .padding(.vertical, 4)
        .animation(.easeInOut, value: item.isCompleted)
    }
}

#Preview {
    TodoListApp()
}
```

### แบบฝึกหัด 3: Form App with Validation

สร้าง Form App ที่มีการตรวจสอบข้อมูล

```swift
// เฉลย: Registration Form with Validation

struct ValidationResult {
    var isValid: Bool
    var message: String
}

@Observable
class RegistrationViewModel {
    var firstName = ""
    var lastName = ""
    var email = ""
    var password = ""
    var confirmPassword = ""
    var birthDate = Date()
    var agreedToTerms = false
    var isSubmitted = false
    var showingAlert = false
    
    var firstNameValidation: ValidationResult {
        if firstName.isEmpty {
            return ValidationResult(isValid: false, message: "กรุณากรอกชื่อ")
        }
        if firstName.count < 2 {
            return ValidationResult(isValid: false, message: "ชื่อต้องมีอย่างน้อย 2 ตัวอักษร")
        }
        return ValidationResult(isValid: true, message: "")
    }
    
    var emailValidation: ValidationResult {
        if email.isEmpty {
            return ValidationResult(isValid: false, message: "กรุณากรอกอีเมล")
        }
        let emailRegex = "[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,64}"
        let emailPredicate = NSPredicate(format: "SELF MATCHES %@", emailRegex)
        if !emailPredicate.evaluate(with: email) {
            return ValidationResult(isValid: false, message: "รูปแบบอีเมลไม่ถูกต้อง")
        }
        return ValidationResult(isValid: true, message: "")
    }
    
    var passwordValidation: ValidationResult {
        if password.isEmpty {
            return ValidationResult(isValid: false, message: "กรุณากรอกรหัสผ่าน")
        }
        if password.count < 8 {
            return ValidationResult(isValid: false, message: "รหัสผ่านต้องมีอย่างน้อย 8 ตัว")
        }
        if !password.contains(where: { $0.isNumber }) {
            return ValidationResult(isValid: false, message: "ต้องมีตัวเลขอย่างน้อย 1 ตัว")
        }
        return ValidationResult(isValid: true, message: "")
    }
    
    var confirmPasswordValidation: ValidationResult {
        if confirmPassword.isEmpty {
            return ValidationResult(isValid: false, message: "กรุณายืนยันรหัสผ่าน")
        }
        if password != confirmPassword {
            return ValidationResult(isValid: false, message: "รหัสผ่านไม่ตรงกัน")
        }
        return ValidationResult(isValid: true, message: "")
    }
    
    var isFormValid: Bool {
        firstNameValidation.isValid &&
        emailValidation.isValid &&
        passwordValidation.isValid &&
        confirmPasswordValidation.isValid &&
        agreedToTerms
    }
    
    func submit() {
        guard isFormValid else { return }
        isSubmitted = true
        showingAlert = true
    }
}

struct RegistrationFormView: View {
    @State private var vm = RegistrationViewModel()
    
    var body: some View {
        NavigationStack {
            Form {
                Section("ข้อมูลส่วนตัว") {
                    ValidatedField(
                        title: "ชื่อ",
                        text: $vm.firstName,
                        validation: vm.firstNameValidation,
                        keyboardType: .default
                    )
                    
                    ValidatedField(
                        title: "นามสกุล",
                        text: $vm.lastName,
                        validation: ValidationResult(isValid: true, message: ""),
                        keyboardType: .default
                    )
                    
                    DatePicker("วันเกิด",
                              selection: $vm.birthDate,
                              in: ...Date(),
                              displayedComponents: .date)
                }
                
                Section("ข้อมูลบัญชี") {
                    ValidatedField(
                        title: "อีเมล",
                        text: $vm.email,
                        validation: vm.emailValidation,
                        keyboardType: .emailAddress
                    )
                    
                    ValidatedSecureField(
                        title: "รหัสผ่าน",
                        text: $vm.password,
                        validation: vm.passwordValidation
                    )
                    
                    ValidatedSecureField(
                        title: "ยืนยันรหัสผ่าน",
                        text: $vm.confirmPassword,
                        validation: vm.confirmPasswordValidation
                    )
                }
                
                Section {
                    Toggle("ยอมรับเงื่อนไขการใช้งาน", isOn: $vm.agreedToTerms)
                }
                
                Section {
                    Button("ลงทะเบียน") {
                        vm.submit()
                    }
                    .frame(maxWidth: .infinity)
                    .disabled(!vm.isFormValid)
                }
            }
            .navigationTitle("ลงทะเบียน")
            .alert("ลงทะเบียนสำเร็จ", isPresented: $vm.showingAlert) {
                Button("ตกลง") { }
            } message: {
                Text("ยินดีต้อนรับ \(vm.firstName) \(vm.lastName)!")
            }
        }
    }
}

struct ValidatedField: View {
    let title: String
    @Binding var text: String
    let validation: ValidationResult
    let keyboardType: UIKeyboardType
    @State private var isTouched = false
    
    var showError: Bool {
        isTouched && !validation.isValid && !text.isEmpty
    }
    
    var body: some View {
        VStack(alignment: .leading, spacing: 4) {
            TextField(title, text: $text)
                .keyboardType(keyboardType)
                .autocapitalization(keyboardType == .emailAddress ? .none : .words)
                .onChange(of: text) { _, _ in isTouched = true }
            
            if showError {
                Text(validation.message)
                    .font(.caption)
                    .foregroundColor(.red)
            }
        }
    }
}

struct ValidatedSecureField: View {
    let title: String
    @Binding var text: String
    let validation: ValidationResult
    @State private var isTouched = false
    @State private var isVisible = false
    
    var showError: Bool {
        isTouched && !validation.isValid && !text.isEmpty
    }
    
    var body: some View {
        VStack(alignment: .leading, spacing: 4) {
            HStack {
                Group {
                    if isVisible {
                        TextField(title, text: $text)
                    } else {
                        SecureField(title, text: $text)
                    }
                }
                .onChange(of: text) { _, _ in isTouched = true }
                
                Button {
                    isVisible.toggle()
                } label: {
                    Image(systemName: isVisible ? "eye.slash" : "eye")
                        .foregroundColor(.secondary)
                }
            }
            
            if showError {
                Text(validation.message)
                    .font(.caption)
                    .foregroundColor(.red)
            }
        }
    }
}

#Preview {
    RegistrationFormView()
}
```

---

## 21. สรุป

ในบทนี้เราได้เรียนรู้การจัดการ State ใน SwiftUI ซึ่งเป็นหัวใจของการพัฒนาแอปด้วย declarative programming

### สิ่งที่เรียนรู้

1. **@State** - เก็บ state ง่ายๆ ภายใน View เดียว
2. **@Binding** - แชร์ state กับ View ลูกแบบสองทาง
3. **ObservableObject + @Published** - สร้าง observable class แบบเดิม
4. **@StateObject** - สร้างและเป็นเจ้าของ ObservableObject
5. **@ObservedObject** - รับ ObservableObject จากภายนอก
6. **@EnvironmentObject** - แชร์ object กับทุก View ในลำดับชั้น
7. **@Observable** (Swift 5.9+) - วิธีใหม่ที่ดีกว่าและมีประสิทธิภาพกว่า
8. **@Bindable** - สร้าง binding กับ @Observable objects
9. **@AppStorage** - บันทึกค่าถาวรใน UserDefaults
10. **@SceneStorage** - บันทึกค่าตาม Scene
11. **Preference Keys** - ส่งข้อมูลจากลูกขึ้นสู่แม่
12. **Environment Values** - อ่านและสร้าง environment values

### หลักการสำคัญ

- **Single Source of Truth**: ข้อมูลควรมีแหล่งที่มาเดียว
- **Declarative**: บอก SwiftUI ว่าต้องการแสดงอะไร ไม่ใช่วิธีแสดง
- **Separation of Concerns**: แยก Logic ออกจาก View
- **Unidirectional Data Flow**: ข้อมูลไหลทางเดียว ง่ายต่อการ debug

### บทถัดไป

บทที่ 24 จะพาไปเรียนรู้ **SwiftUI Navigation** ซึ่งเป็นระบบนำทางระหว่างหน้าต่างๆ ในแอป รวมถึง NavigationStack, NavigationSplitView, TabView และการนำทางแบบ programmatic

---

*จบบทที่ 23: SwiftUI State Management*
