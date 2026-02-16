# Day24 - Go Module 的設計思想以及如何動手寫一個 module 貢獻到 GitHub

## Go Module 的設計思想

Go 模組（Go Module）是 Go 語言的官方套件管理系統，它旨在簡化依賴管理和版本控制。讓我們一起來看看 Go Module 的設計思想和如何動手寫一個 module 並貢獻到 GitHub！

### 為什麼需要 Go Module？

在 Go 語言的早期，依賴管理主要依賴於 GOPATH 和 vendor 目錄。這種方式雖然有效，但缺乏版本控制，會導致版本衝突和依賴混亂。Go Module 的引入解決了這些問題，提供了更好的版本控制、模組管理和簡化的依賴配置。

### Go Module 的核心設計思想

1. **模組化管理**：

   * Go Module 引入了模組（module）概念，每個模組包含一組相關的 Go 程式碼和依賴。這樣可以更好地管理和隔離不同的套件。
2. **版本控制**：

   * 每個模組有一個 `go.mod` 文件來管理版本。這個文件記錄了模組的名稱、版本和依賴，確保專案的重現性。
3. **語意化版本（SemVer）**：

   * Go Module 遵循語意化版本控制（Semantic Versioning）。這意味著模組的版本號反映了 API 的變更，讓用戶可以預測版本升級的影響。
4. **模組檢索和緩存**：

   * Go Module 使用 Go Proxy 服務來檢索和緩存模組。這不僅提高了依賴的可靠性，還加快了下載速度。
5. **版本衝突解決**：

   * Go Module 支援版本衝突解決和依賴消解。它會選擇一個最適合的版本，避免不同依賴間的衝突。

## 如何動手寫一個 Go Module 並貢獻到 GitHub

### 步驟一：設置 Go Module

1. **創建新的專案目錄**：

   ```go
   mkdir mymodule
   cd mymodule
   ```
2. **初始化 Go Module**：

   ```go
   go mod init github.com/username/mymodule
   ```

   * 這條命令會創建一個 `go.mod` 文件，其中 `github.com/username/mymodule` 是模組的名稱。

### 步驟二：撰寫 Go 程式碼

1. **創建一個新的 Go 文件**：

   ```go
   // math.go
   package mymodule

   // Add 函數用於加法運算
   func Add(a int, b int) int {
       return a + b
   }
   ```
2. **撰寫單元測試**：

   ```go
   // math_test.go
   package mymodule

   import "testing"

   func TestAdd(t *testing.T) {
       result := Add(1, 2)
       if result != 3 {
           t.Errorf("Add(1, 2) = %d; want 3", result)
       }
   }
   ```

### 步驟三：測試模組

1. **執行單元測試**：

   ```go
   go test
   ```

   * 確保測試通過，這樣可以保證你的模組運作正常。

### 步驟四：將模組推送到 GitHub

1. **創建 Git 存儲庫**：

   * 在 GitHub 上創建一個新的儲存庫，如 `mymodule`。
2. **初始化 Git 並提交代碼**：

   ```go
   git init
   git add .
   git commit -m "Initial commit"
   git branch -M main
   git remote add origin https://github.com/username/mymodule.git
   git push -u origin main
   ```

### 步驟五：使用模組

1. **在另一個專案中使用你的模組**：
   * 創建一個新的 Go 專案，並在 `go.mod` 文件中添加你的模組作為依賴：

     ```go
     go mod edit -require=github.com/username/mymodule@v0.0.1
     ```
   * 使用你的模組：

     ```go
     package main

     import (
         "fmt"
         "github.com/username/mymodule"
     )

     func main() {
         fmt.Println(mymodule.Add(2, 3))
     }
     ```

```
 Go Module 發布與使用流程圖：

 ═══ 模組開發者 ═══                  ═══ 模組使用者 ═══

 ① 開發模組                         ④ 引用模組
 ┌──────────────────┐              ┌──────────────────┐
 │ mymodule/        │              │ myapp/            │
 │ ├── go.mod       │              │ ├── go.mod        │
 │ ├── math.go      │              │ └── main.go       │
 │ └── math_test.go │              │    import          │
 └──────────────────┘              │    "github.com/    │
                                   │     user/mymodule" │
 ② 測試通過                        └──────────────────┘
 go test ✓
                                   ⑤ go get 下載
 ③ 推送到 GitHub                          │
 git tag v1.0.0                           ▼
 git push origin v1.0.0           ┌──────────────────┐
       │                          │  Go Module Proxy  │
       ▼                          │  (快取 + 校驗)    │
 ┌──────────────────┐             └──────────────────┘
 │  GitHub Repo     │                     │
 │  v1.0.0 tag      │◀════════════════════┘
 └──────────────────┘        拉取模組

 版本號遵循語意化版本 (SemVer)：
 v1.0.0 → v1.0.1 (patch: bug fix)
 v1.0.0 → v1.1.0 (minor: 新功能，向下相容)
 v1.0.0 → v2.0.0 (major: 破壞性變更)
```

### 小結

Go Module 的設計大大改善了 Go 語言的依賴管理，讓開發者能夠輕鬆地管理版本和依賴。通過上述步驟，你可以創建自己的 Go 模組，並將它貢獻到 GitHub 上，與其他開發者共享你的工作。希望這篇文章能夠幫助你更好地理解和使用 Go Module！

  
