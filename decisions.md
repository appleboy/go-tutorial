<!--* toc_depth: 3 *-->

# Go 風格決策

https://google.github.io/styleguide/go/decisions (英文版)

[概覽](index) | [指南](guide) | [決策](decisions) |
[最佳實踐](best-practices)

**注意：** 這是一系列概述 Google 的 [Go 風格](index) 文件的一部分。本文件是 **[規範性](index#normative) 但非 [典範性](index#canonical)**，並且從屬於 [核心風格指南](guide)。更多資訊請見[概覽](index#about)。

<a id="about"></a>

## 關於

本文件包含旨在統一並提供標準指導、解釋和範例的風格決策，以供 Go 可讀性導師提供建議。

本文件**非詳盡無遺**，將隨著時間的推移而增長。在 [核心風格指南](guide) 與此處提供的建議相矛盾的情況下，**風格指南優先**，並且本文件應相應更新。

查看 [概覽](https://google.github.io/styleguide/go#about) 以獲得完整的 Go 風格文件集。

以下部分已從風格決策移至指南的其他部分：

*   **混合大小寫**：見 [指南#混合大小寫](guide#mixed-caps)
    <a id="mixed-caps"></a>

*   **格式化**：見 [指南#格式化](guide#formatting)
    <a id="formatting"></a>

*   **行長**：見 [指南#行長](guide#line-length)
    <a id="line-length"></a>

<a id="naming"></a>

## 命名

有關命名的總體指導，請參見 [核心風格指南](guide#naming) 中的命名部分。以下部分對命名的特定領域提供了進一步的澄清。

<a id="underscores"></a>

### 底線

Go 中的名稱一般不應包含底線。這個原則有三個例外：

1.  僅由生成的代碼導入的套件名稱可能包含底線。有關如何選擇多詞套件名稱的更多細節，請參見 [套件名稱](#package-names)。
1.  `*_test.go` 文件中的測試、基準測試和示例函數名稱可能包含底線。
1.  與操作系統或 cgo 互操作的低級庫可能會重用標識符，如 [`syscall`] 所做的那樣。這在大多數代碼庫中預期非常罕見。

[`syscall`]: https://pkg.go.dev/syscall#pkg-constants

<a id="package-names"></a>

### 套件名稱

<a id="TOC-PackageNames"></a>

Go 套件名稱應該短且僅包含小寫字母。由多個單詞組成的套件名稱應該保持不間斷的全小寫。例如，套件 [`tabwriter`] 不是命名為 `tabWriter`、`TabWriter` 或 `tab_writer`。

避免選擇可能被常用的局部變量名稱 [遮蔽] 的套件名稱。例如，`usercount` 是一個比 `count` 更好的套件名稱，因為 `count` 是一個常用的變量名稱。

Go 套件名稱不應有底線。如果您需要導入一個包含底線的套件名稱（通常來自生成的或第三方代碼），則必須在導入時重命名為適合在 Go 代碼中使用的名稱。

此規則的例外是，僅由生成的代碼導入的套件名稱可能包含底線。具體例子包括：

*   使用 `_test` 後綴的外部測試套件，例如集成測試

*   使用 `_test` 後綴的
    [套件級文檔示例](https://go.dev/blog/examples)

[`tabwriter`]: https://pkg.go.dev/text/tabwriter
[遮蔽]: best-practices#shadowing

避免使用像 `util`、`utility`、`common`、`helper` 等無信息性的套件名稱。更多關於
[所謂的 "工具套件"](best-practices#util-packages)。

當導入的套件被重命名時（例如 `import foopb "path/to/foo_go_proto"`），套件的本地名稱必須遵守上述規則，因為本地名稱決定了文件中如何引用該套件中的符號。如果在多個文件中重命名了給定的導入，特別是在相同或相鄰的套件中，應盡可能使用相同的本地名稱以保持一致性。

<!--#include file="/go/g3doc/style/includes/special-name-exception.md"-->

另見：[Go 博客關於套件名稱的文章](https://go.dev/blog/package-names)。

<a id="receiver-names"></a>

### 接收者名稱

<a id="TOC-ReceiverNames"></a>

[接收者] 變量名稱必須是：

*   短的（通常是一到兩個字母長）
*   該類型本身的縮寫
*   對該類型的每個接收者一致應用

Long Name                   | Better Name
--------------------------- | -------------------------
`func (tray Tray)`          | `func (t Tray)`
`func (info *ResearchInfo)` | `func (ri *ResearchInfo)`
`func (this *ReportWriter)` | `func (w *ReportWriter)`
`func (self *Scanner)`      | `func (s *Scanner)`

[接收者]: https://golang.org/ref/spec#Method_declarations

<a id="constant-names"></a>

### 常量名稱

常量名稱必須使用 [混合大小寫]，就像 Go 中的所有其他名稱一樣。（[導出] 的常量以大寫開頭，而未導出的常量以小寫開頭。）即使這樣做打破了其他語言中的慣例也適用。常量名稱不應該是其值的衍生物，而應該解釋該值表示什麼。

```go
// 好的：
const MaxPacketSize = 512

const (
    ExecuteBit = 1 << iota
    WriteBit
    ReadBit
)
```

[混合大小寫]: guide#mixed-caps
[導出]: https://tour.golang.org/basics/3

不要使用非混合大小寫的常量名稱或帶有 `K` 前綴的常量。

```go
// 不好的範例:
const MAX_PACKET_SIZE = 512
const kMaxBufferSize = 1024
const KMaxUsersPergroup = 500
```

根據常量的角色命名，而不是它們的值。如果一個常量除了它的值之外沒有其他角色，那麼將它定義為常量是不必要的。

```go
// 不好的範例
const Twelve = 12

const (
    UserNameColumn = "username"
    GroupColumn    = "group"
)
```

<!--#include file="/go/g3doc/style/includes/special-name-exception.md"-->

<a id="initialisms"></a>

### 首字母縮略詞

<a id="TOC-Initialisms"></a>

名稱中作為首字母縮略詞或縮寫的單詞（例如，`URL` 和 `NATO`）應該具有相同的大小寫。`URL` 應該出現為 `URL` 或 `url`（如在 `urlPony` 或 `URLPony` 中），永遠不應該是 `Url`。這也適用於 `ID` 當它是 "identifier" 的縮寫時；寫作 `appID` 而不是 `appId`。

*   在包含多個首字母縮略詞的名稱中（例如 `XMLAPI` 因為它包含 `XML` 和 `API`），給定首字母縮略詞內的每個字母應該具有相同的大小寫，但名稱中的每個首字母縮略詞不需要具有相同的大小寫。
*   在包含含有小寫字母的首字母縮略詞的名稱中（例如 `DDoS`、`iOS`、`gRPC`），首字母縮略詞應該出現如同在標準散文中一樣，除非您需要為了 [導出性] 改變第一個字母。在這些情況下，整個首字母縮略詞應該是相同的大小寫（例如 `ddos`、`IOS`、`GRPC`）。

[導出性]: https://golang.org/ref/spec#Exported_identifiers

<!-- 保持這個表格窄。如果必須變寬，用列表替換。 -->

首字母縮略詞 | 範圍       | 正確      | 錯誤
------------- | ---------- | -------- | --------------------------------------
XML API       | 導出       | `XMLAPI` | `XmlApi`, `XMLApi`, `XmlAPI`, `XMLapi`
XML API       | 未導出     | `xmlAPI` | `xmlapi`, `xmlApi`
iOS           | 導出       | `IOS`    | `Ios`, `IoS`
iOS           | 未導出     | `iOS`    | `ios`
gRPC          | 導出       | `GRPC`   | `Grpc`
gRPC          | 未導出     | `gRPC`   | `grpc`
DDoS          | 導出       | `DDoS`   | `DDOS`, `Ddos`
DDoS          | 未導出     | `ddos`   | `dDoS`, `dDOS`

<!--#include file="/go/g3doc/style/includes/special-name-exception.md"-->

<a id="getters"></a>

### Getters

<a id="TOC-Getters"></a>

函數和方法名稱不應使用 `Get` 或 `get` 前綴，除非底層概念使用了 "get" 這個詞（例如一個 HTTP GET）。優先直接以名詞開頭命名，例如使用 `Counts` 而不是 `GetCounts`。

如果函數涉及執行複雜計算或執行遠程調用，可以使用 `Compute` 或 `Fetch` 等不同的詞代替 `Get`，以便讀者清楚地知道函數調用可能需要時間，並且可能會阻塞或失敗。

<!--#include file="/go/g3doc/style/includes/special-name-exception.md"-->

<a id="variable-names"></a>

### 變量名稱

<a id="TOC-VariableNames"></a>

一般的經驗法則是，名稱的長度應該與其範圍的大小成正比，與它在該範圍內使用的次數成反比。在文件範圍創建的變量可能需要多個單詞，而在單個內部塊範圍內的變量可能只是一個單詞甚至只是一兩個字符，以保持代碼清晰並避免多餘的資訊。

這裡是一個大致的基線。這些數字指南不是嚴格的規則。基於上下文、[清晰性] 和 [簡練性] 應用判斷。

*   小範圍是執行一兩個小操作的範圍，比如 1-7 行。
*   中等範圍是幾個小操作或一個大操作，比如 8-15 行。
*   大範圍是一個或幾個大操作，比如 15-25 行。
*   非常大的範圍是跨越超過一頁的任何範圍（比如，超過 25 行）。

[清晰性]: guide#clarity
[簡練性]: guide#concision

在小範圍內可能非常清晰的名稱（例如，`c` 用於計數器）在更大的範圍內可能不夠，並且需要澄清以提醒讀者在代碼中更遠的地方其用途。在有許多變量的範圍內，或者變量代表相似的值或概念，可能需要比範圍建議的更長的變量名稱。

概念的具體性也可以幫助保持變量名稱的簡練。例如，假設只使用一個數據庫，一個短變量名稱如 `db` 通常可能保留給非常小的範圍，即使範圍非常大也可能保持完全清晰。在這種情況下，單詞 `database` 基於範圍的大小可能是可以接受的，但不是必需的，因為 `db` 是該單詞的非常常見的縮寫，幾乎沒有其他解釋。

局部變量的名稱應該反映它包含的內容以及它在當前上下文中的使用方式，而不是值的來源。例如，通常情況下，最好的局部變量名稱與結構或協議緩衝區字段名稱不同。

一般來說：

*   單詞名稱如 `count` 或 `options` 是一個好的起點。
*   可以添加額外的單詞來消除相似名稱的歧義，例如 `userCount` 和 `projectCount`。
*   不要簡單地刪除字母來節省打字。例如，`Sandbox` 比 `Sbx` 更好，特別是對於導出的名稱。
*   從大多數變量名稱中省略 [類型和類型化的詞]。
    *   對於一個數字，`userCount` 是一個比 `numUsers` 或 `usersInt` 更好的名稱。
    *   對於一個切片，`users` 是一個比 `userSlice` 更好的名稱。
    *   如果範圍內有兩個版本的值，可以接受包含類型化的限定詞，例如你可能有一個存儲在 `ageString` 中的輸入，並使用 `age` 作為解析後的值。
*   省略從 [周圍上下文] 中清楚的詞。例如，在 `UserCount` 方法的實現中，一個叫做 `userCount` 的局部變量可能是多餘的；`count`、`users` 或甚至 `c` 一樣可讀。

[類型和類型化的詞]: #repetitive-with-type
[周圍上下文]: #repetitive-in-context

<a id="v"></a>

#### 單字母變量名稱

單字母變量名稱可以是一個有用的工具來最小化
[重複](#repetition)，但也可能使代碼不必要地晦澀。將它們的使用限制在完整單詞是明顯的情況下，並且如果單字母變量代替完整單詞會重複。

一般來說：

*   對於 [方法接收者變量]，一個字母或兩個字母的名稱是首選。
*   對於常見類型使用熟悉的變量名稱通常很有幫助：
    *   `r` 用於 `io.Reader` 或 `*http.Request`
    *   `w` 用於 `io.Writer` 或 `http.ResponseWriter`
*   單字母標識符作為整數循環變量是可以接受的，特別是對於索引（例如，`i`）和坐標（例如，`x` 和 `y`）。
*   當範圍短時，縮寫可以作為可接受的循環標識符，例如 `for _, n := range nodes { ... }`。

[方法接收者變量]: #receiver-names

<a id="repetition"></a>

### 重複

<!--
未來編輯者注意：

不要使用 "stutter" 這個詞來指代名稱重複的情況。
-->

Go 源代碼應該避免不必要的重複。一個常見的來源是重複的名稱，這些名稱經常包含不必要的詞或重複它們的上下文或類型。如果相同或類似的代碼段在接近的位置多次出現，代碼本身也可能是不必要地重複的。

重複命名可以有多種形式，包括：

<a id="repetitive-with-package"></a>

#### 套件與導出符號名稱

在命名導出符號時，套件名稱在套件外部始終可見，因此應該減少或消除兩者之間的冗餘資訊。如果一個套件只導出一種類型，並且它以套件本身命名，如果需要構造函數，則標準名稱為 `New`。

> **例子：** 重複的名稱 -> 更好的名稱
>
> *   `widget.NewWidget` -> `widget.New`
> *   `widget.NewWidgetWithName` -> `widget.NewWithName`
> *   `db.LoadFromDatabase` -> `db.Load`
> *   `goatteleportutil.CountGoatsTeleported` -> `gtutil.CountGoatsTeleported`
>     或 `goatteleport.Count`
> *   `myteampb.MyTeamMethodRequest` -> `mtpb.MyTeamMethodRequest` 或
>     `myteampb.MethodRequest`

<a id="repetitive-with-type"></a>

#### 套件與導出符號名稱

在命名導出符號時，套件的名稱在套件外總是可見的，因此兩者之間的冗餘信息應該被減少或消除。如果一個套件只導出一種類型，且以套件本身命名，則構造函數的標準名稱為 `New`（如果需要的話）。

> **範例：** 重複的名稱 -> 更好的名稱
>
> *   `widget.NewWidget` -> `widget.New`
> *   `widget.NewWidgetWithName` -> `widget.NewWithName`
> *   `db.LoadFromDatabase` -> `db.Load`
> *   `goatteleportutil.CountGoatsTeleported` -> `gtutil.CountGoatsTeleported`
>     或 `goatteleport.Count`
> *   `myteampb.MyTeamMethodRequest` -> `mtpb.MyTeamMethodRequest` 或
>     `myteampb.MethodRequest`

<a id="repetitive-with-type"></a>

#### 變量名稱與類型

編譯器總是知道一個變量的類型，在大多數情況下，讀者也能通過變量的使用方式清楚地知道變量的類型。只有在同一作用域內變量的值出現兩次時，才需要明確變量的類型。

重複的名稱               | 更好的名稱
----------------------------- | ----------------------
`var numUsers int`            | `var users int`
`var nameString string`       | `var name string`
`var primaryProject *Project` | `var primary *Project`

如果值以多種形式出現，可以通過額外的單詞如 `raw` 和 `parsed` 或底層表示來進行說明：

```go
// 好的範例:
limitStr := r.FormValue("limit")
limit, err := strconv.Atoi(limitStr)
```

<a id="repetitive-in-context"></a>

#### 外部上下文與本地名稱

包含來自其周圍上下文信息的名稱經常創造額外的噪音而沒有好處。套件名稱、方法名稱、類型名稱、函數名稱、導入路徑，甚至檔案名稱都可以提供自動資格所有內部名稱的上下文。

```go
// 不好的範例:
// 在套件 "ads/targeting/revenue/reporting"
type AdsTargetingRevenueReport struct{}

func (p *Project) ProjectName() string
```

```go
// 好的範例:
// 在套件 "ads/targeting/revenue/reporting"
type Report struct{}

func (p *Project) Name() string
```

```go
// 不好的範例:
// 在套件 "sqldb"
type DBConnection struct{}
```

```go
// 好的範例:
// 在套件 "sqldb"
type Connection struct{}
```

```go
// 不好的範例:
// 在套件 "ads/targeting"
func Process(in *pb.FooProto) *Report {
    adsTargetingID := in.GetAdsTargetingID()
}
```

```go
// 好的範例:
// 在套件 "ads/targeting"
func Process(in *pb.FooProto) *Report {
    id := in.GetAdsTargetingID()
}
```

重複通常應該在使用符號的用戶的上下文中評估，而不是孤立地。例如，以下代碼有很多名稱在某些情況下可能是好的，但在上下文中是多餘的：

```go
// 不好的範例:
func (db *DB) UserCount() (userCount int, err error) {
    var userCountInt64 int64
    if dbLoadError := db.LoadFromDatabase("count(distinct users)", &userCountInt64); dbLoadError != nil {
        return 0, fmt.Errorf("failed to load user count: %s", dbLoadError)
    }
    userCount = int(userCountInt64)
    return userCount, nil
}
```

相反，從上下文或使用中清楚的名稱信息通常可以省略：

```go
// 好的範例:
func (db *DB) UserCount() (int, error) {
    var count int64
    if err := db.Load("count(distinct users)", &count); err != nil {
        return 0, fmt.Errorf("failed to load user count: %s", err)
    }
    return int(count), nil
}
```

<a id="commentary"></a>

## 註解

關於註解的慣例（包括該評論什麼、使用什麼風格、如何提供可運行的範例等）旨在支持閱讀公共 API 文檔的體驗。有關更多信息，請參見 [Effective Go](http://golang.org/doc/effective_go.html#commentary)。

最佳實踐文件的 [文檔慣例] 部分進一步討論了這個問題。

**最佳實踐：** 在開發和代碼審查期間使用 [doc preview]，以查看文檔和可運行範例是否有用，並且是否按照您期望的方式呈現。

**提示：** Godoc 使用的特殊格式化非常少；列表和代碼片段通常應該縮進以避免換行。除了縮進，通常應避免裝飾。

[doc preview]: best-practices#documentation-preview
[文檔慣例]:  best-practices#documentation-conventions

<a id="comment-line-length"></a>

### 註解行長

確保即使在窄屏幕上也能從源代碼中閱讀註解。

當註解太長時，建議將其拆分為多個單行註解。盡可能瞄準在 80 列寬的終端上閱讀良好的註解，但這不是一個硬性截止；Go 中沒有固定的註解行長限制。例如，標準庫通常選擇根據標點符號來斷開註解，這有時會使個別行接近 60-70 個字符標記。

有很多現有代碼中的註解超過了 80 個字符的長度。這個指南不應該被用作在可讀性審查中更改此類代碼的理由（參見 [一致性](guide#consistency)），儘管鼓勵團隊在其他重構的過程中抓住機會更新註解以遵循這一指南。這個指南的主要目標是確保所有 Go 可讀性導師在提出建議時做出相同的建議。

有關註解的更多信息，請參見 [The Go Blog 上的文章關於文檔]。

[The Go Blog 上的文章關於文檔]: https://blog.golang.org/godoc-documenting-go-code

```text
# 好的範例:
// 這是一段註解。
// 個別行的長度在 Godoc 中並不重要；
// 但選擇換行的方式使其在窄屏幕上易於閱讀。
//
// 不用太擔心長 URL：
// https://supercalifragilisticexpialidocious.example.com:8080/Animalia/Chordata/Mammalia/Rodentia/Geomyoidea/Geomyidae/
//
// 同樣，如果您有其他信息因過多的換行而變得尷尬，
// 請運用您的判斷，如果包含長行有助於而不是妨礙，則包含它。
```

避免會在小屏幕上反复換行的註解，這會導致糟糕的閱讀體驗。

```text
# 不好的範例:
// 這是一段註解。個別行的長度在 Godoc 中並不重要；
// 但選擇換行的方式會在窄屏幕或代碼審查中造成參差不齊的行，
// 這可能很煩人，尤其是在一個會反复換行的註解塊中。
//
// 不用太擔心長 URL：
// https://supercalifragilisticexpialidocious.example.com:8080/Animalia/Chordata/Mammalia/Rodentia/Geomyoidea/Geomyidae/
```

<a id="doc-comments"></a>

### 文件註解

<a id="TOC-DocComments"></a>

所有頂層導出名稱必須有文件註解，未導出的類型或函數聲明如果行為或含義不明顯也應該有。這些註解應該是以被描述對象的名稱開頭的[完整句子]。名稱前可以加上冠詞（"a", "an", "the"）使其讀起來更自然。

```go
// 好的範例:
// A Request represents a request to run a command.
type Request struct { ...

// Encode writes the JSON encoding of req to w.
func Encode(w io.Writer, req *Request) { ...
```

文件註解出現在 [Godoc](https://pkg.go.dev/) 中，並且會被 IDE 展示，因此應該為使用套件的任何人編寫。

[完整句子]: #comment-sentences

文件註解適用於以下符號，或者如果它出現在結構體中則適用於字段群。

```go
// 好的範例:
// Options configure the group management service.
type Options struct {
    // General setup:
    Name  string
    Group *FooGroup

    // Dependencies:
    DB *sql.DB

    // Customization:
    LargeGroupThreshold int // optional; default: 10
    MinimumMembers      int // optional; default: 2
}
```

**最佳實踐：** 如果您對未導出的代碼有文件註解，請遵循與其被導出時相同的習慣（即，註解以未導出的名稱開頭）。這樣通過簡單替換註解和代碼中的未導出名稱與新導出的名稱，就容易將其後來導出。

<a id="comment-sentences"></a>

### 註解句子

<a id="TOC-CommentSentences"></a>

完整句子的註解應該像標準英語句子一樣使用大寫並加上標點符號。（作為一個例外，如果其他方面清楚，以未大寫的標識符名稱開始一個句子是可以接受的。這種情況最好只在段落的開頭進行。）

句子片段的註解對於標點符號或大寫沒有這樣的要求。

[文件註解] 應該總是完整句子，因此應該總是使用大寫和標點符號。簡單的行尾註解（特別是對於結構體字段）可以是假設字段名是主題的簡單短語。

```go
// 好的範例:
// A Server handles serving quotes from the collected works of Shakespeare.
type Server struct {
    // BaseDir points to the base directory under which Shakespeare's works are stored.
    //
    // The directory structure is expected to be the following:
    //   {BaseDir}/manifest.json
    //   {BaseDir}/{name}/{name}-part{number}.txt
    BaseDir string

    WelcomeMessage  string // displayed when user logs in
    ProtocolVersion string // checked against incoming requests
    PageLength      int    // lines per page when printing (optional; default: 20)
}
```

[文件註解]: #doc-comments

<a id="examples"></a>

### 範例

<a id="TOC-Examples"></a>

套件應該清楚地記錄其預期用法。嘗試提供一個[可運行的範例]；範例會出現在 Godoc 中。可運行的範例屬於測試文件，而不是生產源文件。參見此範例（[Godoc]，[源代碼]）。

[可運行的範例]: http://blog.golang.org/examples
[Godoc]: https://pkg.go.dev/time#example-Duration
[源代碼]: https://cs.opensource.google/go/go/+/HEAD:src/time/example_test.go

如果不可行提供一個可運行的範例，範例代碼可以在代碼註解中提供。與註解中的其他代碼和命令行片段一樣，它應該遵循標準格式化慣例。

<a id="named-result-parameters"></a>

### 命名結果參數

<a id="TOC-NamedResultParameters"></a>

在命名參數時，考慮函數簽名在 Godoc 中的顯示方式。函數本身的名稱和結果參數的類型通常已經足夠清楚。

```go
// 好的範例:
func (n *Node) Parent1() *Node
func (n *Node) Parent2() (*Node, error)
```

如果函數返回兩個或多個相同類型的參數，添加名稱可能會有用。

```go
// 好的範例:
func (n *Node) Children() (left, right *Node, err error)
```

如果調用者必須對特定結果參數採取行動，命名它們可以幫助建議什麼是行動：

```go
// 好的範例:
// WithTimeout returns a context that will be canceled no later than d duration
// from now.
//
// The caller must arrange for the returned cancel function to be called when
// the context is no longer needed to prevent a resource leak.
func WithTimeout(parent Context, d time.Duration) (ctx Context, cancel func())
```

在上面的代碼中，取消是調用者必須採取的特定行動。然而，如果結果參數僅寫為 `(Context, func())`，則不清楚所指的“取消函數”是什麼。

不要在名稱產生[不必要的重複](#repetitive-with-type)時使用命名結果參數。

```go
// 不好的範例:
func (n *Node) Parent1() (node *Node)
func (n *Node) Parent2() (node *Node, err error)
```

不要為了避免在函數內聲明一個變量而命名結果參數。這種做法導致不必要的 API 冗長，代價是輕微的實現簡潔。

[裸返回] 只在小函數中是可接受的。一旦它是一個中等大小的函數，請明確地返回您的值。同樣，不要僅因為它使您能夠使用裸返回就命名結果參數。[清晰度](guide#clarity)總是比在函數中節省幾行更重要。

如果必須在延遲閉包中更改結果參數的值，則命名結果參數始終是可接受的。

> **提示：** 在函數簽名中，類型往往比名稱更清晰。
> [GoTip #38: 函數作為命名類型] 展示了這一點。
>
> 在上面的 [`WithTimeout`] 中，真正的代碼使用了一個 [`CancelFunc`] 而不是原始的 `func()` 作為結果參數列表，並且很少需要文檔來說明。

[裸返回]: https://tour.golang.org/basics/7
[GoTip #38: 函數作為命名類型]: https://google.github.io/styleguide/go/index.html#gotip
[`WithTimeout`]: https://pkg.go.dev/context#WithTimeout
[`CancelFunc`]: https://pkg.go.dev/context#CancelFunc

<a id="package-comments"></a>

### 套件註解

<a id="TOC-PackageComments"></a>

套件註解必須出現在套件宣告的正上方，註解和套件名稱之間不能有空行。範例：

```go
// 好的範例:
// Package math 提供基本常數和數學函數。
//
// 本套件不保證在不同架構間有位元完全相同的結果。
package math
```

每個套件必須有一個套件註解。如果一個套件由多個文件組成，則正好有一個文件應該有一個套件註解。

`main` 套件的註解有一種略微不同的形式，其中 BUILD 文件中的 `go_binary` 規則的名稱取代了套件名稱。

```go
// 好的範例:
// seed_generator 命令是一個實用工具，它從一組 JSON 研究配置生成 Finch 種子文件。
package main
```

只要二進制名稱與 BUILD 文件中寫的完全一致，其他風格的註解也是可以的。當二進制名稱是第一個單詞時，即使它並不嚴格匹配命令行調用的拼寫，也需要將其大寫。

```go
// 好的範例:
// 二進制 seed_generator ...
// 命令 seed_generator ...
// 程序 seed_generator ...
// seed_generator 命令 ...
// seed_generator 程序 ...
// Seed_generator ...
```

Tips:

*   示例命令行調用和 API 使用可以是有用的文檔。對於 Godoc 格式，縮進包含代碼的註解行。

*   如果沒有明顯的主文件，或者如果套件註解非常長，則可以接受將文檔註解放在一個名為 `doc.go` 的文件中，該文件僅包含註解和套件子句。

*   多行註解可以代替多個單行註解。如果文檔包含可能需要從源文件中複製和粘貼的部分，這主要是有用的，如示例命令行（對於二進制文件）和模板示例。

    ```go
    // 好的範例:
    /*
    seed_generator 命令是一個實用工具，它從一組 JSON 研究配置生成 Finch 種子文件。

        seed_generator *.json | base64 > finch-seed.base64
    */
    package template
    ```

*   旨在供維護者使用且適用於整個文件的註解通常放在導入聲明之後。這些在 Godoc 中不會顯示，也不受上述套件註解規則的約束。

<a id="imports"></a>

## 引入

<a id="TOC-Imports"></a>

<a id="import-renaming"></a>

### 引入重命名

引入只應該在與其他引入發生名稱衝突時才重命名。（由此衍生的一個結論是，[良好的套件名稱](#package-names)不應該需要重命名。）在名稱衝突發生時，優先重命名最本地或項目特定的引入。對於套件的本地名稱（別名）必須遵循[套件命名的指導原則](#package-names)，包括禁止使用下劃線和大寫字母。

生成的協議緩衝區套件必須重命名以去除其名稱中的下劃線，並且它們的別名必須有一個 `pb` 後綴。有關更多信息，請參見[proto 和 stub 最佳實踐]。

[proto 和 stub 最佳實踐]: best-practices#import-protos

```go
// 好的範例:
import (
    fspb "path/to/package/foo_service_go_proto"
)
```

引入具有沒有有用識別信息的套件名稱（例如 `package v1`）時，應該重新命名以包含前一個路徑組件。重新命名必須與其他本地文件引入相同套件時保持一致，並且可以包含版本號。

**注意：** 建議將套件重新命名以符合[良好的套件名稱](#package-names)，但對於 vendored 目錄中的套件來說，這通常不可行。

```go
// 好的範例:
import (
    core "github.com/kubernetes/api/core/v1"
    meta "github.com/kubernetes/apimachinery/pkg/apis/meta/v1beta1"
)
```

如果您需要引入一個與您想要使用的常見本地變量名稱衝突的套件（例如 `url`、`ssh`），並且您希望重新命名該套件，那麼建議的做法是使用 `pkg` 後綴（例如 `urlpkg`）。請注意，有可能用本地變量遮蔽一個套件；只有在這樣的變量在作用域內時仍需要使用該套件，才需要進行此重命名。

<a id="import-grouping"></a>

### 引入分組

引入應該組織成兩組：

*   標準庫套件

*   其他（專案和 vendored）套件

```go
// 好的範例:
package main

import (
    "fmt"
    "hash/adler32"
    "os"

    "github.com/dsnet/compress/flate"
    "golang.org/x/text/encoding"
    "google.golang.org/protobuf/proto"
    foopb "myproj/foo/proto/proto"
    _ "myproj/rpc/protocols/dial"
    _ "myproj/security/auth/authhooks"
)
```

如果您想要一個單獨的組，則可以將專案套件分成多個組，只要這些組有一些意義即可。這樣做的常見原因包括：

*   重新命名的引入
*   為了它們的副作用而引入的套件

範例:

```go
// 好的範例:
package main

import (
    "fmt"
    "hash/adler32"
    "os"


    "github.com/dsnet/compress/flate"
    "golang.org/x/text/encoding"
    "google.golang.org/protobuf/proto"

    foopb "myproj/foo/proto/proto"

    _ "myproj/rpc/protocols/dial"
    _ "myproj/security/auth/authhooks"
)
```

**注意：** 維護可選組 - 分割超出標準庫和 Google 引入之間的強制性分離所必需的 - 不受 [goimports] 工具的支持。額外的引入子組需要作者和審查者的注意力來保持符合狀態。

[goimports]: golang.org/x/tools/cmd/goimports

同時也是 AppEngine 應用的 Google 程序應該有一個單獨的組用於 AppEngine 引入。

Gofmt 負責按引入路徑對每組進行排序。然而，它不會自動將引入分成組。流行的 [goimports] 工具結合了 Gofmt 和引入管理，根據上述決定將引入分成組。允許讓 [goimports] 完全管理引入排列，但是隨著文件的修訂，其引入列表必須保持內部一致性。

<a id="import-blank"></a>

### 引入「空白」（`import _`）

<a id="TOC-ImportBlank"></a>

只為了它們的副作用而被引入的套件（使用語法 `import _ "package"`）只能在主套件中引入，或在需要它們的測試中引入。

這類套件的一些例子包括：

*   [time/tzdata](https://pkg.go.dev/time/tzdata)

*   [image/jpeg](https://pkg.go.dev/image/jpeg) 在圖像處理代碼中

避免在庫套件中進行空白引入，即使庫間接依賴於它們。將副作用引入限制在主套件中有助於控制依賴關係，並使得撰寫依賴不同引入的測試成為可能，而不會產生衝突或浪費建構成本。

以下是此規則的唯一例外：

*   您可以使用空白引入來繞過 [nogo 靜態檢查器] 中對禁止引入的檢查。

*   您可以在使用 `//go:embed` 編譯器指令的源文件中，空白引入 [embed](https://pkg.go.dev/embed) 套件。

**提示：** 如果您創建了一個在生產中間接依賴副作用引入的庫套件，請記錄預期的使用方式。

[nogo 靜態檢查器]: https://github.com/bazelbuild/rules_go/blob/master/go/nogo.rst

<a id="import-dot"></a>

### 引入「點」（`import .`）

<a id="TOC-ImportDot"></a>

`import .` 形式是一種語言特性，它允許將另一個套件導出的標識符帶到當前套件中，無需資格限制。更多信息請參見 [語言規範](https://go.dev/ref/spec#Import_declarations)。

**不要** 在 Google 代碼庫中使用此功能；它使得更難判斷功能來自何處。

```go
// 不好的範例:
package foo_test

import (
    "bar/testutil" // also imports "foo"
    . "foo"
)

var myThing = Bar() // Bar defined in package foo; no qualification needed.
```

```go
// 好的範例:
package foo_test

import (
    "bar/testutil" // also imports "foo"
    "foo"
)

var myThing = foo.Bar()
```

<a id="errors"></a>

## 錯誤

<a id="returning-errors"></a>

### 返回錯誤

<a id="TOC-ReturningErrors"></a>

使用 `error` 來表示一個函數可能會失敗。按照慣例，`error` 是最後一個結果參數。

```go
// 好的範例:
func Good() error { /* ... */ }
```

返回一個 `nil` 錯誤是表示一個本可以失敗的操作成功的慣用方法。如果一個函數返回一個錯誤，除非另有明確文檔記載，否則調用者必須將所有非錯誤返回值視為未指定。通常，非錯誤返回值是它們的零值，但這不能假定。

```go
// 好的範例:
func GoodLookup() (*Result, error) {
    // ...
    if err != nil {
        return nil, err
    }
    return res, nil
}
```

導出的函數返回錯誤應該使用 `error` 類型返回它們。具體的錯誤類型容易出現微妙的錯誤：一個具體的 `nil` 指針可以被包裝進一個介面，從而變成一個非 `nil` 值（參見 [Go FAQ 中關於此主題的條目][nil error]）。

```go
// 不好的範例:
func Bad() *os.PathError { /*...*/ }
```

**提示**：一個接受 `context.Context` 參數的函數通常應該返回一個 `error`，以便調用者可以確定在函數運行時上下文是否被取消。

[nil error]: https://golang.org/doc/faq#nil_error

<a id="error-strings"></a>

### 錯誤字串

<a id="TOC-ErrorStrings"></a>

錯誤字串不應該首字大寫（除非以導出名稱、專有名詞或首字母縮略詞開頭），並且不應該以標點符號結尾。這是因為錯誤字串通常會在打印給用戶之前出現在其他上下文中。

```go
// 不好的範例:
err := fmt.Errorf("Something bad happened.")
```

```go
// 好的範例:
err := fmt.Errorf("something bad happened")
```

另一方面，完整顯示消息（日誌、測試失敗、API 回應或其他 UI）的風格取決於具體情況，但通常應該首字大寫。

```go
// 好的範例:
log.Infof("Operation aborted: %v", err)
log.Errorf("Operation aborted: %v", err)
t.Errorf("Op(%q) failed unexpectedly; err=%v", args, err)
```

<a id="handle-errors"></a>

### 處理錯誤

<a id="TOC-HandleErrors"></a>

遇到錯誤的代碼應該有意識地選擇如何處理它。通常不適合使用 `_` 變量來丟棄錯誤。如果函數返回一個錯誤，請執行以下操作之一：

*   立即處理並解決錯誤。
*   將錯誤返回給調用者。
*   在特殊情況下，調用 [`log.Fatal`] 或（如果絕對必要）`panic`。

**注意：** `log.Fatalf` 不是標準庫日誌。參見 [#logging]。

在極少數適合忽略或丟棄錯誤的情況下（例如，對 [`(*bytes.Buffer).Write`] 的調用，該調用被記錄為永遠不會失敗），應該有一個相應的註解解釋為什麼這樣做是安全的。

```go
// 好的範例:
var b *bytes.Buffer

n, _ := b.Write(p) // never returns a non-nil error
```

有關錯誤處理的更多討論和示例，請參見 [Effective Go](http://golang.org/doc/effective_go.html#errors) 和 [最佳實踐](best-practices.md#error-handling)。

[`(*bytes.Buffer).Write`]: https://pkg.go.dev/bytes#Buffer.Write

<a id="in-band-errors"></a>

### 帶內錯誤

<a id="TOC-In-Band-Errors"></a>

在 C 語言和類似語言中，函數返回像 -1、null 或空字串這樣的值來表示錯誤或缺失結果是很常見的。這稱為帶內錯誤處理。

```go
// 不好的範例:
// Lookup returns the value for key or -1 if there is no mapping for key.
func Lookup(key string) int
```

未檢查內部錯誤值可能導致錯誤，並將錯誤歸咎於錯誤的函數。

```go
// 不好的範例:
// The following line returns an error that Parse failed for the input value,
// whereas the failure was that there is no mapping for missingKey.
return Parse(Lookup(missingKey))
```

Go 對多返回值的支持提供了一個更好的解決方案（參見[Effective Go 關於多重返回的部分]）。函數不應要求客戶端檢查內部錯誤值，而應返回一個額外的值來指示其其他返回值是否有效。這個返回值可能是一個錯誤或一個布林值（當不需要解釋時），並且應該是最後的返回值。

```go
// 好的範例:
// Lookup returns the value for key or ok=false if there is no mapping for key.
func Lookup(key string) (value string, ok bool)
```

這種 API 防止調用者錯誤地寫 `Parse(Lookup(key))`，這會導致編譯時錯誤，因為 `Lookup(key)` 有 2 個輸出。

以這種方式返回錯誤鼓勵更健壯和明確的錯誤處理：

```go
// 好的範例:
value, ok := Lookup(key)
if !ok {
    return fmt.Errorf("no value for %q", key)
}
return Parse(value)
```

一些標準庫函數，如 `strings` 套件中的函數，返回內部錯誤值。這大大簡化了字符串操作代碼，但代價是需要程序員更加謹慎。一般來說，Google 代碼庫中的 Go 代碼應該返回額外的錯誤值。

[Effective Go 關於多重返回的部分]: http://golang.org/doc/effective_go.html#multiple-returns

<a id="indent-error-flow"></a>

### 縮進錯誤流程 (Indent error flow)

<a id="TOC-IndentErrorFlow"></a>

在繼續處理代碼的其餘部分之前，先處理錯誤。這通過使讀者能夠快速找到正常路徑來提高代碼的可讀性。這個邏輯同樣適用於任何測試條件然後以終止條件結束的塊（例如，`return`、`panic`、`log.Fatal`）。

如果未滿足終止條件，則運行的代碼應該出現在 `if` 塊之後，並且不應該在 `else` 子句中縮進。

```go
// 好的範例:
if err != nil {
    // 錯誤處理
    return // 或者 continue 等等
}
// 正常代碼
```

```go
// 不好的範例:
// 不好的範例:
if err != nil {
    // 錯誤處理
} else {
    // 由於縮進，看起來不正常的正常代碼
}
```

> **提示：** 如果您在多於幾行代碼中使用變量，通常不值得使用帶初始化器的 `if` 風格。在這些情況下，通常最好將聲明移出並使用標準的 `if` 語句：
>
> ```go
> // 好的範例:
> x, err := f()
> if err != nil {
>   // 錯誤處理
>   return
> }
> // 使用 x 的大量代碼
> // 跨越多行
> ```
>
> ```go
> // 不好的範例:
> if x, err := f(); err != nil {
>   // 錯誤處理
>   return
> } else {
>   // 使用 x 的大量代碼
>   // 跨越多行
> }
> ```

有關更多細節，請參見 [Go 提示 #1: 視線範圍] 和
[TotT: 通過減少嵌套降低代碼複雜性](https://testing.googleblog.com/2017/06/code-health-reduce-nesting-reduce.html)。

[Go 提示 #1: 視線範圍]: https://google.github.io/styleguide/go/index.html#gotip

<a id="language"></a>

## 語言

<a id="literal-formatting"></a>

### 字面量格式化

Go 擁有異常強大的[複合字面量語法]，可以用單一表達式來表達深層嵌套、複雜的值。在可能的情況下，應該使用這種字面量語法，而不是逐字段構建值。`gofmt` 對字面量的格式化通常相當不錯，但是還有一些額外的規則來保持這些字面量的可讀性和可維護性。

[複合字面量語法]: https://golang.org/ref/spec#Composite_literals

<a id="literal-field-names"></a>

#### 字段名稱

對於在當前套件外定義的類型，結構字面量必須指定**字段名稱**。

*   包括來自其他套件的類型的字段名稱。

    ```go
    // 好的範例:
    // https://pkg.go.dev/encoding/csv#Reader
    r := csv.Reader{
      Comma: ',',
      Comment: '#',
      FieldsPerRecord: 4,
    }
    ```

    結構中字段的位置和字段的完整集合（省略字段名稱時必須正確的兩個條件）通常不被認為是結構的公共 API 的一部分；指定字段名稱是為了避免不必要的耦合。

    ```go
    // 不好的範例:
    r := csv.Reader{',', '#', 4, false, false, false, false}
    ```

*   對於套件內部類型，字段名稱是可選的。

    ```go
    // 好的範例:
    okay := Type{42}
    also := internalType{4, 2}
    ```

    如果使用字段名稱可以使代碼更清晰，則仍應使用字段名稱，這樣做是非常常見的。例如，具有大量字段的結構幾乎總是應該使用字段名稱進行初始化。

    <!-- TODO: Maybe a better example here that doesn't have many fields. -->

    ```go
    // 好的範例:
    okay := StructWithLotsOfFields{
      field1: 1,
      field2: "two",
      field3: 3.14,
      field4: true,
    }
    ```

<a id="literal-matching-braces"></a>

#### 匹配的大括號

一對大括號的閉合部分應該總是出現在與開括號相同縮進量的行上。單行字面量必然具有此屬性。當字面量跨越多行時，保持此屬性使得字面量的大括號匹配與 Go 常見的語法結構（如函數和 `if` 語句）的大括號匹配相同。

這方面最常見的錯誤是將多行結構字面量中的值與閉合括號放在同一行。在這些情況下，該行應該以逗號結束，閉合括號應該出現在下一行。

```go
// 好的範例:
good := []*Type{{Key: "value"}}
```

```go
// 好的範例:
good := []*Type{
    {Key: "multi"},
    {Key: "line"},
}
```

```go
// 不好的範例:
bad := []*Type{
    {Key: "multi"},
    {Key: "line"}}
```

```go
// 不好的範例:
bad := []*Type{
    {
        Key: "value"},
}
```

<a id="literal-cuddled-braces"></a>

#### 貼合的大括號 (Cuddled braces)

對於切片和數組字面量，只有在以下兩個條件都滿足時，才允許去掉大括號之間的空白（也就是所謂的「緊靠」它們）。

*   [縮進匹配](#literal-matching-braces)
*   內部值也是字面量或 proto 構建器（即不是變量或其他表達式）

```go
// 好的範例:
good := []*Type{
    { // Not cuddled
        Field: "value",
    },
    {
        Field: "value",
    },
}
```

```go
// 好的範例:
good := []*Type{{ // Cuddled correctly
    Field: "value",
}, {
    Field: "value",
}}
```

```go
// 好的範例:
good := []*Type{
    first, // Can't be cuddled
    {Field: "second"},
}
```

```go
// 好的範例:
okay := []*pb.Type{pb.Type_builder{
    Field: "first", // Proto Builders may be cuddled to save vertical space
}.Build(), pb.Type_builder{
    Field: "second",
}.Build()}
```

```go
// 不好的範例:
bad := []*Type{
    first,
    {
        Field: "second",
    }}
```

<a id="literal-repeated-type-names"></a>

#### 重複的類型名稱

從切片和映射字面量中可以省略重複的類型名稱。這有助於減少混亂。當處理在您的項目中不常見的複雜類型時，明確重複類型名稱是一個合理的場合，特別是當重複的類型名稱出現在相隔很遠的行上，可以提醒讀者上下文。

```go
// 好的範例:
good := []*Type{
    {A: 42},
    {A: 43},
}
```

```go
// 不好的範例:
repetitive := []*Type{
    &Type{A: 42},
    &Type{A: 43},
}
```

```go
// 好的範例:
good := map[Type1]*Type2{
    {A: 1}: {B: 2},
    {A: 3}: {B: 4},
}
```

```go
// 不好的範例:
repetitive := map[Type1]*Type2{
    Type1{A: 1}: &Type2{B: 2},
    Type1{A: 3}: &Type2{B: 4},
}
```

**提示：** 如果您想在結構字面量中移除重複的類型名稱，您可以執行 `gofmt -s`。

<a id="literal-zero-value-fields"></a>

#### 零值字段

當不會因此失去清晰度時，可以從結構字面量中省略[零值]字段。

設計良好的 API 經常使用零值構造來增強可讀性。例如，從下面的結構中省略三個零值字段，可以將注意力集中到正在指定的唯一選項上。

[零值]: https://golang.org/ref/spec#The_zero_value

```go
// 不好的範例:
import (
  "github.com/golang/leveldb"
  "github.com/golang/leveldb/db"
)

ldb := leveldb.Open("/my/table", &db.Options{
    BlockSize: 1<<16,
    ErrorIfDBExists: true,

    // These fields all have their zero values.
    BlockRestartInterval: 0,
    Comparer: nil,
    Compression: nil,
    FileSystem: nil,
    FilterPolicy: nil,
    MaxOpenFiles: 0,
    WriteBufferSize: 0,
    VerifyChecksums: false,
})
```

```go
// 好的範例:
import (
  "github.com/golang/leveldb"
  "github.com/golang/leveldb/db"
)

ldb := leveldb.Open("/my/table", &db.Options{
    BlockSize: 1<<16,
    ErrorIfDBExists: true,
})
```

在表驅動測試中的結構體通常會從[明確的字段名稱]中受益，特別是當測試結構體不是微不足道的時候。這允許作者在有關字段與測試案例無關時完全省略零值字段。例如，成功的測試案例應該省略任何與錯誤相關或失敗相關的字段。在零值對於理解測試案例是必要的情況下，例如測試零或 `nil` 輸入，應該指定字段名稱。

[明確的字段名稱]: #literal-field-names

**簡潔**

```go
tests := []struct {
    input      string
    wantPieces []string
    wantErr    error
}{
    {
        input:      "1.2.3.4",
        wantPieces: []string{"1", "2", "3", "4"},
    },
    {
        input:   "hostname",
        wantErr: ErrBadHostname,
    },
}
```

**明確**

```go
tests := []struct {
    input    string
    wantIPv4 bool
    wantIPv6 bool
    wantErr  bool
}{
    {
        input:    "1.2.3.4",
        wantIPv4: true,
        wantIPv6: false,
    },
    {
        input:    "1:2::3:4",
        wantIPv4: false,
        wantIPv6: true,
    },
    {
        input:    "hostname",
        wantIPv4: false,
        wantIPv6: false,
        wantErr:  true,
    },
}
```

<a id="nil-slices"></a>

### Nil 切片

對於大多數目的來說，`nil` 和空切片之間沒有功能上的差異。內建函數如 `len` 和 `cap` 對 `nil` 切片的行為如預期。

```go
// 好的範例:
import "fmt"

var s []int         // nil

fmt.Println(s)      // []
fmt.Println(len(s)) // 0
fmt.Println(cap(s)) // 0
for range s {...}   // 無操作

s = append(s, 42)
fmt.Println(s)      // [42]
```

如果你聲明一個空切片作為局部變量（特別是如果它可以是返回值的來源），優先選擇 nil 初始化以減少呼叫者的錯誤風險。

```go
// 好的範例:
var t []string
```

```go
// 不好的範例:
t := []string{}
```

不要創建強迫客戶端區分 nil 和空切片的 API。

```go
// 好的範例:
// Ping 對其目標進行 ping 操作。
// 返回成功響應的主機。
func Ping(hosts []string) ([]string, error) { ... }
```

```go
// 不好的範例:
// Ping 對其目標進行 ping 操作並返回成功響應的主機列表。
// 如果輸入為空則可以為空。
// nil 表示發生了系統錯誤。
func Ping(hosts []string) []string { ... }
```

在設計介面時，避免區分 `nil` 切片和非 `nil`、長度為零的切片，因為這可能導致微妙的編程錯誤。這通常是通過使用 `len` 來檢查空值，而不是 `== nil` 來實現的。

這個實現接受 `nil` 和長度為零的切片作為「空」：

```go
// 好的範例:
// describeInts 用給定的前綴描述 s，除非 s 為空。
func describeInts(prefix string, s []int) {
    if len(s) == 0 {
        return
    }
    fmt.Println(prefix, s)
}
```

而不是將區分作為 API 的一部分依賴：

```go
// 不好的範例:
func maybeInts() []int { /* ... */ }

// describeInts 用給定的前綴描述 s；傳遞 nil 以完全跳過。
func describeInts(prefix string, s []int) {
  // 這個函數的行為不經意地根據 maybeInts() 在「空」情況下返回的內容（nil 或 []int{}）而改變。
  if s == nil {
    return
  }
  fmt.Println(prefix, s)
}

describeInts("這裡有一些整數：", maybeInts())
```

有關進一步討論，請參見[帶內錯誤]。

[帶內錯誤]: #in-band-errors

<a id="indentation-confusion"></a>

### 縮排混淆

避免引入換行符，如果這樣會使剩餘的行與縮進的代碼塊對齊。如果這是不可避免的，請留一個空格來分隔塊中的代碼與換行後的行。

```go
// 不好的範例:
if longCondition1 && longCondition2 &&
    // Conditions 3 and 4 have the same indentation as the code within the if.
    longCondition3 && longCondition4 {
    log.Info("all conditions met")
}
```

請參閱以下部分以獲得具體指南和示例：

*   [函數格式化](#func-formatting)
*   [條件和循環](#conditional-formatting)
*   [字面量格式化](#literal-formatting)

<a id="func-formatting"></a>

### 函數格式化

函數或方法聲明的簽名應保持在單行上，以避免[縮進混淆](#indentation-confusion)。

函數參數列表可能會使 Go 源文件中的某些行變得很長。然而，它們預示著縮進的變化，因此很難以不會使後續行看起來像是以令人困惑的方式屬於函數體的一部分的方式斷行：

```go
// 不好的範例:
func (r *SomeType) SomeLongFunctionName(foo1, foo2, foo3 string,
    foo4, foo5, foo6 int) {
    foo7 := bar(foo1)
    // ...
}
```

請參閱[最佳實踐](best-practices#funcargs)以獲得一些縮短具有許多參數的函數呼叫站點的選項。

通過提取局部變量，經常可以縮短行。

```go
// 好的範例:
local := helper(some, parameters, here)
good := foo.Call(list, of, parameters, local)
```

同樣，函數和方法呼叫不應僅基於行長度而被分開。

```go
// 好的範例:
good := foo.Call(long, list, of, parameters, all, on, one, line)
```

```go
// 不好的範例:
bad := foo.Call(long, list, of, parameters,
    with, arbitrary, line, breaks)
```

盡可能避免對特定函數參數添加內聯註釋。相反，使用[選項結構](best-practices#option-structure)或在函數文檔中添加更多細節。

```go
// 好的範例:
good := server.New(ctx, server.Options{Port: 42})
```

```go
// 不好的範例:
bad := server.New(
    ctx,
    42, // Port
)
```

如果 API 無法更改或者局部呼叫是不尋常的（無論呼叫是否太長），如果它有助於理解呼叫，始終允許添加斷行。

```go
// 好的範例:
canvas.RenderCube(cube,
    x0, y0, z0,
    x0, y0, z1,
    x0, y1, z0,
    x0, y1, z1,
    x1, y0, z0,
    x1, y0, z1,
    x1, y1, z0,
    x1, y1, z1,
)
```

請注意，上面示例中的行不是在特定的列邊界處換行，而是基於坐標三元組分組。

函數內的長字符串字面量不應該僅為了行長而被斷開。對於包含此類字符串的函數，可以在字符串格式之後添加換行符，並在下一行或後續行提供參數。關於斷行位置的決定應基於輸入的語義分組，而不僅僅是基於行長。

```go
// 好的範例:
log.Warningf("Database key (%q, %d, %q) incompatible in transaction started by (%q, %d, %q)",
    currentCustomer, currentOffset, currentKey,
    txCustomer, txOffset, txKey)
```

```go
// 不好的範例:
log.Warningf("Database key (%q, %d, %q) incompatible in"+
    " transaction started by (%q, %d, %q)",
    currentCustomer, currentOffset, currentKey, txCustomer,
    txOffset, txKey)
```

<a id="conditional-formatting"></a>

### 條件語句和循環

`if` 語句不應該換行；多行 `if` 子句可能導致[縮進混淆](#indentation-confusion)。

```go
// 不好的範例:
// The second if statement is aligned with the code within the if block, causing
// indentation confusion.
if db.CurrentStatusIs(db.InTransaction) &&
    db.ValuesEqual(db.TransactionKey(), row.Key()) {
    return db.Errorf(db.TransactionError, "query failed: row (%v): key does not match transaction key", row)
}
```

如果不需要短路行為，則可以直接提取布林運算元：

```go
// 好的範例:
inTransaction := db.CurrentStatusIs(db.InTransaction)
keysMatch := db.ValuesEqual(db.TransactionKey(), row.Key())
if inTransaction && keysMatch {
    return db.Error(db.TransactionError, "query failed: row (%v): key does not match transaction key", row)
}
```

也可能有其他局部變量可以被提取，特別是如果條件已經重複：

```go
// 好的範例:
uid := user.GetUniqueUserID()
if db.UserIsAdmin(uid) || db.UserHasPermission(uid, perms.ViewServerConfig) || db.UserHasPermission(uid, perms.CreateGroup) {
    // ...
}
```

```go
// 不好的範例:
if db.UserIsAdmin(user.GetUniqueUserID()) || db.UserHasPermission(user.GetUniqueUserID(), perms.ViewServerConfig) || db.UserHasPermission(user.GetUniqueUserID(), perms.CreateGroup) {
    // ...
}
```

包含閉包或多行結構字面量的 `if` 語句應確保[大括號匹配](#literal-matching-braces)以避免[縮進混淆](#indentation-confusion)。

```go
// 好的範例:
if err := db.RunInTransaction(func(tx *db.TX) error {
    return tx.Execute(userUpdate, x, y, z)
}); err != nil {
    return fmt.Errorf("user update failed: %s", err)
}
```

```go
// 好的範例:
if _, err := client.Update(ctx, &upb.UserUpdateRequest{
    ID:   userID,
    User: user,
}); err != nil {
    return fmt.Errorf("user update failed: %s", err)
}
```

同樣，不要嘗試在 `for` 語句中插入人為的換行。如果沒有優雅的重構方式，可以讓行保持長度：

```go
// 好的範例:
for i, max := 0, collection.Size(); i < max && !collection.HasPendingWriters(); i++ {
    // ...
}
```

然而，通常有辦法：

```go
// 好的範例:
for i, max := 0, collection.Size(); i < max; i++ {
    if collection.HasPendingWriters() {
        break
    }
    // ...
}
```

`switch` 和 `case` 語句也應保持在單行上。

```go
// 好的範例:
switch good := db.TransactionStatus(); good {
case db.TransactionStarting, db.TransactionActive, db.TransactionWaiting:
    // ...
case db.TransactionCommitted, db.NoTransaction:
    // ...
default:
    // ...
}
```

```go
// 不好的範例:
switch bad := db.TransactionStatus(); bad {
case db.TransactionStarting,
    db.TransactionActive,
    db.TransactionWaiting:
    // ...
case db.TransactionCommitted,
    db.NoTransaction:
    // ...
default:
    // ...
}
```

如果行過長，將所有案例縮進並用空行分隔，以避免[縮進混淆](#indentation-confusion)：

```go
// 好的範例:
switch db.TransactionStatus() {
case
    db.TransactionStarting,
    db.TransactionActive,
    db.TransactionWaiting,
    db.TransactionCommitted:

    // ...
case db.NoTransaction:
    // ...
default:
    // ...
}
```

在將變量與常量進行比較的條件語句中，將變量值放在等號運算符的左側：

```go
// 好的範例:
if result == "foo" {
  // ...
}
```

而不是不太清晰的語句，其中常量首先出現（["Yoda 風格條件"](https://en.wikipedia.org/wiki/Yoda_conditions)）:

```go
// 不好的範例:
if "foo" == result {
  // ...
}
```

<a id="copying"></a>

### 複製 Copying

<a id="TOC-Copying"></a>

為了避免意外的別名和類似的錯誤，在從另一個包複製結構體時要小心。例如，同步對象如 `sync.Mutex` 不能被複製。

`bytes.Buffer` 類型包含一個 `[]byte` 切片，並且作為對小字符串的優化，它包含一個小的字節數組，切片可能會引用該數組。如果你複製了一個 `Buffer`，副本中的切片可能會與原始對象中的數組產生別名，導致後續的方法調用有意外的效果。

一般來說，如果一個類型 `T` 的方法與指針類型 `*T` 相關聯，則不要複製類型 `T` 的值。

```go
// 不好的範例:
b1 := bytes.Buffer{}
b2 := b1
```

調用一個接收值的方法可以隱藏複製行為。當你設計一個 API 時，如果你的結構體包含不應該被複製的字段，你應該通常接收並返回指針類型。

這些是可以接受的：

```go
// 好的範例:
type Record struct {
  buf bytes.Buffer
  // other fields omitted
}

func New() *Record {...}

func (r *Record) Process(...) {...}

func Consumer(r *Record) {...}
```

但這些通常是錯誤的：

```go
// 不好的範例:
type Record struct {
  buf bytes.Buffer
  // other fields omitted
}


func (r Record) Process(...) {...} // Makes a copy of r.buf

func Consumer(r Record) {...} // Makes a copy of r.buf
```

這個指導原則也適用於複製 `sync.Mutex`。

<a id="dont-panic"></a>

### 不要恐慌 Don't panic

<a id="TOC-Don-t-Panic"></a>

不要使用 `panic` 進行正常的錯誤處理。相反，使用 `error` 和多個返回值。參見 [Effective Go 中關於錯誤的部分]。

在 `package main` 和初始化代碼中，考慮使用 [`log.Exit`] 處理應該終止程序的錯誤（例如，無效配置），因為在許多這樣的情況下，堆棧跟踪不會幫助讀者。請注意，[`log.Exit`] 調用 [`os.Exit`]，任何延遲的函數都不會被執行。

對於表示「不可能」條件的錯誤，即應該在代碼審查和/或測試期間始終被捕獲的錯誤，函數可以合理地返回一個錯誤或調用 [`log.Fatal`]。

**注意：** `log.Fatalf` 不是標準庫日誌。參見 [#logging]。

[Effective Go 中關於錯誤的部分]: http://golang.org/doc/effective_go.html#errors
[`os.Exit`]: https://pkg.go.dev/os#Exit

<a id="must-functions"></a>

### 必須函數 Must functions

在失敗時停止程序的設置輔助函數遵循命名慣例 `MustXYZ`（或 `mustXYZ`）。一般來說，它們應該只在程序啟動初期被調用，而不是像用戶輸入這樣的情況，其中優先使用正常的 Go 錯誤處理。

這通常出現在僅在[包初始化時](https://golang.org/ref/spec#Package_initialization)調用的函數，用於初始化包級變量（例如 [template.Must](https://golang.org/pkg/text/template/#Must) 和 [regexp.MustCompile](https://golang.org/pkg/regexp/#MustCompile)）。

```go
// 好的範例:
func MustParse(version string) *Version {
    v, err := Parse(version)
    if err != nil {
        panic(fmt.Sprintf("MustParse(%q) = _, %v", version, err))
    }
    return v
}

// Package level "constant". If we wanted to use `Parse`, we would have had to
// set the value in `init`.
var DefaultVersion = MustParse("1.2.3")
```

相同的慣例也可以用在測試輔助函數中，這些函數只停止當前測試（使用 `t.Fatal`）。這樣的輔助函數在創建測試值時經常很方便，例如在[表驅動測試](#table-driven-tests)的結構字段中，因為返回錯誤的函數不能直接分配給結構字段。

```go
// 好的範例:
func mustMarshalAny(t *testing.T, m proto.Message) *anypb.Any {
  t.Helper()
  any, err := anypb.New(m)
  if err != nil {
    t.Fatalf("mustMarshalAny(t, m) = %v; want %v", err, nil)
  }
  return any
}

func TestCreateObject(t *testing.T) {
  tests := []struct{
    desc string
    data *anypb.Any
  }{
    {
      desc: "my test case",
      // Creating values directly within table driven test cases.
      data: mustMarshalAny(t, mypb.Object{}),
    },
    // ...
  }
  // ...
}
```

在這兩種情況下，這種模式的價值在於輔助函數可以在「值」上下文中被調用。這些輔助函數不應該在難以確保錯誤會被捕獲的地方調用，或者在應該[檢查](#handle-errors)錯誤的上下文中調用（例如，在許多請求處理程序中）。對於常量輸入，這允許測試輕鬆確保 `Must` 參數格式正確，對於非常量輸入，它允許測試驗證錯誤是否[被適當處理或傳播](best-practices#error-handling)。

在測試中使用 `Must` 函數時，它們通常應該[標記為測試輔助函數](#mark-test-helpers)，並在出錯時調用 `t.Fatal`（參見[測試輔助函數中的錯誤處理](best-practices#test-helper-error-handling)以獲得更多使用該函數的考慮）。

當[普通的錯誤處理](best-practices#error-handling)是可能的時候（包括一些重構），它們不應該被使用：

```go
// 不好的範例:
func Version(o *servicepb.Object) (*version.Version, error) {
    // Return error instead of using Must functions.
    v := version.MustParse(o.GetVersionString())
    return dealiasVersion(v)
}
```

<a id="goroutine-lifetimes"></a>

### Goroutine 生命週期

<a id="TOC-GoroutineLifetimes"></a>

當你啟動 goroutines 時，要清楚它們何時或是否會退出。

Goroutines 可以通過阻塞在通道發送或接收上而泄漏。即使沒有其他 goroutine 引用該通道，垃圾收集器也不會終止阻塞在通道上的 goroutine。

即使 goroutines 沒有泄漏，當它們不再需要時仍然讓它們在執行中，也可能導致其他微妙且難以診斷的問題。在已經關閉的通道上發送會導致恐慌。

```go
// 不好的範例:
ch := make(chan int)
ch <- 42
close(ch)
ch <- 13 // panic
```

在「結果不再需要後」修改仍在使用中的輸入可能導致數據競爭。任意長時間地保留 goroutines 在執行中可能導致不可預測的內存使用。

並發代碼應該寫得使 goroutine 的生命週期明顯。通常這將意味著將與同步相關的代碼限制在函數的範圍內，並將邏輯分解為[同步函數]。如果並發性仍然不明顯，記錄 goroutines 何時以及為什麼退出是很重要的。

遵循最佳實踐的上下文使用相關代碼通常有助於澄清這一點。它通常是用 `context.Context` 管理的：

```go
// 好的範例:
func (w *Worker) Run(ctx context.Context) error {
    // ...
    for item := range w.q {
        // process returns at latest when the context is cancelled.
        go process(ctx, item)
    }
    // ...
}
```

還有其他使用原始信號通道如 `chan struct{}`、同步變量、[條件變量][rethinking-slides] 等的變體。重要的是後續維護者能夠明顯地看到 goroutine 的結束。

相比之下，以下代碼對其啟動的 goroutines 何時結束不夠謹慎：

```go
// 不好的範例:
func (w *Worker) Run() {
    // ...
    for item := range w.q {
        // process returns when it finishes, if ever, possibly not cleanly
        // handling a state transition or termination of the Go program itself.
        go process(item)
    }
    // ...
}
```

這段代碼看起來可能沒問題，但存在幾個潛在問題：

*   代碼在生產中可能有未定義的行為，即使操作系統釋放了資源，程序也可能無法乾淨地終止。
*   由於代碼的不確定生命週期，測試代碼意義不大。
*   如上所述，代碼可能泄漏資源。

另見：

*   [永遠不要啟動一個 goroutine 而不知道它將如何停止][cheney-stop]
*   重新思考傳統並發模式：[幻燈片][rethinking-slides]，[視頻][rethinking-video]
*   [Go 程序何時結束]

[同步函數]: #synchronous-functions
[cheney-stop]: https://dave.cheney.net/2016/12/22/never-start-a-goroutine-without-knowing-how-it-will-stop
[rethinking-slides]: https://drive.google.com/file/d/1nPdvhB0PutEJzdCq5ms6UI58dp50fcAN/view
[rethinking-video]: https://www.youtube.com/watch?v=5zXAHh5tJqQ
[Go 程序何時結束]: https://changelog.com/gotime/165

<a id="interfaces"></a>

### 介面 Interfaces

<a id="TOC-Interfaces"></a>

Go 介面通常屬於*使用*介面類型值的包，而不是*實現*介面類型的包。實現包應該返回具體的（通常是指針或結構體）類型。這樣，新方法可以添加到實現中，而不需要進行大規模重構。有關更多細節，請參見 [GoTip #49: 接受介面，返回具體類型]。

不要從使用它的 API 中導出介面的[測試雙重][double types]實現。相反，設計 API 以便可以使用[真實實現]的[公共 API]進行測試。有關更多細節，請參見 [GoTip #42: 編寫用於測試的 Stub]。即使無法使用真實實現，也可能不需要引入完全涵蓋真實類型中所有方法的介面；消費者可以創建一個只包含其需要的方法的介面，如 [GoTip #78: 最小可行介面] 中所示。

要測試使用 Stubby RPC 客戶端的包，使用真實的客戶端連接。如果在測試中無法運行真實服務器，Google 內部的做法是使用內部的 rpctest 包（即將推出！）獲取到本地[測試雙重]的真實客戶端連接。

在它們被使用之前不要定義介面（參見 [TotT: 代碼健康：消除 YAGNI 味道][tott-438]）。沒有一個現實的使用例子，很難看出介面是否甚至是必要的，更不用說它應該包含哪些方法了。

如果包的用戶不需要傳遞不同類型的參數，則不要使用介面類型的參數。

不要導出包的用戶不需要的介面。

**待辦事項：** 編寫一份關於介面的更深入的文檔，並在這裡鏈接它。

[GoTip #42: 編寫用於測試的 Stub]: https://google.github.io/styleguide/go/index.html#gotip
[GoTip #49: 接受介面，返回具體類型]: https://google.github.io/styleguide/go/index.html#gotip
[GoTip #78: 最小可行介面]: https://google.github.io/styleguide/go/index.html#gotip
[真實實現]: best-practices#use-real-transports
[公共 API]: https://abseil.io/resources/swe-book/html/ch12.html#test_via_public_apis
[double types]: https://abseil.io/resources/swe-book/html/ch13.html#techniques_for_using_test_doubles
[測試雙重]: https://abseil.io/resources/swe-book/html/ch13.html#basic_concepts
[tott-438]: https://testing.googleblog.com/2017/08/code-health-eliminate-yagni-smells.html

```go
// 好的範例:
package consumer // consumer.go

type Thinger interface { Thing() bool }

func Foo(t Thinger) string { ... }
```

```go
// 好的範例:
package consumer // consumer_test.go

type fakeThinger struct{ ... }
func (t fakeThinger) Thing() bool { ... }
...
if Foo(fakeThinger{...}) == "x" { ... }
```

```go
// 不好的範例:
package producer

type Thinger interface { Thing() bool }

type defaultThinger struct{ ... }
func (t defaultThinger) Thing() bool { ... }

func NewThinger() Thinger { return defaultThinger{ ... } }
```

```go
// 好的範例:
package producer

type Thinger struct{ ... }
func (t Thinger) Thing() bool { ... }

func NewThinger() Thinger { return Thinger{ ... } }
```

<a id="generics"></a>

### 泛型 Generics

泛型（正式稱為"[類型參數]")在滿足業務需求的地方是允許的。在許多應用中，使用現有語言特性（切片、映射、介面等）的傳統方法同樣可以工作，而不會增加額外的複雜性，所以要謹慎使用以免過早。參見關於[最小機制](guide#least-mechanism)的討論。

當引入使用泛型的導出 API 時，請確保它有適當的文檔。強烈建議包括動機性的可運行[示例]。

不要僅僅因為你正在實現一個不關心其成員元素類型的算法或數據結構就使用泛型。如果實際上只有一種類型被實例化，首先開始讓你的代碼在那種類型上工作，而不使用泛型。與移除被發現是不必要的抽象相比，後來添加多態性將是直截了當的。

不要使用泛型來發明特定領域的語言（DSL）。特別是，避免引入可能對讀者造成重大負擔的錯誤處理框架。相反，優先使用既定的[錯誤處理](#errors)實踐。對於測試，要特別小心引入導致不太有用的[測試失敗](#useful-test-failures)的[斷言庫](#assert)或框架。

一般來說：

*   [寫代碼，不要設計類型]。來自 Robert Griesemer 和 Ian Lance Taylor 的 GopherCon 演講。
*   如果你有幾種類型共享一個有用的統一介面，考慮使用該介面來建模解決方案。可能不需要泛型。
*   否則，不要依賴 `any` 類型和過度的[類型切換](https://tour.golang.org/methods/16)，考慮使用泛型。

另見：

*   [在 Go 中使用泛型]，Ian Lance Taylor 的演講

*   Go 網頁上的[泛型教程]

[泛型教程]: https://go.dev/doc/tutorial/generics
[類型參數]: https://go.dev/design/43651-type-parameters
[在 Go 中使用泛型]: https://www.youtube.com/watch?v=nr8EpUO9jhw
[寫代碼，不要設計類型]: https://www.youtube.com/watch?v=Pa_e9EeCdy8&t=1250s

<a id="pass-values"></a>

### Pass values

<a id="TOC-PassValues"></a>

Do not pass pointers as function arguments just to save a few bytes. If a
function reads its argument `x` only as `*x` throughout, then the argument
shouldn't be a pointer. Common instances of this include passing a pointer to a
string (`*string`) or a pointer to an interface value (`*io.Reader`). In both
cases, the value itself is a fixed size and can be passed directly.

This advice does not apply to large structs, or even small structs that may
increase in size. In particular, protocol buffer messages should generally be
handled by pointer rather than by value. The pointer type satisfies the
`proto.Message` interface (accepted by `proto.Marshal`, `protocmp.Transform`,
etc.), and protocol buffer messages can be quite large and often grow larger
over time.

<a id="receiver-type"></a>

### Receiver type

<a id="TOC-ReceiverType"></a>

A [method receiver] can be passed either as a value or a pointer, just as if it
were a regular function parameter. The choice between the two is based on which
[method set(s)] the method should be a part of.

[method receiver]: https://golang.org/ref/spec#Method_declarations
[method set(s)]: https://golang.org/ref/spec#Method_sets

**Correctness wins over speed or simplicity.** There are cases where you must
use a pointer value. In other cases, pick pointers for large types or as
future-proofing if you don't have a good sense of how the code will grow, and
use values for simple [plain old data].

The list below spells out each case in further detail:

*   If the receiver is a slice and the method doesn't reslice or reallocate the
    slice, use a value rather than a pointer.

    ```go
    // 好的範例:
    type Buffer []byte

    func (b Buffer) Len() int { return len(b) }
    ```

*   If the method needs to mutate the receiver, the receiver must be a pointer.

    ```go
    // 好的範例:
    type Counter int

    func (c *Counter) Inc() { *c++ }

    // See https://pkg.go.dev/container/heap.
    type Queue []Item

    func (q *Queue) Push(x Item) { *q = append([]Item{x}, *q...) }
    ```

*   If the receiver is a struct containing fields that
    [cannot safely be copied](#copying), use a pointer receiver. Common examples
    are [`sync.Mutex`] and other synchronization types.

    ```go
    // 好的範例:
    type Counter struct {
        mu    sync.Mutex
        total int
    }

    func (c *Counter) Inc() {
        c.mu.Lock()
        defer c.mu.Unlock()
        c.total++
    }
    ```

    **Tip:** Check the type's [Godoc] for information about whether it is safe
    or unsafe to copy.

*   If the receiver is a "large" struct or array, a pointer receiver may be more
    efficient. Passing a struct is equivalent to passing all of its fields or
    elements as arguments to the method. If that seems too large to
    [pass by value](#pass-values), a pointer is a good choice.

*   For methods that will call or run concurrently with other functions that
    modify the receiver, use a value if those modifications should not be
    visible to your method; otherwise use a pointer.

*   If the receiver is a struct or array, any of whose elements is a pointer to
    something that may be mutated, prefer a pointer receiver to make the
    intention of mutability clear to the reader.

    ```go
    // 好的範例:
    type Counter struct {
        m *Metric
    }

    func (c *Counter) Inc() {
        c.m.Add(1)
    }
    ```

*   If the receiver is a [built-in type], such as an integer or a string, that
    does not need to be modified, use a value.

    ```go
    // 好的範例:
    type User string

    func (u User) String() { return string(u) }
    ```

*   If the receiver is a map, function, or channel, use a value rather than a
    pointer.

    ```go
    // 好的範例:
    // See https://pkg.go.dev/net/http#Header.
    type Header map[string][]string

    func (h Header) Add(key, value string) { /* omitted */ }
    ```

*   If the receiver is a "small" array or struct that is naturally a value type
    with no mutable fields and no pointers, a value receiver is usually the
    right choice.

    ```go
    // 好的範例:
    // See https://pkg.go.dev/time#Time.
    type Time struct { /* omitted */ }

    func (t Time) Add(d Duration) Time { /* omitted */ }
    ```

*   When in doubt, use a pointer receiver.

As a general guideline, prefer to make the methods for a type either all pointer
methods or all value methods.

**Note:** There is a lot of misinformation about whether passing a value or a
pointer to a function can affect performance. The compiler can choose to pass
pointers to values on the stack as well as copying values on the stack, but
these considerations should not outweigh the readability and correctness of the
code in most circumstances. When the performance does matter, it is important to
profile both approaches with a realistic benchmark before deciding that one
approach outperforms the other.

[plain old data]: https://en.wikipedia.org/wiki/Passive_data_structure
[`sync.Mutex`]: https://pkg.go.dev/sync#Mutex
[built-in type]: https://pkg.go.dev/builtin

<a id="switch-break"></a>

### `switch` and `break`

<a id="TOC-SwitchBreak"></a>

Do not use `break` statements without target labels at the ends of `switch`
clauses; they are redundant. Unlike in C and Java, `switch` clauses in Go
automatically break, and a `fallthrough` statement is needed to achieve the
C-style behavior. Use a comment rather than `break` if you want to clarify the
purpose of an empty clause.

```go
// 好的範例:
switch x {
case "A", "B":
    buf.WriteString(x)
case "C":
    // handled outside of the switch statement
default:
    return fmt.Errorf("unknown value: %q", x)
}
```

```go
// 不好的範例:
switch x {
case "A", "B":
    buf.WriteString(x)
    break // this break is redundant
case "C":
    break // this break is redundant
default:
    return fmt.Errorf("unknown value: %q", x)
}
```

> **Note:** If a `switch` clause is within a `for` loop, using `break` within
> `switch` does not exit the enclosing `for` loop.
>
> ```go
> for {
>   switch x {
>   case "A":
>      break // exits the switch, not the loop
>   }
> }
> ```
>
> To escape the enclosing loop, use a label on the `for` statement:
>
> ```go
> loop:
>   for {
>     switch x {
>     case "A":
>        break loop // exits the loop
>     }
>   }
> ```

<a id="synchronous-functions"></a>

### Synchronous functions

<a id="TOC-SynchronousFunctions"></a>

Synchronous functions return their results directly and finish any callbacks or
channel operations before returning. Prefer synchronous functions over
asynchronous functions.

Synchronous functions keep goroutines localized within a call. This helps to
reason about their lifetimes, and avoid leaks and data races. Synchronous
functions are also easier to test, since the caller can pass an input and check
the output without the need for polling or synchronization.

If necessary, the caller can add concurrency by calling the function in a
separate goroutine. However, it is quite difficult (sometimes impossible) to
remove unnecessary concurrency at the caller side.

See also:

*   "Rethinking Classical Concurrency Patterns", talk by Bryan Mills:
    [slides][rethinking-slides], [video][rethinking-video]

<a id="type-aliases"></a>

### Type aliases

<a id="TOC-TypeAliases"></a>

Use a *type definition*, `type T1 T2`, to define a new type. Use a
[*type alias*], `type T1 = T2`, to refer to an existing type without defining a
new type. Type aliases are rare; their primary use is to aid migrating packages
to new source code locations. Don't use type aliasing when it is not needed.

[*type alias*]: http://golang.org/ref/spec#Type_declarations

<a id="use-percent-q"></a>

### Use %q

<a id="TOC-UsePercentQ"></a>

Go's format functions (`fmt.Printf` etc.) have a `%q` verb which prints strings
inside double-quotation marks.

```go
// 好的範例:
fmt.Printf("value %q looks like English text", someText)
```

Prefer using `%q` over doing the equivalent manually, using `%s`:

```go
// 不好的範例:
fmt.Printf("value \"%s\" looks like English text", someText)
// Avoid manually wrapping strings with single-quotes too:
fmt.Printf("value '%s' looks like English text", someText)
```

Using `%q` is recommended in output intended for humans where the input value
could possibly be empty or contain control characters. It can be very hard to
notice a silent empty string, but `""` stands out clearly as such.

<a id="use-any"></a>

### Use any

Go 1.18 introduces an `any` type as an [alias] to `interface{}`. Because it is
an alias, `any` is equivalent to `interface{}` in many situations and in others
it is easily interchangeable via an explicit conversion. Prefer to use `any` in
new code.

[alias]: https://go.googlesource.com/proposal/+/master/design/18130-type-alias.md

## Common libraries

<a id="flags"></a>

### Flags

<a id="TOC-Flags"></a>

Go programs in the Google codebase use an internal variant of the
[standard `flag` package]. It has a similar interface but interoperates well
with internal Google systems. Flag names in Go binaries should prefer to use
underscores to separate words, though the variables that hold a flag's value
should follow the standard Go name style ([mixed caps]). Specifically, the flag
name should be in snake case, and the variable name should be the equivalent
name in camel case.

```go
// 好的範例:
var (
    pollInterval = flag.Duration("poll_interval", time.Minute, "Interval to use for polling.")
)
```

```go
// 不好的範例:
var (
    poll_interval = flag.Int("pollIntervalSeconds", 60, "Interval to use for polling in seconds.")
)
```

Flags must only be defined in `package main` or equivalent.

General-purpose packages should be configured using Go APIs, not by punching
through to the command-line interface; don't let importing a library export new
flags as a side effect. That is, prefer explicit function arguments or struct
field assignment or much less frequently and under the strictest of scrutiny
exported global variables. In the extremely rare case that it is necessary to
break this rule, the flag name must clearly indicate the package that it
configures.

If your flags are global variables, place them in their own `var` group,
following the imports section.

There is additional discussion around best practices for creating [complex CLIs]
with subcommands.

See also:

*   [Tip of the Week #45: Avoid Flags, Especially in Library Code][totw-45]
*   [Go Tip #10: Configuration Structs and Flags](https://google.github.io/styleguide/go/index.html#gotip)
*   [Go Tip #80: Dependency Injection Principles](https://google.github.io/styleguide/go/index.html#gotip)

[standard `flag` package]: https://golang.org/pkg/flag/
[mixed caps]: guide#mixed-caps
[complex CLIs]: best-practices#complex-clis
[totw-45]: https://abseil.io/tips/45

<a id="logging"></a>

### Logging

Go programs in the Google codebase use a variant of the standard [`log`]
package. It has a similar but more powerful interface and interoperates well
with internal Google systems. An open source version of this library is
available as [package `glog`], and open source Google projects may use that, but
this guide refers to it as `log` throughout.

**Note:** For abnormal program exits, this library uses `log.Fatal` to abort
with a stacktrace, and `log.Exit` to stop without one. There is no `log.Panic`
function as in the standard library.

**Tip:** `log.Info(v)` is equivalent `log.Infof("%v", v)`, and the same goes for
other logging levels. Prefer the non-formatting version when you have no
formatting to do.

See also:

*   Best practices on [logging errors](best-practices#error-logging) and
    [custom verbosily levels](best-practices#vlog)
*   When and how to use the log package to
    [stop the program](best-practices#checks-and-panics)

[`log`]: https://pkg.go.dev/log
[`log/slog`]: https://pkg.go.dev/log/slog
[package `glog`]: https://pkg.go.dev/github.com/golang/glog
[`log.Exit`]: https://pkg.go.dev/github.com/golang/glog#Exit
[`log.Fatal`]: https://pkg.go.dev/github.com/golang/glog#Fatal

<a id="contexts"></a>

### Contexts

<a id="TOC-Contexts"></a>

Values of the [`context.Context`] type carry security credentials, tracing
information, deadlines, and cancellation signals across API and process
boundaries. Unlike C++ and Java, which in the Google codebase use thread-local
storage, Go programs pass contexts explicitly along the entire function call
chain from incoming RPCs and HTTP requests to outgoing requests.

[`context.Context`]: https://pkg.go.dev/context

When passed to a function or method, `context.Context` is always the first
parameter.

```go
func F(ctx context.Context /* other arguments */) {}
```

Exceptions are:

*   In an HTTP handler, where the context comes from
    [`req.Context()`](https://pkg.go.dev/net/http#Request.Context).
*   In streaming RPC methods, where the context comes from the stream.

    Code using gRPC streaming accesses a context from a `Context()` method in
    the generated server type, which implements `grpc.ServerStream`. See
    [gRPC Generated Code documentation](https://grpc.io/docs/languages/go/generated-code/).

*   In entrypoint functions (see below for examples of such functions), use
    [`context.Background()`](https://pkg.go.dev/context/#Background).

    *   In binary targets: `main`
    *   In general purpose code and libraries: `init`
    *   In tests: `TestXXX`, `BenchmarkXXX`, `FuzzXXX`

> **Note**: It is very rare for code in the middle of a callchain to require
> creating a base context of its own using `context.Background()`. Always prefer
> taking a context from your caller, unless it's the wrong context.
>
> You may come across server libraries (the implementation of Stubby, gRPC, or
> HTTP in Google's server framework for Go) that construct a fresh context
> object per request. These contexts are immediately filled with information
> from the incoming request, so that when passed to the request handler, the
> context's attached values have been propagated to it across the network
> boundary from the client caller. Moreover, these contexts' lifetimes are
> scoped to that of the request: when the request is finished, the context is
> cancelled.
>
> Unless you are implementing a server framework, you shouldn't create contexts
> with `context.Background()` in library code. Instead, prefer using context
> detachment, which is mentioned below, if there is an existing context
> available. If you think you do need `context.Background()` outside of
> entrypoint functions, discuss it with the Google Go style mailing list before
> committing to an implementation.

The convention that `context.Context` comes first in functions also applies to
test helpers.

```go
// 好的範例:
func readTestFile(ctx context.Context, t *testing.T, path string) string {}
```

Do not add a context member to a struct type. Instead, add a context parameter
to each method on the type that needs to pass it along. The one exception is for
methods whose signature must match an interface in the standard library or in a
third party library outside Google's control. Such cases are very rare, and
should be discussed with the Google Go style mailing list before implementation
and readability review.

Code in the Google codebase that must spawn background operations which can run
after the parent context has been cancelled can use an internal package for
detachment. Follow [issue #40221](https://github.com/golang/go/issues/40221) for
discussions on an open source alternative.

Since contexts are immutable, it is fine to pass the same context to multiple
calls that share the same deadline, cancellation signal, credentials, parent
trace, and so on.

See also:

*   [Contexts and structs]

[Contexts and structs]: https://go.dev/blog/context-and-structs

<a id="custom-contexts"></a>

#### Custom contexts

Do not create custom context types or use interfaces other than
`context.Context` in function signatures. There are no exceptions to this rule.

Imagine if every team had a custom context. Every function call from package `p`
to package `q` would have to determine how to convert a `p.Context` to a
`q.Context`, for all pairs of packages `p` and `q`. This is impractical and
error-prone for humans, and it makes automated refactorings that add context
parameters nearly impossible.

If you have application data to pass around, put it in a parameter, in the
receiver, in globals, or in a `Context` value if it truly belongs there.
Creating your own context type is not acceptable since it undermines the ability
of the Go team to make Go programs work properly in production.

<a id="crypto-rand"></a>

### crypto/rand

<a id="TOC-CryptoRand"></a>

Do not use package `math/rand` to generate keys, even throwaway ones. If
unseeded, the generator is completely predictable. Seeded with
`time.Nanoseconds()`, there are just a few bits of entropy. Instead, use
`crypto/rand`'s Reader, and if you need text, print to hexadecimal or base64.

```go
// 好的範例:
import (
    "crypto/rand"
    // "encoding/base64"
    // "encoding/hex"
    "fmt"

    // ...
)

func Key() string {
    buf := make([]byte, 16)
    if _, err := rand.Read(buf); err != nil {
        log.Fatalf("Out of randomness, should never happen: %v", err)
    }
    return fmt.Sprintf("%x", buf)
    // or hex.EncodeToString(buf)
    // or base64.StdEncoding.EncodeToString(buf)
}
```

**Note:** `log.Fatalf` is not the standard library log. See [#logging].

<a id="useful-test-failures"></a>

## Useful test failures

<a id="TOC-UsefulTestFailures"></a>

It should be possible to diagnose a test's failure without reading the test's
source. Tests should fail with helpful messages detailing:

*   What caused the failure
*   What inputs resulted in an error
*   The actual result
*   What was expected

Specific conventions for achieving this goal are outlined below.

<a id="assert"></a>

### Assertion libraries

<a id="TOC-Assert"></a>

Do not create "assertion libraries" as helpers for testing.

Assertion libraries are libraries that attempt to combine the validation and
production of failure messages within a test (though the same pitfalls can apply
to other test helpers as well). For more on the distinction between test helpers
and assertion libraries, see [best practices](best-practices#test-functions).

```go
// 不好的範例:
var obj BlogPost

assert.IsNotNil(t, "obj", obj)
assert.StringEq(t, "obj.Type", obj.Type, "blogPost")
assert.IntEq(t, "obj.Comments", obj.Comments, 2)
assert.StringNotEq(t, "obj.Body", obj.Body, "")
```

Assertion libraries tend to either stop the test early (if `assert` calls
`t.Fatalf` or `panic`) or omit relevant information about what the test got
right:

```go
// 不好的範例:
package assert

func IsNotNil(t *testing.T, name string, val any) {
    if val == nil {
        t.Fatalf("Data %s = nil, want not nil", name)
    }
}

func StringEq(t *testing.T, name, got, want string) {
    if got != want {
        t.Fatalf("Data %s = %q, want %q", name, got, want)
    }
}
```

Complex assertion functions often do not provide [useful failure messages] and
context that exists within the test function. Too many assertion functions and
libraries lead to a fragmented developer experience: which assertion library
should I use, what style of output format should it emit, etc.? Fragmentation
produces unnecessary confusion, especially for library maintainers and authors
of large-scale changes, who are responsible for fixing potential downstream
breakages. Instead of creating a domain-specific language for testing, use Go
itself.

Assertion libraries often factor out comparisons and equality checks. Prefer
using standard libraries such as [`cmp`] and [`fmt`] instead:

```go
// 好的範例:
var got BlogPost

want := BlogPost{
    Comments: 2,
    Body:     "Hello, world!",
}

if !cmp.Equal(got, want) {
    t.Errorf("Blog post = %v, want = %v", got, want)
}
```

For more domain-specific comparison helpers, prefer returning a value or an
error that can be used in the test's failure message instead of passing
`*testing.T` and calling its error reporting methods:

```go
// 好的範例:
func postLength(p BlogPost) int { return len(p.Body) }

func TestBlogPost_VeritableRant(t *testing.T) {
    post := BlogPost{Body: "I am Gunnery Sergeant Hartman, your senior drill instructor."}

    if got, want := postLength(post), 60; got != want {
        t.Errorf("Length of post = %v, want %v", got, want)
    }
}
```

**Best Practice:** Were `postLength` non-trivial, it would make sense to test it
directly, independently of any tests that use it.

See also:

*   [Equality comparison and diffs](#types-of-equality)
*   [Print diffs](#print-diffs)
*   For more on the distinction between test helpers and assertion helpers, see
    [best practices](best-practices#test-functions)

[useful failure messages]: #useful-test-failures
[`fmt`]: https://golang.org/pkg/fmt/
[marking test helpers]: #mark-test-helpers

<a id="identify-the-function"></a>

### Identify the function

In most tests, failure messages should include the name of the function that
failed, even though it seems obvious from the name of the test function.
Specifically, your failure message should be `YourFunc(%v) = %v, want %v`
instead of just `got %v, want %v`.

<a id="identify-the-input"></a>

### Identify the input

In most tests, failure messages should include the function inputs if they are
short. If the relevant properties of the inputs are not obvious (for example,
because the inputs are large or opaque), you should name your test cases with a
description of what's being tested and print the description as part of your
error message.

<a id="got-before-want"></a>

### Got before want

Test outputs should include the actual value that the function returned before
printing the value that was expected. A standard format for printing test
outputs is `YourFunc(%v) = %v, want %v`. Where you would write "actual" and
"expected", prefer using the words "got" and "want", respectively.

For diffs, directionality is less apparent, and as such it is important to
include a key to aid in interpreting the failure. See the
[section on printing diffs]. Whichever diff order you use in your failure
messages, you should explicitly indicate it as a part of the failure message,
because existing code is inconsistent about the ordering.

[section on printing diffs]: #print-diffs

<a id="compare-full-structures"></a>

### Full structure comparisons

If your function returns a struct (or any data type with multiple fields such as
slices, arrays, and maps), avoid writing test code that performs a hand-coded
field-by-field comparison of the struct. Instead, construct the data that you're
expecting your function to return, and compare directly using a
[deep comparison].

**Note:** This does not apply if your data contains irrelevant fields that
obscure the intention of the test.

If your struct needs to be compared for approximate (or equivalent kind of
semantic) equality or it contains fields that cannot be compared for equality
(e.g., if one of the fields is an `io.Reader`), tweaking a [`cmp.Diff`] or
[`cmp.Equal`] comparison with [`cmpopts`] options such as
[`cmpopts.IgnoreInterfaces`] may meet your needs
([example](https://play.golang.org/p/vrCUNVfxsvF)).

If your function returns multiple return values, you don't need to wrap those in
a struct before comparing them. Just compare the return values individually and
print them.

```go
// 好的範例:
val, multi, tail, err := strconv.UnquoteChar(`\"Fran & Freddie's Diner\"`, '"')
if err != nil {
  t.Fatalf(...)
}
if val != `"` {
  t.Errorf(...)
}
if multi {
  t.Errorf(...)
}
if tail != `Fran & Freddie's Diner"` {
  t.Errorf(...)
}
```

[deep comparison]: #types-of-equality
[`cmpopts`]: https://pkg.go.dev/github.com/google/go-cmp/cmp/cmpopts
[`cmpopts.IgnoreInterfaces`]: https://pkg.go.dev/github.com/google/go-cmp/cmp/cmpopts#IgnoreInterfaces

<a id="compare-stable-results"></a>

### Compare stable results

Avoid comparing results that may depend on output stability of a package that
you do not own. Instead, the test should compare on semantically relevant
information that is stable and resistant to changes in dependencies. For
functionality that returns a formatted string or serialized bytes, it is
generally not safe to assume that the output is stable.

For example, [`json.Marshal`] can change (and has changed in the past) the
specific bytes that it emits. Tests that perform string equality on the JSON
string may break if the `json` package changes how it serializes the bytes.
Instead, a more robust test would parse the contents of the JSON string and
ensure that it is semantically equivalent to some expected data structure.

[`json.Marshal`]: https://golang.org/pkg/encoding/json/#Marshal

<a id="keep-going"></a>

### Keep going

Tests should keep going for as long as possible, even after a failure, in order
to print out all of the failed checks in a single run. This way, a developer who
is fixing the failing test doesn't have to re-run the test after fixing each bug
to find the next bug.

Prefer calling `t.Error` over `t.Fatal` for reporting a mismatch. When comparing
several different properties of a function's output, use `t.Error` for each of
those comparisons.

Calling `t.Fatal` is primarily useful for reporting an unexpected error
condition, when subsequent comparison failures are not going to be meaningful.

For table-driven test, consider using subtests and use `t.Fatal` rather than
`t.Error` and `continue`. See also
[GoTip #25: Subtests: Making Your Tests Lean](https://google.github.io/styleguide/go/index.html#gotip).

**Best practice:** For more discussion about when `t.Fatal` should be used, see
[best practices](best-practices#t-fatal).

<a id="types-of-equality"></a>

### Equality comparison and diffs

The `==` operator evaluates equality using [language-defined comparisons].
Scalar values (numbers, booleans, etc) are compared based on their values, but
only some structs and interfaces can be compared in this way. Pointers are
compared based on whether they point to the same variable, rather than based on
the equality of the values to which they point.

The [`cmp`] package can compare more complex data structures not appropriately
handled by `==`, such as slices. Use [`cmp.Equal`] for equality comparison and
[`cmp.Diff`] to obtain a human-readable diff between objects.

```go
// 好的範例:
want := &Doc{
    Type:     "blogPost",
    Comments: 2,
    Body:     "This is the post body.",
    Authors:  []string{"isaac", "albert", "emmy"},
}
if !cmp.Equal(got, want) {
    t.Errorf("AddPost() = %+v, want %+v", got, want)
}
```

As a general-purpose comparison library, `cmp` may not know how to compare
certain types. For example, it can only compare protocol buffer messages if
passed the [`protocmp.Transform`] option.

<!-- The order of want and got here is deliberate. See comment in #print-diffs. -->

```go
// 好的範例:
if diff := cmp.Diff(want, got, protocmp.Transform()); diff != "" {
    t.Errorf("Foo() returned unexpected difference in protobuf messages (-want +got):\n%s", diff)
}
```

Although the `cmp` package is not part of the Go standard library, it is
maintained by the Go team and should produce stable equality results over time.
It is user-configurable and should serve most comparison needs.

[language-defined comparisons]: http://golang.org/ref/spec#Comparison_operators
[`cmp`]: https://pkg.go.dev/github.com/google/go-cmp/cmp
[`cmp.Equal`]: https://pkg.go.dev/github.com/google/go-cmp/cmp#Equal
[`cmp.Diff`]: https://pkg.go.dev/github.com/google/go-cmp/cmp#Diff
[`protocmp.Transform`]: https://pkg.go.dev/google.golang.org/protobuf/testing/protocmp#Transform

Existing code may make use of the following older libraries, and may continue
using them for consistency:

*   [`pretty`] produces aesthetically pleasing difference reports. However, it
    quite deliberately considers values that have the same visual representation
    as equal. In particular, `pretty` does not catch differences between nil
    slices and empty ones, is not sensitive to different interface
    implementations with identical fields, and it is possible to use a nested
    map as the basis for comparison with a struct value. It also serializes the
    entire value into a string before producing a diff, and as such is not a
    good choice for comparing large values. By default, it compares unexported
    fields, which makes it sensitive to changes in implementation details in
    your dependencies. For this reason, it is not appropriate to use `pretty` on
    protobuf messages.

[`pretty`]: https://pkg.go.dev/github.com/kylelemons/godebug/pretty

Prefer using `cmp` for new code, and it is worth considering updating older code
to use `cmp` where and when it is practical to do so.

Older code may use the standard library `reflect.DeepEqual` function to compare
complex structures. `reflect.DeepEqual` should not be used for checking
equality, as it is sensitive to changes in unexported fields and other
implementation details. Code that is using `reflect.DeepEqual` should be updated
to one of the above libraries.

**Note:** The `cmp` package is designed for testing, rather than production use.
As such, it may panic when it suspects that a comparison is performed
incorrectly to provide instruction to users on how to improve the test to be
less brittle. Given cmp's propensity towards panicking, it makes it unsuitable
for code that is used in production as a spurious panic may be fatal.

<a id="level-of-detail"></a>

### Level of detail

The conventional failure message, which is suitable for most Go tests, is
`YourFunc(%v) = %v, want %v`. However, there are cases that may call for more or
less detail:

*   Tests performing complex interactions should describe the interactions too.
    For example, if the same `YourFunc` is called several times, identify which
    call failed the test. If it's important to know any extra state of the
    system, include that in the failure output (or at least in the logs).
*   If the data is a complex struct with significant boilerplate, it is
    acceptable to describe only the important parts in the message, but do not
    overly obscure the data.
*   Setup failures do not require the same level of detail. If a test helper
    populates a Spanner table but Spanner was down, you probably don't need to
    include which test input you were going to store in the database.
    `t.Fatalf("Setup: Failed to set up test database: %s", err)` is usually
    helpful enough to resolve the issue.

**Tip:** Make your failure mode trigger during development. Review what the
failure message looks like and whether a maintainer can effectively deal with
the failure.

There are some techniques for reproducing test inputs and outputs clearly:

*   When printing string data, [`%q` is often useful](#use-percent-q) to
    emphasize that the value is important and to more easily spot bad values.
*   When printing (small) structs, `%+v` can be more useful than `%v`.
*   When validation of larger values fails, [printing a diff](#print-diffs) can
    make it easier to understand the failure.

<a id="print-diffs"></a>

### Print diffs

If your function returns large output then it can be hard for someone reading
the failure message to find the differences when your test fails. Instead of
printing both the returned value and the wanted value, make a diff.

To compute diffs for such values, `cmp.Diff` is preferred, particularly for new
tests and new code, but other tools may be used. See [types of equality] for
guidance regarding the strengths and weaknesses of each function.

*   [`cmp.Diff`]

*   [`pretty.Compare`]

You can use the [`diff`] package to compare multi-line strings or lists of
strings. You can use this as a building block for other kinds of diffs.

[types of equality]: #types-of-equality
[`diff`]: https://pkg.go.dev/github.com/kylelemons/godebug/diff
[`pretty.Compare`]: https://pkg.go.dev/github.com/kylelemons/godebug/pretty#Compare

Add some text to your failure message explaining the direction of the diff.

<!--
The reversed order of want and got in these examples is intentional, as this is
the prevailing order across the Google codebase. The lack of a stance on which
order to use is also intentional, as there is no consensus which is
"most readable."


-->

*   Something like `diff (-want +got)` is good when you're using the `cmp`,
    `pretty`, and `diff` packages (if you pass `(want, got)` to the function),
    because the `-` and `+` that you add to your format string will match the
    `-` and `+` that actually appear at the beginning of the diff lines. If you
    pass `(got, want)` to your function, the correct key would be `(-got +want)`
    instead.

*   The `messagediff` package uses a different output format, so the message
    `diff (want -> got)` is appropriate when you're using it (if you pass
    `(want, got)` to the function), because the direction of the arrow will
    match the direction of the arrow in the "modified" lines.

The diff will span multiple lines, so you should print a newline before you
print the diff.

<a id="test-error-semantics"></a>

### Test error semantics

When a unit test performs string comparisons or uses a vanilla `cmp` to check
that particular kinds of errors are returned for particular inputs, you may find
that your tests are brittle if any of those error messages are reworded in the
future. Since this has the potential to turn your unit test into a change
detector (see [TotT: Change-Detector Tests Considered Harmful][tott-350] ),
don't use string comparison to check what type of error your function returns.
However, it is permissible to use string comparisons to check that error
messages coming from the package under test satisfy certain properties, for
example, that it includes the parameter name.

Error values in Go typically have a component intended for human eyes and a
component intended for semantic control flow. Tests should seek to only test
semantic information that can be reliably observed, rather than display
information that is intended for human debugging, as this is often subject to
future changes. For guidance on constructing errors with semantic meaning see
[best-practices regarding errors](best-practices#error-handling). If an error
with insufficient semantic information is coming from a dependency outside your
control, consider filing a bug against the owner to help improve the API, rather
than relying on parsing the error message.

Within unit tests, it is common to only care whether an error occurred or not.
If so, then it is sufficient to only test whether the error was non-nil when you
expected an error. If you would like to test that the error semantically matches
some other error, then consider using [`errors.Is`] or `cmp` with
[`cmpopts.EquateErrors`].

> **Note:** If a test uses [`cmpopts.EquateErrors`] but all of its `wantErr`
> values are either `nil` or `cmpopts.AnyError`, then using `cmp` is
> [unnecessary mechanism](guide#least-mechanism). Simplify the code by making
> the want field a `bool`. You can then use a simple comparison with `!=`.
>
> ```go
> // 好的範例:
> err := f(test.input)
> gotErr := err != nil
> if gotErr != test.wantErr {
>     t.Errorf("f(%q) = %v, want error presence = %v", test.input, err, test.wantErr)
> }
> ```

See also
[GoTip #13: Designing Errors for Checking](https://google.github.io/styleguide/go/index.html#gotip).

[tott-350]: https://testing.googleblog.com/2015/01/testing-on-toilet-change-detector-tests.html
[`cmpopts.EquateErrors`]: https://pkg.go.dev/github.com/google/go-cmp/cmp/cmpopts#EquateErrors
[`errors.Is`]: https://pkg.go.dev/errors#Is

<a id="test-structure"></a>

## Test structure

<a id="subtests"></a>

### Subtests

The standard Go testing library offers a facility to [define subtests]. This
allows flexibility in setup and cleanup, controlling parallelism, and test
filtering. Subtests can be useful (particularly for table-driven tests), but
using them is not mandatory. See also the
[Go blog post about subtests](https://blog.golang.org/subtests).

Subtests should not depend on the execution of other cases for success or
initial state, because subtests are expected to be able to be run individually
with using `go test -run` flags or with Bazel [test filter] expressions.

[define subtests]: https://pkg.go.dev/testing#hdr-Subtests_and_Sub_benchmarks
[test filter]: https://bazel.build/docs/user-manual#test-filter

<a id="subtest-names"></a>

#### Subtest names

Name your subtest such that it is readable in test output and useful on the
command line for users of test filtering. When you use `t.Run` to create a
subtest, the first argument is used as a descriptive name for the test. To
ensure that test results are legible to humans reading the logs, choose subtest
names that will remain useful and readable after escaping. Think of subtest
names more like a function identifier than a prose description. The test runner
replaces spaces with underscores, and escapes non-printing characters. If your
test data benefits from a longer description, consider putting the description
in a separate field (perhaps to be printed using `t.Log` or alongside failure
messages).

Subtests may be run individually using flags to the [Go test runner] or Bazel
[test filter], so choose descriptive names that are also easy to type.

> **Warning:** Slash characters are particularly unfriendly in subtest names,
> since they have [special meaning for test filters].
>
> > ```sh
> > # 不好的範例:
> > # Assuming TestTime and t.Run("America/New_York", ...)
> > bazel test :mytest --test_filter="Time/New_York"    # Runs nothing!
> > bazel test :mytest --test_filter="Time//New_York"   # Correct, but awkward.
> > ```

To [identify the inputs] of the function, include them in the test's failure
messages, where they won't be escaped by the test runner.

```go
// 好的範例:
func TestTranslate(t *testing.T) {
    data := []struct {
        name, desc, srcLang, dstLang, srcText, wantDstText string
    }{
        {
            name:        "hu=en_bug-1234",
            desc:        "regression test following bug 1234. contact: cleese",
            srcLang:     "hu",
            srcText:     "cigarettát és egy öngyújtót kérek",
            dstLang:     "en",
            wantDstText: "cigarettes and a lighter please",
        }, // ...
    }
    for _, d := range data {
        t.Run(d.name, func(t *testing.T) {
            got := Translate(d.srcLang, d.dstLang, d.srcText)
            if got != d.wantDstText {
                t.Errorf("%s\nTranslate(%q, %q, %q) = %q, want %q",
                    d.desc, d.srcLang, d.dstLang, d.srcText, got, d.wantDstText)
            }
        })
    }
}
```

Here are a few examples of things to avoid:

```go
// 不好的範例:
// Too wordy.
t.Run("check that there is no mention of scratched records or hovercrafts", ...)
// Slashes cause problems on the command line.
t.Run("AM/PM confusion", ...)
```

See also
[Go Tip #117: Subtest Names](https://google.github.io/styleguide/go/index.html#gotip).

[Go test runner]: https://golang.org/cmd/go/#hdr-Testing_flags
[identify the inputs]: #identify-the-input
[special meaning for test filters]: https://blog.golang.org/subtests#:~:text=Perhaps%20a%20bit,match%20any%20tests

<a id="table-driven-tests"></a>

### Table-driven tests

Use table-driven tests when many different test cases can be tested using
similar testing logic.

*   When testing whether the actual output of a function is equal to the
    expected output. For example, the many [tests of `fmt.Sprintf`] or the
    minimal snippet below.
*   When testing whether the outputs of a function always conform to the same
    set of invariants. For example, [tests for `net.Dial`].

[tests of `fmt.Sprintf`]: https://cs.opensource.google/go/go/+/master:src/fmt/fmt_test.go
[tests for `net.Dial`]: https://cs.opensource.google/go/go/+/master:src/net/dial_test.go;l=318;drc=5b606a9d2b7649532fe25794fa6b99bd24e7697c

Here is the minimal structure of a table-driven test. If needed, you may use
different names or add extra facilities such as subtests or setup and cleanup
functions. Always keep [useful test failures](#useful-test-failures) in mind.

```go
// 好的範例:
func TestCompare(t *testing.T) {
    compareTests := []struct {
        a, b string
        want int
    }{
        {"", "", 0},
        {"a", "", 1},
        {"", "a", -1},
        {"abc", "abc", 0},
        {"ab", "abc", -1},
        {"abc", "ab", 1},
        {"x", "ab", 1},
        {"ab", "x", -1},
        {"x", "a", 1},
        {"b", "x", -1},
        // test runtime·memeq's chunked implementation
        {"abcdefgh", "abcdefgh", 0},
        {"abcdefghi", "abcdefghi", 0},
        {"abcdefghi", "abcdefghj", -1},
    }

    for _, test := range compareTests {
        got := Compare(test.a, test.b)
        if got != test.want {
            t.Errorf("Compare(%q, %q) = %v, want %v", test.a, test.b, got, test.want)
        }
    }
}
```

**Note**: The failure messages in this example above fulfill the guidance to
[identify the function](#identify-the-function) and
[identify the input](#identify-the-input). There's no need to
[identify the row numerically](#table-tests-identifying-the-row).

When some test cases need to be checked using different logic from other test
cases, it is more appropriate to write multiple test functions, as explained in
[GoTip #50: Disjoint Table Tests]. The logic of your test code can get difficult
to understand when each entry in a table has its own different conditional logic
to check each output for its inputs. If test cases have different logic but
identical setup, a sequence of [subtests](#subtests) within a single test
function might make sense.

You can combine table-driven tests with multiple test functions. For example,
when testing that a function's output exactly matches the expected output and
that the function returns a non-nil error for an invalid input, then writing two
separate table-driven test functions is the best approach: one for normal
non-error outputs, and one for error outputs.

[GoTip #50: Disjoint Table Tests]: https://google.github.io/styleguide/go/index.html#gotip

<a id="table-tests-data-driven"></a>

#### Data-driven test cases

Table test rows can sometimes become complicated, with the row values dictating
conditional behavior inside the test case. The extra clarity from the
duplication between the test cases is necessary for readability.

```go
// 好的範例:
type decodeCase struct {
    name   string
    input  string
    output string
    err    error
}

func TestDecode(t *testing.T) {
    // setupCodex is slow as it creates a real Codex for the test.
    codex := setupCodex(t)

    var tests []decodeCase // rows omitted for brevity

    for _, test := range tests {
        t.Run(test.name, func(t *testing.T) {
            output, err := Decode(test.input, codex)
            if got, want := output, test.output; got != want {
                t.Errorf("Decode(%q) = %v, want %v", test.input, got, want)
            }
            if got, want := err, test.err; !cmp.Equal(got, want) {
                t.Errorf("Decode(%q) err %q, want %q", test.input, got, want)
            }
        })
    }
}

func TestDecodeWithFake(t *testing.T) {
    // A fakeCodex is a fast approximation of a real Codex.
    codex := newFakeCodex()

    var tests []decodeCase // rows omitted for brevity

    for _, test := range tests {
        t.Run(test.name, func(t *testing.T) {
            output, err := Decode(test.input, codex)
            if got, want := output, test.output; got != want {
                t.Errorf("Decode(%q) = %v, want %v", test.input, got, want)
            }
            if got, want := err, test.err; !cmp.Equal(got, want) {
                t.Errorf("Decode(%q) err %q, want %q", test.input, got, want)
            }
        })
    }
}
```

In the counterexample below, note how hard it is to distinguish between which
type of `Codex` is used per test case in the case setup. (The highlighted parts
run afoul of the advice from [TotT: Data Driven Traps!][tott-97] .)

```go
// 不好的範例:
type decodeCase struct {
  name   string
  input  string
  codex  testCodex
  output string
  err    error
}

type testCodex int

const (
  fake testCodex = iota
  prod
)

func TestDecode(t *testing.T) {
  var tests []decodeCase // rows omitted for brevity

  for _, test := tests {
    t.Run(test.name, func(t *testing.T) {
      var codex Codex
      switch test.codex {
      case fake:
        codex = newFakeCodex()
      case prod:
        codex = setupCodex(t)
      default:
        t.Fatalf("Unknown codex type: %v", codex)
      }
      output, err := Decode(test.input, codex)
      if got, want := output, test.output; got != want {
        t.Errorf("Decode(%q) = %q, want %q", test.input, got, want)
      }
      if got, want := err, test.err; !cmp.Equal(got, want) {
        t.Errorf("Decode(%q) err %q, want %q", test.input, got, want)
      }
    })
  }
}
```

[tott-97]: https://testing.googleblog.com/2008/09/tott-data-driven-traps.html

<a id="table-tests-identifying-the-row"></a>

#### Identifying the row

Do not use the index of the test in the test table as a substitute for naming
your tests or printing the inputs. Nobody wants to go through your test table
and count the entries in order to figure out which test case is failing.

```go
// 不好的範例:
tests := []struct {
    input, want string
}{
    {"hello", "HELLO"},
    {"wORld", "WORLD"},
}
for i, d := range tests {
    if strings.ToUpper(d.input) != d.want {
        t.Errorf("Failed on case #%d", i)
    }
}
```

Add a test description to your test struct and print it along failure messages.
When using subtests, your subtest name should be effective in identifying the
row.

**Important:** Even though `t.Run` scopes the output and execution, you must
always [identify the input]. The table test row names must follow the
[subtest naming] guidance.

[identify the input]: #identify-the-input
[subtest naming]: #subtest-names

<a id="mark-test-helpers"></a>

### Test helpers

A test helper is a function that performs a setup or cleanup task. All failures
that occur in test helpers are expected to be failures of the environment (not
from the code under test) — for example when a test database cannot be started
because there are no more free ports on this machine.

If you pass a `*testing.T`, call [`t.Helper`] to attribute failures in the test
helper to the line where the helper is called. This parameter should come after
a [context](#contexts) parameter, if present, and before any remaining
parameters.

```go
// 好的範例:
func TestSomeFunction(t *testing.T) {
    golden := readFile(t, "testdata/golden-result.txt")
    // ... tests against golden ...
}

// readFile returns the contents of a data file.
// It must only be called from the same goroutine as started the test.
func readFile(t *testing.T, filename string) string {
    t.Helper()
    contents, err := runfiles.ReadFile(filename)
    if err != nil {
        t.Fatal(err)
    }
    return string(contents)
}
```

Do not use this pattern when it obscures the connection between a test failure
and the conditions that led to it. Specifically, the guidance about
[assert libraries](#assert) still applies, and [`t.Helper`] should not be used
to implement such libraries.

**Tip:** For more on the distinction between test helpers and assertion helpers,
see [best practices](best-practices#test-functions).

Although the above refers to `*testing.T`, much of the advice stays the same for
benchmark and fuzz helpers.

[`t.Helper`]: https://pkg.go.dev/testing#T.Helper

<a id="test-package"></a>

### Test package

<a id="TOC-TestPackage"></a>

<a id="test-same-package"></a>

#### Tests in the same package

Tests may be defined in the same package as the code being tested.

To write a test in the same package:

*   Place the tests in a `foo_test.go` file
*   Use `package foo` for the test file
*   Do not explicitly import the package to be tested

```build
# 好的範例:
go_library(
    name = "foo",
    srcs = ["foo.go"],
    deps = [
        ...
    ],
)

go_test(
    name = "foo_test",
    size = "small",
    srcs = ["foo_test.go"],
    library = ":foo",
    deps = [
        ...
    ],
)
```

A test in the same package can access unexported identifiers in the package.
This may enable better test coverage and more concise tests. Be aware that any
[examples] declared in the test will not have the package names that a user will
need in their code.

[`library`]: https://github.com/bazelbuild/rules_go/blob/master/docs/go/core/rules.md#go_library
[examples]: #examples

<a id="test-different-package"></a>

#### Tests in a different package

It is not always appropriate or even possible to define a test in the same
package as the code being tested. In these cases, use a package name with the
`_test` suffix. This is an exception to the "no underscores" rule to
[package names](#package-names). For example:

*   If an integration test does not have an obvious library that it belongs to

    ```go
    // 好的範例:
    package gmailintegration_test

    import "testing"
    ```

*   If defining the tests in the same package results in circular dependencies

    ```go
    // 好的範例:
    package fireworks_test

    import (
      "fireworks"
      "fireworkstestutil" // fireworkstestutil also imports fireworks
    )
    ```

<a id="use-package-testing"></a>

### Use package `testing`

The Go standard library provides the [`testing` package]. This is the only
testing framework permitted for Go code in the Google codebase. In particular,
[assertion libraries](#assert) and third-party testing frameworks are not
allowed.

The `testing` package provides a minimal but complete set of functionality for
writing good tests:

*   Top-level tests
*   Benchmarks
*   [Runnable examples](https://blog.golang.org/examples)
*   Subtests
*   Logging
*   Failures and fatal failures

These are designed to work cohesively with core language features like
[composite literal] and [if-with-initializer] syntax to enable test authors to
write [clear, readable, and maintainable tests].

[`testing` package]: https://pkg.go.dev/testing
[composite literal]: https://go.dev/ref/spec#Composite_literals
[if-with-initializer]: https://go.dev/ref/spec#If_statements

<a id="non-decisions"></a>

## Non-decisions

A style guide cannot enumerate positive prescriptions for all matters, nor can
it enumerate all matters about which it does not offer an opinion. That said,
here are a few things where the readability community has previously debated and
has not achieved consensus about.

*   **Local variable initialization with zero value**. `var i int` and `i := 0`
    are equivalent. See also [initialization best practices].
*   **Empty composite literal vs. `new` or `make`**. `&File{}` and `new(File)`
    are equivalent. So are `map[string]bool{}` and `make(map[string]bool)`. See
    also [composite declaration best practices].
*   **got, want argument ordering in cmp.Diff calls**. Be locally consistent,
    and [include a legend](#print-diffs) in your failure message.
*   **`errors.New` vs `fmt.Errorf` on non-formatted strings**.
    `errors.New("foo")` and `fmt.Errorf("foo")` may be used interchangeably.

If there are special circumstances where they come up again, the readability
mentor might make an optional comment, but in general the author is free to pick
the style they prefer in the given situation.

Naturally, if anything not covered by the style guide does need more discussion,
authors are welcome to ask -- either in the specific review, or on internal
message boards.

[composite declaration best practices]: https://google.github.io/styleguide/go/best-practices#vardeclcomposite
[initialization best practices]: https://google.github.io/styleguide/go/best-practices#vardeclinitialization
