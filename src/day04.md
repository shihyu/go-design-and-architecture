# Day4 - 搞清楚Go語言的生命週期

在剛開始學習 Go 語言的時候，我們可能經常會遇到這樣一個問題：一個 Go 專案中有數十個 Go packages，每個package中又有若干常數、變數、各種函數和方法，那 Go 程式究竟是從哪裡開始執行的呢？後續的執行順序又是什麼樣的呢？

了解這門語言的執行次序，對我們寫出結構合理、邏輯清晰的程式大有助益。無論你使用的是哪種編程範式（Paradigm），過程式的、物件導向的、函數式的，還是其他編程範式，深入了解一下總歸是好的。

今天這篇文章，我就帶你來了解一下 Go 程式的執行次序，這樣在後續閱讀和理解 Go 程式碼的時候，你就好比擁有了“通往寶藏的地圖”，可以直接沿著 Go 程式碼執行次序這張“地圖”去閱讀和理解 Go 程式碼，不會在龐大的程式庫中迷失。

Go 程式由一系列 Go package組成，程式碼的執行也是在各個package之間跳轉。和其他語言一樣，Go 也擁有自己的執行入口 - **main 函數**。這篇文章我們就從 main 函數入手，逐步展開，最終帶你掌握 Go 程式的執行順序。

### main.main 函數：Go 應用的入口函數

Go 語言中有一個特殊的函數：main package中的 main 函數，也就是 main.main，它是所有 Go 可執行程式的用戶層執行邏輯的入口函數。Go 程式的執行邏輯，會在這個函數內按照它的調用順序展開。

入口函數 - main 的定義是這樣的：

```go
package main

func main() {
    // 執行程式邏輯
    ...
}
```

你會發現，main 的定義非常簡單，沒有參數也沒有返回值。而且，Go 語言要求：可執行程式的 main package必須定義 main 函數，否則 Go 編譯器會報錯。在啟動了多個 Goroutine（Go 語言的輕量級userspace執行緒，後面的文章會詳細講解）的 Go 程式中，main.main 函數將在 Go 應用的主要的 Goroutine 中執行。main 函數返回就意味著整個 Go 程式的終結，而且你也不用管這個時候是否還有其他子 Goroutine 正在執行。

另外還值得我們注意的是，除了 main package外，其他package也可以擁有自己的名為 main 的函數或方法。但按照 Go 的可見性規則（小寫字母開頭的函數無法被導出），非 main package中自定義的 main 函數僅限於package內使用，就像下面程式碼這樣，這是一段在非 main package中定義 main 函數的程式碼：

```go
package pkg1

import "fmt"

func Main() {
    main()
}

func main() {
    fmt.Println("main func for pkg1")
}
```

可以看到，這裡 main 函數就主要是用來在package pkg1 內部使用的，它是沒法在package外使用的。

現在我們已經了解了 Go 應用的入口函數 main.main 的特性。不過對於 main package的 main 函數來說，你還需要明確一點，就是它雖然是用戶層邏輯的入口函數，但它卻不一定是Go程式第一個被執行的函數。這是為什麼呢？

這跟 Go 語言的另一個函數 init 有關。

### init 函數：Go package的初始化函數

除了前面講過的 main.main 函數之外，Go 語言還有一個特殊函數，它就是用於進行package初始化的 init 函數。和 main.main 函數一樣，init 函數也是一個無參數無返回值的函數：

```go
func init() {
    // package初始化邏輯
    ...
}
```

不過對於 init 函數來說，我們還需要注意一點，就是在 Go 程式中我們不能顯式地調用 init，否則就會收到編譯錯誤，就像下面這個範例，它表示的手工顯式調用 init 函數的錯誤做法：

```go
package main

import "fmt"

func init() {
  fmt.Println("init invoked")
}

func main() {
   init()
}
```

這樣，在構建並運行上面這些範例程式碼之後，Go 編譯器會報下面這個錯誤：

```go
#> go run call_init.go 
./call_init.go:10:2: undefined: init
```

實際上，Go package可以擁有不止一個 init 函數，每個組成 Go package的 Go 原始檔(.go文件)中，也可以定義 init 函數。

所以說，在初始化 Go package時，Go 會按照一定的次序，逐一、順序地調用這個package的 init 函數。一般來說，先傳遞給 Go 編譯器的原始檔中的 init 函數，會先被執行；而同一個原始檔中的多個 init 函數，會按定義順序依次執行。

現在我們就知曉了 main.main 函數可能並不是第一個被執行的函數的原因。所以，當我們要在 main.main 函數執行之前，執行一些函數或語句的時候，我們只需要將它放入 init 函數中就可以了。

了解了這兩個函數的執行順序之後，我們現在就來整體地看看，一個 Go package的初始化是以何種次序和邏輯進行的。

### Go package的初始化次序

從程式邏輯結構角度來看，Go package是程式邏輯封裝的基本單位，每個package都可以理解為是一個“自治”的、封裝良好的、對外部暴露有限接口的基本單位。一個 Go 程式就是由一組package組成的，程式的初始化就是這些package的初始化。每個 Go package還會有自己所需的外部package、常數、變數、init 函數（其中 main package有 main 函數）等。

在這裡你要注意：**我們在閱讀和理解程式碼的時候，需要知道這些元素在程式初始化過程中的初始化順序，這樣便於我們確定在某一行程式碼處這些元素的當前狀態。**  
簡而言之，記住 Go package的初始化次序並不難，你只需要記住這三點就可以了：

* package按"深度優先DFS"的次序進行初始化；
* 每個package內按以"常數 -> 變數 -> init 函數"的順序進行初始化；
* package內的多個 init 函數按出現次序進行自動調用。

下面這張圖完整呈現了 Go 程式從啟動到 main 函數執行的整個初始化流程：

```
 ┌─────────────────────────────────────────────────────────┐
 │                  Go 程式啟動流程                          │
 └─────────────────────────────────────────────────────────┘

 ┌──────────┐     ┌──────────┐     ┌──────────┐
 │ pkg C    │     │ pkg B    │     │ pkg A    │
 │(最深依賴) │◄────│          │◄────│          │◄──── main package
 └──────────┘     └──────────┘     └──────────┘

 初始化順序（深度優先）：pkg C → pkg B → pkg A → main

 每個 package 內部的初始化順序：
 ┌─────────────────────────────────────────┐
 │  ① const 常數初始化                      │
 │         ↓                               │
 │  ② var   變數初始化                      │
 │         ↓                               │
 │  ③ init() 函數執行（可有多個，按順序執行） │
 │         ↓                               │
 │  ④ (若為 main package) main() 函數執行   │
 └─────────────────────────────────────────┘

 完整時序圖：
 ┌────────┐  ┌────────┐  ┌────────┐  ┌────────────┐
 │ pkg C  │  │ pkg B  │  │ pkg A  │  │   main     │
 │ const  │  │ const  │  │ const  │  │   const    │
 │ var    │  │ var    │  │ var    │  │   var      │
 │ init() │  │ init() │  │ init() │  │   init()   │
 └───┬────┘  └───┬────┘  └───┬────┘  │   main()   │
     │           │           │       └─────┬──────┘
     ▼           ▼           ▼             ▼
 ────●───────────●───────────●─────────────●──────▶ 時間
   最先                                  最後
```

### init 函數的用途

其實，init 函數的這些常用用途，與 init 函數在 Go package初始化過程中的次序密不可分。我們前面講過，Go package初始化時，init 函數的初始化次序在變數之後，這給了開發人員在 init 函數中對package變數進行進一步檢查與操作的機會。

#### 1. 重置package變數值和初始化

初始化 package 內部會用到的各種變數

#### 2. 在 init 函數中實現“註冊模式”

為了讓你更好地理解，首先我們來看一段使用 lib/pq package連接 PostgreSQL 資料庫的程式碼範例：

```go
import (
    "database/sql"
    _ "github.com/lib/pq"
)

func main() {
    db, err := sql.Open("postgres", "user=pqgotest dbname=pqgotest sslmode=verify-full")
    if (err != nil) {
        log.Fatal(err)
    }
    
    age := 21
    rows, err := db.Query("SELECT name FROM users WHERE age = $1", age)
    ...
}
```

這是一段“神奇”的程式碼，你可以看到範例程式碼是以空導入的方式導入 lib/pq package的，main 函數中沒有使用 pq package的任何變數、函數或方法，這樣就實現了對 PostgreSQL 資料庫的訪問。而這一切的奧秘，全在 pq package的 init 函數中：

```go
func init() {
    sql.Register("postgres", &Driver{})
}
```

這個奧秘就在，我們其實是利用了用空導入的方式導入 lib/pq package時產生的一個“副作用”，也就是 lib/pq package作為 main package的所需的Library，它的 init 函數會在 pq package初始化的時候得以執行。

這種通過在 init 函數中註冊自己的實現的模式，就有效降低了 Go package對外的直接暴露，尤其是package變數的暴露，從而避免了外部通過package變數對package狀態的改動。

另外，從標準庫 database/sql package的角度來看，這種“註冊模式”實質是一種工廠設計模式的實現，sql.Open 函數就是這個模式中的工廠方法，它根據外部傳入的驅動名稱“生產”出不同類別的資料庫instance。

現在我們了解了 init 函數的常見用途。init 函數之所以可以勝任這些工作，恰恰是因為它在 Go 應用初始化次序中的特殊“位次”，也就是 main 函數之前，常數和變數初始化之後。

### 總結

**main 函數的特徵**

* Go 應用的用戶層入口函數  
  **init 函數的特徵**
* 執行順位排在package內其他語法元素(var, const)的後面；
* 每個 init 函數在整個 Go 程式生命週期內僅會被執行一次；
* init 函數是順序執行的，只有當一個 init 函數執行完畢後，才會去執行下一個 init 函數。

大多 Go 程式都是併發程式，程式會啟動多個 Goroutine 併發執行程式邏輯，這裡你一定要注意主 Goroutine 的優雅退出，也就是main上的 Goroutine 要根據實際情況來決定，是否要等待其他子 Goroutine 做完清理收尾工作退出後再行退出。

更多Go語言相關的文章歡迎參考我的部落格: <https://kaichiachen.github.io/2023/12/27/golang/go_runtime_cycle/>

  
