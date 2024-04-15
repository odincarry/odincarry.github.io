---
layout: post
title: 探究Flutter Isolate
date: 2024-04-09 13:53 +0800
description:
image:
category: [Frontend]
tags: [Flutter, Dart]
published: true
sitemap: false
---
### 前情提要
當碰到跑大量的演算法，或是下載較大的檔案，都會碰到UI整個卡住的情形，那這到底是怎麼回是?這要從底層開始說起。

首先要知道，Dart是一種單執行緒語言，這意味著在運行時，都會依序處理完佇列中每個工作(Event Loop)。
 ![](/assets/img/post/2024-0409/p3.png)

打開Observatory debugger看一下，當我們在運行時(run main.dart)，可以發現一定會有一個叫main的Isolate。
 ![](/assets/img/post/2024-0409/p1.png)

### 什麼是Isolate
而Isolate作為Dart的Process，是單一執行緒，而每個Isolate記憶體是不共享的，當他們要傳達訊息時，需要透過socket的方式進行，既然Isolate會依序執行工作，那為甚麼還會有卡住個問題呢?

### 非同步產生的問題

下方是一段讀取檔案的code，我們拿讀取檔案的部分為例。

```dart
void main() async {
  // Read some data.
  final fileData = await _readFileAsync();
  final jsonData = jsonDecode(fileData);

  // Use that data.
  print('Number of JSON keys: ${jsonData.length}');
}

Future<String> _readFileAsync() async {
  final file = File(filename);
  final contents = await file.readAsString();
  return contents.trim();
}
```

在處理非同步的問題，會運用到Async await Future的方式，這邊Dart跑到await _readFileAsync，Dart會先跳開去處理File I/O(這邊的非同步)直到檔案資料取完後，Dart才會繼續執行下一段程式。
![](/assets/img/post/2024-0409/p2.png)

而一般如果檔案不大，每段event結束後都可以快速的執行下一段event。
![](/assets/img/post/2024-0409/p4.png)

為了達到畫面保持順暢，Flutter為每個事件(event)添加每秒60次的"paint frame"(所謂的60Hz)。
當檔案太大時，如果使用非同步操作，會導致後續執行的event太晚接收，造成UI卡住，嚴重了話可能造成程式看起來沒有回應(沒有快速執行下一個event)。
![](/assets/img/post/2024-0409/p5.png)

### 如何解決
既然我們都知道，非同步(異步)的操作實際上只是**等待**某個某段程式直到做出回應，那我們要做到的就是平行處理(concurrency)，簡單來說就是創建另一個Isolate去執行那一段處理時間較長的工作。


### 創建方法
運用[dart:isolate]()
* Isolate.spawn(創建Isolate)

  ```dart
  import 'dart:isolate';

  void main() {
    Isolate.spawn(isolateFun, '隔離產生');
  }

  void isolateFun(message) {
    print(message);
  }
  ```

* compute(Spawn的包裝版): **單純做計算用可以直接使用**

  ```dart
  import 'package:flutter/foundation.dart';

  Future<int> computedFunction() async {
    return compute(computeMultiplyFunction, 100);
  }

  // the function must be a top-level function or a static method
  static int computeMultiplyFunction(int num) {
    return num * num;
  }
  ```

### 結語
經過這個事件，可以大致了解Dart在底層是怎麼運作的，而這邊可以嚐試使用[flutter_isolate](https://pub.dev/packages/flutter_isolate)，這個的好處是他可以自己設工作負載，幫助你的應用程式不卡卡。

