# ตอนที่ 40: Security Best Practices ใน iOS

## บทนำ

ความปลอดภัย (Security) เป็นสิ่งที่ขาดไม่ได้ในการพัฒนา iOS Applications สมัยใหม่ แอปพลิเคชันที่ไม่ปลอดภัยสามารถทำให้ข้อมูลของผู้ใช้รั่วไหล หรือเปิดช่องโหว่สำหรับการโจมตีได้

ในบทเรียนนี้เราจะเรียนรู้:
- หลักการพื้นฐานด้านความปลอดภัย
- การเก็บข้อมูลสำคัญใน Keychain
- การยืนยันตัวตนด้วย Biometrics
- การเข้ารหัสด้วย CryptoKit
- การป้องกันการสื่อสารผ่านเครือข่าย
- แนวปฏิบัติด้านความปลอดภัยที่ดี

---

## 40.1 Security Fundamentals สำหรับ iOS

### หลักการ CIA Triad

```swift
// CIA Triad คือ Framework พื้นฐานด้าน Security
// C = Confidentiality (ความลับ): ข้อมูลเข้าถึงได้เฉพาะผู้มีสิทธิ์
// I = Integrity (ความสมบูรณ์): ข้อมูลไม่ถูกแก้ไขโดยไม่ได้รับอนุญาต
// A = Availability (ความพร้อมใช้): ข้อมูลพร้อมใช้งานเมื่อต้องการ

// ตัวอย่าง: การออกแบบระบบตาม CIA Triad
struct UserProfile {
    let id: String
    let username: String
    // password ไม่ควรเก็บใน model - ควรเก็บใน Keychain
    var encryptedData: Data?  // Confidentiality
    var checksum: String      // Integrity
    let lastModified: Date    // สำหรับ Audit
}

// Defense in Depth: ป้องกันหลายชั้น
class SecurityLayeredDefense {
    // Layer 1: App Transport Security (ATS) - Network
    // Layer 2: Keychain - Data at Rest
    // Layer 3: CryptoKit - Encryption
    // Layer 4: Biometrics - Authentication
    // Layer 5: Code Signing - App Integrity
    
    func demonstrateDefenseInDepth() {
        print("Layer 1: Force HTTPS via ATS in Info.plist")
        print("Layer 2: Store secrets in Keychain, not UserDefaults")
        print("Layer 3: Encrypt sensitive data before storage")
        print("Layer 4: Require biometric for sensitive operations")
        print("Layer 5: Enable Hardened Runtime and app signing")
    }
}

// Threat Modeling สำหรับ iOS Apps
struct ThreatModel {
    // ภัยคุกคามที่พบบ่อย:
    enum Threat {
        case insecureDataStorage    // เก็บข้อมูลไม่ปลอดภัย
        case insecureCommunication  // สื่อสารไม่ปลอดภัย
        case insecureAuthentication // ยืนยันตัวตนไม่ปลอดภัย
        case clientSideInjection    // Code Injection
        case reverseEngineering     // การ Reverse Engineer
        case dataTampering          // การแก้ไขข้อมูล
        case unauthorizedAccess     // การเข้าถึงโดยไม่ได้รับอนุญาต
    }
    
    // แนวทางป้องกัน
    static func mitigation(for threat: Threat) -> String {
        switch threat {
        case .insecureDataStorage:
            return "ใช้ Keychain, เข้ารหัส Sensitive Data"
        case .insecureCommunication:
            return "บังคับ HTTPS, ใช้ Certificate Pinning"
        case .insecureAuthentication:
            return "ใช้ Biometrics, Strong Password Policy"
        case .clientSideInjection:
            return "Validate และ Sanitize Input ทั้งหมด"
        case .reverseEngineering:
            return "Obfuscation, Anti-tampering checks"
        case .dataTampering:
            return "Cryptographic Signatures, Checksums"
        case .unauthorizedAccess:
            return "Proper Authorization Checks"
        }
    }
}
```

---

## 40.2 Keychain Services

### Keychain เป็น Secure Storage สำหรับ Sensitive Data

```swift
import Security
import Foundation

// Keychain Wrapper ที่ใช้งานง่าย
class KeychainManager {
    
    enum KeychainError: Error {
        case itemNotFound
        case duplicateItem
        case unexpectedStatus(OSStatus)
        case encodingError
        case decodingError
    }
    
    // เพิ่มข้อมูลใน Keychain
    static func save(key: String, data: Data, service: String = Bundle.main.bundleIdentifier ?? "app") throws {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: key,
            kSecValueData as String: data,
            kSecAttrAccessible as String: kSecAttrAccessibleWhenUnlockedThisDeviceOnly
        ]
        
        // ลบ item เดิมก่อน (ถ้ามี)
        SecItemDelete(query as CFDictionary)
        
        let status = SecItemAdd(query as CFDictionary, nil)
        
        guard status == errSecSuccess else {
            throw KeychainError.unexpectedStatus(status)
        }
    }
    
    // อ่านข้อมูลจาก Keychain
    static func load(key: String, service: String = Bundle.main.bundleIdentifier ?? "app") throws -> Data {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: key,
            kSecReturnData as String: true,
            kSecMatchLimit as String: kSecMatchLimitOne
        ]
        
        var result: AnyObject?
        let status = SecItemCopyMatching(query as CFDictionary, &result)
        
        switch status {
        case errSecSuccess:
            guard let data = result as? Data else {
                throw KeychainError.decodingError
            }
            return data
        case errSecItemNotFound:
            throw KeychainError.itemNotFound
        default:
            throw KeychainError.unexpectedStatus(status)
        }
    }
    
    // ลบข้อมูลจาก Keychain
    static func delete(key: String, service: String = Bundle.main.bundleIdentifier ?? "app") throws {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: key
        ]
        
        let status = SecItemDelete(query as CFDictionary)
        
        guard status == errSecSuccess || status == errSecItemNotFound else {
            throw KeychainError.unexpectedStatus(status)
        }
    }
    
    // ตรวจสอบว่ามี item หรือไม่
    static func exists(key: String, service: String = Bundle.main.bundleIdentifier ?? "app") -> Bool {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: key
        ]
        
        let status = SecItemCopyMatching(query as CFDictionary, nil)
        return status == errSecSuccess
    }
}

// Extension สำหรับ String
extension KeychainManager {
    static func save(key: String, string: String) throws {
        guard let data = string.data(using: .utf8) else {
            throw KeychainError.encodingError
        }
        try save(key: key, data: data)
    }
    
    static func loadString(key: String) throws -> String {
        let data = try load(key: key)
        guard let string = String(data: data, encoding: .utf8) else {
            throw KeychainError.decodingError
        }
        return string
    }
}

// Extension สำหรับ Codable types
extension KeychainManager {
    static func save<T: Encodable>(key: String, value: T) throws {
        let encoder = JSONEncoder()
        let data = try encoder.encode(value)
        try save(key: key, data: data)
    }
    
    static func load<T: Decodable>(key: String, type: T.Type) throws -> T {
        let data = try load(key: key)
        let decoder = JSONDecoder()
        return try decoder.decode(type, from: data)
    }
}
```

---

## 40.3 SecItem API

### การใช้ SecItem API โดยตรง

```swift
import Security

class SecItemExamples {
    
    // การจัดการ Internet Passwords
    static func saveInternetPassword(
        server: String,
        account: String,
        password: String
    ) -> Bool {
        guard let passwordData = password.data(using: .utf8) else { return false }
        
        let query: [String: Any] = [
            kSecClass as String: kSecClassInternetPassword,
            kSecAttrServer as String: server,
            kSecAttrAccount as String: account,
            kSecValueData as String: passwordData,
            kSecAttrAccessible as String: kSecAttrAccessibleAfterFirstUnlockThisDeviceOnly
        ]
        
        // ลบ item เดิมก่อน
        let deleteQuery: [String: Any] = [
            kSecClass as String: kSecClassInternetPassword,
            kSecAttrServer as String: server,
            kSecAttrAccount as String: account
        ]
        SecItemDelete(deleteQuery as CFDictionary)
        
        return SecItemAdd(query as CFDictionary, nil) == errSecSuccess
    }
    
    // การจัดการ Certificates
    static func saveCertificate(_ certData: Data, label: String) -> Bool {
        guard let cert = SecCertificateCreateWithData(nil, certData as CFData) else {
            return false
        }
        
        let query: [String: Any] = [
            kSecClass as String: kSecClassCertificate,
            kSecValueRef as String: cert,
            kSecAttrLabel as String: label
        ]
        
        return SecItemAdd(query as CFDictionary, nil) == errSecSuccess
    }
    
    // การจัดการ Keys
    static func generateAndStoreKey(tag: String) -> SecKey? {
        let tagData = tag.data(using: .utf8)!
        
        let attributes: [String: Any] = [
            kSecAttrKeyType as String: kSecAttrKeyTypeECSECPrimeRandom,
            kSecAttrKeySizeInBits as String: 256,
            kSecAttrTokenID as String: kSecAttrTokenIDSecureEnclave,
            kSecPrivateKeyAttrs as String: [
                kSecAttrIsPermanent as String: true,
                kSecAttrApplicationTag as String: tagData,
                kSecAttrAccessControl as String: createAccessControl()
            ]
        ]
        
        var error: Unmanaged<CFError>?
        guard let privateKey = SecKeyCreateRandomKey(attributes as CFDictionary, &error) else {
            print("Error creating key: \(error!.takeRetainedValue())")
            return nil
        }
        
        return privateKey
    }
    
    static func createAccessControl() -> SecAccessControl? {
        return SecAccessControlCreateWithFlags(
            nil,
            kSecAttrAccessibleWhenUnlockedThisDeviceOnly,
            [.biometryCurrentSet, .privateKeyUsage],
            nil
        )
    }
    
    // ดึง Key จาก Keychain
    static func loadKey(tag: String) -> SecKey? {
        let tagData = tag.data(using: .utf8)!
        
        let query: [String: Any] = [
            kSecClass as String: kSecClassKey,
            kSecAttrKeyType as String: kSecAttrKeyTypeECSECPrimeRandom,
            kSecAttrApplicationTag as String: tagData,
            kSecReturnRef as String: true
        ]
        
        var result: AnyObject?
        let status = SecItemCopyMatching(query as CFDictionary, &result)
        
        guard status == errSecSuccess else { return nil }
        return result as! SecKey?
    }
}
```

---

## 40.4 การเก็บข้อมูล Sensitive อย่างปลอดภัย

### Best Practices สำหรับ Data Storage

```swift
import Foundation

// ผิด: เก็บ Token ใน UserDefaults
class InsecureStorage {
    static func saveTokenBadly(_ token: String) {
        // ❌ UserDefaults ไม่ปลอดภัย - ไม่ถูกเข้ารหัส
        UserDefaults.standard.set(token, forKey: "authToken")
        
        // ❌ เก็บใน plain text file
        let documentsDir = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first!
        let fileURL = documentsDir.appendingPathComponent("token.txt")
        try? token.write(to: fileURL, atomically: true, encoding: .utf8)
        
        // ❌ เก็บใน NSCache (หาย เมื่อ app restart หรือ Memory pressure)
        NSCache<NSString, NSString>().setObject(token as NSString, forKey: "token" as NSString)
    }
}

// ถูก: เก็บ Token ใน Keychain
class SecureStorage {
    private static let tokenKey = "com.example.app.authToken"
    
    static func saveToken(_ token: String) {
        do {
            try KeychainManager.save(key: tokenKey, string: token)
            print("Token saved securely in Keychain")
        } catch {
            print("Failed to save token: \(error)")
        }
    }
    
    static func loadToken() -> String? {
        do {
            return try KeychainManager.loadString(key: tokenKey)
        } catch {
            print("Token not found: \(error)")
            return nil
        }
    }
    
    static func clearToken() {
        try? KeychainManager.delete(key: tokenKey)
    }
    
    // สำหรับ Sensitive Data ที่ต้องเก็บใน File
    static func saveSecureFile(_ data: Data, filename: String) throws {
        let documentsDir = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first!
        let fileURL = documentsDir.appendingPathComponent(filename)
        
        // เข้ารหัสก่อน write
        let encryptedData = try EncryptionHelper.encrypt(data)
        try encryptedData.write(to: fileURL, options: [.completeFileProtection])
        // .completeFileProtection = ไฟล์ถูกเข้ารหัสเพิ่มเติมโดย OS
    }
    
    // ปิดกั้น File จาก iCloud Backup
    static func excludeFromBackup(url: URL) throws {
        var mutableURL = url
        var resourceValues = URLResourceValues()
        resourceValues.isExcludedFromBackup = true
        try mutableURL.setResourceValues(resourceValues)
    }
}

// Placeholder สำหรับ EncryptionHelper (จะ implement ใน CryptoKit section)
struct EncryptionHelper {
    static func encrypt(_ data: Data) throws -> Data {
        // Placeholder - จะ implement ด้วย CryptoKit
        return data
    }
    
    static func decrypt(_ data: Data) throws -> Data {
        // Placeholder - จะ implement ด้วย CryptoKit
        return data
    }
}

// Data Classification
enum DataClassification {
    case public_      // เผยแพร่ได้
    case internal_    // ใช้ภายในแอปเท่านั้น
    case confidential // ข้อมูลส่วนตัวของผู้ใช้
    case restricted   // ข้อมูลสำคัญมาก (passwords, tokens)
    
    var storageRecommendation: String {
        switch self {
        case .public_:
            return "UserDefaults หรือ File System ปกติ"
        case .internal_:
            return "File System พร้อม Data Protection"
        case .confidential:
            return "Keychain หรือ Encrypted File"
        case .restricted:
            return "Keychain เท่านั้น พร้อม Biometric Protection"
        }
    }
}
```

---

## 40.5 Biometric Authentication

### การใช้ Face ID และ Touch ID

```swift
import LocalAuthentication
import Foundation

class BiometricAuthManager {
    
    private let context = LAContext()
    
    enum BiometricError: Error, LocalizedError {
        case notAvailable(LABiometryType)
        case notEnrolled
        case lockout
        case cancelled
        case failed(String)
        
        var errorDescription: String? {
            switch self {
            case .notAvailable(let type):
                return "\(type) ไม่พร้อมใช้งานบนอุปกรณ์นี้"
            case .notEnrolled:
                return "ยังไม่ได้ลงทะเบียน Biometric"
            case .lockout:
                return "Biometric ถูกล็อค กรุณาใช้ Passcode"
            case .cancelled:
                return "ยกเลิกการยืนยันตัวตน"
            case .failed(let reason):
                return "การยืนยันตัวตนล้มเหลว: \(reason)"
            }
        }
    }
    
    // ตรวจสอบว่า Biometric พร้อมใช้งานหรือไม่
    var biometricType: LABiometryType {
        var error: NSError?
        guard context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: &error) else {
            return .none
        }
        return context.biometryType
    }
    
    var isBiometricAvailable: Bool {
        var error: NSError?
        return context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: &error)
    }
    
    // ยืนยันตัวตนด้วย Biometric
    func authenticate(
        reason: String,
        fallbackTitle: String? = "ใช้ Passcode",
        completion: @escaping (Result<Void, BiometricError>) -> Void
    ) {
        var error: NSError?
        
        guard context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: &error) else {
            if let laError = error as? LAError {
                switch laError.code {
                case .biometryNotAvailable:
                    completion(.failure(.notAvailable(context.biometryType)))
                case .biometryNotEnrolled:
                    completion(.failure(.notEnrolled))
                case .biometryLockout:
                    completion(.failure(.lockout))
                default:
                    completion(.failure(.failed(laError.localizedDescription)))
                }
            }
            return
        }
        
        // กำหนด Fallback title (nil = ซ่อน button, "" = ซ่อน button, มีข้อความ = แสดง)
        context.localizedFallbackTitle = fallbackTitle
        
        context.evaluatePolicy(
            .deviceOwnerAuthenticationWithBiometrics,
            localizedReason: reason
        ) { success, error in
            DispatchQueue.main.async {
                if success {
                    completion(.success(()))
                } else if let laError = error as? LAError {
                    switch laError.code {
                    case .userCancel, .appCancel, .systemCancel:
                        completion(.failure(.cancelled))
                    case .biometryLockout:
                        completion(.failure(.lockout))
                    default:
                        completion(.failure(.failed(laError.localizedDescription)))
                    }
                }
            }
        }
    }
    
    // ยืนยันตัวตนพร้อม Passcode Fallback
    func authenticateWithFallback(
        reason: String,
        completion: @escaping (Result<Void, Error>) -> Void
    ) {
        var error: NSError?
        
        guard context.canEvaluatePolicy(.deviceOwnerAuthentication, error: &error) else {
            completion(.failure(error!))
            return
        }
        
        // .deviceOwnerAuthentication รองรับทั้ง Biometric และ Passcode
        context.evaluatePolicy(
            .deviceOwnerAuthentication,
            localizedReason: reason
        ) { success, error in
            DispatchQueue.main.async {
                if success {
                    completion(.success(()))
                } else {
                    completion(.failure(error!))
                }
            }
        }
    }
    
    // Modern Async version (iOS 16+)
    @available(iOS 16.0, *)
    func authenticateAsync(reason: String) async throws {
        try await context.evaluatePolicy(
            .deviceOwnerAuthenticationWithBiometrics,
            localizedReason: reason
        )
    }
}
```

---

## 40.6 LocalAuthentication Framework

### การใช้ LAContext แบบ Advanced

```swift
import LocalAuthentication

class AdvancedBiometricManager {
    
    // Biometric Protected Keychain Access
    static func saveSensitiveData(_ data: Data, key: String) throws {
        // สร้าง Access Control ที่ต้องการ Biometric
        guard let accessControl = SecAccessControlCreateWithFlags(
            nil,
            kSecAttrAccessibleWhenUnlockedThisDeviceOnly,
            .biometryCurrentSet,  // ต้องการ Biometric ที่ลงทะเบียนปัจจุบัน
            nil
        ) else {
            throw NSError(domain: "BiometricError", code: -1)
        }
        
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: key,
            kSecValueData as String: data,
            kSecAttrAccessControl as String: accessControl
        ]
        
        SecItemDelete(query as CFDictionary)
        let status = SecItemAdd(query as CFDictionary, nil)
        
        guard status == errSecSuccess else {
            throw NSError(domain: "KeychainError", code: Int(status))
        }
    }
    
    // อ่านข้อมูลที่ต้องการ Biometric
    static func loadSensitiveData(key: String, reason: String) async throws -> Data {
        let context = LAContext()
        
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: key,
            kSecReturnData as String: true,
            kSecMatchLimit as String: kSecMatchLimitOne,
            kSecUseAuthenticationContext as String: context
        ]
        
        // Context จะขอ Biometric อัตโนมัติเมื่อ access item
        var result: AnyObject?
        let status = SecItemCopyMatching(query as CFDictionary, &result)
        
        guard status == errSecSuccess, let data = result as? Data else {
            throw NSError(domain: "KeychainError", code: Int(status))
        }
        
        return data
    }
    
    // การจัดการ Biometric Changes
    static func detectBiometricChanges() -> Bool {
        let savedDomainState = UserDefaults.standard.data(forKey: "biometricDomainState")
        
        let context = LAContext()
        var error: NSError?
        
        guard context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: &error) else {
            return false
        }
        
        let currentDomainState = context.evaluatedPolicyDomainState
        
        if let saved = savedDomainState,
           let current = currentDomainState,
           saved != current {
            // Biometric เปลี่ยนแปลง (เพิ่มหรือลบ fingerprint/face)
            print("⚠️ Biometric changes detected! Require re-authentication")
            UserDefaults.standard.set(current, forKey: "biometricDomainState")
            return true
        }
        
        if let current = currentDomainState {
            UserDefaults.standard.set(current, forKey: "biometricDomainState")
        }
        
        return false
    }
}

// ViewController integration
import UIKit

class SecureViewController: UIViewController {
    
    let biometricManager = BiometricAuthManager()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        checkBiometricAvailability()
    }
    
    func checkBiometricAvailability() {
        switch biometricManager.biometricType {
        case .faceID:
            print("Face ID พร้อมใช้งาน")
            // แสดง Face ID icon
        case .touchID:
            print("Touch ID พร้อมใช้งาน")
            // แสดง Touch ID icon
        case .none:
            print("ไม่มี Biometric")
            // ซ่อน Biometric button
        case .opticID:
            print("Optic ID พร้อมใช้งาน")
        @unknown default:
            break
        }
    }
    
    func performSensitiveAction() {
        let reason: String
        
        switch biometricManager.biometricType {
        case .faceID:
            reason = "ยืนยันตัวตนด้วย Face ID เพื่อดูข้อมูลสำคัญ"
        case .touchID:
            reason = "สัมผัส Touch ID เพื่อดูข้อมูลสำคัญ"
        default:
            reason = "ยืนยันตัวตนเพื่อดูข้อมูลสำคัญ"
        }
        
        biometricManager.authenticate(reason: reason) { [weak self] result in
            switch result {
            case .success:
                self?.showSensitiveData()
            case .failure(let error):
                self?.showError(error.localizedDescription ?? "Authentication failed")
            }
        }
    }
    
    private func showSensitiveData() {
        print("แสดงข้อมูลสำคัญ")
    }
    
    private func showError(_ message: String) {
        let alert = UIAlertController(title: "Error", message: message, preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "OK", style: .default))
        present(alert, animated: true)
    }
}
```

---

## 40.7 App Transport Security (ATS)

### การตั้งค่า ATS ใน Info.plist

```xml
<!-- Info.plist -->
<!-- ATS บังคับ HTTPS โดย Default ตั้งแต่ iOS 9 -->

<!-- ✅ การตั้งค่าที่แนะนำ: ไม่มีการ exception -->
<!-- ATS จะบังคับ HTTPS กับทุก domain -->

<!-- ⚠️ Exception สำหรับ Development เท่านั้น -->
<!--
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSExceptionDomains</key>
    <dict>
        <key>localhost</key>
        <dict>
            <key>NSExceptionAllowsInsecureHTTPLoads</key>
            <true/>
        </dict>
    </dict>
</dict>
-->

<!-- ❌ อย่าทำแบบนี้ใน Production -->
<!--
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <true/>
</dict>
-->
```

```swift
// การตรวจสอบ ATS Compliance ใน Code
class ATSVerifier {
    
    // ตรวจสอบว่า URL ใช้ HTTPS
    static func isSecureURL(_ url: URL) -> Bool {
        return url.scheme?.lowercased() == "https"
    }
    
    // Force HTTPS
    static func enforceHTTPS(_ url: URL) -> URL? {
        guard var components = URLComponents(url: url, resolvingAgainstBaseURL: false) else {
            return nil
        }
        
        if components.scheme?.lowercased() == "http" {
            components.scheme = "https"
            return components.url
        }
        
        return url
    }
    
    // Validate TLS Certificate Manually (ถ้าต้องการ Custom Logic)
    static func createSecureSession() -> URLSession {
        let config = URLSessionConfiguration.default
        config.tlsMinimumSupportedProtocolVersion = .TLSv12
        
        let delegate = SecureURLSessionDelegate()
        return URLSession(configuration: config, delegate: delegate, delegateQueue: nil)
    }
}

class SecureURLSessionDelegate: NSObject, URLSessionDelegate {
    func urlSession(
        _ session: URLSession,
        didReceive challenge: URLAuthenticationChallenge,
        completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void
    ) {
        guard challenge.protectionSpace.authenticationMethod == NSURLAuthenticationMethodServerTrust,
              let serverTrust = challenge.protectionSpace.serverTrust else {
            completionHandler(.cancelAuthenticationChallenge, nil)
            return
        }
        
        // ตรวจสอบ Certificate ด้วย Default Policy
        let evaluation = SecTrustEvaluateWithError(serverTrust, nil)
        
        if evaluation {
            let credential = URLCredential(trust: serverTrust)
            completionHandler(.useCredential, credential)
        } else {
            completionHandler(.cancelAuthenticationChallenge, nil)
        }
    }
}
```

---

## 40.8 Certificate Pinning

### การ Implement Certificate Pinning

```swift
import Foundation
import CommonCrypto

// Certificate Pinning ป้องกัน Man-in-the-Middle Attack
class CertificatePinningManager: NSObject, URLSessionDelegate {
    
    // Hash ของ Certificate หรือ Public Key ที่เราไว้ใจ
    // ได้มาจาก: openssl x509 -in cert.pem -pubkey -noout | openssl pkey -pubin -outform der | openssl dgst -sha256 -binary | base64
    private let pinnedCertificateHashes: Set<String> = [
        "YOUR_CERTIFICATE_HASH_HERE",    // Production cert
        "BACKUP_CERTIFICATE_HASH_HERE"   // Backup cert
    ]
    
    // URLSession พร้อม Certificate Pinning
    lazy var session: URLSession = {
        let config = URLSessionConfiguration.default
        return URLSession(configuration: config, delegate: self, delegateQueue: nil)
    }()
    
    func urlSession(
        _ session: URLSession,
        didReceive challenge: URLAuthenticationChallenge,
        completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void
    ) {
        guard 
            challenge.protectionSpace.authenticationMethod == NSURLAuthenticationMethodServerTrust,
            let serverTrust = challenge.protectionSpace.serverTrust
        else {
            completionHandler(.cancelAuthenticationChallenge, nil)
            return
        }
        
        // ตรวจสอบ Certificate Chain
        guard verifyCertificate(serverTrust) else {
            print("⚠️ Certificate Pinning Failed! Possible MITM attack!")
            completionHandler(.cancelAuthenticationChallenge, nil)
            return
        }
        
        completionHandler(.useCredential, URLCredential(trust: serverTrust))
    }
    
    private func verifyCertificate(_ trust: SecTrust) -> Bool {
        // ดึง Certificate จาก Trust
        guard let certificates = SecTrustCopyCertificateChain(trust) as? [SecCertificate],
              let leafCertificate = certificates.first else {
            return false
        }
        
        // ดึง Public Key
        guard let publicKey = SecCertificateCopyKey(leafCertificate) else {
            return false
        }
        
        // Hash Public Key
        guard let publicKeyHash = hashPublicKey(publicKey) else {
            return false
        }
        
        // เปรียบเทียบกับ Pinned Hashes
        return pinnedCertificateHashes.contains(publicKeyHash)
    }
    
    private func hashPublicKey(_ key: SecKey) -> String? {
        var error: Unmanaged<CFError>?
        guard let publicKeyData = SecKeyCopyExternalRepresentation(key, &error) as Data? else {
            return nil
        }
        
        // SHA256 hash
        var hash = [UInt8](repeating: 0, count: Int(CC_SHA256_DIGEST_LENGTH))
        publicKeyData.withUnsafeBytes { bytes in
            _ = CC_SHA256(bytes.baseAddress, CC_LONG(publicKeyData.count), &hash)
        }
        
        return Data(hash).base64EncodedString()
    }
    
    // ทดสอบ Request พร้อม Pinning
    func fetchData(from url: URL, completion: @escaping (Data?, Error?) -> Void) {
        session.dataTask(with: url) { data, response, error in
            completion(data, error)
        }.resume()
    }
}

// Public Key Pinning แบบ Modern (iOS 14+)
class ModernCertificatePinning {
    
    // สร้าง URLSession พร้อม Public Key Pinning ผ่าน Configuration
    static func createPinnedSession(pinnedKeys: [SecKey]) -> URLSession {
        // iOS ไม่มี built-in public key pinning ใน URLSession โดยตรง
        // ต้องใช้ delegate หรือ library อย่าง TrustKit
        let config = URLSessionConfiguration.default
        let delegate = PublicKeyPinningDelegate(pinnedKeys: pinnedKeys)
        return URLSession(configuration: config, delegate: delegate, delegateQueue: nil)
    }
}

class PublicKeyPinningDelegate: NSObject, URLSessionDelegate {
    let pinnedKeys: [SecKey]
    
    init(pinnedKeys: [SecKey]) {
        self.pinnedKeys = pinnedKeys
    }
    
    func urlSession(
        _ session: URLSession,
        didReceive challenge: URLAuthenticationChallenge,
        completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void
    ) {
        guard
            challenge.protectionSpace.authenticationMethod == NSURLAuthenticationMethodServerTrust,
            let trust = challenge.protectionSpace.serverTrust,
            let certificates = SecTrustCopyCertificateChain(trust) as? [SecCertificate],
            let serverKey = certificates.first.flatMap({ SecCertificateCopyKey($0) })
        else {
            completionHandler(.cancelAuthenticationChallenge, nil)
            return
        }
        
        let keyMatches = pinnedKeys.contains { pinnedKey in
            guard
                let pinnedData = SecKeyCopyExternalRepresentation(pinnedKey, nil) as Data?,
                let serverData = SecKeyCopyExternalRepresentation(serverKey, nil) as Data?
            else { return false }
            return pinnedData == serverData
        }
        
        if keyMatches {
            completionHandler(.useCredential, URLCredential(trust: trust))
        } else {
            completionHandler(.cancelAuthenticationChallenge, nil)
        }
    }
}
```

---

## 40.9 HTTPS และ TLS

### การจัดการ TLS อย่างถูกต้อง

```swift
import Foundation
import Network

class TLSConfigManager {
    
    // ตรวจสอบ TLS Version
    static func createHighSecurityURLSession() -> URLSession {
        let config = URLSessionConfiguration.default
        
        // กำหนด TLS Version ขั้นต่ำ
        config.tlsMinimumSupportedProtocolVersion = .TLSv12
        
        // ห้ามใช้ TLS 1.0 และ 1.1 (deprecated และไม่ปลอดภัย)
        config.tlsMaximumSupportedProtocolVersion = .TLSv13
        
        return URLSession(configuration: config)
    }
    
    // สร้าง NWConnection พร้อม TLS Options
    static func createSecureConnection(host: String, port: UInt16) -> NWConnection {
        let tlsOptions = NWProtocolTLS.Options()
        
        // กำหนด minimum TLS version
        sec_protocol_options_set_min_tls_protocol_version(
            tlsOptions.securityProtocolOptions,
            .TLSv12
        )
        
        // ตรวจสอบ Certificate Validity
        sec_protocol_options_set_verify_block(
            tlsOptions.securityProtocolOptions,
            { _, trust, completion in
                let isValid = SecTrustEvaluateWithError(trust, nil)
                completion(isValid)
            },
            DispatchQueue.global()
        )
        
        let parameters = NWParameters(tls: tlsOptions, tcp: NWProtocolTCP.Options())
        return NWConnection(host: NWEndpoint.Host(host), port: NWEndpoint.Port(rawValue: port)!, using: parameters)
    }
    
    // Helper: ดึงข้อมูล TLS จาก URLResponse
    static func extractTLSInfo(from response: URLResponse?) -> String {
        guard let httpResponse = response as? HTTPURLResponse else {
            return "ไม่มีข้อมูล"
        }
        
        // Note: iOS ไม่ expose TLS info โดยตรงจาก URLResponse
        // ต้องใช้ Network framework หรือ CFNetwork เพื่อดูรายละเอียด
        return "Status: \(httpResponse.statusCode), URL: \(httpResponse.url?.absoluteString ?? "")"
    }
}
```

---

## 40.10 Data Encryption ด้วย CryptoKit

### CryptoKit สำหรับ Modern Cryptography

```swift
import CryptoKit
import Foundation

class CryptoKitManager {
    
    // ===============================
    // Symmetric Encryption (AES-GCM)
    // ===============================
    
    // Encrypt ข้อมูลด้วย AES-GCM
    static func encryptAES(_ plaintext: Data, key: SymmetricKey) throws -> Data {
        let sealedBox = try AES.GCM.seal(plaintext, using: key)
        
        // Combined = nonce + ciphertext + tag
        guard let combined = sealedBox.combined else {
            throw CryptoError.encryptionFailed
        }
        return combined
    }
    
    // Decrypt ข้อมูลที่ถูก Encrypt ด้วย AES-GCM
    static func decryptAES(_ ciphertext: Data, key: SymmetricKey) throws -> Data {
        let sealedBox = try AES.GCM.SealedBox(combined: ciphertext)
        return try AES.GCM.open(sealedBox, using: key)
    }
    
    // สร้าง Symmetric Key
    static func generateAESKey(bits: Int = 256) -> SymmetricKey {
        return SymmetricKey(size: SymmetricKeySize(bitCount: bits))
    }
    
    // เก็บ Key ใน Keychain
    static func saveKeyToKeychain(_ key: SymmetricKey, identifier: String) throws {
        let keyData = key.withUnsafeBytes { Data($0) }
        try KeychainManager.save(key: identifier, data: keyData)
    }
    
    // โหลด Key จาก Keychain
    static func loadKeyFromKeychain(identifier: String) throws -> SymmetricKey {
        let keyData = try KeychainManager.load(key: identifier)
        return SymmetricKey(data: keyData)
    }
    
    // ===============================
    // ChaCha20-Poly1305 Encryption
    // ===============================
    
    static func encryptChaCha(_ plaintext: Data, key: SymmetricKey) throws -> Data {
        let sealedBox = try ChaChaPoly.seal(plaintext, using: key)
        return sealedBox.combined
    }
    
    static func decryptChaCha(_ ciphertext: Data, key: SymmetricKey) throws -> Data {
        let sealedBox = try ChaChaPoly.SealedBox(combined: ciphertext)
        return try ChaChaPoly.open(sealedBox, using: key)
    }
    
    enum CryptoError: Error {
        case encryptionFailed
        case decryptionFailed
        case keyGenerationFailed
        case invalidData
    }
}

// ตัวอย่างการใช้งาน
func demonstrateAESEncryption() {
    let message = "ข้อมูลลับมาก"
    guard let messageData = message.data(using: .utf8) else { return }
    
    // สร้าง Key
    let key = CryptoKitManager.generateAESKey()
    
    do {
        // Encrypt
        let encrypted = try CryptoKitManager.encryptAES(messageData, key: key)
        print("Encrypted:", encrypted.base64EncodedString())
        
        // Decrypt
        let decrypted = try CryptoKitManager.decryptAES(encrypted, key: key)
        let decryptedMessage = String(data: decrypted, encoding: .utf8)
        print("Decrypted:", decryptedMessage ?? "Error")
        
    } catch {
        print("Error:", error)
    }
}
```

---

## 40.11 Asymmetric Encryption

### RSA และ Curve25519

```swift
import CryptoKit
import Security

class AsymmetricEncryption {
    
    // ============================
    // Curve25519 (Modern, Fast)
    // ============================
    
    // Key Exchange ด้วย Diffie-Hellman (ECDH)
    static func performKeyExchange() -> (sharedSecret: SharedSecret, publicKey: Curve25519.KeyAgreement.PublicKey) {
        // สร้าง Key Pair
        let privateKey = Curve25519.KeyAgreement.PrivateKey()
        let publicKey = privateKey.publicKey
        
        // จำลอง Server's Public Key
        let serverPrivateKey = Curve25519.KeyAgreement.PrivateKey()
        let serverPublicKey = serverPrivateKey.publicKey
        
        // คำนวณ Shared Secret
        let sharedSecret = try! privateKey.sharedSecretFromKeyAgreement(with: serverPublicKey)
        
        return (sharedSecret, publicKey)
    }
    
    // Signing ด้วย Curve25519
    static func signData(_ data: Data) throws -> (signature: Data, publicKey: Curve25519.Signing.PublicKey) {
        let privateKey = Curve25519.Signing.PrivateKey()
        let signature = try privateKey.signature(for: data)
        return (signature, privateKey.publicKey)
    }
    
    static func verifySignature(_ signature: Data, for data: Data, publicKey: Curve25519.Signing.PublicKey) -> Bool {
        return publicKey.isValidSignature(signature, for: data)
    }
    
    // ============================
    // P256 (NIST Standard)
    // ============================
    
    static func generateP256KeyPair() -> (private: P256.KeyAgreement.PrivateKey, public: P256.KeyAgreement.PublicKey) {
        let privateKey = P256.KeyAgreement.PrivateKey()
        return (privateKey, privateKey.publicKey)
    }
    
    static func performP256KeyExchange(
        myPrivate: P256.KeyAgreement.PrivateKey,
        theirPublic: P256.KeyAgreement.PublicKey
    ) throws -> SymmetricKey {
        let sharedSecret = try myPrivate.sharedSecretFromKeyAgreement(with: theirPublic)
        
        // Derive Symmetric Key จาก Shared Secret
        let symmetricKey = sharedSecret.hkdfDerivedSymmetricKey(
            using: SHA256.self,
            salt: "MyApp-Salt".data(using: .utf8)!,
            sharedInfo: Data(),
            outputByteCount: 32
        )
        
        return symmetricKey
    }
    
    // P256 Signing
    static func p256Sign(_ data: Data, with privateKey: P256.Signing.PrivateKey) throws -> P256.Signing.ECDSASignature {
        return try privateKey.signature(for: data)
    }
    
    static func p256Verify(_ signature: P256.Signing.ECDSASignature, for data: Data, with publicKey: P256.Signing.PublicKey) -> Bool {
        return publicKey.isValidSignature(signature, for: data)
    }
}

// End-to-End Encryption Example
class E2EEncryption {
    private let myPrivateKey: Curve25519.KeyAgreement.PrivateKey
    let myPublicKey: Curve25519.KeyAgreement.PublicKey
    
    init() {
        myPrivateKey = Curve25519.KeyAgreement.PrivateKey()
        myPublicKey = myPrivateKey.publicKey
    }
    
    // Encrypt message สำหรับผู้รับ
    func encrypt(message: String, recipientPublicKey: Curve25519.KeyAgreement.PublicKey) throws -> Data {
        guard let messageData = message.data(using: .utf8) else {
            throw NSError(domain: "E2E", code: -1)
        }
        
        // ECDH Key Exchange
        let sharedSecret = try myPrivateKey.sharedSecretFromKeyAgreement(with: recipientPublicKey)
        
        // Derive Symmetric Key
        let symmetricKey = sharedSecret.hkdfDerivedSymmetricKey(
            using: SHA256.self,
            salt: Data(),
            sharedInfo: "E2E-Message".data(using: .utf8)!,
            outputByteCount: 32
        )
        
        // Encrypt ด้วย AES-GCM
        let sealedBox = try AES.GCM.seal(messageData, using: symmetricKey)
        return sealedBox.combined!
    }
    
    // Decrypt message
    func decrypt(ciphertext: Data, senderPublicKey: Curve25519.KeyAgreement.PublicKey) throws -> String {
        let sharedSecret = try myPrivateKey.sharedSecretFromKeyAgreement(with: senderPublicKey)
        
        let symmetricKey = sharedSecret.hkdfDerivedSymmetricKey(
            using: SHA256.self,
            salt: Data(),
            sharedInfo: "E2E-Message".data(using: .utf8)!,
            outputByteCount: 32
        )
        
        let sealedBox = try AES.GCM.SealedBox(combined: ciphertext)
        let decryptedData = try AES.GCM.open(sealedBox, using: symmetricKey)
        
        guard let message = String(data: decryptedData, encoding: .utf8) else {
            throw NSError(domain: "E2E", code: -2)
        }
        
        return message
    }
}
```

---

## 40.12 Hashing

### SHA256, SHA512 และ HMAC

```swift
import CryptoKit
import Foundation

class HashingManager {
    
    // ===============
    // SHA Hashing
    // ===============
    
    // SHA256 Hash
    static func sha256(_ data: Data) -> String {
        let hash = SHA256.hash(data: data)
        return hash.compactMap { String(format: "%02x", $0) }.joined()
    }
    
    // SHA512 Hash
    static func sha512(_ data: Data) -> String {
        let hash = SHA512.hash(data: data)
        return hash.compactMap { String(format: "%02x", $0) }.joined()
    }
    
    // Hash String
    static func hashString(_ string: String, algorithm: HashAlgorithm = .sha256) -> String {
        guard let data = string.data(using: .utf8) else { return "" }
        
        switch algorithm {
        case .sha256:
            return sha256(data)
        case .sha384:
            let hash = SHA384.hash(data: data)
            return hash.compactMap { String(format: "%02x", $0) }.joined()
        case .sha512:
            return sha512(data)
        }
    }
    
    enum HashAlgorithm {
        case sha256, sha384, sha512
    }
    
    // Hash File
    static func hashFile(at url: URL) throws -> String {
        var hasher = SHA256()
        
        let fileHandle = try FileHandle(forReadingFrom: url)
        defer { fileHandle.closeFile() }
        
        // อ่านทีละ chunk เพื่อประหยัด Memory
        let chunkSize = 1024 * 1024  // 1 MB chunks
        while true {
            let chunk = fileHandle.readData(ofLength: chunkSize)
            guard !chunk.isEmpty else { break }
            hasher.update(data: chunk)
        }
        
        let digest = hasher.finalize()
        return digest.compactMap { String(format: "%02x", $0) }.joined()
    }
    
    // ตรวจสอบ Integrity
    static func verifyIntegrity(data: Data, expectedHash: String) -> Bool {
        let actualHash = sha256(data)
        return actualHash == expectedHash
    }
    
    // ===============
    // HMAC
    // ===============
    
    // HMAC-SHA256
    static func hmac256(_ data: Data, key: SymmetricKey) -> Data {
        let mac = HMAC<SHA256>.authenticationCode(for: data, using: key)
        return Data(mac)
    }
    
    // HMAC-SHA512
    static func hmac512(_ data: Data, key: SymmetricKey) -> Data {
        let mac = HMAC<SHA512>.authenticationCode(for: data, using: key)
        return Data(mac)
    }
    
    // ตรวจสอบ HMAC
    static func verifyHMAC(_ data: Data, mac: Data, key: SymmetricKey) -> Bool {
        let expectedMAC = HMAC<SHA256>.authenticationCode(for: data, using: key)
        return HMAC<SHA256>.isValidAuthenticationCode(mac, authenticating: data, using: key)
    }
    
    // Password Hashing (ใช้ BCrypt หรือ Argon2 จะดีกว่า แต่ไม่มีใน Standard Library)
    // สำหรับ iOS ควรใช้ Server-side password hashing
    // หรือ CommonCrypto PBKDF2
    static func pbkdf2SHA1(password: String, salt: Data, iterations: Int, keyLength: Int) -> Data? {
        guard let passwordData = password.data(using: .utf8) else { return nil }
        
        var derivedKey = Data(count: keyLength)
        
        let result = derivedKey.withUnsafeMutableBytes { derivedKeyBytes in
            passwordData.withUnsafeBytes { passwordBytes in
                salt.withUnsafeBytes { saltBytes in
                    CCKeyDerivationPBKDF(
                        CCPBKDFAlgorithm(kCCPBKDF2),
                        passwordBytes.bindMemory(to: Int8.self).baseAddress,
                        passwordData.count,
                        saltBytes.bindMemory(to: UInt8.self).baseAddress,
                        salt.count,
                        CCPseudoRandomAlgorithm(kCCPRFHmacAlgSHA1),
                        UInt32(iterations),
                        derivedKeyBytes.bindMemory(to: UInt8.self).baseAddress,
                        keyLength
                    )
                }
            }
        }
        
        return result == kCCSuccess ? derivedKey : nil
    }
}

import CommonCrypto

// การใช้งาน Hashing
func demonstrateHashing() {
    let message = "สวัสดี Swift Security"
    let hash256 = HashingManager.hashString(message, algorithm: .sha256)
    let hash512 = HashingManager.hashString(message, algorithm: .sha512)
    
    print("SHA256:", hash256)
    print("SHA512:", hash512)
    
    // HMAC
    let key = CryptoKitManager.generateAESKey()
    guard let messageData = message.data(using: .utf8) else { return }
    
    let mac = HashingManager.hmac256(messageData, key: key)
    let isValid = HashingManager.verifyHMAC(messageData, mac: mac, key: key)
    
    print("HMAC Valid:", isValid)
}
```

---

## 40.13 Secure Random Generation

### การสร้างข้อมูล Random ที่ปลอดภัย

```swift
import CryptoKit
import Security
import Foundation

class SecureRandomGenerator {
    
    // สร้าง Random Bytes
    static func randomBytes(count: Int) -> Data {
        var bytes = [UInt8](repeating: 0, count: count)
        let status = SecRandomCopyBytes(kSecRandomDefault, count, &bytes)
        
        guard status == errSecSuccess else {
            // Fallback - แต่ไม่ควรเกิดขึ้น
            var data = Data(count: count)
            data.withUnsafeMutableBytes { ptr in
                arc4random_buf(ptr.baseAddress, count)
            }
            return data
        }
        
        return Data(bytes)
    }
    
    // สร้าง Random Int ในช่วงที่กำหนด (Secure)
    static func randomInt(in range: Range<Int>) -> Int {
        let rangeSize = range.upperBound - range.lowerBound
        let bytesNeeded = (Int.bitWidth - rangeSize.leadingZeroBitCount + 7) / 8
        
        while true {
            let randomData = randomBytes(count: bytesNeeded)
            var randomValue = 0
            for byte in randomData {
                randomValue = (randomValue << 8) | Int(byte)
            }
            
            // Rejection Sampling เพื่อหลีกเลี่ยง Modulo Bias
            let maxUnbiased = Int.max - (Int.max % rangeSize)
            if randomValue <= maxUnbiased {
                return range.lowerBound + (randomValue % rangeSize)
            }
        }
    }
    
    // สร้าง Secure Token (สำหรับ CSRF, API Key, etc.)
    static func generateToken(byteLength: Int = 32) -> String {
        let randomData = randomBytes(count: byteLength)
        return randomData.base64EncodedString()
    }
    
    // สร้าง URL-Safe Token
    static func generateURLSafeToken(byteLength: Int = 32) -> String {
        let randomData = randomBytes(count: byteLength)
        return randomData.base64EncodedString()
            .replacingOccurrences(of: "+", with: "-")
            .replacingOccurrences(of: "/", with: "_")
            .replacingOccurrences(of: "=", with: "")
    }
    
    // สร้าง Salt สำหรับ Password Hashing
    static func generateSalt(byteLength: Int = 32) -> Data {
        return randomBytes(count: byteLength)
    }
    
    // สร้าง Nonce (Number used ONCE)
    static func generateNonce(byteLength: Int = 16) -> Data {
        return randomBytes(count: byteLength)
    }
    
    // สร้าง Secure UUID
    static func generateSecureUUID() -> String {
        let bytes = randomBytes(count: 16)
        var uuidBytes = [UInt8](bytes)
        
        // Set version 4 bits
        uuidBytes[6] = (uuidBytes[6] & 0x0F) | 0x40
        // Set variant bits
        uuidBytes[8] = (uuidBytes[8] & 0x3F) | 0x80
        
        let uuid = NSUUID(uuidBytes: uuidBytes)
        return uuid.uuidString
    }
    
    // CryptoKit Random (Swift native)
    static func generateSymmetricKey(bits: Int = 256) -> SymmetricKey {
        return SymmetricKey(size: SymmetricKeySize(bitCount: bits))
    }
}

// Password Generator
class SecurePasswordGenerator {
    
    static let uppercaseLetters = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
    static let lowercaseLetters = "abcdefghijklmnopqrstuvwxyz"
    static let digits = "0123456789"
    static let specialChars = "!@#$%^&*()_+-=[]{}|;':\",./<>?"
    
    static func generate(
        length: Int = 16,
        includeUppercase: Bool = true,
        includeLowercase: Bool = true,
        includeDigits: Bool = true,
        includeSpecial: Bool = true
    ) -> String {
        var charset = ""
        var requiredChars = [Character]()
        
        if includeUppercase {
            charset += uppercaseLetters
            requiredChars.append(Character(String(uppercaseLetters.randomElement()!)))
        }
        if includeLowercase {
            charset += lowercaseLetters
            requiredChars.append(Character(String(lowercaseLetters.randomElement()!)))
        }
        if includeDigits {
            charset += digits
            requiredChars.append(Character(String(digits.randomElement()!)))
        }
        if includeSpecial {
            charset += specialChars
            requiredChars.append(Character(String(specialChars.randomElement()!)))
        }
        
        guard !charset.isEmpty else { return "" }
        
        let charsetArray = Array(charset)
        var password = requiredChars
        
        for _ in requiredChars.count..<length {
            let index = SecureRandomGenerator.randomInt(in: 0..<charsetArray.count)
            password.append(charsetArray[index])
        }
        
        // Shuffle เพื่อ Randomize position ของ required chars
        for i in stride(from: password.count - 1, through: 1, by: -1) {
            let j = SecureRandomGenerator.randomInt(in: 0...i)
            password.swapAt(i, j)
        }
        
        return String(password)
    }
}
```

---

## 40.14 Jailbreak Detection

### การตรวจสอบ Jailbreak

```swift
import Foundation
import UIKit

class JailbreakDetector {
    
    // ตรวจสอบสัญญาณของ Jailbreak
    static var isJailbroken: Bool {
        return checkSuspiciousFiles() ||
               checkWritePermissions() ||
               checkCydiaBundleID() ||
               checkDynamicLibraries() ||
               checkSystemIntegrity()
    }
    
    // ตรวจสอบไฟล์ที่น่าสงสัย
    private static func checkSuspiciousFiles() -> Bool {
        let suspiciousPaths = [
            "/Applications/Cydia.app",
            "/Applications/FakeCarrier.app",
            "/Applications/Icy.app",
            "/Applications/IntelliScreen.app",
            "/Applications/MxTube.app",
            "/Applications/RockApp.app",
            "/Applications/SBSettings.app",
            "/Applications/WinterBoard.app",
            "/Library/MobileSubstrate/DynamicLibraries/LiveClock.plist",
            "/Library/MobileSubstrate/MobileSubstrate.dylib",
            "/private/var/lib/apt",
            "/private/var/lib/cydia",
            "/private/var/mobile/Library/SBSettings/Themes",
            "/private/var/stash",
            "/private/var/tmp/cydia.log",
            "/System/Library/LaunchDaemons/com.ikey.bbot.plist",
            "/System/Library/LaunchDaemons/com.saurik.Cydia.Startup.plist",
            "/usr/bin/sshd",
            "/usr/libexec/sftp-server",
            "/usr/sbin/sshd",
            "/bin/bash",
            "/bin/sh"
        ]
        
        for path in suspiciousPaths {
            if FileManager.default.fileExists(atPath: path) {
                return true
            }
        }
        
        return false
    }
    
    // ตรวจสอบ Write Permission นอก sandbox
    private static func checkWritePermissions() -> Bool {
        let testPath = "/private/jailbreak_test_\(Int.random(in: 0..<10000))"
        
        do {
            try "test".write(toFile: testPath, atomically: true, encoding: .utf8)
            try FileManager.default.removeItem(atPath: testPath)
            return true  // สามารถ write นอก sandbox = Jailbroken
        } catch {
            return false  // ไม่สามารถ write = ปกติ
        }
    }
    
    // ตรวจสอบ Cydia Bundle ID
    private static func checkCydiaBundleID() -> Bool {
        return UIApplication.shared.canOpenURL(URL(string: "cydia://package/com.example.package")!)
    }
    
    // ตรวจสอบ Dynamic Libraries ที่ถูก inject
    private static func checkDynamicLibraries() -> Bool {
        let suspiciousLibs = [
            "MobileSubstrate",
            "libcycript",
            "cynject",
            "SubstrateBootstrap"
        ]
        
        // ตรวจสอบผ่าน process images
        for index in 0..<_dyld_image_count() {
            if let imageName = _dyld_get_image_name(index) {
                let name = String(cString: imageName)
                if suspiciousLibs.contains(where: { name.contains($0) }) {
                    return true
                }
            }
        }
        
        return false
    }
    
    // ตรวจสอบ System Integrity
    private static func checkSystemIntegrity() -> Bool {
        // ตรวจสอบว่า fork() ทำงานได้หรือไม่ (ไม่ควรได้ใน Sandbox)
        let pid = fork()
        if pid >= 0 {
            if pid > 0 {
                // Parent process - kill child
                kill(pid, SIGTERM)
            }
            return true  // fork() ทำงานได้ = Jailbroken
        }
        return false
    }
    
    // การจัดการเมื่อตรวจพบ Jailbreak
    static func handleJailbreakDetection() {
        if isJailbroken {
            print("⚠️ Jailbreak detected!")
            
            // Option 1: แสดง Alert และปิดแอป
            // Option 2: จำกัดฟีเจอร์
            // Option 3: แค่ Log และ Monitor
            
            // ⚠️ อย่า terminate แอปโดยทันทีในทุกกรณี
            // เพราะอาจทำให้ False Positive ส่งผลต่อ Normal Users
        }
    }
}

// Note: Jailbreak Detection ไม่ 100% reliable
// Sophisticated jailbreaks สามารถ bypass ได้
// ควรใช้เป็นส่วนหนึ่งของ Defense in Depth
```

---

## 40.15 Obfuscation Considerations

### การพิจารณาเรื่อง Code Obfuscation

```swift
// Swift ไม่มี Built-in Obfuscation
// แต่มีเทคนิคที่ช่วยได้บ้าง

// 1. String Obfuscation
class StringObfuscator {
    
    // Simple XOR Obfuscation (ไม่ใช่ Encryption จริงๆ แต่ซ่อนจาก String Analysis)
    static func obfuscate(_ string: String, key: UInt8) -> [UInt8] {
        return string.utf8.map { $0 ^ key }
    }
    
    static func deobfuscate(_ bytes: [UInt8], key: UInt8) -> String? {
        let deobfuscated = bytes.map { $0 ^ key }
        return String(bytes: deobfuscated, encoding: .utf8)
    }
    
    // ตัวอย่าง: ซ่อน API Key จาก Static Analysis
    private static let obfuscatedAPIKey: [UInt8] = [0x4A, 0x5A, 0x4B, 0x1A]  // Obfuscated
    private static let obfuscationKey: UInt8 = 0x42
    
    static var apiKey: String {
        return deobfuscate(obfuscatedAPIKey, key: obfuscationKey) ?? ""
    }
}

// 2. Anti-Debugging (ตรวจสอบว่า Debugger แนบอยู่หรือไม่)
class AntiDebugging {
    
    static var isDebugging: Bool {
        var info = kinfo_proc()
        var mib = [CTL_KERN, KERN_PROC, KERN_PROC_PID, getpid()]
        var size = MemoryLayout<kinfo_proc>.stride
        let result = sysctl(&mib, UInt32(mib.count), &info, &size, nil, 0)
        
        if result != 0 { return false }
        return (info.kp_proc.p_flag & P_TRACED) != 0
    }
    
    static func detectDebugger() {
        #if !DEBUG
        if isDebugging {
            print("⚠️ Debugger detected!")
            // จัดการตามที่เหมาะสม
        }
        #endif
    }
}

// 3. Binary Protection Recommendations
// - Enable Bitcode
// - Strip Debug Symbols (Release builds)
// - ใช้ ProGuard equivalent (ไม่มีใน iOS โดยตรง)
// - ใช้ Commercial Obfuscation Tools เช่น iXGuard, Dotfuscator

// หมายเหตุสำคัญ:
// Obfuscation คือ Security through Obscurity
// ไม่ใช่ Security จริงๆ
// ควรใช้ควบคู่กับ Real Security Measures
```

---

## 40.16 Secure Coding Guidelines

### หลักปฏิบัติการเขียนโค้ดที่ปลอดภัย

```swift
import Foundation

// 1. Input Validation
class InputValidator {
    
    // Validate Email
    static func isValidEmail(_ email: String) -> Bool {
        let emailRegex = #"^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$"#
        let predicate = NSPredicate(format: "SELF MATCHES %@", emailRegex)
        return predicate.evaluate(with: email)
    }
    
    // Validate ด้วย CharacterSet
    static func containsOnlyAlphanumeric(_ string: String) -> Bool {
        let allowedChars = CharacterSet.alphanumerics
        return string.unicodeScalars.allSatisfy { allowedChars.contains($0) }
    }
    
    // ป้องกัน SQL Injection (ควรใช้ Parameterized Queries)
    static func sanitizeForSQL(_ input: String) -> String {
        // Escape single quotes
        return input.replacingOccurrences(of: "'", with: "''")
    }
    
    // Validate URL
    static func isValidURL(_ string: String) -> Bool {
        guard let url = URL(string: string) else { return false }
        return url.scheme != nil && url.host != nil
    }
    
    // Length Validation
    static func validateLength(_ string: String, min: Int, max: Int) -> Bool {
        return string.count >= min && string.count <= max
    }
}

// 2. Secure Comparison (Constant Time)
class SecureCompare {
    
    // Constant-time comparison ป้องกัน Timing Attack
    static func constantTimeCompare(_ a: Data, _ b: Data) -> Bool {
        guard a.count == b.count else { return false }
        
        var result: UInt8 = 0
        for (byte1, byte2) in zip(a, b) {
            result |= byte1 ^ byte2
        }
        
        return result == 0
    }
    
    static func constantTimeCompareStrings(_ a: String, _ b: String) -> Bool {
        guard let dataA = a.data(using: .utf8),
              let dataB = b.data(using: .utf8) else { return false }
        return constantTimeCompare(dataA, dataB)
    }
}

// 3. Memory Zeroing สำหรับ Sensitive Data
class SecureMemory {
    
    static func zeroize(_ data: inout Data) {
        data.withUnsafeMutableBytes { ptr in
            memset(ptr.baseAddress, 0, ptr.count)
        }
        data = Data()
    }
    
    // SecureString: String ที่ถูก Zero-out หลังใช้งาน
    struct SecureString {
        private var storage: [UInt8]
        
        init(_ string: String) {
            storage = Array(string.utf8)
        }
        
        var value: String {
            return String(bytes: storage, encoding: .utf8) ?? ""
        }
        
        mutating func clear() {
            for i in 0..<storage.count {
                storage[i] = 0
            }
            storage.removeAll()
        }
    }
}

// 4. Error Handling ที่ไม่รั่ว Information
class SecureErrorHandler {
    
    enum AppError: Error {
        case authenticationFailed
        case dataNotFound
        case networkError
        case serverError
        
        // ข้อความ Error ที่ปลอดภัย (ไม่เปิดเผยรายละเอียดภายใน)
        var userMessage: String {
            switch self {
            case .authenticationFailed:
                return "ไม่สามารถยืนยันตัวตนได้"  // ไม่บอกว่า username หรือ password ผิด
            case .dataNotFound:
                return "ไม่พบข้อมูลที่ต้องการ"
            case .networkError:
                return "เกิดข้อผิดพลาดในการเชื่อมต่อ"
            case .serverError:
                return "เกิดข้อผิดพลาด กรุณาลองใหม่"
            }
        }
        
        // Log message สำหรับ Developer (ไม่แสดงผู้ใช้)
        var debugMessage: String {
            switch self {
            case .authenticationFailed:
                return "Authentication failed: Invalid credentials"
            case .dataNotFound:
                return "Data not found in storage"
            case .networkError:
                return "Network connection failed"
            case .serverError:
                return "Server returned error status"
            }
        }
    }
    
    static func handle(_ error: AppError) {
        // Log สำหรับ Developer
        #if DEBUG
        print("[DEBUG] Error: \(error.debugMessage)")
        #else
        // Production: ส่ง Log ไปยัง Crash Reporting Service
        // Crashlytics.crashlytics().record(error: error)
        #endif
    }
}
```

---

## 40.17 OWASP Mobile Top 10

### การป้องกัน OWASP Mobile Top 10

```swift
// OWASP Mobile Top 10 (2023)

class OWASPDefenses {
    
    // M1: Improper Platform Usage
    func properPlatformUsage() {
        // ✅ ใช้ Permission อย่างถูกต้อง
        // ✅ ไม่ขอ Permission ที่ไม่จำเป็น
        // ✅ ใช้ Secure APIs ที่ Platform จัดเตรียมให้
        
        // ❌ อย่าใช้ private APIs
        // ❌ อย่า hardcode sensitive data
    }
    
    // M2: Insecure Data Storage
    func secureDataStorage() {
        // ✅ เก็บ Sensitive Data ใน Keychain
        // ✅ เข้ารหัสก่อนเก็บใน Database
        // ✅ ใช้ Data Protection Attributes
        
        let _ = "apiToken"
        do {
            try KeychainManager.save(key: "apiToken", string: "secret_token_value")
        } catch {
            print("Failed to save: \(error)")
        }
    }
    
    // M3: Insecure Communication
    func secureCommunication() {
        // ✅ บังคับ HTTPS ด้วย ATS
        // ✅ Certificate Pinning สำหรับข้อมูลสำคัญ
        // ✅ ตรวจสอบ Certificate Validity
        
        let session = CertificatePinningManager()
        _ = session
    }
    
    // M4: Insecure Authentication
    func secureAuthentication() {
        // ✅ ใช้ Strong Authentication
        // ✅ Biometric + PIN Fallback
        // ✅ Session Timeout
        // ✅ Re-authenticate สำหรับ Critical Operations
        
        let authManager = BiometricAuthManager()
        _ = authManager
    }
    
    // M5: Insufficient Cryptography
    func sufficientCryptography() {
        // ✅ ใช้ AES-256 หรือ ChaCha20
        // ✅ ใช้ Secure Random สำหรับ Keys
        // ✅ ใช้ Proper IV/Nonce
        
        // ❌ อย่าใช้ MD5 หรือ SHA1 สำหรับ Security
        // ❌ อย่า hardcode encryption keys
        // ❌ อย่าใช้ ECB mode
        
        let key = CryptoKitManager.generateAESKey()
        _ = key
    }
    
    // M6: Insecure Authorization
    func secureAuthorization() {
        // ✅ ตรวจสอบ Authorization ทั้ง Client และ Server
        // ✅ Principle of Least Privilege
        // ✅ ตรวจสอบ Token ทุกครั้ง
        
        // ❌ อย่าเชื่อ Client-side Authorization เพียงอย่างเดียว
    }
    
    // M7: Poor Code Quality
    func goodCodeQuality() {
        // ✅ Input Validation
        // ✅ Avoid Buffer Overflow (Swift ป้องกันได้มาก)
        // ✅ Handle Errors อย่างเหมาะสม
        // ✅ ใช้ Static Analysis Tools
        
        let email = "user@example.com"
        let isValid = InputValidator.isValidEmail(email)
        _ = isValid
    }
    
    // M8: Code Tampering
    func preventCodeTampering() {
        // ✅ Code Signing
        // ✅ ตรวจสอบ App Integrity
        // ✅ Anti-Jailbreak Detection
        
        if JailbreakDetector.isJailbroken {
            print("⚠️ Tampered environment detected")
        }
    }
    
    // M9: Reverse Engineering
    func preventReverseEngineering() {
        // ✅ Strip Debug Symbols ใน Release builds
        // ✅ Code Obfuscation (ถ้าจำเป็น)
        // ✅ ไม่ hardcode Secrets ใน Binary
        
        AntiDebugging.detectDebugger()
    }
    
    // M10: Extraneous Functionality
    func removeExtraneousFunctionality() {
        // ✅ ลบ Debug Code ออกจาก Production
        // ✅ ไม่มี Backdoors
        // ✅ ปิด Test Endpoints
        
        #if DEBUG
        // Debug-only code
        print("Debug mode enabled")
        #endif
    }
}
```

---

## 40.18 App Sandbox

### ความเข้าใจ iOS App Sandbox

```swift
import Foundation

class SandboxExplorer {
    
    // โครงสร้าง App Sandbox
    static func exploreDirectories() {
        let fileManager = FileManager.default
        
        // Documents: ข้อมูลที่ User สร้าง (iCloud Backup ได้)
        let documents = fileManager.urls(for: .documentDirectory, in: .userDomainMask).first!
        print("Documents:", documents.path)
        
        // Library/Caches: ข้อมูล Cache (OS อาจลบได้)
        let caches = fileManager.urls(for: .cachesDirectory, in: .userDomainMask).first!
        print("Caches:", caches.path)
        
        // Library/Application Support: App Data (iCloud Backup ได้)
        let appSupport = fileManager.urls(for: .applicationSupportDirectory, in: .userDomainMask).first!
        print("App Support:", appSupport.path)
        
        // tmp: Temporary Files (OS อาจลบได้)
        let tmp = fileManager.temporaryDirectory
        print("Temp:", tmp.path)
    }
    
    // Data Protection Levels
    static func demonstrateDataProtection() {
        let fileURL = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first!
            .appendingPathComponent("sensitive.dat")
        
        let data = "Sensitive Data".data(using: .utf8)!
        
        // เขียนไฟล์พร้อม Protection Level
        try? data.write(to: fileURL, options: [
            .completeFileProtection  // ต้อง Unlock device เพื่อเข้าถึง
        ])
        
        // Protection Levels:
        // .completeFileProtection - Protected เมื่อ device lock
        // .completeFileProtectionUnlessOpen - Protected เว้นแต่ file กำลัง open
        // .completeFileProtectionUntilFirstUserAuthentication - Protected จนกว่า user จะ unlock ครั้งแรก
        // .noFileProtection - ไม่มี Protection เพิ่มเติม
    }
    
    // IPC (Inter-Process Communication)
    static func secureIPCGuidelines() {
        // URL Schemes: ควรตรวจสอบ Origin
        // App Groups: ใช้ Keychain Access Groups สำหรับ Sensitive Data
        // Pasteboard: ระวัง Sensitive Data ใน Clipboard
        // Custom URL Scheme Validation
        
        print("IPC Guidelines:")
        print("1. ตรวจสอบ Source ของ URL Scheme calls")
        print("2. Validate ทุก Parameter")
        print("3. ใช้ Universal Links แทน Custom URL Schemes")
        print("4. ล้าง Pasteboard หลังใช้")
    }
}

// App Groups สำหรับ Share Data ระหว่าง App Extensions
class AppGroupManager {
    private static let groupIdentifier = "group.com.example.myapp"
    
    static var sharedUserDefaults: UserDefaults? {
        return UserDefaults(suiteName: groupIdentifier)
    }
    
    static var sharedContainerURL: URL? {
        return FileManager.default.containerURL(forSecurityApplicationGroupIdentifier: groupIdentifier)
    }
    
    // เก็บ Non-sensitive data ใน App Group
    static func saveSharedData(key: String, value: Any) {
        sharedUserDefaults?.set(value, forKey: key)
    }
    
    // สำหรับ Sensitive Data ใช้ Keychain Access Groups
    static func saveSharedSecret(key: String, value: String) throws {
        guard let secretData = value.data(using: .utf8) else { return }
        
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: "com.example.shared",
            kSecAttrAccount as String: key,
            kSecValueData as String: secretData,
            kSecAttrAccessGroup as String: "TEAMID.com.example.shared"  // Shared Keychain Group
        ]
        
        SecItemDelete(query as CFDictionary)
        let status = SecItemAdd(query as CFDictionary, nil)
        
        if status != errSecSuccess {
            throw NSError(domain: "KeychainError", code: Int(status))
        }
    }
}
```

---

## 40.19 Privacy Manifest

### Privacy Manifest ใน iOS 17+

```xml
<!-- PrivacyInfo.xcprivacy -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <!-- API ที่ใช้และเหตุผล -->
    <key>NSPrivacyAccessedAPITypes</key>
    <array>
        <dict>
            <key>NSPrivacyAccessedAPIType</key>
            <string>NSPrivacyAccessedAPICategoryUserDefaults</string>
            <key>NSPrivacyAccessedAPITypeReasons</key>
            <array>
                <string>CA92.1</string> <!-- เก็บ settings สำหรับ user -->
            </array>
        </dict>
        <dict>
            <key>NSPrivacyAccessedAPIType</key>
            <string>NSPrivacyAccessedAPICategoryFileTimestamp</string>
            <key>NSPrivacyAccessedAPITypeReasons</key>
            <array>
                <string>C617.1</string> <!-- แสดง timestamp ของไฟล์ให้ user -->
            </array>
        </dict>
    </array>
    
    <!-- Data ที่ Collect -->
    <key>NSPrivacyCollectedDataTypes</key>
    <array>
        <dict>
            <key>NSPrivacyCollectedDataType</key>
            <string>NSPrivacyCollectedDataTypeEmailAddress</string>
            <key>NSPrivacyCollectedDataTypeLinked</key>
            <true/>
            <key>NSPrivacyCollectedDataTypeTracking</key>
            <false/>
            <key>NSPrivacyCollectedDataTypePurposes</key>
            <array>
                <string>NSPrivacyCollectedDataTypePurposeAppFunctionality</string>
            </array>
        </dict>
    </array>
    
    <!-- Tracking -->
    <key>NSPrivacyTracking</key>
    <false/>
    
    <!-- Tracking Domains -->
    <key>NSPrivacyTrackingDomains</key>
    <array/>
</dict>
</plist>
```

```swift
// Privacy Best Practices
class PrivacyManager {
    
    // Data Minimization: เก็บแค่ที่จำเป็น
    struct UserData {
        let id: String  // Opaque identifier
        // ❌ let email: String  // ถ้าไม่จำเป็น
        // ❌ let phoneNumber: String  // ถ้าไม่จำเป็น
        let preferences: [String: Bool]
        // ❌ let location: CLLocation  // ถ้าไม่จำเป็น
    }
    
    // Data Retention: ลบเมื่อไม่ต้องการ
    static func deleteUserData(userId: String) {
        // ลบจาก Keychain
        try? KeychainManager.delete(key: "user_\(userId)")
        
        // ลบจาก Database
        // CoreDataManager.shared.deleteUser(id: userId)
        
        // ลบ Local Files
        cleanupUserFiles(userId: userId)
        
        print("User data deleted for ID: \(userId)")
    }
    
    private static func cleanupUserFiles(userId: String) {
        let documentsDir = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first!
        let userDir = documentsDir.appendingPathComponent("user_\(userId)")
        try? FileManager.default.removeItem(at: userDir)
    }
    
    // Anonymization
    static func anonymize(_ data: [String: Any]) -> [String: Any] {
        var anonymized = data
        
        // ลบหรือ hash PII fields
        if let email = anonymized["email"] as? String {
            anonymized["email_hash"] = HashingManager.hashString(email)
            anonymized.removeValue(forKey: "email")
        }
        
        if let name = anonymized["name"] as? String {
            anonymized["name"] = "\(name.prefix(1))***"
        }
        
        return anonymized
    }
    
    // Consent Management
    class ConsentManager {
        private static let consentKey = "user_privacy_consent"
        
        struct ConsentRecord: Codable {
            let version: String
            let timestamp: Date
            let analytics: Bool
            let marketing: Bool
            let thirdParty: Bool
        }
        
        static func recordConsent(analytics: Bool, marketing: Bool, thirdParty: Bool) {
            let record = ConsentRecord(
                version: "1.0",
                timestamp: Date(),
                analytics: analytics,
                marketing: marketing,
                thirdParty: thirdParty
            )
            
            do {
                try KeychainManager.save(key: consentKey, value: record)
            } catch {
                UserDefaults.standard.set(try? JSONEncoder().encode(record), forKey: consentKey)
            }
        }
        
        static func getConsent() -> ConsentRecord? {
            if let record = try? KeychainManager.load(key: consentKey, type: ConsentRecord.self) {
                return record
            }
            
            guard let data = UserDefaults.standard.data(forKey: consentKey),
                  let record = try? JSONDecoder().decode(ConsentRecord.self, from: data) else {
                return nil
            }
            
            return record
        }
        
        static func hasValidConsent() -> Bool {
            guard let consent = getConsent() else { return false }
            // ตรวจสอบว่า Consent ไม่เก่าเกินไป (เช่น 1 ปี)
            let oneYear: TimeInterval = 365 * 24 * 3600
            return Date().timeIntervalSince(consent.timestamp) < oneYear
        }
    }
}
```

---

## 40.20 Practical Exercises

### แบบฝึกหัดที่ 1: Secure User Preferences

```swift
// โจทย์: สร้าง Secure Preferences Manager ที่เก็บข้อมูล Sensitive อย่างปลอดภัย

class SecurePreferencesManager {
    
    enum PreferenceKey: String {
        case authToken = "auth_token"
        case userId = "user_id"
        case lastLoginDate = "last_login"
        case notificationsEnabled = "notifications"  // Non-sensitive
        case theme = "app_theme"  // Non-sensitive
    }
    
    private static let sensitiveKeys: Set<PreferenceKey> = [.authToken]
    
    // เก็บค่าโดยอัตโนมัติเลือก Storage ที่เหมาะสม
    static func set(_ value: String, for key: PreferenceKey) {
        if sensitiveKeys.contains(key) {
            // Sensitive: เก็บใน Keychain
            try? KeychainManager.save(key: key.rawValue, string: value)
        } else {
            // Non-sensitive: เก็บใน UserDefaults
            UserDefaults.standard.set(value, forKey: key.rawValue)
        }
    }
    
    static func get(_ key: PreferenceKey) -> String? {
        if sensitiveKeys.contains(key) {
            return try? KeychainManager.loadString(key: key.rawValue)
        } else {
            return UserDefaults.standard.string(forKey: key.rawValue)
        }
    }
    
    static func remove(_ key: PreferenceKey) {
        if sensitiveKeys.contains(key) {
            try? KeychainManager.delete(key: key.rawValue)
        } else {
            UserDefaults.standard.removeObject(forKey: key.rawValue)
        }
    }
    
    static func clearAll() {
        for key in [PreferenceKey.authToken] {
            remove(key)
        }
    }
}

// การใช้งาน
func exercise1() {
    // เก็บ Token (ไปที่ Keychain อัตโนมัติ)
    SecurePreferencesManager.set("Bearer abc123", for: .authToken)
    
    // เก็บ Theme (ไปที่ UserDefaults อัตโนมัติ)
    SecurePreferencesManager.set("dark", for: .theme)
    
    // อ่าน
    let token = SecurePreferencesManager.get(.authToken)
    let theme = SecurePreferencesManager.get(.theme)
    
    print("Token from Keychain:", token ?? "nil")
    print("Theme from UserDefaults:", theme ?? "nil")
    
    // Logout
    SecurePreferencesManager.clearAll()
}
```

### แบบฝึกหัดที่ 2: Encrypted Notes App

```swift
// โจทย์: สร้าง Note App ที่เก็บข้อมูลแบบเข้ารหัส

import CryptoKit

struct EncryptedNote: Codable {
    let id: String
    let title: String           // Encrypted
    let content: String         // Encrypted
    let createdAt: Date
    let checksum: String        // For integrity verification
    
    static func create(title: String, content: String, key: SymmetricKey) throws -> EncryptedNote {
        guard let titleData = title.data(using: .utf8),
              let contentData = content.data(using: .utf8) else {
            throw NSError(domain: "NoteError", code: -1)
        }
        
        let encryptedTitle = try CryptoKitManager.encryptAES(titleData, key: key)
        let encryptedContent = try CryptoKitManager.encryptAES(contentData, key: key)
        
        // สร้าง checksum สำหรับตรวจสอบ Integrity
        var hasher = SHA256()
        hasher.update(data: encryptedTitle)
        hasher.update(data: encryptedContent)
        let checksum = hasher.finalize().compactMap { String(format: "%02x", $0) }.joined()
        
        return EncryptedNote(
            id: UUID().uuidString,
            title: encryptedTitle.base64EncodedString(),
            content: encryptedContent.base64EncodedString(),
            createdAt: Date(),
            checksum: checksum
        )
    }
    
    func decrypt(with key: SymmetricKey) throws -> (title: String, content: String) {
        // Verify Integrity ก่อน Decrypt
        guard verifyIntegrity() else {
            throw NSError(domain: "IntegrityError", code: -1, userInfo: [NSLocalizedDescriptionKey: "Data tampered!"])
        }
        
        guard let titleData = Data(base64Encoded: title),
              let contentData = Data(base64Encoded: content) else {
            throw NSError(domain: "DecodeError", code: -2)
        }
        
        let decryptedTitle = try CryptoKitManager.decryptAES(titleData, key: key)
        let decryptedContent = try CryptoKitManager.decryptAES(contentData, key: key)
        
        guard let titleStr = String(data: decryptedTitle, encoding: .utf8),
              let contentStr = String(data: decryptedContent, encoding: .utf8) else {
            throw NSError(domain: "DecodeError", code: -3)
        }
        
        return (titleStr, contentStr)
    }
    
    private func verifyIntegrity() -> Bool {
        guard let titleData = Data(base64Encoded: title),
              let contentData = Data(base64Encoded: content) else {
            return false
        }
        
        var hasher = SHA256()
        hasher.update(data: titleData)
        hasher.update(data: contentData)
        let calculatedChecksum = hasher.finalize().compactMap { String(format: "%02x", $0) }.joined()
        
        return SecureCompare.constantTimeCompareStrings(checksum, calculatedChecksum)
    }
}

class EncryptedNotesManager {
    private let key: SymmetricKey
    private let storageKey = "encrypted_notes"
    
    init() {
        // โหลด Key จาก Keychain หรือสร้างใหม่
        if let existingKey = try? CryptoKitManager.loadKeyFromKeychain(identifier: "notes_encryption_key") {
            key = existingKey
        } else {
            key = CryptoKitManager.generateAESKey()
            try? CryptoKitManager.saveKeyToKeychain(key, identifier: "notes_encryption_key")
        }
    }
    
    func saveNote(title: String, content: String) throws {
        let note = try EncryptedNote.create(title: title, content: content, key: key)
        
        var notes = loadNotes()
        notes.append(note)
        
        let encoded = try JSONEncoder().encode(notes)
        UserDefaults.standard.set(encoded, forKey: storageKey)
    }
    
    func loadNotes() -> [EncryptedNote] {
        guard let data = UserDefaults.standard.data(forKey: storageKey),
              let notes = try? JSONDecoder().decode([EncryptedNote].self, from: data) else {
            return []
        }
        return notes
    }
    
    func decryptNote(_ note: EncryptedNote) throws -> (title: String, content: String) {
        return try note.decrypt(with: key)
    }
}

// การใช้งาน
func exercise2() {
    let manager = EncryptedNotesManager()
    
    do {
        try manager.saveNote(title: "รหัสผ่านสำคัญ", content: "password123")
        
        let notes = manager.loadNotes()
        print("จำนวน notes:", notes.count)
        
        if let firstNote = notes.first {
            let decrypted = try manager.decryptNote(firstNote)
            print("Title:", decrypted.title)
            print("Content:", decrypted.content)
        }
    } catch {
        print("Error:", error)
    }
}
```

### แบบฝึกหัดที่ 3: Secure Network Client

```swift
// โจทย์: สร้าง Network Client ที่ปลอดภัย

import Foundation

class SecureNetworkClient {
    private let session: URLSession
    private let baseURL: URL
    private let tokenManager: TokenManager
    
    init(baseURL: URL) {
        self.baseURL = baseURL
        self.tokenManager = TokenManager()
        
        let config = URLSessionConfiguration.default
        config.tlsMinimumSupportedProtocolVersion = .TLSv12
        
        // ตั้งค่า Timeout
        config.timeoutIntervalForRequest = 30
        config.timeoutIntervalForResource = 60
        
        self.session = URLSession(configuration: config)
    }
    
    // GET Request พร้อม Auth Token
    func get<T: Decodable>(path: String, type: T.Type) async throws -> T {
        let url = baseURL.appendingPathComponent(path)
        var request = URLRequest(url: url)
        request.httpMethod = "GET"
        
        // เพิ่ม Authorization Header
        if let token = tokenManager.getToken() {
            request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        }
        
        // เพิ่ม Security Headers
        request.setValue("application/json", forHTTPHeaderField: "Accept")
        request.setValue("no-cache", forHTTPHeaderField: "Cache-Control")
        
        let (data, response) = try await session.data(for: request)
        
        guard let httpResponse = response as? HTTPURLResponse else {
            throw NetworkError.invalidResponse
        }
        
        switch httpResponse.statusCode {
        case 200...299:
            return try JSONDecoder().decode(type, from: data)
        case 401:
            throw NetworkError.unauthorized
        case 403:
            throw NetworkError.forbidden
        case 404:
            throw NetworkError.notFound
        default:
            throw NetworkError.serverError(httpResponse.statusCode)
        }
    }
    
    // POST Request พร้อม CSRF Protection
    func post<T: Decodable, B: Encodable>(path: String, body: B, type: T.Type) async throws -> T {
        let url = baseURL.appendingPathComponent(path)
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        
        // Auth Token
        if let token = tokenManager.getToken() {
            request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        }
        
        // CSRF Token (ถ้า API ต้องการ)
        if let csrfToken = getCsrfToken() {
            request.setValue(csrfToken, forHTTPHeaderField: "X-CSRF-Token")
        }
        
        request.httpBody = try JSONEncoder().encode(body)
        
        let (data, response) = try await session.data(for: request)
        
        guard let httpResponse = response as? HTTPURLResponse,
              (200...299).contains(httpResponse.statusCode) else {
            throw NetworkError.serverError(0)
        }
        
        return try JSONDecoder().decode(type, from: data)
    }
    
    private func getCsrfToken() -> String? {
        return try? KeychainManager.loadString(key: "csrf_token")
    }
    
    enum NetworkError: Error {
        case invalidResponse
        case unauthorized
        case forbidden
        case notFound
        case serverError(Int)
    }
}

class TokenManager {
    private let tokenKey = "auth_token"
    private let refreshTokenKey = "refresh_token"
    
    func getToken() -> String? {
        return try? KeychainManager.loadString(key: tokenKey)
    }
    
    func saveToken(_ token: String, refreshToken: String? = nil) {
        try? KeychainManager.save(key: tokenKey, string: token)
        if let refresh = refreshToken {
            try? KeychainManager.save(key: refreshTokenKey, string: refresh)
        }
    }
    
    func clearTokens() {
        try? KeychainManager.delete(key: tokenKey)
        try? KeychainManager.delete(key: refreshTokenKey)
    }
    
    func isTokenExpired(_ token: String) -> Bool {
        // Parse JWT Token
        let parts = token.components(separatedBy: ".")
        guard parts.count == 3,
              let payloadData = Data(base64Encoded: padBase64(parts[1])),
              let payload = try? JSONSerialization.jsonObject(with: payloadData) as? [String: Any],
              let exp = payload["exp"] as? TimeInterval else {
            return true
        }
        
        return Date() > Date(timeIntervalSince1970: exp)
    }
    
    private func padBase64(_ string: String) -> String {
        let padding = 4 - string.count % 4
        return padding < 4 ? string + String(repeating: "=", count: padding) : string
    }
}

// การใช้งาน
func exercise3() async {
    guard let baseURL = URL(string: "https://api.example.com") else { return }
    let client = SecureNetworkClient(baseURL: baseURL)
    
    struct User: Codable {
        let id: Int
        let name: String
    }
    
    do {
        let user = try await client.get(path: "/users/1", type: User.self)
        print("User:", user.name)
    } catch {
        print("Error:", error)
    }
}
```

---

## 40.21 Building Secure Login Flow

### การสร้าง Secure Login ครบวงจร

```swift
import UIKit
import LocalAuthentication
import CryptoKit

class SecureLoginManager {
    
    // MARK: - Types
    
    struct LoginCredentials {
        let username: String
        let password: String
    }
    
    struct AuthSession {
        let userId: String
        let accessToken: String
        let expiresAt: Date
        
        var isExpired: Bool {
            return Date() > expiresAt
        }
    }
    
    enum LoginError: Error, LocalizedError {
        case invalidCredentials
        case accountLocked
        case networkError
        case biometricFailed
        case sessionExpired
        
        var errorDescription: String? {
            switch self {
            case .invalidCredentials:
                return "ชื่อผู้ใช้หรือรหัสผ่านไม่ถูกต้อง"
            case .accountLocked:
                return "บัญชีถูกล็อค กรุณาติดต่อผู้ดูแล"
            case .networkError:
                return "เกิดข้อผิดพลาดในการเชื่อมต่อ"
            case .biometricFailed:
                return "การยืนยันตัวตนล้มเหลว"
            case .sessionExpired:
                return "Session หมดอายุ กรุณาเข้าสู่ระบบใหม่"
            }
        }
    }
    
    // MARK: - Properties
    
    private let biometricManager = BiometricAuthManager()
    private let tokenManager = TokenManager()
    private var failedAttempts = 0
    private let maxAttempts = 5
    private let lockoutKey = "login_lockout"
    
    // MARK: - Login Methods
    
    // Login ด้วย Username/Password
    func login(credentials: LoginCredentials) async throws -> AuthSession {
        // ตรวจสอบ Lockout
        guard !isAccountLocked else {
            throw LoginError.accountLocked
        }
        
        // Validate Input
        guard InputValidator.validateLength(credentials.username, min: 3, max: 50),
              InputValidator.validateLength(credentials.password, min: 8, max: 100) else {
            throw LoginError.invalidCredentials
        }
        
        do {
            let session = try await performLogin(credentials: credentials)
            
            // Reset failed attempts หลัง login สำเร็จ
            failedAttempts = 0
            
            // เก็บ Token ใน Keychain
            tokenManager.saveToken(session.accessToken)
            
            return session
            
        } catch LoginError.invalidCredentials {
            failedAttempts += 1
            
            if failedAttempts >= maxAttempts {
                lockAccount()
                throw LoginError.accountLocked
            }
            
            throw LoginError.invalidCredentials
        }
    }
    
    // Biometric Login
    func loginWithBiometric() async throws -> AuthSession {
        // ตรวจสอบว่ามี Saved Session
        guard let savedToken = tokenManager.getToken() else {
            throw LoginError.invalidCredentials
        }
        
        // ตรวจสอบ Token Expiry
        if tokenManager.isTokenExpired(savedToken) {
            throw LoginError.sessionExpired
        }
        
        // Biometric Authentication
        do {
            try await biometricManager.authenticateAsync(reason: "เข้าสู่ระบบด้วย Biometric")
        } catch {
            throw LoginError.biometricFailed
        }
        
        // สร้าง Session จาก Saved Token
        return AuthSession(
            userId: "user_id_from_token",  // Extract จาก JWT
            accessToken: savedToken,
            expiresAt: Date().addingTimeInterval(3600)
        )
    }
    
    // Logout
    func logout() {
        tokenManager.clearTokens()
        failedAttempts = 0
    }
    
    // MARK: - Private Methods
    
    private func performLogin(credentials: LoginCredentials) async throws -> AuthSession {
        guard let url = URL(string: "https://api.example.com/auth/login") else {
            throw LoginError.networkError
        }
        
        // Hash password ก่อนส่ง (ถ้า API ต้องการ)
        let hashedPassword = HashingManager.hashString(credentials.password + "salt", algorithm: .sha256)
        
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        
        let body: [String: String] = [
            "username": credentials.username,
            "password": hashedPassword
        ]
        request.httpBody = try? JSONEncoder().encode(body)
        
        // ใช้ Secure Session
        let session = URLSession(configuration: .default)
        let (data, response) = try await session.data(for: request)
        
        guard let httpResponse = response as? HTTPURLResponse else {
            throw LoginError.networkError
        }
        
        switch httpResponse.statusCode {
        case 200:
            struct LoginResponse: Codable {
                let userId: String
                let accessToken: String
                let expiresIn: Int
            }
            
            let loginResponse = try JSONDecoder().decode(LoginResponse.self, from: data)
            return AuthSession(
                userId: loginResponse.userId,
                accessToken: loginResponse.accessToken,
                expiresAt: Date().addingTimeInterval(TimeInterval(loginResponse.expiresIn))
            )
        case 401:
            throw LoginError.invalidCredentials
        case 423:
            throw LoginError.accountLocked
        default:
            throw LoginError.networkError
        }
    }
    
    private var isAccountLocked: Bool {
        guard let lockoutDate = UserDefaults.standard.object(forKey: lockoutKey) as? Date else {
            return false
        }
        
        // Lock 15 นาที
        let lockDuration: TimeInterval = 15 * 60
        return Date().timeIntervalSince(lockoutDate) < lockDuration
    }
    
    private func lockAccount() {
        UserDefaults.standard.set(Date(), forKey: lockoutKey)
    }
}

// Login ViewController
class LoginViewController: UIViewController {
    
    private let loginManager = SecureLoginManager()
    
    @IBOutlet weak var usernameField: UITextField!
    @IBOutlet weak var passwordField: UITextField!
    @IBOutlet weak var loginButton: UIButton!
    @IBOutlet weak var biometricButton: UIButton!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
    }
    
    private func setupUI() {
        // ป้องกัน Screenshot บน Sensitive Screens
        usernameField.textContentType = .username
        passwordField.textContentType = .password
        passwordField.isSecureTextEntry = true
        
        // ปิด AutoFill ถ้าต้องการ
        // usernameField.textContentType = .oneTimeCode  // Trick to disable autofill
    }
    
    @IBAction func loginTapped() {
        guard let username = usernameField.text, !username.isEmpty,
              let password = passwordField.text, !password.isEmpty else {
            showAlert("กรุณากรอกข้อมูลให้ครบ")
            return
        }
        
        loginButton.isEnabled = false
        
        Task {
            do {
                let session = try await loginManager.login(
                    credentials: .init(username: username, password: password)
                )
                
                await MainActor.run {
                    self.handleLoginSuccess(session: session)
                }
            } catch {
                await MainActor.run {
                    self.handleLoginError(error)
                }
            }
            
            await MainActor.run {
                self.loginButton.isEnabled = true
            }
        }
    }
    
    @IBAction func biometricLoginTapped() {
        Task {
            do {
                let session = try await loginManager.loginWithBiometric()
                await MainActor.run {
                    self.handleLoginSuccess(session: session)
                }
            } catch {
                await MainActor.run {
                    self.handleLoginError(error)
                }
            }
        }
    }
    
    private func handleLoginSuccess(session: SecureLoginManager.AuthSession) {
        print("Login successful for user:", session.userId)
        // Navigate to main screen
    }
    
    private func handleLoginError(_ error: Error) {
        let message: String
        if let loginError = error as? SecureLoginManager.LoginError {
            message = loginError.localizedDescription ?? "เกิดข้อผิดพลาด"
        } else {
            message = "เกิดข้อผิดพลาด กรุณาลองใหม่"
        }
        showAlert(message)
    }
    
    private func showAlert(_ message: String) {
        let alert = UIAlertController(title: "แจ้งเตือน", message: message, preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "ตกลง", style: .default))
        present(alert, animated: true)
    }
}
```

---

## 40.22 สรุป

### สิ่งที่เรียนรู้ในบทนี้

1. **Security Fundamentals**: CIA Triad, Defense in Depth, Threat Modeling
2. **Keychain Services**: SecItem API, การเก็บ Passwords, Tokens, Keys
3. **Biometric Auth**: Face ID, Touch ID ด้วย LocalAuthentication
4. **App Transport Security**: การบังคับ HTTPS
5. **Certificate Pinning**: ป้องกัน MITM Attack
6. **CryptoKit**: AES-GCM, ChaCha20, ECDH, Curve25519
7. **Hashing**: SHA256, SHA512, HMAC
8. **Secure Random**: Token Generation, Password Generator
9. **Jailbreak Detection**: ตรวจสอบสัญญาณ Jailbreak
10. **OWASP Mobile Top 10**: การป้องกันช่องโหว่ยอดนิยม
11. **App Sandbox**: Directory Structure, Data Protection
12. **Privacy Manifest**: iOS 17+ Privacy Requirements
13. **Secure Coding**: Input Validation, Constant-time Comparison
14. **Secure Login Flow**: Complete Login System พร้อม Security Best Practices

### Security Checklist

```
✅ เก็บ Sensitive Data ใน Keychain เท่านั้น
✅ บังคับ HTTPS ด้วย ATS
✅ ใช้ CryptoKit สำหรับ Encryption
✅ Validate ทุก Input
✅ ใช้ Secure Random สำหรับ Tokens
✅ Implement Biometric Authentication สำหรับ Sensitive Operations
✅ ตรวจสอบ Certificate Pinning สำหรับ Critical APIs
✅ Handle Errors อย่างไม่รั่ว Information
✅ ล้าง Sensitive Data จาก Memory เมื่อไม่ใช้
✅ ตรวจสอบ Token Expiry
✅ Implement Account Lockout
✅ ลบข้อมูลผู้ใช้เมื่อ Logout
✅ เพิ่ม Privacy Manifest ตาม iOS 17+
✅ ทำ Security Code Review ก่อน Release
```

### คำแนะนำสำหรับ Production

```swift
// Security ไม่ใช่ Feature เดียว แต่เป็น Process ต่อเนื่อง
// 1. ทำ Penetration Testing ก่อน Launch
// 2. ติดตาม Security Advisories ของ Apple
// 3. อัปเดต Dependencies สม่ำเสมอ
// 4. ทำ Threat Modeling เมื่อ Features เปลี่ยน
// 5. Train Developer เรื่อง Secure Coding

print("Security is not a product, it's a process")
```

---

*จบการเรียน Part 40: Security Best Practices ใน iOS*

*ขอแสดงความยินดีที่เรียนจบทั้ง 40 บทแล้ว! คุณมีทักษะ Swift และ iOS Development ที่ครอบคลุมตั้งแต่พื้นฐานจนถึง Advanced Topics อย่างการเพิ่มประสิทธิภาพและการรักษาความปลอดภัย*
