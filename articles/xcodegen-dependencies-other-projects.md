---
title: XcodeGenで他のプロジェクトに依存する
emoji: ⚙️
type: tech
topics: [iOS, Xcode, XcodeGen]
published: true
---

[XcodeGen](https://github.com/yonaskolb/XcodeGen)で他のプロジェクトに依存しているプロジェクトを生成したくなるときがあります。

ワークスペースを使う場合は、ワークスペースに他プロジェクトを追加したあと`dependencies`にimplicitなフレームワークを追加します。

```yaml
name: AwesomeProject
targets:
  AwesomeApp:
    type: application
    platform: iOS
    sources:
      - Sources
    dependencies:
      # 👇
      - framework: Alamofire.framework
        implicit: true
```

サブプロジェクトを使う場合は、`projectReferences`に他プロジェクトを追加したあと`dependencies`にターゲットを追加します。

```yaml
name: AwesomeProject
targets:
  AwesomeApp:
    type: application
    platform: iOS
    sources:
      - Sources
    dependencies:
      # 👇
      - target: Alamofire/Alamofire iOS
projectReferences:
  # 👇
  Alamofire:
    path: ./Carthage/Checkouts/Alamofire/Alamofire.xcodeproj
```

以上です。

- https://github.com/yonaskolb/XcodeGen/blob/master/Docs/ProjectSpec.md#dependency
- https://github.com/yonaskolb/XcodeGen/blob/master/Docs/ProjectSpec.md#project-reference

