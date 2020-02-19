---
title: XCode 中為 macOS Command Line Tool 專案加入 Unit Test
published: true
---

XCTest module 是 XCode 內建提供的單元測試框架模組，如果是建立一個 iOS App 或 macOS Cococa App 這種帶有視窗畫面的應用程式專案，XCode 會自動在創建專案時一併建立測試專案，但如果在 XCode 中建立一個 macOS Command Line Tool，預設是不會自動建立 XCTest Unit Test 的

<figure>
  <img src="assets/20190217/20190217-1.png" alt=""/>
  <figcaption>如果直接在專案中新增一個 Unit Test 檔案，會發生 Connot load underlying module for 'XCTest'</figcaption>
</figure>

正確做法是在左邊的 Project Navigator 中，切換到 "Show the Test navigator"，小圖示是：

![](assets/20190217/20190217-2.png)

接著在最左下方點選 + 的符號，選擇第一個 New Unit Test Target...：
![](assets/20190217/20190217-3.png)

輸入 Test class name
![](assets/20190217/20190217-4.png)

按下 Finish 後，就可以在 Show the Project navigator (左上第一個資料夾圖示) 中看到多了一個 Group、Test class 和 Info.plist，其中 Test class 已經繼承了 XCTestCase，import 的 XCTest 模組也沒有錯誤了
![](assets/20190217/20190217-5.png)

按下 Finish 後，就可以在 Show the Project navigator (左上第一個資料夾圖示) 中看到多了一個 Group、Test class 和 Info.plist，其中 Test class 已經繼承了 XCTestCase，import 的 XCTest 模組也沒有錯誤了
![](assets/20190217/20190217-6.png)

然後在 show the Test navigator 中就會出現新增的 Unit Test 了
