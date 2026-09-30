# Part 86: การเตรียมตัวสัมภาษณ์บริษัทชั้นนำ (Apple, Google, Meta และอื่นๆ)

## บทนำ

การสัมภาษณ์กับบริษัท tech ชั้นนำอย่าง Apple, Google, Meta, Spotify หรือ Airbnb ต้องการการเตรียมตัวอย่างมีระบบ บทนี้จะครอบคลุมทุกอย่างตั้งแต่ algorithm problems ไปจนถึง behavioral questions และ system design สำหรับ iOS developer โดยเฉพาะ

---

## บทที่ 1: Company-Specific iOS Interview Culture

### 1.1 Apple

**วัฒนธรรมการสัมภาษณ์ของ Apple:**
- เน้น **deep technical knowledge** มากกว่า breadth
- ให้ความสำคัญกับ **attention to detail** อย่างมาก
- ถามเกี่ยวกับ iOS frameworks อย่างละเอียด
- ต้องการคนที่เข้าใจ "Apple way" ของการทำงาน

**สิ่งที่ Apple ให้ความสำคัญ:**
```
- ความเข้าใจ Apple platforms อย่างลึกซึ้ง
- การออกแบบ API ที่ elegant
- Performance และ battery efficiency
- User experience ที่ seamless
- Privacy และ security by design
```

**ตัวอย่างคำถามที่ Apple ถาม:**
```swift
// "อธิบาย lifecycle ของ UIViewController ทุก state"
class LifecycleExplainer: UIViewController {
    
    override func loadView() {
        // Called when view needs to be loaded
        // สร้าง view hierarchy ที่นี่ถ้าไม่ใช้ Interface Builder
        super.loadView()
        print("1. loadView: view hierarchy ถูกสร้าง")
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // Called after view is loaded into memory
        // ทำงานเพียงครั้งเดียว
        print("2. viewDidLoad: view ถูกโหลดเข้า memory แล้ว")
        // Setup: delegates, data sources, initial data fetch
    }
    
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        // Called before view is added to window
        // อาจถูกเรียกหลายครั้ง
        print("3. viewWillAppear: กำลังจะแสดง view")
        // Update UI state, register for notifications
    }
    
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        // Called after view is added to window
        print("4. viewDidAppear: view ปรากฏแล้ว")
        // Start animations, start video playback
    }
    
    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        // Called before view is removed
        print("5. viewWillDisappear: กำลังจะซ่อน view")
        // Commit edits, resign first responder
    }
    
    override func viewDidDisappear(_ animated: Bool) {
        super.viewDidDisappear(animated)
        // Called after view is removed
        print("6. viewDidDisappear: view หายไปแล้ว")
        // Stop services, unregister notifications
    }
    
    override func viewWillLayoutSubviews() {
        super.viewWillLayoutSubviews()
        // Called before layoutSubviews
        print("7. viewWillLayoutSubviews: กำลังจะ layout subviews")
    }
    
    override func viewDidLayoutSubviews() {
        super.viewDidLayoutSubviews()
        // Called after layoutSubviews - bounds are final
        print("8. viewDidLayoutSubviews: layout subviews เสร็จแล้ว")
        // Update layers, scroll positions
    }
    
    deinit {
        print("9. deinit: ViewController ถูก deallocated")
        // View controller is being deallocated
    }
}
```

### 1.2 Google

**วัฒนธรรมการสัมภาษณ์ของ Google:**
- เน้น **algorithms และ data structures** อย่างหนัก
- ให้ความสำคัญกับ **code quality และ clean code**
- **System design** ต้องการ scalability thinking
- ถามเรื่อง **testing strategies**

**Google's Hiring Bar:**
```
L3 (Entry): LeetCode Medium สม่ำเสมอ + basic system design
L4 (Mid): LeetCode Medium/Hard + intermediate system design
L5 (Senior): LeetCode Hard + complex system design + leadership
L6 (Staff): Architecture, cross-team impact, deep expertise
```

**ตัวอย่างโจทย์ที่ Google ชอบถาม:**
```swift
// Binary Search Variations
// Google ชอบถาม variations ของ Binary Search

// Standard: Find element
func binarySearch(_ nums: [Int], _ target: Int) -> Int {
    var left = 0, right = nums.count - 1
    
    while left <= right {
        let mid = left + (right - left) / 2
        if nums[mid] == target {
            return mid
        } else if nums[mid] < target {
            left = mid + 1
        } else {
            right = mid - 1
        }
    }
    return -1
}

// Variation: Find leftmost position
func searchFirstOccurrence(_ nums: [Int], _ target: Int) -> Int {
    var left = 0, right = nums.count - 1
    var result = -1
    
    while left <= right {
        let mid = left + (right - left) / 2
        if nums[mid] == target {
            result = mid
            right = mid - 1  // ไปทางซ้ายต่อ
        } else if nums[mid] < target {
            left = mid + 1
        } else {
            right = mid - 1
        }
    }
    return result
}

// Variation: Search in rotated sorted array
func searchRotated(_ nums: [Int], _ target: Int) -> Int {
    var left = 0, right = nums.count - 1
    
    while left <= right {
        let mid = left + (right - left) / 2
        
        if nums[mid] == target { return mid }
        
        // ตรวจว่า left half sorted หรือไม่
        if nums[left] <= nums[mid] {
            if nums[left] <= target && target < nums[mid] {
                right = mid - 1
            } else {
                left = mid + 1
            }
        } else {
            // right half is sorted
            if nums[mid] < target && target <= nums[right] {
                left = mid + 1
            } else {
                right = mid - 1
            }
        }
    }
    return -1
}
```

### 1.3 Meta (Facebook)

**วัฒนธรรมการสัมภาษณ์ของ Meta:**
- เน้น **product sense** - ทำไมเราถึง build สิ่งนี้?
- ให้ความสำคัญกับ **execution at scale**
- ถาม **mobile architecture** ในเชิงลึก
- **Behavioral** เน้น impact และ speed

**Meta's Core Values ที่ควรอ้างถึง:**
```
- Move Fast
- Be Bold
- Focus on Long-Term Impact
- Build Awesome Things
- Live in the Future
```

### 1.4 Spotify, Airbnb, Uber

**Spotify:**
- เน้น **audio/media** technical knowledge
- Agile methodologies
- Backend + iOS full-stack thinking

**Airbnb:**
- UI/UX excellence สำคัญมาก
- **Design thinking** บน technical decisions
- Performance มาก (listing page ต้องเร็ว)
- Accessibility เป็น first-class citizen

**Uber:**
- **Real-time systems** (maps, pricing)
- Location services deep dive
- Multi-threading และ concurrency
- Offline-first thinking

---

## บทที่ 2: Apple-Specific Interview Process

### 2.1 Format ของการสัมภาษณ์ Apple

```
Round 1: Recruiter Screen (30 min)
  - Background, experience, motivation
  - Basic technical questions
  - Logistics

Round 2: Technical Phone Screen (60 min)
  - Coding challenge (LeetCode style)
  - iOS knowledge questions
  - Usually with a senior engineer

Round 3: On-site / Virtual On-site (Full Day)
  - 5-7 interviews, 45-60 min each:
    A. Coding challenge 1 (algorithms)
    B. Coding challenge 2 (data structures)
    C. iOS Technical Deep Dive
    D. System Design (iOS-specific)
    E. Design Exercise (build a feature)
    F. Behavioral / Engineering Excellence
    G. Manager interview (for senior roles)
```

### 2.2 iOS Technical Deep Dive

```swift
// คำถามที่มักถามในส่วน iOS Technical:

// 1. "How does ARC work? Explain retain cycles."
class Parent {
    var child: Child?
    deinit { print("Parent deallocated") }
}

class Child {
    // BAD: Strong reference cycle
    // var parent: Parent?
    
    // GOOD: Weak reference breaks the cycle
    weak var parent: Parent?
    deinit { print("Child deallocated") }
}

// Retain cycle example with closures:
class ViewController: UIViewController {
    var timer: Timer?
    var count = 0
    
    func startTimer() {
        // BAD: Strong reference to self
        // timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { _ in
        //     self.count += 1  // Retain cycle!
        // }
        
        // GOOD: Weak reference
        timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { [weak self] _ in
            self?.count += 1  // ใช้ optional chaining
        }
        
        // ALTERNATIVE: Unowned (use when you're sure self won't be nil)
        timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { [unowned self] _ in
            self.count += 1  // ถ้า self เป็น nil จะ crash
        }
    }
    
    func stopTimer() {
        timer?.invalidate()
        timer = nil
    }
}

// 2. "Explain the difference between frame and bounds"
class FrameVsBoundsDemo: UIView {
    func explain() {
        let childView = UIView(frame: CGRect(x: 100, y: 100, width: 200, height: 200))
        addSubview(childView)
        
        // frame: position and size ใน parent's coordinate system
        print("Frame:", childView.frame)
        // CGRect(x: 100, y: 100, width: 200, height: 200)
        
        // bounds: position and size ใน view's own coordinate system
        print("Bounds:", childView.bounds)
        // CGRect(x: 0, y: 0, width: 200, height: 200)
        
        // เมื่อ scroll: bounds.origin เปลี่ยน แต่ frame ไม่เปลี่ยน
        let scrollView = UIScrollView(frame: CGRect(x: 0, y: 0, width: 200, height: 200))
        scrollView.contentSize = CGSize(width: 400, height: 400)
        scrollView.contentOffset = CGPoint(x: 100, y: 0)
        
        // After scrolling:
        // scrollView.frame ยังคงเดิม
        // scrollView.bounds.origin = CGPoint(x: 100, y: 0)
    }
}

// 3. "How does UITableView work internally? Explain cell reuse."
class ReusableCellExplainer {
    /*
    UITableView ใช้ reuse pool เพื่อประสิทธิภาพ:
    
    1. สร้าง cells เพียงแค่จำนวนที่มองเห็นบนหน้าจอ + buffer เล็กน้อย
    2. เมื่อ cell scroll ออกจากหน้าจอ → ไปที่ reuse queue
    3. เมื่อต้องการ cell ใหม่ → dequeue จาก reuse queue
    4. ถ้า queue ว่าง → สร้าง cell ใหม่
    
    ประโยชน์: ใช้ memory น้อยมากแม้ list จะมี 10,000 items
    */
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        // dequeueReusableCell ดึง cell จาก reuse pool
        let cell = tableView.dequeueReusableCell(withIdentifier: "Cell", for: indexPath)
        
        // IMPORTANT: Configure cell ทุกครั้ง เพราะ cell อาจเป็น reused cell
        // ที่มีข้อมูลเก่าอยู่
        cell.textLabel?.text = "Item \(indexPath.row)"
        
        // Reset states ที่อาจเปลี่ยนไป
        cell.accessoryType = .none
        cell.imageView?.image = nil
        
        return cell
    }
}
```

### 2.3 Design Exercise

```swift
// Apple มักให้ design exercise แบบ:
// "Design a photo editing feature similar to Photos app"
// "Build a contacts list with search"
// "Implement a simple drawing canvas"

// ตัวอย่าง: Design a simple drawing canvas

import UIKit

// Step 1: Define the model
struct DrawingPath {
    var points: [CGPoint]
    var color: UIColor
    var lineWidth: CGFloat
}

// Step 2: Design the view
class DrawingCanvas: UIView {
    
    // State
    private var paths: [DrawingPath] = []
    private var currentPath: DrawingPath?
    
    // Configuration
    var strokeColor: UIColor = .black
    var strokeWidth: CGFloat = 3.0
    
    // MARK: - Drawing
    override func draw(_ rect: CGRect) {
        guard let context = UIGraphicsGetCurrentContext() else { return }
        
        // Draw completed paths
        for path in paths {
            drawPath(path, in: context)
        }
        
        // Draw current path
        if let currentPath = currentPath {
            drawPath(currentPath, in: context)
        }
    }
    
    private func drawPath(_ path: DrawingPath, in context: CGContext) {
        guard path.points.count > 1 else { return }
        
        context.setStrokeColor(path.color.cgColor)
        context.setLineWidth(path.lineWidth)
        context.setLineCap(.round)
        context.setLineJoin(.round)
        
        context.move(to: path.points[0])
        for point in path.points.dropFirst() {
            context.addLine(to: point)
        }
        
        context.strokePath()
    }
    
    // MARK: - Touch Handling
    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        guard let touch = touches.first else { return }
        let point = touch.location(in: self)
        
        currentPath = DrawingPath(
            points: [point],
            color: strokeColor,
            lineWidth: strokeWidth
        )
    }
    
    override func touchesMoved(_ touches: Set<UITouch>, with event: UIEvent?) {
        guard let touch = touches.first else { return }
        let point = touch.location(in: self)
        
        currentPath?.points.append(point)
        setNeedsDisplay() // Trigger redraw
    }
    
    override func touchesEnded(_ touches: Set<UITouch>, with event: UIEvent?) {
        guard let touch = touches.first else { return }
        let point = touch.location(in: self)
        
        currentPath?.points.append(point)
        
        if let completedPath = currentPath {
            paths.append(completedPath)
        }
        currentPath = nil
        setNeedsDisplay()
    }
    
    // MARK: - Actions
    func undo() {
        guard !paths.isEmpty else { return }
        paths.removeLast()
        setNeedsDisplay()
    }
    
    func clear() {
        paths.removeAll()
        setNeedsDisplay()
    }
    
    func exportImage() -> UIImage? {
        UIGraphicsBeginImageContextWithOptions(bounds.size, false, UIScreen.main.scale)
        defer { UIGraphicsEndImageContext() }
        
        drawHierarchy(in: bounds, afterScreenUpdates: true)
        return UIGraphicsGetImageFromCurrentImageContext()
    }
}
```

---

## บทที่ 3: Algorithm Mastery สำหรับ iOS Interviews

### 3.1 Most Common Patterns

**Pattern 1: Two Pointers**
```swift
// ใช้เมื่อ: sorted array, find pairs, palindrome check

// Problem: Two Sum II (sorted array)
func twoSum(_ numbers: [Int], _ target: Int) -> [Int] {
    var left = 0, right = numbers.count - 1
    
    while left < right {
        let sum = numbers[left] + numbers[right]
        if sum == target {
            return [left + 1, right + 1] // 1-indexed
        } else if sum < target {
            left += 1
        } else {
            right -= 1
        }
    }
    return []
}

// Problem: Valid Palindrome
func isPalindrome(_ s: String) -> Bool {
    let chars = s.lowercased().filter { $0.isLetter || $0.isNumber }
    var left = chars.startIndex
    var right = chars.index(before: chars.endIndex)
    
    while left < right {
        if chars[left] != chars[right] {
            return false
        }
        left = chars.index(after: left)
        right = chars.index(before: right)
    }
    return true
}

// Problem: Container With Most Water
func maxArea(_ height: [Int]) -> Int {
    var left = 0, right = height.count - 1
    var maxWater = 0
    
    while left < right {
        let water = min(height[left], height[right]) * (right - left)
        maxWater = max(maxWater, water)
        
        if height[left] < height[right] {
            left += 1
        } else {
            right -= 1
        }
    }
    return maxWater
}
```

**Pattern 2: Sliding Window**
```swift
// ใช้เมื่อ: subarray/substring problems, optimization

// Problem: Longest Substring Without Repeating Characters
func lengthOfLongestSubstring(_ s: String) -> Int {
    var charIndex: [Character: Int] = [:]
    var maxLength = 0
    var start = 0
    
    for (i, char) in s.enumerated() {
        if let prevIndex = charIndex[char], prevIndex >= start {
            start = prevIndex + 1
        }
        charIndex[char] = i
        maxLength = max(maxLength, i - start + 1)
    }
    return maxLength
}

// Problem: Minimum Window Substring
func minWindow(_ s: String, _ t: String) -> String {
    var need: [Character: Int] = [:]
    for c in t { need[c, default: 0] += 1 }
    
    var have = 0
    let required = need.count
    var result = ""
    var resultLen = Int.max
    var left = s.startIndex
    var window: [Character: Int] = [:]
    
    for right in s.indices {
        let c = s[right]
        window[c, default: 0] += 1
        
        if let needed = need[c], window[c] == needed {
            have += 1
        }
        
        while have == required {
            // Update result
            let windowLen = s.distance(from: left, to: right) + 1
            if windowLen < resultLen {
                resultLen = windowLen
                result = String(s[left...right])
            }
            
            // Shrink window
            let leftChar = s[left]
            window[leftChar]! -= 1
            if let needed = need[leftChar], window[leftChar]! < needed {
                have -= 1
            }
            left = s.index(after: left)
        }
    }
    return result
}
```

**Pattern 3: BFS/DFS**
```swift
// BFS: Shortest path, level-order traversal
// DFS: Path existence, backtracking

// BFS: Number of Islands
func numIslands(_ grid: [[Character]]) -> Int {
    var grid = grid
    var count = 0
    
    for i in 0..<grid.count {
        for j in 0..<grid[0].count {
            if grid[i][j] == "1" {
                count += 1
                bfs(&grid, i, j)
            }
        }
    }
    return count
}

private func bfs(_ grid: inout [[Character]], _ row: Int, _ col: Int) {
    var queue: [(Int, Int)] = [(row, col)]
    grid[row][col] = "0"
    let directions = [(0, 1), (0, -1), (1, 0), (-1, 0)]
    
    while !queue.isEmpty {
        let (r, c) = queue.removeFirst()
        
        for (dr, dc) in directions {
            let nr = r + dr, nc = c + dc
            if nr >= 0 && nr < grid.count &&
               nc >= 0 && nc < grid[0].count &&
               grid[nr][nc] == "1" {
                grid[nr][nc] = "0"
                queue.append((nr, nc))
            }
        }
    }
}

// DFS: All Paths from Source to Target (Graph)
func allPathsSourceTarget(_ graph: [[Int]]) -> [[Int]] {
    var results: [[Int]] = []
    var path: [Int] = [0]
    
    func dfs(_ node: Int) {
        if node == graph.count - 1 {
            results.append(path)
            return
        }
        
        for neighbor in graph[node] {
            path.append(neighbor)
            dfs(neighbor)
            path.removeLast()
        }
    }
    
    dfs(0)
    return results
}
```

**Pattern 4: Dynamic Programming**
```swift
// DP: Optimization problems, counting paths

// Fibonacci DP
func fibonacci(_ n: Int) -> Int {
    if n <= 1 { return n }
    var dp = [Int](repeating: 0, count: n + 1)
    dp[1] = 1
    
    for i in 2...n {
        dp[i] = dp[i-1] + dp[i-2]
    }
    return dp[n]
}

// Longest Common Subsequence
func longestCommonSubsequence(_ text1: String, _ text2: String) -> Int {
    let s1 = Array(text1), s2 = Array(text2)
    let m = s1.count, n = s2.count
    var dp = [[Int]](repeating: [Int](repeating: 0, count: n + 1), count: m + 1)
    
    for i in 1...m {
        for j in 1...n {
            if s1[i-1] == s2[j-1] {
                dp[i][j] = dp[i-1][j-1] + 1
            } else {
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
            }
        }
    }
    return dp[m][n]
}
```

### 3.2 30 Must-Know LeetCode Problems with Swift Solutions

**Problem 1: Two Sum (HashMap)**
```swift
// LeetCode #1 - Easy
// Given an array of integers, return indices of the two numbers that add up to target

func twoSum(_ nums: [Int], _ target: Int) -> [Int] {
    var map: [Int: Int] = [:]  // value → index
    
    for (i, num) in nums.enumerated() {
        let complement = target - num
        if let j = map[complement] {
            return [j, i]
        }
        map[num] = i
    }
    return []
}

// Time: O(n), Space: O(n)
// Test:
// twoSum([2,7,11,15], 9) → [0, 1]
// twoSum([3,2,4], 6) → [1, 2]
```

**Problem 2: Valid Parentheses (Stack)**
```swift
// LeetCode #20 - Easy
// Given a string of brackets, return true if valid

func isValid(_ s: String) -> Bool {
    var stack: [Character] = []
    let pairs: [Character: Character] = [")": "(", "]": "[", "}": "{"]
    
    for char in s {
        if "([{".contains(char) {
            stack.append(char)
        } else if let open = pairs[char] {
            if stack.isEmpty || stack.last != open {
                return false
            }
            stack.removeLast()
        }
    }
    return stack.isEmpty
}

// Time: O(n), Space: O(n)
// Test:
// isValid("()[]{}") → true
// isValid("([)]") → false
// isValid("{[]}") → true
```

**Problem 3: Merge Intervals (Sort + Greedy)**
```swift
// LeetCode #56 - Medium
// Merge overlapping intervals

func merge(_ intervals: [[Int]]) -> [[Int]] {
    let sorted = intervals.sorted { $0[0] < $1[0] }
    var result: [[Int]] = []
    
    for interval in sorted {
        if result.isEmpty || result.last![1] < interval[0] {
            result.append(interval)
        } else {
            result[result.count - 1][1] = max(result.last![1], interval[1])
        }
    }
    return result
}

// Time: O(n log n), Space: O(n)
// Test:
// merge([[1,3],[2,6],[8,10],[15,18]]) → [[1,6],[8,10],[15,18]]
// merge([[1,4],[4,5]]) → [[1,5]]
```

**Problem 4: LRU Cache (HashMap + Doubly Linked List)**
```swift
// LeetCode #146 - Medium
// Design a data structure with O(1) get and put

class LRUCache {
    
    class Node {
        var key: Int
        var val: Int
        var prev: Node?
        var next: Node?
        
        init(_ key: Int, _ val: Int) {
            self.key = key
            self.val = val
        }
    }
    
    private let capacity: Int
    private var cache: [Int: Node] = [:]
    private let head = Node(0, 0)  // Dummy head (most recent)
    private let tail = Node(0, 0)  // Dummy tail (least recent)
    
    init(_ capacity: Int) {
        self.capacity = capacity
        head.next = tail
        tail.prev = head
    }
    
    func get(_ key: Int) -> Int {
        guard let node = cache[key] else { return -1 }
        remove(node)
        insertFront(node)
        return node.val
    }
    
    func put(_ key: Int, _ value: Int) {
        if let existing = cache[key] {
            remove(existing)
        }
        
        let node = Node(key, value)
        cache[key] = node
        insertFront(node)
        
        if cache.count > capacity {
            // Remove least recently used (before tail)
            if let lru = tail.prev {
                remove(lru)
                cache.removeValue(forKey: lru.key)
            }
        }
    }
    
    private func remove(_ node: Node) {
        node.prev?.next = node.next
        node.next?.prev = node.prev
    }
    
    private func insertFront(_ node: Node) {
        node.next = head.next
        node.prev = head
        head.next?.prev = node
        head.next = node
    }
}

// Time: O(1) for both get and put
// Space: O(capacity)
```

**Problem 5: Word Break (DP)**
```swift
// LeetCode #139 - Medium
// Given string and word dictionary, can string be segmented into words?

func wordBreak(_ s: String, _ wordDict: [String]) -> Bool {
    let words = Set(wordDict)
    let n = s.count
    var dp = [Bool](repeating: false, count: n + 1)
    dp[0] = true
    
    let chars = Array(s)
    
    for i in 1...n {
        for j in 0..<i {
            if dp[j] {
                let substring = String(chars[j..<i])
                if words.contains(substring) {
                    dp[i] = true
                    break
                }
            }
        }
    }
    return dp[n]
}

// Time: O(n² * m) where m = average word length, Space: O(n)
// Test:
// wordBreak("leetcode", ["leet","code"]) → true
// wordBreak("applepenapple", ["apple","pen"]) → true
// wordBreak("catsandog", ["cats","dog","sand","and","cat"]) → false
```

**Problem 6: Course Schedule (Topological Sort)**
```swift
// LeetCode #207 - Medium
// Detect cycle in directed graph (prerequisites check)

func canFinish(_ numCourses: Int, _ prerequisites: [[Int]]) -> Bool {
    // Build adjacency list
    var graph = [[Int]](repeating: [], count: numCourses)
    var inDegree = [Int](repeating: 0, count: numCourses)
    
    for prereq in prerequisites {
        let course = prereq[0], pre = prereq[1]
        graph[pre].append(course)
        inDegree[course] += 1
    }
    
    // BFS Topological Sort (Kahn's algorithm)
    var queue: [Int] = []
    for i in 0..<numCourses {
        if inDegree[i] == 0 {
            queue.append(i)
        }
    }
    
    var completed = 0
    while !queue.isEmpty {
        let course = queue.removeFirst()
        completed += 1
        
        for next in graph[course] {
            inDegree[next] -= 1
            if inDegree[next] == 0 {
                queue.append(next)
            }
        }
    }
    
    return completed == numCourses
}

// Time: O(V + E), Space: O(V + E)
// Test:
// canFinish(2, [[1,0]]) → true
// canFinish(2, [[1,0],[0,1]]) → false (cycle!)
```

**Problem 7: Serialize/Deserialize Binary Tree**
```swift
// LeetCode #297 - Hard
// Serialize and deserialize a binary tree

public class TreeNode {
    public var val: Int
    public var left: TreeNode?
    public var right: TreeNode?
    public init(_ val: Int) {
        self.val = val
    }
}

class Codec {
    
    // Encodes a tree to a single string using BFS
    func serialize(_ root: TreeNode?) -> String {
        guard let root = root else { return "" }
        
        var result: [String] = []
        var queue: [TreeNode?] = [root]
        
        while !queue.isEmpty {
            let node = queue.removeFirst()
            
            if let node = node {
                result.append(String(node.val))
                queue.append(node.left)
                queue.append(node.right)
            } else {
                result.append("null")
            }
        }
        
        return result.joined(separator: ",")
    }
    
    // Decodes encoded data to tree
    func deserialize(_ data: String) -> TreeNode? {
        guard !data.isEmpty else { return nil }
        
        let values = data.split(separator: ",").map(String.init)
        guard values[0] != "null" else { return nil }
        
        let root = TreeNode(Int(values[0])!)
        var queue: [TreeNode] = [root]
        var i = 1
        
        while !queue.isEmpty && i < values.count {
            let node = queue.removeFirst()
            
            if values[i] != "null" {
                node.left = TreeNode(Int(values[i])!)
                queue.append(node.left!)
            }
            i += 1
            
            if i < values.count && values[i] != "null" {
                node.right = TreeNode(Int(values[i])!)
                queue.append(node.right!)
            }
            i += 1
        }
        
        return root
    }
}
```

**Problem 8: Number of Islands (BFS)**
```swift
// LeetCode #200 - Medium (แสดงไปแล้วด้านบน)
// Additional variations:

// Problem: Max Area of Island
func maxAreaOfIsland(_ grid: [[Int]]) -> Int {
    var grid = grid
    var maxArea = 0
    
    for i in 0..<grid.count {
        for j in 0..<grid[0].count {
            if grid[i][j] == 1 {
                maxArea = max(maxArea, dfsArea(&grid, i, j))
            }
        }
    }
    return maxArea
}

private func dfsArea(_ grid: inout [[Int]], _ i: Int, _ j: Int) -> Int {
    guard i >= 0 && i < grid.count &&
          j >= 0 && j < grid[0].count &&
          grid[i][j] == 1 else { return 0 }
    
    grid[i][j] = 0
    return 1 +
        dfsArea(&grid, i+1, j) +
        dfsArea(&grid, i-1, j) +
        dfsArea(&grid, i, j+1) +
        dfsArea(&grid, i, j-1)
}
```

**Problem 9: Find Median from Data Stream**
```swift
// LeetCode #295 - Hard
// Design MedianFinder class

class MedianFinder {
    // Max-heap for lower half
    private var lowerHalf: [Int] = []
    // Min-heap for upper half
    private var upperHalf: [Int] = []
    
    init() {}
    
    func addNum(_ num: Int) {
        // Add to lower half (max-heap, simulated with negation)
        lowerHalf.append(-num)
        // Heapify up would be here in a real heap implementation
        lowerHalf.sort(by: <)  // Simplified
        
        // Balance: move max of lower to upper if needed
        if let maxLower = lowerHalf.first {
            if upperHalf.isEmpty || -maxLower > upperHalf.first! {
                upperHalf.append(-lowerHalf.removeFirst())
                upperHalf.sort()
            }
        }
        
        // Ensure size difference <= 1
        if lowerHalf.count > upperHalf.count + 1 {
            upperHalf.append(-lowerHalf.removeFirst())
            upperHalf.sort()
        } else if upperHalf.count > lowerHalf.count {
            lowerHalf.append(-upperHalf.removeFirst())
            lowerHalf.sort()
        }
    }
    
    func findMedian() -> Double {
        if lowerHalf.count == upperHalf.count {
            let lower = Double(-(lowerHalf.first ?? 0))
            let upper = Double(upperHalf.first ?? 0)
            return (lower + upper) / 2.0
        } else {
            return Double(-(lowerHalf.first ?? 0))
        }
    }
}
```

**Problem 10: Climbing Stairs (DP)**
```swift
// LeetCode #70 - Easy
func climbStairs(_ n: Int) -> Int {
    if n <= 2 { return n }
    var prev1 = 1, prev2 = 2
    
    for _ in 3...n {
        let current = prev1 + prev2
        prev1 = prev2
        prev2 = current
    }
    return prev2
}
// Time: O(n), Space: O(1)
```

**Problem 11: Maximum Subarray (Kadane's Algorithm)**
```swift
// LeetCode #53 - Medium
func maxSubArray(_ nums: [Int]) -> Int {
    var maxSum = nums[0]
    var currentSum = nums[0]
    
    for num in nums.dropFirst() {
        currentSum = max(num, currentSum + num)
        maxSum = max(maxSum, currentSum)
    }
    return maxSum
}
// Time: O(n), Space: O(1)
```

**Problem 12: Binary Tree Level Order Traversal**
```swift
// LeetCode #102 - Medium
func levelOrder(_ root: TreeNode?) -> [[Int]] {
    guard let root = root else { return [] }
    
    var result: [[Int]] = []
    var queue: [TreeNode] = [root]
    
    while !queue.isEmpty {
        let levelSize = queue.count
        var level: [Int] = []
        
        for _ in 0..<levelSize {
            let node = queue.removeFirst()
            level.append(node.val)
            
            if let left = node.left { queue.append(left) }
            if let right = node.right { queue.append(right) }
        }
        result.append(level)
    }
    return result
}
```

**Problem 13: Reverse Linked List**
```swift
// LeetCode #206 - Easy
public class ListNode {
    public var val: Int
    public var next: ListNode?
    public init(_ val: Int) { self.val = val }
}

func reverseList(_ head: ListNode?) -> ListNode? {
    var prev: ListNode? = nil
    var current = head
    
    while current != nil {
        let next = current?.next
        current?.next = prev
        prev = current
        current = next
    }
    return prev
}

// Recursive version:
func reverseListRecursive(_ head: ListNode?) -> ListNode? {
    guard let head = head, head.next != nil else { return head }
    
    let newHead = reverseListRecursive(head.next)
    head.next?.next = head
    head.next = nil
    return newHead
}
```

**Problem 14: Coin Change (DP)**
```swift
// LeetCode #322 - Medium
func coinChange(_ coins: [Int], _ amount: Int) -> Int {
    var dp = [Int](repeating: amount + 1, count: amount + 1)
    dp[0] = 0
    
    for i in 1...amount {
        for coin in coins {
            if coin <= i {
                dp[i] = min(dp[i], dp[i - coin] + 1)
            }
        }
    }
    
    return dp[amount] > amount ? -1 : dp[amount]
}
// Time: O(amount * coins.count), Space: O(amount)
```

**Problem 15: Validate Binary Search Tree**
```swift
// LeetCode #98 - Medium
func isValidBST(_ root: TreeNode?) -> Bool {
    return validate(root, min: Int.min, max: Int.max)
}

private func validate(_ node: TreeNode?, min: Int, max: Int) -> Bool {
    guard let node = node else { return true }
    
    if node.val <= min || node.val >= max {
        return false
    }
    
    return validate(node.left, min: min, max: node.val) &&
           validate(node.right, min: node.val, max: max)
}
```

**Problem 16: Product of Array Except Self**
```swift
// LeetCode #238 - Medium
func productExceptSelf(_ nums: [Int]) -> [Int] {
    let n = nums.count
    var result = [Int](repeating: 1, count: n)
    
    // Left pass: result[i] = product of all elements to the left
    var left = 1
    for i in 0..<n {
        result[i] = left
        left *= nums[i]
    }
    
    // Right pass: multiply by product of all elements to the right
    var right = 1
    for i in stride(from: n - 1, through: 0, by: -1) {
        result[i] *= right
        right *= nums[i]
    }
    
    return result
}
// Time: O(n), Space: O(1) (excluding output)
```

**Problem 17: Trapping Rain Water**
```swift
// LeetCode #42 - Hard
func trap(_ height: [Int]) -> Int {
    var left = 0, right = height.count - 1
    var maxLeft = 0, maxRight = 0
    var water = 0
    
    while left < right {
        if height[left] < height[right] {
            if height[left] >= maxLeft {
                maxLeft = height[left]
            } else {
                water += maxLeft - height[left]
            }
            left += 1
        } else {
            if height[right] >= maxRight {
                maxRight = height[right]
            } else {
                water += maxRight - height[right]
            }
            right -= 1
        }
    }
    return water
}
// Time: O(n), Space: O(1)
```

**Problem 18: Group Anagrams**
```swift
// LeetCode #49 - Medium
func groupAnagrams(_ strs: [String]) -> [[String]] {
    var map: [String: [String]] = [:]
    
    for str in strs {
        let key = String(str.sorted())
        map[key, default: []].append(str)
    }
    
    return Array(map.values)
}
// Time: O(n * k log k) where k is max string length
```

**Problem 19: Longest Palindromic Substring**
```swift
// LeetCode #5 - Medium
func longestPalindrome(_ s: String) -> String {
    let chars = Array(s)
    let n = chars.count
    
    if n == 0 { return "" }
    
    var start = 0, maxLen = 1
    
    func expandAroundCenter(_ left: Int, _ right: Int) {
        var l = left, r = right
        while l >= 0 && r < n && chars[l] == chars[r] {
            if r - l + 1 > maxLen {
                maxLen = r - l + 1
                start = l
            }
            l -= 1
            r += 1
        }
    }
    
    for i in 0..<n {
        expandAroundCenter(i, i)     // Odd length
        expandAroundCenter(i, i + 1) // Even length
    }
    
    return String(chars[start..<(start + maxLen)])
}
```

**Problem 20: Find All Anagrams in a String**
```swift
// LeetCode #438 - Medium
func findAnagrams(_ s: String, _ p: String) -> [Int] {
    guard s.count >= p.count else { return [] }
    
    let sArray = Array(s), pArray = Array(p)
    var pCount = [Int](repeating: 0, count: 26)
    var windowCount = [Int](repeating: 0, count: 26)
    let aScalar = Int(Character("a").asciiValue!)
    
    for char in pArray {
        pCount[Int(char.asciiValue!) - aScalar] += 1
    }
    
    var result: [Int] = []
    
    for i in 0..<sArray.count {
        // Add right character
        windowCount[Int(sArray[i].asciiValue!) - aScalar] += 1
        
        // Remove left character if window too large
        if i >= pArray.count {
            let leftIndex = Int(sArray[i - pArray.count].asciiValue!) - aScalar
            windowCount[leftIndex] -= 1
        }
        
        // Check if window matches
        if windowCount == pCount {
            result.append(i - pArray.count + 1)
        }
    }
    
    return result
}
```

**Problem 21: Jump Game**
```swift
// LeetCode #55 - Medium
func canJump(_ nums: [Int]) -> Bool {
    var maxReach = 0
    
    for i in 0..<nums.count {
        if i > maxReach { return false }
        maxReach = max(maxReach, i + nums[i])
    }
    return true
}
```

**Problem 22: Implement Trie**
```swift
// LeetCode #208 - Medium
class Trie {
    
    class TrieNode {
        var children: [Character: TrieNode] = [:]
        var isEndOfWord = false
    }
    
    private let root = TrieNode()
    
    init() {}
    
    func insert(_ word: String) {
        var node = root
        for char in word {
            if node.children[char] == nil {
                node.children[char] = TrieNode()
            }
            node = node.children[char]!
        }
        node.isEndOfWord = true
    }
    
    func search(_ word: String) -> Bool {
        var node = root
        for char in word {
            guard let next = node.children[char] else { return false }
            node = next
        }
        return node.isEndOfWord
    }
    
    func startsWith(_ prefix: String) -> Bool {
        var node = root
        for char in prefix {
            guard let next = node.children[char] else { return false }
            node = next
        }
        return true
    }
}
```

**Problem 23: Pacific Atlantic Water Flow**
```swift
// LeetCode #417 - Medium
func pacificAtlantic(_ heights: [[Int]]) -> [[Int]] {
    let rows = heights.count, cols = heights[0].count
    var pacific = Set<[Int]>(), atlantic = Set<[Int]>()
    
    func dfs(_ r: Int, _ c: Int, _ visited: inout Set<[Int]>, _ prevHeight: Int) {
        if visited.contains([r, c]) { return }
        if r < 0 || c < 0 || r >= rows || c >= cols { return }
        if heights[r][c] < prevHeight { return }
        
        visited.insert([r, c])
        dfs(r+1, c, &visited, heights[r][c])
        dfs(r-1, c, &visited, heights[r][c])
        dfs(r, c+1, &visited, heights[r][c])
        dfs(r, c-1, &visited, heights[r][c])
    }
    
    for r in 0..<rows {
        dfs(r, 0, &pacific, heights[r][0])
        dfs(r, cols-1, &atlantic, heights[r][cols-1])
    }
    
    for c in 0..<cols {
        dfs(0, c, &pacific, heights[0][c])
        dfs(rows-1, c, &atlantic, heights[rows-1][c])
    }
    
    return pacific.intersection(atlantic).sorted { $0[0] != $1[0] ? $0[0] < $1[0] : $0[1] < $1[1] }
}
```

**Problem 24: Graph Valid Tree**
```swift
// LeetCode #261 - Medium (Premium)
func validTree(_ n: Int, _ edges: [[Int]]) -> Bool {
    // A valid tree: n nodes, n-1 edges, no cycle, all connected
    if edges.count != n - 1 { return false }
    
    var adj = [[Int]](repeating: [], count: n)
    for edge in edges {
        adj[edge[0]].append(edge[1])
        adj[edge[1]].append(edge[0])
    }
    
    var visited = Set<Int>()
    var queue = [0]
    visited.insert(0)
    
    while !queue.isEmpty {
        let node = queue.removeFirst()
        for neighbor in adj[node] {
            if !visited.contains(neighbor) {
                visited.insert(neighbor)
                queue.append(neighbor)
            }
        }
    }
    
    return visited.count == n
}
```

**Problem 25: Decode Ways**
```swift
// LeetCode #91 - Medium
func numDecodings(_ s: String) -> Int {
    let chars = Array(s)
    let n = chars.count
    
    if n == 0 || chars[0] == "0" { return 0 }
    
    var dp = [Int](repeating: 0, count: n + 1)
    dp[0] = 1
    dp[1] = 1
    
    for i in 2...n {
        let oneDigit = Int(String(chars[i-1]))!
        let twoDigits = Int(String(chars[i-2...i-1]))!
        
        if oneDigit >= 1 {
            dp[i] += dp[i-1]
        }
        if twoDigits >= 10 && twoDigits <= 26 {
            dp[i] += dp[i-2]
        }
    }
    
    return dp[n]
}
```

**Problem 26: Alien Dictionary**
```swift
// LeetCode #269 - Hard (Premium)
// Topological sort to determine character order

func alienOrder(_ words: [String]) -> String {
    var adj: [Character: Set<Character>] = [:]
    var inDegree: [Character: Int] = [:]
    
    // Initialize all characters
    for word in words {
        for c in word {
            if adj[c] == nil { adj[c] = [] }
            if inDegree[c] == nil { inDegree[c] = 0 }
        }
    }
    
    // Find edges
    for i in 0..<words.count - 1 {
        let w1 = Array(words[i]), w2 = Array(words[i+1])
        let minLen = min(w1.count, w2.count)
        
        // Check invalid case
        if w1.count > w2.count && w1.prefix(minLen).elementsEqual(w2.prefix(minLen)) {
            return ""
        }
        
        for j in 0..<minLen {
            if w1[j] != w2[j] {
                if adj[w1[j]]?.contains(w2[j]) == false {
                    adj[w1[j]]?.insert(w2[j])
                    inDegree[w2[j], default: 0] += 1
                }
                break
            }
        }
    }
    
    // BFS Topological Sort
    var queue: [Character] = inDegree.filter { $0.value == 0 }.map { $0.key }
    var result: [Character] = []
    
    while !queue.isEmpty {
        let c = queue.removeFirst()
        result.append(c)
        
        for neighbor in adj[c] ?? [] {
            inDegree[neighbor]! -= 1
            if inDegree[neighbor] == 0 {
                queue.append(neighbor)
            }
        }
    }
    
    return result.count == inDegree.count ? String(result) : ""
}
```

**Problem 27: Meeting Rooms II**
```swift
// LeetCode #253 - Medium (Premium)
func minMeetingRooms(_ intervals: [[Int]]) -> Int {
    let starts = intervals.map { $0[0] }.sorted()
    let ends = intervals.map { $0[1] }.sorted()
    
    var rooms = 0, maxRooms = 0
    var s = 0, e = 0
    
    while s < intervals.count {
        if starts[s] < ends[e] {
            rooms += 1
            s += 1
        } else {
            rooms -= 1
            e += 1
        }
        maxRooms = max(maxRooms, rooms)
    }
    return maxRooms
}
```

**Problem 28: Kth Largest Element in Array**
```swift
// LeetCode #215 - Medium
// QuickSelect algorithm

func findKthLargest(_ nums: [Int], _ k: Int) -> Int {
    var nums = nums
    return quickSelect(&nums, 0, nums.count - 1, nums.count - k)
}

func quickSelect(_ nums: inout [Int], _ left: Int, _ right: Int, _ k: Int) -> Int {
    let pivot = nums[right]
    var partitionIndex = left
    
    for i in left..<right {
        if nums[i] <= pivot {
            nums.swapAt(i, partitionIndex)
            partitionIndex += 1
        }
    }
    nums.swapAt(partitionIndex, right)
    
    if partitionIndex == k {
        return nums[partitionIndex]
    } else if partitionIndex < k {
        return quickSelect(&nums, partitionIndex + 1, right, k)
    } else {
        return quickSelect(&nums, left, partitionIndex - 1, k)
    }
}
// Average: O(n), Worst: O(n²)
```

**Problem 29: Design Twitter**
```swift
// LeetCode #355 - Medium

class Twitter {
    
    private var tweets: [Int: [(Int, Int)]] = [:]  // userId → [(timestamp, tweetId)]
    private var following: [Int: Set<Int>] = [:]   // userId → Set<followeeId>
    private var timestamp = 0
    
    init() {}
    
    func postTweet(_ userId: Int, _ tweetId: Int) {
        tweets[userId, default: []].append((timestamp, tweetId))
        timestamp += 1
    }
    
    func getNewsFeed(_ userId: Int) -> [Int] {
        var allTweets: [(Int, Int)] = []
        
        // Own tweets
        allTweets += tweets[userId] ?? []
        
        // Followees' tweets
        for followeeId in following[userId] ?? [] {
            allTweets += tweets[followeeId] ?? []
        }
        
        // Sort by timestamp and take top 10
        return allTweets
            .sorted { $0.0 > $1.0 }
            .prefix(10)
            .map { $0.1 }
    }
    
    func follow(_ followerId: Int, _ followeeId: Int) {
        following[followerId, default: []].insert(followeeId)
    }
    
    func unfollow(_ followerId: Int, _ followeeId: Int) {
        following[followerId]?.remove(followeeId)
    }
}
```

**Problem 30: Minimum Interval to Include Each Query**
```swift
// LeetCode #2158 - Hard
// Complex interval scheduling with heap

func minInterval(_ intervals: [[Int]], _ queries: [Int]) -> [Int] {
    let sortedIntervals = intervals.sorted { $0[0] < $1[0] }
    let indexedQueries = queries.enumerated().sorted { $0.element < $1.element }
    
    var result = [Int](repeating: -1, count: queries.count)
    // Min-heap: (interval size, right bound)
    var heap: [(Int, Int)] = []
    var i = 0
    
    for (queryIdx, query) in indexedQueries {
        // Add intervals that start <= query
        while i < sortedIntervals.count && sortedIntervals[i][0] <= query {
            let interval = sortedIntervals[i]
            let size = interval[1] - interval[0] + 1
            heap.append((size, interval[1]))
            heap.sort { $0.0 < $1.0 }
            i += 1
        }
        
        // Remove intervals that have ended
        while !heap.isEmpty && heap[0].1 < query {
            heap.removeFirst()
        }
        
        if !heap.isEmpty {
            result[queryIdx] = heap[0].0
        }
    }
    
    return result
}
```

---

## บทที่ 4: System Design Interviews สำหรับ iOS

### 4.1 Framework: RESHADED

```
R - Requirements (Functional & Non-Functional)
E - Estimation (Scale, Traffic, Storage)
S - Storage (Data model, Database choice)
H - High-level Design (Component diagram)
A - APIs (Mobile ↔ Backend contracts)
D - Deep Dive (Critical components)
E - Evaluation (Meeting requirements)
D - Detailed Design (อาจรวมกับ Deep Dive)
```

### 4.2 Design: Instagram Feed

```
Requirements:
Functional:
- User can upload photos
- User can follow/unfollow other users
- User can see feed of people they follow
- Support for likes and comments

Non-Functional:
- 1 billion users, 100M daily active
- Feed must load in < 2 seconds
- High availability (99.99% uptime)
- Eventually consistent (fine for feed)

Estimation:
- 100M DAU × 5 posts/day = 500M posts/day
- Average post size: 100KB = 50TB/day storage for photos
- Feed reads: 100M × 20 views/day = 2B reads/day
- Write: 500M/day = ~5,800 posts/second

High-level Design:
[Mobile App]
    ↓
[API Gateway / CDN]
    ↓
[Feed Service] → [Feed Cache (Redis)]
    ↓
[Media Service] → [Object Storage (S3)]
    ↓
[User Service] → [Graph DB (follows)]
    ↓
[Notification Service]

iOS-Specific Design:
```

```swift
// iOS Client Architecture for Instagram-like Feed

// Network Layer
struct FeedAPI {
    
    static func getFeed(cursor: String?, limit: Int = 20) async throws -> FeedResponse {
        var components = URLComponents(string: "https://api.instagram.com/v1/feed")!
        components.queryItems = [
            URLQueryItem(name: "limit", value: String(limit)),
            cursor.map { URLQueryItem(name: "cursor", value: $0) }
        ].compactMap { $0 }
        
        let (data, _) = try await URLSession.shared.data(from: components.url!)
        return try JSONDecoder().decode(FeedResponse.self, from: data)
    }
}

struct FeedResponse: Codable {
    let posts: [Post]
    let nextCursor: String?
    let hasMore: Bool
}

struct Post: Codable, Identifiable {
    let id: String
    let imageURL: URL
    let caption: String
    let author: User
    let likeCount: Int
    let timestamp: Date
}

// View Model with Pagination
@MainActor
class FeedViewModel: ObservableObject {
    @Published var posts: [Post] = []
    @Published var isLoading = false
    @Published var error: Error?
    
    private var cursor: String?
    private var hasMore = true
    
    func loadInitialFeed() async {
        isLoading = true
        do {
            let response = try await FeedAPI.getFeed(cursor: nil)
            posts = response.posts
            cursor = response.nextCursor
            hasMore = response.hasMore
        } catch {
            self.error = error
        }
        isLoading = false
    }
    
    func loadMore() async {
        guard hasMore && !isLoading else { return }
        isLoading = true
        
        do {
            let response = try await FeedAPI.getFeed(cursor: cursor)
            posts.append(contentsOf: response.posts)
            cursor = response.nextCursor
            hasMore = response.hasMore
        } catch {
            self.error = error
        }
        isLoading = false
    }
    
    func refresh() async {
        cursor = nil
        hasMore = true
        await loadInitialFeed()
    }
}

// Image Prefetching Strategy
class ImagePrefetchManager {
    private let prefetchWindow = 3 // Prefetch 3 posts ahead
    
    func prefetch(currentIndex: Int, posts: [Post]) {
        let startIndex = min(currentIndex + 1, posts.count - 1)
        let endIndex = min(currentIndex + prefetchWindow, posts.count - 1)
        
        guard startIndex <= endIndex else { return }
        
        for i in startIndex...endIndex {
            // Using SDWebImage or Kingfisher for prefetching
            let url = posts[i].imageURL
            URLSession.shared.dataTask(with: url) { _, _, _ in }.resume()
        }
    }
}
```

### 4.3 Design: Maps App Offline Feature

```swift
// Offline Maps Design

/*
Requirements:
- User can download map regions for offline use
- Show downloaded regions on map
- Navigate offline using downloaded data
- Sync updates when back online

Data Model:
- MapRegion: id, bounds, zoomLevels, size, downloadDate, version
- MapTile: regionId, x, y, zoom, imageData

Storage Strategy:
- Tile data: File system (optimized for binary data)
- Metadata: Core Data (queryable, relational)
- ~1GB per major city (reasonable for user storage)
*/

import CoreData
import MapKit

// Core Data Models
class OfflineMapManager {
    
    static let shared = OfflineMapManager()
    
    // Background download queue
    private lazy var downloadQueue: OperationQueue = {
        let queue = OperationQueue()
        queue.maxConcurrentOperationCount = 3
        queue.qualityOfService = .background
        return queue
    }()
    
    // Download a region
    func downloadRegion(_ region: MKCoordinateRegion) async throws -> DownloadTask {
        // 1. Calculate tile set needed for the region
        let tiles = calculateTiles(for: region, zoomLevels: 10...16)
        
        // 2. Estimate download size
        let estimatedSize = tiles.count * averageTileSize
        
        // 3. Check available storage
        guard hasEnoughStorage(bytes: estimatedSize) else {
            throw OfflineMapError.insufficientStorage
        }
        
        // 4. Start download with background URLSession
        return try await startDownload(tiles: tiles, region: region)
    }
    
    // Check if location has offline data
    func hasOfflineData(for coordinate: CLLocationCoordinate2D) -> Bool {
        // Query Core Data for downloaded regions
        // Check if coordinate is within any downloaded region
        return false // placeholder
    }
    
    // Load tile from local storage
    func tile(x: Int, y: Int, zoom: Int) -> UIImage? {
        let path = tilePath(x: x, y: y, zoom: zoom)
        guard let data = try? Data(contentsOf: path) else { return nil }
        return UIImage(data: data)
    }
    
    private func tilePath(x: Int, y: Int, zoom: Int) -> URL {
        let directory = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)[0]
        return directory.appendingPathComponent("tiles/\(zoom)/\(x)/\(y).png")
    }
    
    private func calculateTiles(for region: MKCoordinateRegion, zoomLevels: ClosedRange<Int>) -> [TileCoordinate] {
        var tiles: [TileCoordinate] = []
        // Convert lat/lng bounds to tile coordinates
        // This is the Mercator projection calculation
        return tiles
    }
    
    private func hasEnoughStorage(bytes: Int) -> Bool {
        guard let attributes = try? FileManager.default.attributesOfFileSystem(forPath: NSHomeDirectory()),
              let freeSpace = attributes[.systemFreeSize] as? Int else {
            return false
        }
        return freeSpace > bytes * 2 // 2x safety margin
    }
    
    private func startDownload(tiles: [TileCoordinate], region: MKCoordinateRegion) async throws -> DownloadTask {
        // Implementation using URLSession background download
        return DownloadTask()
    }
    
    private let averageTileSize = 50_000 // 50KB per tile
}

struct TileCoordinate {
    let x: Int, y: Int, zoom: Int
}

class DownloadTask {}
enum OfflineMapError: Error { case insufficientStorage }
```

### 4.4 Design: Real-Time Chat

```swift
// Real-Time Chat Architecture for iOS

/*
Requirements:
- Send and receive text messages
- Show online/offline status
- Delivery receipts (sent, delivered, read)
- Group chats
- Push notifications when offline

Technology Choices:
- WebSocket for real-time communication
- REST API for history and initial load
- APNs for push notifications when offline
- Core Data for local message storage (offline support)
*/

// WebSocket Manager
class ChatWebSocketManager {
    
    private var webSocket: URLSessionWebSocketTask?
    private let session: URLSession
    var onMessage: ((Message) -> Void)?
    var onStatusChange: ((ConnectionStatus) -> Void)?
    
    enum ConnectionStatus {
        case connecting, connected, disconnected(Error?)
    }
    
    init() {
        self.session = URLSession(configuration: .default)
    }
    
    func connect(userToken: String) {
        var request = URLRequest(url: URL(string: "wss://chat.example.com/ws")!)
        request.setValue("Bearer \(userToken)", forHTTPHeaderField: "Authorization")
        
        webSocket = session.webSocketTask(with: request)
        webSocket?.resume()
        
        onStatusChange?(.connected)
        receiveMessages()
        startHeartbeat()
    }
    
    func disconnect() {
        webSocket?.cancel(with: .goingAway, reason: nil)
        webSocket = nil
        onStatusChange?(.disconnected(nil))
    }
    
    func send(message: Message) async throws {
        let data = try JSONEncoder().encode(message)
        let string = String(data: data, encoding: .utf8)!
        try await webSocket?.send(.string(string))
    }
    
    private func receiveMessages() {
        webSocket?.receive { [weak self] result in
            switch result {
            case .success(let message):
                switch message {
                case .string(let text):
                    if let data = text.data(using: .utf8),
                       let message = try? JSONDecoder().decode(Message.self, from: data) {
                        DispatchQueue.main.async {
                            self?.onMessage?(message)
                        }
                    }
                case .data(let data):
                    break
                @unknown default:
                    break
                }
                self?.receiveMessages() // Continue receiving
                
            case .failure(let error):
                self?.onStatusChange?(.disconnected(error))
                // Implement reconnection logic
                self?.scheduleReconnect()
            }
        }
    }
    
    private func startHeartbeat() {
        Timer.scheduledTimer(withTimeInterval: 30, repeats: true) { [weak self] _ in
            self?.webSocket?.sendPing { error in
                if let error = error {
                    print("Heartbeat failed: \(error)")
                }
            }
        }
    }
    
    private func scheduleReconnect() {
        DispatchQueue.main.asyncAfter(deadline: .now() + 3.0) { [weak self] in
            // Exponential backoff reconnection
        }
    }
}

struct Message: Codable, Identifiable {
    let id: String
    let chatId: String
    let senderId: String
    let content: String
    let timestamp: Date
    var status: MessageStatus
}

enum MessageStatus: String, Codable {
    case sending, sent, delivered, read
}
```

---

## บทที่ 5: iOS-Specific Technical Depth Questions

### 5.1 Memory Management Deep Dive

```swift
// คำถาม: "อธิบาย Memory Management ใน Swift อย่างละเอียด"

/*
ANSWER:

1. ARC (Automatic Reference Counting)
- Swift ใช้ ARC สำหรับ class instances (reference types)
- Struct/Enum (value types) ไม่ใช้ ARC - copy on stack
- ARC track จำนวน strong references
- เมื่อ count = 0 → deallocate

2. Strong References (default)
- เพิ่ม reference count
- ป้องกัน deallocation

3. Weak References
- ไม่เพิ่ม reference count
- Optional - กลายเป็น nil เมื่อ object ถูก deallocate
- ใช้สำหรับ delegate patterns, parent-child relationships

4. Unowned References
- ไม่เพิ่ม reference count
- Non-optional - assume ไม่ nil (crash ถ้า nil)
- ใช้เมื่อ lifetime เท่ากัน หรือ child มีอายุน้อยกว่า parent
*/

// Unowned example: Credit Card และ Customer
class Customer {
    let name: String
    var card: CreditCard?
    
    init(name: String) { self.name = name }
    deinit { print("\(name) deallocated") }
}

class CreditCard {
    let number: Int
    unowned let customer: Customer  // card ไม่มีอายุยืนกว่า customer
    
    init(number: Int, customer: Customer) {
        self.number = number
        self.customer = customer
    }
    deinit { print("Card #\(number) deallocated") }
}

// Capture list ใน closures
class NetworkClient {
    var completionHandlers: [() -> Void] = []
    
    func performRequest() {
        // หลากหลาย capture semantics
        
        // 1. Strong capture (default) - อาจสร้าง retain cycle
        completionHandlers.append {
            print(self) // Strong reference - ถ้า completionHandlers อยู่ใน self จะ cycle
        }
        
        // 2. Weak capture - safe เมื่อ self อาจ nil
        completionHandlers.append { [weak self] in
            guard let self = self else { return }
            print(self)
        }
        
        // 3. Unowned capture - safe เมื่อ self ไม่เป็น nil แน่นอน
        completionHandlers.append { [unowned self] in
            print(self)
        }
    }
}

// Memory Graph ใน Xcode
// Debug → Memory Graph Debugger
// หา retain cycles ด้วย
// Edit Scheme → Diagnostics → Malloc Stack → All Allocation and Free History
```

### 5.2 Concurrency Race Condition Puzzles

```swift
// คำถาม: "Code นี้มี bug อะไร?"

class Counter {
    var count = 0
    
    func increment() {
        count += 1  // NOT thread-safe!
    }
}

// BUG: Race condition!
// Thread 1 reads count = 5
// Thread 2 reads count = 5
// Thread 1 writes count = 6
// Thread 2 writes count = 6
// Result: count = 6 แทนที่จะเป็น 7

// Solutions:

// Solution 1: DispatchQueue (simple)
class ThreadSafeCounter1 {
    private var count = 0
    private let queue = DispatchQueue(label: "counter.queue")
    
    func increment() {
        queue.sync { count += 1 }
    }
    
    var value: Int {
        queue.sync { count }
    }
}

// Solution 2: Actor (Swift 5.5+, recommended)
actor ThreadSafeCounter2 {
    private var count = 0
    
    func increment() {
        count += 1
    }
    
    var value: Int { count }
}

// Solution 3: OSAllocatedUnfairLock (iOS 16+, highest performance)
import os

class ThreadSafeCounter3 {
    private var count = 0
    private var lock = OSAllocatedUnfairLock()
    
    func increment() {
        lock.withLock { count += 1 }
    }
}

// Deadlock example:
class DeadlockDemo {
    let lockA = NSLock()
    let lockB = NSLock()
    
    func thread1() {
        lockA.lock()  // Thread 1 holds A
        Thread.sleep(forTimeInterval: 0.001)
        lockB.lock()  // Thread 1 waits for B
        // ...
        lockB.unlock()
        lockA.unlock()
    }
    
    func thread2() {
        lockB.lock()  // Thread 2 holds B
        Thread.sleep(forTimeInterval: 0.001)
        lockA.lock()  // Thread 2 waits for A → DEADLOCK!
        // ...
        lockA.unlock()
        lockB.unlock()
    }
}
```

### 5.3 SwiftUI Rendering Cycle

```swift
// คำถาม: "อธิบาย SwiftUI rendering cycle"

/*
SwiftUI Rendering Cycle:

1. State Change Trigger
   - @State, @StateObject, @ObservedObject, @EnvironmentObject
   - Parent view updates ที่มีผลต่อ child

2. Dependency Tracking
   - SwiftUI track dependencies อัตโนมัติ
   - View จะ re-render เฉพาะเมื่อ dependencies ของมันเปลี่ยน

3. Diff Algorithm
   - SwiftUI compare new view tree กับ old
   - Only update changed parts

4. Layout Pass
   - View proposals sizes to children
   - Children return accepted sizes

5. Render Pass
   - Draw updated views to Metal layer
*/

// Performance considerations:

struct ExpensiveView: View {
    let items: [String]
    
    // BAD: Recomputed every render
    var body: some View {
        List {
            ForEach(items, id: \.self) { item in
                Text(item)
                    .background(calculateComplexGradient()) // Expensive!
            }
        }
    }
    
    // GOOD: Cached
    private let gradient = calculateComplexGradient()
    
    var efficientBody: some View {
        List {
            ForEach(items, id: \.self) { item in
                Text(item)
                    .background(gradient) // Cached
            }
        }
    }
    
    private static func calculateComplexGradient() -> LinearGradient {
        LinearGradient(colors: [.blue, .purple], startPoint: .top, endPoint: .bottom)
    }
}

// Equatable optimization
struct OptimizedRow: View, Equatable {
    let title: String
    let subtitle: String
    
    // SwiftUI จะ skip re-render ถ้า props ไม่เปลี่ยน
    static func == (lhs: OptimizedRow, rhs: OptimizedRow) -> Bool {
        lhs.title == rhs.title && lhs.subtitle == rhs.subtitle
    }
    
    var body: some View {
        VStack(alignment: .leading) {
            Text(title)
            Text(subtitle)
        }
    }
}
```

---

## บทที่ 6: Take-Home Project Strategy

### 6.1 Architecture Decisions

```swift
// Take-Home Assignment: Build a News Reader App
// Timeline: 3-7 days

// Architecture Decision: Clean Architecture + MVVM

// Layer 1: Domain (Business Logic)
protocol NewsRepository {
    func fetchTopStories() async throws -> [Article]
    func fetchArticle(id: String) async throws -> Article
}

struct Article: Identifiable {
    let id: String
    let title: String
    let summary: String
    let imageURL: URL?
    let publishDate: Date
    let source: NewsSource
    var isBookmarked: Bool
}

// Layer 2: Data (Implementation)
class NewsRepositoryImpl: NewsRepository {
    private let apiClient: NewsAPIClient
    private let localStorage: ArticleStorage
    
    init(apiClient: NewsAPIClient, localStorage: ArticleStorage) {
        self.apiClient = apiClient
        self.localStorage = localStorage
    }
    
    func fetchTopStories() async throws -> [Article] {
        do {
            // Try network first
            let articles = try await apiClient.getTopStories()
            await localStorage.save(articles)
            return articles
        } catch {
            // Fallback to cache
            return await localStorage.getCachedArticles()
        }
    }
    
    func fetchArticle(id: String) async throws -> Article {
        try await apiClient.getArticle(id: id)
    }
}

// Layer 3: Presentation (ViewModel)
@MainActor
class ArticleListViewModel: ObservableObject {
    @Published var articles: [Article] = []
    @Published var state: ViewState = .idle
    
    private let repository: NewsRepository
    
    init(repository: NewsRepository) {
        self.repository = repository
    }
    
    enum ViewState {
        case idle, loading, success, failure(Error)
    }
    
    func loadArticles() async {
        state = .loading
        do {
            articles = try await repository.fetchTopStories()
            state = .success
        } catch {
            state = .failure(error)
        }
    }
}
```

### 6.2 Testing Strategy for Take-Home

```swift
// Testing ที่ประทับใจ interviewer:

// 1. Unit Tests สำหรับ Business Logic
class ArticleViewModelTests: XCTestCase {
    
    var viewModel: ArticleListViewModel!
    var mockRepository: MockNewsRepository!
    
    override func setUp() {
        mockRepository = MockNewsRepository()
        viewModel = ArticleListViewModel(repository: mockRepository)
    }
    
    func testLoadArticlesSuccess() async {
        // Arrange
        let expectedArticles = Article.mockArray(count: 5)
        mockRepository.stubbedArticles = expectedArticles
        
        // Act
        await viewModel.loadArticles()
        
        // Assert
        XCTAssertEqual(viewModel.articles.count, 5)
        if case .success = viewModel.state {} else {
            XCTFail("Expected success state")
        }
    }
    
    func testLoadArticlesFailure() async {
        // Arrange
        mockRepository.shouldThrow = true
        
        // Act
        await viewModel.loadArticles()
        
        // Assert
        XCTAssertTrue(viewModel.articles.isEmpty)
        if case .failure = viewModel.state {} else {
            XCTFail("Expected failure state")
        }
    }
    
    func testLoadingState() async {
        // Verify loading state is set
        let loadTask = Task { await viewModel.loadArticles() }
        
        // Check state immediately
        // (In real tests, use continuation or mock to control timing)
        
        await loadTask.value
    }
}

class MockNewsRepository: NewsRepository {
    var stubbedArticles: [Article] = []
    var shouldThrow = false
    
    func fetchTopStories() async throws -> [Article] {
        if shouldThrow { throw MockError.generic }
        return stubbedArticles
    }
    
    func fetchArticle(id: String) async throws -> Article {
        guard let article = stubbedArticles.first(where: { $0.id == id }) else {
            throw MockError.notFound
        }
        return article
    }
}

enum MockError: Error { case generic, notFound }

// 2. Integration Tests
class NewsAPIClientIntegrationTests: XCTestCase {
    
    func testFetchTopStoriesReturnsData() async throws {
        // Only run if API key available
        guard ProcessInfo.processInfo.environment["NEWS_API_KEY"] != nil else {
            throw XCTSkip("API key not available")
        }
        
        let client = NewsAPIClient()
        let articles = try await client.getTopStories()
        
        XCTAssertFalse(articles.isEmpty)
    }
}

// 3. Snapshot Tests (ถ้าใช้ swift-snapshot-testing)
// import SnapshotTesting
// class ArticleRowSnapshotTests: XCTestCase {
//     func testArticleRowAppearance() {
//         let view = ArticleRowView(article: .mock)
//         assertSnapshot(matching: view, as: .image)
//     }
// }
```

---

## บทที่ 7: Behavioral Interview Prep

### 7.1 STAR Format Stories

```
STAR = Situation, Task, Action, Result

สำหรับ iOS Developer: ต้องมี stories เกี่ยวกับ:
1. Performance problem ที่แก้ได้
2. Technical decision ที่ต้อง tradeoff
3. Conflict กับ PM หรือ designer
4. Feature ที่ deliver ได้ในเวลาจำกัด
5. Mentoring junior developer
```

**ตัวอย่าง STAR Story:**
```
คำถาม: "Tell me about a time when you improved app performance."

S (Situation): 
"ที่บริษัทเดิม app ของเรามี launch time 5 วินาทีบน iPhone 8 
ซึ่งทำให้ user retention ลดลง 15%"

T (Task): 
"ผมได้รับมอบหมายให้ลด launch time ให้เหลือต่ำกว่า 2 วินาที
โดยไม่กระทบ features ที่มีอยู่"

A (Action): 
"ผมใช้ Instruments profiler เพื่อหา bottleneck พบว่า:
1. App load โฆษณา SDK 3 ตัวพร้อมกันตอน launch
2. Core Data stack initialize synchronously บน main thread
3. User preferences โหลดจาก iCloud (network call) ตอน launch

ผมแก้ไขโดย:
1. Lazy load SDKs - โหลดเมื่อต้องการใช้จริง
2. ย้าย Core Data init ไป background thread
3. ใช้ cached preferences ก่อน แล้ว sync จาก iCloud ทีหลัง"

R (Result): 
"Launch time ลดจาก 5 วินาที เหลือ 1.8 วินาที (64% improvement)
User retention สัปดาห์แรกเพิ่มขึ้น 12%
App Store rating จาก 3.8 เป็น 4.3"
```

### 7.2 Amazon Leadership Principles

```
16 Leadership Principles ที่ควรเตรียม stories:

1. Customer Obsession → Story เกี่ยวกับการ prioritize user needs
2. Ownership → Story เกี่ยวกับการรับผิดชอบมากกว่า scope ของตัวเอง
3. Invent and Simplify → Story เกี่ยวกับการหา creative solution
4. Are Right, A Lot → Story เกี่ยวกับการตัดสินใจที่ถูกต้อง
5. Learn and Be Curious → Story เกี่ยวกับการเรียนรู้ skills ใหม่
6. Hire and Develop the Best → Story เกี่ยวกับการ mentor
7. Insist on the Highest Standards → Story เกี่ยวกับ code quality
8. Think Big → Story เกี่ยวกับ vision ที่ใหญ่กว่าที่ถูกขอ
9. Bias for Action → Story เกี่ยวกับการตัดสินใจเร็วโดยไม่รอข้อมูลทั้งหมด
10. Frugality → Story เกี่ยวกับการทำมากด้วย resources น้อย
11. Earn Trust → Story เกี่ยวกับการสร้าง trust กับ stakeholders
12. Dive Deep → Story เกี่ยวกับการ debug ปัญหาลึกๆ
13. Have Backbone; Disagree and Commit → Story เกี่ยวกับ respectful disagreement
14. Deliver Results → Story เกี่ยวกับ project ที่ deliver ได้ใน constraints
15. Strive to be Earth's Best Employer → (สำหรับ managers)
16. Success and Scale Bring Broad Responsibility → (senior roles)
```

---

## บทที่ 8: Compensation Negotiation

### 8.1 Compensation Structure

```
Apple IC4 (Senior) - San Francisco Bay Area (2024 estimates):
Base Salary: $175,000 - $220,000
RSU: $80,000 - $160,000 / year (4-year vesting)
Annual Bonus: 10-20% of base
Sign-on Bonus: $20,000 - $50,000
Total Compensation: ~$310,000 - $450,000

Google L5 (Senior) - San Francisco Bay Area:
Base: $185,000 - $230,000
RSU: $100,000 - $200,000 / year
Bonus: 15-25% of base
Total: ~$365,000 - $540,000

Meta E5 (Senior):
Base: $190,000 - $235,000
RSU: $120,000 - $240,000 / year
Bonus: 0-25% of base
Total: ~$410,000 - $580,000

Sources: levels.fyi, glassdoor, blind
```

### 8.2 Negotiation Scripts

```
Scenario 1: Initial Offer
---
HR: "We'd like to offer you a base salary of $180,000 with $80,000 in RSUs
     over 4 years and a $20,000 signing bonus."

You: "Thank you so much, I'm very excited about this opportunity.
     I do have a competing offer I need to consider. Could I have a couple
     of days to review? Also, is there flexibility in the package?
     Based on my research and the competing offer, I was expecting
     something closer to $200,000 base with $120,000 in annual RSUs."

HR: "We have some flexibility. Let me talk to the hiring manager."
---

Scenario 2: Using Competing Offers
---
"I have an offer from [Company X] for $[amount]. I'm genuinely more
excited about this role at [Company], but I want to make sure the
compensation reflects my market value. Is there any way you can
get closer to that offer?"
---

Scenario 3: RSU Negotiation
---
"I understand the base might be fixed, but would it be possible to
increase the initial RSU grant? Given my experience in [specific area],
I believe I can contribute significantly from day one and would love
to have more stake in the company's success."
---
```

---

## บทที่ 9: Rejection Analysis and Resilience

### 9.1 Learning from Failed Interviews

```
Post-Interview Analysis Template:

1. What went well?
   - Questions I answered confidently
   - Topics I knew deeply

2. What didn't go well?
   - Questions I struggled with
   - Topics where I got stuck

3. What should I study?
   - Specific algorithms/data structures
   - System design patterns
   - iOS APIs

4. What could I improve behavior-wise?
   - Did I communicate clearly?
   - Did I ask clarifying questions?
   - Did I think out loud?

5. Action items for next 2 weeks:
   - Study resources
   - Practice problems
   - Mock interviews
```

### 9.2 Interview Tracking Spreadsheet

```
สร้าง spreadsheet พร้อมคอลัมน์:

| Company | Role | Date | Round | Outcome | Notes | Follow-up Action |
|---------|------|------|-------|---------|-------|-----------------|
| Apple | Senior iOS | 2024-02-01 | Phone | Pass | Good on ARC | Study concurrency |
| Google | iOS L5 | 2024-02-08 | Phone | Pass | Weak on graphs | Practice BFS/DFS |
| Meta | iOS E5 | 2024-02-15 | On-site | Reject | Failed system design | Study at-scale design |
```

---

## บทที่ 10: 30-Day Interview Sprint Plan

```
WEEK 1: Foundation
Day 1-2: Arrays and Strings (10 problems)
  - Two Sum, Valid Palindrome, Merge Intervals
  - Group Anagrams, Longest Substring
Day 3-4: Linked Lists and Trees (10 problems)  
  - Reverse Linked List, Merge Two Sorted Lists
  - Level Order Traversal, Validate BST
Day 5-7: Dynamic Programming (10 problems)
  - Climbing Stairs, Coin Change, Longest Common Subsequence
  - Word Break, Decode Ways

WEEK 2: Intermediate
Day 8-9: Graphs (10 problems)
  - Number of Islands, Course Schedule
  - Pacific Atlantic, Clone Graph
Day 10-11: Binary Search (5 problems)
  - Search in Rotated Array, Find Peak Element
Day 12-13: Heap/Priority Queue (5 problems)
  - Kth Largest, Merge K Lists
Day 14: Weekly Review + Mock Interview

WEEK 3: iOS Deep Dive
Day 15-16: iOS System Design
  - Design Instagram Feed
  - Design Chat App
Day 17-18: iOS Technical Deep Dive
  - Memory management questions
  - Concurrency questions
  - SwiftUI rendering
Day 19-20: Take-home project practice
  - Build a small app with Clean Architecture
Day 21: Weekly Review + Mock Interview

WEEK 4: Polish
Day 22-23: Hard LeetCode (10 problems)
  - Trapping Rain Water, Regular Expression Matching
Day 24-25: Behavioral preparation
  - Write 10 STAR stories
  - Practice out loud
Day 26: Mock technical interview (find a partner)
Day 27: Mock behavioral interview
Day 28: Mock system design interview
Day 29: Review weaknesses
Day 30: Final review, rest well
```

---

## บทที่ 11: Mock Interview Session

### Full Mock Interview Simulation

**Interviewer**: "Let's start with a coding problem. Given an array of integers `nums` and an integer `k`, return the `k` most frequent elements."

**Optimal Approach:**

```swift
// LeetCode #347 - Top K Frequent Elements

func topKFrequent(_ nums: [Int], _ k: Int) -> [Int] {
    // Step 1: Count frequencies - O(n)
    var frequency: [Int: Int] = [:]
    for num in nums {
        frequency[num, default: 0] += 1
    }
    
    // Step 2: Bucket sort - O(n)
    // Index = frequency, value = list of numbers with that frequency
    var buckets = [[Int]](repeating: [], count: nums.count + 1)
    for (num, freq) in frequency {
        buckets[freq].append(num)
    }
    
    // Step 3: Collect top k - O(n)
    var result: [Int] = []
    for freq in stride(from: buckets.count - 1, through: 0, by: -1) {
        for num in buckets[freq] {
            result.append(num)
            if result.count == k {
                return result
            }
        }
    }
    return result
}

// Time: O(n), Space: O(n)
// Better than sorting approach O(n log n)!

// Alternative: Using Heap for O(n log k)
func topKFrequentHeap(_ nums: [Int], _ k: Int) -> [Int] {
    var frequency: [Int: Int] = [:]
    for num in nums { frequency[num, default: 0] += 1 }
    
    // Sort by frequency descending and take k
    return frequency
        .sorted { $0.value > $1.value }
        .prefix(k)
        .map { $0.key }
}

// Test cases to verify:
// topKFrequent([1,1,1,2,2,3], 2) → [1, 2]
// topKFrequent([1], 1) → [1]
// topKFrequent([1,2], 2) → [1, 2]
```

**iOS Technical Question:**
```swift
// Interviewer: "What happens when you call DispatchQueue.main.sync from the main thread?"

/*
Answer: DEADLOCK!

DispatchQueue.main.sync จาก main thread:
1. Main thread รอให้ block บน main queue execute
2. Main queue รอให้ main thread ว่าง
3. → Deadlock: both waiting for each other

Example:
*/

// This CRASHES:
func crashingCode() {
    DispatchQueue.main.sync {  // Called from main thread
        print("This will never print")  // Deadlock here!
    }
}

// SAFE: Use async หรือ check ว่าอยู่ thread ไหน
func safeCode() {
    if Thread.isMainThread {
        // Already on main thread, execute directly
        updateUI()
    } else {
        DispatchQueue.main.async {
            updateUI()
        }
    }
}

// Better pattern with modern Swift:
@MainActor
func updateUI() {
    // This always runs on main thread, no need to check
}
```

---

## บทที่ 12: Exercises with Solutions

### Exercise 1: Implement Stack-based Calculator

```swift
// Problem: Evaluate basic arithmetic expression: "3 + 2 * 4 - 1"
// LeetCode #227 Basic Calculator II

func calculate(_ s: String) -> Int {
    var stack: [Int] = []
    var currentNum = 0
    var lastOp: Character = "+"
    
    let str = s + "+"  // Sentinel to process last number
    
    for char in str {
        if char.isNumber {
            currentNum = currentNum * 10 + Int(String(char))!
        } else if char != " " {
            switch lastOp {
            case "+":
                stack.append(currentNum)
            case "-":
                stack.append(-currentNum)
            case "*":
                let top = stack.removeLast()
                stack.append(top * currentNum)
            case "/":
                let top = stack.removeLast()
                stack.append(top / currentNum)
            default:
                break
            }
            lastOp = char
            currentNum = 0
        }
    }
    
    return stack.reduce(0, +)
}

// Tests:
print(calculate("3+2*2"))     // 7
print(calculate(" 3/2 "))     // 1
print(calculate(" 3+5 / 2 ")) // 5
```

### Exercise 2: Design Pattern Quiz

```swift
// Problem: Implement Observer Pattern for iOS theming

// Observable Theme
final class ThemeManager: ObservableObject {
    static let shared = ThemeManager()
    
    @Published private(set) var currentTheme: Theme = .light
    
    func setTheme(_ theme: Theme) {
        currentTheme = theme
    }
    
    private init() {}
}

enum Theme {
    case light, dark, system
    
    var backgroundColor: UIColor {
        switch self {
        case .light: return .white
        case .dark: return .black
        case .system: return .systemBackground
        }
    }
    
    var textColor: UIColor {
        switch self {
        case .light: return .black
        case .dark: return .white
        case .system: return .label
        }
    }
}

// Usage in UIKit
class ThemedViewController: UIViewController {
    private var themeSubscription: AnyCancellable?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Observe theme changes
        themeSubscription = ThemeManager.shared.$currentTheme
            .sink { [weak self] theme in
                self?.applyTheme(theme)
            }
    }
    
    private func applyTheme(_ theme: Theme) {
        view.backgroundColor = theme.backgroundColor
        // Update other UI elements
    }
}

// Usage in SwiftUI
struct ThemedView: View {
    @ObservedObject private var themeManager = ThemeManager.shared
    
    var body: some View {
        Text("Hello")
            .foregroundColor(Color(themeManager.currentTheme.textColor))
            .background(Color(themeManager.currentTheme.backgroundColor))
    }
}
```

### Exercise 3: Concurrent Image Processor

```swift
// Design a concurrent image processing pipeline

import UIKit

actor ImageProcessor {
    
    typealias ImageTransform = (UIImage) -> UIImage
    
    private var cache: [URL: UIImage] = [:]
    
    func process(imageURL: URL, transforms: [ImageTransform]) async throws -> UIImage {
        // Check cache
        if let cached = cache[imageURL] {
            return applyTransforms(cached, transforms: transforms)
        }
        
        // Download
        let (data, _) = try await URLSession.shared.data(from: imageURL)
        guard let image = UIImage(data: data) else {
            throw ImageError.invalidData
        }
        
        // Cache original
        cache[imageURL] = image
        
        // Apply transforms
        return applyTransforms(image, transforms: transforms)
    }
    
    func processBatch(imageURLs: [URL], transforms: [ImageTransform]) async throws -> [UIImage] {
        // Process all images concurrently
        return try await withThrowingTaskGroup(of: (Int, UIImage).self) { group in
            for (index, url) in imageURLs.enumerated() {
                group.addTask {
                    let image = try await self.process(imageURL: url, transforms: transforms)
                    return (index, image)
                }
            }
            
            var results = [(Int, UIImage)]()
            for try await result in group {
                results.append(result)
            }
            
            return results
                .sorted { $0.0 < $1.0 }
                .map { $0.1 }
        }
    }
    
    private func applyTransforms(_ image: UIImage, transforms: [ImageTransform]) -> UIImage {
        transforms.reduce(image) { $1($0) }
    }
}

enum ImageError: Error { case invalidData }

// Common transforms
extension UIImage {
    static func resizeTransform(to size: CGSize) -> ImageProcessor.ImageTransform {
        return { image in
            UIGraphicsImageRenderer(size: size).image { _ in
                image.draw(in: CGRect(origin: .zero, size: size))
            }
        }
    }
    
    static func grayscaleTransform() -> ImageProcessor.ImageTransform {
        return { image in
            let context = CIContext()
            guard let ciImage = CIImage(image: image),
                  let filter = CIFilter(name: "CIColorControls") else {
                return image
            }
            
            filter.setValue(ciImage, forKey: kCIInputImageKey)
            filter.setValue(0.0, forKey: kCIInputSaturationKey)
            
            guard let output = filter.outputImage,
                  let cgImage = context.createCGImage(output, from: output.extent) else {
                return image
            }
            
            return UIImage(cgImage: cgImage)
        }
    }
}
```

### Exercise 4: Mock Interview Practice - Design a Rate Limiter

```swift
// System Design: Rate Limiter for iOS App
// Requirement: Allow 100 API calls per minute per user

class RateLimiter {
    
    struct Config {
        let maxRequests: Int
        let windowDuration: TimeInterval
        
        static let standard = Config(maxRequests: 100, windowDuration: 60)
    }
    
    private let config: Config
    private var requests: [Date] = []
    private let lock = NSLock()
    
    init(config: Config = .standard) {
        self.config = config
    }
    
    /// Returns true if the request is allowed, false if rate limited
    func allowRequest() -> Bool {
        lock.lock()
        defer { lock.unlock() }
        
        let now = Date()
        let windowStart = now.addingTimeInterval(-config.windowDuration)
        
        // Remove expired requests
        requests = requests.filter { $0 > windowStart }
        
        // Check limit
        if requests.count < config.maxRequests {
            requests.append(now)
            return true
        }
        return false
    }
    
    /// Returns time until next available slot
    var timeUntilReset: TimeInterval {
        lock.lock()
        defer { lock.unlock() }
        
        guard !requests.isEmpty else { return 0 }
        let windowStart = Date().addingTimeInterval(-config.windowDuration)
        let validRequests = requests.filter { $0 > windowStart }
        
        guard validRequests.count >= config.maxRequests,
              let oldest = validRequests.min() else { return 0 }
        
        return config.windowDuration - Date().timeIntervalSince(oldest)
    }
}

// Usage with async/await and retry
class APIClient {
    private let rateLimiter = RateLimiter()
    
    func performRequest<T: Decodable>(url: URL, type: T.Type) async throws -> T {
        guard rateLimiter.allowRequest() else {
            let waitTime = rateLimiter.timeUntilReset
            throw APIError.rateLimited(retryAfter: waitTime)
        }
        
        let (data, _) = try await URLSession.shared.data(from: url)
        return try JSONDecoder().decode(T.self, from: data)
    }
}

enum APIError: LocalizedError {
    case rateLimited(retryAfter: TimeInterval)
    
    var errorDescription: String? {
        switch self {
        case .rateLimited(let time):
            return "Rate limit exceeded. Please wait \(Int(time)) seconds."
        }
    }
}
```

---

## สรุป

การเตรียมตัวสัมภาษณ์บริษัทชั้นนำต้องการการลงทุนเวลาอย่างมีระบบ:

**Key Takeaways:**

1. **Algorithms**: เน้น patterns ไม่ใช่ท่องจำ - Two Pointers, BFS/DFS, DP, Sliding Window
2. **iOS Knowledge**: ลึกในทุก layer ตั้งแต่ ARC ถึง UIKit/SwiftUI internals
3. **System Design**: ใช้ RESHADED framework พร้อม iOS-specific considerations
4. **Behavioral**: เตรียม STAR stories ล่วงหน้า อย่างน้อย 10 stories
5. **Practice**: Mock interviews กับคนจริง สำคัญกว่า LeetCode เพียงอย่างเดียว

**Final Checklist ก่อน Interview:**
- [ ] Solve 50+ LeetCode problems (Medium level)
- [ ] Review iOS fundamentals ทุก topic
- [ ] Design 5+ systems using RESHADED
- [ ] Prepare 10+ STAR behavioral stories  
- [ ] Do 3+ mock interviews
- [ ] Research company culture และ products
- [ ] Prepare thoughtful questions สำหรับ interviewer
- [ ] Get good sleep ก่อน interview

**Resources:**
- LeetCode: leetcode.com
- levels.fyi: สำหรับ compensation benchmarking
- Swift Forums: forums.swift.org
- WWDC Sessions: developer.apple.com/wwdc
- Grokking the System Design Interview (Educative)
- "Cracking the Coding Interview" by Gayle Laakmann McDowell

---

*Part 86 จบแล้ว - ขอให้โชคดีในการสัมภาษณ์ครับ/ค่ะ!*

---

## ภาคผนวก: Quick Reference Card

```swift
// Big O Reference:
// O(1)    - Hash map lookup, array index
// O(log n) - Binary search, balanced BST
// O(n)    - Linear scan, single pass
// O(n log n) - Merge sort, heap sort
// O(n²)   - Nested loops, bubble sort
// O(2ⁿ)   - Recursive Fibonacci, subset generation

// Common Data Structures:
// Array    - Random access O(1), insert middle O(n)
// LinkedList - Insert/delete O(1), access O(n)
// HashMap  - Insert/lookup O(1) average, O(n) worst
// Stack    - LIFO, push/pop O(1)
// Queue    - FIFO, enqueue/dequeue O(1)
// Heap     - min/max O(1), insert O(log n)
// BST      - Search/insert O(log n) average
// Trie     - Insert/search O(m) where m = key length
// Graph    - BFS/DFS O(V+E)

// iOS Quick Reference:
// Main Thread  - UI updates, user interaction
// Background Thread - Network, heavy computation
// GCD          - Thread management, serial/concurrent queues
// Actor        - Thread-safe reference types (Swift 5.5+)
// @MainActor   - Ensures execution on main thread
// async/await  - Modern concurrency (Swift 5.5+)
// Combine      - Reactive programming
// SwiftUI      - Declarative UI (iOS 13+)
// UIKit        - Imperative UI (legacy but widely used)
```
