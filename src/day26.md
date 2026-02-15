# Day26 - 用Go來寫一個高併行HTTP壓力測試工具

你可能聽說過很多壓力測試工具，比如Apache JMeter、Locust、Siege等等，但有沒有想過用Go語言自己寫一個簡單又強大的高併行HTTP壓力測試工具呢？Go語言的併發特性，特別是Goroutine的輕量級併發機制，讓它成為編寫高效壓力測試工具的絕佳選擇。今天，我們就來看看如何用Go寫一個自己的HTTP壓力測試工具，並瞭解Goroutine如何在超多I/O併行場景中大放異彩。

## 1. Go語言的併發特性

Go語言最強大的武器之一就是它的併發處理能力，特別是Goroutine。與傳統的Thread（線程）相比，Goroutine更加輕量，每個Goroutine僅佔用幾KB的記憶體，使我們可以輕鬆創建數千甚至數萬個Goroutine來處理高併發任務。這使得Go非常適合處理大量I/O操作，比如HTTP請求、網路通訊等。

### Goroutine的特點

* **輕量級**：每個Goroutine僅佔用極少的資源。
* **易用性**：使用`go`關鍵字即可創建一個新的Goroutine，沒有複雜的配置。
* **併發性**：天生支援多核處理器，能夠充分利用現代CPU的性能。

## 2. 設計HTTP壓力測試工具的思路

我們的目標是創建一個簡單的HTTP壓力測試工具，它能夠：

* 發送大量並行HTTP請求到指定的URL。
* 記錄每個請求的響應時間和狀態碼。
* 簡單的統計結果顯示，如成功次數、失敗次數、平均響應時間等。

### 工具的主要流程

1. 接收目標URL、請求次數和併發數量等參數。
2. 使用Goroutine發送HTTP請求。
3. 收集響應數據並進行統計。

## 3. 實作範例

接下來，我們來看看如何用Go實現這個工具：

```go
package main

import (
    "fmt"
    "net/http"
    "sync"
    "time"
)

// 壓力測試參數
const (
    targetURL      = "http://example.com" // 測試目標URL
    totalRequests  = 1000                 // 總請求數
    concurrency    = 100                  // 併發數量
)

// 統計數據
var (
    successCount int
    failureCount int
    totalTime    time.Duration
    mutex        sync.Mutex
)

func main() {
    // 計時開始
    start := time.Now()

    // 使用WaitGroup等待所有請求完成
    var wg sync.WaitGroup
    requestChan := make(chan struct{}, concurrency) // 控制併發數量

    for i := 0; i < totalRequests; i++ {
        requestChan <- struct{}{} // 將請求加入併發控制管道
        wg.Add(1)
        go func() {
            defer wg.Done()
            sendRequest()
            <-requestChan // 完成請求後釋放chain
        }()
    }

    wg.Wait()
    elapsed := time.Since(start)

    // 輸出結果
    fmt.Printf("Total Requests: %d\n", totalRequests)
    fmt.Printf("Success Count: %d\n", successCount)
    fmt.Printf("Failure Count: %d\n", failureCount)
    fmt.Printf("Total Time: %v\n", elapsed)
    fmt.Printf("Average Time per Request: %v\n", totalTime/time.Duration(successCount+failureCount))
}

// 發送HTTP請求的函式
func sendRequest() {
    start := time.Now()
    resp, err := http.Get(targetURL)
    duration := time.Since(start)

    // 保護統計數據
    mutex.Lock()
    totalTime += duration
    if err != nil || resp.StatusCode != http.StatusOK {
        failureCount++
    } else {
        successCount++
    }
    mutex.Unlock()
}
```

### 範例程式說明：

1. **併發控制**：使用Go的`sync.WaitGroup`來等待所有的請求完成，並使用channel來控制併發數量，避免Goroutine數量過多導致記憶體不足。
2. **數據保護**：使用`sync.Mutex`來保護統計變數的安全，防止數據競爭。
3. **性能統計**：收集每個請求的執行時間並統計成功與失敗次數。

## 4. Go的優勢

透過這個例子，我們可以看出Go語言在高併發場景中的優勢：

* **超高併發**：Goroutine幾乎是零成本的輕量級併發單位，能輕鬆處理大量HTTP請求。
* **簡潔的語法**：Go語言的語法簡單直接，開發這樣的工具幾乎沒有任何語言障礙。
* **穩定性**：Go語言的編譯器和標準庫極為穩定，讓壓力測試工具在長時間運行中依然保持穩定。

## 結論

Go語言以其高效的併發模型，讓我們能輕鬆寫出高性能的HTTP壓力測試工具。無論是系統測試、API性能測試，還是日常開發，Go的輕量併發特性都能大大提高我們的開發效率。

  
