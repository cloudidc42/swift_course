# Part 81: Enterprise iOS Architecture - สถาปัตยกรรม iOS ระดับองค์กร

## บทนำ

ในการพัฒนา iOS Application ระดับองค์กร (Enterprise) ที่มีความซับซ้อนสูง มีทีมพัฒนาหลายทีม และต้องรองรับผู้ใช้งานจำนวนมาก การออกแบบสถาปัตยกรรมที่ดีเป็นสิ่งสำคัญอย่างยิ่ง บทนี้จะครอบคลุมแนวคิดและการ implement สถาปัตยกรรมระดับองค์กรอย่างครบถ้วน

---

## 1. Enterprise Architecture Patterns Overview

### 1.1 Monolith vs Modular vs Micro-services

ก่อนเลือกสถาปัตยกรรม เราต้องเข้าใจข้อดีข้อเสียของแต่ละแนวทาง

#### Monolithic Architecture

```
┌─────────────────────────────────────┐
│         Monolithic App              │
│  ┌─────────────────────────────┐   │
│  │   UI Layer                  │   │
│  ├─────────────────────────────┤   │
│  │   Business Logic            │   │
│  ├─────────────────────────────┤   │
│  │   Data Access               │   │
│  └─────────────────────────────┘   │
└─────────────────────────────────────┘
```

**ข้อดี:**
- ง่ายต่อการพัฒนาเริ่มต้น
- Debug ได้ง่าย
- Deploy ง่าย

**ข้อเสีย:**
- Scale ยาก
- Build time นาน
- ทีมหลายทีมทำงานบน codebase เดียวกัน ทำให้เกิด conflict บ่อย
- Test isolation ทำได้ยาก

#### Modular Architecture

```
┌──────────────────────────────────────────────────┐
│                   App Shell                       │
├──────────┬──────────┬──────────┬─────────────────┤
│ Feature  │ Feature  │ Feature  │    Feature       │
│   Auth   │  Home    │  Cart    │   Profile        │
├──────────┴──────────┴──────────┴─────────────────┤
│              Shared Kernel / Core                 │
├──────────────────────────────────────────────────┤
│         Network  │  Storage  │  Analytics        │
└──────────────────────────────────────────────────┘
```

**ข้อดี:**
- แต่ละ module develop ได้อิสระ
- Build time เร็วขึ้น (เฉพาะ module ที่เปลี่ยน)
- Test แต่ละ feature ได้แยก
- ทีมต่างๆ ทำงานบน module ของตัวเองได้

**ข้อเสีย:**
- Initial setup ซับซ้อน
- ต้องจัดการ module dependencies
- Versioning ระหว่าง module

#### Micro-services (Micro-frontends concept for Mobile)

```
┌──────────────────────────────────────────────────┐
│                   Host App                        │
├──────────────────────────────────────────────────┤
│  Dynamic Framework A  │  Dynamic Framework B     │
│  (loaded at runtime)  │  (loaded at runtime)     │
├──────────────────────────────────────────────────┤
│              Core Runtime                         │
└──────────────────────────────────────────────────┘
```

### 1.2 การเลือก Architecture ที่เหมาะสม

```swift
// ตัวอย่าง: Decision Framework
struct ArchitectureDecision {
    let teamSize: Int
    let projectComplexity: ProjectComplexity
    let buildTimeRequirement: BuildTimeRequirement
    
    enum ProjectComplexity {
        case simple   // < 20 screens
        case medium   // 20-50 screens
        case complex  // 50+ screens
    }
    
    enum BuildTimeRequirement {
        case acceptable  // < 3 minutes
        case fast        // < 1 minute
        case instant     // < 30 seconds
    }
    
    var recommendedArchitecture: String {
        switch (teamSize, projectComplexity) {
        case (_, .simple):
            return "Monolith with MVVM"
        case (1...5, .medium):
            return "Monolith with Clean Architecture"
        case (6...20, .medium), (_, .complex):
            return "Modular with Feature Modules"
        default:
            return "Modular with Dynamic Frameworks"
        }
    }
}
```

---

## 2. Clean Architecture - การ implement อย่างสมบูรณ์

Clean Architecture แบ่ง application ออกเป็น layers ที่แยกจากกันอย่างชัดเจน โดยมีกฎ Dependency Rule ที่ว่า dependencies ต้องชี้เข้าหา center เสมอ

```
┌─────────────────────────────────────────────┐
│           Presentation Layer                 │
│         (ViewModels, Views)                  │
├─────────────────────────────────────────────┤
│           Domain Layer (Core)                │
│      (Entities, Use Cases, Interfaces)       │
├─────────────────────────────────────────────┤
│            Data Layer                        │
│    (Repositories, Data Sources, Mappers)     │
└─────────────────────────────────────────────┘
        ↑ Dependencies point inward ↑
```

### 2.1 Domain Layer

Domain Layer เป็นแกนกลางของ application และไม่มี dependency ต่อ framework ภายนอก

```swift
// MARK: - Domain Entities
// ไฟล์: Domain/Entities/Product.swift

import Foundation

struct Product: Identifiable, Equatable {
    let id: String
    let name: String
    let description: String
    let price: Decimal
    let imageURL: URL?
    let category: Category
    let stockQuantity: Int
    let rating: Double
    
    enum Category: String, CaseIterable {
        case electronics = "electronics"
        case clothing = "clothing"
        case food = "food"
        case books = "books"
    }
    
    var isAvailable: Bool {
        stockQuantity > 0
    }
    
    var formattedPrice: String {
        let formatter = NumberFormatter()
        formatter.numberStyle = .currency
        formatter.locale = Locale(identifier: "th_TH")
        return formatter.string(from: price as NSDecimalNumber) ?? "\(price)"
    }
}

// MARK: - User Entity
struct User: Identifiable, Equatable {
    let id: String
    var name: String
    var email: String
    var profileImageURL: URL?
    var address: Address?
    var paymentMethods: [PaymentMethod]
    
    struct Address: Equatable {
        var street: String
        var city: String
        var province: String
        var postalCode: String
        var country: String
    }
    
    struct PaymentMethod: Identifiable, Equatable {
        let id: String
        let type: PaymentType
        let lastFourDigits: String?
        var isDefault: Bool
        
        enum PaymentType {
            case creditCard
            case debitCard
            case promptPay
            case trueMoney
            case linePay
        }
    }
}

// MARK: - Order Entity
struct Order: Identifiable {
    let id: String
    let userId: String
    var items: [OrderItem]
    var status: OrderStatus
    let createdAt: Date
    var updatedAt: Date
    var shippingAddress: User.Address
    var paymentMethod: User.PaymentMethod
    var trackingNumber: String?
    
    struct OrderItem: Identifiable, Equatable {
        let id: String
        let product: Product
        var quantity: Int
        let priceAtPurchase: Decimal
        
        var subtotal: Decimal {
            priceAtPurchase * Decimal(quantity)
        }
    }
    
    enum OrderStatus: String {
        case pending = "pending"
        case confirmed = "confirmed"
        case processing = "processing"
        case shipped = "shipped"
        case delivered = "delivered"
        case cancelled = "cancelled"
        case refunded = "refunded"
    }
    
    var totalAmount: Decimal {
        items.reduce(Decimal.zero) { $0 + $1.subtotal }
    }
}
```

### 2.2 Repository Interfaces (Use Case Boundaries)

```swift
// MARK: - Repository Protocols
// ไฟล์: Domain/Interfaces/ProductRepository.swift

import Combine
import Foundation

protocol ProductRepository {
    func getProducts(category: Product.Category?, page: Int, pageSize: Int) async throws -> PaginatedResult<Product>
    func getProduct(id: String) async throws -> Product
    func searchProducts(query: String) async throws -> [Product]
    func getFeaturedProducts() async throws -> [Product]
}

protocol UserRepository {
    func getCurrentUser() async throws -> User
    func updateUser(_ user: User) async throws -> User
    func addPaymentMethod(_ method: User.PaymentMethod) async throws -> User
    func removePaymentMethod(id: String) async throws -> User
}

protocol OrderRepository {
    func createOrder(_ order: Order) async throws -> Order
    func getOrders(userId: String) async throws -> [Order]
    func getOrder(id: String) async throws -> Order
    func cancelOrder(id: String) async throws -> Order
    func trackOrder(id: String) -> AsyncStream<Order>
}

protocol CartRepository {
    func getCart(userId: String) async throws -> Cart
    func addItem(_ item: Order.OrderItem, userId: String) async throws -> Cart
    func updateItem(_ item: Order.OrderItem, userId: String) async throws -> Cart
    func removeItem(itemId: String, userId: String) async throws -> Cart
    func clearCart(userId: String) async throws
}

// MARK: - Pagination Model
struct PaginatedResult<T> {
    let items: [T]
    let currentPage: Int
    let totalPages: Int
    let totalItems: Int
    
    var hasNextPage: Bool { currentPage < totalPages }
    var hasPreviousPage: Bool { currentPage > 1 }
}

// MARK: - Cart Entity
struct Cart: Identifiable {
    let id: String
    let userId: String
    var items: [Order.OrderItem]
    var appliedCoupon: Coupon?
    
    struct Coupon {
        let code: String
        let discountPercentage: Double
        let maxDiscountAmount: Decimal?
    }
    
    var subtotal: Decimal {
        items.reduce(Decimal.zero) { $0 + $1.subtotal }
    }
    
    var discount: Decimal {
        guard let coupon = appliedCoupon else { return .zero }
        let calculatedDiscount = subtotal * Decimal(coupon.discountPercentage / 100)
        if let maxDiscount = coupon.maxDiscountAmount {
            return min(calculatedDiscount, maxDiscount)
        }
        return calculatedDiscount
    }
    
    var total: Decimal {
        subtotal - discount
    }
}
```

### 2.3 Use Cases

```swift
// MARK: - Use Cases
// ไฟล์: Domain/UseCases/ProductUseCases.swift

import Foundation

// Base Use Case Protocol
protocol UseCase {
    associatedtype Input
    associatedtype Output
    func execute(_ input: Input) async throws -> Output
}

// Get Products Use Case
final class GetProductsUseCase: UseCase {
    typealias Input = GetProductsInput
    typealias Output = PaginatedResult<Product>
    
    private let productRepository: ProductRepository
    
    init(productRepository: ProductRepository) {
        self.productRepository = productRepository
    }
    
    struct GetProductsInput {
        let category: Product.Category?
        let page: Int
        let pageSize: Int
        
        init(category: Product.Category? = nil, page: Int = 1, pageSize: Int = 20) {
            self.category = category
            self.page = page
            self.pageSize = pageSize
        }
    }
    
    func execute(_ input: GetProductsInput) async throws -> PaginatedResult<Product> {
        return try await productRepository.getProducts(
            category: input.category,
            page: input.page,
            pageSize: input.pageSize
        )
    }
}

// Search Products Use Case
final class SearchProductsUseCase: UseCase {
    typealias Input = String
    typealias Output = [Product]
    
    private let productRepository: ProductRepository
    private let minimumQueryLength = 2
    
    init(productRepository: ProductRepository) {
        self.productRepository = productRepository
    }
    
    func execute(_ input: String) async throws -> [Product] {
        guard input.count >= minimumQueryLength else {
            throw SearchError.queryTooShort(minimum: minimumQueryLength)
        }
        return try await productRepository.searchProducts(query: input.trimmingCharacters(in: .whitespaces))
    }
    
    enum SearchError: LocalizedError {
        case queryTooShort(minimum: Int)
        
        var errorDescription: String? {
            switch self {
            case .queryTooShort(let min):
                return "กรุณากรอกคำค้นหาอย่างน้อย \(min) ตัวอักษร"
            }
        }
    }
}

// Add to Cart Use Case
final class AddToCartUseCase: UseCase {
    typealias Input = AddToCartInput
    typealias Output = Cart
    
    private let cartRepository: CartRepository
    private let productRepository: ProductRepository
    
    init(cartRepository: CartRepository, productRepository: ProductRepository) {
        self.cartRepository = cartRepository
        self.productRepository = productRepository
    }
    
    struct AddToCartInput {
        let userId: String
        let productId: String
        let quantity: Int
    }
    
    func execute(_ input: AddToCartInput) async throws -> Cart {
        // Validate product exists and is available
        let product = try await productRepository.getProduct(id: input.productId)
        
        guard product.isAvailable else {
            throw CartError.productUnavailable(productId: input.productId)
        }
        
        guard product.stockQuantity >= input.quantity else {
            throw CartError.insufficientStock(
                available: product.stockQuantity,
                requested: input.quantity
            )
        }
        
        let orderItem = Order.OrderItem(
            id: UUID().uuidString,
            product: product,
            quantity: input.quantity,
            priceAtPurchase: product.price
        )
        
        return try await cartRepository.addItem(orderItem, userId: input.userId)
    }
    
    enum CartError: LocalizedError {
        case productUnavailable(productId: String)
        case insufficientStock(available: Int, requested: Int)
        
        var errorDescription: String? {
            switch self {
            case .productUnavailable:
                return "สินค้าหมดชั่วคราว"
            case .insufficientStock(let available, let requested):
                return "สินค้าในสต็อกมีเพียง \(available) ชิ้น แต่คุณต้องการ \(requested) ชิ้น"
            }
        }
    }
}

// Place Order Use Case
final class PlaceOrderUseCase: UseCase {
    typealias Input = PlaceOrderInput
    typealias Output = Order
    
    private let orderRepository: OrderRepository
    private let cartRepository: CartRepository
    private let userRepository: UserRepository
    
    init(
        orderRepository: OrderRepository,
        cartRepository: CartRepository,
        userRepository: UserRepository
    ) {
        self.orderRepository = orderRepository
        self.cartRepository = cartRepository
        self.userRepository = userRepository
    }
    
    struct PlaceOrderInput {
        let userId: String
        let shippingAddressId: String?
        let paymentMethodId: String
    }
    
    func execute(_ input: PlaceOrderInput) async throws -> Order {
        // Fetch cart and user concurrently
        async let cartTask = cartRepository.getCart(userId: input.userId)
        async let userTask = userRepository.getCurrentUser()
        
        let (cart, user) = try await (cartTask, userTask)
        
        guard !cart.items.isEmpty else {
            throw OrderError.emptyCart
        }
        
        guard let paymentMethod = user.paymentMethods.first(where: { $0.id == input.paymentMethodId }) else {
            throw OrderError.invalidPaymentMethod
        }
        
        let shippingAddress: User.Address
        if let addressId = input.shippingAddressId,
           let address = user.address {
            shippingAddress = address
        } else if let defaultAddress = user.address {
            shippingAddress = defaultAddress
        } else {
            throw OrderError.noShippingAddress
        }
        
        let order = Order(
            id: UUID().uuidString,
            userId: input.userId,
            items: cart.items,
            status: .pending,
            createdAt: Date(),
            updatedAt: Date(),
            shippingAddress: shippingAddress,
            paymentMethod: paymentMethod,
            trackingNumber: nil
        )
        
        let createdOrder = try await orderRepository.createOrder(order)
        try await cartRepository.clearCart(userId: input.userId)
        
        return createdOrder
    }
    
    enum OrderError: LocalizedError {
        case emptyCart
        case invalidPaymentMethod
        case noShippingAddress
        
        var errorDescription: String? {
            switch self {
            case .emptyCart:
                return "ตะกร้าสินค้าว่างเปล่า"
            case .invalidPaymentMethod:
                return "วิธีการชำระเงินไม่ถูกต้อง"
            case .noShippingAddress:
                return "กรุณาเพิ่มที่อยู่จัดส่ง"
            }
        }
    }
}
```

### 2.4 Data Layer - Repository Implementations

```swift
// MARK: - Data Layer
// ไฟล์: Data/Repositories/ProductRepositoryImpl.swift

import Foundation

final class ProductRepositoryImpl: ProductRepository {
    private let remoteDataSource: ProductRemoteDataSource
    private let localDataSource: ProductLocalDataSource
    private let mapper: ProductMapper
    
    init(
        remoteDataSource: ProductRemoteDataSource,
        localDataSource: ProductLocalDataSource,
        mapper: ProductMapper
    ) {
        self.remoteDataSource = remoteDataSource
        self.localDataSource = localDataSource
        self.mapper = mapper
    }
    
    func getProducts(
        category: Product.Category?,
        page: Int,
        pageSize: Int
    ) async throws -> PaginatedResult<Product> {
        do {
            // Try remote first
            let response = try await remoteDataSource.fetchProducts(
                category: category?.rawValue,
                page: page,
                pageSize: pageSize
            )
            
            // Cache to local
            try await localDataSource.saveProducts(response.items)
            
            return PaginatedResult(
                items: response.items.map { mapper.toDomain($0) },
                currentPage: response.page,
                totalPages: response.totalPages,
                totalItems: response.totalItems
            )
        } catch {
            // Fallback to cache
            let cached = try await localDataSource.getProducts(
                category: category?.rawValue,
                page: page,
                pageSize: pageSize
            )
            return PaginatedResult(
                items: cached.map { mapper.toDomain($0) },
                currentPage: page,
                totalPages: 1,
                totalItems: cached.count
            )
        }
    }
    
    func getProduct(id: String) async throws -> Product {
        // Check cache first
        if let cached = try? await localDataSource.getProduct(id: id) {
            return mapper.toDomain(cached)
        }
        
        let dto = try await remoteDataSource.fetchProduct(id: id)
        try await localDataSource.saveProduct(dto)
        return mapper.toDomain(dto)
    }
    
    func searchProducts(query: String) async throws -> [Product] {
        let dtos = try await remoteDataSource.searchProducts(query: query)
        return dtos.map { mapper.toDomain($0) }
    }
    
    func getFeaturedProducts() async throws -> [Product] {
        let dtos = try await remoteDataSource.fetchFeaturedProducts()
        return dtos.map { mapper.toDomain($0) }
    }
}

// MARK: - Data Transfer Objects (DTOs)
struct ProductDTO: Codable {
    let id: String
    let name: String
    let description: String
    let price: Double
    let imageUrl: String?
    let category: String
    let stockQuantity: Int
    let rating: Double
    
    enum CodingKeys: String, CodingKey {
        case id, name, description, price
        case imageUrl = "image_url"
        case category
        case stockQuantity = "stock_quantity"
        case rating
    }
}

// MARK: - Mapper
struct ProductMapper {
    func toDomain(_ dto: ProductDTO) -> Product {
        Product(
            id: dto.id,
            name: dto.name,
            description: dto.description,
            price: Decimal(dto.price),
            imageURL: dto.imageUrl.flatMap { URL(string: $0) },
            category: Product.Category(rawValue: dto.category) ?? .electronics,
            stockQuantity: dto.stockQuantity,
            rating: dto.rating
        )
    }
    
    func toDTO(_ domain: Product) -> ProductDTO {
        ProductDTO(
            id: domain.id,
            name: domain.name,
            description: domain.description,
            price: NSDecimalNumber(decimal: domain.price).doubleValue,
            imageUrl: domain.imageURL?.absoluteString,
            category: domain.category.rawValue,
            stockQuantity: domain.stockQuantity,
            rating: domain.rating
        )
    }
}

// MARK: - Remote Data Source
protocol ProductRemoteDataSource {
    func fetchProducts(category: String?, page: Int, pageSize: Int) async throws -> PaginatedResponseDTO<ProductDTO>
    func fetchProduct(id: String) async throws -> ProductDTO
    func searchProducts(query: String) async throws -> [ProductDTO]
    func fetchFeaturedProducts() async throws -> [ProductDTO]
}

struct PaginatedResponseDTO<T: Codable>: Codable {
    let items: [T]
    let page: Int
    let totalPages: Int
    let totalItems: Int
}

final class ProductRemoteDataSourceImpl: ProductRemoteDataSource {
    private let apiClient: APIClient
    
    init(apiClient: APIClient) {
        self.apiClient = apiClient
    }
    
    func fetchProducts(category: String?, page: Int, pageSize: Int) async throws -> PaginatedResponseDTO<ProductDTO> {
        var params: [String: Any] = ["page": page, "page_size": pageSize]
        if let category = category {
            params["category"] = category
        }
        return try await apiClient.request(
            endpoint: ProductEndpoint.list(params: params)
        )
    }
    
    func fetchProduct(id: String) async throws -> ProductDTO {
        return try await apiClient.request(endpoint: ProductEndpoint.detail(id: id))
    }
    
    func searchProducts(query: String) async throws -> [ProductDTO] {
        return try await apiClient.request(
            endpoint: ProductEndpoint.search(query: query)
        )
    }
    
    func fetchFeaturedProducts() async throws -> [ProductDTO] {
        return try await apiClient.request(endpoint: ProductEndpoint.featured)
    }
}

// MARK: - Local Data Source (Cache)
protocol ProductLocalDataSource {
    func saveProducts(_ products: [ProductDTO]) async throws
    func saveProduct(_ product: ProductDTO) async throws
    func getProducts(category: String?, page: Int, pageSize: Int) async throws -> [ProductDTO]
    func getProduct(id: String) async throws -> ProductDTO?
}
```

### 2.5 Presentation Layer

```swift
// MARK: - Presentation Layer
// ไฟล์: Presentation/ViewModels/ProductListViewModel.swift

import Foundation
import Combine

@MainActor
final class ProductListViewModel: ObservableObject {
    // MARK: - Published State
    @Published private(set) var products: [Product] = []
    @Published private(set) var isLoading = false
    @Published private(set) var error: Error?
    @Published var selectedCategory: Product.Category?
    @Published var searchQuery = ""
    
    // MARK: - Pagination State
    private var currentPage = 1
    private var totalPages = 1
    @Published private(set) var isLoadingMore = false
    
    // MARK: - Dependencies (Use Cases)
    private let getProductsUseCase: GetProductsUseCase
    private let searchProductsUseCase: SearchProductsUseCase
    
    // MARK: - Private
    private var cancellables = Set<AnyCancellable>()
    private var searchTask: Task<Void, Never>?
    
    init(
        getProductsUseCase: GetProductsUseCase,
        searchProductsUseCase: SearchProductsUseCase
    ) {
        self.getProductsUseCase = getProductsUseCase
        self.searchProductsUseCase = searchProductsUseCase
        
        setupSearchDebounce()
    }
    
    // MARK: - Public Actions
    func loadProducts() async {
        currentPage = 1
        isLoading = true
        error = nil
        
        do {
            let result = try await getProductsUseCase.execute(
                .init(category: selectedCategory, page: 1)
            )
            products = result.items
            totalPages = result.totalPages
        } catch {
            self.error = error
        }
        
        isLoading = false
    }
    
    func loadMore() async {
        guard currentPage < totalPages, !isLoadingMore else { return }
        
        isLoadingMore = true
        let nextPage = currentPage + 1
        
        do {
            let result = try await getProductsUseCase.execute(
                .init(category: selectedCategory, page: nextPage)
            )
            products.append(contentsOf: result.items)
            currentPage = nextPage
        } catch {
            self.error = error
        }
        
        isLoadingMore = false
    }
    
    func selectCategory(_ category: Product.Category?) {
        selectedCategory = category
        Task { await loadProducts() }
    }
    
    // MARK: - Private Helpers
    private func setupSearchDebounce() {
        $searchQuery
            .debounce(for: .milliseconds(500), scheduler: RunLoop.main)
            .removeDuplicates()
            .sink { [weak self] query in
                guard let self else { return }
                if query.isEmpty {
                    Task { await self.loadProducts() }
                } else {
                    self.performSearch(query: query)
                }
            }
            .store(in: &cancellables)
    }
    
    private func performSearch(query: String) {
        searchTask?.cancel()
        searchTask = Task { [weak self] in
            guard let self else { return }
            
            isLoading = true
            
            do {
                let results = try await searchProductsUseCase.execute(query)
                guard !Task.isCancelled else { return }
                products = results
            } catch {
                guard !Task.isCancelled else { return }
                self.error = error
            }
            
            isLoading = false
        }
    }
}

// MARK: - Product List View
import SwiftUI

struct ProductListView: View {
    @StateObject private var viewModel: ProductListViewModel
    
    init(viewModel: ProductListViewModel) {
        _viewModel = StateObject(wrappedValue: viewModel)
    }
    
    var body: some View {
        NavigationView {
            VStack(spacing: 0) {
                // Search Bar
                SearchBar(text: $viewModel.searchQuery)
                    .padding(.horizontal)
                
                // Category Filter
                CategoryFilterView(
                    selectedCategory: viewModel.selectedCategory,
                    onSelect: { viewModel.selectCategory($0) }
                )
                
                // Product List
                if viewModel.isLoading && viewModel.products.isEmpty {
                    ProgressView("กำลังโหลด...")
                        .frame(maxWidth: .infinity, maxHeight: .infinity)
                } else if let error = viewModel.error {
                    ErrorView(error: error) {
                        Task { await viewModel.loadProducts() }
                    }
                } else {
                    productList
                }
            }
            .navigationTitle("สินค้า")
        }
        .task { await viewModel.loadProducts() }
    }
    
    private var productList: some View {
        ScrollView {
            LazyVGrid(columns: [
                GridItem(.flexible()),
                GridItem(.flexible())
            ], spacing: 16) {
                ForEach(viewModel.products) { product in
                    ProductCard(product: product)
                }
                
                // Load more trigger
                if viewModel.isLoadingMore {
                    ProgressView()
                        .gridCellColumns(2)
                } else {
                    Color.clear
                        .frame(height: 1)
                        .gridCellColumns(2)
                        .onAppear {
                            Task { await viewModel.loadMore() }
                        }
                }
            }
            .padding()
        }
    }
}
```

---

## 3. Feature-based Modular Architecture

### 3.1 Module Structure กับ Swift Package Manager

```
ECommerceApp/
├── App/                          # App shell
│   ├── AppDelegate.swift
│   └── RootCoordinator.swift
├── Packages/
│   ├── FeatureAuth/              # Authentication module
│   │   ├── Package.swift
│   │   └── Sources/
│   ├── FeatureHome/              # Home module
│   │   ├── Package.swift
│   │   └── Sources/
│   ├── FeatureCart/              # Cart module
│   │   ├── Package.swift
│   │   └── Sources/
│   ├── FeatureProfile/           # Profile module
│   │   ├── Package.swift
│   │   └── Sources/
│   ├── SharedKernel/             # Shared utilities
│   │   ├── Package.swift
│   │   └── Sources/
│   ├── NetworkKit/               # Networking
│   │   ├── Package.swift
│   │   └── Sources/
│   └── DesignSystem/             # UI Components
│       ├── Package.swift
│       └── Sources/
```

### 3.2 Package.swift สำหรับ Feature Module

```swift
// FeatureCart/Package.swift
// swift-tools-version: 5.9

import PackageDescription

let package = Package(
    name: "FeatureCart",
    platforms: [.iOS(.v16)],
    products: [
        .library(
            name: "FeatureCart",
            targets: ["FeatureCart"]
        ),
    ],
    dependencies: [
        .package(path: "../SharedKernel"),
        .package(path: "../DesignSystem"),
        .package(path: "../NetworkKit"),
    ],
    targets: [
        .target(
            name: "FeatureCart",
            dependencies: [
                "SharedKernel",
                "DesignSystem",
                "NetworkKit",
            ]
        ),
        .testTarget(
            name: "FeatureCartTests",
            dependencies: ["FeatureCart"]
        ),
    ]
)
```

### 3.3 Inter-module Communication via Protocols

```swift
// SharedKernel/Sources/Navigation/AppNavigator.swift
// Protocol สำหรับ navigation ระหว่าง modules

public protocol AppNavigator: AnyObject {
    func navigateToAuth()
    func navigateToProductDetail(productId: String)
    func navigateToCart()
    func navigateToOrderConfirmation(orderId: String)
    func navigateToProfile()
    func dismiss()
}

// SharedKernel/Sources/Events/AppEventBus.swift
public protocol AppEvent {}

public struct UserLoggedInEvent: AppEvent {
    public let userId: String
    public let userEmail: String
    
    public init(userId: String, userEmail: String) {
        self.userId = userId
        self.userEmail = userEmail
    }
}

public struct CartUpdatedEvent: AppEvent {
    public let itemCount: Int
    
    public init(itemCount: Int) {
        self.itemCount = itemCount
    }
}

public struct OrderPlacedEvent: AppEvent {
    public let orderId: String
    
    public init(orderId: String) {
        self.orderId = orderId
    }
}

// Event Bus Implementation
public final class AppEventBus {
    public static let shared = AppEventBus()
    
    private var handlers: [String: [(any AppEvent) -> Void]] = [:]
    private let queue = DispatchQueue(label: "com.app.eventbus", attributes: .concurrent)
    
    private init() {}
    
    public func subscribe<E: AppEvent>(
        to eventType: E.Type,
        handler: @escaping (E) -> Void
    ) -> EventSubscription {
        let key = String(describing: eventType)
        let wrapper: (any AppEvent) -> Void = { event in
            if let typedEvent = event as? E {
                handler(typedEvent)
            }
        }
        
        queue.async(flags: .barrier) {
            if self.handlers[key] == nil {
                self.handlers[key] = []
            }
            self.handlers[key]?.append(wrapper)
        }
        
        return EventSubscription(eventBus: self, key: key)
    }
    
    public func publish<E: AppEvent>(_ event: E) {
        let key = String(describing: type(of: event))
        queue.async {
            self.handlers[key]?.forEach { handler in
                DispatchQueue.main.async { handler(event) }
            }
        }
    }
}

public final class EventSubscription {
    private weak var eventBus: AppEventBus?
    private let key: String
    
    init(eventBus: AppEventBus, key: String) {
        self.eventBus = eventBus
        self.key = key
    }
    
    deinit {
        // Cleanup subscription
    }
}
```

### 3.4 Dependency Injection Container

```swift
// SharedKernel/Sources/DI/DIContainer.swift

import Foundation

// MARK: - DI Container
public final class DIContainer {
    public static let shared = DIContainer()
    
    private var factories: [String: () -> Any] = [:]
    private var singletons: [String: Any] = [:]
    private let lock = NSLock()
    
    private init() {}
    
    // MARK: - Registration
    public func register<T>(_ type: T.Type, factory: @escaping () -> T) {
        let key = String(describing: type)
        lock.lock()
        factories[key] = factory
        lock.unlock()
    }
    
    public func registerSingleton<T>(_ type: T.Type, factory: @escaping () -> T) {
        let key = String(describing: type)
        lock.lock()
        factories[key] = { [weak self] in
            guard let self else { return factory() }
            if let existing = self.singletons[key] as? T {
                return existing
            }
            let instance = factory()
            self.singletons[key] = instance
            return instance
        }
        lock.unlock()
    }
    
    // MARK: - Resolution
    public func resolve<T>(_ type: T.Type) -> T {
        let key = String(describing: type)
        lock.lock()
        let factory = factories[key]
        lock.unlock()
        
        guard let instance = factory?() as? T else {
            fatalError("No registration found for type: \(key)")
        }
        return instance
    }
    
    public func resolveOptional<T>(_ type: T.Type) -> T? {
        let key = String(describing: type)
        lock.lock()
        let factory = factories[key]
        lock.unlock()
        return factory?() as? T
    }
}

// MARK: - @Inject Property Wrapper
@propertyWrapper
public struct Inject<T> {
    private var value: T?
    
    public var wrappedValue: T {
        mutating get {
            if value == nil {
                value = DIContainer.shared.resolve(T.self)
            }
            return value!
        }
    }
    
    public init() {}
}

// MARK: - App DI Setup
// AppDelegate.swift หรือ App.swift

struct AppDependencies {
    static func setup() {
        let container = DIContainer.shared
        
        // Networking
        container.registerSingleton(APIClient.self) {
            APIClientImpl(baseURL: URL(string: "https://api.example.com")!)
        }
        
        // Data Sources
        container.register(ProductRemoteDataSource.self) {
            ProductRemoteDataSourceImpl(
                apiClient: container.resolve(APIClient.self)
            )
        }
        
        // Repositories
        container.registerSingleton(ProductRepository.self) {
            ProductRepositoryImpl(
                remoteDataSource: container.resolve(ProductRemoteDataSource.self),
                localDataSource: ProductLocalDataSourceImpl(),
                mapper: ProductMapper()
            )
        }
        
        // Use Cases
        container.register(GetProductsUseCase.self) {
            GetProductsUseCase(
                productRepository: container.resolve(ProductRepository.self)
            )
        }
        
        container.register(SearchProductsUseCase.self) {
            SearchProductsUseCase(
                productRepository: container.resolve(ProductRepository.self)
            )
        }
        
        // ViewModels
        container.register(ProductListViewModel.self) {
            ProductListViewModel(
                getProductsUseCase: container.resolve(GetProductsUseCase.self),
                searchProductsUseCase: container.resolve(SearchProductsUseCase.self)
            )
        }
    }
}
```

---

## 4. Scalable Networking Layer

### 4.1 Generic API Client

```swift
// NetworkKit/Sources/APIClient.swift

import Foundation

// MARK: - Endpoint Protocol
public protocol Endpoint {
    var path: String { get }
    var method: HTTPMethod { get }
    var headers: [String: String]? { get }
    var parameters: [String: Any]? { get }
    var body: Encodable? { get }
}

public enum HTTPMethod: String {
    case GET, POST, PUT, PATCH, DELETE
}

// MARK: - API Error
public enum APIError: Error, LocalizedError {
    case invalidURL
    case networkError(Error)
    case invalidResponse
    case httpError(statusCode: Int, data: Data?)
    case decodingError(Error)
    case unauthorized
    case notFound
    case serverError(message: String)
    case rateLimitExceeded
    case timeout
    
    public var errorDescription: String? {
        switch self {
        case .invalidURL:
            return "URL ไม่ถูกต้อง"
        case .networkError(let error):
            return "เกิดข้อผิดพลาดเครือข่าย: \(error.localizedDescription)"
        case .invalidResponse:
            return "ได้รับ response ที่ไม่ถูกต้อง"
        case .httpError(let code, _):
            return "HTTP Error: \(code)"
        case .decodingError(let error):
            return "ไม่สามารถ decode ข้อมูล: \(error.localizedDescription)"
        case .unauthorized:
            return "ไม่ได้รับอนุญาต กรุณาล็อกอินใหม่"
        case .notFound:
            return "ไม่พบข้อมูล"
        case .serverError(let message):
            return "เกิดข้อผิดพลาดที่ server: \(message)"
        case .rateLimitExceeded:
            return "คำขอมากเกินไป กรุณารอสักครู่"
        case .timeout:
            return "หมดเวลาการเชื่อมต่อ"
        }
    }
}

// MARK: - API Client Protocol
public protocol APIClient {
    func request<T: Decodable>(endpoint: any Endpoint) async throws -> T
    func requestWithProgress<T: Decodable>(
        endpoint: any Endpoint,
        progressHandler: @escaping (Double) -> Void
    ) async throws -> T
}

// MARK: - API Client Implementation
public final class APIClientImpl: APIClient {
    private let baseURL: URL
    private let session: URLSession
    private var interceptors: [RequestInterceptor]
    private var responseInterceptors: [ResponseInterceptor]
    private let retryPolicy: RetryPolicy
    
    public init(
        baseURL: URL,
        session: URLSession = .shared,
        interceptors: [RequestInterceptor] = [],
        responseInterceptors: [ResponseInterceptor] = [],
        retryPolicy: RetryPolicy = ExponentialBackoffRetryPolicy()
    ) {
        self.baseURL = baseURL
        self.session = session
        self.interceptors = interceptors
        self.responseInterceptors = responseInterceptors
        self.retryPolicy = retryPolicy
    }
    
    public func request<T: Decodable>(endpoint: any Endpoint) async throws -> T {
        let urlRequest = try buildURLRequest(for: endpoint)
        return try await executeWithRetry(request: urlRequest, retryCount: 0)
    }
    
    public func requestWithProgress<T: Decodable>(
        endpoint: any Endpoint,
        progressHandler: @escaping (Double) -> Void
    ) async throws -> T {
        // Implementation for progress tracking
        fatalError("Not implemented")
    }
    
    // MARK: - Private Helpers
    private func buildURLRequest(for endpoint: any Endpoint) throws -> URLRequest {
        guard let url = URL(string: endpoint.path, relativeTo: baseURL) else {
            throw APIError.invalidURL
        }
        
        var urlComponents = URLComponents(url: url, resolvingAgainstBaseURL: true)!
        
        if endpoint.method == .GET, let parameters = endpoint.parameters {
            urlComponents.queryItems = parameters.map { key, value in
                URLQueryItem(name: key, value: "\(value)")
            }
        }
        
        guard let finalURL = urlComponents.url else {
            throw APIError.invalidURL
        }
        
        var request = URLRequest(url: finalURL)
        request.httpMethod = endpoint.method.rawValue
        request.timeoutInterval = 30
        
        // Default headers
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        request.setValue("application/json", forHTTPHeaderField: "Accept")
        
        // Custom headers
        endpoint.headers?.forEach { key, value in
            request.setValue(value, forHTTPHeaderField: key)
        }
        
        // Body
        if let body = endpoint.body {
            let encoder = JSONEncoder()
            encoder.keyEncodingStrategy = .convertToSnakeCase
            request.httpBody = try encoder.encode(body)
        } else if endpoint.method != .GET, let parameters = endpoint.parameters {
            let data = try JSONSerialization.data(withJSONObject: parameters)
            request.httpBody = data
        }
        
        // Apply interceptors
        var interceptedRequest = request
        for interceptor in interceptors {
            interceptedRequest = try await interceptor.intercept(request: interceptedRequest)
        }
        
        return interceptedRequest
    }
    
    private func executeWithRetry<T: Decodable>(
        request: URLRequest,
        retryCount: Int
    ) async throws -> T {
        do {
            let (data, response) = try await session.data(for: request)
            
            // Apply response interceptors
            var interceptedData = data
            for interceptor in responseInterceptors {
                interceptedData = try await interceptor.intercept(
                    data: interceptedData,
                    response: response
                )
            }
            
            guard let httpResponse = response as? HTTPURLResponse else {
                throw APIError.invalidResponse
            }
            
            switch httpResponse.statusCode {
            case 200...299:
                break
            case 401:
                throw APIError.unauthorized
            case 404:
                throw APIError.notFound
            case 429:
                throw APIError.rateLimitExceeded
            case 500...599:
                if let errorBody = try? JSONDecoder().decode(ErrorResponse.self, from: data) {
                    throw APIError.serverError(message: errorBody.message)
                }
                throw APIError.httpError(statusCode: httpResponse.statusCode, data: data)
            default:
                throw APIError.httpError(statusCode: httpResponse.statusCode, data: data)
            }
            
            let decoder = JSONDecoder()
            decoder.keyDecodingStrategy = .convertFromSnakeCase
            decoder.dateDecodingStrategy = .iso8601
            
            return try decoder.decode(T.self, from: interceptedData)
            
        } catch let error as APIError {
            // Check if should retry
            if retryPolicy.shouldRetry(error: error, attempt: retryCount) {
                let delay = retryPolicy.delay(for: retryCount)
                try await Task.sleep(nanoseconds: UInt64(delay * 1_000_000_000))
                return try await executeWithRetry(request: request, retryCount: retryCount + 1)
            }
            throw error
        } catch {
            throw APIError.networkError(error)
        }
    }
}

struct ErrorResponse: Codable {
    let message: String
    let code: String?
}

// MARK: - Interceptors
public protocol RequestInterceptor {
    func intercept(request: URLRequest) async throws -> URLRequest
}

public protocol ResponseInterceptor {
    func intercept(data: Data, response: URLResponse) async throws -> Data
}

// Auth Interceptor
public final class AuthInterceptor: RequestInterceptor {
    private let tokenProvider: TokenProvider
    
    public init(tokenProvider: TokenProvider) {
        self.tokenProvider = tokenProvider
    }
    
    public func intercept(request: URLRequest) async throws -> URLRequest {
        var mutableRequest = request
        if let token = await tokenProvider.getAccessToken() {
            mutableRequest.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        }
        return mutableRequest
    }
}

public protocol TokenProvider {
    func getAccessToken() async -> String?
    func refreshToken() async throws -> String
}

// Logging Interceptor
public final class LoggingInterceptor: RequestInterceptor {
    public func intercept(request: URLRequest) async throws -> URLRequest {
        #if DEBUG
        print("🌐 [\(request.httpMethod ?? "?")] \(request.url?.absoluteString ?? "")")
        if let body = request.httpBody,
           let json = try? JSONSerialization.jsonObject(with: body),
           let prettyData = try? JSONSerialization.data(withJSONObject: json, options: .prettyPrinted),
           let prettyString = String(data: prettyData, encoding: .utf8) {
            print("📤 Request Body:\n\(prettyString)")
        }
        #endif
        return request
    }
}
```

### 4.2 Retry Logic กับ Exponential Backoff

```swift
// NetworkKit/Sources/RetryPolicy.swift

public protocol RetryPolicy {
    func shouldRetry(error: APIError, attempt: Int) -> Bool
    func delay(for attempt: Int) -> Double
}

public struct ExponentialBackoffRetryPolicy: RetryPolicy {
    public let maxAttempts: Int
    public let initialDelay: Double
    public let multiplier: Double
    public let maxDelay: Double
    public let retryableErrors: Set<APIErrorType>
    
    public enum APIErrorType: Hashable {
        case networkError
        case serverError
        case timeout
        case rateLimitExceeded
    }
    
    public init(
        maxAttempts: Int = 3,
        initialDelay: Double = 1.0,
        multiplier: Double = 2.0,
        maxDelay: Double = 30.0,
        retryableErrors: Set<APIErrorType> = [.networkError, .serverError, .timeout]
    ) {
        self.maxAttempts = maxAttempts
        self.initialDelay = initialDelay
        self.multiplier = multiplier
        self.maxDelay = maxDelay
        self.retryableErrors = retryableErrors
    }
    
    public func shouldRetry(error: APIError, attempt: Int) -> Bool {
        guard attempt < maxAttempts else { return false }
        
        switch error {
        case .networkError:
            return retryableErrors.contains(.networkError)
        case .serverError:
            return retryableErrors.contains(.serverError)
        case .timeout:
            return retryableErrors.contains(.timeout)
        case .rateLimitExceeded:
            return retryableErrors.contains(.rateLimitExceeded)
        default:
            return false
        }
    }
    
    public func delay(for attempt: Int) -> Double {
        let jitter = Double.random(in: 0...0.3)
        let exponentialDelay = initialDelay * pow(multiplier, Double(attempt))
        return min(exponentialDelay + jitter, maxDelay)
    }
}
```

### 4.3 Offline-first กับ Sync Queue

```swift
// NetworkKit/Sources/OfflineSync/SyncQueue.swift

import Foundation
import CoreData

// MARK: - Sync Queue สำหรับ Offline-first
final class SyncQueue {
    private let persistenceManager: PersistenceManager
    private let networkMonitor: NetworkMonitor
    private var syncTask: Task<Void, Never>?
    
    init(persistenceManager: PersistenceManager, networkMonitor: NetworkMonitor) {
        self.persistenceManager = persistenceManager
        self.networkMonitor = networkMonitor
        
        setupNetworkObservation()
    }
    
    // MARK: - Enqueue Operation
    func enqueue(_ operation: SyncOperation) async throws {
        try await persistenceManager.save(operation)
        
        if networkMonitor.isConnected {
            await processPendingOperations()
        }
    }
    
    // MARK: - Process Queue
    func processPendingOperations() async {
        guard networkMonitor.isConnected else { return }
        
        syncTask?.cancel()
        syncTask = Task {
            do {
                let operations = try await persistenceManager.fetchPendingOperations()
                
                for operation in operations {
                    guard !Task.isCancelled else { break }
                    
                    do {
                        try await operation.execute()
                        try await persistenceManager.markCompleted(operation)
                    } catch {
                        try await persistenceManager.incrementRetryCount(operation)
                        
                        if operation.retryCount >= 3 {
                            try await persistenceManager.markFailed(operation)
                        }
                    }
                }
            } catch {
                print("Sync error: \(error)")
            }
        }
    }
    
    private func setupNetworkObservation() {
        Task {
            for await isConnected in networkMonitor.statusStream {
                if isConnected {
                    await processPendingOperations()
                }
            }
        }
    }
}

struct SyncOperation: Codable {
    let id: String
    let type: OperationType
    let payload: Data
    let createdAt: Date
    var retryCount: Int
    var status: Status
    
    enum OperationType: String, Codable {
        case createOrder
        case updateCart
        case updateProfile
    }
    
    enum Status: String, Codable {
        case pending
        case processing
        case completed
        case failed
    }
    
    func execute() async throws {
        // Execute based on type
    }
}

// MARK: - Network Monitor
final class NetworkMonitor {
    private let monitor: NWPathMonitor
    private let queue = DispatchQueue(label: "com.app.networkmonitor")
    private let subject = AsyncStream<Bool>.makeStream()
    
    var statusStream: AsyncStream<Bool> { subject.stream }
    
    private(set) var isConnected: Bool = false
    
    init() {
        monitor = NWPathMonitor()
        monitor.pathUpdateHandler = { [weak self] path in
            let connected = path.status == .satisfied
            self?.isConnected = connected
            self?.subject.continuation.yield(connected)
        }
        monitor.start(queue: queue)
    }
    
    deinit {
        monitor.cancel()
    }
}

import Network
```

---

## 5. State Management at Scale

### 5.1 Redux-like State Management

```swift
// StateManagement/Store.swift

import Foundation
import Combine

// MARK: - Redux-like Store
typealias Reducer<State, Action> = (State, Action) -> State
typealias Middleware<State, Action> = (State, Action, @escaping (Action) -> Void) -> Void

@MainActor
final class Store<State, Action>: ObservableObject {
    @Published private(set) var state: State
    
    private let reducer: Reducer<State, Action>
    private var middlewares: [Middleware<State, Action>]
    
    init(
        initialState: State,
        reducer: @escaping Reducer<State, Action>,
        middlewares: [Middleware<State, Action>] = []
    ) {
        self.state = initialState
        self.reducer = reducer
        self.middlewares = middlewares
    }
    
    func dispatch(_ action: Action) {
        // Process middlewares in chain
        var middlewareChain = middlewares
        
        func processNext(_ action: Action) {
            if middlewareChain.isEmpty {
                state = reducer(state, action)
            } else {
                let middleware = middlewareChain.removeFirst()
                middleware(state, action, processNext)
            }
        }
        
        processNext(action)
    }
}

// MARK: - App State
struct AppState {
    var auth: AuthState
    var products: ProductsState
    var cart: CartState
    var orders: OrdersState
    var ui: UIState
}

struct AuthState {
    var isAuthenticated: Bool = false
    var currentUser: User?
    var isLoading: Bool = false
    var error: Error?
}

struct ProductsState {
    var items: [Product] = []
    var isLoading: Bool = false
    var currentPage: Int = 1
    var totalPages: Int = 1
    var selectedCategory: Product.Category?
    var searchQuery: String = ""
    var error: Error?
}

struct CartState {
    var cart: Cart?
    var isLoading: Bool = false
    var error: Error?
}

struct OrdersState {
    var orders: [Order] = []
    var currentOrder: Order?
    var isLoading: Bool = false
    var error: Error?
}

struct UIState {
    var selectedTab: Tab = .home
    var isShowingCart: Bool = false
    
    enum Tab {
        case home, search, cart, orders, profile
    }
}

// MARK: - Actions
enum AppAction {
    case auth(AuthAction)
    case products(ProductsAction)
    case cart(CartAction)
    case orders(OrdersAction)
    case ui(UIAction)
}

enum AuthAction {
    case loginRequested(email: String, password: String)
    case loginSucceeded(user: User)
    case loginFailed(error: Error)
    case logout
}

enum ProductsAction {
    case loadProducts
    case productsLoaded([Product], totalPages: Int)
    case loadingFailed(Error)
    case selectCategory(Product.Category?)
    case search(String)
}

enum CartAction {
    case loadCart
    case cartLoaded(Cart)
    case addItem(productId: String, quantity: Int)
    case removeItem(itemId: String)
    case loadingFailed(Error)
}

enum OrdersAction {
    case placeOrder
    case orderPlaced(Order)
    case loadOrders
    case ordersLoaded([Order])
    case loadingFailed(Error)
}

enum UIAction {
    case selectTab(UIState.Tab)
    case toggleCart
}

// MARK: - Root Reducer
func appReducer(state: AppState, action: AppAction) -> AppState {
    var newState = state
    
    switch action {
    case .auth(let authAction):
        newState.auth = authReducer(state: state.auth, action: authAction)
    case .products(let productsAction):
        newState.products = productsReducer(state: state.products, action: productsAction)
    case .cart(let cartAction):
        newState.cart = cartReducer(state: state.cart, action: cartAction)
    case .orders(let ordersAction):
        newState.orders = ordersReducer(state: state.orders, action: ordersAction)
    case .ui(let uiAction):
        newState.ui = uiReducer(state: state.ui, action: uiAction)
    }
    
    return newState
}

func authReducer(state: AuthState, action: AuthAction) -> AuthState {
    var newState = state
    
    switch action {
    case .loginRequested:
        newState.isLoading = true
        newState.error = nil
    case .loginSucceeded(let user):
        newState.isAuthenticated = true
        newState.currentUser = user
        newState.isLoading = false
    case .loginFailed(let error):
        newState.error = error
        newState.isLoading = false
    case .logout:
        newState = AuthState()
    }
    
    return newState
}

func productsReducer(state: ProductsState, action: ProductsAction) -> ProductsState {
    var newState = state
    
    switch action {
    case .loadProducts:
        newState.isLoading = true
        newState.error = nil
    case .productsLoaded(let products, let totalPages):
        newState.items = products
        newState.totalPages = totalPages
        newState.isLoading = false
    case .loadingFailed(let error):
        newState.error = error
        newState.isLoading = false
    case .selectCategory(let category):
        newState.selectedCategory = category
        newState.items = []
        newState.currentPage = 1
    case .search(let query):
        newState.searchQuery = query
    }
    
    return newState
}

func cartReducer(state: CartState, action: CartAction) -> CartState {
    var newState = state
    
    switch action {
    case .loadCart:
        newState.isLoading = true
    case .cartLoaded(let cart):
        newState.cart = cart
        newState.isLoading = false
    case .addItem:
        newState.isLoading = true
    case .removeItem:
        newState.isLoading = true
    case .loadingFailed(let error):
        newState.error = error
        newState.isLoading = false
    }
    
    return newState
}

func ordersReducer(state: OrdersState, action: OrdersAction) -> OrdersState {
    var newState = state
    switch action {
    case .placeOrder:
        newState.isLoading = true
    case .orderPlaced(let order):
        newState.currentOrder = order
        newState.orders.insert(order, at: 0)
        newState.isLoading = false
    case .loadOrders:
        newState.isLoading = true
    case .ordersLoaded(let orders):
        newState.orders = orders
        newState.isLoading = false
    case .loadingFailed(let error):
        newState.error = error
        newState.isLoading = false
    }
    return newState
}

func uiReducer(state: UIState, action: UIAction) -> UIState {
    var newState = state
    switch action {
    case .selectTab(let tab):
        newState.selectedTab = tab
    case .toggleCart:
        newState.isShowingCart.toggle()
    }
    return newState
}

// MARK: - Middleware Examples
func loggingMiddleware<State, Action>() -> Middleware<State, Action> {
    return { state, action, next in
        #if DEBUG
        print("📦 Action: \(action)")
        #endif
        next(action)
    }
}

func analyticsMiddleware() -> Middleware<AppState, AppAction> {
    return { state, action, next in
        switch action {
        case .orders(.orderPlaced(let order)):
            AnalyticsService.shared.track(.orderPlaced(orderId: order.id, amount: order.totalAmount))
        case .auth(.loginSucceeded(let user)):
            AnalyticsService.shared.identify(userId: user.id)
        default:
            break
        }
        next(action)
    }
}
```

---

## 6. Dependency Injection Patterns

### 6.1 Factory Pattern for Testability

```swift
// DI/Factory/ViewModelFactory.swift

protocol ViewModelFactory {
    func makeProductListViewModel() -> ProductListViewModel
    func makeProductDetailViewModel(productId: String) -> ProductDetailViewModel
    func makeCartViewModel() -> CartViewModel
    func makeOrderViewModel() -> OrderViewModel
}

final class DefaultViewModelFactory: ViewModelFactory {
    private let container: DIContainer
    
    init(container: DIContainer = .shared) {
        self.container = container
    }
    
    func makeProductListViewModel() -> ProductListViewModel {
        ProductListViewModel(
            getProductsUseCase: container.resolve(GetProductsUseCase.self),
            searchProductsUseCase: container.resolve(SearchProductsUseCase.self)
        )
    }
    
    func makeProductDetailViewModel(productId: String) -> ProductDetailViewModel {
        ProductDetailViewModel(
            productId: productId,
            getProductUseCase: container.resolve(GetProductUseCase.self),
            addToCartUseCase: container.resolve(AddToCartUseCase.self)
        )
    }
    
    func makeCartViewModel() -> CartViewModel {
        CartViewModel(
            cartRepository: container.resolve(CartRepository.self),
            placeOrderUseCase: container.resolve(PlaceOrderUseCase.self)
        )
    }
    
    func makeOrderViewModel() -> OrderViewModel {
        OrderViewModel(
            orderRepository: container.resolve(OrderRepository.self)
        )
    }
}

// สำหรับ Testing
final class MockViewModelFactory: ViewModelFactory {
    func makeProductListViewModel() -> ProductListViewModel {
        ProductListViewModel(
            getProductsUseCase: GetProductsUseCase(
                productRepository: MockProductRepository()
            ),
            searchProductsUseCase: SearchProductsUseCase(
                productRepository: MockProductRepository()
            )
        )
    }
    
    func makeProductDetailViewModel(productId: String) -> ProductDetailViewModel {
        // Return mock version
        fatalError("Implement mock")
    }
    
    func makeCartViewModel() -> CartViewModel {
        fatalError("Implement mock")
    }
    
    func makeOrderViewModel() -> OrderViewModel {
        fatalError("Implement mock")
    }
}
```

---

## 7. Multi-team Development

### 7.1 Feature Flags Architecture

```swift
// FeatureFlags/FeatureFlagService.swift

import Foundation

// MARK: - Feature Flag Protocol
protocol FeatureFlagService {
    func isEnabled(_ flag: FeatureFlag) -> Bool
    func getValue<T>(_ flag: FeatureFlag, defaultValue: T) -> T
    func refresh() async throws
}

// MARK: - Feature Flags Definition
enum FeatureFlag: String, CaseIterable {
    // UI Features
    case newCheckoutFlow = "new_checkout_flow"
    case productVideoPlayer = "product_video_player"
    case ar_product_viewer = "ar_product_viewer"
    case darkModeSupport = "dark_mode_support"
    
    // Backend Features
    case v2PaymentAPI = "v2_payment_api"
    case realtimeOrderTracking = "realtime_order_tracking"
    case aiProductRecommendation = "ai_product_recommendation"
    
    // Experiment Features
    case checkoutVariantA = "checkout_variant_a"
    case priceDisplayVariant = "price_display_variant"
    
    var defaultValue: Bool {
        switch self {
        case .darkModeSupport:
            return true
        default:
            return false
        }
    }
}

// MARK: - Feature Flag Service Implementation
final class FeatureFlagServiceImpl: FeatureFlagService {
    private var flags: [String: Any] = [:]
    private let remoteConfigProvider: RemoteConfigProvider
    private let localFlagOverrides: [String: Bool]
    
    init(
        remoteConfigProvider: RemoteConfigProvider,
        localOverrides: [String: Bool] = [:]
    ) {
        self.remoteConfigProvider = remoteConfigProvider
        self.localFlagOverrides = localOverrides
        loadLocalFlags()
    }
    
    func isEnabled(_ flag: FeatureFlag) -> Bool {
        // Local override takes priority (for development)
        if let override = localFlagOverrides[flag.rawValue] {
            return override
        }
        
        // Remote flag
        if let remoteValue = flags[flag.rawValue] as? Bool {
            return remoteValue
        }
        
        // Default value
        return flag.defaultValue
    }
    
    func getValue<T>(_ flag: FeatureFlag, defaultValue: T) -> T {
        return flags[flag.rawValue] as? T ?? defaultValue
    }
    
    func refresh() async throws {
        let remoteFlags = try await remoteConfigProvider.fetchFlags()
        flags = remoteFlags
        saveLocalFlags()
    }
    
    private func loadLocalFlags() {
        if let saved = UserDefaults.standard.dictionary(forKey: "feature_flags") {
            flags = saved
        }
    }
    
    private func saveLocalFlags() {
        UserDefaults.standard.set(flags, forKey: "feature_flags")
    }
}

protocol RemoteConfigProvider {
    func fetchFlags() async throws -> [String: Any]
}

// MARK: - Feature Flag View Modifier
struct FeatureFlagModifier: ViewModifier {
    let flag: FeatureFlag
    @Environment(\.featureFlagService) var flagService
    
    func body(content: Content) -> some View {
        if flagService.isEnabled(flag) {
            content
        }
    }
}

extension View {
    func featureFlag(_ flag: FeatureFlag) -> some View {
        modifier(FeatureFlagModifier(flag: flag))
    }
}

// Environment Key
struct FeatureFlagServiceKey: EnvironmentKey {
    static var defaultValue: FeatureFlagService = FeatureFlagServiceImpl(
        remoteConfigProvider: MockRemoteConfigProvider()
    )
}

extension EnvironmentValues {
    var featureFlagService: FeatureFlagService {
        get { self[FeatureFlagServiceKey.self] }
        set { self[FeatureFlagServiceKey.self] = newValue }
    }
}

struct MockRemoteConfigProvider: RemoteConfigProvider {
    func fetchFlags() async throws -> [String: Any] { [:] }
}
```

### 7.2 A/B Testing Infrastructure

```swift
// ABTesting/ABTestingService.swift

import Foundation

struct Experiment {
    let id: String
    let name: String
    let variants: [Variant]
    
    struct Variant {
        let id: String
        let name: String
        let weight: Double  // 0.0 - 1.0
        let config: [String: Any]
    }
}

protocol ABTestingService {
    func getVariant(for experimentId: String, userId: String) -> Experiment.Variant?
    func logExposure(experimentId: String, variantId: String)
    func logConversion(experimentId: String, variantId: String, event: String)
}

final class ABTestingServiceImpl: ABTestingService {
    private var experiments: [String: Experiment] = [:]
    private var userAssignments: [String: String] = [:]  // userId_experimentId -> variantId
    private let analyticsService: AnalyticsService
    
    init(analyticsService: AnalyticsService) {
        self.analyticsService = analyticsService
        loadAssignments()
    }
    
    func getVariant(for experimentId: String, userId: String) -> Experiment.Variant? {
        guard let experiment = experiments[experimentId] else { return nil }
        
        let assignmentKey = "\(userId)_\(experimentId)"
        
        // Return existing assignment
        if let variantId = userAssignments[assignmentKey],
           let variant = experiment.variants.first(where: { $0.id == variantId }) {
            return variant
        }
        
        // Assign new variant
        let variant = assignVariant(experiment: experiment, userId: userId)
        userAssignments[assignmentKey] = variant.id
        saveAssignments()
        
        return variant
    }
    
    func logExposure(experimentId: String, variantId: String) {
        // Log to analytics
    }
    
    func logConversion(experimentId: String, variantId: String, event: String) {
        // Log to analytics
    }
    
    private func assignVariant(experiment: Experiment, userId: String) -> Experiment.Variant {
        // Deterministic assignment based on userId hash
        let hash = abs(userId.hashValue) % 100
        var cumulative = 0.0
        
        for variant in experiment.variants {
            cumulative += variant.weight * 100
            if Double(hash) < cumulative {
                return variant
            }
        }
        
        return experiment.variants.last!
    }
    
    private func loadAssignments() {
        userAssignments = UserDefaults.standard.dictionary(forKey: "ab_assignments") as? [String: String] ?? [:]
    }
    
    private func saveAssignments() {
        UserDefaults.standard.set(userAssignments, forKey: "ab_assignments")
    }
}
```

---

## 8. Performance at Scale

### 8.1 Image Caching System

```swift
// Performance/ImageCache/ImageCacheManager.swift

import UIKit
import Foundation

// MARK: - Two-tier Cache: Memory + Disk
final class ImageCacheManager {
    static let shared = ImageCacheManager()
    
    private let memoryCache: NSCache<NSString, UIImage>
    private let diskCache: DiskImageCache
    private let downloadQueue = OperationQueue()
    private var downloadTasks: [URL: Task<UIImage, Error>] = [:]
    private let lock = NSLock()
    
    init(
        memoryCapacity: Int = 50 * 1024 * 1024,  // 50MB
        diskCapacity: Int = 200 * 1024 * 1024     // 200MB
    ) {
        memoryCache = NSCache<NSString, UIImage>()
        memoryCache.totalCostLimit = memoryCapacity
        memoryCache.countLimit = 200
        
        diskCache = DiskImageCache(capacity: diskCapacity)
        
        downloadQueue.maxConcurrentOperationCount = 4
        
        setupMemoryWarningObserver()
    }
    
    // MARK: - Public API
    func image(for url: URL) async throws -> UIImage {
        let key = url.absoluteString as NSString
        
        // 1. Check memory cache
        if let cached = memoryCache.object(forKey: key) {
            return cached
        }
        
        // 2. Check disk cache
        if let diskCached = try await diskCache.read(for: url) {
            // Store in memory for faster access
            let cost = estimateCost(of: diskCached)
            memoryCache.setObject(diskCached, forKey: key, cost: cost)
            return diskCached
        }
        
        // 3. Download
        return try await download(url: url)
    }
    
    func prefetch(urls: [URL]) {
        for url in urls {
            Task {
                _ = try? await image(for: url)
            }
        }
    }
    
    func clearMemoryCache() {
        memoryCache.removeAllObjects()
    }
    
    func clearAll() async throws {
        memoryCache.removeAllObjects()
        try await diskCache.clearAll()
    }
    
    // MARK: - Private
    private func download(url: URL) async throws -> UIImage {
        // Deduplicate concurrent downloads for same URL
        lock.lock()
        if let existingTask = downloadTasks[url] {
            lock.unlock()
            return try await existingTask.value
        }
        
        let task = Task<UIImage, Error> {
            let (data, _) = try await URLSession.shared.data(from: url)
            guard let image = UIImage(data: data) else {
                throw ImageCacheError.invalidImageData
            }
            
            // Store in both caches
            let key = url.absoluteString as NSString
            let cost = self.estimateCost(of: image)
            self.memoryCache.setObject(image, forKey: key, cost: cost)
            try await self.diskCache.write(image, for: url)
            
            // Clean up task
            self.lock.lock()
            self.downloadTasks[url] = nil
            self.lock.unlock()
            
            return image
        }
        
        downloadTasks[url] = task
        lock.unlock()
        
        return try await task.value
    }
    
    private func estimateCost(of image: UIImage) -> Int {
        guard let cgImage = image.cgImage else { return 1 }
        return cgImage.bytesPerRow * cgImage.height
    }
    
    private func setupMemoryWarningObserver() {
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(handleMemoryWarning),
            name: UIApplication.didReceiveMemoryWarningNotification,
            object: nil
        )
    }
    
    @objc private func handleMemoryWarning() {
        clearMemoryCache()
    }
    
    enum ImageCacheError: Error {
        case invalidImageData
    }
}

// MARK: - Disk Cache
final class DiskImageCache {
    private let cacheDirectory: URL
    private let capacity: Int
    private let fileManager = FileManager.default
    
    init(capacity: Int) {
        self.capacity = capacity
        
        let caches = fileManager.urls(for: .cachesDirectory, in: .userDomainMask).first!
        cacheDirectory = caches.appendingPathComponent("ImageCache")
        
        try? fileManager.createDirectory(at: cacheDirectory, withIntermediateDirectories: true)
    }
    
    func read(for url: URL) async throws -> UIImage? {
        let filePath = filePath(for: url)
        guard fileManager.fileExists(atPath: filePath.path),
              let data = try? Data(contentsOf: filePath),
              let image = UIImage(data: data) else {
            return nil
        }
        
        // Update access time
        try? fileManager.setAttributes(
            [.modificationDate: Date()],
            ofItemAtPath: filePath.path
        )
        
        return image
    }
    
    func write(_ image: UIImage, for url: URL) async throws {
        guard let data = image.jpegData(compressionQuality: 0.8) else { return }
        let filePath = filePath(for: url)
        try data.write(to: filePath)
        
        await evictIfNeeded()
    }
    
    func clearAll() async throws {
        try fileManager.removeItem(at: cacheDirectory)
        try fileManager.createDirectory(at: cacheDirectory, withIntermediateDirectories: true)
    }
    
    private func filePath(for url: URL) -> URL {
        let filename = url.absoluteString
            .data(using: .utf8)!
            .base64EncodedString()
            .replacingOccurrences(of: "/", with: "_")
        return cacheDirectory.appendingPathComponent(filename)
    }
    
    private func evictIfNeeded() async {
        let files = (try? fileManager.contentsOfDirectory(
            at: cacheDirectory,
            includingPropertiesForKeys: [.fileSizeKey, .contentModificationDateKey]
        )) ?? []
        
        var totalSize = 0
        var fileInfos: [(url: URL, size: Int, date: Date)] = []
        
        for file in files {
            let size = (try? file.resourceValues(forKeys: [.fileSizeKey]))?.fileSize ?? 0
            let date = (try? file.resourceValues(forKeys: [.contentModificationDateKey]))?.contentModificationDate ?? Date.distantPast
            totalSize += size
            fileInfos.append((url: file, size: size, date: date))
        }
        
        if totalSize > capacity {
            // Sort by access date (oldest first)
            let sorted = fileInfos.sorted { $0.date < $1.date }
            
            for fileInfo in sorted {
                try? fileManager.removeItem(at: fileInfo.url)
                totalSize -= fileInfo.size
                if totalSize <= capacity * 8 / 10 { break }
            }
        }
    }
}

// MARK: - SwiftUI Image View with Caching
import SwiftUI

struct CachedAsyncImage: View {
    let url: URL?
    var placeholder: AnyView = AnyView(ProgressView())
    
    @State private var image: UIImage?
    @State private var isLoading = false
    @State private var error: Error?
    
    var body: some View {
        Group {
            if let image = image {
                Image(uiImage: image)
                    .resizable()
                    .scaledToFill()
            } else if isLoading {
                placeholder
            } else if error != nil {
                Image(systemName: "photo.fill")
                    .foregroundColor(.gray)
            }
        }
        .task {
            await loadImage()
        }
    }
    
    private func loadImage() async {
        guard let url = url else { return }
        isLoading = true
        
        do {
            image = try await ImageCacheManager.shared.image(for: url)
        } catch {
            self.error = error
        }
        
        isLoading = false
    }
}
```

---

## 9. Crash-resilient Architecture

### 9.1 Error Boundary Patterns

```swift
// CrashResilience/ErrorBoundary.swift

import SwiftUI

// MARK: - Error Boundary สำหรับ SwiftUI
struct ErrorBoundary<Content: View, Fallback: View>: View {
    let content: () throws -> Content
    let fallback: (Error) -> Fallback
    
    @State private var error: Error?
    
    var body: some View {
        if let error = error {
            fallback(error)
        } else {
            do {
                try content()
            } catch {
                // SwiftUI doesn't support throwing in body directly
                // Use SafeView approach instead
                EmptyView()
            }
        }
    }
}

// MARK: - Safe ViewModel Wrapper
@MainActor
final class SafeViewModel<VM: ObservableObject>: ObservableObject {
    @Published private(set) var viewModel: VM?
    @Published private(set) var error: Error?
    
    private let factory: () throws -> VM
    
    init(factory: @escaping () throws -> VM) {
        self.factory = factory
        initialize()
    }
    
    func initialize() {
        do {
            viewModel = try factory()
            error = nil
        } catch {
            self.error = error
        }
    }
    
    func retry() {
        initialize()
    }
}

// MARK: - Graceful Degradation
protocol Degradable {
    var degradedState: Self { get }
    var isDegraded: Bool { get }
}

extension ProductsState: Degradable {
    var degradedState: ProductsState {
        var state = self
        state.items = []
        state.error = nil
        state.isLoading = false
        return state
    }
    
    var isDegraded: Bool {
        error != nil
    }
}

// MARK: - Recovery Strategies
enum RecoveryStrategy {
    case retry(maxAttempts: Int, delay: Double)
    case fallbackToCache
    case showOfflineMode
    case relaunchFeature
    case notifyUserAndContinue(message: String)
}

protocol RecoverableError: Error {
    var recoveryStrategies: [RecoveryStrategy] { get }
}

// MARK: - Crash Reporter Integration
protocol CrashReporter {
    func record(_ error: Error, context: [String: Any]?)
    func recordFatal(_ error: Error)
    func setUserContext(userId: String?, email: String?)
}

final class CrashReporterService: CrashReporter {
    static let shared = CrashReporterService()
    
    private var isEnabled = true
    private var breadcrumbs: [Breadcrumb] = []
    
    struct Breadcrumb {
        let timestamp: Date
        let category: String
        let message: String
        let data: [String: Any]?
    }
    
    func addBreadcrumb(category: String, message: String, data: [String: Any]? = nil) {
        let breadcrumb = Breadcrumb(
            timestamp: Date(),
            category: category,
            message: message,
            data: data
        )
        breadcrumbs.append(breadcrumb)
        
        // Keep only last 50 breadcrumbs
        if breadcrumbs.count > 50 {
            breadcrumbs.removeFirst()
        }
    }
    
    func record(_ error: Error, context: [String: Any]? = nil) {
        guard isEnabled else { return }
        
        var fullContext = context ?? [:]
        fullContext["breadcrumbs"] = breadcrumbs.map { "\($0.timestamp): [\($0.category)] \($0.message)" }
        
        // Send to crash reporting service (Crashlytics, Sentry, etc.)
        print("📛 Error recorded: \(error.localizedDescription)")
    }
    
    func recordFatal(_ error: Error) {
        record(error, context: ["fatal": true])
    }
    
    func setUserContext(userId: String?, email: String?) {
        // Set user context in crash reporting service
    }
}
```

---

## 10. Analytics Architecture

### 10.1 Analytics Abstraction Layer

```swift
// Analytics/AnalyticsService.swift

import Foundation

// MARK: - Analytics Event Protocol
protocol AnalyticsEvent {
    var name: String { get }
    var properties: [String: Any] { get }
}

// MARK: - Domain-specific Events
enum ECommerceEvent: AnalyticsEvent {
    case productViewed(productId: String, productName: String, price: Decimal)
    case productAddedToCart(productId: String, quantity: Int)
    case checkoutStarted(cartValue: Decimal, itemCount: Int)
    case orderPlaced(orderId: String, amount: Decimal, paymentMethod: String)
    case orderCancelled(orderId: String, reason: String)
    case searchPerformed(query: String, resultsCount: Int)
    case filterApplied(filterType: String, filterValue: String)
    
    var name: String {
        switch self {
        case .productViewed: return "product_viewed"
        case .productAddedToCart: return "product_added_to_cart"
        case .checkoutStarted: return "checkout_started"
        case .orderPlaced: return "order_placed"
        case .orderCancelled: return "order_cancelled"
        case .searchPerformed: return "search_performed"
        case .filterApplied: return "filter_applied"
        }
    }
    
    var properties: [String: Any] {
        switch self {
        case .productViewed(let id, let name, let price):
            return ["product_id": id, "product_name": name, "price": NSDecimalNumber(decimal: price).doubleValue]
        case .productAddedToCart(let id, let qty):
            return ["product_id": id, "quantity": qty]
        case .checkoutStarted(let value, let count):
            return ["cart_value": NSDecimalNumber(decimal: value).doubleValue, "item_count": count]
        case .orderPlaced(let orderId, let amount, let method):
            return ["order_id": orderId, "amount": NSDecimalNumber(decimal: amount).doubleValue, "payment_method": method]
        case .orderCancelled(let orderId, let reason):
            return ["order_id": orderId, "reason": reason]
        case .searchPerformed(let query, let count):
            return ["query": query, "results_count": count]
        case .filterApplied(let type, let value):
            return ["filter_type": type, "filter_value": value]
        }
    }
}

// MARK: - Analytics Service
final class AnalyticsService {
    static let shared = AnalyticsService()
    
    private var providers: [AnalyticsProvider] = []
    private var globalProperties: [String: Any] = [:]
    private var userId: String?
    
    private let eventQueue = DispatchQueue(label: "com.app.analytics", qos: .utility)
    private var eventBuffer: [AnalyticsEvent] = []
    private let bufferSize = 20
    
    private init() {}
    
    func configure(providers: [AnalyticsProvider]) {
        self.providers = providers
    }
    
    func identify(userId: String, traits: [String: Any] = [:]) {
        self.userId = userId
        providers.forEach { $0.identify(userId: userId, traits: traits) }
    }
    
    func track(_ event: AnalyticsEvent) {
        eventQueue.async {
            var enrichedProperties = event.properties
            enrichedProperties.merge(self.globalProperties) { current, _ in current }
            
            if let userId = self.userId {
                enrichedProperties["user_id"] = userId
            }
            
            enrichedProperties["timestamp"] = ISO8601DateFormatter().string(from: Date())
            enrichedProperties["platform"] = "iOS"
            enrichedProperties["app_version"] = Bundle.main.infoDictionary?["CFBundleShortVersionString"] as? String ?? ""
            
            self.providers.forEach { provider in
                provider.track(event: event.name, properties: enrichedProperties)
            }
        }
    }
    
    func setGlobalProperty(_ key: String, value: Any) {
        globalProperties[key] = value
    }
    
    func screen(_ name: String, properties: [String: Any] = [:]) {
        providers.forEach { $0.screen(name: name, properties: properties) }
    }
    
    func reset() {
        userId = nil
        providers.forEach { $0.reset() }
    }
}

// MARK: - Analytics Provider Protocol
protocol AnalyticsProvider {
    func track(event: String, properties: [String: Any])
    func identify(userId: String, traits: [String: Any])
    func screen(name: String, properties: [String: Any])
    func reset()
}

// Firebase Analytics Provider
struct FirebaseAnalyticsProvider: AnalyticsProvider {
    func track(event: String, properties: [String: Any]) {
        // Firebase.Analytics.logEvent(event, parameters: properties)
        print("Firebase: \(event) - \(properties)")
    }
    
    func identify(userId: String, traits: [String: Any]) {
        // Firebase.Analytics.setUserID(userId)
    }
    
    func screen(name: String, properties: [String: Any]) {
        // Firebase.Analytics.logEvent(AnalyticsEventScreenView, ...)
    }
    
    func reset() {
        // Firebase.Analytics.resetAnalyticsData()
    }
}

// Custom Analytics DSL
struct Analytics {
    static func track(_ event: ECommerceEvent) {
        AnalyticsService.shared.track(event)
    }
}

// Usage Example:
// Analytics.track(.productViewed(productId: "123", productName: "iPhone", price: 35000))
// Analytics.track(.orderPlaced(orderId: "ORD-001", amount: 35000, paymentMethod: "credit_card"))
```

---

## 11. Complete Enterprise App Example: Banking App

```swift
// BankingApp/Domain/Entities/Account.swift

struct BankAccount: Identifiable, Equatable {
    let id: String
    let accountNumber: String
    let accountType: AccountType
    var balance: Decimal
    var availableBalance: Decimal
    var currency: Currency
    let ownerName: String
    var isActive: Bool
    
    enum AccountType: String {
        case savings = "savings"        // บัญชีออมทรัพย์
        case current = "current"        // บัญชีกระแสรายวัน
        case fixedDeposit = "fixed"     // บัญชีฝากประจำ
        case investment = "investment"   // บัญชีลงทุน
    }
    
    enum Currency: String {
        case thb = "THB"
        case usd = "USD"
        case eur = "EUR"
    }
}

struct Transaction: Identifiable {
    let id: String
    let accountId: String
    let type: TransactionType
    let amount: Decimal
    let currency: BankAccount.Currency
    let description: String
    let referenceNumber: String
    let timestamp: Date
    var status: TransactionStatus
    var balanceAfter: Decimal
    
    enum TransactionType {
        case debit   // เงินออก
        case credit  // เงินเข้า
        case transfer
        case fee
        case interest
    }
    
    enum TransactionStatus {
        case pending
        case completed
        case failed
        case reversed
    }
}

struct TransferRequest {
    let fromAccountId: String
    let toAccountNumber: String
    let amount: Decimal
    let currency: BankAccount.Currency
    let note: String?
    let scheduledDate: Date?
}

// MARK: - Banking Use Cases
final class TransferFundsUseCase {
    private let accountRepository: BankAccountRepository
    private let transactionRepository: TransactionRepository
    private let notificationService: NotificationService
    private let fraudDetectionService: FraudDetectionService
    
    init(
        accountRepository: BankAccountRepository,
        transactionRepository: TransactionRepository,
        notificationService: NotificationService,
        fraudDetectionService: FraudDetectionService
    ) {
        self.accountRepository = accountRepository
        self.transactionRepository = transactionRepository
        self.notificationService = notificationService
        self.fraudDetectionService = fraudDetectionService
    }
    
    func execute(_ request: TransferRequest) async throws -> Transaction {
        // 1. Validate accounts
        let fromAccount = try await accountRepository.getAccount(id: request.fromAccountId)
        
        guard fromAccount.isActive else {
            throw TransferError.accountInactive
        }
        
        guard fromAccount.availableBalance >= request.amount else {
            throw TransferError.insufficientFunds(
                available: fromAccount.availableBalance,
                requested: request.amount
            )
        }
        
        // 2. Fraud detection
        let fraudScore = await fraudDetectionService.assess(request)
        guard fraudScore < 0.7 else {
            throw TransferError.suspiciousActivity
        }
        
        // 3. Execute transfer
        let transaction = try await transactionRepository.createTransfer(request)
        
        // 4. Send notification
        await notificationService.sendTransferNotification(transaction)
        
        return transaction
    }
    
    enum TransferError: LocalizedError {
        case accountInactive
        case insufficientFunds(available: Decimal, requested: Decimal)
        case suspiciousActivity
        case dailyLimitExceeded
        case recipientNotFound
        
        var errorDescription: String? {
            switch self {
            case .accountInactive:
                return "บัญชีถูกระงับการใช้งาน"
            case .insufficientFunds(let available, _):
                return "ยอดเงินคงเหลือไม่เพียงพอ (คงเหลือ: \(available))"
            case .suspiciousActivity:
                return "ตรวจพบกิจกรรมที่น่าสงสัย กรุณาติดต่อธนาคาร"
            case .dailyLimitExceeded:
                return "เกินวงเงินโอนต่อวัน"
            case .recipientNotFound:
                return "ไม่พบบัญชีปลายทาง"
            }
        }
    }
}

// MARK: - Banking ViewModel
@MainActor
final class BankingHomeViewModel: ObservableObject {
    @Published private(set) var accounts: [BankAccount] = []
    @Published private(set) var recentTransactions: [Transaction] = []
    @Published private(set) var totalBalance: Decimal = 0
    @Published private(set) var isLoading = false
    @Published var error: Error?
    
    private let getAccountsUseCase: GetAccountsUseCase
    private let getTransactionsUseCase: GetTransactionsUseCase
    
    init(
        getAccountsUseCase: GetAccountsUseCase,
        getTransactionsUseCase: GetTransactionsUseCase
    ) {
        self.getAccountsUseCase = getAccountsUseCase
        self.getTransactionsUseCase = getTransactionsUseCase
    }
    
    func loadDashboard() async {
        isLoading = true
        
        do {
            async let accountsTask = getAccountsUseCase.execute()
            async let transactionsTask = getTransactionsUseCase.execute(limit: 10)
            
            let (loadedAccounts, transactions) = try await (accountsTask, transactionsTask)
            
            accounts = loadedAccounts
            recentTransactions = transactions
            totalBalance = loadedAccounts.reduce(Decimal.zero) { $0 + $1.balance }
        } catch {
            self.error = error
        }
        
        isLoading = false
    }
}

// Placeholder protocols for banking
protocol BankAccountRepository {
    func getAccount(id: String) async throws -> BankAccount
    func getAccounts() async throws -> [BankAccount]
}

protocol TransactionRepository {
    func createTransfer(_ request: TransferRequest) async throws -> Transaction
    func getTransactions(accountId: String, limit: Int) async throws -> [Transaction]
}

protocol FraudDetectionService {
    func assess(_ request: TransferRequest) async -> Double
}

protocol NotificationService {
    func sendTransferNotification(_ transaction: Transaction) async
}

final class GetAccountsUseCase {
    private let repository: BankAccountRepository
    init(repository: BankAccountRepository) { self.repository = repository }
    func execute() async throws -> [BankAccount] { try await repository.getAccounts() }
}

final class GetTransactionsUseCase {
    private let repository: TransactionRepository
    private let accountId: String
    init(repository: TransactionRepository, accountId: String = "") {
        self.repository = repository
        self.accountId = accountId
    }
    func execute(limit: Int = 20) async throws -> [Transaction] {
        try await repository.getTransactions(accountId: accountId, limit: limit)
    }
}
```

---

## 12. Team Structure and Code Ownership

```swift
// CODEOWNERS สำหรับ GitHub (ไม่ใช่ Swift แต่สำคัญมาก)
/*
# CODEOWNERS

# Global ownership
* @lead-architect

# Feature teams
/Packages/FeatureAuth/      @auth-team
/Packages/FeatureHome/      @home-team
/Packages/FeatureCart/      @cart-team
/Packages/FeatureProfile/   @profile-team

# Core infrastructure
/Packages/SharedKernel/     @core-team @lead-architect
/Packages/NetworkKit/       @core-team
/Packages/DesignSystem/     @design-team

# CI/CD
/.github/                   @devops-team
/fastlane/                  @devops-team
*/

// MARK: - Code Review Guidelines ใน Swift Comment
/**
 # Code Review Checklist สำหรับ Enterprise iOS
 
 ## Architecture
 - [ ] Code อยู่ใน layer ที่ถูกต้อง (Domain/Data/Presentation)
 - [ ] ไม่มี business logic ใน View
 - [ ] ไม่มี UI code ใน ViewModel/UseCase
 - [ ] Dependencies ชี้เข้าหา Domain layer
 
 ## Code Quality
 - [ ] Method ไม่ยาวเกิน 30 บรรทัด
 - [ ] Function มี single responsibility
 - [ ] ไม่มี force unwrap (!!) นอกจาก IBOutlets
 - [ ] Error handling ครบถ้วน
 
 ## Testing
 - [ ] Unit tests ครอบคลุม Use Cases
 - [ ] ViewModel tests มีครบ
 - [ ] Mock objects ถูกต้อง
 
 ## Performance
 - [ ] ไม่มี blocking calls บน main thread
 - [ ] Images ถูก cache อย่างถูกต้อง
 - [ ] Memory leaks ได้รับการตรวจสอบ
 
 ## Security
 - [ ] Sensitive data ไม่ถูก log
 - [ ] API keys ไม่ hardcode
 - [ ] User input ได้รับการ validate
 */
```

---

## 13. Documentation Standards สำหรับ Large Teams

```swift
// Documentation/DocumentationStandards.swift

/**
 # ProductRepository
 
 Repository สำหรับจัดการข้อมูล Product ระหว่าง application และแหล่งข้อมูล
 
 ## Overview
 `ProductRepository` เป็น protocol ที่กำหนด interface สำหรับการ access ข้อมูล Product
 Implementation อาจเป็น remote API, local database, หรือ combination ของทั้งสอง
 
 ## Usage
 ```swift
 let repository = container.resolve(ProductRepository.self)
 let products = try await repository.getProducts(category: .electronics, page: 1, pageSize: 20)
 ```
 
 ## Threading
 Method ทั้งหมดเป็น async/await และ thread-safe
 
 ## Error Handling
 Throws `RepositoryError` เมื่อเกิดข้อผิดพลาด
 
 - Important: ไม่ควร implement ใน Presentation layer โดยตรง
 - Note: Implementation ควรมี caching strategy
 - Version: 2.0
 - Since: iOS 16
 */

// MARK: - Inline Documentation
extension ProductRepository {
    /// ดึง product ตาม ID
    ///
    /// - Parameter id: Unique identifier ของ product
    /// - Returns: `Product` object ที่ตรงกับ ID
    /// - Throws: `RepositoryError.notFound` ถ้าไม่พบ product
    /// - Throws: `RepositoryError.networkError` ถ้าเกิดข้อผิดพลาดเครือข่าย
    ///
    /// ## Example
    /// ```swift
    /// let product = try await repository.getProduct(id: "PROD-123")
    /// print(product.name) // "iPhone 15 Pro"
    /// ```
    func getProduct(id: String) async throws -> Product
}

enum RepositoryError: Error {
    case notFound
    case networkError(Error)
    case invalidData
    case unauthorized
}
```

---

## 14. แบบฝึกหัดและเฉลย

### แบบฝึกหัดที่ 1: สร้าง Use Case สำหรับ Wishlist

**โจทย์:** สร้าง `AddToWishlistUseCase` และ `GetWishlistUseCase` พร้อม Repository protocol

**เฉลย:**

```swift
// Exercise 1 Solution

// Wishlist Entity
struct WishlistItem: Identifiable, Equatable {
    let id: String
    let userId: String
    let product: Product
    let addedAt: Date
}

// Repository Protocol
protocol WishlistRepository {
    func getWishlist(userId: String) async throws -> [WishlistItem]
    func addItem(userId: String, productId: String) async throws -> WishlistItem
    func removeItem(userId: String, itemId: String) async throws
    func isInWishlist(userId: String, productId: String) async throws -> Bool
}

// Add to Wishlist Use Case
final class AddToWishlistUseCase {
    private let wishlistRepository: WishlistRepository
    private let productRepository: ProductRepository
    
    init(
        wishlistRepository: WishlistRepository,
        productRepository: ProductRepository
    ) {
        self.wishlistRepository = wishlistRepository
        self.productRepository = productRepository
    }
    
    struct Input {
        let userId: String
        let productId: String
    }
    
    func execute(_ input: Input) async throws -> WishlistItem {
        // Check if already in wishlist
        let alreadyAdded = try await wishlistRepository.isInWishlist(
            userId: input.userId,
            productId: input.productId
        )
        
        if alreadyAdded {
            throw WishlistError.alreadyInWishlist
        }
        
        // Verify product exists
        _ = try await productRepository.getProduct(id: input.productId)
        
        return try await wishlistRepository.addItem(
            userId: input.userId,
            productId: input.productId
        )
    }
    
    enum WishlistError: LocalizedError {
        case alreadyInWishlist
        
        var errorDescription: String? {
            "สินค้านี้อยู่ใน Wishlist แล้ว"
        }
    }
}

// Get Wishlist Use Case
final class GetWishlistUseCase {
    private let wishlistRepository: WishlistRepository
    
    init(wishlistRepository: WishlistRepository) {
        self.wishlistRepository = wishlistRepository
    }
    
    func execute(userId: String) async throws -> [WishlistItem] {
        return try await wishlistRepository.getWishlist(userId: userId)
    }
}

// ViewModel
@MainActor
final class WishlistViewModel: ObservableObject {
    @Published private(set) var items: [WishlistItem] = []
    @Published private(set) var isLoading = false
    @Published var error: Error?
    
    private let getWishlistUseCase: GetWishlistUseCase
    private let addToWishlistUseCase: AddToWishlistUseCase
    private let userId: String
    
    init(
        getWishlistUseCase: GetWishlistUseCase,
        addToWishlistUseCase: AddToWishlistUseCase,
        userId: String
    ) {
        self.getWishlistUseCase = getWishlistUseCase
        self.addToWishlistUseCase = addToWishlistUseCase
        self.userId = userId
    }
    
    func loadWishlist() async {
        isLoading = true
        do {
            items = try await getWishlistUseCase.execute(userId: userId)
        } catch {
            self.error = error
        }
        isLoading = false
    }
    
    func addToWishlist(productId: String) async {
        do {
            let item = try await addToWishlistUseCase.execute(
                .init(userId: userId, productId: productId)
            )
            items.append(item)
        } catch {
            self.error = error
        }
    }
}
```

### แบบฝึกหัดที่ 2: Implement Feature Flag Toggle UI

**โจทย์:** สร้าง Settings screen ที่ Developer สามารถ toggle feature flags ได้ (Debug only)

**เฉลย:**

```swift
// Exercise 2 Solution

#if DEBUG
import SwiftUI

struct FeatureFlagSettingsView: View {
    @State private var flagStates: [String: Bool] = [:]
    
    var body: some View {
        NavigationView {
            List {
                ForEach(FeatureFlag.allCases, id: \.rawValue) { flag in
                    Toggle(flag.rawValue, isOn: Binding(
                        get: { flagStates[flag.rawValue] ?? flag.defaultValue },
                        set: { newValue in
                            flagStates[flag.rawValue] = newValue
                            saveOverride(flag: flag, value: newValue)
                        }
                    ))
                }
            }
            .navigationTitle("Feature Flags (Debug)")
        }
        .onAppear {
            loadCurrentStates()
        }
    }
    
    private func loadCurrentStates() {
        for flag in FeatureFlag.allCases {
            flagStates[flag.rawValue] = UserDefaults.standard.bool(
                forKey: "ff_override_\(flag.rawValue)"
            )
        }
    }
    
    private func saveOverride(flag: FeatureFlag, value: Bool) {
        UserDefaults.standard.set(value, forKey: "ff_override_\(flag.rawValue)")
    }
}
#endif
```

### แบบฝึกหัดที่ 3: สร้าง Analytics DSL

**โจทย์:** สร้าง Analytics DSL ที่ใช้งานง่ายและ type-safe

**เฉลย:**

```swift
// Exercise 3 Solution

// Result Builder สำหรับ Analytics Properties
@resultBuilder
struct AnalyticsPropertiesBuilder {
    static func buildBlock(_ properties: AnalyticsProperty...) -> [String: Any] {
        properties.reduce(into: [:]) { result, property in
            result[property.key] = property.value
        }
    }
}

struct AnalyticsProperty {
    let key: String
    let value: Any
    
    static func property(_ key: String, _ value: Any) -> AnalyticsProperty {
        AnalyticsProperty(key: key, value: value)
    }
}

// DSL Usage
struct TrackableEvent {
    let name: String
    let properties: [String: Any]
    
    init(_ name: String, @AnalyticsPropertiesBuilder properties: () -> [String: Any]) {
        self.name = name
        self.properties = properties()
    }
}

// Usage:
// let event = TrackableEvent("purchase_completed") {
//     AnalyticsProperty.property("order_id", "ORD-123")
//     AnalyticsProperty.property("amount", 35000.0)
//     AnalyticsProperty.property("items_count", 3)
// }
// AnalyticsService.shared.track(event)
```

---

## 15. สรุปและแนวทางต่อไป

การออกแบบสถาปัตยกรรม Enterprise iOS ที่ดีต้องคำนึงถึง:

1. **Separation of Concerns** - แต่ละ layer มีหน้าที่ชัดเจน
2. **Testability** - ทุก component สามารถ test ได้โดยอิสระ
3. **Scalability** - รองรับการเติบโตของทีมและ codebase
4. **Maintainability** - Code อ่านง่าย แก้ไขได้โดยไม่กระทบส่วนอื่น
5. **Performance** - ตอบสนองได้เร็ว ใช้ทรัพยากรอย่างมีประสิทธิภาพ

### สิ่งที่ควรเรียนรู้เพิ่มเติม:
- Swift Concurrency (async/await, Actor)
- Combine Framework
- Swift Package Manager
- XCTest และ UI Testing
- CI/CD กับ Fastlane และ GitHub Actions
- Performance Profiling ด้วย Instruments

---

*จบ Part 81: Enterprise iOS Architecture*
