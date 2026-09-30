# Part 32: Core Data Basics ใน Swift

## สารบัญ

1. [Core Data คืออะไร](#core-data-คืออะไร)
2. [Core Data Stack](#core-data-stack)
3. [การสร้าง Data Model](#การสร้าง-data-model)
4. [Entities และ Attributes](#entities-และ-attributes)
5. [Relationships](#relationships)
6. [NSManagedObject Subclasses](#nsmanagedobject-subclasses)
7. [@NSManaged Property Wrapper](#nsmanaged-property-wrapper)
8. [Saving Context](#saving-context)
9. [Fetching Data](#fetching-data)
10. [Predicates (NSPredicate)](#predicates-nspredicate)
11. [Sort Descriptors](#sort-descriptors)
12. [Creating, Updating, Deleting Records](#creating-updating-deleting-records)
13. [Background Context](#background-context)
14. [Merge Policies](#merge-policies)
15. [Versioning and Migration](#versioning-and-migration)
16. [Lightweight Migration](#lightweight-migration)
17. [Core Data กับ SwiftUI](#core-data-กับ-swiftui)
18. [NSFetchedResultsController](#nsfetchedresultscontroller)
19. [CloudKit Sync กับ Core Data](#cloudkit-sync-กับ-core-data)
20. [Practical Exercises](#practical-exercises)
21. [Building a Notes App](#building-a-notes-app)
22. [Summary](#summary)

---

## Core Data คืออะไร

**Core Data** เป็น framework ของ Apple สำหรับจัดการ object graph และ persistence ของข้อมูลในแอปพลิเคชัน iOS, macOS, watchOS, และ tvOS

### Core Data ไม่ใช่ Database

ความเข้าใจผิดที่พบบ่อยคือคิดว่า Core Data คือ database จริงๆ แล้ว Core Data คือ **Object-Relational Mapping (ORM) framework** ที่ใช้ SQLite (หรือ XML, Binary) เป็น backing store

```
Core Data Stack:
┌─────────────────────────────────────────┐
│         Your Application Code           │
│  (NSManagedObject, NSFetchRequest, etc) │
├─────────────────────────────────────────┤
│       NSPersistentContainer             │
│  ┌─────────────────────────────────┐   │
│  │   NSManagedObjectContext        │   │
│  │   (Main Context / Background)   │   │
│  └─────────────────────────────────┘   │
│  ┌─────────────────────────────────┐   │
│  │   NSPersistentStoreCoordinator  │   │
│  └─────────────────────────────────┘   │
│  ┌─────────────────────────────────┐   │
│  │   NSManagedObjectModel          │   │
│  │   (.xcdatamodeld file)          │   │
│  └─────────────────────────────────┘   │
├─────────────────────────────────────────┤
│       Persistent Store                  │
│    (SQLite / XML / Binary / In-Memory)  │
└─────────────────────────────────────────┘
```

### ประโยชน์ของ Core Data

1. **Object Graph Management**: จัดการ relationships ระหว่าง objects อัตโนมัติ
2. **Data Persistence**: บันทึกข้อมูลลง disk
3. **Lazy Loading**: โหลดข้อมูลเมื่อจำเป็นเท่านั้น
4. **Change Tracking**: track การเปลี่ยนแปลงของข้อมูลอัตโนมัติ
5. **Undo/Redo Support**: รองรับ undo/redo
6. **Validation**: validate ข้อมูลก่อน save
7. **Memory Management**: จัดการ memory อัตโนมัติ

### เมื่อใช้ Core Data

```
ใช้ Core Data เมื่อ:
✅ มีข้อมูลจำนวนมากที่ต้อง persist
✅ มี relationships ซับซ้อนระหว่าง objects
✅ ต้องการ query/filter/sort ข้อมูลได้
✅ ต้องการ undo/redo
✅ ต้องการ sync กับ CloudKit

ไม่ต้องใช้ Core Data เมื่อ:
❌ ข้อมูลน้อยมาก (ใช้ UserDefaults แทน)
❌ ข้อมูลเป็น key-value simple (ใช้ UserDefaults)
❌ เพียงแค่ต้องการ cache JSON (ใช้ file system)
❌ ต้องการ share กับ non-Apple platforms
```

---

## Core Data Stack

Core Data Stack ประกอบด้วย 3 component หลัก

### NSPersistentContainer

`NSPersistentContainer` เป็น class ที่ encapsulate Core Data stack ทั้งหมด (เพิ่มใน iOS 10)

```swift
import CoreData

class PersistenceController {
    
    // Singleton pattern
    static let shared = PersistenceController()
    
    // NSPersistentContainer เป็น entry point หลัก
    let container: NSPersistentContainer
    
    init(inMemory: Bool = false) {
        // ชื่อต้องตรงกับชื่อไฟล์ .xcdatamodeld
        container = NSPersistentContainer(name: "MyApp")
        
        if inMemory {
            // ใช้ in-memory store สำหรับ testing
            container.persistentStoreDescriptions.first!.url = URL(fileURLWithPath: "/dev/null")
        }
        
        container.loadPersistentStores { description, error in
            if let error = error {
                // ใน production ควร handle error อย่างเหมาะสม
                fatalError("Unable to load persistent stores: \(error)")
            }
        }
        
        // Configuration
        container.viewContext.automaticallyMergesChangesFromParent = true
        container.viewContext.mergePolicy = NSMergeByPropertyObjectTrumpMergePolicy
    }
    
    // Convenience accessor สำหรับ main context
    var viewContext: NSManagedObjectContext {
        return container.viewContext
    }
    
    // สร้าง background context สำหรับ heavy operations
    func newBackgroundContext() -> NSManagedObjectContext {
        return container.newBackgroundContext()
    }
}

// Preview สำหรับ SwiftUI
extension PersistenceController {
    static var preview: PersistenceController = {
        let controller = PersistenceController(inMemory: true)
        // เพิ่ม sample data สำหรับ preview
        let context = controller.viewContext
        
        for i in 1...5 {
            let note = Note(context: context)
            note.title = "Note \(i)"
            note.content = "Content of note \(i)"
            note.createdAt = Date()
        }
        
        try? context.save()
        return controller
    }()
}
```

### NSManagedObjectContext

`NSManagedObjectContext` (MOC) เป็น "workspace" สำหรับทำงานกับ managed objects

```swift
import CoreData

class ContextManager {
    
    let persistentContainer: NSPersistentContainer
    
    init(container: NSPersistentContainer) {
        self.persistentContainer = container
    }
    
    // Main Queue Context (UI thread)
    var mainContext: NSManagedObjectContext {
        return persistentContainer.viewContext
    }
    
    // Background Context สำหรับ heavy tasks
    var backgroundContext: NSManagedObjectContext {
        return persistentContainer.newBackgroundContext()
    }
    
    // Perform operation บน background context
    func performInBackground(_ block: @escaping (NSManagedObjectContext) -> Void) {
        let context = backgroundContext
        context.perform {
            block(context)
        }
    }
    
    // Perform operation และ save
    func performAndSave(_ block: @escaping (NSManagedObjectContext) throws -> Void) throws {
        let context = backgroundContext
        var thrownError: Error?
        
        context.performAndWait {
            do {
                try block(context)
                if context.hasChanges {
                    try context.save()
                }
            } catch {
                thrownError = error
            }
        }
        
        if let error = thrownError {
            throw error
        }
    }
}
```

### NSManagedObjectModel

`NSManagedObjectModel` คือ schema ของ Core Data (ถูก define ใน .xcdatamodeld file)

```swift
import CoreData

// โดยปกติ NSManagedObjectModel จะถูกโหลดอัตโนมัติจาก .xcdatamodeld
// แต่สามารถสร้างด้วย code ได้

func createModelProgrammatically() -> NSManagedObjectModel {
    let model = NSManagedObjectModel()
    
    // สร้าง Entity "Person"
    let personEntity = NSEntityDescription()
    personEntity.name = "Person"
    personEntity.managedObjectClassName = "Person"
    
    // สร้าง Attributes
    let nameAttribute = NSAttributeDescription()
    nameAttribute.name = "name"
    nameAttribute.attributeType = .stringAttributeType
    nameAttribute.isOptional = false
    
    let ageAttribute = NSAttributeDescription()
    ageAttribute.name = "age"
    ageAttribute.attributeType = .integer16AttributeType
    ageAttribute.isOptional = false
    ageAttribute.defaultValue = 0
    
    let emailAttribute = NSAttributeDescription()
    emailAttribute.name = "email"
    emailAttribute.attributeType = .stringAttributeType
    emailAttribute.isOptional = true
    
    personEntity.properties = [nameAttribute, ageAttribute, emailAttribute]
    model.entities = [personEntity]
    
    return model
}
```

### NSPersistentStoreCoordinator

`NSPersistentStoreCoordinator` เชื่อมต่อระหว่าง NSManagedObjectModel และ persistent store

```swift
import CoreData

// NSPersistentStoreCoordinator ถูกจัดการโดย NSPersistentContainer อัตโนมัติ
// แต่สามารถ access ได้ถ้าต้องการ

class StoreManager {
    
    let container: NSPersistentContainer
    
    init(container: NSPersistentContainer) {
        self.container = container
    }
    
    // Access coordinator
    var coordinator: NSPersistentStoreCoordinator {
        return container.persistentStoreCoordinator
    }
    
    // ดู persistent stores ที่โหลดอยู่
    func printStoreInfo() {
        for store in coordinator.persistentStores {
            print("Store URL: \(store.url?.absoluteString ?? "in-memory")")
            print("Store Type: \(store.type)")
            print("Store Options: \(store.options ?? [:])")
        }
    }
    
    // เพิ่ม store ชนิด SQLite
    func addSQLiteStore(at url: URL) throws {
        let options: [String: Any] = [
            NSMigratePersistentStoresAutomaticallyOption: true,
            NSInferMappingModelAutomaticallyOption: true
        ]
        
        try coordinator.addPersistentStore(
            ofType: NSSQLiteStoreType,
            configurationName: nil,
            at: url,
            options: options
        )
    }
}
```

---

## การสร้าง Data Model

ใน Xcode สร้าง Data Model ผ่าน File → New → File → Core Data → Data Model

### โครงสร้าง .xcdatamodeld File

```
MyApp.xcdatamodeld/
├── MyApp.xcdatamodel/
│   ├── contents          (XML ที่ describe entities)
│   └── .xccurrentversion (version ปัจจุบัน)
```

### การสร้าง Entity ด้วย Code (Programmatic)

```swift
import CoreData

// สร้าง Data Model สมบูรณ์แบบ programmatic
func buildNotesDataModel() -> NSManagedObjectModel {
    let model = NSManagedObjectModel()
    
    // =============================
    // Entity: Note
    // =============================
    let noteEntity = NSEntityDescription()
    noteEntity.name = "Note"
    noteEntity.managedObjectClassName = "Note"
    
    // Attributes
    let noteIDAttr = NSAttributeDescription()
    noteIDAttr.name = "id"
    noteIDAttr.attributeType = .UUIDAttributeType
    noteIDAttr.isOptional = false
    
    let noteTitleAttr = NSAttributeDescription()
    noteTitleAttr.name = "title"
    noteTitleAttr.attributeType = .stringAttributeType
    noteTitleAttr.isOptional = false
    noteTitleAttr.defaultValue = ""
    
    let noteContentAttr = NSAttributeDescription()
    noteContentAttr.name = "content"
    noteContentAttr.attributeType = .stringAttributeType
    noteContentAttr.isOptional = true
    
    let noteCreatedAtAttr = NSAttributeDescription()
    noteCreatedAtAttr.name = "createdAt"
    noteCreatedAtAttr.attributeType = .dateAttributeType
    noteCreatedAtAttr.isOptional = false
    
    let noteUpdatedAtAttr = NSAttributeDescription()
    noteUpdatedAtAttr.name = "updatedAt"
    noteUpdatedAtAttr.attributeType = .dateAttributeType
    noteUpdatedAtAttr.isOptional = true
    
    let noteIsPinnedAttr = NSAttributeDescription()
    noteIsPinnedAttr.name = "isPinned"
    noteIsPinnedAttr.attributeType = .booleanAttributeType
    noteIsPinnedAttr.isOptional = false
    noteIsPinnedAttr.defaultValue = false
    
    noteEntity.properties = [
        noteIDAttr, noteTitleAttr, noteContentAttr,
        noteCreatedAtAttr, noteUpdatedAtAttr, noteIsPinnedAttr
    ]
    
    // =============================
    // Entity: Tag
    // =============================
    let tagEntity = NSEntityDescription()
    tagEntity.name = "Tag"
    tagEntity.managedObjectClassName = "Tag"
    
    let tagNameAttr = NSAttributeDescription()
    tagNameAttr.name = "name"
    tagNameAttr.attributeType = .stringAttributeType
    tagNameAttr.isOptional = false
    
    let tagColorAttr = NSAttributeDescription()
    tagColorAttr.name = "color"
    tagColorAttr.attributeType = .stringAttributeType
    tagColorAttr.isOptional = true
    
    tagEntity.properties = [tagNameAttr, tagColorAttr]
    
    // =============================
    // Relationships
    // =============================
    
    // Note -> Tags (many-to-many)
    let noteToTagsRelationship = NSRelationshipDescription()
    noteToTagsRelationship.name = "tags"
    noteToTagsRelationship.destinationEntity = tagEntity
    noteToTagsRelationship.minCount = 0
    noteToTagsRelationship.maxCount = 0  // 0 = unlimited (many)
    noteToTagsRelationship.deleteRule = .nullifyDeleteRule
    
    // Tag -> Notes (many-to-many inverse)
    let tagToNotesRelationship = NSRelationshipDescription()
    tagToNotesRelationship.name = "notes"
    tagToNotesRelationship.destinationEntity = noteEntity
    tagToNotesRelationship.minCount = 0
    tagToNotesRelationship.maxCount = 0
    tagToNotesRelationship.deleteRule = .nullifyDeleteRule
    
    // เชื่อม inverse relationships
    noteToTagsRelationship.inverseRelationship = tagToNotesRelationship
    tagToNotesRelationship.inverseRelationship = noteToTagsRelationship
    
    noteEntity.properties.append(noteToTagsRelationship)
    tagEntity.properties.append(tagToNotesRelationship)
    
    model.entities = [noteEntity, tagEntity]
    return model
}
```

---

## Entities และ Attributes

### Attribute Types ใน Core Data

```swift
import CoreData

/*
 Attribute Types ที่รองรับ:
 
 Numeric:
 - .integer16AttributeType    (Int16)
 - .integer32AttributeType    (Int32)
 - .integer64AttributeType    (Int64)
 - .decimalAttributeType      (NSDecimalNumber)
 - .doubleAttributeType       (Double)
 - .floatAttributeType        (Float)
 
 Text:
 - .stringAttributeType       (String)
 
 Boolean:
 - .booleanAttributeType      (Bool)
 
 Date/Time:
 - .dateAttributeType         (Date)
 
 Binary:
 - .binaryDataAttributeType   (Data)
 
 Identifier:
 - .UUIDAttributeType         (UUID) - iOS 11+
 - .URIAttributeType          (URL)
 
 Transformable:
 - .transformableAttributeType (Any NSCoding-compliant object)
 
 Undefined:
 - .undefinedAttributeType
 - .objectIDAttributeType
 */
```

### ตัวอย่าง Entity สมบูรณ์

```swift
import CoreData

// ใน Xcode Data Model Editor จะมี Entity ดังนี้:
// Entity: Student
// Attributes:
//   - studentID: UUID, required
//   - firstName: String, required
//   - lastName: String, required
//   - email: String, optional
//   - dateOfBirth: Date, optional
//   - gpa: Double, default 0.0
//   - isEnrolled: Boolean, default true
//   - profilePhoto: Binary Data, optional, Allows External Storage
//   - createdAt: Date, required

// NSManagedObject subclass ที่ generate
@objc(Student)
public class Student: NSManagedObject {
    
    @NSManaged public var studentID: UUID?
    @NSManaged public var firstName: String?
    @NSManaged public var lastName: String?
    @NSManaged public var email: String?
    @NSManaged public var dateOfBirth: Date?
    @NSManaged public var gpa: Double
    @NSManaged public var isEnrolled: Bool
    @NSManaged public var profilePhoto: Data?
    @NSManaged public var createdAt: Date?
    
    // Computed properties
    var fullName: String {
        let first = firstName ?? ""
        let last = lastName ?? ""
        return "\(first) \(last)".trimmingCharacters(in: .whitespaces)
    }
    
    var age: Int? {
        guard let dob = dateOfBirth else { return nil }
        let calendar = Calendar.current
        let components = calendar.dateComponents([.year], from: dob, to: Date())
        return components.year
    }
}

// Extension สำหรับ fetch request
extension Student {
    
    @nonobjc public class func fetchRequest() -> NSFetchRequest<Student> {
        return NSFetchRequest<Student>(entityName: "Student")
    }
    
    // Convenience factory method
    static func create(
        firstName: String,
        lastName: String,
        email: String? = nil,
        in context: NSManagedObjectContext
    ) -> Student {
        let student = Student(context: context)
        student.studentID = UUID()
        student.firstName = firstName
        student.lastName = lastName
        student.email = email
        student.gpa = 0.0
        student.isEnrolled = true
        student.createdAt = Date()
        return student
    }
}
```

### Validation ใน Core Data

```swift
import CoreData

@objc(Product)
public class Product: NSManagedObject {
    
    @NSManaged public var name: String?
    @NSManaged public var price: Double
    @NSManaged public var stock: Int32
    @NSManaged public var sku: String?
    
    // Custom validation
    public override func validateForInsert() throws {
        try super.validateForInsert()
        try validate()
    }
    
    public override func validateForUpdate() throws {
        try super.validateForUpdate()
        try validate()
    }
    
    private func validate() throws {
        guard let name = name, !name.isEmpty else {
            throw NSError(
                domain: "ProductError",
                code: 1001,
                userInfo: [NSLocalizedDescriptionKey: "Product name cannot be empty"]
            )
        }
        
        guard price >= 0 else {
            throw NSError(
                domain: "ProductError",
                code: 1002,
                userInfo: [NSLocalizedDescriptionKey: "Price must be non-negative"]
            )
        }
        
        guard stock >= 0 else {
            throw NSError(
                domain: "ProductError",
                code: 1003,
                userInfo: [NSLocalizedDescriptionKey: "Stock cannot be negative"]
            )
        }
    }
    
    // Validate ด้วย validateValue:forKey:
    public override func validateValue(_ value: AutoreleasingUnsafeMutablePointer<AnyObject?>, forKey key: String) throws {
        try super.validateValue(value, forKey: key)
        
        if key == "sku" {
            if let sku = value.pointee as? String {
                let pattern = "^[A-Z]{3}-\\d{6}$"
                let regex = try! NSRegularExpression(pattern: pattern)
                let range = NSRange(sku.startIndex..., in: sku)
                guard regex.firstMatch(in: sku, range: range) != nil else {
                    throw NSError(
                        domain: "ProductError",
                        code: 1004,
                        userInfo: [NSLocalizedDescriptionKey: "SKU must be in format ABC-123456"]
                    )
                }
            }
        }
    }
}
```

---

## Relationships

Core Data รองรับ relationship 3 ประเภท

### One-to-One Relationship

```swift
import CoreData

/*
 Employee ←→ Workstation (One-to-One)
 
 ใน Data Model Editor:
 Employee Entity:
   - Relationship "workstation": to Workstation, Optional, Inverse: "employee"
 
 Workstation Entity:
   - Relationship "employee": to Employee, Optional, Inverse: "workstation"
*/

@objc(Employee)
public class Employee: NSManagedObject {
    @NSManaged public var name: String?
    @NSManaged public var employeeID: String?
    @NSManaged public var workstation: Workstation?  // One-to-One
}

@objc(Workstation)
public class Workstation: NSManagedObject {
    @NSManaged public var computerModel: String?
    @NSManaged public var location: String?
    @NSManaged public var employee: Employee?  // Inverse relationship
}

// การใช้งาน
func createOneToOne(context: NSManagedObjectContext) {
    let employee = Employee(context: context)
    employee.name = "สมชาย"
    employee.employeeID = "EMP001"
    
    let workstation = Workstation(context: context)
    workstation.computerModel = "MacBook Pro 16\""
    workstation.location = "Building A, Room 301"
    
    // เชื่อมความสัมพันธ์
    employee.workstation = workstation
    // Core Data จะ set inverse อัตโนมัติ
    // workstation.employee จะเป็น employee โดยอัตโนมัติ
    
    try? context.save()
    
    // Access relationship
    if let ws = employee.workstation {
        print("\(employee.name!) uses \(ws.computerModel!)")
    }
}
```

### One-to-Many Relationship

```swift
import CoreData

/*
 Department ←→ Employees (One-to-Many)
 
 ใน Data Model Editor:
 Department Entity:
   - Relationship "employees": to Employee, Many, Optional, 
     Inverse: "department", Delete Rule: Cascade
 
 Employee Entity:
   - Relationship "department": to Department, Optional,
     Inverse: "employees", Delete Rule: Nullify
*/

@objc(Department)
public class Department: NSManagedObject {
    @NSManaged public var name: String?
    @NSManaged public var budget: Double
    @NSManaged public var employees: NSSet?  // One-to-Many (unordered)
    
    // Typed accessor สำหรับ employees
    var employeeSet: Set<Employee> {
        return employees as? Set<Employee> ?? []
    }
    
    // Computed property
    var headCount: Int {
        return employeeSet.count
    }
    
    var averageSalary: Double {
        let salaries = employeeSet.map { $0.salary }
        guard !salaries.isEmpty else { return 0 }
        return salaries.reduce(0, +) / Double(salaries.count)
    }
}

// Auto-generated helper methods สำหรับ NSSet relationship
extension Department {
    
    @objc(addEmployeesObject:)
    @NSManaged public func addToEmployees(_ value: Employee)
    
    @objc(removeEmployeesObject:)
    @NSManaged public func removeFromEmployees(_ value: Employee)
    
    @objc(addEmployees:)
    @NSManaged public func addToEmployees(_ values: NSSet)
    
    @objc(removeEmployees:)
    @NSManaged public func removeFromEmployees(_ values: NSSet)
}

@objc(Employee2)
public class Employee2: NSManagedObject {
    @NSManaged public var name: String?
    @NSManaged public var salary: Double
    @NSManaged public var department: Department?  // Many-to-One
}

// การใช้งาน
func createOneToMany(context: NSManagedObjectContext) {
    let dept = Department(context: context)
    dept.name = "วิศวกรรมซอฟต์แวร์"
    dept.budget = 5000000
    
    let emp1 = Employee2(context: context)
    emp1.name = "สมชาย"
    emp1.salary = 80000
    
    let emp2 = Employee2(context: context)
    emp2.name = "สมหญิง"
    emp2.salary = 85000
    
    let emp3 = Employee2(context: context)
    emp3.name = "วิชัย"
    emp3.salary = 75000
    
    // เพิ่ม employees เข้า department
    dept.addToEmployees(emp1)
    dept.addToEmployees(emp2)
    dept.addToEmployees(emp3)
    
    try? context.save()
    
    // Query
    print("แผนก: \(dept.name!)")
    print("จำนวนพนักงาน: \(dept.headCount)")
    print("เงินเดือนเฉลี่ย: \(dept.averageSalary)")
    
    for employee in dept.employeeSet.sorted(by: { $0.name! < $1.name! }) {
        print("  - \(employee.name!): ฿\(employee.salary)")
    }
}
```

### Many-to-Many Relationship

```swift
import CoreData

/*
 Student ←→ Course (Many-to-Many)
 
 ใน Data Model Editor:
 Student Entity:
   - Relationship "courses": to Course, Many, Optional,
     Inverse: "students", Delete Rule: Nullify
 
 Course Entity:
   - Relationship "students": to Student, Many, Optional,
     Inverse: "courses", Delete Rule: Nullify
*/

@objc(Student2)
public class Student2: NSManagedObject {
    @NSManaged public var name: String?
    @NSManaged public var studentID: String?
    @NSManaged public var courses: NSSet?
    
    var courseSet: Set<Course> {
        return courses as? Set<Course> ?? []
    }
    
    var enrolledCourseCount: Int { courseSet.count }
}

extension Student2 {
    @objc(addCoursesObject:)
    @NSManaged public func addToCourses(_ value: Course)
    
    @objc(removeCoursesObject:)
    @NSManaged public func removeFromCourses(_ value: Course)
}

@objc(Course)
public class Course: NSManagedObject {
    @NSManaged public var title: String?
    @NSManaged public var courseCode: String?
    @NSManaged public var credits: Int16
    @NSManaged public var students: NSSet?
    
    var studentSet: Set<Student2> {
        return students as? Set<Student2> ?? []
    }
    
    var enrollmentCount: Int { studentSet.count }
}

extension Course {
    @objc(addStudentsObject:)
    @NSManaged public func addToStudents(_ value: Student2)
    
    @objc(removeStudentsObject:)
    @NSManaged public func removeFromStudents(_ value: Student2)
}

// การใช้งาน
func createManyToMany(context: NSManagedObjectContext) {
    // สร้าง Students
    let student1 = Student2(context: context)
    student1.name = "สมชาย"
    student1.studentID = "STU001"
    
    let student2 = Student2(context: context)
    student2.name = "มาลี"
    student2.studentID = "STU002"
    
    // สร้าง Courses
    let course1 = Course(context: context)
    course1.title = "iOS Development"
    course1.courseCode = "CS401"
    course1.credits = 3
    
    let course2 = Course(context: context)
    course2.title = "Swift Programming"
    course2.courseCode = "CS301"
    course2.credits = 3
    
    let course3 = Course(context: context)
    course3.title = "UI/UX Design"
    course3.courseCode = "DE201"
    course3.credits = 2
    
    // สมชาย ลงทะเบียน 2 วิชา
    student1.addToCourses(course1)
    student1.addToCourses(course2)
    
    // มาลี ลงทะเบียน 3 วิชา
    student2.addToCourses(course1)
    student2.addToCourses(course2)
    student2.addToCourses(course3)
    
    try? context.save()
    
    // Query
    print("สมชาย ลงทะเบียน: \(student1.enrolledCourseCount) วิชา")
    for course in student1.courseSet {
        print("  - \(course.title!) (\(course.courseCode!))")
    }
    
    print("\nวิชา '\(course1.title!)' มีนักเรียน: \(course1.enrollmentCount) คน")
    for student in course1.studentSet {
        print("  - \(student.name!)")
    }
}
```

---

## NSManagedObject Subclasses

### การสร้าง Subclass อัตโนมัติ (Xcode)

ใน Xcode: เลือก Entity → ใน Data Model Inspector → Codegen เลือก "Class Definition" หรือ "Category/Extension"

```swift
import CoreData

// เมื่อ Codegen = "Class Definition" Xcode จะ generate file เหล่านี้:

// File 1: Note+CoreDataClass.swift (สร้างโดย Xcode)
@objc(Note)
public class Note: NSManagedObject {
    // คุณสามารถเพิ่ม custom code ที่นี่
    
    var formattedDate: String {
        guard let date = createdAt else { return "" }
        let formatter = DateFormatter()
        formatter.dateStyle = .medium
        formatter.timeStyle = .short
        formatter.locale = Locale(identifier: "th_TH")
        return formatter.string(from: date)
    }
    
    var preview: String {
        guard let content = content, !content.isEmpty else {
            return "ไม่มีเนื้อหา"
        }
        let maxLength = 100
        return content.count > maxLength
            ? String(content.prefix(maxLength)) + "..."
            : content
    }
}

// File 2: Note+CoreDataProperties.swift (generate ใหม่ทุกครั้งที่ model เปลี่ยน)
extension Note {
    
    @nonobjc public class func fetchRequest() -> NSFetchRequest<Note> {
        return NSFetchRequest<Note>(entityName: "Note")
    }
    
    @NSManaged public var id: UUID?
    @NSManaged public var title: String?
    @NSManaged public var content: String?
    @NSManaged public var createdAt: Date?
    @NSManaged public var updatedAt: Date?
    @NSManaged public var isPinned: Bool
    @NSManaged public var tags: NSSet?
}

extension Note {
    @objc(addTagsObject:)
    @NSManaged public func addToTags(_ value: NoteTag)
    
    @objc(removeTagsObject:)
    @NSManaged public func removeFromTags(_ value: NoteTag)
    
    @objc(addTags:)
    @NSManaged public func addToTags(_ values: NSSet)
    
    @objc(removeTags:)
    @NSManaged public func removeFromTags(_ values: NSSet)
}

extension Note: Identifiable {}
```

### Manual NSManagedObject Subclass

```swift
import CoreData

// สร้าง subclass เองโดยไม่ใช้ Xcode generation
@objc(Task)
public class Task: NSManagedObject {
    
    @NSManaged public var id: UUID?
    @NSManaged public var title: String?
    @NSManaged public var notes: String?
    @NSManaged public var dueDate: Date?
    @NSManaged public var priority: Int16
    @NSManaged public var isCompleted: Bool
    @NSManaged public var completedAt: Date?
    @NSManaged public var createdAt: Date?
    @NSManaged public var tags: String?  // comma-separated
    
    // Computed properties
    var priorityLevel: PriorityLevel {
        get { PriorityLevel(rawValue: Int(priority)) ?? .medium }
        set { priority = Int16(newValue.rawValue) }
    }
    
    var tagList: [String] {
        get { tags?.components(separatedBy: ",").filter { !$0.isEmpty } ?? [] }
        set { tags = newValue.joined(separator: ",") }
    }
    
    var isOverdue: Bool {
        guard let due = dueDate, !isCompleted else { return false }
        return due < Date()
    }
    
    // Factory method
    static func create(
        title: String,
        dueDate: Date? = nil,
        priority: PriorityLevel = .medium,
        in context: NSManagedObjectContext
    ) -> Task {
        let task = Task(context: context)
        task.id = UUID()
        task.title = title
        task.dueDate = dueDate
        task.priority = Int16(priority.rawValue)
        task.isCompleted = false
        task.createdAt = Date()
        return task
    }
    
    // Mark complete
    func complete() {
        isCompleted = true
        completedAt = Date()
    }
    
    enum PriorityLevel: Int {
        case low = 1
        case medium = 2
        case high = 3
        case urgent = 4
        
        var displayName: String {
            switch self {
            case .low: return "ต่ำ"
            case .medium: return "กลาง"
            case .high: return "สูง"
            case .urgent: return "เร่งด่วน"
            }
        }
    }
}

extension Task {
    @nonobjc public class func fetchRequest() -> NSFetchRequest<Task> {
        return NSFetchRequest<Task>(entityName: "Task")
    }
}

extension Task: Identifiable {}
```

---

## @NSManaged Property Wrapper

`@NSManaged` บอก Swift ว่า property นี้จะถูกจัดการโดย Core Data runtime

```swift
import CoreData

/*
 @NSManaged ทำงานอย่างไร:
 
 1. ไม่มี storage ใน Swift struct/class
 2. Core Data จัดการ backing store
 3. Getter/Setter ถูก synthesize โดย Obj-C runtime
 4. Lazy loading อัตโนมัติ
 5. Change tracking อัตโนมัติ
*/

@objc(Item)
public class Item: NSManagedObject {
    
    // @NSManaged properties - Core Data จัดการ
    @NSManaged public var name: String?
    @NSManaged public var price: Double
    @NSManaged public var quantity: Int32
    @NSManaged public var imageData: Data?
    @NSManaged public var createdAt: Date?
    
    // Regular Swift property - ไม่ได้ persist
    var displayName: String {
        return name ?? "Unknown Item"
    }
    
    // ❌ ห้ามทำแบบนี้ - @NSManaged ต้องไม่มี initial value
    // @NSManaged public var name: String? = nil  // Error!
    
    // ❌ @NSManaged ต้องเป็น @objc class
    // @NSManaged public var customType: MyCustomStruct  // Error!
    
    // ✅ ถ้าต้องการ store custom type ใช้ Transformable
    @NSManaged public var metadata: NSObject?  // Transformable attribute
}

// การใช้ Transformable attribute
// ใน Data Model: ตั้ง Type = Transformable, Custom Class = ColorTransformer

@objc(AppTheme)
public class AppTheme: NSManagedObject {
    @NSManaged public var name: String?
    @NSManaged public var primaryColor: UIColor?  // Transformable
    @NSManaged public var secondaryColor: UIColor?  // Transformable
}

// ValueTransformer สำหรับ UIColor
class ColorTransformer: ValueTransformer {
    
    override class func transformedValueClass() -> AnyClass {
        return NSData.self
    }
    
    override class func allowsReverseTransformation() -> Bool {
        return true
    }
    
    override func transformedValue(_ value: Any?) -> Any? {
        guard let color = value as? UIColor else { return nil }
        return try? NSKeyedArchiver.archivedData(withRootObject: color, requiringSecureCoding: true)
    }
    
    override func reverseTransformedValue(_ value: Any?) -> Any? {
        guard let data = value as? Data else { return nil }
        return try? NSKeyedUnarchiver.unarchivedObject(ofClass: UIColor.self, from: data)
    }
}

// Register transformer
// ColorTransformer.register(withName: NSValueTransformerName("ColorTransformer"))
```

---

## Saving Context

### Basic Save

```swift
import CoreData

class CoreDataManager {
    
    let context: NSManagedObjectContext
    
    init(context: NSManagedObjectContext) {
        self.context = context
    }
    
    // Basic save
    func save() throws {
        guard context.hasChanges else { return }  // ไม่มีการเปลี่ยนแปลง
        try context.save()
    }
    
    // Save with error handling
    func saveIfNeeded() {
        guard context.hasChanges else { return }
        
        do {
            try context.save()
            print("Context saved successfully")
        } catch {
            let nsError = error as NSError
            print("Save error: \(nsError), \(nsError.userInfo)")
            
            // Rollback เพื่อ reset context state
            context.rollback()
        }
    }
    
    // Save parent context ด้วย (สำหรับ child contexts)
    func saveContextHierarchy(_ context: NSManagedObjectContext) throws {
        guard context.hasChanges else { return }
        
        try context.save()
        
        // ถ้ามี parent context ให้ save ด้วย
        if let parent = context.parent {
            try saveContextHierarchy(parent)
        }
    }
}
```

### Transaction Pattern

```swift
import CoreData

// Perform operations ใน transaction-like block
func performTransaction(
    context: NSManagedObjectContext,
    operations: (NSManagedObjectContext) throws -> Void
) throws {
    // เก็บ state เดิม
    let undoManager = context.undoManager ?? UndoManager()
    context.undoManager = undoManager
    
    undoManager.beginUndoGrouping()
    
    do {
        try operations(context)
        
        if context.hasChanges {
            try context.save()
        }
        
        undoManager.endUndoGrouping()
    } catch {
        // ยกเลิกการเปลี่ยนแปลงทั้งหมด
        undoManager.endUndoGrouping()
        undoManager.undo()
        throw error
    }
}

// ตัวอย่างการใช้งาน
func transferBudget(
    from source: Department,
    to destination: Department,
    amount: Double,
    context: NSManagedObjectContext
) throws {
    try performTransaction(context: context) { ctx in
        guard source.budget >= amount else {
            throw NSError(
                domain: "BudgetError",
                code: 1,
                userInfo: [NSLocalizedDescriptionKey: "Insufficient budget"]
            )
        }
        source.budget -= amount
        destination.budget += amount
    }
}
```

---

## Fetching Data

### NSFetchRequest

```swift
import CoreData

class NoteRepository {
    
    let context: NSManagedObjectContext
    
    init(context: NSManagedObjectContext) {
        self.context = context
    }
    
    // Fetch ทั้งหมด
    func fetchAll() throws -> [Note] {
        let request = NSFetchRequest<Note>(entityName: "Note")
        return try context.fetch(request)
    }
    
    // Fetch แบบ typed
    func fetchAllTyped() throws -> [Note] {
        let request: NSFetchRequest<Note> = Note.fetchRequest()
        return try context.fetch(request)
    }
    
    // Fetch พร้อม options
    func fetchRecent(limit: Int = 20) throws -> [Note] {
        let request: NSFetchRequest<Note> = Note.fetchRequest()
        request.fetchLimit = limit
        request.fetchOffset = 0
        request.sortDescriptors = [
            NSSortDescriptor(key: "createdAt", ascending: false)
        ]
        return try context.fetch(request)
    }
    
    // Count records
    func count(predicate: NSPredicate? = nil) throws -> Int {
        let request: NSFetchRequest<Note> = Note.fetchRequest()
        request.predicate = predicate
        return try context.count(for: request)
    }
    
    // Check if exists
    func exists(id: UUID) throws -> Bool {
        let request: NSFetchRequest<Note> = Note.fetchRequest()
        request.predicate = NSPredicate(format: "id == %@", id as CVarArg)
        request.fetchLimit = 1
        return try context.count(for: request) > 0
    }
    
    // Fetch by ID
    func find(id: UUID) throws -> Note? {
        let request: NSFetchRequest<Note> = Note.fetchRequest()
        request.predicate = NSPredicate(format: "id == %@", id as CVarArg)
        request.fetchLimit = 1
        return try context.fetch(request).first
    }
    
    // Fetch IDs only (lightweight)
    func fetchIDs() throws -> [NSManagedObjectID] {
        let request: NSFetchRequest<NSManagedObjectID> = NSFetchRequest(entityName: "Note")
        request.resultType = .managedObjectIDResultType
        return try context.fetch(request)
    }
    
    // Batch fetch สำหรับ large datasets
    func fetchBatched(batchSize: Int = 50) -> [Note] {
        let request: NSFetchRequest<Note> = Note.fetchRequest()
        request.fetchBatchSize = batchSize
        request.sortDescriptors = [NSSortDescriptor(key: "createdAt", ascending: false)]
        
        return (try? context.fetch(request)) ?? []
    }
}
```

---

## Predicates (NSPredicate)

`NSPredicate` ใช้สำหรับ filter ข้อมูลที่ fetch

### Basic Predicates

```swift
import CoreData

class PredicateExamples {
    
    let context: NSManagedObjectContext
    
    init(context: NSManagedObjectContext) {
        self.context = context
    }
    
    // String comparison
    func fetchByTitle(_ title: String) throws -> [Note] {
        let request: NSFetchRequest<Note> = Note.fetchRequest()
        request.predicate = NSPredicate(format: "title == %@", title)
        return try context.fetch(request)
    }
    
    // Case insensitive search
    func searchNotes(keyword: String) throws -> [Note] {
        let request: NSFetchRequest<Note> = Note.fetchRequest()
        // [c] = case insensitive, [d] = diacritic insensitive
        request.predicate = NSPredicate(
            format: "title CONTAINS[cd] %@ OR content CONTAINS[cd] %@",
            keyword, keyword
        )
        return try context.fetch(request)
    }
    
    // BEGINSWITH, ENDSWITH
    func fetchTitlesStartingWith(_ prefix: String) throws -> [Note] {
        let request: NSFetchRequest<Note> = Note.fetchRequest()
        request.predicate = NSPredicate(format: "title BEGINSWITH[c] %@", prefix)
        return try context.fetch(request)
    }
    
    // LIKE (wildcard: * = any chars, ? = one char)
    func fetchWithPattern(_ pattern: String) throws -> [Note] {
        let request: NSFetchRequest<Note> = Note.fetchRequest()
        request.predicate = NSPredicate(format: "title LIKE[c] %@", pattern)
        return try context.fetch(request)
    }
    
    // MATCHES (regex)
    func fetchMatchingRegex(_ pattern: String) throws -> [Note] {
        let request: NSFetchRequest<Note> = Note.fetchRequest()
        request.predicate = NSPredicate(format: "title MATCHES %@", pattern)
        return try context.fetch(request)
    }
    
    // Number comparison
    func fetchHighPriorityTasks() throws -> [Task] {
        let request: NSFetchRequest<Task> = Task.fetchRequest()
        request.predicate = NSPredicate(format: "priority >= %d", 3)
        return try context.fetch(request)
    }
    
    // Range
    func fetchTasksWithPriorityRange(min: Int, max: Int) throws -> [Task] {
        let request: NSFetchRequest<Task> = Task.fetchRequest()
        request.predicate = NSPredicate(
            format: "priority >= %d AND priority <= %d",
            min, max
        )
        return try context.fetch(request)
    }
    
    // BETWEEN
    func fetchTasksBetweenPriorities(min: Int, max: Int) throws -> [Task] {
        let request: NSFetchRequest<Task> = Task.fetchRequest()
        request.predicate = NSPredicate(
            format: "priority BETWEEN {%d, %d}",
            min, max
        )
        return try context.fetch(request)
    }
    
    // IN
    func fetchTasksWithPriorities(_ priorities: [Int]) throws -> [Task] {
        let request: NSFetchRequest<Task> = Task.fetchRequest()
        request.predicate = NSPredicate(format: "priority IN %@", priorities)
        return try context.fetch(request)
    }
    
    // Boolean
    func fetchCompletedTasks() throws -> [Task] {
        let request: NSFetchRequest<Task> = Task.fetchRequest()
        request.predicate = NSPredicate(format: "isCompleted == YES")
        return try context.fetch(request)
    }
    
    // Date comparison
    func fetchNotesCreatedAfter(_ date: Date) throws -> [Note] {
        let request: NSFetchRequest<Note> = Note.fetchRequest()
        request.predicate = NSPredicate(format: "createdAt > %@", date as CVarArg)
        return try context.fetch(request)
    }
    
    // Date range
    func fetchNotesInDateRange(from start: Date, to end: Date) throws -> [Note] {
        let request: NSFetchRequest<Note> = Note.fetchRequest()
        request.predicate = NSPredicate(
            format: "createdAt >= %@ AND createdAt <= %@",
            start as CVarArg, end as CVarArg
        )
        return try context.fetch(request)
    }
    
    // NULL check
    func fetchNotesWithContent() throws -> [Note] {
        let request: NSFetchRequest<Note> = Note.fetchRequest()
        request.predicate = NSPredicate(format: "content != nil AND content != ''")
        return try context.fetch(request)
    }
    
    // Compound predicates
    func fetchPinnedHighPriorityNotes() throws -> [Note] {
        let request: NSFetchRequest<Note> = Note.fetchRequest()
        
        let pinnedPredicate = NSPredicate(format: "isPinned == YES")
        let recentPredicate = NSPredicate(
            format: "createdAt > %@",
            Date().addingTimeInterval(-7 * 24 * 3600) as CVarArg  // 7 วันที่แล้ว
        )
        
        // AND (NSCompoundPredicate)
        request.predicate = NSCompoundPredicate(
            andPredicateWithSubpredicates: [pinnedPredicate, recentPredicate]
        )
        
        return try context.fetch(request)
    }
    
    // OR predicate
    func fetchImportantNotes() throws -> [Note] {
        let request: NSFetchRequest<Note> = Note.fetchRequest()
        
        let pinnedPredicate = NSPredicate(format: "isPinned == YES")
        let keywordPredicate = NSPredicate(format: "title CONTAINS[c] 'important'")
        
        request.predicate = NSCompoundPredicate(
            orPredicateWithSubpredicates: [pinnedPredicate, keywordPredicate]
        )
        
        return try context.fetch(request)
    }
    
    // Relationship predicate
    func fetchNotesWithTag(_ tagName: String) throws -> [Note] {
        let request: NSFetchRequest<Note> = Note.fetchRequest()
        request.predicate = NSPredicate(format: "ANY tags.name == %@", tagName)
        return try context.fetch(request)
    }
}
```

### NSPredicate Format Arguments

```swift
// %@ - Object (String, Date, UUID, etc.)
NSPredicate(format: "name == %@", "สมชาย")

// %d - Integer (Int, Int32, Int64)
NSPredicate(format: "age == %d", 30)

// %i - Integer (same as %d)
NSPredicate(format: "count == %i", 5)

// %f - Float/Double
NSPredicate(format: "price > %f", 100.0)

// %K - Keypath (column name from variable)
let column = "name"
NSPredicate(format: "%K == %@", column, "สมชาย")

// Multiple arguments
NSPredicate(format: "firstName == %@ AND lastName == %@", "สมชาย", "ใจดี")
```

---

## Sort Descriptors

```swift
import CoreData

class SortedFetchExamples {
    
    let context: NSManagedObjectContext
    
    init(context: NSManagedObjectContext) {
        self.context = context
    }
    
    // Single sort
    func fetchNotesSortedByDate() throws -> [Note] {
        let request: NSFetchRequest<Note> = Note.fetchRequest()
        request.sortDescriptors = [
            NSSortDescriptor(key: "createdAt", ascending: false)  // ล่าสุดก่อน
        ]
        return try context.fetch(request)
    }
    
    // Sort by KeyPath (type-safe, Swift 5.0+)
    func fetchNotesSortedByTitle() throws -> [Note] {
        let request: NSFetchRequest<Note> = Note.fetchRequest()
        request.sortDescriptors = [
            NSSortDescriptor(keyPath: \Note.title, ascending: true)
        ]
        return try context.fetch(request)
    }
    
    // Multiple sort descriptors
    func fetchTasksSorted() throws -> [Task] {
        let request: NSFetchRequest<Task> = Task.fetchRequest()
        request.sortDescriptors = [
            NSSortDescriptor(key: "priority", ascending: false),    // priority สูงก่อน
            NSSortDescriptor(key: "dueDate", ascending: true),      // due date เร็วก่อน
            NSSortDescriptor(key: "title", ascending: true)         // ตัวอักษร A-Z
        ]
        return try context.fetch(request)
    }
    
    // Sort with locale (สำหรับภาษาไทย)
    func fetchNotesSortedByTitleThai() throws -> [Note] {
        let request: NSFetchRequest<Note> = Note.fetchRequest()
        
        let sortDescriptor = NSSortDescriptor(
            key: "title",
            ascending: true,
            selector: #selector(NSString.localizedCompare(_:))
        )
        
        request.sortDescriptors = [sortDescriptor]
        return try context.fetch(request)
    }
    
    // Sort with custom comparator
    func fetchNotesCustomSorted() throws -> [Note] {
        let request: NSFetchRequest<Note> = Note.fetchRequest()
        
        let sortByPinned = NSSortDescriptor(key: "isPinned", ascending: false)
        let sortByDate = NSSortDescriptor(key: "updatedAt", ascending: false)
        
        request.sortDescriptors = [sortByPinned, sortByDate]
        return try context.fetch(request)
    }
}
```

---

## Creating, Updating, Deleting Records

### CRUD Operations

```swift
import CoreData

class NoteService {
    
    let context: NSManagedObjectContext
    
    init(context: NSManagedObjectContext) {
        self.context = context
    }
    
    // CREATE
    @discardableResult
    func createNote(title: String, content: String? = nil) throws -> Note {
        let note = Note(context: context)
        note.id = UUID()
        note.title = title
        note.content = content
        note.createdAt = Date()
        note.updatedAt = Date()
        note.isPinned = false
        
        try context.save()
        return note
    }
    
    // READ (Fetch)
    func fetchNote(id: UUID) throws -> Note? {
        let request: NSFetchRequest<Note> = Note.fetchRequest()
        request.predicate = NSPredicate(format: "id == %@", id as CVarArg)
        request.fetchLimit = 1
        return try context.fetch(request).first
    }
    
    func fetchAllNotes() throws -> [Note] {
        let request: NSFetchRequest<Note> = Note.fetchRequest()
        request.sortDescriptors = [
            NSSortDescriptor(key: "updatedAt", ascending: false)
        ]
        return try context.fetch(request)
    }
    
    // UPDATE
    func updateNote(_ note: Note, title: String? = nil, content: String? = nil) throws {
        if let title = title {
            note.title = title
        }
        if let content = content {
            note.content = content
        }
        note.updatedAt = Date()
        
        if context.hasChanges {
            try context.save()
        }
    }
    
    // Toggle pin
    func togglePin(_ note: Note) throws {
        note.isPinned.toggle()
        note.updatedAt = Date()
        try context.save()
    }
    
    // DELETE
    func deleteNote(_ note: Note) throws {
        context.delete(note)
        try context.save()
    }
    
    // DELETE multiple
    func deleteNotes(_ notes: [Note]) throws {
        notes.forEach { context.delete($0) }
        try context.save()
    }
    
    // BATCH DELETE (ประสิทธิภาพสูงสำหรับข้อมูลจำนวนมาก)
    func deleteAllNotes() throws {
        let request: NSFetchRequest<NSFetchRequestResult> = NSFetchRequest(entityName: "Note")
        let deleteRequest = NSBatchDeleteRequest(fetchRequest: request)
        deleteRequest.resultType = .resultTypeObjectIDs
        
        let result = try context.execute(deleteRequest) as? NSBatchDeleteResult
        let objectIDs = result?.result as? [NSManagedObjectID] ?? []
        
        // Update context
        NSManagedObjectContext.mergeChanges(
            fromRemoteContextSave: [NSDeletedObjectsKey: objectIDs],
            into: [context]
        )
    }
    
    // BATCH UPDATE (ประสิทธิภาพสูง)
    func markAllNotesAsRead() throws {
        let request: NSFetchRequest<NSFetchRequestResult> = NSFetchRequest(entityName: "Note")
        let updateRequest = NSBatchUpdateRequest(entityName: "Note")
        updateRequest.propertiesToUpdate = ["isRead": true, "updatedAt": Date()]
        updateRequest.resultType = .updatedObjectIDsResultType
        
        let result = try context.execute(updateRequest) as? NSBatchUpdateResult
        let objectIDs = result?.result as? [NSManagedObjectID] ?? []
        
        // Refresh objects ใน context
        for objectID in objectIDs {
            let object = try? context.existingObject(with: objectID)
            context.refresh(object!, mergeChanges: false)
        }
    }
}
```

---

## Background Context

การทำงานกับ Core Data ใน background thread

```swift
import CoreData

class BackgroundTaskManager {
    
    let container: NSPersistentContainer
    
    init(container: NSPersistentContainer) {
        self.container = container
    }
    
    // Method 1: performBackgroundTask (iOS 10+)
    func importData(items: [(title: String, content: String)]) {
        container.performBackgroundTask { context in
            // ทำงานใน background thread
            for item in items {
                let note = Note(context: context)
                note.id = UUID()
                note.title = item.title
                note.content = item.content
                note.createdAt = Date()
            }
            
            do {
                try context.save()
                print("Imported \(items.count) notes")
            } catch {
                print("Import error: \(error)")
            }
        }
    }
    
    // Method 2: newBackgroundContext
    func processDataInBackground(completion: @escaping (Bool) -> Void) {
        let backgroundContext = container.newBackgroundContext()
        backgroundContext.mergePolicy = NSMergeByPropertyObjectTrumpMergePolicy
        
        backgroundContext.perform {
            // Heavy processing...
            let request: NSFetchRequest<Note> = Note.fetchRequest()
            
            do {
                let notes = try backgroundContext.fetch(request)
                
                for note in notes {
                    // Process each note...
                }
                
                if backgroundContext.hasChanges {
                    try backgroundContext.save()
                }
                
                DispatchQueue.main.async {
                    completion(true)
                }
            } catch {
                print("Background processing error: \(error)")
                DispatchQueue.main.async {
                    completion(false)
                }
            }
        }
    }
    
    // Method 3: async/await (iOS 15+)
    @available(iOS 15.0, *)
    func importDataAsync(items: [(title: String, content: String)]) async throws {
        try await container.performBackgroundTask { context in
            for item in items {
                let note = Note(context: context)
                note.id = UUID()
                note.title = item.title
                note.content = item.content
                note.createdAt = Date()
            }
            
            try context.save()
        }
    }
    
    // Cross-context object passing
    func updateNoteInBackground(noteID: NSManagedObjectID, newTitle: String) {
        let backgroundContext = container.newBackgroundContext()
        
        backgroundContext.perform {
            // ใช้ objectID เพื่อ access object ใน background context
            guard let note = try? backgroundContext.existingObject(with: noteID) as? Note else {
                return
            }
            
            note.title = newTitle
            note.updatedAt = Date()
            
            try? backgroundContext.save()
        }
    }
}
```

---

## Merge Policies

Merge policy กำหนดว่าจะทำอย่างไรเมื่อมี conflict ระหว่าง contexts

```swift
import CoreData

/*
 Merge Policies ที่มี:
 
 1. NSErrorMergePolicy (default)
    - throw error เมื่อมี conflict
    - ต้อง handle เอง
 
 2. NSMergeByPropertyStoreTrumpMergePolicy
    - ค่าใน Persistent Store ชนะ
    - object ใน context ถูก overwrite
 
 3. NSMergeByPropertyObjectTrumpMergePolicy
    - ค่าใน object ชนะ
    - ค่าใน store ถูก overwrite
    - ใช้บ่อยสำหรับ background contexts
 
 4. NSOverwriteMergePolicy
    - object ชนะเสมอ (overwrite ทุกอย่าง)
 
 5. NSRollbackMergePolicy
    - ยกเลิก unsaved changes ใน context
*/

class MergePolicyDemo {
    
    let container: NSPersistentContainer
    
    init(container: NSPersistentContainer) {
        self.container = container
    }
    
    func setupContexts() {
        // Main context - ใช้สำหรับ UI
        let mainContext = container.viewContext
        mainContext.automaticallyMergesChangesFromParent = true
        mainContext.mergePolicy = NSMergeByPropertyObjectTrumpMergePolicy
        
        // Background context - สำหรับ heavy operations
        let bgContext = container.newBackgroundContext()
        bgContext.mergePolicy = NSMergeByPropertyObjectTrumpMergePolicy
        
        // Observe changes จาก background context
        NotificationCenter.default.addObserver(
            forName: .NSManagedObjectContextDidSave,
            object: bgContext,
            queue: .main
        ) { [weak mainContext] notification in
            mainContext?.mergeChanges(fromContextDidSave: notification)
        }
    }
    
    // Handle merge conflict manually
    func handleConflict() {
        let context = container.viewContext
        context.mergePolicy = NSErrorMergePolicy
        
        do {
            try context.save()
        } catch let error as NSError {
            if let conflicts = error.userInfo[NSPersistentStoreSaveConflictsErrorKey] as? [NSMergeConflict] {
                for conflict in conflicts {
                    print("Conflict for: \(conflict.sourceObject)")
                    print("Persistent version: \(String(describing: conflict.persistedSnapshot))")
                    print("In-memory version: \(String(describing: conflict.cachedSnapshot))")
                }
                
                // Resolve ด้วย policy อื่น
                context.mergePolicy = NSMergeByPropertyObjectTrumpMergePolicy
                try? context.save()
            }
        }
    }
}
```

---

## Versioning and Migration

เมื่อ data model เปลี่ยน เราต้อง migrate data เดิม

### ประเภทของ Migration

```
1. Lightweight Migration
   - Core Data ทำให้อัตโนมัติ
   - รองรับการเปลี่ยนแปลงง่ายๆ:
     ✅ เพิ่ม/ลบ attributes
     ✅ เพิ่ม/ลบ entities
     ✅ เพิ่ม/ลบ relationships
     ✅ เปลี่ยนชื่อ (ด้วย Renaming Identifier)
     ✅ เปลี่ยน Optional/Required
     ✅ เพิ่ม/เปลี่ยน default values

2. Custom Migration
   - ต้องสร้าง Mapping Model
   - รองรับการเปลี่ยนแปลงที่ซับซ้อน:
     - Transform data
     - Merge entities
     - Split entities
     - Complex data transformation
```

### Versioning ใน Xcode

```
1. เปิด .xcdatamodeld
2. Editor menu → Add Model Version
3. เลือก Current Version ใหม่
4. แก้ไข model version ใหม่
5. Enable migration ใน NSPersistentContainer
```

### Enable Migration

```swift
import CoreData

class MigrationEnabledPersistenceController {
    
    let container: NSPersistentContainer
    
    init() {
        container = NSPersistentContainer(name: "MyApp")
        
        // Configure migration options
        let description = container.persistentStoreDescriptions.first!
        description.shouldMigrateStoreAutomatically = true
        description.shouldInferMappingModelAutomatically = true
        
        container.loadPersistentStores { _, error in
            if let error = error {
                fatalError("Failed to load stores: \(error)")
            }
        }
    }
}

// หรือใช้ options dictionary
class ManualMigrationController {
    
    func loadStore(container: NSPersistentContainer) {
        let storeURL = container.persistentStoreDescriptions.first!.url!
        
        let options: [String: Any] = [
            NSMigratePersistentStoresAutomaticallyOption: true,
            NSInferMappingModelAutomaticallyOption: true
        ]
        
        do {
            try container.persistentStoreCoordinator.addPersistentStore(
                ofType: NSSQLiteStoreType,
                configurationName: nil,
                at: storeURL,
                options: options
            )
        } catch {
            fatalError("Migration failed: \(error)")
        }
    }
}
```

---

## Lightweight Migration

### เปลี่ยนชื่อ Attribute

```
ใน Xcode Data Model Editor:
1. เลือก Attribute ที่ต้องการ rename
2. ใน Data Model Inspector
3. ตั้ง "Renaming Identifier" เป็นชื่อเดิม
4. เปลี่ยนชื่อ attribute ใหม่

ตัวอย่าง: 
  Model V1: attribute "username" 
  Model V2: rename เป็น "userName", ตั้ง Renaming ID = "username"
```

### ตรวจสอบ Migration Status

```swift
import CoreData

class MigrationChecker {
    
    func checkMigrationNeeded(
        storeURL: URL,
        modelURL: URL
    ) -> Bool {
        guard let metadata = try? NSPersistentStoreCoordinator.metadataForPersistentStore(
            ofType: NSSQLiteStoreType,
            at: storeURL
        ) else {
            return false  // ไม่มี store เลย
        }
        
        guard let model = NSManagedObjectModel(contentsOf: modelURL) else {
            return false
        }
        
        return !model.isConfiguration(withName: nil, compatibleWithStoreMetadata: metadata)
    }
    
    func performMigration(from sourceURL: URL, to destinationURL: URL) {
        // Custom migration process
        guard let sourceModel = NSManagedObjectModel.mergedModel(
            from: Bundle.allBundles,
            forStoreMetadata: try! NSPersistentStoreCoordinator.metadataForPersistentStore(
                ofType: NSSQLiteStoreType, at: sourceURL
            )
        ) else {
            print("Cannot find source model")
            return
        }
        
        // Load destination model (current)
        guard let destinationModel = NSManagedObjectModel.mergedModel(from: [Bundle.main]) else {
            print("Cannot find destination model")
            return
        }
        
        // Find mapping model
        guard let mappingModel = try? NSMappingModel.inferredMappingModel(
            forSourceModel: sourceModel,
            destinationModel: destinationModel
        ) else {
            print("Cannot infer mapping model")
            return
        }
        
        // Migrate
        let migrationManager = NSMigrationManager(
            sourceModel: sourceModel,
            destinationModel: destinationModel
        )
        
        do {
            try migrationManager.migrateStore(
                from: sourceURL,
                sourceType: NSSQLiteStoreType,
                options: nil,
                with: mappingModel,
                toDestinationURL: destinationURL,
                destinationType: NSSQLiteStoreType,
                destinationOptions: nil
            )
            print("Migration successful!")
        } catch {
            print("Migration failed: \(error)")
        }
    }
}
```

---

## Core Data กับ SwiftUI

### @FetchRequest

```swift
import SwiftUI
import CoreData

// Basic @FetchRequest
struct NoteListView: View {
    
    @Environment(\.managedObjectContext) private var viewContext
    
    // Simple fetch
    @FetchRequest(
        sortDescriptors: [NSSortDescriptor(keyPath: \Note.createdAt, ascending: false)],
        animation: .default
    )
    private var notes: FetchedResults<Note>
    
    var body: some View {
        List {
            ForEach(notes) { note in
                NoteRowView(note: note)
            }
            .onDelete(perform: deleteNotes)
        }
        .toolbar {
            ToolbarItem(placement: .navigationBarTrailing) {
                Button(action: addNote) {
                    Label("Add Note", systemImage: "plus")
                }
            }
        }
    }
    
    private func addNote() {
        withAnimation {
            let newNote = Note(viewContext)
            newNote.id = UUID()
            newNote.title = "บันทึกใหม่"
            newNote.createdAt = Date()
            newNote.updatedAt = Date()
            
            try? viewContext.save()
        }
    }
    
    private func deleteNotes(offsets: IndexSet) {
        withAnimation {
            offsets.map { notes[$0] }.forEach(viewContext.delete)
            try? viewContext.save()
        }
    }
}

// @FetchRequest พร้อม predicate
struct FilteredNoteListView: View {
    
    @Environment(\.managedObjectContext) private var viewContext
    @State private var searchText = ""
    
    @FetchRequest(
        sortDescriptors: [NSSortDescriptor(keyPath: \Note.updatedAt, ascending: false)],
        predicate: NSPredicate(format: "isPinned == YES"),
        animation: .default
    )
    private var pinnedNotes: FetchedResults<Note>
    
    var body: some View {
        List(pinnedNotes) { note in
            NoteRowView(note: note)
        }
    }
}

// Dynamic @FetchRequest
struct SearchableNoteListView: View {
    
    @Environment(\.managedObjectContext) private var viewContext
    @State private var searchText = ""
    
    var body: some View {
        DynamicNoteList(searchText: searchText)
            .searchable(text: $searchText)
    }
}

struct DynamicNoteList: View {
    
    @FetchRequest var notes: FetchedResults<Note>
    
    init(searchText: String) {
        let sortDescriptors = [NSSortDescriptor(keyPath: \Note.updatedAt, ascending: false)]
        
        let predicate: NSPredicate?
        if searchText.isEmpty {
            predicate = nil
        } else {
            predicate = NSPredicate(
                format: "title CONTAINS[cd] %@ OR content CONTAINS[cd] %@",
                searchText, searchText
            )
        }
        
        _notes = FetchRequest(
            sortDescriptors: sortDescriptors,
            predicate: predicate,
            animation: .default
        )
    }
    
    var body: some View {
        List(notes) { note in
            VStack(alignment: .leading) {
                Text(note.title ?? "")
                    .font(.headline)
                Text(note.content ?? "")
                    .font(.caption)
                    .foregroundColor(.secondary)
                    .lineLimit(2)
            }
        }
    }
}
```

### NoteRowView

```swift
import SwiftUI
import CoreData

struct NoteRowView: View {
    
    @ObservedObject var note: Note
    @Environment(\.managedObjectContext) private var viewContext
    
    var body: some View {
        VStack(alignment: .leading, spacing: 4) {
            HStack {
                Text(note.title ?? "ไม่มีชื่อ")
                    .font(.headline)
                
                Spacer()
                
                if note.isPinned {
                    Image(systemName: "pin.fill")
                        .foregroundColor(.orange)
                        .font(.caption)
                }
            }
            
            if let content = note.content, !content.isEmpty {
                Text(content)
                    .font(.subheadline)
                    .foregroundColor(.secondary)
                    .lineLimit(2)
            }
            
            if let date = note.updatedAt {
                Text(date, style: .relative)
                    .font(.caption2)
                    .foregroundColor(.secondary)
            }
        }
        .swipeActions(edge: .leading) {
            Button {
                togglePin()
            } label: {
                Label(note.isPinned ? "Unpin" : "Pin",
                      systemImage: note.isPinned ? "pin.slash" : "pin")
            }
            .tint(.orange)
        }
    }
    
    private func togglePin() {
        note.isPinned.toggle()
        note.updatedAt = Date()
        try? viewContext.save()
    }
}
```

### App Entry Point กับ Core Data

```swift
import SwiftUI
import CoreData

@main
struct NotesApp: App {
    
    let persistenceController = PersistenceController.shared
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environment(\.managedObjectContext, persistenceController.viewContext)
        }
    }
}

// ContentView หลัก
struct ContentView: View {
    
    @Environment(\.managedObjectContext) private var viewContext
    
    var body: some View {
        NavigationView {
            NoteListView()
                .navigationTitle("บันทึกของฉัน")
        }
    }
}
```

### ViewModel Pattern กับ Core Data

```swift
import Foundation
import CoreData
import Combine

class NoteViewModel: ObservableObject {
    
    @Published var notes: [Note] = []
    @Published var error: Error?
    @Published var searchText = ""
    
    private let context: NSManagedObjectContext
    private var cancellables = Set<AnyCancellable>()
    
    init(context: NSManagedObjectContext) {
        self.context = context
        
        // React to search text changes
        $searchText
            .debounce(for: .milliseconds(300), scheduler: RunLoop.main)
            .sink { [weak self] _ in
                self?.fetchNotes()
            }
            .store(in: &cancellables)
        
        // Observe context changes
        NotificationCenter.default
            .publisher(for: .NSManagedObjectContextDidSave)
            .sink { [weak self] _ in
                self?.fetchNotes()
            }
            .store(in: &cancellables)
        
        fetchNotes()
    }
    
    func fetchNotes() {
        let request: NSFetchRequest<Note> = Note.fetchRequest()
        request.sortDescriptors = [
            NSSortDescriptor(key: "isPinned", ascending: false),
            NSSortDescriptor(key: "updatedAt", ascending: false)
        ]
        
        if !searchText.isEmpty {
            request.predicate = NSPredicate(
                format: "title CONTAINS[cd] %@ OR content CONTAINS[cd] %@",
                searchText, searchText
            )
        }
        
        do {
            notes = try context.fetch(request)
        } catch {
            self.error = error
        }
    }
    
    func createNote(title: String, content: String = "") {
        let note = Note(context: context)
        note.id = UUID()
        note.title = title
        note.content = content
        note.createdAt = Date()
        note.updatedAt = Date()
        
        save()
    }
    
    func deleteNote(at indexSet: IndexSet) {
        indexSet.map { notes[$0] }.forEach(context.delete)
        save()
    }
    
    func deleteNote(_ note: Note) {
        context.delete(note)
        save()
    }
    
    func togglePin(_ note: Note) {
        note.isPinned.toggle()
        note.updatedAt = Date()
        save()
    }
    
    private func save() {
        do {
            try context.save()
            fetchNotes()
        } catch {
            self.error = error
        }
    }
}
```

---

## NSFetchedResultsController

`NSFetchedResultsController` ใช้สำหรับ drive UITableView/UICollectionView กับ Core Data

```swift
import UIKit
import CoreData

class NoteTableViewController: UITableViewController {
    
    var managedObjectContext: NSManagedObjectContext!
    
    // FetchedResultsController
    lazy var fetchedResultsController: NSFetchedResultsController<Note> = {
        let request: NSFetchRequest<Note> = Note.fetchRequest()
        
        // Sort descriptors
        request.sortDescriptors = [
            NSSortDescriptor(key: "isPinned", ascending: false),
            NSSortDescriptor(key: "updatedAt", ascending: false)
        ]
        
        // sectionNameKeyPath สำหรับ group เป็น sections
        let frc = NSFetchedResultsController(
            fetchRequest: request,
            managedObjectContext: managedObjectContext,
            sectionNameKeyPath: nil,  // หรือ "sectionIdentifier" ถ้าต้องการ sections
            cacheName: "NoteCache"
        )
        
        frc.delegate = self
        return frc
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        do {
            try fetchedResultsController.performFetch()
        } catch {
            print("Fetch error: \(error)")
        }
        
        tableView.register(UITableViewCell.self, forCellReuseIdentifier: "NoteCell")
    }
    
    // MARK: - UITableView DataSource
    
    override func numberOfSections(in tableView: UITableView) -> Int {
        return fetchedResultsController.sections?.count ?? 0
    }
    
    override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return fetchedResultsController.sections?[section].numberOfObjects ?? 0
    }
    
    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "NoteCell", for: indexPath)
        let note = fetchedResultsController.object(at: indexPath)
        
        var content = cell.defaultContentConfiguration()
        content.text = note.title
        content.secondaryText = note.content
        cell.contentConfiguration = content
        
        return cell
    }
    
    override func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCell.EditingStyle, forRowAt indexPath: IndexPath) {
        if editingStyle == .delete {
            let note = fetchedResultsController.object(at: indexPath)
            managedObjectContext.delete(note)
            try? managedObjectContext.save()
        }
    }
    
    // MARK: - Section headers
    
    override func tableView(_ tableView: UITableView, titleForHeaderInSection section: Int) -> String? {
        return fetchedResultsController.sections?[section].name
    }
}

// MARK: - NSFetchedResultsControllerDelegate

extension NoteTableViewController: NSFetchedResultsControllerDelegate {
    
    // Called before changes begin
    func controllerWillChangeContent(_ controller: NSFetchedResultsController<NSFetchRequestResult>) {
        tableView.beginUpdates()
    }
    
    // Called for each change
    func controller(
        _ controller: NSFetchedResultsController<NSFetchRequestResult>,
        didChange anObject: Any,
        at indexPath: IndexPath?,
        for type: NSFetchedResultsChangeType,
        newIndexPath: IndexPath?
    ) {
        switch type {
        case .insert:
            if let newIndexPath = newIndexPath {
                tableView.insertRows(at: [newIndexPath], with: .automatic)
            }
        case .delete:
            if let indexPath = indexPath {
                tableView.deleteRows(at: [indexPath], with: .automatic)
            }
        case .update:
            if let indexPath = indexPath {
                tableView.reloadRows(at: [indexPath], with: .automatic)
            }
        case .move:
            if let indexPath = indexPath, let newIndexPath = newIndexPath {
                tableView.moveRow(at: indexPath, to: newIndexPath)
            }
        @unknown default:
            break
        }
    }
    
    // Called for section changes
    func controller(
        _ controller: NSFetchedResultsController<NSFetchRequestResult>,
        didChange sectionInfo: NSFetchedResultsSectionInfo,
        atSectionIndex sectionIndex: Int,
        for type: NSFetchedResultsChangeType
    ) {
        switch type {
        case .insert:
            tableView.insertSections(IndexSet(integer: sectionIndex), with: .automatic)
        case .delete:
            tableView.deleteSections(IndexSet(integer: sectionIndex), with: .automatic)
        default:
            break
        }
    }
    
    // Called after all changes
    func controllerDidChangeContent(_ controller: NSFetchedResultsController<NSFetchRequestResult>) {
        tableView.endUpdates()
    }
}
```

---

## CloudKit Sync กับ Core Data

### NSPersistentCloudKitContainer

```swift
import CoreData
import CloudKit

class CloudKitPersistenceController {
    
    static let shared = CloudKitPersistenceController()
    
    let container: NSPersistentCloudKitContainer
    
    init() {
        container = NSPersistentCloudKitContainer(name: "MyApp")
        
        // Configure iCloud
        guard let description = container.persistentStoreDescriptions.first else {
            fatalError("No store description found")
        }
        
        // ตั้งค่า CloudKit container identifier
        description.cloudKitContainerOptions = NSPersistentCloudKitContainerOptions(
            containerIdentifier: "iCloud.com.yourcompany.myapp"
        )
        
        // Enable remote change notifications
        description.setOption(true as NSNumber, forKey: NSPersistentHistoryTrackingKey)
        description.setOption(true as NSNumber, forKey: NSPersistentStoreRemoteChangeNotificationPostOptionKey)
        
        container.loadPersistentStores { description, error in
            if let error = error {
                fatalError("Failed to load: \(error)")
            }
        }
        
        // Configure context
        container.viewContext.automaticallyMergesChangesFromParent = true
        container.viewContext.mergePolicy = NSMergeByPropertyObjectTrumpMergePolicy
        
        // Setup remote change notification
        setupRemoteChangeNotification()
    }
    
    private func setupRemoteChangeNotification() {
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(handleRemoteChange),
            name: .NSPersistentStoreRemoteChange,
            object: container.persistentStoreCoordinator
        )
    }
    
    @objc private func handleRemoteChange(notification: Notification) {
        // Handle remote changes from CloudKit
        let context = container.viewContext
        context.perform {
            // Process history
            self.processHistory(in: context)
        }
    }
    
    private func processHistory(in context: NSManagedObjectContext) {
        let lastToken = UserDefaults.standard.data(forKey: "lastHistoryToken")
        
        let request = NSPersistentHistoryChangeRequest.fetchHistory(
            after: lastToken.flatMap { try? NSKeyedUnarchiver.unarchivedObject(ofClass: NSPersistentHistoryToken.self, from: $0) }
        )
        
        do {
            let result = try context.execute(request) as? NSPersistentHistoryResult
            guard let transactions = result?.result as? [NSPersistentHistoryTransaction] else { return }
            
            var newObjectIDs: Set<NSManagedObjectID> = []
            
            for transaction in transactions {
                for change in transaction.changes ?? [] {
                    switch change.changeType {
                    case .insert:
                        newObjectIDs.insert(change.changedObjectID)
                    case .update:
                        newObjectIDs.insert(change.changedObjectID)
                    case .delete:
                        break
                    @unknown default:
                        break
                    }
                }
            }
            
            // Save last token
            if let lastTransaction = transactions.last {
                let tokenData = try? NSKeyedArchiver.archivedData(
                    withRootObject: lastTransaction.token,
                    requiringSecureCoding: true
                )
                UserDefaults.standard.set(tokenData, forKey: "lastHistoryToken")
            }
            
        } catch {
            print("History processing error: \(error)")
        }
    }
}

// ข้อจำกัดของ CloudKit sync:
// 1. ไม่รองรับ transformable attributes (ยกเว้น NSSecureCoding)
// 2. ไม่รองรับ unique constraints
// 3. ไม่รองรับ ordered relationships
// 4. ต้องการ identifier attribute (UUID) ทุก entity
```

---

## Practical Exercises

### Exercise 1: Task Manager

```swift
import CoreData
import SwiftUI

// สร้าง Task Manager แบบง่าย

// MARK: - Model
// Entity: Task
//   - id: UUID
//   - title: String
//   - details: String (optional)
//   - dueDate: Date (optional)
//   - priority: Int16 (1=low, 2=medium, 3=high)
//   - isCompleted: Boolean
//   - completedAt: Date (optional)
//   - createdAt: Date

// MARK: - Repository
class TaskRepository {
    
    let context: NSManagedObjectContext
    
    init(context: NSManagedObjectContext) {
        self.context = context
    }
    
    // Create
    @discardableResult
    func createTask(
        title: String,
        details: String? = nil,
        dueDate: Date? = nil,
        priority: Int = 2
    ) throws -> Task {
        let task = Task(context: context)
        task.id = UUID()
        task.title = title
        task.details = details
        task.dueDate = dueDate
        task.priority = Int16(priority)
        task.isCompleted = false
        task.createdAt = Date()
        
        try context.save()
        return task
    }
    
    // Read
    func fetchAll(showCompleted: Bool = true) throws -> [Task] {
        let request: NSFetchRequest<Task> = Task.fetchRequest()
        
        if !showCompleted {
            request.predicate = NSPredicate(format: "isCompleted == NO")
        }
        
        request.sortDescriptors = [
            NSSortDescriptor(key: "isCompleted", ascending: true),
            NSSortDescriptor(key: "priority", ascending: false),
            NSSortDescriptor(key: "dueDate", ascending: true)
        ]
        
        return try context.fetch(request)
    }
    
    func fetchOverdue() throws -> [Task] {
        let request: NSFetchRequest<Task> = Task.fetchRequest()
        request.predicate = NSPredicate(
            format: "isCompleted == NO AND dueDate < %@",
            Date() as CVarArg
        )
        request.sortDescriptors = [NSSortDescriptor(key: "dueDate", ascending: true)]
        return try context.fetch(request)
    }
    
    func fetchByPriority(_ priority: Int) throws -> [Task] {
        let request: NSFetchRequest<Task> = Task.fetchRequest()
        request.predicate = NSPredicate(format: "priority == %d AND isCompleted == NO", priority)
        request.sortDescriptors = [NSSortDescriptor(key: "dueDate", ascending: true)]
        return try context.fetch(request)
    }
    
    // Update
    func updateTask(_ task: Task, title: String? = nil, details: String? = nil) throws {
        if let title = title { task.title = title }
        if let details = details { task.details = details }
        try context.save()
    }
    
    func completeTask(_ task: Task) throws {
        task.isCompleted = true
        task.completedAt = Date()
        try context.save()
    }
    
    func uncompleteTask(_ task: Task) throws {
        task.isCompleted = false
        task.completedAt = nil
        try context.save()
    }
    
    // Delete
    func deleteTask(_ task: Task) throws {
        context.delete(task)
        try context.save()
    }
    
    func deleteCompletedTasks() throws {
        let request: NSFetchRequest<NSFetchRequestResult> = NSFetchRequest(entityName: "Task")
        request.predicate = NSPredicate(format: "isCompleted == YES")
        
        let deleteRequest = NSBatchDeleteRequest(fetchRequest: request)
        deleteRequest.resultType = .resultTypeObjectIDs
        
        let result = try context.execute(deleteRequest) as? NSBatchDeleteResult
        let objectIDs = result?.result as? [NSManagedObjectID] ?? []
        
        NSManagedObjectContext.mergeChanges(
            fromRemoteContextSave: [NSDeletedObjectsKey: objectIDs],
            into: [context]
        )
    }
    
    // Statistics
    func getStats() throws -> (total: Int, completed: Int, overdue: Int) {
        let total = try context.count(for: Task.fetchRequest())
        
        let completedRequest: NSFetchRequest<Task> = Task.fetchRequest()
        completedRequest.predicate = NSPredicate(format: "isCompleted == YES")
        let completed = try context.count(for: completedRequest)
        
        let overdueRequest: NSFetchRequest<Task> = Task.fetchRequest()
        overdueRequest.predicate = NSPredicate(
            format: "isCompleted == NO AND dueDate < %@",
            Date() as CVarArg
        )
        let overdue = try context.count(for: overdueRequest)
        
        return (total, completed, overdue)
    }
}
```

### Exercise 2: Contact Book

```swift
import CoreData

// สร้าง Contact Book

// Entities:
// Contact: id, firstName, lastName, email, phone, birthday, notes, photo, createdAt
// ContactGroup: id, name, color, contacts (relationship)

class ContactBookManager {
    
    let context: NSManagedObjectContext
    
    init(context: NSManagedObjectContext) {
        self.context = context
    }
    
    // MARK: - Contact Operations
    
    func addContact(
        firstName: String,
        lastName: String,
        email: String? = nil,
        phone: String? = nil,
        birthday: Date? = nil
    ) throws -> Contact {
        let contact = Contact(context: context)
        contact.id = UUID()
        contact.firstName = firstName
        contact.lastName = lastName
        contact.email = email
        contact.phone = phone
        contact.birthday = birthday
        contact.createdAt = Date()
        
        try context.save()
        return contact
    }
    
    func searchContacts(query: String) throws -> [Contact] {
        let request: NSFetchRequest<Contact> = Contact.fetchRequest()
        
        if !query.isEmpty {
            request.predicate = NSCompoundPredicate(orPredicateWithSubpredicates: [
                NSPredicate(format: "firstName CONTAINS[cd] %@", query),
                NSPredicate(format: "lastName CONTAINS[cd] %@", query),
                NSPredicate(format: "email CONTAINS[cd] %@", query),
                NSPredicate(format: "phone CONTAINS[cd] %@", query)
            ])
        }
        
        request.sortDescriptors = [
            NSSortDescriptor(key: "lastName", ascending: true),
            NSSortDescriptor(key: "firstName", ascending: true)
        ]
        
        return try context.fetch(request)
    }
    
    func fetchContactsGrouped() throws -> [String: [Contact]] {
        let contacts = try searchContacts(query: "")
        var grouped: [String: [Contact]] = [:]
        
        for contact in contacts {
            let key = String(contact.lastName?.prefix(1) ?? "#").uppercased()
            grouped[key, default: []].append(contact)
        }
        
        return grouped
    }
    
    func fetchBirthdays(inMonth month: Int) throws -> [Contact] {
        let request: NSFetchRequest<Contact> = Contact.fetchRequest()
        request.predicate = NSPredicate(
            format: "birthday != nil"
        )
        
        let contacts = try context.fetch(request)
        
        let calendar = Calendar.current
        return contacts.filter { contact in
            guard let birthday = contact.birthday else { return false }
            return calendar.component(.month, from: birthday) == month
        }
    }
    
    // MARK: - Group Operations
    
    func createGroup(name: String, color: String = "#007AFF") throws -> ContactGroup {
        let group = ContactGroup(context: context)
        group.id = UUID()
        group.name = name
        group.color = color
        
        try context.save()
        return group
    }
    
    func addContact(_ contact: Contact, toGroup group: ContactGroup) throws {
        group.addToContacts(contact)
        try context.save()
    }
    
    func fetchContactsInGroup(_ group: ContactGroup) -> [Contact] {
        let contactSet = group.contacts as? Set<Contact> ?? []
        return contactSet.sorted {
            let name1 = "\($0.lastName ?? "") \($0.firstName ?? "")"
            let name2 = "\($1.lastName ?? "") \($1.firstName ?? "")"
            return name1 < name2
        }
    }
    
    // MARK: - Import/Export
    
    func importContacts(from jsonData: Data) throws {
        struct ImportContact: Codable {
            var firstName: String
            var lastName: String
            var email: String?
            var phone: String?
        }
        
        let imported = try JSONDecoder().decode([ImportContact].self, from: jsonData)
        
        for item in imported {
            let contact = Contact(context: context)
            contact.id = UUID()
            contact.firstName = item.firstName
            contact.lastName = item.lastName
            contact.email = item.email
            contact.phone = item.phone
            contact.createdAt = Date()
        }
        
        try context.save()
        print("Imported \(imported.count) contacts")
    }
}
```

---

## Building a Notes App

### สร้าง Notes App สมบูรณ์แบบ

```swift
import SwiftUI
import CoreData

// MARK: - Persistence Controller

class NotesPersistenceController {
    
    static let shared = NotesPersistenceController()
    static let preview = NotesPersistenceController(inMemory: true)
    
    let container: NSPersistentContainer
    
    init(inMemory: Bool = false) {
        container = NSPersistentContainer(name: "NotesApp")
        
        if inMemory {
            container.persistentStoreDescriptions.first!.url = URL(fileURLWithPath: "/dev/null")
        }
        
        container.loadPersistentStores { _, error in
            if let error = error {
                fatalError("Core Data failed: \(error)")
            }
        }
        
        container.viewContext.automaticallyMergesChangesFromParent = true
        
        if inMemory {
            createSampleData()
        }
    }
    
    private func createSampleData() {
        let context = container.viewContext
        let sampleNotes = [
            ("การเรียน Swift", "Swift เป็นภาษาที่ทันสมัยและมีประสิทธิภาพสูง"),
            ("รายการช้อปปิ้ง", "นม, ไข่, ขนมปัง, ผักสลัด"),
            ("ไอเดียโปรเจค", "สร้าง app บันทึกค่าใช้จ่าย พร้อม visualization"),
            ("Meeting Notes", "ประชุม 10:00 - นำเสนอ Q4 roadmap"),
            ("คำคม", "\"Code is like humor. When you have to explain it, it's bad.\" - Cory House")
        ]
        
        for (i, (title, content)) in sampleNotes.enumerated() {
            let note = Note(context: context)
            note.id = UUID()
            note.title = title
            note.content = content
            note.createdAt = Date().addingTimeInterval(-Double(i) * 3600 * 24)
            note.updatedAt = Date().addingTimeInterval(-Double(i) * 3600)
            note.isPinned = i == 0
        }
        
        try? context.save()
    }
    
    var viewContext: NSManagedObjectContext {
        container.viewContext
    }
    
    func save() {
        guard viewContext.hasChanges else { return }
        try? viewContext.save()
    }
}

// MARK: - Main App

@main
struct NotesAppMain: App {
    
    let persistence = NotesPersistenceController.shared
    
    var body: some Scene {
        WindowGroup {
            NotesContentView()
                .environment(\.managedObjectContext, persistence.viewContext)
        }
    }
}

// MARK: - Content View

struct NotesContentView: View {
    
    @Environment(\.managedObjectContext) private var viewContext
    @State private var showingNewNote = false
    @State private var searchText = ""
    
    var body: some View {
        NavigationView {
            NoteListContainer(searchText: searchText)
                .navigationTitle("บันทึก")
                .toolbar {
                    ToolbarItemGroup(placement: .navigationBarTrailing) {
                        Button {
                            showingNewNote = true
                        } label: {
                            Image(systemName: "square.and.pencil")
                        }
                    }
                }
                .searchable(text: $searchText, prompt: "ค้นหาบันทึก")
        }
        .sheet(isPresented: $showingNewNote) {
            NewNoteView()
        }
    }
}

// MARK: - Note List

struct NoteListContainer: View {
    
    let searchText: String
    
    @FetchRequest var notes: FetchedResults<Note>
    @Environment(\.managedObjectContext) private var viewContext
    
    init(searchText: String) {
        let sortDescriptors = [
            NSSortDescriptor(key: "isPinned", ascending: false),
            NSSortDescriptor(key: "updatedAt", ascending: false)
        ]
        
        let predicate: NSPredicate? = searchText.isEmpty ? nil :
            NSPredicate(format: "title CONTAINS[cd] %@ OR content CONTAINS[cd] %@",
                       searchText, searchText)
        
        _notes = FetchRequest(
            sortDescriptors: sortDescriptors,
            predicate: predicate,
            animation: .default
        )
    }
    
    var body: some View {
        Group {
            if notes.isEmpty {
                EmptyNotesView()
            } else {
                List {
                    ForEach(notes) { note in
                        NavigationLink(destination: NoteDetailView(note: note)) {
                            NoteRowView(note: note)
                        }
                    }
                    .onDelete(perform: deleteNotes)
                }
            }
        }
    }
    
    private func deleteNotes(offsets: IndexSet) {
        withAnimation {
            offsets.map { notes[$0] }.forEach(viewContext.delete)
            try? viewContext.save()
        }
    }
}

// MARK: - Note Detail View

struct NoteDetailView: View {
    
    @ObservedObject var note: Note
    @Environment(\.managedObjectContext) private var viewContext
    @Environment(\.dismiss) private var dismiss
    
    @State private var title: String
    @State private var content: String
    @State private var isEditing = false
    
    init(note: Note) {
        self.note = note
        _title = State(initialValue: note.title ?? "")
        _content = State(initialValue: note.content ?? "")
    }
    
    var body: some View {
        ScrollView {
            VStack(alignment: .leading, spacing: 16) {
                if isEditing {
                    TextField("ชื่อบันทึก", text: $title)
                        .font(.title)
                        .fontWeight(.bold)
                    
                    TextEditor(text: $content)
                        .frame(minHeight: 300)
                } else {
                    Text(note.title ?? "ไม่มีชื่อ")
                        .font(.title)
                        .fontWeight(.bold)
                    
                    if let updatedAt = note.updatedAt {
                        Text("แก้ไขล่าสุด: \(updatedAt, style: .relative)")
                            .font(.caption)
                            .foregroundColor(.secondary)
                    }
                    
                    Divider()
                    
                    Text(note.content ?? "ไม่มีเนื้อหา")
                        .font(.body)
                }
            }
            .padding()
        }
        .navigationBarTitleDisplayMode(.inline)
        .toolbar {
            ToolbarItemGroup(placement: .navigationBarTrailing) {
                if isEditing {
                    Button("บันทึก") {
                        saveChanges()
                        isEditing = false
                    }
                    .fontWeight(.semibold)
                } else {
                    Button {
                        togglePin()
                    } label: {
                        Image(systemName: note.isPinned ? "pin.fill" : "pin")
                    }
                    
                    Button("แก้ไข") {
                        isEditing = true
                    }
                }
            }
        }
    }
    
    private func saveChanges() {
        note.title = title
        note.content = content
        note.updatedAt = Date()
        try? viewContext.save()
    }
    
    private func togglePin() {
        note.isPinned.toggle()
        note.updatedAt = Date()
        try? viewContext.save()
    }
}

// MARK: - New Note View

struct NewNoteView: View {
    
    @Environment(\.managedObjectContext) private var viewContext
    @Environment(\.dismiss) private var dismiss
    
    @State private var title = ""
    @State private var content = ""
    
    var body: some View {
        NavigationView {
            Form {
                Section("ชื่อบันทึก") {
                    TextField("ชื่อบันทึก", text: $title)
                }
                
                Section("เนื้อหา") {
                    TextEditor(text: $content)
                        .frame(minHeight: 200)
                }
            }
            .navigationTitle("บันทึกใหม่")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .navigationBarLeading) {
                    Button("ยกเลิก") {
                        dismiss()
                    }
                }
                
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button("บันทึก") {
                        createNote()
                    }
                    .disabled(title.isEmpty)
                    .fontWeight(.semibold)
                }
            }
        }
    }
    
    private func createNote() {
        let note = Note(context: viewContext)
        note.id = UUID()
        note.title = title
        note.content = content.isEmpty ? nil : content
        note.createdAt = Date()
        note.updatedAt = Date()
        note.isPinned = false
        
        try? viewContext.save()
        dismiss()
    }
}

// MARK: - Empty State View

struct EmptyNotesView: View {
    var body: some View {
        VStack(spacing: 16) {
            Image(systemName: "note.text")
                .font(.system(size: 60))
                .foregroundColor(.secondary)
            
            Text("ยังไม่มีบันทึก")
                .font(.title2)
                .fontWeight(.semibold)
            
            Text("แตะปุ่ม + เพื่อสร้างบันทึกใหม่")
                .font(.subheadline)
                .foregroundColor(.secondary)
        }
        .frame(maxWidth: .infinity, maxHeight: .infinity)
    }
}
```

---

## Summary

### สรุปหัวข้อที่เรียนรู้

1. **Core Data Stack**
   - `NSPersistentContainer`: จัดการ stack ทั้งหมด
   - `NSManagedObjectContext`: workspace สำหรับทำงานกับ objects
   - `NSManagedObjectModel`: schema ของ data
   - `NSPersistentStoreCoordinator`: เชื่อมระหว่าง model และ store

2. **Data Model**
   - สร้างผ่าน Xcode Data Model Editor
   - Entities, Attributes, Relationships
   - Attribute types: string, number, date, boolean, binary, UUID, transformable

3. **Relationships**
   - One-to-One: ใช้ optional property
   - One-to-Many: ใช้ NSSet
   - Many-to-Many: ใช้ NSSet ทั้งสองฝั่ง
   - Inverse relationships: บังคับมีเสมอ
   - Delete rules: cascade, nullify, deny, no action

4. **NSManagedObject**
   - `@NSManaged` property wrapper
   - Codegen: Class Definition, Category/Extension, Manual
   - Custom validation ใน `validateForInsert/Update`

5. **CRUD Operations**
   - Create: `NSManagedObject(context:)`
   - Read: `NSFetchRequest`
   - Update: เปลี่ยน property แล้ว save
   - Delete: `context.delete(_:)`
   - Batch operations สำหรับ performance

6. **Querying**
   - `NSPredicate`: filter ด้วย format string
   - `NSSortDescriptor`: sort ผลลัพธ์
   - Fetch options: limit, offset, batch size

7. **Background Processing**
   - `container.performBackgroundTask`
   - `container.newBackgroundContext()`
   - Context merging

8. **SwiftUI Integration**
   - `@FetchRequest` property wrapper
   - Dynamic predicates
   - `@Environment(\.managedObjectContext)`

9. **Migration**
   - Lightweight migration: อัตโนมัติ
   - Custom migration: Mapping Model

### Best Practices

```swift
// ✅ ใช้ NSPersistentContainer (iOS 10+)
let container = NSPersistentContainer(name: "MyApp")

// ✅ Enable automaticallyMergesChangesFromParent
container.viewContext.automaticallyMergesChangesFromParent = true

// ✅ ตั้ง mergePolicy
container.viewContext.mergePolicy = NSMergeByPropertyObjectTrumpMergePolicy

// ✅ ใช้ background context สำหรับ heavy operations
container.performBackgroundTask { context in
    // heavy work here
}

// ✅ ตรวจ hasChanges ก่อน save
if context.hasChanges {
    try context.save()
}

// ✅ ใช้ batch operations สำหรับ large datasets
let batchDelete = NSBatchDeleteRequest(fetchRequest: request)

// ✅ ใช้ fetchBatchSize สำหรับ large fetches
request.fetchBatchSize = 50

// ✅ ใช้ KeyPath สำหรับ type-safe sort descriptors
NSSortDescriptor(keyPath: \Note.createdAt, ascending: false)

// ❌ อย่า save จาก background thread ไปยัง main context โดยตรง
// ใช้ automaticallyMergesChangesFromParent แทน
```

### Architecture Patterns

```
MVVM + Core Data:
┌─────────────────┐    ┌──────────────────┐    ┌────────────────┐
│   SwiftUI View  │ ←→ │   ViewModel      │ ←→ │  Repository    │
│                 │    │  @ObservableObj  │    │  (Core Data)   │
│  @FetchRequest  │    │  @Published      │    │                │
└─────────────────┘    └──────────────────┘    └────────────────┘

Repository Pattern:
┌─────────────────┐    ┌──────────────────┐    ┌────────────────┐
│   ViewModel     │ →  │   Repository     │ →  │  Core Data     │
│                 │    │  Protocol        │    │  Context       │
│                 │ ←  │  Implementation  │ ←  │                │
└─────────────────┘    └──────────────────┘    └────────────────┘
```

### Quick Reference

```swift
// Setup
let container = NSPersistentContainer(name: "App")
container.loadPersistentStores { _, error in }
let context = container.viewContext

// Create
let obj = MyEntity(context: context)
obj.attribute = value
try context.save()

// Read
let request: NSFetchRequest<MyEntity> = MyEntity.fetchRequest()
request.predicate = NSPredicate(format: "attribute == %@", value)
request.sortDescriptors = [NSSortDescriptor(key: "name", ascending: true)]
let results = try context.fetch(request)

// Update
obj.attribute = newValue
try context.save()

// Delete
context.delete(obj)
try context.save()

// Background
container.performBackgroundTask { bgContext in
    // work here
    try bgContext.save()
}

// SwiftUI
@FetchRequest(sortDescriptors: [NSSortDescriptor(keyPath: \Note.date, ascending: false)])
private var notes: FetchedResults<Note>
```

ในบทถัดไปเราจะเรียนรู้เกี่ยวกับ **Networking** และการทำงานกับ **URLSession** เพื่อดึงข้อมูลจาก API

---

*จบ Part 32: Core Data Basics ใน Swift*
