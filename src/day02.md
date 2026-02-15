# Day2 - Go語言的設計哲學

Go語言，又稱Golang，是由Google開發的一種現代程式語言。它在2007年誕生，由Robert Griesemer、Rob Pike和Ken Thompson設計。Go語言的設計哲學非常獨特，使它在眾多程式語言中脫穎而出。這篇文章，我們來聊聊Go語言的設計哲學，看看這門語言是如何讓寫程式變得更加簡單、高效和愉快的。

所謂程式語言的設計哲學，就是指決定這門語言演化進程的高級原則和依據。好比Python的設計哲學是**簡單就是美**，使開發者能專注於解決問題本身，而不是被語法所困擾。而Java的設計哲學強調穩定性、可擴展性和跨平台性，Java的座右銘是「Write Once, Run Anywhere」。

**設計哲學之於程式語言，就好比一個人的價值觀之於這個人的行為**  
因為如果你不認同一個人的價值觀，那你其實很難與之持續交往下去，即所謂道不同不相為謀。類似的，如果你不認同一門程式語言的設計哲學，那麼很有可能你在後續的語言學習中，就會遇到上面提到的這些問題，而且可能會讓你失去繼續學習的精神動力。

我認為Go語言的設計哲學可以總結為以下五點：簡單、顯式、組合、併發和工程導向。  

### 簡單

簡單意味著語言本身保持最小的語法和特性集，避免過於複雜的結構和特性，使程式碼更易於閱讀和理解。例如，Go 不支持繼承、多型等傳統 OOP（物件導向程式設計）的特性，而是依賴於更簡單的結構體和接口。

```go
package main

import "fmt"

type Person struct {
    Name string
    Age  int
}

func main() {
    p := Person{Name: "John", Age: 30}
    fmt.Println(p)
}
```

### 顯式

Go 語言強調「顯式」而非「隱式」，這意味著程式碼應該明確表達其意圖。這有助於減少錯誤，提高程式碼的可讀性和可維護性。Go 的錯誤處理機制也是這一理念的體現，它要求開發者顯式處理錯誤，而不是依賴隱式的異常機制。所以在開發Go程式時，經常可以在開發中而不是編譯或運行時才捕捉到人為錯誤。

```go
package main

import (
    "fmt"
    "os"
)

func main() {
    file, err := os.Open("test.txt")
    if err != nil {
        fmt.Println("Error opening file:", err)
        return
    }
    defer file.Close()
    // 處理文件的邏輯
}
```

### 組合

Go 語言提倡通過「組合」來構建複雜的系統，而不是通過繼承。這種方式使得程式碼更加靈活和可重用。Go 的接口系統允許我們定義一組行為，而不需要在類層次上建立複雜的繼承關係。

```go
package main

import "fmt"

type Flyer interface {
    Fly()
}

type Bird struct{}

func (b Bird) Fly() {
    fmt.Println("Bird is flying")
}

type Plane struct{}

func (p Plane) Fly() {
    fmt.Println("Plane is flying")
}

func main() {
    var f Flyer

    f = Bird{}
    f.Fly()

    f = Plane{}
    f.Fly()
}
```

這有點類似物件導向的多型，只要struct滿足事先定義的interface介口方法，就可以實作並且執行。

### 併發

Go 語言原生支持「併發」，並通過 goroutine 和 channel 提供了簡單且強大的併發程式開發模型。這使得開發者能夠輕鬆地寫出高效的併發程式，而不需要處理複雜的併發控制機制。

```go
package main

import (
    "fmt"
    "time"
)

func say(s string) {
    for i := 0; i < 5; i++ {
        time.Sleep(100 * time.Millisecond)
        fmt.Println(s)
    }
}

func main() {
    go say("world")
    say("hello")
}
```

### 工程導向

Go 語言的設計非常「工程導向」，它關注於實際工程中的問題，提供了快速編譯、簡單部署、以及高效執行的特性。這使得 Go 成為構建大型分佈式系統和微服務架構的理想選擇。

### 總結

* Go語言設計哲學強調簡單、顯式、組合、併發和工程導向。
* 它通過減少語法和特性，使程式碼更易讀和理解。
* 強調顯式處理錯誤，減少潛在錯誤風險。
* 鼓勵組合而非繼承，提高程式碼靈活性和重用性。
* 原生支持併發，方便開發高效的併發程式。
* 工程導向設計，適合大型分佈式系統和微服務架構。

更多關於Go語言的文章可以參考我的部落格: <https://kaichiachen.github.io/2023/12/12/golang/design_philosophy/>

  
