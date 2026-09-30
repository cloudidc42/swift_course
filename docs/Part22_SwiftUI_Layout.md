# ตอนที่ 22: SwiftUI Layout ขั้นสูง (SwiftUI Layout)

## สารบัญ
1. [HStack, VStack, ZStack เชิงลึก](#hstack-vstack-zstack-เชิงลึก)
2. [Alignment และ Spacing](#alignment-และ-spacing)
3. [LazyHStack, LazyVStack](#lazyhstack-lazyvstack)
4. [LazyHGrid, LazyVGrid](#lazyhgrid-lazyvgrid)
5. [GridItem](#griditem)
6. [GeometryReader](#geometryreader)
7. [alignmentGuide](#alignmentguide)
8. [Custom Alignment](#custom-alignment)
9. [padding, frame, fixedSize](#padding-frame-fixedsize)
10. [edgesIgnoringSafeArea และ safeAreaInset](#edgesignoringsafearea-และ-safeareainset)
11. [overlay และ background](#overlay-และ-background)
12. [ScrollView](#scrollview)
13. [ScrollViewReader](#scrollviewreader)
14. [List พื้นฐาน](#list-พื้นฐาน)
15. [ForEach](#foreach)
16. [Section ใน List](#section-ใน-list)
17. [Dynamic Content](#dynamic-content)
18. [Spacer Behavior](#spacer-behavior)
19. [ViewThatFits](#viewthatfits)
20. [AnyLayout](#anylayout)
21. [Custom Layout Protocol](#custom-layout-protocol)
22. [แบบฝึกหัดพร้อมเฉลย](#แบบฝึกหัดพร้อมเฉลย)
23. [Real-world UI Layouts](#real-world-ui-layouts)
24. [สรุป](#สรุป)

---

## HStack, VStack, ZStack เชิงลึก

### HStack เชิงลึก

HStack จัดเรียง Views แนวนอนจากซ้ายไปขวา

```swift
struct HStackDeepDive: View {
    var body: some View {
        VStack(spacing: 24) {
            
            // พื้นฐาน
            HStack {
                Text("Item 1")
                Text("Item 2")
                Text("Item 3")
            }
            
            // กำหนด spacing
            HStack(spacing: 0) {
                ForEach(1...5, id: \.self) { i in
                    Text("\(i)")
                        .frame(maxWidth: .infinity)
                        .padding()
                        .background(i % 2 == 0 ? Color.blue : Color.orange)
                        .foregroundColor(.white)
                }
            }
            
            // HStack กับ alignment ต่างๆ
            VStack(spacing: 16) {
                // .top alignment
                HStack(alignment: .top, spacing: 12) {
                    Text("Top\nLine1\nLine2")
                        .padding(8)
                        .background(Color.blue.opacity(0.2))
                    
                    Text("Short")
                        .padding(8)
                        .background(Color.red.opacity(0.2))
                    
                    Text("Medium\nText")
                        .padding(8)
                        .background(Color.green.opacity(0.2))
                }
                
                // .center alignment (default)
                HStack(alignment: .center, spacing: 12) {
                    Text("Center\nLine1\nLine2")
                        .padding(8)
                        .background(Color.blue.opacity(0.2))
                    
                    Text("Short")
                        .padding(8)
                        .background(Color.red.opacity(0.2))
                    
                    Text("Medium\nText")
                        .padding(8)
                        .background(Color.green.opacity(0.2))
                }
                
                // .bottom alignment
                HStack(alignment: .bottom, spacing: 12) {
                    Text("Bottom\nLine1\nLine2")
                        .padding(8)
                        .background(Color.blue.opacity(0.2))
                    
                    Text("Short")
                        .padding(8)
                        .background(Color.red.opacity(0.2))
                    
                    Text("Medium\nText")
                        .padding(8)
                        .background(Color.green.opacity(0.2))
                }
                
                // .firstTextBaseline
                HStack(alignment: .firstTextBaseline, spacing: 12) {
                    Text("Large")
                        .font(.title)
                        .padding(4)
                        .background(Color.purple.opacity(0.2))
                    
                    Text("Small")
                        .font(.caption)
                        .padding(4)
                        .background(Color.pink.opacity(0.2))
                    
                    Text("Medium")
                        .font(.body)
                        .padding(4)
                        .background(Color.teal.opacity(0.2))
                }
                
                // .lastTextBaseline
                HStack(alignment: .lastTextBaseline, spacing: 12) {
                    Text("Multi\nLine\nLarge")
                        .font(.title)
                        .padding(4)
                        .background(Color.purple.opacity(0.2))
                    
                    Text("Single Small")
                        .font(.caption)
                        .padding(4)
                        .background(Color.pink.opacity(0.2))
                }
            }
        }
        .padding()
    }
}
```

### VStack เชิงลึก

```swift
struct VStackDeepDive: View {
    var body: some View {
        HStack(spacing: 24) {
            
            // .leading alignment
            VStack(alignment: .leading, spacing: 8) {
                Text("Leading Alignment")
                    .font(.headline)
                    .underline()
                Text("Short")
                Text("Medium Text")
                Text("This is a longer text")
            }
            .padding(8)
            .background(Color.blue.opacity(0.1))
            
            // .center alignment (default)
            VStack(alignment: .center, spacing: 8) {
                Text("Center")
                    .font(.headline)
                    .underline()
                Text("Short")
                Text("Medium Text")
                Text("Longer text here")
            }
            .padding(8)
            .background(Color.green.opacity(0.1))
            
            // .trailing alignment
            VStack(alignment: .trailing, spacing: 8) {
                Text("Trailing")
                    .font(.headline)
                    .underline()
                Text("Short")
                Text("Medium Text")
                Text("Even longer text")
            }
            .padding(8)
            .background(Color.red.opacity(0.1))
        }
        .padding()
    }
}
```

### ZStack เชิงลึก

```swift
struct ZStackDeepDive: View {
    var body: some View {
        VStack(spacing: 30) {
            
            // ZStack พื้นฐาน - layers ซ้อนกัน
            ZStack {
                Rectangle()
                    .fill(Color.blue)
                    .frame(width: 200, height: 150)
                
                Rectangle()
                    .fill(Color.green.opacity(0.7))
                    .frame(width: 150, height: 100)
                    .offset(x: 20, y: 20)
                
                Text("บนสุด")
                    .foregroundColor(.white)
                    .font(.headline)
            }
            
            // ZStack กับ alignment
            ZStack(alignment: .bottomTrailing) {
                // Background
                RoundedRectangle(cornerRadius: 16)
                    .fill(
                        LinearGradient(
                            colors: [.blue, .purple],
                            startPoint: .topLeading,
                            endPoint: .bottomTrailing
                        )
                    )
                    .frame(width: 200, height: 120)
                
                // Overlay badge
                Text("NEW")
                    .font(.caption)
                    .fontWeight(.bold)
                    .foregroundColor(.white)
                    .padding(.horizontal, 8)
                    .padding(.vertical, 4)
                    .background(Color.orange)
                    .cornerRadius(8)
                    .padding(8)
            }
            
            // ZStack สำหรับ overlay text บนรูปภาพ
            ZStack(alignment: .bottom) {
                // "รูปภาพ" จำลอง
                Rectangle()
                    .fill(Color.gray)
                    .frame(width: 250, height: 180)
                    .cornerRadius(12)
                
                // Gradient overlay
                LinearGradient(
                    colors: [.clear, .black.opacity(0.7)],
                    startPoint: .center,
                    endPoint: .bottom
                )
                .cornerRadius(12)
                
                // Text บน gradient
                VStack(alignment: .leading, spacing: 2) {
                    Text("ชื่อสถานที่")
                        .font(.headline)
                        .foregroundColor(.white)
                    
                    Text("กรุงเทพมหานคร, ประเทศไทย")
                        .font(.caption)
                        .foregroundColor(.white.opacity(0.8))
                }
                .frame(maxWidth: .infinity, alignment: .leading)
                .padding()
            }
        }
        .padding()
    }
}
```

---

## Alignment และ Spacing

### Alignment ใน Stack

```swift
struct AlignmentDeepDive: View {
    var body: some View {
        ScrollView {
            VStack(spacing: 30) {
                
                // Spacing ต่างๆ
                sectionTitle("Spacing Examples")
                
                VStack(alignment: .leading, spacing: 0) {
                    label("spacing: 0")
                    HStack(spacing: 0) {
                        colorBox(.red)
                        colorBox(.blue)
                        colorBox(.green)
                    }
                }
                
                VStack(alignment: .leading, spacing: 0) {
                    label("spacing: 8")
                    HStack(spacing: 8) {
                        colorBox(.red)
                        colorBox(.blue)
                        colorBox(.green)
                    }
                }
                
                VStack(alignment: .leading, spacing: 0) {
                    label("spacing: 20")
                    HStack(spacing: 20) {
                        colorBox(.red)
                        colorBox(.blue)
                        colorBox(.green)
                    }
                }
                
                VStack(alignment: .leading, spacing: 0) {
                    label("spacing: nil (default)")
                    HStack {
                        colorBox(.red)
                        colorBox(.blue)
                        colorBox(.green)
                    }
                }
            }
            .padding()
        }
    }
    
    private func sectionTitle(_ title: String) -> some View {
        Text(title)
            .font(.headline)
            .frame(maxWidth: .infinity, alignment: .leading)
    }
    
    private func label(_ text: String) -> some View {
        Text(text)
            .font(.caption)
            .foregroundColor(.gray)
    }
    
    private func colorBox(_ color: Color) -> some View {
        color
            .frame(width: 60, height: 40)
            .cornerRadius(8)
    }
}
```

### Alignment Guides

```swift
// Custom Alignment Guide
extension HorizontalAlignment {
    struct CustomCenter: AlignmentID {
        static func defaultValue(in context: ViewDimensions) -> CGFloat {
            context[HorizontalAlignment.center]
        }
    }
    
    static let customCenter = HorizontalAlignment(CustomCenter.self)
}

struct AlignmentGuideExample: View {
    var body: some View {
        VStack(alignment: .customCenter, spacing: 8) {
            Text("ข้อความสั้น")
                .alignmentGuide(.customCenter) { d in
                    d[HorizontalAlignment.center]
                }
            
            Text("ข้อความยาวขึ้นมาหน่อย")
                .alignmentGuide(.customCenter) { d in
                    d[HorizontalAlignment.center]
                }
            
            Rectangle()
                .fill(Color.blue)
                .frame(width: 2, height: 20)
                .alignmentGuide(.customCenter) { d in
                    d[HorizontalAlignment.center]
                }
        }
        .padding()
    }
}
```

---

## LazyHStack, LazyVStack

Lazy variants จะสร้าง View เฉพาะเมื่อมองเห็นใน scroll area เท่านั้น ช่วยประหยัด memory

```swift
struct LazyStackExamples: View {
    // ข้อมูลจำนวนมาก
    let items = Array(1...1000)
    
    var body: some View {
        VStack {
            // LazyVStack ใน ScrollView
            Text("LazyVStack (1000 items)")
                .font(.headline)
            
            ScrollView {
                LazyVStack(spacing: 8) {
                    ForEach(items, id: \.self) { item in
                        LazyItemView(number: item)
                    }
                }
                .padding(.horizontal)
            }
            .frame(height: 300)
        }
    }
}

struct LazyItemView: View {
    let number: Int
    
    init(number: Int) {
        self.number = number
        // นี่จะถูกเรียกเฉพาะเมื่อ item ปรากฏในหน้าจอ
        print("Creating item \(number)")
    }
    
    var body: some View {
        HStack {
            Text("#\(number)")
                .font(.headline)
                .frame(width: 60)
            
            Text("รายการที่ \(number)")
                .frame(maxWidth: .infinity, alignment: .leading)
            
            Image(systemName: "chevron.right")
                .foregroundColor(.gray)
        }
        .padding()
        .background(Color.white)
        .cornerRadius(10)
        .shadow(color: .black.opacity(0.05), radius: 2)
    }
}

// เปรียบเทียบ VStack vs LazyVStack
struct PerformanceComparison: View {
    let items = Array(1...100)
    
    var body: some View {
        TabView {
            // VStack - สร้างทุก View ตั้งแต่แรก
            ScrollView {
                VStack(spacing: 4) {
                    ForEach(items, id: \.self) { i in
                        Text("VStack Item \(i)")
                            .padding()
                            .frame(maxWidth: .infinity)
                            .background(Color.blue.opacity(0.1))
                    }
                }
            }
            .tabItem { Label("VStack", systemImage: "rectangle.stack") }
            
            // LazyVStack - สร้าง View เมื่อมองเห็น
            ScrollView {
                LazyVStack(spacing: 4) {
                    ForEach(items, id: \.self) { i in
                        Text("LazyVStack Item \(i)")
                            .padding()
                            .frame(maxWidth: .infinity)
                            .background(Color.green.opacity(0.1))
                    }
                }
            }
            .tabItem { Label("LazyVStack", systemImage: "rectangle.stack.fill") }
        }
    }
}
```

### LazyHStack ใน ScrollView

```swift
struct LazyHStackExample: View {
    let categories = ["ทั้งหมด", "อาหาร", "เทคโนโลยี", "กีฬา", "ท่องเที่ยว", "สุขภาพ", "บันเทิง", "การศึกษา"]
    @State private var selected = "ทั้งหมด"
    
    var body: some View {
        VStack(alignment: .leading) {
            Text("หมวดหมู่")
                .font(.headline)
                .padding(.horizontal)
            
            ScrollView(.horizontal, showsIndicators: false) {
                LazyHStack(spacing: 12) {
                    ForEach(categories, id: \.self) { category in
                        CategoryChip(
                            title: category,
                            isSelected: selected == category
                        ) {
                            selected = category
                        }
                    }
                }
                .padding(.horizontal)
            }
        }
    }
}

struct CategoryChip: View {
    let title: String
    let isSelected: Bool
    let action: () -> Void
    
    var body: some View {
        Button(action: action) {
            Text(title)
                .font(.subheadline)
                .fontWeight(isSelected ? .semibold : .regular)
                .foregroundColor(isSelected ? .white : .primary)
                .padding(.horizontal, 16)
                .padding(.vertical, 8)
                .background(isSelected ? Color.blue : Color.gray.opacity(0.15))
                .cornerRadius(20)
        }
    }
}
```

---

## LazyHGrid, LazyVGrid

Grid สำหรับแสดงข้อมูลในรูปแบบตาราง

### LazyVGrid (Grid แนวตั้ง)

```swift
struct LazyVGridExamples: View {
    let items = Array(1...20)
    
    // กำหนด columns
    let twoColumns = [
        GridItem(.flexible()),
        GridItem(.flexible())
    ]
    
    let threeColumns = [
        GridItem(.flexible()),
        GridItem(.flexible()),
        GridItem(.flexible())
    ]
    
    let adaptiveColumns = [
        GridItem(.adaptive(minimum: 100))
    ]
    
    @State private var columnCount = 2
    
    var currentColumns: [GridItem] {
        switch columnCount {
        case 2: return twoColumns
        case 3: return threeColumns
        default: return adaptiveColumns
        }
    }
    
    var body: some View {
        VStack {
            // Column selector
            Picker("Columns", selection: $columnCount) {
                Text("2").tag(2)
                Text("3").tag(3)
                Text("Adaptive").tag(4)
            }
            .pickerStyle(.segmented)
            .padding()
            
            ScrollView {
                LazyVGrid(columns: currentColumns, spacing: 12) {
                    ForEach(items, id: \.self) { item in
                        GridItemView(number: item)
                    }
                }
                .padding()
            }
        }
    }
}

struct GridItemView: View {
    let number: Int
    
    var body: some View {
        VStack {
            Image(systemName: "\(number <= 50 ? number : number % 50).circle.fill")
                .font(.system(size: 40))
                .foregroundColor(.blue)
            
            Text("Item \(number)")
                .font(.caption)
        }
        .frame(maxWidth: .infinity)
        .aspectRatio(1, contentMode: .fit)
        .padding(8)
        .background(Color.blue.opacity(0.1))
        .cornerRadius(12)
    }
}
```

### LazyHGrid (Grid แนวนอน)

```swift
struct LazyHGridExample: View {
    let colors: [Color] = [
        .red, .orange, .yellow, .green,
        .blue, .purple, .pink, .teal,
        .indigo, .mint, .cyan, .brown
    ]
    
    let rows = [
        GridItem(.fixed(80)),
        GridItem(.fixed(80)),
        GridItem(.fixed(80))
    ]
    
    var body: some View {
        VStack(alignment: .leading) {
            Text("สีที่ใช้บ่อย")
                .font(.headline)
                .padding(.horizontal)
            
            ScrollView(.horizontal, showsIndicators: false) {
                LazyHGrid(rows: rows, spacing: 8) {
                    ForEach(colors.indices, id: \.self) { index in
                        RoundedRectangle(cornerRadius: 12)
                            .fill(colors[index])
                            .frame(width: 80, height: 80)
                            .overlay(
                                Text("\(index + 1)")
                                    .foregroundColor(.white)
                                    .font(.headline)
                            )
                    }
                }
                .padding(.horizontal)
            }
        }
    }
}
```

---

## GridItem

GridItem กำหนดขนาดและพฤติกรรมของแต่ละ column/row ใน Grid

```swift
struct GridItemExamples: View {
    var body: some View {
        ScrollView {
            VStack(alignment: .leading, spacing: 24) {
                
                // .fixed - ขนาดคงที่
                sectionHeader("GridItem(.fixed(80))")
                LazyVGrid(columns: [
                    GridItem(.fixed(80)),
                    GridItem(.fixed(80)),
                    GridItem(.fixed(80))
                ], spacing: 8) {
                    ForEach(1...6, id: \.self) { i in
                        gridCell(i, color: .blue)
                    }
                }
                
                // .flexible - ยืดหยุ่น
                sectionHeader("GridItem(.flexible())")
                LazyVGrid(columns: [
                    GridItem(.flexible()),
                    GridItem(.flexible())
                ], spacing: 8) {
                    ForEach(1...4, id: \.self) { i in
                        gridCell(i, color: .green)
                    }
                }
                
                // .flexible กับ minimum/maximum
                sectionHeader("GridItem(.flexible(minimum: 80, maximum: 200))")
                LazyVGrid(columns: [
                    GridItem(.flexible(minimum: 80, maximum: 200)),
                    GridItem(.flexible(minimum: 80, maximum: 200)),
                    GridItem(.flexible(minimum: 80, maximum: 200))
                ], spacing: 8) {
                    ForEach(1...6, id: \.self) { i in
                        gridCell(i, color: .orange)
                    }
                }
                
                // .adaptive - ปรับจำนวน column อัตโนมัติ
                sectionHeader("GridItem(.adaptive(minimum: 80))")
                LazyVGrid(columns: [
                    GridItem(.adaptive(minimum: 80))
                ], spacing: 8) {
                    ForEach(1...10, id: \.self) { i in
                        gridCell(i, color: .purple)
                    }
                }
                
                // GridItem พร้อม spacing และ alignment
                sectionHeader("GridItem with spacing & alignment")
                LazyVGrid(columns: [
                    GridItem(.flexible(), spacing: 4, alignment: .leading),
                    GridItem(.flexible(), spacing: 4, alignment: .center),
                    GridItem(.flexible(), spacing: 4, alignment: .trailing)
                ], spacing: 8) {
                    ForEach(1...6, id: \.self) { i in
                        Text("Item \(i)")
                            .padding(8)
                            .background(Color.pink.opacity(0.2))
                            .cornerRadius(6)
                    }
                }
            }
            .padding()
        }
    }
    
    private func sectionHeader(_ title: String) -> some View {
        Text(title)
            .font(.caption)
            .foregroundColor(.gray)
            .fontDesign(.monospaced)
            .frame(maxWidth: .infinity, alignment: .leading)
    }
    
    private func gridCell(_ number: Int, color: Color) -> some View {
        Text("\(number)")
            .font(.headline)
            .foregroundColor(.white)
            .frame(maxWidth: .infinity)
            .frame(height: 60)
            .background(color)
            .cornerRadius(8)
    }
}
```

---

## GeometryReader

GeometryReader ให้เข้าถึงขนาดและตำแหน่งของ View

```swift
struct GeometryReaderExamples: View {
    var body: some View {
        VStack(spacing: 24) {
            
            // 1. รับขนาดของ parent
            GeometryReader { geometry in
                VStack {
                    Text("Width: \(Int(geometry.size.width))")
                    Text("Height: \(Int(geometry.size.height))")
                    
                    // สร้าง View ตามสัดส่วน
                    HStack(spacing: 0) {
                        Rectangle()
                            .fill(Color.blue)
                            .frame(width: geometry.size.width * 0.6)
                        
                        Rectangle()
                            .fill(Color.orange)
                            .frame(width: geometry.size.width * 0.4)
                    }
                }
            }
            .frame(height: 100)
            .background(Color.gray.opacity(0.1))
            
            // 2. Responsive layout
            GeometryReader { geo in
                if geo.size.width > 500 {
                    // Wide layout (iPad)
                    HStack {
                        Image(systemName: "ipad")
                            .font(.system(size: 60))
                        Text("iPad Layout - Wide Screen")
                            .font(.title)
                    }
                } else {
                    // Narrow layout (iPhone)
                    VStack {
                        Image(systemName: "iphone")
                            .font(.system(size: 60))
                        Text("iPhone Layout")
                            .font(.headline)
                    }
                }
            }
            .frame(height: 100)
        }
        .padding()
    }
}

// ตัวอย่างขั้นสูง: Scrolling Parallax Effect
struct ParallaxEffect: View {
    var body: some View {
        ScrollView {
            VStack(spacing: 0) {
                ForEach(0..<5) { index in
                    GeometryReader { geometry in
                        let minY = geometry.frame(in: .global).minY
                        
                        // รูปภาพที่เลื่อนแบบ parallax
                        Rectangle()
                            .fill(Color(hue: Double(index) / 5, saturation: 0.7, brightness: 0.8))
                            .frame(height: 250)
                            .offset(y: minY > 0 ? -minY * 0.3 : 0)
                            .overlay(
                                VStack {
                                    Text("Card \(index + 1)")
                                        .font(.title)
                                        .fontWeight(.bold)
                                        .foregroundColor(.white)
                                    Text("minY: \(Int(minY))")
                                        .font(.caption)
                                        .foregroundColor(.white.opacity(0.8))
                                }
                            )
                    }
                    .frame(height: 200)
                    .clipped()
                }
            }
        }
    }
}
```

---

## alignmentGuide

`alignmentGuide` ปรับตำแหน่ง alignment ของ View นั้นๆ

```swift
struct AlignmentGuideExamples: View {
    var body: some View {
        VStack(spacing: 30) {
            
            // ตัวอย่างที่ 1: ชิดขวาแบบกำหนดเอง
            VStack(alignment: .trailing) {
                Text("ข้อความสั้น")
                
                Text("ข้อความยาวกว่า")
                    .alignmentGuide(.trailing) { d in
                        d[.trailing] + 20  // เลื่อนไปทางขวา 20 points
                    }
                
                Text("ข้อความปกติ")
            }
            .padding()
            .background(Color.blue.opacity(0.1))
            
            // ตัวอย่างที่ 2: Center alignment แบบ custom
            VStack(alignment: .center) {
                Text("Header")
                    .font(.title)
                
                // เลื่อน item นี้ไปทางซ้าย
                Text("Offset Item")
                    .alignmentGuide(.center) { d in
                        d[.center] - 30
                    }
                
                Text("Normal Item")
            }
            .padding()
            .background(Color.green.opacity(0.1))
            
            // ตัวอย่างที่ 3: การสร้าง Tag layout
            TagLayoutExample()
        }
        .padding()
    }
}

// Tag Layout ด้วย alignmentGuide
struct TagLayoutExample: View {
    let tags = ["Swift", "SwiftUI", "iOS", "macOS", "Xcode", "Apple", "Developer", "Mobile"]
    
    var body: some View {
        VStack(alignment: .leading) {
            Text("Tags:")
                .font(.headline)
            
            // Wrapping tag layout
            self.generateTags()
        }
        .frame(maxWidth: .infinity, alignment: .leading)
        .padding()
        .background(Color.gray.opacity(0.1))
        .cornerRadius(12)
    }
    
    private func generateTags() -> some View {
        var width: CGFloat = 0
        var height: CGFloat = 0
        
        return ZStack(alignment: .topLeading) {
            ForEach(tags, id: \.self) { tag in
                TagView(text: tag)
                    .alignmentGuide(.leading) { d in
                        if abs(width - d.width) > UIScreen.main.bounds.width - 32 {
                            width = 0
                            height -= d.height + 8
                        }
                        let result = width
                        if tag == tags.last {
                            width = 0
                        } else {
                            width -= d.width + 8
                        }
                        return result
                    }
                    .alignmentGuide(.top) { _ in
                        let result = height
                        if tag == tags.last {
                            height = 0
                        }
                        return result
                    }
            }
        }
    }
}

struct TagView: View {
    let text: String
    
    var body: some View {
        Text(text)
            .font(.caption)
            .fontWeight(.medium)
            .padding(.horizontal, 10)
            .padding(.vertical, 5)
            .background(Color.blue)
            .foregroundColor(.white)
            .cornerRadius(12)
    }
}
```

---

## Custom Alignment

สร้าง alignment ของตัวเองได้

```swift
// Custom Horizontal Alignment
extension HorizontalAlignment {
    private struct TitleAlignment: AlignmentID {
        static func defaultValue(in context: ViewDimensions) -> CGFloat {
            context[.leading]
        }
    }
    
    static let titleAlignment = HorizontalAlignment(TitleAlignment.self)
}

// Custom Vertical Alignment
extension VerticalAlignment {
    private struct MidPoint: AlignmentID {
        static func defaultValue(in context: ViewDimensions) -> CGFloat {
            context[.top]
        }
    }
    
    static let midPoint = VerticalAlignment(MidPoint.self)
}

struct CustomAlignmentExample: View {
    var body: some View {
        // ใช้ custom alignment
        VStack(alignment: .titleAlignment, spacing: 8) {
            HStack {
                Image(systemName: "star.fill")
                    .foregroundColor(.yellow)
                Text("หัวข้อหลัก")
                    .font(.title2)
                    .fontWeight(.bold)
                    .alignmentGuide(.titleAlignment) { d in d[.leading] }
            }
            
            HStack {
                Image(systemName: "info.circle")
                    .foregroundColor(.blue)
                Text("รายละเอียด")
                    .alignmentGuide(.titleAlignment) { d in d[.leading] }
            }
            
            // Indented content
            Text("ข้อความย่อย")
                .font(.caption)
                .foregroundColor(.gray)
                .alignmentGuide(.titleAlignment) { d in d[.leading] - 20 }
        }
        .padding()
        .background(Color.white)
        .cornerRadius(12)
        .shadow(radius: 4)
        .padding()
    }
}
```

---

## padding, frame, fixedSize

### padding

```swift
struct PaddingExamples: View {
    var body: some View {
        VStack(spacing: 16) {
            // padding รอบทุกด้าน
            Text("padding()")
                .padding()
                .background(Color.blue.opacity(0.2))
            
            // padding ค่าเฉพาะ
            Text("padding(20)")
                .padding(20)
                .background(Color.green.opacity(0.2))
            
            // padding เฉพาะด้าน
            Text("padding(.horizontal, 30)")
                .padding(.horizontal, 30)
                .background(Color.orange.opacity(0.2))
            
            Text("padding(.top, 20)")
                .padding(.top, 20)
                .background(Color.red.opacity(0.2))
            
            // padding หลายด้าน
            Text("padding vertical=10, horizontal=20")
                .padding(.vertical, 10)
                .padding(.horizontal, 20)
                .background(Color.purple.opacity(0.2))
            
            // EdgeInsets
            Text("EdgeInsets")
                .padding(EdgeInsets(top: 10, leading: 20, bottom: 5, trailing: 30))
                .background(Color.pink.opacity(0.2))
        }
        .padding()
    }
}
```

### frame

```swift
struct FrameExamples: View {
    var body: some View {
        VStack(spacing: 16) {
            
            // frame ขนาดคงที่
            Text("Fixed frame")
                .frame(width: 200, height: 50)
                .background(Color.blue.opacity(0.2))
            
            // frame maxWidth/maxHeight
            Text("Max Width")
                .frame(maxWidth: .infinity)
                .padding()
                .background(Color.green.opacity(0.2))
            
            // frame ทั้งหมด
            Text("Full frame")
                .frame(maxWidth: .infinity, maxHeight: .infinity)
                .background(Color.orange.opacity(0.2))
                .frame(height: 100)  // จำกัดความสูง
            
            // frame alignment
            Text("Leading")
                .frame(maxWidth: .infinity, alignment: .leading)
                .padding()
                .background(Color.purple.opacity(0.2))
            
            Text("Trailing")
                .frame(maxWidth: .infinity, alignment: .trailing)
                .padding()
                .background(Color.red.opacity(0.2))
            
            // frame min/max/ideal
            Text("Min/Max Width")
                .frame(minWidth: 100, maxWidth: 250)
                .padding()
                .background(Color.teal.opacity(0.2))
        }
        .padding()
    }
}
```

### fixedSize

```swift
struct FixedSizeExamples: View {
    var body: some View {
        VStack(spacing: 20) {
            
            // ปัญหา: ข้อความถูก truncate ใน frame เล็กๆ
            Text("ข้อความที่ยาวเกินกว่าจะแสดงใน frame นี้ได้")
                .frame(width: 100)
                .background(Color.red.opacity(0.2))
            
            // แก้: ใช้ fixedSize(horizontal: true, vertical: false)
            Text("ข้อความที่ยาวมากแต่ใช้ fixedSize เพื่อไม่ให้ถูก truncate")
                .fixedSize(horizontal: false, vertical: true)
                .frame(width: 200)
                .background(Color.green.opacity(0.2))
            
            // fixedSize() ไม่ให้ shrink
            Text("Fixed Size Text ที่ไม่ยอม shrink")
                .fixedSize()
                .background(Color.blue.opacity(0.2))
            
            // ใช้ fixedSize กับ HStack
            HStack {
                Text("Left")
                    .fixedSize()
                Spacer()
                Text("Right")
                    .fixedSize()
            }
            .frame(width: 200)
            .padding()
            .background(Color.orange.opacity(0.2))
        }
        .padding()
    }
}
```

---

## edgesIgnoringSafeArea และ safeAreaInset

### ignoresSafeArea

```swift
struct SafeAreaExamples: View {
    var body: some View {
        ZStack {
            // Background สีที่ครอบทั้งหน้าจอรวม safe area
            Color.blue
                .ignoresSafeArea()
            
            // Content อยู่ใน safe area ปกติ
            VStack {
                Text("Content ใน Safe Area")
                    .foregroundColor(.white)
                    .font(.title)
                
                Spacer()
                
                Text("Bottom Content")
                    .foregroundColor(.white)
            }
            .padding()
        }
    }
}

// ignoresSafeArea พร้อม edges
struct SafeAreaEdgesExample: View {
    var body: some View {
        VStack(spacing: 0) {
            // Header ที่ครอบ top safe area
            Color.blue
                .frame(height: 120)
                .ignoresSafeArea(.all, edges: .top)
                .overlay(
                    Text("Header")
                        .foregroundColor(.white)
                        .font(.title)
                )
            
            Spacer()
            
            // Footer ที่ครอบ bottom safe area
            Color.green
                .frame(height: 80)
                .ignoresSafeArea(.all, edges: .bottom)
                .overlay(
                    Text("Footer")
                        .foregroundColor(.white)
                )
        }
    }
}
```

### safeAreaInset (iOS 15+)

```swift
struct SafeAreaInsetExample: View {
    @State private var showBanner = true
    
    var body: some View {
        ScrollView {
            LazyVStack {
                ForEach(1...20, id: \.self) { i in
                    Text("รายการ \(i)")
                        .frame(maxWidth: .infinity)
                        .padding()
                        .background(Color.white)
                        .cornerRadius(8)
                        .shadow(radius: 2)
                        .padding(.horizontal)
                }
            }
        }
        // เพิ่ม inset ที่ bottom สำหรับ floating button
        .safeAreaInset(edge: .bottom) {
            Button {
                // action
            } label: {
                Label("เพิ่มรายการ", systemImage: "plus")
                    .font(.headline)
                    .foregroundColor(.white)
                    .frame(maxWidth: .infinity)
                    .padding()
                    .background(Color.blue)
                    .cornerRadius(15)
                    .padding()
            }
            .background(.thinMaterial)
        }
        // Banner ที่ top
        .safeAreaInset(edge: .top, spacing: 0) {
            if showBanner {
                HStack {
                    Text("🎉 ยินดีต้อนรับสู่แอป!")
                        .font(.subheadline)
                    Spacer()
                    Button {
                        withAnimation {
                            showBanner = false
                        }
                    } label: {
                        Image(systemName: "xmark")
                            .foregroundColor(.primary)
                    }
                }
                .padding()
                .background(Color.yellow)
            }
        }
    }
}
```

---

## overlay และ background

### overlay

```swift
struct OverlayExamples: View {
    var body: some View {
        VStack(spacing: 24) {
            
            // overlay พื้นฐาน
            Image(systemName: "photo")
                .resizable()
                .aspectRatio(contentMode: .fit)
                .frame(width: 150, height: 100)
                .foregroundColor(.gray)
                .overlay(
                    Text("Overlay")
                        .font(.caption)
                        .foregroundColor(.white)
                        .padding(4)
                        .background(Color.black.opacity(0.5))
                        .cornerRadius(4),
                    alignment: .bottomTrailing
                )
            
            // overlay พร้อม alignment
            ZStack {
                Rectangle()
                    .fill(Color.blue.opacity(0.2))
                    .frame(width: 200, height: 120)
                    .cornerRadius(12)
                    .overlay(
                        RoundedRectangle(cornerRadius: 12)
                            .stroke(Color.blue, lineWidth: 2)
                    )
                
                Text("Bordered Card")
            }
            
            // overlay สำหรับ badge
            Image(systemName: "bell.fill")
                .font(.system(size: 40))
                .foregroundColor(.blue)
                .overlay(alignment: .topTrailing) {
                    Circle()
                        .fill(Color.red)
                        .frame(width: 16, height: 16)
                        .overlay(
                            Text("5")
                                .font(.system(size: 10))
                                .foregroundColor(.white)
                        )
                        .offset(x: 6, y: -6)
                }
        }
        .padding()
    }
}
```

### background

```swift
struct BackgroundExamples: View {
    var body: some View {
        VStack(spacing: 16) {
            
            // background สี
            Text("Color Background")
                .padding()
                .background(Color.blue)
                .foregroundColor(.white)
                .cornerRadius(8)
            
            // background shape
            Text("Shape Background")
                .padding()
                .background(
                    Capsule()
                        .fill(Color.green)
                )
                .foregroundColor(.white)
            
            // background gradient
            Text("Gradient Background")
                .padding()
                .background(
                    LinearGradient(
                        colors: [.purple, .blue],
                        startPoint: .leading,
                        endPoint: .trailing
                    )
                    .cornerRadius(10)
                )
                .foregroundColor(.white)
            
            // background material (iOS 15+)
            Text("Material Background")
                .padding()
                .background(.thinMaterial)
                .cornerRadius(10)
            
            // background view
            Text("Custom Background")
                .font(.headline)
                .padding()
                .background {
                    ZStack {
                        RoundedRectangle(cornerRadius: 12)
                            .fill(Color.white)
                        RoundedRectangle(cornerRadius: 12)
                            .stroke(
                                LinearGradient(
                                    colors: [.blue, .purple],
                                    startPoint: .leading,
                                    endPoint: .trailing
                                ),
                                lineWidth: 2
                            )
                    }
                }
        }
        .padding()
    }
}
```

---

## ScrollView

```swift
struct ScrollViewExamples: View {
    var body: some View {
        VStack {
            // ScrollView แนวตั้ง (default)
            ScrollView {
                VStack(spacing: 12) {
                    ForEach(1...20, id: \.self) { i in
                        Text("รายการ \(i)")
                            .frame(maxWidth: .infinity)
                            .padding()
                            .background(Color.blue.opacity(0.1))
                            .cornerRadius(8)
                    }
                }
                .padding()
            }
            .frame(height: 200)
            
            // ScrollView แนวนอน
            ScrollView(.horizontal, showsIndicators: false) {
                HStack(spacing: 12) {
                    ForEach(1...10, id: \.self) { i in
                        RoundedRectangle(cornerRadius: 12)
                            .fill(Color(hue: Double(i) / 10, saturation: 0.6, brightness: 0.8))
                            .frame(width: 120, height: 80)
                            .overlay(
                                Text("Card \(i)")
                                    .foregroundColor(.white)
                                    .font(.caption)
                            )
                    }
                }
                .padding()
            }
            
            // ScrollView ทั้งสองทิศทาง
            ScrollView([.horizontal, .vertical]) {
                LazyVGrid(columns: Array(repeating: GridItem(.fixed(100)), count: 10), spacing: 8) {
                    ForEach(0..<100) { i in
                        Text("\(i)")
                            .frame(width: 100, height: 100)
                            .background(Color.random)
                            .foregroundColor(.white)
                    }
                }
                .padding()
            }
            .frame(height: 200)
        }
    }
}

// Extension สำหรับ random color
extension Color {
    static var random: Color {
        Color(
            hue: Double.random(in: 0...1),
            saturation: 0.5,
            brightness: 0.8
        )
    }
}
```

---

## ScrollViewReader

`ScrollViewReader` ให้เราควบคุมการ scroll ไปยัง View ที่ต้องการ

```swift
struct ScrollViewReaderExample: View {
    @State private var selectedID: Int = 0
    
    var body: some View {
        VStack {
            // Controls
            HStack {
                Button("ไปบน") {
                    withAnimation {
                        selectedID = 1
                    }
                }
                
                Spacer()
                
                Button("ไปกลาง") {
                    withAnimation {
                        selectedID = 50
                    }
                }
                
                Spacer()
                
                Button("ไปล่าง") {
                    withAnimation {
                        selectedID = 100
                    }
                }
            }
            .padding()
            .buttonStyle(.bordered)
            
            // ScrollView พร้อม ScrollViewReader
            ScrollViewReader { proxy in
                ScrollView {
                    LazyVStack(spacing: 8) {
                        ForEach(1...100, id: \.self) { i in
                            HStack {
                                Text("รายการ #\(i)")
                                    .font(i == selectedID ? .headline : .body)
                                    .foregroundColor(i == selectedID ? .white : .primary)
                                Spacer()
                                Image(systemName: i == selectedID ? "checkmark.circle.fill" : "circle")
                                    .foregroundColor(i == selectedID ? .white : .gray)
                            }
                            .padding()
                            .background(i == selectedID ? Color.blue : Color.white)
                            .cornerRadius(10)
                            .shadow(color: .black.opacity(0.05), radius: 2)
                            .id(i)  // กำหนด ID ให้แต่ละ View
                        }
                    }
                    .padding(.horizontal)
                }
                .onChange(of: selectedID) { newValue in
                    withAnimation(.smooth) {
                        proxy.scrollTo(newValue, anchor: .center)
                    }
                }
            }
        }
    }
}
```

---

## List พื้นฐาน

```swift
struct BasicListExamples: View {
    
    struct Fruit: Identifiable {
        let id = UUID()
        let name: String
        let emoji: String
    }
    
    let fruits = [
        Fruit(name: "แอปเปิ้ล", emoji: "🍎"),
        Fruit(name: "กล้วย", emoji: "🍌"),
        Fruit(name: "ส้ม", emoji: "🍊"),
        Fruit(name: "มะม่วง", emoji: "🥭"),
        Fruit(name: "สตรอว์เบอร์รี", emoji: "🍓"),
        Fruit(name: "ลูกพีช", emoji: "🍑")
    ]
    
    @State private var selectedFruit: Fruit?
    
    var body: some View {
        NavigationStack {
            List(fruits) { fruit in
                HStack {
                    Text(fruit.emoji)
                        .font(.title2)
                    
                    Text(fruit.name)
                        .font(.body)
                    
                    Spacer()
                    
                    if selectedFruit?.id == fruit.id {
                        Image(systemName: "checkmark")
                            .foregroundColor(.blue)
                    }
                }
                .contentShape(Rectangle())
                .onTapGesture {
                    selectedFruit = fruit
                }
            }
            .listStyle(.insetGrouped)
            .navigationTitle("ผลไม้")
        }
    }
}
```

---

## ForEach

```swift
struct ForEachExamples: View {
    
    @State private var items = ["Swift", "Kotlin", "Python", "JavaScript", "Go"]
    
    var body: some View {
        List {
            // ForEach พื้นฐาน
            Section("ภาษา Programming") {
                ForEach(items, id: \.self) { item in
                    Text(item)
                }
            }
            
            // ForEach พร้อม Delete
            Section("แก้ไขได้") {
                ForEach(items, id: \.self) { item in
                    Text(item)
                }
                .onDelete { indexSet in
                    items.remove(atOffsets: indexSet)
                }
                .onMove { from, to in
                    items.move(fromOffsets: from, toOffset: to)
                }
            }
            
            // ForEach กับ Range
            Section("ตัวเลข") {
                ForEach(1..<11) { number in
                    Text("\(number)")
                }
            }
            
            // ForEach กับ Identifiable
            Section("Identifiable") {
                ForEach(sampleProducts) { product in
                    ProductRow(product: product)
                }
            }
        }
        .navigationTitle("ForEach Examples")
        .toolbar {
            EditButton()
        }
    }
    
    private var sampleProducts: [Product] {
        [
            Product(name: "iPhone 16", price: 29900),
            Product(name: "MacBook Pro", price: 89900),
            Product(name: "iPad Pro", price: 39900)
        ]
    }
}

struct Product: Identifiable {
    let id = UUID()
    let name: String
    let price: Int
}

struct ProductRow: View {
    let product: Product
    
    var body: some View {
        HStack {
            VStack(alignment: .leading) {
                Text(product.name)
                    .font(.headline)
                Text("฿\(product.price.formatted())")
                    .font(.subheadline)
                    .foregroundColor(.green)
            }
            Spacer()
            Image(systemName: "chevron.right")
                .foregroundColor(.gray)
        }
        .padding(.vertical, 4)
    }
}
```

---

## Section ใน List

```swift
struct SectionExamples: View {
    
    struct Contact: Identifiable {
        let id = UUID()
        let name: String
        let phone: String
        let group: String
    }
    
    let contacts = [
        Contact(name: "สมชาย", phone: "081-111-1111", group: "ครอบครัว"),
        Contact(name: "สมหญิง", phone: "082-222-2222", group: "ครอบครัว"),
        Contact(name: "มานะ", phone: "083-333-3333", group: "เพื่อน"),
        Contact(name: "วิทยา", phone: "084-444-4444", group: "เพื่อน"),
        Contact(name: "กรุณา", phone: "085-555-5555", group: "งาน"),
        Contact(name: "ศิริ", phone: "086-666-6666", group: "งาน")
    ]
    
    var groupedContacts: [String: [Contact]] {
        Dictionary(grouping: contacts, by: { $0.group })
    }
    
    var sortedGroups: [String] {
        groupedContacts.keys.sorted()
    }
    
    var body: some View {
        NavigationStack {
            List {
                // Section พร้อม header
                ForEach(sortedGroups, id: \.self) { group in
                    Section {
                        ForEach(groupedContacts[group] ?? []) { contact in
                            HStack {
                                VStack(alignment: .leading, spacing: 2) {
                                    Text(contact.name)
                                        .font(.headline)
                                    Text(contact.phone)
                                        .font(.caption)
                                        .foregroundColor(.gray)
                                }
                                Spacer()
                                Image(systemName: "phone.circle.fill")
                                    .foregroundColor(.green)
                                    .font(.title2)
                            }
                        }
                    } header: {
                        Label(group, systemImage: groupIcon(group))
                            .font(.headline)
                            .foregroundColor(.primary)
                    } footer: {
                        Text("\(groupedContacts[group]?.count ?? 0) รายชื่อ")
                    }
                }
            }
            .listStyle(.insetGrouped)
            .navigationTitle("ผู้ติดต่อ")
        }
    }
    
    private func groupIcon(_ group: String) -> String {
        switch group {
        case "ครอบครัว": return "house.fill"
        case "เพื่อน": return "person.2.fill"
        case "งาน": return "briefcase.fill"
        default: return "person.fill"
        }
    }
}
```

---

## Dynamic Content

```swift
struct DynamicContentExample: View {
    @State private var items: [Item] = []
    @State private var isLoading = false
    @State private var searchText = ""
    
    struct Item: Identifiable {
        let id = UUID()
        let title: String
        let subtitle: String
        let icon: String
    }
    
    private let allItems = [
        Item(title: "SwiftUI", subtitle: "UI Framework", icon: "iphone"),
        Item(title: "CoreData", subtitle: "Database", icon: "cylinder"),
        Item(title: "Combine", subtitle: "Reactive Framework", icon: "arrow.triangle.2.circlepath"),
        Item(title: "URLSession", subtitle: "Networking", icon: "network"),
        Item(title: "UserDefaults", subtitle: "Simple Storage", icon: "externaldrive"),
        Item(title: "FileManager", subtitle: "File System", icon: "folder"),
        Item(title: "NotificationCenter", subtitle: "Events", icon: "bell"),
        Item(title: "CoreLocation", subtitle: "GPS", icon: "location")
    ]
    
    var filteredItems: [Item] {
        if searchText.isEmpty {
            return items
        }
        return items.filter {
            $0.title.localizedCaseInsensitiveContains(searchText) ||
            $0.subtitle.localizedCaseInsensitiveContains(searchText)
        }
    }
    
    var body: some View {
        NavigationStack {
            Group {
                if isLoading {
                    ProgressView("กำลังโหลด...")
                        .frame(maxWidth: .infinity, maxHeight: .infinity)
                } else if filteredItems.isEmpty {
                    EmptyStateView(searchText: searchText)
                } else {
                    List(filteredItems) { item in
                        HStack(spacing: 12) {
                            Image(systemName: item.icon)
                                .font(.title2)
                                .foregroundColor(.blue)
                                .frame(width: 40, height: 40)
                                .background(Color.blue.opacity(0.1))
                                .cornerRadius(10)
                            
                            VStack(alignment: .leading, spacing: 2) {
                                Text(item.title)
                                    .font(.headline)
                                Text(item.subtitle)
                                    .font(.caption)
                                    .foregroundColor(.gray)
                            }
                        }
                        .padding(.vertical, 4)
                    }
                }
            }
            .searchable(text: $searchText, prompt: "ค้นหา Framework")
            .navigationTitle("iOS Frameworks")
            .task {
                await loadData()
            }
            .refreshable {
                await loadData()
            }
        }
    }
    
    private func loadData() async {
        isLoading = true
        try? await Task.sleep(nanoseconds: 1_000_000_000)  // จำลอง network delay
        items = allItems
        isLoading = false
    }
}

struct EmptyStateView: View {
    let searchText: String
    
    var body: some View {
        VStack(spacing: 16) {
            Image(systemName: searchText.isEmpty ? "tray" : "magnifyingglass")
                .font(.system(size: 60))
                .foregroundColor(.gray)
            
            Text(searchText.isEmpty ? "ไม่มีข้อมูล" : "ไม่พบผลการค้นหา")
                .font(.title2)
                .fontWeight(.semibold)
            
            if !searchText.isEmpty {
                Text("ไม่พบ \"\(searchText)\"")
                    .font(.subheadline)
                    .foregroundColor(.gray)
            }
        }
        .frame(maxWidth: .infinity, maxHeight: .infinity)
    }
}
```

---

## Spacer Behavior

```swift
struct SpacerBehaviorExamples: View {
    var body: some View {
        VStack(spacing: 30) {
            
            // Spacer ดันไปชิดขอบ
            HStack {
                Text("ซ้าย")
                    .padding()
                    .background(Color.red.opacity(0.2))
                
                Spacer()
                
                Text("ขวา")
                    .padding()
                    .background(Color.blue.opacity(0.2))
            }
            .background(Color.gray.opacity(0.1))
            
            // Spacer หลายอัน - แบ่งพื้นที่เท่ากัน
            HStack {
                Text("A")
                    .padding()
                    .background(Color.red.opacity(0.3))
                
                Spacer()
                
                Text("B")
                    .padding()
                    .background(Color.green.opacity(0.3))
                
                Spacer()
                
                Text("C")
                    .padding()
                    .background(Color.blue.opacity(0.3))
                
                Spacer()
                
                Text("D")
                    .padding()
                    .background(Color.orange.opacity(0.3))
            }
            .background(Color.gray.opacity(0.1))
            
            // Spacer กับ minLength
            HStack {
                Text("Short")
                
                Spacer(minLength: 100)  // อย่างน้อย 100 points
                
                Text("Right")
            }
            .padding()
            .background(Color.gray.opacity(0.1))
            
            // VStack Spacer
            VStack {
                Text("Top")
                    .padding()
                    .background(Color.red.opacity(0.2))
                
                Spacer()
                
                Text("Center ถ้ามีแค่ spacer เดียว จะอยู่กลาง")
                    .padding()
                    .background(Color.green.opacity(0.2))
                
                Spacer()
                
                Text("Bottom")
                    .padding()
                    .background(Color.blue.opacity(0.2))
            }
            .frame(height: 200)
            .background(Color.gray.opacity(0.1))
        }
        .padding()
    }
}
```

---

## ViewThatFits

`ViewThatFits` (iOS 16+) เลือก View ที่พอดีกับพื้นที่ที่มี

```swift
struct ViewThatFitsExample: View {
    var body: some View {
        VStack(spacing: 24) {
            
            // ViewThatFits เลือก View แรกที่พอดี
            ViewThatFits {
                // ลองแสดง horizontal layout ก่อน
                HStack {
                    Image(systemName: "star.fill")
                    Text("ข้อความยาวมากที่อาจไม่พอดีในหน้าจอเล็ก")
                    Image(systemName: "heart.fill")
                }
                
                // ถ้าไม่พอดี ลอง vertical layout
                VStack {
                    Image(systemName: "star.fill")
                    Text("ข้อความที่เหมาะกับ vertical layout")
                    Image(systemName: "heart.fill")
                }
            }
            .padding()
            .background(Color.blue.opacity(0.1))
            .cornerRadius(10)
            
            // ตัวอย่างใช้งานจริง: Toolbar ที่ปรับตัว
            ViewThatFits(in: .horizontal) {
                // Full toolbar
                HStack(spacing: 16) {
                    toolbarButton("house.fill", "หน้าหลัก")
                    toolbarButton("magnifyingglass", "ค้นหา")
                    toolbarButton("bell.fill", "การแจ้งเตือน")
                    toolbarButton("person.fill", "โปรไฟล์")
                    toolbarButton("gearshape.fill", "ตั้งค่า")
                }
                
                // Compact toolbar (3 items)
                HStack(spacing: 16) {
                    toolbarButton("house.fill", "หน้าหลัก")
                    toolbarButton("magnifyingglass", "ค้นหา")
                    toolbarButton("person.fill", "โปรไฟล์")
                }
                
                // Minimal toolbar (2 items)
                HStack(spacing: 16) {
                    toolbarButton("house.fill", "หน้าหลัก")
                    toolbarButton("ellipsis", "เพิ่มเติม")
                }
            }
            .padding()
            .background(Color.white)
            .cornerRadius(15)
            .shadow(radius: 5)
        }
        .padding()
    }
    
    private func toolbarButton(_ icon: String, _ label: String) -> some View {
        VStack(spacing: 4) {
            Image(systemName: icon)
                .font(.title3)
            Text(label)
                .font(.caption2)
        }
        .foregroundColor(.blue)
        .frame(minWidth: 60)
    }
}
```

---

## AnyLayout

`AnyLayout` (iOS 16+) ช่วยสลับระหว่าง layout types โดยไม่ต้องเปลี่ยน content

```swift
struct AnyLayoutExample: View {
    @State private var isVertical = false
    @Environment(\.horizontalSizeClass) private var sizeClass
    
    var layout: AnyLayout {
        if isVertical || sizeClass == .compact {
            return AnyLayout(VStackLayout(spacing: 12))
        } else {
            return AnyLayout(HStackLayout(spacing: 20))
        }
    }
    
    var body: some View {
        VStack {
            // Toggle
            Toggle("Layout แนวตั้ง", isOn: $isVertical)
                .padding()
            
            // Content ที่เปลี่ยน layout อย่าง smooth
            layout {
                ForEach(["Swift", "Kotlin", "Python"], id: \.self) { lang in
                    Text(lang)
                        .font(.headline)
                        .padding()
                        .frame(maxWidth: isVertical ? .infinity : 100)
                        .background(Color.blue.opacity(0.1))
                        .cornerRadius(10)
                }
            }
            .animation(.spring(), value: isVertical)
            .padding()
        }
    }
}

// AnyLayout กับ GridLayout
struct AnyLayoutGridExample: View {
    @State private var columns = 2
    
    var layout: AnyLayout {
        switch columns {
        case 1:
            return AnyLayout(VStackLayout(spacing: 8))
        case 2:
            return AnyLayout(LazyVGridLayout(columns: [GridItem(), GridItem()], spacing: 8))
        default:
            return AnyLayout(LazyVGridLayout(columns: [GridItem(), GridItem(), GridItem()], spacing: 8))
        }
    }
    
    var body: some View {
        VStack {
            Picker("Columns", selection: $columns) {
                Text("1").tag(1)
                Text("2").tag(2)
                Text("3").tag(3)
            }
            .pickerStyle(.segmented)
            .padding()
            
            ScrollView {
                layout {
                    ForEach(1...12, id: \.self) { i in
                        RoundedRectangle(cornerRadius: 12)
                            .fill(Color(hue: Double(i) / 12, saturation: 0.6, brightness: 0.8))
                            .aspectRatio(1, contentMode: .fit)
                            .overlay(
                                Text("\(i)")
                                    .font(.title)
                                    .foregroundColor(.white)
                            )
                    }
                }
                .animation(.spring(), value: columns)
                .padding()
            }
        }
    }
}
```

---

## Custom Layout Protocol

iOS 16+ ให้เราสร้าง Layout ของตัวเองผ่าน `Layout` protocol

```swift
// Custom Layout: Radial/Circular Layout
struct RadialLayout: Layout {
    var radius: CGFloat = 100
    var startAngle: Angle = .degrees(0)
    
    // คำนวณขนาดรวมของ layout
    func sizeThatFits(
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout ()
    ) -> CGSize {
        let diameter = radius * 2
        return CGSize(width: diameter + 60, height: diameter + 60)
    }
    
    // วาง subviews ตามวงกลม
    func placeSubviews(
        in bounds: CGRect,
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout ()
    ) {
        guard !subviews.isEmpty else { return }
        
        let angleStep = Angle.degrees(360.0 / Double(subviews.count))
        let center = CGPoint(x: bounds.midX, y: bounds.midY)
        
        for (index, subview) in subviews.enumerated() {
            let angle = startAngle + angleStep * Double(index)
            let x = center.x + radius * CGFloat(cos(angle.radians))
            let y = center.y + radius * CGFloat(sin(angle.radians))
            
            let size = subview.sizeThatFits(.unspecified)
            subview.place(
                at: CGPoint(x: x - size.width / 2, y: y - size.height / 2),
                proposal: .unspecified
            )
        }
    }
}

// การใช้งาน Custom Layout
struct RadialLayoutExample: View {
    let items = ["🍎", "🍌", "🍊", "🍇", "🍓", "🍑", "🥝", "🍋"]
    @State private var radius: CGFloat = 100
    
    var body: some View {
        VStack {
            Text("Radial Layout")
                .font(.title)
            
            RadialLayout(radius: radius) {
                ForEach(items, id: \.self) { item in
                    Text(item)
                        .font(.title2)
                }
            }
            .frame(width: 280, height: 280)
            .overlay(
                Circle()
                    .stroke(Color.gray.opacity(0.3), lineWidth: 1)
                    .frame(width: radius * 2, height: radius * 2)
            )
            
            Slider(value: $radius, in: 60...130) {
                Text("Radius")
            }
            .padding()
        }
    }
}

// Waterfall/Masonry Layout
struct MasonryLayout: Layout {
    var columns: Int = 2
    var spacing: CGFloat = 8
    
    func sizeThatFits(
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout ()
    ) -> CGSize {
        let width = proposal.width ?? 300
        let columnWidth = (width - CGFloat(columns - 1) * spacing) / CGFloat(columns)
        
        var columnHeights = Array(repeating: CGFloat(0), count: columns)
        
        for subview in subviews {
            let shortestColumn = columnHeights.indices.min(by: {
                columnHeights[$0] < columnHeights[$1]
            }) ?? 0
            
            let height = subview.sizeThatFits(
                ProposedViewSize(width: columnWidth, height: nil)
            ).height
            
            columnHeights[shortestColumn] += height + spacing
        }
        
        let totalHeight = columnHeights.max() ?? 0
        return CGSize(width: width, height: totalHeight)
    }
    
    func placeSubviews(
        in bounds: CGRect,
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout ()
    ) {
        let columnWidth = (bounds.width - CGFloat(columns - 1) * spacing) / CGFloat(columns)
        var columnHeights = Array(repeating: CGFloat(0), count: columns)
        
        for subview in subviews {
            let shortestColumn = columnHeights.indices.min(by: {
                columnHeights[$0] < columnHeights[$1]
            }) ?? 0
            
            let x = bounds.minX + CGFloat(shortestColumn) * (columnWidth + spacing)
            let y = bounds.minY + columnHeights[shortestColumn]
            
            let height = subview.sizeThatFits(
                ProposedViewSize(width: columnWidth, height: nil)
            ).height
            
            subview.place(
                at: CGPoint(x: x, y: y),
                proposal: ProposedViewSize(width: columnWidth, height: height)
            )
            
            columnHeights[shortestColumn] += height + spacing
        }
    }
}
```

---

## แบบฝึกหัดพร้อมเฉลย

### แบบฝึกหัดที่ 1: สร้าง Photo Gallery Grid

```swift
// โจทย์: สร้าง Photo Gallery ที่ใช้ LazyVGrid
// - มี Tab สำหรับเลือกระหว่าง 2 column และ 3 column
// - กดรูปแล้วแสดง detail view
// - ใช้ animation ตอน switch layout

struct PhotoGalleryView: View {
    @State private var columns = 3
    @State private var selectedPhoto: PhotoItem?
    
    struct PhotoItem: Identifiable {
        let id: Int
        let color: Color
        let title: String
        let aspectRatio: CGFloat
    }
    
    let photos: [PhotoItem] = (1...20).map { i in
        PhotoItem(
            id: i,
            color: Color(
                hue: Double(i) / 20,
                saturation: 0.6,
                brightness: 0.7 + Double(i % 3) * 0.1
            ),
            title: "ภาพที่ \(i)",
            aspectRatio: [1, 0.75, 1.33].randomElement()!
        )
    }
    
    var gridColumns: [GridItem] {
        Array(repeating: GridItem(.flexible(), spacing: 2), count: columns)
    }
    
    var body: some View {
        NavigationStack {
            ScrollView {
                LazyVGrid(columns: gridColumns, spacing: 2) {
                    ForEach(photos) { photo in
                        photoCell(photo)
                            .onTapGesture {
                                selectedPhoto = photo
                            }
                    }
                }
            }
            .navigationTitle("Photo Gallery")
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Picker("Columns", selection: $columns) {
                        Image(systemName: "square.grid.2x2").tag(2)
                        Image(systemName: "square.grid.3x3").tag(3)
                    }
                    .pickerStyle(.segmented)
                }
            }
            .sheet(item: $selectedPhoto) { photo in
                PhotoDetailView(photo: photo)
            }
            .animation(.spring(response: 0.4), value: columns)
        }
    }
    
    @ViewBuilder
    private func photoCell(_ photo: PhotoItem) -> some View {
        photo.color
            .aspectRatio(1, contentMode: .fill)
            .overlay(
                Text(photo.title)
                    .font(.caption2)
                    .foregroundColor(.white)
                    .padding(4)
                    .background(Color.black.opacity(0.4))
                    .cornerRadius(4),
                alignment: .bottomLeading
            )
            .clipped()
    }
}

struct PhotoDetailView: View {
    let photo: PhotoGalleryView.PhotoItem
    @Environment(\.dismiss) private var dismiss
    
    var body: some View {
        NavigationStack {
            VStack {
                photo.color
                    .aspectRatio(4/3, contentMode: .fit)
                    .cornerRadius(16)
                    .padding()
                
                Text(photo.title)
                    .font(.title2)
                    .fontWeight(.semibold)
                
                Text("สร้างด้วย SwiftUI LazyVGrid")
                    .font(.body)
                    .foregroundColor(.gray)
                
                Spacer()
            }
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .cancellationAction) {
                    Button("ปิด") { dismiss() }
                }
            }
        }
    }
}

#Preview("Photo Gallery") {
    PhotoGalleryView()
}
```

### แบบฝึกหัดที่ 2: Dashboard Layout

```swift
struct DashboardView: View {
    
    struct Stat {
        let title: String
        let value: String
        let change: Double
        let icon: String
        let color: Color
    }
    
    let stats = [
        Stat(title: "ยอดขาย", value: "฿128,450", change: 12.5, icon: "cart.fill", color: .blue),
        Stat(title: "ผู้ใช้ใหม่", value: "1,284", change: -2.3, icon: "person.badge.plus", color: .green),
        Stat(title: "คำสั่งซื้อ", value: "342", change: 8.7, icon: "doc.text.fill", color: .orange),
        Stat(title: "รายได้สุทธิ", value: "฿45,230", change: 15.2, icon: "banknote.fill", color: .purple)
    ]
    
    var body: some View {
        NavigationStack {
            ScrollView {
                VStack(spacing: 20) {
                    // Stats Grid
                    LazyVGrid(columns: [GridItem(.flexible()), GridItem(.flexible())], spacing: 16) {
                        ForEach(stats.indices, id: \.self) { index in
                            StatCard(stat: stats[index])
                        }
                    }
                    .padding(.horizontal)
                    
                    // Chart Section placeholder
                    VStack(alignment: .leading, spacing: 12) {
                        Text("ภาพรวมรายสัปดาห์")
                            .font(.headline)
                        
                        // Simple bar chart
                        HStack(alignment: .bottom, spacing: 8) {
                            ForEach(weeklyData.indices, id: \.self) { i in
                                VStack(spacing: 4) {
                                    Text("\(weeklyData[i])")
                                        .font(.caption2)
                                    
                                    RoundedRectangle(cornerRadius: 4)
                                        .fill(Color.blue.gradient)
                                        .frame(
                                            width: 30,
                                            height: CGFloat(weeklyData[i]) / CGFloat(weeklyData.max() ?? 1) * 100
                                        )
                                    
                                    Text(dayLabels[i])
                                        .font(.caption2)
                                        .foregroundColor(.gray)
                                }
                            }
                        }
                        .frame(maxWidth: .infinity, alignment: .center)
                        .padding()
                        .background(Color.white)
                        .cornerRadius(12)
                        .shadow(radius: 2)
                    }
                    .padding(.horizontal)
                    
                    // Recent Activity
                    VStack(alignment: .leading, spacing: 12) {
                        HStack {
                            Text("กิจกรรมล่าสุด")
                                .font(.headline)
                            Spacer()
                            Button("ดูทั้งหมด") { }
                                .font(.subheadline)
                        }
                        
                        ForEach(recentActivities, id: \.title) { activity in
                            ActivityRow(activity: activity)
                        }
                    }
                    .padding(.horizontal)
                }
                .padding(.vertical)
            }
            .background(Color(.systemGroupedBackground))
            .navigationTitle("Dashboard")
        }
    }
    
    let weeklyData = [45, 78, 56, 89, 67, 92, 84]
    let dayLabels = ["จ", "อ", "พ", "พฤ", "ศ", "ส", "อา"]
    
    struct Activity {
        let title: String
        let subtitle: String
        let icon: String
        let color: Color
        let time: String
    }
    
    let recentActivities = [
        Activity(title: "คำสั่งซื้อใหม่ #1234", subtitle: "สมชาย ใจดี", icon: "cart", color: .blue, time: "5 นาทีที่แล้ว"),
        Activity(title: "ผู้ใช้ใหม่สมัครสมาชิก", subtitle: "มานะ รักดี", icon: "person.badge.plus", color: .green, time: "12 นาทีที่แล้ว"),
        Activity(title: "ชำระเงินสำเร็จ", subtitle: "฿2,450", icon: "checkmark.circle", color: .green, time: "30 นาทีที่แล้ว")
    ]
}

struct StatCard: View {
    let stat: DashboardView.Stat
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            HStack {
                Image(systemName: stat.icon)
                    .foregroundColor(stat.color)
                    .font(.title3)
                    .frame(width: 36, height: 36)
                    .background(stat.color.opacity(0.1))
                    .cornerRadius(8)
                
                Spacer()
                
                // Change indicator
                Label(
                    String(format: "%.1f%%", abs(stat.change)),
                    systemImage: stat.change >= 0 ? "arrow.up.right" : "arrow.down.right"
                )
                .font(.caption)
                .foregroundColor(stat.change >= 0 ? .green : .red)
            }
            
            Text(stat.value)
                .font(.title2)
                .fontWeight(.bold)
            
            Text(stat.title)
                .font(.caption)
                .foregroundColor(.gray)
        }
        .padding()
        .background(Color.white)
        .cornerRadius(12)
        .shadow(color: .black.opacity(0.05), radius: 5)
    }
}

struct ActivityRow: View {
    let activity: DashboardView.Activity
    
    var body: some View {
        HStack(spacing: 12) {
            Image(systemName: activity.icon)
                .foregroundColor(activity.color)
                .frame(width: 36, height: 36)
                .background(activity.color.opacity(0.1))
                .cornerRadius(8)
            
            VStack(alignment: .leading, spacing: 2) {
                Text(activity.title)
                    .font(.subheadline)
                    .fontWeight(.medium)
                Text(activity.subtitle)
                    .font(.caption)
                    .foregroundColor(.gray)
            }
            
            Spacer()
            
            Text(activity.time)
                .font(.caption2)
                .foregroundColor(.gray)
        }
        .padding()
        .background(Color.white)
        .cornerRadius(10)
        .shadow(radius: 1)
    }
}

#Preview("Dashboard") {
    DashboardView()
}
```

---

## Real-world UI Layouts

### App Store-style Layout

```swift
struct AppStoreLayout: View {
    
    struct AppItem: Identifiable {
        let id = UUID()
        let name: String
        let developer: String
        let rating: Double
        let icon: String
        let price: String
        let category: String
        let color: Color
    }
    
    let featuredApps: [AppItem] = [
        AppItem(name: "Photo Editor Pro", developer: "Creative Labs", rating: 4.8, icon: "camera.filters", price: "ฟรี", category: "Photography", color: .purple),
        AppItem(name: "Budget Tracker", developer: "Finance Co.", rating: 4.6, icon: "banknote", price: "฿99", category: "Finance", color: .green),
        AppItem(name: "Workout Master", developer: "Health Inc.", rating: 4.9, icon: "dumbbell", price: "ฟรี", category: "Health", color: .orange)
    ]
    
    let topApps: [AppItem] = [
        AppItem(name: "Swift Learn", developer: "Dev Academy", rating: 4.7, icon: "swift", price: "ฟรี", category: "Education", color: .orange),
        AppItem(name: "Note Master", developer: "Productivity Tools", rating: 4.5, icon: "note.text", price: "ฟรี", category: "Productivity", color: .yellow),
        AppItem(name: "Weather Now", developer: "Weather Corp", rating: 4.3, icon: "cloud.sun", price: "ฟรี", category: "Weather", color: .blue)
    ]
    
    var body: some View {
        NavigationStack {
            ScrollView {
                VStack(alignment: .leading, spacing: 24) {
                    
                    // Featured Carousel
                    featuredSection
                    
                    Divider()
                        .padding(.horizontal)
                    
                    // Top Charts
                    topChartsSection
                    
                    Divider()
                        .padding(.horizontal)
                    
                    // Categories Grid
                    categoriesSection
                }
                .padding(.vertical)
            }
            .navigationTitle("ค้นพบ")
            .navigationBarTitleDisplayMode(.large)
        }
    }
    
    private var featuredSection: some View {
        VStack(alignment: .leading) {
            sectionHeader("แอปแนะนำ", subtitle: "คัดสรรมาเพื่อคุณ")
            
            ScrollView(.horizontal, showsIndicators: false) {
                LazyHStack(spacing: 16) {
                    ForEach(featuredApps) { app in
                        FeaturedAppCard(app: app)
                    }
                }
                .padding(.horizontal)
            }
        }
    }
    
    private var topChartsSection: some View {
        VStack(alignment: .leading) {
            sectionHeader("อันดับยอดนิยม", subtitle: "แอปที่ดาวน์โหลดมากที่สุด")
            
            VStack(spacing: 0) {
                ForEach(Array(topApps.enumerated()), id: \.element.id) { index, app in
                    AppListRow(app: app, rank: index + 1)
                    
                    if index < topApps.count - 1 {
                        Divider()
                            .padding(.leading, 76)
                            .padding(.horizontal)
                    }
                }
            }
            .background(Color.white)
            .cornerRadius(12)
            .shadow(radius: 2)
            .padding(.horizontal)
        }
    }
    
    private var categoriesSection: some View {
        VStack(alignment: .leading) {
            sectionHeader("หมวดหมู่", subtitle: "ค้นหาตามประเภท")
            
            LazyVGrid(
                columns: [GridItem(.flexible()), GridItem(.flexible())],
                spacing: 12
            ) {
                ForEach([
                    ("เกม", "gamecontroller.fill", Color.blue),
                    ("ผลิตภาพ", "briefcase.fill", Color.orange),
                    ("สุขภาพ", "heart.fill", Color.red),
                    ("การศึกษา", "books.vertical.fill", Color.green),
                    ("ความบันเทิง", "play.circle.fill", Color.purple),
                    ("ไลฟ์สไตล์", "person.2.fill", Color.pink)
                ], id: \.0) { category in
                    CategoryCard(
                        name: category.0,
                        icon: category.1,
                        color: category.2
                    )
                }
            }
            .padding(.horizontal)
        }
    }
    
    private func sectionHeader(_ title: String, subtitle: String) -> some View {
        VStack(alignment: .leading, spacing: 2) {
            Text(title)
                .font(.title2)
                .fontWeight(.bold)
            Text(subtitle)
                .font(.subheadline)
                .foregroundColor(.gray)
        }
        .padding(.horizontal)
    }
}

struct FeaturedAppCard: View {
    let app: AppStoreLayout.AppItem
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            // Hero image
            RoundedRectangle(cornerRadius: 16)
                .fill(
                    LinearGradient(
                        colors: [app.color, app.color.opacity(0.6)],
                        startPoint: .topLeading,
                        endPoint: .bottomTrailing
                    )
                )
                .frame(height: 180)
                .overlay(
                    Image(systemName: app.icon)
                        .font(.system(size: 60))
                        .foregroundColor(.white.opacity(0.8))
                )
            
            // Info
            HStack {
                VStack(alignment: .leading, spacing: 2) {
                    Text(app.category)
                        .font(.caption)
                        .foregroundColor(.gray)
                    
                    Text(app.name)
                        .font(.headline)
                    
                    Text(app.developer)
                        .font(.caption)
                        .foregroundColor(.gray)
                }
                
                Spacer()
                
                Button(app.price) { }
                    .buttonStyle(.bordered)
                    .font(.subheadline)
                    .fontWeight(.semibold)
            }
        }
        .frame(width: 280)
    }
}

struct AppListRow: View {
    let app: AppStoreLayout.AppItem
    let rank: Int
    
    var body: some View {
        HStack(spacing: 12) {
            Text("\(rank)")
                .font(.headline)
                .foregroundColor(.gray)
                .frame(width: 20)
            
            RoundedRectangle(cornerRadius: 12)
                .fill(app.color.opacity(0.2))
                .frame(width: 52, height: 52)
                .overlay(
                    Image(systemName: app.icon)
                        .foregroundColor(app.color)
                        .font(.title3)
                )
            
            VStack(alignment: .leading, spacing: 2) {
                Text(app.name)
                    .font(.subheadline)
                    .fontWeight(.medium)
                Text(app.developer)
                    .font(.caption)
                    .foregroundColor(.gray)
                
                // Star rating
                HStack(spacing: 2) {
                    ForEach(1...5, id: \.self) { i in
                        Image(systemName: Double(i) <= app.rating ? "star.fill" : "star")
                            .font(.system(size: 8))
                            .foregroundColor(.yellow)
                    }
                    Text(String(format: "%.1f", app.rating))
                        .font(.caption2)
                        .foregroundColor(.gray)
                }
            }
            
            Spacer()
            
            Button(app.price) { }
                .buttonStyle(.bordered)
                .font(.caption)
                .fontWeight(.semibold)
        }
        .padding()
    }
}

struct CategoryCard: View {
    let name: String
    let icon: String
    let color: Color
    
    var body: some View {
        HStack {
            Image(systemName: icon)
                .font(.title3)
                .foregroundColor(color)
                .frame(width: 44, height: 44)
                .background(color.opacity(0.15))
                .cornerRadius(10)
            
            Text(name)
                .font(.subheadline)
                .fontWeight(.medium)
            
            Spacer()
            
            Image(systemName: "chevron.right")
                .font(.caption)
                .foregroundColor(.gray)
        }
        .padding()
        .background(Color.white)
        .cornerRadius(12)
        .shadow(radius: 2)
    }
}

#Preview("App Store Layout") {
    AppStoreLayout()
}
```

---

## สรุป

ในบทนี้เราได้เรียนรู้ SwiftUI Layout ขั้นสูงครอบคลุม:

### สิ่งที่เรียนรู้

**Stack Containers:**
- `HStack` - จัดเรียงแนวนอน
- `VStack` - จัดเรียงแนวตั้ง
- `ZStack` - ซ้อนทับกัน
- `LazyHStack/LazyVStack` - ประหยัด memory สำหรับ list ยาว

**Grid Layouts:**
- `LazyVGrid` - Grid แนวตั้งใน ScrollView
- `LazyHGrid` - Grid แนวนอนใน ScrollView
- `GridItem` - กำหนดรูปแบบของ column/row (.fixed, .flexible, .adaptive)

**Advanced Layout:**
- `GeometryReader` - อ่านขนาดและตำแหน่ง
- `alignmentGuide` - ปรับ alignment แบบ custom
- `ViewThatFits` - เลือก View ที่พอดีอัตโนมัติ
- `AnyLayout` - สลับ layout type ด้วย animation
- `Layout protocol` - สร้าง layout เอง

**Sizing:**
- `padding` - ระยะขอบ
- `frame` - กำหนดขนาด
- `fixedSize` - ป้องกันการ shrink
- `ignoresSafeArea` - ขยายออกนอก safe area
- `safeAreaInset` - เพิ่ม inset ให้ safe area

**Content Views:**
- `ScrollView` - เลื่อนดูเนื้อหา
- `ScrollViewReader` - scroll ไปยัง View ที่ต้องการ
- `List` - แสดงรายการ
- `ForEach` - สร้าง View จาก collection
- `Section` - จัดกลุ่มใน List

### Best Practices

```swift
// 1. ใช้ LazyStack แทน Stack สำหรับ list ยาว
// ไม่ดี
ScrollView {
    VStack {
        ForEach(1...1000, id: \.self) { i in
            Text("\(i)")
        }
    }
}

// ดี
ScrollView {
    LazyVStack {
        ForEach(1...1000, id: \.self) { i in
            Text("\(i)")
        }
    }
}

// 2. ใช้ .adaptive ใน GridItem สำหรับ responsive grid
LazyVGrid(columns: [GridItem(.adaptive(minimum: 150))]) {
    // จะปรับจำนวน column ตามหน้าจออัตโนมัติ
}

// 3. ใช้ GeometryReader เฉพาะตอนจำเป็น
// GeometryReader ทำให้ View ขยายเต็มพื้นที่เสมอ ระวังการใช้

// 4. ใช้ ViewThatFits สำหรับ adaptive UI
ViewThatFits {
    HStack { /* horizontal layout */ }
    VStack { /* vertical layout */ }
}
```

### ขั้นตอนต่อไป

- State Management ขั้นสูง (@StateObject, @EnvironmentObject)
- Navigation ขั้นสูง (NavigationStack, NavigationSplitView)
- Animation และ Transitions
- Gestures
- Custom Drawing (Canvas, Path)

---

*บทต่อไป: ตอนที่ 23 - State Management ใน SwiftUI*
