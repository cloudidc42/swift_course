# Part 43: CloudKit - การจัดเก็บข้อมูลบน iCloud

## บทนำ

CloudKit เป็น Framework ของ Apple ที่ช่วยให้นักพัฒนา iOS, macOS, watchOS และ tvOS สามารถจัดเก็บและซิงค์ข้อมูลบน iCloud ได้อย่างง่ายดาย โดยไม่ต้องสร้าง Backend Server เอง Apple จัดการ Infrastructure ทั้งหมดให้ รวมถึงการ Authentication ผ่าน Apple ID ของผู้ใช้

ในบทนี้เราจะเรียนรู้:
- พื้นฐานของ CloudKit และ Architecture
- การทำงานกับ CKRecord, CKQuery, CKSubscription
- การใช้ CKOperation สำหรับ Batch Operations
- การจัดการ Assets และ References
- Zone-based Storage และการ Share Records
- การรวม CloudKit กับ Core Data
- Best Practices และการ Debug

---

## 43.1 What is CloudKit?

CloudKit คือ Backend-as-a-Service (BaaS) ที่ Apple สร้างขึ้นสำหรับ Ecosystem ของตนเอง เปิดตัวครั้งแรกใน iOS 8 (2014) และมีการพัฒนาต่อเนื่องมาจนถึงปัจจุบัน

### คุณสมบัติหลักของ CloudKit

```
CloudKit Features:
├── Free Tier ที่ใจกว้าง (10GB Storage, 2GB transfer/month สำหรับ iCloud Free Tier)
├── Authentication ผ่าน Apple ID อัตโนมัติ
├── Public และ Private Database แยกกัน
├── Real-time Subscription ผ่าน Push Notification
├── Asset Storage สำหรับไฟล์ขนาดใหญ่
├── Record Sharing ระหว่างผู้ใช้
└── CloudKit Dashboard สำหรับ Debug
```

### ทำไมต้องใช้ CloudKit?

**ข้อดี:**
1. **ไม่ต้องมี Server** - Apple จัดการ Infrastructure ให้ทั้งหมด
2. **ฟรี** - ใช้ฟรีในโควต้าที่กำหนด และขยายได้ตามการใช้งาน
3. **Privacy** - ข้อมูล Private Database เข้าถึงได้เฉพาะเจ้าของ Apple ID
4. **Integration** - รวมกับ Ecosystem ของ Apple ได้ดีมาก
5. **Core Data Integration** - NSPersistentCloudKitContainer ใช้งานง่าย

**ข้อจำกัด:**
1. **Apple Only** - ใช้ได้เฉพาะ Platform ของ Apple
2. **ต้องมี Apple ID** - ผู้ใช้ต้อง Login iCloud
3. **ไม่ยืดหยุ่นเท่า Firebase** - Schema ต้องกำหนดเอง

---

## 43.2 CloudKit Containers

Container คือ Namespace หลักที่ใช้แบ่งแยกข้อมูลระหว่าง App ต่างๆ แต่ละ App จะมี Default Container ที่ตั้งชื่อตาม Bundle ID ของ App

### Container Identifier

Format: `iCloud.<bundle_identifier>`

ตัวอย่าง: `iCloud.com.example.myapp`

### การตั้งค่า CloudKit ใน Xcode

1. เปิด Project Settings
2. ไปที่ tab **Signing & Capabilities**
3. คลิก **+ Capability**
4. เลือก **iCloud**
5. เช็ค **CloudKit**
6. เลือก Container หรือสร้างใหม่

### การเข้าถึง Container ใน Code

```swift
import CloudKit

// Default container (ใช้ Bundle ID ของ App)
let defaultContainer = CKContainer.default()

// Custom container
let customContainer = CKContainer(identifier: "iCloud.com.example.myapp")

// ตรวจสอบสถานะ iCloud Account
defaultContainer.accountStatus { status, error in
    switch status {
    case .available:
        print("iCloud พร้อมใช้งาน")
    case .noAccount:
        print("ผู้ใช้ยังไม่ได้ Login iCloud")
    case .restricted:
        print("iCloud ถูกจำกัดโดย MDM หรือ Parental Control")
    case .couldNotDetermine:
        print("ไม่สามารถตรวจสอบสถานะได้")
    case .temporarilyUnavailable:
        print("iCloud ไม่พร้อมใช้งานชั่วคราว")
    @unknown default:
        print("สถานะที่ไม่รู้จัก")
    }
}
```

### การดึง User Record ID

```swift
// ดึง User's Record ID (ไม่เปิดเผย Apple ID จริง)
defaultContainer.fetchUserRecordID { recordID, error in
    if let error = error {
        print("Error: \(error.localizedDescription)")
        return
    }
    
    if let recordID = recordID {
        print("User Record ID: \(recordID.recordName)")
        // ใช้เป็น unique identifier สำหรับ User
    }
}
```

---

## 43.3 Public, Private, and Shared Databases

CloudKit Container มี Database 3 ประเภท:

### 43.3.1 Public Database

```swift
let publicDB = CKContainer.default().publicCloudDatabase
```

**คุณสมบัติ:**
- ทุกคนอ่านได้ ไม่ต้อง Login
- เขียนได้เฉพาะผู้ที่ Login iCloud
- ข้อมูลนับรวมกับ Developer Quota ไม่ใช่ User
- เหมาะสำหรับ: Content สาธารณะ, Leaderboard, Public Posts

### 43.3.2 Private Database

```swift
let privateDB = CKContainer.default().privateCloudDatabase
```

**คุณสมบัติ:**
- เข้าถึงได้เฉพาะเจ้าของ Apple ID
- ข้อมูลนับรวมกับ User's iCloud Storage
- Encrypted at rest
- เหมาะสำหรับ: Personal Data, Notes, Settings

### 43.3.3 Shared Database

```swift
let sharedDB = CKContainer.default().sharedCloudDatabase
```

**คุณสมบัติ:**
- สำหรับ Records ที่ถูก Share จากผู้ใช้คนอื่น
- ต้องมี CKShare เพื่อเข้าถึง
- iOS 10+ เท่านั้น

### ตัวอย่างการเลือก Database ที่เหมาะสม

```swift
class DatabaseSelector {
    private let container = CKContainer.default()
    
    // ข้อมูลส่วนตัว - Private Database
    var userNotes: CKDatabase {
        return container.privateCloudDatabase
    }
    
    // ข้อมูลสาธารณะ - Public Database
    var publicAnnouncements: CKDatabase {
        return container.publicCloudDatabase
    }
    
    // Records ที่รับการ Share - Shared Database
    var sharedContent: CKDatabase {
        return container.sharedCloudDatabase
    }
}
```

---

## 43.4 CKRecord

CKRecord คือหน่วยข้อมูลพื้นฐานใน CloudKit คล้ายกับ Row ใน Database หรือ Document ใน NoSQL

### โครงสร้างของ CKRecord

```swift
// CKRecord มี Properties พื้นฐาน:
// - recordID: CKRecord.ID (unique identifier)
// - recordType: String (ชื่อ Type)
// - creationDate: Date?
// - creatorUserRecordID: CKRecord.ID?
// - modificationDate: Date?
// - lastModifiedUserRecordID: CKRecord.ID?
// - recordChangeTag: String? (สำหรับ Conflict Detection)
```

### การสร้าง CKRecord

```swift
import CloudKit

// สร้าง Record แบบง่าย
let noteRecord = CKRecord(recordType: "Note")
noteRecord["title"] = "บันทึกแรกของฉัน" as CKRecordValue
noteRecord["content"] = "นี่คือเนื้อหาของบันทึก" as CKRecordValue
noteRecord["createdAt"] = Date() as CKRecordValue
noteRecord["isPublic"] = false as CKRecordValue

// สร้าง Record ด้วย Custom Record ID
let recordID = CKRecord.ID(recordName: "unique-note-id-123")
let noteWithCustomID = CKRecord(recordType: "Note", recordID: recordID)
noteWithCustomID["title"] = "บันทึกพิเศษ" as CKRecordValue

// สร้าง Record ใน Specific Zone
let zoneID = CKRecordZone.ID(zoneName: "NotesZone", ownerName: CKCurrentUserDefaultName)
let noteInZone = CKRecord(
    recordType: "Note",
    recordID: CKRecord.ID(recordName: "note-in-zone", zoneID: zoneID)
)
```

### การบันทึก CKRecord

```swift
class CloudKitManager {
    private let privateDB = CKContainer.default().privateCloudDatabase
    
    func saveNote(title: String, content: String, completion: @escaping (Result<CKRecord, Error>) -> Void) {
        let record = CKRecord(recordType: "Note")
        record["title"] = title as CKRecordValue
        record["content"] = content as CKRecordValue
        record["createdAt"] = Date() as CKRecordValue
        record["wordCount"] = content.split(separator: " ").count as CKRecordValue
        
        privateDB.save(record) { savedRecord, error in
            DispatchQueue.main.async {
                if let error = error {
                    completion(.failure(error))
                } else if let savedRecord = savedRecord {
                    completion(.success(savedRecord))
                }
            }
        }
    }
}

// การใช้งาน
let manager = CloudKitManager()
manager.saveNote(title: "Shopping List", content: "นม ไข่ ขนมปัง") { result in
    switch result {
    case .success(let record):
        print("บันทึกสำเร็จ: \(record.recordID.recordName)")
    case .failure(let error):
        print("เกิดข้อผิดพลาด: \(error.localizedDescription)")
    }
}
```

### การอ่าน CKRecord

```swift
func fetchNote(recordID: CKRecord.ID, completion: @escaping (Result<CKRecord, Error>) -> Void) {
    let privateDB = CKContainer.default().privateCloudDatabase
    
    privateDB.fetch(withRecordID: recordID) { record, error in
        DispatchQueue.main.async {
            if let error = error {
                completion(.failure(error))
            } else if let record = record {
                completion(.success(record))
            }
        }
    }
}

// การอ่านค่าจาก Record
func displayNote(_ record: CKRecord) {
    let title = record["title"] as? String ?? "ไม่มีชื่อ"
    let content = record["content"] as? String ?? ""
    let createdAt = record["createdAt"] as? Date
    let wordCount = record["wordCount"] as? Int ?? 0
    
    print("ชื่อ: \(title)")
    print("เนื้อหา: \(content)")
    print("จำนวนคำ: \(wordCount)")
    if let date = createdAt {
        print("สร้างเมื่อ: \(date)")
    }
}
```

### การอัปเดต CKRecord

```swift
func updateNote(recordID: CKRecord.ID, newTitle: String, completion: @escaping (Result<CKRecord, Error>) -> Void) {
    let privateDB = CKContainer.default().privateCloudDatabase
    
    // ต้อง Fetch ก่อนแล้วค่อย Save เพื่อหลีกเลี่ยง Conflict
    privateDB.fetch(withRecordID: recordID) { record, error in
        guard let record = record, error == nil else {
            DispatchQueue.main.async {
                completion(.failure(error ?? NSError(domain: "CloudKit", code: -1)))
            }
            return
        }
        
        // อัปเดตค่า
        record["title"] = newTitle as CKRecordValue
        record["modifiedAt"] = Date() as CKRecordValue
        
        // บันทึกกลับ
        privateDB.save(record) { updatedRecord, saveError in
            DispatchQueue.main.async {
                if let saveError = saveError {
                    completion(.failure(saveError))
                } else if let updatedRecord = updatedRecord {
                    completion(.success(updatedRecord))
                }
            }
        }
    }
}
```

### การลบ CKRecord

```swift
func deleteNote(recordID: CKRecord.ID, completion: @escaping (Result<Void, Error>) -> Void) {
    let privateDB = CKContainer.default().privateCloudDatabase
    
    privateDB.delete(withRecordID: recordID) { deletedID, error in
        DispatchQueue.main.async {
            if let error = error {
                completion(.failure(error))
            } else {
                completion(.success(()))
            }
        }
    }
}
```

---

## 43.5 CKRecordType

CKRecordType คือ String ที่ระบุประเภทของ Record คล้ายกับ Table Name หรือ Class Name

### การกำหนด Record Types

```swift
// ใช้ String ตรงๆ (ไม่แนะนำสำหรับ Production)
let record = CKRecord(recordType: "Note")

// แนะนำให้ใช้ Enum หรือ Constant
enum RecordType {
    static let note = "Note"
    static let tag = "Tag"
    static let user = "User"
    static let photo = "Photo"
    static let comment = "Comment"
}

let noteRecord = CKRecord(recordType: RecordType.note)
let tagRecord = CKRecord(recordType: RecordType.tag)
```

### การสร้าง Model Class สำหรับ CKRecord

```swift
// Model ที่ Wrap CKRecord
struct Note {
    let recordID: CKRecord.ID
    var title: String
    var content: String
    var createdAt: Date
    var isPinned: Bool
    var tags: [String]
    
    // Constants สำหรับ Field Names
    enum Field {
        static let title = "title"
        static let content = "content"
        static let createdAt = "createdAt"
        static let isPinned = "isPinned"
        static let tags = "tags"
    }
    
    // Initializer จาก CKRecord
    init?(record: CKRecord) {
        guard record.recordType == RecordType.note,
              let title = record[Field.title] as? String,
              let content = record[Field.content] as? String,
              let createdAt = record[Field.createdAt] as? Date else {
            return nil
        }
        
        self.recordID = record.recordID
        self.title = title
        self.content = content
        self.createdAt = createdAt
        self.isPinned = record[Field.isPinned] as? Bool ?? false
        self.tags = record[Field.tags] as? [String] ?? []
    }
    
    // แปลงกลับเป็น CKRecord
    func toCKRecord() -> CKRecord {
        let record = CKRecord(recordType: RecordType.note, recordID: recordID)
        record[Field.title] = title as CKRecordValue
        record[Field.content] = content as CKRecordValue
        record[Field.createdAt] = createdAt as CKRecordValue
        record[Field.isPinned] = isPinned as CKRecordValue
        record[Field.tags] = tags as CKRecordValue
        return record
    }
}
```

---

## 43.6 CKField Types

CloudKit รองรับ Data Types ต่อไปนี้สำหรับ Field:

### Types ที่รองรับ

```swift
// 1. String
record["name"] = "John Doe" as CKRecordValue

// 2. Int (NSNumber)
record["age"] = 25 as CKRecordValue
record["score"] = Int64(1000000) as CKRecordValue

// 3. Double/Float (NSNumber)
record["rating"] = 4.5 as CKRecordValue
record["latitude"] = 13.7563 as CKRecordValue

// 4. Date (NSDate)
record["birthDate"] = Date() as CKRecordValue

// 5. Data (NSData)
let jsonData = try? JSONEncoder().encode(["key": "value"])
record["metadata"] = jsonData as? CKRecordValue

// 6. Bool (NSNumber - 0 หรือ 1)
record["isActive"] = true as CKRecordValue

// 7. Array (NSArray ของ Types ข้างบน)
record["tags"] = ["swift", "ios", "cloudkit"] as CKRecordValue
record["scores"] = [100, 200, 150] as CKRecordValue

// 8. CLLocation (สำหรับ Geo Queries)
import CoreLocation
let location = CLLocation(latitude: 13.7563, longitude: 100.5018)
record["location"] = location as CKRecordValue

// 9. CKAsset (สำหรับ Binary Data/Files)
// (จะอธิบายในหัวข้อถัดไป)

// 10. CKRecord.Reference (สำหรับ Relationships)
// (จะอธิบายในหัวข้อถัดไป)
```

### ข้อจำกัดของ Field Types

```swift
// Field ที่ไม่รองรับโดยตรง - ต้องแปลงก่อน
struct ComplexData: Codable {
    let id: UUID
    let name: String
    let values: [Double]
}

extension ComplexData {
    // แปลงเป็น Data เพื่อเก็บใน CloudKit
    func toCKRecordValue() throws -> CKRecordValue {
        let data = try JSONEncoder().encode(self)
        return data as CKRecordValue
    }
    
    // อ่านจาก Data
    static func from(ckData: Data) throws -> ComplexData {
        return try JSONDecoder().decode(ComplexData.self, from: ckData)
    }
}

// การใช้งาน
let complexData = ComplexData(id: UUID(), name: "Test", values: [1.0, 2.0, 3.0])
if let value = try? complexData.toCKRecordValue() {
    record["complexData"] = value
}
```

---

## 43.7 CKQuery และ NSPredicate

CKQuery ใช้สำหรับค้นหา Records ใน Database โดยใช้ NSPredicate เป็นเงื่อนไข

### การสร้าง CKQuery

```swift
import CloudKit

// Query พื้นฐาน - ดึง Records ทั้งหมดของ Type นี้
let allNotesQuery = CKQuery(recordType: "Note", predicate: NSPredicate(value: true))

// Query ด้วย Predicate
let pinnedPredicate = NSPredicate(format: "isPinned == %@", true as CVarArg)
let pinnedQuery = CKQuery(recordType: "Note", predicate: pinnedPredicate)

// Query ด้วย String comparison
let titlePredicate = NSPredicate(format: "title BEGINSWITH %@", "Meeting")
let meetingQuery = CKQuery(recordType: "Note", predicate: titlePredicate)

// Query ด้วย Date range
let oneWeekAgo = Date().addingTimeInterval(-7 * 24 * 3600)
let datePredicate = NSPredicate(format: "createdAt >= %@", oneWeekAgo as CVarArg)
let recentQuery = CKQuery(recordType: "Note", predicate: datePredicate)
```

### Sort Descriptors

```swift
// เรียงลำดับตาม createdAt (ใหม่สุดก่อน)
let sortDescriptor = NSSortDescriptor(key: "createdAt", ascending: false)
allNotesQuery.sortDescriptors = [sortDescriptor]

// เรียงลำดับหลายระดับ
let titleSort = NSSortDescriptor(key: "title", ascending: true)
let dateSort = NSSortDescriptor(key: "createdAt", ascending: false)
allNotesQuery.sortDescriptors = [titleSort, dateSort]
```

### การ Execute Query

```swift
class NoteRepository {
    private let db = CKContainer.default().privateCloudDatabase
    
    // Fetch ด้วย Convenience API
    func fetchAllNotes(completion: @escaping ([Note]?, Error?) -> Void) {
        let predicate = NSPredicate(value: true)
        let query = CKQuery(recordType: RecordType.note, predicate: predicate)
        query.sortDescriptors = [NSSortDescriptor(key: "createdAt", ascending: false)]
        
        db.perform(query, inZoneWith: nil) { records, error in
            DispatchQueue.main.async {
                if let error = error {
                    completion(nil, error)
                    return
                }
                
                let notes = records?.compactMap { Note(record: $0) } ?? []
                completion(notes, nil)
            }
        }
    }
    
    // Fetch Notes ที่มี Tag เฉพาะ
    func fetchNotes(withTag tag: String, completion: @escaping ([Note]?, Error?) -> Void) {
        let predicate = NSPredicate(format: "tags CONTAINS %@", tag)
        let query = CKQuery(recordType: RecordType.note, predicate: predicate)
        
        db.perform(query, inZoneWith: nil) { records, error in
            DispatchQueue.main.async {
                if let error = error {
                    completion(nil, error)
                    return
                }
                
                let notes = records?.compactMap { Note(record: $0) } ?? []
                completion(notes, nil)
            }
        }
    }
    
    // Fetch Notes ที่ถูก Pin
    func fetchPinnedNotes(completion: @escaping ([Note]?, Error?) -> Void) {
        let predicate = NSPredicate(format: "isPinned == %@", true as CVarArg)
        let query = CKQuery(recordType: RecordType.note, predicate: predicate)
        
        db.perform(query, inZoneWith: nil) { records, error in
            DispatchQueue.main.async {
                completion(records?.compactMap { Note(record: $0) }, error)
            }
        }
    }
}
```

### Pagination ด้วย CKQueryCursor

```swift
class PaginatedNoteRepository {
    private let db = CKContainer.default().privateCloudDatabase
    private let pageSize = 20
    
    func fetchNotes(
        cursor: CKQueryOperation.Cursor? = nil,
        completion: @escaping ([CKRecord], CKQueryOperation.Cursor?, Error?) -> Void
    ) {
        let operation: CKQueryOperation
        
        if let cursor = cursor {
            // ดึงหน้าถัดไปด้วย Cursor
            operation = CKQueryOperation(cursor: cursor)
        } else {
            // ดึงหน้าแรก
            let predicate = NSPredicate(value: true)
            let query = CKQuery(recordType: "Note", predicate: predicate)
            query.sortDescriptors = [NSSortDescriptor(key: "createdAt", ascending: false)]
            operation = CKQueryOperation(query: query)
        }
        
        operation.resultsLimit = pageSize
        
        var fetchedRecords: [CKRecord] = []
        
        operation.recordMatchedBlock = { _, result in
            switch result {
            case .success(let record):
                fetchedRecords.append(record)
            case .failure(let error):
                print("Record error: \(error)")
            }
        }
        
        operation.queryResultBlock = { result in
            DispatchQueue.main.async {
                switch result {
                case .success(let cursor):
                    completion(fetchedRecords, cursor, nil)
                case .failure(let error):
                    completion(fetchedRecords, nil, error)
                }
            }
        }
        
        db.add(operation)
    }
}
```

### NSPredicate ที่รองรับใน CloudKit

```swift
// Comparison operators
NSPredicate(format: "age > %d", 18)
NSPredicate(format: "score >= %d", 100)
NSPredicate(format: "name == %@", "John")
NSPredicate(format: "name != %@", "Admin")

// String operations
NSPredicate(format: "title BEGINSWITH %@", "Meeting")
NSPredicate(format: "title CONTAINS %@", "important")
// หมายเหตุ: ENDSWITH ไม่รองรับใน CloudKit

// Array containment
NSPredicate(format: "tags CONTAINS %@", "swift")

// Location-based (ต้องสร้าง Index ก่อน)
let center = CLLocation(latitude: 13.7563, longitude: 100.5018)
let radius: CLLocationDistance = 5000 // 5km
NSPredicate(format: "distanceToLocation:fromLocation:(%K, %@) < %f",
            "location", center, radius)

// Compound predicates
let ageCondition = NSPredicate(format: "age > %d", 18)
let activeCondition = NSPredicate(format: "isActive == %@", true as CVarArg)
let compoundPredicate = NSCompoundPredicate(andPredicateWithSubpredicates: [ageCondition, activeCondition])

// IN operator
let allowedTags = ["swift", "ios", "apple"]
NSPredicate(format: "tag IN %@", allowedTags)
```

---

## 43.8 CKSubscription

CKSubscription ช่วยให้ App รับ Push Notification เมื่อมีการเปลี่ยนแปลงข้อมูลใน CloudKit

### ประเภทของ Subscription

1. **CKQuerySubscription** - Subscribe การเปลี่ยนแปลงที่ตรงกับ Query
2. **CKRecordZoneSubscription** - Subscribe การเปลี่ยนแปลงใน Zone ทั้งหมด
3. **CKDatabaseSubscription** - Subscribe การเปลี่ยนแปลงใน Database ทั้งหมด

### CKQuerySubscription

```swift
func createNoteSubscription() {
    let predicate = NSPredicate(value: true)
    
    let subscription = CKQuerySubscription(
        recordType: "Note",
        predicate: predicate,
        options: [.firesOnRecordCreation, .firesOnRecordUpdate, .firesOnRecordDeletion]
    )
    
    // ตั้งค่า Notification
    let notificationInfo = CKSubscription.NotificationInfo()
    notificationInfo.title = "บันทึกมีการเปลี่ยนแปลง"
    notificationInfo.alertBody = "มีการเปลี่ยนแปลงในบันทึกของคุณ"
    notificationInfo.soundName = "default"
    notificationInfo.shouldBadge = true
    notificationInfo.shouldSendContentAvailable = true // Silent Push
    
    subscription.notificationInfo = notificationInfo
    
    let db = CKContainer.default().privateCloudDatabase
    db.save(subscription) { savedSubscription, error in
        if let error = error {
            print("Error creating subscription: \(error)")
        } else {
            print("Subscription created: \(savedSubscription?.subscriptionID ?? "")")
        }
    }
}
```

### CKRecordZoneSubscription

```swift
func createZoneSubscription(zoneID: CKRecordZone.ID) {
    let subscription = CKRecordZoneSubscription(zoneID: zoneID)
    
    let notificationInfo = CKSubscription.NotificationInfo()
    notificationInfo.shouldSendContentAvailable = true // Background fetch
    
    subscription.notificationInfo = notificationInfo
    
    let db = CKContainer.default().privateCloudDatabase
    db.save(subscription) { _, error in
        if let error = error {
            print("Zone subscription error: \(error)")
        } else {
            print("Zone subscription created")
        }
    }
}
```

### การจัดการ Push Notification

```swift
// AppDelegate.swift
import UIKit
import CloudKit

@main
class AppDelegate: UIResponder, UIApplicationDelegate {
    
    func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
    ) -> Bool {
        // ขอ Permission สำหรับ Push Notification
        UNUserNotificationCenter.current().requestAuthorization(
            options: [.alert, .badge, .sound]
        ) { granted, error in
            if granted {
                DispatchQueue.main.async {
                    UIApplication.shared.registerForRemoteNotifications()
                }
            }
        }
        return true
    }
    
    // รับ Push Token
    func application(
        _ application: UIApplication,
        didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data
    ) {
        // CloudKit จัดการ Token โดยอัตโนมัติ ไม่ต้องทำเพิ่ม
    }
    
    // จัดการ CloudKit Notification
    func application(
        _ application: UIApplication,
        didReceiveRemoteNotification userInfo: [AnyHashable: Any],
        fetchCompletionHandler completionHandler: @escaping (UIBackgroundFetchResult) -> Void
    ) {
        let notification = CKNotification(fromRemoteNotificationDictionary: userInfo)
        
        if notification?.containerIdentifier == CKContainer.default().containerIdentifier {
            handleCloudKitNotification(notification: notification, completion: completionHandler)
        } else {
            completionHandler(.noData)
        }
    }
    
    private func handleCloudKitNotification(
        notification: CKNotification?,
        completion: @escaping (UIBackgroundFetchResult) -> Void
    ) {
        guard let notification = notification else {
            completion(.noData)
            return
        }
        
        switch notification.notificationType {
        case .query:
            if let queryNotification = notification as? CKQueryNotification {
                handleQueryNotification(queryNotification, completion: completion)
            }
        case .recordZone:
            // จัดการ Zone Notification
            fetchChanges(completion: completion)
        default:
            completion(.noData)
        }
    }
    
    private func handleQueryNotification(
        _ notification: CKQueryNotification,
        completion: @escaping (UIBackgroundFetchResult) -> Void
    ) {
        guard let recordID = notification.recordID else {
            completion(.noData)
            return
        }
        
        print("Record changed: \(recordID.recordName)")
        print("Reason: \(notification.queryNotificationReason.rawValue)")
        
        // โหลดข้อมูลใหม่
        NotificationCenter.default.post(name: .cloudKitDataChanged, object: nil)
        completion(.newData)
    }
    
    private func fetchChanges(completion: @escaping (UIBackgroundFetchResult) -> Void) {
        // Fetch changes using CKFetchDatabaseChangesOperation
        completion(.newData)
    }
}

extension Notification.Name {
    static let cloudKitDataChanged = Notification.Name("cloudKitDataChanged")
}
```

---

## 43.9 Push Notifications with CloudKit

การตั้งค่า Push Notification สำหรับ CloudKit ต้องมีขั้นตอนดังนี้:

### การตั้งค่า Entitlements

```xml
<!-- App.entitlements -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>com.apple.developer.icloud-container-identifiers</key>
    <array>
        <string>iCloud.com.example.myapp</string>
    </array>
    <key>com.apple.developer.icloud-services</key>
    <array>
        <string>CloudKit</string>
    </array>
    <key>aps-environment</key>
    <string>development</string>
</dict>
</plist>
```

### Silent Push Notifications

```swift
// ตั้งค่า Subscription ให้ใช้ Silent Push
func setupSilentPushSubscription() {
    let subscription = CKDatabaseSubscription(subscriptionID: "all-changes")
    
    let notificationInfo = CKSubscription.NotificationInfo()
    notificationInfo.shouldSendContentAvailable = true // Silent Push
    // ไม่ต้องกำหนด title/body สำหรับ Silent Push
    
    subscription.notificationInfo = notificationInfo
    
    CKContainer.default().privateCloudDatabase.save(subscription) { _, error in
        if let error = error {
            let ckError = error as? CKError
            if ckError?.code == .serverRejectedRequest {
                print("Subscription already exists")
            } else {
                print("Error: \(error)")
            }
        }
    }
}
```

---

## 43.10 CKOperation vs Convenience API

CloudKit มี 2 วิธีหลักในการทำงาน:

### Convenience API (ง่ายกว่า)

```swift
// บันทึก Record เดียว
db.save(record) { savedRecord, error in
    // handle result
}

// ดึง Record เดียว
db.fetch(withRecordID: recordID) { record, error in
    // handle result
}

// Query
db.perform(query, inZoneWith: nil) { records, error in
    // handle result
}
```

### CKOperation (ยืดหยุ่นกว่า)

```swift
// ดึงหลาย Records พร้อมกัน
let recordIDs = [recordID1, recordID2, recordID3]
let operation = CKFetchRecordsOperation(recordIDs: recordIDs)

operation.perRecordResultBlock = { recordID, result in
    switch result {
    case .success(let record):
        print("Got record: \(recordID.recordName)")
    case .failure(let error):
        print("Error for \(recordID.recordName): \(error)")
    }
}

operation.fetchRecordsResultBlock = { result in
    switch result {
    case .success:
        print("All records fetched")
    case .failure(let error):
        print("Operation failed: \(error)")
    }
}

// ตั้งค่า QoS และ Priority
operation.qualityOfService = .utility
operation.database = CKContainer.default().privateCloudDatabase

db.add(operation)
```

### เมื่อไหรควรใช้อะไร?

| Convenience API | CKOperation |
|----------------|-------------|
| ง่ายต่อการเขียน | ยืดหยุ่นกว่า |
| เหมาะสำหรับ Record เดียว | เหมาะสำหรับ Batch Operations |
| ไม่รองรับ Progress Tracking | รองรับ Progress Tracking |
| ไม่ Cancellable | Cancellable |
| ไม่ Chain ได้ง่าย | Chain ได้ด้วย Dependencies |

---

## 43.11 CKFetchRecordsOperation

ใช้สำหรับดึง Records หลายรายการพร้อมกัน

```swift
class BatchFetcher {
    private let db: CKDatabase
    
    init(database: CKDatabase = CKContainer.default().privateCloudDatabase) {
        self.db = database
    }
    
    func fetchRecords(
        withIDs recordIDs: [CKRecord.ID],
        completion: @escaping ([CKRecord.ID: CKRecord]?, Error?) -> Void
    ) {
        let operation = CKFetchRecordsOperation(recordIDs: recordIDs)
        
        // ระบุ Fields ที่ต้องการ (ลด Data Transfer)
        operation.desiredKeys = ["title", "content", "createdAt"]
        
        var fetchedRecords: [CKRecord.ID: CKRecord] = [:]
        var fetchErrors: [CKRecord.ID: Error] = [:]
        
        operation.perRecordResultBlock = { recordID, result in
            switch result {
            case .success(let record):
                fetchedRecords[recordID] = record
            case .failure(let error):
                fetchErrors[recordID] = error
                print("Error fetching \(recordID.recordName): \(error)")
            }
        }
        
        operation.fetchRecordsResultBlock = { result in
            DispatchQueue.main.async {
                switch result {
                case .success:
                    completion(fetchedRecords, nil)
                case .failure(let error):
                    completion(fetchedRecords.isEmpty ? nil : fetchedRecords, error)
                }
            }
        }
        
        db.add(operation)
    }
    
    // Fetch ด้วย Progress Tracking
    func fetchRecordsWithProgress(
        withIDs recordIDs: [CKRecord.ID],
        progressHandler: @escaping (Double) -> Void,
        completion: @escaping ([CKRecord]?, Error?) -> Void
    ) {
        let operation = CKFetchRecordsOperation(recordIDs: recordIDs)
        
        var fetchedRecords: [CKRecord] = []
        let totalCount = recordIDs.count
        var completedCount = 0
        
        operation.perRecordResultBlock = { _, result in
            completedCount += 1
            let progress = Double(completedCount) / Double(totalCount)
            
            DispatchQueue.main.async {
                progressHandler(progress)
            }
            
            if case .success(let record) = result {
                fetchedRecords.append(record)
            }
        }
        
        operation.fetchRecordsResultBlock = { result in
            DispatchQueue.main.async {
                switch result {
                case .success:
                    completion(fetchedRecords, nil)
                case .failure(let error):
                    completion(nil, error)
                }
            }
        }
        
        db.add(operation)
    }
}
```

---

## 43.12 CKModifyRecordsOperation

ใช้สำหรับ Save และ Delete Records หลายรายการพร้อมกัน

```swift
class BatchModifier {
    private let db: CKDatabase
    
    init(database: CKDatabase = CKContainer.default().privateCloudDatabase) {
        self.db = database
    }
    
    func saveRecords(
        _ records: [CKRecord],
        deleteRecordIDs: [CKRecord.ID] = [],
        completion: @escaping (Result<[CKRecord], Error>) -> Void
    ) {
        let operation = CKModifyRecordsOperation(
            recordsToSave: records,
            recordIDsToDelete: deleteRecordIDs
        )
        
        // Conflict Resolution Policy
        operation.savePolicy = .changedKeys // เฉพาะ Fields ที่เปลี่ยน
        // .allKeys - บันทึกทุก Field
        // .ifServerRecordUnchanged - เฉพาะถ้า Server ยังไม่เปลี่ยน (Safe Update)
        
        operation.isAtomic = true // ถ้า Record ใด Fail ทั้งหมด Fail
        
        var savedRecords: [CKRecord] = []
        
        operation.perRecordSaveBlock = { recordID, result in
            switch result {
            case .success(let record):
                savedRecords.append(record)
            case .failure(let error):
                print("Failed to save \(recordID.recordName): \(error)")
            }
        }
        
        operation.perRecordDeleteBlock = { recordID, result in
            switch result {
            case .success:
                print("Deleted: \(recordID.recordName)")
            case .failure(let error):
                print("Failed to delete \(recordID.recordName): \(error)")
            }
        }
        
        operation.modifyRecordsResultBlock = { result in
            DispatchQueue.main.async {
                switch result {
                case .success:
                    completion(.success(savedRecords))
                case .failure(let error):
                    completion(.failure(error))
                }
            }
        }
        
        db.add(operation)
    }
    
    // Batch Save ที่รองรับ Conflict ด้วย Retry
    func saveWithConflictResolution(
        records: [CKRecord],
        completion: @escaping (Result<[CKRecord], Error>) -> Void
    ) {
        let operation = CKModifyRecordsOperation(recordsToSave: records, recordIDsToDelete: nil)
        operation.savePolicy = .ifServerRecordUnchanged
        
        var savedRecords: [CKRecord] = []
        var conflictedRecords: [(client: CKRecord, server: CKRecord)] = []
        
        operation.perRecordSaveBlock = { _, result in
            switch result {
            case .success(let record):
                savedRecords.append(record)
            case .failure(let error as CKError):
                if error.code == .serverRecordChanged,
                   let serverRecord = error.serverRecord,
                   let clientRecord = error.clientRecord {
                    conflictedRecords.append((client: clientRecord, server: serverRecord))
                }
            default:
                break
            }
        }
        
        operation.modifyRecordsResultBlock = { result in
            if !conflictedRecords.isEmpty {
                // Merge conflicts และลองใหม่
                let mergedRecords = conflictedRecords.map { conflict in
                    self.mergeRecords(client: conflict.client, server: conflict.server)
                }
                self.saveWithConflictResolution(records: mergedRecords, completion: completion)
            } else {
                DispatchQueue.main.async {
                    switch result {
                    case .success:
                        completion(.success(savedRecords))
                    case .failure(let error):
                        completion(.failure(error))
                    }
                }
            }
        }
        
        db.add(operation)
    }
    
    private func mergeRecords(client: CKRecord, server: CKRecord) -> CKRecord {
        // Merge Strategy: Server wins โดยค่าเริ่มต้น
        // คุณสามารถใช้ Logic ที่ซับซ้อนกว่านี้ได้
        let merged = server.copy() as! CKRecord
        // อัปเดตเฉพาะ Fields ที่ Client เปลี่ยนแปลง
        if let clientTitle = client["title"] as? String {
            merged["title"] = clientTitle as CKRecordValue
        }
        return merged
    }
}
```

---

## 43.13 Assets (CKAsset)

CKAsset ใช้สำหรับจัดเก็บ Binary Data ขนาดใหญ่ เช่น รูปภาพ เสียง วิดีโอ

### การสร้างและบันทึก CKAsset

```swift
import CloudKit
import UIKit

class PhotoNoteManager {
    private let db = CKContainer.default().privateCloudDatabase
    
    func savePhotoNote(
        title: String,
        image: UIImage,
        completion: @escaping (Result<CKRecord, Error>) -> Void
    ) {
        // แปลง UIImage เป็น Data
        guard let imageData = image.jpegData(compressionQuality: 0.8) else {
            completion(.failure(NSError(domain: "App", code: -1, userInfo: [NSLocalizedDescriptionKey: "Cannot convert image"])))
            return
        }
        
        // เขียน Data ลงไฟล์ชั่วคราว (CKAsset ต้องการ URL ไฟล์)
        let tempDirectory = FileManager.default.temporaryDirectory
        let tempURL = tempDirectory.appendingPathComponent(UUID().uuidString + ".jpg")
        
        do {
            try imageData.write(to: tempURL)
        } catch {
            completion(.failure(error))
            return
        }
        
        // สร้าง CKAsset จาก URL
        let imageAsset = CKAsset(fileURL: tempURL)
        
        // สร้าง Record พร้อม Asset
        let record = CKRecord(recordType: "PhotoNote")
        record["title"] = title as CKRecordValue
        record["photo"] = imageAsset
        record["createdAt"] = Date() as CKRecordValue
        
        // บันทึก Record
        db.save(record) { savedRecord, error in
            // ลบไฟล์ชั่วคราว
            try? FileManager.default.removeItem(at: tempURL)
            
            DispatchQueue.main.async {
                if let error = error {
                    completion(.failure(error))
                } else if let savedRecord = savedRecord {
                    completion(.success(savedRecord))
                }
            }
        }
    }
    
    // ดาวน์โหลด Asset
    func downloadPhoto(
        from record: CKRecord,
        completion: @escaping (UIImage?) -> Void
    ) {
        guard let asset = record["photo"] as? CKAsset,
              let fileURL = asset.fileURL else {
            completion(nil)
            return
        }
        
        // Asset ถูก Download มาแล้วพร้อม Record
        // fileURL ชี้ไปยังไฟล์ที่ Download แล้ว
        DispatchQueue.global(qos: .background).async {
            guard let data = try? Data(contentsOf: fileURL),
                  let image = UIImage(data: data) else {
                DispatchQueue.main.async { completion(nil) }
                return
            }
            
            DispatchQueue.main.async { completion(image) }
        }
    }
    
    // บันทึก Audio Asset
    func saveAudioNote(
        title: String,
        audioURL: URL,
        completion: @escaping (Result<CKRecord, Error>) -> Void
    ) {
        let audioAsset = CKAsset(fileURL: audioURL)
        
        let record = CKRecord(recordType: "AudioNote")
        record["title"] = title as CKRecordValue
        record["audio"] = audioAsset
        record["duration"] = 0.0 as CKRecordValue // ใส่ duration จริง
        record["createdAt"] = Date() as CKRecordValue
        
        db.save(record) { savedRecord, error in
            DispatchQueue.main.async {
                if let error = error {
                    completion(.failure(error))
                } else if let savedRecord = savedRecord {
                    completion(.success(savedRecord))
                }
            }
        }
    }
}
```

### ข้อจำกัดของ CKAsset

```swift
// CKAsset Limitations:
// - ขนาดสูงสุด: 750MB ต่อ Asset
// - ต้องผ่าน File URL ไม่ใช่ Data โดยตรง
// - Download พร้อมกัน Record (ไม่ Lazy Load)
// - ถ้า Record ถูกลบ Asset จะถูกลบด้วย

// แนะนำให้ใช้ desiredKeys เพื่อหลีกเลี่ยงการ Download Asset ที่ไม่จำเป็น
let operation = CKFetchRecordsOperation(recordIDs: [recordID])
operation.desiredKeys = ["title", "createdAt"] // ไม่รวม "photo"
```

---

## 43.14 References (CKReference)

CKReference ใช้สร้างความสัมพันธ์ระหว่าง Records คล้ายกับ Foreign Key

### การสร้าง Reference

```swift
// Notebook มี Notes หลายรายการ
struct CloudKitRelationships {
    
    // สร้าง Notebook
    func createNotebook(
        name: String,
        completion: @escaping (CKRecord?) -> Void
    ) {
        let notebookRecord = CKRecord(recordType: "Notebook")
        notebookRecord["name"] = name as CKRecordValue
        notebookRecord["createdAt"] = Date() as CKRecordValue
        
        CKContainer.default().privateCloudDatabase.save(notebookRecord) { record, _ in
            DispatchQueue.main.async { completion(record) }
        }
    }
    
    // สร้าง Note ที่อ้างอิง Notebook
    func createNote(
        title: String,
        content: String,
        notebook: CKRecord,
        completion: @escaping (CKRecord?) -> Void
    ) {
        let noteRecord = CKRecord(recordType: "Note")
        noteRecord["title"] = title as CKRecordValue
        noteRecord["content"] = content as CKRecordValue
        
        // สร้าง Reference ไปยัง Notebook
        // .deleteSelf = ถ้า Notebook ถูกลบ Note ก็จะถูกลบตาม (Cascade Delete)
        // .none = ถ้า Notebook ถูกลบ Note ไม่ถูกลบ (Reference เป็น nil)
        let notebookReference = CKRecord.Reference(
            record: notebook,
            action: .deleteSelf
        )
        noteRecord["notebook"] = notebookReference
        
        CKContainer.default().privateCloudDatabase.save(noteRecord) { record, _ in
            DispatchQueue.main.async { completion(record) }
        }
    }
    
    // Query Notes ใน Notebook
    func fetchNotes(
        inNotebook notebook: CKRecord,
        completion: @escaping ([CKRecord]) -> Void
    ) {
        let notebookReference = CKRecord.Reference(
            record: notebook,
            action: .none
        )
        
        let predicate = NSPredicate(format: "notebook == %@", notebookReference)
        let query = CKQuery(recordType: "Note", predicate: predicate)
        
        CKContainer.default().privateCloudDatabase.perform(query, inZoneWith: nil) { records, error in
            DispatchQueue.main.async {
                completion(records ?? [])
            }
        }
    }
    
    // สร้าง Reference ด้วย Record ID
    func createReferenceByID(recordID: CKRecord.ID) -> CKRecord.Reference {
        return CKRecord.Reference(recordID: recordID, action: .deleteSelf)
    }
    
    // Many-to-Many: Note มี Tags หลายรายการ
    func createTagRelationship() {
        let noteRecord = CKRecord(recordType: "Note")
        
        // ใช้ Array ของ References สำหรับ Many-to-Many
        let tagID1 = CKRecord.ID(recordName: "tag-swift")
        let tagID2 = CKRecord.ID(recordName: "tag-ios")
        
        let tagRef1 = CKRecord.Reference(recordID: tagID1, action: .none)
        let tagRef2 = CKRecord.Reference(recordID: tagID2, action: .none)
        
        noteRecord["tags"] = [tagRef1, tagRef2] as CKRecordValue
    }
}
```

---

## 43.15 Zone-Based Storage

Record Zones ช่วยจัดกลุ่ม Records และรองรับ Atomic Transactions และ Fetch Changes

### Default Zone vs Custom Zone

```swift
// Default Zone - ไม่รองรับ Atomic Transactions หรือ Fetch Changes
let defaultZoneID = CKRecordZone.default().zoneID

// Custom Zone - รองรับ Feature ขั้นสูง
let customZoneID = CKRecordZone.ID(
    zoneName: "NotesZone",
    ownerName: CKCurrentUserDefaultName
)
```

### การสร้าง Custom Zone

```swift
class ZoneManager {
    private let db = CKContainer.default().privateCloudDatabase
    let notesZone = CKRecordZone(zoneName: "NotesZone")
    
    func createZone(completion: @escaping (Error?) -> Void) {
        let operation = CKModifyRecordZonesOperation(
            recordZonesToSave: [notesZone],
            recordZoneIDsToDelete: nil
        )
        
        operation.modifyRecordZonesResultBlock = { result in
            DispatchQueue.main.async {
                switch result {
                case .success:
                    completion(nil)
                case .failure(let error):
                    completion(error)
                }
            }
        }
        
        db.add(operation)
    }
    
    // สร้าง Record ใน Custom Zone
    func saveNoteInZone(title: String) {
        let recordID = CKRecord.ID(
            recordName: UUID().uuidString,
            zoneID: notesZone.zoneID
        )
        let noteRecord = CKRecord(recordType: "Note", recordID: recordID)
        noteRecord["title"] = title as CKRecordValue
        
        db.save(noteRecord) { _, error in
            if let error = error {
                print("Error: \(error)")
            }
        }
    }
    
    // Fetch Changes ใน Zone (สำหรับ Sync)
    func fetchChanges(
        since serverChangeToken: CKServerChangeToken?,
        completion: @escaping ([CKRecord], [CKRecord.ID], CKServerChangeToken?, Error?) -> Void
    ) {
        let configuration = CKFetchRecordZoneChangesOperation.ZoneConfiguration()
        configuration.previousServerChangeToken = serverChangeToken
        
        let operation = CKFetchRecordZoneChangesOperation(
            recordZoneIDs: [notesZone.zoneID],
            configurationsByRecordZoneID: [notesZone.zoneID: configuration]
        )
        
        var changedRecords: [CKRecord] = []
        var deletedRecordIDs: [CKRecord.ID] = []
        var newChangeToken: CKServerChangeToken?
        
        operation.recordWasChangedBlock = { _, result in
            if case .success(let record) = result {
                changedRecords.append(record)
            }
        }
        
        operation.recordWithIDWasDeletedBlock = { recordID, _ in
            deletedRecordIDs.append(recordID)
        }
        
        operation.recordZoneChangeTokensUpdatedBlock = { _, token, _ in
            newChangeToken = token
        }
        
        operation.recordZoneFetchResultBlock = { _, result in
            switch result {
            case .success(let (serverToken, _, _)):
                newChangeToken = serverToken
            case .failure(let error):
                print("Zone fetch error: \(error)")
            }
        }
        
        operation.fetchRecordZoneChangesResultBlock = { result in
            DispatchQueue.main.async {
                switch result {
                case .success:
                    completion(changedRecords, deletedRecordIDs, newChangeToken, nil)
                case .failure(let error):
                    completion([], [], nil, error)
                }
            }
        }
        
        db.add(operation)
    }
}
```

---

## 43.16 Sharing Records

CloudKit รองรับการ Share Records ระหว่างผู้ใช้

### การสร้าง CKShare

```swift
import CloudKit
import UIKit

class SharingManager {
    private let db = CKContainer.default().privateCloudDatabase
    
    // สร้าง Share สำหรับ Record
    func shareRecord(
        _ record: CKRecord,
        from viewController: UIViewController,
        completion: @escaping (CKShare?, Error?) -> Void
    ) {
        let share = CKShare(rootRecord: record)
        share[CKShare.SystemFieldKey.title] = "บันทึกที่ฉัน Share" as CKRecordValue
        share.publicPermission = .readOnly // ทุกคนที่มี Link อ่านได้
        
        let operation = CKModifyRecordsOperation(
            recordsToSave: [record, share],
            recordIDsToDelete: nil
        )
        
        operation.modifyRecordsResultBlock = { result in
            DispatchQueue.main.async {
                switch result {
                case .success:
                    completion(share, nil)
                case .failure(let error):
                    completion(nil, error)
                }
            }
        }
        
        db.add(operation)
    }
    
    // แสดง CloudKit Sharing UI
    func presentSharingUI(
        for record: CKRecord,
        from viewController: UIViewController
    ) {
        let share = CKShare(rootRecord: record)
        
        let sharingController = UICloudSharingController { controller, preparationCompletionHandler in
            let operation = CKModifyRecordsOperation(
                recordsToSave: [record, share],
                recordIDsToDelete: nil
            )
            
            operation.modifyRecordsResultBlock = { result in
                switch result {
                case .success:
                    preparationCompletionHandler(share, CKContainer.default(), nil)
                case .failure(let error):
                    preparationCompletionHandler(nil, nil, error)
                }
            }
            
            CKContainer.default().privateCloudDatabase.add(operation)
        }
        
        sharingController.availablePermissions = [.allowReadOnly, .allowPrivate]
        sharingController.delegate = viewController as? UICloudSharingControllerDelegate
        
        viewController.present(sharingController, animated: true)
    }
    
    // รับ Share Invitation
    func acceptShare(from metadata: CKShare.Metadata, completion: @escaping (Error?) -> Void) {
        let operation = CKAcceptSharesOperation(shareMetadatas: [metadata])
        
        operation.acceptSharesResultBlock = { result in
            DispatchQueue.main.async {
                switch result {
                case .success:
                    completion(nil)
                case .failure(let error):
                    completion(error)
                }
            }
        }
        
        CKContainer.default().add(operation)
    }
}

// AppDelegate - จัดการ Share URL
extension AppDelegate {
    func application(
        _ application: UIApplication,
        userDidAcceptCloudKitShareWith cloudKitShareMetadata: CKShare.Metadata
    ) {
        let sharingManager = SharingManager()
        sharingManager.acceptShare(from: cloudKitShareMetadata) { error in
            if let error = error {
                print("Error accepting share: \(error)")
            } else {
                // Navigate ไปยัง Shared Record
                NotificationCenter.default.post(name: .sharedRecordAccepted, object: cloudKitShareMetadata)
            }
        }
    }
}

extension Notification.Name {
    static let sharedRecordAccepted = Notification.Name("sharedRecordAccepted")
}
```

---

## 43.17 CloudKit Dashboard

CloudKit Dashboard คือ Web Interface สำหรับจัดการ CloudKit Container ของคุณ

### วิธีเข้าถึง

1. ไปที่ [developer.apple.com/icloud/cloudkit](https://developer.apple.com/icloud/cloudkit)
2. Login ด้วย Apple Developer Account
3. เลือก Container

### Feature ต่างๆ ใน Dashboard

```
CloudKit Dashboard Features:
├── Schema Management
│   ├── สร้าง/แก้ไข Record Types
│   ├── เพิ่ม/ลบ Fields
│   └── สร้าง Indexes
├── Data Viewer
│   ├── Browse Records
│   ├── Query Records
│   └── แก้ไข/ลบ Records
├── Subscription Management
│   └── ดู Active Subscriptions
├── Telemetry
│   ├── Request Counts
│   ├── Data Transfer
│   └── Error Rates
└── Deploy Schema
    └── Deploy จาก Development ไป Production
```

### สร้าง Index ใน Dashboard

การสร้าง Index จำเป็นสำหรับ:
- การ Query ด้วย Field นั้น
- การ Sort ด้วย Field นั้น

ประเภท Index:
- **Queryable** - ใช้ใน NSPredicate
- **Sortable** - ใช้ใน Sort Descriptors
- **Searchable** - ใช้สำหรับ Full-text Search

---

## 43.18 Core Data + CloudKit (NSPersistentCloudKitContainer)

iOS 13+ รองรับการ Sync Core Data กับ CloudKit โดยอัตโนมัติ

### การตั้งค่า

```swift
// Persistence.swift
import CoreData

struct PersistenceController {
    static let shared = PersistenceController()
    
    let container: NSPersistentCloudKitContainer
    
    init(inMemory: Bool = false) {
        container = NSPersistentCloudKitContainer(name: "MyApp")
        
        if inMemory {
            container.persistentStoreDescriptions.first?.url = URL(fileURLWithPath: "/dev/null")
        }
        
        // ตั้งค่า CloudKit Sync
        guard let description = container.persistentStoreDescriptions.first else {
            fatalError("No persistent store descriptions found.")
        }
        
        // เปิดใช้ Remote Change Notifications
        description.setOption(
            true as NSNumber,
            forKey: NSPersistentStoreRemoteChangeNotificationPostOptionKey
        )
        
        // กำหนด CloudKit Container (ถ้าต้องการ Custom)
        description.cloudKitContainerOptions = NSPersistentCloudKitContainerOptions(
            containerIdentifier: "iCloud.com.example.myapp"
        )
        
        container.loadPersistentStores { storeDescription, error in
            if let error = error as NSError? {
                fatalError("Unresolved error \(error), \(error.userInfo)")
            }
        }
        
        // Automatically merge changes from CloudKit
        container.viewContext.automaticallyMergesChangesFromParent = true
        container.viewContext.mergePolicy = NSMergeByPropertyObjectTrumpMergePolicy
        
        // Listen for Remote Changes
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(processRemoteChanges(_:)),
            name: .NSPersistentStoreRemoteChange,
            object: container.persistentStoreCoordinator
        )
    }
    
    @objc private func processRemoteChanges(_ notification: Notification) {
        Task {
            await container.viewContext.perform {
                // Context จะ Merge Changes อัตโนมัติ
            }
        }
    }
}
```

### การกำหนด Core Data Model สำหรับ CloudKit

```
// ข้อกำหนดสำหรับ CloudKit Sync:
// 1. Entity ทุกตัวต้องมี createdAt, modifiedAt attributes (optional Date)
// 2. Attribute ต้องเป็น Optional (ไม่บังคับ)
// 3. Relationship ต้องมี Inverse
// 4. Ordered Relationship ไม่รองรับ
// 5. Unique Constraints ไม่รองรับ
```

### การใช้งาน Core Data + CloudKit

```swift
// SwiftUI View ที่ใช้ Core Data + CloudKit
import SwiftUI
import CoreData

struct NoteListView: View {
    @Environment(\.managedObjectContext) private var viewContext
    
    @FetchRequest(
        sortDescriptors: [NSSortDescriptor(keyPath: \Note.createdAt, ascending: false)],
        animation: .default
    )
    private var notes: FetchedResults<Note>
    
    var body: some View {
        List {
            ForEach(notes) { note in
                NavigationLink {
                    NoteDetailView(note: note)
                } label: {
                    VStack(alignment: .leading) {
                        Text(note.title ?? "ไม่มีชื่อ")
                            .font(.headline)
                        Text(note.content ?? "")
                            .font(.subheadline)
                            .foregroundColor(.secondary)
                            .lineLimit(2)
                    }
                }
            }
            .onDelete(perform: deleteNotes)
        }
        .toolbar {
            ToolbarItem(placement: .navigationBarTrailing) {
                Button(action: addNote) {
                    Label("เพิ่มบันทึก", systemImage: "plus")
                }
            }
        }
    }
    
    private func addNote() {
        withAnimation {
            let newNote = Note(context: viewContext)
            newNote.title = "บันทึกใหม่"
            newNote.content = ""
            newNote.createdAt = Date()
            newNote.modifiedAt = Date()
            
            do {
                try viewContext.save()
                // CloudKit จะ Sync อัตโนมัติ
            } catch {
                print("Error saving: \(error)")
            }
        }
    }
    
    private func deleteNotes(offsets: IndexSet) {
        withAnimation {
            offsets.map { notes[$0] }.forEach(viewContext.delete)
            
            do {
                try viewContext.save()
            } catch {
                print("Error deleting: \(error)")
            }
        }
    }
}
```

### ข้อควรระวัง Core Data + CloudKit

```swift
// 1. Schema Migration ต้องระวัง
// เมื่อเพิ่ม Attribute ใหม่ต้องเป็น Optional

// 2. ตรวจสอบ Sync Status
extension PersistenceController {
    func checkSyncStatus() {
        // iOS 16+
        if #available(iOS 16, *) {
            do {
                let events = try container.cloudKitContainer?.fetchCloudKitEvents()
                for event in events ?? [] {
                    print("Event: \(event.type) - \(event.succeeded ? "Success" : "Failed")")
                }
            } catch {
                print("Error checking sync: \(error)")
            }
        }
    }
}

// 3. Handle Network Errors Gracefully
// Core Data + CloudKit จัดการ Error อัตโนมัติ แต่ควร Monitor
```

---

## 43.19 CloudKit JS (Brief Mention)

CloudKit JS ช่วยให้ Web App เข้าถึง CloudKit ได้

```javascript
// ตัวอย่าง CloudKit JS (JavaScript)
CloudKit.configure({
    containers: [{
        containerIdentifier: 'iCloud.com.example.myapp',
        apiTokenAuth: {
            apiToken: 'your-api-token',
            persist: true
        },
        environment: 'development'
    }]
});

// Query records
var container = CloudKit.getDefaultContainer();
var publicDB = container.publicCloudDatabase;

publicDB.performQuery({
    recordType: 'Note',
    filterBy: [{
        fieldName: 'isPinned',
        comparator: 'EQUALS',
        fieldValue: { value: 1 }
    }]
}).then(function(response) {
    if (response.hasErrors) {
        console.error(response.errors[0]);
    } else {
        var records = response.records;
        console.log('Found notes:', records.length);
    }
});
```

---

## 43.20 Rate Limits and Best Practices

### Rate Limits

```
CloudKit Rate Limits (โดยประมาณ):
├── Public Database
│   ├── Read: 40 requests/second
│   └── Write: 40 requests/second
├── Private Database
│   ├── Read: 100 requests/second
│   └── Write: 100 requests/second
└── Shared Database
    └── คล้ายกับ Private Database
```

### Best Practices

```swift
// 1. ใช้ CKOperation แทน Convenience API สำหรับ Batch Operations
class BestPractices {
    
    // ✅ ดี - Batch Operation
    func goodBatchSave(records: [CKRecord]) {
        let operation = CKModifyRecordsOperation(
            recordsToSave: records,
            recordIDsToDelete: nil
        )
        operation.savePolicy = .changedKeys
        CKContainer.default().privateCloudDatabase.add(operation)
    }
    
    // ❌ ไม่ดี - หลาย Save calls
    func badMultipleSaves(records: [CKRecord]) {
        for record in records {
            CKContainer.default().privateCloudDatabase.save(record) { _, _ in }
        }
    }
    
    // 2. ใช้ desiredKeys เพื่อลด Data Transfer
    func fetchWithDesiredKeys(recordIDs: [CKRecord.ID]) {
        let operation = CKFetchRecordsOperation(recordIDs: recordIDs)
        operation.desiredKeys = ["title", "createdAt"] // ไม่ดึง photo, audio
        CKContainer.default().privateCloudDatabase.add(operation)
    }
    
    // 3. Cache Server Change Tokens
    func saveChangeToken(_ token: CKServerChangeToken) {
        if let data = try? NSKeyedArchiver.archivedData(withRootObject: token, requiringSecureCoding: true) {
            UserDefaults.standard.set(data, forKey: "serverChangeToken")
        }
    }
    
    func loadChangeToken() -> CKServerChangeToken? {
        guard let data = UserDefaults.standard.data(forKey: "serverChangeToken") else {
            return nil
        }
        return try? NSKeyedUnarchiver.unarchivedObject(ofClass: CKServerChangeToken.self, from: data)
    }
    
    // 4. Handle Errors อย่างเหมาะสม
    func handleCKError(_ error: Error) {
        guard let ckError = error as? CKError else { return }
        
        switch ckError.code {
        case .networkUnavailable, .networkFailure:
            // Retry ภายหลัง
            print("Network error - will retry")
        case .serviceUnavailable, .requestRateLimited:
            // Exponential Backoff
            let retryAfter = ckError.retryAfterSeconds ?? 5.0
            print("Rate limited - retry after \(retryAfter) seconds")
        case .quotaExceeded:
            // แจ้งผู้ใช้ให้ upgrade iCloud Storage
            print("iCloud storage full")
        case .serverRecordChanged:
            // Conflict - Merge และ Retry
            if let serverRecord = ckError.serverRecord,
               let clientRecord = ckError.clientRecord {
                print("Conflict: merge records")
            }
        case .unknownItem:
            print("Record not found - may have been deleted")
        case .permissionFailure:
            print("No permission to access record")
        default:
            print("CloudKit error: \(ckError.localizedDescription)")
        }
    }
    
    // 5. ใช้ Custom Zone สำหรับ Sync
    // - รองรับ Atomic Transactions
    // - รองรับ Fetch Changes (ประหยัด Bandwidth)
    // - รองรับ Sharing
    
    // 6. Minimize Subscriptions
    // - ใช้ Database Subscription แทน Query Subscription หลายๆ อัน
    // - ใช้ Silent Push แทน Visible Notification ถ้าทำได้
}
```

---

## 43.21 Practical Exercises

### Exercise 1: สร้าง Simple CloudKit CRUD

**โจทย์:** สร้าง App ที่บันทึก Task ขึ้น CloudKit และ Sync ระหว่าง Device

```swift
// Task Model
struct Task: Identifiable {
    let id: CKRecord.ID
    var title: String
    var isCompleted: Bool
    var dueDate: Date?
    var priority: Int
    
    static let recordType = "Task"
    
    enum Field {
        static let title = "title"
        static let isCompleted = "isCompleted"
        static let dueDate = "dueDate"
        static let priority = "priority"
    }
    
    init?(record: CKRecord) {
        guard record.recordType == Task.recordType,
              let title = record[Field.title] as? String else {
            return nil
        }
        self.id = record.recordID
        self.title = title
        self.isCompleted = record[Field.isCompleted] as? Bool ?? false
        self.dueDate = record[Field.dueDate] as? Date
        self.priority = record[Field.priority] as? Int ?? 0
    }
    
    func toRecord() -> CKRecord {
        let record = CKRecord(recordType: Task.recordType, recordID: id)
        record[Field.title] = title as CKRecordValue
        record[Field.isCompleted] = isCompleted as CKRecordValue
        if let dueDate = dueDate {
            record[Field.dueDate] = dueDate as CKRecordValue
        }
        record[Field.priority] = priority as CKRecordValue
        return record
    }
    
    // สร้าง Task ใหม่
    static func new(title: String, priority: Int = 0) -> Task {
        let recordID = CKRecord.ID(recordName: UUID().uuidString)
        let record = CKRecord(recordType: Task.recordType, recordID: recordID)
        record[Field.title] = title as CKRecordValue
        record[Field.isCompleted] = false as CKRecordValue
        record[Field.priority] = priority as CKRecordValue
        return Task(record: record)!
    }
}

// Task Repository
@MainActor
class TaskRepository: ObservableObject {
    @Published var tasks: [Task] = []
    @Published var isLoading = false
    @Published var error: Error?
    
    private let db = CKContainer.default().privateCloudDatabase
    
    func fetchTasks() async {
        isLoading = true
        defer { isLoading = false }
        
        do {
            let predicate = NSPredicate(value: true)
            let query = CKQuery(recordType: Task.recordType, predicate: predicate)
            query.sortDescriptors = [
                NSSortDescriptor(key: "isCompleted", ascending: true),
                NSSortDescriptor(key: "priority", ascending: false)
            ]
            
            let (results, _) = try await db.records(matching: query)
            
            tasks = results.compactMap { _, result in
                guard case .success(let record) = result else { return nil }
                return Task(record: record)
            }
        } catch {
            self.error = error
        }
    }
    
    func addTask(title: String) async {
        let newTask = Task.new(title: title)
        
        do {
            let savedRecord = try await db.save(newTask.toRecord())
            if let task = Task(record: savedRecord) {
                tasks.append(task)
            }
        } catch {
            self.error = error
        }
    }
    
    func toggleCompletion(_ task: Task) async {
        guard let index = tasks.firstIndex(where: { $0.id == task.id }) else { return }
        
        var updatedTask = task
        updatedTask.isCompleted.toggle()
        tasks[index] = updatedTask
        
        do {
            _ = try await db.save(updatedTask.toRecord())
        } catch {
            // Revert on failure
            tasks[index] = task
            self.error = error
        }
    }
    
    func deleteTask(_ task: Task) async {
        tasks.removeAll { $0.id == task.id }
        
        do {
            try await db.deleteRecord(withID: task.id)
        } catch {
            self.error = error
        }
    }
}
```

---

## 43.22 Building a Synced Note-Taking App

มาสร้าง Note-taking App ที่ Sync ด้วย CloudKit กัน:

### Architecture Overview

```
NoteApp
├── Models/
│   ├── Note.swift
│   ├── Notebook.swift
│   └── Tag.swift
├── Services/
│   ├── CloudKitService.swift
│   └── SyncManager.swift
├── ViewModels/
│   ├── NotebookListViewModel.swift
│   └── NoteListViewModel.swift
└── Views/
    ├── NotebookListView.swift
    ├── NoteListView.swift
    └── NoteEditorView.swift
```

### CloudKitService.swift

```swift
import CloudKit
import Foundation

actor CloudKitService {
    private let container = CKContainer.default()
    private var privateDB: CKDatabase { container.privateCloudDatabase }
    
    // MARK: - Notebooks
    
    func fetchNotebooks() async throws -> [CKRecord] {
        let predicate = NSPredicate(value: true)
        let query = CKQuery(recordType: "Notebook", predicate: predicate)
        query.sortDescriptors = [NSSortDescriptor(key: "name", ascending: true)]
        
        let (results, _) = try await privateDB.records(matching: query)
        return results.compactMap { _, result in
            guard case .success(let record) = result else { return nil }
            return record
        }
    }
    
    func saveNotebook(_ notebook: CKRecord) async throws -> CKRecord {
        return try await privateDB.save(notebook)
    }
    
    // MARK: - Notes
    
    func fetchNotes(in notebook: CKRecord) async throws -> [CKRecord] {
        let notebookRef = CKRecord.Reference(record: notebook, action: .none)
        let predicate = NSPredicate(format: "notebook == %@", notebookRef)
        let query = CKQuery(recordType: "Note", predicate: predicate)
        query.sortDescriptors = [NSSortDescriptor(key: "modifiedAt", ascending: false)]
        
        let (results, _) = try await privateDB.records(matching: query)
        return results.compactMap { _, result in
            guard case .success(let record) = result else { return nil }
            return record
        }
    }
    
    func saveNote(_ note: CKRecord) async throws -> CKRecord {
        return try await privateDB.save(note)
    }
    
    func deleteNote(recordID: CKRecord.ID) async throws {
        try await privateDB.deleteRecord(withID: recordID)
    }
    
    // MARK: - Sync
    
    func setupSubscription() async throws {
        let subscription = CKDatabaseSubscription(subscriptionID: "all-private-changes")
        let notificationInfo = CKSubscription.NotificationInfo()
        notificationInfo.shouldSendContentAvailable = true
        subscription.notificationInfo = notificationInfo
        
        do {
            try await privateDB.save(subscription)
        } catch let error as CKError where error.code == .duplicateSubscription {
            // Subscription already exists - ไม่ใช่ Error
        }
    }
}
```

### SyncManager.swift

```swift
import CloudKit
import Foundation
import Combine

@MainActor
class SyncManager: ObservableObject {
    @Published var isSyncing = false
    @Published var lastSyncDate: Date?
    @Published var syncError: Error?
    
    private let cloudKitService = CloudKitService()
    private var serverChangeToken: CKServerChangeToken?
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        setupNotificationHandling()
        serverChangeToken = loadChangeToken()
    }
    
    func setupNotificationHandling() {
        NotificationCenter.default.publisher(for: .cloudKitDataChanged)
            .sink { [weak self] _ in
                Task {
                    await self?.syncChanges()
                }
            }
            .store(in: &cancellables)
    }
    
    func syncChanges() async {
        guard !isSyncing else { return }
        
        isSyncing = true
        defer { isSyncing = false }
        
        do {
            try await cloudKitService.setupSubscription()
            lastSyncDate = Date()
            saveChangeToken(serverChangeToken)
        } catch {
            syncError = error
        }
    }
    
    private func saveChangeToken(_ token: CKServerChangeToken?) {
        guard let token = token,
              let data = try? NSKeyedArchiver.archivedData(
                withRootObject: token,
                requiringSecureCoding: true
              ) else { return }
        UserDefaults.standard.set(data, forKey: "ckServerChangeToken")
    }
    
    private func loadChangeToken() -> CKServerChangeToken? {
        guard let data = UserDefaults.standard.data(forKey: "ckServerChangeToken") else {
            return nil
        }
        return try? NSKeyedUnarchiver.unarchivedObject(
            ofClass: CKServerChangeToken.self,
            from: data
        )
    }
}
```

### NotebookListViewModel.swift

```swift
import CloudKit
import Foundation

@MainActor
class NotebookListViewModel: ObservableObject {
    @Published var notebooks: [NotebookModel] = []
    @Published var isLoading = false
    @Published var error: Error?
    
    private let service = CloudKitService()
    
    struct NotebookModel: Identifiable {
        let id: CKRecord.ID
        var name: String
        var noteCount: Int
        let record: CKRecord
        
        init?(record: CKRecord) {
            guard let name = record["name"] as? String else { return nil }
            self.id = record.recordID
            self.name = name
            self.noteCount = record["noteCount"] as? Int ?? 0
            self.record = record
        }
    }
    
    func loadNotebooks() async {
        isLoading = true
        defer { isLoading = false }
        
        do {
            let records = try await service.fetchNotebooks()
            notebooks = records.compactMap { NotebookModel(record: $0) }
        } catch {
            self.error = error
        }
    }
    
    func createNotebook(name: String) async {
        let record = CKRecord(recordType: "Notebook")
        record["name"] = name as CKRecordValue
        record["createdAt"] = Date() as CKRecordValue
        record["noteCount"] = 0 as CKRecordValue
        
        do {
            let savedRecord = try await service.saveNotebook(record)
            if let notebook = NotebookModel(record: savedRecord) {
                notebooks.append(notebook)
                notebooks.sort { $0.name < $1.name }
            }
        } catch {
            self.error = error
        }
    }
}
```

### Views

```swift
import SwiftUI
import CloudKit

// NotebookListView
struct NotebookListView: View {
    @StateObject private var viewModel = NotebookListViewModel()
    @State private var showingCreateSheet = false
    @State private var newNotebookName = ""
    
    var body: some View {
        NavigationView {
            Group {
                if viewModel.isLoading && viewModel.notebooks.isEmpty {
                    ProgressView("กำลังโหลด...")
                } else {
                    List(viewModel.notebooks) { notebook in
                        NavigationLink {
                            NoteListView(notebook: notebook.record)
                        } label: {
                            HStack {
                                Image(systemName: "book.closed.fill")
                                    .foregroundColor(.blue)
                                VStack(alignment: .leading) {
                                    Text(notebook.name)
                                        .font(.headline)
                                    Text("\(notebook.noteCount) บันทึก")
                                        .font(.caption)
                                        .foregroundColor(.secondary)
                                }
                            }
                        }
                    }
                }
            }
            .navigationTitle("Notebooks")
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button {
                        showingCreateSheet = true
                    } label: {
                        Image(systemName: "plus")
                    }
                }
            }
            .sheet(isPresented: $showingCreateSheet) {
                createNotebookSheet
            }
            .task {
                await viewModel.loadNotebooks()
            }
            .alert("เกิดข้อผิดพลาด", isPresented: .constant(viewModel.error != nil)) {
                Button("ตกลง") { viewModel.error = nil }
            } message: {
                Text(viewModel.error?.localizedDescription ?? "")
            }
        }
    }
    
    var createNotebookSheet: some View {
        NavigationView {
            Form {
                Section("ชื่อ Notebook") {
                    TextField("ชื่อ", text: $newNotebookName)
                }
            }
            .navigationTitle("สร้าง Notebook ใหม่")
            .navigationBarItems(
                leading: Button("ยกเลิก") {
                    showingCreateSheet = false
                },
                trailing: Button("สร้าง") {
                    Task {
                        await viewModel.createNotebook(name: newNotebookName)
                        showingCreateSheet = false
                        newNotebookName = ""
                    }
                }
                .disabled(newNotebookName.isEmpty)
            )
        }
    }
}

// NoteEditorView
struct NoteEditorView: View {
    @State var title: String
    @State var content: String
    let onSave: (String, String) async -> Void
    @Environment(\.dismiss) private var dismiss
    
    var body: some View {
        NavigationView {
            VStack(spacing: 0) {
                TextField("ชื่อบันทึก", text: $title)
                    .font(.title2.bold())
                    .padding()
                
                Divider()
                
                TextEditor(text: $content)
                    .padding()
            }
            .navigationTitle("แก้ไขบันทึก")
            .navigationBarItems(
                leading: Button("ยกเลิก") { dismiss() },
                trailing: Button("บันทึก") {
                    Task {
                        await onSave(title, content)
                        dismiss()
                    }
                }
            )
        }
    }
}
```

---

## 43.23 Error Handling ใน CloudKit

```swift
class CloudKitErrorHandler {
    
    static func handle(_ error: Error, retryBlock: (() -> Void)? = nil) {
        guard let ckError = error as? CKError else {
            print("Non-CloudKit error: \(error)")
            return
        }
        
        switch ckError.code {
        // Network Errors - Retry
        case .networkUnavailable:
            print("Network ไม่พร้อมใช้งาน")
            scheduleRetry(retryBlock, after: 5.0)
            
        case .networkFailure:
            print("Network Failure")
            scheduleRetry(retryBlock, after: 3.0)
            
        case .serviceUnavailable:
            let delay = ckError.retryAfterSeconds ?? 60.0
            print("Service unavailable - retry after \(delay)s")
            scheduleRetry(retryBlock, after: delay)
            
        case .requestRateLimited:
            let delay = ckError.retryAfterSeconds ?? 30.0
            print("Rate limited - retry after \(delay)s")
            scheduleRetry(retryBlock, after: delay)
            
        // Auth Errors
        case .notAuthenticated:
            print("User ยังไม่ได้ Login iCloud")
            // แสดง Alert ให้ User Login
            
        case .permissionFailure:
            print("ไม่มีสิทธิ์เข้าถึง Record นี้")
            
        // Data Errors
        case .serverRecordChanged:
            print("Conflict - Record ถูกแก้ไขจาก Device อื่น")
            if let serverRecord = ckError.serverRecord {
                // Merge หรือแสดง Conflict UI
                print("Server version: \(serverRecord)")
            }
            
        case .unknownItem:
            print("Record ไม่พบ - อาจถูกลบแล้ว")
            
        case .invalidArguments:
            print("Arguments ไม่ถูกต้อง: \(ckError.localizedDescription)")
            
        // Storage Errors
        case .quotaExceeded:
            print("iCloud Storage เต็ม")
            // แสดง Alert ให้ User เพิ่ม Storage
            
        case .assetFileNotFound:
            print("ไฟล์ Asset ไม่พบ")
            
        case .assetFileModified:
            print("ไฟล์ Asset ถูกแก้ไขระหว่าง Upload")
            
        // Batch Errors
        case .partialFailure:
            print("Batch Operation ล้มเหลวบางส่วน")
            if let errors = ckError.partialErrorsByItemID {
                for (itemID, itemError) in errors {
                    print("Error for \(itemID): \(itemError)")
                }
            }
            
        case .limitExceeded:
            print("เกิน Limit - แบ่งเป็น Batch ย่อยกว่านี้")
            
        // Zone Errors
        case .zoneNotFound:
            print("Zone ไม่พบ - ต้องสร้างก่อน")
            
        case .userDeletedZone:
            print("User ลบ Zone แล้ว - ต้อง Setup ใหม่")
            
        default:
            print("CloudKit error \(ckError.code): \(ckError.localizedDescription)")
        }
    }
    
    private static func scheduleRetry(_ block: (() -> Void)?, after delay: Double) {
        guard let block = block else { return }
        DispatchQueue.main.asyncAfter(deadline: .now() + delay) {
            block()
        }
    }
}
```

---

## 43.24 Testing CloudKit

```swift
// Unit Testing กับ CloudKit
import XCTest
import CloudKit
@testable import MyApp

class CloudKitTests: XCTestCase {
    
    // ใช้ Development Environment สำหรับ Testing
    var container: CKContainer!
    var database: CKDatabase!
    
    override func setUpWithError() throws {
        container = CKContainer(identifier: "iCloud.com.example.myapp.dev")
        database = container.privateCloudDatabase
    }
    
    // Test Record Creation
    func testCreateRecord() async throws {
        let record = CKRecord(recordType: "TestRecord")
        record["testField"] = "testValue" as CKRecordValue
        
        let savedRecord = try await database.save(record)
        
        XCTAssertEqual(savedRecord["testField"] as? String, "testValue")
        
        // Cleanup
        try await database.deleteRecord(withID: savedRecord.recordID)
    }
    
    // Test Query
    func testQuery() async throws {
        // Create test records
        let record1 = CKRecord(recordType: "TestNote")
        record1["title"] = "Test 1" as CKRecordValue
        
        let record2 = CKRecord(recordType: "TestNote")
        record2["title"] = "Test 2" as CKRecordValue
        
        let _ = try await database.save(record1)
        let _ = try await database.save(record2)
        
        // Query
        let predicate = NSPredicate(format: "title BEGINSWITH %@", "Test")
        let query = CKQuery(recordType: "TestNote", predicate: predicate)
        
        let (results, _) = try await database.records(matching: query)
        
        XCTAssertGreaterThanOrEqual(results.count, 2)
        
        // Cleanup
        try await database.deleteRecord(withID: record1.recordID)
        try await database.deleteRecord(withID: record2.recordID)
    }
}
```

---

## 43.25 สรุป

ในบทนี้เราได้เรียนรู้:

1. **CloudKit พื้นฐาน** - Container, Database ทั้ง 3 ประเภท
2. **CKRecord** - การสร้าง อ่าน อัปเดต ลบ Records
3. **Field Types** - String, Number, Date, Data, Location, Asset, Reference
4. **CKQuery** - การค้นหาด้วย NSPredicate และ Sort Descriptors
5. **CKSubscription** - Real-time Updates ผ่าน Push Notification
6. **CKOperation** - Batch Operations ที่มีประสิทธิภาพ
7. **CKAsset** - จัดเก็บ Binary Files
8. **CKReference** - Relationships ระหว่าง Records
9. **Custom Zones** - Atomic Transactions และ Fetch Changes
10. **Record Sharing** - Share Records ระหว่างผู้ใช้
11. **NSPersistentCloudKitContainer** - Core Data + CloudKit
12. **Error Handling** - จัดการ Error อย่างครบถ้วน

### Checklist

- [ ] สามารถ Setup CloudKit ใน Xcode Project ได้
- [ ] เข้าใจความแตกต่างของ Public/Private/Shared Database
- [ ] สร้าง CKRecord และกำหนด Fields ได้
- [ ] Query Records ด้วย NSPredicate ได้
- [ ] ตั้งค่า CKSubscription สำหรับ Real-time Updates
- [ ] ใช้ CKOperation สำหรับ Batch Operations
- [ ] จัดการ CKAsset สำหรับ Binary Files
- [ ] สร้าง Relationships ด้วย CKReference
- [ ] ใช้ NSPersistentCloudKitContainer กับ Core Data
- [ ] Handle CloudKit Errors อย่างเหมาะสม

### แหล่งเรียนรู้เพิ่มเติม

- [Apple CloudKit Documentation](https://developer.apple.com/documentation/cloudkit)
- [CloudKit Dashboard](https://developer.apple.com/icloud/cloudkit)
- [WWDC Sessions on CloudKit](https://developer.apple.com/videos/frameworks/cloudkit)
- [CloudKit Quickstart](https://developer.apple.com/icloud/cloudkit/)

---

*บทถัดไป: Part 44 - Firebase Integration*
