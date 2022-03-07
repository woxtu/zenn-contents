---
title: "Alamofireで空のレスポンスを取得する"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["swift", "alamofire"]
published: true
---

[Alamofire](https://github.com/Alamofire/Alamofire)で空のレスポンスを取得したいときは、[`Empty`](https://alamofire.github.io/Alamofire/Structs/Empty.html)型もしくは[`EmptyResponse`](https://alamofire.github.io/Alamofire/Protocols/EmptyResponse.html)プロトコルに適合した型を指定します。

```swift
AF.request("https://httpbin.org/status/204").responseDecodable(of: Empty.self) {
  debugPrint($0)
}
```

```swift
struct NoContent: Decodable, EmptyResponse {
  static func emptyValue() -> NoContent {
    return NoContent()
  }
}

AF.request("https://httpbin.org/status/204").responseDecodable(of: NoContent.self) {
  debugPrint($0)
}
```

ただし、デフォルトで空のレスポンスが許容されるステータスコードは`204`と`205`だけなので、それ以外にも対応したい場合は`emptyResponseCodes`も併せて指定する必要があります。

```swift
AF.request("https://httpbin.org/status/201").responseDecodable(of: Empty.self, emptyResponseCodes: [201, 204, 205]) {
  debugPrint($0)
}
```

以上です。

https://github.com/Alamofire/Alamofire/blob/master/Documentation/Usage.md#response-handling
