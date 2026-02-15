# Go語言設計與架構之美

以 30 天系列文章，深入探討 Go 語言的設計哲學、架構模式與實務應用。

原文來源：[iT邦幫忙鐵人賽](https://ithelp.ithome.com.tw/users/20130271/ironman/7195)

## 內容大綱

| 天數 | 主題 |
|------|------|
| Day 1-2 | 開篇與設計哲學 |
| Day 3-5 | 程式專案結構、生命週期、Defer |
| Day 6-10 | Method 與 Interface |
| Day 11-14 | 併發：Goroutine、Channel、sync |
| Day 15-18 | 模組管理、單元測試、環境變數、性能分析 |
| Day 19-20 | 記憶體管理與開發技巧 |
| Day 21-25 | 依賴注入、Web 應用、Uber Library、Go Module、Container/Microservice |
| Day 26-29 | 壓力測試工具、Pool 實作、Go vs Rust、Go 與 AI |
| Day 30 | 結語 |

## 建置與閱讀

本專案使用 [mdbook](https://rust-lang.github.io/mdBook/) 產生靜態網頁。

```bash
# 安裝 mdbook
cargo install mdbook

# 本地預覽（含即時重載）
make serve

# 建置靜態網頁
make build

# 清理建置產物
make clean
```
