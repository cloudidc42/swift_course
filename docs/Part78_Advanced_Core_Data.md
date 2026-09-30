# Part 78: Advanced Core Data - Core Data ขั้นสูง

## บทนำ

Core Data เป็น Framework ที่ทรงพลังสำหรับ Persistence Layer ใน iOS แต่การใช้งานระดับ Advanced ต้องเข้าใจเรื่อง Performance Optimization, Multi-Context Setup, Migration Strategies และการทำงานร่วมกับ Swift Concurrency บทนี้จะครอบคลุมทุกหัวข้อที่จำเป็นสำหรับ Production App

---

## 1. Core Data Performance Optimization

### 1.1 Fetch Request Optimization

```swift
// CoreDataStack.swift
import CoreData

class CoreDataStack {
    static let shared = CoreDataStack()
    
    lazy var persistentContainer: NSPersistentContainer = {
        let container = NSPersistentContainer(name: "ShopSwift")
        
        // Performance Configuration
        let description = container.persistentStoreDescriptions.first
        
        // เปิด Remote Change Notifications สำหรับ CloudKit
        description?.setOption(true as NSNumber, 
                               forKey: NSPersistentStoreRemoteChangeNotificationPostOptionKey)
        
        // เปิด Persistent History Tracking
        description?.setOption(true as NSNumber,
                               forKey: NSPersistentHistoryTrackingKey)
        
        container.loadPersistentStores { _, error in
            if let error = error {
                fatalError("Core Data failed: \(error)")
            }
        }
        
        // Performance Settings
        container.viewContext.automaticallyMergesChangesFromParent = true
        container.viewContext.mergePolicy = NSMergeByPropertyObjectTrumpMergePolicy
        
        // Query Generation - ใช้ Snapshot ที่สม่ำเสมอ
        try? container.viewContext.setQueryGenerationFrom(.current)
        
        return container
    }()
    
    var viewContext: NSManagedObjectContext {
        persistentContainer.viewContext
    }
    
    // Background Context สำหรับ Import / Heavy Operations
    func newBackgroundContext() -> NSManagedObjectContext {
        let context = persistentContainer.newBackgroundContext()
        context.mergePolicy = NSMergeByPropertyObjectTrumpMergePolicy
        context.undoManager = nil  // ไม่ต้องการ Undo ใน Background
        return context
    }
    
    var backgroundContext: NSManagedObjectContext {
        persistentContainer.newBackgroundContext()
    }
}
```

### 1.2 Fetch Request Performance Tips

```swift
// ProductRepository.swift
import CoreData

class ProductRepository {
    private let stack = CoreDataStack.shared
    
    // ✅ Optimized Fetch Request
    func fetchProducts(
        category: String? = nil,
        limit: Int = 20,
        offset: Int = 0
    ) throws -> [Product] {
        let request = ProductEntity.fetchRequest()
        
        // 1. Predicate - ใช้ Index ที่มีเสมอ
        if let category = category {
            request.predicate = NSPredicate(
                format: "categoryId == %@", category
            )
        }
        
        // 2. Fetch Limit - ไม่ Fetch มากเกินความจำเป็น
        request.fetchLimit = limit
        request.fetchOffset = offset
        
        // 3. Sort Descriptors - ใช้ Indexed Attributes
        request.sortDescriptors = [
            NSSortDescriptor(keyPath: \ProductEntity.isFeatured, ascending: false),
            NSSortDescriptor(keyPath: \ProductEntity.updatedAt, ascending: false)
        ]
        
        // 4. Properties to Fetch - Fetch เฉพาะที่ต้องการ
        request.propertiesToFetch = ["id", "name", "price", "imageURL", "isFeatured"]
        request.resultType = .dictionaryResultType  // เร็วกว่า Object Result
        
        // 5. Return as Faults - ลด Memory
        request.returnsObjectsAsFaults = true
        
        // 6. Include Pending Changes = false เมื่อ Read-only
        request.includesPendingChanges = false
        
        return try stack.viewContext.fetch(request)
    }
    
    // Count without fetching objects
    func countProducts(category: String? = nil) throws -> Int {
        let request = ProductEntity.fetchRequest()
        if let category = category {
            request.predicate = NSPredicate(format: "categoryId == %@", category)
        }
        return try stack.viewContext.count(for: request)
    }
    
    // Prefetch Relationships ลด Faulting
    func fetchProductsWithDetails() throws -> [ProductEntity] {
        let request = ProductEntity.fetchRequest()
        
        // Prefetch ความสัมพันธ์ที่จะใช้บ่อย
        request.relationshipKeyPathsForPrefetching = [
            "images",
            "variants",
            "category"
        ]
        
        return try stack.viewContext.fetch(request)
    }
}
```

---

## 2. Batch Operations

### 2.1 NSBatchUpdateRequest

```swift
// BatchOperations.swift
import CoreData

class BatchOperationManager {
    private let stack = CoreDataStack.shared
    
    // Update หลาย Records พร้อมกัน โดยไม่โหลดเข้า Memory
    func updateProductPrices(
        categoryId: String,
        discountPercentage: Double
    ) async throws {
        let context = stack.backgroundContext
        
        try await context.perform {
            let request = NSBatchUpdateRequest(entityName: "ProductEntity")
            
            // Predicate - อัปเดตเฉพาะ Category นี้
            request.predicate = NSPredicate(format: "categoryId == %@", categoryId)
            
            // ค่าที่จะอัปเดต - ใช้ NSExpression สำหรับ Calculation
            request.propertiesToUpdate = [
                "discountPercentage": discountPercentage,
                "updatedAt": Date(),
                "isOnSale": true
            ]
            
            // ต้องการ Object IDs กลับมาเพื่อ Merge
            request.resultType = .updatedObjectIDsResultType
            
            let result = try context.execute(request) as! NSBatchUpdateResult
            
            // Merge การเปลี่ยนแปลงไปยัง View Context
            let objectIDs = result.result as! [NSManagedObjectID]
            let changes = [NSUpdatedObjectsKey: objectIDs]
            
            NSManagedObjectContext.mergeChanges(
                fromRemoteContextSave: changes,
                into: [self.stack.viewContext]
            )
        }
    }
    
    // Mark Orders as Delivered
    func markOrdersAsDelivered(orderIds: [String]) async throws {
        let context = stack.backgroundContext
        
        try await context.perform {
            let request = NSBatchUpdateRequest(entityName: "OrderEntity")
            request.predicate = NSPredicate(format: "id IN %@", orderIds)
            request.propertiesToUpdate = [
                "status": "delivered",
                "deliveredAt": Date()
            ]
            request.resultType = .updatedObjectIDsResultType
            
            let result = try context.execute(request) as! NSBatchUpdateResult
            let changes = [NSUpdatedObjectsKey: result.result as! [NSManagedObjectID]]
            
            NSManagedObjectContext.mergeChanges(
                fromRemoteContextSave: changes,
                into: [self.stack.viewContext]
            )
        }
    }
}
```

### 2.2 NSBatchDeleteRequest

```swift
extension BatchOperationManager {
    
    // ลบ Orders เก่าโดยไม่โหลดเข้า Memory
    func deleteOldOrders(olderThan days: Int) async throws {
        let context = stack.backgroundContext
        
        try await context.perform {
            let cutoffDate = Calendar.current.date(
                byAdding: .day,
                value: -days,
                to: Date()
            )!
            
            let request = NSFetchRequest<NSFetchRequestResult>(entityName: "OrderEntity")
            request.predicate = NSPredicate(
                format: "createdAt < %@ AND status == %@",
                cutoffDate as NSDate,
                "delivered"
            )
            
            let deleteRequest = NSBatchDeleteRequest(fetchRequest: request)
            deleteRequest.resultType = .resultTypeObjectIDs
            
            let result = try context.execute(deleteRequest) as! NSBatchDeleteResult
            let deletedIDs = result.result as! [NSManagedObjectID]
            
            // Merge deletions to view context
            let changes = [NSDeletedObjectsKey: deletedIDs]
            NSManagedObjectContext.mergeChanges(
                fromRemoteContextSave: changes,
                into: [self.stack.viewContext]
            )
            
            Logger.info("Deleted \(deletedIDs.count) old orders")
        }
    }
    
    // ลบ Cache Products ที่หมดอายุ
    func clearExpiredProductCache() async throws {
        let context = stack.backgroundContext
        
        try await context.perform {
            let expiryDate = Date(timeIntervalSinceNow: -3600)  // 1 hour ago
            
            let request = NSFetchRequest<NSFetchRequestResult>(entityName: "CachedProductEntity")
            request.predicate = NSPredicate(format: "cachedAt < %@", expiryDate as NSDate)
            
            let deleteRequest = NSBatchDeleteRequest(fetchRequest: request)
            deleteRequest.resultType = .resultTypeObjectIDs
            
            let result = try context.execute(deleteRequest) as! NSBatchDeleteResult
            let changes = [NSDeletedObjectsKey: result.result as! [NSManagedObjectID]]
            
            NSManagedObjectContext.mergeChanges(
                fromRemoteContextSave: changes,
                into: [self.stack.viewContext]
            )
        }
    }
}
```

### 2.3 NSBatchInsertRequest

```swift
extension BatchOperationManager {
    
    // Import Products จำนวนมากจาก API อย่างมีประสิทธิภาพ
    func batchInsertProducts(_ products: [Product]) async throws {
        let context = stack.backgroundContext
        
        try await context.perform {
            // สร้าง Batch Insert Request
            var index = 0
            let insertRequest = NSBatchInsertRequest(
                entity: ProductEntity.entity(),
                managedObjectHandler: { object in
                    guard index < products.count else { return true }
                    
                    let productEntity = object as! ProductEntity
                    let product = products[index]
                    
                    productEntity.id = product.id
                    productEntity.name = product.name
                    productEntity.price = product.price
                    productEntity.categoryId = product.category.id
                    productEntity.updatedAt = Date()
                    productEntity.cachedAt = Date()
                    
                    index += 1
                    return false  // return true เมื่อเสร็จ
                }
            )
            
            insertRequest.resultType = .objectIDs
            
            let result = try context.execute(insertRequest) as! NSBatchInsertResult
            let insertedIDs = result.result as! [NSManagedObjectID]
            
            // Merge to view context
            let changes = [NSInsertedObjectsKey: insertedIDs]
            NSManagedObjectContext.mergeChanges(
                fromRemoteContextSave: changes,
                into: [self.stack.viewContext]
            )
            
            Logger.info("Batch inserted \(insertedIDs.count) products")
        }
    }
    
    // ประสิทธิภาพเปรียบเทียบ
    // Batch Insert: 10,000 records ใช้เวลา ~0.5 วินาที
    // Normal Insert: 10,000 records ใช้เวลา ~5-10 วินาที
}
```

---

## 3. Persistent History Tracking

### 3.1 NSPersistentHistoryTransaction

Persistent History Tracking ช่วยให้ App Extension และ Main App รู้ว่ามีการเปลี่ยนแปลงอะไรบ้างตั้งแต่ครั้งที่แล้ว

```swift
// PersistentHistoryManager.swift
import CoreData

class PersistentHistoryManager {
    static let shared = PersistentHistoryManager()
    
    private let stack = CoreDataStack.shared
    private let userDefaults = UserDefaults(suiteName: "group.com.shopswift.app")!
    private let tokenKey = "persistent_history_token"
    
    // Process ประวัติการเปลี่ยนแปลงตั้งแต่ Token ล่าสุด
    func processPersistentHistory() async throws {
        let context = stack.backgroundContext
        
        try await context.perform { [self] in
            // โหลด Token ล่าสุดที่ Process แล้ว
            let token = self.loadHistoryToken()
            
            // Fetch History ตั้งแต่ Token นั้น
            let fetchRequest = NSPersistentHistoryChangeRequest.fetchHistory(after: token)
            fetchRequest.fetchRequest = NSFetchRequest<NSFetchRequestResult>()
            
            guard let result = try context.execute(fetchRequest) as? NSPersistentHistoryResult,
                  let transactions = result.result as? [NSPersistentHistoryTransaction],
                  !transactions.isEmpty else {
                return
            }
            
            Logger.info("Processing \(transactions.count) history transactions")
            
            for transaction in transactions {
                self.processTransaction(transaction, in: context)
            }
            
            // อัปเดต Token เป็น Transaction ล่าสุด
            if let lastToken = transactions.last?.token {
                self.saveHistoryToken(lastToken)
                
                // Purge ประวัติเก่า
                try self.purgeHistory(before: lastToken, in: context)
            }
            
            // Merge เข้า View Context
            DispatchQueue.main.async {
                self.mergeChanges(from: transactions)
            }
        }
    }
    
    private func processTransaction(
        _ transaction: NSPersistentHistoryTransaction,
        in context: NSManagedObjectContext
    ) {
        guard let changes = transaction.changes else { return }
        
        for change in changes {
            let entityName = change.changedObjectID.entity.name ?? "Unknown"
            
            switch change.changeType {
            case .insert:
                Logger.debug("Inserted: \(entityName) - \(change.changedObjectID)")
                handleInsert(change, in: context)
                
            case .update:
                Logger.debug("Updated: \(entityName) - \(change.updatedProperties ?? [])")
                handleUpdate(change, in: context)
                
            case .delete:
                Logger.debug("Deleted: \(entityName)")
                handleDelete(change)
                
            @unknown default:
                break
            }
        }
    }
    
    private func handleInsert(
        _ change: NSPersistentHistoryChange,
        in context: NSManagedObjectContext
    ) {
        // ตัวอย่าง: เมื่อมีการเพิ่ม Order ใหม่จาก App Extension
        guard change.changedObjectID.entity.name == "OrderEntity" else { return }
        
        // Notify ว่ามี Order ใหม่
        NotificationCenter.default.post(
            name: .newOrderAdded,
            object: change.changedObjectID
        )
    }
    
    private func handleUpdate(
        _ change: NSPersistentHistoryChange,
        in context: NSManagedObjectContext
    ) {
        // ตรวจสอบว่า Stock เปลี่ยน
        if let updatedProps = change.updatedProperties,
           updatedProps.contains(where: { $0.name == "inventoryQuantity" }) {
            NotificationCenter.default.post(
                name: .inventoryChanged,
                object: change.changedObjectID
            )
        }
    }
    
    private func handleDelete(_ change: NSPersistentHistoryChange) {
        // จัดการ Cascading อื่น ๆ ถ้าจำเป็น
    }
    
    private func mergeChanges(from transactions: [NSPersistentHistoryTransaction]) {
        guard !transactions.isEmpty else { return }
        
        // รวบรวม Changes ทั้งหมด
        var insertedIDs: Set<NSManagedObjectID> = []
        var updatedIDs: Set<NSManagedObjectID> = []
        var deletedIDs: Set<NSManagedObjectID> = []
        
        for transaction in transactions {
            guard let changes = transaction.changes else { continue }
            for change in changes {
                switch change.changeType {
                case .insert: insertedIDs.insert(change.changedObjectID)
                case .update: updatedIDs.insert(change.changedObjectID)
                case .delete: deletedIDs.insert(change.changedObjectID)
                @unknown default: break
                }
            }
        }
        
        // Merge เข้า View Context
        stack.viewContext.perform {
            self.stack.viewContext.mergeChanges(
                fromContextDidSave: [
                    NSInsertedObjectsKey: Array(insertedIDs),
                    NSUpdatedObjectsKey: Array(updatedIDs),
                    NSDeletedObjectsKey: Array(deletedIDs)
                ] as [AnyHashable: Any]
            )
        }
    }
    
    // Token Management
    private func loadHistoryToken() -> NSPersistentHistoryToken? {
        guard let data = userDefaults.data(forKey: tokenKey) else { return nil }
        return try? NSKeyedUnarchiver.unarchivedObject(
            ofClass: NSPersistentHistoryToken.self,
            from: data
        )
    }
    
    private func saveHistoryToken(_ token: NSPersistentHistoryToken) {
        let data = try? NSKeyedArchiver.archivedData(
            withRootObject: token,
            requiringSecureCoding: true
        )
        userDefaults.set(data, forKey: tokenKey)
    }
    
    // Purge ประวัติเก่าเพื่อประหยัด Space
    private func purgeHistory(
        before token: NSPersistentHistoryToken,
        in context: NSManagedObjectContext
    ) throws {
        let deleteRequest = NSPersistentHistoryChangeRequest.deleteHistory(before: token)
        try context.execute(deleteRequest)
    }
}
```

### 3.2 History Fetch Request

```swift
// HistoryQueryExample.swift
extension PersistentHistoryManager {
    
    // Query ประวัติเฉพาะ Entity
    func fetchHistory(
        for entityName: String,
        since date: Date
    ) async throws -> [NSPersistentHistoryTransaction] {
        let context = stack.backgroundContext
        
        return try await context.perform {
            // สร้าง Fetch Request สำหรับ History
            let historyFetchRequest = NSPersistentHistoryTransaction.fetchRequest!
            historyFetchRequest.predicate = NSPredicate(
                format: "timestamp > %@", date as NSDate
            )
            
            // Filter เฉพาะ Entity ที่ต้องการ
            let entityDescription = NSEntityDescription.entity(
                forEntityName: entityName,
                in: context
            )!
            let changeFetchRequest = NSPersistentHistoryChange.fetchRequest!
            changeFetchRequest.predicate = NSPredicate(
                format: "changedObjectID.entity == %@", entityDescription
            )
            
            let request = NSPersistentHistoryChangeRequest.fetchHistory(after: date)
            request.fetchRequest = historyFetchRequest
            
            guard let result = try context.execute(request) as? NSPersistentHistoryResult,
                  let transactions = result.result as? [NSPersistentHistoryTransaction] else {
                return []
            }
            
            return transactions
        }
    }
    
    // ฟัง Remote Change Notification
    func observeRemoteChanges() {
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(handleRemoteChange),
            name: .NSPersistentStoreRemoteChange,
            object: stack.persistentContainer.persistentStoreCoordinator
        )
    }
    
    @objc private func handleRemoteChange(_ notification: Notification) {
        Task {
            try? await processPersistentHistory()
        }
    }
}
```

---

## 4. NSPersistentStoreCoordinator Multi-Store Setup

### 4.1 Multiple Store Configuration

```swift
// MultiStoreSetup.swift
import CoreData

class MultiStoreCoreDataStack {
    static let shared = MultiStoreCoreDataStack()
    
    lazy var persistentContainer: NSPersistentContainer = {
        let container = NSPersistentContainer(name: "ShopSwift")
        
        // Store 1: Products (Read-heavy, SQLite)
        let productsDescription = NSPersistentStoreDescription()
        productsDescription.url = storeURL(for: "Products.sqlite")
        productsDescription.configuration = "Products"  // ใน Data Model
        productsDescription.type = NSSQLiteStoreType
        productsDescription.isReadOnly = false
        
        // Store 2: UserData (Read-write, SQLite)
        let userDataDescription = NSPersistentStoreDescription()
        userDataDescription.url = storeURL(for: "UserData.sqlite")
        userDataDescription.configuration = "UserData"
        userDataDescription.type = NSSQLiteStoreType
        
        // Store 3: Temporary Cache (In-Memory)
        let cacheDescription = NSPersistentStoreDescription()
        cacheDescription.url = nil
        cacheDescription.type = NSInMemoryStoreType
        cacheDescription.configuration = "Cache"
        
        // Store 4: CloudKit Sync (Products + UserData)
        let cloudDescription = NSPersistentStoreDescription()
        cloudDescription.url = storeURL(for: "Cloud.sqlite")
        cloudDescription.configuration = "CloudSync"
        cloudDescription.cloudKitContainerOptions = NSPersistentCloudKitContainerOptions(
            containerIdentifier: "iCloud.com.shopswift.app"
        )
        
        container.persistentStoreDescriptions = [
            productsDescription,
            userDataDescription,
            cacheDescription
        ]
        
        container.loadPersistentStores { description, error in
            if let error = error {
                Logger.error("Failed to load store \(description.url?.lastPathComponent ?? ""): \(error)")
            }
        }
        
        return container
    }()
    
    private func storeURL(for name: String) -> URL {
        let appGroupURL = FileManager.default.containerURL(
            forSecurityApplicationGroupIdentifier: "group.com.shopswift.app"
        )
        return appGroupURL!.appendingPathComponent(name)
    }
    
    // Context สำหรับแต่ละ Store
    var productContext: NSManagedObjectContext {
        let ctx = persistentContainer.newBackgroundContext()
        // กำหนดให้ทำงานกับ Products Store
        return ctx
    }
    
    var userDataContext: NSManagedObjectContext {
        persistentContainer.viewContext
    }
}
```

---

## 5. Context Hierarchy (Parent-Child Contexts)

### 5.1 Context Hierarchy สำหรับ Undo Support

```swift
// ContextHierarchy.swift
import CoreData

class ContextHierarchyManager {
    private let stack = CoreDataStack.shared
    
    // Pattern 1: Main Context เป็น Parent
    // Background Context เป็น Child
    func createChildContext() -> NSManagedObjectContext {
        let childContext = NSManagedObjectContext(concurrencyType: .mainQueueConcurrencyType)
        childContext.parent = stack.viewContext
        childContext.undoManager = UndoManager()  // รองรับ Undo
        return childContext
    }
    
    // ตัวอย่าง: Edit Form ที่ Cancel ได้
    func createEditingContext(for objectID: NSManagedObjectID) -> (
        context: NSManagedObjectContext,
        object: NSManagedObject
    ) {
        let editContext = NSManagedObjectContext(concurrencyType: .mainQueueConcurrencyType)
        editContext.parent = stack.viewContext
        editContext.undoManager = UndoManager()
        
        // นำ Object มาใช้ใน Edit Context
        let editableObject = editContext.object(with: objectID)
        return (editContext, editableObject)
    }
    
    // Save changes ขึ้น Parent Context (ยังไม่ Persist)
    func saveToParent(_ context: NSManagedObjectContext) throws {
        guard context.hasChanges else { return }
        
        try context.save()  // Save ไปยัง Parent Context
        // ยังไม่ถึง Persistent Store
    }
    
    // Save ถึง Persistent Store
    func saveToStore() throws {
        let viewContext = stack.viewContext
        guard viewContext.hasChanges else { return }
        try viewContext.save()
    }
    
    // Pattern 2: Private Queue Context เป็น Parent (Best for Import)
    func setupHeavyImportContext() -> NSManagedObjectContext {
        // Private context เชื่อมกับ Persistent Store
        let privateContext = NSManagedObjectContext(concurrencyType: .privateQueueConcurrencyType)
        privateContext.persistentStoreCoordinator = stack.persistentContainer.persistentStoreCoordinator
        
        // Main context เป็น Child
        let mainContext = NSManagedObjectContext(concurrencyType: .mainQueueConcurrencyType)
        mainContext.parent = privateContext
        
        return mainContext
    }
}

// SwiftUI ViewModel ที่ใช้ Child Context
@MainActor
class ProductEditViewModel: ObservableObject {
    @Published var product: ProductEntity
    @Published var hasChanges = false
    
    private let editContext: NSManagedObjectContext
    private let stack = CoreDataStack.shared
    
    init(productID: NSManagedObjectID) {
        // สร้าง Child Context สำหรับ Edit
        editContext = NSManagedObjectContext(concurrencyType: .mainQueueConcurrencyType)
        editContext.parent = CoreDataStack.shared.viewContext
        editContext.undoManager = UndoManager()
        
        // โหลด Object ใน Child Context
        self.product = editContext.object(with: productID) as! ProductEntity
    }
    
    func updateName(_ name: String) {
        product.name = name
        hasChanges = editContext.hasChanges
    }
    
    func updatePrice(_ price: Double) {
        product.price = price
        hasChanges = editContext.hasChanges
    }
    
    func save() throws {
        try editContext.save()  // Save ไปยัง Parent (View Context)
        try stack.viewContext.save()  // Save ลง Disk
        hasChanges = false
    }
    
    func discard() {
        editContext.rollback()  // ยกเลิกการเปลี่ยนแปลงทั้งหมด
        hasChanges = false
    }
    
    func undo() {
        editContext.undoManager?.undo()
        hasChanges = editContext.hasChanges
    }
    
    func redo() {
        editContext.undoManager?.redo()
        hasChanges = editContext.hasChanges
    }
}
```

---

## 6. Import Optimization

### 6.1 Efficient Large Data Import

```swift
// ImportManager.swift
import CoreData

class ProductImportManager {
    private let stack = CoreDataStack.shared
    
    // Import จำนวนมากแบบ Batch
    func importProducts(_ apiProducts: [APIProduct], progressHandler: ((Double) -> Void)? = nil) async throws {
        let context = stack.backgroundContext
        let batchSize = 500
        let total = apiProducts.count
        
        try await context.perform {
            context.undoManager = nil  // ปิด Undo ระหว่าง Import
            
            // แบ่งเป็น Batches
            let batches = stride(from: 0, to: total, by: batchSize).map { startIndex in
                Array(apiProducts[startIndex..<min(startIndex + batchSize, total)])
            }
            
            for (batchIndex, batch) in batches.enumerated() {
                // Process แต่ละ Batch
                for apiProduct in batch {
                    // ใช้ upsert pattern - หาหรือสร้างใหม่
                    let product = self.findOrCreateProduct(
                        id: apiProduct.id,
                        in: context
                    )
                    product.update(from: apiProduct)
                }
                
                // Save ทีละ Batch เพื่อลด Memory Pressure
                if context.hasChanges {
                    try context.save()
                    context.reset()  // ล้าง Memory หลัง Save
                }
                
                // Report Progress
                let progress = Double(batchIndex + 1) / Double(batches.count)
                DispatchQueue.main.async {
                    progressHandler?(progress)
                }
            }
        }
    }
    
    private func findOrCreateProduct(
        id: String,
        in context: NSManagedObjectContext
    ) -> ProductEntity {
        let request = ProductEntity.fetchRequest()
        request.predicate = NSPredicate(format: "id == %@", id)
        request.fetchLimit = 1
        
        if let existing = (try? context.fetch(request))?.first {
            return existing
        }
        
        return ProductEntity(context: context)
    }
    
    // Unique Constraint Import (เร็วกว่า)
    func importWithUniqueConstraint(_ products: [APIProduct]) async throws {
        let context = stack.backgroundContext
        context.mergePolicy = NSMergeByPropertyObjectTrumpMergePolicy
        
        try await context.perform {
            for product in products {
                let entity = ProductEntity(context: context)
                entity.id = product.id  // Unique Constraint บน id
                entity.name = product.name
                entity.price = product.price
                // Core Data จะ Merge อัตโนมัติถ้า id ซ้ำ
            }
            
            try context.save()
        }
    }
}
```

---

## 7. Faulting และ Unfaulting

### 7.1 Understanding Core Data Faults

```swift
// FaultingDemo.swift
import CoreData

class FaultingDemoManager {
    private let stack = CoreDataStack.shared
    
    func demonstrateFaulting() throws {
        let context = stack.viewContext
        
        // Fetch products - returned as Faults
        let request = ProductEntity.fetchRequest()
        request.returnsObjectsAsFaults = true  // Default
        let products = try context.fetch(request)
        
        // ณ จุดนี้ products เป็น Faults - ยังไม่โหลดข้อมูล
        print("Is fault: \(products.first?.isFault ?? true)")  // true
        
        // เข้าถึง property ทำให้ Unfault
        let _ = products.first?.name
        print("Is fault: \(products.first?.isFault ?? true)")  // false
        
        // Force prefetch เพื่อหลีกเลี่ยง N+1 Problem
        let prefetchRequest = ProductEntity.fetchRequest()
        prefetchRequest.relationshipKeyPathsForPrefetching = ["images", "category"]
        // Core Data จะโหลด images และ category ใน batch เดียว
    }
    
    // Batch Faulting - โหลดทีเดียวหลาย Objects
    func prefetchObjects(_ objectIDs: [NSManagedObjectID]) {
        let context = stack.viewContext
        
        context.perform {
            // Force resolve faults สำหรับ objectIDs ทั้งหมด
            let request = NSBatchFetchRequest(
                objectIDs: objectIDs,
                context: context
            )
        }
    }
    
    // ควรใช้ returnsObjectsAsFaults = false เมื่อ
    // รู้ว่าจะต้องใช้ข้อมูลทุก property
    func fetchAllProductData() throws -> [ProductEntity] {
        let request = ProductEntity.fetchRequest()
        request.returnsObjectsAsFaults = false  // โหลดทุก property ทันที
        request.fetchLimit = 50  // จำกัดจำนวน
        return try stack.viewContext.fetch(request)
    }
}
```

---

## 8. NSManagedObjectContext Merging

### 8.1 Merge Policies

```swift
// MergePolicyDemo.swift
import CoreData

class MergePolicyManager {
    
    // Merge Policies ที่มีใน Core Data:
    
    // 1. NSErrorMergePolicy (Default) - Error ถ้ามี Conflict
    // 2. NSMergeByPropertyStoreTrumpMergePolicy - Store ชนะ
    // 3. NSMergeByPropertyObjectTrumpMergePolicy - Object ชนะ
    // 4. NSOverwriteMergePolicy - Object เขียนทับ Store
    // 5. NSRollbackMergePolicy - Store ชนะ (Rollback Object)
    
    func setupContextsWithMergePolicy() {
        let stack = CoreDataStack.shared
        
        // View Context - ใช้ Object Trump (UI เปลี่ยนชนะ)
        stack.viewContext.mergePolicy = NSMergeByPropertyObjectTrumpMergePolicy
        
        // Background Context - ใช้ Store Trump (Server data ชนะ)
        let bgContext = stack.newBackgroundContext()
        bgContext.mergePolicy = NSMergeByPropertyStoreTrumpMergePolicy
    }
    
    // Manual Merge - Control ทุกอย่างเอง
    func manualMerge(
        backgroundContext: NSManagedObjectContext,
        viewContext: NSManagedObjectContext
    ) {
        NotificationCenter.default.addObserver(
            forName: .NSManagedObjectContextDidSave,
            object: backgroundContext,
            queue: .main
        ) { notification in
            viewContext.performAndWait {
                viewContext.mergeChanges(fromContextDidSave: notification)
            }
        }
    }
    
    // Conflict Handling
    func resolveConflicts(_ conflicts: [NSConstraintConflict]) {
        for conflict in conflicts {
            // conflict.databaseObject - ข้อมูลใน Store
            // conflict.conflictingObjects - Objects ที่ conflict
            
            // Strategy: Server wins (Database Object wins)
            conflict.conflictingObjects.forEach { object in
                // Refresh object จาก Store
                object.managedObjectContext?.refresh(object, mergeChanges: false)
            }
        }
    }
}
```

---

## 9. Core Data กับ CloudKit Advanced Patterns

### 9.1 NSPersistentCloudKitContainer Setup

```swift
// CloudKitCoreDataStack.swift
import CoreData
import CloudKit

class CloudKitCoreDataStack {
    static let shared = CloudKitCoreDataStack()
    
    lazy var container: NSPersistentCloudKitContainer = {
        let container = NSPersistentCloudKitContainer(name: "ShopSwift")
        
        guard let description = container.persistentStoreDescriptions.first else {
            fatalError("Failed to retrieve store description")
        }
        
        // CloudKit Configuration
        description.cloudKitContainerOptions = NSPersistentCloudKitContainerOptions(
            containerIdentifier: "iCloud.com.shopswift.app"
        )
        
        // เปิด Remote Change Notifications
        description.setOption(true as NSNumber,
                              forKey: NSPersistentHistoryTrackingKey)
        description.setOption(true as NSNumber,
                              forKey: NSPersistentStoreRemoteChangeNotificationPostOptionKey)
        
        container.loadPersistentStores { _, error in
            if let error = error {
                fatalError("CloudKit CoreData setup failed: \(error)")
            }
        }
        
        container.viewContext.automaticallyMergesChangesFromParent = true
        container.viewContext.mergePolicy = NSMergeByPropertyObjectTrumpMergePolicy
        
        // Monitor CloudKit sync events
        self.observeSyncEvents(container)
        
        return container
    }()
    
    // Monitor Sync Events
    private func observeSyncEvents(_ container: NSPersistentCloudKitContainer) {
        NotificationCenter.default.addObserver(
            forName: NSPersistentCloudKitContainer.eventChangedNotification,
            object: container,
            queue: .main
        ) { notification in
            guard let event = notification.userInfo?[
                NSPersistentCloudKitContainer.eventNotificationUserInfoKey
            ] as? NSPersistentCloudKitContainer.Event else { return }
            
            switch event.type {
            case .setup:
                Logger.info("CloudKit setup: \(event.succeeded ? "Success" : "Failed")")
            case .import:
                Logger.info("CloudKit import completed: \(event.endDate?.description ?? "ongoing")")
            case .export:
                Logger.info("CloudKit export completed")
            @unknown default:
                break
            }
            
            if let error = event.error {
                // Handle CloudKit errors
                self.handleCloudKitError(error)
            }
        }
    }
    
    private func handleCloudKitError(_ error: Error) {
        if let ckError = error as? CKError {
            switch ckError.code {
            case .quotaExceeded:
                NotificationCenter.default.post(name: .cloudKitQuotaExceeded, object: nil)
            case .notAuthenticated:
                NotificationCenter.default.post(name: .cloudKitNotAuthenticated, object: nil)
            case .networkUnavailable:
                Logger.warning("CloudKit: Network unavailable, will retry")
            default:
                Logger.error("CloudKit error: \(ckError)")
            }
        }
    }
    
    // Separate Public and Private Stores
    func setupSeparateStores() {
        // Private Store (User-specific data)
        let privateDescription = NSPersistentStoreDescription()
        privateDescription.url = privateStoreURL
        privateDescription.configuration = "UserData"
        let privateOptions = NSPersistentCloudKitContainerOptions(
            containerIdentifier: "iCloud.com.shopswift.app"
        )
        privateOptions.databaseScope = .private
        privateDescription.cloudKitContainerOptions = privateOptions
        
        // Public Store (Shared product catalog)
        let publicDescription = NSPersistentStoreDescription()
        publicDescription.url = publicStoreURL
        publicDescription.configuration = "ProductData"
        let publicOptions = NSPersistentCloudKitContainerOptions(
            containerIdentifier: "iCloud.com.shopswift.app"
        )
        publicOptions.databaseScope = .public
        publicDescription.cloudKitContainerOptions = publicOptions
        
        container.persistentStoreDescriptions = [privateDescription, publicDescription]
    }
    
    private var privateStoreURL: URL {
        FileManager.default.urls(for: .applicationSupportDirectory, in: .userDomainMask)
            .first!.appendingPathComponent("UserData.sqlite")
    }
    
    private var publicStoreURL: URL {
        FileManager.default.urls(for: .applicationSupportDirectory, in: .userDomainMask)
            .first!.appendingPathComponent("ProductData.sqlite")
    }
}
```

---

## 10. Core Data Migration Strategies

### 10.1 Lightweight Migration

```swift
// MigrationManager.swift
import CoreData

// Lightweight Migration (V1 -> V2)
// - เพิ่ม Attribute ใหม่ (ต้องมี Default Value)
// - ลบ Attribute
// - เปลี่ยนชื่อ (ใช้ Renaming Identifier)
// - เพิ่ม/ลบ Relationship

class LightweightMigrationSetup {
    static func configure(description: NSPersistentStoreDescription) {
        // เปิด Lightweight Migration อัตโนมัติ
        description.shouldMigrateStoreAutomatically = true
        description.shouldInferMappingModelAutomatically = true
    }
}

// ตัวอย่างการเพิ่ม Attribute ใน Model V2:
// ProductEntity V1: id, name, price
// ProductEntity V2: id, name, price, discountPercentage (default: 0), updatedAt

// Xcode จะ Infer Mapping Model โดยอัตโนมัติสำหรับ Lightweight Changes
```

### 10.2 Mapping Model (Manual Migration)

```swift
// CustomMappingPolicy.swift
import CoreData

// สำหรับ Migration ที่ซับซ้อน เช่น การ Transform ข้อมูล

class ProductToProductV3MigrationPolicy: NSEntityMigrationPolicy {
    
    // ถูกเรียกสำหรับแต่ละ Source Instance
    override func createDestinationInstances(
        forSource sInstance: NSManagedObject,
        in mapping: NSEntityMapping,
        manager: NSMigrationManager
    ) throws {
        // สร้าง Destination Instance
        let destInstance = NSEntityDescription.insertNewObject(
            forEntityName: mapping.destinationEntityName!,
            into: manager.destinationContext
        )
        
        // Copy ข้อมูลพื้นฐาน
        let basicKeys = ["id", "name", "categoryId"]
        for key in basicKeys {
            if let value = sInstance.value(forKey: key) {
                destInstance.setValue(value, forKey: key)
            }
        }
        
        // Transform: price เดิมเป็น Int (Satang) -> Double (Baht)
        if let priceInSatang = sInstance.value(forKey: "price") as? Int {
            destInstance.setValue(Double(priceInSatang) / 100.0, forKey: "price")
        }
        
        // Transform: รวม firstName + lastName -> displayName
        let firstName = sInstance.value(forKey: "firstName") as? String ?? ""
        let lastName = sInstance.value(forKey: "lastName") as? String ?? ""
        destInstance.setValue("\(firstName) \(lastName)".trimmingCharacters(in: .whitespaces),
                             forKey: "displayName")
        
        // เพิ่ม Default Values สำหรับ Attributes ใหม่
        destInstance.setValue(Date(), forKey: "createdAt")
        destInstance.setValue(false, forKey: "isArchived")
        
        // แจ้ง Manager ว่า Source -> Destination mapping
        manager.associate(sourceInstance: sInstance,
                         withDestinationInstance: destInstance,
                         for: mapping)
    }
    
    // หลัง create ทั้งหมด - จัดการ Relationships
    override func createRelationships(
        forDestination dInstance: NSManagedObject,
        in mapping: NSEntityMapping,
        manager: NSMigrationManager
    ) throws {
        try super.createRelationships(forDestination: dInstance, in: mapping, manager: manager)
        
        // เพิ่ม Relationship Logic เพิ่มเติมถ้าต้องการ
    }
}
```

### 10.3 Progressive Migration

```swift
// ProgressiveMigrationManager.swift
import CoreData

// สำหรับ Migration หลายขั้นตอน V1 -> V2 -> V3 -> V4
class ProgressiveMigrationManager {
    
    enum MigrationError: Error {
        case migrationFailed(String)
        case storeNotFound
        case incompatibleModels
    }
    
    // ลำดับ Version ทั้งหมด
    private let migrationOrder = [
        "ShopSwift_V1",
        "ShopSwift_V2", 
        "ShopSwift_V3",
        "ShopSwift_V4"  // Current Version
    ]
    
    func migrateStore(at storeURL: URL) throws {
        guard FileManager.default.fileExists(atPath: storeURL.path) else {
            return  // Store ยังไม่มี ไม่ต้อง Migrate
        }
        
        // ตรวจสอบ Version ของ Store
        let metadata = try NSPersistentStoreCoordinator.metadataForPersistentStore(
            ofType: NSSQLiteStoreType,
            at: storeURL
        )
        
        // หา Version ปัจจุบันของ Store
        guard let currentVersionIndex = findCurrentVersion(metadata: metadata) else {
            throw MigrationError.incompatibleModels
        }
        
        let targetVersionIndex = migrationOrder.count - 1
        guard currentVersionIndex < targetVersionIndex else {
            Logger.info("Store is already at latest version")
            return
        }
        
        // Migrate ทีละขั้น
        var currentURL = storeURL
        for versionIndex in currentVersionIndex..<targetVersionIndex {
            let fromVersion = migrationOrder[versionIndex]
            let toVersion = migrationOrder[versionIndex + 1]
            
            Logger.info("Migrating: \(fromVersion) -> \(toVersion)")
            currentURL = try migrateOneStep(
                storeURL: currentURL,
                fromVersion: fromVersion,
                toVersion: toVersion
            )
        }
        
        // Replace original store
        try replaceStore(at: storeURL, with: currentURL)
    }
    
    private func findCurrentVersion(metadata: [String: Any]) -> Int? {
        for (index, version) in migrationOrder.enumerated() {
            guard let model = NSManagedObjectModel.mergedModel(
                from: nil,
                forStoreMetadata: metadata
            ) else { continue }
            
            // ตรวจสอบว่า Model นี้ Compatible กับ metadata
            if model.isConfiguration(withName: nil, compatibleWithStoreMetadata: metadata) {
                return index
            }
        }
        return nil
    }
    
    private func migrateOneStep(
        storeURL: URL,
        fromVersion: String,
        toVersion: String
    ) throws -> URL {
        let sourceModel = try loadModel(named: fromVersion)
        let destinationModel = try loadModel(named: toVersion)
        
        // หา Mapping Model
        let mappingModel = try NSMappingModel.inferredMappingModel(
            forSourceModel: sourceModel,
            destinationModel: destinationModel
        )
        
        // สร้าง Destination Store URL
        let destinationURL = storeURL.deletingLastPathComponent()
            .appendingPathComponent("migration_\(toVersion).sqlite")
        
        // Perform Migration
        let migrationManager = NSMigrationManager(
            sourceModel: sourceModel,
            destinationModel: destinationModel
        )
        
        try migrationManager.migrateStore(
            from: storeURL,
            sourceType: NSSQLiteStoreType,
            options: nil,
            with: mappingModel,
            toDestinationURL: destinationURL,
            destinationType: NSSQLiteStoreType,
            destinationOptions: nil
        )
        
        return destinationURL
    }
    
    private func loadModel(named name: String) throws -> NSManagedObjectModel {
        guard let url = Bundle.main.url(forResource: name, withExtension: "mom"),
              let model = NSManagedObjectModel(contentsOf: url) else {
            throw MigrationError.migrationFailed("Cannot load model: \(name)")
        }
        return model
    }
    
    private func replaceStore(at originalURL: URL, with migratedURL: URL) throws {
        let coordinator = NSPersistentStoreCoordinator(managedObjectModel: NSManagedObjectModel())
        try coordinator.replacePersistentStore(
            at: originalURL,
            destinationOptions: nil,
            withPersistentStoreFrom: migratedURL,
            sourceOptions: nil,
            ofType: NSSQLiteStoreType
        )
        try FileManager.default.removeItem(at: migratedURL)
    }
}
```

---

## 11. Core Data Testing Strategies

### 11.1 In-Memory Store สำหรับ Tests

```swift
// TestCoreDataStack.swift
import CoreData
import XCTest

class TestCoreDataStack {
    
    // In-Memory Store - เร็ว ไม่มีผลต่อ Production data
    static func makeInMemoryStack(modelName: String = "ShopSwift") -> NSPersistentContainer {
        let container = NSPersistentContainer(name: modelName)
        
        let description = NSPersistentStoreDescription()
        description.type = NSInMemoryStoreType
        description.shouldAddStoreAsynchronously = false
        
        container.persistentStoreDescriptions = [description]
        
        container.loadPersistentStores { _, error in
            if let error = error {
                XCTFail("In-memory store failed: \(error)")
            }
        }
        
        return container
    }
    
    // Convenience context สำหรับ Tests
    static func makeTestContext() -> NSManagedObjectContext {
        return makeInMemoryStack().viewContext
    }
}

// ตัวอย่าง Test
class ProductRepositoryTests: XCTestCase {
    var sut: ProductRepository!
    var context: NSManagedObjectContext!
    
    override func setUp() {
        super.setUp()
        context = TestCoreDataStack.makeTestContext()
        sut = ProductRepository(context: context)
    }
    
    override func tearDown() {
        sut = nil
        context = nil
        super.tearDown()
    }
    
    func testFetchProducts_returnsCorrectCount() throws {
        // Arrange
        createTestProducts(count: 5, in: context)
        
        // Act
        let products = try sut.fetchProducts()
        
        // Assert
        XCTAssertEqual(products.count, 5)
    }
    
    func testFetchProducts_filtersByCategory() throws {
        // Arrange
        createTestProducts(count: 3, category: "electronics", in: context)
        createTestProducts(count: 2, category: "clothing", in: context)
        
        // Act
        let products = try sut.fetchProducts(category: "electronics")
        
        // Assert
        XCTAssertEqual(products.count, 3)
        XCTAssertTrue(products.allSatisfy { $0.categoryId == "electronics" })
    }
    
    func testBatchInsert_insertsAllProducts() async throws {
        // Arrange
        let apiProducts = (0..<100).map { i in
            APIProduct(id: "p\(i)", name: "Product \(i)", price: Double(i * 100))
        }
        
        // Act
        let manager = ProductImportManager()
        try await manager.batchInsertProducts(apiProducts)
        
        // Assert
        let count = try context.count(for: ProductEntity.fetchRequest())
        XCTAssertEqual(count, 100)
    }
    
    // Helper
    @discardableResult
    private func createTestProducts(
        count: Int,
        category: String = "test",
        in context: NSManagedObjectContext
    ) -> [ProductEntity] {
        var products: [ProductEntity] = []
        for i in 0..<count {
            let product = ProductEntity(context: context)
            product.id = UUID().uuidString
            product.name = "Product \(i)"
            product.price = Double(i * 100)
            product.categoryId = category
            products.append(product)
        }
        try? context.save()
        return products
    }
}
```

### 11.2 Advanced Testing Patterns

```swift
// CoreDataTestHelper.swift
import CoreData

// Protocol สำหรับ Testable Repositories
protocol ProductRepositoryProtocol {
    func fetchProducts(category: String?) async throws -> [Product]
    func saveProduct(_ product: Product) async throws
    func deleteProduct(id: String) async throws
}

// Mock สำหรับ Unit Tests
class MockProductRepository: ProductRepositoryProtocol {
    var productsToReturn: [Product] = []
    var savedProducts: [Product] = []
    var deletedProductIDs: [String] = []
    var shouldThrowError: Error?
    
    func fetchProducts(category: String?) async throws -> [Product] {
        if let error = shouldThrowError { throw error }
        return productsToReturn.filter {
            category == nil || $0.category.id == category
        }
    }
    
    func saveProduct(_ product: Product) async throws {
        if let error = shouldThrowError { throw error }
        savedProducts.append(product)
    }
    
    func deleteProduct(id: String) async throws {
        if let error = shouldThrowError { throw error }
        deletedProductIDs.append(id)
    }
}

// Test สำหรับ ViewModel
@MainActor
class ProductListViewModelTests: XCTestCase {
    var sut: ProductListViewModel!
    var mockRepository: MockProductRepository!
    var mockAnalytics: MockAnalyticsService!
    
    override func setUp() {
        super.setUp()
        mockRepository = MockProductRepository()
        mockAnalytics = MockAnalyticsService()
        sut = ProductListViewModel(
            productRepository: mockRepository,
            analyticsService: mockAnalytics
        )
    }
    
    func testLoadInitial_setsProducts() async {
        // Arrange
        mockRepository.productsToReturn = [
            Product.mock(id: "1", name: "iPhone"),
            Product.mock(id: "2", name: "iPad")
        ]
        
        // Act
        sut.loadInitial()
        
        // รอ async operation
        await Task.yield()
        
        // Assert
        XCTAssertEqual(sut.products.count, 2)
        XCTAssertFalse(sut.isLoading)
    }
    
    func testLoadInitial_onError_setsErrorMessage() async {
        // Arrange
        mockRepository.shouldThrowError = AppError.networkError(
            underlying: URLError(.notConnectedToInternet),
            endpoint: "products"
        )
        
        // Act
        sut.loadInitial()
        await Task.yield()
        
        // Assert
        XCTAssertNotNil(sut.error)
        XCTAssertTrue(sut.products.isEmpty)
    }
}
```

---

## 12. Core Data กับ Combine

### 12.1 NSFetchedResultsController กับ Combine

```swift
// ProductFetchedResultsController.swift
import CoreData
import Combine

class ProductFetchedResultsPublisher<T: NSManagedObject>: NSObject,
    NSFetchedResultsControllerDelegate,
    Publisher {
    
    typealias Output = [T]
    typealias Failure = Error
    
    private let controller: NSFetchedResultsController<T>
    private var subscriber: AnySubscriber<[T], Error>?
    
    init(
        request: NSFetchRequest<T>,
        context: NSManagedObjectContext,
        sectionNameKeyPath: String? = nil,
        cacheName: String? = nil
    ) {
        self.controller = NSFetchedResultsController(
            fetchRequest: request,
            managedObjectContext: context,
            sectionNameKeyPath: sectionNameKeyPath,
            cacheName: cacheName
        )
        super.init()
        controller.delegate = self
    }
    
    func receive<S: Subscriber>(subscriber: S)
        where S.Input == [T], S.Failure == Error {
        self.subscriber = AnySubscriber(subscriber)
        
        do {
            try controller.performFetch()
            let objects = controller.fetchedObjects ?? []
            _ = subscriber.receive(objects)
        } catch {
            subscriber.receive(completion: .failure(error))
        }
    }
    
    // NSFetchedResultsControllerDelegate
    func controllerDidChangeContent(_ controller: NSFetchedResultsController<NSFetchRequestResult>) {
        let objects = (controller.fetchedObjects as? [T]) ?? []
        _ = subscriber?.receive(objects)
    }
}

// Extension สำหรับใช้งาน
extension NSManagedObjectContext {
    func publisher<T: NSManagedObject>(for request: NSFetchRequest<T>) -> ProductFetchedResultsPublisher<T> {
        ProductFetchedResultsPublisher(request: request, context: self)
    }
}

// ViewModel ที่ใช้ Combine
@MainActor
class ProductListCombineViewModel: ObservableObject {
    @Published var products: [ProductEntity] = []
    
    private var cancellables = Set<AnyCancellable>()
    private let context = CoreDataStack.shared.viewContext
    
    init() {
        setupProductsSubscription()
    }
    
    private func setupProductsSubscription() {
        let request = ProductEntity.fetchRequest()
        request.sortDescriptors = [
            NSSortDescriptor(keyPath: \ProductEntity.updatedAt, ascending: false)
        ]
        
        context.publisher(for: request)
            .receive(on: DispatchQueue.main)
            .catch { error -> Just<[ProductEntity]> in
                ErrorReporter.shared.report(.unexpectedError(message: error.localizedDescription))
                return Just([])
            }
            .assign(to: &$products)
    }
    
    // Filter with Combine
    func setupFilteredProducts(categoryId: String) {
        let request = ProductEntity.fetchRequest()
        request.predicate = NSPredicate(format: "categoryId == %@", categoryId)
        request.sortDescriptors = [
            NSSortDescriptor(keyPath: \ProductEntity.name, ascending: true)
        ]
        
        context.publisher(for: request)
            .receive(on: DispatchQueue.main)
            .assign(to: &$products)
    }
}

// Using FetchedResults + Combine Publisher
struct ProductsView: View {
    @FetchRequest(
        sortDescriptors: [SortDescriptor(\ProductEntity.updatedAt, order: .reverse)],
        animation: .default
    )
    var products: FetchedResults<ProductEntity>
    
    var body: some View {
        List(products) { product in
            ProductRow(product: product)
        }
    }
}
```

---

## 13. Core Data กับ Swift Concurrency

### 13.1 Async Core Data Operations

```swift
// AsyncCoreDataOperations.swift
import CoreData

// Extension สำหรับ Async/Await
extension NSManagedObjectContext {
    
    // Async perform
    func performAsync<T>(_ block: @escaping () throws -> T) async throws -> T {
        try await withCheckedThrowingContinuation { continuation in
            self.perform {
                do {
                    let result = try block()
                    continuation.resume(returning: result)
                } catch {
                    continuation.resume(throwing: error)
                }
            }
        }
    }
    
    // Save async
    func saveAsync() async throws {
        try await performAsync {
            guard self.hasChanges else { return }
            try self.save()
        }
    }
}

// Actor-based Core Data Access
actor CoreDataActor {
    private let context: NSManagedObjectContext
    
    init(context: NSManagedObjectContext) {
        self.context = context
    }
    
    func fetchProducts(predicate: NSPredicate? = nil) throws -> [ProductEntity] {
        let request = ProductEntity.fetchRequest()
        request.predicate = predicate
        return try context.fetch(request)
    }
    
    func insertProduct(id: String, name: String, price: Double) throws -> NSManagedObjectID {
        let product = ProductEntity(context: context)
        product.id = id
        product.name = name
        product.price = price
        try context.save()
        return product.objectID
    }
    
    func deleteProduct(id: String) throws {
        let request = ProductEntity.fetchRequest()
        request.predicate = NSPredicate(format: "id == %@", id)
        request.fetchLimit = 1
        
        guard let product = try context.fetch(request).first else { return }
        context.delete(product)
        try context.save()
    }
}

// Repository ที่ใช้ async/await
class AsyncProductRepository {
    private let stack = CoreDataStack.shared
    
    // Fetch แบบ Async
    func fetchProducts(
        category: String? = nil,
        page: Int = 1,
        pageSize: Int = 20
    ) async throws -> PaginatedResult<Product> {
        
        return try await stack.persistentContainer.performBackgroundTask { context in
            let request = ProductEntity.fetchRequest()
            
            if let category = category {
                request.predicate = NSPredicate(format: "categoryId == %@", category)
            }
            
            // Count for pagination
            let totalCount = try context.count(for: request)
            
            // Apply pagination
            request.fetchLimit = pageSize
            request.fetchOffset = (page - 1) * pageSize
            request.sortDescriptors = [
                NSSortDescriptor(keyPath: \ProductEntity.updatedAt, ascending: false)
            ]
            
            let entities = try context.fetch(request)
            let products = entities.compactMap { $0.toDomain() }
            
            return PaginatedResult(
                items: products,
                currentPage: page,
                totalPages: Int(ceil(Double(totalCount) / Double(pageSize))),
                totalItems: totalCount
            )
        }
    }
    
    // Save async
    func saveProduct(_ product: Product) async throws {
        try await stack.persistentContainer.performBackgroundTask { context in
            let request = ProductEntity.fetchRequest()
            request.predicate = NSPredicate(format: "id == %@", product.id)
            request.fetchLimit = 1
            
            let entity = (try context.fetch(request).first) ?? ProductEntity(context: context)
            entity.update(from: product)
            
            try context.save()
        }
    }
    
    // Delete async
    func deleteProduct(id: String) async throws {
        try await stack.persistentContainer.performBackgroundTask { context in
            let request = ProductEntity.fetchRequest()
            request.predicate = NSPredicate(format: "id == %@", id)
            request.fetchLimit = 1
            
            guard let entity = try context.fetch(request).first else { return }
            context.delete(entity)
            try context.save()
        }
    }
}

// Task Group สำหรับ Parallel Operations
class ParallelDataLoader {
    func loadDashboardData() async throws -> DashboardData {
        async let products = loadProducts()
        async let categories = loadCategories()
        async let featuredBanners = loadBanners()
        async let recentOrders = loadRecentOrders()
        
        // รอทุก operation พร้อมกัน
        let (p, c, b, o) = try await (products, categories, featuredBanners, recentOrders)
        
        return DashboardData(
            products: p,
            categories: c,
            banners: b,
            recentOrders: o
        )
    }
    
    private func loadProducts() async throws -> [Product] { [] }
    private func loadCategories() async throws -> [Category] { [] }
    private func loadBanners() async throws -> [Banner] { [] }
    private func loadRecentOrders() async throws -> [Order] { [] }
}
```

---

## 14. Core Data Entity Extensions

### 14.1 Mapping Entity <-> Domain Model

```swift
// ProductEntity+Extensions.swift
import CoreData

extension ProductEntity {
    
    // Entity -> Domain Model
    func toDomain() -> Product? {
        guard let id = id,
              let name = name else { return nil }
        
        return Product(
            id: id,
            name: name,
            description: productDescription ?? "",
            price: price,
            compareAtPrice: compareAtPrice > 0 ? compareAtPrice : nil,
            images: (images?.allObjects as? [ProductImageEntity])?.compactMap { $0.toDomain() } ?? [],
            category: category?.toDomain() ?? .unknown,
            tags: tags?.components(separatedBy: ",") ?? [],
            variants: (variants?.allObjects as? [ProductVariantEntity])?.compactMap { $0.toDomain() } ?? [],
            inventory: Product.Inventory(
                quantity: Int(inventoryQuantity),
                isAvailable: inventoryQuantity > 0,
                trackQuantity: trackInventory
            ),
            rating: Product.Rating(
                average: ratingAverage,
                count: Int(ratingCount)
            ),
            isNew: isNew,
            isFeatured: isFeatured
        )
    }
    
    // Domain Model -> Entity
    func update(from product: Product) {
        id = product.id
        name = product.name
        productDescription = product.description
        price = product.price
        compareAtPrice = product.compareAtPrice ?? 0
        tags = product.tags.joined(separator: ",")
        isFeatured = product.isFeatured
        isNew = product.isNew
        inventoryQuantity = Int32(product.inventory.quantity)
        trackInventory = product.inventory.trackQuantity
        ratingAverage = product.rating.average
        ratingCount = Int32(product.rating.count)
        updatedAt = Date()
    }
    
    // Static Factory
    static func findOrCreate(
        id: String,
        in context: NSManagedObjectContext
    ) -> ProductEntity {
        let request = fetchRequest()
        request.predicate = NSPredicate(format: "id == %@", id)
        request.fetchLimit = 1
        
        if let existing = (try? context.fetch(request))?.first {
            return existing
        }
        
        let new = ProductEntity(context: context)
        new.id = id
        new.createdAt = Date()
        return new
    }
}
```

---

## 15. Practical Exercises with Complete Solutions

### แบบฝึกหัดที่ 1: Implement Shopping Cart Persistence

```swift
// Exercise 1: Cart Persistence with Core Data

// CartItemEntity สร้างใน Data Model:
// - id: String
// - productId: String
// - productName: String
// - productPrice: Double
// - quantity: Int32
// - variantId: String (optional)
// - addedAt: Date

class CartPersistenceManager {
    static let shared = CartPersistenceManager()
    private let stack = CoreDataStack.shared
    
    // ✅ Solution: Save cart items
    func saveCartItems(_ items: [CartItem]) async throws {
        try await stack.persistentContainer.performBackgroundTask { context in
            // ลบ items เก่าทั้งหมดก่อน
            let deleteRequest = NSBatchDeleteRequest(
                fetchRequest: CartItemEntity.fetchRequest() as! NSFetchRequest<NSFetchRequestResult>
            )
            try context.execute(deleteRequest)
            
            // Insert items ใหม่
            for item in items {
                let entity = CartItemEntity(context: context)
                entity.id = item.id
                entity.productId = item.product.id
                entity.productName = item.product.name
                entity.productPrice = item.product.price
                entity.quantity = Int32(item.quantity)
                entity.variantId = item.variant?.id
                entity.addedAt = item.addedAt
            }
            
            try context.save()
        }
    }
    
    // ✅ Solution: Load cart items
    func loadCartItems() async throws -> [CartItemData] {
        return try await stack.persistentContainer.performBackgroundTask { context in
            let request = CartItemEntity.fetchRequest()
            request.sortDescriptors = [
                NSSortDescriptor(keyPath: \CartItemEntity.addedAt, ascending: true)
            ]
            
            let entities = try context.fetch(request)
            return entities.map { entity in
                CartItemData(
                    id: entity.id ?? UUID().uuidString,
                    productId: entity.productId ?? "",
                    productName: entity.productName ?? "",
                    productPrice: entity.productPrice,
                    quantity: Int(entity.quantity),
                    variantId: entity.variantId,
                    addedAt: entity.addedAt ?? Date()
                )
            }
        }
    }
}
```

### แบบฝึกหัดที่ 2: Implement Search History

```swift
// Exercise 2: Search History

class SearchHistoryManager {
    private let maxHistory = 20
    private let stack = CoreDataStack.shared
    
    func saveSearch(_ query: String) async throws {
        guard !query.trimmingCharacters(in: .whitespaces).isEmpty else { return }
        
        try await stack.persistentContainer.performBackgroundTask { context in
            // ตรวจสอบว่ามีอยู่แล้วหรือไม่
            let request = SearchHistoryEntity.fetchRequest()
            request.predicate = NSPredicate(format: "query ==[cd] %@", query)
            request.fetchLimit = 1
            
            let existing = try context.fetch(request).first
            
            if let entity = existing {
                // อัปเดต timestamp ถ้ามีอยู่แล้ว
                entity.searchedAt = Date()
                entity.count += 1
            } else {
                // สร้างใหม่
                let entity = SearchHistoryEntity(context: context)
                entity.id = UUID().uuidString
                entity.query = query
                entity.searchedAt = Date()
                entity.count = 1
            }
            
            try context.save()
            
            // จำกัด History ไม่เกิน maxHistory
            try self.pruneOldHistory(in: context)
        }
    }
    
    private func pruneOldHistory(in context: NSManagedObjectContext) throws {
        let countRequest = SearchHistoryEntity.fetchRequest()
        let count = try context.count(for: countRequest)
        
        guard count > maxHistory else { return }
        
        let deleteRequest = NSFetchRequest<NSFetchRequestResult>(entityName: "SearchHistoryEntity")
        deleteRequest.sortDescriptors = [
            NSSortDescriptor(keyPath: \SearchHistoryEntity.searchedAt, ascending: true)
        ]
        deleteRequest.fetchLimit = count - maxHistory
        
        let idsToDelete = try (context.fetch(deleteRequest) as! [SearchHistoryEntity])
            .map { $0.objectID }
        
        idsToDelete.forEach { context.delete(context.object(with: $0)) }
        try context.save()
    }
    
    func loadSearchHistory() async throws -> [SearchHistoryItem] {
        return try await stack.persistentContainer.performBackgroundTask { context in
            let request = SearchHistoryEntity.fetchRequest()
            request.sortDescriptors = [
                NSSortDescriptor(keyPath: \SearchHistoryEntity.searchedAt, ascending: false)
            ]
            request.fetchLimit = self.maxHistory
            
            let entities = try context.fetch(request)
            return entities.map { entity in
                SearchHistoryItem(
                    id: entity.id ?? UUID().uuidString,
                    query: entity.query ?? "",
                    searchedAt: entity.searchedAt ?? Date(),
                    count: Int(entity.count)
                )
            }
        }
    }
    
    func clearHistory() async throws {
        try await stack.persistentContainer.performBackgroundTask { context in
            let request = NSFetchRequest<NSFetchRequestResult>(entityName: "SearchHistoryEntity")
            let deleteRequest = NSBatchDeleteRequest(fetchRequest: request)
            try context.execute(deleteRequest)
        }
    }
}
```

### แบบฝึกหัดที่ 3: Sync Manager กับ Persistent History

```swift
// Exercise 3: Offline Sync with Persistent History

class SyncManager {
    static let shared = SyncManager()
    
    private let stack = CoreDataStack.shared
    private let apiClient: APIClient
    private let historyManager = PersistentHistoryManager.shared
    
    func syncPendingChanges() async throws {
        // 1. ดึง Changes ตั้งแต่ครั้งที่แล้ว
        let transactions = try await historyManager.fetchHistory(
            for: "CartItemEntity",
            since: lastSyncDate
        )
        
        guard !transactions.isEmpty else {
            Logger.debug("No pending changes to sync")
            return
        }
        
        // 2. แปลงเป็น Sync Operations
        var operations: [SyncOperation] = []
        
        for transaction in transactions {
            guard let changes = transaction.changes else { continue }
            for change in changes {
                switch change.changeType {
                case .insert:
                    operations.append(.create(objectID: change.changedObjectID))
                case .update:
                    operations.append(.update(
                        objectID: change.changedObjectID,
                        changedProperties: change.updatedProperties?.map { $0.name } ?? []
                    ))
                case .delete:
                    // ต้องเก็บ ID ก่อนลบ - ใช้ tombstone
                    if let tombstoneId = getTombstoneId(for: change.changedObjectID) {
                        operations.append(.delete(id: tombstoneId))
                    }
                @unknown default:
                    break
                }
            }
        }
        
        // 3. ส่ง Operations ไปยัง Server
        try await apiClient.syncOperations(operations)
        
        // 4. Update last sync date
        lastSyncDate = Date()
    }
    
    private var lastSyncDate: Date {
        get { UserDefaults.standard.object(forKey: "last_sync_date") as? Date ?? .distantPast }
        set { UserDefaults.standard.set(newValue, forKey: "last_sync_date") }
    }
    
    private func getTombstoneId(for objectID: NSManagedObjectID) -> String? {
        // ดึง tombstone จาก separate store
        return TombstoneStore.shared.getId(for: objectID.uriRepresentation())
    }
    
    enum SyncOperation {
        case create(objectID: NSManagedObjectID)
        case update(objectID: NSManagedObjectID, changedProperties: [String])
        case delete(id: String)
    }
}
```

---

## 16. Performance Profiling และ Debugging

### 16.1 Core Data Debug Flags

```bash
# ใน Xcode Scheme > Run > Arguments Passed On Launch
-com.apple.CoreData.SQLDebug 1          # แสดง SQL queries
-com.apple.CoreData.SQLDebug 3          # แสดง SQL + Statistics
-com.apple.CoreData.Logging.stderr 1    # Log errors
-com.apple.CoreData.ConcurrencyDebug 1  # Detect concurrency violations
-com.apple.CoreData.MigrationDebug 1    # Migration debug
```

### 16.2 Performance Monitoring

```swift
// CoreDataPerformanceMonitor.swift
import CoreData
import os.signpost

class CoreDataPerformanceMonitor {
    static let shared = CoreDataPerformanceMonitor()
    
    private let log = OSLog(subsystem: "com.shopswift.app", category: "CoreData")
    
    func monitorFetch<T: NSManagedObject>(
        _ request: NSFetchRequest<T>,
        in context: NSManagedObjectContext
    ) throws -> [T] {
        let signpostID = OSSignpostID(log: log)
        os_signpost(.begin, log: log, name: "CoreData Fetch",
                    signpostID: signpostID,
                    "Fetching %{public}@", request.entityName ?? "Unknown")
        
        defer {
            os_signpost(.end, log: log, name: "CoreData Fetch",
                       signpostID: signpostID)
        }
        
        let start = Date()
        let results = try context.fetch(request)
        let duration = Date().timeIntervalSince(start)
        
        if duration > 0.1 {  // 100ms threshold
            Logger.warning("Slow fetch: \(request.entityName ?? "") took \(duration * 1000)ms, returned \(results.count) objects")
        }
        
        return results
    }
    
    // ตรวจสอบ N+1 Problem
    func detectNPlusOneQueries(in context: NSManagedObjectContext) {
        // Monitor ด้วย SQLDebug flag
        // ใน Debug build: จับตาดู repetitive queries
    }
}
```

---

## 17. Best Practices สรุป

### 17.1 DOs and DON'Ts

```swift
// ✅ DO: ใช้ Background Context สำหรับการ Import
func importData() async throws {
    let bgContext = CoreDataStack.shared.backgroundContext
    try await bgContext.perform {
        // Heavy work here
        try bgContext.save()
    }
}

// ❌ DON'T: ทำงานหนักใน View Context
func badImport() throws {
    let viewContext = CoreDataStack.shared.viewContext  // UI Thread!
    for item in 10000Items {
        let entity = Entity(context: viewContext)
        // ... ทำให้ UI ค้าง
    }
    try viewContext.save()
}

// ✅ DO: ใช้ NSBatchDeleteRequest แทน Loop Delete
func deleteOldItems() async throws {
    let request = NSFetchRequest<NSFetchRequestResult>(entityName: "Item")
    request.predicate = NSPredicate(format: "createdAt < %@", cutoffDate as NSDate)
    let deleteRequest = NSBatchDeleteRequest(fetchRequest: request)
    try CoreDataStack.shared.backgroundContext.execute(deleteRequest)
}

// ❌ DON'T: Loop แล้วลบทีละตัว
func badDelete() throws {
    let items = try fetchOldItems()
    items.forEach { viewContext.delete($0) }  // โหลดทุก Object เข้า Memory
    try viewContext.save()
}

// ✅ DO: ใช้ Predicate ที่ใช้ Indexed Attribute
request.predicate = NSPredicate(format: "id == %@", productId)  // id มี Index

// ❌ DON'T: ใช้ Predicate ที่ไม่มี Index
request.predicate = NSPredicate(format: "description CONTAINS %@", text)  // Slow!

// ✅ DO: ตั้ง fetchLimit เสมอ
request.fetchLimit = 50

// ❌ DON'T: Fetch ทั้งหมดโดยไม่จำกัด
let allProducts = try context.fetch(request)  // อาจ OOM!
```

---

## 18. Summary

ในบทนี้เราได้เรียนรู้ Core Data ขั้นสูงอย่างครบถ้วน:

### สิ่งที่ได้เรียนรู้

1. **Performance Optimization**
   - Fetch Request ที่ดีใช้ Indexed Attributes, fetchLimit, propertiesToFetch
   - returnsObjectsAsFaults ลด Memory
   - Prefetch Relationships ลด N+1 Problem

2. **Batch Operations**
   - `NSBatchUpdateRequest` อัปเดตหลาย Records โดยไม่โหลดเข้า Memory
   - `NSBatchDeleteRequest` ลบอย่างมีประสิทธิภาพ
   - `NSBatchInsertRequest` Import จำนวนมากได้เร็ว 10x

3. **Persistent History Tracking**
   - ติดตามการเปลี่ยนแปลงระหว่าง App Extension กับ Main App
   - Token ช่วยให้รู้ว่า Process ถึงไหนแล้ว
   - Remote Change Notifications แจ้งเมื่อ CloudKit Sync

4. **Multi-Context Architecture**
   - View Context สำหรับ UI
   - Background Context สำหรับ Heavy Operations
   - Parent-Child Contexts สำหรับ Edit-Cancel Pattern

5. **Migration Strategies**
   - Lightweight Migration สำหรับการเปลี่ยนแปลงง่าย ๆ
   - Custom Migration Policy สำหรับ Data Transformation
   - Progressive Migration สำหรับหลาย Version

6. **Testing**
   - In-Memory Store เร็วและไม่กระทบ Production Data
   - Mock Repository สำหรับ Unit Test ViewModel
   - Protocol-based Design ทำให้ Test ง่าย

7. **Combine & Concurrency**
   - `NSFetchedResultsController` + Combine Publisher
   - Actor-based Core Data Access
   - `performBackgroundTask` สำหรับ Async Operations

### Performance Checklist

- [ ] Indexed Attributes ที่ใช้ใน Predicate บ่อย
- [ ] fetchLimit ทุก Fetch Request
- [ ] Batch Operations สำหรับ Mass Updates/Deletes
- [ ] Background Context สำหรับ Import
- [ ] Prefetch Relationships ที่ใช้บ่อย
- [ ] context.reset() หลัง Batch Import
- [ ] undoManager = nil ใน Background Context
- [ ] Persistent History Tracking สำหรับ App Groups

---

*จบ Part 78: Advanced Core Data*

*บทต่อไป: Part 79 - Advanced UIKit Patterns*
