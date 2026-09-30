# Part 42: Camera และ Photos

## บทนำ

ในบทนี้เราจะเรียนรู้เกี่ยวกับการใช้งานกล้องและรูปภาพใน iOS โดยครอบคลุมตั้งแต่วิธีดั้งเดิมอย่าง UIImagePickerController ไปจนถึง API สมัยใหม่อย่าง PHPickerViewController และ AVFoundation สำหรับการควบคุมกล้องเต็มรูปแบบ

---

## 1. UIImagePickerController (Legacy)

### 1.1 ความรู้เบื้องต้น

`UIImagePickerController` เป็นวิธีดั้งเดิมที่ใช้เลือกภาพหรือถ่ายภาพ แม้ว่าจะยังใช้งานได้ แต่ Apple แนะนำให้ใช้ `PHPickerViewController` แทนสำหรับ app ใหม่:

```swift
import UIKit

class LegacyImagePickerViewController: UIViewController {
    var imageView: UIImageView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
    }
    
    func setupUI() {
        imageView = UIImageView(frame: CGRect(x: 20, y: 100, width: 300, height: 300))
        imageView.contentMode = .scaleAspectFit
        imageView.backgroundColor = .systemGray6
        imageView.layer.cornerRadius = 10
        view.addSubview(imageView)
        
        let cameraButton = UIButton(type: .system)
        cameraButton.setTitle("ถ่ายภาพ", for: .normal)
        cameraButton.frame = CGRect(x: 20, y: 420, width: 140, height: 44)
        cameraButton.backgroundColor = .systemBlue
        cameraButton.setTitleColor(.white, for: .normal)
        cameraButton.layer.cornerRadius = 10
        cameraButton.addTarget(self, action: #selector(openCamera), for: .touchUpInside)
        view.addSubview(cameraButton)
        
        let galleryButton = UIButton(type: .system)
        galleryButton.setTitle("เลือกจากคลัง", for: .normal)
        galleryButton.frame = CGRect(x: 180, y: 420, width: 160, height: 44)
        galleryButton.backgroundColor = .systemGreen
        galleryButton.setTitleColor(.white, for: .normal)
        galleryButton.layer.cornerRadius = 10
        galleryButton.addTarget(self, action: #selector(openGallery), for: .touchUpInside)
        view.addSubview(galleryButton)
    }
    
    @objc func openCamera() {
        guard UIImagePickerController.isSourceTypeAvailable(.camera) else {
            print("ไม่มีกล้อง")
            return
        }
        
        let picker = UIImagePickerController()
        picker.sourceType = .camera
        picker.delegate = self
        picker.allowsEditing = true
        picker.cameraDevice = .rear // หรือ .front
        picker.cameraCaptureMode = .photo
        
        present(picker, animated: true)
    }
    
    @objc func openGallery() {
        let picker = UIImagePickerController()
        picker.sourceType = .photoLibrary
        picker.delegate = self
        picker.allowsEditing = false
        picker.mediaTypes = ["public.image"] // รองรับเฉพาะรูปภาพ
        
        present(picker, animated: true)
    }
}

extension LegacyImagePickerViewController: UIImagePickerControllerDelegate,
                                            UINavigationControllerDelegate {
    func imagePickerController(
        _ picker: UIImagePickerController,
        didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey: Any]
    ) {
        picker.dismiss(animated: true)
        
        // รับภาพที่แก้ไขแล้ว (ถ้ามี) หรือภาพต้นฉบับ
        if let editedImage = info[.editedImage] as? UIImage {
            imageView.image = editedImage
        } else if let originalImage = info[.originalImage] as? UIImage {
            imageView.image = originalImage
        }
        
        // รับ URL ของไฟล์
        if let imageURL = info[.imageURL] as? URL {
            print("ไฟล์อยู่ที่: \(imageURL)")
        }
        
        // รับ metadata
        if let metadata = info[.mediaMetadata] as? [String: Any] {
            print("Metadata: \(metadata)")
        }
    }
    
    func imagePickerControllerDidCancel(_ picker: UIImagePickerController) {
        picker.dismiss(animated: true)
        print("ผู้ใช้ยกเลิก")
    }
}
```

---

## 2. PHPickerViewController (Modern)

### 2.1 การใช้งาน PHPickerViewController

`PHPickerViewController` เป็น API ใหม่ที่ Apple แนะนำตั้งแต่ iOS 14 ซึ่งให้ความเป็นส่วนตัวมากกว่า:

```swift
import UIKit
import PhotosUI

class ModernPhotoPickerViewController: UIViewController {
    var imageView: UIImageView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
    }
    
    func setupUI() {
        imageView = UIImageView(frame: CGRect(x: 20, y: 100, width: 350, height: 350))
        imageView.contentMode = .scaleAspectFit
        imageView.backgroundColor = .systemGray6
        imageView.layer.cornerRadius = 12
        imageView.clipsToBounds = true
        view.addSubview(imageView)
        
        let selectButton = UIButton(type: .system)
        selectButton.setTitle("เลือกภาพ", for: .normal)
        selectButton.frame = CGRect(x: 20, y: 470, width: 350, height: 50)
        selectButton.backgroundColor = .systemBlue
        selectButton.setTitleColor(.white, for: .normal)
        selectButton.titleLabel?.font = .systemFont(ofSize: 17, weight: .semibold)
        selectButton.layer.cornerRadius = 12
        selectButton.addTarget(self, action: #selector(openPicker), for: .touchUpInside)
        view.addSubview(selectButton)
    }
    
    @objc func openPicker() {
        var config = PHPickerConfiguration()
        config.selectionLimit = 1  // 0 = ไม่จำกัด
        config.filter = .images    // เฉพาะรูปภาพ
        
        let picker = PHPickerViewController(configuration: config)
        picker.delegate = self
        present(picker, animated: true)
    }
    
    // เปิด picker สำหรับเลือกหลายภาพ
    @objc func openMultiPicker() {
        var config = PHPickerConfiguration(photoLibrary: .shared())
        config.selectionLimit = 5
        config.filter = .any(of: [.images, .livePhotos])
        config.preferredAssetRepresentationMode = .current
        
        let picker = PHPickerViewController(configuration: config)
        picker.delegate = self
        present(picker, animated: true)
    }
    
    // เปิด picker สำหรับวิดีโอ
    @objc func openVideoPicker() {
        var config = PHPickerConfiguration()
        config.selectionLimit = 1
        config.filter = .videos
        
        let picker = PHPickerViewController(configuration: config)
        picker.delegate = self
        present(picker, animated: true)
    }
}

extension ModernPhotoPickerViewController: PHPickerViewControllerDelegate {
    func picker(_ picker: PHPickerViewController, didFinishPicking results: [PHPickerResult]) {
        picker.dismiss(animated: true)
        
        guard let result = results.first else { return }
        
        // โหลดภาพแบบ async
        if result.itemProvider.canLoadObject(ofClass: UIImage.self) {
            result.itemProvider.loadObject(ofClass: UIImage.self) { [weak self] object, error in
                if let error = error {
                    print("เกิดข้อผิดพลาด: \(error)")
                    return
                }
                
                guard let image = object as? UIImage else { return }
                
                DispatchQueue.main.async {
                    self?.imageView.image = image
                }
            }
        }
        
        // โหลดหลายภาพ
        processMultipleResults(results)
    }
    
    private func processMultipleResults(_ results: [PHPickerResult]) {
        var images: [UIImage] = []
        let group = DispatchGroup()
        
        for result in results {
            group.enter()
            
            if result.itemProvider.canLoadObject(ofClass: UIImage.self) {
                result.itemProvider.loadObject(ofClass: UIImage.self) { object, error in
                    defer { group.leave() }
                    
                    if let image = object as? UIImage {
                        images.append(image)
                    }
                }
            } else {
                group.leave()
            }
        }
        
        group.notify(queue: .main) {
            print("โหลดภาพทั้งหมด \(images.count) ภาพ")
        }
    }
}
```

### 2.2 PHPickerConfiguration ขั้นสูง

```swift
class AdvancedPickerManager {
    func createLivePhotoConfig() -> PHPickerConfiguration {
        var config = PHPickerConfiguration(photoLibrary: .shared())
        config.selectionLimit = 3
        config.filter = .livePhotos
        config.preferredAssetRepresentationMode = .current
        return config
    }
    
    func createCinematicVideoConfig() -> PHPickerConfiguration {
        var config = PHPickerConfiguration()
        config.filter = .cinematicVideos
        return config
    }
    
    func processPickerResult(_ result: PHPickerResult) async throws -> PHPickerContent {
        // โหลด data representation
        let itemProvider = result.itemProvider
        
        if itemProvider.hasItemConformingToTypeIdentifier("public.jpeg") {
            let data = try await loadData(from: itemProvider, type: "public.jpeg")
            return .imageData(data)
        }
        
        if itemProvider.canLoadObject(ofClass: UIImage.self) {
            let image = try await loadObject(UIImage.self, from: itemProvider)
            return .image(image)
        }
        
        throw PickerError.unsupportedType
    }
    
    private func loadData(from provider: NSItemProvider, type: String) async throws -> Data {
        try await withCheckedThrowingContinuation { continuation in
            provider.loadDataRepresentation(forTypeIdentifier: type) { data, error in
                if let error = error {
                    continuation.resume(throwing: error)
                } else if let data = data {
                    continuation.resume(returning: data)
                } else {
                    continuation.resume(throwing: PickerError.noData)
                }
            }
        }
    }
    
    private func loadObject<T: NSItemProviderReading>(
        _ type: T.Type,
        from provider: NSItemProvider
    ) async throws -> T {
        try await withCheckedThrowingContinuation { continuation in
            provider.loadObject(ofClass: type) { object, error in
                if let error = error {
                    continuation.resume(throwing: error)
                } else if let result = object as? T {
                    continuation.resume(returning: result)
                } else {
                    continuation.resume(throwing: PickerError.invalidObject)
                }
            }
        }
    }
}

enum PHPickerContent {
    case image(UIImage)
    case imageData(Data)
    case video(URL)
}

enum PickerError: Error {
    case unsupportedType
    case noData
    case invalidObject
}
```

---

## 3. AVFoundation สำหรับกล้อง

### 3.1 AVCaptureSession

`AVCaptureSession` เป็น object หลักที่ประสานงานการรับ input และส่งออก output:

```swift
import AVFoundation
import UIKit

class CameraSessionManager: NSObject {
    var captureSession: AVCaptureSession!
    var videoPreviewLayer: AVCaptureVideoPreviewLayer!
    var photoOutput: AVCapturePhotoOutput!
    var videoOutput: AVCaptureMovieFileOutput!
    
    func setupSession() {
        captureSession = AVCaptureSession()
        
        // กำหนด quality preset
        captureSession.sessionPreset = .photo
        
        setupInputs()
        setupOutputs()
    }
    
    private func setupInputs() {
        // Camera input
        guard let camera = AVCaptureDevice.default(
            .builtInWideAngleCamera,
            for: .video,
            position: .back
        ) else {
            print("ไม่พบกล้อง")
            return
        }
        
        do {
            let cameraInput = try AVCaptureDeviceInput(device: camera)
            if captureSession.canAddInput(cameraInput) {
                captureSession.addInput(cameraInput)
            }
        } catch {
            print("เกิดข้อผิดพลาดกับ camera input: \(error)")
        }
        
        // Microphone input (สำหรับวิดีโอ)
        if let microphone = AVCaptureDevice.default(for: .audio) {
            do {
                let micInput = try AVCaptureDeviceInput(device: microphone)
                if captureSession.canAddInput(micInput) {
                    captureSession.addInput(micInput)
                }
            } catch {
                print("เกิดข้อผิดพลาดกับ microphone: \(error)")
            }
        }
    }
    
    private func setupOutputs() {
        // Photo output
        photoOutput = AVCapturePhotoOutput()
        if captureSession.canAddOutput(photoOutput) {
            captureSession.addOutput(photoOutput)
        }
        
        // Video output
        videoOutput = AVCaptureMovieFileOutput()
        if captureSession.canAddOutput(videoOutput) {
            captureSession.addOutput(videoOutput)
        }
    }
    
    func startSession() {
        DispatchQueue.global(qos: .userInitiated).async { [weak self] in
            self?.captureSession.startRunning()
        }
    }
    
    func stopSession() {
        captureSession.stopRunning()
    }
}
```

### 3.2 AVCaptureDevice

```swift
class CameraDeviceManager {
    // รับ camera device ที่ต้องการ
    func getCamera(position: AVCaptureDevice.Position) -> AVCaptureDevice? {
        // ลอง built-in triple camera ก่อน (iPhone Pro)
        if let device = AVCaptureDevice.default(
            .builtInTripleCamera,
            for: .video,
            position: position
        ) {
            return device
        }
        
        // ลอง dual camera
        if let device = AVCaptureDevice.default(
            .builtInDualCamera,
            for: .video,
            position: position
        ) {
            return device
        }
        
        // fallback ไปยัง wide angle
        return AVCaptureDevice.default(
            .builtInWideAngleCamera,
            for: .video,
            position: position
        )
    }
    
    // ปรับการตั้งค่ากล้อง
    func configureCameraDevice(_ device: AVCaptureDevice) {
        do {
            try device.lockForConfiguration()
            
            // Auto focus
            if device.isFocusModeSupported(.continuousAutoFocus) {
                device.focusMode = .continuousAutoFocus
            }
            
            // Auto exposure
            if device.isExposureModeSupported(.continuousAutoExposure) {
                device.exposureMode = .continuousAutoExposure
            }
            
            // Auto white balance
            if device.isWhiteBalanceModeSupported(.continuousAutoWhiteBalance) {
                device.whiteBalanceMode = .continuousAutoWhiteBalance
            }
            
            // Low light boost (iPhone specific)
            if device.isLowLightBoostSupported {
                device.automaticallyEnablesLowLightBoostWhenAvailable = true
            }
            
            device.unlockForConfiguration()
        } catch {
            print("เกิดข้อผิดพลาดในการตั้งค่ากล้อง: \(error)")
        }
    }
    
    // ปรับ zoom
    func setZoom(_ factor: CGFloat, on device: AVCaptureDevice) {
        do {
            try device.lockForConfiguration()
            
            let minZoom = device.minAvailableVideoZoomFactor
            let maxZoom = device.maxAvailableVideoZoomFactor
            let clampedFactor = min(max(factor, minZoom), maxZoom)
            
            device.videoZoomFactor = clampedFactor
            device.unlockForConfiguration()
        } catch {
            print("เกิดข้อผิดพลาดในการ zoom: \(error)")
        }
    }
    
    // ตั้งค่า focus ที่จุดที่ต้องการ
    func setFocusPoint(_ point: CGPoint, on device: AVCaptureDevice) {
        do {
            try device.lockForConfiguration()
            
            if device.isFocusPointOfInterestSupported {
                device.focusPointOfInterest = point
                device.focusMode = .autoFocus
            }
            
            if device.isExposurePointOfInterestSupported {
                device.exposurePointOfInterest = point
                device.exposureMode = .autoExpose
            }
            
            device.unlockForConfiguration()
        } catch {
            print("เกิดข้อผิดพลาดในการตั้งค่า focus: \(error)")
        }
    }
    
    // เปิด/ปิด torch
    func toggleTorch(on device: AVCaptureDevice, level: Float = 1.0) {
        guard device.hasTorch else { return }
        
        do {
            try device.lockForConfiguration()
            
            if device.torchMode == .off {
                try device.setTorchModeOn(level: level)
            } else {
                device.torchMode = .off
            }
            
            device.unlockForConfiguration()
        } catch {
            print("เกิดข้อผิดพลาดกับ torch: \(error)")
        }
    }
}
```

### 3.3 AVCaptureInput และ AVCaptureOutput

```swift
class CaptureOutputManager: NSObject {
    var captureSession: AVCaptureSession!
    var currentCameraInput: AVCaptureDeviceInput?
    var photoOutput: AVCapturePhotoOutput!
    
    // สลับกล้องหน้า/หลัง
    func switchCamera() {
        guard let currentInput = currentCameraInput else { return }
        
        let currentPosition = currentInput.device.position
        let newPosition: AVCaptureDevice.Position = currentPosition == .back ? .front : .back
        
        guard let newCamera = AVCaptureDevice.default(
            .builtInWideAngleCamera,
            for: .video,
            position: newPosition
        ) else { return }
        
        captureSession.beginConfiguration()
        
        captureSession.removeInput(currentInput)
        
        do {
            let newInput = try AVCaptureDeviceInput(device: newCamera)
            if captureSession.canAddInput(newInput) {
                captureSession.addInput(newInput)
                currentCameraInput = newInput
            }
        } catch {
            print("เกิดข้อผิดพลาด: \(error)")
        }
        
        captureSession.commitConfiguration()
    }
    
    // เพิ่ม video data output สำหรับ real-time processing
    func addVideoDataOutput(to session: AVCaptureSession) {
        let videoDataOutput = AVCaptureVideoDataOutput()
        videoDataOutput.setSampleBufferDelegate(self, queue: DispatchQueue(label: "videoQueue"))
        videoDataOutput.alwaysDiscardsLateVideoFrames = true
        
        if session.canAddOutput(videoDataOutput) {
            session.addOutput(videoDataOutput)
        }
    }
}

extension CaptureOutputManager: AVCaptureVideoDataOutputSampleBufferDelegate {
    // รับ frame แต่ละเฟรม
    func captureOutput(_ output: AVCaptureOutput,
                      didOutput sampleBuffer: CMSampleBuffer,
                      from connection: AVCaptureConnection) {
        // ประมวลผล frame ที่นี่
        guard let pixelBuffer = CMSampleBufferGetImageBuffer(sampleBuffer) else { return }
        
        let ciImage = CIImage(cvPixelBuffer: pixelBuffer)
        // ทำ image processing
    }
}
```

---

## 4. Custom Camera UI

### 4.1 การสร้าง Custom Camera Controller

```swift
import AVFoundation
import UIKit

class CustomCameraViewController: UIViewController {
    // MARK: - Properties
    private var captureSession = AVCaptureSession()
    private var previewLayer: AVCaptureVideoPreviewLayer!
    private var photoOutput = AVCapturePhotoOutput()
    private var currentDevice: AVCaptureDevice?
    private var isCapturing = false
    
    // MARK: - UI Components
    private lazy var previewView: UIView = {
        let view = UIView()
        view.backgroundColor = .black
        view.translatesAutoresizingMaskIntoConstraints = false
        return view
    }()
    
    private lazy var captureButton: UIButton = {
        let button = UIButton(type: .custom)
        button.backgroundColor = .white
        button.layer.cornerRadius = 35
        button.layer.borderWidth = 4
        button.layer.borderColor = UIColor.white.cgColor
        button.translatesAutoresizingMaskIntoConstraints = false
        button.addTarget(self, action: #selector(capturePhoto), for: .touchUpInside)
        return button
    }()
    
    private lazy var switchCameraButton: UIButton = {
        let button = UIButton(type: .system)
        button.setImage(UIImage(systemName: "camera.rotate.fill"), for: .normal)
        button.tintColor = .white
        button.translatesAutoresizingMaskIntoConstraints = false
        button.addTarget(self, action: #selector(switchCamera), for: .touchUpInside)
        return button
    }()
    
    private lazy var flashButton: UIButton = {
        let button = UIButton(type: .system)
        button.setImage(UIImage(systemName: "bolt.slash.fill"), for: .normal)
        button.tintColor = .white
        button.translatesAutoresizingMaskIntoConstraints = false
        button.addTarget(self, action: #selector(toggleFlash), for: .touchUpInside)
        return button
    }()
    
    private lazy var zoomSlider: UISlider = {
        let slider = UISlider()
        slider.minimumValue = 1.0
        slider.maximumValue = 10.0
        slider.value = 1.0
        slider.translatesAutoresizingMaskIntoConstraints = false
        slider.addTarget(self, action: #selector(zoomChanged), for: .valueChanged)
        return slider
    }()
    
    private var isFlashEnabled = false
    
    // MARK: - Lifecycle
    override func viewDidLoad() {
        super.viewDidLoad()
        checkPermissions()
        setupUI()
    }
    
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        if !captureSession.isRunning {
            DispatchQueue.global(qos: .userInitiated).async { [weak self] in
                self?.captureSession.startRunning()
            }
        }
    }
    
    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        if captureSession.isRunning {
            captureSession.stopRunning()
        }
    }
    
    // MARK: - Setup
    private func checkPermissions() {
        switch AVCaptureDevice.authorizationStatus(for: .video) {
        case .authorized:
            setupCaptureSession()
        case .notDetermined:
            AVCaptureDevice.requestAccess(for: .video) { [weak self] granted in
                if granted {
                    DispatchQueue.main.async {
                        self?.setupCaptureSession()
                    }
                }
            }
        case .denied, .restricted:
            showPermissionAlert()
        @unknown default:
            break
        }
    }
    
    private func setupUI() {
        view.backgroundColor = .black
        
        view.addSubview(previewView)
        view.addSubview(captureButton)
        view.addSubview(switchCameraButton)
        view.addSubview(flashButton)
        view.addSubview(zoomSlider)
        
        NSLayoutConstraint.activate([
            previewView.topAnchor.constraint(equalTo: view.topAnchor),
            previewView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            previewView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            previewView.bottomAnchor.constraint(equalTo: view.bottomAnchor),
            
            captureButton.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            captureButton.bottomAnchor.constraint(equalTo: view.safeAreaLayoutGuide.bottomAnchor, constant: -30),
            captureButton.widthAnchor.constraint(equalToConstant: 70),
            captureButton.heightAnchor.constraint(equalToConstant: 70),
            
            switchCameraButton.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -20),
            switchCameraButton.centerYAnchor.constraint(equalTo: captureButton.centerYAnchor),
            switchCameraButton.widthAnchor.constraint(equalToConstant: 44),
            switchCameraButton.heightAnchor.constraint(equalToConstant: 44),
            
            flashButton.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 20),
            flashButton.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 20),
            
            zoomSlider.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 40),
            zoomSlider.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -40),
            zoomSlider.bottomAnchor.constraint(equalTo: captureButton.topAnchor, constant: -30)
        ])
        
        // เพิ่ม pinch gesture สำหรับ zoom
        let pinchGesture = UIPinchGestureRecognizer(target: self, action: #selector(handlePinch(_:)))
        view.addGestureRecognizer(pinchGesture)
        
        // Tap to focus
        let tapGesture = UITapGestureRecognizer(target: self, action: #selector(handleTap(_:)))
        previewView.addGestureRecognizer(tapGesture)
    }
    
    private func setupCaptureSession() {
        captureSession.beginConfiguration()
        captureSession.sessionPreset = .photo
        
        // Camera input
        guard let camera = AVCaptureDevice.default(.builtInWideAngleCamera, for: .video, position: .back) else {
            return
        }
        
        currentDevice = camera
        
        do {
            let input = try AVCaptureDeviceInput(device: camera)
            if captureSession.canAddInput(input) {
                captureSession.addInput(input)
            }
        } catch {
            print("Camera input error: \(error)")
            return
        }
        
        // Photo output
        if captureSession.canAddOutput(photoOutput) {
            captureSession.addOutput(photoOutput)
        }
        
        captureSession.commitConfiguration()
        
        // Preview layer
        DispatchQueue.main.async { [weak self] in
            self?.setupPreviewLayer()
        }
        
        // Start session
        DispatchQueue.global(qos: .userInitiated).async { [weak self] in
            self?.captureSession.startRunning()
        }
    }
    
    private func setupPreviewLayer() {
        previewLayer = AVCaptureVideoPreviewLayer(session: captureSession)
        previewLayer.videoGravity = .resizeAspectFill
        previewLayer.frame = previewView.bounds
        previewView.layer.addSublayer(previewLayer)
    }
    
    override func viewDidLayoutSubviews() {
        super.viewDidLayoutSubviews()
        previewLayer?.frame = previewView.bounds
    }
    
    // MARK: - Actions
    @objc func capturePhoto() {
        guard !isCapturing else { return }
        isCapturing = true
        
        // Animation
        UIView.animate(withDuration: 0.1, animations: {
            self.captureButton.transform = CGAffineTransform(scaleX: 0.9, y: 0.9)
        }) { _ in
            UIView.animate(withDuration: 0.1) {
                self.captureButton.transform = .identity
            }
        }
        
        let settings = AVCapturePhotoSettings()
        
        // กำหนด flash mode
        if isFlashEnabled && currentDevice?.hasFlash == true {
            settings.flashMode = .on
        } else {
            settings.flashMode = .off
        }
        
        // High quality
        settings.isHighResolutionPhotoEnabled = true
        
        photoOutput.capturePhoto(with: settings, delegate: self)
    }
    
    @objc func switchCamera() {
        guard let currentInput = captureSession.inputs.first as? AVCaptureDeviceInput else { return }
        
        let newPosition: AVCaptureDevice.Position = currentInput.device.position == .back ? .front : .back
        
        guard let newCamera = AVCaptureDevice.default(.builtInWideAngleCamera, for: .video, position: newPosition) else { return }
        
        captureSession.beginConfiguration()
        captureSession.removeInput(currentInput)
        
        do {
            let newInput = try AVCaptureDeviceInput(device: newCamera)
            if captureSession.canAddInput(newInput) {
                captureSession.addInput(newInput)
                currentDevice = newCamera
            }
        } catch {
            print("Switch camera error: \(error)")
        }
        
        captureSession.commitConfiguration()
    }
    
    @objc func toggleFlash() {
        isFlashEnabled.toggle()
        let iconName = isFlashEnabled ? "bolt.fill" : "bolt.slash.fill"
        flashButton.setImage(UIImage(systemName: iconName), for: .normal)
    }
    
    @objc func zoomChanged(_ sender: UISlider) {
        setZoom(CGFloat(sender.value))
    }
    
    @objc func handlePinch(_ gesture: UIPinchGestureRecognizer) {
        guard let device = currentDevice else { return }
        
        if gesture.state == .changed {
            let newFactor = device.videoZoomFactor * gesture.scale
            let clampedFactor = min(max(newFactor, 1.0), device.maxAvailableVideoZoomFactor)
            
            do {
                try device.lockForConfiguration()
                device.videoZoomFactor = clampedFactor
                device.unlockForConfiguration()
                
                zoomSlider.value = Float(clampedFactor)
            } catch { }
            
            gesture.scale = 1.0
        }
    }
    
    @objc func handleTap(_ gesture: UITapGestureRecognizer) {
        let location = gesture.location(in: previewView)
        let focusPoint = previewLayer.captureDevicePointConverted(fromLayerPoint: location)
        
        setFocus(at: focusPoint)
        showFocusIndicator(at: location)
    }
    
    private func setZoom(_ factor: CGFloat) {
        guard let device = currentDevice else { return }
        
        do {
            try device.lockForConfiguration()
            let maxZoom = min(device.maxAvailableVideoZoomFactor, 10.0)
            device.videoZoomFactor = min(max(factor, 1.0), maxZoom)
            device.unlockForConfiguration()
        } catch { }
    }
    
    private func setFocus(at point: CGPoint) {
        guard let device = currentDevice else { return }
        
        do {
            try device.lockForConfiguration()
            
            if device.isFocusPointOfInterestSupported {
                device.focusPointOfInterest = point
                device.focusMode = .autoFocus
            }
            
            if device.isExposurePointOfInterestSupported {
                device.exposurePointOfInterest = point
                device.exposureMode = .autoExpose
            }
            
            device.unlockForConfiguration()
        } catch { }
    }
    
    private func showFocusIndicator(at point: CGPoint) {
        let focusView = UIView(frame: CGRect(x: 0, y: 0, width: 60, height: 60))
        focusView.center = point
        focusView.layer.borderColor = UIColor.yellow.cgColor
        focusView.layer.borderWidth = 2
        focusView.backgroundColor = .clear
        previewView.addSubview(focusView)
        
        UIView.animate(withDuration: 0.3, animations: {
            focusView.transform = CGAffineTransform(scaleX: 0.7, y: 0.7)
        }) { _ in
            UIView.animate(withDuration: 0.2, delay: 0.5, options: []) {
                focusView.alpha = 0
            } completion: { _ in
                focusView.removeFromSuperview()
            }
        }
    }
    
    private func showPermissionAlert() {
        let alert = UIAlertController(
            title: "ต้องการสิทธิ์กล้อง",
            message: "กรุณาอนุญาตการเข้าถึงกล้องใน Settings",
            preferredStyle: .alert
        )
        
        alert.addAction(UIAlertAction(title: "ตั้งค่า", style: .default) { _ in
            if let url = URL(string: UIApplication.openSettingsURLString) {
                UIApplication.shared.open(url)
            }
        })
        
        alert.addAction(UIAlertAction(title: "ยกเลิก", style: .cancel))
        
        present(alert, animated: true)
    }
}

// MARK: - Photo Capture Delegate
extension CustomCameraViewController: AVCapturePhotoCaptureDelegate {
    func photoOutput(_ output: AVCapturePhotoOutput,
                    didFinishProcessingPhoto photo: AVCapturePhoto,
                    error: Error?) {
        isCapturing = false
        
        if let error = error {
            print("Photo capture error: \(error)")
            return
        }
        
        guard let imageData = photo.fileDataRepresentation(),
              let image = UIImage(data: imageData) else { return }
        
        // บันทึกหรือแสดงภาพ
        saveToPhotoLibrary(image)
    }
    
    private func saveToPhotoLibrary(_ image: UIImage) {
        PHPhotoLibrary.requestAuthorization(for: .addOnly) { [weak self] status in
            guard status == .authorized || status == .limited else { return }
            
            PHPhotoLibrary.shared().performChanges {
                PHAssetChangeRequest.creationRequestForAsset(from: image)
            } completionHandler: { success, error in
                DispatchQueue.main.async {
                    if success {
                        print("บันทึกภาพสำเร็จ")
                    } else if let error = error {
                        print("บันทึกภาพล้มเหลว: \(error)")
                    }
                }
            }
        }
    }
}
```

---

## 5. การบันทึกวิดีโอ

### 5.1 Video Recording

```swift
import AVFoundation

class VideoRecordingManager: NSObject {
    var captureSession: AVCaptureSession!
    var videoOutput: AVCaptureMovieFileOutput!
    var isRecording = false
    
    func startRecording() {
        guard !isRecording else { return }
        
        let outputURL = getOutputURL()
        
        // กำหนด video codec
        let videoCodec = AVVideoCodecType.h264
        videoOutput.setOutputSettings([AVVideoCodecKey: videoCodec],
                                       for: videoOutput.connection(with: .video)!)
        
        videoOutput.startRecording(to: outputURL, recordingDelegate: self)
        isRecording = true
        
        print("เริ่มบันทึกวิดีโอ")
    }
    
    func stopRecording() {
        guard isRecording else { return }
        videoOutput.stopRecording()
        isRecording = false
    }
    
    private func getOutputURL() -> URL {
        let tempDir = FileManager.default.temporaryDirectory
        let fileName = "video_\(Date().timeIntervalSince1970).mov"
        return tempDir.appendingPathComponent(fileName)
    }
    
    // กำหนดเวลาบันทึกสูงสุด
    func setMaxRecordingDuration(_ seconds: Double) {
        videoOutput.maxRecordedDuration = CMTime(seconds: seconds, preferredTimescale: 600)
    }
    
    // กำหนดขนาดไฟล์สูงสุด
    func setMaxFileSize(_ bytes: Int64) {
        videoOutput.maxRecordedFileSize = bytes
    }
}

extension VideoRecordingManager: AVCaptureFileOutputRecordingDelegate {
    func fileOutput(_ output: AVCaptureFileOutput,
                   didStartRecordingTo fileURL: URL,
                   from connections: [AVCaptureConnection]) {
        print("เริ่มบันทึกที่: \(fileURL)")
    }
    
    func fileOutput(_ output: AVCaptureFileOutput,
                   didFinishRecordingTo outputFileURL: URL,
                   from connections: [AVCaptureConnection],
                   error: Error?) {
        if let error = error {
            print("บันทึกล้มเหลว: \(error)")
            return
        }
        
        print("บันทึกเสร็จสิ้น: \(outputFileURL)")
        
        // บันทึกลงคลัง
        saveVideoToLibrary(at: outputFileURL)
    }
    
    private func saveVideoToLibrary(at url: URL) {
        PHPhotoLibrary.requestAuthorization(for: .addOnly) { status in
            guard status == .authorized || status == .limited else { return }
            
            PHPhotoLibrary.shared().performChanges {
                PHAssetChangeRequest.creationRequestForAssetFromVideo(atFileURL: url)
            } completionHandler: { success, error in
                if success {
                    print("บันทึกวิดีโอสำเร็จ")
                } else if let error = error {
                    print("บันทึกวิดีโอล้มเหลว: \(error)")
                }
                
                // ลบไฟล์ temp
                try? FileManager.default.removeItem(at: url)
            }
        }
    }
}
```

---

## 6. PhotosUI Framework

### 6.1 PHPhotoLibrary

```swift
import Photos

class PhotoLibraryManager {
    // ขอสิทธิ์เข้าถึงคลังภาพ
    func requestAccess() async -> PHAuthorizationStatus {
        let status = await PHPhotoLibrary.requestAuthorization(for: .readWrite)
        return status
    }
    
    // รับภาพทั้งหมด
    func fetchAllPhotos() -> PHFetchResult<PHAsset> {
        let options = PHFetchOptions()
        options.sortDescriptors = [NSSortDescriptor(key: "creationDate", ascending: false)]
        options.predicate = NSPredicate(format: "mediaType = %d", PHAssetMediaType.image.rawValue)
        
        return PHAsset.fetchAssets(with: options)
    }
    
    // รับ albums
    func fetchAlbums() -> PHFetchResult<PHAssetCollection> {
        return PHAssetCollection.fetchAssetCollections(
            with: .album,
            subtype: .any,
            options: nil
        )
    }
    
    // รับภาพจาก album
    func fetchPhotos(from album: PHAssetCollection) -> PHFetchResult<PHAsset> {
        let options = PHFetchOptions()
        options.sortDescriptors = [NSSortDescriptor(key: "creationDate", ascending: false)]
        
        return PHAsset.fetchAssets(in: album, options: options)
    }
    
    // โหลด image จาก PHAsset
    func loadImage(from asset: PHAsset, targetSize: CGSize) async -> UIImage? {
        let manager = PHImageManager.default()
        let options = PHImageRequestOptions()
        options.deliveryMode = .highQualityFormat
        options.isNetworkAccessAllowed = true
        options.isSynchronous = false
        
        return await withCheckedContinuation { continuation in
            manager.requestImage(
                for: asset,
                targetSize: targetSize,
                contentMode: .aspectFit,
                options: options
            ) { image, _ in
                continuation.resume(returning: image)
            }
        }
    }
}
```

---

## 7. การบันทึกและอ่านภาพ

### 7.1 การบันทึกภาพ

```swift
import Photos

class PhotoSaveManager {
    // บันทึกภาพลงคลัง
    func saveImage(_ image: UIImage) async throws {
        let status = await PHPhotoLibrary.requestAuthorization(for: .addOnly)
        guard status == .authorized || status == .limited else {
            throw PhotoError.permissionDenied
        }
        
        try await PHPhotoLibrary.shared().performChanges {
            PHAssetChangeRequest.creationRequestForAsset(from: image)
        }
    }
    
    // บันทึกภาพพร้อม metadata
    func saveImageWithMetadata(_ imageData: Data, metadata: [String: Any]) async throws {
        guard let source = CGImageSourceCreateWithData(imageData as CFData, nil),
              let uti = CGImageSourceGetType(source) else {
            throw PhotoError.invalidData
        }
        
        let mutableData = NSMutableData(data: imageData)
        guard let destination = CGImageDestinationCreateWithData(
            mutableData,
            uti,
            1,
            nil
        ) else {
            throw PhotoError.invalidData
        }
        
        CGImageDestinationAddImageFromSource(destination, source, 0, metadata as CFDictionary)
        CGImageDestinationFinalize(destination)
        
        try await PHPhotoLibrary.shared().performChanges {
            let request = PHAssetCreationRequest.forAsset()
            request.addResource(with: .photo, data: mutableData as Data, options: nil)
        }
    }
    
    // บันทึกลง album ที่ระบุ
    func saveImage(_ image: UIImage, toAlbum albumName: String) async throws {
        let album = try await findOrCreateAlbum(named: albumName)
        
        try await PHPhotoLibrary.shared().performChanges {
            let assetRequest = PHAssetChangeRequest.creationRequestForAsset(from: image)
            
            if let albumRequest = PHAssetCollectionChangeRequest(for: album) {
                let placeholder = assetRequest.placeholderForCreatedAsset
                albumRequest.addAssets([placeholder] as NSFastEnumeration)
            }
        }
    }
    
    private func findOrCreateAlbum(named name: String) async throws -> PHAssetCollection {
        // หา album ที่มีชื่อนี้
        let options = PHFetchOptions()
        options.predicate = NSPredicate(format: "title = %@", name)
        let result = PHAssetCollection.fetchAssetCollections(with: .album, subtype: .any, options: options)
        
        if let album = result.firstObject {
            return album
        }
        
        // สร้าง album ใหม่
        var placeholder: PHObjectPlaceholder?
        
        try await PHPhotoLibrary.shared().performChanges {
            let request = PHAssetCollectionChangeRequest.creationRequestForAssetCollection(withTitle: name)
            placeholder = request.placeholderForCreatedAssetCollection
        }
        
        guard let placeholderID = placeholder?.localIdentifier,
              let album = PHAssetCollection.fetchAssetCollections(
                withLocalIdentifiers: [placeholderID],
                options: nil
              ).firstObject else {
            throw PhotoError.albumCreationFailed
        }
        
        return album
    }
}

enum PhotoError: Error {
    case permissionDenied
    case invalidData
    case albumCreationFailed
    case assetNotFound
}
```

---

## 8. SwiftUI Camera Integration

### 8.1 Camera View ใน SwiftUI

```swift
import SwiftUI
import AVFoundation

// UIViewControllerRepresentable สำหรับใช้ Camera ใน SwiftUI
struct CameraView: UIViewControllerRepresentable {
    @Binding var capturedImage: UIImage?
    @Binding var isPresented: Bool
    
    func makeUIViewController(context: Context) -> CustomCameraViewController {
        let controller = CustomCameraViewController()
        controller.delegate = context.coordinator
        return controller
    }
    
    func updateUIViewController(_ uiViewController: CustomCameraViewController, context: Context) {}
    
    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }
    
    class Coordinator: NSObject, CameraViewControllerDelegate {
        let parent: CameraView
        
        init(_ parent: CameraView) {
            self.parent = parent
        }
        
        func didCapturePhoto(_ image: UIImage) {
            parent.capturedImage = image
            parent.isPresented = false
        }
        
        func didCancel() {
            parent.isPresented = false
        }
    }
}

// Protocol สำหรับ delegate
protocol CameraViewControllerDelegate: AnyObject {
    func didCapturePhoto(_ image: UIImage)
    func didCancel()
}

// การใช้งานใน SwiftUI
struct PhotoCaptureView: View {
    @State private var capturedImage: UIImage?
    @State private var showCamera = false
    @State private var showPicker = false
    
    var body: some View {
        NavigationView {
            VStack(spacing: 20) {
                // แสดงภาพที่ถ่าย
                if let image = capturedImage {
                    Image(uiImage: image)
                        .resizable()
                        .scaledToFit()
                        .frame(maxHeight: 400)
                        .cornerRadius(12)
                        .shadow(radius: 5)
                } else {
                    RoundedRectangle(cornerRadius: 12)
                        .fill(Color(.systemGray6))
                        .frame(height: 300)
                        .overlay(
                            Image(systemName: "photo")
                                .font(.system(size: 60))
                                .foregroundColor(.secondary)
                        )
                }
                
                // ปุ่มถ่ายภาพ
                HStack(spacing: 20) {
                    Button(action: { showCamera = true }) {
                        Label("ถ่ายภาพ", systemImage: "camera.fill")
                            .frame(maxWidth: .infinity)
                            .padding()
                            .background(.blue)
                            .foregroundColor(.white)
                            .cornerRadius(12)
                    }
                    
                    Button(action: { showPicker = true }) {
                        Label("เลือกภาพ", systemImage: "photo.fill")
                            .frame(maxWidth: .infinity)
                            .padding()
                            .background(.green)
                            .foregroundColor(.white)
                            .cornerRadius(12)
                    }
                }
                
                Spacer()
            }
            .padding()
            .navigationTitle("Camera Demo")
            .fullScreenCover(isPresented: $showCamera) {
                CameraView(capturedImage: $capturedImage, isPresented: $showCamera)
                    .ignoresSafeArea()
            }
            .sheet(isPresented: $showPicker) {
                PhotoPickerView(selectedImage: $capturedImage)
            }
        }
    }
}

// SwiftUI Photo Picker Wrapper
struct PhotoPickerView: UIViewControllerRepresentable {
    @Binding var selectedImage: UIImage?
    @Environment(\.dismiss) private var dismiss
    
    func makeUIViewController(context: Context) -> PHPickerViewController {
        var config = PHPickerConfiguration()
        config.filter = .images
        config.selectionLimit = 1
        
        let picker = PHPickerViewController(configuration: config)
        picker.delegate = context.coordinator
        return picker
    }
    
    func updateUIViewController(_ uiViewController: PHPickerViewController, context: Context) {}
    
    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }
    
    class Coordinator: NSObject, PHPickerViewControllerDelegate {
        let parent: PhotoPickerView
        
        init(_ parent: PhotoPickerView) {
            self.parent = parent
        }
        
        func picker(_ picker: PHPickerViewController, didFinishPicking results: [PHPickerResult]) {
            parent.dismiss()
            
            guard let result = results.first else { return }
            
            if result.itemProvider.canLoadObject(ofClass: UIImage.self) {
                result.itemProvider.loadObject(ofClass: UIImage.self) { object, _ in
                    DispatchQueue.main.async {
                        self.parent.selectedImage = object as? UIImage
                    }
                }
            }
        }
    }
}
```

---

## 9. SwiftUI PhotosPicker

### 9.1 PhotosPicker (iOS 16+)

```swift
import SwiftUI
import PhotosUI

@available(iOS 16.0, *)
struct SwiftUIPhotosPicker: View {
    @State private var selectedItem: PhotosPickerItem?
    @State private var selectedImage: Image?
    @State private var imageData: Data?
    
    var body: some View {
        VStack {
            if let selectedImage {
                selectedImage
                    .resizable()
                    .scaledToFit()
                    .frame(maxHeight: 400)
                    .cornerRadius(12)
            } else {
                RoundedRectangle(cornerRadius: 12)
                    .fill(Color(.systemGray6))
                    .frame(height: 300)
                    .overlay(Text("ยังไม่ได้เลือกภาพ").foregroundColor(.secondary))
            }
            
            // PhotosPicker
            PhotosPicker(
                selection: $selectedItem,
                matching: .images,
                photoLibrary: .shared()
            ) {
                Label("เลือกภาพ", systemImage: "photo.fill")
                    .padding()
                    .background(.blue)
                    .foregroundColor(.white)
                    .cornerRadius(10)
            }
            .onChange(of: selectedItem) { newItem in
                Task {
                    await loadSelectedImage(from: newItem)
                }
            }
        }
        .padding()
    }
    
    func loadSelectedImage(from item: PhotosPickerItem?) async {
        guard let item else { return }
        
        // โหลดเป็น Data
        if let data = try? await item.loadTransferable(type: Data.self) {
            imageData = data
            if let uiImage = UIImage(data: data) {
                selectedImage = Image(uiImage: uiImage)
            }
        }
    }
}

// PhotosPicker หลายภาพ
@available(iOS 16.0, *)
struct MultiPhotoPickerView: View {
    @State private var selectedItems: [PhotosPickerItem] = []
    @State private var selectedImages: [UIImage] = []
    
    var body: some View {
        VStack {
            ScrollView(.horizontal) {
                HStack {
                    ForEach(selectedImages, id: \.self) { image in
                        Image(uiImage: image)
                            .resizable()
                            .scaledToFill()
                            .frame(width: 100, height: 100)
                            .clipShape(RoundedRectangle(cornerRadius: 8))
                    }
                }
                .padding()
            }
            
            PhotosPicker(
                selection: $selectedItems,
                maxSelectionCount: 10,
                matching: .any(of: [.images, .livePhotos])
            ) {
                Text("เลือกภาพสูงสุด 10 ภาพ")
                    .padding()
                    .background(.blue)
                    .foregroundColor(.white)
                    .cornerRadius(10)
            }
            .onChange(of: selectedItems) { items in
                Task {
                    await loadImages(from: items)
                }
            }
        }
    }
    
    func loadImages(from items: [PhotosPickerItem]) async {
        var images: [UIImage] = []
        
        for item in items {
            if let data = try? await item.loadTransferable(type: Data.self),
               let image = UIImage(data: data) {
                images.append(image)
            }
        }
        
        selectedImages = images
    }
}
```

---

## 10. Image Processing ด้วย CoreImage

### 10.1 CIFilter พื้นฐาน

```swift
import CoreImage
import CoreImage.CIFilterBuiltins
import UIKit

class ImageProcessor {
    let context = CIContext()
    
    // ปรับสี
    func adjustColor(
        image: UIImage,
        brightness: Double = 0,
        contrast: Double = 1,
        saturation: Double = 1
    ) -> UIImage? {
        guard let ciImage = CIImage(image: image) else { return nil }
        
        let filter = CIFilter.colorControls()
        filter.inputImage = ciImage
        filter.brightness = Float(brightness)
        filter.contrast = Float(contrast)
        filter.saturation = Float(saturation)
        
        return renderCIImage(filter.outputImage, originalSize: image.size)
    }
    
    // Blur
    func applyBlur(image: UIImage, radius: Double = 10) -> UIImage? {
        guard let ciImage = CIImage(image: image) else { return nil }
        
        let filter = CIFilter.gaussianBlur()
        filter.inputImage = ciImage
        filter.radius = Float(radius)
        
        return renderCIImage(filter.outputImage, originalSize: image.size)
    }
    
    // Vignette effect
    func applyVignette(image: UIImage, intensity: Double = 1) -> UIImage? {
        guard let ciImage = CIImage(image: image) else { return nil }
        
        let filter = CIFilter.vignette()
        filter.inputImage = ciImage
        filter.intensity = Float(intensity)
        filter.radius = 1.0
        
        return renderCIImage(filter.outputImage, originalSize: image.size)
    }
    
    // Sepia tone
    func applySepia(image: UIImage, intensity: Double = 0.8) -> UIImage? {
        guard let ciImage = CIImage(image: image) else { return nil }
        
        let filter = CIFilter.sepiaTone()
        filter.inputImage = ciImage
        filter.intensity = Float(intensity)
        
        return renderCIImage(filter.outputImage, originalSize: image.size)
    }
    
    // Sharpen
    func sharpenImage(image: UIImage, sharpness: Double = 0.5) -> UIImage? {
        guard let ciImage = CIImage(image: image) else { return nil }
        
        let filter = CIFilter.sharpenLuminance()
        filter.inputImage = ciImage
        filter.sharpness = Float(sharpness)
        
        return renderCIImage(filter.outputImage, originalSize: image.size)
    }
    
    // Combine filters
    func applyInstagramFilter(image: UIImage) -> UIImage? {
        guard var ciImage = CIImage(image: image) else { return nil }
        
        // Step 1: Adjust colors
        let colorFilter = CIFilter.colorControls()
        colorFilter.inputImage = ciImage
        colorFilter.saturation = 1.3
        colorFilter.brightness = 0.05
        colorFilter.contrast = 1.1
        ciImage = colorFilter.outputImage ?? ciImage
        
        // Step 2: Vignette
        let vignetteFilter = CIFilter.vignette()
        vignetteFilter.inputImage = ciImage
        vignetteFilter.intensity = 0.5
        vignetteFilter.radius = 1.5
        ciImage = vignetteFilter.outputImage ?? ciImage
        
        return renderCIImage(ciImage, originalSize: image.size)
    }
    
    private func renderCIImage(_ ciImage: CIImage?, originalSize: CGSize) -> UIImage? {
        guard let ciImage = ciImage else { return nil }
        
        guard let cgImage = context.createCGImage(ciImage, from: ciImage.extent) else {
            return nil
        }
        
        return UIImage(cgImage: cgImage)
    }
}
```

### 10.2 SwiftUI Image Editor

```swift
import SwiftUI
import CoreImage.CIFilterBuiltins

@available(iOS 16.0, *)
struct ImageEditorView: View {
    let originalImage: UIImage
    
    @State private var brightness: Double = 0
    @State private var contrast: Double = 1
    @State private var saturation: Double = 1
    @State private var blurRadius: Double = 0
    @State private var processedImage: UIImage?
    
    private let processor = ImageProcessor()
    
    var body: some View {
        VStack {
            // แสดงภาพที่แก้ไข
            Group {
                if let processed = processedImage {
                    Image(uiImage: processed)
                        .resizable()
                        .scaledToFit()
                } else {
                    Image(uiImage: originalImage)
                        .resizable()
                        .scaledToFit()
                }
            }
            .frame(maxHeight: 300)
            .cornerRadius(12)
            
            Divider()
            
            // Controls
            ScrollView {
                VStack(spacing: 20) {
                    SliderControl(
                        title: "ความสว่าง",
                        value: $brightness,
                        range: -0.5...0.5
                    )
                    
                    SliderControl(
                        title: "คอนทราสต์",
                        value: $contrast,
                        range: 0.5...2.0
                    )
                    
                    SliderControl(
                        title: "ความอิ่มสี",
                        value: $saturation,
                        range: 0...2
                    )
                    
                    SliderControl(
                        title: "เบลอ",
                        value: $blurRadius,
                        range: 0...20
                    )
                    
                    // Filter presets
                    HStack(spacing: 12) {
                        FilterPresetButton(name: "ต้นฉบับ") {
                            resetFilters()
                        }
                        FilterPresetButton(name: "Vivid") {
                            applyVividPreset()
                        }
                        FilterPresetButton(name: "Matte") {
                            applyMattePreset()
                        }
                        FilterPresetButton(name: "B&W") {
                            applyBWPreset()
                        }
                    }
                }
                .padding()
            }
        }
        .onChange(of: brightness) { _ in applyFilters() }
        .onChange(of: contrast) { _ in applyFilters() }
        .onChange(of: saturation) { _ in applyFilters() }
        .onChange(of: blurRadius) { _ in applyFilters() }
    }
    
    func applyFilters() {
        Task {
            var image = originalImage
            
            if let adjusted = processor.adjustColor(
                image: image,
                brightness: brightness,
                contrast: contrast,
                saturation: saturation
            ) {
                image = adjusted
            }
            
            if blurRadius > 0, let blurred = processor.applyBlur(image: image, radius: blurRadius) {
                image = blurred
            }
            
            processedImage = image
        }
    }
    
    func resetFilters() {
        brightness = 0
        contrast = 1
        saturation = 1
        blurRadius = 0
        processedImage = nil
    }
    
    func applyVividPreset() {
        brightness = 0.05
        contrast = 1.2
        saturation = 1.5
        blurRadius = 0
    }
    
    func applyMattePreset() {
        brightness = 0.02
        contrast = 0.9
        saturation = 0.8
        blurRadius = 0
    }
    
    func applyBWPreset() {
        brightness = 0
        contrast = 1.1
        saturation = 0
        blurRadius = 0
    }
}

struct SliderControl: View {
    let title: String
    @Binding var value: Double
    let range: ClosedRange<Double>
    
    var body: some View {
        VStack(alignment: .leading, spacing: 4) {
            HStack {
                Text(title)
                    .font(.subheadline)
                Spacer()
                Text(String(format: "%.2f", value))
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
            Slider(value: $value, in: range)
                .tint(.blue)
        }
    }
}

struct FilterPresetButton: View {
    let name: String
    let action: () -> Void
    
    var body: some View {
        Button(action: action) {
            Text(name)
                .font(.caption)
                .padding(.horizontal, 12)
                .padding(.vertical, 6)
                .background(Color(.systemGray5))
                .cornerRadius(8)
        }
    }
}
```

---

## 11. Face Detection

### 11.1 การตรวจจับใบหน้าด้วย Vision Framework

```swift
import Vision
import CoreImage
import UIKit

class FaceDetector {
    // ตรวจจับใบหน้าใน image
    func detectFaces(in image: UIImage) async -> [VNFaceObservation] {
        guard let cgImage = image.cgImage else { return [] }
        
        return await withCheckedContinuation { continuation in
            let request = VNDetectFaceRectanglesRequest { request, error in
                if let error = error {
                    print("Face detection error: \(error)")
                    continuation.resume(returning: [])
                    return
                }
                
                let observations = request.results as? [VNFaceObservation] ?? []
                continuation.resume(returning: observations)
            }
            
            let handler = VNImageRequestHandler(cgImage: cgImage, options: [:])
            
            do {
                try handler.perform([request])
            } catch {
                print("Failed to perform request: \(error)")
                continuation.resume(returning: [])
            }
        }
    }
    
    // ตรวจจับ face landmarks (ตา จมูก ปาก)
    func detectFaceLandmarks(in image: UIImage) async -> [VNFaceObservation] {
        guard let cgImage = image.cgImage else { return [] }
        
        return await withCheckedContinuation { continuation in
            let request = VNDetectFaceLandmarksRequest { request, error in
                let observations = request.results as? [VNFaceObservation] ?? []
                continuation.resume(returning: observations)
            }
            
            let handler = VNImageRequestHandler(cgImage: cgImage, options: [:])
            
            do {
                try handler.perform([request])
            } catch {
                continuation.resume(returning: [])
            }
        }
    }
    
    // วาด bounding box รอบใบหน้า
    func drawFaceBoxes(on image: UIImage, faces: [VNFaceObservation]) -> UIImage {
        let imageSize = image.size
        let scale: CGFloat = image.scale
        
        UIGraphicsBeginImageContextWithOptions(imageSize, false, scale)
        defer { UIGraphicsEndImageContext() }
        
        image.draw(at: .zero)
        
        guard let context = UIGraphicsGetCurrentContext() else {
            return image
        }
        
        context.setStrokeColor(UIColor.systemGreen.cgColor)
        context.setLineWidth(3)
        
        for face in faces {
            // แปลงพิกัดจาก normalized (0-1) เป็น image coordinates
            let boundingBox = face.boundingBox
            let faceRect = CGRect(
                x: boundingBox.origin.x * imageSize.width,
                y: (1 - boundingBox.origin.y - boundingBox.height) * imageSize.height,
                width: boundingBox.width * imageSize.width,
                height: boundingBox.height * imageSize.height
            )
            
            context.stroke(faceRect)
            
            // วาด landmarks ถ้ามี
            if let landmarks = face.landmarks {
                drawLandmarks(landmarks, in: faceRect, context: context, imageSize: imageSize)
            }
        }
        
        return UIGraphicsGetImageFromCurrentImageContext() ?? image
    }
    
    private func drawLandmarks(
        _ landmarks: VNFaceLandmarks2D,
        in faceRect: CGRect,
        context: CGContext,
        imageSize: CGSize
    ) {
        context.setStrokeColor(UIColor.systemBlue.cgColor)
        context.setLineWidth(1)
        
        let landmarkRegions: [VNFaceLandmarkRegion2D?] = [
            landmarks.leftEye,
            landmarks.rightEye,
            landmarks.nose,
            landmarks.outerLips,
            landmarks.innerLips
        ]
        
        for region in landmarkRegions.compactMap({ $0 }) {
            guard region.pointCount > 0 else { continue }
            
            let points = region.normalizedPoints.map { point in
                CGPoint(
                    x: faceRect.origin.x + point.x * faceRect.width,
                    y: faceRect.origin.y + (1 - point.y) * faceRect.height
                )
            }
            
            context.beginPath()
            context.addLines(between: points)
            context.closePath()
            context.strokePath()
        }
    }
}

// SwiftUI Face Detection View
@available(iOS 16.0, *)
struct FaceDetectionView: View {
    @State private var selectedItem: PhotosPickerItem?
    @State private var originalImage: UIImage?
    @State private var processedImage: UIImage?
    @State private var faceCount = 0
    @State private var isProcessing = false
    
    private let detector = FaceDetector()
    
    var body: some View {
        VStack(spacing: 16) {
            Group {
                if let processed = processedImage {
                    Image(uiImage: processed)
                        .resizable()
                        .scaledToFit()
                } else if let original = originalImage {
                    Image(uiImage: original)
                        .resizable()
                        .scaledToFit()
                } else {
                    RoundedRectangle(cornerRadius: 12)
                        .fill(Color(.systemGray6))
                        .overlay(Text("เลือกภาพเพื่อตรวจจับใบหน้า").foregroundColor(.secondary))
                }
            }
            .frame(maxHeight: 400)
            .cornerRadius(12)
            
            if faceCount > 0 {
                Text("พบ \(faceCount) ใบหน้า")
                    .font(.headline)
                    .foregroundColor(.green)
            }
            
            if isProcessing {
                ProgressView("กำลังตรวจจับ...")
            }
            
            PhotosPicker(selection: $selectedItem, matching: .images) {
                Label("เลือกภาพ", systemImage: "photo.fill")
                    .padding()
                    .background(.blue)
                    .foregroundColor(.white)
                    .cornerRadius(10)
            }
            .onChange(of: selectedItem) { item in
                Task { await processSelectedImage(item) }
            }
        }
        .padding()
        .navigationTitle("Face Detection")
    }
    
    func processSelectedImage(_ item: PhotosPickerItem?) async {
        guard let item else { return }
        
        isProcessing = true
        
        if let data = try? await item.loadTransferable(type: Data.self),
           let image = UIImage(data: data) {
            originalImage = image
            
            let faces = await detector.detectFaces(in: image)
            faceCount = faces.count
            processedImage = detector.drawFaceBoxes(on: image, faces: faces)
        }
        
        isProcessing = false
    }
}
```

---

## 12. QR Code Scanning

### 12.1 AVMetadataObject สำหรับ QR Code

```swift
import AVFoundation
import UIKit

class QRCodeScannerViewController: UIViewController {
    private var captureSession = AVCaptureSession()
    private var previewLayer: AVCaptureVideoPreviewLayer!
    
    var onCodeDetected: ((String) -> Void)?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupScanner()
    }
    
    private func setupScanner() {
        captureSession.beginConfiguration()
        
        // Camera input
        guard let camera = AVCaptureDevice.default(for: .video) else {
            showAlert(message: "ไม่พบกล้อง")
            return
        }
        
        do {
            let input = try AVCaptureDeviceInput(device: camera)
            captureSession.addInput(input)
        } catch {
            showAlert(message: "ไม่สามารถเข้าถึงกล้อง")
            return
        }
        
        // Metadata output
        let metadataOutput = AVCaptureMetadataOutput()
        
        if captureSession.canAddOutput(metadataOutput) {
            captureSession.addOutput(metadataOutput)
            
            metadataOutput.setMetadataObjectsDelegate(self, queue: .main)
            
            // กำหนดประเภทที่ต้องการสแกน
            metadataOutput.metadataObjectTypes = [
                .qr,           // QR Code
                .ean13,        // บาร์โค้ด 13 หลัก
                .ean8,         // บาร์โค้ด 8 หลัก
                .code128,      // Code 128
                .pdf417,       // PDF417
                .dataMatrix    // Data Matrix
            ]
        }
        
        captureSession.commitConfiguration()
        
        // Preview layer
        previewLayer = AVCaptureVideoPreviewLayer(session: captureSession)
        previewLayer.frame = view.bounds
        previewLayer.videoGravity = .resizeAspectFill
        view.layer.addSublayer(previewLayer)
        
        // เพิ่ม UI overlay
        addScanningOverlay()
        
        // Start
        DispatchQueue.global(qos: .userInitiated).async { [weak self] in
            self?.captureSession.startRunning()
        }
    }
    
    private func addScanningOverlay() {
        // Frame สำหรับ scanning area
        let overlayView = UIView(frame: view.bounds)
        overlayView.backgroundColor = UIColor.black.withAlphaComponent(0.5)
        view.addSubview(overlayView)
        
        // Clear scanning area
        let scanSize = CGSize(width: 250, height: 250)
        let scanRect = CGRect(
            x: (view.bounds.width - scanSize.width) / 2,
            y: (view.bounds.height - scanSize.height) / 2,
            width: scanSize.width,
            height: scanSize.height
        )
        
        let maskLayer = CAShapeLayer()
        let path = UIBezierPath(rect: overlayView.bounds)
        let scanPath = UIBezierPath(roundedRect: scanRect, cornerRadius: 10)
        path.append(scanPath)
        path.usesEvenOddFillRule = true
        maskLayer.path = path.cgPath
        maskLayer.fillRule = .evenOdd
        overlayView.layer.mask = maskLayer
        
        // Border ของ scanning area
        let borderLayer = CAShapeLayer()
        borderLayer.path = UIBezierPath(roundedRect: scanRect, cornerRadius: 10).cgPath
        borderLayer.strokeColor = UIColor.systemGreen.cgColor
        borderLayer.lineWidth = 3
        borderLayer.fillColor = UIColor.clear.cgColor
        view.layer.addSublayer(borderLayer)
        
        // Scanning line animation
        addScanLineAnimation(in: scanRect)
        
        // Label
        let label = UILabel()
        label.text = "วาง QR Code ภายในกรอบ"
        label.textColor = .white
        label.textAlignment = .center
        label.font = .systemFont(ofSize: 15)
        label.frame = CGRect(
            x: 20,
            y: scanRect.maxY + 20,
            width: view.bounds.width - 40,
            height: 30
        )
        view.addSubview(label)
    }
    
    private func addScanLineAnimation(in rect: CGRect) {
        let line = UIView(frame: CGRect(x: rect.minX, y: rect.minY, width: rect.width, height: 2))
        line.backgroundColor = .systemGreen
        view.addSubview(line)
        
        UIView.animate(
            withDuration: 2.0,
            delay: 0,
            options: [.repeat, .autoreverse]
        ) {
            line.frame.origin.y = rect.maxY
        }
    }
    
    private func showAlert(message: String) {
        let alert = UIAlertController(title: "ข้อผิดพลาด", message: message, preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "ตกลง", style: .default))
        present(alert, animated: true)
    }
    
    override func viewDidLayoutSubviews() {
        super.viewDidLayoutSubviews()
        previewLayer?.frame = view.bounds
    }
}

extension QRCodeScannerViewController: AVCaptureMetadataOutputObjectsDelegate {
    func metadataOutput(
        _ output: AVCaptureMetadataOutput,
        didOutput metadataObjects: [AVMetadataObject],
        from connection: AVCaptureConnection
    ) {
        guard let metadataObject = metadataObjects.first as? AVMetadataMachineReadableCodeObject,
              let stringValue = metadataObject.stringValue else { return }
        
        // Haptic feedback
        let generator = UINotificationFeedbackGenerator()
        generator.notificationOccurred(.success)
        
        // หยุด scanning ชั่วคราว
        captureSession.stopRunning()
        
        print("สแกนได้: \(stringValue)")
        onCodeDetected?(stringValue)
        
        // แสดง result
        showResult(stringValue)
    }
    
    private func showResult(_ result: String) {
        let alert = UIAlertController(
            title: "สแกนสำเร็จ",
            message: result,
            preferredStyle: .alert
        )
        
        alert.addAction(UIAlertAction(title: "คัดลอก", style: .default) { _ in
            UIPasteboard.general.string = result
        })
        
        alert.addAction(UIAlertAction(title: "สแกนต่อ", style: .cancel) { [weak self] _ in
            DispatchQueue.global(qos: .userInitiated).async {
                self?.captureSession.startRunning()
            }
        })
        
        present(alert, animated: true)
    }
}

// SwiftUI QR Scanner
struct QRScannerView: UIViewControllerRepresentable {
    @Binding var scannedCode: String?
    @Binding var isPresented: Bool
    
    func makeUIViewController(context: Context) -> QRCodeScannerViewController {
        let vc = QRCodeScannerViewController()
        vc.onCodeDetected = { code in
            scannedCode = code
            isPresented = false
        }
        return vc
    }
    
    func updateUIViewController(_ uiViewController: QRCodeScannerViewController, context: Context) {}
}

// SwiftUI Usage
struct QRScannerApp: View {
    @State private var showScanner = false
    @State private var scannedCode: String?
    
    var body: some View {
        VStack(spacing: 20) {
            if let code = scannedCode {
                VStack {
                    Text("ผลลัพธ์:")
                        .font(.headline)
                    Text(code)
                        .padding()
                        .background(Color(.systemGray6))
                        .cornerRadius(8)
                }
            }
            
            Button("สแกน QR Code") {
                showScanner = true
            }
            .padding()
            .background(.blue)
            .foregroundColor(.white)
            .cornerRadius(10)
        }
        .fullScreenCover(isPresented: $showScanner) {
            QRScannerView(scannedCode: $scannedCode, isPresented: $showScanner)
                .ignoresSafeArea()
        }
    }
}
```

---

## 13. แบบฝึกหัดพร้อมเฉลย

### แบบฝึกหัดที่ 1: Photo Filter App

สร้าง app แก้ไขภาพพร้อม filter ต่างๆ:

```swift
import SwiftUI
import PhotosUI
import CoreImage.CIFilterBuiltins

@available(iOS 16.0, *)
struct PhotoFilterApp: View {
    @State private var selectedItem: PhotosPickerItem?
    @State private var originalImage: UIImage?
    @State private var filteredImage: UIImage?
    @State private var selectedFilter: FilterOption = .none
    @State private var brightness: Double = 0
    @State private var contrast: Double = 1
    @State private var saturation: Double = 1
    
    enum FilterOption: String, CaseIterable {
        case none = "ต้นฉบับ"
        case sepia = "Sepia"
        case noir = "Noir"
        case chrome = "Chrome"
        case fade = "Fade"
        case vivid = "Vivid"
        
        var icon: String {
            switch self {
            case .none: return "photo"
            case .sepia: return "wand.and.stars"
            case .noir: return "moon.fill"
            case .chrome: return "circle.hexagonpath.fill"
            case .fade: return "sun.min"
            case .vivid: return "sun.max.fill"
            }
        }
    }
    
    let processor = ImageProcessor()
    
    var body: some View {
        NavigationView {
            VStack(spacing: 0) {
                // Image Display
                ZStack {
                    Color.black
                    
                    if let filtered = filteredImage {
                        Image(uiImage: filtered)
                            .resizable()
                            .scaledToFit()
                    } else if let original = originalImage {
                        Image(uiImage: original)
                            .resizable()
                            .scaledToFit()
                    } else {
                        VStack {
                            Image(systemName: "photo.fill")
                                .font(.system(size: 60))
                                .foregroundColor(.gray)
                            Text("เลือกภาพเพื่อแก้ไข")
                                .foregroundColor(.gray)
                        }
                    }
                }
                .frame(height: 350)
                
                // Filter Strip
                ScrollView(.horizontal, showsIndicators: false) {
                    HStack(spacing: 12) {
                        ForEach(FilterOption.allCases, id: \.self) { filter in
                            FilterThumbnail(
                                filter: filter,
                                isSelected: selectedFilter == filter,
                                baseImage: originalImage
                            ) {
                                selectedFilter = filter
                                applyFilter()
                            }
                        }
                    }
                    .padding(.horizontal)
                }
                .frame(height: 100)
                .background(Color(.systemGray6))
                
                // Adjustment Controls
                if originalImage != nil {
                    VStack(spacing: 12) {
                        AdjustmentRow(title: "ความสว่าง", value: $brightness, range: -0.5...0.5)
                        AdjustmentRow(title: "คอนทราสต์", value: $contrast, range: 0.5...2.0)
                        AdjustmentRow(title: "ความอิ่มสี", value: $saturation, range: 0...2.0)
                    }
                    .padding()
                }
                
                Spacer()
            }
            .navigationTitle("ตัวกรองภาพ")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .navigationBarLeading) {
                    PhotosPicker(selection: $selectedItem, matching: .images) {
                        Image(systemName: "photo.fill")
                    }
                    .onChange(of: selectedItem) { item in
                        Task { await loadImage(from: item) }
                    }
                }
                
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button("บันทึก") {
                        saveImage()
                    }
                    .disabled(filteredImage == nil && originalImage == nil)
                }
            }
            .onChange(of: brightness) { _ in applyFilter() }
            .onChange(of: contrast) { _ in applyFilter() }
            .onChange(of: saturation) { _ in applyFilter() }
        }
    }
    
    func loadImage(from item: PhotosPickerItem?) async {
        guard let item,
              let data = try? await item.loadTransferable(type: Data.self),
              let image = UIImage(data: data) else { return }
        
        originalImage = image
        applyFilter()
    }
    
    func applyFilter() {
        guard let original = originalImage else { return }
        
        Task {
            var image = original
            
            // Apply preset filter
            switch selectedFilter {
            case .none:
                break
            case .sepia:
                image = processor.applySepia(image: image) ?? image
            case .noir:
                image = processor.adjustColor(image: image, brightness: 0, contrast: 1.2, saturation: 0) ?? image
            case .chrome:
                image = processor.adjustColor(image: image, brightness: 0.1, contrast: 1.2, saturation: 1.5) ?? image
            case .fade:
                image = processor.adjustColor(image: image, brightness: 0.1, contrast: 0.8, saturation: 0.7) ?? image
            case .vivid:
                image = processor.adjustColor(image: image, brightness: 0.05, contrast: 1.2, saturation: 1.8) ?? image
            }
            
            // Apply manual adjustments
            image = processor.adjustColor(
                image: image,
                brightness: brightness,
                contrast: contrast,
                saturation: saturation
            ) ?? image
            
            filteredImage = image
        }
    }
    
    func saveImage() {
        let imageToSave = filteredImage ?? originalImage
        
        guard let image = imageToSave else { return }
        
        Task {
            do {
                let saveManager = PhotoSaveManager()
                try await saveManager.saveImage(image)
                print("บันทึกสำเร็จ")
            } catch {
                print("บันทึกล้มเหลว: \(error)")
            }
        }
    }
}

struct FilterThumbnail: View {
    let filter: PhotoFilterApp.FilterOption
    let isSelected: Bool
    let baseImage: UIImage?
    let action: () -> Void
    
    var body: some View {
        Button(action: action) {
            VStack(spacing: 4) {
                RoundedRectangle(cornerRadius: 8)
                    .fill(Color(.systemGray4))
                    .frame(width: 60, height: 60)
                    .overlay(
                        Image(systemName: filter.icon)
                            .foregroundColor(.secondary)
                    )
                    .overlay(
                        RoundedRectangle(cornerRadius: 8)
                            .stroke(isSelected ? Color.blue : Color.clear, lineWidth: 3)
                    )
                
                Text(filter.rawValue)
                    .font(.caption2)
                    .foregroundColor(isSelected ? .blue : .primary)
            }
        }
    }
}

struct AdjustmentRow: View {
    let title: String
    @Binding var value: Double
    let range: ClosedRange<Double>
    
    var body: some View {
        HStack {
            Text(title)
                .font(.subheadline)
                .frame(width: 100, alignment: .leading)
            
            Slider(value: $value, in: range)
                .tint(.blue)
            
            Text(String(format: "%.2f", value))
                .font(.caption)
                .frame(width: 40)
                .foregroundColor(.secondary)
        }
    }
}
```

---

### แบบฝึกหัดที่ 2: Camera App สมบูรณ์

```swift
import SwiftUI
import AVFoundation
import PhotosUI

// MARK: - Camera ViewModel
class CameraViewModel: NSObject, ObservableObject {
    @Published var captureSession = AVCaptureSession()
    @Published var isCameraReady = false
    @Published var capturedImage: UIImage?
    @Published var isFlashOn = false
    @Published var currentPosition: AVCaptureDevice.Position = .back
    @Published var zoomFactor: CGFloat = 1.0
    @Published var isCapturing = false
    @Published var error: CameraAppError?
    
    private var photoOutput = AVCapturePhotoOutput()
    private var currentInput: AVCaptureDeviceInput?
    
    override init() {
        super.init()
        checkPermissionAndSetup()
    }
    
    func checkPermissionAndSetup() {
        switch AVCaptureDevice.authorizationStatus(for: .video) {
        case .authorized:
            setupSession()
        case .notDetermined:
            Task {
                let granted = await AVCaptureDevice.requestAccess(for: .video)
                if granted {
                    await MainActor.run { setupSession() }
                }
            }
        default:
            error = .permissionDenied
        }
    }
    
    func setupSession() {
        captureSession.beginConfiguration()
        captureSession.sessionPreset = .photo
        
        guard let device = getBestCamera(for: currentPosition) else {
            error = .cameraUnavailable
            captureSession.commitConfiguration()
            return
        }
        
        do {
            let input = try AVCaptureDeviceInput(device: device)
            
            if captureSession.canAddInput(input) {
                captureSession.addInput(input)
                currentInput = input
            }
        } catch {
            self.error = .setupFailed(error)
            captureSession.commitConfiguration()
            return
        }
        
        if captureSession.canAddOutput(photoOutput) {
            captureSession.addOutput(photoOutput)
        }
        
        captureSession.commitConfiguration()
        
        DispatchQueue.global(qos: .userInitiated).async { [weak self] in
            self?.captureSession.startRunning()
            DispatchQueue.main.async {
                self?.isCameraReady = true
            }
        }
    }
    
    func takePhoto() {
        guard !isCapturing else { return }
        isCapturing = true
        
        let settings = AVCapturePhotoSettings()
        settings.flashMode = isFlashOn ? .on : .off
        settings.isHighResolutionPhotoEnabled = true
        
        photoOutput.capturePhoto(with: settings, delegate: self)
    }
    
    func switchCamera() {
        currentPosition = currentPosition == .back ? .front : .back
        
        guard let newDevice = getBestCamera(for: currentPosition),
              let oldInput = currentInput else { return }
        
        captureSession.beginConfiguration()
        captureSession.removeInput(oldInput)
        
        do {
            let newInput = try AVCaptureDeviceInput(device: newDevice)
            if captureSession.canAddInput(newInput) {
                captureSession.addInput(newInput)
                currentInput = newInput
            }
        } catch {
            self.error = .setupFailed(error)
        }
        
        captureSession.commitConfiguration()
        zoomFactor = 1.0
    }
    
    func setZoom(_ factor: CGFloat) {
        guard let device = currentInput?.device else { return }
        
        do {
            try device.lockForConfiguration()
            let maxZoom = min(device.maxAvailableVideoZoomFactor, 10.0)
            device.videoZoomFactor = min(max(factor, 1.0), maxZoom)
            device.unlockForConfiguration()
            zoomFactor = device.videoZoomFactor
        } catch { }
    }
    
    private func getBestCamera(for position: AVCaptureDevice.Position) -> AVCaptureDevice? {
        return AVCaptureDevice.default(.builtInWideAngleCamera, for: .video, position: position)
    }
    
    func savePhoto() {
        guard let image = capturedImage else { return }
        
        Task {
            do {
                let manager = PhotoSaveManager()
                try await manager.saveImage(image)
                print("บันทึกสำเร็จ")
            } catch {
                await MainActor.run { self.error = .saveFailed(error) }
            }
        }
    }
}

extension CameraViewModel: AVCapturePhotoCaptureDelegate {
    func photoOutput(_ output: AVCapturePhotoOutput,
                    didFinishProcessingPhoto photo: AVCapturePhoto,
                    error: Error?) {
        isCapturing = false
        
        if let error = error {
            self.error = .captureFailed(error)
            return
        }
        
        guard let data = photo.fileDataRepresentation(),
              let image = UIImage(data: data) else { return }
        
        capturedImage = image
    }
}

enum CameraAppError: Error, Identifiable {
    case permissionDenied
    case cameraUnavailable
    case setupFailed(Error)
    case captureFailed(Error)
    case saveFailed(Error)
    
    var id: String { localizedDescription }
    
    var message: String {
        switch self {
        case .permissionDenied: return "กรุณาอนุญาตการเข้าถึงกล้องในการตั้งค่า"
        case .cameraUnavailable: return "ไม่พบกล้อง"
        case .setupFailed(let e): return "ตั้งค่าล้มเหลว: \(e.localizedDescription)"
        case .captureFailed(let e): return "ถ่ายภาพล้มเหลว: \(e.localizedDescription)"
        case .saveFailed(let e): return "บันทึกล้มเหลว: \(e.localizedDescription)"
        }
    }
}

// MARK: - Camera Preview
struct CameraPreview: UIViewRepresentable {
    let session: AVCaptureSession
    
    func makeUIView(context: Context) -> UIView {
        let view = UIView(frame: .zero)
        
        let previewLayer = AVCaptureVideoPreviewLayer(session: session)
        previewLayer.videoGravity = .resizeAspectFill
        previewLayer.frame = view.bounds
        view.layer.addSublayer(previewLayer)
        
        return view
    }
    
    func updateUIView(_ uiView: UIView, context: Context) {
        if let layer = uiView.layer.sublayers?.first as? AVCaptureVideoPreviewLayer {
            layer.frame = uiView.bounds
        }
    }
}

// MARK: - Main Camera View
struct MainCameraView: View {
    @StateObject private var viewModel = CameraViewModel()
    @State private var showCapturedPhoto = false
    @State private var captureAnimation = false
    
    var body: some View {
        ZStack {
            Color.black.ignoresSafeArea()
            
            if viewModel.isCameraReady {
                CameraPreview(session: viewModel.captureSession)
                    .ignoresSafeArea()
                    .gesture(
                        MagnificationGesture()
                            .onChanged { value in
                                viewModel.setZoom(viewModel.zoomFactor * value)
                            }
                    )
                
                // Flash overlay
                if captureAnimation {
                    Color.white.opacity(0.7)
                        .ignoresSafeArea()
                }
            }
            
            VStack {
                // Top Controls
                HStack {
                    // Flash toggle
                    Button(action: { viewModel.isFlashOn.toggle() }) {
                        Image(systemName: viewModel.isFlashOn ? "bolt.fill" : "bolt.slash.fill")
                            .font(.title2)
                            .foregroundColor(.white)
                            .padding()
                    }
                    
                    Spacer()
                    
                    // Zoom indicator
                    Text(String(format: "%.1fx", viewModel.zoomFactor))
                        .font(.caption)
                        .foregroundColor(.white)
                        .padding(6)
                        .background(.black.opacity(0.5))
                        .cornerRadius(8)
                    
                    Spacer()
                    
                    // Placeholder
                    Color.clear
                        .frame(width: 44, height: 44)
                }
                .padding(.horizontal)
                
                Spacer()
                
                // Bottom Controls
                HStack(alignment: .center, spacing: 40) {
                    // Gallery button
                    Button(action: {}) {
                        RoundedRectangle(cornerRadius: 8)
                            .fill(Color(.systemGray3))
                            .frame(width: 50, height: 50)
                            .overlay(
                                Image(systemName: "photo.fill")
                                    .foregroundColor(.white)
                            )
                    }
                    
                    // Capture button
                    Button(action: {
                        withAnimation(.easeInOut(duration: 0.1)) {
                            captureAnimation = true
                        }
                        DispatchQueue.main.asyncAfter(deadline: .now() + 0.15) {
                            captureAnimation = false
                        }
                        viewModel.takePhoto()
                        
                        DispatchQueue.main.asyncAfter(deadline: .now() + 0.5) {
                            if viewModel.capturedImage != nil {
                                showCapturedPhoto = true
                            }
                        }
                    }) {
                        ZStack {
                            Circle()
                                .fill(.white)
                                .frame(width: 70, height: 70)
                            
                            Circle()
                                .stroke(.white, lineWidth: 4)
                                .frame(width: 80, height: 80)
                        }
                    }
                    .disabled(viewModel.isCapturing)
                    
                    // Switch camera
                    Button(action: { viewModel.switchCamera() }) {
                        Image(systemName: "camera.rotate.fill")
                            .font(.title2)
                            .foregroundColor(.white)
                            .frame(width: 50, height: 50)
                            .background(Color(.systemGray3).opacity(0.7))
                            .clipShape(Circle())
                    }
                }
                .padding(.bottom, 40)
            }
        }
        .sheet(isPresented: $showCapturedPhoto) {
            if let image = viewModel.capturedImage {
                CapturedPhotoView(image: image) {
                    viewModel.savePhoto()
                }
            }
        }
        .alert(item: $viewModel.error) { error in
            Alert(
                title: Text("ข้อผิดพลาด"),
                message: Text(error.message),
                dismissButton: .default(Text("ตกลง"))
            )
        }
    }
}

struct CapturedPhotoView: View {
    let image: UIImage
    let onSave: () -> Void
    
    @Environment(\.dismiss) private var dismiss
    
    var body: some View {
        NavigationView {
            Image(uiImage: image)
                .resizable()
                .scaledToFit()
                .navigationTitle("ภาพที่ถ่าย")
                .navigationBarTitleDisplayMode(.inline)
                .toolbar {
                    ToolbarItem(placement: .navigationBarLeading) {
                        Button("ลบ") {
                            dismiss()
                        }
                        .foregroundColor(.red)
                    }
                    
                    ToolbarItem(placement: .navigationBarTrailing) {
                        Button("บันทึก") {
                            onSave()
                            dismiss()
                        }
                    }
                }
        }
    }
}
```

---

## 14. สรุป

ในบทนี้เราได้เรียนรู้:

1. **UIImagePickerController** - วิธีดั้งเดิมสำหรับเลือกภาพหรือถ่ายภาพ
2. **PHPickerViewController** - วิธีใหม่ที่ให้ความเป็นส่วนตัวมากกว่า
3. **AVFoundation** - framework สำหรับควบคุมกล้องเต็มรูปแบบ
4. **AVCaptureSession** - การจัดการ session สำหรับบันทึกภาพ/วิดีโอ
5. **AVCaptureDevice** - การควบคุมกล้อง focus, exposure, zoom
6. **Custom Camera UI** - การสร้างหน้าจอกล้องแบบ custom
7. **Video Recording** - การบันทึกวิดีโอ
8. **PhotosUI** - การเข้าถึงคลังภาพ
9. **การบันทึกและอ่านภาพ** - PHPhotoLibrary
10. **SwiftUI Integration** - การใช้กล้องและ picker ใน SwiftUI
11. **PhotosPicker** - API ใหม่ใน SwiftUI (iOS 16+)
12. **CoreImage** - การแก้ไขภาพด้วย filter
13. **CIFilter** - filter ต่างๆ สำหรับ image processing
14. **Face Detection** - Vision framework
15. **QR Code Scanning** - AVMetadataObject

### Tips สำคัญ

- ใช้ `PHPickerViewController` แทน `UIImagePickerController` สำหรับ app ใหม่
- สำหรับ custom camera ให้รัน `AVCaptureSession` บน background thread
- ใช้ `AVCaptureDevice.requestAccess` ก่อนเสมอ
- กรอง location metadata ออกจากภาพก่อนแชร์หากต้องการความเป็นส่วนตัว
- `CIContext` ควรสร้างเพียงครั้งเดียวและนำมาใช้ซ้ำ เพราะการสร้างแต่ละครั้งใช้ทรัพยากรมาก
- ใช้ `Vision` framework แทน `CIDetector` สำหรับ face detection ใน app ใหม่
