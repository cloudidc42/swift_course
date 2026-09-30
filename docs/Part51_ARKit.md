# Part 51: ARKit และ Augmented Reality บน iOS

## บทนำ

Augmented Reality (AR) หรือความเป็นจริงเสริม คือเทคโนโลยีที่ผสานโลกดิจิทัลเข้ากับโลกจริงผ่านกล้องของอุปกรณ์ iOS Apple ได้พัฒนา ARKit framework ซึ่งเป็นเครื่องมือหลักสำหรับสร้างประสบการณ์ AR บน iPhone และ iPad บทนี้จะครอบคลุมทุกด้านของ ARKit ตั้งแต่พื้นฐานไปจนถึงการสร้างแอปพลิเคชัน AR ที่ซับซ้อน

---

## 1. What is ARKit?

ARKit คือ framework ของ Apple ที่ช่วยให้นักพัฒนาสร้างประสบการณ์ Augmented Reality บนอุปกรณ์ iOS โดยใช้กล้อง เซนเซอร์ และ processor ของอุปกรณ์ในการวิเคราะห์สภาพแวดล้อมจริงและวางวัตถุดิจิทัลลงไปในฉาก

### ความสามารถหลักของ ARKit

- **World Tracking**: ติดตามตำแหน่งและทิศทางของอุปกรณ์ใน 6 degrees of freedom (6DoF)
- **Plane Detection**: ตรวจจับพื้นผิวแนวนอนและแนวตั้ง
- **Image Recognition**: รู้จำรูปภาพที่กำหนดไว้ล่วงหน้า
- **Object Detection**: ตรวจจับและติดตาม 3D objects
- **Face Tracking**: ติดตามใบหน้าและแสดงผล AR บนใบหน้า
- **Body Tracking**: ติดตามการเคลื่อนไหวของร่างกาย
- **Light Estimation**: ประมาณค่าแสงในสภาพแวดล้อม
- **People Occlusion**: ทำให้วัตถุ AR ซ่อนหลังคนในฉาก

### ความต้องการของระบบ

```swift
// ARKit ต้องการ iOS 11.0 ขึ้นไป และ A9 chip ขึ้นไป
// ตรวจสอบว่าอุปกรณ์รองรับ ARKit หรือไม่
import ARKit

func checkARKitSupport() {
    if ARWorldTrackingConfiguration.isSupported {
        print("อุปกรณ์นี้รองรับ World Tracking AR")
    } else {
        print("อุปกรณ์นี้ไม่รองรับ ARKit")
    }
    
    if ARFaceTrackingConfiguration.isSupported {
        print("อุปกรณ์นี้รองรับ Face Tracking (ต้องการ TrueDepth camera)")
    }
    
    if ARBodyTrackingConfiguration.isSupported {
        print("อุปกรณ์นี้รองรับ Body Tracking")
    }
}
```

---

## 2. AR Experiences Overview

### ประเภทของประสบการณ์ AR

ARKit รองรับประสบการณ์ AR หลายประเภท:

#### 2.1 World-Scale AR
วางวัตถุ 3D ในโลกจริง เช่น เฟอร์นิเจอร์ AR

#### 2.2 Face AR
ใส่ filter หรือ effect บนใบหน้า เช่น Snapchat filters

#### 2.3 Image AR
แสดงเนื้อหา AR บนรูปภาพที่กำหนด เช่น โปสเตอร์ที่มีชีวิต

#### 2.4 Object AR
ติดตามและแสดงข้อมูลบนวัตถุ 3D จริง

### AR Pipeline

```swift
// AR Pipeline ทำงานดังนี้:
// 1. ARSession รับ frames จากกล้อง
// 2. ARConfiguration กำหนดประเภทของ tracking
// 3. ARFrame ประกอบด้วย captured image + tracking data
// 4. ARCamera ให้ข้อมูล pose และ intrinsics
// 5. ARAnchor กำหนดตำแหน่งในโลก AR
// 6. Renderer (SceneKit/RealityKit) แสดงผล

import ARKit
import RealityKit

class ARViewController: UIViewController {
    var arView: ARView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // สร้าง ARView
        arView = ARView(frame: view.bounds)
        view.addSubview(arView)
        
        // กำหนด configuration
        let config = ARWorldTrackingConfiguration()
        config.planeDetection = [.horizontal, .vertical]
        
        // เริ่ม session
        arView.session.run(config)
    }
}
```

---

## 3. ARSession

ARSession คือ object หลักที่จัดการกระบวนการทำงานของ AR ทั้งหมด มันประสานงานระหว่างกล้อง เซนเซอร์ และ tracking algorithms

### การสร้างและจัดการ ARSession

```swift
import ARKit

class ARSessionManager: NSObject, ARSessionDelegate {
    let session = ARSession()
    
    func setupSession() {
        session.delegate = self
        
        let configuration = ARWorldTrackingConfiguration()
        configuration.planeDetection = [.horizontal, .vertical]
        configuration.environmentTexturing = .automatic
        
        // Options สำหรับ run
        session.run(configuration, options: [
            .resetTracking,      // รีเซ็ต tracking
            .removeExistingAnchors // ลบ anchors เก่า
        ])
    }
    
    func pauseSession() {
        session.pause()
    }
    
    func resumeSession() {
        let configuration = ARWorldTrackingConfiguration()
        session.run(configuration)
    }
    
    // MARK: - ARSessionDelegate
    
    // เรียกทุกครั้งที่มี frame ใหม่
    func session(_ session: ARSession, didUpdate frame: ARFrame) {
        let camera = frame.camera
        let trackingState = camera.trackingState
        
        switch trackingState {
        case .normal:
            print("Tracking ปกติ")
        case .notAvailable:
            print("Tracking ไม่พร้อมใช้งาน")
        case .limited(let reason):
            switch reason {
            case .excessiveMotion:
                print("เคลื่อนไหวเร็วเกินไป")
            case .insufficientFeatures:
                print("ฉากไม่มี features เพียงพอ")
            case .initializing:
                print("กำลังเริ่มต้น tracking")
            case .relocalizing:
                print("กำลัง relocalize")
            @unknown default:
                print("สาเหตุไม่ทราบ")
            }
        }
    }
    
    // เรียกเมื่อ anchor ถูกเพิ่ม
    func session(_ session: ARSession, didAdd anchors: [ARAnchor]) {
        for anchor in anchors {
            print("เพิ่ม anchor: \(anchor.identifier)")
        }
    }
    
    // เรียกเมื่อ anchor ถูกอัพเดต
    func session(_ session: ARSession, didUpdate anchors: [ARAnchor]) {
        for anchor in anchors {
            if let planeAnchor = anchor as? ARPlaneAnchor {
                print("อัพเดต plane anchor: \(planeAnchor.classification)")
            }
        }
    }
    
    // เรียกเมื่อ anchor ถูกลบ
    func session(_ session: ARSession, didRemove anchors: [ARAnchor]) {
        for anchor in anchors {
            print("ลบ anchor: \(anchor.identifier)")
        }
    }
    
    // เรียกเมื่อเกิดข้อผิดพลาด
    func session(_ session: ARSession, didFailWithError error: Error) {
        guard let arError = error as? ARError else { return }
        
        switch arError.errorCode {
        case ARError.cameraUnauthorized.rawValue:
            print("ไม่ได้รับอนุญาตใช้กล้อง")
        case ARError.sensorFailed.rawValue:
            print("เซนเซอร์ล้มเหลว")
        default:
            print("เกิดข้อผิดพลาด: \(error.localizedDescription)")
        }
    }
    
    // เรียกเมื่อ session ถูก interrupted
    func sessionWasInterrupted(_ session: ARSession) {
        print("Session ถูก interrupt")
    }
    
    // เรียกเมื่อ session กลับมาทำงาน
    func sessionInterruptionEnded(_ session: ARSession) {
        print("Session กลับมาทำงาน")
        // รัน session ใหม่
        let config = ARWorldTrackingConfiguration()
        session.run(config, options: [.resetTracking, .removeExistingAnchors])
    }
}
```

### ARSession State Machine

```swift
// จัดการ session states อย่างถูกต้อง
class SessionStateManager {
    enum SessionState {
        case initializing
        case tracking
        case limited(ARCamera.TrackingState.Reason)
        case interrupted
        case failed(Error)
    }
    
    var currentState: SessionState = .initializing
    
    func updateState(from frame: ARFrame) {
        switch frame.camera.trackingState {
        case .normal:
            currentState = .tracking
        case .notAvailable:
            currentState = .initializing
        case .limited(let reason):
            currentState = .limited(reason)
        }
    }
    
    var userMessage: String {
        switch currentState {
        case .initializing:
            return "กำลังเริ่มต้น AR กรุณาเคลื่อนกล้องช้าๆ"
        case .tracking:
            return "AR พร้อมใช้งาน"
        case .limited(let reason):
            switch reason {
            case .excessiveMotion:
                return "เคลื่อนกล้องช้าๆ"
            case .insufficientFeatures:
                return "ชี้กล้องไปยังบริเวณที่มีลวดลายหรือรายละเอียดมากขึ้น"
            case .initializing:
                return "กำลังเริ่มต้น..."
            case .relocalizing:
                return "กำลังกู้คืนตำแหน่ง..."
            @unknown default:
                return "Tracking จำกัด"
            }
        case .interrupted:
            return "AR ถูก interrupt"
        case .failed(let error):
            return "เกิดข้อผิดพลาด: \(error.localizedDescription)"
        }
    }
}
```

---

## 4. ARConfiguration Types

ARKit มี configuration หลายประเภทสำหรับสถานการณ์ต่างๆ

### 4.1 ARWorldTrackingConfiguration

Configuration ที่ใช้บ่อยที่สุด ติดตามตำแหน่งอุปกรณ์ใน 6DoF

```swift
import ARKit

func setupWorldTracking() -> ARWorldTrackingConfiguration {
    let config = ARWorldTrackingConfiguration()
    
    // ตรวจจับระนาบ
    config.planeDetection = [.horizontal, .vertical]
    
    // สร้าง environment texture โดยอัตโนมัติ
    config.environmentTexturing = .automatic
    
    // เปิดใช้ LiDAR mesh reconstruction (iPhone 12 Pro ขึ้นไป)
    if ARWorldTrackingConfiguration.supportsSceneReconstruction(.mesh) {
        config.sceneReconstruction = .mesh
    }
    
    // เปิดใช้ People Occlusion
    if ARWorldTrackingConfiguration.supportsFrameSemantics(.personSegmentationWithDepth) {
        config.frameSemantics.insert(.personSegmentationWithDepth)
    }
    
    // กำหนด image tracking
    if let imageGroup = ARReferenceImage.referenceImages(
        inGroupNamed: "AR Resources",
        bundle: nil
    ) {
        config.detectionImages = imageGroup
        config.maximumNumberOfTrackedImages = 3
    }
    
    // กำหนด object detection
    if let objectGroup = ARReferenceObject.referenceObjects(
        inGroupNamed: "AR Objects",
        bundle: nil
    ) {
        config.detectionObjects = objectGroup
    }
    
    return config
}
```

### 4.2 ARBodyTrackingConfiguration

ติดตามการเคลื่อนไหวของร่างกาย

```swift
func setupBodyTracking() {
    guard ARBodyTrackingConfiguration.isSupported else {
        print("Body Tracking ไม่รองรับบนอุปกรณ์นี้")
        return
    }
    
    let config = ARBodyTrackingConfiguration()
    
    // ตัวอย่างการใช้งาน Body Tracking
    // session.run(config)
}

// ARSessionDelegate สำหรับ Body Tracking
extension ARViewController: ARSessionDelegate {
    func session(_ session: ARSession, didUpdate anchors: [ARAnchor]) {
        for anchor in anchors {
            guard let bodyAnchor = anchor as? ARBodyAnchor else { continue }
            
            // ดึงข้อมูล skeleton
            let skeleton = bodyAnchor.skeleton
            
            // ตรวจสอบ joint ต่างๆ
            if let leftHandJoint = skeleton.jointModelTransforms[
                skeleton.definition.index(for: .leftHand)!
            ] {
                print("ตำแหน่งมือซ้าย: \(leftHandJoint)")
            }
            
            // ตรวจสอบ tracking confidence
            let isTracked = bodyAnchor.isTracked
            print("Body tracked: \(isTracked)")
        }
    }
}
```

### 4.3 ARFaceTrackingConfiguration

ติดตามใบหน้าด้วย TrueDepth camera (กล้องหน้า iPhone X ขึ้นไป)

```swift
func setupFaceTracking() {
    guard ARFaceTrackingConfiguration.isSupported else {
        print("Face Tracking ต้องการ TrueDepth camera")
        return
    }
    
    let config = ARFaceTrackingConfiguration()
    config.maximumNumberOfTrackedFaces = ARFaceTrackingConfiguration.supportedNumberOfTrackedFaces
    
    // รองรับ body tracking พร้อมกัน (บางอุปกรณ์)
    if ARFaceTrackingConfiguration.supportsWorldTracking {
        config.isWorldTrackingEnabled = true
    }
}

// จัดการ Face Anchors
class FaceARDelegate: NSObject, ARSessionDelegate {
    func session(_ session: ARSession, didUpdate anchors: [ARAnchor]) {
        for anchor in anchors {
            guard let faceAnchor = anchor as? ARFaceAnchor else { continue }
            
            // ดึง blend shapes (facial expressions)
            let blendShapes = faceAnchor.blendShapes
            
            // ตรวจสอบการหลับตา
            let leftEyeBlink = blendShapes[.eyeBlinkLeft]?.floatValue ?? 0
            let rightEyeBlink = blendShapes[.eyeBlinkRight]?.floatValue ?? 0
            
            if leftEyeBlink > 0.9 && rightEyeBlink > 0.9 {
                print("หลับตาทั้งสองข้าง")
            }
            
            // ตรวจสอบการยิ้ม
            let mouthSmileLeft = blendShapes[.mouthSmileLeft]?.floatValue ?? 0
            let mouthSmileRight = blendShapes[.mouthSmileRight]?.floatValue ?? 0
            
            if mouthSmileLeft > 0.5 && mouthSmileRight > 0.5 {
                print("กำลังยิ้ม!")
            }
            
            // ดึง face geometry
            let geometry = faceAnchor.geometry
            let vertices = geometry.vertices
            print("จำนวน vertices บนใบหน้า: \(vertices.count)")
        }
    }
}
```

### 4.4 ARImageTrackingConfiguration

ติดตามรูปภาพ 2D โดยไม่ต้องใช้ World Tracking

```swift
func setupImageTracking() {
    guard let imageGroup = ARReferenceImage.referenceImages(
        inGroupNamed: "AR Resources",
        bundle: nil
    ) else {
        print("ไม่พบ reference images")
        return
    }
    
    let config = ARImageTrackingConfiguration()
    config.trackingImages = imageGroup
    config.maximumNumberOfTrackedImages = 5
    config.isAutoFocusEnabled = true
}

// จัดการ Image Anchors
class ImageTrackingDelegate: NSObject, ARSessionDelegate {
    func session(_ session: ARSession, didAdd anchors: [ARAnchor]) {
        for anchor in anchors {
            guard let imageAnchor = anchor as? ARImageAnchor else { continue }
            
            let imageName = imageAnchor.referenceImage.name ?? "ไม่ทราบชื่อ"
            print("พบรูปภาพ: \(imageName)")
            
            // ขนาดของรูปภาพในโลกจริง
            let physicalSize = imageAnchor.referenceImage.physicalSize
            print("ขนาด: \(physicalSize.width) x \(physicalSize.height) เมตร")
            
            // ตำแหน่งของรูปภาพ
            let transform = imageAnchor.transform
            print("Transform: \(transform)")
            
            // ตรวจสอบว่ากำลัง track อยู่หรือไม่
            if imageAnchor.isTracked {
                print("รูปภาพอยู่ในมุมมอง")
            }
        }
    }
}
```

### 4.5 ARObjectScanningConfiguration

สำหรับสแกนวัตถุ 3D เพื่อใช้ใน Object Detection

```swift
func setupObjectScanning() {
    guard ARObjectScanningConfiguration.isSupported else {
        print("Object Scanning ไม่รองรับบนอุปกรณ์นี้")
        return
    }
    
    let config = ARObjectScanningConfiguration()
    config.planeDetection = [.horizontal]
    config.isAutoFocusEnabled = true
    
    // ใช้สำหรับสแกนวัตถุเพื่อสร้าง ARReferenceObject
    // session.run(config)
}
```

---

## 5. ARAnchor Types

ARAnchor คือ object ที่กำหนดตำแหน่งและทิศทางในโลก AR

### 5.1 ARAnchor พื้นฐาน

```swift
import ARKit

// สร้าง anchor ที่ตำแหน่งที่กำหนด
func addCustomAnchor(session: ARSession, transform: simd_float4x4) {
    let anchor = ARAnchor(transform: transform)
    session.add(anchor: anchor)
}

// สร้าง anchor ด้วยชื่อ
func addNamedAnchor(session: ARSession, name: String, transform: simd_float4x4) {
    let anchor = ARAnchor(name: name, transform: transform)
    session.add(anchor: anchor)
}
```

### 5.2 ARPlaneAnchor

ตัวแทนของระนาบที่ตรวจจับได้ในโลกจริง

```swift
class PlaneAnchorHandler {
    func processPlanAnchor(_ planeAnchor: ARPlaneAnchor) {
        // ประเภทของระนาบ
        switch planeAnchor.alignment {
        case .horizontal:
            switch planeAnchor.classification {
            case .floor:
                print("พบพื้น")
            case .ceiling:
                print("พบเพดาน")
            case .table:
                print("พบโต๊ะ")
            case .seat:
                print("พบที่นั่ง")
            default:
                print("ระนาบแนวนอนอื่นๆ")
            }
        case .vertical:
            switch planeAnchor.classification {
            case .wall:
                print("พบผนัง")
            case .door:
                print("พบประตู")
            case .window:
                print("พบหน้าต่าง")
            default:
                print("ระนาบแนวตั้งอื่นๆ")
            }
        @unknown default:
            print("ประเภทไม่ทราบ")
        }
        
        // ขนาดและตำแหน่ง
        let extent = planeAnchor.planeExtent
        print("ขนาด: \(extent.width) x \(extent.height) เมตร")
        
        // Center ของระนาบ (relative to anchor)
        let center = planeAnchor.center
        print("Center: \(center)")
        
        // Geometry ของระนาบ
        let geometry = planeAnchor.geometry
        print("จำนวน boundary vertices: \(geometry.boundaryVertexCount)")
    }
}
```

### 5.3 ARImageAnchor

```swift
class ImageAnchorHandler {
    func processImageAnchor(_ imageAnchor: ARImageAnchor) {
        let refImage = imageAnchor.referenceImage
        
        print("ชื่อรูปภาพ: \(refImage.name ?? "ไม่มีชื่อ")")
        print("ขนาดจริง: \(refImage.physicalSize)")
        print("กำลัง track: \(imageAnchor.isTracked)")
        
        // Transform ของรูปภาพในโลก
        let worldTransform = imageAnchor.transform
        print("World transform: \(worldTransform)")
    }
}
```

### 5.4 ARObjectAnchor

```swift
class ObjectAnchorHandler {
    func processObjectAnchor(_ objectAnchor: ARObjectAnchor) {
        let refObject = objectAnchor.referenceObject
        
        print("ชื่อวัตถุ: \(refObject.name ?? "ไม่มีชื่อ")")
        print("Scale: \(refObject.scale)")
        
        // ขอบเขตของวัตถุ
        let extent = refObject.extent
        print("ขนาด: \(extent)")
    }
}
```

### 5.5 ARParticipantAnchor

สำหรับ Collaborative Sessions

```swift
func setupCollaborativeSession() -> ARWorldTrackingConfiguration {
    let config = ARWorldTrackingConfiguration()
    config.isCollaborationEnabled = true
    return config
}

// จัดการ Participant Anchors
class CollaborativeARDelegate: NSObject, ARSessionDelegate {
    func session(_ session: ARSession, didAdd anchors: [ARAnchor]) {
        for anchor in anchors {
            if let participantAnchor = anchor as? ARParticipantAnchor {
                print("ผู้เข้าร่วม AR ใหม่เชื่อมต่อแล้ว")
                let transform = participantAnchor.transform
                print("ตำแหน่งผู้เข้าร่วม: \(transform)")
            }
        }
    }
}
```

---

## 6. ARFrame

ARFrame คือ snapshot ของ session ณ เวลาหนึ่งๆ ประกอบด้วยข้อมูลทั้งหมดที่ต้องการ

```swift
import ARKit

class ARFrameAnalyzer {
    func analyzeFrame(_ frame: ARFrame) {
        // 1. Captured Image (กล้อง)
        let capturedImage = frame.capturedImage  // CVPixelBuffer
        let width = CVPixelBufferGetWidth(capturedImage)
        let height = CVPixelBufferGetHeight(capturedImage)
        print("ขนาด frame: \(width) x \(height)")
        
        // 2. Camera Information
        let camera = frame.camera
        let transform = camera.transform  // โลก relative to camera
        let projectionMatrix = camera.projectionMatrix(
            for: .portrait,
            viewportSize: CGSize(width: width, height: height),
            zNear: 0.001,
            zFar: 1000
        )
        print("Projection matrix: \(projectionMatrix)")
        
        // 3. Detected Planes
        let anchors = frame.anchors
        let planeAnchors = anchors.compactMap { $0 as? ARPlaneAnchor }
        print("จำนวน planes ที่ตรวจจับได้: \(planeAnchors.count)")
        
        // 4. Light Estimation
        if let lightEstimate = frame.lightEstimate {
            let ambientIntensity = lightEstimate.ambientIntensity
            let ambientColorTemperature = lightEstimate.ambientColorTemperature
            print("ความสว่าง: \(ambientIntensity) lumen")
            print("อุณหภูมิสี: \(ambientColorTemperature) K")
        }
        
        // 5. Depth Data (LiDAR devices)
        if let depthMap = frame.sceneDepth?.depthMap {
            print("มี depth map")
        }
        
        if let smoothedDepthMap = frame.smoothedSceneDepth?.depthMap {
            print("มี smoothed depth map")
        }
        
        // 6. Segmentation Buffer (People Occlusion)
        if let segmentationBuffer = frame.segmentationBuffer {
            print("มี segmentation buffer")
        }
        
        // 7. Timestamp
        let timestamp = frame.timestamp
        print("Timestamp: \(timestamp)")
        
        // 8. World Mapping Status
        switch frame.worldMappingStatus {
        case .notAvailable:
            print("World mapping ไม่พร้อม")
        case .limited:
            print("World mapping จำกัด")
        case .extending:
            print("World mapping กำลังขยาย")
        case .mapped:
            print("World mapping สมบูรณ์")
        @unknown default:
            break
        }
    }
    
    // แปลง 2D point บนหน้าจอเป็น 3D point
    func unproject(point: CGPoint, in frame: ARFrame, viewport: CGSize) -> simd_float3? {
        // ใช้ depth ที่ประมาณจาก plane
        let results = frame.raycastQuery(
            from: point,
            allowing: .estimatedPlane,
            alignment: .horizontal
        )
        
        return nil // ใช้ raycast แทน
    }
}
```

---

## 7. ARCamera

ARCamera ให้ข้อมูลเกี่ยวกับกล้องและ tracking state

```swift
import ARKit

class ARCameraManager {
    func processCameraInfo(_ camera: ARCamera) {
        // 1. Transform (position + orientation ของกล้องในโลก)
        let transform = camera.transform
        let position = simd_make_float3(transform.columns.3)
        print("ตำแหน่งกล้อง: \(position)")
        
        // 2. Euler Angles
        let eulerAngles = camera.eulerAngles
        print("Roll: \(eulerAngles.x), Pitch: \(eulerAngles.y), Yaw: \(eulerAngles.z)")
        
        // 3. Tracking State
        switch camera.trackingState {
        case .normal:
            print("Tracking ปกติ")
        case .notAvailable:
            print("Tracking ไม่พร้อม")
        case .limited(let reason):
            print("Tracking จำกัด: \(reason)")
        }
        
        // 4. Intrinsics (คุณสมบัติภายในของกล้อง)
        let intrinsics = camera.intrinsics
        // intrinsics คือ 3x3 matrix:
        // [fx,  0, cx]
        // [ 0, fy, cy]
        // [ 0,  0,  1]
        let focalLengthX = intrinsics[0][0]
        let focalLengthY = intrinsics[1][1]
        let principalPointX = intrinsics[2][0]
        let principalPointY = intrinsics[2][1]
        
        print("Focal length: \(focalLengthX), \(focalLengthY)")
        print("Principal point: \(principalPointX), \(principalPointY)")
        
        // 5. Image Resolution
        let imageResolution = camera.imageResolution
        print("ขนาด image: \(imageResolution)")
        
        // 6. Project 3D point to 2D screen
        let worldPoint = simd_float3(0, 0, -1)  // 1 เมตรข้างหน้า
        let screenPoint = camera.projectPoint(
            worldPoint,
            orientation: .portrait,
            viewportSize: CGSize(width: 375, height: 812)
        )
        print("3D point (\(worldPoint)) อยู่ที่ screen: \(screenPoint)")
    }
    
    // คำนวณ Field of View
    func calculateFOV(camera: ARCamera) {
        let intrinsics = camera.intrinsics
        let resolution = camera.imageResolution
        
        let fovX = 2 * atan(Float(resolution.width) / (2 * intrinsics[0][0]))
        let fovY = 2 * atan(Float(resolution.height) / (2 * intrinsics[1][1]))
        
        let fovXDegrees = fovX * 180 / Float.pi
        let fovYDegrees = fovY * 180 / Float.pi
        
        print("FOV: \(fovXDegrees)° x \(fovYDegrees)°")
    }
}
```

---

## 8. RealityKit Integration

RealityKit คือ high-level framework ที่ Apple พัฒนาขึ้นมาแทน SceneKit สำหรับ AR

### 8.1 ARView

```swift
import RealityKit
import ARKit

class RealityKitARViewController: UIViewController {
    var arView: ARView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupARView()
    }
    
    func setupARView() {
        // สร้าง ARView
        arView = ARView(frame: view.bounds, cameraMode: .ar, automaticallyConfigureSession: true)
        arView.autoresizingMask = [.flexibleWidth, .flexibleHeight]
        view.addSubview(arView)
        
        // กำหนด debug options
        arView.debugOptions = [.showFeaturePoints, .showWorldOrigin]
        
        // กำหนด render options
        arView.renderOptions = [.disableAREnvironmentLighting]
        
        // ตั้งค่า AR configuration
        let config = ARWorldTrackingConfiguration()
        config.planeDetection = [.horizontal, .vertical]
        config.environmentTexturing = .automatic
        arView.session.run(config)
        
        // เพิ่ม gesture recognizers
        let tapGesture = UITapGestureRecognizer(target: self, action: #selector(handleTap(_:)))
        arView.addGestureRecognizer(tapGesture)
    }
    
    @objc func handleTap(_ sender: UITapGestureRecognizer) {
        let tapLocation = sender.location(in: arView)
        
        // Raycast เพื่อหาตำแหน่งในโลก
        let results = arView.raycast(
            from: tapLocation,
            allowing: .estimatedPlane,
            alignment: .horizontal
        )
        
        if let firstResult = results.first {
            placeObject(at: firstResult)
        }
    }
    
    func placeObject(at result: ARRaycastResult) {
        // สร้าง ModelEntity
        let mesh = MeshResource.generateBox(size: 0.1)
        let material = SimpleMaterial(color: .blue, isMetallic: true)
        let modelEntity = ModelEntity(mesh: mesh, materials: [material])
        
        // สร้าง AnchorEntity จาก raycast result
        let anchor = AnchorEntity(world: result.worldTransform)
        anchor.addChild(modelEntity)
        
        // เพิ่มลงใน scene
        arView.scene.addAnchor(anchor)
    }
}
```

---

## 9. Entity and Component System (ECS)

RealityKit ใช้ Entity-Component System (ECS) ซึ่งเป็น architecture ที่ยืดหยุ่น

### 9.1 Entity

```swift
import RealityKit

// Entity คือ container สำหรับ components
class ECSExample {
    func demonstrateECS() {
        // สร้าง Entity พื้นฐาน
        let entity = Entity()
        entity.name = "MyEntity"
        
        // Entity ทุกตัวมี Transform component โดยอัตโนมัติ
        entity.transform.translation = SIMD3<Float>(0, 0.5, -1)
        entity.transform.rotation = simd_quatf(angle: .pi / 4, axis: SIMD3<Float>(0, 1, 0))
        entity.transform.scale = SIMD3<Float>(2, 2, 2)
        
        // ตรวจสอบว่ามี component หรือไม่
        if entity.components.has(ModelComponent.self) {
            print("มี ModelComponent")
        }
        
        // เพิ่ม custom component
        var customComponent = CustomComponent()
        customComponent.health = 100
        entity.components.set(customComponent)
        
        // ดึง component
        if let component = entity.components[CustomComponent.self] {
            print("Health: \(component.health)")
        }
        
        // ลบ component
        entity.components.remove(CustomComponent.self)
    }
    
    func demonstrateHierarchy() {
        let parent = Entity()
        let child = Entity()
        
        // เพิ่ม child
        parent.addChild(child)
        
        // child transform เป็น relative ต่อ parent
        child.position = SIMD3<Float>(0.5, 0, 0)
        
        // ดูทุก children
        for child in parent.children {
            print("Child: \(child.name)")
        }
        
        // หา entity ด้วยชื่อ
        if let found = parent.findEntity(named: "TargetEntity") {
            print("พบ: \(found.name)")
        }
        
        // ลบ child
        child.removeFromParent()
    }
}

// Custom Component
struct CustomComponent: Component {
    var health: Float = 100
    var speed: Float = 5
    var isActive: Bool = true
}
```

### 9.2 Components ที่สำคัญ

```swift
import RealityKit

class ComponentsExample {
    func demonstrateComponents() {
        let entity = ModelEntity(
            mesh: .generateSphere(radius: 0.1),
            materials: [SimpleMaterial()]
        )
        
        // 1. TransformComponent (มีอยู่แล้วในทุก Entity)
        entity.transform.translation = SIMD3<Float>(0, 1, -2)
        
        // 2. ModelComponent
        let mesh = MeshResource.generateBox(size: 0.2)
        let material = SimpleMaterial(color: .red, isMetallic: false)
        entity.model = ModelComponent(mesh: mesh, materials: [material])
        
        // 3. CollisionComponent
        let shape = ShapeResource.generateBox(size: SIMD3<Float>(0.2, 0.2, 0.2))
        entity.collision = CollisionComponent(shapes: [shape])
        
        // 4. PhysicsBodyComponent
        var physicsBody = PhysicsBodyComponent()
        physicsBody.mode = .dynamic  // .static, .kinematic, .dynamic
        physicsBody.massProperties.mass = 1.0
        entity.physicsBody = physicsBody
        
        // 5. PhysicsMotionComponent
        var motion = PhysicsMotionComponent()
        motion.linearVelocity = SIMD3<Float>(0, 5, 0)  // 던져 5 m/s ขึ้น
        entity.physicsMotion = motion
        
        // 6. SynchronizationComponent (สำหรับ multi-user AR)
        entity.synchronization = SynchronizationComponent()
        
        // 7. AnchoringComponent
        entity.anchoring = AnchoringComponent(.plane(
            .horizontal,
            classification: .floor,
            minimumBounds: SIMD2<Float>(0.2, 0.2)
        ))
    }
}
```

### 9.3 Systems

```swift
import RealityKit

// Custom System สำหรับ update ทุก frame
class RotationSystem: System {
    static let query = EntityQuery(where: .has(RotationComponent.self))
    
    required init(scene: Scene) {}
    
    func update(context: SceneUpdateContext) {
        let deltaTime = Float(context.deltaTime)
        
        for entity in context.entities(matching: Self.query, updatingSystemWhen: .rendering) {
            if let rotComponent = entity.components[RotationComponent.self] {
                let angle = rotComponent.speed * deltaTime
                let rotation = simd_quatf(angle: angle, axis: SIMD3<Float>(0, 1, 0))
                entity.transform.rotation *= rotation
            }
        }
    }
}

struct RotationComponent: Component {
    var speed: Float = 1.0  // radians per second
}

// ลงทะเบียน System
// ต้องทำก่อนใช้งาน
func registerSystems() {
    RotationSystem.registerSystem()
    RotationComponent.registerComponent()
}
```

---

## 10. ModelEntity

ModelEntity คือ Entity ที่มีรูปร่างและวัสดุที่มองเห็นได้

```swift
import RealityKit

class ModelEntityExamples {
    // สร้าง primitive shapes
    func createPrimitives() {
        // Box
        let box = ModelEntity(
            mesh: .generateBox(size: 0.1),
            materials: [SimpleMaterial(color: .blue, isMetallic: false)]
        )
        
        // Sphere
        let sphere = ModelEntity(
            mesh: .generateSphere(radius: 0.05),
            materials: [SimpleMaterial(color: .red, roughness: 0.5, isMetallic: true)]
        )
        
        // Cylinder
        let cylinder = ModelEntity(
            mesh: .generateCylinder(height: 0.2, radius: 0.05),
            materials: [SimpleMaterial(color: .green, isMetallic: false)]
        )
        
        // Plane
        let plane = ModelEntity(
            mesh: .generatePlane(width: 1.0, depth: 1.0),
            materials: [SimpleMaterial(color: .white, isMetallic: false)]
        )
        
        // Text
        let textMesh = MeshResource.generateText(
            "สวัสดี AR!",
            extrusionDepth: 0.01,
            font: .systemFont(ofSize: 0.1),
            containerFrame: .zero,
            alignment: .center,
            lineBreakMode: .byCharWrapping
        )
        let textEntity = ModelEntity(
            mesh: textMesh,
            materials: [SimpleMaterial(color: .white, isMetallic: false)]
        )
    }
    
    // โหลด 3D model จากไฟล์
    func loadModelFromFile() async throws {
        // โหลด .usdz หรือ .reality file
        let modelEntity = try await ModelEntity.loadModel(named: "toy_robot")
        
        // กำหนด scale
        modelEntity.scale = SIMD3<Float>(repeating: 0.001)
        
        // เพิ่ม animation
        if let animation = modelEntity.availableAnimations.first {
            modelEntity.playAnimation(animation.repeat(duration: .infinity))
        }
    }
    
    // สร้าง materials ต่างๆ
    func createMaterials() {
        var simpleMaterial = SimpleMaterial()
        simpleMaterial.color = .init(tint: .blue, texture: nil)
        simpleMaterial.roughness = .float(0.5)
        simpleMaterial.metallic = .float(0.8)
        
        // Unlit Material (ไม่ตอบสนองต่อแสง)
        var unlitMaterial = UnlitMaterial()
        unlitMaterial.color = .init(tint: .red)
        
        // Physically Based Material
        var pbr = PhysicallyBasedMaterial()
        pbr.baseColor = .init(tint: .orange)
        pbr.roughness = .init(floatLiteral: 0.3)
        pbr.metallic = .init(floatLiteral: 0.7)
        pbr.emissiveColor = .init(color: .yellow)
        pbr.emissiveIntensity = 0.5
    }
    
    // Gesture สำหรับ ModelEntity
    func addGesturesToEntity(_ entity: ModelEntity, in arView: ARView) {
        // ต้องมี CollisionComponent ก่อน
        entity.generateCollisionShapes(recursive: true)
        
        // Translation gesture (ลาก)
        arView.installGestures(.translation, for: entity)
        
        // Rotation gesture (หมุน)
        arView.installGestures(.rotation, for: entity)
        
        // Scale gesture (ขยาย/ย่อ)
        arView.installGestures(.scale, for: entity)
        
        // ทุก gesture รวมกัน
        arView.installGestures(.all, for: entity)
    }
}
```

---

## 11. AnchorEntity

AnchorEntity เชื่อมต่อ virtual objects กับโลกจริง

```swift
import RealityKit
import ARKit

class AnchorEntityExamples {
    // 1. Anchor ที่ตำแหน่ง world space
    func createWorldAnchor(transform: simd_float4x4) -> AnchorEntity {
        let anchor = AnchorEntity(world: transform)
        let box = ModelEntity(mesh: .generateBox(size: 0.1), materials: [SimpleMaterial()])
        anchor.addChild(box)
        return anchor
    }
    
    // 2. Anchor บน plane ที่ตรวจจับได้
    func createPlaneAnchor() -> AnchorEntity {
        let anchor = AnchorEntity(
            plane: .horizontal,
            classification: .floor,
            minimumBounds: SIMD2<Float>(0.3, 0.3)
        )
        return anchor
    }
    
    // 3. Anchor บนรูปภาพ
    func createImageAnchor(imageName: String) -> AnchorEntity {
        let anchor = AnchorEntity(.image(group: "AR Resources", name: imageName))
        return anchor
    }
    
    // 4. Anchor บนกล้อง (ตามกล้องไปตลอด)
    func createCameraAnchor() -> AnchorEntity {
        let anchor = AnchorEntity(.camera)
        return anchor
    }
    
    // 5. Anchor จาก ARPlaneAnchor ที่มีอยู่แล้ว
    func createAnchorFromExisting(planeAnchor: ARPlaneAnchor, in arView: ARView) -> AnchorEntity {
        let anchor = AnchorEntity(anchor: planeAnchor)
        return anchor
    }
    
    // 6. Anchor สำหรับ Face
    func createFaceAnchor() -> AnchorEntity {
        let anchor = AnchorEntity(.face)
        return anchor
    }
}
```

---

## 12. RealityView (SwiftUI)

RealityView เป็น SwiftUI view สำหรับแสดง RealityKit content

```swift
import SwiftUI
import RealityKit
import ARKit

struct ARContentView: View {
    @State private var arContent = ARContent()
    
    var body: some View {
        RealityView { content in
            // ตั้งค่า scene ตอนแรก
            await arContent.setupScene(content: content)
        } update: { content in
            // อัพเดตเมื่อ @State เปลี่ยน
            arContent.updateScene(content: content)
        }
        .onTapGesture { location in
            // จัดการ tap gesture
            print("Tapped at: \(location)")
        }
    }
}

// สำหรับ visionOS หรือ immersive apps
struct ImmersiveARView: View {
    var body: some View {
        RealityView { content in
            // เพิ่ม content ลงใน scene
            let box = ModelEntity(
                mesh: .generateBox(size: 0.3),
                materials: [SimpleMaterial(color: .blue, isMetallic: true)]
            )
            box.position = SIMD3<Float>(0, 1.5, -2)
            
            let anchor = AnchorEntity(world: .zero)
            anchor.addChild(box)
            content.add(anchor)
        }
        .edgesIgnoringSafeArea(.all)
    }
}

class ARContent: ObservableObject {
    var entities: [Entity] = []
    
    func setupScene(content: RealityViewContent) async {
        // โหลด model
        do {
            let model = try await ModelEntity.loadModel(named: "toy_car")
            let anchor = AnchorEntity(plane: .horizontal)
            anchor.addChild(model)
            content.add(anchor)
            entities.append(model)
        } catch {
            print("ไม่สามารถโหลด model: \(error)")
        }
    }
    
    func updateScene(content: RealityViewContent) {
        // อัพเดต content ตาม state
    }
}

// SwiftUI AR View พร้อม overlay
struct ARViewWithUI: View {
    @State private var placedCount = 0
    @State private var showInfo = false
    
    var body: some View {
        ZStack {
            RealityView { content in
                // Setup
            }
            .edgesIgnoringSafeArea(.all)
            
            // Overlay UI
            VStack {
                if showInfo {
                    Text("วางวัตถุแล้ว: \(placedCount) ชิ้น")
                        .padding()
                        .background(Color.black.opacity(0.7))
                        .foregroundColor(.white)
                        .cornerRadius(10)
                }
                
                Spacer()
                
                HStack {
                    Button("แสดงข้อมูล") {
                        showInfo.toggle()
                    }
                    .buttonStyle(.borderedProminent)
                    
                    Button("ล้าง") {
                        placedCount = 0
                    }
                    .buttonStyle(.bordered)
                }
                .padding()
            }
        }
    }
}
```

---

## 13. SCNScene with ARKit (SceneKit)

SceneKit เป็น 3D framework รุ่นเก่าที่ยังสามารถใช้กับ ARKit ได้

```swift
import ARKit
import SceneKit

class SceneKitARViewController: UIViewController, ARSCNViewDelegate {
    @IBOutlet weak var sceneView: ARSCNView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupSceneKitAR()
    }
    
    func setupSceneKitAR() {
        // กำหนด delegate
        sceneView.delegate = self
        
        // แสดง statistics (FPS, etc.)
        sceneView.showsStatistics = true
        
        // สร้าง scene เปล่า
        let scene = SCNScene()
        sceneView.scene = scene
        
        // เพิ่ม default lighting
        sceneView.autoenablesDefaultLighting = true
        
        // Debug options
        sceneView.debugOptions = [
            .showBoundingBoxes,
            .showFeaturePoints,
            .showWorldOrigin
        ]
        
        // กำหนด AR configuration
        let configuration = ARWorldTrackingConfiguration()
        configuration.planeDetection = [.horizontal]
        sceneView.session.run(configuration)
    }
    
    // MARK: - ARSCNViewDelegate
    
    // เรียกเมื่อมี anchor ใหม่
    func renderer(_ renderer: SCNSceneRenderer, didAdd node: SCNNode, for anchor: ARAnchor) {
        guard let planeAnchor = anchor as? ARPlaneAnchor else { return }
        
        DispatchQueue.main.async {
            let planeNode = self.createPlaneNode(for: planeAnchor)
            node.addChildNode(planeNode)
        }
    }
    
    // เรียกเมื่อ anchor อัพเดต
    func renderer(_ renderer: SCNSceneRenderer, didUpdate node: SCNNode, for anchor: ARAnchor) {
        guard let planeAnchor = anchor as? ARPlaneAnchor,
              let planeNode = node.childNodes.first,
              let plane = planeNode.geometry as? SCNPlane else { return }
        
        DispatchQueue.main.async {
            // อัพเดตขนาด plane
            plane.width = CGFloat(planeAnchor.planeExtent.width)
            plane.height = CGFloat(planeAnchor.planeExtent.height)
            planeNode.position = SCNVector3(
                planeAnchor.center.x,
                0,
                planeAnchor.center.z
            )
        }
    }
    
    // สร้าง SCNNode สำหรับ plane
    func createPlaneNode(for planeAnchor: ARPlaneAnchor) -> SCNNode {
        let plane = SCNPlane(
            width: CGFloat(planeAnchor.planeExtent.width),
            height: CGFloat(planeAnchor.planeExtent.height)
        )
        
        // สร้าง material ที่โปร่งแสง
        let material = SCNMaterial()
        material.diffuse.contents = UIColor.blue.withAlphaComponent(0.3)
        material.isDoubleSided = true
        plane.materials = [material]
        
        let planeNode = SCNNode(geometry: plane)
        planeNode.position = SCNVector3(
            planeAnchor.center.x,
            0,
            planeAnchor.center.z
        )
        // หมุน plane ให้นอนราบ
        planeNode.eulerAngles.x = -.pi / 2
        
        return planeNode
    }
    
    // เพิ่มวัตถุ 3D ที่ตำแหน่งที่แตะ
    @IBAction func handleTap(_ sender: UITapGestureRecognizer) {
        let tapLocation = sender.location(in: sceneView)
        
        // Hit test กับ plane anchors
        let hitResults = sceneView.hitTest(tapLocation, types: .existingPlaneUsingExtent)
        
        if let hitResult = hitResults.first {
            let box = SCNBox(width: 0.1, height: 0.1, length: 0.1, chamferRadius: 0.01)
            let material = SCNMaterial()
            material.diffuse.contents = UIColor.red
            box.materials = [material]
            
            let boxNode = SCNNode(geometry: box)
            boxNode.position = SCNVector3(
                hitResult.worldTransform.columns.3.x,
                hitResult.worldTransform.columns.3.y + 0.05,
                hitResult.worldTransform.columns.3.z
            )
            
            sceneView.scene.rootNode.addChildNode(boxNode)
        }
    }
    
    // โหลด 3D model
    func load3DModel(named name: String) -> SCNNode? {
        guard let scene = SCNScene(named: "\(name).scn") else { return nil }
        let node = SCNNode()
        for child in scene.rootNode.childNodes {
            node.addChildNode(child)
        }
        return node
    }
    
    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        sceneView.session.pause()
    }
}
```

---

## 14. Hit Testing

Hit Testing ใช้ตรวจสอบว่า ray จากหน้าจอชนกับอะไรในโลก AR

```swift
import ARKit
import SceneKit

class HitTestingExamples {
    // SceneKit Hit Testing
    func sceneKitHitTest(in sceneView: ARSCNView, at point: CGPoint) {
        // 1. Hit test กับ SCNNode ที่มีอยู่ในฉาก
        let nodeResults = sceneView.hitTest(point, options: [
            .backFaceCulling: false,
            .boundingBoxOnly: false,
            .clipToZRange: false
        ])
        
        if let firstNode = nodeResults.first {
            print("ชน node: \(firstNode.node.name ?? "ไม่มีชื่อ")")
            print("ตำแหน่ง: \(firstNode.worldCoordinates)")
        }
        
        // 2. AR Hit Test (legacy - ใช้ raycast แทน)
        let arResults = sceneView.hitTest(point, types: [
            .featurePoint,
            .estimatedHorizontalPlane,
            .existingPlaneUsingExtent,
            .existingPlaneUsingGeometry
        ])
        
        if let firstAR = arResults.first {
            print("AR hit transform: \(firstAR.worldTransform)")
        }
    }
    
    // RealityKit Hit Testing
    func realityKitHitTest(in arView: ARView, at point: CGPoint) {
        // 1. Hit test กับ entities ที่มี CollisionComponent
        let hitEntities = arView.hitTest(point)
        
        if let firstHit = hitEntities.first {
            print("ชน entity: \(firstHit.entity.name)")
            print("ตำแหน่ง: \(firstHit.position)")
            print("ระยะห่าง: \(firstHit.distance)")
        }
    }
}
```

---

## 15. Raycasting

Raycasting คือวิธีที่ Apple แนะนำแทน Hit Testing สำหรับการหาตำแหน่งในโลก AR

```swift
import ARKit

class RaycastingExamples {
    // การ raycast พื้นฐาน
    func performRaycast(in arView: ARView, from screenPoint: CGPoint) {
        // สร้าง raycast query
        guard let raycastQuery = arView.makeRaycastQuery(
            from: screenPoint,
            allowing: .estimatedPlane,  // อนุญาตให้ชนกับ estimated planes
            alignment: .any             // ทั้ง horizontal และ vertical
        ) else { return }
        
        // ทำ raycast
        let results = arView.session.raycast(raycastQuery)
        
        if let firstResult = results.first {
            print("ชน: \(firstResult.anchor?.debugDescription ?? "ไม่มี anchor")")
            print("Transform: \(firstResult.worldTransform)")
        }
    }
    
    // ประเภทของ raycast targets
    func raycastTypes(in arView: ARView, screenPoint: CGPoint) {
        // 1. Existing Planes Only
        let exactQuery = arView.makeRaycastQuery(
            from: screenPoint,
            allowing: .existingPlaneGeometry,
            alignment: .horizontal
        )
        
        // 2. Estimated Planes (แนะนำ)
        let estimatedQuery = arView.makeRaycastQuery(
            from: screenPoint,
            allowing: .estimatedPlane,
            alignment: .any
        )
        
        // 3. Infinite Plane (ขยาย plane ออกไปไม่สิ้นสุด)
        let infiniteQuery = arView.makeRaycastQuery(
            from: screenPoint,
            allowing: .existingPlaneInfinite,
            alignment: .horizontal
        )
    }
    
    // Tracked Raycasting (ติดตามตลอดเวลา)
    func performTrackedRaycast(in arView: ARView, screenPoint: CGPoint) {
        guard let query = arView.makeRaycastQuery(
            from: screenPoint,
            allowing: .estimatedPlane,
            alignment: .horizontal
        ) else { return }
        
        // สร้าง tracked raycast ที่อัพเดตอัตโนมัติ
        let trackedRaycast = arView.session.trackedRaycast(query) { results in
            if let firstResult = results.first {
                // ตำแหน่งอัพเดตทุกครั้งที่มีข้อมูลใหม่
                print("อัพเดตตำแหน่ง: \(firstResult.worldTransform)")
            }
        }
        
        // หยุด tracked raycast เมื่อไม่ต้องการ
        // trackedRaycast?.stopTracking()
    }
    
    // Raycast จาก camera direction
    func raycastFromCamera(session: ARSession) {
        guard let currentFrame = session.currentFrame else { return }
        
        let camera = currentFrame.camera
        let cameraTransform = camera.transform
        
        // Ray origin (กล้อง)
        let rayOrigin = SIMD3<Float>(
            cameraTransform.columns.3.x,
            cameraTransform.columns.3.y,
            cameraTransform.columns.3.z
        )
        
        // Ray direction (ตรงหน้ากล้อง)
        let rayDirection = -SIMD3<Float>(
            cameraTransform.columns.2.x,
            cameraTransform.columns.2.y,
            cameraTransform.columns.2.z
        )
        
        print("Ray from: \(rayOrigin)")
        print("Ray direction: \(rayDirection)")
    }
}
```

---

## 16. Plane Detection

Plane Detection คือความสามารถในการตรวจจับพื้นผิวแนวนอนและแนวตั้ง

```swift
import ARKit
import RealityKit

class PlaneDetectionManager: NSObject {
    var arView: ARView!
    var detectedPlanes: [UUID: AnchorEntity] = [:]
    
    func setupPlaneDetection() {
        let config = ARWorldTrackingConfiguration()
        config.planeDetection = [.horizontal, .vertical]
        
        // เพิ่ม scene reconstruction สำหรับ LiDAR
        if ARWorldTrackingConfiguration.supportsSceneReconstruction(.meshWithClassification) {
            config.sceneReconstruction = .meshWithClassification
        }
        
        arView.session.run(config)
        arView.session.delegate = self
    }
    
    func visualizePlane(_ planeAnchor: ARPlaneAnchor) -> ModelEntity {
        let width = planeAnchor.planeExtent.width
        let height = planeAnchor.planeExtent.height
        
        // สร้าง plane mesh
        let mesh = MeshResource.generatePlane(width: width, depth: height)
        
        // สร้าง material ตาม classification
        var color: UIColor
        switch planeAnchor.classification {
        case .floor:
            color = UIColor.blue.withAlphaComponent(0.3)
        case .ceiling:
            color = UIColor.green.withAlphaComponent(0.3)
        case .wall:
            color = UIColor.red.withAlphaComponent(0.3)
        case .table:
            color = UIColor.orange.withAlphaComponent(0.3)
        default:
            color = UIColor.white.withAlphaComponent(0.3)
        }
        
        var material = UnlitMaterial()
        material.color = .init(tint: color)
        
        return ModelEntity(mesh: mesh, materials: [material])
    }
}

extension PlaneDetectionManager: ARSessionDelegate {
    func session(_ session: ARSession, didAdd anchors: [ARAnchor]) {
        for anchor in anchors {
            guard let planeAnchor = anchor as? ARPlaneAnchor else { continue }
            
            let planeEntity = visualizePlane(planeAnchor)
            let anchorEntity = AnchorEntity(anchor: planeAnchor)
            anchorEntity.addChild(planeEntity)
            
            arView.scene.addAnchor(anchorEntity)
            detectedPlanes[planeAnchor.identifier] = anchorEntity
            
            print("พบ plane: \(planeAnchor.classification)")
        }
    }
    
    func session(_ session: ARSession, didUpdate anchors: [ARAnchor]) {
        for anchor in anchors {
            guard let planeAnchor = anchor as? ARPlaneAnchor,
                  let anchorEntity = detectedPlanes[planeAnchor.identifier] else { continue }
            
            // อัพเดต plane visualization
            anchorEntity.children.removeAll()
            let updatedPlane = visualizePlane(planeAnchor)
            anchorEntity.addChild(updatedPlane)
        }
    }
    
    func session(_ session: ARSession, didRemove anchors: [ARAnchor]) {
        for anchor in anchors {
            guard let planeAnchor = anchor as? ARPlaneAnchor,
                  let anchorEntity = detectedPlanes[planeAnchor.identifier] else { continue }
            
            anchorEntity.removeFromParent()
            detectedPlanes.removeValue(forKey: planeAnchor.identifier)
        }
    }
}
```

---

## 17. Object Placement in AR

การวางวัตถุ 3D ในโลก AR อย่างถูกต้อง

```swift
import ARKit
import RealityKit

class ObjectPlacementManager {
    var arView: ARView!
    var placedObjects: [AnchorEntity] = []
    
    // วางวัตถุบน plane
    func placeObjectOnPlane(at screenPoint: CGPoint) {
        guard let query = arView.makeRaycastQuery(
            from: screenPoint,
            allowing: .estimatedPlane,
            alignment: .horizontal
        ) else { return }
        
        let results = arView.session.raycast(query)
        guard let firstResult = results.first else {
            print("ไม่พบ plane")
            return
        }
        
        // สร้างวัตถุ
        let object = createARObject()
        
        // กำหนด transform จาก raycast result
        let anchor = AnchorEntity(world: firstResult.worldTransform)
        
        // ยกวัตถุขึ้นเล็กน้อยเพื่อไม่ให้ทะลุพื้น
        object.position.y = 0.05
        
        anchor.addChild(object)
        arView.scene.addAnchor(anchor)
        placedObjects.append(anchor)
    }
    
    func createARObject() -> ModelEntity {
        // โหลดหรือสร้าง 3D model
        let mesh = MeshResource.generateBox(
            width: 0.1,
            height: 0.1,
            depth: 0.1,
            cornerRadius: 0.01
        )
        
        var material = PhysicallyBasedMaterial()
        material.baseColor = .init(tint: .systemBlue)
        material.roughness = .init(floatLiteral: 0.3)
        material.metallic = .init(floatLiteral: 0.8)
        
        let entity = ModelEntity(mesh: mesh, materials: [material])
        entity.name = "PlacedObject"
        
        // เพิ่ม collision เพื่อรองรับ gesture
        entity.generateCollisionShapes(recursive: true)
        arView.installGestures(.all, for: entity)
        
        // เพิ่ม animation เมื่อวาง
        let appearAnimation = FromToByAnimation<Transform>(
            from: Transform(scale: SIMD3<Float>(0, 0, 0)),
            to: Transform(scale: SIMD3<Float>(1, 1, 1)),
            duration: 0.3,
            timing: .easeOut,
            bindTarget: .transform
        )
        
        if let animResource = try? AnimationResource.generate(with: appearAnimation) {
            entity.playAnimation(animResource)
        }
        
        return entity
    }
    
    // ลบวัตถุทั้งหมด
    func clearAllObjects() {
        for anchor in placedObjects {
            anchor.removeFromParent()
        }
        placedObjects.removeAll()
    }
    
    // ลบวัตถุตัวล่าสุด
    func undoLastPlacement() {
        if let lastAnchor = placedObjects.last {
            lastAnchor.removeFromParent()
            placedObjects.removeLast()
        }
    }
    
    // วางวัตถุที่กำหนดไว้ล่วงหน้า
    func placeObjectAtAbsolutePosition(x: Float, y: Float, z: Float) {
        let transform = Transform(
            scale: .one,
            rotation: simd_quatf(angle: 0, axis: SIMD3<Float>(0, 1, 0)),
            translation: SIMD3<Float>(x, y, z)
        )
        
        let anchor = AnchorEntity(world: transform.matrix)
        let object = createARObject()
        anchor.addChild(object)
        arView.scene.addAnchor(anchor)
    }
}
```

---

## 18. Image Recognition in AR

การรู้จำและติดตามรูปภาพในโลกจริง

```swift
import ARKit
import RealityKit

class ImageRecognitionManager: NSObject {
    var arView: ARView!
    
    func setupImageTracking() {
        // โหลด reference images จาก asset catalog
        guard let referenceImages = ARReferenceImage.referenceImages(
            inGroupNamed: "AR Resources",
            bundle: .main
        ) else {
            print("ไม่พบ reference images")
            return
        }
        
        let config = ARWorldTrackingConfiguration()
        config.detectionImages = referenceImages
        config.maximumNumberOfTrackedImages = 3
        
        arView.session.run(config)
        arView.session.delegate = self
    }
    
    // สร้าง ARReferenceImage แบบ programmatic
    func createReferenceImageFromURL(url: URL) async throws -> ARReferenceImage? {
        let (data, _) = try await URLSession.shared.data(from: url)
        guard let image = UIImage(data: data),
              let cgImage = image.cgImage else { return nil }
        
        let referenceImage = ARReferenceImage(
            cgImage,
            orientation: .up,
            physicalWidth: 0.1  // ขนาดจริง 10 cm
        )
        referenceImage.name = "downloaded_image"
        
        return referenceImage
    }
    
    func placeContentOnImage(_ imageAnchor: ARImageAnchor) -> AnchorEntity {
        let refImage = imageAnchor.referenceImage
        let width = Float(refImage.physicalSize.width)
        let height = Float(refImage.physicalSize.height)
        
        // สร้าง anchor จาก image anchor
        let anchorEntity = AnchorEntity(anchor: imageAnchor)
        
        // สร้าง content ที่จะแสดงบนรูปภาพ
        // Video plane บนรูปภาพ
        let videoPlane = ModelEntity(
            mesh: .generatePlane(width: width, depth: height),
            materials: [SimpleMaterial(color: .blue.withAlphaComponent(0.5), isMetallic: false)]
        )
        videoPlane.position.y = 0.001  // ยกขึ้นเล็กน้อยเหนือรูปภาพ
        
        // 3D model เหนือรูปภาพ
        let model = ModelEntity(
            mesh: .generateSphere(radius: min(width, height) * 0.3),
            materials: [SimpleMaterial(color: .red, isMetallic: true)]
        )
        model.position.y = 0.1  // ลอยเหนือรูปภาพ
        
        anchorEntity.addChild(videoPlane)
        anchorEntity.addChild(model)
        
        return anchorEntity
    }
}

extension ImageRecognitionManager: ARSessionDelegate {
    func session(_ session: ARSession, didAdd anchors: [ARAnchor]) {
        for anchor in anchors {
            guard let imageAnchor = anchor as? ARImageAnchor else { continue }
            
            let imageName = imageAnchor.referenceImage.name ?? "ไม่ทราบ"
            print("พบรูปภาพ: \(imageName)")
            
            DispatchQueue.main.async {
                let contentAnchor = self.placeContentOnImage(imageAnchor)
                self.arView.scene.addAnchor(contentAnchor)
            }
        }
    }
    
    func session(_ session: ARSession, didUpdate anchors: [ARAnchor]) {
        for anchor in anchors {
            guard let imageAnchor = anchor as? ARImageAnchor else { continue }
            
            if imageAnchor.isTracked {
                print("รูปภาพ \(imageAnchor.referenceImage.name ?? "") อยู่ในมุมมอง")
            } else {
                print("รูปภาพออกไปนอกมุมมองแล้ว")
            }
        }
    }
}
```

---

## 19. Light Estimation

ARKit ประมาณค่าแสงในสภาพแวดล้อมเพื่อให้วัตถุ AR ดูสมจริง

```swift
import ARKit
import RealityKit
import SceneKit

class LightEstimationManager {
    // Basic Light Estimation
    func applyLightEstimation(from frame: ARFrame, to sceneView: ARSCNView) {
        guard let lightEstimate = frame.lightEstimate else { return }
        
        // ความสว่างรวม (lumen)
        let ambientIntensity = lightEstimate.ambientIntensity
        
        // อุณหภูมิสี (Kelvin)
        let ambientColorTemperature = lightEstimate.ambientColorTemperature
        
        print("ความสว่าง: \(ambientIntensity) lm")
        print("อุณหภูมิสี: \(ambientColorTemperature) K")
        
        // ปรับ lighting ของ SceneKit
        sceneView.scene.lightingEnvironment.intensity = CGFloat(ambientIntensity / 1000)
        
        // แปลง color temperature เป็น RGB
        let color = colorFromTemperature(ambientColorTemperature)
        sceneView.scene.lightingEnvironment.contents = color
    }
    
    // Directional Light Estimation (ต้องการ environment texturing)
    func applyDirectionalLightEstimation(from frame: ARFrame) {
        guard let directionalLightEstimate = frame.lightEstimate as? ARDirectionalLightEstimate else {
            return
        }
        
        // ทิศทางของแสง
        let primaryLightDirection = directionalLightEstimate.primaryLightDirection
        print("ทิศทางแสงหลัก: \(primaryLightDirection)")
        
        // ความสว่างของแสงหลัก
        let primaryLightIntensity = directionalLightEstimate.primaryLightIntensity
        print("ความสว่างแสงหลัก: \(primaryLightIntensity) lm")
    }
    
    // RealityKit จัดการ light estimation อัตโนมัติ
    func setupAutoLighting(in arView: ARView) {
        // เปิด environment-based lighting
        arView.environment.lighting.intensityExponent = 2
        
        // ใช้ IBL (Image-Based Lighting) จาก environment
        // RealityKit จัดการโดยอัตโนมัติเมื่อ environmentTexturing = .automatic
    }
    
    func colorFromTemperature(_ kelvin: CGFloat) -> UIColor {
        // แปลง Kelvin เป็นสี RGB (approximation)
        var red, green, blue: CGFloat
        
        let temp = kelvin / 100
        
        if temp <= 66 {
            red = 1.0
        } else {
            let r = 329.698727446 * pow(temp - 60, -0.1332047592) / 255
            red = CGFloat(max(0, min(1, r)))
        }
        
        if temp <= 66 {
            let g = 99.4708025861 * log(temp) - 161.1195681661
            green = CGFloat(max(0, min(1, g / 255)))
        } else {
            let g = 288.1221695283 * pow(temp - 60, -0.0755148492) / 255
            green = CGFloat(max(0, min(1, g)))
        }
        
        if temp >= 66 {
            blue = 1.0
        } else if temp <= 19 {
            blue = 0.0
        } else {
            let b = 138.5177312231 * log(temp - 10) - 305.0447927307
            blue = CGFloat(max(0, min(1, b / 255)))
        }
        
        return UIColor(red: red, green: green, blue: blue, alpha: 1.0)
    }
}
```

---

## 20. Occlusion

Occlusion ทำให้วัตถุ AR ซ่อนอยู่หลังวัตถุจริง

```swift
import ARKit
import RealityKit

class OcclusionManager {
    func setupOcclusion(in arView: ARView) {
        let config = ARWorldTrackingConfiguration()
        
        // เปิด People Occlusion (ต้องการ A12 chip ขึ้นไป)
        if ARWorldTrackingConfiguration.supportsFrameSemantics(.personSegmentationWithDepth) {
            config.frameSemantics = .personSegmentationWithDepth
        } else if ARWorldTrackingConfiguration.supportsFrameSemantics(.personSegmentation) {
            config.frameSemantics = .personSegmentation
        }
        
        arView.session.run(config)
        
        // เปิด Object Occlusion ด้วย LiDAR (ต้องการ LiDAR)
        if ARWorldTrackingConfiguration.supportsSceneReconstruction(.meshWithClassification) {
            config.sceneReconstruction = .meshWithClassification
            arView.environment.sceneUnderstanding.options.insert(.occlusion)
        }
    }
}
```

---

## 21. People Occlusion

```swift
import ARKit
import RealityKit

class PeopleOcclusionManager {
    func enablePeopleOcclusion(in arView: ARView) {
        let config = ARWorldTrackingConfiguration()
        
        // ตรวจสอบว่ารองรับหรือไม่
        guard ARWorldTrackingConfiguration.supportsFrameSemantics(.personSegmentationWithDepth) else {
            print("People Occlusion ต้องการ A12 chip ขึ้นไป")
            return
        }
        
        config.frameSemantics.insert(.personSegmentationWithDepth)
        arView.session.run(config)
        
        print("People Occlusion เปิดใช้งานแล้ว")
    }
    
    // ประมวลผล segmentation buffer
    func processSegmentationBuffer(from frame: ARFrame) {
        guard let segmentationBuffer = frame.segmentationBuffer else { return }
        
        let width = CVPixelBufferGetWidth(segmentationBuffer)
        let height = CVPixelBufferGetHeight(segmentationBuffer)
        
        print("Segmentation buffer: \(width) x \(height)")
        
        // ตรวจสอบ estimated depth
        if let estimatedDepth = frame.estimatedDepthData {
            print("มี estimated depth data")
        }
    }
}
```

---

## 22. ARQuickLook

ARQuickLook ให้ผู้ใช้ดู 3D models ใน AR โดยไม่ต้องเขียน code มาก

```swift
import QuickLook
import ARKit

class ARQuickLookExample: NSObject, QLPreviewControllerDelegate, QLPreviewControllerDataSource {
    var viewController: UIViewController!
    var modelURL: URL!
    
    func showARQuickLook(modelName: String, from viewController: UIViewController) {
        self.viewController = viewController
        
        // URL ของ .usdz ไฟล์
        guard let url = Bundle.main.url(forResource: modelName, withExtension: "usdz") else {
            print("ไม่พบไฟล์ \(modelName).usdz")
            return
        }
        
        modelURL = url
        
        let previewController = QLPreviewController()
        previewController.dataSource = self
        previewController.delegate = self
        
        viewController.present(previewController, animated: true)
    }
    
    // โหลด model จาก URL ระยะไกล
    func showRemoteModel(url: URL, from viewController: UIViewController) {
        let previewController = QLPreviewController()
        previewController.dataSource = self
        previewController.delegate = self
        
        modelURL = url
        viewController.present(previewController, animated: true)
    }
    
    // MARK: - QLPreviewControllerDataSource
    
    func numberOfPreviewItems(in controller: QLPreviewController) -> Int {
        return 1
    }
    
    func previewController(_ controller: QLPreviewController, previewItemAt index: Int) -> QLPreviewItem {
        return ARQuickLookPreviewItem(fileAt: modelURL)
    }
    
    // MARK: - QLPreviewControllerDelegate
    
    func previewControllerWillDismiss(_ controller: QLPreviewController) {
        print("ARQuickLook กำลังปิด")
    }
}

// Custom ARQuickLook Preview Item
class CustomARQuickLookItem: NSObject, QLPreviewItem {
    let previewItemURL: URL?
    let previewItemTitle: String?
    
    // กำหนดขนาดเริ่มต้น
    var canonicalWebPageURL: URL?
    var allowsContentScaling: Bool = true
    
    init(url: URL, title: String) {
        self.previewItemURL = url
        self.previewItemTitle = title
    }
}
```

---

## 23. Reality Composer Basics

Reality Composer เป็นเครื่องมือของ Apple สำหรับสร้าง AR content แบบ visual

```swift
import RealityKit

class RealityComposerExample {
    // โหลด .reality file จาก Reality Composer
    func loadRealityFile() async throws {
        // Reality Composer สร้าง .reality ไฟล์
        let realityURL = Bundle.main.url(forResource: "Experience", withExtension: "reality")!
        
        let anchor = try await Experience.loadBox()
        // Experience เป็น generated code จาก Reality Composer
        
        // เข้าถึง entities ที่กำหนดชื่อไว้ใน Reality Composer
        // anchor.findEntity(named: "MyObject")
    }
    
    // โหลด Reality File แบบ manual
    func loadRealityFilManually() async {
        do {
            let anchor = try await Entity.loadAnchor(
                contentsOf: Bundle.main.url(forResource: "MyScene", withExtension: "reality")!
            )
            
            // หา entities ตามชื่อ
            if let box = anchor.findEntity(named: "Box") {
                print("พบ entity: \(box.name)")
                
                // ปรับแต่ง entity
                if var model = box as? ModelEntity {
                    var material = SimpleMaterial()
                    material.color = .init(tint: .systemBlue)
                    model.model?.materials = [material]
                }
            }
        } catch {
            print("ไม่สามารถโหลด Reality File: \(error)")
        }
    }
    
    // เล่น animation ที่สร้างใน Reality Composer
    func playRealityComposerAnimation(entity: Entity) {
        for animation in entity.availableAnimations {
            print("Animation: \(animation.name)")
            entity.playAnimation(animation.repeat(duration: .infinity))
        }
    }
    
    // Trigger actions จาก Reality Composer
    func setupBehaviors(anchor: HasAnchoring) {
        // Reality Composer สร้าง behaviors ผ่าน Notification triggers
        // ส่ง notification เพื่อ trigger behavior
        NotificationCenter.default.post(
            name: Notification.Name("TapAction"),
            object: nil
        )
    }
}
```

---

## 24. AR Debugging Tools

เครื่องมือสำหรับ debug AR apps

```swift
import ARKit
import RealityKit

class ARDebuggingTools {
    func setupDebugVisualizations(arView: ARView) {
        // 1. Feature Points - จุดที่ ARKit ใช้ติดตาม
        arView.debugOptions.insert(.showFeaturePoints)
        
        // 2. World Origin - แสดง origin axes
        arView.debugOptions.insert(.showWorldOrigin)
        
        // 3. Anchor Origins - แสดงตำแหน่ง anchors
        arView.debugOptions.insert(.showAnchorOrigins)
        
        // 4. Anchor Geometry - แสดง geometry ของ anchors
        arView.debugOptions.insert(.showAnchorGeometry)
        
        // 5. Physics - แสดง collision shapes
        arView.debugOptions.insert(.showPhysics)
        
        // 6. Statistics - แสดง FPS และ other stats
        arView.debugOptions.insert(.showStatistics)
    }
    
    func setupARSCNViewDebugging(sceneView: ARSCNView) {
        // SceneKit debug options
        sceneView.debugOptions = [
            SCNDebugOptions.showBoundingBoxes,
            SCNDebugOptions.showWireframe,
            SCNDebugOptions.showFeaturePoints,
            SCNDebugOptions.showWorldOrigin
        ]
        
        sceneView.showsStatistics = true
    }
    
    // ดู session state
    func logSessionInfo(session: ARSession) {
        guard let currentFrame = session.currentFrame else { return }
        
        let camera = currentFrame.camera
        print("=== AR Session Info ===")
        print("Tracking state: \(camera.trackingState)")
        print("World mapping: \(currentFrame.worldMappingStatus)")
        print("FPS: \(1.0 / currentFrame.timestamp)")
        print("Anchors count: \(currentFrame.anchors.count)")
        print("Light intensity: \(currentFrame.lightEstimate?.ambientIntensity ?? 0)")
    }
    
    // Custom overlay สำหรับ debug
    func createDebugOverlay(in arView: ARView) -> UIView {
        let overlay = UIView(frame: CGRect(x: 10, y: 50, width: 200, height: 100))
        overlay.backgroundColor = UIColor.black.withAlphaComponent(0.7)
        overlay.layer.cornerRadius = 8
        
        let label = UILabel(frame: overlay.bounds.insetBy(dx: 10, dy: 10))
        label.textColor = .white
        label.font = .monospacedSystemFont(ofSize: 11, weight: .regular)
        label.numberOfLines = 0
        label.tag = 100
        overlay.addSubview(label)
        
        arView.addSubview(overlay)
        return overlay
    }
    
    func updateDebugOverlay(_ overlay: UIView, frame: ARFrame) {
        guard let label = overlay.viewWithTag(100) as? UILabel else { return }
        
        let camera = frame.camera
        let position = camera.transform.columns.3
        
        label.text = """
        State: \(trackingStateString(camera.trackingState))
        Pos: (\(String(format: "%.2f", position.x)), \(String(format: "%.2f", position.y)), \(String(format: "%.2f", position.z)))
        Anchors: \(frame.anchors.count)
        """
    }
    
    func trackingStateString(_ state: ARCamera.TrackingState) -> String {
        switch state {
        case .normal: return "Normal"
        case .notAvailable: return "N/A"
        case .limited(let reason):
            switch reason {
            case .initializing: return "Init"
            case .excessiveMotion: return "Motion"
            case .insufficientFeatures: return "Features"
            case .relocalizing: return "Reloc"
            @unknown default: return "Limited"
            }
        }
    }
}
```

---

## 25. Performance in AR

การเพิ่มประสิทธิภาพสำหรับ AR apps

```swift
import ARKit
import RealityKit

class ARPerformanceOptimizer {
    // 1. ลด polygon count
    func createLowPolyModel() -> ModelEntity {
        // ใช้ primitive shapes แทน high-poly models
        let mesh = MeshResource.generateBox(size: 0.1)
        let material = SimpleMaterial(color: .blue, isMetallic: false)
        return ModelEntity(mesh: mesh, materials: [material])
    }
    
    // 2. ใช้ Level of Detail (LOD)
    func applyLOD(entity: Entity, cameraDistance: Float) {
        guard let model = entity as? ModelEntity else { return }
        
        if cameraDistance < 1.0 {
            // ใกล้ - high detail
            model.isEnabled = true
        } else if cameraDistance < 5.0 {
            // กลาง - medium detail
            model.isEnabled = true
        } else {
            // ไกล - ซ่อน
            model.isEnabled = false
        }
    }
    
    // 3. จำกัด anchors
    var maxAnchors = 20
    var activeAnchors: [AnchorEntity] = []
    
    func addAnchorWithLimit(_ anchor: AnchorEntity, to arView: ARView) {
        if activeAnchors.count >= maxAnchors {
            // ลบ anchor เก่าที่สุด
            let oldest = activeAnchors.removeFirst()
            oldest.removeFromParent()
        }
        
        activeAnchors.append(anchor)
        arView.scene.addAnchor(anchor)
    }
    
    // 4. ปิด features ที่ไม่ใช้
    func optimizeConfiguration() -> ARWorldTrackingConfiguration {
        let config = ARWorldTrackingConfiguration()
        
        // เปิดเฉพาะ features ที่ต้องการ
        config.planeDetection = [.horizontal]  // ไม่ตรวจ vertical
        config.environmentTexturing = .none    // ปิด environment texture
        
        // ปิด scene reconstruction ถ้าไม่ต้องการ occlusion
        // config.sceneReconstruction = .none
        
        return config
    }
    
    // 5. Texture optimization
    func createOptimizedMaterial() -> SimpleMaterial {
        var material = SimpleMaterial()
        // ใช้ textures ขนาดเล็ก
        // หลีกเลี่ยง large textures
        return material
    }
    
    // 6. ใช้ background thread สำหรับการคำนวณ
    func performHeavyComputation(completion: @escaping (Any) -> Void) {
        DispatchQueue.global(qos: .userInitiated).async {
            // ทำการคำนวณที่หนัก
            let result = "computed"
            
            DispatchQueue.main.async {
                completion(result)
            }
        }
    }
    
    // 7. Metal optimization
    func useMetalForRendering(in arView: ARView) {
        // ARView ใช้ Metal โดยอัตโนมัติ
        // แต่สามารถ customize render pipeline ได้
        arView.renderOptions = [
            .disableAREnvironmentLighting,  // ปิดถ้าไม่ต้องการ
            .disableGroundingShadows,       // ปิด shadow
            .disableMotionBlur              // ปิด motion blur
        ]
    }
}
```

---

## 26. Practical Exercises

### แบบฝึกหัดที่ 1: AR Solar System

```swift
import ARKit
import RealityKit

class SolarSystemAR: UIViewController {
    var arView: ARView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        arView = ARView(frame: view.bounds)
        arView.autoresizingMask = [.flexibleWidth, .flexibleHeight]
        view.addSubview(arView)
        
        setupAR()
        createSolarSystem()
    }
    
    func setupAR() {
        let config = ARWorldTrackingConfiguration()
        config.planeDetection = [.horizontal]
        arView.session.run(config)
    }
    
    func createSolarSystem() {
        let anchor = AnchorEntity(plane: .horizontal)
        
        // ดวงอาทิตย์
        let sun = createPlanet(radius: 0.08, color: .yellow, emissive: true)
        sun.position = SIMD3<Float>(0, 0.1, 0)
        anchor.addChild(sun)
        
        // โลก
        let earth = createPlanet(radius: 0.03, color: .blue, emissive: false)
        earth.name = "Earth"
        anchor.addChild(earth)
        
        // ดวงจันทร์
        let moon = createPlanet(radius: 0.01, color: .gray, emissive: false)
        
        // สร้าง orbit animation
        animateOrbit(planet: earth, aroundCenter: sun.position, radius: 0.25, duration: 5.0)
        
        arView.scene.addAnchor(anchor)
    }
    
    func createPlanet(radius: Float, color: UIColor, emissive: Bool) -> ModelEntity {
        let mesh = MeshResource.generateSphere(radius: radius)
        
        var material = SimpleMaterial()
        material.color = .init(tint: color)
        if emissive {
            var pbrMaterial = PhysicallyBasedMaterial()
            pbrMaterial.emissiveColor = .init(color: color)
            pbrMaterial.emissiveIntensity = 1.0
            return ModelEntity(mesh: mesh, materials: [pbrMaterial])
        }
        
        return ModelEntity(mesh: mesh, materials: [material])
    }
    
    func animateOrbit(planet: ModelEntity, aroundCenter center: SIMD3<Float>, radius: Float, duration: Double) {
        var transforms: [Transform] = []
        let steps = 60
        
        for i in 0...steps {
            let angle = Float(i) / Float(steps) * 2 * Float.pi
            let x = center.x + radius * cos(angle)
            let z = center.z + radius * sin(angle)
            let transform = Transform(
                scale: .one,
                rotation: simd_quatf(angle: angle, axis: SIMD3<Float>(0, 1, 0)),
                translation: SIMD3<Float>(x, center.y, z)
            )
            transforms.append(transform)
        }
        
        let animation = FromToByAnimation<Transform>(
            name: "orbit",
            from: transforms.first!,
            to: transforms.last!,
            duration: duration,
            timing: .linear,
            bindTarget: .transform
        )
        
        if let animResource = try? AnimationResource.generate(with: animation) {
            planet.playAnimation(animResource.repeat(duration: .infinity))
        }
    }
    
    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        arView.session.pause()
    }
}
```

### แบบฝึกหัดที่ 2: AR Measuring Tool

```swift
import ARKit
import RealityKit

class ARMeasuringTool: UIViewController {
    var arView: ARView!
    var startPoint: SIMD3<Float>?
    var measureLine: ModelEntity?
    var measureLabel: ModelEntity?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        arView = ARView(frame: view.bounds)
        arView.autoresizingMask = [.flexibleWidth, .flexibleHeight]
        view.addSubview(arView)
        
        let tap = UITapGestureRecognizer(target: self, action: #selector(handleTap))
        arView.addGestureRecognizer(tap)
        
        setupAR()
    }
    
    func setupAR() {
        let config = ARWorldTrackingConfiguration()
        config.planeDetection = [.horizontal]
        arView.session.run(config)
    }
    
    @objc func handleTap(_ sender: UITapGestureRecognizer) {
        let location = sender.location(in: arView)
        
        guard let query = arView.makeRaycastQuery(
            from: location,
            allowing: .estimatedPlane,
            alignment: .horizontal
        ),
        let result = arView.session.raycast(query).first else { return }
        
        let worldPosition = SIMD3<Float>(
            result.worldTransform.columns.3.x,
            result.worldTransform.columns.3.y,
            result.worldTransform.columns.3.z
        )
        
        if startPoint == nil {
            // วาง start point
            startPoint = worldPosition
            placeMarker(at: worldPosition, color: .green)
        } else {
            // วาง end point และวัดระยะ
            placeMarker(at: worldPosition, color: .red)
            measureDistance(from: startPoint!, to: worldPosition)
            startPoint = nil
        }
    }
    
    func placeMarker(at position: SIMD3<Float>, color: UIColor) {
        let sphere = ModelEntity(
            mesh: .generateSphere(radius: 0.01),
            materials: [SimpleMaterial(color: color, isMetallic: false)]
        )
        sphere.position = position
        
        let anchor = AnchorEntity(world: .init(position))
        anchor.addChild(sphere)
        arView.scene.addAnchor(anchor)
    }
    
    func measureDistance(from start: SIMD3<Float>, to end: SIMD3<Float>) {
        let distance = simd_distance(start, end)
        let distanceCM = distance * 100
        
        print("ระยะห่าง: \(String(format: "%.1f", distanceCM)) ซม.")
        
        // แสดง text ระยะห่าง
        let midpoint = (start + end) / 2
        showDistanceLabel(distanceCM, at: midpoint)
    }
    
    func showDistanceLabel(_ distance: Float, at position: SIMD3<Float>) {
        let text = String(format: "%.1f ซม.", distance)
        let textMesh = MeshResource.generateText(
            text,
            extrusionDepth: 0.002,
            font: .systemFont(ofSize: 0.05)
        )
        
        let textEntity = ModelEntity(
            mesh: textMesh,
            materials: [SimpleMaterial(color: .white, isMetallic: false)]
        )
        textEntity.position = position
        
        let anchor = AnchorEntity(world: .init(position))
        anchor.addChild(textEntity)
        arView.scene.addAnchor(anchor)
    }
}
```

---

## 27. Building a Furniture AR App

แอปวางเฟอร์นิเจอร์ AR แบบสมบูรณ์

```swift
import ARKit
import RealityKit
import SwiftUI
import Combine

// Model ของเฟอร์นิเจอร์
struct FurnitureItem: Identifiable {
    let id = UUID()
    let name: String
    let modelName: String
    let thumbnail: String
    let price: Double
}

// ViewModel
class FurnitureARViewModel: ObservableObject {
    @Published var furnitureItems: [FurnitureItem] = [
        FurnitureItem(name: "โซฟา 2 ที่นั่ง", modelName: "sofa_2seat", thumbnail: "sofa", price: 15000),
        FurnitureItem(name: "โต๊ะกาแฟ", modelName: "coffee_table", thumbnail: "table", price: 5000),
        FurnitureItem(name: "ตู้หนังสือ", modelName: "bookshelf", thumbnail: "shelf", price: 8000),
        FurnitureItem(name: "เก้าอี้", modelName: "chair", thumbnail: "chair", price: 3500)
    ]
    
    @Published var selectedItem: FurnitureItem?
    @Published var placedItems: [UUID: AnchorEntity] = [:]
    @Published var showFurniturePicker = false
    @Published var isPlacingMode = false
    
    var arView: ARView?
    
    func selectFurniture(_ item: FurnitureItem) {
        selectedItem = item
        isPlacingMode = true
        showFurniturePicker = false
    }
    
    func placeSelectedFurniture(at transform: simd_float4x4) {
        guard let selected = selectedItem else { return }
        
        Task {
            do {
                let entity = try await loadFurnitureModel(named: selected.modelName)
                
                await MainActor.run {
                    let anchor = AnchorEntity(world: transform)
                    anchor.addChild(entity)
                    arView?.scene.addAnchor(anchor)
                    placedItems[selected.id] = anchor
                    isPlacingMode = false
                }
            } catch {
                // ใช้ placeholder แทน
                await MainActor.run {
                    let placeholder = self.createPlaceholderFurniture(for: selected)
                    let anchor = AnchorEntity(world: transform)
                    anchor.addChild(placeholder)
                    self.arView?.scene.addAnchor(anchor)
                    self.placedItems[selected.id] = anchor
                    self.isPlacingMode = false
                }
            }
        }
    }
    
    func loadFurnitureModel(named name: String) async throws -> ModelEntity {
        return try await ModelEntity.loadModel(named: name)
    }
    
    func createPlaceholderFurniture(for item: FurnitureItem) -> ModelEntity {
        let mesh = MeshResource.generateBox(width: 0.5, height: 0.3, depth: 0.5)
        var material = SimpleMaterial()
        material.color = .init(tint: .brown.withAlphaComponent(0.8))
        
        let entity = ModelEntity(mesh: mesh, materials: [material])
        entity.name = item.name
        
        // ข้อความชื่อ
        let textMesh = MeshResource.generateText(
            item.name,
            extrusionDepth: 0.005,
            font: .systemFont(ofSize: 0.05)
        )
        let textEntity = ModelEntity(
            mesh: textMesh,
            materials: [SimpleMaterial(color: .white, isMetallic: false)]
        )
        textEntity.position.y = 0.2
        entity.addChild(textEntity)
        
        entity.generateCollisionShapes(recursive: true)
        return entity
    }
    
    func removeAllFurniture() {
        for (_, anchor) in placedItems {
            anchor.removeFromParent()
        }
        placedItems.removeAll()
    }
    
    func totalCost() -> Double {
        return furnitureItems
            .filter { placedItems[$0.id] != nil }
            .reduce(0) { $0 + $1.price }
    }
}

// SwiftUI View
struct FurnitureARView: View {
    @StateObject private var viewModel = FurnitureARViewModel()
    
    var body: some View {
        ZStack {
            // AR View
            FurnitureARViewRepresentable(viewModel: viewModel)
                .edgesIgnoringSafeArea(.all)
            
            // Crosshair เมื่ออยู่ใน placing mode
            if viewModel.isPlacingMode {
                Image(systemName: "plus.circle")
                    .font(.system(size: 40))
                    .foregroundColor(.white)
                    .shadow(color: .black, radius: 2)
            }
            
            // UI Overlay
            VStack {
                // Header
                HStack {
                    Text("AR Furniture")
                        .font(.headline)
                        .foregroundColor(.white)
                    
                    Spacer()
                    
                    if !viewModel.placedItems.isEmpty {
                        Text("ราคารวม: ฿\(String(format: "%.0f", viewModel.totalCost()))")
                            .font(.caption)
                            .foregroundColor(.yellow)
                    }
                }
                .padding()
                .background(Color.black.opacity(0.6))
                
                Spacer()
                
                // Selected furniture indicator
                if viewModel.isPlacingMode, let selected = viewModel.selectedItem {
                    Text("แตะเพื่อวาง: \(selected.name)")
                        .padding(.horizontal, 20)
                        .padding(.vertical, 10)
                        .background(Color.black.opacity(0.7))
                        .foregroundColor(.white)
                        .cornerRadius(20)
                }
                
                // Bottom Controls
                HStack(spacing: 20) {
                    // เลือกเฟอร์นิเจอร์
                    Button(action: { viewModel.showFurniturePicker = true }) {
                        Label("เลือกเฟอร์นิเจอร์", systemImage: "plus.square")
                            .padding()
                            .background(Color.blue)
                            .foregroundColor(.white)
                            .cornerRadius(10)
                    }
                    
                    // ล้างทั้งหมด
                    if !viewModel.placedItems.isEmpty {
                        Button(action: { viewModel.removeAllFurniture() }) {
                            Label("ล้าง", systemImage: "trash")
                                .padding()
                                .background(Color.red)
                                .foregroundColor(.white)
                                .cornerRadius(10)
                        }
                    }
                }
                .padding()
            }
        }
        .sheet(isPresented: $viewModel.showFurniturePicker) {
            FurniturePickerView(viewModel: viewModel)
        }
    }
}

// Furniture Picker Sheet
struct FurniturePickerView: View {
    @ObservedObject var viewModel: FurnitureARViewModel
    @Environment(\.dismiss) var dismiss
    
    var body: some View {
        NavigationView {
            List(viewModel.furnitureItems) { item in
                Button(action: {
                    viewModel.selectFurniture(item)
                    dismiss()
                }) {
                    HStack {
                        Image(systemName: "chair.fill")
                            .font(.largeTitle)
                            .frame(width: 60, height: 60)
                            .background(Color.gray.opacity(0.2))
                            .cornerRadius(8)
                        
                        VStack(alignment: .leading) {
                            Text(item.name)
                                .font(.headline)
                                .foregroundColor(.primary)
                            
                            Text("฿\(String(format: "%.0f", item.price))")
                                .font(.subheadline)
                                .foregroundColor(.secondary)
                        }
                        
                        Spacer()
                        
                        Image(systemName: "chevron.right")
                            .foregroundColor(.gray)
                    }
                }
            }
            .navigationTitle("เลือกเฟอร์นิเจอร์")
            .navigationBarItems(trailing: Button("ยกเลิก") { dismiss() })
        }
    }
}

// UIViewRepresentable สำหรับ ARView
struct FurnitureARViewRepresentable: UIViewRepresentable {
    @ObservedObject var viewModel: FurnitureARViewModel
    
    func makeUIView(context: Context) -> ARView {
        let arView = ARView(frame: .zero)
        viewModel.arView = arView
        
        let config = ARWorldTrackingConfiguration()
        config.planeDetection = [.horizontal]
        config.environmentTexturing = .automatic
        arView.session.run(config)
        
        let tap = UITapGestureRecognizer(target: context.coordinator, action: #selector(Coordinator.handleTap))
        arView.addGestureRecognizer(tap)
        
        return arView
    }
    
    func updateUIView(_ uiView: ARView, context: Context) {}
    
    func makeCoordinator() -> Coordinator {
        Coordinator(viewModel: viewModel)
    }
    
    class Coordinator: NSObject {
        let viewModel: FurnitureARViewModel
        
        init(viewModel: FurnitureARViewModel) {
            self.viewModel = viewModel
        }
        
        @objc func handleTap(_ sender: UITapGestureRecognizer) {
            guard viewModel.isPlacingMode,
                  let arView = viewModel.arView else { return }
            
            let location = sender.location(in: arView)
            
            guard let query = arView.makeRaycastQuery(
                from: location,
                allowing: .estimatedPlane,
                alignment: .horizontal
            ),
            let result = arView.session.raycast(query).first else { return }
            
            viewModel.placeSelectedFurniture(at: result.worldTransform)
        }
    }
}
```

---

## 28. Summary

### สิ่งที่ได้เรียนรู้ในบทนี้

1. **ARKit Fundamentals**: ARSession, ARConfiguration, ARFrame, ARCamera
2. **Configuration Types**: World Tracking, Face Tracking, Body Tracking, Image Tracking
3. **Anchor System**: ARAnchor, ARPlaneAnchor, ARImageAnchor, ARObjectAnchor
4. **RealityKit**: Entity-Component System, ModelEntity, AnchorEntity
5. **SwiftUI Integration**: RealityView
6. **SceneKit**: ARSCNView, SCNNode
7. **Interaction**: Hit Testing, Raycasting
8. **Detection**: Plane Detection, Image Recognition
9. **Features**: Light Estimation, Occlusion, People Occlusion
10. **Tools**: ARQuickLook, Reality Composer
11. **Debugging**: Debug Options, Performance
12. **Project**: Furniture AR App

### แนวทางปฏิบัติที่ดีที่สุด

```swift
// Best Practices สรุป
struct ARBestPractices {
    /*
    1. เสมอตรวจสอบ device support ก่อนใช้งาน
    2. จัดการ session lifecycle อย่างถูกต้อง (pause/resume)
    3. ให้ feedback แก่ผู้ใช้เมื่อ tracking จำกัด
    4. ใช้ raycast แทน hitTest (modern API)
    5. จำกัดจำนวน anchors และ entities
    6. ปิด features ที่ไม่ใช้เพื่อประสิทธิภาพ
    7. ทดสอบบนอุปกรณ์จริง (ไม่ใช่ Simulator)
    8. ขอ camera permission อย่างถูกต้อง
    9. Handle interruptions และ errors
    10. ใช้ background thread สำหรับการโหลด assets
    */
}
```

### ขั้นตอนต่อไป

- ศึกษา visionOS และ Apple Vision Pro
- ทดลอง Reality Composer Pro
- เรียนรู้ Metal สำหรับ custom AR rendering
- สำรวจ Object Scanning workflow
- ทดลองสร้าง Collaborative AR experiences

---

*จบ Part 51: ARKit และ Augmented Reality บน iOS*
