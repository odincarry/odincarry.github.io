---
layout: post
title: 解鎖AI開發姿勢—規格驅動開發(SDD)實現Cocos小遊戲
date: 2026-04-30 15:14 +0800
description:
image:
category:
tags:
published: [AI]
sitemap: false
---

### 前情提要

因為這幾年的AI爆發，我們幾乎很難脫離對他的使用，在軟體工程師更是如此，從一開始提示詞輔助開發到後來的Vibe Coding，如今甚至不需要打任何的手動coding，AI生成的準確率也是高得嚇人，至於實際怎麼做呢，來趁這次機會，來用沒碰過的遊戲引擎完成一個小遊戲。

### SDD(規格驅動開發)

規格開發驅動其實是一個概念，主要流程是從 **產品需求** &rarr; **技術設計** &rarr; **任務清單 (規格)** &rarr; **AI Coding Agent 執行**，而在軟體業這個流程算是行之有年的東西，但如何導入到AI開發就是這次的課題，這次用的是[OpenSpec](https://github.com/Fission-AI/OpenSpec)，那就開始吧。

### 安裝環境及工具

1. 到官方下載[Cocos Dashboard](https://docs.cocos.com/creator/3.3/manual/zh/getting-started/install/)，他用來管理遊戲引擎(Cocos Creator)。
   ![](/assets/img/post/2026-0430/p3.png)

2. 本身Cocos creator的IDE功能並不完全，這邊去連接日常習慣的IDE，[官方文件](https://docs.cocos.com/creator/2.4/manual/zh/getting-started/coding-setup.html)有做教學。

3. 安裝OpenSpec到本地。

### 建立專案

1. 在Cocos Dashboard 建立一個空的新專案，

2. 最後在專案目錄下 **openspec init**，安裝過程第一步會跳出使用的AI Agent，這裡我選Claude Code。

![](/assets/img/post/2026-0430/p0.png)

3. 接著會自動新增openspec的資料夾以及claude對於openspec相關的skill。
   ![](/assets/img/post/2026-0430/p1.png)

### 提出需求

1. 工具準備好後，我們來試看看功能吧，起首先下Command **openspec-propose**來做需求論述，打完後他不會直接產Code，目前都是做前期**規劃**。
   ![](/assets/img/post/2026-0430/p4.png)

2. 一系列的詢問後，openspec資料夾會生出一些文件，至於文件的分層，可以拆開來這樣看。

```
   openspec/proposals/wof-geme-web/
      ├── proposal.md   # 為什麼做、做什麼
      ├── design.md     # 技術設計決策
      ├── specs/        # 詳細規格（API / 元件）
      └── tasks.md      # 實作任務清單
```

- 實作很難問一次就符合預期，在規劃的前後都會有漏掉的部分，因此可以在執行之前問更多細節，讓整體成品更完善。

### 執行Tasks生成代碼

1. 在詢問過程，tasks.md會有一系列的代辦清單，下Command **/openspec-apply-change** 來依序完成任務。
   ![](/assets/img/post/2026-0430/p5.png)

### 遊戲場景建立

參考**design.md**建立場景，之後綁定相對應的腳本即可，本來預計想連這步都丟給AI，但可惜的是，目前Cocos相較於Unity較大間的引擎，官方還沒有提供MCP工具，其他作者提供的MCP有token消耗過大的問題，這部分就需要手動去實現，最後加上一些資源圖，就可以完成一個小遊戲了。
![](/assets/img/post/2026-0430/p7.png)

- 包括後續要新增驗證功能，也是透過一樣的流程去實現。

![](/assets/img/post/2026-0430/p8.png)

### 結語

目前這樣流程下來，對於AI的產出可以做到相當大的把關，他會在實作前逐步分析並講解該做什麼、怎麼做。不管是想透過AI開發還是透過AI學習新技術，都是一個不錯的方式，而如何將這個方式去銜接傳統的協作開發，就是工程師們要去協調的，另外也有很多其他的SDD開發工具，甚至AWS直接嵌在自家的IDE [KIRO](https://kiro.dev/)裡面，雖然這種開發模式不見得受到所有開發者認同，但往相信這些都是趨勢。
