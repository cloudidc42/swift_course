# Part 92: Protocol-Oriented Programming (POP) Mastery

## บทนำ

Protocol-Oriented Programming (POP) คือแนวทางการเขียนโปรแกรมที่ Apple แนะนำอย่างเป็นทางการในงาน WWDC 2015 ผ่านการบรรยายที่โด่งดัง "Protocol-Oriented Programming in Swift" โดย Dave Abrahams POP เปลี่ยนมุมมองจากการคิดในแง่ class hierarchy มาเป็นการคิดในแง่ capabilities และ behaviors ที่กำหนดผ่าน protocols

---

## 1. POP Philosophy

### 1.1 "Start with a Protocol" Mantra

ใน Object-Oriented Programming เราเริ่มต้นด้วยการถามว่า "นี่คืออะไร?" (สิ่งนี้เป็น Animal หรือ Vehicle)
ใน Protocol-Oriented Programming เราถามว่า "สิ่งนี้ทำอะไรได้?" (สิ่งนี้ Drawable หรือ Serializable หรือเปล่า)

```swift
// แนวทาง OOP: เริ่มจาก base class
class Animal {
    var name: String
    var sound: String
    
    init(name: String, sound: String) {
        self.name = name
        self.sound = sound
    }
    
    func makeSound() -> String {
        return "\(name) says \(sound)"
    }
}

class Dog: Animal {
    init(name: String) {
        super.init(name: name, sound: "Woof")
    }
    
    func fetch() {
        print("\(name) is fetching!")
    }
}

// แนวทาง POP: เริ่มจาก protocol
protocol Identifiable {
    var id: UUID { get }
}

protocol Named {
    var name: String { get }
}

protocol SoundMaking {
    func makeSound() -> String
}

protocol Fetchable {
    func fetch()
}

// สร้าง value types ที่ conform protocols
struct Dog: Named, SoundMaking, Fetchable, Identifiable {
    let id = UUID()
    let name: String
    
    func makeSound() -> String { "\(name) says Woof!" }
    func fetch() { print("\(name) กำลัง fetch!") }
}

struct Cat: Named, SoundMaking, Identifiable {
    let id = UUID()
    let name: String
    
    func makeSound() -> String { "\(name) says Meow!" }
}

// ใช้ protocol composition
func introduce(_ thing: Named & SoundMaking) {
    print("\(thing.name): \(thing.makeSound())")
}

let dog = Dog(name: "Buddy")
let cat = Cat(name: "Whiskers")

introduce(dog)
introduce(cat)
```

### 1.2 Value Types + Protocols vs Class Hierarchies

```swift
// ปัญหาของ class hierarchy: The "Blob" problem
// เมื่อ class hierarchy ลึกขึ้น มันซับซ้อนขึ้นเรื่อยๆ

// ❌ Class hierarchy ที่มีปัญหา
class Vehicle {
    var speed: Double = 0
    func accelerate() { speed += 10 }
}

class LandVehicle: Vehicle {
    var numberOfWheels: Int = 4
}

class WaterVehicle: Vehicle {
    func submerge() { print("กำลังดำน้ำ") }
}

// ปัญหา: จะสร้าง AmphibiousVehicle อย่างไร?
// ไม่สามารถ inherit จากทั้ง LandVehicle และ WaterVehicle

// ✅ POP approach
protocol Drivable {
    var speed: Double { get set }
    mutating func accelerate()
}

protocol LandCapable {
    var numberOfWheels: Int { get }
}

protocol WaterCapable {
    mutating func submerge()
}

// Default implementations
extension Drivable {
    mutating func accelerate() {
        speed += 10
    }
}

// Amphibious vehicle ใช้ได้ทั้ง land และ water
struct AmphibiousVehicle: Drivable, LandCapable, WaterCapable {
    var speed: Double = 0
    let numberOfWheels = 4
    var isSubmerged = false
    
    mutating func submerge() {
        isSubmerged = true
        print("กำลังดำน้ำ")
    }
}

// Value type benefits: ไม่มี shared state
var vehicle1 = AmphibiousVehicle()
var vehicle2 = vehicle1  // copy, ไม่ใช่ reference

vehicle1.accelerate()
print("vehicle1 speed: \(vehicle1.speed)")  // 10
print("vehicle2 speed: \(vehicle2.speed)")  // 0 - ไม่ได้รับผลกระทบ
```

### 1.3 Protocol Composition vs Inheritance

```swift
// Protocol Composition - ประกอบหลาย protocols เข้าด้วยกัน
// ยืดหยุ่นกว่า inheritance เพราะไม่มีลำดับชั้น

protocol Readable {
    func read() -> String
}

protocol Writable {
    mutating func write(_ content: String)
}

protocol Persistable {
    func save() -> Bool
    mutating func load() -> Bool
}

// Composition ผ่าน typealias
typealias ReadWritable = Readable & Writable
typealias FullStorage = Readable & Writable & Persistable

// Struct ที่ implement ReadWritable
struct InMemoryStorage: ReadWritable {
    private var content: String = ""
    
    func read() -> String { content }
    mutating func write(_ content: String) { self.content = content }
}

// Struct ที่ implement FullStorage
struct FileStorage: FullStorage {
    private var content: String = ""
    private let filename: String
    
    init(filename: String) {
        self.filename = filename
    }
    
    func read() -> String { content }
    
    mutating func write(_ content: String) {
        self.content = content
    }
    
    func save() -> Bool {
        // บันทึกลงไฟล์
        print("บันทึก \(filename)")
        return true
    }
    
    mutating func load() -> Bool {
        // โหลดจากไฟล์
        print("โหลด \(filename)")
        return true
    }
}

// Function ที่รับ protocol composition
func processStorage(_ storage: FullStorage) {
    print("อ่าน: \(storage.read())")
    _ = storage.save()
}
```

---

## 2. Protocol Design Principles

### 2.1 Interface Segregation (Small, Focused Protocols)

```swift
// Interface Segregation Principle: ควรมีหลาย protocol เล็กๆ
// แทนที่จะมี protocol ใหญ่ๆ ที่ทำหลายอย่าง

// ❌ Protocol ที่ใหญ่เกินไป (Fat Protocol)
protocol AllInOneMedia {
    func play()
    func pause()
    func stop()
    func record()
    func share()
    func edit()
    func export()
    var duration: Double { get }
    var title: String { get }
    var author: String { get }
}

// ✅ แยกเป็น protocol เล็กๆ ที่เน้นเรื่องเดียว
protocol Playable {
    func play()
    func pause()
    func stop()
    var duration: Double { get }
}

protocol Recordable {
    func record()
    func stopRecording()
}

protocol Shareable {
    func share()
    func shareURL() -> URL?
}

protocol Editable {
    func edit()
    func export()
}

protocol MediaMetadata {
    var title: String { get }
    var author: String { get }
    var duration: Double { get }
}

// Types ที่ implement เฉพาะสิ่งที่ต้องการ
struct AudioPlayer: Playable, MediaMetadata {
    var duration: Double = 0
    var title: String = ""
    var author: String = ""
    
    func play() { print("กำลังเล่น audio") }
    func pause() { print("หยุดชั่วคราว") }
    func stop() { print("หยุด") }
}

struct VideoRecorder: Recordable, Shareable {
    func record() { print("กำลัง record") }
    func stopRecording() { print("หยุด record") }
    func share() { print("กำลัง share") }
    func shareURL() -> URL? { return nil }
}

// Full-featured media player implement หลาย protocols
struct FullMediaPlayer: Playable, Recordable, Shareable, Editable, MediaMetadata {
    var duration: Double = 0
    var title: String = ""
    var author: String = ""
    
    func play() { print("เล่น") }
    func pause() { print("หยุดชั่วคราว") }
    func stop() { print("หยุด") }
    func record() { print("บันทึก") }
    func stopRecording() { print("หยุดบันทึก") }
    func share() { print("แชร์") }
    func shareURL() -> URL? { nil }
    func edit() { print("แก้ไข") }
    func export() { print("ส่งออก") }
}
```

### 2.2 Protocol Naming Conventions

```swift
// Convention 1: "-able" suffix - สิ่งที่ type สามารถทำได้
protocol Comparable {}        // สามารถเปรียบเทียบได้
protocol Encodable {}         // สามารถ encode ได้
protocol Decodable {}         // สามารถ decode ได้
protocol Hashable {}          // สามารถ hash ได้
protocol Equatable {}         // สามารถเปรียบเทียบความเท่ากันได้
protocol Identifiable {}      // มี identity
protocol Sendable {}          // ส่งข้ามแกนได้

// Convention 2: "-ing" suffix - action/behavior
protocol Logging {
    func log(_ message: String)
}

protocol Networking {
    func request(url: URL) async throws -> Data
}

protocol Caching {
    func cache<T>(key: String, value: T)
    func cached<T>(key: String) -> T?
}

// Convention 3: "-Provider" suffix - สิ่งที่ provide ข้อมูล
protocol DataProvider {
    associatedtype DataType
    func provideData() -> [DataType]
}

protocol AuthProvider {
    var currentUser: User? { get }
    func authenticate(credentials: Credentials) async throws -> User
    func signOut() async throws
}

protocol ThemeProvider {
    var primaryColor: Color { get }
    var secondaryColor: Color { get }
    var typography: Typography { get }
}

// Convention 4: Nouns - สำหรับ type constraints
protocol Collection {}        // เป็น collection
protocol Sequence {}          // เป็น sequence
protocol Iterator {}          // เป็น iterator

// Real-world example
protocol ImageLoader {
    func loadImage(from url: URL) async throws -> UIImage
    func cachedImage(for url: URL) -> UIImage?
}

protocol ImageCacheable {
    mutating func cache(_ image: UIImage, for url: URL)
    mutating func evict(url: URL)
    mutating func clearCache()
}

struct RemoteImageLoader: ImageLoader, ImageCacheable {
    private var cache: [URL: UIImage] = [:]
    
    func loadImage(from url: URL) async throws -> UIImage {
        if let cached = cachedImage(for: url) {
            return cached
        }
        // Download image
        let (data, _) = try await URLSession.shared.data(from: url)
        guard let image = UIImage(data: data) else {
            throw URLError(.cannotDecodeContentData)
        }
        return image
    }
    
    func cachedImage(for url: URL) -> UIImage? {
        return cache[url]
    }
    
    mutating func cache(_ image: UIImage, for url: URL) {
        cache[url] = image
    }
    
    mutating func evict(url: URL) {
        cache.removeValue(forKey: url)
    }
    
    mutating func clearCache() {
        cache.removeAll()
    }
}

// placeholder types
struct Color {}
struct Typography {}
struct Credentials {}
```

### 2.3 Retroactive Conformance Patterns

```swift
// Retroactive conformance: เพิ่ม protocol conformance ให้กับ type ที่ไม่ใช่ของเรา

import Foundation

// เพิ่ม protocol ให้กับ standard library type
protocol Serializable {
    func serialize() -> String
}

// Retroactive conformance สำหรับ Int
extension Int: Serializable {
    func serialize() -> String {
        return String(self)
    }
}

// Retroactive conformance สำหรับ Array
extension Array: Serializable where Element: Serializable {
    func serialize() -> String {
        return "[" + map { $0.serialize() }.joined(separator: ",") + "]"
    }
}

// Retroactive conformance สำหรับ third-party type
// สมมติว่า CLLocationCoordinate2D มาจาก framework อื่น
import CoreLocation

extension CLLocationCoordinate2D: Equatable {
    public static func == (lhs: CLLocationCoordinate2D, rhs: CLLocationCoordinate2D) -> Bool {
        return lhs.latitude == rhs.latitude && lhs.longitude == rhs.longitude
    }
}

extension CLLocationCoordinate2D: CustomStringConvertible {
    public var description: String {
        return "(\(latitude), \(longitude))"
    }
}

// เพิ่ม Codable ให้กับ CLLocationCoordinate2D
extension CLLocationCoordinate2D: Codable {
    enum CodingKeys: String, CodingKey {
        case latitude, longitude
    }
    
    public init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        let lat = try container.decode(Double.self, forKey: .latitude)
        let lon = try container.decode(Double.self, forKey: .longitude)
        self.init(latitude: lat, longitude: lon)
    }
    
    public func encode(to encoder: Encoder) throws {
        var container = encoder.container(keyedBy: CodingKeys.self)
        try container.encode(latitude, forKey: .latitude)
        try container.encode(longitude, forKey: .longitude)
    }
}
```

---

## 3. Generic Programming

### 3.1 Constrained Generics

```swift
// Constrained generics ใช้ where clause หรือ : เพื่อจำกัด type parameter

import Foundation

// Basic constraint
func findMax<T: Comparable>(_ array: [T]) -> T? {
    return array.max()
}

// Multiple constraints
func uniqueSorted<T: Comparable & Hashable>(_ array: [T]) -> [T] {
    return Array(Set(array)).sorted()
}

// Where clause constraints
func findDuplicates<T>(_ array: [T]) -> [T] where T: Hashable & Equatable {
    var seen = Set<T>()
    var duplicates: [T] = []
    
    for item in array {
        if !seen.insert(item).inserted {
            if !duplicates.contains(item) {
                duplicates.append(item)
            }
        }
    }
    return duplicates
}

// Complex constraints
struct SortedArray<T: Comparable> {
    private var storage: [T] = []
    
    mutating func insert(_ element: T) {
        let index = storage.firstIndex { $0 > element } ?? storage.endIndex
        storage.insert(element, at: index)
    }
    
    func contains(_ element: T) -> Bool {
        // Binary search
        var low = 0
        var high = storage.count - 1
        
        while low <= high {
            let mid = (low + high) / 2
            if storage[mid] == element {
                return true
            } else if storage[mid] < element {
                low = mid + 1
            } else {
                high = mid - 1
            }
        }
        return false
    }
    
    var elements: [T] { storage }
}

// Generic function กับ protocol constraint ที่ซับซ้อน
func merge<C1: Collection, C2: Collection>(
    _ first: C1,
    _ second: C2
) -> [C1.Element] where C1.Element == C2.Element, C1.Element: Comparable {
    return (Array(first) + Array(second)).sorted()
}

// การใช้งาน
var sortedInts = SortedArray<Int>()
sortedInts.insert(5)
sortedInts.insert(2)
sortedInts.insert(8)
sortedInts.insert(1)
print(sortedInts.elements)  // [1, 2, 5, 8]

let merged = merge([3, 1, 5], [4, 2, 6])
print(merged)  // [1, 2, 3, 4, 5, 6]
```

### 3.2 Primary Associated Types

```swift
// Swift 5.7+: Primary associated types สำหรับ some/any keywords

// Protocol กับ primary associated type
protocol Sequence<Element> {
    associatedtype Element
    func makeIterator() -> some IteratorProtocol
}

// ใช้ some Collection<Int> แทน Collection where Element == Int
func sumOf(_ numbers: some Collection<Int>) -> Int {
    return numbers.reduce(0, +)
}

// เปรียบเทียบ syntax
// เก่า:
func process<C: Collection>(_ collection: C) where C.Element: Numeric {}

// ใหม่ (Swift 5.7+):
func process(_ collection: some Collection<some Numeric>) {}

// ตัวอย่างจริง
protocol Repository<Model> {
    associatedtype Model: Identifiable
    
    func findAll() async throws -> [Model]
    func findById(_ id: Model.ID) async throws -> Model?
    func save(_ model: Model) async throws
    func delete(_ id: Model.ID) async throws
}

struct TodoRepository: Repository {
    typealias Model = Todo
    
    private var storage: [UUID: Todo] = [:]
    
    func findAll() async throws -> [Todo] {
        return Array(storage.values)
    }
    
    func findById(_ id: UUID) async throws -> Todo? {
        return storage[id]
    }
    
    func save(_ model: Todo) async throws {
        storage[model.id] = model
    }
    
    func delete(_ id: UUID) async throws {
        storage.removeValue(forKey: id)
    }
}

struct Todo: Identifiable {
    let id: UUID
    var title: String
    var isCompleted: Bool
    let createdAt: Date
    
    init(title: String) {
        id = UUID()
        self.title = title
        isCompleted = false
        createdAt = Date()
    }
}
```

### 3.3 Type Erasure

```swift
// Type Erasure: ซ่อน concrete type ด้วย wrapper

// ปัญหา: Protocol กับ associated type ไม่สามารถใช้เป็น existential ได้โดยตรง
protocol Animatable {
    associatedtype AnimationState
    func animate(to state: AnimationState, duration: TimeInterval)
    var currentState: AnimationState { get }
}

// ❌ ไม่สามารถทำแบบนี้ได้ใน Swift เก่า
// var animation: any Animatable  // error ถ้า associated type ต้องถูก infer

// ✅ Type Erasure wrapper
class AnyAnimatable<State>: Animatable {
    typealias AnimationState = State
    
    private let _animate: (State, TimeInterval) -> Void
    private let _currentState: () -> State
    
    init<A: Animatable>(_ animatable: A) where A.AnimationState == State {
        _animate = { state, duration in
            animatable.animate(to: state, duration: duration)
        }
        _currentState = { animatable.currentState }
    }
    
    func animate(to state: State, duration: TimeInterval) {
        _animate(state, duration)
    }
    
    var currentState: State {
        _currentState()
    }
}

// ตัวอย่างจริง: AnyPublisher คือ type erasure ของ Publisher
import Combine

struct MyPublisher: Publisher {
    typealias Output = Int
    typealias Failure = Never
    
    func receive<S>(subscriber: S) where S : Subscriber, Never == S.Failure, Int == S.Input {
        // implementation
    }
}

// eraseToAnyPublisher() ทำ type erasure
let myPub = MyPublisher()
let erased: AnyPublisher<Int, Never> = myPub.eraseToAnyPublisher()

// สร้าง type-erased wrapper สำหรับ custom protocol
protocol DataSource {
    associatedtype Item
    var items: [Item] { get }
    func item(at index: Int) -> Item
    var count: Int { get }
}

struct AnyDataSource<T>: DataSource {
    typealias Item = T
    
    private let _items: () -> [T]
    private let _itemAt: (Int) -> T
    private let _count: () -> Int
    
    init<DS: DataSource>(_ dataSource: DS) where DS.Item == T {
        _items = { dataSource.items }
        _itemAt = { dataSource.item(at: $0) }
        _count = { dataSource.count }
    }
    
    var items: [T] { _items() }
    func item(at index: Int) -> T { _itemAt(index) }
    var count: Int { _count() }
}

// Concrete implementation
struct ArrayDataSource<T>: DataSource {
    private let storage: [T]
    
    init(_ items: [T]) {
        self.storage = items
    }
    
    var items: [T] { storage }
    func item(at index: Int) -> T { storage[index] }
    var count: Int { storage.count }
}

// การใช้งาน
let source = ArrayDataSource([1, 2, 3, 4, 5])
let erased2 = AnyDataSource(source)
print("Items: \(erased2.items)")
```

### 3.4 Opaque Return Types

```swift
import SwiftUI

// Opaque return types: some keyword ซ่อน concrete type แต่คงไว้ซึ่ง type safety

// ✅ some View - compiler รู้ concrete type
struct ContentView: View {
    var body: some View {
        VStack {
            Text("Hello, World!")
            Button("Tap me") { }
        }
    }
}

// ต่างกับ any View ที่เป็น existential
func makeView(showButton: Bool) -> some View {
    // ต้อง return ชนิดเดียวกันทุก path
    Group {
        Text("Hello")
        if showButton {
            Button("OK") { }
        }
    }
}

// Opaque types กับ protocol
protocol Shape {
    func area() -> Double
}

struct Circle: Shape {
    let radius: Double
    func area() -> Double { .pi * radius * radius }
}

struct Square: Shape {
    let side: Double
    func area() -> Double { side * side }
}

// some Shape - compiler รู้ว่าเป็น concrete type เดียวกันเสมอ
func makeDefaultShape() -> some Shape {
    Circle(radius: 5.0)
}

// ใช้ Opaque type กับ Combine
import Combine

protocol DataFetcher {
    func fetchData() -> some Publisher<Data, Error>
}

struct APIFetcher: DataFetcher {
    let url: URL
    
    func fetchData() -> some Publisher<Data, Error> {
        URLSession.shared.dataTaskPublisher(for: url)
            .map(\.data)
            .mapError { $0 as Error }
    }
}
```

---

## 4. Protocol Extensions Power

### 4.1 Default Implementations

```swift
// Default implementations - ให้ implementations ที่ types สามารถ override หรือใช้ default ได้

protocol Greetable {
    var name: String { get }
    func greet() -> String
    func greetFormally() -> String
}

// Default implementation
extension Greetable {
    func greet() -> String {
        return "สวัสดี, \(name)!"
    }
    
    func greetFormally() -> String {
        return "สวัสดีครับ/ค่ะ คุณ\(name)"
    }
}

struct Person: Greetable {
    let name: String
    // ใช้ default implementation ของ greet() และ greetFormally()
}

struct Robot: Greetable {
    let name: String
    
    // Override default implementation
    func greet() -> String {
        return "GREETING INITIALIZED FOR: \(name.uppercased())"
    }
    // ใช้ default greetFormally()
}

let person = Person(name: "สมชาย")
let robot = Robot(name: "R2D2")

print(person.greet())         // สวัสดี, สมชาย!
print(robot.greet())          // GREETING INITIALIZED FOR: R2D2
print(person.greetFormally()) // สวัสดีครับ/ค่ะ คุณสมชาย
print(robot.greetFormally())  // สวัสดีครับ/ค่ะ คุณR2D2

// Computed property ใน extension
protocol Area {
    var width: Double { get }
    var height: Double { get }
}

extension Area {
    var area: Double { width * height }
    var perimeter: Double { 2 * (width + height) }
    var isSquare: Bool { width == height }
    var diagonal: Double { (width * width + height * height).squareRoot() }
}

struct Rectangle: Area {
    let width: Double
    let height: Double
}

let rect = Rectangle(width: 10, height: 5)
print("Area: \(rect.area)")           // 50.0
print("Perimeter: \(rect.perimeter)") // 30.0
print("Diagonal: \(rect.diagonal)")   // ~11.18
```

### 4.2 Conditional Extensions

```swift
// Conditional extensions: เพิ่ม functionality เมื่อ generic parameter ตรงตาม constraint

import Foundation

// Extension ที่ทำงานเมื่อ Element เป็น Numeric
extension Array where Element: Numeric {
    func sum() -> Element {
        return reduce(0, +)
    }
    
    func product() -> Element {
        return reduce(1, *)
    }
}

extension Array where Element: BinaryFloatingPoint {
    func average() -> Element {
        guard !isEmpty else { return 0 }
        return sum() / Element(count)
    }
    
    func standardDeviation() -> Element {
        guard count > 1 else { return 0 }
        let avg = average()
        let variance = map { pow($0 - avg, 2) }.average()
        return variance.squareRoot()
    }
}

// Extension ที่ทำงานเมื่อ Element เป็น Equatable
extension Array where Element: Equatable {
    func removeDuplicates() -> [Element] {
        var result: [Element] = []
        for item in self {
            if !result.contains(item) {
                result.append(item)
            }
        }
        return result
    }
    
    func count(of element: Element) -> Int {
        return filter { $0 == element }.count
    }
}

// Extension ที่ทำงานเมื่อ Element เป็น Comparable
extension Array where Element: Comparable {
    func isSorted() -> Bool {
        guard count > 1 else { return true }
        return zip(self, dropFirst()).allSatisfy { $0 <= $1 }
    }
    
    func median() -> Element? {
        guard !isEmpty else { return nil }
        let sorted = self.sorted()
        let mid = sorted.count / 2
        return sorted[mid]
    }
}

// การใช้งาน
let numbers = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3]
print("Sum: \(numbers.sum())")                           // 39
print("Average: \([1.0, 2.0, 3.0, 4.0, 5.0].average())")  // 3.0
print("Duplicates removed: \(numbers.removeDuplicates())")
print("Is sorted: \(numbers.isSorted())")               // false

// Conditional conformance
extension Optional: CustomStringConvertible where Wrapped: CustomStringConvertible {
    public var description: String {
        switch self {
        case .none:
            return "nil"
        case .some(let value):
            return "Optional(\(value.description))"
        }
    }
}
```

### 4.3 Extension with where Clause

```swift
// Extension ที่มี where clause สำหรับ protocol

protocol Container {
    associatedtype Element
    var elements: [Element] { get }
}

// Extension เมื่อ Element เป็น Equatable
extension Container where Element: Equatable {
    func contains(_ element: Element) -> Bool {
        return elements.contains(element)
    }
    
    func index(of element: Element) -> Int? {
        return elements.firstIndex(of: element)
    }
}

// Extension เมื่อ Element เป็น Comparable
extension Container where Element: Comparable {
    var min: Element? { elements.min() }
    var max: Element? { elements.max() }
    var sorted: [Element] { elements.sorted() }
}

// Extension เมื่อ Element เป็น String
extension Container where Element == String {
    func joined(separator: String = "") -> String {
        return elements.joined(separator: separator)
    }
    
    func lowercased() -> [String] {
        return elements.map { $0.lowercased() }
    }
}

// Implementation
struct Bag<T>: Container {
    var elements: [T]
    
    init(_ elements: T...) {
        self.elements = elements
    }
}

let intBag = Bag(3, 1, 4, 1, 5, 9, 2, 6)
print("Min: \(intBag.min!)")         // 1
print("Max: \(intBag.max!)")         // 9
print("Sorted: \(intBag.sorted)")    // [1, 1, 2, 3, 4, 5, 6, 9]
print("Contains 5: \(intBag.contains(5))")  // true

let stringBag = Bag("Hello", "World", "Swift")
print("Joined: \(stringBag.joined(separator: " "))")  // Hello World Swift
print("Lowercased: \(stringBag.lowercased())")
```

### 4.4 Diamond Problem Resolution

```swift
// Diamond problem: เมื่อ type inherit behavior จาก protocols หลายตัว
// ที่มี default implementation เหมือนกัน

protocol Flyable {
    func move()
}

protocol Swimmable {
    func move()
}

extension Flyable {
    func move() { print("บิน") }
}

extension Swimmable {
    func move() { print("ว่ายน้ำ") }
}

// ❌ Ambiguity error
// struct Duck: Flyable, Swimmable {
//     // move() ไม่ชัดเจนว่าใช้ implementation ไหน
// }

// ✅ Resolve ด้วยการ implement เอง
struct Duck: Flyable, Swimmable {
    func move() {
        // เลือกว่าจะทำอะไร หรือทำทั้งสอง
        print("บินและว่ายน้ำ")
    }
    
    // หรือ delegate ไปยัง protocol เฉพาะ
    func fly() {
        (self as Flyable).move()
    }
    
    func swim() {
        (self as Swimmable).move()
    }
}

// Best practice: ใช้ protocol hierarchy เพื่อหลีกเลี่ยง diamond problem
protocol Movable {
    func move()
}

protocol Aerial: Movable {
    func altitude() -> Double
}

protocol Aquatic: Movable {
    func depth() -> Double
}

extension Aerial {
    func move() { print("เคลื่อนที่ทางอากาศ") }
    func altitude() -> Double { 100.0 }
}

extension Aquatic {
    func move() { print("เคลื่อนที่ในน้ำ") }
    func depth() -> Double { -10.0 }
}

struct FlyingFish: Aerial, Aquatic {
    func move() {
        print("เคลื่อนที่ได้ทั้งในอากาศและน้ำ")
    }
    // ต้อง implement ทั้ง altitude() และ depth() หรือใช้ default
}
```

---

## 5. Associated Types Mastery

### 5.1 PAT (Protocols with Associated Types)

```swift
// PAT: Protocol ที่มี associatedtype เพิ่ม flexibility และ type safety

protocol Stack {
    associatedtype Element
    
    var isEmpty: Bool { get }
    var count: Int { get }
    
    mutating func push(_ element: Element)
    mutating func pop() -> Element?
    func peek() -> Element?
}

// Default implementation
extension Stack {
    var isEmpty: Bool { count == 0 }
}

// Concrete implementation
struct ArrayStack<T>: Stack {
    typealias Element = T
    
    private var storage: [T] = []
    
    var count: Int { storage.count }
    
    mutating func push(_ element: T) {
        storage.append(element)
    }
    
    mutating func pop() -> T? {
        return storage.popLast()
    }
    
    func peek() -> T? {
        return storage.last
    }
}

// Function ที่ทำงานกับ Stack protocol
func printAll<S: Stack>(_ stack: S) where S.Element: CustomStringConvertible {
    var copy = stack
    while let element = copy.pop() {
        print(element.description)
    }
}

// PAT กับ multiple associated types
protocol Transformer {
    associatedtype Input
    associatedtype Output
    
    func transform(_ input: Input) -> Output
}

struct DoubleTransformer: Transformer {
    func transform(_ input: Int) -> Double {
        return Double(input)
    }
}

struct StringTransformer: Transformer {
    func transform(_ input: Int) -> String {
        return "Value: \(input)"
    }
}

// Chain transformers
struct ChainedTransformer<A: Transformer, B: Transformer>: Transformer
    where A.Output == B.Input {
    
    typealias Input = A.Input
    typealias Output = B.Output
    
    let first: A
    let second: B
    
    func transform(_ input: A.Input) -> B.Output {
        return second.transform(first.transform(input))
    }
}

// การใช้งาน
let doubler = DoubleTransformer()
let stringer = StringTransformer()

// Chain: Int -> Double -> String
// หมายเหตุ: ต้อง adjust type ให้ตรง
let result = doubler.transform(42)  // 42.0
print("Result: \(result)")
```

### 5.2 Self Requirements

```swift
// Self requirement: protocol ที่ใช้ Self เป็น return type หรือ parameter

protocol Copyable {
    func copy() -> Self
}

protocol Scalable {
    func scaled(by factor: Double) -> Self
}

// Self requirement ใน method
protocol Builder {
    func build() -> Self
    func with(_ configuration: (inout Self) -> Void) -> Self
}

extension Builder {
    func with(_ configuration: (inout Self) -> Void) -> Self {
        var copy = self
        configuration(&copy)
        return copy
    }
}

struct ButtonConfig: Builder, Copyable {
    var title: String = ""
    var backgroundColor: String = "blue"
    var textColor: String = "white"
    var cornerRadius: Double = 8
    var isEnabled: Bool = true
    
    func build() -> ButtonConfig { self }
    
    func copy() -> ButtonConfig { self }
}

// Fluent interface ด้วย Self requirement
let config = ButtonConfig()
    .with { $0.title = "สั่งซื้อ" }
    .with { $0.backgroundColor = "green" }
    .with { $0.cornerRadius = 12 }

print("Button: \(config.title), color: \(config.backgroundColor)")

// Comparable protocol ใช้ Self requirement
protocol Rankable: Comparable {
    var rank: Int { get }
}

extension Rankable {
    static func < (lhs: Self, rhs: Self) -> Bool {
        return lhs.rank < rhs.rank
    }
    
    static func == (lhs: Self, rhs: Self) -> Bool {
        return lhs.rank == rhs.rank
    }
}

struct Player: Rankable {
    let name: String
    let rank: Int
}

let players = [
    Player(name: "สมชาย", rank: 3),
    Player(name: "สมหญิง", rank: 1),
    Player(name: "มานะ", rank: 2)
]

let sorted = players.sorted()
sorted.forEach { print("\($0.rank). \($0.name)") }
```

### 5.3 Existentials vs Generics

```swift
// Swift 5.7+: any Protocol vs some Protocol

// any Protocol - existential (dynamic dispatch, boxing overhead)
// some Protocol - opaque type (static dispatch, no boxing)

protocol Animal {
    var name: String { get }
    func makeSound() -> String
}

struct Dog2: Animal {
    let name: String
    func makeSound() -> String { "Woof" }
}

struct Cat2: Animal {
    let name: String
    func makeSound() -> String { "Meow" }
}

// any Animal - existential, สามารถเก็บ types ต่างกันใน array
func makeNoise(animal: any Animal) {
    print("\(animal.name): \(animal.makeSound())")
}

let animals: [any Animal] = [Dog2(name: "Buddy"), Cat2(name: "Whiskers")]
animals.forEach { makeNoise(animal: $0) }

// some Animal - opaque, compiler รู้ concrete type
func firstAnimal() -> some Animal {
    return Dog2(name: "Rex")
}

// เมื่อไหร่ควรใช้อะไร?
// any - เมื่อต้องการ heterogeneous collection หรือ dynamic dispatch
// some - เมื่อ return type ต้องการ type identity (ใช้บ่อยใน SwiftUI)

// ตัวอย่างความแตกต่าง
func processAny(_ values: [any Equatable]) {
    // ไม่สามารถเปรียบเทียบ values เข้าหากันได้โดยตรง
    // เพราะ compiler ไม่รู้ว่าเป็น type เดียวกัน
}

func processGeneric<T: Equatable>(_ values: [T]) {
    // สามารถเปรียบเทียบได้ เพราะ T เป็น type เดียวกันทั้งหมด
    if values.count >= 2 {
        print("First == Second: \(values[0] == values[1])")
    }
}

// Performance comparison
// Generic (some): static dispatch, inlined, no heap allocation
// Existential (any): dynamic dispatch, boxing, heap allocation for value types
```

---

## 6. Protocol Composition

### 6.1 Typealias สำหรับ Compositions

```swift
// ใช้ typealias เพื่อสร้าง named compositions

protocol Persistable2 {
    func save()
    func load()
}

protocol Validatable {
    func validate() throws
}

protocol Observable2 {
    func addObserver(_ observer: AnyObject)
    func removeObserver(_ observer: AnyObject)
}

// Named compositions
typealias PersistableValidatable = Persistable2 & Validatable
typealias FullFeatured = Persistable2 & Validatable & Observable2

// Function ที่รับ composition
func processModel(_ model: PersistableValidatable) {
    do {
        try model.validate()
        model.save()
    } catch {
        print("Validation failed: \(error)")
    }
}

// Struct ที่ conform composition
struct UserModel: FullFeatured {
    let name: String
    let email: String
    var observers: [AnyObject] = []
    
    func save() { print("Saving user: \(name)") }
    func load() { print("Loading user: \(name)") }
    
    func validate() throws {
        guard !name.isEmpty else {
            throw ValidationError(message: "Name is required")
        }
        guard email.contains("@") else {
            throw ValidationError(message: "Invalid email")
        }
    }
    
    mutating func addObserver(_ observer: AnyObject) {
        observers.append(observer)
    }
    
    mutating func removeObserver(_ observer: AnyObject) {
        observers.removeAll { $0 === observer }
    }
}
```

### 6.2 Dynamic vs Static Dispatch

```swift
// Static dispatch: compiler รู้ method ที่จะเรียกตั้งแต่ compile time (เร็วกว่า)
// Dynamic dispatch: runtime ค้นหา method ที่จะเรียก (ยืดหยุ่นกว่า)

protocol Describable {
    func describe() -> String
}

struct ConcreteType: Describable {
    func describe() -> String { "Concrete" }
}

// Static dispatch - ใช้กับ generics
func describeStatic<T: Describable>(_ item: T) -> String {
    return item.describe()  // Static dispatch - compiler รู้ T เป็น ConcreteType
}

// Dynamic dispatch - ใช้กับ existentials
func describeDynamic(_ item: any Describable) -> String {
    return item.describe()  // Dynamic dispatch - runtime ค้นหา
}

// ตัวอย่างที่เห็นผลชัดกว่า
class Base {
    func method() -> String { "Base" }
}

class Derived: Base {
    override func method() -> String { "Derived" }
}

// Class ใช้ dynamic dispatch โดย default
let obj: Base = Derived()
print(obj.method())  // "Derived" - dynamic dispatch ทำงาน

// Struct ใช้ static dispatch
struct MyStruct: Describable {
    func describe() -> String { "Struct" }
}

// Benchmark mental model:
// Static dispatch: ~0.1 ns per call
// Dynamic dispatch: ~1-3 ns per call (vtable lookup)
// Existential boxing: additional heap allocation cost

// Protocol เมื่อใช้กับ @objc มักจะเป็น dynamic dispatch เสมอ
@objc protocol ObjCProtocol {
    func requiredMethod()
    @objc optional func optionalMethod()
}
```

---

## 7. Result Builders

### 7.1 @resultBuilder สำหรับ DSLs

```swift
// @resultBuilder: สร้าง DSL (Domain Specific Language) ที่อ่านง่าย

// ตัวอย่าง: สร้าง simple ViewBuilder-like DSL

@resultBuilder
struct StringArrayBuilder {
    // Required: buildBlock สร้าง result จาก components
    static func buildBlock(_ components: String...) -> [String] {
        return components
    }
    
    // Optional: รองรับ if statements
    static func buildOptional(_ component: [String]?) -> [String] {
        return component ?? []
    }
    
    // Optional: รองรับ if-else statements
    static func buildEither(first component: [String]) -> [String] {
        return component
    }
    
    static func buildEither(second component: [String]) -> [String] {
        return component
    }
    
    // Optional: รองรับ for loops
    static func buildArray(_ components: [[String]]) -> [String] {
        return components.flatMap { $0 }
    }
}

// ใช้ @resultBuilder
func makeList(@StringArrayBuilder content: () -> [String]) -> [String] {
    return content()
}

// DSL syntax ที่อ่านง่าย
let list = makeList {
    "รายการที่ 1"
    "รายการที่ 2"
    "รายการที่ 3"
}
print(list)

let showExtra = true
let conditionalList = makeList {
    "เสมอมี"
    if showExtra {
        "เงื่อนไข"
    }
}
print(conditionalList)
```

### 7.2 Building HTML Builder

```swift
// HTML Builder DSL

protocol HTMLElement {
    var html: String { get }
}

struct TextElement: HTMLElement {
    let text: String
    var html: String { text }
}

struct TagElement: HTMLElement {
    let tag: String
    let children: [HTMLElement]
    let attributes: [String: String]
    
    var html: String {
        let attrs = attributes.map { " \($0.key)=\"\($0.value)\"" }.joined()
        let childHTML = children.map(\.html).joined()
        return "<\(tag)\(attrs)>\(childHTML)</\(tag)>"
    }
}

@resultBuilder
struct HTMLBuilder {
    static func buildBlock(_ components: HTMLElement...) -> [HTMLElement] {
        return components
    }
    
    static func buildBlock(_ components: [HTMLElement]...) -> [HTMLElement] {
        return components.flatMap { $0 }
    }
    
    static func buildOptional(_ component: [HTMLElement]?) -> [HTMLElement] {
        return component ?? []
    }
    
    static func buildEither(first component: [HTMLElement]) -> [HTMLElement] { component }
    static func buildEither(second component: [HTMLElement]) -> [HTMLElement] { component }
    
    static func buildArray(_ components: [[HTMLElement]]) -> [HTMLElement] {
        return components.flatMap { $0 }
    }
    
    static func buildExpression(_ expression: HTMLElement) -> [HTMLElement] {
        return [expression]
    }
    
    static func buildExpression(_ expression: String) -> [HTMLElement] {
        return [TextElement(text: expression)]
    }
}

// Helper functions สำหรับสร้าง HTML elements
func div(@HTMLBuilder content: () -> [HTMLElement]) -> HTMLElement {
    TagElement(tag: "div", children: content(), attributes: [:])
}

func p(@HTMLBuilder content: () -> [HTMLElement]) -> HTMLElement {
    TagElement(tag: "p", children: content(), attributes: [:])
}

func h1(_ text: String) -> HTMLElement {
    TagElement(tag: "h1", children: [TextElement(text: text)], attributes: [:])
}

func ul(@HTMLBuilder content: () -> [HTMLElement]) -> HTMLElement {
    TagElement(tag: "ul", children: content(), attributes: [:])
}

func li(_ text: String) -> HTMLElement {
    TagElement(tag: "li", children: [TextElement(text: text)], attributes: [:])
}

// การใช้งาน HTML Builder DSL
let page = div {
    h1("สวัสดีชาว Swift!")
    p {
        "นี่คือตัวอย่าง HTML Builder"
    }
    ul {
        li("รายการที่ 1")
        li("รายการที่ 2")
        li("รายการที่ 3")
    }
}

print(page.html)
```

### 7.3 Building Query Builder

```swift
// SQL Query Builder DSL

struct Query {
    var table: String = ""
    var conditions: [String] = []
    var orderBy: [String] = []
    var limitValue: Int? = nil
    var selectColumns: [String] = ["*"]
    
    var sql: String {
        var parts = ["SELECT \(selectColumns.joined(separator: ", "))"]
        parts.append("FROM \(table)")
        
        if !conditions.isEmpty {
            parts.append("WHERE \(conditions.joined(separator: " AND "))")
        }
        
        if !orderBy.isEmpty {
            parts.append("ORDER BY \(orderBy.joined(separator: ", "))")
        }
        
        if let limit = limitValue {
            parts.append("LIMIT \(limit)")
        }
        
        return parts.joined(separator: " ")
    }
}

@resultBuilder
struct QueryBuilder {
    static func buildBlock(_ components: QueryModifier...) -> [QueryModifier] {
        return components
    }
}

protocol QueryModifier {
    func modify(_ query: inout Query)
}

struct SelectModifier: QueryModifier {
    let columns: [String]
    func modify(_ query: inout Query) { query.selectColumns = columns }
}

struct WhereModifier: QueryModifier {
    let condition: String
    func modify(_ query: inout Query) { query.conditions.append(condition) }
}

struct OrderByModifier: QueryModifier {
    let column: String
    let descending: Bool
    func modify(_ query: inout Query) {
        query.orderBy.append(descending ? "\(column) DESC" : column)
    }
}

struct LimitModifier: QueryModifier {
    let value: Int
    func modify(_ query: inout Query) { query.limitValue = value }
}

// DSL functions
func select(_ columns: String...) -> QueryModifier {
    SelectModifier(columns: columns)
}

func `where`(_ condition: String) -> QueryModifier {
    WhereModifier(condition: condition)
}

func orderBy(_ column: String, descending: Bool = false) -> QueryModifier {
    OrderByModifier(column: column, descending: descending)
}

func limit(_ value: Int) -> QueryModifier {
    LimitModifier(value: value)
}

func query(from table: String, @QueryBuilder modifiers: () -> [QueryModifier]) -> Query {
    var q = Query()
    q.table = table
    modifiers().forEach { $0.modify(&q) }
    return q
}

// การใช้งาน Query Builder DSL
let q = query(from: "users") {
    select("id", "name", "email")
    `where`("age > 18")
    `where`("is_active = true")
    orderBy("name")
    limit(10)
}

print(q.sql)
// SELECT id, name, email FROM users WHERE age > 18 AND is_active = true ORDER BY name LIMIT 10
```

---

## 8. @dynamicMemberLookup และ @dynamicCallable

### 8.1 JSON Dynamic Access

```swift
// @dynamicMemberLookup: เข้าถึง member ด้วย dot syntax โดยไม่ต้องประกาศ

import Foundation

@dynamicMemberLookup
struct JSON {
    private let value: Any?
    
    init(_ value: Any?) {
        self.value = value
    }
    
    subscript(dynamicMember member: String) -> JSON {
        if let dict = value as? [String: Any] {
            return JSON(dict[member])
        }
        return JSON(nil)
    }
    
    subscript(index: Int) -> JSON {
        if let array = value as? [Any], index < array.count {
            return JSON(array[index])
        }
        return JSON(nil)
    }
    
    var string: String? { value as? String }
    var int: Int? { value as? Int }
    var double: Double? { value as? Double }
    var bool: Bool? { value as? Bool }
    var array: [JSON]? {
        (value as? [Any])?.map { JSON($0) }
    }
    
    var isNull: Bool { value == nil }
    
    static func parse(_ data: Data) -> JSON? {
        guard let obj = try? JSONSerialization.jsonObject(with: data) else {
            return nil
        }
        return JSON(obj)
    }
}

// การใช้งาน
let jsonData = """
{
    "user": {
        "name": "สมชาย",
        "age": 30,
        "address": {
            "city": "กรุงเทพ"
        },
        "hobbies": ["อ่านหนังสือ", "เขียนโค้ด"]
    }
}
""".data(using: .utf8)!

if let json = JSON.parse(jsonData) {
    print(json.user.name.string ?? "ไม่มีชื่อ")         // สมชาย
    print(json.user.age.int ?? 0)                        // 30
    print(json.user.address.city.string ?? "ไม่มีเมือง")  // กรุงเทพ
    print(json.user.hobbies[0].string ?? "")             // อ่านหนังสือ
}
```

### 8.2 Proxy Patterns

```swift
// @dynamicMemberLookup สำหรับ proxy patterns

@dynamicMemberLookup
class UserDefaultsProxy {
    private let defaults: UserDefaults
    private let prefix: String
    
    init(suite: String? = nil, prefix: String = "") {
        defaults = UserDefaults(suiteName: suite) ?? .standard
        self.prefix = prefix
    }
    
    subscript(dynamicMember key: String) -> Any? {
        get { defaults.object(forKey: prefix + key) }
        set { defaults.set(newValue, forKey: prefix + key) }
    }
}

// การใช้งาน
let userPrefs = UserDefaultsProxy(prefix: "user_")

// เขียน
// userPrefs.theme = "dark"
// userPrefs.fontSize = 16
// userPrefs.notificationsEnabled = true

// อ่าน
// let theme = userPrefs.theme as? String

// @dynamicMemberLookup สำหรับ KeyPath access
@dynamicMemberLookup
struct KeyPathProxy<Root> {
    let root: Root
    
    subscript<T>(dynamicMember keyPath: KeyPath<Root, T>) -> T {
        return root[keyPath: keyPath]
    }
}

struct Person2 {
    let name: String
    let age: Int
    let email: String
}

let person2 = Person2(name: "สมชาย", age: 30, email: "somchai@example.com")
let proxy = KeyPathProxy(root: person2)

print(proxy.name)   // สมชาย
print(proxy.age)    // 30
print(proxy.email)  // somchai@example.com
```

### 8.3 @dynamicCallable

```swift
// @dynamicCallable: ทำให้ type สามารถถูกเรียกเหมือน function

@dynamicCallable
struct Calculator {
    enum Operation {
        case add, subtract, multiply, divide
    }
    
    let operation: Operation
    
    func dynamicallyCall(withArguments args: [Double]) -> Double {
        switch operation {
        case .add:
            return args.reduce(0, +)
        case .subtract:
            guard let first = args.first else { return 0 }
            return args.dropFirst().reduce(first, -)
        case .multiply:
            return args.reduce(1, *)
        case .divide:
            guard let first = args.first, first != 0 else { return 0 }
            return args.dropFirst().reduce(first, /)
        }
    }
    
    func dynamicallyCall(withKeywordArguments args: KeyValuePairs<String, Double>) -> Double {
        let values = args.map(\.value)
        return dynamicallyCall(withArguments: values)
    }
}

let add = Calculator(operation: .add)
let multiply = Calculator(operation: .multiply)

print(add(1, 2, 3, 4, 5))          // 15.0
print(multiply(2, 3, 4))            // 24.0
print(add(a: 10, b: 20, c: 30))     // 60.0

// @dynamicCallable สำหรับ command pattern
@dynamicCallable
struct Command {
    let name: String
    private let executor: ([String]) -> String
    
    init(name: String, execute: @escaping ([String]) -> String) {
        self.name = name
        self.executor = execute
    }
    
    func dynamicallyCall(withArguments args: [String]) -> String {
        return executor(args)
    }
}

let greetCommand = Command(name: "greet") { args in
    let name = args.first ?? "World"
    return "สวัสดี, \(name)!"
}

print(greetCommand("สมชาย"))   // สวัสดี, สมชาย!
print(greetCommand())           // สวัสดี, World!
```

---

## 9. Metatypes และ Protocols

### 9.1 .Type, .Protocol

```swift
// Metatypes: ค่าที่แทนตัว type เอง (ไม่ใช่ instance)

struct Dog3 {
    var name: String
}

// .Type - metatype ของ concrete type
let dogType: Dog3.Type = Dog3.self

// สร้าง instance จาก metatype
func createInstance<T>(_ type: T.Type) -> T where T: ExpressibleByStringLiteral {
    return "" as! T
}

// type(of:) - ได้ metatype จาก instance
let dog3 = Dog3(name: "Buddy")
let dogMetatype = type(of: dog3)  // Dog3.Type
print(dogMetatype)  // Dog3

// .Protocol - metatype ของ protocol
protocol Flyable2 {}
let flyableProtocol: Flyable2.Protocol = Flyable2.self

// ตรวจสอบ conformance ด้วย metatype
func doesConform<T>(_ type: T.Type, to protocol: Any.Type) -> Bool {
    return type is any Flyable2.Type
}

// Factory pattern กับ metatype
protocol ViewCreatable: AnyObject {
    init()
    func render() -> String
}

class ButtonView: ViewCreatable {
    required init() {}
    func render() -> String { "<button>Click me</button>" }
}

class TextFieldView: ViewCreatable {
    required init() {}
    func render() -> String { "<input type='text'/>" }
}

class ViewFactory {
    static func create<T: ViewCreatable>(_ type: T.Type) -> T {
        return T()
    }
    
    static func createAll(_ types: [any ViewCreatable.Type]) -> [any ViewCreatable] {
        return types.map { $0.init() }
    }
}

// การใช้งาน
let button = ViewFactory.create(ButtonView.self)
print(button.render())

let views = ViewFactory.createAll([ButtonView.self, TextFieldView.self])
views.forEach { print($0.render()) }
```

---

## 10. Mirror และ Reflection

### 10.1 Mirror API

```swift
import Foundation

// Mirror: ดู structure ของ type ที่ runtime

struct Person3 {
    let name: String
    let age: Int
    let email: String?
    let hobbies: [String]
}

let person3 = Person3(name: "สมชาย", age: 30, email: "test@example.com", hobbies: ["อ่านหนังสือ"])

let mirror = Mirror(reflecting: person3)

print("Type: \(mirror.subjectType)")
print("Display style: \(mirror.displayStyle ?? .none)")

for child in mirror.children {
    print("  \(child.label ?? "?") = \(child.value)")
}

// Recursive inspection
func inspect(_ value: Any, indent: String = "") {
    let mirror = Mirror(reflecting: value)
    
    if mirror.children.isEmpty {
        print("\(indent)\(value)")
    } else {
        for child in mirror.children {
            if let label = child.label {
                print("\(indent)\(label):")
            }
            inspect(child.value, indent: indent + "  ")
        }
    }
}

inspect(person3)

// Generic description ด้วย Mirror
func description(of value: Any) -> String {
    let mirror = Mirror(reflecting: value)
    var parts: [String] = []
    
    for child in mirror.children {
        let label = child.label ?? "?"
        let value = "\(child.value)"
        parts.append("\(label): \(value)")
    }
    
    return "\(mirror.subjectType)(\(parts.joined(separator: ", ")))"
}

print(description(of: person3))
```

### 10.2 Custom Mirror

```swift
// Custom Mirror: กำหนด representation ของ type เองใน debugger

struct SecureCredentials: CustomReflectable {
    let username: String
    private let password: String
    let createdAt: Date
    
    var customMirror: Mirror {
        // ซ่อน password ใน debug output
        return Mirror(self, children: [
            "username": username,
            "password": "***HIDDEN***",
            "createdAt": createdAt
        ], displayStyle: .struct)
    }
    
    init(username: String, password: String) {
        self.username = username
        self.password = password
        self.createdAt = Date()
    }
}

let creds = SecureCredentials(username: "admin", password: "secret123")
print(Mirror(reflecting: creds).children.map { "\($0.label ?? ""): \($0.value)" })
// password จะแสดง "***HIDDEN***"

// Custom Mirror สำหรับ Linked List
class LinkedList<T>: CustomReflectable {
    var value: T
    var next: LinkedList<T>?
    
    init(_ value: T, next: LinkedList<T>? = nil) {
        self.value = value
        self.next = next
    }
    
    var customMirror: Mirror {
        var children: [(label: String?, value: Any)] = [("value", value)]
        if let next = next {
            children.append(("next", next))
        }
        return Mirror(self, children: children, displayStyle: .class)
    }
}

let list = LinkedList(1, next: LinkedList(2, next: LinkedList(3)))
print(Mirror(reflecting: list))
```

---

## 11. Advanced Codable

### 11.1 Protocol-based Encoding Strategies

```swift
import Foundation

// Custom coding strategy ผ่าน protocol

protocol JSONEncodable {
    func encodeToJSON() throws -> Data
}

protocol JSONDecodable {
    static func decodeFromJSON(_ data: Data) throws -> Self
}

typealias JSONCodable = JSONEncodable & JSONDecodable

extension JSONEncodable where Self: Encodable {
    func encodeToJSON(encoder: JSONEncoder = JSONEncoder()) throws -> Data {
        encoder.outputFormatting = .prettyPrinted
        encoder.dateEncodingStrategy = .iso8601
        return try encoder.encode(self)
    }
}

extension JSONDecodable where Self: Decodable {
    static func decodeFromJSON(
        _ data: Data,
        decoder: JSONDecoder = JSONDecoder()
    ) throws -> Self {
        decoder.dateDecodingStrategy = .iso8601
        return try decoder.decode(Self.self, from: data)
    }
}

// Auto-conform structs
struct Product2: Codable, JSONCodable {
    let id: UUID
    let name: String
    let price: Double
    let createdAt: Date
}

// การใช้งาน
let product2 = Product2(
    id: UUID(),
    name: "iPhone 15",
    price: 35900,
    createdAt: Date()
)

if let json = try? product2.encodeToJSON(),
   let jsonString = String(data: json, encoding: .utf8) {
    print(jsonString)
}
```

### 11.2 Polymorphic Codable

```swift
import Foundation

// Polymorphic Codable: encode/decode types ที่แตกต่างกันผ่าน protocol

protocol Shape2: Codable {
    var shapeType: String { get }
    func area() -> Double
}

struct Circle2: Shape2 {
    let shapeType = "circle"
    let radius: Double
    func area() -> Double { .pi * radius * radius }
}

struct Rectangle2: Shape2 {
    let shapeType = "rectangle"
    let width: Double
    let height: Double
    func area() -> Double { width * height }
}

struct Triangle: Shape2 {
    let shapeType = "triangle"
    let base: Double
    let height: Double
    func area() -> Double { 0.5 * base * height }
}

// Type-tagged encoding
struct AnyShape: Codable {
    let shape: any Shape2
    
    enum CodingKeys: String, CodingKey {
        case type, data
    }
    
    init(_ shape: any Shape2) {
        self.shape = shape
    }
    
    func encode(to encoder: Encoder) throws {
        var container = encoder.container(keyedBy: CodingKeys.self)
        try container.encode(shape.shapeType, forKey: .type)
        
        switch shape.shapeType {
        case "circle":
            try container.encode(shape as! Circle2, forKey: .data)
        case "rectangle":
            try container.encode(shape as! Rectangle2, forKey: .data)
        case "triangle":
            try container.encode(shape as! Triangle, forKey: .data)
        default:
            throw EncodingError.invalidValue(shape, .init(codingPath: [], debugDescription: "Unknown shape type"))
        }
    }
    
    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        let type = try container.decode(String.self, forKey: .type)
        
        switch type {
        case "circle":
            shape = try container.decode(Circle2.self, forKey: .data)
        case "rectangle":
            shape = try container.decode(Rectangle2.self, forKey: .data)
        case "triangle":
            shape = try container.decode(Triangle.self, forKey: .data)
        default:
            throw DecodingError.dataCorruptedError(forKey: .type, in: container, debugDescription: "Unknown type: \(type)")
        }
    }
}

// การใช้งาน
let shapes: [AnyShape] = [
    AnyShape(Circle2(radius: 5)),
    AnyShape(Rectangle2(width: 10, height: 3)),
    AnyShape(Triangle(base: 6, height: 4))
]

if let encoded = try? JSONEncoder().encode(shapes),
   let decoded = try? JSONDecoder().decode([AnyShape].self, from: encoded) {
    decoded.forEach { print("Area: \($0.shape.area())") }
}
```

---

## 12. Real-world POP Case Studies

### 12.1 Validation Framework กับ Protocols

```swift
import Foundation

// Framework สำหรับ validation ที่ใช้ POP

// Core protocol
protocol Validator {
    associatedtype Value
    func validate(_ value: Value) -> ValidationResult
}

enum ValidationResult {
    case success
    case failure([ValidationError2])
    
    var isValid: Bool {
        if case .success = self { return true }
        return false
    }
    
    var errors: [ValidationError2] {
        if case .failure(let errors) = self { return errors }
        return []
    }
}

struct ValidationError2: Error, LocalizedError, Equatable {
    let field: String
    let message: String
    
    var errorDescription: String? { "\(field): \(message)" }
    
    static func == (lhs: ValidationError2, rhs: ValidationError2) -> Bool {
        lhs.field == rhs.field && lhs.message == rhs.message
    }
}

// Concrete validators
struct RangeValidator<T: Comparable>: Validator {
    let min: T
    let max: T
    let field: String
    
    func validate(_ value: T) -> ValidationResult {
        guard value >= min && value <= max else {
            return .failure([ValidationError2(
                field: field,
                message: "ค่าต้องอยู่ระหว่าง \(min) ถึง \(max)"
            )])
        }
        return .success
    }
}

struct RegexValidator: Validator {
    let pattern: String
    let field: String
    let message: String
    
    func validate(_ value: String) -> ValidationResult {
        let regex = try? NSRegularExpression(pattern: pattern)
        let range = NSRange(value.startIndex..., in: value)
        let matches = regex?.firstMatch(in: value, range: range)
        
        if matches == nil {
            return .failure([ValidationError2(field: field, message: message)])
        }
        return .success
    }
}

struct RequiredValidator: Validator {
    let field: String
    
    func validate(_ value: String) -> ValidationResult {
        return value.trimmingCharacters(in: .whitespaces).isEmpty
            ? .failure([ValidationError2(field: field, message: "จำเป็นต้องกรอก")])
            : .success
    }
}

// Composite validator
struct CompositeValidator<V: Validator>: Validator {
    typealias Value = V.Value
    
    private let validators: [V]
    
    init(_ validators: [V]) {
        self.validators = validators
    }
    
    func validate(_ value: V.Value) -> ValidationResult {
        let errors = validators
            .map { $0.validate(value) }
            .flatMap { $0.errors }
        
        return errors.isEmpty ? .success : .failure(errors)
    }
}

// FormValidator
class FormValidator<T> {
    private var fieldValidators: [(keyPath: PartialKeyPath<T>, validate: (T) -> ValidationResult)] = []
    
    func addValidator<V: Validator>(
        for keyPath: KeyPath<T, V.Value>,
        validator: V
    ) {
        fieldValidators.append((keyPath: keyPath, validate: { obj in
            validator.validate(obj[keyPath: keyPath])
        }))
    }
    
    func validate(_ value: T) -> ValidationResult {
        let errors = fieldValidators
            .map { $0.validate(value) }
            .flatMap { $0.errors }
        
        return errors.isEmpty ? .success : .failure(errors)
    }
}

// ตัวอย่างการใช้งาน
struct RegistrationForm {
    var username: String
    var age: Int
    var email: String
}

let formValidator = FormValidator<RegistrationForm>()

formValidator.addValidator(
    for: \.username,
    validator: RequiredValidator(field: "username")
)

formValidator.addValidator(
    for: \.age,
    validator: RangeValidator(min: 18, max: 120, field: "age")
)

formValidator.addValidator(
    for: \.email,
    validator: RegexValidator(
        pattern: #"^[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$"#,
        field: "email",
        message: "รูปแบบ email ไม่ถูกต้อง"
    )
)

let form = RegistrationForm(username: "test", age: 15, email: "invalid-email")
let result = formValidator.validate(form)

switch result {
case .success:
    print("ข้อมูลถูกต้อง")
case .failure(let errors):
    print("พบข้อผิดพลาด:")
    errors.forEach { print("  - \($0.errorDescription ?? "")") }
}
```

### 12.2 Dependency Injection Container

```swift
import Foundation

// Dependency Injection Container ด้วย POP

// Protocols สำหรับ DI
protocol Injectable {}

protocol DependencyContainer {
    func register<T>(_ type: T.Type, factory: @escaping () -> T)
    func resolve<T>(_ type: T.Type) -> T?
    func registerSingleton<T>(_ type: T.Type, factory: @escaping () -> T)
}

// Concrete DI Container
class Container: DependencyContainer {
    static let shared = Container()
    
    private var factories: [ObjectIdentifier: () -> Any] = [:]
    private var singletons: [ObjectIdentifier: Any] = [:]
    private var singletonFactories: [ObjectIdentifier: () -> Any] = [:]
    
    private init() {}
    
    func register<T>(_ type: T.Type, factory: @escaping () -> T) {
        factories[ObjectIdentifier(type)] = factory
    }
    
    func registerSingleton<T>(_ type: T.Type, factory: @escaping () -> T) {
        singletonFactories[ObjectIdentifier(type)] = factory
    }
    
    func resolve<T>(_ type: T.Type) -> T? {
        let key = ObjectIdentifier(type)
        
        // ตรวจสอบ singleton ก่อน
        if let singleton = singletons[key] as? T {
            return singleton
        }
        
        if let factory = singletonFactories[key] {
            let instance = factory() as! T
            singletons[key] = instance
            return instance
        }
        
        // สร้าง instance ใหม่
        return factories[key]?() as? T
    }
    
    func clearSingletons() {
        singletons.removeAll()
    }
}

// Protocols สำหรับ services
protocol UserServiceProtocol {
    func getCurrentUser() -> User2?
    func login(username: String, password: String) async throws -> User2
}

protocol DatabaseServiceProtocol {
    func fetch<T: Codable>(_ type: T.Type, id: String) async throws -> T?
    func save<T: Codable>(_ value: T, id: String) async throws
}

protocol NetworkServiceProtocol {
    func get(url: URL) async throws -> Data
    func post(url: URL, body: Data) async throws -> Data
}

// Concrete implementations
class UserService: UserServiceProtocol {
    private let database: DatabaseServiceProtocol
    private var currentUser: User2?
    
    init(database: DatabaseServiceProtocol) {
        self.database = database
    }
    
    func getCurrentUser() -> User2? { currentUser }
    
    func login(username: String, password: String) async throws -> User2 {
        // Mock implementation
        let user = User2(id: "1", name: username, email: "\(username)@example.com")
        currentUser = user
        return user
    }
}

struct User2: Codable {
    let id: String
    let name: String
    let email: String
}

class MockDatabaseService: DatabaseServiceProtocol {
    private var storage: [String: Data] = [:]
    
    func fetch<T: Codable>(_ type: T.Type, id: String) async throws -> T? {
        guard let data = storage[id] else { return nil }
        return try JSONDecoder().decode(type, from: data)
    }
    
    func save<T: Codable>(_ value: T, id: String) async throws {
        storage[id] = try JSONEncoder().encode(value)
    }
}

// Property wrapper สำหรับ injection
@propertyWrapper
struct Inject<T> {
    private let container: Container
    
    var wrappedValue: T {
        container.resolve(T.self)!
    }
    
    init(container: Container = .shared) {
        self.container = container
    }
}

// การตั้งค่า container
func setupDependencies() {
    let container = Container.shared
    
    container.registerSingleton(DatabaseServiceProtocol.self) {
        MockDatabaseService()
    }
    
    container.register(UserServiceProtocol.self) {
        let db = container.resolve(DatabaseServiceProtocol.self)!
        return UserService(database: db)
    }
}

// การใช้งาน
class ProfileViewModel {
    @Inject var userService: UserServiceProtocol
    
    func loadProfile() {
        if let user = userService.getCurrentUser() {
            print("User: \(user.name)")
        }
    }
}
```

---

## 13. Exercises with Solutions

### Exercise 1: Generic Cache ด้วย Protocol

```swift
// โจทย์: สร้าง generic cache ที่ใช้ protocol-based design
// สนับสนุน TTL (Time To Live) และ max capacity

protocol Cache {
    associatedtype Key: Hashable
    associatedtype Value
    
    func get(_ key: Key) -> Value?
    mutating func set(_ key: Key, value: Value)
    mutating func remove(_ key: Key)
    mutating func clear()
    var count: Int { get }
}

// Solution
struct TTLCache<K: Hashable, V>: Cache {
    typealias Key = K
    typealias Value = V
    
    private struct CacheEntry {
        let value: V
        let expiresAt: Date
        
        var isExpired: Bool {
            Date() > expiresAt
        }
    }
    
    private var storage: [K: CacheEntry] = [:]
    private let ttl: TimeInterval
    private let maxCapacity: Int
    
    init(ttl: TimeInterval = 300, maxCapacity: Int = 100) {
        self.ttl = ttl
        self.maxCapacity = maxCapacity
    }
    
    func get(_ key: K) -> V? {
        guard let entry = storage[key] else { return nil }
        if entry.isExpired {
            return nil  // return nil สำหรับ expired entries
        }
        return entry.value
    }
    
    mutating func set(_ key: K, value: V) {
        // ลบ expired entries ก่อน
        storage = storage.filter { !$0.value.isExpired }
        
        // ตรวจสอบ capacity
        if storage.count >= maxCapacity && storage[key] == nil {
            // ลบ entry เก่าสุด
            if let oldest = storage.min(by: { $0.value.expiresAt < $1.value.expiresAt }) {
                storage.removeValue(forKey: oldest.key)
            }
        }
        
        storage[key] = CacheEntry(
            value: value,
            expiresAt: Date().addingTimeInterval(ttl)
        )
    }
    
    mutating func remove(_ key: K) {
        storage.removeValue(forKey: key)
    }
    
    mutating func clear() {
        storage.removeAll()
    }
    
    var count: Int {
        storage.filter { !$0.value.isExpired }.count
    }
}

// Extension เพิ่ม functionality
extension Cache where Value: Equatable {
    func contains(value: Value, forKey key: Key) -> Bool {
        return get(key) == value
    }
}

// การใช้งาน
var cache = TTLCache<String, Int>(ttl: 60, maxCapacity: 5)
cache.set("a", value: 1)
cache.set("b", value: 2)
cache.set("c", value: 3)

print(cache.get("a"))  // Optional(1)
print(cache.count)     // 3

cache.remove("b")
print(cache.count)     // 2
```

### Exercise 2: Builder Pattern ด้วย Protocol

```swift
// โจทย์: สร้าง Builder pattern ที่ type-safe ด้วย protocols

protocol Buildable {
    associatedtype Built
    func build() -> Built
}

// Generic builder protocol
protocol StepBuilder: Buildable {
    associatedtype NextStep
    func next(_ step: NextStep) -> Self
}

// ตัวอย่าง: Email Builder
struct Email {
    let from: String
    let to: [String]
    let subject: String
    let body: String
    let attachments: [String]
}

// Type-safe step-by-step builder
struct EmailBuilder {
    private var from: String = ""
    private var to: [String] = []
    private var subject: String = ""
    private var body: String = ""
    private var attachments: [String] = []
    
    func from(_ address: String) -> EmailBuilder {
        var copy = self
        copy.from = address
        return copy
    }
    
    func to(_ addresses: String...) -> EmailBuilder {
        var copy = self
        copy.to = addresses
        return copy
    }
    
    func subject(_ text: String) -> EmailBuilder {
        var copy = self
        copy.subject = text
        return copy
    }
    
    func body(_ text: String) -> EmailBuilder {
        var copy = self
        copy.body = text
        return copy
    }
    
    func attach(_ filename: String) -> EmailBuilder {
        var copy = self
        copy.attachments.append(filename)
        return copy
    }
    
    func build() -> Email {
        return Email(
            from: from,
            to: to,
            subject: subject,
            body: body,
            attachments: attachments
        )
    }
}

// การใช้งาน
let email = EmailBuilder()
    .from("sender@example.com")
    .to("recipient1@example.com", "recipient2@example.com")
    .subject("สวัสดีจาก Swift!")
    .body("นี่คือ email ที่สร้างด้วย Builder pattern")
    .attach("document.pdf")
    .build()

print("From: \(email.from)")
print("To: \(email.to.joined(separator: ", "))")
print("Subject: \(email.subject)")
```

### Exercise 3: SwiftUI Component Library ด้วย POP

```swift
import SwiftUI

// โจทย์: สร้าง component library ที่ใช้ POP

// Core protocols
protocol StyledView: View {
    var style: ViewStyle { get }
}

protocol Configurable {
    associatedtype Config
    func configure(_ config: Config) -> Self
}

struct ViewStyle {
    var backgroundColor: Color
    var foregroundColor: Color
    var cornerRadius: CGFloat
    var padding: EdgeInsets
    var shadow: Shadow?
    
    struct Shadow {
        let color: Color
        let radius: CGFloat
        let x: CGFloat
        let y: CGFloat
    }
    
    static let `default` = ViewStyle(
        backgroundColor: .blue,
        foregroundColor: .white,
        cornerRadius: 8,
        padding: EdgeInsets(top: 12, leading: 16, bottom: 12, trailing: 16),
        shadow: nil
    )
    
    static let destructive = ViewStyle(
        backgroundColor: .red,
        foregroundColor: .white,
        cornerRadius: 8,
        padding: EdgeInsets(top: 12, leading: 16, bottom: 12, trailing: 16),
        shadow: nil
    )
    
    static let outlined = ViewStyle(
        backgroundColor: .clear,
        foregroundColor: .blue,
        cornerRadius: 8,
        padding: EdgeInsets(top: 12, leading: 16, bottom: 12, trailing: 16),
        shadow: nil
    )
}

// Custom button component
struct PrimaryButton: View, Configurable {
    struct Config {
        var title: String
        var style: ViewStyle
        var action: () -> Void
        var isLoading: Bool
        var isDisabled: Bool
        
        init(title: String, style: ViewStyle = .default, action: @escaping () -> Void) {
            self.title = title
            self.style = style
            self.action = action
            self.isLoading = false
            self.isDisabled = false
        }
    }
    
    private var config: Config
    
    init(title: String, action: @escaping () -> Void) {
        self.config = Config(title: title, action: action)
    }
    
    func configure(_ config: Config) -> PrimaryButton {
        var copy = self
        copy.config = config
        return copy
    }
    
    var body: some View {
        Button(action: config.action) {
            HStack {
                if config.isLoading {
                    ProgressView()
                        .progressViewStyle(CircularProgressViewStyle(tint: config.style.foregroundColor))
                        .scaleEffect(0.8)
                }
                Text(config.title)
                    .fontWeight(.semibold)
            }
            .frame(maxWidth: .infinity)
            .padding(config.style.padding)
            .background(config.style.backgroundColor)
            .foregroundColor(config.style.foregroundColor)
            .cornerRadius(config.style.cornerRadius)
            .opacity(config.isDisabled ? 0.5 : 1.0)
        }
        .disabled(config.isDisabled || config.isLoading)
    }
}

// Protocol-based card component
protocol CardContent: View {
    var title: String { get }
    var subtitle: String? { get }
}

struct Card<Content: View>: View {
    let title: String
    let subtitle: String?
    let content: () -> Content
    
    init(
        title: String,
        subtitle: String? = nil,
        @ViewBuilder content: @escaping () -> Content
    ) {
        self.title = title
        self.subtitle = subtitle
        self.content = content
    }
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            VStack(alignment: .leading, spacing: 4) {
                Text(title)
                    .font(.headline)
                if let subtitle = subtitle {
                    Text(subtitle)
                        .font(.subheadline)
                        .foregroundColor(.secondary)
                }
            }
            
            content()
        }
        .padding()
        .background(Color(.systemBackground))
        .cornerRadius(12)
        .shadow(color: .black.opacity(0.1), radius: 8, x: 0, y: 2)
    }
}

// Protocol สำหรับ list items
protocol ListItemRepresentable: Identifiable {
    var title: String { get }
    var subtitle: String? { get }
    var iconName: String? { get }
}

struct ListItemView<T: ListItemRepresentable>: View {
    let item: T
    let action: ((T) -> Void)?
    
    init(item: T, action: ((T) -> Void)? = nil) {
        self.item = item
        self.action = action
    }
    
    var body: some View {
        Button {
            action?(item)
        } label: {
            HStack(spacing: 12) {
                if let iconName = item.iconName {
                    Image(systemName: iconName)
                        .foregroundColor(.blue)
                        .frame(width: 30)
                }
                
                VStack(alignment: .leading, spacing: 2) {
                    Text(item.title)
                        .foregroundColor(.primary)
                    
                    if let subtitle = item.subtitle {
                        Text(subtitle)
                            .font(.caption)
                            .foregroundColor(.secondary)
                    }
                }
                
                Spacer()
                
                if action != nil {
                    Image(systemName: "chevron.right")
                        .foregroundColor(.secondary)
                        .font(.caption)
                }
            }
            .padding(.vertical, 8)
        }
    }
}

// ตัวอย่างการใช้งาน component library
struct MenuItem: ListItemRepresentable {
    let id = UUID()
    let title: String
    let subtitle: String?
    let iconName: String?
    let action: () -> Void
}

struct SettingsView: View {
    let menuItems = [
        MenuItem(
            title: "โปรไฟล์",
            subtitle: "แก้ไขข้อมูลส่วนตัว",
            iconName: "person.circle",
            action: { print("เปิดโปรไฟล์") }
        ),
        MenuItem(
            title: "การแจ้งเตือน",
            subtitle: "จัดการการแจ้งเตือน",
            iconName: "bell",
            action: { print("เปิดการแจ้งเตือน") }
        ),
        MenuItem(
            title: "ความปลอดภัย",
            subtitle: "รหัสผ่านและ biometrics",
            iconName: "lock",
            action: { print("เปิดความปลอดภัย") }
        )
    ]
    
    var body: some View {
        NavigationView {
            List {
                Card(title: "บัญชีผู้ใช้", subtitle: "จัดการข้อมูลบัญชี") {
                    ForEach(menuItems) { item in
                        ListItemView(item: item, action: { item in
                            item.action()
                        })
                    }
                }
                .listRowInsets(EdgeInsets())
                .listRowBackground(Color.clear)
            }
            .navigationTitle("ตั้งค่า")
        }
    }
}
```

---

## สรุป

ในบทนี้เราได้เรียนรู้ Protocol-Oriented Programming อย่างลึกซึ้ง:

1. **POP Philosophy**: เริ่มต้นด้วย protocol, value types แทน class hierarchies
2. **Protocol Design**: Interface segregation, naming conventions, retroactive conformance
3. **Generic Programming**: Constrained generics, primary associated types, type erasure, opaque types
4. **Protocol Extensions**: Default implementations, conditional extensions, diamond problem
5. **Associated Types**: PAT, Self requirements, existentials vs generics
6. **Protocol Composition**: Typealias, dynamic vs static dispatch
7. **Result Builders**: @resultBuilder DSLs, HTML builder, query builder
8. **@dynamicMemberLookup**: JSON access, proxy patterns
9. **@dynamicCallable**: Function-like calling
10. **Metatypes**: .Type, factory patterns
11. **Mirror**: Reflection, custom mirror
12. **Advanced Codable**: Protocol-based strategies, polymorphic coding
13. **Real-world cases**: Validation framework, DI container, SwiftUI component library
14. **Exercises**: Cache, Builder pattern, Component library

Protocol-Oriented Programming เป็นหัวใจของ Swift และเป็น skill สำคัญที่ช่วยให้เขียนโค้ดที่ยืดหยุ่น ทดสอบได้ง่าย และบำรุงรักษาได้ในระยะยาว

---

*จบ Part 92: Protocol-Oriented Programming (POP) Mastery*
