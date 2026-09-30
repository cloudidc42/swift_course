# Part 41: MapKit และ Location Services

## บทนำ

ในบทนี้เราจะเรียนรู้เกี่ยวกับการใช้งาน CoreLocation framework และ MapKit framework ซึ่งเป็นเครื่องมือสำคัญในการพัฒนา iOS app ที่เกี่ยวข้องกับตำแหน่งและแผนที่ เราจะครอบคลุมทุกอย่างตั้งแต่การขอสิทธิ์การเข้าถึงตำแหน่ง การแสดงแผนที่ ไปจนถึงการสร้าง annotation และ overlay ต่างๆ

---

## 1. CoreLocation Framework

### 1.1 ความรู้เบื้องต้น

CoreLocation เป็น framework ที่ Apple จัดเตรียมไว้สำหรับการทำงานเกี่ยวกับตำแหน่งทางภูมิศาสตร์ ประกอบด้วยความสามารถหลักดังนี้:

- **GPS Location**: ตำแหน่งจาก GPS satellite
- **WiFi Location**: ตำแหน่งจากเครือข่าย WiFi โดยรอบ
- **Cell Tower Location**: ตำแหน่งจากเสาสัญญาณมือถือ
- **Geocoding**: แปลงระหว่างพิกัดและที่อยู่
- **Region Monitoring**: ติดตามการเข้า-ออกพื้นที่
- **iBeacon Ranging**: ระบุระยะห่างจาก Bluetooth beacon

### 1.2 การเพิ่ม Framework

ใน Xcode ให้ import CoreLocation:

```swift
import CoreLocation
```

สำหรับ SwiftUI project ให้เพิ่ม import ใน file ที่ต้องการใช้งาน

---

## 2. CLLocationManager

### 2.1 การสร้าง CLLocationManager

`CLLocationManager` เป็น class หลักที่ใช้จัดการการทำงานทั้งหมดของ location services:

```swift
import CoreLocation

class LocationManager: NSObject {
    // สร้าง instance ของ CLLocationManager
    let locationManager = CLLocationManager()
    
    override init() {
        super.init()
        setupLocationManager()
    }
    
    private func setupLocationManager() {
        // กำหนด delegate
        locationManager.delegate = self
        
        // กำหนดความแม่นยำ
        locationManager.desiredAccuracy = kCLLocationAccuracyBest
        
        // กำหนดระยะที่ต้องเคลื่อนที่ก่อนอัปเดต (เมตร)
        locationManager.distanceFilter = 10.0
    }
}

// MARK: - CLLocationManagerDelegate
extension LocationManager: CLLocationManagerDelegate {
    func locationManager(_ manager: CLLocationManager, 
                        didUpdateLocations locations: [CLLocation]) {
        guard let location = locations.last else { return }
        print("ตำแหน่งปัจจุบัน: \(location.coordinate.latitude), \(location.coordinate.longitude)")
    }
    
    func locationManager(_ manager: CLLocationManager, 
                        didFailWithError error: Error) {
        print("เกิดข้อผิดพลาด: \(error.localizedDescription)")
    }
}
```

### 2.2 CLLocationManagerDelegate Methods

```swift
extension LocationManager: CLLocationManagerDelegate {
    // เรียกเมื่อได้รับตำแหน่งใหม่
    func locationManager(_ manager: CLLocationManager,
                        didUpdateLocations locations: [CLLocation]) {
        // locations เป็น array เรียงตามเวลา ค่าล่าสุดอยู่ท้ายสุด
        if let latestLocation = locations.last {
            processLocation(latestLocation)
        }
    }
    
    // เรียกเมื่อสถานะการอนุญาตเปลี่ยนแปลง
    func locationManagerDidChangeAuthorization(_ manager: CLLocationManager) {
        switch manager.authorizationStatus {
        case .authorizedWhenInUse:
            print("อนุญาตขณะใช้งาน")
        case .authorizedAlways:
            print("อนุญาตเสมอ")
        case .denied:
            print("ปฏิเสธ")
        case .restricted:
            print("ถูกจำกัด")
        case .notDetermined:
            print("ยังไม่ตัดสินใจ")
        @unknown default:
            break
        }
    }
    
    // เรียกเมื่อเข้าสู่ region
    func locationManager(_ manager: CLLocationManager,
                        didEnterRegion region: CLRegion) {
        print("เข้าสู่ region: \(region.identifier)")
    }
    
    // เรียกเมื่อออกจาก region
    func locationManager(_ manager: CLLocationManager,
                        didExitRegion region: CLRegion) {
        print("ออกจาก region: \(region.identifier)")
    }
    
    private func processLocation(_ location: CLLocation) {
        let coordinate = location.coordinate
        let accuracy = location.horizontalAccuracy
        let timestamp = location.timestamp
        let altitude = location.altitude
        let speed = location.speed
        
        print("""
        พิกัด: \(coordinate.latitude), \(coordinate.longitude)
        ความแม่นยำ: \(accuracy) เมตร
        ความสูง: \(altitude) เมตร
        ความเร็ว: \(speed) m/s
        เวลา: \(timestamp)
        """)
    }
}
```

---

## 3. การขอสิทธิ์การเข้าถึงตำแหน่ง

### 3.1 การตั้งค่า Info.plist

ก่อนขอสิทธิ์ ต้องเพิ่ม key ใน Info.plist:

```xml
<!-- สำหรับการขอตำแหน่งขณะใช้งาน -->
<key>NSLocationWhenInUseUsageDescription</key>
<string>แอปนี้ต้องการตำแหน่งเพื่อแสดงสถานที่ใกล้เคียง</string>

<!-- สำหรับการขอตำแหน่งเสมอ (รวมถึง background) -->
<key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
<string>แอปนี้ต้องการตำแหน่งเพื่อติดตามเส้นทางของคุณ</string>
```

### 3.2 การขอสิทธิ์

```swift
class LocationPermissionManager: NSObject {
    let locationManager = CLLocationManager()
    
    override init() {
        super.init()
        locationManager.delegate = self
    }
    
    // ขอสิทธิ์ขณะใช้งาน (แนะนำสำหรับ app ส่วนใหญ่)
    func requestWhenInUsePermission() {
        locationManager.requestWhenInUseAuthorization()
    }
    
    // ขอสิทธิ์เสมอ (ต้องการเหตุผลที่ชัดเจน)
    func requestAlwaysPermission() {
        locationManager.requestAlwaysAuthorization()
    }
    
    // ตรวจสอบสถานะปัจจุบัน
    func checkCurrentStatus() -> CLAuthorizationStatus {
        return locationManager.authorizationStatus
    }
}

extension LocationPermissionManager: CLLocationManagerDelegate {
    func locationManagerDidChangeAuthorization(_ manager: CLLocationManager) {
        handleAuthorizationChange(manager.authorizationStatus)
    }
    
    private func handleAuthorizationChange(_ status: CLAuthorizationStatus) {
        switch status {
        case .notDetermined:
            // ยังไม่ได้ตัดสินใจ - อาจขอสิทธิ์
            print("ยังไม่ได้ตัดสินใจ")
            
        case .restricted:
            // ถูกจำกัดโดย policy (เช่น parental controls)
            print("ถูกจำกัดการใช้งาน")
            showRestrictedAlert()
            
        case .denied:
            // ผู้ใช้ปฏิเสธ - แนะนำให้ไปตั้งค่า
            print("ถูกปฏิเสธ")
            showDeniedAlert()
            
        case .authorizedWhenInUse:
            // อนุญาตขณะใช้งาน
            print("อนุญาตขณะใช้งาน")
            startLocationUpdates()
            
        case .authorizedAlways:
            // อนุญาตเสมอ รวมถึง background
            print("อนุญาตเสมอ")
            startLocationUpdates()
            enableBackgroundUpdates()
            
        @unknown default:
            break
        }
    }
    
    private func showDeniedAlert() {
        // แสดง alert แนะนำให้ไปตั้งค่า
        guard let settingsURL = URL(string: UIApplication.openSettingsURLString) else { return }
        // นำทางไปยัง Settings
        UIApplication.shared.open(settingsURL)
    }
    
    private func showRestrictedAlert() {
        print("การใช้งาน Location ถูกจำกัดโดยนโยบาย")
    }
    
    private func startLocationUpdates() {
        locationManager.startUpdatingLocation()
    }
    
    private func enableBackgroundUpdates() {
        locationManager.allowsBackgroundLocationUpdates = true
    }
}
```

---

## 4. ระดับความแม่นยำ (Location Accuracy)

### 4.1 ค่าความแม่นยำต่างๆ

```swift
class AccuracyDemoManager: NSObject {
    let locationManager = CLLocationManager()
    
    func demonstrateAccuracyLevels() {
        // ความแม่นยำสูงสุด - ใช้ GPS มาก (ใช้แบตมาก)
        locationManager.desiredAccuracy = kCLLocationAccuracyBestForNavigation
        
        // ความแม่นยำดีที่สุดทั่วไป (~5 เมตร)
        locationManager.desiredAccuracy = kCLLocationAccuracyBest
        
        // ความแม่นยำระดับเมตร (~10 เมตร)
        locationManager.desiredAccuracy = kCLLocationAccuracyNearestTenMeters
        
        // ความแม่นยำระดับ 100 เมตร
        locationManager.desiredAccuracy = kCLLocationAccuracyHundredMeters
        
        // ความแม่นยำระดับกิโลเมตร
        locationManager.desiredAccuracy = kCLLocationAccuracyKilometer
        
        // ความแม่นยำระดับ 3 กิโลเมตร (ประหยัดแบตมากที่สุด)
        locationManager.desiredAccuracy = kCLLocationAccuracyThreeKilometers
        
        // ลดการใช้แบตด้วย distanceFilter
        // จะอัปเดตเมื่อเคลื่อนที่มากกว่า N เมตร
        locationManager.distanceFilter = 50.0 // 50 เมตร
        
        // ไม่มี filter (อัปเดตทุกครั้ง)
        locationManager.distanceFilter = kCLDistanceFilterNone
    }
    
    // เลือก accuracy ตาม use case
    func setAccuracyForUseCase(_ useCase: LocationUseCase) {
        switch useCase {
        case .navigation:
            locationManager.desiredAccuracy = kCLLocationAccuracyBestForNavigation
            locationManager.distanceFilter = kCLDistanceFilterNone
            
        case .fitness:
            locationManager.desiredAccuracy = kCLLocationAccuracyBest
            locationManager.distanceFilter = 5.0
            
        case .nearbySearch:
            locationManager.desiredAccuracy = kCLLocationAccuracyHundredMeters
            locationManager.distanceFilter = 100.0
            
        case .cityLevel:
            locationManager.desiredAccuracy = kCLLocationAccuracyKilometer
            locationManager.distanceFilter = 500.0
        }
    }
}

enum LocationUseCase {
    case navigation
    case fitness
    case nearbySearch
    case cityLevel
}
```

---

## 5. การรับตำแหน่งปัจจุบัน

### 5.1 Single Location Request (iOS 17+)

```swift
import CoreLocation

class SingleLocationManager {
    // ใช้ CLLocationUpdate สำหรับ iOS 17+
    @available(iOS 17.0, *)
    func getCurrentLocation() async throws -> CLLocation {
        let updates = CLLocationUpdate.liveUpdates()
        
        for try await update in updates {
            if let location = update.location {
                return location
            }
        }
        
        throw LocationError.noLocationAvailable
    }
}

enum LocationError: Error {
    case noLocationAvailable
    case permissionDenied
    case locationServicesDisabled
}
```

### 5.2 Traditional Location Request

```swift
import CoreLocation
import Combine

class TraditionalLocationManager: NSObject, ObservableObject {
    private let locationManager = CLLocationManager()
    
    @Published var currentLocation: CLLocation?
    @Published var authorizationStatus: CLAuthorizationStatus = .notDetermined
    
    private var locationContinuation: CheckedContinuation<CLLocation, Error>?
    
    override init() {
        super.init()
        locationManager.delegate = self
        locationManager.desiredAccuracy = kCLLocationAccuracyBest
    }
    
    func requestLocation() async throws -> CLLocation {
        return try await withCheckedThrowingContinuation { continuation in
            self.locationContinuation = continuation
            
            switch locationManager.authorizationStatus {
            case .notDetermined:
                locationManager.requestWhenInUseAuthorization()
            case .authorizedWhenInUse, .authorizedAlways:
                locationManager.requestLocation()
            case .denied, .restricted:
                continuation.resume(throwing: LocationError.permissionDenied)
            @unknown default:
                break
            }
        }
    }
}

extension TraditionalLocationManager: CLLocationManagerDelegate {
    func locationManager(_ manager: CLLocationManager,
                        didUpdateLocations locations: [CLLocation]) {
        guard let location = locations.last else { return }
        
        currentLocation = location
        locationContinuation?.resume(returning: location)
        locationContinuation = nil
    }
    
    func locationManager(_ manager: CLLocationManager,
                        didFailWithError error: Error) {
        locationContinuation?.resume(throwing: error)
        locationContinuation = nil
    }
    
    func locationManagerDidChangeAuthorization(_ manager: CLLocationManager) {
        authorizationStatus = manager.authorizationStatus
        
        if manager.authorizationStatus == .authorizedWhenInUse ||
           manager.authorizationStatus == .authorizedAlways {
            manager.requestLocation()
        } else if manager.authorizationStatus == .denied {
            locationContinuation?.resume(throwing: LocationError.permissionDenied)
            locationContinuation = nil
        }
    }
}

// การใช้งาน
class ViewController: UIViewController {
    let locationManager = TraditionalLocationManager()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        Task {
            do {
                let location = try await locationManager.requestLocation()
                print("ได้รับตำแหน่ง: \(location.coordinate)")
            } catch {
                print("เกิดข้อผิดพลาด: \(error)")
            }
        }
    }
}
```

---

## 6. การติดตามตำแหน่งอย่างต่อเนื่อง

### 6.1 Continuous Location Updates

```swift
import CoreLocation
import Combine

class ContinuousLocationTracker: NSObject, ObservableObject {
    private let locationManager = CLLocationManager()
    
    @Published var locations: [CLLocation] = []
    @Published var isTracking = false
    @Published var currentSpeed: Double = 0
    @Published var totalDistance: Double = 0
    
    override init() {
        super.init()
        locationManager.delegate = self
        locationManager.desiredAccuracy = kCLLocationAccuracyBest
        locationManager.distanceFilter = 5.0
    }
    
    func startTracking() {
        guard !isTracking else { return }
        
        switch locationManager.authorizationStatus {
        case .authorizedWhenInUse, .authorizedAlways:
            locationManager.startUpdatingLocation()
            isTracking = true
            locations.removeAll()
            totalDistance = 0
        case .notDetermined:
            locationManager.requestWhenInUseAuthorization()
        default:
            break
        }
    }
    
    func stopTracking() {
        locationManager.stopUpdatingLocation()
        isTracking = false
    }
    
    // คำนวณระยะทางรวม
    private func calculateTotalDistance() {
        guard locations.count > 1 else { return }
        
        var distance: Double = 0
        for i in 1..<locations.count {
            distance += locations[i].distance(from: locations[i-1])
        }
        totalDistance = distance
    }
    
    // สร้าง path จาก locations
    func createPath() -> [CLLocationCoordinate2D] {
        return locations.map { $0.coordinate }
    }
}

extension ContinuousLocationTracker: CLLocationManagerDelegate {
    func locationManager(_ manager: CLLocationManager,
                        didUpdateLocations locations: [CLLocation]) {
        // กรองตำแหน่งที่ไม่แม่นยำ
        let accurateLocations = locations.filter { location in
            location.horizontalAccuracy > 0 &&
            location.horizontalAccuracy < 100
        }
        
        self.locations.append(contentsOf: accurateLocations)
        
        if let lastLocation = accurateLocations.last {
            currentSpeed = max(0, lastLocation.speed) * 3.6 // แปลงเป็น km/h
        }
        
        calculateTotalDistance()
    }
    
    func locationManagerDidChangeAuthorization(_ manager: CLLocationManager) {
        if manager.authorizationStatus == .authorizedWhenInUse ||
           manager.authorizationStatus == .authorizedAlways {
            if isTracking {
                locationManager.startUpdatingLocation()
            }
        }
    }
}
```

---

## 7. Significant Location Changes

### 7.1 การใช้ Significant Location Changes

Significant Location Changes เป็นโหมดที่ประหยัดแบตกว่า โดยจะแจ้งเตือนเมื่อมีการเปลี่ยนแปลงตำแหน่งครั้งใหญ่:

```swift
class SignificantLocationManager: NSObject {
    private let locationManager = CLLocationManager()
    
    override init() {
        super.init()
        locationManager.delegate = self
    }
    
    // เริ่มติดตาม significant changes
    func startMonitoring() {
        guard CLLocationManager.significantLocationChangeMonitoringAvailable() else {
            print("Significant location change monitoring ไม่พร้อมใช้งาน")
            return
        }
        
        locationManager.startMonitoringSignificantLocationChanges()
        print("เริ่มติดตาม significant location changes")
    }
    
    // หยุดติดตาม
    func stopMonitoring() {
        locationManager.stopMonitoringSignificantLocationChanges()
        print("หยุดติดตาม significant location changes")
    }
}

extension SignificantLocationManager: CLLocationManagerDelegate {
    func locationManager(_ manager: CLLocationManager,
                        didUpdateLocations locations: [CLLocation]) {
        guard let location = locations.last else { return }
        
        print("Significant location change: \(location.coordinate)")
        
        // ทำงานที่ต้องการเมื่อตำแหน่งเปลี่ยนแปลง
        handleSignificantLocationChange(location)
    }
    
    private func handleSignificantLocationChange(_ location: CLLocation) {
        // บันทึกตำแหน่งหรืออัปเดต server
        print("พิกัดใหม่: (\(location.coordinate.latitude), \(location.coordinate.longitude))")
        
        // ตรวจสอบว่าอยู่ใกล้ home หรือไม่
        let homeCoordinate = CLLocation(latitude: 13.7563, longitude: 100.5018)
        let distance = location.distance(from: homeCoordinate)
        
        if distance < 500 {
            print("อยู่ใกล้บ้าน (\(Int(distance)) เมตร)")
        }
    }
}
```

---

## 8. Background Location Updates

### 8.1 การตั้งค่า Background Modes

ใน Xcode ให้เปิด Project Settings > Signing & Capabilities และเพิ่ม Background Modes โดยเลือก "Location updates"

```swift
class BackgroundLocationManager: NSObject {
    private let locationManager = CLLocationManager()
    
    override init() {
        super.init()
        locationManager.delegate = self
        locationManager.desiredAccuracy = kCLLocationAccuracyBest
        
        // เปิดใช้งาน background updates
        locationManager.allowsBackgroundLocationUpdates = true
        
        // แสดง indicator บน status bar เมื่อใช้งาน background
        locationManager.showsBackgroundLocationIndicator = true
        
        // หยุดอัปเดตเมื่อ app ไม่ได้ใช้ (ประหยัดแบต)
        locationManager.pausesLocationUpdatesAutomatically = false
    }
    
    func startBackgroundTracking() {
        // ต้องมี .authorizedAlways permission
        guard locationManager.authorizationStatus == .authorizedAlways else {
            locationManager.requestAlwaysAuthorization()
            return
        }
        
        locationManager.startUpdatingLocation()
    }
    
    // Activity type ช่วยให้ iOS optimize การใช้แบต
    func configureForActivity(_ activityType: CLActivityType) {
        locationManager.activityType = activityType
        
        switch activityType {
        case .automotiveNavigation:
            locationManager.desiredAccuracy = kCLLocationAccuracyBestForNavigation
            locationManager.distanceFilter = 10
            
        case .fitness:
            locationManager.desiredAccuracy = kCLLocationAccuracyBest
            locationManager.distanceFilter = 5
            
        case .other:
            locationManager.desiredAccuracy = kCLLocationAccuracyHundredMeters
            locationManager.distanceFilter = 50
            
        default:
            break
        }
    }
}

extension BackgroundLocationManager: CLLocationManagerDelegate {
    func locationManager(_ manager: CLLocationManager,
                        didUpdateLocations locations: [CLLocation]) {
        guard let location = locations.last else { return }
        
        // บันทึกตำแหน่งใน background
        saveLocationInBackground(location)
    }
    
    private func saveLocationInBackground(_ location: CLLocation) {
        // บันทึกลง UserDefaults (สำหรับตัวอย่าง)
        let data: [String: Any] = [
            "latitude": location.coordinate.latitude,
            "longitude": location.coordinate.longitude,
            "timestamp": location.timestamp.timeIntervalSince1970
        ]
        
        UserDefaults.standard.set(data, forKey: "lastLocation")
        print("บันทึกตำแหน่ง background: \(location.coordinate)")
    }
}
```

---

## 9. CLGeocoder

### 9.1 Reverse Geocoding (พิกัด → ที่อยู่)

```swift
import CoreLocation

class GeocoderManager {
    private let geocoder = CLGeocoder()
    
    // แปลงพิกัดเป็นที่อยู่
    func reverseGeocode(location: CLLocation) async throws -> [CLPlacemark] {
        return try await geocoder.reverseGeocodeLocation(location)
    }
    
    // รับที่อยู่แบบ readable
    func getAddressString(from location: CLLocation) async -> String {
        do {
            let placemarks = try await geocoder.reverseGeocodeLocation(location)
            
            guard let placemark = placemarks.first else {
                return "ไม่พบที่อยู่"
            }
            
            return formatAddress(from: placemark)
        } catch {
            return "เกิดข้อผิดพลาด: \(error.localizedDescription)"
        }
    }
    
    private func formatAddress(from placemark: CLPlacemark) -> String {
        var addressComponents: [String] = []
        
        if let subThoroughfare = placemark.subThoroughfare {
            addressComponents.append(subThoroughfare)
        }
        if let thoroughfare = placemark.thoroughfare {
            addressComponents.append(thoroughfare)
        }
        if let subLocality = placemark.subLocality {
            addressComponents.append(subLocality)
        }
        if let locality = placemark.locality {
            addressComponents.append(locality)
        }
        if let administrativeArea = placemark.administrativeArea {
            addressComponents.append(administrativeArea)
        }
        if let country = placemark.country {
            addressComponents.append(country)
        }
        if let postalCode = placemark.postalCode {
            addressComponents.append(postalCode)
        }
        
        return addressComponents.joined(separator: ", ")
    }
}
```

### 9.2 Forward Geocoding (ที่อยู่ → พิกัด)

```swift
extension GeocoderManager {
    // แปลงที่อยู่เป็นพิกัด
    func forwardGeocode(addressString: String) async throws -> CLLocation {
        let placemarks = try await geocoder.geocodeAddressString(addressString)
        
        guard let placemark = placemarks.first,
              let location = placemark.location else {
            throw GeocoderError.noResults
        }
        
        return location
    }
    
    // ค้นหาพิกัดพร้อม region hint
    func forwardGeocode(
        addressString: String,
        withinRegion region: CLRegion? = nil
    ) async throws -> [CLPlacemark] {
        return try await geocoder.geocodeAddressString(
            addressString,
            in: region
        )
    }
    
    // ใช้กับ PostalAddress
    func geocodePostalAddress() async {
        // สร้าง address dictionary
        let addressDictionary: [String: Any] = [
            "Street": "123 ถนนสุขุมวิท",
            "City": "กรุงเทพมหานคร",
            "State": "กรุงเทพ",
            "Country": "ประเทศไทย",
            "CountryCode": "TH"
        ]
        
        do {
            let placemarks = try await geocoder.geocodeAddressDictionary(addressDictionary)
            for placemark in placemarks {
                print("พบตำแหน่ง: \(placemark.location?.coordinate ?? CLLocationCoordinate2D())")
            }
        } catch {
            print("เกิดข้อผิดพลาด: \(error)")
        }
    }
}

enum GeocoderError: Error {
    case noResults
    case invalidAddress
}

// ตัวอย่างการใช้งาน
class GeocoderExample {
    let geocoderManager = GeocoderManager()
    
    func demonstrateGeocoding() async {
        // Reverse geocoding
        let bangkokLocation = CLLocation(latitude: 13.7563, longitude: 100.5018)
        let address = await geocoderManager.getAddressString(from: bangkokLocation)
        print("ที่อยู่: \(address)")
        
        // Forward geocoding
        do {
            let location = try await geocoderManager.forwardGeocode(
                addressString: "วัดพระแก้ว กรุงเทพมหานคร"
            )
            print("พิกัด: \(location.coordinate.latitude), \(location.coordinate.longitude)")
        } catch {
            print("ไม่พบที่อยู่: \(error)")
        }
    }
}
```

---

## 10. Region Monitoring

### 10.1 Geographic Region Monitoring

```swift
import CoreLocation

class RegionMonitor: NSObject {
    private let locationManager = CLLocationManager()
    var monitoredRegions: Set<CLRegion> = []
    
    override init() {
        super.init()
        locationManager.delegate = self
    }
    
    // เพิ่ม circular region
    func addCircularRegion(
        center: CLLocationCoordinate2D,
        radius: CLLocationDistance,
        identifier: String
    ) {
        // ตรวจสอบว่า region monitoring พร้อมใช้งาน
        guard CLLocationManager.isMonitoringAvailable(for: CLCircularRegion.self) else {
            print("Region monitoring ไม่พร้อมใช้งาน")
            return
        }
        
        // จำกัด radius ตามที่ device รองรับ
        let maxRadius = locationManager.maximumRegionMonitoringDistance
        let clampedRadius = min(radius, maxRadius)
        
        let region = CLCircularRegion(
            center: center,
            radius: clampedRadius,
            identifier: identifier
        )
        
        // กำหนดว่าจะแจ้งเตือนเมื่อเข้าหรือออก
        region.notifyOnEntry = true
        region.notifyOnExit = true
        
        locationManager.startMonitoring(for: region)
        monitoredRegions.insert(region)
        
        print("เพิ่ม region: \(identifier) (radius: \(clampedRadius) เมตร)")
    }
    
    // ลบ region
    func removeRegion(identifier: String) {
        let regionsToRemove = locationManager.monitoredRegions
            .filter { $0.identifier == identifier }
        
        for region in regionsToRemove {
            locationManager.stopMonitoring(for: region)
        }
        
        monitoredRegions = monitoredRegions.filter { $0.identifier != identifier }
    }
    
    // ตรวจสอบสถานะปัจจุบันของ region
    func checkCurrentState(for region: CLRegion) {
        locationManager.requestState(for: region)
    }
    
    // ดูรายการ regions ที่กำลัง monitor
    func listMonitoredRegions() {
        print("Regions ที่กำลัง monitor:")
        for region in locationManager.monitoredRegions {
            if let circularRegion = region as? CLCircularRegion {
                print("- \(circularRegion.identifier): center(\(circularRegion.center.latitude), \(circularRegion.center.longitude)), radius: \(circularRegion.radius)m")
            }
        }
    }
}

extension RegionMonitor: CLLocationManagerDelegate {
    // เข้าสู่ region
    func locationManager(_ manager: CLLocationManager,
                        didEnterRegion region: CLRegion) {
        print("เข้าสู่ region: \(region.identifier)")
        sendNotification(title: "ยินดีต้อนรับ!", 
                        body: "คุณเข้าสู่ \(region.identifier)")
    }
    
    // ออกจาก region
    func locationManager(_ manager: CLLocationManager,
                        didExitRegion region: CLRegion) {
        print("ออกจาก region: \(region.identifier)")
        sendNotification(title: "ลาก่อน!", 
                        body: "คุณออกจาก \(region.identifier)")
    }
    
    // สถานะ region
    func locationManager(_ manager: CLLocationManager,
                        didDetermineState state: CLRegionState,
                        for region: CLRegion) {
        switch state {
        case .inside:
            print("อยู่ใน region: \(region.identifier)")
        case .outside:
            print("อยู่นอก region: \(region.identifier)")
        case .unknown:
            print("ไม่ทราบสถานะสำหรับ region: \(region.identifier)")
        }
    }
    
    // เกิดข้อผิดพลาดกับ region monitoring
    func locationManager(_ manager: CLLocationManager,
                        monitoringDidFailFor region: CLRegion?,
                        withError error: Error) {
        print("Region monitoring เกิดข้อผิดพลาด: \(error.localizedDescription)")
    }
    
    private func sendNotification(title: String, body: String) {
        // ส่ง local notification
        print("Notification: \(title) - \(body)")
    }
}
```

---

## 11. Beacon Ranging

### 11.1 การตั้งค่า iBeacon

```swift
import CoreLocation

class BeaconRangingManager: NSObject {
    private let locationManager = CLLocationManager()
    
    // UUID ของ beacon ที่ต้องการติดตาม
    private let beaconUUID = UUID(uuidString: "12345678-ABCD-1234-ABCD-123456789ABC")!
    
    override init() {
        super.init()
        locationManager.delegate = self
    }
    
    // เริ่ม range beacon
    func startRanging() {
        guard CLLocationManager.isRangingAvailable() else {
            print("Beacon ranging ไม่พร้อมใช้งาน")
            return
        }
        
        let beaconIdentityConstraint = CLBeaconIdentityConstraint(uuid: beaconUUID)
        locationManager.startRangingBeacons(satisfying: beaconIdentityConstraint)
    }
    
    // เริ่ม range พร้อม major/minor
    func startRangingSpecificBeacon(major: UInt16, minor: UInt16) {
        let constraint = CLBeaconIdentityConstraint(
            uuid: beaconUUID,
            major: major,
            minor: minor
        )
        locationManager.startRangingBeacons(satisfying: constraint)
    }
    
    // หยุด range
    func stopRanging() {
        let constraint = CLBeaconIdentityConstraint(uuid: beaconUUID)
        locationManager.stopRangingBeacons(satisfying: constraint)
    }
}

extension BeaconRangingManager: CLLocationManagerDelegate {
    func locationManager(_ manager: CLLocationManager,
                        didRange beacons: [CLBeacon],
                        satisfying beaconConstraint: CLBeaconIdentityConstraint) {
        for beacon in beacons {
            let proximity = proximityDescription(beacon.proximity)
            print("""
            Beacon พบ:
            UUID: \(beacon.uuid)
            Major: \(beacon.major), Minor: \(beacon.minor)
            ระยะ: \(proximity)
            ความแม่นยำ: \(beacon.accuracy) เมตร
            RSSI: \(beacon.rssi)
            """)
        }
    }
    
    func locationManager(_ manager: CLLocationManager,
                        didFailRangingFor beaconConstraint: CLBeaconIdentityConstraint,
                        error: Error) {
        print("Ranging เกิดข้อผิดพลาด: \(error.localizedDescription)")
    }
    
    private func proximityDescription(_ proximity: CLProximity) -> String {
        switch proximity {
        case .immediate: return "ใกล้มาก (< 0.5 เมตร)"
        case .near: return "ใกล้ (0.5 - 3 เมตร)"
        case .far: return "ไกล (> 3 เมตร)"
        case .unknown: return "ไม่ทราบ"
        @unknown default: return "ไม่ทราบ"
        }
    }
}
```

---

## 12. MapKit Basics

### 12.1 การเพิ่ม MapKit

```swift
import MapKit
```

### 12.2 MKMapView พื้นฐาน

```swift
import UIKit
import MapKit

class BasicMapViewController: UIViewController {
    private var mapView: MKMapView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupMapView()
    }
    
    private func setupMapView() {
        mapView = MKMapView(frame: view.bounds)
        mapView.autoresizingMask = [.flexibleWidth, .flexibleHeight]
        view.addSubview(mapView)
        
        // กำหนด delegate
        mapView.delegate = self
        
        // แสดงตำแหน่งผู้ใช้
        mapView.showsUserLocation = true
        
        // ติดตามผู้ใช้
        mapView.userTrackingMode = .follow
        
        // กำหนด region เริ่มต้น
        setInitialRegion()
    }
    
    private func setInitialRegion() {
        // กรุงเทพมหานคร
        let bangkok = CLLocationCoordinate2D(latitude: 13.7563, longitude: 100.5018)
        
        let region = MKCoordinateRegion(
            center: bangkok,
            latitudinalMeters: 10000,  // 10 กิโลเมตร
            longitudinalMeters: 10000
        )
        
        mapView.setRegion(region, animated: true)
    }
}

extension BasicMapViewController: MKMapViewDelegate {
    // เรียกเมื่อ region เปลี่ยนแปลง
    func mapView(_ mapView: MKMapView, regionDidChangeAnimated animated: Bool) {
        let center = mapView.region.center
        print("แผนที่เปลี่ยนไปที่: \(center.latitude), \(center.longitude)")
    }
    
    // เรียกเมื่อผู้ใช้คลิก annotation
    func mapView(_ mapView: MKMapView, didSelect annotation: MKAnnotation) {
        print("เลือก annotation: \(annotation.title ?? "")")
    }
}
```

---

## 13. Map Annotations

### 13.1 MKAnnotation Protocol

```swift
import MapKit

// สร้าง custom annotation
class PlaceAnnotation: NSObject, MKAnnotation {
    let coordinate: CLLocationCoordinate2D
    let title: String?
    let subtitle: String?
    let category: PlaceCategory
    let rating: Double
    
    init(
        coordinate: CLLocationCoordinate2D,
        title: String,
        subtitle: String,
        category: PlaceCategory,
        rating: Double = 0.0
    ) {
        self.coordinate = coordinate
        self.title = title
        self.subtitle = subtitle
        self.category = category
        self.rating = rating
    }
}

enum PlaceCategory {
    case restaurant
    case hotel
    case attraction
    case hospital
    case school
}
```

### 13.2 MKAnnotationView

```swift
// Custom annotation view
class PlaceAnnotationView: MKAnnotationView {
    static let identifier = "PlaceAnnotationView"
    
    private var categoryImageView: UIImageView!
    private var ratingLabel: UILabel!
    
    override init(annotation: MKAnnotation?, reuseIdentifier: String?) {
        super.init(annotation: annotation, reuseIdentifier: reuseIdentifier)
        setupView()
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
        setupView()
    }
    
    private func setupView() {
        // ปรับขนาด
        frame = CGRect(x: 0, y: 0, width: 44, height: 44)
        
        // สร้าง image view
        categoryImageView = UIImageView(frame: bounds)
        categoryImageView.contentMode = .scaleAspectFit
        addSubview(categoryImageView)
        
        // สร้าง rating label
        ratingLabel = UILabel(frame: CGRect(x: 0, y: 30, width: 44, height: 14))
        ratingLabel.font = .systemFont(ofSize: 10)
        ratingLabel.textAlignment = .center
        ratingLabel.backgroundColor = .white
        ratingLabel.layer.cornerRadius = 4
        ratingLabel.layer.masksToBounds = true
        addSubview(ratingLabel)
        
        // เปิดใช้ callout
        canShowCallout = true
        
        // เพิ่มปุ่มด้านขวา
        let button = UIButton(type: .detailDisclosure)
        rightCalloutAccessoryView = button
    }
    
    override var annotation: MKAnnotation? {
        didSet {
            guard let placeAnnotation = annotation as? PlaceAnnotation else { return }
            updateForAnnotation(placeAnnotation)
        }
    }
    
    private func updateForAnnotation(_ annotation: PlaceAnnotation) {
        // อัปเดต image ตาม category
        categoryImageView.image = imageForCategory(annotation.category)
        
        // อัปเดต rating
        if annotation.rating > 0 {
            ratingLabel.text = String(format: "%.1f ⭐", annotation.rating)
            ratingLabel.isHidden = false
        } else {
            ratingLabel.isHidden = true
        }
    }
    
    private func imageForCategory(_ category: PlaceCategory) -> UIImage? {
        switch category {
        case .restaurant:
            return UIImage(systemName: "fork.knife.circle.fill")
        case .hotel:
            return UIImage(systemName: "building.2.fill")
        case .attraction:
            return UIImage(systemName: "camera.fill")
        case .hospital:
            return UIImage(systemName: "cross.fill")
        case .school:
            return UIImage(systemName: "book.fill")
        }
    }
}
```

### 13.3 MKMarkerAnnotationView

```swift
// ใช้ MKMarkerAnnotationView สำหรับ marker แบบ standard
class CustomMarkerAnnotationView: MKMarkerAnnotationView {
    static let identifier = "CustomMarkerAnnotationView"
    
    override init(annotation: MKAnnotation?, reuseIdentifier: String?) {
        super.init(annotation: annotation, reuseIdentifier: reuseIdentifier)
        setupMarker()
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
        setupMarker()
    }
    
    private func setupMarker() {
        canShowCallout = true
        
        // เพิ่ม accessory views
        let directionsButton = UIButton(type: .system)
        directionsButton.setTitle("นำทาง", for: .normal)
        directionsButton.sizeToFit()
        rightCalloutAccessoryView = directionsButton
        
        // เพิ่ม thumbnail ด้านซ้าย
        let thumbnailView = UIImageView(frame: CGRect(x: 0, y: 0, width: 40, height: 40))
        thumbnailView.contentMode = .scaleAspectFill
        thumbnailView.clipsToBounds = true
        thumbnailView.layer.cornerRadius = 5
        leftCalloutAccessoryView = thumbnailView
    }
    
    override var annotation: MKAnnotation? {
        didSet {
            guard let placeAnnotation = annotation as? PlaceAnnotation else { return }
            
            // กำหนดสีของ marker
            markerTintColor = colorForCategory(placeAnnotation.category)
            
            // กำหนด glyph image
            glyphImage = glyphForCategory(placeAnnotation.category)
        }
    }
    
    private func colorForCategory(_ category: PlaceCategory) -> UIColor {
        switch category {
        case .restaurant: return .orange
        case .hotel: return .blue
        case .attraction: return .purple
        case .hospital: return .red
        case .school: return .green
        }
    }
    
    private func glyphForCategory(_ category: PlaceCategory) -> UIImage? {
        switch category {
        case .restaurant: return UIImage(systemName: "fork.knife")
        case .hotel: return UIImage(systemName: "bed.double")
        case .attraction: return UIImage(systemName: "star")
        case .hospital: return UIImage(systemName: "cross")
        case .school: return UIImage(systemName: "book")
        }
    }
}
```

### 13.4 การเพิ่ม Annotations

```swift
extension BasicMapViewController {
    func addAnnotations() {
        // ลบ annotations เดิม
        mapView.removeAnnotations(mapView.annotations)
        
        // สร้าง annotations
        let places: [(name: String, subtitle: String, lat: Double, lng: Double, category: PlaceCategory, rating: Double)] = [
            ("วัดพระแก้ว", "สถานที่ศักดิ์สิทธิ์", 13.7516, 100.4927, .attraction, 4.8),
            ("ร้านอาหารเจ้อร่อย", "อาหารไทยแท้", 13.7563, 100.5018, .restaurant, 4.5),
            ("โรงพยาบาลศิริราช", "โรงพยาบาลชั้นนำ", 13.7578, 100.4860, .hospital, 4.7),
            ("โรงแรมแมนดาริน โอเรียนเต็ล", "โรงแรมหรู", 13.7226, 100.5138, .hotel, 4.9)
        ]
        
        for place in places {
            let annotation = PlaceAnnotation(
                coordinate: CLLocationCoordinate2D(latitude: place.lat, longitude: place.lng),
                title: place.name,
                subtitle: place.subtitle,
                category: place.category,
                rating: place.rating
            )
            mapView.addAnnotation(annotation)
        }
    }
}

// Delegate method สำหรับ custom annotation view
extension BasicMapViewController: MKMapViewDelegate {
    func mapView(_ mapView: MKMapView, 
                viewFor annotation: MKAnnotation) -> MKAnnotationView? {
        // ข้ามสำหรับ user location
        guard !(annotation is MKUserLocation) else { return nil }
        
        guard let placeAnnotation = annotation as? PlaceAnnotation else { return nil }
        
        // ลองนำ view เดิมมาใช้ซ้ำ
        var annotationView = mapView.dequeueReusableAnnotationView(
            withIdentifier: CustomMarkerAnnotationView.identifier
        ) as? CustomMarkerAnnotationView
        
        if annotationView == nil {
            annotationView = CustomMarkerAnnotationView(
                annotation: placeAnnotation,
                reuseIdentifier: CustomMarkerAnnotationView.identifier
            )
        } else {
            annotationView?.annotation = placeAnnotation
        }
        
        return annotationView
    }
    
    func mapView(_ mapView: MKMapView,
                annotationView view: MKAnnotationView,
                calloutAccessoryControlTapped control: UIControl) {
        guard let annotation = view.annotation as? PlaceAnnotation else { return }
        
        // เปิด Maps app สำหรับนำทาง
        let mapItem = MKMapItem(placemark: MKPlacemark(coordinate: annotation.coordinate))
        mapItem.name = annotation.title ?? ""
        mapItem.openInMaps(launchOptions: [
            MKLaunchOptionsDirectionsModeKey: MKLaunchOptionsDirectionsModeDriving
        ])
    }
}
```

---

## 14. Map Overlays

### 14.1 MKPolyline

```swift
extension BasicMapViewController {
    func addRoutePolyline() {
        // สร้างจุดผ่านของเส้นทาง
        var coordinates = [
            CLLocationCoordinate2D(latitude: 13.7563, longitude: 100.5018),
            CLLocationCoordinate2D(latitude: 13.7500, longitude: 100.5100),
            CLLocationCoordinate2D(latitude: 13.7450, longitude: 100.5150),
            CLLocationCoordinate2D(latitude: 13.7400, longitude: 100.5200)
        ]
        
        // สร้าง polyline
        let polyline = MKPolyline(coordinates: &coordinates, count: coordinates.count)
        polyline.title = "เส้นทาง"
        
        // เพิ่มลงแผนที่
        mapView.addOverlay(polyline, level: .aboveRoads)
    }
    
    func addWalkingPath() {
        var pathPoints = [
            CLLocationCoordinate2D(latitude: 13.7516, longitude: 100.4927),
            CLLocationCoordinate2D(latitude: 13.7520, longitude: 100.4950),
            CLLocationCoordinate2D(latitude: 13.7525, longitude: 100.4975),
            CLLocationCoordinate2D(latitude: 13.7530, longitude: 100.5000)
        ]
        
        let polyline = MKPolyline(coordinates: &pathPoints, count: pathPoints.count)
        polyline.title = "เส้นทางเดิน"
        mapView.addOverlay(polyline)
    }
}

// Renderer สำหรับ polyline
extension BasicMapViewController: MKMapViewDelegate {
    func mapView(_ mapView: MKMapView, rendererFor overlay: MKOverlay) -> MKOverlayRenderer {
        if let polyline = overlay as? MKPolyline {
            let renderer = MKPolylineRenderer(polyline: polyline)
            
            if polyline.title == "เส้นทางเดิน" {
                renderer.strokeColor = .systemGreen
                renderer.lineWidth = 4
                renderer.lineDashPattern = [4, 4] // เส้นประ
            } else {
                renderer.strokeColor = .systemBlue
                renderer.lineWidth = 5
                renderer.alpha = 0.8
            }
            
            return renderer
        }
        
        if let polygon = overlay as? MKPolygon {
            let renderer = MKPolygonRenderer(polygon: polygon)
            renderer.fillColor = UIColor.systemBlue.withAlphaComponent(0.3)
            renderer.strokeColor = .systemBlue
            renderer.lineWidth = 2
            return renderer
        }
        
        if let circle = overlay as? MKCircle {
            let renderer = MKCircleRenderer(circle: circle)
            renderer.fillColor = UIColor.systemRed.withAlphaComponent(0.2)
            renderer.strokeColor = .systemRed
            renderer.lineWidth = 2
            return renderer
        }
        
        return MKOverlayRenderer(overlay: overlay)
    }
}
```

### 14.2 MKPolygon

```swift
extension BasicMapViewController {
    func addPolygon() {
        // สร้างพื้นที่รูปหลายเหลี่ยม (เขตพระนคร กรุงเทพ)
        var coordinates = [
            CLLocationCoordinate2D(latitude: 13.7600, longitude: 100.4900),
            CLLocationCoordinate2D(latitude: 13.7600, longitude: 100.5100),
            CLLocationCoordinate2D(latitude: 13.7400, longitude: 100.5100),
            CLLocationCoordinate2D(latitude: 13.7400, longitude: 100.4900)
        ]
        
        let polygon = MKPolygon(coordinates: &coordinates, count: coordinates.count)
        polygon.title = "พื้นที่เขต"
        mapView.addOverlay(polygon, level: .aboveRoads)
    }
    
    // Polygon พร้อม hole (โพรง)
    func addPolygonWithHole() {
        var outerCoordinates = [
            CLLocationCoordinate2D(latitude: 13.7700, longitude: 100.4800),
            CLLocationCoordinate2D(latitude: 13.7700, longitude: 100.5200),
            CLLocationCoordinate2D(latitude: 13.7300, longitude: 100.5200),
            CLLocationCoordinate2D(latitude: 13.7300, longitude: 100.4800)
        ]
        
        var innerCoordinates = [
            CLLocationCoordinate2D(latitude: 13.7600, longitude: 100.4900),
            CLLocationCoordinate2D(latitude: 13.7600, longitude: 100.5100),
            CLLocationCoordinate2D(latitude: 13.7400, longitude: 100.5100),
            CLLocationCoordinate2D(latitude: 13.7400, longitude: 100.4900)
        ]
        
        let hole = MKPolygon(coordinates: &innerCoordinates, count: innerCoordinates.count)
        let polygon = MKPolygon(
            coordinates: &outerCoordinates,
            count: outerCoordinates.count,
            interiorPolygons: [hole]
        )
        
        mapView.addOverlay(polygon)
    }
}
```

### 14.3 MKCircle

```swift
extension BasicMapViewController {
    func addCircleOverlay(center: CLLocationCoordinate2D, radius: CLLocationDistance) {
        let circle = MKCircle(center: center, radius: radius)
        circle.title = "พื้นที่รัศมี"
        mapView.addOverlay(circle, level: .aboveRoads)
    }
    
    // เพิ่มวงกลมหลายวง
    func addConcentricCircles(center: CLLocationCoordinate2D) {
        let radii: [CLLocationDistance] = [500, 1000, 2000]
        
        for radius in radii {
            let circle = MKCircle(center: center, radius: radius)
            mapView.addOverlay(circle)
        }
    }
}
```

---

## 15. Map Camera

### 15.1 การควบคุม Camera

```swift
extension BasicMapViewController {
    // กำหนด camera แบบ basic
    func setCamera(
        lookingAt coordinate: CLLocationCoordinate2D,
        altitude: CLLocationDistance = 1000,
        pitch: CGFloat = 0,
        heading: CLLocationDirection = 0
    ) {
        let camera = MKMapCamera(
            lookingAtCenter: coordinate,
            fromDistance: altitude,
            pitch: pitch,
            heading: heading
        )
        
        mapView.setCamera(camera, animated: true)
    }
    
    // Animation camera
    func animateCameraToLocation(_ coordinate: CLLocationCoordinate2D) {
        UIView.animate(withDuration: 1.5) {
            self.mapView.camera = MKMapCamera(
                lookingAtCenter: coordinate,
                fromDistance: 500,
                pitch: 45,
                heading: 0
            )
        }
    }
    
    // Camera ที่มอง 3D buildings
    func show3DView(at coordinate: CLLocationCoordinate2D) {
        let camera = MKMapCamera(
            lookingAtCenter: coordinate,
            fromDistance: 300,
            pitch: 60, // องศา pitch (0 = top-down, 90 = horizontal)
            heading: 45  // ทิศทาง (0 = เหนือ)
        )
        
        mapView.setCamera(camera, animated: true)
        
        // เปิดใช้ 3D buildings
        mapView.showsBuildings = true
    }
    
    // Fly-through animation
    func performFlythrough(coordinates: [CLLocationCoordinate2D]) {
        guard !coordinates.isEmpty else { return }
        
        animateToNextCoordinate(coordinates: coordinates, index: 0)
    }
    
    private func animateToNextCoordinate(
        coordinates: [CLLocationCoordinate2D],
        index: Int
    ) {
        guard index < coordinates.count else { return }
        
        let coordinate = coordinates[index]
        let camera = MKMapCamera(
            lookingAtCenter: coordinate,
            fromDistance: 500,
            pitch: 45,
            heading: Double(index) * 45
        )
        
        UIView.animate(
            withDuration: 2.0,
            delay: 0,
            options: .curveEaseInOut
        ) {
            self.mapView.camera = camera
        } completion: { _ in
            self.animateToNextCoordinate(coordinates: coordinates, index: index + 1)
        }
    }
}
```

---

## 16. Map Types

### 16.1 ประเภทของแผนที่

```swift
extension BasicMapViewController {
    func changeMapType(_ type: MKMapType) {
        mapView.mapType = type
    }
    
    func demonstrateMapTypes() {
        // แผนที่ถนนทั่วไป
        mapView.mapType = .standard
        
        // ภาพถ่ายดาวเทียม
        mapView.mapType = .satellite
        
        // แผนที่ 3D hybrid (satellite + roads)
        mapView.mapType = .hybrid
        
        // Satellite สำหรับ flyover
        mapView.mapType = .satelliteFlyover
        
        // Hybrid flyover
        mapView.mapType = .hybridFlyover
        
        // Muted standard (สีจาง)
        mapView.mapType = .mutedStandard
    }
    
    // ตั้งค่าเพิ่มเติม
    func configureMapFeatures() {
        // แสดง compass
        mapView.showsCompass = true
        
        // แสดง scale
        mapView.showsScale = true
        
        // แสดง traffic
        mapView.showsTraffic = true
        
        // แสดง points of interest
        mapView.pointOfInterestFilter = .includingAll
        // หรือกรองเฉพาะบาง category
        mapView.pointOfInterestFilter = MKPointOfInterestFilter(
            including: [.restaurant, .hotel]
        )
        
        // ซ่อน points of interest
        mapView.pointOfInterestFilter = .excludingAll
    }
}
```

---

## 17. SwiftUI Map View

### 17.1 พื้นฐาน Map ใน SwiftUI

```swift
import SwiftUI
import MapKit

struct BasicMapView: View {
    // กำหนด region เริ่มต้น
    @State private var region = MKCoordinateRegion(
        center: CLLocationCoordinate2D(latitude: 13.7563, longitude: 100.5018),
        latitudinalMeters: 10000,
        longitudinalMeters: 10000
    )
    
    var body: some View {
        Map(coordinateRegion: $region)
            .edgesIgnoringSafeArea(.all)
    }
}

// แบบ iOS 17+
@available(iOS 17.0, *)
struct ModernMapView: View {
    @State private var position: MapCameraPosition = .region(
        MKCoordinateRegion(
            center: CLLocationCoordinate2D(latitude: 13.7563, longitude: 100.5018),
            latitudinalMeters: 10000,
            longitudinalMeters: 10000
        )
    )
    
    var body: some View {
        Map(position: $position) {
            // เนื้อหาแผนที่จะอยู่ที่นี่
        }
        .mapStyle(.standard)
    }
}
```

---

## 18. MapCamera และ MapCameraPosition

### 18.1 MapCamera

```swift
@available(iOS 17.0, *)
struct MapCameraView: View {
    @State private var camera = MapCamera(
        centerCoordinate: CLLocationCoordinate2D(latitude: 13.7563, longitude: 100.5018),
        distance: 5000,
        heading: 0,
        pitch: 0
    )
    
    var body: some View {
        Map(initialPosition: .camera(camera))
            .mapStyle(.hybrid(elevation: .realistic))
    }
}
```

### 18.2 MapCameraPosition

```swift
@available(iOS 17.0, *)
struct MapCameraPositionView: View {
    @State private var position: MapCameraPosition = .automatic
    
    let places: [Place] = [
        Place(name: "วัดพระแก้ว", coordinate: CLLocationCoordinate2D(latitude: 13.7516, longitude: 100.4927)),
        Place(name: "เซ็นทรัลเวิลด์", coordinate: CLLocationCoordinate2D(latitude: 13.7468, longitude: 100.5392))
    ]
    
    var body: some View {
        Map(position: $position) {
            ForEach(places) { place in
                Marker(place.name, coordinate: place.coordinate)
            }
        }
        .toolbar {
            ToolbarItem(placement: .bottomBar) {
                HStack {
                    Button("Bangkok") {
                        position = .region(MKCoordinateRegion(
                            center: CLLocationCoordinate2D(latitude: 13.7563, longitude: 100.5018),
                            latitudinalMeters: 20000,
                            longitudinalMeters: 20000
                        ))
                    }
                    
                    Button("My Location") {
                        position = .userLocation(fallback: .automatic)
                    }
                    
                    Button("All Places") {
                        position = .automatic
                    }
                }
            }
        }
    }
}

struct Place: Identifiable {
    let id = UUID()
    let name: String
    let coordinate: CLLocationCoordinate2D
}
```

---

## 19. Map Annotations ใน SwiftUI

### 19.1 Marker และ Annotation

```swift
@available(iOS 17.0, *)
struct AnnotatedMapView: View {
    @State private var position: MapCameraPosition = .region(
        MKCoordinateRegion(
            center: CLLocationCoordinate2D(latitude: 13.7563, longitude: 100.5018),
            latitudinalMeters: 15000,
            longitudinalMeters: 15000
        )
    )
    
    let attractions: [TouristAttraction] = [
        TouristAttraction(
            name: "วัดพระแก้ว",
            coordinate: CLLocationCoordinate2D(latitude: 13.7516, longitude: 100.4927),
            category: "วัด",
            rating: 4.8
        ),
        TouristAttraction(
            name: "ท่าช้าง",
            coordinate: CLLocationCoordinate2D(latitude: 13.7529, longitude: 100.4897),
            category: "ท่าเรือ",
            rating: 4.2
        ),
        TouristAttraction(
            name: "วัดอรุณ",
            coordinate: CLLocationCoordinate2D(latitude: 13.7437, longitude: 100.4888),
            category: "วัด",
            rating: 4.7
        )
    ]
    
    var body: some View {
        Map(position: $position) {
            // Marker แบบง่าย
            Marker("วัดพระแก้ว", 
                   systemImage: "building.columns.fill",
                   coordinate: CLLocationCoordinate2D(latitude: 13.7516, longitude: 100.4927))
                .tint(.orange)
            
            // Annotation แบบ custom
            ForEach(attractions) { attraction in
                Annotation(attraction.name, 
                           coordinate: attraction.coordinate) {
                    AttractionMarkerView(attraction: attraction)
                }
            }
            
            // User location
            UserAnnotation()
        }
        .mapStyle(.standard(pointsOfInterest: .all))
    }
}

struct AttractionMarkerView: View {
    let attraction: TouristAttraction
    @State private var showDetails = false
    
    var body: some View {
        VStack(spacing: 0) {
            Button(action: { showDetails.toggle() }) {
                VStack(spacing: 2) {
                    Image(systemName: iconForCategory(attraction.category))
                        .font(.title2)
                        .foregroundColor(.white)
                        .frame(width: 40, height: 40)
                        .background(colorForCategory(attraction.category))
                        .clipShape(Circle())
                        .shadow(radius: 3)
                    
                    Text(String(format: "%.1f ⭐", attraction.rating))
                        .font(.caption2)
                        .padding(.horizontal, 4)
                        .background(.white)
                        .cornerRadius(4)
                }
            }
            
            if showDetails {
                VStack(alignment: .leading, spacing: 4) {
                    Text(attraction.name)
                        .font(.caption)
                        .fontWeight(.bold)
                    Text(attraction.category)
                        .font(.caption2)
                        .foregroundColor(.secondary)
                }
                .padding(8)
                .background(.white)
                .cornerRadius(8)
                .shadow(radius: 3)
            }
        }
    }
    
    func iconForCategory(_ category: String) -> String {
        switch category {
        case "วัด": return "building.columns.fill"
        case "ท่าเรือ": return "ferry.fill"
        default: return "mappin.fill"
        }
    }
    
    func colorForCategory(_ category: String) -> Color {
        switch category {
        case "วัด": return .orange
        case "ท่าเรือ": return .blue
        default: return .red
        }
    }
}

struct TouristAttraction: Identifiable {
    let id = UUID()
    let name: String
    let coordinate: CLLocationCoordinate2D
    let category: String
    let rating: Double
}
```

---

## 20. Directions และ Routing

### 20.1 MKDirections

```swift
import MapKit

class DirectionsManager {
    // ขอเส้นทางระหว่างสองจุด
    func getDirections(
        from source: CLLocationCoordinate2D,
        to destination: CLLocationCoordinate2D,
        transportType: MKDirectionsTransportType = .automobile
    ) async throws -> MKDirections.Response {
        let sourcePlacemark = MKPlacemark(coordinate: source)
        let destinationPlacemark = MKPlacemark(coordinate: destination)
        
        let sourceMapItem = MKMapItem(placemark: sourcePlacemark)
        let destinationMapItem = MKMapItem(placemark: destinationPlacemark)
        
        let request = MKDirections.Request()
        request.source = sourceMapItem
        request.destination = destinationMapItem
        request.transportType = transportType
        request.requestsAlternateRoutes = true // ขอเส้นทางสำรอง
        
        let directions = MKDirections(request: request)
        return try await directions.calculate()
    }
    
    // แสดงเส้นทางบนแผนที่
    func showRoute(on mapView: MKMapView, response: MKDirections.Response) {
        // ลบเส้นทางเดิม
        let existingOverlays = mapView.overlays.filter { $0 is MKPolyline }
        mapView.removeOverlays(existingOverlays)
        
        // เพิ่มเส้นทางใหม่
        for (index, route) in response.routes.enumerated() {
            mapView.addOverlay(route.polyline, level: .aboveRoads)
            
            if index == 0 {
                // Zoom ไปยังเส้นทางแรก
                mapView.setVisibleMapRect(
                    route.polyline.boundingMapRect,
                    edgePadding: UIEdgeInsets(top: 50, left: 50, bottom: 50, right: 50),
                    animated: true
                )
                
                // แสดงข้อมูลเส้นทาง
                let distanceKm = route.distance / 1000
                let timeMin = route.expectedTravelTime / 60
                print("ระยะทาง: \(String(format: "%.1f", distanceKm)) กม.")
                print("เวลาประมาณ: \(Int(timeMin)) นาที")
                
                // แสดง steps
                for step in route.steps {
                    print("- \(step.instructions)")
                }
            }
        }
    }
}

// SwiftUI Directions Example
@available(iOS 17.0, *)
struct DirectionsMapView: View {
    @State private var position: MapCameraPosition = .automatic
    @State private var route: MKRoute?
    @State private var isLoading = false
    
    let origin = CLLocationCoordinate2D(latitude: 13.7516, longitude: 100.4927)
    let destination = CLLocationCoordinate2D(latitude: 13.7468, longitude: 100.5392)
    
    var body: some View {
        Map(position: $position) {
            Marker("ต้นทาง", coordinate: origin)
                .tint(.green)
            Marker("ปลายทาง", coordinate: destination)
                .tint(.red)
            
            if let route {
                MapPolyline(route)
                    .stroke(.blue, lineWidth: 5)
            }
        }
        .overlay(alignment: .bottom) {
            if isLoading {
                ProgressView("กำลังคำนวณเส้นทาง...")
                    .padding()
                    .background(.regularMaterial)
                    .cornerRadius(10)
                    .padding()
            }
            
            if let route {
                RouteInfoView(route: route)
            }
        }
        .task {
            await calculateRoute()
        }
    }
    
    func calculateRoute() async {
        isLoading = true
        
        let request = MKDirections.Request()
        request.source = MKMapItem(placemark: MKPlacemark(coordinate: origin))
        request.destination = MKMapItem(placemark: MKPlacemark(coordinate: destination))
        request.transportType = .automobile
        
        do {
            let response = try await MKDirections(request: request).calculate()
            route = response.routes.first
        } catch {
            print("Error: \(error)")
        }
        
        isLoading = false
    }
}

struct RouteInfoView: View {
    let route: MKRoute
    
    var body: some View {
        VStack(alignment: .leading, spacing: 4) {
            Text(route.name)
                .font(.headline)
            HStack {
                Label(
                    String(format: "%.1f กม.", route.distance / 1000),
                    systemImage: "arrow.triangle.swap"
                )
                Spacer()
                Label(
                    "\(Int(route.expectedTravelTime / 60)) นาที",
                    systemImage: "clock"
                )
            }
            .font(.subheadline)
            .foregroundColor(.secondary)
        }
        .padding()
        .background(.regularMaterial)
        .cornerRadius(12)
        .padding()
    }
}
```

---

## 21. Local Search

### 21.1 MKLocalSearch

```swift
import MapKit

class LocalSearchManager {
    // ค้นหาสถานที่
    func search(
        query: String,
        region: MKCoordinateRegion
    ) async throws -> [MKMapItem] {
        let request = MKLocalSearch.Request()
        request.naturalLanguageQuery = query
        request.region = region
        
        let search = MKLocalSearch(request: request)
        let response = try await search.start()
        
        return response.mapItems
    }
    
    // ค้นหาตาม category
    @available(iOS 18.0, *)
    func searchByCategory(
        _ category: MKPointOfInterestCategory,
        near coordinate: CLLocationCoordinate2D,
        radius: CLLocationDistance = 5000
    ) async throws -> [MKMapItem] {
        let request = MKLocalSearch.Request()
        request.pointOfInterestFilter = MKPointOfInterestFilter(including: [category])
        request.region = MKCoordinateRegion(
            center: coordinate,
            latitudinalMeters: radius * 2,
            longitudinalMeters: radius * 2
        )
        
        let search = MKLocalSearch(request: request)
        let response = try await search.start()
        
        return response.mapItems
    }
}

// SwiftUI Search Example
@available(iOS 17.0, *)
struct SearchableMapView: View {
    @State private var position: MapCameraPosition = .region(
        MKCoordinateRegion(
            center: CLLocationCoordinate2D(latitude: 13.7563, longitude: 100.5018),
            latitudinalMeters: 10000,
            longitudinalMeters: 10000
        )
    )
    @State private var searchText = ""
    @State private var searchResults: [MKMapItem] = []
    @State private var selectedItem: MKMapItem?
    
    var body: some View {
        Map(position: $position, selection: $selectedItem) {
            ForEach(searchResults, id: \.self) { item in
                Marker(item: item)
                    .tint(.blue)
            }
        }
        .searchable(text: $searchText, prompt: "ค้นหาสถานที่")
        .onSubmit(of: .search) {
            Task { await performSearch() }
        }
        .sheet(item: $selectedItem) { item in
            PlaceDetailView(mapItem: item)
        }
    }
    
    func performSearch() async {
        guard !searchText.isEmpty else { return }
        
        let request = MKLocalSearch.Request()
        request.naturalLanguageQuery = searchText
        
        if case .region(let region) = position {
            request.region = region
        }
        
        do {
            let response = try await MKLocalSearch(request: request).start()
            searchResults = response.mapItems
            
            if !searchResults.isEmpty {
                position = .automatic
            }
        } catch {
            print("Search error: \(error)")
        }
    }
}

struct PlaceDetailView: View {
    let mapItem: MKMapItem
    @Environment(\.dismiss) private var dismiss
    
    var body: some View {
        NavigationView {
            VStack(alignment: .leading, spacing: 16) {
                Text(mapItem.name ?? "ไม่มีชื่อ")
                    .font(.title)
                    .fontWeight(.bold)
                
                if let phone = mapItem.phoneNumber {
                    Label(phone, systemImage: "phone.fill")
                }
                
                if let url = mapItem.url {
                    Link(destination: url) {
                        Label("เยี่ยมชมเว็บไซต์", systemImage: "safari.fill")
                    }
                }
                
                Button(action: openInMaps) {
                    Label("เปิดใน Maps", systemImage: "map.fill")
                        .frame(maxWidth: .infinity)
                        .padding()
                        .background(.blue)
                        .foregroundColor(.white)
                        .cornerRadius(12)
                }
                
                Spacer()
            }
            .padding()
            .navigationTitle("รายละเอียด")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button("ปิด") { dismiss() }
                }
            }
        }
    }
    
    func openInMaps() {
        mapItem.openInMaps(launchOptions: [
            MKLaunchOptionsDirectionsModeKey: MKLaunchOptionsDirectionsModeDriving
        ])
    }
}
```

---

## 22. แบบฝึกหัดพร้อมเฉลย

### แบบฝึกหัดที่ 1: Location Tracker App

สร้าง app ที่ติดตามตำแหน่งและบันทึกเส้นทาง:

```swift
import SwiftUI
import CoreLocation
import MapKit

// MARK: - Location Manager
class TrackingLocationManager: NSObject, ObservableObject {
    private let manager = CLLocationManager()
    
    @Published var authorizationStatus: CLAuthorizationStatus = .notDetermined
    @Published var currentLocation: CLLocation?
    @Published var trackingPath: [CLLocationCoordinate2D] = []
    @Published var isTracking = false
    @Published var totalDistance: Double = 0
    @Published var currentSpeed: Double = 0
    @Published var elapsedTime: TimeInterval = 0
    
    private var locations: [CLLocation] = []
    private var startTime: Date?
    private var timer: Timer?
    
    override init() {
        super.init()
        manager.delegate = self
        manager.desiredAccuracy = kCLLocationAccuracyBest
        manager.distanceFilter = 5
        authorizationStatus = manager.authorizationStatus
    }
    
    func requestPermission() {
        manager.requestWhenInUseAuthorization()
    }
    
    func startTracking() {
        guard authorizationStatus == .authorizedWhenInUse ||
              authorizationStatus == .authorizedAlways else {
            requestPermission()
            return
        }
        
        isTracking = true
        trackingPath.removeAll()
        locations.removeAll()
        totalDistance = 0
        startTime = Date()
        
        manager.startUpdatingLocation()
        
        // Timer สำหรับอัปเดตเวลา
        timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { [weak self] _ in
            guard let self = self, let start = self.startTime else { return }
            self.elapsedTime = Date().timeIntervalSince(start)
        }
    }
    
    func stopTracking() {
        isTracking = false
        manager.stopUpdatingLocation()
        timer?.invalidate()
        timer = nil
    }
    
    var formattedElapsedTime: String {
        let hours = Int(elapsedTime) / 3600
        let minutes = Int(elapsedTime) / 60 % 60
        let seconds = Int(elapsedTime) % 60
        return String(format: "%02d:%02d:%02d", hours, minutes, seconds)
    }
    
    var averageSpeed: Double {
        guard elapsedTime > 0 else { return 0 }
        return (totalDistance / elapsedTime) * 3.6
    }
}

extension TrackingLocationManager: CLLocationManagerDelegate {
    func locationManager(_ manager: CLLocationManager,
                        didUpdateLocations newLocations: [CLLocation]) {
        guard isTracking else { return }
        
        let filtered = newLocations.filter { $0.horizontalAccuracy > 0 && $0.horizontalAccuracy < 50 }
        
        for location in filtered {
            if let last = locations.last {
                totalDistance += location.distance(from: last)
            }
            locations.append(location)
            trackingPath.append(location.coordinate)
            currentLocation = location
            currentSpeed = max(0, location.speed) * 3.6
        }
    }
    
    func locationManagerDidChangeAuthorization(_ manager: CLLocationManager) {
        authorizationStatus = manager.authorizationStatus
    }
}

// MARK: - Tracking View
struct TrackingView: View {
    @StateObject private var locationManager = TrackingLocationManager()
    @State private var mapPosition: MapCameraPosition = .userLocation(fallback: .automatic)
    
    var body: some View {
        ZStack {
            // แผนที่
            if #available(iOS 17.0, *) {
                Map(position: $mapPosition) {
                    UserAnnotation()
                    
                    if locationManager.trackingPath.count > 1 {
                        MapPolyline(coordinates: locationManager.trackingPath)
                            .stroke(.blue, lineWidth: 4)
                    }
                }
                .ignoresSafeArea()
            }
            
            // UI Overlay
            VStack {
                // Stats Panel
                if locationManager.isTracking {
                    StatsPanel(manager: locationManager)
                        .padding()
                        .transition(.move(edge: .top).combined(with: .opacity))
                }
                
                Spacer()
                
                // Control Panel
                ControlPanel(manager: locationManager)
                    .padding()
            }
        }
        .navigationTitle("ติดตามเส้นทาง")
        .onAppear {
            locationManager.requestPermission()
        }
    }
}

struct StatsPanel: View {
    @ObservedObject var manager: TrackingLocationManager
    
    var body: some View {
        HStack {
            StatItem(
                title: "ระยะทาง",
                value: String(format: "%.2f กม.", manager.totalDistance / 1000),
                icon: "arrow.triangle.swap"
            )
            Divider()
            StatItem(
                title: "เวลา",
                value: manager.formattedElapsedTime,
                icon: "clock"
            )
            Divider()
            StatItem(
                title: "ความเร็ว",
                value: String(format: "%.1f กม/ชม.", manager.currentSpeed),
                icon: "speedometer"
            )
        }
        .padding()
        .background(.regularMaterial)
        .cornerRadius(16)
    }
}

struct StatItem: View {
    let title: String
    let value: String
    let icon: String
    
    var body: some View {
        VStack(spacing: 4) {
            Image(systemName: icon)
                .font(.caption)
                .foregroundColor(.blue)
            Text(value)
                .font(.headline)
                .fontWeight(.bold)
            Text(title)
                .font(.caption)
                .foregroundColor(.secondary)
        }
        .frame(maxWidth: .infinity)
    }
}

struct ControlPanel: View {
    @ObservedObject var manager: TrackingLocationManager
    
    var body: some View {
        Button(action: toggleTracking) {
            HStack {
                Image(systemName: manager.isTracking ? "stop.fill" : "play.fill")
                Text(manager.isTracking ? "หยุด" : "เริ่มติดตาม")
                    .fontWeight(.semibold)
            }
            .frame(maxWidth: .infinity)
            .padding()
            .background(manager.isTracking ? Color.red : Color.green)
            .foregroundColor(.white)
            .cornerRadius(16)
        }
    }
    
    func toggleTracking() {
        if manager.isTracking {
            manager.stopTracking()
        } else {
            manager.startTracking()
        }
    }
}
```

---

### แบบฝึกหัดที่ 2: Place Finder App

```swift
import SwiftUI
import MapKit

@available(iOS 17.0, *)
struct PlaceFinderView: View {
    @State private var position: MapCameraPosition = .region(
        MKCoordinateRegion(
            center: CLLocationCoordinate2D(latitude: 13.7563, longitude: 100.5018),
            latitudinalMeters: 5000,
            longitudinalMeters: 5000
        )
    )
    @State private var searchText = ""
    @State private var results: [MKMapItem] = []
    @State private var selectedCategory: POICategory = .restaurant
    @State private var selectedItem: MKMapItem?
    
    enum POICategory: String, CaseIterable {
        case restaurant = "ร้านอาหาร"
        case cafe = "คาเฟ่"
        case hotel = "โรงแรม"
        case hospital = "โรงพยาบาล"
        case atm = "ATM"
        
        var searchQuery: String {
            switch self {
            case .restaurant: return "restaurant"
            case .cafe: return "cafe"
            case .hotel: return "hotel"
            case .hospital: return "hospital"
            case .atm: return "ATM"
            }
        }
        
        var icon: String {
            switch self {
            case .restaurant: return "fork.knife"
            case .cafe: return "cup.and.saucer"
            case .hotel: return "bed.double"
            case .hospital: return "cross.fill"
            case .atm: return "banknote"
            }
        }
        
        var color: Color {
            switch self {
            case .restaurant: return .orange
            case .cafe: return .brown
            case .hotel: return .blue
            case .hospital: return .red
            case .atm: return .green
            }
        }
    }
    
    var body: some View {
        NavigationView {
            ZStack {
                Map(position: $position, selection: $selectedItem) {
                    ForEach(results, id: \.self) { item in
                        Marker(item: item)
                            .tint(selectedCategory.color)
                    }
                    UserAnnotation()
                }
                .ignoresSafeArea()
                
                VStack {
                    // Category Picker
                    ScrollView(.horizontal, showsIndicators: false) {
                        HStack(spacing: 8) {
                            ForEach(POICategory.allCases, id: \.self) { category in
                                CategoryChip(
                                    category: category,
                                    isSelected: selectedCategory == category
                                ) {
                                    selectedCategory = category
                                    Task { await searchByCategory() }
                                }
                            }
                        }
                        .padding(.horizontal)
                    }
                    .padding(.vertical, 8)
                    .background(.regularMaterial)
                    
                    Spacer()
                    
                    // Results count
                    if !results.isEmpty {
                        Text("พบ \(results.count) สถานที่")
                            .font(.caption)
                            .padding(.horizontal, 12)
                            .padding(.vertical, 6)
                            .background(.regularMaterial)
                            .cornerRadius(20)
                            .padding(.bottom)
                    }
                }
            }
            .navigationTitle("ค้นหาสถานที่")
            .navigationBarTitleDisplayMode(.inline)
            .searchable(text: $searchText, prompt: "ค้นหา...")
            .onSubmit(of: .search) {
                Task { await performSearch() }
            }
            .sheet(item: $selectedItem) { item in
                PlaceDetailView(mapItem: item)
                    .presentationDetents([.medium])
            }
        }
        .task {
            await searchByCategory()
        }
    }
    
    func searchByCategory() async {
        let request = MKLocalSearch.Request()
        request.naturalLanguageQuery = selectedCategory.searchQuery
        
        if case .region(let region) = position {
            request.region = region
        }
        
        do {
            let response = try await MKLocalSearch(request: request).start()
            results = response.mapItems
        } catch {
            print("Error: \(error)")
        }
    }
    
    func performSearch() async {
        guard !searchText.isEmpty else { return }
        
        let request = MKLocalSearch.Request()
        request.naturalLanguageQuery = searchText
        
        do {
            let response = try await MKLocalSearch(request: request).start()
            results = response.mapItems
        } catch {
            print("Error: \(error)")
        }
    }
}

struct CategoryChip: View {
    let category: PlaceFinderView.POICategory
    let isSelected: Bool
    let action: () -> Void
    
    var body: some View {
        Button(action: action) {
            HStack(spacing: 4) {
                Image(systemName: category.icon)
                Text(category.rawValue)
                    .font(.caption)
            }
            .padding(.horizontal, 12)
            .padding(.vertical, 6)
            .background(isSelected ? category.color : Color(.systemGray5))
            .foregroundColor(isSelected ? .white : .primary)
            .cornerRadius(20)
        }
    }
}
```

---

## 23. การสร้าง Location-Based App สมบูรณ์

### 23.1 Architecture

```swift
// App Structure
import SwiftUI
import CoreLocation
import MapKit

// MARK: - Models
struct Venue: Identifiable, Codable {
    let id: UUID
    let name: String
    let description: String
    let category: VenueCategory
    let latitude: Double
    let longitude: Double
    let rating: Double
    let address: String
    
    var coordinate: CLLocationCoordinate2D {
        CLLocationCoordinate2D(latitude: latitude, longitude: longitude)
    }
    
    enum VenueCategory: String, Codable, CaseIterable {
        case food = "อาหาร"
        case entertainment = "บันเทิง"
        case shopping = "ช้อปปิ้ง"
        case culture = "วัฒนธรรม"
        case nature = "ธรรมชาติ"
    }
}

// MARK: - ViewModel
@MainActor
class VenueMapViewModel: ObservableObject {
    @Published var venues: [Venue] = []
    @Published var nearbyVenues: [Venue] = []
    @Published var selectedVenue: Venue?
    @Published var userLocation: CLLocation?
    @Published var searchText = ""
    @Published var selectedCategory: Venue.VenueCategory?
    @Published var isLoading = false
    @Published var errorMessage: String?
    
    private let locationManager = TrackingLocationManager()
    
    init() {
        setupSampleData()
        observeLocation()
    }
    
    private func observeLocation() {
        // ติดตามตำแหน่งผู้ใช้
        Task {
            for await location in locationUpdates() {
                userLocation = location
                updateNearbyVenues()
            }
        }
    }
    
    private func locationUpdates() -> AsyncStream<CLLocation> {
        AsyncStream { continuation in
            // Implementation
        }
    }
    
    func updateNearbyVenues() {
        guard let userLocation = userLocation else { return }
        
        nearbyVenues = venues
            .filter { venue in
                let venueLocation = CLLocation(
                    latitude: venue.latitude,
                    longitude: venue.longitude
                )
                return userLocation.distance(from: venueLocation) < 5000 // 5 กม.
            }
            .sorted { v1, v2 in
                let l1 = CLLocation(latitude: v1.latitude, longitude: v1.longitude)
                let l2 = CLLocation(latitude: v2.latitude, longitude: v2.longitude)
                return userLocation.distance(from: l1) < userLocation.distance(from: l2)
            }
    }
    
    var filteredVenues: [Venue] {
        var result = venues
        
        if let category = selectedCategory {
            result = result.filter { $0.category == category }
        }
        
        if !searchText.isEmpty {
            result = result.filter { venue in
                venue.name.localizedCaseInsensitiveContains(searchText) ||
                venue.description.localizedCaseInsensitiveContains(searchText)
            }
        }
        
        return result
    }
    
    func distanceString(to venue: Venue) -> String {
        guard let userLocation = userLocation else { return "" }
        
        let venueLocation = CLLocation(latitude: venue.latitude, longitude: venue.longitude)
        let distance = userLocation.distance(from: venueLocation)
        
        if distance < 1000 {
            return String(format: "%.0f เมตร", distance)
        } else {
            return String(format: "%.1f กม.", distance / 1000)
        }
    }
    
    private func setupSampleData() {
        venues = [
            Venue(id: UUID(), name: "วัดพระแก้ว", description: "วัดที่สำคัญที่สุดในประเทศไทย", category: .culture, latitude: 13.7516, longitude: 100.4927, rating: 4.8, address: "พระบรมมหาราชวัง กรุงเทพมหานคร"),
            Venue(id: UUID(), name: "เซ็นทรัลเวิลด์", description: "ศูนย์การค้าขนาดใหญ่", category: .shopping, latitude: 13.7468, longitude: 100.5392, rating: 4.3, address: "ถนนราชดำริ กรุงเทพมหานคร"),
            Venue(id: UUID(), name: "สวนลุมพินี", description: "สวนสาธารณะในใจกลางเมือง", category: .nature, latitude: 13.7302, longitude: 100.5419, rating: 4.5, address: "ถนนราชดำริ กรุงเทพมหานคร")
        ]
    }
}

// MARK: - Main View
@available(iOS 17.0, *)
struct VenueMapView: View {
    @StateObject private var viewModel = VenueMapViewModel()
    @State private var mapPosition: MapCameraPosition = .region(
        MKCoordinateRegion(
            center: CLLocationCoordinate2D(latitude: 13.7563, longitude: 100.5018),
            latitudinalMeters: 10000,
            longitudinalMeters: 10000
        )
    )
    @State private var showList = false
    
    var body: some View {
        NavigationView {
            ZStack {
                // Map
                Map(position: $mapPosition) {
                    UserAnnotation()
                    
                    ForEach(viewModel.filteredVenues) { venue in
                        Annotation(venue.name, coordinate: venue.coordinate) {
                            VenuePin(venue: venue, isSelected: viewModel.selectedVenue?.id == venue.id)
                                .onTapGesture {
                                    viewModel.selectedVenue = venue
                                }
                        }
                    }
                }
                .ignoresSafeArea()
                
                // Overlay Controls
                VStack {
                    HStack {
                        Spacer()
                        
                        Button(action: { showList.toggle() }) {
                            Image(systemName: showList ? "map.fill" : "list.bullet")
                                .font(.title2)
                                .padding()
                                .background(.regularMaterial)
                                .clipShape(Circle())
                        }
                        .padding()
                    }
                    
                    Spacer()
                    
                    if let venue = viewModel.selectedVenue {
                        VenueCard(venue: venue, distanceString: viewModel.distanceString(to: venue))
                            .padding()
                            .transition(.move(edge: .bottom).combined(with: .opacity))
                    }
                }
            }
            .sheet(isPresented: $showList) {
                VenueListView(viewModel: viewModel)
                    .presentationDetents([.medium, .large])
            }
            .navigationTitle("สำรวจสถานที่")
            .navigationBarTitleDisplayMode(.inline)
            .searchable(text: $viewModel.searchText)
        }
    }
}

struct VenuePin: View {
    let venue: Venue
    let isSelected: Bool
    
    var body: some View {
        VStack(spacing: 0) {
            ZStack {
                Circle()
                    .fill(categoryColor(venue.category))
                    .frame(width: isSelected ? 50 : 40, height: isSelected ? 50 : 40)
                    .shadow(radius: isSelected ? 5 : 2)
                
                Image(systemName: categoryIcon(venue.category))
                    .foregroundColor(.white)
                    .font(isSelected ? .title3 : .body)
            }
            
            Triangle()
                .fill(categoryColor(venue.category))
                .frame(width: 12, height: 6)
        }
        .animation(.spring(), value: isSelected)
    }
    
    func categoryColor(_ category: Venue.VenueCategory) -> Color {
        switch category {
        case .food: return .orange
        case .entertainment: return .purple
        case .shopping: return .pink
        case .culture: return .yellow
        case .nature: return .green
        }
    }
    
    func categoryIcon(_ category: Venue.VenueCategory) -> String {
        switch category {
        case .food: return "fork.knife"
        case .entertainment: return "film"
        case .shopping: return "cart"
        case .culture: return "building.columns"
        case .nature: return "leaf"
        }
    }
}

struct Triangle: Shape {
    func path(in rect: CGRect) -> Path {
        var path = Path()
        path.move(to: CGPoint(x: rect.midX, y: rect.maxY))
        path.addLine(to: CGPoint(x: rect.minX, y: rect.minY))
        path.addLine(to: CGPoint(x: rect.maxX, y: rect.minY))
        path.closeSubpath()
        return path
    }
}

struct VenueCard: View {
    let venue: Venue
    let distanceString: String
    
    var body: some View {
        HStack(spacing: 12) {
            RoundedRectangle(cornerRadius: 10)
                .fill(Color(.systemGray5))
                .frame(width: 60, height: 60)
                .overlay(
                    Image(systemName: "photo")
                        .foregroundColor(.secondary)
                )
            
            VStack(alignment: .leading, spacing: 4) {
                Text(venue.name)
                    .font(.headline)
                
                Text(venue.description)
                    .font(.caption)
                    .foregroundColor(.secondary)
                    .lineLimit(1)
                
                HStack {
                    Label(String(format: "%.1f", venue.rating), systemImage: "star.fill")
                        .font(.caption)
                        .foregroundColor(.yellow)
                    
                    if !distanceString.isEmpty {
                        Text("·")
                            .foregroundColor(.secondary)
                        Label(distanceString, systemImage: "location")
                            .font(.caption)
                            .foregroundColor(.secondary)
                    }
                }
            }
            
            Spacer()
            
            Button(action: {}) {
                Image(systemName: "chevron.right")
                    .foregroundColor(.secondary)
            }
        }
        .padding()
        .background(.regularMaterial)
        .cornerRadius(16)
    }
}

struct VenueListView: View {
    @ObservedObject var viewModel: VenueMapViewModel
    
    var body: some View {
        NavigationView {
            List(viewModel.filteredVenues) { venue in
                VenueRow(venue: venue, distance: viewModel.distanceString(to: venue))
            }
            .navigationTitle("รายการสถานที่")
        }
    }
}

struct VenueRow: View {
    let venue: Venue
    let distance: String
    
    var body: some View {
        HStack {
            VStack(alignment: .leading) {
                Text(venue.name)
                    .font(.headline)
                Text(venue.category.rawValue)
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
            
            Spacer()
            
            VStack(alignment: .trailing) {
                Text(String(format: "%.1f", venue.rating))
                    .font(.subheadline)
                    .fontWeight(.bold)
                if !distance.isEmpty {
                    Text(distance)
                        .font(.caption)
                        .foregroundColor(.secondary)
                }
            }
        }
    }
}
```

---

## 24. สรุป

ในบทนี้เราได้เรียนรู้:

1. **CoreLocation Framework** - การตั้งค่าและใช้งาน CLLocationManager
2. **การขอสิทธิ์** - การขอ location permission ทั้ง WhenInUse และ Always
3. **ระดับความแม่นยำ** - การเลือก accuracy ที่เหมาะสมกับ use case
4. **การรับตำแหน่ง** - ทั้งแบบ one-time และ continuous
5. **Significant Location Changes** - โหมดประหยัดแบต
6. **Background Updates** - การติดตามตำแหน่งใน background
7. **Geocoding** - การแปลงพิกัดและที่อยู่
8. **Region Monitoring** - การติดตามการเข้า-ออกพื้นที่
9. **iBeacon Ranging** - การระบุระยะจาก beacon
10. **MapKit** - การแสดงแผนที่ใน UIKit และ SwiftUI
11. **Annotations** - การสร้าง custom annotation และ marker
12. **Overlays** - Polyline, Polygon, Circle
13. **Camera** - การควบคุมมุมมองแผนที่
14. **Directions** - การขอเส้นทาง
15. **Local Search** - การค้นหาสถานที่

### Tips สำคัญ

- ใช้ accuracy ต่ำที่สุดที่ยังตอบโจทย์ใช้งานได้ เพื่อประหยัดแบต
- ขอสิทธิ์ `WhenInUse` ก่อน เว้นแต่จำเป็นต้องใช้ `Always`
- กรองตำแหน่งที่ `horizontalAccuracy` สูง (ไม่แม่นยำ) ออก
- ใช้ `CLGeocoder` อย่างระมัดระวัง เพราะมี rate limit
- สำหรับ SwiftUI ใช้ Map API ใหม่ (iOS 17+) ที่มีความสามารถมากกว่า
