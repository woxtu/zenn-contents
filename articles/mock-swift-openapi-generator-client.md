---
title: "Swift OpenAPI Generatorで生成したクライアントをモックする"
emoji: "🔧"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["swift", "openapi"]
published: true
---

[Swift OpenAPI Generator](https://github.com/apple/swift-openapi-generator)で生成したクライアントをモックしたいときは、`Client`と一緒に生成される`APIProtocol`を使います。

```swift
struct ClientMock: APIProtocol {
  func getGreeting(_ input: Operations.getGreeting.Input) async throws -> Operations.getGreeting.Output {
    let message = "Hello, \(input.query.name ?? "Stranger")!"
    return .ok(.init(body: .json(.init(message: message))))
  }
}

let client: some APIProtocol = ClientMock()
let response = try await client.getGreeting(.init(query: .init(name: "John")))
```

以上です。

https://github.com/apple/swift-openapi-generator/tree/4c8ed5cec75ccf7f3c48f744b44bafb93235f492/Examples/GreetingService/Tests/GreetingServiceMockTests
