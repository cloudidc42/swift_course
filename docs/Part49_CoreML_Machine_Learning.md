# Part 49: Core ML และ Machine Learning ใน iOS

## บทนำ

Machine Learning (ML) ได้กลายเป็นส่วนสำคัญของการพัฒนาแอปพลิเคชัน iOS สมัยใหม่ Apple ได้พัฒนา **Core ML** framework ที่ช่วยให้นักพัฒนาสามารถนำโมเดล Machine Learning มาใช้งานในแอปได้อย่างง่ายดาย โดยไม่จำเป็นต้องมีความรู้เชิงลึกด้าน ML

ในบทนี้เราจะเรียนรู้:
- Core ML คืออะไรและทำงานอย่างไร
- Create ML vs Core ML
- ประเภทของโมเดลต่างๆ
- การนำ `.mlmodel` ไปใช้งาน
- Vision + Core ML pipeline
- การสร้างแอป Image Classifier
- Real-time camera inference
- Performance optimization

---

## 49.1 Core ML คืออะไร

**Core ML** เป็น framework ของ Apple ที่ออกแบบมาเพื่อรัน Machine Learning models บนอุปกรณ์ iOS, macOS, watchOS และ tvOS โดยตรง (on-device inference)

### คุณสมบัติหลักของ Core ML

```
┌─────────────────────────────────────────────────┐
│                   Your App                      │
├─────────────────────────────────────────────────┤
│              Core ML Framework                  │
├──────────────┬──────────────┬───────────────────┤
│   CPU        │    GPU       │    Neural Engine  │
│ (Efficiency) │ (Parallel)   │  (A-Series Chip)  │
└──────────────┴──────────────┴───────────────────┘
```

### ประโยชน์ของ Core ML

1. **On-device Processing**: ประมวลผลบนอุปกรณ์ ไม่ต้องส่งข้อมูลไปยัง server
2. **ความเป็นส่วนตัว**: ข้อมูลผู้ใช้ไม่ถูกส่งออกไปภายนอก
3. **ความเร็ว**: ไม่มี network latency
4. **Offline Support**: ทำงานได้แม้ไม่มีอินเทอร์เน็ต
5. **ประสิทธิภาพพลังงาน**: ใช้ Neural Engine ซึ่งประหยัดพลังงาน

### Core ML Stack

```swift
// Core ML ทำงานร่วมกับ frameworks อื่นๆ
import CoreML        // โมเดล ML หลัก
import Vision        // การประมวลผลภาพ
import NaturalLanguage // การประมวลผลภาษา
import CreateML      // การสร้างโมเดล (macOS)
```

---

## 49.2 Create ML vs Core ML

### Create ML

**Create ML** เป็นเครื่องมือสำหรับ **สร้าง** และ **เทรน** โมเดล ML บน Mac

```
Create ML ใช้สำหรับ:
- การเทรนโมเดลจาก dataset ของคุณเอง
- การ fine-tune โมเดลที่มีอยู่
- การ evaluate ประสิทธิภาพโมเดล
- Export เป็น .mlmodel file
```

### Core ML

**Core ML** ใช้สำหรับ **รัน** โมเดลในแอปพลิเคชัน

```
Core ML ใช้สำหรับ:
- การ import .mlmodel ลงในโปรเจกต์
- การรัน inference บนอุปกรณ์
- การเชื่อมต่อกับ Vision, NLP frameworks
- การ optimize โมเดล
```

### เปรียบเทียบ Create ML กับ Core ML

| คุณสมบัติ | Create ML | Core ML |
|-----------|-----------|---------|
| หน้าที่หลัก | สร้าง/เทรนโมเดล | รันโมเดล |
| Platform | macOS เท่านั้น | iOS, macOS, watchOS |
| ใช้ใน | การพัฒนา | Production app |
| Output | .mlmodel file | Inference results |

### Create ML App บน Mac

```swift
// Create ML - ตัวอย่างการสร้าง Image Classifier
import CreateML

// โหลด training data
let trainingData = try MLImageClassifier.DataSource.labeledFiles(
    at: URL(fileURLWithPath: "/path/to/training/images")
)

// ตั้งค่า parameters
let parameters = MLImageClassifier.ModelParameters(
    validation: .split(strategy: .automatic),
    maxIterations: 25,
    augmentation: [.flip, .rotate, .blur]
)

// เทรนโมเดล
let classifier = try MLImageClassifier(
    trainingData: trainingData,
    parameters: parameters
)

// บันทึกโมเดล
let modelURL = URL(fileURLWithPath: "/path/to/save/MyClassifier.mlmodel")
try classifier.write(to: modelURL)
```

---

## 49.3 ประเภทของโมเดล Core ML

### 49.3.1 Image Classification (การจำแนกรูปภาพ)

โมเดลที่รับรูปภาพเป็น input และส่งออก label ที่บ่งบอกว่ารูปภาพนั้นคืออะไร

```swift
// ตัวอย่าง: MobileNetV2 สำหรับ Image Classification
// Input: รูปภาพ 224x224 pixels
// Output: class label + confidence score

import CoreML
import Vision

class ImageClassifier {
    private var model: VNCoreMLModel?
    
    init() {
        // โหลดโมเดล
        guard let mlModel = try? MobileNetV2(configuration: MLModelConfiguration()).model,
              let visionModel = try? VNCoreMLModel(for: mlModel) else {
            print("ไม่สามารถโหลดโมเดลได้")
            return
        }
        self.model = visionModel
    }
    
    func classify(image: UIImage, completion: @escaping (String, Float) -> Void) {
        guard let model = model,
              let ciImage = CIImage(image: image) else { return }
        
        let request = VNCoreMLRequest(model: model) { request, error in
            guard let results = request.results as? [VNClassificationObservation],
                  let topResult = results.first else { return }
            
            DispatchQueue.main.async {
                completion(topResult.identifier, topResult.confidence)
            }
        }
        
        let handler = VNImageRequestHandler(ciImage: ciImage)
        try? handler.perform([request])
    }
}
```

### 49.3.2 Object Detection (การตรวจจับวัตถุ)

โมเดลที่สามารถระบุตำแหน่งและชนิดของวัตถุหลายชิ้นในรูปภาพเดียว

```swift
// Object Detection ด้วย YOLOv3
import CoreML
import Vision

class ObjectDetector {
    private var model: VNCoreMLModel?
    
    init() {
        guard let mlModel = try? YOLOv3(configuration: MLModelConfiguration()).model,
              let visionModel = try? VNCoreMLModel(for: mlModel) else { return }
        self.model = visionModel
    }
    
    func detect(in image: UIImage, completion: @escaping ([DetectedObject]) -> Void) {
        guard let model = model,
              let ciImage = CIImage(image: image) else { return }
        
        let request = VNCoreMLRequest(model: model) { request, error in
            guard let results = request.results as? [VNRecognizedObjectObservation] else { return }
            
            let objects = results.map { observation -> DetectedObject in
                let label = observation.labels.first?.identifier ?? "unknown"
                let confidence = observation.labels.first?.confidence ?? 0
                let boundingBox = observation.boundingBox
                return DetectedObject(label: label, confidence: confidence, boundingBox: boundingBox)
            }
            
            DispatchQueue.main.async {
                completion(objects)
            }
        }
        
        // กำหนดว่าต้องการ bounding boxes
        request.imageCropAndScaleOption = .scaleFill
        
        let handler = VNImageRequestHandler(ciImage: ciImage)
        try? handler.perform([request])
    }
}

struct DetectedObject {
    let label: String
    let confidence: Float
    let boundingBox: CGRect
}
```

### 49.3.3 Text Classification (การจำแนกข้อความ)

```swift
// Text Classification สำหรับ Sentiment Analysis
import NaturalLanguage
import CoreML

class SentimentAnalyzer {
    private let model: NLModel
    
    init?() {
        // โหลด Custom Text Classifier
        guard let modelURL = Bundle.main.url(forResource: "SentimentClassifier", withExtension: "mlmodelc"),
              let mlModel = try? MLModel(contentsOf: modelURL),
              let nlModel = try? NLModel(mlModel: mlModel) else {
            return nil
        }
        self.model = nlModel
    }
    
    func analyzeSentiment(text: String) -> String {
        return model.predictedLabel(for: text) ?? "neutral"
    }
    
    func analyzeWithConfidence(text: String) -> [(String, Double)] {
        let hypotheses = model.predictedLabelHypotheses(for: text, maximumCount: 3)
        return hypotheses.sorted { $0.value > $1.value }.map { ($0.key, $0.value) }
    }
}

// การใช้งาน
let analyzer = SentimentAnalyzer()
let sentiment = analyzer?.analyzeSentiment(text: "สินค้านี้ดีมากเลย รักเลย!")
print("Sentiment: \(sentiment ?? "unknown")") // Output: positive
```

### 49.3.4 Tabular Regression (การทำนายค่าจากตาราง)

```swift
// Tabular Regression สำหรับการทำนายราคาบ้าน
import CoreML

class HousePricePredictor {
    private let model: HousePriceModel
    
    init?() {
        guard let model = try? HousePriceModel(configuration: MLModelConfiguration()) else {
            return nil
        }
        self.model = model
    }
    
    func predictPrice(bedrooms: Double, bathrooms: Double, squareFeet: Double, location: String) -> Double? {
        let input = HousePriceModelInput(
            bedrooms: bedrooms,
            bathrooms: bathrooms,
            squareFeet: squareFeet,
            location: location
        )
        
        guard let output = try? model.prediction(input: input) else { return nil }
        return output.price
    }
}

// การใช้งาน
let predictor = HousePricePredictor()
if let price = predictor?.predictPrice(
    bedrooms: 3,
    bathrooms: 2,
    squareFeet: 1500,
    location: "Bangkok"
) {
    print("ราคาบ้านที่ทำนาย: \(price) บาท")
}
```

---

## 49.4 การนำ .mlmodel เข้าโปรเจกต์

### ขั้นตอนการนำเข้า .mlmodel

1. **ดาวน์โหลดโมเดล** จาก Apple Developer หรือ Hugging Face
2. **ลากไฟล์** `.mlmodel` เข้าไปใน Xcode project
3. **เลือก Target membership**
4. Xcode จะ **auto-generate Swift class** ให้

### การตั้งค่า MLModelConfiguration

```swift
import CoreML

// การกำหนดค่าสำหรับโมเดล
let config = MLModelConfiguration()

// เลือก compute units
config.computeUnits = .all          // ใช้ทุก compute unit (แนะนำ)
// config.computeUnits = .cpuOnly   // ใช้ CPU เท่านั้น
// config.computeUnits = .cpuAndGPU // ใช้ CPU และ GPU
// config.computeUnits = .cpuAndNeuralEngine // ใช้ CPU และ Neural Engine

// โหลดโมเดลพร้อม configuration
let model = try MobileNetV2(configuration: config)
```

### การโหลดโมเดลแบบ Async (iOS 16+)

```swift
import CoreML

class ModelLoader {
    // โหลดโมเดลแบบ asynchronous (ไม่บล็อก UI thread)
    static func loadModel() async throws -> VNCoreMLModel {
        let config = MLModelConfiguration()
        config.computeUnits = .all
        
        // โหลดแบบ async
        let mlModel = try await MLModel.load(
            contentsOf: Bundle.main.url(forResource: "MobileNetV2", withExtension: "mlmodelc")!,
            configuration: config
        )
        
        return try VNCoreMLModel(for: mlModel)
    }
}

// การใช้งาน
Task {
    do {
        let model = try await ModelLoader.loadModel()
        print("โหลดโมเดลสำเร็จ")
    } catch {
        print("โหลดโมเดลล้มเหลว: \(error)")
    }
}
```

### การ Compile โมเดลล่วงหน้า

```swift
// การ compile .mlmodel เป็น .mlmodelc
let modelURL = URL(fileURLWithPath: "/path/to/MyModel.mlmodel")
let compiledURL = try await MLModel.compileModel(at: modelURL)

// ย้ายไปยัง Documents directory เพื่อใช้งาน
let documentsURL = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first!
let savedURL = documentsURL.appendingPathComponent("MyModel.mlmodelc")
try FileManager.default.copyItem(at: compiledURL, to: savedURL)
```

---

## 49.5 VNCoreMLRequest

**VNCoreMLRequest** เป็น request ที่ใช้รัน Core ML model ผ่าน Vision framework

```swift
import Vision
import CoreML

class CoreMLRequestDemo {
    
    // สร้าง VNCoreMLRequest
    func createRequest(with model: VNCoreMLModel) -> VNCoreMLRequest {
        let request = VNCoreMLRequest(model: model) { [weak self] request, error in
            self?.handleResults(request: request, error: error)
        }
        
        // กำหนด image crop และ scale option
        // .centerCrop - ตัดกลางรูป
        // .scaleFit   - ย่อ/ขยายให้พอดี
        // .scaleFill  - ย่อ/ขยายให้เต็ม (อาจมีการ crop)
        request.imageCropAndScaleOption = .centerCrop
        
        return request
    }
    
    func handleResults(request: VNRequest, error: Error?) {
        if let error = error {
            print("เกิดข้อผิดพลาด: \(error.localizedDescription)")
            return
        }
        
        // ตรวจสอบประเภทของ results
        if let classifications = request.results as? [VNClassificationObservation] {
            // Image Classification
            for result in classifications.prefix(5) {
                print("\(result.identifier): \(result.confidence * 100)%")
            }
        } else if let detections = request.results as? [VNRecognizedObjectObservation] {
            // Object Detection
            for detection in detections {
                print("พบ: \(detection.labels.first?.identifier ?? "unknown")")
                print("ตำแหน่ง: \(detection.boundingBox)")
            }
        }
    }
    
    // การรัน request
    func processImage(_ image: UIImage) {
        guard let model = try? MobileNetV2(configuration: .init()).model,
              let visionModel = try? VNCoreMLModel(for: model) else { return }
        
        let request = createRequest(with: visionModel)
        
        guard let ciImage = CIImage(image: image) else { return }
        let handler = VNImageRequestHandler(ciImage: ciImage, options: [:])
        
        do {
            try handler.perform([request])
        } catch {
            print("การประมวลผลล้มเหลว: \(error)")
        }
    }
}
```

---

## 49.6 VNCoreMLModel

**VNCoreMLModel** เป็น wrapper ที่ใช้ห่อหุ้ม MLModel เพื่อใช้กับ Vision framework

```swift
import Vision
import CoreML

// การสร้าง VNCoreMLModel
func createVisionModel() throws -> VNCoreMLModel {
    // วิธีที่ 1: จาก auto-generated class
    let mlModel1 = try MobileNetV2(configuration: MLModelConfiguration()).model
    let visionModel1 = try VNCoreMLModel(for: mlModel1)
    
    // วิธีที่ 2: จากไฟล์โดยตรง
    guard let modelURL = Bundle.main.url(forResource: "MobileNetV2", withExtension: "mlmodelc") else {
        throw NSError(domain: "ModelError", code: 1, userInfo: [NSLocalizedDescriptionKey: "ไม่พบไฟล์โมเดล"])
    }
    let mlModel2 = try MLModel(contentsOf: modelURL)
    let visionModel2 = try VNCoreMLModel(for: mlModel2)
    
    return visionModel2
}

// ตรวจสอบ model description
func printModelInfo(model: MLModel) {
    let description = model.modelDescription
    
    print("=== Model Information ===")
    print("Input features:")
    for (name, feature) in description.inputDescriptionsByName {
        print("  - \(name): \(feature.type)")
    }
    
    print("Output features:")
    for (name, feature) in description.outputDescriptionsByName {
        print("  - \(name): \(feature.type)")
    }
    
    if let metadata = description.metadata[MLModelMetadataKey.description] {
        print("Description: \(metadata)")
    }
}
```

---

## 49.7 Vision + Core ML Pipeline

Vision + Core ML ทำงานร่วมกันเพื่อประมวลผลภาพ โดย Vision จัดการ preprocessing ภาพ และส่งต่อให้ Core ML

```
Image/Camera Feed
      │
      ▼
VNImageRequestHandler  ← จัดการการ load image
      │
      ▼
VNCoreMLRequest       ← ส่ง image ไปยัง Core ML model
      │
      ▼
Core ML Model         ← ประมวลผล inference
      │
      ▼
VNObservation Results ← ผลลัพธ์ (Classification/Detection/etc.)
      │
      ▼
UI Update            ← แสดงผลลัพธ์
```

### Pipeline Implementation

```swift
import UIKit
import CoreML
import Vision

class VisionCoreMLPipeline {
    
    // MARK: - Properties
    private var classificationModel: VNCoreMLModel?
    private var detectionModel: VNCoreMLModel?
    private let processingQueue = DispatchQueue(label: "com.app.mlprocessing", qos: .userInitiated)
    
    // MARK: - Initialization
    init() {
        setupModels()
    }
    
    private func setupModels() {
        do {
            // Image Classifier
            let classifierMLModel = try MobileNetV2(configuration: MLModelConfiguration()).model
            classificationModel = try VNCoreMLModel(for: classifierMLModel)
            
            // Object Detector
            let detectorMLModel = try YOLOv3(configuration: MLModelConfiguration()).model
            detectionModel = try VNCoreMLModel(for: detectorMLModel)
            
            print("โหลดโมเดลสำเร็จ")
        } catch {
            print("ข้อผิดพลาดในการโหลดโมเดล: \(error)")
        }
    }
    
    // MARK: - Classification Pipeline
    func classifyImage(_ image: UIImage, completion: @escaping ([VNClassificationObservation]) -> Void) {
        guard let model = classificationModel,
              let ciImage = CIImage(image: image) else {
            completion([])
            return
        }
        
        processingQueue.async {
            let request = VNCoreMLRequest(model: model) { request, error in
                guard let results = request.results as? [VNClassificationObservation] else {
                    DispatchQueue.main.async { completion([]) }
                    return
                }
                DispatchQueue.main.async { completion(results) }
            }
            request.imageCropAndScaleOption = .centerCrop
            
            let handler = VNImageRequestHandler(ciImage: ciImage, options: [:])
            try? handler.perform([request])
        }
    }
    
    // MARK: - Detection Pipeline
    func detectObjects(in image: UIImage, completion: @escaping ([VNRecognizedObjectObservation]) -> Void) {
        guard let model = detectionModel,
              let ciImage = CIImage(image: image) else {
            completion([])
            return
        }
        
        processingQueue.async {
            let request = VNCoreMLRequest(model: model) { request, error in
                guard let results = request.results as? [VNRecognizedObjectObservation] else {
                    DispatchQueue.main.async { completion([]) }
                    return
                }
                
                // กรองเฉพาะที่มี confidence สูงกว่า 50%
                let filtered = results.filter { 
                    ($0.labels.first?.confidence ?? 0) > 0.5 
                }
                DispatchQueue.main.async { completion(filtered) }
            }
            request.imageCropAndScaleOption = .scaleFill
            
            let handler = VNImageRequestHandler(ciImage: ciImage, options: [:])
            try? handler.perform([request])
        }
    }
    
    // MARK: - Combined Pipeline (ทำทั้งสองอย่างพร้อมกัน)
    func analyzeImage(_ image: UIImage, completion: @escaping (
        [VNClassificationObservation],
        [VNRecognizedObjectObservation]
    ) -> Void) {
        guard let classModel = classificationModel,
              let detModel = detectionModel,
              let ciImage = CIImage(image: image) else {
            completion([], [])
            return
        }
        
        processingQueue.async {
            var classifications: [VNClassificationObservation] = []
            var detections: [VNRecognizedObjectObservation] = []
            
            let classRequest = VNCoreMLRequest(model: classModel) { request, _ in
                classifications = request.results as? [VNClassificationObservation] ?? []
            }
            
            let detRequest = VNCoreMLRequest(model: detModel) { request, _ in
                detections = request.results as? [VNRecognizedObjectObservation] ?? []
            }
            
            let handler = VNImageRequestHandler(ciImage: ciImage, options: [:])
            
            do {
                // รัน requests พร้อมกัน (batch processing)
                try handler.perform([classRequest, detRequest])
                DispatchQueue.main.async {
                    completion(classifications, detections)
                }
            } catch {
                print("Error: \(error)")
                DispatchQueue.main.async { completion([], []) }
            }
        }
    }
}
```

---

## 49.8 Image Classification Implementation

### สร้าง Image Classifier อย่างสมบูรณ์

```swift
import UIKit
import CoreML
import Vision

// MARK: - Model
class ImageClassificationModel {
    // Result Structure
    struct ClassificationResult {
        let label: String
        let confidence: Float
        let topResults: [(String, Float)]
        
        var confidencePercentage: String {
            return String(format: "%.1f%%", confidence * 100)
        }
    }
    
    // Error Types
    enum ClassificationError: Error {
        case modelLoadFailed
        case imageConversionFailed
        case inferenceFailed(Error)
        
        var localizedDescription: String {
            switch self {
            case .modelLoadFailed:
                return "ไม่สามารถโหลดโมเดลได้"
            case .imageConversionFailed:
                return "ไม่สามารถแปลงรูปภาพได้"
            case .inferenceFailed(let error):
                return "การประมวลผลล้มเหลว: \(error.localizedDescription)"
            }
        }
    }
    
    private let visionModel: VNCoreMLModel
    private let processingQueue = DispatchQueue(label: "com.app.classification")
    
    init() throws {
        let config = MLModelConfiguration()
        config.computeUnits = .all
        
        guard let mlModel = try? MobileNetV2(configuration: config).model else {
            throw ClassificationError.modelLoadFailed
        }
        
        visionModel = try VNCoreMLModel(for: mlModel)
    }
    
    // MARK: - Classification Methods
    
    func classify(
        image: UIImage,
        topK: Int = 5,
        completion: @escaping (Result<ClassificationResult, ClassificationError>) -> Void
    ) {
        guard let ciImage = CIImage(image: image) else {
            completion(.failure(.imageConversionFailed))
            return
        }
        
        processingQueue.async { [weak self] in
            guard let self = self else { return }
            
            let request = VNCoreMLRequest(model: self.visionModel) { request, error in
                if let error = error {
                    DispatchQueue.main.async {
                        completion(.failure(.inferenceFailed(error)))
                    }
                    return
                }
                
                guard let results = request.results as? [VNClassificationObservation],
                      let topResult = results.first else {
                    DispatchQueue.main.async {
                        completion(.failure(.inferenceFailed(
                            NSError(domain: "NoResults", code: 0, userInfo: nil)
                        )))
                    }
                    return
                }
                
                let topResults = results.prefix(topK).map { ($0.identifier, $0.confidence) }
                let result = ClassificationResult(
                    label: topResult.identifier,
                    confidence: topResult.confidence,
                    topResults: topResults
                )
                
                DispatchQueue.main.async {
                    completion(.success(result))
                }
            }
            
            request.imageCropAndScaleOption = .centerCrop
            
            let handler = VNImageRequestHandler(
                ciImage: ciImage,
                orientation: .up,
                options: [:]
            )
            
            do {
                try handler.perform([request])
            } catch {
                DispatchQueue.main.async {
                    completion(.failure(.inferenceFailed(error)))
                }
            }
        }
    }
    
    // Async/Await version
    @available(iOS 15.0, *)
    func classify(image: UIImage, topK: Int = 5) async throws -> ClassificationResult {
        return try await withCheckedThrowingContinuation { continuation in
            classify(image: image, topK: topK) { result in
                continuation.resume(with: result)
            }
        }
    }
}

// MARK: - ViewController
class ImageClassificationViewController: UIViewController {
    
    // UI Elements
    private let imageView: UIImageView = {
        let iv = UIImageView()
        iv.contentMode = .scaleAspectFit
        iv.backgroundColor = .systemGray6
        iv.layer.cornerRadius = 12
        iv.clipsToBounds = true
        iv.translatesAutoresizingMaskIntoConstraints = false
        return iv
    }()
    
    private let resultLabel: UILabel = {
        let label = UILabel()
        label.text = "เลือกรูปภาพเพื่อจำแนก"
        label.textAlignment = .center
        label.numberOfLines = 0
        label.font = .systemFont(ofSize: 18, weight: .medium)
        label.translatesAutoresizingMaskIntoConstraints = false
        return label
    }()
    
    private let confidenceLabel: UILabel = {
        let label = UILabel()
        label.textAlignment = .center
        label.textColor = .systemGray
        label.font = .systemFont(ofSize: 14)
        label.translatesAutoresizingMaskIntoConstraints = false
        return label
    }()
    
    private let selectButton: UIButton = {
        let button = UIButton(type: .system)
        button.setTitle("เลือกรูปภาพ", for: .normal)
        button.titleLabel?.font = .systemFont(ofSize: 16, weight: .semibold)
        button.backgroundColor = .systemBlue
        button.setTitleColor(.white, for: .normal)
        button.layer.cornerRadius = 12
        button.translatesAutoresizingMaskIntoConstraints = false
        return button
    }()
    
    private let activityIndicator: UIActivityIndicatorView = {
        let ai = UIActivityIndicatorView(style: .large)
        ai.hidesWhenStopped = true
        ai.translatesAutoresizingMaskIntoConstraints = false
        return ai
    }()
    
    private var classifier: ImageClassificationModel?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
        setupModel()
    }
    
    private func setupUI() {
        title = "Image Classifier"
        view.backgroundColor = .systemBackground
        
        [imageView, resultLabel, confidenceLabel, selectButton, activityIndicator].forEach {
            view.addSubview($0)
        }
        
        NSLayoutConstraint.activate([
            imageView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 20),
            imageView.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 20),
            imageView.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -20),
            imageView.heightAnchor.constraint(equalTo: imageView.widthAnchor),
            
            activityIndicator.centerXAnchor.constraint(equalTo: imageView.centerXAnchor),
            activityIndicator.centerYAnchor.constraint(equalTo: imageView.centerYAnchor),
            
            resultLabel.topAnchor.constraint(equalTo: imageView.bottomAnchor, constant: 20),
            resultLabel.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 20),
            resultLabel.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -20),
            
            confidenceLabel.topAnchor.constraint(equalTo: resultLabel.bottomAnchor, constant: 8),
            confidenceLabel.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 20),
            confidenceLabel.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -20),
            
            selectButton.bottomAnchor.constraint(equalTo: view.safeAreaLayoutGuide.bottomAnchor, constant: -20),
            selectButton.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 40),
            selectButton.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -40),
            selectButton.heightAnchor.constraint(equalToConstant: 50)
        ])
        
        selectButton.addTarget(self, action: #selector(selectImageTapped), for: .touchUpInside)
    }
    
    private func setupModel() {
        do {
            classifier = try ImageClassificationModel()
            print("โมเดลพร้อมใช้งาน")
        } catch {
            showAlert(title: "ข้อผิดพลาด", message: "ไม่สามารถโหลดโมเดล ML ได้")
        }
    }
    
    @objc private func selectImageTapped() {
        let picker = UIImagePickerController()
        picker.sourceType = .photoLibrary
        picker.delegate = self
        present(picker, animated: true)
    }
    
    private func classifyImage(_ image: UIImage) {
        activityIndicator.startAnimating()
        resultLabel.text = "กำลังวิเคราะห์..."
        confidenceLabel.text = ""
        
        classifier?.classify(image: image, topK: 5) { [weak self] result in
            self?.activityIndicator.stopAnimating()
            
            switch result {
            case .success(let classification):
                self?.resultLabel.text = classification.label
                self?.confidenceLabel.text = "ความมั่นใจ: \(classification.confidencePercentage)"
                
                // แสดง top 5 results
                let topResultsText = classification.topResults
                    .map { "\($0.0): \(String(format: "%.1f%%", $0.1 * 100))" }
                    .joined(separator: "\n")
                print("Top 5 Results:\n\(topResultsText)")
                
            case .failure(let error):
                self?.resultLabel.text = "เกิดข้อผิดพลาด"
                self?.showAlert(title: "ข้อผิดพลาด", message: error.localizedDescription)
            }
        }
    }
    
    private func showAlert(title: String, message: String) {
        let alert = UIAlertController(title: title, message: message, preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "ตกลง", style: .default))
        present(alert, animated: true)
    }
}

// MARK: - UIImagePickerControllerDelegate
extension ImageClassificationViewController: UIImagePickerControllerDelegate, UINavigationControllerDelegate {
    func imagePickerController(
        _ picker: UIImagePickerController,
        didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey: Any]
    ) {
        picker.dismiss(animated: true)
        
        guard let image = info[.originalImage] as? UIImage else { return }
        imageView.image = image
        classifyImage(image)
    }
    
    func imagePickerControllerDidCancel(_ picker: UIImagePickerController) {
        picker.dismiss(animated: true)
    }
}
```

---

## 49.9 Object Detection Implementation

```swift
import UIKit
import CoreML
import Vision

// MARK: - Bounding Box View
class BoundingBoxView: UIView {
    private let label: UILabel = {
        let l = UILabel()
        l.font = .systemFont(ofSize: 12, weight: .bold)
        l.textColor = .white
        l.backgroundColor = .systemBlue.withAlphaComponent(0.8)
        l.padding = UIEdgeInsets(top: 2, left: 4, bottom: 2, right: 4)
        l.translatesAutoresizingMaskIntoConstraints = false
        return l
    }()
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        setupView()
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
        setupView()
    }
    
    private func setupView() {
        layer.borderWidth = 2
        layer.borderColor = UIColor.systemBlue.cgColor
        backgroundColor = UIColor.systemBlue.withAlphaComponent(0.1)
        addSubview(label)
        
        NSLayoutConstraint.activate([
            label.topAnchor.constraint(equalTo: topAnchor),
            label.leadingAnchor.constraint(equalTo: leadingAnchor)
        ])
    }
    
    func configure(label: String, confidence: Float, color: UIColor = .systemBlue) {
        self.label.text = "\(label) \(String(format: "%.0f%%", confidence * 100))"
        layer.borderColor = color.cgColor
        backgroundColor = color.withAlphaComponent(0.1)
        self.label.backgroundColor = color.withAlphaComponent(0.8)
    }
}

// MARK: - Object Detection ViewController
class ObjectDetectionViewController: UIViewController {
    
    private let imageView = UIImageView()
    private let overlayView = UIView()
    private var boundingBoxViews: [BoundingBoxView] = []
    
    // Core ML Model
    private var detector: VNCoreMLModel?
    
    // สีสำหรับ classes ต่างๆ
    private let classColors: [String: UIColor] = [
        "person": .systemRed,
        "car": .systemBlue,
        "dog": .systemGreen,
        "cat": .systemOrange,
        "bicycle": .systemPurple
    ]
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
        loadModel()
    }
    
    private func setupUI() {
        view.backgroundColor = .black
        
        imageView.frame = view.bounds
        imageView.contentMode = .scaleAspectFit
        view.addSubview(imageView)
        
        overlayView.frame = view.bounds
        overlayView.backgroundColor = .clear
        view.addSubview(overlayView)
        
        // ปุ่มเลือกรูป
        let button = UIButton(frame: CGRect(x: 20, y: view.frame.height - 80, width: 120, height: 44))
        button.setTitle("เลือกรูป", for: .normal)
        button.backgroundColor = .systemBlue
        button.layer.cornerRadius = 8
        button.addTarget(self, action: #selector(selectImage), for: .touchUpInside)
        view.addSubview(button)
    }
    
    private func loadModel() {
        do {
            let model = try YOLOv3(configuration: MLModelConfiguration()).model
            detector = try VNCoreMLModel(for: model)
        } catch {
            print("ไม่สามารถโหลด YOLO model ได้: \(error)")
        }
    }
    
    @objc private func selectImage() {
        let picker = UIImagePickerController()
        picker.sourceType = .photoLibrary
        picker.delegate = self
        present(picker, animated: true)
    }
    
    private func detectObjects(in image: UIImage) {
        guard let model = detector,
              let ciImage = CIImage(image: image) else { return }
        
        let request = VNCoreMLRequest(model: model) { [weak self] request, error in
            guard let results = request.results as? [VNRecognizedObjectObservation] else { return }
            
            DispatchQueue.main.async {
                self?.drawBoundingBoxes(results: results, imageSize: image.size)
            }
        }
        request.imageCropAndScaleOption = .scaleFill
        
        let handler = VNImageRequestHandler(ciImage: ciImage)
        DispatchQueue.global(qos: .userInitiated).async {
            try? handler.perform([request])
        }
    }
    
    private func drawBoundingBoxes(results: [VNRecognizedObjectObservation], imageSize: CGSize) {
        // ลบ bounding boxes เดิม
        boundingBoxViews.forEach { $0.removeFromSuperview() }
        boundingBoxViews.removeAll()
        
        guard let displayedImageSize = imageView.displayedImageSize else { return }
        let (offsetX, offsetY) = imageView.imageOffset(for: imageSize)
        
        for result in results {
            guard let topLabel = result.labels.first,
                  topLabel.confidence > 0.5 else { continue }
            
            // แปลง normalized coordinates เป็น view coordinates
            let boundingBox = result.boundingBox
            let x = boundingBox.minX * displayedImageSize.width + offsetX
            let y = (1 - boundingBox.maxY) * displayedImageSize.height + offsetY
            let width = boundingBox.width * displayedImageSize.width
            let height = boundingBox.height * displayedImageSize.height
            
            let boxView = BoundingBoxView(frame: CGRect(x: x, y: y, width: width, height: height))
            let color = classColors[topLabel.identifier] ?? .systemBlue
            boxView.configure(label: topLabel.identifier, confidence: topLabel.confidence, color: color)
            
            overlayView.addSubview(boxView)
            boundingBoxViews.append(boxView)
        }
    }
}

// MARK: - UIImageView Extension
extension UIImageView {
    var displayedImageSize: CGSize? {
        guard let image = image else { return nil }
        let imageAspect = image.size.width / image.size.height
        let viewAspect = bounds.width / bounds.height
        
        if imageAspect > viewAspect {
            return CGSize(width: bounds.width, height: bounds.width / imageAspect)
        } else {
            return CGSize(width: bounds.height * imageAspect, height: bounds.height)
        }
    }
    
    func imageOffset(for imageSize: CGSize) -> (CGFloat, CGFloat) {
        guard let displaySize = displayedImageSize else { return (0, 0) }
        let offsetX = (bounds.width - displaySize.width) / 2
        let offsetY = (bounds.height - displaySize.height) / 2
        return (offsetX, offsetY)
    }
}

extension ObjectDetectionViewController: UIImagePickerControllerDelegate, UINavigationControllerDelegate {
    func imagePickerController(
        _ picker: UIImagePickerController,
        didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey: Any]
    ) {
        picker.dismiss(animated: true)
        guard let image = info[.originalImage] as? UIImage else { return }
        imageView.image = image
        detectObjects(in: image)
    }
    
    func imagePickerControllerDidCancel(_ picker: UIImagePickerController) {
        picker.dismiss(animated: true)
    }
}
```

---

## 49.10 Natural Language Processing ด้วย Create ML

```swift
import NaturalLanguage
import CoreML

// MARK: - Text Classifier
class TextClassificationManager {
    
    // ประเภทของ text classification tasks
    enum ClassificationTask {
        case sentiment      // วิเคราะห์ความรู้สึก
        case topicDetection // ตรวจจับหัวข้อ
        case language       // ตรวจจับภาษา
        case spam           // ตรวจจับ spam
    }
    
    // Language Identifier (built-in)
    func identifyLanguage(text: String) -> String {
        let recognizer = NLLanguageRecognizer()
        recognizer.processString(text)
        
        guard let language = recognizer.dominantLanguage else {
            return "ไม่ทราบภาษา"
        }
        return language.rawValue
    }
    
    // Tokenization
    func tokenize(text: String, unit: NLTokenUnit = .word) -> [String] {
        var tokens: [String] = []
        let tokenizer = NLTokenizer(unit: unit)
        tokenizer.string = text
        
        tokenizer.enumerateTokens(in: text.startIndex..<text.endIndex) { range, _ in
            tokens.append(String(text[range]))
            return true
        }
        return tokens
    }
    
    // Named Entity Recognition
    func extractEntities(from text: String) -> [(String, NLTag)] {
        var entities: [(String, NLTag)] = []
        let tagger = NLTagger(tagSchemes: [.nameType])
        tagger.string = text
        
        let options: NLTagger.Options = [.omitWhitespace, .omitPunctuation, .joinNames]
        let tags: [NLTag] = [.personalName, .placeName, .organizationName]
        
        tagger.enumerateTags(
            in: text.startIndex..<text.endIndex,
            unit: .word,
            scheme: .nameType,
            options: options
        ) { tag, range in
            if let tag = tag, tags.contains(tag) {
                entities.append((String(text[range]), tag))
            }
            return true
        }
        return entities
    }
    
    // Custom Sentiment Model
    func analyzeSentiment(text: String, model: NLModel) -> (label: String, confidence: Double) {
        let label = model.predictedLabel(for: text) ?? "neutral"
        let hypotheses = model.predictedLabelHypotheses(for: text, maximumCount: 1)
        let confidence = hypotheses[label] ?? 0.5
        return (label, confidence)
    }
}

// MARK: - การใช้งาน NLP
func demonstrateNLP() {
    let manager = TextClassificationManager()
    
    // ตรวจจับภาษา
    let text1 = "สวัสดีครับ นี่คือภาษาไทย"
    let text2 = "Hello, this is English text"
    print("ภาษาของ text1: \(manager.identifyLanguage(text: text1))")
    print("ภาษาของ text2: \(manager.identifyLanguage(text: text2))")
    
    // Tokenization
    let sentence = "Core ML เป็น framework ที่ยอดเยี่ยมมาก"
    let words = manager.tokenize(text: sentence)
    print("คำทั้งหมด: \(words)")
    
    // Named Entity Recognition
    let newsText = "นายสมชาย ใจดี จาก กรุงเทพมหานคร เดินทางไป Apple ที่ California"
    let entities = manager.extractEntities(from: newsText)
    for (entity, tag) in entities {
        print("\(entity): \(tag.rawValue)")
    }
}
```

---

## 49.11 Core ML Model Performance

### การวัดประสิทธิภาพ

```swift
import CoreML
import Foundation

class ModelPerformanceTracker {
    
    struct PerformanceMetrics {
        let inferenceTime: TimeInterval
        let preprocessingTime: TimeInterval
        let totalTime: TimeInterval
        var fps: Double { 1.0 / totalTime }
    }
    
    private var metrics: [PerformanceMetrics] = []
    
    func measureInference<T>(
        name: String,
        block: () throws -> T
    ) rethrows -> T {
        let start = CFAbsoluteTimeGetCurrent()
        let result = try block()
        let elapsed = CFAbsoluteTimeGetCurrent() - start
        
        print("⏱ \(name): \(String(format: "%.3f", elapsed * 1000))ms")
        return result
    }
    
    // ทดสอบประสิทธิภาพของโมเดล
    func benchmark(model: MLModel, input: MLFeatureProvider, iterations: Int = 100) {
        var times: [TimeInterval] = []
        
        for i in 0..<iterations {
            let start = CFAbsoluteTimeGetCurrent()
            _ = try? model.prediction(from: input)
            let elapsed = CFAbsoluteTimeGetCurrent() - start
            times.append(elapsed)
            
            if i == 0 {
                print("First inference (cold start): \(String(format: "%.3f", elapsed * 1000))ms")
            }
        }
        
        // คำนวณ statistics
        let warmTimes = Array(times.dropFirst()) // ตัด cold start ออก
        let avgTime = warmTimes.reduce(0, +) / Double(warmTimes.count)
        let minTime = warmTimes.min() ?? 0
        let maxTime = warmTimes.max() ?? 0
        
        print("\n=== Benchmark Results (\(iterations) iterations) ===")
        print("Average: \(String(format: "%.2f", avgTime * 1000))ms")
        print("Min: \(String(format: "%.2f", minTime * 1000))ms")
        print("Max: \(String(format: "%.2f", maxTime * 1000))ms")
        print("FPS (avg): \(String(format: "%.1f", 1.0 / avgTime))")
    }
    
    // เปรียบเทียบ compute units
    func compareComputeUnits(modelURL: URL, input: MLFeatureProvider) {
        let units: [(String, MLComputeUnits)] = [
            ("CPU Only", .cpuOnly),
            ("CPU + GPU", .cpuAndGPU),
            ("CPU + Neural Engine", .cpuAndNeuralEngine),
            ("All (Auto)", .all)
        ]
        
        print("\n=== Compute Units Comparison ===")
        
        for (name, unit) in units {
            let config = MLModelConfiguration()
            config.computeUnits = unit
            
            guard let model = try? MLModel(contentsOf: modelURL, configuration: config) else {
                print("\(name): ไม่สามารถโหลดได้")
                continue
            }
            
            // Warm up
            _ = try? model.prediction(from: input)
            
            // Benchmark
            var times: [TimeInterval] = []
            for _ in 0..<50 {
                let start = CFAbsoluteTimeGetCurrent()
                _ = try? model.prediction(from: input)
                times.append(CFAbsoluteTimeGetCurrent() - start)
            }
            
            let avg = times.reduce(0, +) / Double(times.count)
            print("\(name): \(String(format: "%.2f", avg * 1000))ms avg")
        }
    }
}
```

---

## 49.12 Quantization และ Compression

### Model Quantization

Quantization คือการลดความแม่นยำของน้ำหนักโมเดล เพื่อลดขนาดและเพิ่มความเร็ว

```swift
// การใช้ Core ML Tools (Python) เพื่อ Quantize
// *** รันใน Python environment ***
/*
import coremltools as ct

# โหลดโมเดล
model = ct.models.MLModel('MyModel.mlmodel')

# 8-bit quantization
quantized_model = ct.compression_utils.affine_quantize_weights(model, mode="linear_symmetric")
quantized_model.save('MyModel_Quantized_8bit.mlmodel')

# 4-bit quantization (ขนาดเล็กกว่าแต่ accuracy อาจลดลง)
quantized_model_4bit = ct.compression_utils.affine_quantize_weights(model, mode="linear_symmetric", nbits=4)
quantized_model_4bit.save('MyModel_Quantized_4bit.mlmodel')

# Palettization (clustering weights)
palettized = ct.compression_utils.palettize_weights(model, nbits=4)
palettized.save('MyModel_Palettized.mlmodel')
*/

// Swift: ตรวจสอบขนาดโมเดล
func checkModelSize(modelName: String) {
    guard let url = Bundle.main.url(forResource: modelName, withExtension: "mlmodelc") else {
        print("ไม่พบโมเดล: \(modelName)")
        return
    }
    
    do {
        let attributes = try FileManager.default.attributesOfItem(atPath: url.path)
        if let size = attributes[.size] as? Int64 {
            let sizeInMB = Double(size) / (1024 * 1024)
            print("\(modelName) ขนาด: \(String(format: "%.2f", sizeInMB)) MB")
        }
    } catch {
        print("ข้อผิดพลาด: \(error)")
    }
}
```

### ประเภทของ Compression

```swift
// สรุปประเภทของ Model Compression
struct ModelCompressionTypes {
    /*
    1. Weight Quantization (การลด precision ของ weights)
       - FP32 → FP16: ขนาดลด 50%, ความเร็วเพิ่ม, accuracy คงเดิม
       - FP32 → INT8:  ขนาดลด 75%, เร็วขึ้น, accuracy อาจลดเล็กน้อย
       - FP32 → INT4:  ขนาดลด 87.5%, เร็วมาก, accuracy ลดลงมากขึ้น
    
    2. Pruning (ตัด weights ที่ไม่จำเป็นออก)
       - Unstructured Pruning: ตัด individual weights
       - Structured Pruning: ตัด channels/layers
    
    3. Knowledge Distillation (สอนโมเดลเล็กจากโมเดลใหญ่)
       - Teacher model: โมเดลใหญ่ที่แม่นยำ
       - Student model: โมเดลเล็กที่เรียนรู้จาก teacher
    
    4. Palettization (รวม weights ที่คล้ายกัน)
       - ใช้ lookup table แทน individual values
       - ลดขนาดได้มากโดยสูญเสีย accuracy น้อย
    */
}
```

---

## 49.13 On-Device Inference Benefits

```swift
// ข้อดีของ On-Device Inference
class OnDeviceInferenceDemo {
    
    // 1. Privacy - ข้อมูลไม่ถูกส่งออก
    func processPrivateData(image: UIImage) {
        // รูปภาพประมวลผลบนอุปกรณ์ ไม่ส่งไป server
        // ข้อมูลผู้ใช้ได้รับการปกป้อง
        classifyLocally(image: image)
    }
    
    // 2. Speed - ไม่มี network latency
    func measureLatency() {
        let start = Date()
        
        // On-device: ~10-50ms
        // Server-based: ~200-1000ms (network + compute + return)
        
        let elapsed = Date().timeIntervalSince(start)
        print("Latency: \(elapsed * 1000)ms")
    }
    
    // 3. Offline Capability
    func checkConnectivity() -> Bool {
        // On-device inference ทำงานได้แม้ไม่มีอินเทอร์เน็ต
        return true // ทำงานได้เสมอ
    }
    
    // 4. Cost Efficiency - ไม่มีค่า API calls
    func estimateCostSavings(requestsPerDay: Int) -> Double {
        // ค่าใช้จ่าย server inference: ~$0.001-0.01 per request
        // On-device: ฟรี (หลังจาก download โมเดลครั้งเดียว)
        let serverCostPerRequest = 0.005
        let dailySavings = Double(requestsPerDay) * serverCostPerRequest
        return dailySavings * 365 // yearly savings
    }
    
    private func classifyLocally(image: UIImage) {
        // implementation...
    }
}
```

---

## 49.14 Core ML กับ SwiftUI

```swift
import SwiftUI
import CoreML
import Vision

// MARK: - ViewModel
@MainActor
class ImageClassifierViewModel: ObservableObject {
    @Published var selectedImage: UIImage?
    @Published var classificationResult: String = "เลือกรูปภาพเพื่อวิเคราะห์"
    @Published var confidence: Float = 0
    @Published var isProcessing: Bool = false
    @Published var topResults: [(String, Float)] = []
    
    private var model: VNCoreMLModel?
    
    init() {
        loadModel()
    }
    
    private func loadModel() {
        Task {
            do {
                let config = MLModelConfiguration()
                config.computeUnits = .all
                let mlModel = try MobileNetV2(configuration: config).model
                model = try VNCoreMLModel(for: mlModel)
            } catch {
                classificationResult = "ไม่สามารถโหลดโมเดลได้"
            }
        }
    }
    
    func classify() {
        guard let image = selectedImage, let model = model else { return }
        
        isProcessing = true
        
        Task.detached(priority: .userInitiated) { [weak self] in
            guard let self = self else { return }
            
            guard let ciImage = CIImage(image: image) else {
                await MainActor.run { self.isProcessing = false }
                return
            }
            
            let request = VNCoreMLRequest(model: model) { request, error in
                Task { @MainActor in
                    self.isProcessing = false
                    
                    guard let results = request.results as? [VNClassificationObservation] else { return }
                    
                    if let top = results.first {
                        self.classificationResult = top.identifier
                        self.confidence = top.confidence
                    }
                    
                    self.topResults = results.prefix(5).map { ($0.identifier, $0.confidence) }
                }
            }
            request.imageCropAndScaleOption = .centerCrop
            
            let handler = VNImageRequestHandler(ciImage: ciImage)
            try? handler.perform([request])
        }
    }
}

// MARK: - SwiftUI Views
struct ImageClassifierView: View {
    @StateObject private var viewModel = ImageClassifierViewModel()
    @State private var showingImagePicker = false
    
    var body: some View {
        NavigationView {
            VStack(spacing: 20) {
                // Image Display
                ZStack {
                    if let image = viewModel.selectedImage {
                        Image(uiImage: image)
                            .resizable()
                            .scaledToFit()
                            .cornerRadius(12)
                    } else {
                        RoundedRectangle(cornerRadius: 12)
                            .fill(Color(.systemGray6))
                            .overlay(
                                VStack {
                                    Image(systemName: "photo")
                                        .font(.system(size: 50))
                                        .foregroundColor(.secondary)
                                    Text("แตะเพื่อเลือกรูปภาพ")
                                        .foregroundColor(.secondary)
                                }
                            )
                    }
                    
                    if viewModel.isProcessing {
                        Color.black.opacity(0.4)
                            .cornerRadius(12)
                        ProgressView()
                            .progressViewStyle(CircularProgressViewStyle(tint: .white))
                            .scaleEffect(2)
                    }
                }
                .frame(height: 300)
                .onTapGesture {
                    showingImagePicker = true
                }
                
                // Results
                VStack(alignment: .leading, spacing: 12) {
                    Text("ผลลัพธ์:")
                        .font(.headline)
                    
                    HStack {
                        Text(viewModel.classificationResult)
                            .font(.title2)
                            .bold()
                        Spacer()
                        Text("\(Int(viewModel.confidence * 100))%")
                            .font(.title2)
                            .foregroundColor(.blue)
                    }
                    
                    // Confidence Bar
                    if viewModel.confidence > 0 {
                        GeometryReader { geo in
                            ZStack(alignment: .leading) {
                                Capsule()
                                    .fill(Color(.systemGray5))
                                    .frame(height: 8)
                                Capsule()
                                    .fill(Color.blue)
                                    .frame(width: geo.size.width * CGFloat(viewModel.confidence), height: 8)
                            }
                        }
                        .frame(height: 8)
                    }
                    
                    // Top 5 Results
                    if !viewModel.topResults.isEmpty {
                        Divider()
                        Text("Top 5 Predictions:")
                            .font(.subheadline)
                            .foregroundColor(.secondary)
                        
                        ForEach(viewModel.topResults, id: \.0) { label, confidence in
                            HStack {
                                Text(label)
                                    .font(.caption)
                                Spacer()
                                Text("\(String(format: "%.1f", confidence * 100))%")
                                    .font(.caption)
                                    .foregroundColor(.secondary)
                            }
                        }
                    }
                }
                .padding()
                .background(Color(.systemBackground))
                .cornerRadius(12)
                .shadow(radius: 2)
                
                Spacer()
            }
            .padding()
            .navigationTitle("Image Classifier")
        }
        .sheet(isPresented: $showingImagePicker) {
            ImagePicker(image: $viewModel.selectedImage, onSelect: {
                viewModel.classify()
            })
        }
    }
}

// MARK: - ImagePicker
struct ImagePicker: UIViewControllerRepresentable {
    @Binding var image: UIImage?
    let onSelect: () -> Void
    @Environment(\.dismiss) var dismiss
    
    func makeUIViewController(context: Context) -> UIImagePickerController {
        let picker = UIImagePickerController()
        picker.sourceType = .photoLibrary
        picker.delegate = context.coordinator
        return picker
    }
    
    func updateUIViewController(_ uiViewController: UIImagePickerController, context: Context) {}
    
    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }
    
    class Coordinator: NSObject, UIImagePickerControllerDelegate, UINavigationControllerDelegate {
        let parent: ImagePicker
        
        init(_ parent: ImagePicker) {
            self.parent = parent
        }
        
        func imagePickerController(
            _ picker: UIImagePickerController,
            didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey: Any]
        ) {
            parent.image = info[.originalImage] as? UIImage
            parent.dismiss()
            parent.onSelect()
        }
        
        func imagePickerControllerDidCancel(_ picker: UIImagePickerController) {
            parent.dismiss()
        }
    }
}
```

---

## 49.15 Custom Core ML Models

### การสร้าง Custom Model ด้วย Create ML (macOS)

```swift
// *** รันบน macOS Playground หรือ Create ML App ***
import CreateML
import Foundation

// MARK: - Image Classifier
func createCustomImageClassifier() async throws {
    // โหลด training data จาก folder structure:
    // TrainingData/
    //   ├── cats/
    //   │   ├── cat1.jpg
    //   │   └── cat2.jpg
    //   └── dogs/
    //       ├── dog1.jpg
    //       └── dog2.jpg
    
    let trainingDataURL = URL(fileURLWithPath: "/Users/username/Desktop/TrainingData")
    
    // สร้าง training data source
    let trainingData = MLImageClassifier.DataSource.labeledFiles(at: trainingDataURL)
    
    // กำหนดค่า parameters
    var parameters = MLImageClassifier.ModelParameters()
    parameters.validation = .split(strategy: .automatic)
    parameters.maxIterations = 30
    parameters.augmentation = .init(
        flipping: .horizontal,
        rotation: .init(maximumAngle: 30),
        exposure: .init(magnitude: 0.5),
        noise: .init(magnitude: 0.1)
    )
    parameters.featureExtractor = .scenePrint(revision: 2)
    
    print("เริ่มเทรนโมเดล...")
    
    // เทรนโมเดล
    let classifier = try await MLImageClassifier(
        trainingData: trainingData,
        parameters: parameters
    )
    
    // ดู metrics
    print("Training Accuracy: \(classifier.trainingMetrics.classificationError)")
    if let validationMetrics = classifier.validationMetrics {
        print("Validation Accuracy: \(validationMetrics.classificationError)")
    }
    
    // บันทึกโมเดล
    let outputURL = URL(fileURLWithPath: "/Users/username/Desktop/CatDogClassifier.mlmodel")
    let metadata = MLModelMetadata(
        author: "Your Name",
        shortDescription: "Classifies cats and dogs",
        license: "MIT",
        version: "1.0",
        additional: ["Training images": "1000"]
    )
    
    try classifier.write(to: outputURL, metadata: metadata)
    print("บันทึกโมเดลที่: \(outputURL.path)")
}

// MARK: - Text Classifier
func createCustomTextClassifier() async throws {
    // Training data format: CSV with "text" and "label" columns
    // text,label
    // "สินค้าดีมาก",positive
    // "ไม่ชอบเลย",negative
    
    let trainingDataURL = URL(fileURLWithPath: "/Users/username/Desktop/sentiment_data.csv")
    let trainingData = try MLDataTable(contentsOf: trainingDataURL)
    
    var parameters = MLTextClassifier.ModelParameters()
    parameters.validation = .split(strategy: .automatic)
    parameters.algorithm = .maxEnt(revision: 1)
    
    let classifier = try await MLTextClassifier(
        trainingData: trainingData,
        textColumn: "text",
        labelColumn: "label",
        parameters: parameters
    )
    
    print("Training Accuracy: \(1 - classifier.trainingMetrics.classificationError)")
    
    let outputURL = URL(fileURLWithPath: "/Users/username/Desktop/SentimentClassifier.mlmodel")
    try classifier.write(to: outputURL)
}
```

---

## 49.16 YOLO Integration

```swift
import UIKit
import CoreML
import Vision

// MARK: - YOLO Object Detector
class YOLODetector {
    
    struct Detection {
        let label: String
        let confidence: Float
        let boundingBox: CGRect  // Normalized (0-1)
    }
    
    private var model: VNCoreMLModel?
    private let confidenceThreshold: Float = 0.5
    private let iouThreshold: Float = 0.45
    
    // YOLO class names
    private let classNames = [
        "person", "bicycle", "car", "motorcycle", "airplane", "bus", "train",
        "truck", "boat", "traffic light", "fire hydrant", "stop sign",
        "parking meter", "bench", "bird", "cat", "dog", "horse", "sheep",
        "cow", "elephant", "bear", "zebra", "giraffe", "backpack", "umbrella",
        "handbag", "tie", "suitcase", "frisbee", "skis", "snowboard",
        "sports ball", "kite", "baseball bat", "baseball glove", "skateboard",
        "surfboard", "tennis racket", "bottle", "wine glass", "cup", "fork",
        "knife", "spoon", "bowl", "banana", "apple", "sandwich", "orange",
        "broccoli", "carrot", "hot dog", "pizza", "donut", "cake", "chair",
        "couch", "potted plant", "bed", "dining table", "toilet", "tv",
        "laptop", "mouse", "remote", "keyboard", "cell phone", "microwave",
        "oven", "toaster", "sink", "refrigerator", "book", "clock", "vase",
        "scissors", "teddy bear", "hair drier", "toothbrush"
    ]
    
    init() {
        loadModel()
    }
    
    private func loadModel() {
        do {
            // YOLOv3 หรือ YOLOv3Tiny
            let config = MLModelConfiguration()
            config.computeUnits = .all
            
            // ใช้ YOLOv3Tiny สำหรับ real-time (เร็วกว่า)
            let mlModel = try YOLOv3Tiny(configuration: config).model
            model = try VNCoreMLModel(for: mlModel)
        } catch {
            print("ไม่สามารถโหลด YOLO model: \(error)")
        }
    }
    
    func detect(
        pixelBuffer: CVPixelBuffer,
        completion: @escaping ([Detection]) -> Void
    ) {
        guard let model = model else {
            completion([])
            return
        }
        
        let request = VNCoreMLRequest(model: model) { [weak self] request, error in
            guard let self = self,
                  let results = request.results as? [VNRecognizedObjectObservation] else {
                completion([])
                return
            }
            
            let detections = results
                .filter { ($0.labels.first?.confidence ?? 0) >= self.confidenceThreshold }
                .map { observation -> Detection in
                    let label = observation.labels.first?.identifier ?? "unknown"
                    let confidence = observation.labels.first?.confidence ?? 0
                    return Detection(
                        label: label,
                        confidence: confidence,
                        boundingBox: observation.boundingBox
                    )
                }
            
            completion(detections)
        }
        
        request.imageCropAndScaleOption = .scaleFill
        
        let handler = VNImageRequestHandler(cvPixelBuffer: pixelBuffer, orientation: .up)
        try? handler.perform([request])
    }
}
```

---

## 49.17 Real-Time Camera Inference

```swift
import UIKit
import AVFoundation
import CoreML
import Vision

// MARK: - Real-Time Camera ViewController
class RealTimeCameraMLViewController: UIViewController {
    
    // Camera
    private let captureSession = AVCaptureSession()
    private var previewLayer: AVCaptureVideoPreviewLayer?
    private let videoOutput = AVCaptureVideoDataOutput()
    private let processingQueue = DispatchQueue(label: "com.app.cameraProcessing")
    
    // ML
    private var detector: YOLODetector?
    private var classificationModel: VNCoreMLModel?
    
    // UI
    private var overlayLayer = CALayer()
    private let fpsLabel = UILabel()
    private var frameCount = 0
    private var lastFPSUpdate = Date()
    
    // Throttling - ไม่ต้องประมวลผลทุก frame
    private var lastProcessedTime = Date()
    private let processingInterval: TimeInterval = 0.1 // 10 FPS สำหรับ ML
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupCamera()
        setupML()
        setupOverlay()
        setupFPSLabel()
    }
    
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        DispatchQueue.global(qos: .userInitiated).async { [weak self] in
            self?.captureSession.startRunning()
        }
    }
    
    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        captureSession.stopRunning()
    }
    
    // MARK: - Setup
    private func setupCamera() {
        captureSession.sessionPreset = .hd1280x720
        
        guard let camera = AVCaptureDevice.default(.builtInWideAngleCamera, for: .video, position: .back),
              let input = try? AVCaptureDeviceInput(device: camera) else { return }
        
        captureSession.addInput(input)
        
        // Preview layer
        let previewLayer = AVCaptureVideoPreviewLayer(session: captureSession)
        previewLayer.frame = view.bounds
        previewLayer.videoGravity = .resizeAspectFill
        view.layer.addSublayer(previewLayer)
        self.previewLayer = previewLayer
        
        // Video output
        videoOutput.setSampleBufferDelegate(self, queue: processingQueue)
        videoOutput.alwaysDiscardsLateVideoFrames = true
        captureSession.addOutput(videoOutput)
    }
    
    private func setupML() {
        detector = YOLODetector()
    }
    
    private func setupOverlay() {
        overlayLayer.frame = view.bounds
        view.layer.addSublayer(overlayLayer)
    }
    
    private func setupFPSLabel() {
        fpsLabel.frame = CGRect(x: 16, y: 50, width: 100, height: 30)
        fpsLabel.textColor = .yellow
        fpsLabel.font = .boldSystemFont(ofSize: 14)
        fpsLabel.backgroundColor = .black.withAlphaComponent(0.5)
        fpsLabel.layer.cornerRadius = 4
        fpsLabel.clipsToBounds = true
        fpsLabel.textAlignment = .center
        view.addSubview(fpsLabel)
    }
    
    // MARK: - Draw Detections
    private func drawDetections(_ detections: [YOLODetector.Detection]) {
        DispatchQueue.main.async { [weak self] in
            guard let self = self else { return }
            
            // ลบ layers เดิม
            self.overlayLayer.sublayers?.forEach { $0.removeFromSuperlayer() }
            
            for detection in detections {
                // แปลง YOLO coordinates (origin bottom-left) เป็น UIKit (origin top-left)
                let rect = self.convertBoundingBox(detection.boundingBox)
                
                // Bounding Box
                let boxLayer = CALayer()
                boxLayer.frame = rect
                boxLayer.borderWidth = 2
                boxLayer.borderColor = UIColor.systemGreen.cgColor
                boxLayer.backgroundColor = UIColor.systemGreen.withAlphaComponent(0.1).cgColor
                
                // Label
                let textLayer = CATextLayer()
                textLayer.string = "\(detection.label) \(Int(detection.confidence * 100))%"
                textLayer.fontSize = 12
                textLayer.foregroundColor = UIColor.white.cgColor
                textLayer.backgroundColor = UIColor.systemGreen.withAlphaComponent(0.8).cgColor
                textLayer.frame = CGRect(x: rect.minX, y: rect.minY - 20, width: 120, height: 20)
                textLayer.contentsScale = UIScreen.main.scale
                
                self.overlayLayer.addSublayer(boxLayer)
                self.overlayLayer.addSublayer(textLayer)
            }
            
            // อัพเดต FPS
            self.frameCount += 1
            let now = Date()
            let elapsed = now.timeIntervalSince(self.lastFPSUpdate)
            if elapsed >= 1.0 {
                let fps = Double(self.frameCount) / elapsed
                self.fpsLabel.text = "ML: \(String(format: "%.0f", fps)) FPS"
                self.frameCount = 0
                self.lastFPSUpdate = now
            }
        }
    }
    
    private func convertBoundingBox(_ box: CGRect) -> CGRect {
        let width = view.bounds.width
        let height = view.bounds.height
        
        // YOLO: origin ที่ bottom-left, y เพิ่มขึ้น
        // UIKit: origin ที่ top-left, y เพิ่มลง
        let x = box.minX * width
        let y = (1 - box.maxY) * height
        let w = box.width * width
        let h = box.height * height
        
        return CGRect(x: x, y: y, width: w, height: h)
    }
}

// MARK: - AVCaptureVideoDataOutputSampleBufferDelegate
extension RealTimeCameraMLViewController: AVCaptureVideoDataOutputSampleBufferDelegate {
    func captureOutput(
        _ output: AVCaptureOutput,
        didOutput sampleBuffer: CMSampleBuffer,
        from connection: AVCaptureConnection
    ) {
        // Throttle processing
        let now = Date()
        guard now.timeIntervalSince(lastProcessedTime) >= processingInterval else { return }
        lastProcessedTime = now
        
        guard let pixelBuffer = CMSampleBufferGetImageBuffer(sampleBuffer) else { return }
        
        detector?.detect(pixelBuffer: pixelBuffer) { [weak self] detections in
            self?.drawDetections(detections)
        }
    }
}
```

---

## 49.18 BatchProvider และ MLFeatureProvider

```swift
import CoreML

// MARK: - Custom MLFeatureProvider
class ImageFeatureProvider: NSObject, MLFeatureProvider {
    
    private let pixelBuffer: CVPixelBuffer
    private let inputName: String
    
    var featureNames: Set<String> {
        return [inputName]
    }
    
    init(pixelBuffer: CVPixelBuffer, inputName: String = "image") {
        self.pixelBuffer = pixelBuffer
        self.inputName = inputName
        super.init()
    }
    
    func featureValue(for featureName: String) -> MLFeatureValue? {
        guard featureName == inputName else { return nil }
        return MLFeatureValue(pixelBuffer: pixelBuffer)
    }
}

// MARK: - Batch Processing
class BatchInferenceProcessor {
    
    private let model: MLModel
    
    init(model: MLModel) {
        self.model = model
    }
    
    // MLArrayBatchProvider สำหรับ batch inference
    func processBatch(images: [CVPixelBuffer]) throws -> [MLFeatureProvider] {
        // สร้าง array ของ feature providers
        let providers = images.map { ImageFeatureProvider(pixelBuffer: $0) }
        let batchProvider = MLArrayBatchProvider(array: providers)
        
        // ทำ batch prediction
        let results = try model.predictions(fromBatch: batchProvider)
        
        // รวบรวม results
        var outputs: [MLFeatureProvider] = []
        for i in 0..<results.count {
            outputs.append(results.features(at: i))
        }
        
        return outputs
    }
    
    // ประมวลผลแบบ async batch
    func processBatchAsync(images: [CVPixelBuffer]) async throws -> [[String: Any]] {
        return try await withCheckedThrowingContinuation { continuation in
            DispatchQueue.global(qos: .userInitiated).async {
                do {
                    let providers = images.map { ImageFeatureProvider(pixelBuffer: $0) }
                    let batchProvider = MLArrayBatchProvider(array: providers)
                    let results = try self.model.predictions(fromBatch: batchProvider)
                    
                    var allResults: [[String: Any]] = []
                    for i in 0..<results.count {
                        let features = results.features(at: i)
                        var dict: [String: Any] = [:]
                        
                        for name in features.featureNames {
                            if let value = features.featureValue(for: name) {
                                dict[name] = value
                            }
                        }
                        allResults.append(dict)
                    }
                    
                    continuation.resume(returning: allResults)
                } catch {
                    continuation.resume(throwing: error)
                }
            }
        }
    }
}

// MARK: - Feature Value Extraction Helper
extension MLFeatureValue {
    func asClassificationResult() -> (String, Double)? {
        guard type == .string else { return nil }
        return (stringValue, 1.0)
    }
    
    func asTopClassifications(count: Int = 5) -> [(String, Double)] {
        guard type == .dictionary,
              let dict = dictionaryValue as? [String: Double] else { return [] }
        
        return dict.sorted { $0.value > $1.value }
            .prefix(count)
            .map { ($0.key, $0.value) }
    }
}
```

---

## 49.19 Model Metadata

```swift
import CoreML

// MARK: - การอ่าน Model Metadata
func readModelMetadata(model: MLModel) {
    let description = model.modelDescription
    
    print("=== Model Metadata ===\n")
    
    // Basic info
    if let author = description.metadata[MLModelMetadataKey.author] as? String {
        print("Author: \(author)")
    }
    if let desc = description.metadata[MLModelMetadataKey.description] as? String {
        print("Description: \(desc)")
    }
    if let version = description.metadata[MLModelMetadataKey.versionString] as? String {
        print("Version: \(version)")
    }
    if let license = description.metadata[MLModelMetadataKey.license] as? String {
        print("License: \(license)")
    }
    
    // Input/Output descriptions
    print("\nInputs:")
    for (name, featureDesc) in description.inputDescriptionsByName {
        print("  \(name):")
        print("    Type: \(featureDesc.type.rawValue)")
        
        if let imageConstraint = featureDesc.imageConstraint {
            print("    Size: \(imageConstraint.pixelsWide)x\(imageConstraint.pixelsHigh)")
            print("    Color space: \(imageConstraint.pixelFormatType)")
        }
        
        if let multiArrayConstraint = featureDesc.multiArrayConstraint {
            print("    Shape: \(multiArrayConstraint.shape)")
            print("    Data type: \(multiArrayConstraint.dataType.rawValue)")
        }
    }
    
    print("\nOutputs:")
    for (name, featureDesc) in description.outputDescriptionsByName {
        print("  \(name): \(featureDesc.type.rawValue)")
    }
    
    // Class labels (สำหรับ classifier)
    if let classLabels = description.classLabels as? [String] {
        print("\nClass Labels (\(classLabels.count) classes):")
        classLabels.prefix(10).forEach { print("  - \($0)") }
        if classLabels.count > 10 { print("  ... และอีก \(classLabels.count - 10) classes") }
    }
}

// MARK: - Model Version Management
class ModelVersionManager {
    
    struct ModelVersion: Codable {
        let version: String
        let modelName: String
        let downloadURL: String
        let checksum: String
        let minimumOSVersion: String
    }
    
    // ตรวจสอบว่าต้องอัพเดตโมเดลหรือไม่
    func checkForModelUpdate(currentVersion: String, latestVersion: ModelVersion) -> Bool {
        return currentVersion != latestVersion.version
    }
    
    // บันทึก version ปัจจุบัน
    func saveCurrentVersion(_ version: String, forModel modelName: String) {
        UserDefaults.standard.set(version, forKey: "modelVersion_\(modelName)")
    }
    
    // ดึง version ปัจจุบัน
    func getCurrentVersion(forModel modelName: String) -> String? {
        return UserDefaults.standard.string(forKey: "modelVersion_\(modelName)")
    }
    
    // ดาวน์โหลดและอัพเดตโมเดล
    func updateModel(from version: ModelVersion) async throws -> URL {
        guard let url = URL(string: version.downloadURL) else {
            throw URLError(.badURL)
        }
        
        // ดาวน์โหลดโมเดล
        let (tempURL, _) = try await URLSession.shared.download(from: url)
        
        // Compile โมเดล
        let compiledURL = try await MLModel.compileModel(at: tempURL)
        
        // ย้ายไป Documents
        let documentsURL = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first!
        let destinationURL = documentsURL.appendingPathComponent("\(version.modelName).mlmodelc")
        
        if FileManager.default.fileExists(atPath: destinationURL.path) {
            try FileManager.default.removeItem(at: destinationURL)
        }
        
        try FileManager.default.moveItem(at: compiledURL, to: destinationURL)
        
        // บันทึก version ใหม่
        saveCurrentVersion(version.version, forModel: version.modelName)
        
        return destinationURL
    }
}
```

---

## 49.20 Practical Exercises

### แบบฝึกหัดที่ 1: สร้าง Simple Image Classifier

**โจทย์:** สร้างแอปที่จำแนกรูปภาพว่าเป็น "cat" หรือ "dog"

```swift
import SwiftUI
import CoreML
import Vision

// Exercise 1: Cat vs Dog Classifier
struct CatDogClassifierView: View {
    @State private var selectedImage: UIImage?
    @State private var result: String = ""
    @State private var confidence: Float = 0
    @State private var isProcessing = false
    @State private var showPicker = false
    
    // ใช้ MobileNetV2 เป็นตัวอย่าง (ในการใช้งานจริง ใช้ custom model)
    private let classifierModel = try? VNCoreMLModel(
        for: MobileNetV2(configuration: MLModelConfiguration()).model
    )
    
    var body: some View {
        VStack(spacing: 20) {
            Text("Cat vs Dog Classifier")
                .font(.largeTitle)
                .bold()
            
            // แสดงรูปภาพ
            if let image = selectedImage {
                Image(uiImage: image)
                    .resizable()
                    .scaledToFit()
                    .frame(maxHeight: 300)
                    .cornerRadius(12)
                    .overlay(
                        isProcessing ? 
                            AnyView(ProgressView().scaleEffect(2)) : 
                            AnyView(EmptyView())
                    )
            } else {
                RoundedRectangle(cornerRadius: 12)
                    .fill(Color.gray.opacity(0.2))
                    .frame(height: 250)
                    .overlay(Text("เลือกรูปภาพ").foregroundColor(.gray))
            }
            
            // แสดงผลลัพธ์
            if !result.isEmpty {
                VStack {
                    Text(result.contains("cat") ? "🐱 แมว!" : result.contains("dog") ? "🐕 สุนัข!" : result)
                        .font(.title)
                        .bold()
                    Text("ความมั่นใจ: \(Int(confidence * 100))%")
                        .foregroundColor(.secondary)
                    
                    // Progress bar
                    GeometryReader { geo in
                        ZStack(alignment: .leading) {
                            Capsule().fill(Color.gray.opacity(0.3))
                            Capsule()
                                .fill(confidence > 0.7 ? Color.green : Color.orange)
                                .frame(width: geo.size.width * CGFloat(confidence))
                        }
                    }
                    .frame(height: 8)
                    .padding(.horizontal)
                }
                .padding()
                .background(Color(.systemGray6))
                .cornerRadius(12)
            }
            
            // ปุ่มเลือกรูป
            Button("เลือกรูปภาพ") {
                showPicker = true
            }
            .buttonStyle(.borderedProminent)
            .controlSize(.large)
        }
        .padding()
        .sheet(isPresented: $showPicker) {
            ImagePicker(image: $selectedImage, onSelect: classify)
        }
    }
    
    func classify() {
        guard let image = selectedImage,
              let model = classifierModel,
              let ciImage = CIImage(image: image) else { return }
        
        isProcessing = true
        result = ""
        
        let request = VNCoreMLRequest(model: model) { req, _ in
            DispatchQueue.main.async {
                isProcessing = false
                guard let results = req.results as? [VNClassificationObservation],
                      let top = results.first else { return }
                result = top.identifier
                confidence = top.confidence
            }
        }
        request.imageCropAndScaleOption = .centerCrop
        
        DispatchQueue.global(qos: .userInitiated).async {
            try? VNImageRequestHandler(ciImage: ciImage).perform([request])
        }
    }
}
```

### แบบฝึกหัดที่ 2: Real-Time Object Counter

```swift
// Exercise 2: นับจำนวนวัตถุใน real-time
import UIKit
import AVFoundation
import Vision
import CoreML

class ObjectCounterViewController: UIViewController {
    
    private let session = AVCaptureSession()
    private var previewLayer: AVCaptureVideoPreviewLayer!
    private let output = AVCaptureVideoDataOutput()
    private var model: VNCoreMLModel?
    
    // นับจำนวนวัตถุแต่ละประเภท
    private var objectCounts: [String: Int] = [:]
    
    // UI
    private lazy var statsView: UIView = {
        let view = UIView()
        view.backgroundColor = .black.withAlphaComponent(0.7)
        view.layer.cornerRadius = 12
        view.translatesAutoresizingMaskIntoConstraints = false
        return view
    }()
    
    private lazy var statsLabel: UILabel = {
        let label = UILabel()
        label.textColor = .white
        label.font = .monospacedSystemFont(ofSize: 14, weight: .regular)
        label.numberOfLines = 0
        label.translatesAutoresizingMaskIntoConstraints = false
        return label
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupCamera()
        setupModel()
        setupStatsView()
    }
    
    private func setupCamera() {
        session.sessionPreset = .vga640x480 // ใช้ resolution ต่ำกว่าเพื่อประหยัด resource
        
        guard let camera = AVCaptureDevice.default(for: .video),
              let input = try? AVCaptureDeviceInput(device: camera) else { return }
        
        session.addInput(input)
        
        previewLayer = AVCaptureVideoPreviewLayer(session: session)
        previewLayer.frame = view.bounds
        previewLayer.videoGravity = .resizeAspectFill
        view.layer.addSublayer(previewLayer)
        
        output.setSampleBufferDelegate(self, queue: DispatchQueue(label: "processing"))
        output.alwaysDiscardsLateVideoFrames = true
        session.addOutput(output)
        
        DispatchQueue.global(qos: .background).async { [weak self] in
            self?.session.startRunning()
        }
    }
    
    private func setupModel() {
        guard let mlModel = try? YOLOv3Tiny(configuration: MLModelConfiguration()).model else { return }
        model = try? VNCoreMLModel(for: mlModel)
    }
    
    private func setupStatsView() {
        view.addSubview(statsView)
        statsView.addSubview(statsLabel)
        
        NSLayoutConstraint.activate([
            statsView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 16),
            statsView.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -16),
            statsView.widthAnchor.constraint(equalToConstant: 180),
            
            statsLabel.topAnchor.constraint(equalTo: statsView.topAnchor, constant: 8),
            statsLabel.leadingAnchor.constraint(equalTo: statsView.leadingAnchor, constant: 8),
            statsLabel.trailingAnchor.constraint(equalTo: statsView.trailingAnchor, constant: -8),
            statsLabel.bottomAnchor.constraint(equalTo: statsView.bottomAnchor, constant: -8)
        ])
    }
    
    private func updateStats(with observations: [VNRecognizedObjectObservation]) {
        var counts: [String: Int] = [:]
        
        for obs in observations {
            guard let label = obs.labels.first?.identifier,
                  (obs.labels.first?.confidence ?? 0) > 0.5 else { continue }
            counts[label, default: 0] += 1
        }
        
        objectCounts = counts
        
        DispatchQueue.main.async { [weak self] in
            guard let self = self else { return }
            
            if self.objectCounts.isEmpty {
                self.statsLabel.text = "ไม่พบวัตถุ"
            } else {
                self.statsLabel.text = self.objectCounts
                    .sorted { $0.value > $1.value }
                    .prefix(8)
                    .map { "\($0.key): \($0.value)" }
                    .joined(separator: "\n")
            }
        }
    }
}

extension ObjectCounterViewController: AVCaptureVideoDataOutputSampleBufferDelegate {
    func captureOutput(_ output: AVCaptureOutput, didOutput sampleBuffer: CMSampleBuffer, from connection: AVCaptureConnection) {
        guard let pixelBuffer = CMSampleBufferGetImageBuffer(sampleBuffer),
              let model = model else { return }
        
        let request = VNCoreMLRequest(model: model) { [weak self] req, _ in
            guard let results = req.results as? [VNRecognizedObjectObservation] else { return }
            self?.updateStats(with: results)
        }
        request.imageCropAndScaleOption = .scaleFill
        
        try? VNImageRequestHandler(cvPixelBuffer: pixelBuffer).perform([request])
    }
}
```

---

## 49.21 Building an Image Classifier App (Complete)

โปรเจกต์สมบูรณ์สำหรับ Image Classifier App ด้วย SwiftUI

```swift
// MARK: - Complete Image Classifier App
// File: ImageClassifierApp.swift

import SwiftUI

@main
struct ImageClassifierApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}

// MARK: - ContentView
struct ContentView: View {
    var body: some View {
        TabView {
            ClassifyView()
                .tabItem {
                    Label("จำแนก", systemImage: "photo.on.rectangle")
                }
            
            HistoryView()
                .tabItem {
                    Label("ประวัติ", systemImage: "clock")
                }
            
            SettingsView()
                .tabItem {
                    Label("ตั้งค่า", systemImage: "gear")
                }
        }
    }
}

// MARK: - Classification Result Model
struct ClassificationRecord: Identifiable, Codable {
    let id: UUID
    let timestamp: Date
    let topLabel: String
    let confidence: Float
    let topResults: [(String, Float)]
    
    enum CodingKeys: String, CodingKey {
        case id, timestamp, topLabel, confidence
    }
    
    init(id: UUID = UUID(), timestamp: Date = Date(), 
         topLabel: String, confidence: Float, topResults: [(String, Float)]) {
        self.id = id
        self.timestamp = timestamp
        self.topLabel = topLabel
        self.confidence = confidence
        self.topResults = topResults
    }
    
    var formattedDate: String {
        let formatter = DateFormatter()
        formatter.dateStyle = .medium
        formatter.timeStyle = .short
        formatter.locale = Locale(identifier: "th_TH")
        return formatter.string(from: timestamp)
    }
}

// MARK: - App State
@MainActor
class AppState: ObservableObject {
    @Published var classificationHistory: [ClassificationRecord] = []
    @Published var selectedModel: String = "MobileNetV2"
    
    func addRecord(_ record: ClassificationRecord) {
        classificationHistory.insert(record, at: 0)
        if classificationHistory.count > 50 {
            classificationHistory.removeLast()
        }
    }
}

// MARK: - Classify View
struct ClassifyView: View {
    @EnvironmentObject var appState: AppState
    @StateObject private var viewModel = ClassifyViewModel()
    @State private var showingPicker = false
    @State private var showingCamera = false
    
    var body: some View {
        NavigationView {
            ScrollView {
                VStack(spacing: 24) {
                    // Image Section
                    imageSection
                    
                    // Action Buttons
                    actionButtons
                    
                    // Results Section
                    if !viewModel.results.isEmpty {
                        resultsSection
                    }
                    
                    Spacer(minLength: 40)
                }
                .padding()
            }
            .navigationTitle("Image Classifier")
            .sheet(isPresented: $showingPicker) {
                ImagePicker(image: $viewModel.selectedImage, onSelect: viewModel.classify)
            }
        }
    }
    
    var imageSection: some View {
        ZStack {
            RoundedRectangle(cornerRadius: 16)
                .fill(Color(.systemGray6))
                .frame(height: 280)
            
            if let image = viewModel.selectedImage {
                Image(uiImage: image)
                    .resizable()
                    .scaledToFit()
                    .frame(maxHeight: 280)
                    .cornerRadius(16)
            } else {
                VStack(spacing: 12) {
                    Image(systemName: "photo.badge.plus")
                        .font(.system(size: 60))
                        .foregroundColor(.blue)
                    Text("แตะปุ่มด้านล่างเพื่อเลือกรูปภาพ")
                        .foregroundColor(.secondary)
                }
            }
            
            if viewModel.isProcessing {
                RoundedRectangle(cornerRadius: 16)
                    .fill(Color.black.opacity(0.5))
                VStack(spacing: 12) {
                    ProgressView()
                        .progressViewStyle(CircularProgressViewStyle(tint: .white))
                        .scaleEffect(1.5)
                    Text("กำลังวิเคราะห์...")
                        .foregroundColor(.white)
                        .font(.subheadline)
                }
            }
        }
    }
    
    var actionButtons: some View {
        HStack(spacing: 16) {
            Button {
                showingPicker = true
            } label: {
                Label("Photo Library", systemImage: "photo.on.rectangle")
                    .frame(maxWidth: .infinity)
            }
            .buttonStyle(.borderedProminent)
            .controlSize(.large)
            
            Button {
                showingCamera = true
            } label: {
                Label("กล้อง", systemImage: "camera")
                    .frame(maxWidth: .infinity)
            }
            .buttonStyle(.bordered)
            .controlSize(.large)
        }
    }
    
    var resultsSection: some View {
        VStack(alignment: .leading, spacing: 16) {
            Text("ผลการวิเคราะห์")
                .font(.headline)
            
            // Top Result
            HStack {
                VStack(alignment: .leading) {
                    Text(viewModel.topResult?.0 ?? "")
                        .font(.title2)
                        .bold()
                    Text("ความมั่นใจ: \(Int((viewModel.topResult?.1 ?? 0) * 100))%")
                        .foregroundColor(.secondary)
                }
                Spacer()
                CircularProgressView(progress: CGFloat(viewModel.topResult?.1 ?? 0))
                    .frame(width: 60, height: 60)
            }
            .padding()
            .background(Color(.systemGray6))
            .cornerRadius(12)
            
            // All Results
            VStack(spacing: 8) {
                ForEach(viewModel.results.prefix(5), id: \.0) { label, confidence in
                    HStack {
                        Text(label)
                            .font(.subheadline)
                            .lineLimit(1)
                        Spacer()
                        GeometryReader { geo in
                            ZStack(alignment: .leading) {
                                Capsule().fill(Color(.systemGray5))
                                Capsule()
                                    .fill(Color.blue)
                                    .frame(width: geo.size.width * CGFloat(confidence))
                            }
                        }
                        .frame(width: 100, height: 6)
                        Text("\(Int(confidence * 100))%")
                            .font(.caption)
                            .frame(width: 40, alignment: .trailing)
                    }
                }
            }
        }
        .padding()
        .background(Color(.systemBackground))
        .cornerRadius(16)
        .shadow(color: .black.opacity(0.05), radius: 8)
    }
}

// MARK: - Circular Progress View
struct CircularProgressView: View {
    let progress: CGFloat
    
    var body: some View {
        ZStack {
            Circle()
                .stroke(Color(.systemGray5), lineWidth: 6)
            Circle()
                .trim(from: 0, to: progress)
                .stroke(
                    progress > 0.7 ? Color.green : progress > 0.4 ? Color.orange : Color.red,
                    style: StrokeStyle(lineWidth: 6, lineCap: .round)
                )
                .rotationEffect(.degrees(-90))
                .animation(.easeInOut, value: progress)
            Text("\(Int(progress * 100))%")
                .font(.caption2)
                .bold()
        }
    }
}

// MARK: - ClassifyViewModel
@MainActor
class ClassifyViewModel: ObservableObject {
    @Published var selectedImage: UIImage?
    @Published var results: [(String, Float)] = []
    @Published var isProcessing = false
    
    var topResult: (String, Float)? { results.first }
    
    private var visionModel: VNCoreMLModel?
    
    init() {
        Task { await loadModel() }
    }
    
    func loadModel() async {
        do {
            let config = MLModelConfiguration()
            config.computeUnits = .all
            let mlModel = try MobileNetV2(configuration: config).model
            visionModel = try VNCoreMLModel(for: mlModel)
        } catch {
            print("โหลดโมเดลล้มเหลว: \(error)")
        }
    }
    
    func classify() {
        guard let image = selectedImage,
              let model = visionModel,
              let ciImage = CIImage(image: image) else { return }
        
        isProcessing = true
        results = []
        
        let request = VNCoreMLRequest(model: model) { [weak self] req, _ in
            guard let observations = req.results as? [VNClassificationObservation] else { return }
            Task { @MainActor in
                self?.isProcessing = false
                self?.results = observations.prefix(5).map { ($0.identifier, $0.confidence) }
            }
        }
        request.imageCropAndScaleOption = .centerCrop
        
        Task.detached(priority: .userInitiated) {
            try? VNImageRequestHandler(ciImage: ciImage).perform([request])
        }
    }
}

// MARK: - History View
struct HistoryView: View {
    @EnvironmentObject var appState: AppState
    
    var body: some View {
        NavigationView {
            Group {
                if appState.classificationHistory.isEmpty {
                    VStack(spacing: 16) {
                        Image(systemName: "clock.badge.xmark")
                            .font(.system(size: 60))
                            .foregroundColor(.secondary)
                        Text("ยังไม่มีประวัติการจำแนก")
                            .foregroundColor(.secondary)
                    }
                } else {
                    List(appState.classificationHistory) { record in
                        VStack(alignment: .leading, spacing: 4) {
                            Text(record.topLabel)
                                .font(.headline)
                            HStack {
                                Text("ความมั่นใจ: \(Int(record.confidence * 100))%")
                                Spacer()
                                Text(record.formattedDate)
                            }
                            .font(.caption)
                            .foregroundColor(.secondary)
                        }
                        .padding(.vertical, 4)
                    }
                }
            }
            .navigationTitle("ประวัติ")
        }
    }
}

// MARK: - Settings View
struct SettingsView: View {
    @EnvironmentObject var appState: AppState
    
    var body: some View {
        NavigationView {
            Form {
                Section("โมเดล") {
                    Picker("โมเดลที่ใช้", selection: $appState.selectedModel) {
                        Text("MobileNetV2").tag("MobileNetV2")
                        Text("SqueezeNet").tag("SqueezeNet")
                        Text("Resnet50").tag("Resnet50")
                    }
                }
                
                Section("เกี่ยวกับ") {
                    LabeledContent("เวอร์ชัน", value: "1.0.0")
                    LabeledContent("Core ML Version", value: "7.0")
                }
            }
            .navigationTitle("ตั้งค่า")
        }
    }
}
```

---

## 49.22 สรุป

ในบทนี้เราได้เรียนรู้เกี่ยวกับ **Core ML** และ **Machine Learning** ใน iOS อย่างครอบคลุม:

### สิ่งที่ได้เรียนรู้

| หัวข้อ | ประเด็นสำคัญ |
|--------|-------------|
| **Core ML** | Framework สำหรับ on-device ML inference |
| **Create ML** | เครื่องมือสร้างและเทรนโมเดลบน macOS |
| **Model Types** | Image Classification, Object Detection, Text Classification, Tabular |
| **VNCoreMLModel** | Wrapper สำหรับใช้ MLModel กับ Vision |
| **VNCoreMLRequest** | Request สำหรับประมวลผลภาพด้วย Core ML |
| **Vision Pipeline** | การเชื่อมต่อ Vision กับ Core ML |
| **YOLO Integration** | Object detection แบบ real-time |
| **Camera Inference** | ประมวลผล live camera feed |
| **Quantization** | การลดขนาดโมเดลเพื่อประสิทธิภาพดีขึ้น |
| **SwiftUI Integration** | การใช้ Core ML กับ SwiftUI |

### Best Practices

```swift
// 1. โหลดโมเดลแบบ async
Task {
    let model = try await MLModel.load(contentsOf: url, configuration: config)
}

// 2. ประมวลผลบน background thread
DispatchQueue.global(qos: .userInitiated).async {
    try? handler.perform([request])
}

// 3. ใช้ computeUnits = .all เพื่อประสิทธิภาพสูงสุด
config.computeUnits = .all

// 4. Throttle camera processing
guard now.timeIntervalSince(lastProcessed) >= 0.1 else { return }

// 5. Handle errors อย่างเหมาะสม
guard let model = try? VNCoreMLModel(for: mlModel) else {
    // แสดง fallback UI
    return
}
```

### ขั้นตอนต่อไป

1. ทดลองใช้ **Create ML** บน Mac เพื่อสร้าง custom model
2. ดาวน์โหลดโมเดลจาก [Apple Developer](https://developer.apple.com/machine-learning/models/)
3. ลองใช้ **CoreML Tools** (Python) เพื่อ convert และ optimize โมเดล
4. ศึกษา **Vision framework** ในบทถัดไป

---

*ศึกษาเพิ่มเติม:*
- [Core ML Documentation](https://developer.apple.com/documentation/coreml)
- [Create ML Documentation](https://developer.apple.com/documentation/createml)
- [WWDC: Advances in Core ML](https://developer.apple.com/videos/machine-learning)
