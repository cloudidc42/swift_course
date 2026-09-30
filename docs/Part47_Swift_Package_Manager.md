# Part 47: Swift Package Manager (SPM)

## บทนำ

Swift Package Manager (SPM) คือเครื่องมือจัดการ dependencies อย่างเป็นทางการของ Apple ที่ถูกสร้างขึ้นมาพร้อมกับภาษา Swift โดยถูกรวมเข้ากับ Swift toolchain ตั้งแต่ Swift 3.0 และได้รับการพัฒนาอย่างต่อเนื่องจนกลายเป็นเครื่องมือหลักในการจัดการ packages สำหรับ Swift projects ทั้งบน iOS, macOS, Linux และ Windows

---

## 1. Swift Package Manager คืออะไร?

### ความหมายและประวัติ

Swift Package Manager หรือ SPM คือระบบสำหรับจัดการ:
- **Dependencies** - ไลบรารีภายนอกที่ project ของคุณต้องการ
- **Build process** - ขั้นตอนการ compile และ link code
- **Package distribution** - การแจกจ่าย code ในรูปแบบ package

ก่อนที่จะมี SPM นักพัฒนา iOS/macOS ต้องพึ่งพาเครื่องมืออื่น เช่น CocoaPods หรือ Carthage แต่ SPM ถูกออกแบบมาให้เป็นส่วนหนึ่งของ Swift ecosystem โดยตรง

### ข้อดีของ SPM เทียบกับเครื่องมืออื่น

| คุณสมบัติ | SPM | CocoaPods | Carthage |
|-----------|-----|-----------|----------|
| Official support | ✅ Apple official | ❌ Third-party | ❌ Third-party |
| Xcode integration | ✅ Native | ⚠️ Requires setup | ⚠️ Manual |
| Linux support | ✅ Yes | ❌ No | ❌ No |
| Binary dependencies | ✅ XCFramework | ✅ Yes | ✅ Yes |
| Command line | ✅ Built-in | ✅ Separate tool | ✅ Separate tool |
| Privacy manifest | ✅ Supported | ⚠️ Limited | ❌ No |

### การติดตั้ง

SPM ถูกติดตั้งมาพร้อมกับ Swift toolchain อัตโนมัติ ตรวจสอบเวอร์ชันด้วยคำสั่ง:

```bash
swift --version
swift package --version
```

---

## 2. โครงสร้าง Package

### โครงสร้างไดเรกทอรีพื้นฐาน

```
MyPackage/
├── Package.swift          # Manifest file (จำเป็นต้องมี)
├── README.md
├── LICENSE
├── Sources/               # Source code
│   └── MyLibrary/
│       ├── MyLibrary.swift
│       └── Internal/
│           └── Helper.swift
├── Tests/                 # Test code
│   └── MyLibraryTests/
│       └── MyLibraryTests.swift
└── Package.resolved       # Resolved dependencies (auto-generated)
```

### โครงสร้างสำหรับ Executable Package

```
MyApp/
├── Package.swift
├── Sources/
│   ├── MyApp/             # Executable target
│   │   └── main.swift     # Entry point
│   └── MyLibrary/         # Library target ที่ MyApp ใช้
│       └── Library.swift
└── Tests/
    └── MyLibraryTests/
        └── Tests.swift
```

### การสร้าง Package ใหม่

```bash
# สร้าง library package
mkdir MyLibrary && cd MyLibrary
swift package init --type library

# สร้าง executable package
mkdir MyApp && cd MyApp
swift package init --type executable

# สร้าง empty package
mkdir MyPackage && cd MyPackage
swift package init --type empty
```

---

## 3. Package.swift Manifest

### รูปแบบพื้นฐาน

```swift
// Package.swift
// swift-tools-version: 5.9

import PackageDescription

let package = Package(
    name: "MyLibrary",
    platforms: [
        .macOS(.v13),
        .iOS(.v16),
        .tvOS(.v16),
        .watchOS(.v9),
        .visionOS(.v1)
    ],
    products: [
        .library(
            name: "MyLibrary",
            targets: ["MyLibrary"]
        )
    ],
    dependencies: [
        .package(
            url: "https://github.com/apple/swift-log.git",
            from: "1.5.0"
        )
    ],
    targets: [
        .target(
            name: "MyLibrary",
            dependencies: [
                .product(name: "Logging", package: "swift-log")
            ],
            path: "Sources/MyLibrary",
            swiftSettings: [
                .enableExperimentalFeature("StrictConcurrency")
            ]
        ),
        .testTarget(
            name: "MyLibraryTests",
            dependencies: ["MyLibrary"],
            path: "Tests/MyLibraryTests"
        )
    ]
)
```

### Swift Tools Version

บรรทัดแรกของ Package.swift ต้องระบุ swift-tools-version เสมอ:

```swift
// swift-tools-version: 5.9   // ใช้ฟีเจอร์ของ Swift 5.9
// swift-tools-version: 5.10  // ใช้ฟีเจอร์ของ Swift 5.10
// swift-tools-version: 6.0   // ใช้ฟีเจอร์ของ Swift 6.0
```

เวอร์ชันนี้กำหนดว่า Package.swift รองรับ syntax และ API อะไรบ้าง

### Platforms

```swift
platforms: [
    .macOS(.v12),           // macOS 12 Monterey
    .macOS(.v13),           // macOS 13 Ventura
    .macOS(.v14),           // macOS 14 Sonoma
    .iOS(.v15),
    .iOS(.v16),
    .iOS(.v17),
    .tvOS(.v15),
    .watchOS(.v8),
    .visionOS(.v1),         // Apple Vision Pro
    .linux                  // Linux (ไม่ต้องระบุเวอร์ชัน)
]
```

---

## 4. การสร้าง Library Package

### ตัวอย่าง: NetworkKit Library

```swift
// Package.swift
// swift-tools-version: 5.9

import PackageDescription

let package = Package(
    name: "NetworkKit",
    platforms: [
        .macOS(.v13),
        .iOS(.v16)
    ],
    products: [
        // Public library product
        .library(
            name: "NetworkKit",
            targets: ["NetworkKit"]
        ),
        // Optional: Static library
        .library(
            name: "NetworkKitStatic",
            type: .static,
            targets: ["NetworkKit"]
        ),
        // Optional: Dynamic library
        .library(
            name: "NetworkKitDynamic",
            type: .dynamic,
            targets: ["NetworkKit"]
        )
    ],
    dependencies: [
        .package(
            url: "https://github.com/apple/swift-log.git",
            from: "1.5.0"
        ),
        .package(
            url: "https://github.com/apple/swift-collections.git",
            from: "1.0.0"
        )
    ],
    targets: [
        .target(
            name: "NetworkKit",
            dependencies: [
                .product(name: "Logging", package: "swift-log"),
                .product(name: "Collections", package: "swift-collections")
            ],
            resources: [
                .copy("Resources/certificates.pem"),
                .process("Resources/Localizable.strings")
            ]
        ),
        .testTarget(
            name: "NetworkKitTests",
            dependencies: ["NetworkKit"]
        )
    ]
)
```

### Source Code ของ Library

```swift
// Sources/NetworkKit/NetworkKit.swift

/// NetworkKit - ไลบรารีสำหรับจัดการ HTTP requests
public struct NetworkKit {
    
    public static let version = "1.0.0"
    
    private let session: URLSession
    private let baseURL: URL
    
    public init(baseURL: URL, session: URLSession = .shared) {
        self.baseURL = baseURL
        self.session = session
    }
    
    /// ส่ง GET request
    public func get<T: Decodable>(
        path: String,
        responseType: T.Type
    ) async throws -> T {
        let url = baseURL.appendingPathComponent(path)
        let (data, response) = try await session.data(from: url)
        
        guard let httpResponse = response as? HTTPURLResponse,
              (200...299).contains(httpResponse.statusCode) else {
            throw NetworkError.invalidResponse
        }
        
        return try JSONDecoder().decode(T.self, from: data)
    }
}

// Sources/NetworkKit/NetworkError.swift

/// ข้อผิดพลาดที่อาจเกิดขึ้นในการทำ network requests
public enum NetworkError: LocalizedError {
    case invalidURL
    case invalidResponse
    case decodingError(Error)
    case serverError(Int)
    case noInternetConnection
    
    public var errorDescription: String? {
        switch self {
        case .invalidURL:
            return "URL ไม่ถูกต้อง"
        case .invalidResponse:
            return "Response ไม่ถูกต้อง"
        case .decodingError(let error):
            return "ไม่สามารถ decode ข้อมูลได้: \(error.localizedDescription)"
        case .serverError(let code):
            return "Server error: \(code)"
        case .noInternetConnection:
            return "ไม่มีการเชื่อมต่ออินเทอร์เน็ต"
        }
    }
}
```

### Build และ Test

```bash
# Build package
swift build

# Build แบบ release
swift build -c release

# Run tests
swift test

# Run tests แบบ verbose
swift test --verbose

# Run test เฉพาะ test suite
swift test --filter NetworkKitTests

# Run test เฉพาะ test case
swift test --filter NetworkKitTests/testGETRequest
```

---

## 5. การสร้าง Executable Package

### ตัวอย่าง: CLI Tool

```swift
// Package.swift
// swift-tools-version: 5.9

import PackageDescription

let package = Package(
    name: "SwiftCLI",
    platforms: [
        .macOS(.v13)
    ],
    products: [
        .executable(
            name: "swift-cli",
            targets: ["SwiftCLI"]
        )
    ],
    dependencies: [
        .package(
            url: "https://github.com/apple/swift-argument-parser.git",
            from: "1.3.0"
        )
    ],
    targets: [
        .executableTarget(
            name: "SwiftCLI",
            dependencies: [
                .product(
                    name: "ArgumentParser",
                    package: "swift-argument-parser"
                )
            ]
        )
    ]
)
```

```swift
// Sources/SwiftCLI/main.swift

import ArgumentParser
import Foundation

@main
struct SwiftCLI: ParsableCommand {
    
    static var configuration = CommandConfiguration(
        commandName: "swift-cli",
        abstract: "เครื่องมือ CLI สำหรับจัดการไฟล์",
        subcommands: [
            ListCommand.self,
            CreateCommand.self,
            DeleteCommand.self
        ]
    )
}

// Sources/SwiftCLI/Commands/ListCommand.swift

struct ListCommand: ParsableCommand {
    
    static var configuration = CommandConfiguration(
        commandName: "list",
        abstract: "แสดงรายการไฟล์"
    )
    
    @Argument(help: "Path ที่ต้องการแสดง")
    var path: String = "."
    
    @Flag(name: .shortAndLong, help: "แสดงรายละเอียด")
    var verbose: Bool = false
    
    mutating func run() throws {
        let fileManager = FileManager.default
        let items = try fileManager.contentsOfDirectory(atPath: path)
        
        for item in items.sorted() {
            if verbose {
                let fullPath = (path as NSString).appendingPathComponent(item)
                let attributes = try fileManager.attributesOfItem(atPath: fullPath)
                let size = attributes[.size] as? Int ?? 0
                print("\(item) (\(size) bytes)")
            } else {
                print(item)
            }
        }
    }
}
```

### การ Run Executable

```bash
# Run executable โดยตรง
swift run swift-cli list

# Run พร้อม arguments
swift run swift-cli list /Users/john --verbose

# Build executable
swift build -c release

# ตรวจสอบ executable ที่ build แล้ว
ls .build/release/swift-cli

# Install executable
cp .build/release/swift-cli /usr/local/bin/
```

---

## 6. การเพิ่ม Dependencies

### รูปแบบการระบุ Dependencies

```swift
dependencies: [
    // จาก version หนึ่งขึ้นไป (up to next major)
    .package(
        url: "https://github.com/example/package.git",
        from: "1.2.3"
    ),
    
    // ระบุ version range
    .package(
        url: "https://github.com/example/package.git",
        "1.2.0"..<"2.0.0"
    ),
    
    // ระบุ closed range
    .package(
        url: "https://github.com/example/package.git",
        "1.2.0"..."1.5.0"
    ),
    
    // ระบุ branch (สำหรับ development)
    .package(
        url: "https://github.com/example/package.git",
        branch: "main"
    ),
    
    // ระบุ commit hash
    .package(
        url: "https://github.com/example/package.git",
        revision: "abc1234"
    ),
    
    // ระบุ exact version
    .package(
        url: "https://github.com/example/package.git",
        exact: "1.2.3"
    ),
    
    // Local package
    .package(
        path: "../MyLocalPackage"
    )
]
```

### การใช้งาน Dependencies ใน Targets

```swift
targets: [
    .target(
        name: "MyApp",
        dependencies: [
            // ใช้ product ทั้งหมดจาก package
            "SomePackage",
            
            // ระบุ product เฉพาะจาก package
            .product(name: "SpecificProduct", package: "some-package"),
            
            // Optional dependency
            .product(
                name: "OptionalProduct",
                package: "optional-package",
                condition: .when(platforms: [.macOS])
            )
        ]
    )
]
```

### ตัวอย่าง Package.swift ที่มีหลาย Dependencies

```swift
// Package.swift
// swift-tools-version: 5.9

import PackageDescription

let package = Package(
    name: "ProductionApp",
    platforms: [
        .iOS(.v16),
        .macOS(.v13)
    ],
    products: [
        .library(name: "ProductionApp", targets: ["ProductionApp"])
    ],
    dependencies: [
        // Apple official packages
        .package(
            url: "https://github.com/apple/swift-log.git",
            from: "1.5.0"
        ),
        .package(
            url: "https://github.com/apple/swift-algorithms.git",
            from: "1.2.0"
        ),
        .package(
            url: "https://github.com/apple/swift-collections.git",
            from: "1.1.0"
        ),
        .package(
            url: "https://github.com/apple/swift-crypto.git",
            from: "3.0.0"
        ),
        .package(
            url: "https://github.com/apple/swift-argument-parser.git",
            from: "1.3.0"
        ),
        
        // Third-party packages
        .package(
            url: "https://github.com/Alamofire/Alamofire.git",
            from: "5.8.0"
        ),
        .package(
            url: "https://github.com/onevcat/Kingfisher.git",
            from: "7.0.0"
        )
    ],
    targets: [
        .target(
            name: "ProductionApp",
            dependencies: [
                .product(name: "Logging", package: "swift-log"),
                .product(name: "Algorithms", package: "swift-algorithms"),
                .product(name: "Collections", package: "swift-collections"),
                .product(name: "Crypto", package: "swift-crypto"),
                .product(name: "Alamofire", package: "Alamofire"),
                .product(name: "Kingfisher", package: "Kingfisher")
            ]
        )
    ]
)
```

---

## 7. Semantic Versioning

### หลักการของ Semantic Versioning (SemVer)

Semantic Versioning ใช้รูปแบบ `MAJOR.MINOR.PATCH` เช่น `1.2.3`:

- **MAJOR** - เปลี่ยนแปลงที่ทำให้ไม่ compatible กับเวอร์ชันก่อน (Breaking changes)
- **MINOR** - เพิ่มฟีเจอร์ใหม่ที่ยังคง backward compatible
- **PATCH** - แก้บั๊กที่ยังคง backward compatible

```
1.0.0  → เวอร์ชันแรกที่ stable
1.0.1  → แก้บั๊ก
1.1.0  → เพิ่มฟีเจอร์ใหม่
2.0.0  → Breaking change
```

### Pre-release Versions

```
1.0.0-alpha      → Alpha version (ทดสอบเบื้องต้น)
1.0.0-alpha.1    → Alpha version 1
1.0.0-beta       → Beta version
1.0.0-beta.2     → Beta version 2
1.0.0-rc.1       → Release candidate 1
1.0.0            → Stable release
```

### การ Tag Version ใน Git

```bash
# สร้าง tag
git tag 1.0.0
git tag -a 1.0.0 -m "Release version 1.0.0"

# Push tag ไปยัง remote
git push origin 1.0.0
git push origin --tags

# ดู tags ทั้งหมด
git tag -l

# ลบ tag
git tag -d 1.0.0
git push origin --delete 1.0.0
```

### Version Constraints ใน Package.swift

```swift
// .upToNextMajor(from: "1.2.3") - รองรับ 1.2.3 ถึง < 2.0.0
.package(url: "...", from: "1.2.3")
// เทียบเท่ากับ
.package(url: "...", .upToNextMajor(from: "1.2.3"))

// .upToNextMinor(from: "1.2.3") - รองรับ 1.2.3 ถึง < 1.3.0
.package(url: "...", .upToNextMinor(from: "1.2.3"))

// .exact("1.2.3") - ใช้เฉพาะ 1.2.3
.package(url: "...", exact: "1.2.3")

// range - รองรับ 1.2.0 ถึง < 2.0.0
.package(url: "...", "1.2.0"..<"2.0.0")

// closedRange - รองรับ 1.2.0 ถึง 1.5.0 รวม
.package(url: "...", "1.2.0"..."1.5.0")
```

---

## 8. Targets และ Products

### ประเภทของ Targets

```swift
targets: [
    // Regular target (library)
    .target(
        name: "MyLibrary",
        dependencies: [],
        path: "Sources/MyLibrary",
        exclude: ["README.md"],
        sources: ["File1.swift", "File2.swift"],
        resources: [
            .copy("Resources/data.json"),
            .process("Resources/Images")
        ],
        publicHeadersPath: "include",  // สำหรับ C/C++/Objective-C
        cSettings: [
            .headerSearchPath("include"),
            .define("DEBUG", to: "1", .when(configuration: .debug))
        ],
        swiftSettings: [
            .unsafeFlags(["-Xfrontend", "-disable-reflection-metadata"]),
            .define("ENABLE_FEATURE_X")
        ],
        linkerSettings: [
            .linkedLibrary("z"),
            .linkedFramework("CoreData")
        ]
    ),
    
    // Executable target
    .executableTarget(
        name: "MyApp",
        dependencies: ["MyLibrary"]
    ),
    
    // Test target
    .testTarget(
        name: "MyLibraryTests",
        dependencies: ["MyLibrary"],
        resources: [
            .copy("Fixtures/test_data.json")
        ]
    ),
    
    // System library target (wrapping C library)
    .systemLibrary(
        name: "COpenSSL",
        path: "Sources/COpenSSL",
        pkgConfig: "openssl",
        providers: [
            .brew(["openssl"]),
            .apt(["libssl-dev"])
        ]
    ),
    
    // Binary target (XCFramework)
    .binaryTarget(
        name: "SomeSDK",
        url: "https://example.com/SomeSDK.xcframework.zip",
        checksum: "abc123..."
    ),
    
    // Local binary target
    .binaryTarget(
        name: "LocalSDK",
        path: "Frameworks/LocalSDK.xcframework"
    )
]
```

### ประเภทของ Products

```swift
products: [
    // Dynamic/Static library (Swift decides)
    .library(
        name: "MyLib",
        targets: ["MyLib"]
    ),
    
    // Explicitly static library
    .library(
        name: "MyLibStatic",
        type: .static,
        targets: ["MyLib"]
    ),
    
    // Explicitly dynamic library
    .library(
        name: "MyLibDynamic",
        type: .dynamic,
        targets: ["MyLib"]
    ),
    
    // Executable product
    .executable(
        name: "my-tool",
        targets: ["MyTool"]
    ),
    
    // Plugin product
    .plugin(
        name: "MyPlugin",
        targets: ["MyPlugin"]
    )
]
```

### Multi-target Package

```swift
// Package.swift - ตัวอย่าง package ที่มีหลาย targets
// swift-tools-version: 5.9

import PackageDescription

let package = Package(
    name: "FullStackKit",
    platforms: [
        .macOS(.v13),
        .iOS(.v16)
    ],
    products: [
        .library(name: "NetworkingLayer", targets: ["NetworkingLayer"]),
        .library(name: "DataLayer", targets: ["DataLayer"]),
        .library(name: "UIComponents", targets: ["UIComponents"]),
        .library(name: "FullStackKit", targets: [
            "NetworkingLayer",
            "DataLayer",
            "UIComponents"
        ])
    ],
    dependencies: [
        .package(url: "https://github.com/apple/swift-log.git", from: "1.5.0")
    ],
    targets: [
        // Core networking
        .target(
            name: "NetworkingLayer",
            dependencies: [
                .product(name: "Logging", package: "swift-log")
            ]
        ),
        
        // Data management (depends on Networking)
        .target(
            name: "DataLayer",
            dependencies: ["NetworkingLayer"]
        ),
        
        // UI components (depends on Data)
        .target(
            name: "UIComponents",
            dependencies: ["DataLayer"]
        ),
        
        // Tests
        .testTarget(
            name: "NetworkingLayerTests",
            dependencies: ["NetworkingLayer"]
        ),
        .testTarget(
            name: "DataLayerTests",
            dependencies: ["DataLayer"]
        )
    ]
)
```

---

## 9. Binary Targets (XCFramework)

### XCFramework คืออะไร?

XCFramework คือ format สำหรับ binary framework ที่รองรับหลาย platform และ architecture ในไฟล์เดียว เหมาะสำหรับกรณีที่:
- ต้องการแจกจ่าย closed-source library
- มี pre-built framework จาก third-party
- ต้องการ optimize สำหรับ production builds

### การสร้าง XCFramework

```bash
# สร้าง archive สำหรับแต่ละ platform
xcodebuild archive \
    -scheme MyFramework \
    -destination "generic/platform=iOS" \
    -archivePath ./archives/ios \
    SKIP_INSTALL=NO \
    BUILD_LIBRARY_FOR_DISTRIBUTION=YES

xcodebuild archive \
    -scheme MyFramework \
    -destination "generic/platform=iOS Simulator" \
    -archivePath ./archives/ios-simulator \
    SKIP_INSTALL=NO \
    BUILD_LIBRARY_FOR_DISTRIBUTION=YES

xcodebuild archive \
    -scheme MyFramework \
    -destination "generic/platform=macOS" \
    -archivePath ./archives/macos \
    SKIP_INSTALL=NO \
    BUILD_LIBRARY_FOR_DISTRIBUTION=YES

# รวมเป็น XCFramework
xcodebuild -create-xcframework \
    -framework ./archives/ios.xcarchive/Products/Library/Frameworks/MyFramework.framework \
    -framework ./archives/ios-simulator.xcarchive/Products/Library/Frameworks/MyFramework.framework \
    -framework ./archives/macos.xcarchive/Products/Library/Frameworks/MyFramework.framework \
    -output ./MyFramework.xcframework
```

### การ Compress และ Hash

```bash
# Compress XCFramework
zip -r MyFramework.xcframework.zip MyFramework.xcframework

# คำนวณ checksum
swift package compute-checksum MyFramework.xcframework.zip
# Output: e3b0c44298fc1c149afb...
```

### การใช้ Binary Target ใน Package.swift

```swift
// Package.swift
// swift-tools-version: 5.9

import PackageDescription

let package = Package(
    name: "MyApp",
    platforms: [
        .iOS(.v16),
        .macOS(.v13)
    ],
    products: [
        .library(name: "MyApp", targets: ["MyApp"])
    ],
    targets: [
        .target(
            name: "MyApp",
            dependencies: [
                "MyFramework",
                "AnotherSDK"
            ]
        ),
        
        // Remote binary target
        .binaryTarget(
            name: "MyFramework",
            url: "https://github.com/myorg/myframework/releases/download/1.0.0/MyFramework.xcframework.zip",
            checksum: "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
        ),
        
        // Local binary target
        .binaryTarget(
            name: "AnotherSDK",
            path: "Frameworks/AnotherSDK.xcframework"
        )
    ]
)
```

---

## 10. Plugins

### ประเภทของ Plugins

SPM รองรับ plugins สองประเภทหลัก:

1. **Command Plugins** - รันด้วย `swift package <plugin-name>` หรือใน Xcode
2. **Build Tool Plugins** - รันอัตโนมัติระหว่าง build process

### Command Plugin

```swift
// Package.swift
// swift-tools-version: 5.9

import PackageDescription

let package = Package(
    name: "MyPackageWithPlugin",
    products: [
        .library(name: "MyLibrary", targets: ["MyLibrary"]),
        .plugin(name: "GenerateCode", targets: ["GenerateCodePlugin"])
    ],
    targets: [
        .target(name: "MyLibrary"),
        
        // Command plugin
        .plugin(
            name: "GenerateCodePlugin",
            capability: .command(
                intent: .custom(
                    verb: "generate-code",
                    description: "สร้าง code จาก templates"
                ),
                permissions: [
                    .writeToPackageDirectory(reason: "เขียนไฟล์ที่ generate แล้ว")
                ]
            )
        )
    ]
)
```

```swift
// Sources/GenerateCodePlugin/GenerateCodePlugin.swift

import PackagePlugin
import Foundation

@main
struct GenerateCodePlugin: CommandPlugin {
    
    func performCommand(
        context: PluginContext,
        arguments: [String]
    ) async throws {
        
        // หา target ที่ต้องการ generate code
        let target = try context.package.targets(named: ["MyLibrary"]).first!
        
        // Path สำหรับ output
        let outputDir = context.package.directory.appending("Sources/MyLibrary/Generated")
        
        // สร้าง code
        let generatedCode = """
        // Auto-generated code - Do not edit!
        // Generated at: \\(Date())
        
        public enum GeneratedConstants {
            public static let version = "\\(context.package.displayName)"
        }
        """
        
        // เขียนไฟล์
        let outputFile = outputDir.appending("Constants.swift")
        try FileManager.default.createDirectory(
            atPath: outputDir.string,
            withIntermediateDirectories: true
        )
        try generatedCode.write(
            toFile: outputFile.string,
            atomically: true,
            encoding: .utf8
        )
        
        print("Generated code successfully!")
    }
}
```

### Build Tool Plugin

```swift
// Package.swift - Build Tool Plugin
// swift-tools-version: 5.9

import PackageDescription

let package = Package(
    name: "SwiftGenPlugin",
    platforms: [
        .macOS(.v13)
    ],
    products: [
        .plugin(name: "SwiftGenPlugin", targets: ["SwiftGenPlugin"])
    ],
    dependencies: [
        .package(
            url: "https://github.com/SwiftGen/SwiftGen.git",
            from: "6.6.0"
        )
    ],
    targets: [
        // Build tool plugin
        .plugin(
            name: "SwiftGenPlugin",
            capability: .buildTool(),
            dependencies: [
                .product(name: "swiftgen", package: "SwiftGen")
            ]
        )
    ]
)
```

```swift
// Sources/SwiftGenPlugin/SwiftGenPlugin.swift

import PackagePlugin

@main
struct SwiftGenPlugin: BuildToolPlugin {
    
    func createBuildCommands(
        context: PluginContext,
        target: Target
    ) async throws -> [Command] {
        
        guard let target = target as? SourceModuleTarget else {
            return []
        }
        
        // หาไฟล์ configuration
        let configFiles = target.sourceFiles(withSuffix: "swiftgen.yml")
        
        var commands: [Command] = []
        
        for configFile in configFiles {
            let outputDir = context.pluginWorkDirectory
            
            commands.append(
                .buildCommand(
                    displayName: "Running SwiftGen",
                    executable: try context.tool(named: "swiftgen").path,
                    arguments: [
                        "config",
                        "run",
                        "--config",
                        configFile.path.string,
                        "--output",
                        outputDir.string
                    ],
                    inputFiles: [configFile.path],
                    outputFiles: [outputDir.appending("Generated.swift")]
                )
            )
        }
        
        return commands
    }
}
```

### การรัน Plugin

```bash
# รัน command plugin
swift package generate-code

# รัน plugin สำหรับ target เฉพาะ
swift package --allow-writing-to-package-directory generate-code --target MyLibrary

# ดู plugins ที่มี
swift package plugin --list
```

---

## 11. การ Publish Package

### ขั้นตอนการ Publish Package

```bash
# 1. สร้าง repository บน GitHub

# 2. Initialize git และ push code
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/yourusername/MyPackage.git
git push -u origin main

# 3. สร้าง tag สำหรับ version แรก
git tag 1.0.0
git push origin 1.0.0
```

### Checklist ก่อน Publish

```markdown
## Pre-publish Checklist

### Code Quality
- [ ] Tests ผ่านทั้งหมด (`swift test`)
- [ ] Build สำเร็จ (`swift build`)
- [ ] Code documentation ครบถ้วน
- [ ] Example code ทำงานได้

### Package.swift
- [ ] swift-tools-version อัปเดตเป็นเวอร์ชันล่าสุด
- [ ] Platform requirements ถูกต้อง
- [ ] Dependencies อัปเดตเป็น stable versions
- [ ] Products และ targets กำหนดถูกต้อง

### Documentation
- [ ] README.md อธิบายวิธีใช้งาน
- [ ] CHANGELOG.md อัปเดต
- [ ] LICENSE ถูกต้อง
- [ ] API documentation ครบถ้วน

### Git
- [ ] .gitignore มีรายการที่จำเป็น
- [ ] ไม่มี sensitive data ใน repository
```

### Package.swift สำหรับ Published Package

```swift
// Package.swift
// swift-tools-version: 5.9

import PackageDescription

let package = Package(
    name: "AwesomeSwiftPackage",
    
    // รองรับ platforms อะไรบ้าง
    platforms: [
        .macOS(.v13),
        .iOS(.v16),
        .tvOS(.v16),
        .watchOS(.v9),
        .visionOS(.v1)
    ],
    
    // สิ่งที่ user จะ import
    products: [
        .library(
            name: "AwesomeSwiftPackage",
            targets: ["AwesomeSwiftPackage"]
        )
    ],
    
    // ไม่มี external dependencies (ดีกว่าสำหรับ reusable libraries)
    dependencies: [],
    
    targets: [
        .target(
            name: "AwesomeSwiftPackage",
            path: "Sources"
        ),
        .testTarget(
            name: "AwesomeSwiftPackageTests",
            dependencies: ["AwesomeSwiftPackage"],
            path: "Tests"
        )
    ],
    
    // Swift language version
    swiftLanguageVersions: [.v5]
)
```

---

## 12. Local Package Development

### การพัฒนา Package แบบ Local

ขณะพัฒนา package ใหม่ คุณสามารถทดสอบโดยใช้ local path แทน URL:

```swift
// Package.swift ของ project ที่ใช้ local package
dependencies: [
    // Local path dependency
    .package(path: "../MyLocalLibrary"),
    
    // ถ้า library อยู่ใน sibling directory
    .package(path: "../../SharedLibraries/NetworkKit")
]
```

### Workspace สำหรับ Multi-package Development

```
MyWorkspace/
├── MyApp/
│   ├── Package.swift
│   └── Sources/
├── MyLibrary/
│   ├── Package.swift
│   └── Sources/
└── SharedCore/
    ├── Package.swift
    └── Sources/
```

```swift
// MyApp/Package.swift
// swift-tools-version: 5.9

import PackageDescription

let package = Package(
    name: "MyApp",
    dependencies: [
        // ใช้ local version ระหว่าง development
        .package(path: "../MyLibrary"),
        .package(path: "../SharedCore")
    ],
    targets: [
        .executableTarget(
            name: "MyApp",
            dependencies: [
                .product(name: "MyLibrary", package: "MyLibrary"),
                .product(name: "SharedCore", package: "SharedCore")
            ]
        )
    ]
)
```

### Local Overrides

สามารถ override dependency ด้วย local version ชั่วคราว:

```bash
# แก้ไข Package.resolved โดยตรง (ไม่แนะนำ)

# หรือใช้ local override ใน Xcode:
# File > Add Package Dependencies > Add Local...

# หรือแก้ไข Package.swift ชั่วคราว:
.package(path: "../MyOverrideLibrary")
```

---

## 13. การแก้ไข Dependencies (Resolving)

### คำสั่ง Resolution

```bash
# Resolve dependencies
swift package resolve

# Update dependencies เป็น version ล่าสุด
swift package update

# Update เฉพาะ package หนึ่ง
swift package update swift-log

# ดู dependency graph
swift package show-dependencies

# ดู dependency graph แบบ DOT format
swift package show-dependencies --format dot

# แสดง resolved versions
cat Package.resolved
```

### วิธีแก้ปัญหา Dependency Conflicts

```bash
# ลบ Package.resolved และ resolve ใหม่
rm Package.resolved
swift package resolve

# ล้าง cache ทั้งหมด
swift package clean
rm -rf .build
swift package resolve

# ดู version ที่ conflict
swift package show-dependencies

# Force update
swift package update --allow-writing-to-package-directory
```

---

## 14. Package.resolved

### โครงสร้างของ Package.resolved

```json
{
  "originHash" : "abc123...",
  "pins" : [
    {
      "identity" : "swift-log",
      "kind" : "remoteSourceControl",
      "location" : "https://github.com/apple/swift-log.git",
      "state" : {
        "revision" : "32e8d724467f8fe623624570367e3d50c5638e46",
        "version" : "1.5.3"
      }
    },
    {
      "identity" : "swift-argument-parser",
      "kind" : "remoteSourceControl",
      "location" : "https://github.com/apple/swift-argument-parser.git",
      "state" : {
        "revision" : "0fbc8848e389af3bb55c182bc19ca9d5b2e5cf7e",
        "version" : "1.3.0"
      }
    }
  ],
  "version" : 3
}
```

### การจัดการ Package.resolved

```bash
# ควร commit Package.resolved เข้า git เสมอสำหรับ executable/app
git add Package.resolved

# สำหรับ library package อาจไม่ต้อง commit
# เพราะ users จะ resolve เอง
echo "Package.resolved" >> .gitignore
```

### ความสำคัญของ Package.resolved

Package.resolved ช่วยให้:
1. **Reproducible builds** - ทุกคนใน team ใช้ version เดียวกัน
2. **Security** - ป้องกัน dependency hijacking
3. **Deterministic** - build ผลลัพธ์เหมือนกันทุกครั้ง

---

## 15. Workspace

### Xcode Workspace กับ SPM

```
MyProject.xcworkspace/
├── contents.xcworkspacedata
└── xcuserdata/

MyProject/
├── Package.swift
└── Sources/

MyFramework/
├── Package.swift
└── Sources/
```

```xml
<!-- contents.xcworkspacedata -->
<?xml version="1.0" encoding="UTF-8"?>
<Workspace version = "1.0">
   <FileRef
      location = "group:MyProject">
   </FileRef>
   <FileRef
      location = "group:MyFramework">
   </FileRef>
</Workspace>
```

### SPM Workspace (Swift 5.6+)

```swift
// Package.swift - workspace configuration
// swift-tools-version: 5.9

// ไม่มี workspace concept แบบ explicit ใน SPM
// แต่สามารถใช้ local dependencies แทน
```

---

## 16. การเพิ่ม SPM Packages ใน Xcode

### ผ่าน Xcode UI

1. เปิด Xcode project
2. เลือก **File** > **Add Package Dependencies...**
3. ใส่ URL ของ package repository
4. เลือก version requirement
5. คลิก **Add Package**
6. เลือก products ที่ต้องการเพิ่มใน target

### ผ่าน Package.swift ใน Xcode Project

```swift
// ใน Xcode project คุณสามารถเพิ่ม Package.swift ได้เช่นกัน
// โดยการสร้างไฟล์ Package.swift ที่ root ของ project

// แต่ส่วนใหญ่ Xcode จัดการ via .xcodeproj file
```

### การแก้ไขใน Xcode

```
Project Navigator > Package Dependencies tab
- เห็น packages ทั้งหมดที่ใช้
- Double-click เพื่อแก้ไข version
- Right-click เพื่อลบ package
```

### Pinning Version ใน Xcode

```
File > Packages > Reset Package Caches
File > Packages > Resolve Package Versions
File > Packages > Update to Latest Package Versions
```

---

## 17. การย้ายจาก CocoaPods

### เปรียบเทียบ CocoaPods กับ SPM

| CocoaPods | SPM |
|-----------|-----|
| `Podfile` | `Package.swift` |
| `pod install` | `swift package resolve` |
| `Podfile.lock` | `Package.resolved` |
| `pod update` | `swift package update` |
| `.xcworkspace` | Direct Xcode integration |
| `pod 'Alamofire', '~> 5.0'` | `.package(url: "...", from: "5.0.0")` |

### ขั้นตอนการ Migrate

```bash
# 1. ตรวจสอบ packages ที่ใช้อยู่ใน Podfile
cat Podfile

# 2. ตรวจสอบว่า packages รองรับ SPM หรือไม่
# ดูที่ GitHub repository > ตรวจสอบว่ามี Package.swift หรือไม่

# 3. ลบ CocoaPods
pod deintegrate
pod cache clean --all
rm -rf Pods/ Podfile.lock

# 4. เปิด .xcodeproj แทน .xcworkspace
```

```swift
// Podfile เดิม
// platform :ios, '16.0'
// pod 'Alamofire', '~> 5.8'
// pod 'Kingfisher', '~> 7.0'
// pod 'SnapKit', '~> 5.6'

// เปลี่ยนเป็น Package.swift
// swift-tools-version: 5.9

import PackageDescription

let package = Package(
    name: "MyApp",
    platforms: [.iOS(.v16)],
    dependencies: [
        .package(
            url: "https://github.com/Alamofire/Alamofire.git",
            from: "5.8.0"
        ),
        .package(
            url: "https://github.com/onevcat/Kingfisher.git",
            from: "7.0.0"
        ),
        .package(
            url: "https://github.com/SnapKit/SnapKit.git",
            from: "5.6.0"
        )
    ],
    targets: [
        .target(
            name: "MyApp",
            dependencies: [
                "Alamofire",
                "Kingfisher",
                "SnapKit"
            ]
        )
    ]
)
```

### ปัญหาที่อาจพบ

```swift
// 1. ถ้า package ยังไม่รองรับ SPM
// ต้องหา alternative หรือ fork แล้ว add SPM support

// 2. Objective-C headers
// บาง packages ที่เขียนด้วย ObjC ต้องตั้งค่า publicHeadersPath

// 3. Resources
// ต้องระบุ resources ใน Package.swift
.target(
    name: "MyLibrary",
    resources: [
        .process("Resources/")
    ]
)

// 4. Build settings
// บาง build settings ใน Podfile ต้องย้ายไปที่ project settings
```

---

## 18. การย้ายจาก Carthage

### เปรียบเทียบ Carthage กับ SPM

| Carthage | SPM |
|----------|-----|
| `Cartfile` | `Package.swift` |
| `carthage update` | `swift package resolve` |
| `Cartfile.resolved` | `Package.resolved` |
| ต้องเพิ่ม framework manually | Automatic |
| Pre-built binaries | Source compilation |

### ขั้นตอนการ Migrate

```bash
# 1. ดู Cartfile
cat Cartfile
# github "Alamofire/Alamofire" ~> 5.8
# github "onevcat/Kingfisher" ~> 7.0

# 2. ลบ Carthage artifacts
rm -rf Carthage/
rm Cartfile.resolved

# 3. ลบ frameworks ที่เพิ่มใน Xcode manually
# Xcode > Project > General > Frameworks, Libraries, and Embedded Content
# ลบ frameworks ที่ใช้จาก Carthage

# 4. เพิ่ม SPM dependencies ใน Xcode
```

---

## 19. Popular Swift Packages

### Networking

```swift
// Alamofire - HTTP networking library
.package(url: "https://github.com/Alamofire/Alamofire.git", from: "5.8.0")

// Moya - Network abstraction layer
.package(url: "https://github.com/Moya/Moya.git", from: "15.0.0")
```

### Image Loading

```swift
// Kingfisher - Image loading and caching
.package(url: "https://github.com/onevcat/Kingfisher.git", from: "7.0.0")

// Nuke - Image loading system
.package(url: "https://github.com/kean/Nuke.git", from: "12.0.0")
```

### UI Components

```swift
// SnapKit - Auto Layout DSL
.package(url: "https://github.com/SnapKit/SnapKit.git", from: "5.6.0")

// Then - Syntactic sugar
.package(url: "https://github.com/devxoul/Then.git", from: "3.0.0")
```

### Data & Storage

```swift
// GRDB - SQLite toolkit
.package(url: "https://github.com/groue/GRDB.swift.git", from: "6.0.0")

// Realm - Mobile database
.package(url: "https://github.com/realm/realm-swift.git", from: "10.0.0")

// KeychainAccess - Keychain wrapper
.package(url: "https://github.com/kishikawakatsumi/KeychainAccess.git", from: "4.2.0")
```

### Testing

```swift
// Quick/Nimble - BDD testing
.package(url: "https://github.com/Quick/Quick.git", from: "7.0.0"),
.package(url: "https://github.com/Quick/Nimble.git", from: "13.0.0")

// OHHTTPStubs - HTTP stubbing
.package(url: "https://github.com/AliSoftware/OHHTTPStubs.git", from: "9.0.0")
```

### Apple Official Packages

```swift
// swift-log
.package(url: "https://github.com/apple/swift-log.git", from: "1.5.0")

// swift-algorithms
.package(url: "https://github.com/apple/swift-algorithms.git", from: "1.2.0")

// swift-collections
.package(url: "https://github.com/apple/swift-collections.git", from: "1.1.0")

// swift-crypto
.package(url: "https://github.com/apple/swift-crypto.git", from: "3.0.0")

// swift-argument-parser
.package(url: "https://github.com/apple/swift-argument-parser.git", from: "1.3.0")

// swift-numerics
.package(url: "https://github.com/apple/swift-numerics.git", from: "1.0.0")

// swift-async-algorithms
.package(url: "https://github.com/apple/swift-async-algorithms.git", from: "1.0.0")
```

---

## 20. การสร้าง Reusable Library

### ออกแบบ API ที่ดี

```swift
// Sources/MathKit/MathKit.swift

/// MathKit - ไลบรารีสำหรับการคำนวณทางคณิตศาสตร์
///
/// ## Topics
///
/// ### Arithmetic
/// - ``add(_:_:)``
/// - ``subtract(_:_:)``
/// - ``multiply(_:_:)``
/// - ``divide(_:_:)``
///
/// ### Statistics
/// - ``mean(_:)``
/// - ``median(_:)``
/// - ``standardDeviation(_:)``
public struct MathKit {
    
    /// บวกตัวเลขสองตัว
    /// - Parameters:
    ///   - lhs: ตัวเลขตัวแรก
    ///   - rhs: ตัวเลขตัวที่สอง
    /// - Returns: ผลรวม
    public static func add<T: Numeric>(_ lhs: T, _ rhs: T) -> T {
        lhs + rhs
    }
    
    /// คำนวณค่าเฉลี่ย
    /// - Parameter numbers: ชุดตัวเลข
    /// - Returns: ค่าเฉลี่ย หรือ nil ถ้า array ว่าง
    public static func mean(_ numbers: [Double]) -> Double? {
        guard !numbers.isEmpty else { return nil }
        return numbers.reduce(0, +) / Double(numbers.count)
    }
    
    /// คำนวณ median
    /// - Parameter numbers: ชุดตัวเลข
    /// - Returns: ค่า median หรือ nil ถ้า array ว่าง
    public static func median(_ numbers: [Double]) -> Double? {
        guard !numbers.isEmpty else { return nil }
        
        let sorted = numbers.sorted()
        let count = sorted.count
        
        if count % 2 == 0 {
            return (sorted[count/2 - 1] + sorted[count/2]) / 2
        } else {
            return sorted[count/2]
        }
    }
    
    /// คำนวณ standard deviation
    /// - Parameter numbers: ชุดตัวเลข
    /// - Returns: ค่า standard deviation หรือ nil ถ้า array ว่าง
    public static func standardDeviation(_ numbers: [Double]) -> Double? {
        guard let avg = mean(numbers) else { return nil }
        
        let variance = numbers.map { pow($0 - avg, 2) }.reduce(0, +) / Double(numbers.count)
        return sqrt(variance)
    }
}
```

### การเขียน Tests ที่ดี

```swift
// Tests/MathKitTests/MathKitTests.swift

import XCTest
@testable import MathKit

final class MathKitTests: XCTestCase {
    
    // MARK: - Add Tests
    
    func testAddIntegers() {
        XCTAssertEqual(MathKit.add(2, 3), 5)
        XCTAssertEqual(MathKit.add(-1, 1), 0)
        XCTAssertEqual(MathKit.add(0, 0), 0)
    }
    
    func testAddDoubles() {
        XCTAssertEqual(MathKit.add(1.5, 2.5), 4.0, accuracy: 0.001)
    }
    
    // MARK: - Mean Tests
    
    func testMeanWithNumbers() {
        let numbers = [1.0, 2.0, 3.0, 4.0, 5.0]
        XCTAssertEqual(MathKit.mean(numbers), 3.0, accuracy: 0.001)
    }
    
    func testMeanWithEmptyArray() {
        XCTAssertNil(MathKit.mean([]))
    }
    
    // MARK: - Median Tests
    
    func testMedianWithOddCount() {
        let numbers = [1.0, 3.0, 5.0, 7.0, 9.0]
        XCTAssertEqual(MathKit.median(numbers), 5.0, accuracy: 0.001)
    }
    
    func testMedianWithEvenCount() {
        let numbers = [1.0, 2.0, 3.0, 4.0]
        XCTAssertEqual(MathKit.median(numbers), 2.5, accuracy: 0.001)
    }
    
    func testMedianWithEmptyArray() {
        XCTAssertNil(MathKit.median([]))
    }
    
    // MARK: - Standard Deviation Tests
    
    func testStandardDeviation() {
        let numbers = [2.0, 4.0, 4.0, 4.0, 5.0, 5.0, 7.0, 9.0]
        XCTAssertEqual(MathKit.standardDeviation(numbers), 2.0, accuracy: 0.001)
    }
    
    func testStandardDeviationWithSingleElement() {
        XCTAssertEqual(MathKit.standardDeviation([5.0]), 0.0, accuracy: 0.001)
    }
}
```

### Package.swift สำหรับ Library

```swift
// Package.swift
// swift-tools-version: 5.9

import PackageDescription

let package = Package(
    name: "MathKit",
    platforms: [
        .macOS(.v12),
        .iOS(.v15),
        .tvOS(.v15),
        .watchOS(.v8)
    ],
    products: [
        .library(
            name: "MathKit",
            targets: ["MathKit"]
        )
    ],
    targets: [
        .target(
            name: "MathKit",
            path: "Sources/MathKit",
            swiftSettings: [
                .enableUpcomingFeature("StrictConcurrency")
            ]
        ),
        .testTarget(
            name: "MathKitTests",
            dependencies: ["MathKit"],
            path: "Tests/MathKitTests"
        )
    ],
    swiftLanguageVersions: [.v5]
)
```

---

## 21. บทฝึกหัดปฏิบัติ

### Exercise 1: สร้าง String Utilities Library

สร้าง package ชื่อ `StringUtils` ที่มี utilities สำหรับ String operations

```swift
// Package.swift
// swift-tools-version: 5.9

import PackageDescription

let package = Package(
    name: "StringUtils",
    platforms: [
        .macOS(.v13),
        .iOS(.v16)
    ],
    products: [
        .library(name: "StringUtils", targets: ["StringUtils"])
    ],
    targets: [
        .target(name: "StringUtils"),
        .testTarget(
            name: "StringUtilsTests",
            dependencies: ["StringUtils"]
        )
    ]
)
```

```swift
// Sources/StringUtils/StringExtensions.swift

public extension String {
    
    /// ตรวจสอบว่าเป็น email ที่ valid หรือไม่
    var isValidEmail: Bool {
        let emailRegex = #"^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$"#
        return range(of: emailRegex, options: .regularExpression) != nil
    }
    
    /// ตรวจสอบว่าเป็น URL ที่ valid หรือไม่
    var isValidURL: Bool {
        guard let url = URL(string: self) else { return false }
        return url.scheme != nil && url.host != nil
    }
    
    /// แปลงเป็น camelCase
    var camelCased: String {
        let words = components(separatedBy: CharacterSet.alphanumerics.inverted)
        return words.enumerated().map { index, word in
            index == 0 ? word.lowercased() : word.capitalized
        }.joined()
    }
    
    /// แปลงเป็น snake_case
    var snakeCased: String {
        let pattern = "([a-z0-9])([A-Z])"
        let regex = try? NSRegularExpression(pattern: pattern, options: [])
        let range = NSRange(location: 0, length: utf16.count)
        return regex?.stringByReplacingMatches(
            in: self,
            options: [],
            range: range,
            withTemplate: "$1_$2"
        ).lowercased() ?? lowercased()
    }
    
    /// ตัด whitespace ทั้งหมด
    var trimmed: String {
        trimmingCharacters(in: .whitespacesAndNewlines)
    }
    
    /// นับจำนวนคำ
    var wordCount: Int {
        components(separatedBy: .whitespacesAndNewlines)
            .filter { !$0.isEmpty }
            .count
    }
    
    /// แปลงเป็น slug สำหรับ URL
    var slugified: String {
        lowercased()
            .replacingOccurrences(of: " ", with: "-")
            .filter { $0.isLetter || $0.isNumber || $0 == "-" }
    }
    
    /// Truncate string
    func truncated(to length: Int, trailing: String = "...") -> String {
        guard count > length else { return self }
        return String(prefix(length)) + trailing
    }
    
    /// ตรวจสอบว่า contains substring (case insensitive)
    func containsCaseInsensitive(_ substring: String) -> Bool {
        range(of: substring, options: .caseInsensitive) != nil
    }
}
```

```swift
// Tests/StringUtilsTests/StringExtensionsTests.swift

import XCTest
@testable import StringUtils

final class StringExtensionsTests: XCTestCase {
    
    func testEmailValidation() {
        XCTAssertTrue("user@example.com".isValidEmail)
        XCTAssertTrue("user.name+tag@example.co.uk".isValidEmail)
        XCTAssertFalse("notanemail".isValidEmail)
        XCTAssertFalse("@nodomain.com".isValidEmail)
        XCTAssertFalse("user@".isValidEmail)
    }
    
    func testURLValidation() {
        XCTAssertTrue("https://www.example.com".isValidURL)
        XCTAssertTrue("http://example.com/path?query=value".isValidURL)
        XCTAssertFalse("not a url".isValidURL)
        XCTAssertFalse("".isValidURL)
    }
    
    func testCamelCase() {
        XCTAssertEqual("hello world".camelCased, "helloWorld")
        XCTAssertEqual("my-variable-name".camelCased, "myVariableName")
    }
    
    func testSnakeCase() {
        XCTAssertEqual("camelCase".snakeCased, "camel_case")
        XCTAssertEqual("myVariableName".snakeCased, "my_variable_name")
    }
    
    func testWordCount() {
        XCTAssertEqual("Hello World".wordCount, 2)
        XCTAssertEqual("  spaces   between  ".wordCount, 2)
        XCTAssertEqual("".wordCount, 0)
    }
    
    func testTruncate() {
        XCTAssertEqual("Hello World".truncated(to: 5), "Hello...")
        XCTAssertEqual("Hi".truncated(to: 5), "Hi")
        XCTAssertEqual("Hello World".truncated(to: 5, trailing: "!"), "Hello!")
    }
    
    func testSlugify() {
        XCTAssertEqual("Hello World!".slugified, "hello-world")
        XCTAssertEqual("My Blog Post Title".slugified, "my-blog-post-title")
    }
}
```

### Exercise 2: สร้าง Date Utilities Package

```swift
// Package.swift
// swift-tools-version: 5.9

import PackageDescription

let package = Package(
    name: "DateUtils",
    platforms: [
        .macOS(.v13),
        .iOS(.v16)
    ],
    products: [
        .library(name: "DateUtils", targets: ["DateUtils"])
    ],
    targets: [
        .target(name: "DateUtils"),
        .testTarget(
            name: "DateUtilsTests",
            dependencies: ["DateUtils"]
        )
    ]
)
```

```swift
// Sources/DateUtils/DateExtensions.swift

import Foundation

public extension Date {
    
    /// แสดงวันที่แบบ relative (เช่น "2 hours ago")
    var relativeDescription: String {
        let formatter = RelativeDateTimeFormatter()
        formatter.unitsStyle = .full
        formatter.locale = Locale(identifier: "th_TH")
        return formatter.localizedString(for: self, relativeTo: Date())
    }
    
    /// แสดงวันที่แบบ Thai format
    var thaiDateString: String {
        let formatter = DateFormatter()
        formatter.dateStyle = .long
        formatter.locale = Locale(identifier: "th_TH")
        formatter.calendar = Calendar(identifier: .buddhist)
        return formatter.string(from: self)
    }
    
    /// ตรวจสอบว่าเป็นวันนี้หรือไม่
    var isToday: Bool {
        Calendar.current.isDateInToday(self)
    }
    
    /// ตรวจสอบว่าเป็นเมื่อวานหรือไม่
    var isYesterday: Bool {
        Calendar.current.isDateInYesterday(self)
    }
    
    /// ตรวจสอบว่าเป็นสัปดาห์นี้หรือไม่
    var isThisWeek: Bool {
        Calendar.current.isDate(self, equalTo: Date(), toGranularity: .weekOfYear)
    }
    
    /// เพิ่มวัน
    func adding(days: Int) -> Date {
        Calendar.current.date(byAdding: .day, value: days, to: self) ?? self
    }
    
    /// เพิ่มชั่วโมง
    func adding(hours: Int) -> Date {
        Calendar.current.date(byAdding: .hour, value: hours, to: self) ?? self
    }
    
    /// เริ่มต้นของวัน
    var startOfDay: Date {
        Calendar.current.startOfDay(for: self)
    }
    
    /// สิ้นสุดของวัน
    var endOfDay: Date {
        var components = DateComponents()
        components.day = 1
        components.second = -1
        return Calendar.current.date(byAdding: components, to: startOfDay) ?? self
    }
    
    /// จำนวนวันระหว่างสองวัน
    func daysBetween(_ date: Date) -> Int {
        let components = Calendar.current.dateComponents([.day], from: startOfDay, to: date.startOfDay)
        return abs(components.day ?? 0)
    }
}
```

### Exercise 3: Publish Library ไปยัง GitHub

```bash
#!/bin/bash
# publish_package.sh

# ตรวจสอบว่า tests ผ่าน
echo "Running tests..."
swift test
if [ $? -ne 0 ]; then
    echo "Tests failed! Aborting publish."
    exit 1
fi

# ตรวจสอบ version argument
if [ -z "$1" ]; then
    echo "Usage: $0 <version>"
    echo "Example: $0 1.0.0"
    exit 1
fi

VERSION=$1

# Commit changes
git add .
git commit -m "Release version $VERSION"

# สร้าง tag
git tag -a "$VERSION" -m "Version $VERSION"

# Push ไปยัง remote
git push origin main
git push origin "$VERSION"

echo "Successfully published version $VERSION!"
```

---

## สรุป

Swift Package Manager เป็นเครื่องมือที่ทรงพลังและเป็นมาตรฐานสำหรับการจัดการ dependencies ใน Swift projects ข้อดีหลักๆ ได้แก่:

1. **Official Apple tool** - รับการสนับสนุนโดยตรงจาก Apple
2. **Cross-platform** - ทำงานได้บน iOS, macOS, Linux, Windows
3. **Native Xcode integration** - ไม่ต้องตั้งค่าเพิ่มเติม
4. **Declarative syntax** - Package.swift อ่านและเข้าใจง่าย
5. **Security** - checksum verification สำหรับ binary targets

### สิ่งที่ควรจำ

```swift
// Package.swift พื้นฐาน
// 1. ระบุ swift-tools-version เสมอ
// 2. กำหนด platforms ที่รองรับ
// 3. ระบุ products (สิ่งที่ user ใช้ได้)
// 4. ระบุ dependencies (ของ package ที่ใช้)
// 5. ระบุ targets (code units)
```

### แหล่งข้อมูลเพิ่มเติม

- [Swift Package Manager Documentation](https://swift.org/documentation/package-manager/)
- [Apple's Swift Package Index](https://swiftpackageindex.com/)
- [WWDC Sessions on Swift Package Manager](https://developer.apple.com/videos/frameworks/swift-packages/)

---

*จบ Part 47: Swift Package Manager*
