# Part 46: Localization และ Internationalization

## บทนำ

Localization (L10n) และ Internationalization (i18n) เป็นกระบวนการทำให้แอปพลิเคชันรองรับผู้ใช้จากหลายภาษาและวัฒนธรรม บทนี้จะพาคุณเรียนรู้ตั้งแต่พื้นฐานจนถึงการทำแอปให้รองรับหลายภาษาอย่างสมบูรณ์

---

## 1. Localization vs Internationalization

### 1.1 Internationalization (i18n)

**Internationalization** คือกระบวนการออกแบบและพัฒนาซอฟต์แวร์เพื่อให้สามารถ adapt กับภาษาและวัฒนธรรมต่างๆ ได้ โดยไม่ต้องแก้ไข code หลัก

คำว่า "i18n" มาจากตัวอักษรแรก "i", ตัวอักษร 18 ตัวกลาง, และตัวอักษรสุดท้าย "n" ของคำว่า internationalization

**สิ่งที่ต้องทำใน i18n:**
- แยก string ที่แสดงผลออกจาก code
- รองรับรูปแบบวันที่และเวลาตาม locale
- รองรับรูปแบบตัวเลขและสกุลเงินตาม locale
- รองรับทิศทางข้อความ (LTR/RTL)
- ใช้ unicode ทุกที่
- ออกแบบ layout ที่ flexible

### 1.2 Localization (L10n)

**Localization** คือกระบวนการแปลและปรับ content ของซอฟต์แวร์สำหรับตลาดหรือ locale เฉพาะ

**สิ่งที่ต้องทำใน L10n:**
- แปลข้อความ
- แปลรูปภาพและ assets
- ปรับรูปแบบวันที่ เวลา ตัวเลข สกุลเงิน
- ปรับ layout สำหรับ RTL languages
- ปรับชื่อแอปและ metadata

### 1.3 ความแตกต่าง

```
i18n = เตรียมความพร้อมของระบบ (ทำครั้งเดียว)
L10n = การแปลและปรับแต่ง content (ทำซ้ำสำหรับแต่ละภาษา)
```

### 1.4 ตัวอย่างเปรียบเทียบ

```swift
// ❌ Hard-coded string - ไม่รองรับ l10n
Text("Hello, World!")

// ✅ Localized string - รองรับ l10n
Text("greeting_message") // อ่านจาก Localizable.strings

// ❌ Hard-coded date format
Text("วันที่: 2024-01-15")

// ✅ Localized date format
let formatter = DateFormatter()
formatter.dateStyle = .long
formatter.locale = Locale.current
Text("วันที่: \(formatter.string(from: Date()))")

// ❌ Hard-coded number format
Text("ราคา: $1,234.56")

// ✅ Localized number format
let price = 1234.56
let formatted = price.formatted(.currency(code: Locale.current.currency?.identifier ?? "USD"))
Text("ราคา: \(formatted)")
```

---

## 2. Xcode Localization Workflow

### 2.1 ขั้นตอนการตั้งค่า Localization

**Step 1: เพิ่มภาษาใน Project Settings**

1. เปิด Project Navigator
2. คลิก Project (ไม่ใช่ Target)
3. ไปที่ "Info" tab
4. ใต้ "Localizations" คลิก "+"
5. เลือกภาษาที่ต้องการ เช่น Thai (th), Japanese (ja)

**Step 2: สร้าง Localizable.strings**

1. File > New > File
2. เลือก "Strings File"
3. ตั้งชื่อว่า "Localizable"
4. คลิก "Create"
5. ใน File Inspector คลิก "Localize..."
6. เลือกภาษา base และภาษาอื่นๆ

### 2.2 โครงสร้างไฟล์ Localization

```
MyApp/
├── en.lproj/
│   ├── Localizable.strings
│   ├── Localizable.stringsdict
│   └── InfoPlist.strings
├── th.lproj/
│   ├── Localizable.strings
│   ├── Localizable.stringsdict
│   └── InfoPlist.strings
└── Base.lproj/
    ├── Main.storyboard
    └── LaunchScreen.storyboard
```

### 2.3 ตัวอย่าง Xcode Project Setup

```swift
// AppDelegate.swift หรือ App.swift
// ไม่จำเป็นต้องทำอะไรพิเศษ iOS จัดการ locale อัตโนมัติ

@main
struct LocalizedApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}

// ContentView.swift
struct ContentView: View {
    var body: some View {
        VStack {
            Text("app_title")  // จะหา key นี้ใน Localizable.strings
            Text("welcome_message")
        }
    }
}
```

---

## 3. Localizable.strings

### 3.1 รูปแบบของ Localizable.strings

```
/* Comment describing the string */
"key" = "value";
```

### 3.2 ตัวอย่าง Localizable.strings

**en.lproj/Localizable.strings:**
```
/* App title shown in navigation bar */
"app_title" = "My Shopping App";

/* Welcome message on home screen */
"welcome_message" = "Welcome to the app!";

/* Button labels */
"button_save" = "Save";
"button_cancel" = "Cancel";
"button_delete" = "Delete";
"button_edit" = "Edit";
"button_add" = "Add";

/* Error messages */
"error_network" = "Network connection failed. Please try again.";
"error_not_found" = "Item not found.";
"error_unknown" = "An unknown error occurred.";

/* Navigation titles */
"nav_home" = "Home";
"nav_cart" = "Cart";
"nav_profile" = "Profile";
"nav_settings" = "Settings";

/* Product related */
"product_price" = "Price: %@";
"product_in_stock" = "In Stock";
"product_out_of_stock" = "Out of Stock";
"product_rating" = "Rating: %.1f stars";
```

**th.lproj/Localizable.strings:**
```
/* App title shown in navigation bar */
"app_title" = "แอปช็อปปิ้ง";

/* Welcome message on home screen */
"welcome_message" = "ยินดีต้อนรับสู่แอป!";

/* Button labels */
"button_save" = "บันทึก";
"button_cancel" = "ยกเลิก";
"button_delete" = "ลบ";
"button_edit" = "แก้ไข";
"button_add" = "เพิ่ม";

/* Error messages */
"error_network" = "การเชื่อมต่อเครือข่ายล้มเหลว กรุณาลองใหม่อีกครั้ง";
"error_not_found" = "ไม่พบรายการที่ค้นหา";
"error_unknown" = "เกิดข้อผิดพลาดที่ไม่ทราบสาเหตุ";

/* Navigation titles */
"nav_home" = "หน้าหลัก";
"nav_cart" = "ตะกร้า";
"nav_profile" = "โปรไฟล์";
"nav_settings" = "การตั้งค่า";

/* Product related */
"product_price" = "ราคา: %@";
"product_in_stock" = "มีสินค้า";
"product_out_of_stock" = "สินค้าหมด";
"product_rating" = "คะแนน: %.1f ดาว";
```

---

## 4. String Keys และ Values

### 4.1 หลักการตั้งชื่อ Key

```swift
// รูปแบบ key ที่ดี:
// - ใช้ snake_case
// - ใช้ namespace ด้วย prefix
// - ให้ชื่อ key สื่อถึง context

// ✅ ดี
"home_screen_title" = "หน้าหลัก";
"settings_notifications_toggle" = "การแจ้งเตือน";
"error_network_connection" = "ไม่สามารถเชื่อมต่ออินเทอร์เน็ต";
"button_confirm_delete" = "ยืนยันการลบ";

// ❌ ไม่ดี
"title" = "หน้าหลัก";             // ไม่มี context
"ok" = "ตกลง";                    // ทั่วไปเกินไป
"The network is disconnected" = "ไม่สามารถเชื่อมต่อ"; // ใช้ค่าเป็น key
```

### 4.2 การจัดกลุ่ม Keys

สำหรับโปรเจ็กต์ใหญ่ ควรแบ่งเป็นหลายไฟล์:

```
// สร้างไฟล์หลายไฟล์:
// Localizable.strings     - strings ทั่วไป
// OnboardingStrings.strings
// ProfileStrings.strings
// SettingsStrings.strings
```

```swift
// ใช้ tableName parameter:
NSLocalizedString("welcome_title", tableName: "Onboarding", comment: "Onboarding welcome title")

// หรือใน SwiftUI:
Text("welcome_title", tableName: "Onboarding")
```

### 4.3 String Constants

```swift
// สร้าง Constants enum เพื่อหลีกเลี่ยงการพิมพ์ key ผิด
enum L10n {
    enum Home {
        static let title = NSLocalizedString("home_screen_title", comment: "Home screen title")
        static let welcome = NSLocalizedString("home_welcome_message", comment: "Welcome message")
    }
    
    enum Settings {
        static let title = NSLocalizedString("settings_title", comment: "Settings title")
        static let notifications = NSLocalizedString("settings_notifications_toggle", comment: "")
        static let language = NSLocalizedString("settings_language", comment: "")
    }
    
    enum Error {
        static let network = NSLocalizedString("error_network_connection", comment: "")
        static let notFound = NSLocalizedString("error_not_found", comment: "")
        
        static func unknown(code: Int) -> String {
            String(format: NSLocalizedString("error_unknown_code", comment: ""), code)
        }
    }
    
    enum Button {
        static let save = NSLocalizedString("button_save", comment: "")
        static let cancel = NSLocalizedString("button_cancel", comment: "")
        static let delete = NSLocalizedString("button_delete", comment: "")
    }
}

// การใช้งาน
struct HomeView: View {
    var body: some View {
        VStack {
            Text(L10n.Home.title)
            Text(L10n.Home.welcome)
            Button(L10n.Button.save) {}
        }
    }
}
```

---

## 5. Pluralization Rules (Localizable.stringsdict)

### 5.1 ปัญหาของ Pluralization

ภาษาต่างๆ มีกฎ plural ที่แตกต่างกัน:
- **ภาษาไทย**: ไม่มี plural form (1 คน, 2 คน เหมือนกัน)
- **ภาษาอังกฤษ**: singular (1 item) และ plural (2 items)
- **ภาษาอาหรับ**: มี 6 plural forms
- **ภาษารัสเซีย**: มี 3 plural forms

### 5.2 รูปแบบ stringsdict

**en.lproj/Localizable.stringsdict:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" 
    "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <!-- จำนวน items ในตะกร้า -->
    <key>cart_item_count</key>
    <dict>
        <key>NSStringLocalizedFormatKey</key>
        <string>%#@items@</string>
        <key>items</key>
        <dict>
            <key>NSStringFormatSpecTypeKey</key>
            <string>NSStringPluralRuleType</string>
            <key>NSStringFormatValueTypeKey</key>
            <string>d</string>
            <key>zero</key>
            <string>No items in cart</string>
            <key>one</key>
            <string>%d item in cart</string>
            <key>other</key>
            <string>%d items in cart</string>
        </dict>
    </dict>
    
    <!-- จำนวนวันที่เหลือ -->
    <key>days_remaining</key>
    <dict>
        <key>NSStringLocalizedFormatKey</key>
        <string>%#@days@</string>
        <key>days</key>
        <dict>
            <key>NSStringFormatSpecTypeKey</key>
            <string>NSStringPluralRuleType</string>
            <key>NSStringFormatValueTypeKey</key>
            <string>d</string>
            <key>one</key>
            <string>%d day remaining</string>
            <key>other</key>
            <string>%d days remaining</string>
        </dict>
    </dict>
    
    <!-- จำนวนผู้ใช้ที่ online - ซับซ้อนขึ้น -->
    <key>users_online_format</key>
    <dict>
        <key>NSStringLocalizedFormatKey</key>
        <string>%#@users@ online</string>
        <key>users</key>
        <dict>
            <key>NSStringFormatSpecTypeKey</key>
            <string>NSStringPluralRuleType</string>
            <key>NSStringFormatValueTypeKey</key>
            <string>d</string>
            <key>zero</key>
            <string>No users</string>
            <key>one</key>
            <string>%d user</string>
            <key>other</key>
            <string>%d users</string>
        </dict>
    </dict>
</dict>
</plist>
```

**th.lproj/Localizable.stringsdict:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" 
    "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <!-- ภาษาไทยไม่มี plural form ใช้ other เสมอ -->
    <key>cart_item_count</key>
    <dict>
        <key>NSStringLocalizedFormatKey</key>
        <string>%#@items@</string>
        <key>items</key>
        <dict>
            <key>NSStringFormatSpecTypeKey</key>
            <string>NSStringPluralRuleType</string>
            <key>NSStringFormatValueTypeKey</key>
            <string>d</string>
            <key>zero</key>
            <string>ไม่มีสินค้าในตะกร้า</string>
            <key>other</key>
            <string>%d ชิ้นในตะกร้า</string>
        </dict>
    </dict>
    
    <key>days_remaining</key>
    <dict>
        <key>NSStringLocalizedFormatKey</key>
        <string>%#@days@</string>
        <key>days</key>
        <dict>
            <key>NSStringFormatSpecTypeKey</key>
            <string>NSStringPluralRuleType</string>
            <key>NSStringFormatValueTypeKey</key>
            <string>d</string>
            <key>other</key>
            <string>เหลือ %d วัน</string>
        </dict>
    </dict>
</dict>
</plist>
```

### 5.3 การใช้งาน stringsdict

```swift
struct PluralizationExample: View {
    let itemCount = 3
    let daysRemaining = 1
    
    var body: some View {
        VStack {
            // ใช้ String(format:) กับ NSLocalizedString
            Text(String(format: NSLocalizedString("cart_item_count", comment: ""), itemCount))
            // EN: "3 items in cart"
            // TH: "3 ชิ้นในตะกร้า"
            
            Text(String(format: NSLocalizedString("days_remaining", comment: ""), daysRemaining))
            // EN: "1 day remaining"
            // TH: "เหลือ 1 วัน"
        }
    }
}

// หรือใน Swift 5.7+
struct ModernPluralizationExample: View {
    let count = 5
    
    var body: some View {
        // String(localized:) รองรับ stringsdict อัตโนมัติ
        let message = String(localized: "cart_item_count \(count)")
        Text(message)
    }
}
```

---

## 6. NSLocalizedString Macro

### 6.1 รูปแบบ NSLocalizedString

```swift
NSLocalizedString(_ key: String, 
                  tableName: String?, 
                  bundle: Bundle, 
                  value: String, 
                  comment: String) -> String
```

### 6.2 ตัวอย่างการใช้งาน

```swift
import Foundation

// รูปแบบพื้นฐาน
let title = NSLocalizedString("home_title", comment: "Title of home screen")

// ระบุ table name สำหรับไฟล์ strings อื่น
let onboardingText = NSLocalizedString(
    "onboarding_welcome",
    tableName: "Onboarding",
    comment: "Welcome text on onboarding screen"
)

// ระบุ bundle สำหรับ framework
let frameworkString = NSLocalizedString(
    "framework_message",
    bundle: Bundle(for: MyFrameworkClass.self),
    comment: "Message from framework"
)

// ระบุ default value ถ้าไม่พบ key
let withDefault = NSLocalizedString(
    "optional_feature_title",
    value: "Optional Feature",
    comment: "Title of optional feature"
)

// Format string
let count = 5
let itemsText = String(
    format: NSLocalizedString("item_count_format", comment: "Number of items"),
    count
)
// Localizable.strings: "item_count_format" = "%d รายการ";
```

### 6.3 Helper Function

```swift
// สร้าง convenience function
func L(_ key: String, _ args: CVarArg...) -> String {
    let format = NSLocalizedString(key, comment: "")
    return String(format: format, arguments: args)
}

// การใช้งาน
let text1 = L("welcome_message")
let text2 = L("item_count", 5)         // "5 รายการ"
let text3 = L("user_greeting", "สมชาย") // "สวัสดี, สมชาย!"
```

---

## 7. String(localized:) (Swift 5.7+)

### 7.1 String Interpolation ใหม่

Swift 5.7 แนะนำวิธีใหม่ในการทำ localization ที่สะดวกกว่า `NSLocalizedString`

```swift
import SwiftUI

// รูปแบบใหม่ใน Swift 5.7+
let greeting = String(localized: "greeting_message")

// กับ arguments
let name = "สมชาย"
let welcomeMsg = String(localized: "welcome_format \(name)")

// กำหนด table
let onboardingMsg = String(
    localized: "onboarding_title",
    table: "Onboarding"
)

// กำหนด bundle
let bundleMsg = String(
    localized: "bundle_message",
    bundle: .module  // สำหรับ Swift Package
)

// กำหนด locale เฉพาะ
let englishMsg = String(
    localized: "message_key",
    locale: Locale(identifier: "en")
)

// กำหนด comment
let commentedMsg = String(
    localized: "message_key",
    comment: "Message shown to user on first launch"
)
```

### 7.2 LocalizedStringKey ใน SwiftUI

```swift
struct SwiftUILocalizationExample: View {
    let username: String = "สมชาย"
    
    var body: some View {
        VStack {
            // Text ยอมรับ LocalizedStringKey อัตโนมัติ
            Text("greeting_message")
            // เหมือนกับ: Text(LocalizedStringKey("greeting_message"))
            
            // String interpolation ใน Text
            Text("Welcome, \(username)!")
            // SwiftUI จะหา key: "Welcome, \(username)!" ใน strings file
            
            // ระบุ table name
            Text("onboarding_title", tableName: "Onboarding")
            
            // ระบุ bundle
            Text("bundle_key", bundle: .module)
            
            // ระบุ comment สำหรับ translator
            Text("complex_message", comment: "Shown after completing tutorial")
        }
    }
}
```

### 7.3 String Catalog (.xcstrings) - Xcode 15+

```swift
// ตั้งแต่ Xcode 15, สามารถใช้ String Catalog ได้
// ไฟล์ .xcstrings แทน .strings และ .stringsdict
// จัดการได้ง่ายกว่าในรูปแบบ JSON

// การใช้งานยังเหมือนเดิม - ไม่ต้องเปลี่ยน code
Text("greeting_message")
String(localized: "greeting_message")
NSLocalizedString("greeting_message", comment: "")
```

---

## 8. Format Strings Localization

### 8.1 Format Specifiers

```swift
// Format specifiers ที่ใช้บ่อย:
// %@ - String
// %d - Integer
// %f - Float/Double
// %.2f - Float/Double ทศนิยม 2 ตำแหน่ง
// %ld - Long integer
// %lu - Unsigned long integer

// Localizable.strings:
// "greeting_with_name" = "สวัสดี, %@!";
// "item_price" = "ราคา: %.2f บาท";
// "page_number" = "หน้าที่ %d จาก %d";

// การใช้งาน
let name = "สมชาย"
let greeting = String(format: NSLocalizedString("greeting_with_name", comment: ""), name)
// → "สวัสดี, สมชาย!"

let price = 299.99
let priceText = String(format: NSLocalizedString("item_price", comment: ""), price)
// → "ราคา: 299.99 บาท"

let currentPage = 3
let totalPages = 10
let pageText = String(format: NSLocalizedString("page_number", comment: ""), currentPage, totalPages)
// → "หน้าที่ 3 จาก 10"
```

### 8.2 Positional Format Specifiers

```swift
// ปัญหา: ภาษาต่างๆ อาจเรียงลำดับ argument ต่างกัน
// ภาษาอังกฤษ: "John sent 5 messages to Mary"
// ภาษาญี่ปุ่น: "Johnはmaryに5件のメッセージを送りました"

// ❌ ไม่ดี: ลำดับ argument ตายตัว
"en" = "%@ sent %d messages to %@";
"ja" = "%@は%@に%dのメッセージを送りました"; // ลำดับต่างกัน! ผิดพลาด

// ✅ ดี: ใช้ positional specifiers
"en" = "%1$@ sent %2$d messages to %3$@";
"ja" = "%1$@は%3$@に%2$dのメッセージを送りました"; // สามารถสลับลำดับได้

// การใช้งาน
let sender = "John"
let messageCount = 5
let recipient = "Mary"
let text = String(format: NSLocalizedString("message_sent_format", comment: ""), 
                  sender, messageCount, recipient)
```

### 8.3 Attributed String Localization

```swift
import SwiftUI

struct AttributedLocalizationExample: View {
    var body: some View {
        // SwiftUI AttributedString รองรับ markdown ใน localized strings
        if let attributedString = try? AttributedString(
            localized: "terms_and_conditions_markdown",
            options: .init(interpretedSyntax: .inlineOnlyPreservingWhitespace)
        ) {
            Text(attributedString)
        }
    }
}

// Localizable.strings:
// "terms_and_conditions_markdown" = "โดยการใช้แอปนี้ คุณยอมรับ **เงื่อนไขการใช้งาน** ของเรา";
```

---

## 9. Locale

### 9.1 Locale คืออะไร

`Locale` เป็น struct ที่แสดงข้อมูล locale เช่น ภาษา ประเทศ การจัดรูปแบบตัวเลข สกุลเงิน และวันที่

### 9.2 การใช้งาน Locale

```swift
import Foundation

// Locale ปัจจุบันของผู้ใช้
let currentLocale = Locale.current
print(currentLocale.identifier)  // เช่น "th_TH" หรือ "en_US"
print(currentLocale.language.languageCode?.identifier ?? "")  // "th" หรือ "en"
print(currentLocale.region?.identifier ?? "")  // "TH" หรือ "US"

// สกุลเงิน
if let currency = currentLocale.currency {
    print(currency.identifier)  // "THB" หรือ "USD"
    print(currency.symbol)      // "฿" หรือ "$"
}

// Calendar
let calendar = Calendar.current
print(calendar.identifier)  // gregorian, buddhist (ไทยใช้ buddhist)

// Time zone
let timeZone = TimeZone.current
print(timeZone.identifier)  // "Asia/Bangkok"

// สร้าง Locale เฉพาะ
let thaiLocale = Locale(identifier: "th_TH")
let usLocale = Locale(identifier: "en_US")
let japaneseLocale = Locale(identifier: "ja_JP")

// ตรวจสอบ available locales
let availableLocales = Locale.availableIdentifiers
print(availableLocales.count)  // ประมาณ 1000+ locales
```

### 9.3 Locale Identifiers

```swift
// รูปแบบ locale identifier:
// language_REGION
// "th_TH"  - ไทย, ประเทศไทย
// "en_US"  - อังกฤษ, สหรัฐอเมริกา
// "en_GB"  - อังกฤษ, สหราชอาณาจักร
// "zh_Hans_CN" - จีนตัวย่อ, จีน
// "zh_Hant_TW" - จีนตัวเต็ม, ไต้หวัน
// "ar_SA"  - อาหรับ, ซาอุดิอาระเบีย

func demonstrateLocale() {
    let thLocale = Locale(identifier: "th_TH")
    let enLocale = Locale(identifier: "en_US")
    
    let number = 1234567.89
    
    let thFormatter = NumberFormatter()
    thFormatter.locale = thLocale
    thFormatter.numberStyle = .decimal
    print(thFormatter.string(from: NSNumber(value: number)) ?? "")
    // → "1,234,567.89" (ไทยใช้ , เหมือน US)
    
    let deFormatter = NumberFormatter()
    deFormatter.locale = Locale(identifier: "de_DE")
    deFormatter.numberStyle = .decimal
    print(deFormatter.string(from: NSNumber(value: number)) ?? "")
    // → "1.234.567,89" (เยอรมันสลับ , และ .)
}
```

---

## 10. NumberFormatter

### 10.1 รูปแบบตัวเลขตาม Locale

```swift
import Foundation
import SwiftUI

struct NumberFormatterExamples: View {
    let number = 1234567.89
    let currencyAmount = 1250.50
    let percentage = 0.756
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            // 1. Decimal number
            Text("ทศนิยม: \(formatDecimal(number))")
            
            // 2. Currency
            Text("สกุลเงิน: \(formatCurrency(currencyAmount))")
            
            // 3. Percentage
            Text("เปอร์เซ็นต์: \(formatPercent(percentage))")
            
            // 4. Scientific
            Text("Scientific: \(formatScientific(number))")
            
            // 5. Spell out (อ่านเป็นคำ)
            Text("คำ: \(formatSpellOut(42))")
        }
        .padding()
    }
    
    func formatDecimal(_ number: Double) -> String {
        let formatter = NumberFormatter()
        formatter.numberStyle = .decimal
        formatter.locale = Locale.current
        formatter.maximumFractionDigits = 2
        return formatter.string(from: NSNumber(value: number)) ?? "\(number)"
    }
    
    func formatCurrency(_ amount: Double) -> String {
        let formatter = NumberFormatter()
        formatter.numberStyle = .currency
        formatter.locale = Locale.current
        return formatter.string(from: NSNumber(value: amount)) ?? "\(amount)"
    }
    
    func formatPercent(_ value: Double) -> String {
        let formatter = NumberFormatter()
        formatter.numberStyle = .percent
        formatter.locale = Locale.current
        formatter.maximumFractionDigits = 1
        return formatter.string(from: NSNumber(value: value)) ?? "\(value)"
    }
    
    func formatScientific(_ number: Double) -> String {
        let formatter = NumberFormatter()
        formatter.numberStyle = .scientific
        return formatter.string(from: NSNumber(value: number)) ?? "\(number)"
    }
    
    func formatSpellOut(_ number: Int) -> String {
        let formatter = NumberFormatter()
        formatter.numberStyle = .spellOut
        formatter.locale = Locale.current
        return formatter.string(from: NSNumber(value: number)) ?? "\(number)"
    }
}

// Modern API (iOS 15+)
struct ModernNumberFormatting: View {
    var body: some View {
        VStack {
            // Decimal
            Text(1234.56.formatted(.number.precision(.fractionLength(2))))
            
            // Currency
            Text(1250.50.formatted(.currency(code: "THB")))
            
            // Percentage
            Text(0.756.formatted(.percent.precision(.fractionLength(1))))
        }
    }
}
```

---

## 11. DateFormatter

### 11.1 การจัดรูปแบบวันที่ตาม Locale

```swift
import Foundation
import SwiftUI

struct DateFormatterExamples: View {
    let date = Date()
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            Group {
                Text("วันที่สั้น: \(formatDate(date, style: .short))")
                Text("วันที่กลาง: \(formatDate(date, style: .medium))")
                Text("วันที่ยาว: \(formatDate(date, style: .long))")
                Text("วันที่เต็ม: \(formatDate(date, style: .full))")
            }
            
            Divider()
            
            Group {
                Text("เวลาสั้น: \(formatTime(date, style: .short))")
                Text("เวลากลาง: \(formatTime(date, style: .medium))")
                Text("เวลายาว: \(formatTime(date, style: .long))")
            }
            
            Divider()
            
            Group {
                Text("วันที่และเวลา: \(formatDateTime(date))")
                Text("Relative: \(formatRelative(date))")
            }
        }
        .padding()
    }
    
    func formatDate(_ date: Date, style: DateFormatter.Style) -> String {
        let formatter = DateFormatter()
        formatter.dateStyle = style
        formatter.timeStyle = .none
        formatter.locale = Locale.current
        return formatter.string(from: date)
    }
    
    func formatTime(_ date: Date, style: DateFormatter.Style) -> String {
        let formatter = DateFormatter()
        formatter.dateStyle = .none
        formatter.timeStyle = style
        formatter.locale = Locale.current
        return formatter.string(from: date)
    }
    
    func formatDateTime(_ date: Date) -> String {
        let formatter = DateFormatter()
        formatter.dateStyle = .medium
        formatter.timeStyle = .short
        formatter.locale = Locale.current
        return formatter.string(from: date)
    }
    
    func formatRelative(_ date: Date) -> String {
        let formatter = RelativeDateTimeFormatter()
        formatter.locale = Locale.current
        formatter.unitsStyle = .full
        return formatter.localizedString(for: date, relativeTo: Date())
    }
}

// Modern API (iOS 15+)
struct ModernDateFormatting: View {
    let date = Date()
    
    var body: some View {
        VStack {
            // Date only
            Text(date.formatted(date: .abbreviated, time: .omitted))
            
            // Time only
            Text(date.formatted(date: .omitted, time: .shortened))
            
            // Full
            Text(date.formatted())
            
            // Custom format
            Text(date.formatted(.dateTime
                .year()
                .month(.wide)
                .day()
                .hour()
                .minute()
            ))
            
            // Relative
            Text(date.formatted(.relative(presentation: .named)))
        }
    }
}

// Custom date format สำหรับ API
class DateUtility {
    // ISO 8601 สำหรับ API - ไม่ขึ้นกับ locale
    static let iso8601Formatter: DateFormatter = {
        let formatter = DateFormatter()
        formatter.dateFormat = "yyyy-MM-dd'T'HH:mm:ssZ"
        formatter.locale = Locale(identifier: "en_US_POSIX")
        formatter.timeZone = TimeZone(secondsFromGMT: 0)
        return formatter
    }()
    
    // แสดงผลให้ user - ขึ้นกับ locale
    static func displayDate(_ date: Date) -> String {
        let formatter = DateFormatter()
        formatter.dateStyle = .long
        formatter.timeStyle = .none
        formatter.locale = Locale.current
        return formatter.string(from: date)
    }
}
```

---

## 12. MeasurementFormatter

### 12.1 การจัดรูปแบบหน่วยวัด

```swift
import Foundation
import SwiftUI

struct MeasurementFormatterExamples: View {
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            // น้ำหนัก
            WeightView(kilograms: 68.5)
            
            // ระยะทาง
            DistanceView(kilometers: 5.2)
            
            // อุณหภูมิ
            TemperatureView(celsius: 37.0)
            
            // ความเร็ว
            SpeedView(metersPerSecond: 13.9)
        }
        .padding()
    }
}

struct WeightView: View {
    let kilograms: Double
    
    var body: some View {
        let weight = Measurement(value: kilograms, unit: UnitMass.kilograms)
        let formatter = MeasurementFormatter()
        formatter.unitOptions = .providedUnit
        formatter.locale = Locale.current
        return Text("น้ำหนัก: \(formatter.string(from: weight))")
    }
}

struct DistanceView: View {
    let kilometers: Double
    
    var body: some View {
        let distance = Measurement(value: kilometers, unit: UnitLength.kilometers)
        let formatter = MeasurementFormatter()
        formatter.unitStyle = .long
        formatter.locale = Locale.current
        // จะแปลงเป็น miles อัตโนมัติสำหรับ US locale
        return Text("ระยะทาง: \(formatter.string(from: distance))")
    }
}

struct TemperatureView: View {
    let celsius: Double
    
    var body: some View {
        let temp = Measurement(value: celsius, unit: UnitTemperature.celsius)
        let formatter = MeasurementFormatter()
        formatter.unitStyle = .medium
        formatter.locale = Locale.current
        // จะแสดง °C หรือ °F ตาม locale
        return Text("อุณหภูมิ: \(formatter.string(from: temp))")
    }
}

struct SpeedView: View {
    let metersPerSecond: Double
    
    var body: some View {
        let speed = Measurement(value: metersPerSecond, unit: UnitSpeed.metersPerSecond)
        let formatter = MeasurementFormatter()
        formatter.unitStyle = .abbreviated
        formatter.locale = Locale.current
        return Text("ความเร็ว: \(formatter.string(from: speed))")
    }
}
```

---

## 13. RTL (Right-to-Left) Language Support

### 13.1 ภาษา RTL

ภาษา RTL (Right-to-Left) เช่น อาหรับ (ar), ฮีบรู (he), เปอร์เซีย (fa) เขียนจากขวาไปซ้าย ทำให้ layout ต้องกลับด้านด้วย

### 13.2 SwiftUI รองรับ RTL อัตโนมัติ

```swift
import SwiftUI

struct RTLSupportExample: View {
    var body: some View {
        VStack {
            // SwiftUI กลับด้าน layout อัตโนมัติสำหรับ RTL
            HStack {
                Image(systemName: "person.fill")
                Text("ชื่อผู้ใช้")
                Spacer()
                Text(">")
            }
            // RTL: "> ชื่อผู้ใช้  [icon]"
            // LTR: "[icon] ชื่อผู้ใช้  >"
            
            // leading/trailing จะกลับด้านอัตโนมัติ
            Text("ข้อความ")
                .frame(maxWidth: .infinity, alignment: .leading)
            // RTL: จะ align ขวา
            // LTR: จะ align ซ้าย
        }
    }
}

struct RTLAwareLayout: View {
    @Environment(\.layoutDirection) var layoutDirection
    
    var body: some View {
        VStack {
            if layoutDirection == .rightToLeft {
                Text("RTL Layout")
                // custom RTL layout
            } else {
                Text("LTR Layout")
                // standard layout
            }
            
            // FlipIfRTL modifier
            HStack {
                Image(systemName: "arrow.right")
                    .flipsForRightToLeftLayoutDirection(true)
                Text("ถัดไป")
            }
        }
    }
}
```

### 13.3 สิ่งที่ต้องระวังสำหรับ RTL

```swift
struct RTLConsiderations: View {
    var body: some View {
        VStack {
            // ✅ ดี: ใช้ leading/trailing แทน left/right
            Text("ข้อความ")
                .padding(.leading) // จะเป็น left ใน LTR, right ใน RTL
                .padding(.trailing)
            
            // ❌ ไม่ดี: ใช้ left/right ตายตัว
            Text("ข้อความ")
                .padding(.horizontal) // OK เสมอ
            
            // ✅ ดี: Stack items จะ flip อัตโนมัติ
            HStack {
                Image(systemName: "chevron.backward")
                    .flipsForRightToLeftLayoutDirection(true)
                Text("กลับ")
            }
            
            // ✅ ดี: ไม่ hard-code direction ของ arrow
            Image(systemName: "arrow.right")
                .flipsForRightToLeftLayoutDirection(true)
            
            // ❌ ไม่ดี: Hard-code direction
            Image(systemName: "arrow.right") // ไม่ flip ใน RTL
        }
    }
}

// UIKit RTL Support
class RTLViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let label = UILabel()
        
        // ✅ ดี: Natural text alignment
        label.textAlignment = .natural  // จะ align ตาม locale
        
        // ❌ ไม่ดี: Hard-coded alignment
        label.textAlignment = .left
        
        // ✅ ดี: Semantic content attribute
        let stackView = UIStackView()
        stackView.semanticContentAttribute = .unspecified // กลับอัตโนมัติ
        
        // ตรวจสอบ direction
        let direction = UIView.userInterfaceLayoutDirection(for: stackView.semanticContentAttribute)
        if direction == .rightToLeft {
            // ปรับ layout สำหรับ RTL
        }
    }
}
```

---

## 14. Auto Layout สำหรับ Localization

### 14.1 ปัญหา Layout กับข้อความที่แปลแล้ว

```swift
struct LayoutForLocalizationExample: View {
    var body: some View {
        VStack {
            // ✅ ดี: ยืดหดตามข้อความ
            Button("บันทึก") {}
            // German: "Speichern" - ยาวกว่า
            // Thai: "บันทึก" - สั้นกว่า
            
            // ✅ ดี: ใช้ multiline เมื่อจำเป็น
            Text("ข้อความที่อาจยาวมากเมื่อแปลเป็นภาษาอื่น")
                .multilineTextAlignment(.center)
                .lineLimit(nil) // ไม่จำกัดจำนวนบรรทัด
                .minimumScaleFactor(0.7) // ย่อ font ได้ถึง 70%
            
            // ✅ ดี: Flexible containers
            HStack {
                Text("ยอดรวม:")
                    .layoutPriority(1) // คงรูปแบบนี้ไว้ก่อน
                Spacer()
                Text("฿1,234.56")
            }
        }
    }
}

struct AdaptiveFormLayout: View {
    @State private var username = ""
    @State private var password = ""
    
    var body: some View {
        // Form ที่ปรับได้ตามภาษา
        VStack(alignment: .leading, spacing: 16) {
            // Labels ที่ยาวได้เต็มที่
            Group {
                VStack(alignment: .leading, spacing: 4) {
                    Text("ชื่อผู้ใช้งาน")
                        .font(.caption)
                        .foregroundColor(.secondary)
                    TextField("กรอกชื่อผู้ใช้งาน", text: $username)
                        .textFieldStyle(.roundedBorder)
                }
                
                VStack(alignment: .leading, spacing: 4) {
                    Text("รหัสผ่าน")
                        .font(.caption)
                        .foregroundColor(.secondary)
                    SecureField("กรอกรหัสผ่าน", text: $password)
                        .textFieldStyle(.roundedBorder)
                }
            }
            
            // ปุ่มที่ขยายเต็ม width
            Button("เข้าสู่ระบบ") {}
                .frame(maxWidth: .infinity)
                .padding()
                .background(Color.blue)
                .foregroundColor(.white)
                .cornerRadius(10)
        }
        .padding()
    }
}
```

---

## 15. Localizing Images และ Assets

### 15.1 การ Localize Images

```swift
// Asset Catalog ใน Xcode รองรับ localization สำหรับ images

// โครงสร้างใน Assets.xcassets:
// Images/
// ├── welcome_image (universal)
// ├── welcome_image (th)      ← เพิ่มใน Xcode โดยคลิก "+" ใน Localization
// └── welcome_image (en)

// การใช้งาน - ไม่ต้องเปลี่ยน code
// Xcode จะเลือก image ที่ถูก locale อัตโนมัติ
struct LocalizedImageExample: View {
    var body: some View {
        VStack {
            // รูปที่ localize ใน Asset Catalog
            Image("welcome_image")
                .resizable()
                .aspectRatio(contentMode: .fit)
            
            // รูปที่ต้องแสดง text ที่แตกต่างกันตามภาษา
            LocalizedScreenshot()
        }
    }
}

// สำหรับรูปที่สร้างด้วย code
struct LocalizedScreenshot: View {
    var screenshotName: String {
        let languageCode = Locale.current.language.languageCode?.identifier ?? "en"
        return "screenshot_\(languageCode)"
    }
    
    var body: some View {
        Image(screenshotName)
            .resizable()
            .aspectRatio(contentMode: .fit)
    }
}
```

### 15.2 Localizing Assets อื่นๆ

```swift
// Localizable.strings สำหรับ Asset names
// "app_icon_name" = "AppIconThai"; // ใน th.lproj/Localizable.strings
// "app_icon_name" = "AppIconEnglish"; // ใน en.lproj/Localizable.strings

// Colors ตาม locale
extension Color {
    static var localizedBrand: Color {
        let languageCode = Locale.current.language.languageCode?.identifier ?? "en"
        switch languageCode {
        case "zh": return Color("BrandColorChina")
        case "ja": return Color("BrandColorJapan")
        default: return Color("BrandColor")
        }
    }
}
```

---

## 16. Localizing App Name และ Info.plist

### 16.1 Localize App Name

สร้างไฟล์ `InfoPlist.strings` ใน lproj folder:

**th.lproj/InfoPlist.strings:**
```
"CFBundleDisplayName" = "แอปช็อปปิ้ง";
"CFBundleName" = "แอปช็อปปิ้ง";
"NSCameraUsageDescription" = "แอปต้องการเข้าถึงกล้องเพื่อสแกน QR Code";
"NSPhotoLibraryUsageDescription" = "แอปต้องการเข้าถึงรูปภาพเพื่ออัปโหลดรูปโปรไฟล์";
"NSLocationWhenInUseUsageDescription" = "แอปต้องการทราบตำแหน่งของคุณเพื่อหาร้านค้าใกล้เคียง";
"NSMicrophoneUsageDescription" = "แอปต้องการเข้าถึงไมค์เพื่อค้นหาด้วยเสียง";
```

**en.lproj/InfoPlist.strings:**
```
"CFBundleDisplayName" = "Shopping App";
"CFBundleName" = "ShoppingApp";
"NSCameraUsageDescription" = "The app needs camera access to scan QR codes";
"NSPhotoLibraryUsageDescription" = "The app needs photo library access to upload profile photos";
"NSLocationWhenInUseUsageDescription" = "The app needs your location to find nearby stores";
"NSMicrophoneUsageDescription" = "The app needs microphone access for voice search";
```

### 16.2 Localizing Info.plist Keys อื่นๆ

```swift
// ใน code - อ่านค่าที่ localize แล้ว
class AppInfoHelper {
    static var displayName: String {
        Bundle.main.localizedInfoDictionary?["CFBundleDisplayName"] as? String
        ?? Bundle.main.infoDictionary?["CFBundleDisplayName"] as? String
        ?? "App"
    }
    
    static var cameraPermissionMessage: String {
        Bundle.main.localizedInfoDictionary?["NSCameraUsageDescription"] as? String
        ?? "This app needs camera access"
    }
}
```

---

## 17. Xcode Export/Import สำหรับ XLIFF

### 17.1 XLIFF คืออะไร

XLIFF (XML Localization Interchange File Format) เป็นรูปแบบมาตรฐานสำหรับการแลกเปลี่ยน localization data ใช้ส่งให้นักแปลและนำกลับเข้า Xcode

### 17.2 Export XLIFF

```
1. เปิด Xcode project
2. ไปที่ Product > Export Localizations...
3. เลือก folder ที่จะ export
4. เลือกภาษาที่ต้องการ export
5. คลิก Export
```

ผลลัพธ์: ไฟล์ `.xcloc` ที่มี `.xliff` ข้างใน

### 17.3 ตัวอย่าง XLIFF

```xml
<?xml version="1.0" encoding="UTF-8"?>
<xliff xmlns="urn:oasis:names:tc:xliff:document:1.2" 
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
       version="1.2">
  <file original="MyApp/Localizable.strings" 
        source-language="en" 
        target-language="th" 
        datatype="plaintext">
    <header>
      <tool tool-id="com.apple.dt.xcode" tool-name="Xcode" tool-version="15.0"/>
    </header>
    <body>
      <trans-unit id="app_title">
        <source>Shopping App</source>
        <target>แอปช็อปปิ้ง</target>
        <note>Title of the app</note>
      </trans-unit>
      <trans-unit id="button_save">
        <source>Save</source>
        <target>บันทึก</target>
        <note>Save button label</note>
      </trans-unit>
    </body>
  </file>
</xliff>
```

### 17.4 Import XLIFF

```
1. ได้รับไฟล์ .xcloc จากนักแปล
2. ไปที่ Product > Import Localizations...
3. เลือกไฟล์ .xcloc
4. คลิก Import
```

### 17.5 Automation ด้วย xcodebuild

```bash
# Export localizations
xcodebuild -exportLocalizations \
  -localizationPath ./Localizations \
  -project MyApp.xcodeproj \
  -exportLanguage th \
  -exportLanguage ja

# Import localizations
xcodebuild -importLocalizations \
  -localizationPath ./Localizations/th.xcloc \
  -project MyApp.xcodeproj
```

---

## 18. Testing Localization

### 18.1 ทดสอบด้วย Scheme

```
1. Edit Scheme > Run > Options
2. Application Language: เลือกภาษาที่ต้องการทดสอบ
3. Application Region: เลือก region
```

### 18.2 ทดสอบด้วย Launch Arguments

```swift
// เพิ่ม launch argument ใน scheme
// -AppleLanguages (th) -AppleLocale th_TH

// หรือใน code (สำหรับ testing)
class LocalizationTestHelper {
    static func setLanguage(_ language: String) {
        UserDefaults.standard.set([language], forKey: "AppleLanguages")
        UserDefaults.standard.synchronize()
    }
    
    static func resetLanguage() {
        UserDefaults.standard.removeObject(forKey: "AppleLanguages")
    }
}
```

### 18.3 Unit Tests สำหรับ Localization

```swift
import XCTest

class LocalizationTests: XCTestCase {
    
    func testEnglishStrings() {
        let bundle = Bundle(for: LocalizationTests.self)
        
        // ทดสอบว่า strings ถูกต้อง
        let enBundle = Bundle.bundle(for: "en", in: bundle)
        let title = NSLocalizedString("app_title", bundle: enBundle, comment: "")
        XCTAssertEqual(title, "Shopping App")
    }
    
    func testThaiStrings() {
        let bundle = Bundle(for: LocalizationTests.self)
        
        let thBundle = Bundle.bundle(for: "th", in: bundle)
        let title = NSLocalizedString("app_title", bundle: thBundle, comment: "")
        XCTAssertEqual(title, "แอปช็อปปิ้ง")
    }
    
    func testAllRequiredKeysExist() {
        let requiredKeys = [
            "app_title",
            "button_save",
            "button_cancel",
            "error_network"
        ]
        
        let languages = ["en", "th"]
        
        for language in languages {
            for key in requiredKeys {
                let value = localizedString(key, language: language)
                XCTAssertNotEqual(
                    value, key,
                    "Key '\(key)' not found in '\(language)' strings"
                )
            }
        }
    }
    
    func testNoHardCodedStrings() {
        // ตรวจสอบว่า source code ไม่มี hard-coded strings (ตัวอย่าง simple check)
        // ในความเป็นจริงควรใช้ SwiftLint custom rule
    }
    
    private func localizedString(_ key: String, language: String) -> String {
        guard let path = Bundle.main.path(forResource: "Localizable", 
                                          ofType: "strings", 
                                          inDirectory: nil, 
                                          forLocalization: language),
              let bundle = Bundle(path: path) else {
            return key
        }
        return NSLocalizedString(key, bundle: bundle, comment: "")
    }
}

extension Bundle {
    static func bundle(for language: String, in bundle: Bundle) -> Bundle {
        guard let path = bundle.path(forResource: language, ofType: "lproj"),
              let languageBundle = Bundle(path: path) else {
            return bundle
        }
        return languageBundle
    }
}
```

### 18.4 UI Tests สำหรับ Localization

```swift
import XCTest

class LocalizationUITests: XCTestCase {
    let app = XCUIApplication()
    
    override func setUpWithError() throws {
        continueAfterFailure = false
    }
    
    func testThaiLocalization() {
        app.launchArguments = ["-AppleLanguages", "(th)", "-AppleLocale", "th_TH"]
        app.launch()
        
        // ตรวจสอบว่าข้อความภาษาไทยแสดงถูกต้อง
        XCTAssertTrue(app.staticTexts["แอปช็อปปิ้ง"].exists)
        XCTAssertTrue(app.buttons["บันทึก"].exists || 
                      app.buttons["เพิ่ม"].exists)
    }
    
    func testEnglishLocalization() {
        app.launchArguments = ["-AppleLanguages", "(en)", "-AppleLocale", "en_US"]
        app.launch()
        
        XCTAssertTrue(app.staticTexts["Shopping App"].exists)
    }
    
    func testRTLLayout() {
        // ทดสอบภาษา RTL
        app.launchArguments = ["-AppleLanguages", "(ar)", "-AppleLocale", "ar_SA"]
        app.launch()
        
        // ตรวจสอบว่า layout กลับด้านถูกต้อง
        let backButton = app.buttons.firstMatch
        let frame = backButton.frame
        // ปุ่มย้อนกลับควรอยู่ทางขวาในภาษา Arabic
        XCTAssertGreaterThan(frame.minX, app.frame.width / 2)
    }
}
```

---

## 19. SwiftUI Localization

### 19.1 SwiftUI Localization Features

```swift
import SwiftUI

// SwiftUI รองรับ localization โดยธรรมชาติ
struct SwiftUILocalizationExample: View {
    @State private var name = "สมชาย"
    @State private var itemCount = 5
    
    var body: some View {
        VStack(spacing: 16) {
            // 1. Static string - ค้นหาใน Localizable.strings อัตโนมัติ
            Text("app_title")
            
            // 2. String interpolation - ระวัง! นี่คือ LocalizedStringKey
            Text("greeting_\(name)")
            // ใน Localizable.strings: "greeting_%@" = "สวัสดี, %@!";
            
            // 3. Formatted value
            Text("\(itemCount) รายการ")
            
            // 4. Button label
            Button("button_save") {}
            
            // 5. Label พร้อม image
            Label("nav_home", systemImage: "house")
            
            // 6. Picker
            Picker("เลือกภาษา", selection: .constant("th")) {
                Text("ภาษาไทย").tag("th")
                Text("English").tag("en")
            }
            
            // 7. Alert ที่ localize
            AlertDemo()
        }
    }
}

struct AlertDemo: View {
    @State private var showAlert = false
    
    var body: some View {
        Button("แสดง Alert") {
            showAlert = true
        }
        .alert("alert_delete_title", isPresented: $showAlert) {
            Button("button_delete", role: .destructive) {}
            Button("button_cancel", role: .cancel) {}
        } message: {
            Text("alert_delete_message")
        }
    }
}

// Localizable.strings สำหรับตัวอย่างข้างต้น:
// "app_title" = "แอปของฉัน";
// "button_save" = "บันทึก";
// "nav_home" = "หน้าหลัก";
// "alert_delete_title" = "ยืนยันการลบ";
// "alert_delete_message" = "คุณแน่ใจหรือไม่ว่าต้องการลบรายการนี้?";
// "button_delete" = "ลบ";
// "button_cancel" = "ยกเลิก";
```

### 19.2 Custom Locale ใน Preview

```swift
struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        Group {
            // ทดสอบภาษาไทย
            ContentView()
                .environment(\.locale, Locale(identifier: "th_TH"))
                .previewDisplayName("Thai")
            
            // ทดสอบภาษาอังกฤษ
            ContentView()
                .environment(\.locale, Locale(identifier: "en_US"))
                .previewDisplayName("English")
            
            // ทดสอบภาษา Arabic (RTL)
            ContentView()
                .environment(\.locale, Locale(identifier: "ar_SA"))
                .environment(\.layoutDirection, .rightToLeft)
                .previewDisplayName("Arabic RTL")
        }
    }
}
```

---

## 20. String Catalogs (.xcstrings) - Xcode 15+

### 20.1 String Catalogs คืออะไร

String Catalogs เป็นฟีเจอร์ใหม่ใน Xcode 15 ที่รวม Localizable.strings และ Localizable.stringsdict เป็นไฟล์ JSON เดียว ชื่อ `Localizable.xcstrings`

### 20.2 ข้อดีของ String Catalogs

1. **ไฟล์เดียว** แทนหลายไฟล์ต่อภาษา
2. **เห็น translation state** ได้ชัดเจน (translated, needs review, etc.)
3. **รองรับ plural rules** อัตโนมัติ
4. **Conflict resolution** ง่ายกว่าด้วย JSON format
5. **Xcode ช่วย extract strings** จาก source code

### 20.3 โครงสร้างของ .xcstrings

```json
{
  "sourceLanguage" : "en",
  "strings" : {
    "app_title" : {
      "comment" : "Title shown in navigation bar",
      "localizations" : {
        "en" : {
          "stringUnit" : {
            "state" : "translated",
            "value" : "Shopping App"
          }
        },
        "th" : {
          "stringUnit" : {
            "state" : "translated",
            "value" : "แอปช็อปปิ้ง"
          }
        }
      }
    },
    "cart_item_count" : {
      "localizations" : {
        "en" : {
          "variations" : {
            "plural" : {
              "zero" : {
                "stringUnit" : {
                  "state" : "translated",
                  "value" : "No items in cart"
                }
              },
              "one" : {
                "stringUnit" : {
                  "state" : "translated",
                  "value" : "%d item in cart"
                }
              },
              "other" : {
                "stringUnit" : {
                  "state" : "translated",
                  "value" : "%d items in cart"
                }
              }
            }
          }
        },
        "th" : {
          "variations" : {
            "plural" : {
              "other" : {
                "stringUnit" : {
                  "state" : "translated",
                  "value" : "%d ชิ้นในตะกร้า"
                }
              }
            }
          }
        }
      }
    }
  },
  "version" : "1.0"
}
```

### 20.4 การใช้งาน String Catalog ใน Code

```swift
// ใช้งานเหมือนเดิม - ไม่ต้องเปลี่ยน code
struct StringCatalogExample: View {
    let itemCount = 3
    
    var body: some View {
        VStack {
            // Text ค้นหาใน .xcstrings อัตโนมัติ
            Text("app_title")
            
            // NSLocalizedString ทำงานกับ .xcstrings
            Text(NSLocalizedString("button_save", comment: ""))
            
            // String(localized:) ใหม่ทำงานกับ .xcstrings
            Text(String(localized: "greeting_message"))
            
            // Plural จาก .xcstrings
            Text(String(format: NSLocalizedString("cart_item_count", comment: ""), itemCount))
        }
    }
}
```

### 20.5 Migrate จาก .strings ไป .xcstrings

```
1. เปิด Editor > Convert > To String Catalog...
2. Xcode จะแปลง .strings และ .stringsdict ทั้งหมดเป็น .xcstrings
3. ตรวจสอบว่า translation ครบถ้วน
4. ลบไฟล์ .strings เดิม
```

---

## 21. แบบฝึกหัดพร้อมเฉลย

### แบบฝึกหัดที่ 1: เพิ่ม Localization ให้ Profile Screen

**โจทย์:** สร้าง Profile Screen พร้อม localization ทั้งภาษาไทยและอังกฤษ

**เฉลย:**

**Localizable.strings (en):**
```
"profile_title" = "My Profile";
"profile_name_label" = "Name";
"profile_email_label" = "Email";
"profile_phone_label" = "Phone";
"profile_member_since" = "Member since %@";
"profile_edit_button" = "Edit Profile";
"profile_logout_button" = "Log Out";
"profile_logout_confirm_title" = "Log Out?";
"profile_logout_confirm_message" = "Are you sure you want to log out?";
"profile_logout_confirm_button" = "Log Out";
"button_cancel" = "Cancel";
```

**Localizable.strings (th):**
```
"profile_title" = "โปรไฟล์ของฉัน";
"profile_name_label" = "ชื่อ";
"profile_email_label" = "อีเมล";
"profile_phone_label" = "โทรศัพท์";
"profile_member_since" = "สมาชิกตั้งแต่ %@";
"profile_edit_button" = "แก้ไขโปรไฟล์";
"profile_logout_button" = "ออกจากระบบ";
"profile_logout_confirm_title" = "ออกจากระบบ?";
"profile_logout_confirm_message" = "คุณแน่ใจหรือไม่ว่าต้องการออกจากระบบ?";
"profile_logout_confirm_button" = "ออกจากระบบ";
"button_cancel" = "ยกเลิก";
```

```swift
import SwiftUI

struct ProfileScreen: View {
    let user = UserInfo(
        name: "สมชาย ใจดี",
        email: "somchai@example.com",
        phone: "081-234-5678",
        memberSince: Date(timeIntervalSince1970: 1640000000)
    )
    
    @State private var showingLogoutAlert = false
    
    var body: some View {
        NavigationView {
            List {
                // Profile Header
                Section {
                    HStack {
                        Circle()
                            .fill(Color.blue)
                            .frame(width: 70, height: 70)
                            .overlay(
                                Text(user.initials)
                                    .foregroundColor(.white)
                                    .font(.title2)
                            )
                        
                        VStack(alignment: .leading, spacing: 4) {
                            Text(user.name)
                                .font(.headline)
                            Text(memberSinceText)
                                .font(.caption)
                                .foregroundColor(.secondary)
                        }
                    }
                    .padding(.vertical, 8)
                }
                
                // Contact Information
                Section {
                    ProfileRow(
                        label: NSLocalizedString("profile_name_label", comment: ""),
                        value: user.name,
                        icon: "person.fill"
                    )
                    
                    ProfileRow(
                        label: NSLocalizedString("profile_email_label", comment: ""),
                        value: user.email,
                        icon: "envelope.fill"
                    )
                    
                    ProfileRow(
                        label: NSLocalizedString("profile_phone_label", comment: ""),
                        value: formattedPhone,
                        icon: "phone.fill"
                    )
                }
                
                // Actions
                Section {
                    Button("profile_edit_button") {}
                        .foregroundColor(.blue)
                    
                    Button("profile_logout_button") {
                        showingLogoutAlert = true
                    }
                    .foregroundColor(.red)
                }
            }
            .navigationTitle("profile_title")
            .alert("profile_logout_confirm_title", isPresented: $showingLogoutAlert) {
                Button("profile_logout_confirm_button", role: .destructive) {
                    logout()
                }
                Button("button_cancel", role: .cancel) {}
            } message: {
                Text("profile_logout_confirm_message")
            }
        }
    }
    
    var memberSinceText: String {
        let formatter = DateFormatter()
        formatter.dateStyle = .medium
        formatter.locale = Locale.current
        let dateString = formatter.string(from: user.memberSince)
        return String(format: NSLocalizedString("profile_member_since", comment: ""), dateString)
    }
    
    var formattedPhone: String {
        // ปรับรูปแบบโทรศัพท์ตาม locale
        return user.phone
    }
    
    func logout() {
        // Logout logic
    }
}

struct ProfileRow: View {
    let label: String
    let value: String
    let icon: String
    
    var body: some View {
        HStack {
            Image(systemName: icon)
                .frame(width: 24)
                .foregroundColor(.secondary)
            
            VStack(alignment: .leading, spacing: 2) {
                Text(label)
                    .font(.caption)
                    .foregroundColor(.secondary)
                Text(value)
                    .font(.body)
            }
        }
        .padding(.vertical, 2)
    }
}

struct UserInfo {
    let name: String
    let email: String
    let phone: String
    let memberSince: Date
    
    var initials: String {
        let components = name.components(separatedBy: " ")
        let firstLetters = components.prefix(2).compactMap { $0.first }
        return String(firstLetters)
    }
}
```

### แบบฝึกหัดที่ 2: สร้างแอปที่รองรับทั้งไทยและอังกฤษ

**โจทย์:** สร้างแอปง่ายๆ ที่แสดงรายการสินค้าพร้อม localization ครบถ้วน

**เฉลย:**

```swift
import SwiftUI

// MARK: - Models
struct Product: Identifiable {
    let id: UUID
    let nameKey: String
    let descriptionKey: String
    let price: Decimal
    let category: String
    
    var localizedName: String {
        NSLocalizedString(nameKey, comment: "Product name")
    }
    
    var localizedDescription: String {
        NSLocalizedString(descriptionKey, comment: "Product description")
    }
    
    var formattedPrice: String {
        let formatter = NumberFormatter()
        formatter.numberStyle = .currency
        formatter.locale = Locale.current
        return formatter.string(from: price as NSDecimalNumber) ?? "\(price)"
    }
}

// MARK: - Localizable Strings Files

// en.lproj/Localizable.strings:
// "product_list_title" = "Products";
// "product_search_placeholder" = "Search products";
// "product_detail_title" = "Product Details";
// "product_add_to_cart" = "Add to Cart";
// "product_price_label" = "Price";
// "product_category_label" = "Category";
// "cart_button_title" = "Cart (%d)";
// "product_shirt_name" = "T-Shirt";
// "product_shirt_desc" = "Comfortable cotton t-shirt";
// "product_shoes_name" = "Running Shoes";
// "product_shoes_desc" = "Lightweight running shoes for daily use";

// th.lproj/Localizable.strings:
// "product_list_title" = "สินค้า";
// "product_search_placeholder" = "ค้นหาสินค้า";
// "product_detail_title" = "รายละเอียดสินค้า";
// "product_add_to_cart" = "เพิ่มในตะกร้า";
// "product_price_label" = "ราคา";
// "product_category_label" = "หมวดหมู่";
// "cart_button_title" = "ตะกร้า (%d)";
// "product_shirt_name" = "เสื้อยืด";
// "product_shirt_desc" = "เสื้อยืดผ้าคอตตอนที่สวมใส่สบาย";
// "product_shoes_name" = "รองเท้าวิ่ง";
// "product_shoes_desc" = "รองเท้าวิ่งน้ำหนักเบาสำหรับใช้งานประจำวัน";

// MARK: - Views
struct LocalizedProductListView: View {
    let products = [
        Product(
            id: UUID(),
            nameKey: "product_shirt_name",
            descriptionKey: "product_shirt_desc",
            price: 299,
            category: "เสื้อผ้า"
        ),
        Product(
            id: UUID(),
            nameKey: "product_shoes_name",
            descriptionKey: "product_shoes_desc",
            price: 1490,
            category: "รองเท้า"
        )
    ]
    
    @State private var searchText = ""
    @State private var cartCount = 0
    
    var filteredProducts: [Product] {
        if searchText.isEmpty { return products }
        return products.filter { $0.localizedName.localizedCaseInsensitiveContains(searchText) }
    }
    
    var cartButtonTitle: String {
        String(format: NSLocalizedString("cart_button_title", comment: ""), cartCount)
    }
    
    var body: some View {
        NavigationView {
            List(filteredProducts) { product in
                NavigationLink(destination: LocalizedProductDetailView(
                    product: product,
                    onAddToCart: { cartCount += 1 }
                )) {
                    ProductRowView(product: product)
                }
            }
            .searchable(text: $searchText, 
                       prompt: Text("product_search_placeholder"))
            .navigationTitle(Text("product_list_title"))
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button(cartButtonTitle) {}
                }
            }
        }
    }
}

struct ProductRowView: View {
    let product: Product
    
    var body: some View {
        HStack {
            Image(systemName: "tag.fill")
                .foregroundColor(.blue)
                .frame(width: 40, height: 40)
            
            VStack(alignment: .leading, spacing: 4) {
                Text(product.localizedName)
                    .font(.headline)
                Text(product.localizedDescription)
                    .font(.caption)
                    .foregroundColor(.secondary)
                    .lineLimit(2)
            }
            
            Spacer()
            
            Text(product.formattedPrice)
                .font(.subheadline)
                .bold()
                .foregroundColor(.blue)
        }
        .padding(.vertical, 4)
    }
}

struct LocalizedProductDetailView: View {
    let product: Product
    let onAddToCart: () -> Void
    
    var body: some View {
        ScrollView {
            VStack(alignment: .leading, spacing: 16) {
                // Product Image (placeholder)
                Rectangle()
                    .fill(Color.blue.opacity(0.1))
                    .frame(maxWidth: .infinity)
                    .frame(height: 250)
                    .overlay(
                        Image(systemName: "photo")
                            .font(.system(size: 60))
                            .foregroundColor(.blue.opacity(0.5))
                    )
                
                VStack(alignment: .leading, spacing: 12) {
                    Text(product.localizedName)
                        .font(.title)
                        .bold()
                    
                    HStack {
                        Text("product_price_label")
                            .font(.subheadline)
                            .foregroundColor(.secondary)
                        Spacer()
                        Text(product.formattedPrice)
                            .font(.title2)
                            .bold()
                            .foregroundColor(.blue)
                    }
                    
                    HStack {
                        Text("product_category_label")
                            .font(.subheadline)
                            .foregroundColor(.secondary)
                        Spacer()
                        Text(product.category)
                            .font(.subheadline)
                    }
                    
                    Divider()
                    
                    Text(product.localizedDescription)
                        .font(.body)
                        .lineSpacing(4)
                }
                .padding()
                
                Button(action: onAddToCart) {
                    Text("product_add_to_cart")
                        .frame(maxWidth: .infinity)
                        .padding()
                        .background(Color.blue)
                        .foregroundColor(.white)
                        .cornerRadius(12)
                }
                .padding()
            }
        }
        .navigationTitle("product_detail_title")
        .navigationBarTitleDisplayMode(.inline)
    }
}

// MARK: - Previews
struct LocalizedProductListView_Previews: PreviewProvider {
    static var previews: some View {
        Group {
            LocalizedProductListView()
                .environment(\.locale, Locale(identifier: "th_TH"))
                .previewDisplayName("Thai")
            
            LocalizedProductListView()
                .environment(\.locale, Locale(identifier: "en_US"))
                .previewDisplayName("English")
        }
    }
}
```

---

## 22. การ Localize แอปเป็นภาษาไทยและอังกฤษ (ตัวอย่างครบถ้วน)

### 22.1 โครงสร้างไฟล์ทั้งหมด

```
MyApp/
├── App.swift
├── ContentView.swift
├── en.lproj/
│   ├── Localizable.strings
│   ├── Localizable.stringsdict
│   └── InfoPlist.strings
├── th.lproj/
│   ├── Localizable.strings
│   ├── Localizable.stringsdict
│   └── InfoPlist.strings
└── Assets.xcassets/
    ├── AppIcon.appiconset/
    ├── welcome_banner.imageset/
    │   ├── welcome_banner@2x.png (universal)
    │   ├── welcome_banner_th@2x.png
    │   └── Contents.json
    └── ...
```

### 22.2 ตัวอย่าง Complete App

```swift
// MARK: - App Entry Point
import SwiftUI

@main
struct BilingualApp: App {
    var body: some Scene {
        WindowGroup {
            MainTabView()
        }
    }
}

// MARK: - Tab View
struct MainTabView: View {
    var body: some View {
        TabView {
            HomeView()
                .tabItem {
                    Label("nav_home", systemImage: "house.fill")
                }
            
            SearchView()
                .tabItem {
                    Label("nav_search", systemImage: "magnifyingglass")
                }
            
            ProfileView()
                .tabItem {
                    Label("nav_profile", systemImage: "person.fill")
                }
            
            SettingsView()
                .tabItem {
                    Label("nav_settings", systemImage: "gear")
                }
        }
    }
}

// MARK: - Home View
struct HomeView: View {
    @State private var currentDate = Date()
    
    var greeting: String {
        let hour = Calendar.current.component(.hour, from: currentDate)
        switch hour {
        case 5..<12:
            return NSLocalizedString("greeting_morning", comment: "")
        case 12..<17:
            return NSLocalizedString("greeting_afternoon", comment: "")
        case 17..<21:
            return NSLocalizedString("greeting_evening", comment: "")
        default:
            return NSLocalizedString("greeting_night", comment: "")
        }
    }
    
    var formattedDate: String {
        let formatter = DateFormatter()
        formatter.dateStyle = .full
        formatter.locale = Locale.current
        return formatter.string(from: currentDate)
    }
    
    var body: some View {
        NavigationView {
            ScrollView {
                VStack(alignment: .leading, spacing: 20) {
                    // Greeting
                    VStack(alignment: .leading, spacing: 4) {
                        Text(greeting)
                            .font(.largeTitle)
                            .bold()
                        Text(formattedDate)
                            .font(.subheadline)
                            .foregroundColor(.secondary)
                    }
                    .padding()
                    
                    // Stats
                    LazyVGrid(columns: [GridItem(.flexible()), GridItem(.flexible())], spacing: 16) {
                        StatCard(
                            title: "home_total_orders",
                            value: 42.formatted(),
                            icon: "bag.fill",
                            color: .blue
                        )
                        
                        StatCard(
                            title: "home_saved_amount",
                            value: 1250.50.formatted(.currency(code: Locale.current.currency?.identifier ?? "THB")),
                            icon: "creditcard.fill",
                            color: .green
                        )
                    }
                    .padding(.horizontal)
                }
            }
            .navigationTitle("nav_home")
        }
    }
}

struct StatCard: View {
    let title: LocalizedStringKey
    let value: String
    let icon: String
    let color: Color
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            Image(systemName: icon)
                .font(.title2)
                .foregroundColor(color)
            
            Text(value)
                .font(.title2)
                .bold()
            
            Text(title)
                .font(.caption)
                .foregroundColor(.secondary)
        }
        .padding()
        .frame(maxWidth: .infinity, alignment: .leading)
        .background(Color(.secondarySystemBackground))
        .cornerRadius(12)
    }
}

// MARK: - Settings View
struct SettingsView: View {
    @AppStorage("selectedLanguage") private var selectedLanguage = "auto"
    @AppStorage("notificationsEnabled") private var notificationsEnabled = true
    @AppStorage("darkModeEnabled") private var darkModeEnabled = false
    
    var body: some View {
        NavigationView {
            Form {
                // Language Section
                Section(header: Text("settings_section_language")) {
                    Picker("settings_language", selection: $selectedLanguage) {
                        Text("settings_language_auto").tag("auto")
                        Text("ภาษาไทย").tag("th")
                        Text("English").tag("en")
                    }
                }
                
                // Notifications Section
                Section(header: Text("settings_section_notifications")) {
                    Toggle("settings_notifications", isOn: $notificationsEnabled)
                    
                    if notificationsEnabled {
                        Toggle("settings_notifications_sound", isOn: .constant(true))
                        Toggle("settings_notifications_badge", isOn: .constant(true))
                    }
                }
                
                // Display Section
                Section(header: Text("settings_section_display")) {
                    Toggle("settings_dark_mode", isOn: $darkModeEnabled)
                    
                    NavigationLink("settings_text_size") {
                        TextSizeSettingsView()
                    }
                }
                
                // About Section
                Section(header: Text("settings_section_about")) {
                    HStack {
                        Text("settings_version")
                        Spacer()
                        Text(Bundle.main.infoDictionary?["CFBundleShortVersionString"] as? String ?? "1.0")
                            .foregroundColor(.secondary)
                    }
                    
                    NavigationLink("settings_privacy_policy") {
                        PrivacyPolicyView()
                    }
                    
                    NavigationLink("settings_terms_of_service") {
                        TermsView()
                    }
                }
            }
            .navigationTitle("nav_settings")
        }
    }
}

struct TextSizeSettingsView: View {
    var body: some View {
        List {
            Text("Sample text in current size")
                .font(.body)
            Text("settings_text_size_hint")
                .font(.caption)
                .foregroundColor(.secondary)
        }
        .navigationTitle("settings_text_size")
    }
}

struct PrivacyPolicyView: View {
    var body: some View {
        ScrollView {
            Text("privacy_policy_content")
                .padding()
        }
        .navigationTitle("settings_privacy_policy")
    }
}

struct TermsView: View {
    var body: some View {
        ScrollView {
            Text("terms_content")
                .padding()
        }
        .navigationTitle("settings_terms_of_service")
    }
}

struct SearchView: View {
    @State private var searchText = ""
    
    var body: some View {
        NavigationView {
            Text("search_empty_state")
                .searchable(text: $searchText, prompt: Text("search_placeholder"))
                .navigationTitle("nav_search")
        }
    }
}

struct ProfileView: View {
    var body: some View {
        NavigationView {
            Text("profile_content")
                .navigationTitle("nav_profile")
        }
    }
}
```

---

## 23. สรุป

Localization เป็นกระบวนการที่สำคัญในการขยายตลาดของแอปพลิเคชัน โดยสรุปสิ่งที่ได้เรียนรู้:

### สิ่งสำคัญที่ต้องจำ

1. **i18n** = เตรียมระบบ, **L10n** = แปลเนื้อหา
2. ใช้ `NSLocalizedString` หรือ `String(localized:)` สำหรับทุก string ที่แสดงผล
3. ใช้ `Localizable.stringsdict` สำหรับ plural forms
4. ใช้ `DateFormatter`, `NumberFormatter`, `MeasurementFormatter` ตาม locale
5. Design layout ที่ flexible รองรับข้อความยาว
6. Support RTL languages ด้วย leading/trailing แทน left/right
7. Test กับ multiple locales ด้วย Xcode Scheme

### Best Practices

1. ตั้งชื่อ key อย่างมีความหมายและ namespace
2. เพิ่ม comment สำหรับ translator
3. ใช้ positional format specifiers (%1$@, %2$d)
4. ทดสอบด้วยภาษาที่มีข้อความยาว (เช่น เยอรมัน)
5. ทดสอบ RTL ด้วยภาษาอาหรับ
6. ใช้ semantic colors และ system fonts
7. Localize ไม่ใช่แค่ข้อความ แต่รวมถึงรูปภาพ วันที่ เวลา ตัวเลข
8. ใช้ String Catalogs (.xcstrings) ใน Xcode 15+
9. Export/Import XLIFF สำหรับนักแปล
10. เขียน Unit Tests และ UI Tests สำหรับ localization

### Checklist ก่อน Release

- [ ] ทุก user-facing string ใช้ NSLocalizedString หรือ LocalizedStringKey
- [ ] ทุก string มี comment สำหรับ translator
- [ ] Numbers, dates, currencies ใช้ formatter ที่ locale-aware
- [ ] Layout ทดสอบกับข้อความที่ยาว (2x ความยาวต้น)
- [ ] RTL layout ถูกต้อง
- [ ] App name และ permission strings localize แล้ว
- [ ] ทดสอบกับ Dynamic Type ทุกขนาด
- [ ] XLIFF ส่งให้ professional translator review
- [ ] Screenshot และรูปภาพที่มีข้อความ localize แล้ว
- [ ] ผ่าน localization testing ด้วย Xcode Scheme

---

*จบ Part 46: Localization และ Internationalization*
