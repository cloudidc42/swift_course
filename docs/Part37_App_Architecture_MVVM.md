# Part 37: App Architecture และ MVVM Pattern

## สารบัญ

1. [ความสำคัญของ Architecture](#1-ความสำคัญของ-architecture)
2. [MVC (Model-View-Controller)](#2-mvc-model-view-controller)
3. [ปัญหาและข้อจำกัดของ MVC](#3-ปัญหาและข้อจำกัดของ-mvc)
4. [MVP (Model-View-Presenter)](#4-mvp-model-view-presenter)
5. [MVVM (Model-View-ViewModel)](#5-mvvm-model-view-viewmodel)
6. [MVVM Components](#6-mvvm-components)
7. [ViewModel Implementation](#7-viewmodel-implementation)
8. [Data Binding ด้วย Combine](#8-data-binding-ด้วย-combine)
9. [MVVM กับ SwiftUI](#9-mvvm-กับ-swiftui)
10. [MVVM กับ UIKit](#10-mvvm-กับ-uikit)
11. [VIPER Pattern](#11-viper-pattern)
12. [Clean Architecture](#12-clean-architecture)
13. [Repository Pattern](#13-repository-pattern)
14. [Service Layer](#14-service-layer)
15. [Coordinator Pattern](#15-coordinator-pattern)
16. [Dependency Injection](#16-dependency-injection)
17. [Inversion of Control](#17-inversion-of-control)
18. [Protocol-based DI](#18-protocol-based-di)
19. [Swinject Framework](#19-swinject-framework)
20. [Fat ViewModels Problem](#20-fat-viewmodels-problem)
21. [UseCase Pattern](#21-usecase-pattern)
22. [แบบฝึกหัด](#22-แบบฝึกหัด)
23. [การสร้าง MVVM App สมบูรณ์](#23-การสร้าง-mvvm-app-สมบูรณ์)
24. [สรุป](#24-สรุป)

---

## 1. ความสำคัญของ Architecture

การออกแบบสถาปัตยกรรมแอปพลิเคชัน (App Architecture) คือการจัดระเบียบโค้ดให้มีโครงสร้างที่ชัดเจน ทำให้ง่ายต่อการดูแลรักษา ทดสอบ และขยายระบบในอนาคต

### ทำไม Architecture ถึงสำคัญ?

```swift
// โค้ดที่ไม่มี Architecture (Spaghetti Code)
class MessyViewController: UIViewController {
    var tableView: UITableView!
    var users: [User] = []
    var isLoading = false
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // เรียก API โดยตรงใน ViewController
        URLSession.shared.dataTask(with: URL(string: "https://api.example.com/users")!) { data, _, error in
            if let data = data {
                // Parse JSON โดยตรงใน ViewController
                if let json = try? JSONSerialization.jsonObject(with: data) as? [[String: Any]] {
                    self.users = json.compactMap { dict in
                        guard let id = dict["id"] as? Int,
                              let name = dict["name"] as? String else { return nil }
                        return User(id: id, name: name)
                    }
                    // Update UI
                    DispatchQueue.main.async {
                        self.tableView.reloadData()
                    }
                }
            }
        }.resume()
    }
    
    // Validation, Business Logic, และ UI ปนกันหมด
    func saveUser(_ name: String, email: String) {
        guard !name.isEmpty else {
            showAlert("Error", message: "Name is required")
            return
        }
        guard email.contains("@") else {
            showAlert("Error", message: "Invalid email")
            return
        }
        // บันทึกลง Database โดยตรง
        let context = (UIApplication.shared.delegate as! AppDelegate).persistentContainer.viewContext
        let user = NSEntityDescription.insertNewObject(forEntityName: "User", into: context)
        user.setValue(name, forKey: "name")
        user.setValue(email, forKey: "email")
        try? context.save()
        tableView.reloadData()
    }
}
```

ปัญหาของโค้ดข้างต้น:
- **ทดสอบยาก**: ไม่สามารถ Unit Test ได้เพราะ Logic อยู่ใน ViewController
- **ดูแลรักษายาก**: เมื่อ Requirements เปลี่ยน ต้องแก้ไขหลายที่
- **นำกลับมาใช้ใหม่ไม่ได้**: Business Logic ผูกติดกับ UI
- **ทำงานร่วมกันยาก**: หลายคนแก้ไขไฟล์เดียวกันพร้อมกัน

### หลักการสำคัญของ Good Architecture

1. **Separation of Concerns (SoC)**: แต่ละส่วนรับผิดชอบเฉพาะสิ่งที่ตัวเองควรทำ
2. **Single Responsibility Principle (SRP)**: แต่ละ Class/Module มีหน้าที่เดียว
3. **Testability**: โค้ดต้องทดสอบได้ง่าย
4. **Maintainability**: แก้ไขและขยายได้ง่าย
5. **Reusability**: นำกลับมาใช้ใหม่ได้

---

## 2. MVC (Model-View-Controller)

MVC เป็น Pattern ที่ Apple ใช้เป็น Default Architecture สำหรับ iOS Development

### โครงสร้าง MVC

```
┌─────────────┐     Action      ┌─────────────────┐
│    View     │ ─────────────→  │   Controller    │
│  (UI Layer) │                 │ (Business Logic) │
│             │ ←──────────── │                  │
└─────────────┘   Update UI    └────────┬────────┘
                                        │
                                   Read/Write
                                        │
                               ┌────────▼────────┐
                               │      Model      │
                               │   (Data Layer)  │
                               └─────────────────┘
```

### การ Implement MVC แบบ Apple

```swift
// Model
struct User: Codable {
    let id: Int
    let name: String
    let email: String
    let avatar: String?
}

struct UserResponse: Codable {
    let users: [User]
    let total: Int
}

// Model - Data Manager
class UserDataManager {
    func fetchUsers(completion: @escaping (Result<[User], Error>) -> Void) {
        let url = URL(string: "https://api.example.com/users")!
        URLSession.shared.dataTask(with: url) { data, _, error in
            if let error = error {
                completion(.failure(error))
                return
            }
            guard let data = data else {
                completion(.failure(NSError(domain: "", code: -1)))
                return
            }
            do {
                let response = try JSONDecoder().decode(UserResponse.self, from: data)
                completion(.success(response.users))
            } catch {
                completion(.failure(error))
            }
        }.resume()
    }
}

// Controller
class UserListViewController: UIViewController {
    // IBOutlets (View)
    @IBOutlet weak var tableView: UITableView!
    @IBOutlet weak var activityIndicator: UIActivityIndicatorView!
    
    // Model
    private let dataManager = UserDataManager()
    private var users: [User] = []
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupTableView()
        loadUsers()
    }
    
    private func setupTableView() {
        tableView.dataSource = self
        tableView.delegate = self
        tableView.register(UITableViewCell.self, forCellReuseIdentifier: "UserCell")
    }
    
    private func loadUsers() {
        activityIndicator.startAnimating()
        dataManager.fetchUsers { [weak self] result in
            DispatchQueue.main.async {
                self?.activityIndicator.stopAnimating()
                switch result {
                case .success(let users):
                    self?.users = users
                    self?.tableView.reloadData()
                case .failure(let error):
                    self?.showError(error)
                }
            }
        }
    }
    
    private func showError(_ error: Error) {
        let alert = UIAlertController(title: "Error", message: error.localizedDescription, preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "OK", style: .default))
        present(alert, animated: true)
    }
}

// Controller as DataSource
extension UserListViewController: UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return users.count
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "UserCell", for: indexPath)
        let user = users[indexPath.row]
        cell.textLabel?.text = user.name
        cell.detailTextLabel?.text = user.email
        return cell
    }
}

// Controller as Delegate
extension UserListViewController: UITableViewDelegate {
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        tableView.deselectRow(at: indexPath, animated: true)
        let user = users[indexPath.row]
        navigateToDetail(user: user)
    }
    
    private func navigateToDetail(user: User) {
        let detailVC = UserDetailViewController(user: user)
        navigationController?.pushViewController(detailVC, animated: true)
    }
}
```

---

## 3. ปัญหาและข้อจำกัดของ MVC

### Massive View Controller (MVC = Massive View Controller)

ปัญหาหลักของ MVC ใน iOS คือ ViewController มักจะกลายเป็น "God Object" ที่รับผิดชอบทุกอย่าง

```swift
// ตัวอย่าง Massive View Controller ที่มีปัญหา
class ProfileViewController: UIViewController {
    // UI Elements - ประมาณ 20+ IBOutlets
    @IBOutlet weak var avatarImageView: UIImageView!
    @IBOutlet weak var nameLabel: UILabel!
    @IBOutlet weak var emailLabel: UILabel!
    @IBOutlet weak var bioTextView: UITextView!
    @IBOutlet weak var followButton: UIButton!
    @IBOutlet weak var followersCountLabel: UILabel!
    @IBOutlet weak var followingCountLabel: UILabel!
    @IBOutlet weak var postsTableView: UITableView!
    
    // Data
    var user: User?
    var posts: [Post] = []
    var isFollowing = false
    var isLoading = false
    
    // Networking
    private let session = URLSession.shared
    private var imageCache: [String: UIImage] = [:]
    
    // Analytics
    private var viewStartTime: Date?
    private var lastInteraction: Date?
    
    // Lifecycle - มีโค้ดเยอะมาก
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
        loadUserProfile()
        loadUserPosts()
        setupNotifications()
        trackAnalytics()
        // ... อีกสิบกว่า method calls
    }
    
    // Networking methods ที่ควรอยู่ใน separate layer
    private func loadUserProfile() { /* ... */ }
    private func loadUserPosts() { /* ... */ }
    private func loadAvatar() { /* ... */ }
    private func followUser() { /* ... */ }
    private func unfollowUser() { /* ... */ }
    
    // Business Logic ที่ควรอยู่ใน ViewModel หรือ Use Case
    private func shouldShowFollowButton() -> Bool { /* ... */ }
    private func formatFollowersCount(_ count: Int) -> String { /* ... */ }
    private func canUserPost() -> Bool { /* ... */ }
    
    // UI Logic
    private func setupUI() { /* ... */ }
    private func updateFollowButton() { /* ... */ }
    private func animateAvatarLoad() { /* ... */ }
    
    // Navigation
    private func showFollowersList() { /* ... */ }
    private func showPostDetail(_ post: Post) { /* ... */ }
    private func showEditProfile() { /* ... */ }
    
    // Analytics
    private func trackAnalytics() { /* ... */ }
    private func trackInteraction(_ action: String) { /* ... */ }
    
    // Error Handling
    private func handleNetworkError(_ error: Error) { /* ... */ }
    private func handleAuthError() { /* ... */ }
    
    // DataSource, Delegate methods - อีก 10+ methods
}
// ไฟล์นี้อาจมี 1000+ บรรทัด!
```

### ปัญหาหลักของ MVC

**1. Unit Testing ยาก**
```swift
// ทดสอบได้ยากเพราะ logic อยู่ใน ViewController ที่ต้องการ UIKit
class UserListViewControllerTests: XCTestCase {
    func testLoadUsers() {
        // ต้อง instantiate ViewController ทั้งหมด
        let vc = UserListViewController()
        // ต้อง load view เพื่อให้ viewDidLoad ทำงาน
        _ = vc.view
        // รอ async call - ยุ่งยากมาก
    }
}
```

**2. View และ Controller แยกยาก**
```swift
// ใน UIKit, ViewController เป็นทั้ง View และ Controller
// ทำให้ Separation of Concerns ไม่ชัดเจน
class UserViewController: UIViewController {
    // นี่คือ View หรือ Controller?
    // Apple บอกว่า UIViewController คือ Controller
    // แต่มัน manage views โดยตรง
}
```

**3. Reusability ต่ำ**
```swift
// Business Logic ผูกติดกับ UIViewController
// ไม่สามารถนำ logic ไปใช้ใน watchOS หรือ macOS ได้ง่าย
```

---

## 4. MVP (Model-View-Presenter)

MVP แก้ปัญหา MVC โดยแยก Logic ออกจาก View อย่างชัดเจน

### โครงสร้าง MVP

```
┌───────────────┐   User Action   ┌──────────────┐
│     View      │ ──────────────→ │  Presenter   │
│ (Passive View)│                 │(Logic Layer) │
│               │ ←────────────  │              │
└───────────────┘  Update UI      └──────┬───────┘
                                         │
                                    Read/Write
                                         │
                                ┌────────▼───────┐
                                │     Model      │
                                │  (Data Layer)  │
                                └────────────────┘
```

### การ Implement MVP

```swift
// Protocol สำหรับ View
protocol UserListViewProtocol: AnyObject {
    func showLoading()
    func hideLoading()
    func showUsers(_ users: [User])
    func showError(_ message: String)
    func showEmptyState()
}

// Protocol สำหรับ Presenter
protocol UserListPresenterProtocol: AnyObject {
    var view: UserListViewProtocol? { get set }
    func viewDidLoad()
    func didSelectUser(at index: Int)
    func didPullToRefresh()
}

// Presenter - ไม่มี UIKit import!
class UserListPresenter: UserListPresenterProtocol {
    weak var view: UserListViewProtocol?
    
    private let userService: UserServiceProtocol
    private var users: [User] = []
    
    init(userService: UserServiceProtocol) {
        self.userService = userService
    }
    
    func viewDidLoad() {
        loadUsers()
    }
    
    func didPullToRefresh() {
        loadUsers()
    }
    
    private func loadUsers() {
        view?.showLoading()
        userService.fetchUsers { [weak self] result in
            DispatchQueue.main.async {
                self?.view?.hideLoading()
                switch result {
                case .success(let users):
                    self?.users = users
                    if users.isEmpty {
                        self?.view?.showEmptyState()
                    } else {
                        self?.view?.showUsers(users)
                    }
                case .failure(let error):
                    self?.view?.showError(error.localizedDescription)
                }
            }
        }
    }
    
    func didSelectUser(at index: Int) {
        guard index < users.count else { return }
        let user = users[index]
        // Navigation logic
    }
}

// View (UIViewController)
class UserListViewController: UIViewController {
    @IBOutlet weak var tableView: UITableView!
    @IBOutlet weak var activityIndicator: UIActivityIndicatorView!
    @IBOutlet weak var emptyLabel: UILabel!
    
    var presenter: UserListPresenterProtocol!
    private var users: [User] = []
    
    override func viewDidLoad() {
        super.viewDidLoad()
        presenter.view = self
        presenter.viewDidLoad()
    }
}

// View Conformance
extension UserListViewController: UserListViewProtocol {
    func showLoading() {
        activityIndicator.startAnimating()
        tableView.isHidden = true
        emptyLabel.isHidden = true
    }
    
    func hideLoading() {
        activityIndicator.stopAnimating()
    }
    
    func showUsers(_ users: [User]) {
        self.users = users
        tableView.isHidden = false
        emptyLabel.isHidden = true
        tableView.reloadData()
    }
    
    func showError(_ message: String) {
        let alert = UIAlertController(title: "Error", message: message, preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "OK", style: .default))
        present(alert, animated: true)
    }
    
    func showEmptyState() {
        tableView.isHidden = true
        emptyLabel.isHidden = false
        emptyLabel.text = "No users found"
    }
}

// Unit Testing ง่ายมาก!
class UserListPresenterTests: XCTestCase {
    var presenter: UserListPresenter!
    var mockView: MockUserListView!
    var mockService: MockUserService!
    
    override func setUp() {
        mockView = MockUserListView()
        mockService = MockUserService()
        presenter = UserListPresenter(userService: mockService)
        presenter.view = mockView
    }
    
    func testViewDidLoadShowsLoading() {
        presenter.viewDidLoad()
        XCTAssertTrue(mockView.didShowLoading)
    }
    
    func testSuccessfulLoadShowsUsers() {
        let users = [User(id: 1, name: "Alice", email: "alice@test.com", avatar: nil)]
        mockService.stubbedResult = .success(users)
        
        presenter.viewDidLoad()
        
        XCTAssertEqual(mockView.displayedUsers?.count, 1)
        XCTAssertEqual(mockView.displayedUsers?.first?.name, "Alice")
    }
}

// Mock objects สำหรับ Testing
class MockUserListView: UserListViewProtocol {
    var didShowLoading = false
    var didHideLoading = false
    var displayedUsers: [User]?
    var displayedError: String?
    var didShowEmptyState = false
    
    func showLoading() { didShowLoading = true }
    func hideLoading() { didHideLoading = true }
    func showUsers(_ users: [User]) { displayedUsers = users }
    func showError(_ message: String) { displayedError = message }
    func showEmptyState() { didShowEmptyState = true }
}
```

---

## 5. MVVM (Model-View-ViewModel)

MVVM เป็น Architecture ที่เหมาะสมมากกับ SwiftUI และ Reactive Programming

### โครงสร้าง MVVM

```
┌───────────┐    Binds to    ┌──────────────┐
│   View    │ ─────────────→ │  ViewModel   │
│(UI Layer) │                │(Logic Layer) │
│           │ ←────────────  │              │
└───────────┘  Observes      └──────┬───────┘
                                    │
                               Read/Write
                                    │
                           ┌────────▼────────┐
                           │     Model       │
                           │  (Data + Logic) │
                           └─────────────────┘
```

### ความแตกต่างระหว่าง MVP และ MVVM

| Feature | MVP | MVVM |
|---------|-----|------|
| View Knowledge | Presenter รู้จัก View | ViewModel ไม่รู้จัก View |
| Data Binding | Manual Update | Automatic Binding |
| Testability | ดี | ดีมาก |
| Complexity | ปานกลาง | ปานกลาง-สูง |
| ใช้กับ | UIKit | SwiftUI, UIKit+Combine |

---

## 6. MVVM Components

### Model Layer

```swift
// Domain Model - Pure Swift, ไม่มี Framework dependency
struct Repository: Identifiable, Codable {
    let id: Int
    let name: String
    let fullName: String
    let description: String?
    let starCount: Int
    let forkCount: Int
    let language: String?
    let isPrivate: Bool
    let updatedAt: Date
    let owner: Owner
    
    struct Owner: Codable {
        let id: Int
        let login: String
        let avatarURL: String
        
        enum CodingKeys: String, CodingKey {
            case id
            case login
            case avatarURL = "avatar_url"
        }
    }
    
    enum CodingKeys: String, CodingKey {
        case id
        case name
        case fullName = "full_name"
        case description
        case starCount = "stargazers_count"
        case forkCount = "forks_count"
        case language
        case isPrivate = "private"
        case updatedAt = "updated_at"
        case owner
    }
}

// Value Object
struct SearchQuery {
    let keyword: String
    let language: String?
    let sortBy: SortOption
    let order: Order
    
    enum SortOption: String {
        case stars
        case forks
        case updated
        case bestMatch = "best-match"
    }
    
    enum Order: String {
        case asc
        case desc
    }
    
    // Validation
    var isValid: Bool {
        return !keyword.trimmingCharacters(in: .whitespaces).isEmpty
    }
}
```

### ViewModel Layer

```swift
// ViewModel Protocol
protocol RepositoryListViewModelProtocol: ObservableObject {
    var repositories: [Repository] { get }
    var isLoading: Bool { get }
    var errorMessage: String? { get }
    var searchText: String { get set }
    
    func search()
    func loadMore()
    func refresh()
    func selectRepository(_ repository: Repository)
}

// ViewModel Implementation
@MainActor
class RepositoryListViewModel: RepositoryListViewModelProtocol {
    @Published var repositories: [Repository] = []
    @Published var isLoading = false
    @Published var errorMessage: String?
    @Published var searchText = ""
    
    private let searchUseCase: SearchRepositoriesUseCaseProtocol
    private let coordinator: AppCoordinatorProtocol
    private var currentPage = 1
    private var hasMorePages = true
    private var searchTask: Task<Void, Never>?
    
    init(
        searchUseCase: SearchRepositoriesUseCaseProtocol,
        coordinator: AppCoordinatorProtocol
    ) {
        self.searchUseCase = searchUseCase
        self.coordinator = coordinator
    }
    
    func search() {
        guard !searchText.isEmpty else { return }
        currentPage = 1
        repositories = []
        performSearch()
    }
    
    func loadMore() {
        guard !isLoading && hasMorePages else { return }
        currentPage += 1
        performSearch()
    }
    
    func refresh() {
        currentPage = 1
        repositories = []
        performSearch()
    }
    
    private func performSearch() {
        searchTask?.cancel()
        searchTask = Task {
            isLoading = true
            errorMessage = nil
            
            let query = SearchQuery(
                keyword: searchText,
                language: nil,
                sortBy: .stars,
                order: .desc
            )
            
            do {
                let result = try await searchUseCase.execute(query: query, page: currentPage)
                repositories.append(contentsOf: result.items)
                hasMorePages = result.hasNextPage
            } catch is CancellationError {
                // ถูก cancel - ไม่ต้องทำอะไร
            } catch {
                errorMessage = error.localizedDescription
            }
            
            isLoading = false
        }
    }
    
    func selectRepository(_ repository: Repository) {
        coordinator.showRepositoryDetail(repository)
    }
}
```

---

## 7. ViewModel Implementation

### Best Practices สำหรับ ViewModel

```swift
// 1. ViewModel ไม่ควร import UIKit
import Foundation
import Combine

// 2. ใช้ @Published สำหรับ SwiftUI
class UserProfileViewModel: ObservableObject {
    // Output - สิ่งที่ View แสดงผล
    @Published private(set) var user: User?
    @Published private(set) var posts: [Post] = []
    @Published private(set) var isLoading = false
    @Published private(set) var error: AppError?
    @Published private(set) var isFollowing = false
    
    // Computed properties สำหรับ UI
    var displayName: String {
        guard let user = user else { return "Loading..." }
        return user.name
    }
    
    var followersText: String {
        guard let user = user else { return "" }
        return formatCount(user.followersCount)
    }
    
    var followButtonTitle: String {
        isFollowing ? "Following" : "Follow"
    }
    
    var followButtonStyle: ButtonStyle {
        isFollowing ? .secondary : .primary
    }
    
    // Input - Actions จาก View
    let onViewAppeared = PassthroughSubject<Void, Never>()
    let onFollowTapped = PassthroughSubject<Void, Never>()
    let onRefreshRequested = PassthroughSubject<Void, Never>()
    
    private var cancellables = Set<AnyCancellable>()
    private let userService: UserServiceProtocol
    private let analyticsService: AnalyticsServiceProtocol
    private let userId: Int
    
    init(
        userId: Int,
        userService: UserServiceProtocol,
        analyticsService: AnalyticsServiceProtocol
    ) {
        self.userId = userId
        self.userService = userService
        self.analyticsService = analyticsService
        
        setupBindings()
    }
    
    private func setupBindings() {
        // เมื่อ view appear ให้ load data
        onViewAppeared
            .sink { [weak self] in
                self?.loadProfile()
                self?.analyticsService.track("profile_viewed")
            }
            .store(in: &cancellables)
        
        // เมื่อ follow tapped
        onFollowTapped
            .sink { [weak self] in
                self?.toggleFollow()
            }
            .store(in: &cancellables)
        
        // เมื่อ pull to refresh
        onRefreshRequested
            .sink { [weak self] in
                self?.loadProfile()
            }
            .store(in: &cancellables)
    }
    
    private func loadProfile() {
        isLoading = true
        error = nil
        
        Task { @MainActor in
            do {
                async let userResult = userService.fetchUser(id: userId)
                async let postsResult = userService.fetchUserPosts(userId: userId)
                
                let (user, posts) = try await (userResult, postsResult)
                self.user = user
                self.posts = posts
                self.isFollowing = user.isFollowedByCurrentUser
            } catch {
                self.error = AppError(from: error)
            }
            isLoading = false
        }
    }
    
    private func toggleFollow() {
        guard let user = user else { return }
        
        Task { @MainActor in
            do {
                if isFollowing {
                    try await userService.unfollowUser(id: user.id)
                } else {
                    try await userService.followUser(id: user.id)
                }
                isFollowing.toggle()
                analyticsService.track(isFollowing ? "user_followed" : "user_unfollowed")
            } catch {
                self.error = AppError(from: error)
            }
        }
    }
    
    // Helper
    private func formatCount(_ count: Int) -> String {
        if count >= 1_000_000 {
            return String(format: "%.1fM", Double(count) / 1_000_000)
        } else if count >= 1_000 {
            return String(format: "%.1fK", Double(count) / 1_000)
        }
        return "\(count)"
    }
}

enum ButtonStyle {
    case primary
    case secondary
}
```

---

## 8. Data Binding ด้วย Combine

### Combine Basics สำหรับ MVVM

```swift
import Combine

// Publisher Types
class DataBindingExample {
    
    // 1. @Published - ง่ายที่สุด
    @Published var name: String = ""
    
    // 2. PassthroughSubject - ส่ง events ด้วยตนเอง
    let userDidLogin = PassthroughSubject<User, Never>()
    
    // 3. CurrentValueSubject - มี current value
    let currentUser = CurrentValueSubject<User?, Never>(nil)
    
    func demonstrateBindings() {
        var cancellables = Set<AnyCancellable>()
        
        // Observe @Published
        $name
            .filter { !$0.isEmpty }
            .debounce(for: .milliseconds(300), scheduler: RunLoop.main)
            .removeDuplicates()
            .sink { name in
                print("Name changed to: \(name)")
            }
            .store(in: &cancellables)
        
        // Combine publishers
        let firstName = Just("John")
        let lastName = Just("Doe")
        
        Publishers.CombineLatest(firstName, lastName)
            .map { "\($0) \($1)" }
            .sink { fullName in
                print("Full name: \(fullName)")
            }
            .store(in: &cancellables)
    }
}

// Search ViewModel ที่ใช้ Combine
class SearchViewModel: ObservableObject {
    @Published var searchText = ""
    @Published private(set) var results: [SearchResult] = []
    @Published private(set) var isSearching = false
    
    private let searchService: SearchServiceProtocol
    private var cancellables = Set<AnyCancellable>()
    
    init(searchService: SearchServiceProtocol) {
        self.searchService = searchService
        setupSearch()
    }
    
    private func setupSearch() {
        // การ implement search ที่ดี
        $searchText
            .debounce(for: .milliseconds(500), scheduler: DispatchQueue.main)
            .removeDuplicates()
            .filter { $0.count >= 2 }
            .handleEvents(receiveOutput: { [weak self] _ in
                self?.isSearching = true
            })
            .flatMap { [weak self] query -> AnyPublisher<[SearchResult], Never> in
                guard let self = self else { return Just([]).eraseToAnyPublisher() }
                return self.searchService.search(query: query)
                    .catch { _ in Just([]) }
                    .eraseToAnyPublisher()
            }
            .receive(on: DispatchQueue.main)
            .sink { [weak self] results in
                self?.results = results
                self?.isSearching = false
            }
            .store(in: &cancellables)
    }
}

// Form Validation ด้วย Combine
class RegistrationViewModel: ObservableObject {
    @Published var username = ""
    @Published var email = ""
    @Published var password = ""
    @Published var confirmPassword = ""
    
    @Published private(set) var usernameError: String?
    @Published private(set) var emailError: String?
    @Published private(set) var passwordError: String?
    @Published private(set) var isFormValid = false
    
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        setupValidation()
    }
    
    private func setupValidation() {
        // Username validation
        $username
            .map { username -> String? in
                if username.isEmpty { return nil }
                if username.count < 3 { return "Username must be at least 3 characters" }
                if username.count > 20 { return "Username must be less than 20 characters" }
                if !username.allSatisfy({ $0.isLetter || $0.isNumber || $0 == "_" }) {
                    return "Username can only contain letters, numbers, and underscores"
                }
                return nil
            }
            .assign(to: &$usernameError)
        
        // Email validation
        $email
            .map { email -> String? in
                if email.isEmpty { return nil }
                let emailRegex = "[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}"
                let predicate = NSPredicate(format: "SELF MATCHES %@", emailRegex)
                return predicate.evaluate(with: email) ? nil : "Invalid email format"
            }
            .assign(to: &$emailError)
        
        // Password validation
        $password
            .map { password -> String? in
                if password.isEmpty { return nil }
                if password.count < 8 { return "Password must be at least 8 characters" }
                if !password.contains(where: { $0.isUppercase }) { return "Password must contain uppercase letter" }
                if !password.contains(where: { $0.isNumber }) { return "Password must contain a number" }
                return nil
            }
            .assign(to: &$passwordError)
        
        // Form validity
        Publishers.CombineLatest4($username, $email, $password, $confirmPassword)
            .map { username, email, password, confirm in
                !username.isEmpty &&
                !email.isEmpty &&
                !password.isEmpty &&
                password == confirm &&
                email.contains("@")
            }
            .assign(to: &$isFormValid)
    }
}
```

---

## 9. MVVM กับ SwiftUI

### SwiftUI + MVVM = Natural Fit

```swift
import SwiftUI
import Combine

// ViewModel
@MainActor
class TodoListViewModel: ObservableObject {
    @Published var todos: [TodoItem] = []
    @Published var newTodoTitle = ""
    @Published var filter: Filter = .all
    @Published var isLoading = false
    
    var filteredTodos: [TodoItem] {
        switch filter {
        case .all: return todos
        case .active: return todos.filter { !$0.isCompleted }
        case .completed: return todos.filter { $0.isCompleted }
        }
    }
    
    var completedCount: Int {
        todos.filter { $0.isCompleted }.count
    }
    
    var activeCount: Int {
        todos.filter { !$0.isCompleted }.count
    }
    
    private let repository: TodoRepositoryProtocol
    
    init(repository: TodoRepositoryProtocol) {
        self.repository = repository
    }
    
    func loadTodos() async {
        isLoading = true
        do {
            todos = try await repository.fetchAll()
        } catch {
            print("Error loading todos: \(error)")
        }
        isLoading = false
    }
    
    func addTodo() {
        guard !newTodoTitle.trimmingCharacters(in: .whitespaces).isEmpty else { return }
        let todo = TodoItem(
            id: UUID(),
            title: newTodoTitle,
            isCompleted: false,
            createdAt: Date()
        )
        todos.insert(todo, at: 0)
        newTodoTitle = ""
        Task { try? await repository.save(todo) }
    }
    
    func toggleTodo(_ todo: TodoItem) {
        if let index = todos.firstIndex(where: { $0.id == todo.id }) {
            todos[index].isCompleted.toggle()
            let updated = todos[index]
            Task { try? await repository.update(updated) }
        }
    }
    
    func deleteTodo(at offsets: IndexSet) {
        let todosToDelete = offsets.map { filteredTodos[$0] }
        todos.removeAll { todo in todosToDelete.contains(where: { $0.id == todo.id }) }
        Task {
            for todo in todosToDelete {
                try? await repository.delete(todo.id)
            }
        }
    }
    
    enum Filter: String, CaseIterable {
        case all = "All"
        case active = "Active"
        case completed = "Completed"
    }
}

struct TodoItem: Identifiable {
    let id: UUID
    var title: String
    var isCompleted: Bool
    let createdAt: Date
}

// View
struct TodoListView: View {
    @StateObject private var viewModel: TodoListViewModel
    
    init(viewModel: TodoListViewModel) {
        self._viewModel = StateObject(wrappedValue: viewModel)
    }
    
    var body: some View {
        NavigationView {
            VStack {
                // Filter Picker
                Picker("Filter", selection: $viewModel.filter) {
                    ForEach(TodoListViewModel.Filter.allCases, id: \.self) { filter in
                        Text(filter.rawValue).tag(filter)
                    }
                }
                .pickerStyle(.segmented)
                .padding(.horizontal)
                
                // Stats
                HStack {
                    Text("Active: \(viewModel.activeCount)")
                    Spacer()
                    Text("Completed: \(viewModel.completedCount)")
                }
                .font(.caption)
                .foregroundColor(.secondary)
                .padding(.horizontal)
                
                // Todo List
                if viewModel.isLoading {
                    ProgressView("Loading...")
                        .frame(maxWidth: .infinity, maxHeight: .infinity)
                } else {
                    List {
                        ForEach(viewModel.filteredTodos) { todo in
                            TodoRowView(todo: todo) {
                                viewModel.toggleTodo(todo)
                            }
                        }
                        .onDelete { indexSet in
                            viewModel.deleteTodo(at: indexSet)
                        }
                    }
                }
                
                // Add Todo
                HStack {
                    TextField("New todo...", text: $viewModel.newTodoTitle)
                        .textFieldStyle(.roundedBorder)
                        .onSubmit { viewModel.addTodo() }
                    
                    Button(action: viewModel.addTodo) {
                        Image(systemName: "plus.circle.fill")
                            .font(.title2)
                    }
                    .disabled(viewModel.newTodoTitle.isEmpty)
                }
                .padding()
            }
            .navigationTitle("My Todos")
            .task {
                await viewModel.loadTodos()
            }
        }
    }
}

struct TodoRowView: View {
    let todo: TodoItem
    let onToggle: () -> Void
    
    var body: some View {
        HStack {
            Button(action: onToggle) {
                Image(systemName: todo.isCompleted ? "checkmark.circle.fill" : "circle")
                    .foregroundColor(todo.isCompleted ? .green : .gray)
                    .font(.title2)
            }
            .buttonStyle(.plain)
            
            VStack(alignment: .leading) {
                Text(todo.title)
                    .strikethrough(todo.isCompleted)
                    .foregroundColor(todo.isCompleted ? .secondary : .primary)
                
                Text(todo.createdAt, style: .relative)
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
        }
        .padding(.vertical, 4)
    }
}
```

---

## 10. MVVM กับ UIKit

### UIKit + MVVM ด้วย Combine

```swift
import UIKit
import Combine

// ViewModel
class LoginViewModel: ObservableObject {
    // Inputs
    @Published var email = ""
    @Published var password = ""
    
    // Outputs
    @Published private(set) var isLoginEnabled = false
    @Published private(set) var isLoading = false
    @Published private(set) var loginError: String?
    
    let loginSucceeded = PassthroughSubject<User, Never>()
    
    private let authService: AuthServiceProtocol
    private var cancellables = Set<AnyCancellable>()
    
    init(authService: AuthServiceProtocol) {
        self.authService = authService
        
        Publishers.CombineLatest($email, $password)
            .map { email, password in
                email.count >= 5 && password.count >= 6
            }
            .assign(to: &$isLoginEnabled)
    }
    
    func login() {
        guard isLoginEnabled else { return }
        
        isLoading = true
        loginError = nil
        
        authService.login(email: email, password: password)
            .receive(on: DispatchQueue.main)
            .sink(
                receiveCompletion: { [weak self] completion in
                    self?.isLoading = false
                    if case .failure(let error) = completion {
                        self?.loginError = error.localizedDescription
                    }
                },
                receiveValue: { [weak self] user in
                    self?.loginSucceeded.send(user)
                }
            )
            .store(in: &cancellables)
    }
}

// UIViewController ที่ใช้ MVVM
class LoginViewController: UIViewController {
    // UI Elements
    private let emailTextField = UITextField()
    private let passwordTextField = UITextField()
    private let loginButton = UIButton(type: .system)
    private let activityIndicator = UIActivityIndicatorView(style: .medium)
    private let errorLabel = UILabel()
    
    private let viewModel: LoginViewModel
    private var cancellables = Set<AnyCancellable>()
    
    init(viewModel: LoginViewModel) {
        self.viewModel = viewModel
        super.init(nibName: nil, bundle: nil)
    }
    
    required init?(coder: NSCoder) {
        fatalError("Use init(viewModel:) instead")
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
        bindViewModel()
    }
    
    private func setupUI() {
        view.backgroundColor = .systemBackground
        
        // Email field
        emailTextField.placeholder = "Email"
        emailTextField.keyboardType = .emailAddress
        emailTextField.autocapitalizationType = .none
        emailTextField.borderStyle = .roundedRect
        
        // Password field
        passwordTextField.placeholder = "Password"
        passwordTextField.isSecureTextEntry = true
        passwordTextField.borderStyle = .roundedRect
        
        // Login button
        loginButton.setTitle("Login", for: .normal)
        loginButton.backgroundColor = .systemBlue
        loginButton.setTitleColor(.white, for: .normal)
        loginButton.layer.cornerRadius = 8
        loginButton.addTarget(self, action: #selector(loginButtonTapped), for: .touchUpInside)
        
        // Error label
        errorLabel.textColor = .systemRed
        errorLabel.font = .preferredFont(forTextStyle: .caption1)
        errorLabel.numberOfLines = 0
        errorLabel.isHidden = true
        
        // Stack view
        let stack = UIStackView(arrangedSubviews: [
            emailTextField,
            passwordTextField,
            errorLabel,
            loginButton,
            activityIndicator
        ])
        stack.axis = .vertical
        stack.spacing = 16
        stack.translatesAutoresizingMaskIntoConstraints = false
        
        view.addSubview(stack)
        NSLayoutConstraint.activate([
            stack.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            stack.centerYAnchor.constraint(equalTo: view.centerYAnchor),
            stack.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 24),
            stack.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -24)
        ])
    }
    
    private func bindViewModel() {
        // Bind text fields to ViewModel
        emailTextField.textPublisher
            .assign(to: \.email, on: viewModel)
            .store(in: &cancellables)
        
        passwordTextField.textPublisher
            .assign(to: \.password, on: viewModel)
            .store(in: &cancellables)
        
        // Bind ViewModel outputs to UI
        viewModel.$isLoginEnabled
            .receive(on: DispatchQueue.main)
            .sink { [weak self] isEnabled in
                self?.loginButton.isEnabled = isEnabled
                self?.loginButton.alpha = isEnabled ? 1.0 : 0.5
            }
            .store(in: &cancellables)
        
        viewModel.$isLoading
            .receive(on: DispatchQueue.main)
            .sink { [weak self] isLoading in
                if isLoading {
                    self?.activityIndicator.startAnimating()
                    self?.loginButton.isHidden = true
                } else {
                    self?.activityIndicator.stopAnimating()
                    self?.loginButton.isHidden = false
                }
            }
            .store(in: &cancellables)
        
        viewModel.$loginError
            .receive(on: DispatchQueue.main)
            .sink { [weak self] error in
                self?.errorLabel.text = error
                self?.errorLabel.isHidden = error == nil
            }
            .store(in: &cancellables)
        
        // Handle login success
        viewModel.loginSucceeded
            .receive(on: DispatchQueue.main)
            .sink { [weak self] user in
                self?.handleLoginSuccess(user: user)
            }
            .store(in: &cancellables)
    }
    
    @objc private func loginButtonTapped() {
        viewModel.login()
    }
    
    private func handleLoginSuccess(user: User) {
        // Navigate to main app
        let mainVC = MainTabBarController()
        mainVC.modalPresentationStyle = .fullScreen
        present(mainVC, animated: true)
    }
}

// Extension สำหรับ UITextField Publisher
extension UITextField {
    var textPublisher: AnyPublisher<String, Never> {
        NotificationCenter.default.publisher(for: UITextField.textDidChangeNotification, object: self)
            .compactMap { ($0.object as? UITextField)?.text }
            .eraseToAnyPublisher()
    }
}
```

---

## 11. VIPER Pattern

VIPER ย่อมาจาก View-Interactor-Presenter-Entity-Router เป็น Architecture ที่ Modular มากที่สุด

```swift
// Entity (Model)
struct Product: Codable, Identifiable {
    let id: UUID
    let name: String
    let price: Double
    let category: String
    let imageURL: String?
    
    var formattedPrice: String {
        let formatter = NumberFormatter()
        formatter.numberStyle = .currency
        return formatter.string(from: NSNumber(value: price)) ?? "\(price)"
    }
}

// Interactor Protocol
protocol ProductListInteractorProtocol: AnyObject {
    func fetchProducts(category: String?) async throws -> [Product]
    func searchProducts(query: String) async throws -> [Product]
}

// Interactor Implementation
class ProductListInteractor: ProductListInteractorProtocol {
    private let productService: ProductServiceProtocol
    
    init(productService: ProductServiceProtocol) {
        self.productService = productService
    }
    
    func fetchProducts(category: String?) async throws -> [Product] {
        return try await productService.getProducts(category: category)
    }
    
    func searchProducts(query: String) async throws -> [Product] {
        return try await productService.search(query: query)
    }
}

// Presenter Protocol
protocol ProductListPresenterProtocol: AnyObject {
    func viewDidLoad()
    func userDidSearch(query: String)
    func userDidSelectProduct(at index: Int)
    func userDidTapFilter()
}

// View Protocol
protocol ProductListViewProtocol: AnyObject {
    func showProducts(_ viewModels: [ProductCellViewModel])
    func showLoadingState()
    func hideLoadingState()
    func showError(_ message: String)
}

// Router Protocol
protocol ProductListRouterProtocol: AnyObject {
    func navigateToProductDetail(_ product: Product)
    func navigateToFilter(currentFilter: ProductFilter?, delegate: FilterDelegate)
}

// Cell ViewModel
struct ProductCellViewModel: Identifiable {
    let id: UUID
    let title: String
    let price: String
    let imageURL: URL?
    let categoryBadge: String
}

// Presenter
class ProductListPresenter: ProductListPresenterProtocol {
    weak var view: ProductListViewProtocol?
    
    private let interactor: ProductListInteractorProtocol
    private let router: ProductListRouterProtocol
    private var products: [Product] = []
    
    init(interactor: ProductListInteractorProtocol, router: ProductListRouterProtocol) {
        self.interactor = interactor
        self.router = router
    }
    
    func viewDidLoad() {
        loadProducts()
    }
    
    private func loadProducts() {
        view?.showLoadingState()
        Task { @MainActor in
            do {
                products = try await interactor.fetchProducts(category: nil)
                let viewModels = products.map { mapToViewModel($0) }
                view?.showProducts(viewModels)
            } catch {
                view?.showError(error.localizedDescription)
            }
            view?.hideLoadingState()
        }
    }
    
    func userDidSearch(query: String) {
        guard !query.isEmpty else {
            loadProducts()
            return
        }
        Task { @MainActor in
            do {
                products = try await interactor.searchProducts(query: query)
                let viewModels = products.map { mapToViewModel($0) }
                view?.showProducts(viewModels)
            } catch {
                view?.showError(error.localizedDescription)
            }
        }
    }
    
    func userDidSelectProduct(at index: Int) {
        guard index < products.count else { return }
        router.navigateToProductDetail(products[index])
    }
    
    func userDidTapFilter() {
        router.navigateToFilter(currentFilter: nil, delegate: self)
    }
    
    private func mapToViewModel(_ product: Product) -> ProductCellViewModel {
        ProductCellViewModel(
            id: product.id,
            title: product.name,
            price: product.formattedPrice,
            imageURL: product.imageURL.flatMap { URL(string: $0) },
            categoryBadge: product.category.uppercased()
        )
    }
}

extension ProductListPresenter: FilterDelegate {
    func filterDidUpdate(_ filter: ProductFilter) {
        Task { @MainActor in
            do {
                products = try await interactor.fetchProducts(category: filter.category)
                let viewModels = products.map { mapToViewModel($0) }
                view?.showProducts(viewModels)
            } catch {
                view?.showError(error.localizedDescription)
            }
        }
    }
}

struct ProductFilter {
    var category: String?
    var priceRange: ClosedRange<Double>?
    var sortBy: SortOption
}

protocol FilterDelegate: AnyObject {
    func filterDidUpdate(_ filter: ProductFilter)
}
```

---

## 12. Clean Architecture

Clean Architecture แบ่งแอปออกเป็น Layers ที่มี Dependencies ไหลเข้าสู่ศูนย์กลาง

```swift
// Domain Layer (ศูนย์กลาง - ไม่ขึ้นกับ Framework)
// ─────────────────────────────────────────────

// Domain Entity
struct Article {
    let id: UUID
    let title: String
    let content: String
    let author: Author
    let publishedAt: Date
    let tags: [String]
    
    struct Author {
        let id: UUID
        let name: String
        let bio: String?
    }
}

// Domain Repository Interface
protocol ArticleRepositoryProtocol {
    func fetchArticles(page: Int) async throws -> PaginatedResult<Article>
    func fetchArticle(id: UUID) async throws -> Article
    func saveArticle(_ article: Article) async throws
    func searchArticles(query: String) async throws -> [Article]
}

// Domain Use Case
protocol FetchArticlesUseCaseProtocol {
    func execute(page: Int) async throws -> PaginatedResult<Article>
}

class FetchArticlesUseCase: FetchArticlesUseCaseProtocol {
    private let repository: ArticleRepositoryProtocol
    private let cacheService: CacheServiceProtocol
    
    init(repository: ArticleRepositoryProtocol, cacheService: CacheServiceProtocol) {
        self.repository = repository
        self.cacheService = cacheService
    }
    
    func execute(page: Int) async throws -> PaginatedResult<Article> {
        // Try cache first for first page
        if page == 1, let cached = await cacheService.get([Article].self, key: "articles_page_1") {
            return PaginatedResult(items: cached, currentPage: 1, hasNextPage: true)
        }
        
        let result = try await repository.fetchArticles(page: page)
        
        // Cache first page
        if page == 1 {
            await cacheService.set(result.items, key: "articles_page_1", expiry: 300)
        }
        
        return result
    }
}

struct PaginatedResult<T> {
    let items: [T]
    let currentPage: Int
    let hasNextPage: Bool
}

// Data Layer
// ─────────────────────────────────────────────

// Data Transfer Object (DTO)
struct ArticleDTO: Codable {
    let id: String
    let title: String
    let content: String
    let authorId: String
    let authorName: String
    let authorBio: String?
    let publishedAt: String
    let tags: [String]
    
    enum CodingKeys: String, CodingKey {
        case id, title, content, tags
        case authorId = "author_id"
        case authorName = "author_name"
        case authorBio = "author_bio"
        case publishedAt = "published_at"
    }
}

// Mapper
extension ArticleDTO {
    func toDomain() -> Article? {
        guard let id = UUID(uuidString: id),
              let authorId = UUID(uuidString: authorId) else { return nil }
        
        let dateFormatter = ISO8601DateFormatter()
        guard let date = dateFormatter.date(from: publishedAt) else { return nil }
        
        return Article(
            id: id,
            title: title,
            content: content,
            author: Article.Author(id: authorId, name: authorName, bio: authorBio),
            publishedAt: date,
            tags: tags
        )
    }
}

// Repository Implementation
class ArticleRepository: ArticleRepositoryProtocol {
    private let apiClient: APIClientProtocol
    private let localStore: LocalStoreProtocol
    
    init(apiClient: APIClientProtocol, localStore: LocalStoreProtocol) {
        self.apiClient = apiClient
        self.localStore = localStore
    }
    
    func fetchArticles(page: Int) async throws -> PaginatedResult<Article> {
        let endpoint = ArticleEndpoint.list(page: page)
        let response: ArticlesResponseDTO = try await apiClient.request(endpoint)
        let articles = response.articles.compactMap { $0.toDomain() }
        return PaginatedResult(
            items: articles,
            currentPage: page,
            hasNextPage: response.hasNext
        )
    }
    
    func fetchArticle(id: UUID) async throws -> Article {
        let endpoint = ArticleEndpoint.detail(id: id.uuidString)
        let dto: ArticleDTO = try await apiClient.request(endpoint)
        guard let article = dto.toDomain() else {
            throw RepositoryError.mappingFailed
        }
        return article
    }
    
    func saveArticle(_ article: Article) async throws {
        try await localStore.save(article, key: "article_\(article.id)")
    }
    
    func searchArticles(query: String) async throws -> [Article] {
        let endpoint = ArticleEndpoint.search(query: query)
        let response: ArticlesResponseDTO = try await apiClient.request(endpoint)
        return response.articles.compactMap { $0.toDomain() }
    }
}

enum RepositoryError: Error {
    case mappingFailed
    case notFound
    case networkUnavailable
}
```

---

## 13. Repository Pattern

```swift
// Generic Repository Protocol
protocol Repository {
    associatedtype Entity
    associatedtype ID
    
    func findById(_ id: ID) async throws -> Entity?
    func findAll() async throws -> [Entity]
    func save(_ entity: Entity) async throws -> Entity
    func delete(_ id: ID) async throws
    func exists(_ id: ID) async throws -> Bool
}

// Specific Repository
protocol UserRepositoryProtocol {
    func findById(_ id: UUID) async throws -> User?
    func findByEmail(_ email: String) async throws -> User?
    func findAll(page: Int, limit: Int) async throws -> PagedResult<User>
    func save(_ user: User) async throws -> User
    func update(_ user: User) async throws -> User
    func delete(_ id: UUID) async throws
    func exists(_ id: UUID) async throws -> Bool
}

// Remote Repository Implementation
class RemoteUserRepository: UserRepositoryProtocol {
    private let apiClient: APIClientProtocol
    
    init(apiClient: APIClientProtocol) {
        self.apiClient = apiClient
    }
    
    func findById(_ id: UUID) async throws -> User? {
        do {
            let dto: UserDTO = try await apiClient.request(UserEndpoint.getById(id.uuidString))
            return dto.toDomain()
        } catch APIError.notFound {
            return nil
        }
    }
    
    func findByEmail(_ email: String) async throws -> User? {
        do {
            let dto: UserDTO = try await apiClient.request(UserEndpoint.getByEmail(email))
            return dto.toDomain()
        } catch APIError.notFound {
            return nil
        }
    }
    
    func findAll(page: Int, limit: Int) async throws -> PagedResult<User> {
        let response: PagedResponseDTO<UserDTO> = try await apiClient.request(
            UserEndpoint.list(page: page, limit: limit)
        )
        return PagedResult(
            items: response.data.compactMap { $0.toDomain() },
            currentPage: page,
            totalPages: response.totalPages,
            totalItems: response.totalCount
        )
    }
    
    func save(_ user: User) async throws -> User {
        let dto = UserDTO.fromDomain(user)
        let responseDTO: UserDTO = try await apiClient.request(
            UserEndpoint.create(dto)
        )
        guard let saved = responseDTO.toDomain() else {
            throw RepositoryError.mappingFailed
        }
        return saved
    }
    
    func update(_ user: User) async throws -> User {
        let dto = UserDTO.fromDomain(user)
        let responseDTO: UserDTO = try await apiClient.request(
            UserEndpoint.update(user.id.uuidString, dto)
        )
        guard let updated = responseDTO.toDomain() else {
            throw RepositoryError.mappingFailed
        }
        return updated
    }
    
    func delete(_ id: UUID) async throws {
        try await apiClient.request(UserEndpoint.delete(id.uuidString))
    }
    
    func exists(_ id: UUID) async throws -> Bool {
        return try await findById(id) != nil
    }
}

// Cached Repository (Decorator Pattern)
class CachedUserRepository: UserRepositoryProtocol {
    private let remote: UserRepositoryProtocol
    private let local: LocalUserRepositoryProtocol
    private let cachePolicy: CachePolicyProtocol
    
    init(
        remote: UserRepositoryProtocol,
        local: LocalUserRepositoryProtocol,
        cachePolicy: CachePolicyProtocol
    ) {
        self.remote = remote
        self.local = local
        self.cachePolicy = cachePolicy
    }
    
    func findById(_ id: UUID) async throws -> User? {
        // Check local cache first
        if let cached = try await local.findById(id),
           cachePolicy.isValid(for: cached.updatedAt) {
            return cached
        }
        
        // Fetch from remote
        if let user = try await remote.findById(id) {
            try await local.saveOrUpdate(user)
            return user
        }
        
        // Return stale cache if available
        return try await local.findById(id)
    }
    
    func findByEmail(_ email: String) async throws -> User? {
        return try await remote.findByEmail(email)
    }
    
    func findAll(page: Int, limit: Int) async throws -> PagedResult<User> {
        return try await remote.findAll(page: page, limit: limit)
    }
    
    func save(_ user: User) async throws -> User {
        let saved = try await remote.save(user)
        try await local.saveOrUpdate(saved)
        return saved
    }
    
    func update(_ user: User) async throws -> User {
        let updated = try await remote.update(user)
        try await local.saveOrUpdate(updated)
        return updated
    }
    
    func delete(_ id: UUID) async throws {
        try await remote.delete(id)
        try await local.delete(id)
    }
    
    func exists(_ id: UUID) async throws -> Bool {
        return try await findById(id) != nil
    }
}

struct PagedResult<T> {
    let items: [T]
    let currentPage: Int
    let totalPages: Int
    let totalItems: Int
    
    var hasNextPage: Bool { currentPage < totalPages }
    var hasPreviousPage: Bool { currentPage > 1 }
}
```

---

## 14. Service Layer

```swift
// Service Protocol
protocol AuthServiceProtocol {
    func login(email: String, password: String) async throws -> AuthToken
    func logout() async throws
    func refreshToken(_ token: AuthToken) async throws -> AuthToken
    func resetPassword(email: String) async throws
    var currentUser: User? { get }
    var isLoggedIn: Bool { get }
}

// AuthToken
struct AuthToken: Codable {
    let accessToken: String
    let refreshToken: String
    let expiresAt: Date
    
    var isExpired: Bool {
        return Date() >= expiresAt
    }
    
    var isExpiringSoon: Bool {
        let fiveMinutes = TimeInterval(300)
        return Date().addingTimeInterval(fiveMinutes) >= expiresAt
    }
}

// Auth Service Implementation
class AuthService: AuthServiceProtocol {
    private(set) var currentUser: User?
    private var authToken: AuthToken?
    
    var isLoggedIn: Bool { authToken != nil && !(authToken?.isExpired ?? true) }
    
    private let apiClient: APIClientProtocol
    private let keychain: KeychainProtocol
    private let userRepository: UserRepositoryProtocol
    
    init(
        apiClient: APIClientProtocol,
        keychain: KeychainProtocol,
        userRepository: UserRepositoryProtocol
    ) {
        self.apiClient = apiClient
        self.keychain = keychain
        self.userRepository = userRepository
        restoreSession()
    }
    
    func login(email: String, password: String) async throws -> AuthToken {
        let credentials = LoginCredentials(email: email, password: password)
        let response: AuthResponseDTO = try await apiClient.request(
            AuthEndpoint.login(credentials)
        )
        
        let token = AuthToken(
            accessToken: response.accessToken,
            refreshToken: response.refreshToken,
            expiresAt: Date().addingTimeInterval(response.expiresIn)
        )
        
        authToken = token
        try keychain.save(token, key: "auth_token")
        
        let user = response.user.toDomain()
        currentUser = user
        
        return token
    }
    
    func logout() async throws {
        try await apiClient.request(AuthEndpoint.logout)
        authToken = nil
        currentUser = nil
        keychain.delete(key: "auth_token")
    }
    
    func refreshToken(_ token: AuthToken) async throws -> AuthToken {
        let response: AuthResponseDTO = try await apiClient.request(
            AuthEndpoint.refreshToken(token.refreshToken)
        )
        
        let newToken = AuthToken(
            accessToken: response.accessToken,
            refreshToken: response.refreshToken,
            expiresAt: Date().addingTimeInterval(response.expiresIn)
        )
        
        authToken = newToken
        try keychain.save(newToken, key: "auth_token")
        
        return newToken
    }
    
    func resetPassword(email: String) async throws {
        try await apiClient.request(AuthEndpoint.resetPassword(email))
    }
    
    private func restoreSession() {
        authToken = keychain.get(AuthToken.self, key: "auth_token")
    }
}

// Notification Service
protocol NotificationServiceProtocol {
    func requestPermission() async -> Bool
    func scheduleLocalNotification(_ notification: LocalNotification) async throws
    func cancelNotification(id: String)
    func cancelAllNotifications()
}

struct LocalNotification {
    let id: String
    let title: String
    let body: String
    let trigger: NotificationTrigger
    let userInfo: [String: Any]
    
    enum NotificationTrigger {
        case timeInterval(TimeInterval, repeats: Bool)
        case calendar(DateComponents, repeats: Bool)
        case location(CLRegion, repeats: Bool)
    }
}
```

---

## 15. Coordinator Pattern

Coordinator แยก Navigation Logic ออกจาก ViewController

```swift
import UIKit

// Coordinator Protocol
protocol Coordinator: AnyObject {
    var childCoordinators: [Coordinator] { get set }
    var navigationController: UINavigationController { get }
    func start()
}

extension Coordinator {
    func addChild(_ coordinator: Coordinator) {
        childCoordinators.append(coordinator)
    }
    
    func removeChild(_ coordinator: Coordinator) {
        childCoordinators.removeAll { $0 === coordinator }
    }
}

// App Coordinator
class AppCoordinator: Coordinator {
    var childCoordinators: [Coordinator] = []
    let navigationController: UINavigationController
    private let authService: AuthServiceProtocol
    
    init(navigationController: UINavigationController, authService: AuthServiceProtocol) {
        self.navigationController = navigationController
        self.authService = authService
    }
    
    func start() {
        if authService.isLoggedIn {
            showMainFlow()
        } else {
            showAuthFlow()
        }
    }
    
    private func showAuthFlow() {
        let authCoordinator = AuthCoordinator(
            navigationController: navigationController
        )
        authCoordinator.delegate = self
        addChild(authCoordinator)
        authCoordinator.start()
    }
    
    private func showMainFlow() {
        let mainCoordinator = MainCoordinator(
            navigationController: navigationController
        )
        mainCoordinator.delegate = self
        addChild(mainCoordinator)
        mainCoordinator.start()
    }
}

extension AppCoordinator: AuthCoordinatorDelegate {
    func authCoordinatorDidFinish(_ coordinator: AuthCoordinator) {
        removeChild(coordinator)
        showMainFlow()
    }
}

extension AppCoordinator: MainCoordinatorDelegate {
    func mainCoordinatorDidLogout(_ coordinator: MainCoordinator) {
        removeChild(coordinator)
        navigationController.viewControllers = []
        showAuthFlow()
    }
}

// Auth Coordinator
protocol AuthCoordinatorDelegate: AnyObject {
    func authCoordinatorDidFinish(_ coordinator: AuthCoordinator)
}

class AuthCoordinator: Coordinator {
    var childCoordinators: [Coordinator] = []
    let navigationController: UINavigationController
    weak var delegate: AuthCoordinatorDelegate?
    
    private let container: DependencyContainer
    
    init(navigationController: UINavigationController) {
        self.navigationController = navigationController
        self.container = DependencyContainer.shared
    }
    
    func start() {
        let viewModel = LoginViewModel(authService: container.resolve())
        viewModel.onLoginSuccess = { [weak self] in
            guard let self = self else { return }
            self.delegate?.authCoordinatorDidFinish(self)
        }
        
        let loginVC = LoginViewController(viewModel: viewModel)
        navigationController.setViewControllers([loginVC], animated: true)
    }
    
    func showRegistration() {
        let viewModel = RegistrationViewModel(authService: container.resolve())
        let regVC = RegistrationViewController(viewModel: viewModel)
        navigationController.pushViewController(regVC, animated: true)
    }
    
    func showForgotPassword() {
        let viewModel = ForgotPasswordViewModel(authService: container.resolve())
        let forgotVC = ForgotPasswordViewController(viewModel: viewModel)
        navigationController.pushViewController(forgotVC, animated: true)
    }
}

// Main Coordinator
protocol MainCoordinatorDelegate: AnyObject {
    func mainCoordinatorDidLogout(_ coordinator: MainCoordinator)
}

class MainCoordinator: Coordinator {
    var childCoordinators: [Coordinator] = []
    let navigationController: UINavigationController
    weak var delegate: MainCoordinatorDelegate?
    
    func start() {
        let tabBarVC = UITabBarController()
        
        let homeNav = UINavigationController()
        let homeCoordinator = HomeCoordinator(navigationController: homeNav)
        addChild(homeCoordinator)
        homeCoordinator.start()
        homeNav.tabBarItem = UITabBarItem(title: "Home", image: UIImage(systemName: "house"), tag: 0)
        
        let profileNav = UINavigationController()
        let profileCoordinator = ProfileCoordinator(navigationController: profileNav)
        addChild(profileCoordinator)
        profileCoordinator.start()
        profileNav.tabBarItem = UITabBarItem(title: "Profile", image: UIImage(systemName: "person"), tag: 1)
        
        tabBarVC.viewControllers = [homeNav, profileNav]
        navigationController.setViewControllers([tabBarVC], animated: true)
    }
}
```

---

## 16. Dependency Injection

```swift
// ปัญหาเมื่อไม่มี DI
class BadViewModel {
    // Hard-coded dependencies - ทดสอบยาก!
    private let service = UserService(
        apiClient: APIClient(
            baseURL: URL(string: "https://api.example.com")!,
            session: URLSession.shared
        )
    )
}

// ดีกว่า - ใช้ DI
class GoodViewModel {
    private let service: UserServiceProtocol
    
    // Constructor Injection
    init(service: UserServiceProtocol) {
        self.service = service
    }
}

// DI Container ขั้นพื้นฐาน
class DependencyContainer {
    static let shared = DependencyContainer()
    private var factories: [String: () -> Any] = [:]
    private var singletons: [String: Any] = [:]
    
    private init() {
        registerDependencies()
    }
    
    // Register factory
    func register<T>(_ type: T.Type, factory: @escaping () -> T) {
        let key = String(describing: type)
        factories[key] = factory
    }
    
    // Register singleton
    func registerSingleton<T>(_ type: T.Type, factory: @escaping () -> T) {
        let key = String(describing: type)
        factories[key] = { [weak self] in
            if let existing = self?.singletons[key] as? T {
                return existing
            }
            let instance = factory()
            self?.singletons[key] = instance
            return instance
        }
    }
    
    // Resolve
    func resolve<T>(_ type: T.Type = T.self) -> T {
        let key = String(describing: type)
        guard let factory = factories[key],
              let instance = factory() as? T else {
            fatalError("No factory registered for type \(key)")
        }
        return instance
    }
    
    private func registerDependencies() {
        // API Client
        registerSingleton(APIClientProtocol.self) {
            APIClient(baseURL: URL(string: "https://api.github.com")!)
        }
        
        // Services
        registerSingleton(UserServiceProtocol.self) {
            UserService(apiClient: self.resolve())
        }
        
        // Repositories
        register(UserRepositoryProtocol.self) {
            UserRepository(apiClient: self.resolve())
        }
    }
}
```

---

## 17. Inversion of Control

```swift
// ตัวอย่าง Inversion of Control

// แบบที่ไม่ดี - High-level module ขึ้นกับ Low-level
class UserNotificationSender {
    private let emailService = EmailService()  // ผูกติดกับ EmailService
    private let smsService = SMSService()      // ผูกติดกับ SMSService
    
    func sendWelcome(to user: User) {
        emailService.send("Welcome!", to: user.email)
    }
}

// แบบที่ดี - ใช้ Abstraction
protocol NotificationChannel {
    func send(_ message: String, to recipient: String) async throws
}

class EmailNotificationChannel: NotificationChannel {
    func send(_ message: String, to recipient: String) async throws {
        // Send via email
    }
}

class SMSNotificationChannel: NotificationChannel {
    func send(_ message: String, to recipient: String) async throws {
        // Send via SMS
    }
}

class PushNotificationChannel: NotificationChannel {
    func send(_ message: String, to recipient: String) async throws {
        // Send via push notification
    }
}

class UserNotificationService {
    private let channels: [NotificationChannel]
    
    init(channels: [NotificationChannel]) {
        self.channels = channels
    }
    
    func sendWelcome(to user: User) async throws {
        let message = "Welcome to our app, \(user.name)!"
        for channel in channels {
            try await channel.send(message, to: user.email)
        }
    }
}

// ใช้งาน - เลือก channels ได้
let service = UserNotificationService(channels: [
    EmailNotificationChannel(),
    PushNotificationChannel()
])
```

---

## 18. Protocol-based DI

```swift
// Protocol-based Dependencies

// Protocols สำหรับ Dependencies
protocol DateProviding {
    var now: Date { get }
}

protocol RandomProviding {
    func randomInt(in range: Range<Int>) -> Int
}

protocol NetworkChecking {
    var isConnected: Bool { get }
}

// Production implementations
struct SystemDateProvider: DateProviding {
    var now: Date { Date() }
}

struct SystemRandomProvider: RandomProviding {
    func randomInt(in range: Range<Int>) -> Int {
        Int.random(in: range)
    }
}

// Test implementations
struct FixedDateProvider: DateProviding {
    let now: Date
    
    init(date: Date = Date(timeIntervalSince1970: 1000000)) {
        self.now = date
    }
}

struct DeterministicRandomProvider: RandomProviding {
    private let values: [Int]
    private var index = 0
    
    init(values: [Int]) {
        self.values = values
    }
    
    mutating func randomInt(in range: Range<Int>) -> Int {
        defer { index = (index + 1) % values.count }
        return values[index]
    }
}

// ViewModel ที่ใช้ Protocol-based DI
class GameViewModel {
    private let dateProvider: DateProviding
    private let randomProvider: RandomProviding
    
    private(set) var score = 0
    private(set) var gameStartTime: Date
    
    init(
        dateProvider: DateProviding = SystemDateProvider(),
        randomProvider: RandomProviding = SystemRandomProvider()
    ) {
        self.dateProvider = dateProvider
        self.randomProvider = randomProvider
        self.gameStartTime = dateProvider.now
    }
    
    func generateQuestion() -> Int {
        randomProvider.randomInt(in: 1..<100)
    }
    
    var elapsedTime: TimeInterval {
        dateProvider.now.timeIntervalSince(gameStartTime)
    }
}

// Testing
class GameViewModelTests: XCTestCase {
    func testElapsedTime() {
        let startDate = Date(timeIntervalSince1970: 1000000)
        let dateProvider = FixedDateProvider(date: startDate)
        let viewModel = GameViewModel(dateProvider: dateProvider)
        
        // ไม่มี race condition เพราะ date เป็น fixed value
        XCTAssertEqual(viewModel.elapsedTime, 0, accuracy: 0.001)
    }
}
```

---

## 19. Swinject Framework

```swift
// การใช้ Swinject สำหรับ DI Container ที่ซับซ้อน
// (ต้องเพิ่ม Swinject ใน Package.swift)

import Swinject

// การ Setup Container
class AppAssembler {
    static let shared = AppAssembler()
    let container: Container
    let resolver: Resolver
    
    private init() {
        container = Container()
        
        // Register Network Layer
        container.register(URLSession.self) { _ in
            let config = URLSessionConfiguration.default
            config.timeoutIntervalForRequest = 30
            return URLSession(configuration: config)
        }.inObjectScope(.container) // Singleton
        
        container.register(APIClientProtocol.self) { r in
            APIClient(
                baseURL: URL(string: "https://api.github.com")!,
                session: r.resolve(URLSession.self)!
            )
        }.inObjectScope(.container)
        
        // Register Repositories
        container.register(UserRepositoryProtocol.self) { r in
            UserRepository(apiClient: r.resolve(APIClientProtocol.self)!)
        }
        
        // Register Services
        container.register(AuthServiceProtocol.self) { r in
            AuthService(
                apiClient: r.resolve(APIClientProtocol.self)!,
                keychain: r.resolve(KeychainProtocol.self)!,
                userRepository: r.resolve(UserRepositoryProtocol.self)!
            )
        }.inObjectScope(.container)
        
        // Register ViewModels
        container.register(LoginViewModel.self) { r in
            LoginViewModel(authService: r.resolve(AuthServiceProtocol.self)!)
        }
        
        resolver = container
    }
}

// การใช้งาน
let viewModel = AppAssembler.shared.resolver.resolve(LoginViewModel.self)!
```

---

## 20. Fat ViewModels Problem

```swift
// Fat ViewModel - ปัญหา
class FatViewModel: ObservableObject {
    // มี 30+ @Published properties
    @Published var users: [User] = []
    @Published var posts: [Post] = []
    @Published var messages: [Message] = []
    // ...ต่อไปอีกเยอะ
    
    // 20+ methods ที่ทำหลายอย่าง
    func loadUsers() { /* ... */ }
    func loadPosts() { /* ... */ }
    func sendMessage() { /* ... */ }
    func validateEmail() { /* ... */ }
    func formatDate() { /* ... */ }
    // ...ต่อไปอีกเยอะ
}

// แก้ไขด้วยการแบ่ง Responsibilities
// ─────────────────────────────────────────────

// Use Case แทนที่จะใส่ logic ใน ViewModel
protocol LoadUsersUseCaseProtocol {
    func execute() async throws -> [User]
}

class LoadUsersUseCase: LoadUsersUseCaseProtocol {
    private let repository: UserRepositoryProtocol
    
    init(repository: UserRepositoryProtocol) {
        self.repository = repository
    }
    
    func execute() async throws -> [User] {
        // Business logic อยู่ที่นี่
        let users = try await repository.findAll(page: 1, limit: 50)
        return users.items.sorted { $0.name < $1.name }
    }
}

// Leaner ViewModel ที่ Delegate ไปยัง Use Cases
class UserListViewModel: ObservableObject {
    @Published var users: [User] = []
    @Published var isLoading = false
    @Published var error: String?
    
    private let loadUsersUseCase: LoadUsersUseCaseProtocol
    
    init(loadUsersUseCase: LoadUsersUseCaseProtocol) {
        self.loadUsersUseCase = loadUsersUseCase
    }
    
    func onAppear() {
        Task { @MainActor in
            isLoading = true
            do {
                users = try await loadUsersUseCase.execute()
            } catch {
                self.error = error.localizedDescription
            }
            isLoading = false
        }
    }
}
```

---

## 21. UseCase Pattern

```swift
// UseCase เป็น Application Business Rules

// Base Protocol
protocol UseCase {
    associatedtype Input
    associatedtype Output
    
    func execute(input: Input) async throws -> Output
}

// Concrete Use Cases
struct SearchRepositoriesInput {
    let query: String
    let language: String?
    let page: Int
    let perPage: Int
}

struct SearchRepositoriesOutput {
    let repositories: [Repository]
    let totalCount: Int
    let hasNextPage: Bool
}

class SearchRepositoriesUseCase: UseCase {
    typealias Input = SearchRepositoriesInput
    typealias Output = SearchRepositoriesOutput
    
    private let repository: RepositoryRepositoryProtocol
    private let validator: SearchQueryValidatorProtocol
    
    init(
        repository: RepositoryRepositoryProtocol,
        validator: SearchQueryValidatorProtocol
    ) {
        self.repository = repository
        self.validator = validator
    }
    
    func execute(input: SearchRepositoriesInput) async throws -> SearchRepositoriesOutput {
        // Validate
        try validator.validate(query: input.query)
        
        // Execute
        let result = try await repository.search(
            query: input.query,
            language: input.language,
            page: input.page,
            perPage: input.perPage
        )
        
        return SearchRepositoriesOutput(
            repositories: result.items,
            totalCount: result.totalCount,
            hasNextPage: result.hasNextPage
        )
    }
}

// ViewModel ใช้ Use Case
class GitHubSearchViewModel: ObservableObject {
    @Published var repositories: [Repository] = []
    @Published var isLoading = false
    @Published var errorMessage: String?
    @Published var searchQuery = ""
    
    private let searchUseCase: SearchRepositoriesUseCase
    private var currentPage = 1
    
    init(searchUseCase: SearchRepositoriesUseCase) {
        self.searchUseCase = searchUseCase
    }
    
    func search() {
        Task { @MainActor in
            isLoading = true
            errorMessage = nil
            
            let input = SearchRepositoriesInput(
                query: searchQuery,
                language: nil,
                page: 1,
                perPage: 20
            )
            
            do {
                let output = try await searchUseCase.execute(input: input)
                repositories = output.repositories
            } catch {
                errorMessage = error.localizedDescription
            }
            
            isLoading = false
        }
    }
}
```

---

## 22. แบบฝึกหัด

### แบบฝึกหัดที่ 1: สร้าง Weather App ด้วย MVVM

**โจทย์**: สร้าง Weather App ที่แสดงสภาพอากาศตาม Location โดยใช้ MVVM Architecture

```swift
// Solution

// Model
struct WeatherData {
    let cityName: String
    let temperature: Double
    let feelsLike: Double
    let humidity: Int
    let windSpeed: Double
    let condition: WeatherCondition
    let icon: String
    let forecast: [ForecastDay]
    
    enum WeatherCondition: String {
        case sunny = "Clear"
        case cloudy = "Clouds"
        case rainy = "Rain"
        case snowy = "Snow"
        case stormy = "Thunderstorm"
        case foggy = "Mist"
        
        var emoji: String {
            switch self {
            case .sunny: return "☀️"
            case .cloudy: return "☁️"
            case .rainy: return "🌧️"
            case .snowy: return "❄️"
            case .stormy: return "⛈️"
            case .foggy: return "🌫️"
            }
        }
    }
    
    struct ForecastDay {
        let date: Date
        let maxTemp: Double
        let minTemp: Double
        let condition: WeatherCondition
    }
}

// Service Protocol
protocol WeatherServiceProtocol {
    func fetchWeather(for city: String) async throws -> WeatherData
    func fetchWeatherByCoordinate(lat: Double, lon: Double) async throws -> WeatherData
}

// ViewModel
@MainActor
class WeatherViewModel: ObservableObject {
    @Published var weather: WeatherData?
    @Published var isLoading = false
    @Published var errorMessage: String?
    @Published var cityName = "Bangkok"
    
    var temperatureDisplay: String {
        guard let temp = weather?.temperature else { return "--°" }
        return String(format: "%.0f°C", temp)
    }
    
    var conditionDisplay: String {
        guard let weather = weather else { return "--" }
        return "\(weather.condition.emoji) \(weather.condition.rawValue)"
    }
    
    var humidityDisplay: String {
        guard let humidity = weather?.humidity else { return "--%" }
        return "\(humidity)%"
    }
    
    var windDisplay: String {
        guard let wind = weather?.windSpeed else { return "-- km/h" }
        return String(format: "%.1f km/h", wind)
    }
    
    private let weatherService: WeatherServiceProtocol
    
    init(weatherService: WeatherServiceProtocol) {
        self.weatherService = weatherService
    }
    
    func loadWeather() async {
        guard !cityName.isEmpty else { return }
        
        isLoading = true
        errorMessage = nil
        
        do {
            weather = try await weatherService.fetchWeather(for: cityName)
        } catch {
            errorMessage = "Could not load weather for \(cityName)"
        }
        
        isLoading = false
    }
    
    func refresh() async {
        await loadWeather()
    }
}

// SwiftUI View
struct WeatherView: View {
    @StateObject var viewModel: WeatherViewModel
    
    var body: some View {
        ZStack {
            // Background gradient
            LinearGradient(
                colors: [.blue.opacity(0.6), .blue.opacity(0.2)],
                startPoint: .top,
                endPoint: .bottom
            )
            .ignoresSafeArea()
            
            ScrollView {
                VStack(spacing: 20) {
                    // Search bar
                    HStack {
                        TextField("City name...", text: $viewModel.cityName)
                            .textFieldStyle(.roundedBorder)
                            .onSubmit {
                                Task { await viewModel.loadWeather() }
                            }
                        
                        Button {
                            Task { await viewModel.loadWeather() }
                        } label: {
                            Image(systemName: "magnifyingglass")
                                .foregroundColor(.white)
                        }
                    }
                    .padding(.horizontal)
                    
                    if viewModel.isLoading {
                        ProgressView()
                            .tint(.white)
                            .scaleEffect(1.5)
                    } else if let errorMessage = viewModel.errorMessage {
                        Text(errorMessage)
                            .foregroundColor(.red)
                            .padding()
                            .background(.white.opacity(0.2))
                            .cornerRadius(10)
                    } else if let weather = viewModel.weather {
                        // Main weather display
                        VStack(spacing: 8) {
                            Text(weather.cityName)
                                .font(.title)
                                .fontWeight(.medium)
                                .foregroundColor(.white)
                            
                            Text(viewModel.temperatureDisplay)
                                .font(.system(size: 80, weight: .thin))
                                .foregroundColor(.white)
                            
                            Text(viewModel.conditionDisplay)
                                .font(.title2)
                                .foregroundColor(.white.opacity(0.9))
                            
                            Text("Feels like \(String(format: "%.0f°C", weather.feelsLike))")
                                .foregroundColor(.white.opacity(0.7))
                        }
                        
                        // Stats row
                        HStack(spacing: 40) {
                            WeatherStatView(icon: "humidity", value: viewModel.humidityDisplay, label: "Humidity")
                            WeatherStatView(icon: "wind", value: viewModel.windDisplay, label: "Wind")
                        }
                        .padding()
                        .background(.white.opacity(0.2))
                        .cornerRadius(15)
                        
                        // Forecast
                        VStack(alignment: .leading) {
                            Text("5-Day Forecast")
                                .font(.headline)
                                .foregroundColor(.white)
                                .padding(.horizontal)
                            
                            ForEach(weather.forecast.indices, id: \.self) { i in
                                let day = weather.forecast[i]
                                ForecastRowView(day: day)
                            }
                        }
                        .padding()
                        .background(.white.opacity(0.2))
                        .cornerRadius(15)
                    }
                }
                .padding()
            }
        }
        .task {
            await viewModel.loadWeather()
        }
        .refreshable {
            await viewModel.refresh()
        }
    }
}

struct WeatherStatView: View {
    let icon: String
    let value: String
    let label: String
    
    var body: some View {
        VStack {
            Image(systemName: icon)
                .font(.title2)
                .foregroundColor(.white)
            Text(value)
                .font(.title3)
                .fontWeight(.semibold)
                .foregroundColor(.white)
            Text(label)
                .font(.caption)
                .foregroundColor(.white.opacity(0.7))
        }
    }
}

struct ForecastRowView: View {
    let day: WeatherData.ForecastDay
    
    var body: some View {
        HStack {
            Text(day.date, format: .dateTime.weekday(.wide))
                .foregroundColor(.white)
                .frame(width: 100, alignment: .leading)
            
            Text(day.condition.emoji)
            
            Spacer()
            
            Text(String(format: "%.0f°", day.minTemp))
                .foregroundColor(.white.opacity(0.7))
            
            Text(String(format: "%.0f°", day.maxTemp))
                .foregroundColor(.white)
                .fontWeight(.semibold)
        }
        .padding(.horizontal)
    }
}
```

---

## 23. การสร้าง MVVM App สมบูรณ์

### GitHub Repositories Viewer

```swift
// Complete MVVM GitHub App

// MARK: - Models
struct GitHubRepository: Identifiable, Codable {
    let id: Int
    let name: String
    let fullName: String
    let description: String?
    let starCount: Int
    let forkCount: Int
    let language: String?
    let htmlURL: String
    let owner: Owner
    
    struct Owner: Codable {
        let login: String
        let avatarURL: String
        
        enum CodingKeys: String, CodingKey {
            case login
            case avatarURL = "avatar_url"
        }
    }
    
    enum CodingKeys: String, CodingKey {
        case id, name, description, language
        case fullName = "full_name"
        case starCount = "stargazers_count"
        case forkCount = "forks_count"
        case htmlURL = "html_url"
        case owner
    }
}

struct SearchResponse: Codable {
    let totalCount: Int
    let items: [GitHubRepository]
    
    enum CodingKeys: String, CodingKey {
        case totalCount = "total_count"
        case items
    }
}

// MARK: - Repository Layer
protocol GitHubRepositoryProtocol {
    func searchRepositories(query: String, page: Int) async throws -> SearchResponse
    func getRepository(owner: String, name: String) async throws -> GitHubRepository
}

class GitHubAPIRepository: GitHubRepositoryProtocol {
    private let baseURL = "https://api.github.com"
    private let session: URLSession
    private let decoder: JSONDecoder
    
    init(session: URLSession = .shared) {
        self.session = session
        self.decoder = JSONDecoder()
        // GitHub API ใช้ ISO 8601
        self.decoder.dateDecodingStrategy = .iso8601
    }
    
    func searchRepositories(query: String, page: Int) async throws -> SearchResponse {
        var components = URLComponents(string: "\(baseURL)/search/repositories")!
        components.queryItems = [
            URLQueryItem(name: "q", value: query),
            URLQueryItem(name: "sort", value: "stars"),
            URLQueryItem(name: "order", value: "desc"),
            URLQueryItem(name: "per_page", value: "20"),
            URLQueryItem(name: "page", value: "\(page)")
        ]
        
        var request = URLRequest(url: components.url!)
        request.setValue("application/vnd.github.v3+json", forHTTPHeaderField: "Accept")
        
        let (data, response) = try await session.data(for: request)
        
        guard let httpResponse = response as? HTTPURLResponse else {
            throw GitHubError.invalidResponse
        }
        
        switch httpResponse.statusCode {
        case 200:
            return try decoder.decode(SearchResponse.self, from: data)
        case 403:
            throw GitHubError.rateLimitExceeded
        case 422:
            throw GitHubError.invalidQuery
        default:
            throw GitHubError.httpError(httpResponse.statusCode)
        }
    }
    
    func getRepository(owner: String, name: String) async throws -> GitHubRepository {
        let url = URL(string: "\(baseURL)/repos/\(owner)/\(name)")!
        var request = URLRequest(url: url)
        request.setValue("application/vnd.github.v3+json", forHTTPHeaderField: "Accept")
        
        let (data, _) = try await session.data(for: request)
        return try decoder.decode(GitHubRepository.self, from: data)
    }
}

enum GitHubError: LocalizedError {
    case invalidResponse
    case rateLimitExceeded
    case invalidQuery
    case httpError(Int)
    case notFound
    
    var errorDescription: String? {
        switch self {
        case .invalidResponse: return "Invalid response from server"
        case .rateLimitExceeded: return "Rate limit exceeded. Please wait a moment."
        case .invalidQuery: return "Invalid search query"
        case .httpError(let code): return "HTTP Error: \(code)"
        case .notFound: return "Repository not found"
        }
    }
}

// MARK: - Use Cases
protocol SearchRepositoriesUseCaseProtocol {
    func execute(query: String, page: Int) async throws -> SearchResponse
}

class SearchRepositoriesUseCase: SearchRepositoriesUseCaseProtocol {
    private let repository: GitHubRepositoryProtocol
    
    init(repository: GitHubRepositoryProtocol) {
        self.repository = repository
    }
    
    func execute(query: String, page: Int) async throws -> SearchResponse {
        guard query.count >= 2 else {
            throw ValidationError.queryTooShort
        }
        return try await repository.searchRepositories(query: query, page: page)
    }
}

enum ValidationError: LocalizedError {
    case queryTooShort
    
    var errorDescription: String? {
        switch self {
        case .queryTooShort: return "Search query must be at least 2 characters"
        }
    }
}

// MARK: - ViewModel
@MainActor
class GitHubSearchViewModel: ObservableObject {
    // Published State
    @Published var searchQuery = ""
    @Published var repositories: [GitHubRepository] = []
    @Published var isLoading = false
    @Published var isLoadingMore = false
    @Published var errorMessage: String?
    @Published var hasSearched = false
    
    // Pagination
    private var currentPage = 1
    private var totalCount = 0
    var hasMoreResults: Bool { repositories.count < totalCount }
    
    var resultSummary: String {
        if !hasSearched { return "" }
        if repositories.isEmpty { return "No results found" }
        return "\(totalCount) repositories found"
    }
    
    private let searchUseCase: SearchRepositoriesUseCaseProtocol
    private var searchTask: Task<Void, Never>?
    
    init(searchUseCase: SearchRepositoriesUseCaseProtocol) {
        self.searchUseCase = searchUseCase
    }
    
    func search() {
        guard !searchQuery.trimmingCharacters(in: .whitespaces).isEmpty else { return }
        
        searchTask?.cancel()
        currentPage = 1
        repositories = []
        hasSearched = true
        
        searchTask = Task {
            await performSearch(page: 1)
        }
    }
    
    func loadMore() {
        guard hasMoreResults && !isLoadingMore else { return }
        currentPage += 1
        Task {
            await performSearch(page: currentPage)
        }
    }
    
    private func performSearch(page: Int) async {
        if page == 1 {
            isLoading = true
        } else {
            isLoadingMore = true
        }
        errorMessage = nil
        
        do {
            let response = try await searchUseCase.execute(query: searchQuery, page: page)
            
            if page == 1 {
                repositories = response.items
            } else {
                repositories.append(contentsOf: response.items)
            }
            totalCount = response.totalCount
        } catch is CancellationError {
            // ถูก cancel - ok
        } catch {
            if page == 1 {
                errorMessage = error.localizedDescription
            }
        }
        
        isLoading = false
        isLoadingMore = false
    }
}

// MARK: - Views
struct GitHubSearchView: View {
    @StateObject private var viewModel: GitHubSearchViewModel
    
    init() {
        let repository = GitHubAPIRepository()
        let useCase = SearchRepositoriesUseCase(repository: repository)
        _viewModel = StateObject(wrappedValue: GitHubSearchViewModel(searchUseCase: useCase))
    }
    
    var body: some View {
        NavigationView {
            VStack(spacing: 0) {
                // Search Bar
                HStack {
                    Image(systemName: "magnifyingglass")
                        .foregroundColor(.secondary)
                    
                    TextField("Search repositories...", text: $viewModel.searchQuery)
                        .autocorrectionDisabled()
                        .autocapitalization(.none)
                        .onSubmit { viewModel.search() }
                    
                    if !viewModel.searchQuery.isEmpty {
                        Button {
                            viewModel.searchQuery = ""
                        } label: {
                            Image(systemName: "xmark.circle.fill")
                                .foregroundColor(.secondary)
                        }
                    }
                }
                .padding(10)
                .background(Color(.systemGray6))
                .cornerRadius(10)
                .padding()
                
                // Results summary
                if viewModel.hasSearched {
                    Text(viewModel.resultSummary)
                        .font(.caption)
                        .foregroundColor(.secondary)
                        .frame(maxWidth: .infinity, alignment: .leading)
                        .padding(.horizontal)
                }
                
                // Content
                ZStack {
                    if viewModel.isLoading {
                        ProgressView("Searching...")
                            .frame(maxWidth: .infinity, maxHeight: .infinity)
                    } else if let error = viewModel.errorMessage {
                        ErrorView(message: error) {
                            viewModel.search()
                        }
                    } else if viewModel.repositories.isEmpty && viewModel.hasSearched {
                        EmptySearchView(query: viewModel.searchQuery)
                    } else if !viewModel.hasSearched {
                        SearchPromptView()
                    } else {
                        RepositoryListView(viewModel: viewModel)
                    }
                }
            }
            .navigationTitle("GitHub Search")
            .navigationBarTitleDisplayMode(.inline)
        }
    }
}

struct RepositoryListView: View {
    @ObservedObject var viewModel: GitHubSearchViewModel
    
    var body: some View {
        List {
            ForEach(viewModel.repositories) { repo in
                NavigationLink {
                    RepositoryDetailView(repository: repo)
                } label: {
                    RepositoryRowView(repository: repo)
                }
                .onAppear {
                    // Load more when near end
                    if repo.id == viewModel.repositories.last?.id {
                        viewModel.loadMore()
                    }
                }
            }
            
            if viewModel.isLoadingMore {
                HStack {
                    Spacer()
                    ProgressView()
                    Spacer()
                }
                .padding()
                .listRowSeparator(.hidden)
            }
        }
        .listStyle(.plain)
    }
}

struct RepositoryRowView: View {
    let repository: GitHubRepository
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            HStack {
                AsyncImage(url: URL(string: repository.owner.avatarURL)) { image in
                    image.resizable().scaledToFill()
                } placeholder: {
                    Color.gray
                }
                .frame(width: 24, height: 24)
                .clipShape(Circle())
                
                Text(repository.owner.login)
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
            
            Text(repository.name)
                .font(.headline)
                .foregroundColor(.primary)
            
            if let description = repository.description {
                Text(description)
                    .font(.caption)
                    .foregroundColor(.secondary)
                    .lineLimit(2)
            }
            
            HStack(spacing: 16) {
                if let language = repository.language {
                    Label(language, systemImage: "circle.fill")
                        .font(.caption)
                        .foregroundColor(.secondary)
                }
                
                Label("\(repository.starCount)", systemImage: "star")
                    .font(.caption)
                    .foregroundColor(.secondary)
                
                Label("\(repository.forkCount)", systemImage: "tuningfork")
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
        }
        .padding(.vertical, 4)
    }
}

struct RepositoryDetailView: View {
    let repository: GitHubRepository
    
    var body: some View {
        ScrollView {
            VStack(alignment: .leading, spacing: 20) {
                // Header
                HStack {
                    AsyncImage(url: URL(string: repository.owner.avatarURL)) { image in
                        image.resizable().scaledToFill()
                    } placeholder: {
                        Color.gray
                    }
                    .frame(width: 60, height: 60)
                    .clipShape(Circle())
                    
                    VStack(alignment: .leading) {
                        Text(repository.name)
                            .font(.title2)
                            .fontWeight(.bold)
                        Text(repository.owner.login)
                            .foregroundColor(.secondary)
                    }
                }
                .padding(.horizontal)
                
                if let desc = repository.description {
                    Text(desc)
                        .padding(.horizontal)
                }
                
                // Stats
                HStack(spacing: 30) {
                    StatView(icon: "star.fill", value: "\(repository.starCount)", label: "Stars")
                    StatView(icon: "tuningfork", value: "\(repository.forkCount)", label: "Forks")
                    if let lang = repository.language {
                        StatView(icon: "chevron.left.forwardslash.chevron.right", value: lang, label: "Language")
                    }
                }
                .padding()
                .frame(maxWidth: .infinity)
                .background(Color(.systemGray6))
                .cornerRadius(12)
                .padding(.horizontal)
                
                // Open in Safari
                Link(destination: URL(string: repository.htmlURL)!) {
                    Label("Open on GitHub", systemImage: "safari")
                        .frame(maxWidth: .infinity)
                        .padding()
                        .background(Color.black)
                        .foregroundColor(.white)
                        .cornerRadius(12)
                }
                .padding(.horizontal)
            }
            .padding(.vertical)
        }
        .navigationTitle(repository.name)
        .navigationBarTitleDisplayMode(.inline)
    }
}

struct StatView: View {
    let icon: String
    let value: String
    let label: String
    
    var body: some View {
        VStack {
            Image(systemName: icon)
                .font(.title3)
            Text(value)
                .font(.headline)
            Text(label)
                .font(.caption)
                .foregroundColor(.secondary)
        }
    }
}

struct ErrorView: View {
    let message: String
    let retry: () -> Void
    
    var body: some View {
        VStack(spacing: 16) {
            Image(systemName: "exclamationmark.triangle")
                .font(.largeTitle)
                .foregroundColor(.orange)
            Text(message)
                .multilineTextAlignment(.center)
                .foregroundColor(.secondary)
            Button("Try Again", action: retry)
                .buttonStyle(.bordered)
        }
        .padding()
        .frame(maxWidth: .infinity, maxHeight: .infinity)
    }
}

struct EmptySearchView: View {
    let query: String
    
    var body: some View {
        VStack(spacing: 16) {
            Image(systemName: "magnifyingglass")
                .font(.largeTitle)
                .foregroundColor(.secondary)
            Text("No results for \"\(query)\"")
                .foregroundColor(.secondary)
        }
        .frame(maxWidth: .infinity, maxHeight: .infinity)
    }
}

struct SearchPromptView: View {
    var body: some View {
        VStack(spacing: 16) {
            Image(systemName: "magnifyingglass")
                .font(.system(size: 60))
                .foregroundColor(.secondary)
            Text("Search GitHub Repositories")
                .font(.title2)
                .fontWeight(.medium)
            Text("Enter keywords to find repositories")
                .foregroundColor(.secondary)
        }
        .frame(maxWidth: .infinity, maxHeight: .infinity)
    }
}

// MARK: - App Entry Point
@main
struct GitHubApp: App {
    var body: some Scene {
        WindowGroup {
            GitHubSearchView()
        }
    }
}
```

---

## 24. สรุป

### เปรียบเทียบ Architectures

| Architecture | Complexity | Testability | Team Size | Best For |
|-------------|------------|-------------|-----------|----------|
| MVC | ต่ำ | ปานกลาง | 1-2 | Project เล็กๆ |
| MVP | ปานกลาง | ดี | 2-5 | UIKit Projects |
| MVVM | ปานกลาง | ดีมาก | 2-10 | SwiftUI/Combine |
| VIPER | สูง | ดีมาก | 5+ | Enterprise Apps |
| Clean Arch | สูงมาก | ดีเยี่ยม | 5+ | Large Scale |

### หลักการที่ควรจำ

1. **ไม่มี Architecture ที่ดีที่สุด** - เลือกตามขนาดและความซับซ้อนของ Project
2. **Start Simple** - เริ่มด้วย MVVM แล้วขยายตามความต้องการ
3. **Testability First** - เขียนโค้ดให้ทดสอบได้ก่อน
4. **Dependency Injection** - ใช้เสมอเพื่อความยืดหยุ่น
5. **Single Responsibility** - แต่ละ class ทำเพียงสิ่งเดียว

### Key Takeaways

- **MVC**: เหมาะสำหรับ Project เล็ก แต่เสี่ยงกับ Massive ViewController
- **MVVM**: เหมาะมากกับ SwiftUI, ใช้ Combine สำหรับ Data Binding
- **Clean Architecture**: แบ่ง Layer ชัดเจน, Domain Logic ไม่ขึ้นกับ Framework
- **Coordinator**: แยก Navigation Logic ออกจาก ViewController
- **UseCase**: แยก Business Logic ออกจาก ViewModel
- **DI**: ทำให้โค้ด Testable และ Flexible

### แนวทางการเรียนรู้

1. เริ่มต้นด้วยการเข้าใจ MVC ดั้งเดิม
2. เรียนรู้ MVVM กับ SwiftUI
3. เพิ่ม Combine สำหรับ Reactive Programming
4. ศึกษา Clean Architecture เมื่อ Project ใหญ่ขึ้น
5. Practice ด้วยการสร้าง Real Projects

---

*จบ Part 37: App Architecture และ MVVM Pattern*

**ต่อไป**: Part 38 - Design Patterns in Swift
