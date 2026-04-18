# Go 語言教學文件

## Go 風格指南 (繁體中文譯本)

本目錄收錄 Google 官方 Go Style Guide 的繁體中文翻譯,共四份核心文件:

[總覽](content/_index.md) | [指南](content/guide.md) | [決策](content/decisions.md) |
[最佳實踐](content/best-practices.md)

線上站台(GitHub Pages + Hugo Book theme):
<https://appleboy.github.io/go-tutorial/>

英文原版位於 <https://google.github.io/styleguide/go>,內容對應上游
[google/styleguide](https://github.com/google/styleguide/tree/gh-pages/go)
的 `gh-pages` 分支。

## 本地預覽

```bash
# 安裝 Hugo (macOS)
brew install hugo

# 首次 clone 需要初始化 theme submodule
git submodule update --init --recursive

# 啟動預覽 (http://localhost:1313)
hugo server
```

## 授權

英文原版由 Google LLC 以
[Creative Commons Attribution 3.0 (CC-BY-3.0)](https://creativecommons.org/licenses/by/3.0/)
授權,授權文字見本目錄下的 `LICENSE` 檔。

本繁體中文版本為依 CC-BY-3.0 條款衍生的翻譯著作,翻譯內容亦以相同條款釋出。
轉載時請保留原始來源與本翻譯出處,並註明「中文版本為翻譯之衍生著作」。
