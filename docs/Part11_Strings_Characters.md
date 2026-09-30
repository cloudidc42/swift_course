# Part 11: Strings และ Characters ใน Swift

## บทนำ

String (สตริง) เป็นหนึ่งในประเภทข้อมูลพื้นฐานที่สำคัญที่สุดในการเขียนโปรแกรม Swift ใช้ String เพื่อเก็บและจัดการข้อความ ตั้งแต่ชื่อผู้ใช้ง่ายๆ ไปจนถึงเนื้อหาบทความหรือข้อมูล JSON ที่ซับซ้อน

Swift มี String type ที่ทรงพลังและ Unicode-compliant อย่างสมบูรณ์ ทำให้สามารถจัดการกับข้อความในทุกภาษาทั่วโลกได้อย่างถูกต้อง

---

## 1. String Fundamentals (พื้นฐาน String)

### 1.1 การประกาศ String

```swift
// การประกาศ String แบบพื้นฐาน
var greeting: String = "สวัสดีชาวโลก"
var name = "Swift Programming"  // Type inference

// String ว่าง
var emptyString = ""
var anotherEmpty = String()

// ตรวจสอบว่า String ว่างหรือไม่
if emptyString.isEmpty {
    print("String นี้ว่างเปล่า")
}

// String ที่ไม่เปลี่ยนค่า (constant)
let fixedString = "ค่าคงที่"
// fixedString = "เปลี่ยนไม่ได้"  // Error: ไม่สามารถเปลี่ยนค่า let ได้

// String ที่เปลี่ยนค่าได้ (variable)
var mutableString = "เปลี่ยนได้"
mutableString = "เปลี่ยนค่าแล้ว"
print(mutableString)  // "เปลี่ยนค่าแล้ว"
```

### 1.2 String เป็น Value Type

Swift String เป็น value type ซึ่งหมายความว่าเมื่อคุณส่ง String ให้ฟังก์ชันหรือกำหนดให้ตัวแปรใหม่ จะมีการสร้าง copy ใหม่

```swift
var original = "Hello"
var copy = original    // สร้าง copy ของ original

copy += " World"       // แก้ไข copy

print(original)        // "Hello" - ไม่เปลี่ยนแปลง
print(copy)            // "Hello World"

// เปรียบเทียบกับ Reference Type
// ถ้า String เป็น reference type copy จะชี้ไปที่ object เดียวกัน
// แต่เนื่องจากเป็น value type จึงเป็นอิสระจากกัน
```

---

## 2. String Literals (ค่าคงที่สตริง)

### 2.1 Single-line String Literals

```swift
// Single-line string literal ใช้เครื่องหมาย double quote
let singleLine = "นี่คือ single-line string"

// การใช้ escape characters
let withNewline = "บรรทัดแรก\nบรรทัดสอง"
let withTab = "คอลัมน์1\tคอลัมน์2"
let withQuote = "เขาพูดว่า \"สวัสดี\""
let withBackslash = "C:\\Users\\Documents"
let withNull = "ก่อน\0หลัง"  // null character

// Unicode escape
let unicodeChar = "\u{0041}"     // "A"
let thaiChar = "\u{0E2A}"        // "ส"
let emoji = "\u{1F600}"          // 😀

print(withNewline)
print(withTab)
print(withQuote)
```

### 2.2 Multi-line String Literals

Multi-line string literals ใช้เครื่องหมาย triple double quotes `"""`

```swift
// Multi-line string literal
let poem = """
    กลางดึกเดือนดับ
    ดาวแจ่มจรัส
    ใจหวั่นไหว
    """

print(poem)
// กลางดึกเดือนดับ
// ดาวแจ่มจรัส
// ใจหวั่นไหว

// การเยื้อง (indentation) ใน multi-line string
// Swift จะลบ whitespace ที่อยู่หน้า closing """
let indented = """
    บรรทัดแรก
        บรรทัดที่มี indent เพิ่ม
    บรรทัดสุดท้าย
    """
print(indented)
// บรรทัดแรก
//     บรรทัดที่มี indent เพิ่ม
// บรรทัดสุดท้าย

// Multi-line ที่ขึ้นบรรทัดใหม่ต่อกัน
let joined = """
    บรรทัดแรก \
    ต่อในบรรทัดเดียวกัน
    """
print(joined)  // "บรรทัดแรก ต่อในบรรทัดเดียวกัน"

// Multi-line กับ special characters
let htmlTemplate = """
    <html>
        <head>
            <title>หน้าเว็บของฉัน</title>
        </head>
        <body>
            <p>สวัสดีชาวโลก</p>
        </body>
    </html>
    """
print(htmlTemplate)
```

### 2.3 Extended String Delimiters

Extended string delimiters (`#"..."#`) ช่วยให้ใช้ backslash และ special characters ได้โดยไม่ต้อง escape

```swift
// Extended delimiter
let rawString = #"นี่คือ backslash: \ และ quote: ""#
print(rawString)  // "นี่คือ backslash: \ และ quote: ""

// ถ้าต้องการ interpolation ใช้ \#(...)
let value = 42
let extended = #"ค่าคือ \#(value)"#
print(extended)  // "ค่าคือ 42"

// Multi-line extended
let multiExtended = #"""
    ตัวอย่าง \n ไม่ถูก escape
    ต้องใช้ \#n เพื่อขึ้นบรรทัดใหม่
    """#
print(multiExtended)

// Double extended
let doubleExtended = ##"ต้องการ \# ใน string นี้"##
print(doubleExtended)
```

---

## 3. String Interpolation (การแทรกค่าในสตริง)

String interpolation เป็นวิธีสร้าง String ใหม่โดยแทรกค่าจากตัวแปรหรือ expression

```swift
// พื้นฐาน String interpolation
let firstName = "สมชาย"
let lastName = "ใจดี"
let age = 25

let intro = "ชื่อของฉันคือ \(firstName) \(lastName) อายุ \(age) ปี"
print(intro)

// Interpolation กับ expressions
let price = 150.0
let quantity = 3
let total = "ราคารวม: \(price * Double(quantity)) บาท"
print(total)  // "ราคารวม: 450.0 บาท"

// Interpolation กับ function calls
func greet(_ name: String) -> String {
    return "สวัสดี \(name)!"
}

let message = "ข้อความ: \(greet("คุณสมหมาย"))"
print(message)

// Interpolation กับ ternary operator
let score = 85
let grade = "เกรด: \(score >= 80 ? "A" : score >= 70 ? "B" : "C")"
print(grade)  // "เกรด: A"

// Interpolation กับ optional
let optionalName: String? = "มานี"
let optionalMessage = "ชื่อ: \(optionalName ?? "ไม่ระบุ")"
print(optionalMessage)

// Format numbers ด้วย String interpolation
let pi = 3.14159265
let formattedPi = String(format: "π ≈ %.2f", pi)
print(formattedPi)  // "π ≈ 3.14"

// Custom interpolation (Advanced)
extension String.StringInterpolation {
    mutating func appendInterpolation(_ value: Double, decimals: Int) {
        let formatted = String(format: "%.\(decimals)f", value)
        appendLiteral(formatted)
    }
}

let customPi = "π = \(pi, decimals: 4)"
print(customPi)  // "π = 3.1416"
```

---

## 4. String Concatenation (การต่อสตริง)

```swift
// การต่อ String ด้วย + operator
let hello = "สวัสดี"
let world = " ชาวโลก"
let helloWorld = hello + world
print(helloWorld)  // "สวัสดี ชาวโลก"

// การใช้ += operator
var greeting2 = "กิน"
greeting2 += "ข้าว"
greeting2 += "แล้วหรือยัง"
print(greeting2)  // "กินข้าวแล้วหรือยัง"

// การต่อ String กับ Character
var phrase = "Hello"
let exclamation: Character = "!"
phrase.append(exclamation)
print(phrase)  // "Hello!"

// การต่อหลาย String ด้วย joined
let words = ["Swift", "is", "awesome"]
let sentence = words.joined(separator: " ")
print(sentence)  // "Swift is awesome"

// joined กับ Array ของ String
let items = ["แอปเปิ้ล", "กล้วย", "ส้ม"]
let list = items.joined(separator: ", ")
print(list)  // "แอปเปิ้ล, กล้วย, ส้ม"

// Performance: ใช้ Array.joined แทน loop + concatenation
var result = ""
let numbers = (1...1000).map { "\($0)" }

// ไม่ดี (O(n²)):
// for num in numbers { result += num }

// ดีกว่า (O(n)):
result = numbers.joined(separator: ", ")
```

---

## 5. String Characters (ตัวอักษรในสตริง)

### 5.1 การเข้าถึง Characters

```swift
// ทุก String เป็น Collection ของ Character
let word = "Swift"

// วนลูปผ่าน characters
for char in word {
    print(char)
}
// S
// w
// i
// f
// t

// นับจำนวน characters
print(word.count)  // 5

// สร้าง String จาก Array ของ Character
let chars: [Character] = ["S", "w", "i", "f", "t"]
let fromChars = String(chars)
print(fromChars)  // "Swift"

// Character literals
let singleChar: Character = "A"
let thaiChar2: Character = "ก"
let emojiChar: Character = "😀"

// ตรวจสอบ Character
let testChar: Character = "5"
print(testChar.isLetter)     // false
print(testChar.isNumber)     // true
print(testChar.isWhitespace) // false
print(testChar.isUppercase)  // false

let letterChar: Character = "A"
print(letterChar.isLetter)   // true
print(letterChar.isUppercase) // true
print(letterChar.isASCII)    // true

// Unicode properties
let thaiLetter: Character = "ก"
print(thaiLetter.unicodeScalars.first!.value)  // 3585 (0x0E01)
```

### 5.2 Extended Grapheme Clusters

ใน Swift ทุก Character เป็น extended grapheme cluster ซึ่งอาจประกอบด้วย Unicode scalar หลายตัว

```swift
// ตัวอย่าง extended grapheme cluster
let café1 = "café"   // e + combining acute accent
let café2 = "café"   // precomposed character é

// ทั้งสองมีความหมายเหมือนกัน
print(café1 == café2)  // true

// จำนวน characters อาจต่างจากจำนวน Unicode scalars
let eAcute: Character = "\u{E9}"         // é precomposed
let eAcute2: Character = "\u{65}\u{301}" // e + combining accent

print(eAcute == eAcute2)  // true

// Emoji ที่ซับซ้อน
let familyEmoji = "👨‍👩‍👧‍👦"  // ครอบครัว
print(familyEmoji.count)  // 1 (เป็น 1 Character)
print(familyEmoji.unicodeScalars.count)  // หลาย scalars

// Flag emoji
let flag = "🇹🇭"  // ธงไทย
print(flag.count)  // 1
print(flag.unicodeScalars.count)  // 2 (regional indicators)
```

---

## 6. String Indices และ Subscripting

### 6.1 String.Index

เนื่องจาก Swift String รองรับ Unicode อย่างสมบูรณ์ จึงไม่สามารถ access ด้วย integer index ได้โดยตรง ต้องใช้ `String.Index`

```swift
let str = "Hello, World!"

// startIndex และ endIndex
let start = str.startIndex           // ชี้ไปที่ 'H'
let end = str.endIndex               // ชี้ไปหลังตัวสุดท้าย

// การ access ด้วย index
print(str[start])  // "H"
// print(str[end])  // Error! endIndex ชี้ไปหลังตัวสุดท้าย

// การเลื่อน index
let secondIndex = str.index(after: start)    // ชี้ไปที่ 'e'
let thirdIndex = str.index(start, offsetBy: 2)  // ชี้ไปที่ 'l'
let lastIndex = str.index(before: end)           // ชี้ไปที่ '!'

print(str[secondIndex])  // "e"
print(str[thirdIndex])   // "l"
print(str[lastIndex])    // "!"

// การใช้ offsetBy กับ limitedBy
if let safeIndex = str.index(start, offsetBy: 100, limitedBy: end) {
    print(str[safeIndex])
} else {
    print("Index เกิน range")  // จะแสดงข้อความนี้
}

// วนลูปด้วย index
var currentIndex = str.startIndex
while currentIndex < str.endIndex {
    print(str[currentIndex], terminator: " ")
    currentIndex = str.index(after: currentIndex)
}
print()
```

### 6.2 Substrings ด้วย Range

```swift
let message = "Hello, Swift World!"

// การสร้าง range
let startIdx = message.startIndex
let commaIdx = message.firstIndex(of: ",")!

// Substring ด้วย range
let hello = message[startIdx..<commaIdx]
print(hello)  // "Hello"

// ด้วย partial ranges
let fromStart = message[...commaIdx]  // "Hello,"
let toEnd = message[commaIdx...]      // ", Swift World!"
print(fromStart)
print(toEnd)

// หา index ของ substring
if let spaceRange = message.range(of: "Swift") {
    let swiftWord = message[spaceRange]
    print(swiftWord)  // "Swift"
}

// ตัวอย่างกับภาษาไทย
let thaiStr = "สวัสดีชาวโลก"
let thaiStart = thaiStr.startIndex
let thaiThirdIndex = thaiStr.index(thaiStart, offsetBy: 3)
let thaiSubstring = thaiStr[thaiStart..<thaiThirdIndex]
print(thaiSubstring)  // "สวั"
```

---

## 7. Unicode Support (การรองรับ Unicode)

### 7.1 Unicode Scalar Values

```swift
// การเข้าถึง Unicode scalars
let dog = "🐕"
for scalar in dog.unicodeScalars {
    print("U+\(String(scalar.value, radix: 16).uppercased())")
}
// U+1F415

// ตัวอักษรไทย
let thai = "กขค"
for scalar in thai.unicodeScalars {
    print("U+\(String(scalar.value, radix: 16).uppercased()): \(scalar)")
}
// U+E01: ก
// U+E02: ข
// U+E03: ค

// สร้าง String จาก Unicode scalar
let heart = "\u{2764}"          // ❤
let sparkles = "\u{2728}"       // ✨
let combined = heart + sparkles
print(combined)  // ❤✨
```

### 7.2 UTF-8 และ UTF-16 Representations

```swift
let greeting3 = "สวัสดี"

// UTF-8 representation
print("UTF-8:")
for byte in greeting3.utf8 {
    print(String(byte, radix: 16), terminator: " ")
}
print()

// UTF-16 representation
print("UTF-16:")
for unit in greeting3.utf16 {
    print(String(unit, radix: 16), terminator: " ")
}
print()

// Unicode Scalar representation
print("Unicode Scalars:")
for scalar in greeting3.unicodeScalars {
    print("\(scalar.value)", terminator: " ")
}
print()

// ตัวอย่าง emoji ที่มี UTF-8 หลาย bytes
let emoji2 = "😊"
print("Emoji UTF-8 bytes: \(emoji2.utf8.count)")  // 4 bytes
print("Emoji characters: \(emoji2.count)")         // 1 character
```

---

## 8. String Mutability (การเปลี่ยนแปลงสตริง)

```swift
// var string สามารถแก้ไขได้
var mutableStr = "Hello"

// เพิ่มตัวอักษร
mutableStr.append("!")
print(mutableStr)  // "Hello!"

// เพิ่ม String
mutableStr.append(" How are you?")
print(mutableStr)  // "Hello! How are you?"

// แทรก character ที่ตำแหน่งที่กำหนด
let insertIdx = mutableStr.index(mutableStr.startIndex, offsetBy: 5)
mutableStr.insert(",", at: insertIdx)
print(mutableStr)  // "Hello,! How are you?"

// แทรก string ที่ตำแหน่งที่กำหนด
mutableStr.insert(contentsOf: " World", at: insertIdx)
print(mutableStr)  // "Hello World,! How are you?"

// ลบ character
let removeIdx = mutableStr.index(mutableStr.startIndex, offsetBy: 5)
mutableStr.remove(at: removeIdx)
print(mutableStr)  // "HelloWorld,! How are you?"

// ลบ range ของ characters
let rangeStart = mutableStr.index(mutableStr.startIndex, offsetBy: 5)
let rangeEnd = mutableStr.index(rangeStart, offsetBy: 5)
mutableStr.removeSubrange(rangeStart..<rangeEnd)
print(mutableStr)  // "Hello,! How are you?"

// แทนที่ด้วย replaceSubrange
var replaceStr = "Hello World"
let replaceStart = replaceStr.index(replaceStr.startIndex, offsetBy: 6)
let replaceEnd = replaceStr.endIndex
replaceStr.replaceSubrange(replaceStart..<replaceEnd, with: "Swift")
print(replaceStr)  // "Hello Swift"
```

---

## 9. String Comparison (การเปรียบเทียบสตริง)

```swift
// การเปรียบเทียบด้วย == และ !=
let str1 = "Hello"
let str2 = "Hello"
let str3 = "World"

print(str1 == str2)  // true
print(str1 == str3)  // false
print(str1 != str3)  // true

// Unicode-aware comparison
let eComposed = "é"      // U+00E9
let eDecomposed = "e\u{301}"  // e + combining acute

print(eComposed == eDecomposed)  // true (canonical equivalence)

// Lexicographic comparison (<, >, <=, >=)
print("apple" < "banana")  // true
print("zebra" > "apple")   // true
print("abc" <= "abd")      // true

// ภาษาไทย
print("กข" < "ขค")  // true (based on Unicode code points)

// Case-sensitive vs case-insensitive
let lower = "hello"
let upper = "HELLO"

print(lower == upper)  // false (case-sensitive)
print(lower.lowercased() == upper.lowercased())  // true

// Locale-aware comparison
import Foundation

let localCompare = "café".compare("CAFÉ", options: .caseInsensitive)
print(localCompare == .orderedSame)  // true

// เปรียบเทียบแบบ natural order (สำหรับตัวเลขใน string)
let files = ["file10", "file2", "file1", "file20"]
let sorted = files.sorted { $0.localizedStandardCompare($1) == .orderedAscending }
print(sorted)  // ["file1", "file2", "file10", "file20"]
```

---

## 10. String Searching (การค้นหาในสตริง)

```swift
import Foundation

let text = "Swift programming is fun and Swift is powerful"

// contains - ตรวจสอบว่ามี substring อยู่หรือไม่
print(text.contains("Swift"))       // true
print(text.contains("Python"))      // false
print(text.contains("swift"))       // false (case-sensitive)

// hasPrefix - ตรวจสอบที่ต้นสตริง
print(text.hasPrefix("Swift"))      // true
print(text.hasPrefix("Programming"))// false

// hasSuffix - ตรวจสอบที่ปลายสตริง
print(text.hasSuffix("powerful"))   // true
print(text.hasSuffix("fun"))        // false

// range(of:) - หาตำแหน่งของ substring
if let range = text.range(of: "fun") {
    print("พบ 'fun' ที่ตำแหน่ง: \(text.distance(from: text.startIndex, to: range.lowerBound))")
    print("ข้อความที่พบ: \(text[range])")
}

// range(of:options:) - ค้นหาแบบ case-insensitive
if let range = text.range(of: "SWIFT", options: .caseInsensitive) {
    print("พบ 'SWIFT' (case-insensitive): \(text[range])")
}

// ค้นหาทั้งหมด
var searchRange = text.startIndex..<text.endIndex
var occurrences: [Range<String.Index>] = []

while let found = text.range(of: "Swift", range: searchRange) {
    occurrences.append(found)
    searchRange = found.upperBound..<text.endIndex
}

print("พบ 'Swift' \(occurrences.count) ครั้ง")

// firstIndex และ lastIndex
if let firstE = text.firstIndex(of: "i") {
    print("ตำแหน่งแรกของ 'i': \(text.distance(from: text.startIndex, to: firstE))")
}

if let lastE = text.lastIndex(of: "i") {
    print("ตำแหน่งสุดท้ายของ 'i': \(text.distance(from: text.startIndex, to: lastE))")
}
```

---

## 11. String Manipulation (การจัดการสตริง)

### 11.1 Case Conversion

```swift
let mixed = "Hello World Swift"

// uppercased() - แปลงเป็นตัวพิมพ์ใหญ่ทั้งหมด
print(mixed.uppercased())  // "HELLO WORLD SWIFT"

// lowercased() - แปลงเป็นตัวพิมพ์เล็กทั้งหมด
print(mixed.lowercased())  // "hello world swift"

// capitalized - ทำให้ตัวแรกของทุกคำเป็นตัวพิมพ์ใหญ่
print(mixed.capitalized)   // "Hello World Swift"

// ตัวอักษรแรก uppercase
let sentence = "hello world"
let firstUppercased = sentence.prefix(1).uppercased() + sentence.dropFirst()
print(firstUppercased)  // "Hello world"

// ภาษาไทย - uppercased/lowercased ไม่มีผลกับภาษาไทย
let thai = "สวัสดีชาวโลก"
print(thai.uppercased())  // "สวัสดีชาวโลก" (เหมือนเดิม)
```

### 11.2 String Splitting

```swift
import Foundation

let csv = "แอปเปิ้ล,กล้วย,ส้ม,มะม่วง"

// components(separatedBy:) - แบ่งด้วย string separator
let fruits = csv.components(separatedBy: ",")
print(fruits)  // ["แอปเปิ้ล", "กล้วย", "ส้ม", "มะม่วง"]

// split(separator:) - แบ่งด้วย character
let data = "1 2 3 4 5"
let numbers = data.split(separator: " ")
print(numbers)  // ["1", "2", "3", "4", "5"]

// split ที่มี maxSplits
let limited = data.split(separator: " ", maxSplits: 2)
print(limited)  // ["1", "2", "3 4 5"]

// split ที่ omitEmptySubsequences
let withSpaces = "a  b   c"
let noEmpty = withSpaces.split(separator: " ", omitEmptySubsequences: true)
let withEmpty = withSpaces.split(separator: " ", omitEmptySubsequences: false)
print(noEmpty)   // ["a", "b", "c"]
print(withEmpty) // ["a", "", "b", "", "", "c"]

// components กับ CharacterSet
let text2 = "Hello,World;Swift:Language"
let separators = CharacterSet(charactersIn: ",;:")
let parts = text2.components(separatedBy: separators)
print(parts)  // ["Hello", "World", "Swift", "Language"]

// แบ่งตาม newline
let multiline = """
    บรรทัดที่ 1
    บรรทัดที่ 2
    บรรทัดที่ 3
    """
let lines = multiline.components(separatedBy: "\n")
print(lines.count)  // 3
```

### 11.3 String Trimming

```swift
import Foundation

let padded = "   Hello, World!   "

// trimmingCharacters(in:) - ตัด whitespace
let trimmed = padded.trimmingCharacters(in: .whitespaces)
print(trimmed)  // "Hello, World!"

// ตัด newlines ด้วย
let withNewlines = "\n\tHello\n\t"
let trimmedNewlines = withNewlines.trimmingCharacters(in: .whitespacesAndNewlines)
print(trimmedNewlines)  // "Hello"

// ตัดตัวอักษรที่กำหนดเอง
let customTrim = "***Hello***"
let customTrimmed = customTrim.trimmingCharacters(in: CharacterSet(charactersIn: "*"))
print(customTrimmed)  // "Hello"

// ตัดเฉพาะข้างหน้า
extension String {
    func trimPrefix(_ characters: CharacterSet) -> String {
        var result = self
        while let first = result.unicodeScalars.first,
              characters.contains(first) {
            result = String(result.dropFirst())
        }
        return result
    }
    
    func trimSuffix(_ characters: CharacterSet) -> String {
        var result = self
        while let last = result.unicodeScalars.last,
              characters.contains(last) {
            result = String(result.dropLast())
        }
        return result
    }
}

let testStr = "   hello   "
print(testStr.trimPrefix(.whitespaces))  // "hello   "
print(testStr.trimSuffix(.whitespaces))  // "   hello"
```

### 11.4 String Replacing

```swift
import Foundation

let original = "The quick brown fox jumps over the lazy dog"

// replacingOccurrences(of:with:)
let replaced = original.replacingOccurrences(of: "fox", with: "cat")
print(replaced)  // "The quick brown cat jumps over the lazy dog"

// replace case-insensitive
let caseReplaced = original.replacingOccurrences(
    of: "THE",
    with: "a",
    options: .caseInsensitive
)
print(caseReplaced)  // "a quick brown fox jumps over a lazy dog"

// replace ด้วย regex
let numbers2 = "abc123def456ghi"
let noNumbers = numbers2.replacingOccurrences(
    of: "[0-9]+",
    with: "#",
    options: .regularExpression
)
print(noNumbers)  // "abc#def#ghi"

// ใช้ range-based replacement
var mutable = "Hello World"
if let range = mutable.range(of: "World") {
    mutable.replaceSubrange(range, with: "Swift")
}
print(mutable)  // "Hello Swift"
```

---

## 12. String Trimming และ Padding

```swift
import Foundation

// Padding ด้วย Swift
extension String {
    func padLeft(toLength length: Int, withPad pad: Character = " ") -> String {
        if self.count >= length { return self }
        return String(repeating: pad, count: length - self.count) + self
    }
    
    func padRight(toLength length: Int, withPad pad: Character = " ") -> String {
        if self.count >= length { return self }
        return self + String(repeating: pad, count: length - self.count)
    }
    
    func padCenter(toLength length: Int, withPad pad: Character = " ") -> String {
        if self.count >= length { return self }
        let totalPad = length - self.count
        let leftPad = totalPad / 2
        let rightPad = totalPad - leftPad
        return String(repeating: pad, count: leftPad) + self + String(repeating: pad, count: rightPad)
    }
}

let name = "Swift"
print(name.padLeft(toLength: 10))         // "     Swift"
print(name.padRight(toLength: 10))        // "Swift     "
print(name.padCenter(toLength: 11))       // "   Swift   "
print(name.padLeft(toLength: 10, withPad: "0"))  // "00000Swift"

// String repeating
let dashes = String(repeating: "-", count: 20)
print(dashes)  // "--------------------"
```

---

## 13. Regular Expressions กับ NSRegularExpression

```swift
import Foundation

// สร้าง NSRegularExpression
func findMatches(pattern: String, in text: String) -> [String] {
    guard let regex = try? NSRegularExpression(pattern: pattern, options: []) else {
        return []
    }
    
    let range = NSRange(text.startIndex..., in: text)
    let matches = regex.matches(in: text, range: range)
    
    return matches.compactMap { match -> String? in
        guard let range = Range(match.range, in: text) else { return nil }
        return String(text[range])
    }
}

// ตัวอย่างการใช้งาน
let email = "contact@example.com is the email, also try info@test.org"
let emailPattern = "[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}"
let emails = findMatches(pattern: emailPattern, in: email)
print(emails)  // ["contact@example.com", "info@test.org"]

// ตรวจสอบว่า string ตรงกับ pattern
func matches(pattern: String, in text: String) -> Bool {
    guard let regex = try? NSRegularExpression(pattern: pattern) else { return false }
    let range = NSRange(text.startIndex..., in: text)
    return regex.firstMatch(in: text, range: range) != nil
}

let phoneNumber = "081-234-5678"
let phonePattern = "^[0-9]{3}-[0-9]{3}-[0-9]{4}$"
print(matches(pattern: phonePattern, in: phoneNumber))  // true

// Capture groups
func extractGroups(pattern: String, from text: String) -> [[String]] {
    guard let regex = try? NSRegularExpression(pattern: pattern) else { return [] }
    let range = NSRange(text.startIndex..., in: text)
    let matches = regex.matches(in: text, range: range)
    
    return matches.map { match -> [String] in
        return (0..<match.numberOfRanges).compactMap { index -> String? in
            guard let range = Range(match.range(at: index), in: text) else { return nil }
            return String(text[range])
        }
    }
}

let dateText = "วันที่ 2024-01-15 และ 2024-02-20"
let datePattern = "(\\d{4})-(\\d{2})-(\\d{2})"
let dateParts = extractGroups(pattern: datePattern, from: dateText)
for parts in dateParts {
    print("Full: \(parts[0]), Year: \(parts[1]), Month: \(parts[2]), Day: \(parts[3])")
}

// Replace ด้วย regex
func replaceAll(pattern: String, replacement: String, in text: String) -> String {
    guard let regex = try? NSRegularExpression(pattern: pattern) else { return text }
    let range = NSRange(text.startIndex..., in: text)
    return regex.stringByReplacingMatches(in: text, range: range, withTemplate: replacement)
}

let htmlText = "<p>Hello <b>World</b></p>"
let strippedHTML = replaceAll(pattern: "<[^>]+>", replacement: "", in: htmlText)
print(strippedHTML)  // "Hello World"
```

---

## 14. String Formatting (การจัดรูปแบบสตริง)

```swift
import Foundation

// String(format:) - C-style formatting
let name2 = "สมชาย"
let score2 = 95.5
let formatted = String(format: "ชื่อ: %@, คะแนน: %.1f%%", name2, score2)
print(formatted)  // "ชื่อ: สมชาย, คะแนน: 95.5%"

// Format specifiers ที่ใช้บ่อย
let integer = 42
let double = 3.14159
let str = "Hello"

print(String(format: "%d", integer))      // "42" (integer)
print(String(format: "%05d", integer))    // "00042" (padded)
print(String(format: "%f", double))       // "3.141590"
print(String(format: "%.2f", double))     // "3.14"
print(String(format: "%e", double))       // "3.141590e+00"
print(String(format: "%s", str))          // "Hello"
print(String(format: "%@", str as AnyObject)) // "Hello"
print(String(format: "%x", integer))      // "2a" (hex)
print(String(format: "%X", integer))      // "2A" (hex uppercase)
print(String(format: "%o", integer))      // "52" (octal)
print(String(format: "%b", integer))      // ไม่รองรับโดยตรง

// NumberFormatter
let numberFormatter = NumberFormatter()
numberFormatter.numberStyle = .decimal
numberFormatter.groupingSeparator = ","
numberFormatter.minimumFractionDigits = 2

let bigNumber = 1234567.89
if let formatted2 = numberFormatter.string(from: NSNumber(value: bigNumber)) {
    print(formatted2)  // "1,234,567.89"
}

// Currency formatting
numberFormatter.numberStyle = .currency
numberFormatter.currencyCode = "THB"
if let currency = numberFormatter.string(from: NSNumber(value: 1500.50)) {
    print(currency)  // "฿1,500.50"
}

// DateFormatter
let dateFormatter = DateFormatter()
dateFormatter.dateFormat = "dd/MM/yyyy HH:mm:ss"
let now = Date()
let dateString = dateFormatter.string(from: now)
print(dateString)  // "30/09/2026 12:00:00" (ขึ้นอยู่กับเวลาปัจจุบัน)

// Locale-aware formatting
dateFormatter.locale = Locale(identifier: "th_TH")
dateFormatter.dateStyle = .long
let thaiDate = dateFormatter.string(from: now)
print(thaiDate)  // วันที่แบบไทย
```

---

## 15. String Encoding (การเข้ารหัสสตริง)

```swift
import Foundation

let str = "สวัสดี Hello 🌏"

// UTF-8 Encoding
if let utf8Data = str.data(using: .utf8) {
    print("UTF-8 bytes: \(utf8Data.count)")
    
    // Decode กลับ
    if let decoded = String(data: utf8Data, encoding: .utf8) {
        print("Decoded: \(decoded)")
    }
}

// UTF-16 Encoding
if let utf16Data = str.data(using: .utf16) {
    print("UTF-16 bytes: \(utf16Data.count)")
}

// ISO-8859-11 (Thai encoding)
// Note: อาจไม่รองรับบางตัวอักษร
if let isoData = str.data(using: .isoLatin1) {
    print("ISO-8859-1 bytes: \(isoData.count)")
} else {
    print("ไม่สามารถ encode ด้วย ISO-8859-1 ได้")
}

// การ encode/decode Base64
func encodeBase64(_ string: String) -> String? {
    return string.data(using: .utf8)?.base64EncodedString()
}

func decodeBase64(_ base64: String) -> String? {
    guard let data = Data(base64Encoded: base64) else { return nil }
    return String(data: data, encoding: .utf8)
}

let original = "สวัสดีชาวโลก"
if let encoded = encodeBase64(original) {
    print("Base64: \(encoded)")
    if let decoded = decodeBase64(encoded) {
        print("Decoded: \(decoded)")  // "สวัสดีชาวโลก"
    }
}

// URL Encoding
let urlString = "https://example.com/search?q=สวัสดี ชาวโลก"
if let encoded = urlString.addingPercentEncoding(withAllowedCharacters: .urlQueryAllowed) {
    print("URL Encoded: \(encoded)")
}

// URL Decoding
let encodedURL = "https://example.com/search?q=%E0%B8%AA%E0%B8%A7%E0%B8%B1%E0%B8%AA%E0%B8%94%E0%B8%B5"
if let decoded = encodedURL.removingPercentEncoding {
    print("URL Decoded: \(decoded)")
}
```

---

## 16. Substring vs String

```swift
// Substring คือ view ของ String ต้นฉบับ ไม่มีการ copy memory
let fullString = "Hello, Swift World!"

// สร้าง Substring
let substring: Substring = fullString.prefix(5)
print(substring)       // "Hello"
print(type(of: substring))  // Substring

// Substring ยังอ้างอิง memory ของ String ต้นฉบับ
// ดังนั้นควรแปลงเป็น String เมื่อต้องการเก็บค่าระยะยาว
let newString: String = String(substring)
print(type(of: newString))  // String

// ตัวอย่างที่แสดงความแตกต่าง
func processString(_ s: String) -> String {
    return s.uppercased()
}

// ต้องแปลง Substring เป็น String ก่อน
let sub = fullString[fullString.startIndex..<fullString.index(fullString.startIndex, offsetBy: 5)]
let result = processString(String(sub))
print(result)  // "HELLO"

// Substring methods (เหมือน String เกือบทั้งหมด)
let sentence = "The quick brown fox"
let words = sentence.split(separator: " ")

for word in words {
    // word เป็น Substring
    let wordString = String(word)
    print("\(wordString): \(wordString.count) characters")
}

// Prefix, Suffix, DropFirst, DropLast
let greeting4 = "Hello, World!"
print(greeting4.prefix(5))           // "Hello"
print(greeting4.suffix(6))           // "World!"
print(greeting4.dropFirst(7))        // "World!"
print(greeting4.dropLast(1))         // "Hello, World"
print(greeting4.dropFirst(7).dropLast(1))  // "World"
```

---

## 17. String Conversion (การแปลงประเภทข้อมูล)

```swift
// String <-> Int
let intStr = "42"
if let number = Int(intStr) {
    print("Int: \(number)")       // 42
    print("String: \(String(number))")  // "42"
}

// ตรวจสอบก่อนแปลง
let invalidStr = "abc"
if let num = Int(invalidStr) {
    print("Valid: \(num)")
} else {
    print("ไม่ใช่ตัวเลข!")  // จะแสดงข้อความนี้
}

// String <-> Double
let doubleStr = "3.14"
if let decimal = Double(doubleStr) {
    print("Double: \(decimal)")
}

// String <-> Float
let floatStr = "1.5"
if let float = Float(floatStr) {
    print("Float: \(float)")
}

// String <-> Bool
let boolStr = "true"
if let bool = Bool(boolStr) {
    print("Bool: \(bool)")  // true
}

// Number ฐานต่างๆ
let hex = Int("FF", radix: 16)   // 255
let binary = Int("1010", radix: 2)  // 10
let octal = Int("17", radix: 8)  // 15

print("Hex: \(hex!)")
print("Binary: \(binary!)")
print("Octal: \(octal!)")

// แปลง Int เป็น String ฐานต่างๆ
let num = 255
print(String(num, radix: 16))  // "ff"
print(String(num, radix: 2))   // "11111111"
print(String(num, radix: 8))   // "377"

// String <-> [Character]
let strToChars = "Hello"
let charArray = Array(strToChars)
print(charArray)  // ["H", "e", "l", "l", "o"]

let charsToStr = String(charArray)
print(charsToStr)  // "Hello"

// String <-> Data
let strToData = "Hello World"
let data = strToData.data(using: .utf8)!
let dataToStr = String(data: data, encoding: .utf8)!
print(dataToStr)  // "Hello World"
```

---

## 18. Common String Algorithms (อัลกอริทึมสตริงที่ใช้บ่อย)

```swift
import Foundation

// 1. Palindrome check
func isPalindrome(_ str: String) -> Bool {
    let cleaned = str.lowercased().filter { $0.isLetter || $0.isNumber }
    return cleaned == String(cleaned.reversed())
}

print(isPalindrome("รถนนทร"))  // true
print(isPalindrome("madam"))   // true
print(isPalindrome("hello"))   // false

// 2. Anagram check
func isAnagram(_ str1: String, _ str2: String) -> Bool {
    let sorted1 = str1.lowercased().filter { !$0.isWhitespace }.sorted()
    let sorted2 = str2.lowercased().filter { !$0.isWhitespace }.sorted()
    return sorted1 == sorted2
}

print(isAnagram("listen", "silent"))  // true
print(isAnagram("hello", "world"))    // false

// 3. Count occurrences
func countOccurrences(of substring: String, in text: String) -> Int {
    var count = 0
    var searchRange = text.startIndex..<text.endIndex
    
    while let range = text.range(of: substring, range: searchRange) {
        count += 1
        searchRange = range.upperBound..<text.endIndex
    }
    
    return count
}

let text3 = "banana"
print(countOccurrences(of: "a", in: text3))  // 3
print(countOccurrences(of: "na", in: text3)) // 2

// 4. Reverse words
func reverseWords(_ sentence: String) -> String {
    return sentence.split(separator: " ")
                   .reversed()
                   .joined(separator: " ")
}

print(reverseWords("Hello World Swift"))  // "Swift World Hello"

// 5. Longest common prefix
func longestCommonPrefix(_ strings: [String]) -> String {
    guard !strings.isEmpty else { return "" }
    guard strings.count > 1 else { return strings[0] }
    
    var prefix = strings[0]
    
    for string in strings.dropFirst() {
        while !string.hasPrefix(prefix) {
            prefix = String(prefix.dropLast())
            if prefix.isEmpty { return "" }
        }
    }
    
    return prefix
}

let words2 = ["flower", "flow", "flight"]
print(longestCommonPrefix(words2))  // "fl"

// 6. Caesar cipher
func caesarCipher(_ text: String, shift: Int) -> String {
    let shift = ((shift % 26) + 26) % 26
    return String(text.map { char -> Character in
        guard char.isLetter else { return char }
        let base: UInt32 = char.isUppercase ? 65 : 97
        let charValue = char.asciiValue.map { UInt32($0) } ?? 0
        let shifted = (charValue - base + UInt32(shift)) % 26 + base
        return Character(UnicodeScalar(shifted)!)
    })
}

print(caesarCipher("Hello World", shift: 3))   // "Khoor Zruog"
print(caesarCipher("Khoor Zruog", shift: -3))  // "Hello World"

// 7. Word frequency count
func wordFrequency(_ text: String) -> [String: Int] {
    let words = text.lowercased()
                    .components(separatedBy: .whitespacesAndNewlines)
                    .filter { !$0.isEmpty }
    
    var frequency: [String: Int] = [:]
    for word in words {
        let cleaned = word.trimmingCharacters(in: .punctuationCharacters)
        frequency[cleaned, default: 0] += 1
    }
    return frequency
}

let paragraph = "the quick brown fox jumps over the lazy dog the fox"
let freq = wordFrequency(paragraph)
let sorted = freq.sorted { $0.value > $1.value }
for (word, count) in sorted.prefix(5) {
    print("\(word): \(count)")
}
// the: 3
// fox: 2
// ...
```

---

## 19. Real-world Examples

### 19.1 Email Validation

```swift
import Foundation

struct EmailValidator {
    static let pattern = "^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}$"
    
    static func isValid(_ email: String) -> Bool {
        guard let regex = try? NSRegularExpression(pattern: pattern) else {
            return false
        }
        let range = NSRange(email.startIndex..., in: email)
        return regex.firstMatch(in: email, range: range) != nil
    }
    
    static func extractDomain(from email: String) -> String? {
        guard isValid(email) else { return nil }
        let parts = email.components(separatedBy: "@")
        return parts.last
    }
    
    static func normalize(_ email: String) -> String {
        return email.lowercased().trimmingCharacters(in: .whitespaces)
    }
}

// ทดสอบ
let emails2 = [
    "user@example.com",
    "invalid-email",
    "user@.com",
    "user@domain.co.th",
    "  User@Example.COM  "
]

for email3 in emails2 {
    let normalized = EmailValidator.normalize(email3)
    let valid = EmailValidator.isValid(normalized)
    let domain = EmailValidator.extractDomain(from: normalized)
    print("\(email3): valid=\(valid), domain=\(domain ?? "none")")
}
```

### 19.2 URL Parsing

```swift
import Foundation

struct URLParser {
    let url: URL
    
    init?(_ urlString: String) {
        guard let url = URL(string: urlString) else { return nil }
        self.url = url
    }
    
    var scheme: String { url.scheme ?? "" }
    var host: String { url.host ?? "" }
    var path: String { url.path }
    var port: Int? { url.port }
    var fragment: String? { url.fragment }
    
    var queryParameters: [String: String] {
        var params: [String: String] = [:]
        guard let components = URLComponents(url: url, resolvingAgainstBaseURL: false),
              let queryItems = components.queryItems else {
            return params
        }
        for item in queryItems {
            params[item.name] = item.value ?? ""
        }
        return params
    }
}

let urlString2 = "https://api.example.com:8080/users/search?name=สมชาย&age=25#results"

if let parser = URLParser(urlString2) {
    print("Scheme: \(parser.scheme)")       // "https"
    print("Host: \(parser.host)")           // "api.example.com"
    print("Port: \(parser.port ?? 80)")     // 8080
    print("Path: \(parser.path)")           // "/users/search"
    print("Fragment: \(parser.fragment ?? "")") // "results"
    
    let params = parser.queryParameters
    print("Parameters:")
    for (key, value) in params {
        print("  \(key) = \(value)")
    }
    // name = สมชาย
    // age = 25
}
```

### 19.3 Text Processing

```swift
import Foundation

class TextProcessor {
    var text: String
    
    init(_ text: String) {
        self.text = text
    }
    
    // สรุปข้อความ (ตัดให้เหลือ n คำ)
    func summarize(maxWords: Int) -> String {
        let words = text.components(separatedBy: .whitespacesAndNewlines)
                        .filter { !$0.isEmpty }
        if words.count <= maxWords {
            return text
        }
        return words.prefix(maxWords).joined(separator: " ") + "..."
    }
    
    // นับคำ
    var wordCount: Int {
        text.components(separatedBy: .whitespacesAndNewlines)
            .filter { !$0.isEmpty }
            .count
    }
    
    // นับประโยค
    var sentenceCount: Int {
        text.components(separatedBy: CharacterSet(charactersIn: ".!?"))
            .filter { !$0.trimmingCharacters(in: .whitespaces).isEmpty }
            .count
    }
    
    // ลบ HTML tags
    func stripHTML() -> String {
        return text.replacingOccurrences(
            of: "<[^>]+>",
            with: "",
            options: .regularExpression
        )
    }
    
    // Slug สำหรับ URL
    func toSlug() -> String {
        return text.lowercased()
                   .replacingOccurrences(of: " ", with: "-")
                   .replacingOccurrences(of: "[^a-z0-9-]", with: "", options: .regularExpression)
                   .components(separatedBy: "-")
                   .filter { !$0.isEmpty }
                   .joined(separator: "-")
    }
    
    // Highlight ข้อความ
    func highlight(_ keyword: String) -> String {
        return text.replacingOccurrences(
            of: keyword,
            with: "[\(keyword)]",
            options: .caseInsensitive
        )
    }
    
    // Extract ตัวเลขทั้งหมด
    func extractNumbers() -> [Double] {
        let pattern = "-?\\d+\\.?\\d*"
        guard let regex = try? NSRegularExpression(pattern: pattern) else { return [] }
        
        let range = NSRange(text.startIndex..., in: text)
        let matches = regex.matches(in: text, range: range)
        
        return matches.compactMap { match -> Double? in
            guard let range = Range(match.range, in: text) else { return nil }
            return Double(text[range])
        }
    }
}

// ทดสอบ
let article = """
    Swift เป็นภาษาโปรแกรมที่ทรงพลัง ออกแบบโดย Apple ในปี 2014
    ใช้สำหรับพัฒนา iOS, macOS, watchOS และ tvOS
    Swift version 5.9 มีฟีเจอร์ใหม่มากมาย รวมถึง macros
    ราคาเครื่องมือ Developer Program อยู่ที่ 99 ดอลลาร์ต่อปี
    """

let processor = TextProcessor(article)
print("จำนวนคำ: \(processor.wordCount)")
print("สรุป: \(processor.summarize(maxWords: 10))")
print("ตัวเลข: \(processor.extractNumbers())")
print("Slug: \(processor.toSlug())")
print("Highlight: \(processor.highlight("Swift"))")
```

---

## 20. Practical Exercises (แบบฝึกหัดพร้อมเฉลย)

### Exercise 1: String Reversal

```swift
// โจทย์: เขียนฟังก์ชันที่กลับตัวอักษรใน string โดยรักษา word order
// Input: "Hello World" -> Output: "olleH dlroW"

func reverseEachWord(_ sentence: String) -> String {
    return sentence.split(separator: " ")
                   .map { String($0.reversed()) }
                   .joined(separator: " ")
}

print(reverseEachWord("Hello World"))         // "olleH dlroW"
print(reverseEachWord("Swift Programming"))   // "tfiwS gnimmargorP"
print(reverseEachWord("สวัสดี ชาวโลก"))        // "ีดัสวส กโวาช"
```

### Exercise 2: Password Validator

```swift
// โจทย์: ตรวจสอบรหัสผ่านตาม criteria ต่อไปนี้:
// - ความยาวอย่างน้อย 8 ตัวอักษร
// - มีตัวพิมพ์ใหญ่อย่างน้อย 1 ตัว
// - มีตัวพิมพ์เล็กอย่างน้อย 1 ตัว
// - มีตัวเลขอย่างน้อย 1 ตัว
// - มีอักขระพิเศษอย่างน้อย 1 ตัว

struct PasswordValidator {
    enum ValidationError: Error, CustomStringConvertible {
        case tooShort(minimum: Int)
        case noUppercase
        case noLowercase
        case noNumber
        case noSpecialCharacter
        
        var description: String {
            switch self {
            case .tooShort(let min): return "รหัสผ่านต้องมีอย่างน้อย \(min) ตัวอักษร"
            case .noUppercase: return "ต้องมีตัวพิมพ์ใหญ่อย่างน้อย 1 ตัว"
            case .noLowercase: return "ต้องมีตัวพิมพ์เล็กอย่างน้อย 1 ตัว"
            case .noNumber: return "ต้องมีตัวเลขอย่างน้อย 1 ตัว"
            case .noSpecialCharacter: return "ต้องมีอักขระพิเศษอย่างน้อย 1 ตัว"
            }
        }
    }
    
    static func validate(_ password: String) -> [ValidationError] {
        var errors: [ValidationError] = []
        
        if password.count < 8 {
            errors.append(.tooShort(minimum: 8))
        }
        if !password.contains(where: { $0.isUppercase }) {
            errors.append(.noUppercase)
        }
        if !password.contains(where: { $0.isLowercase }) {
            errors.append(.noLowercase)
        }
        if !password.contains(where: { $0.isNumber }) {
            errors.append(.noNumber)
        }
        let specialChars = "!@#$%^&*()_+-=[]{}|;:,.<>?"
        if !password.contains(where: { specialChars.contains($0) }) {
            errors.append(.noSpecialCharacter)
        }
        
        return errors
    }
    
    static func isValid(_ password: String) -> Bool {
        return validate(password).isEmpty
    }
    
    static func strength(_ password: String) -> String {
        let errors = validate(password)
        switch errors.count {
        case 0: return "แข็งแกร่ง"
        case 1: return "ปานกลาง"
        case 2: return "อ่อนแอ"
        default: return "อ่อนแอมาก"
        }
    }
}

let passwords = ["abc", "Password1!", "weakpass", "Str0ng!Pass"]
for pass in passwords {
    let errors = PasswordValidator.validate(pass)
    let strength = PasswordValidator.strength(pass)
    print("\(pass): \(strength)")
    if !errors.isEmpty {
        for error in errors {
            print("  - \(error)")
        }
    }
}
```

### Exercise 3: Text Statistics

```swift
// โจทย์: วิเคราะห์ข้อความและแสดงสถิติต่างๆ

struct TextStatistics {
    let text: String
    
    var characterCount: Int { text.count }
    var characterCountWithoutSpaces: Int {
        text.filter { !$0.isWhitespace }.count
    }
    
    var wordCount: Int {
        text.components(separatedBy: .whitespacesAndNewlines)
            .filter { !$0.isEmpty }
            .count
    }
    
    var lineCount: Int {
        text.components(separatedBy: "\n").count
    }
    
    var uniqueWordCount: Int {
        Set(text.lowercased()
                .components(separatedBy: .whitespacesAndNewlines)
                .filter { !$0.isEmpty }
                .map { $0.trimmingCharacters(in: .punctuationCharacters) })
            .count
    }
    
    var mostFrequentWords: [(word: String, count: Int)] {
        let words = text.lowercased()
                        .components(separatedBy: .whitespacesAndNewlines)
                        .filter { !$0.isEmpty }
                        .map { $0.trimmingCharacters(in: .punctuationCharacters) }
        
        var freq: [String: Int] = [:]
        for word in words { freq[word, default: 0] += 1 }
        
        return freq.sorted { $0.value > $1.value }
                   .prefix(5)
                   .map { (word: $0.key, count: $0.value) }
    }
    
    var averageWordLength: Double {
        let words = text.components(separatedBy: .whitespacesAndNewlines)
                        .filter { !$0.isEmpty }
        guard !words.isEmpty else { return 0 }
        let totalLength = words.reduce(0) { $0 + $1.count }
        return Double(totalLength) / Double(words.count)
    }
    
    func report() -> String {
        var lines = [
            "=== สถิติข้อความ ===",
            "จำนวนตัวอักษรทั้งหมด: \(characterCount)",
            "จำนวนตัวอักษร (ไม่รวม space): \(characterCountWithoutSpaces)",
            "จำนวนคำ: \(wordCount)",
            "จำนวนคำที่ไม่ซ้ำ: \(uniqueWordCount)",
            "จำนวนบรรทัด: \(lineCount)",
            String(format: "ความยาวคำเฉลี่ย: %.1f", averageWordLength),
            "\nคำที่พบบ่อยที่สุด:"
        ]
        
        for (word, count) in mostFrequentWords {
            lines.append("  '\(word)': \(count) ครั้ง")
        }
        
        return lines.joined(separator: "\n")
    }
}

let sampleText = """
    Swift is a powerful and intuitive programming language for Apple platforms.
    Swift code is interactive and fun, the syntax is concise yet expressive.
    Swift includes modern features developers love and gives the freedom to code 
    with a design philosophy that Apple embraces.
    """

let stats = TextStatistics(text: sampleText)
print(stats.report())
```

### Exercise 4: Markdown Parser (Simple)

```swift
// โจทย์: แปลง Markdown บางส่วนเป็น HTML

struct MarkdownParser {
    static func parse(_ markdown: String) -> String {
        var html = markdown
        
        // Headers
        let headerPatterns = [
            ("^### (.+)$", "<h3>$1</h3>"),
            ("^## (.+)$", "<h2>$1</h2>"),
            ("^# (.+)$", "<h1>$1</h1>")
        ]
        
        for (pattern, replacement) in headerPatterns {
            if let regex = try? NSRegularExpression(pattern: pattern, options: .anchorsMatchLines) {
                let range = NSRange(html.startIndex..., in: html)
                html = regex.stringByReplacingMatches(in: html, range: range, withTemplate: replacement)
            }
        }
        
        // Bold
        if let boldRegex = try? NSRegularExpression(pattern: "\\*\\*(.+?)\\*\\*") {
            let range = NSRange(html.startIndex..., in: html)
            html = boldRegex.stringByReplacingMatches(in: html, range: range, withTemplate: "<strong>$1</strong>")
        }
        
        // Italic
        if let italicRegex = try? NSRegularExpression(pattern: "\\*(.+?)\\*") {
            let range = NSRange(html.startIndex..., in: html)
            html = italicRegex.stringByReplacingMatches(in: html, range: range, withTemplate: "<em>$1</em>")
        }
        
        // Code
        if let codeRegex = try? NSRegularExpression(pattern: "`(.+?)`") {
            let range = NSRange(html.startIndex..., in: html)
            html = codeRegex.stringByReplacingMatches(in: html, range: range, withTemplate: "<code>$1</code>")
        }
        
        // Links
        if let linkRegex = try? NSRegularExpression(pattern: "\\[(.+?)\\]\\((.+?)\\)") {
            let range = NSRange(html.startIndex..., in: html)
            html = linkRegex.stringByReplacingMatches(in: html, range: range, withTemplate: "<a href=\"$2\">$1</a>")
        }
        
        // Wrap paragraphs
        let lines = html.components(separatedBy: "\n")
        html = lines.map { line -> String in
            let trimmed = line.trimmingCharacters(in: .whitespaces)
            if trimmed.hasPrefix("<h") || trimmed.isEmpty { return line }
            return "<p>\(line)</p>"
        }.joined(separator: "\n")
        
        return html
    }
}

let markdown = """
    # หัวข้อหลัก
    ## หัวข้อรอง
    
    นี่คือ **ข้อความตัวหนา** และ *ข้อความตัวเอียง*
    ใช้ `print()` เพื่อแสดงผล
    ดูเพิ่มเติมที่ [Swift.org](https://swift.org)
    """

let html2 = MarkdownParser.parse(markdown)
print(html2)
```

---

## 21. สรุป (Summary)

ในบทนี้เราได้เรียนรู้เกี่ยวกับ String และ Character ใน Swift อย่างครบถ้วน ตั้งแต่:

### หัวข้อที่ครอบคลุม

1. **String Fundamentals**: String เป็น value type, การประกาศตัวแปร
2. **String Literals**: Single-line, multi-line, extended delimiters
3. **String Interpolation**: การแทรกค่าและ custom interpolation
4. **String Concatenation**: การต่อ string ด้วยวิธีต่างๆ
5. **Characters**: Extended grapheme clusters, Unicode properties
6. **String.Index**: การ navigate ผ่าน string อย่างปลอดภัย
7. **Unicode Support**: UTF-8, UTF-16, Unicode scalars
8. **String Mutability**: การเพิ่ม, ลบ, แทรก characters
9. **String Comparison**: Case-sensitive, locale-aware, natural order
10. **String Searching**: contains, hasPrefix, hasSuffix, range(of:)
11. **String Manipulation**: Case conversion, splitting, trimming, replacing
12. **Regular Expressions**: NSRegularExpression, capture groups
13. **String Formatting**: String(format:), NumberFormatter, DateFormatter
14. **String Encoding**: UTF-8, UTF-16, Base64, URL encoding
15. **Substring vs String**: Memory model, การแปลง
16. **String Conversion**: ระหว่าง String กับ Int, Double, Bool
17. **Common Algorithms**: Palindrome, anagram, Caesar cipher
18. **Real-world Examples**: Email validation, URL parsing, text processing

### Key Takeaways

```swift
// 1. Swift String เป็น Unicode-aware และ value type
let s1 = "café"  // 4 characters
let s2 = "cafe\u{301}"  // 4 characters (สมมูล)
print(s1 == s2)  // true

// 2. ใช้ String.Index ไม่ใช่ integer index
let str = "Hello"
let idx = str.index(str.startIndex, offsetBy: 2)
print(str[idx])  // "l"

// 3. Substring ประหยัด memory, แต่ต้องแปลงเป็น String เมื่อเก็บนาน
let sub2 = str.prefix(3)  // Substring
let fullStr: String = String(sub2)  // String

// 4. ใช้ interpolation แทน concatenation เมื่อทำได้
let name3 = "Swift"
let msg = "Hello, \(name3)!"  // ดีกว่า "Hello, " + name + "!"

// 5. Foundation เพิ่มความสามารถให้ String อีกมาก
import Foundation
let sorted3 = ["file10", "file2"].sorted { 
    $0.localizedStandardCompare($1) == .orderedAscending 
}
```

### Best Practices

- ใช้ `String.Index` แทน integer index เสมอ
- ระวัง performance เมื่อ concatenate string ในลูป ใช้ `joined()` แทน
- ใช้ `Substring` เมื่อต้องการ view ชั่วคราว แต่แปลงเป็น `String` เมื่อจะเก็บค่า
- ใช้ `NSRegularExpression` เมื่อต้องการ pattern matching ที่ซับซ้อน
- ทดสอบกับ Unicode characters (emoji, ภาษาไทย, ภาษาอาหรับ) เสมอ
- ใช้ `localizedCompare` หรือ `localizedStandardCompare` เมื่อเรียงลำดับสำหรับผู้ใช้

---

*จบ Part 11: Strings และ Characters*

> **ต่อไป**: Part 12 จะอธิบาย Enumerations ซึ่งเป็นหนึ่งในฟีเจอร์ที่ทรงพลังที่สุดของ Swift
