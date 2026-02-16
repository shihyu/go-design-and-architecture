# Day8 - Interface即類型的使用定義

在 Go 語言中，`interface` 是一組方法的集合，透過 `type` 和 `interface` 關鍵字來定義。這些方法的集合唯一確定了這個 `interface` 類型所表示的契約。讓我們來看看一個典型的 `interface` 類型 `MyInterface` 的定義：

```go
type MyInterface interface {
    M1(int) error
    M2(io.Writer, ...string)
}
```

從這個定義中，我們可以看到，`interface` 類型 `MyInterface` 包含了兩個方法：`M1` 和 `M2`。Go 語言要求 `interface` 類型聲明中的方法必須是具名的，並且方法名字在這個 `interface` 類型的方法集合中必須唯一。從 Go 1.14 版本開始，Go 允許嵌入的不同 `interface` 類型的方法集合存在交集，但前提是交集中的方法不僅名字一樣，連方法簽名（參數列表和返回值列表）也必須一致，否則 Go 編譯器會報錯。

例如，以下示例中 `Interface3` 嵌入了 `Interface1` 和 `Interface2`，但它們之間的 `M1` 方法簽名不同，導致編譯出錯：

```go
type Interface1 interface {
    M1()
}
type Interface2 interface {
    M1(string) 
    M2()
}

type Interface3 interface {
    Interface1
    Interface2 // 編譯器報錯：duplicate method M1
    M3()
}
```

Go 語言也可以定義空 `interface` 類型，代表沒有定義任何方法集合的 `interface`：

```go
type EmptyInterface interface {}
```

如果一個變數的類型是空 `interface` 類型，由於空 `interface` 類型的方法集合為空，這意味著**任何類型都實現了空 `interface` 的方法集合**。所以我們可以將任何類型的值賦值給空 `interface` 類型的變數。例如：

```go
var i interface{} = 15 // ok
i = "hello, golang" // ok
type T struct{}
var t T
i = t  // ok
i = &t // ok
```

空 `interface` 類型能夠接受任意類型變數值，使它成為 Go 在引入泛型之前唯一具備"泛型"能力的語法元素。

```
 Interface 內部記憶體結構示意圖：

 ═══════════════════════════════════════════════
 非空 Interface（iface）：
 ┌─────────────────────────────────────────┐
 │  type MyInterface interface {           │
 │      M1(int) error                      │
 │      M2(io.Writer, ...string)           │
 │  }                                      │
 │                                         │
 │  var i MyInterface = someValue          │
 │                                         │
 │  iface 結構：                            │
 │  ┌──────────┬──────────┐               │
 │  │  tab     │  data    │               │
 │  │  (*itab) │  (ptr)   │               │
 │  └────┬─────┴────┬─────┘               │
 │       │          │                      │
 │       ▼          ▼                      │
 │  ┌─────────┐  ┌──────────┐             │
 │  │  itab   │  │ 實際數據  │             │
 │  │ ┌─────┐ │  │ (具體值) │             │
 │  │ │_type│ │  └──────────┘             │
 │  │ ├─────┤ │                            │
 │  │ │ M1  │─┼──▶ 具體類型的 M1 實作      │
 │  │ ├─────┤ │                            │
 │  │ │ M2  │─┼──▶ 具體類型的 M2 實作      │
 │  │ └─────┘ │                            │
 │  └─────────┘                            │
 └─────────────────────────────────────────┘

 ═══════════════════════════════════════════════
 空 Interface（eface）：
 ┌─────────────────────────────────────────┐
 │  var i interface{} = 15                 │
 │                                         │
 │  eface 結構：                            │
 │  ┌──────────┬──────────┐               │
 │  │  _type   │  data    │               │
 │  │  (*_type)│  (ptr)   │               │
 │  └────┬─────┴────┬─────┘               │
 │       │          │                      │
 │       ▼          ▼                      │
 │  ┌─────────┐  ┌──────────┐             │
 │  │ int 的  │  │    15    │             │
 │  │ 類型描述│  │ (int值)  │             │
 │  └─────────┘  └──────────┘             │
 └─────────────────────────────────────────┘
```包括 Go 標準庫在內的一些通用資料結構與算法的實現，都使用了空 `interface` `interface{}` 作為參數元素的類型，這樣我們就無需為每種支持的元素類型單獨編寫程式碼或方法。

Go 語言還支持從 `interface` 類型變數“還原”其右值的類型與值信息，這個過程稱為“Type Assertion”。Type Assertion 通常使用以下語法形式：

```go
v, ok := i.(T)
```

其中 `i` 是某個 `interface` 類型變數。如果 `T` 是一個非 `interface` 類型且 `T` 是想要還原的類型，這句程式碼的含義是“斷言”儲存在 `interface` 類型變數 `i` 中的值的類型為 `T`。

如果 `interface` 類型變數 `i` 之前被賦予的值確為 `T` 類型的值，這個語句執行後，左側 `comma, ok` 語句中的變數 `ok` 的值將為 `true`，變數 `v` 的類型為 `T`，其值為之前賦予給 `i` 的值。如果 `i` 之前被賦予的值不是 `T` 類型，這個語句執行後，變數 `ok` 的值為 `false`，變數 `v` 的類型仍然是 `T`，但其值是類型 `T` 的初始值。

好了，到這裡關於 `interface` 類型的基礎語法我們已經全部講完了。有了這個基礎後，我們再來看看 Go 語言 `interface` 定義的慣例，也就是盡量定義“小 `interface`”。

### 盡量定義“小 `interface`”

`interface` 類型的背後，是透過將類型的行為封裝成契約，建立雙方共同遵守的約定，從而降低耦合度。和生活中的契約有繁有簡、形式多樣一樣，程式碼間的契約也有大小之分。而 Go 選擇了簡單的形式，主要體現在以下兩點上：

#### 隱式契約，無需顯示，立即生效

在 Go 語言中，`interface` 類型與其實現者之間的關係是隱式的，不需要像其他語言（如 Java）那樣要求實現者顯式聲明 `implements`。實現者只需要實現 `interface` 方法集合中的全部方法，便自動生效。

#### 更傾向於“小契約”

這點也不難理解。若契約太繁雜，就會束縛手腳，缺少靈活性。因此 Go 選擇了“小契約”，表現在程式碼上就是盡量定義小 `interface`，即方法個數在 1~3 個之間的 `interface`。Go 語言之父 Rob Pike 曾說過，“`interface` 越大，封裝程度越弱”，這也是 Go 社群傾向於定義小 `interface` 的原因之一。

##### 小 `interface` 的優點

* **`interface` 越小，封裝程度越高**  
  封裝程度越高，對應的集合空間就越大；封裝程度越低，也就是越具像化，更接近事物真實面貌，對應的集合空間越小。
* **小 `interface` 易於實現和測試**  
  小`interface`擁有較少的Method，一般情況下只有一個Method。所以要想滿足這一`interface`，我們只需要實現一個Method或者少數幾個Method就可以了，這顯然要比實現擁有較多Method的`interface`要容易得多。尤其是在單元測試環節，構建類型去實現只有少量Method的`interface`要比實現擁有較多Method的`interface`付出的勞動要少許多。
* **小 `interface` 表示職責單一，易於複用和組合**  
  寫程式時，如果有眾多候選`interface`類型供我們選擇，我們會怎麼選擇呢？顯然，我們會選擇那些新`interface`類型需要的契約職責，同時也要求不要引入我們不需要的契約職責。在這樣的情況下，擁有單一或少數Method的小`interface`便更有可能成為我們的目標，而那些擁有較多Method的大`interface`，可能會因引入了諸多不需要的契約職責而被放棄。由此可見，小`interface`更契合 Go 的組合思想，也更容易發揮出組合的威力。

### 如何定義小 `interface`

保持簡單有時比複雜更難。小 `interface` 雖好，但如何定義出小 `interface` 是每個 Gopher 面前的一道難題。雖然這道題沒有標準答案，但有一些建議可供參考：

1. **首先，先封裝出 `interface`，不管大小**：在定義小 `interface` 之前，我們需要深入理解問題領域，聚焦封裝並發現 `interface`。初期，先不要介意 `interface` 裡的方法數量，因為對問題域的理解是循序漸進的。在第一版程式碼中直接定義出小 `interface` 可能不現實。例如，標準庫中的 `io.Reader` 和 `io.Writer` 也不是 Go 剛誕生時就有的，而是在發現對網路、文件、其他資料處理的實現十分相似後才封裝出來的。
2. **將大 `interface` 拆分為小 `interface`**：有了 `interface` 後，我們會在程式碼中各處使用 `interface`。一段時間後，分析哪些場合使用了 `interface` 的哪些方法，是否可以將這些方法提取出來，放入一個新的小 `interface` 中。
3. **注意 `interface` 的單一職責**：被拆分成的小 `interface` 是否需要進一步拆分，以滿足單一職責原則，如同 `io.Reader` 一樣。如果需要，就進一步拆分，提升封裝程度。

### 總結

* **`interface` 是方法的集合**：用來定義類型的行為契約。
* **Go 中的 `interface` 是隱式實現的**：不需要顯式聲明 `implements`。
* **小 `interface` 更具靈活性和複用性**：方法數量在 1~3 個之間。
* **將大 `interface` 拆分為小 `interface`**：聚焦單一職責，提升封裝程度，符合高內聚、低耦合的精神。

這些建議和技巧將幫助你在 Go 語言中有效地使用 `interface`，寫出更具高內聚、低耦合的程式碼。

更多Go語言相關的文章，歡迎關注我的部落格: <https://kaichiachen.github.io/2024/01/31/golang/go_interface/>

  
