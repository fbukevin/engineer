---
title: AJAX state 和 readyState
published: true
---

# Key

- AJAX 中的 request 是由 XMLHttpRequest 這個類別的物件處理的
- 所謂 AJAX 最主要就是要跟伺服器做溝通(request)

## readyState

一個溝通一定會有幾個步驟，一開始在跟伺服器進行溝通時，我們需要透過檢查 readyState 這個 XMLHttpRequest 物件的屬性值，來判斷目前的溝通進行到第幾步驟，有以下幾個步驟：

- `0` 還沒開始
- `1` 存取中
- `2` 已存取
- `3` 資訊交換中
- `4` 一切完成

伺服器會回應目前溝通做到第幾步驟給 httpRequest，httpRequest 把回應的值放到 httpRequest.readyState 這個屬性中，所以我們可以透過檢查他的值來判斷現在是做到第幾步驟，其中步驟 4 在 XMLHttpRequest 中有定義一個屬性代表它，用來給我們做判斷

```javascript
// 建立一個 XMLHttpRequest 物件叫做 httpRequest
var httpRequest = new XMLHttpRequest();

// (跟伺服器進行溝通中..)
...

// 檢查伺服器回應目前溝通做到第幾步驟
// (下面 if 中把 XMLHttpRequest.DONE 替換成 4 效果一樣)
if(httpRequest.readyState == XMLHttpRequest.DONE){
    // 一切 OK，完成溝通，繼續解析伺服器回傳結果
} else {
    // 還沒完成
}
```

## state

確認溝通完畢後，接下來我們要做的就是解析伺服器回傳(response)了什麼給我們，第一個要檢查的就是回傳的結果是不是正常，如果正常我們才有必要進一步解析回傳的內容，不正常就要去檢查為什麼不正常，這個正不正常的狀態，就是透過 XMLHttpRequest 物件的 state 這個屬性的值判斷，我們又稱之為 HTTP status code(狀態碼)，可以參考[這裡](https://zh.wikipedia.org/wiki/HTTP%E7%8A%B6%E6%80%81%E7%A0%81)有完整的狀態碼列表

所以我們可以繼續完成前面的程式：

```javascript
// 建立一個 XMLHttpRequest 物件叫做 httpRequest
var httpRequest = new XMLHttpRequest();

// (跟伺服器進行溝通中..)
...

// 檢查是否到最後一個步驟了
if(httpRequest.readyState == XMLHttpRequest.DONE){
    // 一切 OK，完成溝通，繼續解析伺服器回傳結果
    if (httpRequest.status === 200) {
        // HTTP status 200 表示一切正常，可以安心解析伺服器回傳內容啦！
    } else {
        // 似乎有點問題。
        // 或許伺服器傳回了 404（查無此頁）
        // 或者 500（內部錯誤）什麼的
        // 或是 ...
    }
} else {
    // 還沒完成
}
```
