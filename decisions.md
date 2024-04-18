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

* **混合大小寫**：見 [指南#混合大小寫](guide#mixed-caps)
    <a id="mixed-caps"></a>

* **格式化**：見 [指南#格式化](guide#formatting)
    <a id="formatting"></a>

* **行長**：見 [指南#行長](guide#line-length)
    <a id="line-length"></a>

<a id="naming"></a>

## 命名

有關命名的總體指導，請參見 [核心風格指南](guide#naming) 中的命名部分。以下部分對命名的特定領域提供了進一步的澄清。

<a id="underscores"></a>

### 底線

Go 中的名稱一般不應包含底線。這個原則有三個例外：

1. 僅由生成的代碼導入的套件名稱可能包含底線。有關如何選擇多詞套件名稱的更多細節，請參見 [套件名稱](#package-names)。
1. `*_test.go` 文件中的測試、基準測試和示例函數名稱可能包含底線。
1. 與操作系統或 cgo 互操作的低級庫可能會重用標識符，如 [`syscall`] 所做的那樣。這在大多數代碼庫中預期非常罕見。

[`syscall`]: https://pkg.go.dev/syscall#pkg-constants

<a id="package-names"></a>

### 套件名稱

<a id="TOC-PackageNames"></a>

Go 套件名稱應該短且僅包含小寫字母。由多個單詞組成的套件名稱應該保持不間斷的全小寫。例如，套件 [`tabwriter`] 不是命名為 `tabWriter`、`TabWriter` 或 `tab_writer`。

避免選擇可能被常用的局部變量名稱 [遮蔽] 的套件名稱。例如，`usercount` 是一個比 `count` 更好的套件名稱，因為 `count` 是一個常用的變量名稱。

Go 套件名稱不應有底線。如果您需要導入一個包含底線的套件名稱（通常來自生成的或第三方代碼），則必須在導入時重命名為適合在 Go 代碼中使用的名稱。

此規則的例外是，僅由生成的代碼導入的套件名稱可能包含底線。具體例子包括：

* 使用 `_test` 後綴的外部測試套件，例如集成測試

* 使用 `_test` 後綴的
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

* 短的（通常是一到兩個字母長）
* 該類型本身的縮寫
* 對該類型的每個接收者一致應用

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
// 不佳：
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

* 在包含多個首字母縮略詞的名稱中（例如 `XMLAPI` 因為它包含 `XML` 和 `API`），給定首字母縮略詞內的每個字母應該具有相同的大小寫，但名稱中的每個首字母縮略詞不需要具有相同的大小寫。
* 在包含含有小寫字母的首字母縮略詞的名稱中（例如 `DDoS`、`iOS`、`gRPC`），首字母縮略詞應該出現如同在標準散文中一樣，除非您需要為了 [導出性] 改變第一個字母。在這些情況下，整個首字母縮略詞應該是相同的大小寫（例如 `ddos`、`IOS`、`GRPC`）。

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

* 小範圍是執行一兩個小操作的範圍，比如 1-7 行。
* 中等範圍是幾個小操作或一個大操作，比如 8-15 行。
* 大範圍是一個或幾個大操作，比如 15-25 行。
* 非常大的範圍是跨越超過一頁的任何範圍（比如，超過 25 行）。

[清晰性]: guide#clarity
[簡練性]: guide#concision

在小範圍內可能非常清晰的名稱（例如，`c` 用於計數器）在更大的範圍內可能不夠，並且需要澄清以提醒讀者在代碼中更遠的地方其用途。在有許多變量的範圍內，或者變量代表相似的值或概念，可能需要比範圍建議的更長的變量名稱。

概念的具體性也可以幫助保持變量名稱的簡練。例如，假設只使用一個數據庫，一個短變量名稱如 `db` 通常可能保留給非常小的範圍，即使範圍非常大也可能保持完全清晰。在這種情況下，單詞 `database` 基於範圍的大小可能是可以接受的，但不是必需的，因為 `db` 是該單詞的非常常見的縮寫，幾乎沒有其他解釋。

局部變量的名稱應該反映它包含的內容以及它在當前上下文中的使用方式，而不是值的來源。例如，通常情況下，最好的局部變量名稱與結構或協議緩衝區字段名稱不同。

一般來說：

* 單詞名稱如 `count` 或 `options` 是一個好的起點。
* 可以添加額外的單詞來消除相似名稱的歧義，例如 `userCount` 和 `projectCount`。
* 不要簡單地刪除字母來節省打字。例如，`Sandbox` 比 `Sbx` 更好，特別是對於導出的名稱。
* 從大多數變量名稱中省略 [類型和類型化的詞]。
    * 對於一個數字，`userCount` 是一個比 `numUsers` 或 `usersInt` 更好的名稱。
    * 對於一個切片，`users` 是一個比 `userSlice` 更好的名稱。
    * 如果範圍內有兩個版本的值，可以接受包含類型化的限定詞，例如你可能有一個存儲在 `ageString` 中的輸入，並使用 `age` 作為解析後的值。
* 省略從 [周圍上下文] 中清楚的詞。例如，在 `UserCount` 方法的實現中，一個叫做 `userCount` 的局部變量可能是多餘的；`count`、`users` 或甚至 `c` 一樣可讀。

[類型和類型化的詞]: #repetitive-with-type
[周圍上下文]: #repetitive-in-context

<a id="v"></a>

#### 單字母變量名稱

單字母變量名稱可以是一個有用的工具來最小化
[重複](#repetition)，但也可能使代碼不必要地晦澀。將它們的使用限制在完整單詞是明顯的情況下，並且如果單字母變量代替完整單詞會重複。

一般來說：

* 對於 [方法接收者變量]，一個字母或兩個字母的名稱是首選。
* 對於常見類型使用熟悉的變量名稱通常很有幫助：
    * `r` 用於 `io.Reader` 或 `*http.Request`
    * `w` 用於 `io.Writer` 或 `http.ResponseWriter`
* 單字母標識符作為整數循環變量是可以接受的，特別是對於索引（例如，`i`）和坐標（例如，`x` 和 `y`）。
* 當範圍短時，縮寫可以作為可接受的循環標識符，例如 `for _, n := range nodes { ... }`。

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
> * `widget.NewWidget` -> `widget.New`
> * `widget.NewWidgetWithName` -> `widget.NewWithName`
> * `db.LoadFromDatabase` -> `db.Load`
> * `goatteleportutil.CountGoatsTeleported` -> `gtutil.CountGoatsTeleported`
>     或 `goatteleport.Count`
> * `myteampb.MyTeamMethodRequest` -> `mtpb.MyTeamMethodRequest` 或
>     `myteampb.MethodRequest`

<a id="repetitive-with-type"></a>

#### 套件與導出符號名稱

在命名導出符號時，套件的名稱在套件外總是可見的，因此兩者之間的冗餘信息應該被減少或消除。如果一個套件只導出一種類型，且以套件本身命名，則構造函數的標準名稱為 `New`（如果需要的話）。

> **範例：** 重複的名稱 -> 更好的名稱
>
> * `widget.NewWidget` -> `widget.New`
> * `widget.NewWidgetWithName` -> `widget.NewWithName`
> * `db.LoadFromDatabase` -> `db.Load`
> * `goatteleportutil.CountGoatsTeleported` -> `gtutil.CountGoatsTeleported`
>     或 `goatteleport.Count`
> * `myteampb.MyTeamMethodRequest` -> `mtpb.MyTeamMethodRequest` 或
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
// 較佳：
limitStr := r.FormValue("limit")
limit, err := strconv.Atoi(limitStr)
```

<a id="repetitive-in-context"></a>

#### 外部上下文與本地名稱

包含來自其周圍上下文信息的名稱經常創造額外的噪音而沒有好處。套件名稱、方法名稱、類型名稱、函數名稱、導入路徑，甚至檔案名稱都可以提供自動資格所有內部名稱的上下文。

```go
// 不佳：
// 在套件 "ads/targeting/revenue/reporting"
type AdsTargetingRevenueReport struct{}

func (p *Project) ProjectName() string
```

```go
// 較佳：
// 在套件 "ads/targeting/revenue/reporting"
type Report struct{}

func (p *Project) Name() string
```

```go
// 不佳：
// 在套件 "sqldb"
type DBConnection struct{}
```

```go
// 較佳：
// 在套件 "sqldb"
type Connection struct{}
```

```go
// 不佳：
// 在套件 "ads/targeting"
func Process(in *pb.FooProto) *Report {
    adsTargetingID := in.GetAdsTargetingID()
}
```

```go
// 較佳：
// 在套件 "ads/targeting"
func Process(in *pb.FooProto) *Report {
    id := in.GetAdsTargetingID()
}
```

重複通常應該在使用符號的用戶的上下文中評估，而不是孤立地。例如，以下代碼有很多名稱在某些情況下可能是好的，但在上下文中是多餘的：

```go
// 不佳：
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
// 較佳：
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
# 較佳：
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
# 不佳：
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
// 較佳：
// A Request represents a request to run a command.
type Request struct { ...

// Encode writes the JSON encoding of req to w.
func Encode(w io.Writer, req *Request) { ...
```

文件註解出現在 [Godoc](https://pkg.go.dev/) 中，並且會被 IDE 展示，因此應該為使用套件的任何人編寫。

[完整句子]: #comment-sentences

文件註解適用於以下符號，或者如果它出現在結構體中則適用於字段群。

```go
// 較佳：
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
// 較佳：
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
// 較佳：
func (n *Node) Parent1() *Node
func (n *Node) Parent2() (*Node, error)
```

如果函數返回兩個或多個相同類型的參數，添加名稱可能會有用。

```go
// 較佳：
func (n *Node) Children() (left, right *Node, err error)
```

如果調用者必須對特定結果參數採取行動，命名它們可以幫助建議什麼是行動：

```go
// 較佳：
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
// 不佳：
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
// 較佳：
// Package math 提供基本常數和數學函數。
//
// 本套件不保證在不同架構間有位元完全相同的結果。
package math
```

每個套件必須有一個套件註解。如果一個套件由多個文件組成，則正好有一個文件應該有一個套件註解。

`main` 套件的註解有一種略微不同的形式，其中 BUILD 文件中的 `go_binary` 規則的名稱取代了套件名稱。

```go
// 較佳：
// seed_generator 命令是一個實用工具，它從一組 JSON 研究配置生成 Finch 種子文件。
package main
```

只要二進制名稱與 BUILD 文件中寫的完全一致，其他風格的註解也是可以的。當二進制名稱是第一個單詞時，即使它並不嚴格匹配命令行調用的拼寫，也需要將其大寫。

```go
// 較佳：
// 二進制 seed_generator ...
// 命令 seed_generator ...
// 程序 seed_generator ...
// seed_generator 命令 ...
// seed_generator 程序 ...
// Seed_generator ...
```

Tips:

* 示例命令行調用和 API 使用可以是有用的文檔。對於 Godoc 格式，縮進包含代碼的註解行。

* 如果沒有明顯的主文件，或者如果套件註解非常長，則可以接受將文檔註解放在一個名為 `doc.go` 的文件中，該文件僅包含註解和套件子句。

* 多行註解可以代替多個單行註解。如果文檔包含可能需要從源文件中複製和粘貼的部分，這主要是有用的，如示例命令行（對於二進制文件）和模板示例。

    ```go
    // 較佳：
    /*
    seed_generator 命令是一個實用工具，它從一組 JSON 研究配置生成 Finch 種子文件。

        seed_generator *.json | base64 > finch-seed.base64
    */
    package template
    ```

* 旨在供維護者使用且適用於整個文件的註解通常放在導入聲明之後。這些在 Godoc 中不會顯示，也不受上述套件註解規則的約束。

<a id="imports"></a>

## 引入

<a id="TOC-Imports"></a>

<a id="import-renaming"></a>

### 引入重命名

引入只應該在與其他引入發生名稱衝突時才重命名。（由此衍生的一個結論是，[良好的套件名稱](#package-names)不應該需要重命名。）在名稱衝突發生時，優先重命名最本地或項目特定的引入。對於套件的本地名稱（別名）必須遵循[套件命名的指導原則](#package-names)，包括禁止使用下劃線和大寫字母。

生成的協議緩衝區套件必須重命名以去除其名稱中的下劃線，並且它們的別名必須有一個 `pb` 後綴。有關更多信息，請參見[proto 和 stub 最佳實踐]。

[proto 和 stub 最佳實踐]: best-practices#import-protos

```go
// 較佳：
import (
    fspb "path/to/package/foo_service_go_proto"
)
```

引入具有沒有有用識別信息的套件名稱（例如 `package v1`）時，應該重新命名以包含前一個路徑組件。重新命名必須與其他本地文件引入相同套件時保持一致，並且可以包含版本號。

**注意：** 建議將套件重新命名以符合[良好的套件名稱](#package-names)，但對於 vendored 目錄中的套件來說，這通常不可行。

```go
// 較佳：
import (
    core "github.com/kubernetes/api/core/v1"
    meta "github.com/kubernetes/apimachinery/pkg/apis/meta/v1beta1"
)
```

如果您需要引入一個與您想要使用的常見本地變量名稱衝突的套件（例如 `url`、`ssh`），並且您希望重新命名該套件，那麼建議的做法是使用 `pkg` 後綴（例如 `urlpkg`）。請注意，有可能用本地變量遮蔽一個套件；只有在這樣的變量在作用域內時仍需要使用該套件，才需要進行此重命名。

<a id="import-grouping"></a>

### 引入分組

引入應該組織成兩組：

* 標準庫套件

* 其他（專案和 vendored）套件

```go
// 較佳：
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

* 重新命名的引入
* 為了它們的副作用而引入的套件

範例:

```go
// 較佳：
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

* [time/tzdata](https://pkg.go.dev/time/tzdata)

* [image/jpeg](https://pkg.go.dev/image/jpeg) 在圖像處理代碼中

避免在庫套件中進行空白引入，即使庫間接依賴於它們。將副作用引入限制在主套件中有助於控制依賴關係，並使得撰寫依賴不同引入的測試成為可能，而不會產生衝突或浪費建構成本。

以下是此規則的唯一例外：

* 您可以使用空白引入來繞過 [nogo 靜態檢查器] 中對禁止引入的檢查。

* 您可以在使用 `//go:embed` 編譯器指令的源文件中，空白引入 [embed](https://pkg.go.dev/embed) 套件。

**提示：** 如果您創建了一個在生產中間接依賴副作用引入的庫套件，請記錄預期的使用方式。

[nogo 靜態檢查器]: https://github.com/bazelbuild/rules_go/blob/master/go/nogo.rst

<a id="import-dot"></a>

### 引入「點」（`import .`）

<a id="TOC-ImportDot"></a>

`import .` 形式是一種語言特性，它允許將另一個套件導出的標識符帶到當前套件中，無需資格限制。更多信息請參見 [語言規範](https://go.dev/ref/spec#Import_declarations)。

**不要** 在 Google 代碼庫中使用此功能；它使得更難判斷功能來自何處。

```go
// 不佳：
package foo_test

import (
    "bar/testutil" // also imports "foo"
    . "foo"
)

var myThing = Bar() // Bar defined in package foo; no qualification needed.
```

```go
// 較佳：
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
// 較佳：
func Good() error { /* ... */ }
```

返回一個 `nil` 錯誤是表示一個本可以失敗的操作成功的慣用方法。如果一個函數返回一個錯誤，除非另有明確文檔記載，否則調用者必須將所有非錯誤返回值視為未指定。通常，非錯誤返回值是它們的零值，但這不能假定。

```go
// 較佳：
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
// 不佳：
func Bad() *os.PathError { /*...*/ }
```

**提示**：一個接受 `context.Context` 參數的函數通常應該返回一個 `error`，以便調用者可以確定在函數運行時上下文是否被取消。

[nil error]: https://golang.org/doc/faq#nil_error

<a id="error-strings"></a>

### 錯誤字串

<a id="TOC-ErrorStrings"></a>

錯誤字串不應該首字大寫（除非以導出名稱、專有名詞或首字母縮略詞開頭），並且不應該以標點符號結尾。這是因為錯誤字串通常會在打印給用戶之前出現在其他上下文中。

```go
// 不佳：
err := fmt.Errorf("Something bad happened.")
```

```go
// 較佳：
err := fmt.Errorf("something bad happened")
```

另一方面，完整顯示消息（日誌、測試失敗、API 回應或其他 UI）的風格取決於具體情況，但通常應該首字大寫。

```go
// 較佳：
log.Infof("Operation aborted: %v", err)
log.Errorf("Operation aborted: %v", err)
t.Errorf("Op(%q) failed unexpectedly; err=%v", args, err)
```

<a id="handle-errors"></a>

### 處理錯誤

<a id="TOC-HandleErrors"></a>

遇到錯誤的代碼應該有意識地選擇如何處理它。通常不適合使用 `_` 變量來丟棄錯誤。如果函數返回一個錯誤，請執行以下操作之一：

* 立即處理並解決錯誤。
* 將錯誤返回給調用者。
* 在特殊情況下，調用 [`log.Fatal`] 或（如果絕對必要）`panic`。

**注意：** `log.Fatalf` 不是標準庫日誌。參見 [#logging]。

在極少數適合忽略或丟棄錯誤的情況下（例如，對 [`(*bytes.Buffer).Write`] 的調用，該調用被記錄為永遠不會失敗），應該有一個相應的註解解釋為什麼這樣做是安全的。

```go
// 較佳：
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
// 不佳：
// Lookup returns the value for key or -1 if there is no mapping for key.
func Lookup(key string) int
```

未檢查內部錯誤值可能導致錯誤，並將錯誤歸咎於錯誤的函數。

```go
// 不佳：
// The following line returns an error that Parse failed for the input value,
// whereas the failure was that there is no mapping for missingKey.
return Parse(Lookup(missingKey))
```

Go 對多返回值的支持提供了一個更好的解決方案（參見[Effective Go 關於多重返回的部分]）。函數不應要求客戶端檢查內部錯誤值，而應返回一個額外的值來指示其其他返回值是否有效。這個返回值可能是一個錯誤或一個布林值（當不需要解釋時），並且應該是最後的返回值。

```go
// 較佳：
// Lookup returns the value for key or ok=false if there is no mapping for key.
func Lookup(key string) (value string, ok bool)
```

這種 API 防止調用者錯誤地寫 `Parse(Lookup(key))`，這會導致編譯時錯誤，因為 `Lookup(key)` 有 2 個輸出。

以這種方式返回錯誤鼓勵更健壯和明確的錯誤處理：

```go
// 較佳：
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
// 較佳：
if err != nil {
    // 錯誤處理
    return // 或者 continue 等等
}
// 正常代碼
```

```go
// 不佳：
// 不佳：
if err != nil {
    // 錯誤處理
} else {
    // 由於縮進，看起來不正常的正常代碼
}
```

> **提示：** 如果您在多於幾行代碼中使用變量，通常不值得使用帶初始化器的 `if` 風格。在這些情況下，通常最好將聲明移出並使用標準的 `if` 語句：
>
> ```go
> // 較佳：
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
> // 不佳：
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

* 包括來自其他套件的類型的字段名稱。

    ```go
    // 較佳：
    // https://pkg.go.dev/encoding/csv#Reader
    r := csv.Reader{
      Comma: ',',
      Comment: '#',
      FieldsPerRecord: 4,
    }
    ```

    結構中字段的位置和字段的完整集合（省略字段名稱時必須正確的兩個條件）通常不被認為是結構的公共 API 的一部分；指定字段名稱是為了避免不必要的耦合。

    ```go
    // 不佳：
    r := csv.Reader{',', '#', 4, false, false, false, false}
    ```

* 對於套件內部類型，字段名稱是可選的。

    ```go
    // 較佳：
    okay := Type{42}
    also := internalType{4, 2}
    ```

    如果使用字段名稱可以使代碼更清晰，則仍應使用字段名稱，這樣做是非常常見的。例如，具有大量字段的結構幾乎總是應該使用字段名稱進行初始化。

    <!-- TODO: Maybe a better example here that doesn't have many fields. -->

    ```go
    // 較佳：
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
// 較佳：
good := []*Type{{Key: "value"}}
```

```go
// 較佳：
good := []*Type{
    {Key: "multi"},
    {Key: "line"},
}
```

```go
// 不佳：
bad := []*Type{
    {Key: "multi"},
    {Key: "line"}}
```

```go
// 不佳：
bad := []*Type{
    {
        Key: "value"},
}
```

<a id="literal-cuddled-braces"></a>

#### 貼合的大括號 (Cuddled braces)

對於切片和數組字面量，只有在以下兩個條件都滿足時，才允許去掉大括號之間的空白（也就是所謂的「緊靠」它們）。

* [縮進匹配](#literal-matching-braces)
* 內部值也是字面量或 proto 構建器（即不是變量或其他表達式）

```go
// 較佳：
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
// 較佳：
good := []*Type{{ // Cuddled correctly
    Field: "value",
}, {
    Field: "value",
}}
```

```go
// 較佳：
good := []*Type{
    first, // Can't be cuddled
    {Field: "second"},
}
```

```go
// 較佳：
okay := []*pb.Type{pb.Type_builder{
    Field: "first", // Proto Builders may be cuddled to save vertical space
}.Build(), pb.Type_builder{
    Field: "second",
}.Build()}
```

```go
// 不佳：
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
// 較佳：
good := []*Type{
    {A: 42},
    {A: 43},
}
```

```go
// 不佳：
repetitive := []*Type{
    &Type{A: 42},
    &Type{A: 43},
}
```

```go
// 較佳：
good := map[Type1]*Type2{
    {A: 1}: {B: 2},
    {A: 3}: {B: 4},
}
```

```go
// 不佳：
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
// 不佳：
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
// 較佳：
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
// 較佳：
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
// 較佳：
var t []string
```

```go
// 不佳：
t := []string{}
```

不要創建強迫客戶端區分 nil 和空切片的 API。

```go
// 較佳：
// Ping 對其目標進行 ping 操作。
// 返回成功響應的主機。
func Ping(hosts []string) ([]string, error) { ... }
```

```go
// 不佳：
// Ping 對其目標進行 ping 操作並返回成功響應的主機列表。
// 如果輸入為空則可以為空。
// nil 表示發生了系統錯誤。
func Ping(hosts []string) []string { ... }
```

在設計介面時，避免區分 `nil` 切片和非 `nil`、長度為零的切片，因為這可能導致微妙的編程錯誤。這通常是通過使用 `len` 來檢查空值，而不是 `== nil` 來實現的。

這個實現接受 `nil` 和長度為零的切片作為「空」：

```go
// 較佳：
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
// 不佳：
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
// 不佳：
if longCondition1 && longCondition2 &&
    // Conditions 3 and 4 have the same indentation as the code within the if.
    longCondition3 && longCondition4 {
    log.Info("all conditions met")
}
```

請參閱以下部分以獲得具體指南和示例：

* [函數格式化](#func-formatting)
* [條件和循環](#conditional-formatting)
* [字面量格式化](#literal-formatting)

<a id="func-formatting"></a>

### 函數格式化

函數或方法聲明的簽名應保持在單行上，以避免[縮進混淆](#indentation-confusion)。

函數參數列表可能會使 Go 源文件中的某些行變得很長。然而，它們預示著縮進的變化，因此很難以不會使後續行看起來像是以令人困惑的方式屬於函數體的一部分的方式斷行：

```go
// 不佳：
func (r *SomeType) SomeLongFunctionName(foo1, foo2, foo3 string,
    foo4, foo5, foo6 int) {
    foo7 := bar(foo1)
    // ...
}
```

請參閱[最佳實踐](best-practices#funcargs)以獲得一些縮短具有許多參數的函數呼叫站點的選項。

通過提取局部變量，經常可以縮短行。

```go
// 較佳：
local := helper(some, parameters, here)
good := foo.Call(list, of, parameters, local)
```

同樣，函數和方法呼叫不應僅基於行長度而被分開。

```go
// 較佳：
good := foo.Call(long, list, of, parameters, all, on, one, line)
```

```go
// 不佳：
bad := foo.Call(long, list, of, parameters,
    with, arbitrary, line, breaks)
```

盡可能避免對特定函數參數添加內聯註釋。相反，使用[選項結構](best-practices#option-structure)或在函數文檔中添加更多細節。

```go
// 較佳：
good := server.New(ctx, server.Options{Port: 42})
```

```go
// 不佳：
bad := server.New(
    ctx,
    42, // Port
)
```

如果 API 無法更改或者局部呼叫是不尋常的（無論呼叫是否太長），如果它有助於理解呼叫，始終允許添加斷行。

```go
// 較佳：
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
// 較佳：
log.Warningf("Database key (%q, %d, %q) incompatible in transaction started by (%q, %d, %q)",
    currentCustomer, currentOffset, currentKey,
    txCustomer, txOffset, txKey)
```

```go
// 不佳：
log.Warningf("Database key (%q, %d, %q) incompatible in"+
    " transaction started by (%q, %d, %q)",
    currentCustomer, currentOffset, currentKey, txCustomer,
    txOffset, txKey)
```

<a id="conditional-formatting"></a>

### 條件語句和循環

`if` 語句不應該換行；多行 `if` 子句可能導致[縮進混淆](#indentation-confusion)。

```go
// 不佳：
// The second if statement is aligned with the code within the if block, causing
// indentation confusion.
if db.CurrentStatusIs(db.InTransaction) &&
    db.ValuesEqual(db.TransactionKey(), row.Key()) {
    return db.Errorf(db.TransactionError, "query failed: row (%v): key does not match transaction key", row)
}
```

如果不需要短路行為，則可以直接提取布林運算元：

```go
// 較佳：
inTransaction := db.CurrentStatusIs(db.InTransaction)
keysMatch := db.ValuesEqual(db.TransactionKey(), row.Key())
if inTransaction && keysMatch {
    return db.Error(db.TransactionError, "query failed: row (%v): key does not match transaction key", row)
}
```

也可能有其他局部變量可以被提取，特別是如果條件已經重複：

```go
// 較佳：
uid := user.GetUniqueUserID()
if db.UserIsAdmin(uid) || db.UserHasPermission(uid, perms.ViewServerConfig) || db.UserHasPermission(uid, perms.CreateGroup) {
    // ...
}
```

```go
// 不佳：
if db.UserIsAdmin(user.GetUniqueUserID()) || db.UserHasPermission(user.GetUniqueUserID(), perms.ViewServerConfig) || db.UserHasPermission(user.GetUniqueUserID(), perms.CreateGroup) {
    // ...
}
```

包含閉包或多行結構字面量的 `if` 語句應確保[大括號匹配](#literal-matching-braces)以避免[縮進混淆](#indentation-confusion)。

```go
// 較佳：
if err := db.RunInTransaction(func(tx *db.TX) error {
    return tx.Execute(userUpdate, x, y, z)
}); err != nil {
    return fmt.Errorf("user update failed: %s", err)
}
```

```go
// 較佳：
if _, err := client.Update(ctx, &upb.UserUpdateRequest{
    ID:   userID,
    User: user,
}); err != nil {
    return fmt.Errorf("user update failed: %s", err)
}
```

同樣，不要嘗試在 `for` 語句中插入人為的換行。如果沒有優雅的重構方式，可以讓行保持長度：

```go
// 較佳：
for i, max := 0, collection.Size(); i < max && !collection.HasPendingWriters(); i++ {
    // ...
}
```

然而，通常有辦法：

```go
// 較佳：
for i, max := 0, collection.Size(); i < max; i++ {
    if collection.HasPendingWriters() {
        break
    }
    // ...
}
```

`switch` 和 `case` 語句也應保持在單行上。

```go
// 較佳：
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
// 不佳：
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
// 較佳：
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
// 較佳：
if result == "foo" {
  // ...
}
```

而不是不太清晰的語句，其中常量首先出現（["Yoda 風格條件"](https://en.wikipedia.org/wiki/Yoda_conditions)）:

```go
// 不佳：
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
// 不佳：
b1 := bytes.Buffer{}
b2 := b1
```

調用一個接收值的方法可以隱藏複製行為。當你設計一個 API 時，如果你的結構體包含不應該被複製的字段，你應該通常接收並返回指針類型。

這些是可以接受的：

```go
// 較佳：
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
// 不佳：
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
// 較佳：
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
// 較佳：
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
// 不佳：
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
// 不佳：
ch := make(chan int)
ch <- 42
close(ch)
ch <- 13 // panic
```

在「結果不再需要後」修改仍在使用中的輸入可能導致數據競爭。任意長時間地保留 goroutines 在執行中可能導致不可預測的內存使用。

並發代碼應該寫得使 goroutine 的生命週期明顯。通常這將意味著將與同步相關的代碼限制在函數的範圍內，並將邏輯分解為[同步函數]。如果並發性仍然不明顯，記錄 goroutines 何時以及為什麼退出是很重要的。

遵循最佳實踐的上下文使用相關代碼通常有助於澄清這一點。它通常是用 `context.Context` 管理的：

```go
// 較佳：
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
// 不佳：
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

* 代碼在生產中可能有未定義的行為，即使操作系統釋放了資源，程序也可能無法乾淨地終止。
* 由於代碼的不確定生命週期，測試代碼意義不大。
* 如上所述，代碼可能泄漏資源。

另見：

* [永遠不要啟動一個 goroutine 而不知道它將如何停止][cheney-stop]
* 重新思考傳統並發模式：[幻燈片][rethinking-slides]，[視頻][rethinking-video]
* [Go 程序何時結束]

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
// 較佳：
package consumer // consumer.go

type Thinger interface { Thing() bool }

func Foo(t Thinger) string { ... }
```

```go
// 較佳：
package consumer // consumer_test.go

type fakeThinger struct{ ... }
func (t fakeThinger) Thing() bool { ... }
...
if Foo(fakeThinger{...}) == "x" { ... }
```

```go
// 不佳：
package producer

type Thinger interface { Thing() bool }

type defaultThinger struct{ ... }
func (t defaultThinger) Thing() bool { ... }

func NewThinger() Thinger { return defaultThinger{ ... } }
```

```go
// 較佳：
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

* [寫代碼，不要設計類型]。來自 Robert Griesemer 和 Ian Lance Taylor 的 GopherCon 演講。
* 如果你有幾種類型共享一個有用的統一介面，考慮使用該介面來建模解決方案。可能不需要泛型。
* 否則，不要依賴 `any` 類型和過度的[類型切換](https://tour.golang.org/methods/16)，考慮使用泛型。

另見：

* [在 Go 中使用泛型]，Ian Lance Taylor 的演講

* Go 網頁上的[泛型教程]

[泛型教程]: https://go.dev/doc/tutorial/generics
[類型參數]: https://go.dev/design/43651-type-parameters
[在 Go 中使用泛型]: https://www.youtube.com/watch?v=nr8EpUO9jhw
[寫代碼，不要設計類型]: https://www.youtube.com/watch?v=Pa_e9EeCdy8&t=1250s

<a id="pass-values"></a>

### 傳遞值 Pass values

<a id="TOC-PassValues"></a>

不要僅僅為了節省幾個字節就在函數參數中傳遞指針。如果一個函數只是作為 `*x` 讀取其參數 `x`，那麼該參數不應該是一個指針。這種情況的常見例子包括傳遞一個字符串的指針（`*string`）或一個介面值的指針（`*io.Reader`）。在這兩種情況下，值本身是固定大小，可以直接傳遞。

這條建議不適用於大型結構體，或者即使是小型結構體也可能增加大小。特別是，協議緩衝消息通常應該通過指針而不是值來處理。指針類型滿足 `proto.Message` 介面（被 `proto.Marshal`、`protocmp.Transform` 等接受），且協議緩衝消息可能相當大，並且經常隨時間增長。

<a id="receiver-type"></a>

### 接收器類型 Receiver type

<a id="TOC-ReceiverType"></a>

[方法接收器]可以作為值或指針傳遞，就像它是一個普通函數參數一樣。在兩者之間的選擇基於方法應該是哪個[方法集]的一部分。

[方法接收器]: https://golang.org/ref/spec#Method_declarations
[方法集]: https://golang.org/ref/spec#Method_sets

**正確性勝過速度或簡單性。** 有些情況下你必須使用指針值。在其他情況下，對於大型類型或作為未來證明如果你不確定代碼將如何增長，選擇指針；對於簡單的[普通舊數據]，使用值。

下面的列表進一步詳細說明了每種情況：

* 如果接收器是一個切片且方法不重新切片或重新分配切片，使用值而不是指針。

    ```go
    // 較佳：
    type Buffer []byte

    func (b Buffer) Len() int { return len(b) }
    ```

* 如果方法需要改變接收器，接收器必須是一個指針。

    ```go
    // 較佳：
    type Counter int

    func (c *Counter) Inc() { *c++ }

    // 參見 https://pkg.go.dev/container/heap。
    type Queue []Item

    func (q *Queue) Push(x Item) { *q = append([]Item{x}, *q...) }
    ```

* 如果接收器是一個包含[不能安全複製]的字段的結構體，使用指針接收器。常見例子是 [`sync.Mutex`] 和其他同步類型。

    ```go
    // 較佳：
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

    **提示：** 檢查類型的 [Godoc] 以獲取關於它是否安全或不安全複製的信息。

* 如果接收器是一個“大型”結構體或數組，指針接收器可能更有效。傳遞一個結構體等同於將其所有字段或元素作為參數傳遞給方法。如果這看起來太大而無法[按值傳遞]，指針是一個好選擇。

* 對於將與修改接收器的其他函數同時調用或運行的方法，如果這些修改不應該對你的方法可見，使用值；否則使用指針。

* 如果接收器是一個結構體或數組，其任何元素是指向可能被改變的東西的指針，優先選擇指針接收器以使可變性的意圖對讀者清晰。

    ```go
    // 較佳：
    type Counter struct {
        m *Metric
    }

    func (c *Counter) Inc() {
        c.m.Add(1)
    }
    ```

* 如果接收器是一個[內建類型]，如整數或字符串，不需要被修改，使用值。

    ```go
    // 較佳：
    type User string

    func (u User) String() { return string(u) }
    ```

* 如果接收器是一個映射、函數或通道，使用值而不是指針。

    ```go
    // 較佳：
    // 參見 https://pkg.go.dev/net/http#Header。
    type Header map[string][]string

    func (h Header) Add(key, value string) { /* 省略 */ }
    ```

* 如果接收器是一個“小型”數組或結構體，本質上是一個沒有可變字段和指針的值類型，值接收器通常是正確的選擇。

    ```go
    // 較佳：
    // 參見 https://pkg.go.dev/time#Time。
    type Time struct { /* 省略 */ }

    func (t Time) Add(d Duration) Time { /* 省略 */ }
    ```

* 如有疑問，使用指針接收器。

作為一般指導原則，傾向於使一個類型的方法要麼全部是指針方法，要麼全部是值方法。

**注意：** 關於將值或指針傳遞給函數是否會影響性能有很多誤解。編譯器可以選擇將值的指針傳遞到棧上，也可以在棧上複製值，但在大多數情況下，這些考慮不應該超過代碼的可讀性和正確性。當性能確實重要時，重要的是在決定哪種方法性能更好之前，使用現實的基準測試對兩種方法進行分析。

[普通舊數據]: https://en.wikipedia.org/wiki/Passive_data_structure
[`sync.Mutex`]: https://pkg.go.dev/sync#Mutex
[內建類型]: https://pkg.go.dev/builtin

<a id="switch-break"></a>

### `switch` 與 `break`

<a id="TOC-SwitchBreak"></a>

不要在 `switch` 語句的末尾使用沒有目標標籤的 `break` 語句；它們是多餘的。與 C 和 Java 不同，Go 中的 `switch` 語句會自動中斷，需要 `fallthrough` 語句來實現 C 風格的行為。如果你想澄清一個空語句的目的，請使用註釋而不是 `break`。

```go
// 較佳：
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
// 不佳：
switch x {
case "A", "B":
    buf.WriteString(x)
    break // 這個 break 是多餘的
case "C":
    break // 這個 break 是多餘的
default:
    return fmt.Errorf("unknown value: %q", x)
}
```

> **注意：** 如果 `switch` 語句在一個 `for` 循環內，那麼在 `switch` 內使用 `break` 不會退出包圍的 `for` 循環。
>
> ```go
> for {
>   switch x {
>   case "A":
>      break // 退出 switch，不是循環
>   }
> }
> ```
>
> 要跳出包圍的循環，請在 `for` 語句上使用標籤：
>
> ```go
> loop:
>   for {
>     switch x {
>     case "A":
>        break loop // 退出循環
>     }
>   }
> ```

<a id="synchronous-functions"></a>

### 同步函數 Synchronous functions

<a id="TOC-SynchronousFunctions"></a>

同步函數直接返回它們的結果，並在返回之前完成任何回調或通道操作。相比於異步函數，更推薦使用同步函數。

同步函數將 goroutine 局限在調用內。這有助於推理它們的生命週期，並避免泄漏和數據競爭。同步函數也更容易測試，因為調用者可以傳遞輸入並檢查輸出，無需輪詢或同步。

如果必要，調用者可以通過在單獨的 goroutine 中調用函數來添加並發性。然而，在調用方側移除不必要的並發性有時是相當困難（有時甚至是不可能的）。

另見：

* Bryan Mills 的演講 "Rethinking Classical Concurrency Patterns"：[幻燈片][rethinking-slides]，[視頻][rethinking-video]

<a id="type-aliases"></a>

### 類型別名 Type aliases

<a id="TOC-TypeAliases"></a>

使用*類型定義* `type T1 T2` 來定義一個新類型。使用[*類型別名*] `type T1 = T2` 來引用一個現有的類型，而不定義一個新類型。類型別名很少見；它們的主要用途是幫助將包遷移到新的源代碼位置。當不需要時，不要使用類型別名。

[*類型別名*]: http://golang.org/ref/spec#Type_declarations

<a id="use-percent-q"></a>

### 使用 %q

<a id="TOC-UsePercentQ"></a>

Go 的格式化函數（`fmt.Printf` 等）有一個 `%q` 動詞，它會在雙引號內打印字符串。

```go
// 較佳：
fmt.Printf("value %q looks like English text", someText)
```

優先使用 `%q` 而不是手動進行等效操作，使用 `%s`：

```go
// 不佳：
fmt.Printf("value \"%s\" looks like English text", someText)
// 也避免手動用單引號包圍字符串：
fmt.Printf("value '%s' looks like English text", someText)
```

在為人類準備的輸出中，當輸入值可能為空或包含控制字符時，推薦使用 `%q`。一個無聲的空字符串可能很難被注意到，但 `""` 清楚地突出顯示為此。

<a id="use-any"></a>

### 使用 any

Go 1.18 引入了 `any` 類型作為 `interface{}` 的[別名]。因為它是一個別名，`any` 在許多情況下等同於 `interface{}`，在其他情況下可以通過顯式轉換輕鬆互換。在新代碼中優先使用 `any`。

[別名]: https://go.googlesource.com/proposal/+/master/design/18130-type-alias.md

## 常用庫 Common libraries

<a id="flags"></a>

### 標誌 Flags

<a id="TOC-Flags"></a>

Google 代碼庫中的 Go 程序使用 [標準 `flag` 包] 的內部變體。它具有類似的接口，但與內部 Google 系統良好互操作。Go 二進制文件中的標誌名稱應該優先使用下劃線來分隔單詞，儘管持有標誌值的變量應該遵循標準 Go 名稱風格（[混合大小寫]）。具體來說，標誌名稱應該是蛇形大小寫，變量名稱懲罰是駝峰大小寫的等效名稱。

```go
// 較佳：
var (
    pollInterval = flag.Duration("poll_interval", time.Minute, "Interval to use for polling.")
)
```

```go
// 不佳：
var (
    poll_interval = flag.Int("pollIntervalSeconds", 60, "Interval to use for polling in seconds.")
)
```

標誌只能在 `package main` 或等效包中定義。

通用包應該使用 Go API 進行配置，而不是穿透到命令行界面；不要讓導入一個庫作為副作用導出新的標誌。也就是說，優先考慮顯式函數參數或結構體字段賦值，或者在最嚴格的審查下不太頻繁地導出全局變量。在極少數必要打破此規則的情況下，標誌名稱必須清楚地指示它配置的包。

如果你的標誌是全局變量，請將它們放在自己的 `var` 組中，跟在導入部分之後。

有關創建[複雜 CLI]與子命令的最佳實踐的額外討論。

另見：

* [每週提示 #45: 避免標誌，特別是在庫代碼中][totw-45]
* [Go 提示 #10: 配置結構體和標誌](https://google.github.io/styleguide/go/index.html#gotip)
* [Go 提示 #80: 依賴注入原則](https://google.github.io/styleguide/go/index.html#gotip)

[標準 `flag` 包]: https://golang.org/pkg/flag/
[混合大小寫]: guide#mixed-caps
[複雜 CLI]: best-practices#complex-clis
[totw-45]: https://abseil.io/tips/45

<a id="logging"></a>

### 日誌記錄

Google 代碼庫中的 Go 程序使用標準 [`log`] 包的一個變體。它具有類似但更強大的接口，並且與內部 Google 系統良好互操作。這個庫的開源版本可作為 [package `glog`] 獲得，開源的 Google 項目可以使用它，但本指南始終將其稱為 `log`。

**注意：** 對於異常程序退出，這個庫使用 `log.Fatal` 來中止並帶有堆棧跟蹤，使用 `log.Exit` 來停止但不帶堆棧跟蹤。標準庫中的 `log.Panic` 函數在這裡不存在。

**提示：** `log.Info(v)` 等同於 `log.Infof("%v", v)`，其他日誌級別也是如此。當你沒有格式化要做時，優先使用非格式化版本。

另見：

* 關於[錯誤日誌記錄](best-practices#error-logging)和[自定義詳細級別](best-practices#vlog)的最佳實踐
* 何時以及如何使用 log 包來[停止程序](best-practices#checks-and-panics)

[`log`]: https://pkg.go.dev/log
[`log/slog`]: https://pkg.go.dev/log/slog
[package `glog`]: https://pkg.go.dev/github.com/golang/glog
[`log.Exit`]: https://pkg.go.dev/github.com/golang/glog#Exit
[`log.Fatal`]: https://pkg.go.dev/github.com/golang/glog#Fatal

<a id="contexts"></a>

### 上下文 Contexts

<a id="TOC-Contexts"></a>

[`context.Context`] 類型的值在 API 和進程邊界之間傳遞安全憑證、追蹤信息、截止時間和取消信號。與在 Google 代碼庫中使用線程局部存儲的 C++ 和 Java 不同，Go 程序明確地沿著從接收 RPC 和 HTTP 請求到發出請求的整個函數調用鏈傳遞上下文。

[`context.Context`]: https://pkg.go.dev/context

當傳遞給函數或方法時，`context.Context` 總是第一個參數。

```go
func F(ctx context.Context /* other arguments */) {}
```

例外情況有：

* 在 HTTP 處理器中，上下文來自 [`req.Context()`](https://pkg.go.dev/net/http#Request.Context)。
* 在流式 RPC 方法中，上下文來自流。

    使用 gRPC 流的代碼從生成的服務器類型中的 `Context()` 方法訪問上下文，該方法實現了 `grpc.ServerStream`。參見 [gRPC 生成代碼文檔](https://grpc.io/docs/languages/go/generated-code/)。

* 在入口點函數中（見下面的例子），使用 [`context.Background()`](https://pkg.go.dev/context/#Background)。

    * 在二進制目標中：`main`
    * 在通用代碼和庫中：`init`
    * 在測試中：`TestXXX`、`BenchmarkXXX`、`FuzzXXX`

> **注意**：在調用鏈中間的代碼很少需要使用 `context.Background()` 創建自己的基礎上下文。總是優先從你的調用者那裡獲取上下文，除非它是錯誤的上下文。
>
> 你可能會遇到服務器庫（Stubby、gRPC 或 Google 的 Go 服務器框架中的 HTTP 的實現），它們會為每個請求構造一個新的上下文對象。這些上下文立即填充了來自傳入請求的信息，因此當傳遞給請求處理器時，上下文的附加值已經跨網絡邊界從客戶端調用者傳播到它。此外，這些上下文的生命週期限於請求的生命週期：當請求完成時，上下文被取消。
>
> 除非你正在實現一個服務器框架，否則你不應該在庫代碼中使用 `context.Background()` 創建上下文。相反，如果有現有的上下文可用，優先使用下面提到的上下文分離。如果你認為在入口點函數之外需要 `context.Background()`，在承諾實施之前請與 Google Go 風格郵件列表討論。

`context.Context` 在函數中排在第一位的慣例也適用於測試輔助函數。

```go
// 較佳：
func readTestFile(ctx context.Context, t *testing.T, path string) string {}
```

不要將上下文成員添加到結構體類型中。相反，為類型上需要傳遞它的每個方法添加一個上下文參數。唯一的例外是對於其簽名必須與標準庫或 Google 控制之外的第三方庫中的接口匹配的方法。這種情況非常罕見，在實施和可讀性審查之前應與 Google Go 風格郵件列表討論。

Google 代碼庫中必須啟動可以在父上下文被取消後運行的後台操作的代碼可以使用內部包進行分離。關注 [問題 #40221](https://github.com/golang/go/issues/40221) 以獲取開源替代方案的討論。

由於上下文是不可變的，將相同的上下文傳遞給共享相同截止時間、取消信號、憑證、父追蹤等的多個調用是可以的。

另見：

* [上下文和結構體]

[上下文和結構體]: https://go.dev/blog/context-and-structs

<a id="custom-contexts"></a>

#### 自定義上下文 Custom contexts

不要創建自定義上下文類型或在函數簽名中使用除 `context.Context` 以外的接口。這條規則沒有例外。

想像如果每個團隊都有一個自定義上下文。從包 `p` 到包 `q` 的每個函數調用都必須確定如何將 `p.Context` 轉換為 `q.Context`，對於所有的包 `p` 和 `q` 組合。這對人類來說是不切實際和容易出錯的，它使得添加上下文參數的自動重構幾乎不可能。

如果你有應用數據要傳遞，將它放在參數中、接收器中、全局變量中，或者如果它真的屬於那裡，則放在 `Context` 值中。創建自己的上下文類型是不可接受的，因為它破壞了 Go 團隊使 Go 程序在生產中正常工作的能力。

<a id="crypto-rand"></a>

### crypto/rand

<a id="TOC-CryptoRand"></a>

不要使用包 `math/rand` 來生成密鑰，即使是臨時的也不行。如果未種子化，生成器是完全可預測的。使用 `time.Nanoseconds()` 作為種子，只有幾位的熵。相反，使用 `crypto/rand` 的 Reader，如果你需要文本，輸出為十六進制或 base64。

```go
// 較佳：
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

**注意：** `log.Fatalf` 不是標準庫的 log。參見 [#logging]。

<a id="useful-test-failures"></a>

## 有用的測試失敗信息

<a id="TOC-UsefulTestFailures"></a>

在不閱讀測試源碼的情況下，應該能夠診斷出測試的失敗原因。測試應該提供有幫助的消息來詳細說明：

* 導致失敗的原因
* 什麼輸入導致了錯誤
* 實際結果
* 預期的結果

下面概述了實現此目標的具體慣例。

<a id="assert"></a>

### 斷言庫 Assertion libraries

<a id="TOC-Assert"></a>

不要創建“斷言庫”作為測試的輔助工具。

斷言庫是試圖在測試中結合驗證和產生失敗消息的庫（儘管相同的陷阱也可能適用於其他測試輔助工具）。有關測試輔助工具和斷言庫之間區別的更多信息，請參見[最佳實踐](best-practices#test-functions)。

```go
// 不佳：
var obj BlogPost

assert.IsNotNil(t, "obj", obj)
assert.StringEq(t, "obj.Type", obj.Type, "blogPost")
assert.IntEq(t, "obj.Comments", obj.Comments, 2)
assert.StringNotEq(t, "obj.Body", obj.Body, "")
```

斷言庫傾向於要麼提前停止測試（如果 `assert` 調用了 `t.Fatalf` 或 `panic`），要麼省略了關於測試正確部分的相關信息：

```go
// 不佳：
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

複雜的斷言函數通常不提供[有用的失敗消息]和存在於測試函數內的上下文。過多的斷言函數和庫導致了開發者體驗的碎片化：我應該使用哪個斷言庫，它應該輸出什麼樣的輸出格式等等？碎片化產生了不必要的混亂，特別是對於庫維護者和大規模變更的作者，他們負責修復潛在的下游破壞。不要創建一個特定於領域的測試語言，而應該使用 Go 本身。

斷言庫經常將比較和相等檢查分離出來。優先使用標準庫，如 [`cmp`] 和 [`fmt`]：

```go
// 較佳：
var got BlogPost

want := BlogPost{
    Comments: 2,
    Body:     "Hello, world!",
}

if !cmp.Equal(got, want) {
    t.Errorf("Blog post = %v, want = %v", got, want)
}
```

對於更具領域特定的比較輔助工具，優先返回一個值或錯誤，這可以在測試的失敗消息中使用，而不是傳遞 `*testing.T` 並調用其錯誤報告方法：

```go
// 較佳：
func postLength(p BlogPost) int { return len(p.Body) }

func TestBlogPost_VeritableRant(t *testing.T) {
    post := BlogPost{Body: "I am Gunnery Sergeant Hartman, your senior drill instructor."}

    if got, want := postLength(post), 60; got != want {
        t.Errorf("Length of post = %v, want %v", got, want)
    }
}
```

**最佳實踐：** 如果 `postLength` 不是微不足道的，直接對其進行測試是有意義的，獨立於使用它的任何測試。

另見：

* [相等性比較和差異](#types-of-equality)
* [打印差異](#print-diffs)
* 有關測試輔助工具和斷言輔助工具之間區別的更多信息，請參見[最佳實踐](best-practices#test-functions)

[有用的失敗消息]: #useful-test-failures
[`fmt`]: https://golang.org/pkg/fmt/
[標記測試輔助工具]: #mark-test-helpers

<a id="identify-the-function"></a>

### 指明函數 Identify the function

在大多數測試中，失敗消息應該包括失敗的函數名稱，即使從測試函數的名稱看似顯而易見。具體來說，你的失敗消息應該是 `YourFunc(%v) = %v, want %v` 而不僅僅是 `got %v, want %v`。

<a id="identify-the-input"></a>

### 指明輸入 Identify the input

在大多數測試中，如果輸入短小，失敗消息應該包括函數輸入。如果輸入的相關屬性不明顯（例如，因為輸入很大或不透明），你應該用正在測試的內容描述來命名你的測試用例，並將描述作為錯誤消息的一部分打印出來。

<a id="got-before-want"></a>

### 先得到再期望 Got before want

測試輸出應該包括函數返回的實際值，然後再打印期望的值。打印測試輸出的標準格式是 `YourFunc(%v) = %v, want %v`。在你會寫“實際”和“期望”的地方，優先使用“got”和“want”。

對於差異，方向性不那麼明顯，因此包括一個鍵以幫助解釋失敗是很重要的。參見[打印差異的部分]。無論你在失敗消息中使用哪種差異順序，你應該明確地指出它，因為現有代碼對於排序是不一致的。

[打印差異的部分]: #print-diffs

<a id="compare-full-structures"></a>

### 完整結構比較 Full structure comparisons


如果你的函數返回一個結構體（或任何具有多個字段的數據類型，如切片、數組和映射），避免編寫進行手工逐字段比較結構體的測試代碼。相反，構造你期望你的函數返回的數據，並直接使用[深度比較]進行比較。

**注意：** 如果你的數據包含不相關的字段，這些字段會掩蓋測試的意圖，則此建議不適用。

如果你的結構體需要進行大致相等（或等價類型的語義）比較，或者它包含無法進行相等比較的字段（例如，如果其中一個字段是 `io.Reader`），使用 [`cmp.Diff`] 或 [`cmp.Equal`] 比較並配合 [`cmpopts`] 選項，如 [`cmpopts.IgnoreInterfaces`]，可能滿足你的需求（[示例](https://play.golang.org/p/vrCUNVfxsvF)）。

如果你的函數返回多個返回值，你不需要在比較它們之前將這些值包裝在一個結構體中。只需單獨比較返回值並打印它們。

```go
// 較佳：
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

[深度比較]: #types-of-equality
[`cmpopts`]: https://pkg.go.dev/github.com/google/go-cmp/cmp/cmpopts
[`cmpopts.IgnoreInterfaces`]: https://pkg.go.dev/github.com/google/go-cmp/cmp/cmpopts#IgnoreInterfaces

<a id="compare-stable-results"></a>

### 比較穩定的結果 Compare stable results

避免比較可能依賴於你不擁有的套件輸出穩定性的結果。相反，測試應該比較在語義上相關的資訊，這些資訊是穩定的且能抵抗依賴性的變化。對於返回格式化字符串或序列化字節的功能，通常不應假設輸出是穩定的。

例如，[`json.Marshal`] 可以改變（並且在過去已經改變過）它發出的特定字節。如果 `json` 套件改變了它序列化字節的方式，那麼對 JSON 字符串進行字符串等值比較的測試可能會失敗。相反，一個更穩健的測試將解析 JSON 字符串的內容，並確保它在語義上等同於某些預期的數據結構。

[`json.Marshal`]: https://golang.org/pkg/encoding/json/#Marshal

<a id="keep-going"></a>

### 繼續進行 Keep going

即使在失敗之後，測試也應該盡可能地繼續進行，以便在單次運行中打印出所有失敗的檢查。這樣，正在修復失敗測試的開發者就不必在修復每個錯誤後重新運行測試來找到下一個錯誤。

對於報告不匹配，優先調用 `t.Error` 而不是 `t.Fatal`。當比較一個函數輸出的幾個不同屬性時，對於這些比較中的每一個都使用 `t.Error`。

調用 `t.Fatal` 主要用於報告意外的錯誤條件，當後續的比較失敗不再有意義時。

對於表驅動測試，考慮使用子測試並使用 `t.Fatal` 而不是 `t.Error` 和 `continue`。另見
[GoTip #25: Subtests: Making Your Tests Lean](https://google.github.io/styleguide/go/index.html#gotip)。

**最佳實踐：** 關於何時應該使用 `t.Fatal` 的更多討論，請參見
[最佳實踐](best-practices#t-fatal)。

<a id="types-of-equality"></a>

### 等值比較和差異 Equality comparison and diffs

`==` 運算子使用[語言定義的比較]來評估等值。標量值（數字、布林值等）基於它們的值進行比較，但只有某些結構體和介面可以以這種方式比較。指針基於它們是否指向相同的變數進行比較，而不是基於它們指向的值的等值。

[`cmp`] 套件可以比較 `==` 無法適當處理的更複雜的數據結構，如切片。使用 [`cmp.Equal`] 進行等值比較和 [`cmp.Diff`] 獲取對象之間的人類可讀差異。

```go
// 較佳：
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

作為通用比較庫，`cmp` 可能不知道如何比較某些類型。例如，它只能在傳遞了 [`protocmp.Transform`] 選項的情況下比較協議緩衝消息。

<!-- 這裡 want 和 got 的順序是故意的。參見 #print-diffs 中的評論。 -->

```go
// 較佳：
if diff := cmp.Diff(want, got, protocmp.Transform()); diff != "" {
    t.Errorf("Foo() returned unexpected difference in protobuf messages (-want +got):\n%s", diff)
}
```

儘管 `cmp` 套件不是 Go 標準庫的一部分，但它由 Go 團隊維護，應該隨著時間產生穩定的等值結果。
它是用戶可配置的，應該滿足大多數比較需求。

[語言定義的比較]: http://golang.org/ref/spec#Comparison_operators
[`cmp`]: https://pkg.go.dev/github.com/google/go-cmp/cmp
[`cmp.Equal`]: https://pkg.go.dev/github.com/google/go-cmp/cmp#Equal
[`cmp.Diff`]: https://pkg.go.dev/github.com/google/go-cmp/cmp#Diff
[`protocmp.Transform`]: https://pkg.go.dev/google.golang.org/protobuf/testing/protocmp#Transform

現有代碼可能使用以下較舊的庫，並可能繼續使用它們以保持一致性：

* [`pretty`] 產生美觀的差異報告。然而，它相當故意地將視覺表示相同的值視為等值。特別是，`pretty` 不會捕捉到 nil 切片和空切片之間的差異，對於具有相同字段的不同介面實現不敏感，並且可以使用嵌套映射作為與結構體值比較的基礎。它還在產生差異之前將整個值序列化為字符串，因此不適合比較大值。默認情況下，它比較未導出的字段，這使它對依賴性中的實現細節的變化很敏感。因此，不適合在 protobuf 消息上使用 `pretty`。

[`pretty`]: https://pkg.go.dev/github.com/kylelemons/godebug/pretty

優先為新代碼使用 `cmp`，並且值得考慮在實際可行的情況下更新舊代碼以使用 `cmp`。

較舊的代碼可能使用標準庫 `reflect.DeepEqual` 函數來比較複雜結構。不應使用 `reflect.DeepEqual` 進行等值檢查，因為它對未導出字段和其他實現細節的變化敏感。使用 `reflect.DeepEqual` 的代碼應該更新為上述庫之一。

**注意：** `cmp` 套件是為測試而設計的，而不是生產使用。因此，當它懷疑比較執行不正確時，它可能會恐慌，以向用戶提供指導，如何改進測試以使其不那麼脆弱。鑑於 cmp 傾向於恐慌，這使它不適合在生產中使用的代碼，因為偶發的恐慌可能是致命的。

<a id="level-of-detail"></a>

### 細節層次 Level of detail

傳統的失敗訊息，適用於大多數Go測試，是
`YourFunc(%v) = %v, want %v`。然而，在某些情況下，可能需要更多或更少的細節：

* 執行複雜互動的測試應該描述這些互動。例如，如果同一個`YourFunc`被多次呼叫，應該識別哪次呼叫導致測試失敗。如果系統的額外狀態很重要，應該在失敗輸出中包含這些資訊（或至少在日誌中）。
* 如果數據是一個包含大量樣板的複雜結構，只描述訊息中重要的部分是可以接受的，但不要過度隱藏數據。
* 設置失敗不需要同樣的細節層次。如果一個測試助手填充了一個Spanner表，但Spanner當機了，你可能不需要包含你打算儲存在數據庫中的哪個測試輸入。通常`t.Fatalf("Setup: Failed to set up test database: %s", err)`就足夠有幫助來解決問題。

**提示：** 在開發過程中觸發你的失敗模式。審視失敗訊息的樣子，以及維護者是否能有效處理失敗。

以下是一些清晰重現測試輸入和輸出的技巧：

* 在打印字符串數據時，[`%q`經常很有用](#use-percent-q)來強調值的重要性，並更容易發現錯誤的值。
* 在打印（小）結構時，`%+v`可能比`%v`更有用。
* 當大值的驗證失敗時，[打印差異](#print-diffs)可以使理解失敗變得更容易。

<a id="print-diffs"></a>

### 打印差異 Print diffs

如果你的函數返回大量輸出，當你的測試失敗時，閱讀失敗訊息的人可能很難找到差異。不要打印返回值和期望值，而是製作一個差異比較。

為了計算這些值的差異，首選使用`cmp.Diff`，特別是對於新的測試和新的代碼，但也可以使用其他工具。有關每個函數的優缺點，請參見[等值類型]的指南。

* [`cmp.Diff`]

* [`pretty.Compare`]

你可以使用[`diff`]包來比較多行字符串或字符串列表。你可以將其作為其他類型差異的構建塊。

[等值類型]: #types-of-equality
[`diff`]: https://pkg.go.dev/github.com/kylelemons/godebug/diff
[`pretty.Compare`]: https://pkg.go.dev/github.com/kylelemons/godebug/pretty#Compare

在你的失敗訊息中添加一些文本，解釋差異的方向。

<!--
這些例子中期望和實際值的順序顛倒是有意的，因為這是Google代碼庫中普遍的順序。關於使用哪個順序也沒有明確立場，因為沒有共識哪個是“最易讀的。”
-->

* 當你使用`cmp`、`pretty`和`diff`包時（如果你將`(want, got)`傳遞給函數），像`diff (-want +got)`這樣的東西是好的，因為你添加到格式字符串的`-`和`+`將與實際出現在差異行開頭的`-`和`+`匹配。如果你將`(got, want)`傳遞給你的函數，正確的鍵將是`(-got +want)`。

* `messagediff`包使用不同的輸出格式，所以當你使用它時（如果你將`(want, got)`傳遞給函數），`diff (want -> got)`訊息是合適的，因為箭頭的方向將與“修改”行中的箭頭方向匹配。

差異將跨越多行，所以你應該在打印差異之前打印一個換行符。

<a id="test-error-semantics"></a>

### 測試錯誤語義 Test error semantics

當單元測試進行字符串比較或使用普通的`cmp`來檢查特定輸入是否返回特定類型的錯誤時，如果將來這些錯誤訊息被改寫，你可能會發現你的測試很脆弱。由於這可能將你的單元測試變成變更檢測器（參見[ToTT: 考慮有害的變更檢測器測試][tott-350]），不要使用字符串比較來檢查你的函數返回了哪種類型的錯誤。然而，允許使用字符串比較來檢查來自被測試包的錯誤訊息是否滿足某些特性，例如，它是否包含了參數名稱。

Go中的錯誤值通常有一部分是為人眼而設計的，一部分是為了語義控制流程。測試應該只檢查可以可靠觀察到的語義信息，而不是為人類調試而設計的顯示信息，因為這經常會受到未來變化的影響。有關構建具有語義意義的錯誤的指南，請參見[關於錯誤的最佳實踐](best-practices#error-handling)。如果來自你無法控制的依賴項的錯誤缺乏語義信息，考慮對所有者提出錯誤報告以幫助改進API，而不是依賴於解析錯誤訊息。

在單元測試中，通常只關心是否發生了錯誤。如果是這樣，那麼當你期望出現錯誤時，只測試錯誤是否非空就足夠了。如果你想測試錯誤在語義上是否與某些其他錯誤匹配，那麼考慮使用[`errors.Is`]或`cmp`與[`cmpopts.EquateErrors`]。

> **注意：** 如果一個測試使用了[`cmpopts.EquateErrors`]，但它的所有`wantErr`值要么是`nil`要么是`cmpopts.AnyError`，那麼使用`cmp`是[不必要的機制](guide#least-mechanism)。通過將want字段簡化為`bool`來簡化代碼。然後你可以使用一個簡單的比較與`!=`。
>
> ```go
> // 較佳：
> err := f(test.input)
> gotErr := err != nil
> if gotErr != test.wantErr {
>     t.Errorf("f(%q) = %v, want error presence = %v", test.input, err, test.wantErr)
> }
> ```

另見
[GoTip #13: 設計錯誤以便檢查](https://google.github.io/styleguide/go/index.html#gotip)。

[tott-350]: https://testing.googleblog.com/2015/01/testing-on-toilet-change-detector-tests.html
[`cmpopts.EquateErrors`]: https://pkg.go.dev/github.com/google/go-cmp/cmp/cmpopts#EquateErrors
[`errors.Is`]: https://pkg.go.dev/errors#Is

<a id="test-structure"></a>

## Test structure

<a id="subtests"></a>

### 子測試 Subtests

標準Go測試庫提供了一種[定義子測試]的功能。這允許在設置和清理、控制並行性以及測試過濾方面提供靈活性。子測試可能很有用（特別是對於表驅動測試），但使用它們並非強制性的。另見
[Go博客關於子測試的文章](https://blog.golang.org/subtests)。

子測試不應該依賴其他案例的執行來確保成功或初始狀態，因為預期子測試能夠單獨運行，使用`go test -run`標誌或Bazel [測試過濾]表達式。

[定義子測試]: https://pkg.go.dev/testing#hdr-Subtests_and_Sub_benchmarks
[測試過濾]: https://bazel.build/docs/user-manual#test-filter

<a id="subtest-names"></a>

#### 子測試名稱 Subtest names

命名你的子測試，使其在測試輸出中可讀並對測試過濾的用戶在命令行上有用。當你使用`t.Run`創建子測試時，第一個參數被用作測試的描述性名稱。為了確保測試結果對閱讀日誌的人類來說是易讀的，選擇在轉義後仍然有用且可讀的子測試名稱。將子測試名稱視為函數標識符，而不是散文描述。測試運行器會將空格替換為下劃線，並轉義非打印字符。如果你的測試數據受益於更長的描述，請考慮將描述放在一個單獨的字段中（也許是使用`t.Log`打印或與失敗訊息一起）。

子測試可以使用[Go測試運行器]或Bazel [測試過濾]的標誌單獨運行，因此選擇描述性名稱，同時也易於輸入。

> **警告：** 在子測試名稱中使用斜線字符特別不友好，因為它們在測試過濾中有[特殊含義]。
>
> > ```sh
> > # 不佳：
> > # 假設TestTime和t.Run("America/New_York", ...)
> > bazel test :mytest --test_filter="Time/New_York"    # 什麼都不運行！
> > bazel test :mytest --test_filter="Time//New_York"   # 正確，但尷尬。
> > ```

為了[識別函數的輸入]，將它們包含在測試的失敗訊息中，在那裡它們不會被測試運行器轉義。

```go
// 較佳：
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

以下是一些需要避免的例子：

```go
// 不佳：
// Too wordy.
t.Run("check that there is no mention of scratched records or hovercrafts", ...)
// Slashes cause problems on the command line.
t.Run("AM/PM confusion", ...)
```

另見
[Go 提示 #117：子測試名稱](https://google.github.io/styleguide/go/index.html#gotip)。

[Go 測試運行器]: https://golang.org/cmd/go/#hdr-Testing_flags
[識別輸入]: #identify-the-input
[測試過濾的特殊含義]: https://blog.golang.org/subtests#:~:text=Perhaps%20a%20bit,match%20any%20tests

<a id="table-driven-tests"></a>

### 表驅動測試 Table-driven tests

當許多不同的測試案例可以使用類似的測試邏輯進行測試時，請使用表驅動測試。

* 當測試函數的實際輸出是否等於預期輸出時。例如，許多[`fmt.Sprintf`的測試]或下面的最小片段。
* 當測試函數的輸出始終符合同一組不變量時。例如，[`net.Dial`的測試]。

[`fmt.Sprintf`的測試]: https://cs.opensource.google/go/go/+/master:src/fmt/fmt_test.go
[`net.Dial`的測試]: https://cs.opensource.google/go/go/+/master:src/net/dial_test.go;l=318;drc=5b606a9d2b7649532fe25794fa6b99bd24e7697c

以下是表驅動測試的最小結構。如果需要，您可以使用不同的名稱或添加額外的設施，如子測試或設置和清理函數。始終記住[有用的測試失敗](#useful-test-failures)。

```go
// 較佳：
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

**注意**：上面例子中的失敗訊息滿足了[識別函數](#identify-the-function)和[識別輸入](#identify-the-input)的指導。沒有必要[數字識別行](#table-tests-identifying-the-row)

當某些測試案例需要使用與其他測試案例不同的邏輯進行檢查時，寫多個測試函數更為合適，如[GoTip #50: 不相關的表測試]中所解釋的。當表中的每個條目都有自己不同的條件邏輯來檢查其輸入的每個輸出時，你的測試代碼的邏輯可能會變得難以理解。如果測試案例有不同的邏輯但相同的設置，單個測試函數中的一系列[子測試](#subtests)可能是有意義的。

您可以將表驅動測試與多個測試函數結合使用。例如，當測試函數的輸出完全匹配預期輸出，並且函數對無效輸入返回非空錯誤時，則編寫兩個單獨的表驅動測試函數是最好的方法：一個用於正常的非錯誤輸出，一個用於錯誤輸出。

[GoTip #50: 不相關的表測試]: https://google.github.io/styleguide/go/index.html#gotip

<a id="table-tests-data-driven"></a>

#### 數據驅動的測試案例 Data-driven test cases

表測試的行有時可能變得複雜，行值在測試案例內指定條件行為。從測試案例之間的重複中獲得的額外清晰度對於可讀性是必要的。

```go
// 較佳：
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

在下面的反例中，請注意在案例設置中很難區分每個測試案例使用的`Codex`類型。（突出顯示的部分違反了[ToTT: 數據驅動陷阱！][tott-97]的建議。）

```go
// 不佳：
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

#### 識別行 Identifying the row

不要使用測試表中測試的索引作為命名測試或打印輸入的替代品。沒有人想要通過你的測試表並計算條目數量來弄清楚哪個測試案例失敗了。

```go
// 不佳：
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

不要使用測試表中測試的索引作為命名測試或打印輸入的替代品。沒有人想要通過你的測試表並計算條目數量來弄清楚哪個測試案例失敗了。

在你的測試結構中添加一個測試描述，並在失敗訊息中打印它。使用子測試時，你的子測試名稱應該有效地識別行。

**重要：** 即使`t.Run`限定了輸出和執行，你必須始終[識別輸入]。表測試行名稱必須遵循[子測試命名]指南。

[識別輸入]: #identify-the-input
[子測試命名]: #subtest-names

<a id="mark-test-helpers"></a>

### 測試助手 Test helpers

測試助手是執行設置或清理任務的函數。在測試助手中發生的所有失敗都預期是環境的失敗（不是被測試代碼的失敗）——例如，當無法啟動測試數據庫，因為這台機器上沒有更多的空閒端口時。

如果你傳遞了一個`*testing.T`，調用[`t.Helper`]以將測試助手中的失敗歸因於調用助手的行。如果存在，這個參數應該在[上下文](#contexts)參數之後，以及在任何剩餘參數之前。

```go
// 較佳：
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

當這種模式模糊了測試失敗與導致它的條件之間的聯繫時，不要使用它。具體來說，關於[斷言庫](#assert)的指導仍然適用，[`t.Helper`]不應該用來實現這樣的庫。

**提示：** 有關測試助手和斷言助手之間區別的更多信息，請參見[最佳實踐](best-practices#test-functions)。

雖然上面提到了`*testing.T`，但對於基準測試和模糊測試助手，大部分建議仍然相同。

[`t.Helper`]: https://pkg.go.dev/testing#T.Helper

<a id="test-package"></a>

### Test package

<a id="TOC-TestPackage"></a>

<a id="test-same-package"></a>

#### 同一套件中的測試 Tests in the same package

測試可以定義在與被測試代碼相同的套件中。

要在同一套件中寫測試：

* 將測試放在`foo_test.go`文件中
* 對測試文件使用`package foo`
* 不要明確導入要測試的套件

```build
# 較佳：
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

同一套件中的測試可以訪問套件中未導出的標識符。這可能使得測試覆蓋率更好並且測試更簡潔。請注意，測試中聲明的任何[範例]都不會有用戶在其代碼中需要的套件名稱。

[`library`]: https://github.com/bazelbuild/rules_go/blob/master/docs/go/core/rules.md#go_library
[範例]: #examples

<a id="test-different-package"></a>

#### 不同套件中的測試 Tests in a different package

並不總是適合或甚至可能在與被測試代碼相同的套件中定義測試。在這些情況下，使用帶有`_test`後綴的套件名稱。這是對[套件名稱](#package-names)中“無下劃線”規則的一個例外。例如：

* 如果集成測試沒有一個明顯屬於它的套件

    ```go
    // 好的範例：
    package gmailintegration_test

    import "testing"
    ```

* 如果在同一套件中定義測試會導致循環依賴

    ```go
    // 好的範例：
    package fireworks_test

    import (
      "fireworks"
      "fireworkstestutil" // fireworkstestutil 也導入了 fireworks
    )
    ```

<a id="use-package-testing"></a>

### 使用 `testing` 套件

Go標準庫提供了[`testing`套件]。這是Google代碼庫中Go代碼唯一允許使用的測試框架。特別是，不允許使用[斷言庫](#assert)和第三方測試框架。

`testing`套件為編寫良好的測試提供了一套最小但完整的功能：

* 頂層測試
* 基準測試
* [可運行範例](https://blog.golang.org/examples)
* 子測試
* 日誌記錄
* 失敗和致命失敗

這些旨在與核心語言特性如[複合字面量]和[帶初始化器的if語句]語法協同工作，使測試作者能夠編寫[清晰、可讀和可維護的測試]。

[`testing`套件]: https://pkg.go.dev/testing
[複合字面量]: https://go.dev/ref/spec#Composite_literals
[帶初始化器的if語句]: https://go.dev/ref/spec#If_statements

<a id="non-decisions"></a>

## 無法決策 Non-decisions

A style guide cannot enumerate positive prescriptions for all matters, nor can
it enumerate all matters about which it does not offer an opinion. That said,
here are a few things where the readability community has previously debated and
has not achieved consensus about.

* **使用零值初始化局部變量**。`var i int`和`i := 0`是等價的。另見[初始化最佳實踐]。
* **空的複合字面量與`new`或`make`的對比**。`&File{}`和`new(File)`是等價的。`map[string]bool{}`和`make(map[string]bool)`也是如此。另見[複合聲明最佳實踐]。
* **在cmp.Diff調用中got, want參數排序**。保持局部一致性，
    並在您的失敗訊息中[包含一個圖例](#print-diffs)。
* **`errors.New`與`fmt.Errorf`在非格式化字符串上的使用**。
    `errors.New("foo")`和`fmt.Errorf("foo")`可以互換使用。

如果在特殊情況下它們再次出現，可讀性導師可能會做出一個可選的評論，但通常作者可以自由選擇他們在給定情況下偏好的風格。

當然，如果風格指南未涵蓋的任何內容確實需要更多討論，
作者歡迎提問 —— 不論是在特定的審查中，還是在內部訊息板上。

[複合聲明最佳實踐]: https://google.github.io/styleguide/go/best-practices#vardeclcomposite
[初始化最佳實踐]: https://google.github.io/styleguide/go/best-practices#vardeclinitialization
