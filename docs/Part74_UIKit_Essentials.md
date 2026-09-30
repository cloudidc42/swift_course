# Part 74: UIKit Essentials — พื้นฐาน UIKit สำหรับ iOS Development

UIKit เป็น framework หลักสำหรับสร้าง User Interface บน iOS มาตั้งแต่ปี 2008 แม้ว่า SwiftUI จะเปิดตัวในปี 2019 แต่ UIKit ยังคงมีบทบาทสำคัญในหลายโปรเจกต์ บทนี้ครอบคลุมแนวคิดหลักและ API ที่ใช้บ่อยที่สุดใน UIKit

---

## 1. UIKit vs SwiftUI

### เมื่อไหรควรเลือก UIKit

| เกณฑ์ | UIKit | SwiftUI |
|-------|-------|---------|
| iOS version target | iOS 12 หรือต่ำกว่า | iOS 14+ (แนะนำ iOS 16+) |
| Codebase เดิม | ใช้ UIKit อยู่แล้ว | โปรเจกต์ใหม่ |
| Custom animations | ควบคุมได้ละเอียดกว่า | จำกัดกว่า |
| Third-party libraries | library เก่ายังรองรับ UIKit | library ใหม่รองรับ SwiftUI |
| Team expertise | ทีมคุ้นเคย UIKit | ทีมเรียนรู้ใหม่ |

### Migration Path

แนวทางการย้ายจาก UIKit ไป SwiftUI แบบค่อยเป็นค่อยไป:
1. เริ่มจากหน้าจอใหม่ด้วย SwiftUI
2. ใช้ `UIHostingController` เพื่อฝัง SwiftUI ใน UIKit
3. ใช้ `UIViewRepresentable` / `UIViewControllerRepresentable` เพื่อใช้ UIKit ใน SwiftUI
4. ค่อยๆ แทนที่ UIKit screen ด้วย SwiftUI ทีละหน้า

### Interoperability Overview

```swift
// UIKit -> SwiftUI: ใช้ UIHostingController
let swiftUIView = ContentView()
let hostingVC = UIHostingController(rootView: swiftUIView)
present(hostingVC, animated: true)

// SwiftUI -> UIKit: ใช้ UIViewRepresentable
struct MyUIKitWrapper: UIViewRepresentable {
    func makeUIView(context: Context) -> UITextField {
        UITextField()
    }
    func updateUIView(_ uiView: UITextField, context: Context) {}
}
```

---

## 2. UIViewController Lifecycle

### วงจรชีวิตของ UIViewController

```
init → loadView → viewDidLoad → viewWillAppear → viewDidAppear
                                                       ↓
                               viewWillDisappear ← (user leaves)
                                       ↓
                               viewDidDisappear
```

### ตัวอย่างสมบูรณ์: Lifecycle Logging

```swift
import UIKit

class LifecycleViewController: UIViewController {

    private let label = UILabel()

    // MARK: - Initialization
    override init(nibName nibNameOrNil: String?, bundle nibBundleOrNil: Bundle?) {
        super.init(nibName: nibNameOrNil, bundle: nibBundleOrNil)
        print("🔵 init — ViewController ถูกสร้าง")
    }

    required init?(coder: NSCoder) {
        super.init(coder: coder)
        print("🔵 init(coder:) — ถูกสร้างจาก Storyboard")
    }

    // MARK: - View Loading
    override func loadView() {
        super.loadView()
        print("⚪ loadView — กำลังโหลด view hierarchy")
    }

    override func viewDidLoad() {
        super.viewDidLoad()
        print("🟢 viewDidLoad — view โหลดเสร็จ เรียกครั้งเดียวตลอดชีวิต")
        setupUI()
    }

    // MARK: - View Appearance
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        print("🟡 viewWillAppear — view กำลังจะแสดง (อาจเรียกหลายครั้ง)")
        // เหมาะสำหรับ: refresh data, start animations
    }

    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        print("🟠 viewDidAppear — view แสดงเสร็จแล้ว")
        // เหมาะสำหรับ: start timers, play video, track analytics
    }

    // MARK: - View Disappearance
    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        print("🔴 viewWillDisappear — view กำลังจะหายไป")
        // เหมาะสำหรับ: save data, stop animations
    }

    override func viewDidDisappear(_ animated: Bool) {
        super.viewDidDisappear(animated)
        print("⚫ viewDidDisappear — view หายไปแล้ว")
        // เหมาะสำหรับ: stop timers, release resources
    }

    // MARK: - Setup
    private func setupUI() {
        view.backgroundColor = .systemBackground
        label.text = "ดู Console สำหรับ lifecycle logs"
        label.numberOfLines = 0
        label.textAlignment = .center
        label.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(label)
        NSLayoutConstraint.activate([
            label.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            label.centerYAnchor.constraint(equalTo: view.centerYAnchor),
            label.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 20),
            label.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -20)
        ])
    }

    deinit {
        print("🗑 deinit — ViewController ถูก deallocate")
    }
}
```

---

## 3. UIView และ Auto Layout

Auto Layout ใช้ constraint กำหนดตำแหน่ง/ขนาด view ผ่าน Anchor API; UIStackView จัด view แนวตั้ง/แนวนอนโดยอัตโนมัติ

### ตัวอย่างสมบูรณ์: Programmatic UI (ไม่ใช้ Storyboard)

```swift
import UIKit

class ProfileViewController: UIViewController {

    // MARK: - UI Components
    private let scrollView = UIScrollView()
    private let contentView = UIView()
    private let avatarImageView = UIImageView()
    private let nameLabel = UILabel()
    private let emailLabel = UILabel()
    private let bioLabel = UILabel()
    private let followButton = UIButton(type: .system)
    private let statsStack = UIStackView()

    // MARK: - Lifecycle
    override func viewDidLoad() {
        super.viewDidLoad()
        setupScrollView()
        setupAvatar()
        setupLabels()
        setupStatsStack()
        setupFollowButton()
        configureContent()
    }

    // MARK: - Setup Methods
    private func setupScrollView() {
        view.backgroundColor = .systemBackground
        scrollView.translatesAutoresizingMaskIntoConstraints = false
        contentView.translatesAutoresizingMaskIntoConstraints = false

        view.addSubview(scrollView)
        scrollView.addSubview(contentView)

        NSLayoutConstraint.activate([
            // ScrollView เต็ม safe area
            scrollView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            scrollView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            scrollView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            scrollView.bottomAnchor.constraint(equalTo: view.bottomAnchor),

            // ContentView ขนาดเท่า ScrollView (กว้าง), สูงได้ตามเนื้อหา
            contentView.topAnchor.constraint(equalTo: scrollView.topAnchor),
            contentView.leadingAnchor.constraint(equalTo: scrollView.leadingAnchor),
            contentView.trailingAnchor.constraint(equalTo: scrollView.trailingAnchor),
            contentView.bottomAnchor.constraint(equalTo: scrollView.bottomAnchor),
            contentView.widthAnchor.constraint(equalTo: scrollView.widthAnchor)
        ])
    }

    private func setupAvatar() {
        avatarImageView.translatesAutoresizingMaskIntoConstraints = false
        avatarImageView.contentMode = .scaleAspectFill
        avatarImageView.clipsToBounds = true
        avatarImageView.backgroundColor = .systemGray4
        avatarImageView.layer.cornerRadius = 50
        avatarImageView.layer.borderWidth = 3
        avatarImageView.layer.borderColor = UIColor.systemBlue.cgColor
        contentView.addSubview(avatarImageView)

        NSLayoutConstraint.activate([
            avatarImageView.topAnchor.constraint(equalTo: contentView.topAnchor, constant: 24),
            avatarImageView.centerXAnchor.constraint(equalTo: contentView.centerXAnchor),
            avatarImageView.widthAnchor.constraint(equalToConstant: 100),
            avatarImageView.heightAnchor.constraint(equalToConstant: 100)
        ])
    }

    private func setupLabels() {
        // Name Label
        nameLabel.translatesAutoresizingMaskIntoConstraints = false
        nameLabel.font = .systemFont(ofSize: 22, weight: .bold)
        nameLabel.textAlignment = .center
        contentView.addSubview(nameLabel)

        // Email Label
        emailLabel.translatesAutoresizingMaskIntoConstraints = false
        emailLabel.font = .systemFont(ofSize: 14)
        emailLabel.textColor = .secondaryLabel
        emailLabel.textAlignment = .center
        contentView.addSubview(emailLabel)

        // Bio Label
        bioLabel.translatesAutoresizingMaskIntoConstraints = false
        bioLabel.font = .systemFont(ofSize: 15)
        bioLabel.numberOfLines = 0
        bioLabel.textAlignment = .center
        bioLabel.textColor = .label
        contentView.addSubview(bioLabel)

        NSLayoutConstraint.activate([
            nameLabel.topAnchor.constraint(equalTo: avatarImageView.bottomAnchor, constant: 12),
            nameLabel.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 16),
            nameLabel.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: -16),

            emailLabel.topAnchor.constraint(equalTo: nameLabel.bottomAnchor, constant: 4),
            emailLabel.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 16),
            emailLabel.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: -16),

            bioLabel.topAnchor.constraint(equalTo: emailLabel.bottomAnchor, constant: 12),
            bioLabel.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 24),
            bioLabel.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: -24)
        ])
    }

    private func setupStatsStack() {
        statsStack.translatesAutoresizingMaskIntoConstraints = false
        statsStack.axis = .horizontal
        statsStack.distribution = .fillEqually
        statsStack.spacing = 1
        statsStack.backgroundColor = .separator

        let stats = [("128", "Posts"), ("4.2K", "Followers"), ("312", "Following")]
        for (value, title) in stats {
            let statView = createStatView(value: value, title: title)
            statsStack.addArrangedSubview(statView)
        }
        contentView.addSubview(statsStack)

        NSLayoutConstraint.activate([
            statsStack.topAnchor.constraint(equalTo: bioLabel.bottomAnchor, constant: 20),
            statsStack.leadingAnchor.constraint(equalTo: contentView.leadingAnchor),
            statsStack.trailingAnchor.constraint(equalTo: contentView.trailingAnchor),
            statsStack.heightAnchor.constraint(equalToConstant: 60)
        ])
    }

    private func createStatView(value: String, title: String) -> UIView {
        let container = UIView()
        container.backgroundColor = .systemBackground

        let valueLabel = UILabel()
        valueLabel.text = value
        valueLabel.font = .systemFont(ofSize: 18, weight: .bold)
        valueLabel.textAlignment = .center

        let titleLabel = UILabel()
        titleLabel.text = title
        titleLabel.font = .systemFont(ofSize: 12)
        titleLabel.textColor = .secondaryLabel
        titleLabel.textAlignment = .center

        let stack = UIStackView(arrangedSubviews: [valueLabel, titleLabel])
        stack.axis = .vertical
        stack.spacing = 2
        stack.translatesAutoresizingMaskIntoConstraints = false
        container.addSubview(stack)

        NSLayoutConstraint.activate([
            stack.centerXAnchor.constraint(equalTo: container.centerXAnchor),
            stack.centerYAnchor.constraint(equalTo: container.centerYAnchor)
        ])
        return container
    }

    private func setupFollowButton() {
        followButton.translatesAutoresizingMaskIntoConstraints = false
        followButton.setTitle("Follow", for: .normal)
        followButton.backgroundColor = .systemBlue
        followButton.setTitleColor(.white, for: .normal)
        followButton.titleLabel?.font = .systemFont(ofSize: 16, weight: .semibold)
        followButton.layer.cornerRadius = 12
        followButton.addTarget(self, action: #selector(followTapped), for: .touchUpInside)
        contentView.addSubview(followButton)

        NSLayoutConstraint.activate([
            followButton.topAnchor.constraint(equalTo: statsStack.bottomAnchor, constant: 20),
            followButton.centerXAnchor.constraint(equalTo: contentView.centerXAnchor),
            followButton.widthAnchor.constraint(equalToConstant: 160),
            followButton.heightAnchor.constraint(equalToConstant: 44),
            followButton.bottomAnchor.constraint(equalTo: contentView.bottomAnchor, constant: -24)
        ])
    }

    private func configureContent() {
        avatarImageView.image = UIImage(systemName: "person.circle.fill")
        avatarImageView.tintColor = .systemBlue
        nameLabel.text = "สมชาย ใจดี"
        emailLabel.text = "somchai@example.com"
        bioLabel.text = "iOS Developer ที่รัก Swift และ UIKit เป็นชีวิตจิตใจ ☕️"
    }

    @objc private func followTapped() {
        let isFollowing = followButton.title(for: .normal) == "Following"
        followButton.setTitle(isFollowing ? "Follow" : "Following", for: .normal)
        followButton.backgroundColor = isFollowing ? .systemBlue : .systemGray4
    }
}
```

---

## 4. UITableView

UITableView เป็น component ที่ใช้แสดงรายการข้อมูลแบบ scrollable ซึ่งพบได้ทั่วไปใน iOS

### ตัวอย่างสมบูรณ์: Contacts List App

```swift
import UIKit

// MARK: - Data Model
struct Contact {
    let id: UUID
    var name: String
    var phone: String
    var isFavorite: Bool

    init(name: String, phone: String, isFavorite: Bool = false) {
        self.id = UUID()
        self.name = name
        self.phone = phone
        self.isFavorite = isFavorite
    }
}

// MARK: - Custom Cell
class ContactCell: UITableViewCell {
    static let reuseIdentifier = "ContactCell"

    private let nameLabel = UILabel()
    private let phoneLabel = UILabel()
    private let favoriteIcon = UIImageView()

    override init(style: UITableViewCell.CellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
        setupUI()
    }

    required init?(coder: NSCoder) { fatalError() }

    private func setupUI() {
        nameLabel.font = .systemFont(ofSize: 16, weight: .medium)
        phoneLabel.font = .systemFont(ofSize: 14)
        phoneLabel.textColor = .secondaryLabel
        favoriteIcon.image = UIImage(systemName: "star.fill")
        favoriteIcon.tintColor = .systemYellow
        favoriteIcon.contentMode = .scaleAspectFit

        let textStack = UIStackView(arrangedSubviews: [nameLabel, phoneLabel])
        textStack.axis = .vertical
        textStack.spacing = 3

        let mainStack = UIStackView(arrangedSubviews: [textStack, favoriteIcon])
        mainStack.axis = .horizontal
        mainStack.spacing = 8
        mainStack.translatesAutoresizingMaskIntoConstraints = false
        contentView.addSubview(mainStack)

        NSLayoutConstraint.activate([
            mainStack.topAnchor.constraint(equalTo: contentView.topAnchor, constant: 10),
            mainStack.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 16),
            mainStack.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: -16),
            mainStack.bottomAnchor.constraint(equalTo: contentView.bottomAnchor, constant: -10),
            favoriteIcon.widthAnchor.constraint(equalToConstant: 20)
        ])
    }

    func configure(with contact: Contact) {
        nameLabel.text = contact.name
        phoneLabel.text = contact.phone
        favoriteIcon.isHidden = !contact.isFavorite
    }
}

// MARK: - Main ViewController
class ContactsViewController: UIViewController {

    private var contacts: [Contact] = [
        Contact(name: "สมชาย ใจดี", phone: "081-234-5678", isFavorite: true),
        Contact(name: "สมหญิง สวยงาม", phone: "082-345-6789"),
        Contact(name: "ประเสริฐ มั่งมี", phone: "083-456-7890", isFavorite: true),
        Contact(name: "วิไล รักดี", phone: "084-567-8901"),
        Contact(name: "มานะ ขยันทำ", phone: "085-678-9012")
    ]

    private lazy var tableView: UITableView = {
        let tv = UITableView(frame: .zero, style: .insetGrouped)
        tv.translatesAutoresizingMaskIntoConstraints = false
        tv.dataSource = self
        tv.delegate = self
        // ลงทะเบียน cell class เพื่อใช้ reuse
        tv.register(ContactCell.self, forCellReuseIdentifier: ContactCell.reuseIdentifier)
        // Self-sizing cells: ให้ Auto Layout คำนวณความสูงเอง
        tv.rowHeight = UITableView.automaticDimension
        tv.estimatedRowHeight = 60
        return tv
    }()

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "ผู้ติดต่อ"
        navigationItem.rightBarButtonItem = UIBarButtonItem(
            barButtonSystemItem: .add,
            target: self,
            action: #selector(addContact)
        )
        setupTableView()
    }

    private func setupTableView() {
        view.addSubview(tableView)
        NSLayoutConstraint.activate([
            tableView.topAnchor.constraint(equalTo: view.topAnchor),
            tableView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            tableView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            tableView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
        ])
    }

    @objc private func addContact() {
        let newContact = Contact(name: "ผู้ติดต่อใหม่ \(contacts.count + 1)", phone: "000-000-0000")
        contacts.append(newContact)
        let indexPath = IndexPath(row: contacts.count - 1, section: 0)
        tableView.insertRows(at: [indexPath], with: .automatic)
    }
}

// MARK: - UITableViewDataSource
extension ContactsViewController: UITableViewDataSource {

    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return contacts.count
    }

    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        // dequeueReusableCell — นำ cell กลับมาใช้ซ้ำเพื่อประหยัด memory
        guard let cell = tableView.dequeueReusableCell(
            withIdentifier: ContactCell.reuseIdentifier,
            for: indexPath
        ) as? ContactCell else {
            return UITableViewCell()
        }
        cell.configure(with: contacts[indexPath.row])
        return cell
    }

    // รองรับการลบด้วย swipe (built-in)
    func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCell.EditingStyle, forRowAt indexPath: IndexPath) {
        if editingStyle == .delete {
            contacts.remove(at: indexPath.row)
            tableView.deleteRows(at: [indexPath], with: .fade)
        }
    }
}

// MARK: - UITableViewDelegate
extension ContactsViewController: UITableViewDelegate {

    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        tableView.deselectRow(at: indexPath, animated: true)
        let contact = contacts[indexPath.row]
        print("เลือก: \(contact.name)")
    }

    func tableView(_ tableView: UITableView, titleForHeaderInSection section: Int) -> String? {
        return "รายชื่อทั้งหมด (\(contacts.count) คน)"
    }

    // Leading swipe action (ด้านซ้าย) — toggle favorite
    func tableView(_ tableView: UITableView, leadingSwipeActionsConfigurationForRowAt indexPath: IndexPath) -> UISwipeActionsConfiguration? {
        let isFav = contacts[indexPath.row].isFavorite
        let action = UIContextualAction(style: .normal, title: isFav ? "Unfav" : "Fav") { [weak self] _, _, completion in
            self?.contacts[indexPath.row].isFavorite.toggle()
            tableView.reloadRows(at: [indexPath], with: .automatic)
            completion(true)
        }
        action.backgroundColor = .systemYellow
        action.image = UIImage(systemName: isFav ? "star.slash" : "star.fill")
        return UISwipeActionsConfiguration(actions: [action])
    }

    // Trailing swipe action (ด้านขวา) — delete
    func tableView(_ tableView: UITableView, trailingSwipeActionsConfigurationForRowAt indexPath: IndexPath) -> UISwipeActionsConfiguration? {
        let deleteAction = UIContextualAction(style: .destructive, title: "ลบ") { [weak self] _, _, completion in
            self?.contacts.remove(at: indexPath.row)
            tableView.deleteRows(at: [indexPath], with: .automatic)
            completion(true)
        }
        deleteAction.image = UIImage(systemName: "trash")
        return UISwipeActionsConfiguration(actions: [deleteAction])
    }
}
```

---

## 5. UICollectionView

UICollectionView ใช้แสดงข้อมูลในรูปแบบ grid หรือ custom layout ที่ซับซ้อนกว่า UITableView

### ตัวอย่างสมบูรณ์: Photo Grid Gallery

```swift
import UIKit

// MARK: - Data Model
struct PhotoItem: Hashable {
    let id = UUID()
    let imageName: String
    let title: String
}

// MARK: - Gallery View Controller
class GalleryViewController: UIViewController {

    // MARK: - Data
    private var photos: [PhotoItem] = (1...12).map {
        PhotoItem(imageName: "photo.\($0)", title: "รูปที่ \($0)")
    }

    // DiffableDataSource ใช้ Hashable snapshots แทน reloadData()
    private var dataSource: UICollectionViewDiffableDataSource<Section, PhotoItem>!

    enum Section { case main }

    // MARK: - CollectionView Setup
    private lazy var collectionView: UICollectionView = {
        let layout = createCompositionalLayout()
        let cv = UICollectionView(frame: .zero, collectionViewLayout: layout)
        cv.translatesAutoresizingMaskIntoConstraints = false
        cv.backgroundColor = .systemBackground
        cv.delegate = self
        return cv
    }()

    // MARK: - Compositional Layout
    private func createCompositionalLayout() -> UICollectionViewCompositionalLayout {
        // Item — แต่ละรูปใช้ 1 ใน 3 ของความกว้าง
        let itemSize = NSCollectionLayoutSize(
            widthDimension: .fractionalWidth(1.0/3.0),
            heightDimension: .fractionalHeight(1.0)
        )
        let item = NSCollectionLayoutItem(layoutSize: itemSize)
        item.contentInsets = NSDirectionalEdgeInsets(top: 2, leading: 2, bottom: 2, trailing: 2)

        // Group — แถวละ 3 ชิ้น, สูง = กว้าง (square)
        let groupSize = NSCollectionLayoutSize(
            widthDimension: .fractionalWidth(1.0),
            heightDimension: .fractionalWidth(1.0/3.0)
        )
        let group = NSCollectionLayoutGroup.horizontal(layoutSize: groupSize, subitems: [item])

        // Section
        let section = NSCollectionLayoutSection(group: group)
        section.contentInsets = NSDirectionalEdgeInsets(top: 4, leading: 4, bottom: 4, trailing: 4)

        return UICollectionViewCompositionalLayout(section: section)
    }

    // MARK: - DiffableDataSource
    private func configureDataSource() {
        // ลงทะเบียน cell ด้วย CellRegistration (iOS 14+)
        let cellRegistration = UICollectionView.CellRegistration<UICollectionViewCell, PhotoItem> { cell, indexPath, item in
            var config = UIBackgroundConfiguration.listPlainCell()
            config.backgroundColor = .systemGray5
            cell.backgroundConfiguration = config

            // สร้าง ImageView ภายใน cell
            if cell.contentView.subviews.isEmpty {
                let imageView = UIImageView()
                imageView.contentMode = .scaleAspectFill
                imageView.clipsToBounds = true
                imageView.tag = 100
                imageView.translatesAutoresizingMaskIntoConstraints = false
                cell.contentView.addSubview(imageView)
                NSLayoutConstraint.activate([
                    imageView.topAnchor.constraint(equalTo: cell.contentView.topAnchor),
                    imageView.leadingAnchor.constraint(equalTo: cell.contentView.leadingAnchor),
                    imageView.trailingAnchor.constraint(equalTo: cell.contentView.trailingAnchor),
                    imageView.bottomAnchor.constraint(equalTo: cell.contentView.bottomAnchor)
                ])
            }

            if let imageView = cell.contentView.viewWithTag(100) as? UIImageView {
                // ใช้ SF Symbols เป็นตัวอย่าง
                let systemNames = ["sun.max", "moon", "star", "cloud", "bolt", "flame",
                                   "drop", "leaf", "globe", "heart", "bell", "bookmark"]
                let idx = indexPath.item % systemNames.count
                imageView.image = UIImage(systemName: systemNames[idx])
                imageView.tintColor = .systemBlue
                imageView.backgroundColor = .systemGray6
            }
        }

        dataSource = UICollectionViewDiffableDataSource<Section, PhotoItem>(
            collectionView: collectionView
        ) { collectionView, indexPath, item in
            collectionView.dequeueConfiguredReusableCell(using: cellRegistration, for: indexPath, item: item)
        }
    }

    // MARK: - Apply Snapshot
    private func applySnapshot(animatingDifferences: Bool = true) {
        var snapshot = NSDiffableDataSourceSnapshot<Section, PhotoItem>()
        snapshot.appendSections([.main])
        snapshot.appendItems(photos)
        dataSource.apply(snapshot, animatingDifferences: animatingDifferences)
    }

    // MARK: - Lifecycle
    override func viewDidLoad() {
        super.viewDidLoad()
        title = "แกลเลอรี"
        setupCollectionView()
        configureDataSource()
        applySnapshot(animatingDifferences: false)

        navigationItem.rightBarButtonItem = UIBarButtonItem(
            title: "สุ่มลบ",
            style: .plain,
            target: self,
            action: #selector(removeRandomPhoto)
        )
    }

    private func setupCollectionView() {
        view.addSubview(collectionView)
        NSLayoutConstraint.activate([
            collectionView.topAnchor.constraint(equalTo: view.topAnchor),
            collectionView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            collectionView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            collectionView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
        ])
    }

    // DiffableDataSource จัดการ animation การลบให้อัตโนมัติ
    @objc private func removeRandomPhoto() {
        guard !photos.isEmpty else { return }
        photos.remove(at: Int.random(in: 0..<photos.count))
        applySnapshot()
    }
}

// MARK: - UICollectionViewDelegate
extension GalleryViewController: UICollectionViewDelegate {
    func collectionView(_ collectionView: UICollectionView, didSelectItemAt indexPath: IndexPath) {
        guard let item = dataSource.itemIdentifier(for: indexPath) else { return }
        print("เลือก: \(item.title)")
    }
}
```

---

## 6. UINavigationController + UITabBarController

### Navigation Controller: Push/Pop

```swift
import UIKit

// MARK: - Tab Bar Setup
class AppTabBarController: UITabBarController {

    override func viewDidLoad() {
        super.viewDidLoad()
        setupTabs()
        customizeAppearance()
    }

    private func setupTabs() {
        let homeVC = makeNavController(
            rootVC: HomeViewController(),
            title: "หน้าหลัก",
            systemImage: "house.fill"
        )
        let searchVC = makeNavController(
            rootVC: SearchViewController(),
            title: "ค้นหา",
            systemImage: "magnifyingglass"
        )
        let profileVC = makeNavController(
            rootVC: ProfileViewController(),
            title: "โปรไฟล์",
            systemImage: "person.fill"
        )
        viewControllers = [homeVC, searchVC, profileVC]
    }

    private func makeNavController(rootVC: UIViewController, title: String, systemImage: String) -> UINavigationController {
        rootVC.title = title
        let navVC = UINavigationController(rootViewController: rootVC)
        navVC.tabBarItem = UITabBarItem(
            title: title,
            image: UIImage(systemName: systemImage),
            selectedImage: UIImage(systemName: systemImage)
        )
        return navVC
    }

    // MARK: - Appearance Customization
    private func customizeAppearance() {
        // Navigation Bar Appearance (iOS 15+)
        let navAppearance = UINavigationBarAppearance()
        navAppearance.configureWithOpaqueBackground()
        navAppearance.backgroundColor = .systemBackground
        navAppearance.titleTextAttributes = [
            .font: UIFont.systemFont(ofSize: 18, weight: .semibold)
        ]
        navAppearance.largeTitleTextAttributes = [
            .font: UIFont.systemFont(ofSize: 28, weight: .bold)
        ]

        UINavigationBar.appearance().standardAppearance = navAppearance
        UINavigationBar.appearance().scrollEdgeAppearance = navAppearance
        UINavigationBar.appearance().compactAppearance = navAppearance

        // Tab Bar Appearance
        let tabAppearance = UITabBarAppearance()
        tabAppearance.configureWithOpaqueBackground()
        tabAppearance.backgroundColor = .systemBackground
        UITabBar.appearance().standardAppearance = tabAppearance
        UITabBar.appearance().scrollEdgeAppearance = tabAppearance
        UITabBar.appearance().tintColor = .systemBlue
    }
}

// MARK: - Home with Push Navigation
class HomeViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground
        navigationItem.largeTitleDisplayMode = .always
        navigationController?.navigationBar.prefersLargeTitles = true

        let button = UIButton(type: .system)
        button.setTitle("ไปหน้าถัดไป", for: .normal)
        button.titleLabel?.font = .systemFont(ofSize: 18)
        button.translatesAutoresizingMaskIntoConstraints = false
        button.addTarget(self, action: #selector(pushNext), for: .touchUpInside)
        view.addSubview(button)
        NSLayoutConstraint.activate([
            button.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            button.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
    }

    @objc private func pushNext() {
        let detailVC = DetailViewController()
        // push เพิ่ม ViewController เข้า navigation stack
        navigationController?.pushViewController(detailVC, animated: true)
    }
}

class DetailViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemGroupedBackground
        title = "รายละเอียด"

        // Back button แสดงอัตโนมัติ; ถ้าต้องการ pop ด้วยโค้ด:
        navigationItem.leftBarButtonItem = UIBarButtonItem(
            title: "กลับ",
            style: .plain,
            target: self,
            action: #selector(goBack)
        )
    }

    @objc private func goBack() {
        // pop ออกจาก navigation stack
        navigationController?.popViewController(animated: true)
    }
}

class SearchViewController: UIViewController {
    override func viewDidLoad() { super.viewDidLoad(); view.backgroundColor = .systemBackground }
}
```

---

## 7. Common UIKit Controls

### ตัวอย่างสมบูรณ์: Controls Showcase

```swift
import UIKit

class ControlsViewController: UIViewController {

    // MARK: - Controls
    private let modernButton = UIButton(configuration: .filled())
    private let textField = UITextField()
    private let imageView = UIImageView()
    private let statusLabel = UILabel()

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemGroupedBackground
        title = "Controls"
        setupControls()
    }

    private func setupControls() {
        let stack = UIStackView()
        stack.axis = .vertical
        stack.spacing = 16
        stack.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(stack)

        NSLayoutConstraint.activate([
            stack.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 20),
            stack.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 20),
            stack.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -20)
        ])

        // 1. UIButton — Modern UIButtonConfiguration API (iOS 15+)
        var config = UIButton.Configuration.filled()
        config.title = "กดฉันสิ!"
        config.subtitle = "Modern Button API"
        config.image = UIImage(systemName: "hand.tap")
        config.imagePadding = 8
        config.cornerStyle = .medium
        config.baseBackgroundColor = .systemIndigo
        modernButton.configuration = config
        modernButton.addTarget(self, action: #selector(buttonTapped), for: .touchUpInside)
        stack.addArrangedSubview(modernButton)

        // 2. UITextField with delegate
        textField.placeholder = "พิมพ์ข้อความที่นี่..."
        textField.borderStyle = .roundedRect
        textField.clearButtonMode = .whileEditing
        textField.returnKeyType = .done
        textField.autocorrectionType = .no
        textField.delegate = self
        stack.addArrangedSubview(textField)

        // 3. UIImageView
        imageView.image = UIImage(systemName: "photo.artframe")
        imageView.tintColor = .systemGray3
        imageView.contentMode = .scaleAspectFit
        imageView.heightAnchor.constraint(equalToConstant: 80).isActive = true
        stack.addArrangedSubview(imageView)

        // 4. UILabel
        statusLabel.text = "รอรับ input..."
        statusLabel.font = .systemFont(ofSize: 15)
        statusLabel.textColor = .secondaryLabel
        statusLabel.numberOfLines = 0
        statusLabel.textAlignment = .center
        stack.addArrangedSubview(statusLabel)

        // 5. Alert button
        let alertButton = UIButton(configuration: .borderedTinted())
        var alertConfig = UIButton.Configuration.borderedTinted()
        alertConfig.title = "แสดง Alert"
        alertConfig.baseBackgroundColor = .systemOrange
        alertButton.configuration = alertConfig
        alertButton.addTarget(self, action: #selector(showAlert), for: .touchUpInside)
        stack.addArrangedSubview(alertButton)

        let sheetButton = UIButton(configuration: .borderedTinted())
        var sheetConfig = UIButton.Configuration.borderedTinted()
        sheetConfig.title = "แสดง Action Sheet"
        sheetConfig.baseBackgroundColor = .systemTeal
        sheetButton.configuration = sheetConfig
        sheetButton.addTarget(self, action: #selector(showActionSheet), for: .touchUpInside)
        stack.addArrangedSubview(sheetButton)
    }

    @objc private func buttonTapped() {
        statusLabel.text = "ปุ่มถูกกด!"
        statusLabel.textColor = .systemGreen
    }

    // MARK: - UIAlertController: Alert
    @objc private func showAlert() {
        let alert = UIAlertController(
            title: "ยืนยัน",
            message: "คุณต้องการดำเนินการต่อหรือไม่?",
            preferredStyle: .alert
        )
        alert.addAction(UIAlertAction(title: "ยกเลิก", style: .cancel))
        alert.addAction(UIAlertAction(title: "ตกลง", style: .default) { _ in
            self.statusLabel.text = "เลือก: ตกลง"
            self.statusLabel.textColor = .systemBlue
        })
        present(alert, animated: true)
    }

    // MARK: - UIAlertController: Action Sheet
    @objc private func showActionSheet() {
        let sheet = UIAlertController(title: "เลือกตัวเลือก", message: nil, preferredStyle: .actionSheet)
        sheet.addAction(UIAlertAction(title: "ถ่ายรูป", style: .default) { _ in
            self.statusLabel.text = "เลือก: ถ่ายรูป"
        })
        sheet.addAction(UIAlertAction(title: "เลือกจากคลัง", style: .default) { _ in
            self.statusLabel.text = "เลือก: คลังรูป"
        })
        sheet.addAction(UIAlertAction(title: "ลบรูป", style: .destructive) { _ in
            self.statusLabel.text = "เลือก: ลบรูป"
        })
        sheet.addAction(UIAlertAction(title: "ยกเลิก", style: .cancel))

        // iPad ต้องกำหนด sourceView
        if let popover = sheet.popoverPresentationController {
            popover.sourceView = view
            popover.sourceRect = CGRect(x: view.bounds.midX, y: view.bounds.midY, width: 0, height: 0)
        }
        present(sheet, animated: true)
    }
}

// MARK: - UITextFieldDelegate
extension ControlsViewController: UITextFieldDelegate {
    func textFieldShouldReturn(_ textField: UITextField) -> Bool {
        textField.resignFirstResponder() // ซ่อน keyboard
        return true
    }

    func textFieldDidChangeSelection(_ textField: UITextField) {
        statusLabel.text = "พิมพ์: \(textField.text ?? "")"
        statusLabel.textColor = .label
    }
}
```

---

## 8. Gestures

### ตัวอย่างสมบูรณ์: Drag-to-Move Card

```swift
import UIKit

class GestureViewController: UIViewController {

    private let cardView = UIView()
    private var cardCenter: CGPoint = .zero

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground
        title = "Gesture Demo"
        setupCard()
        setupGestures()
    }

    private func setupCard() {
        cardView.frame = CGRect(x: 0, y: 0, width: 200, height: 120)
        cardView.center = view.center
        cardView.backgroundColor = .systemBlue
        cardView.layer.cornerRadius = 16
        cardView.layer.shadowColor = UIColor.black.cgColor
        cardView.layer.shadowOpacity = 0.3
        cardView.layer.shadowRadius = 8
        cardView.layer.shadowOffset = CGSize(width: 0, height: 4)

        let label = UILabel()
        label.text = "ลากฉันได้เลย\n(ใช้ 2 นิ้ว Pinch ย่อ-ขยาย)"
        label.textColor = .white
        label.font = .systemFont(ofSize: 14, weight: .medium)
        label.numberOfLines = 2
        label.textAlignment = .center
        label.translatesAutoresizingMaskIntoConstraints = false
        cardView.addSubview(label)
        NSLayoutConstraint.activate([
            label.centerXAnchor.constraint(equalTo: cardView.centerXAnchor),
            label.centerYAnchor.constraint(equalTo: cardView.centerYAnchor),
            label.leadingAnchor.constraint(equalTo: cardView.leadingAnchor, constant: 8),
            label.trailingAnchor.constraint(equalTo: cardView.trailingAnchor, constant: -8)
        ])
        view.addSubview(cardView)
    }

    private func setupGestures() {
        // 1. Pan Gesture — ลาก card ไปมา
        let panGesture = UIPanGestureRecognizer(target: self, action: #selector(handlePan(_:)))
        cardView.addGestureRecognizer(panGesture)

        // 2. Tap Gesture — reset ตำแหน่ง
        let tapGesture = UITapGestureRecognizer(target: self, action: #selector(handleTap(_:)))
        tapGesture.numberOfTapsRequired = 2 // double tap
        cardView.addGestureRecognizer(tapGesture)

        // 3. Pinch Gesture — ย่อ-ขยาย
        let pinchGesture = UIPinchGestureRecognizer(target: self, action: #selector(handlePinch(_:)))
        cardView.addGestureRecognizer(pinchGesture)

        cardView.isUserInteractionEnabled = true
    }

    // MARK: - Pan Handler
    @objc private func handlePan(_ gesture: UIPanGestureRecognizer) {
        let translation = gesture.translation(in: view)

        switch gesture.state {
        case .began:
            cardCenter = cardView.center
            // ขยายเล็กน้อยเมื่อเริ่มลาก
            UIView.animate(withDuration: 0.1) {
                self.cardView.transform = self.cardView.transform.scaledBy(x: 1.05, y: 1.05)
            }
        case .changed:
            cardView.center = CGPoint(
                x: cardCenter.x + translation.x,
                y: cardCenter.y + translation.y
            )
        case .ended, .cancelled:
            // Spring animation เมื่อปล่อย
            let velocity = gesture.velocity(in: view)
            UIView.animate(
                withDuration: 0.5,
                delay: 0,
                usingSpringWithDamping: 0.7,
                initialSpringVelocity: 0.5
            ) {
                self.cardView.transform = .identity
                // กระเด้งกลับถ้าออกนอกขอบ
                let bounds = self.view.bounds.insetBy(dx: 60, dy: 80)
                var center = self.cardView.center
                center.x = min(max(center.x, bounds.minX), bounds.maxX)
                center.y = min(max(center.y, bounds.minY), bounds.maxY)
                self.cardView.center = center
            }
            _ = velocity // ใช้ velocity เพื่อ momentum ถ้าต้องการ
        default:
            break
        }
    }

    // MARK: - Tap Handler
    @objc private func handleTap(_ gesture: UITapGestureRecognizer) {
        // Double tap: reset ตำแหน่งกลาง
        UIView.animate(withDuration: 0.4, delay: 0, usingSpringWithDamping: 0.6, initialSpringVelocity: 0.8) {
            self.cardView.center = self.view.center
            self.cardView.transform = .identity
        }
    }

    // MARK: - Pinch Handler
    @objc private func handlePinch(_ gesture: UIPinchGestureRecognizer) {
        switch gesture.state {
        case .changed:
            cardView.transform = cardView.transform.scaledBy(x: gesture.scale, y: gesture.scale)
            gesture.scale = 1.0 // reset เพื่อให้ scale เป็น incremental
        case .ended:
            // ควบคุมขนาดไม่ให้เล็กหรือใหญ่เกินไป
            let currentScale = cardView.transform.a
            let clampedScale = min(max(currentScale, 0.5), 2.0)
            UIView.animate(withDuration: 0.2) {
                self.cardView.transform = CGAffineTransform(scaleX: clampedScale, y: clampedScale)
            }
        default:
            break
        }
    }
}
```

---

## 9. UIView Animations

### ตัวอย่างสมบูรณ์: Card Flip Animation

```swift
import UIKit

class CardFlipViewController: UIViewController {

    // MARK: - Properties
    private let cardContainer = UIView()
    private let frontCard = UIView()
    private let backCard = UIView()
    private var isShowingFront = true
    private var animator: UIViewPropertyAnimator?

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemGroupedBackground
        title = "Card Flip"
        setupCard()
        setupAnimationButtons()
    }

    // MARK: - Card Setup
    private func setupCard() {
        cardContainer.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(cardContainer)
        NSLayoutConstraint.activate([
            cardContainer.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            cardContainer.centerYAnchor.constraint(equalTo: view.centerYAnchor, constant: -60),
            cardContainer.widthAnchor.constraint(equalToConstant: 240),
            cardContainer.heightAnchor.constraint(equalToConstant: 160)
        ])

        // Front Card
        frontCard.frame = cardContainer.bounds
        frontCard.autoresizingMask = [.flexibleWidth, .flexibleHeight]
        frontCard.backgroundColor = .systemBlue
        frontCard.layer.cornerRadius = 20
        frontCard.layer.shadowColor = UIColor.black.cgColor
        frontCard.layer.shadowOpacity = 0.25
        frontCard.layer.shadowRadius = 10

        let frontLabel = UILabel()
        frontLabel.text = "🃏 หน้า"
        frontLabel.font = .systemFont(ofSize: 28, weight: .bold)
        frontLabel.textColor = .white
        frontLabel.textAlignment = .center
        frontLabel.translatesAutoresizingMaskIntoConstraints = false
        frontCard.addSubview(frontLabel)
        NSLayoutConstraint.activate([
            frontLabel.centerXAnchor.constraint(equalTo: frontCard.centerXAnchor),
            frontLabel.centerYAnchor.constraint(equalTo: frontCard.centerYAnchor)
        ])

        // Back Card
        backCard.frame = cardContainer.bounds
        backCard.autoresizingMask = [.flexibleWidth, .flexibleHeight]
        backCard.backgroundColor = .systemOrange
        backCard.layer.cornerRadius = 20
        backCard.layer.shadowColor = UIColor.black.cgColor
        backCard.layer.shadowOpacity = 0.25
        backCard.layer.shadowRadius = 10
        backCard.alpha = 0
        backCard.transform = CGAffineTransform(scaleX: -1, y: 1) // เริ่มพลิกหน้า

        let backLabel = UILabel()
        backLabel.text = "🎴 หลัง"
        backLabel.font = .systemFont(ofSize: 28, weight: .bold)
        backLabel.textColor = .white
        backLabel.textAlignment = .center
        backLabel.translatesAutoresizingMaskIntoConstraints = false
        backCard.addSubview(backLabel)
        NSLayoutConstraint.activate([
            backLabel.centerXAnchor.constraint(equalTo: backCard.centerXAnchor),
            backLabel.centerYAnchor.constraint(equalTo: backCard.centerYAnchor)
        ])

        cardContainer.addSubview(backCard)
        cardContainer.addSubview(frontCard)

        // Tap to flip
        let tap = UITapGestureRecognizer(target: self, action: #selector(flipCard))
        cardContainer.addGestureRecognizer(cardContainer.gestureRecognizers?.first ?? tap)
        cardContainer.addGestureRecognizer(tap)
        cardContainer.isUserInteractionEnabled = true
    }

    private func setupAnimationButtons() {
        let stack = UIStackView()
        stack.axis = .vertical
        stack.spacing = 12
        stack.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(stack)

        NSLayoutConstraint.activate([
            stack.topAnchor.constraint(equalTo: cardContainer.bottomAnchor, constant: 32),
            stack.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            stack.widthAnchor.constraint(equalToConstant: 200)
        ])

        let configs: [(String, Selector, UIColor)] = [
            ("พลิกการ์ด", #selector(flipCard), .systemBlue),
            ("Spring Bounce", #selector(springBounce), .systemGreen),
            ("Property Animator", #selector(propertyAnimator), .systemPurple)
        ]

        for (title, action, color) in configs {
            var config = UIButton.Configuration.filled()
            config.title = title
            config.baseBackgroundColor = color
            config.cornerStyle = .medium
            let btn = UIButton(configuration: config)
            btn.addTarget(self, action: action, for: .touchUpInside)
            stack.addArrangedSubview(btn)
        }
    }

    // MARK: - Flip Animation
    @objc private func flipCard() {
        let fromView = isShowingFront ? frontCard : backCard
        let toView = isShowingFront ? backCard : frontCard

        UIView.transition(
            from: fromView,
            to: toView,
            duration: 0.5,
            options: [.transitionFlipFromRight, .showHideTransitionViews]
        )
        isShowingFront.toggle()
    }

    // MARK: - Spring Animation
    @objc private func springBounce() {
        // UIView.animate spring animation
        UIView.animate(
            withDuration: 0.6,
            delay: 0,
            usingSpringWithDamping: 0.3,        // ค่าน้อย = กระเด้งมาก
            initialSpringVelocity: 0.8,
            options: []
        ) {
            self.cardContainer.transform = CGAffineTransform(scaleX: 1.2, y: 1.2)
        } completion: { _ in
            UIView.animate(withDuration: 0.3) {
                self.cardContainer.transform = .identity
            }
        }
    }

    // MARK: - UIViewPropertyAnimator (Interactive / Interruptible)
    @objc private func propertyAnimator() {
        // หยุด animator เดิมถ้ายังทำงานอยู่
        animator?.stopAnimation(true)

        animator = UIViewPropertyAnimator(duration: 0.8, dampingRatio: 0.5) {
            self.cardContainer.transform = CGAffineTransform(rotationAngle: .pi)
        }

        animator?.addCompletion { _ in
            UIView.animate(withDuration: 0.4) {
                self.cardContainer.transform = .identity
            }
        }

        animator?.startAnimation()
    }
}
```

---

## 10. UIKit + SwiftUI Bridge

### UIViewRepresentable: ใช้ UIKit View ใน SwiftUI

```swift
import SwiftUI
import UIKit

// MARK: - Wrap UITextView ใน SwiftUI
struct RichTextEditor: UIViewRepresentable {

    @Binding var text: String
    var placeholder: String = "เริ่มพิมพ์..."

    // Coordinator จัดการ delegate callbacks
    class Coordinator: NSObject, UITextViewDelegate {
        var parent: RichTextEditor

        init(_ parent: RichTextEditor) {
            self.parent = parent
        }

        func textViewDidChange(_ textView: UITextView) {
            parent.text = textView.text
        }

        func textViewDidBeginEditing(_ textView: UITextView) {
            if textView.textColor == UIColor.placeholderText {
                textView.text = ""
                textView.textColor = .label
            }
        }

        func textViewDidEndEditing(_ textView: UITextView) {
            if textView.text.isEmpty {
                textView.text = parent.placeholder
                textView.textColor = .placeholderText
            }
        }
    }

    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }

    func makeUIView(context: Context) -> UITextView {
        let textView = UITextView()
        textView.delegate = context.coordinator
        textView.font = .systemFont(ofSize: 16)
        textView.backgroundColor = .systemBackground
        textView.layer.borderColor = UIColor.separator.cgColor
        textView.layer.borderWidth = 1
        textView.layer.cornerRadius = 8
        textView.textContainerInset = UIEdgeInsets(top: 8, left: 8, bottom: 8, right: 8)

        if text.isEmpty {
            textView.text = placeholder
            textView.textColor = .placeholderText
        } else {
            textView.text = text
            textView.textColor = .label
        }
        return textView
    }

    func updateUIView(_ uiView: UITextView, context: Context) {
        if uiView.textColor != .placeholderText {
            uiView.text = text
        }
    }
}

// SwiftUI View ที่ใช้ RichTextEditor
struct NoteEditorView: View {
    @State private var noteText = ""

    var body: some View {
        NavigationView {
            VStack(alignment: .leading, spacing: 12) {
                Text("จำนวนตัวอักษร: \(noteText.count)")
                    .font(.caption)
                    .foregroundColor(.secondary)

                RichTextEditor(text: $noteText, placeholder: "เขียนโน้ตของคุณที่นี่...")
                    .frame(minHeight: 200)
            }
            .padding()
            .navigationTitle("บันทึกของฉัน")
        }
    }
}

// MARK: - UIHostingController: ฝัง SwiftUI ใน UIKit
class UIKitHostViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "UIKit + SwiftUI"
        view.backgroundColor = .systemBackground
        embedSwiftUIView()
    }

    private func embedSwiftUIView() {
        // สร้าง SwiftUI view
        let swiftUIView = NoteEditorView()
        let hostingController = UIHostingController(rootView: swiftUIView)

        // เพิ่มเป็น child view controller
        addChild(hostingController)
        hostingController.view.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(hostingController.view)
        hostingController.didMove(toParent: self)

        NSLayoutConstraint.activate([
            hostingController.view.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            hostingController.view.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            hostingController.view.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            hostingController.view.bottomAnchor.constraint(equalTo: view.bottomAnchor)
        ])
    }
}
```

---

## 11. แบบฝึกหัด (Practical Exercises)

### Exercise 1: Notes App ด้วย UITableView

สร้างแอป Notes พร้อมฟีเจอร์ เพิ่ม/แก้ไข/ลบ note

```swift
import UIKit

// MARK: - Model
struct Note: Identifiable {
    let id = UUID()
    var title: String
    var body: String
    var date: Date = Date()
}

// MARK: - Notes List ViewController
class NotesListViewController: UIViewController {

    private var notes: [Note] = [
        Note(title: "Shopping List", body: "นม, ไข่, ขนมปัง"),
        Note(title: "Meeting Notes", body: "ประชุม 10am ห้อง A"),
        Note(title: "Ideas", body: "สร้างแอป productivity ใหม่")
    ]

    private lazy var tableView: UITableView = {
        let tv = UITableView(frame: .zero, style: .insetGrouped)
        tv.translatesAutoresizingMaskIntoConstraints = false
        tv.dataSource = self
        tv.delegate = self
        tv.register(UITableViewCell.self, forCellReuseIdentifier: "NoteCell")
        tv.rowHeight = UITableView.automaticDimension
        tv.estimatedRowHeight = 60
        return tv
    }()

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "โน้ต"
        navigationItem.rightBarButtonItem = UIBarButtonItem(
            barButtonSystemItem: .compose,
            target: self,
            action: #selector(addNote)
        )
        navigationItem.leftBarButtonItem = editButtonItem
        view.addSubview(tableView)
        NSLayoutConstraint.activate([
            tableView.topAnchor.constraint(equalTo: view.topAnchor),
            tableView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            tableView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            tableView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
        ])
    }

    override func setEditing(_ editing: Bool, animated: Bool) {
        super.setEditing(editing, animated: animated)
        tableView.setEditing(editing, animated: animated)
    }

    @objc private func addNote() {
        showNoteEditor(note: nil, at: nil)
    }

    private func showNoteEditor(note: Note?, at indexPath: IndexPath?) {
        let editorVC = NoteEditorViewController()
        editorVC.note = note
        editorVC.onSave = { [weak self] savedNote in
            guard let self = self else { return }
            if let indexPath = indexPath {
                self.notes[indexPath.row] = savedNote
                self.tableView.reloadRows(at: [indexPath], with: .automatic)
            } else {
                self.notes.insert(savedNote, at: 0)
                self.tableView.insertRows(at: [IndexPath(row: 0, section: 0)], with: .automatic)
            }
        }
        navigationController?.pushViewController(editorVC, animated: true)
    }
}

extension NotesListViewController: UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int { notes.count }

    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "NoteCell", for: indexPath)
        let note = notes[indexPath.row]
        var config = cell.defaultContentConfiguration()
        config.text = note.title
        config.secondaryText = note.body
        config.secondaryTextProperties.numberOfLines = 2
        cell.contentConfiguration = config
        cell.accessoryType = .disclosureIndicator
        return cell
    }

    func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCell.EditingStyle, forRowAt indexPath: IndexPath) {
        if editingStyle == .delete {
            notes.remove(at: indexPath.row)
            tableView.deleteRows(at: [indexPath], with: .fade)
        }
    }

    func tableView(_ tableView: UITableView, moveRowAt fromIndexPath: IndexPath, to: IndexPath) {
        let note = notes.remove(at: fromIndexPath.row)
        notes.insert(note, at: to.row)
    }

    func tableView(_ tableView: UITableView, canMoveRowAt indexPath: IndexPath) -> Bool { true }
}

extension NotesListViewController: UITableViewDelegate {
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        tableView.deselectRow(at: indexPath, animated: true)
        showNoteEditor(note: notes[indexPath.row], at: indexPath)
    }
}

// MARK: - Note Editor ViewController
class NoteEditorViewController: UIViewController {

    var note: Note?
    var onSave: ((Note) -> Void)?

    private let titleField = UITextField()
    private let bodyTextView = UITextView()

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground
        title = note == nil ? "โน้ตใหม่" : "แก้ไขโน้ต"
        navigationItem.rightBarButtonItem = UIBarButtonItem(
            barButtonSystemItem: .save,
            target: self,
            action: #selector(saveNote)
        )
        setupUI()
        loadNote()
    }

    private func setupUI() {
        titleField.placeholder = "หัวข้อ"
        titleField.font = .systemFont(ofSize: 20, weight: .semibold)
        titleField.borderStyle = .none
        titleField.translatesAutoresizingMaskIntoConstraints = false

        let separator = UIView()
        separator.backgroundColor = .separator
        separator.translatesAutoresizingMaskIntoConstraints = false

        bodyTextView.font = .systemFont(ofSize: 16)
        bodyTextView.translatesAutoresizingMaskIntoConstraints = false

        view.addSubview(titleField)
        view.addSubview(separator)
        view.addSubview(bodyTextView)

        NSLayoutConstraint.activate([
            titleField.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 16),
            titleField.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 16),
            titleField.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -16),

            separator.topAnchor.constraint(equalTo: titleField.bottomAnchor, constant: 8),
            separator.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 16),
            separator.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -16),
            separator.heightAnchor.constraint(equalToConstant: 0.5),

            bodyTextView.topAnchor.constraint(equalTo: separator.bottomAnchor, constant: 8),
            bodyTextView.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 12),
            bodyTextView.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -12),
            bodyTextView.bottomAnchor.constraint(equalTo: view.safeAreaLayoutGuide.bottomAnchor)
        ])
    }

    private func loadNote() {
        titleField.text = note?.title
        bodyTextView.text = note?.body
    }

    @objc private func saveNote() {
        let title = titleField.text?.trimmingCharacters(in: .whitespaces) ?? ""
        let body = bodyTextView.text ?? ""
        guard !title.isEmpty else {
            let alert = UIAlertController(title: "กรุณาใส่หัวข้อ", message: nil, preferredStyle: .alert)
            alert.addAction(UIAlertAction(title: "ตกลง", style: .default))
            present(alert, animated: true)
            return
        }
        var saved = note ?? Note(title: "", body: "")
        saved.title = title
        saved.body = body
        onSave?(saved)
        navigationController?.popViewController(animated: true)
    }
}
```

---

### Exercise 2: Custom Segmented Picker ด้วย UIControl

สร้าง segmented control แบบ custom ที่ support tap + ส่ง UIControl events

```swift
import UIKit

// MARK: - Custom Segmented Control
class CustomSegmentedControl: UIControl {

    // MARK: - Properties
    private(set) var selectedIndex: Int = 0 {
        didSet { updateSelection(animated: true) }
    }
    private var segments: [String] = []
    private var buttons: [UIButton] = []
    private let indicator = UIView()

    // Appearance
    var selectedColor: UIColor = .systemBlue { didSet { updateAppearance() } }
    var normalColor: UIColor = .secondaryLabel { didSet { updateAppearance() } }
    var indicatorColor: UIColor = .systemBlue { didSet { updateAppearance() } }

    // MARK: - Init
    init(segments: [String]) {
        self.segments = segments
        super.init(frame: .zero)
        setupUI()
    }

    required init?(coder: NSCoder) { fatalError() }

    // MARK: - Setup
    private func setupUI() {
        backgroundColor = .secondarySystemBackground
        layer.cornerRadius = 12
        clipsToBounds = true

        // Sliding indicator
        indicator.backgroundColor = indicatorColor.withAlphaComponent(0.15)
        indicator.layer.cornerRadius = 10
        addSubview(indicator)

        // Buttons
        let stack = UIStackView()
        stack.axis = .horizontal
        stack.distribution = .fillEqually
        stack.spacing = 4
        stack.translatesAutoresizingMaskIntoConstraints = false
        addSubview(stack)

        NSLayoutConstraint.activate([
            stack.topAnchor.constraint(equalTo: topAnchor, constant: 4),
            stack.leadingAnchor.constraint(equalTo: leadingAnchor, constant: 4),
            stack.trailingAnchor.constraint(equalTo: trailingAnchor, constant: -4),
            stack.bottomAnchor.constraint(equalTo: bottomAnchor, constant: -4)
        ])

        for (i, title) in segments.enumerated() {
            let btn = UIButton(type: .system)
            btn.setTitle(title, for: .normal)
            btn.titleLabel?.font = .systemFont(ofSize: 14, weight: .medium)
            btn.tag = i
            btn.addTarget(self, action: #selector(segmentTapped(_:)), for: .touchUpInside)
            buttons.append(btn)
            stack.addArrangedSubview(btn)
        }
        updateAppearance()
    }

    override func layoutSubviews() {
        super.layoutSubviews()
        // คำนวณตำแหน่ง indicator เมื่อ layout พร้อม
        updateIndicatorPosition(animated: false)
    }

    // MARK: - Actions
    @objc private func segmentTapped(_ sender: UIButton) {
        guard sender.tag != selectedIndex else { return }
        selectedIndex = sender.tag
        // ส่ง UIControl event ให้ผู้ใช้ control subscribe ได้
        sendActions(for: .valueChanged)
    }

    // MARK: - Update
    private func updateSelection(animated: Bool) {
        updateIndicatorPosition(animated: animated)
        updateAppearance()
    }

    private func updateIndicatorPosition(animated: Bool) {
        guard !buttons.isEmpty else { return }
        let segmentWidth = bounds.width / CGFloat(segments.count)
        let indicatorX = segmentWidth * CGFloat(selectedIndex) + 4
        let indicatorWidth = segmentWidth - 8
        let targetFrame = CGRect(x: indicatorX, y: 4, width: indicatorWidth, height: bounds.height - 8)

        if animated {
            UIView.animate(withDuration: 0.25, delay: 0, usingSpringWithDamping: 0.8, initialSpringVelocity: 0.5) {
                self.indicator.frame = targetFrame
            }
        } else {
            indicator.frame = targetFrame
        }
    }

    private func updateAppearance() {
        for (i, btn) in buttons.enumerated() {
            let isSelected = i == selectedIndex
            btn.setTitleColor(isSelected ? selectedColor : normalColor, for: .normal)
            btn.titleLabel?.font = .systemFont(ofSize: 14, weight: isSelected ? .semibold : .medium)
        }
        indicator.backgroundColor = indicatorColor.withAlphaComponent(0.15)
    }

    // MARK: - Public API
    func setSelectedIndex(_ index: Int, animated: Bool = true) {
        guard index >= 0 && index < segments.count else { return }
        selectedIndex = index
    }
}

// MARK: - Demo ViewController
class SegmentedPickerDemoViewController: UIViewController {

    private let picker = CustomSegmentedControl(segments: ["วันนี้", "สัปดาห์", "เดือน", "ปี"])
    private let contentLabel = UILabel()

    private let contentMap = [
        "วันนี้": "แสดงข้อมูลของวันนี้",
        "สัปดาห์": "แสดงข้อมูล 7 วันที่ผ่านมา",
        "เดือน": "แสดงข้อมูล 30 วันที่ผ่านมา",
        "ปี": "แสดงข้อมูล 365 วันที่ผ่านมา"
    ]

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemGroupedBackground
        title = "Custom Segmented Picker"
        setupUI()
    }

    private func setupUI() {
        picker.translatesAutoresizingMaskIntoConstraints = false
        picker.addTarget(self, action: #selector(pickerChanged), for: .valueChanged)
        view.addSubview(picker)

        contentLabel.translatesAutoresizingMaskIntoConstraints = false
        contentLabel.font = .systemFont(ofSize: 18)
        contentLabel.textAlignment = .center
        contentLabel.numberOfLines = 0
        contentLabel.textColor = .secondaryLabel
        view.addSubview(contentLabel)

        NSLayoutConstraint.activate([
            picker.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 24),
            picker.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 20),
            picker.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -20),
            picker.heightAnchor.constraint(equalToConstant: 44),

            contentLabel.topAnchor.constraint(equalTo: picker.bottomAnchor, constant: 40),
            contentLabel.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 20),
            contentLabel.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -20)
        ])

        updateContent()
    }

    @objc private func pickerChanged() {
        updateContent()
    }

    private func updateContent() {
        let segments = ["วันนี้", "สัปดาห์", "เดือน", "ปี"]
        let selected = segments[picker.selectedIndex]
        contentLabel.text = contentMap[selected]

        UIView.transition(with: contentLabel, duration: 0.2, options: .transitionCrossDissolve) {}
    }
}
```

---

## สรุป: สิ่งที่ได้เรียนในบทนี้

| หัวข้อ | สิ่งที่ได้เรียน |
|-------|--------------|
| UIKit vs SwiftUI | เลือกใช้ตามความเหมาะสม, interop pattern |
| ViewController Lifecycle | วงจรชีวิตทุก method, เวลาที่ควรทำงานใน method ใด |
| Auto Layout | Anchor API, UIStackView, programmatic UI |
| UITableView | DataSource/Delegate, cell reuse, swipe actions |
| UICollectionView | CompositionalLayout, DiffableDataSource |
| Navigation/Tab | push/pop, tab setup, UINavigationBarAppearance |
| Controls | Modern button API, UITextField, UIAlertController |
| Gestures | Pan, Tap, Pinch — สร้าง interactive UI |
| Animations | UIView.animate, spring, UIViewPropertyAnimator |
| UIKit+SwiftUI | UIViewRepresentable, UIHostingController |
| Exercises | Notes app, Custom segmented control |

**ขั้นตอนต่อไป:** UICollectionView List (iOS 14+), UISheetPresentationController, XCTest UI Testing, Core Animation
