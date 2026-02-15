# Day 10 - Interface的使用技巧

這篇文章要來聊聊如何運用Interface改善我們的程式專案，換句話說，我們講的是Go語言中interface的使用模式和習慣。

首先得強調一點，那就是「適度封裝」。換句話說，我們不是為了封裝而封裝。我們之前提到過，接口其實就是一種封裝的手段，主要目的是為了解耦。所以，重點是我們應該在真正需要的時候才使用接口。

大部分情況下，接口都能對應用設計帶來好處。但如何恰當地使用interface來優化我們的程式設計呢？這就要從Go語言的「組合」設計哲學說起了。

### 一切皆組合

Go語言的創始人Rob Pike有句名言：如果說C++和Java是講究類型層次和分類的語言，那Go語言則是專注於組合的語言。想像Go應用就像一台機器，組合的藝術就在於如何把分散於各個包中的零件有效組合起來。這是Go的核心設計理念之一，而「正交性」則讓這種設計更加方便實現。

所謂正交性，其實是個幾何學上的概念，意指兩條直線在直角交會。在程式設計中，這表示兩個或多個元素之間的相互獨立，互不影響。這種設計使得當一個元素改變時，不會牽連到其他元素。

#### 垂直組合

在傳統的面向對象語言中，如C++，通常透過繼承來建立一個類型體系。但在Go語言中，我們不使用繼承，而是採用「組合」來賦予單一類型更多的功能，這種方法有點類似於硬體升級時的垂直擴充。因為這不涉及繼承，所以新類型與被嵌入的類型之間不存在「父子關係」，它們之間的關聯僅由方法名來決定。更多關於垂直組合(繼承)可以參考我之前寫的這篇文章Method: 怎麼用變數模擬繼承

#### 水平組合

當通過垂直組合確立了類型之後，我們如何將這些獨立的component連結成一個完整的應用呢？來看一個具體的例子：

假設我們要寫一個函數，負責把數據寫到硬碟上：

```go
func Save(f *os.File, data []byte) error
```

上面函數用\*os.File來定位數據要寫入的目標。但這樣設計有幾個問題。首先，它很難進行測試，因為\*os.File涉及到實際的file descripter，我們必須操作實體文件來測試這個函數，這非常不方便。

其次，Save函數直接指定\*os.File最為參數型態限制了它的可擴充性。如果我們想改為向網路儲存寫數據，就必須修改這個函數的參數型態，這會影響到所有使用這個函數的程式碼。

那麼，如何改進Save函數的設計呢？我們可以嘗試使用interface。以下是新版Save函數的原型：

```go
func Save(w io.Writer, data []byte) error
```

這裡，我們用io.Writer接口替代了\*os.File。這種設計遵循了interface分離的原則，因為io.Writer只包含一個Write方法，恰恰是Save函數所需要的。

如此一來，新版的Save函數不僅可以向硬碟寫入數據，也能向網路儲存(NAS)寫入數據，並支持任何實現了Write方法的類型，這大大提升了其可擴充性。而且，這也使得測試變得簡單，只需使用bytes.NewBuffer創建一個\*bytes.Buffer，傳入Save函數，完成測試後比對數據即可。

### Interface的其他設計模式

Go語言中有幾種常見的接口設計模式：

#### 工廠模式

這種模式在原生的Library sync、log和bufio中都有使用。以log中的New函數為例：

```go
type Logger struct {
    mu     sync.Mutex
    prefix string
    flag   int
    out    io.Writer
}

func New(out io.Writer, prefix string, flag int) *Logger {
    return &Logger{out: out, prefix: prefix, flag: flag}
}
```

#### 包裝器模式

這種模式用於對輸入數據進行過濾或變換。一個典型的例子是Go標準庫`io`中的`LimitReader`函數：

```go
func LimitReader(r Reader, n int64) Reader { 
    // Do something
    reader := &LimitedReader{r, n}
    // Do something
    return reader 
}

type LimitedReader struct {
    R Reader
    N int64
}

func (l *LimitedReader) Read(p []byte) (n int, err error) {
    // ...
}
```

`LimitReader`是一個新的Reader，在init `io.Reader`前後對Read函數做客製化包裝。

#### 適配器模式

這種模式的核心是適配函數的類型轉換。最常見的例子是`http.HandlerFunc`：

```go
func greetings(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Welcome!")
}

func main() {
    http.ListenAndServe(":8080", http.HandlerFunc(greetings))
}
```

這個例子展示了怎麼透過 `http.HandlerFunc` 這個適配器函數類型，將一般的 `greetings` 函數迅速轉換成符合 `http.Handler` interface的類型。在 Go 語言的 `http` Library中，`http.HandlerFunc` 的定義如下所示：

```go
// $GOROOT/src/net/http/server.go
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}

type HandlerFunc func(ResponseWriter, *Request)

func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
    f(w, r)
}
```

經過 `HandlerFunc` 的適配轉換後，我們可以將其object用作參數，pass給接收 `http.Handler` 的 `http.ListenAndServe` 函數，這樣一來，就能夠實現基於interface的組合設計，轉化後`greetings`就有了`ServeHTTP`方法。這種設計方式不僅簡化了程式碼的複雜性，也增強了程式的封裝化和可重用性，讓開發者能夠更靈活地應對不同的網路服務需求。

#### 中間件（Middleware）

中間件實質上是結合了包裝模式和適配器模式。看看這個例子：

```go
func validateAuth(s string) error {
    if s != "123456" {
        return fmt.Errorf("%s", "bad auth token")
    }
    return nil
}

func greetings(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Welcome!")
}

func logHandler(h http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        log.Printf("[%s] %q\n", r.Method, r.URL.String())
        h.ServeHTTP(w, r)
    })
}

func authHandler(h http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        err := validateAuth(r.Header.Get("auth"))
        if err != nil {
            http.Error(w, "bad auth param", http.StatusUnauthorized)
            return
        }
        h.ServeHTTP(w, r)
    })
}

func main() {
    http.ListenAndServe(":8080", logHandler(authHandler(http.HandlerFunc(greetings))))
}
```

這裡的 logHandler 和 authHandler 是中間件，通過適配器函數類型（http.HandlerFunc）將普通函數轉為實現 http.Handler 的類型的object。

### 總結

* 了解接口的本質和原則，不為了封裝而封裝。
* 善用 Go 語言的組合設計哲學。
  + **垂直組合**：透過傳統程式語言繼承的方式來垂直增加類型功能。
  + **水平組合**：將不同 component 但是共用的Method封裝成一個interface，把這個interface最為參數型態，類似傳統程式語言的多型。
* 透過Go語言的Interface也能實現各種不同的設計模式。
  + **工廠模式**：靈活創建多種類型同個class的object，例如Logger，不同的Logger object主要差別可能就是寫入的路徑，所以創建不同的Logger適合工廠模式。
  + **包裝器模式**：增加新功能而不改變原始interface，有點類似Python的decorator可以對class前後進行前置或是後置處理。
  + **適配器模式**：轉化函數為特定其他但相同簽名的函數，以適配所需函數的Method實現。
  + **中間件**：增加新功能而不改變原始function功能，有點類似Python的decorator可以對function前後進行前置或是後置處理。

更多Go語言相關的文章，歡迎參閱我的部落格: <https://kaichiachen.github.io/2024/02/16/golang/interface_usage/>

  
