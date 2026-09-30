# Part 57: HealthKit

## HealthKit คืออะไร

HealthKit เป็น framework ของ Apple ที่ช่วยให้แอปพลิเคชันสามารถเข้าถึงและจัดการข้อมูลสุขภาพของผู้ใช้ได้ HealthKit ถูกเปิดตัวใน iOS 8 และกลายเป็นส่วนสำคัญของ ecosystem สุขภาพของ Apple ซึ่งรวมถึง Apple Watch, iPhone และ iPad

### ความสามารถหลักของ HealthKit

1. **รวบรวมข้อมูลสุขภาพ** - จากแหล่งต่างๆ เช่น Apple Watch, แอปอื่นๆ และอุปกรณ์ Bluetooth
2. **จัดเก็บข้อมูลอย่างปลอดภัย** - ข้อมูลถูกเข้ารหัสและเก็บไว้ในอุปกรณ์
3. **แบ่งปันข้อมูล** - ระหว่างแอปต่างๆ ด้วยการอนุญาตจากผู้ใช้
4. **วิเคราะห์ข้อมูล** - ด้วย queries ที่หลากหลาย

### ประเภทข้อมูลที่รองรับ

- **Quantity Types** - ข้อมูลตัวเลข เช่น จำนวนก้าว, อัตราการเต้นของหัวใจ, น้ำหนัก
- **Category Types** - ข้อมูลหมวดหมู่ เช่น การนอนหลับ, mindfulness
- **Correlation Types** - ข้อมูลที่รวมหลายค่า เช่น ความดันโลหิต
- **Workout Types** - ข้อมูลการออกกำลังกาย
- **Document Types** - เอกสารทางการแพทย์ (FHIR)
- **Clinical Types** - ข้อมูลทางคลินิก

---

## การเตรียมโปรเจกต์สำหรับ HealthKit

### 1. เพิ่ม HealthKit Capability

ใน Xcode:
1. เลือก Target ของโปรเจกต์
2. ไปที่แท็บ "Signing & Capabilities"
3. คลิก "+" และเพิ่ม "HealthKit"

### 2. เพิ่ม Privacy Descriptions ใน Info.plist

```xml
<key>NSHealthShareUsageDescription</key>
<string>แอปนี้ต้องการอ่านข้อมูลสุขภาพของคุณเพื่อแสดงสถิติการออกกำลังกาย</string>
<key>NSHealthUpdateUsageDescription</key>
<string>แอปนี้ต้องการบันทึกข้อมูลสุขภาพของคุณ</string>
```

### 3. Import Framework

```swift
import HealthKit
```

---

## HKHealthStore

`HKHealthStore` คือ central object ที่ใช้ในการโต้ตอบกับ HealthKit database ควรสร้างเพียง instance เดียวในแอป

### การสร้าง HKHealthStore

```swift
import HealthKit

class HealthManager: ObservableObject {
    // Singleton instance
    static let shared = HealthManager()
    
    // HKHealthStore instance
    let healthStore = HKHealthStore()
    
    // ตรวจสอบว่า HealthKit พร้อมใช้งานหรือไม่
    var isHealthDataAvailable: Bool {
        return HKHealthStore.isHealthDataAvailable()
    }
    
    private init() {}
}
```

### การตรวจสอบความพร้อมใช้งาน

```swift
func checkHealthKitAvailability() {
    if HKHealthStore.isHealthDataAvailable() {
        print("HealthKit พร้อมใช้งาน")
    } else {
        print("HealthKit ไม่พร้อมใช้งาน (อาจเป็น iPad รุ่นเก่า)")
    }
}
```

---

## การขอ Authorization

HealthKit ต้องการการอนุญาตจากผู้ใช้ก่อนที่จะเข้าถึงข้อมูลสุขภาพ

### ประเภทของ Authorization

1. **Read (Share)** - อ่านข้อมูลจาก HealthKit store
2. **Write (Update)** - เขียนข้อมูลไปยัง HealthKit store

### การขอ Authorization พื้นฐาน

```swift
import HealthKit

class HealthManager: ObservableObject {
    let healthStore = HKHealthStore()
    
    // กำหนด types ที่ต้องการอ่าน
    var typesToRead: Set<HKSampleType> {
        guard let stepCount = HKQuantityType.quantityType(forIdentifier: .stepCount),
              let heartRate = HKQuantityType.quantityType(forIdentifier: .heartRate),
              let activeCalories = HKQuantityType.quantityType(forIdentifier: .activeEnergyBurned),
              let sleepAnalysis = HKCategoryType.categoryType(forIdentifier: .sleepAnalysis) else {
            return []
        }
        return [stepCount, heartRate, activeCalories, sleepAnalysis]
    }
    
    // กำหนด types ที่ต้องการเขียน
    var typesToWrite: Set<HKSampleType> {
        guard let stepCount = HKQuantityType.quantityType(forIdentifier: .stepCount),
              let workoutType = HKObjectType.workoutType() as? HKSampleType else {
            return []
        }
        return [stepCount, workoutType]
    }
    
    func requestAuthorization() async throws {
        try await healthStore.requestAuthorization(
            toShare: typesToWrite,
            read: typesToRead
        )
    }
}
```

### การขอ Authorization พร้อม Error Handling

```swift
func requestHealthKitAuthorization() {
    // ตรวจสอบว่า HealthKit พร้อมใช้งาน
    guard HKHealthStore.isHealthDataAvailable() else {
        print("HealthKit ไม่พร้อมใช้งานบนอุปกรณ์นี้")
        return
    }
    
    // กำหนด quantity types
    guard let stepType = HKQuantityType.quantityType(forIdentifier: .stepCount),
          let heartRateType = HKQuantityType.quantityType(forIdentifier: .heartRate),
          let weightType = HKQuantityType.quantityType(forIdentifier: .bodyMass),
          let heightType = HKQuantityType.quantityType(forIdentifier: .height) else {
        print("ไม่สามารถสร้าง quantity types ได้")
        return
    }
    
    // กำหนด category types
    guard let sleepType = HKCategoryType.categoryType(forIdentifier: .sleepAnalysis) else {
        print("ไม่สามารถสร้าง category types ได้")
        return
    }
    
    // รวม types ทั้งหมดที่ต้องการอ่าน
    let readTypes: Set<HKObjectType> = [
        stepType, heartRateType, weightType, heightType, sleepType,
        HKObjectType.workoutType()
    ]
    
    // รวม types ทั้งหมดที่ต้องการเขียน
    let writeTypes: Set<HKSampleType> = [
        stepType, weightType, HKObjectType.workoutType()
    ]
    
    // ขอ authorization
    healthStore.requestAuthorization(toShare: writeTypes, read: readTypes) { success, error in
        DispatchQueue.main.async {
            if let error = error {
                print("Authorization Error: \(error.localizedDescription)")
                return
            }
            
            if success {
                print("ได้รับ authorization สำเร็จ")
            } else {
                print("ผู้ใช้ปฏิเสธ authorization")
            }
        }
    }
}
```

### การตรวจสอบสถานะ Authorization

```swift
func checkAuthorizationStatus() {
    guard let stepType = HKQuantityType.quantityType(forIdentifier: .stepCount) else { return }
    
    let status = healthStore.authorizationStatus(for: stepType)
    
    switch status {
    case .notDetermined:
        print("ยังไม่ได้ขอ authorization")
    case .sharingDenied:
        print("ผู้ใช้ปฏิเสธการ share (write)")
    case .sharingAuthorized:
        print("ได้รับอนุญาตให้ share (write)")
    @unknown default:
        print("สถานะไม่ทราบ")
    }
}
```

---

## HKSample Types

HKSample คือ class พื้นฐานสำหรับข้อมูลสุขภาพทั้งหมดที่มี timestamp

### ลำดับชั้น Class

```
HKObject
└── HKSample
    ├── HKQuantitySample  (ข้อมูลตัวเลขพร้อมหน่วย)
    ├── HKCategorySample  (ข้อมูลหมวดหมู่)
    ├── HKCorrelation     (ข้อมูลที่รวมหลายค่า)
    └── HKWorkout         (ข้อมูลการออกกำลังกาย)
```

### HKQuantitySample

```swift
// สร้าง quantity sample สำหรับจำนวนก้าว
func createStepSample() -> HKQuantitySample? {
    guard let stepType = HKQuantityType.quantityType(forIdentifier: .stepCount) else {
        return nil
    }
    
    let quantity = HKQuantity(unit: .count(), doubleValue: 1000)
    let now = Date()
    let startDate = Calendar.current.date(byAdding: .hour, value: -1, to: now)!
    
    let sample = HKQuantitySample(
        type: stepType,
        quantity: quantity,
        start: startDate,
        end: now
    )
    
    return sample
}
```

---

## HKQuantityType

HKQuantityType ใช้สำหรับข้อมูลที่เป็นตัวเลขพร้อมหน่วยวัด

### ประเภทข้อมูลยอดนิยม

```swift
// Activity
let stepCount = HKQuantityType.quantityType(forIdentifier: .stepCount)!
let distanceWalking = HKQuantityType.quantityType(forIdentifier: .distanceWalkingRunning)!
let activeCalories = HKQuantityType.quantityType(forIdentifier: .activeEnergyBurned)!
let basalCalories = HKQuantityType.quantityType(forIdentifier: .basalEnergyBurned)!
let flightsClimbed = HKQuantityType.quantityType(forIdentifier: .flightsClimbed)!

// Heart
let heartRate = HKQuantityType.quantityType(forIdentifier: .heartRate)!
let heartRateVariability = HKQuantityType.quantityType(forIdentifier: .heartRateVariabilitySDNN)!
let restingHeartRate = HKQuantityType.quantityType(forIdentifier: .restingHeartRate)!

// Body Measurements
let bodyMass = HKQuantityType.quantityType(forIdentifier: .bodyMass)!
let height = HKQuantityType.quantityType(forIdentifier: .height)!
let bmi = HKQuantityType.quantityType(forIdentifier: .bodyMassIndex)!
let bodyFatPercentage = HKQuantityType.quantityType(forIdentifier: .bodyFatPercentage)!

// Nutrition
let dietaryCalories = HKQuantityType.quantityType(forIdentifier: .dietaryEnergyConsumed)!
let dietaryProtein = HKQuantityType.quantityType(forIdentifier: .dietaryProtein)!
let dietaryCarbohydrates = HKQuantityType.quantityType(forIdentifier: .dietaryCarbohydrates)!
let dietaryFat = HKQuantityType.quantityType(forIdentifier: .dietaryFatTotal)!
let dietaryWater = HKQuantityType.quantityType(forIdentifier: .dietaryWater)!

// Vitals
let bloodGlucose = HKQuantityType.quantityType(forIdentifier: .bloodGlucose)!
let bloodPressureSystolic = HKQuantityType.quantityType(forIdentifier: .bloodPressureSystolic)!
let bloodPressureDiastolic = HKQuantityType.quantityType(forIdentifier: .bloodPressureDiastolic)!
let oxygenSaturation = HKQuantityType.quantityType(forIdentifier: .oxygenSaturation)!
let bodyTemperature = HKQuantityType.quantityType(forIdentifier: .bodyTemperature)!
let respiratoryRate = HKQuantityType.quantityType(forIdentifier: .respiratoryRate)!

// Sleep
let sleepDuration = HKQuantityType.quantityType(forIdentifier: .appleSleepingWristTemperature)!
```

### การทำงานกับ HKUnit

```swift
// หน่วยพื้นฐาน
let countUnit = HKUnit.count()
let meterUnit = HKUnit.meter()
let kilogramUnit = HKUnit.gramUnit(with: .kilo)
let calorieUnit = HKUnit.kilocalorie()
let beatsPerMinute = HKUnit(from: "count/min")
let percentage = HKUnit.percent()

// หน่วยผสม
let stepsPerMinute = HKUnit.count().unitDivided(by: HKUnit.minute())

// การแปลงหน่วย
let heartRateQuantity = HKQuantity(unit: beatsPerMinute, doubleValue: 72)
if heartRateQuantity.is(compatibleWith: beatsPerMinute) {
    let bpm = heartRateQuantity.doubleValue(for: beatsPerMinute)
    print("Heart Rate: \(bpm) BPM")
}

// หน่วยน้ำหนัก
let weightInKg = HKQuantity(unit: .gramUnit(with: .kilo), doubleValue: 70)
let weightInLbs = weightInKg.doubleValue(for: .pound())
print("Weight: \(weightInLbs) lbs")
```

---

## HKCategoryType

HKCategoryType ใช้สำหรับข้อมูลที่เป็นหมวดหมู่ ไม่ใช่ตัวเลข

### ประเภทหมวดหมู่

```swift
// Sleep Analysis
let sleepAnalysis = HKCategoryType.categoryType(forIdentifier: .sleepAnalysis)!

// Mindfulness
let mindfulSession = HKCategoryType.categoryType(forIdentifier: .mindfulSession)!

// Menstrual Cycle
let menstrualFlow = HKCategoryType.categoryType(forIdentifier: .menstrualFlow)!

// Symptoms
let headache = HKCategoryType.categoryType(forIdentifier: .headache)!
let nausea = HKCategoryType.categoryType(forIdentifier: .nausea)!

// Low Heart Rate
let lowHeartRateEvent = HKCategoryType.categoryType(forIdentifier: .lowHeartRateEvent)!
```

### HKCategoryValueSleepAnalysis

```swift
// ค่าที่ใช้ได้สำหรับ Sleep Analysis
// HKCategoryValueSleepAnalysis.inBed      - อยู่บนเตียง
// HKCategoryValueSleepAnalysis.asleep     - หลับ (deprecated)
// HKCategoryValueSleepAnalysis.awake      - ตื่น
// HKCategoryValueSleepAnalysis.asleepCore - หลับปกติ
// HKCategoryValueSleepAnalysis.asleepDeep - หลับลึก
// HKCategoryValueSleepAnalysis.asleepREM  - หลับ REM

func saveSleepData(startDate: Date, endDate: Date, value: HKCategoryValueSleepAnalysis) {
    guard let sleepType = HKCategoryType.categoryType(forIdentifier: .sleepAnalysis) else { return }
    
    let sample = HKCategorySample(
        type: sleepType,
        value: value.rawValue,
        start: startDate,
        end: endDate
    )
    
    healthStore.save(sample) { success, error in
        if let error = error {
            print("Error saving sleep data: \(error.localizedDescription)")
        } else {
            print("บันทึกข้อมูลการนอนหลับสำเร็จ")
        }
    }
}
```

---

## การอ่านข้อมูลสุขภาพ

### HKSampleQuery - การอ่านข้อมูลพื้นฐาน

```swift
func fetchStepCount(for date: Date, completion: @escaping (Double) -> Void) {
    guard let stepType = HKQuantityType.quantityType(forIdentifier: .stepCount) else {
        completion(0)
        return
    }
    
    // สร้าง predicate สำหรับวันที่ต้องการ
    let calendar = Calendar.current
    let startOfDay = calendar.startOfDay(for: date)
    let endOfDay = calendar.date(byAdding: .day, value: 1, to: startOfDay)!
    
    let predicate = HKQuery.predicateForSamples(
        withStart: startOfDay,
        end: endOfDay,
        options: .strictStartDate
    )
    
    // สร้าง query
    let query = HKSampleQuery(
        sampleType: stepType,
        predicate: predicate,
        limit: HKObjectQueryNoLimit,
        sortDescriptors: [NSSortDescriptor(key: HKSampleSortIdentifierStartDate, ascending: false)]
    ) { _, samples, error in
        if let error = error {
            print("Error fetching steps: \(error.localizedDescription)")
            completion(0)
            return
        }
        
        // คำนวณจำนวนก้าวรวม
        let totalSteps = samples?.compactMap { $0 as? HKQuantitySample }
            .reduce(0.0) { $0 + $1.quantity.doubleValue(for: .count()) } ?? 0
        
        completion(totalSteps)
    }
    
    healthStore.execute(query)
}
```

### HKStatisticsQuery - การคำนวณสถิติ

```swift
func fetchTodayStepCount(completion: @escaping (Double) -> Void) {
    guard let stepType = HKQuantityType.quantityType(forIdentifier: .stepCount) else {
        completion(0)
        return
    }
    
    let calendar = Calendar.current
    let now = Date()
    let startOfDay = calendar.startOfDay(for: now)
    
    let predicate = HKQuery.predicateForSamples(
        withStart: startOfDay,
        end: now,
        options: .strictStartDate
    )
    
    let query = HKStatisticsQuery(
        quantityType: stepType,
        quantitySamplePredicate: predicate,
        options: .cumulativeSum
    ) { _, statistics, error in
        if let error = error {
            print("Error: \(error.localizedDescription)")
            completion(0)
            return
        }
        
        let steps = statistics?.sumQuantity()?.doubleValue(for: .count()) ?? 0
        completion(steps)
    }
    
    healthStore.execute(query)
}
```

### HKStatisticsCollectionQuery - การรวบรวมสถิติตามช่วงเวลา

```swift
func fetchWeeklySteps(completion: @escaping ([Date: Double]) -> Void) {
    guard let stepType = HKQuantityType.quantityType(forIdentifier: .stepCount) else {
        completion([:])
        return
    }
    
    let calendar = Calendar.current
    let now = Date()
    let startOfWeek = calendar.date(byAdding: .day, value: -7, to: now)!
    
    // กำหนด interval เป็น 1 วัน
    var interval = DateComponents()
    interval.day = 1
    
    let anchorDate = calendar.startOfDay(for: startOfWeek)
    
    let query = HKStatisticsCollectionQuery(
        quantityType: stepType,
        quantitySamplePredicate: nil,
        options: .cumulativeSum,
        anchorDate: anchorDate,
        intervalComponents: interval
    )
    
    query.initialResultsHandler = { _, collection, error in
        if let error = error {
            print("Error: \(error.localizedDescription)")
            completion([:])
            return
        }
        
        var stepsByDate: [Date: Double] = [:]
        
        collection?.enumerateStatistics(from: startOfWeek, to: now) { statistics, _ in
            let steps = statistics.sumQuantity()?.doubleValue(for: .count()) ?? 0
            stepsByDate[statistics.startDate] = steps
        }
        
        completion(stepsByDate)
    }
    
    healthStore.execute(query)
}
```

### HKObserverQuery - การติดตามการเปลี่ยนแปลง

```swift
var stepObserverQuery: HKObserverQuery?

func startObservingStepChanges() {
    guard let stepType = HKQuantityType.quantityType(forIdentifier: .stepCount) else { return }
    
    stepObserverQuery = HKObserverQuery(sampleType: stepType, predicate: nil) { [weak self] _, completionHandler, error in
        if let error = error {
            print("Observer Error: \(error.localizedDescription)")
            completionHandler()
            return
        }
        
        // ดึงข้อมูลใหม่เมื่อมีการเปลี่ยนแปลง
        self?.fetchTodayStepCount { steps in
            print("จำนวนก้าวอัปเดต: \(steps)")
        }
        
        completionHandler()
    }
    
    if let query = stepObserverQuery {
        healthStore.execute(query)
    }
}

func stopObservingStepChanges() {
    if let query = stepObserverQuery {
        healthStore.stop(query)
        stepObserverQuery = nil
    }
}
```

### HKAnchoredObjectQuery - การดึงข้อมูลใหม่ตั้งแต่ครั้งล่าสุด

```swift
var anchor: HKQueryAnchor?

func fetchNewStepSamples() {
    guard let stepType = HKQuantityType.quantityType(forIdentifier: .stepCount) else { return }
    
    let query = HKAnchoredObjectQuery(
        type: stepType,
        predicate: nil,
        anchor: anchor,
        limit: HKObjectQueryNoLimit
    ) { [weak self] _, samplesOrNil, deletedObjectsOrNil, newAnchor, errorOrNil in
        guard let samples = samplesOrNil else { return }
        
        // อัปเดต anchor สำหรับการ query ครั้งต่อไป
        self?.anchor = newAnchor
        
        // ประมวลผล samples ใหม่
        let newSteps = samples.compactMap { $0 as? HKQuantitySample }
        print("พบ samples ใหม่: \(newSteps.count)")
        
        // ประมวลผล deleted objects
        if let deleted = deletedObjectsOrNil {
            print("ลบข้อมูล: \(deleted.count) รายการ")
        }
    }
    
    healthStore.execute(query)
}
```

---

## การเขียนข้อมูลสุขภาพ

### การบันทึกจำนวนก้าว

```swift
func saveStepCount(steps: Double, startDate: Date, endDate: Date) async throws {
    guard let stepType = HKQuantityType.quantityType(forIdentifier: .stepCount) else {
        throw HealthKitError.typeNotAvailable
    }
    
    let quantity = HKQuantity(unit: .count(), doubleValue: steps)
    let sample = HKQuantitySample(
        type: stepType,
        quantity: quantity,
        start: startDate,
        end: endDate
    )
    
    try await healthStore.save(sample)
    print("บันทึกจำนวนก้าว \(steps) สำเร็จ")
}

enum HealthKitError: Error {
    case typeNotAvailable
    case notAuthorized
    case saveFailed
}
```

### การบันทึกน้ำหนัก

```swift
func saveBodyWeight(weightInKg: Double) async throws {
    guard let weightType = HKQuantityType.quantityType(forIdentifier: .bodyMass) else {
        throw HealthKitError.typeNotAvailable
    }
    
    let quantity = HKQuantity(unit: .gramUnit(with: .kilo), doubleValue: weightInKg)
    let now = Date()
    
    let sample = HKQuantitySample(
        type: weightType,
        quantity: quantity,
        start: now,
        end: now
    )
    
    try await healthStore.save(sample)
    print("บันทึกน้ำหนัก \(weightInKg) kg สำเร็จ")
}
```

### การบันทึก Mindfulness Session

```swift
func saveMindfulnessSession(startDate: Date, endDate: Date) async throws {
    guard let mindfulType = HKCategoryType.categoryType(forIdentifier: .mindfulSession) else {
        throw HealthKitError.typeNotAvailable
    }
    
    let sample = HKCategorySample(
        type: mindfulType,
        value: HKCategoryValue.notApplicable.rawValue,
        start: startDate,
        end: endDate
    )
    
    try await healthStore.save(sample)
    print("บันทึก Mindfulness Session สำเร็จ")
}
```

### การบันทึกหลาย Samples พร้อมกัน

```swift
func saveMultipleSamples(samples: [HKObject]) async throws {
    try await healthStore.save(samples)
    print("บันทึก \(samples.count) samples สำเร็จ")
}
```

### การลบข้อมูล

```swift
func deleteHealthSample(_ sample: HKObject) async throws {
    try await healthStore.delete(sample)
    print("ลบข้อมูลสำเร็จ")
}

func deleteHealthSamples(_ samples: [HKObject]) async throws {
    try await healthStore.delete(samples)
    print("ลบข้อมูล \(samples.count) รายการสำเร็จ")
}
```

---

## HKWorkout

HKWorkout ใช้สำหรับบันทึกข้อมูลการออกกำลังกาย

### ประเภทการออกกำลังกาย

```swift
// ตัวอย่าง HKWorkoutActivityType ที่นิยม
let walkingType = HKWorkoutActivityType.walking
let runningType = HKWorkoutActivityType.running
let cyclingType = HKWorkoutActivityType.cycling
let swimmingType = HKWorkoutActivityType.swimming
let yogaType = HKWorkoutActivityType.yoga
let hiitType = HKWorkoutActivityType.highIntensityIntervalTraining
let strengthType = HKWorkoutActivityType.traditionalStrengthTraining
let soccerType = HKWorkoutActivityType.soccer
let basketballType = HKWorkoutActivityType.basketball
```

### การบันทึก Workout

```swift
func saveWorkout(
    activityType: HKWorkoutActivityType,
    startDate: Date,
    endDate: Date,
    totalCalories: Double,
    totalDistance: Double?
) async throws {
    
    let caloriesQuantity = HKQuantity(unit: .kilocalorie(), doubleValue: totalCalories)
    
    var distanceQuantity: HKQuantity?
    if let distance = totalDistance {
        distanceQuantity = HKQuantity(unit: .meter(), doubleValue: distance)
    }
    
    let workout = HKWorkout(
        activityType: activityType,
        start: startDate,
        end: endDate,
        duration: endDate.timeIntervalSince(startDate),
        totalEnergyBurned: caloriesQuantity,
        totalDistance: distanceQuantity,
        metadata: nil
    )
    
    try await healthStore.save(workout)
    print("บันทึก Workout สำเร็จ")
}
```

### การอ่านข้อมูล Workout

```swift
func fetchRecentWorkouts(limit: Int = 10, completion: @escaping ([HKWorkout]) -> Void) {
    let workoutType = HKObjectType.workoutType()
    
    let sortDescriptor = NSSortDescriptor(
        key: HKSampleSortIdentifierStartDate,
        ascending: false
    )
    
    let query = HKSampleQuery(
        sampleType: workoutType,
        predicate: nil,
        limit: limit,
        sortDescriptors: [sortDescriptor]
    ) { _, samples, error in
        if let error = error {
            print("Error fetching workouts: \(error.localizedDescription)")
            completion([])
            return
        }
        
        let workouts = samples?.compactMap { $0 as? HKWorkout } ?? []
        completion(workouts)
    }
    
    healthStore.execute(query)
}
```

---

## HKWorkoutSession (watchOS)

HKWorkoutSession ใช้สำหรับการติดตาม workout แบบ live บน Apple Watch

```swift
// หมายเหตุ: HKWorkoutSession ใช้ได้เฉพาะบน watchOS
#if os(watchOS)
import HealthKit

class WorkoutManager: NSObject, ObservableObject {
    var healthStore = HKHealthStore()
    var session: HKWorkoutSession?
    var builder: HKLiveWorkoutBuilder?
    
    @Published var heartRate: Double = 0
    @Published var activeCalories: Double = 0
    @Published var distance: Double = 0
    @Published var isWorkoutActive = false
    
    func startWorkout(activityType: HKWorkoutActivityType) {
        let configuration = HKWorkoutConfiguration()
        configuration.activityType = activityType
        configuration.locationType = .outdoor
        
        do {
            session = try HKWorkoutSession(healthStore: healthStore, configuration: configuration)
            builder = session?.associatedWorkoutBuilder()
            
            session?.delegate = self
            builder?.delegate = self
            builder?.dataSource = HKLiveWorkoutDataSource(
                healthStore: healthStore,
                workoutConfiguration: configuration
            )
            
            let startDate = Date()
            session?.startActivity(with: startDate)
            builder?.beginCollection(withStart: startDate) { success, error in
                if let error = error {
                    print("Error starting collection: \(error)")
                }
            }
            
            isWorkoutActive = true
        } catch {
            print("Error creating workout session: \(error)")
        }
    }
    
    func stopWorkout() {
        session?.end()
        isWorkoutActive = false
        
        builder?.endCollection(withEnd: Date()) { success, error in
            self.builder?.finishWorkout { workout, error in
                if let error = error {
                    print("Error finishing workout: \(error)")
                } else if let workout = workout {
                    print("Workout saved: \(workout)")
                }
            }
        }
    }
    
    func pauseWorkout() {
        session?.pause()
    }
    
    func resumeWorkout() {
        session?.resume()
    }
}

// MARK: - HKWorkoutSessionDelegate
extension WorkoutManager: HKWorkoutSessionDelegate {
    func workoutSession(_ workoutSession: HKWorkoutSession,
                       didChangeTo toState: HKWorkoutSessionState,
                       from fromState: HKWorkoutSessionState,
                       date: Date) {
        DispatchQueue.main.async {
            self.isWorkoutActive = toState == .running
        }
    }
    
    func workoutSession(_ workoutSession: HKWorkoutSession, didFailWithError error: Error) {
        print("Workout session failed: \(error)")
    }
}

// MARK: - HKLiveWorkoutBuilderDelegate
extension WorkoutManager: HKLiveWorkoutBuilderDelegate {
    func workoutBuilder(_ workoutBuilder: HKLiveWorkoutBuilder, didCollectDataOf collectedTypes: Set<HKSampleType>) {
        for type in collectedTypes {
            guard let quantityType = type as? HKQuantityType else { return }
            
            let statistics = workoutBuilder.statistics(for: quantityType)
            
            DispatchQueue.main.async {
                switch quantityType {
                case HKQuantityType.quantityType(forIdentifier: .heartRate)!:
                    self.heartRate = statistics?.mostRecentQuantity()?.doubleValue(for: HKUnit(from: "count/min")) ?? 0
                    
                case HKQuantityType.quantityType(forIdentifier: .activeEnergyBurned)!:
                    self.activeCalories = statistics?.sumQuantity()?.doubleValue(for: .kilocalorie()) ?? 0
                    
                case HKQuantityType.quantityType(forIdentifier: .distanceWalkingRunning)!:
                    self.distance = statistics?.sumQuantity()?.doubleValue(for: .meter()) ?? 0
                    
                default:
                    break
                }
            }
        }
    }
    
    func workoutBuilderDidCollectEvent(_ workoutBuilder: HKLiveWorkoutBuilder) {}
}
#endif
```

---

## Background Delivery

Background delivery ช่วยให้แอปรับการแจ้งเตือนเมื่อข้อมูลใหม่ถูกบันทึกลงใน HealthKit แม้แอปจะอยู่ใน background

### การเปิดใช้งาน Background Delivery

```swift
func enableBackgroundDelivery() {
    guard let stepType = HKQuantityType.quantityType(forIdentifier: .stepCount) else { return }
    
    // ต้องตั้งค่า Observer Query ก่อน
    let observerQuery = HKObserverQuery(
        sampleType: stepType,
        predicate: nil
    ) { _, completionHandler, error in
        if let error = error {
            print("Observer Error: \(error)")
            completionHandler()
            return
        }
        
        // ดึงข้อมูลใหม่
        print("ข้อมูลก้าวใหม่มาถึง!")
        completionHandler()
    }
    
    healthStore.execute(observerQuery)
    
    // เปิดใช้งาน background delivery
    healthStore.enableBackgroundDelivery(
        for: stepType,
        frequency: .immediate
    ) { success, error in
        if let error = error {
            print("Error enabling background delivery: \(error)")
        } else if success {
            print("Background delivery เปิดใช้งานแล้ว")
        }
    }
}

func disableBackgroundDelivery() {
    guard let stepType = HKQuantityType.quantityType(forIdentifier: .stepCount) else { return }
    
    healthStore.disableBackgroundDelivery(for: stepType) { success, error in
        if success {
            print("Background delivery ปิดแล้ว")
        }
    }
}

func disableAllBackgroundDeliveries() {
    healthStore.disableAllBackgroundDelivery { success, error in
        if success {
            print("Background deliveries ทั้งหมดปิดแล้ว")
        }
    }
}
```

### ความถี่ของ Background Delivery

```swift
// HKUpdateFrequency options:
// .immediate  - ทันทีที่มีข้อมูลใหม่
// .hourly     - ทุกชั่วโมง
// .daily      - ทุกวัน
// .weekly     - ทุกสัปดาห์
```

---

## Health Records (FHIR)

HealthKit รองรับ FHIR (Fast Healthcare Interoperability Resources) สำหรับข้อมูลทางคลินิก

### Clinical Record Types

```swift
// ตรวจสอบว่ารองรับ Clinical Records หรือไม่
func checkClinicalRecordSupport() {
    if #available(iOS 12.0, *) {
        guard HKHealthStore.isHealthDataAvailable() else { return }
        
        // ประเภทข้อมูลทางคลินิก
        let allergyType = HKClinicalType.clinicalType(forIdentifier: .allergyRecord)!
        let conditionType = HKClinicalType.clinicalType(forIdentifier: .conditionRecord)!
        let immunizationType = HKClinicalType.clinicalType(forIdentifier: .immunizationRecord)!
        let labResultType = HKClinicalType.clinicalType(forIdentifier: .labResultRecord)!
        let medicationType = HKClinicalType.clinicalType(forIdentifier: .medicationRecord)!
        let procedureType = HKClinicalType.clinicalType(forIdentifier: .procedureRecord)!
        let vitalSignType = HKClinicalType.clinicalType(forIdentifier: .vitalSignRecord)!
        
        let typesToRead: Set<HKObjectType> = [
            allergyType, conditionType, immunizationType,
            labResultType, medicationType, procedureType, vitalSignType
        ]
        
        healthStore.requestAuthorization(toShare: nil, read: typesToRead) { success, error in
            print("Clinical records authorization: \(success)")
        }
    }
}

// การอ่าน Clinical Records
@available(iOS 12.0, *)
func fetchClinicalRecords(type: HKClinicalTypeIdentifier, completion: @escaping ([HKClinicalRecord]) -> Void) {
    guard let clinicalType = HKClinicalType.clinicalType(forIdentifier: type) else {
        completion([])
        return
    }
    
    let query = HKSampleQuery(
        sampleType: clinicalType,
        predicate: nil,
        limit: HKObjectQueryNoLimit,
        sortDescriptors: nil
    ) { _, samples, error in
        let records = samples?.compactMap { $0 as? HKClinicalRecord } ?? []
        completion(records)
    }
    
    healthStore.execute(query)
}
```

---

## HealthKit กับ SwiftUI

### HealthManager Observable Object

```swift
import SwiftUI
import HealthKit

@MainActor
class HealthViewModel: ObservableObject {
    private let healthStore = HKHealthStore()
    
    @Published var todaySteps: Double = 0
    @Published var heartRate: Double = 0
    @Published var activeCalories: Double = 0
    @Published var isAuthorized = false
    @Published var errorMessage: String?
    
    func requestAuthorization() async {
        guard HKHealthStore.isHealthDataAvailable() else {
            errorMessage = "HealthKit ไม่พร้อมใช้งาน"
            return
        }
        
        guard let stepType = HKQuantityType.quantityType(forIdentifier: .stepCount),
              let heartRateType = HKQuantityType.quantityType(forIdentifier: .heartRate),
              let calorieType = HKQuantityType.quantityType(forIdentifier: .activeEnergyBurned) else {
            return
        }
        
        let readTypes: Set<HKObjectType> = [stepType, heartRateType, calorieType]
        
        do {
            try await healthStore.requestAuthorization(toShare: [], read: readTypes)
            isAuthorized = true
            await fetchAllData()
        } catch {
            errorMessage = error.localizedDescription
        }
    }
    
    func fetchAllData() async {
        await withTaskGroup(of: Void.self) { group in
            group.addTask { await self.fetchTodaySteps() }
            group.addTask { await self.fetchLatestHeartRate() }
            group.addTask { await self.fetchTodayCalories() }
        }
    }
    
    private func fetchTodaySteps() async {
        guard let stepType = HKQuantityType.quantityType(forIdentifier: .stepCount) else { return }
        
        let calendar = Calendar.current
        let startOfDay = calendar.startOfDay(for: Date())
        let predicate = HKQuery.predicateForSamples(withStart: startOfDay, end: Date())
        
        return await withCheckedContinuation { continuation in
            let query = HKStatisticsQuery(
                quantityType: stepType,
                quantitySamplePredicate: predicate,
                options: .cumulativeSum
            ) { [weak self] _, result, _ in
                let steps = result?.sumQuantity()?.doubleValue(for: .count()) ?? 0
                Task { @MainActor in
                    self?.todaySteps = steps
                }
                continuation.resume()
            }
            healthStore.execute(query)
        }
    }
    
    private func fetchLatestHeartRate() async {
        guard let heartRateType = HKQuantityType.quantityType(forIdentifier: .heartRate) else { return }
        
        return await withCheckedContinuation { continuation in
            let sortDescriptor = NSSortDescriptor(key: HKSampleSortIdentifierStartDate, ascending: false)
            let query = HKSampleQuery(
                sampleType: heartRateType,
                predicate: nil,
                limit: 1,
                sortDescriptors: [sortDescriptor]
            ) { [weak self] _, samples, _ in
                let bpmUnit = HKUnit(from: "count/min")
                let heartRate = (samples?.first as? HKQuantitySample)?.quantity.doubleValue(for: bpmUnit) ?? 0
                Task { @MainActor in
                    self?.heartRate = heartRate
                }
                continuation.resume()
            }
            healthStore.execute(query)
        }
    }
    
    private func fetchTodayCalories() async {
        guard let calorieType = HKQuantityType.quantityType(forIdentifier: .activeEnergyBurned) else { return }
        
        let calendar = Calendar.current
        let startOfDay = calendar.startOfDay(for: Date())
        let predicate = HKQuery.predicateForSamples(withStart: startOfDay, end: Date())
        
        return await withCheckedContinuation { continuation in
            let query = HKStatisticsQuery(
                quantityType: calorieType,
                quantitySamplePredicate: predicate,
                options: .cumulativeSum
            ) { [weak self] _, result, _ in
                let calories = result?.sumQuantity()?.doubleValue(for: .kilocalorie()) ?? 0
                Task { @MainActor in
                    self?.activeCalories = calories
                }
                continuation.resume()
            }
            healthStore.execute(query)
        }
    }
}
```

### SwiftUI Views สำหรับ HealthKit

```swift
struct HealthDashboardView: View {
    @StateObject private var viewModel = HealthViewModel()
    
    var body: some View {
        NavigationView {
            ScrollView {
                VStack(spacing: 20) {
                    if !viewModel.isAuthorized {
                        AuthorizationRequestView(viewModel: viewModel)
                    } else {
                        HealthMetricsGrid(viewModel: viewModel)
                        StepProgressView(steps: viewModel.todaySteps)
                        HeartRateView(heartRate: viewModel.heartRate)
                        CalorieView(calories: viewModel.activeCalories)
                    }
                }
                .padding()
            }
            .navigationTitle("สุขภาพของฉัน")
            .task {
                await viewModel.requestAuthorization()
            }
            .refreshable {
                await viewModel.fetchAllData()
            }
        }
    }
}

struct AuthorizationRequestView: View {
    @ObservedObject var viewModel: HealthViewModel
    
    var body: some View {
        VStack(spacing: 16) {
            Image(systemName: "heart.fill")
                .font(.system(size: 60))
                .foregroundColor(.red)
            
            Text("ต้องการเข้าถึงข้อมูลสุขภาพ")
                .font(.headline)
            
            Text("แอปนี้ต้องการอนุญาตเพื่อแสดงข้อมูลสุขภาพของคุณ")
                .multilineTextAlignment(.center)
                .foregroundColor(.secondary)
            
            Button("ขอ Authorization") {
                Task {
                    await viewModel.requestAuthorization()
                }
            }
            .buttonStyle(.borderedProminent)
        }
        .padding()
    }
}

struct HealthMetricsGrid: View {
    @ObservedObject var viewModel: HealthViewModel
    
    let columns = [
        GridItem(.flexible()),
        GridItem(.flexible())
    ]
    
    var body: some View {
        LazyVGrid(columns: columns, spacing: 16) {
            HealthMetricCard(
                title: "ก้าว",
                value: "\(Int(viewModel.todaySteps))",
                unit: "steps",
                icon: "figure.walk",
                color: .green
            )
            
            HealthMetricCard(
                title: "หัวใจ",
                value: "\(Int(viewModel.heartRate))",
                unit: "BPM",
                icon: "heart.fill",
                color: .red
            )
            
            HealthMetricCard(
                title: "แคลอรี่",
                value: "\(Int(viewModel.activeCalories))",
                unit: "kcal",
                icon: "flame.fill",
                color: .orange
            )
        }
    }
}

struct HealthMetricCard: View {
    let title: String
    let value: String
    let unit: String
    let icon: String
    let color: Color
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            HStack {
                Image(systemName: icon)
                    .foregroundColor(color)
                Text(title)
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
            
            Text(value)
                .font(.title2)
                .fontWeight(.bold)
            
            Text(unit)
                .font(.caption)
                .foregroundColor(.secondary)
        }
        .padding()
        .background(Color(.systemBackground))
        .cornerRadius(12)
        .shadow(radius: 2)
    }
}

struct StepProgressView: View {
    let steps: Double
    let goal: Double = 10000
    
    var progress: Double {
        min(steps / goal, 1.0)
    }
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            Text("เป้าหมายก้าว")
                .font(.headline)
            
            HStack {
                Text("\(Int(steps))")
                    .font(.title)
                    .fontWeight(.bold)
                    .foregroundColor(.green)
                
                Text("/ \(Int(goal)) steps")
                    .foregroundColor(.secondary)
                
                Spacer()
                
                Text("\(Int(progress * 100))%")
                    .fontWeight(.medium)
            }
            
            ProgressView(value: progress)
                .progressViewStyle(.linear)
                .tint(.green)
        }
        .padding()
        .background(Color(.systemBackground))
        .cornerRadius(12)
        .shadow(radius: 2)
    }
}

struct HeartRateView: View {
    let heartRate: Double
    
    var heartRateZone: String {
        switch heartRate {
        case 0..<60: return "ต่ำ"
        case 60..<100: return "ปกติ"
        case 100..<140: return "เผาผลาญไขมัน"
        case 140..<170: return "คาร์ดิโอ"
        default: return "สูงมาก"
        }
    }
    
    var zoneColor: Color {
        switch heartRate {
        case 0..<60: return .blue
        case 60..<100: return .green
        case 100..<140: return .yellow
        case 140..<170: return .orange
        default: return .red
        }
    }
    
    var body: some View {
        HStack {
            VStack(alignment: .leading, spacing: 4) {
                Text("อัตราการเต้นของหัวใจ")
                    .font(.headline)
                
                HStack(alignment: .bottom, spacing: 4) {
                    Text("\(Int(heartRate))")
                        .font(.title)
                        .fontWeight(.bold)
                        .foregroundColor(zoneColor)
                    
                    Text("BPM")
                        .font(.callout)
                        .foregroundColor(.secondary)
                        .padding(.bottom, 4)
                }
                
                Text("โซน: \(heartRateZone)")
                    .font(.caption)
                    .padding(.horizontal, 8)
                    .padding(.vertical, 4)
                    .background(zoneColor.opacity(0.2))
                    .foregroundColor(zoneColor)
                    .cornerRadius(8)
            }
            
            Spacer()
            
            Image(systemName: "heart.fill")
                .font(.system(size: 40))
                .foregroundColor(zoneColor)
        }
        .padding()
        .background(Color(.systemBackground))
        .cornerRadius(12)
        .shadow(radius: 2)
    }
}

struct CalorieView: View {
    let calories: Double
    
    var body: some View {
        HStack {
            VStack(alignment: .leading, spacing: 4) {
                Text("แคลอรี่ที่เผาผลาญ")
                    .font(.headline)
                
                HStack(alignment: .bottom, spacing: 4) {
                    Text("\(Int(calories))")
                        .font(.title)
                        .fontWeight(.bold)
                        .foregroundColor(.orange)
                    
                    Text("kcal")
                        .font(.callout)
                        .foregroundColor(.secondary)
                        .padding(.bottom, 4)
                }
            }
            
            Spacer()
            
            Image(systemName: "flame.fill")
                .font(.system(size: 40))
                .foregroundColor(.orange)
        }
        .padding()
        .background(Color(.systemBackground))
        .cornerRadius(12)
        .shadow(radius: 2)
    }
}
```

---

## การนับก้าว (Step Counting)

### Complete Step Counter Implementation

```swift
class StepCountManager: ObservableObject {
    private let healthStore = HKHealthStore()
    
    @Published var todaySteps: Int = 0
    @Published var weeklySteps: [DaySteps] = []
    @Published var monthlyAverage: Double = 0
    
    struct DaySteps: Identifiable {
        let id = UUID()
        let date: Date
        let steps: Int
        
        var dayName: String {
            let formatter = DateFormatter()
            formatter.dateFormat = "EEE"
            formatter.locale = Locale(identifier: "th_TH")
            return formatter.string(from: date)
        }
    }
    
    func fetchStepData() async {
        await withTaskGroup(of: Void.self) { group in
            group.addTask { await self.fetchTodaySteps() }
            group.addTask { await self.fetchWeeklySteps() }
            group.addTask { await self.calculateMonthlyAverage() }
        }
    }
    
    @MainActor
    private func fetchTodaySteps() async {
        guard let stepType = HKQuantityType.quantityType(forIdentifier: .stepCount) else { return }
        
        let calendar = Calendar.current
        let startOfDay = calendar.startOfDay(for: Date())
        let predicate = HKQuery.predicateForSamples(withStart: startOfDay, end: Date())
        
        return await withCheckedContinuation { continuation in
            let query = HKStatisticsQuery(
                quantityType: stepType,
                quantitySamplePredicate: predicate,
                options: .cumulativeSum
            ) { _, result, _ in
                let steps = Int(result?.sumQuantity()?.doubleValue(for: .count()) ?? 0)
                Task { @MainActor in
                    self.todaySteps = steps
                }
                continuation.resume()
            }
            healthStore.execute(query)
        }
    }
    
    @MainActor
    private func fetchWeeklySteps() async {
        guard let stepType = HKQuantityType.quantityType(forIdentifier: .stepCount) else { return }
        
        let calendar = Calendar.current
        let endDate = Date()
        let startDate = calendar.date(byAdding: .day, value: -6, to: calendar.startOfDay(for: endDate))!
        
        var intervalComponents = DateComponents()
        intervalComponents.day = 1
        
        let anchorDate = calendar.startOfDay(for: startDate)
        
        return await withCheckedContinuation { continuation in
            let query = HKStatisticsCollectionQuery(
                quantityType: stepType,
                quantitySamplePredicate: nil,
                options: .cumulativeSum,
                anchorDate: anchorDate,
                intervalComponents: intervalComponents
            )
            
            query.initialResultsHandler = { _, collection, _ in
                var steps: [DaySteps] = []
                
                collection?.enumerateStatistics(from: startDate, to: endDate) { stats, _ in
                    let stepCount = Int(stats.sumQuantity()?.doubleValue(for: .count()) ?? 0)
                    steps.append(DaySteps(date: stats.startDate, steps: stepCount))
                }
                
                Task { @MainActor in
                    self.weeklySteps = steps
                }
                continuation.resume()
            }
            
            healthStore.execute(query)
        }
    }
    
    @MainActor
    private func calculateMonthlyAverage() async {
        guard let stepType = HKQuantityType.quantityType(forIdentifier: .stepCount) else { return }
        
        let calendar = Calendar.current
        let endDate = Date()
        let startDate = calendar.date(byAdding: .day, value: -30, to: endDate)!
        
        let predicate = HKQuery.predicateForSamples(withStart: startDate, end: endDate)
        
        var intervalComponents = DateComponents()
        intervalComponents.day = 1
        
        return await withCheckedContinuation { continuation in
            let query = HKStatisticsCollectionQuery(
                quantityType: stepType,
                quantitySamplePredicate: predicate,
                options: .cumulativeSum,
                anchorDate: calendar.startOfDay(for: startDate),
                intervalComponents: intervalComponents
            )
            
            query.initialResultsHandler = { _, collection, _ in
                var totalSteps = 0.0
                var daysWithData = 0
                
                collection?.enumerateStatistics(from: startDate, to: endDate) { stats, _ in
                    if let steps = stats.sumQuantity()?.doubleValue(for: .count()), steps > 0 {
                        totalSteps += steps
                        daysWithData += 1
                    }
                }
                
                let average = daysWithData > 0 ? totalSteps / Double(daysWithData) : 0
                Task { @MainActor in
                    self.monthlyAverage = average
                }
                continuation.resume()
            }
            
            healthStore.execute(query)
        }
    }
}
```

---

## การติดตาม Heart Rate

```swift
class HeartRateMonitor: ObservableObject {
    private let healthStore = HKHealthStore()
    private var heartRateQuery: HKAnchoredObjectQuery?
    
    @Published var currentHeartRate: Double = 0
    @Published var heartRateHistory: [HeartRateSample] = []
    @Published var isMonitoring = false
    
    struct HeartRateSample: Identifiable {
        let id = UUID()
        let date: Date
        let bpm: Double
    }
    
    func startMonitoring() {
        guard let heartRateType = HKQuantityType.quantityType(forIdentifier: .heartRate) else { return }
        
        isMonitoring = true
        
        let predicate = HKQuery.predicateForSamples(
            withStart: Date().addingTimeInterval(-3600), // 1 ชั่วโมงที่แล้ว
            end: nil
        )
        
        heartRateQuery = HKAnchoredObjectQuery(
            type: heartRateType,
            predicate: predicate,
            anchor: nil,
            limit: HKObjectQueryNoLimit
        ) { [weak self] _, samples, _, _, _ in
            self?.processSamples(samples)
        }
        
        heartRateQuery?.updateHandler = { [weak self] _, samples, _, _, _ in
            self?.processSamples(samples)
        }
        
        if let query = heartRateQuery {
            healthStore.execute(query)
        }
    }
    
    func stopMonitoring() {
        if let query = heartRateQuery {
            healthStore.stop(query)
            heartRateQuery = nil
        }
        isMonitoring = false
    }
    
    private func processSamples(_ samples: [HKSample]?) {
        guard let samples = samples as? [HKQuantitySample] else { return }
        
        let bpmUnit = HKUnit(from: "count/min")
        let newSamples = samples.map { sample in
            HeartRateSample(
                date: sample.startDate,
                bpm: sample.quantity.doubleValue(for: bpmUnit)
            )
        }
        
        DispatchQueue.main.async { [weak self] in
            self?.heartRateHistory.append(contentsOf: newSamples)
            self?.heartRateHistory.sort { $0.date < $1.date }
            self?.currentHeartRate = newSamples.last?.bpm ?? self?.currentHeartRate ?? 0
        }
    }
    
    func fetchHeartRateZones() -> HeartRateZone {
        return HeartRateZone(bpm: currentHeartRate)
    }
    
    struct HeartRateZone {
        let bpm: Double
        
        var name: String {
            switch bpm {
            case 0..<50: return "โซน 1 - พักฟื้น"
            case 50..<60: return "โซน 2 - ออกกำลังกายเบา"
            case 60..<70: return "โซน 3 - เผาผลาญไขมัน"
            case 70..<80: return "โซน 4 - คาร์ดิโอ"
            case 80..<90: return "โซน 5 - หนักมาก"
            default: return "โซน 6 - สูงสุด"
            }
        }
        
        var color: String {
            switch bpm {
            case 0..<50: return "blue"
            case 50..<60: return "green"
            case 60..<70: return "yellow"
            case 70..<80: return "orange"
            default: return "red"
            }
        }
    }
}
```

---

## การติดตามการนอนหลับ

```swift
class SleepTracker: ObservableObject {
    private let healthStore = HKHealthStore()
    
    @Published var sleepSessions: [SleepSession] = []
    @Published var lastNightSummary: SleepSummary?
    
    struct SleepSession: Identifiable {
        let id = UUID()
        let startDate: Date
        let endDate: Date
        let stage: SleepStage
        
        var duration: TimeInterval {
            endDate.timeIntervalSince(startDate)
        }
    }
    
    enum SleepStage: String, CaseIterable {
        case awake = "ตื่น"
        case core = "หลับปกติ"
        case deep = "หลับลึก"
        case rem = "REM"
        case inBed = "บนเตียง"
        
        var color: String {
            switch self {
            case .awake: return "red"
            case .core: return "blue"
            case .deep: return "purple"
            case .rem: return "cyan"
            case .inBed: return "gray"
            }
        }
    }
    
    struct SleepSummary {
        let totalDuration: TimeInterval
        let deepSleepDuration: TimeInterval
        let remDuration: TimeInterval
        let awakeDuration: TimeInterval
        let sleepScore: Int
        
        var formattedDuration: String {
            let hours = Int(totalDuration) / 3600
            let minutes = (Int(totalDuration) % 3600) / 60
            return "\(hours) ชั่วโมง \(minutes) นาที"
        }
    }
    
    func fetchLastNightSleep() async {
        guard let sleepType = HKCategoryType.categoryType(forIdentifier: .sleepAnalysis) else { return }
        
        let calendar = Calendar.current
        let now = Date()
        let startDate = calendar.date(byAdding: .hour, value: -24, to: now)!
        
        let predicate = HKQuery.predicateForSamples(withStart: startDate, end: now)
        let sortDescriptor = NSSortDescriptor(key: HKSampleSortIdentifierStartDate, ascending: true)
        
        return await withCheckedContinuation { continuation in
            let query = HKSampleQuery(
                sampleType: sleepType,
                predicate: predicate,
                limit: HKObjectQueryNoLimit,
                sortDescriptors: [sortDescriptor]
            ) { [weak self] _, samples, _ in
                guard let samples = samples as? [HKCategorySample] else {
                    continuation.resume()
                    return
                }
                
                let sessions = samples.compactMap { sample -> SleepSession? in
                    let stage: SleepStage
                    switch HKCategoryValueSleepAnalysis(rawValue: sample.value) {
                    case .awake: stage = .awake
                    case .asleepCore: stage = .core
                    case .asleepDeep: stage = .deep
                    case .asleepREM: stage = .rem
                    case .inBed: stage = .inBed
                    default: return nil
                    }
                    
                    return SleepSession(
                        startDate: sample.startDate,
                        endDate: sample.endDate,
                        stage: stage
                    )
                }
                
                let summary = self?.calculateSummary(from: sessions)
                
                Task { @MainActor in
                    self?.sleepSessions = sessions
                    self?.lastNightSummary = summary
                }
                continuation.resume()
            }
            
            healthStore.execute(query)
        }
    }
    
    private func calculateSummary(from sessions: [SleepSession]) -> SleepSummary {
        var totalDuration: TimeInterval = 0
        var deepDuration: TimeInterval = 0
        var remDuration: TimeInterval = 0
        var awakeDuration: TimeInterval = 0
        
        for session in sessions {
            switch session.stage {
            case .core:
                totalDuration += session.duration
            case .deep:
                totalDuration += session.duration
                deepDuration += session.duration
            case .rem:
                totalDuration += session.duration
                remDuration += session.duration
            case .awake:
                awakeDuration += session.duration
            case .inBed:
                break
            }
        }
        
        // คำนวณ Sleep Score (0-100)
        let idealSleep: TimeInterval = 8 * 3600 // 8 ชั่วโมง
        let durationScore = min(totalDuration / idealSleep, 1.0) * 40
        
        let idealDeep = 0.15 // 15% ของการนอนทั้งหมด
        let deepScore = totalDuration > 0 ? min(deepDuration / totalDuration / idealDeep, 1.0) * 30 : 0
        
        let idealREM = 0.20 // 20% ของการนอนทั้งหมด
        let remScore = totalDuration > 0 ? min(remDuration / totalDuration / idealREM, 1.0) * 30 : 0
        
        let sleepScore = Int(durationScore + deepScore + remScore)
        
        return SleepSummary(
            totalDuration: totalDuration,
            deepSleepDuration: deepDuration,
            remDuration: remDuration,
            awakeDuration: awakeDuration,
            sleepScore: sleepScore
        )
    }
}
```

---

## ข้อควรระวังด้านความเป็นส่วนตัว

### Privacy Best Practices

1. **ขอเฉพาะสิ่งที่จำเป็น** - อย่าขอ authorization สำหรับข้อมูลที่ไม่ได้ใช้
2. **อธิบายเหตุผลชัดเจน** - ใน Info.plist ต้องอธิบายว่าทำไมถึงต้องการข้อมูลนั้น
3. **ไม่ share ข้อมูลโดยไม่ได้รับอนุญาต** - ข้อมูลสุขภาพเป็นข้อมูลส่วนตัวสูง
4. **ใช้ข้อมูลตามที่ระบุ** - ใช้ข้อมูลเฉพาะตามวัตถุประสงค์ที่บอกผู้ใช้ไว้

```swift
// ตัวอย่าง Privacy Notice View
struct PrivacyNoticeView: View {
    let onAccept: () -> Void
    let onDecline: () -> Void
    
    var body: some View {
        ScrollView {
            VStack(alignment: .leading, spacing: 16) {
                Text("นโยบายความเป็นส่วนตัว")
                    .font(.title2)
                    .fontWeight(.bold)
                
                Group {
                    Text("ข้อมูลที่เราเข้าถึง:")
                        .fontWeight(.semibold)
                    
                    BulletPoint("จำนวนก้าวเดินประจำวัน")
                    BulletPoint("อัตราการเต้นของหัวใจ")
                    BulletPoint("แคลอรี่ที่เผาผลาญ")
                    BulletPoint("ข้อมูลการออกกำลังกาย")
                }
                
                Group {
                    Text("การใช้ข้อมูล:")
                        .fontWeight(.semibold)
                    
                    Text("เราใช้ข้อมูลสุขภาพของคุณเพื่อ:")
                    BulletPoint("แสดงสถิติสุขภาพประจำวัน")
                    BulletPoint("ติดตามความคืบหน้าการออกกำลังกาย")
                    BulletPoint("ให้คำแนะนำส่วนตัว")
                }
                
                Group {
                    Text("การรักษาความปลอดภัย:")
                        .fontWeight(.semibold)
                    
                    Text("ข้อมูลทั้งหมดถูกเก็บไว้บนอุปกรณ์ของคุณ ไม่มีการส่งข้อมูลไปยังเซิร์ฟเวอร์ภายนอก")
                }
                
                HStack(spacing: 16) {
                    Button("ปฏิเสธ") {
                        onDecline()
                    }
                    .buttonStyle(.bordered)
                    
                    Button("ยอมรับ") {
                        onAccept()
                    }
                    .buttonStyle(.borderedProminent)
                }
                .frame(maxWidth: .infinity)
            }
            .padding()
        }
    }
}

struct BulletPoint: View {
    let text: String
    
    init(_ text: String) {
        self.text = text
    }
    
    var body: some View {
        HStack(alignment: .top, spacing: 8) {
            Text("•")
            Text(text)
        }
        .foregroundColor(.secondary)
    }
}
```

---

## แบบฝึกหัดปฏิบัติ

### แบบฝึกหัดที่ 1: สร้าง Health Dashboard

**โจทย์:** สร้างหน้า dashboard ที่แสดงข้อมูลสุขภาพประจำวัน

```swift
// Solution
struct HealthDashboardExercise: View {
    @StateObject private var manager = ExerciseHealthManager()
    
    var body: some View {
        NavigationStack {
            Group {
                if manager.isLoading {
                    ProgressView("กำลังโหลด...")
                } else if let error = manager.errorMessage {
                    ErrorView(message: error) {
                        Task { await manager.refresh() }
                    }
                } else {
                    DashboardContent(manager: manager)
                }
            }
            .navigationTitle("แดชบอร์ดสุขภาพ")
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button(action: { Task { await manager.refresh() } }) {
                        Image(systemName: "arrow.clockwise")
                    }
                }
            }
        }
        .task {
            await manager.initialize()
        }
    }
}

@MainActor
class ExerciseHealthManager: ObservableObject {
    private let healthStore = HKHealthStore()
    
    @Published var steps: Int = 0
    @Published var heartRate: Double = 0
    @Published var calories: Double = 0
    @Published var distance: Double = 0
    @Published var isLoading = false
    @Published var errorMessage: String?
    
    func initialize() async {
        isLoading = true
        defer { isLoading = false }
        
        do {
            try await requestAuthorization()
            await refresh()
        } catch {
            errorMessage = error.localizedDescription
        }
    }
    
    private func requestAuthorization() async throws {
        guard HKHealthStore.isHealthDataAvailable() else {
            throw NSError(domain: "HealthKit", code: -1, userInfo: [NSLocalizedDescriptionKey: "HealthKit ไม่พร้อมใช้งาน"])
        }
        
        let types: Set<HKObjectType> = [
            HKQuantityType.quantityType(forIdentifier: .stepCount)!,
            HKQuantityType.quantityType(forIdentifier: .heartRate)!,
            HKQuantityType.quantityType(forIdentifier: .activeEnergyBurned)!,
            HKQuantityType.quantityType(forIdentifier: .distanceWalkingRunning)!
        ]
        
        try await healthStore.requestAuthorization(toShare: [], read: types)
    }
    
    func refresh() async {
        await withTaskGroup(of: Void.self) { group in
            group.addTask { await self.loadSteps() }
            group.addTask { await self.loadHeartRate() }
            group.addTask { await self.loadCalories() }
            group.addTask { await self.loadDistance() }
        }
    }
    
    private func loadSteps() async {
        guard let type = HKQuantityType.quantityType(forIdentifier: .stepCount) else { return }
        let calendar = Calendar.current
        let start = calendar.startOfDay(for: Date())
        let predicate = HKQuery.predicateForSamples(withStart: start, end: Date())
        
        steps = await withCheckedContinuation { cont in
            let q = HKStatisticsQuery(quantityType: type, quantitySamplePredicate: predicate, options: .cumulativeSum) { _, r, _ in
                cont.resume(returning: Int(r?.sumQuantity()?.doubleValue(for: .count()) ?? 0))
            }
            healthStore.execute(q)
        }
    }
    
    private func loadHeartRate() async {
        guard let type = HKQuantityType.quantityType(forIdentifier: .heartRate) else { return }
        let sort = NSSortDescriptor(key: HKSampleSortIdentifierStartDate, ascending: false)
        
        heartRate = await withCheckedContinuation { cont in
            let q = HKSampleQuery(sampleType: type, predicate: nil, limit: 1, sortDescriptors: [sort]) { _, s, _ in
                let bpm = (s?.first as? HKQuantitySample)?.quantity.doubleValue(for: HKUnit(from: "count/min")) ?? 0
                cont.resume(returning: bpm)
            }
            healthStore.execute(q)
        }
    }
    
    private func loadCalories() async {
        guard let type = HKQuantityType.quantityType(forIdentifier: .activeEnergyBurned) else { return }
        let calendar = Calendar.current
        let start = calendar.startOfDay(for: Date())
        let predicate = HKQuery.predicateForSamples(withStart: start, end: Date())
        
        calories = await withCheckedContinuation { cont in
            let q = HKStatisticsQuery(quantityType: type, quantitySamplePredicate: predicate, options: .cumulativeSum) { _, r, _ in
                cont.resume(returning: r?.sumQuantity()?.doubleValue(for: .kilocalorie()) ?? 0)
            }
            healthStore.execute(q)
        }
    }
    
    private func loadDistance() async {
        guard let type = HKQuantityType.quantityType(forIdentifier: .distanceWalkingRunning) else { return }
        let calendar = Calendar.current
        let start = calendar.startOfDay(for: Date())
        let predicate = HKQuery.predicateForSamples(withStart: start, end: Date())
        
        distance = await withCheckedContinuation { cont in
            let q = HKStatisticsQuery(quantityType: type, quantitySamplePredicate: predicate, options: .cumulativeSum) { _, r, _ in
                cont.resume(returning: r?.sumQuantity()?.doubleValue(for: .meter()) ?? 0)
            }
            healthStore.execute(q)
        }
    }
}

struct DashboardContent: View {
    @ObservedObject var manager: ExerciseHealthManager
    
    var body: some View {
        ScrollView {
            VStack(spacing: 16) {
                // Summary Cards
                LazyVGrid(columns: [GridItem(.flexible()), GridItem(.flexible())], spacing: 16) {
                    MetricCard(icon: "figure.walk", title: "ก้าว", value: "\(manager.steps)", unit: "steps", color: .green)
                    MetricCard(icon: "heart.fill", title: "หัวใจ", value: "\(Int(manager.heartRate))", unit: "BPM", color: .red)
                    MetricCard(icon: "flame.fill", title: "แคลอรี่", value: "\(Int(manager.calories))", unit: "kcal", color: .orange)
                    MetricCard(icon: "map.fill", title: "ระยะทาง", value: String(format: "%.1f", manager.distance / 1000), unit: "km", color: .blue)
                }
                
                // Step Progress
                VStack(alignment: .leading) {
                    Text("เป้าหมายก้าว: 10,000")
                        .font(.headline)
                    ProgressView(value: Double(manager.steps), total: 10000)
                        .tint(.green)
                    Text("\(manager.steps) / 10,000 steps (\(Int(Double(manager.steps) / 10000 * 100))%)")
                        .font(.caption)
                        .foregroundColor(.secondary)
                }
                .padding()
                .background(Color(.systemBackground))
                .cornerRadius(12)
                .shadow(radius: 2)
            }
            .padding()
        }
        .background(Color(.systemGroupedBackground))
    }
}

struct MetricCard: View {
    let icon: String
    let title: String
    let value: String
    let unit: String
    let color: Color
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            HStack {
                Image(systemName: icon).foregroundColor(color)
                Spacer()
            }
            Text(value).font(.title2).fontWeight(.bold)
            Text(title + " (" + unit + ")").font(.caption).foregroundColor(.secondary)
        }
        .padding()
        .background(Color(.systemBackground))
        .cornerRadius(12)
        .shadow(radius: 2)
    }
}

struct ErrorView: View {
    let message: String
    let onRetry: () -> Void
    
    var body: some View {
        VStack(spacing: 16) {
            Image(systemName: "exclamationmark.triangle.fill")
                .font(.system(size: 50))
                .foregroundColor(.orange)
            Text("เกิดข้อผิดพลาด").font(.headline)
            Text(message).foregroundColor(.secondary).multilineTextAlignment(.center)
            Button("ลองอีกครั้ง", action: onRetry).buttonStyle(.borderedProminent)
        }
        .padding()
    }
}
```

### แบบฝึกหัดที่ 2: Workout Logger

```swift
// Solution: Workout Logger
class WorkoutLogger: ObservableObject {
    private let healthStore = HKHealthStore()
    
    @Published var workoutHistory: [WorkoutRecord] = []
    @Published var isLogging = false
    
    struct WorkoutRecord: Identifiable {
        let id = UUID()
        let type: HKWorkoutActivityType
        let startDate: Date
        let endDate: Date
        let calories: Double
        let distance: Double?
        
        var duration: TimeInterval { endDate.timeIntervalSince(startDate) }
        
        var typeName: String {
            switch type {
            case .running: return "วิ่ง"
            case .walking: return "เดิน"
            case .cycling: return "ปั่นจักรยาน"
            case .swimming: return "ว่ายน้ำ"
            case .yoga: return "โยคะ"
            default: return "ออกกำลังกาย"
            }
        }
        
        var formattedDuration: String {
            let minutes = Int(duration) / 60
            let seconds = Int(duration) % 60
            return String(format: "%02d:%02d", minutes, seconds)
        }
    }
    
    var workoutStartTime: Date?
    var currentWorkoutType: HKWorkoutActivityType = .running
    
    func startWorkout(type: HKWorkoutActivityType) {
        currentWorkoutType = type
        workoutStartTime = Date()
        isLogging = true
    }
    
    func stopWorkout(calories: Double, distance: Double?) async throws {
        guard let startTime = workoutStartTime else { return }
        
        let endTime = Date()
        isLogging = false
        workoutStartTime = nil
        
        // บันทึกลง HealthKit
        let caloriesQty = HKQuantity(unit: .kilocalorie(), doubleValue: calories)
        var distanceQty: HKQuantity?
        if let dist = distance {
            distanceQty = HKQuantity(unit: .meter(), doubleValue: dist)
        }
        
        let workout = HKWorkout(
            activityType: currentWorkoutType,
            start: startTime,
            end: endTime,
            duration: endTime.timeIntervalSince(startTime),
            totalEnergyBurned: caloriesQty,
            totalDistance: distanceQty,
            metadata: nil
        )
        
        try await healthStore.save(workout)
        
        // เพิ่มลงในประวัติ
        let record = WorkoutRecord(
            type: currentWorkoutType,
            startDate: startTime,
            endDate: endTime,
            calories: calories,
            distance: distance
        )
        
        await MainActor.run {
            workoutHistory.insert(record, at: 0)
        }
    }
    
    func fetchWorkoutHistory() async {
        let workoutType = HKObjectType.workoutType()
        let sort = NSSortDescriptor(key: HKSampleSortIdentifierStartDate, ascending: false)
        
        return await withCheckedContinuation { cont in
            let q = HKSampleQuery(sampleType: workoutType, predicate: nil, limit: 20, sortDescriptors: [sort]) { [weak self] _, samples, _ in
                let records = (samples as? [HKWorkout])?.map { workout in
                    WorkoutRecord(
                        type: workout.workoutActivityType,
                        startDate: workout.startDate,
                        endDate: workout.endDate,
                        calories: workout.totalEnergyBurned?.doubleValue(for: .kilocalorie()) ?? 0,
                        distance: workout.totalDistance?.doubleValue(for: .meter())
                    )
                } ?? []
                
                Task { @MainActor in
                    self?.workoutHistory = records
                }
                cont.resume()
            }
            healthStore.execute(q)
        }
    }
}
```

---

## การสร้าง Fitness Tracking App

### App Structure

```
FitnessApp/
├── App/
│   └── FitnessApp.swift
├── Models/
│   └── HealthModels.swift
├── ViewModels/
│   ├── HealthViewModel.swift
│   ├── WorkoutViewModel.swift
│   └── SleepViewModel.swift
├── Views/
│   ├── DashboardView.swift
│   ├── WorkoutView.swift
│   ├── SleepView.swift
│   └── ProfileView.swift
└── Services/
    └── HealthKitService.swift
```

### HealthKitService.swift

```swift
import HealthKit
import Combine

class HealthKitService {
    static let shared = HealthKitService()
    private let healthStore = HKHealthStore()
    
    private init() {}
    
    // MARK: - Authorization
    
    func requestFullAuthorization() async throws {
        guard HKHealthStore.isHealthDataAvailable() else {
            throw HealthError.notAvailable
        }
        
        let readTypes: Set<HKObjectType> = [
            HKQuantityType.quantityType(forIdentifier: .stepCount)!,
            HKQuantityType.quantityType(forIdentifier: .heartRate)!,
            HKQuantityType.quantityType(forIdentifier: .activeEnergyBurned)!,
            HKQuantityType.quantityType(forIdentifier: .distanceWalkingRunning)!,
            HKQuantityType.quantityType(forIdentifier: .bodyMass)!,
            HKQuantityType.quantityType(forIdentifier: .height)!,
            HKCategoryType.categoryType(forIdentifier: .sleepAnalysis)!,
            HKObjectType.workoutType()
        ]
        
        let writeTypes: Set<HKSampleType> = [
            HKQuantityType.quantityType(forIdentifier: .bodyMass)!,
            HKObjectType.workoutType()
        ]
        
        try await healthStore.requestAuthorization(toShare: writeTypes, read: readTypes)
    }
    
    // MARK: - Steps
    
    func fetchTodaySteps() async -> Int {
        guard let type = HKQuantityType.quantityType(forIdentifier: .stepCount) else { return 0 }
        let start = Calendar.current.startOfDay(for: Date())
        let predicate = HKQuery.predicateForSamples(withStart: start, end: Date())
        
        return await withCheckedContinuation { cont in
            let q = HKStatisticsQuery(quantityType: type, quantitySamplePredicate: predicate, options: .cumulativeSum) { _, r, _ in
                cont.resume(returning: Int(r?.sumQuantity()?.doubleValue(for: .count()) ?? 0))
            }
            healthStore.execute(q)
        }
    }
    
    func fetchWeeklySteps() async -> [(Date, Int)] {
        guard let type = HKQuantityType.quantityType(forIdentifier: .stepCount) else { return [] }
        
        let calendar = Calendar.current
        let end = Date()
        let start = calendar.date(byAdding: .day, value: -6, to: calendar.startOfDay(for: end))!
        
        var interval = DateComponents()
        interval.day = 1
        
        return await withCheckedContinuation { cont in
            let q = HKStatisticsCollectionQuery(
                quantityType: type,
                quantitySamplePredicate: nil,
                options: .cumulativeSum,
                anchorDate: start,
                intervalComponents: interval
            )
            
            q.initialResultsHandler = { _, collection, _ in
                var result: [(Date, Int)] = []
                collection?.enumerateStatistics(from: start, to: end) { stats, _ in
                    let steps = Int(stats.sumQuantity()?.doubleValue(for: .count()) ?? 0)
                    result.append((stats.startDate, steps))
                }
                cont.resume(returning: result)
            }
            
            healthStore.execute(q)
        }
    }
    
    // MARK: - Heart Rate
    
    func fetchLatestHeartRate() async -> Double {
        guard let type = HKQuantityType.quantityType(forIdentifier: .heartRate) else { return 0 }
        let sort = NSSortDescriptor(key: HKSampleSortIdentifierStartDate, ascending: false)
        
        return await withCheckedContinuation { cont in
            let q = HKSampleQuery(sampleType: type, predicate: nil, limit: 1, sortDescriptors: [sort]) { _, s, _ in
                let bpm = (s?.first as? HKQuantitySample)?.quantity.doubleValue(for: HKUnit(from: "count/min")) ?? 0
                cont.resume(returning: bpm)
            }
            healthStore.execute(q)
        }
    }
    
    // MARK: - Workouts
    
    func saveWorkout(_ workout: HKWorkout) async throws {
        try await healthStore.save(workout)
    }
    
    func fetchRecentWorkouts(limit: Int = 10) async -> [HKWorkout] {
        let sort = NSSortDescriptor(key: HKSampleSortIdentifierStartDate, ascending: false)
        
        return await withCheckedContinuation { cont in
            let q = HKSampleQuery(sampleType: HKObjectType.workoutType(), predicate: nil, limit: limit, sortDescriptors: [sort]) { _, s, _ in
                cont.resume(returning: s?.compactMap { $0 as? HKWorkout } ?? [])
            }
            healthStore.execute(q)
        }
    }
    
    // MARK: - Sleep
    
    func fetchSleepData(for date: Date) async -> [HKCategorySample] {
        guard let type = HKCategoryType.categoryType(forIdentifier: .sleepAnalysis) else { return [] }
        
        let calendar = Calendar.current
        let start = calendar.date(byAdding: .hour, value: -20, to: calendar.startOfDay(for: date))!
        let end = calendar.date(byAdding: .hour, value: 12, to: calendar.startOfDay(for: date))!
        let predicate = HKQuery.predicateForSamples(withStart: start, end: end)
        
        return await withCheckedContinuation { cont in
            let q = HKSampleQuery(sampleType: type, predicate: predicate, limit: HKObjectQueryNoLimit, sortDescriptors: nil) { _, s, _ in
                cont.resume(returning: s?.compactMap { $0 as? HKCategorySample } ?? [])
            }
            healthStore.execute(q)
        }
    }
    
    enum HealthError: LocalizedError {
        case notAvailable
        case notAuthorized
        case fetchFailed(String)
        
        var errorDescription: String? {
            switch self {
            case .notAvailable: return "HealthKit ไม่พร้อมใช้งานบนอุปกรณ์นี้"
            case .notAuthorized: return "ไม่ได้รับอนุญาตให้เข้าถึงข้อมูลสุขภาพ"
            case .fetchFailed(let msg): return "ไม่สามารถดึงข้อมูลได้: \(msg)"
            }
        }
    }
}
```

### Main App View

```swift
import SwiftUI

@main
struct FitnessApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}

struct ContentView: View {
    @StateObject private var viewModel = MainViewModel()
    
    var body: some View {
        TabView {
            DashboardTabView()
                .tabItem {
                    Label("หน้าหลัก", systemImage: "house.fill")
                }
            
            WorkoutTabView()
                .tabItem {
                    Label("ออกกำลังกาย", systemImage: "figure.run")
                }
            
            SleepTabView()
                .tabItem {
                    Label("การนอน", systemImage: "moon.zzz.fill")
                }
            
            ProfileTabView()
                .tabItem {
                    Label("โปรไฟล์", systemImage: "person.fill")
                }
        }
        .task {
            await viewModel.setup()
        }
    }
}

@MainActor
class MainViewModel: ObservableObject {
    let service = HealthKitService.shared
    
    func setup() async {
        do {
            try await service.requestFullAuthorization()
        } catch {
            print("Setup error: \(error)")
        }
    }
}

struct DashboardTabView: View {
    @StateObject private var vm = DashboardViewModel()
    
    var body: some View {
        NavigationStack {
            ScrollView {
                LazyVStack(spacing: 16) {
                    TodaySummaryCard(vm: vm)
                    WeeklyStepsChart(data: vm.weeklySteps)
                    RecentWorkoutsCard(workouts: vm.recentWorkouts)
                }
                .padding()
            }
            .navigationTitle("สุขภาพของฉัน")
            .refreshable { await vm.refresh() }
        }
        .task { await vm.load() }
    }
}

@MainActor
class DashboardViewModel: ObservableObject {
    @Published var todaySteps = 0
    @Published var heartRate = 0.0
    @Published var calories = 0.0
    @Published var weeklySteps: [(Date, Int)] = []
    @Published var recentWorkouts: [HKWorkout] = []
    
    let service = HealthKitService.shared
    
    func load() async {
        await withTaskGroup(of: Void.self) { g in
            g.addTask { self.todaySteps = await self.service.fetchTodaySteps() }
            g.addTask { self.heartRate = await self.service.fetchLatestHeartRate() }
            g.addTask { self.weeklySteps = await self.service.fetchWeeklySteps() }
            g.addTask { self.recentWorkouts = await self.service.fetchRecentWorkouts() }
        }
    }
    
    func refresh() async { await load() }
}

struct TodaySummaryCard: View {
    @ObservedObject var vm: DashboardViewModel
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            Text("วันนี้")
                .font(.headline)
            
            HStack(spacing: 20) {
                StatItem(icon: "figure.walk", value: "\(vm.todaySteps)", label: "ก้าว", color: .green)
                Divider()
                StatItem(icon: "heart.fill", value: "\(Int(vm.heartRate))", label: "BPM", color: .red)
                Divider()
                StatItem(icon: "flame.fill", value: "\(Int(vm.calories))", label: "kcal", color: .orange)
            }
        }
        .padding()
        .background(Color(.systemBackground))
        .cornerRadius(16)
        .shadow(color: .black.opacity(0.05), radius: 8, y: 2)
    }
}

struct StatItem: View {
    let icon: String
    let value: String
    let label: String
    let color: Color
    
    var body: some View {
        VStack(spacing: 4) {
            Image(systemName: icon).foregroundColor(color)
            Text(value).font(.title3).fontWeight(.bold)
            Text(label).font(.caption).foregroundColor(.secondary)
        }
        .frame(maxWidth: .infinity)
    }
}

struct WeeklyStepsChart: View {
    let data: [(Date, Int)]
    let maxSteps = 10000
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            Text("ก้าวรายสัปดาห์")
                .font(.headline)
            
            HStack(alignment: .bottom, spacing: 8) {
                ForEach(data, id: \.0) { date, steps in
                    VStack(spacing: 4) {
                        Text("\(steps / 1000)k")
                            .font(.caption2)
                            .foregroundColor(.secondary)
                        
                        RoundedRectangle(cornerRadius: 4)
                            .fill(steps >= maxSteps ? Color.green : Color.green.opacity(0.5))
                            .frame(width: 32, height: max(CGFloat(steps) / CGFloat(maxSteps) * 100, 4))
                        
                        Text(dayLabel(date))
                            .font(.caption2)
                            .foregroundColor(.secondary)
                    }
                }
            }
            .frame(height: 120, alignment: .bottom)
        }
        .padding()
        .background(Color(.systemBackground))
        .cornerRadius(16)
        .shadow(color: .black.opacity(0.05), radius: 8, y: 2)
    }
    
    func dayLabel(_ date: Date) -> String {
        let f = DateFormatter()
        f.dateFormat = "EEE"
        f.locale = Locale(identifier: "th_TH")
        return f.string(from: date)
    }
}

struct RecentWorkoutsCard: View {
    let workouts: [HKWorkout]
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            Text("การออกกำลังกายล่าสุด")
                .font(.headline)
            
            if workouts.isEmpty {
                Text("ยังไม่มีข้อมูลการออกกำลังกาย")
                    .foregroundColor(.secondary)
                    .frame(maxWidth: .infinity)
                    .padding()
            } else {
                ForEach(workouts.prefix(5), id: \.uuid) { workout in
                    WorkoutRow(workout: workout)
                    if workout != workouts.prefix(5).last {
                        Divider()
                    }
                }
            }
        }
        .padding()
        .background(Color(.systemBackground))
        .cornerRadius(16)
        .shadow(color: .black.opacity(0.05), radius: 8, y: 2)
    }
}

struct WorkoutRow: View {
    let workout: HKWorkout
    
    var typeName: String {
        switch workout.workoutActivityType {
        case .running: return "วิ่ง"
        case .walking: return "เดิน"
        case .cycling: return "ปั่นจักรยาน"
        default: return "ออกกำลังกาย"
        }
    }
    
    var body: some View {
        HStack {
            Image(systemName: "figure.run")
                .foregroundColor(.blue)
                .frame(width: 30)
            
            VStack(alignment: .leading) {
                Text(typeName).fontWeight(.medium)
                Text(workout.startDate, style: .date)
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
            
            Spacer()
            
            VStack(alignment: .trailing) {
                let minutes = Int(workout.duration) / 60
                Text("\(minutes) นาที").font(.callout)
                
                if let cal = workout.totalEnergyBurned?.doubleValue(for: .kilocalorie()) {
                    Text("\(Int(cal)) kcal")
                        .font(.caption)
                        .foregroundColor(.orange)
                }
            }
        }
    }
}

struct WorkoutTabView: View {
    var body: some View {
        NavigationStack {
            Text("หน้า Workout")
                .navigationTitle("ออกกำลังกาย")
        }
    }
}

struct SleepTabView: View {
    var body: some View {
        NavigationStack {
            Text("หน้า Sleep")
                .navigationTitle("การนอนหลับ")
        }
    }
}

struct ProfileTabView: View {
    var body: some View {
        NavigationStack {
            Text("หน้า Profile")
                .navigationTitle("โปรไฟล์")
        }
    }
}
```

---

## สรุป

HealthKit เป็น framework ที่ทรงพลังสำหรับการพัฒนาแอปสุขภาพบน iOS สิ่งสำคัญที่ต้องจำ:

### Key Points

1. **HKHealthStore** - จุดเข้าถึงหลักของ HealthKit ควรใช้ instance เดียว
2. **Authorization** - ต้องขอ permission จากผู้ใช้เสมอ ทั้ง read และ write
3. **HKQuantityType** - สำหรับข้อมูลตัวเลข เช่น ก้าว, หัวใจ, น้ำหนัก
4. **HKCategoryType** - สำหรับข้อมูลหมวดหมู่ เช่น การนอนหลับ
5. **Query Types** - มีหลายประเภทตามวัตถุประสงค์:
   - `HKSampleQuery` - ดึง samples ทั่วไป
   - `HKStatisticsQuery` - คำนวณสถิติ
   - `HKStatisticsCollectionQuery` - สถิติตามช่วงเวลา
   - `HKObserverQuery` - ติดตามการเปลี่ยนแปลง
   - `HKAnchoredObjectQuery` - ดึงข้อมูลใหม่
6. **Background Delivery** - รับแจ้งเตือนเมื่อมีข้อมูลใหม่ แม้แอปอยู่ใน background
7. **Privacy** - ข้อมูลสุขภาพเป็นข้อมูลส่วนตัวสูง ต้องใช้อย่างระมัดระวัง

### Best Practices

```swift
// 1. ตรวจสอบความพร้อมใช้งานก่อนเสมอ
guard HKHealthStore.isHealthDataAvailable() else { return }

// 2. ใช้ async/await สำหรับ modern code
try await healthStore.requestAuthorization(toShare: writeTypes, read: readTypes)

// 3. จัดการ error อย่างเหมาะสม
do {
    let steps = try await fetchSteps()
    updateUI(with: steps)
} catch {
    showError(error)
}

// 4. อัปเดต UI บน main thread เสมอ
await MainActor.run {
    self.todaySteps = steps
}

// 5. ขอ permission เฉพาะสิ่งที่จำเป็น
let readTypes: Set<HKObjectType> = [
    // ขอเฉพาะ types ที่แอปใช้จริงๆ
]
```

---

*จบ Part 57: HealthKit*
