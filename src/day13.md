# Day13 - channel中蘊含的大智慧

在探索Go語言的併發模型時，我們會接觸到兩個核心元素：Goroutine和channel。Goroutine是構成Go應用併發架構的基石，而channel則在整個併發模型中扮演了無可替代的角色。透過channel，不僅能夠實現Goroutine之間的數據通信，還能夠達到同步的效果。換句話說，只有當你能夠熟練地運用channel時，才能真正稱得上精通Go的併發設計。

因此，在本篇文章中，我們將詳細學習channel的基本語法及其在實際開發中的應用。

使用channel就像操作普通變數一樣簡單。我們可以定義一個channel類型的變數、對其進行賦值、把channel作為參數傳遞給其他函數或方法，甚至可以從函數或方法中return channel作為返回值，或者將一個channel直接發送至另一個channel。這些操作使得channel的應用變得靈活多變，大大提升了開發者在設計和實現併發功能時的便利和效率。

在談到 Go 語言的 channel 這個概念時，我們可以把它想象成一個用來傳遞數據的管道。這個管道能夠讓不同的 Goroutine 之間進行數據的交流。就像我們平常使用的其他複合資料結構 — array、struct、以及 map 一樣，channel 也是一個非常實用的複合資料結構。具體來說，當我們宣告一個 channel 的時候，必須指明這個 channel 能夠傳遞什麼類型的數據。舉例來說：

```go
var ch chan int
```

在這行程式碼中，`ch` 是一個可以傳遞 `int` 型別數據的 channel 變數。

如果在宣告的時候沒有初始化 channel，那它的預設值就是 `nil`，這表示這個 channel 是空的，還沒有準備好傳遞數據。和其他複合資料結構有所不同的是，初始化一個 channel 的方式是使用 Go 語言built-in的 `make` 函數。例如：

```go
ch1 := make(chan int)
ch2 := make(chan int, 5)
```

這段程式碼展示了如何初始化兩個 `int` 型別的 channel 變數：`ch1` 和 `ch2`。值得注意的是，`ch1` 和 `ch2` 的初始化方式有細微的差別。

`ch1` 是一個無緩衝的 channel，意味著每次傳遞數據時都需要有一個接收者同時準備接收數據，否則發送操作會被阻塞。而 `ch2` 則是一個有緩衝的 channel，它可以存儲最多 5 個 `int` 型別的數據，讓發送者在沒有立即接收者的情況下也能發送一定數量的數據而不被阻塞。

```
 Channel 內部記憶體結構示意圖（runtime.hchan）：

 ═══════════════════════════════════════════════════
 無緩衝 Channel: make(chan int)
 ┌──────────────────────────────────────────┐
 │  hchan struct                            │
 │  ┌───────────┬───────────────────────┐  │
 │  │ qcount: 0 │ 緩衝區元素數量         │  │
 │  │ dataqsiz:0│ 緩衝區大小 = 0        │  │
 │  │ buf: nil  │ 無緩衝區              │  │
 │  │ sendx: 0  │ 發送索引              │  │
 │  │ recvx: 0  │ 接收索引              │  │
 │  │ sendq     │ 等待發送的 goroutine  │  │
 │  │ recvq     │ 等待接收的 goroutine  │  │
 │  │ lock      │ 互斥鎖               │  │
 │  └───────────┴───────────────────────┘  │
 └──────────────────────────────────────────┘

 同步行為：發送和接收必須同時就緒
 Goroutine A          Channel          Goroutine B
 ┌────────┐                           ┌────────┐
 │ ch<-13 │──── 阻塞等待 ────────▶    │        │
 │        │     直到 B 準備好接收      │ <-ch   │
 │        │◀──── 資料直接傳遞 ────────│        │
 └────────┘                           └────────┘

 ═══════════════════════════════════════════════════
 有緩衝 Channel: make(chan int, 5)
 ┌──────────────────────────────────────────┐
 │  hchan struct                            │
 │  ┌───────────┬───────────────────────┐  │
 │  │ dataqsiz:5│ 緩衝區大小 = 5        │  │
 │  │ buf ──────┼──▶ 環形緩衝區 (Ring)  │  │
 │  └───────────┘   ┌──┬──┬──┬──┬──┐   │  │
 │                  │  │  │  │  │  │   │  │
 │   sendx ────▶    │ 3│ 7│  │  │  │   │  │
 │   recvx ────▶    │▲ │  │  │  │  │   │  │
 │                  └──┴──┴──┴──┴──┘   │  │
 │                  ↑              ↑    │  │
 │                recvx          sendx  │  │
 └──────────────────────────────────────────┘

 非同步行為：只要緩衝區未滿/非空就不阻塞
 Goroutine A          Buffer           Goroutine B
 ┌────────┐     ┌──┬──┬──┬──┬──┐     ┌────────┐
 │ ch<-1  │──▶  │ 1│  │  │  │  │     │        │
 │ ch<-2  │──▶  │ 1│ 2│  │  │  │     │        │
 │ ch<-3  │──▶  │ 1│ 2│ 3│  │  │     │ <-ch   │──▶ 得到 1
 │        │     │  │ 2│ 3│  │  │     │ <-ch   │──▶ 得到 2
 └────────┘     └──┴──┴──┴──┴──┘     └────────┘
```

接下來我們會進一步探討這兩種 channel 在數據發送和接收時的操作和特性，讓大家能更好地理解它們在實際開發中的應用。

### 發送與接收

在 Go 語言中，`<-` 操作符是用於 channel 的發送與接收。以下是幾個實際的例子來說明如何操作：

```go
ch1 <- 13    // 把int值13發送到一個無緩衝的channel變數ch1中
n := <-ch1   // 從同一個無緩衝channel變數ch1接收一個int數值，並存到變數n
ch2 <- 17    // 把int值17發送到一個有緩衝的channel變數ch2中
m := <-ch2   // 從這個有緩衝的channel變數ch2接收一個int數值，並存到變數m
```

使用 channel 時需注意，channel 主要是為了在不同的 Goroutine 之間進行通信。因此，大部分情況下，對 channel 的操作（讀取與寫入）最好放在不同的 Goroutine 中。

先來了解一下無緩衝 channel 的操作：

無緩衝 channel 沒有內置的緩衝區，所以其發送和接收操作是同步進行的。這意味著，若操作無緩衝 channel，發送和接收的 Goroutine 必須同時就緒，否則會造成 Goroutine 卡住/等待，形成死鎖。看看下面的例子：

```go
func main() {
    ch1 := make(chan int)
    ch1 <- 13 // 若沒有其他 Goroutine，將導致死鎖
    n := <-ch1
    println(n)
}
```

這段程式碼會引起 fatal error，因為它在單一的 Goroutine 中進行讀寫，導致死鎖。解決這個問題的方法是將發送或接收的操作移到另一個 Goroutine 中：

```go
func main() {
    ch1 := make(chan int)
    go func() {
        ch1 <- 13 // 將發送操作移到新的 Goroutine 中
    }()
    n := <-ch1
    println(n)
}
```

這樣可以保證無緩衝 channel 在不同的 Goroutine 中操作，避免死鎖。

接下來，探討一下有緩衝的 channel：

有緩衝的 channel 具備內置的緩衝區，因此其發送操作在緩衝區未滿時不會阻塞，接收操作在緩衝區非空時也不會阻塞。但是，當緩衝區滿或空時，相應的操作將會阻塞。看看這個例子：

```go
ch2 := make(chan int, 1)
n := <-ch2 // 緩衝區空，接收操作阻塞

ch3 := make(chan int, 1)
ch3 <- 17  // 發送成功，緩衝區未滿
ch3 <- 27  // 緩衝區滿，再次發送操作阻塞
```

有緩衝與無緩衝 channel 在實際應用上各有適用場景，具體選擇視需要而定。這裡先暫且介紹到這裡，後續還會有更多有關 channel 的實用範例和詳解。

### 單向 Channel 的使用說明

在 Go 語言中，我們可以透過使用 `<-` 運算符來創建單向 Channel，包括只發送（send-only）和只接收（recv-only）的 Channel。以下是一些實際的例子來說明如何操作這些 Channel：

```go
ch1 := make(chan<- int, 1) // 宣告一個只發送的 Channel
ch2 := make(<-chan int, 1) // 宣告一個只接收的 Channel

<-ch1       // 這是無效的操作：不能從一個只發送的 Channel 接收數據
ch2 <- 13   // 這也是無效的操作：不能向一個只接收的 Channel 發送數據
```

在實際的應用中，這種只發送或只接收的 Channel 常用於函數的參數或返回值，這有助於限制對 Channel 的操作。例如，在下面的程式碼中：

```go
func produce(ch chan<- int) {
    for i := 0; i < 10; i++ {
        ch <- i + 1  // 向 Channel 發送數據
        time.Sleep(time.Second)  // 每次發送後暫停一秒
    }
    close(ch)  // 發送完成後關閉 Channel
}

func consume(ch <-chan int) {
    for n := range ch {
        println(n)  // 從 Channel 接收數據並輸出
    }
}

func main() {
    ch := make(chan int, 5)  // 創建一個可傳輸 int 的雙向 Channel
    var wg sync.WaitGroup
    wg.Add(2)
    go func() {
        produce(ch)  // 啟動生產者goroutine
        wg.Done()
    }()

    go func() {
        consume(ch)  // 啟動消費者goroutine
        wg.Done()
    }()

    wg.Wait()  // 等待所有goroutine執行完畢
}
```

在此範例中，`produce` 函數使用了 `chan<- int` 類型參數，僅允許向 `ch` 發送數據，而 `consume` 函數則使用了 `<-chan int` 類型參數，僅允許從 `ch` 接收數據。這種設計明確劃分了函數的職責，有助於避免在程式執行中發生錯誤。

```
 單向 Channel 與 Producer-Consumer 模式示意圖：

 ┌──────────────────┐     ┌──────────────────┐
 │  produce()       │     │  consume()       │
 │  chan<- int       │     │  <-chan int        │
 │  (只能發送)       │     │  (只能接收)       │
 └────────┬─────────┘     └────────┬─────────┘
          │                        │
          │   ┌────────────────┐   │
          └──▶│  ch (chan int)  │──▶┘
     送入     │  緩衝區 cap=5  │    取出
              │ ┌─┬─┬─┬─┬─┐   │
              │ │1│2│3│4│5│   │
              │ └─┴─┴─┴─┴─┘   │
              └────────────────┘

 類型轉換關係：
 ┌──────────────┐
 │  chan int     │  ← 雙向 channel
 │ (可讀可寫)    │
 └──────┬───────┘
        │ 隱式轉換
   ┌────┴────┐
   ▼         ▼
 ┌─────────┐  ┌─────────┐
 │chan<- int│  │<-chan int│
 │(只寫)   │  │(只讀)   │
 └─────────┘  └─────────┘
 注意：單向不能轉回雙向
```

在上述範例中，我們看到了如何在 Go 語言中透過built-in的 `close` 函數來關閉 channel。當你執行了 `close(ch)` 之後，channel 會被正式關閉，這個動作會使所有在等待 channel 傳送資料的程序都接收到一個訊號 — channel 已經關閉。

當 channel 關閉後，如果你嘗試從這個 channel 接收資料，將會發生以下情形：

```go
n := <-ch      // 當 ch 被關閉時，n 會被賦予 ch 元素類型的零值
m, ok := <-ch  // 當 ch 被關閉時，m 會被賦予 ch 元素類型的零值，並且 ok 會被設為 false
for v := range ch { // 當 ch 被關閉後，這個 for range 迴圈就會結束
    ... ...
}
```

這些接收方式讓我們能夠檢查 channel 是否已經關閉。使用“comma, ok”慣用法或透過 for range 迴圈，我們能夠明確知道 channel 的狀態。但是，如果你僅僅使用 `n := <-ch`，則無法確定 channel 是否已關閉。

需要特別留意的一點是，關閉 channel 通常是`發送端的責任`。發送端沒有安全的方式來檢查 channel 是否已經被關閉，並且如果嘗試向一個已經關閉的 channel 發送數據，會觸發 panic 狀況，例如下面的程式碼所示：

```go
ch := make(chan int, 5)
close(ch)
ch <- 13 // panic: send on closed channel
```

這裡提醒你，一旦 channel 被關閉，就不應該再有數據被發送至此 channel，以避免程序崩潰。

在進行 Go 語言的編程過程中，當面對需要對多個 channel 同時操作的情況，`select` 就顯得非常有用。`select` 能夠讓我們在多個 channel 上進行發送或接收操作，這樣的設計使得 Go 能夠更有效地處理並行問題。下面將透過一個例子來詳細解釋 `select` 的使用方式和基本語法。

```go
select {
case x := <-ch1:     // 從 channel ch1 接收數據，如果 ch1 有數據可讀，則執行此分支
  .....

case y, ok := <-ch2: // 從 channel ch2 接收數據，ok 是一個bool值，用來檢查 channel 是否已經被關閉
                     // 如果 ch2 還開著，根據 y 的值來執行相應的邏輯
  .....

case ch3 <- z:       // 將值 z 發送到 channel ch3，如果 ch3 可以接收，則進行數據發送
  .....

default:             // 如果上述所有 channel 的操作都無法進行，則執行 default 分支
                     // 通常用來處理非阻塞或超時情況，確保 select 能夠有效地繼續執行
}
```

### select

```
 select 多路復用示意圖：

 ┌─────────────────────────────────────────────────────┐
 │  select {                                            │
 │                                                     │
 │    case x := <-ch1:    ←─┐                          │
 │    case y, ok := <-ch2: ←─┤ 哪個 channel 先就緒     │
 │    case ch3 <- z:      ←─┤ 就執行哪個 case          │
 │    default:            ←─┘ 都沒就緒走 default        │
 │  }                                                  │
 └─────────────────────────────────────────────────────┘

          ch1 ──── 有數據？──── 是 ──▶ 執行 case 1
           │
          ch2 ──── 有數據？──── 是 ──▶ 執行 case 2
           │
          ch3 ──── 可寫入？──── 是 ──▶ 執行 case 3
           │
          都不行？ ──── 有 default ──▶ 執行 default
                  └──── 無 default ──▶ 阻塞等待

 注意：如果多個 case 同時就緒，Go 會隨機選擇一個執行！
```

在這個 `select` 結構中，如果沒有 `default` 分支且所有的 `case` 分支因為 channel 阻塞而不能執行，則整個 `select` 會等待直到至少有一個 channel 可以進行通訊操作。接下來的文章中，我們將進一步探討 `select` 在實際應用中的巧妙用途和技巧，敬請期待。

### Go 語言中無緩衝 Channel 的實用指南

在 Go 語言中，無緩衝 channel 是一種非常有用的通信和同步工具。它們廣泛應用於各種場景中，尤其在處理同步和信號傳遞問題時表現出色。以下是無緩衝 channel 的一些典型使用方式：

#### 信號傳遞的應用

無緩衝 channel 常用於執行信號傳遞，支持 1 對 1 和 1 對 n 的通信模式。首先，來看一個 1 對 1 的例子：

```go
type signal struct{}

func worker() {
    println("worker is working...")
    time.Sleep(1 * time.Second)
}

func spawn(f func()) <-chan signal {
    c := make(chan signal)
    go func() {
        println("worker start to work...")
        f()
        c <- signal{}
    }()
    return c
}

func main() {
    println("start a worker...")
    c := spawn(worker)
    <-c
    fmt.Println("worker work done!")
}
```

在這段程式碼中，`spawn` 函數會啟動一個 goroutine 來執行傳入的函數，並在完成後通過 channel 發送信號。`main` 函數調用 `spawn` 後會等待這個信號，從而同步 `worker` 的執行狀態。

無緩衝 channel 也適用於實現 1 對 n 的信號通知機制，例如：

```go
func worker(i int) {
    fmt.Printf("worker %d: is working...\n", i)
    time.Sleep(1 * time.Second)
    fmt.Printf("worker %d: works done\n", i)
}

type signal struct{}
func spawnGroup(f func(i int), num int, groupSignal <-chan signal) <-chan signal {
    c := make(chan signal)
    var wg sync.WaitGroup

    for i := 0; i < num; i++ {
        wg.Add(1)
        go func(i int) {
            <-groupSignal
            fmt.Printf("worker %d: start to work...\n", i)
            f(i)
            wg.Done()
        }(i + 1)
    }

    go func() {
        wg.Wait()
        c <- signal{}
    }()
    return c
}

func main() {
    fmt.Println("start a group of workers...")
    groupSignal := make(chan signal)
    c := spawnGroup(worker, 5, groupSignal)
    time.Sleep(5 * time.Second)
    fmt.Println("the group of workers start to work...")
    close(groupSignal)
    <-c
    fmt.Println("the group of workers work done!")
}
```

這個例子展示了如何同步一組 worker 的開始工作時機。main goroutine 創建一組 worker goroutine，它們會在接收到 groupSignal 的關閉信號後同時開始執行任務。這是通過關閉一個無緩衝 channel 實現的，`這種方法可以使所有等待在這個 channel 上的 goroutine 接收到信號`，實現有效的 1 對 n 通信。

#### 使用無緩衝 channel 替代傳統鎖機制

在 Go 語言的世界裡，無緩衝 channel 擁有天然的同步特性，讓我們可以用它來優雅地替代傳統的鎖機制，進而使程式碼更為清晰與直觀。以一個簡單的計數器為例，我們先看看傳統的基於鎖的實作方法：

```go
type counter struct {
    sync.Mutex
    i int
}

var cter counter

func Increase() int {
    cter.Lock()
    defer cter.Unlock()
    cter.i++
    return cter.i
}

func main() {
    var wg sync.WaitGroup

    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func(i int) {
            v := Increase()
            fmt.Printf("goroutine-%d: current counter value is %d\n", i, v)
            wg.Done()
        }(i)
    }

    wg.Wait()
}
```

這段程式碼中，我們使用了互斥鎖來確保計數器的值在多個 Goroutine 中正確增加。每次調用 `Increase()` 都會上鎖和解鎖，確保計數操作不會發生競爭狀態。

現在，我們來看看如何利用無緩衝 channel 來達到同樣的效果，但是以一種更符合 Go 設計哲學的方式：“通過通信來共享內存”：

```go
type counter struct {
    c chan int
    i int
}

func NewCounter() *counter {
    cter := &counter{
        c: make(chan int),
    }
    go func() {
        for {
            cter.i++
            cter.c <- cter.i
        }
    }()
    return cter
}

func (cter *counter) Increase() int {
    return <-cter.c
}

func main() {
    cter := NewCounter()
    var wg sync.WaitGroup
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func(i int) {
            v := cter.Increase()
            fmt.Printf("goroutine-%d: current counter value is %d\n", i, v)
            wg.Done()
        }(i)
    }
    wg.Wait()
}
```

在這個版本中，計數器的每次增加操作都由一個獨立的 Goroutine 負責，並通過無緩衝 channel 發送給其他 Goroutine。這樣的設計不僅減少了鎖的使用，也使得程序整體更加符合 Go 的設計理念，提高了程式碼的效率和可讀性。

### 帶緩衝 channel 的實踐應用

帶緩衝的 channel 相較於無緩衝的 channel，最大的特點在於其async性質。當緩衝區尚未滿載時，執行發送操作的 Goroutine 不會發生阻塞；反之，當緩衝區內有數據存在時，執行接收操作的 Goroutine 也能保持非阻塞狀態。這一點讓帶緩衝的 channel 在特定應用場景中展現出與無緩衝 channel 不同的效能和適用性。接下來，讓我們詳細探討這些應用場景。

#### 作為消息隊列(Message Queue)使用

在多個 Goroutine 之間的通信過程中，帶緩衝的 channel 經常被設定為消息隊列。與無緩衝的 channel 相比，帶緩衝的設計在性能上更加優異，能夠更有效地管理數據流的傳遞。

#### 作為計數信號量（counting semaphore）

在 Go 的併發設計模式中，利用帶緩衝的 channel 作為計數信號量是一種常見的實踐。通過控制 channel 中的數據長度，我們可以精確地管理目前活動的 Goroutine 的數量，而 channel 的容量則定義了允許的最大同時活動 Goroutine 數。在此模型中，向 channel 發送數據即代表請求一個信號量，而從 channel 接收數據則意味著釋放一個信號量。

以下是一個具體的程式碼範例：

```go
var active = make(chan struct{}, 3)
var jobs = make(chan int, 10)

func main() {
    go func() {
        for i := 0; i < 8; i++ {
            jobs <- (i + 1)
        }
        close(jobs)
    }()

    var wg sync.WaitGroup

    for j := range jobs {
        wg.Add(1)
        go func(j int) {
            active <- struct{}{}
            log.Printf("handle job: %d\n", j)
            time.Sleep(2 * time.Second)
            <-active
            wg.Done()
        }(j)
    }
    wg.Wait()
}
```

在這個示例中，一組 Goroutine 被設計來處理不同的 job，而系統最多允許 3 個 Goroutine 同時處於活動狀態。

運行結果如下：

```go
2024/03/03 10:08:55 handle job: 1
2024/03/03 10:08:55 handle job: 4
2024/03/03 10:08:55 handle job: 8
2024/03/03 10:08:57 handle job: 5
2024/03/03 10:08:57 handle job: 7
2024/03/03 10:08:57 handle job: 6
2024/03/03 10:08:59 handle job: 3
2024/03/03 10:08:59 handle job: 2
```

從這些時間戳記錄可見，雖然程式創建了多個 Goroutine，但在計數信號量的調控下，同時處於活動狀態的 Goroutine 數量被有效地控制在了三個。

```
 帶緩衝 Channel 作為計數信號量示意圖：

 active channel (cap=3)：
 ┌───┬───┬───┐
 │ ■ │ ■ │ ■ │  ← 已滿，第 4 個 goroutine 被阻塞
 └───┴───┴───┘

 時間線：
 t=0s:   [Job1] [Job4] [Job8]       ← 3 個同時執行
         [Job2][Job3][Job5][Job6][Job7] ← 等待中

 t=2s:   [Job5] [Job7] [Job6]       ← 前 3 個完成，新 3 個進入
         [Job3][Job2] ← 等待中

 t=4s:   [Job3] [Job2]              ← 最後 2 個執行
```

### 總結

* Goroutine 是 Go 語言的併發基本單元，channel 則是通信和同步的工具。
* 無緩衝 channel 用於同步通信，帶緩衝 channel 用於異步通信。
* 無緩衝 channel 適合用於信號傳遞和替代鎖機制。
* 帶緩衝 channel 適合用於消息隊列和計數信號量。

更多Go語言相關的文章，歡迎參考我的部落格: <https://kaichiachen.github.io/2024/03/08/golang/go_channel/>

  
