# Part 97: Advanced UIKit Mastery

## บทนำ

บทนี้ครอบคลุมหัวข้อขั้นสูงของ UIKit ที่นักพัฒนา iOS ระดับ Senior ต้องรู้ ตั้งแต่ Responder Chain, Custom Transitions, Compositional Layout, Custom Controls, CALayer Animations ไปจนถึง Interactive Dismissal และการทดสอบ

**สิ่งที่จะได้เรียนรู้:**
- เข้าใจ Responder Chain และ Touch Handling ระดับลึก
- สร้าง Custom View Controller Transition แบบ Card และ Interactive
- ใช้ Compositional Layout สร้าง Complex Collection View
- สร้าง Custom UIControl ที่ใช้งานร่วมกับ Target-Action
- ทำ CALayer Animation ระดับ Professional
- เพิ่มประสิทธิภาพ UIKit ด้วย Prefetching และ Rasterization
- เขียน UIKit Tests ทั้ง Unit Test และ UI Test

**Prerequisites:**
- Swift พื้นฐาน (Part 1–20)
- Auto Layout (Part 60+)
- UIViewController Lifecycle
- Closure และ Protocol เบื้องต้น

**iOS Version:** iOS 14+ สำหรับ Compositional List และ CellRegistration, iOS 13+ สำหรับส่วนอื่น

> หมายเหตุ: ตัวอย่างทุกชิ้นในบทนี้สามารถ copy ไปรันใน Xcode Playground หรือสร้างเป็น Single View App ได้ทันที โดยไม่ต้องพึ่ง third-party library ใดๆ

---

## 1. UIKit Responder Chain

### แนวคิดและการทำงาน

Responder Chain คือกลไกที่ UIKit ใช้ส่งต่อ Event ผ่านลำดับชั้น:
```
UIApplication → UIWindow → UIViewController → UIView → Subviews
```

ทุก Object ที่สืบทอดจาก `UIResponder` สามารถรับและจัดการ Event ได้ ได้แก่:
- `UIView` และ subclasses ทั้งหมด
- `UIViewController`
- `UIApplication`
- `AppDelegate`

**ลำดับการค้นหา First Responder ด้วย hitTest:**
1. UIApplication รับ UIEvent จาก OS
2. ส่งไปยัง Key Window
3. Window เรียก `hitTest(_:with:)` บน root view
4. แต่ละ View ตรวจสอบ subviews จาก index สูงสุด (top-most) ลงมา
5. View แรกที่ `pointInside` คืนค่า `true` และไม่ hidden/disabled จะถูกเลือก

**hitTest(_:with:)** ค้นหา View ที่รับ touch ได้ — ตรวจ `isHidden`, `isUserInteractionEnabled`, `alpha` แล้ววนลงไปใน subviews (top-most ก่อน)

**First Responder:**
```swift
textField.becomeFirstResponder()   // กำหนด first responder (แสดง keyboard)
textField.resignFirstResponder()   // ยกเลิก first responder (ซ่อน keyboard)
view.endEditing(true)              // force resign ทุก field ใน view hierarchy

// ตรวจสอบ
if textField.isFirstResponder {
    print("textField กำลัง active อยู่")
}
```

**การส่งต่อ Event ขึ้น Chain:**
```swift
// ถ้า View ไม่จัดการ Event จะส่งต่อขึ้น next responder
override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
    // จัดการเอง หรือ...
    super.touchesBegan(touches, with: event)  // ส่งต่อขึ้น parent
}
```

### ตัวอย่าง: Custom hitTest ขยายพื้นที่รับ Touch

```swift
import UIKit

class LargeTouchAreaButton: UIButton {
    var touchAreaPadding = UIEdgeInsets(top: 20, left: 20, bottom: 20, right: 20)

    override func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView? {
        guard isUserInteractionEnabled, !isHidden, alpha > 0.01 else { return nil }

        let expanded = bounds.inset(by: UIEdgeInsets(
            top: -touchAreaPadding.top, left: -touchAreaPadding.left,
            bottom: -touchAreaPadding.bottom, right: -touchAreaPadding.right
        ))

        if expanded.contains(point) {
            for subview in subviews.reversed() {
                let converted = subview.convert(point, from: self)
                if let result = subview.hitTest(converted, with: event) { return result }
            }
            return self
        }
        return nil
    }
}

// การใช้งาน
let button = LargeTouchAreaButton(type: .system)
button.setTitle("กด", for: .normal)
button.frame = CGRect(x: 150, y: 300, width: 60, height: 44)
button.touchAreaPadding = UIEdgeInsets(top: 30, left: 30, bottom: 30, right: 30)
```

---

## 2. Custom View Controller Transitions

### UIViewControllerAnimatedTransitioning Protocol

```swift
protocol UIViewControllerAnimatedTransitioning {
    func transitionDuration(using: UIViewControllerContextTransitioning?) -> TimeInterval
    func animateTransition(using: UIViewControllerContextTransitioning)
}
```

**ข้อมูลจาก context:**
```swift
let containerView = transitionContext.containerView
let fromVC = transitionContext.viewController(forKey: .from)
let toVC   = transitionContext.viewController(forKey: .to)
transitionContext.completeTransition(!transitionContext.transitionWasCancelled)
```

### ตัวอย่าง: Card-Style Present/Dismiss Transition

```swift
import UIKit

// MARK: - Animators

class CardPresentAnimator: NSObject, UIViewControllerAnimatedTransitioning {
    func transitionDuration(using ctx: UIViewControllerContextTransitioning?) -> TimeInterval { 0.45 }

    func animateTransition(using ctx: UIViewControllerContextTransitioning) {
        guard let toVC = ctx.viewController(forKey: .to) else { return }
        let container = ctx.containerView
        let finalFrame = ctx.finalFrame(for: toVC)
        toVC.view.frame = finalFrame.offsetBy(dx: 0, dy: finalFrame.height)
        toVC.view.layer.cornerRadius = 20
        toVC.view.clipsToBounds = true
        container.addSubview(toVC.view)

        UIView.animate(withDuration: transitionDuration(using: ctx),
                       delay: 0, usingSpringWithDamping: 0.78,
                       initialSpringVelocity: 0.5, options: .curveEaseOut) {
            toVC.view.frame = finalFrame
        } completion: { _ in
            ctx.completeTransition(!ctx.transitionWasCancelled)
        }
    }
}

class CardDismissAnimator: NSObject, UIViewControllerAnimatedTransitioning {
    func transitionDuration(using ctx: UIViewControllerContextTransitioning?) -> TimeInterval { 0.35 }

    func animateTransition(using ctx: UIViewControllerContextTransitioning) {
        guard let fromVC = ctx.viewController(forKey: .from) else { return }
        let finalY = fromVC.view.frame.maxY
        UIView.animate(withDuration: transitionDuration(using: ctx), options: .curveEaseIn) {
            fromVC.view.frame.origin.y = finalY
        } completion: { _ in
            fromVC.view.removeFromSuperview()
            ctx.completeTransition(!ctx.transitionWasCancelled)
        }
    }
}

// MARK: - UIPresentationController (Dimming)

class CardPresentationController: UIPresentationController {
    private let dimmingView: UIView = {
        let v = UIView()
        v.backgroundColor = UIColor.black.withAlphaComponent(0.5)
        v.alpha = 0
        return v
    }()

    override func presentationTransitionWillBegin() {
        guard let container = containerView else { return }
        dimmingView.frame = container.bounds
        container.insertSubview(dimmingView, at: 0)
        presentedViewController.transitionCoordinator?.animate { _ in self.dimmingView.alpha = 1 }
    }

    override func dismissalTransitionWillBegin() {
        presentedViewController.transitionCoordinator?.animate { _ in
            self.dimmingView.alpha = 0
        } completion: { _ in self.dimmingView.removeFromSuperview() }
    }

    override var frameOfPresentedViewInContainerView: CGRect {
        guard let container = containerView else { return .zero }
        let height = container.bounds.height * 0.55
        return CGRect(x: 0, y: container.bounds.height - height,
                      width: container.bounds.width, height: height)
    }
}

// MARK: - Transitioning Delegate

class CardTransitioningDelegate: NSObject, UIViewControllerTransitioningDelegate {
    func animationController(forPresented p: UIViewController,
                              presenting: UIViewController,
                              source: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        CardPresentAnimator()
    }
    func animationController(forDismissed d: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        CardDismissAnimator()
    }
    func presentationController(forPresented presented: UIViewController,
                                 presenting: UIViewController?,
                                 source: UIViewController) -> UIPresentationController? {
        CardPresentationController(presentedViewController: presented, presenting: presenting)
    }
}

// การใช้งาน
class HomeViewController: UIViewController {
    private let cardDelegate = CardTransitioningDelegate()

    @objc func showCard() {
        let cardVC = UIViewController()
        cardVC.view.backgroundColor = .systemBackground
        cardVC.modalPresentationStyle = .custom
        cardVC.transitioningDelegate = cardDelegate
        present(cardVC, animated: true)
    }
}
```

---

## 3. UICollectionView Compositional Layout

Compositional Layout มี 3 ระดับ: **Item → Group → Section**

```swift
// Header/Footer
let headerSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                        heightDimension: .estimated(44))
let header = NSCollectionLayoutBoundarySupplementaryItem(
    layoutSize: headerSize,
    elementKind: UICollectionView.elementKindSectionHeader, alignment: .top)
section.boundarySupplementaryItems = [header]
```

### ตัวอย่าง: Layout 3 Section (Banner, Grid, List)

```swift
import UIKit

enum SectionKind: Int, CaseIterable { case banner, grid, list }
struct Item: Hashable { let id = UUID(); let title: String; let color: UIColor }

class CompositionalDemoVC: UIViewController {
    typealias DataSource = UICollectionViewDiffableDataSource<SectionKind, Item>
    private var collectionView: UICollectionView!
    private var dataSource: DataSource!

    override func viewDidLoad() {
        super.viewDidLoad()
        collectionView = UICollectionView(frame: view.bounds, collectionViewLayout: createLayout())
        collectionView.autoresizingMask = [.flexibleWidth, .flexibleHeight]
        view.addSubview(collectionView)
        configureDataSource()
        applySnapshot()
    }

    private func createLayout() -> UICollectionViewLayout {
        UICollectionViewCompositionalLayout { idx, _ in
            switch SectionKind(rawValue: idx) {
            case .banner:
                let item  = NSCollectionLayoutItem(layoutSize: .init(widthDimension: .fractionalWidth(1),
                                                                      heightDimension: .fractionalHeight(1)))
                let group = NSCollectionLayoutGroup.horizontal(
                    layoutSize: .init(widthDimension: .fractionalWidth(1), heightDimension: .absolute(200)),
                    subitems: [item])
                return NSCollectionLayoutSection(group: group)

            case .grid:
                let item  = NSCollectionLayoutItem(layoutSize: .init(widthDimension: .fractionalWidth(1/3),
                                                                      heightDimension: .fractionalHeight(1)))
                item.contentInsets = .init(top: 4, leading: 4, bottom: 4, trailing: 4)
                let group = NSCollectionLayoutGroup.horizontal(
                    layoutSize: .init(widthDimension: .fractionalWidth(1), heightDimension: .absolute(100)),
                    subitems: [item])
                return NSCollectionLayoutSection(group: group)

            default: // .list
                let item  = NSCollectionLayoutItem(layoutSize: .init(widthDimension: .fractionalWidth(1),
                                                                      heightDimension: .absolute(56)))
                let group = NSCollectionLayoutGroup.vertical(
                    layoutSize: .init(widthDimension: .fractionalWidth(1), heightDimension: .estimated(56)),
                    subitems: [item])
                return NSCollectionLayoutSection(group: group)
            }
        }
    }

    private func configureDataSource() {
        let cellReg = UICollectionView.CellRegistration<UICollectionViewCell, Item> { cell, _, item in
            cell.backgroundColor = item.color
            cell.layer.cornerRadius = 8
            var cfg = UIListContentConfiguration.cell()
            cfg.text = item.title
            cfg.textProperties.color = .white
            cell.contentConfiguration = cfg
        }
        dataSource = DataSource(collectionView: collectionView) { cv, idx, item in
            cv.dequeueConfiguredReusableCell(using: cellReg, for: idx, item: item)
        }
    }

    private func applySnapshot() {
        var snap = NSDiffableDataSourceSnapshot<SectionKind, Item>()
        snap.appendSections(SectionKind.allCases)
        snap.appendItems([Item(title: "Banner", color: .systemRed)], toSection: .banner)
        snap.appendItems((1...9).map { Item(title: "Grid \($0)", color: .systemBlue) }, toSection: .grid)
        snap.appendItems((1...5).map { Item(title: "List \($0)", color: .systemGreen) }, toSection: .list)
        dataSource.apply(snap, animatingDifferences: false)
    }
}
```

---

## 4. UICollectionView List (iOS 14+)

### UICollectionLayoutListConfiguration

```swift
var config = UICollectionLayoutListConfiguration(appearance: .insetGrouped)
config.showsSeparators = true
config.headerMode = .supplementary
let layout = UICollectionViewCompositionalLayout.list(using: config)
```

### UICellAccessory และ Expandable Outline

```swift
cell.accessories = [.disclosureIndicator()]       // ลูกศรขวา
cell.accessories = [.checkmark()]                  // checkmark
cell.accessories = [.outlineDisclosure()]          // expandable outline
```

### ตัวอย่าง: Settings-style List

```swift
import UIKit

struct SettingItem: Hashable {
    let title: String; let icon: String; let iconColor: UIColor
}

class SettingsListVC: UIViewController {
    typealias DS = UICollectionViewDiffableDataSource<String, SettingItem>
    private var collectionView: UICollectionView!
    private var dataSource: DS!

    private let sections: [(title: String, items: [SettingItem])] = [
        ("บัญชีผู้ใช้", [
            SettingItem(title: "โปรไฟล์",      icon: "person.circle",    iconColor: .systemBlue),
            SettingItem(title: "รหัสผ่าน",     icon: "lock.fill",        iconColor: .systemOrange),
            SettingItem(title: "การแจ้งเตือน", icon: "bell.fill",        iconColor: .systemRed)
        ]),
        ("ทั่วไป", [
            SettingItem(title: "ภาษา",   icon: "globe",              iconColor: .systemGreen),
            SettingItem(title: "ธีม",    icon: "paintpalette.fill",  iconColor: .systemPurple),
            SettingItem(title: "เกี่ยวกับ", icon: "info.circle.fill", iconColor: .systemGray)
        ])
    ]

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "การตั้งค่า"
        var lc = UICollectionLayoutListConfiguration(appearance: .insetGrouped)
        lc.headerMode = .supplementary
        collectionView = UICollectionView(
            frame: view.bounds,
            collectionViewLayout: UICollectionViewCompositionalLayout.list(using: lc))
        collectionView.autoresizingMask = [.flexibleWidth, .flexibleHeight]
        view.addSubview(collectionView)
        setupDataSource()
        applySnapshot()
    }

    private func setupDataSource() {
        let cellReg = UICollectionView.CellRegistration<UICollectionViewListCell, SettingItem> {
            cell, _, item in
            var cfg = cell.defaultContentConfiguration()
            cfg.text = item.title
            cfg.image = UIImage(systemName: item.icon)
            cfg.imageProperties.tintColor = item.iconColor
            cell.contentConfiguration = cfg
            cell.accessories = [.disclosureIndicator()]
        }
        let headerReg = UICollectionView.SupplementaryRegistration<UICollectionViewListCell>(
            elementKind: UICollectionView.elementKindSectionHeader) { [weak self] cell, _, idx in
            var cfg = cell.defaultContentConfiguration()
            cfg.text = self?.sections[idx.section].title
            cfg.textProperties.font = .systemFont(ofSize: 13, weight: .semibold)
            cfg.textProperties.color = .secondaryLabel
            cell.contentConfiguration = cfg
        }
        dataSource = DS(collectionView: collectionView) { cv, idx, item in
            cv.dequeueConfiguredReusableCell(using: cellReg, for: idx, item: item)
        }
        dataSource.supplementaryViewProvider = { cv, _, idx in
            cv.dequeueConfiguredReusableSupplementary(using: headerReg, for: idx)
        }
    }

    private func applySnapshot() {
        var snap = DS.Snapshot()
        sections.forEach { s in
            snap.appendSections([s.title])
            snap.appendItems(s.items, toSection: s.title)
        }
        dataSource.apply(snap, animatingDifferences: false)
    }
}
```

---

## 5. Custom UIControl

### การ Subclass UIControl

```swift
class MyControl: UIControl {
    override func beginTracking(_ touch: UITouch, with event: UIEvent?) -> Bool { true }
    override func continueTracking(_ touch: UITouch, with event: UIEvent?) -> Bool { true }
    override func endTracking(_ touch: UITouch?, with event: UIEvent?) {
        sendActions(for: .valueChanged)   // ส่ง event ไปยัง targets
    }
    override func cancelTracking(with event: UIEvent?) { }
}
```

### ตัวอย่าง: Star Rating Control

```swift
import UIKit

class StarRatingControl: UIControl {
    var rating: Int = 0 { didSet { rating = min(max(rating, 0), maxStars); updateStars() } }
    var maxStars: Int = 5
    var starSize: CGFloat = 36

    private var starButtons: [UIButton] = []
    private let stack = UIStackView()

    override init(frame: CGRect) { super.init(frame: frame); setupStars() }
    required init?(coder: NSCoder) { super.init(coder: coder); setupStars() }

    private func setupStars() {
        starButtons.forEach { $0.removeFromSuperview() }
        starButtons.removeAll()

        stack.axis = .horizontal
        stack.spacing = 6
        stack.isUserInteractionEnabled = false
        stack.translatesAutoresizingMaskIntoConstraints = false
        addSubview(stack)
        NSLayoutConstraint.activate([
            stack.topAnchor.constraint(equalTo: topAnchor),
            stack.bottomAnchor.constraint(equalTo: bottomAnchor),
            stack.leadingAnchor.constraint(equalTo: leadingAnchor),
            stack.trailingAnchor.constraint(equalTo: trailingAnchor)
        ])

        for i in 0..<maxStars {
            let btn = UIButton()
            btn.tag = i + 1
            let cfg = UIImage.SymbolConfiguration(pointSize: starSize)
            btn.setImage(UIImage(systemName: "star.fill", withConfiguration: cfg), for: .normal)
            btn.tintColor = .systemGray4
            btn.widthAnchor.constraint(equalToConstant: starSize + 4).isActive = true
            btn.heightAnchor.constraint(equalToConstant: starSize + 4).isActive = true
            starButtons.append(btn)
            stack.addArrangedSubview(btn)
        }
        updateStars()
    }

    private func updateStars() {
        starButtons.enumerated().forEach { i, btn in
            btn.tintColor = i < rating ? .systemYellow : .systemGray4
        }
    }

    override func beginTracking(_ touch: UITouch, with event: UIEvent?) -> Bool {
        updateRating(from: touch); return true
    }
    override func continueTracking(_ touch: UITouch, with event: UIEvent?) -> Bool {
        updateRating(from: touch); return true
    }
    override func endTracking(_ touch: UITouch?, with event: UIEvent?) {
        if let t = touch { updateRating(from: t) }
        sendActions(for: .valueChanged)
    }

    private func updateRating(from touch: UITouch) {
        let x = touch.location(in: stack).x
        let starW = starSize + 6
        for i in 0..<maxStars {
            if x <= CGFloat(i + 1) * starW { rating = i + 1; return }
        }
        rating = maxStars
    }

    override var intrinsicContentSize: CGSize {
        CGSize(width: CGFloat(maxStars) * (starSize + 6) - 6, height: starSize + 4)
    }
}
```

---

## 6. CALayer Animations

### CABasicAnimation และ CAKeyframeAnimation

CALayer animation ทำงานบน **Presentation Layer** ซึ่งแยกจาก **Model Layer** ทำให้ animation ที่เห็นบนหน้าจออาจต่างจาก property จริง

```swift
// CABasicAnimation: animate จาก A → B
let fade = CABasicAnimation(keyPath: "opacity")
fade.fromValue    = 1.0
fade.toValue      = 0.0
fade.duration     = 0.5
fade.repeatCount  = .infinity
fade.autoreverses = true
layer.add(fade, forKey: "fade")

// CAKeyframeAnimation: animate ผ่านหลาย keyframes
let bounce = CAKeyframeAnimation(keyPath: "transform.scale")
bounce.values   = [1.0, 1.2, 0.9, 1.05, 1.0]
bounce.keyTimes = [0, 0.3, 0.6, 0.8, 1.0]
bounce.duration = 0.8
layer.add(bounce, forKey: "bounce")

// CAAnimationGroup: เล่นหลาย animation พร้อมกัน
let scaleAnim = CABasicAnimation(keyPath: "transform.scale")
scaleAnim.fromValue = 1.0; scaleAnim.toValue = 1.3

let colorAnim = CABasicAnimation(keyPath: "backgroundColor")
colorAnim.fromValue = UIColor.systemBlue.cgColor
colorAnim.toValue   = UIColor.systemRed.cgColor

let group = CAAnimationGroup()
group.animations = [scaleAnim, colorAnim]
group.duration   = 0.6; group.autoreverses = true; group.repeatCount = .infinity
layer.add(group, forKey: "groupAnim")

// CADisplayLink: callback ทุก frame (60/120fps)
let link = CADisplayLink(target: self, selector: #selector(tick))
link.add(to: .main, forMode: .common)

// อย่าลืม invalidate เมื่อเลิกใช้
// link.invalidate()
```

**fillMode และ isRemovedOnCompletion:**
```swift
let anim = CABasicAnimation(keyPath: "position.y")
anim.toValue = 200
anim.duration = 0.5
// ป้องกัน animation กระโดดกลับเมื่อเสร็จ
anim.fillMode = .forwards
anim.isRemovedOnCompletion = false
// จากนั้นต้องอัปเดต model layer ด้วย
layer.position.y = 200
layer.add(anim, forKey: "move")
```

### ตัวอย่าง: Animated Progress Ring ด้วย CAShapeLayer

```swift
import UIKit

class AnimatedRingView: UIView {
    var progress: CGFloat = 0 { didSet { animateTo(min(max(progress, 0), 1)) } }
    var ringColor: UIColor = .systemBlue { didSet { progressLayer.strokeColor = ringColor.cgColor } }
    var lineWidth: CGFloat = 12

    private let trackLayer    = CAShapeLayer()
    private let progressLayer = CAShapeLayer()
    private let label: UILabel = {
        let l = UILabel(); l.textAlignment = .center
        l.font = .boldSystemFont(ofSize: 28); l.text = "0%"; return l
    }()

    override init(frame: CGRect) { super.init(frame: frame); setup() }
    required init?(coder: NSCoder) { super.init(coder: coder); setup() }

    private func setup() {
        trackLayer.fillColor    = UIColor.clear.cgColor
        trackLayer.strokeColor  = UIColor.systemGray5.cgColor
        trackLayer.lineWidth    = lineWidth; trackLayer.lineCap = .round
        layer.addSublayer(trackLayer)

        progressLayer.fillColor    = UIColor.clear.cgColor
        progressLayer.strokeColor  = ringColor.cgColor
        progressLayer.lineWidth    = lineWidth; progressLayer.lineCap = .round
        progressLayer.strokeEnd    = 0
        progressLayer.transform = CATransform3DMakeRotation(-.pi / 2, 0, 0, 1)
        layer.addSublayer(progressLayer)

        label.translatesAutoresizingMaskIntoConstraints = false
        addSubview(label)
        NSLayoutConstraint.activate([
            label.centerXAnchor.constraint(equalTo: centerXAnchor),
            label.centerYAnchor.constraint(equalTo: centerYAnchor)
        ])
    }

    override func layoutSubviews() {
        super.layoutSubviews()
        [trackLayer, progressLayer].forEach { $0.frame = bounds }
        let radius = (min(bounds.width, bounds.height) / 2) - lineWidth / 2
        let path = UIBezierPath(arcCenter: CGPoint(x: bounds.midX, y: bounds.midY),
                                radius: radius, startAngle: 0, endAngle: 2 * .pi, clockwise: true)
        trackLayer.path = path.cgPath
        progressLayer.path = path.cgPath
    }

    private func animateTo(_ value: CGFloat) {
        let anim = CABasicAnimation(keyPath: "strokeEnd")
        anim.fromValue = progressLayer.presentation()?.strokeEnd ?? progressLayer.strokeEnd
        anim.toValue   = value; anim.duration = 0.6
        anim.timingFunction = CAMediaTimingFunction(name: .easeInEaseOut)
        progressLayer.strokeEnd = value
        progressLayer.add(anim, forKey: "progress")
        label.text = "\(Int(value * 100))%"
    }
}
```

---

## 7. UIKit Performance

### UITableViewDataSourcePrefetching

```swift
class MyTableVC: UITableViewController, UITableViewDataSourcePrefetching {
    override func viewDidLoad() {
        super.viewDidLoad()
        tableView.prefetchDataSource = self
    }

    // เรียกก่อน cell แสดงผล — เริ่ม fetch
    func tableView(_ tv: UITableView, prefetchRowsAt indexPaths: [IndexPath]) {
        indexPaths.forEach { loadImage(for: $0.row) }
    }

    // เรียกเมื่อ cell scroll ออกไปก่อน fetch เสร็จ — cancel
    func tableView(_ tv: UITableView, cancelPrefetchingForRowsAt indexPaths: [IndexPath]) {
        indexPaths.forEach { cancelImage(for: $0.row) }
    }
}
```

### shouldRasterize และ Off-Screen Rendering

**Off-screen rendering** เกิดขึ้นเมื่อ GPU ต้องวาด layer นอกหน้าจอก่อน แล้ว composite กลับมา เป็นสาเหตุหลักของ scroll jank

**สิ่งที่ทำให้เกิด off-screen rendering:**
- `masksToBounds = true` + `cornerRadius`
- `shadow` บน layer ที่ไม่มี `shadowPath`
- `mask` layer แบบ custom
- `allowsGroupOpacity`

```swift
// BAD: ทำให้ off-screen rendering
cell.layer.masksToBounds = true
cell.layer.cornerRadius  = 12

// BETTER: ตั้ง shadowPath ลด off-screen
cell.layer.shadowPath    = UIBezierPath(roundedRect: cell.bounds, cornerRadius: 12).cgPath
cell.layer.cornerRadius  = 12
cell.clipsToBounds       = false

// Rasterize: แปลง layer เป็น bitmap สำหรับ static view
// ดีสำหรับ complex view ที่ไม่เปลี่ยน เช่น shadow + rounded corner
cell.layer.shouldRasterize      = true
cell.layer.rasterizationScale   = UIScreen.main.scale

// หลีกเลี่ยง off-screen rendering:
// iOS 13+: cornerRadius ทำงานได้โดยไม่ต้องใช้ masksToBounds สำหรับ background ธรรมดา
cell.layer.cornerRadius  = 12
cell.layer.cornerCurve   = .continuous  // Apple-style rounded corner
```

**Debug off-screen rendering:** เปิด Simulator → Debug → Color Off-screen Rendered (พื้นที่สีเหลืองคือ off-screen)

### ตัวอย่าง: Smooth Image-Heavy Table View

```swift
import UIKit

class NewsItem {
    let title: String; let imageURL: URL
    var image: UIImage?; var loadTask: URLSessionDataTask?
    init(title: String, url: String) { self.title = title; imageURL = URL(string: url)! }
}

class SmoothTableVC: UITableViewController, UITableViewDataSourcePrefetching {
    private var items = (1...50).map {
        NewsItem(title: "บทความที่ \($0)", url: "https://picsum.photos/seed/\($0)/200/200")
    }
    private let cache = NSCache<NSURL, UIImage>()

    override func viewDidLoad() {
        super.viewDidLoad()
        tableView.prefetchDataSource = self
        tableView.register(UITableViewCell.self, forCellReuseIdentifier: "cell")
    }

    override func tableView(_ tv: UITableView, numberOfRowsInSection s: Int) -> Int { items.count }

    override func tableView(_ tv: UITableView, cellForRowAt ip: IndexPath) -> UITableViewCell {
        let cell = tv.dequeueReusableCell(withIdentifier: "cell", for: ip)
        let item = items[ip.row]
        var cfg = cell.defaultContentConfiguration()
        cfg.text = item.title
        cfg.image = item.image ?? UIImage(systemName: "photo")
        cfg.imageProperties.maximumSize = CGSize(width: 60, height: 60)
        cell.contentConfiguration = cfg
        cell.layer.shouldRasterize = true
        cell.layer.rasterizationScale = UIScreen.main.scale
        if item.image == nil { loadImage(for: ip.row) { tv.reloadRows(at: [ip], with: .none) } }
        return cell
    }

    override func tableView(_ tv: UITableView, heightForRowAt ip: IndexPath) -> CGFloat { 80 }

    func tableView(_ tv: UITableView, prefetchRowsAt ips: [IndexPath]) {
        ips.forEach { loadImage(for: $0.row, completion: nil) }
    }
    func tableView(_ tv: UITableView, cancelPrefetchingForRowsAt ips: [IndexPath]) {
        ips.forEach { items[$0.row].loadTask?.cancel() }
    }

    private func loadImage(for index: Int, completion: (() -> Void)? = nil) {
        let item = items[index]
        guard item.image == nil, item.loadTask == nil else { completion?(); return }
        if let cached = cache.object(forKey: item.imageURL as NSURL) {
            item.image = cached; completion?(); return
        }
        let task = URLSession.shared.dataTask(with: item.imageURL) { [weak self] data, _, _ in
            guard let data = data, let img = UIImage(data: data) else { return }
            self?.cache.setObject(img, forKey: item.imageURL as NSURL)
            DispatchQueue.main.async { item.image = img; item.loadTask = nil; completion?() }
        }
        item.loadTask = task; task.resume()
    }
}
```

---

## 8. Custom Interactive Dismissal

### UIViewControllerInteractiveTransitioning

Protocol ที่ให้ controller ระดับ interactive ระหว่าง transition มี 2 แนวทาง:
1. **UIPercentDrivenInteractiveTransition** — คลาสพร้อมใช้ เชื่อมกับ Animator ที่มีอยู่
2. **Custom implementation** — ควบคุม framerate/physics เองทั้งหมด

### UIPercentDrivenInteractiveTransition

```swift
class MyInteractiveTransition: UIPercentDrivenInteractiveTransition {
    var isInteracting = false

    // อัปเดต progress (0.0 - 1.0)
    func updateProgress(_ p: CGFloat) { update(p) }
    // เสร็จสิ้น transition ตาม direction ที่ตั้งไว้
    func finishInteraction() { finish() }
    // ยกเลิกและกลับไปสถานะเดิม
    func cancelInteraction() { cancel() }
}
```

**หลักการ:**
- ต้องเริ่ม `dismiss(animated: true)` หรือ navigation push/pop ก่อน
- `interactionControllerForDismissal` ต้องคืน transition object ที่ยังทำงานอยู่
- เรียก `finish()` หรือ `cancel()` ใน `.ended` / `.cancelled` gesture state เสมอ

### ตัวอย่าง: Swipe-Down to Dismiss พร้อม Rubber-Band Effect

```swift
import UIKit

class InteractiveDismissVC: UIViewController {
    var interactiveTransition: UIPercentDrivenInteractiveTransition?

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemTeal

        let indicator = UIView()
        indicator.backgroundColor = UIColor.white.withAlphaComponent(0.6)
        indicator.layer.cornerRadius = 3
        indicator.frame = CGRect(x: 0, y: 12, width: 40, height: 6)
        indicator.center.x = view.center.x
        view.addSubview(indicator)

        let pan = UIPanGestureRecognizer(target: self, action: #selector(handlePan(_:)))
        view.addGestureRecognizer(pan)
    }

    @objc private func handlePan(_ g: UIPanGestureRecognizer) {
        let translation = g.translation(in: view)
        let velocity    = g.velocity(in: view)
        let height      = view.bounds.height

        switch g.state {
        case .began:
            interactiveTransition = UIPercentDrivenInteractiveTransition()
            interactiveTransition?.completionCurve = .easeOut
            dismiss(animated: true)

        case .changed:
            var percent = translation.y / height
            if percent < 0 { percent /= (1 + abs(percent) * 3) }   // rubber-band
            interactiveTransition?.update(max(0, percent))

        case .ended, .cancelled:
            let percent = translation.y / height
            if percent > 0.4 || velocity.y > 800 {
                interactiveTransition?.finish()
            } else {
                interactiveTransition?.cancel()
            }
            interactiveTransition = nil

        default: break
        }
    }

    // Animator สำหรับ dismiss
    class SlideDownAnimator: NSObject, UIViewControllerAnimatedTransitioning {
        func transitionDuration(using ctx: UIViewControllerContextTransitioning?) -> TimeInterval { 0.4 }
        func animateTransition(using ctx: UIViewControllerContextTransitioning) {
            guard let fromView = ctx.view(forKey: .from) else { return }
            UIView.animate(withDuration: 0.4, options: .curveEaseInOut) {
                fromView.frame.origin.y = fromView.frame.maxY
            } completion: { _ in
                fromView.removeFromSuperview()
                ctx.completeTransition(!ctx.transitionWasCancelled)
            }
        }
    }
}

// Transitioning Delegate
class InteractiveDismissDelegate: NSObject, UIViewControllerTransitioningDelegate {
    weak var vc: InteractiveDismissVC?

    func animationController(forDismissed d: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        InteractiveDismissVC.SlideDownAnimator()
    }
    func interactionControllerForDismissal(
        using a: UIViewControllerAnimatedTransitioning) -> UIViewControllerInteractiveTransitioning? {
        vc?.interactiveTransition
    }
}
```

---

## 9. UIKit Testing

### การตั้งค่า Accessibility สำหรับ Testing

Accessibility Identifier ต่างจาก Accessibility Label — identifier ใช้สำหรับ testing เท่านั้น ไม่อ่านออกเสียงโดย VoiceOver

```swift
// กำหนดใน code
button.accessibilityIdentifier = "submitButton"
tableView.accessibilityIdentifier = "mainTableView"
label.accessibilityIdentifier = "welcomeLabel"

// กำหนดใน Interface Builder: Identity Inspector → Accessibility → Identifier
```

**Element Types ใน XCUITest:**
```swift
app.buttons["id"]            // UIButton
app.textFields["id"]         // UITextField (not secure)
app.secureTextFields["id"]   // UITextField (isSecureTextEntry)
app.labels["id"]             // UILabel
app.tables["id"]             // UITableView
app.cells["id"]              // UITableViewCell / UICollectionViewCell
app.navigationBars["title"]  // UINavigationBar
app.switches["id"]           // UISwitch
app.sliders["id"]            // UISlider
app.images["id"]             // UIImageView
```

### XCUITest Basics

```swift
import XCTest

class LoginUITests: XCTestCase {
    let app = XCUIApplication()

    override func setUpWithError() throws {
        continueAfterFailure = false
        app.launch()
    }

    func testLoginFlow() {
        let emailField    = app.textFields["loginEmailField"]
        let passwordField = app.secureTextFields["loginPasswordField"]
        let loginButton   = app.buttons["loginButton"]

        XCTAssertTrue(emailField.waitForExistence(timeout: 5))
        emailField.tap();    emailField.typeText("user@test.com")
        passwordField.tap(); passwordField.typeText("password123")
        loginButton.tap()

        XCTAssertTrue(app.navigationBars["หน้าหลัก"].waitForExistence(timeout: 5))
    }
}
```

### Unit Testing ViewController

```swift
import XCTest
@testable import MyApp

class LoginViewControllerTests: XCTestCase {
    var sut: LoginViewController!

    override func setUpWithError() throws {
        sut = LoginViewController(); sut.loadViewIfNeeded()
    }
    override func tearDownWithError() throws { sut = nil }

    func testEmailValidation_empty_returnsFalse() {
        XCTAssertFalse(sut.validateEmail(""))
    }
    func testEmailValidation_valid_returnsTrue() {
        XCTAssertTrue(sut.validateEmail("test@example.com"))
    }
    func testLoginButton_disabledWhenEmpty() {
        sut.emailTextField.text = ""; sut.passwordTextField.text = ""
        sut.textFieldDidChange()
        XCTAssertFalse(sut.loginButton.isEnabled)
    }
}
```

### ตัวอย่าง: Login Form พร้อม Accessibility Identifiers

```swift
import UIKit

class LoginViewController: UIViewController {
    lazy var emailTextField: UITextField = {
        let tf = UITextField()
        tf.placeholder = "อีเมล"; tf.keyboardType = .emailAddress
        tf.autocapitalizationType = .none; tf.borderStyle = .roundedRect
        tf.accessibilityIdentifier = "loginEmailField"; return tf
    }()
    lazy var passwordTextField: UITextField = {
        let tf = UITextField()
        tf.placeholder = "รหัสผ่าน"; tf.isSecureTextEntry = true
        tf.borderStyle = .roundedRect
        tf.accessibilityIdentifier = "loginPasswordField"; return tf
    }()
    lazy var loginButton: UIButton = {
        let btn = UIButton(type: .system)
        btn.setTitle("เข้าสู่ระบบ", for: .normal); btn.isEnabled = false
        btn.accessibilityIdentifier = "loginButton"; return btn
    }()

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "เข้าสู่ระบบ"; view.backgroundColor = .systemBackground
        let stack = UIStackView(arrangedSubviews: [emailTextField, passwordTextField, loginButton])
        stack.axis = .vertical; stack.spacing = 16
        stack.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(stack)
        NSLayoutConstraint.activate([
            stack.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 32),
            stack.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -32),
            stack.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
        [emailTextField, passwordTextField].forEach {
            $0.addTarget(self, action: #selector(textFieldDidChange), for: .editingChanged)
        }
        loginButton.addTarget(self, action: #selector(loginTapped), for: .touchUpInside)
    }

    func validateEmail(_ email: String) -> Bool {
        email.range(of: "[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}",
                    options: .regularExpression) != nil
    }

    @objc func textFieldDidChange() {
        loginButton.isEnabled = validateEmail(emailTextField.text ?? "")
            && (passwordTextField.text?.count ?? 0) >= 6
    }

    @objc func loginTapped() { print("เข้าสู่ระบบ...") }
}
```

---

## 10. UIPageViewController

### แนวคิด UIPageViewController

UIPageViewController จัดการการ navigate ระหว่าง View Controller ต่างๆ โดยใช้ gesture swipe เหมือน book page

**Transition Styles:**
- `.scroll` — swipe แบบเลื่อน (เหมือน scroll view)
- `.pageCurl` — พลิกหน้าเหมือนหนังสือจริง

**Navigation Orientations:**
- `.horizontal` — swipe ซ้าย/ขวา
- `.vertical` — swipe ขึ้น/ลง

**Spine Location (สำหรับ pageCurl เท่านั้น):**
- `.min` — สันอยู่ด้านซ้ายหรือบน (แสดงครั้งละ 1 หน้า)
- `.mid` — สันอยู่กลาง (แสดงครั้งละ 2 หน้า)

```swift
// Data Source: กำหนดหน้าก่อน/หลัง
func pageViewController(_ pvc: UIPageViewController,
                        viewControllerBefore vc: UIViewController) -> UIViewController?
func pageViewController(_ pvc: UIPageViewController,
                        viewControllerAfter vc: UIViewController) -> UIViewController?

// ตั้งหน้าเริ่มต้น
pageVC.setViewControllers([firstVC], direction: .forward, animated: false)

// เปลี่ยนหน้าแบบ programmatic
pageVC.setViewControllers([targetVC], direction: .forward, animated: true)
```

**เทคนิค:** เก็บ index ไว้ใน ViewController แต่ละหน้า เพื่อให้ dataSource ใช้หา before/after VC ได้ง่าย

### ตัวอย่าง: Onboarding Flow

```swift
import UIKit

struct OnboardingPage {
    let title: String; let subtitle: String
    let systemImage: String; let backgroundColor: UIColor
}

class OnboardingViewController: UIViewController {
    private let pages: [OnboardingPage] = [
        OnboardingPage(title: "ยินดีต้อนรับ", subtitle: "แอปช่วยให้ชีวิตง่ายขึ้น",
                       systemImage: "hand.wave.fill", backgroundColor: .systemBlue),
        OnboardingPage(title: "ติดตามทุกอย่าง", subtitle: "จัดการงานและเป้าหมาย",
                       systemImage: "checkmark.circle.fill", backgroundColor: .systemGreen),
        OnboardingPage(title: "เริ่มต้นได้เลย", subtitle: "สร้างบัญชีหรือเข้าสู่ระบบ",
                       systemImage: "arrow.right.circle.fill", backgroundColor: .systemOrange)
    ]

    private lazy var pageVC = UIPageViewController(transitionStyle: .scroll,
                                                   navigationOrientation: .horizontal)
    private let pageControl = UIPageControl()
    private let nextButton  = UIButton(type: .system)
    private var currentIndex = 0

    override func viewDidLoad() {
        super.viewDidLoad()
        pageVC.dataSource = self; pageVC.delegate = self
        addChild(pageVC); view.addSubview(pageVC.view)
        pageVC.view.frame = view.bounds; pageVC.didMove(toParent: self)
        pageVC.setViewControllers([makePageVC(0)!], direction: .forward, animated: false)

        pageControl.numberOfPages = pages.count
        pageControl.currentPageIndicatorTintColor = .white
        pageControl.pageIndicatorTintColor = UIColor.white.withAlphaComponent(0.4)

        nextButton.setTitle("ถัดไป", for: .normal)
        nextButton.tintColor = .white
        nextButton.titleLabel?.font = .boldSystemFont(ofSize: 18)
        nextButton.addTarget(self, action: #selector(nextTapped), for: .touchUpInside)

        [pageControl, nextButton].forEach {
            $0.translatesAutoresizingMaskIntoConstraints = false; view.addSubview($0)
        }
        NSLayoutConstraint.activate([
            pageControl.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            pageControl.bottomAnchor.constraint(equalTo: view.safeAreaLayoutGuide.bottomAnchor, constant: -60),
            nextButton.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            nextButton.bottomAnchor.constraint(equalTo: view.safeAreaLayoutGuide.bottomAnchor, constant: -20)
        ])
    }

    private func makePageVC(_ index: Int) -> OnboardingPageContentVC? {
        guard (0..<pages.count).contains(index) else { return nil }
        let vc = OnboardingPageContentVC()
        vc.configure(with: pages[index]); vc.pageIndex = index; return vc
    }

    @objc func nextTapped() {
        let next = currentIndex + 1
        guard next < pages.count else { print("Onboarding เสร็จสิ้น!"); return }
        pageVC.setViewControllers([makePageVC(next)!], direction: .forward, animated: true)
        currentIndex = next; pageControl.currentPage = next
        if next == pages.count - 1 { nextButton.setTitle("เริ่มต้น", for: .normal) }
    }
}

extension OnboardingViewController: UIPageViewControllerDataSource {
    func pageViewController(_ p: UIPageViewController, viewControllerBefore vc: UIViewController) -> UIViewController? {
        makePageVC((vc as! OnboardingPageContentVC).pageIndex - 1)
    }
    func pageViewController(_ p: UIPageViewController, viewControllerAfter vc: UIViewController) -> UIViewController? {
        makePageVC((vc as! OnboardingPageContentVC).pageIndex + 1)
    }
}

extension OnboardingViewController: UIPageViewControllerDelegate {
    func pageViewController(_ p: UIPageViewController, didFinishAnimating finished: Bool,
                            previousViewControllers: [UIViewController], transitionCompleted ok: Bool) {
        guard ok, let cur = p.viewControllers?.first as? OnboardingPageContentVC else { return }
        currentIndex = cur.pageIndex; pageControl.currentPage = currentIndex
        nextButton.setTitle(currentIndex == pages.count - 1 ? "เริ่มต้น" : "ถัดไป", for: .normal)
    }
}

class OnboardingPageContentVC: UIViewController {
    var pageIndex = 0
    private let imageView  = UIImageView()
    private let titleLabel = UILabel()
    private let subtitleLabel = UILabel()

    override func viewDidLoad() {
        super.viewDidLoad()
        imageView.contentMode = .scaleAspectFit
        titleLabel.font = .boldSystemFont(ofSize: 28)
        titleLabel.textColor = .white; titleLabel.textAlignment = .center
        subtitleLabel.font = .systemFont(ofSize: 16)
        subtitleLabel.textColor = UIColor.white.withAlphaComponent(0.85)
        subtitleLabel.textAlignment = .center; subtitleLabel.numberOfLines = 0

        let stack = UIStackView(arrangedSubviews: [imageView, titleLabel, subtitleLabel])
        stack.axis = .vertical; stack.spacing = 24; stack.alignment = .center
        stack.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(stack)
        NSLayoutConstraint.activate([
            imageView.heightAnchor.constraint(equalToConstant: 120),
            stack.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            stack.centerYAnchor.constraint(equalTo: view.centerYAnchor, constant: -40),
            stack.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 32),
            stack.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -32)
        ])
    }

    func configure(with page: OnboardingPage) {
        view.backgroundColor = page.backgroundColor
        let cfg = UIImage.SymbolConfiguration(pointSize: 80)
        imageView.image = UIImage(systemName: page.systemImage, withConfiguration: cfg)
        imageView.tintColor = .white
        titleLabel.text = page.title; subtitleLabel.text = page.subtitle
    }
}
```

---

## 11. แบบฝึกหัด

### แบบฝึกหัดที่ 1: Draggable Floating Button

**โจทย์:** สร้าง FAB ที่ลากได้และ snap เข้าขอบซ้าย/ขวา

```swift
import UIKit

class DraggableFAB: UIButton {
    private var initialCenter = CGPoint.zero

    override init(frame: CGRect) { super.init(frame: frame); setup() }
    required init?(coder: NSCoder) { super.init(coder: coder); setup() }

    private func setup() {
        backgroundColor = .systemBlue; layer.cornerRadius = frame.width / 2
        tintColor = .white; clipsToBounds = false
        let cfg = UIImage.SymbolConfiguration(pointSize: 24, weight: .bold)
        setImage(UIImage(systemName: "plus", withConfiguration: cfg), for: .normal)
        layer.shadowColor = UIColor.black.cgColor; layer.shadowOpacity = 0.3
        layer.shadowOffset = CGSize(width: 0, height: 4); layer.shadowRadius = 8

        let pan = UIPanGestureRecognizer(target: self, action: #selector(handlePan(_:)))
        addGestureRecognizer(pan)
    }

    @objc private func handlePan(_ g: UIPanGestureRecognizer) {
        guard let sv = superview else { return }
        let t = g.translation(in: sv)

        switch g.state {
        case .began:
            initialCenter = center
            UIView.animate(withDuration: 0.1) { self.transform = CGAffineTransform(scaleX: 1.1, y: 1.1) }

        case .changed:
            let hw = bounds.width / 2, hh = bounds.height / 2
            let safeT = sv.safeAreaInsets.top, safeB = sv.safeAreaInsets.bottom
            center = CGPoint(
                x: min(max(initialCenter.x + t.x, hw), sv.bounds.width - hw),
                y: min(max(initialCenter.y + t.y, hh + safeT), sv.bounds.height - hh - safeB)
            )

        case .ended, .cancelled:
            UIView.animate(withDuration: 0.2) { self.transform = .identity }
            snapToEdge(in: sv)

        default: break
        }
    }

    private func snapToEdge(in sv: UIView) {
        let padding: CGFloat = 16, hw = bounds.width / 2
        let targetX = center.x > sv.bounds.midX ? sv.bounds.width - hw - padding : hw + padding
        UIView.animate(withDuration: 0.45, delay: 0, usingSpringWithDamping: 0.65,
                       initialSpringVelocity: 0.8, options: .curveEaseOut) {
            self.center.x = targetX
        }
    }
}

// การใช้งาน
class FABDemoVC: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemGroupedBackground
        let fab = DraggableFAB(frame: CGRect(x: 0, y: 0, width: 60, height: 60))
        fab.layer.cornerRadius = 30
        fab.center = CGPoint(x: view.bounds.width - 46, y: view.bounds.height - 120)
        fab.addTarget(self, action: #selector(fabTapped), for: .touchUpInside)
        view.addSubview(fab)
    }
    @objc func fabTapped() { print("FAB tapped!") }
}
```

---

### แบบฝึกหัดที่ 2: Custom Bottom Sheet ด้วย Pan Gesture

**โจทย์:** Bottom sheet ที่มี 3 snap points (20%, 50%, 90%) ลากขึ้น/ลงได้

```swift
import UIKit

class BottomSheetVC: UIViewController {
    enum Snap { case collapsed, half, expanded
        func offset(for h: CGFloat) -> CGFloat {
            switch self {
            case .collapsed: return h * 0.80
            case .half:      return h * 0.50
            case .expanded:  return h * 0.10
            }
        }
    }

    private var currentSnap: Snap = .collapsed
    private var panStartOffset: CGFloat = 0
    private var sheetTopConstraint: NSLayoutConstraint!

    private let dimmingView: UIView = {
        let v = UIView(); v.backgroundColor = UIColor.black.withAlphaComponent(0.4); v.alpha = 0; return v
    }()
    private let sheetView: UIView = {
        let v = UIView(); v.backgroundColor = .systemBackground
        v.layer.cornerRadius = 20; v.layer.maskedCorners = [.layerMinXMinYCorner, .layerMaxXMinYCorner]
        v.layer.shadowColor = UIColor.black.cgColor; v.layer.shadowOpacity = 0.15
        v.layer.shadowOffset = CGSize(width: 0, height: -4); v.layer.shadowRadius = 12
        return v
    }()
    private let handleView: UIView = {
        let v = UIView(); v.backgroundColor = .systemGray4; v.layer.cornerRadius = 3; return v
    }()

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemTeal
        setupViews()
    }

    override func viewDidLayoutSubviews() {
        super.viewDidLayoutSubviews()
        updateSheet(to: currentSnap, animated: false)
    }

    private func setupViews() {
        dimmingView.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(dimmingView)
        NSLayoutConstraint.activate([
            dimmingView.topAnchor.constraint(equalTo: view.topAnchor),
            dimmingView.bottomAnchor.constraint(equalTo: view.bottomAnchor),
            dimmingView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            dimmingView.trailingAnchor.constraint(equalTo: view.trailingAnchor)
        ])

        sheetView.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(sheetView)
        sheetTopConstraint = sheetView.topAnchor.constraint(equalTo: view.topAnchor,
                                                            constant: view.bounds.height)
        NSLayoutConstraint.activate([
            sheetTopConstraint,
            sheetView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            sheetView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            sheetView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
        ])

        handleView.translatesAutoresizingMaskIntoConstraints = false
        sheetView.addSubview(handleView)
        NSLayoutConstraint.activate([
            handleView.topAnchor.constraint(equalTo: sheetView.topAnchor, constant: 10),
            handleView.centerXAnchor.constraint(equalTo: sheetView.centerXAnchor),
            handleView.widthAnchor.constraint(equalToConstant: 40),
            handleView.heightAnchor.constraint(equalToConstant: 6)
        ])

        let content = UILabel()
        content.text = "ลากขึ้น-ลงเพื่อเปิด/ปิด\nBottom Sheet"
        content.numberOfLines = 0; content.textAlignment = .center
        content.translatesAutoresizingMaskIntoConstraints = false
        sheetView.addSubview(content)
        NSLayoutConstraint.activate([
            content.topAnchor.constraint(equalTo: handleView.bottomAnchor, constant: 24),
            content.leadingAnchor.constraint(equalTo: sheetView.leadingAnchor, constant: 24),
            content.trailingAnchor.constraint(equalTo: sheetView.trailingAnchor, constant: -24)
        ])

        let pan = UIPanGestureRecognizer(target: self, action: #selector(handlePan(_:)))
        sheetView.addGestureRecognizer(pan)

        let tap = UITapGestureRecognizer(target: self, action: #selector(dimmingTapped))
        dimmingView.addGestureRecognizer(tap)

        let openBtn = UIButton(type: .system)
        openBtn.setTitle("เปิด Bottom Sheet", for: .normal); openBtn.tintColor = .white
        openBtn.titleLabel?.font = .boldSystemFont(ofSize: 18)
        openBtn.translatesAutoresizingMaskIntoConstraints = false
        view.insertSubview(openBtn, belowSubview: dimmingView)
        NSLayoutConstraint.activate([
            openBtn.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            openBtn.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
        openBtn.addTarget(self, action: #selector(openSheet), for: .touchUpInside)
    }

    @objc private func openSheet()     { updateSheet(to: .half,      animated: true) }
    @objc private func dimmingTapped() { updateSheet(to: .collapsed, animated: true) }

    @objc private func handlePan(_ g: UIPanGestureRecognizer) {
        let ty = g.translation(in: view).y
        let vy = g.velocity(in: view).y
        let h  = view.bounds.height

        switch g.state {
        case .began:
            panStartOffset = sheetTopConstraint.constant

        case .changed:
            let newTop = panStartOffset + ty
            sheetTopConstraint.constant = max(newTop, h * 0.10)
            let progress = 1 - ((sheetTopConstraint.constant - h * 0.10) / (h * 0.70))
            dimmingView.alpha = min(max(progress * 0.4, 0), 0.4)

        case .ended, .cancelled:
            let snap = nearestSnap(offset: sheetTopConstraint.constant, velocity: vy, total: h)
            updateSheet(to: snap, animated: true)

        default: break
        }
    }

    private func nearestSnap(offset: CGFloat, velocity: CGFloat, total: CGFloat) -> Snap {
        if velocity > 1000  { return .collapsed }
        if velocity < -1000 { return .expanded  }
        let snaps: [Snap] = [.collapsed, .half, .expanded]
        return snaps.min { abs($0.offset(for: total) - offset) < abs($1.offset(for: total) - offset) }
            ?? .collapsed
    }

    private func updateSheet(to snap: Snap, animated: Bool) {
        currentSnap = snap
        sheetTopConstraint.constant = snap.offset(for: view.bounds.height)
        let dimming: CGFloat = snap == .collapsed ? 0 : 0.4
        if animated {
            UIView.animate(withDuration: 0.45, delay: 0,
                           usingSpringWithDamping: 0.8, initialSpringVelocity: 0.5,
                           options: .curveEaseOut) {
                self.view.layoutIfNeeded(); self.dimmingView.alpha = dimming
            }
        } else {
            view.layoutIfNeeded(); dimmingView.alpha = dimming
        }
    }
}
```

---

## สรุปและแนวทางปฏิบัติที่ดี

| หัวข้อ | สิ่งสำคัญ | ข้อควรระวัง |
|---|---|---|
| Responder Chain | hitTest, pointInside, first responder | อย่าลืม call super ใน touchesBegan ถ้าไม่จัดการ |
| Custom Transitions | AnimatedTransitioning, PresentationController | ต้อง completeTransition เสมอ แม้ cancelled |
| Compositional Layout | Item/Group/Section, DiffableDataSource | ใช้ estimated size สำหรับ dynamic content |
| CollectionView List | ListConfiguration, UICellAccessory | reloadData ใช้ animatingDifferences: false ครั้งแรก |
| Custom UIControl | Target-action, touch tracking | เรียก sendActions ใน endTracking เท่านั้น |
| CALayer Animations | BasicAnimation, KeyframeAnimation, CADisplayLink | invalidate CADisplayLink เมื่อเลิกใช้ |
| UIKit Performance | Prefetching, shouldRasterize, off-screen rendering | วัดด้วย Instruments ก่อน optimize |
| Interactive Dismissal | UIPercentDrivenInteractiveTransition | ต้อง finish/cancel เสมอ ไม่งั้น transition ค้าง |
| UIKit Testing | XCUITest, Unit testing VC | แยก business logic ออกจาก VC เพื่อ test ง่าย |
| UIPageViewController | DataSource, Delegate, onboarding | คืน nil จาก before/after เพื่อหยุดที่ขอบ |

### Checklist ก่อน Release

**Performance:**
- [ ] เปิด Instruments Time Profiler ตรวจหา hot spot
- [ ] เปิด Core Animation instrument ตรวจ FPS
- [ ] เปิด Color Off-screen Rendered ใน Simulator
- [ ] ตรวจสอบ memory leak ด้วย Leaks instrument
- [ ] ทดสอบบนอุปกรณ์จริง ไม่ใช่แค่ Simulator

**Accessibility:**
- [ ] ตั้ง accessibilityIdentifier สำหรับ interactive elements
- [ ] ตั้ง accessibilityLabel สำหรับ VoiceOver
- [ ] ทดสอบด้วย VoiceOver เปิด

**Testing:**
- [ ] Unit test ครอบคลุม validation logic
- [ ] UI test ครอบคลุม happy path หลัก
- [ ] ทดสอบบนหลาย screen sizes

### Pattern ที่ควรจำ

```swift
// 1. DiffableDataSource snapshot pattern
var snapshot = NSDiffableDataSourceSnapshot<Section, Item>()
snapshot.appendSections([.main])
snapshot.appendItems(newItems)
dataSource.apply(snapshot, animatingDifferences: true)

// 2. Compositional Layout helper pattern
static func makeSection(columns: Int, height: CGFloat) -> NSCollectionLayoutSection {
    let item = NSCollectionLayoutItem(
        layoutSize: .init(widthDimension: .fractionalWidth(1 / CGFloat(columns)),
                          heightDimension: .fractionalHeight(1)))
    let group = NSCollectionLayoutGroup.horizontal(
        layoutSize: .init(widthDimension: .fractionalWidth(1),
                          heightDimension: .absolute(height)),
        subitems: [item])
    return NSCollectionLayoutSection(group: group)
}

// 3. Spring animation shorthand
UIView.animate(withDuration: 0.5, delay: 0,
               usingSpringWithDamping: 0.7, initialSpringVelocity: 0.5,
               options: .curveEaseOut) {
    view.transform = .identity
}

// 4. Safe CALayer animation (ป้องกัน jump-back)
let anim = CABasicAnimation(keyPath: "opacity")
anim.toValue = 0
anim.duration = 0.3
anim.fillMode = .forwards
anim.isRemovedOnCompletion = false
layer.opacity = 0          // อัปเดต model layer ก่อน
layer.add(anim, forKey: "fadeOut")
```

> **เคล็ดลับ:** ใช้ Instruments (Time Profiler + Core Animation) วัด performance จริงก่อนเสมอ อย่าเดา bottleneck

### บทต่อไป

- **Part 98** — SwiftUI Interoperability (UIHostingController, UIViewRepresentable)
- **Part 99** — Combine Framework กับ UIKit
- **Part 100** — Course Conclusion & Mastery Review

---

*Part 97 — Advanced UIKit Mastery | Swift Course*
