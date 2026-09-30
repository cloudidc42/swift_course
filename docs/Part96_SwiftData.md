# Part 96: SwiftData — Modern Persistence Framework (iOS 17+)

## บทนำ

SwiftData คือ persistence framework ที่ Apple แนะนำใน iOS 17, macOS 14, watchOS 10, และ tvOS 17 โดยสร้างบน Core Data แต่ใช้ Swift-native API ที่ทันสมัยพร้อม macros, property wrappers, และ Swift Concurrency integration ทำให้การจัดการข้อมูลง่ายขึ้นอย่างมาก

---

## 1. SwiftData Overview

### 1.1 SwiftData vs Core Data

| ฟีเจอร์ | Core Data | SwiftData |
|---------|-----------|-----------|
| ภาษา | Objective-C origin | Swift-native |
| Model definition | .xcdatamodeld file | `@Model` macro |
| Querying | NSFetchRequest | FetchDescriptor + #Predicate |
| Context | NSManagedObjectContext | ModelContext |
| SwiftUI integration | @FetchRequest | @Query |
| Concurrency | PerformAndWait | async/await + ModelActor |
| CloudKit | NSPersistentCloudKitContainer | อัตโนมัติ |

**Core Data (วิธีเดิม):**
```swift
// Core Data - verbose และ error-prone
import CoreData

class PersonEntity: NSManagedObject {
    @NSManaged var name: String?
    @NSManaged var age: Int32
    @NSManaged var email: String?
}

// สร้าง persistent container
lazy var persistentContainer: NSPersistentContainer = {
    let container = NSPersistentContainer(name: "DataModel")
    container.loadPersistentStores { _, error in
        if let error = error {
            fatalError("Failed to load Core Data: \(error)")
        }
    }
    return container
}()

// Insert
let context = persistentContainer.viewContext
let person = PersonEntity(context: context)
person.name = "Alice"
person.age = 30
try? context.save()

// Fetch
let request: NSFetchRequest<PersonEntity> = PersonEntity.fetchRequest()
request.predicate = NSPredicate(format: "age > %d", 25)
let results = try? context.fetch(request)
```

**SwiftData (วิธีใหม่):**
```swift
// SwiftData - clean และ Swift-native
import SwiftData

@Model
class Person {
    var name: String
    var age: Int
    var email: String
    
    init(name: String, age: Int, email: String) {
        self.name = name
        self.age = age
        self.email = email
    }
}

// Setup
let container = try ModelContainer(for: Person.self)
let context = container.mainContext

// Insert
let person = Person(name: "Alice", age: 30, email: "alice@example.com")
context.insert(person)
try context.save()

// Fetch
let descriptor = FetchDescriptor<Person>(
    predicate: #Predicate { $0.age > 25 },
    sortBy: [SortDescriptor(\.name)]
)
let results = try context.fetch(descriptor)
```

### 1.2 @Model Macro

`@Model` คือ macro หลักของ SwiftData ที่แปลง Swift class ธรรมดาให้กลายเป็น persisted model

```swift
import SwiftData

@Model
final class TodoItem {
    // Properties ทั้งหมดจะถูก persisted โดยอัตโนมัติ
    var title: String
    var isCompleted: Bool
    var createdAt: Date
    var priority: Int
    
    // Optional properties รองรับ
    var notes: String?
    var dueDate: Date?
    
    init(title: String, priority: Int = 0) {
        self.title = title
        self.isCompleted = false
        self.createdAt = Date()
        self.priority = priority
    }
}

// @Model expands เป็น:
// - PersistentModel conformance
// - Observation support (จาก @Observable)
// - Schema metadata
// - Property change tracking
```

### 1.3 ModelContext และ ModelContainer

```swift
// ModelContainer - manages persistent stores
// ModelContext - active space สำหรับ model objects

// Relationship:
// App
// └── ModelContainer (ตัวเดียว)
//     ├── Schema (model types)
//     ├── Persistent Store (SQLite)
//     └── ModelContext (หนึ่งหรือหลายตัว)
//         ├── mainContext (main thread)
//         └── background contexts

// สร้าง ModelContainer
let container = try ModelContainer(
    for: TodoItem.self, Category.self,
    configurations: ModelConfiguration(isStoredInMemoryOnly: false)
)

// Access main context
let context = container.mainContext
```

### 1.4 Migration from Core Data

```swift
// ถ้ามี Core Data app อยู่แล้ว สามารถ migrate ได้

// 1. ย้าย existing .xcdatamodeld data
let config = ModelConfiguration(
    url: existingStoreURL  // ชี้ไปยัง existing Core Data store
)

// 2. Map Core Data entities ไป SwiftData models
// Core Data entity "PersonEntity" → SwiftData @Model class "Person"

// 3. Handle migration
// SwiftData จะ auto-migrate ถ้า schema compatible
// สำหรับ breaking changes ต้องใช้ VersionedSchema (ดูใน section 8)
```

---

## 2. Defining Models with @Model

### 2.1 @Model Class Definition

```swift
import SwiftData
import Foundation

// Basic model
@Model
final class Product {
    var name: String
    var price: Decimal
    var quantity: Int
    var createdAt: Date
    
    // SwiftData รองรับ enum ถ้า enum เป็น Codable
    var status: ProductStatus
    
    // Computed properties - ไม่ถูก persist
    var isInStock: Bool { quantity > 0 }
    
    init(name: String, price: Decimal, quantity: Int) {
        self.name = name
        self.price = price
        self.quantity = quantity
        self.createdAt = Date()
        self.status = .active
    }
}

// Enum ต้อง Codable เพื่อใช้ใน @Model
enum ProductStatus: String, Codable {
    case active
    case inactive
    case discontinued
}
```

### 2.2 @Attribute with Options

```swift
import SwiftData

@Model
final class User {
    // @Attribute(.unique) - ต้องไม่ซ้ำกัน
    @Attribute(.unique)
    var email: String
    
    // @Attribute(.unique) บน UUID - common pattern
    @Attribute(.unique)
    var id: UUID
    
    var username: String
    var passwordHash: String
    var createdAt: Date
    
    // @Attribute(.preserveValueOnDeletion) - เก็บค่าแม้ related object ถูกลบ
    @Attribute(.preserveValueOnDeletion)
    var lastKnownLocation: String?
    
    // External storage สำหรับ large binary data
    @Attribute(.externalStorage)
    var profileImageData: Data?
    
    // Spotlight searchable - index สำหรับ Core Spotlight
    @Attribute(.spotlight)
    var displayName: String
    
    init(email: String, username: String, passwordHash: String) {
        self.id = UUID()
        self.email = email
        self.username = username
        self.passwordHash = passwordHash
        self.createdAt = Date()
        self.displayName = username
    }
}
```

### 2.3 @Relationship with Delete Rules

```swift
import SwiftData

@Model
final class Author {
    var name: String
    var bio: String?
    
    // One-to-many: Author มีหลาย Books
    // deleteRule: .cascade - ลบ Author แล้วลบ Books ด้วย
    @Relationship(deleteRule: .cascade, inverse: \Book.author)
    var books: [Book] = []
    
    init(name: String) {
        self.name = name
    }
}

@Model
final class Book {
    var title: String
    var publishedYear: Int
    var isbn: String
    
    // Many-to-one: Book มี Author คนเดียว
    // deleteRule: .nullify - ถ้า Author ถูกลบ ให้ set เป็น nil
    @Relationship(deleteRule: .nullify)
    var author: Author?
    
    // Many-to-many: Book มีหลาย Tags
    @Relationship(inverse: \Tag.books)
    var tags: [Tag] = []
    
    init(title: String, publishedYear: Int, isbn: String) {
        self.title = title
        self.publishedYear = publishedYear
        self.isbn = isbn
    }
}

@Model
final class Tag {
    var name: String
    var color: String
    
    // Many-to-many inverse
    @Relationship(inverse: \Book.tags)
    var books: [Book] = []
    
    init(name: String, color: String = "#000000") {
        self.name = name
        self.color = color
    }
}

// Delete rules:
// .nullify  - set relationship to nil (default)
// .cascade  - delete related objects
// .deny     - prevent deletion if relationship exists
// .noAction - do nothing (dangerous - can leave orphans)
```

### 2.4 @Transient for Non-Persisted Properties

```swift
import SwiftData

@Model
final class Document {
    var title: String
    var content: String
    var lastModified: Date
    
    // @Transient - ไม่ถูก persist ลง database
    @Transient
    var isEditing: Bool = false
    
    @Transient
    var unsavedChanges: String = ""
    
    @Transient
    var cachedWordCount: Int = 0
    
    // Computed properties ก็ไม่ถูก persist เช่นกัน
    var wordCount: Int {
        content.split(separator: " ").count
    }
    
    init(title: String, content: String = "") {
        self.title = title
        self.content = content
        self.lastModified = Date()
    }
}
```

### 2.5 Supported Property Types

```swift
import SwiftData
import Foundation

@Model
final class TypeShowcase {
    // Primitive types
    var boolValue: Bool = false
    var intValue: Int = 0
    var int8Value: Int8 = 0
    var int16Value: Int16 = 0
    var int32Value: Int32 = 0
    var int64Value: Int64 = 0
    var floatValue: Float = 0.0
    var doubleValue: Double = 0.0
    var stringValue: String = ""
    
    // Foundation types
    var dateValue: Date = Date()
    var uuidValue: UUID = UUID()
    var urlValue: URL?
    var dataValue: Data?
    var decimalValue: Decimal = 0
    
    // Collections (must contain supported types)
    var stringArray: [String] = []
    var intArray: [Int] = []
    var dateArray: [Date] = []
    
    // Codable types
    var codableEnum: MyEnum = .first
    var codableStruct: MyStruct?
    
    init() {}
}

// Codable enum
enum MyEnum: String, Codable {
    case first
    case second
    case third
}

// Codable struct
struct MyStruct: Codable {
    var x: Double
    var y: Double
}
```

---

## 3. ModelContainer Setup

### 3.1 ModelConfiguration

```swift
import SwiftData

// In-memory store (สำหรับ tests หรือ previews)
let inMemoryConfig = ModelConfiguration(isStoredInMemoryOnly: true)

// On-disk store (default location)
let diskConfig = ModelConfiguration(isStoredInMemoryOnly: false)

// Custom URL
let customURL = URL.documentsDirectory.appendingPathComponent("myapp.store")
let customConfig = ModelConfiguration(url: customURL)

// Read-only store
let readOnlyConfig = ModelConfiguration(
    url: bundleStoreURL,
    isStoredInMemoryOnly: false,
    allowsSave: false
)

// สร้าง container
let container = try ModelContainer(
    for: TodoItem.self, Category.self,
    configurations: diskConfig
)
```

### 3.2 Handling Multiple Configurations

```swift
import SwiftData

// ใช้หลาย configurations สำหรับ model types ต่างกัน
let userConfig = ModelConfiguration(
    "Users",
    schema: Schema([User.self]),
    url: URL.documentsDirectory.appendingPathComponent("users.store"),
    cloudKitDatabase: .automatic
)

let cacheConfig = ModelConfiguration(
    "Cache",
    schema: Schema([CachedItem.self]),
    url: URL.documentsDirectory.appendingPathComponent("cache.store"),
    isStoredInMemoryOnly: false
)

let container = try ModelContainer(
    for: User.self, CachedItem.self,
    configurations: userConfig, cacheConfig
)
```

### 3.3 App Group Container

```swift
import SwiftData

// App Group สำหรับ share data ระหว่าง app และ extensions
let groupURL = FileManager.default
    .containerURL(forSecurityApplicationGroupIdentifier: "group.com.myapp")!
    .appendingPathComponent("shared.store")

let sharedConfig = ModelConfiguration(
    "Shared",
    schema: Schema([SharedItem.self]),
    url: groupURL
)

let container = try ModelContainer(
    for: SharedItem.self,
    configurations: sharedConfig
)
```

### 3.4 Default SwiftUI .modelContainer Modifier

```swift
import SwiftUI
import SwiftData

// App entry point
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(for: [
            TodoItem.self,
            Category.self,
            Tag.self
        ])
    }
}

// Custom configuration
@main
struct MyApp: App {
    
    let container: ModelContainer
    
    init() {
        do {
            let schema = Schema([TodoItem.self, Category.self])
            let config = ModelConfiguration(
                schema: schema,
                isStoredInMemoryOnly: false,
                cloudKitDatabase: .automatic
            )
            container = try ModelContainer(for: schema, configurations: config)
        } catch {
            fatalError("Failed to create ModelContainer: \(error)")
        }
    }
    
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(container)
    }
}
```

---

## 4. ModelContext Operations

### 4.1 insert() และ delete()

```swift
import SwiftData

class DataService {
    private let context: ModelContext
    
    init(context: ModelContext) {
        self.context = context
    }
    
    // Insert
    func createTodo(title: String, priority: Int = 0) throws -> TodoItem {
        let todo = TodoItem(title: title, priority: priority)
        context.insert(todo)
        try context.save()
        return todo
    }
    
    // Insert หลายตัว
    func createTodos(_ items: [(title: String, priority: Int)]) throws {
        for item in items {
            let todo = TodoItem(title: item.title, priority: item.priority)
            context.insert(todo)
        }
        try context.save()
    }
    
    // Delete
    func deleteTodo(_ todo: TodoItem) throws {
        context.delete(todo)
        try context.save()
    }
    
    // Delete หลายตัว
    func deleteTodos(_ todos: [TodoItem]) throws {
        todos.forEach { context.delete($0) }
        try context.save()
    }
    
    // Delete ด้วย predicate (batch delete)
    func deleteCompletedTodos() throws {
        try context.delete(
            model: TodoItem.self,
            where: #Predicate { $0.isCompleted }
        )
    }
}
```

### 4.2 save() และ autosave

```swift
import SwiftData

// Manual save
let context = container.mainContext
let todo = TodoItem(title: "Learn SwiftData")
context.insert(todo)

do {
    try context.save()
    print("Saved successfully")
} catch {
    print("Save failed: \(error)")
}

// Autosave
// SwiftData's mainContext มี autosave enabled by default
// ทำงานเมื่อ:
// 1. App goes to background
// 2. Context has pending changes และ run loop idle

// Disable autosave
let config = ModelConfiguration(isStoredInMemoryOnly: false)
let container = try ModelContainer(for: TodoItem.self, configurations: config)
container.mainContext.autosaveEnabled = false

// ตรวจสอบว่ามี unsaved changes
if context.hasChanges {
    try context.save()
}

// Rollback unsaved changes
context.rollback()
```

### 4.3 fetch() กับ FetchDescriptor

```swift
import SwiftData

func fetchExamples(context: ModelContext) throws {
    
    // Fetch ทั้งหมด
    let allTodos = try context.fetch(FetchDescriptor<TodoItem>())
    
    // Fetch พร้อม predicate
    let incompleteTodos = try context.fetch(
        FetchDescriptor<TodoItem>(
            predicate: #Predicate { !$0.isCompleted }
        )
    )
    
    // Fetch พร้อม sort
    let sortedTodos = try context.fetch(
        FetchDescriptor<TodoItem>(
            sortBy: [
                SortDescriptor(\.priority, order: .reverse),
                SortDescriptor(\.createdAt)
            ]
        )
    )
    
    // Fetch พร้อม limit
    var descriptor = FetchDescriptor<TodoItem>(
        predicate: #Predicate { !$0.isCompleted },
        sortBy: [SortDescriptor(\.priority, order: .reverse)]
    )
    descriptor.fetchLimit = 10
    descriptor.fetchOffset = 0
    let topTodos = try context.fetch(descriptor)
    
    // Count
    let count = try context.fetchCount(
        FetchDescriptor<TodoItem>(
            predicate: #Predicate { $0.isCompleted }
        )
    )
    print("Completed: \(count)")
}
```

### 4.4 Background Context

```swift
import SwiftData

// สร้าง background context สำหรับ heavy operations
func performBackgroundWork(container: ModelContainer) async {
    
    // สร้าง context ใหม่สำหรับ background work
    let bgContext = ModelContext(container)
    
    // ทำงานบน background
    await Task.detached(priority: .background) {
        do {
            // Fetch ใน background
            let descriptor = FetchDescriptor<TodoItem>(
                predicate: #Predicate { !$0.isCompleted }
            )
            let todos = try bgContext.fetch(descriptor)
            
            // Process
            for todo in todos {
                // heavy processing...
                todo.processedAt = Date()
            }
            
            // Save
            try bgContext.save()
            
        } catch {
            print("Background error: \(error)")
        }
    }.value
}

// ใช้ @ModelActor สำหรับ cleaner background work
// (ดูใน section 11)
```

---

## 5. Querying Data

### 5.1 FetchDescriptor กับ Predicate

```swift
import SwiftData

// FetchDescriptor คือ type-safe query builder
let descriptor = FetchDescriptor<TodoItem>()

// เพิ่ม predicate
var filteredDescriptor = FetchDescriptor<TodoItem>(
    predicate: #Predicate<TodoItem> { item in
        item.priority > 2 && !item.isCompleted
    }
)

// Sort
filteredDescriptor.sortBy = [
    SortDescriptor(\TodoItem.priority, order: .reverse),
    SortDescriptor(\TodoItem.createdAt, order: .forward)
]

// Pagination
filteredDescriptor.fetchLimit = 20
filteredDescriptor.fetchOffset = 40  // page 3 (0-indexed)

// Execute
let results = try context.fetch(filteredDescriptor)
```

### 5.2 #Predicate Macro สำหรับ Type-Safe Queries

```swift
import SwiftData

// #Predicate macro - compile-time validated predicates
// ไม่มี runtime string format errors แบบ NSPredicate

// Simple comparison
let highPriority = #Predicate<TodoItem> { $0.priority >= 3 }

// String operations
let containsSwift = #Predicate<TodoItem> { 
    $0.title.contains("Swift")
}

// Date comparison
let dueToday = #Predicate<TodoItem> { item in
    let today = Date()
    let tomorrow = Calendar.current.date(byAdding: .day, value: 1, to: today)!
    return item.dueDate != nil && item.dueDate! < tomorrow
}

// Compound predicates
let urgentIncomplete = #Predicate<TodoItem> { item in
    item.priority >= 3 && !item.isCompleted
}

// Optional handling
let withNotes = #Predicate<TodoItem> { item in
    item.notes != nil
}

// Relationship predicate
let booksBy2023 = #Predicate<Book> { book in
    book.publishedYear == 2023
}

// ใช้ใน FetchDescriptor
let descriptor = FetchDescriptor<TodoItem>(
    predicate: urgentIncomplete,
    sortBy: [SortDescriptor(\.priority, order: .reverse)]
)
```

### 5.3 SortDescriptor

```swift
import SwiftData

// Single sort
let byName = SortDescriptor(\TodoItem.title)
let byNameDesc = SortDescriptor(\TodoItem.title, order: .reverse)

// Multiple sorts
let multiSort: [SortDescriptor<TodoItem>] = [
    SortDescriptor(\.priority, order: .reverse),  // Priority สูงก่อน
    SortDescriptor(\.createdAt, order: .forward)   // เก่าสุดก่อน
]

// ใน FetchDescriptor
let descriptor = FetchDescriptor<TodoItem>(
    sortBy: multiSort
)
```

### 5.4 FetchDescriptor กับ limit/offset

```swift
import SwiftData

// Pagination helper
struct PaginatedFetch<T: PersistentModel> {
    let pageSize: Int
    var currentPage: Int = 0
    var predicate: Predicate<T>?
    var sortBy: [SortDescriptor<T>] = []
    
    func fetchDescriptor() -> FetchDescriptor<T> {
        var descriptor = FetchDescriptor<T>(
            predicate: predicate,
            sortBy: sortBy
        )
        descriptor.fetchLimit = pageSize
        descriptor.fetchOffset = currentPage * pageSize
        return descriptor
    }
    
    mutating func nextPage() {
        currentPage += 1
    }
    
    mutating func reset() {
        currentPage = 0
    }
}

// การใช้งาน
var paginator = PaginatedFetch<TodoItem>(
    pageSize: 20,
    predicate: #Predicate { !$0.isCompleted },
    sortBy: [SortDescriptor(\.createdAt, order: .reverse)]
)

// หน้าแรก
let firstPage = try context.fetch(paginator.fetchDescriptor())

// หน้าถัดไป
paginator.nextPage()
let secondPage = try context.fetch(paginator.fetchDescriptor())
```

### 5.5 @Query Property Wrapper ใน SwiftUI

```swift
import SwiftUI
import SwiftData

struct TodoListView: View {
    // @Query จัดการ fetch และ live updates อัตโนมัติ
    @Query var todos: [TodoItem]
    @Environment(\.modelContext) private var context
    
    var body: some View {
        List(todos) { todo in
            TodoRowView(todo: todo)
        }
        .navigationTitle("Todos (\(todos.count))")
    }
}

// @Query พร้อม predicate และ sort
struct FilteredTodoView: View {
    @Query(
        filter: #Predicate<TodoItem> { !$0.isCompleted },
        sort: \.priority,
        order: .reverse
    ) var activeTodos: [TodoItem]
    
    var body: some View {
        List(activeTodos) { todo in
            Text(todo.title)
        }
    }
}
```

---

## 6. @Query Property Wrapper

### 6.1 Basic @Query Usage

```swift
import SwiftUI
import SwiftData

struct AllItemsView: View {
    // Fetch ทุก item
    @Query var items: [TodoItem]
    
    var body: some View {
        ForEach(items) { item in
            Text(item.title)
        }
    }
}
```

### 6.2 Filtering กับ filter Parameter

```swift
import SwiftUI
import SwiftData

struct CompletedView: View {
    @Query(filter: #Predicate<TodoItem> { $0.isCompleted })
    var completedItems: [TodoItem]
    
    var body: some View {
        List(completedItems) { item in
            HStack {
                Image(systemName: "checkmark.circle.fill")
                    .foregroundColor(.green)
                Text(item.title)
                    .strikethrough()
            }
        }
    }
}

struct HighPriorityView: View {
    @Query(filter: #Predicate<TodoItem> { $0.priority >= 3 && !$0.isCompleted })
    var urgentItems: [TodoItem]
    
    var body: some View {
        List(urgentItems) { item in
            HStack {
                Image(systemName: "exclamationmark.triangle.fill")
                    .foregroundColor(.red)
                Text(item.title)
            }
        }
    }
}
```

### 6.3 Sorting กับ sort Parameter

```swift
import SwiftUI
import SwiftData

// Sort by single property
struct ByTitleView: View {
    @Query(sort: \TodoItem.title)
    var items: [TodoItem]
    
    var body: some View {
        List(items) { item in Text(item.title) }
    }
}

// Sort by multiple properties
struct MultiSortView: View {
    @Query(sort: [
        SortDescriptor(\TodoItem.priority, order: .reverse),
        SortDescriptor(\TodoItem.createdAt)
    ])
    var items: [TodoItem]
    
    var body: some View {
        List(items) { item in Text(item.title) }
    }
}
```

### 6.4 Dynamic Filter Changes

```swift
import SwiftUI
import SwiftData

// Dynamic filter ต้องสร้าง @Query ใน init
struct DynamicFilterView: View {
    let filterCompleted: Bool
    
    @Query var todos: [TodoItem]
    @Environment(\.modelContext) private var context
    
    init(filterCompleted: Bool) {
        self.filterCompleted = filterCompleted
        
        // สร้าง predicate แบบ dynamic
        let predicate = #Predicate<TodoItem> { item in
            item.isCompleted == filterCompleted
        }
        
        _todos = Query(filter: predicate, sort: \.createdAt)
    }
    
    var body: some View {
        List(todos) { todo in
            TodoRowView(todo: todo)
        }
    }
}

// ใช้งาน
struct ParentView: View {
    @State private var showCompleted = false
    
    var body: some View {
        VStack {
            Toggle("Show Completed", isOn: $showCompleted)
            DynamicFilterView(filterCompleted: showCompleted)
        }
    }
}

// Dynamic search
struct SearchableView: View {
    @State private var searchText = ""
    
    var body: some View {
        SearchResultsView(searchText: searchText)
            .searchable(text: $searchText)
    }
}

struct SearchResultsView: View {
    let searchText: String
    
    @Query var items: [TodoItem]
    
    init(searchText: String) {
        self.searchText = searchText
        
        if searchText.isEmpty {
            _items = Query(sort: \.createdAt, order: .reverse)
        } else {
            let predicate = #Predicate<TodoItem> { item in
                item.title.localizedStandardContains(searchText)
            }
            _items = Query(filter: predicate, sort: \.createdAt, order: .reverse)
        }
    }
    
    var body: some View {
        List(items) { item in
            Text(item.title)
        }
    }
}
```

---

## 7. Relationships

### 7.1 One-to-Many

```swift
import SwiftData

@Model
final class Category {
    var name: String
    var colorHex: String
    
    // One-to-many: Category มีหลาย TodoItems
    @Relationship(deleteRule: .cascade, inverse: \TodoItem.category)
    var items: [TodoItem] = []
    
    init(name: String, colorHex: String = "#007AFF") {
        self.name = name
        self.colorHex = colorHex
    }
}

@Model
final class TodoItem {
    var title: String
    var isCompleted: Bool
    var createdAt: Date
    var priority: Int
    
    // Many-to-one: TodoItem อยู่ใน Category เดียว
    var category: Category?
    
    init(title: String) {
        self.title = title
        self.isCompleted = false
        self.createdAt = Date()
        self.priority = 0
    }
}

// การใช้งาน
let workCategory = Category(name: "Work", colorHex: "#FF3B30")
let personalCategory = Category(name: "Personal", colorHex: "#34C759")
context.insert(workCategory)
context.insert(personalCategory)

let meetingTodo = TodoItem(title: "Team Meeting")
meetingTodo.category = workCategory  // Set relationship
context.insert(meetingTodo)

try context.save()

// Access relationship
print(workCategory.items.count)  // 1
print(meetingTodo.category?.name)  // "Work"
```

### 7.2 Many-to-Many

```swift
import SwiftData

@Model
final class Student {
    var name: String
    var studentId: String
    
    // Many-to-many: Student เรียนหลาย Courses
    @Relationship(inverse: \Course.students)
    var courses: [Course] = []
    
    init(name: String, studentId: String) {
        self.name = name
        self.studentId = studentId
    }
}

@Model
final class Course {
    var title: String
    var credits: Int
    
    // Many-to-many inverse
    @Relationship(inverse: \Student.courses)
    var students: [Student] = []
    
    init(title: String, credits: Int) {
        self.title = title
        self.credits = credits
    }
}

// การใช้งาน
let swift101 = Course(title: "Swift 101", credits: 3)
let ios101 = Course(title: "iOS 101", credits: 3)
context.insert(swift101)
context.insert(ios101)

let alice = Student(name: "Alice", studentId: "S001")
alice.courses = [swift101, ios101]  // Enroll in both courses
context.insert(alice)

let bob = Student(name: "Bob", studentId: "S002")
bob.courses = [swift101]  // Only Swift 101
context.insert(bob)

try context.save()

print(swift101.students.count)  // 2 (Alice and Bob)
print(ios101.students.count)    // 1 (Alice only)
print(alice.courses.count)      // 2
```

### 7.3 Inverse Relationships

```swift
import SwiftData

// SwiftData ต้องการ inverse relationship เพื่อ maintain referential integrity

@Model
final class Post {
    var title: String
    var content: String
    var createdAt: Date
    
    // Post มีหลาย Comments
    @Relationship(deleteRule: .cascade, inverse: \Comment.post)
    var comments: [Comment] = []
    
    // Post มี Author (many-to-one)
    @Relationship(deleteRule: .nullify, inverse: \User.posts)
    var author: User?
    
    init(title: String, content: String) {
        self.title = title
        self.content = content
        self.createdAt = Date()
    }
}

@Model
final class Comment {
    var content: String
    var createdAt: Date
    var upvotes: Int
    
    // Inverse ของ Post.comments
    var post: Post?
    
    // Comment มี Author
    @Relationship(inverse: \User.comments)
    var author: User?
    
    init(content: String) {
        self.content = content
        self.createdAt = Date()
        self.upvotes = 0
    }
}

@Model
final class User {
    @Attribute(.unique)
    var username: String
    var email: String
    
    // Inverse ของ Post.author
    @Relationship(deleteRule: .nullify, inverse: \Post.author)
    var posts: [Post] = []
    
    // Inverse ของ Comment.author
    @Relationship(deleteRule: .nullify, inverse: \Comment.author)
    var comments: [Comment] = []
    
    init(username: String, email: String) {
        self.username = username
        self.email = email
    }
}
```

### 7.4 Cascade Delete vs Nullify

```swift
// Delete Rules:

// .cascade - ลบ parent แล้วลบ children ด้วย
// ใช้เมื่อ children ไม่มีความหมายโดยไม่มี parent
@Relationship(deleteRule: .cascade, inverse: \OrderItem.order)
var items: [OrderItem] = []

// .nullify - ลบ parent แล้ว set children เป็น nil
// ใช้เมื่อ children ยังมีความหมายโดยไม่มี parent
@Relationship(deleteRule: .nullify, inverse: \Post.author)
var posts: [Post] = []

// .deny - ห้ามลบถ้ายังมี children
// ใช้เมื่อต้องการ strict referential integrity
@Relationship(deleteRule: .deny, inverse: \BankAccount.owner)
var accounts: [BankAccount] = []

// .noAction - ไม่ทำอะไร (อันตราย - อาจเกิด orphan records)
// หลีกเลี่ยงถ้าไม่มีเหตุผลพิเศษ
```

---

## 8. Migrations

### 8.1 VersionedSchema Protocol

```swift
import SwiftData

// Version 1: Schema ดั้งเดิม
enum SchemaV1: VersionedSchema {
    static var versionIdentifier = Schema.Version(1, 0, 0)
    
    static var models: [any PersistentModel.Type] {
        [TodoItem.self]
    }
    
    @Model
    final class TodoItem {
        var title: String
        var isCompleted: Bool
        var createdAt: Date
        
        init(title: String) {
            self.title = title
            self.isCompleted = false
            self.createdAt = Date()
        }
    }
}

// Version 2: เพิ่ม priority และ category
enum SchemaV2: VersionedSchema {
    static var versionIdentifier = Schema.Version(2, 0, 0)
    
    static var models: [any PersistentModel.Type] {
        [TodoItem.self, Category.self]
    }
    
    @Model
    final class TodoItem {
        var title: String
        var isCompleted: Bool
        var createdAt: Date
        var priority: Int          // ใหม่!
        var category: Category?    // ใหม่!
        
        init(title: String, priority: Int = 0) {
            self.title = title
            self.isCompleted = false
            self.createdAt = Date()
            self.priority = priority
        }
    }
    
    @Model
    final class Category {
        var name: String
        var colorHex: String
        
        @Relationship(deleteRule: .cascade, inverse: \TodoItem.category)
        var items: [TodoItem] = []
        
        init(name: String, colorHex: String = "#007AFF") {
            self.name = name
            self.colorHex = colorHex
        }
    }
}
```

### 8.2 SchemaMigrationPlan

```swift
import SwiftData

// Migration plan
enum TodoMigrationPlan: SchemaMigrationPlan {
    static var schemas: [any VersionedSchema.Type] {
        [SchemaV1.self, SchemaV2.self]
    }
    
    static var stages: [MigrationStage] {
        [migrateV1toV2]
    }
    
    // Migration stage จาก V1 ไป V2
    static let migrateV1toV2 = MigrationStage.custom(
        fromVersion: SchemaV1.self,
        toVersion: SchemaV2.self,
        willMigrate: { context in
            // ทำงานก่อน migration
            print("Starting migration from V1 to V2")
        },
        didMigrate: { context in
            // ทำงานหลัง migration
            // ตั้งค่า default priority สำหรับ existing records
            let todos = try context.fetch(FetchDescriptor<SchemaV2.TodoItem>())
            for todo in todos {
                if todo.priority == 0 {
                    // Set default priority based on title keywords
                    if todo.title.lowercased().contains("urgent") {
                        todo.priority = 3
                    } else {
                        todo.priority = 1
                    }
                }
            }
            try context.save()
            print("Migration V1 to V2 completed")
        }
    )
}
```

### 8.3 MigrationStage

```swift
import SwiftData

// Lightweight migration - สำหรับ changes ที่ไม่ต้องการ data transformation
// เช่น เพิ่ม optional property, rename property
static let lightweightMigration = MigrationStage.lightweight(
    fromVersion: SchemaV1.self,
    toVersion: SchemaV2.self
)

// Custom migration - สำหรับ data transformation
static let customMigration = MigrationStage.custom(
    fromVersion: SchemaV2.self,
    toVersion: SchemaV3.self,
    willMigrate: nil,
    didMigrate: { context in
        // Transform data after migration
        let items = try context.fetch(FetchDescriptor<SchemaV3.TodoItem>())
        for item in items {
            // Custom data transformation
        }
        try context.save()
    }
)

// ใช้ migration plan ใน ModelContainer
let container = try ModelContainer(
    for: SchemaV2.TodoItem.self, SchemaV2.Category.self,
    migrationPlan: TodoMigrationPlan.self,
    configurations: ModelConfiguration(isStoredInMemoryOnly: false)
)
```

### 8.4 Adding Properties และ Relationships

```swift
import SwiftData

// เพิ่ม optional property - Lightweight migration (ไม่ต้องทำ custom)
enum SchemaV3: VersionedSchema {
    static var versionIdentifier = Schema.Version(3, 0, 0)
    static var models: [any PersistentModel.Type] { [TodoItem.self] }
    
    @Model
    final class TodoItem {
        var title: String
        var isCompleted: Bool
        var createdAt: Date
        var priority: Int
        var notes: String?     // เพิ่ม optional property - OK สำหรับ lightweight
        var dueDate: Date?     // เพิ่ม optional Date - OK
        var tags: [String] = [] // เพิ่ม array - OK
        
        init(title: String) {
            self.title = title
            self.isCompleted = false
            self.createdAt = Date()
            self.priority = 0
        }
    }
}

// Migration plan ที่รองรับ V1 → V2 → V3
enum FullMigrationPlan: SchemaMigrationPlan {
    static var schemas: [any VersionedSchema.Type] {
        [SchemaV1.self, SchemaV2.self, SchemaV3.self]
    }
    
    static var stages: [MigrationStage] {
        [
            migrateV1toV2,
            // V2 to V3 เป็น lightweight เพราะแค่เพิ่ม optional properties
            MigrationStage.lightweight(
                fromVersion: SchemaV2.self,
                toVersion: SchemaV3.self
            )
        ]
    }
    
    static let migrateV1toV2 = MigrationStage.custom(
        fromVersion: SchemaV1.self,
        toVersion: SchemaV2.self,
        willMigrate: nil,
        didMigrate: { context in
            try context.save()
        }
    )
}
```

---

## 9. SwiftData + iCloud

### 9.1 CloudKit Sync กับ SwiftData

```swift
import SwiftData

// Enable CloudKit sync
let config = ModelConfiguration(
    schema: Schema([TodoItem.self, Category.self]),
    cloudKitDatabase: .automatic  // ใช้ default CloudKit container
)

// หรือ specify container
let config2 = ModelConfiguration(
    schema: Schema([TodoItem.self]),
    cloudKitDatabase: .private("iCloud.com.yourcompany.yourapp")
)

let container = try ModelContainer(
    for: TodoItem.self,
    configurations: config
)

// Requirements สำหรับ CloudKit:
// 1. Enable iCloud capability ใน Xcode project
// 2. Add CloudKit entitlement
// 3. Create CloudKit container ใน Apple Developer portal
```

### 9.2 Required Schema Constraints สำหรับ CloudKit

```swift
import SwiftData

// CloudKit ต้องการ constraints เหล่านี้:

@Model
final class CloudSyncedTodo {
    // 1. ต้องไม่มี required (non-optional) relationships
    // ❌ var category: Category  (ไม่ได้)
    // ✅ var category: Category?  (ได้)
    var category: Category?
    
    // 2. Properties ต้องมี default values หรือเป็น optional
    var title: String = ""  // ✅ has default
    var notes: String?      // ✅ optional
    
    // 3. @Attribute(.unique) สำหรับ primary key
    @Attribute(.unique)
    var id: UUID = UUID()
    
    // 4. ไม่รองรับ .deny delete rule
    // ❌ @Relationship(deleteRule: .deny, ...)
    // ✅ @Relationship(deleteRule: .nullify, ...)
    
    init() {}
}
```

### 9.3 Sync Conflict Resolution

```swift
import SwiftData

// SwiftData + CloudKit จัดการ conflicts อัตโนมัติ
// โดยใช้ "last write wins" strategy

// สำหรับ custom conflict handling:
class ConflictAwareContext {
    private let context: ModelContext
    
    init(context: ModelContext) {
        self.context = context
    }
    
    func saveWithConflictResolution() throws {
        do {
            try context.save()
        } catch let error as NSError {
            // Handle merge conflicts
            if error.domain == NSCocoaErrorDomain &&
               error.code == NSManagedObjectMergeError {
                // Refresh conflicted objects
                context.refreshAllRegisteredModels()
                try context.save()
            } else {
                throw error
            }
        }
    }
}
```

---

## 10. SwiftData + SwiftUI

### 10.1 @Query ใน Lists

```swift
import SwiftUI
import SwiftData

struct TodoListView: View {
    @Query(
        filter: #Predicate<TodoItem> { !$0.isCompleted },
        sort: [
            SortDescriptor(\TodoItem.priority, order: .reverse),
            SortDescriptor(\TodoItem.createdAt)
        ]
    )
    var activeTodos: [TodoItem]
    
    @Environment(\.modelContext) private var context
    
    var body: some View {
        NavigationStack {
            List {
                ForEach(activeTodos) { todo in
                    TodoRow(todo: todo)
                }
                .onDelete(perform: deleteTodos)
            }
            .navigationTitle("Active Todos")
            .toolbar {
                ToolbarItem(placement: .primaryAction) {
                    Button(action: addTodo) {
                        Image(systemName: "plus")
                    }
                }
            }
        }
    }
    
    private func addTodo() {
        let newTodo = TodoItem(title: "New Todo")
        context.insert(newTodo)
    }
    
    private func deleteTodos(at offsets: IndexSet) {
        for offset in offsets {
            context.delete(activeTodos[offset])
        }
    }
}

struct TodoRow: View {
    @Bindable var todo: TodoItem
    
    var body: some View {
        HStack {
            Button {
                todo.isCompleted.toggle()
            } label: {
                Image(systemName: todo.isCompleted ? "checkmark.circle.fill" : "circle")
                    .foregroundColor(todo.isCompleted ? .green : .gray)
            }
            .buttonStyle(.plain)
            
            VStack(alignment: .leading) {
                Text(todo.title)
                    .strikethrough(todo.isCompleted)
                if let dueDate = todo.dueDate {
                    Text(dueDate, style: .date)
                        .font(.caption)
                        .foregroundColor(.secondary)
                }
            }
            
            Spacer()
            
            PriorityBadge(priority: todo.priority)
        }
    }
}
```

### 10.2 Creating และ Editing ใน Forms

```swift
import SwiftUI
import SwiftData

struct CreateTodoView: View {
    @Environment(\.modelContext) private var context
    @Environment(\.dismiss) private var dismiss
    
    @State private var title = ""
    @State private var priority = 1
    @State private var dueDate = Date()
    @State private var hasDueDate = false
    @State private var notes = ""
    
    var body: some View {
        NavigationStack {
            Form {
                Section("Details") {
                    TextField("Title", text: $title)
                    
                    Picker("Priority", selection: $priority) {
                        Text("Low").tag(1)
                        Text("Medium").tag(2)
                        Text("High").tag(3)
                    }
                }
                
                Section("Due Date") {
                    Toggle("Has Due Date", isOn: $hasDueDate)
                    if hasDueDate {
                        DatePicker(
                            "Due Date",
                            selection: $dueDate,
                            displayedComponents: .date
                        )
                    }
                }
                
                Section("Notes") {
                    TextEditor(text: $notes)
                        .frame(minHeight: 100)
                }
            }
            .navigationTitle("New Todo")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .cancellationAction) {
                    Button("Cancel") { dismiss() }
                }
                ToolbarItem(placement: .confirmationAction) {
                    Button("Save") {
                        saveTodo()
                        dismiss()
                    }
                    .disabled(title.isEmpty)
                }
            }
        }
    }
    
    private func saveTodo() {
        let todo = TodoItem(title: title, priority: priority)
        todo.notes = notes.isEmpty ? nil : notes
        todo.dueDate = hasDueDate ? dueDate : nil
        context.insert(todo)
    }
}

// Edit existing todo
struct EditTodoView: View {
    @Bindable var todo: TodoItem  // @Bindable สำหรับ @Model objects
    @Environment(\.dismiss) private var dismiss
    
    var body: some View {
        NavigationStack {
            Form {
                Section("Details") {
                    TextField("Title", text: $todo.title)
                    
                    Picker("Priority", selection: $todo.priority) {
                        Text("Low").tag(1)
                        Text("Medium").tag(2)
                        Text("High").tag(3)
                    }
                    
                    Toggle("Completed", isOn: $todo.isCompleted)
                }
                
                Section("Notes") {
                    TextField("Notes", text: Binding(
                        get: { todo.notes ?? "" },
                        set: { todo.notes = $0.isEmpty ? nil : $0 }
                    ))
                }
            }
            .navigationTitle("Edit Todo")
            .toolbar {
                ToolbarItem(placement: .confirmationAction) {
                    Button("Done") { dismiss() }
                }
            }
        }
    }
}
```

### 10.3 Real-time Updates ใน Views

```swift
import SwiftUI
import SwiftData

// @Query updates view อัตโนมัติเมื่อ data เปลี่ยน
struct StatsView: View {
    @Query var allTodos: [TodoItem]
    @Query(filter: #Predicate<TodoItem> { $0.isCompleted }) var completedTodos: [TodoItem]
    @Query(filter: #Predicate<TodoItem> { !$0.isCompleted }) var activeTodos: [TodoItem]
    
    var completionRate: Double {
        guard !allTodos.isEmpty else { return 0 }
        return Double(completedTodos.count) / Double(allTodos.count) * 100
    }
    
    var body: some View {
        VStack(spacing: 16) {
            StatCard(title: "Total", value: allTodos.count, color: .blue)
            StatCard(title: "Active", value: activeTodos.count, color: .orange)
            StatCard(title: "Completed", value: completedTodos.count, color: .green)
            
            ProgressView(value: completionRate, total: 100) {
                Text("Completion Rate: \(Int(completionRate))%")
            }
        }
        .padding()
    }
}

struct StatCard: View {
    let title: String
    let value: Int
    let color: Color
    
    var body: some View {
        HStack {
            Text(title)
                .font(.headline)
            Spacer()
            Text("\(value)")
                .font(.title2.bold())
                .foregroundColor(color)
        }
        .padding()
        .background(color.opacity(0.1))
        .cornerRadius(10)
    }
}
```

### 10.4 Preview กับ In-Memory Store

```swift
import SwiftUI
import SwiftData

// Preview helper
extension ModelContainer {
    static var preview: ModelContainer {
        let config = ModelConfiguration(isStoredInMemoryOnly: true)
        let container = try! ModelContainer(
            for: TodoItem.self, Category.self,
            configurations: config
        )
        
        // Seed preview data
        let context = container.mainContext
        
        let work = Category(name: "Work", colorHex: "#FF3B30")
        let personal = Category(name: "Personal", colorHex: "#34C759")
        context.insert(work)
        context.insert(personal)
        
        let todos: [(String, Int, Category?, Bool)] = [
            ("Team standup meeting", 3, work, false),
            ("Review PR #123", 2, work, true),
            ("Buy groceries", 1, personal, false),
            ("Call dentist", 2, personal, false),
            ("Deploy to production", 3, work, false),
        ]
        
        for (title, priority, category, completed) in todos {
            let todo = TodoItem(title: title, priority: priority)
            todo.isCompleted = completed
            todo.category = category
            context.insert(todo)
        }
        
        return container
    }
}

// ใช้ใน Preview
#Preview {
    TodoListView()
        .modelContainer(.preview)
}

#Preview("Stats") {
    StatsView()
        .modelContainer(.preview)
}
```

---

## 11. SwiftData + Swift Concurrency

### 11.1 Background Fetch กับ ModelActor

```swift
import SwiftData

// ModelActor - actor ที่มี dedicated ModelContext
@ModelActor
actor DataSyncActor {
    
    func syncData(from remoteItems: [RemoteItem]) async throws {
        for item in remoteItems {
            // ทำงานบน background thread อย่างปลอดภัย
            if let existing = try fetchExisting(id: item.id) {
                existing.title = item.title
                existing.updatedAt = item.updatedAt
            } else {
                let newItem = TodoItem(title: item.title)
                newItem.remoteId = item.id
                modelContext.insert(newItem)
            }
        }
        try modelContext.save()
    }
    
    private func fetchExisting(id: String) throws -> TodoItem? {
        let predicate = #Predicate<TodoItem> { $0.remoteId == id }
        let descriptor = FetchDescriptor<TodoItem>(predicate: predicate)
        descriptor.fetchLimit = 1
        return try modelContext.fetch(descriptor).first
    }
    
    func fetchStatistics() async throws -> Stats {
        let allCount = try modelContext.fetchCount(FetchDescriptor<TodoItem>())
        let completedCount = try modelContext.fetchCount(
            FetchDescriptor<TodoItem>(
                predicate: #Predicate { $0.isCompleted }
            )
        )
        return Stats(total: allCount, completed: completedCount)
    }
}

struct Stats {
    let total: Int
    let completed: Int
    var completionRate: Double { Double(completed) / Double(total) }
}

struct RemoteItem: Codable {
    let id: String
    let title: String
    let updatedAt: Date
}
```

### 11.2 @ModelActor Macro

```swift
import SwiftData

// @ModelActor สร้าง actor ที่:
// 1. มี modelContext ของตัวเอง
// 2. Thread-safe
// 3. Isolated จาก main context

// Usage
@ModelActor
actor BackgroundProcessor {
    
    // modelContext มีให้ใช้โดยอัตโนมัติ
    
    func processLargeDataset() async throws {
        let descriptor = FetchDescriptor<TodoItem>()
        let items = try modelContext.fetch(descriptor)
        
        for item in items {
            // CPU-intensive processing
            await processItem(item)
        }
        
        try modelContext.save()
    }
    
    private func processItem(_ item: TodoItem) async {
        // Simulate processing
        item.processedAt = Date()
    }
    
    func importCSV(_ csvString: String) async throws {
        let lines = csvString.components(separatedBy: "\n")
        
        for line in lines.dropFirst() { // Skip header
            let columns = line.components(separatedBy: ",")
            guard columns.count >= 2 else { continue }
            
            let todo = TodoItem(title: columns[0].trimmingCharacters(in: .whitespaces))
            if let priority = Int(columns[1].trimmingCharacters(in: .whitespaces)) {
                todo.priority = priority
            }
            modelContext.insert(todo)
        }
        
        try modelContext.save()
    }
}

// การใช้งาน
struct ViewModel: ObservableObject {
    private let container: ModelContainer
    private lazy var processor = BackgroundProcessor(modelContainer: container)
    
    init(container: ModelContainer) {
        self.container = container
    }
    
    func syncInBackground() async {
        do {
            try await processor.processLargeDataset()
            print("Processing complete!")
        } catch {
            print("Error: \(error)")
        }
    }
}
```

### 11.3 Thread-Safe Model Access

```swift
import SwiftData

// ❌ ไม่ถูกต้อง - ส่ง model object ข้าม actors
@ModelActor
actor UnsafeActor {
    func badExample(todo: TodoItem) async {
        // ❌ ไม่ควรส่ง model object จาก main context
        print(todo.title)
    }
}

// ✅ ถูกต้อง - ส่ง PersistentIdentifier แทน
@ModelActor
actor SafeActor {
    
    func goodExample(todoID: PersistentIdentifier) async throws {
        // ✅ Fetch model ใน actor's own context
        guard let todo = modelContext.model(for: todoID) as? TodoItem else {
            return
        }
        print(todo.title)
    }
    
    func updateTodo(id: PersistentIdentifier, newTitle: String) async throws {
        guard let todo = modelContext.model(for: id) as? TodoItem else {
            throw DataError.notFound
        }
        todo.title = newTitle
        try modelContext.save()
    }
}

enum DataError: Error {
    case notFound
    case saveFailed
}

// Main thread usage
class TodoService {
    private let container: ModelContainer
    private lazy var safeActor = SafeActor(modelContainer: container)
    
    init(container: ModelContainer) {
        self.container = container
    }
    
    @MainActor
    func updateTitle(for todo: TodoItem, newTitle: String) async throws {
        // ส่ง identifier แทน object
        let id = todo.persistentModelID
        try await safeActor.updateTodo(id: id, newTitle: newTitle)
    }
}
```

---

## 12. Testing SwiftData

### 12.1 In-Memory ModelContainer สำหรับ Tests

```swift
import XCTest
import SwiftData
@testable import MyApp

class SwiftDataTests: XCTestCase {
    
    var container: ModelContainer!
    var context: ModelContext!
    
    override func setUpWithError() throws {
        // สร้าง in-memory container สำหรับ test isolation
        let config = ModelConfiguration(isStoredInMemoryOnly: true)
        container = try ModelContainer(
            for: TodoItem.self, Category.self,
            configurations: config
        )
        context = container.mainContext
    }
    
    override func tearDownWithError() throws {
        container = nil
        context = nil
    }
    
    func testCreateTodo() throws {
        // Given
        let title = "Test Todo"
        
        // When
        let todo = TodoItem(title: title)
        context.insert(todo)
        try context.save()
        
        // Then
        let descriptor = FetchDescriptor<TodoItem>()
        let todos = try context.fetch(descriptor)
        XCTAssertEqual(todos.count, 1)
        XCTAssertEqual(todos.first?.title, title)
        XCTAssertFalse(todos.first?.isCompleted ?? true)
    }
    
    func testDeleteTodo() throws {
        // Given
        let todo = TodoItem(title: "To Delete")
        context.insert(todo)
        try context.save()
        
        // When
        context.delete(todo)
        try context.save()
        
        // Then
        let todos = try context.fetch(FetchDescriptor<TodoItem>())
        XCTAssertTrue(todos.isEmpty)
    }
    
    func testFetchWithPredicate() throws {
        // Given
        let high = TodoItem(title: "High", priority: 3)
        let low = TodoItem(title: "Low", priority: 1)
        let medium = TodoItem(title: "Medium", priority: 2)
        
        [high, low, medium].forEach { context.insert($0) }
        try context.save()
        
        // When
        let descriptor = FetchDescriptor<TodoItem>(
            predicate: #Predicate { $0.priority >= 2 }
        )
        let results = try context.fetch(descriptor)
        
        // Then
        XCTAssertEqual(results.count, 2)
        XCTAssertTrue(results.contains { $0.title == "High" })
        XCTAssertTrue(results.contains { $0.title == "Medium" })
    }
    
    func testRelationship() throws {
        // Given
        let category = Category(name: "Work")
        let todo = TodoItem(title: "Work Task")
        todo.category = category
        context.insert(category)
        context.insert(todo)
        try context.save()
        
        // When
        let todos = try context.fetch(FetchDescriptor<TodoItem>())
        
        // Then
        XCTAssertEqual(todos.first?.category?.name, "Work")
        XCTAssertEqual(category.items.count, 1)
    }
}
```

### 12.2 Test Isolation Strategies

```swift
import XCTest
import SwiftData

// Strategy 1: Fresh container per test (default approach)
class IsolatedTests: XCTestCase {
    
    var container: ModelContainer!
    
    override func setUp() {
        super.setUp()
        container = try! ModelContainer(
            for: TodoItem.self,
            configurations: ModelConfiguration(isStoredInMemoryOnly: true)
        )
    }
    
    override func tearDown() {
        container = nil
        super.tearDown()
    }
}

// Strategy 2: Rollback after each test
class RollbackTests: XCTestCase {
    
    var container: ModelContainer!
    var context: ModelContext!
    
    override func setUp() {
        super.setUp()
        container = try! ModelContainer(
            for: TodoItem.self,
            configurations: ModelConfiguration(isStoredInMemoryOnly: true)
        )
        context = container.mainContext
        context.autosaveEnabled = false  // Manual save เท่านั้น
    }
    
    override func tearDown() {
        context.rollback()  // Rollback ทุก unsaved changes
        container = nil
        context = nil
        super.tearDown()
    }
    
    func testWithRollback() throws {
        let todo = TodoItem(title: "Test")
        context.insert(todo)
        // ไม่ save - จะ rollback ใน tearDown
        
        let count = try context.fetchCount(FetchDescriptor<TodoItem>())
        XCTAssertEqual(count, 1)
        // หลัง tearDown, container ใหม่จะว่างเปล่า
    }
}
```

### 12.3 Mock Data Setup

```swift
import SwiftData
import Foundation

// Shared test fixtures
struct TestFixtures {
    
    static func seedBasicData(in context: ModelContext) throws {
        let categories = [
            Category(name: "Work", colorHex: "#FF3B30"),
            Category(name: "Personal", colorHex: "#34C759"),
            Category(name: "Shopping", colorHex: "#007AFF"),
        ]
        categories.forEach { context.insert($0) }
        
        let todos = [
            (title: "Team standup", priority: 2, category: categories[0], completed: false),
            (title: "Write tests", priority: 3, category: categories[0], completed: true),
            (title: "Call mom", priority: 2, category: categories[1], completed: false),
            (title: "Buy milk", priority: 1, category: categories[2], completed: false),
            (title: "Deploy app", priority: 3, category: categories[0], completed: false),
        ]
        
        for item in todos {
            let todo = TodoItem(title: item.title, priority: item.priority)
            todo.isCompleted = item.completed
            todo.category = item.category
            context.insert(todo)
        }
        
        try context.save()
    }
    
    static func createTodo(
        title: String = "Test Todo",
        priority: Int = 1,
        completed: Bool = false,
        in context: ModelContext
    ) throws -> TodoItem {
        let todo = TodoItem(title: title, priority: priority)
        todo.isCompleted = completed
        context.insert(todo)
        try context.save()
        return todo
    }
}

// ใช้ fixtures ใน tests
class TodoServiceTests: XCTestCase {
    
    var container: ModelContainer!
    var context: ModelContext!
    var service: TodoService!
    
    override func setUp() {
        super.setUp()
        container = try! ModelContainer(
            for: TodoItem.self, Category.self,
            configurations: ModelConfiguration(isStoredInMemoryOnly: true)
        )
        context = container.mainContext
        service = TodoService(context: context)
        
        // Seed test data
        try! TestFixtures.seedBasicData(in: context)
    }
    
    func testCompletionRate() throws {
        let stats = try service.getStats()
        XCTAssertEqual(stats.total, 5)
        XCTAssertEqual(stats.completed, 1)
        XCTAssertEqual(stats.completionRate, 0.2, accuracy: 0.001)
    }
}
```

---

## 13. Advanced Features

### 13.1 Compound Unique Constraints

```swift
import SwiftData

// Composite unique constraint
@Model
final class UserRole {
    // ไม่สามารถมี user+role combination ซ้ำกัน
    var userId: String
    var roleId: String
    var assignedAt: Date
    
    init(userId: String, roleId: String) {
        self.userId = userId
        self.roleId = roleId
        self.assignedAt = Date()
    }
}

// ใช้ Schema กำหนด unique constraint แบบ compound
// (ใน SwiftData 1.0 ยังต้องใช้ Core Data approach สำหรับ compound unique)
// ใน Schema definition:
extension UserRole {
    static var fetchDescriptorForUniqueness: FetchDescriptor<UserRole> {
        FetchDescriptor<UserRole>()
    }
}
```

### 13.2 Index Optimization

```swift
import SwiftData

@Model
final class Article {
    // Single index สำหรับ fast lookup
    @Attribute(.unique)
    var slug: String
    
    // Spotlight index
    @Attribute(.spotlight)
    var title: String
    
    var content: String
    var publishedAt: Date
    var viewCount: Int
    var category: String
    
    init(slug: String, title: String, content: String, category: String) {
        self.slug = slug
        self.title = title
        self.content = content
        self.publishedAt = Date()
        self.viewCount = 0
        self.category = category
    }
}

// Performance tips:
// 1. ใช้ @Attribute(.unique) สำหรับ properties ที่ fetch บ่อยด้วย predicate
// 2. ใช้ fetchLimit เสมอเมื่อไม่ต้องการทุก records
// 3. ใช้ FetchDescriptor.includePendingChanges = false เมื่อไม่ต้องการ unsaved changes
```

### 13.3 History Tracking

```swift
import SwiftData
import CoreData

// SwiftData ใช้ Core Data history tracking ภายใต้
// สามารถเปิดใช้ผ่าน NSPersistentHistoryTrackingKey

extension ModelContainer {
    static func withHistoryTracking(for types: [any PersistentModel.Type]) throws -> ModelContainer {
        let config = ModelConfiguration(isStoredInMemoryOnly: false)
        
        // Access underlying NSPersistentStoreDescription
        // Note: ใน SwiftData 1.0 history tracking ยังไม่ fully exposed
        // ใช้ผ่าน Core Data bridge ถ้าต้องการ
        
        return try ModelContainer(for: Schema(types), configurations: config)
    }
}
```

---

## 14. Performance

### 14.1 Lazy Loading Relationships

```swift
import SwiftData

@Model
final class Post {
    var title: String
    var content: String
    
    // Relationships ใน SwiftData เป็น lazy by default
    // จะ fetch เมื่อ access เท่านั้น
    @Relationship(deleteRule: .cascade, inverse: \Comment.post)
    var comments: [Comment] = []
    
    init(title: String, content: String) {
        self.title = title
        self.content = content
    }
}

// ❌ N+1 query problem
func badFetch(context: ModelContext) throws {
    let posts = try context.fetch(FetchDescriptor<Post>())
    for post in posts {
        // ทุก access ไปยัง comments จะ trigger fetch แยก
        print(post.comments.count)  // N queries!
    }
}

// ✅ Prefetch relationships
func goodFetch(context: ModelContext) throws {
    var descriptor = FetchDescriptor<Post>()
    // SwiftData จัดการ lazy loading อัตโนมัติ
    // ใช้ fetchLimit เพื่อ limit จำนวน records
    descriptor.fetchLimit = 50
    
    let posts = try context.fetch(descriptor)
    // Access comments จะ batch fetch เมื่อจำเป็น
}
```

### 14.2 Batch Operations

```swift
import SwiftData

// Batch delete ด้วย predicate (ไม่ต้อง fetch ก่อน)
func batchDeleteOldTodos(context: ModelContext, olderThan days: Int) throws {
    let cutoffDate = Calendar.current.date(
        byAdding: .day,
        value: -days,
        to: Date()
    )!
    
    try context.delete(
        model: TodoItem.self,
        where: #Predicate { item in
            item.isCompleted && item.createdAt < cutoffDate
        }
    )
}

// Batch update (ผ่าน fetch + update)
func markAllCompleted(context: ModelContext, in category: Category) throws {
    let categoryId = category.persistentModelID
    
    let descriptor = FetchDescriptor<TodoItem>(
        predicate: #Predicate { item in
            item.category?.persistentModelID == categoryId && !item.isCompleted
        }
    )
    
    let items = try context.fetch(descriptor)
    for item in items {
        item.isCompleted = true
    }
    
    try context.save()
}

// ใช้ background context สำหรับ large batch operations
func batchImport(items: [ImportItem], container: ModelContainer) async throws {
    let actor = BackgroundProcessor(modelContainer: container)
    try await actor.importItems(items)
}
```

### 14.3 Profiling กับ Instruments

```swift
// Tips สำหรับ profiling SwiftData:

// 1. ใช้ Core Data Instruments template
// Xcode > Profile > Core Data template

// 2. Enable SQLite debug logging
// Add launch argument: -com.apple.CoreData.SQLDebug 1

// 3. Measure fetch performance
func measureFetchPerformance(context: ModelContext) throws {
    let start = Date()
    
    let descriptor = FetchDescriptor<TodoItem>(
        sortBy: [SortDescriptor(\.createdAt, order: .reverse)]
    )
    let results = try context.fetch(descriptor)
    
    let elapsed = Date().timeIntervalSince(start)
    print("Fetched \(results.count) items in \(String(format: "%.3f", elapsed))s")
}

// 4. ใช้ fetchLimit เสมอสำหรับ large datasets
func efficientFetch(context: ModelContext, limit: Int = 50) throws -> [TodoItem] {
    var descriptor = FetchDescriptor<TodoItem>(
        sortBy: [SortDescriptor(\.createdAt, order: .reverse)]
    )
    descriptor.fetchLimit = limit
    return try context.fetch(descriptor)
}
```

---

## 15. Complete App: Task Manager กับ SwiftData + CloudKit Sync

### 15.1 App Architecture

```swift
// Models
import SwiftData
import Foundation

@Model
final class Task {
    @Attribute(.unique)
    var id: UUID
    
    var title: String
    var notes: String?
    var isCompleted: Bool
    var priority: TaskPriority
    var createdAt: Date
    var completedAt: Date?
    var dueDate: Date?
    
    @Relationship(deleteRule: .nullify, inverse: \TaskList.tasks)
    var list: TaskList?
    
    @Relationship(deleteRule: .nullify, inverse: \Label.tasks)
    var labels: [Label] = []
    
    init(title: String, priority: TaskPriority = .medium) {
        self.id = UUID()
        self.title = title
        self.priority = priority
        self.isCompleted = false
        self.createdAt = Date()
    }
    
    func complete() {
        isCompleted = true
        completedAt = Date()
    }
}

enum TaskPriority: Int, Codable, CaseIterable {
    case low = 1
    case medium = 2
    case high = 3
    case urgent = 4
    
    var label: String {
        switch self {
        case .low: return "Low"
        case .medium: return "Medium"
        case .high: return "High"
        case .urgent: return "Urgent"
        }
    }
    
    var color: String {
        switch self {
        case .low: return "#8E8E93"
        case .medium: return "#007AFF"
        case .high: return "#FF9500"
        case .urgent: return "#FF3B30"
        }
    }
}

@Model
final class TaskList {
    @Attribute(.unique)
    var id: UUID
    
    var name: String
    var colorHex: String
    var iconName: String
    var createdAt: Date
    
    @Relationship(deleteRule: .cascade, inverse: \Task.list)
    var tasks: [Task] = []
    
    var activeTasks: [Task] { tasks.filter { !$0.isCompleted } }
    var completedTasks: [Task] { tasks.filter { $0.isCompleted } }
    
    init(name: String, colorHex: String = "#007AFF", iconName: String = "list.bullet") {
        self.id = UUID()
        self.name = name
        self.colorHex = colorHex
        self.iconName = iconName
        self.createdAt = Date()
    }
}

@Model
final class Label {
    @Attribute(.unique)
    var id: UUID
    
    var name: String
    var colorHex: String
    
    @Relationship(inverse: \Task.labels)
    var tasks: [Task] = []
    
    init(name: String, colorHex: String = "#007AFF") {
        self.id = UUID()
        self.name = name
        self.colorHex = colorHex
    }
}
```

### 15.2 App Setup

```swift
import SwiftUI
import SwiftData

@main
struct TaskManagerApp: App {
    let container: ModelContainer
    
    init() {
        do {
            let schema = Schema([Task.self, TaskList.self, Label.self])
            let config = ModelConfiguration(
                schema: schema,
                isStoredInMemoryOnly: false,
                cloudKitDatabase: .automatic
            )
            container = try ModelContainer(for: schema, configurations: config)
            
            // Seed initial data if needed
            seedInitialDataIfNeeded()
        } catch {
            fatalError("Failed to initialize ModelContainer: \(error)")
        }
    }
    
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(container)
    }
    
    private func seedInitialDataIfNeeded() {
        let context = container.mainContext
        let count = (try? context.fetchCount(FetchDescriptor<TaskList>())) ?? 0
        
        guard count == 0 else { return }
        
        let defaultLists = [
            TaskList(name: "Inbox", colorHex: "#007AFF", iconName: "tray"),
            TaskList(name: "Work", colorHex: "#FF3B30", iconName: "briefcase"),
            TaskList(name: "Personal", colorHex: "#34C759", iconName: "person"),
        ]
        
        defaultLists.forEach { context.insert($0) }
        try? context.save()
    }
}
```

### 15.3 Main Views

```swift
import SwiftUI
import SwiftData

struct ContentView: View {
    @Query(sort: \TaskList.createdAt) var lists: [TaskList]
    @Environment(\.modelContext) private var context
    @State private var selectedList: TaskList?
    @State private var showCreateList = false
    
    var body: some View {
        NavigationSplitView {
            // Sidebar - Lists
            List(selection: $selectedList) {
                ForEach(lists) { list in
                    NavigationLink(value: list) {
                        TaskListRow(list: list)
                    }
                }
                .onDelete(perform: deleteLists)
            }
            .navigationTitle("My Lists")
            .toolbar {
                ToolbarItem(placement: .primaryAction) {
                    Button {
                        showCreateList = true
                    } label: {
                        Image(systemName: "plus")
                    }
                }
            }
        } detail: {
            if let list = selectedList {
                TaskListDetailView(list: list)
            } else {
                AllTasksView()
            }
        }
        .sheet(isPresented: $showCreateList) {
            CreateListView()
        }
    }
    
    private func deleteLists(at offsets: IndexSet) {
        for offset in offsets {
            context.delete(lists[offset])
        }
    }
}

struct TaskListRow: View {
    let list: TaskList
    
    var body: some View {
        HStack {
            Image(systemName: list.iconName)
                .foregroundColor(Color(hex: list.colorHex))
                .frame(width: 24)
            
            Text(list.name)
            
            Spacer()
            
            if !list.activeTasks.isEmpty {
                Text("\(list.activeTasks.count)")
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
        }
    }
}

struct TaskListDetailView: View {
    let list: TaskList
    @Query var tasks: [Task]
    @Environment(\.modelContext) private var context
    @State private var showCreateTask = false
    @State private var showCompleted = false
    
    init(list: TaskList) {
        self.list = list
        let listId = list.persistentModelID
        _tasks = Query(
            filter: #Predicate<Task> { task in
                task.list?.persistentModelID == listId
            },
            sort: [
                SortDescriptor(\Task.priority, order: .reverse),
                SortDescriptor(\Task.createdAt)
            ]
        )
    }
    
    var activeTasks: [Task] { tasks.filter { !$0.isCompleted } }
    var completedTasks: [Task] { tasks.filter { $0.isCompleted } }
    
    var body: some View {
        List {
            Section {
                ForEach(activeTasks) { task in
                    TaskRow(task: task)
                }
                .onDelete { offsets in
                    deleteTasks(from: activeTasks, at: offsets)
                }
            }
            
            if showCompleted && !completedTasks.isEmpty {
                Section("Completed") {
                    ForEach(completedTasks) { task in
                        TaskRow(task: task)
                    }
                }
            }
        }
        .navigationTitle(list.name)
        .toolbar {
            ToolbarItem(placement: .primaryAction) {
                Button {
                    showCreateTask = true
                } label: {
                    Image(systemName: "plus")
                }
            }
            
            ToolbarItem(placement: .secondaryAction) {
                Toggle("Show Completed", isOn: $showCompleted)
            }
        }
        .sheet(isPresented: $showCreateTask) {
            CreateTaskView(defaultList: list)
        }
    }
    
    private func deleteTasks(from tasks: [Task], at offsets: IndexSet) {
        for offset in offsets {
            context.delete(tasks[offset])
        }
    }
}

struct TaskRow: View {
    @Bindable var task: Task
    
    var body: some View {
        HStack(spacing: 12) {
            // Priority indicator
            Circle()
                .fill(Color(hex: task.priority.color))
                .frame(width: 8, height: 8)
            
            // Completion toggle
            Button {
                withAnimation {
                    if task.isCompleted {
                        task.isCompleted = false
                        task.completedAt = nil
                    } else {
                        task.complete()
                    }
                }
            } label: {
                Image(systemName: task.isCompleted ? "checkmark.circle.fill" : "circle")
                    .foregroundColor(task.isCompleted ? .green : .gray)
                    .font(.title3)
            }
            .buttonStyle(.plain)
            
            VStack(alignment: .leading, spacing: 2) {
                Text(task.title)
                    .strikethrough(task.isCompleted, color: .gray)
                    .foregroundColor(task.isCompleted ? .secondary : .primary)
                
                if let dueDate = task.dueDate {
                    HStack(spacing: 4) {
                        Image(systemName: "calendar")
                        Text(dueDate, style: .date)
                    }
                    .font(.caption)
                    .foregroundColor(
                        dueDate < Date() && !task.isCompleted ? .red : .secondary
                    )
                }
                
                if !task.labels.isEmpty {
                    HStack {
                        ForEach(task.labels, id: \.id) { label in
                            Text(label.name)
                                .font(.caption2)
                                .padding(.horizontal, 6)
                                .padding(.vertical, 2)
                                .background(Color(hex: label.colorHex).opacity(0.2))
                                .cornerRadius(4)
                        }
                    }
                }
            }
        }
        .padding(.vertical, 4)
    }
}
```

### 15.4 All Tasks View

```swift
import SwiftUI
import SwiftData

struct AllTasksView: View {
    @Query(
        filter: #Predicate<Task> { !$0.isCompleted },
        sort: [
            SortDescriptor(\Task.priority, order: .reverse),
            SortDescriptor(\Task.createdAt)
        ]
    ) var activeTasks: [Task]
    
    @Query(
        filter: #Predicate<Task> { 
            $0.dueDate != nil && !$0.isCompleted
        },
        sort: \.dueDate
    ) var dueSoonTasks: [Task]
    
    var body: some View {
        List {
            if !dueSoonTasks.isEmpty {
                Section("Due Soon") {
                    ForEach(dueSoonTasks.prefix(5)) { task in
                        TaskRow(task: task)
                    }
                }
            }
            
            Section("All Active") {
                ForEach(activeTasks) { task in
                    TaskRow(task: task)
                }
            }
        }
        .navigationTitle("All Tasks")
    }
}
```

### 15.5 Create Task View

```swift
import SwiftUI
import SwiftData

struct CreateTaskView: View {
    @Environment(\.modelContext) private var context
    @Environment(\.dismiss) private var dismiss
    
    var defaultList: TaskList?
    
    @State private var title = ""
    @State private var notes = ""
    @State private var priority: TaskPriority = .medium
    @State private var selectedList: TaskList?
    @State private var hasDueDate = false
    @State private var dueDate = Date()
    
    @Query(sort: \TaskList.name) var lists: [TaskList]
    
    var body: some View {
        NavigationStack {
            Form {
                Section {
                    TextField("Task title", text: $title)
                    
                    Picker("Priority", selection: $priority) {
                        ForEach(TaskPriority.allCases, id: \.self) { p in
                            HStack {
                                Circle()
                                    .fill(Color(hex: p.color))
                                    .frame(width: 8, height: 8)
                                Text(p.label)
                            }
                            .tag(p)
                        }
                    }
                }
                
                Section {
                    Picker("List", selection: $selectedList) {
                        Text("None").tag(Optional<TaskList>.none)
                        ForEach(lists) { list in
                            Text(list.name).tag(Optional(list))
                        }
                    }
                }
                
                Section {
                    Toggle("Due Date", isOn: $hasDueDate)
                    if hasDueDate {
                        DatePicker(
                            "Date",
                            selection: $dueDate,
                            displayedComponents: .date
                        )
                    }
                }
                
                Section("Notes") {
                    TextEditor(text: $notes)
                        .frame(minHeight: 80)
                }
            }
            .navigationTitle("New Task")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .cancellationAction) {
                    Button("Cancel") { dismiss() }
                }
                ToolbarItem(placement: .confirmationAction) {
                    Button("Add") {
                        createTask()
                        dismiss()
                    }
                    .disabled(title.trimmingCharacters(in: .whitespacesAndNewlines).isEmpty)
                }
            }
            .onAppear {
                selectedList = defaultList
            }
        }
    }
    
    private func createTask() {
        let task = Task(
            title: title.trimmingCharacters(in: .whitespacesAndNewlines),
            priority: priority
        )
        task.notes = notes.isEmpty ? nil : notes
        task.list = selectedList
        task.dueDate = hasDueDate ? dueDate : nil
        context.insert(task)
    }
}
```

### 15.6 Color Helper

```swift
import SwiftUI

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
```

---

## 16. Exercises with Solutions

### Exercise 1: Note-Taking App Model

**โจทย์:** ออกแบบ SwiftData models สำหรับ note-taking app พร้อม folders, tags, และ attachments

**Solution:**
```swift
import SwiftData
import Foundation

@Model
final class Note {
    @Attribute(.unique)
    var id: UUID
    
    var title: String
    var content: String
    var createdAt: Date
    var modifiedAt: Date
    var isPinned: Bool
    var isFavorite: Bool
    
    @Attribute(.externalStorage)
    var thumbnailData: Data?
    
    var folder: Folder?
    
    @Relationship(deleteRule: .cascade, inverse: \Attachment.note)
    var attachments: [Attachment] = []
    
    @Relationship(inverse: \Tag.notes)
    var tags: [Tag] = []
    
    init(title: String, content: String = "") {
        self.id = UUID()
        self.title = title
        self.content = content
        self.createdAt = Date()
        self.modifiedAt = Date()
        self.isPinned = false
        self.isFavorite = false
    }
    
    func updateContent(_ newContent: String) {
        content = newContent
        modifiedAt = Date()
    }
}

@Model
final class Folder {
    @Attribute(.unique)
    var id: UUID
    var name: String
    var colorHex: String
    var createdAt: Date
    
    var parent: Folder?
    
    @Relationship(deleteRule: .nullify, inverse: \Folder.parent)
    var subfolders: [Folder] = []
    
    @Relationship(deleteRule: .nullify, inverse: \Note.folder)
    var notes: [Note] = []
    
    init(name: String, colorHex: String = "#007AFF") {
        self.id = UUID()
        self.name = name
        self.colorHex = colorHex
        self.createdAt = Date()
    }
}

@Model
final class Tag {
    @Attribute(.unique)
    var name: String
    var colorHex: String
    
    @Relationship(inverse: \Note.tags)
    var notes: [Note] = []
    
    init(name: String, colorHex: String = "#007AFF") {
        self.name = name
        self.colorHex = colorHex
    }
}

@Model
final class Attachment {
    @Attribute(.unique)
    var id: UUID
    var filename: String
    var mimeType: String
    var fileSize: Int64
    var createdAt: Date
    
    @Attribute(.externalStorage)
    var data: Data
    
    var note: Note?
    
    init(filename: String, mimeType: String, data: Data) {
        self.id = UUID()
        self.filename = filename
        self.mimeType = mimeType
        self.data = data
        self.fileSize = Int64(data.count)
        self.createdAt = Date()
    }
}
```

### Exercise 2: Search Implementation

**โจทย์:** สร้าง full-text search สำหรับ notes

```swift
import SwiftUI
import SwiftData

struct NoteSearchView: View {
    @State private var searchText = ""
    @State private var searchField: SearchField = .all
    
    enum SearchField: String, CaseIterable {
        case all = "All"
        case title = "Title"
        case content = "Content"
        case tag = "Tag"
    }
    
    var body: some View {
        VStack {
            Picker("Search in", selection: $searchField) {
                ForEach(SearchField.allCases, id: \.self) { field in
                    Text(field.rawValue).tag(field)
                }
            }
            .pickerStyle(.segmented)
            .padding(.horizontal)
            
            NoteSearchResults(
                searchText: searchText,
                searchField: searchField
            )
        }
        .searchable(text: $searchText, prompt: "Search notes...")
    }
}

struct NoteSearchResults: View {
    let searchText: String
    let searchField: NoteSearchView.SearchField
    
    @Query var notes: [Note]
    
    init(searchText: String, searchField: NoteSearchView.SearchField) {
        self.searchText = searchText
        self.searchField = searchField
        
        if searchText.isEmpty {
            _notes = Query(sort: \.modifiedAt, order: .reverse)
        } else {
            let predicate: Predicate<Note>
            
            switch searchField {
            case .all:
                predicate = #Predicate<Note> { note in
                    note.title.localizedStandardContains(searchText) ||
                    note.content.localizedStandardContains(searchText)
                }
            case .title:
                predicate = #Predicate<Note> { note in
                    note.title.localizedStandardContains(searchText)
                }
            case .content:
                predicate = #Predicate<Note> { note in
                    note.content.localizedStandardContains(searchText)
                }
            case .tag:
                predicate = #Predicate<Note> { note in
                    note.tags.contains { tag in
                        tag.name.localizedStandardContains(searchText)
                    }
                }
            }
            
            _notes = Query(
                filter: predicate,
                sort: \.modifiedAt,
                order: .reverse
            )
        }
    }
    
    var body: some View {
        List(notes) { note in
            VStack(alignment: .leading) {
                Text(note.title)
                    .font(.headline)
                Text(note.content.prefix(100))
                    .font(.caption)
                    .foregroundColor(.secondary)
                    .lineLimit(2)
                HStack {
                    Text(note.modifiedAt, style: .relative)
                        .font(.caption2)
                        .foregroundColor(.tertiary)
                    ForEach(note.tags, id: \.name) { tag in
                        Text(tag.name)
                            .font(.caption2)
                            .padding(.horizontal, 4)
                            .background(Color(hex: tag.colorHex).opacity(0.2))
                            .cornerRadius(3)
                    }
                }
            }
        }
    }
}
```

### Exercise 3: Migration Test

**โจทย์:** เขียน test เพื่อตรวจสอบว่า migration ทำงานถูกต้อง

```swift
import XCTest
import SwiftData

class MigrationTests: XCTestCase {
    
    func testMigrationFromV1toV2() throws {
        // 1. สร้าง V1 container และ insert data
        let v1Config = ModelConfiguration(isStoredInMemoryOnly: true)
        let v1Container = try ModelContainer(
            for: SchemaV1.TodoItem.self,
            configurations: v1Config
        )
        
        let v1Context = v1Container.mainContext
        
        // Insert V1 data
        let todo1 = SchemaV1.TodoItem(title: "Urgent Task")
        let todo2 = SchemaV1.TodoItem(title: "Regular Task")
        v1Context.insert(todo1)
        v1Context.insert(todo2)
        try v1Context.save()
        
        // Verify V1 data
        let v1Count = try v1Context.fetchCount(
            FetchDescriptor<SchemaV1.TodoItem>()
        )
        XCTAssertEqual(v1Count, 2)
        
        // 2. สร้าง V2 container กับ migration
        // Note: ใน test จริงต้องใช้ shared store URL
        // นี่เป็น conceptual test
        
        let v2Config = ModelConfiguration(isStoredInMemoryOnly: true)
        let v2Container = try ModelContainer(
            for: SchemaV2.TodoItem.self, SchemaV2.Category.self,
            migrationPlan: TodoMigrationPlan.self,
            configurations: v2Config
        )
        
        let v2Context = v2Container.mainContext
        
        // 3. Verify migration results
        let v2Todos = try v2Context.fetch(
            FetchDescriptor<SchemaV2.TodoItem>(
                sortBy: [SortDescriptor(\.title)]
            )
        )
        
        // ตรวจสอบ default priority ถูก set
        XCTAssertEqual(v2Todos.count, 2)
        
        let urgentTodo = v2Todos.first { $0.title.lowercased().contains("urgent") }
        XCTAssertNotNil(urgentTodo)
        XCTAssertEqual(urgentTodo?.priority, 3)  // High priority for "urgent" keyword
        
        let regularTodo = v2Todos.first { $0.title.lowercased().contains("regular") }
        XCTAssertNotNil(regularTodo)
        XCTAssertEqual(regularTodo?.priority, 1)  // Default low priority
    }
}
```

### Exercise 4: Sync Status Tracking

**โจทย์:** สร้าง model ที่ track sync status กับ remote API

```swift
import SwiftData
import Foundation

enum SyncStatus: String, Codable {
    case local       // ยังไม่ sync
    case syncing     // กำลัง sync
    case synced      // sync แล้ว
    case conflict    // มี conflict
    case failed      // sync ล้มเหลว
}

@Model
final class SyncableItem {
    @Attribute(.unique)
    var localId: UUID
    
    var remoteId: String?
    var syncStatus: SyncStatus
    var lastSyncedAt: Date?
    var syncError: String?
    var version: Int
    
    // Actual data
    var title: String
    var content: String
    var modifiedAt: Date
    
    @Transient
    var isSyncing: Bool { syncStatus == .syncing }
    
    @Transient
    var needsSync: Bool { syncStatus == .local || syncStatus == .failed }
    
    init(title: String, content: String) {
        self.localId = UUID()
        self.title = title
        self.content = content
        self.syncStatus = .local
        self.modifiedAt = Date()
        self.version = 1
    }
    
    func markAsSynced(remoteId: String) {
        self.remoteId = remoteId
        self.syncStatus = .synced
        self.lastSyncedAt = Date()
        self.syncError = nil
    }
    
    func markAsFailed(error: String) {
        self.syncStatus = .failed
        self.syncError = error
    }
}

// Sync service
@ModelActor
actor SyncService {
    
    func syncPendingItems(apiClient: APIClient) async {
        do {
            let pendingDescriptor = FetchDescriptor<SyncableItem>(
                predicate: #Predicate { item in
                    item.syncStatus == .local || item.syncStatus == .failed
                }
            )
            
            let pending = try modelContext.fetch(pendingDescriptor)
            
            for item in pending {
                item.syncStatus = .syncing
                do {
                    try modelContext.save()
                } catch { continue }
                
                do {
                    let remoteId = try await apiClient.uploadItem(
                        title: item.title,
                        content: item.content
                    )
                    item.markAsSynced(remoteId: remoteId)
                } catch {
                    item.markAsFailed(error: error.localizedDescription)
                }
                
                try? modelContext.save()
            }
        } catch {
            print("Sync error: \(error)")
        }
    }
}

// Mock API client
class APIClient {
    func uploadItem(title: String, content: String) async throws -> String {
        // Simulate API call
        try await Task.sleep(for: .milliseconds(500))
        return UUID().uuidString
    }
}
```

### Exercise 5: Performance Optimization Challenge

**โจทย์:** ปรับปรุง performance ของ app ที่มี 10,000+ records

```swift
import SwiftData
import SwiftUI

// ❌ Naive implementation - ช้า
struct SlowListView: View {
    @Query var allItems: [TodoItem]  // Fetch ทุกอย่าง!
    
    var body: some View {
        List(allItems) { item in  // Render 10,000 rows!
            Text(item.title)
        }
    }
}

// ✅ Optimized implementation
struct OptimizedListView: View {
    
    @State private var currentPage = 0
    private let pageSize = 50
    
    // Fetch เฉพาะ page ปัจจุบัน
    @State private var displayedItems: [TodoItem] = []
    
    @Environment(\.modelContext) private var context
    
    var body: some View {
        List {
            ForEach(displayedItems) { item in
                TodoRow(task: item as! Task)
                    .onAppear {
                        // Infinite scroll - load more when near end
                        if item.id == displayedItems.last?.id {
                            loadNextPage()
                        }
                    }
            }
            
            if displayedItems.count < totalCount {
                ProgressView()
                    .frame(maxWidth: .infinity)
            }
        }
        .onAppear {
            loadFirstPage()
        }
    }
    
    private var totalCount: Int {
        (try? context.fetchCount(FetchDescriptor<TodoItem>())) ?? 0
    }
    
    private func loadFirstPage() {
        displayedItems = fetchPage(0)
    }
    
    private func loadNextPage() {
        currentPage += 1
        let newItems = fetchPage(currentPage)
        displayedItems.append(contentsOf: newItems)
    }
    
    private func fetchPage(_ page: Int) -> [TodoItem] {
        var descriptor = FetchDescriptor<TodoItem>(
            sortBy: [SortDescriptor(\.createdAt, order: .reverse)]
        )
        descriptor.fetchLimit = pageSize
        descriptor.fetchOffset = page * pageSize
        descriptor.includePendingChanges = false  // Skip unsaved changes for performance
        
        return (try? context.fetch(descriptor)) ?? []
    }
}

// Section-based grouping สำหรับ better UX
struct GroupedTodoView: View {
    
    @Query(sort: \TodoItem.createdAt, order: .reverse)
    var allTodos: [TodoItem]
    
    var groupedTodos: [(String, [TodoItem])] {
        let calendar = Calendar.current
        let grouped = Dictionary(grouping: allTodos) { todo in
            if calendar.isDateInToday(todo.createdAt) {
                return "Today"
            } else if calendar.isDateInYesterday(todo.createdAt) {
                return "Yesterday"
            } else {
                let formatter = DateFormatter()
                formatter.dateStyle = .medium
                return formatter.string(from: todo.createdAt)
            }
        }
        return grouped.sorted { $0.key > $1.key }
    }
    
    var body: some View {
        List {
            ForEach(groupedTodos, id: \.0) { section, todos in
                Section(section) {
                    ForEach(todos) { todo in
                        Text(todo.title)
                    }
                }
            }
        }
    }
}
```

---

## สรุป

SwiftData คือ framework ที่ทรงพลังสำหรับการ persistence ใน iOS 17+ โดยมีข้อดีหลักๆ:

### Key Concepts

1. **@Model macro** - แปลง Swift class ธรรมดาเป็น persistent model
2. **ModelContainer** - จัดการ persistent stores
3. **ModelContext** - active working space สำหรับ model objects
4. **#Predicate macro** - type-safe query building
5. **@Query property wrapper** - reactive data binding ใน SwiftUI
6. **VersionedSchema + SchemaMigrationPlan** - safe database migrations
7. **@ModelActor** - thread-safe background operations

### Best Practices

- ใช้ `final class` สำหรับ `@Model` types เพื่อ performance
- กำหนด inverse relationships เสมอ
- ใช้ `fetchLimit` สำหรับ large datasets
- ทดสอบด้วย in-memory `ModelConfiguration`
- ใช้ `@ModelActor` สำหรับ background work
- กำหนด `@Attribute(.unique)` สำหรับ identifiers

### Migration from Core Data

SwiftData ถูกสร้างบน Core Data ดังนั้นสามารถ:
- ใช้ร่วมกับ Core Data ได้
- Migrate existing stores ได้
- ใช้ CloudKit sync ได้โดยอัตโนมัติ

### When to Use SwiftData

- ✅ iOS 17+ only apps
- ✅ New projects ที่ต้องการ modern API
- ✅ Apps ที่ต้องการ iCloud sync ง่ายๆ
- ✅ SwiftUI-first apps
- ❌ Apps ที่ต้องการ iOS 16 หรือต่ำกว่า (ใช้ Core Data)
- ❌ Complex migration scenarios (Core Data ยืดหยุ่นกว่า)

### Resources

- [SwiftData Documentation](https://developer.apple.com/documentation/swiftdata)
- [WWDC23: Meet SwiftData](https://developer.apple.com/videos/play/wwdc2023/10187/)
- [WWDC23: Model your schema with SwiftData](https://developer.apple.com/videos/play/wwdc2023/10195/)
- [WWDC23: Migrate to SwiftData](https://developer.apple.com/videos/play/wwdc2023/10189/)
- [WWDC23: Build an app with SwiftData](https://developer.apple.com/videos/play/wwdc2023/10154/)

---

*จบ Part 96: SwiftData — Modern Persistence Framework*
