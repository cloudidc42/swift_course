# Part 63: CI/CD สำหรับ iOS Development

## สารบัญ

1. [CI/CD คืออะไร?](#cicd-คืออะไร)
2. [GitHub Actions สำหรับ iOS](#github-actions-สำหรับ-ios)
3. [Xcode Cloud](#xcode-cloud)
4. [Fastlane](#fastlane)
5. [Bitrise](#bitrise)
6. [CircleCI สำหรับ iOS](#circleci-สำหรับ-ios)
7. [Jenkins สำหรับ iOS](#jenkins-สำหรับ-ios)
8. [Code Signing Automation](#code-signing-automation)
9. [Simulator Testing ใน CI](#simulator-testing-ใน-ci)
10. [Parallel Testing](#parallel-testing)
11. [Test Result Reporting](#test-result-reporting)
12. [Code Coverage ใน CI](#code-coverage-ใน-ci)
13. [Slack Notifications จาก CI](#slack-notifications-จาก-ci)
14. [Version Bumping Automation](#version-bumping-automation)
15. [App Store Deployment Automation](#app-store-deployment-automation)
16. [.xcconfig Files](#xcconfig-files)
17. [Build Configurations](#build-configurations)
18. [Environment Variables ใน CI](#environment-variables-ใน-ci)
19. [Secrets Management](#secrets-management)
20. [Practical Exercises](#practical-exercises)
21. [Setting Up a Complete CI/CD Pipeline](#setting-up-a-complete-cicd-pipeline)
22. [สรุป](#สรุป)

---

## CI/CD คืออะไร?

**CI/CD** ย่อมาจาก **Continuous Integration / Continuous Delivery (หรือ Continuous Deployment)** เป็นแนวทางการพัฒนาซอฟต์แวร์ที่ช่วยให้ทีมสามารถส่งมอบโค้ดได้เร็วและมีความน่าเชื่อถือมากขึ้น

### Continuous Integration (CI)

CI เป็นกระบวนการที่นักพัฒนา **merge โค้ด** เข้า repository หลักบ่อยๆ (อย่างน้อยวันละครั้ง) โดยทุกครั้งที่ push โค้ด ระบบจะทำการ:

- **Build** โปรเจกต์อัตโนมัติ
- **Run Tests** อัตโนมัติ
- **Report ผลลัพธ์** ให้ทีม

```
Developer → Push Code → CI Server → Build → Test → Report
```

### Continuous Delivery (CD)

CD เป็นกระบวนการต่อจาก CI ที่ช่วยให้:

- โค้ดที่ผ่านการทดสอบพร้อม **deploy ได้ตลอดเวลา**
- การ deploy ขึ้น staging/production เป็นแบบ **อัตโนมัติหรือกึ่งอัตโนมัติ**

### ประโยชน์ของ CI/CD

1. **ลด Integration Hell** - หาข้อผิดพลาดได้เร็วขึ้น
2. **เพิ่มความมั่นใจ** ในการ release
3. **ลดเวลา** ในการ review และ merge โค้ด
4. **Automate งานซ้ำซ้อน** เช่น build, test, deploy
5. **Feedback เร็วขึ้น** - นักพัฒนารู้ผลภายในนาที

### CI/CD Pipeline สำหรับ iOS

```
Source Code
    ↓
Code Review (Pull Request)
    ↓
CI Trigger (Push/PR)
    ↓
┌─────────────────────────────┐
│  CI Pipeline                 │
│  1. Checkout code            │
│  2. Install dependencies     │
│  3. Build app                │
│  4. Run unit tests           │
│  5. Run UI tests             │
│  6. Code coverage check      │
│  7. Code quality check       │
│  8. Archive app              │
│  9. Sign app                 │
│  10. Upload to TestFlight    │
└─────────────────────────────┘
    ↓
Notification (Slack/Email)
    ↓
QA Testing
    ↓
App Store Release
```

### เครื่องมือ CI/CD ยอดนิยมสำหรับ iOS

| เครื่องมือ | ข้อดี | ข้อเสีย |
|-----------|-------|---------|
| GitHub Actions | ฟรีสำหรับ public repo, integrate กับ GitHub | ต้องการ macOS runner |
| Xcode Cloud | native สำหรับ Apple, ง่ายในการตั้งค่า | ราคาแพง, ต้องมี Apple Developer |
| Fastlane | open source, ยืดหยุ่น | ต้องตั้งค่าเอง |
| Bitrise | ง่าย, มี iOS stack พร้อม | ราคาสูงสำหรับ large team |
| CircleCI | เร็ว, ยืดหยุ่น | ซับซ้อนในการตั้งค่า |
| Jenkins | ฟรี, customizable | ต้องดูแลเซิร์ฟเวอร์เอง |

---

## GitHub Actions สำหรับ iOS

**GitHub Actions** เป็นบริการ CI/CD ที่ built-in อยู่ใน GitHub ช่วยให้คุณสร้าง workflow อัตโนมัติได้ตรงใน repository

### โครงสร้าง GitHub Actions

```
.github/
└── workflows/
    ├── ios-ci.yml
    ├── ios-cd.yml
    └── pr-check.yml
```

### Workflow พื้นฐาน

สร้างไฟล์ `.github/workflows/ios-ci.yml`:

```yaml
name: iOS CI

# กำหนดว่า workflow นี้จะทำงานเมื่อไร
on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  build-and-test:
    # ใช้ macOS runner เพราะต้องการ Xcode
    runs-on: macos-latest
    
    steps:
    # Step 1: Checkout โค้ด
    - name: Checkout
      uses: actions/checkout@v4
    
    # Step 2: เลือกเวอร์ชัน Xcode
    - name: Select Xcode
      run: sudo xcode-select -switch /Applications/Xcode_15.0.app
    
    # Step 3: Cache CocoaPods dependencies
    - name: Cache CocoaPods
      uses: actions/cache@v3
      with:
        path: Pods
        key: ${{ runner.os }}-pods-${{ hashFiles('**/Podfile.lock') }}
        restore-keys: |
          ${{ runner.os }}-pods-
    
    # Step 4: Install CocoaPods
    - name: Install CocoaPods
      run: pod install --repo-update
    
    # Step 5: Build app
    - name: Build
      run: |
        xcodebuild build \
          -workspace MyApp.xcworkspace \
          -scheme MyApp \
          -destination 'platform=iOS Simulator,name=iPhone 15,OS=17.0' \
          CODE_SIGNING_ALLOWED=NO
    
    # Step 6: Run tests
    - name: Test
      run: |
        xcodebuild test \
          -workspace MyApp.xcworkspace \
          -scheme MyApp \
          -destination 'platform=iOS Simulator,name=iPhone 15,OS=17.0' \
          -resultBundlePath TestResults \
          CODE_SIGNING_ALLOWED=NO
    
    # Step 7: Upload test results
    - name: Upload Test Results
      uses: actions/upload-artifact@v3
      if: always()
      with:
        name: test-results
        path: TestResults.xcresult
```

### Workflow สำหรับ Swift Package Manager

```yaml
name: SPM Build and Test

on:
  push:
    branches: [ main ]
  pull_request:

jobs:
  test:
    runs-on: macos-latest
    
    strategy:
      matrix:
        xcode: ['15.0', '14.3']
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Select Xcode ${{ matrix.xcode }}
      run: sudo xcode-select -switch /Applications/Xcode_${{ matrix.xcode }}.app
    
    - name: Build
      run: swift build -v
    
    - name: Test
      run: swift test -v --enable-code-coverage
    
    - name: Generate Coverage Report
      run: |
        xcrun llvm-cov export \
          .build/debug/MyPackagePackageTests.xctest/Contents/MacOS/MyPackagePackageTests \
          -instr-profile .build/debug/codecov/default.profdata \
          -format="lcov" > coverage.lcov
    
    - name: Upload Coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage.lcov
```

### Workflow สำหรับ Release

```yaml
name: iOS Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: macos-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Ruby
      uses: ruby/setup-ruby@v1
      with:
        ruby-version: '3.0'
        bundler-cache: true
    
    - name: Install Fastlane
      run: bundle install
    
    - name: Setup signing certificates
      env:
        MATCH_PASSWORD: ${{ secrets.MATCH_PASSWORD }}
        FASTLANE_APPLE_APPLICATION_SPECIFIC_PASSWORD: ${{ secrets.APP_SPECIFIC_PASSWORD }}
        MATCH_GIT_URL: ${{ secrets.MATCH_GIT_URL }}
      run: bundle exec fastlane setup_certificates
    
    - name: Build and Upload to TestFlight
      env:
        APP_STORE_CONNECT_API_KEY_ID: ${{ secrets.APP_STORE_CONNECT_API_KEY_ID }}
        APP_STORE_CONNECT_API_ISSUER_ID: ${{ secrets.APP_STORE_CONNECT_API_ISSUER_ID }}
        APP_STORE_CONNECT_API_KEY_CONTENT: ${{ secrets.APP_STORE_CONNECT_API_KEY_CONTENT }}
      run: bundle exec fastlane beta
```

### การใช้ Actions Secrets

```yaml
# การอ่านค่า secret ใน workflow
- name: Decode Provisioning Profile
  run: |
    echo "${{ secrets.PROVISIONING_PROFILE }}" | base64 --decode > profile.mobileprovision
```

### GitHub Actions Matrix Strategy

ใช้เมื่อต้องการทดสอบบน multiple configurations:

```yaml
jobs:
  test:
    runs-on: macos-latest
    
    strategy:
      fail-fast: false
      matrix:
        destination:
          - 'platform=iOS Simulator,name=iPhone 15,OS=17.0'
          - 'platform=iOS Simulator,name=iPhone SE (3rd generation),OS=17.0'
          - 'platform=iOS Simulator,name=iPad Pro (12.9-inch) (6th generation),OS=17.0'
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Test on ${{ matrix.destination }}
      run: |
        xcodebuild test \
          -scheme MyApp \
          -destination '${{ matrix.destination }}'
```

---

## Xcode Cloud

**Xcode Cloud** เป็นบริการ CI/CD ของ Apple ที่ built-in อยู่ใน Xcode และ App Store Connect

### การตั้งค่า Xcode Cloud

1. เปิด Xcode → เลือก Product → Xcode Cloud → Create Workflow
2. เลือก product (App, Framework, etc.)
3. กำหนด workflow conditions
4. เพิ่ม actions (Build, Test, Archive, etc.)

### โครงสร้าง ci_scripts

Xcode Cloud รองรับ custom scripts ผ่าน folder `ci_scripts`:

```
YourApp/
└── ci_scripts/
    ├── ci_post_clone.sh      # หลัง clone repository
    ├── ci_pre_xcodebuild.sh  # ก่อน build
    └── ci_post_xcodebuild.sh # หลัง build
```

### ci_post_clone.sh

```bash
#!/bin/sh

# ตั้งค่าให้ script หยุดทำงานเมื่อเกิด error
set -e

# ติดตั้ง CocoaPods ถ้าใช้งาน
if [ -f "Podfile" ]; then
    echo "Installing CocoaPods dependencies..."
    pod install
fi

# ติดตั้ง dependencies ผ่าน Mint
if [ -f "Mintfile" ]; then
    echo "Installing Mint packages..."
    brew install mint
    mint bootstrap
fi

echo "Post-clone script completed successfully"
```

### ci_pre_xcodebuild.sh

```bash
#!/bin/sh

set -e

# กำหนด version จาก tag
if [ -n "$CI_TAG" ]; then
    # ดึง version number จาก tag (เช่น v1.2.3 → 1.2.3)
    VERSION=$(echo "$CI_TAG" | sed 's/^v//')
    
    # อัปเดต Info.plist
    /usr/libexec/PlistBuddy -c "Set :CFBundleShortVersionString $VERSION" \
        "$CI_PRIMARY_REPOSITORY_PATH/MyApp/Info.plist"
    
    echo "Updated version to: $VERSION"
fi

# ตั้งค่า API endpoint ตาม environment
if [ "$CI_WORKFLOW" = "Production" ]; then
    echo "API_BASE_URL=https://api.production.com" >> "$CI_DERIVED_DATA_PATH/.env"
else
    echo "API_BASE_URL=https://api.staging.com" >> "$CI_DERIVED_DATA_PATH/.env"
fi
```

### Environment Variables ของ Xcode Cloud

```bash
# ตัวแปรที่ Xcode Cloud ให้มาอัตโนมัติ
CI_WORKSPACE             # path ของ workspace
CI_PROJECT_FILE_PATH     # path ของ .xcodeproj
CI_SCHEME                # scheme ที่ใช้ build
CI_WORKFLOW              # ชื่อ workflow
CI_TAG                   # git tag (ถ้ามี)
CI_BRANCH                # ชื่อ branch
CI_COMMIT                # commit SHA
CI_BUILD_NUMBER          # build number อัตโนมัติ
CI_PRODUCT_PLATFORM      # platform (iOS, macOS, etc.)
CI_XCODE_PROJECT         # ชื่อ Xcode project
CI_XCODE_SCHEME          # ชื่อ scheme
```

---

## Fastlane

**Fastlane** เป็น open-source automation tool สำหรับ iOS และ Android development ช่วย automate งาน repetitive ต่างๆ

### การติดตั้ง Fastlane

```bash
# วิธีที่แนะนำ: ผ่าน Bundler
gem install bundler

# สร้าง Gemfile
bundle init

# เพิ่ม fastlane ใน Gemfile
echo 'gem "fastlane"' >> Gemfile

# ติดตั้ง
bundle install

# หรือติดตั้งโดยตรง
gem install fastlane

# หรือผ่าน Homebrew
brew install fastlane
```

### Fastlane Setup

```bash
# เริ่มต้น fastlane ในโปรเจกต์
cd /path/to/your/project
fastlane init

# จะถามว่าต้องการทำอะไร:
# 1. Automate screenshots
# 2. Automate beta distribution to TestFlight
# 3. Automate App Store distribution
# 4. Manual setup
```

หลังจาก init จะได้โครงสร้าง:

```
fastlane/
├── Appfile          # ข้อมูล app (bundle ID, Apple ID)
├── Fastfile         # กำหนด lanes
├── Matchfile        # การตั้งค่า code signing
└── Deliverfile      # ข้อมูลสำหรับ App Store
```

### Appfile

```ruby
# fastlane/Appfile

# Bundle identifier ของ app
app_identifier("com.example.myapp")

# Apple ID ของนักพัฒนา
apple_id("developer@example.com")

# Team ID (ดูได้จาก Apple Developer Portal)
team_id("ABCDE12345")

# App Store Connect Team ID (ถ้าต่างจาก team_id)
itc_team_id("12345678")
```

### Fastfile

**Fastfile** คือไฟล์หลักที่กำหนด lanes (กระบวนการอัตโนมัติ):

```ruby
# fastlane/Fastfile

# กำหนด platform
default_platform(:ios)

platform :ios do
  
  # ============================================
  # LANE: setup - ตั้งค่าเริ่มต้น
  # ============================================
  desc "Setup project dependencies"
  lane :setup do
    # ติดตั้ง CocoaPods
    cocoapods(
      clean_install: true,
      repo_update: true
    )
    
    # ดึง certificates และ profiles
    match(type: "development")
  end
  
  # ============================================
  # LANE: test - รัน unit tests
  # ============================================
  desc "Run all tests"
  lane :test do
    run_tests(
      scheme: "MyApp",
      devices: ["iPhone 15"],
      clean: true,
      code_coverage: true,
      output_directory: "fastlane/test_output",
      output_types: "html,junit"
    )
  end
  
  # ============================================
  # LANE: beta - build และ upload ไป TestFlight
  # ============================================
  desc "Build and distribute to TestFlight"
  lane :beta do
    # อัปเดต version
    increment_build_number(
      build_number: latest_testflight_build_number + 1
    )
    
    # ดึง certificates
    match(type: "appstore")
    
    # Build app
    build_app(
      scheme: "MyApp",
      export_method: "app-store",
      output_directory: "./build",
      output_name: "MyApp.ipa"
    )
    
    # Upload ไป TestFlight
    upload_to_testflight(
      skip_waiting_for_build_processing: true
    )
    
    # แจ้งเตือนผ่าน Slack
    slack(
      message: "New beta build uploaded to TestFlight! 🚀",
      channel: "#ios-deployments",
      success: true
    )
  end
  
  # ============================================
  # LANE: release - release ไป App Store
  # ============================================
  desc "Deploy to App Store"
  lane :release do
    # ตรวจสอบ git status
    ensure_git_status_clean
    
    # อัปเดต version
    increment_version_number(
      version_number: get_version_number
    )
    
    # Build app
    match(type: "appstore")
    build_app(
      scheme: "MyApp",
      export_method: "app-store"
    )
    
    # Upload ไป App Store
    upload_to_app_store(
      submit_for_review: true,
      automatic_release: true,
      skip_metadata: false,
      skip_screenshots: false
    )
    
    # สร้าง git tag
    add_git_tag(
      tag: "v#{get_version_number}"
    )
    push_git_tags
  end
  
  # ============================================
  # ERROR HANDLING
  # ============================================
  error do |lane, exception|
    slack(
      message: "Error in lane #{lane}: #{exception.message}",
      channel: "#ios-errors",
      success: false
    )
  end
  
end
```

### Match (Code Signing)

**match** เป็น tool ที่ช่วยจัดการ certificates และ provisioning profiles แบบ team-based

#### วิธีการทำงานของ match

```
Developer Machine ──────────────────────────────────────→ Git Repo (encrypted)
    │                                                          │
    │  match(type: "development")                              │
    │  1. ดึง certificate จาก repo                           │
    │  2. Install ลงเครื่อง                                  │
    │  3. ดึง provisioning profile                           │
    │  4. Install ลงเครื่อง                                  │
    └──────────────────────────────────────────────────────────┘
```

#### การตั้งค่า match

```bash
# Initialize match
fastlane match init

# จะถามเกี่ยวกับ storage type:
# 1. git (encrypted git repository)
# 2. google_cloud
# 3. s3
# 4. gitlab_secure_files
```

#### Matchfile

```ruby
# fastlane/Matchfile

# URL ของ repository ที่เก็บ certificates
git_url("https://github.com/your-org/certificates-repo")

# ประเภทของ storage
storage_mode("git")

# ประเภทของ certificate
type("development")  # สามารถเป็น development, adhoc, appstore, enterprise

# App identifier
app_identifier(["com.example.myapp", "com.example.myapp.extension"])

# Apple ID
username("developer@example.com")

# Team
team_id("ABCDE12345")
```

#### การสร้าง certificates ใหม่

```bash
# สร้าง development certificates
fastlane match development

# สร้าง app-store certificates
fastlane match appstore

# สร้าง adhoc certificates
fastlane match adhoc

# Force recreate (ลบของเก่าและสร้างใหม่)
fastlane match development --force

# Readonly mode (ไม่สร้างใหม่ แค่ดึงของที่มีอยู่)
fastlane match development --readonly
```

#### ใช้ match ใน Fastfile

```ruby
lane :sign do
  match(
    type: "appstore",
    readonly: is_ci,           # CI จะใช้ readonly mode
    app_identifier: "com.example.myapp",
    git_url: "https://github.com/your-org/certs",
    git_branch: "main"
  )
end
```

### Gym (Building)

**gym** (หรือ `build_app`) ช่วย build และ archive iOS apps:

```ruby
# การใช้งาน gym
lane :build do
  gym(
    # ชื่อ scheme
    scheme: "MyApp",
    
    # ประเภทการ export
    export_method: "app-store",  # app-store, ad-hoc, development, enterprise
    
    # ตำแหน่งที่เก็บผลลัพธ์
    output_directory: "./build",
    output_name: "MyApp.ipa",
    
    # ข้าม clean (เร็วกว่าแต่อาจมีปัญหา)
    clean: false,
    
    # Xcode configuration
    configuration: "Release",
    
    # Build settings เพิ่มเติม
    xcargs: "OTHER_SWIFT_FLAGS='-DPRODUCTION'",
    
    # Workspace หรือ Project
    workspace: "MyApp.xcworkspace",
    # project: "MyApp.xcodeproj",  # ถ้าไม่ใช้ workspace
    
    # Export options
    export_options: {
      provisioningProfiles: {
        "com.example.myapp" => "Match AppStore com.example.myapp"
      }
    }
  )
end
```

### Deliver (Uploading)

**deliver** (หรือ `upload_to_app_store`) ช่วย upload app และ metadata ไป App Store:

```ruby
lane :upload do
  deliver(
    # ข้อมูล app
    app_identifier: "com.example.myapp",
    
    # ภาษาหลัก
    primary_first_sub_category: "UTILITIES",
    
    # Metadata
    metadata_path: "./fastlane/metadata",
    
    # Screenshots
    screenshots_path: "./fastlane/screenshots",
    
    # ตัวเลือกการ submit
    submit_for_review: true,
    automatic_release: false,
    
    # ข้ามการ upload บางส่วน
    skip_binary_upload: false,
    skip_metadata: false,
    skip_screenshots: false,
    
    # Force ให้ overwrite
    force: true,
    
    # Submission information
    submission_information: {
      add_id_info_uses_idfa: false,
      export_compliance_uses_encryption: false
    }
  )
end
```

#### โครงสร้าง metadata

```
fastlane/
└── metadata/
    ├── en-US/
    │   ├── name.txt
    │   ├── subtitle.txt
    │   ├── description.txt
    │   ├── keywords.txt
    │   ├── release_notes.txt
    │   ├── support_url.txt
    │   └── privacy_url.txt
    └── th/
        ├── name.txt
        ├── description.txt
        └── keywords.txt
```

### Scan (Testing)

**scan** (หรือ `run_tests`) ช่วยรัน tests:

```ruby
lane :test do
  scan(
    # Scheme
    scheme: "MyApp",
    
    # Devices
    devices: ["iPhone 15", "iPad Pro (12.9-inch) (6th generation)"],
    
    # ล้างก่อน test
    clean: true,
    
    # Code coverage
    code_coverage: true,
    
    # ผลลัพธ์
    output_directory: "fastlane/test_output",
    output_types: "html,junit",
    output_files: "test-results.html,test-results.xml",
    
    # Fail on errors
    fail_build: true,
    
    # จำนวนครั้งที่ retry ถ้า test fail
    number_of_retries: 3,
    
    # ข้าม build
    skip_build: false
  )
end
```

### Pilot (TestFlight)

**pilot** (หรือ `upload_to_testflight`) จัดการ TestFlight:

```ruby
lane :testflight do
  pilot(
    # ข้าม wait
    skip_waiting_for_build_processing: false,
    
    # อัปเดต info ของ build
    changelog: "Bug fixes and performance improvements",
    
    # เพิ่ม testers โดยอัตโนมัติ
    distribute_external: true,
    groups: ["Beta Testers", "Internal Team"],
    
    # Notification
    notify_external_testers: true,
    
    # ข้อความสำหรับ testers
    beta_app_review_info: {
      contact_email: "support@example.com",
      contact_first_name: "John",
      contact_last_name: "Doe",
      contact_phone: "+1234567890",
      demo_account_name: "test@example.com",
      demo_account_password: "password123",
      notes: "Please test the new feature..."
    }
  )
end
```

### Screengrab (Screenshots)

**screengrab** ช่วย automate การถ่าย screenshots สำหรับ App Store:

```ruby
lane :screenshots do
  capture_ios_screenshots(
    # Scheme สำหรับ UI Tests
    scheme: "MyAppUITests",
    
    # Devices ที่ต้องการ screenshots
    devices: [
      "iPhone 15 Pro Max",
      "iPhone SE (3rd generation)",
      "iPad Pro (12.9-inch) (6th generation)"
    ],
    
    # ภาษา
    languages: ["en-US", "th"],
    
    # ตำแหน่งเก็บ screenshots
    output_directory: "./fastlane/screenshots",
    
    # ลบ screenshots เก่าก่อน
    clear_previous_screenshots: true,
    
    # concurrent simulators
    concurrent_simulators: true
  )
  
  # สร้าง framed screenshots
  frame_screenshots(
    white: true
  )
end
```

---

## Bitrise

**Bitrise** เป็น CI/CD platform ที่เน้น mobile development มี pre-configured iOS stacks

### Bitrise Configuration (bitrise.yml)

```yaml
format_version: '11'
default_step_lib_source: https://github.com/bitrise-io/bitrise-steplib.git

app:
  envs:
  - BITRISE_PROJECT_PATH: MyApp.xcworkspace
  - BITRISE_SCHEME: MyApp
  - BITRISE_EXPORT_METHOD: app-store

workflows:
  # Workflow สำหรับ PR checks
  pr-check:
    steps:
    - activate-ssh-key@4:
        run_if: '{{getenv "SSH_RSA_PRIVATE_KEY" | ne ""}}'
    
    - git-clone@6: {}
    
    - certificate-and-profile-installer@1: {}
    
    - cocoapods-install@2:
        inputs:
        - is_update_cocoapods: 'false'
    
    - xcode-test@4:
        inputs:
        - project_path: $BITRISE_PROJECT_PATH
        - scheme: $BITRISE_SCHEME
        - destination: platform=iOS Simulator,name=iPhone 15,OS=latest
        - test_repetition_mode: none
    
    - deploy-to-bitrise-io@2: {}
  
  # Workflow สำหรับ deployment ไป TestFlight
  deploy:
    steps:
    - activate-ssh-key@4: {}
    
    - git-clone@6: {}
    
    - certificate-and-profile-installer@1: {}
    
    - fastlane@3:
        inputs:
        - lane: beta
        - work_dir: $BITRISE_SOURCE_DIR
    
    - slack@4:
        inputs:
        - webhook_url: $SLACK_WEBHOOK_URL
        - channel: '#ios-deployments'
        - message: 'New build deployed to TestFlight!'
```

---

## CircleCI สำหรับ iOS

```yaml
# .circleci/config.yml
version: 2.1

orbs:
  macos: circleci/macos@2

jobs:
  build-and-test:
    macos:
      xcode: "15.0.0"
    
    environment:
      FL_OUTPUT_DIR: output
      FASTLANE_LANE: test
    
    steps:
      - checkout
      
      - restore_cache:
          key: gems-{{ checksum "Gemfile.lock" }}
      
      - run:
          name: Install Bundler dependencies
          command: bundle install --path vendor/bundle
      
      - save_cache:
          key: gems-{{ checksum "Gemfile.lock" }}
          paths:
            - vendor/bundle
      
      - restore_cache:
          key: pods-{{ checksum "Podfile.lock" }}
      
      - run:
          name: Install CocoaPods
          command: bundle exec pod install --repo-update
      
      - save_cache:
          key: pods-{{ checksum "Podfile.lock" }}
          paths:
            - Pods
      
      - run:
          name: Run Tests
          command: bundle exec fastlane test
      
      - store_test_results:
          path: output/scan
      
      - store_artifacts:
          path: output
  
  deploy:
    macos:
      xcode: "15.0.0"
    
    steps:
      - checkout
      
      - run:
          name: Install dependencies
          command: bundle install --path vendor/bundle
      
      - run:
          name: Deploy to TestFlight
          command: bundle exec fastlane beta
          no_output_timeout: 30m

workflows:
  build-test-deploy:
    jobs:
      - build-and-test:
          filters:
            branches:
              only:
                - main
                - develop
      
      - deploy:
          requires:
            - build-and-test
          filters:
            branches:
              only: main
```

---

## Jenkins สำหรับ iOS

**Jenkins** เป็น open-source CI server ที่ต้องการการดูแลเซิร์ฟเวอร์เอง

### Jenkinsfile (Declarative Pipeline)

```groovy
// Jenkinsfile
pipeline {
    agent {
        // ต้องใช้ macOS agent สำหรับ iOS builds
        label 'macos'
    }
    
    environment {
        DEVELOPER_DIR = '/Applications/Xcode.app/Contents/Developer'
        FASTLANE_SKIP_UPDATE_CHECK = '1'
        FASTLANE_HIDE_TIMESTAMP = '1'
        // ดึง secrets จาก Jenkins Credentials
        MATCH_PASSWORD = credentials('match-password')
        APP_STORE_API_KEY = credentials('app-store-api-key')
    }
    
    options {
        timeout(time: 1, unit: 'HOURS')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh 'bundle install'
                sh 'bundle exec pod install --repo-update'
            }
        }
        
        stage('Run Tests') {
            steps {
                sh 'bundle exec fastlane test'
            }
            post {
                always {
                    junit 'fastlane/test_output/*.xml'
                    publishHTML(target: [
                        reportDir: 'fastlane/test_output',
                        reportFiles: '*.html',
                        reportName: 'Test Results'
                    ])
                }
            }
        }
        
        stage('Build') {
            when {
                branch 'main'
            }
            steps {
                sh 'bundle exec fastlane build'
            }
        }
        
        stage('Deploy to TestFlight') {
            when {
                branch 'main'
            }
            steps {
                sh 'bundle exec fastlane beta'
            }
        }
    }
    
    post {
        success {
            slackSend(
                color: 'good',
                message: "Build succeeded: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            )
        }
        failure {
            slackSend(
                color: 'danger',
                message: "Build failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            )
        }
    }
}
```

---

## Code Signing Automation

Code signing เป็นหนึ่งในส่วนที่ซับซ้อนที่สุดของ iOS CI/CD

### ประเภทของ Code Signing

```
1. Development     → ใช้ test บน physical devices
2. Ad Hoc          → ใช้ distribute ให้ specific devices
3. App Store       → ใช้ submit ไป App Store
4. Enterprise      → ใช้ distribute internally ใน organization
```

### วิธีการ Code Signing ใน CI

#### วิธีที่ 1: ใช้ match (แนะนำ)

```ruby
# Fastfile
lane :setup_certificates do
  match(
    type: "appstore",
    readonly: true,  # ใน CI ควรเป็น readonly
    git_url: ENV["MATCH_GIT_URL"],
    git_branch: "main",
    app_identifier: ENV["APP_BUNDLE_ID"],
    team_id: ENV["TEAM_ID"]
  )
end
```

#### วิธีที่ 2: Manual Certificate Import

```bash
#!/bin/bash
# Import certificate จาก base64-encoded string ใน secret

# Decode certificate
echo "$CERTIFICATE_BASE64" | base64 --decode > certificate.p12

# Import ไป keychain
security create-keychain -p "$KEYCHAIN_PASSWORD" build.keychain
security default-keychain -s build.keychain
security unlock-keychain -p "$KEYCHAIN_PASSWORD" build.keychain
security import certificate.p12 \
    -k build.keychain \
    -P "$CERTIFICATE_PASSWORD" \
    -T /usr/bin/codesign

# Allow codesign to access keychain
security set-key-partition-list \
    -S apple-tool:,apple: \
    -s \
    -k "$KEYCHAIN_PASSWORD" \
    build.keychain

# Install provisioning profile
mkdir -p ~/Library/MobileDevice/Provisioning\ Profiles
echo "$PROVISIONING_PROFILE_BASE64" | base64 --decode > profile.mobileprovision
cp profile.mobileprovision ~/Library/MobileDevice/Provisioning\ Profiles/
```

#### วิธีที่ 3: App Store Connect API Key

```ruby
# fastlane/Fastfile
lane :upload_with_api_key do
  # ใช้ API Key แทน Apple ID password
  api_key = app_store_connect_api_key(
    key_id: ENV["ASC_KEY_ID"],
    issuer_id: ENV["ASC_ISSUER_ID"],
    key_content: ENV["ASC_KEY_CONTENT"],
    is_key_content_base64: true
  )
  
  upload_to_testflight(
    api_key: api_key,
    skip_waiting_for_build_processing: true
  )
end
```

---

## Simulator Testing ใน CI

### การตั้งค่า Simulators สำหรับ CI

```bash
# ดูรายการ simulators ที่มีอยู่
xcrun simctl list devices

# สร้าง simulator ใหม่
xcrun simctl create "iPhone 15 CI" \
    "com.apple.CoreSimulator.SimDeviceType.iPhone-15" \
    "com.apple.CoreSimulator.SimRuntime.iOS-17-0"

# Boot simulator
xcrun simctl boot "iPhone 15 CI"

# Shutdown simulator
xcrun simctl shutdown "iPhone 15 CI"

# ลบ simulator
xcrun simctl delete "iPhone 15 CI"
```

### xcodebuild สำหรับ Testing

```bash
# Run tests บน specific simulator
xcodebuild test \
    -workspace MyApp.xcworkspace \
    -scheme MyApp \
    -destination 'platform=iOS Simulator,name=iPhone 15,OS=17.0' \
    -resultBundlePath ./TestResults.xcresult \
    CODE_SIGNING_ALLOWED=NO \
    -enableCodeCoverage YES

# Run ด้วย parallel testing
xcodebuild test \
    -workspace MyApp.xcworkspace \
    -scheme MyApp \
    -destination 'platform=iOS Simulator,name=iPhone 15,OS=17.0' \
    -parallel-testing-enabled YES \
    -maximum-parallel-testing-workers 4 \
    CODE_SIGNING_ALLOWED=NO
```

### Fastlane scan configuration

```ruby
# fastlane/Scanfile
scheme("MyApp")
devices(["iPhone 15", "iPad Pro (12.9-inch) (6th generation)"])
code_coverage(true)
output_directory("fastlane/test_output")
output_types("html,junit")
clean(false)
skip_slack(true)
```

---

## Parallel Testing

Parallel testing ช่วยลดเวลาในการ run tests อย่างมาก

### การตั้งค่า Parallel Testing ใน Xcode

```swift
// ใน test scheme, เปิด "Execute in parallel on Simulator"
// หรือกำหนดผ่าน xcodebuild

// Test class ที่รองรับ parallel testing
import XCTest

class NetworkTests: XCTestCase {
    
    // ไม่ควรมี shared state ระหว่าง tests
    override func setUp() {
        super.setUp()
        // สร้าง fresh state สำหรับแต่ละ test
    }
    
    func testAPICall1() async throws {
        // test ที่ไม่ depend on other tests
        let result = try await APIClient.shared.fetchUser(id: 1)
        XCTAssertNotNil(result)
    }
    
    func testAPICall2() async throws {
        let result = try await APIClient.shared.fetchPosts(userID: 1)
        XCTAssertFalse(result.isEmpty)
    }
}
```

### GitHub Actions Parallel Testing

```yaml
jobs:
  test:
    strategy:
      matrix:
        test-targets:
          - 'MyAppUnitTests'
          - 'MyAppIntegrationTests'
          - 'MyAppUITests'
    
    runs-on: macos-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Run ${{ matrix.test-targets }}
      run: |
        xcodebuild test \
          -scheme MyApp \
          -testPlan ${{ matrix.test-targets }} \
          -destination 'platform=iOS Simulator,name=iPhone 15'
```

---

## Test Result Reporting

### xcresult Parser

```bash
# แปลง xcresult เป็น JUnit XML
xcrun xcresulttool get \
    --format json \
    --path TestResults.xcresult > results.json

# ใช้ tool อื่นแปลงเป็น JUnit
# gem install xcpretty
xcodebuild test \
    -scheme MyApp \
    -destination '...' \
    | xcpretty --report junit --output junit.xml
```

### Fastlane Test Result Upload

```ruby
lane :test_and_report do
  run_tests(
    scheme: "MyApp",
    output_directory: "test_output",
    output_types: "html,junit"
  )
  
  # Upload ไป GitHub Actions
  if ENV["GITHUB_ACTIONS"]
    # GitHub Actions จะ pick up JUnit XML อัตโนมัติ
    sh "cp test_output/*.xml $GITHUB_WORKSPACE/"
  end
  
  # หรือ upload ไป Slack
  slack(
    message: "Tests passed! ✅",
    payload: {
      "Tests Passed": lane_context[SharedValues::NUMBER_OF_TESTS],
      "Tests Failed": lane_context[SharedValues::NUMBER_OF_FAILING_TESTS]
    }
  )
end
```

---

## Code Coverage ใน CI

### การเก็บ Code Coverage

```ruby
# Fastfile
lane :coverage do
  run_tests(
    scheme: "MyApp",
    code_coverage: true,
    output_directory: "coverage_output"
  )
  
  # Generate coverage report
  sh "xcrun xccov view --report --json TestResults.xcresult > coverage.json"
  
  # Check minimum coverage threshold
  coverage = JSON.parse(File.read("coverage.json"))
  coverage_percent = coverage["lineCoverage"] * 100
  
  if coverage_percent < 80
    UI.user_error!("Code coverage #{coverage_percent.round(2)}% is below minimum 80%")
  end
  
  UI.success("Code coverage: #{coverage_percent.round(2)}%")
end
```

### GitHub Actions + Codecov

```yaml
- name: Run Tests with Coverage
  run: |
    xcodebuild test \
      -scheme MyApp \
      -destination 'platform=iOS Simulator,name=iPhone 15' \
      -enableCodeCoverage YES \
      -resultBundlePath coverage.xcresult

- name: Generate Coverage Report
  run: |
    xcrun xccov view \
      --report \
      --json coverage.xcresult > coverage.json

- name: Upload to Codecov
  uses: codecov/codecov-action@v3
  with:
    files: ./coverage.json
    fail_ci_if_error: true
    verbose: true
```

---

## Slack Notifications จาก CI

### Fastlane Slack Integration

```ruby
# Fastfile
def notify_slack(message:, success:, additional_info: {})
  slack(
    message: message,
    success: success,
    slack_url: ENV["SLACK_WEBHOOK_URL"],
    channel: "#ios-ci",
    
    # ข้อมูลเพิ่มเติม
    payload: {
      "App Version": get_version_number,
      "Build Number": get_build_number,
      "Branch": ENV["CI_BRANCH"] || `git rev-parse --abbrev-ref HEAD`.strip,
      "Commit": `git log --format="%h %s" -n 1`.strip
    }.merge(additional_info),
    
    # ปุ่ม action
    attachment_properties: {
      thumb_url: "https://example.com/app-icon.png",
      footer: "iOS CI/CD Pipeline",
      footer_icon: "https://example.com/ci-icon.png"
    },
    
    default_payloads: [:git_branch, :lane, :test_result]
  )
end

lane :beta do
  # ...build steps...
  
  notify_slack(
    message: "🚀 New beta build deployed to TestFlight!",
    success: true,
    additional_info: {
      "Environment" => "Staging",
      "TestFlight Group" => "Beta Testers"
    }
  )
end

error do |lane, exception|
  notify_slack(
    message: "❌ Build failed in lane: #{lane}",
    success: false,
    additional_info: {
      "Error" => exception.message
    }
  )
end
```

### GitHub Actions Slack Notification

```yaml
- name: Notify Slack on Success
  if: success()
  uses: slackapi/slack-github-action@v1.24.0
  with:
    payload: |
      {
        "channel": "#ios-ci",
        "attachments": [
          {
            "color": "#36a64f",
            "title": "✅ iOS Build Succeeded",
            "fields": [
              {
                "title": "Branch",
                "value": "${{ github.ref_name }}",
                "short": true
              },
              {
                "title": "Commit",
                "value": "${{ github.sha }}",
                "short": true
              }
            ]
          }
        ]
      }
  env:
    SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}

- name: Notify Slack on Failure
  if: failure()
  uses: slackapi/slack-github-action@v1.24.0
  with:
    payload: |
      {
        "channel": "#ios-ci",
        "attachments": [
          {
            "color": "#ff0000",
            "title": "❌ iOS Build Failed",
            "text": "Build failed on branch: ${{ github.ref_name }}"
          }
        ]
      }
  env:
    SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
```

---

## Version Bumping Automation

### Semantic Versioning

```
Major.Minor.Patch
  1  .  2  .  3

Major: breaking changes
Minor: new features (backward compatible)
Patch: bug fixes
```

### Fastlane Version Bumping

```ruby
# Fastfile
lane :bump_version do |options|
  # ประเภทของ bump: patch, minor, major
  bump_type = options[:type] || "patch"
  
  # ดึง current version
  current_version = get_version_number(xcodeproj: "MyApp.xcodeproj")
  
  # คำนวณ new version
  version_parts = current_version.split(".")
  case bump_type
  when "major"
    version_parts[0] = (version_parts[0].to_i + 1).to_s
    version_parts[1] = "0"
    version_parts[2] = "0"
  when "minor"
    version_parts[1] = (version_parts[1].to_i + 1).to_s
    version_parts[2] = "0"
  when "patch"
    version_parts[2] = (version_parts[2].to_i + 1).to_s
  end
  
  new_version = version_parts.join(".")
  
  # อัปเดต version ใน Xcode project
  increment_version_number(
    version_number: new_version,
    xcodeproj: "MyApp.xcodeproj"
  )
  
  # อัปเดต build number
  increment_build_number(
    build_number: Time.now.strftime("%Y%m%d%H%M"),
    xcodeproj: "MyApp.xcodeproj"
  )
  
  # Commit การเปลี่ยนแปลง
  git_commit(
    path: ["MyApp.xcodeproj/project.pbxproj"],
    message: "Bump version to #{new_version}"
  )
  
  # สร้าง tag
  add_git_tag(tag: "v#{new_version}")
  push_to_git_remote
end
```

### Script สำหรับ Auto Version Bump

```bash
#!/bin/bash
# scripts/bump_version.sh

# ดึง latest tag
LATEST_TAG=$(git describe --tags --abbrev=0 2>/dev/null || echo "v0.0.0")

# แยก version components
VERSION=${LATEST_TAG#v}
MAJOR=$(echo $VERSION | cut -d. -f1)
MINOR=$(echo $VERSION | cut -d. -f2)
PATCH=$(echo $VERSION | cut -d. -f3)

# Increment patch version
NEW_PATCH=$((PATCH + 1))
NEW_VERSION="$MAJOR.$MINOR.$NEW_PATCH"

echo "Bumping version from $VERSION to $NEW_VERSION"

# อัปเดต Info.plist
/usr/libexec/PlistBuddy -c "Set :CFBundleShortVersionString $NEW_VERSION" \
    "$PROJECT_DIR/$TARGET_NAME/Info.plist"

# อัปเดต build number เป็น timestamp
BUILD_NUMBER=$(date +%Y%m%d%H%M)
/usr/libexec/PlistBuddy -c "Set :CFBundleVersion $BUILD_NUMBER" \
    "$PROJECT_DIR/$TARGET_NAME/Info.plist"

echo "Version: $NEW_VERSION, Build: $BUILD_NUMBER"
```

---

## App Store Deployment Automation

### Full Deployment Lane

```ruby
# Fastfile

lane :deploy_to_appstore do
  # 1. ตรวจสอบ branch
  ensure_git_branch(branch: 'main')
  ensure_git_status_clean
  
  # 2. Run tests
  run_tests(scheme: "MyApp")
  
  # 3. Bump version
  version = increment_version_number(bump_type: "minor")
  build_number = increment_build_number(
    build_number: latest_testflight_build_number + 1
  )
  
  # 4. Sign app
  match(type: "appstore", readonly: true)
  
  # 5. Build app
  build_app(
    scheme: "MyApp",
    export_method: "app-store",
    include_bitcode: false
  )
  
  # 6. Upload metadata (optional)
  upload_to_app_store(
    skip_binary_upload: true,
    skip_screenshots: true,
    reject_if_possible: true
  ) if ENV["UPDATE_METADATA"] == "true"
  
  # 7. Upload to App Store
  upload_to_app_store(
    submit_for_review: true,
    automatic_release: false,
    skip_metadata: true,
    skip_screenshots: true,
    
    # Phased release
    phased_release: true,
    
    submission_information: {
      add_id_info_uses_idfa: false,
      export_compliance_uses_encryption: false,
      content_rights_has_third_party_content: false
    }
  )
  
  # 8. Commit version bump
  git_commit(
    path: ".",
    message: "Release v#{version} (#{build_number})"
  )
  
  add_git_tag(tag: "v#{version}")
  push_to_git_remote
  
  # 9. แจ้งเตือน
  slack(
    message: "🎉 v#{version} submitted to App Store!",
    success: true
  )
end
```

---

## .xcconfig Files

**.xcconfig** ไฟล์ช่วยแยกการตั้งค่า build settings ออกจาก Xcode project

### โครงสร้าง xcconfig

```
Configs/
├── Base.xcconfig           # settings ที่ใช้ร่วมกัน
├── Debug.xcconfig          # Debug-specific settings
├── Release.xcconfig        # Release-specific settings
├── Staging.xcconfig        # Staging-specific settings
└── Test.xcconfig           # Test-specific settings
```

### Base.xcconfig

```
// Configs/Base.xcconfig

// App Info
PRODUCT_BUNDLE_IDENTIFIER = com.example.myapp
PRODUCT_NAME = MyApp
MARKETING_VERSION = 1.0.0

// Swift
SWIFT_VERSION = 5.9
SWIFT_OPTIMIZATION_LEVEL = -Onone

// Deployment Target
IPHONEOS_DEPLOYMENT_TARGET = 16.0

// Code Signing
CODE_SIGN_STYLE = Manual
DEVELOPMENT_TEAM = ABCDE12345

// Architectures
VALID_ARCHS = arm64 x86_64
EXCLUDED_ARCHS[sdk=iphonesimulator*] = arm64

// Other
ENABLE_BITCODE = NO
```

### Debug.xcconfig

```
// Configs/Debug.xcconfig
#include "Base.xcconfig"

// Debug-specific overrides
SWIFT_OPTIMIZATION_LEVEL = -Onone
SWIFT_ACTIVE_COMPILATION_CONDITIONS = DEBUG
DEBUG_INFORMATION_FORMAT = dwarf

// Development signing
CODE_SIGN_IDENTITY = iPhone Developer
PROVISIONING_PROFILE_SPECIFIER = match Development com.example.myapp

// API configuration
API_BASE_URL = https://api-dev.example.com
```

### Release.xcconfig

```
// Configs/Release.xcconfig
#include "Base.xcconfig"

// Release optimizations
SWIFT_OPTIMIZATION_LEVEL = -O
SWIFT_ACTIVE_COMPILATION_CONDITIONS = 
DEBUG_INFORMATION_FORMAT = dwarf-with-dsym

// App Store signing
CODE_SIGN_IDENTITY = iPhone Distribution
PROVISIONING_PROFILE_SPECIFIER = match AppStore com.example.myapp

// API configuration
API_BASE_URL = https://api.example.com

// Strip symbols
STRIP_INSTALLED_PRODUCT = YES
```

### Staging.xcconfig

```
// Configs/Staging.xcconfig
#include "Release.xcconfig"

// Override bundle ID for staging
PRODUCT_BUNDLE_IDENTIFIER = com.example.myapp.staging
PRODUCT_NAME = MyApp (Staging)

// Staging API
API_BASE_URL = https://api-staging.example.com

// Staging signing
PROVISIONING_PROFILE_SPECIFIER = match AppStore com.example.myapp.staging
```

### การใช้ xcconfig ใน Swift Code

```swift
// AppConfiguration.swift
import Foundation

struct AppConfiguration {
    static let apiBaseURL: String = {
        guard let urlString = Bundle.main.infoDictionary?["API_BASE_URL"] as? String else {
            fatalError("API_BASE_URL not configured")
        }
        return urlString
    }()
    
    static let isDebug: Bool = {
        #if DEBUG
        return true
        #else
        return false
        #endif
    }()
}

// การใช้งาน
let url = URL(string: AppConfiguration.apiBaseURL)!
print("Connecting to: \(AppConfiguration.apiBaseURL)")
```

---

## Build Configurations

### Debug, Release, Staging

```
Xcode Build Configurations:
├── Debug     → development, testing
├── Release   → App Store submission  
└── Staging   → QA testing, TestFlight
```

### การเพิ่ม Build Configuration ใน Xcode

1. Project → Info → Configurations
2. กด "+" → Duplicate "Release"
3. ตั้งชื่อ "Staging"
4. กำหนด .xcconfig file สำหรับแต่ละ configuration

### Compiler Flags

```swift
// การใช้ #if สำหรับ conditional compilation

#if DEBUG
    let logLevel = LogLevel.verbose
    let environment = "Development"
#elseif STAGING
    let logLevel = LogLevel.info
    let environment = "Staging"
#else
    let logLevel = LogLevel.error
    let environment = "Production"
#endif

// Feature flags
#if ENABLE_BETA_FEATURES
    showBetaFeatures()
#endif
```

### Scheme สำหรับแต่ละ Environment

```
Schemes:
├── MyApp           → Debug configuration
├── MyApp-Staging   → Staging configuration
├── MyApp-Release   → Release configuration
└── MyApp-Tests     → Test configuration
```

---

## Environment Variables ใน CI

### GitHub Actions Environment Variables

```yaml
# ตัวแปร default ที่ GitHub Actions ให้มา
GITHUB_WORKFLOW    # ชื่อ workflow
GITHUB_RUN_ID      # ID ของ run นี้
GITHUB_RUN_NUMBER  # ลำดับที่ของ run
GITHUB_JOB         # ชื่อ job
GITHUB_ACTION      # ชื่อ action
GITHUB_ACTOR       # username ที่ trigger workflow
GITHUB_REPOSITORY  # owner/repo
GITHUB_EVENT_NAME  # event ที่ trigger (push, pull_request, etc.)
GITHUB_SHA         # commit SHA
GITHUB_REF         # branch/tag ref
GITHUB_REF_NAME    # ชื่อ branch/tag

# การใช้งาน
- name: Print branch name
  run: echo "Branch: $GITHUB_REF_NAME"

# กำหนดตัวแปรเอง
env:
  MY_VARIABLE: "my_value"
  BUILD_NUMBER: ${{ github.run_number }}
```

### การส่ง Environment Variables ไปยัง Fastlane

```yaml
- name: Run Fastlane
  env:
    MATCH_PASSWORD: ${{ secrets.MATCH_PASSWORD }}
    APP_STORE_CONNECT_API_KEY_ID: ${{ secrets.ASC_KEY_ID }}
    SLACK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
  run: bundle exec fastlane beta
```

```ruby
# Fastfile - การอ่าน environment variables
lane :beta do
  # อ่าน API key จาก env
  api_key = app_store_connect_api_key(
    key_id: ENV["APP_STORE_CONNECT_API_KEY_ID"],
    issuer_id: ENV["APP_STORE_CONNECT_API_ISSUER_ID"],
    key_content: ENV["APP_STORE_CONNECT_API_KEY_CONTENT"]
  )
end
```

---

## Secrets Management

### GitHub Actions Secrets

```yaml
# การเพิ่ม secret ผ่าน GitHub CLI
gh secret set MATCH_PASSWORD --body "your-password"
gh secret set APP_STORE_API_KEY --body-file path/to/key.json

# การใช้ secret ใน workflow
steps:
- name: Use secret
  env:
    MY_SECRET: ${{ secrets.MY_SECRET }}
  run: echo "Secret length: ${#MY_SECRET}"  # ไม่ print ค่าตรงๆ!
```

### Environment-based Secrets

```yaml
# กำหนด environment-specific secrets
environments:
  production:
    required_reviewers:
      - username
    secrets:
      - PRODUCTION_API_KEY

jobs:
  deploy:
    environment: production
    steps:
    - run: echo ${{ secrets.PRODUCTION_API_KEY }}
```

### การจัดการ Certificates อย่างปลอดภัย

```bash
# Encode certificate เป็น base64 สำหรับ CI secret
base64 -i certificate.p12 | pbcopy

# Decode ใน CI
echo "$CERTIFICATE_BASE64" | base64 --decode > certificate.p12
```

```ruby
# fastlane/actions/setup_signing.rb
module Fastlane
  module Actions
    class SetupSigningAction < Action
      def self.run(params)
        # สร้าง temporary keychain
        keychain_name = "CI_Keychain"
        keychain_password = SecureRandom.hex
        
        create_keychain(
          name: keychain_name,
          password: keychain_password,
          default_keychain: true,
          unlock: true,
          timeout: 3600,
          add_to_search_list: true
        )
        
        # Import certificate
        import_certificate(
          certificate_path: "certificate.p12",
          certificate_password: ENV["CERTIFICATE_PASSWORD"],
          keychain_name: keychain_name,
          keychain_password: keychain_password
        )
      end
    end
  end
end
```

---

## Practical Exercises

### Exercise 1: สร้าง GitHub Actions Workflow เบื้องต้น

**โจทย์**: สร้าง GitHub Actions workflow ที่:
1. Build app บน push ไป main branch
2. Run unit tests
3. Upload test results

**วิธีทำ**:

```yaml
# .github/workflows/exercise1.yml
name: Exercise 1 - Basic CI

on:
  push:
    branches: [ main ]

jobs:
  build-and-test:
    runs-on: macos-latest
    
    steps:
    - name: Checkout
      uses: actions/checkout@v4
    
    - name: Select Xcode 15
      run: sudo xcode-select -switch /Applications/Xcode_15.0.app
    
    - name: Build
      run: |
        xcodebuild build-for-testing \
          -scheme MyApp \
          -destination 'platform=iOS Simulator,name=iPhone 15' \
          CODE_SIGNING_ALLOWED=NO \
          | xcpretty
    
    - name: Test
      run: |
        xcodebuild test-without-building \
          -scheme MyApp \
          -destination 'platform=iOS Simulator,name=iPhone 15' \
          -resultBundlePath ./TestResults.xcresult \
          CODE_SIGNING_ALLOWED=NO \
          | xcpretty --report junit
    
    - name: Upload Results
      if: always()
      uses: actions/upload-artifact@v3
      with:
        name: test-results
        path: |
          TestResults.xcresult
          build/reports/
```

### Exercise 2: ตั้งค่า Fastlane สำหรับ TestFlight

**โจทย์**: สร้าง Fastlane configuration ที่:
1. Run tests ก่อน build
2. Build และ sign app
3. Upload ไป TestFlight
4. แจ้งเตือน Slack

**วิธีทำ**:

```ruby
# fastlane/Fastfile - Exercise 2

default_platform(:ios)

platform :ios do
  
  before_all do
    # ตรวจสอบว่าอยู่บน CI
    if is_ci
      setup_ci
    end
  end
  
  lane :exercise2_beta do
    
    # Step 1: Ensure clean state
    ensure_git_status_clean unless is_ci
    
    # Step 2: Run tests
    begin
      run_tests(
        scheme: "MyApp",
        devices: ["iPhone 15"],
        clean: true,
        output_directory: "test_output"
      )
    rescue => e
      slack(
        message: "Tests failed: #{e.message}",
        success: false,
        slack_url: ENV["SLACK_WEBHOOK_URL"]
      )
      raise e
    end
    
    # Step 3: Set up code signing
    api_key = app_store_connect_api_key(
      key_id: ENV["ASC_KEY_ID"],
      issuer_id: ENV["ASC_ISSUER_ID"],
      key_content: ENV["ASC_KEY_CONTENT"],
      is_key_content_base64: true
    )
    
    match(
      type: "appstore",
      api_key: api_key,
      readonly: is_ci
    )
    
    # Step 4: Bump build number
    current_build = latest_testflight_build_number(
      api_key: api_key
    )
    increment_build_number(build_number: current_build + 1)
    
    # Step 5: Build
    build_app(
      scheme: "MyApp",
      export_method: "app-store",
      output_directory: "build"
    )
    
    # Step 6: Upload to TestFlight
    upload_to_testflight(
      api_key: api_key,
      changelog: last_git_commit[:message],
      distribute_external: false,
      notify_external_testers: false,
      skip_waiting_for_build_processing: true
    )
    
    # Step 7: Notify success
    slack(
      message: "✅ New build #{get_build_number} uploaded to TestFlight!",
      success: true,
      slack_url: ENV["SLACK_WEBHOOK_URL"],
      payload: {
        "Version" => get_version_number,
        "Build" => get_build_number,
        "Branch" => git_branch,
        "Commit" => last_git_commit[:abbreviated_commit_hash]
      }
    )
  end
end
```

### Exercise 3: Code Signing ด้วย match

**โจทย์**: ตั้งค่า match สำหรับ team ที่มีนักพัฒนา 5 คน

**วิธีทำ**:

```ruby
# fastlane/Matchfile
git_url("git@github.com:your-org/ios-certificates.git")
storage_mode("git")
type("development")
app_identifier([
  "com.example.myapp",
  "com.example.myapp.widget",
  "com.example.myapp.notification-service"
])
username("admin@example.com")
team_id("ABCDE12345")
```

```bash
# สร้าง certificates ครั้งแรก
fastlane match development
fastlane match adhoc
fastlane match appstore

# นักพัฒนาคนใหม่ต้องการ setup
fastlane match development --readonly

# รัน match ใน CI
fastlane match appstore --readonly
```

---

## Setting Up a Complete CI/CD Pipeline

### โครงสร้างโปรเจกต์

```
MyApp/
├── .github/
│   ├── workflows/
│   │   ├── pr-check.yml
│   │   ├── develop-ci.yml
│   │   └── release.yml
│   └── CODEOWNERS
├── fastlane/
│   ├── Appfile
│   ├── Fastfile
│   ├── Matchfile
│   ├── Deliverfile
│   └── Scanfile
├── Configs/
│   ├── Base.xcconfig
│   ├── Debug.xcconfig
│   ├── Release.xcconfig
│   └── Staging.xcconfig
├── scripts/
│   ├── setup.sh
│   └── bump_version.sh
├── Gemfile
├── Gemfile.lock
├── Podfile
├── Podfile.lock
└── MyApp.xcworkspace
```

### Complete PR Check Workflow

```yaml
# .github/workflows/pr-check.yml
name: PR Check

on:
  pull_request:
    branches: [ main, develop ]

jobs:
  lint:
    name: SwiftLint
    runs-on: macos-latest
    steps:
    - uses: actions/checkout@v4
    - name: Run SwiftLint
      run: swiftlint --strict

  build-and-test:
    name: Build and Test
    needs: lint
    runs-on: macos-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Ruby
      uses: ruby/setup-ruby@v1
      with:
        ruby-version: '3.0'
        bundler-cache: true
    
    - name: Cache Pods
      uses: actions/cache@v3
      with:
        path: Pods
        key: ${{ runner.os }}-pods-${{ hashFiles('Podfile.lock') }}
    
    - name: Install dependencies
      run: |
        bundle install
        bundle exec pod install
    
    - name: Build and Test
      run: bundle exec fastlane test
    
    - name: Check Coverage
      run: bundle exec fastlane coverage_check
    
    - name: Upload Results
      if: always()
      uses: actions/upload-artifact@v3
      with:
        name: test-results-${{ github.sha }}
        path: fastlane/test_output/
    
    - name: Comment PR
      if: always()
      uses: actions/github-script@v6
      with:
        script: |
          const fs = require('fs');
          const results = JSON.parse(
            fs.readFileSync('fastlane/test_output/summary.json')
          );
          
          const message = `## Test Results
          - ✅ Passed: ${results.passed}
          - ❌ Failed: ${results.failed}
          - 📊 Coverage: ${results.coverage}%`;
          
          github.rest.issues.createComment({
            issue_number: context.issue.number,
            owner: context.repo.owner,
            repo: context.repo.repo,
            body: message
          });
```

### Complete Release Workflow

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    tags:
      - 'v*.*.*'

jobs:
  validate:
    runs-on: macos-latest
    outputs:
      version: ${{ steps.get_version.outputs.version }}
    steps:
    - uses: actions/checkout@v4
    
    - name: Get version from tag
      id: get_version
      run: echo "version=${GITHUB_REF#refs/tags/v}" >> $GITHUB_OUTPUT
    
    - name: Validate version format
      run: |
        VERSION="${{ steps.get_version.outputs.version }}"
        if ! [[ "$VERSION" =~ ^[0-9]+\.[0-9]+\.[0-9]+$ ]]; then
          echo "Invalid version format: $VERSION"
          exit 1
        fi
  
  test:
    needs: validate
    runs-on: macos-latest
    steps:
    - uses: actions/checkout@v4
    - uses: ruby/setup-ruby@v1
      with:
        ruby-version: '3.0'
        bundler-cache: true
    - run: bundle exec pod install
    - run: bundle exec fastlane test
  
  deploy:
    needs: [validate, test]
    runs-on: macos-latest
    environment: production
    
    steps:
    - uses: actions/checkout@v4
    
    - uses: ruby/setup-ruby@v1
      with:
        ruby-version: '3.0'
        bundler-cache: true
    
    - run: bundle exec pod install
    
    - name: Deploy to App Store
      env:
        MATCH_PASSWORD: ${{ secrets.MATCH_PASSWORD }}
        MATCH_GIT_URL: ${{ secrets.MATCH_GIT_URL }}
        ASC_KEY_ID: ${{ secrets.ASC_KEY_ID }}
        ASC_ISSUER_ID: ${{ secrets.ASC_ISSUER_ID }}
        ASC_KEY_CONTENT: ${{ secrets.ASC_KEY_CONTENT }}
        SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
        APP_VERSION: ${{ needs.validate.outputs.version }}
      run: bundle exec fastlane release version:$APP_VERSION
    
    - name: Create GitHub Release
      uses: actions/create-release@v1
      with:
        tag_name: ${{ github.ref_name }}
        release_name: Release ${{ github.ref_name }}
        draft: false
        prerelease: false
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Fastfile Complete

```ruby
# fastlane/Fastfile - Complete Pipeline

require 'json'

default_platform(:ios)

# Constants
APP_IDENTIFIER = "com.example.myapp"
SCHEME = "MyApp"
XCWORKSPACE = "MyApp.xcworkspace"

platform :ios do
  
  # ============================================
  # BEFORE ALL
  # ============================================
  before_all do |lane|
    if is_ci
      setup_ci(force: true)
    end
  end
  
  # ============================================
  # LANE: test
  # ============================================
  desc "Run tests and generate coverage report"
  lane :test do
    run_tests(
      workspace: XCWORKSPACE,
      scheme: SCHEME,
      devices: ["iPhone 15"],
      clean: true,
      code_coverage: true,
      output_directory: "fastlane/test_output",
      output_types: "html,junit,json-compilation-database",
      formatter: "xcpretty"
    )
    
    # Generate coverage
    xcov(
      workspace: XCWORKSPACE,
      scheme: SCHEME,
      output_directory: "fastlane/coverage",
      minimum_coverage_percentage: 70.0
    )
  end
  
  # ============================================
  # LANE: coverage_check
  # ============================================
  desc "Check if code coverage meets minimum threshold"
  lane :coverage_check do
    result_path = lane_context[SharedValues::SCAN_GENERATED_XCRESULT_PATH]
    
    coverage_json = sh(
      "xcrun xccov view --report --json '#{result_path}'"
    )
    coverage_data = JSON.parse(coverage_json)
    coverage = coverage_data["lineCoverage"] * 100
    
    # สร้าง summary file
    summary = {
      "passed" => lane_context[SharedValues::NUMBER_OF_TESTS] - 
                  lane_context[SharedValues::NUMBER_OF_FAILING_TESTS],
      "failed" => lane_context[SharedValues::NUMBER_OF_FAILING_TESTS],
      "coverage" => coverage.round(2)
    }
    
    File.write("fastlane/test_output/summary.json", summary.to_json)
    
    UI.user_error!("Coverage #{coverage.round(2)}% < 70%") if coverage < 70
    UI.success("Coverage: #{coverage.round(2)}%")
  end
  
  # ============================================
  # LANE: beta
  # ============================================
  desc "Build and distribute to TestFlight"
  lane :beta do
    api_key = get_api_key
    
    match(
      type: "appstore",
      api_key: api_key,
      readonly: is_ci,
      app_identifier: APP_IDENTIFIER
    )
    
    increment_build_number(
      build_number: latest_testflight_build_number(
        api_key: api_key,
        app_identifier: APP_IDENTIFIER
      ) + 1
    )
    
    build_app(
      workspace: XCWORKSPACE,
      scheme: SCHEME,
      export_method: "app-store",
      output_directory: "./build"
    )
    
    upload_to_testflight(
      api_key: api_key,
      app_identifier: APP_IDENTIFIER,
      skip_waiting_for_build_processing: true,
      distribute_external: true,
      groups: ["External Testers"],
      changelog: changelog_from_git_commits(
        between: [last_git_tag, "HEAD"],
        pretty: "- %s",
        match_lightweight_tag: false
      )
    )
    
    notify_success("New beta build #{get_build_number} uploaded!")
  end
  
  # ============================================
  # LANE: release
  # ============================================
  desc "Release to App Store"
  lane :release do |options|
    version = options[:version]
    
    api_key = get_api_key
    
    # ตั้ง version ตาม parameter
    if version
      increment_version_number(version_number: version)
      increment_build_number(build_number: "1")
    end
    
    match(
      type: "appstore",
      api_key: api_key,
      readonly: is_ci
    )
    
    build_app(
      workspace: XCWORKSPACE,
      scheme: SCHEME,
      export_method: "app-store"
    )
    
    upload_to_app_store(
      api_key: api_key,
      app_identifier: APP_IDENTIFIER,
      submit_for_review: true,
      automatic_release: true,
      force: true,
      
      submission_information: {
        add_id_info_uses_idfa: false,
        export_compliance_uses_encryption: false
      }
    )
    
    notify_success("🎉 v#{get_version_number} submitted to App Store!")
  end
  
  # ============================================
  # PRIVATE LANES
  # ============================================
  
  private_lane :get_api_key do
    app_store_connect_api_key(
      key_id: ENV["ASC_KEY_ID"],
      issuer_id: ENV["ASC_ISSUER_ID"],
      key_content: ENV["ASC_KEY_CONTENT"],
      is_key_content_base64: true
    )
  end
  
  private_lane :notify_success do |message|
    if ENV["SLACK_WEBHOOK_URL"]
      slack(
        message: message,
        success: true,
        slack_url: ENV["SLACK_WEBHOOK_URL"],
        payload: {
          "Version" => get_version_number,
          "Build" => get_build_number,
          "Branch" => git_branch
        }
      )
    end
  end
  
  # ============================================
  # ERROR HANDLING
  # ============================================
  error do |lane, exception|
    if ENV["SLACK_WEBHOOK_URL"]
      slack(
        message: "❌ Pipeline failed in lane '#{lane}'",
        success: false,
        slack_url: ENV["SLACK_WEBHOOK_URL"],
        attachment_properties: {
          fields: [
            { title: "Error", value: exception.message, short: false },
            { title: "Lane", value: lane.to_s, short: true },
            { title: "Branch", value: git_branch, short: true }
          ]
        }
      )
    end
  end
  
end
```

---

## สรุป

CI/CD เป็นส่วนสำคัญของการพัฒนา iOS ที่ทันสมัย ในบทนี้เราได้เรียนรู้:

### สิ่งที่ได้เรียนรู้

1. **CI/CD Fundamentals**
   - Continuous Integration คือการ merge และ test โค้ดบ่อยๆ
   - Continuous Delivery คือการเตรียม release ให้พร้อมตลอดเวลา

2. **เครื่องมือ CI/CD**
   - **GitHub Actions**: ดีสำหรับ open source projects และ GitHub users
   - **Xcode Cloud**: Native Apple solution, ง่ายที่สุดสำหรับ iOS
   - **Fastlane**: Open source, เป็น automation tool ที่ยืดหยุ่น
   - **Bitrise**: Managed platform สำหรับ mobile teams
   - **CircleCI / Jenkins**: Enterprise-grade solutions

3. **Code Signing**
   - **match** เป็นวิธีที่แนะนำสำหรับ team management
   - เก็บ certificates ใน encrypted git repository
   - ใช้ readonly mode ใน CI

4. **Testing ใน CI**
   - Parallel testing ลดเวลา
   - Code coverage monitoring
   - Test result reporting

5. **Deployment Automation**
   - TestFlight distribution ผ่าน Fastlane pilot
   - App Store submission อัตโนมัติ
   - Version bumping อัตโนมัติ

### Best Practices

```
✅ ทำ CI บน every push/PR
✅ ตั้ง minimum code coverage threshold
✅ ใช้ match สำหรับ code signing
✅ เก็บ secrets ใน CI secrets manager
✅ ใช้ .xcconfig แยก configuration
✅ Notify team ผ่าน Slack
✅ Tag release ใน git
✅ ใช้ App Store Connect API Key (ไม่ใช้ password)
```

### Anti-Patterns

```
❌ commit certificates/private keys ลง git
❌ hardcode API keys ใน source code
❌ skip tests ใน CI เพื่อให้เร็วขึ้น
❌ ใช้ password แทน API key สำหรับ App Store Connect
❌ ignore CI failures และ force merge
```

---

*Part 63 จบ - ไปต่อ Part 64: Core Graphics Drawing*
