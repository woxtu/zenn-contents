---
title: "一般社団法人サイバー技術・インターネット自由研究会の電子公告をSwiftで読む"
emoji: "📟"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["swift", "telnet"]
published: true
---

https://zenn.dev/uzulla/articles/1e344dfbf45035

当然Swiftでも読みたい。

```swift
import Network

let connection = NWConnection(host: "koukoku.shadan.open.ad.jp", port: 23, using: .tcp)
connection.start(queue: .main)
while await withCheckedContinuation({ continuation in
  connection.receive(minimumIncompleteLength: 0, maximumLength: .max) { data, _, isComplete, _ in
    if let data {
      print(String(data: data, encoding: .shiftJIS)!, terminator: "")
    }
    continuation.resume(returning: !isComplete)
  }
}) {}
```

以上です。

https://developer.apple.com/documentation/network
