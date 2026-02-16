# Day7 - Method: 怎麼用變數模擬繼承

### Go 語言的繼承與組合

Go 語言沒有像 C++/Java 等語言可以透過 `extend` 關鍵字來繼承某個 class。因此，Go class的所有方法都是自己顯式實現的。讓我們舉個例子，自定義class `T` 有兩個methods `M1` 和 `M2`，如果 `T` 是一個獨立的自定義class，那我們在聲明class `T` 的 Go 包原始碼文件中一定可以找到其所有method的實現程式碼，比如：

```go
func (T) M1() {...}
func (T) M2() {...}
```

在 Go 語言中，如果我們真的想模擬繼承的效果，可以透過 Go 語言設計思想的“組合”來實現。具體的 Go 語言設計思想可以參考我的這篇文章 - [Go語言的設計哲學](../../../../../2023/12/12/golang/design_philosophy/)

這種“繼承”是通過 Go 語言的類型嵌入（Type Embedding）來實現的。因此，這篇文章將帶你了解這種語法，看看我們如何通過這種語法來實現對嵌入類型的方法的“繼承”，同時也搞清楚這種方式對新定義的類型的方法集合的影響。首先，我們來學習一下什麼是類型嵌入。

### 類型嵌入

類型嵌入指的是在一個類型的定義中嵌入了其他類型。Go 語言支持兩種類型嵌入，分別是 `interface` 類型的類型嵌入和 `struct` 類型的類型嵌入。

#### interface 類型的類型嵌入

interface類型聲明了由一個方法集合代表的interface，比如下面interface類型 `E`：

```go
type E interface {
    M1()
    M2()
}
```

我們再定義另外一個interface類型 `I`，它的方法集合中包含了三個方法 `M1`、`M2` 和 `M3`，如下所示：

```go
type I interface {
    M1()
    M2()
    M3()
}
```

我們看到interface類型 `I` 方法集合中的 `M1` 和 `M2`，與接口類型 `E` 的方法集合中的方法完全相同。在這種情況下，我們可以用interface類型 `E` 替代上面interface類型 `I` 定義中的 `M1` 和 `M2`，如下所示：

```go
type I interface {
    E
    M3()
}
```

這種在一個interface類型（`I`）定義中嵌入另外一個interface類型（`E`）的方式，就是我們說的interface類型的**類型嵌入**。

#### struct 類型的類型嵌入

`struct` 類型的類型嵌入就要更複雜一些了，以下是 Go struct類型的“完全體”：

```go
type T1 int
type t2 struct{
    n int
    m int
}

type I interface {
    M1()
}

type S1 struct {
    T1
    *t2
    I            
    a int
    b string
}
```

我們看到，struct `S1` 定義中有三個“非常規形式”的attribute，分別是 `T1`、`t2` 和 `I`。這三個struct裡的attribute究竟代表的是什麼呢？是名字還是類型呢？

這裡我直接告訴你答案：它們既代表attribute的名字，也代表attribute的類型。我們分別以這三個attribute為例，說明一下它們的具體含義：

* `T1` 表示attribute名為 `T1`，它的類型為自定義類型 `T1`；
* `t2` 表示attribute名為 `t2`，它的類型為自定義struct類型 `t2` 的pointer類型；
* `I` 表示attribute名為 `I`，它的類型為interface類型 `I`。

```go
type MyInt int

func (n *MyInt) Add(m int) {
    *n = *n + MyInt(m)
}

type t struct {
    a int
    b int
}

type S struct {
    *MyInt
    t
    io.Reader
    s string
    n int
}

func main() {
    m := MyInt(17)
    r := strings.NewReader("hello, go")
    s := S{
        MyInt: &m,
        t: t{
            a: 1,
            b: 2,
        },
        Reader: r,
        s:      "demo",
    }

    var sl = make([]byte, len("hello, go"))
    s.Read(sl)              // 等同於 s.Reader.Read(sl)
    fmt.Println(string(sl)) // 輸出：hello, go
    s.Add(5) 
    fmt.Println(*(s.MyInt)) // 輸出：22
}
```

看到這段程式碼，你可能會問：類型 `S` 沒有定義 `Read` 方法和 `Add` 方法啊，這樣寫不會導致 Go 編譯器報錯嗎？如果你有這個疑問，可以暫停一下，先用你手上的 Go 編譯器編譯運行一下這段程式碼看看。

是不是很驚喜？這段程序不但沒有引發編譯器報錯，還可以正常運行並輸出正確的結果！

這段程式碼似乎在告訴我們：`Read` 方法和 `Add` 方法就是類型 `S` 方法集合中的方法。其實，這兩個方法來自於struct類型 `S` 的兩個嵌入attribute `Reader` 和 `MyInt`。struct類型 `S` “繼承”了 `Reader` attribute的方法 `Read` 的實現，也“繼承”了 `*MyInt` 的 `Add` 方法的實現。注意，我這裡的“繼承”用了引號，說明這並不是真正的繼承，而是 Go 語言的一種“障眼法”。

這種“障眼法”的工作機制是這樣的：當我們通過struct類型 `S` 的變量 `s` 呼叫 `Read` 方法時，Go 發現struct類型 `S` 自身並沒有定義 `Read` 方法，於是 Go 會查看 `S` 的嵌入attribute對應的類型是否定義了 `Read` 方法。這時，`Reader` attribute就被找了出來，之後 `s.Read` 的呼叫就被轉換為 `s.Reader.Read` 的呼叫。

這樣一來，嵌入attribute `Reader` 的 `Read` 方法就被提升為 `S` 的方法，放入了類型 `S` 的方法集合。同理，`*MyInt` 的 `Add` 方法也被提升為 `S` 的方法而放入 `S` 的方法集合。從外部來看，這種嵌入attribute的方法的提升就給了我們一種struct類型 `S` "繼承"了 `io.Reader` 類型 `Read` 方法的實現，以及 `*MyInt` 類型 `Add` 方法的實現的錯覺。嵌入attribute的使用的確可以幫我們在 Go 中實現方法的"繼承"。

```
 Type Embedding（類型嵌入）記憶體佈局示意圖：

 type S struct {
     *MyInt          // 嵌入 pointer 類型
     t               // 嵌入 struct 類型
     io.Reader       // 嵌入 interface 類型
     s string
     n int
 }

 S 的記憶體結構：
 ┌─────────────────────────────────────────────────┐
 │                  S struct                        │
 ├──────────────┬──────────────────────────────────┤
 │  MyInt (*MyInt)  │  pointer ──▶ MyInt(17)       │
 ├──────────────┼──────────────────────────────────┤
 │  t (struct t)    │  ┌─────┬─────┐               │
 │                  │  │ a:1 │ b:2 │  (內嵌展開)   │
 │                  │  └─────┴─────┘               │
 ├──────────────┼──────────────────────────────────┤
 │  Reader          │  ┌──────┬──────┐             │
 │  (io.Reader)     │  │ itab │ data │──▶ Reader   │
 │                  │  └──────┴──────┘  實體       │
 ├──────────────┼──────────────────────────────────┤
 │  s (string)      │  ┌─────┬─────┐               │
 │                  │  │ ptr │ len │──▶ "demo"     │
 │                  │  └─────┴─────┘               │
 ├──────────────┼──────────────────────────────────┤
 │  n (int)         │  0                           │
 └──────────────┴──────────────────────────────────┘

 方法查找機制（Method Resolution）：
 ┌────────────────────────────────────────────┐
 │  s.Read(sl)                                │
 │    │                                       │
 │    ▼                                       │
 │  S 有 Read 嗎？ → 沒有                     │
 │    │                                       │
 │    ▼                                       │
 │  檢查嵌入欄位 → Reader 有 Read() → 找到！  │
 │    │                                       │
 │    ▼                                       │
 │  轉換為 s.Reader.Read(sl)                  │
 └────────────────────────────────────────────┘
```

### 多重繼承

不過有一種情況需要注意，那就是當struct嵌入的多個接口類型的方法集合存在交集時，你要小心編譯器可能會出現的錯誤提示。

Go 1.14 版本解決了嵌入接口類型的方法集合有交集的情況，僅限於interface類型中嵌入interface類型。這裡我們討論的是在struct類型中嵌入方法集合有交集的interface類型。

如果struct自身實現了該方法，Go 就會優先使用struct自己實現的方法。如果沒有實現，那麼 Go 就會查找struct中的嵌入attribute的方法集合中是否包含這個方法。如果多個嵌入attribute的方法集合中都包含這個方法，那麼我們就說方法集合存在交集。這時，Go 編譯器就會因無法確定究竟使用哪個方法而報錯，所以 Go 會在編譯階段就限制多重繼承的問題。

### 總結

* Go 語言沒有傳統意義上的繼承，所有方法都必須顯式實現。
* 可以通過類型嵌入（Type Embedding）來模擬繼承的效果，將嵌入類型的方法提升為新類型的方法。
* `interface` 和 `struct` 都支持類型嵌入。
* 使用嵌入attribute到class的方法提升機制，可以實現方法的“繼承”，但需注意多重繼承的衝突問題。

這些技巧和知識將幫助你在 Go 語言中有效地使用組合和嵌入，編寫出更靈活且可維護的程式碼。

更多Go語言相關的文章，歡迎追蹤我的部落格: <https://kaichiachen.github.io/2024/01/23/golang/go_inherit/>

  
