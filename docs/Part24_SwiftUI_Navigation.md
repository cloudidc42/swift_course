# ตอนที่ 24: SwiftUI Navigation

## บทนำ

Navigation (การนำทาง) เป็นส่วนสำคัญของแอป iOS ที่ช่วยให้ผู้ใช้เดินทางระหว่างหน้าต่างๆ ได้ SwiftUI มีระบบ Navigation ที่พัฒนาขึ้นเรื่อยๆ โดยใน iOS 16+ มี `NavigationStack` ที่ทรงพลังกว่า `NavigationView` เดิมมาก ในบทนี้เราจะเรียนรู้ระบบ Navigation ทั้งหมดใน SwiftUI อย่างละเอียด

---

## 1. NavigationStack (iOS 16+)

`NavigationStack` เป็น Container View ที่จัดการ stack ของ View สำหรับการนำทางแบบ push/pop ซึ่งเป็น pattern หลักบน iPhone

### โครงสร้างพื้นฐาน

```swift
import SwiftUI

struct BasicNavigationApp: View {
    var body: some View {
        NavigationStack {
            ContentView()
                .navigationTitle("หน้าหลัก")
        }
    }
}

struct ContentView: View {
    var body: some View {
        List {
            NavigationLink("ไปหน้าที่ 1") {
                SecondView()
            }
            NavigationLink("ไปหน้าที่ 2") {
                ThirdView()
            }
        }
    }
}

struct SecondView: View {
    var body: some View {
        Text("นี่คือหน้าที่ 2")
            .navigationTitle("หน้า 2")
    }
}

struct ThirdView: View {
    var body: some View {
        Text("นี่คือหน้าที่ 3")
            .navigationTitle("หน้า 3")
    }
}
```

### ทำไมต้องใช้ NavigationStack แทน NavigationView?

```swift
// ❌ NavigationView (deprecated ใน iOS 16)
struct OldNavigation: View {
    var body: some View {
        NavigationView {
            Text("เนื้อหา")
                .navigationTitle("หัวข้อ")
        }
    }
}

// ✅ NavigationStack (iOS 16+) - ดีกว่าเพราะ:
// 1. รองรับ programmatic navigation
// 2. Type-safe navigation paths
// 3. Deep linking ง่ายกว่า
// 4. ประสิทธิภาพดีกว่า
struct NewNavigation: View {
    var body: some View {
        NavigationStack {
            Text("เนื้อหา")
                .navigationTitle("หัวข้อ")
        }
    }
}
```

---

## 2. NavigationLink

`NavigationLink` สร้าง Link ที่เมื่อกดแล้วจะ push View ใหม่เข้าไปใน NavigationStack

### รูปแบบต่างๆ ของ NavigationLink

```swift
struct NavigationLinkVariants: View {
    var body: some View {
        NavigationStack {
            List {
                // แบบที่ 1: Label เป็น Text
                NavigationLink("ข้อความธรรมดา") {
                    DetailView(title: "หน้า 1")
                }
                
                // แบบที่ 2: Custom Label
                NavigationLink {
                    DetailView(title: "หน้า 2")
                } label: {
                    HStack {
                        Image(systemName: "star.fill")
                            .foregroundColor(.yellow)
                        VStack(alignment: .leading) {
                            Text("หัวข้อ")
                                .font(.headline)
                            Text("รายละเอียด")
                                .font(.caption)
                                .foregroundColor(.secondary)
                        }
                    }
                }
                
                // แบบที่ 3: Navigation ด้วย value (iOS 16+)
                NavigationLink("ด้วย value", value: "หน้า 3")
            }
            .navigationDestination(for: String.self) { title in
                DetailView(title: title)
            }
            .navigationTitle("ตัวอย่าง")
        }
    }
}

struct DetailView: View {
    let title: String
    
    var body: some View {
        Text("หน้า: \(title)")
            .font(.largeTitle)
            .navigationTitle(title)
    }
}
```

### NavigationLink กับ List

```swift
struct Contact: Identifiable {
    let id = UUID()
    var name: String
    var phone: String
    var email: String
    var isFavorite: Bool
}

struct ContactListView: View {
    let contacts = [
        Contact(name: "สมชาย ใจดี", phone: "081-234-5678", email: "somchai@example.com", isFavorite: true),
        Contact(name: "สมหญิง รักเรียน", phone: "082-345-6789", email: "somying@example.com", isFavorite: false),
        Contact(name: "วิชัย สุขใจ", phone: "083-456-7890", email: "vichai@example.com", isFavorite: true),
    ]
    
    var body: some View {
        NavigationStack {
            List(contacts) { contact in
                NavigationLink {
                    ContactDetailView(contact: contact)
                } label: {
                    ContactRow(contact: contact)
                }
            }
            .navigationTitle("รายชื่อ")
        }
    }
}

struct ContactRow: View {
    let contact: Contact
    
    var body: some View {
        HStack {
            Circle()
                .fill(Color.blue.opacity(0.2))
                .frame(width: 44, height: 44)
                .overlay(
                    Text(String(contact.name.prefix(1)))
                        .font(.title2)
                        .fontWeight(.bold)
                        .foregroundColor(.blue)
                )
            
            VStack(alignment: .leading, spacing: 2) {
                HStack {
                    Text(contact.name)
                        .font(.headline)
                    if contact.isFavorite {
                        Image(systemName: "star.fill")
                            .foregroundColor(.yellow)
                            .font(.caption)
                    }
                }
                Text(contact.phone)
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
        }
    }
}

struct ContactDetailView: View {
    let contact: Contact
    
    var body: some View {
        Form {
            Section {
                HStack {
                    Spacer()
                    VStack {
                        Circle()
                            .fill(Color.blue.opacity(0.2))
                            .frame(width: 80, height: 80)
                            .overlay(
                                Text(String(contact.name.prefix(1)))
                                    .font(.largeTitle)
                                    .fontWeight(.bold)
                                    .foregroundColor(.blue)
                            )
                        Text(contact.name)
                            .font(.title2)
                            .fontWeight(.bold)
                    }
                    Spacer()
                }
                .listRowBackground(Color.clear)
            }
            
            Section("ข้อมูลการติดต่อ") {
                LabeledContent("โทรศัพท์", value: contact.phone)
                LabeledContent("อีเมล", value: contact.email)
            }
            
            Section {
                Button {
                    // โทร
                } label: {
                    Label("โทร", systemImage: "phone.fill")
                }
                
                Button {
                    // ส่งข้อความ
                } label: {
                    Label("ส่งข้อความ", systemImage: "message.fill")
                }
                
                Button {
                    // ส่งอีเมล
                } label: {
                    Label("อีเมล", systemImage: "envelope.fill")
                }
            }
        }
        .navigationTitle(contact.name)
        .navigationBarTitleDisplayMode(.inline)
    }
}
```

---

## 3. navigationDestination

`navigationDestination(for:destination:)` ใช้ร่วมกับ `NavigationLink(value:)` เพื่อแยก "ข้อมูล" ออกจาก "View ปลายทาง" ทำให้โค้ดสะอาดกว่า

### ข้อดีของ navigationDestination

1. แยก Navigation Logic ออกจาก List items
2. รองรับ Type-safe navigation
3. รองรับ programmatic navigation
4. สามารถ define หลาย destination types ได้

```swift
// ประกาศ Model
struct Product: Identifiable, Hashable {
    let id = UUID()
    var name: String
    var price: Double
    var category: ProductCategory
    var description: String
    var rating: Double
    
    enum ProductCategory: String, CaseIterable, Hashable {
        case electronics = "อิเล็กทรอนิกส์"
        case clothing = "เสื้อผ้า"
        case food = "อาหาร"
    }
}

struct Category: Identifiable, Hashable {
    let id = UUID()
    var name: String
    var icon: String
}

// View หลัก
struct ProductStoreView: View {
    let products: [Product] = [
        Product(name: "iPhone 15", price: 35900, category: .electronics, description: "สมาร์ตโฟนรุ่นใหม่ล่าสุด", rating: 4.8),
        Product(name: "เสื้อยืด", price: 299, category: .clothing, description: "เสื้อยืดสบาย", rating: 4.2),
        Product(name: "ข้าวผัด", price: 80, category: .food, description: "ข้าวผัดอร่อยๆ", rating: 4.5),
    ]
    
    let categories: [Category] = [
        Category(name: "อิเล็กทรอนิกส์", icon: "iphone"),
        Category(name: "เสื้อผ้า", icon: "tshirt"),
        Category(name: "อาหาร", icon: "fork.knife"),
    ]
    
    var body: some View {
        NavigationStack {
            List {
                Section("หมวดหมู่") {
                    ForEach(categories) { category in
                        // ส่ง value เป็น Category
                        NavigationLink(value: category) {
                            Label(category.name, systemImage: category.icon)
                        }
                    }
                }
                
                Section("สินค้าทั้งหมด") {
                    ForEach(products) { product in
                        // ส่ง value เป็น Product
                        NavigationLink(value: product) {
                            ProductRow(product: product)
                        }
                    }
                }
            }
            // กำหนด destination สำหรับ Product
            .navigationDestination(for: Product.self) { product in
                ProductDetailPage(product: product)
            }
            // กำหนด destination สำหรับ Category
            .navigationDestination(for: Category.self) { category in
                CategoryPage(category: category, products: products)
            }
            .navigationTitle("ร้านค้า")
        }
    }
}

struct ProductRow: View {
    let product: Product
    
    var body: some View {
        HStack {
            VStack(alignment: .leading) {
                Text(product.name)
                    .font(.headline)
                Text(product.category.rawValue)
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
            Spacer()
            Text(product.price, format: .currency(code: "THB"))
                .font(.subheadline)
                .fontWeight(.semibold)
        }
    }
}

struct ProductDetailPage: View {
    let product: Product
    
    var body: some View {
        ScrollView {
            VStack(alignment: .leading, spacing: 20) {
                RoundedRectangle(cornerRadius: 12)
                    .fill(Color.blue.opacity(0.1))
                    .frame(height: 200)
                    .overlay(
                        Image(systemName: "photo")
                            .font(.largeTitle)
                            .foregroundColor(.blue)
                    )
                
                VStack(alignment: .leading, spacing: 8) {
                    Text(product.name)
                        .font(.title)
                        .fontWeight(.bold)
                    
                    Text(product.price, format: .currency(code: "THB"))
                        .font(.title2)
                        .foregroundColor(.blue)
                    
                    HStack {
                        ForEach(1...5, id: \.self) { star in
                            Image(systemName: star <= Int(product.rating) ? "star.fill" : "star")
                                .foregroundColor(.yellow)
                        }
                        Text(String(format: "%.1f", product.rating))
                            .foregroundColor(.secondary)
                    }
                    
                    Text(product.description)
                        .foregroundColor(.secondary)
                }
                .padding(.horizontal)
                
                Button("เพิ่มในตะกร้า") {
                    // เพิ่มสินค้า
                }
                .buttonStyle(.borderedProminent)
                .frame(maxWidth: .infinity)
                .padding()
            }
        }
        .navigationTitle(product.name)
        .navigationBarTitleDisplayMode(.inline)
    }
}

struct CategoryPage: View {
    let category: Category
    let products: [Product]
    
    var categoryProducts: [Product] {
        products.filter { $0.category.rawValue == category.name }
    }
    
    var body: some View {
        List(categoryProducts) { product in
            NavigationLink(value: product) {
                ProductRow(product: product)
            }
        }
        .navigationDestination(for: Product.self) { product in
            ProductDetailPage(product: product)
        }
        .navigationTitle(category.name)
    }
}
```

---

## 4. Programmatic Navigation และ NavigationPath

`NavigationPath` ช่วยให้เราควบคุม Navigation Stack ด้วยโค้ดได้อย่างสมบูรณ์

### NavigationPath พื้นฐาน

```swift
struct ProgrammaticNavigationView: View {
    @State private var path = NavigationPath()
    
    var body: some View {
        NavigationStack(path: $path) {
            VStack(spacing: 20) {
                Text("หน้าหลัก")
                    .font(.title)
                
                Button("ไปหน้าที่ 2") {
                    path.append("หน้า 2")
                }
                
                Button("ไปหน้าที่ 2 แล้ว 3") {
                    path.append("หน้า 2")
                    path.append("หน้า 3")
                }
                
                Button("กลับหน้าหลักทันที") {
                    path = NavigationPath()  // ล้าง stack
                }
            }
            .navigationDestination(for: String.self) { page in
                VStack {
                    Text(page)
                        .font(.title)
                    
                    Button("ไปหน้าถัดไป") {
                        path.append("หน้า \(path.count + 2)")
                    }
                    
                    Button("กลับหน้าหลัก") {
                        path.removeLast(path.count)
                    }
                }
                .navigationTitle(page)
            }
            .navigationTitle("หน้าหลัก")
        }
    }
}
```

### Type-safe NavigationPath

```swift
// Navigation Destination Types
struct UserProfile: Hashable {
    let id: Int
    let name: String
}

struct PostDetail: Hashable {
    let id: Int
    let title: String
    let content: String
}

struct CommentThread: Hashable {
    let postId: Int
    let commentCount: Int
}

// Complex Navigation Flow
struct SocialApp: View {
    @State private var path = NavigationPath()
    
    var body: some View {
        NavigationStack(path: $path) {
            FeedView(onNavigate: { destination in
                switch destination {
                case .profile(let user):
                    path.append(user)
                case .post(let post):
                    path.append(post)
                case .comments(let thread):
                    path.append(thread)
                }
            })
            .navigationDestination(for: UserProfile.self) { user in
                UserProfileView(user: user, path: $path)
            }
            .navigationDestination(for: PostDetail.self) { post in
                PostDetailView(post: post, path: $path)
            }
            .navigationDestination(for: CommentThread.self) { thread in
                CommentThreadView(thread: thread)
            }
            .navigationTitle("Feed")
        }
    }
}

enum NavigationDestination {
    case profile(UserProfile)
    case post(PostDetail)
    case comments(CommentThread)
}

struct FeedView: View {
    let onNavigate: (NavigationDestination) -> Void
    
    let samplePosts = [
        PostDetail(id: 1, title: "โพสต์แรก", content: "เนื้อหาของโพสต์แรก"),
        PostDetail(id: 2, title: "โพสต์ที่สอง", content: "เนื้อหาของโพสต์ที่สอง"),
    ]
    
    var body: some View {
        List(samplePosts, id: \.id) { post in
            VStack(alignment: .leading) {
                Button(post.title) {
                    onNavigate(.post(post))
                }
                .font(.headline)
                
                Button("ดูโปรไฟล์") {
                    onNavigate(.profile(UserProfile(id: 1, name: "ผู้ใช้")))
                }
                .font(.caption)
                .foregroundColor(.secondary)
            }
        }
    }
}

struct UserProfileView: View {
    let user: UserProfile
    @Binding var path: NavigationPath
    
    var body: some View {
        VStack {
            Text("โปรไฟล์: \(user.name)")
                .font(.title)
            
            Button("กลับหน้าหลัก") {
                path.removeLast(path.count)
            }
        }
        .navigationTitle(user.name)
    }
}

struct PostDetailView: View {
    let post: PostDetail
    @Binding var path: NavigationPath
    
    var body: some View {
        VStack(alignment: .leading, spacing: 20) {
            Text(post.title)
                .font(.title)
            Text(post.content)
            
            Button("ดูความคิดเห็น") {
                path.append(CommentThread(postId: post.id, commentCount: 5))
            }
            .buttonStyle(.bordered)
        }
        .padding()
        .navigationTitle(post.title)
    }
}

struct CommentThreadView: View {
    let thread: CommentThread
    
    var body: some View {
        List {
            ForEach(1...thread.commentCount, id: \.self) { i in
                Text("ความคิดเห็นที่ \(i)")
            }
        }
        .navigationTitle("ความคิดเห็น (\(thread.commentCount))")
    }
}
```

---

## 5. NavigationSplitView

`NavigationSplitView` ใช้สำหรับ iPad และ Mac ที่มีพื้นที่จอกว้าง แสดง sidebar และ content พร้อมกัน

### NavigationSplitView สองคอลัมน์

```swift
struct TwoColumnApp: View {
    @State private var selectedCategory: String? = nil
    
    let categories = ["ทั้งหมด", "โปรดปราน", "ล่าสุด", "เก็บถาวร"]
    
    var body: some View {
        NavigationSplitView {
            // Sidebar (คอลัมน์ซ้าย)
            List(categories, id: \.self, selection: $selectedCategory) { category in
                Label(category, systemImage: iconName(for: category))
            }
            .navigationTitle("เมนู")
        } detail: {
            // Detail (คอลัมน์ขวา)
            if let category = selectedCategory {
                CategoryDetailView(category: category)
            } else {
                Text("เลือกหมวดหมู่")
                    .foregroundColor(.secondary)
            }
        }
    }
    
    func iconName(for category: String) -> String {
        switch category {
        case "ทั้งหมด": return "list.bullet"
        case "โปรดปราน": return "heart.fill"
        case "ล่าสุด": return "clock"
        case "เก็บถาวร": return "archivebox"
        default: return "folder"
        }
    }
}

struct CategoryDetailView: View {
    let category: String
    
    var body: some View {
        List {
            ForEach(1...5, id: \.self) { i in
                Text("รายการ \(i) ใน \(category)")
            }
        }
        .navigationTitle(category)
    }
}
```

### NavigationSplitView สามคอลัมน์

```swift
struct ThreeColumnApp: View {
    @State private var selectedMailbox: String? = "กล่องขาเข้า"
    @State private var selectedEmail: Email? = nil
    
    let mailboxes = ["กล่องขาเข้า", "ส่งแล้ว", "แบบร่าง", "ถังขยะ"]
    
    let emails = [
        Email(id: 1, subject: "ข่าวสารประจำสัปดาห์", from: "news@example.com", body: "เนื้อหาข่าว..."),
        Email(id: 2, subject: "ยืนยันการสั่งซื้อ", from: "shop@example.com", body: "ขอบคุณที่สั่งซื้อ..."),
        Email(id: 3, subject: "นัดประชุม", from: "boss@company.com", body: "ขอนัดประชุมวันพรุ่งนี้..."),
    ]
    
    var body: some View {
        NavigationSplitView {
            // Sidebar
            List(mailboxes, id: \.self, selection: $selectedMailbox) { mailbox in
                Label(mailbox, systemImage: mailboxIcon(mailbox))
            }
            .navigationTitle("เมล")
        } content: {
            // Content (รายการเมล)
            if let mailbox = selectedMailbox {
                List(emails, selection: $selectedEmail) { email in
                    EmailRow(email: email)
                        .tag(email)
                }
                .navigationTitle(mailbox)
            } else {
                Text("เลือกกล่องจดหมาย")
            }
        } detail: {
            // Detail (เนื้อหาเมล)
            if let email = selectedEmail {
                EmailDetailView(email: email)
            } else {
                Text("เลือกอีเมล")
                    .foregroundColor(.secondary)
            }
        }
    }
    
    func mailboxIcon(_ mailbox: String) -> String {
        switch mailbox {
        case "กล่องขาเข้า": return "tray"
        case "ส่งแล้ว": return "paperplane"
        case "แบบร่าง": return "doc"
        case "ถังขยะ": return "trash"
        default: return "folder"
        }
    }
}

struct Email: Identifiable, Hashable {
    let id: Int
    var subject: String
    var from: String
    var body: String
}

struct EmailRow: View {
    let email: Email
    
    var body: some View {
        VStack(alignment: .leading, spacing: 4) {
            Text(email.subject)
                .font(.headline)
                .lineLimit(1)
            Text(email.from)
                .font(.caption)
                .foregroundColor(.secondary)
        }
        .padding(.vertical, 4)
    }
}

struct EmailDetailView: View {
    let email: Email
    
    var body: some View {
        ScrollView {
            VStack(alignment: .leading, spacing: 20) {
                Text(email.subject)
                    .font(.title)
                    .fontWeight(.bold)
                
                Text("จาก: \(email.from)")
                    .foregroundColor(.secondary)
                
                Divider()
                
                Text(email.body)
            }
            .padding()
        }
        .navigationTitle(email.subject)
        .navigationBarTitleDisplayMode(.inline)
    }
}
```

---

## 6. TabView

`TabView` แสดง Tab Bar ที่ด้านล่างหน้าจอ ช่วยให้ผู้ใช้สลับระหว่าง section หลักๆ ของแอปได้

### TabView พื้นฐาน

```swift
struct MainTabView: View {
    @State private var selectedTab = 0
    
    var body: some View {
        TabView(selection: $selectedTab) {
            HomeView()
                .tabItem {
                    Label("หน้าหลัก", systemImage: "house.fill")
                }
                .tag(0)
            
            SearchView()
                .tabItem {
                    Label("ค้นหา", systemImage: "magnifyingglass")
                }
                .tag(1)
            
            NotificationsView()
                .tabItem {
                    Label("แจ้งเตือน", systemImage: "bell.fill")
                }
                .badge(3)  // แสดง badge จำนวน
                .tag(2)
            
            ProfileView2()
                .tabItem {
                    Label("โปรไฟล์", systemImage: "person.fill")
                }
                .tag(3)
        }
        .tint(.blue)  // สีของ tab ที่เลือก
    }
}
```

### TabView กับ NavigationStack

```swift
// Pattern ที่ดี: แต่ละ Tab มี NavigationStack ของตัวเอง
struct AppWithTabs: View {
    var body: some View {
        TabView {
            // Tab 1: NavigationStack ของตัวเอง
            NavigationStack {
                HomeView()
                    .navigationTitle("หน้าหลัก")
            }
            .tabItem {
                Label("หน้าหลัก", systemImage: "house")
            }
            
            // Tab 2: NavigationStack ของตัวเอง
            NavigationStack {
                ExploreView()
                    .navigationTitle("สำรวจ")
            }
            .tabItem {
                Label("สำรวจ", systemImage: "safari")
            }
            
            // Tab 3: ไม่จำเป็นต้องมี NavigationStack
            MessagesView()
                .tabItem {
                    Label("ข้อความ", systemImage: "message")
                }
        }
    }
}
```

### Tab Badge และ Customization

```swift
struct AdvancedTabView: View {
    @State private var selectedTab = Tab.home
    @State private var notificationCount = 5
    @State private var messageCount = 0
    
    enum Tab {
        case home, search, notifications, messages, profile
    }
    
    var body: some View {
        TabView(selection: $selectedTab) {
            NavigationStack {
                Text("หน้าหลัก")
                    .navigationTitle("หน้าหลัก")
            }
            .tabItem {
                Label("หน้าหลัก", systemImage: selectedTab == .home ? "house.fill" : "house")
            }
            .tag(Tab.home)
            
            NavigationStack {
                Text("ค้นหา")
                    .navigationTitle("ค้นหา")
            }
            .tabItem {
                Label("ค้นหา", systemImage: "magnifyingglass")
            }
            .tag(Tab.search)
            
            NavigationStack {
                Text("การแจ้งเตือน")
                    .navigationTitle("การแจ้งเตือน")
            }
            .tabItem {
                Label("แจ้งเตือน", systemImage: "bell")
            }
            .badge(notificationCount > 0 ? notificationCount : nil)
            .tag(Tab.notifications)
            
            NavigationStack {
                Text("ข้อความ")
                    .navigationTitle("ข้อความ")
            }
            .tabItem {
                Label("ข้อความ", systemImage: "message")
            }
            .badge(messageCount > 0 ? "\(messageCount)" : nil)
            .tag(Tab.messages)
        }
    }
}
```

---

## 7. Sheet Presentation

`sheet` ใช้สำหรับแสดง View แบบ modal ที่ลื่นขึ้นมาจากด้านล่าง

### Sheet พื้นฐาน

```swift
struct SheetDemoView: View {
    @State private var showingSheet = false
    @State private var selectedUser: UserProfile? = nil
    
    var body: some View {
        VStack(spacing: 20) {
            Button("แสดง Sheet") {
                showingSheet = true
            }
            .buttonStyle(.borderedProminent)
            
            Button("แสดง Sheet พร้อมข้อมูล") {
                selectedUser = UserProfile(id: 1, name: "สมชาย")
            }
            .buttonStyle(.bordered)
        }
        // Sheet แบบ Boolean
        .sheet(isPresented: $showingSheet) {
            SheetContent()
        }
        // Sheet แบบ Optional (แสดงเมื่อมีค่า)
        .sheet(item: $selectedUser) { user in
            UserSheetView(user: user)
        }
    }
}

struct SheetContent: View {
    @Environment(\.dismiss) var dismiss
    
    var body: some View {
        NavigationStack {
            VStack {
                Text("นี่คือ Sheet")
                    .font(.title)
            }
            .navigationTitle("Sheet")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .cancellationAction) {
                    Button("ปิด") {
                        dismiss()
                    }
                }
            }
        }
    }
}

struct UserSheetView: View {
    let user: UserProfile
    @Environment(\.dismiss) var dismiss
    
    var body: some View {
        NavigationStack {
            VStack {
                Text("ผู้ใช้: \(user.name)")
                    .font(.title)
            }
            .navigationTitle("โปรไฟล์")
            .toolbar {
                ToolbarItem(placement: .confirmationAction) {
                    Button("เสร็จสิ้น") {
                        dismiss()
                    }
                }
            }
        }
    }
}
```

### Sheet ขนาดต่างๆ (detents)

```swift
struct DetentSheetView: View {
    @State private var showSmall = false
    @State private var showMedium = false
    @State private var showLarge = false
    @State private var showCustom = false
    
    var body: some View {
        VStack(spacing: 20) {
            Button("Sheet เล็ก") { showSmall = true }
            Button("Sheet กลาง") { showMedium = true }
            Button("Sheet ใหญ่") { showLarge = true }
            Button("Sheet กำหนดเอง") { showCustom = true }
        }
        .sheet(isPresented: $showSmall) {
            Text("Sheet เล็ก")
                .presentationDetents([.height(200)])
        }
        .sheet(isPresented: $showMedium) {
            Text("Sheet กลาง")
                .presentationDetents([.medium])
        }
        .sheet(isPresented: $showLarge) {
            Text("Sheet ใหญ่")
                .presentationDetents([.large])
        }
        .sheet(isPresented: $showCustom) {
            Text("Sheet กำหนดเอง (สามารถลากได้)")
                .presentationDetents([.height(200), .medium, .large])
                .presentationDragIndicator(.visible)
        }
    }
}
```

---

## 8. fullScreenCover

`fullScreenCover` แสดง View แบบ fullscreen บน modal ซึ่งต่างจาก sheet ที่ cover ทั้งหน้าจอ

```swift
struct FullScreenDemoView: View {
    @State private var showingFullScreen = false
    @State private var showingOnboarding = false
    
    var body: some View {
        VStack {
            Button("แสดง Full Screen") {
                showingFullScreen = true
            }
            .buttonStyle(.borderedProminent)
            
            Button("Onboarding") {
                showingOnboarding = true
            }
        }
        .fullScreenCover(isPresented: $showingFullScreen) {
            FullScreenContent()
        }
        .fullScreenCover(isPresented: $showingOnboarding) {
            OnboardingView()
        }
    }
}

struct FullScreenContent: View {
    @Environment(\.dismiss) var dismiss
    
    var body: some View {
        ZStack {
            Color.purple.ignoresSafeArea()
            
            VStack(spacing: 20) {
                Text("Full Screen View")
                    .font(.largeTitle)
                    .fontWeight(.bold)
                    .foregroundColor(.white)
                
                Button("ปิด") {
                    dismiss()
                }
                .buttonStyle(.bordered)
                .tint(.white)
            }
        }
    }
}

struct OnboardingView: View {
    @Environment(\.dismiss) var dismiss
    @State private var currentPage = 0
    
    let pages = [
        OnboardingPage(title: "ยินดีต้อนรับ", description: "เริ่มต้นการเดินทางของคุณ", icon: "hand.wave.fill", color: .blue),
        OnboardingPage(title: "ฟีเจอร์เด็ด", description: "ค้นพบสิ่งใหม่ๆ", icon: "star.fill", color: .orange),
        OnboardingPage(title: "พร้อมแล้ว!", description: "เริ่มใช้งานได้เลย", icon: "checkmark.circle.fill", color: .green),
    ]
    
    var body: some View {
        TabView(selection: $currentPage) {
            ForEach(pages.indices, id: \.self) { index in
                OnboardingPageView(page: pages[index])
                    .tag(index)
            }
        }
        .tabViewStyle(.page)
        .overlay(alignment: .bottom) {
            VStack(spacing: 20) {
                // Page Indicators
                HStack {
                    ForEach(pages.indices, id: \.self) { index in
                        Circle()
                            .fill(currentPage == index ? Color.primary : Color.secondary.opacity(0.5))
                            .frame(width: 8, height: 8)
                            .animation(.easeInOut, value: currentPage)
                    }
                }
                
                if currentPage == pages.count - 1 {
                    Button("เริ่มใช้งาน") {
                        dismiss()
                    }
                    .buttonStyle(.borderedProminent)
                    .frame(maxWidth: .infinity)
                    .padding(.horizontal)
                } else {
                    HStack {
                        Button("ข้าม") {
                            dismiss()
                        }
                        .foregroundColor(.secondary)
                        
                        Spacer()
                        
                        Button("ถัดไป") {
                            withAnimation {
                                currentPage += 1
                            }
                        }
                        .buttonStyle(.borderedProminent)
                    }
                    .padding(.horizontal)
                }
            }
            .padding(.bottom, 40)
        }
    }
}

struct OnboardingPage: Identifiable {
    let id = UUID()
    var title: String
    var description: String
    var icon: String
    var color: Color
}

struct OnboardingPageView: View {
    let page: OnboardingPage
    
    var body: some View {
        VStack(spacing: 30) {
            Image(systemName: page.icon)
                .font(.system(size: 80))
                .foregroundColor(page.color)
            
            Text(page.title)
                .font(.largeTitle)
                .fontWeight(.bold)
            
            Text(page.description)
                .font(.body)
                .foregroundColor(.secondary)
                .multilineTextAlignment(.center)
        }
        .padding(40)
    }
}
```

---

## 9. Popover

`popover` แสดง popup เล็กๆ บน iPad หรือ Mac ส่วนบน iPhone จะแสดงเป็น sheet

```swift
struct PopoverDemoView: View {
    @State private var showingPopover = false
    @State private var selectedColor = Color.blue
    
    var body: some View {
        HStack {
            Text("สีที่เลือก:")
            
            Circle()
                .fill(selectedColor)
                .frame(width: 30, height: 30)
                .onTapGesture {
                    showingPopover = true
                }
                .popover(isPresented: $showingPopover, arrowEdge: .bottom) {
                    ColorPickerPopover(selectedColor: $selectedColor)
                        .frame(width: 250, height: 200)
                        .presentationCompactAdaptation(.popover)  // iPad จะแสดงเป็น popover เสมอ
                }
        }
        .padding()
    }
}

struct ColorPickerPopover: View {
    @Binding var selectedColor: Color
    @Environment(\.dismiss) var dismiss
    
    let colors: [Color] = [.red, .orange, .yellow, .green, .blue, .purple, .pink, .gray]
    
    var body: some View {
        VStack(spacing: 15) {
            Text("เลือกสี")
                .font(.headline)
            
            LazyVGrid(columns: Array(repeating: GridItem(.flexible()), count: 4), spacing: 10) {
                ForEach(colors, id: \.self) { color in
                    Circle()
                        .fill(color)
                        .frame(width: 40, height: 40)
                        .overlay(
                            Circle()
                                .stroke(selectedColor == color ? Color.white : Color.clear, lineWidth: 3)
                        )
                        .shadow(color: selectedColor == color ? color : .clear, radius: 4)
                        .onTapGesture {
                            selectedColor = color
                            dismiss()
                        }
                }
            }
        }
        .padding()
    }
}
```

---

## 10. Alert และ ConfirmationDialog

### Alert พื้นฐาน

```swift
struct AlertDemoView: View {
    @State private var showingSimpleAlert = false
    @State private var showingActionAlert = false
    @State private var showingInputAlert = false
    @State private var alertMessage = ""
    @State private var inputText = ""
    
    var body: some View {
        VStack(spacing: 20) {
            // Alert ธรรมดา
            Button("Alert ธรรมดา") {
                showingSimpleAlert = true
            }
            .alert("แจ้งเตือน", isPresented: $showingSimpleAlert) {
                Button("ตกลง") { }
            } message: {
                Text("นี่คือข้อความแจ้งเตือน")
            }
            
            // Alert พร้อม Actions
            Button("Alert พร้อม Actions") {
                showingActionAlert = true
            }
            .alert("ยืนยันการลบ", isPresented: $showingActionAlert) {
                Button("ลบ", role: .destructive) {
                    alertMessage = "ลบแล้ว"
                }
                Button("ยกเลิก", role: .cancel) { }
            } message: {
                Text("คุณต้องการลบรายการนี้หรือไม่? การกระทำนี้ไม่สามารถยกเลิกได้")
            }
            
            if !alertMessage.isEmpty {
                Text(alertMessage)
                    .foregroundColor(.secondary)
            }
        }
        .padding()
    }
}
```

### ConfirmationDialog

```swift
struct ConfirmationDialogDemo: View {
    @State private var showingDialog = false
    @State private var selectedAction = ""
    
    var body: some View {
        VStack {
            Button("แสดง ConfirmationDialog") {
                showingDialog = true
            }
            .buttonStyle(.bordered)
            
            if !selectedAction.isEmpty {
                Text("เลือก: \(selectedAction)")
            }
        }
        .confirmationDialog("เลือกการกระทำ", isPresented: $showingDialog, titleVisibility: .visible) {
            Button("แชร์") {
                selectedAction = "แชร์"
            }
            
            Button("คัดลอกลิงก์") {
                selectedAction = "คัดลอกลิงก์"
            }
            
            Button("บันทึกภาพ") {
                selectedAction = "บันทึกภาพ"
            }
            
            Button("รายงาน", role: .destructive) {
                selectedAction = "รายงาน"
            }
            
            Button("ยกเลิก", role: .cancel) { }
        } message: {
            Text("เลือกสิ่งที่ต้องการทำกับโพสต์นี้")
        }
    }
}
```

---

## 11. Toolbar Items

`toolbar` ใช้ในการเพิ่มปุ่มและ item ลงใน Navigation Bar และ Bottom Bar

```swift
struct ToolbarDemoView: View {
    @State private var showingFilter = false
    @State private var isEditing = false
    @State private var sortOrder = "name"
    
    var body: some View {
        NavigationStack {
            List {
                ForEach(1...5, id: \.self) { i in
                    Text("รายการ \(i)")
                }
            }
            .navigationTitle("รายการ")
            .toolbar {
                // ปุ่มซ้ายบน
                ToolbarItem(placement: .navigationBarLeading) {
                    Button("กรอง") {
                        showingFilter = true
                    }
                }
                
                // ปุ่มขวาบน (หลายปุ่ม)
                ToolbarItemGroup(placement: .navigationBarTrailing) {
                    Menu {
                        Button("เรียงตามชื่อ") { sortOrder = "name" }
                        Button("เรียงตามวันที่") { sortOrder = "date" }
                        Button("เรียงตามขนาด") { sortOrder = "size" }
                    } label: {
                        Image(systemName: "arrow.up.arrow.down")
                    }
                    
                    Button(isEditing ? "เสร็จสิ้น" : "แก้ไข") {
                        isEditing.toggle()
                    }
                }
                
                // ปุ่มล่าง (iOS 16+)
                ToolbarItemGroup(placement: .bottomBar) {
                    Button {
                        // Action
                    } label: {
                        Label("เพิ่ม", systemImage: "plus")
                    }
                    
                    Spacer()
                    
                    Button {
                        // Action
                    } label: {
                        Label("ลบ", systemImage: "trash")
                    }
                    .foregroundColor(.red)
                }
            }
        }
    }
}
```

---

## 12. navigationTitle และ navigationBarTitleDisplayMode

```swift
struct NavigationTitleDemo: View {
    var body: some View {
        NavigationStack {
            VStack {
                NavigationLink("หัวข้อใหญ่ (Large)") {
                    Text("หน้าถัดไป")
                        .navigationTitle("หัวข้อใหญ่")
                        .navigationBarTitleDisplayMode(.large)  // default
                }
                
                NavigationLink("หัวข้อเล็ก (Inline)") {
                    Text("หน้าถัดไป")
                        .navigationTitle("หัวข้อเล็ก")
                        .navigationBarTitleDisplayMode(.inline)
                }
                
                NavigationLink("ซ่อนหัวข้อ") {
                    Text("หน้าไม่มีหัวข้อ")
                        .navigationTitle("")
                        .navigationBarBackButtonHidden(false)
                }
            }
            .navigationTitle("ตัวอย่างหัวข้อ")
        }
    }
}
```

---

## 13. Back Button และ Navigation Behavior

### ควบคุม Back Button

```swift
struct CustomBackButtonView: View {
    @Environment(\.dismiss) var dismiss
    @State private var hasChanges = false
    @State private var showingDiscardAlert = false
    
    var body: some View {
        Form {
            Toggle("มีการเปลี่ยนแปลง", isOn: $hasChanges)
        }
        .navigationTitle("แก้ไขข้อมูล")
        .navigationBarBackButtonHidden(hasChanges)  // ซ่อน back button เมื่อมีการเปลี่ยนแปลง
        .toolbar {
            if hasChanges {
                ToolbarItem(placement: .navigationBarLeading) {
                    Button("ยกเลิก") {
                        showingDiscardAlert = true
                    }
                    .foregroundColor(.red)
                }
                
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button("บันทึก") {
                        // บันทึกข้อมูล
                        hasChanges = false
                        dismiss()
                    }
                    .fontWeight(.bold)
                }
            }
        }
        .alert("ยกเลิกการเปลี่ยนแปลง?", isPresented: $showingDiscardAlert) {
            Button("ยกเลิก", role: .destructive) {
                hasChanges = false
                dismiss()
            }
            Button("แก้ไขต่อ", role: .cancel) { }
        } message: {
            Text("การเปลี่ยนแปลงที่ยังไม่ได้บันทึกจะหายไป")
        }
    }
}
```

---

## 14. Deep Linking และ Navigation State

Deep Linking ช่วยให้เราสามารถเปิดแอปไปยังหน้าที่ต้องการได้โดยตรงจาก URL

```swift
// กำหนด App URL Scheme ใน Info.plist:
// CFBundleURLSchemes: myapp

// Navigation Model
@Observable
class AppNavigationState {
    var path = NavigationPath()
    var selectedTab = 0
    var presentedSheet: SheetType? = nil
    
    enum SheetType: Identifiable {
        case settings
        case newPost
        case userProfile(Int)
        
        var id: String {
            switch self {
            case .settings: return "settings"
            case .newPost: return "newPost"
            case .userProfile(let id): return "user-\(id)"
            }
        }
    }
    
    func handleDeepLink(_ url: URL) {
        guard url.scheme == "myapp" else { return }
        
        let components = url.pathComponents
        
        switch url.host {
        case "home":
            selectedTab = 0
            path.removeLast(path.count)
            
        case "profile":
            selectedTab = 3
            if let userId = Int(components.dropFirst().first ?? "") {
                path.append(UserProfile(id: userId, name: "User \(userId)"))
            }
            
        case "settings":
            presentedSheet = .settings
            
        case "post":
            selectedTab = 0
            if let postId = Int(components.dropFirst().first ?? "") {
                path.append(PostDetail(id: postId, title: "Post \(postId)", content: ""))
            }
            
        default:
            break
        }
    }
}

struct DeepLinkApp: View {
    @State private var navigationState = AppNavigationState()
    
    var body: some View {
        TabView(selection: $navigationState.selectedTab) {
            NavigationStack(path: $navigationState.path) {
                HomeScreen()
                    .navigationDestination(for: PostDetail.self) { post in
                        PostDetailView(post: post, path: $navigationState.path)
                    }
                    .navigationDestination(for: UserProfile.self) { user in
                        UserProfileView(user: user, path: $navigationState.path)
                    }
            }
            .tabItem { Label("หน้าหลัก", systemImage: "house") }
            .tag(0)
            
            NavigationStack {
                Text("ค้นหา").navigationTitle("ค้นหา")
            }
            .tabItem { Label("ค้นหา", systemImage: "magnifyingglass") }
            .tag(1)
        }
        .sheet(item: $navigationState.presentedSheet) { sheet in
            switch sheet {
            case .settings:
                Text("การตั้งค่า")
            case .newPost:
                Text("โพสต์ใหม่")
            case .userProfile(let id):
                Text("โปรไฟล์ผู้ใช้ \(id)")
            }
        }
        .onOpenURL { url in
            navigationState.handleDeepLink(url)
        }
    }
}

struct HomeScreen: View {
    var body: some View {
        List {
            Text("หน้าหลัก")
        }
        .navigationTitle("หน้าหลัก")
    }
}
```

---

## 15. แบบฝึกหัด: Multi-Screen App

### โจทย์: สร้างแอป Recipe Book

สร้างแอปสมุดสูตรอาหารที่มี:
- TabView: "สูตรทั้งหมด", "โปรดปราน", "เพิ่มสูตร"
- NavigationStack: รายการสูตร → รายละเอียดสูตร
- Sheet: ฟอร์มเพิ่มสูตรอาหาร
- Alert: ยืนยันการลบสูตร

```swift
// เฉลย: Recipe Book App

// MARK: - Models

struct Recipe: Identifiable, Hashable, Codable {
    let id: UUID
    var title: String
    var description: String
    var category: RecipeCategory
    var ingredients: [String]
    var steps: [String]
    var cookingTime: Int  // minutes
    var isFavorite: Bool
    var difficulty: Difficulty
    
    init(title: String, description: String, category: RecipeCategory,
         ingredients: [String], steps: [String], cookingTime: Int,
         difficulty: Difficulty = .medium) {
        self.id = UUID()
        self.title = title
        self.description = description
        self.category = category
        self.ingredients = ingredients
        self.steps = steps
        self.cookingTime = cookingTime
        self.isFavorite = false
        self.difficulty = difficulty
    }
    
    enum RecipeCategory: String, Codable, CaseIterable {
        case breakfast = "อาหารเช้า"
        case lunch = "อาหารกลางวัน"
        case dinner = "อาหารเย็น"
        case dessert = "ของหวาน"
        case drink = "เครื่องดื่ม"
        
        var icon: String {
            switch self {
            case .breakfast: return "sun.rise"
            case .lunch: return "sun.max"
            case .dinner: return "moon.stars"
            case .dessert: return "birthday.cake"
            case .drink: return "cup.and.saucer"
            }
        }
    }
    
    enum Difficulty: String, Codable, CaseIterable {
        case easy = "ง่าย"
        case medium = "ปานกลาง"
        case hard = "ยาก"
        
        var color: Color {
            switch self {
            case .easy: return .green
            case .medium: return .orange
            case .hard: return .red
            }
        }
    }
}

// MARK: - ViewModel

@Observable
class RecipeStore {
    var recipes: [Recipe] = Recipe.sampleRecipes
    
    var favoriteRecipes: [Recipe] {
        recipes.filter { $0.isFavorite }
    }
    
    func toggleFavorite(_ recipe: Recipe) {
        if let index = recipes.firstIndex(where: { $0.id == recipe.id }) {
            recipes[index].isFavorite.toggle()
        }
    }
    
    func addRecipe(_ recipe: Recipe) {
        recipes.insert(recipe, at: 0)
    }
    
    func deleteRecipe(_ recipe: Recipe) {
        recipes.removeAll { $0.id == recipe.id }
    }
}

// MARK: - Main App View

struct RecipeBookApp: View {
    @State private var store = RecipeStore()
    @State private var selectedTab = 0
    @State private var showingAddRecipe = false
    
    var body: some View {
        TabView(selection: $selectedTab) {
            // Tab 1: สูตรทั้งหมด
            RecipeListTab(store: store)
                .tabItem {
                    Label("สูตรทั้งหมด", systemImage: "list.bullet")
                }
                .tag(0)
            
            // Tab 2: โปรดปราน
            FavoriteRecipesTab(store: store)
                .tabItem {
                    Label("โปรดปราน", systemImage: "heart.fill")
                }
                .badge(store.favoriteRecipes.count)
                .tag(1)
            
            // Tab 3: เพิ่มสูตร
            NavigationStack {
                AddRecipeView(store: store)
                    .navigationTitle("เพิ่มสูตรอาหาร")
            }
            .tabItem {
                Label("เพิ่มสูตร", systemImage: "plus.circle.fill")
            }
            .tag(2)
        }
        .tint(.orange)
    }
}

// MARK: - Recipe List Tab

struct RecipeListTab: View {
    let store: RecipeStore
    @State private var searchText = ""
    @State private var selectedCategory: Recipe.RecipeCategory? = nil
    
    var filteredRecipes: [Recipe] {
        var result = store.recipes
        
        if !searchText.isEmpty {
            result = result.filter { $0.title.localizedCaseInsensitiveContains(searchText) }
        }
        
        if let category = selectedCategory {
            result = result.filter { $0.category == category }
        }
        
        return result
    }
    
    var body: some View {
        NavigationStack {
            VStack(spacing: 0) {
                // Category Filter
                ScrollView(.horizontal, showsIndicators: false) {
                    HStack(spacing: 10) {
                        CategoryChip(
                            title: "ทั้งหมด",
                            icon: "list.bullet",
                            isSelected: selectedCategory == nil,
                            action: { selectedCategory = nil }
                        )
                        
                        ForEach(Recipe.RecipeCategory.allCases, id: \.self) { category in
                            CategoryChip(
                                title: category.rawValue,
                                icon: category.icon,
                                isSelected: selectedCategory == category,
                                action: { selectedCategory = category }
                            )
                        }
                    }
                    .padding(.horizontal)
                    .padding(.vertical, 10)
                }
                
                // Recipe List
                List {
                    ForEach(filteredRecipes) { recipe in
                        NavigationLink(value: recipe) {
                            RecipeRow(recipe: recipe, store: store)
                        }
                    }
                    .onDelete { indexSet in
                        indexSet.forEach { index in
                            store.deleteRecipe(filteredRecipes[index])
                        }
                    }
                }
                .listStyle(.plain)
            }
            .searchable(text: $searchText, prompt: "ค้นหาสูตรอาหาร")
            .navigationDestination(for: Recipe.self) { recipe in
                RecipeDetailView(recipe: recipe, store: store)
            }
            .navigationTitle("สมุดสูตรอาหาร")
        }
    }
}

struct CategoryChip: View {
    let title: String
    let icon: String
    let isSelected: Bool
    let action: () -> Void
    
    var body: some View {
        Button(action: action) {
            Label(title, systemImage: icon)
                .font(.caption)
                .fontWeight(isSelected ? .semibold : .regular)
                .padding(.horizontal, 12)
                .padding(.vertical, 6)
                .background(isSelected ? Color.orange : Color(.systemGray6))
                .foregroundColor(isSelected ? .white : .primary)
                .clipShape(Capsule())
        }
    }
}

struct RecipeRow: View {
    let recipe: Recipe
    let store: RecipeStore
    
    var body: some View {
        HStack(spacing: 15) {
            RoundedRectangle(cornerRadius: 10)
                .fill(Color.orange.opacity(0.1))
                .frame(width: 60, height: 60)
                .overlay(
                    Image(systemName: recipe.category.icon)
                        .font(.title2)
                        .foregroundColor(.orange)
                )
            
            VStack(alignment: .leading, spacing: 4) {
                Text(recipe.title)
                    .font(.headline)
                    .lineLimit(1)
                
                HStack(spacing: 8) {
                    Label("\(recipe.cookingTime) นาที", systemImage: "clock")
                    Text("•")
                    Text(recipe.difficulty.rawValue)
                        .foregroundColor(recipe.difficulty.color)
                }
                .font(.caption)
                .foregroundColor(.secondary)
            }
            
            Spacer()
            
            Button {
                withAnimation {
                    store.toggleFavorite(recipe)
                }
            } label: {
                Image(systemName: recipe.isFavorite ? "heart.fill" : "heart")
                    .foregroundColor(recipe.isFavorite ? .red : .gray)
            }
            .buttonStyle(.plain)
        }
        .padding(.vertical, 4)
    }
}

// MARK: - Recipe Detail View

struct RecipeDetailView: View {
    let recipe: Recipe
    let store: RecipeStore
    @State private var showingDeleteAlert = false
    @Environment(\.dismiss) var dismiss
    
    // ใช้ computed property เพื่อดึงข้อมูลล่าสุด
    var currentRecipe: Recipe {
        store.recipes.first { $0.id == recipe.id } ?? recipe
    }
    
    var body: some View {
        ScrollView {
            VStack(alignment: .leading, spacing: 20) {
                // Header Image
                RoundedRectangle(cornerRadius: 0)
                    .fill(Color.orange.opacity(0.1))
                    .frame(height: 250)
                    .overlay(
                        Image(systemName: currentRecipe.category.icon)
                            .font(.system(size: 80))
                            .foregroundColor(.orange)
                    )
                
                VStack(alignment: .leading, spacing: 16) {
                    // Title & Info
                    VStack(alignment: .leading, spacing: 8) {
                        Text(currentRecipe.title)
                            .font(.title)
                            .fontWeight(.bold)
                        
                        Text(currentRecipe.description)
                            .foregroundColor(.secondary)
                        
                        HStack(spacing: 20) {
                            InfoBadge(
                                icon: "clock",
                                text: "\(currentRecipe.cookingTime) นาที",
                                color: .blue
                            )
                            InfoBadge(
                                icon: "chart.bar",
                                text: currentRecipe.difficulty.rawValue,
                                color: currentRecipe.difficulty.color
                            )
                            InfoBadge(
                                icon: currentRecipe.category.icon,
                                text: currentRecipe.category.rawValue,
                                color: .orange
                            )
                        }
                    }
                    
                    Divider()
                    
                    // Ingredients
                    VStack(alignment: .leading, spacing: 10) {
                        Text("ส่วนผสม")
                            .font(.title2)
                            .fontWeight(.bold)
                        
                        ForEach(currentRecipe.ingredients.indices, id: \.self) { index in
                            HStack(alignment: .top, spacing: 10) {
                                Circle()
                                    .fill(Color.orange)
                                    .frame(width: 8, height: 8)
                                    .padding(.top, 6)
                                Text(currentRecipe.ingredients[index])
                            }
                        }
                    }
                    
                    Divider()
                    
                    // Steps
                    VStack(alignment: .leading, spacing: 12) {
                        Text("วิธีทำ")
                            .font(.title2)
                            .fontWeight(.bold)
                        
                        ForEach(currentRecipe.steps.indices, id: \.self) { index in
                            HStack(alignment: .top, spacing: 12) {
                                Text("\(index + 1)")
                                    .font(.headline)
                                    .foregroundColor(.white)
                                    .frame(width: 28, height: 28)
                                    .background(Color.orange)
                                    .clipShape(Circle())
                                
                                Text(currentRecipe.steps[index])
                                    .fixedSize(horizontal: false, vertical: true)
                            }
                        }
                    }
                }
                .padding(.horizontal)
            }
        }
        .ignoresSafeArea(edges: .top)
        .navigationTitle(currentRecipe.title)
        .navigationBarTitleDisplayMode(.inline)
        .toolbar {
            ToolbarItemGroup(placement: .navigationBarTrailing) {
                Button {
                    withAnimation {
                        store.toggleFavorite(currentRecipe)
                    }
                } label: {
                    Image(systemName: currentRecipe.isFavorite ? "heart.fill" : "heart")
                        .foregroundColor(currentRecipe.isFavorite ? .red : .primary)
                }
                
                Menu {
                    Button("แชร์สูตร") { }
                    Button("แก้ไขสูตร") { }
                    Divider()
                    Button("ลบสูตร", role: .destructive) {
                        showingDeleteAlert = true
                    }
                } label: {
                    Image(systemName: "ellipsis.circle")
                }
            }
        }
        .alert("ลบสูตรอาหาร?", isPresented: $showingDeleteAlert) {
            Button("ลบ", role: .destructive) {
                store.deleteRecipe(currentRecipe)
                dismiss()
            }
            Button("ยกเลิก", role: .cancel) { }
        } message: {
            Text("คุณต้องการลบสูตร \"\(currentRecipe.title)\" หรือไม่?")
        }
    }
}

struct InfoBadge: View {
    let icon: String
    let text: String
    let color: Color
    
    var body: some View {
        HStack(spacing: 4) {
            Image(systemName: icon)
            Text(text)
        }
        .font(.caption)
        .padding(.horizontal, 10)
        .padding(.vertical, 6)
        .background(color.opacity(0.1))
        .foregroundColor(color)
        .clipShape(Capsule())
    }
}

// MARK: - Favorites Tab

struct FavoriteRecipesTab: View {
    let store: RecipeStore
    
    var body: some View {
        NavigationStack {
            Group {
                if store.favoriteRecipes.isEmpty {
                    VStack(spacing: 20) {
                        Image(systemName: "heart.slash")
                            .font(.system(size: 60))
                            .foregroundColor(.secondary)
                        Text("ยังไม่มีสูตรโปรด")
                            .font(.title2)
                        Text("กดหัวใจที่สูตรอาหารเพื่อเพิ่มในรายการโปรด")
                            .foregroundColor(.secondary)
                            .multilineTextAlignment(.center)
                    }
                    .padding()
                } else {
                    List(store.favoriteRecipes) { recipe in
                        NavigationLink(value: recipe) {
                            RecipeRow(recipe: recipe, store: store)
                        }
                    }
                    .navigationDestination(for: Recipe.self) { recipe in
                        RecipeDetailView(recipe: recipe, store: store)
                    }
                }
            }
            .navigationTitle("สูตรโปรด")
        }
    }
}

// MARK: - Add Recipe View

@Observable
class NewRecipeForm {
    var title = ""
    var description = ""
    var category: Recipe.RecipeCategory = .dinner
    var ingredients: [String] = [""]
    var steps: [String] = [""]
    var cookingTime = 30
    var difficulty: Recipe.Difficulty = .medium
    
    var isValid: Bool {
        !title.trimmingCharacters(in: .whitespaces).isEmpty &&
        ingredients.contains { !$0.trimmingCharacters(in: .whitespaces).isEmpty } &&
        steps.contains { !$0.trimmingCharacters(in: .whitespaces).isEmpty }
    }
    
    func toRecipe() -> Recipe {
        Recipe(
            title: title.trimmingCharacters(in: .whitespaces),
            description: description,
            category: category,
            ingredients: ingredients.filter { !$0.trimmingCharacters(in: .whitespaces).isEmpty },
            steps: steps.filter { !$0.trimmingCharacters(in: .whitespaces).isEmpty },
            cookingTime: cookingTime,
            difficulty: difficulty
        )
    }
    
    func reset() {
        title = ""
        description = ""
        category = .dinner
        ingredients = [""]
        steps = [""]
        cookingTime = 30
        difficulty = .medium
    }
}

struct AddRecipeView: View {
    let store: RecipeStore
    @State private var form = NewRecipeForm()
    @State private var showingSuccessAlert = false
    
    var body: some View {
        Form {
            Section("ข้อมูลทั่วไป") {
                TextField("ชื่อสูตรอาหาร", text: $form.title)
                TextField("คำอธิบาย (ไม่บังคับ)", text: $form.description, axis: .vertical)
                    .lineLimit(3)
                
                Picker("หมวดหมู่", selection: $form.category) {
                    ForEach(Recipe.RecipeCategory.allCases, id: \.self) { category in
                        Label(category.rawValue, systemImage: category.icon)
                            .tag(category)
                    }
                }
                
                Picker("ระดับความยาก", selection: $form.difficulty) {
                    ForEach(Recipe.Difficulty.allCases, id: \.self) { diff in
                        Text(diff.rawValue).tag(diff)
                    }
                }
                
                Stepper("เวลาทำ: \(form.cookingTime) นาที",
                       value: $form.cookingTime,
                       in: 5...300,
                       step: 5)
            }
            
            Section("ส่วนผสม") {
                ForEach(form.ingredients.indices, id: \.self) { index in
                    HStack {
                        TextField("ส่วนผสมที่ \(index + 1)", text: $form.ingredients[index])
                        if form.ingredients.count > 1 {
                            Button {
                                form.ingredients.remove(at: index)
                            } label: {
                                Image(systemName: "minus.circle.fill")
                                    .foregroundColor(.red)
                            }
                            .buttonStyle(.plain)
                        }
                    }
                }
                
                Button {
                    form.ingredients.append("")
                } label: {
                    Label("เพิ่มส่วนผสม", systemImage: "plus.circle")
                }
            }
            
            Section("ขั้นตอนการทำ") {
                ForEach(form.steps.indices, id: \.self) { index in
                    HStack(alignment: .top) {
                        Text("\(index + 1).")
                            .foregroundColor(.orange)
                            .fontWeight(.bold)
                        
                        TextField("ขั้นตอนที่ \(index + 1)", text: $form.steps[index], axis: .vertical)
                            .lineLimit(3)
                        
                        if form.steps.count > 1 {
                            Button {
                                form.steps.remove(at: index)
                            } label: {
                                Image(systemName: "minus.circle.fill")
                                    .foregroundColor(.red)
                            }
                            .buttonStyle(.plain)
                        }
                    }
                }
                
                Button {
                    form.steps.append("")
                } label: {
                    Label("เพิ่มขั้นตอน", systemImage: "plus.circle")
                }
            }
            
            Section {
                Button("บันทึกสูตร") {
                    let recipe = form.toRecipe()
                    store.addRecipe(recipe)
                    form.reset()
                    showingSuccessAlert = true
                }
                .frame(maxWidth: .infinity)
                .fontWeight(.semibold)
                .disabled(!form.isValid)
            }
        }
        .alert("บันทึกสูตรสำเร็จ!", isPresented: $showingSuccessAlert) {
            Button("ตกลง") { }
        } message: {
            Text("สูตรอาหารถูกเพิ่มในรายการแล้ว")
        }
    }
}

// MARK: - Sample Data

extension Recipe {
    static var sampleRecipes: [Recipe] = [
        Recipe(
            title: "ข้าวผัดกะเพรา",
            description: "ข้าวผัดกะเพราไก่แบบดั้งเดิม อร่อย เผ็ด จัดจ้าน",
            category: .lunch,
            ingredients: ["ข้าวสวย 2 ถ้วย", "ไก่สับ 200 กรัม", "กะเพรา 1 กำมือ", "กระเทียม 5 กลีบ", "พริก 5 เม็ด"],
            steps: ["ตั้งกระทะ ใส่น้ำมัน", "ผัดกระเทียมและพริกให้หอม", "ใส่ไก่ผัดให้สุก", "ใส่กะเพรา ปรุงรส", "ใส่ข้าว ผัดให้เข้ากัน"],
            cookingTime: 15,
            difficulty: .easy
        ),
        Recipe(
            title: "ต้มยำกุ้ง",
            description: "ต้มยำกุ้งน้ำข้น รสเด็ดเผ็ดร้อน",
            category: .dinner,
            ingredients: ["กุ้งใหญ่ 300 กรัม", "เห็ดฟาง 100 กรัม", "ตะไคร้ 2 ต้น", "ข่า 5 แว่น", "ใบมะกรูด 5 ใบ"],
            steps: ["ต้มน้ำให้เดือด", "ใส่ตะไคร้ ข่า ใบมะกรูด", "ใส่กุ้งและเห็ด", "ปรุงรส ใส่น้ำมะนาว"],
            cookingTime: 20,
            difficulty: .medium
        ),
    ]
}

#Preview {
    RecipeBookApp()
}
```

---

## 16. สรุป

ในบทนี้เราได้เรียนรู้ระบบ Navigation ทั้งหมดของ SwiftUI อย่างละเอียด

### สิ่งที่เรียนรู้

1. **NavigationStack** - Container สำหรับ stack-based navigation (iOS 16+)
2. **NavigationLink** - สร้าง link สำหรับ navigate ไปหน้าอื่น
3. **navigationDestination** - กำหนด destination สำหรับแต่ละ type
4. **NavigationPath** - ควบคุม navigation stack ด้วยโค้ด
5. **NavigationSplitView** - สำหรับ iPad/Mac แบบ sidebar + content
6. **TabView** - แสดง tab bar สำหรับสลับ section หลัก
7. **sheet** - แสดง modal view แบบ slide up
8. **fullScreenCover** - แสดง modal view แบบ fullscreen
9. **popover** - แสดง popup เล็กๆ
10. **alert** - แสดง dialog แจ้งเตือน
11. **confirmationDialog** - แสดง action sheet
12. **toolbar** - เพิ่มปุ่มใน navigation bar
13. **navigationTitle** - กำหนดหัวข้อใน navigation bar
14. **Deep Linking** - เปิดแอปไปหน้าที่ต้องการจาก URL

### หลักการสำคัญ

- ใช้ **NavigationStack** แทน NavigationView สำหรับ iOS 16+
- ใช้ **navigationDestination** แทนการ embed View ใน NavigationLink
- จัดการ Navigation State ด้วย **NavigationPath** สำหรับ programmatic navigation
- แต่ละ Tab ควรมี **NavigationStack** ของตัวเอง
- ใช้ **@Environment(\.dismiss)** เพื่อปิด modal view

### บทถัดไป

บทที่ 25 จะพาไปเรียนรู้ **SwiftUI Lists and Collections** ซึ่งครอบคลุม List, LazyVStack, LazyHStack, Grid และวิธีการแสดงผลข้อมูลในรูปแบบต่างๆ

---

*จบบทที่ 24: SwiftUI Navigation*
