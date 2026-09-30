# Part 73: Advanced SwiftUI

## บทนำ

ในบทนี้เราจะเจาะลึกเทคนิค SwiftUI ขั้นสูงที่ช่วยให้คุณสร้าง UI ที่ซับซ้อน มีประสิทธิภาพ และยืดหยุ่นสูง เนื้อหาครอบคลุมตั้งแต่การสร้าง custom view modifiers ไปจนถึงการทำ interop กับ UIKit/AppKit และการจัดการ focus state อย่างมืออาชีพ

---

## 1. Custom View Modifiers แบบเจาะลึก

### ViewModifier Protocol

`ViewModifier` คือ protocol ที่ช่วยให้เราสร้าง modifier แบบ reusable ได้ การสร้าง custom modifier ที่ดีควรมีความชัดเจนในหน้าที่และนำกลับมาใช้ใหม่ได้

```swift
import SwiftUI

// Basic custom modifier
struct CardStyle: ViewModifier {
    var cornerRadius: CGFloat = 12
    var shadowRadius: CGFloat = 8
    
    func body(content: Content) -> some View {
        content
            .padding()
            .background(Color(.systemBackground))
            .cornerRadius(cornerRadius)
            .shadow(
                color: .black.opacity(0.1),
                radius: shadowRadius,
                x: 0,
                y: 4
            )
    }
}

extension View {
    func cardStyle(
        cornerRadius: CGFloat = 12,
        shadowRadius: CGFloat = 8
    ) -> some View {
        modifier(CardStyle(
            cornerRadius: cornerRadius,
            shadowRadius: shadowRadius
        ))
    }
}

// การใช้งาน
struct ContentView: View {
    var body: some View {
        VStack(spacing: 16) {
            Text("Card 1")
                .cardStyle()
            
            Text("Card 2")
                .cardStyle(cornerRadius: 20, shadowRadius: 12)
        }
        .padding()
    }
}
```

### Animated ViewModifier

```swift
// Modifier ที่มี animation
struct ShakeEffect: ViewModifier {
    var times: CGFloat
    var delta: CGFloat = 8
    
    func body(content: Content) -> some View {
        content
            .offset(x: sin(times * .pi * 2) * delta)
    }
}

extension View {
    func shake(times: CGFloat) -> some View {
        modifier(ShakeEffect(times: times))
    }
}

// ใช้กับ animation
struct ShakeDemo: View {
    @State private var shakeTimes: CGFloat = 0
    
    var body: some View {
        VStack {
            TextField("Enter password", text: .constant(""))
                .textFieldStyle(.roundedBorder)
                .shake(times: shakeTimes)
            
            Button("Shake") {
                withAnimation(.linear(duration: 0.5)) {
                    shakeTimes += 1
                }
            }
        }
        .padding()
    }
}
```

### Conditional Modifier

```swift
// Modifier ที่ใช้ได้กับเงื่อนไข
extension View {
    @ViewBuilder
    func `if`<Transform: View>(
        _ condition: Bool,
        transform: (Self) -> Transform
    ) -> some View {
        if condition {
            transform(self)
        } else {
            self
        }
    }
    
    @ViewBuilder
    func ifLet<Value, Transform: View>(
        _ value: Value?,
        transform: (Self, Value) -> Transform
    ) -> some View {
        if let value = value {
            transform(self, value)
        } else {
            self
        }
    }
}

struct ConditionalDemo: View {
    @State private var isHighlighted = false
    @State private var badge: Int? = nil
    
    var body: some View {
        VStack {
            Text("Hello")
                .if(isHighlighted) { view in
                    view.foregroundColor(.yellow)
                        .background(Color.blue)
                }
            
            Image(systemName: "bell")
                .ifLet(badge) { image, count in
                    image.overlay(
                        Text("\(count)")
                            .font(.caption2)
                            .foregroundColor(.white)
                            .padding(4)
                            .background(Color.red)
                            .clipShape(Circle())
                            .offset(x: 8, y: -8),
                        alignment: .topTrailing
                    )
                }
        }
    }
}
```

### Chainable Modifier Pattern

```swift
// Pattern สำหรับ modifier ที่ chain ได้
struct StyleConfiguration {
    var backgroundColor: Color = .white
    var foregroundColor: Color = .primary
    var font: Font = .body
    var padding: CGFloat = 8
    var cornerRadius: CGFloat = 8
}

struct StyledView: ViewModifier {
    let config: StyleConfiguration
    
    func body(content: Content) -> some View {
        content
            .font(config.font)
            .foregroundColor(config.foregroundColor)
            .padding(config.padding)
            .background(config.backgroundColor)
            .cornerRadius(config.cornerRadius)
    }
}

// Builder pattern
class StyleBuilder {
    private var config = StyleConfiguration()
    
    func background(_ color: Color) -> Self {
        config.backgroundColor = color
        return self
    }
    
    func foreground(_ color: Color) -> Self {
        config.foregroundColor = color
        return self
    }
    
    func font(_ font: Font) -> Self {
        config.font = font
        return self
    }
    
    func build() -> StyledView {
        StyledView(config: config)
    }
}
```

---

## 2. Custom Container Views กับ ViewBuilder

### ViewBuilder คืออะไร

`@ViewBuilder` เป็น result builder ที่ทำให้เราสร้าง view hierarchy แบบ declarative ได้ ใช้ใน SwiftUI ทุกที่ที่ต้องการ combine multiple views

```swift
import SwiftUI

// Custom container view ด้วย ViewBuilder
struct Section<Content: View>: View {
    let title: String
    let content: Content
    
    init(
        title: String,
        @ViewBuilder content: () -> Content
    ) {
        self.title = title
        self.content = content()
    }
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            Text(title)
                .font(.headline)
                .foregroundColor(.secondary)
            
            content
                .padding()
                .background(Color(.systemGray6))
                .cornerRadius(12)
        }
    }
}

// การใช้งาน
struct SectionDemo: View {
    var body: some View {
        ScrollView {
            VStack(spacing: 20) {
                Section(title: "Personal Info") {
                    HStack {
                        Image(systemName: "person")
                        Text("John Doe")
                    }
                    HStack {
                        Image(systemName: "envelope")
                        Text("john@example.com")
                    }
                }
                
                Section(title: "Settings") {
                    Toggle("Notifications", isOn: .constant(true))
                    Toggle("Dark Mode", isOn: .constant(false))
                }
            }
            .padding()
        }
    }
}
```

### Generic Container ขั้นสูง

```swift
// Container ที่รับหลาย content type
struct Card<Header: View, Body: View, Footer: View>: View {
    let header: Header
    let body: Body
    let footer: Footer
    
    init(
        @ViewBuilder header: () -> Header,
        @ViewBuilder body: () -> Body,
        @ViewBuilder footer: () -> Footer
    ) {
        self.header = header()
        self.body = body()
        self.footer = footer()
    }
    
    var body: some View {
        VStack(spacing: 0) {
            header
                .padding()
                .frame(maxWidth: .infinity)
                .background(Color.blue.opacity(0.1))
            
            Divider()
            
            body
                .padding()
            
            Divider()
            
            footer
                .padding()
                .frame(maxWidth: .infinity)
                .background(Color.gray.opacity(0.05))
        }
        .background(Color(.systemBackground))
        .cornerRadius(16)
        .shadow(radius: 8)
    }
}

// การใช้งาน
struct CardDemo: View {
    var body: some View {
        Card {
            Text("Card Title")
                .font(.headline)
        } body: {
            VStack(alignment: .leading) {
                Text("Card content goes here")
                Text("More content")
            }
        } footer: {
            HStack {
                Button("Cancel") {}
                Spacer()
                Button("Confirm") {}
            }
        }
        .padding()
    }
}
```

### Conditional ViewBuilder

```swift
// Container ที่ handle nil content ได้
struct OptionalSection<Content: View>: View {
    let title: String
    let items: [String]
    let content: (String) -> Content
    
    var body: some View {
        if items.isEmpty {
            EmptyStateView(title: title)
        } else {
            VStack(alignment: .leading) {
                Text(title).font(.headline)
                ForEach(items, id: \.self) { item in
                    content(item)
                }
            }
        }
    }
}

struct EmptyStateView: View {
    let title: String
    
    var body: some View {
        VStack {
            Image(systemName: "tray")
                .font(.largeTitle)
                .foregroundColor(.secondary)
            Text("No \(title)")
                .foregroundColor(.secondary)
        }
        .frame(maxWidth: .infinity)
        .padding()
    }
}
```

---

## 3. PreferenceKey แบบเจาะลึก

### ทำความเข้าใจ PreferenceKey

`PreferenceKey` ช่วยให้ child view ส่งข้อมูลขึ้นไปหา parent view ได้ ซึ่งตรงข้ามกับ `Environment` ที่ส่งข้อมูลจาก parent ลงไป child

```swift
import SwiftUI

// Basic PreferenceKey
struct HeightPreferenceKey: PreferenceKey {
    static var defaultValue: CGFloat = 0
    
    static func reduce(value: inout CGFloat, nextValue: () -> CGFloat) {
        // เลือกค่าที่ใหญ่ที่สุด
        value = max(value, nextValue())
    }
}

// การใช้งาน
struct EqualHeightHStack: View {
    @State private var maxHeight: CGFloat = 0
    
    var body: some View {
        HStack(alignment: .top) {
            CardContent(title: "Short", color: .blue)
                .frame(height: maxHeight)
            
            CardContent(title: "Medium\nTwo lines", color: .green)
                .frame(height: maxHeight)
            
            CardContent(title: "Long\nThree\nlines", color: .orange)
                .frame(height: maxHeight)
        }
        .onPreferenceChange(HeightPreferenceKey.self) { height in
            maxHeight = height
        }
    }
}

struct CardContent: View {
    let title: String
    let color: Color
    
    var body: some View {
        Text(title)
            .padding()
            .frame(maxWidth: .infinity)
            .background(color.opacity(0.2))
            .cornerRadius(8)
            .background(
                GeometryReader { geometry in
                    Color.clear
                        .preference(
                            key: HeightPreferenceKey.self,
                            value: geometry.size.height
                        )
                }
            )
    }
}
```

### PreferenceKey สำหรับ Scroll Detection

```swift
// ติดตาม scroll offset ด้วย PreferenceKey
struct ScrollOffsetPreferenceKey: PreferenceKey {
    static var defaultValue: CGFloat = 0
    
    static func reduce(value: inout CGFloat, nextValue: () -> CGFloat) {
        value = nextValue()
    }
}

struct ScrollOffsetView: View {
    @State private var scrollOffset: CGFloat = 0
    
    var body: some View {
        ScrollView {
            VStack {
                GeometryReader { geometry in
                    Color.clear
                        .preference(
                            key: ScrollOffsetPreferenceKey.self,
                            value: geometry.frame(in: .named("scrollView")).minY
                        )
                }
                .frame(height: 0)
                
                ForEach(0..<20) { index in
                    Text("Item \(index)")
                        .frame(maxWidth: .infinity)
                        .padding()
                        .background(Color.blue.opacity(0.1))
                        .padding(.horizontal)
                }
            }
        }
        .coordinateSpace(name: "scrollView")
        .onPreferenceChange(ScrollOffsetPreferenceKey.self) { offset in
            scrollOffset = offset
        }
        .overlay(
            Text("Offset: \(Int(scrollOffset))")
                .padding()
                .background(.ultraThinMaterial)
                .cornerRadius(8)
                .padding(),
            alignment: .topTrailing
        )
    }
}
```

### การรวม PreferenceKey หลายตัว

```swift
// Collect ข้อมูลจาก multiple children
struct BoundsPreferenceKey: PreferenceKey {
    typealias Value = [String: CGRect]
    
    static var defaultValue: [String: CGRect] = [:]
    
    static func reduce(
        value: inout [String: CGRect],
        nextValue: () -> [String: CGRect]
    ) {
        value.merge(nextValue()) { _, new in new }
    }
}

struct BoundsTracker: View {
    @State private var bounds: [String: CGRect] = [:]
    
    var body: some View {
        GeometryReader { proxy in
            VStack(spacing: 20) {
                ForEach(["Item A", "Item B", "Item C"], id: \.self) { item in
                    Text(item)
                        .padding()
                        .background(Color.blue.opacity(0.2))
                        .anchorPreference(
                            key: BoundsPreferenceKey.self,
                            value: .bounds
                        ) { anchor in
                            [item: proxy[anchor]]
                        }
                }
            }
            .frame(maxWidth: .infinity, maxHeight: .infinity)
        }
        .onPreferenceChange(BoundsPreferenceKey.self) { newBounds in
            bounds = newBounds
        }
        .overlay(
            VStack(alignment: .leading) {
                ForEach(bounds.sorted(by: { $0.key < $1.key }), id: \.key) { key, rect in
                    Text("\(key): x=\(Int(rect.minX)), y=\(Int(rect.minY))")
                        .font(.caption)
                }
            }
            .padding()
            .background(.ultraThinMaterial)
            .cornerRadius(8)
            .padding(),
            alignment: .bottomLeading
        )
    }
}
```

---

## 4. AnchorPreference

### ความแตกต่างระหว่าง PreferenceKey และ AnchorPreference

`AnchorPreference` ทำงานคล้าย `PreferenceKey` แต่ใช้ `Anchor<Value>` แทน ซึ่ง Anchor จะถูก resolve ในบริบทของ coordinate space ที่ถูกต้อง ทำให้ได้ค่าที่แม่นยำกว่า

```swift
import SwiftUI

// AnchorPreference สำหรับ highlight effect
struct SelectionAnchorKey: PreferenceKey {
    typealias Value = Anchor<CGRect>?
    
    static var defaultValue: Value = nil
    
    static func reduce(value: inout Value, nextValue: () -> Value) {
        value = nextValue() ?? value
    }
}

struct SelectableList: View {
    @State private var selectedIndex: Int? = nil
    
    let items = ["Swift", "SwiftUI", "Combine", "UIKit", "Core Data"]
    
    var body: some View {
        VStack(alignment: .leading, spacing: 0) {
            ForEach(Array(items.enumerated()), id: \.offset) { index, item in
                Text(item)
                    .padding()
                    .frame(maxWidth: .infinity, alignment: .leading)
                    .contentShape(Rectangle())
                    .onTapGesture {
                        withAnimation(.spring()) {
                            selectedIndex = selectedIndex == index ? nil : index
                        }
                    }
                    .anchorPreference(
                        key: SelectionAnchorKey.self,
                        value: .bounds
                    ) { anchor in
                        selectedIndex == index ? anchor : nil
                    }
            }
        }
        .overlayPreferenceValue(SelectionAnchorKey.self) { anchor in
            if let anchor = anchor {
                GeometryReader { proxy in
                    let rect = proxy[anchor]
                    RoundedRectangle(cornerRadius: 8)
                        .fill(Color.blue.opacity(0.2))
                        .frame(width: rect.width, height: rect.height)
                        .position(
                            x: rect.midX,
                            y: rect.midY
                        )
                }
            }
        }
    }
}
```

### ใช้ AnchorPreference สำหรับ Tooltip

```swift
// Tooltip system ด้วย AnchorPreference
struct TooltipAnchorKey: PreferenceKey {
    typealias Value = [(id: AnyHashable, anchor: Anchor<CGRect>, text: String)]
    
    static var defaultValue: Value = []
    
    static func reduce(value: inout Value, nextValue: () -> Value) {
        value.append(contentsOf: nextValue())
    }
}

struct TooltipModifier: ViewModifier {
    let id: AnyHashable
    let text: String
    
    func body(content: Content) -> some View {
        content
            .anchorPreference(
                key: TooltipAnchorKey.self,
                value: .bounds
            ) { anchor in
                [(id: id, anchor: anchor, text: text)]
            }
    }
}

extension View {
    func tooltip(id: AnyHashable, text: String) -> some View {
        modifier(TooltipModifier(id: id, text: text))
    }
}

struct TooltipContainer<Content: View>: View {
    @State private var activeTooltip: AnyHashable? = nil
    let content: Content
    
    init(@ViewBuilder content: () -> Content) {
        self.content = content()
    }
    
    var body: some View {
        content
            .overlayPreferenceValue(TooltipAnchorKey.self) { tooltips in
                GeometryReader { proxy in
                    ForEach(tooltips, id: \.id) { tooltip in
                        if activeTooltip == tooltip.id {
                            let rect = proxy[tooltip.anchor]
                            Text(tooltip.text)
                                .font(.caption)
                                .padding(8)
                                .background(Color.black.opacity(0.8))
                                .foregroundColor(.white)
                                .cornerRadius(6)
                                .position(
                                    x: rect.midX,
                                    y: rect.minY - 30
                                )
                        }
                    }
                }
            }
    }
}
```

---

## 5. Custom Coordinate Spaces

### สร้าง Named Coordinate Space

Coordinate spaces ช่วยให้เราระบุตำแหน่งของ view ในบริบทต่างๆ ได้อย่างแม่นยำ

```swift
import SwiftUI

// Named coordinate space สำหรับ custom scroll behavior
struct ParallaxScrollView: View {
    var body: some View {
        ScrollView {
            VStack(spacing: 0) {
                // Parallax Header
                GeometryReader { geometry in
                    let frame = geometry.frame(in: .named("scroll"))
                    let offset = frame.minY
                    
                    Image("hero")
                        .resizable()
                        .scaledToFill()
                        .frame(
                            width: geometry.size.width,
                            height: max(300, 300 + offset)
                        )
                        .clipped()
                        .offset(y: min(0, -offset / 2))
                }
                .frame(height: 300)
                
                // Content
                VStack(alignment: .leading, spacing: 16) {
                    ForEach(0..<10) { i in
                        Text("Content item \(i)")
                            .frame(maxWidth: .infinity, alignment: .leading)
                            .padding()
                            .background(Color.gray.opacity(0.1))
                            .cornerRadius(8)
                    }
                }
                .padding()
            }
        }
        .coordinateSpace(name: "scroll")
    }
}
```

### Custom Coordinate Space สำหรับ Drag

```swift
// ติดตามตำแหน่ง drag ใน coordinate space
struct DragTracker: View {
    @State private var dragLocation: CGPoint = .zero
    @State private var isDragging = false
    
    var body: some View {
        ZStack {
            // Background
            Color.gray.opacity(0.1)
                .coordinateSpace(name: "dragArea")
            
            // Draggable item
            Circle()
                .fill(Color.blue)
                .frame(width: 50, height: 50)
                .position(dragLocation)
                .gesture(
                    DragGesture(coordinateSpace: .named("dragArea"))
                        .onChanged { value in
                            isDragging = true
                            dragLocation = value.location
                        }
                        .onEnded { _ in
                            isDragging = false
                        }
                )
            
            // Position indicator
            if isDragging {
                Text("x: \(Int(dragLocation.x)), y: \(Int(dragLocation.y))")
                    .font(.caption)
                    .padding(8)
                    .background(.ultraThinMaterial)
                    .cornerRadius(8)
                    .position(x: dragLocation.x, y: dragLocation.y - 40)
            }
        }
        .frame(maxWidth: .infinity, maxHeight: 400)
        .onAppear {
            dragLocation = CGPoint(x: 150, y: 200)
        }
    }
}
```

---

## 6. Custom Alignment Guides กับ AlignmentID

### AlignmentID Protocol

`AlignmentID` ช่วยให้เราสร้าง custom alignment ที่ทำให้ views จาก sibling containers จัดเรียงกันได้

```swift
import SwiftUI

// Custom alignment ID
private struct IconAlignmentID: AlignmentID {
    static func defaultValue(in context: ViewDimensions) -> CGFloat {
        context[.leading]
    }
}

extension HorizontalAlignment {
    static let iconAlignment = HorizontalAlignment(IconAlignmentID.self)
}

// การใช้งาน
struct AlignedList: View {
    let items = [
        ("house", "Home", "Your main feed"),
        ("magnifyingglass", "Search", "Find people and posts"),
        ("bell", "Notifications", "Your activity"),
        ("person", "Profile", "View your profile")
    ]
    
    var body: some View {
        VStack(alignment: .iconAlignment, spacing: 16) {
            ForEach(items, id: \.1) { icon, title, subtitle in
                HStack(spacing: 12) {
                    Image(systemName: icon)
                        .frame(width: 30)
                        .alignmentGuide(.iconAlignment) { d in
                            d[HorizontalAlignment.center]
                        }
                    
                    VStack(alignment: .leading) {
                        Text(title)
                            .font(.headline)
                        Text(subtitle)
                            .font(.caption)
                            .foregroundColor(.secondary)
                    }
                }
            }
        }
        .padding()
    }
}
```

### Vertical Alignment Custom Guide

```swift
// Custom vertical alignment
private struct FirstTextAlignmentID: AlignmentID {
    static func defaultValue(in context: ViewDimensions) -> CGFloat {
        context[VerticalAlignment.firstTextBaseline]
    }
}

extension VerticalAlignment {
    static let firstText = VerticalAlignment(FirstTextAlignmentID.self)
}

struct BaselineAlignment: View {
    var body: some View {
        HStack(alignment: .firstText) {
            // Icon อยากให้ align กับ text แรก
            Image(systemName: "star.fill")
                .foregroundColor(.yellow)
                .alignmentGuide(.firstText) { d in
                    d[VerticalAlignment.center]
                }
            
            VStack(alignment: .leading) {
                Text("Featured")
                    .font(.headline)
                Text("This is a featured item")
                    .font(.body)
                Text("Additional info")
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
        }
    }
}
```

---

## 7. Drawing Custom Shapes กับ Shape Protocol

### Shape Protocol เบื้องต้น

```swift
import SwiftUI

// Custom shape ง่ายๆ
struct Triangle: Shape {
    func path(in rect: CGRect) -> Path {
        var path = Path()
        
        path.move(to: CGPoint(x: rect.midX, y: rect.minY))
        path.addLine(to: CGPoint(x: rect.maxX, y: rect.maxY))
        path.addLine(to: CGPoint(x: rect.minX, y: rect.maxY))
        path.closeSubpath()
        
        return path
    }
}

// Star shape
struct Star: Shape {
    let points: Int
    let innerRatio: CGFloat
    
    init(points: Int = 5, innerRatio: CGFloat = 0.4) {
        self.points = points
        self.innerRatio = innerRatio
    }
    
    func path(in rect: CGRect) -> Path {
        let center = CGPoint(x: rect.midX, y: rect.midY)
        let outerRadius = min(rect.width, rect.height) / 2
        let innerRadius = outerRadius * innerRatio
        
        var path = Path()
        
        for i in 0..<points * 2 {
            let angle = Double(i) * .pi / Double(points) - .pi / 2
            let radius = i.isMultiple(of: 2) ? outerRadius : innerRadius
            let point = CGPoint(
                x: center.x + CGFloat(cos(angle)) * radius,
                y: center.y + CGFloat(sin(angle)) * radius
            )
            
            if i == 0 {
                path.move(to: point)
            } else {
                path.addLine(to: point)
            }
        }
        
        path.closeSubpath()
        return path
    }
}

// Bubble shape
struct SpeechBubble: Shape {
    var tailPosition: CGFloat = 0.5
    var tailHeight: CGFloat = 20
    var cornerRadius: CGFloat = 16
    
    func path(in rect: CGRect) -> Path {
        let bodyRect = CGRect(
            x: rect.minX,
            y: rect.minY,
            width: rect.width,
            height: rect.height - tailHeight
        )
        
        var path = Path(
            roundedRect: bodyRect,
            cornerRadius: cornerRadius
        )
        
        let tailX = bodyRect.minX + bodyRect.width * tailPosition
        
        path.move(to: CGPoint(x: tailX - 10, y: bodyRect.maxY))
        path.addLine(to: CGPoint(x: tailX, y: rect.maxY))
        path.addLine(to: CGPoint(x: tailX + 10, y: bodyRect.maxY))
        
        return path
    }
}

// การแสดงผล
struct ShapesDemo: View {
    var body: some View {
        VStack(spacing: 30) {
            Triangle()
                .fill(Color.blue)
                .frame(width: 100, height: 100)
            
            Star(points: 5)
                .fill(Color.yellow)
                .frame(width: 100, height: 100)
            
            Star(points: 6, innerRatio: 0.5)
                .stroke(Color.orange, lineWidth: 2)
                .frame(width: 100, height: 100)
            
            SpeechBubble()
                .fill(Color.green.opacity(0.3))
                .frame(width: 200, height: 80)
                .overlay(
                    Text("Hello!")
                        .offset(y: -10)
                )
        }
    }
}
```

---

## 8. AnimatableData Protocol

### ทำให้ Shape Animatable

```swift
import SwiftUI

// Animatable shape - wave
struct WaveShape: Shape, Animatable {
    var phase: CGFloat
    var amplitude: CGFloat
    var frequency: CGFloat
    
    var animatableData: AnimatablePair<CGFloat, CGFloat> {
        get { AnimatablePair(phase, amplitude) }
        set {
            phase = newValue.first
            amplitude = newValue.second
        }
    }
    
    func path(in rect: CGRect) -> Path {
        var path = Path()
        let width = rect.width
        let height = rect.height
        let midHeight = height / 2
        
        path.move(to: CGPoint(x: 0, y: midHeight))
        
        for x in stride(from: 0, through: width, by: 1) {
            let angle = (x / width) * frequency * 2 * .pi + phase
            let y = midHeight + amplitude * sin(angle)
            path.addLine(to: CGPoint(x: x, y: y))
        }
        
        path.addLine(to: CGPoint(x: width, y: height))
        path.addLine(to: CGPoint(x: 0, y: height))
        path.closeSubpath()
        
        return path
    }
}

// Animated wave view
struct AnimatedWave: View {
    @State private var phase: CGFloat = 0
    
    var body: some View {
        ZStack {
            WaveShape(phase: phase, amplitude: 20, frequency: 3)
                .fill(Color.blue.opacity(0.5))
            
            WaveShape(phase: phase + 1, amplitude: 25, frequency: 2)
                .fill(Color.blue.opacity(0.3))
            
            WaveShape(phase: phase - 0.5, amplitude: 15, frequency: 4)
                .fill(Color.blue.opacity(0.7))
        }
        .frame(height: 100)
        .onAppear {
            withAnimation(.linear(duration: 2).repeatForever(autoreverses: false)) {
                phase = .pi * 2
            }
        }
    }
}
```

### Morphing Shapes

```swift
// Shape ที่ morph ระหว่างสองรูปร่าง
struct MorphShape: Shape, Animatable {
    var progress: CGFloat // 0 = circle, 1 = square
    
    var animatableData: CGFloat {
        get { progress }
        set { progress = newValue }
    }
    
    func path(in rect: CGRect) -> Path {
        let circlePath = Circle().path(in: rect)
        let squarePath = RoundedRectangle(cornerRadius: 4).path(in: rect)
        
        // ใช้ CGPath interpolation
        return interpolatedPath(
            from: circlePath,
            to: squarePath,
            progress: progress
        )
    }
    
    private func interpolatedPath(
        from: Path,
        to: Path,
        progress: CGFloat
    ) -> Path {
        // Simplified interpolation - ในงานจริงใช้ library
        let t = progress
        var result = Path()
        
        let fromElements = from.elements
        let toElements = to.elements
        
        let count = min(fromElements.count, toElements.count)
        
        for i in 0..<count {
            let fromElement = fromElements[i]
            let toElement = toElements[i]
            
            switch (fromElement, toElement) {
            case (.move(let p1), .move(let p2)):
                result.move(to: interpolate(p1, p2, t: t))
            case (.line(let p1), .line(let p2)):
                result.addLine(to: interpolate(p1, p2, t: t))
            case (.curve(let to1, let cp1a, let cp1b), .curve(let to2, let cp2a, let cp2b)):
                result.addCurve(
                    to: interpolate(to1, to2, t: t),
                    control1: interpolate(cp1a, cp2a, t: t),
                    control2: interpolate(cp1b, cp2b, t: t)
                )
            case (.closeSubpath, .closeSubpath):
                result.closeSubpath()
            default:
                break
            }
        }
        
        return result
    }
    
    private func interpolate(_ p1: CGPoint, _ p2: CGPoint, t: CGFloat) -> CGPoint {
        CGPoint(
            x: p1.x + (p2.x - p1.x) * t,
            y: p1.y + (p2.y - p1.y) * t
        )
    }
}

struct MorphDemo: View {
    @State private var progress: CGFloat = 0
    
    var body: some View {
        VStack {
            MorphShape(progress: progress)
                .fill(Color.purple)
                .frame(width: 150, height: 150)
            
            Slider(value: $progress)
                .padding()
            
            Button("Animate") {
                withAnimation(.easeInOut(duration: 1.5)) {
                    progress = progress < 0.5 ? 1 : 0
                }
            }
        }
    }
}
```

---

## 9. Custom View Transitions กับ ViewModifier

### สร้าง Custom Transition

```swift
import SwiftUI

// Custom transition - slide and fade
struct SlideAndFadeModifier: ViewModifier {
    let isActive: Bool
    let direction: Edge
    
    func body(content: Content) -> some View {
        content
            .opacity(isActive ? 1 : 0)
            .offset(
                x: isActive ? 0 : offsetX,
                y: isActive ? 0 : offsetY
            )
    }
    
    private var offsetX: CGFloat {
        switch direction {
        case .leading: return -50
        case .trailing: return 50
        default: return 0
        }
    }
    
    private var offsetY: CGFloat {
        switch direction {
        case .top: return -50
        case .bottom: return 50
        default: return 0
        }
    }
}

extension AnyTransition {
    static func slideAndFade(from direction: Edge = .leading) -> AnyTransition {
        .modifier(
            active: SlideAndFadeModifier(isActive: false, direction: direction),
            identity: SlideAndFadeModifier(isActive: true, direction: direction)
        )
    }
}

// Flip transition
struct FlipModifier: ViewModifier {
    let angle: Double
    let axis: (x: CGFloat, y: CGFloat, z: CGFloat)
    
    func body(content: Content) -> some View {
        content
            .rotation3DEffect(
                .degrees(angle),
                axis: axis,
                perspective: 0.5
            )
    }
}

extension AnyTransition {
    static var flipFromLeft: AnyTransition {
        .asymmetric(
            insertion: .modifier(
                active: FlipModifier(angle: -90, axis: (0, 1, 0)),
                identity: FlipModifier(angle: 0, axis: (0, 1, 0))
            ),
            removal: .modifier(
                active: FlipModifier(angle: 90, axis: (0, 1, 0)),
                identity: FlipModifier(angle: 0, axis: (0, 1, 0))
            )
        )
    }
}

// การใช้งาน
struct TransitionDemo: View {
    @State private var showCard = true
    @State private var selectedTransition = 0
    
    var body: some View {
        VStack {
            Picker("Transition", selection: $selectedTransition) {
                Text("Slide").tag(0)
                Text("Flip").tag(1)
                Text("Scale").tag(2)
            }
            .pickerStyle(.segmented)
            .padding()
            
            if showCard {
                RoundedRectangle(cornerRadius: 16)
                    .fill(Color.blue)
                    .frame(width: 200, height: 200)
                    .overlay(Text("Card").foregroundColor(.white).font(.title))
                    .transition(transition)
            }
            
            Button("Toggle") {
                withAnimation(.spring()) {
                    showCard.toggle()
                }
            }
            .padding()
        }
    }
    
    var transition: AnyTransition {
        switch selectedTransition {
        case 0: return .slideAndFade(from: .leading)
        case 1: return .flipFromLeft
        default: return .scale.combined(with: .opacity)
        }
    }
}
```

---

## 10. Custom Layout Protocol แบบเจาะลึก

### Layout Protocol เบื้องต้น

`Layout` protocol (iOS 16+) ช่วยให้เราสร้าง custom layout container โดยกำหนดวิธีที่ subviews จัดเรียงตัวเอง

```swift
import SwiftUI

// Flow Layout - จัดเรียงแบบ word wrap
struct FlowLayout: Layout {
    var spacing: CGFloat = 8
    
    func sizeThatFits(
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout Void
    ) -> CGSize {
        let maxWidth = proposal.width ?? .infinity
        var height: CGFloat = 0
        var currentRowWidth: CGFloat = 0
        var currentRowHeight: CGFloat = 0
        
        for subview in subviews {
            let size = subview.sizeThatFits(.unspecified)
            
            if currentRowWidth + size.width > maxWidth && currentRowWidth > 0 {
                height += currentRowHeight + spacing
                currentRowWidth = 0
                currentRowHeight = 0
            }
            
            currentRowWidth += size.width + spacing
            currentRowHeight = max(currentRowHeight, size.height)
        }
        
        height += currentRowHeight
        
        return CGSize(width: maxWidth, height: height)
    }
    
    func placeSubviews(
        in bounds: CGRect,
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout Void
    ) {
        var x = bounds.minX
        var y = bounds.minY
        var rowHeight: CGFloat = 0
        
        for subview in subviews {
            let size = subview.sizeThatFits(.unspecified)
            
            if x + size.width > bounds.maxX && x > bounds.minX {
                y += rowHeight + spacing
                x = bounds.minX
                rowHeight = 0
            }
            
            subview.place(
                at: CGPoint(x: x, y: y),
                anchor: .topLeading,
                proposal: ProposedViewSize(size)
            )
            
            x += size.width + spacing
            rowHeight = max(rowHeight, size.height)
        }
    }
}

// การใช้งาน
struct TagCloud: View {
    let tags = [
        "Swift", "SwiftUI", "iOS", "Apple", "Xcode",
        "UIKit", "Combine", "CoreData", "CloudKit",
        "ARKit", "Metal", "CoreML", "Vision"
    ]
    
    var body: some View {
        FlowLayout(spacing: 8) {
            ForEach(tags, id: \.self) { tag in
                Text(tag)
                    .font(.caption)
                    .padding(.horizontal, 12)
                    .padding(.vertical, 6)
                    .background(Color.blue.opacity(0.15))
                    .foregroundColor(.blue)
                    .cornerRadius(20)
            }
        }
        .padding()
    }
}
```

### Masonry Layout

```swift
// Masonry Layout สำหรับ Pinterest-style grid
struct MasonryLayout: Layout {
    var columns: Int
    var spacing: CGFloat = 8
    
    func sizeThatFits(
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout [CGFloat]
    ) -> CGSize {
        let columnWidth = columnWidth(for: proposal.width ?? 300)
        var columnHeights = Array(repeating: CGFloat(0), count: columns)
        
        for subview in subviews {
            let columnIndex = shortestColumnIndex(columnHeights: columnHeights)
            let size = subview.sizeThatFits(
                ProposedViewSize(width: columnWidth, height: nil)
            )
            
            if columnHeights[columnIndex] > 0 {
                columnHeights[columnIndex] += spacing
            }
            columnHeights[columnIndex] += size.height
        }
        
        cache = columnHeights
        
        return CGSize(
            width: proposal.width ?? 0,
            height: columnHeights.max() ?? 0
        )
    }
    
    func placeSubviews(
        in bounds: CGRect,
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout [CGFloat]
    ) {
        let colWidth = columnWidth(for: bounds.width)
        var columnHeights = Array(repeating: bounds.minY, count: columns)
        
        for subview in subviews {
            let columnIndex = shortestColumnIndex(columnHeights: columnHeights)
            let x = bounds.minX + CGFloat(columnIndex) * (colWidth + spacing)
            let y = columnHeights[columnIndex]
            
            let size = subview.sizeThatFits(
                ProposedViewSize(width: colWidth, height: nil)
            )
            
            subview.place(
                at: CGPoint(x: x, y: y),
                anchor: .topLeading,
                proposal: ProposedViewSize(width: colWidth, height: size.height)
            )
            
            columnHeights[columnIndex] += size.height + spacing
        }
    }
    
    func makeCache(subviews: Subviews) -> [CGFloat] {
        Array(repeating: 0, count: columns)
    }
    
    private func columnWidth(for totalWidth: CGFloat) -> CGFloat {
        (totalWidth - spacing * CGFloat(columns - 1)) / CGFloat(columns)
    }
    
    private func shortestColumnIndex(columnHeights: [CGFloat]) -> Int {
        columnHeights.enumerated().min(by: { $0.element < $1.element })?.offset ?? 0
    }
}

// ใช้กับ random height cards
struct MasonryDemo: View {
    let items = (1...20).map { i in
        (id: i, height: CGFloat.random(in: 100...250), color: Color.random)
    }
    
    var body: some View {
        ScrollView {
            MasonryLayout(columns: 2, spacing: 12) {
                ForEach(items, id: \.id) { item in
                    RoundedRectangle(cornerRadius: 12)
                        .fill(item.color.opacity(0.3))
                        .frame(height: item.height)
                        .overlay(
                            Text("Item \(item.id)")
                        )
                }
            }
            .padding()
        }
    }
}

extension Color {
    static var random: Color {
        Color(
            red: .random(in: 0...1),
            green: .random(in: 0...1),
            blue: .random(in: 0...1)
        )
    }
}
```

### Layout ที่มี Animation

```swift
// Radial Layout ที่ animate ได้
struct RadialLayout: Layout {
    var radius: CGFloat
    
    func sizeThatFits(
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout Void
    ) -> CGSize {
        let diameter = radius * 2 + 60 // 60 สำหรับ item size
        return CGSize(width: diameter, height: diameter)
    }
    
    func placeSubviews(
        in bounds: CGRect,
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout Void
    ) {
        let center = CGPoint(x: bounds.midX, y: bounds.midY)
        let count = subviews.count
        
        for (index, subview) in subviews.enumerated() {
            let angle = Double(index) / Double(count) * 2 * .pi - .pi / 2
            let x = center.x + radius * CGFloat(cos(angle))
            let y = center.y + radius * CGFloat(sin(angle))
            
            subview.place(
                at: CGPoint(x: x, y: y),
                anchor: .center,
                proposal: .unspecified
            )
        }
    }
}

struct RadialLayoutDemo: View {
    @State private var radius: CGFloat = 100
    
    let icons = ["house", "magnifyingglass", "bell", "person", "gear",
                 "heart", "bookmark", "share", "trash"]
    
    var body: some View {
        VStack {
            RadialLayout(radius: radius) {
                ForEach(icons, id: \.self) { icon in
                    Image(systemName: icon)
                        .padding(12)
                        .background(Color.blue.opacity(0.2))
                        .clipShape(Circle())
                }
            }
            .animation(.spring(), value: radius)
            
            Slider(value: $radius, in: 50...150)
                .padding()
        }
    }
}
```

---

## 11. GeometryReader Tricks และ Proxy

### GeometryProxy ขั้นสูง

```swift
import SwiftUI

// GeometryReader สำหรับ sticky header
struct StickyHeader: View {
    var body: some View {
        ScrollView {
            LazyVStack(pinnedViews: .sectionHeaders) {
                Section {
                    ForEach(0..<20) { i in
                        Text("Item \(i)")
                            .frame(maxWidth: .infinity)
                            .padding()
                            .background(Color.gray.opacity(0.1))
                    }
                } header: {
                    GeometryReader { geometry in
                        let frame = geometry.frame(in: .global)
                        Text("Section Header")
                            .frame(maxWidth: .infinity)
                            .padding()
                            .background(
                                frame.minY < 50
                                    ? Color.blue.opacity(0.9)
                                    : Color.blue.opacity(0.3)
                            )
                    }
                    .frame(height: 50)
                }
            }
        }
    }
}
```

### Scale Effect ด้วย GeometryReader

```swift
// Card ที่ scale เมื่อ scroll เข้ามา
struct ScrollScaleEffect: View {
    var body: some View {
        ScrollView(.horizontal, showsIndicators: false) {
            HStack(spacing: 20) {
                ForEach(0..<10) { i in
                    GeometryReader { geometry in
                        let frame = geometry.frame(in: .global)
                        let midX = UIScreen.main.bounds.width / 2
                        let distance = abs(frame.midX - midX)
                        let scale = max(0.8, 1 - distance / 500)
                        
                        RoundedRectangle(cornerRadius: 16)
                            .fill(Color(hue: Double(i) / 10, saturation: 0.6, brightness: 0.8))
                            .overlay(
                                Text("Card \(i)")
                                    .foregroundColor(.white)
                                    .font(.headline)
                            )
                            .scaleEffect(scale)
                    }
                    .frame(width: 200, height: 280)
                }
            }
            .padding(.horizontal, (UIScreen.main.bounds.width - 200) / 2)
        }
    }
}
```

### Safe Area ด้วย GeometryProxy

```swift
// เข้าถึง safe area insets
struct SafeAreaDemo: View {
    var body: some View {
        GeometryReader { geometry in
            VStack {
                Text("Top inset: \(geometry.safeAreaInsets.top)")
                Text("Bottom inset: \(geometry.safeAreaInsets.bottom)")
                Text("Leading inset: \(geometry.safeAreaInsets.leading)")
                Text("Trailing inset: \(geometry.safeAreaInsets.trailing)")
                
                Spacer()
                
                // Custom bottom bar ที่คำนึงถึง safe area
                HStack {
                    ForEach(["house", "search", "bell", "person"], id: \.self) { icon in
                        Image(systemName: icon)
                            .frame(maxWidth: .infinity)
                    }
                }
                .padding(.bottom, geometry.safeAreaInsets.bottom)
                .padding(.top, 16)
                .background(Color(.systemBackground))
            }
        }
        .ignoresSafeArea()
    }
}
```

---

## 12. SwiftUI + UIKit Interop

### UIViewRepresentable

`UIViewRepresentable` ช่วยให้เรานำ UIKit views มาใช้ใน SwiftUI ได้ สิ่งสำคัญคือ `makeUIView`, `updateUIView` และ `Coordinator`

```swift
import SwiftUI
import UIKit

// UITextField ใน SwiftUI ที่มี custom features
struct CustomTextField: UIViewRepresentable {
    @Binding var text: String
    var placeholder: String
    var keyboardType: UIKeyboardType = .default
    var returnKeyType: UIReturnKeyType = .default
    var onReturn: (() -> Void)?
    
    func makeUIView(context: Context) -> UITextField {
        let textField = UITextField()
        textField.placeholder = placeholder
        textField.keyboardType = keyboardType
        textField.returnKeyType = returnKeyType
        textField.delegate = context.coordinator
        textField.setContentHuggingPriority(.defaultHigh, for: .vertical)
        return textField
    }
    
    func updateUIView(_ uiView: UITextField, context: Context) {
        if uiView.text != text {
            uiView.text = text
        }
        uiView.placeholder = placeholder
    }
    
    func makeCoordinator() -> Coordinator {
        Coordinator(text: $text, onReturn: onReturn)
    }
    
    class Coordinator: NSObject, UITextFieldDelegate {
        @Binding var text: String
        var onReturn: (() -> Void)?
        
        init(text: Binding<String>, onReturn: (() -> Void)?) {
            self._text = text
            self.onReturn = onReturn
        }
        
        func textFieldDidChangeSelection(_ textField: UITextField) {
            text = textField.text ?? ""
        }
        
        func textFieldShouldReturn(_ textField: UITextField) -> Bool {
            onReturn?()
            textField.resignFirstResponder()
            return true
        }
    }
}

// MapView wrapper
import MapKit

struct MapView: UIViewRepresentable {
    @Binding var region: MKCoordinateRegion
    var annotations: [MKPointAnnotation]
    var onTap: ((CLLocationCoordinate2D) -> Void)?
    
    func makeUIView(context: Context) -> MKMapView {
        let mapView = MKMapView()
        mapView.delegate = context.coordinator
        
        let tapGesture = UITapGestureRecognizer(
            target: context.coordinator,
            action: #selector(Coordinator.handleTap)
        )
        mapView.addGestureRecognizer(tapGesture)
        
        return mapView
    }
    
    func updateUIView(_ mapView: MKMapView, context: Context) {
        mapView.setRegion(region, animated: true)
        
        mapView.removeAnnotations(mapView.annotations)
        mapView.addAnnotations(annotations)
    }
    
    func makeCoordinator() -> Coordinator {
        Coordinator(parent: self)
    }
    
    class Coordinator: NSObject, MKMapViewDelegate {
        var parent: MapView
        
        init(parent: MapView) {
            self.parent = parent
        }
        
        @objc func handleTap(_ gesture: UITapGestureRecognizer) {
            guard let mapView = gesture.view as? MKMapView else { return }
            let point = gesture.location(in: mapView)
            let coordinate = mapView.convert(point, toCoordinateFrom: mapView)
            parent.onTap?(coordinate)
        }
        
        func mapViewDidChangeVisibleRegion(_ mapView: MKMapView) {
            parent.region = mapView.region
        }
    }
}
```

### UIViewControllerRepresentable

```swift
import SwiftUI
import UIKit
import PhotosUI

// PHPickerViewController ใน SwiftUI
struct PhotoPicker: UIViewControllerRepresentable {
    @Binding var selectedImage: UIImage?
    @Environment(\.dismiss) private var dismiss
    
    func makeUIViewController(context: Context) -> PHPickerViewController {
        var config = PHPickerConfiguration()
        config.selectionLimit = 1
        config.filter = .images
        
        let picker = PHPickerViewController(configuration: config)
        picker.delegate = context.coordinator
        return picker
    }
    
    func updateUIViewController(
        _ uiViewController: PHPickerViewController,
        context: Context
    ) {}
    
    func makeCoordinator() -> Coordinator {
        Coordinator(parent: self)
    }
    
    class Coordinator: NSObject, PHPickerViewControllerDelegate {
        var parent: PhotoPicker
        
        init(parent: PhotoPicker) {
            self.parent = parent
        }
        
        func picker(
            _ picker: PHPickerViewController,
            didFinishPicking results: [PHPickerResult]
        ) {
            parent.dismiss()
            
            guard let provider = results.first?.itemProvider,
                  provider.canLoadObject(ofClass: UIImage.self) else { return }
            
            provider.loadObject(ofClass: UIImage.self) { image, error in
                DispatchQueue.main.async {
                    self.parent.selectedImage = image as? UIImage
                }
            }
        }
    }
}

// DocumentPicker
import UniformTypeIdentifiers

struct DocumentPicker: UIViewControllerRepresentable {
    var contentTypes: [UTType]
    var onSelect: (URL) -> Void
    
    func makeUIViewController(context: Context) -> UIDocumentPickerViewController {
        let picker = UIDocumentPickerViewController(
            forOpeningContentTypes: contentTypes
        )
        picker.delegate = context.coordinator
        return picker
    }
    
    func updateUIViewController(
        _ uiViewController: UIDocumentPickerViewController,
        context: Context
    ) {}
    
    func makeCoordinator() -> Coordinator {
        Coordinator(onSelect: onSelect)
    }
    
    class Coordinator: NSObject, UIDocumentPickerDelegate {
        var onSelect: (URL) -> Void
        
        init(onSelect: @escaping (URL) -> Void) {
            self.onSelect = onSelect
        }
        
        func documentPicker(
            _ controller: UIDocumentPickerViewController,
            didPickDocumentsAt urls: [URL]
        ) {
            guard let url = urls.first else { return }
            onSelect(url)
        }
    }
}
```

---

## 13. SwiftUI + AppKit Interop

### NSViewRepresentable

```swift
import SwiftUI
import AppKit

// NSTextView ใน SwiftUI (macOS)
struct RichTextEditor: NSViewRepresentable {
    @Binding var text: NSAttributedString
    
    func makeNSView(context: Context) -> NSScrollView {
        let scrollView = NSTextView.scrollableTextView()
        let textView = scrollView.documentView as! NSTextView
        
        textView.delegate = context.coordinator
        textView.isRichText = true
        textView.allowsUndo = true
        textView.isEditable = true
        textView.isSelectable = true
        textView.textContainerInset = NSSize(width: 8, height: 8)
        
        return scrollView
    }
    
    func updateNSView(_ scrollView: NSScrollView, context: Context) {
        let textView = scrollView.documentView as! NSTextView
        
        if textView.attributedString() != text {
            textView.textStorage?.setAttributedString(text)
        }
    }
    
    func makeCoordinator() -> Coordinator {
        Coordinator(text: $text)
    }
    
    class Coordinator: NSObject, NSTextViewDelegate {
        @Binding var text: NSAttributedString
        
        init(text: Binding<NSAttributedString>) {
            self._text = text
        }
        
        func textDidChange(_ notification: Notification) {
            guard let textView = notification.object as? NSTextView else { return }
            text = textView.attributedString()
        }
    }
}

// NSColorWell ใน SwiftUI
struct ColorWell: NSViewRepresentable {
    @Binding var color: Color
    
    func makeNSView(context: Context) -> NSColorWell {
        let colorWell = NSColorWell()
        colorWell.target = context.coordinator
        colorWell.action = #selector(Coordinator.colorChanged)
        return colorWell
    }
    
    func updateNSView(_ colorWell: NSColorWell, context: Context) {
        colorWell.color = NSColor(color)
    }
    
    func makeCoordinator() -> Coordinator {
        Coordinator(color: $color)
    }
    
    class Coordinator: NSObject {
        @Binding var color: Color
        
        init(color: Binding<Color>) {
            self._color = color
        }
        
        @objc func colorChanged(_ colorWell: NSColorWell) {
            color = Color(colorWell.color)
        }
    }
}
```

---

## 14. Focus Management กับ @FocusState

### @FocusState เบื้องต้น

```swift
import SwiftUI

// Form ที่ manage focus ด้วย @FocusState
enum LoginField: Hashable {
    case username
    case password
}

struct LoginForm: View {
    @State private var username = ""
    @State private var password = ""
    @FocusState private var focusedField: LoginField?
    
    var body: some View {
        VStack(spacing: 20) {
            TextField("Username", text: $username)
                .focused($focusedField, equals: .username)
                .textInputAutocapitalization(.never)
                .submitLabel(.next)
                .onSubmit {
                    focusedField = .password
                }
            
            SecureField("Password", text: $password)
                .focused($focusedField, equals: .password)
                .submitLabel(.done)
                .onSubmit {
                    focusedField = nil
                    login()
                }
            
            Button("Login") {
                login()
            }
            .buttonStyle(.borderedProminent)
            
            HStack {
                Button("Focus Username") {
                    focusedField = .username
                }
                Button("Focus Password") {
                    focusedField = .password
                }
                Button("Dismiss") {
                    focusedField = nil
                }
            }
            .buttonStyle(.bordered)
        }
        .padding()
        .onAppear {
            DispatchQueue.main.asyncAfter(deadline: .now() + 0.5) {
                focusedField = .username
            }
        }
    }
    
    func login() {
        print("Login: \(username)")
    }
}
```

### Focus State กับ List

```swift
// Editable list ที่ manage focus
struct EditableList: View {
    @State private var items = ["Swift", "Python", "JavaScript"]
    @FocusState private var focusedItem: Int?
    
    var body: some View {
        List {
            ForEach(Array(items.enumerated()), id: \.offset) { index, item in
                TextField("Item", text: $items[index])
                    .focused($focusedItem, equals: index)
                    .submitLabel(index < items.count - 1 ? .next : .done)
                    .onSubmit {
                        if index < items.count - 1 {
                            focusedItem = index + 1
                        } else {
                            focusedItem = nil
                        }
                    }
            }
            .onDelete { indexSet in
                items.remove(atOffsets: indexSet)
            }
        }
        .toolbar {
            ToolbarItem(placement: .navigationBarTrailing) {
                Button("Add") {
                    items.append("")
                    DispatchQueue.main.asyncAfter(deadline: .now() + 0.1) {
                        focusedItem = items.count - 1
                    }
                }
            }
        }
    }
}
```

---

## 15. FocusedValue และ FocusedBinding

### FocusedValue

`FocusedValue` และ `FocusedBinding` ช่วยให้ toolbar และ menu items เข้าถึงค่าจาก focused window ได้

```swift
import SwiftUI

// FocusedValue key
struct DocumentSelectionKey: FocusedValueKey {
    typealias Value = Binding<String>
}

extension FocusedValues {
    var selectedText: Binding<String>? {
        get { self[DocumentSelectionKey.self] }
        set { self[DocumentSelectionKey.self] = newValue }
    }
}

// Document view ที่ expose focused value
struct DocumentView: View {
    @State private var selectedText = ""
    
    var body: some View {
        TextField("Type here...", text: $selectedText)
            .textFieldStyle(.roundedBorder)
            .focusedValue(\.selectedText, $selectedText)
    }
}

// Toolbar ที่ใช้ FocusedValue
struct MainView: View {
    @FocusedBinding(\.selectedText) private var selectedText: String?
    
    var body: some View {
        DocumentView()
            .toolbar {
                ToolbarItem {
                    Button("Uppercase") {
                        selectedText = selectedText?.uppercased()
                    }
                    .disabled(selectedText == nil || selectedText?.isEmpty == true)
                }
                
                ToolbarItem {
                    Button("Clear") {
                        selectedText = ""
                    }
                    .disabled(selectedText == nil || selectedText?.isEmpty == true)
                }
            }
    }
}
```

---

## 16. SwiftUI Rendering Performance

### _printChanges() สำหรับ Debug

```swift
import SwiftUI

// View ที่ใช้ _printChanges เพื่อ debug
struct DebugView: View {
    @State private var count = 0
    @State private var name = "Hello"
    
    var body: some View {
        // Uncomment เพื่อ debug: พิมพ์ว่า property ใดที่เปลี่ยน
        // let _ = Self._printChanges()
        
        VStack {
            Text("\(count)")
            Text(name)
            Button("Increment") { count += 1 }
            Button("Change Name") { name = "World" }
        }
    }
}

// แยก view เพื่อลด rerender
struct CounterDisplay: View {
    let count: Int
    
    var body: some View {
        // View นี้จะ rerender เฉพาะเมื่อ count เปลี่ยน
        Text("\(count)")
            .font(.largeTitle)
    }
}

struct NameDisplay: View {
    let name: String
    
    var body: some View {
        // View นี้จะ rerender เฉพาะเมื่อ name เปลี่ยน
        Text(name)
            .font(.headline)
    }
}

struct OptimizedDebugView: View {
    @State private var count = 0
    @State private var name = "Hello"
    
    var body: some View {
        VStack {
            CounterDisplay(count: count)
            NameDisplay(name: name)
            Button("Increment") { count += 1 }
            Button("Change Name") { name = "World" }
        }
    }
}
```

### EquatableView และ EquatableView

```swift
// ใช้ .equatable() เพื่อป้องกัน unnecessary rerenders
struct ExpensiveView: View, Equatable {
    let value: Int
    
    static func == (lhs: ExpensiveView, rhs: ExpensiveView) -> Bool {
        lhs.value == rhs.value
    }
    
    var body: some View {
        // expensive rendering
        VStack {
            ForEach(0..<value, id: \.self) { i in
                Text("Item \(i)")
            }
        }
    }
}

struct ParentView: View {
    @State private var value = 5
    @State private var unrelatedState = false
    
    var body: some View {
        VStack {
            // ExpensiveView จะไม่ rerender เมื่อ unrelatedState เปลี่ยน
            ExpensiveView(value: value)
                .equatable()
            
            Toggle("Unrelated", isOn: $unrelatedState)
            
            Button("Change Value") {
                value = Int.random(in: 1...10)
            }
        }
    }
}
```

---

## 17. View Identity และ State Preservation

### Structural Identity

```swift
import SwiftUI

// Structural identity - SwiftUI track ด้วย position ใน view tree
struct StructuralIdentity: View {
    @State private var showCircle = true
    
    var body: some View {
        VStack {
            // ปัญหา: state สลับเพราะ structural identity เปลี่ยน
            if showCircle {
                Circle()
                    .fill(Color.blue)
                    .frame(width: 50, height: 50)
            } else {
                Rectangle()
                    .fill(Color.red)
                    .frame(width: 50, height: 50)
            }
            
            // แก้ด้วย explicit identity
            Button("Toggle") {
                showCircle.toggle()
            }
        }
    }
}

// Explicit Identity ด้วย .id()
struct ExplicitIdentity: View {
    @State private var id = UUID()
    @State private var count = 0
    
    var body: some View {
        VStack {
            // View นี้จะ reset state เมื่อ id เปลี่ยน
            CounterView()
                .id(id)
            
            Button("Reset Counter") {
                id = UUID()
            }
        }
    }
}

struct CounterView: View {
    @State private var count = 0
    
    var body: some View {
        VStack {
            Text("\(count)")
            Button("Add") { count += 1 }
        }
    }
}
```

---

## 18. @SceneStorage สำหรับ State Preservation

### @SceneStorage เบื้องต้น

`@SceneStorage` บันทึก state ของ scene อัตโนมัติ เมื่อ app ถูก restore state จะถูก restore ด้วย

```swift
import SwiftUI

struct NotepadView: View {
    @SceneStorage("notepad.text") private var text = ""
    @SceneStorage("notepad.selectedTab") private var selectedTab = 0
    @SceneStorage("notepad.fontSize") private var fontSize = 16.0
    
    var body: some View {
        VStack {
            Picker("Tab", selection: $selectedTab) {
                Text("Edit").tag(0)
                Text("Preview").tag(1)
            }
            .pickerStyle(.segmented)
            
            if selectedTab == 0 {
                TextEditor(text: $text)
                    .font(.system(size: fontSize))
            } else {
                ScrollView {
                    Text(text)
                        .font(.system(size: fontSize))
                        .frame(maxWidth: .infinity, alignment: .leading)
                        .padding()
                }
            }
            
            HStack {
                Text("Font Size:")
                Slider(value: $fontSize, in: 10...32)
                Text("\(Int(fontSize))pt")
            }
            .padding()
        }
    }
}
```

### @SceneStorage กับ Complex Types

```swift
// SceneStorage กับ Codable type
struct AppState: Codable {
    var selectedCategory: String = "All"
    var sortOrder: String = "Name"
    var showFavorites: Bool = false
}

struct ContentListView: View {
    // SceneStorage รองรับแค่ primitive types โดยตรง
    // ต้องใช้ manual encoding สำหรับ complex types
    @SceneStorage("contentList.selectedCategory") private var selectedCategory = "All"
    @SceneStorage("contentList.sortOrder") private var sortOrder = "Name"
    @SceneStorage("contentList.showFavorites") private var showFavorites = false
    
    let categories = ["All", "Swift", "UIKit", "SwiftUI", "Combine"]
    
    var body: some View {
        NavigationView {
            List {
                ForEach(filteredItems, id: \.self) { item in
                    Text(item)
                }
            }
            .toolbar {
                ToolbarItem(placement: .navigationBarLeading) {
                    Picker("Category", selection: $selectedCategory) {
                        ForEach(categories, id: \.self) { category in
                            Text(category).tag(category)
                        }
                    }
                }
                
                ToolbarItem(placement: .navigationBarTrailing) {
                    Toggle(isOn: $showFavorites) {
                        Image(systemName: "star")
                    }
                }
            }
        }
    }
    
    var filteredItems: [String] {
        // Filter logic
        ["Swift Basics", "UIKit Guide", "SwiftUI Tutorial"]
    }
}
```

---

## 19. Environment Key Advanced Patterns

### Custom EnvironmentKey

```swift
import SwiftUI

// Custom Environment value
struct ThemeKey: EnvironmentKey {
    static let defaultValue: AppTheme = .light
}

struct AppTheme {
    var primaryColor: Color
    var secondaryColor: Color
    var backgroundColor: Color
    var font: Font
    
    static let light = AppTheme(
        primaryColor: .blue,
        secondaryColor: .gray,
        backgroundColor: .white,
        font: .body
    )
    
    static let dark = AppTheme(
        primaryColor: .cyan,
        secondaryColor: .gray,
        backgroundColor: Color(white: 0.1),
        font: .body
    )
    
    static let high = AppTheme(
        primaryColor: .yellow,
        secondaryColor: .white,
        backgroundColor: .black,
        font: .body.bold()
    )
}

extension EnvironmentValues {
    var appTheme: AppTheme {
        get { self[ThemeKey.self] }
        set { self[ThemeKey.self] = newValue }
    }
}

// Modifier สำหรับ apply theme
extension View {
    func appTheme(_ theme: AppTheme) -> some View {
        environment(\.appTheme, theme)
    }
}

// ใช้งาน
struct ThemedButton: View {
    @Environment(\.appTheme) private var theme
    
    let title: String
    let action: () -> Void
    
    var body: some View {
        Button(action: action) {
            Text(title)
                .font(theme.font)
                .foregroundColor(theme.backgroundColor)
                .padding()
                .background(theme.primaryColor)
                .cornerRadius(8)
        }
    }
}

struct ThemedApp: View {
    @State private var theme: AppTheme = .light
    
    var body: some View {
        VStack(spacing: 20) {
            ThemedButton(title: "Click me") {
                print("Clicked")
            }
            
            HStack {
                Button("Light") { theme = .light }
                Button("Dark") { theme = .dark }
                Button("High Contrast") { theme = .high }
            }
        }
        .appTheme(theme)
    }
}
```

### Environment กับ Observable

```swift
// @Observable + Environment (iOS 17+)
@Observable
class AppSettings {
    var theme: String = "light"
    var language: String = "en"
    var fontSize: Double = 16
}

struct SettingsKey: EnvironmentKey {
    static let defaultValue = AppSettings()
}

extension EnvironmentValues {
    var settings: AppSettings {
        get { self[SettingsKey.self] }
        set { self[SettingsKey.self] = newValue }
    }
}

struct SettingsConsumer: View {
    @Environment(\.settings) private var settings
    
    var body: some View {
        VStack {
            Text("Theme: \(settings.theme)")
            Text("Language: \(settings.language)")
            Text("Font Size: \(Int(settings.fontSize))")
        }
    }
}
```

---

## 20. Custom ButtonStyle, LabelStyle, TextFieldStyle

### Custom ButtonStyle

```swift
import SwiftUI

// Glassmorphism button style
struct GlassButtonStyle: ButtonStyle {
    var tintColor: Color = .blue
    
    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .padding(.horizontal, 24)
            .padding(.vertical, 12)
            .background(
                ZStack {
                    // Background blur
                    RoundedRectangle(cornerRadius: 12)
                        .fill(.ultraThinMaterial)
                    
                    // Tint overlay
                    RoundedRectangle(cornerRadius: 12)
                        .fill(tintColor.opacity(0.2))
                    
                    // Inner highlight
                    RoundedRectangle(cornerRadius: 12)
                        .stroke(Color.white.opacity(0.3), lineWidth: 1)
                }
            )
            .foregroundColor(tintColor)
            .scaleEffect(configuration.isPressed ? 0.95 : 1)
            .opacity(configuration.isPressed ? 0.8 : 1)
            .animation(.spring(response: 0.3), value: configuration.isPressed)
    }
}

// Outline button style
struct OutlineButtonStyle: ButtonStyle {
    var color: Color = .blue
    var lineWidth: CGFloat = 2
    var cornerRadius: CGFloat = 10
    
    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .padding(.horizontal, 20)
            .padding(.vertical, 10)
            .background(
                configuration.isPressed
                    ? color.opacity(0.1)
                    : Color.clear
            )
            .overlay(
                RoundedRectangle(cornerRadius: cornerRadius)
                    .stroke(color, lineWidth: lineWidth)
            )
            .foregroundColor(color)
            .cornerRadius(cornerRadius)
            .animation(.easeInOut(duration: 0.15), value: configuration.isPressed)
    }
}

// Gradient button style
struct GradientButtonStyle: ButtonStyle {
    var colors: [Color] = [.blue, .purple]
    
    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .padding(.horizontal, 24)
            .padding(.vertical, 14)
            .background(
                LinearGradient(
                    colors: configuration.isPressed
                        ? colors.map { $0.opacity(0.7) }
                        : colors,
                    startPoint: .leading,
                    endPoint: .trailing
                )
            )
            .foregroundColor(.white)
            .cornerRadius(12)
            .shadow(
                color: colors.first?.opacity(0.4) ?? .clear,
                radius: configuration.isPressed ? 2 : 8,
                y: configuration.isPressed ? 2 : 4
            )
            .scaleEffect(configuration.isPressed ? 0.97 : 1)
            .animation(.spring(response: 0.3), value: configuration.isPressed)
    }
}

// การใช้งาน
struct ButtonStyleDemo: View {
    var body: some View {
        VStack(spacing: 20) {
            Button("Glass Button") {}
                .buttonStyle(GlassButtonStyle())
            
            Button("Outline Button") {}
                .buttonStyle(OutlineButtonStyle(color: .purple))
            
            Button("Gradient Button") {}
                .buttonStyle(GradientButtonStyle(colors: [.orange, .pink]))
        }
        .padding()
        .background(Color.blue.opacity(0.1))
    }
}
```

### Custom LabelStyle

```swift
// Custom label styles
struct IconAboveLabelStyle: LabelStyle {
    func makeBody(configuration: Configuration) -> some View {
        VStack(spacing: 8) {
            configuration.icon
                .font(.title2)
            configuration.title
                .font(.caption)
        }
    }
}

struct BadgeLabelStyle: LabelStyle {
    var badgeCount: Int
    
    func makeBody(configuration: Configuration) -> some View {
        ZStack(alignment: .topTrailing) {
            HStack {
                configuration.icon
                configuration.title
            }
            
            if badgeCount > 0 {
                Text("\(badgeCount)")
                    .font(.caption2)
                    .foregroundColor(.white)
                    .padding(4)
                    .background(Color.red)
                    .clipShape(Circle())
                    .offset(x: 8, y: -8)
            }
        }
    }
}

struct LabelStyleDemo: View {
    var body: some View {
        HStack(spacing: 30) {
            Label("Home", systemImage: "house")
                .labelStyle(IconAboveLabelStyle())
            
            Label("Messages", systemImage: "message")
                .labelStyle(BadgeLabelStyle(badgeCount: 5))
            
            Label("Profile", systemImage: "person")
                .labelStyle(IconAboveLabelStyle())
        }
    }
}
```

### Custom TextFieldStyle

```swift
// Floating label TextField style
struct FloatingLabelTextFieldStyle: TextFieldStyle {
    var label: String
    var isFocused: Bool
    
    func _body(configuration: TextField<_Label>) -> some View {
        ZStack(alignment: .leading) {
            // Label
            Text(label)
                .font(isFocused ? .caption : .body)
                .foregroundColor(isFocused ? .blue : .gray)
                .offset(y: isFocused ? -25 : 0)
                .animation(.spring(response: 0.3), value: isFocused)
            
            // TextField
            configuration
                .padding(.top, isFocused ? 12 : 0)
        }
        .padding(.horizontal, 12)
        .padding(.vertical, 8)
        .overlay(
            Rectangle()
                .frame(height: 1)
                .foregroundColor(isFocused ? .blue : .gray.opacity(0.5)),
            alignment: .bottom
        )
    }
}

struct FloatingLabelDemo: View {
    @State private var email = ""
    @State private var password = ""
    @FocusState private var focusedField: String?
    
    var body: some View {
        VStack(spacing: 24) {
            TextField("", text: $email)
                .textFieldStyle(
                    FloatingLabelTextFieldStyle(
                        label: "Email",
                        isFocused: focusedField == "email" || !email.isEmpty
                    )
                )
                .focused($focusedField, equals: "email")
            
            SecureField("", text: $password)
                .textFieldStyle(
                    FloatingLabelTextFieldStyle(
                        label: "Password",
                        isFocused: focusedField == "password" || !password.isEmpty
                    )
                )
                .focused($focusedField, equals: "password")
        }
        .padding()
    }
}
```

---

## 21. แบบฝึกหัดพร้อมเฉลย

### แบบฝึกหัดที่ 1: Custom Rating View

สร้าง rating view ที่ใช้ Layout protocol และ custom shape

```swift
import SwiftUI

// เฉลย: RatingView
struct RatingView: View {
    @Binding var rating: Int
    var maxRating: Int = 5
    var starSize: CGFloat = 30
    var spacing: CGFloat = 4
    var color: Color = .yellow
    
    var body: some View {
        HStack(spacing: spacing) {
            ForEach(1...maxRating, id: \.self) { star in
                Star(points: 5)
                    .fill(star <= rating ? color : Color.gray.opacity(0.3))
                    .frame(width: starSize, height: starSize)
                    .contentShape(Rectangle())
                    .onTapGesture {
                        withAnimation(.spring()) {
                            rating = star
                        }
                    }
                    .scaleEffect(star <= rating ? 1.1 : 1.0)
                    .animation(.spring(response: 0.3), value: rating)
            }
        }
    }
}

// ทดสอบ
struct RatingDemo: View {
    @State private var rating = 3
    
    var body: some View {
        VStack(spacing: 20) {
            RatingView(rating: $rating)
            RatingView(rating: $rating, color: .blue, maxRating: 10)
            Text("Rating: \(rating)")
        }
    }
}
```

### แบบฝึกหัดที่ 2: Animated Progress Ring

```swift
// เฉลย: AnimatedProgressRing
struct ProgressRing: Shape {
    var progress: Double // 0.0 - 1.0
    
    var animatableData: Double {
        get { progress }
        set { progress = newValue }
    }
    
    func path(in rect: CGRect) -> Path {
        var path = Path()
        let center = CGPoint(x: rect.midX, y: rect.midY)
        let radius = min(rect.width, rect.height) / 2
        
        path.addArc(
            center: center,
            radius: radius,
            startAngle: .degrees(-90),
            endAngle: .degrees(-90 + 360 * progress),
            clockwise: false
        )
        
        return path
    }
}

struct AnimatedProgressRingView: View {
    let progress: Double
    let color: Color
    let lineWidth: CGFloat
    
    var body: some View {
        ZStack {
            Circle()
                .stroke(color.opacity(0.2), lineWidth: lineWidth)
            
            ProgressRing(progress: progress)
                .stroke(
                    color,
                    style: StrokeStyle(
                        lineWidth: lineWidth,
                        lineCap: .round
                    )
                )
                .animation(.spring(response: 0.8), value: progress)
            
            Text("\(Int(progress * 100))%")
                .font(.headline.bold())
        }
    }
}

struct ProgressRingDemo: View {
    @State private var progress = 0.0
    
    var body: some View {
        VStack(spacing: 30) {
            HStack(spacing: 20) {
                AnimatedProgressRingView(
                    progress: progress,
                    color: .blue,
                    lineWidth: 10
                )
                .frame(width: 100, height: 100)
                
                AnimatedProgressRingView(
                    progress: min(1.0, progress * 1.5),
                    color: .green,
                    lineWidth: 8
                )
                .frame(width: 80, height: 80)
                
                AnimatedProgressRingView(
                    progress: min(1.0, progress * 2),
                    color: .orange,
                    lineWidth: 6
                )
                .frame(width: 60, height: 60)
            }
            
            Slider(value: $progress)
                .padding()
            
            Button("Animate") {
                withAnimation(.easeInOut(duration: 2)) {
                    progress = progress < 0.5 ? 1.0 : 0.0
                }
            }
        }
        .padding()
    }
}
```

### แบบฝึกหัดที่ 3: Custom Tab Bar

```swift
// เฉลย: CustomTabBar
struct TabBarItem {
    let icon: String
    let selectedIcon: String
    let title: String
}

struct CustomTabBar: View {
    @Binding var selectedIndex: Int
    let items: [TabBarItem]
    
    var body: some View {
        HStack(spacing: 0) {
            ForEach(Array(items.enumerated()), id: \.offset) { index, item in
                TabBarButton(
                    item: item,
                    isSelected: selectedIndex == index
                ) {
                    withAnimation(.spring(response: 0.3)) {
                        selectedIndex = index
                    }
                }
                .frame(maxWidth: .infinity)
            }
        }
        .padding(.horizontal)
        .padding(.bottom, 8)
        .background(
            Color(.systemBackground)
                .shadow(color: .black.opacity(0.1), radius: 8, y: -4)
        )
    }
}

struct TabBarButton: View {
    let item: TabBarItem
    let isSelected: Bool
    let action: () -> Void
    
    var body: some View {
        Button(action: action) {
            VStack(spacing: 4) {
                Image(systemName: isSelected ? item.selectedIcon : item.icon)
                    .font(.title3)
                    .symbolEffect(.bounce, value: isSelected)
                
                Text(item.title)
                    .font(.caption2)
            }
            .foregroundColor(isSelected ? .blue : .gray)
            .padding(.vertical, 8)
        }
    }
}

struct CustomTabBarDemo: View {
    @State private var selectedTab = 0
    
    let tabs = [
        TabBarItem(icon: "house", selectedIcon: "house.fill", title: "Home"),
        TabBarItem(icon: "magnifyingglass", selectedIcon: "magnifyingglass", title: "Search"),
        TabBarItem(icon: "plus.app", selectedIcon: "plus.app.fill", title: "Create"),
        TabBarItem(icon: "bell", selectedIcon: "bell.fill", title: "Activity"),
        TabBarItem(icon: "person", selectedIcon: "person.fill", title: "Profile")
    ]
    
    var body: some View {
        VStack(spacing: 0) {
            // Content
            TabView(selection: $selectedTab) {
                ForEach(Array(tabs.enumerated()), id: \.offset) { index, tab in
                    Text(tab.title)
                        .font(.largeTitle)
                        .tag(index)
                }
            }
            .tabViewStyle(.page(indexDisplayMode: .never))
            
            CustomTabBar(selectedIndex: $selectedTab, items: tabs)
        }
        .ignoresSafeArea(edges: .bottom)
    }
}
```

---

## 22. การสร้าง Custom Chart Library

### Foundation ของ Chart Library

```swift
import SwiftUI

// Data model สำหรับ chart
struct ChartData {
    var points: [DataPoint]
    var title: String
    
    struct DataPoint: Identifiable {
        let id = UUID()
        var x: Double
        var y: Double
        var label: String?
        var color: Color?
    }
}

// Chart protocol
protocol ChartView: View {
    var data: ChartData { get }
}

// Line Chart
struct LineChart: View {
    let data: ChartData
    var lineColor: Color = .blue
    var showPoints: Bool = true
    var fillGradient: Bool = true
    var lineWidth: CGFloat = 2
    
    private var xRange: ClosedRange<Double> {
        let xs = data.points.map(\.x)
        return (xs.min() ?? 0)...(xs.max() ?? 1)
    }
    
    private var yRange: ClosedRange<Double> {
        let ys = data.points.map(\.y)
        return (ys.min() ?? 0)...(ys.max() ?? 1)
    }
    
    var body: some View {
        GeometryReader { geometry in
            let width = geometry.size.width
            let height = geometry.size.height
            
            ZStack {
                // Grid lines
                ChartGrid()
                    .stroke(Color.gray.opacity(0.2), lineWidth: 0.5)
                
                // Fill area
                if fillGradient {
                    chartFillPath(in: geometry)
                        .fill(
                            LinearGradient(
                                colors: [lineColor.opacity(0.3), .clear],
                                startPoint: .top,
                                endPoint: .bottom
                            )
                        )
                }
                
                // Line
                chartLinePath(in: geometry)
                    .stroke(lineColor, style: StrokeStyle(
                        lineWidth: lineWidth,
                        lineCap: .round,
                        lineJoin: .round
                    ))
                
                // Points
                if showPoints {
                    ForEach(data.points) { point in
                        Circle()
                            .fill(point.color ?? lineColor)
                            .frame(width: 8, height: 8)
                            .position(
                                x: xPosition(for: point.x, width: width),
                                y: yPosition(for: point.y, height: height)
                            )
                    }
                }
            }
        }
    }
    
    private func xPosition(for x: Double, width: CGFloat) -> CGFloat {
        let range = xRange.upperBound - xRange.lowerBound
        guard range > 0 else { return 0 }
        return CGFloat((x - xRange.lowerBound) / range) * width
    }
    
    private func yPosition(for y: Double, height: CGFloat) -> CGFloat {
        let range = yRange.upperBound - yRange.lowerBound
        guard range > 0 else { return height }
        return height - CGFloat((y - yRange.lowerBound) / range) * height
    }
    
    private func chartLinePath(in geometry: GeometryProxy) -> Path {
        var path = Path()
        let width = geometry.size.width
        let height = geometry.size.height
        
        for (index, point) in data.points.enumerated() {
            let x = xPosition(for: point.x, width: width)
            let y = yPosition(for: point.y, height: height)
            
            if index == 0 {
                path.move(to: CGPoint(x: x, y: y))
            } else {
                // Smooth curve
                let prev = data.points[index - 1]
                let prevX = xPosition(for: prev.x, width: width)
                let prevY = yPosition(for: prev.y, height: height)
                let midX = (prevX + x) / 2
                
                path.addCurve(
                    to: CGPoint(x: x, y: y),
                    control1: CGPoint(x: midX, y: prevY),
                    control2: CGPoint(x: midX, y: y)
                )
            }
        }
        
        return path
    }
    
    private func chartFillPath(in geometry: GeometryProxy) -> Path {
        var path = chartLinePath(in: geometry)
        let width = geometry.size.width
        let height = geometry.size.height
        
        if let lastPoint = data.points.last, let firstPoint = data.points.first {
            let lastX = xPosition(for: lastPoint.x, width: width)
            let firstX = xPosition(for: firstPoint.x, width: width)
            
            path.addLine(to: CGPoint(x: lastX, y: height))
            path.addLine(to: CGPoint(x: firstX, y: height))
            path.closeSubpath()
        }
        
        return path
    }
}

// Bar Chart
struct BarChart: View {
    let data: ChartData
    var barColor: Color = .blue
    var spacing: CGFloat = 4
    var cornerRadius: CGFloat = 4
    var showValues: Bool = true
    var animate: Bool = true
    
    @State private var animationProgress: Double = 0
    
    private var maxY: Double {
        data.points.map(\.y).max() ?? 1
    }
    
    var body: some View {
        GeometryReader { geometry in
            HStack(alignment: .bottom, spacing: spacing) {
                ForEach(data.points) { point in
                    BarView(
                        value: point.y,
                        maxValue: maxY,
                        color: point.color ?? barColor,
                        label: point.label ?? "",
                        cornerRadius: cornerRadius,
                        showValue: showValues,
                        animationProgress: animationProgress
                    )
                }
            }
        }
        .onAppear {
            if animate {
                withAnimation(.spring(response: 0.8, dampingFraction: 0.7).delay(0.1)) {
                    animationProgress = 1.0
                }
            } else {
                animationProgress = 1.0
            }
        }
    }
}

struct BarView: View {
    let value: Double
    let maxValue: Double
    let color: Color
    let label: String
    let cornerRadius: CGFloat
    let showValue: Bool
    let animationProgress: Double
    
    var heightRatio: Double {
        maxValue > 0 ? (value / maxValue) * animationProgress : 0
    }
    
    var body: some View {
        GeometryReader { geometry in
            let barHeight = geometry.size.height * heightRatio
            
            VStack(spacing: 4) {
                if showValue {
                    Text(String(format: "%.1f", value))
                        .font(.caption2)
                        .opacity(animationProgress)
                }
                
                Spacer()
                
                RoundedRectangle(cornerRadius: cornerRadius)
                    .fill(color)
                    .frame(height: barHeight)
            }
            .frame(maxWidth: .infinity)
        }
    }
}

// Grid helper
struct ChartGrid: Shape {
    var horizontalLines: Int = 5
    var verticalLines: Int = 5
    
    func path(in rect: CGRect) -> Path {
        var path = Path()
        
        // Horizontal lines
        for i in 0...horizontalLines {
            let y = rect.minY + rect.height * CGFloat(i) / CGFloat(horizontalLines)
            path.move(to: CGPoint(x: rect.minX, y: y))
            path.addLine(to: CGPoint(x: rect.maxX, y: y))
        }
        
        // Vertical lines
        for i in 0...verticalLines {
            let x = rect.minX + rect.width * CGFloat(i) / CGFloat(verticalLines)
            path.move(to: CGPoint(x: x, y: rect.minY))
            path.addLine(to: CGPoint(x: x, y: rect.maxY))
        }
        
        return path
    }
}

// Demo
struct ChartLibraryDemo: View {
    let lineData = ChartData(
        points: [
            .init(x: 0, y: 20, label: "Jan"),
            .init(x: 1, y: 45, label: "Feb"),
            .init(x: 2, y: 30, label: "Mar"),
            .init(x: 3, y: 60, label: "Apr"),
            .init(x: 4, y: 40, label: "May"),
            .init(x: 5, y: 75, label: "Jun"),
            .init(x: 6, y: 55, label: "Jul")
        ],
        title: "Monthly Sales"
    )
    
    let barData = ChartData(
        points: [
            .init(x: 0, y: 35, label: "Mon", color: .blue),
            .init(x: 1, y: 60, label: "Tue", color: .green),
            .init(x: 2, y: 45, label: "Wed", color: .orange),
            .init(x: 3, y: 80, label: "Thu", color: .purple),
            .init(x: 4, y: 55, label: "Fri", color: .red),
            .init(x: 5, y: 30, label: "Sat", color: .teal),
            .init(x: 6, y: 25, label: "Sun", color: .pink)
        ],
        title: "Weekly Activity"
    )
    
    var body: some View {
        ScrollView {
            VStack(spacing: 30) {
                VStack(alignment: .leading) {
                    Text(lineData.title)
                        .font(.headline)
                    LineChart(data: lineData)
                        .frame(height: 200)
                        .padding()
                        .background(Color(.systemGray6))
                        .cornerRadius(16)
                }
                
                VStack(alignment: .leading) {
                    Text(barData.title)
                        .font(.headline)
                    BarChart(data: barData)
                        .frame(height: 200)
                        .padding()
                        .background(Color(.systemGray6))
                        .cornerRadius(16)
                }
            }
            .padding()
        }
    }
}
```

---

## 23. สรุป

ในบทนี้เราได้เรียนรู้เทคนิค SwiftUI ขั้นสูงหลายอย่าง:

### สิ่งที่ได้เรียนรู้

| หัวข้อ | สิ่งสำคัญ |
|--------|-----------|
| Custom ViewModifier | สร้าง reusable modifiers ที่ animate ได้ |
| ViewBuilder | สร้าง custom container views |
| PreferenceKey | ส่งข้อมูลจาก child ไป parent |
| AnchorPreference | ติดตามตำแหน่งใน coordinate space |
| Custom Layout | FlowLayout, MasonryLayout, RadialLayout |
| AnimatableData | ทำให้ Shape animate ได้ |
| Custom Transitions | สร้าง transition แบบกำหนดเอง |
| GeometryReader | Parallax, scale effects, safe area |
| UIKit Interop | UIViewRepresentable, UIViewControllerRepresentable |
| AppKit Interop | NSViewRepresentable |
| @FocusState | จัดการ focus ใน form |
| FocusedValue | Toolbar ที่ responsive ต่อ focused window |
| Rendering Performance | _printChanges, EquatableView |
| View Identity | Structural vs explicit identity |
| @SceneStorage | State preservation |
| Environment Keys | Custom environment values |
| Custom Styles | ButtonStyle, LabelStyle, TextFieldStyle |

### Best Practices

1. **แยก view ขนาดเล็กๆ** เพื่อลด rerendering
2. **ใช้ .equatable()** สำหรับ view ที่มีการ compute ซับซ้อน
3. **ระวัง GeometryReader** เพราะมีผลต่อ performance
4. **ใช้ PreferenceKey** สำหรับ bottom-up communication
5. **ใช้ Environment** สำหรับ top-down configuration
6. **Layout protocol** ดีกว่า GeometryReader สำหรับ custom layouts
7. **@FocusState** ช่วยสร้าง UX ที่ดีสำหรับ forms

### แนะนำอ่านเพิ่มเติม

- [SwiftUI Layout Protocol - WWDC 2022](https://developer.apple.com/videos/play/wwdc2022/10056/)
- [Demystify SwiftUI - WWDC 2021](https://developer.apple.com/videos/play/wwdc2021/10022/)
- [Advanced SwiftUI Animations - WWDC 2023](https://developer.apple.com/videos/play/wwdc2023/10157/)

---

*จบ Part 73: Advanced SwiftUI*
