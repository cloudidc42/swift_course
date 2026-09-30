# Part 36: การ Debug และ Profiling ใน Swift (Debugging & Profiling)

## บทนำ

การ debug คือกระบวนการค้นหาและแก้ไข bug ในโปรแกรม ในขณะที่ profiling คือการวิเคราะห์ประสิทธิภาพของแอปพลิเคชัน ในบทนี้เราจะเรียนรู้เครื่องมือและเทคนิคต่างๆ ที่ Xcode และ Swift มีให้สำหรับกระบวนการเหล่านี้

---

## 36.1 Xcode Debugger พื้นฐาน

Xcode มี integrated debugger ที่ทรงพลัง ซึ่งช่วยให้หยุด execution ดู state ของโปรแกรม และ step ผ่านโค้ดได้

### Debug Area

Debug Area ใน Xcode แบ่งออกเป็น 2 ส่วน:
- **Variables View** (ซ้าย) - แสดงค่าของ variables ปัจจุบัน
- **Console** (ขวา) - แสดง output และรับ LLDB commands

### การเริ่ม Debug Session

1. **รัน app ใน Debug mode** - กด ▶ หรือ Cmd+R
2. **Attach to running process** - Debug → Attach to Process
3. **รัน scheme แบบ Debug** - เลือก Debug configuration ใน scheme

### Debug Toolbar

```
▶  ‖  ↷  →  ↓  ↑
│  │  │  │  │  │
│  │  │  │  │  └─ Step Out (Shift+F7)
│  │  │  │  └──── Step Into (F7)
│  │  │  └─────── Step Over (F6)
│  │  └────────── Continue (F5)
│  └───────────── Pause/Stop
└──────────────── Run
```

---

## 36.2 Breakpoints

Breakpoint คือจุดที่บอกให้ debugger หยุด execution เพื่อให้เราตรวจสอบ state ของโปรแกรม

### การสร้าง Breakpoint

**วิธีที่ 1: คลิกที่ gutter**
```
คลิกที่ตัวเลขบรรทัดทางซ้ายของ editor
⚫ = breakpoint ที่เปิดอยู่
○ = breakpoint ที่ปิดอยู่
```

**วิธีที่ 2: ใช้ keyboard shortcut**
- วาง cursor บนบรรทัดที่ต้องการ แล้วกด Cmd+\\

**วิธีที่ 3: เขียน breakpoint ใน code**
```swift
// ใน Swift สามารถเรียก breakpoint() function
func processData(_ data: [Int]) -> [Int] {
    // หยุดที่นี่เสมอ (Development เท่านั้น)
    // ต้องลบออกก่อน production
    
    let filtered = data.filter { $0 > 0 }
    return filtered.sorted()
}
```

### การจัดการ Breakpoints

```
Breakpoints Navigator (Cmd+8):
├── Project Breakpoints
│   ├── LoginViewController.swift:45
│   ├── NetworkService.swift:120
│   └── UserRepository.swift:88
├── Exception Breakpoints
│   └── All Exceptions
└── Swift Error Breakpoints
    └── All Swift Errors
```

### การเปิด/ปิด Breakpoints

- **ปิด breakpoint เดียว** - คลิกที่ breakpoint icon (เป็นสีเทา)
- **ปิด breakpoints ทั้งหมด** - Debug → Deactivate Breakpoints (Cmd+Y)
- **ลบ breakpoint** - ลาก breakpoint ออกจาก gutter หรือ right-click → Delete

---

## 36.3 Conditional Breakpoints

Conditional Breakpoint จะหยุดเฉพาะเมื่อเงื่อนไขที่กำหนดเป็นจริง

### การสร้าง Conditional Breakpoint

1. Right-click บน breakpoint icon
2. เลือก "Edit Breakpoint..."
3. กรอก condition ใน "Condition" field

```swift
// ตัวอย่าง code
func processItems(_ items: [Item]) {
    for (index, item) in items.enumerated() {
        // สร้าง conditional breakpoint ที่บรรทัดนี้
        // Condition: index == 5 && item.price > 1000
        process(item)
    }
}

// ตัวอย่าง conditions ที่ใช้บ่อย:
// index > 100
// user.id == "target-user-id"
// array.count == 0
// error != nil
// name == "สมชาย"
```

### Actions บน Breakpoint

```
Breakpoint Actions:
├── Log Message - print message เมื่อถึง breakpoint
├── Debugger Command - รัน LLDB command
├── Shell Command - รัน shell command
├── Sound - เล่นเสียง
├── Capture GPU Frame - สำหรับ Metal debugging
└── AppleScript
```

**ตัวอย่าง Log Message Action:**
```
Log: "เข้าถึง function processItems ด้วย items count: @items.count@"
```
- ใช้ `@expression@` เพื่อ evaluate expression

**ตัวอย่าง Debugger Command Action:**
```
po items
expr print("debug: \(items.count) items")
```

---

## 36.4 Symbolic Breakpoints

Symbolic Breakpoint หยุดเมื่อมีการเรียก function หรือ method เฉพาะ

### การสร้าง Symbolic Breakpoint

1. Breakpoints Navigator → ปุ่ม + ด้านล่าง
2. เลือก "Symbolic Breakpoint..."
3. ใส่ชื่อ symbol

```
ตัวอย่าง Symbols:
├── -[UIViewController viewDidLoad]     ← Objective-C method
├── UIViewController.viewDidLoad        ← Swift method
├── malloc                              ← C function
├── objc_msgSend                        ← ObjC runtime
└── MyApp.UserService.fetchUsers        ← Swift qualified name
```

### Use Cases สำหรับ Symbolic Breakpoints

```swift
// 1. หา view controller ที่ load
// Symbol: -[UIViewController viewDidLoad]
// → หยุดทุกครั้งที่ viewDidLoad ถูกเรียก

// 2. Debug Auto Layout constraint violations
// Symbol: UIViewAlertForUnsatisfiableConstraints

// 3. ตรวจสอบ deallocation
// Symbol: -[MyClass dealloc]

// 4. หา deprecated methods
// Symbol: -[UIAlertView show]
```

---

## 36.5 Exception Breakpoints

Exception Breakpoint หยุดเมื่อเกิด exception ก่อนที่จะ crash

### การสร้าง Exception Breakpoint

1. Breakpoints Navigator → +
2. เลือก "Exception Breakpoint..."
3. เลือก exception type:
   - **Objective-C** - สำหรับ NSException
   - **C++** - สำหรับ C++ exceptions
   - **All** - ทุก type

```swift
// โค้ดที่มักทำให้เกิด exception
class ViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // ❌ จะเกิด NSException ถ้า key ไม่ถูกต้อง
        let value = dict["nonexistent"] as! String  // force cast
        
        // ❌ Array out of bounds
        let array = [1, 2, 3]
        let element = array[10]  // Index out of range
        
        // ❌ Optional force unwrap
        let optional: String? = nil
        let value2 = optional!  // nil unwrapped
    }
}
```

### Swift Error Breakpoint

```
Breakpoints Navigator → + → Swift Error Breakpoint
```
หยุดเมื่อ Swift ขว้าง error (ก่อนที่จะถูก catch)

---

## 36.6 Watchpoints

Watchpoint หยุด execution เมื่อค่าของ variable เปลี่ยนแปลง

### การสร้าง Watchpoint

1. หยุดที่ breakpoint ก่อน
2. ใน Variables View, right-click บน variable
3. เลือก "Watch 'variableName'"

```swift
// ตัวอย่างการใช้งาน watchpoint
class UserSession {
    var isLoggedIn = false {
        didSet {
            // watchpoint จะหยุดตรงนี้เมื่อ isLoggedIn เปลี่ยน
            print("isLoggedIn changed to \(isLoggedIn)")
        }
    }
    
    var currentUser: User?
}

// การใช้ watchpoint ใน LLDB
// (lldb) watchpoint set variable self.isLoggedIn
// (lldb) watchpoint set expression -- &someVariable
// (lldb) watchpoint list
// (lldb) watchpoint delete 1
```

---

## 36.7 LLDB Commands

LLDB (Low Level Debugger) เป็น debugger ที่ Xcode ใช้ สามารถพิมพ์ command ใน console ได้โดยตรง

### คำสั่งพื้นฐาน

```bash
# ดู help
(lldb) help
(lldb) help expression

# ควบคุม execution
(lldb) continue    # หรือ c - ทำงานต่อ
(lldb) step        # หรือ s - step into
(lldb) next        # หรือ n - step over
(lldb) finish      # หรือ f - step out
(lldb) quit        # ออกจาก debugger

# ดู backtrace
(lldb) bt          # backtrace ทั้งหมด
(lldb) bt 10       # 10 frames ล่าสุด

# ดู current frame
(lldb) frame info
(lldb) frame select 2  # เลือก frame ที่ 2
```

---

## 36.8 po Command

`po` ย่อมาจาก "print object" ใช้สำหรับ print ค่าของ expression

```bash
# print ค่าพื้นฐาน
(lldb) po name
(lldb) po user.email
(lldb) po array.count

# print object
(lldb) po user
(lldb) po viewController.view.subviews

# print expression
(lldb) po 2 + 2
(lldb) po "Hello, " + name

# print array element
(lldb) po items[0]
(lldb) po items.first

# ตรวจสอบ type
(lldb) po type(of: user)

# call method
(lldb) po user.fullName()
(lldb) po service.isConnected
```

### การใช้ po กับ Custom Types

```swift
// ทำให้ po แสดงผลได้ดีขึ้น ด้วย CustomDebugStringConvertible
struct User {
    let id: String
    let name: String
    let email: String
}

extension User: CustomDebugStringConvertible {
    var debugDescription: String {
        "User(id: \(id), name: \(name), email: \(email))"
    }
}

// ใน LLDB:
// (lldb) po user
// User(id: "123", name: "สมชาย", email: "somchai@example.com")
```

---

## 36.9 p Command

`p` คือ "print" แบบง่าย แสดงค่าในรูปแบบ structured

```bash
# p แสดงรายละเอียดมากกว่า po ในบางกรณี
(lldb) p user
(lldb) p 42
(lldb) p "hello"

# p กับ type annotation
(lldb) p user as? AdminUser

# เปรียบเทียบ p vs po
# p: แสดง type information และ structure
# po: เรียก debugDescription/description

# ตัวอย่าง:
(lldb) p 42
# (Int) $R0 = 42

(lldb) po 42
# 42

(lldb) p "Hello"
# (String) $R1 = "Hello"

(lldb) po "Hello"
# Hello
```

---

## 36.10 Frame Commands

Frame commands ช่วยสำรวจ call stack

```bash
# ดู current frame
(lldb) frame info
# frame #0: 0x00000001... MyApp.processData(_:) at ViewController.swift:45

# ดู variables ใน current frame
(lldb) frame variable
(lldb) fr v           # ย่อ

# ดู variable เฉพาะ
(lldb) frame variable items
(lldb) fr v items

# ดู all local variables
(lldb) frame variable --no-args

# เลือก frame
(lldb) frame select 0   # frame ปัจจุบัน
(lldb) frame select 1   # frame ก่อนหน้า
(lldb) frame select -r 1  # relative frame

# ย้าย frame ขึ้น/ลง
(lldb) up    # หรือ frame select -r -1
(lldb) down  # หรือ frame select -r 1
```

---

## 36.11 Thread Commands

Thread commands ช่วยจัดการ multi-threaded debugging

```bash
# ดู threads ทั้งหมด
(lldb) thread list

# เลือก thread
(lldb) thread select 2

# ดู backtrace ของ thread ปัจจุบัน
(lldb) thread backtrace
(lldb) bt

# ดู backtrace ของทุก thread
(lldb) thread backtrace all
(lldb) bt all

# ดูสถานะ thread
(lldb) thread info

# Step ใน current thread
(lldb) thread step-over   # step over
(lldb) thread step-in     # step into
(lldb) thread step-out    # step out

# หยุด thread อื่น (ระหว่าง debug)
(lldb) thread return       # return จาก function ปัจจุบัน
(lldb) thread return 42    # return ค่า 42 จาก function
```

### ตัวอย่างการ Debug Race Condition

```swift
// โค้ดที่มี race condition
class Counter {
    var count = 0
    
    func increment() {
        // ❌ ไม่ thread-safe
        count += 1
    }
}

// การ debug ด้วย LLDB:
// (lldb) bt all  -- ดู backtrace ของทุก thread
// (lldb) thread list  -- ดูว่ามี threads อะไรบ้าง
// เมื่อพบว่า 2 threads เข้าถึง count พร้อมกัน = race condition
```

---

## 36.12 Debug View Hierarchy

View Hierarchy Debugger ช่วยตรวจสอบ UI ที่ซับซ้อน

### การเปิด View Hierarchy Debugger

1. รัน app บน simulator หรือ device
2. กด Debug → View Debugging → Capture View Hierarchy
3. หรือกดปุ่ม "Debug View Hierarchy" ใน debug toolbar

### คุณสมบัติของ View Hierarchy Debugger

```
3D Exploded View:
├── แสดง view layers ทั้งหมดแบบ 3D
├── คลิก view เพื่อดู properties
├── ตรวจสอบ constraints
├── ค้นหา view ที่ซ้อนกัน
└── ตรวจสอบ Auto Layout ปัญหา

Inspector Panel (ขวา):
├── Object Inspector - properties ของ view
│   ├── Frame
│   ├── Bounds
│   ├── Background color
│   └── Hidden/Alpha
├── Size Inspector - sizing information
└── Constraints - Auto Layout constraints
```

### การใช้งาน

```swift
// ตัวอย่าง Auto Layout bug ที่ View Hierarchy Debugger ช่วยได้
class BuggyViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let label = UILabel()
        label.text = "Hello"
        view.addSubview(label)
        
        // ❌ ลืมเพิ่ม constraints
        // label.translatesAutoresizingMaskIntoConstraints = false
        // NSLayoutConstraint.activate([...])
        
        // Debug View Hierarchy จะแสดงให้เห็นว่า
        // label อยู่ผิดตำแหน่งหรือมีขนาด 0
    }
}
```

---

## 36.13 Memory Graph Debugger

Memory Graph Debugger ช่วยค้นหา memory leaks และ retain cycles

### การเปิด Memory Graph Debugger

1. รัน app
2. Debug → Memory Graph หรือกดปุ่ม "Debug Memory Graph" ใน toolbar

### ทำความเข้าใจ Memory Graph

```
Memory Graph แสดง:
├── Objects ที่ยังอยู่ใน memory (nodes)
├── References ระหว่าง objects (edges)
├── Retain counts
└── Strong/Weak references
```

### ตัวอย่าง Retain Cycle

```swift
// ❌ Retain Cycle ที่ตรวจพบด้วย Memory Graph Debugger
class Parent {
    var child: Child?
    
    deinit {
        print("Parent deallocated")
    }
}

class Child {
    var parent: Parent?  // ❌ Strong reference ทำให้เกิด retain cycle
    
    deinit {
        print("Child deallocated")
    }
}

// ✅ แก้ด้วย weak reference
class Child {
    weak var parent: Parent?  // ✅ Weak reference
    
    deinit {
        print("Child deallocated")
    }
}

// Retain Cycle ที่พบบ่อยใน closures
class ViewController: UIViewController {
    var timer: Timer?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // ❌ Retain cycle: ViewController → closure → ViewController
        timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { _ in
            self.updateUI()  // strong reference to self
        }
    }
    
    // ✅ ใช้ [weak self]
    func fixedVersion() {
        timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { [weak self] _ in
            self?.updateUI()
        }
    }
    
    private func updateUI() { }
}
```

---

## 36.14 Instruments

Instruments เป็นเครื่องมือ profiling และ analysis ที่ทรงพลังของ Apple

### การเปิด Instruments

```
Product → Profile (Cmd+I)
หรือ
Xcode → Open Developer Tool → Instruments
```

### หน้าต่าง Instruments

```
Instruments UI:
├── Template Chooser - เลือก instrument type
├── Timeline - แสดง data ตามเวลา
├── Detail Pane - รายละเอียดของ selected item
└── Inspector - ข้อมูลเพิ่มเติม
```

---

## 36.15 Time Profiler

Time Profiler วัดว่าแต่ละ function ใช้เวลาทำงานนานแค่ไหน

### การใช้ Time Profiler

```
1. เปิด Instruments → เลือก "Time Profiler"
2. กด Record
3. ใช้ app (ทำ operations ที่ต้องการ profile)
4. กด Stop
5. วิเคราะห์ผลลัพธ์
```

### การอ่านผลลัพธ์ Time Profiler

```
Call Tree View:
├── Self Time - เวลาที่ function ใช้เอง
├── Total Time - รวม call ของ function ลูก
└── Calls - จำนวนครั้งที่เรียก

Heavy Side-bar (มีประโยชน์มาก):
└── แสดง call tree เรียงตาม self time
    ← หาตรงนี้เพื่อ optimize
```

### ตัวอย่างการ Optimize

```swift
// ❌ โค้ดที่ช้า (จะแสดงใน Time Profiler)
func processLargeDataset(_ data: [Int]) -> [Int] {
    var result: [Int] = []
    
    for item in data {
        // ❌ O(n²) - nested loop
        if !result.contains(item) {  // contains เป็น O(n)
            result.append(item)
        }
    }
    
    return result
}

// ✅ โค้ดที่เร็วกว่า
func processLargeDatasetOptimized(_ data: [Int]) -> [Int] {
    var seen = Set<Int>()  // O(1) lookup
    var result: [Int] = []
    
    for item in data {
        if seen.insert(item).inserted {
            result.append(item)
        }
    }
    
    return result
}
```

---

## 36.16 Allocations Instrument

Allocations instrument ติดตาม memory allocation ของแอป

### การใช้ Allocations

```
Template: Allocations
ข้อมูลที่แสดง:
├── All Allocations - allocation ทั้งหมด
├── Created & Destroyed - สร้างและลบ
├── Created & Still Living - ยังอยู่ใน memory
└── Persistent Bytes - bytes ที่ยังใช้อยู่
```

### Generation Analysis

```
Allocations Instrument → Generations:
1. Mark Generation A
2. ทำ operations ที่ต้องการตรวจสอบ
3. Mark Generation B
4. ทำ operations อีกครั้ง
5. Mark Generation C

เปรียบเทียบ:
- ถ้า B มี objects มากกว่า A หลังจากทำ operations เดิม
  → อาจมี memory leak
```

### ตัวอย่าง Memory Leak

```swift
// ❌ Memory Leak
class ImageCache {
    private var cache: [String: UIImage] = [:]
    
    func loadImage(url: String) -> UIImage? {
        if let cached = cache[url] {
            return cached
        }
        
        // โหลด image และ cache ไว้
        let image = downloadImage(from: url)
        cache[url] = image  // ❌ cache ไม่มี size limit = memory leak
        return image
    }
}

// ✅ แก้ด้วย NSCache
class ImageCacheFixed {
    private let cache = NSCache<NSString, UIImage>()
    
    init() {
        cache.countLimit = 100    // จำกัด 100 items
        cache.totalCostLimit = 50 * 1024 * 1024  // จำกัด 50MB
    }
    
    func loadImage(url: String) -> UIImage? {
        let key = url as NSString
        if let cached = cache.object(forKey: key) {
            return cached
        }
        
        let image = downloadImage(from: url)
        if let image = image {
            cache.setObject(image, forKey: key)
        }
        return image
    }
    
    private func downloadImage(from url: String) -> UIImage? {
        return nil  // Implementation
    }
}
```

---

## 36.17 Leaks Instrument

Leaks instrument ตรวจหา memory objects ที่ไม่มี reference แต่ยังอยู่ใน memory

### การใช้ Leaks Instrument

```
Template: Leaks
หรือ ใช้ร่วมกับ Allocations

Leaks Cycle ที่แสดง:
- Object graph ของ leaked objects
- Reference chain ที่ทำให้ leak
- Class name และ address
```

```swift
// ตัวอย่าง leak ที่ Leaks instrument ตรวจพบ
class NetworkManager {
    var completionHandlers: [String: (Data) -> Void] = [:]
    
    func request(url: String, completion: @escaping (Data) -> Void) {
        completionHandlers[url] = completion  // ❌ เก็บ completion handler
        // ถ้าลืม remove จาก dictionary = leak
        
        performRequest(url: url) { [weak self] data in
            self?.completionHandlers[url]?(data)
            // ❌ ลืม remove: self?.completionHandlers.removeValue(forKey: url)
        }
    }
    
    private func performRequest(url: String, completion: @escaping (Data) -> Void) {
        // Implementation
    }
}
```

---

## 36.18 Network Instrument

Network instrument ติดตาม network activity ของแอป

```
Network Instrument แสดง:
├── Connections - การเชื่อมต่อ
├── Bytes Sent/Received - ข้อมูลที่ส่ง/รับ
├── Duration - เวลาแต่ละ request
└── Errors - error ที่เกิดขึ้น
```

```swift
// ตัวอย่างการ optimize network ตาม Network Instrument findings
class APIClient {
    
    // ❌ ไม่มี caching
    func fetchUser(id: String) async throws -> User {
        let url = URL(string: "https://api.example.com/users/\(id)")!
        let (data, _) = try await URLSession.shared.data(from: url)
        return try JSONDecoder().decode(User.self, from: data)
    }
    
    // ✅ มี caching
    private var userCache: [String: User] = [:]
    
    func fetchUserCached(id: String) async throws -> User {
        if let cached = userCache[id] {
            return cached  // return จาก cache ไม่ต้อง network request
        }
        
        let url = URL(string: "https://api.example.com/users/\(id)")!
        let (data, _) = try await URLSession.shared.data(from: url)
        let user = try JSONDecoder().decode(User.self, from: data)
        userCache[id] = user
        return user
    }
}
```

---

## 36.19 Energy Instrument

Energy instrument วัดการใช้พลังงานของแอป

```
Energy Impact Indicators:
├── CPU usage
├── Network activity
├── Location services
├── Motion sensors
└── Background activity

Levels:
- Low (เขียว): ดี
- Medium (เหลือง): พอรับได้
- High (แดง): ควรปรับปรุง
```

```swift
// การ optimize energy usage

// ❌ ใช้พลังงานมาก - location updates บ่อยๆ
class LocationTracker {
    let locationManager = CLLocationManager()
    
    func startTracking() {
        locationManager.desiredAccuracy = kCLLocationAccuracyBest  // ❌ precise มาก
        locationManager.distanceFilter = 1  // ❌ update ทุก 1 เมตร
        locationManager.startUpdatingLocation()
    }
}

// ✅ ประหยัดพลังงาน
class EfficientLocationTracker {
    let locationManager = CLLocationManager()
    
    func startTracking() {
        locationManager.desiredAccuracy = kCLLocationAccuracyHundredMeters  // ✅ แม่นยำน้อยลง
        locationManager.distanceFilter = 50  // ✅ update ทุก 50 เมตร
        locationManager.startUpdatingLocation()
    }
    
    func stopTracking() {
        locationManager.stopUpdatingLocation()  // ✅ หยุดเมื่อไม่ต้องการ
    }
}
```

---

## 36.20 print vs debugPrint

Swift มีทั้ง `print` และ `debugPrint` สำหรับ output ข้อมูล

```swift
struct User {
    let name: String
    let age: Int
}

extension User: CustomStringConvertible {
    var description: String {
        "User: \(name), อายุ \(age) ปี"
    }
}

extension User: CustomDebugStringConvertible {
    var debugDescription: String {
        "User(name: \"\(name)\", age: \(age))"
    }
}

let user = User(name: "สมชาย", age: 30)

// print ใช้ description
print(user)
// Output: User: สมชาย, อายุ 30 ปี

// debugPrint ใช้ debugDescription
debugPrint(user)
// Output: User(name: "สมชาย", age: 30)

// ความแตกต่าง String:
let greeting = "สวัสดี"
print(greeting)       // สวัสดี
debugPrint(greeting)  // "สวัสดี"  (มี quotes)

// ตัวอย่างกับ Array
let names = ["สมชาย", "สมหญิง"]
print(names)       // ["สมชาย", "สมหญิง"]
debugPrint(names)  // ["สมชาย", "สมหญิง"]
```

### การใช้ print เพื่อ debug

```swift
// ❌ print แบบง่าย
func calculateTotal(items: [CartItem]) -> Double {
    let total = items.reduce(0) { $0 + $1.price }
    print(total)  // ไม่รู้ว่า output จาก function ไหน
    return total
}

// ✅ print พร้อม context
func calculateTotal(items: [CartItem]) -> Double {
    print("📊 calculateTotal: items.count = \(items.count)")
    
    let total = items.reduce(0) { $0 + $1.price }
    print("📊 calculateTotal: total = \(total)")
    
    return total
}

// ✅✅ ใช้ #function สำหรับ context อัตโนมัติ
func calculateTotal(items: [CartItem]) -> Double {
    print("[\(#function)] items: \(items.count), processing...")
    
    let total = items.reduce(0) { $0 + $1.price }
    print("[\(#function)] result: \(total)")
    
    return total
}

// สร้าง debug print function
func debugLog(_ message: Any,
              file: String = #file,
              function: String = #function,
              line: Int = #line) {
    #if DEBUG
    let fileName = (file as NSString).lastPathComponent
    print("🐛 [\(fileName):\(line)] \(function) → \(message)")
    #endif
}

// การใช้งาน
func processOrder(_ order: Order) {
    debugLog("Processing order: \(order.id)")
    // Output: 🐛 [OrderService.swift:45] processOrder(_:) → Processing order: ORD-001
}
```

---

## 36.21 dump Function

`dump` แสดงรายละเอียดของ object แบบ tree structure

```swift
struct Address {
    let street: String
    let city: String
    let country: String
}

struct Person {
    let name: String
    let age: Int
    let address: Address
    let hobbies: [String]
}

let person = Person(
    name: "สมชาย",
    age: 30,
    address: Address(street: "ถนนสุขุมวิท", city: "กรุงเทพ", country: "ไทย"),
    hobbies: ["อ่านหนังสือ", "ว่ายน้ำ", "เขียนโค้ด"]
)

// print แสดงแบบ CustomStringConvertible
print(person)

// dump แสดงแบบ tree
dump(person)
// ▿ Person
//   - name: "สมชาย"
//   - age: 30
//   ▿ address: Address
//     - street: "ถนนสุขุมวิท"
//     - city: "กรุงเทพ"
//     - country: "ไทย"
//   ▿ hobbies: 3 elements
//     - "อ่านหนังสือ"
//     - "ว่ายน้ำ"
//     - "เขียนโค้ด"

// dump กับ output parameter
var output = ""
dump(person, to: &output)
print("Dumped: \(output)")

// dump กับ maxDepth
dump(complexObject, maxDepth: 2)

// dump กับ maxItems
let bigArray = Array(1...100)
dump(bigArray, maxItems: 5)
// ▿ 100 elements
//   - 1
//   - 2
//   - 3
//   - 4
//   - 5
//     ... (95 more)
```

---

## 36.22 assert และ precondition

`assert` และ `precondition` ช่วยตรวจสอบสมมติฐานในโค้ด

### assert

```swift
// assert - ตรวจสอบเฉพาะ Debug build
func withdraw(amount: Double) {
    assert(amount > 0, "จำนวนเงินต้องมากกว่า 0")
    assert(amount <= balance, "ยอดเงินไม่เพียงพอ")
    
    balance -= amount
}

// assertionFailure - fail ทันที (Debug เท่านั้น)
func processPaymentMethod(_ method: String) {
    switch method {
    case "credit_card", "debit_card", "paypal":
        processCard(method)
    default:
        assertionFailure("ไม่รู้จัก payment method: \(method)")
    }
}

// assert ใช้ใน:
// - ตรวจสอบ preconditions ใน development
// - หยุดทำงานใน Debug เมื่อพบสถานะที่ไม่ควรเกิดขึ้น
// - จะ IGNORE ใน Release build
```

### precondition

```swift
// precondition - ตรวจสอบทั้ง Debug และ Release build
func getElement(at index: Int, from array: [Int]) -> Int {
    precondition(index >= 0, "Index ต้องไม่ติดลบ")
    precondition(index < array.count, "Index เกิน array bounds")
    
    return array[index]
}

// preconditionFailure - fail ทันที (ทั้ง Debug และ Release)
func divide(_ a: Double, by b: Double) -> Double {
    guard b != 0 else {
        preconditionFailure("ห้ามหารด้วย 0")
    }
    return a / b
}

// ใช้ precondition เมื่อ:
// - เป็นเงื่อนไขที่จำเป็นต้องเป็นจริงเสมอ
// - ถ้าไม่เป็นจริง = programmer error ที่ต้องแก้ไข
// - ทำงานทั้ง Debug และ Release
```

### ความแตกต่าง

```swift
// สรุปการใช้งาน:
//
// assert(condition, message)
// - ใช้ debug build เท่านั้น
// - ตรวจสอบ internal consistency
// - เมื่อ fail: print message + crash (debug only)
//
// precondition(condition, message)
// - ใช้ทั้ง debug และ release
// - ตรวจสอบ API contract
// - เมื่อ fail: crash ทุก build configuration
//
// fatalError(message)
// - ใช้ทุก build
// - โค้ดที่ไม่ควรเข้าถึง
// - never returns

enum Season { case spring, summer, autumn, winter }

func temperatureRange(for season: Season) -> ClosedRange<Double> {
    switch season {
    case .spring:  return 15...25
    case .summer:  return 30...40
    case .autumn:  return 15...25
    case .winter:  return 5...15
    // ไม่จำเป็นต้องมี default เพราะ enum exhaustive
    }
}
```

---

## 36.23 fatalError

`fatalError` หยุดโปรแกรมทันทีพร้อม message และไม่มีทางกลับ

```swift
// fatalError ใช้สำหรับ:

// 1. Protocol method ที่ต้อง override
class BaseViewController: UIViewController {
    func setupUI() {
        fatalError("ต้อง override setupUI() ใน subclass")
    }
}

class HomeViewController: BaseViewController {
    override func setupUI() {
        // ✅ override แล้ว ไม่ crash
        view.backgroundColor = .white
    }
}

// 2. Switch ที่ควรไม่มี default case
func processHTTPStatus(_ statusCode: Int) {
    switch statusCode {
    case 200:
        handleSuccess()
    case 404:
        handleNotFound()
    case 500:
        handleServerError()
    default:
        // โค้ดนี้ไม่ควรถูกเรียกถ้าเราจัดการทุก case แล้ว
        fatalError("HTTP status code ที่ไม่ได้จัดการ: \(statusCode)")
    }
}

// 3. Required initializer ที่ไม่รองรับ
class CustomView: UIView {
    init(frame: CGRect, style: ViewStyle) {
        super.init(frame: frame)
        configure(style: style)
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) ไม่รองรับ - ใช้ init(frame:style:) แทน")
    }
}

// 4. โค้ดที่ไม่ควรถึงได้
func getDay(from date: Date) -> String {
    let weekday = Calendar.current.component(.weekday, from: date)
    
    // weekday ใน Calendar: 1=Sun, 2=Mon, ..., 7=Sat
    switch weekday {
    case 1: return "อาทิตย์"
    case 2: return "จันทร์"
    case 3: return "อังคาร"
    case 4: return "พุธ"
    case 5: return "พฤหัสบดี"
    case 6: return "ศุกร์"
    case 7: return "เสาร์"
    default:
        fatalError("weekday ต้องอยู่ระหว่าง 1-7 แต่ได้ \(weekday)")
    }
}
```

---

## 36.24 os_log

`os_log` เป็น logging system ที่มีประสิทธิภาพสูงจาก Apple

```swift
import os.log

// การสร้าง log object
let log = OSLog(subsystem: "com.myapp.networking", category: "API")

// Log levels:
// .default - ข้อมูลทั่วไป
// .info    - ข้อมูลที่เป็นประโยชน์
// .debug   - debug information (ไม่แสดงใน release)
// .error   - error ที่เกิดขึ้น
// .fault   - critical error

func fetchData(from url: URL) {
    os_log("เริ่มโหลดข้อมูลจาก: %{public}@", log: log, type: .info, url.absoluteString)
    
    // ทำการ fetch...
    
    os_log("โหลดข้อมูลสำเร็จ", log: log, type: .info)
}

func handleError(_ error: Error) {
    os_log("เกิด error: %{public}@", log: log, type: .error, error.localizedDescription)
}

// Privacy Annotations:
// %{public}@  - แสดงค่าจริง (ใน logs ทั้งหมด)
// %{private}@ - ซ่อนค่าในบางกรณี (default สำหรับ string)
// %{sensitive}@ - ซ่อนค่าเสมอ (สำหรับข้อมูลส่วนตัว)

let userID = "user-123"
let password = "secret"
os_log("User: %{public}@, Password: %{private}@", log: log, userID, password)
// ใน release: User: user-123, Password: <private>
```

---

## 36.25 Logger (Unified Logging)

`Logger` เป็น API ใหม่ที่ type-safe กว่า `os_log` (iOS 14+)

```swift
import os

// สร้าง Logger
let logger = Logger(subsystem: "com.myapp", category: "network")

// Log ด้วย interpolation (type-safe)
func fetchUsers() async throws -> [User] {
    logger.info("เริ่มโหลด users")
    
    let users = try await networkClient.getUsers()
    logger.info("โหลดสำเร็จ: \(users.count) users")
    
    return users
}

func handleNetworkError(_ error: Error, url: URL) {
    logger.error("Network error: \(error.localizedDescription)")
    logger.error("URL ที่ fail: \(url.absoluteString, privacy: .public)")
}

// Privacy levels ใน Logger:
let userEmail = "user@example.com"
let sensitiveData = "credit-card-number"

logger.info("Email: \(userEmail, privacy: .public)")     // แสดงเสมอ
logger.info("Data: \(sensitiveData, privacy: .private)")  // ซ่อนใน release
logger.info("Hash: \(sensitiveData, privacy: .sensitive)") // hash ใน release

// Format options
let price: Double = 99.99
logger.info("ราคา: \(price, format: .fixed(precision: 2))")

let count: Int = 1500
logger.info("จำนวน: \(count, format: .decimal)")

// Log levels
logger.trace("Trace - ละเอียดมาก")    // Debug only
logger.debug("Debug - development")    // Debug only
logger.info("Info - ข้อมูลทั่วไป")   // ทุก build
logger.notice("Notice - สำคัญ")        // ทุก build
logger.warning("Warning - คำเตือน")    // ทุก build  
logger.error("Error - ข้อผิดพลาด")    // ทุก build
logger.critical("Critical - วิกฤต")   // ทุก build
logger.fault("Fault - ผิดพลาดร้ายแรง") // ทุก build
```

### การดู Logs ใน Console App

```
1. เปิด Console.app (ใน /Applications/Utilities/)
2. เลือก device/simulator
3. ค้นหาด้วย subsystem: "com.myapp"
4. Filter ตาม category, level
```

---

## 36.26 OSSignpost สำหรับ Performance Marks

OSSignpost ช่วยวัดประสิทธิภาพของโค้ดใน Instruments

```swift
import os

let signposter = OSSignposter(subsystem: "com.myapp", category: "performance")

// การใช้ signpost สำหรับวัด interval
func processLargeData(_ data: [Int]) -> [Int] {
    // เริ่มวัด
    let signpostID = signposter.makeSignpostID()
    let state = signposter.beginInterval("processLargeData", id: signpostID)
    
    defer {
        // หยุดวัดเมื่อออกจาก function
        signposter.endInterval("processLargeData", state)
    }
    
    // โค้ดที่ต้องการวัด
    return data.filter { $0 > 0 }.sorted()
}

// การใช้ signpost สำหรับ event
func userTappedButton() {
    signposter.emitEvent("ButtonTapped", "User tapped checkout button")
}

// การใช้ signpost แบบ withIntervalSignpost
func loadImages(urls: [URL]) async throws -> [UIImage] {
    try await signposter.withIntervalSignpost("loadImages") {
        var images: [UIImage] = []
        for url in urls {
            if let image = try await loadImage(from: url) {
                images.append(image)
            }
        }
        return images
    }
}

// ดูผลลัพธ์ใน Instruments:
// 1. เปิด Instruments
// 2. เลือก "os_signpost" template
// 3. รัน app
// 4. ดู intervals และ events ใน timeline
```

---

## 36.27 แบบฝึกหัดพร้อมเฉลย (Practical Exercises)

### แบบฝึกหัดที่ 1: การหา Memory Leak

**โจทย์:** ค้นหาและแก้ไข memory leak ในโค้ดต่อไปนี้

```swift
// โค้ดที่มี memory leak
class TaskManager {
    var tasks: [Task] = []
    var onTaskAdded: ((Task) -> Void)?
    
    func addTask(_ task: Task) {
        tasks.append(task)
        onTaskAdded?(task)
    }
}

class TaskViewController: UIViewController {
    let manager = TaskManager()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupCallbacks()
    }
    
    func setupCallbacks() {
        // ❌ Strong reference cycle
        manager.onTaskAdded = { task in
            self.showNotification(for: task)  // strong reference to self
        }
    }
    
    func showNotification(for task: Task) {
        print("เพิ่ม task: \(task.name)")
    }
}
```

**เฉลย:**

```swift
class TaskViewController: UIViewController {
    let manager = TaskManager()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupCallbacks()
    }
    
    func setupCallbacks() {
        // ✅ แก้ด้วย [weak self]
        manager.onTaskAdded = { [weak self] task in
            guard let self = self else { return }
            self.showNotification(for: task)
        }
        
        // หรือใช้ [unowned self] ถ้า self ต้องมีอยู่เสมอ
        // manager.onTaskAdded = { [unowned self] task in
        //     self.showNotification(for: task)
        // }
    }
    
    func showNotification(for task: Task) {
        print("เพิ่ม task: \(task.name)")
    }
}
```

### แบบฝึกหัดที่ 2: การ Debug ด้วย LLDB

**โจทย์:** โค้ดต่อไปนี้มี bug - หาและแก้ไข

```swift
// โค้ดที่มี bug
func calculateAverage(_ numbers: [Double]) -> Double {
    var total = 0.0
    
    for number in numbers {
        total = total + number  // อาจมี bug ตรงนี้
    }
    
    return total / Double(numbers.count)  // อาจ crash ถ้า empty array
}

// Bug 1: ถ้า numbers เป็น empty array จะได้ 0/0 = nan หรือ crash
// Bug 2: ไม่มี early return สำหรับ empty array
```

**เฉลย พร้อม debug steps:**

```swift
// Step 1: เพิ่ม breakpoint ที่บรรทัด return
// Step 2: ใน LLDB: po numbers (ดูค่า)
// Step 3: ใน LLDB: po total (ดู total ก่อน divide)
// Step 4: ใน LLDB: po numbers.count (ดู count)
// Step 5: พบว่า empty array ทำให้หารด้วย 0

// แก้ไข
func calculateAverage(_ numbers: [Double]) -> Double {
    guard !numbers.isEmpty else {
        return 0.0  // หรือ return nil ถ้า return type เป็น Double?
    }
    
    let total = numbers.reduce(0.0, +)
    return total / Double(numbers.count)
}

// หรือ
func calculateAverageSafe(_ numbers: [Double]) -> Double? {
    guard !numbers.isEmpty else { return nil }
    return numbers.reduce(0.0, +) / Double(numbers.count)
}
```

### แบบฝึกหัดที่ 3: Performance Optimization

**โจทย์:** Optimize function ต่อไปนี้

```swift
// โค้ดที่ช้า
func findDuplicates(in array: [Int]) -> [Int] {
    var duplicates: [Int] = []
    
    for i in 0..<array.count {
        for j in (i+1)..<array.count {
            if array[i] == array[j] && !duplicates.contains(array[i]) {
                duplicates.append(array[i])
            }
        }
    }
    
    return duplicates
}
// O(n³) complexity!
```

**เฉลย:**

```swift
// ✅ O(n) complexity
func findDuplicatesOptimized(in array: [Int]) -> [Int] {
    var seen = Set<Int>()
    var duplicates = Set<Int>()
    
    for number in array {
        if seen.contains(number) {
            duplicates.insert(number)
        } else {
            seen.insert(number)
        }
    }
    
    return Array(duplicates)
}

// ทดสอบประสิทธิภาพ
func testPerformance() {
    let largeArray = (1...10000).map { _ in Int.random(in: 1...1000) }
    
    let start1 = Date()
    let _ = findDuplicates(in: Array(largeArray.prefix(1000)))
    let time1 = Date().timeIntervalSince(start1)
    
    let start2 = Date()
    let _ = findDuplicatesOptimized(in: largeArray)
    let time2 = Date().timeIntervalSince(start2)
    
    print("แบบช้า (1000 elements): \(time1) วินาที")
    print("แบบเร็ว (10000 elements): \(time2) วินาที")
}
```

---

## 36.28 การหาและแก้ไข Bug ที่พบบ่อย

### 1. Force Unwrap Crash

```swift
// ❌ Force unwrap ที่อาจ crash
func getUsername() -> String {
    return UserDefaults.standard.string(forKey: "username")!
}

// ✅ แก้ด้วย nil coalescing
func getUsernameSafe() -> String {
    return UserDefaults.standard.string(forKey: "username") ?? "ผู้ใช้ไม่ระบุชื่อ"
}

// หรือ return optional
func getUsernameOptional() -> String? {
    return UserDefaults.standard.string(forKey: "username")
}
```

### 2. Index Out of Bounds

```swift
// ❌ อาจ crash
func getFirstItem<T>(_ array: [T]) -> T {
    return array[0]  // crash ถ้า array ว่าง
}

// ✅ ปลอดภัย
func getFirstItemSafe<T>(_ array: [T]) -> T? {
    return array.first
}

// หรือใช้ extension
extension Array {
    subscript(safe index: Int) -> Element? {
        guard index >= 0 && index < count else { return nil }
        return self[index]
    }
}

// ใช้งาน
let array = [1, 2, 3]
let element = array[safe: 10]  // nil แทน crash
```

### 3. Main Thread Violation

```swift
// ❌ Update UI จาก background thread
URLSession.shared.dataTask(with: url) { data, _, _ in
    if let data = data {
        self.imageView.image = UIImage(data: data)  // ❌ background thread!
    }
}.resume()

// ✅ ใช้ DispatchQueue.main
URLSession.shared.dataTask(with: url) { data, _, _ in
    if let data = data {
        DispatchQueue.main.async {
            self.imageView.image = UIImage(data: data)  // ✅ main thread
        }
    }
}.resume()

// ✅✅ ใช้ async/await (Swift 5.5+)
func loadImage(from url: URL) async {
    let (data, _) = try! await URLSession.shared.data(from: url)
    
    await MainActor.run {
        imageView.image = UIImage(data: data)
    }
}
```

### 4. Retain Cycle ใน Delegate

```swift
// ❌ Retain cycle ระหว่าง VC และ delegate
class MyViewController: UIViewController {
    var service: DataService!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        service = DataService()
        service.delegate = self  // VC owns service, service owns VC = cycle
    }
}

// ✅ ใช้ weak delegate
protocol DataServiceDelegate: AnyObject {
    func didReceiveData(_ data: [Item])
}

class DataService {
    weak var delegate: DataServiceDelegate?  // weak!
    
    func fetchData() {
        // fetch...
        delegate?.didReceiveData([])
    }
}
```

### 5. Thread Safety Issue

```swift
// ❌ Race condition
class Counter {
    var value = 0
    
    func increment() {
        value += 1  // ❌ ไม่ thread-safe
    }
}

// ✅ ใช้ actor
actor SafeCounter {
    var value = 0
    
    func increment() {
        value += 1  // ✅ actor-isolated
    }
}

// ✅ ใช้ DispatchQueue
class ThreadSafeCounter {
    private var _value = 0
    private let queue = DispatchQueue(label: "counter.queue", attributes: .concurrent)
    
    var value: Int {
        queue.sync { _value }
    }
    
    func increment() {
        queue.async(flags: .barrier) {
            self._value += 1
        }
    }
}
```

---

## 36.29 เทคนิค Debug ขั้นสูง

### Symbolic Breakpoint สำหรับ Auto Layout

```
Symbolic Breakpoint:
Symbol: UIViewAlertForUnsatisfiableConstraints

Action: Debugger Command
Command: po [[UIWindow keyWindow] _autolayoutTrace]
```

### การใช้ LLDB Script

```python
# ~/.lldbinit - เพิ่ม custom commands
command script import ~/.lldb/pretty_print.py
```

```python
# pretty_print.py
import lldb

def pretty_print(debugger, command, result, internal_dict):
    """Custom print command"""
    target = debugger.GetSelectedTarget()
    process = target.GetProcess()
    thread = process.GetSelectedThread()
    frame = thread.GetSelectedFrame()
    
    var = frame.FindVariable(command)
    print(f"Value: {var.GetValue()}")

def __lldb_init_module(debugger, internal_dict):
    debugger.HandleCommand('command script add -f pretty_print.pretty_print pp')
```

### การ Debug SwiftUI

```swift
// SwiftUI debugging techniques

struct ContentView: View {
    @State private var count = 0
    
    var body: some View {
        VStack {
            // Debug ว่า view render กี่ครั้ง
            let _ = print("ContentView body called, count: \(count)")
            
            Text("Count: \(count)")
            
            Button("Increment") {
                count += 1
            }
        }
        // ดู view hierarchy
        .border(Color.red, width: 1)  // Debug border
        // Debug background
        .background(Color.yellow.opacity(0.1))
    }
}

// ใช้ _printChanges() สำหรับ debug state changes (iOS 15+)
struct DebugView: View {
    @State private var name = ""
    
    var body: some View {
        // พิมพ์ว่า property ไหนทำให้ body re-render
        let _ = Self._printChanges()
        
        TextField("ชื่อ", text: $name)
    }
}
```

---

## 36.30 สรุป (Summary)

ในบทนี้เราได้เรียนรู้เครื่องมือและเทคนิคสำหรับการ debug และ profiling ใน Swift:

### เครื่องมือหลัก

| เครื่องมือ | ใช้สำหรับ |
|-----------|----------|
| Xcode Debugger | Step through โค้ด, ดู variables |
| Breakpoints | หยุด execution ณ จุดที่ต้องการ |
| Conditional Breakpoints | หยุดเมื่อเงื่อนไขเป็นจริง |
| Symbolic Breakpoints | หยุดเมื่อเรียก function เฉพาะ |
| Exception Breakpoints | หยุดเมื่อเกิด exception |
| LLDB | Debug จาก command line |
| View Hierarchy Debugger | ตรวจสอบ UI |
| Memory Graph | ค้นหา retain cycles |
| Instruments | Profiling ประสิทธิภาพ |
| Time Profiler | วัดเวลา execution |
| Allocations | วัด memory usage |
| Leaks | ค้นหา memory leaks |

### Logging APIs

| API | ใช้สำหรับ |
|----|---------|
| print | Quick debug output |
| debugPrint | แสดง debug description |
| dump | แสดง tree structure |
| os_log | Efficient logging (เก่า) |
| Logger | Modern logging (iOS 14+) |
| OSSignpost | Performance measurement |

### Best Practices

1. **ใช้ Conditional Breakpoints** - แทนการเพิ่ม print statements
2. **ใช้ Logger แทน print** - ใน production code
3. **Profile ก่อน optimize** - อย่า optimize สิ่งที่ไม่ใช่ bottleneck
4. **ใช้ weak references** - ใน closures และ delegate pattern
5. **ตรวจสอบ thread safety** - โดยเฉพาะ UI updates
6. **ใช้ assert/precondition** - เพื่อ document assumptions
7. **Memory Graph ก่อน Leaks** - เช็ค retain cycles ก่อน

### Quick Reference LLDB Commands

```bash
# Execution control
continue (c)    # ทำงานต่อ
next (n)        # step over
step (s)        # step into
finish (f)      # step out

# Inspection
po <expr>       # print object
p <expr>        # print
frame variable  # ดู variables
bt              # backtrace

# Breakpoints
b <function>    # breakpoint ที่ function
b <file>:<line> # breakpoint ที่ file:line
br list         # ดู breakpoints ทั้งหมด
br delete <n>   # ลบ breakpoint

# Thread
thread list     # ดู threads
thread select 2 # เลือก thread
bt all          # backtrace ทุก threads
```

---

*บทต่อไป: Part 37 - Advanced Swift Patterns*
