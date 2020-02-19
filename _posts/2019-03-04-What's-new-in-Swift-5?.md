---
title: What's new in Swift 5?
published: true
---

原文出處: https://learnappmaking.com/swift-5-whats-new/

## Filter And Count With “count(where:)”

在 iOS 中我們很常需要從一個列表中有條件的選擇元素並計算個數，例如：

```swift
let scores = [1, 3, 8, 2, 5, 6, 2, 10]
let count = scores.filter({ $0 > 5 }).count
print(count)  // output 3
```

但這樣有兩個缺點：

- 太囉唆(too verbose)：我們先做了 filtering，接著我們才做 counting，但我們實際上只想做 counting，在閱讀上也很容易忽略掉 `.count`
- 太浪費(too wasted)：我們做了一個中介運算 `filter(_:)`，但這個運算結果被遺棄了(discarded)，這就像我們從一個水果籃複製一個蘋果，只為了算『蘋果一個』，接著我們就丟掉它了

因此我們有了 `count(where:)` ([SE-0220](https://github.com/apple/swift-evolution/blob/master/proposals/0220-count-where.md))，只在一個運算中做 filter 和 count

```swift
let scores = [1, 3, 8, 2, 5, 6, 2, 10]
let count = scores.count(where: { $0 > 5 })
print(count) // Output: 3
```

## isMultiple(of: )

很多時候我們會需要測試 **`一個數是否可以被另一個數整除(divisible)`** 此時我們可能會這樣寫：

```swift
let result = 5 % 2 == 0
if result {
	print("5 is even")
} else {
	print("5 is odd")
}
```

在 [SE-0225](https://github.com/apple/swift-evolution/blob/master/proposals/0225-binaryinteger-iseven-isodd-ismultiple.md) 提出了一新的函數實作 isMultiple(of:)給 integer 變數，用來檢查該變數是否是另一個整數的倍數：

```Swift
let number = 42
if number.isMultiple(of: 2) {
	print("\\(number) is even")
}
```

這樣寫有比較簡單吧！

這樣寫有幾個好處：

- 比較接近一般英語的語法：if number is multiple of 2，因此增加了可讀性
- XCode 可以探索到 functions，可以避免開發者沒注意到使用了 %
- 因為 % 運算元造成的錯誤並不罕見，何況在不同程式語言中 % 的實作還有所不同

## Result 型態

在 [SE-0235](https://github.com/apple/swift-evolution/blob/master/proposals/0235-add-result.md) 中，為 Swift 新加入了一個資料型態 `Result`

你可以有 2-3 種方式做例外處理，例如 do-try-catch, try?, guard，社群中因此發展出了 [Result type](https://www.swiftbysundell.com/posts/the-power-of-result-types-in-swift) 的用法，這個資料型態封裝了兩種 state：success 和 failure，而因為這個資料型態太廣為使用，因此被收錄進了 Swift 的標準函式庫，以下是提案中提及的解決方案：

```swift
public enum Result<Success, Failure: Error> {
    case success(Success), failure(Failure)
}
```

上述方法定義了一個 [enumeration](https://learnappmaking.com/enums-swift) 包含了 `.success` 和 `.failure` 兩個案例，這兩個案例各有一個關聯的型態 `Success` 和 `Failure`，所有的資料型態在這裡都是 [generic](https://www.notion.so/Swift-5-29141c39e7f842b6a880d0321fbb5394#7b31b9b194da4c5a94b485e5dd426a19)，因此 `Success` 可以是任何值，但你傳遞給 `Failure` 的值必須符合 Error 這個 protocol。

簡單來說，`Result` 資料型態可以在非同步函數中用來當作參數傳遞：

```swift
dataTask(with: url) { (result: Result<Data, Error>) in
	switch result {
    case let .success(data):
        handleResponse(data: data)
    case let .error(error):
        handleError(error)
    }
}
```

The `Result` type encapsulates possible return values and errors in one object. It also uses the power of enumerations to help you write expressive code, i.e. a `switch` block that gracefully deals with return values and errors.

This change to Swift is definitely more involved than just adding a new convenient function, so it’s worth it to [investigate it further](https://github.com/apple/swift-evolution/blob/master/proposals/0235-add-result.md). You could even call it philosophical: the Swift language designers actively discuss how the language is used, as opposed to just providing Swift’s syntax.

## Handling Future Enum Cases

[SE-0192](https://github.com/apple/swift-evolution/blob/master/proposals/0192-non-exhaustive-enums.md) 針對以下議題提出一個解決方案：

- 當你對 enum 使用 switch 時，你需要列舉窮盡所有 case
- 你未來有可能需要增加一個 case 到 enum 中，這對 switch 來說是一個 code-breaking change (意思是這樣的改變會導致 switch code 被 break 而不能用)

由於 switching enums 需要窮盡列舉，因此當你想要增加一個新的 case 到 enum，原本已經寫好在用的 switch 就會爛掉而不能用，因為你必須也新增這個新的 enum case 到 switch-case 中。

這對 libraries, SDKs 和 frameworks 來說都是很笨重的，因為每次你新增一個 case 進去，你就可能毀了別人寫的 code，此外，code-breaking change 還可能對相容性有負面影響。

例如以下的 enum：

```Swift
enum Fruit {
    case apple
    case orange
    case banana
}
```

然後我們用 switch 來列舉：

```Swift
let fruit = ...

switch fruit {
case .apple:
    print("Purchasing apple for $0.50")
case .orange:
    print("Purchasing orange for $0.39")
case default:
    print("We don't sell that kind of fruit here.")
}
```

我們加入了 `default` 這個 `case`，因此這個 `switch` 敘述有達到窮盡列舉，然而，假如你沒有使用 `default`，而是僅僅將 `.apple`, `.orange` 和 `.banana` 列舉，接著之後你或其他開發者在 `Fruit` 中新增了一個新的水果，就會導致你的 switch code 炸掉。

這個作法有兩個部分：

- Enumerations in the Swift standard library, and imported from elsewhere, can be either frozen or non-frozen. A frozen enumeration cannot change in the future. A non-frozen enum can change in the future, so you’ll need to deal with that.
- When switching over a non-frozen enum, i.e. one that can change in the future, you should include a “catch-all” `default` case that matches any values that the `switch` doesn’t already explicitly match. If you don’t use `default` when you need to, you’ll get a warning.

這會衍生出另外一個問題：如果你有不想包含的 case，但你沒有想到，你怎麼能確定這個 `default` 有排除掉你可能並不想要包含的值？或是有包含到之後可能被加進 `Fruit` 的 case 呢？Swift compiler 也不會知道，因此 compiler 並不會提醒你有沒列舉到的 case。

假設你現在新增了一個 `.pineapple` 到 `Fruit` 中，你要如何知道我們可以賣這個水果但我們不會這麼做？又或者你要怎麼知道這是一個由 framework 新加入的水果，而且我們可以賣它？

在 Swift 5.0 中，可以在 default 這個 case 中加入一個新的 keyword `@unknown`，這樣並不會改變原本我們理解的 `default` 行為：

```swift
switch fruit {
case .apple:
    ...
case @unknown default:
    print("We don't sell that kind of fruit here.")
}
```

`@unknown` 是用來在 Xcode 中觸發 warning 的，當你正在處理有潛在為窮盡列舉的 switch 敘述時，警告就會被觸發。

## Flatten Nested Optionals With “try?

_Nested optionals_ are… strange. Here’s an example:

```swift
let number:Int?? = 5
print(number)
// Output: 5
```

The above `number` constant is doubly wrapped in an [optional](https://learnappmaking.com/swift-optionals-how-to/), and its type is `Int??` or `Optional<Optional<Int>>`. Although it’s OK to have nested optionals, it’s also downright confusing and unnecessary.

Swift has a few ways to avoid accidentally ending up with nested optionals, for example in casting with as? and in optional chaining. However, when using try? to convert errors to optionals, you can still end up with nested optionals.

Here’s an example:

```swift
let car:Car? = ...
let engine   = try? car?.getEngine()
```

In the above example, `car` is an optional of type `Car?`. On the second line, we’re using optional chaining, because `car` is an optional. The return value of the expression `car?.getEngine()` is optional too, because of the optional chaining. It’s type is `Engine?`.

當你結合了 `try?`，它的返回值是 optional，你因此會得的 double/nested optional，因此 `engine` 最後的型態是 `Engine??`(因為 `car?.getEngine()` 回傳的 `Engine?`，被 `try?` 再次包裝成 `Engine??`)，這就會產生問題了，因為你必須要 unwrap 兩次才能得 engine 真正的值

因為 `as?` 已經 flattens optionals，一個脫離 nested optional 的方法是採用以下的方式：

```swift
if let engine = (try? car?.getEngine()) as? Engine {
    // OMG!
}
```

這段程式碼將 `Engine??` 轉型(cast)成 `Engine`，因為 `as?` 的關係而 flatten optionals 了，轉型本身是沒有用的，因為我們要的是 `Engine`。

既然如此，為什麼不直接讓 `try?` 也具有 `as?` 的功能(flatten optionals)，因此在 [SE-0230](https://github.com/apple/swift-evolution/blob/master/proposals/0230-flatten-optional-try.md) 中就提出了解決方案：

> It flattens nested optionals resulting from `try?`, giving it the same behavior as `as?` and optional chaining.

## The New “compactMapValues()” Function For Dictionaries

The Swift standard library includes two useful functions for arrays and dictionaries:

- The `map(_:)` function [applies a function to array items](https://learnappmaking.com/map-reduce-filter-swift-programming/), and returns the resulting array, while the `compactMap(_:)` function also [discards array items](https://learnappmaking.com/swift-flatmap-compactmap-how-to/) that are `nil`
  The `mapValues(_:)` function does the same for dictionaries, i.e. it applies a function to dictionary values, and returns the resulting dictionary – but it doesn’t have a `nil` discarding counterpart

[SE-0218](https://github.com/apple/swift-evolution/blob/master/proposals/0218-introduce-compact-map-values.md) 修改了函示庫，並為 dictionaries 加入了 `compactMapValues(_:)` 函式，這個 function 結合了 array 的 `compactMap(_: )` 和 dictionaries 的 `mapValues，更有效率的` mapping-and-filtering dictionary 的值

考慮一個情境，當你要調查你家成員的年齡(integer)時，你那愚笨的叔叔 Bob 拼出了他的年紀而非數字，因此你可能會得到以下的程式碼內容：

```Swift
let ages = [
    "Mary": "42",
    "Bob": "twenty-five har har har!!",
    "Alice": "39",
    "John": "22"
]

let filteredAges = ages.compactMapValues({ Int($0) })
print(filteredAges)
// Output: ["Mary": 42, "Alice": 39, "John": 22]
```

`compactMapValues(_:)` 在這個情境中可以有用的移除 dictionary 中的 `nil` 值，或是當你正在使用 failable initializers 來轉換 dictionary values 也很有用。
