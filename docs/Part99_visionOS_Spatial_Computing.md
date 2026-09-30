# Part 99: visionOS และ Spatial Computing สำหรับ Apple Vision Pro

## บทนำ

Apple Vision Pro และ visionOS เป็นการปฏิวัติครั้งใหม่ในวงการ computing โดย Apple ได้นำเสนอแนวคิด **Spatial Computing** ที่ผสมผสานโลกจริงและโลกดิจิทัลเข้าด้วยกันอย่างไร้รอยต่อ ในบทนี้เราจะเรียนรู้การพัฒนาแอปพลิเคชันสำหรับ visionOS ตั้งแต่พื้นฐานจนถึงระดับ advanced

---

## 1. visionOS Overview: แนวคิด Spatial Computing

### 1.1 Spatial Computing คืออะไร?

**Spatial Computing** คือการคำนวณและการโต้ตอบกับข้อมูลในพื้นที่สามมิติรอบตัวผู้ใช้ แทนที่จะถูกจำกัดอยู่แค่หน้าจอสองมิติ Apple Vision Pro ทำให้ผู้ใช้สามารถ:

- วาง **Windows** และ **3D content** ในพื้นที่จริงรอบตัว
- โต้ตอบด้วย **eye tracking**, **hand tracking**, และ **voice**
- ดำดิ่งสู่ประสบการณ์ **Immersive** เต็มรูปแบบ
- ทำงานร่วมกับผู้อื่นในพื้นที่เสมือน (**SharePlay**)

```
โลกจริง (Physical World)
        +
ดิจิทัล Content (Digital Content)
        =
Spatial Computing Experience
```

### 1.2 visionOS vs iOS: ความแตกต่างหลัก

| คุณสมบัติ | iOS/iPadOS | visionOS |
|-----------|-----------|----------|
| หน้าจอ | 2D flat screen | 3D spatial environment |
| Input | Touch, keyboard | Eyes, hands, voice |
| Window system | Single/multitasking windows | Spatial windows + volumes + immersive |
| Depth | ไม่มี | Z-axis เต็มรูปแบบ |
| Rendering | UIKit/SwiftUI 2D | SwiftUI + RealityKit 3D |
| Audio | Stereo | Spatial audio 3D |
| Privacy | Camera/microphone | Eye tracking, room scanning |

### 1.3 visionOS Architecture

```
┌─────────────────────────────────────────┐
│           Application Layer              │
│  SwiftUI + RealityKit + ARKit           │
├─────────────────────────────────────────┤
│           visionOS Frameworks           │
│  WindowKit | RealityKit | ARKit | AVF   │
├─────────────────────────────────────────┤
│           System Services               │
│  Compositor | Eye Tracking | Hand Track │
├─────────────────────────────────────────┤
│           Hardware                      │
│  Display | Cameras | Sensors | Audio   │
└─────────────────────────────────────────┘
```

### 1.4 Hardware ของ Apple Vision Pro

**Eye Tracking System:**
- กล้อง infrared สำหรับติดตามดวงตา
- ความแม่นยำระดับ sub-degree
- ใช้สำหรับ focus และ interaction
- ประมวลผลใน Secure Enclave เพื่อความเป็นส่วนตัว

**Hand Tracking System:**
- กล้องและ sensors ติดตามมือและนิ้ว
- รู้จัก 26 joint positions ต่อมือ
- Gesture recognition แบบ real-time
- ทำงานได้โดยไม่ต้องสวม controller

**Spatial Audio:**
- ลำโพงที่อยู่ใกล้หูแต่ไม่ได้ใส่หู
- Personalized spatial audio สำหรับแต่ละคน
- Environmental matching สำหรับ reverb
- Head tracking สำหรับ audio positioning

**Sensors อื่นๆ:**
- LiDAR scanner สำหรับ depth sensing
- Cameras สำหรับ passthrough และ tracking
- IMU สำหรับ head movement
- Proximity sensors

---

## 2. Window Types ใน visionOS

visionOS มี 3 ประเภทหลักของ "spaces" ที่แอปใช้งานได้:

### 2.1 Window (2D Window)

Window ธรรมดาเหมือน iOS แต่ลอยอยู่ในอากาศ สามารถย้ายและ resize ได้โดยผู้ใช้

```swift
import SwiftUI

@main
struct MyVisionApp: App {
    var body: some Scene {
        // WindowGroup แบบปกติ = 2D Window
        WindowGroup {
            ContentView()
        }
    }
}
```

### 2.2 Volume (3D Window)

Volume คือ window ที่มีความลึก (depth) สำหรับแสดง 3D content

```swift
@main
struct MyVisionApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        
        // Volumetric window สำหรับ 3D content
        WindowGroup(id: "3d-content") {
            Volume3DView()
        }
        .windowStyle(.volumetric)
    }
}
```

### 2.3 Immersive Space

Immersive Space เปิดให้แอปควบคุมพื้นที่รอบตัวผู้ใช้ทั้งหมด

```swift
@main
struct MyVisionApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        
        // Immersive Space
        ImmersiveSpace(id: "immersive-experience") {
            ImmersiveView()
        }
        .immersionStyle(selection: .constant(.mixed), in: .mixed, .progressive, .full)
    }
}
```

**Immersion Styles:**
- `.mixed` - เห็นโลกจริงและ digital content ผสมกัน
- `.progressive` - ค่อยๆ เพิ่ม immersion
- `.full` - ดำดิ่งเต็มรูปแบบ ไม่เห็นโลกจริง

---

## 3. การตั้งค่า visionOS Project

### 3.1 การสร้าง visionOS Project ใน Xcode

1. เปิด **Xcode** → **File** → **New** → **Project**
2. เลือก **visionOS** platform
3. เลือก template:
   - **App** - แอปพื้นฐาน
   - **Game** - สำหรับเกม
4. ตั้งชื่อ project และ bundle identifier
5. เลือก **Swift** และ **SwiftUI**

### 3.2 Project Structure

```
MyVisionApp/
├── MyVisionApp.swift          # App entry point
├── ContentView.swift          # Main 2D view
├── ImmersiveView.swift        # Immersive experience
├── RealityKitContent/         # Reality Composer Pro package
│   ├── Package.swift
│   └── Sources/
│       └── RealityKitContent/
│           └── RealityKitContent.rkassets  # 3D assets
└── Assets.xcassets
```

### 3.3 Info.plist สำหรับ visionOS

```xml
<!-- NSWorldSensingUsageDescription - สำหรับ plane detection -->
<key>NSWorldSensingUsageDescription</key>
<string>แอปนี้ต้องการสแกนพื้นที่เพื่อวาง 3D content</string>

<!-- NSHandsTrackingUsageDescription -->
<key>NSHandsTrackingUsageDescription</key>
<string>แอปนี้ใช้การติดตามมือสำหรับ interaction</string>
```

### 3.4 Required Capabilities

```swift
// ใน Signing & Capabilities เพิ่ม:
// - Hand Tracking (com.apple.developer.arkit.hand-tracking)
// - World Sensing (com.apple.developer.arkit.world-sensing)
```

### 3.5 visionOS Simulator

**การใช้ visionOS Simulator:**
- เลือก "Apple Vision Pro" ใน device picker
- ใช้ mouse สำหรับจำลอง eye cursor
- ใช้ Option+drag สำหรับจำลอง hands
- Simulator ไม่รองรับ passthrough จริง

**ข้อจำกัดของ Simulator:**
- ไม่มี eye tracking จริง
- Hand tracking เป็นแบบจำลอง
- Spatial audio ทำงานแบบ stereo
- Performance แตกต่างจาก device จริง

---

## 4. SwiftUI บน visionOS

### 4.1 WindowGroup สำหรับ 2D Windows

```swift
import SwiftUI

@main
struct ProductViewerApp: App {
    @State private var showImmersive = false
    @State private var immersionStyle: ImmersionStyle = .mixed
    
    var body: some Scene {
        // 2D Window หลัก
        WindowGroup {
            MainMenuView(showImmersive: $showImmersive)
        }
        .windowStyle(.plain)
        .defaultSize(width: 800, height: 600)
        
        // 3D Volume window
        WindowGroup(id: "product-3d") {
            Product3DView()
        }
        .windowStyle(.volumetric)
        .defaultSize(width: 0.5, height: 0.5, depth: 0.5, in: .meters)
        
        // Immersive Space
        ImmersiveSpace(id: "full-experience") {
            FullImmersiveView()
        }
        .immersionStyle(selection: $immersionStyle, in: .mixed, .full)
    }
}
```

### 4.2 การเปิด/ปิด Windows และ Spaces

```swift
struct MainMenuView: View {
    @Environment(\.openWindow) private var openWindow
    @Environment(\.openImmersiveSpace) private var openImmersiveSpace
    @Environment(\.dismissImmersiveSpace) private var dismissImmersiveSpace
    @Environment(\.dismissWindow) private var dismissWindow
    
    @Binding var showImmersive: Bool
    
    var body: some View {
        VStack(spacing: 20) {
            Text("Product Viewer")
                .font(.extraLargeTitle)
            
            Button("เปิด 3D View") {
                openWindow(id: "product-3d")
            }
            
            Button(showImmersive ? "ปิด Immersive" : "เปิด Immersive") {
                Task {
                    if showImmersive {
                        await dismissImmersiveSpace()
                        showImmersive = false
                    } else {
                        let result = await openImmersiveSpace(id: "full-experience")
                        switch result {
                        case .opened:
                            showImmersive = true
                        case .userCancelled, .error:
                            showImmersive = false
                        @unknown default:
                            showImmersive = false
                        }
                    }
                }
            }
            .buttonStyle(.borderedProminent)
        }
        .padding(40)
    }
}
```

### 4.3 Ornament Modifier

**Ornament** คือ UI element ที่ "ลอย" อยู่รอบๆ window เช่น toolbar ด้านล่าง

```swift
struct ContentView: View {
    @State private var selectedTab = 0
    
    var body: some View {
        NavigationStack {
            TabView(selection: $selectedTab) {
                HomeView()
                    .tabItem { Label("Home", systemImage: "house") }
                    .tag(0)
                
                LibraryView()
                    .tabItem { Label("Library", systemImage: "books.vertical") }
                    .tag(1)
            }
        }
        .ornament(
            visibility: .visible,
            attachmentAnchor: .scene(.bottom)
        ) {
            // Ornament content ที่ลอยอยู่ด้านล่าง window
            HStack(spacing: 20) {
                Button("Home") { selectedTab = 0 }
                    .buttonStyle(.borderless)
                
                Button("Library") { selectedTab = 1 }
                    .buttonStyle(.borderless)
            }
            .padding()
            .glassBackgroundEffect()
        }
    }
}
```

### 4.4 glassBackgroundEffect

Effect พิเศษของ visionOS ที่ทำให้ background ดูเหมือนกระจกใส

```swift
struct GlassCard: View {
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            Text("Product Details")
                .font(.title2)
                .bold()
            
            Text("ราคา: ฿29,900")
                .font(.headline)
            
            Text("Apple Vision Pro เปลี่ยนวิธีที่คุณโต้ตอบกับ digital content")
                .font(.body)
                .foregroundStyle(.secondary)
        }
        .padding(20)
        .frame(width: 300)
        // Glass background effect
        .glassBackgroundEffect()
    }
}
```

### 4.5 3D Transformations ใน SwiftUI

```swift
struct Rotating3DView: View {
    @State private var rotation: Double = 0
    
    var body: some View {
        Model3D(named: "Robot", bundle: realityKitContentBundle) { model in
            model
                .resizable()
                .scaledToFit()
        } placeholder: {
            ProgressView()
        }
        .rotation3DEffect(
            .degrees(rotation),
            axis: (x: 0, y: 1, z: 0)
        )
        .onAppear {
            withAnimation(.linear(duration: 3).repeatForever(autoreverses: false)) {
                rotation = 360
            }
        }
        .frame(width: 300, height: 300)
    }
}
```

---

## 5. RealityKit บน visionOS

### 5.1 RealityView: SwiftUI Component ใหม่

**RealityView** คือ SwiftUI view ที่ฝัง RealityKit content ได้โดยตรง

```swift
import SwiftUI
import RealityKit

struct BasicRealityView: View {
    var body: some View {
        RealityView { content in
            // content: RealityViewContent
            // เพิ่ม entities ที่นี่
            
            // สร้าง box entity
            let mesh = MeshResource.generateBox(size: 0.3)
            let material = SimpleMaterial(color: .blue, isMetallic: true)
            let boxEntity = ModelEntity(mesh: mesh, materials: [material])
            
            // วางตำแหน่ง
            boxEntity.position = [0, 0, -0.5]
            
            // เพิ่มใน content
            content.add(boxEntity)
            
        } update: { content in
            // อัปเดต content เมื่อ state เปลี่ยน
        }
        .frame(width: 400, height: 400)
    }
}
```

### 5.2 Entity-Component System (ECS)

RealityKit ใช้ **Entity-Component System** ซึ่งเป็น architecture pattern สำหรับ 3D

```swift
// Entity = container
let entity = Entity()

// Component = data/behavior
struct HealthComponent: Component {
    var currentHP: Float
    var maxHP: Float
    
    init(maxHP: Float) {
        self.maxHP = maxHP
        self.currentHP = maxHP
    }
}

// เพิ่ม component
entity.components.set(HealthComponent(maxHP: 100))

// อ่าน component
if let health = entity.components[HealthComponent.self] {
    print("HP: \(health.currentHP)/\(health.maxHP)")
}

// ModelEntity = Entity + Model component
let modelEntity = ModelEntity(
    mesh: .generateSphere(radius: 0.1),
    materials: [SimpleMaterial(color: .red, isMetallic: false)]
)
```

### 5.3 AnchorEntity ใน visionOS

```swift
struct AnchoredContentView: View {
    var body: some View {
        RealityView { content in
            // AnchorEntity.world = ยึดกับโลกจริง
            let worldAnchor = AnchorEntity(world: [0, 0, -1])
            
            // สร้าง sphere ที่ 1 เมตรข้างหน้า
            let sphere = ModelEntity(
                mesh: .generateSphere(radius: 0.1),
                materials: [SimpleMaterial(color: .cyan, isMetallic: true)]
            )
            
            worldAnchor.addChild(sphere)
            content.add(worldAnchor)
            
            // Plane anchor - วางบนพื้น
            let planeAnchor = AnchorEntity(.plane(.horizontal, classification: .floor, minimumBounds: [0.2, 0.2]))
            
            let floorSphere = ModelEntity(
                mesh: .generateSphere(radius: 0.05),
                materials: [SimpleMaterial(color: .green, isMetallic: false)]
            )
            
            planeAnchor.addChild(floorSphere)
            content.add(planeAnchor)
        }
    }
}
```

### 5.4 Materials ใน visionOS

```swift
// SimpleMaterial - ง่ายและเร็ว
let simple = SimpleMaterial(color: .blue, roughness: 0.3, isMetallic: true)

// PhysicallyBasedMaterial - PBR material
var pbr = PhysicallyBasedMaterial()
pbr.baseColor = .init(tint: .white)
pbr.metallic = .init(floatLiteral: 0.8)
pbr.roughness = .init(floatLiteral: 0.2)

// UnlitMaterial - ไม่รับแสง เหมาะสำหรับ UI elements
var unlit = UnlitMaterial()
unlit.color = .init(tint: .red.withAlphaComponent(0.8))

// VideoMaterial - แสดงวิดีโอบน surface
let player = AVPlayer(url: videoURL)
var videoMat = VideoMaterial(avPlayer: player)

// OcclusionMaterial - ทำให้พื้นที่โปร่งใสแต่ block objects ที่อยู่ข้างหลัง
let occlusion = OcclusionMaterial()
```

### 5.5 RealityView Attachments

นำ SwiftUI views มาใส่ใน RealityKit scene ได้:

```swift
struct AttachmentExample: View {
    var body: some View {
        RealityView { content, attachments in
            // โหลด 3D entity
            if let robot = try? await ModelEntity(named: "Robot") {
                robot.position = [0, 0, -1]
                content.add(robot)
            }
            
            // ใส่ SwiftUI view เป็น attachment
            if let nameTag = attachments.entity(for: "nameTag") {
                nameTag.position = [0, 0.3, -1]
                content.add(nameTag)
            }
            
        } update: { content, attachments in
            // อัปเดต
        } attachments: {
            // กำหนด SwiftUI view สำหรับ attachment
            Attachment(id: "nameTag") {
                Text("Robot #1")
                    .font(.title3)
                    .padding(8)
                    .glassBackgroundEffect()
            }
        }
    }
}
```

---

## 6. 3D Content และ Assets

### 6.1 USDZ Format

**USDZ** (Universal Scene Description Zip) คือ format มาตรฐานของ Apple สำหรับ 3D content

```swift
// โหลด USDZ จาก bundle
let entity = try await ModelEntity(named: "AirPods.usdz")

// โหลดจาก URL
let url = Bundle.main.url(forResource: "product", withExtension: "usdz")!
let entity2 = try await ModelEntity(contentsOf: url)

// โหลดพร้อม configuration
let config = Entity.LoadConfig(loadPhysics: true, loadCollision: true)
let entity3 = try await Entity(named: "Scene", in: realityKitContentBundle)
```

### 6.2 Reality Composer Pro

**Reality Composer Pro** เป็นเครื่องมือใน Xcode สำหรับออกแบบ 3D scenes

**การสร้าง RealityComposerPro package:**

1. ใน Xcode → **File** → **New** → **Package**
2. เลือก **Reality Composer Pro Package**
3. สร้าง `.rkassets` bundle

```swift
// เข้าถึง assets จาก Reality Composer Pro
import RealityKitContent

struct SceneFromRCPro: View {
    var body: some View {
        RealityView { content in
            // โหลด scene จาก Reality Composer Pro
            if let scene = try? await Entity(
                named: "MainScene",
                in: realityKitContentBundle
            ) {
                content.add(scene)
            }
        }
    }
}
```

### 6.3 Physics Simulation

```swift
struct PhysicsDemo: View {
    var body: some View {
        RealityView { content in
            // สร้าง physics world
            let floor = ModelEntity(
                mesh: .generateBox(size: [2, 0.01, 2]),
                materials: [SimpleMaterial(color: .gray, isMetallic: false)]
            )
            floor.position = [0, -0.5, -1]
            
            // เพิ่ม physics collider
            floor.components.set(CollisionComponent(
                shapes: [.generateBox(size: [2, 0.01, 2])]
            ))
            floor.components.set(PhysicsBodyComponent(
                shapes: [.generateBox(size: [2, 0.01, 2])],
                mass: 0,
                mode: .static  // ไม่เคลื่อนที่
            ))
            
            content.add(floor)
            
            // สร้าง ball ที่มี physics
            let ball = ModelEntity(
                mesh: .generateSphere(radius: 0.05),
                materials: [SimpleMaterial(color: .red, isMetallic: true)]
            )
            ball.position = [0, 0.5, -1]
            
            ball.components.set(CollisionComponent(
                shapes: [.generateSphere(radius: 0.05)]
            ))
            ball.components.set(PhysicsBodyComponent(
                shapes: [.generateSphere(radius: 0.05)],
                mass: 1.0,
                mode: .dynamic  // ตกลงตาม gravity
            ))
            
            content.add(ball)
        }
    }
}
```

### 6.4 Particle Systems

```swift
struct ParticleEffect: View {
    var body: some View {
        RealityView { content in
            // สร้าง particle emitter
            var emitter = ParticleEmitterComponent()
            emitter.emitterShape = .sphere
            emitter.emitterShapeSize = [0.01, 0.01, 0.01]
            emitter.birthRate = 100
            emitter.lifeSpan = 2.0
            
            // กำหนด particle properties
            emitter.mainEmitter.size = 0.005
            emitter.mainEmitter.sizeVariation = 0.002
            
            emitter.mainEmitter.color = .evolving(
                start: .single(.init(tint: .yellow, opacity: 1.0)),
                end: .single(.init(tint: .orange, opacity: 0.0))
            )
            
            emitter.mainEmitter.speed = 0.1
            emitter.mainEmitter.speedVariation = 0.05
            
            let emitterEntity = Entity()
            emitterEntity.components.set(emitter)
            emitterEntity.position = [0, 0, -0.8]
            
            content.add(emitterEntity)
        }
    }
}
```

---

## 7. Hand Tracking

### 7.1 HandAnchor และ HandSkeleton

```swift
import ARKit
import RealityKit
import SwiftUI

struct HandTrackingView: View {
    let session = ARKitSession()
    let handTracking = HandTrackingProvider()
    
    @State private var leftHandPosition: SIMD3<Float>?
    @State private var rightHandPosition: SIMD3<Float>?
    
    var body: some View {
        RealityView { content in
            // เพิ่ม visual indicators สำหรับ hands
        }
        .task {
            await startHandTracking()
        }
    }
    
    func startHandTracking() async {
        // ขอ permission
        let authResult = await session.requestAuthorization(for: [.handTracking])
        guard authResult[.handTracking] == .allowed else {
            print("Hand tracking ไม่ได้รับอนุญาต")
            return
        }
        
        // เริ่ม tracking
        do {
            try await session.run([handTracking])
            
            // รับข้อมูลแบบ async stream
            for await update in handTracking.anchorUpdates {
                let anchor = update.anchor
                
                switch anchor.chirality {
                case .left:
                    leftHandPosition = anchor.originFromAnchorTransform.translation
                case .right:
                    rightHandPosition = anchor.originFromAnchorTransform.translation
                @unknown default:
                    break
                }
                
                // เข้าถึง skeleton
                if let skeleton = anchor.handSkeleton {
                    // ดูตำแหน่งนิ้วโป้ง
                    let thumbTip = skeleton.joint(.thumbTip)
                    let thumbPos = thumbTip.anchorFromJointTransform.translation
                    
                    // ดูตำแหน่งนิ้วชี้
                    let indexTip = skeleton.joint(.indexFingerTip)
                    let indexPos = indexTip.anchorFromJointTransform.translation
                    
                    // คำนวณระยะห่างระหว่างนิ้วโป้งและนิ้วชี้
                    let distance = length(thumbPos - indexPos)
                    
                    if distance < 0.02 {
                        print("Pinch gesture detected!")
                    }
                }
            }
        } catch {
            print("Error: \(error)")
        }
    }
}
```

### 7.2 Hand Skeleton Joints

```swift
// Joint names ที่สำคัญ
HandSkeleton.JointName.wrist                // ข้อมือ
HandSkeleton.JointName.thumbKnuckle         // ข้อนิ้วโป้ง
HandSkeleton.JointName.thumbIntermediateBase
HandSkeleton.JointName.thumbIntermediateTip
HandSkeleton.JointName.thumbTip            // ปลายนิ้วโป้ง
HandSkeleton.JointName.indexFingerMetacarpal
HandSkeleton.JointName.indexFingerKnuckle
HandSkeleton.JointName.indexFingerIntermediateBase
HandSkeleton.JointName.indexFingerIntermediateTip
HandSkeleton.JointName.indexFingerTip      // ปลายนิ้วชี้
// ... (ทำนองเดียวกันสำหรับนิ้วอื่นๆ)
```

### 7.3 Custom Gesture Recognition

```swift
class PinchGestureDetector {
    private let pinchThreshold: Float = 0.02  // 2 cm
    
    struct PinchState {
        var isPinching: Bool
        var pinchPosition: SIMD3<Float>
        var pinchStrength: Float  // 0 = open, 1 = fully pinched
    }
    
    func detectPinch(in skeleton: HandSkeleton) -> PinchState {
        let thumbTip = skeleton.joint(.thumbTip)
        let indexTip = skeleton.joint(.indexFingerTip)
        
        let thumbPos = SIMD3<Float>(thumbTip.anchorFromJointTransform.columns.3.x,
                                    thumbTip.anchorFromJointTransform.columns.3.y,
                                    thumbTip.anchorFromJointTransform.columns.3.z)
        
        let indexPos = SIMD3<Float>(indexTip.anchorFromJointTransform.columns.3.x,
                                     indexTip.anchorFromJointTransform.columns.3.y,
                                     indexTip.anchorFromJointTransform.columns.3.z)
        
        let distance = length(thumbPos - indexPos)
        let isPinching = distance < pinchThreshold
        let strength = max(0, min(1, (pinchThreshold - distance) / pinchThreshold))
        let midpoint = (thumbPos + indexPos) / 2
        
        return PinchState(
            isPinching: isPinching,
            pinchPosition: midpoint,
            pinchStrength: strength
        )
    }
}

// Spatial Tap Gesture
struct GestureView: View {
    var body: some View {
        RealityView { content in
            let box = ModelEntity(
                mesh: .generateBox(size: 0.1),
                materials: [SimpleMaterial(color: .blue, isMetallic: true)]
            )
            box.position = [0, 0, -0.5]
            box.components.set(CollisionComponent(shapes: [.generateBox(size: [0.1, 0.1, 0.1])]))
            box.components.set(InputTargetComponent())
            content.add(box)
        }
        .gesture(
            SpatialTapGesture()
                .targetedToAnyEntity()
                .onEnded { value in
                    // ผู้ใช้ pinch หรือ tap entity
                    let entity = value.entity
                    // เปลี่ยนสี
                    if var model = entity.components[ModelComponent.self] {
                        model.materials = [SimpleMaterial(color: .red, isMetallic: true)]
                        entity.components.set(model)
                    }
                }
        )
    }
}
```

### 7.4 AnchorEntity สำหรับ Hands

```swift
struct HandAttachedContent: View {
    var body: some View {
        RealityView { content in
            // สร้าง entity ที่ติดกับมือขวา
            let handAnchor = AnchorEntity(.hand(.right, location: .palm))
            
            let orb = ModelEntity(
                mesh: .generateSphere(radius: 0.02),
                materials: [SimpleMaterial(color: .cyan, isMetallic: true)]
            )
            
            handAnchor.addChild(orb)
            content.add(handAnchor)
            
            // ติดกับนิ้วชี้
            let indexAnchor = AnchorEntity(.hand(.right, location: .indexFingerTip))
            
            let tip = ModelEntity(
                mesh: .generateSphere(radius: 0.01),
                materials: [SimpleMaterial(color: .yellow, isMetallic: true)]
            )
            
            indexAnchor.addChild(tip)
            content.add(indexAnchor)
        }
    }
}
```

---

## 8. Eye Tracking

### 8.1 SpatialEventGesture

```swift
struct EyeTrackingView: View {
    @State private var gazedEntity: Entity?
    
    var body: some View {
        RealityView { content in
            // สร้าง sphere ที่ตอบสนองต่อ gaze
            for i in 0..<5 {
                let sphere = ModelEntity(
                    mesh: .generateSphere(radius: 0.05),
                    materials: [SimpleMaterial(color: .blue, isMetallic: true)]
                )
                sphere.position = [Float(i) * 0.15 - 0.3, 0, -0.8]
                sphere.components.set(CollisionComponent(shapes: [.generateSphere(radius: 0.05)]))
                sphere.components.set(InputTargetComponent())
                sphere.name = "sphere_\(i)"
                content.add(sphere)
            }
        }
        .gesture(
            // Tap gesture (triggered by pinch while looking)
            SpatialTapGesture()
                .targetedToAnyEntity()
                .onEnded { value in
                    print("Tapped: \(value.entity.name)")
                    animateEntity(value.entity)
                }
        )
    }
    
    func animateEntity(_ entity: Entity) {
        var transform = entity.transform
        transform.scale = [1.5, 1.5, 1.5]
        
        entity.move(to: transform, relativeTo: entity.parent, duration: 0.1, timingFunction: .easeIn)
        
        // Reset
        DispatchQueue.main.asyncAfter(deadline: .now() + 0.2) {
            var original = entity.transform
            original.scale = [1, 1, 1]
            entity.move(to: original, relativeTo: entity.parent, duration: 0.1, timingFunction: .easeOut)
        }
    }
}
```

### 8.2 Hover Effects

**Hover effect** ทำงานเมื่อผู้ใช้มองไปที่ element (eye gaze)

```swift
struct HoverEffectView: View {
    @State private var isHovered = false
    
    var body: some View {
        VStack {
            // Standard hover effect (visionOS จัดการ highlight อัตโนมัติ)
            Button("คลิกฉัน") {
                print("Tapped!")
            }
            .buttonStyle(.bordered)
            .hoverEffect()  // เพิ่ม highlight effect อัตโนมัติ
            
            // Custom hover effect
            RoundedRectangle(cornerRadius: 12)
                .fill(isHovered ? Color.blue : Color.gray)
                .frame(width: 200, height: 60)
                .hoverEffect { effect, isActive, _ in
                    effect.scaleEffect(isActive ? 1.05 : 1.0)
                }
            
            // Highlight effect
            Text("Hover บนฉัน")
                .padding()
                .background(.regularMaterial)
                .hoverEffect(.highlight)
        }
    }
}
```

### 8.3 Gaze-based Interactions

```swift
// Custom component สำหรับ track gaze
struct GazeTrackingComponent: Component {
    var isBeingGazed: Bool = false
    var gazeStartTime: Date?
    var dwellDuration: TimeInterval = 2.0  // เวลาที่ต้องมองเพื่อ activate
}

// System สำหรับประมวลผล gaze
class GazeTrackingSystem: System {
    required init(scene: RealityKit.Scene) {}
    
    func update(context: SceneUpdateContext) {
        // ในแอปจริง ต้องใช้ ARKit eye tracking data
        // นี่เป็น pseudocode
    }
}
```

---

## 9. Spatial Audio

### 9.1 AudioFileGroupComponent

```swift
struct SpatialAudioDemo: View {
    var body: some View {
        RealityView { content in
            // สร้าง audio source entity
            let audioEntity = Entity()
            audioEntity.position = [0, 0, -1]
            
            // เพิ่ม spatial audio component
            let audioResource = try! AudioFileResource(
                named: "ambient_sound.m4a",
                configuration: .init(
                    loadingStrategy: .preload,
                    shouldLoop: true
                )
            )
            
            // Spatial audio - เสียงมาจากตำแหน่งของ entity
            audioEntity.components.set(SpatialAudioComponent(
                gain: 0,           // dB
                directivity: .beam(focus: 0.75),
                reverbLevel: -6    // dB
            ))
            
            content.add(audioEntity)
            
            // เล่นเสียง
            audioEntity.playAudio(audioResource)
        }
    }
}
```

### 9.2 Ambient Audio

```swift
// Ambient audio - เสียงรอบทิศทาง ไม่มีตำแหน่งเฉพาะ
let ambientEntity = Entity()

ambientEntity.components.set(AmbientAudioComponent(gain: -10))

let ambientResource = try! AudioFileResource(
    named: "background_music.m4a",
    configuration: .init(shouldLoop: true)
)

ambientEntity.playAudio(ambientResource)
```

### 9.3 AudioFileGroupComponent สำหรับ Multiple Sounds

```swift
// สำหรับเล่นเสียงหลายแบบจาก entity เดียว
let characterEntity = Entity()

let audioGroup = AudioFileGroupResource(
    ["footstep_1.m4a", "footstep_2.m4a", "footstep_3.m4a"]
        .map { try! AudioFileResource(named: $0) }
)

characterEntity.components.set(SpatialAudioComponent())

// เล่น random จาก group
characterEntity.playAudio(audioGroup.randomElement()!)
```

### 9.4 Environment-based Reverb

```swift
// visionOS จะ analyze ห้องจริงและใส่ reverb ที่เหมาะสมอัตโนมัติ
// ผ่าน World Sensing + acoustic analysis

var spatialAudio = SpatialAudioComponent()
spatialAudio.reverbLevel = 0  // 0 = match environment
spatialAudio.directLevel = -6  // เสียงตรง

entity.components.set(spatialAudio)
```

---

## 10. SharePlay ใน visionOS

### 10.1 GroupActivity Protocol

```swift
import GroupActivities

// กำหนด shared activity
struct WatchTogetherActivity: GroupActivity {
    // ข้อมูลที่แชร์กัน
    var productID: String
    var productName: String
    
    static var activityIdentifier = "com.example.app.watch-together"
    
    var metadata: GroupActivityMetadata {
        var metadata = GroupActivityMetadata()
        metadata.title = "ดูสินค้าด้วยกัน"
        metadata.subtitle = productName
        metadata.type = .watchTogether
        return metadata
    }
}
```

### 10.2 Shared Immersive Experience

```swift
class SharedExperienceModel: ObservableObject {
    @Published var session: GroupSession<WatchTogetherActivity>?
    @Published var sharedProductID: String?
    
    private var messenger: GroupSessionMessenger?
    private var tasks = Set<Task<Void, Never>>()
    
    func start(productID: String, productName: String) async {
        let activity = WatchTogetherActivity(
            productID: productID,
            productName: productName
        )
        
        switch await activity.prepareForActivation() {
        case .activationPreferred:
            _ = try? await activity.activate()
        case .activationDisabled:
            // SharePlay ไม่ available
            break
        case .cancelled:
            break
        @unknown default:
            break
        }
    }
    
    func configureGroupSession(_ session: GroupSession<WatchTogetherActivity>) {
        self.session = session
        
        let messenger = GroupSessionMessenger(session: session)
        self.messenger = messenger
        
        // รับ messages จาก participants อื่น
        let task = Task {
            for await (message, _) in messenger.messages(of: ProductSyncMessage.self) {
                await MainActor.run {
                    self.sharedProductID = message.productID
                }
            }
        }
        tasks.insert(task)
        
        session.join()
    }
    
    func syncProduct(_ productID: String) async {
        let message = ProductSyncMessage(productID: productID)
        try? await messenger?.send(message)
    }
}

struct ProductSyncMessage: Codable {
    var productID: String
}
```

---

## 11. ARKit ใน visionOS

### 11.1 World Tracking

```swift
import ARKit

struct WorldTrackingView: View {
    let session = ARKitSession()
    let worldTracking = WorldTrackingProvider()
    
    var body: some View {
        RealityView { content in
            // เพิ่ม content
        }
        .task {
            guard WorldTrackingProvider.isSupported else { return }
            
            let authResult = await session.requestAuthorization(for: [.worldSensing])
            guard authResult[.worldSensing] == .allowed else { return }
            
            do {
                try await session.run([worldTracking])
                
                // Query device position
                for await _ in worldTracking.anchorUpdates {
                    // worldTracking.queryDeviceAnchor(atTimestamp:) ได้ position
                    if let deviceAnchor = worldTracking.queryDeviceAnchor(atTimestamp: CACurrentMediaTime()) {
                        let transform = deviceAnchor.originFromAnchorTransform
                        let position = transform.translation
                        print("Device position: \(position)")
                    }
                }
            } catch {
                print("Error: \(error)")
            }
        }
    }
}
```

### 11.2 Plane Detection

```swift
struct PlaneDetectionView: View {
    let session = ARKitSession()
    let planeDetection = PlaneDetectionProvider(alignments: [.horizontal, .vertical])
    
    @State private var planes: [PlaneAnchor] = []
    
    var body: some View {
        RealityView { content in
            // แสดง detected planes
        } update: { content in
            // อัปเดต plane visualizations
            for plane in planes {
                // สร้าง visual representation
            }
        }
        .task {
            guard PlaneDetectionProvider.isSupported else { return }
            
            do {
                try await session.run([planeDetection])
                
                for await update in planeDetection.anchorUpdates {
                    switch update.event {
                    case .added:
                        planes.append(update.anchor)
                    case .updated:
                        if let index = planes.firstIndex(where: { $0.id == update.anchor.id }) {
                            planes[index] = update.anchor
                        }
                    case .removed:
                        planes.removeAll { $0.id == update.anchor.id }
                    }
                }
            } catch {
                print("Error: \(error)")
            }
        }
    }
}
```

### 11.3 Image Tracking

```swift
struct ImageTrackingView: View {
    let session = ARKitSession()
    
    // กำหนด reference images ที่ต้องการ track
    let imageTracking = ImageTrackingProvider(
        referenceImages: ReferenceImage.loadReferenceImages(
            inGroupNamed: "AR Resources",
            bundle: .main
        )
    )
    
    var body: some View {
        RealityView { content in
            // content จะอัปเดตเมื่อ detect image
        }
        .task {
            guard ImageTrackingProvider.isSupported else { return }
            
            do {
                try await session.run([imageTracking])
                
                for await update in imageTracking.anchorUpdates {
                    let anchor = update.anchor
                    
                    switch update.event {
                    case .added:
                        print("Detected image: \(anchor.referenceImage.name ?? "unknown")")
                        // วาง 3D content บน detected image
                    case .updated:
                        // อัปเดต position
                        break
                    case .removed:
                        print("Image lost")
                    }
                }
            } catch {
                print("Error: \(error)")
            }
        }
    }
}
```

### 11.4 Scene Reconstruction

```swift
struct SceneReconstructionView: View {
    let session = ARKitSession()
    let sceneReconstruction = SceneReconstructionProvider(modes: [.classification])
    
    var body: some View {
        RealityView { content in
            // Visualize mesh
        }
        .task {
            guard SceneReconstructionProvider.isSupported else { return }
            
            let authResult = await session.requestAuthorization(for: [.worldSensing])
            guard authResult[.worldSensing] == .allowed else { return }
            
            do {
                try await session.run([sceneReconstruction])
                
                for await update in sceneReconstruction.anchorUpdates {
                    let anchor = update.anchor
                    
                    // anchor.geometry ประกอบด้วย mesh ของ environment
                    // anchor.geometryDescriptions มี classification (wall, floor, ceiling, etc.)
                    
                    if update.event == .added || update.event == .updated {
                        // สร้าง collision mesh จาก scene geometry
                        // เพื่อให้ physics objects ไม่ทะลุผ่านผนังจริง
                    }
                }
            } catch {
                print("Error: \(error)")
            }
        }
    }
}
```

---

## 12. Accessibility ใน visionOS

### 12.1 VoiceOver ใน Spatial Environment

```swift
struct AccessibleSpatialView: View {
    var body: some View {
        RealityView { content in
            let productEntity = ModelEntity(
                mesh: .generateBox(size: 0.1),
                materials: [SimpleMaterial(color: .blue, isMetallic: true)]
            )
            productEntity.position = [0, 0, -0.5]
            content.add(productEntity)
        }
        .accessibilityLabel("กล่องสีน้ำเงิน")
        .accessibilityHint("แตะสองครั้งเพื่อดูรายละเอียด")
        .accessibilityAddTraits(.isButton)
    }
}

// SwiftUI views ใน visionOS รองรับ accessibility เหมือน iOS
struct ProductCard: View {
    let product: Product
    
    var body: some View {
        VStack(alignment: .leading) {
            Text(product.name)
                .font(.headline)
            Text(product.price)
                .font(.subheadline)
                .foregroundStyle(.secondary)
        }
        .padding()
        .glassBackgroundEffect()
        // VoiceOver จะอ่าน combined label
        .accessibilityElement(children: .combine)
    }
}
```

### 12.2 Motor Accommodation

```swift
// รองรับผู้ใช้ที่มีข้อจำกัดด้านการเคลื่อนไหว
struct MotorAccommodationView: View {
    @Environment(\.accessibilityReduceMotion) var reduceMotion
    @Environment(\.accessibilityButtonShapes) var buttonShapes
    
    var body: some View {
        VStack {
            // ลด animation สำหรับ reduce motion
            if reduceMotion {
                StaticProductView()
            } else {
                AnimatedProductView()
            }
            
            // ปุ่มที่ใหญ่ขึ้นสำหรับ motor accessibility
            Button("เลือกสินค้า") {}
                .frame(minWidth: buttonShapes ? 44 : 30,
                       minHeight: buttonShapes ? 44 : 30)
        }
    }
}
```

---

## 13. App Portability

### 13.1 Running iOS Apps บน visionOS

iOS และ iPadOS apps สามารถรันบน visionOS ได้ใน **Compatible Mode**:
- แสดงใน 2D window
- ไม่ใช้ spatial features
- Touch gestures แปลงเป็น eye + hand interactions

### 13.2 สิ่งที่ต้องปรับสำหรับ Native Experience

```swift
// ตรวจสอบว่าอยู่บน visionOS หรือไม่
#if os(visionOS)
    // Code เฉพาะ visionOS
    Text("Hello visionOS!")
        .glassBackgroundEffect()
#else
    // Code สำหรับ iOS/iPadOS
    Text("Hello iOS!")
        .background(.ultraThinMaterial)
#endif

// ตรวจสอบ compile-time
struct AdaptiveView: View {
    var body: some View {
        Group {
#if os(visionOS)
            VisionOSLayout()
#else
            StandardLayout()
#endif
        }
    }
}
```

### 13.3 Recommended Changes

**1. UIKit → SwiftUI:**
แนะนำให้ใช้ SwiftUI แทน UIKit เพราะ visionOS optimize สำหรับ SwiftUI

**2. Custom buttons → Standard buttons:**
Standard buttons มี hover effect อัตโนมัติ

**3. Touch gestures → Spatial gestures:**
```swift
// แทนที่ TapGesture แบบ iOS
.onTapGesture { }

// ใช้ SpatialTapGesture บน visionOS
.gesture(SpatialTapGesture().targetedToAnyEntity().onEnded { })
```

**4. 2D images → 3D content:**
```swift
// เพิ่ม 3D model แทน 2D image
Model3D(named: "ProductModel") { model in
    model.resizable().scaledToFit()
} placeholder: {
    Image("product_2d")
}
```

---

## 14. Complete visionOS App: Interactive 3D Product Viewer

### 14.1 App Structure

```swift
// ProductViewerApp.swift
import SwiftUI
import RealityKit

@main
struct ProductViewerApp: App {
    @StateObject private var appModel = AppModel()
    
    var body: some Scene {
        WindowGroup {
            ProductCatalogView()
                .environmentObject(appModel)
        }
        .defaultSize(width: 900, height: 650)
        
        WindowGroup(id: "product-detail", for: Product.ID.self) { $productID in
            if let id = productID,
               let product = appModel.product(id: id) {
                ProductDetailView(product: product)
                    .environmentObject(appModel)
            }
        }
        .windowStyle(.plain)
        
        WindowGroup(id: "product-3d", for: Product.ID.self) { $productID in
            if let id = productID,
               let product = appModel.product(id: id) {
                Product3DViewer(product: product)
                    .environmentObject(appModel)
            }
        }
        .windowStyle(.volumetric)
        .defaultSize(width: 0.6, height: 0.6, depth: 0.6, in: .meters)
        
        ImmersiveSpace(id: "product-showcase", for: Product.ID.self) { $productID in
            if let id = productID,
               let product = appModel.product(id: id) {
                ProductShowcaseImmersive(product: product)
                    .environmentObject(appModel)
            }
        }
        .immersionStyle(selection: .constant(.mixed), in: .mixed)
    }
}
```

### 14.2 Data Model

```swift
// Models.swift
import Foundation

struct Product: Identifiable, Hashable {
    let id: UUID
    var name: String
    var description: String
    var price: Double
    var modelName: String  // USDZ file name
    var thumbnailName: String
    var features: [String]
    var colorOptions: [ProductColor]
    
    struct ProductColor: Identifiable, Hashable {
        let id: UUID = UUID()
        var name: String
        var color: SIMD4<Float>  // RGBA
    }
}

@MainActor
class AppModel: ObservableObject {
    @Published var products: [Product] = Product.sampleProducts
    @Published var selectedProduct: Product?
    @Published var showingImmersive = false
    
    func product(id: Product.ID) -> Product? {
        products.first { $0.id == id }
    }
}

extension Product {
    static let sampleProducts: [Product] = [
        Product(
            id: UUID(),
            name: "AirPods Pro",
            description: "หูฟังไร้สายพร้อม Active Noise Cancellation",
            price: 9990,
            modelName: "AirPodsPro",
            thumbnailName: "airpods_thumb",
            features: ["ANC", "Transparency Mode", "Spatial Audio", "MagSafe"],
            colorOptions: [
                .init(name: "White", color: [1, 1, 1, 1]),
                .init(name: "Midnight", color: [0.1, 0.1, 0.1, 1])
            ]
        ),
        Product(
            id: UUID(),
            name: "Apple Watch Ultra 2",
            description: "นาฬิกาสำหรับนักผจญภัยขั้นสุด",
            price: 29900,
            modelName: "AppleWatchUltra",
            thumbnailName: "watch_thumb",
            features: ["Titanium", "100m water", "Action Button", "49mm"],
            colorOptions: [
                .init(name: "Natural Titanium", color: [0.8, 0.8, 0.75, 1])
            ]
        )
    ]
}
```

### 14.3 Catalog View

```swift
// ProductCatalogView.swift
import SwiftUI

struct ProductCatalogView: View {
    @EnvironmentObject var appModel: AppModel
    @Environment(\.openWindow) private var openWindow
    
    let columns = [GridItem(.adaptive(minimum: 250, maximum: 300))]
    
    var body: some View {
        NavigationStack {
            ScrollView {
                LazyVGrid(columns: columns, spacing: 20) {
                    ForEach(appModel.products) { product in
                        ProductCardView(product: product)
                            .onTapGesture {
                                openWindow(id: "product-detail", value: product.id)
                            }
                    }
                }
                .padding(20)
            }
            .navigationTitle("Product Viewer")
            .toolbar {
                ToolbarItem(placement: .topBarTrailing) {
                    Button("About") {
                        // เปิด about window
                    }
                }
            }
        }
    }
}

struct ProductCardView: View {
    let product: Product
    @State private var isHovered = false
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            // Product image/3D preview
            Model3D(named: product.modelName, bundle: realityKitContentBundle) { model in
                model
                    .resizable()
                    .scaledToFit()
                    .frame(height: 150)
            } placeholder: {
                Image(product.thumbnailName)
                    .resizable()
                    .scaledToFit()
                    .frame(height: 150)
            }
            
            VStack(alignment: .leading, spacing: 4) {
                Text(product.name)
                    .font(.headline)
                
                Text("฿\(product.price, format: .number)")
                    .font(.subheadline)
                    .foregroundStyle(.secondary)
            }
            .padding(.horizontal, 4)
        }
        .padding(16)
        .frame(maxWidth: .infinity)
        .glassBackgroundEffect()
        .scaleEffect(isHovered ? 1.02 : 1.0)
        .animation(.easeInOut(duration: 0.15), value: isHovered)
        .hoverEffect { effect, isActive, _ in
            effect.scaleEffect(isActive ? 1.02 : 1.0)
        }
    }
}
```

### 14.4 3D Viewer Component

```swift
// Product3DViewer.swift
import SwiftUI
import RealityKit

struct Product3DViewer: View {
    let product: Product
    @State private var selectedColorIndex = 0
    @State private var rotation: Double = 0
    @State private var isAutoRotating = true
    @State private var modelEntity: ModelEntity?
    
    var body: some View {
        VStack {
            RealityView { content, attachments in
                // โหลด 3D model
                if let entity = try? await ModelEntity(named: product.modelName) {
                    entity.position = [0, 0, 0]
                    entity.scale = [0.3, 0.3, 0.3]
                    
                    // เพิ่ม interaction
                    entity.components.set(CollisionComponent(
                        shapes: [.generateBox(size: [0.3, 0.3, 0.3])]
                    ))
                    entity.components.set(InputTargetComponent())
                    
                    content.add(entity)
                    modelEntity = entity
                }
                
                // Color selector attachment
                if let colorAttachment = attachments.entity(for: "colorSelector") {
                    colorAttachment.position = [0, -0.25, 0]
                    content.add(colorAttachment)
                }
                
            } update: { content, attachments in
                // อัปเดต material เมื่อ color เปลี่ยน
                if let entity = modelEntity,
                   let selectedColor = product.colorOptions[safe: selectedColorIndex] {
                    applyColor(selectedColor.color, to: entity)
                }
                
            } attachments: {
                Attachment(id: "colorSelector") {
                    ColorSelectorView(
                        colors: product.colorOptions,
                        selectedIndex: $selectedColorIndex
                    )
                }
            }
            // Drag gesture สำหรับ rotate
            .gesture(
                DragGesture()
                    .targetedToAnyEntity()
                    .onChanged { value in
                        isAutoRotating = false
                        modelEntity?.transform.rotation = simd_quatf(
                            angle: Float(value.translation.width) * 0.01,
                            axis: [0, 1, 0]
                        )
                    }
            )
        }
        .onAppear {
            // Auto rotation
            withAnimation(.linear(duration: 8).repeatForever(autoreverses: false)) {
                rotation = 360
            }
        }
    }
    
    func applyColor(_ color: SIMD4<Float>, to entity: ModelEntity) {
        guard var model = entity.components[ModelComponent.self] else { return }
        
        var material = PhysicallyBasedMaterial()
        material.baseColor = PhysicallyBasedMaterial.BaseColor(
            tint: .init(red: CGFloat(color.x),
                       green: CGFloat(color.y),
                       blue: CGFloat(color.z),
                       alpha: CGFloat(color.w))
        )
        material.metallic = .init(floatLiteral: 0.7)
        material.roughness = .init(floatLiteral: 0.3)
        
        model.materials = [material]
        entity.components.set(model)
    }
}

struct ColorSelectorView: View {
    let colors: [Product.ProductColor]
    @Binding var selectedIndex: Int
    
    var body: some View {
        HStack(spacing: 12) {
            ForEach(Array(colors.enumerated()), id: \.element.id) { index, color in
                Circle()
                    .fill(Color(
                        red: Double(color.color.x),
                        green: Double(color.color.y),
                        blue: Double(color.color.z)
                    ))
                    .frame(width: 30, height: 30)
                    .overlay(
                        Circle()
                            .stroke(Color.white, lineWidth: selectedIndex == index ? 3 : 0)
                    )
                    .scaleEffect(selectedIndex == index ? 1.2 : 1.0)
                    .animation(.spring(duration: 0.2), value: selectedIndex)
                    .onTapGesture {
                        selectedIndex = index
                    }
            }
        }
        .padding(12)
        .glassBackgroundEffect()
    }
}
```

### 14.5 Immersive Showcase

```swift
// ProductShowcaseImmersive.swift
import SwiftUI
import RealityKit
import ARKit

struct ProductShowcaseImmersive: View {
    let product: Product
    
    let session = ARKitSession()
    let worldTracking = WorldTrackingProvider()
    let handTracking = HandTrackingProvider()
    
    @State private var productEntity: ModelEntity?
    @State private var leftPinching = false
    @State private var rightPinching = false
    
    var body: some View {
        RealityView { content in
            // โหลด product model ขนาดใหญ่
            if let entity = try? await ModelEntity(named: product.modelName) {
                entity.scale = [1, 1, 1]  // ขนาดจริง
                entity.position = [0, 0, -1.5]
                
                entity.components.set(CollisionComponent(
                    shapes: [.generateBox(size: [0.5, 0.5, 0.5])]
                ))
                entity.components.set(InputTargetComponent())
                
                content.add(entity)
                productEntity = entity
            }
            
            // เพิ่ม atmospheric lighting
            let lightEntity = Entity()
            lightEntity.components.set(DirectionalLightComponent(
                color: .white,
                intensity: 1000,
                isRealWorldProxy: true  // ใช้แสงจากโลกจริง
            ))
            content.add(lightEntity)
            
        } update: { content in
            // อัปเดตตาม hand tracking state
        }
        .gesture(
            DragGesture()
                .targetedToAnyEntity()
                .onChanged { value in
                    guard let entity = productEntity else { return }
                    let translation = value.convert(value.gestureValue.translation3D, from: .local, to: .scene)
                    entity.position = entity.position + SIMD3<Float>(translation)
                }
        )
        .task {
            await startTracking()
        }
    }
    
    func startTracking() async {
        guard HandTrackingProvider.isSupported else { return }
        
        do {
            try await session.run([worldTracking, handTracking])
            
            for await update in handTracking.anchorUpdates {
                let anchor = update.anchor
                guard let skeleton = anchor.handSkeleton else { continue }
                
                // Detect pinch
                let thumbTip = skeleton.joint(.thumbTip)
                let indexTip = skeleton.joint(.indexFingerTip)
                
                let thumbPos = SIMD3<Float>(
                    thumbTip.anchorFromJointTransform.columns.3.x,
                    thumbTip.anchorFromJointTransform.columns.3.y,
                    thumbTip.anchorFromJointTransform.columns.3.z
                )
                
                let indexPos = SIMD3<Float>(
                    indexTip.anchorFromJointTransform.columns.3.x,
                    indexTip.anchorFromJointTransform.columns.3.y,
                    indexTip.anchorFromJointTransform.columns.3.z
                )
                
                let distance = length(thumbPos - indexPos)
                let isPinching = distance < 0.02
                
                await MainActor.run {
                    if anchor.chirality == .left {
                        leftPinching = isPinching
                    } else {
                        rightPinching = isPinching
                    }
                }
            }
        } catch {
            print("Tracking error: \(error)")
        }
    }
}
```

---

## 15. Exercises with Solutions

### Exercise 1: สร้าง Floating Menu

**โจทย์:** สร้าง floating menu ที่ลอยอยู่ใน space และมี hover effects

```swift
// Solution
struct FloatingMenu: View {
    @State private var selectedItem: String?
    let menuItems = ["Home", "Products", "Cart", "Profile"]
    
    var body: some View {
        VStack(spacing: 8) {
            ForEach(menuItems, id: \.self) { item in
                Button(item) {
                    selectedItem = item
                }
                .buttonStyle(.bordered)
                .tint(selectedItem == item ? .blue : .primary)
            }
        }
        .padding(16)
        .glassBackgroundEffect()
        .hoverEffect()
    }
}
```

### Exercise 2: 3D Loading Animation

**โจทย์:** สร้าง loading animation ด้วย RealityKit

```swift
// Solution
struct LoadingAnimation: View {
    @State private var isAnimating = false
    
    var body: some View {
        RealityView { content in
            // สร้าง rotating rings
            for i in 0..<3 {
                let ringEntity = ModelEntity(
                    mesh: .generateBox(size: [0.2, 0.01, 0.01]),
                    materials: [SimpleMaterial(
                        color: [.blue, .green, .purple][i],
                        isMetallic: true
                    )]
                )
                ringEntity.position = [0, Float(i) * 0.05 - 0.05, -0.5]
                ringEntity.name = "ring_\(i)"
                content.add(ringEntity)
            }
        }
        .onAppear {
            isAnimating = true
        }
    }
}
```

### Exercise 3: Hand-controlled Object

**โจทย์:** สร้าง object ที่เคลื่อนที่ตามมือ

```swift
// Solution - ดูตัวอย่างใน HandAttachedContent ด้านบน
// และลองต่อยอดด้วยการเพิ่ม:
// - Visual feedback เมื่อ pinch
// - Particle effect จากปลายนิ้ว
// - เสียงเมื่อ grab/release
```

### Exercise 4: Interactive Solar System

**โจทย์:** สร้าง solar system ใน immersive space ที่ผู้ใช้เดินสำรวจได้

```swift
struct SolarSystemImmersive: View {
    let planets: [(name: String, distance: Float, radius: Float, color: UIColor, speed: Float)] = [
        ("Mercury", 0.4, 0.04, .gray, 4.7),
        ("Venus", 0.7, 0.09, .orange, 3.5),
        ("Earth", 1.0, 0.1, .blue, 3.0),
        ("Mars", 1.5, 0.07, .red, 2.4),
        ("Jupiter", 2.5, 0.3, .brown, 1.3),
    ]
    
    var body: some View {
        RealityView { content in
            // Sun
            let sun = ModelEntity(
                mesh: .generateSphere(radius: 0.2),
                materials: [SimpleMaterial(color: .yellow, isMetallic: false)]
            )
            sun.position = [0, 0, -2]
            
            // เพิ่ม glow effect ให้ sun
            sun.components.set(PointLightComponent(
                color: .yellow,
                intensity: 5000,
                attenuationRadius: 5.0
            ))
            
            content.add(sun)
            
            // Planets
            for planet in planets {
                let planetEntity = ModelEntity(
                    mesh: .generateSphere(radius: planet.radius),
                    materials: [SimpleMaterial(color: planet.color, isMetallic: false)]
                )
                
                // วางตำแหน่งเริ่มต้น
                planetEntity.position = [planet.distance, 0, -2]
                planetEntity.name = planet.name
                
                content.add(planetEntity)
                
                // Animate orbit
                // (ใน production ใช้ RealityKit animations)
            }
        }
    }
}
```

---

## สรุปบทที่ 99

ในบทนี้เราได้เรียนรู้:

1. **visionOS Architecture** - ความแตกต่างจาก iOS และ Spatial Computing concepts
2. **Window Types** - Window, Volume, ImmersiveSpace และเมื่อควรใช้แต่ละแบบ
3. **SwiftUI บน visionOS** - glassBackgroundEffect, ornament, 3D transforms
4. **RealityKit** - RealityView, ECS, AnchorEntity, Materials
5. **3D Assets** - USDZ, Reality Composer Pro, Physics, Particles
6. **Hand Tracking** - HandAnchor, gesture recognition, custom gestures
7. **Eye Tracking** - SpatialTapGesture, hoverEffect, gaze interactions
8. **Spatial Audio** - SpatialAudioComponent, ambient audio, environment matching
9. **SharePlay** - GroupActivity สำหรับ shared spatial experiences
10. **ARKit** - World tracking, plane detection, image tracking, scene reconstruction
11. **Accessibility** - VoiceOver ใน spatial environment
12. **App Portability** - iOS apps บน visionOS และการ adapt

**บทถัดไป:** Part 100 - Course Conclusion and Mastery Reference

---

*หมายเหตุ: visionOS development ต้องการ Xcode 15+ และ macOS Sonoma 14+ สำหรับ visionOS Simulator ต้องการ Mac Pro หรือ Mac ที่มี Apple Silicon M2 Pro หรือสูงกว่า*
