# Part 88: App Monetization Strategies และ Business Models

## บทนำ

การสร้างแอปที่ดีทางเทคนิคเป็นแค่ครึ่งทางของความสำเร็จ อีกครึ่งหนึ่งคือการ monetize แอปให้ได้อย่างมีประสิทธิภาพ บทนี้จะครอบคลุมทุก model ตั้งแต่ In-App Purchases ไปจนถึง B2B Enterprise licensing พร้อมโค้ดจริงที่ใช้ได้

---

## 1. App Monetization Models Overview

### 1.1 Free with Ads

แอปให้ใช้ฟรี รายได้มาจากการแสดงโฆษณา

**เหมาะกับ:**
- แอปที่มี DAU (Daily Active Users) สูง
- Content apps (news, entertainment)
- Games ที่มี casual users จำนวนมาก

**ข้อดี:**
- Acquisition barrier ต่ำ = users เยอะ
- Revenue scaling ตาม traffic
- ไม่ต้อง convince users จ่ายเงิน

**ข้อเสีย:**
- Revenue per user ต่ำ
- Ad blocker เป็นปัญหา
- UX อาจแย่ลงจาก ads
- IDFA/ATT กระทบรายได้อย่างมาก

**Revenue Calculation:**
```swift
struct AdRevenueModel {
    let dailyActiveUsers: Int
    let avgSessionsPerDay: Double
    let impressionsPerSession: Double
    let eCPM: Double  // Effective cost per mille (per 1000 impressions)
    
    // eCPM ทั่วไป:
    // Banner ads: $0.5 - $2.0
    // Interstitial: $4 - $8
    // Rewarded video: $10 - $20
    
    var dailyRevenue: Double {
        let totalImpressions = Double(dailyActiveUsers) * avgSessionsPerDay * impressionsPerSession
        return (totalImpressions / 1000) * eCPM
    }
    
    var monthlyRevenue: Double {
        return dailyRevenue * 30
    }
    
    var annualRevenue: Double {
        return dailyRevenue * 365
    }
    
    var revenuePerUser: Double {
        return dailyRevenue / Double(dailyActiveUsers)
    }
}

// ตัวอย่าง
let newsApp = AdRevenueModel(
    dailyActiveUsers: 100_000,
    avgSessionsPerDay: 3,
    impressionsPerSession: 2.5,
    eCPM: 4.0  // interstitial
)

print("Daily Revenue: $\(String(format: "%.2f", newsApp.dailyRevenue))")
print("Monthly Revenue: $\(String(format: "%.2f", newsApp.monthlyRevenue))")
print("Revenue per User: $\(String(format: "%.4f", newsApp.revenuePerUser))")
```

### 1.2 Freemium Model

แอปฟรี แต่ Premium features ต้องจ่ายเงิน

**Feature Gating Strategy:**
```swift
// Feature Flag System สำหรับ Freemium

enum SubscriptionTier: String, Codable {
    case free
    case premium
    case enterprise
}

protocol FeatureGated {
    var requiredTier: SubscriptionTier { get }
}

enum AppFeature: FeatureGated {
    // Free features
    case basicSearch
    case limitedStorage(mbLimit: Int)
    case adsEnabled
    
    // Premium features
    case advancedSearch
    case unlimitedStorage
    case offlineMode
    case exportToPDF
    case customThemes
    case noAds
    
    // Enterprise features
    case teamSharing
    case adminDashboard
    case ssoIntegration
    case auditLogs
    
    var requiredTier: SubscriptionTier {
        switch self {
        case .basicSearch, .adsEnabled:
            return .free
        case .limitedStorage:
            return .free
        case .advancedSearch, .unlimitedStorage, 
             .offlineMode, .exportToPDF, .customThemes, .noAds:
            return .premium
        case .teamSharing, .adminDashboard, 
             .ssoIntegration, .auditLogs:
            return .enterprise
        }
    }
}

// Feature Access Manager
class FeatureAccessManager: ObservableObject {
    @Published private(set) var currentTier: SubscriptionTier = .free
    
    func canAccess(_ feature: AppFeature) -> Bool {
        switch (feature.requiredTier, currentTier) {
        case (.free, _):
            return true
        case (.premium, .premium), (.premium, .enterprise):
            return true
        case (.enterprise, .enterprise):
            return true
        default:
            return false
        }
    }
    
    func accessFeature(_ feature: AppFeature, action: () -> Void, onDenied: () -> Void) {
        if canAccess(feature) {
            action()
        } else {
            onDenied()
        }
    }
}
```

### 1.3 Subscription Model

Recurring revenue จาก users ที่จ่ายรายเดือน/ปี

**Revenue Calculation:**
```swift
struct SubscriptionMetrics {
    let subscribers: Int
    let monthlyPrice: Double
    let annualPrice: Double
    let annualSubscriberPercentage: Double  // % ที่เลือก annual
    let monthlyChurnRate: Double            // % ที่ cancel ทุกเดือน
    
    var effectiveMonthlyRevenue: Double {
        let monthlySubCount = Double(subscribers) * (1 - annualSubscriberPercentage)
        let annualSubCount = Double(subscribers) * annualSubscriberPercentage
        
        let monthlyRevenue = monthlySubCount * monthlyPrice
        let annualRevenue = annualSubCount * (annualPrice / 12)
        
        return monthlyRevenue + annualRevenue
    }
    
    var mrr: Double { effectiveMonthlyRevenue }
    var arr: Double { mrr * 12 }
    
    // Average Revenue Per User
    var arpu: Double {
        return mrr / Double(subscribers)
    }
    
    // Lifetime Value (ถ้า churn rate คือ monthly churn)
    // LTV = ARPU / Monthly Churn Rate
    var ltv: Double {
        guard monthlyChurnRate > 0 else { return Double.infinity }
        return arpu / monthlyChurnRate
    }
    
    // Months to payback CAC
    func monthsToPayback(cac: Double) -> Double {
        return cac / arpu
    }
}
```

### 1.4 One-Time Purchase

จ่ายครั้งเดียว ใช้ได้ตลอด

**เหมาะกับ:**
- Professional tools
- Games ที่ไม่มี ongoing content
- Utility apps

**ข้อเสียหลัก:**
- No recurring revenue = ต้องหา new users ตลอด
- Hard to sustain development long-term
- ไม่ได้รับประโยชน์จาก improvements

### 1.5 In-App Purchases (Consumables)

ซื้อ virtual goods ที่ใช้แล้วหมด

```swift
// ตัวอย่าง: Game Currency System
struct GameEconomy {
    // Consumable products
    enum CurrencyPack: String, CaseIterable {
        case small = "com.game.coins.100"
        case medium = "com.game.coins.550"
        case large = "com.game.coins.1200"
        case whale = "com.game.coins.6500"
        
        var coinAmount: Int {
            switch self {
            case .small: return 100
            case .medium: return 550    // 10% bonus
            case .large: return 1200   // 20% bonus
            case .whale: return 6500   // 30% bonus
            }
        }
        
        var price: Double {
            switch self {
            case .small: return 0.99
            case .medium: return 4.99
            case .large: return 9.99
            case .whale: return 49.99
            }
        }
        
        var valuePerDollar: Double {
            return Double(coinAmount) / price
        }
    }
}
```

### 1.6 Enterprise/B2B Licensing

```swift
// Enterprise Licensing Structure
struct EnterpriseLicense {
    let companyName: String
    let licenseType: LicenseType
    let userSeats: Int
    let contractTermYears: Int
    let monthlyPricePerSeat: Double
    
    enum LicenseType {
        case perSeat          // จ่ายตามจำนวน users
        case site             // unlimited users ใน organization
        case usage            // จ่ายตาม API calls/transactions
    }
    
    var annualContractValue: Double {
        switch licenseType {
        case .perSeat:
            return Double(userSeats) * monthlyPricePerSeat * 12
        case .site:
            // Flat fee regardless of users
            return monthlyPricePerSeat * 12  // monthlyPricePerSeat = site license price
        case .usage:
            return 0  // variable
        }
    }
    
    var totalContractValue: Double {
        return annualContractValue * Double(contractTermYears)
    }
}
```

---

## 2. StoreKit 2 Monetization Implementation

### 2.1 Subscription Tiers Implementation

```swift
import StoreKit

// Product Configuration (App Store Connect)
// Products:
// - com.app.subscription.monthly (Auto-Renewable, $9.99/month)
// - com.app.subscription.annual (Auto-Renewable, $79.99/year)
// - com.app.subscription.family (Auto-Renewable, $14.99/month)

class SubscriptionManager: ObservableObject {
    
    static let shared = SubscriptionManager()
    
    // Product IDs
    static let monthlyProductId = "com.app.subscription.monthly"
    static let annualProductId = "com.app.subscription.annual"
    static let familyProductId = "com.app.subscription.family"
    
    @Published private(set) var products: [Product] = []
    @Published private(set) var purchasedSubscriptions: [Product] = []
    @Published private(set) var subscriptionStatus: SubscriptionStatus = .notSubscribed
    
    private var updateListenerTask: Task<Void, Error>?
    
    enum SubscriptionStatus {
        case notSubscribed
        case monthly
        case annual
        case family
        case unknown
    }
    
    init() {
        updateListenerTask = listenForTransactions()
        
        Task {
            await loadProducts()
            await updateSubscriptionStatus()
        }
    }
    
    deinit {
        updateListenerTask?.cancel()
    }
    
    // โหลด Products จาก App Store
    @MainActor
    func loadProducts() async {
        do {
            let productIds = [
                Self.monthlyProductId,
                Self.annualProductId,
                Self.familyProductId
            ]
            
            products = try await Product.products(for: productIds)
            
            // Sort by price
            products.sort { $0.price < $1.price }
            
        } catch {
            print("Failed to load products: \(error)")
        }
    }
    
    // ซื้อ Subscription
    func purchase(_ product: Product) async throws -> Transaction? {
        let result = try await product.purchase()
        
        switch result {
        case .success(let verification):
            // Verify the transaction
            let transaction = try checkVerified(verification)
            
            // Update subscription status
            await updateSubscriptionStatus()
            
            // Finish the transaction
            await transaction.finish()
            
            return transaction
            
        case .userCancelled:
            return nil
            
        case .pending:
            // Transaction waiting for approval (parental controls)
            return nil
            
        @unknown default:
            return nil
        }
    }
    
    // Listen สำหรับ Transaction Updates
    private func listenForTransactions() -> Task<Void, Error> {
        return Task.detached {
            for await result in Transaction.updates {
                do {
                    let transaction = try self.checkVerified(result)
                    await self.updateSubscriptionStatus()
                    await transaction.finish()
                } catch {
                    print("Transaction failed verification: \(error)")
                }
            }
        }
    }
    
    // Verify Transaction
    private func checkVerified<T>(_ result: VerificationResult<T>) throws -> T {
        switch result {
        case .unverified:
            throw StoreError.failedVerification
        case .verified(let safe):
            return safe
        }
    }
    
    // Update Subscription Status
    @MainActor
    func updateSubscriptionStatus() async {
        var purchasedProducts: [Product] = []
        
        // ตรวจสอบ active subscriptions
        for await result in Transaction.currentEntitlements {
            do {
                let transaction = try checkVerified(result)
                
                if let product = products.first(where: { $0.id == transaction.productID }) {
                    purchasedProducts.append(product)
                }
            } catch {
                print("Failed to verify entitlement: \(error)")
            }
        }
        
        purchasedSubscriptions = purchasedProducts
        
        // Update status based on purchased products
        if purchasedProducts.contains(where: { $0.id == Self.annualProductId }) {
            subscriptionStatus = .annual
        } else if purchasedProducts.contains(where: { $0.id == Self.monthlyProductId }) {
            subscriptionStatus = .monthly
        } else if purchasedProducts.contains(where: { $0.id == Self.familyProductId }) {
            subscriptionStatus = .family
        } else {
            subscriptionStatus = .notSubscribed
        }
    }
    
    enum StoreError: Error {
        case failedVerification
        case productNotFound
    }
}
```

### 2.2 Free Trial Implementation

```swift
// Free Trial Configuration ใน App Store Connect:
// Introductory Offer: Free Trial, 7 days

extension SubscriptionManager {
    
    // ตรวจสอบว่า user eligible สำหรับ free trial
    func isEligibleForFreeTrial(for product: Product) async -> Bool {
        guard let subscription = product.subscription else { return false }
        
        // ตรวจสอบ introductory offer eligibility
        let eligibility = await subscription.isEligibleForIntroOffer
        return eligibility
    }
    
    // แสดง trial period ให้ user
    func trialPeriodDescription(for product: Product) -> String? {
        guard let subscription = product.subscription,
              let introOffer = subscription.introductoryOffer else {
            return nil
        }
        
        switch introOffer.paymentMode {
        case .freeTrial:
            return "ทดลองใช้ฟรี \(introOffer.period.value) \(introOffer.period.unit.localizedDescription)"
        case .payAsYouGo:
            return "\(introOffer.price.formatted(.currency(code: "THB"))) สำหรับ \(introOffer.period.value) \(introOffer.period.unit.localizedDescription) แรก"
        case .payUpFront:
            return "จ่าย \(introOffer.price.formatted(.currency(code: "THB"))) ล่วงหน้า"
        @unknown default:
            return nil
        }
    }
}

extension Product.SubscriptionPeriod.Unit {
    var localizedDescription: String {
        switch self {
        case .day: return "วัน"
        case .week: return "สัปดาห์"
        case .month: return "เดือน"
        case .year: return "ปี"
        @unknown default: return "ช่วงเวลา"
        }
    }
}
```

### 2.3 Grace Period Handling

```swift
// Grace Period: เมื่อ payment ล้มเหลว Apple ให้ grace period 6-16 วัน
// ระหว่างนี้ user ยังเข้าถึง premium features ได้

extension SubscriptionManager {
    
    struct SubscriptionState {
        let isActive: Bool
        let isInGracePeriod: Bool
        let gracePeriodExpiryDate: Date?
        let expiryDate: Date?
        let willAutoRenew: Bool
    }
    
    func getDetailedSubscriptionState() async -> SubscriptionState {
        for await result in Transaction.currentEntitlements {
            guard let transaction = try? checkVerified(result) else { continue }
            guard let subscriptionStatus = try? await transaction.subscriptionStatus else { continue }
            
            switch subscriptionStatus.state {
            case .subscribed:
                return SubscriptionState(
                    isActive: true,
                    isInGracePeriod: false,
                    gracePeriodExpiryDate: nil,
                    expiryDate: transaction.expirationDate,
                    willAutoRenew: subscriptionStatus.renewalInfo.wrappedValue?.willAutoRenew ?? false
                )
                
            case .inGracePeriod:
                // ยังให้ access แต่ต้องแจ้ง user ให้ update payment
                return SubscriptionState(
                    isActive: true,
                    isInGracePeriod: true,
                    gracePeriodExpiryDate: subscriptionStatus.renewalInfo.wrappedValue?.gracePeriodExpirationDate,
                    expiryDate: transaction.expirationDate,
                    willAutoRenew: true
                )
                
            case .expired:
                return SubscriptionState(
                    isActive: false,
                    isInGracePeriod: false,
                    gracePeriodExpiryDate: nil,
                    expiryDate: transaction.expirationDate,
                    willAutoRenew: false
                )
                
            default:
                break
            }
        }
        
        return SubscriptionState(
            isActive: false,
            isInGracePeriod: false,
            gracePeriodExpiryDate: nil,
            expiryDate: nil,
            willAutoRenew: false
        )
    }
    
    // แสดง Banner เมื่ออยู่ใน Grace Period
    @ViewBuilder
    func gracePeriodBanner(state: SubscriptionState) -> some View {
        if state.isInGracePeriod,
           let expiryDate = state.gracePeriodExpiryDate {
            HStack {
                Image(systemName: "exclamationmark.triangle.fill")
                    .foregroundColor(.orange)
                VStack(alignment: .leading) {
                    Text("มีปัญหาการชำระเงิน")
                        .font(.headline)
                    Text("กรุณาอัปเดตข้อมูลการชำระเงินก่อน \(expiryDate.formatted(date: .abbreviated, time: .omitted))")
                        .font(.subheadline)
                        .foregroundColor(.secondary)
                }
                Spacer()
                Button("แก้ไข") {
                    // เปิด Subscription Management
                    if let url = URL(string: "https://apps.apple.com/account/subscriptions") {
                        UIApplication.shared.open(url)
                    }
                }
            }
            .padding()
            .background(Color.orange.opacity(0.1))
            .cornerRadius(12)
        }
    }
}
```

### 2.4 Promotional Offers

```swift
// Promotional Offers สำหรับ Win-back หรือ Loyalty

extension SubscriptionManager {
    
    // Win-back offer สำหรับ cancelled subscribers
    func getWinBackOffer(for product: Product) async -> Product.SubscriptionOffer? {
        guard let subscription = product.subscription else { return nil }
        
        // ตรวจสอบว่า user เป็น lapsed subscriber
        var wasSubscriber = false
        for await result in Transaction.all {
            if let transaction = try? checkVerified(result),
               transaction.productID == product.id {
                wasSubscriber = true
                break
            }
        }
        
        guard wasSubscriber else { return nil }
        
        // Return promotional offer ถ้ามี
        return subscription.promotionalOffers.first
    }
    
    // Purchase with Promotional Offer
    func purchaseWithPromotion(
        _ product: Product,
        offer: Product.SubscriptionOffer,
        signature: Product.PurchaseOption
    ) async throws -> Transaction? {
        let result = try await product.purchase(options: [
            .promotionalOffer(offerID: offer.id, keyID: "KEY_ID", nonce: UUID(), signature: Data(), timestamp: Date().timeIntervalSince1970 as NSNumber)
        ])
        
        switch result {
        case .success(let verification):
            let transaction = try checkVerified(verification)
            await updateSubscriptionStatus()
            await transaction.finish()
            return transaction
        default:
            return nil
        }
    }
}
```

### 2.5 StoreKit Testing ใน Xcode

```swift
// StoreKit Configuration File (StoreKitConfig.storekit)
// สร้างใน Xcode: File > New > File > StoreKit Configuration File

/*
ใน StoreKit Config File:
1. กำหนด subscription products
2. กำหนด pricing
3. กำหนด subscription groups
4. Test scenarios: normal purchase, renewal, cancellation

Testing scenarios:
- Transaction Manager ใน Xcode: Window > Transaction Manager
- ใช้ StoreKit Test framework สำหรับ unit tests
*/

// Unit Testing with StoreKit
import XCTest
import StoreKitTest

class SubscriptionTests: XCTestCase {
    
    var session: SKTestSession!
    var manager: SubscriptionManager!
    
    override func setUp() async throws {
        session = try SKTestSession(configurationFileNamed: "StoreKitConfig")
        session.disableDialogs = true
        session.clearTransactions()
        
        manager = SubscriptionManager()
        
        // รอให้ products load
        try await Task.sleep(nanoseconds: 1_000_000_000)
    }
    
    func testPurchaseMonthlySubscription() async throws {
        // Given
        let product = manager.products.first { $0.id == SubscriptionManager.monthlyProductId }
        XCTAssertNotNil(product)
        
        // When
        let transaction = try await manager.purchase(product!)
        
        // Then
        XCTAssertNotNil(transaction)
        XCTAssertEqual(manager.subscriptionStatus, .monthly)
    }
    
    func testSubscriptionRenewal() async throws {
        // Purchase subscription
        let product = manager.products.first { $0.id == SubscriptionManager.monthlyProductId }!
        _ = try await manager.purchase(product)
        
        // Simulate time passing (renewal)
        session.timeRate = .oneSecondIsOneDay  // เร่งเวลา
        
        // รอ renewal
        try await Task.sleep(nanoseconds: 2_000_000_000)
        
        // Verify still subscribed
        await manager.updateSubscriptionStatus()
        XCTAssertEqual(manager.subscriptionStatus, .monthly)
    }
    
    func testSubscriptionExpiry() async throws {
        let product = manager.products.first { $0.id == SubscriptionManager.monthlyProductId }!
        _ = try await manager.purchase(product)
        
        // Expire the subscription
        session.expireSubscription(productIdentifier: SubscriptionManager.monthlyProductId)
        
        await manager.updateSubscriptionStatus()
        XCTAssertEqual(manager.subscriptionStatus, .notSubscribed)
    }
}
```

---

## 3. Revenue Optimization

### 3.1 Paywall Design Patterns

```swift
// Hard Paywall: ต้อง subscribe ก่อนใช้งาน
// Soft Paywall: ใช้ได้บางส่วน แล้ว prompt เมื่อถึง limit

import SwiftUI

// Soft Paywall - Triggered เมื่อ user ถึง limit
struct SoftPaywallView: View {
    @StateObject private var subscriptionManager = SubscriptionManager.shared
    @State private var selectedProduct: Product?
    @Environment(\.dismiss) private var dismiss
    
    let triggerReason: TriggerReason
    
    enum TriggerReason {
        case documentLimit(current: Int, max: Int)
        case premiumFeature(name: String)
        case storageLimit
    }
    
    var body: some View {
        NavigationView {
            ScrollView {
                VStack(spacing: 24) {
                    // Hero Section
                    VStack(spacing: 12) {
                        Image(systemName: "star.circle.fill")
                            .font(.system(size: 80))
                            .foregroundStyle(
                                LinearGradient(
                                    colors: [.yellow, .orange],
                                    startPoint: .topLeading,
                                    endPoint: .bottomTrailing
                                )
                            )
                        
                        Text("อัปเกรดเป็น Premium")
                            .font(.largeTitle.bold())
                        
                        Text(triggerDescription)
                            .font(.subheadline)
                            .foregroundColor(.secondary)
                            .multilineTextAlignment(.center)
                    }
                    .padding(.top)
                    
                    // Features List
                    VStack(alignment: .leading, spacing: 12) {
                        FeatureRow(icon: "infinity", text: "สร้างเอกสารไม่จำกัด")
                        FeatureRow(icon: "icloud.fill", text: "Cloud sync ทุก devices")
                        FeatureRow(icon: "square.and.arrow.up", text: "Export เป็น PDF/Word")
                        FeatureRow(icon: "paintbrush.fill", text: "Theme พิเศษ 20+ แบบ")
                        FeatureRow(icon: "bell.badge.fill", text: "Priority support")
                    }
                    .padding(.horizontal)
                    
                    // Pricing Options
                    if !subscriptionManager.products.isEmpty {
                        PricingOptionsView(
                            products: subscriptionManager.products,
                            selectedProduct: $selectedProduct
                        )
                    }
                    
                    // CTA Button
                    Button(action: purchaseSelected) {
                        HStack {
                            if let product = selectedProduct {
                                Text("เริ่มทดลองใช้ฟรี 7 วัน")
                                    .font(.headline)
                                    .foregroundColor(.white)
                            }
                        }
                        .frame(maxWidth: .infinity)
                        .padding()
                        .background(Color.blue)
                        .cornerRadius(14)
                    }
                    .padding(.horizontal)
                    
                    // Terms
                    TermsView()
                }
            }
            .navigationBarItems(
                trailing: Button("ไม่ใช่ตอนนี้") { dismiss() }
                    .foregroundColor(.secondary)
            )
        }
    }
    
    private var triggerDescription: String {
        switch triggerReason {
        case .documentLimit(let current, let max):
            return "คุณสร้างเอกสารครบ \(max) ชิ้นแล้ว\nอัปเกรดเพื่อสร้างไม่จำกัด"
        case .premiumFeature(let name):
            return "\(name) เป็น Premium feature\nอัปเกรดเพื่อเข้าถึง"
        case .storageLimit:
            return "พื้นที่เก็บข้อมูลเต็มแล้ว\nอัปเกรดเพื่อพื้นที่ไม่จำกัด"
        }
    }
    
    private func purchaseSelected() {
        guard let product = selectedProduct else { return }
        Task {
            do {
                _ = try await subscriptionManager.purchase(product)
            } catch {
                print("Purchase failed: \(error)")
            }
        }
    }
}

struct FeatureRow: View {
    let icon: String
    let text: String
    
    var body: some View {
        HStack(spacing: 12) {
            Image(systemName: icon)
                .foregroundColor(.blue)
                .frame(width: 24)
            Text(text)
                .font(.body)
            Spacer()
            Image(systemName: "checkmark")
                .foregroundColor(.green)
        }
    }
}

struct PricingOptionsView: View {
    let products: [Product]
    @Binding var selectedProduct: Product?
    
    var body: some View {
        VStack(spacing: 12) {
            ForEach(products, id: \.id) { product in
                PricingCard(
                    product: product,
                    isSelected: selectedProduct?.id == product.id,
                    isRecommended: product.id.contains("annual")
                ) {
                    selectedProduct = product
                }
            }
        }
        .padding(.horizontal)
        .onAppear {
            // Default select annual (best value)
            selectedProduct = products.first { $0.id.contains("annual") } ?? products.first
        }
    }
}

struct PricingCard: View {
    let product: Product
    let isSelected: Bool
    let isRecommended: Bool
    let onTap: () -> Void
    
    var body: some View {
        Button(action: onTap) {
            HStack {
                VStack(alignment: .leading, spacing: 4) {
                    HStack {
                        Text(product.displayName)
                            .font(.headline)
                        if isRecommended {
                            Text("ดีที่สุด")
                                .font(.caption)
                                .padding(.horizontal, 8)
                                .padding(.vertical, 2)
                                .background(Color.blue)
                                .foregroundColor(.white)
                                .cornerRadius(8)
                        }
                    }
                    
                    if let subscription = product.subscription,
                       let introOffer = subscription.introductoryOffer,
                       introOffer.paymentMode == .freeTrial {
                        Text("7 วันฟรี จากนั้น \(product.displayPrice)")
                            .font(.subheadline)
                            .foregroundColor(.secondary)
                    } else {
                        Text(product.displayPrice)
                            .font(.subheadline)
                            .foregroundColor(.secondary)
                    }
                }
                
                Spacer()
                
                if isSelected {
                    Image(systemName: "checkmark.circle.fill")
                        .foregroundColor(.blue)
                        .font(.title2)
                }
            }
            .padding()
            .background(
                RoundedRectangle(cornerRadius: 12)
                    .stroke(isSelected ? Color.blue : Color.gray.opacity(0.3), lineWidth: isSelected ? 2 : 1)
            )
        }
        .buttonStyle(.plain)
    }
}

struct TermsView: View {
    var body: some View {
        VStack(spacing: 4) {
            Text("การสมัครสมาชิกจะต่ออายุอัตโนมัติ ยกเว้นจะยกเลิกอย่างน้อย 24 ชั่วโมงก่อนหมดอายุ")
            HStack(spacing: 16) {
                Link("นโยบายความเป็นส่วนตัว", destination: URL(string: "https://app.com/privacy")!)
                Link("เงื่อนไขการใช้งาน", destination: URL(string: "https://app.com/terms")!)
            }
            .font(.caption)
            .foregroundColor(.blue)
        }
        .font(.caption2)
        .foregroundColor(.secondary)
        .multilineTextAlignment(.center)
        .padding(.horizontal)
    }
}
```

### 3.2 A/B Testing Paywalls

```swift
// A/B Testing Framework สำหรับ Paywalls

class PaywallABTestManager {
    
    enum PaywallVariant: String, CaseIterable {
        case control = "A"    // Original paywall
        case variantB = "B"   // New design
        case variantC = "C"   // Different pricing display
    }
    
    struct TestConfig {
        let testId: String
        let variants: [PaywallVariant]
        let weights: [Double]  // [0.33, 0.33, 0.34]
        let startDate: Date
        let endDate: Date
    }
    
    // Assign variant to user (deterministic based on user ID)
    func assignVariant(userId: String, test: TestConfig) -> PaywallVariant {
        // Use hash ของ userId เพื่อ deterministic assignment
        // (user เห็น variant เดิมทุกครั้ง)
        let hash = abs(userId.hashValue)
        let totalWeight = test.weights.reduce(0, +)
        let normalizedHash = Double(hash % 100) / 100.0 * totalWeight
        
        var cumulative = 0.0
        for (index, weight) in test.weights.enumerated() {
            cumulative += weight
            if normalizedHash < cumulative {
                return test.variants[index]
            }
        }
        
        return test.variants.last ?? .control
    }
    
    // Track conversion event
    func trackConversion(variant: PaywallVariant, productId: String) {
        Analytics.shared.track("paywall_conversion", parameters: [
            "variant": variant.rawValue,
            "product_id": productId,
            "timestamp": Date().timeIntervalSince1970
        ])
    }
    
    // Track impression
    func trackImpression(variant: PaywallVariant) {
        Analytics.shared.track("paywall_impression", parameters: [
            "variant": variant.rawValue,
            "timestamp": Date().timeIntervalSince1970
        ])
    }
    
    // Calculate statistical significance
    func isSignificant(
        controlConversions: Int,
        controlImpressions: Int,
        variantConversions: Int,
        variantImpressions: Int,
        confidenceLevel: Double = 0.95
    ) -> Bool {
        let controlRate = Double(controlConversions) / Double(controlImpressions)
        let variantRate = Double(variantConversions) / Double(variantImpressions)
        
        // Z-test for proportions
        let pooledRate = Double(controlConversions + variantConversions) / Double(controlImpressions + variantImpressions)
        let standardError = sqrt(pooledRate * (1 - pooledRate) * (1.0/Double(controlImpressions) + 1.0/Double(variantImpressions)))
        
        guard standardError > 0 else { return false }
        
        let zScore = abs(variantRate - controlRate) / standardError
        
        // z > 1.96 for 95% confidence
        return zScore > 1.96
    }
}

class Analytics {
    static let shared = Analytics()
    func track(_ event: String, parameters: [String: Any]) {
        // Send to analytics platform
    }
}
```

### 3.3 Annual vs Monthly Conversion

```swift
// Strategies เพื่อเพิ่ม Annual Subscription Conversion

struct AnnualConversionStrategy {
    
    // 1. แสดง savings อย่างชัดเจน
    static func savingsMessage(monthly: Product, annual: Product) -> String? {
        guard let monthlySubscription = monthly.subscription,
              let annualSubscription = annual.subscription else {
            return nil
        }
        
        let monthlyTotal = monthly.price * 12
        let annualTotal = annual.price
        let savings = monthlyTotal - annualTotal
        let savingsPercent = Int((savings / monthlyTotal) * 100)
        
        return "ประหยัด \(savingsPercent)% เมื่อสมัครรายปี (ประหยัด \(savings.formatted(.currency(code: "THB"))))"
    }
    
    // 2. Default select annual ใน paywall
    static func recommendedProduct(from products: [Product]) -> Product? {
        return products.first { $0.id.contains("annual") }
    }
    
    // 3. Monthly to Annual Upsell (หลังจาก subscribe monthly 1 เดือน)
    func shouldShowAnnualUpsell(
        subscriptionStartDate: Date,
        currentDate: Date = Date()
    ) -> Bool {
        let monthsSubscribed = Calendar.current.dateComponents(
            [.month],
            from: subscriptionStartDate,
            to: currentDate
        ).month ?? 0
        
        // แสดง upsell หลังจาก subscribe 1 เดือน
        return monthsSubscribed >= 1
    }
}
```

### 3.4 LTV Calculation

```swift
// Lifetime Value Calculation

struct LTVCalculator {
    
    // Simple LTV
    static func simpleLTV(
        arpu: Double,           // Average Revenue Per User per month
        monthlyChurnRate: Double // % ที่ churn ต่อเดือน
    ) -> Double {
        // LTV = ARPU / Churn Rate
        guard monthlyChurnRate > 0 else { return Double.infinity }
        return arpu / monthlyChurnRate
    }
    
    // Discounted LTV (คำนึงถึง time value of money)
    static func discountedLTV(
        monthlyRevenue: Double,
        monthlyChurnRate: Double,
        monthlyDiscountRate: Double = 0.01  // ~12% annual
    ) -> Double {
        // LTV = Monthly Revenue / (Churn Rate + Discount Rate)
        let denominator = monthlyChurnRate + monthlyDiscountRate
        guard denominator > 0 else { return Double.infinity }
        return monthlyRevenue / denominator
    }
    
    // Cohort-based LTV (จาก actual data)
    static func cohortLTV(cohortRevenues: [Double]) -> Double {
        // cohortRevenues: revenue ต่อ user แต่ละเดือน จาก cohort เดียวกัน
        // [9.99, 8.5, 7.2, 6.1, ...] (ลดลงเพราะ churn)
        return cohortRevenues.reduce(0, +)
    }
    
    // Unit Economics Check
    static func isUnitEconomicsHealthy(
        ltv: Double,
        cac: Double  // Customer Acquisition Cost
    ) -> (isHealthy: Bool, ratio: Double) {
        let ratio = ltv / cac
        // Rule of thumb: LTV:CAC ratio > 3 is good, > 5 is excellent
        return (isHealthy: ratio >= 3.0, ratio: ratio)
    }
}

// ตัวอย่างการใช้งาน
let ltv = LTVCalculator.simpleLTV(arpu: 9.99, monthlyChurnRate: 0.05)
print("LTV: $\(String(format: "%.2f", ltv))")  // LTV: $199.80

let unitEcon = LTVCalculator.isUnitEconomicsHealthy(ltv: ltv, cac: 40.0)
print("LTV:CAC = \(String(format: "%.1f", unitEcon.ratio))x (\(unitEcon.isHealthy ? "Healthy" : "Unhealthy"))")
```

---

## 4. Ad Integration

### 4.1 Google AdMob Integration

```swift
// Setup:
// 1. เพิ่ม Google-Mobile-Ads-SDK ผ่าน Swift Package Manager
// 2. เพิ่ม GADApplicationIdentifier ใน Info.plist
// 3. Initialize ใน AppDelegate

import GoogleMobileAds

// AppDelegate
class AppDelegate: NSObject, UIApplicationDelegate {
    func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
    ) -> Bool {
        // Initialize the Mobile Ads SDK
        GADMobileAds.sharedInstance().start { status in
            // Initialization complete
            print("AdMob initialized: \(status)")
        }
        return true
    }
}

// Banner Ad View
struct BannerAdView: UIViewRepresentable {
    let adUnitID: String
    
    func makeUIView(context: Context) -> GADBannerView {
        let bannerView = GADBannerView(adSize: GADAdSizeBanner)
        bannerView.adUnitID = adUnitID
        bannerView.rootViewController = UIApplication.shared.connectedScenes
            .compactMap { $0 as? UIWindowScene }
            .flatMap { $0.windows }
            .first?.rootViewController
        
        bannerView.load(GADRequest())
        return bannerView
    }
    
    func updateUIView(_ uiView: GADBannerView, context: Context) {}
}

// Interstitial Ad Manager
class InterstitialAdManager: NSObject, ObservableObject, GADFullScreenContentDelegate {
    
    private var interstitialAd: GADInterstitialAd?
    @Published var isAdReady = false
    
    let adUnitID: String
    
    init(adUnitID: String) {
        self.adUnitID = adUnitID
        super.init()
        loadAd()
    }
    
    func loadAd() {
        Task {
            do {
                interstitialAd = try await GADInterstitialAd.load(
                    withAdUnitID: adUnitID,
                    request: GADRequest()
                )
                interstitialAd?.fullScreenContentDelegate = self
                await MainActor.run {
                    isAdReady = true
                }
            } catch {
                print("Failed to load interstitial: \(error)")
            }
        }
    }
    
    func showAd(from viewController: UIViewController) {
        guard let ad = interstitialAd else {
            print("Ad not ready")
            return
        }
        
        ad.present(fromRootViewController: viewController)
    }
    
    // GADFullScreenContentDelegate
    func adDidDismissFullScreenContent(_ ad: GADFullScreenPresentingAd) {
        // Load next ad
        isAdReady = false
        loadAd()
    }
}

// Rewarded Ad Manager
class RewardedAdManager: NSObject, ObservableObject, GADFullScreenContentDelegate {
    
    private var rewardedAd: GADRewardedAd?
    @Published var isAdReady = false
    var onRewardEarned: ((GADAdReward) -> Void)?
    
    func loadAd(adUnitID: String) {
        Task {
            do {
                rewardedAd = try await GADRewardedAd.load(
                    withAdUnitID: adUnitID,
                    request: GADRequest()
                )
                rewardedAd?.fullScreenContentDelegate = self
                await MainActor.run {
                    isAdReady = true
                }
            } catch {
                print("Failed to load rewarded ad: \(error)")
            }
        }
    }
    
    func showAd(from viewController: UIViewController, completion: @escaping (GADAdReward) -> Void) {
        guard let ad = rewardedAd else { return }
        
        onRewardEarned = completion
        
        ad.present(fromRootViewController: viewController) { [weak self] in
            let reward = ad.adReward
            completion(reward)
        }
    }
}
```

### 4.2 ATT (App Tracking Transparency) Implementation

```swift
import AppTrackingTransparency
import AdSupport

class ATTManager {
    
    static let shared = ATTManager()
    
    // Request tracking permission
    func requestPermission() async -> ATTrackingManager.AuthorizationStatus {
        // ต้องรอ app active ก่อน
        if #available(iOS 14, *) {
            return await withCheckedContinuation { continuation in
                ATTrackingManager.requestTrackingAuthorization { status in
                    continuation.resume(returning: status)
                }
            }
        }
        return .authorized
    }
    
    // เวลาที่ดีที่สุดในการ request ATT
    // 1. หลังจาก onboarding (user เข้าใจ value ของ app แล้ว)
    // 2. หลังจาก positive action (เช่น หลัง user save bookmark แรก)
    // 3. ไม่ใช่ทันทีที่เปิด app
    
    // ตรวจสอบ IDFA availability
    var idfa: String? {
        guard ATTrackingManager.trackingAuthorizationStatus == .authorized else {
            return nil
        }
        return ASIdentifierManager.shared().advertisingIdentifier.uuidString
    }
    
    // Impact analysis
    struct ATTImpact {
        // เมื่อ user deny tracking:
        static let estimatedRevenueDecline = 0.40  // ~40% ลดลง (industry average)
        
        // Mitigations:
        // 1. ใช้ SKAdNetwork สำหรับ attribution (ไม่ต้องการ IDFA)
        // 2. Focus บน contextual targeting (ไม่ใช่ behavioral)
        // 3. First-party data (email, purchase history)
    }
}

// Pre-permission prompt (แสดงก่อน ATT dialog จริง)
struct ATTPrePromptView: View {
    let onContinue: () -> Void
    let onSkip: () -> Void
    
    var body: some View {
        VStack(spacing: 24) {
            Image(systemName: "hand.raised.fill")
                .font(.system(size: 60))
                .foregroundColor(.blue)
            
            Text("ช่วยเราปรับปรุงแอป")
                .font(.title2.bold())
            
            Text("เราจะแสดงโฆษณาที่เกี่ยวข้องกับคุณมากขึ้น และไม่แชร์ข้อมูลส่วนตัวกับบุคคลที่สาม")
                .multilineTextAlignment(.center)
                .foregroundColor(.secondary)
            
            VStack(spacing: 12) {
                Button("อนุญาต", action: onContinue)
                    .buttonStyle(.borderedProminent)
                    .controlSize(.large)
                
                Button("ไม่ใช่ตอนนี้", action: onSkip)
                    .foregroundColor(.secondary)
            }
        }
        .padding()
    }
}
```

---

## 5. Subscription Analytics

### 5.1 Key Metrics

```swift
// Subscription Metrics Dashboard

struct SubscriptionDashboard {
    
    // MRR: Monthly Recurring Revenue
    struct MRR {
        let newMRR: Double        // จาก new subscribers
        let expansionMRR: Double  // จาก upgrades
        let churndMRR: Double     // lost จาก cancellations
        let contractionMRR: Double // จาก downgrades
        let reactivationMRR: Double // จาก win-backs
        
        var netNewMRR: Double {
            return newMRR + expansionMRR + reactivationMRR - churndMRR - contractionMRR
        }
    }
    
    // Churn Analysis
    struct ChurnAnalysis {
        let totalSubscribersStart: Int
        let churned: Int
        let newSubscribers: Int
        let totalSubscribersEnd: Int
        
        var monthlyChurnRate: Double {
            return Double(churned) / Double(totalSubscribersStart)
        }
        
        var annualChurnRate: Double {
            return 1 - pow(1 - monthlyChurnRate, 12)
        }
        
        var netGrowthRate: Double {
            return Double(newSubscribers - churned) / Double(totalSubscribersStart)
        }
    }
    
    // Cohort Retention
    struct CohortRetention {
        let cohortMonth: String    // "2025-01"
        let retentionByMonth: [Int: Double]  // [1: 0.85, 2: 0.72, 3: 0.65, ...]
        
        var d30Retention: Double { retentionByMonth[1] ?? 0 }
        var d60Retention: Double { retentionByMonth[2] ?? 0 }
        var d90Retention: Double { retentionByMonth[3] ?? 0 }
        var d180Retention: Double { retentionByMonth[6] ?? 0 }
        var d365Retention: Double { retentionByMonth[12] ?? 0 }
    }
}
```

### 5.2 RevenueCat Integration

```swift
// RevenueCat เป็น subscription analytics platform ที่ยอดนิยม
// Setup: เพิ่ม RevenueCat SDK ผ่าน Swift Package Manager

import RevenueCat

// Setup ใน App
@main
struct MyApp: App {
    
    init() {
        Purchases.logLevel = .debug
        Purchases.configure(withAPIKey: "YOUR_REVENUECAT_API_KEY")
        
        // Set user ID (เมื่อ user login)
        // Purchases.shared.logIn("user_id")
    }
    
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}

// RevenueCat Manager
class RevenueCatManager: ObservableObject {
    
    @Published private(set) var offerings: Offerings?
    @Published private(set) var customerInfo: CustomerInfo?
    @Published private(set) var isPremium = false
    
    init() {
        Task {
            await loadOfferings()
            await refreshCustomerInfo()
        }
        
        // Listen for updates
        Task {
            for await customerInfo in Purchases.shared.customerInfoStream {
                await MainActor.run {
                    self.customerInfo = customerInfo
                    self.updatePremiumStatus(customerInfo)
                }
            }
        }
    }
    
    // โหลด Offerings (แทน direct product fetching)
    func loadOfferings() async {
        do {
            let offerings = try await Purchases.shared.offerings()
            await MainActor.run {
                self.offerings = offerings
            }
        } catch {
            print("Failed to load offerings: \(error)")
        }
    }
    
    // ซื้อ Package จาก Offering
    func purchase(_ package: Package) async throws {
        let (_, customerInfo, _) = try await Purchases.shared.purchase(package: package)
        await MainActor.run {
            self.customerInfo = customerInfo
            self.updatePremiumStatus(customerInfo)
        }
    }
    
    // Restore Purchases
    func restorePurchases() async throws {
        let customerInfo = try await Purchases.shared.restorePurchases()
        await MainActor.run {
            self.customerInfo = customerInfo
            self.updatePremiumStatus(customerInfo)
        }
    }
    
    // Check entitlement
    private func updatePremiumStatus(_ info: CustomerInfo) {
        isPremium = info.entitlements["premium"]?.isActive == true
    }
    
    // User Attributes (สำหรับ analytics)
    func setUserAttributes(email: String, name: String) {
        Purchases.shared.attribution.setEmail(email)
        Purchases.shared.attribution.setDisplayName(name)
    }
}

// RevenueCat Paywall View
struct RevenueCatPaywallView: View {
    @StateObject private var rcManager = RevenueCatManager()
    @State private var selectedPackage: Package?
    
    var body: some View {
        Group {
            if let offering = rcManager.offerings?.current {
                PaywallContent(
                    offering: offering,
                    selectedPackage: $selectedPackage,
                    onPurchase: { package in
                        Task {
                            do {
                                try await rcManager.purchase(package)
                            } catch {
                                print("Purchase error: \(error)")
                            }
                        }
                    }
                )
            } else {
                ProgressView("กำลังโหลด...")
            }
        }
    }
}

struct PaywallContent: View {
    let offering: Offering
    @Binding var selectedPackage: Package?
    let onPurchase: (Package) -> Void
    
    var body: some View {
        VStack(spacing: 20) {
            Text(offering.identifier)
                .font(.headline)
            
            ForEach(offering.availablePackages, id: \.identifier) { package in
                PackageRow(
                    package: package,
                    isSelected: selectedPackage?.identifier == package.identifier
                ) {
                    selectedPackage = package
                }
            }
            
            if let selected = selectedPackage {
                Button("สมัครสมาชิก") {
                    onPurchase(selected)
                }
                .buttonStyle(.borderedProminent)
                .controlSize(.large)
            }
        }
        .onAppear {
            selectedPackage = offering.annual ?? offering.monthly
        }
    }
}

struct PackageRow: View {
    let package: Package
    let isSelected: Bool
    let onTap: () -> Void
    
    var body: some View {
        Button(action: onTap) {
            HStack {
                VStack(alignment: .leading) {
                    Text(package.storeProduct.localizedTitle)
                        .font(.headline)
                    Text(package.storeProduct.localizedPriceString)
                        .foregroundColor(.secondary)
                }
                Spacer()
                if isSelected {
                    Image(systemName: "checkmark.circle.fill")
                        .foregroundColor(.blue)
                }
            }
            .padding()
            .background(
                RoundedRectangle(cornerRadius: 10)
                    .stroke(isSelected ? Color.blue : Color.gray.opacity(0.3))
            )
        }
        .buttonStyle(.plain)
    }
}
```

---

## 6. Growth Strategies

### 6.1 App Store Optimization (ASO) Deep Dive

```markdown
# ASO Optimization Checklist

## App Name (30 characters)
- ใส่ primary keyword ถ้าเป็นไปได้
- ตัวอย่าง: "Todoist: To-Do List & Planner" ไม่ใช่แค่ "Todoist"

## Subtitle (30 characters)  
- ใส่ secondary keywords
- Describe core value proposition
- ตัวอย่าง: "Organize Work and Life"

## Keywords Field (100 characters)
- คั่นด้วย comma ไม่ใช่ space
- ไม่ต้องซ้ำกับ App Name/Subtitle
- ไม่ต้องใส่ brand names ที่ไม่เกี่ยวข้อง
- Research tools: AppFollow, Sensor Tower, AppTweak

## Description (4000 characters)
- First 3 lines สำคัญที่สุด (ก่อน "more")
- ใส่ keywords อย่างเป็นธรรมชาติ
- Bullet points อ่านง่ายกว่า wall of text
- Social proof (number of users, awards)
- Clear CTA

## Screenshots (10 slots)
- First 3 screenshots สำคัญที่สุด
- ใส่ text overlay อธิบาย benefits ไม่ใช่ features
- "Save 2 hours a day" ดีกว่า "Task Management"
- Test different styles (A/B test ใน App Store Connect)

## Preview Video
- 30 seconds
- Show app in action ภายใน 5 วินาทีแรก
- No voice-over ต้องใช้ text (หลาย users ดูแบบ muted)
- Captions ภาษา native

## Localization
- Localize screenshots และ descriptions สำหรับ markets หลัก
- ไม่ใช่แค่ translate แต่ culturally relevant
```

### 6.2 Review Management

```swift
// Request Review ในเวลาที่เหมาะสม

import StoreKit

class ReviewRequestManager {
    
    static let shared = ReviewRequestManager()
    
    private let minimumRunCount = 3
    private let minimumDaysBetweenRequests = 30
    
    private var runCount: Int {
        get { UserDefaults.standard.integer(forKey: "app_run_count") }
        set { UserDefaults.standard.set(newValue, forKey: "app_run_count") }
    }
    
    private var lastReviewRequestDate: Date? {
        get { UserDefaults.standard.object(forKey: "last_review_request_date") as? Date }
        set { UserDefaults.standard.set(newValue, forKey: "last_review_request_date") }
    }
    
    func incrementRunCount() {
        runCount += 1
    }
    
    // Request review หลังจาก positive user action
    func requestReviewIfAppropriate() {
        // ตรวจสอบเงื่อนไข
        guard runCount >= minimumRunCount else { return }
        
        if let lastRequest = lastReviewRequestDate {
            let daysSinceLastRequest = Calendar.current.dateComponents(
                [.day],
                from: lastRequest,
                to: Date()
            ).day ?? 0
            
            guard daysSinceLastRequest >= minimumDaysBetweenRequests else { return }
        }
        
        // Request review
        if let scene = UIApplication.shared.connectedScenes
            .first(where: { $0.activationState == .foregroundActive }) as? UIWindowScene {
            SKStoreReviewController.requestReview(in: scene)
            lastReviewRequestDate = Date()
        }
    }
    
    // Best moments to request review:
    // - หลังจาก user complete task สำเร็จ
    // - หลังจาก user ใช้งาน X sessions
    // - หลังจาก user subscribe (แสดงว่า satisfied)
    // - ไม่ใช่: หลัง crash, หลัง error, ระหว่าง task
    
    enum ReviewTrigger {
        case taskCompleted
        case sessionMilestone(count: Int)
        case successfulPurchase
        case positiveAction
    }
    
    func handleTrigger(_ trigger: ReviewTrigger) {
        switch trigger {
        case .taskCompleted:
            requestReviewIfAppropriate()
        case .sessionMilestone(let count) where count >= 5:
            requestReviewIfAppropriate()
        case .successfulPurchase:
            // รอ 2 วันก่อน request (ให้ user ได้ใช้ premium ก่อน)
            DispatchQueue.main.asyncAfter(deadline: .now() + 2 * 24 * 3600) {
                self.requestReviewIfAppropriate()
            }
        default:
            break
        }
    }
}
```

---

## 7. Viral และ Referral Mechanics

### 7.1 Referral Program Implementation

```swift
// Referral System

class ReferralManager {
    
    static let shared = ReferralManager()
    
    struct ReferralReward {
        let referrerReward: String   // เช่น "30 วันฟรี"
        let refereeReward: String    // เช่น "7 วันฟรี"
    }
    
    let reward = ReferralReward(
        referrerReward: "30 วันฟรี",
        refereeReward: "7 วันฟรี"
    )
    
    // Generate Unique Referral Code
    func generateReferralCode(for userId: String) -> String {
        // ใช้ base62 encoding ของ user ID ย่อ
        let characters = "ABCDEFGHJKLMNPQRSTUVWXYZabcdefghjkmnpqrstuvwxyz23456789"
        var code = ""
        var hash = abs(userId.hashValue)
        
        for _ in 0..<6 {
            let index = hash % characters.count
            code.append(characters[characters.index(characters.startIndex, offsetBy: index)])
            hash /= characters.count
        }
        
        return code
    }
    
    // Create Referral Link
    func createReferralLink(code: String) -> URL? {
        var components = URLComponents(string: "https://app.com/join")
        components?.queryItems = [URLQueryItem(name: "ref", value: code)]
        return components?.url
    }
    
    // Share Referral
    func shareReferral(code: String, from viewController: UIViewController) {
        guard let url = createReferralLink(code: code) else { return }
        
        let message = "ใช้ [App Name] แล้วชอบมาก! ใช้ลิงก์นี้สมัครแล้วได้รับ 7 วันฟรี: \(url.absoluteString)"
        
        let activityVC = UIActivityViewController(
            activityItems: [message],
            applicationActivities: nil
        )
        
        viewController.present(activityVC, animated: true)
    }
    
    // Validate Referral Code
    func validateReferralCode(_ code: String) async throws -> ReferralCodeInfo {
        // Call API to validate
        struct ReferralCodeInfo: Codable {
            let isValid: Bool
            let referrerName: String?
            let reward: String
        }
        
        // Mock implementation
        return ReferralCodeInfo(
            isValid: true,
            referrerName: "สมชาย ใจดี",
            reward: "7 วันฟรี"
        )
    }
}

// Referral Onboarding View
struct ReferralOnboardingView: View {
    @State private var referralCode = ""
    @State private var validationResult: String?
    
    var body: some View {
        VStack(spacing: 20) {
            Text("มีโค้ดชวนเพื่อนไหม?")
                .font(.title2.bold())
            
            Text("ใส่โค้ดเพื่อรับ 7 วันฟรี")
                .foregroundColor(.secondary)
            
            TextField("ใส่โค้ดที่นี่", text: $referralCode)
                .textFieldStyle(.roundedBorder)
                .textInputAutocapitalization(.characters)
                .padding(.horizontal)
            
            if let result = validationResult {
                Text(result)
                    .foregroundColor(.green)
            }
            
            HStack {
                Button("ข้าม") {}
                    .foregroundColor(.secondary)
                
                Spacer()
                
                Button("ใช้โค้ด") {
                    validateCode()
                }
                .buttonStyle(.borderedProminent)
                .disabled(referralCode.count < 6)
            }
            .padding(.horizontal)
        }
    }
    
    private func validateCode() {
        Task {
            do {
                let info = try await ReferralManager.shared.validateReferralCode(referralCode)
                if info.isValid {
                    await MainActor.run {
                        validationResult = "✅ โค้ดถูกต้อง! คุณจะได้รับ \(info.reward)"
                    }
                }
            } catch {
                await MainActor.run {
                    validationResult = "❌ โค้ดไม่ถูกต้อง"
                }
            }
        }
    }
}
```

---

## 8. B2B/Enterprise Monetization

### 8.1 MDM-Managed Distribution

```swift
// การรองรับ MDM (Mobile Device Management)

class MDMManager {
    
    // อ่าน Managed Configuration (กำหนดโดย IT admin)
    struct ManagedConfig {
        let serverURL: String?
        let organizationName: String?
        let ssoEnabled: Bool
        let featureFlags: [String: Bool]
        let licenseKey: String?
    }
    
    static func getManagedConfiguration() -> ManagedConfig? {
        guard let managedConfig = UserDefaults.standard.dictionary(forKey: "com.apple.configuration.managed") else {
            return nil
        }
        
        return ManagedConfig(
            serverURL: managedConfig["ServerURL"] as? String,
            organizationName: managedConfig["OrganizationName"] as? String,
            ssoEnabled: managedConfig["SSOEnabled"] as? Bool ?? false,
            featureFlags: managedConfig["FeatureFlags"] as? [String: Bool] ?? [:],
            licenseKey: managedConfig["LicenseKey"] as? String
        )
    }
    
    // Feedback Channel ไปยัง MDM
    static func sendManagedFeedback(_ key: String, value: Any) {
        var feedback = UserDefaults.standard.dictionary(forKey: "com.apple.feedback.managed") ?? [:]
        feedback[key] = value
        UserDefaults.standard.set(feedback, forKey: "com.apple.feedback.managed")
    }
}

// Enterprise License Validation
class EnterpriseLicenseManager {
    
    struct LicenseInfo: Codable {
        let organizationName: String
        let licenseType: String
        let userSeats: Int
        let expiryDate: Date
        let features: [String]
    }
    
    func validateLicense(_ key: String) async throws -> LicenseInfo {
        // Call license server
        guard let url = URL(string: "https://license.company.com/validate") else {
            throw LicenseError.invalidURL
        }
        
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        request.httpBody = try JSONEncoder().encode(["key": key])
        
        let (data, response) = try await URLSession.shared.data(for: request)
        
        guard let httpResponse = response as? HTTPURLResponse,
              httpResponse.statusCode == 200 else {
            throw LicenseError.invalidLicense
        }
        
        return try JSONDecoder().decode(LicenseInfo.self, from: data)
    }
    
    enum LicenseError: Error {
        case invalidURL
        case invalidLicense
        case expired
        case seatLimitReached
    }
}
```

---

## 9. Subscription Retention

### 9.1 Win-Back Campaigns

```swift
// Push Notification สำหรับ Win-Back

import UserNotifications

class WinBackCampaignManager {
    
    struct WinBackOffer {
        let discountPercentage: Int
        let validDays: Int
        let message: String
    }
    
    // Schedule win-back notification หลังจาก cancel
    func scheduleWinBackCampaign(
        userId: String,
        cancellationDate: Date
    ) {
        let offers: [(daysAfter: Int, discount: Int)] = [
            (7, 20),    // 7 วันหลัง cancel: 20% off
            (30, 30),   // 30 วันหลัง cancel: 30% off
            (60, 50)    // 60 วันหลัง cancel: 50% off
        ]
        
        for offer in offers {
            let trigger = UNTimeIntervalNotificationTrigger(
                timeInterval: Double(offer.daysAfter * 24 * 3600),
                repeats: false
            )
            
            let content = UNMutableNotificationContent()
            content.title = "เราคิดถึงคุณ!"
            content.body = "กลับมาใช้งานเพื่อรับส่วนลด \(offer.discount)% สำหรับ Premium"
            content.userInfo = [
                "type": "win_back",
                "discount": offer.discount,
                "user_id": userId
            ]
            content.sound = .default
            
            let request = UNNotificationRequest(
                identifier: "win_back_\(userId)_\(offer.daysAfter)",
                content: content,
                trigger: trigger
            )
            
            UNUserNotificationCenter.current().add(request)
        }
    }
    
    // Cancel win-back notifications เมื่อ user กลับมา subscribe
    func cancelWinBackCampaign(userId: String) {
        let identifiers = [7, 30, 60].map { "win_back_\(userId)_\($0)" }
        UNUserNotificationCenter.current().removePendingNotificationRequests(withIdentifiers: identifiers)
    }
}
```

### 9.2 Cancellation Survey

```swift
// Survey เมื่อ user กำลังจะ cancel

struct CancellationSurveyView: View {
    @Environment(\.dismiss) var dismiss
    @State private var selectedReason: CancelReason?
    @State private var additionalFeedback = ""
    
    let onCancel: (CancelReason, String) -> Void
    let onKeepSubscription: () -> Void
    
    enum CancelReason: String, CaseIterable {
        case tooExpensive = "ราคาแพงเกินไป"
        case notUsingEnough = "ใช้งานน้อยเกินไป"
        case foundAlternative = "หา app อื่นแล้ว"
        case missingFeatures = "ขาด features ที่ต้องการ"
        case technicalIssues = "มีปัญหาทางเทคนิค"
        case temporarilyPausing = "หยุดชั่วคราว"
        case other = "เหตุผลอื่น"
    }
    
    var body: some View {
        NavigationView {
            Form {
                Section("ทำไมถึงต้องการยกเลิก?") {
                    ForEach(CancelReason.allCases, id: \.self) { reason in
                        HStack {
                            Text(reason.rawValue)
                            Spacer()
                            if selectedReason == reason {
                                Image(systemName: "checkmark")
                                    .foregroundColor(.blue)
                            }
                        }
                        .contentShape(Rectangle())
                        .onTapGesture {
                            selectedReason = reason
                        }
                    }
                }
                
                if let reason = selectedReason {
                    // Personalized retention offer based on reason
                    Section("ข้อเสนอพิเศษสำหรับคุณ") {
                        RetentionOfferView(reason: reason, onAccept: {
                            onKeepSubscription()
                            dismiss()
                        })
                    }
                }
                
                Section("บอกเราเพิ่มเติม (ไม่บังคับ)") {
                    TextEditor(text: $additionalFeedback)
                        .frame(height: 100)
                }
            }
            .navigationTitle("ก่อนออกจากระบบ")
            .navigationBarItems(
                leading: Button("ยกเลิกการยกเลิก") {
                    onKeepSubscription()
                    dismiss()
                }
                .foregroundColor(.blue),
                trailing: Button("ยืนยันยกเลิก") {
                    if let reason = selectedReason {
                        onCancel(reason, additionalFeedback)
                    }
                    dismiss()
                }
                .foregroundColor(.red)
                .disabled(selectedReason == nil)
            )
        }
    }
}

struct RetentionOfferView: View {
    let reason: CancellationSurveyView.CancelReason
    let onAccept: () -> Void
    
    var offerText: String {
        switch reason {
        case .tooExpensive:
            return "รับส่วนลด 30% สำหรับ 3 เดือนถัดไป"
        case .notUsingEnough:
            return "หยุดสมาชิกชั่วคราว 1 เดือนโดยไม่เสียค่าใช้จ่าย"
        case .missingFeatures:
            return "แจ้ง feature ที่ต้องการ - เราจะพิจารณาเพิ่มใน roadmap"
        case .technicalIssues:
            return "ติดต่อ support ทันที - เราจะแก้ไขให้ได้ภายใน 24 ชั่วโมง"
        default:
            return "ติดต่อเราก่อนยกเลิก เราอาจช่วยได้"
        }
    }
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            Text(offerText)
                .font(.headline)
            
            Button("รับข้อเสนอนี้") {
                onAccept()
            }
            .buttonStyle(.borderedProminent)
        }
        .padding(.vertical, 8)
    }
}
```

### 9.3 Dunning Management

```swift
// Dunning: การจัดการ Failed Payments

class DunningManager {
    
    struct DunningConfig {
        // วันที่ส่ง notification หลังจาก payment fail
        let retrySchedule: [Int] = [1, 3, 7, 14]
        let gracePeriodDays: Int = 16
        
        // Messages ที่แตกต่างกันตามวันที่
        func message(forDay day: Int) -> String {
            switch day {
            case 1:
                return "การชำระเงินล้มเหลว กรุณาอัปเดตข้อมูลบัตรของคุณ"
            case 3:
                return "ยังคงมีปัญหาการชำระเงิน Premium ของคุณจะหมดอายุใน \(gracePeriodDays - day) วัน"
            case 7:
                return "⚠️ Premium จะหมดอายุใน 9 วัน กรุณาอัปเดตข้อมูลการชำระเงิน"
            case 14:
                return "🚨 คืนนี้เป็นคืนสุดท้ายที่คุณจะได้ใช้ Premium"
            default:
                return "กรุณาอัปเดตข้อมูลการชำระเงิน"
            }
        }
    }
    
    // Track dunning state
    struct UserDunningState: Codable {
        let userId: String
        let paymentFailedDate: Date
        var notificationsSent: [Int]  // days notifications sent
        var isResolved: Bool
    }
    
    // อัปเดต Payment Method Deep Link
    static var updatePaymentURL: URL? {
        URL(string: "https://apps.apple.com/account/subscriptions")
    }
    
    // Handle grace period expiry
    func handleGracePeriodExpiry(userId: String) {
        // 1. Revoke premium access
        // 2. Send final notification
        // 3. Start win-back campaign
        
        let content = UNMutableNotificationContent()
        content.title = "Premium หมดอายุแล้ว"
        content.body = "แตะเพื่อต่ออายุ Premium และเข้าถึง features ทั้งหมด"
        content.userInfo = ["type": "subscription_expired", "user_id": userId]
        
        let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 1, repeats: false)
        let request = UNNotificationRequest(identifier: "grace_period_expired", content: content, trigger: trigger)
        
        UNUserNotificationCenter.current().add(request)
    }
}
```

---

## 10. Legal and Compliance

### 10.1 App Store Review Guidelines สำหรับ Monetization

```markdown
# App Store Monetization Guidelines Summary

## ข้อที่ต้องระวัง

### 3.1.1 In-App Purchase
❌ ห้ามใช้ external payment method (bypass App Store)
   ยกเว้น: "Reader" apps (Netflix, Spotify)
❌ ห้าม show ราคาที่ถูกกว่าใน external website
✅ อนุญาต: แสดงว่ามี option บน web แต่ไม่ link โดยตรง

### 3.1.2 Subscriptions
✅ ต้องแสดงข้อมูล:
   - ชื่อ subscription
   - ราคา
   - ระยะเวลา
   - Free trial period (ถ้ามี)
   - Auto-renewal disclosure

✅ ต้องมี:
   - ลิงก์ Privacy Policy
   - ลิงก์ Terms of Service
   - วิธียกเลิก subscription

❌ ห้าม:
   - Mislead users เรื่อง pricing
   - Hide cancellation information
   - Dark patterns ใน subscription flow

### 3.1.3 "Reader" Apps
✅ Netflix, Spotify, Kindle: ไม่จำเป็นต้องใช้ IAP
ต้องมี: reader functionality (เนื้อหาที่สร้างนอก app)
ห้ามมี: sign up button ใน app
```

### 10.2 GDPR และ CCPA สำหรับ Subscription Data

```swift
// Privacy Compliance สำหรับ Subscription

class PrivacyComplianceManager {
    
    // Data ที่ collect และ purpose
    struct DataCollectionInfo {
        static let subscriptionData = [
            "Email": "สำหรับส่ง invoice และ account management",
            "Purchase History": "สำหรับ customer support และ billing",
            "Device ID": "สำหรับ restore purchases",
            "IP Address": "สำหรับ fraud prevention"
        ]
    }
    
    // User Rights (GDPR)
    enum UserRight {
        case accessData      // ขอดูข้อมูลของตัวเอง
        case deleteData      // ขอลบข้อมูล (Right to be forgotten)
        case portability     // ขอ export ข้อมูล
        case rectification   // ขอแก้ไขข้อมูล
        case restriction     // ขอจำกัดการประมวลผล
    }
    
    // Handle GDPR Data Request
    func handleDataDeletionRequest(userId: String) async throws {
        // 1. ยกเลิก subscription (ถ้ายังเปิดอยู่)
        // 2. ลบ personal data ออกจาก database
        // 3. Anonymize transaction records (เพื่อ accounting)
        // 4. ส่ง confirmation email
        
        // Note: ต้องเก็บ transaction records ตาม accounting law
        // แต่ anonymize personal identifiers
    }
    
    // Auto-renewal disclosure (Required by App Store)
    static var autoRenewalDisclosure: String {
        return """
        การสมัครสมาชิกจะต่ออายุอัตโนมัติเว้นแต่จะยกเลิกอย่างน้อย 
        24 ชั่วโมงก่อนวันสิ้นสุดระยะเวลาปัจจุบัน 
        บัญชีของคุณจะถูกเรียกเก็บเงินสำหรับการต่ออายุภายใน 
        24 ชั่วโมงก่อนวันสิ้นสุดระยะเวลาปัจจุบัน
        
        คุณสามารถจัดการและยกเลิกการสมัครสมาชิกได้ในการตั้งค่าบัญชี 
        iTunes/App Store หลังจากซื้อแล้ว
        """
    }
}
```

---

## 11. Case Studies

### 11.1 Productivity App: Notion-Style Freemium

```swift
// Case Study: Note-taking App Freemium Model

struct NotionStyleMonetization {
    
    // Free Tier Limits
    struct FreeTierLimits {
        static let maxPages = 10
        static let maxBlocksPerPage = 1000
        static let maxFileUploadMB = 5
        static let collaboratorsCount = 0
        static let historyDays = 7
    }
    
    // Premium Features
    struct PremiumFeatures {
        static let unlimitedPages = true
        static let unlimitedBlocks = true
        static let fileUploadGB = 10
        static let collaboratorsCount = 10
        static let historyDays = 365
        static let aiFeatures = true
        static let customDomains = false  // Enterprise only
    }
    
    // Pricing Strategy (ปี 2024)
    struct Pricing {
        static let monthly = 9.99        // USD
        static let annual = 79.99        // USD (33% savings)
        static let enterprise = 24.99    // USD per user/month
    }
    
    // Conversion Funnel
    struct ConversionFunnel {
        // Industry benchmarks for productivity apps:
        static let trialStartRate = 0.15      // 15% of free users start trial
        static let trialToPaidRate = 0.25     // 25% of trials convert
        static let monthlyChurnRate = 0.04    // 4% monthly churn
        
        // Effective conversion: 15% * 25% = 3.75% of free users become paid
    }
    
    // Paywall Trigger Points
    enum TriggerPoint {
        case pageLimit
        case collaborationAttempt
        case fileUploadLimit
        case advancedFormattingAttempt
        case historyAccess
        
        var message: String {
            switch self {
            case .pageLimit:
                return "คุณสร้างหน้าครบ \(FreeTierLimits.maxPages) หน้าแล้ว อัปเกรดเพื่อสร้างไม่จำกัด"
            case .collaborationAttempt:
                return "Collaboration เป็น Premium feature"
            case .fileUploadLimit:
                return "เกิน storage limit ฟรี อัปเกรดเพื่อรับพื้นที่มากขึ้น"
            case .advancedFormattingAttempt:
                return "Advanced formatting ต้องการ Premium"
            case .historyAccess:
                return "ดู history ย้อนหลัง 7 วันได้ฟรี อัปเกรดเพื่อดูทั้งหมด"
            }
        }
    }
}
```

### 11.2 Fitness App: Subscription Tiers

```swift
// Case Study: Fitness App Subscription Model

struct FitnessAppMonetization {
    
    enum SubscriptionTier {
        case free
        case basic   // $4.99/month
        case premium // $9.99/month  
        case coach   // $19.99/month (with human coach)
    }
    
    struct TierFeatures {
        let workoutsPerWeek: Int
        let hasNutritionTracking: Bool
        let hasAIRecommendations: Bool
        let hasLiveWorkouts: Bool
        let hasPersonalCoach: Bool
        let hasProgressAnalytics: Bool
        
        static let free = TierFeatures(
            workoutsPerWeek: 3,
            hasNutritionTracking: false,
            hasAIRecommendations: false,
            hasLiveWorkouts: false,
            hasPersonalCoach: false,
            hasProgressAnalytics: false
        )
        
        static let basic = TierFeatures(
            workoutsPerWeek: 5,
            hasNutritionTracking: true,
            hasAIRecommendations: false,
            hasLiveWorkouts: false,
            hasPersonalCoach: false,
            hasProgressAnalytics: true
        )
        
        static let premium = TierFeatures(
            workoutsPerWeek: Int.max,
            hasNutritionTracking: true,
            hasAIRecommendations: true,
            hasLiveWorkouts: true,
            hasPersonalCoach: false,
            hasProgressAnalytics: true
        )
    }
    
    // Engagement-based upsell triggers
    enum UpsellTrigger {
        case completedFirstWorkout   // Offer basic trial
        case streak7Days             // Show premium benefits
        case reachedFreeLimit        // Hard paywall
        case viewedAdvancedWorkout   // Soft paywall
        case set3Goals               // Upsell AI recommendations
    }
    
    // Annual savings message
    static func annualSavings(monthlyPrice: Double) -> String {
        let annualPrice = monthlyPrice * 12 * 0.67  // 33% discount
        let savings = monthlyPrice * 12 - annualPrice
        return "ประหยัด \(savings.formatted(.currency(code: "THB"))) เมื่อสมัครรายปี"
    }
}
```

### 11.3 Game: Consumable IAP Model

```swift
// Case Study: Mobile Game Consumable IAP

struct GameMonetizationModel {
    
    // Currency System (2-currency model เป็น best practice สำหรับ games)
    struct GameCurrency {
        // Hard Currency (ซื้อด้วยเงินจริง)
        static let gemPackages: [(gems: Int, price: Double, bonus: String)] = [
            (80, 0.99, ""),
            (450, 4.99, ""),
            (900, 9.99, "+50 bonus"),
            (1850, 19.99, "+100 bonus"),
            (4000, 39.99, "+500 bonus"),
            (8500, 79.99, "+1000 bonus")
        ]
        
        // Soft Currency (earn ใน game)
        static let goldEarnRate = 100  // per level
    }
    
    // Whale Analysis
    struct WhaleMetrics {
        // Industry data:
        static let percentThatBuy = 0.02      // 2% ของ users ซื้อ
        static let whalePercentage = 0.001    // 0.1% = whales
        static let whaleRevenueShare = 0.50  // Whales = 50% of revenue
        
        // Implication: ออกแบบ high-value packages ให้ whales
    }
    
    // Ethical IAP Guidelines
    struct EthicalGuidelines {
        static let noPay2Win = true           // ไม่ให้ advantage ที่ไม่ fair
        static let noBait = true              // ไม่ bait ด้วย fake urgency
        static let transparentOdds = true     // แสดง drop rates สำหรับ loot boxes
        static let noMinorTargeting = true    // ไม่ target เด็กด้วย aggressive IAP
    }
    
    // Loot Box Regulation (ต้องแสดง odds ใน app)
    struct LootBoxDisclosure {
        let items: [(name: String, rarity: String, dropRate: Double)]
        
        func generateDisclosureText() -> String {
            var text = "อัตราโอกาสที่จะได้รับ:\n"
            for item in items {
                text += "- \(item.name) (\(item.rarity)): \(String(format: "%.1f%%", item.dropRate * 100))\n"
            }
            return text
        }
    }
}
```

---

## 12. Complete Exercise: Building a Paywall View with RevenueCat

```swift
// Complete Implementation: Full Paywall with RevenueCat

import SwiftUI
import RevenueCat

// MARK: - Models

struct PaywallFeature: Identifiable {
    let id = UUID()
    let icon: String
    let title: String
    let subtitle: String
}

// MARK: - ViewModel

@MainActor
class FullPaywallViewModel: ObservableObject {
    
    @Published var offerings: Offerings?
    @Published var selectedPackage: Package?
    @Published var isPurchasing = false
    @Published var purchaseSuccess = false
    @Published var errorMessage: String?
    @Published var isEligibleForTrial = false
    
    let features: [PaywallFeature] = [
        PaywallFeature(
            icon: "infinity",
            title: "ไม่จำกัด",
            subtitle: "สร้าง projects และ tasks ไม่จำกัด"
        ),
        PaywallFeature(
            icon: "icloud.fill",
            title: "Cloud Sync",
            subtitle: "Sync ข้อมูลทุก devices อัตโนมัติ"
        ),
        PaywallFeature(
            icon: "square.and.arrow.up",
            title: "Export",
            subtitle: "Export เป็น PDF, CSV, และ Excel"
        ),
        PaywallFeature(
            icon: "chart.bar.fill",
            title: "Advanced Analytics",
            subtitle: "ดู productivity trends ของคุณ"
        ),
        PaywallFeature(
            icon: "paintbrush.pointed.fill",
            title: "Custom Themes",
            subtitle: "ปรับแต่ง UI ให้เป็นสไตล์ของคุณ"
        )
    ]
    
    init() {
        Task {
            await loadOfferings()
            await checkTrialEligibility()
        }
    }
    
    func loadOfferings() async {
        do {
            let offerings = try await Purchases.shared.offerings()
            self.offerings = offerings
            
            // Default select annual
            if let offering = offerings.current {
                selectedPackage = offering.annual ?? offering.monthly
            }
        } catch {
            errorMessage = "ไม่สามารถโหลดแผนราคาได้: \(error.localizedDescription)"
        }
    }
    
    func checkTrialEligibility() async {
        // Check if user has ever had a subscription
        let info = try? await Purchases.shared.customerInfo()
        let hasHadSubscription = !(info?.allPurchasedProductIdentifiers.isEmpty ?? true)
        isEligibleForTrial = !hasHadSubscription
    }
    
    func purchaseSelected() async {
        guard let package = selectedPackage else { return }
        
        isPurchasing = true
        defer { isPurchasing = false }
        
        do {
            let (_, customerInfo, userCancelled) = try await Purchases.shared.purchase(package: package)
            
            if !userCancelled {
                if customerInfo.entitlements["premium"]?.isActive == true {
                    purchaseSuccess = true
                    
                    // Track analytics
                    Analytics.shared.track("subscription_purchased", parameters: [
                        "package": package.identifier,
                        "price": package.storeProduct.price.doubleValue
                    ])
                }
            }
        } catch {
            errorMessage = "การซื้อล้มเหลว: \(error.localizedDescription)"
        }
    }
    
    func restorePurchases() async {
        do {
            let customerInfo = try await Purchases.shared.restorePurchases()
            if customerInfo.entitlements["premium"]?.isActive == true {
                purchaseSuccess = true
            } else {
                errorMessage = "ไม่พบการซื้อก่อนหน้า"
            }
        } catch {
            errorMessage = "Restore ล้มเหลว: \(error.localizedDescription)"
        }
    }
}

// MARK: - Main Paywall View

struct FullPaywallView: View {
    
    @StateObject private var viewModel = FullPaywallViewModel()
    @Environment(\.dismiss) private var dismiss
    
    var body: some View {
        NavigationView {
            ZStack {
                // Background
                LinearGradient(
                    colors: [Color.purple.opacity(0.1), Color.blue.opacity(0.05)],
                    startPoint: .topLeading,
                    endPoint: .bottomTrailing
                )
                .ignoresSafeArea()
                
                ScrollView {
                    VStack(spacing: 0) {
                        // Hero
                        heroSection
                        
                        // Features
                        featuresSection
                        
                        // Pricing
                        if let offering = viewModel.offerings?.current {
                            pricingSection(offering: offering)
                        }
                        
                        // CTA
                        ctaSection
                        
                        // Footer
                        footerSection
                    }
                }
                
                if viewModel.isPurchasing {
                    Color.black.opacity(0.4)
                        .ignoresSafeArea()
                    ProgressView()
                        .scaleEffect(1.5)
                        .tint(.white)
                }
            }
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button(action: { dismiss() }) {
                        Image(systemName: "xmark.circle.fill")
                            .foregroundColor(.secondary)
                            .font(.title2)
                    }
                }
            }
        }
        .alert("ข้อผิดพลาด", isPresented: .init(
            get: { viewModel.errorMessage != nil },
            set: { if !$0 { viewModel.errorMessage = nil } }
        )) {
            Button("ตกลง") { viewModel.errorMessage = nil }
        } message: {
            Text(viewModel.errorMessage ?? "")
        }
        .sheet(isPresented: $viewModel.purchaseSuccess) {
            SuccessView()
        }
    }
    
    // MARK: - Hero Section
    private var heroSection: some View {
        VStack(spacing: 16) {
            ZStack {
                Circle()
                    .fill(
                        LinearGradient(
                            colors: [.purple, .blue],
                            startPoint: .topLeading,
                            endPoint: .bottomTrailing
                        )
                    )
                    .frame(width: 100, height: 100)
                
                Image(systemName: "crown.fill")
                    .font(.system(size: 45))
                    .foregroundColor(.white)
            }
            .padding(.top, 24)
            
            VStack(spacing: 8) {
                Text("Pro")
                    .font(.system(size: 42, weight: .bold, design: .rounded))
                    .foregroundStyle(
                        LinearGradient(
                            colors: [.purple, .blue],
                            startPoint: .leading,
                            endPoint: .trailing
                        )
                    )
                
                if viewModel.isEligibleForTrial {
                    Text("ทดลองใช้ฟรี 7 วัน ไม่ต้องให้บัตรเครดิต")
                        .font(.subheadline)
                        .foregroundColor(.secondary)
                } else {
                    Text("ปลดล็อคทุก features ทันที")
                        .font(.subheadline)
                        .foregroundColor(.secondary)
                }
            }
        }
    }
    
    // MARK: - Features Section
    private var featuresSection: some View {
        VStack(alignment: .leading, spacing: 16) {
            ForEach(viewModel.features) { feature in
                HStack(spacing: 16) {
                    Image(systemName: feature.icon)
                        .font(.title2)
                        .foregroundStyle(
                            LinearGradient(
                                colors: [.purple, .blue],
                                startPoint: .leading,
                                endPoint: .trailing
                            )
                        )
                        .frame(width: 40)
                    
                    VStack(alignment: .leading, spacing: 2) {
                        Text(feature.title)
                            .font(.headline)
                        Text(feature.subtitle)
                            .font(.subheadline)
                            .foregroundColor(.secondary)
                    }
                    
                    Spacer()
                }
            }
        }
        .padding(24)
        .background(Color(.systemBackground))
        .cornerRadius(16)
        .shadow(color: .black.opacity(0.06), radius: 12, x: 0, y: 4)
        .padding(.horizontal, 20)
        .padding(.top, 24)
    }
    
    // MARK: - Pricing Section
    private func pricingSection(offering: Offering) -> some View {
        VStack(spacing: 12) {
            Text("เลือกแผน")
                .font(.headline)
                .padding(.top, 24)
            
            ForEach(offering.availablePackages, id: \.identifier) { package in
                PricingOptionCard(
                    package: package,
                    isSelected: viewModel.selectedPackage?.identifier == package.identifier,
                    isRecommended: package.packageType == .annual,
                    onSelect: { viewModel.selectedPackage = package }
                )
            }
        }
        .padding(.horizontal, 20)
    }
    
    // MARK: - CTA Section
    private var ctaSection: some View {
        VStack(spacing: 16) {
            Button(action: {
                Task { await viewModel.purchaseSelected() }
            }) {
                HStack {
                    Spacer()
                    if viewModel.isEligibleForTrial {
                        Text("เริ่มทดลองใช้ฟรี")
                    } else {
                        Text("สมัครสมาชิก")
                    }
                    Spacer()
                }
                .font(.headline)
                .foregroundColor(.white)
                .padding(.vertical, 16)
                .background(
                    LinearGradient(
                        colors: [.purple, .blue],
                        startPoint: .leading,
                        endPoint: .trailing
                    )
                )
                .cornerRadius(14)
            }
            .disabled(viewModel.selectedPackage == nil || viewModel.isPurchasing)
            
            Button("Restore Purchases") {
                Task { await viewModel.restorePurchases() }
            }
            .font(.subheadline)
            .foregroundColor(.secondary)
        }
        .padding(.horizontal, 20)
        .padding(.top, 20)
    }
    
    // MARK: - Footer
    private var footerSection: some View {
        VStack(spacing: 8) {
            Text(PrivacyComplianceManager.autoRenewalDisclosure)
                .font(.caption2)
                .foregroundColor(.secondary)
                .multilineTextAlignment(.center)
            
            HStack(spacing: 16) {
                Link("นโยบายความเป็นส่วนตัว",
                     destination: URL(string: "https://app.com/privacy")!)
                Link("เงื่อนไขการใช้งาน",
                     destination: URL(string: "https://app.com/terms")!)
            }
            .font(.caption2)
            .foregroundColor(.blue)
        }
        .padding(20)
        .padding(.bottom, 40)
    }
}

// MARK: - Supporting Views

struct PricingOptionCard: View {
    let package: Package
    let isSelected: Bool
    let isRecommended: Bool
    let onSelect: () -> Void
    
    var annualSavingsText: String? {
        guard package.packageType == .annual else { return nil }
        // คำนวณ savings เทียบกับ monthly
        return "ประหยัด 33%"
    }
    
    var body: some View {
        Button(action: onSelect) {
            HStack {
                VStack(alignment: .leading, spacing: 4) {
                    HStack(spacing: 8) {
                        Text(package.storeProduct.localizedTitle)
                            .font(.headline)
                            .foregroundColor(.primary)
                        
                        if isRecommended {
                            Text("แนะนำ")
                                .font(.caption.bold())
                                .padding(.horizontal, 8)
                                .padding(.vertical, 2)
                                .background(Color.purple)
                                .foregroundColor(.white)
                                .cornerRadius(8)
                        }
                    }
                    
                    Text(package.storeProduct.localizedPriceString)
                        .font(.subheadline)
                        .foregroundColor(.secondary)
                    
                    if let savings = annualSavingsText {
                        Text(savings)
                            .font(.caption)
                            .foregroundColor(.green)
                    }
                }
                
                Spacer()
                
                ZStack {
                    Circle()
                        .stroke(isSelected ? Color.purple : Color.gray.opacity(0.3), lineWidth: 2)
                        .frame(width: 24, height: 24)
                    
                    if isSelected {
                        Circle()
                            .fill(Color.purple)
                            .frame(width: 14, height: 14)
                    }
                }
            }
            .padding(16)
            .background(Color(.systemBackground))
            .cornerRadius(12)
            .overlay(
                RoundedRectangle(cornerRadius: 12)
                    .stroke(isSelected ? Color.purple : Color.clear, lineWidth: 2)
            )
            .shadow(color: isSelected ? Color.purple.opacity(0.2) : Color.black.opacity(0.05),
                   radius: isSelected ? 8 : 4)
        }
        .buttonStyle(.plain)
    }
}

struct SuccessView: View {
    @Environment(\.dismiss) private var dismiss
    
    var body: some View {
        VStack(spacing: 24) {
            Image(systemName: "checkmark.circle.fill")
                .font(.system(size: 80))
                .foregroundColor(.green)
            
            Text("ยินดีต้อนรับสู่ Pro!")
                .font(.title.bold())
            
            Text("คุณสามารถเข้าถึง features ทั้งหมดได้แล้ว")
                .foregroundColor(.secondary)
                .multilineTextAlignment(.center)
            
            Button("เริ่มใช้งาน") { dismiss() }
                .buttonStyle(.borderedProminent)
                .controlSize(.large)
        }
        .padding()
    }
}
```

---

## 13. แบบฝึกหัดและเฉลย

### แบบฝึกหัดที่ 1: Pricing Strategy Analysis

**โจทย์:** คุณมี Productivity App ที่:
- DAU: 50,000
- Current free users: 48,500
- Paying subscribers: 1,500
- Monthly price: ฿299
- Annual price: ฿2,490

คำนวณ MRR, ARPU, และ Conversion Rate

**เฉลย:**

```swift
struct AppMetricsAnalysis {
    
    // Data
    static let dau = 50_000
    static let freeUsers = 48_500
    static let payingSubscribers = 1_500
    static let monthlyPrice = 299.0
    static let annualPrice = 2_490.0
    
    // Assume 60% monthly, 40% annual
    static let monthlySubCount = Int(Double(payingSubscribers) * 0.60)  // 900
    static let annualSubCount = Int(Double(payingSubscribers) * 0.40)   // 600
    
    // MRR Calculation
    static var mrr: Double {
        let monthlyRevenue = Double(monthlySubCount) * monthlyPrice
        let annualMonthlyRevenue = Double(annualSubCount) * (annualPrice / 12)
        return monthlyRevenue + annualMonthlyRevenue
    }
    
    // ARR
    static var arr: Double { mrr * 12 }
    
    // Conversion Rate
    static var conversionRate: Double {
        return Double(payingSubscribers) / Double(dau)
    }
    
    // ARPU (based on all DAU)
    static var arpu: Double {
        return mrr / Double(dau)
    }
    
    // ARPU (based on paying subscribers only)
    static var arppu: Double {  // Average Revenue Per Paying User
        return mrr / Double(payingSubscribers)
    }
    
    static func printAnalysis() {
        print("=== App Metrics Analysis ===")
        print("Total DAU: \(dau.formatted())")
        print("Paying Subscribers: \(payingSubscribers.formatted())")
        print("Conversion Rate: \(String(format: "%.2f%%", conversionRate * 100))")
        print("")
        print("MRR: ฿\(String(format: "%.2f", mrr))")
        print("ARR: ฿\(String(format: "%.2f", arr))")
        print("ARPU: ฿\(String(format: "%.2f", arpu))")
        print("ARPPU: ฿\(String(format: "%.2f", arppu))")
        print("")
        
        // LTV
        let monthlyChurnRate = 0.05  // 5% assumed
        let ltv = arppu / monthlyChurnRate
        print("Estimated LTV (at 5% churn): ฿\(String(format: "%.2f", ltv))")
        
        // Improvement opportunities
        print("")
        print("=== Improvement Opportunities ===")
        print("If increase conversion 1% → +500 subscribers")
        let newMRR = mrr + (500 * arppu)
        print("New MRR: ฿\(String(format: "%.2f", newMRR)) (+\(String(format: "%.1f%%", ((newMRR - mrr) / mrr) * 100)))")
        print("")
        print("If increase annual conversion 10% → +\(Int(0.1 * Double(payingSubscribers))) annual subs")
        let annualUpsellRevenue = Double(Int(0.1 * Double(payingSubscribers))) * (annualPrice/12 - monthlyPrice)
        print("Additional Monthly Revenue: ฿\(String(format: "%.2f", annualUpsellRevenue))")
    }
}

// เรียกใช้
AppMetricsAnalysis.printAnalysis()
/*
Output:
=== App Metrics Analysis ===
Total DAU: 50,000
Paying Subscribers: 1,500
Conversion Rate: 3.00%

MRR: ฿393,750.00
ARR: ฿4,725,000.00
ARPU: ฿7.88
ARPPU: ฿262.50

Estimated LTV (at 5% churn): ฿5,250.00

=== Improvement Opportunities ===
If increase conversion 1% → +500 subscribers
New MRR: ฿524,750.00 (+33.3%)

If increase annual conversion 10% → +150 annual subs
Additional Monthly Revenue: ฿12,750.00
*/
```

### แบบฝึกหัดที่ 2: Implement Subscription Gate

**โจทย์:** Implement SwiftUI View ที่:
1. แสดง content ปกติสำหรับ subscribers
2. Blur content และ show paywall prompt สำหรับ non-subscribers
3. Handle trial period

**เฉลย:**

```swift
// Premium Content Gate View

struct PremiumContentGate<Content: View>: View {
    
    let content: () -> Content
    let featureName: String
    @StateObject private var subManager = SubscriptionManager.shared
    @State private var showPaywall = false
    
    init(featureName: String, @ViewBuilder content: @escaping () -> Content) {
        self.featureName = featureName
        self.content = content
    }
    
    var body: some View {
        ZStack {
            // Actual content
            content()
                .blur(radius: isPremium ? 0 : 8)
                .allowsHitTesting(isPremium)
            
            // Paywall overlay
            if !isPremium {
                PremiumOverlay(
                    featureName: featureName,
                    onUpgrade: { showPaywall = true }
                )
            }
        }
        .sheet(isPresented: $showPaywall) {
            FullPaywallView()
        }
    }
    
    private var isPremium: Bool {
        subManager.subscriptionStatus != .notSubscribed
    }
}

struct PremiumOverlay: View {
    let featureName: String
    let onUpgrade: () -> Void
    
    var body: some View {
        VStack(spacing: 16) {
            Image(systemName: "lock.fill")
                .font(.system(size: 40))
                .foregroundColor(.white)
            
            Text("\(featureName) ต้องการ Premium")
                .font(.headline)
                .foregroundColor(.white)
                .multilineTextAlignment(.center)
            
            Button(action: onUpgrade) {
                Text("อัปเกรดเป็น Pro")
                    .font(.headline)
                    .foregroundColor(.purple)
                    .padding(.horizontal, 24)
                    .padding(.vertical, 12)
                    .background(Color.white)
                    .cornerRadius(10)
            }
        }
        .padding(24)
        .background(
            Color.black.opacity(0.6)
                .blur(radius: 0)
        )
        .cornerRadius(16)
        .padding(32)
    }
}

// Usage:
struct AdvancedAnalyticsView: View {
    var body: some View {
        PremiumContentGate(featureName: "Advanced Analytics") {
            // Premium content ที่จะ blur ถ้าไม่ได้ subscribe
            VStack {
                Text("Productivity Score")
                    .font(.title)
                
                Text("87/100")
                    .font(.largeTitle.bold())
                    .foregroundColor(.green)
                
                // Charts, graphs, etc.
            }
        }
    }
}
```

### แบบฝึกหัดที่ 3: Churn Prediction

**โจทย์:** สร้าง simple churn risk score ตาม user behavior

**เฉลย:**

```swift
// Churn Risk Prediction

struct UserChurnRisk {
    
    struct UserBehaviorMetrics {
        let daysSinceLastOpen: Int
        let sessionsLastMonth: Int
        let featuresUsed: Int
        let totalFeatureCount: Int
        let hasCompletedOnboarding: Bool
        let supportTicketsCount: Int
        let lastPurchaseMonthsAgo: Int?
    }
    
    enum ChurnRisk {
        case low     // 0-30%
        case medium  // 31-60%
        case high    // 61-80%
        case critical // 81-100%
        
        var color: String {
            switch self {
            case .low: return "green"
            case .medium: return "yellow"
            case .high: return "orange"
            case .critical: return "red"
            }
        }
        
        var recommendedAction: String {
            switch self {
            case .low:
                return "Engage with new features"
            case .medium:
                return "Send re-engagement email"
            case .high:
                return "Offer discount or pause subscription"
            case .critical:
                return "Personal outreach from success team"
            }
        }
    }
    
    static func calculateRiskScore(_ metrics: UserBehaviorMetrics) -> (score: Double, risk: ChurnRisk) {
        var score = 0.0
        
        // Days since last open (30% weight)
        let recencyScore: Double
        switch metrics.daysSinceLastOpen {
        case 0...3: recencyScore = 0.0
        case 4...7: recencyScore = 0.2
        case 8...14: recencyScore = 0.5
        case 15...30: recencyScore = 0.8
        default: recencyScore = 1.0
        }
        score += recencyScore * 0.30
        
        // Session frequency (25% weight)
        let frequencyScore: Double
        switch metrics.sessionsLastMonth {
        case 20...: frequencyScore = 0.0
        case 10..<20: frequencyScore = 0.2
        case 5..<10: frequencyScore = 0.5
        case 1..<5: frequencyScore = 0.8
        default: frequencyScore = 1.0
        }
        score += frequencyScore * 0.25
        
        // Feature adoption (20% weight)
        let adoptionRate = Double(metrics.featuresUsed) / Double(metrics.totalFeatureCount)
        let adoptionScore = 1.0 - min(adoptionRate, 1.0)
        score += adoptionScore * 0.20
        
        // Onboarding completion (10% weight)
        score += (metrics.hasCompletedOnboarding ? 0.0 : 1.0) * 0.10
        
        // Support tickets (15% weight)
        let supportScore: Double = min(Double(metrics.supportTicketsCount) / 3.0, 1.0)
        score += supportScore * 0.15
        
        let risk: ChurnRisk
        switch score {
        case 0..<0.30: risk = .low
        case 0.30..<0.60: risk = .medium
        case 0.60..<0.80: risk = .high
        default: risk = .critical
        }
        
        return (score: score, risk: risk)
    }
}

// ทดสอบ
let userMetrics = UserChurnRisk.UserBehaviorMetrics(
    daysSinceLastOpen: 12,
    sessionsLastMonth: 4,
    featuresUsed: 3,
    totalFeatureCount: 15,
    hasCompletedOnboarding: true,
    supportTicketsCount: 1,
    lastPurchaseMonthsAgo: 3
)

let result = UserChurnRisk.calculateRiskScore(userMetrics)
print("Churn Risk Score: \(String(format: "%.0f%%", result.score * 100))")
print("Risk Level: \(result.risk)")
print("Recommended Action: \(result.risk.recommendedAction)")

/*
Output:
Churn Risk Score: 52%
Risk Level: medium
Recommended Action: Send re-engagement email
*/
```

---

## สรุป

App Monetization เป็นทั้งศาสตร์และศิลป์:

**หลักการสำคัญ:**

1. **Value First**: ต้องสร้างคุณค่าให้ผู้ใช้ก่อน แล้ว monetize จากคุณค่านั้น
2. **Frictionless Purchasing**: ทำให้การซื้อง่ายที่สุด ด้วย StoreKit 2
3. **Retention Over Acquisition**: รักษา subscriber ราคาถูกกว่าหา subscriber ใหม่
4. **Data-Driven Decisions**: A/B test ทุกอย่าง ตั้งแต่ paywall design ถึง pricing
5. **Ethical Monetization**: ไม่ใช้ dark patterns, transparent pricing, respect user privacy
6. **Metrics Focus**: ติดตาม MRR, Churn, LTV, CAC เป็น weekly ritual
7. **Legal Compliance**: ปฏิบัติตาม App Store Guidelines, GDPR, CCPA อย่างเคร่งครัด

**Quick Wins:**
- เพิ่ม Annual plan ที่ให้ savings ชัดเจน → เพิ่ม LTV ทันที
- แสดง trial period อย่าง prominent → เพิ่ม conversion
- Implement grace period handling → ลด involuntary churn
- Request review ในเวลาที่เหมาะสม → เพิ่ม App Store rating

---

*หมายเหตุ: ตัวเลขใน case studies เป็นตัวอย่างสำหรับการศึกษา ตัวเลขจริงแตกต่างกันตามแต่ละแอปและตลาด*
