# Part 66: Algorithms ใน Swift

## บทนำ

Algorithm (อัลกอริทึม) คือชุดขั้นตอนที่ชัดเจนและสิ้นสุดสำหรับแก้ปัญหา การเรียนรู้ algorithms ที่สำคัญและวิธีนำไปใช้งานจะช่วยให้แก้ปัญหา programming ได้อย่างมีประสิทธิภาพ ในบทนี้เราจะครอบคลุม algorithms ที่สำคัญพร้อม implementation ใน Swift และตัวอย่างสำหรับการสัมภาษณ์งาน

---

## 1. Sorting Algorithms (อัลกอริทึมการเรียงลำดับ)

### 1.1 Bubble Sort

Bubble Sort เป็น algorithm ที่ง่ายที่สุด — เปรียบเทียบ element ที่อยู่ติดกันและสลับหากลำดับผิด

```swift
// MARK: - Bubble Sort - O(n²) time, O(1) space

func bubbleSort(_ array: inout [Int]) {
    let n = array.count
    for i in 0..<n {
        var swapped = false
        for j in 0..<(n - i - 1) {
            if array[j] > array[j + 1] {
                array.swapAt(j, j + 1)
                swapped = true
            }
        }
        if !swapped { break }  // Optimization: หยุดเมื่อไม่มีการสลับ
    }
}

var arr = [64, 34, 25, 12, 22, 11, 90]
bubbleSort(&arr)
print("Bubble Sort: \(arr)")  // [11, 12, 22, 25, 34, 64, 90]

// Generic version
func bubbleSortGeneric<T: Comparable>(_ array: inout [T]) {
    let n = array.count
    for i in 0..<n {
        for j in 0..<(n - i - 1) {
            if array[j] > array[j + 1] {
                array.swapAt(j, j + 1)
            }
        }
    }
}
```

### 1.2 Selection Sort

Selection Sort หา minimum element และวางในตำแหน่งที่ถูกต้อง

```swift
// MARK: - Selection Sort - O(n²) time, O(1) space

func selectionSort(_ array: inout [Int]) {
    let n = array.count
    for i in 0..<n {
        var minIdx = i
        for j in (i + 1)..<n {
            if array[j] < array[minIdx] {
                minIdx = j
            }
        }
        if minIdx != i {
            array.swapAt(i, minIdx)
        }
    }
}

var arr2 = [64, 25, 12, 22, 11]
selectionSort(&arr2)
print("Selection Sort: \(arr2)")  // [11, 12, 22, 25, 64]
```

### 1.3 Insertion Sort

Insertion Sort สร้าง sorted array ทีละ element — ดีสำหรับ nearly sorted data

```swift
// MARK: - Insertion Sort - O(n²) worst, O(n) best, O(1) space

func insertionSort(_ array: inout [Int]) {
    for i in 1..<array.count {
        let key = array[i]
        var j = i - 1
        while j >= 0 && array[j] > key {
            array[j + 1] = array[j]
            j -= 1
        }
        array[j + 1] = key
    }
}

var arr3 = [12, 11, 13, 5, 6]
insertionSort(&arr3)
print("Insertion Sort: \(arr3)")  // [5, 6, 11, 12, 13]

// Binary Insertion Sort - ใช้ binary search หาตำแหน่ง O(n log n) comparisons, O(n²) swaps
func binaryInsertionSort(_ array: inout [Int]) {
    for i in 1..<array.count {
        let key = array[i]
        var left = 0, right = i
        while left < right {
            let mid = left + (right - left) / 2
            if array[mid] > key { right = mid }
            else { left = mid + 1 }
        }
        for j in stride(from: i, through: left + 1, by: -1) {
            array[j] = array[j - 1]
        }
        array[left] = key
    }
}
```

### 1.4 Merge Sort

Merge Sort ใช้ divide and conquer — O(n log n) ทุกกรณี

```swift
// MARK: - Merge Sort - O(n log n) time, O(n) space

func mergeSort(_ array: [Int]) -> [Int] {
    guard array.count > 1 else { return array }
    
    let mid = array.count / 2
    let left = mergeSort(Array(array[..<mid]))
    let right = mergeSort(Array(array[mid...]))
    
    return merge(left, right)
}

private func merge(_ left: [Int], _ right: [Int]) -> [Int] {
    var result: [Int] = []
    var i = 0, j = 0
    
    while i < left.count && j < right.count {
        if left[i] <= right[j] {
            result.append(left[i])
            i += 1
        } else {
            result.append(right[j])
            j += 1
        }
    }
    
    result.append(contentsOf: left[i...])
    result.append(contentsOf: right[j...])
    return result
}

let sorted = mergeSort([38, 27, 43, 3, 9, 82, 10])
print("Merge Sort: \(sorted)")  // [3, 9, 10, 27, 38, 43, 82]

// In-place Merge Sort (ลด memory usage)
func mergeSortInPlace(_ array: inout [Int], _ left: Int, _ right: Int) {
    guard left < right else { return }
    let mid = left + (right - left) / 2
    mergeSortInPlace(&array, left, mid)
    mergeSortInPlace(&array, mid + 1, right)
    mergeInPlace(&array, left, mid, right)
}

private func mergeInPlace(_ array: inout [Int], _ left: Int, _ mid: Int, _ right: Int) {
    let leftPart = Array(array[left...mid])
    let rightPart = Array(array[(mid + 1)...right])
    var i = 0, j = 0, k = left
    
    while i < leftPart.count && j < rightPart.count {
        if leftPart[i] <= rightPart[j] {
            array[k] = leftPart[i]; i += 1
        } else {
            array[k] = rightPart[j]; j += 1
        }
        k += 1
    }
    while i < leftPart.count { array[k] = leftPart[i]; i += 1; k += 1 }
    while j < rightPart.count { array[k] = rightPart[j]; j += 1; k += 1 }
}
```

### 1.5 Quick Sort

Quick Sort ใช้ divide and conquer กับ pivot — O(n log n) average, O(n²) worst case

```swift
// MARK: - Quick Sort - O(n log n) average, O(n²) worst, O(log n) space

func quickSort(_ array: inout [Int], _ low: Int, _ high: Int) {
    guard low < high else { return }
    let pivot = partition(&array, low, high)
    quickSort(&array, low, pivot - 1)
    quickSort(&array, pivot + 1, high)
}

// Lomuto partition scheme
private func partition(_ array: inout [Int], _ low: Int, _ high: Int) -> Int {
    let pivot = array[high]
    var i = low - 1
    
    for j in low..<high {
        if array[j] <= pivot {
            i += 1
            array.swapAt(i, j)
        }
    }
    array.swapAt(i + 1, high)
    return i + 1
}

// Hoare partition (more efficient)
private func hoarePartition(_ array: inout [Int], _ low: Int, _ high: Int) -> Int {
    let pivot = array[(low + high) / 2]
    var i = low - 1
    var j = high + 1
    
    while true {
        repeat { i += 1 } while array[i] < pivot
        repeat { j -= 1 } while array[j] > pivot
        if i >= j { return j }
        array.swapAt(i, j)
    }
}

// 3-Way Quick Sort (Dutch National Flag) - ดีสำหรับ many duplicates
func quickSort3Way(_ array: inout [Int], _ low: Int, _ high: Int) {
    guard low < high else { return }
    var lt = low, gt = high, i = low + 1
    let pivot = array[low]
    
    while i <= gt {
        if array[i] < pivot {
            array.swapAt(lt, i)
            lt += 1; i += 1
        } else if array[i] > pivot {
            array.swapAt(i, gt)
            gt -= 1
        } else {
            i += 1
        }
    }
    quickSort3Way(&array, low, lt - 1)
    quickSort3Way(&array, gt + 1, high)
}

var arr4 = [10, 80, 30, 90, 40, 50, 70]
quickSort(&arr4, 0, arr4.count - 1)
print("Quick Sort: \(arr4)")

// Quick Select - หา k-th smallest ใน O(n) average
func quickSelect(_ array: inout [Int], _ low: Int, _ high: Int, _ k: Int) -> Int {
    if low == high { return array[low] }
    let pivot = partition(&array, low, high)
    if k == pivot { return array[pivot] }
    else if k < pivot { return quickSelect(&array, low, pivot - 1, k) }
    else { return quickSelect(&array, pivot + 1, high, k) }
}
```

### 1.6 Heap Sort

Heap Sort ใช้ heap data structure — O(n log n) time, O(1) space

```swift
// MARK: - Heap Sort - O(n log n) time, O(1) space

func heapSort(_ array: inout [Int]) {
    let n = array.count
    
    // Build max heap - O(n)
    for i in stride(from: n / 2 - 1, through: 0, by: -1) {
        siftDown(&array, i, n)
    }
    
    // Extract elements one by one - O(n log n)
    for i in stride(from: n - 1, through: 1, by: -1) {
        array.swapAt(0, i)
        siftDown(&array, 0, i)
    }
}

private func siftDown(_ array: inout [Int], _ root: Int, _ size: Int) {
    var largest = root
    let left = 2 * root + 1
    let right = 2 * root + 2
    
    if left < size && array[left] > array[largest] { largest = left }
    if right < size && array[right] > array[largest] { largest = right }
    
    if largest != root {
        array.swapAt(root, largest)
        siftDown(&array, largest, size)
    }
}

var arr5 = [12, 11, 13, 5, 6, 7]
heapSort(&arr5)
print("Heap Sort: \(arr5)")
```

### 1.7 Counting Sort และ Radix Sort

```swift
// MARK: - Counting Sort - O(n + k) time, k = range of input

func countingSort(_ array: [Int]) -> [Int] {
    guard !array.isEmpty else { return [] }
    let maxVal = array.max()!
    let minVal = array.min()!
    let range = maxVal - minVal + 1
    
    var count = Array(repeating: 0, count: range)
    for num in array {
        count[num - minVal] += 1
    }
    
    // Cumulative count
    for i in 1..<range {
        count[i] += count[i - 1]
    }
    
    var output = Array(repeating: 0, count: array.count)
    for num in array.reversed() {
        let idx = num - minVal
        count[idx] -= 1
        output[count[idx]] = num
    }
    return output
}

print("Counting Sort: \(countingSort([4, 2, 2, 8, 3, 3, 1]))")

// MARK: - Radix Sort - O(d * (n + k)) where d = digits, k = base

func radixSort(_ array: inout [Int]) {
    guard !array.isEmpty else { return }
    let maxVal = array.max()!
    var exp = 1
    while maxVal / exp > 0 {
        countingSortByDigit(&array, exp)
        exp *= 10
    }
}

private func countingSortByDigit(_ array: inout [Int], _ exp: Int) {
    let n = array.count
    var output = Array(repeating: 0, count: n)
    var count = Array(repeating: 0, count: 10)
    
    for num in array {
        count[(num / exp) % 10] += 1
    }
    for i in 1..<10 {
        count[i] += count[i - 1]
    }
    for i in stride(from: n - 1, through: 0, by: -1) {
        let digit = (array[i] / exp) % 10
        count[digit] -= 1
        output[count[digit]] = array[i]
    }
    array = output
}

var arr6 = [170, 45, 75, 90, 802, 24, 2, 66]
radixSort(&arr6)
print("Radix Sort: \(arr6)")
```

### สรุป Sorting Algorithms

| Algorithm | Best | Average | Worst | Space | Stable |
|-----------|------|---------|-------|-------|--------|
| Bubble | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Selection | O(n²) | O(n²) | O(n²) | O(1) | No |
| Insertion | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Merge | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes |
| Quick | O(n log n) | O(n log n) | O(n²) | O(log n) | No |
| Heap | O(n log n) | O(n log n) | O(n log n) | O(1) | No |
| Counting | O(n+k) | O(n+k) | O(n+k) | O(k) | Yes |
| Radix | O(nk) | O(nk) | O(nk) | O(n+k) | Yes |

---

## 2. Searching Algorithms (อัลกอริทึมการค้นหา)

### 2.1 Linear Search

```swift
// MARK: - Linear Search - O(n)

func linearSearch<T: Equatable>(_ array: [T], target: T) -> Int? {
    for (index, element) in array.enumerated() {
        if element == target { return index }
    }
    return nil
}

// หา all occurrences
func linearSearchAll<T: Equatable>(_ array: [T], target: T) -> [Int] {
    return array.enumerated().compactMap { $0.element == target ? $0.offset : nil }
}
```

### 2.2 Binary Search

```swift
// MARK: - Binary Search - O(log n) - requires sorted array

func binarySearch<T: Comparable>(_ array: [T], target: T) -> Int? {
    var left = 0
    var right = array.count - 1
    
    while left <= right {
        let mid = left + (right - left) / 2
        if array[mid] == target { return mid }
        else if array[mid] < target { left = mid + 1 }
        else { right = mid - 1 }
    }
    return nil
}

// Recursive version
func binarySearchRecursive<T: Comparable>(_ array: [T], target: T, left: Int, right: Int) -> Int? {
    guard left <= right else { return nil }
    let mid = left + (right - left) / 2
    if array[mid] == target { return mid }
    else if array[mid] < target { return binarySearchRecursive(array, target: target, left: mid + 1, right: right) }
    else { return binarySearchRecursive(array, target: target, left: left, right: mid - 1) }
}

// Find first/last occurrence
func findFirst(_ array: [Int], target: Int) -> Int? {
    var left = 0, right = array.count - 1, result: Int? = nil
    while left <= right {
        let mid = left + (right - left) / 2
        if array[mid] == target { result = mid; right = mid - 1 }
        else if array[mid] < target { left = mid + 1 }
        else { right = mid - 1 }
    }
    return result
}

func findLast(_ array: [Int], target: Int) -> Int? {
    var left = 0, right = array.count - 1, result: Int? = nil
    while left <= right {
        let mid = left + (right - left) / 2
        if array[mid] == target { result = mid; left = mid + 1 }
        else if array[mid] < target { left = mid + 1 }
        else { right = mid - 1 }
    }
    return result
}

// Search in rotated sorted array
func searchRotated(_ nums: [Int], _ target: Int) -> Int {
    var left = 0, right = nums.count - 1
    while left <= right {
        let mid = left + (right - left) / 2
        if nums[mid] == target { return mid }
        
        if nums[left] <= nums[mid] {  // left half is sorted
            if nums[left] <= target && target < nums[mid] { right = mid - 1 }
            else { left = mid + 1 }
        } else {  // right half is sorted
            if nums[mid] < target && target <= nums[right] { left = mid + 1 }
            else { right = mid - 1 }
        }
    }
    return -1
}

print(searchRotated([4,5,6,7,0,1,2], 0))  // 4
print(searchRotated([4,5,6,7,0,1,2], 3))  // -1
```

### 2.3 Jump Search

```swift
// MARK: - Jump Search - O(√n)

func jumpSearch(_ array: [Int], target: Int) -> Int? {
    let n = array.count
    let step = Int(Double(n).squareRoot())
    var prev = 0
    var curr = step
    
    while curr < n && array[curr] < target {
        prev = curr
        curr += step
    }
    
    // Linear search in the block
    for i in prev..<min(curr, n) {
        if array[i] == target { return i }
    }
    return nil
}
```

### 2.4 Interpolation Search

```swift
// MARK: - Interpolation Search - O(log log n) average for uniform distribution

func interpolationSearch(_ array: [Int], target: Int) -> Int? {
    var low = 0, high = array.count - 1
    
    while low <= high && target >= array[low] && target <= array[high] {
        if low == high {
            return array[low] == target ? low : nil
        }
        let pos = low + ((target - array[low]) * (high - low)) / (array[high] - array[low])
        if array[pos] == target { return pos }
        else if array[pos] < target { low = pos + 1 }
        else { high = pos - 1 }
    }
    return nil
}
```

### 2.5 Exponential Search

```swift
// MARK: - Exponential Search - O(log n) - ดีสำหรับ unbounded/infinite arrays

func exponentialSearch(_ array: [Int], target: Int) -> Int? {
    if array.isEmpty { return nil }
    if array[0] == target { return 0 }
    
    var i = 1
    while i < array.count && array[i] <= target { i *= 2 }
    
    let left = i / 2
    let right = min(i, array.count - 1)
    return binarySearch(Array(array[left...right]), target: target).map { $0 + left }
}
```

---

## 3. Graph Algorithms

### 3.1 Breadth-First Search (BFS)

```swift
// MARK: - BFS - O(V + E)

class GraphAlgorithms {
    // BFS - หาเส้นทางที่สั้นที่สุด (unweighted)
    static func bfs(graph: [Int: [Int]], start: Int) -> (distances: [Int: Int], parent: [Int: Int]) {
        var distances: [Int: Int] = [start: 0]
        var parent: [Int: Int] = [:]
        var queue: [Int] = [start]
        
        while !queue.isEmpty {
            let vertex = queue.removeFirst()
            for neighbor in graph[vertex] ?? [] {
                if distances[neighbor] == nil {
                    distances[neighbor] = distances[vertex]! + 1
                    parent[neighbor] = vertex
                    queue.append(neighbor)
                }
            }
        }
        return (distances, parent)
    }
    
    // Reconstruct path
    static func getPath(parent: [Int: Int], from start: Int, to end: Int) -> [Int] {
        var path: [Int] = []
        var current = end
        while current != start {
            path.append(current)
            guard let prev = parent[current] else { return [] }  // no path
            current = prev
        }
        path.append(start)
        return path.reversed()
    }
    
    // Multi-source BFS
    static func multiSourceBFS(graph: [Int: [Int]], sources: [Int]) -> [Int: Int] {
        var distances: [Int: Int] = [:]
        var queue: [Int] = sources
        for s in sources { distances[s] = 0 }
        
        while !queue.isEmpty {
            let vertex = queue.removeFirst()
            for neighbor in graph[vertex] ?? [] {
                if distances[neighbor] == nil {
                    distances[neighbor] = distances[vertex]! + 1
                    queue.append(neighbor)
                }
            }
        }
        return distances
    }
}
```

### 3.2 Depth-First Search (DFS)

```swift
// MARK: - DFS - O(V + E)

extension GraphAlgorithms {
    // Iterative DFS
    static func dfsIterative(graph: [Int: [Int]], start: Int) -> [Int] {
        var visited: Set<Int> = []
        var stack: [Int] = [start]
        var order: [Int] = []
        
        while !stack.isEmpty {
            let vertex = stack.removeLast()
            if !visited.contains(vertex) {
                visited.insert(vertex)
                order.append(vertex)
                for neighbor in (graph[vertex] ?? []).reversed() {
                    if !visited.contains(neighbor) {
                        stack.append(neighbor)
                    }
                }
            }
        }
        return order
    }
    
    // Detect cycle in directed graph
    static func hasCycleDirected(graph: [Int: [Int]], vertices: Int) -> Bool {
        var color = Array(repeating: 0, count: vertices)  // 0=white, 1=gray, 2=black
        
        func dfs(_ v: Int) -> Bool {
            color[v] = 1
            for neighbor in graph[v] ?? [] {
                if color[neighbor] == 1 { return true }  // back edge = cycle
                if color[neighbor] == 0 && dfs(neighbor) { return true }
            }
            color[v] = 2
            return false
        }
        
        for v in 0..<vertices {
            if color[v] == 0 && dfs(v) { return true }
        }
        return false
    }
    
    // Find all paths
    static func allPaths(graph: [Int: [Int]], start: Int, end: Int) -> [[Int]] {
        var results: [[Int]] = []
        var path: [Int] = [start]
        
        func dfs(_ vertex: Int) {
            if vertex == end {
                results.append(path)
                return
            }
            for neighbor in graph[vertex] ?? [] {
                if !path.contains(neighbor) {
                    path.append(neighbor)
                    dfs(neighbor)
                    path.removeLast()
                }
            }
        }
        dfs(start)
        return results
    }
    
    // Connected components
    static func connectedComponents(graph: [Int: [Int]], vertices: Int) -> Int {
        var visited: Set<Int> = []
        var components = 0
        
        func dfs(_ v: Int) {
            visited.insert(v)
            for neighbor in graph[v] ?? [] where !visited.contains(neighbor) {
                dfs(neighbor)
            }
        }
        
        for v in 0..<vertices {
            if !visited.contains(v) {
                dfs(v)
                components += 1
            }
        }
        return components
    }
}
```

### 3.3 Dijkstra's Algorithm

```swift
// MARK: - Dijkstra's Algorithm - O((V + E) log V)

struct DijkstraState: Comparable {
    let vertex: Int
    let distance: Int
    static func < (lhs: DijkstraState, rhs: DijkstraState) -> Bool {
        return lhs.distance < rhs.distance
    }
}

extension GraphAlgorithms {
    static func dijkstra(graph: [Int: [(vertex: Int, weight: Int)]], start: Int, vertices: Int) -> [Int] {
        var dist = Array(repeating: Int.max, count: vertices)
        dist[start] = 0
        var pq = [DijkstraState(vertex: start, distance: 0)]
        
        while !pq.isEmpty {
            pq.sort()  // ในทางปฏิบัติควรใช้ priority queue จริงๆ
            let current = pq.removeFirst()
            
            if current.distance > dist[current.vertex] { continue }
            
            for (neighbor, weight) in graph[current.vertex] ?? [] {
                let newDist = dist[current.vertex] + weight
                if newDist < dist[neighbor] {
                    dist[neighbor] = newDist
                    pq.append(DijkstraState(vertex: neighbor, distance: newDist))
                }
            }
        }
        return dist
    }
    
    // Network Delay Time (LeetCode 743)
    static func networkDelayTime(_ times: [[Int]], _ n: Int, _ k: Int) -> Int {
        var graph: [Int: [(Int, Int)]] = [:]
        for time in times {
            graph[time[0], default: []].append((time[1], time[2]))
        }
        
        var dist = Array(repeating: Int.max, count: n + 1)
        dist[k] = 0
        var pq = [(0, k)]  // (distance, vertex)
        
        while !pq.isEmpty {
            pq.sort { $0.0 < $1.0 }
            let (d, u) = pq.removeFirst()
            if d > dist[u] { continue }
            
            for (v, w) in graph[u] ?? [] {
                let newDist = d + w
                if newDist < dist[v] {
                    dist[v] = newDist
                    pq.append((newDist, v))
                }
            }
        }
        
        let maxDist = dist[1...n].max()!
        return maxDist == Int.max ? -1 : maxDist
    }
}
```

### 3.4 Bellman-Ford Algorithm

```swift
// MARK: - Bellman-Ford - O(VE), handles negative weights

extension GraphAlgorithms {
    struct Edge {
        let from, to, weight: Int
    }
    
    static func bellmanFord(edges: [Edge], vertices: Int, start: Int) -> [Int]? {
        var dist = Array(repeating: Int.max, count: vertices)
        dist[start] = 0
        
        // Relax edges V-1 times
        for _ in 0..<(vertices - 1) {
            for edge in edges {
                if dist[edge.from] != Int.max &&
                   dist[edge.from] + edge.weight < dist[edge.to] {
                    dist[edge.to] = dist[edge.from] + edge.weight
                }
            }
        }
        
        // Detect negative cycle
        for edge in edges {
            if dist[edge.from] != Int.max &&
               dist[edge.from] + edge.weight < dist[edge.to] {
                return nil  // negative cycle exists
            }
        }
        
        return dist
    }
}
```

### 3.5 Floyd-Warshall Algorithm

```swift
// MARK: - Floyd-Warshall - O(V³), all pairs shortest paths

extension GraphAlgorithms {
    static func floydWarshall(_ matrix: [[Int]]) -> [[Int]] {
        let n = matrix.count
        var dist = matrix
        
        for k in 0..<n {
            for i in 0..<n {
                for j in 0..<n {
                    if dist[i][k] != Int.max && dist[k][j] != Int.max {
                        dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])
                    }
                }
            }
        }
        return dist
    }
}

// A* Algorithm - Informed BFS with heuristic
struct AStarNode: Comparable {
    let position: (Int, Int)
    let g: Int  // cost from start
    let h: Int  // heuristic to goal
    var f: Int { g + h }
    
    static func < (lhs: AStarNode, rhs: AStarNode) -> Bool {
        return lhs.f < rhs.f
    }
}

func aStar(grid: [[Int]], start: (Int, Int), goal: (Int, Int)) -> Int? {
    let rows = grid.count, cols = grid[0].count
    
    func heuristic(_ pos: (Int, Int)) -> Int {
        return abs(pos.0 - goal.0) + abs(pos.1 - goal.1)  // Manhattan distance
    }
    
    var openSet = [AStarNode(position: start, g: 0, h: heuristic(start))]
    var gScore: [String: Int] = ["\(start.0),\(start.1)": 0]
    let directions = [(0,1),(0,-1),(1,0),(-1,0)]
    
    while !openSet.isEmpty {
        openSet.sort()
        let current = openSet.removeFirst()
        
        if current.position == goal { return current.g }
        
        for (dr, dc) in directions {
            let nr = current.position.0 + dr
            let nc = current.position.1 + dc
            guard nr >= 0 && nr < rows && nc >= 0 && nc < cols && grid[nr][nc] == 0 else { continue }
            
            let newG = current.g + 1
            let key = "\(nr),\(nc)"
            if newG < (gScore[key] ?? Int.max) {
                gScore[key] = newG
                openSet.append(AStarNode(position: (nr, nc), g: newG, h: heuristic((nr, nc))))
            }
        }
    }
    return nil  // no path
}
```

---

## 4. Dynamic Programming (การเขียนโปรแกรมแบบไดนามิก)

Dynamic Programming แก้ปัญหาโดยแบ่งเป็นปัญหาย่อย และเก็บผลลัพธ์เพื่อหลีกเลี่ยงการคำนวณซ้ำ

### 4.1 Memoization (Top-Down)

```swift
// MARK: - Memoization

// Fibonacci ด้วย Memoization - O(n) time, O(n) space
func fibMemo(_ n: Int, memo: inout [Int: Int]) -> Int {
    if n <= 1 { return n }
    if let cached = memo[n] { return cached }
    let result = fibMemo(n - 1, memo: &memo) + fibMemo(n - 2, memo: &memo)
    memo[n] = result
    return result
}

var memo: [Int: Int] = [:]
print("Fib(10): \(fibMemo(10, memo: &memo))")  // 55
print("Fib(40): \(fibMemo(40, memo: &memo))")  // 102334155
```

### 4.2 Tabulation (Bottom-Up)

```swift
// MARK: - Tabulation

// Fibonacci ด้วย Tabulation - O(n) time, O(n) space
func fibTab(_ n: Int) -> Int {
    if n <= 1 { return n }
    var dp = Array(repeating: 0, count: n + 1)
    dp[1] = 1
    for i in 2...n {
        dp[i] = dp[i-1] + dp[i-2]
    }
    return dp[n]
}

// Space-optimized Fibonacci - O(n) time, O(1) space
func fibOptimized(_ n: Int) -> Int {
    if n <= 1 { return n }
    var prev = 0, curr = 1
    for _ in 2...n {
        let next = prev + curr
        prev = curr
        curr = next
    }
    return curr
}
```

### 4.3 Classic DP Problems

```swift
// MARK: - Coin Change Problem

// Minimum coins to make amount
func coinChange(_ coins: [Int], _ amount: Int) -> Int {
    var dp = Array(repeating: amount + 1, count: amount + 1)
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

print("Coin change [1,5,11] -> 15: \(coinChange([1,5,11], 15))")  // 3 (11+3*1 or 3*5)

// Count ways to make change
func changeWays(_ amount: Int, _ coins: [Int]) -> Int {
    var dp = Array(repeating: 0, count: amount + 1)
    dp[0] = 1
    for coin in coins {
        for i in coin...amount {
            dp[i] += dp[i - coin]
        }
    }
    return dp[amount]
}

// MARK: - 0/1 Knapsack Problem

func knapsack(_ weights: [Int], _ values: [Int], _ capacity: Int) -> Int {
    let n = weights.count
    var dp = Array(repeating: Array(repeating: 0, count: capacity + 1), count: n + 1)
    
    for i in 1...n {
        for w in 0...capacity {
            dp[i][w] = dp[i-1][w]  // don't take item i
            if weights[i-1] <= w {
                dp[i][w] = max(dp[i][w], dp[i-1][w - weights[i-1]] + values[i-1])
            }
        }
    }
    return dp[n][capacity]
}

let weights = [1, 3, 4, 5]
let values = [1, 4, 5, 7]
print("Knapsack (capacity=7): \(knapsack(weights, values, 7))")  // 9

// Space-optimized knapsack
func knapsack1D(_ weights: [Int], _ values: [Int], _ capacity: Int) -> Int {
    var dp = Array(repeating: 0, count: capacity + 1)
    for i in 0..<weights.count {
        for w in stride(from: capacity, through: weights[i], by: -1) {
            dp[w] = max(dp[w], dp[w - weights[i]] + values[i])
        }
    }
    return dp[capacity]
}

// MARK: - Longest Common Subsequence (LCS)

func lcs(_ s1: String, _ s2: String) -> Int {
    let a = Array(s1), b = Array(s2)
    let m = a.count, n = b.count
    var dp = Array(repeating: Array(repeating: 0, count: n + 1), count: m + 1)
    
    for i in 1...m {
        for j in 1...n {
            if a[i-1] == b[j-1] {
                dp[i][j] = dp[i-1][j-1] + 1
            } else {
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
            }
        }
    }
    return dp[m][n]
}

// Reconstruct LCS
func getLCS(_ s1: String, _ s2: String) -> String {
    let a = Array(s1), b = Array(s2)
    let m = a.count, n = b.count
    var dp = Array(repeating: Array(repeating: 0, count: n + 1), count: m + 1)
    
    for i in 1...m {
        for j in 1...n {
            dp[i][j] = a[i-1] == b[j-1] ? dp[i-1][j-1] + 1 : max(dp[i-1][j], dp[i][j-1])
        }
    }
    
    var result: [Character] = []
    var i = m, j = n
    while i > 0 && j > 0 {
        if a[i-1] == b[j-1] {
            result.append(a[i-1])
            i -= 1; j -= 1
        } else if dp[i-1][j] > dp[i][j-1] {
            i -= 1
        } else {
            j -= 1
        }
    }
    return String(result.reversed())
}

print("LCS('ABCBDAB', 'BDCAB'): \(lcs("ABCBDAB", "BDCAB"))")  // 4

// MARK: - Longest Increasing Subsequence (LIS)

func lis(_ nums: [Int]) -> Int {
    if nums.isEmpty { return 0 }
    var dp = Array(repeating: 1, count: nums.count)
    
    for i in 1..<nums.count {
        for j in 0..<i {
            if nums[j] < nums[i] {
                dp[i] = max(dp[i], dp[j] + 1)
            }
        }
    }
    return dp.max()!
}

// O(n log n) LIS using patience sort
func lisOptimized(_ nums: [Int]) -> Int {
    var tails: [Int] = []
    for num in nums {
        var left = 0, right = tails.count
        while left < right {
            let mid = left + (right - left) / 2
            if tails[mid] < num { left = mid + 1 }
            else { right = mid }
        }
        if left == tails.count { tails.append(num) }
        else { tails[left] = num }
    }
    return tails.count
}

print("LIS [10,9,2,5,3,7,101,18]: \(lisOptimized([10,9,2,5,3,7,101,18]))")  // 4

// MARK: - Edit Distance (Levenshtein Distance)

func editDistance(_ word1: String, _ word2: String) -> Int {
    let a = Array(word1), b = Array(word2)
    let m = a.count, n = b.count
    var dp = Array(repeating: Array(repeating: 0, count: n + 1), count: m + 1)
    
    for i in 0...m { dp[i][0] = i }
    for j in 0...n { dp[0][j] = j }
    
    for i in 1...m {
        for j in 1...n {
            if a[i-1] == b[j-1] {
                dp[i][j] = dp[i-1][j-1]
            } else {
                dp[i][j] = 1 + min(dp[i-1][j],    // delete
                                   dp[i][j-1],    // insert
                                   dp[i-1][j-1])  // replace
            }
        }
    }
    return dp[m][n]
}

print("Edit distance 'horse' -> 'ros': \(editDistance("horse", "ros"))")  // 3

// MARK: - Matrix Chain Multiplication

func matrixChain(_ dims: [Int]) -> Int {
    let n = dims.count - 1
    var dp = Array(repeating: Array(repeating: 0, count: n), count: n)
    
    for len in 2...n {
        for i in 0...(n - len) {
            let j = i + len - 1
            dp[i][j] = Int.max
            for k in i..<j {
                let cost = dp[i][k] + dp[k+1][j] + dims[i] * dims[k+1] * dims[j+1]
                dp[i][j] = min(dp[i][j], cost)
            }
        }
    }
    return dp[0][n-1]
}

// MARK: - Unique Paths

func uniquePaths(_ m: Int, _ n: Int) -> Int {
    var dp = Array(repeating: Array(repeating: 1, count: n), count: m)
    for i in 1..<m {
        for j in 1..<n {
            dp[i][j] = dp[i-1][j] + dp[i][j-1]
        }
    }
    return dp[m-1][n-1]
}

// MARK: - Maximum Subarray (Kadane's Algorithm)

func maxSubarray(_ nums: [Int]) -> Int {
    var maxSum = nums[0]
    var currentSum = nums[0]
    
    for i in 1..<nums.count {
        currentSum = max(nums[i], currentSum + nums[i])
        maxSum = max(maxSum, currentSum)
    }
    return maxSum
}

print("Max subarray [-2,1,-3,4,-1,2,1,-5,4]: \(maxSubarray([-2,1,-3,4,-1,2,1,-5,4]))")  // 6

// MARK: - Palindrome Partition

func minCutPalindrome(_ s: String) -> Int {
    let chars = Array(s)
    let n = chars.count
    var isPalin = Array(repeating: Array(repeating: false, count: n), count: n)
    var dp = Array(repeating: 0, count: n)
    
    for i in 0..<n {
        dp[i] = i  // worst case: cut at every character
        for j in 0...i {
            if chars[j] == chars[i] && (i - j <= 2 || isPalin[j+1][i-1]) {
                isPalin[j][i] = true
                dp[i] = j == 0 ? 0 : min(dp[i], dp[j-1] + 1)
            }
        }
    }
    return dp[n-1]
}
```

---

## 5. Greedy Algorithms (อัลกอริทึมแบบโลภ)

Greedy algorithm เลือก optimal choice ในแต่ละขั้นตอน โดยหวังว่าจะได้ผลลัพธ์ที่ optimal โดยรวม

```swift
// MARK: - Greedy Algorithms

// Activity Selection Problem
func activitySelection(_ start: [Int], _ end: [Int]) -> [Int] {
    let n = start.count
    var activities = (0..<n).sorted { end[$0] < end[$1] }
    var selected: [Int] = [activities[0]]
    var lastEnd = end[activities[0]]
    
    for i in 1..<n {
        let activity = activities[i]
        if start[activity] >= lastEnd {
            selected.append(activity)
            lastEnd = end[activity]
        }
    }
    return selected
}

// Fractional Knapsack
func fractionalKnapsack(_ weights: [Double], _ values: [Double], _ capacity: Double) -> Double {
    let items = zip(weights, values)
        .map { ($0.0, $0.1, $0.1 / $0.0) }  // (weight, value, ratio)
        .sorted { $0.2 > $1.2 }
    
    var totalValue = 0.0
    var remaining = capacity
    
    for (weight, value, _) in items {
        if remaining <= 0 { break }
        if weight <= remaining {
            totalValue += value
            remaining -= weight
        } else {
            totalValue += value * (remaining / weight)
            remaining = 0
        }
    }
    return totalValue
}

// Jump Game - can reach end?
func canJump(_ nums: [Int]) -> Bool {
    var maxReach = 0
    for i in 0..<nums.count {
        if i > maxReach { return false }
        maxReach = max(maxReach, i + nums[i])
    }
    return true
}

// Jump Game II - minimum jumps
func jumpMinimum(_ nums: [Int]) -> Int {
    var jumps = 0, currentEnd = 0, farthest = 0
    for i in 0..<(nums.count - 1) {
        farthest = max(farthest, i + nums[i])
        if i == currentEnd {
            jumps += 1
            currentEnd = farthest
        }
    }
    return jumps
}

// Meeting Rooms II (minimum rooms needed)
func minMeetingRooms(_ intervals: [[Int]]) -> Int {
    let starts = intervals.map { $0[0] }.sorted()
    let ends = intervals.map { $0[1] }.sorted()
    var rooms = 0, maxRooms = 0, j = 0
    
    for i in 0..<starts.count {
        if starts[i] < ends[j] {
            rooms += 1
        } else {
            j += 1
        }
        maxRooms = max(maxRooms, rooms)
    }
    return maxRooms
}

// Gas Station
func canCompleteCircuit(_ gas: [Int], _ cost: [Int]) -> Int {
    let totalGas = gas.reduce(0, +)
    let totalCost = cost.reduce(0, +)
    guard totalGas >= totalCost else { return -1 }
    
    var tank = 0, start = 0
    for i in 0..<gas.count {
        tank += gas[i] - cost[i]
        if tank < 0 {
            start = i + 1
            tank = 0
        }
    }
    return start
}

// Huffman Encoding (Greedy compression)
class HuffmanNode: Comparable {
    var char: Character?
    var freq: Int
    var left: HuffmanNode?
    var right: HuffmanNode?
    
    init(char: Character? = nil, freq: Int) {
        self.char = char
        self.freq = freq
    }
    
    static func < (lhs: HuffmanNode, rhs: HuffmanNode) -> Bool {
        return lhs.freq < rhs.freq
    }
    
    static func == (lhs: HuffmanNode, rhs: HuffmanNode) -> Bool {
        return lhs.freq == rhs.freq
    }
}

func buildHuffmanTree(_ text: String) -> HuffmanNode? {
    var freq: [Character: Int] = [:]
    for char in text { freq[char, default: 0] += 1 }
    
    var heap = freq.map { HuffmanNode(char: $0.key, freq: $0.value) }.sorted()
    
    while heap.count > 1 {
        let left = heap.removeFirst()
        let right = heap.removeFirst()
        let merged = HuffmanNode(freq: left.freq + right.freq)
        merged.left = left
        merged.right = right
        heap.append(merged)
        heap.sort()
    }
    return heap.first
}

func getCodes(_ node: HuffmanNode?, _ code: String, _ codes: inout [Character: String]) {
    guard let node = node else { return }
    if let char = node.char {
        codes[char] = code.isEmpty ? "0" : code
        return
    }
    getCodes(node.left, code + "0", &codes)
    getCodes(node.right, code + "1", &codes)
}
```

---

## 6. Divide and Conquer (แบ่งแล้วพิชิต)

```swift
// MARK: - Divide and Conquer

// Maximum Subarray (Divide and Conquer approach)
func maxSubarrayDC(_ nums: [Int], _ left: Int, _ right: Int) -> Int {
    if left == right { return nums[left] }
    
    let mid = left + (right - left) / 2
    let leftMax = maxSubarrayDC(nums, left, mid)
    let rightMax = maxSubarrayDC(nums, mid + 1, right)
    let crossMax = maxCrossing(nums, left, mid, right)
    
    return max(leftMax, rightMax, crossMax)
}

private func maxCrossing(_ nums: [Int], _ left: Int, _ mid: Int, _ right: Int) -> Int {
    var leftSum = Int.min, sum = 0
    for i in stride(from: mid, through: left, by: -1) {
        sum += nums[i]
        leftSum = max(leftSum, sum)
    }
    var rightSum = Int.min
    sum = 0
    for i in (mid + 1)...right {
        sum += nums[i]
        rightSum = max(rightSum, sum)
    }
    return leftSum + rightSum
}

// Count inversions (modified merge sort)
func countInversions(_ array: [Int]) -> Int {
    var count = 0
    func mergeCount(_ arr: [Int]) -> [Int] {
        if arr.count <= 1 { return arr }
        let mid = arr.count / 2
        let left = mergeCount(Array(arr[..<mid]))
        let right = mergeCount(Array(arr[mid...]))
        var result: [Int] = []
        var i = 0, j = 0
        while i < left.count && j < right.count {
            if left[i] <= right[j] {
                result.append(left[i]); i += 1
            } else {
                count += left.count - i  // all remaining left elements form inversions
                result.append(right[j]); j += 1
            }
        }
        result.append(contentsOf: left[i...])
        result.append(contentsOf: right[j...])
        return result
    }
    _ = mergeCount(array)
    return count
}

// Closest Pair of Points
func closestPair(_ points: [(Double, Double)]) -> Double {
    func dist(_ p1: (Double, Double), _ p2: (Double, Double)) -> Double {
        let dx = p1.0 - p2.0, dy = p1.1 - p2.1
        return (dx*dx + dy*dy).squareRoot()
    }
    
    func closestRec(_ pts: [(Double, Double)]) -> Double {
        if pts.count <= 3 {
            var minD = Double.infinity
            for i in 0..<pts.count {
                for j in (i+1)..<pts.count {
                    minD = min(minD, dist(pts[i], pts[j]))
                }
            }
            return minD
        }
        
        let mid = pts.count / 2
        let midX = pts[mid].0
        var d = min(closestRec(Array(pts[..<mid])), closestRec(Array(pts[mid...])))
        
        let strip = pts.filter { abs($0.0 - midX) < d }
        let sortedStrip = strip.sorted { $0.1 < $1.1 }
        
        for i in 0..<sortedStrip.count {
            for j in (i+1)..<sortedStrip.count where sortedStrip[j].1 - sortedStrip[i].1 < d {
                d = min(d, dist(sortedStrip[i], sortedStrip[j]))
            }
        }
        return d
    }
    
    let sorted = points.sorted { $0.0 < $1.0 }
    return closestRec(sorted)
}
```

---

## 7. Backtracking (การย้อนรอย)

```swift
// MARK: - Backtracking

// N-Queens Problem
func solveNQueens(_ n: Int) -> [[String]] {
    var results: [[String]] = []
    var board = Array(repeating: Array(repeating: ".", count: n), count: n)
    
    func isValid(_ row: Int, _ col: Int) -> Bool {
        // Check column
        for i in 0..<row { if board[i][col] == "Q" { return false } }
        // Check upper-left diagonal
        var i = row - 1, j = col - 1
        while i >= 0 && j >= 0 { if board[i][j] == "Q" { return false }; i -= 1; j -= 1 }
        // Check upper-right diagonal
        i = row - 1; j = col + 1
        while i >= 0 && j < n { if board[i][j] == "Q" { return false }; i -= 1; j += 1 }
        return true
    }
    
    func backtrack(_ row: Int) {
        if row == n {
            results.append(board.map { String($0) })
            return
        }
        for col in 0..<n {
            if isValid(row, col) {
                board[row][col] = "Q"
                backtrack(row + 1)
                board[row][col] = "."
            }
        }
    }
    
    backtrack(0)
    return results
}

print("4-Queens solutions: \(solveNQueens(4).count)")  // 2

// Sudoku Solver
func solveSudoku(_ board: inout [[Character]]) {
    func isValid(_ board: [[Character]], _ row: Int, _ col: Int, _ num: Character) -> Bool {
        let boxRow = (row / 3) * 3, boxCol = (col / 3) * 3
        for i in 0..<9 {
            if board[row][i] == num || board[i][col] == num { return false }
            if board[boxRow + i/3][boxCol + i%3] == num { return false }
        }
        return true
    }
    
    func solve(_ board: inout [[Character]]) -> Bool {
        for i in 0..<9 {
            for j in 0..<9 {
                if board[i][j] == "." {
                    for num: Character in ["1","2","3","4","5","6","7","8","9"] {
                        if isValid(board, i, j, num) {
                            board[i][j] = num
                            if solve(&board) { return true }
                            board[i][j] = "."
                        }
                    }
                    return false
                }
            }
        }
        return true
    }
    _ = solve(&board)
}

// Generate Permutations
func permutations(_ nums: [Int]) -> [[Int]] {
    var results: [[Int]] = []
    var current: [Int] = []
    var used = Array(repeating: false, count: nums.count)
    
    func backtrack() {
        if current.count == nums.count {
            results.append(current)
            return
        }
        for i in 0..<nums.count {
            if !used[i] {
                used[i] = true
                current.append(nums[i])
                backtrack()
                current.removeLast()
                used[i] = false
            }
        }
    }
    
    backtrack()
    return results
}

// Generate Combinations
func combinations(_ n: Int, _ k: Int) -> [[Int]] {
    var results: [[Int]] = []
    var current: [Int] = []
    
    func backtrack(_ start: Int) {
        if current.count == k {
            results.append(current)
            return
        }
        for i in start...n {
            current.append(i)
            backtrack(i + 1)
            current.removeLast()
        }
    }
    
    backtrack(1)
    return results
}

// Word Search
func exist(_ board: [[Character]], _ word: String) -> Bool {
    let rows = board.count, cols = board[0].count
    let chars = Array(word)
    var board = board  // mutable copy
    
    func dfs(_ r: Int, _ c: Int, _ idx: Int) -> Bool {
        if idx == chars.count { return true }
        if r < 0 || r >= rows || c < 0 || c >= cols || board[r][c] != chars[idx] { return false }
        
        let temp = board[r][c]
        board[r][c] = "#"
        let found = dfs(r+1,c,idx+1) || dfs(r-1,c,idx+1) || dfs(r,c+1,idx+1) || dfs(r,c-1,idx+1)
        board[r][c] = temp
        return found
    }
    
    for r in 0..<rows {
        for c in 0..<cols {
            if dfs(r, c, 0) { return true }
        }
    }
    return false
}
```

---

## 8. Two-Pointer Technique

```swift
// MARK: - Two Pointer

// Two Sum (sorted array)
func twoSumSorted(_ nums: [Int], _ target: Int) -> [Int] {
    var left = 0, right = nums.count - 1
    while left < right {
        let sum = nums[left] + nums[right]
        if sum == target { return [left + 1, right + 1] }
        else if sum < target { left += 1 }
        else { right -= 1 }
    }
    return []
}

// Three Sum
func threeSum(_ nums: [Int]) -> [[Int]] {
    let sorted = nums.sorted()
    var results: [[Int]] = []
    
    for i in 0..<sorted.count - 2 {
        if i > 0 && sorted[i] == sorted[i-1] { continue }  // skip duplicates
        var left = i + 1, right = sorted.count - 1
        
        while left < right {
            let sum = sorted[i] + sorted[left] + sorted[right]
            if sum == 0 {
                results.append([sorted[i], sorted[left], sorted[right]])
                while left < right && sorted[left] == sorted[left+1] { left += 1 }
                while left < right && sorted[right] == sorted[right-1] { right -= 1 }
                left += 1; right -= 1
            } else if sum < 0 {
                left += 1
            } else {
                right -= 1
            }
        }
    }
    return results
}

// Container With Most Water
func maxWater(_ height: [Int]) -> Int {
    var left = 0, right = height.count - 1, maxArea = 0
    while left < right {
        let area = min(height[left], height[right]) * (right - left)
        maxArea = max(maxArea, area)
        if height[left] < height[right] { left += 1 }
        else { right -= 1 }
    }
    return maxArea
}

// Remove Duplicates from Sorted Array
func removeDuplicates(_ nums: inout [Int]) -> Int {
    if nums.isEmpty { return 0 }
    var slow = 0
    for fast in 1..<nums.count {
        if nums[fast] != nums[slow] {
            slow += 1
            nums[slow] = nums[fast]
        }
    }
    return slow + 1
}

// Palindrome Check
func isPalindrome(_ s: String) -> Bool {
    let filtered = s.lowercased().filter { $0.isLetter || $0.isNumber }
    let chars = Array(filtered)
    var left = 0, right = chars.count - 1
    while left < right {
        if chars[left] != chars[right] { return false }
        left += 1; right -= 1
    }
    return true
}

// Sort Colors (Dutch National Flag)
func sortColors(_ nums: inout [Int]) {
    var low = 0, mid = 0, high = nums.count - 1
    while mid <= high {
        switch nums[mid] {
        case 0: nums.swapAt(low, mid); low += 1; mid += 1
        case 1: mid += 1
        default: nums.swapAt(mid, high); high -= 1
        }
    }
}
```

---

## 9. Sliding Window

```swift
// MARK: - Sliding Window

// Maximum Sum Subarray of size k
func maxSumSubarray(_ nums: [Int], _ k: Int) -> Int {
    var windowSum = nums[..<k].reduce(0, +)
    var maxSum = windowSum
    
    for i in k..<nums.count {
        windowSum += nums[i] - nums[i - k]
        maxSum = max(maxSum, windowSum)
    }
    return maxSum
}

// Longest substring without repeating characters
func lengthOfLongestSubstring(_ s: String) -> Int {
    var charIndex: [Character: Int] = [:]
    var maxLen = 0, left = 0
    let chars = Array(s)
    
    for right in 0..<chars.count {
        if let prevIdx = charIndex[chars[right]], prevIdx >= left {
            left = prevIdx + 1
        }
        charIndex[chars[right]] = right
        maxLen = max(maxLen, right - left + 1)
    }
    return maxLen
}

print("Longest no repeat 'abcabcbb': \(lengthOfLongestSubstring("abcabcbb"))")  // 3

// Minimum Window Substring
func minWindow(_ s: String, _ t: String) -> String {
    var need: [Character: Int] = [:]
    for c in t { need[c, default: 0] += 1 }
    
    let sArr = Array(s)
    var window: [Character: Int] = [:]
    var have = 0, required = need.count
    var result = ""
    var minLen = Int.max
    var left = 0
    
    for right in 0..<sArr.count {
        let c = sArr[right]
        window[c, default: 0] += 1
        if let needed = need[c], window[c] == needed { have += 1 }
        
        while have == required {
            if right - left + 1 < minLen {
                minLen = right - left + 1
                result = String(sArr[left...right])
            }
            let leftChar = sArr[left]
            window[leftChar]! -= 1
            if let needed = need[leftChar], window[leftChar]! < needed { have -= 1 }
            left += 1
        }
    }
    return result
}

// Longest Subarray with at most k distinct characters
func longestSubstringKDistinct(_ s: String, _ k: Int) -> Int {
    var freq: [Character: Int] = [:]
    var left = 0, maxLen = 0
    let chars = Array(s)
    
    for right in 0..<chars.count {
        freq[chars[right], default: 0] += 1
        
        while freq.count > k {
            let leftChar = chars[left]
            freq[leftChar]! -= 1
            if freq[leftChar]! == 0 { freq.removeValue(forKey: leftChar) }
            left += 1
        }
        maxLen = max(maxLen, right - left + 1)
    }
    return maxLen
}

// Find All Anagrams in a String
func findAnagrams(_ s: String, _ p: String) -> [Int] {
    guard s.count >= p.count else { return [] }
    let sArr = Array(s)
    var pCount = [Int](repeating: 0, count: 26)
    var wCount = [Int](repeating: 0, count: 26)
    let aVal = Int(("a" as UnicodeScalar).value)
    
    for c in p { pCount[Int(c.unicodeScalars.first!.value) - aVal] += 1 }
    
    var results: [Int] = []
    for i in 0..<sArr.count {
        wCount[Int(sArr[i].unicodeScalars.first!.value) - aVal] += 1
        if i >= p.count {
            wCount[Int(sArr[i - p.count].unicodeScalars.first!.value) - aVal] -= 1
        }
        if pCount == wCount { results.append(i - p.count + 1) }
    }
    return results
}
```

---

## 10. String Algorithms

### 10.1 KMP Algorithm

```swift
// MARK: - KMP (Knuth-Morris-Pratt) Pattern Matching - O(n + m)

func kmpSearch(_ text: String, _ pattern: String) -> [Int] {
    let txt = Array(text)
    let pat = Array(pattern)
    let n = txt.count, m = pat.count
    guard m > 0 else { return [] }
    
    // Build failure function (LPS - Longest Proper Prefix Suffix)
    func buildLPS(_ pat: [Character]) -> [Int] {
        var lps = Array(repeating: 0, count: pat.count)
        var len = 0, i = 1
        
        while i < pat.count {
            if pat[i] == pat[len] {
                len += 1
                lps[i] = len
                i += 1
            } else if len != 0 {
                len = lps[len - 1]
            } else {
                lps[i] = 0
                i += 1
            }
        }
        return lps
    }
    
    let lps = buildLPS(pat)
    var occurrences: [Int] = []
    var i = 0, j = 0
    
    while i < n {
        if txt[i] == pat[j] {
            i += 1; j += 1
        }
        if j == m {
            occurrences.append(i - j)
            j = lps[j - 1]
        } else if i < n && txt[i] != pat[j] {
            j = j != 0 ? lps[j - 1] : 0
            if j == 0 { i += 1 }
        }
    }
    return occurrences
}

print("KMP 'AABABCAAAB' find 'AAB': \(kmpSearch("AABABCAAAB", "AAB"))")  // [1, 7]

// MARK: - Rabin-Karp Algorithm - O(n + m) average, O(nm) worst

func rabinKarp(_ text: String, _ pattern: String) -> [Int] {
    let txt = Array(text)
    let pat = Array(pattern)
    let n = txt.count, m = pat.count
    let prime = 101
    let base = 256
    
    func charVal(_ c: Character) -> Int { Int(c.unicodeScalars.first!.value) }
    
    func hash(_ chars: [Character], _ start: Int, _ len: Int) -> Int {
        var h = 0
        for i in 0..<len { h = (h * base + charVal(chars[start + i])) % prime }
        return h
    }
    
    var patHash = hash(pat, 0, m)
    var winHash = hash(txt, 0, m)
    var highPow = 1
    for _ in 0..<(m - 1) { highPow = (highPow * base) % prime }
    
    var results: [Int] = []
    
    for i in 0...(n - m) {
        if winHash == patHash {
            if Array(txt[i..<(i+m)]) == pat { results.append(i) }
        }
        if i < n - m {
            winHash = (base * (winHash - charVal(txt[i]) * highPow) + charVal(txt[i + m])) % prime
            if winHash < 0 { winHash += prime }
        }
    }
    return results
}

// Longest Palindromic Substring (Manacher's Algorithm)
func longestPalindrome(_ s: String) -> String {
    var chars = Array("#" + s.map { String($0) }.joined(separator: "#") + "#")
    let n = chars.count
    var p = Array(repeating: 0, count: n)
    var center = 0, right = 0
    var maxLen = 0, maxCenter = 0
    
    for i in 0..<n {
        if i < right { p[i] = min(right - i, p[2 * center - i]) }
        var l = i - p[i] - 1, r = i + p[i] + 1
        while l >= 0 && r < n && chars[l] == chars[r] { p[i] += 1; l -= 1; r += 1 }
        if i + p[i] > right { center = i; right = i + p[i] }
        if p[i] > maxLen { maxLen = p[i]; maxCenter = i }
    }
    
    let start = (maxCenter - maxLen) / 2
    return String(Array(s)[start..<(start + maxLen)])
}

print("Longest palindrome 'babad': \(longestPalindrome("babad"))")  // "bab"
```

---

## 11. Tree Algorithms

```swift
// MARK: - Tree Algorithms

class TreeNodeBasic {
    var val: Int
    var left: TreeNodeBasic?
    var right: TreeNodeBasic?
    init(_ val: Int) { self.val = val }
}

// MARK: - LCA (Lowest Common Ancestor)

// LCA for BST - O(log n)
func lcaBST(_ root: TreeNodeBasic?, _ p: Int, _ q: Int) -> TreeNodeBasic? {
    guard let node = root else { return nil }
    if p < node.val && q < node.val { return lcaBST(node.left, p, q) }
    if p > node.val && q > node.val { return lcaBST(node.right, p, q) }
    return node
}

// LCA for Binary Tree - O(n)
func lcaBinaryTree(_ root: TreeNodeBasic?, _ p: Int, _ q: Int) -> TreeNodeBasic? {
    guard let node = root else { return nil }
    if node.val == p || node.val == q { return node }
    
    let left = lcaBinaryTree(node.left, p, q)
    let right = lcaBinaryTree(node.right, p, q)
    
    if left != nil && right != nil { return node }  // p and q on different sides
    return left ?? right
}

// MARK: - Tree DP

// House Robber III - เลือก node ที่ไม่ติดกัน
func robTree(_ root: TreeNodeBasic?) -> Int {
    func dp(_ node: TreeNodeBasic?) -> (withNode: Int, withoutNode: Int) {
        guard let node = node else { return (0, 0) }
        let left = dp(node.left)
        let right = dp(node.right)
        let withNode = node.val + left.withoutNode + right.withoutNode
        let withoutNode = max(left.withNode, left.withoutNode) + max(right.withNode, right.withoutNode)
        return (withNode, withoutNode)
    }
    let result = dp(root)
    return max(result.withNode, result.withoutNode)
}

// Diameter of Binary Tree
func diameterOfTree(_ root: TreeNodeBasic?) -> Int {
    var diameter = 0
    
    func depth(_ node: TreeNodeBasic?) -> Int {
        guard let node = node else { return 0 }
        let leftD = depth(node.left)
        let rightD = depth(node.right)
        diameter = max(diameter, leftD + rightD)
        return max(leftD, rightD) + 1
    }
    _ = depth(root)
    return diameter
}

// Serialize and Deserialize Binary Tree
func serialize(_ root: TreeNodeBasic?) -> String {
    guard let node = root else { return "null," }
    return "\(node.val)," + serialize(node.left) + serialize(node.right)
}

func deserialize(_ data: String) -> TreeNodeBasic? {
    var vals = data.components(separatedBy: ",").filter { !$0.isEmpty }
    var idx = 0
    
    func build() -> TreeNodeBasic? {
        if idx >= vals.count || vals[idx] == "null" { idx += 1; return nil }
        let node = TreeNodeBasic(Int(vals[idx])!)
        idx += 1
        node.left = build()
        node.right = build()
        return node
    }
    return build()
}

// Path Sum II
func pathSum(_ root: TreeNodeBasic?, _ target: Int) -> [[Int]] {
    var results: [[Int]] = []
    var path: [Int] = []
    
    func dfs(_ node: TreeNodeBasic?, _ remaining: Int) {
        guard let node = node else { return }
        path.append(node.val)
        if node.left == nil && node.right == nil && remaining == node.val {
            results.append(path)
        }
        dfs(node.left, remaining - node.val)
        dfs(node.right, remaining - node.val)
        path.removeLast()
    }
    dfs(root, target)
    return results
}

// Morris Traversal - In-order without recursion or stack, O(1) space
func morrisInOrder(_ root: TreeNodeBasic?) -> [Int] {
    var result: [Int] = []
    var current = root
    
    while let node = current {
        if node.left == nil {
            result.append(node.val)
            current = node.right
        } else {
            var predecessor = node.left
            while predecessor?.right != nil && predecessor?.right !== node {
                predecessor = predecessor?.right
            }
            if predecessor?.right == nil {
                predecessor?.right = node
                current = node.left
            } else {
                predecessor?.right = nil
                result.append(node.val)
                current = node.right
            }
        }
    }
    return result
}
```

---

## 12. Recursion vs Iteration

```swift
// MARK: - Recursion vs Iteration

// Tower of Hanoi
func hanoi(_ n: Int, _ from: String, _ to: String, _ aux: String) {
    if n == 1 {
        print("Move disk 1 from \(from) to \(to)")
        return
    }
    hanoi(n - 1, from, aux, to)
    print("Move disk \(n) from \(from) to \(to)")
    hanoi(n - 1, aux, to, from)
}

// Iterative DFS using explicit stack
func iterativeDFS(_ root: TreeNodeBasic?) -> [Int] {
    guard let root = root else { return [] }
    var stack = [root]
    var result: [Int] = []
    
    while !stack.isEmpty {
        let node = stack.removeLast()
        result.append(node.val)
        if let right = node.right { stack.append(right) }
        if let left = node.left { stack.append(left) }
    }
    return result
}

// Tail Recursion Optimization (TCO)
func factTail(_ n: Int, _ acc: Int = 1) -> Int {
    return n <= 1 ? acc : factTail(n - 1, acc * n)
}

// Convert recursive to iterative with explicit stack
func flattenNested(_ nested: Any) -> [Int] {
    var stack: [Any] = [nested]
    var result: [Int] = []
    
    while !stack.isEmpty {
        let item = stack.removeLast()
        if let arr = item as? [Any] {
            stack.append(contentsOf: arr.reversed())
        } else if let num = item as? Int {
            result.append(num)
        }
    }
    return result
}
```

---

## 13. Interview Preparation Tips

### LeetCode Patterns

```swift
// MARK: - Common Interview Patterns

// Pattern 1: Fast & Slow Pointers (Floyd's Cycle Detection)
func detectCycleStart(_ head: SinglyNode<Int>?) -> SinglyNode<Int>? {
    var slow = head, fast = head
    while fast?.next != nil {
        slow = slow?.next
        fast = fast?.next?.next
        if slow === fast {
            slow = head
            while slow !== fast {
                slow = slow?.next
                fast = fast?.next
            }
            return slow
        }
    }
    return nil
}

// Pattern 2: Merge Intervals
func mergeIntervals(_ intervals: [[Int]]) -> [[Int]] {
    guard !intervals.isEmpty else { return [] }
    let sorted = intervals.sorted { $0[0] < $1[0] }
    var result: [[Int]] = [sorted[0]]
    
    for interval in sorted[1...] {
        if interval[0] <= result.last![1] {
            result[result.count - 1][1] = max(result.last![1], interval[1])
        } else {
            result.append(interval)
        }
    }
    return result
}

print("Merge intervals: \(mergeIntervals([[1,3],[2,6],[8,10],[15,18]]))")

// Pattern 3: Cyclic Sort (for finding missing/duplicate in range 1-n)
func cyclicSort(_ nums: inout [Int]) {
    var i = 0
    while i < nums.count {
        let j = nums[i] - 1
        if nums[i] != nums[j] {
            nums.swapAt(i, j)
        } else {
            i += 1
        }
    }
}

func findMissingNumber(_ nums: [Int]) -> Int {
    var nums = nums
    cyclicSort(&nums)
    for i in 0..<nums.count {
        if nums[i] != i + 1 { return i + 1 }
    }
    return nums.count + 1
}

// Pattern 4: Top K Elements
func topKFrequent(_ nums: [Int], _ k: Int) -> [Int] {
    var freq: [Int: Int] = [:]
    for num in nums { freq[num, default: 0] += 1 }
    
    return freq.sorted { $0.value > $1.value }
              .prefix(k)
              .map { $0.key }
}

print("Top 2 frequent [1,1,1,2,2,3]: \(topKFrequent([1,1,1,2,2,3], 2))")  // [1, 2]

// Pattern 5: Trie for string problems
// (ดู implementation ใน Part 65)

// Pattern 6: Monotonic Stack
func dailyTemperatures(_ temperatures: [Int]) -> [Int] {
    var result = Array(repeating: 0, count: temperatures.count)
    var stack: [Int] = []  // indices
    
    for i in 0..<temperatures.count {
        while !stack.isEmpty && temperatures[stack.last!] < temperatures[i] {
            let idx = stack.removeLast()
            result[idx] = i - idx
        }
        stack.append(i)
    }
    return result
}

print("Daily temps [73,74,75,71,69,72,76,73]: \(dailyTemperatures([73,74,75,71,69,72,76,73]))")

// Pattern 7: Subsets
func subsets(_ nums: [Int]) -> [[Int]] {
    var result: [[Int]] = [[]]
    for num in nums {
        result = result + result.map { $0 + [num] }
    }
    return result
}

// Subsets with backtracking
func subsetsBacktrack(_ nums: [Int]) -> [[Int]] {
    var result: [[Int]] = []
    var current: [Int] = []
    
    func backtrack(_ start: Int) {
        result.append(current)
        for i in start..<nums.count {
            current.append(nums[i])
            backtrack(i + 1)
            current.removeLast()
        }
    }
    backtrack(0)
    return result
}
```

### การจัดการ Edge Cases

```swift
// MARK: - Edge Cases ที่ควรพิจารณาเสมอ

// 1. Empty input
func safeOperation(_ nums: [Int]) -> Int {
    guard !nums.isEmpty else { return 0 }
    // ...
    return nums.first!
}

// 2. Single element
func singleElement(_ nums: [Int]) -> Int {
    if nums.count == 1 { return nums[0] }
    // ...
    return 0
}

// 3. Overflow (use Int64 or check bounds)
func safeMultiply(_ a: Int, _ b: Int) -> Int? {
    let (result, overflow) = a.multipliedReportingOverflow(by: b)
    return overflow ? nil : result
}

// 4. Negative numbers
func absoluteMax(_ nums: [Int]) -> Int {
    return nums.max(by: { abs($0) < abs($1) }) ?? 0
}

// 5. Nil handling in trees
func safeTreeOp(_ root: TreeNodeBasic?) -> Int {
    guard let root = root else { return 0 }
    return root.val
}
```

### Time และ Space Complexity Summary

```
Algorithm            | Time         | Space
---------------------|--------------|-------
Sorting:
  Bubble/Select/Ins  | O(n²)        | O(1)
  Merge Sort         | O(n log n)   | O(n)
  Quick Sort (avg)   | O(n log n)   | O(log n)
  Heap Sort          | O(n log n)   | O(1)
  Counting/Radix     | O(n+k)       | O(k)

Searching:
  Linear             | O(n)         | O(1)
  Binary             | O(log n)     | O(1)

Graph:
  BFS/DFS            | O(V+E)       | O(V)
  Dijkstra           | O((V+E)logV) | O(V)
  Bellman-Ford       | O(VE)        | O(V)
  Floyd-Warshall     | O(V³)        | O(V²)

DP (common):
  Fibonacci          | O(n)         | O(1) optimized
  Knapsack           | O(nW)        | O(W) optimized
  LCS                | O(mn)        | O(mn) or O(n)
  Edit Distance      | O(mn)        | O(mn) or O(n)
```

---

## 14. สรุปและแนวทางการแก้ปัญหา

### Framework สำหรับ Coding Interviews

```swift
// MARK: - Problem-Solving Framework

/*
1. CLARIFY the problem:
   - อ่านโจทย์ให้เข้าใจ
   - ถามเกี่ยวกับ edge cases
   - ยืนยัน input/output format

2. PLAN your approach:
   - คิดถึง data structure ที่เหมาะสม
   - ระบุ algorithm pattern ที่ตรงกับปัญหา
   - วิเคราะห์ time/space complexity

3. CODE the solution:
   - เขียน code ที่สะอาด อ่านง่าย
   - แยกเป็น function ย่อย
   - จัดการ edge cases

4. TEST your solution:
   - ทดสอบกับ example cases
   - ทดสอบ edge cases
   - trace through ด้วย example

5. OPTIMIZE if needed:
   - ระบุ bottleneck
   - ปรับปรุง time/space complexity
*/

// Pattern Recognition Cheat Sheet:
// - Array/String + window -> Sliding Window
// - Sorted array + target sum -> Two Pointers
// - Linked list cycle -> Fast & Slow Pointers
// - Find all combinations/permutations -> Backtracking
// - Optimization with overlapping subproblems -> DP
// - Shortest path unweighted -> BFS
// - Shortest path weighted -> Dijkstra
// - Prefix matching -> Trie
// - Top K elements -> Heap/Quick Select
// - Range query on sorted array -> Binary Search
// - Parentheses/calculator -> Stack
// - Matrix traversal -> BFS/DFS

// Template สำหรับ Binary Search
func binarySearchTemplate(_ nums: [Int], _ target: Int) -> Int {
    var left = 0, right = nums.count - 1
    while left <= right {
        let mid = left + (right - left) / 2
        if nums[mid] == target { return mid }
        else if nums[mid] < target { left = mid + 1 }
        else { right = mid - 1 }
    }
    return -1
}

// Template สำหรับ Sliding Window
func slidingWindowTemplate(_ nums: [Int], _ k: Int) -> Int {
    var left = 0, maxVal = 0
    var windowState = 0  // ปรับตาม problem

    for right in 0..<nums.count {
        // expand window: update windowState with nums[right]
        windowState += nums[right]
        
        // shrink if needed
        while /* condition */ right - left + 1 > k {
            windowState -= nums[left]
            left += 1
        }
        
        // update answer
        maxVal = max(maxVal, windowState)
    }
    return maxVal
}

// Template สำหรับ DFS
func dfsTemplate(_ root: TreeNodeBasic?) -> Int {
    guard let node = root else { return 0 }
    let leftResult = dfsTemplate(node.left)
    let rightResult = dfsTemplate(node.right)
    // process current node with leftResult and rightResult
    return max(leftResult, rightResult) + 1
}

// Template สำหรับ BFS
func bfsTemplate(_ root: TreeNodeBasic?) -> [Int] {
    guard let root = root else { return [] }
    var queue: [TreeNodeBasic] = [root]
    var result: [Int] = []
    
    while !queue.isEmpty {
        let size = queue.count
        for _ in 0..<size {
            let node = queue.removeFirst()
            result.append(node.val)
            if let left = node.left { queue.append(left) }
            if let right = node.right { queue.append(right) }
        }
    }
    return result
}

// Template สำหรับ Backtracking
func backtrackTemplate(_ nums: [Int]) -> [[Int]] {
    var results: [[Int]] = []
    var current: [Int] = []
    
    func backtrack(_ start: Int) {
        // Base case
        if current.count == nums.count {
            results.append(current)
            return
        }
        
        for i in start..<nums.count {
            // Skip duplicates
            if i > start && nums[i] == nums[i-1] { continue }
            
            // Choose
            current.append(nums[i])
            
            // Explore
            backtrack(i + 1)
            
            // Unchoose
            current.removeLast()
        }
    }
    
    backtrack(0)
    return results
}
```

### สิ่งที่ต้องจำสำหรับ Technical Interview

```swift
// MARK: - Key Points to Remember

/*
1. Data Structure Selection:
   - O(1) access needed -> Array/HashMap
   - O(1) insert/delete at ends -> Deque
   - Ordered operations -> BST/AVL
   - Priority access -> Heap
   - Prefix matching -> Trie
   - Relationship/paths -> Graph

2. Time Complexity Goals:
   - Simple problems: O(n) or O(n log n)
   - Hard problems: O(n log n) or better
   - Never accept O(n²) without justification

3. Space Optimization:
   - 2D DP -> 1D DP (rolling array)
   - Recursion -> Iteration with stack
   - Copy -> In-place operations

4. Common Mistakes:
   - Off-by-one errors
   - Integer overflow
   - Null/nil checks
   - Infinite loops in while
   - Forgetting to handle empty input
   - Mutable default parameters

5. Swift-specific:
   - Use value types (struct) for safety
   - inout for in-place modifications
   - guard for early exit
   - defer for cleanup
   - Optional chaining (?.) over force unwrap (!)
*/
```

---

*จบ Part 66: Algorithms ใน Swift*
