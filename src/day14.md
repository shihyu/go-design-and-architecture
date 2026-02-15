# Day14 - 更低級別的Go的變數同步技巧 - sync

Go 語言的併發模型中，一句由 Rob Pike 所提的經典名言非常關鍵：「不要透過共享記憶體來通信，而是透過通信來共享記憶體（Don’t communicate by sharing memory, share memory by communicating）」。這不僅是一句格言，更深刻地影響了 Go 語言在併發編程中的設計哲學，主要是倡導使用 channel 作為 Goroutine 之間的溝通橋樑。

儘管 Go 語言在設計上鼓勵使用通信而非共享記憶體的方法來實現併發，但它並未完全排斥傳統的基於共享記憶體的併發模型。例如，Go 的標準庫中的 sync 包就提供了多種低級的同步方法，包括互斥鎖（sync.Mutex）、讀寫鎖（sync.RWMutex）、條件變數（sync.Cond）以及原子操作等工具，這些都是在特定情況下仍然非常有用的工具。

### sync 包低級同步的應用場景

* 在需要高效率的臨界區同步時，低級同步方法如互斥鎖比 channel 更為適合，因為它們的執行成本更低。
* 當需要在多個 Goroutine 之間同步訪問某個結構體的內部狀態，但又不希望轉移該結構體對象的所有權時，使用 sync 包的同步方法是一個好選擇。

在實際應用中，使用 sync 包的互斥鎖和讀寫鎖需要注意一些事項，例如避免對含有鎖的結構進行複製，因為這會導致鎖的行為出現非預期的錯誤。此外，使用互斥鎖時應盡量縮短鎖定時間，以減少對程序效率的影響，並且必須記得在適當時候釋放鎖，避免死鎖的發生。

### 使用例子和注意事項

互斥鎖（Mutex）和讀寫鎖（RWMutex）的基本用法如下：

```go
var mu sync.Mutex
mu.Lock()   // 加鎖
doSomething()
mu.Unlock() // 解鎖
```

這段程式碼展示了一個 Goroutine 如何獨占資源進行操作。當使用互斥鎖時，其他試圖執行 Lock 操作的 Goroutine 將會被阻塞，直到鎖被釋放。

讀寫鎖則允許多個讀指令同時進行，但寫指令會獨占鎖：

```go
var rwmu sync.RWMutex
rwmu.RLock()   // 加讀鎖
readSomething()
rwmu.RUnlock() // 解讀鎖
rwmu.Lock()    // 加寫鎖
changeSomething()
rwmu.Unlock()  // 解寫鎖
```

這種鎖的設計允許多個讀者同時讀取數據而不被阻塞，只有當需要修改數據時，寫鎖才會阻止其他讀取或寫入操作，保證數據的一致性。

### sync.Cond：條件變數

`sync.Cond` 是 Go 語言中實現條件變數的一種方式，主要用於管理那些需要等待某個特定條件成立的 Goroutines。這個概念可以類比於百米賽跑中運動員等待發令槍的情境。當條件變數所等待的條件被觸發時，原本處於等待狀態的 Goroutines 會接收到通知，繼續他們的任務。

使用條件變數的好處在於，它能有效避免 Goroutines 進行無意義的連續輪詢，這種輪詢不僅效率低，還會浪費大量系統資源。以下是一個示例，展示如何使用 `sync.Cond` 達到與無緩衝 channel 相似的效果：

```go
type signal struct{}

var ready bool

func worker(i int) {
  fmt.Printf("worker %d: is working...\n", i)
  time.Sleep(1 * time.Second)
  fmt.Printf("worker %d: works done\n", i)
}

func spawnGroup(f func(i int), num int, groupSignal *sync.Cond) <-chan signal {
  c := make(chan signal)
  var wg sync.WaitGroup

  for i := 0; i < num; i++ {
    wg.Add(1)
    go func(i int) {
      groupSignal.L.Lock()
      while !ready {
        groupSignal.Wait()
      }
      groupSignal.L.Unlock()
      fmt.Printf("worker %d: start to work...\n", i)
      f(i)
      wg.Done()
    }(i + 1)
  }

  go func() {
    wg.Wait()
    c <- signal(struct{}{})
  }()
  return c
}

func main() {
  fmt.Println("start a group of workers...")
  groupSignal := sync.NewCond(&sync.Mutex{})
  c := spawnGroup(worker, 5, groupSignal)

  time.Sleep(5 * time.Second) // 模擬 ready 前的準備工作
  fmt.Println("the group of workers start to work...")

  groupSignal.L.Lock()
  ready = true
  groupSignal.Broadcast()
  groupSignal.L.Unlock()

  <-c
  fmt.Println("the group of workers work done!")
}
```

### 原子操作（Atomic Operations）

`atomic` 包提供了原子操作的接口，這些操作相比普通的指令操作具有不可中斷的特性。舉例來說，一個簡單的整數++操作：

```go
var a int
a++
```

這樣的操作需要三條`指令`來完成：

* **LOAD**：將變數從內存加載到 CPU 寄存器；
* **ADD**：執行加法指令；
* **STORE**：將計算結果存回記憶體。  
  這三個步驟在執行過程中都有可能被中斷。然而，原子操作可以保證像一個完整的事務那樣一次性完成，不會被中斷。原子操作的實現依賴於底層硬體的支持，提供了一種比作業系統層面更低級的同步技術。`atomic` Lib為開發者封裝了這些底層指令，使得實現複雜的併發同步控制變得更加便捷。

### 總結

* sync 包提供了多種低級同步方法，包括互斥鎖、讀寫鎖和條件變數。
* 使用互斥鎖和讀寫鎖時要注意避免複製變數，並記得及時解鎖。
* sync.Cond 是條件變數的實現，用於避免 Goroutine 的連續輪詢。
* atomic Lib提供了底層的原子操作支持，適用於實現高級併發同步技術。

更多Go語言相關的文章，歡迎參閱我的部落格: <https://kaichiachen.github.io/2024/03/20/golang/go_happ_path/>

  
