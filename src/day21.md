# Day21 - 用Fx進行依賴注入(Dependency Injection)

## 什麼是依賴注入?

首先，什麼是依賴注入呢？依賴注入（Dependency Injection，簡稱DI）是一種設計模式，主要用來解決物件之間的依賴性問題。當一個物件需要使用另一個物件時，我們通常會在這個物件的內部去創建它所需的依賴物件。這樣一來，物件之間就會產生緊密的耦合性，使得程式碼變得不靈活，也很難進行測試和維護。

依賴注入的目的是將依賴物件的創建責任轉移給外部系統，通過單例模式或構造函數(constructor)來將依賴注入到需要的物件中。這樣一來，我們可以更方便地替換或擴展物件的依賴，也能更容易進行單元測試，因為可以輕鬆地注入模擬物件（mock objects）。

### 傳統的依賴注入

在進入主題之前，我們來看一個不使用任何依賴注入框架的範例：

```go
package main

import "fmt"

type Database struct {
    Name string
}

func (db *Database) Connect() {
    fmt.Println("Connecting to database:", db.Name)
}

type Server struct {
    Database *Database
}

func (s *Server) Start() {
    s.Database.Connect()
    fmt.Println("Server started!")
}

func main() {
    db := &Database{Name: "MySQL"}
    server := &Server{Database: db}
    server.Start()
}
```

在這個範例中，我們有一個`Database`的struct，它模擬了連接到資料庫的功能。`Server`則依賴於`Database`來執行其操作。在`main`函數中，我們手動創建`Database`和`Server`，並將`Database`作為依賴注入到`Server`中。

這個範例雖然簡單易懂，但當我們的專案規模變大、依賴物件變得複雜時，手動管理這些依賴會變得相當麻煩。這時候，我們就需要一個更高效的方式來處理依賴注入，而這正是Fx出現的原因。

## 什麼是Fx？

Fx 是一個用於 Go 語言的依賴注入框架，它基於 Uber 的開源專案開發，旨在簡化和標準化 Go 應用程式的初始化過程。Fx 通過自動管理物件的生命週期，幫助開發者集中精力在業務邏輯上，而不必煩惱於如何管理各種依賴關係。

Fx 的核心概念是使用依賴注入來管理應用程式中的各種物件和資源。它可以自動解析依賴關係，並確保在合適的時候創建和銷毀物件。這樣的好處是我們不必手動編寫繁瑣的初始化程式碼，還可以更方便地進行測試。

### 用Fx進行依賴注入

讓我們來看看如何用 Fx 來改寫之前的範例。

```go
package main

import (
    "go.uber.org/fx"
    "fmt"
)

type Database struct {
    Name string
}

func NewDatabase() *Database {
    return &Database{Name: "MySQL"}
}

func (db *Database) Connect() {
    fmt.Println("Connecting to database:", db.Name)
}

type Server struct {
    Database *Database
}

func NewServer(db *Database) *Server {
    return &Server{Database: db}
}

func (s *Server) Start() {
    s.Database.Connect()
    fmt.Println("Server started!")
}

func main() {
    app := fx.New(
        fx.Provide(NewDatabase),
        fx.Provide(NewServer),
        fx.Invoke(func(s *Server) {
            s.Start()
        }),
    )
    app.Run()
}
```

在這個範例中，我們使用了 Fx 來管理`Database`和`Server`的創建和注入。`fx.Provide`函數用來註冊依賴，它告訴 Fx 如何創建特定的物件。而`fx.Invoke`函數則用來在應用程式啟動時執行特定的操作。

Fx 的好處是，它自動處理了物件的創建和銷毀，並確保依賴物件按照正確的順序被創建。這不僅讓程式碼更加簡潔，也降低了錯誤的發生機率。

## 結論

1. 依賴注入（DI）的概念：解決物件之間的依賴問題，透過外部注入依賴，減少程式碼耦合，提升靈活性和測試便利性。
2. Fx 的優勢：Fx 是一個Go的依賴注入框架，自動管理物件的創建與銷毀，簡化初始化過程，讓程式碼更容易維護和測試。

Fx 不僅簡化了應用程式的初始化過程，也讓程式碼變得更容易測試和維護。如果你正在尋找一個高效的方式來處理 Go 專案中的依賴問題，那麼 Fx 是我強力推薦的工具！

更多Go語言相關的文章，歡迎關注我的部落格: <https://kaichiachen.github.io/2024/06/01/golang/go_dependency_injection/>

  
