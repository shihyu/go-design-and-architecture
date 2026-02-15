# Day3 - 一個Go的程式專案是怎麼樣的

我們編寫的 Go 程式一般都是簡單程式，一般由一個或幾個 Go 原始碼檔案組成，而且所有原始碼檔案都在同一個目錄中。但是生產環境中運行的程式可不會這麼簡單，通常它們都有著複雜的專案結構佈局。弄清楚一個實用 Go 專案的專案佈局標準是 Go 開發者走向編寫複雜 Go 程式的第一步，也是必經的一步。

但 Go 官方到目前為止也沒有給出一個關於 Go 專案佈局標準的正式定義。那在這樣的情況下，Go 社群是否有我們可以遵循的參考佈局，或者標準呢？有的。在這一篇文章裡，我就來告訴你 Go 社群廣泛採用的 Go 專案佈局是什麼樣子的。

一個 Go 專案通常分為可執行程式專案和第三方module/lib，現在我們就來分析一下這兩類 Go 專案的典型結構佈局分別是怎樣的。首先我們先來看 Go 可執行程式專案的典型結構佈局。可執行程式專案是以建構可執行程式為目的的專案，Go 社群針對這類 Go 專案所形成的典型結構佈局是這樣的：

```go
#> tree -F goProject
goProject
├── cmd/
│   ├── app1/
│   │   └── main.go
│   └── app2/
│       └── main.go
├── go.mod
├── go.sum
├── internal/
│   ├── pkga/
│   │   └── pkg_a.go
│   └── pkgb/
│       └── pkg_b.go
├── pkg1/
│   └── pkg1.go
└── pkg2/
    └── pkg2.go
```

我來解釋一下這裡面的幾個重點。

我們從上往下按順序來，先來看 cmd 目錄。cmd 目錄就是存放專案要編譯構建的可執行檔案對應的帶有`func main()`.go檔案。如果你的專案中有多個可執行檔案需要構建，每個可執行檔案的 main 套件單獨放在一個子目錄中，比如圖中的 app1、app2，cmd 目錄下的各 app 的 main 套件將整個專案的依賴連接在一起。

而且通常來說，main 應該很簡潔。我們在 main 裡的實作會做一些command參數解析、資源初始化、Logger初始化、資料庫連接初始化等工作，之後就程式就會進去其他package去執行更高級的邏輯控制。另外，也有一些 Go 專案將 cmd 這個名字改為 app 或其他名字，但它的功能其實並沒有變。

接著我們來看 pkgN 目錄，這是一個存放專案自身要使用、同樣也是可執行檔案對應 main 所要依賴的Library，同時這些目錄下的Library還可以被外部Project引用。

然後是 go.mod 和 go.sum，它們是管理 Go 語言第三方Library所使用的配置檔。建議所有新專案都基於 Go Module 來進行Library管理，因為這是目前 Go 官方推薦的標準構建模式。

**好了到這裡，我們已經了解了 Go 可執行程式專案的典型佈局，現在我們再來看看 Go Library的典型結構佈局是怎樣的。**  
Go Library僅對外暴露 Go 套件，這類專案的典型佈局形式是這樣的

```go
#> tree -F libLayout 
libLayout
├── go.mod
├── internal/
│   ├── pkga/
│   │   └── pkg_a.go
│   └── pkgb/
│       └── pkg_b.go
├── pkg1/
│   └── pkg1.go
└── pkg2/
    └── pkg2.go
```

我們看到，Library類型專案相比於 Go 可執行程式專案的佈局要簡單一些。因為這類Project不需要構建可執行程式，所以去除了 cmd 目錄。

Go Library的初衷是為了對外部（開源或組織內部公開）暴露 API，對於僅限Project內部使用而不想暴露到外部的套件，可以放在Project頂層的 internal 目錄下面。當然 internal 也可以有多個並存在於Project結構中的任一目錄層級中，關鍵是專案結構設計人員要明確各級 internal 套件的應用層次和範圍。

對於有一個且僅有一個套件的 Go Library來說，我們也可以將上面的佈局做進一步簡化，簡化的佈局如下所示：

```go
#> tree -L 1 -F singleLibLayout
singleLibLayout
├── feature1.go
├── feature2.go
├── go.mod
└── internal/
```

簡化後，我們將這唯一套件的所有原始碼檔案放置在Project的頂層目錄下（比如上面的 feature1.go 和 feature2.go），其他佈局元素位置和功用不變。

### 總結

首先，對於以生產可執行程式為目的的 Go Project，它的典型Project結構分為五部分：

* 放在Project頂層的 Go Module 相關檔案，包括 go.mod 和 go.sum
* cmd 目錄：存放Project要編譯構建的可執行檔案所對應的 main 套件的源檔
* Project套件目錄：每個Project下的非 main 套件都“平鋪”在Project的根目錄下，每個目錄對應一個 Go 套件
* internal 目錄：存放僅Project內部引用的 Go 套件，這些套件無法被Project外引用

第二，對於以生產可復用Library為目的的 Go Project，它的典型結構則要簡單許多，我們可以直接理解為在 Go 可執行程式專案的基礎上去掉 cmd 目錄。

更多Go相關的文章可以參閱我的部落格: <https://kaichiachen.github.io/2023/12/12/golang/design_philosophy/>

  
