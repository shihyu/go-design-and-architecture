# Day22 - 使用 Golang 打造 Web 應用程式

## 1. 簡介

Golang（或 Go 語言）是由 Google 開發的一種高效能、並發性強且語法簡潔的程式語言，非常適合用來開發 Web 應用程式。本文將帶你從零開始，手把手教你如何使用 Go 語言打造一個基本的 Web 應用程式，並且會穿插一些實用的小技巧，讓你快速上手。

## 2. 為什麼選擇 Golang？

* **高效能**：Golang 編譯後的程式是原生的機器碼，速度飛快，不像 JavaScript 這種需要經過中介層解釋執行的語言。
* **簡單易學**：Go 語法簡潔，容易閱讀且容易維護，學起來不會太痛苦。
* **併發性強**：內建的 Goroutines 讓你輕鬆實現高併發的程式設計，不需要搞懂複雜的線程管理。
* **強大的社群與工具**：Golang 有豐富的第三方套件、工具與文件，學習曲線非常友善。

## 3. 環境準備

在開始之前，你需要安裝 Golang 和一個適合的編輯器，例如 VS Code。確保 Golang 安裝成功，可以用以下指令檢查版本：

```go
go version
```

## 4. 開始打造 Web 應用程式

首先，我們來建立一個簡單的 Web 伺服器，這個伺服器會聆聽特定的端口，並回傳一段簡單的訊息。

### Step 1: 建立專案資料夾

在你的工作目錄下建立一個新的資料夾，並且進入這個資料夾。

```go
mkdir go-web-app
cd go-web-app
```

### Step 2: 初始化 Go 模組

初始化你的 Go 專案，這會在資料夾中生成一個 `go.mod` 檔案，負責管理套件。

```go
go mod init go-web-app
```

### Step 3: 寫一個基本的 Web 伺服器

建立一個檔案 `main.go`，然後在裡面寫入以下程式碼：

```go
package main

import (
    "fmt"
    "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, Go Web Server!")
}

func main() {
    http.HandleFunc("/", handler)
    fmt.Println("伺服器已啟動，正在監聽 :8080 端口...")
    http.ListenAndServe(":8080", nil)
}
```

### Step 4: 執行伺服器

在終端機輸入以下指令來執行你的 Web 伺服器：

```go
go run main.go
```

打開瀏覽器並輸入 `http://localhost:8080`，你會看到瀏覽器顯示「Hello, Go Web Server!」，這表示你的伺服器已經成功運作了！

## 5. 加入更多功能

現在我們有了一個基本的 Web 伺服器，接下來我們可以嘗試加入更多功能，例如路由、靜態檔案處理或是資料庫連接等。

### 加入路由

讓我們來加入更多的路由，來處理不同的請求路徑。

```go
func aboutHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "這是關於我們的頁面")
}

func main() {
    http.HandleFunc("/", handler)
    http.HandleFunc("/about", aboutHandler)
    fmt.Println("伺服器已啟動，正在監聽 :8080 端口...")
    http.ListenAndServe(":8080", nil)
}
```

現在當你訪問 `http://localhost:8080/about` 時，會看到「這是關於我們的頁面」的訊息。

### 靜態檔案處理

如果你想要提供靜態資源（如圖片、CSS、JS），可以這樣做：

```go
http.Handle("/static/", http.StripPrefix("/static/", http.FileServer(http.Dir("static"))))
```

這樣，你只需要將靜態資源放入 `static` 資料夾，就可以透過 `/static` 路徑訪問。

### 連接資料庫

Go 支援多種資料庫，這裡簡單示範如何使用 `database/sql` 連接 MySQL：

```go
import (
    "database/sql"
    _ "github.com/go-sql-driver/mysql"
)

func connectDB() (*sql.DB, error) {
    db, err := sql.Open("mysql", "user:password@tcp(127.0.0.1:3306)/dbname")
    if err != nil {
        return nil, err
    }
    return db, nil
}
```

透過以上方式，你可以輕鬆連接 MySQL 資料庫，並進一步操作資料。

## 6. 部署應用程式

當你完成了開發和測試，可以將應用程式部署到雲端，常見的選擇包括 Docker 容器化後部署到 Kubernetes、Heroku 或是 Google Cloud Platform。

## 7. 小結

Golang 是一個非常適合用來開發 Web 應用程式的語言，不僅效能卓越，還有簡單的語法和強大的併發能力。透過這篇文章，你應該已經掌握了如何從頭打造一個基本的 Go Web 應用程式，並了解如何加入更多功能。希望這篇文章對你有幫助，讓你能夠更輕鬆地進入 Golang 的世界！

  
