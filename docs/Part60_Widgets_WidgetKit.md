# Part 60: Widgets และ WidgetKit

## บทนำ

Widgets เป็นส่วนสำคัญของ Apple ecosystem ที่ช่วยให้ผู้ใช้เห็นข้อมูลสำคัญได้รวดเร็วโดยไม่ต้องเปิดแอป WidgetKit เป็น framework ที่ Apple สร้างขึ้นสำหรับการพัฒนา Widgets บน iOS, iPadOS, macOS, watchOS และ visionOS

---

## 60.1 What are Widgets?

Widgets คือ mini-views ที่แสดงข้อมูลจากแอปบน Home Screen, Lock Screen หรือ Today View โดยมีคุณสมบัติสำคัญ:

- **Read-only** (ก่อน iOS 17): ผู้ใช้ดูได้แต่ไม่สามารถโต้ตอบได้
- **Interactive** (iOS 17+): รองรับ Button และ Toggle
- **Timeline-based**: ข้อมูลอัพเดทตาม Timeline ที่กำหนด
- **Multiple Sizes**: มีหลายขนาดให้เลือก

```
Widget Types:
├── Home Screen Widgets (iOS 14+)
│   ├── Small (2x2 grid units)
│   ├── Medium (4x2 grid units)
│   └── Large (4x4 grid units)
├── Lock Screen Widgets (iOS 16+)
│   ├── accessoryCircular
│   ├── accessoryRectangular
│   └── accessoryInline
├── iPad Extra Large (iOS 15+)
│   └── extraLarge (4x4 or larger)
├── Mac Desktop Widgets (macOS 14+)
└── StandBy Widgets (iOS 17+)
```

---

## 60.2 WidgetKit Overview

WidgetKit เป็น framework ใหม่ที่แทนที่ Today Extension เดิม

### สร้าง Widget Extension

```swift
// ใน Xcode: File > New > Target > Widget Extension

// WidgetKit Import
import WidgetKit
import SwiftUI

// Widget Structure
struct MyWidget: Widget {
    // Bundle Identifier
    let kind: String = "com.example.app.MyWidget"
    
    var body: some WidgetConfiguration {
        // StaticConfiguration - ไม่มีการตั้งค่า
        StaticConfiguration(kind: kind, provider: MyTimelineProvider()) { entry in
            MyWidgetEntryView(entry: entry)
        }
        .configurationDisplayName("My Widget")
        .description("แสดงข้อมูลจากแอป")
        .supportedFamilies([.systemSmall, .systemMedium, .systemLarge])
    }
}
```

### Widget Extension Bundle Structure

```
MyApp.xcodeproj
├── MyApp/ (Main App Target)
│   └── ...
└── MyAppWidget/ (Widget Extension Target)
    ├── MyAppWidget.swift (Widget Bundle)
    ├── Provider.swift (Timeline Provider)
    ├── Entry.swift (Timeline Entry)
    ├── EntryView.swift (Widget View)
    └── Assets.xcassets
```

---

## 60.3 Widget Entry

TimelineEntry คือโครงสร้างที่เก็บข้อมูลที่ Widget จะแสดง ณ เวลาหนึ่ง

```swift
import WidgetKit

// Basic Entry
struct SimpleEntry: TimelineEntry {
    let date: Date
    let message: String
}

// Entry ที่ซับซ้อน
struct WeatherEntry: TimelineEntry {
    let date: Date
    let temperature: Double
    let condition: WeatherCondition
    let city: String
    let humidity: Int
    let feelsLike: Double
    let forecast: [DailyForecast]
    
    enum WeatherCondition: String, CaseIterable {
        case sunny = "สภาพแจ่มใส"
        case cloudy = "มีเมฆมาก"
        case rainy = "ฝนตก"
        case stormy = "พายุ"
        case snowy = "หิมะตก"
        case foggy = "มีหมอก"
        
        var systemImage: String {
            switch self {
            case .sunny: return "sun.max.fill"
            case .cloudy: return "cloud.fill"
            case .rainy: return "cloud.rain.fill"
            case .stormy: return "cloud.bolt.rain.fill"
            case .snowy: return "snow"
            case .foggy: return "cloud.fog.fill"
            }
        }
        
        var color: Color {
            switch self {
            case .sunny: return .yellow
            case .cloudy: return .gray
            case .rainy: return .blue
            case .stormy: return .purple
            case .snowy: return .cyan
            case .foggy: return .gray.opacity(0.7)
            }
        }
    }
    
    struct DailyForecast {
        let day: String
        let high: Double
        let low: Double
        let condition: WeatherCondition
    }
    
    // Placeholder Entry
    static var placeholder: WeatherEntry {
        WeatherEntry(
            date: Date(),
            temperature: 28,
            condition: .sunny,
            city: "กรุงเทพฯ",
            humidity: 65,
            feelsLike: 32,
            forecast: [
                DailyForecast(day: "จ.", high: 30, low: 25, condition: .sunny),
                DailyForecast(day: "อ.", high: 28, low: 24, condition: .cloudy),
                DailyForecast(day: "พ.", high: 26, low: 23, condition: .rainy)
            ]
        )
    }
}
```

---

## 60.4 IntentConfiguration vs StaticConfiguration

### StaticConfiguration

ใช้เมื่อ Widget ไม่มีการตั้งค่าที่ผู้ใช้สามารถปรับได้

```swift
import WidgetKit
import SwiftUI

struct ClockWidget: Widget {
    let kind = "ClockWidget"
    
    var body: some WidgetConfiguration {
        StaticConfiguration(
            kind: kind,
            provider: ClockProvider()
        ) { entry in
            ClockWidgetView(entry: entry)
                .containerBackground(.fill.tertiary, for: .widget)
        }
        .configurationDisplayName("นาฬิกา")
        .description("แสดงเวลาปัจจุบัน")
        .supportedFamilies([.systemSmall, .systemMedium])
    }
}
```

### AppIntentConfiguration (iOS 17+)

ใช้เมื่อต้องการให้ผู้ใช้ตั้งค่า Widget ได้

```swift
import WidgetKit
import AppIntents
import SwiftUI

// AppIntent สำหรับ Widget Configuration
struct CitySelectionIntent: WidgetConfigurationIntent {
    static var title: LocalizedStringResource = "เลือกเมือง"
    static var description = IntentDescription("เลือกเมืองที่ต้องการดูสภาพอากาศ")
    
    @Parameter(title: "เมือง")
    var city: CityEntity?
    
    // Default Values
    init() {}
    init(city: CityEntity) {
        self.city = city
    }
}

// Entity สำหรับ City
struct CityEntity: AppEntity {
    let id: String
    let name: String
    let country: String
    
    static var typeDisplayRepresentation: TypeDisplayRepresentation = "เมือง"
    static var defaultQuery = CityQuery()
    
    var displayRepresentation: DisplayRepresentation {
        DisplayRepresentation(title: "\(name), \(country)")
    }
}

struct CityQuery: EntityQuery {
    func entities(for identifiers: [String]) async throws -> [CityEntity] {
        return availableCities.filter { identifiers.contains($0.id) }
    }
    
    func suggestedEntities() async throws -> [CityEntity] {
        return availableCities
    }
    
    var availableCities: [CityEntity] {
        [
            CityEntity(id: "bkk", name: "กรุงเทพฯ", country: "ไทย"),
            CityEntity(id: "cm", name: "เชียงใหม่", country: "ไทย"),
            CityEntity(id: "pkt", name: "ภูเก็ต", country: "ไทย"),
            CityEntity(id: "tokyo", name: "โตเกียว", country: "ญี่ปุ่น"),
            CityEntity(id: "nyc", name: "นิวยอร์ก", country: "สหรัฐฯ")
        ]
    }
}

// Widget ด้วย AppIntentConfiguration
struct WeatherWidget: Widget {
    let kind = "WeatherWidget"
    
    var body: some WidgetConfiguration {
        AppIntentConfiguration(
            kind: kind,
            intent: CitySelectionIntent.self,
            provider: WeatherProvider()
        ) { entry in
            WeatherWidgetView(entry: entry)
                .containerBackground(.fill.tertiary, for: .widget)
        }
        .configurationDisplayName("สภาพอากาศ")
        .description("ดูสภาพอากาศของเมืองที่เลือก")
        .supportedFamilies([.systemSmall, .systemMedium, .systemLarge])
    }
}
```

---

## 60.5 Timeline Provider

Timeline Provider กำหนดว่า Widget จะแสดงข้อมูลอะไรและเมื่อไหร่

```swift
import WidgetKit

// Basic TimelineProvider
struct ClockProvider: TimelineProvider {
    typealias Entry = ClockEntry
    
    // Placeholder - แสดงขณะโหลด
    func placeholder(in context: Context) -> ClockEntry {
        ClockEntry(date: Date(), displayTime: "12:00")
    }
    
    // Snapshot - Preview ใน Widget Gallery
    func getSnapshot(in context: Context, completion: @escaping (ClockEntry) -> Void) {
        let entry = ClockEntry(date: Date(), displayTime: formatTime(Date()))
        completion(entry)
    }
    
    // Timeline - ข้อมูลจริงที่จะแสดง
    func getTimeline(in context: Context, completion: @escaping (Timeline<ClockEntry>) -> Void) {
        var entries: [ClockEntry] = []
        
        let calendar = Calendar.current
        let currentDate = Date()
        
        // สร้าง Entries สำหรับ 24 ชั่วโมง (ทุกนาที)
        for minuteOffset in 0..<60 {
            let entryDate = calendar.date(
                byAdding: .minute,
                value: minuteOffset,
                to: currentDate
            )!
            
            let entry = ClockEntry(
                date: entryDate,
                displayTime: formatTime(entryDate)
            )
            entries.append(entry)
        }
        
        // Reload หลังจาก 1 ชั่วโมง
        let timeline = Timeline(entries: entries, policy: .atEnd)
        completion(timeline)
    }
    
    private func formatTime(_ date: Date) -> String {
        let formatter = DateFormatter()
        formatter.dateFormat = "HH:mm"
        return formatter.string(from: date)
    }
}

struct ClockEntry: TimelineEntry {
    let date: Date
    let displayTime: String
}
```

### AppIntentTimelineProvider (iOS 17+)

```swift
import WidgetKit
import AppIntents

struct WeatherProvider: AppIntentTimelineProvider {
    typealias Entry = WeatherEntry
    typealias Intent = CitySelectionIntent
    
    func placeholder(in context: Context) -> WeatherEntry {
        WeatherEntry.placeholder
    }
    
    func snapshot(for configuration: CitySelectionIntent, in context: Context) async -> WeatherEntry {
        await fetchWeather(for: configuration.city?.id ?? "bkk")
    }
    
    func timeline(for configuration: CitySelectionIntent, in context: Context) async -> Timeline<WeatherEntry> {
        let cityID = configuration.city?.id ?? "bkk"
        let entry = await fetchWeather(for: cityID)
        
        // อัพเดทอีกครั้งใน 1 ชั่วโมง
        let nextUpdate = Calendar.current.date(byAdding: .hour, value: 1, to: Date())!
        return Timeline(entries: [entry], policy: .after(nextUpdate))
    }
    
    private func fetchWeather(for cityID: String) async -> WeatherEntry {
        // จำลอง Network Request
        // ในแอปจริงจะเรียก API
        try? await Task.sleep(nanoseconds: 100_000_000) // 0.1 วินาที
        
        return WeatherEntry(
            date: Date(),
            temperature: Double.random(in: 25...35),
            condition: WeatherEntry.WeatherCondition.allCases.randomElement()!,
            city: cityID == "bkk" ? "กรุงเทพฯ" : "เมืองอื่น",
            humidity: Int.random(in: 50...90),
            feelsLike: Double.random(in: 28...38),
            forecast: WeatherEntry.placeholder.forecast
        )
    }
}
```

---

## 60.6 TimelineEntry

TimelineEntry เพิ่มเติม

```swift
import WidgetKit
import SwiftUI

// Entry สำหรับ Stock Price Widget
struct StockEntry: TimelineEntry {
    let date: Date
    let symbol: String
    let price: Double
    let change: Double
    let changePercent: Double
    let chartData: [Double] // Historical prices
    let isMarketOpen: Bool
    
    var changeColor: Color {
        change >= 0 ? .green : .red
    }
    
    var changeIcon: String {
        change >= 0 ? "arrow.up.right" : "arrow.down.right"
    }
    
    var formattedPrice: String {
        String(format: "%.2f", price)
    }
    
    var formattedChange: String {
        let sign = change >= 0 ? "+" : ""
        return "\(sign)\(String(format: "%.2f", change)) (\(sign)\(String(format: "%.2f", changePercent))%)"
    }
    
    static var sample: StockEntry {
        StockEntry(
            date: Date(),
            symbol: "AAPL",
            price: 175.50,
            change: 2.30,
            changePercent: 1.33,
            chartData: [170, 172, 168, 173, 175, 174, 175.5],
            isMarketOpen: true
        )
    }
}

// Entry สำหรับ Calendar Widget
struct CalendarEntry: TimelineEntry {
    let date: Date
    let events: [CalendarEvent]
    let upcomingEvent: CalendarEvent?
    
    struct CalendarEvent: Identifiable {
        let id: UUID
        let title: String
        let startTime: Date
        let endTime: Date
        let color: Color
        let location: String?
        
        var timeString: String {
            let formatter = DateFormatter()
            formatter.dateFormat = "HH:mm"
            return "\(formatter.string(from: startTime)) - \(formatter.string(from: endTime))"
        }
        
        var duration: TimeInterval {
            endTime.timeIntervalSince(startTime)
        }
    }
}
```

---

## 60.7 TimelineReloadPolicy

TimelineReloadPolicy กำหนดว่า Widget จะอัพเดทข้อมูลเมื่อไหร่

```swift
import WidgetKit

struct NewsProvider: TimelineProvider {
    typealias Entry = NewsEntry
    
    func placeholder(in context: Context) -> NewsEntry {
        NewsEntry(date: Date(), headlines: [])
    }
    
    func getSnapshot(in context: Context, completion: @escaping (NewsEntry) -> Void) {
        completion(NewsEntry(date: Date(), headlines: ["ข่าวล่าสุด"]))
    }
    
    func getTimeline(in context: Context, completion: @escaping (Timeline<NewsEntry>) -> Void) {
        fetchLatestNews { news in
            let entry = NewsEntry(date: Date(), headlines: news)
            
            // Policy.atEnd - รีโหลดหลังจาก Entry สุดท้ายผ่านไป
            let timeline1 = Timeline(entries: [entry], policy: .atEnd)
            
            // Policy.after - รีโหลดหลังจากวันที่/เวลาที่กำหนด
            let nextRefresh = Calendar.current.date(byAdding: .hour, value: 1, to: Date())!
            let timeline2 = Timeline(entries: [entry], policy: .after(nextRefresh))
            
            // Policy.never - ไม่รีโหลดอัตโนมัติ ต้องรีโหลดด้วยตนเอง
            let timeline3 = Timeline(entries: [entry], policy: .never)
            
            // เลือกใช้ timeline ที่เหมาะสม
            completion(timeline2) // รีโหลดทุก 1 ชั่วโมง
        }
    }
    
    private func fetchLatestNews(completion: @escaping ([String]) -> Void) {
        // จำลอง API Call
        DispatchQueue.global().asyncAfter(deadline: .now() + 0.5) {
            completion([
                "ข่าวด่วน: ตลาดหุ้นปิดบวก",
                "สภาพอากาศ: ฝนตกหนักทั่วกรุงเทพฯ",
                "กีฬา: ทีมชาติไทยชนะเลิศ"
            ])
        }
    }
}

struct NewsEntry: TimelineEntry {
    let date: Date
    let headlines: [String]
}

// รีโหลด Widget ด้วยตนเองจากแอปหลัก
class WidgetReloader {
    
    // รีโหลด Widget ทั้งหมด
    static func reloadAll() {
        WidgetCenter.shared.reloadAllTimelines()
    }
    
    // รีโหลด Widget เฉพาะชนิด
    static func reload(kind: String) {
        WidgetCenter.shared.reloadTimelines(ofKind: kind)
    }
    
    // ดูข้อมูล Widget ที่มีอยู่
    static func getCurrentWidgets() async -> [WidgetInfo] {
        return await WidgetCenter.shared.currentConfigurations()
    }
}
```

---

## 60.8 Widget Families

### รองรับทุก Family

```swift
import WidgetKit
import SwiftUI

struct MultiSizeWidget: Widget {
    let kind = "MultiSizeWidget"
    
    var body: some WidgetConfiguration {
        StaticConfiguration(kind: kind, provider: MultiSizeProvider()) { entry in
            MultiSizeWidgetView(entry: entry)
                .containerBackground(.fill.tertiary, for: .widget)
        }
        .supportedFamilies([
            // Home Screen
            .systemSmall,
            .systemMedium,
            .systemLarge,
            .systemExtraLarge,  // iPad only
            
            // Lock Screen (iOS 16+)
            .accessoryCircular,
            .accessoryRectangular,
            .accessoryInline
        ])
    }
}

// View ที่ปรับตาม Family
struct MultiSizeWidgetView: View {
    let entry: MultiSizeEntry
    @Environment(\.widgetFamily) var family
    
    var body: some View {
        switch family {
        case .systemSmall:
            SmallWidgetView(entry: entry)
        case .systemMedium:
            MediumWidgetView(entry: entry)
        case .systemLarge:
            LargeWidgetView(entry: entry)
        case .systemExtraLarge:
            ExtraLargeWidgetView(entry: entry)
        case .accessoryCircular:
            AccessoryCircularView(entry: entry)
        case .accessoryRectangular:
            AccessoryRectangularView(entry: entry)
        case .accessoryInline:
            AccessoryInlineView(entry: entry)
        @unknown default:
            SmallWidgetView(entry: entry)
        }
    }
}

// Small Widget View
struct SmallWidgetView: View {
    let entry: MultiSizeEntry
    
    var body: some View {
        VStack(alignment: .leading, spacing: 4) {
            Image(systemName: entry.icon)
                .font(.title2)
                .foregroundColor(.accentColor)
            
            Spacer()
            
            Text(entry.title)
                .font(.headline)
                .lineLimit(2)
            
            Text(entry.subtitle)
                .font(.caption)
                .foregroundColor(.secondary)
        }
        .padding()
    }
}

// Medium Widget View
struct MediumWidgetView: View {
    let entry: MultiSizeEntry
    
    var body: some View {
        HStack(spacing: 16) {
            // Left Column
            VStack(alignment: .leading, spacing: 4) {
                Image(systemName: entry.icon)
                    .font(.largeTitle)
                    .foregroundColor(.accentColor)
                
                Text(entry.title)
                    .font(.headline)
                
                Text(entry.subtitle)
                    .font(.subheadline)
                    .foregroundColor(.secondary)
            }
            
            Divider()
            
            // Right Column
            VStack(alignment: .leading, spacing: 8) {
                ForEach(entry.details.prefix(3), id: \.self) { detail in
                    Label(detail, systemImage: "checkmark.circle.fill")
                        .font(.caption)
                        .foregroundColor(.secondary)
                }
            }
        }
        .padding()
    }
}

// Large Widget View
struct LargeWidgetView: View {
    let entry: MultiSizeEntry
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            // Header
            HStack {
                Image(systemName: entry.icon)
                    .font(.title)
                    .foregroundColor(.accentColor)
                
                VStack(alignment: .leading) {
                    Text(entry.title)
                        .font(.title2)
                        .fontWeight(.bold)
                    
                    Text(entry.subtitle)
                        .foregroundColor(.secondary)
                }
                
                Spacer()
            }
            
            Divider()
            
            // Content
            ForEach(entry.details, id: \.self) { detail in
                HStack {
                    Image(systemName: "arrow.right.circle.fill")
                        .foregroundColor(.accentColor)
                    
                    Text(detail)
                    
                    Spacer()
                }
            }
            
            Spacer()
            
            // Footer
            Text("อัพเดท: \(entry.date, style: .time)")
                .font(.caption2)
                .foregroundColor(.secondary)
        }
        .padding()
    }
}

// Extra Large Widget (iPad)
struct ExtraLargeWidgetView: View {
    let entry: MultiSizeEntry
    
    var body: some View {
        LazyVGrid(columns: [GridItem(.flexible()), GridItem(.flexible())], spacing: 16) {
            ForEach(entry.details, id: \.self) { detail in
                RoundedRectangle(cornerRadius: 12)
                    .fill(.quaternary)
                    .frame(height: 80)
                    .overlay(
                        Text(detail)
                            .font(.subheadline)
                            .padding()
                    )
            }
        }
        .padding()
    }
}

// Accessory Circular (Lock Screen)
struct AccessoryCircularView: View {
    let entry: MultiSizeEntry
    
    var body: some View {
        ZStack {
            AccessoryWidgetBackground()
            
            VStack(spacing: 2) {
                Image(systemName: entry.icon)
                    .font(.title3)
                
                Text(entry.value)
                    .font(.caption2)
                    .fontWeight(.semibold)
            }
        }
    }
}

// Accessory Rectangular (Lock Screen)
struct AccessoryRectangularView: View {
    let entry: MultiSizeEntry
    
    var body: some View {
        HStack(spacing: 8) {
            Image(systemName: entry.icon)
                .font(.title2)
            
            VStack(alignment: .leading, spacing: 2) {
                Text(entry.title)
                    .font(.headline)
                    .lineLimit(1)
                
                Text(entry.value)
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
        }
    }
}

// Accessory Inline (Lock Screen)
struct AccessoryInlineView: View {
    let entry: MultiSizeEntry
    
    var body: some View {
        Label(entry.title, systemImage: entry.icon)
    }
}

// Entry
struct MultiSizeEntry: TimelineEntry {
    let date: Date
    let icon: String
    let title: String
    let subtitle: String
    let value: String
    let details: [String]
    
    static var sample: MultiSizeEntry {
        MultiSizeEntry(
            date: Date(),
            icon: "star.fill",
            title: "Widget ตัวอย่าง",
            subtitle: "คำอธิบายสั้นๆ",
            value: "42",
            details: ["รายละเอียด 1", "รายละเอียด 2", "รายละเอียด 3", "รายละเอียด 4"]
        )
    }
}

struct MultiSizeProvider: TimelineProvider {
    typealias Entry = MultiSizeEntry
    
    func placeholder(in context: Context) -> MultiSizeEntry { .sample }
    func getSnapshot(in context: Context, completion: @escaping (MultiSizeEntry) -> Void) { completion(.sample) }
    func getTimeline(in context: Context, completion: @escaping (Timeline<MultiSizeEntry>) -> Void) {
        let timeline = Timeline(entries: [MultiSizeEntry.sample], policy: .atEnd)
        completion(timeline)
    }
}
```

---

## 60.9 Widget Views

### สร้าง Widget View ที่สวยงาม

```swift
import SwiftUI
import WidgetKit

// Weather Widget View
struct WeatherWidgetView: View {
    let entry: WeatherEntry
    @Environment(\.widgetFamily) var family
    @Environment(\.colorScheme) var colorScheme
    
    var body: some View {
        Group {
            switch family {
            case .systemSmall:
                smallView
            case .systemMedium:
                mediumView
            case .systemLarge:
                largeView
            default:
                smallView
            }
        }
        .foregroundColor(.white)
    }
    
    // Small View
    var smallView: some View {
        ZStack {
            // Background Gradient
            LinearGradient(
                colors: [
                    entry.condition.color,
                    entry.condition.color.opacity(0.7)
                ],
                startPoint: .topLeading,
                endPoint: .bottomTrailing
            )
            
            VStack(alignment: .leading, spacing: 8) {
                // City
                Text(entry.city)
                    .font(.caption)
                    .opacity(0.8)
                
                Spacer()
                
                // Weather Icon
                Image(systemName: entry.condition.systemImage)
                    .font(.title)
                
                // Temperature
                Text("\(Int(entry.temperature))°")
                    .font(.title)
                    .fontWeight(.bold)
                
                // Condition
                Text(entry.condition.rawValue)
                    .font(.caption2)
                    .opacity(0.8)
            }
            .padding()
        }
    }
    
    // Medium View
    var mediumView: some View {
        ZStack {
            LinearGradient(
                colors: [entry.condition.color, entry.condition.color.opacity(0.6)],
                startPoint: .topLeading,
                endPoint: .bottomTrailing
            )
            
            HStack(spacing: 16) {
                // Left: Current Weather
                VStack(alignment: .leading, spacing: 4) {
                    Text(entry.city)
                        .font(.subheadline)
                        .opacity(0.8)
                    
                    Image(systemName: entry.condition.systemImage)
                        .font(.largeTitle)
                    
                    Text("\(Int(entry.temperature))°")
                        .font(.largeTitle)
                        .fontWeight(.bold)
                    
                    Text(entry.condition.rawValue)
                        .font(.caption)
                        .opacity(0.8)
                }
                
                Divider()
                    .background(.white.opacity(0.3))
                
                // Right: Details
                VStack(alignment: .leading, spacing: 8) {
                    DetailRow(icon: "thermometer", label: "รู้สึก", value: "\(Int(entry.feelsLike))°")
                    DetailRow(icon: "drop.fill", label: "ความชื้น", value: "\(entry.humidity)%")
                    DetailRow(icon: "clock", label: "อัพเดท", value: entry.date, style: .time)
                }
            }
            .padding()
        }
    }
    
    // Large View
    var largeView: some View {
        ZStack {
            LinearGradient(
                colors: [entry.condition.color, entry.condition.color.opacity(0.5)],
                startPoint: .top,
                endPoint: .bottom
            )
            
            VStack(spacing: 0) {
                // Current Weather Section
                HStack {
                    VStack(alignment: .leading, spacing: 4) {
                        Text(entry.city)
                            .font(.title3)
                            .opacity(0.8)
                        
                        Text("\(Int(entry.temperature))°")
                            .font(.system(size: 64, weight: .thin))
                        
                        Text(entry.condition.rawValue)
                            .font(.subheadline)
                            .opacity(0.8)
                    }
                    
                    Spacer()
                    
                    Image(systemName: entry.condition.systemImage)
                        .font(.system(size: 64))
                        .opacity(0.8)
                }
                .padding()
                
                Divider()
                    .background(.white.opacity(0.3))
                
                // Additional Details
                HStack(spacing: 24) {
                    WeatherDetail(icon: "thermometer", value: "\(Int(entry.feelsLike))°", label: "รู้สึก")
                    WeatherDetail(icon: "drop.fill", value: "\(entry.humidity)%", label: "ความชื้น")
                }
                .padding()
                
                Divider()
                    .background(.white.opacity(0.3))
                
                // Forecast
                HStack(spacing: 0) {
                    ForEach(entry.forecast.prefix(3), id: \.day) { day in
                        ForecastDay(forecast: day)
                        
                        if day.day != entry.forecast.prefix(3).last?.day {
                            Divider()
                                .background(.white.opacity(0.3))
                        }
                    }
                }
                .padding(.vertical)
            }
        }
    }
}

// Helper Views
struct DetailRow: View {
    let icon: String
    let label: String
    let value: String
    
    init(icon: String, label: String, value: String) {
        self.icon = icon
        self.label = label
        self.value = value
    }
    
    init(icon: String, label: String, value: Date, style: Text.DateStyle) {
        self.icon = icon
        self.label = label
        self.value = "" // จะใช้ Date display แทน
    }
    
    var body: some View {
        HStack(spacing: 4) {
            Image(systemName: icon)
                .font(.caption)
                .frame(width: 16)
                .opacity(0.7)
            
            Text(label)
                .font(.caption)
                .opacity(0.7)
            
            Spacer()
            
            Text(value)
                .font(.caption)
                .fontWeight(.semibold)
        }
    }
}

struct WeatherDetail: View {
    let icon: String
    let value: String
    let label: String
    
    var body: some View {
        VStack(spacing: 4) {
            Image(systemName: icon)
                .font(.title3)
                .opacity(0.8)
            
            Text(value)
                .font(.headline)
            
            Text(label)
                .font(.caption2)
                .opacity(0.7)
        }
        .frame(maxWidth: .infinity)
    }
}

struct ForecastDay: View {
    let forecast: WeatherEntry.DailyForecast
    
    var body: some View {
        VStack(spacing: 4) {
            Text(forecast.day)
                .font(.caption)
                .opacity(0.7)
            
            Image(systemName: forecast.condition.systemImage)
                .font(.title3)
            
            Text("\(Int(forecast.high))°")
                .font(.subheadline)
                .fontWeight(.semibold)
            
            Text("\(Int(forecast.low))°")
                .font(.caption)
                .opacity(0.7)
        }
        .frame(maxWidth: .infinity)
    }
}
```

---

## 60.10 AppIntents สำหรับ Configurable Widgets

```swift
import AppIntents
import WidgetKit

// Step 1: สร้าง Entity
struct TaskListEntity: AppEntity {
    let id: String
    let name: String
    let taskCount: Int
    let color: String
    
    static var typeDisplayRepresentation: TypeDisplayRepresentation = "รายการงาน"
    static var defaultQuery = TaskListQuery()
    
    var displayRepresentation: DisplayRepresentation {
        DisplayRepresentation(title: "\(name) (\(taskCount) งาน)")
    }
}

// Step 2: สร้าง EntityQuery
struct TaskListQuery: EntityQuery {
    func entities(for identifiers: [String]) async throws -> [TaskListEntity] {
        // โหลดจาก Storage
        return TaskListStorage.shared.lists.filter { identifiers.contains($0.id) }
    }
    
    func suggestedEntities() async throws -> [TaskListEntity] {
        return TaskListStorage.shared.lists
    }
    
    func defaultResult() async -> TaskListEntity? {
        return TaskListStorage.shared.lists.first
    }
}

// Step 3: สร้าง Intent
struct TaskListWidgetIntent: WidgetConfigurationIntent {
    static var title: LocalizedStringResource = "เลือกรายการงาน"
    static var description = IntentDescription("เลือกรายการงานที่ต้องการแสดง")
    
    @Parameter(title: "รายการ", default: nil)
    var taskList: TaskListEntity?
    
    @Parameter(title: "แสดงงานสำเร็จ", default: false)
    var showCompleted: Bool
    
    @Parameter(title: "จำนวนสูงสุด", default: 5, inclusiveRange: (1, 10))
    var maxTasks: Int
}

// Step 4: สร้าง Timeline Provider
struct TaskListProvider: AppIntentTimelineProvider {
    typealias Entry = TaskListEntry
    typealias Intent = TaskListWidgetIntent
    
    func placeholder(in context: Context) -> TaskListEntry {
        TaskListEntry.sample
    }
    
    func snapshot(for configuration: TaskListWidgetIntent, in context: Context) async -> TaskListEntry {
        await loadEntry(for: configuration)
    }
    
    func timeline(for configuration: TaskListWidgetIntent, in context: Context) async -> Timeline<TaskListEntry> {
        let entry = await loadEntry(for: configuration)
        let nextUpdate = Calendar.current.date(byAdding: .minute, value: 15, to: Date())!
        return Timeline(entries: [entry], policy: .after(nextUpdate))
    }
    
    private func loadEntry(for configuration: TaskListWidgetIntent) async -> TaskListEntry {
        let listID = configuration.taskList?.id ?? "default"
        let tasks = TaskListStorage.shared.tasks(for: listID, showCompleted: configuration.showCompleted)
        let limited = Array(tasks.prefix(configuration.maxTasks))
        
        return TaskListEntry(
            date: Date(),
            listName: configuration.taskList?.name ?? "รายการงาน",
            tasks: limited,
            totalCount: tasks.count
        )
    }
}

// Task Entry
struct TaskListEntry: TimelineEntry {
    let date: Date
    let listName: String
    let tasks: [Task]
    let totalCount: Int
    
    struct Task: Identifiable {
        let id: UUID
        let title: String
        let isCompleted: Bool
        let priority: Priority
        
        enum Priority {
            case low, medium, high
            
            var color: Color {
                switch self {
                case .low: return .green
                case .medium: return .orange
                case .high: return .red
                }
            }
        }
    }
    
    static var sample: TaskListEntry {
        TaskListEntry(
            date: Date(),
            listName: "งานวันนี้",
            tasks: [
                Task(id: UUID(), title: "ประชุมทีม 10:00", isCompleted: false, priority: .high),
                Task(id: UUID(), title: "ส่งรายงาน", isCompleted: false, priority: .medium),
                Task(id: UUID(), title: "ตอบ email", isCompleted: true, priority: .low),
                Task(id: UUID(), title: "Review code", isCompleted: false, priority: .medium)
            ],
            totalCount: 4
        )
    }
}

// Storage (Shared with App)
class TaskListStorage {
    static let shared = TaskListStorage()
    
    var lists: [TaskListEntity] = [
        TaskListEntity(id: "default", name: "งานทั้งหมด", taskCount: 5, color: "blue"),
        TaskListEntity(id: "today", name: "งานวันนี้", taskCount: 3, color: "green"),
        TaskListEntity(id: "important", name: "สำคัญ", taskCount: 2, color: "red")
    ]
    
    func tasks(for listID: String, showCompleted: Bool) -> [TaskListEntry.Task] {
        // โหลดจาก App Group UserDefaults หรือ Core Data
        return TaskListEntry.sample.tasks.filter { task in
            showCompleted || !task.isCompleted
        }
    }
}
```

---

## 60.11 Widget with Links

```swift
import SwiftUI
import WidgetKit

// widgetURL - เปิดแอปด้วย URL เฉพาะ
struct LinkableWidgetView: View {
    let entry: ArticleEntry
    @Environment(\.widgetFamily) var family
    
    var body: some View {
        Group {
            if family == .systemSmall {
                // Small widget ใช้ widgetURL ได้
                smallView
                    .widgetURL(URL(string: "myapp://article/\(entry.articleID)"))
            } else {
                // Medium/Large ใช้ Link สำหรับแต่ละ item
                mediumView
            }
        }
    }
    
    var smallView: some View {
        VStack(alignment: .leading) {
            Text(entry.category)
                .font(.caption)
                .foregroundColor(.secondary)
            
            Text(entry.title)
                .font(.headline)
                .lineLimit(3)
            
            Spacer()
            
            Text(entry.date, style: .relative)
                .font(.caption2)
                .foregroundColor(.secondary)
        }
        .padding()
    }
    
    var mediumView: some View {
        VStack(alignment: .leading, spacing: 8) {
            Text("บทความล่าสุด")
                .font(.caption)
                .foregroundColor(.secondary)
            
            // ใช้ Link สำหรับแต่ละบทความ
            ForEach(entry.relatedArticles.prefix(2)) { article in
                Link(destination: URL(string: "myapp://article/\(article.id)")!) {
                    HStack {
                        Circle()
                            .fill(article.categoryColor)
                            .frame(width: 8, height: 8)
                        
                        Text(article.title)
                            .font(.subheadline)
                            .lineLimit(1)
                        
                        Spacer()
                        
                        Image(systemName: "chevron.right")
                            .font(.caption)
                            .foregroundColor(.secondary)
                    }
                }
                .buttonStyle(.plain)
            }
        }
        .padding()
        .widgetURL(URL(string: "myapp://articles")) // Fallback URL
    }
}

// รับ URL ใน App
// ใน App @main
@main
struct NewsApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
                .onOpenURL { url in
                    handleWidgetURL(url)
                }
        }
    }
    
    func handleWidgetURL(_ url: URL) {
        guard url.scheme == "myapp" else { return }
        
        switch url.host {
        case "article":
            let articleID = url.pathComponents.last
            print("เปิดบทความ ID: \(articleID ?? "unknown")")
            
        case "articles":
            print("เปิดรายการบทความ")
            
        default:
            print("URL ไม่รู้จัก: \(url)")
        }
    }
}

struct ArticleEntry: TimelineEntry {
    let date: Date
    let articleID: String
    let title: String
    let category: String
    let relatedArticles: [Article]
    
    struct Article: Identifiable {
        let id: String
        let title: String
        let categoryColor: Color
    }
}
```

---

## 60.12 Widget Previews

```swift
import SwiftUI
import WidgetKit

// Preview ใน Xcode
struct WeatherWidget_Previews: PreviewProvider {
    static var previews: some View {
        Group {
            // Preview แต่ละ Family
            WeatherWidgetView(entry: .placeholder)
                .previewContext(WidgetPreviewContext(family: .systemSmall))
                .previewDisplayName("Small")
            
            WeatherWidgetView(entry: .placeholder)
                .previewContext(WidgetPreviewContext(family: .systemMedium))
                .previewDisplayName("Medium")
            
            WeatherWidgetView(entry: .placeholder)
                .previewContext(WidgetPreviewContext(family: .systemLarge))
                .previewDisplayName("Large")
            
            WeatherWidgetView(entry: .placeholder)
                .previewContext(WidgetPreviewContext(family: .accessoryCircular))
                .previewDisplayName("Circular")
                .previewDevice("iPhone 14 Pro")
            
            WeatherWidgetView(entry: .placeholder)
                .previewContext(WidgetPreviewContext(family: .accessoryRectangular))
                .previewDisplayName("Rectangular")
        }
    }
}

// SwiftUI Macro-based Preview (iOS 17+)
#Preview(as: .systemSmall) {
    WeatherWidget()
} timeline: {
    WeatherEntry.placeholder
    WeatherEntry(
        date: Date(),
        temperature: 32,
        condition: .sunny,
        city: "กรุงเทพฯ",
        humidity: 70,
        feelsLike: 38,
        forecast: WeatherEntry.placeholder.forecast
    )
}

#Preview(as: .systemLarge) {
    WeatherWidget()
} timeline: {
    WeatherEntry.placeholder
}
```

---

## 60.13 Widget Bundle

Widget Bundle ช่วยให้แอปมีหลาย Widget ใน Extension เดียว

```swift
import WidgetKit
import SwiftUI

// รวม Widgets ทั้งหมด
@main
struct MyAppWidgetBundle: WidgetBundle {
    var body: some Widget {
        // Weather Widget
        WeatherWidget()
        
        // Task List Widget
        TaskListWidget()
        
        // Clock Widget
        ClockWidget()
        
        // News Widget
        NewsWidget()
        
        // Stock Widget
        StockWidget()
    }
}

// Task List Widget
struct TaskListWidget: Widget {
    let kind = "TaskListWidget"
    
    var body: some WidgetConfiguration {
        AppIntentConfiguration(
            kind: kind,
            intent: TaskListWidgetIntent.self,
            provider: TaskListProvider()
        ) { entry in
            TaskListWidgetView(entry: entry)
                .containerBackground(.fill.tertiary, for: .widget)
        }
        .configurationDisplayName("รายการงาน")
        .description("แสดงงานที่ต้องทำวันนี้")
        .supportedFamilies([.systemSmall, .systemMedium, .systemLarge])
    }
}

// Stock Widget
struct StockWidget: Widget {
    let kind = "StockWidget"
    
    var body: some WidgetConfiguration {
        AppIntentConfiguration(
            kind: kind,
            intent: StockSelectionIntent.self,
            provider: StockProvider()
        ) { entry in
            StockWidgetView(entry: entry)
                .containerBackground(.fill.tertiary, for: .widget)
        }
        .configurationDisplayName("ราคาหุ้น")
        .description("ติดตามราคาหุ้นที่คุณสนใจ")
        .supportedFamilies([.systemSmall, .systemMedium])
    }
}

struct StockSelectionIntent: WidgetConfigurationIntent {
    static var title: LocalizedStringResource = "เลือกหุ้น"
    static var description = IntentDescription("เลือกหุ้นที่ต้องการติดตาม")
    
    @Parameter(title: "สัญลักษณ์หุ้น")
    var symbol: String
    
    init() { self.symbol = "AAPL" }
    init(symbol: String) { self.symbol = symbol }
}
```

---

## 60.14 Interactive Widgets (iOS 17+)

iOS 17 ทำให้ Widget มีปุ่มที่กดได้และ Toggle

```swift
import SwiftUI
import WidgetKit
import AppIntents

// AppIntent สำหรับ Widget Action
struct ToggleTaskIntent: AppIntent {
    static var title: LocalizedStringResource = "สลับสถานะงาน"
    
    @Parameter(title: "Task ID")
    var taskID: String
    
    init() { self.taskID = "" }
    init(taskID: String) { self.taskID = taskID }
    
    func perform() async throws -> some IntentResult {
        // อัพเดทสถานะงาน
        TaskListStorage.shared.toggleTask(id: taskID)
        
        // รีโหลด Widget
        WidgetCenter.shared.reloadTimelines(ofKind: "TaskListWidget")
        
        return .result()
    }
}

struct AddTaskIntent: AppIntent {
    static var title: LocalizedStringResource = "เพิ่มงานใหม่"
    static var openAppWhenRun: Bool = true
    
    func perform() async throws -> some IntentResult {
        return .result()
    }
}

// Interactive Task Widget View
struct InteractiveTaskWidgetView: View {
    let entry: TaskListEntry
    @Environment(\.widgetFamily) var family
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            // Header
            HStack {
                Text(entry.listName)
                    .font(.headline)
                
                Spacer()
                
                // Interactive Button - เพิ่มงานใหม่
                Button(intent: AddTaskIntent()) {
                    Image(systemName: "plus.circle.fill")
                        .foregroundColor(.accentColor)
                }
                .buttonStyle(.plain)
            }
            
            Divider()
            
            // Task List with Toggle
            ForEach(entry.tasks) { task in
                HStack(spacing: 8) {
                    // Interactive Toggle Button
                    Button(intent: ToggleTaskIntent(taskID: task.id.uuidString)) {
                        Image(systemName: task.isCompleted ? "checkmark.circle.fill" : "circle")
                            .foregroundColor(task.isCompleted ? .green : .secondary)
                    }
                    .buttonStyle(.plain)
                    
                    Text(task.title)
                        .strikethrough(task.isCompleted)
                        .foregroundColor(task.isCompleted ? .secondary : .primary)
                        .lineLimit(1)
                    
                    Spacer()
                    
                    // Priority Indicator
                    Circle()
                        .fill(task.priority.color)
                        .frame(width: 6, height: 6)
                }
            }
            
            if entry.tasks.isEmpty {
                Text("ไม่มีงานที่ต้องทำ 🎉")
                    .foregroundColor(.secondary)
                    .font(.subheadline)
            }
            
            Spacer()
            
            // Footer
            Text("\(entry.tasks.filter { !$0.isCompleted }.count) งานที่เหลือ")
                .font(.caption2)
                .foregroundColor(.secondary)
        }
        .padding()
    }
}

// Toggle View ใน Widget (iOS 17+)
struct LampControlWidget: Widget {
    let kind = "LampControl"
    
    var body: some WidgetConfiguration {
        StaticConfiguration(kind: kind, provider: LampProvider()) { entry in
            LampControlView(entry: entry)
                .containerBackground(.fill.tertiary, for: .widget)
        }
        .configurationDisplayName("ควบคุมไฟ")
        .supportedFamilies([.systemSmall])
    }
}

struct LampEntry: TimelineEntry {
    let date: Date
    var isOn: Bool
    var brightness: Double
}

struct ToggleLampIntent: AppIntent {
    static var title: LocalizedStringResource = "สลับไฟ"
    
    func perform() async throws -> some IntentResult {
        LampController.shared.toggle()
        WidgetCenter.shared.reloadTimelines(ofKind: "LampControl")
        return .result()
    }
}

struct LampControlView: View {
    let entry: LampEntry
    
    var body: some View {
        VStack(spacing: 12) {
            Image(systemName: entry.isOn ? "lightbulb.fill" : "lightbulb")
                .font(.largeTitle)
                .foregroundColor(entry.isOn ? .yellow : .secondary)
            
            // Interactive Toggle
            Toggle(isOn: entry.isOn, intent: ToggleLampIntent()) {
                Text(entry.isOn ? "เปิด" : "ปิด")
            }
            .toggleStyle(.button)
            .tint(.yellow)
        }
    }
}

class LampController {
    static let shared = LampController()
    
    private(set) var isOn: Bool = false
    
    func toggle() {
        isOn.toggle()
        // บันทึกสถานะ
        UserDefaults(suiteName: "group.example.app")?.set(isOn, forKey: "lampState")
    }
}

struct LampProvider: TimelineProvider {
    typealias Entry = LampEntry
    
    func placeholder(in context: Context) -> LampEntry {
        LampEntry(date: Date(), isOn: true, brightness: 0.8)
    }
    
    func getSnapshot(in context: Context, completion: @escaping (LampEntry) -> Void) {
        let state = UserDefaults(suiteName: "group.example.app")?.bool(forKey: "lampState") ?? false
        completion(LampEntry(date: Date(), isOn: state, brightness: 0.8))
    }
    
    func getTimeline(in context: Context, completion: @escaping (Timeline<LampEntry>) -> Void) {
        let state = UserDefaults(suiteName: "group.example.app")?.bool(forKey: "lampState") ?? false
        let entry = LampEntry(date: Date(), isOn: state, brightness: 0.8)
        let timeline = Timeline(entries: [entry], policy: .never)
        completion(timeline)
    }
}
```

---

## 60.15 Live Activities

Live Activities แสดงข้อมูลแบบ real-time บน Lock Screen และ Dynamic Island

```swift
import ActivityKit
import SwiftUI

// 1. สร้าง Activity Attributes
struct DeliveryActivityAttributes: ActivityAttributes {
    // Static content (ไม่เปลี่ยน)
    let orderNumber: String
    let restaurantName: String
    let estimatedTime: Date
    
    // Dynamic content (เปลี่ยนได้)
    struct ContentState: Codable, Hashable {
        var currentStatus: DeliveryStatus
        var driverName: String
        var currentLocation: String
        var progress: Double // 0.0 - 1.0
        var minutesLeft: Int
        
        enum DeliveryStatus: String, Codable, CaseIterable {
            case preparing = "กำลังเตรียม"
            case cooking = "กำลังทำ"
            case ready = "พร้อมส่ง"
            case onTheWay = "กำลังส่ง"
            case nearBy = "ใกล้ถึงแล้ว"
            case delivered = "ส่งแล้ว"
            
            var icon: String {
                switch self {
                case .preparing: return "clock"
                case .cooking: return "flame"
                case .ready: return "bag.fill"
                case .onTheWay: return "bicycle"
                case .nearBy: return "location.fill"
                case .delivered: return "checkmark.circle.fill"
                }
            }
        }
    }
}

// 2. เริ่มต้น Live Activity
class DeliveryActivityManager {
    
    static var currentActivity: Activity<DeliveryActivityAttributes>?
    
    static func startActivity(for order: Order) async {
        guard ActivityAuthorizationInfo().areActivitiesEnabled else {
            print("Live Activities ไม่ได้รับอนุญาต")
            return
        }
        
        let attributes = DeliveryActivityAttributes(
            orderNumber: order.id,
            restaurantName: order.restaurantName,
            estimatedTime: order.estimatedDeliveryTime
        )
        
        let initialState = DeliveryActivityAttributes.ContentState(
            currentStatus: .preparing,
            driverName: "กำลังมอบหมาย",
            currentLocation: "ร้านอาหาร",
            progress: 0.1,
            minutesLeft: 30
        )
        
        let content = ActivityContent(
            state: initialState,
            staleDate: Calendar.current.date(byAdding: .minute, value: 60, to: Date())
        )
        
        do {
            currentActivity = try Activity.request(
                attributes: attributes,
                content: content,
                pushType: .token // ใช้ Push Notification อัพเดท
            )
            print("เริ่ม Live Activity: \(currentActivity?.id ?? "unknown")")
        } catch {
            print("ไม่สามารถเริ่ม Live Activity: \(error)")
        }
    }
    
    // อัพเดท Live Activity
    static func updateActivity(status: DeliveryActivityAttributes.ContentState.DeliveryStatus,
                               driverName: String,
                               minutesLeft: Int,
                               progress: Double) async {
        let updatedState = DeliveryActivityAttributes.ContentState(
            currentStatus: status,
            driverName: driverName,
            currentLocation: "กำลังเดินทาง",
            progress: progress,
            minutesLeft: minutesLeft
        )
        
        let content = ActivityContent(
            state: updatedState,
            staleDate: Calendar.current.date(byAdding: .minute, value: minutesLeft + 5, to: Date())
        )
        
        await currentActivity?.update(content)
    }
    
    // สิ้นสุด Live Activity
    static func endActivity() async {
        let finalState = DeliveryActivityAttributes.ContentState(
            currentStatus: .delivered,
            driverName: "สำเร็จ",
            currentLocation: "ส่งแล้ว",
            progress: 1.0,
            minutesLeft: 0
        )
        
        let content = ActivityContent(state: finalState, staleDate: nil)
        
        await currentActivity?.end(content, dismissalPolicy: .after(
            Calendar.current.date(byAdding: .minute, value: 5, to: Date())!
        ))
        
        currentActivity = nil
    }
}

// Order Model
struct Order {
    let id: String
    let restaurantName: String
    let estimatedDeliveryTime: Date
}
```

---

## 60.16 ActivityKit

```swift
import ActivityKit
import SwiftUI

// Live Activity Views
struct DeliveryLiveActivityView: View {
    let attributes: DeliveryActivityAttributes
    let state: DeliveryActivityAttributes.ContentState
    
    var body: some View {
        VStack(spacing: 8) {
            // Header
            HStack {
                Image(systemName: "bag.fill")
                    .foregroundColor(.orange)
                
                Text("คำสั่ง #\(attributes.orderNumber)")
                    .font(.headline)
                
                Spacer()
                
                Text("\(state.minutesLeft) นาที")
                    .font(.subheadline)
                    .foregroundColor(.secondary)
            }
            
            // Status
            HStack(spacing: 4) {
                Image(systemName: state.currentStatus.icon)
                    .foregroundColor(.accentColor)
                
                Text(state.currentStatus.rawValue)
                    .font(.subheadline)
                
                Spacer()
                
                Text(state.driverName)
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
            
            // Progress Bar
            GeometryReader { geometry in
                ZStack(alignment: .leading) {
                    RoundedRectangle(cornerRadius: 4)
                        .fill(.secondary.opacity(0.2))
                        .frame(height: 6)
                    
                    RoundedRectangle(cornerRadius: 4)
                        .fill(.orange)
                        .frame(width: geometry.size.width * state.progress, height: 6)
                }
            }
            .frame(height: 6)
            
            // ETA
            HStack {
                Text("คาดว่าจะถึง:")
                    .font(.caption)
                    .foregroundColor(.secondary)
                
                Text(attributes.estimatedTime, style: .time)
                    .font(.caption)
                    .fontWeight(.semibold)
                
                Spacer()
                
                Text("จาก \(attributes.restaurantName)")
                    .font(.caption2)
                    .foregroundColor(.secondary)
            }
        }
        .padding()
    }
}
```

---

## 60.17 Dynamic Island

Dynamic Island แสดงข้อมูลรอบๆ กล้องหน้าบน iPhone 14 Pro ขึ้นไป

```swift
import ActivityKit
import SwiftUI

// Widget Configuration for Dynamic Island
struct DeliveryActivityWidget: Widget {
    var body: some WidgetConfiguration {
        ActivityConfiguration(for: DeliveryActivityAttributes.self) { context in
            // Lock Screen / Banner View
            DeliveryLiveActivityView(
                attributes: context.attributes,
                state: context.state
            )
            .activityBackgroundTint(.black)
            .activitySystemActionForegroundColor(.white)
            
        } dynamicIsland: { context in
            // Dynamic Island
            DynamicIsland {
                // Expanded Region
                DynamicIslandExpandedRegion(.leading) {
                    // ซ้าย
                    HStack {
                        Image(systemName: context.state.currentStatus.icon)
                            .foregroundColor(.orange)
                        Text(context.state.currentStatus.rawValue)
                            .font(.caption)
                    }
                }
                
                DynamicIslandExpandedRegion(.trailing) {
                    // ขวา
                    Text("\(context.state.minutesLeft) นาที")
                        .font(.caption)
                        .foregroundColor(.secondary)
                }
                
                DynamicIslandExpandedRegion(.center) {
                    // กลาง
                    Text("คำสั่ง #\(context.attributes.orderNumber)")
                        .font(.headline)
                }
                
                DynamicIslandExpandedRegion(.bottom) {
                    // ด้านล่าง
                    ProgressView(value: context.state.progress)
                        .tint(.orange)
                        .padding(.horizontal)
                }
                
            } compactLeading: {
                // Compact - ซ้าย
                Image(systemName: "bag.fill")
                    .foregroundColor(.orange)
                
            } compactTrailing: {
                // Compact - ขวา
                Text("\(context.state.minutesLeft)m")
                    .font(.caption2)
                    .foregroundColor(.orange)
                
            } minimal: {
                // Minimal (เมื่อมีหลาย Activities)
                Image(systemName: context.state.currentStatus.icon)
                    .foregroundColor(.orange)
            }
            .keylineTint(.orange)
        }
    }
}
```

---

## 60.18 CompactLeading, CompactTrailing, Expanded, Minimal

```swift
import SwiftUI
import ActivityKit

// Sports Score Dynamic Island
struct SportsScoreAttributes: ActivityAttributes {
    let homeTeam: String
    let awayTeam: String
    let sport: String
    
    struct ContentState: Codable, Hashable {
        var homeScore: Int
        var awayScore: Int
        var period: String
        var timeRemaining: String
        var isLive: Bool
    }
}

struct SportsScoreWidget: Widget {
    var body: some WidgetConfiguration {
        ActivityConfiguration(for: SportsScoreAttributes.self) { context in
            // Lock Screen View
            HStack {
                VStack {
                    Text(context.attributes.homeTeam)
                        .font(.caption)
                    Text("\(context.state.homeScore)")
                        .font(.largeTitle)
                        .fontWeight(.bold)
                }
                
                VStack {
                    Text(context.state.period)
                        .font(.caption)
                        .foregroundColor(.secondary)
                    Text(context.state.timeRemaining)
                        .font(.subheadline)
                        .foregroundColor(.red)
                    if context.state.isLive {
                        Circle()
                            .fill(.red)
                            .frame(width: 6, height: 6)
                    }
                }
                
                VStack {
                    Text(context.attributes.awayTeam)
                        .font(.caption)
                    Text("\(context.state.awayScore)")
                        .font(.largeTitle)
                        .fontWeight(.bold)
                }
            }
            .padding()
            .activityBackgroundTint(.black)
            
        } dynamicIsland: { context in
            DynamicIsland {
                // Expanded
                DynamicIslandExpandedRegion(.leading) {
                    VStack {
                        Text(context.attributes.homeTeam)
                            .font(.caption2)
                        Text("\(context.state.homeScore)")
                            .font(.title2)
                            .fontWeight(.bold)
                    }
                }
                
                DynamicIslandExpandedRegion(.center) {
                    VStack(spacing: 2) {
                        Text(context.state.period)
                            .font(.caption)
                        if context.state.isLive {
                            HStack(spacing: 2) {
                                Circle()
                                    .fill(.red)
                                    .frame(width: 4, height: 4)
                                Text("LIVE")
                                    .font(.caption2)
                                    .foregroundColor(.red)
                            }
                        }
                        Text(context.state.timeRemaining)
                            .font(.caption2)
                            .foregroundColor(.secondary)
                    }
                }
                
                DynamicIslandExpandedRegion(.trailing) {
                    VStack {
                        Text(context.attributes.awayTeam)
                            .font(.caption2)
                        Text("\(context.state.awayScore)")
                            .font(.title2)
                            .fontWeight(.bold)
                    }
                }
                
                DynamicIslandExpandedRegion(.bottom) {
                    Text(context.attributes.sport)
                        .font(.caption2)
                        .foregroundColor(.secondary)
                }
                
            } compactLeading: {
                // แสดงคะแนนฝั่งเหย้า
                HStack(spacing: 2) {
                    Text(context.attributes.homeTeam.prefix(3))
                        .font(.caption2)
                    Text("\(context.state.homeScore)")
                        .font(.caption)
                        .fontWeight(.bold)
                }
                
            } compactTrailing: {
                // แสดงคะแนนฝั่งเยือน
                HStack(spacing: 2) {
                    Text("\(context.state.awayScore)")
                        .font(.caption)
                        .fontWeight(.bold)
                    Text(context.attributes.awayTeam.prefix(3))
                        .font(.caption2)
                }
                
            } minimal: {
                // แสดงคะแนนรวม
                Text("\(context.state.homeScore)-\(context.state.awayScore)")
                    .font(.caption2)
                    .fontWeight(.bold)
            }
        }
    }
}
```

---

## 60.19 Lock Screen Widgets

Lock Screen Widgets ปรากฏบน Lock Screen ของ iPhone

```swift
import SwiftUI
import WidgetKit

// Lock Screen Widget
struct LockScreenBatteryWidget: Widget {
    let kind = "LockScreenBattery"
    
    var body: some WidgetConfiguration {
        StaticConfiguration(kind: kind, provider: BatteryProvider()) { entry in
            BatteryWidgetView(entry: entry)
                .containerBackground(.fill.tertiary, for: .widget)
        }
        .configurationDisplayName("แบตเตอรี่")
        .description("แสดงระดับแบตเตอรี่")
        .supportedFamilies([
            .accessoryCircular,
            .accessoryRectangular,
            .accessoryInline
        ])
    }
}

struct BatteryEntry: TimelineEntry {
    let date: Date
    let batteryLevel: Double // 0.0 - 1.0
    let isCharging: Bool
    let timeRemaining: String
    
    var batteryIcon: String {
        if isCharging { return "battery.100.bolt" }
        switch batteryLevel {
        case 0.75...: return "battery.100"
        case 0.5...: return "battery.75"
        case 0.25...: return "battery.50"
        case 0.1...: return "battery.25"
        default: return "battery.0"
        }
    }
    
    var batteryColor: Color {
        if isCharging { return .green }
        if batteryLevel > 0.2 { return .green }
        if batteryLevel > 0.1 { return .yellow }
        return .red
    }
}

struct BatteryWidgetView: View {
    let entry: BatteryEntry
    @Environment(\.widgetFamily) var family
    
    var body: some View {
        switch family {
        case .accessoryCircular:
            circularView
        case .accessoryRectangular:
            rectangularView
        case .accessoryInline:
            inlineView
        default:
            circularView
        }
    }
    
    var circularView: some View {
        ZStack {
            AccessoryWidgetBackground()
            
            VStack(spacing: 2) {
                Image(systemName: entry.batteryIcon)
                    .font(.title3)
                    .foregroundColor(entry.batteryColor)
                
                Text("\(Int(entry.batteryLevel * 100))%")
                    .font(.caption2)
                    .fontWeight(.semibold)
            }
        }
    }
    
    var rectangularView: some View {
        HStack(spacing: 8) {
            // Circular Battery Gauge
            ZStack {
                Circle()
                    .stroke(.secondary.opacity(0.3), lineWidth: 4)
                
                Circle()
                    .trim(from: 0, to: entry.batteryLevel)
                    .stroke(entry.batteryColor, style: StrokeStyle(lineWidth: 4, lineCap: .round))
                    .rotationEffect(.degrees(-90))
                
                Image(systemName: entry.isCharging ? "bolt.fill" : "")
                    .font(.caption)
                    .foregroundColor(.yellow)
            }
            .frame(width: 30, height: 30)
            
            VStack(alignment: .leading, spacing: 2) {
                Text("\(Int(entry.batteryLevel * 100))%")
                    .font(.headline)
                    .foregroundColor(entry.batteryColor)
                
                Text(entry.isCharging ? "กำลังชาร์จ" : entry.timeRemaining)
                    .font(.caption2)
                    .foregroundColor(.secondary)
            }
        }
    }
    
    var inlineView: some View {
        Label("\(Int(entry.batteryLevel * 100))%", systemImage: entry.batteryIcon)
    }
}

struct BatteryProvider: TimelineProvider {
    typealias Entry = BatteryEntry
    
    func placeholder(in context: Context) -> BatteryEntry {
        BatteryEntry(date: Date(), batteryLevel: 0.75, isCharging: false, timeRemaining: "5 ชั่วโมง")
    }
    
    func getSnapshot(in context: Context, completion: @escaping (BatteryEntry) -> Void) {
        completion(placeholder(in: context))
    }
    
    func getTimeline(in context: Context, completion: @escaping (Timeline<BatteryEntry>) -> Void) {
        // อ่านข้อมูลแบตเตอรี่จาก UIDevice
        let level = UIDevice.current.batteryLevel
        let isCharging = UIDevice.current.batteryState == .charging
        
        let entry = BatteryEntry(
            date: Date(),
            batteryLevel: max(0, Double(level)),
            isCharging: isCharging,
            timeRemaining: "3 ชั่วโมง"
        )
        
        // อัพเดททุก 5 นาที
        let nextUpdate = Calendar.current.date(byAdding: .minute, value: 5, to: Date())!
        let timeline = Timeline(entries: [entry], policy: .after(nextUpdate))
        completion(timeline)
    }
}
```

---

## 60.20 Sharing Data กับ App Group

App Group ช่วยให้แอปหลักและ Widget แชร์ข้อมูลกันได้

```swift
import Foundation
import WidgetKit

// MARK: - App Group Configuration

// กำหนดใน Entitlements ทั้ง Main App และ Widget Extension:
// com.apple.security.application-groups: ["group.com.example.app"]

// MARK: - Shared Data Store

class SharedDataStore {
    static let shared = SharedDataStore()
    
    private let groupID = "group.com.example.app"
    private var defaults: UserDefaults?
    
    init() {
        defaults = UserDefaults(suiteName: groupID)
    }
    
    // MARK: - UserDefaults Methods
    
    func save<T: Codable>(_ value: T, for key: String) {
        guard let defaults = defaults else { return }
        
        if let encoded = try? JSONEncoder().encode(value) {
            defaults.set(encoded, forKey: key)
            defaults.synchronize()
            
            // รีโหลด Widget
            WidgetCenter.shared.reloadAllTimelines()
        }
    }
    
    func load<T: Codable>(_ type: T.Type, for key: String) -> T? {
        guard let defaults = defaults,
              let data = defaults.data(forKey: key),
              let decoded = try? JSONDecoder().decode(type, from: data)
        else { return nil }
        
        return decoded
    }
    
    // MARK: - File-based Storage
    
    var sharedContainerURL: URL? {
        FileManager.default.containerURL(forSecurityApplicationGroupIdentifier: groupID)
    }
    
    func saveFile<T: Codable>(_ value: T, named filename: String) {
        guard let containerURL = sharedContainerURL,
              let encoded = try? JSONEncoder().encode(value)
        else { return }
        
        let fileURL = containerURL.appendingPathComponent(filename)
        try? encoded.write(to: fileURL)
        
        WidgetCenter.shared.reloadAllTimelines()
    }
    
    func loadFile<T: Codable>(_ type: T.Type, named filename: String) -> T? {
        guard let containerURL = sharedContainerURL else { return nil }
        
        let fileURL = containerURL.appendingPathComponent(filename)
        guard let data = try? Data(contentsOf: fileURL),
              let decoded = try? JSONDecoder().decode(type, from: data)
        else { return nil }
        
        return decoded
    }
}

// MARK: - Data Models (Shared)

struct WidgetData: Codable {
    var tasks: [TodoTask]
    var lastUpdated: Date
    
    struct TodoTask: Codable, Identifiable {
        let id: UUID
        var title: String
        var isCompleted: Bool
        var dueDate: Date?
        var priority: Int // 1-3
        
        init(title: String, priority: Int = 2) {
            self.id = UUID()
            self.title = title
            self.isCompleted = false
            self.priority = priority
        }
    }
}

// ใช้งานใน Main App
class TodoViewModel: ObservableObject {
    @Published var tasks: [WidgetData.TodoTask] = []
    
    private let storage = SharedDataStore.shared
    private let dataKey = "widget_todo_data"
    
    init() {
        loadTasks()
    }
    
    func loadTasks() {
        let data = storage.load(WidgetData.self, for: dataKey)
        tasks = data?.tasks ?? []
    }
    
    func addTask(title: String, priority: Int = 2) {
        let task = WidgetData.TodoTask(title: title, priority: priority)
        tasks.append(task)
        saveTasks()
    }
    
    func toggleTask(id: UUID) {
        if let index = tasks.firstIndex(where: { $0.id == id }) {
            tasks[index].isCompleted.toggle()
            saveTasks()
        }
    }
    
    func deleteTasks(at offsets: IndexSet) {
        tasks.remove(atOffsets: offsets)
        saveTasks()
    }
    
    private func saveTasks() {
        let data = WidgetData(tasks: tasks, lastUpdated: Date())
        storage.save(data, for: dataKey)
    }
}

// ใช้งานใน Widget
struct TodoWidgetProvider: TimelineProvider {
    typealias Entry = TodoWidgetEntry
    
    private let storage = SharedDataStore.shared
    private let dataKey = "widget_todo_data"
    
    func placeholder(in context: Context) -> TodoWidgetEntry {
        TodoWidgetEntry.sample
    }
    
    func getSnapshot(in context: Context, completion: @escaping (TodoWidgetEntry) -> Void) {
        completion(loadEntry())
    }
    
    func getTimeline(in context: Context, completion: @escaping (Timeline<TodoWidgetEntry>) -> Void) {
        let entry = loadEntry()
        // ไม่มี automatic reload - อัพเดทเมื่อแอปหลักบันทึกข้อมูล
        let timeline = Timeline(entries: [entry], policy: .never)
        completion(timeline)
    }
    
    private func loadEntry() -> TodoWidgetEntry {
        let data = storage.load(WidgetData.self, for: dataKey)
        let tasks = data?.tasks ?? []
        let pendingTasks = tasks.filter { !$0.isCompleted }
        
        return TodoWidgetEntry(
            date: Date(),
            tasks: Array(pendingTasks.prefix(5)),
            totalPending: pendingTasks.count,
            lastUpdated: data?.lastUpdated ?? Date()
        )
    }
}

struct TodoWidgetEntry: TimelineEntry {
    let date: Date
    let tasks: [WidgetData.TodoTask]
    let totalPending: Int
    let lastUpdated: Date
    
    static var sample: TodoWidgetEntry {
        TodoWidgetEntry(
            date: Date(),
            tasks: [
                WidgetData.TodoTask(title: "ประชุมทีม 10:00", priority: 3),
                WidgetData.TodoTask(title: "ส่งรายงาน", priority: 2),
                WidgetData.TodoTask(title: "ตอบ email", priority: 1)
            ],
            totalPending: 3,
            lastUpdated: Date()
        )
    }
}
```

---

## 60.21 UserDefaults กับ App Group

```swift
import Foundation
import WidgetKit

// Helper สำหรับ App Group UserDefaults
extension UserDefaults {
    
    static var appGroup: UserDefaults {
        return UserDefaults(suiteName: "group.com.example.myapp") ?? .standard
    }
    
    // Typed Properties
    var widgetLastRefresh: Date {
        get { return (object(forKey: "widgetLastRefresh") as? Date) ?? Date() }
        set { set(newValue, forKey: "widgetLastRefresh") }
    }
    
    var taskCount: Int {
        get { return integer(forKey: "taskCount") }
        set { set(newValue, forKey: "taskCount") }
    }
    
    var userName: String {
        get { return string(forKey: "userName") ?? "ผู้ใช้" }
        set { set(newValue, forKey: "userName") }
    }
    
    var isPremium: Bool {
        get { return bool(forKey: "isPremium") }
        set { set(newValue, forKey: "isPremium") }
    }
    
    // Codable Support
    func setCodable<T: Codable>(_ value: T, forKey key: String) {
        if let data = try? JSONEncoder().encode(value) {
            set(data, forKey: key)
        }
    }
    
    func codable<T: Codable>(_ type: T.Type, forKey key: String) -> T? {
        guard let data = data(forKey: key) else { return nil }
        return try? JSONDecoder().decode(type, from: data)
    }
}

// ใช้งาน
class AppSettingsSync {
    
    // บันทึกจาก Main App
    static func updateWidgetData(taskCount: Int, userName: String) {
        let defaults = UserDefaults.appGroup
        defaults.taskCount = taskCount
        defaults.userName = userName
        defaults.widgetLastRefresh = Date()
        defaults.synchronize()
        
        // รีโหลด Widgets
        WidgetCenter.shared.reloadAllTimelines()
    }
    
    // อ่านใน Widget
    static func loadWidgetData() -> (taskCount: Int, userName: String) {
        let defaults = UserDefaults.appGroup
        return (defaults.taskCount, defaults.userName)
    }
}
```

---

## 60.22 Practical Exercises

### แบบฝึกหัดที่ 1: สร้าง Quote of the Day Widget

```swift
import WidgetKit
import SwiftUI

// Quote Model
struct Quote: Codable {
    let text: String
    let author: String
    let category: String
    let date: Date
    
    static var sampleQuotes: [Quote] = [
        Quote(text: "ความสำเร็จคือผลรวมของความพยายามเล็กๆ ที่ทำซ้ำแล้วซ้ำเล่า", author: "โรเบิร์ต คอลิเออร์", category: "แรงบันดาลใจ", date: Date()),
        Quote(text: "การเดินทางพันลี้เริ่มต้นจากก้าวแรก", author: "เล่าจื๊อ", category: "ปัญญา", date: Date()),
        Quote(text: "จงเป็นการเปลี่ยนแปลงที่คุณอยากเห็นในโลก", author: "มหาตมะ คานธี", category: "แรงบันดาลใจ", date: Date()),
        Quote(text: "ชีวิตคือสิ่งที่เกิดขึ้นระหว่างที่คุณกำลังวางแผนอื่น", author: "จอห์น เลนนอน", category: "ปรัชญา", date: Date())
    ]
    
    // เลือก Quote ตามวันที่
    static func todaysQuote() -> Quote {
        let dayOfYear = Calendar.current.ordinality(of: .day, in: .year, for: Date()) ?? 1
        let index = dayOfYear % sampleQuotes.count
        return sampleQuotes[index]
    }
}

// Timeline Entry
struct QuoteEntry: TimelineEntry {
    let date: Date
    let quote: Quote
}

// Timeline Provider
struct QuoteProvider: TimelineProvider {
    typealias Entry = QuoteEntry
    
    func placeholder(in context: Context) -> QuoteEntry {
        QuoteEntry(date: Date(), quote: Quote.sampleQuotes[0])
    }
    
    func getSnapshot(in context: Context, completion: @escaping (QuoteEntry) -> Void) {
        completion(QuoteEntry(date: Date(), quote: Quote.todaysQuote()))
    }
    
    func getTimeline(in context: Context, completion: @escaping (Timeline<QuoteEntry>) -> Void) {
        let quote = Quote.todaysQuote()
        let entry = QuoteEntry(date: Date(), quote: quote)
        
        // อัพเดทเที่ยงคืน
        var nextMidnight = Calendar.current.startOfDay(for: Date())
        nextMidnight = Calendar.current.date(byAdding: .day, value: 1, to: nextMidnight)!
        
        let timeline = Timeline(entries: [entry], policy: .after(nextMidnight))
        completion(timeline)
    }
}

// Widget Views
struct QuoteWidgetView: View {
    let entry: QuoteEntry
    @Environment(\.widgetFamily) var family
    
    var body: some View {
        ZStack {
            // Background
            LinearGradient(
                colors: [Color(hex: "#667eea"), Color(hex: "#764ba2")],
                startPoint: .topLeading,
                endPoint: .bottomTrailing
            )
            
            switch family {
            case .systemSmall:
                smallView
            case .systemMedium:
                mediumView
            case .systemLarge:
                largeView
            default:
                mediumView
            }
        }
        .foregroundColor(.white)
    }
    
    var smallView: some View {
        VStack(alignment: .leading, spacing: 8) {
            Image(systemName: "quote.bubble.fill")
                .font(.title2)
                .opacity(0.7)
            
            Spacer()
            
            Text(entry.quote.text)
                .font(.caption)
                .lineLimit(4)
                .italic()
            
            Text("— \(entry.quote.author)")
                .font(.caption2)
                .opacity(0.7)
        }
        .padding()
    }
    
    var mediumView: some View {
        HStack(spacing: 12) {
            // Left Decoration
            Rectangle()
                .fill(.white.opacity(0.3))
                .frame(width: 3)
                .cornerRadius(1.5)
            
            VStack(alignment: .leading, spacing: 8) {
                Image(systemName: "quote.bubble.fill")
                    .font(.title3)
                    .opacity(0.7)
                
                Text(entry.quote.text)
                    .font(.subheadline)
                    .lineLimit(3)
                    .italic()
                
                HStack {
                    Text("— \(entry.quote.author)")
                        .font(.caption)
                        .opacity(0.7)
                    
                    Spacer()
                    
                    Text(entry.quote.category)
                        .font(.caption2)
                        .padding(.horizontal, 6)
                        .padding(.vertical, 2)
                        .background(.white.opacity(0.2))
                        .cornerRadius(6)
                }
            }
        }
        .padding()
    }
    
    var largeView: some View {
        VStack(alignment: .leading, spacing: 16) {
            HStack {
                Image(systemName: "quote.bubble.fill")
                    .font(.largeTitle)
                    .opacity(0.5)
                
                Spacer()
                
                Text(entry.quote.category)
                    .font(.subheadline)
                    .padding(.horizontal, 10)
                    .padding(.vertical, 4)
                    .background(.white.opacity(0.2))
                    .cornerRadius(10)
            }
            
            Text(entry.quote.text)
                .font(.title3)
                .italic()
                .lineLimit(8)
            
            Spacer()
            
            Divider()
                .background(.white.opacity(0.3))
            
            HStack {
                Text("— \(entry.quote.author)")
                    .font(.subheadline)
                    .opacity(0.8)
                
                Spacer()
                
                Text(entry.date, style: .date)
                    .font(.caption)
                    .opacity(0.6)
            }
        }
        .padding()
    }
}

// Color Extension
extension Color {
    init(hex: String) {
        let hex = hex.trimmingCharacters(in: CharacterSet.alphanumerics.inverted)
        var int: UInt64 = 0
        Scanner(string: hex).scanHexInt64(&int)
        let a, r, g, b: UInt64
        switch hex.count {
        case 3:
            (a, r, g, b) = (255, (int >> 8) * 17, (int >> 4 & 0xF) * 17, (int & 0xF) * 17)
        case 6:
            (a, r, g, b) = (255, int >> 16, int >> 8 & 0xFF, int & 0xFF)
        case 8:
            (a, r, g, b) = (int >> 24, int >> 16 & 0xFF, int >> 8 & 0xFF, int & 0xFF)
        default:
            (a, r, g, b) = (255, 0, 0, 0)
        }
        self.init(
            .sRGB,
            red: Double(r) / 255,
            green: Double(g) / 255,
            blue: Double(b) / 255,
            opacity: Double(a) / 255
        )
    }
}

// Widget Definition
struct QuoteWidget: Widget {
    let kind = "QuoteWidget"
    
    var body: some WidgetConfiguration {
        StaticConfiguration(kind: kind, provider: QuoteProvider()) { entry in
            QuoteWidgetView(entry: entry)
                .containerBackground(.fill.tertiary, for: .widget)
        }
        .configurationDisplayName("คำคมประจำวัน")
        .description("คำคมสร้างแรงบันดาลใจทุกวัน")
        .supportedFamilies([.systemSmall, .systemMedium, .systemLarge])
    }
}

// Previews
#Preview(as: .systemMedium) {
    QuoteWidget()
} timeline: {
    QuoteEntry(date: Date(), quote: Quote.sampleQuotes[0])
    QuoteEntry(date: Date(), quote: Quote.sampleQuotes[1])
}
```

---

## 60.23 Building a Custom Widget (Complete Example)

```swift
import WidgetKit
import SwiftUI
import AppIntents

// MARK: - Complete Habit Tracker Widget

// Data Models
struct HabitData: Codable {
    var habits: [Habit]
    var lastUpdated: Date
    
    struct Habit: Codable, Identifiable {
        let id: UUID
        var name: String
        var emoji: String
        var completedDates: [String] // "yyyy-MM-dd" format
        var targetDays: [Int] // 1-7 (Sunday-Saturday)
        var color: String
        
        var isCompletedToday: Bool {
            let formatter = DateFormatter()
            formatter.dateFormat = "yyyy-MM-dd"
            let today = formatter.string(from: Date())
            return completedDates.contains(today)
        }
        
        var streakDays: Int {
            var streak = 0
            var date = Date()
            let calendar = Calendar.current
            let formatter = DateFormatter()
            formatter.dateFormat = "yyyy-MM-dd"
            
            while true {
                let dateString = formatter.string(from: date)
                if completedDates.contains(dateString) {
                    streak += 1
                    date = calendar.date(byAdding: .day, value: -1, to: date)!
                } else {
                    break
                }
            }
            
            return streak
        }
        
        var weeklyProgress: [Bool] {
            let calendar = Calendar.current
            let formatter = DateFormatter()
            formatter.dateFormat = "yyyy-MM-dd"
            
            return (0..<7).reversed().map { daysAgo in
                let date = calendar.date(byAdding: .day, value: -daysAgo, to: Date())!
                let dateString = formatter.string(from: date)
                return completedDates.contains(dateString)
            }
        }
    }
}

// AppIntent for Toggle Habit
struct ToggleHabitIntent: AppIntent {
    static var title: LocalizedStringResource = "สลับสถานะนิสัย"
    
    @Parameter(title: "Habit ID")
    var habitID: String
    
    init() { self.habitID = "" }
    init(habitID: String) { self.habitID = habitID }
    
    func perform() async throws -> some IntentResult {
        // Toggle habit completion
        if let data = UserDefaults.appGroup.codable(HabitData.self, forKey: "habits") {
            var updatedData = data
            let formatter = DateFormatter()
            formatter.dateFormat = "yyyy-MM-dd"
            let today = formatter.string(from: Date())
            
            if let index = updatedData.habits.firstIndex(where: { $0.id.uuidString == habitID }) {
                if updatedData.habits[index].completedDates.contains(today) {
                    updatedData.habits[index].completedDates.removeAll { $0 == today }
                } else {
                    updatedData.habits[index].completedDates.append(today)
                }
                updatedData.lastUpdated = Date()
                UserDefaults.appGroup.setCodable(updatedData, forKey: "habits")
            }
        }
        
        WidgetCenter.shared.reloadTimelines(ofKind: "HabitTracker")
        return .result()
    }
}

// Widget Configuration Intent
struct HabitListSelectionIntent: WidgetConfigurationIntent {
    static var title: LocalizedStringResource = "เลือกนิสัยที่แสดง"
    static var description = IntentDescription("เลือกจำนวนนิสัยที่ต้องการแสดงใน Widget")
    
    @Parameter(title: "จำนวนสูงสุด", default: 4, inclusiveRange: (1, 6))
    var maxHabits: Int
    
    @Parameter(title: "แสดงสตรีค", default: true)
    var showStreak: Bool
}

// Timeline Entry
struct HabitEntry: TimelineEntry {
    let date: Date
    let habits: [HabitData.Habit]
    let completedToday: Int
    let totalToday: Int
    let maxHabits: Int
    let showStreak: Bool
    
    var progressPercent: Double {
        guard totalToday > 0 else { return 0 }
        return Double(completedToday) / Double(totalToday)
    }
    
    static var sample: HabitEntry {
        let habits: [HabitData.Habit] = [
            {
                var h = HabitData.Habit(
                    id: UUID(), name: "ออกกำลังกาย", emoji: "🏃",
                    completedDates: [], targetDays: [1,2,3,4,5], color: "blue"
                )
                h.completedDates = ["2024-01-01", "2024-01-02"]
                return h
            }(),
            {
                var h = HabitData.Habit(
                    id: UUID(), name: "อ่านหนังสือ", emoji: "📚",
                    completedDates: [], targetDays: [1,2,3,4,5,6,7], color: "green"
                )
                return h
            }(),
            {
                var h = HabitData.Habit(
                    id: UUID(), name: "ทำสมาธิ", emoji: "🧘",
                    completedDates: [], targetDays: [1,2,3,4,5,6,7], color: "purple"
                )
                return h
            }()
        ]
        
        return HabitEntry(
            date: Date(), habits: habits,
            completedToday: 1, totalToday: 3,
            maxHabits: 4, showStreak: true
        )
    }
}

// Timeline Provider
struct HabitProvider: AppIntentTimelineProvider {
    typealias Entry = HabitEntry
    typealias Intent = HabitListSelectionIntent
    
    func placeholder(in context: Context) -> HabitEntry {
        HabitEntry.sample
    }
    
    func snapshot(for configuration: HabitListSelectionIntent, in context: Context) async -> HabitEntry {
        loadEntry(configuration: configuration)
    }
    
    func timeline(for configuration: HabitListSelectionIntent, in context: Context) async -> Timeline<HabitEntry> {
        let entry = loadEntry(configuration: configuration)
        
        // อัพเดทเที่ยงคืน
        var midnight = Calendar.current.startOfDay(for: Date())
        midnight = Calendar.current.date(byAdding: .day, value: 1, to: midnight)!
        
        return Timeline(entries: [entry], policy: .after(midnight))
    }
    
    private func loadEntry(configuration: HabitListSelectionIntent) -> HabitEntry {
        let data = UserDefaults.appGroup.codable(HabitData.self, forKey: "habits")
        let allHabits = data?.habits ?? HabitEntry.sample.habits
        let todayWeekday = Calendar.current.component(.weekday, from: Date())
        
        // กรองนิสัยที่ต้องทำวันนี้
        let todayHabits = allHabits.filter { $0.targetDays.contains(todayWeekday) }
        let limited = Array(todayHabits.prefix(configuration.maxHabits))
        let completed = todayHabits.filter { $0.isCompletedToday }.count
        
        return HabitEntry(
            date: Date(),
            habits: limited,
            completedToday: completed,
            totalToday: todayHabits.count,
            maxHabits: configuration.maxHabits,
            showStreak: configuration.showStreak
        )
    }
}

// MARK: - Widget Views

struct HabitTrackerSmallView: View {
    let entry: HabitEntry
    
    var body: some View {
        VStack(spacing: 8) {
            // Progress Ring
            ZStack {
                Circle()
                    .stroke(.secondary.opacity(0.2), lineWidth: 8)
                
                Circle()
                    .trim(from: 0, to: entry.progressPercent)
                    .stroke(.green, style: StrokeStyle(lineWidth: 8, lineCap: .round))
                    .rotationEffect(.degrees(-90))
                
                VStack(spacing: 2) {
                    Text("\(entry.completedToday)")
                        .font(.title2)
                        .fontWeight(.bold)
                    Text("/\(entry.totalToday)")
                        .font(.caption2)
                        .foregroundColor(.secondary)
                }
            }
            .padding(.horizontal, 20)
            
            Text("นิสัยวันนี้")
                .font(.caption)
                .foregroundColor(.secondary)
        }
        .padding()
    }
}

struct HabitTrackerMediumView: View {
    let entry: HabitEntry
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            // Header
            HStack {
                Text("นิสัยวันนี้")
                    .font(.headline)
                
                Spacer()
                
                Text("\(entry.completedToday)/\(entry.totalToday)")
                    .font(.subheadline)
                    .foregroundColor(.secondary)
            }
            
            // Habit Rows
            ForEach(entry.habits.prefix(3)) { habit in
                HabitRowView(habit: habit, showStreak: entry.showStreak)
            }
            
            if entry.habits.isEmpty {
                Text("ยินดีด้วย! ทำครบแล้ว 🎉")
                    .font(.subheadline)
                    .foregroundColor(.secondary)
            }
        }
        .padding()
    }
}

struct HabitRowView: View {
    let habit: HabitData.Habit
    let showStreak: Bool
    
    var body: some View {
        HStack(spacing: 8) {
            // Interactive Toggle Button
            Button(intent: ToggleHabitIntent(habitID: habit.id.uuidString)) {
                ZStack {
                    Circle()
                        .fill(habit.isCompletedToday ? .green : .secondary.opacity(0.2))
                        .frame(width: 24, height: 24)
                    
                    if habit.isCompletedToday {
                        Image(systemName: "checkmark")
                            .font(.caption)
                            .fontWeight(.bold)
                            .foregroundColor(.white)
                    }
                }
            }
            .buttonStyle(.plain)
            
            // Emoji + Name
            Text(habit.emoji)
                .font(.subheadline)
            
            Text(habit.name)
                .font(.subheadline)
                .strikethrough(habit.isCompletedToday, color: .secondary)
                .foregroundColor(habit.isCompletedToday ? .secondary : .primary)
            
            Spacer()
            
            // Streak
            if showStreak && habit.streakDays > 0 {
                HStack(spacing: 2) {
                    Image(systemName: "flame.fill")
                        .font(.caption)
                        .foregroundColor(.orange)
                    Text("\(habit.streakDays)")
                        .font(.caption)
                        .foregroundColor(.orange)
                }
            }
            
            // Weekly Dots
            HStack(spacing: 2) {
                ForEach(habit.weeklyProgress.indices, id: \.self) { index in
                    Circle()
                        .fill(habit.weeklyProgress[index] ? .green : .secondary.opacity(0.3))
                        .frame(width: 5, height: 5)
                }
            }
        }
    }
}

struct HabitTrackerLargeView: View {
    let entry: HabitEntry
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            // Header
            HStack {
                VStack(alignment: .leading) {
                    Text("นิสัยวันนี้")
                        .font(.title3)
                        .fontWeight(.bold)
                    
                    Text(Date(), style: .date)
                        .font(.caption)
                        .foregroundColor(.secondary)
                }
                
                Spacer()
                
                // Progress Circle
                ZStack {
                    Circle()
                        .stroke(.secondary.opacity(0.2), lineWidth: 4)
                    
                    Circle()
                        .trim(from: 0, to: entry.progressPercent)
                        .stroke(.green, style: StrokeStyle(lineWidth: 4, lineCap: .round))
                        .rotationEffect(.degrees(-90))
                    
                    Text("\(Int(entry.progressPercent * 100))%")
                        .font(.caption2)
                        .fontWeight(.bold)
                }
                .frame(width: 44, height: 44)
            }
            
            Divider()
            
            // Habit List
            ForEach(entry.habits) { habit in
                HabitRowView(habit: habit, showStreak: entry.showStreak)
            }
            
            Spacer()
            
            // Motivation Message
            if entry.completedToday == entry.totalToday && entry.totalToday > 0 {
                HStack {
                    Image(systemName: "star.fill")
                        .foregroundColor(.yellow)
                    Text("ทำครบทุกนิสัยวันนี้แล้ว!")
                        .font(.caption)
                        .foregroundColor(.secondary)
                }
            } else {
                Text("เหลืออีก \(entry.totalToday - entry.completedToday) นิสัย")
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
        }
        .padding()
    }
}

// Main Widget View
struct HabitTrackerWidgetView: View {
    let entry: HabitEntry
    @Environment(\.widgetFamily) var family
    
    var body: some View {
        switch family {
        case .systemSmall:
            HabitTrackerSmallView(entry: entry)
        case .systemMedium:
            HabitTrackerMediumView(entry: entry)
        case .systemLarge:
            HabitTrackerLargeView(entry: entry)
        default:
            HabitTrackerMediumView(entry: entry)
        }
    }
}

// Widget Definition
struct HabitTrackerWidget: Widget {
    let kind = "HabitTracker"
    
    var body: some WidgetConfiguration {
        AppIntentConfiguration(
            kind: kind,
            intent: HabitListSelectionIntent.self,
            provider: HabitProvider()
        ) { entry in
            HabitTrackerWidgetView(entry: entry)
                .containerBackground(.fill.tertiary, for: .widget)
        }
        .configurationDisplayName("ติดตามนิสัย")
        .description("ติดตามและทำครบนิสัยประจำวัน")
        .supportedFamilies([.systemSmall, .systemMedium, .systemLarge])
    }
}

// Preview
#Preview(as: .systemMedium) {
    HabitTrackerWidget()
} timeline: {
    HabitEntry.sample
}

#Preview(as: .systemLarge) {
    HabitTrackerWidget()
} timeline: {
    HabitEntry.sample
}
```

---

## 60.24 สรุป

ในบทนี้เราได้เรียนรู้ทุกอย่างเกี่ยวกับ WidgetKit:

1. **What are Widgets** - ความเข้าใจ Widget และประเภทต่างๆ
2. **WidgetKit Overview** - โครงสร้างและการตั้งค่า Widget Extension
3. **Widget Entry** - โครงสร้างข้อมูลสำหรับ Widget
4. **IntentConfiguration vs StaticConfiguration** - เลือก Configuration ที่เหมาะสม
5. **Timeline Provider** - กำหนดเวลาและข้อมูลของ Widget
6. **TimelineReloadPolicy** - ควบคุมการอัพเดทข้อมูล
7. **Widget Families** - รองรับ Widget หลายขนาด
8. **Widget Views** - สร้าง UI ที่สวยงามสำหรับแต่ละขนาด
9. **AppIntents** - Widget ที่ผู้ใช้ตั้งค่าได้
10. **Widget Links** - เปิดแอปจาก Widget
11. **Widget Previews** - Preview ใน Xcode
12. **Widget Bundle** - รวมหลาย Widget ใน Extension เดียว
13. **Interactive Widgets** - Button และ Toggle ใน Widget (iOS 17+)
14. **Live Activities** - แสดงข้อมูล real-time บน Lock Screen
15. **ActivityKit** - จัดการ Live Activities
16. **Dynamic Island** - แสดงข้อมูลบน Dynamic Island
17. **Lock Screen Widgets** - Widget บน Lock Screen
18. **App Groups** - แชร์ข้อมูลระหว่างแอปและ Widget
19. **UserDefaults with App Group** - จัดการ UserDefaults ร่วมกัน
20. **Complete Examples** - Quote Widget และ Habit Tracker Widget

---

## แบบฝึกหัดเพิ่มเติม

1. สร้าง Countdown Widget สำหรับนับถอยหลังถึงวันสำคัญ
2. สร้าง Crypto Price Widget ที่ดึงข้อมูลจาก API
3. สร้าง Photo Widget แสดงรูปจาก Photo Library
4. ปรับ Habit Tracker Widget ให้รองรับ Lock Screen
5. สร้าง Live Activity สำหรับ Sports Score Tracking
6. เพิ่ม Interactive Button ใน Task Widget สำหรับ Complete Task
7. สร้าง Calendar Widget แสดงนัดหมายวันนี้

---

*บทก่อนหน้า: Part 59 - macOS Development*
*จบหลักสูตร Swift Programming - ยินดีด้วย!*
