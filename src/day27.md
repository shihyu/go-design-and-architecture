# Day27 - 如何用Go寫process pool, thread pool和goroutine pool

當我們在開發程式的時候，常常需要執行大量的任務。這些任務有時候可以是I/O密集型（像是網路請求、檔案讀寫），也可能是CPU密集型（像是複雜的計算）。如果我們讓所有的任務都在主執行緒裡面依序跑，這樣不僅效能差，還可能導致我們的程式在處理慢的任務時卡住。所以，這時候我們會想要有一個 **pool** 來幫我們管理這些任務的執行。

這篇文章會帶你了解什麼是 `process pool`、`thread pool` 和 `goroutine pool`，以及如何在Go語言中實現它們。

---

### 1. Process Pool

#### 什麼是 Process Pool？

`Process pool` 是一組預先創建好的 `process`，用來執行多個任務。每個 `process` 獨立於其他 `process`，擁有自己的記憶體空間和資源，並且不會共享資料。由於這些 `process` 是預先創建的，因此我們可以避免重複創建和銷毀 `process` 的開銷，從而提升效能。

#### Go 語言中如何實現 Process Pool？

在Go語言中，標準函式庫並沒有直接支援 `process pool`，但我們可以透過 `os/exec` 套件來創建和管理 `process`。假設我們有一個簡單的範例，想讓多個 `process` 同時執行一個外部命令：

```go
package main

import (
	"fmt"
	"os/exec"
	"sync"
)

func worker(id int, wg *sync.WaitGroup) {
	defer wg.Done()
	cmd := exec.Command("echo", fmt.Sprintf("Process %d", id))
	output, err := cmd.CombinedOutput()
	if err != nil {
		fmt.Printf("Process %d failed: %s\n", id, err)
		return
	}
	fmt.Printf("Process %d finished: %s\n", id, output)
}

func main() {
	var wg sync.WaitGroup
	processCount := 5
	for i := 1; i <= processCount; i++ {
		wg.Add(1)
		go worker(i, &wg)
	}
	wg.Wait()
}
```

在這個範例中，我們創建了5個 `process`，並讓它們同時執行 `echo` 命令。

### 2. Thread Pool

#### 什麼是 Thread Pool？

`Thread pool` 是一組預先創建好的 `thread`，這些 `thread` 可以用來執行多個任務。與 `process` 不同的是，`thread` 是共享同一個記憶體空間的，因此它們之間的通訊和資料共享相對容易。但要小心的是，`thread` 之間共享資料也會導致同步問題（例如 race condition），這時我們就需要使用鎖來避免這些問題。

#### Go 語言中如何實現 Thread Pool？

雖然Go語言沒有直接的 `thread pool`，但我們可以透過 `goroutine` 來達到類似的效果。`goroutine` 是Go語言的輕量級執行緒，我們可以將多個任務分配給多個 `goroutine` 來執行。這裡是一個簡單的 `thread pool` 模擬範例：

```go
package main

import (
	"fmt"
	"time"
)

func worker(id int, jobs <-chan int, results chan<- int) {
	for j := range jobs {
		fmt.Printf("Worker %d processing job %d\n", id, j)
		time.Sleep(time.Second) // 模擬工作
		results <- j * 2 // 模擬計算結果
	}
}

func main() {
	const workerCount = 3
	const jobCount = 5

	jobs := make(chan int, jobCount)
	results := make(chan int, jobCount)

	// 創建worker pool
	for w := 1; w <= workerCount; w++ {
		go worker(w, jobs, results)
	}

	// 發送任務到jobs通道
	for j := 1; j <= jobCount; j++ {
		jobs <- j
	}
	close(jobs)

	// 接收結果
	for a := 1; a <= jobCount; a++ {
		fmt.Printf("Result: %d\n", <-results)
	}
}
```

在這個範例中，我們創建了一個簡單的 `worker pool`，每個 `worker` 從 `jobs` 通道中接收任務，並將結果送回 `results` 通道。

### 3. Goroutine Pool

#### 什麼是 Goroutine Pool？

`Goroutine pool` 是Go語言特有的概念，類似於 `thread pool`，但它是使用 `goroutine` 來實現的。`goroutine` 是Go語言中的輕量級執行單位，每個 `goroutine` 只佔用少量的記憶體（大約2KB），因此我們可以創建大量的 `goroutine` 來同時執行任務。

不過，如果我們不控制 `goroutine` 的數量，創建過多的 `goroutine` 也會導致記憶體不足，或是效能下降。因此，我們需要一個 `goroutine pool` 來限制同時執行的 `goroutine` 數量。

#### Go 語言中如何實現 Goroutine Pool？

這裡是一個簡單的 `goroutine pool` 範例，我們使用一個緩衝通道來限制同時執行的 `goroutine` 數量：

```go
package main

import (
	"fmt"
	"time"
)

func worker(id int) {
	fmt.Printf("Worker %d starting\n", id)
	time.Sleep(time.Second) // 模擬工作
	fmt.Printf("Worker %d done\n", id)
}

func main() {
	const poolSize = 3
	const jobCount = 5

	sem := make(chan struct{}, poolSize)

	for i := 1; i <= jobCount; i++ {
		sem <- struct{}{} // 塞入一個空結構體，來限制 goroutine 數量
		go func(id int) {
			defer func() { <-sem }() // 完成後釋放
			worker(id)
		}(i)
	}

	// 等待所有goroutine完成
	for i := 0; i < poolSize; i++ {
		sem <- struct{}{}
	}
}
```

在這個範例中，我們透過 `sem`（一個緩衝通道）來控制同時執行的 `goroutine` 數量，確保最多同時只有3個 `goroutine` 在執行任務。

---

### 結論

`process pool`、`thread pool` 和 `goroutine pool` 各有其應用場景。在Go語言中，雖然我們無法直接使用 `process` 和 `thread`，但 `goroutine` 的設計使得我們可以更輕鬆地處理並行任務。當然，使用 `goroutine` 也要小心控制數量，避免創建過多 `goroutine` 造成記憶體耗盡的問題。

透過這篇文章的範例，相信你已經對如何在Go語言中實現這些 `pool` 有了基本的概念。你可以根據自己的應用場景，選擇合適的解決方案來提升程式的效能。

  
