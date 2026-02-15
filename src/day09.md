# Day9 - Interface: 為什麼nil接口不等於nil?

Interface 是 Go 這門靜態語言中具有「動靜兼備」特性的語法元素。它既展示了 Go 的強大表達能力，也經常讓初學者感到迷惑。為了釐清這些迷惑，本文將深入探討 Go 在 runtime 時是如何處理 Interface 的類型表現。

在我們揭開 Interface 的神秘面紗之前，先來理解其「動靜兼備」的特性究竟是什麼。

### 接口的靜態與動態特性

接口的靜態特性主要是指接口類型變數有其靜態類型，例如在 `var err error` 中，`err` 的靜態類型為 `error`。有了這樣的靜態類型，編譯器在編譯階段對所有接口類型變數的賦值進行類型檢查，確保賦值操作的右值實現了該接口的所有方法，否則會報錯：

```go
var err error = 1 // cannot use 1 (type int) as type error in assignment: int does not implement error (missing Error method)
```

接口的動態特性則顯示在接口類型變數在 runtime 中儲存了右值的真實類型，這種特性讓 Go 的接口變數具有類似動態語言的靈活性，如 Python 的 Duck Typing。這種類型的特性不是由其繼承關係決定的，而是由類型表現出來的行為決定的。例如：

```go
var err error
err = errors.New("error1")
fmt.Printf("%T\n", err)  // *errors.errorString
```

在這裡，我們通過 `errors.New` 創建了一個錯誤值並賦值給 `error` 類型的接口變數 `err`，通過 `fmt.Printf` 輸出了 `err` 的動態類型為 `*errors.errorString`。

這種「動靜兼備」的特性具體好處包括：

* 程序在 runtime 可以將接口類型變數賦值為不同的動態類型，增加了語言的靈活性。
* 接口的動態特性還保障了使用時的安全性，如編譯器能在編譯期捕捉到錯誤的賦值。

### nil error 值不等於 nil 的疑惑

這裡我們通過一個範例來探討常見的疑惑 —— 「nil 的 error 值不等於 nil」。讓我們看下這段程式碼：

```go
type MyError struct {
    error
}

var ErrBad = MyError{
    error: errors.New("bad things happened"),
}

func bad() bool {
    return false
}

func returnsError() error {
    var p *MyError = nil
    if bad() {
        p = &ErrBad
    }
    return p
}

func main() {
    err := returnsError()
    if err != nil {
        fmt.Printf("error occur: %+v\n", err)
        return
    }
    fmt.Println("ok")
}
```

在這個例子中，`returnsError` 函數返回的是 `error` 接口類型的變數 `err`，即使其動態類型 `p` 為 `nil`，`err` 與 `nil` 進行比較時並不相等，這是因為接口變數的內部表示除了數據pointer，還包括了類型信息。

### 接口類型變數的內部表示詳解

在 Go 語言中，接口類型變數的內部表示是理解其動態行為的關鍵。接口類型變數在內部主要通過兩種結構表達：`eface` 和 `iface`。這兩種結構分別對應於不帶有Method的空接口和帶Method的接口。

#### 1. `eface` 結構：空接口的表示

* **用途**：用於表示空接口 `interface{}`，這種接口不包含任何方法。
* **結構**：

  ```go
  type eface struct {
      _type *_type         // 表示動態類型的 `_type` 結構的pointer
      data  unsafe.Pointer // 實際數據的pointer
  }
  ```
* **功能**：`_type` 指向一個描述數據的動態類型的結構，而 `data` pointer直接指向實際的數據。這個簡單的表示方式使得空接口能夠儲存任何類型的值。

#### 2. `iface` 結構：帶方法的接口表示

* **用途**：用於表示包含方法的接口。
* **結構**：

  ```go
  type iface struct {
      tab  *itab           // 指向 `itab` 結構，包含類型信息和方法pointer
      data unsafe.Pointer  // 指向實際數據的pointer
  }
  ```
* **功能**：`tab` 指向的 `itab` 結構不僅保存了接口的動態類型信息，還包括了指向實現接口方法的函數pointer。這使得 Go 在執行時能夠通過接口調用具體類型的方法。

#### 內部實現的影響

* **比較行為**：當兩個接口變數進行比較時，Go 不僅比較 `data` pointer，還要比較他們的類型pointer（`eface._type` 或 `iface.tab._type`）。只有當這兩部分都相同時，兩個接口變數才視為相等。

再回到開頭的問題，是不是已經豁然開朗了？`returnsError` 函數返回的 `error` 接口類型變數 `err` 的數據pointer雖然為空，但其類型（`iface.tab`）不為空，而是 `*MyError` 對應的類型，這樣 `err` 與 `nil`（0x0, 0x0）相比自然不相等，這就是問題的答案。

### 總結

* **接口的靜態特性**：保障了類型的正確性和編譯期的類型檢查。
* **接口的動態特性**：提供了如動態語言般的靈活使用方式。
* **nil 的 error 值問題**：揭示了接口類型變數的內部複雜性，說明了為何 `nil` 的 `error` 值在某些情況下不等於 `nil`。  
  通過深入理解這些內部結構和行為，Go 開發者可以更好地掌握接口的使用規則，避免一些常見的錯誤，尤其是在處理接口和 nil 值比較時的特殊行為。

更多Go語言相關的文章歡迎參考我的部落格: <https://kaichiachen.github.io/2024/02/08/golang/interface_nil/>

  
