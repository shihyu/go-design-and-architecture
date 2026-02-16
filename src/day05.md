# Day5 - Defer的妙用 - 跟蹤函數調用鏈

在 Go 語言中，defer 關鍵字是一個強大而且實用的工具，能讓你更優雅地處理資源釋放和一些在函數結束前必須執行的清理工作。本文將通過生動的比喻和簡單的示例，幫助你理解 defer 的使用方法及其重要性。

### defer 是什麼？

簡單來說，defer 是用來延遲執行一段程式碼，直到所在的函數即將返回時才執行。就像你在餐廳吃飯，最後才會付錢一樣，defer 讓你在程式的適當位置安排程式碼，但等到適當的時候才執行。

### 基本語法

使用 defer 非常簡單，只需在你要延遲執行的函數或表達式前面加上 defer 關鍵字即可：

```go
package main

import "fmt"

func main() {
    fmt.Println("Start")
    defer fmt.Println("End")
    fmt.Println("Doing something...")
}
```

輸出結果將是：

```go
Start
Doing something...
End
```

可以看到，defer 語句會在 main 函數即將返回前執行，無論函數中間發生了什麼。

```
 defer 的執行機制 - 堆疊(Stack)結構：

 函數執行過程中，defer 語句被「壓入」一個 LIFO 堆疊：

  執行順序          defer 堆疊（後進先出）
  ─────────        ─────────────────────
  Println("Start")
                    ┌─────────────────┐
  defer "End"  ──▶  │  "End"          │  ← 第一個壓入
                    └─────────────────┘
  Println("Doing")

  函數即將返回時，從堆疊頂部彈出執行：

                    ┌─────────────────┐
  彈出執行 ◀──────  │  "End"          │  ← 彈出並執行
                    └─────────────────┘

  輸出：Start → Doing something... → End
```

### 典型使用場景

defer 最常見的使用場景是資源釋放，例如文件操作、網絡連接、資料庫連接等。以下是一個簡單的文件操作示例：

```go
package main

import (
    "fmt"
    "os"
)

func main() {
    file, err := os.Open("example.txt")
    if err != nil {
        fmt.Println("Error opening file:", err)
        return
    }
    defer file.Close() // 延遲關閉文件

    // 假設這裡有一些讀取文件的操作
    fmt.Println("Reading file...")
}
```

在這個例子中，file.Close() 被延遲執行，確保無論 main 函數如何返回，文件都會被正確關閉，避免資源洩漏。

### 延遲執行的捕捉

你可以用 defer 來捕捉並處理異常，確保在異常情況下也能執行必要的清理工作：

```go
package main

import "fmt"

func main() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered from panic:", r)
        }
    }()

    fmt.Println("Starting")
    panic("Something went wrong")
    fmt.Println("This will not be printed")
}
```

在這個例子中，recover 函數會捕捉到 panic，並且 defer 語句保證了即使發生異常，也能執行清理工作。

### 多個 defer 語句

如果你在一個函數中使用多個 defer 語句，它們會按照後進先出的順序執行（LIFO）。這就像你堆疊盤子，最後放上的盤子會最先被拿走。

```
 多個 defer 的堆疊示意圖：

  程式碼執行順序           defer 堆疊變化
  ────────────           ────────────────

  defer "First"          ┌──────────┐
                         │ "First"  │  ← 底部
                         └──────────┘

  defer "Second"         ┌──────────┐
                         │ "Second" │  ← 頂部
                         ├──────────┤
                         │ "First"  │  ← 底部
                         └──────────┘

  defer "Third"          ┌──────────┐
                         │ "Third"  │  ← 頂部（最後壓入）
                         ├──────────┤
                         │ "Second" │
                         ├──────────┤
                         │ "First"  │  ← 底部（最先壓入）
                         └──────────┘

  函數返回時，依序彈出：
  Third → Second → First （後進先出 LIFO）
```

```go
package main

import "fmt"

func main() {
    defer fmt.Println("First")
    defer fmt.Println("Second")
    defer fmt.Println("Third")
    fmt.Println("Doing something...")
}
```

輸出結果將是：

```go
Doing something...
Third
Second
First
```

### 高效使用 defer

雖然 defer 是一個非常有用的工具，但它並非免費的。每次調用 defer 都會有一些額外的開銷，尤其是在高頻率調用的場景中。例如，在一個非常深的遞迴函數中頻繁使用 defer 可能會帶來性能問題。可以考慮在這種情況下使用顯式的資源管理方式來代替 defer。  
以下是defer 的最佳實踐

* 資源管理：defer 非常適合用來管理資源的釋放，如文件、網路socket連接等。
* 錯誤處理：在需要進行清理操作的錯誤處理中使用 defer 確保資源被正確釋放。
* 讀取程式碼順序：在程式碼靠近資源分配的地方使用 defer，提高程式碼的可讀性和可維護性。

### 總結

defer 是 Go 語言中一個簡單但強大的工具，可以讓你的程式更加優雅和可靠。無論是資源釋放、捕捉異常還是簡單的清理工作，defer 都能幫助你寫出更加整潔和易於維護的程式碼。

記得，像處理餐廳賬單一樣，把 defer 放在適當的位置，等到最後才執行，就能讓你的程式運行更加順暢！

  
