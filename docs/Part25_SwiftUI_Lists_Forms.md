# ส่วนที่ 25: SwiftUI Lists และ Forms

## บทนำ

ในส่วนนี้เราจะเรียนรู้เกี่ยวกับ **List** และ **Form** ซึ่งเป็น View ที่สำคัญมากใน SwiftUI สำหรับการแสดงข้อมูลแบบรายการและการสร้างหน้าจอตั้งค่า List ช่วยให้เราแสดงข้อมูลจำนวนมากได้อย่างมีประสิทธิภาพ ในขณะที่ Form เหมาะสำหรับการรับ input จากผู้ใช้

---

## 1. List Fundamentals (พื้นฐานของ List)

**List** ใน SwiftUI คือ View ที่แสดงข้อมูลเป็นแถวในแนวตั้ง คล้ายกับ `UITableView` ใน UIKit แต่ง่ายกว่ามากในการใช้งาน

### ลักษณะสำคัญของ List

- แสดงข้อมูลเป็นแถวแนวตั้ง (scrollable)
- รองรับการ swipe เพื่อลบหรือทำ action อื่นๆ
- รองรับ Edit Mode สำหรับการเรียงลำดับและลบ
- มี style หลายแบบให้เลือก
- ประสิทธิภาพดีกว่า ScrollView + VStack สำหรับข้อมูลจำนวนมาก

```swift
import SwiftUI

struct BasicListView: View {
    var body: some View {
        List {
            Text("รายการที่ 1")
            Text("รายการที่ 2")
            Text("รายการที่ 3")
        }
    }
}
```

### โครงสร้างพื้นฐาน

```swift
// List ง่ายๆ
struct SimpleListExample: View {
    var body: some View {
        NavigationStack {
            List {
                Label("บ้าน", systemImage: "house.fill")
                Label("ดาว", systemImage: "star.fill")
                Label("หัวใจ", systemImage: "heart.fill")
            }
            .navigationTitle("รายการ")
        }
    }
}
```

---

## 2. Static Lists (List แบบ Static)

Static List คือ List ที่กำหนดรายการไว้ตายตัวในโค้ด ไม่ได้มาจากข้อมูลแบบ dynamic

```swift
struct StaticListView: View {
    var body: some View {
        NavigationStack {
            List {
                // Row แบบ text ธรรมดา
                Text("ข้าวผัด")
                
                // Row แบบ Label
                Label("โทรศัพท์", systemImage: "phone.fill")
                
                // Row แบบ HStack
                HStack {
                    Image(systemName: "star.fill")
                        .foregroundColor(.yellow)
                    Text("รายการโปรด")
                    Spacer()
                    Text("5 รายการ")
                        .foregroundColor(.secondary)
                }
                
                // Row ที่ navigate ไปหน้าอื่น
                NavigationLink("รายละเอียด") {
                    Text("หน้ารายละเอียด")
                }
            }
            .navigationTitle("Static List")
        }
    }
}
```

### Static List พร้อม Section

```swift
struct StaticSectionedList: View {
    var body: some View {
        NavigationStack {
            List {
                Section("ผลไม้") {
                    Text("แอปเปิ้ล")
                    Text("กล้วย")
                    Text("มะม่วง")
                }
                
                Section("ผัก") {
                    Text("ผักโขม")
                    Text("แครอท")
                    Text("บรอกโคลี")
                }
                
                Section {
                    Text("รายการอื่นๆ")
                } header: {
                    Text("หมวดหมู่อื่น")
                } footer: {
                    Text("ข้อมูล ณ วันที่ 30 กันยายน 2569")
                }
            }
            .navigationTitle("Static Sections")
        }
    }
}
```

---

## 3. Dynamic Lists with ForEach (List แบบ Dynamic)

Dynamic List ใช้ข้อมูลจาก Array หรือ Collection โดยใช้ `ForEach` ภายใน List

### ForEach พื้นฐาน

```swift
struct DynamicListView: View {
    let fruits = ["แอปเปิ้ล", "กล้วย", "มะม่วง", "สตรอเบอร์รี่", "องุ่น"]
    
    var body: some View {
        NavigationStack {
            List {
                ForEach(fruits, id: \.self) { fruit in
                    Text(fruit)
                }
            }
            .navigationTitle("ผลไม้")
        }
    }
}
```

### List กับ ForEach โดยตรง

เมื่อ List มีแค่ ForEach เดียว สามารถเขียนแบบ shorthand ได้

```swift
struct ShorthandListView: View {
    let fruits = ["แอปเปิ้ล", "กล้วย", "มะม่วง", "สตรอเบอร์รี่", "องุ่น"]
    
    var body: some View {
        NavigationStack {
            // แบบย่อ - ส่ง array เข้า List โดยตรง
            List(fruits, id: \.self) { fruit in
                Text(fruit)
            }
            .navigationTitle("ผลไม้")
        }
    }
}
```

---

## 4. Identifiable Protocol

`Identifiable` Protocol กำหนดให้ type ต้องมี property `id` ที่ไม่ซ้ำกัน เมื่อ conform protocol นี้แล้ว ไม่จำเป็นต้องระบุ `id:` parameter ใน ForEach หรือ List

```swift
// กำหนด struct ที่ conform Identifiable
struct Product: Identifiable {
    let id: UUID  // ต้องมี property ชื่อ id
    let name: String
    let price: Double
    let category: String
    
    // สร้าง initializer ที่สะดวก
    init(name: String, price: Double, category: String) {
        self.id = UUID()
        self.name = name
        self.price = price
        self.category = category
    }
}

struct ProductListView: View {
    let products = [
        Product(name: "iPhone 16 Pro", price: 45900, category: "สมาร์ทโฟน"),
        Product(name: "MacBook Air M3", price: 42900, category: "คอมพิวเตอร์"),
        Product(name: "AirPods Pro", price: 9990, category: "หูฟัง"),
        Product(name: "iPad Air", price: 22900, category: "แท็บเล็ต"),
        Product(name: "Apple Watch", price: 14900, category: "สมาร์ทวอทช์")
    ]
    
    var body: some View {
        NavigationStack {
            // ไม่ต้องระบุ id: เพราะ Product conform Identifiable แล้ว
            List(products) { product in
                VStack(alignment: .leading, spacing: 4) {
                    Text(product.name)
                        .font(.headline)
                    HStack {
                        Text(product.category)
                            .font(.caption)
                            .foregroundColor(.secondary)
                        Spacer()
                        Text("฿\(product.price, format: .number)")
                            .font(.subheadline)
                            .foregroundColor(.blue)
                    }
                }
                .padding(.vertical, 4)
            }
            .navigationTitle("สินค้า")
        }
    }
}
```

### ใช้ Int เป็น id

```swift
struct City: Identifiable {
    let id: Int  // ใช้ Int แทน UUID ได้
    let name: String
    let country: String
    let population: Int
}

// หรือใช้ String เป็น id
struct Country: Identifiable {
    let id: String  // ใช้ code ประเทศ เช่น "TH", "US"
    let name: String
    let continent: String
}

struct CountryListView: View {
    let countries = [
        Country(id: "TH", name: "ไทย", continent: "เอเชีย"),
        Country(id: "JP", name: "ญี่ปุ่น", continent: "เอเชีย"),
        Country(id: "US", name: "สหรัฐอเมริกา", continent: "อเมริกาเหนือ"),
        Country(id: "GB", name: "สหราชอาณาจักร", continent: "ยุโรป")
    ]
    
    var body: some View {
        List(countries) { country in
            HStack {
                Text(country.id)
                    .font(.system(.body, design: .monospaced))
                    .foregroundColor(.secondary)
                    .frame(width: 40)
                VStack(alignment: .leading) {
                    Text(country.name)
                        .font(.headline)
                    Text(country.continent)
                        .font(.caption)
                        .foregroundColor(.secondary)
                }
            }
        }
    }
}
```

---

## 5. id Parameter ใน ForEach

เมื่อข้อมูลไม่ได้ conform `Identifiable` เราต้องระบุ `id:` parameter เพื่อบอก SwiftUI ว่าจะใช้ property ใดเป็น identifier

```swift
struct ForEachIDExample: View {
    // String conform Hashable แต่ไม่ conform Identifiable
    let colors = ["แดง", "เขียว", "น้ำเงิน", "เหลือง", "ม่วง"]
    
    // Int array
    let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    
    var body: some View {
        List {
            Section("สีต่างๆ") {
                // ใช้ \.self เมื่อ element เอง เป็น unique identifier
                ForEach(colors, id: \.self) { color in
                    Text(color)
                }
            }
            
            Section("ตัวเลข") {
                ForEach(numbers, id: \.self) { number in
                    Text("เลข \(number)")
                }
            }
        }
    }
}
```

### ระวัง: ซ้ำกันได้ทำให้เกิดปัญหา

```swift
struct DuplicateWarning: View {
    // มีค่าซ้ำ - อาจเกิดปัญหา animation และ update
    let items = ["ข้าว", "ข้าว", "ก๋วยเตี๋ยว", "ก๋วยเตี๋ยว"]
    
    var body: some View {
        // ❌ ไม่ดี - มีค่าซ้ำ
        List(items, id: \.self) { item in
            Text(item)
        }
    }
}

// วิธีแก้: ใช้ enumerated() เพื่อได้ index ที่ unique
struct FixedDuplicateView: View {
    let items = ["ข้าว", "ข้าว", "ก๋วยเตี๋ยว", "ก๋วยเตี๋ยว"]
    
    var body: some View {
        // ✅ ดีกว่า - ใช้ offset เป็น id
        List(Array(items.enumerated()), id: \.offset) { index, item in
            Text("\(index + 1). \(item)")
        }
    }
}
```

---

## 6. List Styles

SwiftUI มี List style หลายแบบให้เลือกตามความต้องการของ UI

### Plain Style

```swift
struct PlainStyleList: View {
    let items = ["รายการ 1", "รายการ 2", "รายการ 3"]
    
    var body: some View {
        List(items, id: \.self) { item in
            Text(item)
        }
        .listStyle(.plain)  // หรือ PlainListStyle()
    }
}
```

### Grouped Style

```swift
struct GroupedStyleList: View {
    var body: some View {
        List {
            Section("กลุ่ม A") {
                Text("รายการ A1")
                Text("รายการ A2")
            }
            Section("กลุ่ม B") {
                Text("รายการ B1")
                Text("รายการ B2")
            }
        }
        .listStyle(.grouped)
    }
}
```

### Inset Style

```swift
struct InsetStyleList: View {
    let items = ["รายการ 1", "รายการ 2", "รายการ 3"]
    
    var body: some View {
        List(items, id: \.self) { item in
            Text(item)
        }
        .listStyle(.inset)
    }
}
```

### InsetGrouped Style

```swift
struct InsetGroupedStyleList: View {
    var body: some View {
        List {
            Section("กลุ่ม 1") {
                Text("รายการ 1-1")
                Text("รายการ 1-2")
            }
            Section("กลุ่ม 2") {
                Text("รายการ 2-1")
                Text("รายการ 2-2")
            }
        }
        .listStyle(.insetGrouped)  // สไตล์ที่นิยมใน iOS settings
    }
}
```

### Sidebar Style

```swift
struct SidebarStyleList: View {
    var body: some View {
        NavigationSplitView {
            List {
                Section("โปรดปราน") {
                    Label("บ้าน", systemImage: "house.fill")
                    Label("ดาว", systemImage: "star.fill")
                }
                Section("การจัดการ") {
                    Label("เอกสาร", systemImage: "doc.fill")
                    Label("รูปภาพ", systemImage: "photo.fill")
                }
            }
            .listStyle(.sidebar)
            .navigationTitle("เมนู")
        } detail: {
            Text("เลือกเมนูทางซ้าย")
        }
    }
}
```

### เปรียบเทียบ Styles ทั้งหมด

```swift
struct AllStylesComparison: View {
    @State private var selectedStyle = "insetGrouped"
    
    let styles = ["plain", "grouped", "inset", "insetGrouped", "sidebar"]
    
    var body: some View {
        NavigationStack {
            VStack {
                Picker("Style", selection: $selectedStyle) {
                    ForEach(styles, id: \.self) { style in
                        Text(style).tag(style)
                    }
                }
                .pickerStyle(.segmented)
                .padding()
                
                // แสดง List ตาม style ที่เลือก
                listForStyle(selectedStyle)
            }
            .navigationTitle("List Styles")
        }
    }
    
    @ViewBuilder
    func listForStyle(_ style: String) -> some View {
        switch style {
        case "plain":
            sampleList.listStyle(.plain)
        case "grouped":
            sampleList.listStyle(.grouped)
        case "inset":
            sampleList.listStyle(.inset)
        case "insetGrouped":
            sampleList.listStyle(.insetGrouped)
        default:
            sampleList.listStyle(.plain)
        }
    }
    
    var sampleList: some View {
        List {
            Section("ส่วนที่ 1") {
                Text("รายการ A")
                Text("รายการ B")
            }
            Section("ส่วนที่ 2") {
                Text("รายการ C")
                Text("รายการ D")
            }
        }
    }
}
```

---

## 7. Section Headers and Footers

`Section` ใน List ช่วยจัดกลุ่มรายการและสามารถมี header และ footer ได้

### Header แบบ String

```swift
struct SectionHeaderExample: View {
    var body: some View {
        List {
            // Header แบบง่าย (String)
            Section("รายการโปรด") {
                Label("บ้าน", systemImage: "house")
                Label("งาน", systemImage: "briefcase")
            }
        }
    }
}
```

### Header แบบ Custom View

```swift
struct CustomHeaderList: View {
    var body: some View {
        List {
            Section {
                Text("รายการ 1")
                Text("รายการ 2")
                Text("รายการ 3")
            } header: {
                HStack {
                    Image(systemName: "star.fill")
                        .foregroundColor(.yellow)
                    Text("รายการพิเศษ")
                        .font(.headline)
                        .foregroundColor(.primary)
                    Spacer()
                    Button("ดูทั้งหมด") {
                        print("ดูทั้งหมด")
                    }
                    .font(.caption)
                }
            } footer: {
                Text("* รายการเหล่านี้จะอัปเดตทุกสัปดาห์")
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
        }
        .listStyle(.insetGrouped)
    }
}
```

### Header แบบ Dynamic

```swift
struct DynamicSectionHeader: View {
    struct Category: Identifiable {
        let id = UUID()
        let name: String
        let icon: String
        let items: [String]
    }
    
    let categories = [
        Category(name: "อาหาร", icon: "fork.knife", 
                items: ["ข้าวผัด", "ต้มยำ", "ผัดไทย"]),
        Category(name: "เครื่องดื่ม", icon: "cup.and.saucer",
                items: ["กาแฟ", "ชา", "น้ำผลไม้"]),
        Category(name: "ของหวาน", icon: "birthday.cake",
                items: ["ไอศกรีม", "เค้ก", "ขนมไทย"])
    ]
    
    var body: some View {
        NavigationStack {
            List {
                ForEach(categories) { category in
                    Section {
                        ForEach(category.items, id: \.self) { item in
                            Text(item)
                        }
                    } header: {
                        Label(category.name, systemImage: category.icon)
                    }
                }
            }
            .navigationTitle("เมนูอาหาร")
        }
    }
}
```

---

## 8. List Selection (การเลือกรายการ)

### Single Selection

```swift
struct SingleSelectionList: View {
    let fruits = ["แอปเปิ้ล", "กล้วย", "มะม่วง", "สตรอเบอร์รี่", "ส้ม"]
    @State private var selectedFruit: String?
    
    var body: some View {
        NavigationStack {
            List(fruits, id: \.self, selection: $selectedFruit) { fruit in
                Text(fruit)
            }
            .navigationTitle("เลือกผลไม้")
            .toolbar {
                if let selected = selectedFruit {
                    ToolbarItem(placement: .bottomBar) {
                        Text("เลือก: \(selected)")
                            .font(.headline)
                    }
                }
            }
        }
    }
}
```

### Multiple Selection

```swift
struct MultipleSelectionList: View {
    struct Task: Identifiable {
        let id = UUID()
        let title: String
        let priority: String
    }
    
    let tasks = [
        Task(title: "ซื้อของ", priority: "สูง"),
        Task(title: "โทรหาเพื่อน", priority: "กลาง"),
        Task(title: "อ่านหนังสือ", priority: "ต่ำ"),
        Task(title: "ออกกำลังกาย", priority: "สูง"),
        Task(title: "ทำการบ้าน", priority: "กลาง")
    ]
    
    @State private var selectedTasks: Set<Task.ID> = []
    @State private var isEditing = false
    
    var body: some View {
        NavigationStack {
            List(tasks, selection: $selectedTasks) { task in
                HStack {
                    VStack(alignment: .leading) {
                        Text(task.title)
                            .font(.headline)
                        Text("ความสำคัญ: \(task.priority)")
                            .font(.caption)
                            .foregroundColor(.secondary)
                    }
                    Spacer()
                }
            }
            .environment(\.editMode, .constant(isEditing ? .active : .inactive))
            .navigationTitle("งานที่ต้องทำ")
            .toolbar {
                ToolbarItem(placement: .topBarTrailing) {
                    Button(isEditing ? "เสร็จ" : "เลือก") {
                        isEditing.toggle()
                        if !isEditing {
                            selectedTasks.removeAll()
                        }
                    }
                }
                if isEditing && !selectedTasks.isEmpty {
                    ToolbarItem(placement: .bottomBar) {
                        Text("เลือกแล้ว \(selectedTasks.count) รายการ")
                    }
                }
            }
        }
    }
}
```

---

## 9. Swipe Actions (การ Swipe เพื่อทำ action)

Swipe Actions ช่วยให้ผู้ใช้ swipe แถวในรายการเพื่อทำ action ต่างๆ

### Swipe to Delete แบบพื้นฐาน

```swift
struct SwipeToDeleteList: View {
    @State private var items = ["รายการ 1", "รายการ 2", "รายการ 3", "รายการ 4", "รายการ 5"]
    
    var body: some View {
        NavigationStack {
            List {
                ForEach(items, id: \.self) { item in
                    Text(item)
                }
                .onDelete { indexSet in
                    items.remove(atOffsets: indexSet)
                }
            }
            .navigationTitle("Swipe to Delete")
            .toolbar {
                EditButton()
            }
        }
    }
}
```

### Custom Swipe Actions

```swift
struct CustomSwipeActionsView: View {
    struct Message: Identifiable {
        let id = UUID()
        var title: String
        var isRead: Bool
        var isStarred: Bool
    }
    
    @State private var messages = [
        Message(title: "ข้อความจาก Alice", isRead: false, isStarred: false),
        Message(title: "การประชุมวันพรุ่งนี้", isRead: true, isStarred: true),
        Message(title: "รายงานประจำเดือน", isRead: false, isStarred: false),
        Message(title: "ยืนยันการจองโรงแรม", isRead: true, isStarred: false),
        Message(title: "อัปเดตแอป SwiftUI", isRead: false, isStarred: true)
    ]
    
    var body: some View {
        NavigationStack {
            List {
                ForEach($messages) { $message in
                    HStack {
                        Circle()
                            .fill(message.isRead ? Color.clear : Color.blue)
                            .frame(width: 10, height: 10)
                        VStack(alignment: .leading) {
                            Text(message.title)
                                .font(.headline)
                                .fontWeight(message.isRead ? .regular : .bold)
                        }
                        Spacer()
                        if message.isStarred {
                            Image(systemName: "star.fill")
                                .foregroundColor(.yellow)
                        }
                    }
                    .swipeActions(edge: .trailing, allowsFullSwipe: true) {
                        // ปุ่มลบ (สีแดง)
                        Button(role: .destructive) {
                            if let index = messages.firstIndex(where: { $0.id == message.id }) {
                                messages.remove(at: index)
                            }
                        } label: {
                            Label("ลบ", systemImage: "trash")
                        }
                        
                        // ปุ่มเก็บถาวร (สีส้ม)
                        Button {
                            print("เก็บถาวร \(message.title)")
                        } label: {
                            Label("เก็บถาวร", systemImage: "archivebox")
                        }
                        .tint(.orange)
                    }
                    .swipeActions(edge: .leading) {
                        // ปุ่มทำเครื่องหมายอ่านแล้ว (สีน้ำเงิน)
                        Button {
                            message.isRead.toggle()
                        } label: {
                            Label(
                                message.isRead ? "ยังไม่ได้อ่าน" : "อ่านแล้ว",
                                systemImage: message.isRead ? "envelope.badge" : "envelope.open"
                            )
                        }
                        .tint(.blue)
                        
                        // ปุ่มติดดาว (สีเหลือง)
                        Button {
                            message.isStarred.toggle()
                        } label: {
                            Label(
                                message.isStarred ? "เลิกติดดาว" : "ติดดาว",
                                systemImage: message.isStarred ? "star.slash" : "star"
                            )
                        }
                        .tint(.yellow)
                    }
                }
            }
            .navigationTitle("ข้อความ")
        }
    }
}
```

---

## 10. Edit Mode (โหมดแก้ไข)

Edit Mode ใช้สำหรับจัดการรายการ เช่น การลบและการเรียงลำดับ

### EditButton แบบพื้นฐาน

```swift
struct EditModeList: View {
    @State private var fruits = ["แอปเปิ้ล", "กล้วย", "มะม่วง", "สตรอเบอร์รี่", "ส้ม"]
    
    var body: some View {
        NavigationStack {
            List {
                ForEach(fruits, id: \.self) { fruit in
                    Text(fruit)
                }
                .onDelete { indexSet in
                    fruits.remove(atOffsets: indexSet)
                }
                .onMove { fromOffsets, toOffset in
                    fruits.move(fromOffsets: fromOffsets, toOffset: toOffset)
                }
            }
            .navigationTitle("รายการผลไม้")
            .toolbar {
                // EditButton เป็น built-in button ที่จัดการ edit mode เอง
                EditButton()
            }
        }
    }
}
```

### Custom Edit Mode Control

```swift
struct CustomEditModeList: View {
    @State private var items = ["รายการ A", "รายการ B", "รายการ C", "รายการ D", "รายการ E"]
    @State private var editMode: EditMode = .inactive
    
    var body: some View {
        NavigationStack {
            List {
                ForEach(items, id: \.self) { item in
                    HStack {
                        Text(item)
                        Spacer()
                        // แสดง checkmark ใน edit mode
                        if editMode.isEditing {
                            Image(systemName: "checkmark.circle")
                                .foregroundColor(.blue)
                        }
                    }
                }
                .onDelete(perform: deleteItems)
                .onMove(perform: moveItems)
            }
            .environment(\.editMode, $editMode)
            .navigationTitle("Custom Edit Mode")
            .toolbar {
                ToolbarItem(placement: .topBarTrailing) {
                    Button(editMode.isEditing ? "เสร็จ" : "แก้ไข") {
                        withAnimation {
                            editMode = editMode.isEditing ? .inactive : .active
                        }
                    }
                }
            }
        }
    }
    
    func deleteItems(at indexSet: IndexSet) {
        items.remove(atOffsets: indexSet)
    }
    
    func moveItems(from source: IndexSet, to destination: Int) {
        items.move(fromOffsets: source, toOffset: destination)
    }
}
```

---

## 11. Move and Delete Items (การย้ายและลบรายการ)

### onDelete และ onMove

```swift
struct MoveDeleteExample: View {
    @State private var tasks = [
        "ซื้อของในซุปเปอร์มาร์เก็ต",
        "ทำรายงานส่งหัวหน้า",
        "โทรหาแม่",
        "นัดหมอฟัน",
        "จ่ายค่าไฟ",
        "ซักผ้า"
    ]
    
    var body: some View {
        NavigationStack {
            List {
                ForEach(tasks, id: \.self) { task in
                    Label(task, systemImage: "checkmark.square")
                }
                .onDelete(perform: deleteTasks)
                .onMove(perform: moveTasks)
            }
            .navigationTitle("งานที่ต้องทำ")
            .toolbar {
                ToolbarItem(placement: .topBarLeading) {
                    Button("เพิ่มงาน") {
                        tasks.insert("งานใหม่ \(tasks.count + 1)", at: 0)
                    }
                }
                ToolbarItem(placement: .topBarTrailing) {
                    EditButton()
                }
            }
        }
    }
    
    private func deleteTasks(at offsets: IndexSet) {
        tasks.remove(atOffsets: offsets)
    }
    
    private func moveTasks(from source: IndexSet, to destination: Int) {
        tasks.move(fromOffsets: source, toOffset: destination)
    }
}
```

### Delete ด้วย Confirmation Dialog

```swift
struct DeleteWithConfirmationView: View {
    struct Note: Identifiable {
        let id = UUID()
        var title: String
        var content: String
    }
    
    @State private var notes = [
        Note(title: "ไอเดียโปรเจค", content: "สร้างแอป Swift course"),
        Note(title: "สูตรอาหาร", content: "ผัดไทยไม่ใส่ผักชี"),
        Note(title: "รายการซื้อของ", content: "นม ไข่ ขนมปัง"),
        Note(title: "คำพูดโปรด", content: "ทำวันนี้ให้ดีที่สุด")
    ]
    
    @State private var noteToDelete: Note?
    @State private var showDeleteAlert = false
    
    var body: some View {
        NavigationStack {
            List {
                ForEach(notes) { note in
                    VStack(alignment: .leading) {
                        Text(note.title)
                            .font(.headline)
                        Text(note.content)
                            .font(.subheadline)
                            .foregroundColor(.secondary)
                            .lineLimit(1)
                    }
                    .swipeActions(edge: .trailing) {
                        Button(role: .destructive) {
                            noteToDelete = note
                            showDeleteAlert = true
                        } label: {
                            Label("ลบ", systemImage: "trash")
                        }
                    }
                }
            }
            .navigationTitle("โน้ต")
            .alert("ยืนยันการลบ", isPresented: $showDeleteAlert) {
                Button("ลบ", role: .destructive) {
                    if let note = noteToDelete,
                       let index = notes.firstIndex(where: { $0.id == note.id }) {
                        notes.remove(at: index)
                    }
                }
                Button("ยกเลิก", role: .cancel) {}
            } message: {
                Text("คุณต้องการลบ '\(noteToDelete?.title ?? "")' ใช่หรือไม่?")
            }
        }
    }
}
```

---

## 12. searchable Modifier

`.searchable` modifier เพิ่มช่องค้นหาให้กับ List หรือ NavigationStack

### การใช้งานพื้นฐาน

```swift
struct SearchableListView: View {
    let allFruits = [
        "แอปเปิ้ล", "กล้วย", "มะม่วง", "สตรอเบอร์รี่", "ส้ม",
        "องุ่น", "แตงโม", "มะละกอ", "สับปะรด", "ลิ้นจี่",
        "มังคุด", "ทุเรียน", "เงาะ", "ลำไย", "ชมพู่"
    ]
    
    @State private var searchText = ""
    
    var filteredFruits: [String] {
        if searchText.isEmpty {
            return allFruits
        }
        return allFruits.filter { $0.localizedCaseInsensitiveContains(searchText) }
    }
    
    var body: some View {
        NavigationStack {
            List(filteredFruits, id: \.self) { fruit in
                Text(fruit)
            }
            .navigationTitle("ผลไม้")
            .searchable(text: $searchText, prompt: "ค้นหาผลไม้")
        }
    }
}
```

### searchable พร้อม Suggestions

```swift
struct SearchWithSuggestionsView: View {
    struct Contact: Identifiable {
        let id = UUID()
        let name: String
        let phone: String
        let tag: String
    }
    
    let contacts = [
        Contact(name: "สมชาย ใจดี", phone: "081-234-5678", tag: "เพื่อน"),
        Contact(name: "สมหญิง รักเรียน", phone: "082-345-6789", tag: "ครอบครัว"),
        Contact(name: "อนุชา ทำงาน", phone: "083-456-7890", tag: "งาน"),
        Contact(name: "มนัส สร้างสรรค์", phone: "084-567-8901", tag: "งาน"),
        Contact(name: "นารี สุขใจ", phone: "085-678-9012", tag: "เพื่อน"),
        Contact(name: "วิชัย กล้าหาญ", phone: "086-789-0123", tag: "ครอบครัว")
    ]
    
    @State private var searchText = ""
    @State private var searchScope = "ทั้งหมด"
    
    let scopes = ["ทั้งหมด", "เพื่อน", "ครอบครัว", "งาน"]
    
    var filteredContacts: [Contact] {
        var result = contacts
        
        if searchScope != "ทั้งหมด" {
            result = result.filter { $0.tag == searchScope }
        }
        
        if !searchText.isEmpty {
            result = result.filter {
                $0.name.localizedCaseInsensitiveContains(searchText) ||
                $0.phone.contains(searchText)
            }
        }
        
        return result
    }
    
    var body: some View {
        NavigationStack {
            List(filteredContacts) { contact in
                HStack {
                    Circle()
                        .fill(tagColor(contact.tag))
                        .frame(width: 40, height: 40)
                        .overlay(
                            Text(String(contact.name.prefix(1)))
                                .foregroundColor(.white)
                                .font(.headline)
                        )
                    VStack(alignment: .leading) {
                        Text(contact.name)
                            .font(.headline)
                        Text(contact.phone)
                            .font(.subheadline)
                            .foregroundColor(.secondary)
                    }
                    Spacer()
                    Text(contact.tag)
                        .font(.caption)
                        .padding(.horizontal, 8)
                        .padding(.vertical, 4)
                        .background(tagColor(contact.tag).opacity(0.2))
                        .cornerRadius(8)
                }
            }
            .navigationTitle("ผู้ติดต่อ")
            .searchable(
                text: $searchText,
                placement: .navigationBarDrawer(displayMode: .always),
                prompt: "ค้นหาชื่อหรือเบอร์โทร"
            ) {
                // Suggestions
                ForEach(scopes, id: \.self) { scope in
                    Text(scope).searchCompletion(scope)
                }
            }
            .searchScopes($searchScope) {
                ForEach(scopes, id: \.self) { scope in
                    Text(scope).tag(scope)
                }
            }
        }
    }
    
    func tagColor(_ tag: String) -> Color {
        switch tag {
        case "เพื่อน": return .blue
        case "ครอบครัว": return .green
        case "งาน": return .orange
        default: return .gray
        }
    }
}
```

---

## 13. List Row Customization (การปรับแต่ง Row)

### listRowBackground

```swift
struct CustomRowBackground: View {
    let items = ["รายการ 1", "รายการ 2", "รายการ 3", "รายการ 4"]
    
    var body: some View {
        List(items, id: \.self) { item in
            Text(item)
                .padding()
                .listRowBackground(
                    RoundedRectangle(cornerRadius: 10)
                        .fill(Color.blue.opacity(0.1))
                        .padding(.horizontal, 4)
                )
        }
        .listStyle(.plain)
    }
}
```

### listRowSeparator

```swift
struct CustomRowSeparator: View {
    var body: some View {
        List {
            ForEach(1...5, id: \.self) { i in
                Text("รายการที่ \(i)")
                    // ซ่อน separator ด้านล่าง
                    .listRowSeparator(.hidden)
            }
            
            ForEach(6...10, id: \.self) { i in
                Text("รายการที่ \(i)")
                    // แสดง separator พร้อมกำหนดสี
                    .listRowSeparatorTint(.red)
            }
        }
    }
}
```

### listRowInsets

```swift
struct CustomRowInsets: View {
    var body: some View {
        List {
            ForEach(1...5, id: \.self) { i in
                Text("รายการที่ \(i)")
                    .listRowInsets(EdgeInsets(
                        top: 12,
                        leading: 24,
                        bottom: 12,
                        trailing: 24
                    ))
            }
        }
    }
}
```

### Custom Row ที่สมบูรณ์

```swift
struct FullyCustomRow: View {
    struct Article: Identifiable {
        let id = UUID()
        let title: String
        let author: String
        let date: Date
        let readTime: Int
        let category: String
        let isBookmarked: Bool
    }
    
    @State private var articles = [
        Article(title: "SwiftUI 6 คุณสมบัติใหม่", author: "นักพัฒนา Swift",
               date: Date(), readTime: 5, category: "iOS", isBookmarked: true),
        Article(title: "การใช้ async/await ใน Swift", author: "iOS Developer",
               date: Date().addingTimeInterval(-86400), readTime: 8, category: "Swift", isBookmarked: false),
        Article(title: "Design Patterns ใน SwiftUI", author: "อาจารย์ Swift",
               date: Date().addingTimeInterval(-172800), readTime: 12, category: "Architecture", isBookmarked: true)
    ]
    
    var body: some View {
        NavigationStack {
            List(articles) { article in
                ArticleRow(article: article)
                    .listRowInsets(EdgeInsets(top: 8, leading: 16, bottom: 8, trailing: 16))
                    .listRowBackground(Color.clear)
                    .listRowSeparator(.hidden)
            }
            .listStyle(.plain)
            .navigationTitle("บทความ")
        }
    }
}

struct ArticleRow: View {
    let article: FullyCustomRow.Article
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            HStack {
                Text(article.category)
                    .font(.caption)
                    .fontWeight(.semibold)
                    .padding(.horizontal, 8)
                    .padding(.vertical, 3)
                    .background(Color.blue.opacity(0.15))
                    .foregroundColor(.blue)
                    .cornerRadius(4)
                Spacer()
                if article.isBookmarked {
                    Image(systemName: "bookmark.fill")
                        .foregroundColor(.orange)
                        .font(.caption)
                }
            }
            
            Text(article.title)
                .font(.headline)
                .lineLimit(2)
            
            HStack {
                Text(article.author)
                    .font(.caption)
                    .foregroundColor(.secondary)
                Text("•")
                    .foregroundColor(.secondary)
                Text("\(article.readTime) นาที")
                    .font(.caption)
                    .foregroundColor(.secondary)
                Spacer()
                Text(article.date, style: .relative)
                    .font(.caption2)
                    .foregroundColor(.secondary)
            }
        }
        .padding()
        .background(Color(.systemBackground))
        .cornerRadius(12)
        .shadow(color: .black.opacity(0.05), radius: 4, x: 0, y: 2)
    }
}
```

---

## 14. Form (แบบฟอร์ม)

`Form` ใน SwiftUI เป็น View ที่ออกแบบมาสำหรับการรับ input จากผู้ใช้ มีลักษณะคล้าย List แต่เหมาะกับการสร้าง settings หรือ form กรอกข้อมูล

### Form พื้นฐาน

```swift
struct BasicFormView: View {
    @State private var firstName = ""
    @State private var lastName = ""
    @State private var age = 25
    @State private var isStudent = false
    
    var body: some View {
        NavigationStack {
            Form {
                Section("ข้อมูลส่วนตัว") {
                    TextField("ชื่อ", text: $firstName)
                    TextField("นามสกุล", text: $lastName)
                }
                
                Section("อื่นๆ") {
                    Stepper("อายุ: \(age) ปี", value: $age, in: 1...120)
                    Toggle("เป็นนักเรียน", isOn: $isStudent)
                }
            }
            .navigationTitle("แบบฟอร์ม")
        }
    }
}
```

---

## 15. Section ใน Form

```swift
struct SectionedFormView: View {
    @State private var username = ""
    @State private var email = ""
    @State private var password = ""
    @State private var confirmPassword = ""
    @State private var agreeToTerms = false
    @State private var receiveNewsletter = true
    
    var body: some View {
        NavigationStack {
            Form {
                Section {
                    TextField("ชื่อผู้ใช้", text: $username)
                        .autocapitalization(.none)
                        .autocorrectionDisabled()
                    TextField("อีเมล", text: $email)
                        .keyboardType(.emailAddress)
                        .autocapitalization(.none)
                } header: {
                    Text("ข้อมูลบัญชี")
                } footer: {
                    Text("ชื่อผู้ใช้ต้องมีอย่างน้อย 4 ตัวอักษร")
                }
                
                Section("รหัสผ่าน") {
                    SecureField("รหัสผ่าน", text: $password)
                    SecureField("ยืนยันรหัสผ่าน", text: $confirmPassword)
                }
                
                Section {
                    Toggle("ยอมรับข้อกำหนดการใช้งาน", isOn: $agreeToTerms)
                    Toggle("รับจดหมายข่าว", isOn: $receiveNewsletter)
                } footer: {
                    Text("โดยการสมัครสมาชิก คุณยอมรับนโยบายความเป็นส่วนตัวของเรา")
                }
                
                Section {
                    Button("สมัครสมาชิก") {
                        print("สมัครสมาชิก")
                    }
                    .frame(maxWidth: .infinity)
                    .disabled(!agreeToTerms || username.isEmpty || email.isEmpty)
                }
            }
            .navigationTitle("สมัครสมาชิก")
        }
    }
}
```

---

## 16. Form Inputs

### TextField และ SecureField

```swift
struct FormTextFieldExample: View {
    @State private var name = ""
    @State private var bio = ""
    @State private var website = ""
    @State private var password = ""
    
    var body: some View {
        Form {
            Section("ข้อมูลโปรไฟล์") {
                // TextField ธรรมดา
                TextField("ชื่อ", text: $name)
                
                // TextField หลายบรรทัด
                TextField("ประวัติส่วนตัว", text: $bio, axis: .vertical)
                    .lineLimit(3...6)
                
                // TextField สำหรับ URL
                TextField("เว็บไซต์", text: $website)
                    .keyboardType(.URL)
                    .autocapitalization(.none)
                    .autocorrectionDisabled()
            }
            
            Section("ความปลอดภัย") {
                // SecureField สำหรับรหัสผ่าน
                SecureField("รหัสผ่าน", text: $password)
            }
        }
    }
}
```

### Toggle

```swift
struct FormToggleExample: View {
    @State private var notifications = true
    @State private var darkMode = false
    @State private var locationAccess = true
    @State private var faceID = true
    @State private var autoBackup = false
    
    var body: some View {
        Form {
            Section("การแจ้งเตือน") {
                Toggle("เปิดการแจ้งเตือน", isOn: $notifications)
                    .tint(.blue)
                
                if notifications {
                    Toggle("เสียงแจ้งเตือน", isOn: .constant(true))
                    Toggle("การสั่น", isOn: .constant(true))
                    Toggle("แสดงในหน้าล็อก", isOn: .constant(false))
                }
            }
            
            Section("การแสดงผล") {
                Toggle("โหมดมืด", isOn: $darkMode)
                    .tint(.purple)
            }
            
            Section("ความเป็นส่วนตัว") {
                Toggle("อนุญาตตำแหน่ง", isOn: $locationAccess)
                Toggle("Face ID", isOn: $faceID)
                Toggle("สำรองข้อมูลอัตโนมัติ", isOn: $autoBackup)
            }
        }
    }
}
```

### Picker

```swift
struct FormPickerExample: View {
    @State private var selectedColor = "น้ำเงิน"
    @State private var selectedFont = "System"
    @State private var fontSize = 16.0
    
    let colors = ["แดง", "เขียว", "น้ำเงิน", "เหลือง", "ม่วง"]
    let fonts = ["System", "Helvetica", "Georgia", "Courier"]
    
    var body: some View {
        Form {
            Section("ธีม") {
                Picker("สีหลัก", selection: $selectedColor) {
                    ForEach(colors, id: \.self) { color in
                        Text(color).tag(color)
                    }
                }
                
                Picker("แบบตัวอักษร", selection: $selectedFont) {
                    ForEach(fonts, id: \.self) { font in
                        Text(font)
                            .font(.custom(font == "System" ? ".SF UI Text" : font, size: 14))
                            .tag(font)
                    }
                }
                .pickerStyle(.menu)
            }
            
            Section("ตัวอักษร") {
                Slider(value: $fontSize, in: 12...24, step: 1) {
                    Text("ขนาด: \(Int(fontSize))pt")
                } minimumValueLabel: {
                    Text("12")
                } maximumValueLabel: {
                    Text("24")
                }
                
                Text("ตัวอย่าง: นี่คือข้อความทดสอบ")
                    .font(.system(size: fontSize))
            }
        }
    }
}
```

### DatePicker

```swift
struct FormDatePickerExample: View {
    @State private var birthDate = Date()
    @State private var appointmentDate = Date()
    @State private var reminderTime = Date()
    
    var body: some View {
        Form {
            Section("วันเกิด") {
                DatePicker(
                    "วันเกิด",
                    selection: $birthDate,
                    displayedComponents: .date
                )
                .datePickerStyle(.compact)
            }
            
            Section("นัดหมาย") {
                DatePicker(
                    "วันและเวลา",
                    selection: $appointmentDate,
                    in: Date()...,  // ตั้งแต่วันนี้เป็นต้นไป
                    displayedComponents: [.date, .hourAndMinute]
                )
                .datePickerStyle(.graphical)
            }
            
            Section("เตือนความจำ") {
                DatePicker(
                    "เวลา",
                    selection: $reminderTime,
                    displayedComponents: .hourAndMinute
                )
                .datePickerStyle(.wheel)
            }
        }
    }
}
```

### Slider และ Stepper

```swift
struct FormSliderStepperExample: View {
    @State private var volume = 0.5
    @State private var brightness = 0.8
    @State private var itemCount = 5
    @State private var pages = 1
    
    var body: some View {
        Form {
            Section("การควบคุมเสียง") {
                Slider(value: $volume) {
                    Text("ระดับเสียง")
                } minimumValueLabel: {
                    Image(systemName: "speaker")
                } maximumValueLabel: {
                    Image(systemName: "speaker.3")
                }
                
                Text("ระดับเสียง: \(Int(volume * 100))%")
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
            
            Section("ความสว่าง") {
                Slider(value: $brightness, in: 0...1, step: 0.1) {
                    Text("ความสว่าง")
                } minimumValueLabel: {
                    Image(systemName: "sun.min")
                } maximumValueLabel: {
                    Image(systemName: "sun.max")
                }
            }
            
            Section("จำนวน") {
                Stepper("จำนวนสินค้า: \(itemCount)", value: $itemCount, in: 1...100)
                
                Stepper("จำนวนหน้า: \(pages)", value: $pages, in: 1...10, step: 1) {
                    print("เพิ่ม")
                } onDecrement: {
                    print("ลด")
                }
            }
        }
    }
}
```

---

## 17. Picker Styles (สไตล์ของ Picker)

```swift
struct PickerStylesExample: View {
    @State private var selection1 = 0
    @State private var selection2 = 0
    @State private var selection3 = 0
    @State private var selection4 = 0
    
    let options = ["ตัวเลือก 1", "ตัวเลือก 2", "ตัวเลือก 3", "ตัวเลือก 4"]
    
    var body: some View {
        Form {
            Section("Menu Style (ค่าเริ่มต้น)") {
                Picker("เลือก", selection: $selection1) {
                    ForEach(options.indices, id: \.self) { i in
                        Text(options[i]).tag(i)
                    }
                }
                .pickerStyle(.menu)
            }
            
            Section("Segmented Style") {
                Picker("เลือก", selection: $selection2) {
                    Text("A").tag(0)
                    Text("B").tag(1)
                    Text("C").tag(2)
                }
                .pickerStyle(.segmented)
            }
            
            Section("Inline Style") {
                Picker("เลือก", selection: $selection3) {
                    ForEach(options.indices, id: \.self) { i in
                        Text(options[i]).tag(i)
                    }
                }
                .pickerStyle(.inline)
            }
            
            Section("Wheel Style") {
                Picker("เลือก", selection: $selection4) {
                    ForEach(options.indices, id: \.self) { i in
                        Text(options[i]).tag(i)
                    }
                }
                .pickerStyle(.wheel)
                .frame(height: 120)
            }
            
            Section("NavigationLink Style") {
                NavigationLink {
                    Form {
                        Picker("เลือก", selection: $selection1) {
                            ForEach(options.indices, id: \.self) { i in
                                Text(options[i]).tag(i)
                            }
                        }
                        .pickerStyle(.inline)
                    }
                    .navigationTitle("เลือกตัวเลือก")
                } label: {
                    HStack {
                        Text("เลือกตัวเลือก")
                        Spacer()
                        Text(options[selection1])
                            .foregroundColor(.secondary)
                    }
                }
            }
        }
        .navigationTitle("Picker Styles")
    }
}
```

---

## 18. ScrollView กับ LazyVStack เทียบกับ List

### ประสิทธิภาพ ScrollView + LazyVStack vs List

```swift
// ScrollView + LazyVStack
// - Lazy: สร้าง View เฉพาะที่ปรากฏบนหน้าจอ
// - ยืดหยุ่นกว่า List ในการจัด layout
// - ไม่มี built-in separator, swipe actions, edit mode
struct LazyVStackExample: View {
    let items = Array(1...1000)
    
    var body: some View {
        ScrollView {
            LazyVStack(spacing: 0) {
                ForEach(items, id: \.self) { item in
                    HStack {
                        Text("รายการ \(item)")
                        Spacer()
                    }
                    .padding()
                    .background(Color(.systemBackground))
                    Divider()
                }
            }
        }
    }
}

// List
// - มี built-in separator, swipe actions, edit mode, selection
// - เหมาะสำหรับข้อมูลรายการมาตรฐาน
// - ประสิทธิภาพดีสำหรับข้อมูลจำนวนมาก
struct ListComparisonExample: View {
    let items = Array(1...1000)
    
    var body: some View {
        List(items, id: \.self) { item in
            Text("รายการ \(item)")
        }
    }
}
```

### เมื่อควรใช้อะไร

```swift
// ใช้ List เมื่อ:
// 1. ต้องการ swipe actions
// 2. ต้องการ edit mode (delete, reorder)
// 3. ต้องการ selection
// 4. ข้อมูลเป็น standard list UI
// 5. ใช้ Section headers/footers

// ใช้ ScrollView + LazyVStack เมื่อ:
// 1. ต้องการ layout ที่ยืดหยุ่น
// 2. ต้องการ custom separators หรือ dividers
// 3. ต้องการ mixed content (ไม่ใช่แค่ rows)
// 4. ต้องการ horizontal scrolling ด้วย
// 5. สร้าง card-based UI

struct WhenToUseWhich: View {
    @State private var useList = true
    let items = Array(1...50)
    
    var body: some View {
        VStack(spacing: 0) {
            Picker("Layout", selection: $useList) {
                Text("List").tag(true)
                Text("LazyVStack").tag(false)
            }
            .pickerStyle(.segmented)
            .padding()
            
            if useList {
                List(items, id: \.self) { item in
                    Text("รายการ \(item)")
                }
            } else {
                ScrollView {
                    LazyVStack(spacing: 8) {
                        ForEach(items, id: \.self) { item in
                            RoundedRectangle(cornerRadius: 10)
                                .fill(Color.blue.opacity(0.1))
                                .frame(height: 60)
                                .overlay(
                                    Text("รายการ \(item)")
                                        .foregroundColor(.blue)
                                )
                        }
                        .padding(.horizontal)
                    }
                }
            }
        }
    }
}
```

---

## 19. Refresh กับ refreshable (Pull-to-Refresh)

### refreshable Modifier

```swift
struct RefreshableListView: View {
    @State private var items: [String] = ["รายการ A", "รายการ B", "รายการ C"]
    @State private var isLoading = false
    
    var body: some View {
        NavigationStack {
            List(items, id: \.self) { item in
                Text(item)
            }
            .navigationTitle("ดึงเพื่อรีเฟรช")
            .refreshable {
                // จำลองการดึงข้อมูลจาก API
                await refreshData()
            }
        }
    }
    
    func refreshData() async {
        // จำลอง network delay
        try? await Task.sleep(nanoseconds: 2_000_000_000) // 2 วินาที
        
        // อัปเดตข้อมูล
        let newItems = ["รายการใหม่ 1", "รายการใหม่ 2", "รายการใหม่ 3", "รายการใหม่ 4"]
        items = newItems
    }
}
```

### refreshable กับ async/await จริง

```swift
struct NewsRefreshableView: View {
    struct NewsItem: Identifiable {
        let id = UUID()
        let title: String
        let source: String
        let time: String
    }
    
    @State private var news: [NewsItem] = []
    @State private var lastUpdate = Date()
    
    var body: some View {
        NavigationStack {
            Group {
                if news.isEmpty {
                    ContentUnavailableView(
                        "ยังไม่มีข่าว",
                        systemImage: "newspaper",
                        description: Text("ดึงลงเพื่อโหลดข่าว")
                    )
                } else {
                    List(news) { item in
                        VStack(alignment: .leading, spacing: 4) {
                            Text(item.title)
                                .font(.headline)
                            HStack {
                                Text(item.source)
                                    .font(.caption)
                                    .foregroundColor(.blue)
                                Spacer()
                                Text(item.time)
                                    .font(.caption)
                                    .foregroundColor(.secondary)
                            }
                        }
                        .padding(.vertical, 4)
                    }
                }
            }
            .navigationTitle("ข่าววันนี้")
            .toolbar {
                ToolbarItem(placement: .topBarTrailing) {
                    Text("อัปเดต: \(lastUpdate, style: .time)")
                        .font(.caption)
                        .foregroundColor(.secondary)
                }
            }
            .refreshable {
                await loadNews()
            }
            .task {
                await loadNews()
            }
        }
    }
    
    func loadNews() async {
        try? await Task.sleep(nanoseconds: 1_500_000_000)
        
        let sampleNews = [
            NewsItem(title: "Apple ประกาศ WWDC 2027", source: "TechCrunch", time: "5 นาทีที่แล้ว"),
            NewsItem(title: "Swift 6.0 ออกแล้ว", source: "Swift.org", time: "1 ชั่วโมงที่แล้ว"),
            NewsItem(title: "iOS 20 คุณสมบัติใหม่", source: "9to5Mac", time: "2 ชั่วโมงที่แล้ว"),
            NewsItem(title: "SwiftUI Performance Improvements", source: "WWDC Notes", time: "3 ชั่วโมงที่แล้ว")
        ]
        
        news = sampleNews
        lastUpdate = Date()
    }
}
```

---

## 20. Infinite Scroll (การโหลดข้อมูลแบบไม่สิ้นสุด)

```swift
struct InfiniteScrollView: View {
    struct Post: Identifiable {
        let id: Int
        let title: String
        let author: String
        let likes: Int
    }
    
    @State private var posts: [Post] = []
    @State private var page = 0
    @State private var isLoadingMore = false
    @State private var hasMore = true
    
    let pageSize = 20
    
    var body: some View {
        NavigationStack {
            List {
                ForEach(posts) { post in
                    PostRow(post: post)
                        .onAppear {
                            // โหลดเพิ่มเมื่อเห็นรายการสุดท้าย
                            if post.id == posts.last?.id {
                                Task {
                                    await loadMorePosts()
                                }
                            }
                        }
                }
                
                // Loading indicator ท้ายรายการ
                if isLoadingMore {
                    HStack {
                        Spacer()
                        ProgressView()
                            .padding()
                        Spacer()
                    }
                    .listRowSeparator(.hidden)
                }
                
                // ข้อความเมื่อโหลดครบแล้ว
                if !hasMore && !posts.isEmpty {
                    Text("แสดงครบทุกรายการแล้ว")
                        .frame(maxWidth: .infinity)
                        .font(.caption)
                        .foregroundColor(.secondary)
                        .padding()
                        .listRowSeparator(.hidden)
                }
            }
            .navigationTitle("โพสต์")
            .task {
                await loadMorePosts()
            }
            .refreshable {
                posts.removeAll()
                page = 0
                hasMore = true
                await loadMorePosts()
            }
        }
    }
    
    func loadMorePosts() async {
        guard !isLoadingMore && hasMore else { return }
        
        isLoadingMore = true
        
        // จำลอง API call
        try? await Task.sleep(nanoseconds: 1_000_000_000)
        
        let startId = page * pageSize + 1
        let endId = startId + pageSize - 1
        
        let newPosts = (startId...min(endId, 100)).map { id in
            Post(
                id: id,
                title: "โพสต์ที่ \(id): เรื่องราวที่น่าสนใจ",
                author: "ผู้เขียน \(id % 5 + 1)",
                likes: Int.random(in: 0...500)
            )
        }
        
        posts.append(contentsOf: newPosts)
        page += 1
        
        // หยุดโหลดเมื่อถึง 100 รายการ
        if posts.count >= 100 {
            hasMore = false
        }
        
        isLoadingMore = false
    }
}

struct PostRow: View {
    let post: InfiniteScrollView.Post
    
    var body: some View {
        VStack(alignment: .leading, spacing: 6) {
            Text(post.title)
                .font(.headline)
                .lineLimit(2)
            HStack {
                Text(post.author)
                    .font(.subheadline)
                    .foregroundColor(.secondary)
                Spacer()
                Label("\(post.likes)", systemImage: "heart")
                    .font(.subheadline)
                    .foregroundColor(.red)
            }
        }
        .padding(.vertical, 4)
    }
}
```

---

## 21. แบบฝึกหัด 1: แอป Contact List

```swift
// แบบฝึกหัดที่ 1: สร้าง Contact List App ที่สมบูรณ์

import SwiftUI

// MARK: - Model
struct Contact: Identifiable, Codable {
    let id: UUID
    var firstName: String
    var lastName: String
    var phone: String
    var email: String
    var group: ContactGroup
    var isFavorite: Bool
    
    var fullName: String { "\(firstName) \(lastName)" }
    var initials: String {
        let f = String(firstName.prefix(1))
        let l = String(lastName.prefix(1))
        return "\(f)\(l)"
    }
    
    init(firstName: String, lastName: String, phone: String = "",
         email: String = "", group: ContactGroup = .personal, isFavorite: Bool = false) {
        self.id = UUID()
        self.firstName = firstName
        self.lastName = lastName
        self.phone = phone
        self.email = email
        self.group = group
        self.isFavorite = isFavorite
    }
}

enum ContactGroup: String, CaseIterable, Codable {
    case personal = "ส่วนตัว"
    case work = "งาน"
    case family = "ครอบครัว"
    case other = "อื่นๆ"
    
    var icon: String {
        switch self {
        case .personal: return "person.fill"
        case .work: return "briefcase.fill"
        case .family: return "house.fill"
        case .other: return "ellipsis.circle.fill"
        }
    }
    
    var color: Color {
        switch self {
        case .personal: return .blue
        case .work: return .orange
        case .family: return .green
        case .other: return .gray
        }
    }
}

// MARK: - ViewModel
@Observable
class ContactsViewModel {
    var contacts: [Contact] = Contact.sampleContacts
    var searchText = ""
    var selectedGroup: ContactGroup? = nil
    var sortByName = true
    
    var filteredAndSortedContacts: [Contact] {
        var result = contacts
        
        // กรองตาม group
        if let group = selectedGroup {
            result = result.filter { $0.group == group }
        }
        
        // กรองตาม search text
        if !searchText.isEmpty {
            result = result.filter {
                $0.fullName.localizedCaseInsensitiveContains(searchText) ||
                $0.phone.contains(searchText) ||
                $0.email.localizedCaseInsensitiveContains(searchText)
            }
        }
        
        // เรียงลำดับ
        if sortByName {
            result.sort { $0.fullName < $1.fullName }
        } else {
            result.sort { $0.group.rawValue < $1.group.rawValue }
        }
        
        return result
    }
    
    var groupedContacts: [String: [Contact]] {
        Dictionary(grouping: filteredAndSortedContacts) {
            String($0.firstName.prefix(1)).uppercased()
        }
    }
    
    var favoriteContacts: [Contact] {
        contacts.filter { $0.isFavorite }
    }
    
    func addContact(_ contact: Contact) {
        contacts.append(contact)
    }
    
    func deleteContacts(at offsets: IndexSet, from group: String) {
        let groupContacts = groupedContacts[group] ?? []
        let idsToDelete = offsets.map { groupContacts[$0].id }
        contacts.removeAll { idsToDelete.contains($0.id) }
    }
    
    func toggleFavorite(_ contact: Contact) {
        if let index = contacts.firstIndex(where: { $0.id == contact.id }) {
            contacts[index].isFavorite.toggle()
        }
    }
    
    func deleteContact(_ contact: Contact) {
        contacts.removeAll { $0.id == contact.id }
    }
}

extension Contact {
    static let sampleContacts = [
        Contact(firstName: "สมชาย", lastName: "ใจดี", phone: "081-234-5678", email: "somchai@example.com", group: .personal, isFavorite: true),
        Contact(firstName: "สมหญิง", lastName: "รักเรียน", phone: "082-345-6789", email: "somying@example.com", group: .family, isFavorite: true),
        Contact(firstName: "อนุชา", lastName: "ทำงาน", phone: "083-456-7890", email: "anucha@work.com", group: .work),
        Contact(firstName: "มนัส", lastName: "สร้างสรรค์", phone: "084-567-8901", email: "manas@example.com", group: .personal),
        Contact(firstName: "นารี", lastName: "สุขใจ", phone: "085-678-9012", email: "naree@example.com", group: .family, isFavorite: true),
        Contact(firstName: "วิชัย", lastName: "กล้าหาญ", phone: "086-789-0123", email: "wichai@work.com", group: .work),
        Contact(firstName: "กนกวรรณ", lastName: "ดีงาม", phone: "087-890-1234", email: "kanok@example.com", group: .other),
        Contact(firstName: "ประเสริฐ", lastName: "เจริญ", phone: "088-901-2345", email: "prasert@example.com", group: .personal)
    ]
}

// MARK: - Main View
struct ContactListApp: View {
    @State private var viewModel = ContactsViewModel()
    @State private var showingAddContact = false
    @State private var showingFavorites = false
    
    var sortedKeys: [String] {
        viewModel.groupedContacts.keys.sorted()
    }
    
    var body: some View {
        NavigationStack {
            List {
                // ส่วน Favorites
                if !viewModel.favoriteContacts.isEmpty && viewModel.searchText.isEmpty {
                    Section("รายการโปรด") {
                        ScrollView(.horizontal, showsIndicators: false) {
                            HStack(spacing: 16) {
                                ForEach(viewModel.favoriteContacts) { contact in
                                    FavoriteContactChip(contact: contact)
                                }
                            }
                            .padding(.horizontal, 4)
                        }
                        .listRowInsets(EdgeInsets(top: 8, leading: 16, bottom: 8, trailing: 16))
                        .listRowSeparator(.hidden)
                    }
                }
                
                // รายการตามตัวอักษร
                ForEach(sortedKeys, id: \.self) { key in
                    Section(key) {
                        ForEach(viewModel.groupedContacts[key] ?? []) { contact in
                            NavigationLink {
                                ContactDetailView(
                                    contact: contact,
                                    viewModel: viewModel
                                )
                            } label: {
                                ContactRow(contact: contact, viewModel: viewModel)
                            }
                        }
                        .onDelete { offsets in
                            viewModel.deleteContacts(at: offsets, from: key)
                        }
                    }
                }
            }
            .navigationTitle("ผู้ติดต่อ")
            .searchable(text: $viewModel.searchText, prompt: "ค้นหาผู้ติดต่อ")
            .toolbar {
                ToolbarItem(placement: .topBarLeading) {
                    Menu {
                        Button("เรียงตามชื่อ") { viewModel.sortByName = true }
                        Button("เรียงตามกลุ่ม") { viewModel.sortByName = false }
                        Divider()
                        Menu("กรองตามกลุ่ม") {
                            Button("ทั้งหมด") { viewModel.selectedGroup = nil }
                            ForEach(ContactGroup.allCases, id: \.self) { group in
                                Button(group.rawValue) { viewModel.selectedGroup = group }
                            }
                        }
                    } label: {
                        Image(systemName: "line.3.horizontal.decrease.circle")
                    }
                }
                
                ToolbarItem(placement: .topBarTrailing) {
                    Button {
                        showingAddContact = true
                    } label: {
                        Image(systemName: "plus")
                    }
                }
            }
            .sheet(isPresented: $showingAddContact) {
                AddContactView(viewModel: viewModel)
            }
        }
    }
}

// MARK: - Contact Row
struct ContactRow: View {
    let contact: Contact
    let viewModel: ContactsViewModel
    
    var body: some View {
        HStack(spacing: 12) {
            Circle()
                .fill(contact.group.color.gradient)
                .frame(width: 44, height: 44)
                .overlay(
                    Text(contact.initials)
                        .font(.headline)
                        .foregroundColor(.white)
                )
            
            VStack(alignment: .leading, spacing: 2) {
                Text(contact.fullName)
                    .font(.body)
                Text(contact.phone.isEmpty ? contact.email : contact.phone)
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
            
            Spacer()
            
            if contact.isFavorite {
                Image(systemName: "star.fill")
                    .foregroundColor(.yellow)
                    .font(.caption)
            }
        }
        .swipeActions(edge: .trailing) {
            Button(role: .destructive) {
                viewModel.deleteContact(contact)
            } label: {
                Label("ลบ", systemImage: "trash")
            }
        }
        .swipeActions(edge: .leading) {
            Button {
                viewModel.toggleFavorite(contact)
            } label: {
                Label(
                    contact.isFavorite ? "เลิกโปรด" : "โปรด",
                    systemImage: contact.isFavorite ? "star.slash" : "star"
                )
            }
            .tint(.yellow)
        }
    }
}

// MARK: - Favorite Chip
struct FavoriteContactChip: View {
    let contact: Contact
    
    var body: some View {
        VStack(spacing: 4) {
            Circle()
                .fill(contact.group.color.gradient)
                .frame(width: 52, height: 52)
                .overlay(
                    Text(contact.initials)
                        .font(.title3)
                        .fontWeight(.semibold)
                        .foregroundColor(.white)
                )
            Text(contact.firstName)
                .font(.caption)
                .lineLimit(1)
        }
        .frame(width: 60)
    }
}

// MARK: - Contact Detail View
struct ContactDetailView: View {
    let contact: Contact
    let viewModel: ContactsViewModel
    
    var body: some View {
        List {
            Section {
                HStack {
                    Spacer()
                    VStack(spacing: 8) {
                        Circle()
                            .fill(contact.group.color.gradient)
                            .frame(width: 80, height: 80)
                            .overlay(
                                Text(contact.initials)
                                    .font(.largeTitle)
                                    .fontWeight(.semibold)
                                    .foregroundColor(.white)
                            )
                        Text(contact.fullName)
                            .font(.title2)
                            .fontWeight(.semibold)
                        Label(contact.group.rawValue, systemImage: contact.group.icon)
                            .font(.subheadline)
                            .foregroundColor(contact.group.color)
                    }
                    Spacer()
                }
                .listRowBackground(Color.clear)
            }
            
            if !contact.phone.isEmpty {
                Section("โทรศัพท์") {
                    HStack {
                        Image(systemName: "phone.fill")
                            .foregroundColor(.green)
                        Text(contact.phone)
                        Spacer()
                        Button("โทร") {
                            print("โทร \(contact.phone)")
                        }
                        .buttonStyle(.borderless)
                        .foregroundColor(.blue)
                    }
                }
            }
            
            if !contact.email.isEmpty {
                Section("อีเมล") {
                    HStack {
                        Image(systemName: "envelope.fill")
                            .foregroundColor(.blue)
                        Text(contact.email)
                        Spacer()
                        Button("ส่ง") {
                            print("ส่งอีเมล \(contact.email)")
                        }
                        .buttonStyle(.borderless)
                        .foregroundColor(.blue)
                    }
                }
            }
            
            Section {
                Button {
                    viewModel.toggleFavorite(contact)
                } label: {
                    Label(
                        contact.isFavorite ? "เลิกเพิ่มในรายการโปรด" : "เพิ่มในรายการโปรด",
                        systemImage: contact.isFavorite ? "star.slash" : "star"
                    )
                }
                
                Button(role: .destructive) {
                    viewModel.deleteContact(contact)
                } label: {
                    Label("ลบผู้ติดต่อ", systemImage: "trash")
                }
            }
        }
        .navigationTitle(contact.fullName)
        .navigationBarTitleDisplayMode(.inline)
    }
}

// MARK: - Add Contact View
struct AddContactView: View {
    let viewModel: ContactsViewModel
    @Environment(\.dismiss) var dismiss
    
    @State private var firstName = ""
    @State private var lastName = ""
    @State private var phone = ""
    @State private var email = ""
    @State private var group: ContactGroup = .personal
    @State private var isFavorite = false
    
    var isValid: Bool {
        !firstName.isEmpty && !lastName.isEmpty
    }
    
    var body: some View {
        NavigationStack {
            Form {
                Section("ชื่อ-นามสกุล") {
                    TextField("ชื่อ *", text: $firstName)
                    TextField("นามสกุล *", text: $lastName)
                }
                
                Section("ข้อมูลติดต่อ") {
                    TextField("เบอร์โทร", text: $phone)
                        .keyboardType(.phonePad)
                    TextField("อีเมล", text: $email)
                        .keyboardType(.emailAddress)
                        .autocapitalization(.none)
                }
                
                Section("จัดกลุ่ม") {
                    Picker("กลุ่ม", selection: $group) {
                        ForEach(ContactGroup.allCases, id: \.self) { g in
                            Label(g.rawValue, systemImage: g.icon).tag(g)
                        }
                    }
                    
                    Toggle("เพิ่มในรายการโปรด", isOn: $isFavorite)
                }
            }
            .navigationTitle("เพิ่มผู้ติดต่อ")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .topBarLeading) {
                    Button("ยกเลิก") { dismiss() }
                }
                ToolbarItem(placement: .topBarTrailing) {
                    Button("บันทึก") {
                        let newContact = Contact(
                            firstName: firstName,
                            lastName: lastName,
                            phone: phone,
                            email: email,
                            group: group,
                            isFavorite: isFavorite
                        )
                        viewModel.addContact(newContact)
                        dismiss()
                    }
                    .disabled(!isValid)
                    .fontWeight(.semibold)
                }
            }
        }
    }
}
```

---

## 22. แบบฝึกหัด 2: Settings Screen

```swift
// แบบฝึกหัดที่ 2: สร้าง Settings Screen ที่สมบูรณ์

import SwiftUI

// MARK: - Settings Model
@Observable
class AppSettings {
    // ธีม
    var colorScheme: ColorSchemeOption = .system
    var accentColor: AccentColorOption = .blue
    var fontSize: Double = 16
    
    // การแจ้งเตือน
    var pushNotifications = true
    var emailNotifications = true
    var soundEnabled = true
    var vibrationEnabled = true
    var badgeCount = true
    
    // ความเป็นส่วนตัว
    var shareUsageData = false
    var locationAccess = true
    var faceIDEnabled = true
    var autoLock: AutoLockOption = .fiveMinutes
    
    // บัญชี
    var username = "user123"
    var email = "user@example.com"
    var isPremium = false
    
    // ทั่วไป
    var language: LanguageOption = .thai
    var region: String = "Thailand"
    var dateFormat: DateFormatOption = .ddmmyyyy
    var autoUpdate = true
    var backgroundRefresh = true
    var storageUsed: Double = 2.4 // GB
    var storageTotal: Double = 16.0 // GB
}

enum ColorSchemeOption: String, CaseIterable {
    case light = "สว่าง"
    case dark = "มืด"
    case system = "ตามระบบ"
    
    var icon: String {
        switch self {
        case .light: return "sun.max"
        case .dark: return "moon"
        case .system: return "circle.lefthalf.filled"
        }
    }
}

enum AccentColorOption: String, CaseIterable {
    case blue = "น้ำเงิน"
    case red = "แดง"
    case green = "เขียว"
    case orange = "ส้ม"
    case purple = "ม่วง"
    case pink = "ชมพู"
    
    var color: Color {
        switch self {
        case .blue: return .blue
        case .red: return .red
        case .green: return .green
        case .orange: return .orange
        case .purple: return .purple
        case .pink: return .pink
        }
    }
}

enum AutoLockOption: String, CaseIterable {
    case oneMinute = "1 นาที"
    case threeMinutes = "3 นาที"
    case fiveMinutes = "5 นาที"
    case tenMinutes = "10 นาที"
    case never = "ไม่ล็อก"
}

enum LanguageOption: String, CaseIterable {
    case thai = "ภาษาไทย"
    case english = "English"
    case japanese = "日本語"
}

enum DateFormatOption: String, CaseIterable {
    case ddmmyyyy = "DD/MM/YYYY"
    case mmddyyyy = "MM/DD/YYYY"
    case yyyymmdd = "YYYY/MM/DD"
}

// MARK: - Settings Screen
struct SettingsScreen: View {
    @State private var settings = AppSettings()
    @State private var showingSignOut = false
    @State private var showingDeleteAccount = false
    @State private var showingClearCache = false
    
    var storagePercentage: Double {
        settings.storageUsed / settings.storageTotal
    }
    
    var body: some View {
        NavigationStack {
            Form {
                // MARK: โปรไฟล์
                Section {
                    ProfileHeaderRow(settings: settings)
                        .listRowBackground(Color.clear)
                        .listRowInsets(EdgeInsets())
                }
                
                // MARK: บัญชี
                Section("บัญชี") {
                    NavigationLink {
                        AccountSettingsView(settings: settings)
                    } label: {
                        Label("ข้อมูลบัญชี", systemImage: "person.circle")
                    }
                    
                    if !settings.isPremium {
                        Button {
                            settings.isPremium = true
                        } label: {
                            HStack {
                                Label("อัปเกรดเป็น Premium", systemImage: "star.fill")
                                    .foregroundColor(.orange)
                                Spacer()
                                Text("ฟรีทดลอง 7 วัน")
                                    .font(.caption)
                                    .foregroundColor(.secondary)
                            }
                        }
                    } else {
                        HStack {
                            Label("Premium Member", systemImage: "crown.fill")
                                .foregroundColor(.orange)
                            Spacer()
                            Image(systemName: "checkmark.circle.fill")
                                .foregroundColor(.green)
                        }
                    }
                }
                
                // MARK: ธีม
                Section("การแสดงผล") {
                    NavigationLink {
                        AppearanceSettingsView(settings: settings)
                    } label: {
                        HStack {
                            Label("ธีม", systemImage: "paintbrush")
                            Spacer()
                            Text(settings.colorScheme.rawValue)
                                .foregroundColor(.secondary)
                        }
                    }
                    
                    HStack {
                        Label("สีหลัก", systemImage: "circle.fill")
                        Spacer()
                        ForEach(AccentColorOption.allCases, id: \.self) { option in
                            Button {
                                settings.accentColor = option
                            } label: {
                                Circle()
                                    .fill(option.color)
                                    .frame(width: 20, height: 20)
                                    .overlay(
                                        Circle()
                                            .stroke(Color.primary, lineWidth: settings.accentColor == option ? 2 : 0)
                                            .padding(2)
                                    )
                            }
                            .buttonStyle(.plain)
                        }
                    }
                    
                    VStack(alignment: .leading, spacing: 4) {
                        HStack {
                            Label("ขนาดตัวอักษร", systemImage: "textformat.size")
                            Spacer()
                            Text("\(Int(settings.fontSize))pt")
                                .foregroundColor(.secondary)
                        }
                        Slider(value: $settings.fontSize, in: 12...24, step: 1)
                    }
                }
                
                // MARK: การแจ้งเตือน
                Section("การแจ้งเตือน") {
                    Toggle(isOn: $settings.pushNotifications) {
                        Label("Push Notifications", systemImage: "bell.badge")
                    }
                    
                    if settings.pushNotifications {
                        Group {
                            Toggle(isOn: $settings.soundEnabled) {
                                Label("เสียง", systemImage: "speaker.wave.2")
                            }
                            Toggle(isOn: $settings.vibrationEnabled) {
                                Label("การสั่น", systemImage: "iphone.radiowaves.left.and.right")
                            }
                            Toggle(isOn: $settings.badgeCount) {
                                Label("Badge Count", systemImage: "app.badge")
                            }
                        }
                        .padding(.leading, 8)
                    }
                    
                    Toggle(isOn: $settings.emailNotifications) {
                        Label("อีเมลแจ้งเตือน", systemImage: "envelope")
                    }
                }
                
                // MARK: ความเป็นส่วนตัว
                Section("ความเป็นส่วนตัวและความปลอดภัย") {
                    Toggle(isOn: $settings.faceIDEnabled) {
                        Label("Face ID", systemImage: "faceid")
                    }
                    
                    Picker(selection: $settings.autoLock) {
                        ForEach(AutoLockOption.allCases, id: \.self) { option in
                            Text(option.rawValue).tag(option)
                        }
                    } label: {
                        Label("ล็อกอัตโนมัติ", systemImage: "lock")
                    }
                    
                    Toggle(isOn: $settings.locationAccess) {
                        Label("ตำแหน่ง", systemImage: "location")
                    }
                    
                    Toggle(isOn: $settings.shareUsageData) {
                        Label("แชร์ข้อมูลการใช้งาน", systemImage: "chart.bar")
                    }
                }
                
                // MARK: ทั่วไป
                Section("ทั่วไป") {
                    Picker(selection: $settings.language) {
                        ForEach(LanguageOption.allCases, id: \.self) { lang in
                            Text(lang.rawValue).tag(lang)
                        }
                    } label: {
                        Label("ภาษา", systemImage: "globe")
                    }
                    
                    Picker(selection: $settings.dateFormat) {
                        ForEach(DateFormatOption.allCases, id: \.self) { fmt in
                            Text(fmt.rawValue).tag(fmt)
                        }
                    } label: {
                        Label("รูปแบบวันที่", systemImage: "calendar")
                    }
                    
                    Toggle(isOn: $settings.autoUpdate) {
                        Label("อัปเดตอัตโนมัติ", systemImage: "arrow.clockwise")
                    }
                    
                    Toggle(isOn: $settings.backgroundRefresh) {
                        Label("รีเฟรชเบื้องหลัง", systemImage: "arrow.2.circlepath")
                    }
                }
                
                // MARK: พื้นที่เก็บข้อมูล
                Section {
                    VStack(alignment: .leading, spacing: 8) {
                        HStack {
                            Label("พื้นที่เก็บข้อมูล", systemImage: "internaldrive")
                            Spacer()
                            Text(String(format: "%.1f/%.0f GB", settings.storageUsed, settings.storageTotal))
                                .font(.subheadline)
                                .foregroundColor(.secondary)
                        }
                        
                        ProgressView(value: storagePercentage)
                            .tint(storagePercentage > 0.8 ? .red : .blue)
                        
                        Text(String(format: "ใช้ไป %.0f%%", storagePercentage * 100))
                            .font(.caption)
                            .foregroundColor(.secondary)
                    }
                    
                    Button {
                        showingClearCache = true
                    } label: {
                        Label("ล้างแคช", systemImage: "trash")
                            .foregroundColor(.orange)
                    }
                } header: {
                    Text("พื้นที่เก็บข้อมูล")
                }
                
                // MARK: เกี่ยวกับ
                Section("เกี่ยวกับ") {
                    HStack {
                        Text("เวอร์ชัน")
                        Spacer()
                        Text("1.0.0 (100)")
                            .foregroundColor(.secondary)
                    }
                    
                    Link(destination: URL(string: "https://example.com/privacy")!) {
                        Label("นโยบายความเป็นส่วนตัว", systemImage: "doc.text")
                    }
                    
                    Link(destination: URL(string: "https://example.com/terms")!) {
                        Label("ข้อกำหนดการใช้งาน", systemImage: "doc.plaintext")
                    }
                    
                    Button {
                        print("ติดต่อฝ่ายสนับสนุน")
                    } label: {
                        Label("ติดต่อฝ่ายสนับสนุน", systemImage: "message")
                    }
                }
                
                // MARK: ออกจากระบบ
                Section {
                    Button {
                        showingSignOut = true
                    } label: {
                        Text("ออกจากระบบ")
                            .frame(maxWidth: .infinity)
                            .foregroundColor(.red)
                    }
                    
                    Button {
                        showingDeleteAccount = true
                    } label: {
                        Text("ลบบัญชี")
                            .frame(maxWidth: .infinity)
                            .foregroundColor(.red)
                    }
                }
            }
            .navigationTitle("การตั้งค่า")
            .alert("ออกจากระบบ", isPresented: $showingSignOut) {
                Button("ออกจากระบบ", role: .destructive) {
                    print("ออกจากระบบ")
                }
                Button("ยกเลิก", role: .cancel) {}
            } message: {
                Text("คุณต้องการออกจากระบบใช่หรือไม่?")
            }
            .alert("ลบบัญชี", isPresented: $showingDeleteAccount) {
                Button("ลบบัญชี", role: .destructive) {
                    print("ลบบัญชี")
                }
                Button("ยกเลิก", role: .cancel) {}
            } message: {
                Text("การลบบัญชีจะไม่สามารถกู้คืนได้ คุณแน่ใจหรือไม่?")
            }
            .alert("ล้างแคช", isPresented: $showingClearCache) {
                Button("ล้างแคช") {
                    settings.storageUsed = max(0, settings.storageUsed - 0.5)
                }
                Button("ยกเลิก", role: .cancel) {}
            } message: {
                Text("ล้างข้อมูลแคชเพื่อเพิ่มพื้นที่ว่าง")
            }
        }
    }
}

// MARK: - Profile Header Row
struct ProfileHeaderRow: View {
    let settings: AppSettings
    
    var body: some View {
        HStack(spacing: 16) {
            Circle()
                .fill(Color.blue.gradient)
                .frame(width: 60, height: 60)
                .overlay(
                    Text(String(settings.username.prefix(2)).uppercased())
                        .font(.title3)
                        .fontWeight(.semibold)
                        .foregroundColor(.white)
                )
            
            VStack(alignment: .leading, spacing: 4) {
                Text(settings.username)
                    .font(.headline)
                Text(settings.email)
                    .font(.subheadline)
                    .foregroundColor(.secondary)
                if settings.isPremium {
                    Label("Premium Member", systemImage: "crown.fill")
                        .font(.caption)
                        .foregroundColor(.orange)
                }
            }
            
            Spacer()
            
            Image(systemName: "chevron.right")
                .font(.caption)
                .foregroundColor(.secondary)
        }
        .padding()
        .background(Color(.systemBackground))
    }
}

// MARK: - Appearance Settings
struct AppearanceSettingsView: View {
    @Bindable var settings: AppSettings
    
    var body: some View {
        Form {
            Section("โหมดสี") {
                ForEach(ColorSchemeOption.allCases, id: \.self) { option in
                    HStack {
                        Label(option.rawValue, systemImage: option.icon)
                        Spacer()
                        if settings.colorScheme == option {
                            Image(systemName: "checkmark")
                                .foregroundColor(.blue)
                        }
                    }
                    .contentShape(Rectangle())
                    .onTapGesture {
                        settings.colorScheme = option
                    }
                }
            }
        }
        .navigationTitle("การแสดงผล")
    }
}

// MARK: - Account Settings
struct AccountSettingsView: View {
    @Bindable var settings: AppSettings
    
    var body: some View {
        Form {
            Section("ข้อมูลบัญชี") {
                HStack {
                    Text("ชื่อผู้ใช้")
                    Spacer()
                    Text(settings.username)
                        .foregroundColor(.secondary)
                }
                HStack {
                    Text("อีเมล")
                    Spacer()
                    Text(settings.email)
                        .foregroundColor(.secondary)
                }
            }
            
            Section {
                Button("เปลี่ยนรหัสผ่าน") {
                    print("เปลี่ยนรหัสผ่าน")
                }
            }
        }
        .navigationTitle("ข้อมูลบัญชี")
    }
}
```

---

## สรุป

ในส่วนนี้เราได้เรียนรู้เกี่ยวกับ **List** และ **Form** ใน SwiftUI อย่างครอบคลุม:

### สิ่งที่ได้เรียนรู้

1. **List Fundamentals** - พื้นฐานการสร้าง List และ Static Lists
2. **Dynamic Lists** - การใช้ ForEach กับข้อมูลแบบ dynamic
3. **Identifiable Protocol** - การทำให้ model เป็น unique identifier
4. **id Parameter** - การระบุ identifier ใน ForEach
5. **List Styles** - plain, grouped, inset, insetGrouped, sidebar
6. **Section Headers/Footers** - การจัดกลุ่มด้วย Section
7. **List Selection** - การเลือก single และ multiple items
8. **Swipe Actions** - การเพิ่ม action ด้วย swipe
9. **Edit Mode** - การเปิด edit mode สำหรับ delete/move
10. **Move และ Delete** - `onDelete` และ `onMove` modifiers
11. **searchable** - การเพิ่มช่องค้นหา
12. **Row Customization** - การปรับแต่ง listRowBackground, Separator, Insets
13. **Form** - การสร้างแบบฟอร์มสำหรับ input
14. **Form Inputs** - TextField, Toggle, Picker, DatePicker, Slider, Stepper
15. **Picker Styles** - menu, segmented, inline, wheel
16. **Performance** - ScrollView + LazyVStack vs List
17. **refreshable** - Pull-to-Refresh
18. **Infinite Scroll** - การโหลดข้อมูลทีละหน้า

### Best Practices

- ใช้ `Identifiable` protocol แทนการระบุ `id: \.self` เมื่อเป็นไปได้
- ใช้ `List` สำหรับรายการมาตรฐาน, ใช้ `ScrollView + LazyVStack` สำหรับ custom layouts
- แบ่ง Row เป็น sub-View เพื่อให้โค้ดอ่านง่ายและ reusable
- ใช้ `@Observable` หรือ `ObservableObject` สำหรับ ViewModel
- เพิ่ม loading state สำหรับ async operations

---

*ส่วนที่ 25 จบ - ต่อไปส่วนที่ 26: SwiftUI Animations*
