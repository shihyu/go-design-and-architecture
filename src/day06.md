# Day6 - Method: Method與如何選擇Receiver類型

和多數程式語言一樣，在 Go 語言中，我們需要考慮如何設計方法。由於在 Go 語言中，方法本質上就是函數，所以我們之前講解的關於函數設計的內容同樣適用於方法，例如錯誤處理設計、針對異常的處理策略、使用 `defer` 提升簡潔性，等等。

Receiver（Receiver）也是設計方法時需要考量的一點。Receiver定義了一個Method所屬的類型，有點類似於其他語言中的 class 或 object 的概念，如下所示：

```go
func (t T) M1() {}
func (t *T) M2() {}
```

上面範例中的 `t` 就是Receiver，而 `M1` 和 `M2` 是Receiver定義的Method。`M1` 方法代表Receiver類型為 `T` 的Method，而 `M2` 方法則代表Receiver類型為 `*T` 的Method。下面我們來看看不同的Receiver類型對 `M1` 和 `M2` 的影響。

### 當Receiver類型為 `T` 時:

代表 `T` 類型Object的Receiver參數以pass by value傳遞到 `M1` 方法中，實際上是 `T` 類型Object的副本，因此 `M1` 方法中對副本的任何修改操作，都不會影響原 `T` 類型Object。

### 當Receiver類型為 `*T` 時:

代表 `*T` 類型實例的Receiver參數以pass by reference傳遞到 `M2` 方法中，實際上是 `T` 類型Object的地址，因此 `M2` 方法可以通過該address對原 `T` 類型Object進行修改操作。

我們來看看一個更直觀的範例，來證明上述分析結果，並觀察 Go 方法選擇不同的Receiver類型對原類型Object的影響：

```go
package main

type T struct {
    a int
}

func (t T) M1() {
    t.a = 10
}

func (t *T) M2() {
    t.a = 11
}

func main() {
    var t T
    println(t.a) // 0

    t.M1()
    println(t.a) // 0

    p := &t
    p.M2()
    println(t.a) // 11
}
```

在這個範例中，我們為類型 `T` 定義了兩個Method - `M1` 和 `M2`，其中 `M1` 的Receiver類型為 `T`，而 `M2` 的Receiver類型為 `*T`。`M1` 和 `M2` 方法都通過Receiver參數 `t` 修改了 `t` 的attribute - `a`。

```
 Value Receiver vs Pointer Receiver 記憶體示意圖：

 ═══════════════════════════════════════════════════════
 var t T  (t.a = 0)

 Stack 記憶體：
 ┌──────────────┐
 │  t (T)       │
 │  a: 0        │  ← 原始變數
 └──────────────┘

 ═══════════════════════════════════════════════════════
 呼叫 t.M1()  (Value Receiver → pass by value)

 ┌──────────────┐        ┌──────────────┐
 │  t (原始)    │        │  t (副本)    │
 │  a: 0        │  複製▶  │  a: 0 → 10  │  ← M1 修改的是副本
 └──────────────┘        └──────────────┘
 t.a 仍然 = 0  ✓         副本被丟棄

 ═══════════════════════════════════════════════════════
 呼叫 p.M2()  (Pointer Receiver → pass by reference)

 ┌──────────────┐        ┌──────────────┐
 │  p (*T)      │───────▶│  t (原始)    │
 │  (地址)      │  指向   │  a: 0 → 11  │  ← M2 直接修改原始
 └──────────────┘        └──────────────┘
                         t.a = 11  ✓
```

了解了不同Receiver類型對 Go 方法的影響後，我們可以總結一下，日常寫程式中選擇Receiver參數類型時可以參考的原則：

1. **如果 Go 方法需要修改Receiver代表的類型Object，並將修改反映到原類型Object上，應選擇 `*T` 作為Receiver參數類型。**
2. **如果Receiver的 size 太大，應選擇 `*T` 作為Receiver參數類型，以避免複製大Object。**
3. **如果需要縮小外部接觸面，盡量少暴露可以修改內部型態的方法，可以考慮使用 `T` 作為Receiver。**
4. **T 類型是否需要實作某個interface**：  
   如果 `T` 類型需要實作某個interface，那我們就要使用 `T` 作為Receiver參數類型，以滿足interface類型方法集合中的所有Method。如果 `T` 不需要實作某個interface，但 `*T` 需要實作該interface，則 `*T` 的方法集合包含 `T` 的方法集合。因此，我們在確定 Go 方法的Receiver類型時，可以參考上述原則。

這裡可以解釋一下什麼是**方法集合**，我們先通過一個範例來直觀了解為什麼要有方法集合，以及它主要用來解決什麼問題：

```go
type Interface interface {
    M1()
    M2()
}

type T struct{}

func (t T) M1() {}
func (t *T) M2() {}

func main() {
    var t T
    var pt *T
    var i Interface

    i = pt
    i = t // cannot use t (type T) as type Interface in assignment: T does not implement Interface (M2 method has pointer receiver)
}
```

運行這個範例程式，我們在 `i = t` 這一行會得到 Go 編譯器的錯誤提示，Go 編譯器提示我們：`T` 沒有實作 `Interface` 類型方法列表中的 `M2`，因此類型 `T` 的Object `t` 不能賦值給 `Interface` 變量。

Interface類型相對特殊，它只會列出代表interface的方法列表，不會具體定義某個方法，類似C++的virtual和Java的interface，其方法集合就是**它的方法列表中的所有方法**，我們可以一目了然地看到。因此，我們下面重點講解的是非interface類型的方法集合。

為了方便查看一個非interface類型的方法集合，我這裡提供了一個函數 `dumpMethodSet`，用於輸出一個非interface類型的方法集合：

```go
func dumpMethodSet(i interface{}) {
    dynTyp := reflect.TypeOf(i)

    if dynTyp == nil {
        fmt.Printf("there is no dynamic type\n")
        return
    }

    n := dynTyp.NumMethod()
    if n == 0 {
        fmt.Printf("%s's method set is empty!\n", dynTyp)
        return
    }

    fmt.Printf("%s's method set:\n", dynTyp)
    for j := 0; j < n; j++ {
        fmt.Println("-", dynTyp.Method(j).Name)
    }
    fmt.Printf("\n")
}
```

下面我們利用這個函數，試著輸出一下 Go 原生類型以及自定義類型的方法集合：

```go
type T struct{}

func (T) M1() {}
func (T) M2() {}

func (*T) M3() {}
func (*T) M4() {}

func main() {
    var n int
    dumpMethodSet(n)
    dumpMethodSet(&n)

    var t T
    dumpMethodSet(t)
    dumpMethodSet(&t)
}
```

我們得到如下結果：

```go
int's method set is empty!
*int's method set is empty!
main.T's method set:
- M1
- M2

*main.T's method set:
- M1
- M2
- M3
- M4
```

我們看到以 `int` 和 `*int` 為代表的 Go 原生類型，由於沒有定義Method，所以它們的方法集合都是空的。自定義類型 `T` 定義了方法 `M1` 和 `M2`，因此其方法集合包含了 `M1` 和 `M2`，也符合我們的預期。然而，`*T` 的方法集合中除了預期的 `M3` 和 `M4` 外，還包含了類型 `T` 的方法 `M1` 和 `M2`！

這是因為，Go 語言規定，**`*T` 類型的方法集合包含所有以 `*T` 為Receiver參數類型的方法，以及所有以 `T` 為Receiver參數類型的方法**。這就是為何 `*T` 類型的方法集合包含四個方法的原因，以及第一個範例會報錯的原因。

```
 方法集合（Method Set）示意圖：

 T 類型的方法集合：          *T 類型的方法集合：
 ┌─────────────────┐       ┌─────────────────────┐
 │  T 的方法集合    │       │  *T 的方法集合       │
 │                 │       │                     │
 │  ● M1()  (T)   │       │  ● M1()  (T)  ─┐   │
 │  ● M2()  (T)   │       │  ● M2()  (T)   │繼承│
 │                 │       │  ● M3()  (*T)  │   │
 └─────────────────┘       │  ● M4()  (*T) ─┘   │
                           └─────────────────────┘

 Interface 滿足性檢查：
 ┌──────────────────────────────────────────────────┐
 │  type Interface interface { M1(); M2() }         │
 │                                                  │
 │  T  的方法集合 = {M1, M2}     ✓ 滿足 Interface   │
 │  *T 的方法集合 = {M1,M2,M3,M4} ✓ 滿足 Interface │
 │                                                  │
 │  type Interface2 interface { M1(); M2(); M3() }  │
 │                                                  │
 │  T  的方法集合 = {M1, M2}     ✗ 缺少 M3         │
 │  *T 的方法集合 = {M1,M2,M3,M4} ✓ 滿足 Interface2│
 └──────────────────────────────────────────────────┘
```

### 總結

* **Receiver類型為 `T` 時**：方法Receiver是一個值，任何修改都不會影響原Object。
* **Receiver類型為 `*T` 時**：方法Receiver是一個pointer，可以修改原Object。
* **選擇Receiver類型的原則**：
  + 如果需要修改原Object，使用 `*T`。
  + 如果Receiver的 size 太大，使用 `*T`。
  + 如果希望縮小外部接觸面，減少Method, Attribute暴露，使用 `T`。
  + 根據是否需要實作Interface來決定使用 `T` 還是 `*T`。

這些原則可以幫助你在 Go 程式設計中做出更好的Method設計決策。

  
