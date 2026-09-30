# Part 50: Vision Framework ใน iOS

## บทนำ

**Vision framework** เป็น framework ของ Apple ที่ทรงพลังสำหรับการประมวลผลภาพและวิดีโอ ครอบคลุมตั้งแต่การตรวจจับใบหน้า การจดจำข้อความ การตรวจจับวัตถุ ไปจนถึงการวิเคราะห์ท่าทางของร่างกาย

ในบทนี้เราจะเรียนรู้:
- ภาพรวมของ Vision framework
- VNRequest ประเภทต่างๆ
- การตรวจจับใบหน้าและ landmarks
- Body/Hand pose estimation
- การจดจำข้อความ (OCR)
- การสแกนเอกสาร
- การผสาน Vision กับกล้อง
- การสร้างแอป Document Scanner

---

## 50.1 Vision Framework Overview

Vision framework มี API สำหรับงาน Computer Vision หลากหลายประเภท โดยไม่ต้องเขียน ML model เอง

```
Vision Framework
├── Face Analysis
│   ├── VNDetectFaceRectanglesRequest      - ตรวจจับกรอบใบหน้า
│   ├── VNDetectFaceLandmarksRequest       - จุดสำคัญบนใบหน้า
│   └── VNDetectFaceCaptureQualityRequest  - คุณภาพภาพใบหน้า
├── Human Body
│   ├── VNDetectHumanBodyPoseRequest       - ท่าทางร่างกาย
│   └── VNDetectHumanHandPoseRequest       - ท่าทางมือ
├── Text
│   └── VNRecognizeTextRequest             - จดจำข้อความ (OCR)
├── Barcodes
│   └── VNDetectBarcodesRequest            - บาร์โค้ด/QR Code
├── Object Tracking
│   └── VNTrackObjectRequest               - ติดตามวัตถุ
├── Scene
│   ├── VNGenerateAttentionBasedSaliencyImageRequest
│   └── VNClassifyImageRequest
└── Rectangles
    └── VNDetectRectanglesRequest          - ตรวจจับสี่เหลี่ยม
```

### การ import และ setup

```swift
import Vision
import UIKit
import AVFoundation
```

---

## 50.2 VNRequest Types

VNRequest เป็น base class ของทุก request ใน Vision framework

```swift
import Vision

// ประเภทของ VNRequest หลักๆ
class VisionRequestDemo {
    
    // Image-based requests (ประมวลผลรูปภาพ)
    func imageBasedRequests() {
        // 1. Face Detection
        let faceRequest = VNDetectFaceRectanglesRequest()
        
        // 2. Face Landmarks
        let landmarksRequest = VNDetectFaceLandmarksRequest()
        
        // 3. Text Recognition
        let textRequest = VNRecognizeTextRequest()
        
        // 4. Barcode Detection
        let barcodeRequest = VNDetectBarcodesRequest()
        
        // 5. Rectangle Detection
        let rectRequest = VNDetectRectanglesRequest()
        
        // 6. Body Pose
        let bodyPoseRequest = VNDetectHumanBodyPoseRequest()
        
        // 7. Hand Pose
        let handPoseRequest = VNDetectHumanHandPoseRequest()
        
        // 8. Saliency
        let saliencyRequest = VNGenerateAttentionBasedSaliencyImageRequest()
        
        // 9. Animal Detection
        let animalRequest = VNRecognizeAnimalsRequest()
        
        // 10. Horizon Detection
        let horizonRequest = VNDetectHorizonRequest()
        
        // 11. Image Classification
        let classifyRequest = VNClassifyImageRequest()
        
        _ = [faceRequest, landmarksRequest, textRequest, barcodeRequest,
             rectRequest, bodyPoseRequest, handPoseRequest, saliencyRequest,
             animalRequest, horizonRequest, classifyRequest]
    }
    
    // Sequence-based requests (ประมวลผลวิดีโอ)
    func sequenceBasedRequests() {
        // Object Tracking
        let trackingRequest = VNTrackObjectRequest(detectedObjectObservation:
            VNDetectedObjectObservation(boundingBox: .zero))
        
        // Rectangle Tracking
        let rectTrackingRequest = VNTrackRectangleRequest(rectangleObservation:
            VNRectangleObservation())
        
        _ = [trackingRequest, rectTrackingRequest]
    }
}
```

### ลำดับชั้นของ VNObservation

```swift
// VNObservation hierarchy
// VNObservation (base)
//   ├── VNDetectedObjectObservation
//   │   ├── VNFaceObservation
//   │   ├── VNRecognizedObjectObservation  
//   │   ├── VNRectangleObservation
//   │   └── VNHumanObservation
//   ├── VNClassificationObservation
//   ├── VNRecognizedTextObservation
//   ├── VNBarcodeObservation
//   ├── VNHorizonObservation
//   ├── VNSaliencyImageObservation
//   └── VNHumanBodyPoseObservation

// ตัวอย่างการใช้งาน VNFaceObservation
func processFaceObservation(_ observation: VNFaceObservation) {
    print("Face bounding box: \(observation.boundingBox)")
    print("Face roll: \(observation.roll?.doubleValue ?? 0)")
    print("Face yaw: \(observation.yaw?.doubleValue ?? 0)")
    print("Face pitch: \(observation.pitch?.doubleValue ?? 0)")
    
    if let landmarks = observation.landmarks {
        print("Has landmarks: true")
        print("Left eye points: \(landmarks.leftEye?.normalizedPoints ?? [])")
    }
}
```

---

## 50.3 VNImageRequestHandler

**VNImageRequestHandler** ใช้สำหรับประมวลผล requests บนรูปภาพเดียว

```swift
import Vision
import UIKit

class ImageRequestHandlerDemo {
    
    // MARK: - สร้าง Handler จากแหล่งข้อมูลต่างๆ
    
    func createHandler(from image: UIImage) -> VNImageRequestHandler? {
        guard let ciImage = CIImage(image: image) else { return nil }
        return VNImageRequestHandler(ciImage: ciImage, options: [:])
    }
    
    func createHandler(from cgImage: CGImage) -> VNImageRequestHandler {
        return VNImageRequestHandler(cgImage: cgImage, options: [:])
    }
    
    func createHandler(from pixelBuffer: CVPixelBuffer) -> VNImageRequestHandler {
        return VNImageRequestHandler(cvPixelBuffer: pixelBuffer, options: [:])
    }
    
    func createHandler(from url: URL) -> VNImageRequestHandler {
        return VNImageRequestHandler(url: url, options: [:])
    }
    
    func createHandler(from data: Data) -> VNImageRequestHandler? {
        guard let ciImage = CIImage(data: data) else { return nil }
        return VNImageRequestHandler(ciImage: ciImage, options: [:])
    }
    
    // MARK: - Orientation
    func createHandlerWithOrientation(from image: UIImage) -> VNImageRequestHandler? {
        guard let ciImage = CIImage(image: image) else { return nil }
        
        // ระบุ orientation ของภาพ
        let orientation = CGImagePropertyOrientation(image.imageOrientation)
        return VNImageRequestHandler(ciImage: ciImage, orientation: orientation, options: [:])
    }
    
    // MARK: - การรัน Multiple Requests
    func performMultipleRequests(on image: UIImage) {
        guard let handler = createHandler(from: image) else { return }
        
        let faceRequest = VNDetectFaceRectanglesRequest { request, error in
            guard let faces = request.results as? [VNFaceObservation] else { return }
            print("พบใบหน้า: \(faces.count) หน้า")
        }
        
        let textRequest = VNRecognizeTextRequest { request, error in
            guard let texts = request.results as? [VNRecognizedTextObservation] else { return }
            print("พบข้อความ: \(texts.count) บรรทัด")
        }
        
        let barcodeRequest = VNDetectBarcodesRequest { request, error in
            guard let barcodes = request.results as? [VNBarcodeObservation] else { return }
            print("พบบาร์โค้ด: \(barcodes.count) อัน")
        }
        
        // รัน requests พร้อมกันทีเดียว (batch processing)
        do {
            try handler.perform([faceRequest, textRequest, barcodeRequest])
        } catch {
            print("เกิดข้อผิดพลาด: \(error.localizedDescription)")
        }
    }
}

// MARK: - CGImagePropertyOrientation Extension
extension CGImagePropertyOrientation {
    init(_ uiOrientation: UIImage.Orientation) {
        switch uiOrientation {
        case .up: self = .up
        case .down: self = .down
        case .left: self = .left
        case .right: self = .right
        case .upMirrored: self = .upMirrored
        case .downMirrored: self = .downMirrored
        case .leftMirrored: self = .leftMirrored
        case .rightMirrored: self = .rightMirrored
        @unknown default: self = .up
        }
    }
}
```

---

## 50.4 VNSequenceRequestHandler

**VNSequenceRequestHandler** ใช้สำหรับประมวลผลลำดับของภาพ (วิดีโอ) โดยเก็บ state ระหว่าง frames

```swift
import Vision
import AVFoundation

class SequenceRequestHandlerDemo {
    
    // ใช้ VNSequenceRequestHandler สำหรับ video processing
    private let sequenceHandler = VNSequenceRequestHandler()
    
    // Object Tracking ระหว่าง frames
    private var trackingRequest: VNTrackObjectRequest?
    
    // เริ่มต้น tracking จาก initial observation
    func startTracking(initialObservation: VNDetectedObjectObservation) {
        let request = VNTrackObjectRequest(detectedObjectObservation: initialObservation)
        request.trackingLevel = .accurate // .accurate หรือ .fast
        self.trackingRequest = request
    }
    
    // ประมวลผลแต่ละ frame
    func processFrame(_ pixelBuffer: CVPixelBuffer) {
        guard let request = trackingRequest else { return }
        
        do {
            try sequenceHandler.perform([request], on: pixelBuffer)
            
            if let result = request.results?.first as? VNDetectedObjectObservation {
                print("Tracking confidence: \(result.confidence)")
                print("Tracked box: \(result.boundingBox)")
                
                // อัพเดต request ด้วย observation ใหม่
                trackingRequest = VNTrackObjectRequest(detectedObjectObservation: result)
                trackingRequest?.trackingLevel = .accurate
            }
        } catch {
            print("Tracking error: \(error)")
        }
    }
    
    // Face tracking ระหว่าง frames
    func trackFaces(in pixelBuffer: CVPixelBuffer) {
        let faceRequest = VNDetectFaceRectanglesRequest { request, error in
            guard let faces = request.results as? [VNFaceObservation] else { return }
            // ประมวลผลแต่ละ face
            for face in faces {
                print("Face at: \(face.boundingBox)")
            }
        }
        
        do {
            try self.sequenceHandler.perform([faceRequest], on: pixelBuffer)
        } catch {
            print("Error: \(error)")
        }
    }
}
```

---

## 50.5 Face Detection (VNDetectFaceRectanglesRequest)

```swift
import UIKit
import Vision

class FaceDetectionManager {
    
    // MARK: - ตรวจจับใบหน้า
    func detectFaces(
        in image: UIImage,
        completion: @escaping ([VNFaceObservation]) -> Void
    ) {
        guard let cgImage = image.cgImage else {
            completion([])
            return
        }
        
        let request = VNDetectFaceRectanglesRequest { request, error in
            if let error = error {
                print("Face detection error: \(error)")
                completion([])
                return
            }
            
            let faces = request.results as? [VNFaceObservation] ?? []
            DispatchQueue.main.async {
                completion(faces)
            }
        }
        
        // กำหนด revision ของ algorithm
        request.revision = VNDetectFaceRectanglesRequestRevision3
        
        let handler = VNImageRequestHandler(
            cgImage: cgImage,
            orientation: CGImagePropertyOrientation(image.imageOrientation)
        )
        
        DispatchQueue.global(qos: .userInitiated).async {
            do {
                try handler.perform([request])
            } catch {
                print("Perform error: \(error)")
                DispatchQueue.main.async { completion([]) }
            }
        }
    }
    
    // MARK: - แปลง normalized coordinates เป็น UIKit coordinates
    func convertBoundingBox(_ box: CGRect, to imageSize: CGSize) -> CGRect {
        // Vision ใช้ normalized coordinates (0-1) โดย origin อยู่ที่ bottom-left
        // UIKit ใช้ pixel coordinates โดย origin อยู่ที่ top-left
        
        let x = box.minX * imageSize.width
        let y = (1 - box.maxY) * imageSize.height  // flip Y axis
        let width = box.width * imageSize.width
        let height = box.height * imageSize.height
        
        return CGRect(x: x, y: y, width: width, height: height)
    }
    
    // MARK: - วาดกรอบใบหน้า
    func drawFaceBoxes(
        on image: UIImage,
        faces: [VNFaceObservation]
    ) -> UIImage {
        let renderer = UIGraphicsImageRenderer(size: image.size)
        
        return renderer.image { context in
            // วาดรูปต้นฉบับ
            image.draw(in: CGRect(origin: .zero, size: image.size))
            
            let cgContext = context.cgContext
            cgContext.setStrokeColor(UIColor.systemGreen.cgColor)
            cgContext.setLineWidth(3)
            
            for face in faces {
                let box = convertBoundingBox(face.boundingBox, to: image.size)
                cgContext.stroke(box)
                
                // แสดงค่า roll, yaw, pitch
                if let roll = face.roll, let yaw = face.yaw {
                    let text = "R:\(String(format: "%.1f°", roll.doubleValue * 180 / .pi)) Y:\(String(format: "%.1f°", yaw.doubleValue * 180 / .pi))"
                    let attrs: [NSAttributedString.Key: Any] = [
                        .foregroundColor: UIColor.systemGreen,
                        .font: UIFont.systemFont(ofSize: 12)
                    ]
                    text.draw(at: CGPoint(x: box.minX, y: box.minY - 16), withAttributes: attrs)
                }
            }
        }
    }
    
    // MARK: - ViewController ที่ใช้งาน Face Detection
    class FaceDetectionViewController: UIViewController {
        
        private let imageView = UIImageView()
        private let countLabel = UILabel()
        private let manager = FaceDetectionManager()
        
        override func viewDidLoad() {
            super.viewDidLoad()
            setupUI()
            
            // ทดสอบกับรูปภาพตัวอย่าง
            if let testImage = UIImage(named: "test_faces") {
                detectAndDraw(image: testImage)
            }
        }
        
        private func setupUI() {
            view.backgroundColor = .systemBackground
            
            imageView.contentMode = .scaleAspectFit
            imageView.translatesAutoresizingMaskIntoConstraints = false
            view.addSubview(imageView)
            
            countLabel.textAlignment = .center
            countLabel.font = .systemFont(ofSize: 18, weight: .medium)
            countLabel.translatesAutoresizingMaskIntoConstraints = false
            view.addSubview(countLabel)
            
            NSLayoutConstraint.activate([
                imageView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
                imageView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
                imageView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
                imageView.heightAnchor.constraint(equalTo: view.heightAnchor, multiplier: 0.8),
                
                countLabel.topAnchor.constraint(equalTo: imageView.bottomAnchor, constant: 16),
                countLabel.leadingAnchor.constraint(equalTo: view.leadingAnchor),
                countLabel.trailingAnchor.constraint(equalTo: view.trailingAnchor)
            ])
        }
        
        func detectAndDraw(image: UIImage) {
            manager.detectFaces(in: image) { [weak self] faces in
                let annotated = self?.manager.drawFaceBoxes(on: image, faces: faces)
                self?.imageView.image = annotated
                self?.countLabel.text = "พบใบหน้า: \(faces.count) ใบหน้า"
            }
        }
    }
}
```

---

## 50.6 Face Landmarks (VNDetectFaceLandmarksRequest)

```swift
import UIKit
import Vision

class FaceLandmarksDetector {
    
    // ตรวจจับ landmarks บนใบหน้า
    func detectLandmarks(
        in image: UIImage,
        completion: @escaping ([VNFaceObservation]) -> Void
    ) {
        guard let cgImage = image.cgImage else {
            completion([])
            return
        }
        
        let request = VNDetectFaceLandmarksRequest { request, error in
            let observations = request.results as? [VNFaceObservation] ?? []
            DispatchQueue.main.async { completion(observations) }
        }
        
        let handler = VNImageRequestHandler(
            cgImage: cgImage,
            orientation: CGImagePropertyOrientation(image.imageOrientation)
        )
        
        DispatchQueue.global(qos: .userInitiated).async {
            try? handler.perform([request])
        }
    }
    
    // วาด landmarks บนรูปภาพ
    func drawLandmarks(on image: UIImage, faces: [VNFaceObservation]) -> UIImage {
        let renderer = UIGraphicsImageRenderer(size: image.size)
        
        return renderer.image { ctx in
            image.draw(in: CGRect(origin: .zero, size: image.size))
            
            let context = ctx.cgContext
            
            for face in faces {
                guard let landmarks = face.landmarks else { continue }
                
                // แปลง face bounding box
                let faceBox = CGRect(
                    x: face.boundingBox.minX * image.size.width,
                    y: (1 - face.boundingBox.maxY) * image.size.height,
                    width: face.boundingBox.width * image.size.width,
                    height: face.boundingBox.height * image.size.height
                )
                
                // วาดแต่ละ landmark region
                let landmarkGroups: [(VNFaceLandmarkRegion2D?, UIColor)] = [
                    (landmarks.leftEye, .systemBlue),
                    (landmarks.rightEye, .systemBlue),
                    (landmarks.leftEyebrow, .systemGreen),
                    (landmarks.rightEyebrow, .systemGreen),
                    (landmarks.nose, .systemOrange),
                    (landmarks.noseCrest, .systemOrange),
                    (landmarks.outerLips, .systemRed),
                    (landmarks.innerLips, .systemPink),
                    (landmarks.leftPupil, .systemCyan),
                    (landmarks.rightPupil, .systemCyan),
                    (landmarks.faceContour, .systemYellow),
                    (landmarks.medianLine, .systemPurple)
                ]
                
                for (region, color) in landmarkGroups {
                    guard let region = region, region.pointCount > 0 else { continue }
                    drawLandmarkRegion(region, in: faceBox, on: context, color: color)
                }
            }
        }
    }
    
    private func drawLandmarkRegion(
        _ region: VNFaceLandmarkRegion2D,
        in faceBox: CGRect,
        on context: CGContext,
        color: UIColor
    ) {
        let points = region.normalizedPoints
        guard !points.isEmpty else { return }
        
        context.setStrokeColor(color.cgColor)
        context.setFillColor(color.cgColor)
        context.setLineWidth(1.5)
        
        let path = UIBezierPath()
        
        // แปลง normalized points เป็น image coordinates
        func convert(_ point: CGPoint) -> CGPoint {
            let x = faceBox.minX + point.x * faceBox.width
            let y = faceBox.minY + (1 - point.y) * faceBox.height
            return CGPoint(x: x, y: y)
        }
        
        path.move(to: convert(points[0]))
        for i in 1..<points.count {
            path.addLine(to: convert(points[i]))
        }
        
        // ปิด path สำหรับ closed regions
        if region == VNFaceLandmarkRegion2D() { // check if closed
            path.close()
        }
        
        context.addPath(path.cgPath)
        context.strokePath()
        
        // วาดจุดที่แต่ละ landmark
        for point in points {
            let converted = convert(point)
            let dotRect = CGRect(x: converted.x - 2, y: converted.y - 2, width: 4, height: 4)
            context.fillEllipse(in: dotRect)
        }
    }
    
    // ดึง specific landmark data
    func extractFaceData(from observation: VNFaceObservation) -> [String: Any] {
        var data: [String: Any] = [
            "boundingBox": observation.boundingBox,
            "roll": observation.roll?.doubleValue ?? 0,
            "yaw": observation.yaw?.doubleValue ?? 0,
            "pitch": observation.pitch?.doubleValue ?? 0
        ]
        
        if let landmarks = observation.landmarks {
            data["hasLeftEye"] = landmarks.leftEye != nil
            data["hasRightEye"] = landmarks.rightEye != nil
            data["hasNose"] = landmarks.nose != nil
            data["hasMouth"] = landmarks.outerLips != nil
            
            // ตรวจสอบว่าตาเปิดหรือปิด (โดยประมาณ)
            if let leftEye = landmarks.leftEye {
                let points = leftEye.normalizedPoints
                if points.count >= 4 {
                    let eyeHeight = abs(points[1].y - points[5].y)
                    let eyeWidth = abs(points[0].x - points[3].x)
                    data["leftEyeOpenness"] = eyeWidth > 0 ? eyeHeight / eyeWidth : 0
                }
            }
        }
        
        return data
    }
}
```

---

## 50.7 Face Capture Quality

```swift
import Vision

class FaceCaptureQualityAnalyzer {
    
    // ตรวจสอบคุณภาพภาพใบหน้า
    func analyzeFaceQuality(
        in image: UIImage,
        completion: @escaping ([(VNFaceObservation, Float)]) -> Void
    ) {
        guard let cgImage = image.cgImage else {
            completion([])
            return
        }
        
        let request = VNDetectFaceCaptureQualityRequest { request, error in
            guard let observations = request.results as? [VNFaceObservation] else {
                DispatchQueue.main.async { completion([]) }
                return
            }
            
            let facesWithQuality = observations.compactMap { face -> (VNFaceObservation, Float)? in
                guard let quality = face.faceCaptureQuality else { return nil }
                return (face, quality)
            }
            
            DispatchQueue.main.async {
                completion(facesWithQuality.sorted { $0.1 > $1.1 })
            }
        }
        
        let handler = VNImageRequestHandler(cgImage: cgImage)
        DispatchQueue.global(qos: .userInitiated).async {
            try? handler.perform([request])
        }
    }
    
    // ประเมินคุณภาพ
    func qualityDescription(_ quality: Float) -> (String, UIColor) {
        switch quality {
        case 0.8...1.0: return ("ยอดเยี่ยม", .systemGreen)
        case 0.6..<0.8: return ("ดี", .systemBlue)
        case 0.4..<0.6: return ("พอใช้", .systemOrange)
        case 0.2..<0.4: return ("ต่ำ", .systemYellow)
        default:         return ("ไม่ดี", .systemRed)
        }
    }
    
    // เลือกภาพที่ดีที่สุดจากหลายภาพ
    func selectBestImage(
        from images: [UIImage],
        completion: @escaping (UIImage?) -> Void
    ) {
        var bestImage: UIImage?
        var bestQuality: Float = 0
        let group = DispatchGroup()
        let lock = NSLock()
        
        for image in images {
            group.enter()
            analyzeFaceQuality(in: image) { facesWithQuality in
                defer { group.leave() }
                
                guard let topFace = facesWithQuality.first else { return }
                
                lock.lock()
                if topFace.1 > bestQuality {
                    bestQuality = topFace.1
                    bestImage = image
                }
                lock.unlock()
            }
        }
        
        group.notify(queue: .main) {
            completion(bestImage)
        }
    }
}
```

---

## 50.8 Body Pose Estimation (VNDetectHumanBodyPoseRequest)

```swift
import UIKit
import Vision

class BodyPoseEstimator {
    
    // Joint names ทั้งหมด
    static let bodyJoints: [VNHumanBodyPoseObservation.JointName] = [
        .nose, .leftEye, .rightEye, .leftEar, .rightEar,
        .leftShoulder, .rightShoulder,
        .leftElbow, .rightElbow,
        .leftWrist, .rightWrist,
        .leftHip, .rightHip,
        .leftKnee, .rightKnee,
        .leftAnkle, .rightAnkle,
        .neck, .root
    ]
    
    // Skeleton connections
    static let skeletonConnections: [(VNHumanBodyPoseObservation.JointName, VNHumanBodyPoseObservation.JointName)] = [
        (.nose, .neck),
        (.neck, .leftShoulder), (.neck, .rightShoulder),
        (.leftShoulder, .leftElbow), (.leftElbow, .leftWrist),
        (.rightShoulder, .rightElbow), (.rightElbow, .rightWrist),
        (.neck, .root),
        (.root, .leftHip), (.root, .rightHip),
        (.leftHip, .leftKnee), (.leftKnee, .leftAnkle),
        (.rightHip, .rightKnee), (.rightKnee, .rightAnkle)
    ]
    
    // MARK: - Detect Body Pose
    func detectBodyPose(
        in image: UIImage,
        completion: @escaping ([VNHumanBodyPoseObservation]) -> Void
    ) {
        guard let cgImage = image.cgImage else {
            completion([])
            return
        }
        
        let request = VNDetectHumanBodyPoseRequest { request, error in
            let observations = request.results as? [VNHumanBodyPoseObservation] ?? []
            DispatchQueue.main.async { completion(observations) }
        }
        
        let handler = VNImageRequestHandler(
            cgImage: cgImage,
            orientation: CGImagePropertyOrientation(image.imageOrientation)
        )
        
        DispatchQueue.global(qos: .userInitiated).async {
            try? handler.perform([request])
        }
    }
    
    // MARK: - วาด Skeleton
    func drawSkeleton(on image: UIImage, observations: [VNHumanBodyPoseObservation]) -> UIImage {
        let renderer = UIGraphicsImageRenderer(size: image.size)
        
        return renderer.image { ctx in
            image.draw(in: CGRect(origin: .zero, size: image.size))
            
            for observation in observations {
                drawPersonSkeleton(observation, on: ctx.cgContext, imageSize: image.size)
            }
        }
    }
    
    private func drawPersonSkeleton(
        _ observation: VNHumanBodyPoseObservation,
        on context: CGContext,
        imageSize: CGSize
    ) {
        // ดึง recognized points
        guard let recognizedPoints = try? observation.recognizedPoints(.all) else { return }
        
        // Helper แปลง normalized coordinates
        func convert(_ point: VNRecognizedPoint) -> CGPoint? {
            guard point.confidence > 0.3 else { return nil }
            return CGPoint(
                x: point.location.x * imageSize.width,
                y: (1 - point.location.y) * imageSize.height
            )
        }
        
        // วาด connections (skeleton lines)
        context.setStrokeColor(UIColor.systemGreen.withAlphaComponent(0.8).cgColor)
        context.setLineWidth(3)
        
        for (joint1, joint2) in Self.skeletonConnections {
            guard let point1 = recognizedPoints[joint1],
                  let point2 = recognizedPoints[joint2],
                  let p1 = convert(point1),
                  let p2 = convert(point2) else { continue }
            
            context.move(to: p1)
            context.addLine(to: p2)
            context.strokePath()
        }
        
        // วาด joints (จุดต่อ)
        for jointName in Self.bodyJoints {
            guard let point = recognizedPoints[jointName],
                  let location = convert(point) else { continue }
            
            // สีตามระดับ confidence
            let color = point.confidence > 0.7 ? UIColor.systemBlue : UIColor.systemOrange
            context.setFillColor(color.cgColor)
            
            let radius: CGFloat = 6
            let dotRect = CGRect(
                x: location.x - radius,
                y: location.y - radius,
                width: radius * 2,
                height: radius * 2
            )
            context.fillEllipse(in: dotRect)
        }
    }
    
    // MARK: - Pose Analysis
    func analyzePose(_ observation: VNHumanBodyPoseObservation) -> PoseAnalysis {
        guard let points = try? observation.recognizedPoints(.all) else {
            return PoseAnalysis(isStanding: false, isRaisingHands: false, poseScore: 0)
        }
        
        // ตรวจสอบว่ากำลังยืนอยู่
        let isStanding: Bool = {
            guard let leftAnkle = points[.leftAnkle],
                  let rightAnkle = points[.rightAnkle],
                  let leftHip = points[.leftHip],
                  let rightHip = points[.rightHip] else { return false }
            
            let ankleY = (leftAnkle.location.y + rightAnkle.location.y) / 2
            let hipY = (leftHip.location.y + rightHip.location.y) / 2
            
            return hipY > ankleY + 0.2
        }()
        
        // ตรวจสอบว่ายกมืออยู่
        let isRaisingHands: Bool = {
            guard let leftWrist = points[.leftWrist],
                  let rightWrist = points[.rightWrist],
                  let leftShoulder = points[.leftShoulder],
                  let rightShoulder = points[.rightShoulder] else { return false }
            
            return leftWrist.location.y > leftShoulder.location.y &&
                   rightWrist.location.y > rightShoulder.location.y
        }()
        
        // คำนวณ pose score (ค่าเฉลี่ยของ confidence)
        let confidences = Self.bodyJoints.compactMap { points[$0]?.confidence }
        let avgConfidence = confidences.isEmpty ? 0 : Float(confidences.reduce(0, +)) / Float(confidences.count)
        
        return PoseAnalysis(
            isStanding: isStanding,
            isRaisingHands: isRaisingHands,
            poseScore: avgConfidence
        )
    }
    
    struct PoseAnalysis {
        let isStanding: Bool
        let isRaisingHands: Bool
        let poseScore: Float
        
        var description: String {
            var parts: [String] = []
            if isStanding { parts.append("กำลังยืน") }
            if isRaisingHands { parts.append("ยกมือ") }
            if parts.isEmpty { parts.append("ท่าทางอื่น") }
            return parts.joined(separator: ", ") + " (score: \(String(format: "%.0f%%", poseScore * 100)))"
        }
    }
}
```

---

## 50.9 Hand Pose Estimation

```swift
import Vision
import UIKit

class HandPoseEstimator {
    
    // Hand joint names
    static let handJoints: [[VNHumanHandPoseObservation.JointName]] = [
        // Thumb
        [.thumbCMC, .thumbMP, .thumbIP, .thumbTip],
        // Index finger
        [.indexMCP, .indexPIP, .indexDIP, .indexTip],
        // Middle finger
        [.middleMCP, .middlePIP, .middleDIP, .middleTip],
        // Ring finger
        [.ringMCP, .ringPIP, .ringDIP, .ringTip],
        // Little finger
        [.littleMCP, .littlePIP, .littleDIP, .littleTip]
    ]
    
    // MARK: - Detect Hand Pose
    func detectHandPose(
        in image: UIImage,
        maximumHands: Int = 2,
        completion: @escaping ([VNHumanHandPoseObservation]) -> Void
    ) {
        guard let cgImage = image.cgImage else {
            completion([])
            return
        }
        
        let request = VNDetectHumanHandPoseRequest { request, error in
            let observations = request.results as? [VNHumanHandPoseObservation] ?? []
            DispatchQueue.main.async { completion(observations) }
        }
        request.maximumHandCount = maximumHands
        
        let handler = VNImageRequestHandler(cgImage: cgImage)
        DispatchQueue.global(qos: .userInitiated).async {
            try? handler.perform([request])
        }
    }
    
    // MARK: - นับนิ้วที่ยกขึ้น
    func countRaisedFingers(_ observation: VNHumanHandPoseObservation) -> Int {
        guard let allPoints = try? observation.recognizedPoints(.all) else { return 0 }
        
        var raisedCount = 0
        
        // ตรวจสอบแต่ละนิ้ว (ยกเว้นหัวแม่มือ)
        let fingerTips: [(VNHumanHandPoseObservation.JointName, VNHumanHandPoseObservation.JointName)] = [
            (.indexTip, .indexMCP),
            (.middleTip, .middleMCP),
            (.ringTip, .ringMCP),
            (.littleTip, .littleMCP)
        ]
        
        for (tip, base) in fingerTips {
            guard let tipPoint = allPoints[tip],
                  let basePoint = allPoints[base],
                  tipPoint.confidence > 0.5,
                  basePoint.confidence > 0.5 else { continue }
            
            // ถ้า tip อยู่สูงกว่า base = นิ้วยกขึ้น
            if tipPoint.location.y > basePoint.location.y {
                raisedCount += 1
            }
        }
        
        return raisedCount
    }
    
    // MARK: - ตรวจจับ gestures
    func detectGesture(_ observation: VNHumanHandPoseObservation) -> HandGesture {
        let fingerCount = countRaisedFingers(observation)
        
        guard let allPoints = try? observation.recognizedPoints(.all) else { return .unknown }
        
        // Thumbs up
        if let thumbTip = allPoints[.thumbTip],
           let thumbMCP = allPoints[.thumbMCP],
           thumbTip.confidence > 0.5, thumbMCP.confidence > 0.5,
           thumbTip.location.y > thumbMCP.location.y + 0.1,
           fingerCount == 0 {
            return .thumbsUp
        }
        
        // Peace sign (2 นิ้ว)
        if fingerCount == 2 {
            if let indexTip = allPoints[.indexTip],
               let middleTip = allPoints[.middleTip],
               indexTip.confidence > 0.5, middleTip.confidence > 0.5 {
                return .peace
            }
        }
        
        switch fingerCount {
        case 0: return .fist
        case 1: return .oneFingerPoint
        case 5: return .openHand
        default: return .unknown
        }
    }
    
    enum HandGesture: String {
        case fist = "กำหมัด ✊"
        case thumbsUp = "ยกนิ้วโป้ง 👍"
        case peace = "Victory ✌️"
        case oneFingerPoint = "ชี้นิ้ว ☝️"
        case openHand = "ฝ่ามือ ✋"
        case unknown = "ท่าทางไม่ชัดเจน"
    }
    
    // MARK: - วาด Hand Skeleton
    func drawHandSkeleton(on image: UIImage, observations: [VNHumanHandPoseObservation]) -> UIImage {
        let renderer = UIGraphicsImageRenderer(size: image.size)
        
        let fingerColors: [UIColor] = [.systemRed, .systemBlue, .systemGreen, .systemOrange, .systemPurple]
        
        return renderer.image { ctx in
            image.draw(in: CGRect(origin: .zero, size: image.size))
            
            let context = ctx.cgContext
            
            for observation in observations {
                guard let allPoints = try? observation.recognizedPoints(.all) else { continue }
                
                func convert(_ point: VNRecognizedPoint) -> CGPoint? {
                    guard point.confidence > 0.3 else { return nil }
                    return CGPoint(
                        x: point.location.x * image.size.width,
                        y: (1 - point.location.y) * image.size.height
                    )
                }
                
                // วาดแต่ละนิ้ว
                for (fingerIndex, finger) in Self.handJoints.enumerated() {
                    let color = fingerColors[fingerIndex % fingerColors.count]
                    context.setStrokeColor(color.cgColor)
                    context.setFillColor(color.cgColor)
                    context.setLineWidth(2.5)
                    
                    var prevPoint: CGPoint?
                    
                    for jointName in finger {
                        guard let joint = allPoints[jointName],
                              let location = convert(joint) else { continue }
                        
                        // วาดเส้นเชื่อม
                        if let prev = prevPoint {
                            context.move(to: prev)
                            context.addLine(to: location)
                            context.strokePath()
                        }
                        prevPoint = location
                        
                        // วาดจุด
                        let radius: CGFloat = 5
                        context.fillEllipse(in: CGRect(
                            x: location.x - radius,
                            y: location.y - radius,
                            width: radius * 2,
                            height: radius * 2
                        ))
                    }
                }
            }
        }
    }
}
```

---

## 50.10 Object Tracking (VNTrackObjectRequest)

```swift
import UIKit
import Vision
import AVFoundation

class ObjectTracker {
    
    private let sequenceHandler = VNSequenceRequestHandler()
    private var trackingRequest: VNTrackObjectRequest?
    private var isTracking = false
    
    var onTrackingUpdate: ((CGRect, Float) -> Void)?
    var onTrackingLost: (() -> Void)?
    
    // เริ่ม tracking จาก bounding box ที่กำหนด
    func startTracking(boundingBox: CGRect, in frame: CVPixelBuffer) {
        let observation = VNDetectedObjectObservation(boundingBox: boundingBox)
        trackingRequest = VNTrackObjectRequest(detectedObjectObservation: observation)
        trackingRequest?.trackingLevel = .accurate
        isTracking = true
        
        // Track ใน frame แรก
        processFrame(frame)
    }
    
    // ประมวลผลแต่ละ frame
    func processFrame(_ pixelBuffer: CVPixelBuffer) {
        guard isTracking, let request = trackingRequest else { return }
        
        do {
            try sequenceHandler.perform([request], on: pixelBuffer)
            
            if let result = request.results?.first as? VNDetectedObjectObservation {
                if result.confidence > 0.3 {
                    // อัพเดต tracking
                    onTrackingUpdate?(result.boundingBox, result.confidence)
                    
                    // อัพเดต request สำหรับ frame ถัดไป
                    let newObservation = VNDetectedObjectObservation(boundingBox: result.boundingBox)
                    trackingRequest = VNTrackObjectRequest(detectedObjectObservation: newObservation)
                    trackingRequest?.trackingLevel = .accurate
                } else {
                    // Tracking lost
                    stopTracking()
                    onTrackingLost?()
                }
            }
        } catch {
            print("Tracking error: \(error)")
            stopTracking()
            onTrackingLost?()
        }
    }
    
    func stopTracking() {
        isTracking = false
        trackingRequest = nil
    }
    
    // MARK: - ViewController ที่ใช้ Object Tracking
    class ObjectTrackingViewController: UIViewController {
        
        private let captureSession = AVCaptureSession()
        private var previewLayer: AVCaptureVideoPreviewLayer!
        private let videoOutput = AVCaptureVideoDataOutput()
        
        private let tracker = ObjectTracker()
        private let trackingBoxLayer = CALayer()
        
        private var isSelectingObject = false
        private var selectedRect: CGRect?
        
        override func viewDidLoad() {
            super.viewDidLoad()
            setupCamera()
            setupUI()
            setupTracker()
        }
        
        private func setupCamera() {
            captureSession.sessionPreset = .hd1280x720
            
            guard let camera = AVCaptureDevice.default(for: .video),
                  let input = try? AVCaptureDeviceInput(device: camera) else { return }
            
            captureSession.addInput(input)
            
            previewLayer = AVCaptureVideoPreviewLayer(session: captureSession)
            previewLayer.frame = view.bounds
            previewLayer.videoGravity = .resizeAspectFill
            view.layer.addSublayer(previewLayer)
            
            trackingBoxLayer.frame = view.bounds
            trackingBoxLayer.borderWidth = 0
            view.layer.addSublayer(trackingBoxLayer)
            
            videoOutput.setSampleBufferDelegate(self, queue: DispatchQueue(label: "tracking"))
            videoOutput.alwaysDiscardsLateVideoFrames = true
            captureSession.addOutput(videoOutput)
            
            DispatchQueue.global(qos: .background).async { [weak self] in
                self?.captureSession.startRunning()
            }
        }
        
        private func setupUI() {
            let tapGesture = UITapGestureRecognizer(target: self, action: #selector(handleTap))
            view.addGestureRecognizer(tapGesture)
            
            let instructionLabel = UILabel()
            instructionLabel.text = "แตะเพื่อเลือกวัตถุที่ต้องการติดตาม"
            instructionLabel.textColor = .white
            instructionLabel.backgroundColor = .black.withAlphaComponent(0.6)
            instructionLabel.textAlignment = .center
            instructionLabel.frame = CGRect(x: 0, y: view.bounds.height - 80, width: view.bounds.width, height: 44)
            view.addSubview(instructionLabel)
        }
        
        private func setupTracker() {
            tracker.onTrackingUpdate = { [weak self] boundingBox, confidence in
                DispatchQueue.main.async {
                    self?.updateTrackingBox(boundingBox, confidence: confidence)
                }
            }
            
            tracker.onTrackingLost = { [weak self] in
                DispatchQueue.main.async {
                    self?.trackingBoxLayer.sublayers?.removeAll()
                    print("Tracking lost!")
                }
            }
        }
        
        @objc private func handleTap(_ gesture: UITapGestureRecognizer) {
            let tapPoint = gesture.location(in: view)
            // เริ่ม track บริเวณที่แตะ (กำหนดขนาด 100x100)
            let size: CGFloat = 100
            let normalizedBox = CGRect(
                x: (tapPoint.x - size/2) / view.bounds.width,
                y: (tapPoint.y - size/2) / view.bounds.height,
                width: size / view.bounds.width,
                height: size / view.bounds.height
            )
            
            // แปลงจาก UIKit coordinates เป็น Vision coordinates
            let visionBox = CGRect(
                x: normalizedBox.minX,
                y: 1 - normalizedBox.maxY,
                width: normalizedBox.width,
                height: normalizedBox.height
            )
            
            isSelectingObject = true
            selectedRect = visionBox
        }
        
        private func updateTrackingBox(_ box: CGRect, confidence: Float) {
            trackingBoxLayer.sublayers?.removeAll()
            
            let x = box.minX * view.bounds.width
            let y = (1 - box.maxY) * view.bounds.height
            let width = box.width * view.bounds.width
            let height = box.height * view.bounds.height
            
            let boxLayer = CALayer()
            boxLayer.frame = CGRect(x: x, y: y, width: width, height: height)
            boxLayer.borderWidth = 3
            boxLayer.borderColor = confidence > 0.7 ? UIColor.green.cgColor : UIColor.orange.cgColor
            
            trackingBoxLayer.addSublayer(boxLayer)
        }
    }
}

extension ObjectTracker.ObjectTrackingViewController: AVCaptureVideoDataOutputSampleBufferDelegate {
    func captureOutput(_ output: AVCaptureOutput, didOutput sampleBuffer: CMSampleBuffer, from connection: AVCaptureConnection) {
        guard let pixelBuffer = CMSampleBufferGetImageBuffer(sampleBuffer) else { return }
        
        if isSelectingObject, let rect = selectedRect {
            tracker.startTracking(boundingBox: rect, in: pixelBuffer)
            isSelectingObject = false
            selectedRect = nil
        } else {
            tracker.processFrame(pixelBuffer)
        }
    }
}
```

---

## 50.11 Rectangle Detection

```swift
import Vision
import UIKit

class RectangleDetector {
    
    // ตรวจจับสี่เหลี่ยม
    func detectRectangles(
        in image: UIImage,
        minimumAspectRatio: Float = 0.3,
        maximumAspectRatio: Float = 1.0,
        minimumSize: Float = 0.1,
        maximumObservations: Int = 10,
        completion: @escaping ([VNRectangleObservation]) -> Void
    ) {
        guard let cgImage = image.cgImage else {
            completion([])
            return
        }
        
        let request = VNDetectRectanglesRequest { request, error in
            let rects = request.results as? [VNRectangleObservation] ?? []
            DispatchQueue.main.async { completion(rects) }
        }
        
        // กำหนด parameters
        request.minimumAspectRatio = VNAspectRatio(minimumAspectRatio)
        request.maximumAspectRatio = VNAspectRatio(maximumAspectRatio)
        request.minimumSize = Float(minimumSize)
        request.maximumObservations = maximumObservations
        request.minimumConfidence = 0.5
        
        let handler = VNImageRequestHandler(cgImage: cgImage)
        DispatchQueue.global(qos: .userInitiated).async {
            try? handler.perform([request])
        }
    }
    
    // แก้ไขมุมมองของสี่เหลี่ยม (perspective correction)
    func perspectiveCorrect(
        image: UIImage,
        observation: VNRectangleObservation
    ) -> UIImage? {
        guard let ciImage = CIImage(image: image) else { return nil }
        
        let imageSize = ciImage.extent.size
        
        // แปลง normalized coordinates เป็น image coordinates
        func convertPoint(_ point: CGPoint) -> CIVector {
            return CIVector(
                x: point.x * imageSize.width,
                y: point.y * imageSize.height
            )
        }
        
        // ใช้ CIPerspectiveCorrection filter
        guard let filter = CIFilter(name: "CIPerspectiveCorrection") else { return nil }
        filter.setValue(ciImage, forKey: kCIInputImageKey)
        filter.setValue(convertPoint(observation.topLeft), forKey: "inputTopLeft")
        filter.setValue(convertPoint(observation.topRight), forKey: "inputTopRight")
        filter.setValue(convertPoint(observation.bottomLeft), forKey: "inputBottomLeft")
        filter.setValue(convertPoint(observation.bottomRight), forKey: "inputBottomRight")
        
        guard let outputImage = filter.outputImage else { return nil }
        
        let context = CIContext()
        guard let cgImage = context.createCGImage(outputImage, from: outputImage.extent) else { return nil }
        
        return UIImage(cgImage: cgImage)
    }
}
```

---

## 50.12 Text Recognition (VNRecognizeTextRequest)

```swift
import Vision
import UIKit

class TextRecognizer {
    
    // MARK: - Basic Text Recognition
    func recognizeText(
        in image: UIImage,
        recognitionLevel: VNRequestTextRecognitionLevel = .accurate,
        languages: [String] = ["th", "en"],
        completion: @escaping ([String]) -> Void
    ) {
        guard let cgImage = image.cgImage else {
            completion([])
            return
        }
        
        let request = VNRecognizeTextRequest { request, error in
            guard let observations = request.results as? [VNRecognizedTextObservation] else {
                DispatchQueue.main.async { completion([]) }
                return
            }
            
            let texts = observations.compactMap { observation -> String? in
                observation.topCandidates(1).first?.string
            }
            
            DispatchQueue.main.async { completion(texts) }
        }
        
        // กำหนดค่า
        request.recognitionLevel = recognitionLevel  // .accurate หรือ .fast
        request.recognitionLanguages = languages      // ภาษาที่ต้องการรองรับ
        request.usesLanguageCorrection = true         // แก้ไขคำผิดอัตโนมัติ
        request.minimumTextHeight = 0.01              // ขนาดขั้นต่ำของข้อความ
        
        let handler = VNImageRequestHandler(
            cgImage: cgImage,
            orientation: CGImagePropertyOrientation(image.imageOrientation)
        )
        
        DispatchQueue.global(qos: .userInitiated).async {
            try? handler.perform([request])
        }
    }
    
    // MARK: - Text Recognition พร้อม bounding boxes
    func recognizeTextWithBoxes(
        in image: UIImage,
        completion: @escaping ([(String, CGRect, Float)]) -> Void
    ) {
        guard let cgImage = image.cgImage else {
            completion([])
            return
        }
        
        let request = VNRecognizeTextRequest { request, error in
            guard let observations = request.results as? [VNRecognizedTextObservation] else {
                DispatchQueue.main.async { completion([]) }
                return
            }
            
            let results = observations.compactMap { obs -> (String, CGRect, Float)? in
                guard let candidate = obs.topCandidates(1).first else { return nil }
                
                // คำนวณ bounding box ของข้อความทั้งหมดใน observation
                let bbox = obs.boundingBox
                
                return (candidate.string, bbox, candidate.confidence)
            }
            
            DispatchQueue.main.async { completion(results) }
        }
        
        request.recognitionLevel = .accurate
        request.recognitionLanguages = ["th", "en"]
        request.usesLanguageCorrection = true
        
        let handler = VNImageRequestHandler(cgImage: cgImage)
        DispatchQueue.global(qos: .userInitiated).async {
            try? handler.perform([request])
        }
    }
    
    // MARK: - วาดกรอบข้อความ
    func drawTextBoxes(on image: UIImage, results: [(String, CGRect, Float)]) -> UIImage {
        let renderer = UIGraphicsImageRenderer(size: image.size)
        
        return renderer.image { ctx in
            image.draw(in: CGRect(origin: .zero, size: image.size))
            
            let context = ctx.cgContext
            context.setStrokeColor(UIColor.systemBlue.cgColor)
            context.setLineWidth(2)
            
            for (text, box, confidence) in results {
                let rect = CGRect(
                    x: box.minX * image.size.width,
                    y: (1 - box.maxY) * image.size.height,
                    width: box.width * image.size.width,
                    height: box.height * image.size.height
                )
                
                // วาดกรอบ
                context.stroke(rect)
                
                // แสดงข้อความและความมั่นใจ
                let label = "\(text) (\(Int(confidence * 100))%)"
                let attrs: [NSAttributedString.Key: Any] = [
                    .foregroundColor: UIColor.white,
                    .backgroundColor: UIColor.systemBlue.withAlphaComponent(0.7),
                    .font: UIFont.systemFont(ofSize: 10)
                ]
                label.draw(at: CGPoint(x: rect.minX, y: rect.minY - 16), withAttributes: attrs)
            }
        }
    }
    
    // MARK: - Text Recognition ViewModel สำหรับ SwiftUI
    @MainActor
    class TextRecognizerViewModel: ObservableObject {
        @Published var recognizedText: String = ""
        @Published var textBlocks: [(String, CGRect, Float)] = []
        @Published var isProcessing: Bool = false
        @Published var annotatedImage: UIImage?
        
        private let recognizer = TextRecognizer()
        
        func processImage(_ image: UIImage) {
            isProcessing = true
            recognizedText = ""
            textBlocks = []
            
            recognizer.recognizeTextWithBoxes(in: image) { [weak self] results in
                guard let self = self else { return }
                
                self.isProcessing = false
                self.textBlocks = results
                self.recognizedText = results.map { $0.0 }.joined(separator: "\n")
                self.annotatedImage = self.recognizer.drawTextBoxes(on: image, results: results)
            }
        }
    }
}
```

---

## 50.13 Barcode Detection (VNDetectBarcodesRequest)

```swift
import Vision
import UIKit

class BarcodeDetector {
    
    // Barcode symbologies ที่รองรับ
    static let supportedSymbologies: [VNBarcodeSymbology] = [
        .QR,
        .aztec,
        .code128,
        .code39,
        .code93,
        .dataMatrix,
        .ean8,
        .ean13,
        .itf14,
        .pdf417,
        .upce
    ]
    
    // MARK: - ตรวจจับ Barcode/QR Code
    func detectBarcodes(
        in image: UIImage,
        symbologies: [VNBarcodeSymbology] = [],
        completion: @escaping ([VNBarcodeObservation]) -> Void
    ) {
        guard let cgImage = image.cgImage else {
            completion([])
            return
        }
        
        let request = VNDetectBarcodesRequest { request, error in
            let barcodes = request.results as? [VNBarcodeObservation] ?? []
            DispatchQueue.main.async { completion(barcodes) }
        }
        
        // กำหนด symbologies (ถ้าไม่ระบุจะตรวจสอบทุกประเภท)
        if !symbologies.isEmpty {
            request.symbologies = symbologies
        }
        
        let handler = VNImageRequestHandler(
            cgImage: cgImage,
            orientation: CGImagePropertyOrientation(image.imageOrientation)
        )
        
        DispatchQueue.global(qos: .userInitiated).async {
            try? handler.perform([request])
        }
    }
    
    // MARK: - Real-time Barcode Scanner
    class BarcodeScannerViewController: UIViewController {
        
        private let session = AVCaptureSession()
        private var previewLayer: AVCaptureVideoPreviewLayer!
        private let output = AVCaptureVideoDataOutput()
        private let detector = BarcodeDetector()
        
        var onBarcodeDetected: ((String, VNBarcodeSymbology) -> Void)?
        
        private var lastDetectedPayload: String?
        private var lastDetectionTime = Date.distantPast
        private let cooldownInterval: TimeInterval = 2.0  // 2 วินาทีก่อน scan ซ้ำ
        
        // Overlay layer
        private let overlayLayer = CALayer()
        
        override func viewDidLoad() {
            super.viewDidLoad()
            setupCamera()
            setupOverlay()
        }
        
        private func setupCamera() {
            session.sessionPreset = .hd1280x720
            
            guard let camera = AVCaptureDevice.default(for: .video),
                  let input = try? AVCaptureDeviceInput(device: camera) else { return }
            
            session.addInput(input)
            
            previewLayer = AVCaptureVideoPreviewLayer(session: session)
            previewLayer.frame = view.bounds
            previewLayer.videoGravity = .resizeAspectFill
            view.layer.addSublayer(previewLayer)
            
            output.setSampleBufferDelegate(self, queue: DispatchQueue(label: "barcode"))
            output.alwaysDiscardsLateVideoFrames = true
            session.addOutput(output)
            
            DispatchQueue.global(qos: .background).async { [weak self] in
                self?.session.startRunning()
            }
        }
        
        private func setupOverlay() {
            overlayLayer.frame = view.bounds
            view.layer.addSublayer(overlayLayer)
            
            // Viewfinder
            let viewfinderSize = CGSize(width: 250, height: 250)
            let viewfinderRect = CGRect(
                x: (view.bounds.width - viewfinderSize.width) / 2,
                y: (view.bounds.height - viewfinderSize.height) / 2,
                width: viewfinderSize.width,
                height: viewfinderSize.height
            )
            
            let viewfinderLayer = CALayer()
            viewfinderLayer.frame = viewfinderRect
            viewfinderLayer.borderWidth = 2
            viewfinderLayer.borderColor = UIColor.white.cgColor
            viewfinderLayer.cornerRadius = 12
            overlayLayer.addSublayer(viewfinderLayer)
        }
        
        func processBarcode(_ observation: VNBarcodeObservation) {
            guard let payload = observation.payloadStringValue else { return }
            
            // Cooldown check
            let now = Date()
            guard now.timeIntervalSince(lastDetectionTime) >= cooldownInterval ||
                  payload != lastDetectedPayload else { return }
            
            lastDetectedPayload = payload
            lastDetectionTime = now
            
            DispatchQueue.main.async { [weak self] in
                // Haptic feedback
                let feedback = UINotificationFeedbackGenerator()
                feedback.notificationOccurred(.success)
                
                self?.onBarcodeDetected?(payload, observation.symbology)
            }
        }
    }
}

extension BarcodeDetector.BarcodeScannerViewController: AVCaptureVideoDataOutputSampleBufferDelegate {
    func captureOutput(_ output: AVCaptureOutput, didOutput sampleBuffer: CMSampleBuffer, from connection: AVCaptureConnection) {
        guard let pixelBuffer = CMSampleBufferGetImageBuffer(sampleBuffer) else { return }
        
        let request = VNDetectBarcodesRequest { [weak self] request, _ in
            guard let barcodes = request.results as? [VNBarcodeObservation],
                  let first = barcodes.first else { return }
            self?.processBarcode(first)
        }
        request.symbologies = [.QR, .ean13, .code128]
        
        try? VNImageRequestHandler(cvPixelBuffer: pixelBuffer).perform([request])
    }
}
```

---

## 50.14 QR Code Reading

```swift
import Vision
import UIKit

// MARK: - QR Code Reader
class QRCodeReader {
    
    struct QRCodeResult {
        let content: String
        let boundingBox: CGRect
        let cornerPoints: [CGPoint]
        
        // พยายาม parse เป็น URL
        var url: URL? { URL(string: content) }
        
        // พยายาม parse เป็น JSON
        var jsonData: [String: Any]? {
            guard let data = content.data(using: .utf8),
                  let json = try? JSONSerialization.jsonObject(with: data) as? [String: Any] else {
                return nil
            }
            return json
        }
        
        // ตรวจสอบว่าเป็น URL หรือไม่
        var isURL: Bool { url != nil }
        
        // ตรวจสอบว่าเป็น WiFi QR Code
        var wifiConfig: WiFiConfig? {
            guard content.hasPrefix("WIFI:") else { return nil }
            return WiFiConfig(qrString: content)
        }
    }
    
    struct WiFiConfig {
        let ssid: String
        let password: String
        let security: String
        let isHidden: Bool
        
        init?(qrString: String) {
            // WIFI:T:WPA;S:MyNetwork;P:MyPassword;H:false;;
            let components = qrString
                .replacingOccurrences(of: "WIFI:", with: "")
                .components(separatedBy: ";")
                .filter { !$0.isEmpty }
            
            var dict: [String: String] = [:]
            for component in components {
                let parts = component.components(separatedBy: ":")
                if parts.count >= 2 {
                    dict[parts[0]] = parts[1...].joined(separator: ":")
                }
            }
            
            guard let ssid = dict["S"] else { return nil }
            self.ssid = ssid
            self.password = dict["P"] ?? ""
            self.security = dict["T"] ?? "nopass"
            self.isHidden = dict["H"] == "true"
        }
    }
    
    func readQRCodes(
        from image: UIImage,
        completion: @escaping ([QRCodeResult]) -> Void
    ) {
        guard let cgImage = image.cgImage else {
            completion([])
            return
        }
        
        let request = VNDetectBarcodesRequest { request, error in
            guard let observations = request.results as? [VNBarcodeObservation] else {
                DispatchQueue.main.async { completion([]) }
                return
            }
            
            let results = observations
                .filter { $0.symbology == .QR }
                .compactMap { obs -> QRCodeResult? in
                    guard let payload = obs.payloadStringValue else { return nil }
                    
                    let corners = [
                        obs.topLeft, obs.topRight,
                        obs.bottomRight, obs.bottomLeft
                    ]
                    
                    return QRCodeResult(
                        content: payload,
                        boundingBox: obs.boundingBox,
                        cornerPoints: corners
                    )
                }
            
            DispatchQueue.main.async { completion(results) }
        }
        request.symbologies = [.QR]
        
        let handler = VNImageRequestHandler(cgImage: cgImage)
        DispatchQueue.global(qos: .userInitiated).async {
            try? handler.perform([request])
        }
    }
}
```

---

## 50.15 Document Scanning

```swift
import Vision
import UIKit
import VisionKit

// MARK: - Document Scanner using VisionKit
class DocumentScannerManager: NSObject {
    
    weak var presentingViewController: UIViewController?
    var onScanComplete: ((VNDocumentCameraScan) -> Void)?
    
    // ใช้ VNDocumentCameraViewController สำหรับ scanning UI
    func startScanning(from viewController: UIViewController) {
        guard VNDocumentCameraViewController.isSupported else {
            print("Document scanning ไม่รองรับในอุปกรณ์นี้")
            return
        }
        
        presentingViewController = viewController
        
        let scanner = VNDocumentCameraViewController()
        scanner.delegate = self
        viewController.present(scanner, animated: true)
    }
    
    // ประมวลผลภาพที่สแกนได้
    func processScannedDocument(_ scan: VNDocumentCameraScan) async -> [ScannedPage] {
        var pages: [ScannedPage] = []
        
        for i in 0..<scan.pageCount {
            let image = scan.imageOfPage(at: i)
            let texts = await recognizeText(in: image)
            pages.append(ScannedPage(image: image, text: texts.joined(separator: "\n"), pageNumber: i + 1))
        }
        
        return pages
    }
    
    private func recognizeText(in image: UIImage) async -> [String] {
        return await withCheckedContinuation { continuation in
            guard let cgImage = image.cgImage else {
                continuation.resume(returning: [])
                return
            }
            
            let request = VNRecognizeTextRequest { request, _ in
                let texts = (request.results as? [VNRecognizedTextObservation] ?? [])
                    .compactMap { $0.topCandidates(1).first?.string }
                continuation.resume(returning: texts)
            }
            
            request.recognitionLevel = .accurate
            request.recognitionLanguages = ["th", "en"]
            request.usesLanguageCorrection = true
            
            let handler = VNImageRequestHandler(cgImage: cgImage)
            try? handler.perform([request])
        }
    }
    
    struct ScannedPage {
        let image: UIImage
        let text: String
        let pageNumber: Int
    }
}

extension DocumentScannerManager: VNDocumentCameraViewControllerDelegate {
    func documentCameraViewController(
        _ controller: VNDocumentCameraViewController,
        didFinishWith scan: VNDocumentCameraScan
    ) {
        controller.dismiss(animated: true) {
            self.onScanComplete?(scan)
        }
    }
    
    func documentCameraViewControllerDidCancel(_ controller: VNDocumentCameraViewController) {
        controller.dismiss(animated: true)
    }
    
    func documentCameraViewController(
        _ controller: VNDocumentCameraViewController,
        didFailWithError error: Error
    ) {
        controller.dismiss(animated: true)
        print("Document scanning failed: \(error)")
    }
}

// MARK: - Custom Document Scanner (ไม่ใช้ VisionKit)
class CustomDocumentScanner {
    
    func scanDocument(in image: UIImage) async -> UIImage? {
        guard let cgImage = image.cgImage else { return nil }
        
        return await withCheckedContinuation { continuation in
            let request = VNDetectRectanglesRequest { request, _ in
                guard let rectangles = request.results as? [VNRectangleObservation],
                      let largest = rectangles.max(by: {
                          $0.boundingBox.width * $0.boundingBox.height <
                          $1.boundingBox.width * $1.boundingBox.height
                      }) else {
                    continuation.resume(returning: nil)
                    return
                }
                
                // Perspective correction
                let corrected = self.applyPerspectiveCorrection(
                    to: image,
                    rectangle: largest
                )
                continuation.resume(returning: corrected)
            }
            
            request.minimumAspectRatio = 0.3
            request.maximumAspectRatio = 0.95
            request.minimumSize = 0.2
            request.maximumObservations = 1
            request.minimumConfidence = 0.7
            
            let handler = VNImageRequestHandler(cgImage: cgImage)
            try? handler.perform([request])
        }
    }
    
    private func applyPerspectiveCorrection(
        to image: UIImage,
        rectangle: VNRectangleObservation
    ) -> UIImage? {
        guard let ciImage = CIImage(image: image) else { return nil }
        
        let size = ciImage.extent.size
        
        func toVector(_ point: CGPoint) -> CIVector {
            CIVector(x: point.x * size.width, y: point.y * size.height)
        }
        
        guard let filter = CIFilter(name: "CIPerspectiveCorrection") else { return nil }
        filter.setValue(ciImage, forKey: kCIInputImageKey)
        filter.setValue(toVector(rectangle.topLeft), forKey: "inputTopLeft")
        filter.setValue(toVector(rectangle.topRight), forKey: "inputTopRight")
        filter.setValue(toVector(rectangle.bottomLeft), forKey: "inputBottomLeft")
        filter.setValue(toVector(rectangle.bottomRight), forKey: "inputBottomRight")
        
        guard let outputCIImage = filter.outputImage else { return nil }
        
        // เพิ่ม enhancement
        let enhanced = outputCIImage
            .applyingFilter("CIColorControls", parameters: [
                "inputContrast": 1.2,
                "inputBrightness": 0.05,
                "inputSaturation": 0.0  // ทำให้เป็นขาวดำ
            ])
        
        let context = CIContext()
        guard let cgOutput = context.createCGImage(enhanced, from: enhanced.extent) else { return nil }
        
        return UIImage(cgImage: cgOutput)
    }
}
```

---

## 50.16 Image Saliency

```swift
import Vision
import UIKit

class ImageSaliencyAnalyzer {
    
    // Attention-based Saliency (ส่วนที่ดึงดูดความสนใจ)
    func generateAttentionSaliency(
        for image: UIImage,
        completion: @escaping (VNSaliencyImageObservation?) -> Void
    ) {
        guard let cgImage = image.cgImage else {
            completion(nil)
            return
        }
        
        let request = VNGenerateAttentionBasedSaliencyImageRequest { request, error in
            let observation = request.results?.first as? VNSaliencyImageObservation
            DispatchQueue.main.async { completion(observation) }
        }
        
        let handler = VNImageRequestHandler(cgImage: cgImage)
        DispatchQueue.global(qos: .userInitiated).async {
            try? handler.perform([request])
        }
    }
    
    // Objectness-based Saliency (ตำแหน่งของวัตถุ)
    func generateObjectnessSaliency(
        for image: UIImage,
        completion: @escaping (VNSaliencyImageObservation?) -> Void
    ) {
        guard let cgImage = image.cgImage else {
            completion(nil)
            return
        }
        
        let request = VNGenerateObjectnessBasedSaliencyImageRequest { request, error in
            let observation = request.results?.first as? VNSaliencyImageObservation
            DispatchQueue.main.async { completion(observation) }
        }
        
        let handler = VNImageRequestHandler(cgImage: cgImage)
        DispatchQueue.global(qos: .userInitiated).async {
            try? handler.perform([request])
        }
    }
    
    // สร้าง heatmap จาก saliency
    func createSaliencyHeatmap(
        for image: UIImage,
        observation: VNSaliencyImageObservation,
        alpha: CGFloat = 0.6
    ) -> UIImage? {
        guard let pixelBuffer = observation.pixelBuffer else { return nil }
        
        let ciImage = CIImage(cvPixelBuffer: pixelBuffer)
        
        // Scale saliency map ให้ตรงกับขนาดภาพต้นฉบับ
        let scaleX = image.size.width / ciImage.extent.width
        let scaleY = image.size.height / ciImage.extent.height
        let scaledSaliency = ciImage.transformed(by: CGAffineTransform(scaleX: scaleX, y: scaleY))
        
        // Apply color map (ใช้สี heat map)
        let coloredSaliency = scaledSaliency
            .applyingFilter("CIFalseColor", parameters: [
                "inputColor0": CIColor.blue,
                "inputColor1": CIColor.red
            ])
        
        let context = CIContext()
        guard let cgSaliency = context.createCGImage(coloredSaliency, from: coloredSaliency.extent) else { return nil }
        
        // ผสม saliency กับภาพต้นฉบับ
        let renderer = UIGraphicsImageRenderer(size: image.size)
        return renderer.image { ctx in
            image.draw(in: CGRect(origin: .zero, size: image.size))
            UIImage(cgImage: cgSaliency).draw(
                in: CGRect(origin: .zero, size: image.size),
                blendMode: .normal,
                alpha: alpha
            )
        }
    }
    
    // หา salient regions
    func getSalientRegions(from observation: VNSaliencyImageObservation) -> [CGRect] {
        return observation.salientObjects?.map { $0.boundingBox } ?? []
    }
    
    // Smart crop - ตัดรูปโดยเน้นส่วนที่สำคัญ
    func smartCrop(
        image: UIImage,
        to targetSize: CGSize,
        completion: @escaping (UIImage?) -> Void
    ) {
        generateAttentionSaliency(for: image) { [weak self] observation in
            guard let self = self,
                  let observation = observation,
                  let salientRegions = observation.salientObjects,
                  !salientRegions.isEmpty else {
                // Fallback: center crop
                let cropped = self?.centerCrop(image: image, to: targetSize)
                completion(cropped)
                return
            }
            
            // หา region ที่สำคัญที่สุด
            let mainRegion = salientRegions.max { $0.confidence < $1.confidence }
            guard let region = mainRegion else {
                completion(self.centerCrop(image: image, to: targetSize))
                return
            }
            
            // Crop รอบ salient region
            let cropRect = self.calculateCropRect(
                salientBox: region.boundingBox,
                imageSize: image.size,
                targetAspect: targetSize.width / targetSize.height
            )
            
            let cropped = self.cropImage(image, to: cropRect, targetSize: targetSize)
            completion(cropped)
        }
    }
    
    private func calculateCropRect(
        salientBox: CGRect,
        imageSize: CGSize,
        targetAspect: CGFloat
    ) -> CGRect {
        // แปลง normalized coordinates
        let salientRect = CGRect(
            x: salientBox.minX * imageSize.width,
            y: (1 - salientBox.maxY) * imageSize.height,
            width: salientBox.width * imageSize.width,
            height: salientBox.height * imageSize.height
        )
        
        // คำนวณขนาด crop ที่เหมาะสม
        let salientCenter = CGPoint(x: salientRect.midX, y: salientRect.midY)
        
        var cropWidth: CGFloat
        var cropHeight: CGFloat
        
        if imageSize.width / imageSize.height > targetAspect {
            cropHeight = imageSize.height
            cropWidth = cropHeight * targetAspect
        } else {
            cropWidth = imageSize.width
            cropHeight = cropWidth / targetAspect
        }
        
        // Center crop on salient area
        var cropX = salientCenter.x - cropWidth / 2
        var cropY = salientCenter.y - cropHeight / 2
        
        // Clamp to image bounds
        cropX = max(0, min(cropX, imageSize.width - cropWidth))
        cropY = max(0, min(cropY, imageSize.height - cropHeight))
        
        return CGRect(x: cropX, y: cropY, width: cropWidth, height: cropHeight)
    }
    
    private func cropImage(_ image: UIImage, to rect: CGRect, targetSize: CGSize) -> UIImage? {
        guard let cgImage = image.cgImage?.cropping(to: rect) else { return nil }
        let cropped = UIImage(cgImage: cgImage)
        
        UIGraphicsBeginImageContextWithOptions(targetSize, false, image.scale)
        cropped.draw(in: CGRect(origin: .zero, size: targetSize))
        let result = UIGraphicsGetImageFromCurrentImageContext()
        UIGraphicsEndImageContext()
        return result
    }
    
    private func centerCrop(image: UIImage, to targetSize: CGSize) -> UIImage? {
        let targetAspect = targetSize.width / targetSize.height
        let sourceAspect = image.size.width / image.size.height
        
        var cropRect: CGRect
        if sourceAspect > targetAspect {
            let cropWidth = image.size.height * targetAspect
            cropRect = CGRect(
                x: (image.size.width - cropWidth) / 2,
                y: 0,
                width: cropWidth,
                height: image.size.height
            )
        } else {
            let cropHeight = image.size.width / targetAspect
            cropRect = CGRect(
                x: 0,
                y: (image.size.height - cropHeight) / 2,
                width: image.size.width,
                height: cropHeight
            )
        }
        
        return cropImage(image, to: cropRect, targetSize: targetSize)
    }
}
```

---

## 50.17 Animal Detection

```swift
import Vision
import UIKit

class AnimalDetector {
    
    // ตรวจจับสัตว์ในรูปภาพ
    func detectAnimals(
        in image: UIImage,
        completion: @escaping ([VNRecognizedObjectObservation]) -> Void
    ) {
        guard let cgImage = image.cgImage else {
            completion([])
            return
        }
        
        let request = VNRecognizeAnimalsRequest { request, error in
            let animals = request.results as? [VNRecognizedObjectObservation] ?? []
            DispatchQueue.main.async { completion(animals) }
        }
        
        let handler = VNImageRequestHandler(cgImage: cgImage)
        DispatchQueue.global(qos: .userInitiated).async {
            try? handler.perform([request])
        }
    }
    
    // วาดกรอบรอบสัตว์
    func drawAnimalBoxes(on image: UIImage, animals: [VNRecognizedObjectObservation]) -> UIImage {
        let renderer = UIGraphicsImageRenderer(size: image.size)
        
        return renderer.image { ctx in
            image.draw(in: CGRect(origin: .zero, size: image.size))
            
            let context = ctx.cgContext
            context.setStrokeColor(UIColor.systemOrange.cgColor)
            context.setLineWidth(3)
            
            for animal in animals {
                let box = CGRect(
                    x: animal.boundingBox.minX * image.size.width,
                    y: (1 - animal.boundingBox.maxY) * image.size.height,
                    width: animal.boundingBox.width * image.size.width,
                    height: animal.boundingBox.height * image.size.height
                )
                
                context.stroke(box)
                
                if let topLabel = animal.labels.first {
                    let text = "\(topLabel.identifier) \(Int(topLabel.confidence * 100))%"
                    let attrs: [NSAttributedString.Key: Any] = [
                        .foregroundColor: UIColor.white,
                        .backgroundColor: UIColor.systemOrange.withAlphaComponent(0.8),
                        .font: UIFont.boldSystemFont(ofSize: 14)
                    ]
                    text.draw(at: CGPoint(x: box.minX + 4, y: box.minY + 4), withAttributes: attrs)
                }
            }
        }
    }
}
```

---

## 50.18 Horizon Detection

```swift
import Vision
import UIKit

class HorizonDetector {
    
    // ตรวจจับแนวขอบฟ้าในรูปภาพ
    func detectHorizon(
        in image: UIImage,
        completion: @escaping (VNHorizonObservation?) -> Void
    ) {
        guard let cgImage = image.cgImage else {
            completion(nil)
            return
        }
        
        let request = VNDetectHorizonRequest { request, error in
            let horizon = request.results?.first as? VNHorizonObservation
            DispatchQueue.main.async { completion(horizon) }
        }
        
        let handler = VNImageRequestHandler(cgImage: cgImage)
        DispatchQueue.global(qos: .userInitiated).async {
            try? handler.perform([request])
        }
    }
    
    // แก้ไขความเอียงของรูปภาพ
    func straightenImage(
        _ image: UIImage,
        completion: @escaping (UIImage?) -> Void
    ) {
        detectHorizon(in: image) { horizon in
            guard let horizon = horizon else {
                completion(image)
                return
            }
            
            let angle = horizon.angle
            let straightened = self.rotateImage(image, by: -angle)
            completion(straightened)
        }
    }
    
    private func rotateImage(_ image: UIImage, by angle: CGFloat) -> UIImage {
        let renderer = UIGraphicsImageRenderer(size: image.size)
        return renderer.image { context in
            let transform = CGAffineTransform(
                translationX: image.size.width / 2,
                y: image.size.height / 2
            ).rotated(by: angle).translatedBy(
                x: -image.size.width / 2,
                y: -image.size.height / 2
            )
            context.cgContext.concatenate(transform)
            image.draw(in: CGRect(origin: .zero, size: image.size))
        }
    }
    
    // วาดเส้น horizon
    func drawHorizon(on image: UIImage, horizon: VNHorizonObservation) -> UIImage {
        let renderer = UIGraphicsImageRenderer(size: image.size)
        
        return renderer.image { ctx in
            image.draw(in: CGRect(origin: .zero, size: image.size))
            
            let context = ctx.cgContext
            context.setStrokeColor(UIColor.systemRed.cgColor)
            context.setLineWidth(3)
            
            let transform = horizon.transform(for: image.size)
            context.concatenate(transform)
            
            // วาดเส้น horizon
            context.move(to: CGPoint(x: 0, y: image.size.height / 2))
            context.addLine(to: CGPoint(x: image.size.width, y: image.size.height / 2))
            context.strokePath()
        }
    }
}
```

---

## 50.19 Camera Integration กับ Vision

```swift
import UIKit
import AVFoundation
import Vision

// MARK: - Vision Camera Manager (Reusable)
class VisionCameraManager: NSObject {
    
    // MARK: - Configuration
    struct Configuration {
        var sessionPreset: AVCaptureSession.Preset = .hd1280x720
        var cameraPosition: AVCaptureDevice.Position = .back
        var frameRate: Int = 30
        var enableAudio: Bool = false
    }
    
    // MARK: - Properties
    private let session = AVCaptureSession()
    private let output = AVCaptureVideoDataOutput()
    private let processingQueue = DispatchQueue(label: "com.app.vision.camera", qos: .userInitiated)
    
    var previewLayer: AVCaptureVideoPreviewLayer?
    var onFrame: ((CVPixelBuffer) -> Void)?
    
    private(set) var isRunning = false
    
    // MARK: - Setup
    func setup(configuration: Configuration = Configuration()) throws {
        session.beginConfiguration()
        
        // Session preset
        session.sessionPreset = configuration.sessionPreset
        
        // Camera input
        guard let camera = AVCaptureDevice.default(
            .builtInWideAngleCamera,
            for: .video,
            position: configuration.cameraPosition
        ) else {
            throw CameraError.deviceNotFound
        }
        
        // กำหนด frame rate
        try camera.lockForConfiguration()
        camera.activeVideoMinFrameDuration = CMTime(value: 1, timescale: CMTimeScale(configuration.frameRate))
        camera.activeVideoMaxFrameDuration = CMTime(value: 1, timescale: CMTimeScale(configuration.frameRate))
        camera.unlockForConfiguration()
        
        guard let input = try? AVCaptureDeviceInput(device: camera) else {
            throw CameraError.inputCreationFailed
        }
        
        if session.canAddInput(input) {
            session.addInput(input)
        }
        
        // Video output
        output.setSampleBufferDelegate(self, queue: processingQueue)
        output.alwaysDiscardsLateVideoFrames = true
        output.videoSettings = [
            kCVPixelBufferPixelFormatTypeKey as String: kCVPixelFormatType_32BGRA
        ]
        
        if session.canAddOutput(output) {
            session.addOutput(output)
        }
        
        session.commitConfiguration()
        
        // Preview layer
        previewLayer = AVCaptureVideoPreviewLayer(session: session)
        previewLayer?.videoGravity = .resizeAspectFill
    }
    
    func start() {
        guard !isRunning else { return }
        processingQueue.async { [weak self] in
            self?.session.startRunning()
            self?.isRunning = true
        }
    }
    
    func stop() {
        guard isRunning else { return }
        session.stopRunning()
        isRunning = false
    }
    
    enum CameraError: Error {
        case deviceNotFound
        case inputCreationFailed
        case permissionDenied
    }
}

extension VisionCameraManager: AVCaptureVideoDataOutputSampleBufferDelegate {
    func captureOutput(_ output: AVCaptureOutput, didOutput sampleBuffer: CMSampleBuffer, from connection: AVCaptureConnection) {
        guard let pixelBuffer = CMSampleBufferGetImageBuffer(sampleBuffer) else { return }
        onFrame?(pixelBuffer)
    }
}

// MARK: - Vision Pipeline สำหรับ Camera
class CameraVisionPipeline {
    
    private let cameraManager = VisionCameraManager()
    private var requests: [VNRequest] = []
    private var lastProcessedTime = Date()
    private let processingInterval: TimeInterval
    
    init(processingInterval: TimeInterval = 0.1) {
        self.processingInterval = processingInterval
    }
    
    func setup(in view: UIView, configuration: VisionCameraManager.Configuration = .init()) throws {
        try cameraManager.setup(configuration: configuration)
        
        if let previewLayer = cameraManager.previewLayer {
            previewLayer.frame = view.bounds
            view.layer.insertSublayer(previewLayer, at: 0)
        }
        
        cameraManager.onFrame = { [weak self] pixelBuffer in
            self?.processFrame(pixelBuffer)
        }
    }
    
    func addRequest(_ request: VNRequest) {
        requests.append(request)
    }
    
    func clearRequests() {
        requests.removeAll()
    }
    
    func start() { cameraManager.start() }
    func stop() { cameraManager.stop() }
    
    private func processFrame(_ pixelBuffer: CVPixelBuffer) {
        let now = Date()
        guard now.timeIntervalSince(lastProcessedTime) >= processingInterval,
              !requests.isEmpty else { return }
        lastProcessedTime = now
        
        let handler = VNImageRequestHandler(cvPixelBuffer: pixelBuffer, orientation: .up)
        try? handler.perform(requests)
    }
}
```

---

## 50.20 Practical Exercises

### แบบฝึกหัดที่ 1: Multi-Face Analyzer

```swift
import SwiftUI
import Vision
import UIKit

// Exercise 1: วิเคราะห์ใบหน้าหลายๆ คนพร้อมกัน
struct MultiFaceAnalyzerView: View {
    @State private var selectedImage: UIImage?
    @State private var annotatedImage: UIImage?
    @State private var faceCount = 0
    @State private var showPicker = false
    @State private var isAnalyzing = false
    @State private var faceDetails: [FaceDetail] = []
    
    struct FaceDetail: Identifiable {
        let id = UUID()
        let index: Int
        let boundingBox: CGRect
        let roll: Double
        let yaw: Double
        let quality: Float
        var hasLandmarks: Bool
    }
    
    var body: some View {
        NavigationView {
            ScrollView {
                VStack(spacing: 16) {
                    // Image
                    Group {
                        if let img = annotatedImage ?? selectedImage {
                            Image(uiImage: img)
                                .resizable()
                                .scaledToFit()
                                .cornerRadius(12)
                        } else {
                            RoundedRectangle(cornerRadius: 12)
                                .fill(Color.gray.opacity(0.2))
                                .frame(height: 250)
                                .overlay(
                                    Label("เลือกรูปภาพ", systemImage: "person.3")
                                        .foregroundColor(.secondary)
                                )
                        }
                    }
                    .frame(maxHeight: 300)
                    
                    if isAnalyzing {
                        ProgressView("กำลังวิเคราะห์ใบหน้า...")
                    }
                    
                    if !faceDetails.isEmpty {
                        VStack(alignment: .leading, spacing: 12) {
                            Text("พบ \(faceDetails.count) ใบหน้า")
                                .font(.headline)
                            
                            ForEach(faceDetails) { face in
                                HStack {
                                    VStack(alignment: .leading) {
                                        Text("ใบหน้าที่ \(face.index + 1)")
                                            .font(.subheadline)
                                            .bold()
                                        Text("คุณภาพ: \(Int(face.quality * 100))%")
                                            .font(.caption)
                                            .foregroundColor(.secondary)
                                    }
                                    Spacer()
                                    VStack(alignment: .trailing) {
                                        Text("R: \(String(format: "%.1f°", face.roll * 180 / .pi))")
                                        Text("Y: \(String(format: "%.1f°", face.yaw * 180 / .pi))")
                                    }
                                    .font(.caption)
                                    .foregroundColor(.secondary)
                                }
                                .padding(8)
                                .background(Color(.systemGray6))
                                .cornerRadius(8)
                            }
                        }
                        .padding()
                        .background(Color(.systemBackground))
                        .cornerRadius(12)
                        .shadow(radius: 2)
                    }
                    
                    Button("เลือกรูปภาพ") { showPicker = true }
                        .buttonStyle(.borderedProminent)
                        .controlSize(.large)
                }
                .padding()
            }
            .navigationTitle("Multi-Face Analyzer")
            .sheet(isPresented: $showPicker) {
                ImagePicker(image: $selectedImage, onSelect: analyzeImage)
            }
        }
    }
    
    func analyzeImage() {
        guard let image = selectedImage else { return }
        isAnalyzing = true
        faceDetails = []
        
        guard let cgImage = image.cgImage else { return }
        
        // ใช้ทั้ง landmarks และ quality requests
        let landmarkRequest = VNDetectFaceLandmarksRequest { request, _ in
            guard let faces = request.results as? [VNFaceObservation] else { return }
            
            // วาด annotations
            let renderer = UIGraphicsImageRenderer(size: image.size)
            let annotated = renderer.image { ctx in
                image.draw(in: CGRect(origin: .zero, size: image.size))
                let context = ctx.cgContext
                
                for (i, face) in faces.enumerated() {
                    let box = CGRect(
                        x: face.boundingBox.minX * image.size.width,
                        y: (1 - face.boundingBox.maxY) * image.size.height,
                        width: face.boundingBox.width * image.size.width,
                        height: face.boundingBox.height * image.size.height
                    )
                    
                    // วาดกรอบสีตามลำดับ
                    let colors: [UIColor] = [.systemGreen, .systemBlue, .systemOrange, .systemRed, .systemPurple]
                    context.setStrokeColor(colors[i % colors.count].cgColor)
                    context.setLineWidth(3)
                    context.stroke(box)
                    
                    // เลขใบหน้า
                    let label = "\(i + 1)"
                    let attrs: [NSAttributedString.Key: Any] = [
                        .foregroundColor: UIColor.white,
                        .backgroundColor: colors[i % colors.count].withAlphaComponent(0.8),
                        .font: UIFont.boldSystemFont(ofSize: 16)
                    ]
                    label.draw(at: CGPoint(x: box.minX + 4, y: box.minY + 4), withAttributes: attrs)
                }
            }
            
            DispatchQueue.main.async {
                self.isAnalyzing = false
                self.annotatedImage = annotated
                self.faceCount = faces.count
                self.faceDetails = faces.enumerated().map { i, face in
                    FaceDetail(
                        index: i,
                        boundingBox: face.boundingBox,
                        roll: face.roll?.doubleValue ?? 0,
                        yaw: face.yaw?.doubleValue ?? 0,
                        quality: face.faceCaptureQuality ?? 0,
                        hasLandmarks: face.landmarks != nil
                    )
                }
            }
        }
        
        let handler = VNImageRequestHandler(cgImage: cgImage)
        DispatchQueue.global(qos: .userInitiated).async {
            try? handler.perform([landmarkRequest])
        }
    }
}
```

---

## 50.21 Building a Document Scanner App (Complete)

```swift
import SwiftUI
import Vision
import VisionKit

// MARK: - Document Scanner App

@main
struct DocumentScannerApp: App {
    var body: some Scene {
        WindowGroup {
            DocumentScannerMainView()
        }
    }
}

// MARK: - Main View
struct DocumentScannerMainView: View {
    @StateObject private var viewModel = DocumentScannerViewModel()
    
    var body: some View {
        NavigationView {
            Group {
                if viewModel.scannedPages.isEmpty {
                    emptyStateView
                } else {
                    scannedPagesView
                }
            }
            .navigationTitle("Document Scanner")
            .toolbar {
                ToolbarItemGroup(placement: .navigationBarTrailing) {
                    if !viewModel.scannedPages.isEmpty {
                        Button("แชร์") { viewModel.sharePDF() }
                        Button("ล้าง") { viewModel.clearAll() }
                    }
                    Button {
                        viewModel.startScanning()
                    } label: {
                        Image(systemName: "doc.viewfinder")
                    }
                }
            }
        }
        .sheet(isPresented: $viewModel.showingScanner) {
            DocumentCameraView(onScan: viewModel.processScan)
        }
        .alert("ข้อผิดพลาด", isPresented: $viewModel.showError) {
            Button("ตกลง") {}
        } message: {
            Text(viewModel.errorMessage)
        }
    }
    
    var emptyStateView: some View {
        VStack(spacing: 24) {
            Image(systemName: "doc.text.viewfinder")
                .font(.system(size: 80))
                .foregroundColor(.blue)
            
            Text("ไม่มีเอกสาร")
                .font(.title2)
                .bold()
            
            Text("กดปุ่มเพื่อเริ่มสแกนเอกสาร")
                .foregroundColor(.secondary)
                .multilineTextAlignment(.center)
            
            Button("เริ่มสแกน") {
                viewModel.startScanning()
            }
            .buttonStyle(.borderedProminent)
            .controlSize(.large)
        }
        .padding()
    }
    
    var scannedPagesView: some View {
        List {
            ForEach(Array(viewModel.scannedPages.enumerated()), id: \.offset) { index, page in
                NavigationLink {
                    PageDetailView(page: page, pageNumber: index + 1)
                } label: {
                    HStack(spacing: 12) {
                        Image(uiImage: page.image)
                            .resizable()
                            .scaledToFit()
                            .frame(width: 60, height: 80)
                            .cornerRadius(6)
                            .shadow(radius: 2)
                        
                        VStack(alignment: .leading, spacing: 4) {
                            Text("หน้า \(index + 1)")
                                .font(.headline)
                            
                            if !page.recognizedText.isEmpty {
                                Text(page.recognizedText)
                                    .font(.caption)
                                    .foregroundColor(.secondary)
                                    .lineLimit(2)
                            } else if page.isProcessing {
                                HStack {
                                    ProgressView().scaleEffect(0.7)
                                    Text("กำลังอ่านข้อความ...")
                                        .font(.caption)
                                        .foregroundColor(.secondary)
                                }
                            }
                        }
                        
                        Spacer()
                    }
                    .padding(.vertical, 4)
                }
            }
            .onDelete(perform: viewModel.deletePage)
        }
    }
}

// MARK: - Page Detail View
struct PageDetailView: View {
    let page: ScannedPage
    let pageNumber: Int
    @State private var showText = false
    
    var body: some View {
        ScrollView {
            VStack(spacing: 16) {
                Image(uiImage: page.image)
                    .resizable()
                    .scaledToFit()
                    .cornerRadius(12)
                    .shadow(radius: 4)
                
                if !page.recognizedText.isEmpty {
                    VStack(alignment: .leading, spacing: 8) {
                        HStack {
                            Text("ข้อความที่รู้จำได้")
                                .font(.headline)
                            Spacer()
                            Button {
                                UIPasteboard.general.string = page.recognizedText
                            } label: {
                                Label("คัดลอก", systemImage: "doc.on.doc")
                                    .font(.caption)
                            }
                        }
                        
                        Text(page.recognizedText)
                            .font(.body)
                            .padding()
                            .frame(maxWidth: .infinity, alignment: .leading)
                            .background(Color(.systemGray6))
                            .cornerRadius(8)
                    }
                    .padding()
                    .background(Color(.systemBackground))
                    .cornerRadius(12)
                    .shadow(radius: 2)
                }
            }
            .padding()
        }
        .navigationTitle("หน้า \(pageNumber)")
    }
}

// MARK: - ScannedPage Model
class ScannedPage: ObservableObject, Identifiable {
    let id = UUID()
    let image: UIImage
    @Published var recognizedText: String = ""
    @Published var isProcessing: Bool = true
    
    init(image: UIImage) {
        self.image = image
    }
}

// MARK: - ViewModel
@MainActor
class DocumentScannerViewModel: ObservableObject {
    @Published var scannedPages: [ScannedPage] = []
    @Published var showingScanner = false
    @Published var showError = false
    @Published var errorMessage = ""
    
    func startScanning() {
        guard VNDocumentCameraViewController.isSupported else {
            errorMessage = "อุปกรณ์นี้ไม่รองรับการสแกนเอกสาร"
            showError = true
            return
        }
        showingScanner = true
    }
    
    func processScan(_ scan: VNDocumentCameraScan) {
        var newPages: [ScannedPage] = []
        
        for i in 0..<scan.pageCount {
            let image = scan.imageOfPage(at: i)
            let page = ScannedPage(image: image)
            newPages.append(page)
        }
        
        scannedPages.append(contentsOf: newPages)
        
        // เริ่ม OCR สำหรับทุกหน้า
        for page in newPages {
            recognizeText(in: page)
        }
    }
    
    private func recognizeText(in page: ScannedPage) {
        guard let cgImage = page.image.cgImage else {
            page.isProcessing = false
            return
        }
        
        let request = VNRecognizeTextRequest { [weak page] request, _ in
            let texts = (request.results as? [VNRecognizedTextObservation] ?? [])
                .compactMap { $0.topCandidates(1).first?.string }
            
            Task { @MainActor in
                page?.recognizedText = texts.joined(separator: "\n")
                page?.isProcessing = false
            }
        }
        
        request.recognitionLevel = .accurate
        request.recognitionLanguages = ["th", "en"]
        request.usesLanguageCorrection = true
        
        let handler = VNImageRequestHandler(cgImage: cgImage)
        Task.detached(priority: .utility) {
            try? handler.perform([request])
        }
    }
    
    func deletePage(at offsets: IndexSet) {
        scannedPages.remove(atOffsets: offsets)
    }
    
    func clearAll() {
        scannedPages.removeAll()
    }
    
    func sharePDF() {
        let images = scannedPages.map { $0.image }
        guard !images.isEmpty else { return }
        
        // สร้าง PDF
        let pdfData = createPDF(from: images)
        
        let activityVC = UIActivityViewController(
            activityItems: [pdfData],
            applicationActivities: nil
        )
        
        if let windowScene = UIApplication.shared.connectedScenes.first as? UIWindowScene,
           let window = windowScene.windows.first,
           let rootVC = window.rootViewController {
            rootVC.present(activityVC, animated: true)
        }
    }
    
    private func createPDF(from images: [UIImage]) -> Data {
        let pdfData = NSMutableData()
        UIGraphicsBeginPDFContextToData(pdfData, .zero, nil)
        
        for image in images {
            let pageSize = CGSize(
                width: image.size.width,
                height: image.size.height
            )
            UIGraphicsBeginPDFPageWithInfo(CGRect(origin: .zero, size: pageSize), nil)
            image.draw(in: CGRect(origin: .zero, size: pageSize))
        }
        
        UIGraphicsEndPDFContext()
        return pdfData as Data
    }
}

// MARK: - Document Camera View
struct DocumentCameraView: UIViewControllerRepresentable {
    let onScan: (VNDocumentCameraScan) -> Void
    @Environment(\.dismiss) var dismiss
    
    func makeUIViewController(context: Context) -> VNDocumentCameraViewController {
        let vc = VNDocumentCameraViewController()
        vc.delegate = context.coordinator
        return vc
    }
    
    func updateUIViewController(_ uiViewController: VNDocumentCameraViewController, context: Context) {}
    
    func makeCoordinator() -> Coordinator { Coordinator(self) }
    
    class Coordinator: NSObject, VNDocumentCameraViewControllerDelegate {
        let parent: DocumentCameraView
        init(_ parent: DocumentCameraView) { self.parent = parent }
        
        func documentCameraViewController(_ controller: VNDocumentCameraViewController, didFinishWith scan: VNDocumentCameraScan) {
            parent.onScan(scan)
            parent.dismiss()
        }
        
        func documentCameraViewControllerDidCancel(_ controller: VNDocumentCameraViewController) {
            parent.dismiss()
        }
        
        func documentCameraViewController(_ controller: VNDocumentCameraViewController, didFailWithError error: Error) {
            parent.dismiss()
        }
    }
}
```

---

## 50.22 สรุป

ในบทนี้เราได้เรียนรู้ **Vision Framework** อย่างครอบคลุม:

### สิ่งที่ได้เรียนรู้

| หัวข้อ | ประเด็นสำคัญ |
|--------|-------------|
| **Vision Overview** | Framework สำหรับ computer vision บน iOS |
| **VNRequest Types** | Face, Body, Text, Barcode, Object detection |
| **VNImageRequestHandler** | ประมวลผลรูปภาพเดี่ยว |
| **VNSequenceRequestHandler** | ประมวลผลลำดับภาพ (วิดีโอ) |
| **Face Detection** | VNDetectFaceRectanglesRequest |
| **Face Landmarks** | จุดสำคัญบนใบหน้า 76 จุด |
| **Body Pose** | VNDetectHumanBodyPoseRequest - 19 joints |
| **Hand Pose** | VNDetectHumanHandPoseRequest - 21 joints |
| **Object Tracking** | VNTrackObjectRequest ระหว่าง frames |
| **Text Recognition** | VNRecognizeTextRequest (OCR) |
| **Barcode/QR** | VNDetectBarcodesRequest |
| **Document Scanner** | VNDocumentCameraViewController |
| **Saliency** | ส่วนที่ดึงดูดความสนใจในรูปภาพ |

### Best Practices

```swift
// 1. รัน Vision requests บน background thread
DispatchQueue.global(qos: .userInitiated).async {
    try? handler.perform([request])
}

// 2. อัพเดต UI บน main thread
DispatchQueue.main.async {
    // UI updates here
}

// 3. ใช้ VNSequenceRequestHandler สำหรับวิดีโอ
let sequenceHandler = VNSequenceRequestHandler() // สร้างครั้งเดียว

// 4. Throttle frame processing
guard Date().timeIntervalSince(lastTime) >= 0.1 else { return }

// 5. ตรวจสอบ confidence ก่อนใช้ผลลัพธ์
guard observation.confidence > 0.5 else { continue }

// 6. ระวัง coordinate system conversion
// Vision: origin bottom-left, y เพิ่มขึ้น
// UIKit: origin top-left, y เพิ่มลง
let uiKitY = (1 - visionY) * imageHeight
```

### Coordinate Conversion อย่างสมบูรณ์

```swift
extension CGRect {
    // แปลง Vision normalized rect เป็น UIKit rect
    func visionToUIKit(imageSize: CGSize) -> CGRect {
        return CGRect(
            x: self.minX * imageSize.width,
            y: (1 - self.maxY) * imageSize.height,
            width: self.width * imageSize.width,
            height: self.height * imageSize.height
        )
    }
}
```

### ขั้นตอนต่อไป

1. ทดลองผสาน **Vision กับ Core ML** (บทที่ 49)
2. สร้าง **AR experience** ด้วย RealityKit + Vision
3. ลองใช้ **VisionKit** สำหรับ Document Scanner
4. ศึกษา **ARKit** สำหรับ Augmented Reality ในบทถัดไป

---

*ศึกษาเพิ่มเติม:*
- [Vision Framework Documentation](https://developer.apple.com/documentation/vision)
- [WWDC: Detect Body and Hand Pose with Vision](https://developer.apple.com/videos/)
- [VisionKit Documentation](https://developer.apple.com/documentation/visionkit)
