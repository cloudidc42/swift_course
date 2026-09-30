# Part 52: StoreKit และ In-App Purchase

## บทนำ

In-App Purchase (IAP) คือระบบที่ให้ผู้ใช้ซื้อเนื้อหาหรือฟีเจอร์เพิ่มเติมภายในแอปโดยตรง Apple ได้พัฒนา StoreKit framework เพื่อจัดการ transactions ทั้งหมด StoreKit 2 ซึ่งเปิดตัวใน iOS 15 เป็น API สมัยใหม่ที่ใช้ Swift Concurrency ทำให้การ implement IAP ง่ายขึ้นมาก บทนี้จะครอบคลุมทุกด้านของ StoreKit ตั้งแต่พื้นฐานไปจนถึงการสร้างระบบ subscription ที่สมบูรณ์

---

## 1. In-App Purchase Types

Apple รองรับ IAP 4 ประเภทหลัก:

### 1.1 Consumable (สินค้าใช้แล้วหมด)

```swift
/*
Consumable IAP:
- ซื้อได้หลายครั้ง
- ใช้แล้วหมดไป
- ตัวอย่าง: เหรียญในเกม, ชีวิตเพิ่ม, boost
- ต้อง track state เอง (เช่น เก็บใน server)
- ไม่สามารถ restore ได้
*/

// ตัวอย่าง Product IDs
let consumableProducts = [
    "com.myapp.coins.100",
    "com.myapp.coins.500",
    "com.myapp.lives.3",
    "com.myapp.boost.speed"
]
```

### 1.2 Non-Consumable (สินค้าถาวร)

```swift
/*
Non-Consumable IAP:
- ซื้อครั้งเดียว ใช้ได้ตลอด
- ตัวอย่าง: Remove Ads, Premium Features, Extra Levels
- Restore ได้โดยอัตโนมัติ
- มีอยู่ตลอดชีพของ Apple ID
*/

let nonConsumableProducts = [
    "com.myapp.remove_ads",
    "com.myapp.premium_unlock",
    "com.myapp.extra_levels",
    "com.myapp.pro_tools"
]
```

### 1.3 Auto-Renewable Subscription (subscription ต่ออัตโนมัติ)

```swift
/*
Auto-Renewable Subscription:
- เรียกเก็บเงินซ้ำทุก period (weekly/monthly/yearly)
- ต่ออัตโนมัติจนกว่าจะยกเลิก
- ตัวอย่าง: Netflix, Spotify, Apple Music
- รองรับ introductory offers, promotional offers
- รองรับ Family Sharing
*/

let subscriptionProducts = [
    "com.myapp.premium.monthly",
    "com.myapp.premium.yearly",
    "com.myapp.basic.monthly"
]
```

### 1.4 Non-Renewing Subscription (subscription ไม่ต่ออัตโนมัติ)

```swift
/*
Non-Renewing Subscription:
- access ในช่วงเวลาที่กำหนด
- ไม่ต่ออัตโนมัติ - ผู้ใช้ต้องซื้อใหม่เอง
- ตัวอย่าง: season pass, annual sports subscription
- ต้อง track expiry date เอง
*/

let nonRenewingProducts = [
    "com.myapp.season_pass.2024",
    "com.myapp.annual_sports"
]
```

---

## 2. StoreKit 2 (Modern API)

StoreKit 2 ใช้ Swift Concurrency (async/await) และ Typed Transactions

### 2.1 Setup

```swift
import StoreKit

// Info.plist ต้องมี:
// NSAppTransportSecurity -> NSAllowsArbitraryLoads: false
// StoreKit Configuration ใช้ใน testing เท่านั้น

class StoreKitSetup {
    // ตรวจสอบว่า StoreKit 2 รองรับหรือไม่
    static func isAvailable() -> Bool {
        if #available(iOS 15.0, *) {
            return true
        }
        return false
    }
}
```

### 2.2 Store Manager

```swift
import StoreKit

@MainActor
class StoreManager: ObservableObject {
    @Published var products: [Product] = []
    @Published var purchasedProductIDs: Set<String> = []
    @Published var subscriptionStatus: Product.SubscriptionInfo.Status?
    
    private var transactionListener: Task<Void, Error>?
    
    // Product IDs ทั้งหมดของแอป
    private let productIDs: Set<String> = [
        "com.myapp.remove_ads",
        "com.myapp.premium.monthly",
        "com.myapp.premium.yearly",
        "com.myapp.coins.100",
        "com.myapp.coins.500"
    ]
    
    init() {
        // เริ่ม listen transactions
        transactionListener = listenForTransactions()
        
        // โหลด products และตรวจสอบ entitlements
        Task {
            await loadProducts()
            await updateCustomerProductStatus()
        }
    }
    
    deinit {
        transactionListener?.cancel()
    }
    
    // โหลด products จาก App Store
    func loadProducts() async {
        do {
            let storeProducts = try await Product.products(for: productIDs)
            products = storeProducts.sorted { $0.price < $1.price }
            print("โหลด \(products.count) products สำเร็จ")
        } catch {
            print("ไม่สามารถโหลด products: \(error)")
        }
    }
    
    // Listen for transactions ใหม่
    private func listenForTransactions() -> Task<Void, Error> {
        return Task.detached {
            // ฟัง updates ของ transactions
            for await result in Transaction.updates {
                do {
                    let transaction = try await self.checkVerified(result)
                    
                    // อัพเดต UI
                    await self.updateCustomerProductStatus()
                    
                    // Finish transaction
                    await transaction.finish()
                } catch {
                    print("Transaction verification ล้มเหลว: \(error)")
                }
            }
        }
    }
    
    // ตรวจสอบ verification result
    func checkVerified<T>(_ result: VerificationResult<T>) throws -> T {
        switch result {
        case .unverified:
            // Verification ล้มเหลว - ไม่ควร grant access
            throw StoreError.failedVerification
        case .verified(let transaction):
            // Verification สำเร็จ
            return transaction
        }
    }
    
    // อัพเดต purchased products
    func updateCustomerProductStatus() async {
        var purchasedIDs: Set<String> = []
        
        // ตรวจสอบ current entitlements
        for await result in Transaction.currentEntitlements {
            do {
                let transaction = try checkVerified(result)
                
                switch transaction.productType {
                case .nonConsumable:
                    purchasedIDs.insert(transaction.productID)
                    
                case .autoRenewable:
                    // ตรวจสอบว่า subscription ยังใช้งานได้
                    if transaction.revocationDate == nil {
                        purchasedIDs.insert(transaction.productID)
                    }
                    
                default:
                    break
                }
            } catch {
                print("ไม่สามารถ verify transaction: \(error)")
            }
        }
        
        purchasedProductIDs = purchasedIDs
    }
}

// Custom Errors
enum StoreError: Error {
    case failedVerification
    case productNotFound
    case purchaseFailed
}
```

---

## 3. Product.products(for:)

การดึงข้อมูล products จาก App Store

```swift
import StoreKit

class ProductLoader {
    // โหลด products หลายตัวพร้อมกัน
    func loadProducts(ids: Set<String>) async throws -> [Product] {
        let products = try await Product.products(for: ids)
        return products
    }
    
    // จัดกลุ่ม products ตาม type
    func categorizeProducts(_ products: [Product]) -> ProductCategories {
        var consumables: [Product] = []
        var nonConsumables: [Product] = []
        var subscriptions: [Product] = []
        var nonRenewing: [Product] = []
        
        for product in products {
            switch product.type {
            case .consumable:
                consumables.append(product)
            case .nonConsumable:
                nonConsumables.append(product)
            case .autoRenewable:
                subscriptions.append(product)
            case .nonRenewable:
                nonRenewing.append(product)
            default:
                break
            }
        }
        
        return ProductCategories(
            consumables: consumables,
            nonConsumables: nonConsumables,
            subscriptions: subscriptions,
            nonRenewing: nonRenewing
        )
    }
    
    // แสดงข้อมูล product
    func displayProductInfo(_ product: Product) {
        print("=== Product Info ===")
        print("ID: \(product.id)")
        print("Display Name: \(product.displayName)")
        print("Description: \(product.description)")
        print("Price: \(product.displayPrice)")  // "฿129"
        print("Type: \(product.type)")
        
        // Subscription info
        if let subscription = product.subscription {
            print("Subscription Period: \(subscription.subscriptionPeriod.unit) x \(subscription.subscriptionPeriod.value)")
            
            // Introductory offer
            if let introOffer = subscription.introductoryOffer {
                print("Introductory Offer: \(introOffer.displayPrice) for \(introOffer.period.value) \(introOffer.period.unit)")
                print("Introductory Type: \(introOffer.paymentMode)")
            }
            
            // Promotional offers
            for promoOffer in subscription.promotionalOffers {
                print("Promotional Offer: \(promoOffer.id)")
            }
        }
        
        // Subscription group
        if let groupID = product.subscription?.subscriptionGroupID {
            print("Subscription Group: \(groupID)")
        }
    }
}

struct ProductCategories {
    let consumables: [Product]
    let nonConsumables: [Product]
    let subscriptions: [Product]
    let nonRenewing: [Product]
}
```

---

## 4. Transaction

Transaction แทน record การซื้อที่ verified แล้ว

```swift
import StoreKit

class TransactionManager {
    // ดึง properties จาก Transaction
    func inspectTransaction(_ transaction: Transaction) {
        print("=== Transaction ===")
        print("ID: \(transaction.id)")
        print("Product ID: \(transaction.productID)")
        print("Product Type: \(transaction.productType)")
        print("Purchase Date: \(transaction.purchaseDate)")
        print("Original Purchase Date: \(transaction.originalPurchaseDate)")
        print("Original ID: \(transaction.originalID)")
        
        // Expiration (สำหรับ subscriptions)
        if let expirationDate = transaction.expirationDate {
            print("Expiration: \(expirationDate)")
            
            let isExpired = expirationDate < Date()
            print("Expired: \(isExpired)")
        }
        
        // Revocation (ถูกยกเลิกโดย Apple)
        if let revocationDate = transaction.revocationDate {
            print("Revoked at: \(revocationDate)")
            print("Revocation Reason: \(transaction.revocationReason?.rawValue ?? "unknown")")
        }
        
        // Upgrade/Downgrade
        if let upgradeDate = transaction.upgradedFromTransaction {
            print("Upgraded from transaction: \(upgradeDate)")
        }
        
        // Family sharing
        if let ownershipType = transaction.ownershipType {
            switch ownershipType {
            case .purchased:
                print("ซื้อโดยตรง")
            case .familyShared:
                print("แชร์จากครอบครัว")
            default:
                print("ประเภทไม่ทราบ")
            }
        }
        
        // App version ที่ซื้อ
        print("App Version: \(transaction.appVersion ?? "unknown")")
        
        // Environment
        print("Environment: \(transaction.environment)")  // .production หรือ .xcode
        
        // Storefront
        if let storefront = transaction.storefront {
            print("Country: \(storefront.countryCode)")
        }
    }
}
```

---

## 5. Purchase Flow

กระบวนการซื้อสินค้าแบบสมบูรณ์

```swift
import StoreKit

@MainActor
class PurchaseManager: ObservableObject {
    @Published var isPurchasing = false
    @Published var purchaseError: String?
    @Published var showSuccessAlert = false
    
    // ซื้อ product
    func purchase(_ product: Product) async {
        isPurchasing = true
        purchaseError = nil
        
        do {
            let result = try await product.purchase()
            
            switch result {
            case .success(let verification):
                // ตรวจสอบ transaction
                let transaction = try verifyTransaction(verification)
                
                // Grant entitlement
                await grantEntitlement(for: transaction)
                
                // Finish transaction (สำคัญมาก!)
                await transaction.finish()
                
                showSuccessAlert = true
                print("ซื้อ \(product.displayName) สำเร็จ")
                
            case .userCancelled:
                print("ผู้ใช้ยกเลิก")
                
            case .pending:
                // รอการอนุมัติ (Ask to Buy, SCA)
                print("Transaction กำลังรอการอนุมัติ")
                
            @unknown default:
                print("ผลการซื้อไม่ทราบ")
            }
        } catch StoreKitError.userCancelled {
            print("ผู้ใช้ยกเลิก")
        } catch StoreKitError.networkError(let networkError) {
            purchaseError = "เกิดข้อผิดพลาดเครือข่าย: \(networkError.localizedDescription)"
        } catch StoreKitError.notEntitled {
            purchaseError = "ไม่มีสิทธิ์ซื้อ"
        } catch {
            purchaseError = "เกิดข้อผิดพลาด: \(error.localizedDescription)"
        }
        
        isPurchasing = false
    }
    
    // ซื้อพร้อม options
    func purchaseWithOptions(_ product: Product) async {
        var purchaseOptions: Set<Product.PurchaseOption> = []
        
        // กำหนด quantity สำหรับ consumables
        // purchaseOptions.insert(.quantity(3))
        
        // กำหนด promotional offer
        // purchaseOptions.insert(.promotionalOffer(offerID: "myOffer", keyID: "keyID", nonce: UUID(), signature: signature, timestamp: timestamp))
        
        do {
            let result = try await product.purchase(options: purchaseOptions)
            // จัดการ result...
        } catch {
            print("Error: \(error)")
        }
    }
    
    private func verifyTransaction(_ verification: VerificationResult<Transaction>) throws -> Transaction {
        switch verification {
        case .verified(let transaction):
            return transaction
        case .unverified(_, let error):
            throw error
        }
    }
    
    private func grantEntitlement(for transaction: Transaction) async {
        // ให้สิทธิ์ตาม product type
        switch transaction.productType {
        case .consumable:
            await grantConsumable(productID: transaction.productID)
        case .nonConsumable:
            await unlockFeature(productID: transaction.productID)
        case .autoRenewable:
            await activateSubscription(productID: transaction.productID)
        case .nonRenewable:
            if let expiryDate = transaction.expirationDate {
                await activateNonRenewingAccess(until: expiryDate)
            }
        default:
            break
        }
    }
    
    private func grantConsumable(productID: String) async {
        // เพิ่ม coins/lives/etc ใน local storage หรือ server
        switch productID {
        case "com.myapp.coins.100":
            print("เพิ่ม 100 coins")
            // UserDefaults.standard.coins += 100
        case "com.myapp.coins.500":
            print("เพิ่ม 500 coins")
        default:
            break
        }
    }
    
    private func unlockFeature(productID: String) async {
        print("ปลดล็อค feature: \(productID)")
        // บันทึกใน UserDefaults หรือ Keychain
    }
    
    private func activateSubscription(productID: String) async {
        print("เปิดใช้งาน subscription: \(productID)")
    }
    
    private func activateNonRenewingAccess(until date: Date) async {
        print("เปิดใช้งานจนถึง: \(date)")
    }
}
```

---

## 6. Transaction Verification

การตรวจสอบความถูกต้องของ transactions

```swift
import StoreKit

class VerificationService {
    // Automatic verification โดย StoreKit 2
    func handleVerificationResult<T>(_ result: VerificationResult<T>) -> T? {
        switch result {
        case .verified(let value):
            // JWS signature verified โดย Apple
            return value
        case .unverified(let value, let error):
            // การ verify ล้มเหลว
            print("Verification error: \(error)")
            print("เหตุผล: \(verificationErrorDescription(error))")
            return nil  // ไม่ควร grant access
        }
    }
    
    func verificationErrorDescription(_ error: VerificationResult<Transaction>.VerificationError) -> String {
        switch error {
        case .invalidCertificateChain:
            return "Certificate chain ไม่ถูกต้อง"
        case .invalidDeviceVerification:
            return "Device verification ล้มเหลว"
        case .invalidEncoding:
            return "Encoding ไม่ถูกต้อง"
        case .invalidSignature:
            return "Signature ไม่ถูกต้อง"
        case .missingRequiredProperties:
            return "Missing required properties"
        case .revoked:
            return "Transaction ถูก revoke แล้ว"
        case .unknown:
            return "ข้อผิดพลาดไม่ทราบ"
        @unknown default:
            return "Unknown error"
        }
    }
    
    // ตรวจสอบ transaction อย่างละเอียด
    func validateTransaction(_ transaction: Transaction) -> Bool {
        // 1. ตรวจสอบ revocation
        if transaction.revocationDate != nil {
            print("Transaction ถูก revoke")
            return false
        }
        
        // 2. ตรวจสอบ expiration
        if let expirationDate = transaction.expirationDate {
            if expirationDate < Date() {
                print("Transaction หมดอายุ")
                return false
            }
        }
        
        // 3. ตรวจสอบ environment (ไม่รับ sandbox ใน production)
        #if !DEBUG
        if transaction.environment == .xcode {
            print("Sandbox transaction ใน production")
            return false
        }
        #endif
        
        return true
    }
}
```

---

## 7. Transaction.currentEntitlements

ดู entitlements ปัจจุบันของผู้ใช้

```swift
import StoreKit

class EntitlementChecker {
    // ดู current entitlements ทั้งหมด
    func checkCurrentEntitlements() async -> [Transaction] {
        var activeTransactions: [Transaction] = []
        
        for await result in Transaction.currentEntitlements {
            if case .verified(let transaction) = result {
                // ตรวจสอบว่ายังใช้งานได้
                if transaction.revocationDate == nil {
                    activeTransactions.append(transaction)
                }
            }
        }
        
        return activeTransactions
    }
    
    // ตรวจสอบว่าผู้ใช้มีสิทธิ์ใช้ feature หรือไม่
    func hasEntitlement(for productID: String) async -> Bool {
        for await result in Transaction.currentEntitlements {
            if case .verified(let transaction) = result {
                if transaction.productID == productID &&
                   transaction.revocationDate == nil {
                    return true
                }
            }
        }
        return false
    }
    
    // ตรวจสอบ subscription access
    func hasActiveSubscription(in groupID: String) async -> Bool {
        for await result in Transaction.currentEntitlements {
            if case .verified(let transaction) = result {
                if transaction.productType == .autoRenewable,
                   transaction.revocationDate == nil {
                    // ตรวจสอบ subscription group
                    return true
                }
            }
        }
        return false
    }
    
    // สรุป entitlements
    func summarizeEntitlements() async {
        print("=== Current Entitlements ===")
        
        var count = 0
        for await result in Transaction.currentEntitlements {
            count += 1
            switch result {
            case .verified(let transaction):
                print("\(count). ✅ \(transaction.productID) (\(transaction.productType))")
                if let expiry = transaction.expirationDate {
                    print("   หมดอายุ: \(expiry)")
                }
            case .unverified(let transaction, _):
                print("\(count). ❌ \(transaction.productID) - verification failed")
            }
        }
        
        if count == 0 {
            print("ไม่มี active entitlements")
        }
    }
}
```

---

## 8. Transaction.updates

ฟัง transaction updates ในเวลาจริง

```swift
import StoreKit

class TransactionUpdateListener {
    private var task: Task<Void, Error>?
    
    func startListening() {
        task = Task {
            for await verificationResult in Transaction.updates {
                await handleTransactionUpdate(verificationResult)
            }
        }
    }
    
    func stopListening() {
        task?.cancel()
        task = nil
    }
    
    private func handleTransactionUpdate(_ result: VerificationResult<Transaction>) async {
        do {
            let transaction = try checkVerified(result)
            
            // ประเภทการ update
            await processTransactionUpdate(transaction)
            
            // Finish transaction เพื่อให้ App Store รู้ว่าเราจัดการแล้ว
            await transaction.finish()
            
        } catch {
            print("Transaction verification ล้มเหลว: \(error)")
        }
    }
    
    private func processTransactionUpdate(_ transaction: Transaction) async {
        // จัดการตาม product type
        switch transaction.productType {
        case .consumable:
            // Consumable ซื้อใหม่
            print("Consumable ใหม่: \(transaction.productID)")
            await grantConsumableReward(transaction)
            
        case .nonConsumable:
            if transaction.revocationDate != nil {
                // Non-consumable ถูก revoke (เช่น refund)
                print("Non-consumable ถูก revoke: \(transaction.productID)")
                await revokeNonConsumable(transaction)
            } else {
                print("Non-consumable ซื้อ/restore: \(transaction.productID)")
                await grantNonConsumable(transaction)
            }
            
        case .autoRenewable:
            if transaction.revocationDate != nil {
                print("Subscription ถูก revoke")
                await deactivateSubscription(transaction)
            } else if let expiryDate = transaction.expirationDate, expiryDate > Date() {
                print("Subscription ต่ออายุ/ซื้อใหม่: \(transaction.productID)")
                await activateSubscription(transaction)
            }
            
        default:
            break
        }
    }
    
    private func checkVerified<T>(_ result: VerificationResult<T>) throws -> T {
        switch result {
        case .verified(let value): return value
        case .unverified(_, let error): throw error
        }
    }
    
    private func grantConsumableReward(_ transaction: Transaction) async {
        // เพิ่ม reward ให้ผู้ใช้
    }
    
    private func grantNonConsumable(_ transaction: Transaction) async {
        // ปลดล็อค feature
    }
    
    private func revokeNonConsumable(_ transaction: Transaction) async {
        // ล็อค feature กลับ
    }
    
    private func activateSubscription(_ transaction: Transaction) async {
        // เปิดใช้งาน subscription
    }
    
    private func deactivateSubscription(_ transaction: Transaction) async {
        // ปิดใช้งาน subscription
    }
}
```

---

## 9. Subscription Renewal Info

ข้อมูลการต่ออายุ subscription

```swift
import StoreKit

class SubscriptionRenewalManager {
    // ดูข้อมูลการต่ออายุ
    func getRenewalInfo(for product: Product) async throws {
        guard let subscription = product.subscription else { return }
        
        // ดู statuses ใน subscription group
        let statuses = try await product.subscription?.status
        
        for status in statuses ?? [] {
            let renewalInfo = try checkVerified(status.renewalInfo)
            let transaction = try checkVerified(status.transaction)
            
            print("=== Renewal Info ===")
            print("State: \(status.state)")
            print("Product ID: \(renewalInfo.currentProductID)")
            print("Auto Renew: \(renewalInfo.willAutoRenew)")
            print("Auto Renew Product: \(renewalInfo.autoRenewProductID ?? "same")")
            
            // ตรวจสอบ expiration reason
            if let expirationReason = renewalInfo.expirationReason {
                switch expirationReason {
                case .autoRenewDisabled:
                    print("ผู้ใช้ปิด Auto-Renew")
                case .billingError:
                    print("เกิดข้อผิดพลาดในการเรียกเก็บเงิน")
                case .didNotConsentToPriceIncrease:
                    print("ไม่ยอมรับราคาที่เพิ่มขึ้น")
                case .productNotForSale:
                    print("Product ไม่วางขายแล้ว")
                default:
                    print("เหตุผลอื่น")
                }
            }
            
            // Grace period (Apple ให้เวลา retry เรียกเก็บเงิน)
            if let gracePeriodDate = renewalInfo.gracePeriodExpirationDate {
                print("Grace period จนถึง: \(gracePeriodDate)")
            }
            
            // Offer redemption
            if let offerID = renewalInfo.offerID {
                print("Offer: \(offerID)")
                print("Offer Type: \(renewalInfo.offerType?.rawValue ?? "unknown")")
            }
        }
    }
    
    private func checkVerified<T>(_ result: VerificationResult<T>) throws -> T {
        switch result {
        case .verified(let value): return value
        case .unverified(_, let error): throw error
        }
    }
    
    // ตรวจสอบ subscription state
    func getSubscriptionState(for product: Product) async -> SubscriptionState {
        do {
            guard let statuses = try await product.subscription?.status else {
                return .notSubscribed
            }
            
            for status in statuses {
                switch status.state {
                case .subscribed:
                    return .active
                case .expired:
                    return .expired
                case .inBillingRetryPeriod:
                    return .billingRetry
                case .inGracePeriod:
                    return .gracePeriod
                case .revoked:
                    return .revoked
                default:
                    continue
                }
            }
        } catch {
            print("Error: \(error)")
        }
        
        return .notSubscribed
    }
}

enum SubscriptionState {
    case notSubscribed
    case active
    case expired
    case billingRetry
    case gracePeriod
    case revoked
    
    var displayText: String {
        switch self {
        case .notSubscribed: return "ยังไม่ได้สมัคร"
        case .active: return "ใช้งานอยู่"
        case .expired: return "หมดอายุ"
        case .billingRetry: return "กำลังเรียกเก็บเงินซ้ำ"
        case .gracePeriod: return "อยู่ในช่วง Grace Period"
        case .revoked: return "ถูกยกเลิก"
        }
    }
    
    var isActive: Bool {
        switch self {
        case .active, .billingRetry, .gracePeriod:
            return true
        default:
            return false
        }
    }
}
```

---

## 10. Introductory Offers

ข้อเสนอพิเศษสำหรับผู้ใช้ใหม่

```swift
import StoreKit

class IntroductoryOfferManager {
    // ตรวจสอบ introductory offer
    func checkIntroductoryOffer(for product: Product) {
        guard let subscription = product.subscription,
              let introOffer = subscription.introductoryOffer else {
            print("\(product.displayName) ไม่มี introductory offer")
            return
        }
        
        print("=== Introductory Offer ===")
        print("ราคา: \(introOffer.displayPrice)")
        print("ระยะเวลา: \(introOffer.period.value) \(introOffer.period.unit)")
        
        switch introOffer.paymentMode {
        case .freeTrial:
            print("ประเภท: ทดลองฟรี")
        case .payAsYouGo:
            print("ประเภท: Pay As You Go - จ่ายแต่ละ period")
        case .payUpFront:
            print("ประเภท: Pay Up Front - จ่ายล่วงหน้า")
        @unknown default:
            print("ประเภทไม่ทราบ")
        }
    }
    
    // ตรวจสอบว่าผู้ใช้ eligible สำหรับ intro offer หรือไม่
    func isEligibleForIntroductoryOffer(product: Product) async -> Bool {
        guard let subscription = product.subscription else { return false }
        
        let isEligible = await subscription.isEligibleForIntroOffer
        return isEligible
    }
    
    // แสดง pricing information ที่ถูกต้อง
    func displayPricing(for product: Product) async -> String {
        guard let subscription = product.subscription else {
            return product.displayPrice
        }
        
        // ตรวจสอบ intro offer eligibility
        let isEligible = await subscription.isEligibleForIntroOffer
        
        if isEligible, let introOffer = subscription.introductoryOffer {
            switch introOffer.paymentMode {
            case .freeTrial:
                let trialDuration = "\(introOffer.period.value) \(periodUnitString(introOffer.period.unit))"
                return "ทดลองฟรี \(trialDuration) จากนั้น \(product.displayPrice)"
            case .payAsYouGo:
                return "\(introOffer.displayPrice) ต่อ \(periodUnitString(introOffer.period.unit)) (ช่วงแรก)"
            case .payUpFront:
                return "ราคาพิเศษ: \(introOffer.displayPrice)"
            @unknown default:
                return product.displayPrice
            }
        }
        
        return product.displayPrice
    }
    
    func periodUnitString(_ unit: Product.SubscriptionPeriod.Unit) -> String {
        switch unit {
        case .day: return "วัน"
        case .week: return "สัปดาห์"
        case .month: return "เดือน"
        case .year: return "ปี"
        @unknown default: return ""
        }
    }
}
```

---

## 11. Promotional Offers

ข้อเสนอพิเศษสำหรับผู้ใช้ที่มีอยู่แล้ว

```swift
import StoreKit
import CryptoKit

class PromotionalOfferManager {
    // ดูรายการ promotional offers
    func listPromotionalOffers(for product: Product) {
        guard let subscription = product.subscription else { return }
        
        for offer in subscription.promotionalOffers {
            print("=== Promotional Offer ===")
            print("ID: \(offer.id)")
            print("ราคา: \(offer.displayPrice)")
            
            switch offer.paymentMode {
            case .freeTrial:
                print("ประเภท: ทดลองฟรี")
            case .payAsYouGo:
                print("ประเภท: Pay As You Go")
            case .payUpFront:
                print("ประเภท: Pay Up Front")
            @unknown default:
                break
            }
            
            print("ระยะเวลา: \(offer.period.value) \(offer.period.unit)")
        }
    }
    
    // ซื้อด้วย promotional offer
    // หมายเหตุ: ต้องสร้าง signature จาก server
    func purchaseWithPromoOffer(
        product: Product,
        offerID: String,
        keyID: String,
        nonce: UUID,
        signature: Data,
        timestamp: Int
    ) async throws {
        let promoOffer = try await Product.PromotionalOffer(
            offerID: offerID,
            keyID: keyID,
            nonce: nonce,
            signature: signature,
            timestamp: timestamp
        )
        
        let result = try await product.purchase(options: [
            .promotionalOffer(promoOffer)
        ])
        
        switch result {
        case .success(let verification):
            print("ซื้อด้วย promotional offer สำเร็จ")
        case .userCancelled:
            print("ผู้ใช้ยกเลิก")
        case .pending:
            print("รอการอนุมัติ")
        @unknown default:
            break
        }
    }
}

// Server-side signature generation (ตัวอย่าง)
// ต้องทำบน server จริงๆ ไม่ใช่ใน client!
class PromotionalOfferSignatureServer {
    func generateSignature(
        appBundleID: String,
        keyID: String,
        productID: String,
        offerID: String,
        applicationUsername: String,
        nonce: UUID,
        timestamp: Int,
        privateKey: P256.Signing.PrivateKey
    ) throws -> Data {
        // สร้าง message ที่ต้อง sign
        let message = "\(appBundleID)\0\(keyID)\0\(productID)\0\(offerID)\0\(applicationUsername)\0\(nonce.uuidString.lowercased())\0\(timestamp)"
        
        guard let messageData = message.data(using: .utf8) else {
            throw NSError(domain: "SignatureError", code: 0)
        }
        
        // Sign ด้วย ECDSA P-256
        let signature = try privateKey.signature(for: messageData)
        return signature.derRepresentation
    }
}
```

---

## 12. Offer Codes

Redeem codes สำหรับ subscriptions

```swift
import StoreKit

class OfferCodeManager {
    // แสดง sheet สำหรับ redeem offer code
    func presentOfferCodeRedemption(from viewController: UIViewController) {
        Task {
            do {
                try await AppStore.presentOfferCodeRedeemSheet(in: viewController)
            } catch {
                print("ไม่สามารถแสดง offer code sheet: \(error)")
            }
        }
    }
    
    // ใน SwiftUI
    // .offerCodeRedemption(isPresented: $showOfferCode) ใช้ modifier บน View
}

// SwiftUI Integration
struct OfferCodeView: View {
    @State private var showOfferCodeRedemption = false
    
    var body: some View {
        Button("แลก Offer Code") {
            showOfferCodeRedemption = true
        }
        .offerCodeRedemption(isPresented: $showOfferCodeRedemption) { result in
            switch result {
            case .success:
                print("Offer code แลกสำเร็จ")
            case .failure(let error):
                print("Error: \(error)")
            }
        }
    }
}
```

---

## 13. Family Sharing

การแชร์ purchases กับสมาชิกในครอบครัว

```swift
import StoreKit

class FamilySharingManager {
    // ตรวจสอบ ownership type ของ transaction
    func checkOwnershipType(_ transaction: Transaction) {
        switch transaction.ownershipType {
        case .purchased:
            print("ซื้อโดย Apple ID นี้")
        case .familyShared:
            print("แชร์มาจากสมาชิกครอบครัว")
        default:
            print("ประเภทไม่ทราบ")
        }
    }
    
    // App Store Connect settings:
    // - เปิด Family Sharing ใน product configuration
    // - Non-consumable และ Auto-Renewable Subscription รองรับ Family Sharing
    // - Consumable ไม่รองรับ
    
    // In-App Purchase Family Sharing requirement:
    // - ต้อง enable ใน App Store Connect
    // - Products ที่ enable Family Sharing จะ inherit entitlement ให้สมาชิกครอบครัว
    
    func handleFamilySharingTransaction(_ transaction: Transaction) async {
        // เมื่อสมาชิกครอบครัวซื้อ หรือเมื่อ owner ซื้อและ share
        if transaction.ownershipType == .familyShared {
            // Grant access ให้กับ account นี้
            // (มักจะ handle เหมือน purchased ปกติ)
            print("Grant access จาก Family Sharing")
        }
    }
}
```

---

## 14. StoreKit Testing (Xcode StoreKit Configuration)

การทดสอบ IAP โดยไม่ต้องเชื่อมต่อ App Store

### สร้าง StoreKit Configuration File

```swift
// ใน Xcode:
// File > New > File > StoreKit Configuration File
// ตั้งชื่อ: Products.storekit

// กำหนด products ใน .storekit file (JSON format):
/*
{
  "identifier" : "com.myapp.coins.100",
  "localizations" : [
    {
      "description" : "100 Gold Coins",
      "displayName" : "100 เหรียญ",
      "locale" : "th"
    }
  ],
  "price" : "29",
  "productID" : "com.myapp.coins.100",
  "referenceName" : "100 Coins",
  "type" : "consumable"
}
*/

// ใน Scheme > Run > Options > StoreKit Configuration
// เลือก Products.storekit

// ตั้งค่า Configuration ใน code
class StoreKitTestConfiguration {
    static func configure() {
        #if DEBUG
        // StoreKit ใช้ configuration file โดยอัตโนมัติเมื่อตั้งใน Scheme
        print("Using StoreKit Configuration file for testing")
        #endif
    }
}
```

### Testing ด้วย Xcode

```swift
// StoreKit Testing features:
// 1. Purchase without real money
// 2. Test subscription renewal (accelerated)
// 3. Test expired subscriptions
// 4. Test refunds
// 5. Test Ask to Buy
// 6. Test introductory offers

// ใน Debug > StoreKit > Manage Transactions
// - ดู transaction history
// - Approve/Reject pending transactions (Ask to Buy)
// - Refund transactions
// - Expire subscriptions

// Accelerated subscription renewal rates:
// 1 week -> 3 minutes
// 1 month -> 5 minutes  
// 2 months -> 10 minutes
// 3 months -> 15 minutes
// 6 months -> 30 minutes
// 1 year -> 1 hour

class StoreKitTestHelper {
    // Reset สำหรับ testing
    static func resetForTesting() {
        // ใน Xcode: Debug > StoreKit > Reset Purchased Products
        // จะล้าง purchases ทั้งหมดในสภาพแวดล้อม test
    }
    
    // ตรวจสอบว่าอยู่ใน test environment
    static var isTestEnvironment: Bool {
        #if DEBUG
        return true
        #else
        return false
        #endif
    }
}
```

---

## 15. SKTestSession

```swift
import StoreKitTest
import XCTest

class InAppPurchaseTests: XCTestCase {
    var session: SKTestSession!
    var storeManager: StoreManager!
    
    override func setUp() async throws {
        // สร้าง test session
        session = try SKTestSession(configurationFileNamed: "Products")
        session.resetToDefaultState()
        session.disableDialogs = true  // ไม่แสดง dialogs ระหว่าง test
        session.clearTransactions()
        
        storeManager = await StoreManager()
    }
    
    override func tearDown() async throws {
        session.clearTransactions()
    }
    
    // ทดสอบ consumable purchase
    func testConsumablePurchase() async throws {
        // โหลด products
        await storeManager.loadProducts()
        
        guard let coinsProduct = storeManager.products.first(where: { $0.id == "com.myapp.coins.100" }) else {
            XCTFail("ไม่พบ product")
            return
        }
        
        // ซื้อ
        await storeManager.purchase(coinsProduct)
        
        // รอให้ transaction เสร็จ
        try await Task.sleep(nanoseconds: 500_000_000)
        
        // ตรวจสอบ
        XCTAssertNil(storeManager.purchaseError)
    }
    
    // ทดสอบ non-consumable purchase
    func testNonConsumablePurchase() async throws {
        await storeManager.loadProducts()
        
        guard let premiumProduct = storeManager.products.first(where: { $0.id == "com.myapp.remove_ads" }) else {
            XCTFail("ไม่พบ product")
            return
        }
        
        // ซื้อ
        await storeManager.purchase(premiumProduct)
        try await Task.sleep(nanoseconds: 500_000_000)
        
        // ตรวจสอบว่า unlock แล้ว
        let hasAccess = await storeManager.hasEntitlement(for: premiumProduct.id)
        XCTAssertTrue(hasAccess)
    }
    
    // ทดสอบ subscription
    func testSubscriptionPurchase() async throws {
        await storeManager.loadProducts()
        
        guard let monthlyProduct = storeManager.products.first(where: { $0.id == "com.myapp.premium.monthly" }) else {
            XCTFail("ไม่พบ product")
            return
        }
        
        await storeManager.purchase(monthlyProduct)
        try await Task.sleep(nanoseconds: 500_000_000)
        
        let isSubscribed = await storeManager.hasActiveSubscription(groupID: "premium")
        XCTAssertTrue(isSubscribed)
    }
    
    // ทดสอบ subscription expiry
    func testSubscriptionExpiry() async throws {
        await storeManager.loadProducts()
        
        guard let monthlyProduct = storeManager.products.first(where: { $0.id == "com.myapp.premium.monthly" }) else {
            return
        }
        
        await storeManager.purchase(monthlyProduct)
        try await Task.sleep(nanoseconds: 500_000_000)
        
        // Expire subscription ด้วย session
        session.expireSubscription(productIdentifier: monthlyProduct.id)
        try await Task.sleep(nanoseconds: 500_000_000)
        
        let isSubscribed = await storeManager.hasActiveSubscription(groupID: "premium")
        XCTAssertFalse(isSubscribed)
    }
    
    // ทดสอบ restore
    func testRestorePurchases() async throws {
        // ล้าง state ก่อน
        session.clearTransactions()
        
        await storeManager.loadProducts()
        
        guard let premiumProduct = storeManager.products.first(where: { $0.id == "com.myapp.remove_ads" }) else {
            return
        }
        
        // ซื้อ
        await storeManager.purchase(premiumProduct)
        try await Task.sleep(nanoseconds: 500_000_000)
        
        // Restore (simulate new installation)
        await AppStore.sync()
        
        let hasAccess = await storeManager.hasEntitlement(for: premiumProduct.id)
        XCTAssertTrue(hasAccess)
    }
}
```

---

## 16. Receipt Validation (Legacy)

สำหรับ apps ที่ยังใช้ StoreKit 1

```swift
import StoreKit

// หมายเหตุ: StoreKit 2 ไม่ต้องใช้ receipt validation แบบนี้อีกต่อไป
// แต่ยังต้องรู้ไว้สำหรับ apps รุ่นเก่า

class LegacyReceiptValidator {
    func validateReceipt() -> Data? {
        // ดึง receipt จาก bundle
        guard let receiptURL = Bundle.main.appStoreReceiptURL,
              FileManager.default.fileExists(atPath: receiptURL.path) else {
            print("ไม่พบ receipt")
            return nil
        }
        
        do {
            let receiptData = try Data(contentsOf: receiptURL)
            let receiptBase64 = receiptData.base64EncodedString()
            print("Receipt ขนาด: \(receiptData.count) bytes")
            return receiptData
        } catch {
            print("ไม่สามารถอ่าน receipt: \(error)")
            return nil
        }
    }
    
    func requestReceipt(from viewController: UIViewController) {
        let refreshRequest = SKReceiptRefreshRequest()
        refreshRequest.start()
    }
}

// Legacy SKPaymentTransactionObserver
class LegacyPaymentObserver: NSObject, SKPaymentTransactionObserver {
    static let shared = LegacyPaymentObserver()
    
    func startObserving() {
        SKPaymentQueue.default().add(self)
    }
    
    func stopObserving() {
        SKPaymentQueue.default().remove(self)
    }
    
    // ตรวจสอบว่า payment allowed
    func canMakePayments() -> Bool {
        return SKPaymentQueue.canMakePayments()
    }
    
    // ซื้อ product (legacy)
    func buy(productIdentifier: String) {
        guard SKPaymentQueue.canMakePayments() else {
            print("ไม่สามารถซื้อได้")
            return
        }
        
        let payment = SKPayment(productIdentifier: productIdentifier)
        SKPaymentQueue.default().add(payment)
    }
    
    // MARK: - SKPaymentTransactionObserver
    
    func paymentQueue(_ queue: SKPaymentQueue, updatedTransactions transactions: [SKPaymentTransaction]) {
        for transaction in transactions {
            switch transaction.transactionState {
            case .purchased:
                handlePurchased(transaction)
            case .restored:
                handleRestored(transaction)
            case .failed:
                handleFailed(transaction)
            case .deferred:
                print("Transaction deferred (Ask to Buy)")
            case .purchasing:
                print("กำลังซื้อ...")
            @unknown default:
                break
            }
        }
    }
    
    private func handlePurchased(_ transaction: SKPaymentTransaction) {
        print("ซื้อสำเร็จ: \(transaction.payment.productIdentifier)")
        // Grant entitlement
        SKPaymentQueue.default().finishTransaction(transaction)
    }
    
    private func handleRestored(_ transaction: SKPaymentTransaction) {
        print("Restored: \(transaction.original?.payment.productIdentifier ?? "")")
        SKPaymentQueue.default().finishTransaction(transaction)
    }
    
    private func handleFailed(_ transaction: SKPaymentTransaction) {
        if let error = transaction.error as? SKError {
            if error.code != .paymentCancelled {
                print("Purchase failed: \(error.localizedDescription)")
            }
        }
        SKPaymentQueue.default().finishTransaction(transaction)
    }
    
    // Restore purchases
    func restorePurchases() {
        SKPaymentQueue.default().restoreCompletedTransactions()
    }
    
    func paymentQueueRestoreCompletedTransactionsFinished(_ queue: SKPaymentQueue) {
        print("Restore เสร็จสิ้น")
    }
    
    func paymentQueue(_ queue: SKPaymentQueue, restoreCompletedTransactionsFailedWithError error: Error) {
        print("Restore ล้มเหลว: \(error)")
    }
}
```

---

## 17. Server-Side Validation

การ validate receipts บน server

```swift
import Foundation

// Server-Side Receipt Validation
// ใช้สำหรับ consumables และเพื่อความปลอดภัย

struct ServerReceiptValidator {
    // App Store endpoints
    static let productionURL = "https://buy.itunes.apple.com/verifyReceipt"
    static let sandboxURL = "https://sandbox.itunes.apple.com/verifyReceipt"
    
    // ส่ง receipt ไปยัง server ของคุณ
    func validateWithServer(receiptData: Data) async throws -> ServerValidationResult {
        let base64Receipt = receiptData.base64EncodedString()
        
        // ส่งไปยัง backend ของคุณ
        let url = URL(string: "https://your-server.com/validate-receipt")!
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        
        let body = ["receipt": base64Receipt]
        request.httpBody = try JSONSerialization.data(withJSONObject: body)
        
        let (data, response) = try await URLSession.shared.data(for: request)
        
        guard let httpResponse = response as? HTTPURLResponse,
              httpResponse.statusCode == 200 else {
            throw ValidationError.serverError
        }
        
        let result = try JSONDecoder().decode(ServerValidationResult.self, from: data)
        return result
    }
    
    // Backend ส่ง receipt ไปยัง Apple
    // (นี่คือ code ที่ควรรันบน server ของคุณ ไม่ใช่ใน app)
    func validateWithAppleServer(receiptBase64: String, isProduction: Bool) async throws -> AppleReceiptResponse {
        let endpoint = isProduction ? Self.productionURL : Self.sandboxURL
        let url = URL(string: endpoint)!
        
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        
        let body: [String: Any] = [
            "receipt-data": receiptBase64,
            "password": "YOUR_APP_SHARED_SECRET",  // จาก App Store Connect
            "exclude-old-transactions": true
        ]
        
        request.httpBody = try JSONSerialization.data(withJSONObject: body)
        
        let (data, _) = try await URLSession.shared.data(for: request)
        return try JSONDecoder().decode(AppleReceiptResponse.self, from: data)
    }
}

// Response models
struct ServerValidationResult: Codable {
    let isValid: Bool
    let productID: String?
    let expirationDate: Date?
    let purchases: [PurchaseRecord]
}

struct PurchaseRecord: Codable {
    let productID: String
    let purchaseDate: Date
    let transactionID: String
    let isValid: Bool
}

struct AppleReceiptResponse: Codable {
    let status: Int  // 0 = valid
    let receipt: ReceiptInfo?
    let latestReceiptInfo: [InAppPurchaseInfo]?
    
    enum CodingKeys: String, CodingKey {
        case status
        case receipt
        case latestReceiptInfo = "latest_receipt_info"
    }
}

struct ReceiptInfo: Codable {
    let bundleID: String
    let appVersion: String
    let inApp: [InAppPurchaseInfo]
    
    enum CodingKeys: String, CodingKey {
        case bundleID = "bundle_id"
        case appVersion = "application_version"
        case inApp = "in_app"
    }
}

struct InAppPurchaseInfo: Codable {
    let productID: String
    let transactionID: String
    let purchaseDateMS: String
    let expiresDateMS: String?
    
    enum CodingKeys: String, CodingKey {
        case productID = "product_id"
        case transactionID = "transaction_id"
        case purchaseDateMS = "purchase_date_ms"
        case expiresDateMS = "expires_date_ms"
    }
}

enum ValidationError: Error {
    case serverError
    case invalidReceipt
    case expired
}
```

---

## 18. App Store Connect Configuration

การตั้งค่าใน App Store Connect

```swift
/*
ขั้นตอนการสร้าง In-App Purchase ใน App Store Connect:

1. เข้า App Store Connect (appstoreconnect.apple.com)
2. เลือกแอปของคุณ
3. Features > In-App Purchases
4. คลิก "+" เพื่อเพิ่ม product

5. กำหนด:
   - Reference Name: ชื่อภายใน (ไม่แสดงแก่ผู้ใช้)
   - Product ID: unique identifier (เช่น com.myapp.coins.100)
   - Type: เลือกประเภท
   
6. สำหรับ Subscriptions:
   - สร้าง Subscription Group ก่อน
   - กำหนด Subscription Duration
   - กำหนด Family Sharing
   
7. Localizations:
   - Display Name (ชื่อที่แสดงให้ผู้ใช้)
   - Description
   
8. Pricing:
   - กำหนดราคาจาก Price Schedule
   
9. Review Information:
   - Screenshot สำหรับ Review team
   - Notes

สำหรับ Subscriptions เพิ่มเติม:
- Subscription Groups: จัดกลุ่ม subscription ระดับต่างๆ
- Free Trials: กำหนดระยะเวลาทดลองใช้ฟรี
- Introductory Offers: ราคาพิเศษช่วงแรก
- Promotional Offers: สำหรับ win-back campaigns
- Offer Codes: สร้าง codes สำหรับแจกจ่าย

App Shared Secret:
- ใช้สำหรับ server-side receipt validation
- Users and Access > Shared Secret
*/

// App Shared Secret สำหรับ Server Validation
let appSharedSecret = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"  // จาก App Store Connect
```

---

## 19. Subscription Management UI

```swift
import StoreKit
import SwiftUI

struct SubscriptionManagementView: View {
    @StateObject private var storeManager = StoreManager()
    @State private var showManageSubscriptions = false
    
    var body: some View {
        List {
            Section("Plans") {
                ForEach(storeManager.subscriptionProducts) { product in
                    SubscriptionProductRow(
                        product: product,
                        isCurrentPlan: storeManager.isCurrentSubscription(product),
                        onPurchase: {
                            Task { await storeManager.purchase(product) }
                        }
                    )
                }
            }
            
            if storeManager.hasActiveSubscription {
                Section("Current Subscription") {
                    SubscriptionStatusView(storeManager: storeManager)
                    
                    Button("จัดการ Subscription") {
                        showManageSubscriptions = true
                    }
                    .foregroundColor(.blue)
                }
            }
        }
        .navigationTitle("Subscription")
        .manageSubscriptionsSheet(isPresented: $showManageSubscriptions)
        .task {
            await storeManager.loadProducts()
        }
    }
}

struct SubscriptionProductRow: View {
    let product: Product
    let isCurrentPlan: Bool
    let onPurchase: () -> Void
    
    var body: some View {
        HStack {
            VStack(alignment: .leading, spacing: 4) {
                Text(product.displayName)
                    .font(.headline)
                
                Text(product.description)
                    .font(.caption)
                    .foregroundColor(.secondary)
                
                if let subscription = product.subscription {
                    Text(periodDescription(subscription.subscriptionPeriod))
                        .font(.caption2)
                        .foregroundColor(.secondary)
                }
            }
            
            Spacer()
            
            if isCurrentPlan {
                Text("ปัจจุบัน")
                    .font(.caption)
                    .padding(.horizontal, 12)
                    .padding(.vertical, 6)
                    .background(Color.green.opacity(0.2))
                    .foregroundColor(.green)
                    .cornerRadius(12)
            } else {
                Button(action: onPurchase) {
                    Text(product.displayPrice)
                        .font(.callout.bold())
                        .padding(.horizontal, 12)
                        .padding(.vertical, 6)
                        .background(Color.blue)
                        .foregroundColor(.white)
                        .cornerRadius(12)
                }
            }
        }
        .padding(.vertical, 4)
    }
    
    func periodDescription(_ period: Product.SubscriptionPeriod) -> String {
        let unit: String
        switch period.unit {
        case .day: unit = "วัน"
        case .week: unit = "สัปดาห์"
        case .month: unit = "เดือน"
        case .year: unit = "ปี"
        @unknown default: unit = ""
        }
        return "ต่อ \(period.value > 1 ? "\(period.value) " : "")\(unit)"
    }
}

struct SubscriptionStatusView: View {
    @ObservedObject var storeManager: StoreManager
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            if let status = storeManager.subscriptionStatus {
                HStack {
                    Image(systemName: "checkmark.circle.fill")
                        .foregroundColor(.green)
                    Text("Active")
                        .foregroundColor(.green)
                }
                
                if let renewalDate = storeManager.nextRenewalDate {
                    Text("ต่ออายุวันที่: \(renewalDate, style: .date)")
                        .font(.caption)
                        .foregroundColor(.secondary)
                }
            }
        }
    }
}
```

---

## 20. Restore Purchases

```swift
import StoreKit

class RestoreManager {
    // StoreKit 2: ใช้ AppStore.sync()
    func restorePurchases() async throws {
        do {
            try await AppStore.sync()
            print("Restore สำเร็จ")
        } catch {
            print("Restore ล้มเหลว: \(error)")
            throw error
        }
    }
    
    // SwiftUI Button
    var restoreButton: some View {
        Button("กู้คืนการซื้อ") {
            Task {
                do {
                    try await AppStore.sync()
                    // แจ้งผู้ใช้ว่าสำเร็จ
                } catch {
                    // แจ้งผู้ใช้ว่าล้มเหลว
                }
            }
        }
    }
    
    // Legacy StoreKit 1
    func legacyRestorePurchases() {
        SKPaymentQueue.default().restoreCompletedTransactions()
    }
    
    // หมายเหตุ:
    // - Restore ทำงานสำหรับ Non-Consumable และ Auto-Renewable Subscription
    // - Consumable ไม่สามารถ restore ได้
    // - Apple กำหนดให้แอปต้องมีปุ่ม Restore Purchases
    // - ต้องแสดงข้อความ "Already Purchased? Restore" ในหน้าซื้อ
}

// ตรวจสอบก่อนแสดงปุ่ม Restore
struct RestoreButtonView: View {
    @State private var isRestoring = false
    @State private var showResult = false
    @State private var restoreSuccess = false
    
    var body: some View {
        Button {
            isRestoring = true
            Task {
                do {
                    try await AppStore.sync()
                    restoreSuccess = true
                } catch {
                    restoreSuccess = false
                }
                isRestoring = false
                showResult = true
            }
        } label: {
            if isRestoring {
                ProgressView()
                    .progressViewStyle(.circular)
            } else {
                Text("กู้คืนการซื้อ")
            }
        }
        .disabled(isRestoring)
        .alert(
            restoreSuccess ? "กู้คืนสำเร็จ" : "กู้คืนล้มเหลว",
            isPresented: $showResult
        ) {
            Button("ตกลง") {}
        } message: {
            Text(restoreSuccess
                ? "กู้คืนการซื้อทั้งหมดเรียบร้อยแล้ว"
                : "ไม่สามารถกู้คืนได้ กรุณาลองใหม่อีกครั้ง")
        }
    }
}
```

---

## 21. StoreKit with SwiftUI

```swift
import StoreKit
import SwiftUI

// ProductView - แสดงข้อมูล product อย่างง่าย
struct SimpleProductView: View {
    let productID: String
    
    var body: some View {
        // StoreKit 2 มี ProductView built-in (iOS 17+)
        if #available(iOS 17.0, *) {
            // ProductView(id: productID)
        }
        
        // Custom implementation
        ProductRowView(productID: productID)
    }
}

// StoreView (iOS 17+) - แสดง products จาก IDs
struct StoreViewExample: View {
    let productIDs: [String] = [
        "com.myapp.premium.monthly",
        "com.myapp.premium.yearly"
    ]
    
    var body: some View {
        if #available(iOS 17.0, *) {
            // StoreView(ids: productIDs) {
            //     // header
            // }
        }
    }
}

// SubscriptionStoreView (iOS 17+)
struct SubscriptionView: View {
    var body: some View {
        if #available(iOS 17.0, *) {
            // SubscriptionStoreView(groupID: "com.myapp.premium") {
            //     // header content
            // }
        }
    }
}

// Complete Store Screen
struct StoreScreen: View {
    @StateObject private var viewModel = StoreViewModel()
    
    var body: some View {
        NavigationView {
            ScrollView {
                VStack(spacing: 24) {
                    // Header
                    StoreHeaderView()
                    
                    // Subscription Products
                    if !viewModel.subscriptionProducts.isEmpty {
                        SubscriptionSection(
                            products: viewModel.subscriptionProducts,
                            purchasedIDs: viewModel.purchasedProductIDs,
                            onPurchase: { product in
                                Task { await viewModel.purchase(product) }
                            }
                        )
                    }
                    
                    // One-time purchases
                    if !viewModel.nonConsumables.isEmpty {
                        OneTimePurchaseSection(
                            products: viewModel.nonConsumables,
                            purchasedIDs: viewModel.purchasedProductIDs,
                            onPurchase: { product in
                                Task { await viewModel.purchase(product) }
                            }
                        )
                    }
                    
                    // Consumables
                    if !viewModel.consumables.isEmpty {
                        ConsumableSection(
                            products: viewModel.consumables,
                            onPurchase: { product in
                                Task { await viewModel.purchase(product) }
                            }
                        )
                    }
                    
                    // Restore button
                    RestoreButtonView()
                    
                    // Terms & Privacy
                    TermsView()
                }
                .padding()
            }
            .navigationTitle("Store")
            .task {
                await viewModel.loadProducts()
            }
        }
    }
}

struct StoreHeaderView: View {
    var body: some View {
        VStack(spacing: 12) {
            Image(systemName: "crown.fill")
                .font(.system(size: 60))
                .foregroundColor(.yellow)
            
            Text("Premium")
                .font(.largeTitle.bold())
            
            Text("ปลดล็อคทุกฟีเจอร์")
                .font(.subheadline)
                .foregroundColor(.secondary)
        }
        .padding()
    }
}

struct SubscriptionSection: View {
    let products: [Product]
    let purchasedIDs: Set<String>
    let onPurchase: (Product) -> Void
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            Text("SUBSCRIPTION")
                .font(.caption.bold())
                .foregroundColor(.secondary)
            
            ForEach(products) { product in
                SubscriptionProductCard(
                    product: product,
                    isPurchased: purchasedIDs.contains(product.id),
                    onPurchase: { onPurchase(product) }
                )
            }
        }
    }
}

struct SubscriptionProductCard: View {
    let product: Product
    let isPurchased: Bool
    let onPurchase: () -> Void
    
    var body: some View {
        HStack {
            VStack(alignment: .leading, spacing: 4) {
                Text(product.displayName)
                    .font(.headline)
                Text(product.description)
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
            
            Spacer()
            
            if isPurchased {
                Label("ซื้อแล้ว", systemImage: "checkmark.circle.fill")
                    .font(.caption)
                    .foregroundColor(.green)
            } else {
                Button(action: onPurchase) {
                    Text(product.displayPrice)
                        .font(.callout.bold())
                }
                .buttonStyle(.borderedProminent)
            }
        }
        .padding()
        .background(Color(.secondarySystemBackground))
        .cornerRadius(12)
    }
}

struct OneTimePurchaseSection: View {
    let products: [Product]
    let purchasedIDs: Set<String>
    let onPurchase: (Product) -> Void
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            Text("ONE-TIME PURCHASE")
                .font(.caption.bold())
                .foregroundColor(.secondary)
            
            ForEach(products) { product in
                HStack {
                    VStack(alignment: .leading) {
                        Text(product.displayName).font(.headline)
                        Text(product.description).font(.caption).foregroundColor(.secondary)
                    }
                    Spacer()
                    if purchasedIDs.contains(product.id) {
                        Image(systemName: "checkmark.circle.fill").foregroundColor(.green)
                    } else {
                        Button(product.displayPrice) { onPurchase(product) }
                            .buttonStyle(.borderedProminent)
                    }
                }
                .padding()
                .background(Color(.secondarySystemBackground))
                .cornerRadius(12)
            }
        }
    }
}

struct ConsumableSection: View {
    let products: [Product]
    let onPurchase: (Product) -> Void
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            Text("COINS & ITEMS")
                .font(.caption.bold())
                .foregroundColor(.secondary)
            
            ForEach(products) { product in
                HStack {
                    Image(systemName: "dollarsign.circle.fill")
                        .foregroundColor(.yellow)
                    VStack(alignment: .leading) {
                        Text(product.displayName).font(.headline)
                    }
                    Spacer()
                    Button(product.displayPrice) { onPurchase(product) }
                        .buttonStyle(.borderedProminent)
                }
                .padding()
                .background(Color(.secondarySystemBackground))
                .cornerRadius(12)
            }
        }
    }
}

struct TermsView: View {
    var body: some View {
        VStack(spacing: 8) {
            Text("By purchasing, you agree to our Terms of Service and Privacy Policy")
                .font(.caption)
                .foregroundColor(.secondary)
                .multilineTextAlignment(.center)
            
            HStack(spacing: 20) {
                Link("Terms", destination: URL(string: "https://example.com/terms")!)
                    .font(.caption)
                Link("Privacy", destination: URL(string: "https://example.com/privacy")!)
                    .font(.caption)
            }
        }
        .padding(.top)
    }
}

// ViewModel
@MainActor
class StoreViewModel: ObservableObject {
    @Published var subscriptionProducts: [Product] = []
    @Published var nonConsumables: [Product] = []
    @Published var consumables: [Product] = []
    @Published var purchasedProductIDs: Set<String> = []
    @Published var isLoading = false
    @Published var errorMessage: String?
    
    private let storeManager = StoreManager()
    
    func loadProducts() async {
        isLoading = true
        await storeManager.loadProducts()
        
        let allProducts = storeManager.products
        subscriptionProducts = allProducts.filter { $0.type == .autoRenewable }
        nonConsumables = allProducts.filter { $0.type == .nonConsumable }
        consumables = allProducts.filter { $0.type == .consumable }
        
        await updateEntitlements()
        isLoading = false
    }
    
    func purchase(_ product: Product) async {
        await storeManager.purchase(product)
        await updateEntitlements()
    }
    
    func updateEntitlements() async {
        await storeManager.updateCustomerProductStatus()
        purchasedProductIDs = storeManager.purchasedProductIDs
    }
    
    func hasActiveSubscription(groupID: String) async -> Bool {
        return await storeManager.hasActiveSubscription(groupID: groupID)
    }
}
```

---

## 22. Practical Exercises

### แบบฝึกหัดที่ 1: Coin Shop

```swift
import StoreKit
import SwiftUI

// Model
struct CoinPackage: Identifiable {
    let id: String  // Product ID
    let coins: Int
    let bonus: Int  // โบนัส coins
    let isPopular: Bool
}

@MainActor
class CoinShopViewModel: ObservableObject {
    @Published var products: [Product] = []
    @Published var userCoins: Int = 0
    @Published var isPurchasing = false
    @Published var errorMessage: String?
    
    let coinPackages: [CoinPackage] = [
        CoinPackage(id: "com.myapp.coins.100", coins: 100, bonus: 0, isPopular: false),
        CoinPackage(id: "com.myapp.coins.500", coins: 500, bonus: 50, isPopular: true),
        CoinPackage(id: "com.myapp.coins.1000", coins: 1000, bonus: 200, isPopular: false)
    ]
    
    private var transactionListener: Task<Void, Error>?
    
    init() {
        // โหลด saved coins
        userCoins = UserDefaults.standard.integer(forKey: "userCoins")
        
        // เริ่ม listen transactions
        transactionListener = listenForTransactions()
        
        Task { await loadProducts() }
    }
    
    deinit {
        transactionListener?.cancel()
    }
    
    func loadProducts() async {
        do {
            let productIDs = Set(coinPackages.map { $0.id })
            products = try await Product.products(for: productIDs)
            products.sort { p1, p2 in
                let c1 = coinPackages.first { $0.id == p1.id }?.coins ?? 0
                let c2 = coinPackages.first { $0.id == p2.id }?.coins ?? 0
                return c1 < c2
            }
        } catch {
            errorMessage = "ไม่สามารถโหลด products: \(error.localizedDescription)"
        }
    }
    
    func purchase(_ product: Product) async {
        isPurchasing = true
        do {
            let result = try await product.purchase()
            
            switch result {
            case .success(let verification):
                if case .verified(let transaction) = verification {
                    await grantCoins(for: transaction.productID)
                    await transaction.finish()
                }
            case .userCancelled:
                break
            case .pending:
                errorMessage = "Transaction รอการอนุมัติ"
            @unknown default:
                break
            }
        } catch {
            errorMessage = "เกิดข้อผิดพลาด: \(error.localizedDescription)"
        }
        isPurchasing = false
    }
    
    func grantCoins(for productID: String) async {
        guard let package = coinPackages.first(where: { $0.id == productID }) else { return }
        let totalCoins = package.coins + package.bonus
        userCoins += totalCoins
        UserDefaults.standard.set(userCoins, forKey: "userCoins")
        print("เพิ่ม \(totalCoins) coins (รวม \(package.bonus) โบนัส)")
    }
    
    private func listenForTransactions() -> Task<Void, Error> {
        Task.detached {
            for await result in Transaction.updates {
                if case .verified(let transaction) = result {
                    await self.grantCoins(for: transaction.productID)
                    await transaction.finish()
                }
            }
        }
    }
    
    func package(for product: Product) -> CoinPackage? {
        coinPackages.first { $0.id == product.id }
    }
}

struct CoinShopView: View {
    @StateObject private var viewModel = CoinShopViewModel()
    
    var body: some View {
        NavigationView {
            VStack {
                // Header - แสดง coins ปัจจุบัน
                HStack {
                    Image(systemName: "dollarsign.circle.fill")
                        .foregroundColor(.yellow)
                        .font(.title2)
                    Text("\(viewModel.userCoins) Coins")
                        .font(.title2.bold())
                }
                .padding()
                .background(Color(.secondarySystemBackground))
                .cornerRadius(12)
                .padding()
                
                if viewModel.products.isEmpty {
                    ProgressView("กำลังโหลด...")
                        .padding()
                } else {
                    ScrollView {
                        VStack(spacing: 12) {
                            ForEach(viewModel.products) { product in
                                CoinPackageCard(
                                    product: product,
                                    package: viewModel.package(for: product),
                                    onPurchase: {
                                        Task { await viewModel.purchase(product) }
                                    }
                                )
                            }
                        }
                        .padding()
                    }
                }
            }
            .navigationTitle("Coin Shop")
            .overlay {
                if viewModel.isPurchasing {
                    Color.black.opacity(0.3)
                        .ignoresSafeArea()
                    ProgressView("กำลังซื้อ...")
                        .padding()
                        .background(Color.white)
                        .cornerRadius(12)
                }
            }
            .alert(
                "เกิดข้อผิดพลาด",
                isPresented: .init(
                    get: { viewModel.errorMessage != nil },
                    set: { if !$0 { viewModel.errorMessage = nil } }
                )
            ) {
                Button("ตกลง") { viewModel.errorMessage = nil }
            } message: {
                Text(viewModel.errorMessage ?? "")
            }
        }
    }
}

struct CoinPackageCard: View {
    let product: Product
    let package: CoinPackage?
    let onPurchase: () -> Void
    
    var body: some View {
        ZStack {
            RoundedRectangle(cornerRadius: 16)
                .fill(Color(.secondarySystemBackground))
            
            if package?.isPopular == true {
                RoundedRectangle(cornerRadius: 16)
                    .stroke(Color.yellow, lineWidth: 2)
                
                Text("ยอดนิยม")
                    .font(.caption.bold())
                    .foregroundColor(.yellow)
                    .padding(.horizontal, 8)
                    .padding(.vertical, 4)
                    .background(Color.yellow.opacity(0.2))
                    .cornerRadius(8)
                    .frame(maxWidth: .infinity, maxHeight: .infinity, alignment: .topTrailing)
                    .padding(8)
            }
            
            HStack(spacing: 16) {
                // Icon
                ZStack {
                    Circle()
                        .fill(Color.yellow.opacity(0.2))
                        .frame(width: 60, height: 60)
                    Image(systemName: "dollarsign.circle.fill")
                        .font(.title)
                        .foregroundColor(.yellow)
                }
                
                // Info
                VStack(alignment: .leading, spacing: 4) {
                    Text(product.displayName)
                        .font(.headline)
                    
                    if let pkg = package, pkg.bonus > 0 {
                        Text("+\(pkg.bonus) โบนัส!")
                            .font(.caption)
                            .foregroundColor(.green)
                    }
                }
                
                Spacer()
                
                // Price
                Button(action: onPurchase) {
                    Text(product.displayPrice)
                        .font(.callout.bold())
                        .padding(.horizontal, 16)
                        .padding(.vertical, 8)
                        .background(Color.blue)
                        .foregroundColor(.white)
                        .cornerRadius(8)
                }
            }
            .padding(16)
        }
    }
}
```

---

## 23. Building a Subscription App

แอป subscription ที่สมบูรณ์

```swift
import StoreKit
import SwiftUI

// MARK: - Models

struct SubscriptionTier: Identifiable {
    let id: String
    let name: String
    let features: [String]
    let productID: String
    let period: String
    let isRecommended: Bool
}

// MARK: - ViewModel

@MainActor
class SubscriptionAppViewModel: ObservableObject {
    @Published var products: [Product] = []
    @Published var activeSubscription: Transaction?
    @Published var subscriptionState: SubscriptionState = .notSubscribed
    @Published var isLoading = false
    @Published var purchaseResult: PurchaseResult?
    
    private var transactionListener: Task<Void, Error>?
    
    let tiers: [SubscriptionTier] = [
        SubscriptionTier(
            id: "basic",
            name: "Basic",
            features: ["ฟีเจอร์พื้นฐาน", "Storage 5GB", "Support ทั่วไป"],
            productID: "com.myapp.basic.monthly",
            period: "ต่อเดือน",
            isRecommended: false
        ),
        SubscriptionTier(
            id: "premium",
            name: "Premium",
            features: ["ทุกฟีเจอร์ใน Basic", "Storage 50GB", "Priority Support", "ไม่มีโฆษณา", "Export HD"],
            productID: "com.myapp.premium.monthly",
            period: "ต่อเดือน",
            isRecommended: true
        ),
        SubscriptionTier(
            id: "annual",
            name: "Premium ราคาดีที่สุด",
            features: ["ทุกอย่างใน Premium", "ประหยัด 30%!"],
            productID: "com.myapp.premium.yearly",
            period: "ต่อปี",
            isRecommended: false
        )
    ]
    
    init() {
        transactionListener = listenForTransactions()
        Task {
            isLoading = true
            await loadProducts()
            await checkSubscriptionStatus()
            isLoading = false
        }
    }
    
    deinit {
        transactionListener?.cancel()
    }
    
    func loadProducts() async {
        let productIDs = Set(tiers.map { $0.productID })
        do {
            products = try await Product.products(for: productIDs)
            products.sort { $0.price < $1.price }
        } catch {
            print("ไม่สามารถโหลด products: \(error)")
        }
    }
    
    func checkSubscriptionStatus() async {
        for await result in Transaction.currentEntitlements {
            if case .verified(let transaction) = result,
               transaction.productType == .autoRenewable,
               transaction.revocationDate == nil {
                
                if let expiry = transaction.expirationDate, expiry > Date() {
                    activeSubscription = transaction
                    subscriptionState = .active
                    return
                }
            }
        }
        subscriptionState = .notSubscribed
    }
    
    func subscribe(to product: Product) async {
        isLoading = true
        do {
            let result = try await product.purchase()
            
            switch result {
            case .success(let verification):
                if case .verified(let transaction) = verification {
                    activeSubscription = transaction
                    subscriptionState = .active
                    await transaction.finish()
                    purchaseResult = .success(product.displayName)
                }
            case .userCancelled:
                purchaseResult = .cancelled
            case .pending:
                purchaseResult = .pending
            @unknown default:
                break
            }
        } catch {
            purchaseResult = .failed(error.localizedDescription)
        }
        isLoading = false
    }
    
    func restorePurchases() async {
        isLoading = true
        do {
            try await AppStore.sync()
            await checkSubscriptionStatus()
            if subscriptionState == .active {
                purchaseResult = .restored
            } else {
                purchaseResult = .nothingToRestore
            }
        } catch {
            purchaseResult = .failed(error.localizedDescription)
        }
        isLoading = false
    }
    
    func product(for tier: SubscriptionTier) -> Product? {
        products.first { $0.id == tier.productID }
    }
    
    private func listenForTransactions() -> Task<Void, Error> {
        Task.detached {
            for await result in Transaction.updates {
                if case .verified(let transaction) = result {
                    await self.checkSubscriptionStatus()
                    await transaction.finish()
                }
            }
        }
    }
}

enum PurchaseResult {
    case success(String)
    case cancelled
    case pending
    case failed(String)
    case restored
    case nothingToRestore
}

// MARK: - Views

struct SubscriptionAppView: View {
    @StateObject private var viewModel = SubscriptionAppViewModel()
    @State private var showManageSubscriptions = false
    
    var body: some View {
        NavigationView {
            ZStack {
                // Background gradient
                LinearGradient(
                    colors: [Color.purple.opacity(0.3), Color.blue.opacity(0.1)],
                    startPoint: .topLeading,
                    endPoint: .bottomTrailing
                )
                .ignoresSafeArea()
                
                ScrollView {
                    VStack(spacing: 24) {
                        // Header
                        SubscriptionHeaderView(state: viewModel.subscriptionState)
                        
                        // Features comparison
                        if viewModel.subscriptionState == .notSubscribed {
                            FeatureComparisonView()
                        }
                        
                        // Subscription tiers
                        VStack(spacing: 16) {
                            ForEach(viewModel.tiers) { tier in
                                if let product = viewModel.product(for: tier) {
                                    SubscriptionTierCard(
                                        tier: tier,
                                        product: product,
                                        isCurrentPlan: viewModel.activeSubscription?.productID == tier.productID,
                                        onSubscribe: {
                                            Task { await viewModel.subscribe(to: product) }
                                        }
                                    )
                                }
                            }
                        }
                        .padding(.horizontal)
                        
                        // Manage & Restore
                        VStack(spacing: 12) {
                            if viewModel.subscriptionState == .active {
                                Button("จัดการ Subscription") {
                                    showManageSubscriptions = true
                                }
                                .buttonStyle(.bordered)
                            }
                            
                            Button("กู้คืนการซื้อ") {
                                Task { await viewModel.restorePurchases() }
                            }
                            .foregroundColor(.secondary)
                        }
                        
                        TermsView()
                    }
                    .padding()
                }
            }
            .navigationTitle("Premium")
            .navigationBarTitleDisplayMode(.large)
            .manageSubscriptionsSheet(isPresented: $showManageSubscriptions)
            .overlay {
                if viewModel.isLoading {
                    ProgressView()
                        .padding(20)
                        .background(Color.white.opacity(0.9))
                        .cornerRadius(12)
                }
            }
        }
        .onChange(of: viewModel.purchaseResult) { result in
            // Handle result alerts
        }
    }
}

struct SubscriptionHeaderView: View {
    let state: SubscriptionState
    
    var body: some View {
        VStack(spacing: 12) {
            Image(systemName: state == .active ? "crown.fill" : "crown")
                .font(.system(size: 70))
                .foregroundColor(state == .active ? .yellow : .gray)
                .symbolEffect(.pulse, options: .repeating, isActive: state != .active)
            
            Text(state == .active ? "Premium Active" : "Upgrade to Premium")
                .font(.title.bold())
            
            Text(state == .active
                ? "คุณกำลังใช้งาน Premium อยู่"
                : "ปลดล็อคฟีเจอร์ทั้งหมดและนำประสบการณ์ของคุณไปอีกระดับ")
                .font(.subheadline)
                .foregroundColor(.secondary)
                .multilineTextAlignment(.center)
        }
        .padding()
    }
}

struct FeatureComparisonView: View {
    let features = [
        ("person.fill", "ไม่จำกัดผู้ใช้", true),
        ("internaldrive.fill", "Storage 50GB", true),
        ("star.fill", "Priority Support", true),
        ("nosign", "ไม่มีโฆษณา", true),
        ("arrow.up.doc.fill", "Export คุณภาพสูง", true)
    ]
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            Text("สิ่งที่คุณจะได้รับ")
                .font(.headline)
                .padding(.horizontal)
            
            ForEach(features, id: \.0) { icon, text, included in
                HStack(spacing: 12) {
                    Image(systemName: included ? "checkmark.circle.fill" : "xmark.circle.fill")
                        .foregroundColor(included ? .green : .red)
                    
                    Image(systemName: icon)
                        .frame(width: 20)
                        .foregroundColor(.secondary)
                    
                    Text(text)
                }
                .padding(.horizontal)
            }
        }
        .padding(.vertical)
        .background(Color(.secondarySystemBackground))
        .cornerRadius(16)
        .padding(.horizontal)
    }
}

struct SubscriptionTierCard: View {
    let tier: SubscriptionTier
    let product: Product
    let isCurrentPlan: Bool
    let onSubscribe: () -> Void
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            // Header
            HStack {
                VStack(alignment: .leading) {
                    HStack {
                        Text(tier.name)
                            .font(.title2.bold())
                        
                        if tier.isRecommended {
                            Text("แนะนำ")
                                .font(.caption.bold())
                                .padding(.horizontal, 8)
                                .padding(.vertical, 3)
                                .background(Color.blue)
                                .foregroundColor(.white)
                                .cornerRadius(8)
                        }
                    }
                    
                    HStack(alignment: .firstTextBaseline) {
                        Text(product.displayPrice)
                            .font(.title3.bold())
                            .foregroundColor(.primary)
                        Text(tier.period)
                            .font(.caption)
                            .foregroundColor(.secondary)
                    }
                }
                
                Spacer()
                
                if isCurrentPlan {
                    Image(systemName: "checkmark.circle.fill")
                        .font(.title2)
                        .foregroundColor(.green)
                }
            }
            
            Divider()
            
            // Features
            VStack(alignment: .leading, spacing: 6) {
                ForEach(tier.features, id: \.self) { feature in
                    HStack(spacing: 8) {
                        Image(systemName: "checkmark")
                            .font(.caption.bold())
                            .foregroundColor(.green)
                        Text(feature)
                            .font(.subheadline)
                    }
                }
            }
            
            // Subscribe Button
            if !isCurrentPlan {
                Button(action: onSubscribe) {
                    Text(tier.isRecommended ? "เริ่มต้น \(product.displayPrice)" : "สมัคร \(product.displayPrice)")
                        .font(.callout.bold())
                        .frame(maxWidth: .infinity)
                        .padding()
                        .background(tier.isRecommended ? Color.blue : Color.secondary)
                        .foregroundColor(.white)
                        .cornerRadius(12)
                }
            }
        }
        .padding()
        .background(Color(.secondarySystemBackground))
        .cornerRadius(16)
        .overlay(
            RoundedRectangle(cornerRadius: 16)
                .stroke(tier.isRecommended ? Color.blue : Color.clear, lineWidth: 2)
        )
    }
}
```

---

## 24. Summary

### สิ่งที่ได้เรียนรู้ในบทนี้

1. **IAP Types**: Consumable, Non-Consumable, Auto-Renewable Subscription, Non-Renewing Subscription
2. **StoreKit 2**: Modern async/await API
3. **Product Loading**: `Product.products(for:)`
4. **Purchase Flow**: `product.purchase()` และจัดการ results
5. **Transaction Verification**: VerificationResult และ JWS
6. **Entitlements**: `Transaction.currentEntitlements`
7. **Updates**: `Transaction.updates` listener
8. **Subscription Management**: Renewal info, states
9. **Offers**: Introductory, Promotional, Offer Codes
10. **Family Sharing**: Ownership types
11. **Testing**: Xcode StoreKit Configuration, SKTestSession
12. **Legacy**: Receipt validation, SKPaymentTransactionObserver
13. **Server-Side**: Validation endpoints
14. **SwiftUI Integration**: Complete store UI

### Best Practices

```swift
/*
StoreKit Best Practices:

1. เสมอ verify transactions ก่อน grant access
2. Finish transaction หลัง grant เสมอ
3. Listen Transaction.updates ตลอดชีวิตของ app
4. ตรวจสอบ revocationDate ก่อน grant access
5. จัดการ .pending (Ask to Buy) อย่างถูกต้อง
6. ทดสอบด้วย StoreKit Configuration file
7. ทดสอบ edge cases: expiry, revocation, billing failures
8. แสดงปุ่ม Restore Purchases ตามกฎของ Apple
9. Server-side validation สำหรับ consumables สำคัญ
10. ใช้ Subscription Group อย่างถูกต้อง
*/
```

### กฎของ Apple สำคัญ

```swift
/*
Apple Guidelines สำหรับ IAP:

1. ต้องใช้ IAP สำหรับ digital goods และ services
2. ห้ามลิงก์ไปยัง external purchase page
3. ต้องแสดง price อย่างชัดเจนก่อน purchase
4. ต้องมีปุ่ม Restore Purchases
5. Free trial ต้องระบุชัดเจน
6. Auto-renewing subscription ต้องมี disclosure
7. Subscription management ต้องเข้าถึงได้จาก Settings
8. Pricing ต้องใช้ App Store Pricing Matrix
9. ห้ามใช้ IAP สำหรับ physical goods
10. Game currencies ต้องใช้ IAP
*/
```

---

*จบ Part 52: StoreKit และ In-App Purchase*
