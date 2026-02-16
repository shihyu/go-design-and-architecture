# Day17 - 更多環境變數: GOOS, GOARCH等等

在使用 Go 語言開發應用程式時，環境變數扮演著關鍵的角色。它們幫助我們配置編譯器、執行環境、調試資訊等多個方面。理解這些變數的作用，可以讓我們更好地掌控 Go 開發環境，寫出更高效、更具跨平台性的程式。今天，我們就來聊聊一些重要的 Go 環境變數，比如 `GOOS` 和 `GOARCH`，以及它們在實際開發中的應用。

## 為什麼要了解環境變數？

首先，環境變數是一種簡單而強大的配置方式，讓我們能夠根據需要調整開發環境。例如，當你需要在 Windows 上開發，但希望你的程式碼最終執行在 Linux 伺服器上時，環境變數可以幫助你在開發過程中模擬不同的作業系統環境。

```
 Go 交叉編譯環境變數示意圖：

 開發機器 (macOS/amd64)
 ┌──────────────────────────────────────────────────────────┐
 │                                                          │
 │  GOOS=linux GOARCH=amd64  go build → myapp-linux        │
 │  GOOS=linux GOARCH=arm    go build → myapp-arm           │
 │  GOOS=windows GOARCH=amd64 go build → myapp.exe          │
 │                                                          │
 └──────────────────────┬───────────────────────────────────┘
                        │ 編譯輸出
           ┌────────────┼────────────┐
           ▼            ▼            ▼
 ┌──────────────┐ ┌──────────┐ ┌──────────────┐
 │ Linux Server │ │ RPi(ARM) │ │ Windows PC   │
 │ myapp-linux  │ │ myapp-arm│ │ myapp.exe    │
 └──────────────┘ └──────────┘ └──────────────┘

 環境變數層次結構：
 ┌─────────────────────────────────────────────┐
 │  GOROOT = /usr/local/go      (Go 安裝目錄)  │
 │  GOPATH = $HOME/go           (工作區目錄)    │
 │    ├── src/                  (原始碼)        │
 │    ├── pkg/                  (編譯快取)      │
 │    └── bin/                  (可執行檔)      │
 │  GOMOD  = /project/go.mod   (模組定義檔)    │
 │  GOGC   = 100                (GC 觸發閾值)  │
 └─────────────────────────────────────────────┘
```

## 常見的 Go 環境變數

### 1. `GOOS`

* **定義：** `GOOS` 是用來指定目標作業系統的環境變數。
* **可選值：** 常見值有 `linux`、`windows`、`darwin`（對應 macOS）等等。
* **用途：** 當我們需要交叉編譯時，設定 `GOOS` 可以讓 Go 編譯器為指定的作業系統生成可執行檔案。

#### 範例：

假設我們在 macOS 上開發，但需要生成一個可以在 Linux 上執行的程式。我們可以這樣做：

```go
GOOS=linux go build -o myapp-linux
```

這樣就會生成一個名為 `myapp-linux` 的可執行檔案，它可以直接在 Linux 系統上執行。

### 2. `GOARCH`

* **定義：** `GOARCH` 用來指定目標系統的架構。
* **可選值：** 常見值有 `amd64`、`386`、`arm`、`arm64` 等。
* **用途：** 這個變數特別有用於需要支持不同處理器架構的應用程式開發中。

#### 範例：

如果我們想要為 Raspberry Pi 編譯 Go 應用程式，我們可以這樣做：

```go
GOOS=linux GOARCH=arm go build -o myapp-arm
```

這將生成一個適用於 ARM 架構的可執行檔案，可以在 Raspberry Pi 上執行。

### 3. `GOPATH`

* **定義：** `GOPATH` 是 Go 工作區的路徑，包含三個子目錄：`src`、`pkg`、和 `bin`。
* **用途：** `GOPATH` 用於管理第三方套件和工具。如果你的專案不在 `GOPATH` 目錄下，那麼你就需要設定這個變數以包含你的專案路徑。

#### 範例：

```go
export GOPATH=$HOME/go
```

設定好 `GOPATH` 後，你的 Go 專案應該放在 `$HOME/go/src` 目錄中。

### 4. `GOROOT`

* **定義：** `GOROOT` 是 Go 編譯器和工具鏈的安裝目錄。
* **用途：** 在大多數情況下，`GOROOT` 是不需要手動設定的，因為 Go 會自動偵測安裝位置。但在某些情況下（例如多個 Go 版本並存時），你可能需要手動設定。

#### 範例：

```go
export GOROOT=/usr/local/go
```

### 5. `GOMOD`

* **定義：** `GOMOD` 代表當前專案的 `go.mod` 文件的路徑。
* **用途：** 當你想確認當前的工作目錄是否在一個 Go 模組中時，這個變數非常有用。

#### 範例：

在終端中執行：

```go
go env GOMOD
```

會返回當前模組的 `go.mod` 文件路徑。

## 總結

1. 跨平台與跨架構編譯：

* 使用 GOOS 和 GOARCH 環境變數，可以為不同作業系統（如 Linux、Windows）和硬體架構（如 amd64、arm）進行交叉編譯，生成適用於不同平台的可執行檔案。
* 範例指令：GOOS=linux GOARCH=arm go build -o myapp-arm 可生成適用於 Linux 和 ARM 架構的應用程式。

2. 工作空間與模組管理：

* GOPATH 定義了 Go 工作空間的路徑，用於管理專案和第三方套件。在Module下，GOMOD 用於標識當前專案的模組文件 go.mod，便於模組化管理。
* 設定範例：export GOPATH=$HOME/go 會將工作區設定在 $HOME/go 下。

3. 編譯器與工具鏈配置：

* GOROOT 指定了 Go 編譯器和工具鏈的安裝目錄，通常不需手動設定，但在多版本並存時可能需要手動指定。
* 設定範例：export GOROOT=/usr/local/go 用於手動設定 Go 工具鏈的路徑。

更多Go語言的文章歡迎參閱我的部落格: <https://kaichiachen.github.io/2024/04/30/golang/go_env_var/>

  
