# Part 89: การสร้าง Developer Tools และ Xcode Integrations ด้วย Swift

## บทนำ

การสร้าง developer tools เป็นทักษะสำคัญที่ช่วยเพิ่มประสิทธิภาพการทำงานของทีม Swift/iOS เครื่องมือที่ดีสามารถลดงานซ้ำซ้อน, บังคับใช้ coding standards, และทำให้ workflow ของทีมเป็นอัตโนมัติได้อย่างมีประสิทธิภาพ

ในบทนี้เราจะสำรวจการสร้างเครื่องมือสำหรับนักพัฒนาหลายประเภท ตั้งแต่ command-line tools ง่ายๆ ไปจนถึง Xcode extensions ที่ซับซ้อน และ Swift Package Manager plugins

---

## 1. ประเภทของ Developer Tools

### 1.1 Command-line Tools (CLI Tools)

Command-line tools เป็นโปรแกรมที่รันจาก terminal โดยตรง เหมาะสำหรับ:
- การ automate งานที่ทำซ้ำๆ
- การประมวลผล files
- การ integrate กับ CI/CD pipelines
- การสร้าง code generators

```swift
// ตัวอย่าง CLI tool พื้นฐาน
import Foundation

// รับ arguments จาก command line
let arguments = CommandLine.arguments
print("Tool name: \(arguments[0])")
print("Arguments: \(arguments.dropFirst().joined(separator: ", "))")
```

### 1.2 Xcode Extensions

Xcode extensions ช่วยให้เราสามารถเพิ่มฟีเจอร์ใหม่เข้าไปใน Xcode IDE ได้โดยตรง มี 2 ประเภทหลัก:
- **Source Editor Extensions**: แก้ไข source code ใน editor
- **Extension Types**: เพิ่ม menu items, keyboard shortcuts

### 1.3 Xcode Source Editor Extensions

Source Editor Extensions ใช้ XcodeKit framework ในการ manipulate source code:
- อ่านและเขียน text ใน editor
- จัดการ selections
- Insert/delete/replace text

### 1.4 Build Tool Plugins

SPM Build Tool Plugins รันอัตโนมัติในระหว่าง build process:
- **BuildToolPlugin**: รันระหว่าง build
- **CommandPlugin**: รันเมื่อผู้ใช้สั่ง

### 1.5 Code Generators

Code generators สร้าง Swift code โดยอัตโนมัติจาก:
- Schema files (JSON Schema, Protobuf)
- Templates
- Existing Swift code (reflection/parsing)
- API specifications (OpenAPI)

---

## 2. การสร้าง CLI Tools ด้วย Swift ArgumentParser

### 2.1 ติดตั้ง Swift ArgumentParser

เพิ่ม dependency ใน `Package.swift`:

```swift
// Package.swift
import PackageDescription

let package = Package(
    name: "MyDevTool",
    platforms: [
        .macOS(.v12)
    ],
    dependencies: [
        .package(
            url: "https://github.com/apple/swift-argument-parser",
            from: "1.3.0"
        ),
    ],
    targets: [
        .executableTarget(
            name: "mydevtool",
            dependencies: [
                .product(name: "ArgumentParser", package: "swift-argument-parser"),
            ]
        ),
        .testTarget(
            name: "MyDevToolTests",
            dependencies: ["mydevtool"]
        ),
    ]
)
```

### 2.2 Command พื้นฐาน

```swift
// Sources/mydevtool/main.swift
import ArgumentParser

@main
struct MyDevTool: ParsableCommand {
    static let configuration = CommandConfiguration(
        commandName: "mydevtool",
        abstract: "เครื่องมือสำหรับนักพัฒนา Swift",
        version: "1.0.0"
    )
    
    @Argument(help: "Path ของ Swift file ที่ต้องการประมวลผล")
    var inputFile: String
    
    @Option(name: .shortAndLong, help: "Output directory")
    var output: String = "."
    
    @Flag(name: .shortAndLong, help: "แสดง verbose output")
    var verbose: Bool = false
    
    mutating func run() throws {
        if verbose {
            print("Processing: \(inputFile)")
            print("Output directory: \(output)")
        }
        
        // ประมวลผล file
        try processFile(inputFile, outputDir: output)
    }
    
    private func processFile(_ path: String, outputDir: String) throws {
        let fileURL = URL(fileURLWithPath: path)
        guard FileManager.default.fileExists(atPath: path) else {
            throw ValidationError("ไม่พบไฟล์: \(path)")
        }
        
        let content = try String(contentsOf: fileURL, encoding: .utf8)
        print("File size: \(content.count) characters")
    }
}
```

### 2.3 Subcommands

```swift
// Sources/mydevtool/MyDevTool.swift
import ArgumentParser

@main
struct MyDevTool: ParsableCommand {
    static let configuration = CommandConfiguration(
        commandName: "mydevtool",
        abstract: "เครื่องมือสำหรับนักพัฒนา Swift",
        version: "1.0.0",
        subcommands: [
            GenerateCommand.self,
            LintCommand.self,
            FormatCommand.self,
            AnalyzeCommand.self
        ],
        defaultSubcommand: GenerateCommand.self
    )
}

// Generate subcommand
struct GenerateCommand: ParsableCommand {
    static let configuration = CommandConfiguration(
        commandName: "generate",
        abstract: "สร้าง Swift code จาก template หรือ schema"
    )
    
    @Argument(help: "ประเภทของ code ที่ต้องการสร้าง (model, viewmodel, test)")
    var type: GenerationType
    
    @Argument(help: "ชื่อของ component ที่จะสร้าง")
    var name: String
    
    @Option(name: .shortAndLong, help: "Output directory")
    var output: String = "."
    
    @Flag(name: [.customShort("f"), .long], help: "Overwrite existing files")
    var force: Bool = false
    
    mutating func run() throws {
        print("Generating \(type.rawValue): \(name)")
        
        let generator = CodeGenerator()
        try generator.generate(
            type: type,
            name: name,
            outputDir: output,
            force: force
        )
        
        print("✅ Generated successfully!")
    }
}

enum GenerationType: String, ExpressibleByArgument, CaseIterable {
    case model
    case viewModel = "viewmodel"
    case test
    case view
    case coordinator
    
    var defaultValueDescription: String { "model" }
}

// Lint subcommand
struct LintCommand: ParsableCommand {
    static let configuration = CommandConfiguration(
        commandName: "lint",
        abstract: "ตรวจสอบ Swift code ตาม coding standards"
    )
    
    @Argument(
        help: "Paths ของ files หรือ directories ที่ต้องการ lint",
        completion: .file(extensions: ["swift"])
    )
    var paths: [String]
    
    @Option(name: .shortAndLong, help: "Path ของ config file")
    var config: String = ".swiftlint.yml"
    
    @Flag(help: "แก้ไข violations อัตโนมัติ")
    var fix: Bool = false
    
    @Flag(help: "แสดงผลเป็น JSON format")
    var json: Bool = false
    
    mutating func run() throws {
        let linter = SwiftLinter(configPath: config)
        let results = try linter.lint(paths: paths, fix: fix)
        
        if json {
            let encoder = JSONEncoder()
            encoder.outputFormatting = .prettyPrinted
            let data = try encoder.encode(results)
            print(String(data: data, encoding: .utf8)!)
        } else {
            results.printSummary()
        }
        
        if results.hasErrors {
            throw ExitCode.failure
        }
    }
}
```

### 2.4 @Argument, @Option, @Flag ใช้งานอย่างละเอียด

```swift
import ArgumentParser

struct DetailedCommand: ParsableCommand {
    static let configuration = CommandConfiguration(
        abstract: "แสดงการใช้งาน @Argument, @Option, @Flag"
    )
    
    // @Argument - positional argument
    @Argument(
        help: ArgumentHelp(
            "Input file path",
            discussion: "Swift source file ที่ต้องการประมวลผล",
            valueName: "file"
        ),
        completion: .file(extensions: ["swift"])
    )
    var inputFile: String
    
    // @Argument ที่ optional
    @Argument(help: "Output file path (optional)")
    var outputFile: String?
    
    // @Argument ที่รับหลายค่า
    @Argument(
        help: "รายการ files ที่ต้องการประมวลผล",
        completion: .file(extensions: ["swift"])
    )
    var additionalFiles: [String] = []
    
    // @Option แบบ short และ long
    @Option(
        name: [.customShort("t"), .customLong("timeout")],
        help: "Timeout ในหน่วย seconds"
    )
    var timeout: Int = 30
    
    // @Option กับ custom type
    @Option(
        name: .shortAndLong,
        help: "Log level (debug, info, warning, error)"
    )
    var logLevel: LogLevel = .info
    
    // @Option ที่รับหลายค่า
    @Option(
        name: .shortAndLong,
        help: "Paths เพิ่มเติมสำหรับ search",
        completion: .directory
    )
    var searchPaths: [String] = []
    
    // @Flag แบบ boolean
    @Flag(
        name: .shortAndLong,
        help: "เปิดใช้ verbose output"
    )
    var verbose: Bool = false
    
    // @Flag แบบ inversion pair
    @Flag(
        inversion: .prefixedWithNo,
        help: "เปิด/ปิด color output"
    )
    var color: Bool = true
    
    // @Flag แบบ enum
    @Flag(help: "ระดับ optimization")
    var optimizationLevel: OptimizationLevel = .none
    
    mutating func run() throws {
        print("Input: \(inputFile)")
        print("Output: \(outputFile ?? "stdout")")
        print("Timeout: \(timeout)s")
        print("Log level: \(logLevel)")
        print("Verbose: \(verbose)")
        print("Color: \(color)")
        print("Optimization: \(optimizationLevel)")
        
        if !searchPaths.isEmpty {
            print("Search paths: \(searchPaths.joined(separator: ", "))")
        }
    }
}

enum LogLevel: String, ExpressibleByArgument, CaseIterable {
    case debug, info, warning, error
}

enum OptimizationLevel: EnumerableFlag {
    case none, basic, aggressive
    
    static func name(for value: Self) -> NameSpecification {
        switch value {
        case .none: return [.customLong("no-optimize")]
        case .basic: return [.customLong("optimize")]
        case .aggressive: return [.customLong("optimize-aggressive")]
        }
    }
}
```

### 2.5 Custom Validation

```swift
import ArgumentParser
import Foundation

struct ValidatedCommand: ParsableCommand {
    @Argument(help: "Swift source file")
    var inputFile: String
    
    @Option(name: .shortAndLong, help: "จำนวน threads")
    var threads: Int = 4
    
    @Option(name: .shortAndLong, help: "Output format")
    var format: OutputFormat = .text
    
    // Custom validation
    mutating func validate() throws {
        // ตรวจสอบว่า file มีอยู่จริง
        guard FileManager.default.fileExists(atPath: inputFile) else {
            throw ValidationError("ไม่พบไฟล์: '\(inputFile)'")
        }
        
        // ตรวจสอบ extension
        guard inputFile.hasSuffix(".swift") else {
            throw ValidationError("ไฟล์ต้องเป็น .swift file")
        }
        
        // ตรวจสอบ threads range
        guard (1...32).contains(threads) else {
            throw ValidationError("จำนวน threads ต้องอยู่ระหว่าง 1-32 (ได้รับ: \(threads))")
        }
    }
    
    mutating func run() throws {
        print("Processing \(inputFile) with \(threads) threads")
    }
}

enum OutputFormat: String, ExpressibleByArgument {
    case text, json, xml, html
}
```

### 2.6 Async CLI Commands

```swift
import ArgumentParser
import Foundation

struct AsyncCommand: AsyncParsableCommand {
    static let configuration = CommandConfiguration(
        commandName: "fetch",
        abstract: "ดึงข้อมูลจาก API แบบ async"
    )
    
    @Argument(help: "URL ที่ต้องการดึงข้อมูล")
    var url: String
    
    @Option(name: .shortAndLong, help: "Timeout ในหน่วย seconds")
    var timeout: Double = 30.0
    
    @Flag(name: .shortAndLong, help: "แสดง headers")
    var showHeaders: Bool = false
    
    mutating func run() async throws {
        guard let requestURL = URL(string: url) else {
            throw ValidationError("URL ไม่ถูกต้อง: \(url)")
        }
        
        print("Fetching \(url)...")
        
        var request = URLRequest(url: requestURL)
        request.timeoutInterval = timeout
        
        let (data, response) = try await URLSession.shared.data(for: request)
        
        if showHeaders, let httpResponse = response as? HTTPURLResponse {
            print("\n=== Headers ===")
            for (key, value) in httpResponse.allHeaderFields {
                print("\(key): \(value)")
            }
        }
        
        if let body = String(data: data, encoding: .utf8) {
            print("\n=== Body ===")
            print(body)
        }
        
        print("\n✅ Done! (\(data.count) bytes)")
    }
}
```

### 2.7 Progress Reporting

```swift
import ArgumentParser
import Foundation

struct ProcessFilesCommand: AsyncParsableCommand {
    static let configuration = CommandConfiguration(
        commandName: "process",
        abstract: "ประมวลผล Swift files พร้อม progress"
    )
    
    @Argument(help: "Directory ที่ต้องการประมวลผล")
    var directory: String
    
    mutating func run() async throws {
        let fileManager = FileManager.default
        let dirURL = URL(fileURLWithPath: directory)
        
        // หา Swift files ทั้งหมด
        guard let enumerator = fileManager.enumerator(
            at: dirURL,
            includingPropertiesForKeys: [.isRegularFileKey],
            options: [.skipsHiddenFiles]
        ) else {
            throw ValidationError("ไม่สามารถอ่าน directory: \(directory)")
        }
        
        var swiftFiles: [URL] = []
        for case let fileURL as URL in enumerator {
            if fileURL.pathExtension == "swift" {
                swiftFiles.append(fileURL)
            }
        }
        
        let total = swiftFiles.count
        print("พบ \(total) Swift files")
        
        // ประมวลผลพร้อม progress bar
        for (index, fileURL) in swiftFiles.enumerated() {
            let current = index + 1
            let progress = Double(current) / Double(total)
            
            // แสดง progress bar
            printProgress(current: current, total: total, progress: progress)
            
            // ประมวลผล file
            try await processSwiftFile(fileURL)
        }
        
        // Clear progress line แล้วแสดงผลสำเร็จ
        print("\r\u{1B}[K✅ ประมวลผลสำเร็จ \(total) files")
    }
    
    private func printProgress(current: Int, total: Int, progress: Double) {
        let barWidth = 40
        let filled = Int(Double(barWidth) * progress)
        let empty = barWidth - filled
        
        let bar = String(repeating: "█", count: filled) + String(repeating: "░", count: empty)
        let percentage = Int(progress * 100)
        
        // \r ไปต้นบรรทัด, \u{1B}[K ลบถึงสิ้นบรรทัด
        print("\r\u{1B}[K[\(bar)] \(percentage)% (\(current)/\(total))", terminator: "")
        fflush(stdout)
    }
    
    private func processSwiftFile(_ url: URL) async throws {
        // จำลองการประมวลผล
        try await Task.sleep(nanoseconds: 10_000_000) // 10ms
        _ = try String(contentsOf: url, encoding: .utf8)
    }
}
```

### 2.8 Color Output ด้วย ANSI Escape Codes

```swift
// Sources/mydevtool/ANSIColor.swift
import Foundation

// ANSI color codes สำหรับ terminal output
enum ANSIColor: String {
    // Text colors
    case black = "\u{1B}[30m"
    case red = "\u{1B}[31m"
    case green = "\u{1B}[32m"
    case yellow = "\u{1B}[33m"
    case blue = "\u{1B}[34m"
    case magenta = "\u{1B}[35m"
    case cyan = "\u{1B}[36m"
    case white = "\u{1B}[37m"
    
    // Bright colors
    case brightRed = "\u{1B}[91m"
    case brightGreen = "\u{1B}[92m"
    case brightYellow = "\u{1B}[93m"
    case brightBlue = "\u{1B}[94m"
    
    // Background colors
    case bgRed = "\u{1B}[41m"
    case bgGreen = "\u{1B}[42m"
    case bgYellow = "\u{1B}[43m"
    case bgBlue = "\u{1B}[44m"
    
    // Styles
    case bold = "\u{1B}[1m"
    case italic = "\u{1B}[3m"
    case underline = "\u{1B}[4m"
    
    // Reset
    case reset = "\u{1B}[0m"
}

extension String {
    // ตรวจสอบว่า terminal รองรับ color หรือไม่
    static var supportsANSI: Bool {
        guard let term = ProcessInfo.processInfo.environment["TERM"] else {
            return false
        }
        return term != "dumb" && isatty(STDOUT_FILENO) != 0
    }
    
    func colored(_ color: ANSIColor) -> String {
        guard String.supportsANSI else { return self }
        return "\(color.rawValue)\(self)\(ANSIColor.reset.rawValue)"
    }
    
    var bold: String { colored(.bold) }
    var red: String { colored(.red) }
    var green: String { colored(.green) }
    var yellow: String { colored(.yellow) }
    var blue: String { colored(.blue) }
    var cyan: String { colored(.cyan) }
}

// ตัวอย่างการใช้งาน
struct ColorOutputExample {
    static func demonstrate() {
        print("✅ Success".green.bold)
        print("⚠️ Warning".yellow)
        print("❌ Error".red.bold)
        print("ℹ️ Info".blue)
        
        // Print table with colors
        let headers = ["File", "Lines", "Issues"]
        let rows = [
            ["main.swift", "150", "0"],
            ["ContentView.swift", "89", "2"],
            ["ViewModel.swift", "210", "1"]
        ]
        
        printTable(headers: headers, rows: rows)
    }
    
    static func printTable(headers: [String], rows: [[String]]) {
        // คำนวณ column widths
        var widths = headers.map { $0.count }
        for row in rows {
            for (i, cell) in row.enumerated() {
                widths[i] = max(widths[i], cell.count)
            }
        }
        
        // Print header
        let headerRow = headers.enumerated().map { i, h in
            h.padding(toLength: widths[i], withPad: " ", startingAt: 0).bold
        }.joined(separator: " | ")
        
        let separator = widths.map { String(repeating: "-", count: $0) }.joined(separator: "-+-")
        
        print(headerRow)
        print(separator)
        
        // Print rows
        for row in rows {
            let rowStr = row.enumerated().map { i, cell in
                cell.padding(toLength: widths[i], withPad: " ", startingAt: 0)
            }.joined(separator: " | ")
            print(rowStr)
        }
    }
}
```

### 2.9 Unit Testing CLI Tools

```swift
// Tests/MyDevToolTests/GenerateCommandTests.swift
import XCTest
import ArgumentParser
@testable import mydevtool

final class GenerateCommandTests: XCTestCase {
    var tempDir: URL!
    
    override func setUp() {
        super.setUp()
        // สร้าง temporary directory สำหรับ test output
        tempDir = FileManager.default.temporaryDirectory
            .appendingPathComponent(UUID().uuidString)
        try! FileManager.default.createDirectory(
            at: tempDir,
            withIntermediateDirectories: true
        )
    }
    
    override func tearDown() {
        try? FileManager.default.removeItem(at: tempDir)
        super.tearDown()
    }
    
    func testGenerateModel() throws {
        var command = GenerateCommand()
        command.type = .model
        command.name = "User"
        command.output = tempDir.path
        
        XCTAssertNoThrow(try command.run())
        
        let generatedFile = tempDir
            .appendingPathComponent("User.swift")
        
        XCTAssertTrue(
            FileManager.default.fileExists(atPath: generatedFile.path),
            "ควรสร้าง User.swift"
        )
        
        let content = try String(contentsOf: generatedFile, encoding: .utf8)
        XCTAssertTrue(content.contains("struct User"), "ควรมี struct User")
        XCTAssertTrue(content.contains("Codable"), "ควร conform Codable")
    }
    
    func testGenerateViewModel() throws {
        var command = GenerateCommand()
        command.type = .viewModel
        command.name = "Profile"
        command.output = tempDir.path
        
        XCTAssertNoThrow(try command.run())
        
        let generatedFile = tempDir
            .appendingPathComponent("ProfileViewModel.swift")
        
        XCTAssertTrue(
            FileManager.default.fileExists(atPath: generatedFile.path)
        )
        
        let content = try String(contentsOf: generatedFile, encoding: .utf8)
        XCTAssertTrue(content.contains("@MainActor"))
        XCTAssertTrue(content.contains("ObservableObject"))
    }
    
    func testValidation_MissingName() {
        var command = GenerateCommand()
        command.type = .model
        command.name = ""
        command.output = tempDir.path
        
        XCTAssertThrowsError(try command.validate()) { error in
            XCTAssertTrue(error is ValidationError)
        }
    }
    
    func testParsing() throws {
        // ทดสอบการ parse arguments
        let command = try GenerateCommand.parse([
            "model",
            "User",
            "--output", "/tmp",
            "--force"
        ])
        
        XCTAssertEqual(command.type, .model)
        XCTAssertEqual(command.name, "User")
        XCTAssertEqual(command.output, "/tmp")
        XCTAssertTrue(command.force)
    }
}

// Test สำหรับ Lint command
final class LintCommandTests: XCTestCase {
    func testValidSwiftFile() throws {
        let tempFile = FileManager.default.temporaryDirectory
            .appendingPathComponent("test_\(UUID().uuidString).swift")
        
        let validCode = """
        import Foundation
        
        struct MyModel: Codable {
            let id: Int
            let name: String
        }
        """
        
        try validCode.write(to: tempFile, atomically: true, encoding: .utf8)
        defer { try? FileManager.default.removeItem(at: tempFile) }
        
        var command = LintCommand()
        command.paths = [tempFile.path]
        
        XCTAssertNoThrow(try command.run())
    }
}
```

---

## 3. Swift ArgumentParser Deep Dive

### 3.1 Completion Scripts

ArgumentParser สามารถ generate completion scripts สำหรับ shell ต่างๆ ได้:

```swift
import ArgumentParser

@main
struct MainTool: ParsableCommand {
    static let configuration = CommandConfiguration(
        commandName: "swifttool",
        abstract: "Swift Developer Tool",
        subcommands: [
            GenerateCommand.self,
            LintCommand.self,
        ]
    )
}

// รันเพื่อ generate completion script:
// swifttool --generate-completion-script zsh
// swifttool --generate-completion-script bash
// swifttool --generate-completion-script fish
```

การติดตั้ง completion script:

```bash
# สำหรับ Zsh
swifttool --generate-completion-script zsh > ~/.zsh/completions/_swifttool
source ~/.zshrc

# สำหรับ Bash
swifttool --generate-completion-script bash > ~/.bash_completion.d/swifttool
source ~/.bashrc
```

### 3.2 Custom Completion

```swift
struct CommandWithCompletions: ParsableCommand {
    @Option(
        name: .shortAndLong,
        help: "ชื่อ target",
        completion: .custom { _ in
            // Return list ของ targets จาก project
            return loadTargetsFromProject()
        }
    )
    var target: String = ""
    
    @Argument(
        help: "File path",
        completion: .file(extensions: ["swift", "swiftinterface"])
    )
    var file: String
    
    @Option(
        help: "Configuration",
        completion: .list(["debug", "release", "profile"])
    )
    var configuration: String = "debug"
}

func loadTargetsFromProject() -> [String] {
    // อ่าน targets จาก .xcodeproj หรือ Package.swift
    return ["MyApp", "MyAppTests", "MyFramework"]
}
```

### 3.3 Help Text Generation

```swift
struct WellDocumentedCommand: ParsableCommand {
    static let configuration = CommandConfiguration(
        commandName: "analyze",
        abstract: "วิเคราะห์ Swift source code",
        discussion: """
        คำสั่งนี้ทำการวิเคราะห์ Swift source code และรายงาน:
        - Complexity metrics
        - Code duplicates
        - Unused declarations
        - Memory management issues
        
        ตัวอย่างการใช้งาน:
        
          analyze ./Sources
          analyze ./Sources --format json --output report.json
          analyze ./Sources --threshold 10 --ignore-tests
        """
    )
    
    @Argument(
        help: ArgumentHelp(
            "Path ของ source code",
            discussion: "สามารถระบุเป็น file หรือ directory ก็ได้",
            valueName: "path"
        ),
        completion: .directory
    )
    var sourcePath: String
    
    @Option(
        name: [.short, .long],
        help: ArgumentHelp(
            "Complexity threshold",
            discussion: "Functions ที่มี complexity เกินกว่านี้จะถูก flag",
            valueName: "number"
        )
    )
    var threshold: Int = 10
    
    @Flag(
        help: "ละเว้น test files"
    )
    var ignoreTests: Bool = false
    
    mutating func run() throws {
        print("Analyzing: \(sourcePath)")
    }
}
```

---

## 4. Xcode Source Editor Extensions

### 4.1 การตั้งค่า Project

สร้าง macOS App project ใหม่ แล้วเพิ่ม Xcode Source Editor Extension target:

1. File → New → Target
2. เลือก "Xcode Source Editor Extension"
3. ตั้งชื่อ extension

### 4.2 XCSourceEditorCommand

```swift
// MyExtension/SourceEditorCommand.swift
import Foundation
import XcodeKit

class SourceEditorCommand: NSObject, XCSourceEditorCommand {
    func perform(
        with invocation: XCSourceEditorCommandInvocation,
        completionHandler: @escaping (Error?) -> Void
    ) {
        defer { completionHandler(nil) }
        
        let buffer = invocation.buffer
        
        // ดึง selected text
        guard let selection = buffer.selections.firstObject as? XCSourceTextRange else {
            return
        }
        
        // ประมวลผล
        performTransformation(buffer: buffer, selection: selection)
    }
    
    private func performTransformation(
        buffer: XCSourceTextBuffer,
        selection: XCSourceTextRange
    ) {
        let lines = buffer.lines as! [String]
        let startLine = selection.start.line
        let endLine = selection.end.line
        
        // ประมวลผลแต่ละบรรทัดใน selection
        for lineIndex in startLine...endLine {
            let line = lines[lineIndex]
            buffer.lines[lineIndex] = transformLine(line)
        }
    }
    
    private func transformLine(_ line: String) -> String {
        // override ใน subclasses
        return line
    }
}
```

### 4.3 Text Manipulation APIs

```swift
// SortLinesCommand.swift
import Foundation
import XcodeKit

class SortLinesCommand: NSObject, XCSourceEditorCommand {
    func perform(
        with invocation: XCSourceEditorCommandInvocation,
        completionHandler: @escaping (Error?) -> Void
    ) {
        let buffer = invocation.buffer
        
        guard let selection = buffer.selections.firstObject as? XCSourceTextRange else {
            completionHandler(nil)
            return
        }
        
        let startLine = selection.start.line
        let endLine = selection.end.line
        
        guard startLine < endLine else {
            completionHandler(nil)
            return
        }
        
        // ดึงบรรทัดที่ต้องการ sort
        var selectedLines: [String] = []
        for i in startLine...endLine {
            selectedLines.append(buffer.lines[i] as! String)
        }
        
        // Sort lines
        let sortedLines = selectedLines.sorted { $0.trimmingCharacters(in: .whitespaces) < $1.trimmingCharacters(in: .whitespaces) }
        
        // แทนที่บรรทัดเดิม
        for (offset, line) in sortedLines.enumerated() {
            buffer.lines[startLine + offset] = line
        }
        
        completionHandler(nil)
    }
}

// AddDocumentationCommand.swift
class AddDocumentationCommand: NSObject, XCSourceEditorCommand {
    func perform(
        with invocation: XCSourceEditorCommandInvocation,
        completionHandler: @escaping (Error?) -> Void
    ) {
        let buffer = invocation.buffer
        
        guard let selection = buffer.selections.firstObject as? XCSourceTextRange else {
            completionHandler(nil)
            return
        }
        
        let currentLine = selection.start.line
        let line = buffer.lines[currentLine] as! String
        
        // ตรวจสอบว่าเป็น function declaration
        if let docComment = generateDocComment(for: line) {
            // Insert doc comment ก่อน function
            let indentation = extractIndentation(from: line)
            let commentLines = docComment.components(separatedBy: "\n")
                .map { indentation + $0 + "\n" }
            
            let insertionIndex = currentLine
            for (offset, commentLine) in commentLines.enumerated() {
                buffer.lines.insert(commentLine, at: insertionIndex + offset)
            }
        }
        
        completionHandler(nil)
    }
    
    private func generateDocComment(for line: String) -> String? {
        let trimmed = line.trimmingCharacters(in: .whitespaces)
        
        // ตรวจหา function pattern
        let funcPattern = #"func\s+(\w+)\s*\(([^)]*)\)\s*(?:->.*)?$"#
        guard let regex = try? NSRegularExpression(pattern: funcPattern),
              let match = regex.firstMatch(
                in: trimmed,
                range: NSRange(trimmed.startIndex..., in: trimmed)
              ) else {
            return nil
        }
        
        let funcName = String(trimmed[Range(match.range(at: 1), in: trimmed)!])
        let params = String(trimmed[Range(match.range(at: 2), in: trimmed)!])
        
        var doc = "/// <#Description#>\n"
        
        // Parse parameters
        if !params.isEmpty {
            doc += "///\n"
            let paramList = params.components(separatedBy: ",")
            for param in paramList {
                let parts = param.trimmingCharacters(in: .whitespaces)
                    .components(separatedBy: ":")
                let paramName = parts.first?.trimmingCharacters(in: .whitespaces) ?? ""
                if !paramName.isEmpty {
                    doc += "/// - Parameter \(paramName): <#\(paramName) description#>\n"
                }
            }
        }
        
        _ = funcName // suppress warning
        return doc
    }
    
    private func extractIndentation(from line: String) -> String {
        var indentation = ""
        for char in line {
            if char == " " || char == "\t" {
                indentation.append(char)
            } else {
                break
            }
        }
        return indentation
    }
}
```

### 4.4 Extension Info.plist Configuration

```xml
<!-- MyExtension/Info.plist -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
    "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>XCSourceEditorCommandDefinitions</key>
    <array>
        <dict>
            <key>XCSourceEditorCommandClassName</key>
            <string>$(PRODUCT_MODULE_NAME).SortLinesCommand</string>
            <key>XCSourceEditorCommandIdentifier</key>
            <string>$(PRODUCT_BUNDLE_IDENTIFIER).SortLines</string>
            <key>XCSourceEditorCommandName</key>
            <string>Sort Selected Lines</string>
        </dict>
        <dict>
            <key>XCSourceEditorCommandClassName</key>
            <string>$(PRODUCT_MODULE_NAME).AddDocumentationCommand</string>
            <key>XCSourceEditorCommandIdentifier</key>
            <string>$(PRODUCT_BUNDLE_IDENTIFIER).AddDocumentation</string>
            <key>XCSourceEditorCommandName</key>
            <string>Add Documentation Comment</string>
        </dict>
    </array>
    <key>XCSourceEditorExtensionPrincipalClass</key>
    <string>$(PRODUCT_MODULE_NAME).SourceEditorExtension</string>
</dict>
</plist>
```

---

## 5. SPM Build Tool Plugins

### 5.1 BuildToolPlugin

BuildToolPlugin รันอัตโนมัติระหว่าง build process:

```swift
// Plugins/MyBuildPlugin/plugin.swift
import PackagePlugin

@main
struct MyBuildPlugin: BuildToolPlugin {
    func createBuildCommands(
        context: PluginContext,
        target: Target
    ) async throws -> [Command] {
        
        // หา executable tool
        let generator = try context.tool(named: "MyCodeGenerator")
        
        // หา input files
        guard let sourceTarget = target as? SourceModuleTarget else {
            return []
        }
        
        // หา .schema files
        let schemaFiles = sourceTarget.sourceFiles.filter { file in
            file.path.extension == "schema"
        }
        
        // สร้าง command สำหรับแต่ละ schema file
        return schemaFiles.map { schemaFile in
            let outputFile = context.pluginWorkDirectory
                .appending(schemaFile.path.stem + ".swift")
            
            return .buildCommand(
                displayName: "Generating code from \(schemaFile.path.lastComponent)",
                executable: generator.path,
                arguments: [
                    schemaFile.path.string,
                    "--output", outputFile.string
                ],
                inputFiles: [schemaFile.path],
                outputFiles: [outputFile]
            )
        }
    }
}
```

Package.swift สำหรับ plugin:

```swift
// Package.swift
import PackageDescription

let package = Package(
    name: "MyPlugin",
    products: [
        .plugin(
            name: "MyBuildPlugin",
            targets: ["MyBuildPlugin"]
        ),
        .executable(
            name: "MyCodeGenerator",
            targets: ["MyCodeGenerator"]
        )
    ],
    targets: [
        .plugin(
            name: "MyBuildPlugin",
            capability: .buildTool(),
            dependencies: ["MyCodeGenerator"]
        ),
        .executableTarget(
            name: "MyCodeGenerator",
            path: "Sources/MyCodeGenerator"
        )
    ]
)
```

### 5.2 CommandPlugin

CommandPlugin รันเมื่อผู้ใช้สั่ง (ไม่ได้รันอัตโนมัติ):

```swift
// Plugins/SwiftFormatPlugin/plugin.swift
import PackagePlugin
import Foundation

@main
struct SwiftFormatPlugin: CommandPlugin {
    func performCommand(
        context: PluginContext,
        arguments: [String]
    ) async throws {
        
        let swiftFormat = try context.tool(named: "swift-format")
        
        // Parse arguments
        var argExtractor = ArgumentExtractor(arguments)
        let targetNames = argExtractor.extractOption(named: "target")
        let inPlace = argExtractor.extractFlag(named: "in-place")
        
        // เลือก targets
        let targets: [Target]
        if targetNames.isEmpty {
            targets = context.package.targets
        } else {
            targets = try targetNames.map {
                try context.package.targets(named: [$0]).first!
            }
        }
        
        // รัน swift-format สำหรับแต่ละ target
        for target in targets {
            guard let sourceTarget = target as? SourceModuleTarget else {
                continue
            }
            
            let swiftFiles = sourceTarget.sourceFiles
                .filter { $0.path.extension == "swift" }
                .map { $0.path.string }
            
            if swiftFiles.isEmpty { continue }
            
            var formatArgs = ["format"]
            if inPlace { formatArgs.append("--in-place") }
            formatArgs.append(contentsOf: swiftFiles)
            
            let process = Process()
            process.executableURL = URL(fileURLWithPath: swiftFormat.path.string)
            process.arguments = formatArgs
            
            try process.run()
            process.waitUntilExit()
            
            if process.terminationStatus != 0 {
                throw PluginError.formatFailed(target: target.name)
            }
            
            print("✅ Formatted \(target.name)")
        }
    }
}

enum PluginError: Error, CustomStringConvertible {
    case formatFailed(target: String)
    
    var description: String {
        switch self {
        case .formatFailed(let target):
            return "Format failed for target: \(target)"
        }
    }
}
```

### 5.3 SwiftLint Plugin ตัวอย่าง

```swift
// Plugins/SwiftLintPlugin/plugin.swift
import PackagePlugin

@main
struct SwiftLintPlugin: BuildToolPlugin {
    func createBuildCommands(
        context: PluginContext,
        target: Target
    ) async throws -> [Command] {
        
        let swiftLint = try context.tool(named: "swiftlint")
        
        guard let sourceTarget = target as? SourceModuleTarget else {
            return []
        }
        
        // หา config file
        let configFile = context.package.directory.appending(".swiftlint.yml")
        
        let swiftFiles = sourceTarget.sourceFiles
            .filter { $0.path.extension == "swift" }
            .map { $0.path }
        
        // สร้าง output directory
        let outputDir = context.pluginWorkDirectory
        
        return [
            .prebuildCommand(
                displayName: "SwiftLint",
                executable: swiftLint.path,
                arguments: [
                    "lint",
                    "--config", configFile.string,
                    "--reporter", "xcode"
                ] + swiftFiles.map { $0.string },
                outputFilesDirectory: outputDir
            )
        ]
    }
}
```

### 5.4 Protobuf Code Generation Plugin

```swift
// Plugins/SwiftProtobufPlugin/plugin.swift
import PackagePlugin
import Foundation

@main
struct SwiftProtobufPlugin: BuildToolPlugin {
    func createBuildCommands(
        context: PluginContext,
        target: Target
    ) async throws -> [Command] {
        
        let protoc = try context.tool(named: "protoc")
        let protocGenSwift = try context.tool(named: "protoc-gen-swift")
        
        guard let sourceTarget = target as? SourceModuleTarget else {
            return []
        }
        
        // หา .proto files
        let protoFiles = sourceTarget.sourceFiles.filter {
            $0.path.extension == "proto"
        }
        
        guard !protoFiles.isEmpty else { return [] }
        
        // Output directory
        let outputDir = context.pluginWorkDirectory.appending("GeneratedSources")
        
        // สร้าง command สำหรับทุก proto files พร้อมกัน
        let outputFiles = protoFiles.map { protoFile in
            outputDir.appending(protoFile.path.stem + ".pb.swift")
        }
        
        let command = Command.buildCommand(
            displayName: "Generating Swift from Protobuf",
            executable: protoc.path,
            arguments: [
                "--swift_out=\(outputDir.string)",
                "--swift_opt=Visibility=Public",
                "--plugin=protoc-gen-swift=\(protocGenSwift.path.string)",
                "-I", sourceTarget.directory.string
            ] + protoFiles.map { $0.path.string },
            environment: ["PATH": "/usr/local/bin:/usr/bin:/bin"],
            inputFiles: protoFiles.map { $0.path },
            outputFiles: outputFiles
        )
        
        return [command]
    }
}
```

---

## 6. Swift Code Generation

### 6.1 SwiftSyntax สำหรับ Parsing

```swift
// Package.swift dependency
// .package(url: "https://github.com/apple/swift-syntax", from: "509.0.0")

import SwiftSyntax
import SwiftSyntaxParser

// Parse Swift code
func parseSwiftFile(_ content: String) throws -> SourceFileSyntax {
    return try SyntaxParser.parse(source: content)
}

// Custom SyntaxVisitor สำหรับเก็บข้อมูล
class StructCollector: SyntaxVisitor {
    var structs: [StructInfo] = []
    
    override func visit(_ node: StructDeclSyntax) -> SyntaxVisitorContinueKind {
        let name = node.name.text
        let properties = extractProperties(from: node)
        let protocols = extractProtocols(from: node)
        
        structs.append(StructInfo(
            name: name,
            properties: properties,
            conformedProtocols: protocols
        ))
        
        return .visitChildren
    }
    
    private func extractProperties(from node: StructDeclSyntax) -> [PropertyInfo] {
        var properties: [PropertyInfo] = []
        
        for member in node.memberBlock.members {
            if let varDecl = member.decl.as(VariableDeclSyntax.self) {
                for binding in varDecl.bindings {
                    if let identifier = binding.pattern.as(IdentifierPatternSyntax.self) {
                        let name = identifier.identifier.text
                        let type = binding.typeAnnotation?.type.description ?? "Unknown"
                        let isOptional = type.hasSuffix("?")
                        
                        properties.append(PropertyInfo(
                            name: name,
                            typeName: type,
                            isOptional: isOptional
                        ))
                    }
                }
            }
        }
        
        return properties
    }
    
    private func extractProtocols(from node: StructDeclSyntax) -> [String] {
        guard let inheritanceClause = node.inheritanceClause else {
            return []
        }
        
        return inheritanceClause.inheritedTypes.compactMap { type in
            type.type.as(IdentifierTypeSyntax.self)?.name.text
        }
    }
}

struct StructInfo {
    let name: String
    let properties: [PropertyInfo]
    let conformedProtocols: [String]
}

struct PropertyInfo {
    let name: String
    let typeName: String
    let isOptional: Bool
}
```

### 6.2 Code Generation จาก Template

```swift
// Sources/CodeGenerator/Templates.swift
import Foundation

struct CodeTemplate {
    static func generateModel(name: String, properties: [(String, String)]) -> String {
        var code = """
        // Generated by SwiftDevTool - DO NOT EDIT
        // Generated at: \(ISO8601DateFormatter().string(from: Date()))
        
        import Foundation
        
        struct \(name): Codable, Equatable {
        
        """
        
        // Properties
        for (propName, propType) in properties {
            code += "    let \(propName): \(propType)\n"
        }
        
        code += "\n"
        
        // CodingKeys
        code += "    enum CodingKeys: String, CodingKey {\n"
        for (propName, _) in properties {
            let snakeCase = toSnakeCase(propName)
            if snakeCase != propName {
                code += "        case \(propName) = \"\(snakeCase)\"\n"
            } else {
                code += "        case \(propName)\n"
            }
        }
        code += "    }\n"
        
        code += "}\n"
        
        return code
    }
    
    static func generateViewModel(name: String) -> String {
        return """
        // Generated by SwiftDevTool - DO NOT EDIT
        
        import Foundation
        import SwiftUI
        
        @MainActor
        final class \(name)ViewModel: ObservableObject {
            
            // MARK: - Published Properties
            @Published private(set) var isLoading = false
            @Published private(set) var error: Error?
            
            // MARK: - Private Properties
            private let service: \(name)ServiceProtocol
            
            // MARK: - Init
            init(service: \(name)ServiceProtocol = \(name)Service()) {
                self.service = service
            }
            
            // MARK: - Methods
            func load() async {
                isLoading = true
                defer { isLoading = false }
                
                do {
                    // TODO: Implement load logic
                } catch {
                    self.error = error
                }
            }
        }
        
        protocol \(name)ServiceProtocol {
            // TODO: Define service protocol
        }
        
        final class \(name)Service: \(name)ServiceProtocol {
            // TODO: Implement service
        }
        """
    }
    
    private static func toSnakeCase(_ camelCase: String) -> String {
        var result = ""
        for (index, char) in camelCase.enumerated() {
            if char.isUppercase && index > 0 {
                result += "_"
            }
            result += char.lowercased()
        }
        return result
    }
}
```

### 6.3 Sourcery Usage

Sourcery เป็น tool สำหรับ meta-programming ใน Swift:

```swift
// sourcery templates/AutoEquatable.stencil
{% for type in types.structs where type.implements.AutoEquatable %}
// MARK: - \{{ type.name }} AutoEquatable
extension \{{ type.name }}: Equatable {
    static func == (lhs: \{{ type.name }}, rhs: \{{ type.name }}) -> Bool {
        return {% for variable in type.variables where !variable.isStatic %}
        lhs.\{{ variable.name }} == rhs.\{{ variable.name }}{% if not forloop.last %} &&{% endif %}
        {% endfor %}
    }
}
{% endfor %}
```

```swift
// ใช้ annotation ในโค้ด
// sourcery: AutoEquatable
struct User {
    let id: UUID
    let name: String
    let email: String
}

// Sourcery จะ generate:
extension User: Equatable {
    static func == (lhs: User, rhs: User) -> Bool {
        return lhs.id == rhs.id &&
        lhs.name == rhs.name &&
        lhs.email == rhs.email
    }
}
```

---

## 7. Swift Macros

### 7.1 Expression Macros

```swift
// Sources/MyMacros/StringifyMacro.swift
import SwiftSyntax
import SwiftSyntaxMacros
import SwiftCompilerPlugin

public struct StringifyMacro: ExpressionMacro {
    public static func expansion(
        of node: some FreestandingMacroExpansionSyntax,
        in context: some MacroExpansionContext
    ) throws -> ExprSyntax {
        guard let argument = node.arguments.first?.expression else {
            throw MacroExpansionErrorMessage("ต้องระบุ argument")
        }
        
        return "(\(argument), \(literal: argument.description))"
    }
}

// Client-side declaration
@freestanding(expression)
public macro stringify<T>(_ value: T) -> (T, String) = #externalMacro(
    module: "MyMacros",
    type: "StringifyMacro"
)

// การใช้งาน
let result = #stringify(1 + 2)
// result = (3, "1 + 2")
```

### 7.2 Declaration Macros

```swift
// Sources/MyMacros/AutoEquatableMacro.swift
import SwiftSyntax
import SwiftSyntaxMacros

public struct AutoEquatableMacro: MemberMacro {
    public static func expansion(
        of node: AttributeSyntax,
        providingMembersOf declaration: some DeclGroupSyntax,
        in context: some MacroExpansionContext
    ) throws -> [DeclSyntax] {
        
        // ดึง stored properties
        let members = declaration.memberBlock.members
        let properties = members.compactMap { member -> String? in
            guard let varDecl = member.decl.as(VariableDeclSyntax.self),
                  varDecl.bindingSpecifier.tokenKind == .keyword(.let) ||
                  varDecl.bindingSpecifier.tokenKind == .keyword(.var) else {
                return nil
            }
            
            return varDecl.bindings.first?
                .pattern.as(IdentifierPatternSyntax.self)?
                .identifier.text
        }
        
        guard !properties.isEmpty else { return [] }
        
        let comparisons = properties
            .map { "lhs.\($0) == rhs.\($0)" }
            .joined(separator: " &&\n        ")
        
        let equatableImpl: DeclSyntax = """
        static func == (lhs: Self, rhs: Self) -> Bool {
            \(raw: comparisons)
        }
        """
        
        return [equatableImpl]
    }
}

// Peer macro เพื่อ add Equatable conformance
public struct AutoEquatableConformanceMacro: ExtensionMacro {
    public static func expansion(
        of node: AttributeSyntax,
        attachedTo declaration: some DeclGroupSyntax,
        providingExtensionsOf type: some TypeSyntaxProtocol,
        conformingTo protocols: [TypeSyntax],
        in context: some MacroExpansionContext
    ) throws -> [ExtensionDeclSyntax] {
        
        let equatableExtension: DeclSyntax = """
        extension \(type): Equatable {}
        """
        
        return [equatableExtension.cast(ExtensionDeclSyntax.self)]
    }
}
```

### 7.3 การ Register Macros

```swift
// Sources/MyMacros/MyMacrosPlugin.swift
import SwiftCompilerPlugin
import SwiftSyntaxMacros

@main
struct MyMacrosPlugin: CompilerPlugin {
    let providingMacros: [Macro.Type] = [
        StringifyMacro.self,
        AutoEquatableMacro.self,
    ]
}
```

### 7.4 @AutoEquatable Macro สมบูรณ์

```swift
// Sources/MyMacroClient/main.swift
import MyMacroDefinitions

// Macro declaration
@attached(member, names: named(==))
@attached(extension, conformances: Equatable)
public macro AutoEquatable() = #externalMacro(
    module: "MyMacros",
    type: "AutoEquatableMacro"
)

// การใช้งาน
@AutoEquatable
struct Person {
    let id: UUID
    let firstName: String
    let lastName: String
    let age: Int
}

// Macro จะ generate:
// extension Person: Equatable {
//     static func == (lhs: Person, rhs: Person) -> Bool {
//         lhs.id == rhs.id &&
//         lhs.firstName == rhs.firstName &&
//         lhs.lastName == rhs.lastName &&
//         lhs.age == rhs.age
//     }
// }

// ทดสอบ
let p1 = Person(id: UUID(), firstName: "John", lastName: "Doe", age: 30)
let p2 = Person(id: p1.id, firstName: "John", lastName: "Doe", age: 30)
let p3 = Person(id: UUID(), firstName: "Jane", lastName: "Doe", age: 25)

print(p1 == p2) // true
print(p1 == p3) // false
```

### 7.5 Testing Macros

```swift
// Tests/MyMacrosTests/AutoEquatableTests.swift
import SwiftSyntaxMacros
import SwiftSyntaxMacrosTestSupport
import XCTest

final class AutoEquatableTests: XCTestCase {
    let testMacros: [String: Macro.Type] = [
        "AutoEquatable": AutoEquatableMacro.self,
    ]
    
    func testBasicEquatable() {
        assertMacroExpansion(
            """
            @AutoEquatable
            struct Point {
                let x: Double
                let y: Double
            }
            """,
            expandedSource: """
            struct Point {
                let x: Double
                let y: Double
                static func == (lhs: Self, rhs: Self) -> Bool {
                    lhs.x == rhs.x &&
                    lhs.y == rhs.y
                }
            }
            
            extension Point: Equatable {}
            """,
            macros: testMacros
        )
    }
    
    func testEmptyStruct() {
        assertMacroExpansion(
            """
            @AutoEquatable
            struct Empty {}
            """,
            expandedSource: """
            struct Empty {}
            
            extension Empty: Equatable {}
            """,
            macros: testMacros
        )
    }
}
```

---

## 8. Code Analysis Tools

### 8.1 SwiftSyntax สำหรับ Static Analysis

```swift
// Sources/Analyzer/ComplexityAnalyzer.swift
import SwiftSyntax
import SwiftSyntaxParser
import Foundation

struct ComplexityResult {
    let functionName: String
    let filePath: String
    let lineNumber: Int
    let cyclomaticComplexity: Int
    let isOverThreshold: Bool
}

class CyclomaticComplexityVisitor: SyntaxVisitor {
    var results: [ComplexityResult] = []
    let threshold: Int
    private var currentFunction: String = ""
    private var currentComplexity = 0
    private var currentLine = 0
    private let filePath: String
    
    init(filePath: String, threshold: Int = 10) {
        self.filePath = filePath
        self.threshold = threshold
        super.init(viewMode: .sourceAccurate)
    }
    
    override func visit(_ node: FunctionDeclSyntax) -> SyntaxVisitorContinueKind {
        currentFunction = node.name.text
        currentComplexity = 1 // เริ่มต้นที่ 1
        currentLine = node.position.line
        return .visitChildren
    }
    
    override func visitPost(_ node: FunctionDeclSyntax) {
        results.append(ComplexityResult(
            functionName: currentFunction,
            filePath: filePath,
            lineNumber: currentLine,
            cyclomaticComplexity: currentComplexity,
            isOverThreshold: currentComplexity > threshold
        ))
    }
    
    // เพิ่ม complexity สำหรับ branching statements
    override func visit(_ node: IfExprSyntax) -> SyntaxVisitorContinueKind {
        currentComplexity += 1
        return .visitChildren
    }
    
    override func visit(_ node: ForStmtSyntax) -> SyntaxVisitorContinueKind {
        currentComplexity += 1
        return .visitChildren
    }
    
    override func visit(_ node: WhileStmtSyntax) -> SyntaxVisitorContinueKind {
        currentComplexity += 1
        return .visitChildren
    }
    
    override func visit(_ node: SwitchCaseSyntax) -> SyntaxVisitorContinueKind {
        currentComplexity += 1
        return .visitChildren
    }
    
    override func visit(_ node: CatchClauseSyntax) -> SyntaxVisitorContinueKind {
        currentComplexity += 1
        return .visitChildren
    }
    
    override func visit(_ node: TernaryExprSyntax) -> SyntaxVisitorContinueKind {
        currentComplexity += 1
        return .visitChildren
    }
    
    override func visit(_ node: GuardStmtSyntax) -> SyntaxVisitorContinueKind {
        currentComplexity += 1
        return .visitChildren
    }
}
```

### 8.2 Custom SwiftLint Rules

```swift
// Sources/CustomRules/ForceCastRule.swift
import SwiftLintFramework
import SwiftSyntax

struct NoForcedOptionalRule: Rule, ConfigurationProviderRule {
    var configuration = SeverityConfiguration<Self>(.warning)
    
    static let description = RuleDescription(
        identifier: "no_forced_optional",
        name: "No Forced Optional Unwrapping",
        description: "หลีกเลี่ยงการใช้ force unwrap (!)",
        kind: .idiomatic,
        nonTriggeringExamples: [
            Example("guard let value = optional else { return }"),
            Example("if let value = optional { use(value) }"),
            Example("let value = optional ?? defaultValue")
        ],
        triggeringExamples: [
            Example("let value = ↓optional!"),
            Example("↓optional!.method()")
        ]
    )
    
    func validate(file: SwiftLintFile) -> [StyleViolation] {
        return file.match(
            pattern: #"\w+!"#,
            with: [.identifier]
        ).map { range in
            StyleViolation(
                ruleDescription: Self.description,
                severity: configuration.severity,
                location: Location(file: file, characterOffset: range.location)
            )
        }
    }
}
```

### 8.3 Simple Linter

```swift
// Sources/SimpleLinter/Linter.swift
import Foundation
import SwiftSyntax
import SwiftSyntaxParser

struct LintViolation {
    let file: String
    let line: Int
    let column: Int
    let severity: Severity
    let message: String
    let ruleID: String
    
    enum Severity: String {
        case warning = "warning"
        case error = "error"
    }
}

protocol LintRule {
    var id: String { get }
    var description: String { get }
    func check(syntax: SourceFileSyntax, in file: String) -> [LintViolation]
}

struct LineLengthRule: LintRule {
    let id = "line_length"
    let description = "บรรทัดไม่ควรยาวเกิน 120 characters"
    let maxLength = 120
    
    func check(syntax: SourceFileSyntax, in file: String) -> [LintViolation] {
        let source = syntax.description
        let lines = source.components(separatedBy: "\n")
        
        return lines.enumerated().compactMap { lineNumber, line in
            guard line.count > maxLength else { return nil }
            return LintViolation(
                file: file,
                line: lineNumber + 1,
                column: maxLength + 1,
                severity: .warning,
                message: "บรรทัดยาว \(line.count) characters (สูงสุด \(maxLength))",
                ruleID: id
            )
        }
    }
}

class SimpleLinter {
    private let rules: [LintRule]
    
    init(rules: [LintRule] = [LineLengthRule()]) {
        self.rules = rules
    }
    
    func lint(files: [String]) throws -> [LintViolation] {
        var violations: [LintViolation] = []
        
        for filePath in files {
            let content = try String(contentsOfFile: filePath, encoding: .utf8)
            let syntax = try SyntaxParser.parse(source: content)
            
            for rule in rules {
                violations.append(contentsOf: rule.check(syntax: syntax, in: filePath))
            }
        }
        
        return violations.sorted { $0.file < $1.file || ($0.file == $1.file && $0.line < $1.line) }
    }
}
```

---

## 9. Building Tuist (Overview)

### 9.1 Project Generation Concepts

Tuist และ XcodeGen เป็น tools สำหรับ generate `.xcodeproj` files จาก definition files:

```swift
// Project.swift (Tuist)
import ProjectDescription

let project = Project(
    name: "MyApp",
    organizationName: "MyOrg",
    settings: .settings(
        base: [
            "SWIFT_VERSION": "5.9",
            "IPHONEOS_DEPLOYMENT_TARGET": "16.0"
        ]
    ),
    targets: [
        Target(
            name: "MyApp",
            platform: .iOS,
            product: .app,
            bundleId: "com.myorg.myapp",
            infoPlist: .extendingDefault(with: [
                "UILaunchScreen": .dictionary([:])
            ]),
            sources: ["Sources/**"],
            resources: ["Resources/**"],
            dependencies: [
                .target(name: "MyFramework"),
                .external(name: "Alamofire")
            ]
        ),
        Target(
            name: "MyFramework",
            platform: .iOS,
            product: .framework,
            bundleId: "com.myorg.myframework",
            sources: ["Framework/**"]
        ),
        Target(
            name: "MyAppTests",
            platform: .iOS,
            product: .unitTests,
            bundleId: "com.myorg.myapptests",
            sources: ["Tests/**"],
            dependencies: [
                .target(name: "MyApp")
            ]
        )
    ]
)
```

### 9.2 XcodeGen Format

```yaml
# project.yml (XcodeGen)
name: MyApp
options:
  bundleIdPrefix: com.myorg
  deploymentTarget:
    iOS: 16.0

settings:
  base:
    SWIFT_VERSION: "5.9"

targets:
  MyApp:
    type: application
    platform: iOS
    sources:
      - Sources
    resources:
      - Resources
    settings:
      base:
        PRODUCT_BUNDLE_IDENTIFIER: com.myorg.myapp
    dependencies:
      - target: MyFramework
      - package: Alamofire
    
  MyFramework:
    type: framework
    platform: iOS
    sources:
      - Framework

packages:
  Alamofire:
    url: https://github.com/Alamofire/Alamofire
    from: 5.8.0
```

---

## 10. Developer Productivity Tools

### 10.1 License Header Inserter

```swift
// Sources/LicenseInserter/main.swift
import Foundation
import ArgumentParser

@main
struct LicenseInserter: ParsableCommand {
    static let configuration = CommandConfiguration(
        commandName: "license-inserter",
        abstract: "เพิ่ม license header ใน Swift files"
    )
    
    @Argument(help: "Directory ที่ต้องการประมวลผล")
    var directory: String
    
    @Option(name: .shortAndLong, help: "License template file")
    var template: String = "LICENSE_HEADER.txt"
    
    @Flag(help: "Overwrite existing headers")
    var force: Bool = false
    
    @Flag(name: .shortAndLong, help: "Dry run mode")
    var dryRun: Bool = false
    
    mutating func run() throws {
        let templateURL = URL(fileURLWithPath: template)
        let licenseTemplate = try String(contentsOf: templateURL, encoding: .utf8)
        
        let fileManager = FileManager.default
        let dirURL = URL(fileURLWithPath: directory)
        
        guard let enumerator = fileManager.enumerator(
            at: dirURL,
            includingPropertiesForKeys: [.isRegularFileKey],
            options: [.skipsHiddenFiles]
        ) else {
            throw ValidationError("ไม่สามารถอ่าน directory")
        }
        
        var processed = 0
        var skipped = 0
        
        for case let fileURL as URL in enumerator {
            guard fileURL.pathExtension == "swift" else { continue }
            
            let content = try String(contentsOf: fileURL, encoding: .utf8)
            
            // ตรวจสอบว่ามี license header แล้วหรือไม่
            if !force && content.hasPrefix("//") {
                let firstLine = content.components(separatedBy: "\n").first ?? ""
                if firstLine.contains("Copyright") || firstLine.contains("Licensed") {
                    skipped += 1
                    continue
                }
            }
            
            // สร้าง header สำหรับ file นี้
            let fileName = fileURL.lastPathComponent
            let year = Calendar.current.component(.year, from: Date())
            let header = licenseTemplate
                .replacingOccurrences(of: "{FILENAME}", with: fileName)
                .replacingOccurrences(of: "{YEAR}", with: "\(year)")
            
            let newContent = header + "\n\n" + content
            
            if dryRun {
                print("Would update: \(fileURL.path)")
            } else {
                try newContent.write(to: fileURL, atomically: true, encoding: .utf8)
                print("Updated: \(fileURL.lastPathComponent)")
            }
            
            processed += 1
        }
        
        print("\n✅ Processed: \(processed), Skipped: \(skipped)")
    }
}
```

### 10.2 Localization Key Extractor

```swift
// Sources/LocalizationExtractor/main.swift
import Foundation
import ArgumentParser
import SwiftSyntax
import SwiftSyntaxParser

@main
struct LocalizationExtractor: AsyncParsableCommand {
    static let configuration = CommandConfiguration(
        commandName: "extract-strings",
        abstract: "ดึง localization keys จาก Swift source code"
    )
    
    @Argument(help: "Source directory")
    var sourceDirectory: String
    
    @Option(name: .shortAndLong, help: "Output Strings file")
    var output: String = "Localizable.strings"
    
    @Flag(help: "Merge กับ existing strings file")
    var merge: Bool = false
    
    mutating func run() async throws {
        let extractor = LocalizationKeyExtractor()
        
        print("Scanning \(sourceDirectory)...")
        let keys = try await extractor.extractKeys(from: sourceDirectory)
        
        print("พบ \(keys.count) keys")
        
        var existingKeys: [String: String] = [:]
        
        if merge, let existing = try? String(contentsOfFile: output, encoding: .utf8) {
            existingKeys = parseStringsFile(existing)
            print("Existing keys: \(existingKeys.count)")
        }
        
        // Merge keys
        for key in keys {
            if existingKeys[key] == nil {
                existingKeys[key] = ""  // Empty translation
            }
        }
        
        // Write output
        let outputContent = generateStringsFile(existingKeys)
        let outputURL = URL(fileURLWithPath: output)
        try outputContent.write(to: outputURL, atomically: true, encoding: .utf8)
        
        print("✅ Written to \(output)")
    }
    
    private func parseStringsFile(_ content: String) -> [String: String] {
        var result: [String: String] = [:]
        let pattern = #""([^"]+)"\s*=\s*"([^"]*)"\s*;"#
        
        guard let regex = try? NSRegularExpression(pattern: pattern) else { return result }
        
        let matches = regex.matches(in: content, range: NSRange(content.startIndex..., in: content))
        for match in matches {
            if let keyRange = Range(match.range(at: 1), in: content),
               let valueRange = Range(match.range(at: 2), in: content) {
                result[String(content[keyRange])] = String(content[valueRange])
            }
        }
        
        return result
    }
    
    private func generateStringsFile(_ keys: [String: String]) -> String {
        var lines: [String] = [
            "/* Auto-generated by LocalizationExtractor */",
            ""
        ]
        
        for (key, value) in keys.sorted(by: { $0.key < $1.key }) {
            lines.append("\"\(key)\" = \"\(value.isEmpty ? key : value)\";")
        }
        
        return lines.joined(separator: "\n")
    }
}

class LocalizationKeyExtractor: SyntaxVisitor {
    private var extractedKeys: Set<String> = []
    
    init() {
        super.init(viewMode: .sourceAccurate)
    }
    
    func extractKeys(from directory: String) async throws -> [String] {
        let fileManager = FileManager.default
        let dirURL = URL(fileURLWithPath: directory)
        
        guard let enumerator = fileManager.enumerator(
            at: dirURL,
            includingPropertiesForKeys: [.isRegularFileKey],
            options: [.skipsHiddenFiles]
        ) else {
            return []
        }
        
        for case let fileURL as URL in enumerator {
            guard fileURL.pathExtension == "swift" else { continue }
            
            let content = try String(contentsOf: fileURL, encoding: .utf8)
            let syntax = try SyntaxParser.parse(source: content)
            walk(syntax)
        }
        
        return Array(extractedKeys).sorted()
    }
    
    // ดัก String(localized:) calls
    override func visit(_ node: FunctionCallExprSyntax) -> SyntaxVisitorContinueKind {
        if let memberAccess = node.calledExpression.as(MemberAccessExprSyntax.self) {
            // NSLocalizedString("key", ...)
            _ = memberAccess
        }
        
        // String(localized: "key")
        if let declRef = node.calledExpression.as(DeclReferenceExprSyntax.self),
           declRef.baseName.text == "String" {
            for arg in node.arguments {
                if arg.label?.text == "localized",
                   let stringLiteral = arg.expression.as(StringLiteralExprSyntax.self) {
                    let key = stringLiteral.segments.description
                        .trimmingCharacters(in: .init(charactersIn: "\""))
                    extractedKeys.insert(key)
                }
            }
        }
        
        return .visitChildren
    }
    
    override func visit(_ node: MacroExpansionExprSyntax) -> SyntaxVisitorContinueKind {
        // #localized("key")
        if node.macroName.text == "localized" {
            if let firstArg = node.arguments.first,
               let stringLiteral = firstArg.expression.as(StringLiteralExprSyntax.self) {
                let key = stringLiteral.segments.description
                    .trimmingCharacters(in: .init(charactersIn: "\""))
                extractedKeys.insert(key)
            }
        }
        return .visitChildren
    }
}
```

### 10.3 API Client Generator จาก OpenAPI Spec

```swift
// Sources/APIGenerator/OpenAPIParser.swift
import Foundation
import Yams // YAML parser

struct OpenAPISpec: Codable {
    let openapi: String
    let info: APIInfo
    let paths: [String: PathItem]
    let components: Components?
    
    struct APIInfo: Codable {
        let title: String
        let version: String
    }
    
    struct PathItem: Codable {
        let get: Operation?
        let post: Operation?
        let put: Operation?
        let delete: Operation?
        let patch: Operation?
    }
    
    struct Operation: Codable {
        let operationId: String?
        let summary: String?
        let parameters: [Parameter]?
        let requestBody: RequestBody?
        let responses: [String: Response]
    }
    
    struct Parameter: Codable {
        let name: String
        let `in`: ParameterLocation
        let required: Bool?
        let schema: Schema?
        
        enum ParameterLocation: String, Codable {
            case path, query, header, cookie
        }
    }
    
    struct RequestBody: Codable {
        let required: Bool?
        let content: [String: MediaType]
    }
    
    struct MediaType: Codable {
        let schema: Schema?
    }
    
    struct Response: Codable {
        let description: String
        let content: [String: MediaType]?
    }
    
    struct Schema: Codable {
        let type: String?
        let ref: String?
        let properties: [String: Schema]?
        let items: Schema?
        
        enum CodingKeys: String, CodingKey {
            case type
            case ref = "$ref"
            case properties
            case items
        }
    }
    
    struct Components: Codable {
        let schemas: [String: Schema]?
    }
}

class APIClientGenerator {
    func generate(from specPath: String, outputDir: String) throws {
        let specContent = try String(contentsOfFile: specPath, encoding: .utf8)
        
        // Parse YAML หรือ JSON
        let spec: OpenAPISpec
        if specPath.hasSuffix(".yaml") || specPath.hasSuffix(".yml") {
            // ใช้ Yams library
            spec = try YAMLDecoder().decode(OpenAPISpec.self, from: specContent)
        } else {
            spec = try JSONDecoder().decode(OpenAPISpec.self, from: specContent.data(using: .utf8)!)
        }
        
        // Generate API client
        let clientCode = generateAPIClient(from: spec)
        let clientURL = URL(fileURLWithPath: outputDir)
            .appendingPathComponent("APIClient.swift")
        try clientCode.write(to: clientURL, atomically: true, encoding: .utf8)
        
        // Generate models
        if let schemas = spec.components?.schemas {
            let modelsCode = generateModels(from: schemas)
            let modelsURL = URL(fileURLWithPath: outputDir)
                .appendingPathComponent("APIModels.swift")
            try modelsCode.write(to: modelsURL, atomically: true, encoding: .utf8)
        }
        
        print("✅ Generated API client in \(outputDir)")
    }
    
    private func generateAPIClient(from spec: OpenAPISpec) -> String {
        var code = """
        // Generated from OpenAPI spec: \(spec.info.title) v\(spec.info.version)
        // DO NOT EDIT - Auto-generated file
        
        import Foundation
        
        class APIClient {
            private let baseURL: URL
            private let session: URLSession
            
            init(baseURL: URL, session: URLSession = .shared) {
                self.baseURL = baseURL
                self.session = session
            }
        
        """
        
        for (path, pathItem) in spec.paths {
            if let getOp = pathItem.get {
                code += generateMethod(path: path, method: "GET", operation: getOp)
            }
            if let postOp = pathItem.post {
                code += generateMethod(path: path, method: "POST", operation: postOp)
            }
            if let putOp = pathItem.put {
                code += generateMethod(path: path, method: "PUT", operation: putOp)
            }
            if let deleteOp = pathItem.delete {
                code += generateMethod(path: path, method: "DELETE", operation: deleteOp)
            }
        }
        
        code += "}\n"
        return code
    }
    
    private func generateMethod(
        path: String,
        method: String,
        operation: OpenAPISpec.Operation
    ) -> String {
        let funcName = operation.operationId ?? "\(method.lowercased())\(path.replacingOccurrences(of: "/", with: "_").capitalized)"
        
        return """
            func \(funcName)() async throws -> Data {
                var url = baseURL.appendingPathComponent("\(path)")
                var request = URLRequest(url: url)
                request.httpMethod = "\(method)"
                let (data, _) = try await session.data(for: request)
                return data
            }
        
        """
    }
    
    private func generateModels(from schemas: [String: OpenAPISpec.Schema]) -> String {
        var code = """
        // Generated Models
        // DO NOT EDIT - Auto-generated file
        
        import Foundation
        
        """
        
        for (name, schema) in schemas {
            code += generateModel(name: name, schema: schema)
            code += "\n"
        }
        
        return code
    }
    
    private func generateModel(name: String, schema: OpenAPISpec.Schema) -> String {
        guard let properties = schema.properties else {
            return "struct \(name): Codable {}\n"
        }
        
        var code = "struct \(name): Codable {\n"
        
        for (propName, propSchema) in properties {
            let swiftType = swiftType(for: propSchema)
            code += "    let \(propName): \(swiftType)\n"
        }
        
        code += "}\n"
        return code
    }
    
    private func swiftType(for schema: OpenAPISpec.Schema) -> String {
        if let ref = schema.ref {
            return ref.components(separatedBy: "/").last ?? "Any"
        }
        
        switch schema.type {
        case "string": return "String"
        case "integer": return "Int"
        case "number": return "Double"
        case "boolean": return "Bool"
        case "array":
            if let items = schema.items {
                return "[\(swiftType(for: items))]"
            }
            return "[Any]"
        default: return "Any"
        }
    }
}
```

---

## 11. Publishing และ Distribution

### 11.1 Homebrew Formula

```ruby
# Formula/mydevtool.rb
class Mydevtool < Formula
  desc "Swift developer tool for code generation and analysis"
  homepage "https://github.com/myorg/mydevtool"
  url "https://github.com/myorg/mydevtool/archive/refs/tags/v1.0.0.tar.gz"
  sha256 "abc123..." # SHA256 ของ release archive
  license "MIT"
  
  depends_on xcode: ["14.0", :build]
  
  def install
    system "swift", "build", "--configuration", "release",
           "--disable-sandbox"
    
    bin.install ".build/release/mydevtool"
    
    # Install shell completions
    generate_completions_from_executable(
      bin/"mydevtool",
      "--generate-completion-script"
    )
  end
  
  test do
    assert_match "mydevtool", shell_output("#{bin}/mydevtool --version")
  end
end
```

### 11.2 Mint Package Manager

Mint เป็น package manager สำหรับ Swift command-line tools:

```bash
# ติดตั้ง tool ด้วย Mint
mint install apple/swift-argument-parser
mint install realm/SwiftLint

# รัน tool ด้วย Mint
mint run swiftlint

# ระบุ version
mint install realm/SwiftLint@0.53.0
```

```swift
// Mintfile - ระบุ dependencies สำหรับ project
SwiftLint/SwiftLint@0.53.0
apple/swift-format@509.0.0
nicklockwood/SwiftFormat@0.52.0
```

### 11.3 Swift Package Index

```swift
// Package.swift สำหรับ publish CLI tool
import PackageDescription

let package = Package(
    name: "MyDevTool",
    platforms: [
        .macOS(.v12)
    ],
    products: [
        // ระบุเป็น executable เพื่อให้ Swift Package Index รู้
        .executable(
            name: "mydevtool",
            targets: ["MyDevTool"]
        )
    ],
    dependencies: [
        .package(
            url: "https://github.com/apple/swift-argument-parser",
            from: "1.3.0"
        ),
    ],
    targets: [
        .executableTarget(
            name: "MyDevTool",
            dependencies: [
                .product(name: "ArgumentParser", package: "swift-argument-parser"),
            ]
        ),
    ]
)
```

---

## 12. Complete Project: Swift Code Formatter/Linter CLI

### 12.1 Project Structure

```
SwiftFormatter/
├── Package.swift
├── Sources/
│   └── SwiftFormatter/
│       ├── main.swift
│       ├── Commands/
│       │   ├── FormatCommand.swift
│       │   ├── LintCommand.swift
│       │   └── CheckCommand.swift
│       ├── Formatter/
│       │   ├── SwiftFormatter.swift
│       │   ├── Rules/
│       │   │   ├── IndentationRule.swift
│       │   │   ├── TrailingWhitespaceRule.swift
│       │   │   └── ImportSortingRule.swift
│       │   └── Configuration.swift
│       └── Output/
│           ├── Reporter.swift
│           └── Diff.swift
└── Tests/
    └── SwiftFormatterTests/
        ├── FormatCommandTests.swift
        ├── Rules/
        │   ├── IndentationRuleTests.swift
        │   └── TrailingWhitespaceRuleTests.swift
        └── FormatterTests.swift
```

### 12.2 Package.swift

```swift
import PackageDescription

let package = Package(
    name: "SwiftFormatter",
    platforms: [.macOS(.v12)],
    dependencies: [
        .package(url: "https://github.com/apple/swift-argument-parser", from: "1.3.0"),
        .package(url: "https://github.com/apple/swift-syntax", from: "509.0.0"),
    ],
    targets: [
        .executableTarget(
            name: "SwiftFormatter",
            dependencies: [
                .product(name: "ArgumentParser", package: "swift-argument-parser"),
                .product(name: "SwiftSyntax", package: "swift-syntax"),
                .product(name: "SwiftSyntaxParser", package: "swift-syntax"),
                .product(name: "SwiftOperators", package: "swift-syntax"),
            ]
        ),
        .testTarget(
            name: "SwiftFormatterTests",
            dependencies: ["SwiftFormatter"]
        ),
    ]
)
```

### 12.3 Main Entry Point

```swift
// Sources/SwiftFormatter/main.swift
import ArgumentParser

@main
struct SwiftFormatterTool: ParsableCommand {
    static let configuration = CommandConfiguration(
        commandName: "swiftfmt",
        abstract: "Swift code formatter และ linter",
        version: "1.0.0",
        subcommands: [
            FormatCommand.self,
            LintCommand.self,
            CheckCommand.self,
        ]
    )
}
```

### 12.4 Format Command

```swift
// Sources/SwiftFormatter/Commands/FormatCommand.swift
import ArgumentParser
import Foundation

struct FormatCommand: AsyncParsableCommand {
    static let configuration = CommandConfiguration(
        commandName: "format",
        abstract: "จัดรูปแบบ Swift code"
    )
    
    @Argument(
        help: "Files หรือ directories ที่ต้องการ format",
        completion: .file(extensions: ["swift"])
    )
    var paths: [String]
    
    @Option(name: .shortAndLong, help: "Config file path")
    var config: String = ".swiftfmt.json"
    
    @Flag(name: .shortAndLong, help: "แสดง diff แทนที่จะ write")
    var dryRun: Bool = false
    
    @Flag(name: .shortAndLong, help: "Verbose output")
    var verbose: Bool = false
    
    mutating func run() async throws {
        let configuration = try loadConfiguration()
        let formatter = SwiftCodeFormatter(configuration: configuration)
        
        let swiftFiles = try collectSwiftFiles(from: paths)
        
        print("Formatting \(swiftFiles.count) files...")
        
        var formattedCount = 0
        var errorCount = 0
        
        for filePath in swiftFiles {
            do {
                let result = try await formatter.format(file: filePath)
                
                switch result {
                case .unchanged:
                    if verbose { print("  \(filePath) (unchanged)") }
                    
                case .formatted(let newContent):
                    if dryRun {
                        print("Would format: \(filePath)")
                        if verbose {
                            printDiff(original: result.original ?? "", formatted: newContent, file: filePath)
                        }
                    } else {
                        try newContent.write(toFile: filePath, atomically: true, encoding: .utf8)
                        print("  Formatted: \((filePath as NSString).lastPathComponent)")
                        formattedCount += 1
                    }
                }
            } catch {
                print("  ❌ Error in \(filePath): \(error.localizedDescription)".red)
                errorCount += 1
            }
        }
        
        print("\n✅ Done: \(formattedCount) formatted, \(errorCount) errors")
        
        if errorCount > 0 {
            throw ExitCode.failure
        }
    }
    
    private func loadConfiguration() throws -> FormatterConfiguration {
        let configURL = URL(fileURLWithPath: config)
        
        if FileManager.default.fileExists(atPath: config) {
            let data = try Data(contentsOf: configURL)
            return try JSONDecoder().decode(FormatterConfiguration.self, from: data)
        }
        
        return FormatterConfiguration.default
    }
    
    private func collectSwiftFiles(from paths: [String]) throws -> [String] {
        var files: [String] = []
        let fileManager = FileManager.default
        
        for path in paths {
            var isDirectory: ObjCBool = false
            guard fileManager.fileExists(atPath: path, isDirectory: &isDirectory) else {
                throw ValidationError("ไม่พบ path: \(path)")
            }
            
            if isDirectory.boolValue {
                guard let enumerator = fileManager.enumerator(
                    at: URL(fileURLWithPath: path),
                    includingPropertiesForKeys: nil,
                    options: [.skipsHiddenFiles]
                ) else { continue }
                
                for case let fileURL as URL in enumerator {
                    if fileURL.pathExtension == "swift" {
                        files.append(fileURL.path)
                    }
                }
            } else if path.hasSuffix(".swift") {
                files.append(path)
            }
        }
        
        return files.sorted()
    }
    
    private func printDiff(original: String, formatted: String, file: String) {
        print("--- \(file) (original)")
        print("+++ \(file) (formatted)")
        
        let originalLines = original.components(separatedBy: "\n")
        let formattedLines = formatted.components(separatedBy: "\n")
        
        // Simple line-by-line diff
        let maxLines = max(originalLines.count, formattedLines.count)
        for i in 0..<maxLines {
            let orig = i < originalLines.count ? originalLines[i] : nil
            let fmt = i < formattedLines.count ? formattedLines[i] : nil
            
            if orig != fmt {
                if let orig = orig { print("- \(orig)".red) }
                if let fmt = fmt { print("+ \(fmt)".green) }
            }
        }
    }
}
```

### 12.5 Formatter Implementation

```swift
// Sources/SwiftFormatter/Formatter/SwiftFormatter.swift
import Foundation
import SwiftSyntax
import SwiftSyntaxParser

enum FormatterResult {
    case unchanged
    case formatted(String)
    
    var original: String? { nil }
}

struct FormatterConfiguration: Codable {
    var indentationWidth: Int = 4
    var useTabs: Bool = false
    var maxLineLength: Int = 120
    var sortImports: Bool = true
    var trailingNewline: Bool = true
    var removeTrailingWhitespace: Bool = true
    
    static let `default` = FormatterConfiguration()
}

class SwiftCodeFormatter {
    private let configuration: FormatterConfiguration
    private let rules: [FormatterRule]
    
    init(configuration: FormatterConfiguration) {
        self.configuration = configuration
        self.rules = [
            TrailingWhitespaceRule(config: configuration),
            ImportSortingRule(config: configuration),
            TrailingNewlineRule(config: configuration),
        ]
    }
    
    func format(file path: String) async throws -> FormatterResult {
        let original = try String(contentsOfFile: path, encoding: .utf8)
        var content = original
        
        // Apply each rule
        for rule in rules {
            content = try rule.apply(to: content, filePath: path)
        }
        
        if content == original {
            return .unchanged
        } else {
            return .formatted(content)
        }
    }
}

protocol FormatterRule {
    func apply(to content: String, filePath: String) throws -> String
}

struct TrailingWhitespaceRule: FormatterRule {
    let config: FormatterConfiguration
    
    func apply(to content: String, filePath: String) throws -> String {
        guard config.removeTrailingWhitespace else { return content }
        
        let lines = content.components(separatedBy: "\n")
        let trimmedLines = lines.map { $0.replacingOccurrences(of: #"\s+$"#, with: "", options: .regularExpression) }
        return trimmedLines.joined(separator: "\n")
    }
}

struct ImportSortingRule: FormatterRule {
    let config: FormatterConfiguration
    
    func apply(to content: String, filePath: String) throws -> String {
        guard config.sortImports else { return content }
        
        var lines = content.components(separatedBy: "\n")
        
        // หา import blocks
        var importBlocks: [(start: Int, end: Int, imports: [String])] = []
        var currentBlockStart: Int? = nil
        var currentImports: [String] = []
        
        for (index, line) in lines.enumerated() {
            let trimmed = line.trimmingCharacters(in: .whitespaces)
            
            if trimmed.hasPrefix("import ") {
                if currentBlockStart == nil {
                    currentBlockStart = index
                }
                currentImports.append(line)
            } else if !trimmed.isEmpty || currentBlockStart == nil {
                if let start = currentBlockStart {
                    importBlocks.append((
                        start: start,
                        end: index - 1,
                        imports: currentImports
                    ))
                    currentBlockStart = nil
                    currentImports = []
                }
            }
        }
        
        // Sort each import block
        for block in importBlocks.reversed() {
            let sortedImports = block.imports.sorted()
            lines.replaceSubrange(block.start...block.end, with: sortedImports)
        }
        
        return lines.joined(separator: "\n")
    }
}

struct TrailingNewlineRule: FormatterRule {
    let config: FormatterConfiguration
    
    func apply(to content: String, filePath: String) throws -> String {
        guard config.trailingNewline else { return content }
        
        var result = content
        while result.hasSuffix("\n\n") {
            result = String(result.dropLast())
        }
        if !result.hasSuffix("\n") {
            result += "\n"
        }
        return result
    }
}
```

### 12.6 Tests

```swift
// Tests/SwiftFormatterTests/FormatterTests.swift
import XCTest
@testable import SwiftFormatter

final class FormatterTests: XCTestCase {
    var formatter: SwiftCodeFormatter!
    var tempDir: URL!
    
    override func setUp() {
        super.setUp()
        formatter = SwiftCodeFormatter(configuration: .default)
        tempDir = FileManager.default.temporaryDirectory
            .appendingPathComponent(UUID().uuidString)
        try! FileManager.default.createDirectory(at: tempDir, withIntermediateDirectories: true)
    }
    
    override func tearDown() {
        try? FileManager.default.removeItem(at: tempDir)
        super.tearDown()
    }
    
    func testTrailingWhitespaceRemoval() async throws {
        let content = "let x = 1   \nlet y = 2  \n"
        let tempFile = tempDir.appendingPathComponent("test.swift")
        try content.write(to: tempFile, atomically: true, encoding: .utf8)
        
        let result = try await formatter.format(file: tempFile.path)
        
        if case .formatted(let formatted) = result {
            XCTAssertFalse(formatted.contains("   \n"))
            XCTAssertFalse(formatted.contains("  \n"))
        } else {
            XCTFail("ควรได้รับ formatted result")
        }
    }
    
    func testImportSorting() async throws {
        let content = """
        import UIKit
        import Foundation
        import SwiftUI
        import Combine
        
        struct MyView: View {
            var body: some View { EmptyView() }
        }
        """
        
        let tempFile = tempDir.appendingPathComponent("test.swift")
        try content.write(to: tempFile, atomically: true, encoding: .utf8)
        
        let result = try await formatter.format(file: tempFile.path)
        
        if case .formatted(let formatted) = result {
            let lines = formatted.components(separatedBy: "\n")
            let importLines = lines.filter { $0.hasPrefix("import ") }
            XCTAssertEqual(importLines, importLines.sorted())
        }
    }
    
    func testTrailingNewline() async throws {
        let content = "let x = 1"
        let tempFile = tempDir.appendingPathComponent("test.swift")
        try content.write(to: tempFile, atomically: true, encoding: .utf8)
        
        let result = try await formatter.format(file: tempFile.path)
        
        if case .formatted(let formatted) = result {
            XCTAssertTrue(formatted.hasSuffix("\n"))
        }
    }
    
    func testUnchangedFile() async throws {
        let content = "import Foundation\n\nlet x = 1\n"
        let tempFile = tempDir.appendingPathComponent("test.swift")
        try content.write(to: tempFile, atomically: true, encoding: .utf8)
        
        let result = try await formatter.format(file: tempFile.path)
        
        if case .unchanged = result {
            // ✅ ถูกต้อง
        } else {
            XCTFail("ควรได้รับ unchanged result")
        }
    }
}
```

---

## 13. แบบฝึกหัดพร้อมเฉลย

### แบบฝึกหัดที่ 1: สร้าง CLI Tool สำหรับ Count Lines of Code

```swift
// โจทย์: สร้าง CLI tool ที่นับจำนวน lines of code ใน Swift project
// โดยแยก: total lines, code lines, comment lines, blank lines

// เฉลย:
import ArgumentParser
import Foundation

@main
struct LOCCounter: ParsableCommand {
    static let configuration = CommandConfiguration(
        commandName: "loc",
        abstract: "นับ Lines of Code ใน Swift project"
    )
    
    @Argument(help: "Directory ที่ต้องการนับ")
    var directory: String = "."
    
    @Flag(name: .shortAndLong, help: "แสดงรายละเอียดแต่ละไฟล์")
    var detailed: Bool = false
    
    @Flag(help: "แสดงเฉพาะ summary")
    var summaryOnly: Bool = false
    
    mutating func run() throws {
        let counter = LineCounter()
        let result = try counter.count(directory: directory)
        
        if !summaryOnly {
            if detailed {
                // แสดงรายละเอียดแต่ละ file
                for fileResult in result.files.sorted(by: { $0.path < $1.path }) {
                    print("\n📄 \(fileResult.path)")
                    print("   Total: \(fileResult.total), Code: \(fileResult.code), Comment: \(fileResult.comment), Blank: \(fileResult.blank)")
                }
            }
        }
        
        // Summary
        print("\n=== Summary ===")
        print("Files: \(result.files.count)")
        print("Total lines:   \(result.totalLines)")
        print("Code lines:    \(result.codeLines) (\(percentage(result.codeLines, of: result.totalLines))%)")
        print("Comment lines: \(result.commentLines) (\(percentage(result.commentLines, of: result.totalLines))%)")
        print("Blank lines:   \(result.blankLines) (\(percentage(result.blankLines, of: result.totalLines))%)")
    }
    
    private func percentage(_ part: Int, of total: Int) -> Int {
        guard total > 0 else { return 0 }
        return Int(Double(part) / Double(total) * 100)
    }
}

struct FileResult {
    let path: String
    var total: Int = 0
    var code: Int = 0
    var comment: Int = 0
    var blank: Int = 0
}

struct CountResult {
    var files: [FileResult] = []
    var totalLines: Int { files.reduce(0) { $0 + $1.total } }
    var codeLines: Int { files.reduce(0) { $0 + $1.code } }
    var commentLines: Int { files.reduce(0) { $0 + $1.comment } }
    var blankLines: Int { files.reduce(0) { $0 + $1.blank } }
}

class LineCounter {
    func count(directory: String) throws -> CountResult {
        var result = CountResult()
        
        let fileManager = FileManager.default
        let dirURL = URL(fileURLWithPath: directory)
        
        guard let enumerator = fileManager.enumerator(
            at: dirURL,
            includingPropertiesForKeys: nil,
            options: [.skipsHiddenFiles]
        ) else {
            throw ValidationError("ไม่สามารถอ่าน directory: \(directory)")
        }
        
        for case let fileURL as URL in enumerator {
            guard fileURL.pathExtension == "swift" else { continue }
            
            let fileResult = try countLines(in: fileURL)
            result.files.append(fileResult)
        }
        
        return result
    }
    
    private func countLines(in url: URL) throws -> FileResult {
        let content = try String(contentsOf: url, encoding: .utf8)
        let lines = content.components(separatedBy: "\n")
        
        var result = FileResult(path: url.path)
        var inBlockComment = false
        
        for line in lines {
            result.total += 1
            let trimmed = line.trimmingCharacters(in: .whitespaces)
            
            if trimmed.isEmpty {
                result.blank += 1
            } else if inBlockComment {
                result.comment += 1
                if trimmed.contains("*/") {
                    inBlockComment = false
                }
            } else if trimmed.hasPrefix("//") {
                result.comment += 1
            } else if trimmed.hasPrefix("/*") {
                result.comment += 1
                if !trimmed.contains("*/") {
                    inBlockComment = true
                }
            } else {
                result.code += 1
            }
        }
        
        return result
    }
}
```

### แบบฝึกหัดที่ 2: SPM Plugin สำหรับ Generate Mock Objects

```swift
// โจทย์: สร้าง SPM Build Tool Plugin ที่ generate mock classes
// สำหรับ protocols ที่ annotated ด้วย // @GenerateMock

// MockGeneratorPlugin/plugin.swift
import PackagePlugin

@main
struct MockGeneratorPlugin: BuildToolPlugin {
    func createBuildCommands(
        context: PluginContext,
        target: Target
    ) async throws -> [Command] {
        let generator = try context.tool(named: "MockGenerator")
        
        guard let sourceTarget = target as? SourceModuleTarget else {
            return []
        }
        
        // หา Swift files ที่มี @GenerateMock annotation
        let swiftFiles = sourceTarget.sourceFiles
            .filter { $0.path.extension == "swift" }
        
        let outputDir = context.pluginWorkDirectory
            .appending("GeneratedMocks")
        
        return swiftFiles.map { file in
            let outputFile = outputDir.appending(
                file.path.stem + "Mock.swift"
            )
            
            return .buildCommand(
                displayName: "Generating mocks for \(file.path.lastComponent)",
                executable: generator.path,
                arguments: [
                    file.path.string,
                    "--output", outputFile.string
                ],
                inputFiles: [file.path],
                outputFiles: [outputFile]
            )
        }
    }
}

// MockGenerator/main.swift
import Foundation
import ArgumentParser
import SwiftSyntax
import SwiftSyntaxParser

@main
struct MockGenerator: ParsableCommand {
    @Argument var inputFile: String
    @Option(name: .shortAndLong) var output: String
    
    mutating func run() throws {
        let content = try String(contentsOfFile: inputFile, encoding: .utf8)
        
        guard content.contains("@GenerateMock") else { return }
        
        let syntax = try SyntaxParser.parse(source: content)
        let collector = ProtocolCollector()
        collector.walk(syntax)
        
        guard !collector.protocols.isEmpty else { return }
        
        var mockCode = "// Generated Mocks - DO NOT EDIT\n\n"
        for proto in collector.protocols {
            mockCode += generateMock(for: proto)
        }
        
        let outputURL = URL(fileURLWithPath: output)
        try FileManager.default.createDirectory(
            at: outputURL.deletingLastPathComponent(),
            withIntermediateDirectories: true
        )
        try mockCode.write(to: outputURL, atomically: true, encoding: .utf8)
    }
    
    private func generateMock(for proto: ProtocolInfo) -> String {
        var code = "class \(proto.name)Mock: \(proto.name) {\n"
        
        for method in proto.methods {
            code += "    var \(method.name)Called = false\n"
            code += "    var \(method.name)CallCount = 0\n"
            
            if !method.returnType.isEmpty && method.returnType != "Void" {
                code += "    var \(method.name)ReturnValue: \(method.returnType)!\n"
            }
            
            code += "\n"
            code += "    func \(method.name)(\(method.parameters)) \(method.returnType.isEmpty ? "" : "-> \(method.returnType)") {\n"
            code += "        \(method.name)Called = true\n"
            code += "        \(method.name)CallCount += 1\n"
            
            if !method.returnType.isEmpty && method.returnType != "Void" {
                code += "        return \(method.name)ReturnValue\n"
            }
            
            code += "    }\n\n"
        }
        
        code += "}\n\n"
        return code
    }
}

struct ProtocolInfo {
    let name: String
    let methods: [MethodInfo]
}

struct MethodInfo {
    let name: String
    let parameters: String
    let returnType: String
}

class ProtocolCollector: SyntaxVisitor {
    var protocols: [ProtocolInfo] = []
    private var shouldGenerate = false
    
    init() {
        super.init(viewMode: .sourceAccurate)
    }
    
    override func visit(_ node: ProtocolDeclSyntax) -> SyntaxVisitorContinueKind {
        // ตรวจสอบ leading trivia หา @GenerateMock comment
        let leadingTrivia = node.leadingTrivia?.description ?? ""
        shouldGenerate = leadingTrivia.contains("@GenerateMock")
        return .visitChildren
    }
    
    override func visitPost(_ node: ProtocolDeclSyntax) {
        guard shouldGenerate else {
            shouldGenerate = false
            return
        }
        
        let name = node.name.text
        var methods: [MethodInfo] = []
        
        for member in node.memberBlock.members {
            if let funcDecl = member.decl.as(FunctionDeclSyntax.self) {
                let methodName = funcDecl.name.text
                let params = funcDecl.signature.parameterClause.parameters.description
                let returnType = funcDecl.signature.returnClause?.type.description ?? ""
                
                methods.append(MethodInfo(
                    name: methodName,
                    parameters: params,
                    returnType: returnType.trimmingCharacters(in: .whitespaces)
                ))
            }
        }
        
        protocols.append(ProtocolInfo(name: name, methods: methods))
        shouldGenerate = false
    }
}
```

### แบบฝึกหัดที่ 3: Xcode Source Editor Extension สำหรับ JSON to Swift

```swift
// JSONToSwiftCommand.swift
import Foundation
import XcodeKit

class JSONToSwiftCommand: NSObject, XCSourceEditorCommand {
    func perform(
        with invocation: XCSourceEditorCommandInvocation,
        completionHandler: @escaping (Error?) -> Void
    ) {
        let buffer = invocation.buffer
        
        // ดึง selected text
        guard let selection = buffer.selections.firstObject as? XCSourceTextRange else {
            completionHandler(
                NSError(domain: "JSONToSwift", code: 1,
                        userInfo: [NSLocalizedDescriptionKey: "ไม่มี selection"])
            )
            return
        }
        
        // เก็บ selected lines
        let lines = buffer.lines as! [String]
        let startLine = selection.start.line
        let endLine = selection.end.line
        
        let selectedText = lines[startLine...endLine]
            .joined()
            .trimmingCharacters(in: .whitespacesAndNewlines)
        
        // Parse JSON
        guard let data = selectedText.data(using: .utf8),
              let json = try? JSONSerialization.jsonObject(with: data) as? [String: Any] else {
            completionHandler(
                NSError(domain: "JSONToSwift", code: 2,
                        userInfo: [NSLocalizedDescriptionKey: "JSON ไม่ถูกต้อง"])
            )
            return
        }
        
        // Generate Swift struct
        let swiftCode = generateSwiftStruct(from: json, name: "GeneratedModel")
        
        // Replace selection กับ generated code
        let swiftLines = swiftCode.components(separatedBy: "\n").map { $0 + "\n" }
        buffer.lines.removeObjects(in: NSRange(location: startLine, length: endLine - startLine + 1))
        
        for (index, line) in swiftLines.enumerated() {
            buffer.lines.insert(line, at: startLine + index)
        }
        
        completionHandler(nil)
    }
    
    private func generateSwiftStruct(from json: [String: Any], name: String) -> String {
        var code = "struct \(name): Codable {\n"
        
        for (key, value) in json.sorted(by: { $0.key < $1.key }) {
            let type = swiftType(for: value)
            code += "    let \(key): \(type)\n"
        }
        
        code += "}"
        return code
    }
    
    private func swiftType(for value: Any) -> String {
        switch value {
        case is String: return "String"
        case is Int: return "Int"
        case is Double: return "Double"
        case is Bool: return "Bool"
        case is [Any]: return "[Any]"
        case is [String: Any]: return "[String: Any]"
        default: return "Any"
        }
    }
}
```

---

## สรุป

ในบทนี้เราได้เรียนรู้:

1. **Command-line tools** ด้วย Swift ArgumentParser - การสร้าง CLI tools ที่มีประสิทธิภาพพร้อม subcommands, validation, และ async support

2. **Xcode Source Editor Extensions** - การ manipulate source code ใน Xcode editor ผ่าน XcodeKit framework

3. **SPM Build Tool Plugins** - ทั้ง BuildToolPlugin และ CommandPlugin สำหรับ automate งานใน build process

4. **Swift Macros** - การสร้าง custom macros เพื่อ reduce boilerplate code

5. **Code Analysis** - การใช้ SwiftSyntax สำหรับ static analysis และ custom linting rules

6. **Developer Productivity Tools** - ตัวอย่างจริงของ tools ที่มีประโยชน์ในการทำงาน

7. **Publishing Tools** - การ distribute tools ผ่าน Homebrew, Mint, และ Swift Package Index

ทักษะเหล่านี้ช่วยให้คุณสามารถสร้างเครื่องมือที่ช่วยเพิ่มประสิทธิภาพทีมและทำให้ workflow ของการพัฒนา iOS/macOS apps เป็นอัตโนมัติได้อย่างมีประสิทธิภาพ

---

*จบ Part 89: Creating Developer Tools*
