+++
title = '總覽'
type = 'docs'
weight = 0

[cascade]
  type = 'docs'
+++

<!--
  本文件譯自 https://google.github.io/styleguide/go,授權條款為
  Creative Commons Attribution 3.0 (CC-BY-3.0)。原文版權屬於 Google LLC,
  本中文版本為翻譯之衍生著作,僅供學習與內部參考。
-->

<https://google.github.io/styleguide/go>

[總覽](/) | [指南](/guide/) | [決策](/decisions/) |
[最佳實踐](/best-practices/)

<!--

-->

<a id="about"></a>

## 關於

Go 風格指南與其相關文件,記錄了 Google 內部目前撰寫易讀且符合 Go 慣用寫法的最佳做法。遵循風格指南並非絕對的硬性規定,這些文件也不可能涵蓋所有情境。我們的目的在於減少撰寫易讀 Go 程式碼時的猜測成本,讓初學者得以避開常見錯誤;同時讓 Google 內部所有審查 Go 程式碼的人,在風格指引上能保持一致。

| 文件         | 連結                                                  | 主要對象       | [規範性] | [典範性] |
| ------------ | ----------------------------------------------------- | -------------- | -------- | -------- |
| **風格指南** | <https://google.github.io/styleguide/go/guide>        | 所有人         | 是       | 是       |
| **風格決策** | <https://google.github.io/styleguide/go/decisions>    | 可讀性導師     | 是       | 否       |
| **最佳實踐** | <https://google.github.io/styleguide/go/best-practices> | 任何感興趣的人 | 否       | 否       |

[規範性]: #normative
[典範性]: #canonical

<a id="docs"></a>

### 文件

1. **[風格指南](https://google.github.io/styleguide/go/guide)** 概述 Google 內部 Go 風格的根本原則。本份文件具有最終決定權,亦作為「風格決策」與「最佳實踐」中各項建議的依據。

1. **[風格決策](https://google.github.io/styleguide/go/decisions)** 是一份內容較為詳盡的文件,彙整對特定風格議題所做的決定,並在適當之處說明決定背後的考量。

   這些決定可能會因應新資料、新語言特性、新函式庫或新興模式而調整,但 Google 內部一般 Go 工程師並不需要時時追蹤本文件的更新。

1. **[最佳實踐](https://google.github.io/styleguide/go/best-practices)** 收錄一些隨時間累積、用於解決常見問題、易讀且能適應後續維護需求的程式碼模式。

   這些最佳實踐並非典範規定,但鼓勵 Google 內部的 Go 工程師在合適情境下儘量採用,以維持整體程式碼庫的一致性與一致觀感。

這些文件意在:

- 對於不同風格選項之間的取捨原則達成共識
- 為已成定論的 Go 風格議題定下文字記錄
- 為 Go 慣用寫法提供說明與典範範例
- 記錄各項風格決定的優缺點
- 減少 Go 可讀性審查中出現意外
- 協助可讀性導師在用語與指引上保持一致

這些文件**不**意在:

- 列出在可讀性審查中可能出現的所有意見
- 列出所有人都應隨時記得並遵守的所有規則
- 取代運用語言特性與風格時的良好判斷
- 作為大規模改寫程式碼以消除風格差異的正當理由

不同的 Go 工程師之間、不同團隊的程式碼庫之間,永遠會存在差異。然而,對 Google 與 Alphabet 來說,讓我們的程式碼庫盡可能一致是符合整體利益的。 (關於一致性的更多說明,請參考[指南](/guide/#consistency)。) 因此,你可以隨時做出你認為合適的風格改進,但不必對找到的每一個風格指南違反之處都吹毛求疵。特別是,這些文件本身會隨時間調整,但這並不構成在既有程式碼庫中製造額外動盪的理由;只要新程式碼採用最新的最佳實踐撰寫,並隨時間就近處理鄰近問題即可。

我們也應認識到,風格議題本質上帶有個人色彩,而且永遠存在固有的取捨。本系列文件中的多數指引帶有主觀成分,但就像 `gofmt` 一樣,它們所帶來的一致性具有相當高的價值。基於此,風格建議在未經充分討論前不會更動;即使 Google 內部 Go 工程師個人有不同意見,也鼓勵遵循風格指南。

<a id="definitions"></a>

## 定義

下列在風格文件中反覆出現的詞彙,定義如下:

- **典範性 (Canonical)**:確立規範性且具持久性的規則
  <a id="canonical"></a>

  在這些文件中,「典範性」用來描述被視為標準、所有程式碼 (新舊皆然) 都應遵守、且預期在較長時間內不會大幅變動的內容。典範文件中的原則,撰寫者與審查者都應同樣理解,因此典範文件中的所有內容都必須達到較高的門檻。也因此,典範文件通常較短,所規範的風格元素也少於非典範文件。

  <https://google.github.io/styleguide/go#canonical>

- **規範性 (Normative)**:意在建立一致性 <a id="normative"></a>

  在這些文件中,「規範性」用來描述某項已由 Go 程式碼審查者達成共識的風格元素,讓建議內容、用語與理由能維持一致。這些元素可能隨時間調整,文件也會反映這些變化,讓審查者得以保持一致與最新。Go 程式碼的撰寫者並不需要熟悉規範性文件,但審查者在進行可讀性審查時會常常引用這些文件作為參考。

  <https://google.github.io/styleguide/go#normative>

- **慣用 (Idiomatic)**:常見且熟悉 <a id="idiomatic"></a>

  在這些文件中,「慣用」用來指 Go 程式碼中普遍出現、已成為易於辨識且熟悉的模式。一般而言,如果慣用模式與非慣用模式在語境中達成相同目的,應優先採用慣用模式,因為這對讀者而言最為熟悉。

  <https://google.github.io/styleguide/go#idiomatic>

<a id="references"></a>

## 補充參考資料

本指南假設讀者已熟悉 [Effective Go],因為它為整個 Go 社群的程式碼提供了共同的基準。

以下提供一些額外資源,供希望自學 Go 風格的讀者,以及希望在審查中提供更多可連結內容的審查者使用。參與 Go 可讀性流程的人並不需要熟悉這些資源,但它們可能會在可讀性審查中作為背景出現。

[Effective Go]: https://go.dev/doc/effective_go

**外部參考資料**

- [Go 語言規範](https://go.dev/ref/spec)
- [Go FAQ](https://go.dev/doc/faq)
- [Go 記憶體模型](https://go.dev/ref/mem)
- [Go 資料結構](https://research.swtch.com/godata)
- [Go 介面](https://research.swtch.com/interfaces)
- [Go 諺語](https://go-proverbs.github.io/)

- <a id="gotip"></a> Go Tip 系列 — 敬請期待。

- <a id="unit-testing-practices"></a> 單元測試實踐 — 敬請期待。

**相關 Testing-on-the-Toilet 文章**

- [TotT:識別字命名][tott-431]
- [TotT:測試狀態 vs. 測試互動][tott-281]
- [TotT:有效的測試][tott-324]
- [TotT:風險導向測試][tott-329]
- [TotT:變更偵測測試的有害之處][tott-350]

[tott-431]: https://testing.googleblog.com/2017/10/code-health-identifiernamingpostforworl.html
[tott-281]: https://testing.googleblog.com/2013/03/testing-on-toilet-testing-state-vs.html
[tott-324]: https://testing.googleblog.com/2014/05/testing-on-toilet-effective-testing.html
[tott-329]: https://testing.googleblog.com/2014/05/testing-on-toilet-risk-driven-testing.html
[tott-350]: https://testing.googleblog.com/2015/01/testing-on-toilet-change-detector-tests.html

**其他延伸閱讀**

- [Go 與教條 (Go and Dogma)](https://research.swtch.com/dogma)
- [Less is exponentially more](https://commandcenter.blogspot.com/2012/06/less-is-exponentially-more.html)
- [Esmerelda's Imagination](https://commandcenter.blogspot.com/2011/12/esmereldas-imagination.html)
- [用於剖析的正規表達式 (Regular expressions for parsing)](https://commandcenter.blogspot.com/2011/08/regular-expressions-in-lexing-and.html)
- [Gofmt 的風格沒人最愛,但 Gofmt 卻是大家最愛 (YouTube)](https://www.youtube.com/watch?v=PAAkCSZUG1c&t=8m43s)

<!--

-->
