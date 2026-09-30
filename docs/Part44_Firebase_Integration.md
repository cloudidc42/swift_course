# Part 44: Firebase Integration - การรวม Firebase กับ iOS App

## บทนำ

Firebase คือ Platform สำหรับพัฒนา Application ที่ Google สร้างขึ้น ประกอบด้วย Services ต่างๆ มากมาย ตั้งแต่ Authentication, Database, Storage, Analytics ไปจนถึง Crash Reporting Firebase เป็นทางเลือกที่ยอดนิยมสำหรับ iOS Developer ที่ต้องการ Backend ที่ยืดหยุ่นและรองรับหลาย Platform

ในบทนี้เราจะเรียนรู้:
- การ Setup Firebase ใน iOS Project
- Firebase Authentication ทุกรูปแบบ
- Cloud Firestore - Database หลัก
- Firebase Storage สำหรับไฟล์
- Firebase Cloud Messaging (FCM)
- Firebase Analytics, Crashlytics, Remote Config
- Security Rules
- การสร้าง Chat App ด้วย Firebase

---

## 44.1 Firebase Overview

Firebase คือ Backend-as-a-Service (BaaS) ที่ Google พัฒนา เหมาะสำหรับ App ที่ต้องการ:

### Product Suite ของ Firebase

```
Firebase Products:
├── Build
│   ├── Authentication      - ระบบล็อกอิน
│   ├── Cloud Firestore     - NoSQL Database
│   ├── Realtime Database   - JSON Database (Legacy)
│   ├── Storage             - File Storage
│   ├── Hosting             - Web Hosting
│   ├── Functions           - Serverless Functions
│   ├── Machine Learning    - ML Kit
│   └── Extensions          - Pre-built Solutions
├── Release & Monitor
│   ├── Crashlytics         - Crash Reporting
│   ├── Performance         - App Performance Monitoring
│   ├── App Distribution    - Beta Testing
│   └── Remote Config       - Remote Configuration
└── Engage
    ├── Analytics           - App Analytics
    ├── Cloud Messaging     - Push Notifications
    ├── In-App Messaging    - In-App Messages
    ├── A/B Testing         - Experiment Testing
    └── Dynamic Links       - Deep Links
```

### Firebase vs CloudKit

| Feature | Firebase | CloudKit |
|---------|----------|----------|
| Platform | Cross-platform | Apple Only |
| Auth | หลาย Provider | Apple ID Only |
| Database | Firestore/RTDB | CloudKit DB |
| Free Tier | ใจกว้าง | ใจกว้าง |
| Offline | ✅ ดีมาก | ✅ ดี |
| Real-time | ✅ | ✅ (ผ่าน Subscription) |
| Backend Logic | Cloud Functions | ไม่มี |

---

## 44.2 Adding Firebase to iOS Project

### ขั้นตอนที่ 1: สร้าง Firebase Project

1. ไปที่ [console.firebase.google.com](https://console.firebase.google.com)
2. คลิก **Add project**
3. ตั้งชื่อ Project
4. เปิด/ปิด Google Analytics
5. คลิก **Create project**

### ขั้นตอนที่ 2: เพิ่ม iOS App

1. ในหน้า Project Overview คลิก **iOS** icon
2. ใส่ iOS Bundle ID (เช่น `com.example.myapp`)
3. ใส่ App Nickname (Optional)
4. ดาวน์โหลด `GoogleService-Info.plist`
5. ลาก `GoogleService-Info.plist` เข้า Xcode Project

### ขั้นตอนที่ 3: ติดตั้ง Firebase SDK ด้วย Swift Package Manager

1. เปิด Xcode → File → Add Packages
2. ใส่ URL: `https://github.com/firebase/firebase-ios-sdk`
3. เลือก Package Products ที่ต้องการ:
   - `FirebaseAnalytics`
   - `FirebaseAuth`
   - `FirebaseFirestore`
   - `FirebaseStorage`
   - `FirebaseMessaging`
   - `FirebaseCrashlytics`
   - `FirebaseRemoteConfig`

### ขั้นตอนที่ 4: Initialize Firebase

```swift
// App.swift หรือ AppDelegate.swift
import SwiftUI
import FirebaseCore

@main
struct MyApp: App {
    
    init() {
        FirebaseApp.configure()
    }
    
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}

// หรือใน AppDelegate (UIKit)
import UIKit
import FirebaseCore

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
    
    func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
    ) -> Bool {
        FirebaseApp.configure()
        return true
    }
}
```

### โครงสร้างไฟล์ที่แนะนำ

```
MyApp/
├── App/
│   ├── MyApp.swift           - App Entry Point + Firebase init
│   ├── AppDelegate.swift     - Legacy support
│   └── GoogleService-Info.plist
├── Services/
│   ├── AuthService.swift
│   ├── FirestoreService.swift
│   └── StorageService.swift
├── Models/
│   ├── User.swift
│   └── Message.swift
└── Views/
    ├── AuthViews/
    └── MainViews/
```

---

## 44.3 Firebase Authentication

Firebase Authentication รองรับหลาย Authentication Methods

### 44.3.1 Email/Password Authentication

```swift
import FirebaseAuth

class AuthService: ObservableObject {
    @Published var currentUser: User?
    @Published var isLoggedIn = false
    
    private var authStateHandle: AuthStateDidChangeListenerHandle?
    
    init() {
        setupAuthStateListener()
    }
    
    deinit {
        if let handle = authStateHandle {
            Auth.auth().removeStateDidChangeListener(handle)
        }
    }
    
    private func setupAuthStateListener() {
        authStateHandle = Auth.auth().addStateDidChangeListener { [weak self] _, user in
            DispatchQueue.main.async {
                self?.currentUser = user
                self?.isLoggedIn = user != nil
            }
        }
    }
    
    // Register ด้วย Email/Password
    func register(email: String, password: String) async throws -> User {
        let result = try await Auth.auth().createUser(withEmail: email, password: password)
        return result.user
    }
    
    // Login ด้วย Email/Password
    func login(email: String, password: String) async throws -> User {
        let result = try await Auth.auth().signIn(withEmail: email, password: password)
        return result.user
    }
    
    // Logout
    func logout() throws {
        try Auth.auth().signOut()
    }
    
    // Reset Password
    func resetPassword(email: String) async throws {
        try await Auth.auth().sendPasswordReset(withEmail: email)
    }
    
    // อัปเดต Profile
    func updateProfile(displayName: String?, photoURL: URL?) async throws {
        guard let user = Auth.auth().currentUser else {
            throw AuthError.notLoggedIn
        }
        
        let changeRequest = user.createProfileChangeRequest()
        changeRequest.displayName = displayName
        changeRequest.photoURL = photoURL
        try await changeRequest.commitChanges()
    }
    
    // อัปเดต Email
    func updateEmail(newEmail: String) async throws {
        guard let user = Auth.auth().currentUser else {
            throw AuthError.notLoggedIn
        }
        try await user.updateEmail(to: newEmail)
    }
    
    // อัปเดต Password
    func updatePassword(newPassword: String) async throws {
        guard let user = Auth.auth().currentUser else {
            throw AuthError.notLoggedIn
        }
        try await user.updatePassword(to: newPassword)
    }
    
    // ลบ Account
    func deleteAccount() async throws {
        guard let user = Auth.auth().currentUser else {
            throw AuthError.notLoggedIn
        }
        try await user.delete()
    }
    
    // Re-authenticate (จำเป็นก่อน Sensitive Operations)
    func reauthenticate(email: String, password: String) async throws {
        guard let user = Auth.auth().currentUser else {
            throw AuthError.notLoggedIn
        }
        let credential = EmailAuthProvider.credential(withEmail: email, password: password)
        try await user.reauthenticate(with: credential)
    }
}

enum AuthError: LocalizedError {
    case notLoggedIn
    case invalidCredential
    
    var errorDescription: String? {
        switch self {
        case .notLoggedIn:
            return "กรุณา Login ก่อน"
        case .invalidCredential:
            return "Email หรือ Password ไม่ถูกต้อง"
        }
    }
}
```

### 44.3.2 Google Sign-In

```swift
// ติดตั้ง Package เพิ่มเติม:
// https://github.com/google/GoogleSignIn-iOS

import FirebaseAuth
import GoogleSignIn

extension AuthService {
    
    func signInWithGoogle(presenting viewController: UIViewController) async throws -> User {
        // ดึง Client ID จาก Firebase
        guard let clientID = FirebaseApp.app()?.options.clientID else {
            throw NSError(domain: "Auth", code: -1)
        }
        
        // ตั้งค่า Google Sign-In
        let config = GIDConfiguration(clientID: clientID)
        GIDSignIn.sharedInstance.configuration = config
        
        // เริ่ม Sign-In Flow
        let result = try await GIDSignIn.sharedInstance.signIn(withPresenting: viewController)
        
        guard let idToken = result.user.idToken?.tokenString else {
            throw NSError(domain: "Auth", code: -1, userInfo: [NSLocalizedDescriptionKey: "ไม่ได้รับ ID Token"])
        }
        
        let accessToken = result.user.accessToken.tokenString
        
        // สร้าง Firebase Credential
        let credential = GoogleAuthProvider.credential(
            withIDToken: idToken,
            accessToken: accessToken
        )
        
        // Sign In กับ Firebase
        let authResult = try await Auth.auth().signIn(with: credential)
        return authResult.user
    }
}

// SwiftUI Version ด้วย @MainActor
@MainActor
extension AuthService {
    
    func signInWithGoogleFromScene() async throws -> User {
        guard let windowScene = UIApplication.shared.connectedScenes.first as? UIWindowScene,
              let window = windowScene.windows.first,
              let rootViewController = window.rootViewController else {
            throw NSError(domain: "Auth", code: -1)
        }
        
        return try await signInWithGoogle(presenting: rootViewController)
    }
}
```

### 44.3.3 Apple Sign-In

```swift
import FirebaseAuth
import AuthenticationServices
import CryptoKit

class AppleSignInCoordinator: NSObject, ASAuthorizationControllerDelegate {
    
    private var currentNonce: String?
    private var completion: ((Result<User, Error>) -> Void)?
    
    func signInWithApple(completion: @escaping (Result<User, Error>) -> Void) {
        self.completion = completion
        
        let nonce = randomNonceString()
        currentNonce = nonce
        
        let appleIDProvider = ASAuthorizationAppleIDProvider()
        let request = appleIDProvider.createRequest()
        request.requestedScopes = [.fullName, .email]
        request.nonce = sha256(nonce)
        
        let controller = ASAuthorizationController(authorizationRequests: [request])
        controller.delegate = self
        controller.performRequests()
    }
    
    // MARK: - ASAuthorizationControllerDelegate
    
    func authorizationController(
        controller: ASAuthorizationController,
        didCompleteWithAuthorization authorization: ASAuthorization
    ) {
        guard let appleIDCredential = authorization.credential as? ASAuthorizationAppleIDCredential,
              let nonce = currentNonce,
              let appleIDToken = appleIDCredential.identityToken,
              let idTokenString = String(data: appleIDToken, encoding: .utf8) else {
            completion?(.failure(NSError(domain: "Auth", code: -1)))
            return
        }
        
        let credential = OAuthProvider.appleCredential(
            withIDToken: idTokenString,
            rawNonce: nonce,
            fullName: appleIDCredential.fullName
        )
        
        Task {
            do {
                let result = try await Auth.auth().signIn(with: credential)
                completion?(.success(result.user))
            } catch {
                completion?(.failure(error))
            }
        }
    }
    
    func authorizationController(
        controller: ASAuthorizationController,
        didCompleteWithError error: Error
    ) {
        completion?(.failure(error))
    }
    
    // MARK: - Helpers
    
    private func randomNonceString(length: Int = 32) -> String {
        var randomBytes = [UInt8](repeating: 0, count: length)
        let errorCode = SecRandomCopyBytes(kSecRandomDefault, randomBytes.count, &randomBytes)
        if errorCode != errSecSuccess {
            fatalError("Unable to generate nonce")
        }
        
        let charset: [Character] = Array("0123456789ABCDEFGHIJKLMNOPQRSTUVXYZabcdefghijklmnopqrstuvwxyz-._")
        let nonce = randomBytes.map { byte in
            charset[Int(byte) % charset.count]
        }
        return String(nonce)
    }
    
    private func sha256(_ input: String) -> String {
        let inputData = Data(input.utf8)
        let hashedData = SHA256.hash(data: inputData)
        return hashedData.compactMap { String(format: "%02x", $0) }.joined()
    }
}
```

### 44.3.4 Anonymous Authentication

```swift
extension AuthService {
    
    // Sign In แบบ Anonymous (ไม่ต้องสมัครสมาชิก)
    func signInAnonymously() async throws -> User {
        let result = try await Auth.auth().signInAnonymously()
        return result.user
    }
    
    // Convert Anonymous Account เป็น Permanent Account
    func linkAnonymousWithEmail(email: String, password: String) async throws -> User {
        guard let currentUser = Auth.auth().currentUser, currentUser.isAnonymous else {
            throw AuthError.notLoggedIn
        }
        
        let credential = EmailAuthProvider.credential(withEmail: email, password: password)
        let result = try await currentUser.link(with: credential)
        return result.user
    }
    
    // Link กับ Google
    func linkAnonymousWithGoogle(presenting viewController: UIViewController) async throws -> User {
        guard let clientID = FirebaseApp.app()?.options.clientID else {
            throw NSError(domain: "Auth", code: -1)
        }
        
        let config = GIDConfiguration(clientID: clientID)
        GIDSignIn.sharedInstance.configuration = config
        
        let result = try await GIDSignIn.sharedInstance.signIn(withPresenting: viewController)
        
        guard let idToken = result.user.idToken?.tokenString else {
            throw NSError(domain: "Auth", code: -1)
        }
        
        let credential = GoogleAuthProvider.credential(
            withIDToken: idToken,
            accessToken: result.user.accessToken.tokenString
        )
        
        guard let currentUser = Auth.auth().currentUser else {
            throw AuthError.notLoggedIn
        }
        
        let authResult = try await currentUser.link(with: credential)
        return authResult.user
    }
    
    var isAnonymous: Bool {
        return Auth.auth().currentUser?.isAnonymous ?? false
    }
}
```

### Auth View ด้วย SwiftUI

```swift
import SwiftUI
import FirebaseAuth

struct LoginView: View {
    @StateObject private var authService = AuthService()
    @State private var email = ""
    @State private var password = ""
    @State private var isRegistering = false
    @State private var showError = false
    @State private var errorMessage = ""
    @State private var isLoading = false
    
    var body: some View {
        NavigationView {
            VStack(spacing: 20) {
                // Logo
                Image(systemName: "flame.fill")
                    .resizable()
                    .frame(width: 80, height: 100)
                    .foregroundColor(.orange)
                
                Text("MyFirebaseApp")
                    .font(.largeTitle.bold())
                
                // Form
                VStack(spacing: 15) {
                    TextField("Email", text: $email)
                        .textFieldStyle(RoundedBorderTextFieldStyle())
                        .keyboardType(.emailAddress)
                        .autocapitalization(.none)
                    
                    SecureField("Password", text: $password)
                        .textFieldStyle(RoundedBorderTextFieldStyle())
                }
                .padding(.horizontal)
                
                // Action Buttons
                VStack(spacing: 10) {
                    Button(action: handleEmailAuth) {
                        HStack {
                            if isLoading {
                                ProgressView()
                                    .progressViewStyle(CircularProgressViewStyle(tint: .white))
                            }
                            Text(isRegistering ? "สมัครสมาชิก" : "เข้าสู่ระบบ")
                        }
                        .frame(maxWidth: .infinity)
                        .padding()
                        .background(Color.blue)
                        .foregroundColor(.white)
                        .cornerRadius(10)
                    }
                    .disabled(isLoading)
                    .padding(.horizontal)
                    
                    Button("เข้าใช้แบบไม่สมัครสมาชิก") {
                        handleAnonymousSignIn()
                    }
                    .foregroundColor(.secondary)
                }
                
                // Toggle Register/Login
                Button(isRegistering ? "มีบัญชีอยู่แล้ว? เข้าสู่ระบบ" : "ยังไม่มีบัญชี? สมัครสมาชิก") {
                    isRegistering.toggle()
                }
                .foregroundColor(.blue)
                
                Spacer()
            }
            .padding(.top, 50)
            .navigationBarHidden(true)
            .alert("เกิดข้อผิดพลาด", isPresented: $showError) {
                Button("ตกลง") {}
            } message: {
                Text(errorMessage)
            }
        }
    }
    
    private func handleEmailAuth() {
        isLoading = true
        
        Task {
            do {
                if isRegistering {
                    _ = try await authService.register(email: email, password: password)
                } else {
                    _ = try await authService.login(email: email, password: password)
                }
            } catch {
                showError = true
                errorMessage = error.localizedDescription
            }
            isLoading = false
        }
    }
    
    private func handleAnonymousSignIn() {
        Task {
            do {
                _ = try await authService.signInAnonymously()
            } catch {
                showError = true
                errorMessage = error.localizedDescription
            }
        }
    }
}
```

---

## 44.4 Cloud Firestore

Cloud Firestore คือ NoSQL Document Database ที่ยืดหยุ่นและรองรับ Real-time Updates

### 44.4.1 Collections and Documents

```
Firestore Structure:
collection (users) /
    document (user123) /
        name: "John"
        email: "john@example.com"
        subcollection (posts) /
            document (post456) /
                title: "Hello World"
                content: "..."
                createdAt: Timestamp
```

### การเข้าถึง Firestore

```swift
import FirebaseFirestore

// Singleton Reference
let db = Firestore.firestore()

// Document Reference
let userRef = db.collection("users").document("user123")
let postRef = db.collection("users").document("user123")
                 .collection("posts").document("post456")

// Collection Reference
let usersRef = db.collection("users")
```

### 44.4.2 Reading Data

```swift
import FirebaseFirestore
import FirebaseAuth

// Model
struct UserProfile: Codable, Identifiable {
    @DocumentID var id: String?
    var name: String
    var email: String
    var bio: String
    var createdAt: Date
    var followersCount: Int
    
    enum CodingKeys: String, CodingKey {
        case id
        case name
        case email
        case bio
        case createdAt
        case followersCount = "followers_count"
    }
}

class UserService {
    private let db = Firestore.firestore()
    
    // อ่าน Document เดียว
    func fetchUser(userID: String) async throws -> UserProfile {
        let document = try await db.collection("users").document(userID).getDocument()
        
        guard document.exists else {
            throw FirestoreError.documentNotFound
        }
        
        return try document.data(as: UserProfile.self)
    }
    
    // อ่านหลาย Documents
    func fetchUsers(userIDs: [String]) async throws -> [UserProfile] {
        var profiles: [UserProfile] = []
        
        for userID in userIDs {
            if let profile = try? await fetchUser(userID: userID) {
                profiles.append(profile)
            }
        }
        
        return profiles
    }
    
    // อ่าน Collection ทั้งหมด
    func fetchAllUsers() async throws -> [UserProfile] {
        let snapshot = try await db.collection("users").getDocuments()
        return snapshot.documents.compactMap { document in
            try? document.data(as: UserProfile.self)
        }
    }
}

enum FirestoreError: LocalizedError {
    case documentNotFound
    case invalidData
    
    var errorDescription: String? {
        switch self {
        case .documentNotFound:
            return "ไม่พบข้อมูลที่ต้องการ"
        case .invalidData:
            return "ข้อมูลไม่ถูกต้อง"
        }
    }
}
```

### 44.4.3 Writing Data

```swift
extension UserService {
    
    // สร้าง Document ด้วย Auto ID
    func createUser(name: String, email: String) async throws -> UserProfile {
        let newUser = UserProfile(
            name: name,
            email: email,
            bio: "",
            createdAt: Date(),
            followersCount: 0
        )
        
        let reference = try db.collection("users").addDocument(from: newUser)
        
        // อ่านกลับมาเพื่อได้ ID
        let document = try await reference.getDocument()
        return try document.data(as: UserProfile.self)
    }
    
    // สร้าง Document ด้วย Custom ID (ใช้ User UID จาก Auth)
    func createUserProfile(for authUser: FirebaseAuth.User) async throws {
        let profile = UserProfile(
            id: authUser.uid,
            name: authUser.displayName ?? "ผู้ใช้ใหม่",
            email: authUser.email ?? "",
            bio: "",
            createdAt: Date(),
            followersCount: 0
        )
        
        try db.collection("users").document(authUser.uid).setData(from: profile)
    }
    
    // อัปเดต Document ทั้งหมด (แทนที่ข้อมูลเดิม)
    func updateUserProfile(_ profile: UserProfile) async throws {
        guard let id = profile.id else { throw FirestoreError.invalidData }
        try db.collection("users").document(id).setData(from: profile)
    }
    
    // อัปเดตเฉพาะบาง Fields (merge)
    func updateUserBio(_ bio: String, userID: String) async throws {
        try await db.collection("users").document(userID).updateData([
            "bio": bio,
            "updatedAt": Timestamp(date: Date())
        ])
    }
    
    // ลบ Document
    func deleteUser(userID: String) async throws {
        try await db.collection("users").document(userID).delete()
    }
    
    // ใช้ FieldValue สำหรับ Atomic Operations
    func incrementFollowers(userID: String) async throws {
        try await db.collection("users").document(userID).updateData([
            "followers_count": FieldValue.increment(Int64(1))
        ])
    }
    
    // เพิ่มค่าใน Array
    func addTag(_ tag: String, toUserID userID: String) async throws {
        try await db.collection("users").document(userID).updateData([
            "tags": FieldValue.arrayUnion([tag])
        ])
    }
    
    // ลบค่าจาก Array
    func removeTag(_ tag: String, fromUserID userID: String) async throws {
        try await db.collection("users").document(userID).updateData([
            "tags": FieldValue.arrayRemove([tag])
        ])
    }
    
    // อัปเดต Server Timestamp
    func updateLastSeen(userID: String) async throws {
        try await db.collection("users").document(userID).updateData([
            "lastSeenAt": FieldValue.serverTimestamp()
        ])
    }
}
```

### 44.4.4 Real-time Updates

```swift
import FirebaseFirestore
import Combine

class RealtimeService {
    private let db = Firestore.firestore()
    private var listeners: [ListenerRegistration] = []
    
    // Listen to Document Changes
    func listenToUser(
        userID: String,
        onUpdate: @escaping (UserProfile?) -> Void
    ) -> ListenerRegistration {
        return db.collection("users").document(userID)
            .addSnapshotListener { snapshot, error in
                guard let snapshot = snapshot, error == nil else {
                    onUpdate(nil)
                    return
                }
                
                let profile = try? snapshot.data(as: UserProfile.self)
                DispatchQueue.main.async {
                    onUpdate(profile)
                }
            }
    }
    
    // Listen to Collection Changes
    func listenToMessages(
        in chatRoomID: String,
        onUpdate: @escaping ([Message]) -> Void
    ) -> ListenerRegistration {
        return db.collection("chatRooms")
            .document(chatRoomID)
            .collection("messages")
            .order(by: "sentAt", descending: false)
            .addSnapshotListener { snapshot, error in
                guard let snapshot = snapshot, error == nil else {
                    onUpdate([])
                    return
                }
                
                let messages = snapshot.documents.compactMap { doc in
                    try? doc.data(as: Message.self)
                }
                
                DispatchQueue.main.async {
                    onUpdate(messages)
                }
            }
    }
    
    // Listen พร้อมแยก Changed/Added/Removed
    func listenWithChanges(
        collectionPath: String,
        onAdded: @escaping ([QueryDocumentSnapshot]) -> Void,
        onModified: @escaping ([QueryDocumentSnapshot]) -> Void,
        onRemoved: @escaping ([QueryDocumentSnapshot]) -> Void
    ) -> ListenerRegistration {
        return db.collection(collectionPath)
            .addSnapshotListener { snapshot, error in
                guard let snapshot = snapshot else { return }
                
                var added: [QueryDocumentSnapshot] = []
                var modified: [QueryDocumentSnapshot] = []
                var removed: [QueryDocumentSnapshot] = []
                
                for change in snapshot.documentChanges {
                    switch change.type {
                    case .added:
                        added.append(change.document)
                    case .modified:
                        modified.append(change.document)
                    case .removed:
                        removed.append(change.document)
                    }
                }
                
                DispatchQueue.main.async {
                    if !added.isEmpty { onAdded(added) }
                    if !modified.isEmpty { onModified(modified) }
                    if !removed.isEmpty { onRemoved(removed) }
                }
            }
    }
    
    // ยกเลิก Listener ทั้งหมด
    func removeAllListeners() {
        listeners.forEach { $0.remove() }
        listeners.removeAll()
    }
}
```

### 44.4.5 Queries and Filters

```swift
class QueryService {
    private let db = Firestore.firestore()
    
    // Simple Filter
    func fetchPublishedPosts() async throws -> [Post] {
        let snapshot = try await db.collection("posts")
            .whereField("isPublished", isEqualTo: true)
            .getDocuments()
        
        return snapshot.documents.compactMap { try? $0.data(as: Post.self) }
    }
    
    // Multiple Conditions
    func fetchRecentActivePosts() async throws -> [Post] {
        let oneWeekAgo = Date().addingTimeInterval(-7 * 24 * 3600)
        
        let snapshot = try await db.collection("posts")
            .whereField("isPublished", isEqualTo: true)
            .whereField("createdAt", isGreaterThan: Timestamp(date: oneWeekAgo))
            .order(by: "createdAt", descending: true)
            .limit(to: 20)
            .getDocuments()
        
        return snapshot.documents.compactMap { try? $0.data(as: Post.self) }
    }
    
    // Array Contains
    func fetchPostsWithTag(_ tag: String) async throws -> [Post] {
        let snapshot = try await db.collection("posts")
            .whereField("tags", arrayContains: tag)
            .getDocuments()
        
        return snapshot.documents.compactMap { try? $0.data(as: Post.self) }
    }
    
    // Array Contains Any
    func fetchPostsWithAnyTag(_ tags: [String]) async throws -> [Post] {
        let snapshot = try await db.collection("posts")
            .whereField("tags", arrayContainsAny: tags)
            .getDocuments()
        
        return snapshot.documents.compactMap { try? $0.data(as: Post.self) }
    }
    
    // In Filter
    func fetchPostsByAuthors(_ authorIDs: [String]) async throws -> [Post] {
        let snapshot = try await db.collection("posts")
            .whereField("authorID", in: authorIDs)
            .getDocuments()
        
        return snapshot.documents.compactMap { try? $0.data(as: Post.self) }
    }
    
    // Pagination ด้วย startAfter
    var lastDocument: QueryDocumentSnapshot?
    
    func fetchNextPage() async throws -> [Post] {
        var query = db.collection("posts")
            .order(by: "createdAt", descending: true)
            .limit(to: 10)
        
        if let lastDoc = lastDocument {
            query = query.start(afterDocument: lastDoc)
        }
        
        let snapshot = try await query.getDocuments()
        lastDocument = snapshot.documents.last
        
        return snapshot.documents.compactMap { try? $0.data(as: Post.self) }
    }
    
    // Full-text Search (Firestore ไม่รองรับโดยตรง - ต้องใช้ Algolia หรือ Typesense)
    // วิธีง่ายๆ: ค้นหาจาก Title ขึ้นต้นด้วย
    func searchPostsByTitle(_ searchText: String) async throws -> [Post] {
        let endText = searchText + "\u{f8ff}" // Unicode สูงสุด
        
        let snapshot = try await db.collection("posts")
            .whereField("title", isGreaterThanOrEqualTo: searchText)
            .whereField("title", isLessThanOrEqualTo: endText)
            .getDocuments()
        
        return snapshot.documents.compactMap { try? $0.data(as: Post.self) }
    }
}
```

### 44.4.6 Batch Writes and Transactions

```swift
class BatchService {
    private let db = Firestore.firestore()
    
    // Batch Write - บันทึกหลาย Documents พร้อมกัน (สูงสุด 500)
    func batchCreatePosts(_ posts: [Post]) async throws {
        let batch = db.batch()
        
        for post in posts {
            let ref = db.collection("posts").document()
            try batch.setData(from: post, forDocument: ref)
        }
        
        try await batch.commit()
    }
    
    // Batch Delete
    func batchDeletePosts(postIDs: [String]) async throws {
        let batch = db.batch()
        
        for id in postIDs {
            let ref = db.collection("posts").document(id)
            batch.deleteDocument(ref)
        }
        
        try await batch.commit()
    }
    
    // Transaction - อ่านและเขียนแบบ Atomic
    func transferPoints(from senderID: String, to receiverID: String, amount: Int) async throws {
        try await db.runTransaction { [weak self] transaction, errorPointer in
            guard let self = self else { return nil }
            
            let senderRef = self.db.collection("users").document(senderID)
            let receiverRef = self.db.collection("users").document(receiverID)
            
            // อ่านข้อมูลทั้งสองฝั่ง
            let senderDoc: DocumentSnapshot
            let receiverDoc: DocumentSnapshot
            
            do {
                senderDoc = try transaction.getDocument(senderRef)
                receiverDoc = try transaction.getDocument(receiverRef)
            } catch let fetchError as NSError {
                errorPointer?.pointee = fetchError
                return nil
            }
            
            // ตรวจสอบ Balance
            guard let senderPoints = senderDoc.data()?["points"] as? Int,
                  senderPoints >= amount else {
                errorPointer?.pointee = NSError(
                    domain: "App",
                    code: -1,
                    userInfo: [NSLocalizedDescriptionKey: "Points ไม่เพียงพอ"]
                )
                return nil
            }
            
            // อัปเดต Balance
            transaction.updateData(["points": senderPoints - amount], forDocument: senderRef)
            
            let receiverPoints = receiverDoc.data()?["points"] as? Int ?? 0
            transaction.updateData(["points": receiverPoints + amount], forDocument: receiverRef)
            
            return nil
        }
    }
    
    // Like Post ด้วย Transaction
    func likePost(postID: String, userID: String) async throws {
        let postRef = db.collection("posts").document(postID)
        let likeRef = db.collection("posts").document(postID)
                        .collection("likes").document(userID)
        
        try await db.runTransaction { transaction, errorPointer in
            let postDoc: DocumentSnapshot
            let likeDoc: DocumentSnapshot
            
            do {
                postDoc = try transaction.getDocument(postRef)
                likeDoc = try transaction.getDocument(likeRef)
            } catch let error as NSError {
                errorPointer?.pointee = error
                return nil
            }
            
            if likeDoc.exists {
                // Unlike
                transaction.deleteDocument(likeRef)
                let currentLikes = postDoc.data()?["likesCount"] as? Int ?? 1
                transaction.updateData(["likesCount": max(0, currentLikes - 1)], forDocument: postRef)
            } else {
                // Like
                transaction.setData(["userID": userID, "likedAt": Timestamp()], forDocument: likeRef)
                let currentLikes = postDoc.data()?["likesCount"] as? Int ?? 0
                transaction.updateData(["likesCount": currentLikes + 1], forDocument: postRef)
            }
            
            return nil
        }
    }
}
```

---

## 44.5 Firebase Realtime Database vs Firestore

### ความแตกต่าง

| Feature | Realtime Database | Cloud Firestore |
|---------|-------------------|-----------------|
| Data Model | JSON Tree | Document/Collection |
| Querying | จำกัด | ยืดหยุ่นมาก |
| Offline Support | ✅ | ✅ (ดีกว่า) |
| Pricing | ขนาด DB | Operations |
| Scalability | Single Region | Multi-region |
| Indexes | Manual | Auto + Manual |
| Full-text | ❌ | ❌ |

### Realtime Database (Legacy)

```swift
import FirebaseDatabase

class RealtimeDBService {
    private let ref = Database.database().reference()
    
    // Write
    func saveMessage(_ message: String, to chatRoom: String) {
        let messageRef = ref.child("chatRooms").child(chatRoom).child("messages").childByAutoId()
        messageRef.setValue([
            "text": message,
            "timestamp": ServerValue.timestamp(),
            "senderID": Auth.auth().currentUser?.uid ?? ""
        ])
    }
    
    // Read (Once)
    func fetchMessages(from chatRoom: String, completion: @escaping ([[String: Any]]) -> Void) {
        ref.child("chatRooms").child(chatRoom).child("messages")
            .queryOrderedByChild("timestamp")
            .queryLimited(toLast: 50)
            .observeSingleEvent(of: .value) { snapshot in
                var messages: [[String: Any]] = []
                for child in snapshot.children {
                    if let childSnapshot = child as? DataSnapshot,
                       let value = childSnapshot.value as? [String: Any] {
                        messages.append(value)
                    }
                }
                completion(messages)
            }
    }
    
    // Real-time Listener
    func listenToMessages(from chatRoom: String, onMessage: @escaping ([String: Any]) -> Void) {
        ref.child("chatRooms").child(chatRoom).child("messages")
            .observe(.childAdded) { snapshot in
                if let value = snapshot.value as? [String: Any] {
                    onMessage(value)
                }
            }
    }
}
```

### แนะนำ: ใช้ Firestore สำหรับ Project ใหม่

---

## 44.6 Firebase Storage

Firebase Storage ใช้สำหรับจัดเก็บไฟล์ขนาดใหญ่ เช่น รูปภาพ วิดีโอ เอกสาร

### 44.6.1 Uploading Files

```swift
import FirebaseStorage
import UIKit

class StorageService {
    private let storage = Storage.storage()
    
    // Upload รูปภาพ
    func uploadImage(
        _ image: UIImage,
        path: String,
        progressHandler: ((Double) -> Void)? = nil,
        completion: @escaping (Result<URL, Error>) -> Void
    ) {
        guard let imageData = image.jpegData(compressionQuality: 0.8) else {
            completion(.failure(StorageError.invalidData))
            return
        }
        
        let storageRef = storage.reference().child(path)
        let metadata = StorageMetadata()
        metadata.contentType = "image/jpeg"
        
        let uploadTask = storageRef.putData(imageData, metadata: metadata) { metadata, error in
            if let error = error {
                completion(.failure(error))
                return
            }
            
            // ดึง Download URL
            storageRef.downloadURL { url, error in
                if let error = error {
                    completion(.failure(error))
                } else if let url = url {
                    completion(.success(url))
                }
            }
        }
        
        // Track Progress
        uploadTask.observe(.progress) { snapshot in
            guard let progress = snapshot.progress else { return }
            let percentage = Double(progress.completedUnitCount) / Double(progress.totalUnitCount)
            progressHandler?(percentage)
        }
    }
    
    // Upload ด้วย async/await
    func uploadImageAsync(
        _ image: UIImage,
        userID: String,
        type: ImageType = .profile
    ) async throws -> URL {
        guard let imageData = image.jpegData(compressionQuality: 0.8) else {
            throw StorageError.invalidData
        }
        
        let fileName = "\(UUID().uuidString).jpg"
        let path = "\(type.rawValue)/\(userID)/\(fileName)"
        let storageRef = storage.reference().child(path)
        
        let metadata = StorageMetadata()
        metadata.contentType = "image/jpeg"
        
        _ = try await storageRef.putDataAsync(imageData, metadata: metadata)
        
        let downloadURL = try await storageRef.downloadURL()
        return downloadURL
    }
    
    // Upload ไฟล์จาก URL
    func uploadFile(from localURL: URL, to remotePath: String) async throws -> URL {
        let storageRef = storage.reference().child(remotePath)
        _ = try await storageRef.putFileAsync(from: localURL)
        return try await storageRef.downloadURL()
    }
    
    enum ImageType: String {
        case profile = "profileImages"
        case post = "postImages"
        case chat = "chatImages"
    }
}

enum StorageError: LocalizedError {
    case invalidData
    case uploadFailed
    
    var errorDescription: String? {
        switch self {
        case .invalidData:
            return "ข้อมูลรูปภาพไม่ถูกต้อง"
        case .uploadFailed:
            return "Upload ไม่สำเร็จ"
        }
    }
}
```

### 44.6.2 Downloading Files

```swift
extension StorageService {
    
    // Download เป็น Data
    func downloadImageData(from path: String) async throws -> Data {
        let storageRef = storage.reference().child(path)
        // สูงสุด 10MB
        return try await storageRef.data(maxSize: 10 * 1024 * 1024)
    }
    
    // Download เป็น UIImage
    func downloadImage(from url: URL) async throws -> UIImage {
        let (data, _) = try await URLSession.shared.data(from: url)
        guard let image = UIImage(data: data) else {
            throw StorageError.invalidData
        }
        return image
    }
    
    // Download ไปยังไฟล์ Local
    func downloadToFile(from path: String, to localURL: URL) async throws {
        let storageRef = storage.reference().child(path)
        _ = try await storageRef.writeAsync(toFile: localURL)
    }
    
    // ลบไฟล์
    func deleteFile(at path: String) async throws {
        let storageRef = storage.reference().child(path)
        try await storageRef.delete()
    }
    
    // ดู Metadata
    func getMetadata(for path: String) async throws -> StorageMetadata {
        let storageRef = storage.reference().child(path)
        return try await storageRef.getMetadata()
    }
    
    // List ไฟล์ใน Directory
    func listFiles(in directory: String) async throws -> [StorageReference] {
        let storageRef = storage.reference().child(directory)
        let result = try await storageRef.listAll()
        return result.items
    }
}
```

### 44.6.3 Storage Security Rules

```javascript
// storage.rules
rules_version = '2';
service firebase.storage {
    match /b/{bucket}/o {
        
        // Profile Images - เจ้าของเขียนได้, ทุกคนอ่านได้
        match /profileImages/{userId}/{allPaths=**} {
            allow read: if true;
            allow write: if request.auth != null && request.auth.uid == userId
                         && request.resource.size < 5 * 1024 * 1024 // 5MB
                         && request.resource.contentType.matches('image/.*');
        }
        
        // Post Images - เจ้าของเขียนได้, Login แล้วอ่านได้
        match /postImages/{userId}/{allPaths=**} {
            allow read: if request.auth != null;
            allow write: if request.auth != null && request.auth.uid == userId
                         && request.resource.size < 10 * 1024 * 1024; // 10MB
        }
        
        // Private Files - เจ้าของเท่านั้น
        match /privateFiles/{userId}/{allPaths=**} {
            allow read, write: if request.auth != null && request.auth.uid == userId;
        }
    }
}
```

---

## 44.7 Firebase Cloud Messaging (FCM)

FCM ช่วยส่ง Push Notifications ไปยัง iOS App

### การตั้งค่า FCM

```swift
// AppDelegate.swift
import FirebaseMessaging
import UserNotifications

class AppDelegate: NSObject, UIApplicationDelegate {
    
    func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
    ) -> Bool {
        FirebaseApp.configure()
        
        // ตั้งค่า FCM Delegate
        Messaging.messaging().delegate = self
        
        // ขอ Permission
        UNUserNotificationCenter.current().delegate = self
        UNUserNotificationCenter.current().requestAuthorization(
            options: [.alert, .badge, .sound]
        ) { granted, _ in
            if granted {
                DispatchQueue.main.async {
                    application.registerForRemoteNotifications()
                }
            }
        }
        
        return true
    }
    
    func application(
        _ application: UIApplication,
        didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data
    ) {
        Messaging.messaging().apnsToken = deviceToken
    }
}

// MARK: - MessagingDelegate
extension AppDelegate: MessagingDelegate {
    
    func messaging(_ messaging: Messaging, didReceiveRegistrationToken fcmToken: String?) {
        guard let token = fcmToken else { return }
        print("FCM Token: \(token)")
        
        // บันทึก Token ใน Firestore
        if let userID = Auth.auth().currentUser?.uid {
            Task {
                try? await Firestore.firestore()
                    .collection("users")
                    .document(userID)
                    .updateData(["fcmToken": token])
            }
        }
    }
}

// MARK: - UNUserNotificationCenterDelegate
extension AppDelegate: UNUserNotificationCenterDelegate {
    
    // แสดง Notification ขณะใช้งาน App
    func userNotificationCenter(
        _ center: UNUserNotificationCenter,
        willPresent notification: UNNotification,
        withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void
    ) {
        completionHandler([.banner, .sound, .badge])
    }
    
    // จัดการเมื่อ User แตะ Notification
    func userNotificationCenter(
        _ center: UNUserNotificationCenter,
        didReceive response: UNNotificationResponse,
        withCompletionHandler completionHandler: @escaping () -> Void
    ) {
        let userInfo = response.notification.request.content.userInfo
        
        // ดึงข้อมูลจาก Notification
        if let chatRoomID = userInfo["chatRoomID"] as? String {
            // Navigate ไปยัง Chat Room
            NotificationCenter.default.post(
                name: .openChatRoom,
                object: nil,
                userInfo: ["chatRoomID": chatRoomID]
            )
        }
        
        completionHandler()
    }
}

extension Notification.Name {
    static let openChatRoom = Notification.Name("openChatRoom")
}
```

### การส่ง Notification จาก Server (Cloud Functions)

```javascript
// functions/index.js (Node.js)
const functions = require('firebase-functions');
const admin = require('firebase-admin');
admin.initializeApp();

// ส่ง Notification เมื่อมี Message ใหม่
exports.sendChatNotification = functions.firestore
    .document('chatRooms/{chatRoomId}/messages/{messageId}')
    .onCreate(async (snapshot, context) => {
        const message = snapshot.data();
        const { chatRoomId } = context.params;
        
        // ดึงสมาชิกของ Chat Room
        const chatRoomDoc = await admin.firestore()
            .collection('chatRooms')
            .doc(chatRoomId)
            .get();
        
        const members = chatRoomDoc.data().members;
        
        // ส่ง Notification ให้ทุกคนยกเว้น Sender
        const sendPromises = members
            .filter(memberID => memberID !== message.senderID)
            .map(async (memberID) => {
                const userDoc = await admin.firestore()
                    .collection('users')
                    .doc(memberID)
                    .get();
                
                const fcmToken = userDoc.data()?.fcmToken;
                if (!fcmToken) return;
                
                return admin.messaging().send({
                    token: fcmToken,
                    notification: {
                        title: message.senderName,
                        body: message.text
                    },
                    data: {
                        chatRoomID: chatRoomId,
                        type: 'chat_message'
                    },
                    apns: {
                        payload: {
                            aps: {
                                badge: 1,
                                sound: 'default'
                            }
                        }
                    }
                });
            });
        
        await Promise.all(sendPromises);
    });
```

---

## 44.8 Firebase Analytics

```swift
import FirebaseAnalytics

class AnalyticsService {
    
    // Log Event พื้นฐาน
    static func logEvent(_ name: String, parameters: [String: Any]? = nil) {
        Analytics.logEvent(name, parameters: parameters)
    }
    
    // Predefined Events
    static func logLogin(method: String) {
        Analytics.logEvent(AnalyticsEventLogin, parameters: [
            AnalyticsParameterMethod: method
        ])
    }
    
    static func logSignUp(method: String) {
        Analytics.logEvent(AnalyticsEventSignUp, parameters: [
            AnalyticsParameterMethod: method
        ])
    }
    
    static func logViewItem(itemID: String, itemName: String, category: String) {
        Analytics.logEvent(AnalyticsEventViewItem, parameters: [
            AnalyticsParameterItemID: itemID,
            AnalyticsParameterItemName: itemName,
            AnalyticsParameterItemCategory: category
        ])
    }
    
    static func logPurchase(amount: Double, currency: String, itemID: String) {
        Analytics.logEvent(AnalyticsEventPurchase, parameters: [
            AnalyticsParameterValue: amount,
            AnalyticsParameterCurrency: currency,
            AnalyticsParameterItemID: itemID
        ])
    }
    
    // Custom Events
    static func logNoteCreated(noteType: String) {
        Analytics.logEvent("note_created", parameters: [
            "note_type": noteType,
            "timestamp": Int(Date().timeIntervalSince1970)
        ])
    }
    
    static func logSearch(query: String, resultCount: Int) {
        Analytics.logEvent("search_performed", parameters: [
            "query": query,
            "result_count": resultCount
        ])
    }
    
    // Set User Properties
    static func setUserProperty(_ value: String?, forName name: String) {
        Analytics.setUserProperty(value, forName: name)
    }
    
    static func setUserID(_ userID: String?) {
        Analytics.setUserID(userID)
    }
    
    // Screen Tracking
    static func logScreenView(screenName: String, screenClass: String) {
        Analytics.logEvent(AnalyticsEventScreenView, parameters: [
            AnalyticsParameterScreenName: screenName,
            AnalyticsParameterScreenClass: screenClass
        ])
    }
}

// SwiftUI View Modifier สำหรับ Screen Tracking
struct AnalyticsScreenView: ViewModifier {
    let screenName: String
    let screenClass: String
    
    func body(content: Content) -> some View {
        content.onAppear {
            AnalyticsService.logScreenView(
                screenName: screenName,
                screenClass: screenClass
            )
        }
    }
}

extension View {
    func trackScreen(_ name: String, class screenClass: String = "SwiftUI") -> some View {
        modifier(AnalyticsScreenView(screenName: name, screenClass: screenClass))
    }
}

// การใช้งาน
struct HomeView: View {
    var body: some View {
        Text("Home")
            .trackScreen("Home", class: "HomeView")
    }
}
```

---

## 44.9 Firebase Crashlytics

```swift
import FirebaseCrashlytics

class CrashlyticsService {
    
    // ตั้งค่า User Information
    static func setUser(_ userID: String) {
        Crashlytics.crashlytics().setUserID(userID)
    }
    
    // Log Custom Key-Value
    static func setCustomKey(_ key: String, value: String) {
        Crashlytics.crashlytics().setCustomValue(value, forKey: key)
    }
    
    // Log Custom Message
    static func log(_ message: String) {
        Crashlytics.crashlytics().log(message)
    }
    
    // Record Non-Fatal Error
    static func recordError(_ error: Error, additionalInfo: [String: Any]? = nil) {
        if let info = additionalInfo {
            for (key, value) in info {
                Crashlytics.crashlytics().setCustomValue(value, forKey: key)
            }
        }
        Crashlytics.crashlytics().record(error: error)
    }
    
    // สร้าง Test Crash (สำหรับ Testing เท่านั้น!)
    static func testCrash() {
        #if DEBUG
        Crashlytics.crashlytics().setCrashlyticsCollectionEnabled(false)
        fatalError("Test crash")
        #endif
    }
}

// การใช้งานใน Error Handling
func fetchUserData() async throws -> UserProfile {
    do {
        return try await userService.fetchUser(userID: "123")
    } catch {
        CrashlyticsService.recordError(error, additionalInfo: [
            "operation": "fetchUserData",
            "userID": "123"
        ])
        throw error
    }
}
```

---

## 44.10 Firebase Remote Config

```swift
import FirebaseRemoteConfig

class RemoteConfigService {
    private let remoteConfig = RemoteConfig.remoteConfig()
    
    // ค่า Default
    private let defaults: [String: NSObject] = [
        "welcome_message": "ยินดีต้อนรับ!" as NSObject,
        "max_items_per_page": 20 as NSObject,
        "feature_dark_mode": false as NSObject,
        "minimum_app_version": "1.0.0" as NSObject,
        "maintenance_mode": false as NSObject
    ]
    
    func configure() {
        remoteConfig.setDefaults(defaults)
        
        // ตั้งค่า Fetch Interval
        let settings = RemoteConfigSettings()
        settings.minimumFetchInterval = 3600 // 1 ชั่วโมง (Production)
        #if DEBUG
        settings.minimumFetchInterval = 0 // Debug: Fetch ทุกครั้ง
        #endif
        remoteConfig.configSettings = settings
    }
    
    // Fetch และ Activate
    func fetchAndActivate() async throws {
        let status = try await remoteConfig.fetchAndActivate()
        print("Remote Config status: \(status)")
    }
    
    // อ่านค่า
    var welcomeMessage: String {
        remoteConfig["welcome_message"].stringValue ?? "ยินดีต้อนรับ!"
    }
    
    var maxItemsPerPage: Int {
        Int(remoteConfig["max_items_per_page"].numberValue)
    }
    
    var isDarkModeEnabled: Bool {
        remoteConfig["feature_dark_mode"].boolValue
    }
    
    var isMaintenanceMode: Bool {
        remoteConfig["maintenance_mode"].boolValue
    }
    
    var minimumAppVersion: String {
        remoteConfig["minimum_app_version"].stringValue ?? "1.0.0"
    }
    
    // ตรวจสอบว่า App Version ผ่านขั้นต่ำ
    func isAppVersionValid() -> Bool {
        let currentVersion = Bundle.main.infoDictionary?["CFBundleShortVersionString"] as? String ?? "0.0.0"
        return currentVersion.compare(minimumAppVersion, options: .numeric) != .orderedAscending
    }
}
```

---

## 44.11 Security Rules for Firestore

```javascript
// firestore.rules
rules_version = '2';
service cloud.firestore {
    match /databases/{database}/documents {
        
        // Helper Functions
        function isAuthenticated() {
            return request.auth != null;
        }
        
        function isOwner(userID) {
            return request.auth.uid == userID;
        }
        
        function isAdmin() {
            return get(/databases/$(database)/documents/admins/$(request.auth.uid)).data.isAdmin == true;
        }
        
        function isValidUser() {
            return request.resource.data.keys().hasAll(['name', 'email', 'createdAt'])
                && request.resource.data.name is string
                && request.resource.data.name.size() > 0
                && request.resource.data.email is string;
        }
        
        // Users Collection
        match /users/{userID} {
            allow read: if isAuthenticated();
            allow create: if isOwner(userID) && isValidUser();
            allow update: if isOwner(userID) || isAdmin();
            allow delete: if isAdmin();
        }
        
        // Posts Collection
        match /posts/{postID} {
            allow read: if resource.data.isPublished == true || isOwner(resource.data.authorID);
            allow create: if isAuthenticated() 
                          && request.resource.data.authorID == request.auth.uid
                          && request.resource.data.keys().hasAll(['title', 'content', 'authorID']);
            allow update: if isOwner(resource.data.authorID) || isAdmin();
            allow delete: if isOwner(resource.data.authorID) || isAdmin();
            
            // Comments Subcollection
            match /comments/{commentID} {
                allow read: if isAuthenticated();
                allow create: if isAuthenticated()
                              && request.resource.data.authorID == request.auth.uid;
                allow update, delete: if isOwner(resource.data.authorID) || isAdmin();
            }
        }
        
        // Chat Rooms
        match /chatRooms/{chatRoomID} {
            allow read: if isAuthenticated() 
                        && request.auth.uid in resource.data.members;
            allow create: if isAuthenticated()
                          && request.auth.uid in request.resource.data.members;
            allow update: if isAuthenticated()
                          && request.auth.uid in resource.data.members;
            
            // Messages
            match /messages/{messageID} {
                allow read: if isAuthenticated()
                            && request.auth.uid in get(/databases/$(database)/documents/chatRooms/$(chatRoomID)).data.members;
                allow create: if isAuthenticated()
                              && request.resource.data.senderID == request.auth.uid
                              && request.auth.uid in get(/databases/$(database)/documents/chatRooms/$(chatRoomID)).data.members;
            }
        }
        
        // Admin Only
        match /admins/{userID} {
            allow read, write: if isAdmin();
        }
        
        // Block everything else
        match /{document=**} {
            allow read, write: if false;
        }
    }
}
```

---

## 44.12 Practical Exercises

### Exercise 1: User Authentication Flow

**โจทย์:** สร้าง Complete Authentication Flow รวม Register, Login, Logout

```swift
// AuthViewModel.swift
import SwiftUI
import FirebaseAuth
import FirebaseFirestore

@MainActor
class AuthViewModel: ObservableObject {
    @Published var currentUser: FirebaseAuth.User?
    @Published var isLoading = false
    @Published var errorMessage: String?
    
    private let auth = Auth.auth()
    private let db = Firestore.firestore()
    
    init() {
        currentUser = auth.currentUser
        
        auth.addStateDidChangeListener { [weak self] _, user in
            self?.currentUser = user
        }
    }
    
    var isLoggedIn: Bool { currentUser != nil }
    
    func register(email: String, password: String, displayName: String) async {
        isLoading = true
        errorMessage = nil
        defer { isLoading = false }
        
        do {
            // สร้าง Account
            let result = try await auth.createUser(withEmail: email, password: password)
            
            // อัปเดต Display Name
            let request = result.user.createProfileChangeRequest()
            request.displayName = displayName
            try await request.commitChanges()
            
            // สร้าง User Document ใน Firestore
            try await db.collection("users").document(result.user.uid).setData([
                "uid": result.user.uid,
                "email": email,
                "displayName": displayName,
                "createdAt": Timestamp(),
                "isActive": true
            ])
            
        } catch {
            errorMessage = handleAuthError(error)
        }
    }
    
    func login(email: String, password: String) async {
        isLoading = true
        errorMessage = nil
        defer { isLoading = false }
        
        do {
            _ = try await auth.signIn(withEmail: email, password: password)
        } catch {
            errorMessage = handleAuthError(error)
        }
    }
    
    func logout() {
        do {
            try auth.signOut()
        } catch {
            errorMessage = error.localizedDescription
        }
    }
    
    func resetPassword(email: String) async {
        isLoading = true
        defer { isLoading = false }
        
        do {
            try await auth.sendPasswordReset(withEmail: email)
            errorMessage = "ส่งอีเมล Reset Password แล้ว"
        } catch {
            errorMessage = error.localizedDescription
        }
    }
    
    private func handleAuthError(_ error: Error) -> String {
        let authError = error as NSError
        switch AuthErrorCode(rawValue: authError.code) {
        case .emailAlreadyInUse:
            return "Email นี้มีการใช้งานแล้ว"
        case .wrongPassword:
            return "Password ไม่ถูกต้อง"
        case .userNotFound:
            return "ไม่พบบัญชีผู้ใช้นี้"
        case .weakPassword:
            return "Password ต้องมีอย่างน้อย 6 ตัวอักษร"
        case .invalidEmail:
            return "Email ไม่ถูกต้อง"
        case .networkError:
            return "เกิดข้อผิดพลาดเครือข่าย กรุณาลองใหม่"
        default:
            return error.localizedDescription
        }
    }
}
```

---

## 44.13 Building a Chat App with Firebase

มาสร้าง Chat App แบบ Real-time กัน!

### Model

```swift
// Models/ChatMessage.swift
import FirebaseFirestore
import Foundation

struct ChatMessage: Codable, Identifiable {
    @DocumentID var id: String?
    var text: String
    var senderID: String
    var senderName: String
    var senderAvatarURL: String?
    var sentAt: Date
    var isRead: Bool
    var type: MessageType
    var mediaURL: String?
    
    enum MessageType: String, Codable {
        case text
        case image
        case audio
        case file
    }
    
    var isCurrentUser: Bool {
        return senderID == AuthManager.shared.currentUserID
    }
}

// Models/ChatRoom.swift
struct ChatRoom: Codable, Identifiable {
    @DocumentID var id: String?
    var name: String?
    var members: [String]
    var lastMessage: String?
    var lastMessageAt: Date?
    var isGroupChat: Bool
    var createdBy: String
    var createdAt: Date
    var unreadCounts: [String: Int]
    
    var otherMemberID: String? {
        guard !isGroupChat else { return nil }
        return members.first { $0 != AuthManager.shared.currentUserID }
    }
}

// Singleton for current user ID
class AuthManager {
    static let shared = AuthManager()
    private init() {}
    
    var currentUserID: String {
        return Auth.auth().currentUser?.uid ?? ""
    }
}
```

### ChatService

```swift
import FirebaseFirestore
import FirebaseAuth

class ChatService: ObservableObject {
    private let db = Firestore.firestore()
    @Published var chatRooms: [ChatRoom] = []
    private var listener: ListenerRegistration?
    
    // MARK: - Chat Rooms
    
    func startListeningToChatRooms() {
        guard let userID = Auth.auth().currentUser?.uid else { return }
        
        listener = db.collection("chatRooms")
            .whereField("members", arrayContains: userID)
            .order(by: "lastMessageAt", descending: true)
            .addSnapshotListener { [weak self] snapshot, error in
                guard let snapshot = snapshot else { return }
                
                self?.chatRooms = snapshot.documents.compactMap { doc in
                    try? doc.data(as: ChatRoom.self)
                }
            }
    }
    
    func stopListening() {
        listener?.remove()
        listener = nil
    }
    
    func createChatRoom(with otherUserID: String) async throws -> ChatRoom {
        guard let currentUserID = Auth.auth().currentUser?.uid else {
            throw ChatError.notAuthenticated
        }
        
        // ตรวจสอบว่า Chat Room มีอยู่แล้วหรือยัง
        let existingRooms = try await db.collection("chatRooms")
            .whereField("members", arrayContains: currentUserID)
            .whereField("isGroupChat", isEqualTo: false)
            .getDocuments()
        
        for document in existingRooms.documents {
            if let room = try? document.data(as: ChatRoom.self),
               room.members.contains(otherUserID) {
                return room
            }
        }
        
        // สร้าง Chat Room ใหม่
        let newRoom = ChatRoom(
            members: [currentUserID, otherUserID],
            isGroupChat: false,
            createdBy: currentUserID,
            createdAt: Date(),
            unreadCounts: [currentUserID: 0, otherUserID: 0]
        )
        
        let ref = try db.collection("chatRooms").addDocument(from: newRoom)
        let document = try await ref.getDocument()
        return try document.data(as: ChatRoom.self)
    }
    
    func createGroupChat(name: String, memberIDs: [String]) async throws -> ChatRoom {
        guard let currentUserID = Auth.auth().currentUser?.uid else {
            throw ChatError.notAuthenticated
        }
        
        var allMembers = memberIDs
        if !allMembers.contains(currentUserID) {
            allMembers.append(currentUserID)
        }
        
        var unreadCounts: [String: Int] = [:]
        allMembers.forEach { unreadCounts[$0] = 0 }
        
        let newRoom = ChatRoom(
            name: name,
            members: allMembers,
            isGroupChat: true,
            createdBy: currentUserID,
            createdAt: Date(),
            unreadCounts: unreadCounts
        )
        
        let ref = try db.collection("chatRooms").addDocument(from: newRoom)
        let document = try await ref.getDocument()
        return try document.data(as: ChatRoom.self)
    }
    
    // MARK: - Messages
    
    func sendMessage(text: String, in chatRoomID: String) async throws {
        guard let currentUser = Auth.auth().currentUser else {
            throw ChatError.notAuthenticated
        }
        
        let message = ChatMessage(
            text: text,
            senderID: currentUser.uid,
            senderName: currentUser.displayName ?? "Unknown",
            sentAt: Date(),
            isRead: false,
            type: .text
        )
        
        let batch = db.batch()
        
        // เพิ่ม Message
        let messageRef = db.collection("chatRooms")
            .document(chatRoomID)
            .collection("messages")
            .document()
        
        try batch.setData(from: message, forDocument: messageRef)
        
        // อัปเดต Last Message ใน Chat Room
        let chatRoomRef = db.collection("chatRooms").document(chatRoomID)
        batch.updateData([
            "lastMessage": text,
            "lastMessageAt": Timestamp(date: Date())
        ], forDocument: chatRoomRef)
        
        try await batch.commit()
    }
    
    func listenToMessages(
        in chatRoomID: String,
        handler: @escaping ([ChatMessage]) -> Void
    ) -> ListenerRegistration {
        return db.collection("chatRooms")
            .document(chatRoomID)
            .collection("messages")
            .order(by: "sentAt", descending: false)
            .limit(toLast: 50)
            .addSnapshotListener { snapshot, _ in
                let messages = snapshot?.documents.compactMap {
                    try? $0.data(as: ChatMessage.self)
                } ?? []
                
                DispatchQueue.main.async {
                    handler(messages)
                }
            }
    }
    
    func markMessagesAsRead(in chatRoomID: String, userID: String) async throws {
        try await db.collection("chatRooms")
            .document(chatRoomID)
            .updateData(["unreadCounts.\(userID)": 0])
    }
}

enum ChatError: LocalizedError {
    case notAuthenticated
    case roomNotFound
    
    var errorDescription: String? {
        switch self {
        case .notAuthenticated: return "กรุณาเข้าสู่ระบบ"
        case .roomNotFound: return "ไม่พบห้องสนทนา"
        }
    }
}
```

### ChatView

```swift
import SwiftUI
import FirebaseFirestore

struct ChatRoomView: View {
    let chatRoom: ChatRoom
    @StateObject private var viewModel: ChatRoomViewModel
    @State private var messageText = ""
    @State private var scrollProxy: ScrollViewProxy?
    
    init(chatRoom: ChatRoom) {
        self.chatRoom = chatRoom
        _viewModel = StateObject(wrappedValue: ChatRoomViewModel(chatRoomID: chatRoom.id ?? ""))
    }
    
    var body: some View {
        VStack(spacing: 0) {
            // Messages List
            ScrollViewReader { proxy in
                ScrollView {
                    LazyVStack(spacing: 8) {
                        ForEach(viewModel.messages) { message in
                            MessageBubble(message: message)
                                .id(message.id)
                        }
                    }
                    .padding(.horizontal)
                    .padding(.vertical, 8)
                }
                .onChange(of: viewModel.messages.count) { _ in
                    if let lastMessage = viewModel.messages.last {
                        withAnimation(.easeOut(duration: 0.3)) {
                            proxy.scrollTo(lastMessage.id, anchor: .bottom)
                        }
                    }
                }
            }
            
            Divider()
            
            // Input Area
            HStack(spacing: 12) {
                Button {
                    // เพิ่ม Attachment
                } label: {
                    Image(systemName: "plus.circle.fill")
                        .font(.title2)
                        .foregroundColor(.blue)
                }
                
                TextField("ข้อความ...", text: $messageText, axis: .vertical)
                    .textFieldStyle(.roundedBorder)
                    .lineLimit(1...4)
                
                Button {
                    sendMessage()
                } label: {
                    Image(systemName: "arrow.up.circle.fill")
                        .font(.title2)
                        .foregroundColor(messageText.isEmpty ? .gray : .blue)
                }
                .disabled(messageText.isEmpty)
            }
            .padding()
            .background(Color(.systemBackground))
        }
        .navigationTitle(chatRoomTitle)
        .navigationBarTitleDisplayMode(.inline)
        .onDisappear {
            viewModel.stopListening()
        }
    }
    
    private var chatRoomTitle: String {
        return chatRoom.name ?? "Chat"
    }
    
    private func sendMessage() {
        let text = messageText.trimmingCharacters(in: .whitespacesAndNewlines)
        guard !text.isEmpty else { return }
        
        messageText = ""
        
        Task {
            await viewModel.sendMessage(text: text)
        }
    }
}

// MessageBubble
struct MessageBubble: View {
    let message: ChatMessage
    
    var body: some View {
        HStack(alignment: .bottom, spacing: 8) {
            if message.isCurrentUser {
                Spacer()
            } else {
                // Avatar
                Circle()
                    .fill(Color.gray.opacity(0.3))
                    .frame(width: 32, height: 32)
                    .overlay(
                        Text(message.senderName.prefix(1))
                            .font(.caption.bold())
                    )
            }
            
            VStack(alignment: message.isCurrentUser ? .trailing : .leading, spacing: 4) {
                if !message.isCurrentUser {
                    Text(message.senderName)
                        .font(.caption)
                        .foregroundColor(.secondary)
                }
                
                Text(message.text)
                    .padding(.horizontal, 12)
                    .padding(.vertical, 8)
                    .background(
                        message.isCurrentUser ? Color.blue : Color(.systemGray5)
                    )
                    .foregroundColor(message.isCurrentUser ? .white : .primary)
                    .clipShape(RoundedRectangle(cornerRadius: 16))
                
                Text(message.sentAt.formatted(date: .omitted, time: .shortened))
                    .font(.caption2)
                    .foregroundColor(.secondary)
            }
            
            if !message.isCurrentUser {
                Spacer()
            }
        }
    }
}

// ChatRoomViewModel
@MainActor
class ChatRoomViewModel: ObservableObject {
    @Published var messages: [ChatMessage] = []
    @Published var isLoading = false
    
    private let chatService = ChatService()
    private let chatRoomID: String
    private var listener: ListenerRegistration?
    
    init(chatRoomID: String) {
        self.chatRoomID = chatRoomID
        startListening()
    }
    
    func startListening() {
        listener = chatService.listenToMessages(in: chatRoomID) { [weak self] messages in
            self?.messages = messages
        }
    }
    
    func stopListening() {
        listener?.remove()
    }
    
    func sendMessage(text: String) async {
        do {
            try await chatService.sendMessage(text: text, in: chatRoomID)
        } catch {
            print("Error sending message: \(error)")
        }
    }
}
```

---

## 44.14 Offline Support ใน Firestore

```swift
// ตั้งค่า Offline Persistence
class FirebaseConfig {
    static func configure() {
        FirebaseApp.configure()
        
        let settings = FirestoreSettings()
        settings.isPersistenceEnabled = true  // Default: true บน Mobile
        settings.cacheSizeBytes = FirestoreCacheSizeUnlimited  // หรือกำหนดเอง
        Firestore.firestore().settings = settings
    }
}

// ใช้ Cache เมื่อ Offline
class OfflineAwareService {
    private let db = Firestore.firestore()
    
    func fetchPosts(useCache: Bool = false) async throws -> [Post] {
        let source: FirestoreSource = useCache ? .cache : .default
        
        let snapshot = try await db.collection("posts")
            .getDocuments(source: source)
        
        return snapshot.documents.compactMap { try? $0.data(as: Post.self) }
    }
    
    // ตรวจสอบว่าข้อมูลมาจาก Cache หรือ Server
    func fetchWithSourceInfo() async throws {
        let snapshot = try await db.collection("posts").getDocuments()
        print("Is from cache: \(snapshot.metadata.isFromCache)")
        print("Has pending writes: \(snapshot.metadata.hasPendingWrites)")
    }
}
```

---

## 44.15 Firebase Performance Monitoring

```swift
import FirebasePerformance

class PerformanceService {
    
    // Track Custom Trace
    func measureOperation(name: String, operation: @escaping () async throws -> Void) async throws {
        let trace = Performance.startTrace(name: name)
        
        defer { trace?.stop() }
        
        try await operation()
    }
    
    // Track Network Request
    func trackNetworkRequest(url: String) -> HTTPMetric? {
        guard let url = URL(string: url) else { return nil }
        return HTTPMetric(url: url, httpMethod: .get)
    }
    
    // Custom Metrics
    func measureDataFetch() async {
        let trace = Performance.startTrace(name: "data_fetch")
        trace?.setValue(Int64(0), forMetric: "items_fetched")
        
        // Perform operation...
        let itemCount = 50
        
        trace?.setValue(Int64(itemCount), forMetric: "items_fetched")
        trace?.stop()
    }
}
```

---

## 44.16 Error Handling ใน Firebase

```swift
import FirebaseFirestore
import FirebaseAuth
import FirebaseStorage

class FirebaseErrorHandler {
    
    static func handleFirestoreError(_ error: Error) -> String {
        let nsError = error as NSError
        
        switch nsError.code {
        case FirestoreErrorCode.cancelled.rawValue:
            return "การดำเนินการถูกยกเลิก"
        case FirestoreErrorCode.unknown.rawValue:
            return "เกิดข้อผิดพลาดที่ไม่ทราบสาเหตุ"
        case FirestoreErrorCode.invalidArgument.rawValue:
            return "ข้อมูลที่ส่งไม่ถูกต้อง"
        case FirestoreErrorCode.notFound.rawValue:
            return "ไม่พบข้อมูลที่ต้องการ"
        case FirestoreErrorCode.alreadyExists.rawValue:
            return "ข้อมูลนี้มีอยู่แล้ว"
        case FirestoreErrorCode.permissionDenied.rawValue:
            return "ไม่มีสิทธิ์เข้าถึงข้อมูลนี้"
        case FirestoreErrorCode.resourceExhausted.rawValue:
            return "เกินขีดจำกัดการใช้งาน"
        case FirestoreErrorCode.unauthenticated.rawValue:
            return "กรุณาเข้าสู่ระบบ"
        case FirestoreErrorCode.unavailable.rawValue:
            return "บริการไม่พร้อมใช้งาน กรุณาลองใหม่"
        case FirestoreErrorCode.dataLoss.rawValue:
            return "เกิดการสูญหายของข้อมูล"
        default:
            return error.localizedDescription
        }
    }
    
    static func handleStorageError(_ error: Error) -> String {
        let nsError = error as NSError
        
        switch nsError.code {
        case StorageErrorCode.objectNotFound.rawValue:
            return "ไม่พบไฟล์ที่ต้องการ"
        case StorageErrorCode.bucketNotFound.rawValue:
            return "ไม่พบ Storage Bucket"
        case StorageErrorCode.unauthorized.rawValue:
            return "ไม่มีสิทธิ์เข้าถึงไฟล์นี้"
        case StorageErrorCode.cancelled.rawValue:
            return "การ Upload/Download ถูกยกเลิก"
        case StorageErrorCode.unknown.rawValue:
            return "เกิดข้อผิดพลาด กรุณาลองใหม่"
        case StorageErrorCode.quotaExceeded.rawValue:
            return "พื้นที่จัดเก็บเต็ม"
        case StorageErrorCode.retryLimitExceeded.rawValue:
            return "เกินจำนวนครั้งที่ลองใหม่"
        default:
            return error.localizedDescription
        }
    }
}
```

---

## 44.17 Testing Firebase

```swift
import XCTest
import FirebaseFirestore
@testable import MyApp

// ใช้ Firebase Emulator สำหรับ Testing
class FirebaseEmulatorTests: XCTestCase {
    
    override class func setUp() {
        super.setUp()
        
        // เชื่อมต่อ Emulator
        let settings = Firestore.firestore().settings
        settings.host = "localhost:8080"
        settings.isSSLEnabled = false
        Firestore.firestore().settings = settings
        
        // Auth Emulator
        Auth.auth().useEmulator(withHost: "localhost", port: 9099)
    }
    
    func testCreateUser() async throws {
        let authResult = try await Auth.auth().createUser(
            withEmail: "test@example.com",
            password: "password123"
        )
        
        XCTAssertNotNil(authResult.user)
        XCTAssertEqual(authResult.user.email, "test@example.com")
    }
    
    func testFirestoreWrite() async throws {
        let db = Firestore.firestore()
        
        try await db.collection("test").document("doc1").setData([
            "title": "Test Document",
            "value": 42
        ])
        
        let document = try await db.collection("test").document("doc1").getDocument()
        
        XCTAssertEqual(document.data()?["title"] as? String, "Test Document")
        XCTAssertEqual(document.data()?["value"] as? Int, 42)
    }
}
```

---

## 44.18 Firebase Best Practices

```swift
// 1. ใช้ async/await แทน Callback
// ✅ ดี
func fetchUser() async throws -> UserProfile {
    let doc = try await db.collection("users").document("id").getDocument()
    return try doc.data(as: UserProfile.self)
}

// ❌ ไม่ดีเท่า
func fetchUser(completion: @escaping (UserProfile?) -> Void) {
    db.collection("users").document("id").getDocument { doc, _ in
        completion(try? doc?.data(as: UserProfile.self))
    }
}

// 2. จัดการ Listener Lifecycle อย่างถูกต้อง
class ProperViewModel: ObservableObject {
    private var listener: ListenerRegistration?
    
    deinit {
        listener?.remove()  // สำคัญมาก! ต้องลบ Listener เมื่อไม่ใช้งาน
    }
}

// 3. ใช้ Codable สำหรับ Data Mapping
// ✅ ดี - ใช้ @DocumentID
struct Post: Codable, Identifiable {
    @DocumentID var id: String?
    var title: String
    var content: String
}

// 4. Denormalize Data สำหรับ Read Performance
// แทนที่จะ Join หลาย Collections ให้เก็บข้อมูลซ้ำ
struct Message: Codable {
    var text: String
    var senderID: String
    var senderName: String    // เก็บซ้ำเพื่อไม่ต้อง Query อีก Collection
    var senderAvatarURL: String?  // เก็บซ้ำ
}

// 5. จำกัดจำนวน Documents ที่ Query
func fetchRecentPosts() async throws -> [Post] {
    try await db.collection("posts")
        .order(by: "createdAt", descending: true)
        .limit(to: 20)  // จำกัดไว้เสมอ!
        .getDocuments()
        .documents
        .compactMap { try? $0.data(as: Post.self) }
}

// 6. ใช้ Offline Persistence ให้เหมาะสม
class OfflineManager {
    func disableNetwork() async throws {
        try await Firestore.firestore().disableNetwork()
    }
    
    func enableNetwork() async throws {
        try await Firestore.firestore().enableNetwork()
    }
    
    func clearCache() async throws {
        try await Firestore.firestore().clearPersistence()
    }
}
```

---

## 44.19 Migration: Firestore Schema Changes

```swift
// การจัดการ Schema Migration
class MigrationManager {
    private let db = Firestore.firestore()
    
    // เพิ่ม Field ใหม่ให้ทุก Document (ทำที่ Server ด้วย Cloud Functions ดีกว่า)
    func addDefaultFieldToAllPosts() async throws {
        let snapshot = try await db.collection("posts").getDocuments()
        
        let batch = db.batch()
        var updateCount = 0
        
        for document in snapshot.documents {
            if document.data()["version"] == nil {
                batch.updateData(["version": 1], forDocument: document.reference)
                updateCount += 1
                
                // Batch limit: 500 operations
                if updateCount == 499 {
                    try await batch.commit()
                    updateCount = 0
                }
            }
        }
        
        if updateCount > 0 {
            try await batch.commit()
        }
    }
}
```

---

## 44.20 สรุป

ในบทนี้เราได้เรียนรู้:

1. **Firebase Setup** - การเพิ่ม Firebase เข้า iOS Project
2. **Authentication** - Email/Password, Google, Apple, Anonymous
3. **Cloud Firestore** - CRUD Operations, Real-time Updates, Queries
4. **Batch Writes & Transactions** - Atomic Operations
5. **Firebase Storage** - Upload/Download Files
6. **FCM** - Push Notifications
7. **Analytics** - Event Tracking
8. **Crashlytics** - Crash Reporting
9. **Remote Config** - Dynamic Configuration
10. **Security Rules** - ปกป้องข้อมูล
11. **Chat App** - Real-world Application

### สิ่งที่ต้องระวัง

```swift
// Common Mistakes:
// 1. ไม่ลบ Listener → Memory Leak
// 2. ไม่จำกัด Query Results → ค่าใช้จ่ายสูง
// 3. ไม่ตรวจสอบ Auth State ก่อน Write
// 4. Security Rules ที่หละหลวมเกินไป
// 5. ไม่ Handle Offline State
// 6. Nested Callbacks (Callback Hell)
```

### Checklist

- [ ] Setup Firebase Project และ Add iOS App
- [ ] Implement Email/Password Authentication
- [ ] ทำ Firestore CRUD Operations
- [ ] ตั้งค่า Real-time Listener
- [ ] Upload/Download ด้วย Firebase Storage
- [ ] ตั้งค่า Push Notification ด้วย FCM
- [ ] เขียน Security Rules ที่ปลอดภัย
- [ ] ใช้ Firebase Analytics
- [ ] Setup Crashlytics
- [ ] Handle Offline State อย่างถูกต้อง

### แหล่งเรียนรู้เพิ่มเติม

- [Firebase Documentation](https://firebase.google.com/docs)
- [Firebase iOS Codelab](https://firebase.google.com/codelabs/firestore-ios)
- [Firebase YouTube Channel](https://www.youtube.com/c/firebase)
- [FlutterFire Documentation](https://firebase.flutter.dev) (สำหรับ Flutter)
- [Firebase Extensions](https://extensions.dev)

---

*บทถัดไป: Part 45 - RESTful API Integration*
