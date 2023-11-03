---
title: "UIImageWriteToSavedPhotosAlbum(_:_:_:_:)をasyncな関数にする"
emoji: "🔨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ios", "swift", "uikit"]
published: true
---

カメラロールに画像を保存する[`UIImageWriteToSavedPhotosAlbum(_:_:_:_:)`](https://developer.apple.com/documentation/uikit/1619125-uiimagewritetosavedphotosalbum)という関数があるんですがセレクターを要求することもあって処理の完了を待つのがつらいので次のようにすると良いと思います。

```swift
private final class UIImageWriteToSavedPhotosAlbumCompletion: NSObject {
  var continuation: CheckedContinuation<Void, any Error>?

  @objc func image(_ image: UIImage, didFinishSavingWithError error: Error?, contextInfo: UnsafeRawPointer) {
    if let error {
      continuation?.resume(throwing: error)
    } else {
      continuation?.resume()
    }
    continuation = nil
  }
}

func UIImageWriteToSavedPhotosAlbum(_ image: UIImage) async throws {
  let completion = UIImageWriteToSavedPhotosAlbumCompletion()
  return try await withCheckedThrowingContinuation { continuation in
    completion.continuation = continuation

    UIImageWriteToSavedPhotosAlbum(
      image,
      completion,
      #selector(UIImageWriteToSavedPhotosAlbumCompletion.image(_:didFinishSavingWithError:contextInfo:)),
      nil
    )
  }
}
```

使います。

```swift
try await UIImageWriteToSavedPhotosAlbum(image)
```

以上です。

https://developer.apple.com/documentation/swift/withcheckedthrowingcontinuation(function:_:)
