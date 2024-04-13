<!--* toc_depth: 3 *-->

# Go 風格指南

https://google.github.io/styleguide/go/guide (英文版)

[概覽](index) | [指南](guide) | [決策](decisions) |
[最佳實踐](best-practices)

**注意：** 這是一系列概述 Google 的 [Go 風格](index) 文件的一部分。本文件是 **[規範性](index#normative) 和 [典範性](index#canonical)** 的。更多資訊請見[概覽](index#about)。

<a id="principles"></a>

## 風格原則

有幾個總體原則概括了如何思考編寫可讀的 Go 代碼。以下是按重要性排序的可讀代碼的屬性：

1.  **[清晰性][Clarity]**：代碼的目的和理由對讀者來說是清楚的。
1.  **[簡潔性][Simplicity]**：代碼以最簡單的方式實現其目標。
1.  **[簡練性][Concision]**：代碼具有高信噪比。
1.  **[可維護性][Maintainability]**：代碼的編寫使其易於維護。
1.  **[一致性][Consistency]**：代碼與更廣泛的 Google 代碼庫保持一致。

[Clarity]: #clarity
[Simplicity]: #simplicity
[Concision]: #concision
[Maintainability]: #maintainability
[Consistency]: #consistency

<a id="clarity"></a>

### 清晰性

可讀性的核心目標是產生對讀者來說清晰的代碼。

清晰性主要通過有效的命名、有幫助的評論和高效的代碼組織來實現。

清晰性應該從讀者的角度來看，而不是代碼的作者。代碼易於閱讀比易於編寫更重要。代碼的清晰性有兩個不同的方面：

*   [代碼實際上在做什麼？](#clarity-purpose)
*   [代碼為什麼要這樣做？](#clarity-rationale)

<a id="clarity-purpose"></a>

#### 代碼實際上在做什麼？

Go 的設計使得應該相對直觀地看到代碼在做什麼。在不確定的情況下，或者當讀者可能需要先驗知識才能理解代碼時，值得投入時間使代碼的目的對未來的讀者更清晰。例如，可能有助於：

*   使用更描述性的變量名
*   添加額外的評論
*   用空白和評論分隔代碼
*   將代碼重構為單獨的函數/方法使其更模塊化

這裡沒有一刀切的方法，但在開發 Go 代碼時優先考慮清晰性是重要的。

<a id="clarity-rationale"></a>

#### 代碼為什麼要這樣做？

代碼的理由通常通過變量、函數、方法或包的名稱充分傳達。在不是的情況下，添加評論很重要。當代碼包含讀者可能不熟悉的細微差別時，“為什麼？”尤其重要，例如：

*   語言的一個細微差別，例如，一個閉包將捕獲一個循環變量，但閉包距離很遠
*   業務邏輯的一個細微差別，例如，一個需要區分實際用戶和冒充用戶的訪問控制檢查

一個 API 可能需要小心使用。例如，一段代碼可能因為性能原因而錯綜複雜，難以跟隨，或者一系列複雜的數學操作可能以意想不到的方式使用類型轉換。在這些情況下，重要的是附帶的評論和文檔解釋這些方面，以便未來的維護者不會犯錯，並且讀者可以理解代碼而無需逆向工程。

同樣重要的是要意識到，一些試圖提供清晰性的嘗試（例如添加額外的評論）實際上可能會通過添加雜亂、重述代碼已經說的話、與代碼矛盾或增加維護負擔來保持評論的最新狀態來模糊代碼的目的。允許代碼自己說話（例如，使符號名稱本身自我描述）而不是添加多餘的評論。評論通常更好地解釋為什麼要這樣做，而不是代碼在做什麼。

Google 的代碼庫在很大程度上是統一和一致的。通常情況下，代碼脫穎而出（例如，通過使用不熟悉的模式）是出於好的原因，通常是出於性能。保持這一特性對於讓讀者清楚他們在閱讀新代碼時應該專注的地方很重要。

標準庫包含了許多這一原則的實例。其中包括：

*   [`package sort`](https://cs.opensource.google/go/go/+/refs/tags/go1.19.2:src/sort/sort.go) 中的維護者評論。
*   同一包中的好的
    [可運行示例](https://cs.opensource.google/go/go/+/refs/tags/go1.19.2:src/sort/example_search_test.go)，
    這對用戶（它們
    [顯示在 godoc 中](https://pkg.go.dev/sort#pkg-examples)）和維護者（它們 [作為測試的一部分運行](decisions#examples)）都有益。
*   [`strings.Cut`](https://pkg.go.dev/strings#Cut) 只有四行代碼，
    但它們改善了
    [調用點的清晰性和正確性](https://github.com/golang/go/issues/46336)。

<a id="simplicity"></a>

### 簡潔性

您的 Go 代碼應該對使用它、閱讀它和維護它的人來說是簡單的。

Go 代碼應該以實現其目標的最簡單方式編寫，無論是在行為還是性能方面。在 Google 的 Go 代碼庫中，簡單的代碼：

*   從上到下易於閱讀
*   不假設你已經知道它在做什麼
*   不假設你能記住所有前面的代碼
*   沒有不必要的抽象層次
*   沒有引起注意某些平凡事物的名稱
*   讓值和決策的傳播對讀者清晰
*   有評論解釋為什麼，而不是代碼在做什麼，以避免未來的偏差
*   有獨立的文檔
*   有有用的錯誤和有用的測試失敗
*   經常與“聰明”的代碼相互排斥

在代碼簡單性和 API 使用簡單性之間可能會出現權衡。例如，可能值得讓代碼更複雜，以便最終用戶更容易正確調用 API。相反，也可能值得留下一些額外的工作給 API 的最終用戶，以便代碼保持簡單易懂。

當代碼需要複雜性時，應該有意識地添加複雜性。這通常是必要的，如果需要額外的性能或有多個不同的客戶使用特定的庫或服務。複雜性可能是合理的，但它應該伴隨著相應的文檔，以便客戶和未來的維護者能夠理解和應對複雜性。這應該通過測試和示例補充，這些測試和示例展示了其正確的使用方式，特別是如果有“簡單”和“複雜”的使用代碼的方式。

這一原則並不意味著複雜的代碼不能或不應該用 Go 編寫，或者 Go 代碼不允許複雜。我們努力實現一個代碼庫，避免不必要的複雜性，以便當複雜性出現時，它表明有問題的代碼需要小心理解和維護。理想情況下，應該有附帶的評論解釋理由並識別應該採取的護理。這經常出現在為性能優化代碼時；這樣做通常需要更複雜的方法，比如預分配一個緩衝區並在一個 goroutine 的生命週期內重複使用它。當維護者看到這一點時，應該是一個線索，表明有問題的代碼是性能關鍵的，這應該影響未來變更時採取的護理。另一方面，如果不必要地使用，這種複雜性對於未來需要閱讀或更改代碼的人來說是一種負擔。

如果代碼的目的應該很簡單，但結果非常複雜，這通常是重新審視實現以查看是否有更簡單的方法來實現相同事物的信號。

<a id="least-mechanism"></a>

#### 最少機制

當有幾種方式可以表達相同的想法時，優先選擇使用最標準工具的那一個。復雜的機制經常存在，但不應該無故使用。向代碼中添加復雜性很容易，而在發現不必要後移除現有復雜性則要困難得多。

1.  當足夠滿足您的用例時，優先使用核心語言構造（例如通道、切片、映射、循環或結構）。
2.  如果沒有，尋找標準庫中的工具（如 HTTP 客戶端或模板引擎）。
3.  最後，考慮在引入新依賴或創建自己的東西之前，Google 代碼庫中是否有足夠的核心庫。

例如，考慮包含一個綁定到具有默認值的變量的標誌的生產代碼，該默認值必須在測試中覆蓋。除非打算測試程序的命令行界面本身（比如使用 `os/exec`），直接覆蓋綁定值而不是使用 `flag.Set` 更簡單，因此更可取。

同樣，如果一段代碼需要一個集合成員資格檢查，一個布爾值映射（例如 `map[string]bool`）通常就足夠了。只有在需要更複雜的操作且無法或過於複雜地使用映射時，才應該使用提供類似集合類型和功能的庫。

<a id="concision"></a>

### 簡練性

簡練的 Go 代碼具有高信噪比。相關細節容易辨識，命名和結構引導讀者了解這些細節。

有許多事情可能妨礙在任何特定時間顯示最突出的細節：

*   重複的代碼
*   多餘的語法
*   [不透明的名稱](#naming)
*   不必要的抽象
*   空白

重複的代碼尤其掩蓋了每個幾乎相同部分之間的差異，並要求讀者視覺比較類似的代碼行以找到變化。[表驅動測試]是一個好的機制，可以簡潔地從每次重複的重要細節中提取出共同的代碼，但選擇哪些部分包含在表中將影響表的易於理解程度。

在考慮多種結構代碼的方式時，值得考慮哪種方式使重要細節最為明顯。

理解和使用常見的代碼構造和慣用語也對於保持高信噪比很重要。例如，以下代碼塊在[錯誤處理]中非常常見，讀者可以快速理解這個塊的目的。

```go
// 好的：
if err := doSomething(); err != nil {
    // ...
}
```

If code looks very similar to this but is subtly different, a reader may not
notice the change. In cases like this, it is worth intentionally ["boosting"]
the signal of the error check by adding a comment to call attention to it.

```go
// Good:
if err := doSomething(); err == nil { // if NO error
    // ...
}
```

[Table-driven testing]: https://github.com/golang/go/wiki/TableDrivenTests
[error handling]: https://go.dev/blog/errors-are-values
["boosting"]: best-practices#signal-boost

<a id="maintainability"></a>

### 可維護性

代碼被編輯的次數比被寫的次數多得多。可讀的代碼不僅對試圖理解其運作方式的讀者有意義，對需要更改它的程序員也有意義。清晰是關鍵。

可維護的代碼:

* 方便未來的程序員正確修改
* 擁有結構化的 API，使其可以優雅地成長
* 清楚它所做的假設，選擇映射到問題結構而非代碼結構的抽象
* 避免不必要的耦合，不包括未使用的功能
* 擁有全面的測試套件以確保維護承諾的行為並確保重要邏輯正確，並且測試在失敗時提供清晰、可操作的診斷

在使用接口和類型等抽象時，這些抽象本質上從它們被使用的上下文中移除了資訊，確保它們提供足夠的好處很重要。當使用具體類型時，編輯器和 IDE 可以直接連接到方法定義並顯示相應的文檔，但否則只能參考接口定義。接口是一個強大的工具，但帶來了成本，因為維護者可能需要理解底層實現的具體情況才能正確使用接口，這必須在接口文檔或調用現場解釋。

可維護的代碼還避免在容易忽視的地方隱藏重要細節。例如，在以下每行代碼中，單個字符的存在或缺乏對於理解該行至關重要：

```go
// Bad:
// The use of = instead of := can change this line completely.
if user, err = db.UserByID(userID); err != nil {
    // ...
}
```

```go
// Bad:
// The ! in the middle of this line is very easy to miss.
leap := (year%4 == 0) && (!(year%100 == 0) || (year%400 == 0))
```

這兩者都不是錯誤的，但都可以用更明確的方式書寫，或者可以有一個附帶的評論來提醒注意重要的行為:

```go
// Good:
u, err := db.UserByID(userID)
if err != nil {
    return fmt.Errorf("invalid origin user: %s", err)
}
user = u
```

```go
// Good:
// Gregorian leap years aren't just year%4 == 0.
// See https://en.wikipedia.org/wiki/Leap_year#Algorithm.
var (
    leap4   = year%4 == 0
    leap100 = year%100 == 0
    leap400 = year%400 == 0
)
leap := leap4 && (!leap100 || leap400)
```

In the same way, a helper function that hides critical logic or an important
edge-case could make it easy for a future change to fail to account for it
properly.

Predictable names are another feature of maintainable code. A user of a package
or a maintainer of a piece of code should be able to predict the name of a
variable, method, or function in a given context. Function parameters and
receiver names for identical concepts should typically share the same name, both
to keep documentation understandable and to facilitate refactoring code with
minimal overhead.

Maintainable code minimizes its dependencies (both implicit and explicit).
Depending on fewer packages means fewer lines of code that can affect behavior.
Avoiding dependencies on internal or undocumented behavior makes code less
likely to impose a maintenance burden when those behaviors change in the future.

When considering how to structure or write code, it is worth taking the time to
think through ways in which the code may evolve over time. If a given approach
is more conducive to easier and safer future changes, that is often a good
trade-off, even if it means a slightly more complicated design.

<a id="consistency"></a>

### Consistency

Consistent code is code that looks, feels, and behaves like similar code
throughout the broader codebase, within the context of a team or package, and
even within a single file.

Consistency concerns do not override any of the principles above, but if a tie
must be broken, it is often beneficial to break it in favor of consistency.

Consistency within a package is often the most immediately important level of
consistency. It can be very jarring if the same problem is approached in
multiple ways throughout a package, or if the same concept has many names within
a file. However, even this should not override documented style principles or
global consistency.

<a id="core"></a>

## Core guidelines

These guidelines collect the most important aspects of Go style that all Go code
is expected to follow. We expect that these principles be learned and followed
by the time readability is granted. These are not expected to change frequently,
and new additions will have to clear a high bar.

The guidelines below expand on the recommendations in [Effective Go], which
provide a common baseline for Go code across the entire community.

[Effective Go]: https://go.dev/doc/effective_go

<a id="formatting"></a>

### Formatting

All Go source files must conform to the format outputted by the `gofmt` tool.
This format is enforced by a presubmit check in the Google codebase.
[Generated code] should generally also be formatted (e.g., by using
[`format.Source`]), as it is also browsable in Code Search.

[Generated code]: https://docs.bazel.build/versions/main/be/general.html#genrule
[`format.Source`]: https://pkg.go.dev/go/format#Source

<a id="mixed-caps"></a>

### MixedCaps

Go source code uses `MixedCaps` or `mixedCaps` (camel case) rather than
underscores (snake case) when writing multi-word names.

This applies even when it breaks conventions in other languages. For example, a
constant is `MaxLength` (not `MAX_LENGTH`) if exported and `maxLength` (not
`max_length`) if unexported.

Local variables are considered [unexported] for the purpose of choosing the
initial capitalization.

<!--#include file="/go/g3doc/style/includes/special-name-exception.md"-->

[unexported]: https://go.dev/ref/spec#Exported_identifiers

<a id="line-length"></a>

### Line length

There is no fixed line length for Go source code. If a line feels too long,
prefer refactoring instead of splitting it. If it is already as short as it is
practical for it to be, the line should be allowed to remain long.

Do not split a line:

*   Before an [indentation change](decisions#indentation-confusion) (e.g.,
    function declaration, conditional)
*   To make a long string (e.g., a URL) fit into multiple shorter lines

<a id="naming"></a>

### Naming

Naming is more art than science. In Go, names tend to be somewhat shorter than
in many other languages, but the same [general guidelines] apply. Names should:

*   Not feel [repetitive](decisions#repetition) when they are used
*   Take the context into consideration
*   Not repeat concepts that are already clear

You can find more specific guidance on naming in [decisions](decisions#naming).

[general guidelines]: https://testing.googleblog.com/2017/10/code-health-identifiernamingpostforworl.html

<a id="local-consistency"></a>

### Local consistency

Where the style guide has nothing to say about a particular point of style,
authors are free to choose the style that they prefer, unless the code in close
proximity (usually within the same file or package, but sometimes within a team
or project directory) has taken a consistent stance on the issue.

Examples of **valid** local style considerations:

*   Use of `%s` or `%v` for formatted printing of errors
*   Usage of buffered channels in lieu of mutexes

Examples of **invalid** local style considerations:

*   Line length restrictions for code
*   Use of assertion-based testing libraries

If the local style disagrees with the style guide but the readability impact is
limited to one file, it will generally be surfaced in a code review for which a
consistent fix would be outside the scope of the CL in question. At that point,
it is appropriate to file a bug to track the fix.

If a change would worsen an existing style deviation, expose it in more API
surfaces, expand the number of files in which the deviation is present, or
introduce an actual bug, then local consistency is no longer a valid
justification for violating the style guide for new code. In these cases, it is
appropriate for the author to clean up the existing codebase in the same CL,
perform a refactor in advance of the current CL, or find an alternative that at
least does not make the local problem worse.
