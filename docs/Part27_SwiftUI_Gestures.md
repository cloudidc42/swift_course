# Part 27: SwiftUI Gestures

## บทนำ (Introduction)

Gestures เป็นส่วนสำคัญของการโต้ตอบกับแอปพลิเคชัน iOS และ macOS ใน SwiftUI มีระบบจัดการ Gesture ที่ทรงพลังและใช้งานง่าย ในบทนี้เราจะเรียนรู้ Gesture ทุกประเภทที่มีใน SwiftUI พร้อมตัวอย่างโค้ดที่สมบูรณ์และแบบฝึกหัดเชิงปฏิบัติ

---

## 1. TapGesture

`TapGesture` เป็น Gesture พื้นฐานที่สุด ใช้ตรวจจับการแตะหน้าจอ

### การใช้งานพื้นฐาน

```swift
import SwiftUI

struct TapGestureExample: View {
    @State private var tapCount = 0
    @State private var message = "แตะที่นี่"
    
    var body: some View {
        VStack(spacing: 20) {
            Text(message)
                .font(.title)
                .padding()
                .background(Color.blue.opacity(0.2))
                .cornerRadius(10)
                .gesture(
                    TapGesture()
                        .onEnded {
                            tapCount += 1
                            message = "แตะแล้ว \(tapCount) ครั้ง"
                        }
                )
            
            Button("รีเซ็ต") {
                tapCount = 0
                message = "แตะที่นี่"
            }
        }
        .padding()
    }
}
```

### TapGesture กับจำนวนครั้ง (Count)

```swift
struct DoubleTapExample: View {
    @State private var backgroundColor = Color.blue
    
    var body: some View {
        RoundedRectangle(cornerRadius: 20)
            .fill(backgroundColor)
            .frame(width: 200, height: 200)
            .gesture(
                TapGesture(count: 2)
                    .onEnded {
                        withAnimation(.easeInOut(duration: 0.3)) {
                            backgroundColor = backgroundColor == Color.blue ? Color.red : Color.blue
                        }
                    }
            )
            .overlay(
                Text("ดับเบิ้ลแตะเพื่อเปลี่ยนสี")
                    .foregroundColor(.white)
                    .multilineTextAlignment(.center)
            )
    }
}
```

### onTapGesture shorthand

SwiftUI มี modifier `onTapGesture` ที่ใช้งานง่ายกว่า:

```swift
struct OnTapGestureShorthand: View {
    @State private var scale: CGFloat = 1.0
    
    var body: some View {
        Image(systemName: "star.fill")
            .resizable()
            .frame(width: 100, height: 100)
            .foregroundColor(.yellow)
            .scaleEffect(scale)
            // single tap
            .onTapGesture {
                withAnimation(.spring(response: 0.3, dampingFraction: 0.5)) {
                    scale = scale == 1.0 ? 1.5 : 1.0
                }
            }
    }
}

struct DoubleTapShorthand: View {
    @State private var isFavorite = false
    
    var body: some View {
        Image(systemName: isFavorite ? "heart.fill" : "heart")
            .resizable()
            .frame(width: 80, height: 80)
            .foregroundColor(isFavorite ? .red : .gray)
            // double tap
            .onTapGesture(count: 2) {
                withAnimation(.spring()) {
                    isFavorite.toggle()
                }
            }
    }
}
```

---

## 2. LongPressGesture

`LongPressGesture` ตรวจจับการกดค้างไว้บนหน้าจอ

### การใช้งานพื้นฐาน

```swift
struct LongPressExample: View {
    @State private var isPressed = false
    @State private var showMenu = false
    
    var body: some View {
        VStack {
            RoundedRectangle(cornerRadius: 15)
                .fill(isPressed ? Color.orange : Color.blue)
                .frame(width: 200, height: 100)
                .overlay(
                    Text(isPressed ? "กำลังกด..." : "กดค้าง 1 วินาที")
                        .foregroundColor(.white)
                        .font(.headline)
                )
                .gesture(
                    LongPressGesture(minimumDuration: 1.0)
                        .onChanged { isPressing in
                            withAnimation {
                                isPressed = isPressing
                            }
                        }
                        .onEnded { _ in
                            withAnimation {
                                isPressed = false
                                showMenu = true
                            }
                        }
                )
            
            if showMenu {
                VStack {
                    Text("เมนูที่ปรากฏจากการกดค้าง")
                        .font(.headline)
                    HStack {
                        Button("แก้ไข") { showMenu = false }
                        Button("ลบ") { showMenu = false }
                        Button("แชร์") { showMenu = false }
                    }
                    .padding()
                }
                .transition(.move(edge: .bottom).combined(with: .opacity))
                .padding()
                .background(Color.gray.opacity(0.1))
                .cornerRadius(10)
            }
        }
        .padding()
        .animation(.easeInOut, value: showMenu)
    }
}
```

### LongPressGesture กับ maximumDistance

```swift
struct LongPressPrecise: View {
    @GestureState private var isDetectingLongPress = false
    @State private var completedLongPress = false
    
    var longPress: some Gesture {
        LongPressGesture(minimumDuration: 1.5, maximumDistance: 10)
            .updating($isDetectingLongPress) { currentState, gestureState, transaction in
                gestureState = currentState
                transaction.animation = Animation.easeIn(duration: 0.5)
            }
            .onEnded { finished in
                completedLongPress = finished
            }
    }
    
    var body: some View {
        Circle()
            .fill(isDetectingLongPress ? Color.red : Color.green)
            .frame(width: isDetectingLongPress ? 150 : 100,
                   height: isDetectingLongPress ? 150 : 100)
            .gesture(longPress)
            .overlay(
                Text(completedLongPress ? "เสร็จแล้ว!" : (isDetectingLongPress ? "กด..." : "กดค้าง"))
                    .foregroundColor(.white)
            )
            .animation(.spring(), value: isDetectingLongPress)
    }
}
```

---

## 3. DragGesture

`DragGesture` ตรวจจับการลากนิ้วบนหน้าจอ

### การใช้งานพื้นฐาน

```swift
struct DragGestureBasic: View {
    @State private var offset = CGSize.zero
    @State private var isDragging = false
    
    var body: some View {
        RoundedRectangle(cornerRadius: 20)
            .fill(isDragging ? Color.red : Color.blue)
            .frame(width: 100, height: 100)
            .offset(offset)
            .gesture(
                DragGesture()
                    .onChanged { value in
                        offset = value.translation
                        isDragging = true
                    }
                    .onEnded { value in
                        withAnimation(.spring()) {
                            offset = .zero
                            isDragging = false
                        }
                    }
            )
            .animation(.spring(), value: isDragging)
    }
}
```

### DragGesture พร้อม Velocity

```swift
struct DragWithVelocity: View {
    @State private var position = CGPoint(x: 200, y: 400)
    @State private var dragStartPosition = CGPoint.zero
    @GestureState private var dragOffset = CGSize.zero
    
    var body: some View {
        ZStack {
            Color.gray.opacity(0.1)
                .ignoresSafeArea()
            
            Circle()
                .fill(Color.purple)
                .frame(width: 80, height: 80)
                .position(
                    x: position.x + dragOffset.width,
                    y: position.y + dragOffset.height
                )
                .gesture(
                    DragGesture(minimumDistance: 0, coordinateSpace: .global)
                        .updating($dragOffset) { value, state, _ in
                            state = value.translation
                        }
                        .onEnded { value in
                            position.x += value.translation.width
                            position.y += value.translation.height
                        }
                )
            
            VStack {
                Spacer()
                Text("ลากลูกบอลไปไว้ที่ใดก็ได้")
                    .padding()
            }
        }
    }
}
```

### Drag กับ Snap to Grid

```swift
struct SnapToGridDrag: View {
    @State private var position = CGPoint(x: 100, y: 100)
    let gridSize: CGFloat = 50
    
    var body: some View {
        ZStack {
            // วาด Grid
            Canvas { context, size in
                for x in stride(from: 0, to: size.width, by: gridSize) {
                    context.stroke(
                        Path { path in
                            path.move(to: CGPoint(x: x, y: 0))
                            path.addLine(to: CGPoint(x: x, y: size.height))
                        },
                        with: .color(.gray.opacity(0.3)),
                        lineWidth: 0.5
                    )
                }
                for y in stride(from: 0, to: size.height, by: gridSize) {
                    context.stroke(
                        Path { path in
                            path.move(to: CGPoint(x: 0, y: y))
                            path.addLine(to: CGPoint(x: size.width, y: y))
                        },
                        with: .color(.gray.opacity(0.3)),
                        lineWidth: 0.5
                    )
                }
            }
            
            RoundedRectangle(cornerRadius: 8)
                .fill(Color.blue)
                .frame(width: 40, height: 40)
                .position(position)
                .gesture(
                    DragGesture()
                        .onEnded { value in
                            withAnimation(.spring()) {
                                let newX = (value.location.x / gridSize).rounded() * gridSize
                                let newY = (value.location.y / gridSize).rounded() * gridSize
                                position = CGPoint(x: newX, y: newY)
                            }
                        }
                )
        }
    }
}
```

---

## 4. MagnifyGesture (Pinch)

`MagnifyGesture` ตรวจจับการบีบ/ขยายนิ้ว (pinch gesture) สำหรับการย่อ/ขยาย

```swift
struct MagnifyGestureExample: View {
    @State private var currentScale: CGFloat = 1.0
    @GestureState private var gestureScale: CGFloat = 1.0
    
    var body: some View {
        VStack {
            Image(systemName: "photo")
                .resizable()
                .scaledToFit()
                .frame(width: 200, height: 200)
                .scaleEffect(currentScale * gestureScale)
                .gesture(
                    MagnifyGesture()
                        .updating($gestureScale) { value, state, _ in
                            state = value.magnification
                        }
                        .onEnded { value in
                            currentScale *= value.magnification
                            // จำกัดขนาดไม่ให้เล็กหรือใหญ่เกินไป
                            currentScale = max(0.5, min(currentScale, 4.0))
                        }
                )
            
            Text("Scale: \(String(format: "%.2f", currentScale * gestureScale))x")
                .padding()
            
            Button("รีเซ็ต") {
                withAnimation(.spring()) {
                    currentScale = 1.0
                }
            }
        }
        .padding()
    }
}
```

### Pinch-to-Zoom สำหรับรูปภาพ

```swift
struct ImageZoomView: View {
    @State private var scale: CGFloat = 1.0
    @State private var lastScale: CGFloat = 1.0
    @State private var offset: CGSize = .zero
    @State private var lastOffset: CGSize = .zero
    
    var magnifyGesture: some Gesture {
        MagnifyGesture()
            .onChanged { value in
                let delta = value.magnification / lastScale
                lastScale = value.magnification
                scale *= delta
                scale = max(1.0, min(scale, 5.0))
            }
            .onEnded { _ in
                lastScale = 1.0
                if scale < 1.0 {
                    withAnimation(.spring()) {
                        scale = 1.0
                        offset = .zero
                    }
                }
            }
    }
    
    var dragGesture: some Gesture {
        DragGesture()
            .onChanged { value in
                offset = CGSize(
                    width: lastOffset.width + value.translation.width,
                    height: lastOffset.height + value.translation.height
                )
            }
            .onEnded { _ in
                lastOffset = offset
            }
    }
    
    var body: some View {
        VStack {
            Text("ขยาย/ย่อด้วย Pinch หรือลากเพื่อเลื่อน")
                .font(.caption)
                .padding()
            
            Image(systemName: "map.fill")
                .resizable()
                .scaledToFit()
                .frame(width: 300, height: 300)
                .background(Color.blue.opacity(0.1))
                .scaleEffect(scale)
                .offset(offset)
                .gesture(magnifyGesture.simultaneously(with: dragGesture))
                .clipped()
            
            Button("รีเซ็ต") {
                withAnimation(.spring()) {
                    scale = 1.0
                    offset = .zero
                    lastOffset = .zero
                }
            }
            .padding()
        }
    }
}
```

---

## 5. RotateGesture

`RotateGesture` ตรวจจับการหมุนด้วยสองนิ้ว

```swift
struct RotateGestureExample: View {
    @State private var angle: Angle = .zero
    @GestureState private var gestureAngle: Angle = .zero
    
    var body: some View {
        VStack {
            RoundedRectangle(cornerRadius: 20)
                .fill(
                    LinearGradient(
                        colors: [.blue, .purple],
                        startPoint: .topLeading,
                        endPoint: .bottomTrailing
                    )
                )
                .frame(width: 200, height: 150)
                .rotationEffect(angle + gestureAngle)
                .gesture(
                    RotateGesture()
                        .updating($gestureAngle) { value, state, _ in
                            state = value.rotation
                        }
                        .onEnded { value in
                            angle += value.rotation
                        }
                )
                .overlay(
                    Text("หมุนด้วยสองนิ้ว")
                        .foregroundColor(.white)
                        .font(.headline)
                )
            
            Text("มุม: \(String(format: "%.1f", angle.degrees + gestureAngle.degrees))°")
                .padding()
            
            Button("รีเซ็ต") {
                withAnimation(.spring()) {
                    angle = .zero
                }
            }
        }
        .padding()
    }
}
```

### Rotate กับ Scale พร้อมกัน

```swift
struct RotateAndScaleView: View {
    @State private var angle: Angle = .zero
    @State private var scale: CGFloat = 1.0
    @GestureState private var gestureState: (angle: Angle, scale: CGFloat) = (.zero, 1.0)
    
    var combinedGesture: some Gesture {
        RotateGesture()
            .simultaneously(with: MagnifyGesture())
            .updating($gestureState) { value, state, _ in
                state.angle = value.first?.rotation ?? .zero
                state.scale = value.second?.magnification ?? 1.0
            }
            .onEnded { value in
                angle += value.first?.rotation ?? .zero
                scale *= value.second?.magnification ?? 1.0
                scale = max(0.5, min(scale, 4.0))
            }
    }
    
    var body: some View {
        Image(systemName: "photo.artframe")
            .resizable()
            .scaledToFit()
            .frame(width: 200, height: 200)
            .foregroundColor(.blue)
            .scaleEffect(scale * gestureState.scale)
            .rotationEffect(angle + gestureState.angle)
            .gesture(combinedGesture)
    }
}
```

---

## 6. Composing Gestures (การรวม Gesture)

SwiftUI รองรับการรวม Gesture หลายรูปแบบ:

### simultaneously (พร้อมกัน)

```swift
struct SimultaneousGestureExample: View {
    @State private var dragOffset: CGSize = .zero
    @State private var scale: CGFloat = 1.0
    @GestureState private var gestureDragOffset: CGSize = .zero
    @GestureState private var gestureScale: CGFloat = 1.0
    
    var body: some View {
        Circle()
            .fill(Color.orange)
            .frame(width: 100, height: 100)
            .scaleEffect(scale * gestureScale)
            .offset(
                x: dragOffset.width + gestureDragOffset.width,
                y: dragOffset.height + gestureDragOffset.height
            )
            .gesture(
                DragGesture()
                    .updating($gestureDragOffset) { value, state, _ in
                        state = value.translation
                    }
                    .onEnded { value in
                        dragOffset.width += value.translation.width
                        dragOffset.height += value.translation.height
                    }
                    .simultaneously(with:
                        MagnifyGesture()
                            .updating($gestureScale) { value, state, _ in
                                state = value.magnification
                            }
                            .onEnded { value in
                                scale *= value.magnification
                            }
                    )
            )
    }
}
```

### sequentially (ตามลำดับ)

```swift
struct SequentialGestureExample: View {
    @State private var isLongPressed = false
    @State private var dragOffset: CGSize = .zero
    @GestureState private var offset: CGSize = .zero
    
    // ต้องกดค้างก่อนถึงจะลากได้
    var sequentialGesture: some Gesture {
        LongPressGesture(minimumDuration: 0.5)
            .onEnded { _ in
                isLongPressed = true
            }
            .sequenced(before:
                DragGesture()
                    .updating($offset) { value, state, _ in
                        state = value.translation
                    }
                    .onEnded { value in
                        dragOffset.width += value.translation.width
                        dragOffset.height += value.translation.height
                        isLongPressed = false
                    }
            )
    }
    
    var body: some View {
        RoundedRectangle(cornerRadius: 15)
            .fill(isLongPressed ? Color.green : Color.blue)
            .frame(width: 100, height: 100)
            .offset(x: dragOffset.width + offset.width,
                    y: dragOffset.height + offset.height)
            .gesture(sequentialGesture)
            .overlay(
                Text(isLongPressed ? "ลากได้แล้ว" : "กดค้าง")
                    .foregroundColor(.white)
                    .font(.caption)
            )
            .animation(.easeInOut, value: isLongPressed)
    }
}
```

### exclusively (เฉพาะอย่างเดียว)

```swift
struct ExclusiveGestureExample: View {
    @State private var lastGesture = "ยังไม่มี"
    
    var exclusiveGesture: some Gesture {
        TapGesture()
            .onEnded {
                lastGesture = "แตะครั้งเดียว"
            }
            .exclusively(before:
                TapGesture(count: 2)
                    .onEnded {
                        lastGesture = "แตะสองครั้ง"
                    }
            )
    }
    
    var body: some View {
        VStack {
            RoundedRectangle(cornerRadius: 15)
                .fill(Color.purple)
                .frame(width: 200, height: 100)
                .gesture(exclusiveGesture)
                .overlay(
                    Text("แตะ 1 หรือ 2 ครั้ง")
                        .foregroundColor(.white)
                )
            
            Text("Gesture ล่าสุด: \(lastGesture)")
                .padding()
        }
    }
}
```

---

## 7. GestureState และ @GestureState

`@GestureState` เป็น property wrapper พิเศษที่รีเซ็ตค่าโดยอัตโนมัติเมื่อ gesture สิ้นสุด

### ความแตกต่างระหว่าง @State และ @GestureState

```swift
struct GestureStateComparison: View {
    // @State ไม่รีเซ็ตเองเมื่อ gesture สิ้นสุด
    @State private var stateOffset: CGSize = .zero
    
    // @GestureState รีเซ็ตเป็น .zero โดยอัตโนมัติเมื่อ gesture สิ้นสุด
    @GestureState private var gestureOffset: CGSize = .zero
    
    var body: some View {
        VStack(spacing: 40) {
            // ใช้ @State - จะอยู่ที่ตำแหน่งสุดท้ายที่ลาก
            RoundedRectangle(cornerRadius: 10)
                .fill(Color.blue)
                .frame(width: 80, height: 80)
                .offset(stateOffset)
                .gesture(
                    DragGesture()
                        .onChanged { value in
                            stateOffset = value.translation
                        }
                )
                .overlay(Text("@State").foregroundColor(.white).font(.caption))
            
            // ใช้ @GestureState - จะกลับมาที่ตำแหน่งเดิมเสมอ
            RoundedRectangle(cornerRadius: 10)
                .fill(Color.red)
                .frame(width: 80, height: 80)
                .offset(gestureOffset)
                .gesture(
                    DragGesture()
                        .updating($gestureOffset) { value, state, _ in
                            state = value.translation
                        }
                )
                .overlay(Text("@GestureState").foregroundColor(.white).font(.caption))
        }
    }
}
```

### การใช้ @GestureState กับ Struct ซับซ้อน

```swift
struct ComplexGestureState: View {
    struct TransformState {
        var scale: CGFloat = 1.0
        var angle: Angle = .zero
        var offset: CGSize = .zero
    }
    
    @GestureState private var transform = TransformState()
    @State private var finalTransform = TransformState()
    
    var complexGesture: some Gesture {
        MagnifyGesture()
            .simultaneously(with: RotateGesture())
            .simultaneously(with: DragGesture())
            .updating($transform) { value, state, _ in
                state.scale = value.first?.first?.magnification ?? 1.0
                state.angle = value.first?.second?.rotation ?? .zero
                state.offset = value.second?.translation ?? .zero
            }
            .onEnded { value in
                finalTransform.scale *= value.first?.first?.magnification ?? 1.0
                finalTransform.angle += value.first?.second?.rotation ?? .zero
                finalTransform.offset.width += value.second?.translation.width ?? 0
                finalTransform.offset.height += value.second?.translation.height ?? 0
            }
    }
    
    var body: some View {
        Image(systemName: "rectangle.stack.fill")
            .resizable()
            .scaledToFit()
            .frame(width: 150, height: 150)
            .foregroundColor(.blue)
            .scaleEffect(finalTransform.scale * transform.scale)
            .rotationEffect(finalTransform.angle + transform.angle)
            .offset(
                x: finalTransform.offset.width + transform.offset.width,
                y: finalTransform.offset.height + transform.offset.height
            )
            .gesture(complexGesture)
    }
}
```

---

## 8. Gesture Callbacks: onEnded, onChanged, updating

### onChanged

เรียกเมื่อ gesture เปลี่ยนแปลงระหว่าง interaction

```swift
struct OnChangedExample: View {
    @State private var progress: CGFloat = 0.0
    @State private var startX: CGFloat = 0
    
    var body: some View {
        VStack(spacing: 20) {
            ProgressView(value: progress)
                .padding()
            
            Text("ความคืบหน้า: \(Int(progress * 100))%")
            
            RoundedRectangle(cornerRadius: 10)
                .fill(Color.blue)
                .frame(width: 300, height: 50)
                .gesture(
                    DragGesture(minimumDistance: 0)
                        .onChanged { value in
                            // คำนวณ progress จากตำแหน่ง X
                            let width: CGFloat = 300
                            let newProgress = (value.location.x / width).clamped(to: 0...1)
                            progress = newProgress
                        }
                )
                .overlay(
                    Text("ลากเพื่อเพิ่ม progress")
                        .foregroundColor(.white)
                )
        }
        .padding()
    }
}

extension Comparable {
    func clamped(to limits: ClosedRange<Self>) -> Self {
        min(max(self, limits.lowerBound), limits.upperBound)
    }
}
```

### onEnded

เรียกเมื่อ gesture สิ้นสุด

```swift
struct OnEndedExample: View {
    @State private var cards: [CardModel] = [
        CardModel(id: 1, title: "การ์ด 1", color: .blue),
        CardModel(id: 2, title: "การ์ด 2", color: .red),
        CardModel(id: 3, title: "การ์ด 3", color: .green)
    ]
    @State private var draggedCard: CardModel?
    
    var body: some View {
        VStack {
            ForEach(cards) { card in
                CardView(card: card)
                    .gesture(
                        DragGesture()
                            .onChanged { _ in
                                draggedCard = card
                            }
                            .onEnded { value in
                                handleDrop(card: card, location: value.location)
                                draggedCard = nil
                            }
                    )
            }
        }
    }
    
    func handleDrop(card: CardModel, location: CGPoint) {
        // จัดการเมื่อปล่อยการ์ด
        print("ปล่อยการ์ด \(card.title) ที่ตำแหน่ง \(location)")
    }
}

struct CardModel: Identifiable {
    let id: Int
    let title: String
    let color: Color
}

struct CardView: View {
    let card: CardModel
    
    var body: some View {
        RoundedRectangle(cornerRadius: 12)
            .fill(card.color)
            .frame(height: 80)
            .overlay(
                Text(card.title)
                    .foregroundColor(.white)
                    .font(.headline)
            )
            .padding(.horizontal)
    }
}
```

### updating

เรียกตลอดเวลาระหว่าง gesture พร้อม transaction สำหรับ animation

```swift
struct UpdatingExample: View {
    @GestureState private var dragInfo = DragInfo()
    
    struct DragInfo {
        var offset: CGSize = .zero
        var isDragging: Bool = false
    }
    
    var body: some View {
        RoundedRectangle(cornerRadius: 15)
            .fill(dragInfo.isDragging ? Color.red : Color.blue)
            .frame(width: 100, height: 100)
            .offset(dragInfo.offset)
            .shadow(radius: dragInfo.isDragging ? 20 : 5)
            .gesture(
                DragGesture()
                    .updating($dragInfo) { value, state, transaction in
                        state.offset = value.translation
                        state.isDragging = true
                        // ตั้งค่า animation สำหรับ state นี้
                        transaction.animation = .interactiveSpring()
                    }
            )
            .animation(.spring(), value: dragInfo.isDragging)
    }
}
```

---

## 9. Gesture กับ Animations

### Spring Animation

```swift
struct SpringAnimationGesture: View {
    @State private var isExpanded = false
    
    var body: some View {
        VStack {
            RoundedRectangle(cornerRadius: isExpanded ? 30 : 15)
                .fill(isExpanded ? Color.purple : Color.blue)
                .frame(
                    width: isExpanded ? 300 : 100,
                    height: isExpanded ? 200 : 100
                )
                .overlay(
                    VStack {
                        Text(isExpanded ? "คลิกเพื่อย่อ" : "คลิกเพื่อขยาย")
                            .foregroundColor(.white)
                    }
                )
                .onTapGesture {
                    withAnimation(.spring(response: 0.5, dampingFraction: 0.6)) {
                        isExpanded.toggle()
                    }
                }
        }
    }
}
```

### Bounce Effect

```swift
struct BounceGestureEffect: View {
    @State private var scale: CGFloat = 1.0
    
    var body: some View {
        Image(systemName: "heart.fill")
            .resizable()
            .frame(width: 80, height: 80)
            .foregroundColor(.red)
            .scaleEffect(scale)
            .onTapGesture {
                withAnimation(.interpolatingSpring(stiffness: 300, damping: 10)) {
                    scale = 1.5
                }
                DispatchQueue.main.asyncAfter(deadline: .now() + 0.1) {
                    withAnimation(.interpolatingSpring(stiffness: 300, damping: 10)) {
                        scale = 1.0
                    }
                }
            }
    }
}
```

### Interactive Animation

```swift
struct InteractiveAnimationGesture: View {
    @State private var position: CGPoint = CGPoint(x: 200, y: 400)
    @State private var velocity: CGPoint = .zero
    @GestureState private var startPosition: CGPoint? = nil
    
    var body: some View {
        Circle()
            .fill(
                RadialGradient(
                    gradient: Gradient(colors: [.yellow, .orange]),
                    center: .center,
                    startRadius: 5,
                    endRadius: 40
                )
            )
            .frame(width: 80, height: 80)
            .position(position)
            .gesture(
                DragGesture(minimumDistance: 0, coordinateSpace: .local)
                    .updating($startPosition) { value, state, _ in
                        if state == nil {
                            state = position
                        }
                    }
                    .onChanged { value in
                        position = CGPoint(
                            x: (startPosition?.x ?? position.x) + value.translation.width,
                            y: (startPosition?.y ?? position.y) + value.translation.height
                        )
                    }
                    .onEnded { value in
                        withAnimation(.interpolatingSpring(
                            stiffness: 100,
                            damping: 10,
                            initialVelocity: value.velocity.width / 100
                        )) {
                            position.x += value.predictedEndTranslation.width * 0.3
                            position.y += value.predictedEndTranslation.height * 0.3
                        }
                    }
            )
    }
}
```

---

## 10. Custom Gestures

### การสร้าง Custom Gesture

```swift
struct ShakeGesture: Gesture {
    var minimumShakes: Int = 3
    var onShake: () -> Void
    
    var body: some Gesture {
        DragGesture(minimumDistance: 20)
            .onEnded { value in
                let translation = value.translation
                let isHorizontalShake = abs(translation.width) > abs(translation.height) * 2
                if isHorizontalShake {
                    onShake()
                }
            }
    }
}

// การใช้งาน Custom Gesture
struct CustomGestureView: View {
    @State private var shakeCount = 0
    @State private var rotation: Double = 0
    
    var body: some View {
        VStack {
            Image(systemName: "cube.fill")
                .resizable()
                .frame(width: 100, height: 100)
                .foregroundColor(.blue)
                .rotationEffect(.degrees(rotation))
                .gesture(
                    ShakeGesture {
                        shakeCount += 1
                        withAnimation(.spring()) {
                            rotation += 30
                        }
                    }
                )
            
            Text("เขย่าแล้ว \(shakeCount) ครั้ง")
                .padding()
        }
    }
}
```

### Custom Multi-Touch Gesture

```swift
struct CustomTwoFingerTap: View {
    @State private var twoFingerTapped = false
    
    var body: some View {
        ZStack {
            Color.blue.opacity(0.1)
            
            Text(twoFingerTapped ? "แตะสองนิ้ว!" : "แตะด้วยสองนิ้ว")
                .font(.title)
        }
        .frame(width: 300, height: 300)
        .cornerRadius(20)
        .gesture(
            TapGesture(count: 1)
                .simultaneously(with: TapGesture(count: 1))
                .onEnded { _ in
                    withAnimation(.spring()) {
                        twoFingerTapped = true
                    }
                    DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
                        withAnimation {
                            twoFingerTapped = false
                        }
                    }
                }
        )
    }
}
```

---

## 11. Gesture Conflicts (การจัดการความขัดแย้ง)

### highPriorityGesture

```swift
struct GestureConflictExample: View {
    @State private var parentTapped = false
    @State private var childTapped = false
    
    var body: some View {
        VStack {
            // Parent ใช้ highPriorityGesture ทำให้มีสิทธิ์สูงกว่า
            RoundedRectangle(cornerRadius: 20)
                .fill(parentTapped ? Color.red.opacity(0.3) : Color.blue.opacity(0.1))
                .frame(width: 300, height: 200)
                .highPriorityGesture(
                    TapGesture()
                        .onEnded {
                            parentTapped = true
                            childTapped = false
                        }
                )
                .overlay(
                    RoundedRectangle(cornerRadius: 10)
                        .fill(childTapped ? Color.green : Color.orange)
                        .frame(width: 100, height: 60)
                        .gesture(
                            TapGesture()
                                .onEnded {
                                    childTapped = true
                                    parentTapped = false
                                }
                        )
                        .overlay(Text("ลูก").foregroundColor(.white))
                )
            
            Text(parentTapped ? "แตะ Parent" : (childTapped ? "แตะ Child" : "แตะที่ใดก็ได้"))
                .padding()
        }
    }
}
```

### simultaneousGesture modifier

```swift
struct SimultaneousGestureModifier: View {
    @State private var scrollOffset: CGFloat = 0
    @State private var likedItems: Set<Int> = []
    
    var body: some View {
        ScrollView {
            LazyVStack {
                ForEach(1...20, id: \.self) { index in
                    HStack {
                        Text("รายการที่ \(index)")
                            .frame(maxWidth: .infinity, alignment: .leading)
                        
                        Image(systemName: likedItems.contains(index) ? "heart.fill" : "heart")
                            .foregroundColor(likedItems.contains(index) ? .red : .gray)
                            // ใช้ simultaneousGesture เพื่อไม่รบกวน ScrollView
                            .simultaneousGesture(
                                TapGesture()
                                    .onEnded {
                                        if likedItems.contains(index) {
                                            likedItems.remove(index)
                                        } else {
                                            likedItems.insert(index)
                                        }
                                    }
                            )
                            .padding()
                    }
                    .padding()
                    .background(Color.gray.opacity(0.05))
                    .cornerRadius(8)
                    .padding(.horizontal)
                }
            }
        }
    }
}
```

---

## 12. Accessibility กับ Gestures

### AccessibilityAction

```swift
struct AccessibleGestureView: View {
    @State private var count = 0
    @State private var isExpanded = false
    
    var body: some View {
        VStack {
            RoundedRectangle(cornerRadius: 15)
                .fill(Color.blue)
                .frame(width: 200, height: 100)
                .overlay(
                    Text("นับ: \(count)")
                        .foregroundColor(.white)
                        .font(.title2)
                )
                .onTapGesture {
                    count += 1
                }
                // เพิ่ม Accessibility
                .accessibilityLabel("ปุ่มนับ")
                .accessibilityValue("\(count) ครั้ง")
                .accessibilityHint("แตะเพื่อเพิ่มจำนวน")
                .accessibilityAddTraits(.isButton)
                // เพิ่ม custom action สำหรับ VoiceOver
                .accessibilityAction(named: "รีเซ็ต") {
                    count = 0
                }
            
            // ตัวอย่าง Expandable content
            VStack {
                HStack {
                    Text("เนื้อหาขยายได้")
                        .font(.headline)
                    Spacer()
                    Image(systemName: isExpanded ? "chevron.up" : "chevron.down")
                }
                .contentShape(Rectangle())
                .onTapGesture {
                    withAnimation {
                        isExpanded.toggle()
                    }
                }
                .accessibilityLabel(isExpanded ? "ย่อเนื้อหา" : "ขยายเนื้อหา")
                .accessibilityAddTraits(.isButton)
                
                if isExpanded {
                    Text("นี่คือเนื้อหาที่ซ่อนอยู่ภายใน")
                        .padding(.top)
                }
            }
            .padding()
            .background(Color.gray.opacity(0.1))
            .cornerRadius(10)
            .padding()
        }
    }
}
```

### Gesture กับ accessibilityActivationPoint

```swift
struct AccessibilityGestureTarget: View {
    @State private var isActive = false
    
    var body: some View {
        ZStack {
            // พื้นที่ gesture ใหญ่
            Color.clear
                .frame(width: 200, height: 200)
                .contentShape(Rectangle())
                .onTapGesture {
                    isActive.toggle()
                }
            
            // Visual ขนาดเล็ก
            Circle()
                .fill(isActive ? Color.green : Color.red)
                .frame(width: 50, height: 50)
        }
        .accessibilityElement()
        .accessibilityLabel("สวิตช์")
        .accessibilityValue(isActive ? "เปิด" : "ปิด")
        .accessibilityAddTraits(.isButton)
        // กำหนดจุดที่ VoiceOver จะกด (ตรงกลางพื้นที่ขนาดใหญ่)
        .accessibilityActivationPoint(CGPoint(x: 100, y: 100))
    }
}
```

---

## 13. แบบฝึกหัดเชิงปฏิบัติ

### แบบฝึกหัดที่ 1: Draggable Cards

```swift
import SwiftUI

struct DraggableCard: Identifiable {
    let id = UUID()
    var title: String
    var subtitle: String
    var color: Color
    var offset: CGSize = .zero
    var rotation: Double = 0
    var isRemoved: Bool = false
}

struct DraggableCardsGame: View {
    @State private var cards: [DraggableCard] = [
        DraggableCard(title: "Swift", subtitle: "ภาษาโปรแกรมมิ่ง", color: .orange),
        DraggableCard(title: "SwiftUI", subtitle: "UI Framework", color: .blue),
        DraggableCard(title: "Xcode", subtitle: "IDE", color: .purple),
        DraggableCard(title: "iOS", subtitle: "ระบบปฏิบัติการ", color: .green),
    ]
    @State private var dragging = false
    
    var activeCards: [DraggableCard] {
        cards.filter { !$0.isRemoved }
    }
    
    var body: some View {
        ZStack {
            Color.gray.opacity(0.1).ignoresSafeArea()
            
            VStack {
                Text("ปัดการ์ดไปซ้ายหรือขวา")
                    .font(.headline)
                    .padding()
                
                ZStack {
                    ForEach(Array(activeCards.enumerated()), id: \.element.id) { index, card in
                        CardDragView(
                            card: card,
                            isTop: index == activeCards.count - 1
                        ) { direction in
                            removeCard(id: card.id, direction: direction)
                        }
                        .offset(y: CGFloat(activeCards.count - 1 - index) * 5)
                        .scaleEffect(1.0 - CGFloat(activeCards.count - 1 - index) * 0.03)
                        .zIndex(Double(index))
                    }
                }
                .frame(width: 300, height: 400)
                
                if activeCards.isEmpty {
                    Text("หมดการ์ดแล้ว!")
                        .font(.title)
                        .padding()
                    
                    Button("เริ่มใหม่") {
                        resetCards()
                    }
                    .buttonStyle(.borderedProminent)
                }
            }
        }
    }
    
    func removeCard(id: UUID, direction: SwipeDirection) {
        if let index = cards.firstIndex(where: { $0.id == id }) {
            withAnimation(.easeOut(duration: 0.3)) {
                cards[index].isRemoved = true
            }
        }
    }
    
    func resetCards() {
        withAnimation {
            for index in cards.indices {
                cards[index].isRemoved = false
                cards[index].offset = .zero
                cards[index].rotation = 0
            }
        }
    }
}

enum SwipeDirection {
    case left, right
}

struct CardDragView: View {
    var card: DraggableCard
    var isTop: Bool
    var onSwipe: (SwipeDirection) -> Void
    
    @State private var offset: CGSize = .zero
    
    let swipeThreshold: CGFloat = 100
    
    var body: some View {
        RoundedRectangle(cornerRadius: 20)
            .fill(card.color)
            .frame(width: 280, height: 380)
            .overlay(
                VStack {
                    Text(card.title)
                        .font(.largeTitle)
                        .bold()
                        .foregroundColor(.white)
                    Text(card.subtitle)
                        .font(.title3)
                        .foregroundColor(.white.opacity(0.8))
                }
            )
            .overlay(
                // แสดง indicator ขณะลาก
                Group {
                    if isTop && offset.width > 30 {
                        Image(systemName: "checkmark.circle.fill")
                            .resizable()
                            .frame(width: 60, height: 60)
                            .foregroundColor(.green)
                            .opacity(Double(offset.width / swipeThreshold).clamped(to: 0...1))
                            .frame(maxWidth: .infinity, alignment: .leading)
                            .padding()
                    }
                    if isTop && offset.width < -30 {
                        Image(systemName: "xmark.circle.fill")
                            .resizable()
                            .frame(width: 60, height: 60)
                            .foregroundColor(.red)
                            .opacity(Double(abs(offset.width) / swipeThreshold).clamped(to: 0...1))
                            .frame(maxWidth: .infinity, alignment: .trailing)
                            .padding()
                    }
                }
            )
            .rotationEffect(.degrees(Double(offset.width / 20)))
            .offset(offset)
            .gesture(
                isTop ? DragGesture()
                    .onChanged { value in
                        offset = value.translation
                    }
                    .onEnded { value in
                        if abs(value.translation.width) > swipeThreshold {
                            let direction: SwipeDirection = value.translation.width > 0 ? .right : .left
                            withAnimation(.easeOut(duration: 0.3)) {
                                offset = CGSize(
                                    width: direction == .right ? 500 : -500,
                                    height: value.translation.height
                                )
                            }
                            onSwipe(direction)
                        } else {
                            withAnimation(.spring()) {
                                offset = .zero
                            }
                        }
                    } : nil
            )
    }
}
```

### แบบฝึกหัดที่ 2: Pinch-to-Zoom Image Viewer

```swift
struct ImageZoomViewer: View {
    let imageName: String
    
    @State private var scale: CGFloat = 1.0
    @State private var lastScale: CGFloat = 1.0
    @State private var offset: CGSize = .zero
    @State private var lastOffset: CGSize = .zero
    
    var body: some View {
        NavigationView {
            ZStack {
                Color.black.ignoresSafeArea()
                
                Image(systemName: imageName)
                    .resizable()
                    .scaledToFit()
                    .foregroundColor(.white)
                    .scaleEffect(scale)
                    .offset(offset)
                    .gesture(
                        MagnifyGesture()
                            .onChanged { value in
                                let delta = value.magnification / lastScale
                                lastScale = value.magnification
                                scale = min(max(scale * delta, 1.0), 5.0)
                            }
                            .onEnded { _ in
                                lastScale = 1.0
                                if scale < 1.0 {
                                    withAnimation(.spring()) {
                                        scale = 1.0
                                        offset = .zero
                                    }
                                }
                            }
                            .simultaneously(with:
                                DragGesture()
                                    .onChanged { value in
                                        if scale > 1.0 {
                                            offset = CGSize(
                                                width: lastOffset.width + value.translation.width,
                                                height: lastOffset.height + value.translation.height
                                            )
                                        }
                                    }
                                    .onEnded { _ in
                                        lastOffset = offset
                                    }
                            )
                    )
                    .onTapGesture(count: 2) {
                        withAnimation(.spring()) {
                            if scale > 1.0 {
                                scale = 1.0
                                offset = .zero
                                lastOffset = .zero
                            } else {
                                scale = 2.5
                            }
                        }
                    }
            }
            .navigationTitle("Pinch to Zoom")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button("รีเซ็ต") {
                        withAnimation(.spring()) {
                            scale = 1.0
                            offset = .zero
                            lastOffset = .zero
                        }
                    }
                    .foregroundColor(.white)
                }
            }
        }
    }
}
```

### แบบฝึกหัดที่ 3: Gesture-Based Game

```swift
struct GestureGame: View {
    @State private var score = 0
    @State private var timeRemaining = 30
    @State private var isPlaying = false
    @State private var targets: [GameTarget] = []
    @State private var timer: Timer?
    @State private var spawnTimer: Timer?
    
    var body: some View {
        ZStack {
            Color.black.ignoresSafeArea()
            
            if !isPlaying {
                VStack(spacing: 20) {
                    Text("Gesture Game")
                        .font(.largeTitle)
                        .bold()
                        .foregroundColor(.white)
                    
                    Text("คะแนน: \(score)")
                        .font(.title)
                        .foregroundColor(.yellow)
                    
                    Button("เริ่มเกม") {
                        startGame()
                    }
                    .font(.title2)
                    .buttonStyle(.borderedProminent)
                }
            } else {
                // Game UI
                VStack {
                    HStack {
                        Text("คะแนน: \(score)")
                            .font(.headline)
                            .foregroundColor(.white)
                        Spacer()
                        Text("เวลา: \(timeRemaining)s")
                            .font(.headline)
                            .foregroundColor(timeRemaining <= 5 ? .red : .white)
                    }
                    .padding()
                    
                    Spacer()
                }
                
                // Targets
                ForEach(targets) { target in
                    TargetView(target: target) {
                        hitTarget(target: target)
                    }
                }
            }
        }
    }
    
    func startGame() {
        score = 0
        timeRemaining = 30
        targets = []
        isPlaying = true
        
        // Timer นับถอยหลัง
        timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { _ in
            if timeRemaining > 0 {
                timeRemaining -= 1
            } else {
                endGame()
            }
        }
        
        // Timer สร้าง target
        spawnTimer = Timer.scheduledTimer(withTimeInterval: 0.8, repeats: true) { _ in
            spawnTarget()
        }
    }
    
    func spawnTarget() {
        let target = GameTarget(
            position: CGPoint(
                x: CGFloat.random(in: 50...350),
                y: CGFloat.random(in: 100...700)
            ),
            size: CGFloat.random(in: 40...80),
            color: [Color.red, Color.blue, Color.green, Color.yellow, Color.orange].randomElement() ?? .red
        )
        targets.append(target)
        
        // ลบ target หลังจาก 2 วินาที
        DispatchQueue.main.asyncAfter(deadline: .now() + 2) {
            targets.removeAll { $0.id == target.id }
        }
    }
    
    func hitTarget(target: GameTarget) {
        score += 10
        targets.removeAll { $0.id == target.id }
    }
    
    func endGame() {
        timer?.invalidate()
        spawnTimer?.invalidate()
        isPlaying = false
        targets = []
    }
}

struct GameTarget: Identifiable {
    let id = UUID()
    var position: CGPoint
    var size: CGFloat
    var color: Color
}

struct TargetView: View {
    let target: GameTarget
    let onTap: () -> Void
    
    @State private var scale: CGFloat = 0
    @State private var opacity: Double = 1
    
    var body: some View {
        Circle()
            .fill(target.color)
            .frame(width: target.size, height: target.size)
            .scaleEffect(scale)
            .opacity(opacity)
            .position(target.position)
            .onTapGesture {
                withAnimation(.easeOut(duration: 0.1)) {
                    scale = 1.5
                    opacity = 0
                }
                onTap()
            }
            .onAppear {
                withAnimation(.spring(response: 0.3)) {
                    scale = 1.0
                }
            }
    }
}
```

---

## 14. Gesture กับ ScrollView

### ScrollView กับ Gesture

```swift
struct ScrollViewGestureExample: View {
    @State private var selectedItem: Int?
    
    var body: some View {
        ScrollView {
            LazyVStack(spacing: 10) {
                ForEach(1...30, id: \.self) { index in
                    RoundedRectangle(cornerRadius: 12)
                        .fill(selectedItem == index ? Color.blue : Color.gray.opacity(0.2))
                        .frame(height: 60)
                        .overlay(
                            Text("รายการ \(index)")
                                .foregroundColor(selectedItem == index ? .white : .primary)
                        )
                        // ใช้ simultaneousGesture เพื่อไม่รบกวน scroll
                        .simultaneousGesture(
                            TapGesture()
                                .onEnded {
                                    withAnimation {
                                        selectedItem = selectedItem == index ? nil : index
                                    }
                                }
                        )
                        .padding(.horizontal)
                }
            }
        }
    }
}
```

---

## 15. สรุป (Summary)

ในบทนี้เราได้เรียนรู้เกี่ยวกับ Gestures ใน SwiftUI ทั้งหมด:

### สิ่งที่เรียนรู้

| Gesture | การใช้งาน |
|---------|-----------|
| `TapGesture` | ตรวจจับการแตะ 1 หรือหลายครั้ง |
| `LongPressGesture` | ตรวจจับการกดค้าง |
| `DragGesture` | ตรวจจับการลาก |
| `MagnifyGesture` | ตรวจจับการ pinch ย่อ/ขยาย |
| `RotateGesture` | ตรวจจับการหมุน |

### Gesture Composition

- **simultaneously**: ทำงานพร้อมกัน
- **sequenced**: ทำงานตามลำดับ
- **exclusively**: ทำงานอย่างใดอย่างหนึ่ง

### Property Wrappers

- **@GestureState**: รีเซ็ตอัตโนมัติเมื่อ gesture สิ้นสุด
- **@State**: ไม่รีเซ็ตอัตโนมัติ

### Callbacks

- **onChanged**: เรียกระหว่าง gesture
- **onEnded**: เรียกเมื่อสิ้นสุด
- **updating**: เรียกพร้อม transaction

### Conflict Resolution

- **highPriorityGesture**: ให้สิทธิ์ gesture สูงกว่า
- **simultaneousGesture**: ทำงานพร้อมกับ parent gestures
- **gesture(_, including:)**: ควบคุม gesture priority

### Best Practices

1. ใช้ `@GestureState` สำหรับค่าที่ต้องรีเซ็ตเมื่อ gesture สิ้นสุด
2. ใช้ `withAnimation` เพื่อเพิ่ม animation ให้ smooth
3. เพิ่ม Accessibility label/hint เสมอ
4. ทดสอบบนอุปกรณ์จริงสำหรับ multi-touch gestures
5. ใช้ `simultaneousGesture` เมื่อต้องการทำงานร่วมกับ ScrollView

---

## แบบฝึกหัดเพิ่มเติม

### แบบฝึกหัดท้าทาย 1: Drawing App

สร้าง App วาดรูปด้วย DragGesture:
- ใช้ DragGesture วาดเส้น
- รองรับการเปลี่ยนสี
- รองรับการ undo
- เพิ่มเครื่องมือ eraser

### แบบฝึกหัดท้าทาย 2: Card Sorting

สร้าง App จัดเรียงการ์ดด้วย Drag:
- Drag การ์ดจากตำแหน่งหนึ่งไปอีกตำแหน่ง
- ตรวจจับการ drop บน zone ที่กำหนด
- Animation เมื่อ sort เสร็จ

### แบบฝึกหัดท้าทาย 3: Gesture-Controlled Slider

สร้าง custom slider ด้วย DragGesture:
- ลากซ้าย-ขวาเพื่อปรับค่า
- แสดงค่าปัจจุบัน
- รองรับ haptic feedback

---

*จบบท Part 27: SwiftUI Gestures*
