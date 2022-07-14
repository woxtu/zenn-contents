---
title: "Zigで出力をバッファリングする"
emoji: "🦎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["zig"]
published: true
---

Zigで出力をバッファリングさせたいときは、[`std.io.bufferedWriter`](https://ziglang.org/documentation/master/std/#std;io.bufferedWriter)を使います。

```zig
var bufferedWriter = std.io.bufferedWriter(std.io.getStdOut().writer());
var writer = bufferedWriter.writer();
try writer.print("Hello, world!", .{});
try bufferedWriter.flush();
```

以上です。
