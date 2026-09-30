# Part 72: The Composable Architecture (TCA)

## บทนำ

The Composable Architecture (TCA) คือ library สำหรับสร้าง iOS/macOS application โดย [Point-Free](https://www.pointfree.co) ที่ให้วิธีคิดเรื่อง application architecture แบบ consistent ช่วยแก้ปัญหาที่นักพัฒนา iOS เจอบ่อยๆ เช่น:

- **State management** - จัดการ state อย่างไรให้ predictable
- **Composition** - แบ่ง feature ใหญ่เป็น feature เล็กๆ อย่างไร
- **Side effects** - จัดการ async operations, network calls อย่างไร
- **Testing** - ทดสอบ business logic อย่างไรได้ง่ายๆ
- **Ergonomics** - เขียนโค้ดให้ readable และ maintainable

---

## 1. What is TCA?

### 1.1 Core Concepts

TCA ตั้งอยู่บน 5 concepts หลัก:

```
┌─────────────────────────────────────────────┐
│                   View                       │
│  สังเกต State และส่ง Action ไปยัง Store       │
└──────────────┬──────────────────────────────┘
               │ Actions
               ▼
┌─────────────────────────────────────────────┐
│                   Store                      │
│  เก็บ State และ run Reducer                  │
└──────────────┬──────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────┐
│                  Reducer                     │
│  รับ Action + State → ส่งคืน new State       │
│  + Effects (async operations)               │
└──────────────┬──────────────────────────────┘
               │ Effects
               ▼
┌─────────────────────────────────────────────┐
│               Dependencies                   │
│  APIClient, Database, Timer, etc.            │
└─────────────────────────────────────────────┘
```

**State** - ข้อมูลทั้งหมดที่ view ต้องการแสดงผล

**Action** - ทุก event ที่เกิดขึ้นใน app (ปุ่มถูกกด, response มาถึง, timer หมดเวลา)

**Reducer** - function ที่รับ state + action แล้วคืน state ใหม่ + effects

**Effects** - side effects เช่น network calls, database access

**Store** - runtime ที่รัน reducer และจัดการ state

### 1.2 ทำไมต้องใช้ TCA?

```swift
// ปัญหาที่ TCA แก้ไข:

// 1. State กระจัดกระจายใน ViewModel หลายตัว
class ProfileViewModel: ObservableObject {
    @Published var user: User?
    @Published var posts: [Post] = []
    // state อยู่ใน ViewModel แต่ละตัว ทำให้ยาก share state
}

// 2. Testing ยาก เพราะมี side effects ใน ViewModel
class SettingsViewModel: ObservableObject {
    func saveSettings() {
        // directly call URLSession - ยากต่อการ test
        URLSession.shared.dataTask(...)
    }
}

// 3. Navigation logic ซับซ้อนและ fragmented

// TCA แก้ทุกปัญหาเหล่านี้ด้วย:
// - Single source of truth
// - Unidirectional data flow
// - Dependencies injection
// - Easy testing via TestStore
```

---

## 2. TCA Installation with SPM

### 2.1 เพิ่ม Package ด้วย Swift Package Manager

```swift
// Package.swift
// swift-tools-version: 5.9
import PackageDescription

let package = Package(
    name: "MyApp",
    platforms: [.iOS(.v17), .macOS(.v14)],
    dependencies: [
        .package(
            url: "https://github.com/pointfreeco/swift-composable-architecture",
            from: "1.0.0"
        )
    ],
    targets: [
        .target(
            name: "MyApp",
            dependencies: [
                .product(
                    name: "ComposableArchitecture",
                    package: "swift-composable-architecture"
                )
            ]
        )
    ]
)
```

หรือเพิ่มผ่าน Xcode:
1. File → Add Package Dependencies
2. ใส่ URL: `https://github.com/pointfreeco/swift-composable-architecture`
3. เลือก version: "Up to Next Major Version" from 1.0.0

### 2.2 Import

```swift
import ComposableArchitecture
```

---

## 3. Reducer Macro

### 3.1 @Reducer Macro

ใน TCA 1.0+ ใช้ `@Reducer` macro เพื่อสร้าง feature

```swift
import ComposableArchitecture

// @Reducer macro จะ:
// 1. Synthesize Equatable สำหรับ State
// 2. จัดการ CaseReducible สำหรับ Action
// 3. ทำให้ Reducer conform ถูกต้อง
@Reducer
struct CounterFeature {
    
    // State - ข้อมูลทั้งหมดของ feature นี้
    struct State: Equatable {
        var count = 0
        var isLoading = false
        var factText: String?
    }
    
    // Action - ทุก event ที่เกิดขึ้น
    enum Action {
        case incrementButtonTapped
        case decrementButtonTapped
        case factButtonTapped
        case factResponse(Result<String, Error>)
    }
    
    // Reducer - business logic
    var body: some ReducerOf<Self> {
        Reduce { state, action in
            switch action {
            case .incrementButtonTapped:
                state.count += 1
                return .none
                
            case .decrementButtonTapped:
                state.count -= 1
                return .none
                
            case .factButtonTapped:
                state.isLoading = true
                state.factText = nil
                return .run { [count = state.count] send in
                    let (data, _) = try await URLSession.shared.data(
                        from: URL(string: "http://numbersapi.com/\(count)")!
                    )
                    let fact = String(data: data, encoding: .utf8) ?? ""
                    await send(.factResponse(.success(fact)))
                } catch: { error, send in
                    await send(.factResponse(.failure(error)))
                }
                
            case .factResponse(.success(let fact)):
                state.isLoading = false
                state.factText = fact
                return .none
                
            case .factResponse(.failure):
                state.isLoading = false
                state.factText = "Failed to load fact"
                return .none
            }
        }
    }
}
```

---

## 4. State Struct

### 4.1 การออกแบบ State

```swift
@Reducer
struct TodoListFeature {
    
    // State ต้อง:
    // 1. Equatable - เพื่อ diff changes
    // 2. เก็บข้อมูลทุกอย่างที่ view ต้องการ
    // 3. ไม่มี computed property ที่ expensive
    
    struct State: Equatable {
        // ใช้ IdentifiedArray สำหรับ collections ที่ต้องการ O(1) lookup
        var todos: IdentifiedArrayOf<Todo> = []
        var filter: Filter = .all
        var isLoading = false
        var error: String?
        
        // Derived state ที่ compute จาก todos
        var filteredTodos: IdentifiedArrayOf<Todo> {
            switch filter {
            case .all:
                return todos
            case .active:
                return todos.filter { !$0.isCompleted }
            case .completed:
                return todos.filter { $0.isCompleted }
            }
        }
        
        var completedCount: Int {
            todos.filter(\.isCompleted).count
        }
        
        enum Filter: String, CaseIterable, Equatable {
            case all = "ทั้งหมด"
            case active = "ยังไม่เสร็จ"
            case completed = "เสร็จแล้ว"
        }
    }
    
    struct Todo: Equatable, Identifiable {
        let id: UUID
        var title: String
        var isCompleted: Bool
        var createdAt: Date
    }
}
```

### 4.2 State ที่มี Nested Feature

```swift
@Reducer
struct AppFeature {
    struct State: Equatable {
        var counter = CounterFeature.State()  // Nested state
        var profile = ProfileFeature.State()  // อีก feature
        var selectedTab = Tab.counter
        
        enum Tab: Equatable {
            case counter, profile
        }
    }
}
```

---

## 5. Action Enum

### 5.1 Action Naming Convention

```swift
@Reducer
struct UserProfileFeature {
    
    // Action ควร:
    // 1. ตั้งชื่อตาม "สิ่งที่เกิดขึ้น" ไม่ใช่ "สิ่งที่ต้องทำ"
    // 2. Group ตาม source (view, delegate, internal)
    
    enum Action {
        // View Actions - ผู้ใช้ทำอะไร
        case editButtonTapped
        case saveButtonTapped
        case cancelButtonTapped
        case avatarImageChanged(UIImage)
        case nameChanged(String)
        case bioChanged(String)
        
        // Internal Actions - responses จาก effects
        case profileLoaded(Result<User, Error>)
        case profileSaved(Result<User, Error>)
        case imageUploaded(Result<URL, Error>)
        
        // Delegate Actions - สิ่งที่ parent ต้องรู้
        enum Delegate {
            case profileUpdated(User)
            case cancelled
        }
        case delegate(Delegate)
    }
    
    var body: some ReducerOf<Self> {
        Reduce { state, action in
            switch action {
            case .editButtonTapped:
                state.isEditing = true
                return .none
                
            case .saveButtonTapped:
                guard state.isEditing else { return .none }
                state.isSaving = true
                return .run { [state] send in
                    let result = await Result {
                        try await apiClient.updateProfile(state.editedUser)
                    }
                    await send(.profileSaved(result))
                }
                
            case .profileSaved(.success(let user)):
                state.isSaving = false
                state.isEditing = false
                state.user = user
                return .send(.delegate(.profileUpdated(user)))
                
            case .profileSaved(.failure(let error)):
                state.isSaving = false
                state.error = error.localizedDescription
                return .none
                
            default:
                return .none
            }
        }
    }
    
    struct State: Equatable {
        var user: User?
        var editedUser: User = User()
        var isEditing = false
        var isSaving = false
        var error: String?
    }
    
    @Dependency(\.apiClient) var apiClient
}
```

---

## 6. Effect

### 6.1 Effect Types

```swift
@Reducer
struct SearchFeature {
    
    struct State: Equatable {
        var query = ""
        var results: [SearchResult] = []
        var isSearching = false
    }
    
    enum Action {
        case queryChanged(String)
        case searchResults([SearchResult])
        case cancelButtonTapped
    }
    
    // Effect.none - ไม่มี side effect
    // Effect.run - async side effect
    // Effect.send - ส่ง action อื่น
    // Effect.cancel - cancel effect ที่กำลังทำงาน
    // Effect.merge - รัน effects พร้อมกัน
    // Effect.concatenate - รัน effects ตามลำดับ
    
    enum CancelID { case search }
    
    var body: some ReducerOf<Self> {
        Reduce { state, action in
            switch action {
            case .queryChanged(let query):
                state.query = query
                
                if query.isEmpty {
                    state.results = []
                    return .cancel(id: CancelID.search)
                }
                
                state.isSearching = true
                
                return .run { send in
                    // Debounce
                    try await Task.sleep(for: .milliseconds(300))
                    
                    let results = try await searchClient.search(query)
                    await send(.searchResults(results))
                }
                .cancellable(id: CancelID.search, cancelInFlight: true)
                // cancelInFlight: true = cancel request เก่าถ้ามี request ใหม่
                
            case .searchResults(let results):
                state.isSearching = false
                state.results = results
                return .none
                
            case .cancelButtonTapped:
                state.query = ""
                state.results = []
                return .cancel(id: CancelID.search)
            }
        }
    }
    
    @Dependency(\.searchClient) var searchClient
}

// Effect.merge - รัน parallel
// return .merge(
//     .run { ... },  // effect 1
//     .run { ... }   // effect 2 (รันพร้อมกัน)
// )

// Effect.concatenate - รัน sequential  
// return .concatenate(
//     .run { ... },  // effect 1 ก่อน
//     .run { ... }   // แล้วค่อย effect 2
// )
```

---

## 7. @Dependency Property Wrapper

### 7.1 ใช้ Dependencies แทน Direct Calls

```swift
// แทนที่จะเรียก URLSession.shared ตรงๆ
// TCA ใช้ dependency injection ผ่าน @Dependency

@Reducer
struct WeatherFeature {
    
    struct State: Equatable {
        var weather: Weather?
        var isLoading = false
        var error: String?
    }
    
    enum Action {
        case fetchWeatherButtonTapped
        case weatherResponse(Result<Weather, Error>)
    }
    
    // ใช้ @Dependency แทน
    @Dependency(\.weatherClient) var weatherClient
    
    var body: some ReducerOf<Self> {
        Reduce { state, action in
            switch action {
            case .fetchWeatherButtonTapped:
                state.isLoading = true
                return .run { send in
                    await send(.weatherResponse(
                        await Result { try await weatherClient.fetchWeather("Bangkok") }
                    ))
                }
                
            case .weatherResponse(.success(let weather)):
                state.isLoading = false
                state.weather = weather
                return .none
                
            case .weatherResponse(.failure(let error)):
                state.isLoading = false
                state.error = error.localizedDescription
                return .none
            }
        }
    }
}
```

---

## 8. DependencyKey and DependencyValues

### 8.1 การสร้าง Custom Dependency

```swift
// Step 1: กำหนด protocol สำหรับ dependency
struct WeatherClient {
    var fetchWeather: @Sendable (_ city: String) async throws -> Weather
}

// Step 2: สร้าง live implementation
extension WeatherClient: DependencyKey {
    static let liveValue = WeatherClient(
        fetchWeather: { city in
            let url = URL(string: "https://api.weather.com/current?city=\(city)")!
            let (data, _) = try await URLSession.shared.data(from: url)
            return try JSONDecoder().decode(Weather.self, from: data)
        }
    )
    
    // Test implementation
    static let testValue = WeatherClient(
        fetchWeather: { _ in
            Weather(temperature: 30, description: "Sunny", city: "Bangkok")
        }
    )
    
    // Preview implementation
    static let previewValue = testValue
}

// Step 3: Register ใน DependencyValues
extension DependencyValues {
    var weatherClient: WeatherClient {
        get { self[WeatherClient.self] }
        set { self[WeatherClient.self] = newValue }
    }
}

// Step 4: ใช้ใน Reducer
@Reducer
struct MyFeature {
    @Dependency(\.weatherClient) var weatherClient
    // ใช้ weatherClient.fetchWeather("Bangkok") ได้เลย
}

// Step 5: Override ใน Tests
func testFetchWeather() async {
    let store = TestStore(initialState: WeatherFeature.State()) {
        WeatherFeature()
    } withDependencies: {
        $0.weatherClient.fetchWeather = { _ in
            Weather(temperature: 25, description: "Cloudy", city: "Bangkok")
        }
    }
    
    await store.send(.fetchWeatherButtonTapped) {
        $0.isLoading = true
    }
    
    await store.receive(.weatherResponse(.success(
        Weather(temperature: 25, description: "Cloudy", city: "Bangkok")
    ))) {
        $0.isLoading = false
        $0.weather = Weather(temperature: 25, description: "Cloudy", city: "Bangkok")
    }
}
```

### 8.2 Built-in Dependencies

```swift
// TCA มี built-in dependencies มากมาย

@Reducer
struct TimerFeature {
    
    @Dependency(\.continuousClock) var clock     // สำหรับ timing
    @Dependency(\.uuid) var uuid                  // UUID generation
    @Dependency(\.date) var date                  // Current date
    @Dependency(\.mainQueue) var mainQueue        // Main thread
    
    struct State: Equatable {
        var seconds = 0
        var isRunning = false
    }
    
    enum Action {
        case toggleTimer
        case timerTick
    }
    
    enum CancelID { case timer }
    
    var body: some ReducerOf<Self> {
        Reduce { state, action in
            switch action {
            case .toggleTimer:
                state.isRunning.toggle()
                
                if state.isRunning {
                    return .run { send in
                        for await _ in self.clock.timer(interval: .seconds(1)) {
                            await send(.timerTick)
                        }
                    }
                    .cancellable(id: CancelID.timer)
                } else {
                    return .cancel(id: CancelID.timer)
                }
                
            case .timerTick:
                state.seconds += 1
                return .none
            }
        }
    }
}
```

---

## 9. Store Initialization

### 9.1 สร้างและใช้ Store

```swift
// Store เป็น runtime ที่รัน reducer
// ทุก Store ต้องมี initial state

// App Entry Point
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView(
                store: Store(initialState: AppFeature.State()) {
                    AppFeature()
                        ._printChanges()  // Debug: print state changes
                }
            )
        }
    }
}

// View ที่ใช้ Store
struct ContentView: View {
    let store: StoreOf<AppFeature>
    
    var body: some View {
        WithPerceptionTracking {
            VStack {
                Text("Count: \(store.counter.count)")
                Button("Increment") {
                    store.send(.counter(.incrementButtonTapped))
                }
            }
        }
    }
}
```

---

## 10. StoreOf Shorthand

### 10.1 StoreOf type alias

```swift
// StoreOf<Feature> เป็น type alias ของ Store<Feature.State, Feature.Action>
// ใช้ให้กระชับขึ้น

// แทนที่:
// let store: Store<CounterFeature.State, CounterFeature.Action>

// ใช้:
// let store: StoreOf<CounterFeature>

struct CounterView: View {
    let store: StoreOf<CounterFeature>
    
    var body: some View {
        WithPerceptionTracking {
            VStack(spacing: 20) {
                Text("\(store.count)")
                    .font(.largeTitle)
                
                HStack {
                    Button("-") {
                        store.send(.decrementButtonTapped)
                    }
                    Button("+") {
                        store.send(.incrementButtonTapped)
                    }
                }
                
                if store.isLoading {
                    ProgressView()
                } else if let fact = store.factText {
                    Text(fact)
                        .multilineTextAlignment(.center)
                        .padding()
                }
                
                Button("Get Fact") {
                    store.send(.factButtonTapped)
                }
            }
            .padding()
        }
    }
}

// Preview
#Preview {
    CounterView(
        store: Store(initialState: CounterFeature.State()) {
            CounterFeature()
        }
    )
}
```

---

## 11. ViewStore (Older API)

### 11.1 ViewStore ใน TCA เก่า (< 1.7)

```swift
// ViewStore ใช้สำหรับ observe state changes และ send actions
// ปัจจุบัน deprecated แล้วใน favor ของ @Perception

// Old API (ยังใช้ได้ แต่ไม่แนะนำสำหรับโปรเจค ใหม่)
struct OldCounterView: View {
    let store: StoreOf<CounterFeature>
    
    var body: some View {
        WithViewStore(store, observe: { $0 }) { viewStore in
            VStack {
                Text("\(viewStore.count)")
                Button("+") { viewStore.send(.incrementButtonTapped) }
                Button("-") { viewStore.send(.decrementButtonTapped) }
            }
        }
    }
}

// observe: { $0 } - observe state ทั้งหมด
// ถ้าต้องการ observe เฉพาะบางส่วน เพื่อลด re-renders:
struct OldOptimizedView: View {
    let store: StoreOf<CounterFeature>
    
    var body: some View {
        // observe แค่ count ไม่ re-render เมื่อ isLoading เปลี่ยน
        WithViewStore(store, observe: \.count) { viewStore in
            Text("\(viewStore.state)")  // viewStore.state ตรงๆ เมื่อ observe เฉพาะ field
        }
    }
}
```

---

## 12. WithViewStore (Older API)

### 12.1 Pattern การใช้ WithViewStore

```swift
// WithViewStore เป็น ViewBuilder ที่ observe state
// ปัจจุบัน prefer ใช้ @Perception แทน

// วิธีใช้กับ NavigationStack
struct OldAppView: View {
    let store: StoreOf<AppFeature>
    
    var body: some View {
        WithViewStore(store, observe: { $0 }) { viewStore in
            TabView(selection: viewStore.binding(
                get: \.selectedTab,
                send: AppFeature.Action.tabSelected
            )) {
                CounterView(store: store.scope(
                    state: \.counter,
                    action: \.counter
                ))
                .tabItem { Label("Counter", systemImage: "number") }
                .tag(AppFeature.State.Tab.counter)
                
                ProfileView(store: store.scope(
                    state: \.profile,
                    action: \.profile
                ))
                .tabItem { Label("Profile", systemImage: "person") }
                .tag(AppFeature.State.Tab.profile)
            }
        }
    }
}
```

---

## 13. @BindingState และ BindingViewStore

### 13.1 Two-way Binding ใน TCA

```swift
// ปัญหา: SwiftUI ต้องการ Binding แต่ TCA ใช้ unidirectional data flow
// แก้โดยใช้ @BindingState

@Reducer
struct FormFeature {
    
    struct State: Equatable {
        @BindingState var name: String = ""     // สามารถ bind กับ TextField ได้
        @BindingState var email: String = ""
        @BindingState var age: Int = 0
        @BindingState var isSubscribed: Bool = false
        var isSubmitting = false
        var error: String?
    }
    
    enum Action: BindableAction {    // ต้อง conform BindableAction
        case binding(BindingAction<State>)  // จัดการ @BindingState ทั้งหมด
        case submitButtonTapped
        case submitResponse(Result<Void, Error>)
    }
    
    var body: some ReducerOf<Self> {
        BindingReducer()  // ต้องใส่ BindingReducer ด้วย
        
        Reduce { state, action in
            switch action {
            case .binding:
                return .none  // BindingReducer จัดการแล้ว
                
            case .submitButtonTapped:
                state.isSubmitting = true
                return .run { [state] send in
                    let result = await Result {
                        try await apiClient.submit(name: state.name, email: state.email)
                    }
                    await send(.submitResponse(result))
                }
                
            case .submitResponse(.success):
                state.isSubmitting = false
                return .none
                
            case .submitResponse(.failure(let error)):
                state.isSubmitting = false
                state.error = error.localizedDescription
                return .none
            }
        }
    }
    
    @Dependency(\.apiClient) var apiClient
}

// View ใช้ $store.fieldName สำหรับ binding
struct FormView: View {
    @Bindable var store: StoreOf<FormFeature>
    
    var body: some View {
        Form {
            TextField("ชื่อ", text: $store.name)
            TextField("อีเมล", text: $store.email)
            Stepper("อายุ: \(store.age)", value: $store.age, in: 0...120)
            Toggle("รับข่าวสาร", isOn: $store.isSubscribed)
            
            Button("ส่ง") {
                store.send(.submitButtonTapped)
            }
            .disabled(store.isSubmitting)
        }
    }
}
```

---

## 14. ObservableState (Newer Macro-based API)

### 14.1 @ObservableState Macro

ใน TCA 1.7+ มี `@ObservableState` macro ที่ทำให้ swiftUI observation ทำงานได้โดยตรง

```swift
import ComposableArchitecture
import Perception

@Reducer
struct ModernCounterFeature {
    
    @ObservableState  // Macro ใหม่ใน TCA 1.7+
    struct State: Equatable {
        var count = 0
        var isLoading = false
        var factText: String?
    }
    
    enum Action {
        case incrementButtonTapped
        case decrementButtonTapped
        case factButtonTapped
        case factResponse(Result<String, Error>)
    }
    
    var body: some ReducerOf<Self> {
        Reduce { state, action in
            switch action {
            case .incrementButtonTapped:
                state.count += 1
                return .none
            case .decrementButtonTapped:
                state.count -= 1
                return .none
            case .factButtonTapped:
                state.isLoading = true
                return .run { [count = state.count] send in
                    try await Task.sleep(for: .seconds(1))
                    await send(.factResponse(.success("\(count) is a great number!")))
                }
            case .factResponse(.success(let fact)):
                state.isLoading = false
                state.factText = fact
                return .none
            case .factResponse(.failure):
                state.isLoading = false
                return .none
            }
        }
    }
}

// View ที่ใช้ @ObservableState
struct ModernCounterView: View {
    let store: StoreOf<ModernCounterFeature>
    
    var body: some View {
        // ไม่ต้องใช้ WithViewStore หรือ WithPerceptionTracking อีกแล้ว!
        // SwiftUI observation ทำงานได้ตรงๆ
        VStack {
            Text("\(store.count)")
                .font(.largeTitle)
            
            HStack {
                Button("-") { store.send(.decrementButtonTapped) }
                Button("+") { store.send(.incrementButtonTapped) }
            }
            
            if store.isLoading {
                ProgressView()
            }
            
            if let fact = store.factText {
                Text(fact).padding()
            }
            
            Button("Get Fact") {
                store.send(.factButtonTapped)
            }
        }
    }
}
```

---

## 15. Composition with Child Features

### 15.1 Parent-Child Relationship

```swift
// Child Feature
@Reducer
struct ChildCounterFeature {
    
    @ObservableState
    struct State: Equatable {
        var count = 0
        var label: String
    }
    
    enum Action {
        case incrementTapped
        case decrementTapped
    }
    
    var body: some ReducerOf<Self> {
        Reduce { state, action in
            switch action {
            case .incrementTapped:
                state.count += 1
                return .none
            case .decrementTapped:
                state.count -= 1
                return .none
            }
        }
    }
}

// Parent Feature ที่มี child features
@Reducer
struct ParentFeature {
    
    @ObservableState
    struct State: Equatable {
        var counter1 = ChildCounterFeature.State(count: 0, label: "Counter A")
        var counter2 = ChildCounterFeature.State(count: 0, label: "Counter B")
        var total: Int { counter1.count + counter2.count }
    }
    
    enum Action {
        case counter1(ChildCounterFeature.Action)
        case counter2(ChildCounterFeature.Action)
    }
    
    var body: some ReducerOf<Self> {
        // ใช้ Scope เพื่อ forward actions ไปยัง child reducers
        Scope(state: \.counter1, action: \.counter1) {
            ChildCounterFeature()
        }
        
        Scope(state: \.counter2, action: \.counter2) {
            ChildCounterFeature()
        }
        
        // Parent logic เพิ่มเติม (ถ้ามี)
        Reduce { state, action in
            return .none
        }
    }
}

// Parent View
struct ParentView: View {
    let store: StoreOf<ParentFeature>
    
    var body: some View {
        VStack {
            Text("Total: \(store.total)")
                .font(.headline)
            
            // ส่ง scoped store ไปยัง child view
            ChildCounterView(
                store: store.scope(
                    state: \.counter1,
                    action: \.counter1
                )
            )
            
            ChildCounterView(
                store: store.scope(
                    state: \.counter2,
                    action: \.counter2
                )
            )
        }
    }
}

// Child View
struct ChildCounterView: View {
    let store: StoreOf<ChildCounterFeature>
    
    var body: some View {
        HStack {
            Text(store.label)
            Spacer()
            Button("-") { store.send(.decrementTapped) }
            Text("\(store.count)")
            Button("+") { store.send(.incrementTapped) }
        }
        .padding()
    }
}
```

---

## 16. Scope

### 16.1 Scope ใน Reducer

```swift
// Scope ใช้สำหรับ embed child reducer ใน parent
// syntax: Scope(state: KeyPath, action: CaseKeyPath) { ChildReducer() }

@Reducer
struct AppFeature {
    
    @ObservableState
    struct State: Equatable {
        var home = HomeFeature.State()
        var settings = SettingsFeature.State()
        var selectedTab = Tab.home
        
        enum Tab { case home, settings }
    }
    
    enum Action {
        case home(HomeFeature.Action)
        case settings(SettingsFeature.Action)
        case tabSelected(State.Tab)
    }
    
    var body: some ReducerOf<Self> {
        Scope(state: \.home, action: \.home) {
            HomeFeature()
        }
        
        Scope(state: \.settings, action: \.settings) {
            SettingsFeature()
        }
        
        Reduce { state, action in
            switch action {
            case .tabSelected(let tab):
                state.selectedTab = tab
                return .none
                
            case .home(.logoutButtonTapped):
                // Parent intercepts child action
                // Do something here
                return .none
                
            default:
                return .none
            }
        }
    }
}
```

---

## 17. IfLetReducer

### 17.1 Optional State

```swift
// IfLetReducer ใช้สำหรับ optional child state
// เช่น modal, sheet, detail view

@Reducer
struct MasterFeature {
    
    @ObservableState
    struct State: Equatable {
        var items: [Item] = []
        var selectedItem: DetailFeature.State? = nil  // Optional
    }
    
    enum Action {
        case itemTapped(Item)
        case detail(DetailFeature.Action)
        case closeDetail
    }
    
    var body: some ReducerOf<Self> {
        Reduce { state, action in
            switch action {
            case .itemTapped(let item):
                state.selectedItem = DetailFeature.State(item: item)
                return .none
                
            case .closeDetail:
                state.selectedItem = nil
                return .none
                
            case .detail:
                return .none
            }
        }
        .ifLet(\.selectedItem, action: \.detail) {
            DetailFeature()
        }
        // .ifLet จะ:
        // - รัน DetailFeature เมื่อ selectedItem != nil
        // - Cancel effects ของ child เมื่อ selectedItem เป็น nil
    }
}

// View
struct MasterView: View {
    @Bindable var store: StoreOf<MasterFeature>
    
    var body: some View {
        NavigationStack {
            List(store.items) { item in
                Button(item.name) {
                    store.send(.itemTapped(item))
                }
            }
            .sheet(item: $store.scope(state: \.selectedItem, action: \.detail)) { detailStore in
                DetailView(store: detailStore)
            }
        }
    }
}
```

---

## 18. ForEachReducer

### 18.1 Collection ของ Child Features

```swift
// ForEachReducer ใช้สำหรับ IdentifiedArray ของ child states

@Reducer
struct TodoListFeature {
    
    @ObservableState
    struct State: Equatable {
        var todos: IdentifiedArrayOf<TodoFeature.State> = []
    }
    
    enum Action {
        case addTodoButtonTapped
        case todos(IdentifiedActionOf<TodoFeature>)  // Actions สำหรับแต่ละ todo
        case deleteButtonTapped(IndexSet)
    }
    
    var body: some ReducerOf<Self> {
        Reduce { state, action in
            switch action {
            case .addTodoButtonTapped:
                let id = UUID()
                state.todos.insert(
                    TodoFeature.State(id: id, title: "New Todo"),
                    at: 0
                )
                return .none
                
            case .deleteButtonTapped(let indexSet):
                state.todos.remove(atOffsets: indexSet)
                return .none
                
            case .todos:
                return .none
            }
        }
        .forEach(\.todos, action: \.todos) {
            TodoFeature()
        }
    }
}

// Child Feature
@Reducer
struct TodoFeature {
    
    @ObservableState
    struct State: Equatable, Identifiable {
        let id: UUID
        var title: String
        var isCompleted: Bool = false
        var isEditing: Bool = false
    }
    
    enum Action {
        case toggleCompleted
        case editButtonTapped
        case titleChanged(String)
        case saveButtonTapped
    }
    
    var body: some ReducerOf<Self> {
        Reduce { state, action in
            switch action {
            case .toggleCompleted:
                state.isCompleted.toggle()
                return .none
                
            case .editButtonTapped:
                state.isEditing = true
                return .none
                
            case .titleChanged(let title):
                state.title = title
                return .none
                
            case .saveButtonTapped:
                state.isEditing = false
                return .none
            }
        }
    }
}

// View
struct TodoListView: View {
    @Bindable var store: StoreOf<TodoListFeature>
    
    var body: some View {
        List {
            ForEach(store.scope(state: \.todos, action: \.todos)) { todoStore in
                TodoRow(store: todoStore)
            }
            .onDelete { indexSet in
                store.send(.deleteButtonTapped(indexSet))
            }
        }
        .toolbar {
            Button("Add") {
                store.send(.addTodoButtonTapped)
            }
        }
    }
}

struct TodoRow: View {
    let store: StoreOf<TodoFeature>
    
    var body: some View {
        HStack {
            Button {
                store.send(.toggleCompleted)
            } label: {
                Image(systemName: store.isCompleted ? "checkmark.circle.fill" : "circle")
            }
            
            if store.isEditing {
                TextField("Title", text: Binding(
                    get: { store.title },
                    set: { store.send(.titleChanged($0)) }
                ))
            } else {
                Text(store.title)
                    .strikethrough(store.isCompleted)
            }
            
            Spacer()
            
            Button(store.isEditing ? "Save" : "Edit") {
                if store.isEditing {
                    store.send(.saveButtonTapped)
                } else {
                    store.send(.editButtonTapped)
                }
            }
        }
    }
}
```

---

## 19. Combines Reducers

### 19.1 CombineReducers

```swift
// วิธี combine หลาย reducers เข้าด้วยกัน

@Reducer
struct AppReducer {
    var body: some ReducerOf<AppFeature> {
        // วิธีที่ 1: ใช้ body property
        Scope(state: \.tab1, action: \.tab1) { Tab1Feature() }
        Scope(state: \.tab2, action: \.tab2) { Tab2Feature() }
        Reduce { state, action in .none }  // Additional logic
    }
}

// วิธีที่ 2: _PresentationReducer ใน-built
// วิธีที่ 3: custom ReducerBuilder
struct MyCustomReducer: Reducer {
    let reducers: [any Reducer<AppFeature.State, AppFeature.Action>]
    
    var body: some ReducerOf<AppFeature> {
        for reducer in reducers {
            AnyReducer(reducer)
        }
    }
}

// Combining แบบ Protocol-oriented
@Reducer
struct LoggingReducer<R: Reducer>: Reducer {
    let base: R
    
    var body: some ReducerOf<R> {
        Reduce { state, action in
            print("Action: \(action)")
            let effect = base.reduce(into: &state, action: action)
            print("New State: \(state)")
            return effect
        }
    }
}

// ใช้งาน
let myFeature = LoggingReducer(base: CounterFeature())
```

---

## 20. Navigation with PresentationState/Action

### 20.1 Sheet Navigation

```swift
// TCA มี built-in support สำหรับ navigation ผ่าน PresentationState

@Reducer
struct ParentNavigationFeature {
    
    @ObservableState
    struct State: Equatable {
        @Presents var sheet: SheetFeature.State?          // Sheet
        @Presents var fullScreenCover: FullScreenFeature.State? // Full Screen
        @Presents var alert: AlertState<Action.Alert>?   // Alert
    }
    
    enum Action {
        case showSheetButtonTapped
        case showAlertButtonTapped
        case sheet(PresentationAction<SheetFeature.Action>)
        case fullScreenCover(PresentationAction<FullScreenFeature.Action>)
        case alert(PresentationAction<Alert>)
        
        enum Alert {
            case confirmDelete
            case cancel
        }
    }
    
    var body: some ReducerOf<Self> {
        Reduce { state, action in
            switch action {
            case .showSheetButtonTapped:
                state.sheet = SheetFeature.State()
                return .none
                
            case .showAlertButtonTapped:
                state.alert = AlertState {
                    TextState("ลบข้อมูล?")
                } actions: {
                    ButtonState(role: .destructive, action: .confirmDelete) {
                        TextState("ลบ")
                    }
                    ButtonState(role: .cancel, action: .cancel) {
                        TextState("ยกเลิก")
                    }
                } message: {
                    TextState("ข้อมูลจะถูกลบถาวร")
                }
                return .none
                
            case .sheet(.dismiss), .sheet(.presented(.closeButtonTapped)):
                state.sheet = nil
                return .none
                
            case .alert(.presented(.confirmDelete)):
                // Handle delete
                return .none
                
            case .sheet, .fullScreenCover, .alert:
                return .none
            }
        }
        .ifLet(\.$sheet, action: \.sheet) {
            SheetFeature()
        }
        .ifLet(\.$fullScreenCover, action: \.fullScreenCover) {
            FullScreenFeature()
        }
    }
}

// View
struct ParentNavigationView: View {
    @Bindable var store: StoreOf<ParentNavigationFeature>
    
    var body: some View {
        VStack {
            Button("แสดง Sheet") {
                store.send(.showSheetButtonTapped)
            }
            Button("แสดง Alert") {
                store.send(.showAlertButtonTapped)
            }
        }
        .sheet(
            item: $store.scope(state: \.sheet, action: \.sheet)
        ) { sheetStore in
            SheetView(store: sheetStore)
        }
        .alert(
            store: store.scope(state: \.alert, action: \.alert)
        )
    }
}
```

### 20.2 NavigationStack Integration

```swift
// Navigation Stack ใน TCA
@Reducer
struct NavigationFeature {
    
    @Reducer
    enum Path {
        case detail(DetailFeature)
        case settings(SettingsFeature)
        case profile(ProfileFeature)
    }
    
    @ObservableState
    struct State: Equatable {
        var path = StackState<Path.State>()
        var items: [Item] = []
    }
    
    enum Action {
        case path(StackActionOf<Path>)
        case itemTapped(Item)
    }
    
    var body: some ReducerOf<Self> {
        Reduce { state, action in
            switch action {
            case .itemTapped(let item):
                state.path.append(.detail(DetailFeature.State(item: item)))
                return .none
                
            case .path(.element(id: _, action: .detail(.settingsButtonTapped))):
                state.path.append(.settings(SettingsFeature.State()))
                return .none
                
            case .path:
                return .none
            }
        }
        .forEach(\.path, action: \.path)
    }
}

// View
struct NavigationView: View {
    @Bindable var store: StoreOf<NavigationFeature>
    
    var body: some View {
        NavigationStack(path: $store.scope(state: \.path, action: \.path)) {
            List(store.items) { item in
                Button(item.name) {
                    store.send(.itemTapped(item))
                }
            }
            .navigationTitle("Items")
        } destination: { store in
            switch store.case {
            case .detail(let detailStore):
                DetailView(store: detailStore)
            case .settings(let settingsStore):
                SettingsView(store: settingsStore)
            case .profile(let profileStore):
                ProfileView(store: profileStore)
            }
        }
    }
}
```

---

## 21. TestStore

### 21.1 การใช้ TestStore

```swift
import ComposableArchitecture
import XCTest

class CounterFeatureTests: XCTestCase {
    
    @MainActor
    func testIncrement() async {
        // สร้าง TestStore
        let store = TestStore(initialState: CounterFeature.State()) {
            CounterFeature()
        }
        
        // ส่ง action
        await store.send(.incrementButtonTapped) { state in
            // assert state หลังจาก action
            state.count = 1  // บอกว่า count ควรเป็น 1
        }
        
        await store.send(.incrementButtonTapped) {
            $0.count = 2
        }
        
        await store.send(.decrementButtonTapped) {
            $0.count = 1
        }
    }
    
    @MainActor
    func testDecrementBelowZero() async {
        let store = TestStore(initialState: CounterFeature.State()) {
            CounterFeature()
        }
        
        // ถ้า reducer ไม่อนุญาตให้ลบต่ำกว่า 0
        await store.send(.decrementButtonTapped)  // ไม่มี state change
        // ถ้า state ไม่เปลี่ยน ไม่ต้องใส่ closure
    }
    
    @MainActor
    func testFetchFact() async {
        let store = TestStore(initialState: CounterFeature.State(count: 7)) {
            CounterFeature()
        } withDependencies: {
            $0.numberFact.fetch = { number in
                "\(number) is lucky!"
            }
        }
        
        await store.send(.factButtonTapped) {
            $0.isLoading = true
        }
        
        await store.receive(.factResponse(.success("7 is lucky!"))) {
            $0.isLoading = false
            $0.factText = "7 is lucky!"
        }
    }
}
```

---

## 22. Testing Actions and Effects

### 22.1 Testing Async Effects

```swift
class WeatherFeatureTests: XCTestCase {
    
    @MainActor
    func testFetchWeather() async {
        let mockWeather = Weather(
            temperature: 30,
            description: "Sunny",
            city: "Bangkok"
        )
        
        let store = TestStore(initialState: WeatherFeature.State()) {
            WeatherFeature()
        } withDependencies: {
            // Override dependency สำหรับ test
            $0.weatherClient.fetchWeather = { city in
                XCTAssertEqual(city, "Bangkok")
                return mockWeather
            }
        }
        
        await store.send(.fetchButtonTapped) {
            $0.isLoading = true
        }
        
        await store.receive(.weatherResponse(.success(mockWeather))) {
            $0.isLoading = false
            $0.weather = mockWeather
        }
    }
    
    @MainActor
    func testFetchWeatherFailure() async {
        struct MockError: Error, Equatable {}
        
        let store = TestStore(initialState: WeatherFeature.State()) {
            WeatherFeature()
        } withDependencies: {
            $0.weatherClient.fetchWeather = { _ in
                throw MockError()
            }
        }
        
        await store.send(.fetchButtonTapped) {
            $0.isLoading = true
        }
        
        await store.receive(.weatherResponse(.failure(MockError()))) {
            $0.isLoading = false
            $0.errorMessage = "Failed to load weather"
        }
    }
    
    @MainActor
    func testCancelEffect() async {
        let store = TestStore(initialState: SearchFeature.State()) {
            SearchFeature()
        }
        
        // เริ่ม search
        await store.send(.queryChanged("swift")) {
            $0.query = "swift"
            $0.isSearching = true
        }
        
        // Cancel ก่อน effect เสร็จ
        await store.send(.cancelButtonTapped) {
            $0.query = ""
            $0.isSearching = false
        }
        
        // ไม่ควรมี receive หลังจากนี้
    }
}
```

---

## 23. Async Testing in TCA

### 23.1 Testing Timer และ Clock

```swift
class TimerFeatureTests: XCTestCase {
    
    @MainActor
    func testTimer() async {
        // ใช้ TestClock เพื่อควบคุม time
        let clock = TestClock()
        
        let store = TestStore(initialState: TimerFeature.State()) {
            TimerFeature()
        } withDependencies: {
            $0.continuousClock = clock  // inject TestClock
        }
        
        // Start timer
        await store.send(.toggleTimer) {
            $0.isRunning = true
        }
        
        // Advance clock 1 second
        await clock.advance(by: .seconds(1))
        await store.receive(.timerTick) {
            $0.seconds = 1
        }
        
        // Advance clock 3 more seconds
        await clock.advance(by: .seconds(3))
        await store.receive(.timerTick) { $0.seconds = 2 }
        await store.receive(.timerTick) { $0.seconds = 3 }
        await store.receive(.timerTick) { $0.seconds = 4 }
        
        // Stop timer
        await store.send(.toggleTimer) {
            $0.isRunning = false
        }
        
        // ไม่ควรมี tick หลังจากหยุด
        await clock.advance(by: .seconds(5))
    }
}

// Testing ด้วย exhaustivity
class FormFeatureTests: XCTestCase {
    
    @MainActor
    func testNonExhaustiveTest() async {
        let store = TestStore(initialState: FormFeature.State()) {
            FormFeature()
        }
        
        // ใช้ store.exhaustivity = .off สำหรับ test ที่ไม่ต้องการ assert ทุก action
        store.exhaustivity = .off
        
        await store.send(.nameChanged("John"))
        // ไม่ต้องใส่ state assertion
        
        await store.send(.emailChanged("john@example.com"))
        // ข้าม actions ที่ไม่ต้องการ test
        
        // ตรวจสอบเฉพาะส่วนที่ต้องการ
        XCTAssertEqual(store.state.name, "John")
        XCTAssertEqual(store.state.email, "john@example.com")
    }
}
```

---

## 24. Practical Exercise: Todo App with TCA

### 24.1 Complete Todo App

```swift
import ComposableArchitecture
import SwiftUI

// MARK: - Models

struct Todo: Equatable, Identifiable, Codable {
    var id: UUID
    var title: String
    var isCompleted: Bool = false
    var priority: Priority = .medium
    var createdAt: Date = Date()
    
    enum Priority: String, CaseIterable, Codable {
        case low = "ต่ำ"
        case medium = "ปานกลาง"
        case high = "สูง"
        
        var color: Color {
            switch self {
            case .low: return .green
            case .medium: return .orange
            case .high: return .red
            }
        }
    }
}

// MARK: - Dependencies

struct TodoStorageClient {
    var load: @Sendable () async throws -> [Todo]
    var save: @Sendable ([Todo]) async throws -> Void
}

extension TodoStorageClient: DependencyKey {
    static let liveValue = TodoStorageClient(
        load: {
            guard let data = UserDefaults.standard.data(forKey: "todos") else { return [] }
            return try JSONDecoder().decode([Todo].self, from: data)
        },
        save: { todos in
            let data = try JSONEncoder().encode(todos)
            UserDefaults.standard.set(data, forKey: "todos")
        }
    )
    
    static let testValue = TodoStorageClient(
        load: { [] },
        save: { _ in }
    )
}

extension DependencyValues {
    var todoStorage: TodoStorageClient {
        get { self[TodoStorageClient.self] }
        set { self[TodoStorageClient.self] = newValue }
    }
}

// MARK: - Todo Row Feature

@Reducer
struct TodoRowFeature {
    
    @ObservableState
    struct State: Equatable, Identifiable {
        var id: UUID { todo.id }
        var todo: Todo
        var isEditing = false
        var editTitle = ""
    }
    
    enum Action {
        case toggleCompletedTapped
        case editButtonTapped
        case cancelEditTapped
        case saveEditTapped
        case editTitleChanged(String)
        case deleteTapped
        
        // Delegate - events ที่ parent ต้องรู้
        enum Delegate: Equatable {
            case delete(Todo)
            case updated(Todo)
        }
        case delegate(Delegate)
    }
    
    var body: some ReducerOf<Self> {
        Reduce { state, action in
            switch action {
            case .toggleCompletedTapped:
                state.todo.isCompleted.toggle()
                return .send(.delegate(.updated(state.todo)))
                
            case .editButtonTapped:
                state.isEditing = true
                state.editTitle = state.todo.title
                return .none
                
            case .cancelEditTapped:
                state.isEditing = false
                state.editTitle = ""
                return .none
                
            case .saveEditTapped:
                guard !state.editTitle.isEmpty else { return .none }
                state.todo.title = state.editTitle
                state.isEditing = false
                return .send(.delegate(.updated(state.todo)))
                
            case .editTitleChanged(let title):
                state.editTitle = title
                return .none
                
            case .deleteTapped:
                return .send(.delegate(.delete(state.todo)))
                
            case .delegate:
                return .none
            }
        }
    }
}

// MARK: - Todo List Feature

@Reducer
struct TodoListFeature {
    
    @ObservableState
    struct State: Equatable {
        var todos: IdentifiedArrayOf<TodoRowFeature.State> = []
        var newTodoTitle = ""
        var filter: Filter = .all
        var isLoading = false
        
        enum Filter: String, CaseIterable {
            case all = "ทั้งหมด"
            case active = "ยังไม่เสร็จ"
            case completed = "เสร็จแล้ว"
        }
        
        var filteredTodos: IdentifiedArrayOf<TodoRowFeature.State> {
            switch filter {
            case .all:
                return todos
            case .active:
                return todos.filter { !$0.todo.isCompleted }
            case .completed:
                return todos.filter { $0.todo.isCompleted }
            }
        }
        
        var stats: Stats {
            Stats(
                total: todos.count,
                completed: todos.filter(\.todo.isCompleted).count
            )
        }
        
        struct Stats: Equatable {
            let total: Int
            let completed: Int
            var pending: Int { total - completed }
        }
    }
    
    enum Action {
        case onAppear
        case todosLoaded(Result<[Todo], Error>)
        case newTodoTitleChanged(String)
        case addTodoButtonTapped
        case filterChanged(State.Filter)
        case clearCompletedButtonTapped
        case todos(IdentifiedActionOf<TodoRowFeature>)
    }
    
    @Dependency(\.todoStorage) var storage
    @Dependency(\.uuid) var uuid
    @Dependency(\.date) var date
    
    var body: some ReducerOf<Self> {
        Reduce { state, action in
            switch action {
            case .onAppear:
                state.isLoading = true
                return .run { send in
                    await send(.todosLoaded(
                        await Result { try await storage.load() }
                    ))
                }
                
            case .todosLoaded(.success(let todos)):
                state.isLoading = false
                state.todos = IdentifiedArray(
                    uniqueElements: todos.map { TodoRowFeature.State(todo: $0) }
                )
                return .none
                
            case .todosLoaded(.failure):
                state.isLoading = false
                return .none
                
            case .newTodoTitleChanged(let title):
                state.newTodoTitle = title
                return .none
                
            case .addTodoButtonTapped:
                let title = state.newTodoTitle.trimmingCharacters(in: .whitespaces)
                guard !title.isEmpty else { return .none }
                
                let todo = Todo(id: uuid(), title: title, createdAt: date.now)
                state.todos.insert(TodoRowFeature.State(todo: todo), at: 0)
                state.newTodoTitle = ""
                
                return saveEffect(state: state)
                
            case .filterChanged(let filter):
                state.filter = filter
                return .none
                
            case .clearCompletedButtonTapped:
                state.todos.removeAll { $0.todo.isCompleted }
                return saveEffect(state: state)
                
            case .todos(.element(id: _, action: .delegate(.delete(let todo)))):
                state.todos.remove(id: TodoRowFeature.State.ID(todo.id))
                return saveEffect(state: state)
                
            case .todos(.element(id: let id, action: .delegate(.updated(let todo)))):
                state.todos[id: id]?.todo = todo
                return saveEffect(state: state)
                
            case .todos:
                return .none
            }
        }
        .forEach(\.todos, action: \.todos) {
            TodoRowFeature()
        }
    }
    
    private func saveEffect(state: State) -> Effect<Action> {
        let todos = state.todos.map(\.todo)
        return .run { _ in
            try? await storage.save(todos)
        }
    }
}

// MARK: - Views

struct TodoListView: View {
    @Bindable var store: StoreOf<TodoListFeature>
    
    var body: some View {
        NavigationStack {
            VStack(spacing: 0) {
                // Stats bar
                statsBar
                
                // Filter picker
                Picker("Filter", selection: $store.filter.sending(\.filterChanged)) {
                    ForEach(TodoListFeature.State.Filter.allCases, id: \.self) { filter in
                        Text(filter.rawValue).tag(filter)
                    }
                }
                .pickerStyle(.segmented)
                .padding()
                
                // Todo list
                if store.isLoading {
                    ProgressView("กำลังโหลด...")
                        .frame(maxWidth: .infinity, maxHeight: .infinity)
                } else {
                    List {
                        ForEach(store.scope(
                            state: \.filteredTodos,
                            action: \.todos
                        )) { rowStore in
                            TodoRowView(store: rowStore)
                        }
                    }
                    .listStyle(.plain)
                }
                
                // Add todo input
                addTodoInput
            }
            .navigationTitle("My Todos")
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button("ลบที่เสร็จแล้ว") {
                        store.send(.clearCompletedButtonTapped)
                    }
                    .disabled(store.stats.completed == 0)
                }
            }
            .task {
                store.send(.onAppear)
            }
        }
    }
    
    var statsBar: some View {
        HStack {
            statItem(label: "ทั้งหมด", count: store.stats.total, color: .blue)
            Divider()
            statItem(label: "ยังไม่เสร็จ", count: store.stats.pending, color: .orange)
            Divider()
            statItem(label: "เสร็จแล้ว", count: store.stats.completed, color: .green)
        }
        .padding()
        .background(Color(.systemGroupedBackground))
    }
    
    func statItem(label: String, count: Int, color: Color) -> some View {
        VStack(spacing: 4) {
            Text("\(count)")
                .font(.title2.bold())
                .foregroundStyle(color)
            Text(label)
                .font(.caption)
                .foregroundStyle(.secondary)
        }
        .frame(maxWidth: .infinity)
    }
    
    var addTodoInput: some View {
        HStack {
            TextField("เพิ่ม todo ใหม่...", text: $store.newTodoTitle.sending(\.newTodoTitleChanged))
                .textFieldStyle(.roundedBorder)
                .onSubmit {
                    store.send(.addTodoButtonTapped)
                }
            
            Button {
                store.send(.addTodoButtonTapped)
            } label: {
                Image(systemName: "plus.circle.fill")
                    .font(.title2)
            }
            .disabled(store.newTodoTitle.trimmingCharacters(in: .whitespaces).isEmpty)
        }
        .padding()
        .background(Color(.systemBackground))
        .shadow(radius: 1)
    }
}

struct TodoRowView: View {
    @Bindable var store: StoreOf<TodoRowFeature>
    
    var body: some View {
        HStack {
            Button {
                store.send(.toggleCompletedTapped)
            } label: {
                Image(systemName: store.todo.isCompleted ? "checkmark.circle.fill" : "circle")
                    .font(.title3)
                    .foregroundStyle(store.todo.isCompleted ? .green : .secondary)
            }
            .buttonStyle(.plain)
            
            if store.isEditing {
                TextField("Title", text: $store.editTitle.sending(\.editTitleChanged))
                    .textFieldStyle(.roundedBorder)
                    .onSubmit {
                        store.send(.saveEditTapped)
                    }
                
                Button("บันทึก") { store.send(.saveEditTapped) }
                    .tint(.blue)
                Button("ยกเลิก") { store.send(.cancelEditTapped) }
                    .tint(.secondary)
            } else {
                VStack(alignment: .leading) {
                    Text(store.todo.title)
                        .strikethrough(store.todo.isCompleted)
                        .foregroundStyle(store.todo.isCompleted ? .secondary : .primary)
                    
                    Text(store.todo.priority.rawValue)
                        .font(.caption)
                        .foregroundStyle(store.todo.priority.color)
                }
                
                Spacer()
                
                Button {
                    store.send(.editButtonTapped)
                } label: {
                    Image(systemName: "pencil")
                }
                .buttonStyle(.plain)
                .foregroundStyle(.blue)
                
                Button {
                    store.send(.deleteTapped)
                } label: {
                    Image(systemName: "trash")
                }
                .buttonStyle(.plain)
                .foregroundStyle(.red)
            }
        }
        .padding(.vertical, 4)
    }
}

// Tests
class TodoListTests: XCTestCase {
    
    @MainActor
    func testAddTodo() async {
        var todos: [Todo] = []
        
        let store = TestStore(
            initialState: TodoListFeature.State()
        ) {
            TodoListFeature()
        } withDependencies: {
            $0.uuid = .incrementing
            $0.date = .constant(Date(timeIntervalSince1970: 0))
            $0.todoStorage.load = { [] }
            $0.todoStorage.save = { todos = $0 }
        }
        
        await store.send(.newTodoTitleChanged("Buy groceries")) {
            $0.newTodoTitle = "Buy groceries"
        }
        
        await store.send(.addTodoButtonTapped) {
            $0.newTodoTitle = ""
            $0.todos = [
                TodoRowFeature.State(
                    todo: Todo(
                        id: UUID(0),
                        title: "Buy groceries",
                        createdAt: Date(timeIntervalSince1970: 0)
                    )
                )
            ]
        }
        
        XCTAssertEqual(todos.count, 1)
        XCTAssertEqual(todos[0].title, "Buy groceries")
    }
}
```

---

## 25. Building a Complete Weather App with TCA

### 25.1 Weather App Architecture

```swift
import ComposableArchitecture
import SwiftUI

// MARK: - Models
struct WeatherData: Equatable, Codable {
    let city: String
    let temperature: Double
    let feelsLike: Double
    let humidity: Int
    let description: String
    let icon: String
    let forecast: [DailyForecast]
    
    struct DailyForecast: Equatable, Codable, Identifiable {
        let id: String
        let date: Date
        let high: Double
        let low: Double
        let description: String
        let icon: String
    }
}

// MARK: - Weather API Client
struct WeatherAPIClient {
    var fetchWeather: @Sendable (_ city: String) async throws -> WeatherData
    var fetchWeatherByLocation: @Sendable (_ lat: Double, _ lon: Double) async throws -> WeatherData
    var searchCities: @Sendable (_ query: String) async throws -> [String]
}

extension WeatherAPIClient: DependencyKey {
    static let liveValue = WeatherAPIClient(
        fetchWeather: { city in
            let encoded = city.addingPercentEncoding(withAllowedCharacters: .urlQueryAllowed) ?? city
            let url = URL(string: "https://api.openweathermap.org/data/2.5/weather?q=\(encoded)&appid=YOUR_KEY&units=metric")!
            let (data, _) = try await URLSession.shared.data(from: url)
            // Parse response...
            fatalError("Implement parsing")
        },
        fetchWeatherByLocation: { lat, lon in
            fatalError("Implement")
        },
        searchCities: { query in
            fatalError("Implement")
        }
    )
    
    static let testValue = WeatherAPIClient(
        fetchWeather: { city in
            WeatherData(
                city: city,
                temperature: 30,
                feelsLike: 34,
                humidity: 70,
                description: "อากาศร้อน ชื้น",
                icon: "01d",
                forecast: []
            )
        },
        fetchWeatherByLocation: { _, _ in fatalError() },
        searchCities: { _ in ["Bangkok", "Chiang Mai", "Phuket"] }
    )
}

extension DependencyValues {
    var weatherAPI: WeatherAPIClient {
        get { self[WeatherAPIClient.self] }
        set { self[WeatherAPIClient.self] = newValue }
    }
}

// MARK: - Features

@Reducer
struct WeatherSearchFeature {
    
    @ObservableState
    struct State: Equatable {
        var query = ""
        var suggestions: [String] = []
        var isSearching = false
    }
    
    enum Action {
        case queryChanged(String)
        case suggestionsLoaded([String])
        case citySelected(String)
        
        enum Delegate {
            case citySelected(String)
        }
        case delegate(Delegate)
    }
    
    enum CancelID { case search }
    
    @Dependency(\.weatherAPI) var api
    
    var body: some ReducerOf<Self> {
        Reduce { state, action in
            switch action {
            case .queryChanged(let query):
                state.query = query
                
                if query.count < 2 {
                    state.suggestions = []
                    return .cancel(id: CancelID.search)
                }
                
                state.isSearching = true
                return .run { send in
                    try await Task.sleep(for: .milliseconds(300))
                    let cities = try await api.searchCities(query)
                    await send(.suggestionsLoaded(cities))
                }
                .cancellable(id: CancelID.search, cancelInFlight: true)
                
            case .suggestionsLoaded(let cities):
                state.isSearching = false
                state.suggestions = cities
                return .none
                
            case .citySelected(let city):
                return .send(.delegate(.citySelected(city)))
                
            case .delegate:
                return .none
            }
        }
    }
}

@Reducer
struct WeatherDetailFeature {
    
    @ObservableState
    struct State: Equatable {
        var city: String
        var weather: WeatherData?
        var isLoading = false
        var error: String?
    }
    
    enum Action {
        case onAppear
        case refreshButtonTapped
        case weatherLoaded(Result<WeatherData, Error>)
    }
    
    @Dependency(\.weatherAPI) var api
    
    var body: some ReducerOf<Self> {
        Reduce { state, action in
            switch action {
            case .onAppear, .refreshButtonTapped:
                state.isLoading = true
                state.error = nil
                return .run { [city = state.city] send in
                    await send(.weatherLoaded(
                        await Result { try await api.fetchWeather(city) }
                    ))
                }
                
            case .weatherLoaded(.success(let weather)):
                state.isLoading = false
                state.weather = weather
                return .none
                
            case .weatherLoaded(.failure(let error)):
                state.isLoading = false
                state.error = error.localizedDescription
                return .none
            }
        }
    }
}

@Reducer
struct WeatherAppFeature {
    
    @ObservableState
    struct State: Equatable {
        var search = WeatherSearchFeature.State()
        var savedCities: [String] = ["Bangkok", "Chiang Mai"]
        var selectedCity: String?
        @Presents var detail: WeatherDetailFeature.State?
    }
    
    enum Action {
        case search(WeatherSearchFeature.Action)
        case cityTapped(String)
        case detail(PresentationAction<WeatherDetailFeature.Action>)
    }
    
    var body: some ReducerOf<Self> {
        Scope(state: \.search, action: \.search) {
            WeatherSearchFeature()
        }
        
        Reduce { state, action in
            switch action {
            case .search(.delegate(.citySelected(let city))):
                state.detail = WeatherDetailFeature.State(city: city)
                if !state.savedCities.contains(city) {
                    state.savedCities.append(city)
                }
                return .none
                
            case .cityTapped(let city):
                state.detail = WeatherDetailFeature.State(city: city)
                return .none
                
            case .detail, .search:
                return .none
            }
        }
        .ifLet(\.$detail, action: \.detail) {
            WeatherDetailFeature()
        }
    }
}

// MARK: - Views

struct WeatherAppView: View {
    @Bindable var store: StoreOf<WeatherAppFeature>
    
    var body: some View {
        NavigationStack {
            VStack {
                // Search bar
                WeatherSearchView(
                    store: store.scope(state: \.search, action: \.search)
                )
                
                // Saved cities list
                List(store.savedCities, id: \.self) { city in
                    Button(city) {
                        store.send(.cityTapped(city))
                    }
                }
            }
            .navigationTitle("Weather")
        }
        .sheet(
            item: $store.scope(state: \.detail, action: \.detail)
        ) { detailStore in
            WeatherDetailView(store: detailStore)
        }
    }
}

struct WeatherSearchView: View {
    @Bindable var store: StoreOf<WeatherSearchFeature>
    
    var body: some View {
        VStack {
            HStack {
                Image(systemName: "magnifyingglass")
                    .foregroundStyle(.secondary)
                
                TextField("ค้นหาเมือง...", text: $store.query.sending(\.queryChanged))
                
                if store.isSearching {
                    ProgressView()
                        .scaleEffect(0.8)
                }
            }
            .padding()
            .background(Color(.systemGray6))
            .cornerRadius(10)
            .padding()
            
            if !store.suggestions.isEmpty {
                List(store.suggestions, id: \.self) { city in
                    Button(city) {
                        store.send(.citySelected(city))
                    }
                }
                .listStyle(.plain)
                .frame(maxHeight: 200)
            }
        }
    }
}

struct WeatherDetailView: View {
    let store: StoreOf<WeatherDetailFeature>
    
    var body: some View {
        NavigationStack {
            Group {
                if store.isLoading {
                    ProgressView("กำลังโหลด...")
                } else if let error = store.error {
                    VStack {
                        Image(systemName: "cloud.slash")
                            .font(.largeTitle)
                        Text(error)
                        Button("ลองใหม่") { store.send(.refreshButtonTapped) }
                    }
                } else if let weather = store.weather {
                    ScrollView {
                        VStack(spacing: 20) {
                            // Main weather info
                            VStack {
                                Text(weather.city)
                                    .font(.largeTitle.bold())
                                Text("\(Int(weather.temperature))°C")
                                    .font(.system(size: 64, weight: .thin))
                                Text(weather.description)
                                    .font(.title3)
                                    .foregroundStyle(.secondary)
                                Text("รู้สึกเหมือน \(Int(weather.feelsLike))°C")
                                    .foregroundStyle(.secondary)
                            }
                            
                            // Details
                            HStack {
                                weatherDetail(
                                    label: "ความชื้น",
                                    value: "\(weather.humidity)%",
                                    icon: "humidity"
                                )
                            }
                            
                            // Forecast
                            if !weather.forecast.isEmpty {
                                VStack(alignment: .leading) {
                                    Text("พยากรณ์ 7 วัน")
                                        .font(.headline)
                                    
                                    ForEach(weather.forecast) { day in
                                        HStack {
                                            Text(day.date, style: .date)
                                            Spacer()
                                            Text("\(Int(day.high))° / \(Int(day.low))°")
                                        }
                                    }
                                }
                                .padding()
                                .background(Color(.systemGray6))
                                .cornerRadius(12)
                            }
                        }
                        .padding()
                    }
                }
            }
            .navigationTitle(store.city)
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                Button {
                    store.send(.refreshButtonTapped)
                } label: {
                    Image(systemName: "arrow.clockwise")
                }
            }
            .task {
                store.send(.onAppear)
            }
        }
    }
    
    func weatherDetail(label: String, value: String, icon: String) -> some View {
        VStack {
            Label(label, systemImage: icon)
                .font(.caption)
                .foregroundStyle(.secondary)
            Text(value)
                .font(.title3.bold())
        }
        .frame(maxWidth: .infinity)
        .padding()
        .background(Color(.systemGray6))
        .cornerRadius(12)
    }
}
```

---

## 26. Summary

ในบทนี้เราได้เรียนรู้ The Composable Architecture (TCA) ครบถ้วน:

### Core Concepts
1. **@Reducer Macro** - สร้าง feature ด้วย `@Reducer` macro ที่ทำให้โค้ดสั้นลงและ type-safe

2. **State** - ข้อมูลทั้งหมดของ feature เป็น struct ที่ Equatable
   - `@BindingState` สำหรับ two-way binding
   - `@ObservableState` สำหรับ modern API

3. **Action** - enum ที่ครอบคลุมทุก event
   - Naming convention: past tense สำหรับ UI events
   - Delegate actions สำหรับสื่อสารกับ parent

4. **Effect** - side effects ที่ return จาก reducer
   - `.none` - ไม่มี effect
   - `.run` - async effect
   - `.cancel` - cancel effect ที่กำลังทำงาน
   - `.send` - ส่ง action อื่น

5. **Store** - runtime ที่รัน reducer และจัดการ state
   - `StoreOf<Feature>` - type alias ที่กระชับ
   - `@Bindable var store` - สำหรับ two-way binding

### Dependency Management
6. **@Dependency** - inject dependencies
7. **DependencyKey** - protocol สำหรับ register dependency
8. **DependencyValues** - registry สำหรับ dependencies

### Composition
9. **Scope** - embed child reducer ใน parent
10. **IfLetReducer** - สำหรับ optional child state
11. **ForEachReducer** - สำหรับ collection ของ child features

### Navigation
12. **PresentationState** - `@Presents` สำหรับ sheets, alerts, navigation
13. **StackState** - สำหรับ NavigationStack

### Testing
14. **TestStore** - test store ที่ exhaustive โดย default
15. **TestClock** - control time ใน tests
16. **withDependencies** - override dependencies สำหรับ tests

### Key Takeaways

- **Unidirectional data flow** - ง่ายต่อการ debug และ test
- **Exhaustive testing** - TestStore บังคับให้ assert ทุก state change
- **Dependency injection** - ทำให้ test ได้ง่ายโดยไม่ต้อง mock URLSession จริงๆ
- **Composition** - แบ่ง feature ใหญ่เป็น feature เล็กๆ ที่ test ได้แยกกัน
- **Navigation first-class** - TCA จัดการ navigation state เป็น first-class citizen

### เมื่อไหรควรใช้ TCA?

**ควรใช้:**
- App ขนาดใหญ่ที่มีหลาย features
- ทีมที่ต้องการ consistency
- เมื่อ testing เป็น priority สูง
- App ที่มี complex state management

**อาจไม่จำเป็น:**
- App เล็กๆ ที่ง่าย
- Prototype หรือ MVP เร็วๆ
- เมื่อทีมยังไม่คุ้นเคยกับ functional programming

### แหล่งเรียนรู้เพิ่มเติม

- [Point-Free TCA Documentation](https://pointfreeco.github.io/swift-composable-architecture/)
- [Point-Free Video Series](https://www.pointfree.co)
- [TCA GitHub Repository](https://github.com/pointfreeco/swift-composable-architecture)
- [TCA Case Studies](https://github.com/pointfreeco/swift-composable-architecture/tree/main/Examples)

---

*บทถัดไป: Part 73 - Advanced SwiftUI Animations and Transitions*
