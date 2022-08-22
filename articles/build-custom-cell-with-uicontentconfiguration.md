---
title: "UIContentConfigurationでカスタムセルを実装する"
emoji: "🔨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ios", "swift", "uikit"]
published: true
---

iOS 14で導入された[`UIContentConfiguration`](https://developer.apple.com/documentation/uikit/uicontentconfiguration)を使ってカスタムセルを実装します。

まず、セルのコンテンツやスタイルを決定するオブジェクトを定義します。

```swift
struct ContentConfiguration: UIContentConfiguration {
  var image: UIImage?
  var text: String?

  func makeContentView() -> UIView & UIContentView {
    return ContentView(configuration: self)
  }

  func updated(for state: UIConfigurationState) -> ContentConfiguration {
    return self
  }
}
```

それらを反映するビューも定義します。

```swift
final class ContentView: UIView, UIContentView {
  let imageView: UIImageView = .init()
  let textLabel: UILabel = .init()

  var configuration: UIContentConfiguration {
    didSet {
      guard let configuration = configuration as? ContentConfiguration else { return }
      imageView.image = configuration.image
      textLabel.text = configuration.text
    }
  }

  init(configuration: ContentConfiguration) {
    self.configuration = configuration

    super.init(frame: .zero)
  }
}
```

セルにオブジェクトを設定して完了です。

```swift
cell.contentConfiguration = ContentConfiguration(image: UIImage(systemName: "globe"), text: item)
```

使用例です。

```swift
import UIKit

struct ContentConfiguration: UIContentConfiguration {
  var image: UIImage?
  var text: String?

  func makeContentView() -> UIView & UIContentView {
    return ContentView(configuration: self)
  }

  func updated(for state: UIConfigurationState) -> ContentConfiguration {
    return self
  }
}

final class ContentView: UIView, UIContentView {
  private let stackView: UIStackView = {
    let view = UIStackView()
    view.alignment = .center
    view.axis = .horizontal
    view.spacing = 16
    view.translatesAutoresizingMaskIntoConstraints = false
    return view
  }()

  let imageView: UIImageView = .init()

  let textLabel: UILabel = .init()

  var configuration: UIContentConfiguration {
    didSet {
      guard let configuration = configuration as? ContentConfiguration else { return }
      imageView.image = configuration.image
      textLabel.text = configuration.text
    }
  }

  init(configuration: ContentConfiguration) {
    self.configuration = configuration

    super.init(frame: .zero)

    addSubview(stackView)

    stackView.addArrangedSubview(imageView)
    stackView.addArrangedSubview(textLabel)

    NSLayoutConstraint.activate([
      stackView.leadingAnchor.constraint(equalTo: layoutMarginsGuide.leadingAnchor),
      stackView.trailingAnchor.constraint(equalTo: layoutMarginsGuide.trailingAnchor),
      stackView.topAnchor.constraint(equalTo: layoutMarginsGuide.topAnchor),
      stackView.bottomAnchor.constraint(equalTo: layoutMarginsGuide.bottomAnchor),

      imageView.widthAnchor.constraint(equalToConstant: 20),
      imageView.heightAnchor.constraint(equalTo: imageView.widthAnchor),
    ])
  }

  @available(*, unavailable)
  required init?(coder: NSCoder) {
    fatalError("init(coder:) has not been implemented")
  }
}

final class ViewController: UIViewController {
  private let collectionViewLayout: UICollectionViewCompositionalLayout = {
    var configuration = UICollectionLayoutListConfiguration(appearance: .plain)
    configuration.showsSeparators = false
    return UICollectionViewCompositionalLayout.list(using: configuration)
  }()

  lazy var collectionView: UICollectionView = {
    let collectionView = UICollectionView(frame: .zero, collectionViewLayout: collectionViewLayout)
    collectionView.register(UICollectionViewCell.self, forCellWithReuseIdentifier: "Cell")
    return collectionView
  }()

  private lazy var dataSource: UICollectionViewDiffableDataSource<Int, String> =
    .init(collectionView: collectionView) { collectionView, indexPath, item in
      let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "Cell", for: indexPath)
      cell.contentConfiguration = ContentConfiguration(image: UIImage(systemName: "globe"), text: item)
      cell.backgroundConfiguration = UIBackgroundConfiguration.listPlainCell()
      return cell
    }

  let items: [String] = TimeZone.knownTimeZoneIdentifiers

  override func viewDidLoad() {
    super.viewDidLoad()

    view.addSubview(collectionView)
    collectionView.autoresizingMask = [.flexibleWidth, .flexibleHeight]
    collectionView.frame = view.bounds

    var snapshot = NSDiffableDataSourceSnapshot<Int, String>()
    snapshot.appendSections([0])
    snapshot.appendItems(items)
    dataSource.apply(snapshot)
  }
}
```

それと、iOS 14から使える[`UIHostingConfiguration`](https://developer.apple.com/documentation/SwiftUI/UIHostingConfiguration)も書いてみました。

https://github.com/woxtu/UIHostingConfigurationBackport

以上です。

https://developer.apple.com/documentation/uikit/appearance_customization/configurations
