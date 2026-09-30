# Part 87: iOS Team Leadership และการจัดการทีม

## บทนำ

การเป็น iOS Tech Lead หรือ Engineering Manager ไม่ใช่แค่การเขียนโค้ดเก่ง แต่ต้องพัฒนาทักษะใหม่ที่แตกต่างออกไปอย่างสิ้นเชิง บทนี้จะพาคุณผ่านทุกมิติของการเป็นผู้นำทีม iOS ตั้งแต่การตัดสินใจด้าน Architecture ไปจนถึงการ Mentor นักพัฒนาหน้าใหม่

---

## 1. การเปลี่ยนผ่านจาก Engineer สู่ Tech Lead

### 1.1 ความแตกต่างระหว่าง IC, Manager, และ Tech Lead

ก่อนอื่นต้องเข้าใจว่า Role เหล่านี้แตกต่างกันอย่างไร:

**Individual Contributor (IC)**
- เขียนโค้ดและส่งมอบ Feature โดยตรง
- Scope of impact อยู่ที่ตัวเอง
- วัดผลจาก Output ของตัวเอง
- Career Path: Junior → Senior → Staff → Principal

**Tech Lead**
- ยังเขียนโค้ดอยู่ แต่ใช้เวลาน้อยลง (~50-60%)
- กำหนดทิศทางทางเทคนิคให้ทีม
- เป็น Multiplier สำหรับ Engineer คนอื่น
- ไม่ได้มี Direct Reports แต่มี Technical Influence
- วัดผลจาก Output ของทีม

**Engineering Manager**
- บริหารคน ไม่ใช่เทคนิคเป็นหลัก
- ดูแล Career Growth, Performance, Hiring
- ทำหน้าที่เป็น Shield ให้ทีมจาก Organizational Noise
- วัดผลจาก Team Health และ Delivery

```
┌─────────────────────────────────────────────────────┐
│                    Career Paths                      │
│                                                      │
│  IC Track:                                           │
│  Junior → Mid → Senior → Staff → Principal → Fellow │
│                                                      │
│  Management Track:                                   │
│  Senior → Tech Lead → EM → Director → VP → CTO      │
│                                                      │
│  Dual IC/Lead Track (Many companies):                │
│  Senior → Staff (Tech Lead) → Principal              │
└─────────────────────────────────────────────────────┘
```

### 1.2 Technical Credibility ขณะเป็น Lead

ปัญหาที่พบบ่อยที่สุดของ Tech Lead ใหม่คือการสูญเสีย Technical Credibility เมื่อหยุดเขียนโค้ด

**กลยุทธ์รักษา Credibility:**

```swift
// ตัวอย่าง: Tech Lead ควร own งานเทคนิคที่สำคัญ
// เช่น การออกแบบ Core Architecture ใหม่

// ❌ ผิด: สั่งให้คนอื่นทำทุกอย่างโดยไม่ลงมือเอง
class TechLead {
    func planArchitecture() {
        // แค่เขียน doc แล้วส่งให้ทีมทำ
        createDocument()
        assignToTeam()
    }
}

// ✅ ถูก: ลงมือทำ Proof of Concept เอง
class TechLead {
    func planArchitecture() {
        // สร้าง PoC เองก่อน
        let poc = createProofOfConcept()
        
        // Document decisions ที่เรียนรู้มา
        let adr = writeADR(from: poc)
        
        // นำเสนอแนวทางให้ทีม และขอ Feedback
        presentToTeam(poc, adr)
        
        // Delegate implementation ให้ทีม โดย Stay engaged
        delegateWithContext()
    }
    
    private func createProofOfConcept() -> POC {
        // ลงมือเขียนโค้ดจริงๆ แม้จะไม่ Production-ready
        // เพื่อ validate แนวคิด
        return POC()
    }
}
```

**Time allocation สำหรับ Tech Lead:**

| Activity | % เวลา |
|----------|--------|
| Coding (Core/Critical features) | 40-50% |
| Architecture & Design | 15-20% |
| Code Reviews | 10-15% |
| Meetings & Planning | 10-15% |
| Mentoring | 10-15% |
| Documentation | 5-10% |

### 1.3 ความผิดพลาดที่ Tech Lead ใหม่มักทำ

**ความผิดพลาด #1: Coding Too Much**
```
ปัญหา: Tech Lead ยังคงเป็น Top Contributor ในทีม
ผลที่ตามมา: ทีมไม่ได้รับ Context ทางเทคนิค, 
            Bottleneck อยู่ที่ Lead, 
            ทีมไม่เติบโต
            
แนวทางแก้ไข: ค่อยๆ ลด coding ลง 10% ต่อเดือน
             แล้วเพิ่มเวลาให้ Mentoring และ Architecture
```

**ความผิดพลาด #2: Avoiding Conflict**
```swift
// ❌ ผิด: ยอมทุกอย่างเพื่อไม่ให้เกิด Conflict
func reviewPR(_ pr: PullRequest) -> Review {
    // "It looks good to me" ทั้งๆ ที่มีปัญหา
    return .approve(comment: "LGTM!")
}

// ✅ ถูก: ให้ Feedback ที่ Constructive และตรงประเด็น
func reviewPR(_ pr: PullRequest) -> Review {
    let issues = findIssues(in: pr)
    let feedback = issues.map { createConstructiveFeedback($0) }
    
    if feedback.isEmpty {
        return .approve(comment: "Great implementation! 🚀")
    }
    
    return .requestChanges(
        comments: feedback,
        // เน้นเรื่องโค้ด ไม่ใช่ตัวบุคคล
        tone: .collaborative
    )
}
```

**ความผิดพลาด #3: Hero Syndrome**
```
ปัญหา: "ฉันทำเองได้เร็วกว่า ดีกว่า"
ผลที่ตามมา: Burnout, Single Point of Failure, 
            ทีมไม่พัฒนาทักษะ
            
แนวทางแก้ไข: ยอมรับว่าต้องให้ทีมทำ แม้จะช้ากว่า
             ลงทุนกับ Pair Programming
             เชื่อมั่นในทีม
```

**ความผิดพลาด #4: ไม่ Push Back กับ Management**
```
ปัญหา: รับ Commitment ทุกอย่างมาโดยไม่ negotiate
ผลที่ตามมา: ทีม Overload, Quality ลดลง, Morale ต่ำ

แนวทางแก้ไข: เรียนรู้การ negotiate scope/timeline
             ใช้ Data สนับสนุน (velocity, complexity)
             "We can do X, Y, Z by the deadline, 
              or X and Y with full quality. Which matters more?"
```

---

## 2. Technical Leadership Responsibilities

### 2.1 Architecture Decision Making

การตัดสินใจด้าน Architecture เป็นหนึ่งในหน้าที่สำคัญที่สุดของ Tech Lead ต้องทำอย่างมีกระบวนการ ไม่ใช่ตามความรู้สึก

**กรอบการตัดสินใจ Architecture:**

```swift
// ตัวอย่าง: Framework สำหรับ Architecture Decision

struct ArchitectureDecision {
    let context: String           // ทำไมต้องตัดสินใจตอนนี้
    let options: [Option]         // ตัวเลือกที่มี
    let constraints: [Constraint] // ข้อจำกัด
    let qualityAttributes: [QualityAttribute] // สิ่งที่ให้ความสำคัญ
    
    struct Option {
        let name: String
        let description: String
        let pros: [String]
        let cons: [String]
        let effort: EffortLevel
        let risk: RiskLevel
    }
    
    enum QualityAttribute {
        case maintainability
        case performance
        case scalability
        case testability
        case developerExperience
        case timeToMarket
    }
    
    // Method ในการ score ตัวเลือก
    func score(_ option: Option, weights: [QualityAttribute: Double]) -> Double {
        // Weighted scoring matrix
        var totalScore = 0.0
        for (attribute, weight) in weights {
            let attributeScore = evaluateOption(option, for: attribute)
            totalScore += attributeScore * weight
        }
        return totalScore
    }
}
```

**ตัวอย่างจริง: การเลือก Networking Layer**

```swift
// สถานการณ์: ทีมต้องเลือก Networking approach

// ตัวเลือก 1: URLSession โดยตรง
class URLSessionNetworkLayer {
    // ข้อดี: ไม่มี dependency, Apple native
    // ข้อเสีย: ต้องเขียน boilerplate เยอะ
    
    func fetch<T: Codable>(_ endpoint: Endpoint) async throws -> T {
        let (data, response) = try await URLSession.shared.data(for: endpoint.request)
        guard let httpResponse = response as? HTTPURLResponse,
              200...299 ~= httpResponse.statusCode else {
            throw NetworkError.invalidResponse
        }
        return try JSONDecoder().decode(T.self, from: data)
    }
}

// ตัวเลือก 2: Alamofire
// ข้อดี: Feature-rich, Community support
// ข้อเสีย: External dependency, อาจ over-engineer สำหรับ simple apps

// ตัวเลือก 3: Custom wrapper บน URLSession
protocol NetworkClient {
    func request<T: Decodable>(_ endpoint: Endpoint) async throws -> T
}

// Tech Lead ต้อง document ว่าเลือกอะไรและทำไม
```

### 2.2 ADR (Architecture Decision Records) ในทางปฏิบัติ

ADR คือ Document ที่ capture การตัดสินใจ Architecture สำคัญๆ รวมถึง Context และ Consequences

**Template ADR สำหรับ iOS Teams:**

```markdown
# ADR-001: การเลือก State Management Pattern

## Status
Accepted / Proposed / Deprecated / Superseded by ADR-XXX

## Context
แอปของเราเติบโตขึ้น และการจัดการ State เริ่มซับซ้อน
ปัจจุบันใช้ @State และ @ObservableObject แต่มีปัญหา:
- State ถูกส่งผ่าน Views หลายชั้น (Prop Drilling)
- Logic กระจัดกระจาย
- Test ยาก

## Decision
เราจะใช้ TCA (The Composable Architecture) เป็น 
State Management Pattern หลัก

## Consequences
### ดี
- Predictable state
- Excellent testability  
- Clear data flow
- Good for complex features

### ไม่ดี
- Learning curve สูง
- Boilerplate มากขึ้น
- Dependency ต่อ third-party library

## Alternatives Considered
1. Redux-style (ReSwift) - ไม่เลือกเพราะ less Swift-idiomatic
2. MVVM + Combine - พิจารณาแล้ว แต่ยังขาด structure สำหรับ large team
3. Clean Architecture - ดี แต่ overhead เกินไปสำหรับ team ขนาดนี้

## Notes
- ทีมต้อง train TCA ก่อน implement
- เริ่มจาก Feature ใหม่ก่อน
- Review อีกครั้งใน Q3
```

**การ organize ADRs ใน Codebase:**

```
project/
├── docs/
│   └── architecture/
│       └── decisions/
│           ├── 0001-state-management.md
│           ├── 0002-networking-layer.md
│           ├── 0003-dependency-injection.md
│           ├── 0004-testing-strategy.md
│           └── README.md (Index of all ADRs)
```

**Script สำหรับ create ADR ใหม่:**

```bash
#!/bin/bash
# create-adr.sh

NEXT_NUMBER=$(ls docs/architecture/decisions/*.md 2>/dev/null | wc -l)
NEXT_NUMBER=$((NEXT_NUMBER + 1))
PADDED=$(printf "%04d" $NEXT_NUMBER)

TITLE="$1"
FILENAME="docs/architecture/decisions/${PADDED}-$(echo $TITLE | tr ' ' '-' | tr '[:upper:]' '[:lower:]').md"

cat > "$FILENAME" << EOF
# ADR-${PADDED}: $TITLE

## Status
Proposed

## Context
<!-- ทำไมต้องตัดสินใจ? -->

## Decision
<!-- ตัดสินใจอะไร? -->

## Consequences
### ดี
- 

### ไม่ดี
- 

## Alternatives Considered
1. 
EOF

echo "Created: $FILENAME"
```

### 2.3 Technical Roadmap Creation

Tech Lead ต้องสร้าง Technical Roadmap ที่สอดคล้องกับ Business Goals

```swift
// Swift code สำหรับ tracking Technical Roadmap items
// (ใช้ใน internal tools หรือ scripts)

struct TechnicalRoadmapItem {
    let id: String
    let title: String
    let category: Category
    let priority: Priority
    let effort: Effort
    let quarter: String  // "Q1 2025"
    let status: Status
    let dependencies: [String]  // IDs ของ items อื่น
    let businessValue: String
    
    enum Category {
        case infrastructure
        case devExperience
        case performance
        case security
        case technicalDebt
        case newCapability
    }
    
    enum Priority: Int, Comparable {
        case critical = 4
        case high = 3
        case medium = 2
        case low = 1
        
        static func < (lhs: Priority, rhs: Priority) -> Bool {
            return lhs.rawValue < rhs.rawValue
        }
    }
    
    enum Effort: String {
        case small = "S"    // < 1 sprint
        case medium = "M"   // 1-2 sprints
        case large = "L"    // 3-5 sprints
        case extraLarge = "XL" // > 5 sprints
    }
    
    enum Status {
        case backlog
        case planned
        case inProgress
        case completed
        case cancelled
    }
}

// ตัวอย่าง Roadmap สำหรับ iOS Team
let technicalRoadmap: [TechnicalRoadmapItem] = [
    TechnicalRoadmapItem(
        id: "TR-001",
        title: "Migrate to Swift Concurrency (async/await)",
        category: .infrastructure,
        priority: .high,
        effort: .large,
        quarter: "Q1 2025",
        status: .planned,
        dependencies: [],
        businessValue: "ลด crash rate จาก race conditions ~30%"
    ),
    TechnicalRoadmapItem(
        id: "TR-002",
        title: "Implement App Performance Monitoring",
        category: .infrastructure,
        priority: .critical,
        effort: .medium,
        quarter: "Q1 2025",
        status: .inProgress,
        dependencies: [],
        businessValue: "Visibility ใน production performance issues"
    ),
    TechnicalRoadmapItem(
        id: "TR-003",
        title: "Modularize App into Feature Packages",
        category: .devExperience,
        priority: .high,
        effort: .extraLarge,
        quarter: "Q2 2025",
        status: .backlog,
        dependencies: ["TR-001"],
        businessValue: "ลด build time 60%, enable parallel team development"
    )
]
```

### 2.4 Estimating Complexity และ Timelines

การ estimate เป็นทักษะสำคัญที่หลายคนไม่ถนัด

**Estimation Framework:**

```swift
struct FeatureEstimate {
    let feature: String
    let bestCase: Int      // วัน (ทุกอย่างราบรื่น)
    let mostLikely: Int    // วัน (ปกติ)
    let worstCase: Int     // วัน (มีอุปสรรค)
    
    // PERT Estimate: (Best + 4*MostLikely + Worst) / 6
    var pertEstimate: Double {
        return Double(bestCase + 4 * mostLikely + worstCase) / 6.0
    }
    
    // Standard Deviation
    var standardDeviation: Double {
        return Double(worstCase - bestCase) / 6.0
    }
}

// ตัวอย่างการ estimate Feature
let loginFeature = FeatureEstimate(
    feature: "Biometric Login with Face ID",
    bestCase: 3,      // ทุกอย่างใช้ได้เลย
    mostLikely: 5,    // ต้องเขียน unit tests, handle edge cases
    worstCase: 10     // LocalAuthentication API มีปัญหา, ต้องทำ fallback
)

print("PERT Estimate: \(loginFeature.pertEstimate) days")
// PERT Estimate: 5.5 days

// Rule of thumb: เพิ่ม 30-50% สำหรับ Unknown unknowns
let finalEstimate = loginFeature.pertEstimate * 1.4
print("Final Estimate: ~\(Int(finalEstimate)) days")
```

**Complexity Factors ที่ต้องคิดถึง:**

```swift
enum ComplexityFactor {
    case existingCodeQuality(quality: CodeQuality)
    case externalDependencies(count: Int)
    case teamFamiliarity(level: FamiliarityLevel)
    case testingRequirements(level: TestingLevel)
    case uiComplexity(level: UIComplexityLevel)
    case backendIntegration(required: Bool)
    case designSpecCompleteness(percentage: Int)
    
    enum CodeQuality { case good, fair, poor }
    enum FamiliarityLevel { case expert, familiar, learning, unknown }
    enum TestingLevel { case minimal, standard, comprehensive }
    enum UIComplexityLevel { case simple, moderate, complex, custom }
    
    // ค่า multiplier สำหรับแต่ละ factor
    var multiplier: Double {
        switch self {
        case .existingCodeQuality(let q):
            switch q {
            case .good: return 1.0
            case .fair: return 1.3
            case .poor: return 1.8
            }
        case .teamFamiliarity(let l):
            switch l {
            case .expert: return 0.8
            case .familiar: return 1.0
            case .learning: return 1.5
            case .unknown: return 2.0
            }
        case .designSpecCompleteness(let pct):
            return pct < 80 ? 1.4 : 1.0
        default:
            return 1.0
        }
    }
}
```

---

## 3. Code Review ในฐานะ Leader

### 3.1 Setting Code Review Standards

ก่อนอื่น ทีมต้องมี Code Review Standard ที่เป็นลายลักษณ์อักษร

**ตัวอย่าง Code Review Standards Document:**

```markdown
# iOS Team Code Review Standards

## Goals
1. Ensure code quality และ maintainability
2. Knowledge sharing ในทีม  
3. Early bug detection
4. Consistency ใน codebase

## SLA (Service Level Agreement)
- Critical bugs/security: Review ภายใน 4 ชั่วโมง
- Regular PRs: Review ภายใน 1 วันทำการ
- Large PRs (>500 lines): Review ภายใน 2 วัน

## PR Size Guidelines
- Small: < 200 lines (ideal)
- Medium: 200-500 lines (acceptable)
- Large: > 500 lines (ต้อง split ถ้าเป็นไปได้)

## What Reviewers Look For
1. Correctness: ทำงานได้ถูกต้องตาม requirements?
2. Tests: มี test coverage เพียงพอ?
3. Performance: มี obvious performance issues?
4. Security: มี security vulnerabilities?
5. Readability: อ่านแล้วเข้าใจง่ายไหม?

## What Reviewers Don't Block On
- Style preferences (ใช้ SwiftLint handle)
- Minor naming debates
- Personal preferences ที่ไม่กระทบ clarity

## Approval Requirements
- อย่างน้อย 1 approval จาก senior engineer
- Tech Lead approval สำหรับ core changes
- ไม่มี unresolved critical comments
```

### 3.2 PR Review Guidelines สำหรับทีม

**การเขียน PR Description ที่ดี:**

```markdown
# PR Template

## ทำอะไร (What)
<!-- อธิบายการเปลี่ยนแปลงในระดับสูง -->
เพิ่ม Offline Support สำหรับ Product Listing

## ทำไม (Why)
<!-- Context และ Business justification -->
Users ร้องเรียนว่าแอปใช้ไม่ได้เมื่อ internet ช้า
Jira ticket: MOBILE-1234

## วิธีทดสอบ (How to Test)
1. Build แอป
2. เปิด Product List page
3. ปิด internet
4. ดู cached content

## Screenshots/Videos
<!-- สำหรับ UI changes -->
| Before | After |
|--------|-------|
| [img]  | [img] |

## Checklist
- [ ] มี Unit tests
- [ ] มี UI tests (ถ้า relevant)
- [ ] Documentation updated
- [ ] No SwiftLint warnings
- [ ] Tested on iPhone SE (smallest supported)
- [ ] Tested on iPad (ถ้า Universal app)
- [ ] Tested on iOS 16 (minimum supported)
- [ ] Accessibility checked

## Dependencies
<!-- PRs อื่นที่ต้อง merge ก่อน -->
Depends on #456 (Core Data setup)

## Breaking Changes
None / Describe if any
```

### 3.3 Giving Effective Feedback

**หลัก Conventional Comments:**

```
# Format: <label>: <comment>

# Labels:
# nitpick: - เรื่องเล็กน้อย ไม่จำเป็นต้องแก้
# suggestion: - แนะนำ ไม่ block
# issue: - ปัญหาที่ต้องแก้
# question: - ขอให้อธิบาย
# thought: - ความคิดเห็น สำหรับ discussion
# praise: - คำชม

# ตัวอย่างการ comment ที่ดี:

nitpick: ชื่อ variable `d` ไม่ descriptive เท่า `date`

suggestion: อาจจะใช้ `compactMap` แทน `map` + `filter` เพื่อ simplify

issue: มี potential retain cycle ที่นี่ ควรใช้ [weak self] ใน closure

question: ทำไมถึงเลือก 500ms เป็น timeout? มีเหตุผลเฉพาะไหม?

praise: Clean implementation ของ retry logic! 

thought: ถ้าเราใช้ Result type ที่นี่ อาจจะ handle error ได้ชัดเจนกว่า
```

**ตัวอย่าง Code Review Feedback ที่ดีและไม่ดี:**

```swift
// โค้ดที่ถูก Review
func fetchUser(id: String) {
    networkClient.get("/users/\(id)") { result in
        if result.error != nil {
            print("Error!")
        }
        self.user = result.data
    }
}

// ❌ Feedback ไม่ดี:
// "This code is wrong"
// "Why did you write it like this?"
// "This is bad practice"

// ✅ Feedback ที่ดี:
/*
issue: มี 2 ปัญหาที่นี่:

1. Retain cycle: closure capture `self` strongly
   แนะนำใช้ `[weak self]`:
   ```swift
   networkClient.get("/users/\(id)") { [weak self] result in
   ```

2. Error handling ไม่สมบูรณ์: เมื่อมี error, `result.data` อาจจะเป็น nil
   ทำให้ `self.user = result.data` อาจ reset user ที่มีอยู่แล้ว
   
   แนะนำ:
   ```swift
   switch result {
   case .success(let user):
       self?.user = user
   case .failure(let error):
       self?.handleError(error)
   }
   ```
*/
```

### 3.4 Nurturing Junior Developers ผ่าน Code Reviews

**Sandwich Feedback Model:**

```
1. เริ่มด้วยสิ่งที่ดี (Specific praise)
2. ให้ feedback เชิง constructive (เฉพาะเจาะจง)
3. จบด้วย encouragement

ตัวอย่าง:
"โครงสร้างโดยรวมของโค้ดดีมาก, แบ่ง responsibilities ชัดเจน (1)
มีจุดที่ควรปรับ: error handling ใน line 45 ต้องจัดการ edge cases... (2)
พอ fix เรื่องนี้แล้ว PR นี้จะ merge ได้เลย! (3)"
```

**Rubber Duck Reviews สำหรับ Junior Devs:**

```markdown
เมื่อ Review code ของ Junior Developer ควรทำสิ่งเหล่านี้:

1. อธิบายว่าทำไมไม่ใช่แค่ว่าอะไร
   ❌ "ใช้ weak reference ที่นี่"
   ✅ "ใช้ weak reference ที่นี่เพื่อป้องกัน retain cycle - 
      ลอง อ่าน https://... เพื่อ understand memory management"

2. ให้ Resources
   "มีบทความดีๆ เรื่อง Swift Concurrency: [link]"

3. ถามคำถามที่ช่วยให้คิด
   "ถ้า user ปิดหน้านี้ระหว่างที่ fetch อยู่ จะเกิดอะไรขึ้น?"

4. Pair review สำหรับ complex issues
   "มาคุยกันสัก 15 นาทีเรื่อง threading issue นี้ดีกว่า"
```

---

## 4. Engineering Excellence Culture

### 4.1 Definition of Done

**ตัวอย่าง Definition of Done สำหรับ iOS Team:**

```markdown
# Definition of Done (DoD) - iOS Team

## Code Complete
- [ ] Feature ทำงานตาม Acceptance Criteria
- [ ] Code ผ่าน Code Review (≥1 approval)
- [ ] No compiler warnings
- [ ] SwiftLint pass (0 errors, 0 warnings)

## Testing
- [ ] Unit test coverage ≥ 80% สำหรับ business logic
- [ ] UI tests สำหรับ critical user flows
- [ ] Manual testing บน device จริง
- [ ] Tested บน minimum supported iOS version
- [ ] Regression test ผ่าน

## Quality
- [ ] ไม่มี memory leaks (Instruments check)
- [ ] ไม่มี main thread violations
- [ ] Accessibility labels ครบถ้วน
- [ ] Dark mode ทำงานถูกต้อง

## Documentation
- [ ] Code comments สำหรับ complex logic
- [ ] PR description ครบถ้วน
- [ ] CHANGELOG updated (ถ้ามี)
- [ ] API documentation updated (DocC)

## Deployment Ready
- [ ] Feature flag ตั้งค่าถูกต้อง
- [ ] Analytics events ครบ
- [ ] Error tracking setup
- [ ] No debug/temporary code
```

### 4.2 Technical Debt Budget

**การจัดการ Technical Debt แบบมีระบบ:**

```swift
// Framework สำหรับ tracking Technical Debt

struct TechnicalDebtItem {
    let id: String
    let title: String
    let description: String
    let impact: Impact
    let effort: Effort
    let category: Category
    let createdDate: Date
    let estimatedCostPerSprint: Int  // hours
    
    enum Impact {
        case velocity    // ทำให้ team ช้าลง
        case quality     // ทำให้ bug เยอะขึ้น  
        case security    // security risk
        case maintenance // ยาก maintain
        
        var score: Int {
            switch self {
            case .security: return 4
            case .quality: return 3
            case .velocity: return 2
            case .maintenance: return 1
            }
        }
    }
    
    enum Effort: String {
        case quick = "< 1 day"
        case small = "1-3 days"
        case medium = "1-2 weeks"
        case large = "1+ month"
    }
    
    enum Category {
        case codeSmell
        case outdatedDependency
        case poorTestCoverage
        case performanceIssue
        case securityVulnerability
        case poorDocumentation
    }
    
    // Priority score สำหรับ sorting
    var priorityScore: Int {
        return impact.score * estimatedCostPerSprint
    }
}

// กฎ 20% สำหรับ Technical Debt
// ทุก Sprint ควร allocate 20% ของ capacity สำหรับ tech debt

class TechDebtBudget {
    let totalSprintCapacity: Int  // hours
    
    var techDebtBudget: Int {
        return Int(Double(totalSprintCapacity) * 0.20)
    }
    
    // เลือก items ที่ควรทำใน sprint นี้
    func selectItemsForSprint(
        from backlog: [TechnicalDebtItem],
        availableHours: Int
    ) -> [TechnicalDebtItem] {
        let sorted = backlog.sorted { $0.priorityScore > $1.priorityScore }
        var selected: [TechnicalDebtItem] = []
        var remainingHours = availableHours
        
        for item in sorted {
            let itemHours = estimateHours(item)
            if itemHours <= remainingHours {
                selected.append(item)
                remainingHours -= itemHours
            }
        }
        
        return selected
    }
    
    private func estimateHours(_ item: TechnicalDebtItem) -> Int {
        switch item.effort {
        case .quick: return 4
        case .small: return 16
        case .medium: return 60
        case .large: return 160
        }
    }
}
```

### 4.3 Zero-Bug Policy Implementation

```swift
// Zero-Bug Policy: Bug ใหม่ต้อง fix ก่อน feature ใหม่

// Severity Levels
enum BugSeverity: Int, Comparable {
    case p0 = 4  // Production down, data loss - fix within hours
    case p1 = 3  // Core feature broken - fix within 24 hours  
    case p2 = 2  // Important feature degraded - fix within sprint
    case p3 = 1  // Minor issue - fix when time permits
    
    static func < (lhs: BugSeverity, rhs: BugSeverity) -> Bool {
        return lhs.rawValue < rhs.rawValue
    }
}

// Policy Rules
struct ZeroBugPolicy {
    // Rule 1: ห้าม start feature ใหม่ถ้ามี P0/P1 open
    func canStartNewFeature(openBugs: [Bug]) -> Bool {
        let criticalBugs = openBugs.filter { $0.severity >= .p1 }
        return criticalBugs.isEmpty
    }
    
    // Rule 2: Sprint Goal ต้องรวม bug fixing
    func calculateBugBudget(
        sprintCapacity: Int,
        openP2Bugs: Int,
        openP3Bugs: Int
    ) -> Int {
        // ถ้ามี bugs > threshold, เพิ่ม budget
        if openP2Bugs > 5 || openP3Bugs > 20 {
            return Int(Double(sprintCapacity) * 0.40)
        }
        return Int(Double(sprintCapacity) * 0.20)
    }
    
    // Rule 3: Bug ต้อง reproducible ก่อน assign
    func validateBugReport(_ bug: Bug) -> ValidationResult {
        guard bug.stepsToReproduce != nil else {
            return .invalid(reason: "Missing steps to reproduce")
        }
        guard bug.affectedVersions != nil else {
            return .invalid(reason: "Missing affected versions")
        }
        return .valid
    }
}
```

### 4.4 Performance SLOs สำหรับ Mobile Apps

```swift
// SLO = Service Level Objective

struct MobilePerformanceSLO {
    // Launch Time
    static let coldLaunchTarget: TimeInterval = 2.0      // seconds
    static let warmLaunchTarget: TimeInterval = 1.0      // seconds
    
    // UI Responsiveness  
    static let scrollFPS: Int = 60                        // frames per second
    static let tapResponseTime: TimeInterval = 0.1        // 100ms
    static let screenTransitionTime: TimeInterval = 0.3   // 300ms
    
    // Network
    static let apiResponseTimeout: TimeInterval = 10.0
    static let imageLoadingTarget: TimeInterval = 1.5
    
    // Stability
    static let crashFreeRate: Double = 0.999             // 99.9%
    static let anrFreeRate: Double = 0.998               // 99.8%
    
    // Memory
    static let maxMemoryUsage: Int = 200 * 1024 * 1024  // 200MB
    
    // Battery
    static let backgroundTaskMaxDuration: TimeInterval = 30.0
}

// Performance Monitoring Implementation
class PerformanceMonitor {
    private var launchStartTime: Date?
    private var metrics: [String: Double] = [:]
    
    func recordAppLaunch() {
        launchStartTime = Date()
    }
    
    func recordFirstMeaningfulContent() {
        guard let startTime = launchStartTime else { return }
        let launchTime = Date().timeIntervalSince(startTime)
        
        metrics["launch_time"] = launchTime
        
        // Check against SLO
        if launchTime > MobilePerformanceSLO.coldLaunchTarget {
            reportSLOViolation(
                metric: "launch_time",
                value: launchTime,
                target: MobilePerformanceSLO.coldLaunchTarget
            )
        }
        
        // Send to analytics
        Analytics.record(
            event: "app_launch",
            parameters: ["duration": launchTime]
        )
    }
    
    private func reportSLOViolation(
        metric: String,
        value: Double,
        target: Double
    ) {
        let violation = SLOViolation(
            metric: metric,
            actualValue: value,
            targetValue: target,
            timestamp: Date()
        )
        // Send to Datadog/Firebase Performance/etc.
        print("⚠️ SLO Violation: \(metric) = \(value)s (target: \(target)s)")
    }
}

struct SLOViolation {
    let metric: String
    let actualValue: Double
    let targetValue: Double
    let timestamp: Date
}

// Placeholder types
class Analytics {
    static func record(event: String, parameters: [String: Any]) {}
}
```

---

## 5. Team Processes

### 5.1 Sprint Planning สำหรับ iOS Teams

**Sprint Planning Agenda:**

```markdown
# Sprint Planning Template (2 ชั่วโมงสำหรับ 2-week sprint)

## Part 1: Sprint Review (30 min)
- Velocity จาก sprint ที่แล้ว
- Completed stories
- Incomplete stories และสาเหตุ

## Part 2: Product Backlog Review (30 min)
- PM นำเสนอ top priority items
- Tech Lead ตรวจสอบ technical readiness
- ถาม clarifying questions

## Part 3: Capacity Planning (15 min)
- นับ working days (ลบ PTO, meetings)
- ลบ 20% สำหรับ overhead
- คำนวณ available story points

## Part 4: Sprint Backlog Selection (35 min)
- เลือก stories ให้ fit กับ capacity
- Break down complex stories
- ระบุ technical dependencies

## Part 5: Commitment (10 min)
- Team confirm sprint goal
- ระบุ risks
- Action items
```

**Capacity Calculation:**

```swift
struct SprintCapacity {
    let engineers: [Engineer]
    let sprintLengthDays: Int
    let startDate: Date
    let endDate: Date
    
    struct Engineer {
        let name: String
        let ptoDays: Int
        let onCallDays: Int
        let meetingHoursPerDay: Double
        let velocityMultiplier: Double  // 0.7 สำหรับ junior, 1.0 สำหรับ mid, 1.2 สำหรับ senior
    }
    
    // คำนวณ effective capacity
    var effectiveCapacityHours: Double {
        return engineers.reduce(0) { total, engineer in
            let workingDays = Double(sprintLengthDays - engineer.ptoDays - engineer.onCallDays)
            let dailyWorkHours = 8.0 - engineer.meetingHoursPerDay
            let rawHours = workingDays * dailyWorkHours
            
            // 80% efficiency (overhead, slack, unexpected issues)
            let effectiveHours = rawHours * 0.80 * engineer.velocityMultiplier
            
            return total + effectiveHours
        }
    }
    
    // แปลงเป็น Story Points (assumption: 1 SP = 4 hours)
    var estimatedStoryPoints: Int {
        return Int(effectiveCapacityHours / 4.0)
    }
}

// ตัวอย่างการใช้งาน
let sprint = SprintCapacity(
    engineers: [
        .init(name: "Alice", ptoDays: 0, onCallDays: 2, meetingHoursPerDay: 2, velocityMultiplier: 1.2),
        .init(name: "Bob", ptoDays: 2, onCallDays: 0, meetingHoursPerDay: 1.5, velocityMultiplier: 1.0),
        .init(name: "Charlie", ptoDays: 0, onCallDays: 0, meetingHoursPerDay: 1, velocityMultiplier: 0.8)
    ],
    sprintLengthDays: 10,
    startDate: Date(),
    endDate: Calendar.current.date(byAdding: .day, value: 14, to: Date())!
)

print("Sprint Capacity: \(sprint.estimatedStoryPoints) story points")
```

### 5.2 Incident Response สำหรับ Mobile

**Mobile Incident Runbook:**

```markdown
# Mobile Incident Response Runbook

## Severity Levels

### P0 - Critical (Response: 15 min)
- Crash rate > 5% (จากปกติ < 1%)
- Payment/Purchase flow ไม่ทำงาน
- Login ไม่สามารถเข้าได้
- Data loss หรือ Data corruption

### P1 - High (Response: 1 hour)  
- Core feature ทำงานผิดพลาดสำหรับ > 20% users
- Performance degradation ร้องเรียน spike
- Push notification ไม่ส่ง

### P2 - Medium (Response: Next business day)
- Feature เดี่ยวทำงานผิดพลาด
- UI bug ที่ไม่กระทบ functionality

## Response Process

### Step 1: Acknowledge (< 15 min สำหรับ P0)
- Post ใน #incident channel: 
  "Acknowledged P0 incident: [description]. Investigating..."
- Assign Incident Commander

### Step 2: Investigate
Checklist:
- [ ] เปิด Firebase Crashlytics / Bugsnag
- [ ] เช็ค crash rate graph
- [ ] ดู affected app version
- [ ] ดู affected device/OS
- [ ] Review recent deployments

### Step 3: Communicate (every 30 min)
"Status update: [what we know, what we're doing, ETA]"

### Step 4: Mitigate
Options:
1. Force update ผ่าน Remote Config
2. Feature flag disable
3. Server-side fix (ถ้าเป็น API issue)
4. Hotfix build + expedited review

### Step 5: Resolve
- Confirm crash rate กลับสู่ normal
- Post resolution: "Resolved: [root cause, fix applied]"
- Schedule postmortem (48h)
```

### 5.3 Postmortem Writing

```markdown
# Postmortem Template

## Summary
[2-3 ประโยค อธิบาย incident]

## Timeline
| Time (UTC) | Event |
|------------|-------|
| 14:32 | Crash spike detected in Crashlytics |
| 14:45 | On-call engineer acknowledged |
| 15:00 | Root cause identified |
| 15:30 | Fix deployed |
| 16:00 | Crash rate normalized |

## Root Cause Analysis
### Immediate Cause
[อะไรที่ trigger incident โดยตรง]

### Contributing Factors  
1. [Factor 1]
2. [Factor 2]

### 5 Whys
Why did the app crash?
→ Because X was nil
Why was X nil?
→ Because the API returned unexpected response
Why did the API return unexpected response?
→ Because the new API version wasn't backward compatible
Why wasn't backward compatibility maintained?
→ Because there was no contract testing
Why was there no contract testing?
→ Because we don't have a process for it

Root Cause: ไม่มี contract testing

## Impact
- Duration: 1.5 hours
- Affected users: ~15,000 (12% of DAU)
- Crash rate peak: 8.5% (normal: 0.3%)
- Revenue impact: ประมาณ $X,XXX

## Action Items
| Action | Owner | Due Date | Priority |
|--------|-------|----------|----------|
| Add contract testing | Alice | 2025-02-15 | P1 |
| Add monitoring alert for crash spike | Bob | 2025-02-10 | P0 |
| Update runbook | Charlie | 2025-02-08 | P2 |

## What Went Well
- Detection เร็ว (< 15 min)
- Communication ชัดเจน
- Team response รวดเร็ว

## What Could Be Improved
- Alert threshold ควร sensitive กว่านี้
- Need better rollback mechanism

## Lessons Learned
1. API versioning ต้อง explicit
2. Contract tests ควรเป็น standard
```

---

## 6. Hiring iOS Engineers

### 6.1 Crafting Good iOS Interview Questions

**Interview Structure:**

```markdown
# iOS Engineer Interview Process (4 rounds)

## Round 1: Recruiter Screen (30 min)
- Background overview
- Motivation
- Availability/compensation

## Round 2: Technical Phone Screen (45 min)
Topics:
- Swift fundamentals (optionals, protocols, closures)
- iOS frameworks knowledge
- Problem-solving approach
- ไม่ต้องใช้ whiteboard coding

## Round 3: Technical Interview (90 min)
- Coding challenge (realistic, take-home style)
- System design (mobile architecture)
- Code review exercise

## Round 4: Cultural Fit & Team Fit (60 min)
- Behavioral questions (STAR format)
- Meet the team
- Discuss career goals
```

**Technical Questions ที่ดี:**

```swift
// ❌ คำถามที่ไม่ดี: "อธิบาย ARC"
// (ตอบได้จาก memorization ไม่บอกว่า debug เป็นไหม)

// ✅ คำถามที่ดี: Debug the following code

class MessageManager {
    var messages: [Message] = []
    var networkClient: NetworkClient
    
    func loadMessages() {
        networkClient.fetchMessages { messages in
            self.messages = messages  // ❓ ปัญหาคืออะไร?
            self.updateUI()
        }
    }
    
    func updateUI() {
        // Update table view
    }
}

// Expected answer:
// 1. Retain cycle: self captures networkClient, 
//    networkClient's closure captures self
//    Solution: [weak self] ใน closure
// 2. Thread safety: updateUI อาจ call จาก background thread
//    Solution: DispatchQueue.main.async { self?.updateUI() }
```

**System Design Questions สำหรับ iOS:**

```markdown
# ตัวอย่าง System Design Questions

1. "ออกแบบ Instagram Feed ใน iOS app"
   ที่ต้องการเห็น:
   - Data model design
   - Pagination strategy
   - Image caching (NSCache, disk cache)
   - Offline support
   - Infinite scroll implementation
   - Architecture choice (MVVM, MVI)

2. "ออกแบบ Real-time Chat feature"
   ที่ต้องการเห็น:
   - WebSocket vs Long Polling vs APNS
   - Local persistence strategy
   - Message ordering
   - Retry mechanism
   - Read receipts

3. "ออกแบบ Photo Upload System"
   ที่ต้องการเห็น:
   - Background uploads
   - Progress tracking
   - Compression strategy
   - Error handling + retry
   - Permission handling
```

### 6.2 Technical Assessment Design

**Take-Home Assignment Template:**

```markdown
# iOS Take-Home Assignment

## Overview
สร้าง simple iOS app ที่แสดงรายการ GitHub repositories

## Requirements
- Fetch repositories จาก GitHub API
- แสดง list of repositories (name, description, stars)
- Tap เพื่อเปิด detail view
- Support offline reading (cache previous results)

## Technical Requirements
- SwiftUI หรือ UIKit (ตามใจชอบ)
- No 3rd party networking libraries
- Must have unit tests
- README explaining decisions

## Evaluation Criteria (100 points)
- Code quality & readability (25)
- Architecture (20)
- Error handling (15)
- Testing (20)
- UI/UX (10)
- README & documentation (10)

## Time Limit
ประมาณ 3-4 ชั่วโมง (ไม่ต้อง perfect)

## Submission
GitHub repository (public หรือ invite reviewer)
```

### 6.3 Onboarding Checklist

```markdown
# iOS Engineer Onboarding Checklist

## Week 1: Setup & Orientation

### Day 1
- [ ] Hardware setup (Mac, iPhone test devices)
- [ ] Account creation (GitHub, Jira, Slack, etc.)
- [ ] Xcode + dependencies setup
- [ ] First build สำเร็จ
- [ ] Meet the team (buddy assigned)

### Day 2-3
- [ ] Architecture walkthrough กับ Tech Lead
- [ ] Codebase tour
- [ ] Development workflow overview
- [ ] First PR: อ่าน README แล้ว fix typo หรือ add comment

### Day 4-5
- [ ] Shadow pair programming session
- [ ] Review existing PRs เพื่อ understand standards
- [ ] Setup local dev environment ครบ
- [ ] อ่าน ADRs หลัก (5-10 most important)

## Week 2: First Contribution

- [ ] Assign first small bug fix
- [ ] Complete with mentor support
- [ ] Submit PR and receive feedback
- [ ] Attend sprint ceremonies

## Month 1: Getting Productive

- [ ] Complete onboarding project (small feature)
- [ ] Understand release process
- [ ] Know how to use monitoring tools
- [ ] Comfortable with team's testing approach

## Month 3: Independence

- [ ] Working independently on medium features
- [ ] Active participant in code reviews
- [ ] Contributing to technical discussions
- [ ] Identified areas for improvement (tech debt items)
```

---

## 7. Working with Designers และ PMs

### 7.1 Design Handoff Process (Figma to Xcode)

```swift
// ขั้นตอน Design Handoff ที่มีประสิทธิภาพ

// 1. Design Review Meeting (ก่อน Dev เริ่ม)
struct DesignReviewChecklist {
    var allStatesDesigned: Bool          // loading, empty, error, success
    var responsiveDesignSpecified: Bool  // different screen sizes
    var animationsSpecified: Bool        // transitions, micro-interactions
    var accessibilityConsidered: Bool    // labels, minimum touch targets
    var darkModeDesigned: Bool
    var edgeCasesHandled: Bool           // long text, empty state
    
    var isReadyForDev: Bool {
        return allStatesDesigned &&
               responsiveDesignSpecified &&
               accessibilityConsidered
    }
}

// 2. Token Extraction จาก Figma
// ใช้ Figma Tokens plugin หรือ Style Dictionary

// สร้าง Swift Design Tokens จาก Figma
extension Color {
    // Primary
    static let brand = Color("BrandPrimary")
    static let brandSecondary = Color("BrandSecondary")
    
    // Semantic
    static let background = Color("Background")
    static let surface = Color("Surface")
    static let onSurface = Color("OnSurface")
    
    // Status
    static let success = Color("Success")
    static let warning = Color("Warning")
    static let error = Color("Error")
}

extension Font {
    // Typography Scale
    static let displayLarge = Font.system(size: 57, weight: .regular)
    static let displayMedium = Font.system(size: 45, weight: .regular)
    static let headlineLarge = Font.system(size: 32, weight: .bold)
    static let bodyLarge = Font.system(size: 16, weight: .regular)
    static let bodyMedium = Font.system(size: 14, weight: .regular)
    static let labelSmall = Font.system(size: 11, weight: .medium)
}

// 3. Component Implementation
struct ProductCard: View {
    let product: Product
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            // Image
            AsyncImage(url: product.imageURL) { image in
                image
                    .resizable()
                    .aspectRatio(contentMode: .fill)
            } placeholder: {
                Rectangle()
                    .fill(Color.surface)
                    .overlay(
                        ProgressView()
                    )
            }
            .frame(height: 200)
            .clipped()
            .cornerRadius(12)
            
            // Title
            Text(product.name)
                .font(.headlineLarge)
                .foregroundColor(.onSurface)
                .lineLimit(2)
            
            // Price
            Text(product.price.formatted(.currency(code: "THB")))
                .font(.bodyLarge)
                .foregroundColor(.brand)
        }
        .padding(16)
        .background(Color.surface)
        .cornerRadius(16)
        .shadow(color: .black.opacity(0.08), radius: 8, x: 0, y: 2)
    }
}
```

### 7.2 Feature Specification Review

**Checklist ก่อน Engineer เริ่ม:**

```markdown
# Feature Spec Review Checklist (Tech Lead)

## Functional Requirements
- [ ] User stories ชัดเจนมี acceptance criteria
- [ ] Edge cases ระบุครบ (empty states, error states)
- [ ] Permission requirements ระบุ (camera, location, etc.)

## Technical Requirements
- [ ] API endpoints documented
- [ ] Data models defined
- [ ] Performance requirements specified
- [ ] Security requirements (data encryption, auth)

## Design
- [ ] All screens designed (ทุก states)
- [ ] Component library compliance
- [ ] Responsive design specified
- [ ] Animation specs (หรือ ไม่ต้องการ animation)

## Analytics & Tracking
- [ ] Events to track specified
- [ ] Properties for each event
- [ ] Funnel definition

## Release Strategy
- [ ] Feature flag needed?
- [ ] Rollout percentage
- [ ] Kill switch plan

## Questions/Concerns
[Tech lead notes questions ที่ต้อง answer]
```

### 7.3 Pushback with Data

```swift
// ตัวอย่าง: การ pushback request ที่ไม่ realistic

// ❌ วิธีที่ไม่ดี:
// "ทำไม่ได้ครับ"

// ✅ วิธีที่ดี: ใช้ Data + Alternatives

struct FeaturePushback {
    let requestedFeature: String
    let requestedTimeline: String
    
    var analysis: String {
        return """
        Feature: \(requestedFeature)
        
        Requested Timeline: \(requestedTimeline)
        
        Technical Analysis:
        - Complexity: HIGH
        - Required: New API endpoints (2 weeks BE)
        - Required: Design for 5 new screens
        - Required: Core Data migration
        - Estimated effort: 6 weeks
        
        Risks:
        1. Dependency on Backend team availability
        2. App Store review time (typically 3-7 days)
        3. Testing complexity (multiple devices/iOS versions)
        
        Alternatives:
        Option A (4 weeks): MVP version
        - ลด features 50%
        - Defer advanced filters to v2
        
        Option B (6 weeks): Full version
        - ต้องเลื่อน Feature X
        - หรือ เพิ่ม engineer
        
        Option C (2 weeks): Quick win
        - ใช้ existing data + limited UI
        - Shows progress to users
        - Can iterate later
        
        Recommendation: Option A
        - Meets core user need
        - Achievable in 4 weeks
        - Can ship v2 next quarter
        """
    }
}
```

---

## 8. iOS-Specific Team Challenges

### 8.1 App Store Review Delays ใน Sprint Planning

```markdown
# Handling App Store Review ใน Agile Process

## ปัญหา
App Store review ใช้เวลา 1-3 วัน (บางครั้ง longer)
Sprint อาจจบแต่ app ยัง pending review

## Solutions

### 1. Release Buffer
- Plan release 1 sprint ก่อน deadline
- "Sprint N-1": Code complete + submit
- "Sprint N": Monitor review + handle rejections

### 2. Feature Flags
- Submit code with feature disabled
- Enable via remote config หลัง review pass
- เร่งประกาศ release ได้ทันที

### 3. Phased Release
- อย่า release 100% ทันที
- เริ่มที่ 1% → 10% → 100%
- ลด blast radius ถ้า bug หลุด production

### 4. Expedited Review Request
- ใช้สำหรับ critical bug fixes เท่านั้น
- Submit ผ่าน App Store Connect
- Apple พิจารณาตาม case-by-case

## Sprint Ceremony Adjustments
- Sprint Review: demo บน TestFlight ไม่ใช่ production
- Definition of Done: "TestFlight ready" ไม่ใช่ "production"
- Separate "Code Complete" จาก "User Available"
```

### 8.2 Coordinating iOS/Android Parity

```swift
// Feature Parity Tracking System

struct FeatureParity {
    let featureName: String
    let iOSStatus: PlatformStatus
    let androidStatus: PlatformStatus
    let priority: Priority
    let notes: String
    
    enum PlatformStatus {
        case notStarted
        case inDevelopment
        case inReview
        case shipped(version: String)
        case blocked(reason: String)
        case notApplicable  // platform-specific feature
    }
    
    enum Priority {
        case mustHave      // ต้อง ship พร้อมกัน
        case shouldHave    // ควร ship ภายใน 1 sprint
        case niceToHave    // best effort
    }
    
    var isInParity: Bool {
        if case .shipped = iOSStatus, case .shipped = androidStatus {
            return true
        }
        return false
    }
    
    var parityGap: String? {
        switch (iOSStatus, androidStatus) {
        case (.shipped(let iosV), .notStarted):
            return "iOS shipped v\(iosV), Android hasn't started"
        case (.notStarted, .shipped(let androidV)):
            return "Android shipped v\(androidV), iOS hasn't started"
        default:
            return nil
        }
    }
}

// Cross-platform sync meeting template
struct WeeklySyncAgenda {
    static let items = [
        "1. Feature Parity Dashboard review (10 min)",
        "2. Blockers ที่ affect both platforms (10 min)",
        "3. Shared API changes ที่ impact both (10 min)",
        "4. Design consistency review (5 min)",
        "5. Action items (5 min)"
    ]
}
```

### 8.3 Dependency on Apple (WWDC Changes)

```markdown
# จัดการ WWDC ใน Team Planning

## ก่อน WWDC (มิถุนายน)
- Reserve 1 sprint หลัง WWDC สำหรับ exploration
- ไม่ commit major deliverables ใน sprint นั้น
- Prepare test devices กับ beta OS

## ระหว่าง WWDC Week
- Watch sessions relevant ต่อ product
- Test app บน beta Xcode/iOS
- Identify breaking changes
- อัปเดต tech radar

## หลัง WWDC (มิถุนายน - กันยายน)
- สร้าง Adoption Backlog (new APIs ที่น่าสนใจ)
- Fix deprecation warnings
- Plan iOS version support drop (ถ้า relevant)
- Prepare for new OS launch (กันยายน)

## iOS Version Support Policy
ตัวอย่าง policy:
- Support: iOS current และ iOS current-1
- เมื่อ iOS X+2 ออก → drop iOS X
- Review annually หลัง September release
```

### 8.4 Beta Testing Management

```swift
// TestFlight Distribution Strategy

struct BetaTestingStrategy {
    // Tiers of beta testers
    enum TesterGroup: String {
        case internal = "Internal Team"        // ~20 people
        case closedBeta = "Closed Beta"        // ~200 people
        case openBeta = "Public Beta"          // unlimited
    }
    
    // Distribution schedule
    struct ReleaseSchedule {
        // Feature complete → Internal testing
        let internalTestingStart: Date
        
        // แก้ bugs จาก internal → Closed beta
        let closedBetaStart: Date
        
        // Confidence ขึ้น → Open beta
        let openBetaStart: Date
        
        // Production release
        let productionRelease: Date
    }
    
    // TestFlight Feedback Collection
    struct FeedbackTemplate {
        static let crashReportGuidelines = """
        เมื่อพบ crash:
        1. บอกว่าทำอะไรอยู่ก่อน crash
        2. บอก device model และ iOS version
        3. Crash ซ้ำได้ไหม?
        """
        
        static let featureFeedbackGuidelines = """
        เมื่อ feedback feature:
        1. Rate ความ useful (1-5)
        2. อธิบาย use case ของคุณ
        3. ปัญหาที่เจอ (ถ้ามี)
        """
    }
}
```

---

## 9. Documentation Culture

### 9.1 Wiki Strategies สำหรับ iOS Teams

**โครงสร้าง Wiki ที่แนะนำ:**

```
iOS Team Wiki Structure:
├── Getting Started
│   ├── Setup Guide
│   ├── Development Environment
│   └── First PR Guide
├── Architecture
│   ├── High-level Architecture
│   ├── Module Structure
│   ├── ADRs (Architecture Decision Records)
│   └── Dependencies
├── Development Guides
│   ├── Code Style Guide
│   ├── Testing Guide
│   ├── Accessibility Guide
│   └── Performance Guide
├── Release Process
│   ├── Release Checklist
│   ├── Hotfix Process
│   └── App Store Guidelines
├── Runbooks
│   ├── Incident Response
│   ├── On-call Guide
│   └── Monitoring Guide
└── Team
    ├── Team Charter
    ├── Meeting Templates
    └── Onboarding
```

### 9.2 API Documentation ด้วย DocC

```swift
/// ตัวอย่างการเขียน DocC documentation ที่ดี

/// A manager class that handles user authentication operations.
///
/// Use `AuthManager` to handle all authentication-related tasks in your app.
/// It provides methods for signing in, signing out, and managing tokens.
///
/// ## Overview
/// `AuthManager` is a singleton that maintains the current authentication state.
/// It automatically refreshes tokens when they expire.
///
/// ## Usage
/// ```swift
/// let manager = AuthManager.shared
/// 
/// do {
///     let user = try await manager.signIn(email: "user@example.com", password: "password")
///     print("Signed in as: \(user.displayName)")
/// } catch AuthError.invalidCredentials {
///     print("Invalid credentials")
/// } catch {
///     print("Unexpected error: \(error)")
/// }
/// ```
///
/// ## Topics
/// ### Authentication
/// - ``signIn(email:password:)``
/// - ``signOut()``
///
/// ### Token Management
/// - ``currentToken``
/// - ``refreshToken()``
public class AuthManager {
    
    /// The shared instance of `AuthManager`.
    public static let shared = AuthManager()
    
    /// The current authentication token.
    ///
    /// Returns `nil` if the user is not authenticated.
    public var currentToken: String? { nil }
    
    /// Signs in with email and password.
    ///
    /// - Parameters:
    ///   - email: The user's email address.
    ///   - password: The user's password.
    /// - Returns: The authenticated ``User`` object.
    /// - Throws: ``AuthError/invalidCredentials`` if credentials are wrong.
    /// - Throws: ``AuthError/networkError`` if the request fails.
    ///
    /// - Important: This method must be called on the main thread.
    public func signIn(email: String, password: String) async throws -> User {
        // Implementation
        fatalError("Not implemented")
    }
    
    /// Signs out the current user.
    ///
    /// After calling this method, ``currentToken`` will be `nil`.
    public func signOut() {
        // Implementation
    }
}

/// Represents an authenticated user.
public struct User {
    /// The user's unique identifier.
    public let id: String
    
    /// The user's display name.
    public let displayName: String
    
    /// The user's email address.
    public let email: String
}

/// Errors that can occur during authentication.
public enum AuthError: Error {
    /// The provided credentials are invalid.
    case invalidCredentials
    
    /// A network error occurred.
    case networkError(underlying: Error)
    
    /// The account has been locked due to too many failed attempts.
    case accountLocked
}
```

---

## 10. Mentorship และ Growth

### 10.1 Growth Framework สำหรับ iOS Engineers (L1-L6)

```markdown
# iOS Engineering Levels

## L1 - Junior iOS Engineer
Technical:
- เข้าใจ Swift fundamentals
- สามารถ implement features จาก spec
- ขอ help เมื่อ stuck

Ownership:
- ดูแล tasks ระดับ day-to-day
- Deliver well-defined stories

Collaboration:
- Participate ใน team meetings
- Ask good questions

## L2 - iOS Engineer
Technical:
- ลึกใน iOS frameworks (UIKit/SwiftUI, URLSession, etc.)
- Debug complex issues
- Write unit tests

Ownership:
- ดูแล features end-to-end
- Proactively identify issues

Collaboration:
- Code review ของ peers
- Contribute ใน technical discussions

## L3 - Senior iOS Engineer
Technical:
- Deep expertise ใน iOS architecture
- Performance optimization
- Lead technical design สำหรับ features ขนาดกลาง

Ownership:
- ดูแล multiple features ใน parallel
- Define quality standards ของ work ตัวเอง

Collaboration:
- Mentor junior engineers
- Drive technical decisions ใน team

## L4 - Staff iOS Engineer / Tech Lead
Technical:
- Architectural expertise ข้าม teams
- Define coding standards ให้ทีม
- Evaluate new technologies

Ownership:
- ดูแล technical health ของ whole team
- Drive multi-sprint initiatives

Collaboration:
- Cross-team influence
- Stakeholder management

## L5 - Principal iOS Engineer
Technical:
- Company-wide technical vision
- Author of key ADRs
- External thought leader (talks, OSS)

Ownership:
- ดูแล multi-team, multi-quarter work
- Define platform strategy

## L6 - Distinguished/Fellow
- Industry recognition
- Sets direction for entire mobile org
```

### 10.2 1:1 Conversation Frameworks

```swift
// 1:1 Meeting Framework

struct OneOnOneMeeting {
    let date: Date
    let engineer: String
    let format: Format
    
    enum Format {
        case checkin      // 15 min: ทุกอาทิตย์
        case deep         // 30 min: ทุก 2 อาทิตย์
        case careerReview // 60 min: ทุก quarter
    }
}

// Weekly Check-in (15 min) Template
struct WeeklyCheckinAgenda {
    static let questions = [
        "อาทิตย์นี้ทำอะไรไปบ้าง? มีอะไรที่ block ไหม?",
        "มีอะไรที่ต้องการ support ไหม?",
        "มีอะไรที่อยากให้ฉันรู้ไหม?",
        "ฉันจะ unblock อะไรให้ได้บ้าง?"
    ]
}

// Monthly Deep Dive (30 min) Template
struct MonthlyDeepDiveAgenda {
    static let topics = [
        "Review: งานที่ทำในเดือนนี้ดีไหม? (ให้ engineer assess ก่อน)",
        "Growth: เรียนรู้อะไรใหม่บ้าง? อยากเรียนรู้อะไรเพิ่ม?",
        "Challenge: มีอะไรที่ยากหรือน่าหงุดหน่ายไหม?",
        "Relationships: การ work กับทีม/stakeholders เป็นยังไง?",
        "Goals: ความคืบหน้าของ quarterly goals เป็นยังไง?"
    ]
}

// Quarterly Career Review Template
struct QuarterlyCareerReview {
    // Self-assessment topics
    static let selfAssessmentPrompts = [
        "Strengths ที่แสดงออกมาใน quarter นี้คืออะไร?",
        "Areas ที่ improve ได้ใน quarter นี้คืออะไร?",
        "ช่วงเวลาที่ภูมิใจที่สุดใน quarter นี้คืออะไร?",
        "ช่วงเวลาที่ยากที่สุดคืออะไร? เรียนรู้อะไร?"
    ]
    
    // Manager feedback topics
    static let managerFeedbackAreas = [
        "Technical contributions",
        "Collaboration & communication",
        "Ownership & reliability",
        "Growth trajectory",
        "Areas to focus next quarter"
    ]
    
    // Goal setting
    static let goalCategories = [
        "Technical skills",
        "Leadership/influence",
        "Delivery",
        "Personal development"
    ]
}
```

### 10.3 Supporting Engineers Toward Promotion

```markdown
# Promotion Preparation Framework

## 6 เดือนก่อน Promotion Cycle

### Calibrate Level Expectations
- หา examples ของ L(n+1) behavior ที่ชัดเจน
- Compare กับ peers ที่ promoted successfully
- Identify gaps อย่างตรงไปตรงมา

### Design Opportunities
- Assign projects ที่ showcase L(n+1) skills
- ให้ lead technical design meeting
- ให้ present ต่อ wider audience
- ให้ mentor someone

### Document Evidence
- Keep running list ของ achievements
- Quantify impact where possible
  "ลด build time จาก 15 min เป็น 8 min, saving team 3h/day"
- Collect peer feedback

## 3 เดือนก่อน Promotion Cycle

### Close Gaps
- Address areas ที่ยังขาดหลักฐาน
- Complete in-progress projects
- Ensure visibility ของ work ให้ stakeholders

### Prepare Promo Doc
- Summary of impact
- Evidence for each dimension
- Growth trajectory
- What they'll do at next level

## Conversations
"ฉัน support การ promote คุณ"
ต้องพูดอย่างชัดเจน อย่า leave ambiguous

"ยังไม่พร้อมเพราะ..."
ต้องบอกเหตุผลชัดเจน และ concrete actions
```

---

## 11. Remote iOS Team Management

### 11.1 Async Communication Tools

```markdown
# Async-First Communication Framework

## Principles
1. Write first, meet second
2. Document decisions ไม่ใช่แค่ communicate orally
3. Explicit > Implicit
4. Overcommunicate context

## Tools by Use Case

### ด่วน (< 1 ชั่วโมง) → Slack DM
### ทั่วไป (< 1 วัน) → Slack Channel
### Complex discussion → Linear/Jira + Slack thread
### Decision → ADR document
### Meeting notes → Confluence/Notion

## Effective Async Messages

❌ ไม่ดี:
"มีปัญหาเรื่อง login ครับ"

✅ ดี:
"[ACTION NEEDED] Login crash on iOS 16
- ปัญหา: App crashes เมื่อ login บน iOS 16.4
- Impact: ประมาณ 15% ของ users
- Steps: 1. Open app 2. Tap Login 3. Enter credentials 4. Crash
- Context: Regression จาก PR #456 (merged เมื่อวาน)
- Need: ใครช่วย review ได้ไหม? Target fix: EOD
- Crashlytics: [link]"

## Meeting Hygiene
- Every meeting ต้องมี agenda ที่ส่งล่วงหน้า
- เริ่ม meeting ด้วย 2-min async update sharing
- Record meetings ที่สำคัญ
- Meeting notes ใน 30 min หลัง meeting
```

### 11.2 Code Review ข้าม Time Zones

```swift
// Code Review Protocol สำหรับ Distributed Teams

struct CodeReviewProtocol {
    // Office Hours Overlap (กำหนดเวลาที่ทีม overlap กัน)
    let coreHours: TimeInterval  // เช่น 4 ชั่วโมงที่ทุกคน online พร้อมกัน
    
    // Rotation สำหรับ First Review
    // เพื่อให้ทุกคนได้ review ทุกส่วนของ codebase
    struct ReviewRotation {
        let week: Int
        let primaryReviewer: String
        let secondaryReviewer: String
    }
    
    // Async Review Guidelines
    static let asyncReviewGuidelines = """
    1. Comment ต้องชัดเจนและ self-contained
       (ไม่ expect real-time clarification)
    
    2. ถ้า comment มีหลาย options ให้ rank preference:
       "Option 1 (preferred): ... 
        Option 2: ... 
        Either works for me"
    
    3. ถ้า blocking issue → mark clearly:
       "🚨 BLOCKER: ..."
       "💡 SUGGESTION (non-blocking): ..."
    
    4. ถ้า unclear → request sync:
       "มี concern เรื่อง architecture ที่นี่
        ขอนัด 15 min sync ดีกว่า"
    
    5. Respond ต่อ review comments ใน 24 ชั่วโมง
    """
}
```

---

## 12. Metrics สำหรับ iOS Teams

### 12.1 Crash-Free Rate Targets

```swift
// Mobile App Metrics Dashboard

struct AppHealthMetrics {
    // Stability
    var crashFreeRate: Double           // target: > 99.9%
    var anrFreeRate: Double             // target: > 99.8% (Android equivalent)
    var p99LaunchTime: TimeInterval     // target: < 3s
    
    // Engagement
    var dailyActiveUsers: Int
    var sessionLength: TimeInterval
    var retentionD1: Double             // Day 1 retention
    var retentionD7: Double             // Day 7 retention
    var retentionD30: Double            // Day 30 retention
    
    // Quality
    var appStoreRating: Double          // target: > 4.5
    var ratingCount: Int
    
    // Performance
    var p50ApiLatency: TimeInterval     // 50th percentile
    var p99ApiLatency: TimeInterval     // 99th percentile
    var offlineSuccessRate: Double      // % actions that work offline
    
    // Development
    var ciBuildTime: TimeInterval       // target: < 15 min
    var releaseFrequency: Int           // releases per month
    var leadTime: TimeInterval          // commit to production
    var changeFailureRate: Double       // % releases requiring hotfix
    var meanTimeToRestore: TimeInterval // MTTR สำหรับ incidents
}

// Weekly Metrics Review
struct WeeklyMetricsSnapshot {
    let week: String  // "2025-W04"
    let metrics: AppHealthMetrics
    let changes: [MetricChange]
    let alerts: [MetricAlert]
    
    struct MetricChange {
        let metric: String
        let previous: Double
        let current: Double
        var trend: String {
            current > previous ? "📈" : current < previous ? "📉" : "➡️"
        }
    }
    
    struct MetricAlert {
        let metric: String
        let severity: Severity
        let message: String
        enum Severity { case warning, critical }
    }
}
```

### 12.2 CI Build Time Optimization

```swift
// Build Time Optimization Strategies

struct BuildOptimization {
    
    // 1. Module Caching
    static let cachingStrategies = [
        "Precompile slow dependencies (e.g., SwiftUI, Combine)",
        "Cache DerivedData ใน CI (restore จาก branch-specific key)",
        "ใช้ XCFrameworks แทน source packages สำหรับ stable deps"
    ]
    
    // 2. Incremental Builds
    static let incrementalBuildTips = [
        "Minimize Whole Module Optimization ใน debug builds",
        "ใช้ build phases carefully (avoid clean builds)",
        "Enable incremental compilation"
    ]
    
    // 3. Parallel Testing
    static let parallelTestingConfig = """
    # In Fastlane
    run_tests(
      project: "MyApp.xcodeproj",
      scheme: "MyApp",
      parallel_testing: true,
      concurrent_workers: 4,
      max_concurrent_simulators: 4
    )
    """
    
    // 4. Test Splitting
    static let testSplitting = """
    # GitLab CI: Split tests across jobs
    test-unit:
      script: xcodebuild test -testPlan UnitTests
    test-integration:
      script: xcodebuild test -testPlan IntegrationTests
    test-ui:
      script: xcodebuild test -testPlan UITests
    """
}

// Build Time Tracking
class BuildTimeTracker {
    static func trackBuildTime() {
        // ใน post-build script
        let buildTimeScript = """
        #!/bin/bash
        BUILD_TIME=$(date -d "$(tail -n1 build_log.txt | awk '{print $1, $2}')" +%s)
        CURRENT_TIME=$(date +%s)
        DURATION=$((CURRENT_TIME - BUILD_TIME))
        
        curl -X POST https://metrics.company.com/build-times \\
          -d "branch=$CI_BRANCH&duration=$DURATION&result=$BUILD_RESULT"
        """
        print(buildTimeScript)
    }
}
```

### 12.3 Release Cadence Metrics

```swift
// Release Velocity Dashboard

struct ReleaseMetrics {
    let quarter: String
    let releases: [Release]
    
    struct Release {
        let version: String
        let releaseDate: Date
        let submissionDate: Date
        let reviewDuration: TimeInterval  // ใน hours
        let featuresIncluded: Int
        let bugsFixed: Int
        let crashFreeRateAtLaunch: Double
        let appStoreRatingChange: Double  // +/- 
    }
    
    // คำนวณ metrics
    var averageReleaseCadence: TimeInterval {
        guard releases.count > 1 else { return 0 }
        let gaps = zip(releases, releases.dropFirst()).map { prev, next in
            next.releaseDate.timeIntervalSince(prev.releaseDate)
        }
        return gaps.reduce(0, +) / Double(gaps.count)
    }
    
    var averageReviewTime: TimeInterval {
        return releases.map { $0.reviewDuration }.reduce(0, +) / Double(releases.count)
    }
    
    var hotfixRate: Double {
        // % ของ releases ที่ตามด้วย hotfix ใน 48h
        let hotfixCount = releases.filter { release in
            // check if next release within 48h
            if let nextRelease = releases.first(where: { $0.releaseDate > release.releaseDate }) {
                return nextRelease.releaseDate.timeIntervalSince(release.releaseDate) < 48 * 3600
            }
            return false
        }.count
        return Double(hotfixCount) / Double(releases.count)
    }
}
```

---

## 13. แบบฝึกหัดสมบูรณ์

### Exercise 1: Team Retrospective Format

```markdown
# Sprint Retrospective Template

## Format: 4Ls Retrospective (60 min สำหรับ 2-week sprint)

### Liked (10 min)
คำถาม: อะไรที่ทำได้ดีใน sprint นี้?

ตัวอย่าง responses:
- "Code review process เร็วขึ้นมาก"
- "Architecture decision ที่ทำร่วมกันรู้สึก collaborative"
- "เราช่วยกัน debug ปัญหาซับซ้อนได้ดี"

### Learned (10 min)
คำถาม: เรียนรู้อะไรใหม่บ้าง?

ตัวอย่าง responses:
- "SwiftUI animation API ที่ยังไม่เคยรู้"
- "การใช้ Instruments เพื่อ find memory leaks"
- "Cross-team dependency ต้องเริ่ม communicate เร็วกว่านี้"

### Lacked (10 min)
คำถาม: อะไรที่ขาดไปใน sprint นี้?

ตัวอย่าง responses:
- "Design spec ไม่ครบตอนเริ่ม sprint"
- "ขาด documentation สำหรับ complex module"
- "ไม่มี time สำหรับ tech debt"

### Longed For (10 min)
คำถาม: อยากให้มีอะไรเพิ่มเติม?

ตัวอย่าง responses:
- "อยากมี dedicated time สำหรับ learning"
- "อยากมี better test infrastructure"
- "อยากมี more async collaboration tools"

### Action Items (20 min)
จาก retrospective items เลือก 1-3 items ที่จะ address ใน sprint ถัดไป

Format:
| Action | Owner | Done By |
|--------|-------|---------|
| Setup lint rule สำหรับ common issues | Alice | Sprint N+1 |
| Write design handoff checklist | Bob (with PM) | Week 1 |
| Schedule architecture review | Charlie | Sprint N+1 |
```

### Exercise 2: ADR Template สมบูรณ์

```markdown
# ADR-005: การเลือก Image Loading และ Caching Solution

## Status
Accepted (2025-01-15)

## Context
แอปของเราโหลด images จากหลาย sources:
- Product images (CDN)
- User avatars (S3)
- Marketing banners (CMS)

ปัจจุบัน implement image loading เองด้วย URLSession ทำให้:
1. Code ซ้ำซ้อนหลายที่
2. Memory management inconsistent
3. Disk caching ไม่มีประสิทธิภาพ
4. Progressive loading ไม่รองรับ

## Decision
ใช้ Kingfisher เป็น image loading library หลัก

## Rationale

### Comparison Matrix

| Feature | Custom URLSession | Kingfisher | SDWebImage | Nuke |
|---------|-----------------|------------|------------|------|
| Memory Cache | Manual | ✅ Built-in | ✅ Built-in | ✅ Built-in |
| Disk Cache | ❌ None | ✅ Automatic | ✅ Automatic | ✅ Automatic |
| Progressive JPEG | ❌ | ✅ | ✅ | ✅ |
| SwiftUI Support | Manual | ✅ Native | Partial | ✅ |
| Active Maintenance | N/A | ✅ High | ✅ Medium | ✅ High |
| Swift 5.9+ | N/A | ✅ | ✅ | ✅ |
| Binary Size | 0 | ~2MB | ~3MB | ~1.5MB |
| Team Familiarity | High | Medium | Low | Low |

### Why Kingfisher over Nuke
แม้ Nuke จะเบากว่า แต่ Kingfisher เลือกเพราะ:
1. Team มี existing experience
2. SwiftUI integration ดีกว่า
3. Documentation ครอบคลุมกว่า
4. Community size ใหญ่กว่า

## Consequences

### ดี
- ลด boilerplate code ~500 lines
- Memory usage ลด ~30% จาก better caching
- Image loading time ลด ~40% จาก disk cache
- Progressive loading ทำให้ UX ดีขึ้น

### ไม่ดี
- External dependency เพิ่ม
- Binary size เพิ่ม ~2MB
- ต้อง migrate existing code

## Migration Plan
Phase 1 (Sprint 1): Setup Kingfisher, configure caching
Phase 2 (Sprint 2-3): Migrate Product images
Phase 3 (Sprint 4): Migrate remaining image usage
Phase 4 (Sprint 5): Remove old custom implementation

## Review Date
Q3 2025 - reevaluate if Apple provides native solution
```

---

## 14. แบบฝึกหัดและเฉลย

### แบบฝึกหัดที่ 1: Technical Debt Prioritization

**โจทย์:** ทีมมี Technical Debt 5 items ต่อไปนี้ ให้ prioritize และบอกว่าควรทำใน order ไหน:

1. Upgrade Alamofire จาก v4 ถึง v5 (ใช้เวลา 2 วัน)
2. Add missing unit tests ใน payment module (ใช้เวลา 1 สัปดาห์)
3. Fix potential crash ใน background sync (ใช้เวลา 1 วัน)
4. Refactor 2,000-line view controller (ใช้เวลา 2 สัปดาห์)
5. Update minimum iOS version จาก 14 เป็น 16 (ใช้เวลา 3 วัน)

**เฉลย:**

```swift
// วิธีคิด: ใช้ Impact x Urgency matrix

let techDebtItems = [
    TechnicalDebtItem(
        id: "TD-001",
        title: "Fix crash in background sync",
        impact: .quality,
        effort: .quick,        // 1 วัน
        category: .codeSmell,
        createdDate: Date(),
        estimatedCostPerSprint: 8
    ),
    TechnicalDebtItem(
        id: "TD-002",
        title: "Add unit tests in payment module",
        impact: .quality,
        effort: .small,        // 1 สัปดาห์
        category: .poorTestCoverage,
        createdDate: Date(),
        estimatedCostPerSprint: 12
    ),
    TechnicalDebtItem(
        id: "TD-003",
        title: "Upgrade Alamofire v4 to v5",
        impact: .maintenance,
        effort: .quick,        // 2 วัน
        category: .outdatedDependency,
        createdDate: Date(),
        estimatedCostPerSprint: 4
    ),
    TechnicalDebtItem(
        id: "TD-004",
        title: "Update minimum iOS 14 to 16",
        impact: .velocity,
        effort: .small,        // 3 วัน
        category: .codeSmell,
        createdDate: Date(),
        estimatedCostPerSprint: 6
    ),
    TechnicalDebtItem(
        id: "TD-005",
        title: "Refactor massive view controller",
        impact: .maintenance,
        effort: .large,        // 2 สัปดาห์
        category: .codeSmell,
        createdDate: Date(),
        estimatedCostPerSprint: 8
    )
]

/*
Priority Order:
1. TD-001: Fix crash (Impact: quality/stability, Effort: เล็ก, ROI: สูงมาก)
2. TD-002: Payment tests (Impact: quality/risk mitigation สูง, payment critical path)
3. TD-004: iOS version update (unlock new APIs, ลด maintenance burden)
4. TD-003: Alamofire upgrade (dependency hygiene, security patches)
5. TD-005: Refactor VC (valuable แต่ effort สูง, ทำทีละส่วน)

Key Insight: TD-001 ทำก่อนเพราะ crash = user impact ทันที
            TD-005 ทำทีละ piece ไม่ต้อง big bang refactor
*/
```

### แบบฝึกหัดที่ 2: Code Review Feedback

**โจทย์:** เขียน constructive code review feedback สำหรับโค้ดต่อไปนี้:

```swift
// โค้ดที่ถูก review
class ProfileViewController: UIViewController {
    var user: User?
    var networkManager = NetworkManager()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        fetchUser()
    }
    
    func fetchUser() {
        networkManager.fetchUser(id: "123") { user, error in
            if error == nil {
                self.user = user
                self.updateUI()
            }
        }
    }
    
    func updateUI() {
        // update labels
    }
}
```

**เฉลย:**

```swift
/*
Code Review Feedback:

issue: Retain cycle ใน closure
Line: `networkManager.fetchUser(id: "123") { user, error in`

`self` ถูก capture strongly ใน closure ทำให้:
1. ProfileViewController ไม่ถูก deallocate จนกว่า network request จะเสร็จ
2. ถ้า network request ช้า + user navigate away = memory leak

Fix:
```swift
networkManager.fetchUser(id: "123") { [weak self] user, error in
    guard let self = self else { return }
    ...
}
```

---

issue: Main thread violation
Line: `self.updateUI()`

Network callback อาจ execute บน background thread
การ update UI ต้องทำบน main thread เสมอ

Fix:
```swift
DispatchQueue.main.async {
    self?.updateUI()
}
```

---

issue: Error handling ไม่สมบูรณ์
Line: `if error == nil {`

ตอนนี้เมื่อ error ไม่เป็น nil ก็ไม่ทำอะไรเลย
User จะไม่รู้ว่าเกิด error

Fix:
```swift
if let error = error {
    showErrorAlert(error.localizedDescription)
    return
}
guard let user = user else {
    showErrorAlert("Unexpected: no user data")
    return
}
self?.user = user
self?.updateUI()
```

---

suggestion: hardcoded user ID
Line: `networkManager.fetchUser(id: "123")`

User ID "123" เป็น hardcoded string
ควร inject จากข้างนอก หรืออ่านจาก session

Fix:
```swift
guard let userId = currentSession?.userId else { return }
networkManager.fetchUser(id: userId) { ... }
```

---

nitpick: NetworkManager ควรเป็น dependency injection
ตอนนี้ ProfileViewController สร้าง NetworkManager เอง ทำให้ test ยาก

แนะนำ:
```swift
init(networkManager: NetworkManagerProtocol = NetworkManager()) {
    self.networkManager = networkManager
    super.init(nibName: nil, bundle: nil)
}
```
*/

// โค้ดที่แก้แล้ว
class ProfileViewController: UIViewController {
    var user: User?
    private let networkManager: NetworkManagerProtocol
    
    init(networkManager: NetworkManagerProtocol = NetworkManager()) {
        self.networkManager = networkManager
        super.init(nibName: nil, bundle: nil)
    }
    
    required init?(coder: NSCoder) {
        self.networkManager = NetworkManager()
        super.init(coder: coder)
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        fetchUser()
    }
    
    func fetchUser() {
        guard let userId = SessionManager.shared.currentUserId else {
            showErrorAlert("User not logged in")
            return
        }
        
        networkManager.fetchUser(id: userId) { [weak self] result in
            DispatchQueue.main.async {
                switch result {
                case .success(let user):
                    self?.user = user
                    self?.updateUI()
                case .failure(let error):
                    self?.showErrorAlert(error.localizedDescription)
                }
            }
        }
    }
    
    func updateUI() {
        // update labels
    }
    
    private func showErrorAlert(_ message: String) {
        // show alert
    }
}

protocol NetworkManagerProtocol {
    func fetchUser(id: String, completion: @escaping (Result<User, Error>) -> Void)
}

class NetworkManager: NetworkManagerProtocol {
    func fetchUser(id: String, completion: @escaping (Result<User, Error>) -> Void) {
        // implementation
    }
}

class SessionManager {
    static let shared = SessionManager()
    var currentUserId: String?
}
```

### แบบฝึกหัดที่ 3: 1:1 Scenario Practice

**สถานการณ์:** Engineer คนหนึ่งใน team ของคุณ (Bob) ส่งงานช้าลงเรื่อยๆ ใน 2 sprints ที่ผ่านมา และดูเงียบในที่ประชุม

**โจทย์:** วางแผน 1:1 conversation กับ Bob

**เฉลย:**

```markdown
# 1:1 Conversation Plan with Bob

## Preparation (Before Meeting)
- ดู sprint metrics ของ Bob
- ดู code commits ช่วง 2 เดือนที่ผ่านมา
- ทบทวน PRs และ code review comments
- จดสิ่งที่ observe ไว้อย่างเป็น objective fact

## Opening (5 min)
เริ่มด้วย rapport building ก่อน:
"เป็นยังไงบ้างช่วงนี้? งานนอก office เป็นยังไง?"

## Observation (5 min)
แชร์ observation อย่าง non-judgmental:
"ฉันสังเกตว่า 2 sprints ที่ผ่านมา มีบางงานที่ carry over มา
และ meeting ช่วงนี้ดูเงียบกว่าปกติ
อยากถามตรงๆ ว่ามีอะไรเกิดขึ้นไหม?"

## Listen (10-15 min)
ฟังอย่าง actively
อย่า interrupt หรือ jump to solutions ทันที
ถามเพื่อให้เข้าใจ ไม่ใช่เพื่อ judge:

"บอกเล่าให้ฟังได้ไหมว่าตอนนี้รู้สึกยังไงกับงาน?"
"มีอะไรที่ block อยู่ไหม ทั้ง technical หรือ non-technical?"
"มีอะไรที่ฉันทำได้เพื่อ support คุณได้บ้าง?"

## Possible Scenarios and Responses

Scenario A: Personal issues (health, family)
Response: "ขอบคุณที่เล่าให้ฟัง เรื่องนี้สำคัญมาก
          HR มีโปรแกรม EAP ที่ช่วยได้ ลองดูไหม?
          งาน ฉัน adjust ได้ ขอให้บอก"

Scenario B: Technical challenge
Response: "โอเค ปัญหาตรงไหนที่ยากที่สุด?
          ลอง pair กัน 30 นาที แล้วไป unstuck ด้วยกันไหม?"

Scenario C: Motivation/Engagement
Response: "ฟังดูเหมือน feeling stuck ใน current work
          อะไรที่ทำให้คุณ excited ใน iOS development?
          เราจะหาทางให้คุณทำงานที่ motivating กว่านี้ได้ไหม?"

Scenario D: Team/Interpersonal issue
Response: "บอกฉันเพิ่มเติมได้ไหม?
          ฉันอยากทำให้แน่ใจว่า team environment safe สำหรับทุกคน"

## Follow-up (Action items)
- กำหนด concrete next steps
- นัด check-in ในอีก 1 สัปดาห์
- Document (privately) ว่าคุยอะไรบ้าง

## Key Principle
จุดประสงค์ของ 1:1 นี้คือ UNDERSTAND ไม่ใช่ JUDGE
Bob ควรออกจาก meeting รู้สึกว่ามีคนรับฟัง ไม่ใช่ถูกตัดสิน
```

---

## สรุป

การเป็น iOS Tech Lead ที่ดีต้องพัฒนาทักษะหลายด้านพร้อมกัน:

1. **Technical Credibility**: ยังต้อง involved ในงานเทคนิคจริงๆ ไม่ใช่แค่ manage
2. **Decision Making**: ใช้ data และกระบวนการ (ADR) ในการตัดสินใจ
3. **People Skills**: Feedback, mentoring, 1:1 conversations
4. **Process Design**: Sprint ceremonies, incident response, code review standards
5. **Strategic Thinking**: Technical roadmap ที่สอดคล้องกับ business
6. **Metrics Orientation**: วัดผลทุกอย่าง crash rate, build time, velocity
7. **Hiring**: สร้างทีมที่แข็งแกร่งตั้งแต่ต้น

ที่สำคัญที่สุดคือ: **ความสำเร็จของทีมคือความสำเร็จของคุณ** ไม่ใช่โค้ดที่คุณเขียนเอง

---

*หมายเหตุ: บทนี้เป็นส่วนหนึ่งของหลักสูตร iOS Development ขั้นสูง ควรศึกษาควบคู่กับการปฏิบัติจริงในทีม*
