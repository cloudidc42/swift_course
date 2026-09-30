# Part 01: Introduction to Swift — แนะนำ Swift และการเตรียมสภาพแวดล้อม

[![Swift](https://img.shields.io/badge/Swift-5.9+-orange?logo=swift)](https://swift.org)
[![Level](https://img.shields.io/badge/Level-Beginner-green)](https://swift.org)
[![Part](https://img.shields.io/badge/Part-01%20of%20100-blue)](../README.md)

---

## 📋 สิ่งที่จะได้เรียนในตอนนี้

- Swift คืออะไร และประวัติความเป็นมา
- ทำไมต้องเรียน Swift
- Swift Ecosystem — แพลตฟอร์มที่ Swift รองรับ
- การติดตั้ง Xcode บน macOS
- การติดตั้ง Swift บน Linux
- Swift Playgrounds
- โปรแกรมแรก: Hello, World!
- Swift REPL
- ทำความเข้าใจ Syntax พื้นฐาน
- Comments ใน Swift
- โครงสร้างไฟล์ Swift
- การ Compile และรันโค้ด
- Swift Package Manager เบื้องต้น
- ภาพรวม Xcode Interface
- การรันแอปบน iOS Simulator
- แบบฝึกหัดพร้อมเฉลย
- สรุปและก้าวต่อไป

---

## 1. Swift คืออะไร?

**Swift** คือภาษาโปรแกรมมิ่งที่ **Apple** พัฒนาขึ้นมาเพื่อใช้สร้างแอปพลิเคชันบนแพลตฟอร์มของตัวเอง ได้แก่ iOS, macOS, watchOS, tvOS และยังรองรับการพัฒนาบน Linux อีกด้วย

Swift ถูกออกแบบมาให้เป็นภาษาที่:
- **ปลอดภัย (Safe)** — ตรวจจับข้อผิดพลาดตั้งแต่ตอน Compile Time
- **รวดเร็ว (Fast)** — ประสิทธิภาพเทียบเท่า C/C++
- **แสดงออกได้ดี (Expressive)** — โค้ดอ่านง่ายและเขียนสั้น
- **สนุก (Fun)** — Syntax ทันสมัย ไม่ยุ่งยาก

### 1.1 ประวัติ Swift

```
2010 — Chris Lattner เริ่มพัฒนา Swift เป็น Side Project ที่ Apple
2014 — Apple ประกาศ Swift 1.0 ที่งาน WWDC 2014
2015 — Swift 2.0 เปิด Open Source บน GitHub
2016 — Swift 3.0 เปลี่ยน API Design Guidelines ครั้งใหญ่
2017 — Swift 4.0 เพิ่ม Codable, String Improvements
2019 — Swift 5.0 เสถียร ABI ครั้งแรก (ABI Stability)
2019 — SwiftUI เปิดตัวที่ WWDC 2019
2021 — Swift 5.5 เพิ่ม async/await, Actors
2022 — Swift 5.7 เพิ่ม Opaque Types, Regex
2023 — Swift 5.9 เพิ่ม Macros, Parameter Packs
2024 — Swift 6.0 (Strict Concurrency)
```

### 1.2 Chris Lattner กับ Swift

**Chris Lattner** เป็นผู้สร้าง Swift และยังเป็นผู้สร้าง **LLVM Compiler Infrastructure** ซึ่งเป็น Compiler ที่ใช้กันอย่างแพร่หลายในปัจจุบัน

> "Swift is designed to be the first industrial-quality systems programming language that is as expressive and enjoyable as a scripting language."
> — Chris Lattner

---

## 2. ทำไมต้องเรียน Swift?

### 2.1 ตลาดงานและโอกาส

Swift เป็นภาษาหลักสำหรับการพัฒนาแอป Apple โดย App Store มีแอปกว่า **1.8 ล้านแอป** และนักพัฒนา iOS/macOS มีรายได้เฉลี่ยสูงมากในตลาดงาน

```
💼 ตำแหน่งงานที่ต้องการ Swift:
   - iOS Developer
   - macOS Developer
   - tvOS/watchOS Developer
   - Full-Stack Swift Developer (Server-Side)
   - Swift Framework Engineer
```

### 2.2 เปรียบเทียบ Swift กับ Objective-C

| ด้าน | Swift | Objective-C |
|------|-------|-------------|
| ไวยากรณ์ | ทันสมัย อ่านง่าย | เก่า ยากอ่าน |
| ความปลอดภัย | Optionals, Type Safety | ไม่มี Optionals |
| ประสิทธิภาพ | เร็วกว่า | ช้ากว่า |
| Interop | รองรับ ObjC | รองรับ Swift บางส่วน |
| อนาคต | หลักของ Apple | Legacy |

### 2.3 เปรียบเทียบ Swift กับ Python

```swift
// Swift
let numbers = [1, 2, 3, 4, 5]
let doubled = numbers.map { $0 * 2 }
print(doubled) // [2, 4, 6, 8, 10]
```

```python
# Python
numbers = [1, 2, 3, 4, 5]
doubled = list(map(lambda x: x * 2, numbers))
print(doubled)  # [2, 4, 6, 8, 10]
```

Swift สะอาดกว่า Python ในหลายกรณี แต่ยังมี Type Safety ที่ Python ขาด

### 2.4 Swift บน GitHub

```
⭐ Swift Repository: github.com/apple/swift
   Stars: 67,000+
   Contributors: 1,000+
   ปล่อยเป็น Open Source ตั้งแต่ปี 2015
```

---

## 3. Swift Ecosystem — แพลตฟอร์มที่รองรับ

### 3.1 Apple Platforms

```
📱 iOS       — iPhone & iPad Apps
💻 macOS     — Mac Desktop/Laptop Apps
⌚ watchOS   — Apple Watch Apps
📺 tvOS      — Apple TV Apps
🥽 visionOS  — Apple Vision Pro Apps (ใหม่!)
```

### 3.2 Non-Apple Platforms

```
🐧 Linux     — Ubuntu, CentOS, Amazon Linux
🪟 Windows   — Windows 10/11 (Experimental)
🌐 WASM      — WebAssembly (Swift 5.9+)
🔌 Embedded  — Microcontrollers (RP2040, STM32)
```

### 3.3 Use Cases ของ Swift

```
📱 Mobile Apps    — iOS/Android (via multiplatform)
🖥️ Desktop Apps   — macOS, Linux
🌐 Web Backend    — Vapor, Hummingbird
🤖 Machine Learning — Core ML, Create ML
🔧 System Tools   — CLI Tools, Scripts
📦 Libraries      — Swift Packages
```

---

## 4. การติดตั้ง Xcode บน macOS

### 4.1 ความต้องการระบบ

| ข้อกำหนด | ค่าต่ำสุด | แนะนำ |
|---------|---------|------|
| macOS | 13.0 (Ventura) | 14.0 (Sonoma) |
| Storage | 12 GB | 20 GB |
| RAM | 8 GB | 16 GB |
| Xcode | 14.0 | 15.0+ |

### 4.2 วิธีติดตั้ง Xcode

**วิธีที่ 1: Mac App Store (แนะนำ)**
1. เปิด App Store บน Mac
2. ค้นหา "Xcode"
3. กด "Get" หรือ "Install"
4. รอการดาวน์โหลด (ประมาณ 10-12 GB)

**วิธีที่ 2: Apple Developer Website**
1. ไปที่ [developer.apple.com/downloads](https://developer.apple.com/downloads)
2. ล็อกอินด้วย Apple ID
3. ดาวน์โหลด `.xip` file
4. ดับเบิลคลิกเพื่อแตกไฟล์
5. ย้ายไปที่โฟลเดอร์ Applications

### 4.3 ตรวจสอบการติดตั้ง

เปิด Terminal และพิมพ์:

```bash
# ตรวจสอบ Swift version
swift --version
# ผลลัพธ์ตัวอย่าง:
# swift-driver version: 1.87.1 Apple Swift version 5.9 (swiftlang-5.9.0.128.6 clang-1500.0.40.1)
# Target: arm64-apple-macosx14.0

# ตรวจสอบ Xcode version
xcodebuild -version
# ผลลัพธ์ตัวอย่าง:
# Xcode 15.0
# Build version 15A240d

# ตรวจสอบ Command Line Tools
xcode-select --print-path
# ผลลัพธ์: /Applications/Xcode.app/Contents/Developer
```

### 4.4 ติดตั้ง Command Line Tools เท่านั้น (ไม่มี Xcode)

ถ้าต้องการแค่ Swift สำหรับการพัฒนา Command Line:

```bash
xcode-select --install
```

---

## 5. การติดตั้ง Swift บน Linux

### 5.1 ติดตั้งบน Ubuntu 22.04

```bash
# 1. อัพเดต package list
sudo apt-get update

# 2. ติดตั้ง dependencies
sudo apt-get install -y \
    binutils \
    git \
    gnupg2 \
    libc6-dev \
    libcurl4-openssl-dev \
    libedit2 \
    libgcc-9-dev \
    libpython3.8 \
    libsqlite3-0 \
    libstdc++-9-dev \
    libxml2-dev \
    libz3-dev \
    pkg-config \
    tzdata \
    unzip \
    zlib1g-dev

# 3. ดาวน์โหลด Swift
SWIFT_VERSION="swift-5.9.2-RELEASE"
UBUNTU_VERSION="ubuntu22.04"
wget https://download.swift.org/swift-5.9.2-release/${UBUNTU_VERSION}/${SWIFT_VERSION}/${SWIFT_VERSION}-${UBUNTU_VERSION}.tar.gz

# 4. แตกไฟล์
tar xzf ${SWIFT_VERSION}-${UBUNTU_VERSION}.tar.gz

# 5. ย้ายไปที่ /usr/local
sudo mv ${SWIFT_VERSION}-${UBUNTU_VERSION} /usr/local/${SWIFT_VERSION}

# 6. เพิ่ม PATH
echo 'export PATH="/usr/local/swift-5.9.2-RELEASE/usr/bin:${PATH}"' >> ~/.bashrc
source ~/.bashrc

# 7. ตรวจสอบ
swift --version
```

### 5.2 ติดตั้งผ่าน Docker (ง่ายที่สุด)

```bash
# ดึง Swift image
docker pull swift:5.9

# รัน Container
docker run -it swift:5.9 swift repl
```

### 5.3 ใช้ Swift บน Windows (WSL2)

```powershell
# 1. ติดตั้ง WSL2
wsl --install -d Ubuntu-22.04

# 2. เปิด Ubuntu แล้วทำตามขั้นตอน Linux ด้านบน
```

---

## 6. Swift Playgrounds

**Swift Playgrounds** คือสภาพแวดล้อมแบบ Interactive ที่ช่วยให้เขียนและทดสอบโค้ด Swift ได้ทันที โดยเห็นผลลัพธ์ฝั่งขวาทันที

### 6.1 Playgrounds บน Xcode

1. เปิด Xcode
2. เลือก **File → New → Playground**
3. เลือก Template (Blank, Game, Map, Single View)
4. ตั้งชื่อและกด Create

```swift
// ตัวอย่างใน Playground
import Foundation

// ลองพิมพ์ข้อความ
print("สวัสดี Swift!")

// คำนวณค่า
let result = 10 + 20
print("ผลลัพธ์คือ: \(result)") // 30

// Playground แสดงผลทางขวาทันที!
```

### 6.2 Playgrounds บน iPad

แอป **Swift Playgrounds** บน iPad/Mac ช่วยให้เด็กและผู้เริ่มต้นเรียน Swift ได้สนุก:
- มี Curriculum พร้อมใช้
- รองรับ SwiftUI
- Drag & Drop Code Blocks สำหรับเด็ก

### 6.3 Online Playgrounds

สำหรับผู้ที่ยังไม่มี Mac:

| เว็บไซต์ | URL | หมายเหตุ |
|---------|-----|---------|
| Online Swift Playground | swiftplayground.run | ฟรี |
| repl.it | replit.com | มี Collaboration |
| Ideone | ideone.com | รองรับ Swift |

---

## 7. โปรแกรมแรก: Hello, World!

### 7.1 สร้างไฟล์ Hello.swift

สร้างไฟล์ชื่อ `hello.swift`:

```swift
// hello.swift
// โปรแกรมแรกของเรา

print("Hello, World!")
print("สวัสดี โลก!")
print("Hello, Swift!")
```

### 7.2 Compile และรัน

```bash
# วิธีที่ 1: รันตรง (ไม่ต้อง compile)
swift hello.swift
# Output:
# Hello, World!
# สวัสดี โลก!
# Hello, Swift!

# วิธีที่ 2: Compile ก่อนแล้วรัน
swiftc hello.swift -o hello
./hello
```

### 7.3 ทำความเข้าใจโค้ด

```swift
print("Hello, World!")
```

- `print` คือ **ฟังก์ชัน** (Function) ที่ Built-in มาใน Swift
- `"Hello, World!"` คือ **String Literal** — ข้อความที่อยู่ในเครื่องหมายคำพูด
- ใน Swift **ไม่ต้องใส่ Semicolon** `;` ท้ายบรรทัด (แต่ใส่ได้ถ้าต้องการ)
- **ไม่ต้องมี main() function** สำหรับ script ธรรมดา

### 7.4 Hello World ขั้นสูงขึ้น

```swift
// ใช้ตัวแปร
let greeting = "Hello"
let name = "Swift"
print("\(greeting), \(name)!")  // Hello, Swift!

// String Interpolation — การแทรกตัวแปรในสตริง
let year = 2024
print("Swift เปิดตัวปี \(2024 - 10) และยังคงพัฒนาถึงปี \(year)")

// Multi-line String
let poem = """
    สวัสดี Swift
    ภาษาแห่งอนาคต
    ปลอดภัย รวดเร็ว สวยงาม
    """
print(poem)
```

---

## 8. Swift REPL

**REPL** ย่อมาจาก **Read-Eval-Print Loop** คือสภาพแวดล้อมที่ให้เราพิมพ์โค้ดทีละบรรทัดและเห็นผลทันที

### 8.1 เปิด Swift REPL

```bash
swift repl
# หรือ
swift
```

จะเห็น Prompt แบบนี้:
```
Welcome to Apple Swift version 5.9 (swiftlang-5.9.0.128.6 clang-1500.0.40.1).
Type :help for assistance.
  1>
```

### 8.2 ใช้งาน REPL

```swift
// พิมพ์ใน REPL ทีละบรรทัด
  1> let x = 10
x: Int = 10
  2> let y = 20
y: Int = 20
  3> x + y
$R0: Int = 30
  4> print("ผลรวม = \(x + y)")
ผลรวม = 30
  5> :quit  // ออกจาก REPL
```

### 8.3 คำสั่ง REPL ที่มีประโยชน์

```
:help       — แสดงคำช่วยเหลือ
:quit       — ออกจาก REPL
:type <expr>— แสดง Type ของ expression
:print <var>— แสดงค่าตัวแปร
```

---

## 9. ทำความเข้าใจ Swift Syntax พื้นฐาน

### 9.1 Statements

Swift ใช้ **Newline** (ขึ้นบรรทัดใหม่) เป็นตัวแบ่ง Statement แต่ก็ใช้ `;` ได้:

```swift
// แบบปกติ — แต่ละ Statement อยู่คนละบรรทัด
let a = 1
let b = 2
let c = a + b

// หลาย Statement ในบรรทัดเดียว (ใช้ ; คั่น)
let x = 1; let y = 2; let z = x + y
```

### 9.2 Case Sensitivity

Swift **แยกแยะตัวพิมพ์ใหญ่-เล็ก** (Case Sensitive):

```swift
let name = "Swift"    // ตัวแปรชื่อ name
let Name = "Apple"    // ตัวแปรคนละตัว! ชื่อ Name
let NAME = "SWIFT"    // อีกตัว ชื่อ NAME

// print, Print, PRINT คือคนละสิ่ง
print("hello")   // ถูกต้อง
// Print("hello") // Error! ไม่มีฟังก์ชัน Print
```

### 9.3 Keywords ใน Swift

```swift
// Keywords ที่ต้องรู้ในเบื้องต้น:
let      // ประกาศค่าคงที่
var      // ประกาศตัวแปร
func     // ประกาศฟังก์ชัน
class    // ประกาศคลาส
struct   // ประกาศ Structure
enum     // ประกาศ Enum
if       // เงื่อนไข
else     // เงื่อนไขทางเลือก
for      // วนซ้ำ
while    // วนซ้ำ
return   // ส่งค่ากลับ
true     // ค่าจริง (Boolean)
false    // ค่าเท็จ (Boolean)
nil      // ไม่มีค่า (Optional)
```

---

## 10. Comments ใน Swift

Comments คือข้อความอธิบายโค้ดที่ Compiler จะไม่สนใจ มีประโยชน์มากสำหรับการอธิบายโค้ดให้คนอื่น (หรือตัวเองในอนาคต) เข้าใจ

### 10.1 Single-Line Comment

```swift
// นี่คือ Comment บรรทัดเดียว
let x = 10  // ประกาศตัวแปร x มีค่า 10

// สามารถใช้ภาษาไทยใน Comment ได้
// คำนวณ BMI
let weight = 70.0   // น้ำหนักเป็นกิโลกรัม
let height = 1.75   // ส่วนสูงเป็นเมตร
let bmi = weight / (height * height)
```

### 10.2 Multi-Line Comment

```swift
/*
   นี่คือ Comment
   หลายบรรทัด
   สามารถเขียนได้ยาวเท่าที่ต้องการ
*/

let y = 20

/*
 * ฟังก์ชัน calculateArea
 * คำนวณพื้นที่สี่เหลี่ยม
 * Parameters:
 *   - width: ความกว้าง
 *   - height: ความสูง
 * Returns: พื้นที่
 */
func calculateArea(width: Double, height: Double) -> Double {
    return width * height
}
```

### 10.3 Nested Comments (คุณสมบัติพิเศษของ Swift!)

Swift รองรับ **Nested Comments** ซึ่ง C/Java ไม่รองรับ:

```swift
/* ส่วนนี้ทั้งหมดเป็น Comment
   /* แม้แต่ Comment ซ้อนก็ได้! */
   print("บรรทัดนี้จะไม่ถูกรัน")
*/

print("บรรทัดนี้รันได้ตามปกติ")
```

### 10.4 Documentation Comments

ใช้สำหรับ Generate เอกสาร (ใช้กับ Xcode):

```swift
/// คำนวณพื้นที่วงกลม
///
/// - Parameter radius: รัศมีของวงกลม
/// - Returns: พื้นที่ของวงกลม
/// - Note: ใช้ค่า π ≈ 3.14159265
func circleArea(radius: Double) -> Double {
    return Double.pi * radius * radius
}
```

---

## 11. โครงสร้างไฟล์ Swift

### 11.1 ไฟล์ Swift ทั่วไป

```swift
// MyFile.swift

// 1. Import Statements
import Foundation
import UIKit  // ใช้เฉพาะใน iOS Projects

// 2. Global Constants/Variables (ไม่แนะนำสำหรับโปรเจกต์ใหญ่)
let appVersion = "1.0.0"

// 3. Type Definitions (struct, class, enum, protocol)
struct Person {
    var name: String
    var age: Int
}

// 4. Extension
extension Person {
    func greet() -> String {
        return "สวัสดี ฉันชื่อ \(name) อายุ \(age) ปี"
    }
}

// 5. Global Functions (ไม่แนะนำ แต่ใช้ได้)
func createPerson(name: String, age: Int) -> Person {
    return Person(name: name, age: age)
}
```

### 11.2 โครงสร้าง Swift Project

```
MyProject/
├── MyProject/
│   ├── MyProjectApp.swift     ← Entry Point
│   ├── ContentView.swift      ← SwiftUI View
│   ├── Models/
│   │   └── User.swift
│   ├── ViewModels/
│   │   └── UserViewModel.swift
│   ├── Views/
│   │   └── UserListView.swift
│   └── Resources/
│       └── Assets.xcassets
├── MyProjectTests/
│   └── MyProjectTests.swift
└── Package.swift              ← สำหรับ Swift Package
```

### 11.3 Entry Point ของโปรแกรม

**Command Line Program:**
```swift
// main.swift — ไฟล์พิเศษที่เป็น Entry Point
print("โปรแกรมเริ่มต้น")
let name = "Swift"
print("Hello, \(name)!")
```

**iOS App (SwiftUI):**
```swift
// MyApp.swift
import SwiftUI

@main  // บอก Compiler ว่านี่คือ Entry Point
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

---

## 12. การ Compile และรันโค้ด Swift

### 12.1 swiftc — Swift Compiler

```bash
# Compile ไฟล์เดียว
swiftc hello.swift

# Compile และตั้งชื่อ Output
swiftc hello.swift -o myprogram
./myprogram

# Compile หลายไฟล์
swiftc main.swift helper.swift -o myapp
./myapp

# ดู Assembly Code
swiftc -emit-assembly hello.swift

# ดู SIL (Swift Intermediate Language)
swiftc -emit-sil hello.swift

# Optimize (สำหรับ Production)
swiftc -O hello.swift -o hello_optimized
```

### 12.2 swift — รันตรงโดยไม่ต้อง Compile

```bash
# รันไฟล์โดยตรง
swift hello.swift

# รัน Script แบบ Unix Shebang
#!/usr/bin/swift
# บันทึกเป็น script.swift แล้ว chmod +x
```

### 12.3 Debug vs Release

```bash
# Debug Build (ค่าเริ่มต้น) — มี Debug Info
swiftc -g hello.swift

# Release Build — Optimize สูงสุด
swiftc -O -wmo hello.swift

# ดู Optimization Levels
# -O    — Optimize (แนะนำ)
# -Onone — ไม่ Optimize (Debug)
# -Osize — Optimize for Size
```

---

## 13. Swift Package Manager (SPM) เบื้องต้น

**Swift Package Manager (SPM)** เป็นเครื่องมือสำหรับจัดการ Dependencies และ Build โปรเจกต์ Swift — คล้าย npm (Node.js), pip (Python), Gradle (Java)

### 13.1 สร้าง Package ใหม่

```bash
# สร้างโฟลเดอร์
mkdir MyPackage
cd MyPackage

# สร้าง Package (executable)
swift package init --type executable

# โครงสร้างที่ได้:
# MyPackage/
# ├── Package.swift
# ├── Sources/
# │   └── MyPackage/
# │       └── main.swift
# └── Tests/
#     └── MyPackageTests/
#         └── MyPackageTests.swift
```

### 13.2 Package.swift

```swift
// Package.swift
// swift-tools-version: 5.9
import PackageDescription

let package = Package(
    name: "MyPackage",
    platforms: [
        .macOS(.v13)  // รองรับ macOS 13+
    ],
    dependencies: [
        // เพิ่ม Dependencies ที่นี่
        .package(
            url: "https://github.com/apple/swift-argument-parser",
            from: "1.2.0"
        ),
    ],
    targets: [
        .executableTarget(
            name: "MyPackage",
            dependencies: [
                .product(name: "ArgumentParser", package: "swift-argument-parser")
            ]
        ),
        .testTarget(
            name: "MyPackageTests",
            dependencies: ["MyPackage"]
        ),
    ]
)
```

### 13.3 คำสั่ง SPM ที่ใช้บ่อย

```bash
# Build
swift build

# รัน
swift run

# Test
swift test

# แสดง Dependencies
swift package show-dependencies

# อัพเดต Dependencies
swift package update

# Generate Xcode Project
swift package generate-xcodeproj

# Clean Build
swift package clean
```

---

## 14. ภาพรวม Xcode Interface

### 14.1 ส่วนต่างๆ ของ Xcode

```
┌─────────────────────────────────────────────────────┐
│  Toolbar: Run, Stop, Scheme Selector, Status Bar    │
├──────────┬─────────────────────────────┬────────────┤
│          │                             │            │
│ Navigator│      Editor Area            │ Inspector  │
│ (ซ้าย)   │      (กลาง)                │ (ขวา)     │
│          │                             │            │
│ - Files  │  โค้ด/Interface Builder     │ - Attributes│
│ - Search │                             │ - Size     │
│ - Issues │                             │ - Identity │
│ - Tests  │                             │            │
│          │                             │            │
├──────────┴─────────────────────────────┴────────────┤
│  Debug Area (ล่าง): Console Output, Variables       │
└─────────────────────────────────────────────────────┘
```

### 14.2 Keyboard Shortcuts ที่ต้องรู้

| Shortcut | Action |
|---------|--------|
| `⌘ + R` | Build & Run |
| `⌘ + B` | Build |
| `⌘ + .` | Stop |
| `⌘ + /` | Comment/Uncomment |
| `⌘ + shift + K` | Clean Build Folder |
| `⌘ + shift + O` | Open Quickly |
| `⌘ + click` | Jump to Definition |
| `ctrl + I` | Re-indent Code |

### 14.3 สร้างโปรเจกต์ iOS ใหม่

1. เปิด Xcode
2. **File → New → Project**
3. เลือก Template: **iOS → App**
4. กรอกข้อมูล:
   - Product Name: MyFirstApp
   - Team: ใส่ Apple ID (ฟรี)
   - Bundle Identifier: com.yourname.MyFirstApp
   - Interface: SwiftUI
   - Language: Swift
5. กด **Next** แล้วเลือกที่บันทึก

---

## 15. รันแอปแรกบน iOS Simulator

### 15.1 สร้างแอป Hello World ด้วย SwiftUI

หลังจากสร้างโปรเจกต์ ไฟล์ `ContentView.swift` จะมีโค้ดดังนี้:

```swift
// ContentView.swift
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

### 15.2 แก้ไขให้แสดงข้อความภาษาไทย

```swift
// ContentView.swift
import SwiftUI

struct ContentView: View {
    @State private var message = "สวัสดี Swift!"
    
    var body: some View {
        VStack(spacing: 20) {
            Image(systemName: "swift")
                .imageScale(.large)
                .foregroundStyle(.orange)
                .font(.system(size: 60))
            
            Text(message)
                .font(.title)
                .fontWeight(.bold)
            
            Text("ยินดีต้อนรับสู่โลกของ Swift")
                .font(.subheadline)
                .foregroundStyle(.secondary)
            
            Button("เปลี่ยนข้อความ") {
                message = "Hello, Swift World! 🎉"
            }
            .buttonStyle(.borderedProminent)
        }
        .padding()
    }
}

#Preview {
    ContentView()
}
```

### 15.3 เลือก Simulator และรัน

1. คลิกที่ **Scheme Selector** (ส่วนบน Toolbar)
2. เลือก Simulator เช่น "iPhone 15 Pro"
3. กด **⌘ + R** หรือคลิกปุ่ม Run (▶)
4. รอ Simulator เปิด
5. แอปจะรันบน Simulator

---

## 16. ตัวอย่างโค้ดเพิ่มเติม

### 16.1 โปรแกรม Calculator อย่างง่าย

```swift
// calculator.swift

func add(_ a: Double, _ b: Double) -> Double {
    return a + b
}

func subtract(_ a: Double, _ b: Double) -> Double {
    return a - b
}

func multiply(_ a: Double, _ b: Double) -> Double {
    return a * b
}

func divide(_ a: Double, _ b: Double) -> Double? {
    guard b != 0 else {
        print("⚠️ ไม่สามารถหารด้วยศูนย์ได้!")
        return nil
    }
    return a / b
}

// ทดสอบ
let x = 10.0
let y = 3.0

print("บวก: \(x) + \(y) = \(add(x, y))")
print("ลบ: \(x) - \(y) = \(subtract(x, y))")
print("คูณ: \(x) × \(y) = \(multiply(x, y))")

if let result = divide(x, y) {
    print("หาร: \(x) ÷ \(y) = \(String(format: "%.4f", result))")
}

// ทดสอบหารด้วย 0
let _ = divide(10, 0)
```

**Output:**
```
บวก: 10.0 + 3.0 = 13.0
ลบ: 10.0 - 3.0 = 7.0
คูณ: 10.0 × 3.0 = 30.0
หาร: 10.0 ÷ 3.0 = 3.3333
⚠️ ไม่สามารถหารด้วยศูนย์ได้!
```

### 16.2 โปรแกรมแปลงอุณหภูมิ

```swift
// temperature.swift

// แปลงเซลเซียสเป็นฟาเรนไฮต์
func celsiusToFahrenheit(_ celsius: Double) -> Double {
    return (celsius * 9/5) + 32
}

// แปลงฟาเรนไฮต์เป็นเซลเซียส
func fahrenheitToCelsius(_ fahrenheit: Double) -> Double {
    return (fahrenheit - 32) * 5/9
}

// แปลงเซลเซียสเป็นเคลวิน
func celsiusToKelvin(_ celsius: Double) -> Double {
    return celsius + 273.15
}

// แสดงตาราง
print("═══════════════════════════════════")
print("  ตารางแปลงอุณหภูมิ")
print("═══════════════════════════════════")
print(String(format: "%-12s %-12s %-12s", "Celsius", "Fahrenheit", "Kelvin"))
print("───────────────────────────────────")

let temperatures = [-40.0, 0.0, 20.0, 37.0, 100.0]
for temp in temperatures {
    let f = celsiusToFahrenheit(temp)
    let k = celsiusToKelvin(temp)
    print(String(format: "%-12.1f %-12.1f %-12.2f", temp, f, k))
}
print("═══════════════════════════════════")
```

**Output:**
```
═══════════════════════════════════
  ตารางแปลงอุณหภูมิ
═══════════════════════════════════
Celsius      Fahrenheit   Kelvin      
───────────────────────────────────
-40.0        -40.0        233.15      
0.0          32.0         273.15      
20.0         68.0         293.15      
37.0         98.6         310.15      
100.0        212.0        373.15      
═══════════════════════════════════
```

---

## 17. แบบฝึกหัด (Exercises)

### แบบฝึกหัดที่ 1: Hello Personal

เขียนโปรแกรมที่พิมพ์ข้อมูลส่วนตัวของคุณ:
- ชื่อ
- อายุ
- ภาษาโปรแกรมที่อยากเรียน
- เป้าหมายการเรียน Swift

**เฉลย:**

```swift
// exercise01.swift

let name = "สมชาย ใจดี"
let age = 25
let favoriteLanguage = "Swift"
let goal = "พัฒนาแอป iOS มืออาชีพ"

print("=== ข้อมูลส่วนตัว ===")
print("ชื่อ: \(name)")
print("อายุ: \(age) ปี")
print("ภาษาที่อยากเรียน: \(favoriteLanguage)")
print("เป้าหมาย: \(goal)")
print("=====================")
```

---

### แบบฝึกหัดที่ 2: Shape Calculator

เขียนโปรแกรมคำนวณพื้นที่และเส้นรอบวงของรูปทรงต่างๆ:
1. สี่เหลี่ยมผืนผ้า (กว้าง 5, ยาว 8)
2. วงกลม (รัศมี 7)
3. สามเหลี่ยม (ฐาน 6, สูง 4)

**เฉลย:**

```swift
// exercise02.swift
import Foundation

// สี่เหลี่ยมผืนผ้า
let width = 5.0
let length = 8.0
let rectArea = width * length
let rectPerimeter = 2 * (width + length)

// วงกลม
let radius = 7.0
let circleArea = Double.pi * radius * radius
let circlePerimeter = 2 * Double.pi * radius

// สามเหลี่ยม
let base = 6.0
let triangleHeight = 4.0
let triangleArea = 0.5 * base * triangleHeight

print("=== Shape Calculator ===\n")

print("🔲 สี่เหลี่ยมผืนผ้า (กว้าง=\(width), ยาว=\(length))")
print("   พื้นที่ = \(rectArea) ตร.หน่วย")
print("   เส้นรอบวง = \(rectPerimeter) หน่วย\n")

print("⭕ วงกลม (รัศมี=\(radius))")
print(String(format: "   พื้นที่ = %.4f ตร.หน่วย", circleArea))
print(String(format: "   เส้นรอบวง = %.4f หน่วย\n", circlePerimeter))

print("🔺 สามเหลี่ยม (ฐาน=\(base), สูง=\(triangleHeight))")
print("   พื้นที่ = \(triangleArea) ตร.หน่วย")
```

---

### แบบฝึกหัดที่ 3: FizzBuzz

เขียนโปรแกรม FizzBuzz คลาสสิก:
- ถ้าหารด้วย 3 ลงตัว พิมพ์ "Fizz"
- ถ้าหารด้วย 5 ลงตัว พิมพ์ "Buzz"
- ถ้าหารด้วยทั้ง 3 และ 5 พิมพ์ "FizzBuzz"
- นอกนั้นพิมพ์ตัวเลข

**เฉลย:**

```swift
// exercise03.swift

for number in 1...20 {
    if number % 15 == 0 {
        print("FizzBuzz")
    } else if number % 3 == 0 {
        print("Fizz")
    } else if number % 5 == 0 {
        print("Buzz")
    } else {
        print(number)
    }
}
```

**Output:**
```
1
2
Fizz
4
Buzz
Fizz
7
8
Fizz
Buzz
11
Fizz
13
14
FizzBuzz
16
17
Fizz
19
Buzz
```

---

### แบบฝึกหัดที่ 4: สร้าง Swift Package

ลองสร้าง Swift Package แรกของคุณ:
1. สร้างโฟลเดอร์ `HelloPackage`
2. รัน `swift package init --type executable`
3. แก้ไข `main.swift` ให้พิมพ์ข้อความเป็นภาษาไทย
4. Build และรัน

**เฉลย:**

```bash
mkdir HelloPackage
cd HelloPackage
swift package init --type executable
```

แก้ไข `Sources/HelloPackage/main.swift`:

```swift
// main.swift

import Foundation

print("╔══════════════════════════════╗")
print("║   ยินดีต้อนรับสู่ Swift!     ║")
print("╚══════════════════════════════╝")
print()

let swiftVersion = "5.9"
let year = Calendar.current.component(.year, from: Date())
print("Swift Version: \(swiftVersion)")
print("ปีปัจจุบัน: \(year)")
print()
print("คุณกำลังเริ่มต้นการเดินทางที่ยอดเยี่ยม!")
```

```bash
swift build
swift run
```

---

## 18. สรุปและก้าวต่อไป

### สรุปสิ่งที่เรียนในตอนนี้

✅ **Swift คือ** ภาษาโปรแกรมที่ปลอดภัย รวดเร็ว สร้างโดย Apple (2014)  
✅ **รองรับ** iOS, macOS, watchOS, tvOS, Linux, Windows  
✅ **ติดตั้ง** ผ่าน Xcode (macOS) หรือ swift.org (Linux)  
✅ **Swift Playgrounds** ช่วยทดสอบโค้ดแบบ Interactive  
✅ **Hello World** เขียนง่ายมาก แค่ `print("Hello, World!")`  
✅ **Swift REPL** สำหรับทดลองโค้ดทีละบรรทัด  
✅ **swiftc** ใช้ compile, `swift` ใช้รันตรง  
✅ **SPM** สำหรับจัดการ Dependencies  
✅ **Xcode** คือ IDE หลักสำหรับ Apple Platforms  

### สิ่งที่จะเรียนต่อไป

➡️ **[Part 02: Variables, Constants & Data Types](Part02_Variables_Constants_DataTypes.md)**
- ตัวแปรและค่าคงที่
- ชนิดข้อมูลทุกแบบ
- Type Inference
- Type Safety

---

## 📚 อ่านเพิ่มเติม

| แหล่งข้อมูล | URL |
|------------|-----|
| Swift.org (Official) | https://swift.org |
| Apple Swift Documentation | https://docs.swift.org |
| Swift Book (Free) | https://docs.swift.org/swift-book |
| Swift Forums | https://forums.swift.org |
| Swift GitHub | https://github.com/apple/swift |
| Hacking with Swift | https://www.hackingwithswift.com |
| Ray Wenderlich | https://www.kodeco.com |

---

*Part 01 จาก 100 | หลักสูตร Swift ฉบับสมบูรณ์*
