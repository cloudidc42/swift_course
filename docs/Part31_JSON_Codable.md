# Part 31: JSON และ Codable ใน Swift

## สารบัญ

1. [JSON คืออะไร](#json-คืออะไร)
2. [JSONEncoder และ JSONDecoder](#jsonencoder-และ-jsondecoder)
3. [Codable Protocol](#codable-protocol)
4. [Automatic Synthesis](#automatic-synthesis)
5. [CodingKeys Enum](#codingkeys-enum)
6. [Custom Encoding/Decoding](#custom-encodingdecoding)
7. [Nested Structures](#nested-structures)
8. [Arrays ใน JSON](#arrays-ใน-json)
9. [Optional JSON Fields](#optional-json-fields)
10. [Date Encoding Strategies](#date-encoding-strategies)
11. [Data Encoding Strategies](#data-encoding-strategies)
12. [Key Decoding Strategies](#key-decoding-strategies)
13. [Handling Unknown JSON Keys](#handling-unknown-json-keys)
14. [JSONSerialization (Legacy)](#jsonserialization-legacy)
15. [PropertyListEncoder/Decoder](#propertylistencoderdecoder)
16. [Custom CodingKey](#custom-codingkey)
17. [Encoding Inheritance](#encoding-inheritance)
18. [Polymorphic Decoding](#polymorphic-decoding)
19. [JSON with Generics](#json-with-generics)
20. [Error Handling ใน Codable](#error-handling-ใน-codable)
21. [Testing Codable Implementations](#testing-codable-implementations)
22. [Practical Exercises](#practical-exercises)
23. [Building a JSON API Client](#building-a-json-api-client)
24. [Summary](#summary)

---

## JSON คืออะไร

**JSON (JavaScript Object Notation)** เป็น format สำหรับการแลกเปลี่ยนข้อมูลที่มีน้ำหนักเบา (lightweight) อ่านง่ายสำหรับมนุษย์ และง่ายต่อการ parse สำหรับเครื่อง JSON เป็น subset ของ JavaScript แต่ถูกใช้งานอย่างแพร่หลายในทุกภาษาโปรแกรม

### โครงสร้างพื้นฐานของ JSON

JSON ประกอบด้วยประเภทข้อมูล 6 ประเภทหลัก:

```
1. String   : "Hello, World!"
2. Number   : 42, 3.14
3. Boolean  : true, false
4. Null     : null
5. Array    : [1, 2, 3]
6. Object   : {"key": "value"}
```

### ตัวอย่าง JSON

```json
{
  "name": "สมชาย ใจดี",
  "age": 30,
  "isStudent": false,
  "email": "somchai@example.com",
  "address": {
    "street": "123 ถนนสุขุมวิท",
    "city": "กรุงเทพมหานคร",
    "zipCode": "10110"
  },
  "hobbies": ["อ่านหนังสือ", "เล่นกีตาร์", "ท่องเที่ยว"],
  "scores": [85.5, 92.0, 78.3],
  "profilePicture": null
}
```

### ทำไม JSON ถึงนิยมใช้?

1. **ใช้งานง่าย**: Syntax เรียบง่าย อ่านได้ง่าย
2. **Universal**: รองรับทุกภาษาโปรแกรม
3. **Lightweight**: ขนาดเล็กกว่า XML
4. **Native Support**: Browser และ server รองรับ natively
5. **REST API Standard**: เป็น standard ของ REST API

### JSON vs XML

```xml
<!-- XML - verbose -->
<person>
  <name>สมชาย</name>
  <age>30</age>
</person>
```

```json
// JSON - concise
{
  "name": "สมชาย",
  "age": 30
}
```

---

## JSONEncoder และ JSONDecoder

Swift มี built-in class สำหรับทำงานกับ JSON คือ `JSONEncoder` (แปลง Swift object เป็น JSON) และ `JSONDecoder` (แปลง JSON เป็น Swift object)

### JSONEncoder - การแปลงข้อมูลเป็น JSON

```swift
import Foundation

struct Person: Codable {
    var name: String
    var age: Int
    var email: String
}

// สร้าง object
let person = Person(name: "สมชาย ใจดี", age: 30, email: "somchai@example.com")

// สร้าง encoder
let encoder = JSONEncoder()

// ตั้งค่าให้ output สวยงาม
encoder.outputFormatting = .prettyPrinted

do {
    // encode เป็น Data
    let jsonData = try encoder.encode(person)
    
    // แปลง Data เป็น String เพื่อดู
    if let jsonString = String(data: jsonData, encoding: .utf8) {
        print(jsonString)
    }
} catch {
    print("Encoding error: \(error)")
}

/* Output:
{
  "name" : "สมชาย ใจดี",
  "age" : 30,
  "email" : "somchai@example.com"
}
*/
```

### JSONDecoder - การแปลง JSON เป็นข้อมูล

```swift
import Foundation

struct Person: Codable {
    var name: String
    var age: Int
    var email: String
}

// JSON String
let jsonString = """
{
    "name": "สมชาย ใจดี",
    "age": 30,
    "email": "somchai@example.com"
}
"""

// แปลง String เป็น Data
guard let jsonData = jsonString.data(using: .utf8) else {
    fatalError("Cannot convert string to data")
}

// สร้าง decoder
let decoder = JSONDecoder()

do {
    // decode เป็น Person
    let person = try decoder.decode(Person.self, from: jsonData)
    print("ชื่อ: \(person.name)")
    print("อายุ: \(person.age)")
    print("อีเมล: \(person.email)")
} catch {
    print("Decoding error: \(error)")
}

/* Output:
ชื่อ: สมชาย ใจดี
อายุ: 30
อีเมล: somchai@example.com
*/
```

### OutputFormatting Options

```swift
import Foundation

struct Product: Codable {
    var id: Int
    var name: String
    var price: Double
    var inStock: Bool
}

let product = Product(id: 1, name: "iPhone 15", price: 35900.0, inStock: true)
let encoder = JSONEncoder()

// ตัวเลือกการจัดรูปแบบ output
// 1. Pretty printed (เพิ่ม whitespace)
encoder.outputFormatting = .prettyPrinted

// 2. Sorted keys (เรียง key ตาม alphabet)
encoder.outputFormatting = [.prettyPrinted, .sortedKeys]

// 3. Without escaping slashes
encoder.outputFormatting = [.prettyPrinted, .withoutEscapingSlashes]

// รวมทุก options
encoder.outputFormatting = [.prettyPrinted, .sortedKeys, .withoutEscapingSlashes]

let jsonData = try! encoder.encode(product)
print(String(data: jsonData, encoding: .utf8)!)

/* Output with sortedKeys + prettyPrinted:
{
  "id" : 1,
  "inStock" : true,
  "name" : "iPhone 15",
  "price" : 35900
}
*/
```

---

## Codable Protocol

`Codable` เป็น type alias ที่รวม `Encodable` และ `Decodable` protocol เข้าด้วยกัน

```swift
typealias Codable = Encodable & Decodable
```

### Encodable Protocol

`Encodable` protocol กำหนดให้ type สามารถ encode ตัวเองเป็น external representation (เช่น JSON) ได้

```swift
public protocol Encodable {
    func encode(to encoder: Encoder) throws
}
```

### Decodable Protocol

`Decodable` protocol กำหนดให้ type สามารถ decode ตัวเองจาก external representation ได้

```swift
public protocol Decodable {
    init(from decoder: Decoder) throws
}
```

### การใช้งาน Codable

```swift
import Foundation

// Struct with Codable
struct Student: Codable {
    var studentID: String
    var firstName: String
    var lastName: String
    var gpa: Double
    var year: Int
}

// Encode
let student = Student(
    studentID: "STU001",
    firstName: "วิชัย",
    lastName: "สมบูรณ์",
    gpa: 3.75,
    year: 3
)

let encoder = JSONEncoder()
encoder.outputFormatting = .prettyPrinted

if let data = try? encoder.encode(student),
   let jsonString = String(data: data, encoding: .utf8) {
    print("Encoded JSON:")
    print(jsonString)
}

// Decode
let jsonString = """
{
    "studentID": "STU002",
    "firstName": "มาลี",
    "lastName": "รักไทย",
    "gpa": 3.90,
    "year": 2
}
"""

let decoder = JSONDecoder()
if let data = jsonString.data(using: .utf8),
   let decodedStudent = try? decoder.decode(Student.self, from: data) {
    print("\nDecoded Student:")
    print("ID: \(decodedStudent.studentID)")
    print("ชื่อ: \(decodedStudent.firstName) \(decodedStudent.lastName)")
    print("GPA: \(decodedStudent.gpa)")
    print("ชั้นปี: \(decodedStudent.year)")
}
```

### Class กับ Codable

```swift
import Foundation

class Animal: Codable {
    var name: String
    var species: String
    var age: Int
    
    init(name: String, species: String, age: Int) {
        self.name = name
        self.species = species
        self.age = age
    }
}

let cat = Animal(name: "Whiskers", species: "Cat", age: 3)
let encoder = JSONEncoder()
encoder.outputFormatting = .prettyPrinted

if let data = try? encoder.encode(cat),
   let json = String(data: data, encoding: .utf8) {
    print(json)
}

/* Output:
{
  "name" : "Whiskers",
  "species" : "Cat",
  "age" : 3
}
*/
```

### Enum กับ Codable

```swift
import Foundation

// Enum with RawValue ใช้ Codable ได้ทันที
enum Color: String, Codable {
    case red = "red"
    case green = "green"
    case blue = "blue"
}

enum Priority: Int, Codable {
    case low = 1
    case medium = 2
    case high = 3
}

struct Task: Codable {
    var title: String
    var color: Color
    var priority: Priority
}

let task = Task(title: "เรียน Swift", color: .blue, priority: .high)
let encoder = JSONEncoder()
encoder.outputFormatting = .prettyPrinted

if let data = try? encoder.encode(task),
   let json = String(data: data, encoding: .utf8) {
    print(json)
}

/* Output:
{
  "title" : "เรียน Swift",
  "color" : "blue",
  "priority" : 3
}
*/
```

---

## Automatic Synthesis

Swift compiler สามารถ synthesize (สร้างอัตโนมัติ) การ implementation ของ `Codable` ให้เราได้ โดยไม่ต้องเขียน code เอง

### เงื่อนไขของ Automatic Synthesis

1. Type ทุกตัวใน properties ต้อง implement `Codable` ด้วย
2. Property ทุกตัวต้องมี key ที่ตรงกับ JSON

```swift
import Foundation

// ทุก property type implement Codable อยู่แล้ว (String, Int, Double, Bool)
struct UserProfile: Codable {
    var username: String    // String: Codable ✓
    var age: Int            // Int: Codable ✓
    var height: Double      // Double: Codable ✓
    var isVerified: Bool    // Bool: Codable ✓
}

// Swift จะ synthesize encode(to:) และ init(from:) ให้อัตโนมัติ
```

### Built-in Types ที่รองรับ Codable

```swift
// Numeric types
let int: Int = 42
let float: Float = 3.14
let double: Double = 2.718
let int8: Int8 = 127
let int16: Int16 = 32767
let int32: Int32 = 2147483647
let int64: Int64 = 9223372036854775807
let uint: UInt = 42
let uint8: UInt8 = 255

// Text
let string: String = "Hello"
let character: Character = "A"  // ไม่รองรับ Codable โดยตรง

// Boolean
let bool: Bool = true

// Optional
let optionalInt: Int? = nil  // Optional ของ Codable type ก็ Codable

// Collections
let array: [Int] = [1, 2, 3]
let dictionary: [String: Int] = ["a": 1]
let set: Set<Int> = [1, 2, 3]

// Date, URL, Data
let date: Date = Date()
let url: URL = URL(string: "https://example.com")!
let data: Data = Data()
```

### ตัวอย่าง Automatic Synthesis ที่ซับซ้อนขึ้น

```swift
import Foundation

struct Address: Codable {
    var street: String
    var city: String
    var country: String
    var postalCode: String
}

struct ContactInfo: Codable {
    var email: String
    var phone: String?  // Optional
    var website: URL?   // Optional URL
}

struct Employee: Codable {
    var id: Int
    var name: String
    var salary: Double
    var isFullTime: Bool
    var address: Address        // Nested Codable struct
    var contact: ContactInfo    // Nested Codable struct
    var skills: [String]        // Array of Codable
    var ratings: [Double]       // Array of Codable
}

// สร้าง Employee
let emp = Employee(
    id: 1001,
    name: "ประวิตร มั่นคง",
    salary: 75000.0,
    isFullTime: true,
    address: Address(
        street: "456 ถนนเพลินจิต",
        city: "กรุงเทพมหานคร",
        country: "Thailand",
        postalCode: "10330"
    ),
    contact: ContactInfo(
        email: "prawit@company.com",
        phone: "081-234-5678",
        website: URL(string: "https://prawit.dev")
    ),
    skills: ["Swift", "Python", "SQL"],
    ratings: [4.5, 4.8, 4.2]
)

let encoder = JSONEncoder()
encoder.outputFormatting = .prettyPrinted

if let data = try? encoder.encode(emp),
   let json = String(data: data, encoding: .utf8) {
    print(json)
}
```

---

## CodingKeys Enum

บางครั้ง JSON key ไม่ตรงกับชื่อ property ใน Swift เราใช้ `CodingKeys` enum เพื่อ map ระหว่างกัน

### ปัญหาที่พบบ่อย

```swift
// JSON มี key แบบนี้:
// {
//     "user_name": "john_doe",
//     "first_name": "John",
//     "last_name": "Doe"
// }

// แต่เราอยากใช้ camelCase ใน Swift:
struct User: Codable {
    var userName: String    // ✗ ไม่ตรงกับ "user_name"
    var firstName: String   // ✗ ไม่ตรงกับ "first_name"
    var lastName: String    // ✗ ไม่ตรงกับ "last_name"
}
```

### การใช้ CodingKeys

```swift
import Foundation

struct User: Codable {
    var userName: String
    var firstName: String
    var lastName: String
    var emailAddress: String
    
    // CodingKeys ต้อง conform to CodingKey protocol
    enum CodingKeys: String, CodingKey {
        case userName = "user_name"
        case firstName = "first_name"
        case lastName = "last_name"
        case emailAddress = "email_address"
    }
}

let jsonString = """
{
    "user_name": "john_doe",
    "first_name": "John",
    "last_name": "Doe",
    "email_address": "john@example.com"
}
"""

let decoder = JSONDecoder()
if let data = jsonString.data(using: .utf8),
   let user = try? decoder.decode(User.self, from: data) {
    print("Username: \(user.userName)")
    print("Name: \(user.firstName) \(user.lastName)")
    print("Email: \(user.emailAddress)")
}
```

### CodingKeys บางส่วน

```swift
import Foundation

struct Product: Codable {
    var productID: Int      // จะถูก map กับ "product_id"
    var name: String        // ใช้ key เดิม "name"
    var price: Double       // ใช้ key เดิม "price"
    var stockCount: Int     // จะถูก map กับ "stock_count"
    
    enum CodingKeys: String, CodingKey {
        case productID = "product_id"
        case name               // ไม่ต้องใส่ value ถ้า key เหมือนกัน
        case price              // ไม่ต้องใส่ value
        case stockCount = "stock_count"
    }
}

// ทดสอบ
let json = """
{
    "product_id": 101,
    "name": "MacBook Pro",
    "price": 79900.0,
    "stock_count": 50
}
"""

let decoder = JSONDecoder()
if let data = json.data(using: .utf8),
   let product = try? decoder.decode(Product.self, from: data) {
    print("Product: \(product.name)")
    print("ID: \(product.productID)")
    print("Price: \(product.price)")
    print("Stock: \(product.stockCount)")
}
```

### CodingKeys สำหรับ Exclude Properties

```swift
import Foundation

struct UserSession: Codable {
    var userID: String
    var accessToken: String
    var refreshToken: String
    // cachedData จะไม่ถูก encode/decode
    var cachedData: [String: Any]? = nil
    
    // ไม่รวม cachedData ใน CodingKeys
    enum CodingKeys: String, CodingKey {
        case userID = "user_id"
        case accessToken = "access_token"
        case refreshToken = "refresh_token"
        // ไม่มี cachedData
    }
}
```

---

## Custom Encoding/Decoding

เมื่อ automatic synthesis ไม่เพียงพอ เราสามารถ implement `encode(to:)` และ `init(from:)` เองได้

### Custom Encoding

```swift
import Foundation

struct Temperature: Codable {
    var celsius: Double
    
    // Custom encoding - encode เป็น celsius เสมอ
    func encode(to encoder: Encoder) throws {
        var container = encoder.singleValueContainer()
        try container.encode(celsius)
    }
    
    // Custom decoding - decode จากทั้ง object หรือ number
    init(from decoder: Decoder) throws {
        // ลองอ่านเป็น number ก่อน
        if let container = try? decoder.singleValueContainer(),
           let value = try? container.decode(Double.self) {
            celsius = value
            return
        }
        
        // ถ้าไม่ได้ลองอ่านเป็น object
        let container = try decoder.container(keyedBy: CodingKeys.self)
        
        if let c = try? container.decode(Double.self, forKey: .celsius) {
            celsius = c
        } else if let f = try? container.decode(Double.self, forKey: .fahrenheit) {
            celsius = (f - 32) * 5 / 9
        } else {
            throw DecodingError.dataCorrupted(
                DecodingError.Context(
                    codingPath: decoder.codingPath,
                    debugDescription: "Cannot decode temperature"
                )
            )
        }
    }
    
    enum CodingKeys: String, CodingKey {
        case celsius
        case fahrenheit
    }
}

// ทดสอบ
let jsonCelsius = "25.5"
let jsonObject = """{"celsius": 25.5}"""
let jsonFahrenheit = """{"fahrenheit": 77.9}"""

let decoder = JSONDecoder()

if let data = jsonCelsius.data(using: .utf8),
   let temp = try? decoder.decode(Temperature.self, from: data) {
    print("จาก number: \(temp.celsius)°C")
}

if let data = jsonObject.data(using: .utf8),
   let temp = try? decoder.decode(Temperature.self, from: data) {
    print("จาก celsius object: \(temp.celsius)°C")
}

if let data = jsonFahrenheit.data(using: .utf8),
   let temp = try? decoder.decode(Temperature.self, from: data) {
    print("จาก fahrenheit object: \(String(format: "%.1f", temp.celsius))°C")
}
```

### Custom Decoding ที่ซับซ้อน

```swift
import Foundation

struct APIResponse: Decodable {
    var success: Bool
    var message: String
    var data: [String: String]?
    var errorCode: Int?
    var timestamp: Date
    
    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        
        // Decode ค่าปกติ
        success = try container.decode(Bool.self, forKey: .success)
        message = try container.decode(String.self, forKey: .message)
        
        // Decode optional values
        data = try container.decodeIfPresent([String: String].self, forKey: .data)
        errorCode = try container.decodeIfPresent(Int.self, forKey: .errorCode)
        
        // Decode timestamp จาก unix timestamp
        let timestampValue = try container.decode(Double.self, forKey: .timestamp)
        timestamp = Date(timeIntervalSince1970: timestampValue)
    }
    
    enum CodingKeys: String, CodingKey {
        case success
        case message
        case data
        case errorCode = "error_code"
        case timestamp
    }
}

// ทดสอบ
let json = """
{
    "success": true,
    "message": "Operation completed",
    "data": {"userId": "123", "role": "admin"},
    "timestamp": 1693500000
}
"""

let decoder = JSONDecoder()
if let jsonData = json.data(using: .utf8),
   let response = try? decoder.decode(APIResponse.self, from: jsonData) {
    print("Success: \(response.success)")
    print("Message: \(response.message)")
    print("Data: \(response.data ?? [:])")
    print("Timestamp: \(response.timestamp)")
}
```

### Custom Encoding สำหรับ Computed Properties

```swift
import Foundation

struct Circle: Codable {
    var radius: Double
    
    // Computed property - ไม่ต้องการ encode
    var area: Double {
        return Double.pi * radius * radius
    }
    
    var circumference: Double {
        return 2 * Double.pi * radius
    }
    
    enum CodingKeys: String, CodingKey {
        case radius
        // ไม่มี area, circumference เพราะเป็น computed
    }
}

let circle = Circle(radius: 5.0)
print("Radius: \(circle.radius)")
print("Area: \(String(format: "%.2f", circle.area))")
print("Circumference: \(String(format: "%.2f", circle.circumference))")

let encoder = JSONEncoder()
encoder.outputFormatting = .prettyPrinted
if let data = try? encoder.encode(circle),
   let json = String(data: data, encoding: .utf8) {
    print("\nJSON:")
    print(json)  // จะมีแค่ radius
}
```

---

## Nested Structures

JSON มักมีโครงสร้างซับซ้อนที่มี nested objects เราสามารถ map กับ nested Swift structs ได้

### Basic Nested Structures

```swift
import Foundation

struct GeoLocation: Codable {
    var latitude: Double
    var longitude: Double
    var altitude: Double?
}

struct Restaurant: Codable {
    var id: Int
    var name: String
    var cuisine: String
    var rating: Double
    var location: GeoLocation  // Nested struct
    var openingHours: OpeningHours
    
    struct OpeningHours: Codable {  // Nested type definition
        var open: String    // "09:00"
        var close: String   // "22:00"
        var isOpen24Hours: Bool
    }
}

let json = """
{
    "id": 1,
    "name": "ร้านอาหารไทยแท้",
    "cuisine": "Thai",
    "rating": 4.8,
    "location": {
        "latitude": 13.7563,
        "longitude": 100.5018,
        "altitude": 10.0
    },
    "openingHours": {
        "open": "10:00",
        "close": "22:00",
        "isOpen24Hours": false
    }
}
"""

let decoder = JSONDecoder()
if let data = json.data(using: .utf8),
   let restaurant = try? decoder.decode(Restaurant.self, from: data) {
    print("Restaurant: \(restaurant.name)")
    print("Rating: \(restaurant.rating)")
    print("Location: \(restaurant.location.latitude), \(restaurant.location.longitude)")
    print("Hours: \(restaurant.openingHours.open) - \(restaurant.openingHours.close)")
}
```

### Deep Nested Structures

```swift
import Foundation

struct Country: Codable {
    var name: String
    var code: String
}

struct City: Codable {
    var name: String
    var country: Country
}

struct Neighborhood: Codable {
    var name: String
    var city: City
}

struct FullAddress: Codable {
    var streetNumber: String
    var streetName: String
    var neighborhood: Neighborhood
    
    enum CodingKeys: String, CodingKey {
        case streetNumber = "street_number"
        case streetName = "street_name"
        case neighborhood
    }
}

let json = """
{
    "street_number": "123",
    "street_name": "ถนนสีลม",
    "neighborhood": {
        "name": "สีลม",
        "city": {
            "name": "กรุงเทพมหานคร",
            "country": {
                "name": "Thailand",
                "code": "TH"
            }
        }
    }
}
"""

let decoder = JSONDecoder()
if let data = json.data(using: .utf8),
   let address = try? decoder.decode(FullAddress.self, from: data) {
    print("ที่อยู่: \(address.streetNumber) \(address.streetName)")
    print("แขวง: \(address.neighborhood.name)")
    print("เมือง: \(address.neighborhood.city.name)")
    print("ประเทศ: \(address.neighborhood.city.country.name)")
}
```

### Flattening Nested JSON

```swift
import Foundation

// บางครั้งเราอยากรวม nested JSON เป็น flat struct
// JSON:
// {
//     "user": {
//         "id": 1,
//         "name": "สมชาย"
//     },
//     "profile": {
//         "bio": "นักพัฒนา iOS",
//         "avatar": "https://..."
//     }
// }

struct FlatUserProfile: Decodable {
    var id: Int
    var name: String
    var bio: String
    var avatar: String
    
    enum CodingKeys: String, CodingKey {
        case user
        case profile
    }
    
    enum UserKeys: String, CodingKey {
        case id
        case name
    }
    
    enum ProfileKeys: String, CodingKey {
        case bio
        case avatar
    }
    
    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        
        let userContainer = try container.nestedContainer(keyedBy: UserKeys.self, forKey: .user)
        id = try userContainer.decode(Int.self, forKey: .id)
        name = try userContainer.decode(String.self, forKey: .name)
        
        let profileContainer = try container.nestedContainer(keyedBy: ProfileKeys.self, forKey: .profile)
        bio = try profileContainer.decode(String.self, forKey: .bio)
        avatar = try profileContainer.decode(String.self, forKey: .avatar)
    }
}

let json = """
{
    "user": {
        "id": 1,
        "name": "สมชาย"
    },
    "profile": {
        "bio": "นักพัฒนา iOS",
        "avatar": "https://example.com/avatar.jpg"
    }
}
"""

let decoder = JSONDecoder()
if let data = json.data(using: .utf8),
   let profile = try? decoder.decode(FlatUserProfile.self, from: data) {
    print("ID: \(profile.id)")
    print("Name: \(profile.name)")
    print("Bio: \(profile.bio)")
    print("Avatar: \(profile.avatar)")
}
```

---

## Arrays ใน JSON

JSON รองรับ arrays ที่ประกอบด้วยข้อมูลหลายประเภท

### Basic Array Decoding

```swift
import Foundation

struct Movie: Codable {
    var title: String
    var year: Int
    var rating: Double
    var genres: [String]  // Array of strings
}

// Decode array ของ objects
let jsonArray = """
[
    {
        "title": "Inception",
        "year": 2010,
        "rating": 8.8,
        "genres": ["Action", "Adventure", "Sci-Fi"]
    },
    {
        "title": "The Dark Knight",
        "year": 2008,
        "rating": 9.0,
        "genres": ["Action", "Crime", "Drama"]
    }
]
"""

let decoder = JSONDecoder()
if let data = jsonArray.data(using: .utf8),
   let movies = try? decoder.decode([Movie].self, from: data) {
    for movie in movies {
        print("\(movie.title) (\(movie.year)) - Rating: \(movie.rating)")
        print("  Genres: \(movie.genres.joined(separator: ", "))")
    }
}
```

### Nested Arrays

```swift
import Foundation

struct Matrix: Codable {
    var rows: [[Double]]  // 2D array
    var name: String
}

struct Classroom: Codable {
    var name: String
    var students: [Student]
    var schedule: [[String]]  // 2D string array
    
    struct Student: Codable {
        var name: String
        var grades: [Int]
    }
}

let json = """
{
    "name": "ห้อง 5/1",
    "students": [
        {"name": "สมชาย", "grades": [85, 90, 78, 92]},
        {"name": "สมหญิง", "grades": [92, 88, 95, 87]},
        {"name": "วิชัย", "grades": [75, 80, 70, 85]}
    ],
    "schedule": [
        ["คณิตศาสตร์", "วิทยาศาสตร์", "ภาษาไทย"],
        ["ภาษาอังกฤษ", "สังคม", "พลศึกษา"]
    ]
}
"""

let decoder = JSONDecoder()
if let data = json.data(using: .utf8),
   let classroom = try? decoder.decode(Classroom.self, from: data) {
    print("ห้องเรียน: \(classroom.name)")
    print("\nนักเรียน:")
    for student in classroom.students {
        let avg = Double(student.grades.reduce(0, +)) / Double(student.grades.count)
        print("  \(student.name) - เฉลี่ย: \(String(format: "%.1f", avg))")
    }
    print("\nตารางเรียน:")
    for (i, day) in classroom.schedule.enumerated() {
        print("  วัน \(i+1): \(day.joined(separator: ", "))")
    }
}
```

### Array ของ Enums

```swift
import Foundation

enum DayOfWeek: String, Codable, CaseIterable {
    case monday = "Mon"
    case tuesday = "Tue"
    case wednesday = "Wed"
    case thursday = "Thu"
    case friday = "Fri"
    case saturday = "Sat"
    case sunday = "Sun"
}

struct WorkSchedule: Codable {
    var employeeName: String
    var workDays: [DayOfWeek]
    
    enum CodingKeys: String, CodingKey {
        case employeeName = "employee_name"
        case workDays = "work_days"
    }
}

let json = """
{
    "employee_name": "สมชาย",
    "work_days": ["Mon", "Tue", "Wed", "Thu", "Fri"]
}
"""

let decoder = JSONDecoder()
if let data = json.data(using: .utf8),
   let schedule = try? decoder.decode(WorkSchedule.self, from: data) {
    print("พนักงาน: \(schedule.employeeName)")
    print("วันทำงาน: \(schedule.workDays.map { $0.rawValue }.joined(separator: ", "))")
}
```

---

## Optional JSON Fields

การจัดการกับ JSON fields ที่อาจมีหรือไม่มีก็ได้

### Optional Properties

```swift
import Foundation

struct UserProfile: Codable {
    var id: Int
    var username: String
    var bio: String?            // Optional
    var profilePicture: URL?    // Optional URL
    var websiteURL: String?     // Optional
    var age: Int?               // Optional
}

// JSON ที่ขาด optional fields
let jsonMinimal = """
{
    "id": 1,
    "username": "john_doe"
}
"""

// JSON ที่มีครบ
let jsonFull = """
{
    "id": 2,
    "username": "jane_doe",
    "bio": "iOS Developer",
    "profilePicture": "https://example.com/avatar.jpg",
    "websiteURL": "https://jane.dev",
    "age": 28
}
"""

let decoder = JSONDecoder()

if let data = jsonMinimal.data(using: .utf8),
   let profile = try? decoder.decode(UserProfile.self, from: data) {
    print("Minimal Profile:")
    print("  ID: \(profile.id)")
    print("  Username: \(profile.username)")
    print("  Bio: \(profile.bio ?? "ไม่มี")")
    print("  Age: \(profile.age.map { String($0) } ?? "ไม่ระบุ")")
}

if let data = jsonFull.data(using: .utf8),
   let profile = try? decoder.decode(UserProfile.self, from: data) {
    print("\nFull Profile:")
    print("  ID: \(profile.id)")
    print("  Username: \(profile.username)")
    if let bio = profile.bio {
        print("  Bio: \(bio)")
    }
    if let age = profile.age {
        print("  Age: \(age)")
    }
}
```

### Null vs Missing Fields

```swift
import Foundation

struct Config: Codable {
    var setting1: String?    // อาจเป็น null หรือไม่มี field เลย
    var setting2: String?    // เหมือนกัน
}

// null value
let jsonWithNull = """{"setting1": null, "setting2": "value"}"""
// missing field
let jsonMissing = """{"setting2": "value"}"""
// มีทั้งคู่
let jsonBoth = """{"setting1": "val1", "setting2": "val2"}"""

let decoder = JSONDecoder()

for jsonString in [jsonWithNull, jsonMissing, jsonBoth] {
    if let data = jsonString.data(using: .utf8),
       let config = try? decoder.decode(Config.self, from: data) {
        print("setting1: \(config.setting1 ?? "nil"), setting2: \(config.setting2 ?? "nil")")
    }
}
// ทั้ง null และ missing จะได้ nil เหมือนกัน
```

### Default Values สำหรับ Missing Fields

```swift
import Foundation

struct AppSettings: Decodable {
    var theme: String
    var fontSize: Int
    var notificationsEnabled: Bool
    var language: String
    
    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        
        // ถ้าไม่มี field ใช้ default value
        theme = try container.decodeIfPresent(String.self, forKey: .theme) ?? "light"
        fontSize = try container.decodeIfPresent(Int.self, forKey: .fontSize) ?? 16
        notificationsEnabled = try container.decodeIfPresent(Bool.self, forKey: .notificationsEnabled) ?? true
        language = try container.decodeIfPresent(String.self, forKey: .language) ?? "th"
    }
    
    enum CodingKeys: String, CodingKey {
        case theme
        case fontSize = "font_size"
        case notificationsEnabled = "notifications_enabled"
        case language
    }
}

let jsonPartial = """
{
    "theme": "dark",
    "language": "en"
}
"""

let decoder = JSONDecoder()
if let data = jsonPartial.data(using: .utf8),
   let settings = try? decoder.decode(AppSettings.self, from: data) {
    print("Theme: \(settings.theme)")              // dark
    print("Font Size: \(settings.fontSize)")        // 16 (default)
    print("Notifications: \(settings.notificationsEnabled)")  // true (default)
    print("Language: \(settings.language)")         // en
}
```

---

## Date Encoding Strategies

Swift รองรับหลาย strategy ในการ encode/decode `Date`

### Built-in Date Strategies

```swift
import Foundation

struct Event: Codable {
    var name: String
    var date: Date
    var endDate: Date?
}

// 1. deferredToDate (default) - seconds since 2001-01-01
let encoder1 = JSONEncoder()
encoder1.outputFormatting = .prettyPrinted
// encoder1.dateEncodingStrategy = .deferredToDate  // default

// 2. secondsSince1970 - Unix timestamp
let encoder2 = JSONEncoder()
encoder2.outputFormatting = .prettyPrinted
encoder2.dateEncodingStrategy = .secondsSince1970

// 3. millisecondsSince1970
let encoder3 = JSONEncoder()
encoder3.outputFormatting = .prettyPrinted
encoder3.dateEncodingStrategy = .millisecondsSince1970

// 4. iso8601 - "2023-09-01T12:00:00Z"
let encoder4 = JSONEncoder()
encoder4.outputFormatting = .prettyPrinted
encoder4.dateEncodingStrategy = .iso8601

let event = Event(name: "Swift Conference", date: Date(), endDate: nil)

print("Deferred to Date:")
if let data = try? encoder1.encode(event),
   let json = String(data: data, encoding: .utf8) { print(json) }

print("\nSeconds Since 1970:")
if let data = try? encoder2.encode(event),
   let json = String(data: data, encoding: .utf8) { print(json) }

print("\nISO 8601:")
if let data = try? encoder4.encode(event),
   let json = String(data: data, encoding: .utf8) { print(json) }
```

### Custom Date Format

```swift
import Foundation

struct Appointment: Codable {
    var title: String
    var date: Date
    var reminderDate: Date?
}

// สร้าง DateFormatter
let dateFormatter = DateFormatter()
dateFormatter.dateFormat = "yyyy-MM-dd HH:mm:ss"
dateFormatter.timeZone = TimeZone(identifier: "Asia/Bangkok")

let encoder = JSONEncoder()
encoder.outputFormatting = .prettyPrinted
encoder.dateEncodingStrategy = .formatted(dateFormatter)

let decoder = JSONDecoder()
decoder.dateDecodingStrategy = .formatted(dateFormatter)

let appointment = Appointment(
    title: "นัดหมายแพทย์",
    date: Date(),
    reminderDate: Date().addingTimeInterval(-3600)  // 1 ชั่วโมงก่อน
)

if let data = try? encoder.encode(appointment),
   let json = String(data: data, encoding: .utf8) {
    print("Encoded:")
    print(json)
    
    // Decode กลับ
    if let decoded = try? decoder.decode(Appointment.self, from: data) {
        print("\nDecoded date: \(decoded.date)")
    }
}
```

### Custom Date Decoding

```swift
import Foundation

// API บางตัวส่ง date ในรูปแบบแปลกๆ
struct APIDate: Decodable {
    var createdAt: Date
    
    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        
        // อ่าน string ก่อน
        let dateString = try container.decode(String.self, forKey: .createdAt)
        
        // ลองหลาย format
        let formats = [
            "yyyy-MM-dd'T'HH:mm:ssZ",
            "yyyy-MM-dd HH:mm:ss",
            "yyyy-MM-dd",
            "dd/MM/yyyy"
        ]
        
        var parsedDate: Date? = nil
        let formatter = DateFormatter()
        
        for format in formats {
            formatter.dateFormat = format
            if let date = formatter.date(from: dateString) {
                parsedDate = date
                break
            }
        }
        
        guard let date = parsedDate else {
            throw DecodingError.dataCorruptedError(
                forKey: .createdAt,
                in: container,
                debugDescription: "Cannot parse date: \(dateString)"
            )
        }
        
        createdAt = date
    }
    
    enum CodingKeys: String, CodingKey {
        case createdAt = "created_at"
    }
}

// ทดสอบกับหลาย format
let dates = [
    """{"created_at": "2023-09-01T12:00:00+0700"}""",
    """{"created_at": "2023-09-01 12:00:00"}""",
    """{"created_at": "2023-09-01"}""",
    """{"created_at": "01/09/2023"}"""
]

let decoder = JSONDecoder()
for jsonString in dates {
    if let data = jsonString.data(using: .utf8),
       let apiDate = try? decoder.decode(APIDate.self, from: data) {
        print("Parsed: \(apiDate.createdAt)")
    }
}
```

---

## Data Encoding Strategies

การ encode/decode `Data` type

```swift
import Foundation

struct FileUpload: Codable {
    var filename: String
    var content: Data
    var thumbnail: Data?
}

// สร้าง sample data
let sampleText = "Hello, Swift!"
let sampleData = sampleText.data(using: .utf8)!

let upload = FileUpload(
    filename: "test.txt",
    content: sampleData,
    thumbnail: nil
)

// 1. base64 (default)
let encoder1 = JSONEncoder()
encoder1.outputFormatting = .prettyPrinted
encoder1.dataEncodingStrategy = .base64

if let data = try? encoder1.encode(upload),
   let json = String(data: data, encoding: .utf8) {
    print("Base64 encoding:")
    print(json)
}

// 2. Custom encoding
let encoder2 = JSONEncoder()
encoder2.outputFormatting = .prettyPrinted
encoder2.dataEncodingStrategy = .custom { data, encoder in
    var container = encoder.singleValueContainer()
    // Encode เป็น hex string
    let hexString = data.map { String(format: "%02x", $0) }.joined()
    try container.encode(hexString)
}

if let data = try? encoder2.encode(upload),
   let json = String(data: data, encoding: .utf8) {
    print("\nCustom (hex) encoding:")
    print(json)
}

// Decode
let decoder = JSONDecoder()
decoder.dataDecodingStrategy = .base64

if let encodedData = try? encoder1.encode(upload),
   let decoded = try? decoder.decode(FileUpload.self, from: encodedData) {
    let decodedText = String(data: decoded.content, encoding: .utf8)!
    print("\nDecoded content: \(decodedText)")
}
```

---

## Key Decoding Strategies

Strategy สำหรับ convert key names ระหว่าง JSON และ Swift

```swift
import Foundation

struct ServerResponse: Codable {
    var userId: Int
    var firstName: String
    var lastName: String
    var emailAddress: String
    var createdAt: String
    var isActiveUser: Bool
}

// JSON ที่ใช้ snake_case
let jsonSnakeCase = """
{
    "user_id": 123,
    "first_name": "สมชาย",
    "last_name": "ใจดี",
    "email_address": "somchai@example.com",
    "created_at": "2023-01-15",
    "is_active_user": true
}
"""

// ใช้ convertFromSnakeCase strategy
let decoder = JSONDecoder()
decoder.keyDecodingStrategy = .convertFromSnakeCase

if let data = jsonSnakeCase.data(using: .utf8),
   let response = try? decoder.decode(ServerResponse.self, from: data) {
    print("User ID: \(response.userId)")
    print("Name: \(response.firstName) \(response.lastName)")
    print("Email: \(response.emailAddress)")
    print("Created: \(response.createdAt)")
    print("Active: \(response.isActiveUser)")
}

// Encode กลับเป็น snake_case
let encoder = JSONEncoder()
encoder.outputFormatting = .prettyPrinted
encoder.keyEncodingStrategy = .convertToSnakeCase

let obj = ServerResponse(
    userId: 456,
    firstName: "มาลี",
    lastName: "รักไทย",
    emailAddress: "malee@example.com",
    createdAt: "2023-05-20",
    isActiveUser: false
)

if let data = try? encoder.encode(obj),
   let json = String(data: data, encoding: .utf8) {
    print("\nEncoded with snake_case:")
    print(json)
}
```

### Custom Key Strategy

```swift
import Foundation

// Custom strategy - convert to uppercase
let encoder = JSONEncoder()
encoder.outputFormatting = .prettyPrinted
encoder.keyEncodingStrategy = .custom { codingKeys in
    let lastKey = codingKeys.last!
    return AnyCodingKey(stringValue: lastKey.stringValue.uppercased())!
}

struct Config: Codable {
    var apiKey: String
    var baseUrl: String
    var timeout: Int
}

let config = Config(apiKey: "abc123", baseUrl: "https://api.example.com", timeout: 30)

if let data = try? encoder.encode(config),
   let json = String(data: data, encoding: .utf8) {
    print("Uppercase keys:")
    print(json)
}

// AnyCodingKey helper
struct AnyCodingKey: CodingKey {
    var stringValue: String
    var intValue: Int?
    
    init?(stringValue: String) {
        self.stringValue = stringValue
        self.intValue = nil
    }
    
    init?(intValue: Int) {
        self.intValue = intValue
        self.stringValue = String(intValue)
    }
}
```

---

## Handling Unknown JSON Keys

การจัดการกับ JSON keys ที่ไม่รู้จัก

```swift
import Foundation

// โดยปกติ JSONDecoder จะ ignore unknown keys
struct KnownFields: Codable {
    var name: String
    var age: Int
    // ไม่มี field "unknownField"
}

let jsonWithExtra = """
{
    "name": "สมชาย",
    "age": 30,
    "unknownField": "some value",
    "anotherUnknown": 42
}
"""

// ใช้งานได้ปกติ - unknown keys ถูก ignore
let decoder = JSONDecoder()
if let data = jsonWithExtra.data(using: .utf8),
   let obj = try? decoder.decode(KnownFields.self, from: data) {
    print("Name: \(obj.name), Age: \(obj.age)")
}
```

### เก็บ Unknown Keys ด้วย Custom Decoder

```swift
import Foundation

struct FlexibleConfig: Decodable {
    var name: String
    var version: String
    var additionalConfig: [String: AnyDecodable]  // เก็บ keys ที่ไม่รู้จัก
    
    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: AnyCodingKey.self)
        
        name = try container.decode(String.self, forKey: AnyCodingKey(stringValue: "name")!)
        version = try container.decode(String.self, forKey: AnyCodingKey(stringValue: "version")!)
        
        // เก็บ keys ที่เหลือ
        var additional: [String: AnyDecodable] = [:]
        let knownKeys = Set(["name", "version"])
        
        for key in container.allKeys {
            if !knownKeys.contains(key.stringValue) {
                if let value = try? container.decode(AnyDecodable.self, forKey: key) {
                    additional[key.stringValue] = value
                }
            }
        }
        additionalConfig = additional
    }
}

// AnyDecodable - wrapper สำหรับ decode ค่าใดๆ
struct AnyDecodable: Decodable {
    var value: Any
    
    init(from decoder: Decoder) throws {
        let container = try decoder.singleValueContainer()
        
        if let intValue = try? container.decode(Int.self) {
            value = intValue
        } else if let doubleValue = try? container.decode(Double.self) {
            value = doubleValue
        } else if let stringValue = try? container.decode(String.self) {
            value = stringValue
        } else if let boolValue = try? container.decode(Bool.self) {
            value = boolValue
        } else {
            value = NSNull()
        }
    }
}

struct AnyCodingKey: CodingKey {
    var stringValue: String
    var intValue: Int?
    
    init?(stringValue: String) {
        self.stringValue = stringValue
        self.intValue = nil
    }
    
    init?(intValue: Int) {
        self.intValue = intValue
        self.stringValue = String(intValue)
    }
}

let json = """
{
    "name": "MyApp",
    "version": "1.0.0",
    "debug": true,
    "maxRetries": 3,
    "apiEndpoint": "https://api.example.com"
}
"""

let decoder = JSONDecoder()
if let data = json.data(using: .utf8),
   let config = try? decoder.decode(FlexibleConfig.self, from: data) {
    print("Name: \(config.name)")
    print("Version: \(config.version)")
    print("Additional config:")
    for (key, val) in config.additionalConfig {
        print("  \(key): \(val.value)")
    }
}
```

---

## JSONSerialization (Legacy)

ก่อน Swift 4 เราใช้ `JSONSerialization` ซึ่งยังคงมีประโยชน์ในบางกรณี

```swift
import Foundation

// Serialize (Swift -> JSON)
let dictionary: [String: Any] = [
    "name": "สมชาย",
    "age": 30,
    "hobbies": ["อ่านหนังสือ", "เล่นกีตาร์"],
    "address": [
        "city": "กรุงเทพ",
        "country": "Thailand"
    ]
]

do {
    let jsonData = try JSONSerialization.data(
        withJSONObject: dictionary,
        options: [.prettyPrinted, .sortedKeys]
    )
    
    if let jsonString = String(data: jsonData, encoding: .utf8) {
        print("Serialized JSON:")
        print(jsonString)
    }
} catch {
    print("Serialization error: \(error)")
}

// Deserialize (JSON -> Swift)
let jsonString = """
{
    "products": [
        {"id": 1, "name": "iPhone", "price": 35900},
        {"id": 2, "name": "iPad", "price": 25900}
    ],
    "total": 61800,
    "currency": "THB"
}
"""

if let jsonData = jsonString.data(using: .utf8) {
    do {
        if let jsonObject = try JSONSerialization.jsonObject(
            with: jsonData,
            options: []
        ) as? [String: Any] {
            
            if let products = jsonObject["products"] as? [[String: Any]] {
                print("\nProducts:")
                for product in products {
                    let id = product["id"] as? Int ?? 0
                    let name = product["name"] as? String ?? ""
                    let price = product["price"] as? Double ?? 0
                    print("  \(id). \(name) - ฿\(price)")
                }
            }
            
            if let total = jsonObject["total"] as? Int {
                print("Total: ฿\(total)")
            }
        }
    } catch {
        print("Deserialization error: \(error)")
    }
}
```

### เมื่อใช้ JSONSerialization แทน Codable

```swift
import Foundation

// กรณีที่ JSONSerialization เหมาะกว่า:

// 1. JSON structure ไม่แน่นอน / dynamic
func processUnknownJSON(_ jsonString: String) {
    guard let data = jsonString.data(using: .utf8),
          let jsonObject = try? JSONSerialization.jsonObject(with: data) else {
        return
    }
    
    func printValue(_ value: Any, indent: String = "") {
        switch value {
        case let dict as [String: Any]:
            for (key, val) in dict {
                print("\(indent)\(key):")
                printValue(val, indent: indent + "  ")
            }
        case let array as [Any]:
            for (i, item) in array.enumerated() {
                print("\(indent)[\(i)]:")
                printValue(item, indent: indent + "  ")
            }
        case let string as String:
            print("\(indent)\(string)")
        case let number as NSNumber:
            print("\(indent)\(number)")
        default:
            print("\(indent)\(value)")
        }
    }
    
    printValue(jsonObject)
}

// 2. Validate JSON
func isValidJSON(_ string: String) -> Bool {
    guard let data = string.data(using: .utf8) else { return false }
    return (try? JSONSerialization.jsonObject(with: data)) != nil
}

print(isValidJSON("""{"valid": true}"""))  // true
print(isValidJSON("""{"invalid: true}"""))  // false
```

---

## PropertyListEncoder/Decoder

นอกจาก JSON Swift ยังรองรับ Property List format ซึ่งใช้ใน iOS/macOS

```swift
import Foundation

struct AppPreferences: Codable {
    var theme: String
    var language: String
    var fontSize: Int
    var showNotifications: Bool
    var recentSearches: [String]
}

let prefs = AppPreferences(
    theme: "dark",
    language: "th",
    fontSize: 16,
    showNotifications: true,
    recentSearches: ["Swift", "iOS", "Xcode"]
)

// Encode เป็น Property List
let plistEncoder = PropertyListEncoder()
plistEncoder.outputFormat = .xml  // หรือ .binary

do {
    let plistData = try plistEncoder.encode(prefs)
    
    // บันทึกลง UserDefaults
    UserDefaults.standard.set(plistData, forKey: "appPreferences")
    print("Saved to UserDefaults")
    
    // อ่านกลับจาก UserDefaults
    if let savedData = UserDefaults.standard.data(forKey: "appPreferences") {
        let plistDecoder = PropertyListDecoder()
        let decoded = try plistDecoder.decode(AppPreferences.self, from: savedData)
        print("Theme: \(decoded.theme)")
        print("Language: \(decoded.language)")
        print("Font Size: \(decoded.fontSize)")
        print("Recent searches: \(decoded.recentSearches)")
    }
} catch {
    print("Error: \(error)")
}

// ดู XML format
let xmlEncoder = PropertyListEncoder()
xmlEncoder.outputFormat = .xml
if let data = try? xmlEncoder.encode(prefs),
   let xmlString = String(data: data, encoding: .utf8) {
    print("\nXML Property List:")
    print(xmlString)
}
```

---

## Custom CodingKey

สร้าง CodingKey แบบ custom เพื่อความยืดหยุ่นมากขึ้น

```swift
import Foundation

// Dynamic CodingKey สำหรับ dictionary-like structures
struct DynamicCodingKey: CodingKey {
    var stringValue: String
    var intValue: Int?
    
    init?(stringValue: String) {
        self.stringValue = stringValue
        self.intValue = nil
    }
    
    init?(intValue: Int) {
        self.intValue = intValue
        self.stringValue = String(intValue)
    }
}

// ใช้ decode dictionary ที่ keys ไม่รู้จัก
struct LocalizedString: Decodable {
    var translations: [String: String]
    
    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: DynamicCodingKey.self)
        var result: [String: String] = [:]
        
        for key in container.allKeys {
            if let value = try? container.decode(String.self, forKey: key) {
                result[key.stringValue] = value
            }
        }
        translations = result
    }
}

let json = """
{
    "en": "Hello",
    "th": "สวัสดี",
    "ja": "こんにちは",
    "zh": "你好",
    "ko": "안녕하세요"
}
"""

let decoder = JSONDecoder()
if let data = json.data(using: .utf8),
   let localized = try? decoder.decode(LocalizedString.self, from: data) {
    print("Translations:")
    for (lang, text) in localized.translations.sorted(by: { $0.key < $1.key }) {
        print("  \(lang): \(text)")
    }
}
```

---

## Encoding Inheritance

การใช้ Codable กับ class hierarchy

```swift
import Foundation

class Vehicle: Codable {
    var make: String
    var model: String
    var year: Int
    var color: String
    
    init(make: String, model: String, year: Int, color: String) {
        self.make = make
        self.model = model
        self.year = year
        self.color = color
    }
    
    enum CodingKeys: String, CodingKey {
        case make, model, year, color
    }
    
    required init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        make = try container.decode(String.self, forKey: .make)
        model = try container.decode(String.self, forKey: .model)
        year = try container.decode(Int.self, forKey: .year)
        color = try container.decode(String.self, forKey: .color)
    }
    
    func encode(to encoder: Encoder) throws {
        var container = encoder.container(keyedBy: CodingKeys.self)
        try container.encode(make, forKey: .make)
        try container.encode(model, forKey: .model)
        try container.encode(year, forKey: .year)
        try container.encode(color, forKey: .color)
    }
}

class ElectricCar: Vehicle {
    var batteryCapacity: Double  // kWh
    var range: Int               // km
    
    init(make: String, model: String, year: Int, color: String,
         batteryCapacity: Double, range: Int) {
        self.batteryCapacity = batteryCapacity
        self.range = range
        super.init(make: make, model: model, year: year, color: color)
    }
    
    enum CarCodingKeys: String, CodingKey {
        case batteryCapacity = "battery_capacity"
        case range
    }
    
    required init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CarCodingKeys.self)
        batteryCapacity = try container.decode(Double.self, forKey: .batteryCapacity)
        range = try container.decode(Int.self, forKey: .range)
        try super.init(from: decoder)
    }
    
    override func encode(to encoder: Encoder) throws {
        try super.encode(to: encoder)
        var container = encoder.container(keyedBy: CarCodingKeys.self)
        try container.encode(batteryCapacity, forKey: .batteryCapacity)
        try container.encode(range, forKey: .range)
    }
}

let tesla = ElectricCar(
    make: "Tesla",
    model: "Model 3",
    year: 2023,
    color: "White",
    batteryCapacity: 75.0,
    range: 600
)

let encoder = JSONEncoder()
encoder.outputFormatting = .prettyPrinted

if let data = try? encoder.encode(tesla),
   let json = String(data: data, encoding: .utf8) {
    print("Electric Car JSON:")
    print(json)
    
    // Decode กลับ
    let decoder = JSONDecoder()
    if let decodedCar = try? decoder.decode(ElectricCar.self, from: data) {
        print("\nDecoded: \(decodedCar.make) \(decodedCar.model)")
        print("Battery: \(decodedCar.batteryCapacity)kWh")
        print("Range: \(decodedCar.range)km")
    }
}
```

---

## Polymorphic Decoding

การ decode objects ที่มี type ต่างกันจาก JSON เดียวกัน

```swift
import Foundation

// Pattern 1: ใช้ type discriminator
protocol Shape: Codable {
    var type: String { get }
    var area: Double { get }
}

struct Circle: Shape {
    let type = "circle"
    var radius: Double
    var area: Double { Double.pi * radius * radius }
    
    enum CodingKeys: String, CodingKey {
        case type, radius
    }
}

struct Rectangle: Shape {
    let type = "rectangle"
    var width: Double
    var height: Double
    var area: Double { width * height }
    
    enum CodingKeys: String, CodingKey {
        case type, width, height
    }
}

struct Triangle: Shape {
    let type = "triangle"
    var base: Double
    var height: Double
    var area: Double { 0.5 * base * height }
    
    enum CodingKeys: String, CodingKey {
        case type, base, height
    }
}

// Custom decoder สำหรับ polymorphic decoding
enum AnyShape: Decodable {
    case circle(Circle)
    case rectangle(Rectangle)
    case triangle(Triangle)
    case unknown
    
    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: TypeKey.self)
        let type = try container.decode(String.self, forKey: .type)
        
        switch type {
        case "circle":
            self = .circle(try Circle(from: decoder))
        case "rectangle":
            self = .rectangle(try Rectangle(from: decoder))
        case "triangle":
            self = .triangle(try Triangle(from: decoder))
        default:
            self = .unknown
        }
    }
    
    enum TypeKey: String, CodingKey {
        case type
    }
    
    var area: Double {
        switch self {
        case .circle(let c): return c.area
        case .rectangle(let r): return r.area
        case .triangle(let t): return t.area
        case .unknown: return 0
        }
    }
}

let shapesJSON = """
[
    {"type": "circle", "radius": 5.0},
    {"type": "rectangle", "width": 10.0, "height": 4.0},
    {"type": "triangle", "base": 6.0, "height": 8.0}
]
"""

let decoder = JSONDecoder()
if let data = shapesJSON.data(using: .utf8),
   let shapes = try? decoder.decode([AnyShape].self, from: data) {
    print("Shapes:")
    for shape in shapes {
        switch shape {
        case .circle(let c):
            print("  Circle (r=\(c.radius)): area = \(String(format: "%.2f", c.area))")
        case .rectangle(let r):
            print("  Rectangle (\(r.width)x\(r.height)): area = \(r.area)")
        case .triangle(let t):
            print("  Triangle (b=\(t.base), h=\(t.height)): area = \(t.area)")
        case .unknown:
            print("  Unknown shape")
        }
    }
}
```

---

## JSON with Generics

การใช้ Codable กับ Generic types

```swift
import Foundation

// Generic Response wrapper
struct APIResponse<T: Codable>: Codable {
    var success: Bool
    var message: String
    var data: T?
    var errorCode: Int?
    
    enum CodingKeys: String, CodingKey {
        case success, message, data
        case errorCode = "error_code"
    }
}

// ใช้กับ type ต่างๆ
struct User: Codable {
    var id: Int
    var name: String
    var email: String
}

struct Product: Codable {
    var id: Int
    var name: String
    var price: Double
}

// Decode user response
let userJSON = """
{
    "success": true,
    "message": "User found",
    "data": {
        "id": 1,
        "name": "สมชาย",
        "email": "somchai@example.com"
    }
}
"""

let decoder = JSONDecoder()

if let data = userJSON.data(using: .utf8),
   let response = try? decoder.decode(APIResponse<User>.self, from: data) {
    print("Success: \(response.success)")
    if let user = response.data {
        print("User: \(user.name) - \(user.email)")
    }
}

// Decode array response
let productsJSON = """
{
    "success": true,
    "message": "Products found",
    "data": [
        {"id": 1, "name": "iPhone", "price": 35900},
        {"id": 2, "name": "iPad", "price": 25900}
    ]
}
"""

if let data = productsJSON.data(using: .utf8),
   let response = try? decoder.decode(APIResponse<[Product]>.self, from: data) {
    print("\nProducts:")
    for product in response.data ?? [] {
        print("  \(product.name): ฿\(product.price)")
    }
}
```

### Generic Pagination

```swift
import Foundation

struct PaginatedResponse<T: Codable>: Codable {
    var items: [T]
    var totalCount: Int
    var currentPage: Int
    var totalPages: Int
    var hasNextPage: Bool
    
    enum CodingKeys: String, CodingKey {
        case items
        case totalCount = "total_count"
        case currentPage = "current_page"
        case totalPages = "total_pages"
        case hasNextPage = "has_next_page"
    }
}

struct Article: Codable {
    var id: Int
    var title: String
    var author: String
    var publishedAt: String
    
    enum CodingKeys: String, CodingKey {
        case id, title, author
        case publishedAt = "published_at"
    }
}

let paginatedJSON = """
{
    "items": [
        {"id": 1, "title": "Swift 5.9 Features", "author": "John", "published_at": "2023-09-01"},
        {"id": 2, "title": "iOS 17 Updates", "author": "Jane", "published_at": "2023-09-05"}
    ],
    "total_count": 100,
    "current_page": 1,
    "total_pages": 50,
    "has_next_page": true
}
"""

let decoder = JSONDecoder()
if let data = paginatedJSON.data(using: .utf8),
   let response = try? decoder.decode(PaginatedResponse<Article>.self, from: data) {
    print("Page \(response.currentPage) of \(response.totalPages)")
    print("Total articles: \(response.totalCount)")
    print("\nArticles:")
    for article in response.items {
        print("  [\(article.id)] \(article.title) by \(article.author)")
    }
    if response.hasNextPage {
        print("\nMore articles available...")
    }
}
```

---

## Error Handling ใน Codable

การจัดการ errors ที่เกิดขึ้นระหว่าง encoding/decoding

### DecodingError Types

```swift
import Foundation

struct StrictModel: Codable {
    var name: String
    var age: Int
    var score: Double
}

func decodeJSON(_ jsonString: String) {
    guard let data = jsonString.data(using: .utf8) else {
        print("Cannot convert string to data")
        return
    }
    
    let decoder = JSONDecoder()
    
    do {
        let model = try decoder.decode(StrictModel.self, from: data)
        print("Decoded: \(model.name), \(model.age), \(model.score)")
    } catch DecodingError.typeMismatch(let type, let context) {
        print("Type mismatch error:")
        print("  Expected: \(type)")
        print("  Path: \(context.codingPath.map { $0.stringValue }.joined(separator: "."))")
        print("  Description: \(context.debugDescription)")
        
    } catch DecodingError.valueNotFound(let type, let context) {
        print("Value not found:")
        print("  Type: \(type)")
        print("  Path: \(context.codingPath.map { $0.stringValue }.joined(separator: "."))")
        
    } catch DecodingError.keyNotFound(let key, let context) {
        print("Key not found:")
        print("  Key: \(key.stringValue)")
        print("  Path: \(context.codingPath.map { $0.stringValue }.joined(separator: "."))")
        
    } catch DecodingError.dataCorrupted(let context) {
        print("Data corrupted:")
        print("  Path: \(context.codingPath.map { $0.stringValue }.joined(separator: "."))")
        print("  Description: \(context.debugDescription)")
        
    } catch {
        print("Other error: \(error)")
    }
}

// ทดสอบ errors ต่างๆ
print("=== Type Mismatch ===")
decodeJSON("""{"name": "สมชาย", "age": "thirty", "score": 95.5}""")  // age ควรเป็น Int

print("\n=== Key Not Found ===")
decodeJSON("""{"name": "สมชาย", "score": 95.5}""")  // ขาด age

print("\n=== Data Corrupted ===")
decodeJSON("""{"name": สมชาย, "age": 30}""")  // JSON ไม่ valid
```

### Custom Error Messages

```swift
import Foundation

enum ValidationError: Error, LocalizedError {
    case invalidAge(Int)
    case emptyName
    case invalidEmail(String)
    
    var errorDescription: String? {
        switch self {
        case .invalidAge(let age):
            return "อายุ \(age) ไม่ถูกต้อง (ต้องอยู่ระหว่าง 0-150)"
        case .emptyName:
            return "ชื่อไม่สามารถเว้นว่างได้"
        case .invalidEmail(let email):
            return "อีเมล '\(email)' ไม่ถูกต้อง"
        }
    }
}

struct ValidatedUser: Decodable {
    var name: String
    var age: Int
    var email: String
    
    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        
        // Decode name
        let rawName = try container.decode(String.self, forKey: .name)
        if rawName.isEmpty {
            throw ValidationError.emptyName
        }
        name = rawName.trimmingCharacters(in: .whitespaces)
        
        // Decode age with validation
        let rawAge = try container.decode(Int.self, forKey: .age)
        if rawAge < 0 || rawAge > 150 {
            throw ValidationError.invalidAge(rawAge)
        }
        age = rawAge
        
        // Decode email with validation
        let rawEmail = try container.decode(String.self, forKey: .email)
        if !rawEmail.contains("@") || !rawEmail.contains(".") {
            throw ValidationError.invalidEmail(rawEmail)
        }
        email = rawEmail
    }
    
    enum CodingKeys: String, CodingKey {
        case name, age, email
    }
}

func testValidation(_ json: String) {
    guard let data = json.data(using: .utf8) else { return }
    let decoder = JSONDecoder()
    
    do {
        let user = try decoder.decode(ValidatedUser.self, from: data)
        print("Valid user: \(user.name), \(user.age)")
    } catch let validationError as ValidationError {
        print("Validation failed: \(validationError.localizedDescription)")
    } catch {
        print("Decode error: \(error)")
    }
}

testValidation("""{"name": "สมชาย", "age": 30, "email": "somchai@example.com"}""")
testValidation("""{"name": "", "age": 30, "email": "somchai@example.com"}""")
testValidation("""{"name": "สมชาย", "age": -5, "email": "somchai@example.com"}""")
testValidation("""{"name": "สมชาย", "age": 30, "email": "invalidemail"}""")
```

---

## Testing Codable Implementations

การทดสอบ Codable implementations ให้ถูกต้อง

```swift
import XCTest
import Foundation

struct Product: Codable, Equatable {
    var id: Int
    var name: String
    var price: Double
    var inStock: Bool
    var tags: [String]
    
    enum CodingKeys: String, CodingKey {
        case id, name, price
        case inStock = "in_stock"
        case tags
    }
}

class ProductCodableTests: XCTestCase {
    
    let encoder: JSONEncoder = {
        let enc = JSONEncoder()
        enc.outputFormatting = .sortedKeys
        return enc
    }()
    
    let decoder = JSONDecoder()
    
    // Test encoding
    func testEncoding() throws {
        let product = Product(
            id: 1,
            name: "MacBook Pro",
            price: 79900.0,
            inStock: true,
            tags: ["laptop", "apple"]
        )
        
        let data = try encoder.encode(product)
        let json = try JSONSerialization.jsonObject(with: data) as! [String: Any]
        
        XCTAssertEqual(json["id"] as? Int, 1)
        XCTAssertEqual(json["name"] as? String, "MacBook Pro")
        XCTAssertEqual(json["price"] as? Double, 79900.0)
        XCTAssertEqual(json["in_stock"] as? Bool, true)  // snake_case
        XCTAssertEqual(json["tags"] as? [String], ["laptop", "apple"])
    }
    
    // Test decoding
    func testDecoding() throws {
        let json = """
        {
            "id": 2,
            "name": "iPhone 15",
            "price": 35900.0,
            "in_stock": false,
            "tags": ["phone", "apple", "mobile"]
        }
        """.data(using: .utf8)!
        
        let product = try decoder.decode(Product.self, from: json)
        
        XCTAssertEqual(product.id, 2)
        XCTAssertEqual(product.name, "iPhone 15")
        XCTAssertEqual(product.price, 35900.0)
        XCTAssertFalse(product.inStock)
        XCTAssertEqual(product.tags.count, 3)
    }
    
    // Test round-trip (encode then decode)
    func testRoundTrip() throws {
        let original = Product(
            id: 3,
            name: "iPad Air",
            price: 25900.0,
            inStock: true,
            tags: ["tablet"]
        )
        
        let data = try encoder.encode(original)
        let decoded = try decoder.decode(Product.self, from: data)
        
        XCTAssertEqual(original, decoded)
    }
    
    // Test missing required key
    func testMissingRequiredKey() {
        let json = """{"id": 1, "name": "Test"}""".data(using: .utf8)!
        
        XCTAssertThrowsError(try decoder.decode(Product.self, from: json)) { error in
            guard case DecodingError.keyNotFound(let key, _) = error else {
                XCTFail("Expected keyNotFound error")
                return
            }
            XCTAssertEqual(key.stringValue, "price")
        }
    }
    
    // Test type mismatch
    func testTypeMismatch() {
        let json = """
        {"id": "not-a-number", "name": "Test", "price": 100, "in_stock": true, "tags": []}
        """.data(using: .utf8)!
        
        XCTAssertThrowsError(try decoder.decode(Product.self, from: json)) { error in
            if case DecodingError.typeMismatch = error {
                // Expected
            } else {
                XCTFail("Expected typeMismatch error")
            }
        }
    }
}
```

---

## Practical Exercises

### Exercise 1: Social Media Post

```swift
import Foundation

/*
 โจทย์: สร้าง Codable struct สำหรับ Social Media Post
 JSON ที่ต้องรองรับ:
 {
     "post_id": "abc123",
     "author": {
         "user_id": 1001,
         "username": "swift_dev",
         "display_name": "Swift Developer",
         "verified": true
     },
     "content": "เรียน Swift แล้วสนุกมาก!",
     "media": [
         {"url": "https://...", "type": "image"},
         {"url": "https://...", "type": "video"}
     ],
     "stats": {
         "likes": 1250,
         "comments": 89,
         "shares": 45,
         "views": 15000
     },
     "created_at": "2023-09-01T10:30:00Z",
     "hashtags": ["swift", "ios", "programming"],
     "is_pinned": false
 }
*/

// Solution:
struct Author: Codable {
    var userID: Int
    var username: String
    var displayName: String
    var isVerified: Bool
    
    enum CodingKeys: String, CodingKey {
        case userID = "user_id"
        case username
        case displayName = "display_name"
        case isVerified = "verified"
    }
}

struct MediaItem: Codable {
    var url: URL
    var type: MediaType
    
    enum MediaType: String, Codable {
        case image, video, gif
    }
}

struct PostStats: Codable {
    var likes: Int
    var comments: Int
    var shares: Int
    var views: Int
}

struct SocialPost: Codable {
    var postID: String
    var author: Author
    var content: String
    var media: [MediaItem]
    var stats: PostStats
    var createdAt: Date
    var hashtags: [String]
    var isPinned: Bool
    
    enum CodingKeys: String, CodingKey {
        case postID = "post_id"
        case author, content, media, stats
        case createdAt = "created_at"
        case hashtags
        case isPinned = "is_pinned"
    }
}

let postJSON = """
{
    "post_id": "abc123",
    "author": {
        "user_id": 1001,
        "username": "swift_dev",
        "display_name": "Swift Developer",
        "verified": true
    },
    "content": "เรียน Swift แล้วสนุกมาก!",
    "media": [
        {"url": "https://example.com/image.jpg", "type": "image"}
    ],
    "stats": {
        "likes": 1250,
        "comments": 89,
        "shares": 45,
        "views": 15000
    },
    "created_at": "2023-09-01T10:30:00Z",
    "hashtags": ["swift", "ios", "programming"],
    "is_pinned": false
}
"""

let decoder = JSONDecoder()
decoder.dateDecodingStrategy = .iso8601

if let data = postJSON.data(using: .utf8),
   let post = try? decoder.decode(SocialPost.self, from: data) {
    print("Post ID: \(post.postID)")
    print("Author: \(post.author.displayName) (@\(post.author.username))")
    print("Content: \(post.content)")
    print("Likes: \(post.stats.likes), Views: \(post.stats.views)")
    print("Hashtags: \(post.hashtags.map { "#\($0)" }.joined(separator: " "))")
}
```

### Exercise 2: Weather API Response

```swift
import Foundation

/*
 โจทย์: Decode weather API response
*/

struct WeatherData: Codable {
    var city: String
    var country: String
    var temperature: Temperature
    var humidity: Int
    var windSpeed: Double
    var conditions: [WeatherCondition]
    var forecast: [DailyForecast]
    var lastUpdated: Date
    
    struct Temperature: Codable {
        var current: Double
        var feelsLike: Double
        var min: Double
        var max: Double
        var unit: TemperatureUnit
        
        enum TemperatureUnit: String, Codable {
            case celsius = "C"
            case fahrenheit = "F"
        }
        
        enum CodingKeys: String, CodingKey {
            case current
            case feelsLike = "feels_like"
            case min, max, unit
        }
    }
    
    struct WeatherCondition: Codable {
        var id: Int
        var main: String
        var description: String
        var icon: String
    }
    
    struct DailyForecast: Codable {
        var date: String
        var tempMin: Double
        var tempMax: Double
        var condition: String
        var chanceOfRain: Double
        
        enum CodingKeys: String, CodingKey {
            case date
            case tempMin = "temp_min"
            case tempMax = "temp_max"
            case condition
            case chanceOfRain = "chance_of_rain"
        }
    }
    
    enum CodingKeys: String, CodingKey {
        case city, country, temperature, humidity
        case windSpeed = "wind_speed"
        case conditions, forecast
        case lastUpdated = "last_updated"
    }
}

let weatherJSON = """
{
    "city": "กรุงเทพมหานคร",
    "country": "TH",
    "temperature": {
        "current": 32.5,
        "feels_like": 38.2,
        "min": 28.0,
        "max": 35.0,
        "unit": "C"
    },
    "humidity": 75,
    "wind_speed": 15.5,
    "conditions": [
        {
            "id": 801,
            "main": "Clouds",
            "description": "few clouds",
            "icon": "02d"
        }
    ],
    "forecast": [
        {
            "date": "2023-09-01",
            "temp_min": 28.0,
            "temp_max": 35.0,
            "condition": "Partly Cloudy",
            "chance_of_rain": 0.2
        },
        {
            "date": "2023-09-02",
            "temp_min": 27.0,
            "temp_max": 33.0,
            "condition": "Rainy",
            "chance_of_rain": 0.8
        }
    ],
    "last_updated": 1693500000
}
"""

let decoder = JSONDecoder()
decoder.dateDecodingStrategy = .secondsSince1970

if let data = weatherJSON.data(using: .utf8),
   let weather = try? decoder.decode(WeatherData.self, from: data) {
    print("เมือง: \(weather.city), \(weather.country)")
    print("อุณหภูมิ: \(weather.temperature.current)°\(weather.temperature.unit.rawValue)")
    print("รู้สึกเหมือน: \(weather.temperature.feelsLike)°C")
    print("ความชื้น: \(weather.humidity)%")
    print("ลม: \(weather.windSpeed) km/h")
    print("\nพยากรณ์ 2 วัน:")
    for day in weather.forecast {
        let rain = Int(day.chanceOfRain * 100)
        print("  \(day.date): \(day.tempMin)°-\(day.tempMax)°C, \(day.condition), ฝน \(rain)%")
    }
}
```

### Exercise 3: E-Commerce Order

```swift
import Foundation

struct Order: Codable {
    var orderNumber: String
    var status: OrderStatus
    var customer: Customer
    var items: [OrderItem]
    var pricing: Pricing
    var shipping: ShippingInfo
    var createdAt: Date
    var updatedAt: Date
    
    enum OrderStatus: String, Codable {
        case pending
        case confirmed
        case processing
        case shipped
        case delivered
        case cancelled
        case refunded
    }
    
    struct Customer: Codable {
        var id: String
        var name: String
        var email: String
        var phone: String?
    }
    
    struct OrderItem: Codable {
        var productID: String
        var name: String
        var quantity: Int
        var unitPrice: Double
        var discount: Double
        
        var totalPrice: Double {
            return Double(quantity) * unitPrice * (1 - discount)
        }
        
        enum CodingKeys: String, CodingKey {
            case productID = "product_id"
            case name, quantity
            case unitPrice = "unit_price"
            case discount
        }
    }
    
    struct Pricing: Codable {
        var subtotal: Double
        var discountAmount: Double
        var shippingCost: Double
        var taxAmount: Double
        var total: Double
        
        enum CodingKeys: String, CodingKey {
            case subtotal
            case discountAmount = "discount_amount"
            case shippingCost = "shipping_cost"
            case taxAmount = "tax_amount"
            case total
        }
    }
    
    struct ShippingInfo: Codable {
        var method: String
        var address: String
        var trackingNumber: String?
        var estimatedDelivery: String?
        
        enum CodingKeys: String, CodingKey {
            case method, address
            case trackingNumber = "tracking_number"
            case estimatedDelivery = "estimated_delivery"
        }
    }
    
    enum CodingKeys: String, CodingKey {
        case orderNumber = "order_number"
        case status, customer, items, pricing, shipping
        case createdAt = "created_at"
        case updatedAt = "updated_at"
    }
}

let orderJSON = """
{
    "order_number": "ORD-2023-001234",
    "status": "shipped",
    "customer": {
        "id": "CUST001",
        "name": "สมชาย ใจดี",
        "email": "somchai@example.com",
        "phone": "081-234-5678"
    },
    "items": [
        {
            "product_id": "PROD001",
            "name": "iPhone 15 Pro",
            "quantity": 1,
            "unit_price": 48900.0,
            "discount": 0.05
        },
        {
            "product_id": "PROD002",
            "name": "AirPods Pro",
            "quantity": 1,
            "unit_price": 9590.0,
            "discount": 0.0
        }
    ],
    "pricing": {
        "subtotal": 56045.0,
        "discount_amount": 2445.0,
        "shipping_cost": 0.0,
        "tax_amount": 3924.0,
        "total": 57524.0
    },
    "shipping": {
        "method": "Express",
        "address": "123 ถนนสุขุมวิท กรุงเทพฯ 10110",
        "tracking_number": "TH1234567890",
        "estimated_delivery": "2023-09-05"
    },
    "created_at": 1693400000,
    "updated_at": 1693450000
}
"""

let decoder = JSONDecoder()
decoder.dateDecodingStrategy = .secondsSince1970

if let data = orderJSON.data(using: .utf8),
   let order = try? decoder.decode(Order.self, from: data) {
    print("คำสั่งซื้อ: \(order.orderNumber)")
    print("สถานะ: \(order.status.rawValue)")
    print("ลูกค้า: \(order.customer.name)")
    print("\nรายการสินค้า:")
    for item in order.items {
        print("  \(item.name) x\(item.quantity) = ฿\(String(format: "%.2f", item.totalPrice))")
    }
    print("\nราคารวม: ฿\(String(format: "%.2f", order.pricing.total))")
    print("Tracking: \(order.shipping.trackingNumber ?? "N/A")")
}
```

---

## Building a JSON API Client

สร้าง API Client ที่ใช้ Codable เพื่อทำงานกับ REST API

```swift
import Foundation

// MARK: - Models

struct GitHubUser: Codable {
    var login: String
    var id: Int
    var name: String?
    var bio: String?
    var publicRepos: Int
    var followers: Int
    var following: Int
    var avatarURL: String
    var htmlURL: String
    var createdAt: Date
    
    enum CodingKeys: String, CodingKey {
        case login, id, name, bio
        case publicRepos = "public_repos"
        case followers, following
        case avatarURL = "avatar_url"
        case htmlURL = "html_url"
        case createdAt = "created_at"
    }
}

struct GitHubRepo: Codable {
    var id: Int
    var name: String
    var fullName: String
    var description: String?
    var stargazersCount: Int
    var forksCount: Int
    var language: String?
    var htmlURL: String
    var isPrivate: Bool
    var updatedAt: Date
    
    enum CodingKeys: String, CodingKey {
        case id, name, language
        case fullName = "full_name"
        case description
        case stargazersCount = "stargazers_count"
        case forksCount = "forks_count"
        case htmlURL = "html_url"
        case isPrivate = "private"
        case updatedAt = "updated_at"
    }
}

// MARK: - API Errors

enum APIError: Error, LocalizedError {
    case invalidURL
    case networkError(Error)
    case invalidResponse(Int)
    case decodingError(Error)
    case noData
    
    var errorDescription: String? {
        switch self {
        case .invalidURL:
            return "URL ไม่ถูกต้อง"
        case .networkError(let error):
            return "Network error: \(error.localizedDescription)"
        case .invalidResponse(let code):
            return "Server ตอบกลับด้วย HTTP \(code)"
        case .decodingError(let error):
            return "ไม่สามารถ decode ข้อมูลได้: \(error.localizedDescription)"
        case .noData:
            return "ไม่มีข้อมูล"
        }
    }
}

// MARK: - API Client

class GitHubAPIClient {
    
    private let session: URLSession
    private let baseURL = "https://api.github.com"
    
    private lazy var decoder: JSONDecoder = {
        let dec = JSONDecoder()
        let formatter = ISO8601DateFormatter()
        formatter.formatOptions = [.withInternetDateTime]
        dec.dateDecodingStrategy = .custom { decoder in
            let container = try decoder.singleValueContainer()
            let string = try container.decode(String.self)
            if let date = formatter.date(from: string) {
                return date
            }
            throw DecodingError.dataCorruptedError(in: container, debugDescription: "Invalid date: \(string)")
        }
        return dec
    }()
    
    init(session: URLSession = .shared) {
        self.session = session
    }
    
    // MARK: - Generic request method
    
    func request<T: Decodable>(
        endpoint: String,
        completion: @escaping (Result<T, APIError>) -> Void
    ) {
        guard let url = URL(string: baseURL + endpoint) else {
            completion(.failure(.invalidURL))
            return
        }
        
        var request = URLRequest(url: url)
        request.setValue("application/vnd.github.v3+json", forHTTPHeaderField: "Accept")
        
        session.dataTask(with: request) { [weak self] data, response, error in
            guard let self = self else { return }
            
            if let error = error {
                completion(.failure(.networkError(error)))
                return
            }
            
            guard let httpResponse = response as? HTTPURLResponse else {
                completion(.failure(.noData))
                return
            }
            
            guard (200...299).contains(httpResponse.statusCode) else {
                completion(.failure(.invalidResponse(httpResponse.statusCode)))
                return
            }
            
            guard let data = data else {
                completion(.failure(.noData))
                return
            }
            
            do {
                let decoded = try self.decoder.decode(T.self, from: data)
                completion(.success(decoded))
            } catch {
                completion(.failure(.decodingError(error)))
            }
        }.resume()
    }
    
    // MARK: - Specific API Methods
    
    func getUser(username: String, completion: @escaping (Result<GitHubUser, APIError>) -> Void) {
        request(endpoint: "/users/\(username)", completion: completion)
    }
    
    func getUserRepos(username: String, completion: @escaping (Result<[GitHubRepo], APIError>) -> Void) {
        request(endpoint: "/users/\(username)/repos?sort=stars&per_page=10", completion: completion)
    }
    
    // MARK: - Async/Await version (iOS 15+)
    
    @available(iOS 15.0, *)
    func getUser(username: String) async throws -> GitHubUser {
        guard let url = URL(string: "\(baseURL)/users/\(username)") else {
            throw APIError.invalidURL
        }
        
        var request = URLRequest(url: url)
        request.setValue("application/vnd.github.v3+json", forHTTPHeaderField: "Accept")
        
        let (data, response) = try await session.data(for: request)
        
        guard let httpResponse = response as? HTTPURLResponse,
              (200...299).contains(httpResponse.statusCode) else {
            let statusCode = (response as? HTTPURLResponse)?.statusCode ?? 0
            throw APIError.invalidResponse(statusCode)
        }
        
        do {
            return try decoder.decode(GitHubUser.self, from: data)
        } catch {
            throw APIError.decodingError(error)
        }
    }
}

// MARK: - Usage Example

// Callback style
let client = GitHubAPIClient()
/*
client.getUser(username: "apple") { result in
    switch result {
    case .success(let user):
        print("User: \(user.login)")
        print("Repos: \(user.publicRepos)")
        print("Followers: \(user.followers)")
    case .failure(let error):
        print("Error: \(error.localizedDescription)")
    }
}
*/

// Async/Await style
/*
Task {
    do {
        let user = try await client.getUser(username: "apple")
        print("User: \(user.login)")
        let repos = try await client.getUserRepos(username: "apple")  // ต้อง implement
        for repo in repos.prefix(5) {
            print("  \(repo.name): ⭐ \(repo.stargazersCount)")
        }
    } catch {
        print("Error: \(error.localizedDescription)")
    }
}
*/

print("GitHubAPIClient ready to use!")
print("Call client.getUser(username:) to fetch user data")
```

### Caching Layer สำหรับ API Client

```swift
import Foundation

// Simple in-memory cache
class JSONCache {
    private var cache: [String: (data: Data, expiry: Date)] = [:]
    private let ttl: TimeInterval
    
    init(ttl: TimeInterval = 300) {  // 5 minutes default
        self.ttl = ttl
    }
    
    func get(key: String) -> Data? {
        guard let entry = cache[key],
              entry.expiry > Date() else {
            cache.removeValue(forKey: key)
            return nil
        }
        return entry.data
    }
    
    func set(key: String, data: Data) {
        cache[key] = (data, Date().addingTimeInterval(ttl))
    }
    
    func invalidate(key: String) {
        cache.removeValue(forKey: key)
    }
    
    func clear() {
        cache.removeAll()
    }
}

// ใช้งานกับ API Client
class CachedAPIClient {
    private let cache = JSONCache(ttl: 600)
    private let decoder = JSONDecoder()
    
    func fetch<T: Decodable>(
        url: URL,
        type: T.Type,
        forceRefresh: Bool = false
    ) async throws -> T {
        let cacheKey = url.absoluteString
        
        // ตรวจสอบ cache ก่อน
        if !forceRefresh, let cachedData = cache.get(key: cacheKey) {
            return try decoder.decode(T.self, from: cachedData)
        }
        
        // ดึงข้อมูลใหม่
        let (data, _) = try await URLSession.shared.data(from: url)
        
        // บันทึก cache
        cache.set(key: cacheKey, data: data)
        
        return try decoder.decode(T.self, from: data)
    }
}

print("CachedAPIClient ready!")
```

---

## Summary

### สรุปหัวข้อที่เรียนรู้

1. **JSON พื้นฐาน**
   - JSON เป็น format แลกเปลี่ยนข้อมูลที่เป็นมาตรฐาน
   - รองรับ 6 ประเภทข้อมูล: String, Number, Boolean, Null, Array, Object

2. **Codable Protocol**
   - `Codable` = `Encodable` + `Decodable`
   - Swift synthesize implementation อัตโนมัติสำหรับ simple types
   - รองรับ struct, class, enum

3. **JSONEncoder/JSONDecoder**
   - `JSONEncoder` แปลง Swift object เป็น JSON Data
   - `JSONDecoder` แปลง JSON Data เป็น Swift object
   - มี options หลายอย่างสำหรับ customize behavior

4. **CodingKeys**
   - ใช้ map ระหว่าง JSON key และ Swift property name
   - รองรับการ exclude properties จาก encoding/decoding

5. **Custom Encoding/Decoding**
   - Override `encode(to:)` และ `init(from:)` สำหรับ custom behavior
   - ใช้ container methods: `container(keyedBy:)`, `nestedContainer(keyedBy:forKey:)`, `singleValueContainer()`

6. **Strategies**
   - Date strategies: deferredToDate, secondsSince1970, iso8601, formatted
   - Key strategies: convertFromSnakeCase, convertToSnakeCase, custom
   - Data strategies: base64, custom

7. **Advanced Patterns**
   - Polymorphic decoding ด้วย type discriminator
   - Generic types กับ Codable
   - Inheritance กับ Codable

8. **Error Handling**
   - DecodingError types: typeMismatch, valueNotFound, keyNotFound, dataCorrupted
   - Custom validation ใน init(from:)

### Best Practices

```swift
// ✅ ดี - ใช้ CodingKeys เมื่อจำเป็น
struct User: Codable {
    var userID: Int
    enum CodingKeys: String, CodingKey {
        case userID = "user_id"
    }
}

// ✅ ดี - ใช้ keyDecodingStrategy แทน CodingKeys เมื่อทำได้
decoder.keyDecodingStrategy = .convertFromSnakeCase

// ✅ ดี - Handle errors อย่างชัดเจน
do {
    let result = try decoder.decode(MyType.self, from: data)
} catch DecodingError.keyNotFound(let key, let context) {
    // จัดการ key not found
} catch {
    // จัดการ error อื่นๆ
}

// ✅ ดี - ใช้ Optional สำหรับ fields ที่อาจไม่มี
struct FlexibleModel: Codable {
    var requiredField: String
    var optionalField: String?  // ไม่บังคับ
}

// ✅ ดี - ใช้ Generic wrapper สำหรับ API responses
struct APIResponse<T: Codable>: Codable {
    var data: T
    var success: Bool
    var message: String
}
```

### Quick Reference

```swift
// Encode
let encoder = JSONEncoder()
encoder.outputFormatting = .prettyPrinted
encoder.dateEncodingStrategy = .iso8601
encoder.keyEncodingStrategy = .convertToSnakeCase
let data = try encoder.encode(myObject)

// Decode
let decoder = JSONDecoder()
decoder.dateDecodingStrategy = .iso8601
decoder.keyDecodingStrategy = .convertFromSnakeCase
let object = try decoder.decode(MyType.self, from: data)

// Decode array
let objects = try decoder.decode([MyType].self, from: data)

// Convert to/from String
let jsonString = String(data: data, encoding: .utf8)
let jsonData = jsonString.data(using: .utf8)
```

ในบทถัดไปเราจะเรียนรู้เกี่ยวกับ **Core Data** ซึ่งเป็น framework สำหรับ persist ข้อมูลใน local database บน iOS และ macOS

---

*จบ Part 31: JSON และ Codable ใน Swift*
