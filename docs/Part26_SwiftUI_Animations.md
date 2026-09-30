# ส่วนที่ 26: SwiftUI Animations

## บทนำ

Animation เป็นส่วนสำคัญที่ทำให้แอปพลิเคชันรู้สึกมีชีวิตชีวา SwiftUI ทำให้การสร้าง animation เป็นเรื่องง่ายด้วย API ที่ออกแบบมาอย่างดี ในส่วนนี้เราจะเรียนรู้ตั้งแต่ animation พื้นฐานไปจนถึง animation ขั้นสูงเช่น matchedGeometryEffect, Keyframe animations และ Canvas

---

## 1. Animation Basics (พื้นฐาน Animation)

Animation ใน SwiftUI คือการเปลี่ยนแปลง state ที่เกิดขึ้นอย่างราบรื่นแทนที่จะเปลี่ยนทันที SwiftUI ติดตาม state และสร้าง intermediate frames ให้โดยอัตโนมัติ

### หลักการของ SwiftUI Animation

```swift
import SwiftUI

// Animation ทำงานเมื่อ @State เปลี่ยนค่า
struct AnimationBasics: View {
    @State private var isExpanded = false
    
    var body: some View {
        VStack(spacing: 20) {
            // ขนาดเปลี่ยนตาม state
            RoundedRectangle(cornerRadius: 20)
                .fill(Color.blue)
                .frame(
                    width: isExpanded ? 300 : 150,
                    height: isExpanded ? 300 : 150
                )
                .animation(.default, value: isExpanded)
            
            Button(isExpanded ? "ย่อ" : "ขยาย") {
                isExpanded.toggle()
            }
            .buttonStyle(.borderedProminent)
        }
    }
}
```

### Animation Pipeline

SwiftUI Animation ทำงานตามลำดับนี้:
1. ผู้ใช้กระทำ action (กดปุ่ม, swipe, etc.)
2. State เปลี่ยนค่า
3. SwiftUI คำนวณ View ใหม่ (after state)
4. Animation interpolates ระหว่าง before และ after
5. แสดงผล frame ต่อ frame

```swift
struct AnimationPipeline: View {
    @State private var opacity: Double = 1.0
    @State private var scale: CGFloat = 1.0
    @State private var rotation: Double = 0.0
    
    var body: some View {
        VStack(spacing: 30) {
            // View ที่มี multiple animated properties
            Image(systemName: "star.fill")
                .font(.system(size: 60))
                .foregroundColor(.yellow)
                .opacity(opacity)
                .scaleEffect(scale)
                .rotationEffect(.degrees(rotation))
            
            // ปุ่มควบคุม
            HStack(spacing: 16) {
                Button("Fade") {
                    withAnimation(.easeInOut(duration: 0.5)) {
                        opacity = opacity == 1.0 ? 0.2 : 1.0
                    }
                }
                
                Button("Scale") {
                    withAnimation(.spring(response: 0.4, dampingFraction: 0.5)) {
                        scale = scale == 1.0 ? 1.5 : 1.0
                    }
                }
                
                Button("Rotate") {
                    withAnimation(.linear(duration: 0.5)) {
                        rotation += 360
                    }
                }
            }
            .buttonStyle(.bordered)
        }
    }
}
```

---

## 2. withAnimation

`withAnimation` คือ function ที่ wrap การเปลี่ยน state เพื่อให้เกิด animation

### การใช้งานพื้นฐาน

```swift
struct WithAnimationExample: View {
    @State private var showDetail = false
    @State private var offset: CGFloat = 0
    @State private var color = Color.blue
    
    var body: some View {
        VStack(spacing: 20) {
            // ปุ่มที่เปิด animation ด้วย withAnimation
            Button("Toggle Detail") {
                withAnimation {
                    showDetail.toggle()
                }
            }
            
            if showDetail {
                Text("รายละเอียดที่ซ่อนอยู่")
                    .padding()
                    .background(Color.blue.opacity(0.2))
                    .cornerRadius(10)
            }
            
            Button("เลื่อน") {
                withAnimation(.spring()) {
                    offset = offset == 0 ? 100 : 0
                }
            }
            
            Circle()
                .fill(color)
                .frame(width: 60, height: 60)
                .offset(x: offset)
            
            Button("เปลี่ยนสี") {
                withAnimation(.easeInOut(duration: 0.8)) {
                    color = color == .blue ? .red : .blue
                }
            }
        }
        .padding()
    }
}
```

### withAnimation กับ Completion Handler

```swift
struct AnimationWithCompletion: View {
    @State private var isAnimating = false
    @State private var message = ""
    
    var body: some View {
        VStack(spacing: 20) {
            Circle()
                .fill(isAnimating ? Color.green : Color.red)
                .frame(width: 80, height: 80)
                .scaleEffect(isAnimating ? 1.3 : 1.0)
            
            Text(message)
                .font(.headline)
            
            Button("เริ่ม Animation") {
                message = "กำลัง animate..."
                withAnimation(.spring(duration: 0.5)) {
                    isAnimating = true
                } completion: {
                    message = "Animation เสร็จสิ้น!"
                    // Reset หลัง delay
                    DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
                        withAnimation {
                            isAnimating = false
                            message = ""
                        }
                    }
                }
            }
            .disabled(isAnimating)
        }
    }
}
```

---

## 3. Animation Types

SwiftUI มี animation type หลายแบบให้เลือกตามลักษณะการเคลื่อนไหวที่ต้องการ

### Linear Animation

```swift
struct LinearAnimationDemo: View {
    @State private var progress: CGFloat = 0
    @State private var isAnimating = false
    
    var body: some View {
        VStack(spacing: 20) {
            // Progress bar แบบ linear
            GeometryReader { geo in
                ZStack(alignment: .leading) {
                    RoundedRectangle(cornerRadius: 8)
                        .fill(Color.gray.opacity(0.3))
                        .frame(height: 20)
                    
                    RoundedRectangle(cornerRadius: 8)
                        .fill(Color.blue)
                        .frame(width: geo.size.width * progress, height: 20)
                        .animation(.linear(duration: 2.0), value: progress)
                }
            }
            .frame(height: 20)
            .padding(.horizontal)
            
            Button("เริ่ม Linear") {
                progress = progress == 0 ? 1 : 0
            }
            .buttonStyle(.borderedProminent)
        }
    }
}
```

### easeIn, easeOut, easeInOut

```swift
struct EaseAnimationsDemo: View {
    @State private var offsets: [CGFloat] = [0, 0, 0, 0]
    let animationTypes = ["Linear", "easeIn", "easeOut", "easeInOut"]
    
    var body: some View {
        VStack(spacing: 16) {
            Text("เปรียบเทียบ Ease Types")
                .font(.headline)
            
            ForEach(0..<4, id: \.self) { i in
                HStack {
                    Text(animationTypes[i])
                        .frame(width: 100, alignment: .leading)
                        .font(.caption)
                    
                    GeometryReader { geo in
                        Circle()
                            .fill(Color.blue)
                            .frame(width: 20, height: 20)
                            .offset(x: offsets[i] == 0 ? 0 : geo.size.width - 20)
                            .animation(animationForIndex(i), value: offsets[i])
                    }
                    .frame(height: 20)
                }
            }
            
            Button("เล่น") {
                for i in 0..<4 {
                    offsets[i] = offsets[i] == 0 ? 1 : 0
                }
            }
            .buttonStyle(.borderedProminent)
        }
        .padding()
    }
    
    func animationForIndex(_ i: Int) -> Animation {
        switch i {
        case 0: return .linear(duration: 1.5)
        case 1: return .easeIn(duration: 1.5)
        case 2: return .easeOut(duration: 1.5)
        case 3: return .easeInOut(duration: 1.5)
        default: return .default
        }
    }
}
```

### Spring Animation

```swift
struct SpringAnimationDemo: View {
    @State private var isExpanded = false
    
    var body: some View {
        VStack(spacing: 20) {
            Text("Spring Animations")
                .font(.headline)
            
            // Spring ค่าเริ่มต้น
            Group {
                Text("Default Spring")
                    .font(.caption)
                Circle()
                    .fill(Color.blue)
                    .frame(width: isExpanded ? 100 : 50, height: isExpanded ? 100 : 50)
                    .animation(.spring(), value: isExpanded)
            }
            
            // Spring กำหนดค่าเอง
            Group {
                Text("Custom Spring (bouncy)")
                    .font(.caption)
                Circle()
                    .fill(Color.green)
                    .frame(width: isExpanded ? 100 : 50, height: isExpanded ? 100 : 50)
                    .animation(
                        .spring(response: 0.5, dampingFraction: 0.3, blendDuration: 0),
                        value: isExpanded
                    )
            }
            
            // Spring แบบ iOS 17+
            Group {
                Text("Bouncy Spring")
                    .font(.caption)
                Circle()
                    .fill(Color.orange)
                    .frame(width: isExpanded ? 100 : 50, height: isExpanded ? 100 : 50)
                    .animation(.bouncy, value: isExpanded)
            }
            
            // Spring แบบ smooth
            Group {
                Text("Smooth Spring")
                    .font(.caption)
                Circle()
                    .fill(Color.purple)
                    .frame(width: isExpanded ? 100 : 50, height: isExpanded ? 100 : 50)
                    .animation(.smooth, value: isExpanded)
            }
            
            Button(isExpanded ? "ย่อ" : "ขยาย") {
                isExpanded.toggle()
            }
            .buttonStyle(.borderedProminent)
        }
    }
}
```

### Spring Parameters (iOS 17+)

```swift
struct SpringParametersDemo: View {
    @State private var isBouncing = false
    
    var body: some View {
        VStack(spacing: 16) {
            // duration: ระยะเวลาทั้งหมด
            // bounce: ความกระเด้ง (0 = ไม่กระเด้ง, 1 = กระเด้งมาก)
            
            ForEach([0.0, 0.2, 0.4, 0.6, 0.8], id: \.self) { bounce in
                HStack {
                    Text("bounce: \(bounce, format: .number.precision(.fractionLength(1)))")
                        .font(.caption)
                        .frame(width: 100)
                    
                    RoundedRectangle(cornerRadius: 8)
                        .fill(Color.blue)
                        .frame(width: isBouncing ? 200 : 50, height: 30)
                        .animation(
                            .spring(duration: 0.8, bounce: bounce),
                            value: isBouncing
                        )
                    
                    Spacer()
                }
            }
            
            Button("เล่น") {
                isBouncing.toggle()
            }
            .buttonStyle(.borderedProminent)
        }
        .padding()
    }
}
```

### Animation Duration และ Delay

```swift
struct DurationDelayDemo: View {
    @State private var animate = false
    
    var body: some View {
        VStack(spacing: 20) {
            Text("Duration และ Delay")
                .font(.headline)
            
            // Duration ต่างกัน
            ForEach([0.3, 0.8, 1.5, 2.5], id: \.self) { duration in
                HStack {
                    Text("\(duration, format: .number)s")
                        .font(.caption)
                        .frame(width: 40)
                    
                    Circle()
                        .fill(Color.blue)
                        .frame(width: 20, height: 20)
                        .offset(x: animate ? 200 : 0)
                        .animation(.linear(duration: duration), value: animate)
                }
            }
            
            Divider()
            
            // Delay ต่างกัน
            Text("Staggered Animation (Delay)")
                .font(.subheadline)
            
            ForEach(0..<5, id: \.self) { i in
                Circle()
                    .fill(Color.orange)
                    .frame(width: 20, height: 20)
                    .offset(x: animate ? 150 : 0)
                    .animation(
                        .spring().delay(Double(i) * 0.1),
                        value: animate
                    )
            }
            
            Button("Animate") {
                animate.toggle()
            }
            .buttonStyle(.borderedProminent)
        }
        .padding()
    }
}
```

---

## 4. Implicit Animations (.animation modifier)

**Implicit Animation** ใช้ `.animation` modifier เพื่อกำหนด animation ให้กับ View โดยตรง animation จะทำงานเมื่อ value ที่กำหนดเปลี่ยนค่า

```swift
struct ImplicitAnimationExample: View {
    @State private var isLarge = false
    @State private var isMoved = false
    @State private var isRotated = false
    
    var body: some View {
        VStack(spacing: 30) {
            // Implicit animation ด้วย .animation
            RoundedRectangle(cornerRadius: 20)
                .fill(Color.blue)
                .frame(
                    width: isLarge ? 250 : 100,
                    height: isLarge ? 250 : 100
                )
                // กำหนด animation ให้ทำงานเมื่อ isLarge เปลี่ยน
                .animation(.spring(), value: isLarge)
            
            // ควบคุม animation แต่ละอย่างแยกกัน
            Image(systemName: "arrow.right.circle.fill")
                .font(.system(size: 60))
                .foregroundColor(.orange)
                .offset(x: isMoved ? 100 : 0)
                .rotationEffect(.degrees(isRotated ? 360 : 0))
                // animation สำหรับ offset
                .animation(.easeOut(duration: 0.4), value: isMoved)
                // animation สำหรับ rotation (แยกต่างหาก)
                .animation(.linear(duration: 1.0), value: isRotated)
            
            HStack(spacing: 16) {
                Button("ขยาย/ย่อ") { isLarge.toggle() }
                Button("เลื่อน") { isMoved.toggle() }
                Button("หมุน") { isRotated.toggle() }
            }
            .buttonStyle(.bordered)
        }
    }
}
```

### .animation กับ Binding

```swift
struct ImplicitWithBinding: View {
    @State private var value: Double = 0
    
    var body: some View {
        VStack(spacing: 20) {
            // Slider ที่ animate การเปลี่ยนแปลง
            Slider(value: $value.animation(.spring()), in: 0...100)
                .padding()
            
            // View ที่ตอบสนองต่อ value
            Circle()
                .fill(
                    Color(hue: value / 100, saturation: 0.8, brightness: 0.9)
                )
                .frame(width: 50 + value * 2, height: 50 + value * 2)
                .animation(.spring(), value: value)
            
            Text("\(Int(value))%")
                .font(.headline)
                .contentTransition(.numericText())
        }
    }
}
```

---

## 5. Explicit Animations (withAnimation)

**Explicit Animation** ใช้ `withAnimation { }` block เพื่อควบคุมได้ว่า state change ไหนจะ animate

```swift
struct ExplicitAnimationExample: View {
    @State private var position = CGPoint(x: 0, y: 0)
    @State private var scale: CGFloat = 1.0
    @State private var color = Color.blue
    @State private var opacity: Double = 1.0
    
    var body: some View {
        ZStack {
            // Background grid
            Color.gray.opacity(0.1)
            
            // Animated circle
            Circle()
                .fill(color)
                .frame(width: 60 * scale, height: 60 * scale)
                .opacity(opacity)
                .position(position)
                .onAppear {
                    position = CGPoint(x: 150, y: 300)
                }
        }
        .frame(height: 400)
        .onTapGesture { location in
            // Explicit animation: ทุกอย่างใน withAnimation จะ animate
            withAnimation(.spring(response: 0.4, dampingFraction: 0.6)) {
                position = location
                scale = CGFloat.random(in: 0.8...2.0)
                color = [.blue, .red, .green, .orange, .purple].randomElement()!
            }
        }
        .overlay(alignment: .bottom) {
            VStack(spacing: 8) {
                Text("แตะที่ไหนก็ได้เพื่อเคลื่อนที่")
                    .font(.caption)
                    .foregroundColor(.secondary)
                
                HStack(spacing: 12) {
                    Button("Fade In/Out") {
                        withAnimation(.easeInOut(duration: 0.5)) {
                            opacity = opacity == 1.0 ? 0.2 : 1.0
                        }
                    }
                    .buttonStyle(.bordered)
                    
                    Button("Reset") {
                        withAnimation(.spring()) {
                            position = CGPoint(x: 150, y: 300)
                            scale = 1.0
                            color = .blue
                            opacity = 1.0
                        }
                    }
                    .buttonStyle(.borderedProminent)
                }
            }
            .padding()
        }
    }
}
```

---

## 6. Transition (การเปลี่ยนผ่าน)

`Transition` กำหนดว่า View จะปรากฏและหายไปอย่างไร เมื่อ View ถูกเพิ่มหรือลบออกจาก hierarchy

### Built-in Transitions

```swift
struct TransitionExamples: View {
    @State private var showView = false
    @State private var selectedTransition = "opacity"
    
    let transitions = [
        "opacity", "scale", "slide", "move(top)", "move(bottom)",
        "move(leading)", "move(trailing)", "push"
    ]
    
    var currentTransition: AnyTransition {
        switch selectedTransition {
        case "opacity": return .opacity
        case "scale": return .scale
        case "slide": return .slide
        case "move(top)": return .move(edge: .top)
        case "move(bottom)": return .move(edge: .bottom)
        case "move(leading)": return .move(edge: .leading)
        case "move(trailing)": return .move(edge: .trailing)
        case "push": return .push(from: .leading)
        default: return .opacity
        }
    }
    
    var body: some View {
        VStack(spacing: 16) {
            // Picker เลือก transition
            Picker("Transition", selection: $selectedTransition) {
                ForEach(transitions, id: \.self) { t in
                    Text(t).tag(t)
                }
            }
            .pickerStyle(.wheel)
            .frame(height: 120)
            
            // พื้นที่แสดง transition
            ZStack {
                RoundedRectangle(cornerRadius: 16)
                    .fill(Color.gray.opacity(0.1))
                    .frame(height: 150)
                
                if showView {
                    RoundedRectangle(cornerRadius: 12)
                        .fill(Color.blue.gradient)
                        .frame(width: 150, height: 100)
                        .overlay(
                            Text(selectedTransition)
                                .font(.caption)
                                .foregroundColor(.white)
                        )
                        .transition(currentTransition)
                }
            }
            
            Button(showView ? "ซ่อน" : "แสดง") {
                withAnimation(.spring(duration: 0.5)) {
                    showView.toggle()
                }
            }
            .buttonStyle(.borderedProminent)
        }
        .padding()
    }
}
```

### Opacity Transition

```swift
struct OpacityTransitionDemo: View {
    @State private var isVisible = true
    
    var body: some View {
        VStack {
            if isVisible {
                Text("ข้อความที่ fade in/out")
                    .padding()
                    .background(Color.blue.opacity(0.2))
                    .cornerRadius(10)
                    .transition(.opacity)  // default transition
            }
            
            Button("Toggle") {
                withAnimation(.easeInOut(duration: 0.5)) {
                    isVisible.toggle()
                }
            }
        }
    }
}
```

### Scale Transition

```swift
struct ScaleTransitionDemo: View {
    @State private var showCard = false
    
    var body: some View {
        VStack {
            if showCard {
                RoundedRectangle(cornerRadius: 20)
                    .fill(Color.green.gradient)
                    .frame(width: 200, height: 200)
                    .overlay(
                        Text("Card")
                            .font(.title)
                            .foregroundColor(.white)
                    )
                    .transition(.scale(scale: 0.1, anchor: .center))
            }
            
            Button("Toggle Card") {
                withAnimation(.spring(response: 0.4, dampingFraction: 0.6)) {
                    showCard.toggle()
                }
            }
            .buttonStyle(.borderedProminent)
        }
    }
}
```

### Slide และ Move Transitions

```swift
struct SlideTransitionDemo: View {
    @State private var showNotification = false
    
    var body: some View {
        ZStack(alignment: .top) {
            Color.clear
            
            if showNotification {
                HStack {
                    Image(systemName: "bell.fill")
                        .foregroundColor(.white)
                    Text("คุณมีการแจ้งเตือนใหม่!")
                        .foregroundColor(.white)
                        .fontWeight(.semibold)
                    Spacer()
                    Button {
                        withAnimation(.spring()) {
                            showNotification = false
                        }
                    } label: {
                        Image(systemName: "xmark")
                            .foregroundColor(.white)
                    }
                }
                .padding()
                .background(Color.blue.gradient)
                .cornerRadius(12)
                .padding(.horizontal)
                // slide มาจากด้านบน
                .transition(.move(edge: .top).combined(with: .opacity))
            }
        }
        .onAppear {
            DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
                withAnimation(.spring()) {
                    showNotification = true
                }
            }
        }
        
        VStack {
            Spacer()
            Button("แสดงการแจ้งเตือน") {
                withAnimation(.spring()) {
                    showNotification = true
                }
            }
            .buttonStyle(.borderedProminent)
            .padding()
        }
    }
}
```

### Asymmetric Transition

```swift
struct AsymmetricTransitionDemo: View {
    @State private var showView = false
    
    var body: some View {
        VStack(spacing: 20) {
            if showView {
                RoundedRectangle(cornerRadius: 16)
                    .fill(Color.purple.gradient)
                    .frame(height: 150)
                    .overlay(
                        Text("Asymmetric Transition")
                            .font(.headline)
                            .foregroundColor(.white)
                    )
                    // ปรากฏจาก bottom, หายไปทาง top
                    .transition(.asymmetric(
                        insertion: .move(edge: .bottom).combined(with: .opacity),
                        removal: .move(edge: .top).combined(with: .opacity)
                    ))
            }
            
            Button(showView ? "ซ่อน (ไปทาง top)" : "แสดง (จาก bottom)") {
                withAnimation(.spring(duration: 0.5)) {
                    showView.toggle()
                }
            }
            .buttonStyle(.borderedProminent)
        }
        .padding()
    }
}
```

---

## 7. Custom Transitions

เราสร้าง custom transition ได้โดยใช้ `ViewModifier` และ `AnyTransition.modifier(active:identity:)`

### Custom Scale Rotation Transition

```swift
struct ScaleRotateModifier: ViewModifier {
    let scale: CGFloat
    let rotation: Angle
    let opacity: Double
    
    func body(content: Content) -> some View {
        content
            .scaleEffect(scale)
            .rotationEffect(rotation)
            .opacity(opacity)
    }
}

extension AnyTransition {
    static var scaleRotate: AnyTransition {
        .modifier(
            active: ScaleRotateModifier(scale: 0.1, rotation: .degrees(-180), opacity: 0),
            identity: ScaleRotateModifier(scale: 1, rotation: .degrees(0), opacity: 1)
        )
    }
    
    // Transition แบบ flip
    static var flip: AnyTransition {
        .modifier(
            active: ScaleRotateModifier(scale: 0.01, rotation: .degrees(90), opacity: 0),
            identity: ScaleRotateModifier(scale: 1, rotation: .degrees(0), opacity: 1)
        )
    }
}

struct CustomTransitionDemo: View {
    @State private var showCard = false
    @State private var selectedTransition = "scaleRotate"
    
    var body: some View {
        VStack(spacing: 20) {
            Picker("Custom Transition", selection: $selectedTransition) {
                Text("scaleRotate").tag("scaleRotate")
                Text("flip").tag("flip")
            }
            .pickerStyle(.segmented)
            
            ZStack {
                Color.gray.opacity(0.1)
                    .frame(height: 200)
                    .cornerRadius(16)
                
                if showCard {
                    RoundedRectangle(cornerRadius: 16)
                        .fill(Color.indigo.gradient)
                        .frame(width: 160, height: 160)
                        .overlay(
                            VStack {
                                Image(systemName: "sparkles")
                                    .font(.largeTitle)
                                Text("Custom!")
                                    .font(.headline)
                            }
                            .foregroundColor(.white)
                        )
                        .transition(selectedTransition == "scaleRotate" ? .scaleRotate : .flip)
                }
            }
            
            Button(showCard ? "ซ่อน" : "แสดง") {
                withAnimation(.spring(response: 0.5, dampingFraction: 0.7)) {
                    showCard.toggle()
                }
            }
            .buttonStyle(.borderedProminent)
        }
        .padding()
    }
}
```

### Blur Transition

```swift
struct BlurModifier: ViewModifier {
    let radius: CGFloat
    let opacity: Double
    
    func body(content: Content) -> some View {
        content
            .blur(radius: radius)
            .opacity(opacity)
    }
}

extension AnyTransition {
    static var blur: AnyTransition {
        .modifier(
            active: BlurModifier(radius: 20, opacity: 0),
            identity: BlurModifier(radius: 0, opacity: 1)
        )
    }
}

struct BlurTransitionDemo: View {
    @State private var showContent = false
    
    var body: some View {
        VStack(spacing: 20) {
            ZStack {
                Color.gray.opacity(0.1)
                    .frame(height: 200)
                    .cornerRadius(16)
                
                if showContent {
                    VStack(spacing: 8) {
                        Image(systemName: "photo.fill")
                            .font(.system(size: 50))
                            .foregroundColor(.blue)
                        Text("ภาพเบลอ Transition")
                            .font(.headline)
                    }
                    .transition(.blur)
                }
            }
            
            Button(showContent ? "ซ่อน" : "แสดง") {
                withAnimation(.easeInOut(duration: 0.6)) {
                    showContent.toggle()
                }
            }
            .buttonStyle(.borderedProminent)
        }
        .padding()
    }
}
```

---

## 8. matchedGeometryEffect

`matchedGeometryEffect` ทำให้ View ดูเหมือนย้ายจากตำแหน่งหนึ่งไปอีกตำแหน่งหนึ่งอย่างราบรื่น แม้จะเป็น View คนละตัวกัน

### Namespace

ก่อนใช้ `matchedGeometryEffect` ต้องสร้าง `@Namespace` ก่อน:

```swift
struct NamespaceExample: View {
    @Namespace private var animation
    @State private var isExpanded = false
    
    var body: some View {
        VStack {
            if isExpanded {
                // รูปใหญ่
                RoundedRectangle(cornerRadius: 20)
                    .fill(Color.blue.gradient)
                    .matchedGeometryEffect(id: "card", in: animation)
                    .frame(width: 300, height: 300)
                    .onTapGesture {
                        withAnimation(.spring()) {
                            isExpanded.toggle()
                        }
                    }
            } else {
                // รูปเล็ก
                RoundedRectangle(cornerRadius: 10)
                    .fill(Color.blue.gradient)
                    .matchedGeometryEffect(id: "card", in: animation)
                    .frame(width: 60, height: 60)
                    .onTapGesture {
                        withAnimation(.spring()) {
                            isExpanded.toggle()
                        }
                    }
            }
        }
    }
}
```

### matchedGeometryEffect สำหรับ Tab Bar

```swift
struct AnimatedTabBar: View {
    @Namespace private var tabAnimation
    @State private var selectedTab = "home"
    
    let tabs = [
        ("home", "house.fill", "บ้าน"),
        ("search", "magnifyingglass", "ค้นหา"),
        ("add", "plus.circle.fill", "เพิ่ม"),
        ("notifications", "bell.fill", "แจ้งเตือน"),
        ("profile", "person.fill", "โปรไฟล์")
    ]
    
    var body: some View {
        VStack(spacing: 0) {
            // เนื้อหาหลัก
            TabContentView(selectedTab: selectedTab)
                .frame(maxWidth: .infinity, maxHeight: .infinity)
            
            Divider()
            
            // Custom Tab Bar
            HStack(spacing: 0) {
                ForEach(tabs, id: \.0) { id, icon, label in
                    Spacer()
                    
                    VStack(spacing: 4) {
                        ZStack {
                            if selectedTab == id {
                                RoundedRectangle(cornerRadius: 10)
                                    .fill(Color.blue.opacity(0.15))
                                    .frame(width: 44, height: 32)
                                    .matchedGeometryEffect(id: "tabBg", in: tabAnimation)
                            }
                            
                            Image(systemName: icon)
                                .font(.system(size: 20))
                                .foregroundColor(selectedTab == id ? .blue : .gray)
                                .frame(width: 44, height: 32)
                        }
                        
                        Text(label)
                            .font(.system(size: 10))
                            .foregroundColor(selectedTab == id ? .blue : .gray)
                    }
                    .onTapGesture {
                        withAnimation(.spring(response: 0.3, dampingFraction: 0.7)) {
                            selectedTab = id
                        }
                    }
                    
                    Spacer()
                }
            }
            .padding(.vertical, 8)
            .background(Color(.systemBackground))
        }
    }
}

struct TabContentView: View {
    let selectedTab: String
    
    var body: some View {
        VStack {
            Image(systemName: selectedTab == "home" ? "house.fill" :
                             selectedTab == "search" ? "magnifyingglass" :
                             selectedTab == "add" ? "plus.circle.fill" :
                             selectedTab == "notifications" ? "bell.fill" : "person.fill")
                .font(.system(size: 80))
                .foregroundColor(.blue)
            Text(selectedTab.uppercased())
                .font(.title)
                .fontWeight(.bold)
        }
    }
}
```

### Card Expand Effect

```swift
struct CardExpandEffect: View {
    struct Item: Identifiable {
        let id = UUID()
        let title: String
        let description: String
        let color: Color
        let icon: String
    }
    
    let items = [
        Item(title: "SwiftUI", description: "เฟรมเวิร์คสำหรับสร้าง UI บน Apple platforms ด้วย declarative syntax", color: .blue, icon: "swift"),
        Item(title: "Combine", description: "Framework สำหรับ reactive programming และการจัดการ data flow", color: .orange, icon: "arrow.triangle.merge"),
        Item(title: "CoreData", description: "Framework สำหรับจัดการ persistent storage บน Apple platforms", color: .green, icon: "cylinder.fill")
    ]
    
    @Namespace private var cardAnimation
    @State private var selectedItem: Item?
    
    var body: some View {
        ZStack {
            // Card Grid
            ScrollView {
                LazyVGrid(columns: [GridItem(.flexible()), GridItem(.flexible())], spacing: 16) {
                    ForEach(items) { item in
                        if selectedItem?.id != item.id {
                            SmallCard(item: item, namespace: cardAnimation)
                                .onTapGesture {
                                    withAnimation(.spring(response: 0.5, dampingFraction: 0.75)) {
                                        selectedItem = item
                                    }
                                }
                        } else {
                            Color.clear
                                .frame(height: 150)
                        }
                    }
                }
                .padding()
            }
            
            // Expanded Card Overlay
            if let item = selectedItem {
                ExpandedCard(item: item, namespace: cardAnimation) {
                    withAnimation(.spring(response: 0.5, dampingFraction: 0.75)) {
                        selectedItem = nil
                    }
                }
            }
        }
    }
}

struct SmallCard: View {
    let item: CardExpandEffect.Item
    let namespace: Namespace.ID
    
    var body: some View {
        RoundedRectangle(cornerRadius: 16)
            .fill(item.color.gradient)
            .frame(height: 150)
            .overlay(
                VStack(alignment: .leading, spacing: 8) {
                    Image(systemName: item.icon)
                        .font(.title)
                        .foregroundColor(.white)
                    Spacer()
                    Text(item.title)
                        .font(.headline)
                        .foregroundColor(.white)
                }
                .padding()
            )
            .matchedGeometryEffect(id: "card-\(item.id)", in: namespace)
    }
}

struct ExpandedCard: View {
    let item: CardExpandEffect.Item
    let namespace: Namespace.ID
    let onClose: () -> Void
    
    var body: some View {
        RoundedRectangle(cornerRadius: 24)
            .fill(item.color.gradient)
            .ignoresSafeArea()
            .overlay(
                VStack(alignment: .leading, spacing: 16) {
                    HStack {
                        Spacer()
                        Button {
                            onClose()
                        } label: {
                            Image(systemName: "xmark.circle.fill")
                                .font(.title)
                                .foregroundColor(.white.opacity(0.8))
                        }
                    }
                    
                    Image(systemName: item.icon)
                        .font(.system(size: 80))
                        .foregroundColor(.white)
                    
                    Text(item.title)
                        .font(.largeTitle)
                        .fontWeight(.bold)
                        .foregroundColor(.white)
                    
                    Text(item.description)
                        .font(.body)
                        .foregroundColor(.white.opacity(0.9))
                        .lineSpacing(4)
                    
                    Spacer()
                }
                .padding(30)
            )
            .matchedGeometryEffect(id: "card-\(item.id)", in: namespace)
    }
}
```

---

## 9. Phase Animators (iOS 17+)

`PhaseAnimator` ทำให้ animate ผ่าน phase หลายขั้นตอนโดยอัตโนมัติ

### PhaseAnimator พื้นฐาน

```swift
struct PhaseAnimatorBasic: View {
    @State private var isAnimating = false
    
    var body: some View {
        VStack(spacing: 30) {
            // Bounce animation ด้วย PhaseAnimator
            PhaseAnimator([false, true]) { phase in
                Image(systemName: "heart.fill")
                    .font(.system(size: 60))
                    .foregroundColor(.red)
                    .scaleEffect(phase ? 1.3 : 1.0)
                    .shadow(
                        color: .red.opacity(phase ? 0.5 : 0),
                        radius: phase ? 20 : 0
                    )
            } animation: { phase in
                phase ? .easeOut(duration: 0.3) : .easeIn(duration: 0.5)
            }
            
            Text("หัวใจกำลังเต้น!")
                .font(.headline)
        }
    }
}
```

### PhaseAnimator กับ Custom Phases

```swift
enum LoadingPhase: CaseIterable {
    case initial, growing, full, shrinking
    
    var scale: CGFloat {
        switch self {
        case .initial: return 0.5
        case .growing: return 1.2
        case .full: return 1.0
        case .shrinking: return 0.7
        }
    }
    
    var opacity: Double {
        switch self {
        case .initial: return 0.3
        case .growing: return 1.0
        case .full: return 0.8
        case .shrinking: return 0.5
        }
    }
    
    var color: Color {
        switch self {
        case .initial: return .blue
        case .growing: return .purple
        case .full: return .pink
        case .shrinking: return .blue
        }
    }
}

struct CustomPhaseAnimator: View {
    var body: some View {
        VStack {
            Text("Custom Phase Animator")
                .font(.headline)
            
            HStack(spacing: 8) {
                ForEach(0..<3, id: \.self) { i in
                    PhaseAnimator(LoadingPhase.allCases) { phase in
                        Circle()
                            .fill(phase.color)
                            .frame(width: 20, height: 20)
                            .scaleEffect(phase.scale)
                            .opacity(phase.opacity)
                    } animation: { _ in
                        .spring(duration: 0.4)
                        .delay(Double(i) * 0.15)
                    }
                }
            }
        }
    }
}
```

---

## 10. Keyframe Animations (iOS 17+)

`KeyframeAnimator` ช่วยสร้าง animation ที่ซับซ้อนโดยกำหนด keyframes สำหรับแต่ละ property

### KeyframeAnimator พื้นฐาน

```swift
struct KeyframeBasic: View {
    @State private var trigger = false
    
    struct AnimationValues {
        var scale = 1.0
        var rotation = Angle.zero
        var verticalOffset = 0.0
    }
    
    var body: some View {
        VStack(spacing: 20) {
            KeyframeAnimator(
                initialValue: AnimationValues(),
                trigger: trigger
            ) { values in
                Image(systemName: "airplane")
                    .font(.system(size: 60))
                    .foregroundColor(.blue)
                    .scaleEffect(values.scale)
                    .rotationEffect(values.rotation)
                    .offset(y: values.verticalOffset)
            } keyframes: { _ in
                // keyframes สำหรับ scale
                KeyframeTrack(\.scale) {
                    LinearKeyframe(1.0, duration: 0.1)
                    SpringKeyframe(1.5, duration: 0.3, spring: .bouncy)
                    SpringKeyframe(1.0, duration: 0.4, spring: .smooth)
                }
                
                // keyframes สำหรับ rotation
                KeyframeTrack(\.rotation) {
                    LinearKeyframe(.zero, duration: 0.1)
                    CubicKeyframe(.degrees(-20), duration: 0.2)
                    CubicKeyframe(.degrees(20), duration: 0.3)
                    CubicKeyframe(.degrees(-10), duration: 0.2)
                    SpringKeyframe(.zero, duration: 0.3, spring: .smooth)
                }
                
                // keyframes สำหรับ vertical offset
                KeyframeTrack(\.verticalOffset) {
                    LinearKeyframe(0, duration: 0.1)
                    SpringKeyframe(-50, duration: 0.4, spring: .bouncy)
                    SpringKeyframe(0, duration: 0.5, spring: .bouncy)
                }
            }
            
            Button("เหินฟ้า!") {
                trigger.toggle()
            }
            .buttonStyle(.borderedProminent)
        }
    }
}
```

### Keyframe Animation ที่ซับซ้อน

```swift
struct ComplexKeyframeAnimation: View {
    @State private var bouncing = false
    
    struct BallValues {
        var yOffset: CGFloat = 0
        var xScale: CGFloat = 1
        var yScale: CGFloat = 1
        var shadowRadius: CGFloat = 10
        var shadowY: CGFloat = 10
    }
    
    var body: some View {
        VStack {
            Spacer()
            
            ZStack(alignment: .bottom) {
                // พื้น
                Rectangle()
                    .fill(Color.gray.opacity(0.3))
                    .frame(height: 4)
                    .cornerRadius(2)
                
                // เงา
                KeyframeAnimator(initialValue: BallValues(), trigger: bouncing) { values in
                    Ellipse()
                        .fill(Color.black.opacity(0.2))
                        .frame(width: 60, height: 10)
                        .blur(radius: values.shadowRadius)
                        .offset(y: values.shadowY)
                } keyframes: { _ in
                    KeyframeTrack(\.shadowRadius) {
                        CubicKeyframe(15, duration: 0.3)
                        CubicKeyframe(5, duration: 0.3)
                        CubicKeyframe(15, duration: 0.3)
                        CubicKeyframe(5, duration: 0.3)
                        CubicKeyframe(10, duration: 0.2)
                    }
                    KeyframeTrack(\.shadowY) {
                        CubicKeyframe(20, duration: 0.3)
                        CubicKeyframe(5, duration: 0.3)
                        CubicKeyframe(20, duration: 0.3)
                        CubicKeyframe(5, duration: 0.3)
                        CubicKeyframe(10, duration: 0.2)
                    }
                }
                
                // ลูกบอล
                KeyframeAnimator(initialValue: BallValues(), trigger: bouncing) { values in
                    Circle()
                        .fill(Color.red.gradient)
                        .frame(width: 60, height: 60)
                        .scaleEffect(x: values.xScale, y: values.yScale)
                        .offset(y: values.yOffset)
                } keyframes: { _ in
                    // เด้งขึ้น
                    KeyframeTrack(\.yOffset) {
                        SpringKeyframe(-150, duration: 0.3, spring: .bouncy)
                        SpringKeyframe(-130, duration: 0.1, spring: .smooth)
                        SpringKeyframe(-80, duration: 0.2, spring: .bouncy)
                        SpringKeyframe(-70, duration: 0.1, spring: .smooth)
                        SpringKeyframe(0, duration: 0.25, spring: .bouncy)
                    }
                    
                    // บีบเมื่อกระทบพื้น
                    KeyframeTrack(\.xScale) {
                        LinearKeyframe(1.0, duration: 0.3)
                        CubicKeyframe(1.3, duration: 0.05)
                        CubicKeyframe(1.0, duration: 0.2)
                        CubicKeyframe(1.2, duration: 0.05)
                        CubicKeyframe(1.0, duration: 0.3)
                    }
                    
                    KeyframeTrack(\.yScale) {
                        LinearKeyframe(1.0, duration: 0.3)
                        CubicKeyframe(0.7, duration: 0.05)
                        CubicKeyframe(1.0, duration: 0.2)
                        CubicKeyframe(0.8, duration: 0.05)
                        CubicKeyframe(1.0, duration: 0.3)
                    }
                }
            }
            .frame(height: 200)
            
            Spacer()
            
            Button("กระเด้ง!") {
                bouncing.toggle()
            }
            .buttonStyle(.borderedProminent)
            .padding()
        }
    }
}
```

---

## 11. Canvas View

`Canvas` เป็น View สำหรับวาดกราฟิกแบบ low-level ด้วย 2D drawing API มีประสิทธิภาพสูงสำหรับ custom graphics

### Canvas พื้นฐาน

```swift
struct CanvasBasic: View {
    var body: some View {
        Canvas { context, size in
            // วาดวงกลม
            context.fill(
                Path(ellipseIn: CGRect(x: 50, y: 50, width: 100, height: 100)),
                with: .color(.blue)
            )
            
            // วาดสี่เหลี่ยม
            context.fill(
                Path(CGRect(x: 200, y: 50, width: 100, height: 100)),
                with: .color(.red)
            )
            
            // วาดเส้น
            var path = Path()
            path.move(to: CGPoint(x: 0, y: size.height / 2))
            path.addLine(to: CGPoint(x: size.width, y: size.height / 2))
            context.stroke(path, with: .color(.green), lineWidth: 3)
            
            // วาดข้อความ
            context.draw(
                Text("Canvas Drawing")
                    .font(.headline)
                    .foregroundColor(.primary),
                at: CGPoint(x: size.width / 2, y: size.height - 40)
            )
        }
        .frame(height: 200)
        .background(Color(.systemGray6))
        .cornerRadius(16)
        .padding()
    }
}
```

### Canvas กับ Animation

```swift
struct AnimatedCanvas: View {
    @State private var time: Double = 0
    let timer = Timer.publish(every: 1/60, on: .main, in: .common).autoconnect()
    
    var body: some View {
        Canvas { context, size in
            let centerX = size.width / 2
            let centerY = size.height / 2
            
            // วาดวงโคจรของดาวเคราะห์
            for i in 0..<8 {
                let orbitRadius = Double(50 + i * 30)
                let speed = 1.0 / (1.0 + Double(i) * 0.5)
                let angle = time * speed + Double(i) * 0.5
                
                let x = centerX + orbitRadius * cos(angle)
                let y = centerY + orbitRadius * sin(angle)
                
                // วาดเส้นวงโคจร
                let orbitPath = Path(ellipseIn: CGRect(
                    x: centerX - orbitRadius,
                    y: centerY - orbitRadius,
                    width: orbitRadius * 2,
                    height: orbitRadius * 2
                ))
                context.stroke(orbitPath, with: .color(.white.opacity(0.1)), lineWidth: 1)
                
                // วาดดาวเคราะห์
                let planetColors: [Color] = [.blue, .red, .green, .orange, .purple, .yellow, .cyan, .pink]
                let planetSize = Double(6 + i * 2)
                
                context.fill(
                    Path(ellipseIn: CGRect(
                        x: x - planetSize/2,
                        y: y - planetSize/2,
                        width: planetSize,
                        height: planetSize
                    )),
                    with: .color(planetColors[i % planetColors.count])
                )
            }
            
            // วาดดวงอาทิตย์ตรงกลาง
            let sunGlow = Path(ellipseIn: CGRect(
                x: centerX - 25,
                y: centerY - 25,
                width: 50,
                height: 50
            ))
            context.fill(sunGlow, with: .color(.yellow.opacity(0.3)))
            
            let sun = Path(ellipseIn: CGRect(
                x: centerX - 18,
                y: centerY - 18,
                width: 36,
                height: 36
            ))
            context.fill(sun, with: .color(.yellow))
        }
        .frame(height: 350)
        .background(Color.black)
        .cornerRadius(20)
        .padding()
        .onReceive(timer) { _ in
            time += 0.02
        }
    }
}
```

### Canvas สำหรับ Particle System

```swift
struct ParticleCanvas: View {
    struct Particle {
        var x: Double
        var y: Double
        var vx: Double
        var vy: Double
        var life: Double
        var size: Double
        var color: Color
    }
    
    @State private var particles: [Particle] = []
    @State private var time: Double = 0
    let timer = Timer.publish(every: 1/60, on: .main, in: .common).autoconnect()
    
    var body: some View {
        Canvas { context, size in
            for particle in particles {
                let alpha = particle.life
                context.fill(
                    Path(ellipseIn: CGRect(
                        x: particle.x - particle.size/2,
                        y: particle.y - particle.size/2,
                        width: particle.size,
                        height: particle.size
                    )),
                    with: .color(particle.color.opacity(alpha))
                )
            }
        }
        .frame(height: 300)
        .background(Color.black)
        .cornerRadius(20)
        .padding()
        .onTapGesture { location in
            // สร้าง particles ที่ตำแหน่งที่แตะ
            for _ in 0..<30 {
                let angle = Double.random(in: 0...360) * .pi / 180
                let speed = Double.random(in: 2...8)
                let colors: [Color] = [.blue, .red, .yellow, .green, .orange, .purple, .pink, .cyan]
                particles.append(Particle(
                    x: location.x,
                    y: location.y,
                    vx: cos(angle) * speed,
                    vy: sin(angle) * speed,
                    life: 1.0,
                    size: Double.random(in: 4...12),
                    color: colors.randomElement()!
                ))
            }
        }
        .onReceive(timer) { _ in
            // อัปเดต particles
            particles = particles.compactMap { p in
                var updated = p
                updated.x += p.vx
                updated.y += p.vy
                updated.vy += 0.2 // gravity
                updated.life -= 0.02
                return updated.life > 0 ? updated : nil
            }
        }
    }
}
```

---

## 12. TimelineView

`TimelineView` เป็น View ที่อัปเดตตาม schedule ที่กำหนด เหมาะสำหรับ live content เช่น นาฬิกา หรือ animation

### TimelineView พื้นฐาน

```swift
struct TimelineViewBasic: View {
    var body: some View {
        TimelineView(.animation) { timeline in
            let date = timeline.date
            let seconds = Calendar.current.component(.second, from: date)
            let minutes = Calendar.current.component(.minute, from: date)
            let hours = Calendar.current.component(.hour, from: date)
            
            // นาฬิกาอะนาล็อก
            AnalogClock(hours: hours, minutes: minutes, seconds: seconds)
                .frame(width: 200, height: 200)
        }
    }
}

struct AnalogClock: View {
    let hours: Int
    let minutes: Int
    let seconds: Int
    
    var secondAngle: Double { Double(seconds) * 6 }
    var minuteAngle: Double { Double(minutes) * 6 + Double(seconds) * 0.1 }
    var hourAngle: Double { Double(hours % 12) * 30 + Double(minutes) * 0.5 }
    
    var body: some View {
        ZStack {
            // หน้าปัด
            Circle()
                .fill(Color(.systemBackground))
                .shadow(radius: 10)
            
            Circle()
                .stroke(Color.gray.opacity(0.3), lineWidth: 2)
            
            // ขีดบอกชั่วโมง
            ForEach(0..<12, id: \.self) { hour in
                Rectangle()
                    .fill(Color.primary)
                    .frame(width: 2, height: 10)
                    .offset(y: -80)
                    .rotationEffect(.degrees(Double(hour) * 30))
            }
            
            // เข็มชั่วโมง
            ClockHand(angle: hourAngle, length: 55, width: 5, color: .primary)
            
            // เข็มนาที
            ClockHand(angle: minuteAngle, length: 70, width: 3, color: .primary)
            
            // เข็มวินาที
            ClockHand(angle: secondAngle, length: 75, width: 1.5, color: .red)
            
            // จุดกลาง
            Circle()
                .fill(Color.red)
                .frame(width: 10, height: 10)
        }
    }
}

struct ClockHand: View {
    let angle: Double
    let length: CGFloat
    let width: CGFloat
    let color: Color
    
    var body: some View {
        Rectangle()
            .fill(color)
            .frame(width: width, height: length)
            .offset(y: -length/2)
            .rotationEffect(.degrees(angle))
    }
}
```

### TimelineView กับ เอฟเฟกต์

```swift
struct TimelineEffects: View {
    var body: some View {
        TimelineView(.animation(minimumInterval: 1/60)) { timeline in
            let elapsed = timeline.date.timeIntervalSince1970
            
            AnimatedGradientBackground(time: elapsed)
        }
    }
}

struct AnimatedGradientBackground: View {
    let time: Double
    
    var body: some View {
        Canvas { context, size in
            let width = size.width
            let height = size.height
            
            // สร้าง animated gradient
            let gradient = Gradient(stops: [
                .init(color: Color(hue: fmod(time * 0.1, 1.0), saturation: 0.7, brightness: 0.9), location: 0),
                .init(color: Color(hue: fmod(time * 0.1 + 0.3, 1.0), saturation: 0.8, brightness: 0.7), location: 0.5),
                .init(color: Color(hue: fmod(time * 0.1 + 0.6, 1.0), saturation: 0.9, brightness: 0.8), location: 1)
            ])
            
            context.fill(
                Path(CGRect(origin: .zero, size: size)),
                with: .linearGradient(
                    gradient,
                    startPoint: CGPoint(x: 0, y: 0),
                    endPoint: CGPoint(x: width, y: height)
                )
            )
            
            // วาดคลื่น
            var wavePath = Path()
            wavePath.move(to: CGPoint(x: 0, y: height/2))
            
            for x in stride(from: 0, through: width, by: 2) {
                let y = height/2 + sin(x * 0.02 + time * 2) * 30 + sin(x * 0.05 + time * 3) * 15
                wavePath.addLine(to: CGPoint(x: x, y: y))
            }
            
            context.stroke(wavePath, with: .color(.white.opacity(0.4)), lineWidth: 2)
        }
        .frame(height: 300)
        .cornerRadius(20)
        .padding()
    }
}
```

---

## 13. Animation Sequences

การสร้าง sequence ของ animation โดยใช้ delay และ completion handlers

### Sequential Animation ด้วย Delay

```swift
struct SequentialAnimation: View {
    @State private var step1 = false
    @State private var step2 = false
    @State private var step3 = false
    @State private var step4 = false
    
    var body: some View {
        VStack(spacing: 20) {
            // Step indicators ที่ animate ทีละขั้น
            HStack(spacing: 0) {
                ForEach(1...4, id: \.self) { step in
                    let isActive = (step == 1 && step1) ||
                                   (step == 2 && step2) ||
                                   (step == 3 && step3) ||
                                   (step == 4 && step4)
                    
                    Circle()
                        .fill(isActive ? Color.blue : Color.gray.opacity(0.3))
                        .frame(width: 40, height: 40)
                        .overlay(
                            Text("\(step)")
                                .foregroundColor(isActive ? .white : .gray)
                                .font(.headline)
                        )
                        .scaleEffect(isActive ? 1.1 : 1.0)
                    
                    if step < 4 {
                        Rectangle()
                            .fill(step <= 3 && (step == 1 && step2 || step == 2 && step3 || step == 3 && step4) ?
                                  Color.blue : Color.gray.opacity(0.3))
                            .frame(height: 3)
                    }
                }
            }
            .padding(.horizontal)
            
            Button("เริ่ม Sequence") {
                runSequence()
            }
            .buttonStyle(.borderedProminent)
            
            Button("รีเซ็ต") {
                step1 = false; step2 = false; step3 = false; step4 = false
            }
            .buttonStyle(.bordered)
        }
    }
    
    func runSequence() {
        withAnimation(.spring()) {
            step1 = true
        }
        
        DispatchQueue.main.asyncAfter(deadline: .now() + 0.5) {
            withAnimation(.spring()) {
                step2 = true
            }
        }
        
        DispatchQueue.main.asyncAfter(deadline: .now() + 1.0) {
            withAnimation(.spring()) {
                step3 = true
            }
        }
        
        DispatchQueue.main.asyncAfter(deadline: .now() + 1.5) {
            withAnimation(.spring()) {
                step4 = true
            }
        }
    }
}
```

### ใช้ Task กับ async/await สำหรับ Sequential Animation

```swift
struct AsyncSequentialAnimation: View {
    @State private var boxes: [Bool] = Array(repeating: false, count: 6)
    @State private var isRunning = false
    
    var body: some View {
        VStack(spacing: 20) {
            LazyVGrid(columns: Array(repeating: GridItem(.flexible()), count: 3), spacing: 16) {
                ForEach(0..<6, id: \.self) { i in
                    RoundedRectangle(cornerRadius: 12)
                        .fill(boxes[i] ? Color.blue.gradient : Color.gray.opacity(0.2))
                        .frame(height: 80)
                        .scaleEffect(boxes[i] ? 1.1 : 1.0)
                        .shadow(color: boxes[i] ? .blue.opacity(0.3) : .clear, radius: 8)
                }
            }
            .padding()
            
            Button(isRunning ? "กำลังเล่น..." : "เล่น Sequence") {
                Task {
                    await runAnimationSequence()
                }
            }
            .buttonStyle(.borderedProminent)
            .disabled(isRunning)
        }
    }
    
    func runAnimationSequence() async {
        isRunning = true
        
        // Reset
        for i in 0..<6 {
            withAnimation { boxes[i] = false }
        }
        
        try? await Task.sleep(nanoseconds: 300_000_000)
        
        // Animate ทีละกล่อง
        for i in 0..<6 {
            withAnimation(.spring(response: 0.3, dampingFraction: 0.6)) {
                boxes[i] = true
            }
            try? await Task.sleep(nanoseconds: 200_000_000)
        }
        
        try? await Task.sleep(nanoseconds: 500_000_000)
        
        // ปิดทีละกล่องกลับ
        for i in stride(from: 5, through: 0, by: -1) {
            withAnimation(.easeOut(duration: 0.2)) {
                boxes[i] = false
            }
            try? await Task.sleep(nanoseconds: 150_000_000)
        }
        
        isRunning = false
    }
}
```

---

## 14. Symbol Effects (iOS 17+)

iOS 17 เพิ่ม `symbolEffect` modifier สำหรับ SF Symbols ทำให้ animate icons ได้ง่าย

```swift
struct SymbolEffectsDemo: View {
    @State private var isActive = false
    @State private var bounce = false
    @State private var breathe = false
    @State private var rotate = false
    @State private var appear = false
    
    var body: some View {
        List {
            Section("Symbol Effects (iOS 17+)") {
                // Bounce
                HStack {
                    Image(systemName: "bell.fill")
                        .font(.title)
                        .foregroundColor(.blue)
                        .symbolEffect(.bounce, value: bounce)
                    Text("Bounce")
                    Spacer()
                    Button("เล่น") { bounce.toggle() }
                        .buttonStyle(.bordered)
                }
                
                // Pulse / Breathe
                HStack {
                    Image(systemName: "heart.fill")
                        .font(.title)
                        .foregroundColor(.red)
                        .symbolEffect(.pulse, isActive: isActive)
                    Text("Pulse (continuous)")
                    Spacer()
                    Toggle("", isOn: $isActive)
                }
                
                // Rotate
                HStack {
                    Image(systemName: "gear")
                        .font(.title)
                        .foregroundColor(.gray)
                        .symbolEffect(.rotate, isActive: rotate)
                    Text("Rotate")
                    Spacer()
                    Toggle("", isOn: $rotate)
                }
                
                // Wiggle
                HStack {
                    Image(systemName: "exclamationmark.triangle.fill")
                        .font(.title)
                        .foregroundColor(.orange)
                        .symbolEffect(.wiggle, value: bounce)
                    Text("Wiggle")
                    Spacer()
                    Button("เล่น") { bounce.toggle() }
                        .buttonStyle(.bordered)
                }
                
                // Scale
                HStack {
                    Image(systemName: "star.fill")
                        .font(.title)
                        .foregroundColor(.yellow)
                        .symbolEffect(.scale.up, isActive: isActive)
                    Text("Scale Up")
                    Spacer()
                    Toggle("", isOn: $isActive)
                }
            }
            
            Section("Content Transition") {
                // Replace
                HStack {
                    Button {
                        appear.toggle()
                    } label: {
                        Image(systemName: appear ? "checkmark.circle.fill" : "circle")
                            .font(.title)
                            .foregroundColor(appear ? .green : .gray)
                            .contentTransition(.symbolEffect(.replace))
                    }
                    Text("Replace transition")
                    Spacer()
                }
            }
        }
    }
}
```

---

## 15. แบบฝึกหัด: Building Animated UI Components

```swift
// แบบฝึกหัดที่ 1: Animated Toggle Button

import SwiftUI

struct AnimatedToggleButton: View {
    @State private var isOn = false
    
    var body: some View {
        Button {
            withAnimation(.spring(response: 0.3, dampingFraction: 0.7)) {
                isOn.toggle()
            }
        } label: {
            ZStack {
                // Track
                RoundedRectangle(cornerRadius: 30)
                    .fill(isOn ? Color.green : Color.gray.opacity(0.4))
                    .frame(width: 70, height: 38)
                
                // Thumb
                Circle()
                    .fill(.white)
                    .frame(width: 30, height: 30)
                    .shadow(color: .black.opacity(0.2), radius: 3, x: 0, y: 2)
                    .offset(x: isOn ? 16 : -16)
                
                // Icon
                Image(systemName: isOn ? "checkmark" : "xmark")
                    .font(.system(size: 12, weight: .bold))
                    .foregroundColor(isOn ? .green : .gray)
                    .offset(x: isOn ? -12 : 12)
            }
        }
    }
}

// แบบฝึกหัดที่ 2: Animated Like Button

struct AnimatedLikeButton: View {
    @State private var isLiked = false
    @State private var likeCount = 42
    @State private var showParticles = false
    
    var body: some View {
        ZStack {
            // Particles
            if showParticles {
                ParticlesView()
                    .allowsHitTesting(false)
            }
            
            HStack(spacing: 8) {
                Button {
                    withAnimation(.spring(response: 0.3, dampingFraction: 0.5)) {
                        isLiked.toggle()
                        likeCount += isLiked ? 1 : -1
                    }
                    
                    if isLiked {
                        showParticles = true
                        DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
                            showParticles = false
                        }
                    }
                } label: {
                    Image(systemName: isLiked ? "heart.fill" : "heart")
                        .font(.title2)
                        .foregroundColor(isLiked ? .red : .gray)
                        .scaleEffect(isLiked ? 1.2 : 1.0)
                        .symbolEffect(.bounce, value: isLiked)
                }
                
                Text("\(likeCount)")
                    .font(.subheadline)
                    .foregroundColor(.secondary)
                    .contentTransition(.numericText())
            }
        }
    }
}

struct ParticlesView: View {
    let particleCount = 12
    
    var body: some View {
        ZStack {
            ForEach(0..<particleCount, id: \.self) { i in
                ParticleView(index: i)
            }
        }
    }
}

struct ParticleView: View {
    let index: Int
    @State private var offset: CGSize = .zero
    @State private var opacity: Double = 1.0
    @State private var scale: CGFloat = 1.0
    
    let colors: [Color] = [.red, .orange, .yellow, .pink, .purple, .blue]
    
    var body: some View {
        Circle()
            .fill(colors[index % colors.count])
            .frame(width: 8, height: 8)
            .scaleEffect(scale)
            .offset(offset)
            .opacity(opacity)
            .onAppear {
                let angle = Double(index) * 30 * .pi / 180
                let distance: CGFloat = 40
                
                withAnimation(.easeOut(duration: 0.6)) {
                    offset = CGSize(
                        width: cos(angle) * distance,
                        height: sin(angle) * distance
                    )
                    scale = 0.3
                    opacity = 0
                }
            }
    }
}

// แบบฝึกหัดที่ 3: Animated Progress Ring

struct AnimatedProgressRing: View {
    let progress: Double  // 0.0 - 1.0
    let color: Color
    let lineWidth: CGFloat
    
    @State private var animatedProgress: Double = 0
    
    var body: some View {
        ZStack {
            // Background Ring
            Circle()
                .stroke(color.opacity(0.2), lineWidth: lineWidth)
            
            // Progress Ring
            Circle()
                .trim(from: 0, to: animatedProgress)
                .stroke(
                    color,
                    style: StrokeStyle(
                        lineWidth: lineWidth,
                        lineCap: .round
                    )
                )
                .rotationEffect(.degrees(-90))
                .animation(.spring(duration: 1.2, bounce: 0.2), value: animatedProgress)
            
            // Percentage Text
            VStack(spacing: 2) {
                Text("\(Int(animatedProgress * 100))%")
                    .font(.title2)
                    .fontWeight(.bold)
                    .contentTransition(.numericText())
                Text("เสร็จสิ้น")
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
        }
        .onAppear {
            animatedProgress = progress
        }
        .onChange(of: progress) { _, newValue in
            animatedProgress = newValue
        }
    }
}

// Demo
struct AnimatedComponentsDemo: View {
    @State private var progress: Double = 0.65
    
    var body: some View {
        ScrollView {
            VStack(spacing: 40) {
                Text("Animated UI Components")
                    .font(.headline)
                
                // Toggle
                VStack {
                    Text("Animated Toggle")
                        .font(.subheadline)
                        .foregroundColor(.secondary)
                    AnimatedToggleButton()
                }
                
                // Like Button
                VStack {
                    Text("Animated Like Button")
                        .font(.subheadline)
                        .foregroundColor(.secondary)
                    AnimatedLikeButton()
                }
                
                // Progress Ring
                VStack(spacing: 16) {
                    Text("Animated Progress Ring")
                        .font(.subheadline)
                        .foregroundColor(.secondary)
                    
                    HStack(spacing: 30) {
                        AnimatedProgressRing(progress: 0.75, color: .blue, lineWidth: 12)
                            .frame(width: 100, height: 100)
                        
                        AnimatedProgressRing(progress: 0.45, color: .green, lineWidth: 10)
                            .frame(width: 80, height: 80)
                        
                        AnimatedProgressRing(progress: 0.9, color: .orange, lineWidth: 8)
                            .frame(width: 60, height: 60)
                    }
                    
                    Slider(value: $progress)
                        .padding(.horizontal)
                    
                    AnimatedProgressRing(progress: progress, color: .purple, lineWidth: 15)
                        .frame(width: 150, height: 150)
                }
                
                Spacer()
            }
            .padding()
        }
    }
}
```

---

## 16. แบบฝึกหัดที่สมบูรณ์: Animated Onboarding Screen

```swift
// สร้าง Onboarding Screen พร้อม Animation ที่สมบูรณ์

import SwiftUI

struct OnboardingScreen: View {
    struct OnboardingPage: Identifiable {
        let id = UUID()
        let icon: String
        let color: Color
        let title: String
        let description: String
    }
    
    let pages = [
        OnboardingPage(
            icon: "swift",
            color: .orange,
            title: "เรียนรู้ Swift",
            description: "เริ่มต้นเรียนรู้ภาษา Swift สมัยใหม่ที่ทรงพลัง สำหรับการพัฒนา iOS, macOS, watchOS และ tvOS"
        ),
        OnboardingPage(
            icon: "rectangle.3.group.fill",
            color: .blue,
            title: "สร้าง UI ด้วย SwiftUI",
            description: "ออกแบบ UI ที่สวยงามด้วย declarative syntax เข้าใจง่าย เขียนน้อย ได้มาก"
        ),
        OnboardingPage(
            icon: "network",
            color: .green,
            title: "Connect กับ API",
            description: "เชื่อมต่อกับ API ภายนอก จัดการข้อมูลแบบ async/await และแสดงผลแบบ real-time"
        ),
        OnboardingPage(
            icon: "star.fill",
            color: .purple,
            title: "เป็น iOS Developer",
            description: "ก้าวเข้าสู่วงการ iOS Development อย่างมั่นใจ ด้วยความรู้ที่ครบครัน"
        )
    ]
    
    @State private var currentPage = 0
    @State private var isFinished = false
    @Namespace private var pageTransition
    
    var body: some View {
        if isFinished {
            MainAppView()
                .transition(.asymmetric(
                    insertion: .move(edge: .trailing).combined(with: .opacity),
                    removal: .opacity
                ))
        } else {
            onboardingContent
                .transition(.opacity)
        }
    }
    
    var onboardingContent: some View {
        ZStack {
            // Animated Background
            AnimatedBackground(color: pages[currentPage].color)
            
            VStack(spacing: 0) {
                // Page Content
                TabView(selection: $currentPage) {
                    ForEach(Array(pages.enumerated()), id: \.offset) { index, page in
                        PageContentView(page: page)
                            .tag(index)
                    }
                }
                .tabViewStyle(.page(indexDisplayMode: .never))
                .animation(.spring(response: 0.5, dampingFraction: 0.8), value: currentPage)
                
                // Bottom Controls
                BottomControls(
                    currentPage: $currentPage,
                    totalPages: pages.count,
                    color: pages[currentPage].color,
                    onFinish: {
                        withAnimation(.spring()) {
                            isFinished = true
                        }
                    }
                )
                .padding(.bottom, 50)
            }
        }
    }
}

// MARK: - Animated Background
struct AnimatedBackground: View {
    let color: Color
    
    var body: some View {
        ZStack {
            color
                .opacity(0.1)
                .ignoresSafeArea()
                .animation(.easeInOut(duration: 0.5), value: color)
            
            // Decorative circles
            Circle()
                .fill(color.opacity(0.15))
                .frame(width: 400, height: 400)
                .offset(x: 150, y: -200)
                .animation(.spring(duration: 0.8), value: color)
            
            Circle()
                .fill(color.opacity(0.1))
                .frame(width: 300, height: 300)
                .offset(x: -180, y: 250)
                .animation(.spring(duration: 0.9), value: color)
        }
    }
}

// MARK: - Page Content
struct PageContentView: View {
    let page: OnboardingScreen.OnboardingPage
    @State private var appeared = false
    
    var body: some View {
        VStack(spacing: 32) {
            Spacer()
            
            // Icon
            ZStack {
                Circle()
                    .fill(page.color.opacity(0.15))
                    .frame(width: 160, height: 160)
                    .scaleEffect(appeared ? 1.0 : 0.5)
                    .opacity(appeared ? 1.0 : 0)
                
                Image(systemName: page.icon)
                    .font(.system(size: 70))
                    .foregroundColor(page.color)
                    .scaleEffect(appeared ? 1.0 : 0.3)
                    .rotationEffect(.degrees(appeared ? 0 : -30))
                    .opacity(appeared ? 1.0 : 0)
            }
            .animation(.spring(response: 0.6, dampingFraction: 0.6).delay(0.1), value: appeared)
            
            // Title
            Text(page.title)
                .font(.largeTitle)
                .fontWeight(.bold)
                .multilineTextAlignment(.center)
                .offset(y: appeared ? 0 : 30)
                .opacity(appeared ? 1.0 : 0)
                .animation(.spring(response: 0.6, dampingFraction: 0.8).delay(0.2), value: appeared)
            
            // Description
            Text(page.description)
                .font(.body)
                .foregroundColor(.secondary)
                .multilineTextAlignment(.center)
                .padding(.horizontal, 32)
                .lineSpacing(4)
                .offset(y: appeared ? 0 : 20)
                .opacity(appeared ? 1.0 : 0)
                .animation(.spring(response: 0.6, dampingFraction: 0.8).delay(0.3), value: appeared)
            
            Spacer()
        }
        .onAppear {
            appeared = true
        }
        .onDisappear {
            appeared = false
        }
    }
}

// MARK: - Bottom Controls
struct BottomControls: View {
    @Binding var currentPage: Int
    let totalPages: Int
    let color: Color
    let onFinish: () -> Void
    
    var isLastPage: Bool { currentPage == totalPages - 1 }
    
    var body: some View {
        VStack(spacing: 24) {
            // Page Indicators
            HStack(spacing: 8) {
                ForEach(0..<totalPages, id: \.self) { i in
                    Capsule()
                        .fill(i == currentPage ? color : color.opacity(0.3))
                        .frame(
                            width: i == currentPage ? 24 : 8,
                            height: 8
                        )
                        .animation(.spring(response: 0.3, dampingFraction: 0.7), value: currentPage)
                }
            }
            
            // Navigation Buttons
            HStack(spacing: 16) {
                // Skip
                if !isLastPage {
                    Button("ข้ามไปก่อน") {
                        withAnimation {
                            currentPage = totalPages - 1
                        }
                    }
                    .foregroundColor(.secondary)
                    .font(.subheadline)
                }
                
                Spacer()
                
                // Next / Finish
                Button {
                    if isLastPage {
                        onFinish()
                    } else {
                        withAnimation(.spring(response: 0.5, dampingFraction: 0.8)) {
                            currentPage += 1
                        }
                    }
                } label: {
                    HStack(spacing: 8) {
                        Text(isLastPage ? "เริ่มต้นใช้งาน" : "ถัดไป")
                            .fontWeight(.semibold)
                        Image(systemName: isLastPage ? "checkmark" : "arrow.right")
                    }
                    .foregroundColor(.white)
                    .padding(.horizontal, 28)
                    .padding(.vertical, 14)
                    .background(color)
                    .cornerRadius(30)
                    .shadow(color: color.opacity(0.4), radius: 10, x: 0, y: 5)
                }
                .scaleEffect(isLastPage ? 1.05 : 1.0)
                .animation(.spring(response: 0.3), value: isLastPage)
            }
            .padding(.horizontal, 32)
        }
    }
}

// MARK: - Main App
struct MainAppView: View {
    var body: some View {
        VStack(spacing: 20) {
            Image(systemName: "checkmark.circle.fill")
                .font(.system(size: 80))
                .foregroundColor(.green)
                .symbolEffect(.bounce)
            
            Text("ยินดีต้อนรับ!")
                .font(.largeTitle)
                .fontWeight(.bold)
            
            Text("คุณพร้อมเริ่มต้นเรียนรู้ Swift แล้ว")
                .font(.subheadline)
                .foregroundColor(.secondary)
        }
    }
}
```

---

## 17. แบบฝึกหัดขั้นสูง: Animated Card Game

```swift
// Card Flip Animation
struct CardFlipGame: View {
    struct Card: Identifiable {
        let id = UUID()
        let emoji: String
        var isFaceUp = false
        var isMatched = false
    }
    
    @State private var cards: [Card]
    @State private var selectedCards: [Card] = []
    @State private var score = 0
    @State private var moves = 0
    @Namespace private var cardNamespace
    
    init() {
        let emojis = ["🐶", "🐱", "🐭", "🐹", "🐰", "🦊", "🐻", "🐼"]
        let doubled = (emojis + emojis).shuffled()
        self._cards = State(initialValue: doubled.map { Card(emoji: $0) })
    }
    
    var body: some View {
        NavigationStack {
            VStack(spacing: 16) {
                // Score
                HStack {
                    Label("คะแนน: \(score)", systemImage: "star.fill")
                        .foregroundColor(.yellow)
                    Spacer()
                    Label("เล่น: \(moves)", systemImage: "arrow.2.circlepath")
                        .foregroundColor(.blue)
                }
                .font(.headline)
                .padding(.horizontal)
                
                // Cards Grid
                LazyVGrid(columns: Array(repeating: GridItem(.flexible()), count: 4), spacing: 10) {
                    ForEach($cards) { $card in
                        CardView(card: card)
                            .onTapGesture {
                                selectCard(&card)
                            }
                    }
                }
                .padding(.horizontal)
                
                // Reset Button
                Button("เริ่มใหม่") {
                    withAnimation(.spring()) {
                        resetGame()
                    }
                }
                .buttonStyle(.borderedProminent)
            }
            .navigationTitle("Card Matching Game")
        }
    }
    
    func selectCard(_ card: inout Card) {
        guard !card.isFaceUp && !card.isMatched else { return }
        guard selectedCards.count < 2 else { return }
        
        withAnimation(.spring(response: 0.4, dampingFraction: 0.7)) {
            card.isFaceUp = true
        }
        
        let selectedCard = card
        selectedCards.append(selectedCard)
        
        if selectedCards.count == 2 {
            moves += 1
            
            if selectedCards[0].emoji == selectedCards[1].emoji {
                // Match!
                score += 10
                DispatchQueue.main.asyncAfter(deadline: .now() + 0.5) {
                    withAnimation(.spring()) {
                        for i in cards.indices {
                            if cards[i].emoji == selectedCards[0].emoji {
                                cards[i].isMatched = true
                            }
                        }
                        selectedCards.removeAll()
                    }
                }
            } else {
                // No match
                DispatchQueue.main.asyncAfter(deadline: .now() + 0.8) {
                    withAnimation(.spring()) {
                        for i in cards.indices {
                            if cards[i].id == selectedCards[0].id ||
                               cards[i].id == selectedCards[1].id {
                                cards[i].isFaceUp = false
                            }
                        }
                        selectedCards.removeAll()
                    }
                }
            }
        }
    }
    
    func resetGame() {
        let emojis = ["🐶", "🐱", "🐭", "🐹", "🐰", "🦊", "🐻", "🐼"]
        let doubled = (emojis + emojis).shuffled()
        cards = doubled.map { Card(emoji: $0) }
        selectedCards = []
        score = 0
        moves = 0
    }
}

struct CardView: View {
    let card: CardFlipGame.Card
    
    var body: some View {
        ZStack {
            if card.isMatched {
                RoundedRectangle(cornerRadius: 10)
                    .fill(Color.green.opacity(0.3))
                    .frame(height: 70)
                    .overlay(
                        Text(card.emoji)
                            .font(.title2)
                    )
            } else if card.isFaceUp {
                RoundedRectangle(cornerRadius: 10)
                    .fill(Color.white)
                    .frame(height: 70)
                    .shadow(radius: 4)
                    .overlay(
                        Text(card.emoji)
                            .font(.title2)
                    )
                    .rotation3DEffect(
                        .degrees(0),
                        axis: (x: 0, y: 1, z: 0)
                    )
            } else {
                RoundedRectangle(cornerRadius: 10)
                    .fill(Color.blue.gradient)
                    .frame(height: 70)
                    .overlay(
                        Image(systemName: "questionmark")
                            .font(.title2)
                            .foregroundColor(.white)
                    )
                    .rotation3DEffect(
                        .degrees(0),
                        axis: (x: 0, y: 1, z: 0)
                    )
            }
        }
        .rotation3DEffect(
            .degrees(card.isFaceUp || card.isMatched ? 0 : 180),
            axis: (x: 0, y: 1, z: 0)
        )
        .animation(.spring(response: 0.4, dampingFraction: 0.7), value: card.isFaceUp)
        .animation(.spring(response: 0.4, dampingFraction: 0.7), value: card.isMatched)
        .opacity(card.isMatched ? 0.6 : 1.0)
    }
}
```

---

## สรุป

ในส่วนนี้เราได้เรียนรู้เกี่ยวกับ **SwiftUI Animations** อย่างครอบคลุม:

### สิ่งที่ได้เรียนรู้

1. **Animation Basics** - หลักการทำงานของ animation ใน SwiftUI
2. **withAnimation** - การ wrap state change ให้ animate พร้อม completion handler
3. **Animation Types** - linear, easeIn, easeOut, easeInOut, spring, bouncy
4. **Implicit Animations** - `.animation` modifier สำหรับ declarative animation
5. **Explicit Animations** - `withAnimation { }` สำหรับ fine-grained control
6. **Transitions** - opacity, scale, slide, move, asymmetric
7. **Custom Transitions** - สร้าง transition เองด้วย ViewModifier
8. **matchedGeometryEffect** - การทำ hero animations ระหว่าง Views
9. **Namespace** - สำหรับ matchedGeometryEffect
10. **Phase Animators** - animate ผ่าน phases อัตโนมัติ (iOS 17+)
11. **Keyframe Animations** - animation แบบ multi-step ขั้นสูง (iOS 17+)
12. **Canvas View** - custom 2D drawing สำหรับ graphics ประสิทธิภาพสูง
13. **TimelineView** - View ที่อัปเดตตาม schedule
14. **Animation Sequences** - การสร้าง sequential และ staggered animations
15. **Symbol Effects** - animate SF Symbols (iOS 17+)

### Tips สำหรับ Animation

- ใช้ `.spring()` สำหรับ interaction-based animation (bounce ดูเป็นธรรมชาติ)
- ใช้ `.easeInOut()` สำหรับ content transition
- ใช้ `.linear()` เฉพาะเมื่อต้องการความสม่ำเสมอ (เช่น progress, rotation)
- หลีกเลี่ยง animation ที่นานเกินไป (> 0.5 วินาที สำหรับ UI interactions)
- ใช้ `matchedGeometryEffect` สำหรับ hero transitions ระหว่าง screens
- `PhaseAnimator` และ `KeyframeAnimator` เหมาะสำหรับ complex multi-step animations
- ทดสอบ animation บน device จริงเสมอ เพราะ Simulator อาจช้ากว่า

---

*ส่วนที่ 26 จบ - ยินดีด้วยที่เรียนรู้ SwiftUI Animations ครบถ้วนแล้ว!*
