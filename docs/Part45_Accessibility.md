# Part 45: Accessibility ใน iOS Development

## บทนำ

Accessibility หรือการเข้าถึงได้ เป็นหนึ่งในหัวข้อที่สำคัญที่สุดในการพัฒนาแอปพลิเคชัน iOS แต่มักถูกมองข้ามโดยนักพัฒนาหลายคน บทนี้จะพาคุณเรียนรู้วิธีทำให้แอปของคุณใช้งานได้สำหรับทุกคน รวมถึงผู้ที่มีความพิการทางร่างกาย การมองเห็น หรือการได้ยิน

---

## 1. ทำไม Accessibility ถึงสำคัญ

### 1.1 ความหมายและความสำคัญ

Accessibility หมายถึงการออกแบบผลิตภัณฑ์และบริการให้ผู้คนทุกกลุ่มสามารถใช้งานได้ รวมถึงผู้ที่มีข้อจำกัดทางร่างกายหรือความสามารถ

ในโลกดิจิทัล มีประชากรราว 15% ของโลก หรือกว่า 1 พันล้านคน ที่อาศัยอยู่กับความพิการบางรูปแบบ ซึ่งรวมถึง:

- **ความพิการทางการมองเห็น** (Visual Disabilities): ตาบอด สายตาเลือนราง ตาบอดสี
- **ความพิการทางการได้ยิน** (Hearing Disabilities): หูหนวก การได้ยินบกพร่อง
- **ความพิการทางการเคลื่อนไหว** (Motor Disabilities): อัมพาต สั่นกระตุก ขาดแขน
- **ความพิการทางสติปัญญา** (Cognitive Disabilities): ดิสเล็กเซีย ออทิสซึม ADHD

### 1.2 เหตุผลในการทำ Accessibility

**เหตุผลด้านจริยธรรม:**
- ทุกคนมีสิทธิ์เข้าถึงเทคโนโลยี
- การสร้างผลิตภัณฑ์ที่ inclusive เป็นการแสดงความรับผิดชอบต่อสังคม

**เหตุผลทางธุรกิจ:**
- ขยายฐานผู้ใช้งาน
- หลีกเลี่ยงการถูกฟ้องร้องทางกฎหมาย (ADA Compliance)
- ปรับปรุง UX สำหรับผู้ใช้ทุกคน

**เหตุผลทางเทคนิค:**
- Accessibility code มักสะอาดและมีโครงสร้างดีกว่า
- ช่วยใน SEO และ discoverability
- เป็น requirement ของ App Store ใน Apple

### 1.3 Apple และ Accessibility

Apple เป็นบริษัทที่ให้ความสำคัญกับ accessibility มาก นับตั้งแต่ปี 2001 ที่เริ่มมี VoiceOver บน Mac โดย iOS มี accessibility features ที่ครอบคลุมมาก เช่น:

- **VoiceOver**: Screen reader สำหรับผู้พิการทางการมองเห็น
- **Switch Control**: สำหรับผู้ที่มีข้อจำกัดด้านการเคลื่อนไหว
- **Voice Control**: ควบคุมด้วยเสียง
- **Display Accommodations**: ปรับหน้าจอสำหรับความต้องการต่างๆ
- **Dynamic Type**: ปรับขนาดตัวอักษร

---

## 2. VoiceOver

### 2.1 VoiceOver คืออะไร

VoiceOver เป็น screen reader ที่มาพร้อมกับ iOS ที่อ่านสิ่งที่อยู่บนหน้าจอออกเสียง ผู้ใช้ควบคุมด้วย gestures บน touchscreen แทนการกดปุ่ม

### 2.2 วิธีใช้ VoiceOver

เปิดใช้งาน VoiceOver:
- **Settings** > **Accessibility** > **VoiceOver**
- หรือ Triple-click home/side button (ถ้าตั้งค่า Accessibility Shortcut ไว้)

**Gestures พื้นฐาน:**
- **Single tap**: เลือก item และอ่านออกเสียง
- **Double tap**: activate ปุ่มหรือ control
- **Swipe left/right**: เลื่อนไปยัง element ก่อนหน้า/ถัดไป
- **Three-finger swipe**: scroll
- **Two-finger swipe up**: อ่านทั้งหน้า

### 2.3 วิธีที่ VoiceOver อ่าน Elements

เมื่อ VoiceOver focus บน element จะอ่านข้อมูล 4 ส่วน:
1. **Label**: ชื่อของ element
2. **Value**: ค่าปัจจุบัน (เช่น ค่าของ slider)
3. **Traits**: ประเภทของ element (เช่น "button", "heading")
4. **Hint**: คำอธิบายการกระทำที่จะเกิดขึ้น

### 2.4 ตัวอย่างการทดสอบ VoiceOver ด้วย Simulator

```swift
// ใน Xcode Simulator, ไปที่ Hardware > Home > ตั้งค่า Accessibility
// หรือใช้ Accessibility Inspector

import SwiftUI

struct VoiceOverDemoView: View {
    @State private var isPlaying = false
    
    var body: some View {
        VStack(spacing: 20) {
            Text("เพลงโปรด")
                .font(.largeTitle)
            
            // ปุ่มธรรมดา - VoiceOver จะอ่านว่า "Play, button"
            Button(action: { isPlaying.toggle() }) {
                Image(systemName: isPlaying ? "pause.fill" : "play.fill")
                    .font(.system(size: 44))
            }
            // เพิ่ม accessibility label เพื่อให้ชัดเจนขึ้น
            .accessibilityLabel(isPlaying ? "หยุดเล่น" : "เล่น")
            .accessibilityHint("แตะสองครั้งเพื่อ\(isPlaying ? "หยุดเล่น" : "เล่น")เพลง")
        }
    }
}
```

---

## 3. Accessibility Inspector

### 3.1 Accessibility Inspector คืออะไร

Accessibility Inspector เป็นเครื่องมือใน Xcode ที่ช่วยให้คุณตรวจสอบ accessibility properties ของ UI elements ในแอปโดยไม่ต้องใช้ VoiceOver จริงๆ

### 3.2 วิธีใช้ Accessibility Inspector

1. เปิด Xcode
2. ไปที่ **Xcode** > **Open Developer Tool** > **Accessibility Inspector**
3. เลือก Simulator หรือ Device ที่ต้องการตรวจสอบ
4. คลิกปุ่ม "Inspection" (ลูกศรที่มีวงกลม)
5. hover เมาส์บน element ใน simulator เพื่อดู accessibility info

### 3.3 ฟีเจอร์ของ Accessibility Inspector

**Inspection Panel:**
- แสดง label, value, traits, hint
- แสดง frame และ position
- แสดง activation point

**Audit:**
- ตรวจสอบปัญหา accessibility อัตโนมัติ
- แสดง warning เช่น contrast ratio ต่ำ, element ขนาดเล็กเกินไป

**Settings:**
- จำลองการตั้งค่า accessibility ต่างๆ
- ทดสอบ Dynamic Type, Bold Text, Color Filters

```swift
// ตัวอย่าง: element ที่ Accessibility Inspector จะแสดง warning
struct PoorAccessibilityView: View {
    var body: some View {
        VStack {
            // Warning: ขนาดเล็กเกินไป (ต่ำกว่า 44x44 points)
            Button("X") {
                // dismiss action
            }
            .frame(width: 20, height: 20)
            
            // Warning: ไม่มี accessibility label
            Image(systemName: "heart.fill")
                .foregroundColor(.red)
            
            // Warning: contrast ratio ต่ำ
            Text("ข้อความสีจาง")
                .foregroundColor(Color.gray.opacity(0.3))
                .background(Color.white)
        }
    }
}

// ตัวอย่าง: element ที่ผ่าน accessibility audit
struct GoodAccessibilityView: View {
    var body: some View {
        VStack {
            // ขนาดเพียงพอ
            Button("ปิด") {
                // dismiss action
            }
            .frame(minWidth: 44, minHeight: 44)
            
            // มี accessibility label
            Image(systemName: "heart.fill")
                .foregroundColor(.red)
                .accessibilityLabel("ชื่นชอบ")
            
            // contrast ratio เพียงพอ
            Text("ข้อความชัดเจน")
                .foregroundColor(.black)
                .background(Color.white)
        }
    }
}
```

---

## 4. accessibilityLabel

### 4.1 ความหมาย

`accessibilityLabel` คือชื่อหรือคำอธิบายของ element ที่ VoiceOver จะอ่านออกเสียง เป็น property ที่สำคัญที่สุดใน accessibility

### 4.2 เมื่อไหร่ควรใช้

- เมื่อ element เป็น icon หรือ image ที่ไม่มีข้อความ
- เมื่อข้อความบน element ไม่สื่อความหมายพอ
- เมื่อต้องการรวม context เพิ่มเติม

### 4.3 ตัวอย่างการใช้งาน

```swift
import SwiftUI

struct AccessibilityLabelExamples: View {
    var body: some View {
        VStack(spacing: 20) {
            // 1. Image ที่ไม่มีข้อความ
            Image(systemName: "trash")
                .accessibilityLabel("ลบ")
            
            // 2. ปุ่มที่มีแค่ icon
            Button(action: {}) {
                Image(systemName: "square.and.arrow.up")
            }
            .accessibilityLabel("แชร์บทความ")
            
            // 3. ข้อมูลที่ต้องการ context เพิ่ม
            VStack {
                Text("฿1,250.00")
                    .accessibilityLabel("ราคา หนึ่งพันสองร้อยห้าสิบบาท")
                
                Text("★★★★☆")
                    .accessibilityLabel("คะแนน สี่จากห้าดาว")
            }
            
            // 4. ปุ่มที่มีข้อความแต่ต้องการ label ที่ชัดเจนกว่า
            Button("ไปต่อ") {}
                .accessibilityLabel("ไปหน้าถัดไป: ยืนยันการสั่งซื้อ")
            
            // 5. TextField
            TextField("อีเมล", text: .constant(""))
                .accessibilityLabel("อีเมลสำหรับติดต่อ")
        }
        .padding()
    }
}

// ใน UIKit
class AccessibilityLabelUIKitExample: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let deleteButton = UIButton()
        deleteButton.setImage(UIImage(systemName: "trash"), for: .normal)
        deleteButton.accessibilityLabel = "ลบรายการ"
        
        let starRating = UILabel()
        starRating.text = "★★★☆☆"
        starRating.accessibilityLabel = "คะแนน สามจากห้าดาว"
        
        let priceLabel = UILabel()
        priceLabel.text = "$12.99"
        priceLabel.accessibilityLabel = "ราคา สิบสองดอลลาร์เก้าสิบเก้าเซนต์"
    }
}
```

### 4.4 Best Practices สำหรับ accessibilityLabel

```swift
struct LabelBestPractices: View {
    var body: some View {
        VStack {
            // ✅ ดี: สั้น, ชัดเจน, ไม่มีคำว่า "button" (VoiceOver เพิ่มให้เอง)
            Button("บันทึก") {}
                .accessibilityLabel("บันทึกการเปลี่ยนแปลง")
            
            // ❌ ไม่ดี: ยาวและซ้ำซ้อน
            Button("X") {}
                .accessibilityLabel("นี่คือปุ่ม X สำหรับปิดหน้าต่างนี้ กรุณาแตะสองครั้งเพื่อปิด")
            
            // ✅ ดี: ใช้ภาษาธรรมชาติ
            Toggle("การแจ้งเตือน", isOn: .constant(true))
                .accessibilityLabel("เปิดรับการแจ้งเตือน")
            
            // ✅ ดี: รวม state เข้าไปใน label เมื่อจำเป็น
            let isLiked = true
            Button(action: {}) {
                Image(systemName: isLiked ? "heart.fill" : "heart")
            }
            .accessibilityLabel(isLiked ? "เลิกชื่นชอบ" : "เพิ่มในรายการชื่นชอบ")
        }
    }
}
```

---

## 5. accessibilityHint

### 5.1 ความหมาย

`accessibilityHint` คือคำอธิบายเพิ่มเติมที่บอกว่าจะเกิดอะไรขึ้นเมื่อผู้ใช้โต้ตอบกับ element นั้น VoiceOver จะอ่าน hint หลังจาก label และ value

### 5.2 เมื่อไหร่ควรใช้

- เมื่อการกระทำที่จะเกิดขึ้นไม่ชัดเจนจาก label
- เมื่อต้องการอธิบาย side effects ของการกระทำ
- เมื่อมีหลายวิธีโต้ตอบกับ element

### 5.3 ตัวอย่างการใช้งาน

```swift
struct AccessibilityHintExamples: View {
    @State private var showingDetail = false
    @State private var isFavorite = false
    
    var body: some View {
        VStack(spacing: 20) {
            // 1. ปุ่มที่อธิบายผลลัพธ์
            Button("ส่งข้อความ") {}
                .accessibilityHint("ส่งข้อความไปยังผู้รับที่เลือกไว้ทั้งหมด")
            
            // 2. Toggle พร้อม hint
            Toggle("โหมดกลางคืน", isOn: .constant(false))
                .accessibilityHint("เปิดใช้งานเพื่อลดความสว่างหน้าจอในที่มืด")
            
            // 3. Link ที่บอกว่าจะเปิดที่ไหน
            Link("ดูนโยบายความเป็นส่วนตัว", destination: URL(string: "https://example.com")!)
                .accessibilityHint("เปิดใน Safari")
            
            // 4. Slider พร้อม hint
            Slider(value: .constant(0.5))
                .accessibilityLabel("ระดับเสียง")
                .accessibilityHint("ปัดซ้ายหรือขวาเพื่อปรับระดับเสียง")
            
            // 5. ปุ่มที่มีสถานะ
            Button(action: { isFavorite.toggle() }) {
                Image(systemName: isFavorite ? "star.fill" : "star")
            }
            .accessibilityLabel(isFavorite ? "อยู่ในรายการโปรด" : "ไม่อยู่ในรายการโปรด")
            .accessibilityHint("แตะสองครั้งเพื่อ\(isFavorite ? "นำออก" : "เพิ่ม")จากรายการโปรด")
            
            // 6. Cell ที่มีหลาย action
            Text("รายการ 1")
                .accessibilityHint("แตะสองครั้งเพื่อเปิด, ปัดซ้ายเพื่อดูตัวเลือกเพิ่มเติม")
        }
        .padding()
    }
}
```

### 5.4 Hint Best Practices

```swift
struct HintBestPractices: View {
    var body: some View {
        VStack {
            // ✅ ดี: อธิบายผลลัพธ์
            Button("ลบ") {}
                .accessibilityHint("ลบรายการนี้ออกจากรายการถาวร")
            
            // ❌ ไม่ดี: บอก action ซ้ำกับ label
            Button("ลบ") {}
                .accessibilityHint("แตะเพื่อลบ") // ซ้ำซ้อน
            
            // ✅ ดี: ใช้รูปแบบกริยา "เพื่อ..."
            Button("แก้ไข") {}
                .accessibilityHint("เพื่อเปิดหน้าแก้ไขข้อมูลส่วนตัว")
            
            // ✅ ดี: Hint ควรเป็น optional - ไม่จำเป็นต้องใส่ทุก element
            // elements ที่ชัดเจนอยู่แล้วไม่ต้องการ hint
            Button("OK") {}
            // ไม่จำเป็นต้องเพิ่ม hint
        }
    }
}
```

---

## 6. accessibilityValue

### 6.1 ความหมาย

`accessibilityValue` คือค่าปัจจุบันของ element ที่อาจเปลี่ยนแปลงได้ เช่น ค่าของ slider, สถานะของ toggle, หรือ progress ของ progress bar

### 6.2 ตัวอย่างการใช้งาน

```swift
struct AccessibilityValueExamples: View {
    @State private var volume: Double = 0.7
    @State private var brightness: Double = 0.5
    @State private var currentPage = 3
    let totalPages = 10
    
    var body: some View {
        VStack(spacing: 20) {
            // 1. Slider ที่แสดง value เป็นเปอร์เซ็นต์
            VStack {
                Text("ระดับเสียง")
                Slider(value: $volume)
                    .accessibilityLabel("ระดับเสียง")
                    .accessibilityValue("\(Int(volume * 100)) เปอร์เซ็นต์")
            }
            
            // 2. Pagination
            VStack {
                Text("หน้า \(currentPage) จาก \(totalPages)")
                    .accessibilityValue("หน้า \(currentPage) จาก \(totalPages) หน้า")
                
                HStack {
                    Button("ก่อนหน้า") { 
                        if currentPage > 1 { currentPage -= 1 }
                    }
                    Button("ถัดไป") { 
                        if currentPage < totalPages { currentPage += 1 }
                    }
                }
            }
            
            // 3. Custom progress indicator
            CustomProgressView(progress: 0.6)
                .accessibilityLabel("ความคืบหน้าการดาวน์โหลด")
                .accessibilityValue("60 เปอร์เซ็นต์")
            
            // 4. Star rating
            StarRatingView(rating: 4)
                .accessibilityLabel("คะแนน")
                .accessibilityValue("4 จาก 5 ดาว")
            
            // 5. Custom toggle
            CustomToggle(isOn: .constant(true))
                .accessibilityLabel("การแจ้งเตือน")
                .accessibilityValue("เปิดใช้งาน")
        }
        .padding()
    }
}

struct CustomProgressView: View {
    let progress: Double
    
    var body: some View {
        ProgressView(value: progress)
            .progressViewStyle(.linear)
    }
}

struct StarRatingView: View {
    let rating: Int
    
    var body: some View {
        HStack {
            ForEach(1...5, id: \.self) { star in
                Image(systemName: star <= rating ? "star.fill" : "star")
                    .foregroundColor(.yellow)
            }
        }
        .accessibilityElement(children: .ignore)
        .accessibilityLabel("คะแนน \(rating) ดาว")
    }
}

struct CustomToggle: View {
    @Binding var isOn: Bool
    
    var body: some View {
        RoundedRectangle(cornerRadius: 15)
            .fill(isOn ? Color.green : Color.gray)
            .frame(width: 51, height: 31)
            .onTapGesture { isOn.toggle() }
    }
}
```

---

## 7. accessibilityTraits

### 7.1 ความหมาย

`accessibilityTraits` บอก VoiceOver ว่า element นั้นเป็นประเภทอะไรและทำงานอย่างไร ช่วยให้ผู้ใช้เข้าใจว่าจะโต้ตอบกับ element อย่างไร

### 7.2 Traits ที่มีใน iOS

```swift
// AccessibilityTraits ที่ใช้บ่อย:
// .button         - element ที่กดได้
// .link           - เปิด URL หรือเนื้อหาอื่น
// .header         - หัวข้อส่วน
// .image          - รูปภาพ
// .selected       - รายการที่เลือกอยู่
// .notEnabled     - ปิดใช้งาน
// .updatesFrequently - อัปเดตบ่อย (เช่น timer)
// .adjustable     - ปรับค่าได้ (เช่น slider)
// .allowsDirectInteraction - โต้ตอบโดยตรง (เช่น keyboard, piano)
// .staticText     - ข้อความที่ไม่เปลี่ยน
// .playsSound     - เล่นเสียงเมื่อ activate
// .keyboardKey    - ปุ่มบน keyboard
// .summaryElement - แสดงสรุปข้อมูลสำคัญ
// .searchField    - ช่องค้นหา
// .tabBar         - tab bar item
```

### 7.3 ตัวอย่างการใช้งาน

```swift
struct AccessibilityTraitsExamples: View {
    @State private var selectedTab = 0
    @State private var sliderValue: Double = 50
    
    var body: some View {
        VStack(spacing: 20) {
            // 1. Header
            Text("รายการสินค้า")
                .font(.title)
                .accessibilityAddTraits(.isHeader)
            
            // 2. ข้อความที่เป็น link
            Text("ดูเพิ่มเติม")
                .foregroundColor(.blue)
                .accessibilityAddTraits(.isLink)
            
            // 3. Image ที่เป็น decorative (ไม่ต้องอ่าน)
            Image("background_decoration")
                .accessibilityHidden(true) // ซ่อนจาก VoiceOver
            
            // 4. Custom tab selector
            HStack {
                ForEach(["ทั้งหมด", "ยอดนิยม", "ใหม่ล่าสุด"].indices, id: \.self) { index in
                    Text(["ทั้งหมด", "ยอดนิยม", "ใหม่ล่าสุด"][index])
                        .padding(8)
                        .background(selectedTab == index ? Color.blue : Color.gray.opacity(0.2))
                        .cornerRadius(8)
                        .onTapGesture { selectedTab = index }
                        .accessibilityAddTraits(selectedTab == index ? [.isButton, .isSelected] : .isButton)
                }
            }
            
            // 5. Adjustable slider
            CustomSlider(value: $sliderValue)
                .accessibilityLabel("ความสว่าง")
                .accessibilityValue("\(Int(sliderValue))%")
                .accessibilityAddTraits(.isAdjustable)
                .accessibilityAdjustableAction { direction in
                    switch direction {
                    case .increment: sliderValue = min(100, sliderValue + 10)
                    case .decrement: sliderValue = max(0, sliderValue - 10)
                    @unknown default: break
                    }
                }
            
            // 6. ปุ่มที่ disabled
            Button("ยืนยัน") {}
                .disabled(true)
                .accessibilityAddTraits(.isNotEnabled)
            
            // 7. Status ที่อัปเดตบ่อย
            Text("เวลาปัจจุบัน: 12:34:56")
                .accessibilityAddTraits(.updatesFrequently)
        }
        .padding()
    }
}

struct CustomSlider: View {
    @Binding var value: Double
    
    var body: some View {
        Slider(value: $value, in: 0...100)
    }
}
```

---

## 8. accessibilityAction

### 8.1 ความหมาย

`accessibilityAction` ช่วยให้ผู้ใช้ VoiceOver สามารถ activate actions ที่ไม่ใช่ default double-tap gesture ได้ เช่น การ swipe หรือ long press

### 8.2 ประเภทของ Actions

```swift
// Actions ที่ built-in:
// .default     - action หลัก (double tap)
// .longPress   - long press
// .activate    - เปิดใช้งาน
// .adjust      - ปรับค่า
// .delete      - ลบ
// .escape      - ยกเลิก/ปิด

// Custom actions
```

### 8.3 ตัวอย่างการใช้งาน

```swift
struct AccessibilityActionExamples: View {
    @State private var items = ["รายการ 1", "รายการ 2", "รายการ 3"]
    @State private var showingAlert = false
    @State private var alertMessage = ""
    
    var body: some View {
        VStack {
            List {
                ForEach(items, id: \.self) { item in
                    Text(item)
                        .accessibilityActions {
                            // 1. Custom actions บน List item
                            Button("แก้ไข") {
                                alertMessage = "แก้ไข: \(item)"
                                showingAlert = true
                            }
                            Button("แชร์") {
                                alertMessage = "แชร์: \(item)"
                                showingAlert = true
                            }
                            Button("ลบ") {
                                items.removeAll { $0 == item }
                            }
                        }
                }
            }
            
            // 2. เพิ่ม custom action บนปุ่ม
            Button("บทความ") {}
                .accessibilityAction(named: "บันทึกสำหรับอ่านภายหลัง") {
                    // save article
                }
                .accessibilityAction(named: "แชร์กับเพื่อน") {
                    // share
                }
            
            // 3. Action ที่ใช้ gesture
            Rectangle()
                .fill(Color.blue.opacity(0.3))
                .frame(height: 100)
                .accessibilityLabel("พื้นที่วาดภาพ")
                .accessibilityAction(.default) {
                    // Clear canvas
                }
                .accessibilityAction(named: "บันทึกภาพ") {
                    // save
                }
        }
        .alert("Action", isPresented: $showingAlert) {
            Button("OK") {}
        } message: {
            Text(alertMessage)
        }
    }
}
```

---

## 9. Semantic Views ใน SwiftUI

### 9.1 ความหมายของ Semantic Views

SwiftUI มี views หลายตัวที่มี accessibility semantics ในตัวเอง เช่น `Button`, `Toggle`, `Slider` เหล่านี้มี traits และ behaviors ที่เหมาะสมโดยอัตโนมัติ

### 9.2 ตัวอย่าง Semantic Views

```swift
struct SemanticViewsExample: View {
    @State private var toggleValue = false
    @State private var sliderValue: Double = 50
    @State private var selectedDate = Date()
    @State private var stepperValue = 1
    
    var body: some View {
        Form {
            // Button - มี .isButton trait อัตโนมัติ
            Section("การกระทำ") {
                Button("ส่งข้อความ") {}
                // VoiceOver: "ส่งข้อความ, ปุ่ม"
                
                Button(role: .destructive) {
                    // delete
                } label: {
                    Text("ลบบัญชี")
                }
                // VoiceOver: "ลบบัญชี, ปุ่ม"
            }
            
            // Toggle - มี .isButton และ value อัตโนมัติ
            Section("การตั้งค่า") {
                Toggle("การแจ้งเตือน", isOn: $toggleValue)
                // VoiceOver: "การแจ้งเตือน, สวิตช์, [เปิด/ปิด]"
            }
            
            // Slider - มี .isAdjustable trait อัตโนมัติ
            Section("ปรับค่า") {
                Slider(value: $sliderValue, in: 0...100, step: 1) {
                    Text("ระดับเสียง")
                }
                // VoiceOver: "ระดับเสียง, สไลเดอร์, [ค่า]"
                
                Stepper("จำนวน: \(stepperValue)", value: $stepperValue, in: 1...10)
                // VoiceOver: "จำนวน: [ค่า], สเตปเปอร์"
            }
            
            // DatePicker - มี accessibility ในตัว
            Section("เลือกวันที่") {
                DatePicker("วันเกิด", selection: $selectedDate, displayedComponents: .date)
                // VoiceOver: "วันเกิด, [วันที่ปัจจุบัน], เลือกวันที่"
            }
            
            // Picker - มี accessibility ในตัว
            Section("เลือกตัวเลือก") {
                Picker("ภาษา", selection: .constant("ไทย")) {
                    Text("ไทย").tag("ไทย")
                    Text("English").tag("English")
                }
                // VoiceOver: "ภาษา, [ตัวเลือกที่เลือก], เลือกตัวเลือก"
            }
        }
    }
}
```

### 9.3 Custom View ที่มี Accessibility

```swift
// สร้าง custom view ที่มี semantics ชัดเจน
struct RatingSelector: View {
    @Binding var rating: Int
    let maxRating: Int
    
    var body: some View {
        HStack {
            ForEach(1...maxRating, id: \.self) { star in
                Image(systemName: star <= rating ? "star.fill" : "star")
                    .foregroundColor(.yellow)
                    .onTapGesture { rating = star }
                    .accessibilityHidden(true) // ซ่อน individual stars
            }
        }
        // ใช้ container level accessibility
        .accessibilityElement(children: .ignore)
        .accessibilityLabel("คะแนน")
        .accessibilityValue("\(rating) จาก \(maxRating) ดาว")
        .accessibilityAdjustableAction { direction in
            switch direction {
            case .increment: rating = min(maxRating, rating + 1)
            case .decrement: rating = max(0, rating - 1)
            @unknown default: break
            }
        }
        .accessibilityAddTraits(.isAdjustable)
    }
}
```

---

## 10. accessibilityElement(children:)

### 10.1 ความหมาย

`accessibilityElement(children:)` modifier ช่วยควบคุมว่า VoiceOver จะโต้ตอบกับ view นั้นอย่างไร โดยสามารถรวม children เป็น element เดียว หรือแยกออก

### 10.2 ตัวเลือกของ children parameter

```swift
// .combine - รวม children เป็น element เดียว อ่านเนื้อหาทั้งหมดรวมกัน
// .contain - แต่ละ child เป็น element แยก (ค่า default)
// .ignore  - ซ่อน children, ใช้ parent เป็น element เดียว
```

### 10.3 ตัวอย่างการใช้งาน

```swift
struct AccessibilityElementExamples: View {
    var body: some View {
        VStack(spacing: 20) {
            // 1. .combine - รวมข้อมูลหลายชิ้นเป็น element เดียว
            VStack(alignment: .leading) {
                Text("สมชาย ใจดี")
                    .font(.headline)
                Text("ผู้จัดการ")
                    .font(.subheadline)
                    .foregroundColor(.secondary)
                Text("02-123-4567")
                    .font(.caption)
            }
            .accessibilityElement(children: .combine)
            // VoiceOver จะอ่าน: "สมชาย ใจดี, ผู้จัดการ, 02-123-4567"
            
            // 2. .ignore - ซ่อน children, กำหนด accessibility เองทั้งหมด
            ZStack {
                Circle()
                    .fill(Color.blue)
                    .frame(width: 80, height: 80)
                
                VStack {
                    Text("72")
                        .font(.title)
                        .foregroundColor(.white)
                    Text("BPM")
                        .font(.caption)
                        .foregroundColor(.white)
                }
            }
            .accessibilityElement(children: .ignore)
            .accessibilityLabel("อัตราการเต้นของหัวใจ")
            .accessibilityValue("72 ครั้งต่อนาที")
            
            // 3. .contain - แต่ละ child accessible แยกกัน (default)
            HStack {
                Button("ยกเลิก") {}
                Button("ตกลง") {}
            }
            .accessibilityElement(children: .contain)
            // VoiceOver สามารถนำทางระหว่าง "ยกเลิก" และ "ตกลง" แยกกัน
            
            // 4. Card ที่รวมข้อมูลทั้งหมด
            ProductCard(name: "เสื้อยืด", price: 299, rating: 4)
        }
        .padding()
    }
}

struct ProductCard: View {
    let name: String
    let price: Int
    let rating: Int
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            Image(systemName: "tshirt")
                .resizable()
                .frame(width: 100, height: 100)
                .accessibilityHidden(true) // ซ่อนรูป decorative
            
            Text(name)
                .font(.headline)
            
            HStack {
                ForEach(1...5, id: \.self) { star in
                    Image(systemName: star <= rating ? "star.fill" : "star")
                        .foregroundColor(.yellow)
                        .font(.caption)
                }
            }
            .accessibilityHidden(true)
            
            Text("฿\(price)")
                .font(.title3)
                .bold()
        }
        .padding()
        .background(Color.white)
        .cornerRadius(12)
        .shadow(radius: 2)
        // รวมทั้ง card เป็น element เดียว
        .accessibilityElement(children: .ignore)
        .accessibilityLabel("\(name)")
        .accessibilityValue("ราคา \(price) บาท คะแนน \(rating) จาก 5 ดาว")
        .accessibilityHint("แตะสองครั้งเพื่อดูรายละเอียด")
        .accessibilityAddTraits(.isButton)
    }
}
```

---

## 11. accessibilityRepresentation

### 11.1 ความหมาย

`accessibilityRepresentation(representation:)` ช่วยให้คุณสร้าง accessibility representation ที่แตกต่างจาก visual representation ของ view โดยสิ้นเชิง มีประโยชน์เมื่อ visual design ซับซ้อนแต่ต้องการ accessibility ที่เรียบง่าย

### 11.2 ตัวอย่างการใช้งาน

```swift
struct AccessibilityRepresentationExamples: View {
    @State private var selectedColor: Color = .blue
    
    var body: some View {
        VStack(spacing: 20) {
            // 1. Color picker ที่ซับซ้อน → แสดงเป็น Picker สำหรับ accessibility
            CustomColorPicker(selectedColor: $selectedColor)
            
            // 2. Graph → แสดงเป็น text สำหรับ accessibility
            SalesGraph()
        }
    }
}

struct CustomColorPicker: View {
    @Binding var selectedColor: Color
    
    let colors: [(name: String, color: Color)] = [
        ("แดง", .red),
        ("เขียว", .green),
        ("น้ำเงิน", .blue),
        ("เหลือง", .yellow),
        ("ม่วง", .purple)
    ]
    
    var selectedColorName: String {
        colors.first { $0.color == selectedColor }?.name ?? "ไม่ทราบ"
    }
    
    var body: some View {
        HStack {
            ForEach(colors, id: \.name) { item in
                Circle()
                    .fill(item.color)
                    .frame(width: 44, height: 44)
                    .overlay(
                        Circle()
                            .stroke(Color.white, lineWidth: selectedColor == item.color ? 3 : 0)
                    )
                    .onTapGesture {
                        selectedColor = item.color
                    }
            }
        }
        // ใช้ Picker เป็น accessibility representation
        .accessibilityRepresentation {
            Picker("เลือกสี", selection: $selectedColor) {
                ForEach(colors, id: \.name) { item in
                    Text(item.name).tag(item.color)
                }
            }
        }
    }
}

struct SalesGraph: View {
    let data = [
        ("ม.ค.", 1200),
        ("ก.พ.", 1800),
        ("มี.ค.", 1500),
        ("เม.ย.", 2100),
        ("พ.ค.", 1900)
    ]
    
    var body: some View {
        // Visual graph ที่ซับซ้อน
        GeometryReader { geometry in
            HStack(alignment: .bottom, spacing: 4) {
                ForEach(data, id: \.0) { item in
                    VStack {
                        Rectangle()
                            .fill(Color.blue)
                            .frame(
                                width: geometry.size.width / CGFloat(data.count) - 4,
                                height: CGFloat(item.1) / 2100.0 * geometry.size.height
                            )
                        Text(item.0)
                            .font(.caption2)
                    }
                }
            }
        }
        .frame(height: 200)
        // แทนที่ด้วย accessible version
        .accessibilityRepresentation {
            VStack(alignment: .leading) {
                Text("ยอดขายรายเดือน")
                    .accessibilityAddTraits(.isHeader)
                ForEach(data, id: \.0) { item in
                    Text("\(item.0): \(item.1) บาท")
                }
            }
        }
    }
}
```

---

## 12. accessibilityChildren

### 12.1 ความหมาย

`accessibilityChildren(children:)` ใช้เพื่อกำหนด accessibility children ที่กำหนดเองสำหรับ view ซึ่งแตกต่างจาก visual children

### 12.2 ตัวอย่างการใช้งาน

```swift
struct AccessibilityChildrenExample: View {
    let notifications = [
        "ข้อความใหม่จาก สมชาย",
        "เตือนความจำ: นัดหมายพรุ่งนี้",
        "อัปเดต: แอปรุ่นใหม่พร้อมใช้งาน"
    ]
    
    var body: some View {
        // Notification badge ที่แสดงจำนวน
        ZStack(alignment: .topTrailing) {
            Image(systemName: "bell")
                .font(.title)
            
            if !notifications.isEmpty {
                Circle()
                    .fill(Color.red)
                    .frame(width: 20, height: 20)
                    .overlay(
                        Text("\(notifications.count)")
                            .foregroundColor(.white)
                            .font(.caption2)
                    )
            }
        }
        .accessibilityElement(children: .ignore)
        .accessibilityLabel("การแจ้งเตือน")
        .accessibilityValue("\(notifications.count) รายการที่ยังไม่ได้อ่าน")
        // กำหนด children ที่สามารถนำทางได้
        .accessibilityChildren {
            ForEach(notifications, id: \.self) { notification in
                Text(notification)
                    .accessibilityAddTraits(.isButton)
            }
        }
    }
}
```

---

## 13. Dynamic Type Support

### 13.1 Dynamic Type คืออะไร

Dynamic Type เป็นฟีเจอร์ของ iOS ที่ช่วยให้ผู้ใช้สามารถปรับขนาดตัวอักษรตามความต้องการของตัวเอง ตั้งแต่ Extra Small ถึง Accessibility XXXL

### 13.2 ขนาด Text Style ต่างๆ

```swift
struct DynamicTypeExample: View {
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            // Text styles ที่ scale ตาม Dynamic Type
            Text("Large Title").font(.largeTitle)
            Text("Title 1").font(.title)
            Text("Title 2").font(.title2)
            Text("Title 3").font(.title3)
            Text("Headline").font(.headline)
            Text("Body").font(.body)
            Text("Callout").font(.callout)
            Text("Subheadline").font(.subheadline)
            Text("Footnote").font(.footnote)
            Text("Caption 1").font(.caption)
            Text("Caption 2").font(.caption2)
        }
    }
}
```

### 13.3 Custom Font ที่ Scale ได้

```swift
struct ScaledFontExample: View {
    @ScaledMetric var iconSize: CGFloat = 24
    @ScaledMetric(relativeTo: .body) var padding: CGFloat = 16
    
    var body: some View {
        HStack(spacing: padding) {
            Image(systemName: "star.fill")
                .font(.system(size: iconSize))
            
            Text("ชื่นชอบ")
                // Font ที่ scale ตาม Dynamic Type
                .font(.system(.body, design: .default))
        }
        .padding(padding)
    }
}

// Custom font ที่ support Dynamic Type
struct CustomScaledFont: View {
    var body: some View {
        Text("Custom Font")
            .font(Font.custom("Helvetica", size: 17, relativeTo: .body))
        // relativeTo ทำให้ scale ตาม body text style
    }
}
```

---

## 14. Font Scaling

### 14.1 ตรวจสอบขนาด Font ปัจจุบัน

```swift
struct FontScalingExample: View {
    @Environment(\.sizeCategory) var sizeCategory
    
    var body: some View {
        VStack {
            Text("ขนาดปัจจุบัน: \(sizeCategory.name)")
            
            // ปรับ layout ตามขนาด
            if sizeCategory.isAccessibilityCategory {
                // Layout สำหรับ accessibility sizes (XXXL)
                VStack(alignment: .leading) {
                    Image(systemName: "star.fill")
                        .font(.largeTitle)
                    Text("รายการโปรด")
                        .font(.headline)
                }
            } else {
                // Layout ปกติ
                HStack {
                    Image(systemName: "star.fill")
                    Text("รายการโปรด")
                        .font(.headline)
                }
            }
        }
    }
}

extension ContentSizeCategory {
    var name: String {
        switch self {
        case .extraSmall: return "Extra Small"
        case .small: return "Small"
        case .medium: return "Medium"
        case .large: return "Large (Default)"
        case .extraLarge: return "Extra Large"
        case .extraExtraLarge: return "Extra Extra Large"
        case .extraExtraExtraLarge: return "Extra Extra Extra Large"
        case .accessibilityMedium: return "Accessibility M"
        case .accessibilityLarge: return "Accessibility L"
        case .accessibilityExtraLarge: return "Accessibility XL"
        case .accessibilityExtraExtraLarge: return "Accessibility XXL"
        case .accessibilityExtraExtraExtraLarge: return "Accessibility XXXL"
        @unknown default: return "Unknown"
        }
    }
    
    var isAccessibilityCategory: Bool {
        switch self {
        case .accessibilityMedium, .accessibilityLarge,
             .accessibilityExtraLarge, .accessibilityExtraExtraLarge,
             .accessibilityExtraExtraExtraLarge:
            return true
        default:
            return false
        }
    }
}
```

### 14.2 Layout ที่ Adaptive กับ Dynamic Type

```swift
struct AdaptiveLayoutForDynamicType: View {
    @Environment(\.sizeCategory) var sizeCategory
    @ScaledMetric var thumbnailSize: CGFloat = 60
    
    var body: some View {
        // ViewThatFits ช่วย switch layout อัตโนมัติ
        ViewThatFits {
            // Horizontal layout (สำหรับขนาดปกติ)
            HStack {
                Image(systemName: "photo")
                    .resizable()
                    .frame(width: thumbnailSize, height: thumbnailSize)
                
                VStack(alignment: .leading) {
                    Text("ชื่อรูปภาพ")
                        .font(.headline)
                    Text("รายละเอียด")
                        .font(.subheadline)
                        .foregroundColor(.secondary)
                }
                
                Spacer()
                
                Button("แก้ไข") {}
            }
            
            // Vertical layout (สำหรับ accessibility sizes)
            VStack(alignment: .leading) {
                Image(systemName: "photo")
                    .resizable()
                    .frame(width: thumbnailSize, height: thumbnailSize)
                
                Text("ชื่อรูปภาพ")
                    .font(.headline)
                Text("รายละเอียด")
                    .font(.subheadline)
                    .foregroundColor(.secondary)
                
                Button("แก้ไข") {}
                    .frame(maxWidth: .infinity)
            }
        }
        .padding()
    }
}
```

---

## 15. Minimum Touch Target Size

### 15.1 ขนาด Touch Target ที่แนะนำ

Apple แนะนำขนาด minimum touch target ที่ 44x44 points เพื่อให้ผู้ใช้ทุกคนสามารถแตะได้ง่าย โดยเฉพาะผู้ที่มีปัญหาด้านการเคลื่อนไหว

### 15.2 วิธีปรับขนาด Touch Target

```swift
struct TouchTargetExamples: View {
    var body: some View {
        VStack(spacing: 20) {
            // 1. ปุ่มขนาดเล็กแต่ touch target ใหญ่
            Button(action: {}) {
                Image(systemName: "xmark")
                    .font(.caption)
            }
            .frame(minWidth: 44, minHeight: 44)
            // หรือใช้ contentShape
            
            // 2. ใช้ contentShape เพื่อขยาย touch area
            Button(action: {}) {
                Image(systemName: "xmark")
                    .font(.caption)
                    .padding(10) // เพิ่ม padding เพื่อขยาย touch area
            }
            
            // 3. Hit test area
            Button(action: {}) {
                Text("X")
                    .font(.caption)
            }
            .contentShape(Rectangle())
            .frame(minWidth: 44, minHeight: 44)
            
            // 4. Custom Button Style ที่มี minimum size
            Button("บันทึก") {}
                .buttonStyle(AccessibleButtonStyle())
        }
    }
}

struct AccessibleButtonStyle: ButtonStyle {
    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .padding(.horizontal, 16)
            .padding(.vertical, 12)
            .frame(minWidth: 44, minHeight: 44)
            .background(Color.blue)
            .foregroundColor(.white)
            .cornerRadius(8)
            .scaleEffect(configuration.isPressed ? 0.95 : 1.0)
    }
}

// UIKit version
class TouchTargetViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let smallButton = UIButton()
        smallButton.setImage(UIImage(systemName: "xmark"), for: .normal)
        
        // เพิ่ม padding ให้ touch area ใหญ่ขึ้น
        smallButton.contentEdgeInsets = UIEdgeInsets(top: 12, left: 12, bottom: 12, right: 12)
        
        // หรือ override hitTest
        view.addSubview(smallButton)
    }
}

// Custom UIButton ที่มี minimum touch target
class MinTouchTargetButton: UIButton {
    let minimumTargetSize: CGFloat = 44
    
    override func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView? {
        // ขยาย hit test area ถ้า button เล็กกว่า minimum
        if isHidden || !isUserInteractionEnabled || alpha < 0.01 {
            return nil
        }
        
        let expandedBounds = bounds.insetBy(
            dx: min(0, (bounds.width - minimumTargetSize) / 2),
            dy: min(0, (bounds.height - minimumTargetSize) / 2)
        )
        
        return expandedBounds.contains(point) ? self : nil
    }
}
```

---

## 16. Color Contrast

### 16.1 ความสำคัญของ Color Contrast

WCAG (Web Content Accessibility Guidelines) กำหนดอัตราส่วน contrast ขั้นต่ำ:
- **Level AA**: ข้อความปกติ 4.5:1, ข้อความใหญ่ 3:1
- **Level AAA**: ข้อความปกติ 7:1, ข้อความใหญ่ 4.5:1

### 16.2 ตัวอย่างการตรวจสอบ Contrast

```swift
struct ColorContrastExample: View {
    var body: some View {
        VStack(spacing: 20) {
            // ❌ ไม่ผ่าน: contrast ต่ำ
            Text("ข้อความสีจาง")
                .foregroundColor(Color(red: 0.7, green: 0.7, blue: 0.7))
                .background(Color.white)
                .padding()
            
            // ✅ ผ่าน: contrast ดี
            Text("ข้อความสีเข้ม")
                .foregroundColor(Color(red: 0.1, green: 0.1, blue: 0.1))
                .background(Color.white)
                .padding()
            
            // ✅ ใช้ semantic colors ที่ปรับตาม dark mode อัตโนมัติ
            Text("ข้อความ Primary")
                .foregroundColor(.primary)
                .background(Color(.systemBackground))
                .padding()
            
            // ✅ ใช้ adaptive colors
            AdaptiveColorTextView()
        }
    }
}

struct AdaptiveColorTextView: View {
    @Environment(\.colorScheme) var colorScheme
    
    var textColor: Color {
        colorScheme == .dark ? Color.white : Color.black
    }
    
    var backgroundColor: Color {
        colorScheme == .dark ? Color.black : Color.white
    }
    
    var body: some View {
        Text("Adaptive Color Text")
            .foregroundColor(textColor)
            .background(backgroundColor)
            .padding()
    }
}

// ฟังก์ชันคำนวณ contrast ratio
func contrastRatio(color1: UIColor, color2: UIColor) -> Double {
    func luminance(_ color: UIColor) -> Double {
        var r: CGFloat = 0, g: CGFloat = 0, b: CGFloat = 0
        color.getRed(&r, green: &g, blue: &b, alpha: nil)
        
        let adjust: (CGFloat) -> Double = { component in
            let c = Double(component)
            return c <= 0.04045 ? c / 12.92 : pow((c + 0.055) / 1.055, 2.4)
        }
        
        return 0.2126 * adjust(r) + 0.7152 * adjust(g) + 0.0722 * adjust(b)
    }
    
    let l1 = luminance(color1)
    let l2 = luminance(color2)
    let lighter = max(l1, l2)
    let darker = min(l1, l2)
    return (lighter + 0.05) / (darker + 0.05)
}
```

---

## 17. Motion Reduction

### 17.1 Reduce Motion คืออะไร

ผู้ใช้บางคนมีอาการ vestibular disorder (ความผิดปกติของหูชั้นใน) ที่ทำให้การเคลื่อนไหวบนหน้าจอทำให้รู้สึกเวียนศีรษะ iOS มี "Reduce Motion" option ที่ช่วยให้แอปลดการ animate

### 17.2 การตรวจสอบ Reduce Motion

```swift
struct ReduceMotionExample: View {
    @Environment(\.accessibilityReduceMotion) var reduceMotion
    @State private var isExpanded = false
    
    var body: some View {
        VStack {
            Button("Toggle") {
                if reduceMotion {
                    // ไม่มี animation
                    isExpanded.toggle()
                } else {
                    withAnimation(.spring()) {
                        isExpanded.toggle()
                    }
                }
            }
            
            if isExpanded {
                Text("เนื้อหาที่ซ่อนอยู่")
                    .transition(reduceMotion ? .opacity : .slide)
            }
        }
    }
}

// Extension สำหรับ conditional animation
extension View {
    func animationIfAllowed<V: Equatable>(_ animation: Animation?, value: V) -> some View {
        modifier(AnimationIfAllowedModifier(animation: animation, value: value))
    }
}

struct AnimationIfAllowedModifier<V: Equatable>: ViewModifier {
    @Environment(\.accessibilityReduceMotion) var reduceMotion
    let animation: Animation?
    let value: V
    
    func body(content: Content) -> some View {
        if reduceMotion {
            content.animation(nil, value: value)
        } else {
            content.animation(animation, value: value)
        }
    }
}

// ใช้งาน
struct AnimationExample: View {
    @State private var scale: CGFloat = 1.0
    
    var body: some View {
        Circle()
            .scaleEffect(scale)
            .animationIfAllowed(.spring(), value: scale)
            .onTapGesture {
                scale = scale == 1.0 ? 1.5 : 1.0
            }
    }
}

// UIKit version
class MotionViewController: UIViewController {
    func animate() {
        let reduceMotion = UIAccessibility.isReduceMotionEnabled
        
        if reduceMotion {
            // ไม่มี animation หรือใช้ fade แทน
            UIView.animate(withDuration: 0.2) {
                // minimal animation
            }
        } else {
            UIView.animate(withDuration: 0.5, delay: 0, usingSpringWithDamping: 0.7,
                          initialSpringVelocity: 0.5) {
                // full animation
            }
        }
    }
}
```

---

## 18. Reduced Transparency

### 18.1 Reduce Transparency คืออะไร

ผู้ใช้บางคนพบว่า transparency effects เช่น blur background ทำให้อ่านยาก iOS มี "Reduce Transparency" option ที่ทำให้ background ทึบขึ้น

### 18.2 การตรวจสอบ Reduce Transparency

```swift
struct ReduceTransparencyExample: View {
    @Environment(\.accessibilityReduceTransparency) var reduceTransparency
    
    var body: some View {
        ZStack {
            // Background
            LinearGradient(colors: [.blue, .purple], startPoint: .top, endPoint: .bottom)
                .ignoresSafeArea()
            
            // Content card
            VStack(spacing: 16) {
                Text("เนื้อหาบัตร")
                    .font(.title)
                Button("ดำเนินการ") {}
            }
            .padding()
            .background {
                if reduceTransparency {
                    // ใช้ background ทึบ
                    Color(.systemBackground)
                } else {
                    // ใช้ blur effect
                    Rectangle()
                        .fill(.ultraThinMaterial)
                }
            }
            .cornerRadius(16)
        }
    }
}

// UIKit version
class TransparencyViewController: UIViewController {
    let blurView: UIVisualEffectView = {
        let effect = UIBlurEffect(style: .systemMaterial)
        return UIVisualEffectView(effect: effect)
    }()
    
    let solidView: UIView = {
        let view = UIView()
        view.backgroundColor = .systemBackground
        return view
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        updateBackground()
        
        // ฟัง notification เมื่อ setting เปลี่ยน
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(updateBackground),
            name: UIAccessibility.reduceTransparencyStatusDidChangeNotification,
            object: nil
        )
    }
    
    @objc func updateBackground() {
        if UIAccessibility.isReduceTransparencyEnabled {
            blurView.removeFromSuperview()
            view.addSubview(solidView)
        } else {
            solidView.removeFromSuperview()
            view.addSubview(blurView)
        }
    }
}
```

---

## 19. Bold Text

### 19.1 Bold Text คืออะไร

ผู้ใช้บางคนต้องการข้อความตัวหนาเพื่ออ่านได้ง่ายขึ้น iOS มี "Bold Text" option ที่ทำให้ระบบ fonts ทุกตัวกลายเป็น bold

### 19.2 การตรวจสอบ Bold Text

```swift
struct BoldTextExample: View {
    @Environment(\.accessibilityBoldText) var boldText
    
    var body: some View {
        VStack(spacing: 16) {
            // Text ที่ปรับตาม Bold Text setting
            Text("หัวข้อ")
                .font(.system(.title, design: .default, weight: boldText ? .black : .bold))
            
            Text("เนื้อหา")
                .font(.system(.body, design: .default, weight: boldText ? .semibold : .regular))
            
            // Custom font ที่ปรับตาม Bold Text
            Text("Custom Font")
                .font(boldText ? 
                      Font.custom("YourFont-Bold", size: 17) : 
                      Font.custom("YourFont-Regular", size: 17))
        }
    }
}

// UIKit
class BoldTextViewController: UIViewController {
    let label: UILabel = {
        let label = UILabel()
        label.font = UIFont.preferredFont(forTextStyle: .body)
        label.adjustsFontForContentSizeCategory = true
        return label
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // ฟัง notification
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(updateFont),
            name: UIAccessibility.boldTextStatusDidChangeNotification,
            object: nil
        )
    }
    
    @objc func updateFont() {
        if UIAccessibility.isBoldTextEnabled {
            label.font = UIFont.systemFont(ofSize: 17, weight: .bold)
        } else {
            label.font = UIFont.systemFont(ofSize: 17, weight: .regular)
        }
    }
}
```

---

## 20. Switch Control

### 20.1 Switch Control คืออะไร

Switch Control เป็น accessibility feature สำหรับผู้ที่มีข้อจำกัดด้านการเคลื่อนไหว ที่ไม่สามารถใช้ touchscreen ได้โดยตรง ผู้ใช้จะใช้อุปกรณ์ switch หนึ่งหรือหลายตัวในการควบคุมแอป

### 20.2 วิธีการทำงาน

Switch Control ทำงานด้วย auto-scanning หรือ manual scanning:
- **Auto-scanning**: ระบบจะ highlight elements ทีละตัวอัตโนมัติ ผู้ใช้กด switch เมื่อ highlight ถึง element ที่ต้องการ
- **Manual scanning**: ผู้ใช้กด switch เพื่อ advance และ switch อีกตัวเพื่อ select

### 20.3 การออกแบบสำหรับ Switch Control

```swift
struct SwitchControlFriendlyView: View {
    @State private var selectedItem: String? = nil
    
    let menuItems = ["หน้าหลัก", "ค้นหา", "รายการโปรด", "การตั้งค่า"]
    
    var body: some View {
        VStack(spacing: 0) {
            // 1. จัดกลุ่ม elements ที่เกี่ยวข้องกัน
            // Switch Control จะ scan ทีละ group
            ForEach(menuItems, id: \.self) { item in
                Button(item) {
                    selectedItem = item
                }
                .frame(maxWidth: .infinity, minHeight: 60)
                // หลีกเลี่ยง overlapping elements ที่ทำให้ scan ยาก
                .contentShape(Rectangle())
                Divider()
            }
            
            // 2. ลำดับ scanning ที่สมเหตุสมผล
            HStack {
                Button("ก่อนหน้า") {}
                Spacer()
                Button("ถัดไป") {}
            }
            .padding()
        }
    }
}

// UIKit - กำหนด Switch Control group
class SwitchControlViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Group ปุ่มที่เกี่ยวข้องกัน
        let buttonGroup = UIView()
        buttonGroup.accessibilityContainerType = .semanticGroup
        // Switch Control จะ scan ทั้ง group นี้เป็น unit
    }
}
```

---

## 21. Voice Control

### 21.1 Voice Control คืออะไร

Voice Control ช่วยให้ผู้ใช้สามารถควบคุม iOS ด้วยเสียง โดยพูดชื่อปุ่มหรือ elements เพื่อกด

### 21.2 การออกแบบสำหรับ Voice Control

```swift
struct VoiceControlFriendlyView: View {
    var body: some View {
        VStack(spacing: 20) {
            // 1. ปุ่มที่มี label ชัดเจน
            // ผู้ใช้พูดว่า "ส่งข้อความ" เพื่อกดปุ่ม
            Button("ส่งข้อความ") {}
            
            // 2. หลีกเลี่ยงชื่อซ้ำกัน ถ้าหลายปุ่มมีฟังก์ชัน "ลบ"
            // ให้มี label ที่แตกต่างกัน
            HStack {
                Button("ลบรูปที่ 1") {}
                Button("ลบรูปที่ 2") {}
            }
            
            // 3. Icon-only buttons ต้องมี accessibilityLabel
            Button(action: {}) {
                Image(systemName: "mic")
            }
            .accessibilityLabel("บันทึกเสียง")
            // ผู้ใช้พูดว่า "บันทึกเสียง" เพื่อกดปุ่ม
            
            // 4. ใช้ accessibilityUserInputLabels เพื่อกำหนด alternative names
            Button("ยืนยัน") {}
                .accessibilityInputLabels(["ยืนยัน", "ตกลง", "OK", "Confirm"])
            // ผู้ใช้สามารถพูดได้หลายแบบ
        }
        .padding()
    }
}
```

---

## 22. Display Accommodations

### 22.1 ประเภทของ Display Accommodations

```swift
struct DisplayAccommodationsExample: View {
    @Environment(\.colorScheme) var colorScheme
    @Environment(\.accessibilityDifferentiateWithoutColor) var differentiateWithoutColor
    @Environment(\.accessibilityInvertColors) var invertColors
    
    var body: some View {
        VStack(spacing: 16) {
            // 1. Dark/Light Mode
            Text("ข้อความที่ปรับตาม Dark Mode")
                .foregroundColor(.primary)
                .background(Color(.systemBackground))
            
            // 2. Differentiate Without Color
            // ผู้ใช้ที่ตาบอดสีต้องการการแยกแยะที่ไม่ใช่แค่สี
            StatusIndicator(status: .success)
            
            // 3. Smart Invert Colors
            // App ควรทำเครื่องหมาย images ที่ไม่ควร invert
            Image(systemName: "photo")
                .resizable()
                .frame(width: 100, height: 100)
                .accessibilityIgnoresInvertColors() // ไม่ invert รูปภาพ
        }
    }
}

struct StatusIndicator: View {
    enum Status { case success, warning, error }
    
    let status: Status
    
    @Environment(\.accessibilityDifferentiateWithoutColor) var differentiateWithoutColor
    
    var statusColor: Color {
        switch status {
        case .success: return .green
        case .warning: return .orange
        case .error: return .red
        }
    }
    
    var statusIcon: String {
        switch status {
        case .success: return "checkmark.circle.fill"
        case .warning: return "exclamationmark.triangle.fill"
        case .error: return "xmark.circle.fill"
        }
    }
    
    var statusText: String {
        switch status {
        case .success: return "สำเร็จ"
        case .warning: return "คำเตือน"
        case .error: return "ผิดพลาด"
        }
    }
    
    var body: some View {
        HStack {
            // ใช้ทั้ง icon และ text เมื่อ differentiateWithoutColor = true
            if differentiateWithoutColor {
                Image(systemName: statusIcon)
                Text(statusText)
            } else {
                Circle()
                    .fill(statusColor)
                    .frame(width: 12, height: 12)
                Text(statusText)
            }
        }
        .foregroundColor(statusColor)
    }
}
```

---

## 23. UIAccessibility Notifications

### 23.1 ประเภทของ Notifications

```swift
// Notifications ที่สำคัญ:
// UIAccessibility.layoutChangedNotification   - layout เปลี่ยนแปลง
// UIAccessibility.screenChangedNotification   - หน้าจอเปลี่ยนไปทั้งหมด
// UIAccessibility.announcementNotification    - ประกาศข้อความ
// UIAccessibility.pageScrolledNotification    - scroll เสร็จ
```

### 23.2 การส่ง Notifications

```swift
import UIKit
import SwiftUI

// UIKit
class NotificationViewController: UIViewController {
    @IBOutlet weak var statusLabel: UILabel!
    
    func updateStatus(message: String) {
        statusLabel.text = message
        
        // ประกาศให้ VoiceOver รู้ว่า layout เปลี่ยน
        UIAccessibility.post(
            notification: .layoutChanged,
            argument: statusLabel // VoiceOver จะ focus บน element นี้
        )
    }
    
    func showNewScreen() {
        // เมื่อเปิดหน้าจอใหม่
        UIAccessibility.post(
            notification: .screenChanged,
            argument: "หน้ารายการสินค้า" // หรือ element ที่ต้องการ focus
        )
    }
    
    func announceMessage(_ message: String) {
        // ประกาศข้อความที่สำคัญ
        UIAccessibility.post(
            notification: .announcement,
            argument: message
        )
    }
    
    func loadData() {
        // แสดง loading indicator
        announceMessage("กำลังโหลดข้อมูล")
        
        // เมื่อโหลดเสร็จ
        DispatchQueue.main.async {
            self.announceMessage("โหลดข้อมูลเสร็จแล้ว \(42) รายการ")
            UIAccessibility.post(notification: .layoutChanged, argument: nil)
        }
    }
}

// SwiftUI - ใช้ AccessibilityNotification
struct SwiftUINotificationExample: View {
    @State private var items: [String] = []
    @State private var isLoading = false
    @AccessibilityFocusState private var focusedElement: Bool
    
    var body: some View {
        VStack {
            if isLoading {
                ProgressView("กำลังโหลด...")
            } else {
                List(items, id: \.self) { item in
                    Text(item)
                }
            }
            
            Button("โหลดข้อมูล") {
                loadData()
            }
            .accessibilityFocused($focusedElement)
        }
    }
    
    func loadData() {
        isLoading = true
        
        // จำลองการโหลดข้อมูล
        DispatchQueue.main.asyncAfter(deadline: .now() + 2) {
            items = ["รายการ 1", "รายการ 2", "รายการ 3"]
            isLoading = false
            
            // ประกาศว่าโหลดเสร็จ
            UIAccessibility.post(
                notification: .announcement,
                argument: "โหลดข้อมูลเสร็จแล้ว \(items.count) รายการ"
            )
        }
    }
}
```

---

## 24. Testing with Accessibility

### 24.1 Manual Testing

```swift
// วิธีทดสอบ:
// 1. เปิด VoiceOver ใน Settings > Accessibility > VoiceOver
// 2. ใช้ Accessibility Inspector ใน Xcode
// 3. ทดสอบกับ Dynamic Type ขนาดต่างๆ
// 4. ทดสอบ Dark Mode
// 5. ทดสอบ Reduce Motion
// 6. ทดสอบ Bold Text
```

### 24.2 UI Testing สำหรับ Accessibility

```swift
import XCTest

class AccessibilityUITests: XCTestCase {
    let app = XCUIApplication()
    
    override func setUpWithError() throws {
        continueAfterFailure = false
        app.launch()
    }
    
    func testAccessibilityLabels() throws {
        // ตรวจสอบว่าปุ่มมี accessibility label
        let sendButton = app.buttons["ส่งข้อความ"]
        XCTAssertTrue(sendButton.exists, "ปุ่มส่งข้อความควรมี accessibility label")
        XCTAssertTrue(sendButton.isEnabled, "ปุ่มส่งข้อความควรสามารถใช้งานได้")
    }
    
    func testVoiceOverNavigation() throws {
        // ทดสอบการนำทางด้วย VoiceOver
        let firstElement = app.staticTexts.firstMatch
        XCTAssertTrue(firstElement.exists)
        
        // ตรวจสอบว่า elements มี accessibility ที่ถูกต้อง
        let allButtons = app.buttons.allElementsBoundByIndex
        for button in allButtons {
            XCTAssertFalse(
                button.label.isEmpty,
                "ปุ่ม '\(button.identifier)' ไม่มี accessibility label"
            )
        }
    }
    
    func testAccessibilityHierarchy() throws {
        // ตรวจสอบ accessibility hierarchy
        let productCell = app.cells["ProductCell"]
        XCTAssertTrue(productCell.exists)
        
        // ตรวจสอบว่า cell มีข้อมูลที่จำเป็น
        XCTAssertFalse(productCell.label.isEmpty)
    }
    
    func testDynamicType() throws {
        // ทดสอบกับ accessibility text size
        app.launchEnvironment["UIContentSizeCategory"] = "UICTContentSizeCategoryAccessibilityXXXL"
        app.launch()
        
        // ตรวจสอบว่า layout ยังใช้งานได้
        let mainContent = app.staticTexts.firstMatch
        XCTAssertTrue(mainContent.exists)
        XCTAssertTrue(mainContent.isHittable)
    }
    
    func testMinimumTouchTargets() throws {
        // ตรวจสอบขนาด touch target
        let allButtons = app.buttons.allElementsBoundByIndex
        for button in allButtons {
            let frame = button.frame
            XCTAssertGreaterThanOrEqual(
                frame.width, 44,
                "ปุ่ม '\(button.label)' กว้างไม่พอ: \(frame.width)"
            )
            XCTAssertGreaterThanOrEqual(
                frame.height, 44,
                "ปุ่ม '\(button.label)' สูงไม่พอ: \(frame.height)"
            )
        }
    }
}
```

### 24.3 Unit Testing สำหรับ Accessibility

```swift
import XCTest
@testable import MyApp

class AccessibilityUnitTests: XCTestCase {
    
    func testAccessibilityLabelGeneration() {
        let product = Product(name: "เสื้อยืด", price: 299, rating: 4.5)
        let accessibilityLabel = product.accessibilityLabel
        
        XCTAssertTrue(accessibilityLabel.contains("เสื้อยืด"))
        XCTAssertTrue(accessibilityLabel.contains("299"))
        XCTAssertTrue(accessibilityLabel.contains("4.5"))
    }
    
    func testContrastRatio() {
        let textColor = UIColor.label
        let backgroundColor = UIColor.systemBackground
        
        let ratio = contrastRatio(color1: textColor, color2: backgroundColor)
        XCTAssertGreaterThanOrEqual(ratio, 4.5, "Contrast ratio ต่ำกว่า WCAG AA")
    }
}

struct Product {
    let name: String
    let price: Int
    let rating: Double
    
    var accessibilityLabel: String {
        "สินค้า: \(name), ราคา \(price) บาท, คะแนน \(rating) ดาว"
    }
}
```

---

## 25. แบบฝึกหัดพร้อมเฉลย

### แบบฝึกหัดที่ 1: เพิ่ม Accessibility ให้กับ Profile Card

**โจทย์:** เพิ่ม accessibility ที่เหมาะสมให้กับ Profile Card ด้านล่าง

```swift
// โจทย์ - เพิ่ม accessibility ให้ view นี้
struct ProfileCardWithoutAccessibility: View {
    let user: UserProfile
    
    var body: some View {
        HStack {
            Image(user.avatarName)
                .resizable()
                .frame(width: 60, height: 60)
                .clipShape(Circle())
            
            VStack(alignment: .leading) {
                Text(user.name)
                    .font(.headline)
                Text(user.title)
                    .font(.subheadline)
                    .foregroundColor(.secondary)
                Text("\(user.followers) followers")
                    .font(.caption)
            }
            
            Spacer()
            
            Button(action: {}) {
                Image(systemName: "person.badge.plus")
            }
        }
        .padding()
    }
}

struct UserProfile {
    let name: String
    let title: String
    let avatarName: String
    let followers: Int
    let isFollowing: Bool
}
```

**เฉลย:**

```swift
struct ProfileCardWithAccessibility: View {
    let user: UserProfile
    @State private var isFollowing: Bool
    
    init(user: UserProfile) {
        self.user = user
        self._isFollowing = State(initialValue: user.isFollowing)
    }
    
    var body: some View {
        HStack {
            // รูปโปรไฟล์ - ไม่ต้องอ่านถ้าชื่อ user อยู่ใน label
            Image(user.avatarName)
                .resizable()
                .frame(width: 60, height: 60)
                .clipShape(Circle())
                .accessibilityHidden(true)
            
            VStack(alignment: .leading) {
                Text(user.name)
                    .font(.headline)
                Text(user.title)
                    .font(.subheadline)
                    .foregroundColor(.secondary)
                Text("\(user.followers) followers")
                    .font(.caption)
            }
            
            Spacer()
            
            // ปุ่ม Follow/Unfollow
            Button(action: { isFollowing.toggle() }) {
                Image(systemName: isFollowing ? "person.badge.checkmark" : "person.badge.plus")
            }
            .accessibilityLabel(isFollowing ? "เลิกติดตาม \(user.name)" : "ติดตาม \(user.name)")
            .accessibilityHint(isFollowing ? "แตะสองครั้งเพื่อเลิกติดตาม" : "แตะสองครั้งเพื่อติดตาม")
            .frame(minWidth: 44, minHeight: 44)
        }
        .padding()
        // รวม card เป็น element เดียว
        .accessibilityElement(children: .combine)
        .accessibilityLabel("\(user.name), \(user.title)")
        .accessibilityValue("\(user.followers) ผู้ติดตาม, \(isFollowing ? "กำลังติดตาม" : "ยังไม่ติดตาม")")
        .accessibilityHint("แตะสองครั้งเพื่อดูโปรไฟล์")
    }
}
```

### แบบฝึกหัดที่ 2: Accessible Form

**โจทย์:** สร้าง Login Form ที่มี accessibility ที่ดี

**เฉลย:**

```swift
struct AccessibleLoginForm: View {
    @State private var email = ""
    @State private var password = ""
    @State private var showPassword = false
    @State private var isLoading = false
    @State private var errorMessage = ""
    
    var body: some View {
        VStack(spacing: 24) {
            // Header
            Text("เข้าสู่ระบบ")
                .font(.largeTitle)
                .bold()
                .accessibilityAddTraits(.isHeader)
            
            // Email Field
            VStack(alignment: .leading, spacing: 4) {
                Text("อีเมล")
                    .font(.caption)
                    .foregroundColor(.secondary)
                
                TextField("กรอกอีเมลของคุณ", text: $email)
                    .textContentType(.emailAddress)
                    .keyboardType(.emailAddress)
                    .autocapitalization(.none)
                    .accessibilityLabel("อีเมล")
                    .accessibilityHint("กรอกอีเมลที่ใช้ลงทะเบียน")
            }
            
            // Password Field
            VStack(alignment: .leading, spacing: 4) {
                Text("รหัสผ่าน")
                    .font(.caption)
                    .foregroundColor(.secondary)
                
                HStack {
                    if showPassword {
                        TextField("กรอกรหัสผ่าน", text: $password)
                    } else {
                        SecureField("กรอกรหัสผ่าน", text: $password)
                    }
                    
                    Button(action: { showPassword.toggle() }) {
                        Image(systemName: showPassword ? "eye.slash" : "eye")
                    }
                    .accessibilityLabel(showPassword ? "ซ่อนรหัสผ่าน" : "แสดงรหัสผ่าน")
                    .frame(minWidth: 44, minHeight: 44)
                }
                .accessibilityElement(children: .contain)
            }
            
            // Error Message
            if !errorMessage.isEmpty {
                Text(errorMessage)
                    .foregroundColor(.red)
                    .font(.caption)
                    .accessibilityAddTraits(.isStaticText)
                    .onAppear {
                        UIAccessibility.post(notification: .announcement, argument: errorMessage)
                    }
            }
            
            // Login Button
            Button(action: { login() }) {
                if isLoading {
                    ProgressView()
                        .progressViewStyle(CircularProgressViewStyle(tint: .white))
                } else {
                    Text("เข้าสู่ระบบ")
                }
            }
            .frame(maxWidth: .infinity, minHeight: 50)
            .background(Color.blue)
            .foregroundColor(.white)
            .cornerRadius(12)
            .accessibilityLabel("เข้าสู่ระบบ")
            .accessibilityHint(isLoading ? "กำลังดำเนินการ" : "แตะสองครั้งเพื่อเข้าสู่ระบบ")
            .disabled(email.isEmpty || password.isEmpty || isLoading)
        }
        .padding()
    }
    
    func login() {
        guard !email.isEmpty, !password.isEmpty else {
            errorMessage = "กรุณากรอกอีเมลและรหัสผ่าน"
            return
        }
        
        isLoading = true
        UIAccessibility.post(notification: .announcement, argument: "กำลังเข้าสู่ระบบ")
        
        // จำลองการ login
        DispatchQueue.main.asyncAfter(deadline: .now() + 2) {
            isLoading = false
            // สมมติว่าสำเร็จ
            UIAccessibility.post(notification: .announcement, argument: "เข้าสู่ระบบสำเร็จ")
        }
    }
}
```

---

## 26. Making a Complete App Accessible

### 26.1 ตัวอย่าง: แอป Shopping List ที่ Accessible ครบถ้วน

```swift
import SwiftUI

// Model
struct ShoppingItem: Identifiable {
    let id = UUID()
    var name: String
    var quantity: Int
    var isChecked: Bool
    var category: String
    
    var accessibilityLabel: String {
        "\(name), \(quantity) ชิ้น, \(category)"
    }
    
    var accessibilityValue: String {
        isChecked ? "ซื้อแล้ว" : "ยังไม่ได้ซื้อ"
    }
}

// Main View
struct AccessibleShoppingListApp: View {
    @State private var items: [ShoppingItem] = [
        ShoppingItem(name: "แอปเปิ้ล", quantity: 3, isChecked: false, category: "ผลไม้"),
        ShoppingItem(name: "นม", quantity: 2, isChecked: true, category: "เครื่องดื่ม"),
        ShoppingItem(name: "ขนมปัง", quantity: 1, isChecked: false, category: "เบเกอรี่")
    ]
    @State private var showingAddItem = false
    @State private var searchText = ""
    
    var filteredItems: [ShoppingItem] {
        if searchText.isEmpty {
            return items
        }
        return items.filter { $0.name.contains(searchText) }
    }
    
    var checkedCount: Int {
        items.filter { $0.isChecked }.count
    }
    
    var body: some View {
        NavigationView {
            VStack {
                // Progress summary
                HStack {
                    Text("ซื้อแล้ว \(checkedCount) จาก \(items.count) รายการ")
                        .font(.subheadline)
                        .foregroundColor(.secondary)
                    Spacer()
                }
                .padding(.horizontal)
                .accessibilityLabel("ความคืบหน้า: ซื้อแล้ว \(checkedCount) จาก \(items.count) รายการ")
                
                // Search
                SearchBar(text: $searchText)
                    .accessibilityLabel("ค้นหารายการ")
                    .accessibilityHint("พิมพ์เพื่อกรองรายการ")
                
                // List
                List {
                    ForEach($items) { $item in
                        if filteredItems.contains(where: { $0.id == item.id }) {
                            ShoppingItemRow(item: $item)
                        }
                    }
                    .onDelete(perform: deleteItems)
                }
                .listStyle(.plain)
            }
            .navigationTitle("รายการซื้อของ")
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button(action: { showingAddItem = true }) {
                        Image(systemName: "plus")
                    }
                    .accessibilityLabel("เพิ่มรายการใหม่")
                    .frame(minWidth: 44, minHeight: 44)
                }
                
                ToolbarItem(placement: .navigationBarLeading) {
                    Button("ล้างที่ซื้อแล้ว") {
                        clearCheckedItems()
                    }
                    .disabled(checkedCount == 0)
                    .accessibilityHint("ลบรายการที่ซื้อแล้วทั้งหมดออก")
                }
            }
            .sheet(isPresented: $showingAddItem) {
                AddItemView(items: $items)
            }
        }
    }
    
    func deleteItems(at offsets: IndexSet) {
        let deletedNames = offsets.map { filteredItems[$0].name }.joined(separator: ", ")
        items.remove(atOffsets: offsets)
        UIAccessibility.post(
            notification: .announcement,
            argument: "ลบ \(deletedNames) แล้ว"
        )
    }
    
    func clearCheckedItems() {
        let count = checkedCount
        items.removeAll { $0.isChecked }
        UIAccessibility.post(
            notification: .announcement,
            argument: "ลบรายการที่ซื้อแล้ว \(count) รายการ"
        )
    }
}

// Row View
struct ShoppingItemRow: View {
    @Binding var item: ShoppingItem
    
    var body: some View {
        HStack {
            Button(action: { item.isChecked.toggle() }) {
                Image(systemName: item.isChecked ? "checkmark.circle.fill" : "circle")
                    .foregroundColor(item.isChecked ? .green : .gray)
                    .font(.title2)
            }
            .accessibilityHidden(true)
            
            VStack(alignment: .leading) {
                Text(item.name)
                    .font(.body)
                    .strikethrough(item.isChecked)
                    .foregroundColor(item.isChecked ? .secondary : .primary)
                
                HStack {
                    Text(item.category)
                        .font(.caption)
                        .foregroundColor(.secondary)
                    Text("×\(item.quantity)")
                        .font(.caption)
                        .foregroundColor(.secondary)
                }
            }
            
            Spacer()
        }
        .contentShape(Rectangle())
        .onTapGesture { item.isChecked.toggle() }
        // Accessibility
        .accessibilityElement(children: .ignore)
        .accessibilityLabel(item.accessibilityLabel)
        .accessibilityValue(item.accessibilityValue)
        .accessibilityHint("แตะสองครั้งเพื่อ\(item.isChecked ? "ยกเลิก" : "ทำเครื่องหมาย")ว่าซื้อแล้ว")
        .accessibilityActions {
            Button("เพิ่มจำนวน") {
                item.quantity += 1
                UIAccessibility.post(notification: .announcement,
                    argument: "\(item.name) \(item.quantity) ชิ้น")
            }
            Button("ลดจำนวน") {
                if item.quantity > 1 { item.quantity -= 1 }
                UIAccessibility.post(notification: .announcement,
                    argument: "\(item.name) \(item.quantity) ชิ้น")
            }
        }
    }
}

// Add Item View
struct AddItemView: View {
    @Binding var items: [ShoppingItem]
    @Environment(\.dismiss) var dismiss
    @State private var name = ""
    @State private var quantity = 1
    @State private var category = "ทั่วไป"
    @AccessibilityFocusState private var nameFieldFocused: Bool
    
    let categories = ["ผลไม้", "ผัก", "เนื้อสัตว์", "เครื่องดื่ม", "เบเกอรี่", "ทั่วไป"]
    
    var body: some View {
        NavigationView {
            Form {
                Section("ข้อมูลรายการ") {
                    TextField("ชื่อสินค้า", text: $name)
                        .accessibilityLabel("ชื่อสินค้า")
                        .accessibilityHint("กรอกชื่อสินค้าที่ต้องการซื้อ")
                        .accessibilityFocused($nameFieldFocused)
                    
                    Stepper("จำนวน: \(quantity)", value: $quantity, in: 1...99)
                        .accessibilityLabel("จำนวน")
                        .accessibilityValue("\(quantity) ชิ้น")
                        .accessibilityHint("ปรับจำนวนสินค้า")
                    
                    Picker("หมวดหมู่", selection: $category) {
                        ForEach(categories, id: \.self) { cat in
                            Text(cat).tag(cat)
                        }
                    }
                    .accessibilityLabel("หมวดหมู่สินค้า")
                }
            }
            .navigationTitle("เพิ่มรายการ")
            .toolbar {
                ToolbarItem(placement: .navigationBarLeading) {
                    Button("ยกเลิก") { dismiss() }
                }
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button("เพิ่ม") { addItem() }
                        .disabled(name.isEmpty)
                        .accessibilityHint(name.isEmpty ? "กรอกชื่อสินค้าก่อน" : "เพิ่มสินค้าลงในรายการ")
                }
            }
            .onAppear {
                DispatchQueue.main.asyncAfter(deadline: .now() + 0.5) {
                    nameFieldFocused = true
                }
            }
        }
    }
    
    func addItem() {
        let newItem = ShoppingItem(name: name, quantity: quantity, isChecked: false, category: category)
        items.append(newItem)
        UIAccessibility.post(notification: .announcement, argument: "เพิ่ม \(name) แล้ว")
        dismiss()
    }
}

struct SearchBar: View {
    @Binding var text: String
    
    var body: some View {
        HStack {
            Image(systemName: "magnifyingglass")
                .foregroundColor(.secondary)
            TextField("ค้นหา...", text: $text)
        }
        .padding(8)
        .background(Color(.systemGray6))
        .cornerRadius(10)
        .padding(.horizontal)
    }
}
```

---

## 27. สรุป

การทำ Accessibility เป็นสิ่งที่สำคัญมากในการพัฒนา iOS apps โดยสรุปสิ่งที่ได้เรียนรู้:

### สิ่งสำคัญที่ต้องจำ

1. **accessibilityLabel**: ชื่อของ element - ควรสั้น ชัดเจน สื่อความหมาย
2. **accessibilityHint**: อธิบายผลลัพธ์ของการกระทำ - ไม่ซ้ำกับ label
3. **accessibilityValue**: ค่าปัจจุบัน - สำหรับ dynamic elements
4. **accessibilityTraits**: ประเภทของ element - ช่วย VoiceOver เข้าใจการโต้ตอบ
5. **accessibilityHidden**: ซ่อน decorative elements
6. **accessibilityElement(children:)**: ควบคุม accessibility hierarchy

### Best Practices

1. ทดสอบด้วย VoiceOver จริงๆ บน device
2. ทดสอบกับ Dynamic Type ทุกขนาด
3. ใช้ Accessibility Inspector สม่ำเสมอ
4. ตรวจสอบ color contrast ratio
5. ตรวจสอบขนาด touch target (minimum 44x44 points)
6. Support Reduce Motion สำหรับ animations
7. เพิ่ม UIAccessibility notifications เมื่อ state เปลี่ยน
8. ทดสอบกับ Switch Control และ Voice Control
9. ใช้ semantic SwiftUI views เมื่อเป็นไปได้
10. ทดสอบใน dark mode และ bold text

### Checklist สำหรับ Release

- [ ] ทุก image มี accessibilityLabel หรือ accessibilityHidden
- [ ] ทุกปุ่มมี label ที่อธิบายได้
- [ ] Contrast ratio ผ่าน WCAG AA (4.5:1)
- [ ] Touch targets ≥ 44×44 points
- [ ] Support Dynamic Type ถึง Accessibility XXXL
- [ ] Animations ลดลงเมื่อ Reduce Motion เปิด
- [ ] VoiceOver navigation สมเหตุสมผล
- [ ] Form fields มี labels และ hints ที่ถูกต้อง
- [ ] Error messages ประกาศผ่าน accessibility notifications
- [ ] ผ่าน Accessibility Inspector audit

---

*จบ Part 45: Accessibility ใน iOS Development*
