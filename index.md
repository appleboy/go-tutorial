# Go 風格指南

<https://google.github.io/styleguide/go> (英文版)

[概覽](index) | [指南](guide) | [決策](decisions) |
[最佳實踐](best-practices)

<a id="about"></a>

## 關於

Go 風格指南及其附帶文件編碼了當前編寫可讀性和符合慣用語法的 Go 語言的最佳方法。遵循風格指南並非絕對要求，這些文件也永遠不會是全面的。我們的目的是最小化編寫可讀 Go 語言的猜測工作，以便新手能夠避免常見錯誤。風格指南還旨在統一 Google 內部審查 Go 代碼時給出的風格指導。

文件             | 鏈接                                                  | 主要受眾    | [規範性] | [典範性]
------------------- | ----------------------------------------------------- | ------------------- | ----------- | -----------
**風格指南**   | <https://google.github.io/styleguide/go/guide>          | 所有人            | Yes         | Yes
**風格決策** | <https://google.github.io/styleguide/go/decisions>      | 可讀性導師 | Yes         | No
**最佳實踐**  | <https://google.github.io/styleguide/go/best-practices> | 任何有興趣的人   | No          | No

[規範性]: #normative
[典範性]: #canonical

<a id="docs"></a>

### 文件

1. **[風格指南](https://google.github.io/styleguide/go/guide)** 概述了 Google 的 Go 風格基礎。此文件是決定性的，並用作風格決策和最佳實踐中建議的基礎。

1. **[風格決策](https://google.github.io/styleguide/go/decisions)** 是一份更詳細的文件，總結了特定風格點的決策，並在適當的情況下討論決策背後的原因。

    這些決策可能會根據新的數據、新的語言特性、新的庫或新興模式而偶爾變化，但不期望 Google 內部的個別 Go 程序員需要隨時更新此文件。

1. **[最佳實踐](https://google.github.io/styleguide/go/best-practices)** 文件記錄了一些隨著時間演變而解決常見問題、易於閱讀且對代碼維護需求健壯的模式。

    這些最佳實踐不是典範的，但鼓勵 Google 的 Go 程序員在可能的情況下使用它們，以保持代碼庫的統一性和一致性。

這些文件旨在：

* 就權衡不同風格的原則達成一致
* 編碼 Go 風格的既定事項
* 為 Go 慣用語法提供文件和典範示例
* 記錄各種風格決策的優缺點
* 幫助最小化 Go 可讀性審查中的驚喜
* 幫助可讀性導師使用一致的術語和指導

這些文件**不**旨在：

* 列出可在可讀性審查中給出的所有評論
* 列出所有人都預期記住並隨時遵循的所有規則
* 替代在使用語言特性和風格時的良好判斷
* 為了消除風格差異而辯解大規模變更

從一位 Go 程序員到另一位，從一個團隊的代碼庫到另一個，總會有差異。然而，對 Google 和 Alphabet 而言，我們的代碼庫盡可能一致是最好的。 (參見[指南](guide#consistency)以獲得更多關於一致性的信息。) 為此，隨時可以進行風格改進，但您不需要吹毛求疵地指出您發現的每一個風格指南違規情況。特別是，這些文件可能會隨著時間而變化，這不是在現有代碼庫中引起額外變動的理由；使用最新的最佳實踐編寫新代碼並隨時間解決附近問題就足夠了。

重要的是要認識到風格問題本質上是個人的，總是存在固有的權衡。這些文件中的許多指導是主觀的，但就像 `gofmt` 一樣，它們提供的統一性具有重大價值。因此，沒有充分討論，不會更改風格建議，即使在可能不同意的情況下，也鼓勵 Google 的 Go 程序員遵循風格指南。

<a id="definitions"></a>

## 定義

以下在風格文件中使用的詞語在下面定義：

* **典範性**：建立規範和持久的規則
    <a id="canonical"></a>

    在這些文件中，“典範性”用於描述被認為是所有代碼（新舊）都應遵循的標準，且不預期會隨時間大幅改變。典範文件中的原則應由作者和審查者共同理解，因此典範文件中包含的所有內容都必須達到高標準。因此，典範文件通常較短，且比非典範文件規定的風格元素更少。

    <https://google.github.io/styleguide/go#canonical>

* **規範性**：旨在建立一致性 <a id="normative"></a>

    在這些文件中，“規範性”用於描述某些被 Go 代碼審查者同意的風格元素，以便建議、術語和理由保持一致。這些元素可能會隨著時間而改變，這些文件將反映這種變化，以便審查者能夠保持一致性和最新狀態。Go 代碼的作者不預期熟悉規範文件，但審查者在可讀性審查中經常使用這些文件作為參考。

    <https://google.github.io/styleguide/go#normative>

* **慣用語**：常見且熟悉 <a id="idiomatic"></a>

    在這些文件中，“慣用語”用於指代在 Go 代碼中普遍存在的、已成為易於識別的熟悉模式的東西。一般來說，如果兩者在上下文中服務於相同的目的，則應優先選擇慣用模式而非非慣用模式，因為這是讀者最熟悉的。

    <https://google.github.io/styleguide/go#idiomatic>

<a id="references"></a>

## 附加參考

本指南假定讀者熟悉 [Effective Go]，因為它為整個 Go 社區的 Go 代碼提供了一個共同的基線。

以下是一些額外的資源，供那些希望自學 Go 風格的人以及希望在其審查中提供更多可鏈接上下文的審查者使用。參與 Go 可讀性過程的人不預期熟悉這些資源，但它們可能作為可讀性審查中的上下文出現。

[Effective Go]: https://go.dev/doc/effective_go

**外部參考**

* [Go 語言規範](https://go.dev/ref/spec)
* [Go FAQ](https://go.dev/doc/faq)
* [Go 內存模型](https://go.dev/ref/mem)
* [Go 數據結構](https://research.swtch.com/godata)
* [Go 接口](https://research.swtch.com/interfaces)
* [Go 諺語](https://go-proverbs.github.io/)
* <a id="gotip"></a> Go 提示集 - 敬請期待。
* <a id="unit-testing-practices"></a> 單元測試實踐 - 敬請期待。

**相關的 Testing-on-the-Toilet 文章**

* [TotT: 標識符命名][tott-431]
* [TotT: 測試狀態與測試互動][tott-281]
* [TotT: 有效測試][tott-324]
* [TotT: 風險驅動測試][tott-329]
* [TotT: 考慮有害的變更檢測測試][tott-350]

[tott-431]: https://testing.googleblog.com/2017/10/code-health-identifiernamingpostforworl.html
[tott-281]: https://testing.googleblog.com/2013/03/testing-on-toilet-testing-state-vs.html
[tott-324]: https://testing.googleblog.com/2014/05/testing-on-toilet-effective-testing.html
[tott-329]: https://testing.googleblog.com/2014/05/testing-on-toilet-risk-driven-testing.html
[tott-350]: https://testing.googleblog.com/2015/01/testing-on-toilet-change-detector-tests.html

**額外的外部著作**

* [Go 與教條](https://research.swtch.com/dogma)
* [少即是多](https://commandcenter.blogspot.com/2012/06/less-is-exponentially-more.html)
* [Esmerelda 的想像力](https://commandcenter.blogspot.com/2011/12/esmereldas-imagination.html)
* [用於解析的正則表達式](https://commandcenter.blogspot.com/2011/08/regular-expressions-in-lexing-and.html)
* [Gofmt 的風格沒有人最喜歡，但 Gofmt 是每個人的最愛](https://www.youtube.com/watch?v=PAAkCSZUG1c&t=8m43s)
    (YouTube)
