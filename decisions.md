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

#### Package vs. exported symbol name

When naming exported symbols, the name of the package is always visible outside
your package, so redundant information between the two should be reduced or
eliminated. If a package exports only one type and it is named after the package
itself, the canonical name for the constructor is `New` if one is required.

> **Examples:** Repetitive Name -> Better Name
>
> *   `widget.NewWidget` -> `widget.New`
> *   `widget.NewWidgetWithName` -> `widget.NewWithName`
> *   `db.LoadFromDatabase` -> `db.Load`
> *   `goatteleportutil.CountGoatsTeleported` -> `gtutil.CountGoatsTeleported`
>     or `goatteleport.Count`
> *   `myteampb.MyTeamMethodRequest` -> `mtpb.MyTeamMethodRequest` or
>     `myteampb.MethodRequest`

<a id="repetitive-with-type"></a>

#### Variable name vs. type

The compiler always knows the type of a variable, and in most cases it is also
clear to the reader what type a variable is by how it is used. It is only
necessary to clarify the type of a variable if its value appears twice in the
same scope.

Repetitive Name               | Better Name
----------------------------- | ----------------------
`var numUsers int`            | `var users int`
`var nameString string`       | `var name string`
`var primaryProject *Project` | `var primary *Project`

If the value appears in multiple forms, this can be clarified either with an
extra word like `raw` and `parsed` or with the underlying representation:

```go
// 好的範例:
limitStr := r.FormValue("limit")
limit, err := strconv.Atoi(limitStr)
```

```go
// 好的範例:
limitRaw := r.FormValue("limit")
limit, err := strconv.Atoi(limitRaw)
```

<a id="repetitive-in-context"></a>

#### External context vs. local names

Names that include information from their surrounding context often create extra
noise without benefit. The package name, method name, type name, function name,
import path, and even filename can all provide context that automatically
qualifies all names within.

```go
// 不好的範例:
// In package "ads/targeting/revenue/reporting"
type AdsTargetingRevenueReport struct{}

func (p *Project) ProjectName() string
```

```go
// 好的範例:
// In package "ads/targeting/revenue/reporting"
type Report struct{}

func (p *Project) Name() string
```

```go
// 不好的範例:
// In package "sqldb"
type DBConnection struct{}
```

```go
// 好的範例:
// In package "sqldb"
type Connection struct{}
```

```go
// 不好的範例:
// In package "ads/targeting"
func Process(in *pb.FooProto) *Report {
    adsTargetingID := in.GetAdsTargetingID()
}
```

```go
// 好的範例:
// In package "ads/targeting"
func Process(in *pb.FooProto) *Report {
    id := in.GetAdsTargetingID()
}
```

Repetition should generally be evaluated in the context of the user of the
symbol, rather than in isolation. For example, the following code has lots of
names that may be fine in some circumstances, but redundant in context:

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

Instead, information about names that are clear from context or usage can often
be omitted:

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

## Commentary

The conventions around commentary (which include what to comment, what style to
use, how to provide runnable examples, etc.) are intended to support the
experience of reading the documentation of a public API. See
[Effective Go](http://golang.org/doc/effective_go.html#commentary) for more
information.

The best practices document's section on [documentation conventions] discusses
this further.

**Best Practice:** Use [doc preview] during development and code review to see
whether the documentation and runnable examples are useful and are presented the
way you expect them to be.

**Tip:** Godoc uses very little special formatting; lists and code snippets
should usually be indented to avoid linewrapping. Apart from indentation,
decoration should generally be avoided.

[doc preview]: best-practices#documentation-preview
[documentation conventions]:  best-practices#documentation-conventions

<a id="comment-line-length"></a>

### Comment line length

Ensure that commentary is readable from source even on narrow screens.

When a comment gets too long, it is recommended to wrap it into multiple
single-line comments. When possible, aim for comments that will read well on an
80-column wide terminal, however this is not a hard cut-off; there is no fixed
line length limit for comments in Go. The standard library, for example, often
chooses to break a comment based on punctuation, which sometimes leaves the
individual lines closer to the 60-70 character mark.

There is plenty of existing code in which comments exceed 80 characters in
length. This guidance should not be used as a justification to change such code
as part of a readability review (see [consistency](guide#consistency)), though
teams are encouraged to opportunistically update comments to follow this
guideline as a part of other refactors. The primary goal of this guideline is to
ensure that all Go readability mentors make the same recommendation when and if
recommendations are made.

See this [post from The Go Blog on documentation] for more on commentary.

[post from The Go Blog on documentation]: https://blog.golang.org/godoc-documenting-go-code

```text
# 好的範例:
// This is a comment paragraph.
// The length of individual lines doesn't matter in Godoc;
// but the choice of wrapping makes it easy to read on narrow screens.
//
// Don't worry too much about the long URL:
// https://supercalifragilisticexpialidocious.example.com:8080/Animalia/Chordata/Mammalia/Rodentia/Geomyoidea/Geomyidae/
//
// Similarly, if you have other information that is made awkward
// by too many line breaks, use your judgment and include a long line
// if it helps rather than hinders.
```

Avoid comments that will wrap repeatedly on small screens, which is a poor
reader experience.

```text
# 不好的範例:
// This is a comment paragraph. The length of individual lines doesn't matter in
Godoc;
// but the choice of wrapping causes jagged lines on narrow screens or in code
review,
// which can be annoying, especially when in a comment block that will wrap
repeatedly.
//
// Don't worry too much about the long URL:
// https://supercalifragilisticexpialidocious.example.com:8080/Animalia/Chordata/Mammalia/Rodentia/Geomyoidea/Geomyidae/
```

<a id="doc-comments"></a>

### Doc comments

<a id="TOC-DocComments"></a>

All top-level exported names must have doc comments, as should unexported type
or function declarations with unobvious behavior or meaning. These comments
should be [full sentences] that begin with the name of the object being
described. An article ("a", "an", "the") can precede the name to make it read
more naturally.

```go
// 好的範例:
// A Request represents a request to run a command.
type Request struct { ...

// Encode writes the JSON encoding of req to w.
func Encode(w io.Writer, req *Request) { ...
```

Doc comments appear in [Godoc](https://pkg.go.dev/) and are surfaced by IDEs,
and therefore should be written for anyone using the package.

[full sentences]: #comment-sentences

A documentation comment applies to the following symbol, or the group of fields
if it appears in a struct.

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

**Best Practice:** If you have doc comments for unexported code, follow the same
custom as if it were exported (namely, starting the comment with the unexported
name). This makes it easy to export it later by simply replacing the unexported
name with the newly-exported one across both comments and code.

<a id="comment-sentences"></a>

### Comment sentences

<a id="TOC-CommentSentences"></a>

Comments that are complete sentences should be capitalized and punctuated like
standard English sentences. (As an exception, it is okay to begin a sentence
with an uncapitalized identifier name if it is otherwise clear. Such cases are
probably best done only at the beginning of a paragraph.)

Comments that are sentence fragments have no such requirements for punctuation
or capitalization.

[Documentation comments] should always be complete sentences, and as such should
always be capitalized and punctuated. Simple end-of-line comments (especially
for struct fields) can be simple phrases that assume the field name is the
subject.

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

[Documentation comments]: #doc-comments

<a id="examples"></a>

### Examples

<a id="TOC-Examples"></a>

Packages should clearly document their intended usage. Try to provide a
[runnable example]; examples show up in Godoc. Runnable examples belong in the
test file, not the production source file. See this example ([Godoc], [source]).

[runnable example]: http://blog.golang.org/examples
[Godoc]: https://pkg.go.dev/time#example-Duration
[source]: https://cs.opensource.google/go/go/+/HEAD:src/time/example_test.go

If it isn't feasible to provide a runnable example, example code can be provided
within code comments. As with other code and command-line snippets in comments,
it should follow standard formatting conventions.

<a id="named-result-parameters"></a>

### Named result parameters

<a id="TOC-NamedResultParameters"></a>

When naming parameters, consider how function signatures appear in Godoc. The
name of the function itself and the type of the result parameters are often
sufficiently clear.

```go
// 好的範例:
func (n *Node) Parent1() *Node
func (n *Node) Parent2() (*Node, error)
```

If a function returns two or more parameters of the same type, adding names can
be useful.

```go
// 好的範例:
func (n *Node) Children() (left, right *Node, err error)
```

If the caller must take action on particular result parameters, naming them can
help suggest what the action is:

```go
// 好的範例:
// WithTimeout returns a context that will be canceled no later than d duration
// from now.
//
// The caller must arrange for the returned cancel function to be called when
// the context is no longer needed to prevent a resource leak.
func WithTimeout(parent Context, d time.Duration) (ctx Context, cancel func())
```

In the code above, cancellation is a particular action a caller must take.
However, were the result parameters written as `(Context, func())` alone, it
would be unclear what is meant by "cancel function".

Don't use named result parameters when the names produce
[unnecessary repetition](#repetitive-with-type).

```go
// 不好的範例:
func (n *Node) Parent1() (node *Node)
func (n *Node) Parent2() (node *Node, err error)
```

Don't name result parameters in order to avoid declaring a variable inside the
function. This practice results in unnecessary API verbosity at the cost of
minor implementation brevity.

[Naked returns] are acceptable only in a small function. Once it's a
medium-sized function, be explicit with your returned values. Similarly, do not
name result parameters just because it enables you to use naked returns.
[Clarity](guide#clarity) is always more important than saving a few lines in
your function.

It is always acceptable to name a result parameter if its value must be changed
in a deferred closure.

> **Tip:** Types can often be clearer than names in function signatures.
> [GoTip #38: Functions as Named Types] demonstrates this.
>
> In, [`WithTimeout`] above, the real code uses a [`CancelFunc`] instead of a
> raw `func()` in the result parameter list and requires little effort to
> document.

[Naked returns]: https://tour.golang.org/basics/7
[GoTip #38: Functions as Named Types]: https://google.github.io/styleguide/go/index.html#gotip
[`WithTimeout`]: https://pkg.go.dev/context#WithTimeout
[`CancelFunc`]: https://pkg.go.dev/context#CancelFunc

<a id="package-comments"></a>

### Package comments

<a id="TOC-PackageComments"></a>

Package comments must appear immediately above the package clause with no blank
line between the comment and the package name. Example:

```go
// 好的範例:
// Package math provides basic constants and mathematical functions.
//
// This package does not guarantee bit-identical results across architectures.
package math
```

There must be a single package comment per package. If a package is composed of
multiple files, exactly one of the files should have a package comment.

Comments for `main` packages have a slightly different form, where the name of
the `go_binary` rule in the BUILD file takes the place of the package name.

```go
// 好的範例:
// The seed_generator command is a utility that generates a Finch seed file
// from a set of JSON study configs.
package main
```

Other styles of comment are fine as long as the name of the binary is exactly as
written in the BUILD file. When the binary name is the first word, capitalizing
it is required even though it does not strictly match the spelling of the
command-line invocation.

```go
// 好的範例:
// Binary seed_generator ...
// Command seed_generator ...
// Program seed_generator ...
// The seed_generator command ...
// The seed_generator program ...
// Seed_generator ...
```

Tips:

*   Example command-line invocations and API usage can be useful documentation.
    For Godoc formatting, indent the comment lines containing code.

*   If there is no obvious primary file or if the package comment is
    extraordinarily long, it is acceptable to put the doc comment in a file
    named `doc.go` with only the comment and the package clause.

*   Multiline comments can be used instead of multiple single-line comments.
    This is primarily useful if the documentation contains sections which may be
    useful to copy and paste from the source file, as with sample command-lines
    (for binaries) and template examples.

    ```go
    // 好的範例:
    /*
    The seed_generator command is a utility that generates a Finch seed file
    from a set of JSON study configs.

        seed_generator *.json | base64 > finch-seed.base64
    */
    package template
    ```

*   Comments intended for maintainers and that apply to the whole file are
    typically placed after import declarations. These are not surfaced in Godoc
    and are not subject to the rules above on package comments.

<a id="imports"></a>

## Imports

<a id="TOC-Imports"></a>

<a id="import-renaming"></a>

### Import renaming

Imports should only be renamed to avoid a name collision with other imports. (A
corollary of this is that [good package names](#package-names) should not
require renaming.) In the event of a name collision, prefer to rename the most
local or project-specific import. Local names (aliases) for packages must follow
[the guidance around package naming](#package-names), including the prohibition
on the use of underscores and capital letters.

Generated protocol buffer packages must be renamed to remove underscores from
their names, and their aliases must have a `pb` suffix. See
[proto and stub best practices] for more information.

[proto and stub best practices]: best-practices#import-protos

```go
// 好的範例:
import (
    fspb "path/to/package/foo_service_go_proto"
)
```

Imports that have package names with no useful identifying information (e.g.
`package v1`) should be renamed to include the previous path component. The
rename must be consistent with other local files importing the same package and
may include the version number.

**Note:** It is preferred to rename the package to conform with
[good package names](#package-names), but that is often not feasible for
packages in vendored directories.

```go
// 好的範例:
import (
    core "github.com/kubernetes/api/core/v1"
    meta "github.com/kubernetes/apimachinery/pkg/apis/meta/v1beta1"
)
```

If you need to import a package whose name collides with a common local variable
name that you want to use (e.g. `url`, `ssh`) and you wish to rename the
package, the preferred way to do so is with the `pkg` suffix (e.g. `urlpkg`).
Note that it is possible to shadow a package with a local variable; this rename
is only necessary if the package still needs to be used when such a variable is
in scope.

<a id="import-grouping"></a>

### Import grouping

Imports should be organized in two groups:

*   Standard library packages

*   Other (project and vendored) packages

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

It is acceptable to split the project packages into multiple groups if you want
a separate group, as long as the groups have some meaning. Common reasons to do
this:

*   renamed imports
*   packages imported for their side-effects

Example:

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

**Note:** Maintaining optional groups - splitting beyond what is necessary for
the mandatory separation between standard library and Google imports - is not
supported by the [goimports] tool. Additional import subgroups require attention
on the part of both authors and reviewers to maintain in a conforming state.

[goimports]: golang.org/x/tools/cmd/goimports

Google programs that are also AppEngine apps should have a separate group for
AppEngine imports.

Gofmt takes care of sorting each group by import path. However, it does not
automatically separate imports into groups. The popular [goimports] tool
combines Gofmt and import management, separating imports into groups based on
the decision above. It is permissible to let [goimports] manage import
arrangement entirely, but as a file is revised its import list must remain
internally consistent.

<a id="import-blank"></a>

### Import "blank" (`import _`)

<a id="TOC-ImportBlank"></a>

Packages that are imported only for their side effects (using the syntax `import
_ "package"`) may only be imported in a main package, or in tests that require
them.

Some examples of such packages include:

*   [time/tzdata](https://pkg.go.dev/time/tzdata)

*   [image/jpeg](https://pkg.go.dev/image/jpeg) in image processing code

Avoid blank imports in library packages, even if the library indirectly depends
on them. Constraining side-effect imports to the main package helps control
dependencies, and makes it possible to write tests that rely on a different
import without conflict or wasted build costs.

The following are the only exceptions to this rule:

*   You may use a blank import to bypass the check for disallowed imports in the
    [nogo static checker].

*   You may use a blank import of the [embed](https://pkg.go.dev/embed) package
    in a source file which uses the `//go:embed` compiler directive.

**Tip:** If you create a library package that indirectly depends on a
side-effect import in production, document the intended usage.

[nogo static checker]: https://github.com/bazelbuild/rules_go/blob/master/go/nogo.rst

<a id="import-dot"></a>

### Import "dot" (`import .`)

<a id="TOC-ImportDot"></a>

The `import .` form is a language feature that allows bringing identifiers
exported from another package to the current package without qualification. See
the [language spec](https://go.dev/ref/spec#Import_declarations) for more.

Do **not** use this feature in the Google codebase; it makes it harder to tell
where the functionality is coming from.

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

## Errors

<a id="returning-errors"></a>

### Returning errors

<a id="TOC-ReturningErrors"></a>

Use `error` to signal that a function can fail. By convention, `error` is the
last result parameter.

```go
// 好的範例:
func Good() error { /* ... */ }
```

Returning a `nil` error is the idiomatic way to signal a successful operation
that could otherwise fail. If a function returns an error, callers must treat
all non-error return values as unspecified unless explicitly documented
otherwise. Commonly, the non-error return values are their zero values, but this
cannot be assumed.

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

Exported functions that return errors should return them using the `error` type.
Concrete error types are susceptible to subtle bugs: a concrete `nil` pointer
can get wrapped into an interface and thus become a non-nil value (see the
[Go FAQ entry on the topic][nil error]).

```go
// 不好的範例:
func Bad() *os.PathError { /*...*/ }
```

**Tip**: A function that takes a `context.Context` argument should usually
return an `error` so that the caller can determine if the context was cancelled
while the function was running.

[nil error]: https://golang.org/doc/faq#nil_error

<a id="error-strings"></a>

### Error strings

<a id="TOC-ErrorStrings"></a>

Error strings should not be capitalized (unless beginning with an exported name,
a proper noun or an acronym) and should not end with punctuation. This is
because error strings usually appear within other context before being printed
to the user.

```go
// 不好的範例:
err := fmt.Errorf("Something bad happened.")
```

```go
// 好的範例:
err := fmt.Errorf("something bad happened")
```

On the other hand, the style for the full displayed message (logging, test
failure, API response, or other UI) depends, but should typically be
capitalized.

```go
// 好的範例:
log.Infof("Operation aborted: %v", err)
log.Errorf("Operation aborted: %v", err)
t.Errorf("Op(%q) failed unexpectedly; err=%v", args, err)
```

<a id="handle-errors"></a>

### Handle errors

<a id="TOC-HandleErrors"></a>

Code that encounters an error should make a deliberate choice about how to
handle it. It is not usually appropriate to discard errors using `_` variables.
If a function returns an error, do one of the following:

*   Handle and address the error immediately.
*   Return the error to the caller.
*   In exceptional situations, call [`log.Fatal`] or (if absolutely necessary)
    `panic`.

**Note:** `log.Fatalf` is not the standard library log. See [#logging].

In the rare circumstance where it is appropriate to ignore or discard an error
(for example a call to [`(*bytes.Buffer).Write`] that is documented to never
fail), an accompanying comment should explain why this is safe.

```go
// 好的範例:
var b *bytes.Buffer

n, _ := b.Write(p) // never returns a non-nil error
```

For more discussion and examples of error handling, see
[Effective Go](http://golang.org/doc/effective_go.html#errors) and
[best practices](best-practices.md#error-handling).

[`(*bytes.Buffer).Write`]: https://pkg.go.dev/bytes#Buffer.Write

<a id="in-band-errors"></a>

### In-band errors

<a id="TOC-In-Band-Errors"></a>

In C and similar languages, it is common for functions to return values like -1,
null, or the empty string to signal errors or missing results. This is known as
in-band error handling.

```go
// 不好的範例:
// Lookup returns the value for key or -1 if there is no mapping for key.
func Lookup(key string) int
```

Failing to check for an in-band error value can lead to bugs and can attribute
errors to the wrong function.

```go
// 不好的範例:
// The following line returns an error that Parse failed for the input value,
// whereas the failure was that there is no mapping for missingKey.
return Parse(Lookup(missingKey))
```

Go's support for multiple return values provides a better solution (see the
[Effective Go section on multiple returns]). Instead of requiring clients to
check for an in-band error value, a function should return an additional value
to indicate whether its other return values are valid. This return value may be
an error or a boolean when no explanation is needed, and should be the final
return value.

```go
// 好的範例:
// Lookup returns the value for key or ok=false if there is no mapping for key.
func Lookup(key string) (value string, ok bool)
```

This API prevents the caller from incorrectly writing `Parse(Lookup(key))` which
causes a compile-time error, since `Lookup(key)` has 2 outputs.

Returning errors in this way encourages more robust and explicit error handling:

```go
// 好的範例:
value, ok := Lookup(key)
if !ok {
    return fmt.Errorf("no value for %q", key)
}
return Parse(value)
```

Some standard library functions, like those in package `strings`, return in-band
error values. This greatly simplifies string-manipulation code at the cost of
requiring more diligence from the programmer. In general, Go code in the Google
codebase should return additional values for errors.

[Effective Go section on multiple returns]: http://golang.org/doc/effective_go.html#multiple-returns

<a id="indent-error-flow"></a>

### Indent error flow

<a id="TOC-IndentErrorFlow"></a>

Handle errors before proceeding with the rest of your code. This improves the
readability of the code by enabling the reader to find the normal path quickly.
This same logic applies to any block which tests a condition then ends in a
terminal condition (e.g., `return`, `panic`, `log.Fatal`).

Code that runs if the terminal condition is not met should appear after the `if`
block, and should not be indented in an `else` clause.

```go
// 好的範例:
if err != nil {
    // error handling
    return // or continue, etc.
}
// normal code
```

```go
// 不好的範例:
if err != nil {
    // error handling
} else {
    // normal code that looks abnormal due to indentation
}
```

> **Tip:** If you are using a variable for more than a few lines of code, it is
> generally not worth using the `if`-with-initializer style. In these cases, it
> is usually better to move the declaration out and use a standard `if`
> statement:
>
> ```go
> // 好的範例:
> x, err := f()
> if err != nil {
>   // error handling
>   return
> }
> // lots of code that uses x
> // across multiple lines
> ```
>
> ```go
> // 不好的範例:
> if x, err := f(); err != nil {
>   // error handling
>   return
> } else {
>   // lots of code that uses x
>   // across multiple lines
> }
> ```

See [Go Tip #1: Line of Sight] and
[TotT: Reduce Code Complexity by Reducing Nesting](https://testing.googleblog.com/2017/06/code-health-reduce-nesting-reduce.html)
for more details.

[Go Tip #1: Line of Sight]: https://google.github.io/styleguide/go/index.html#gotip

<a id="language"></a>

## Language

<a id="literal-formatting"></a>

### Literal formatting

Go has an exceptionally powerful [composite literal syntax], with which it is
possible to express deeply-nested, complicated values in a single expression.
Where possible, this literal syntax should be used instead of building values
field-by-field. The `gofmt` formatting for literals is generally quite good, but
there are some additional rules for keeping these literals readable and
maintainable.

[composite literal syntax]: https://golang.org/ref/spec#Composite_literals

<a id="literal-field-names"></a>

#### Field names

Struct literals must specify **field names** for types defined outside the
current package.

*   Include field names for types from other packages.

    ```go
    // 好的範例:
    // https://pkg.go.dev/encoding/csv#Reader
    r := csv.Reader{
      Comma: ',',
      Comment: '#',
      FieldsPerRecord: 4,
    }
    ```

    The position of fields in a struct and the full set of fields (both of which
    are necessary to get right when field names are omitted) are not usually
    considered to be part of a struct's public API; specifying the field name is
    needed to avoid unnecessary coupling.

    ```go
    // 不好的範例:
    r := csv.Reader{',', '#', 4, false, false, false, false}
    ```

*   For package-local types, field names are optional.

    ```go
    // 好的範例:
    okay := Type{42}
    also := internalType{4, 2}
    ```

    Field names should still be used if it makes the code clearer, and it is
    very common to do so. For example, a struct with a large number of fields
    should almost always be initialized with field names.

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

#### Matching braces

The closing half of a brace pair should always appear on a line with the same
amount of indentation as the opening brace. One-line literals necessarily have
this property. When the literal spans multiple lines, maintaining this property
keeps the brace matching for literals the same as brace matching for common Go
syntactic constructs like functions and `if` statements.

The most common mistake in this area is putting the closing brace on the same
line as a value in a multi-line struct literal. In these cases, the line should
end with a comma and the closing brace should appear on the next line.

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

#### Cuddled braces

Dropping whitespace between braces (aka "cuddling" them) for slice and array
literals is only permitted when both of the following are true.

*   The [indentation matches](#literal-matching-braces)
*   The inner values are also literals or proto builders (i.e. not a variable or
    other expression)

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

#### Repeated type names

Repeated type names may be omitted from slice and map literals. This can be
helpful in reducing clutter. A reasonable occasion for repeating the type names
explicitly is when dealing with a complex type that is not common in your
project, when the repetitive type names are on lines that are far apart and can
remind the reader of the context.

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

**Tip:** If you want to remove repetitive type names in struct literals, you can
run `gofmt -s`.

<a id="literal-zero-value-fields"></a>

#### Zero-value fields

[Zero-value] fields may be omitted from struct literals when clarity is not lost
as a result.

Well-designed APIs often employ zero-value construction for enhanced
readability. For example, omitting the three zero-value fields from the
following struct draws attention to the only option that is being specified.

[Zero-value]: https://golang.org/ref/spec#The_zero_value

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

Structs within table-driven tests often benefit from [explicit field names],
especially when the test struct is not trivial. This allows the author to omit
the zero-valued fields entirely when the fields in question are not related to
the test case. For example, successful test cases should omit any error-related
or failure-related fields. In cases where the zero value is necessary to
understand the test case, such as testing for zero or `nil` inputs, the field
names should be specified.

[explicit field names]: #literal-field-names

**Concise**

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

**Explicit**

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

### Nil slices

For most purposes, there is no functional difference between `nil` and the empty
slice. Built-in functions like `len` and `cap` behave as expected on `nil`
slices.

```go
// 好的範例:
import "fmt"

var s []int         // nil

fmt.Println(s)      // []
fmt.Println(len(s)) // 0
fmt.Println(cap(s)) // 0
for range s {...}   // no-op

s = append(s, 42)
fmt.Println(s)      // [42]
```

If you declare an empty slice as a local variable (especially if it can be the
source of a return value), prefer the nil initialization to reduce the risk of
bugs by callers.

```go
// 好的範例:
var t []string
```

```go
// 不好的範例:
t := []string{}
```

Do not create APIs that force their clients to make distinctions between nil and
the empty slice.

```go
// 好的範例:
// Ping pings its targets.
// Returns hosts that successfully responded.
func Ping(hosts []string) ([]string, error) { ... }
```

```go
// 不好的範例:
// Ping pings its targets and returns a list of hosts
// that successfully responded. Can be empty if the input was empty.
// nil signifies that a system error occurred.
func Ping(hosts []string) []string { ... }
```

When designing interfaces, avoid making a distinction between a `nil` slice and
a non-`nil`, zero-length slice, as this can lead to subtle programming errors.
This is typically accomplished by using `len` to check for emptiness, rather
than `== nil`.

This implementation accepts both `nil` and zero-length slices as "empty":

```go
// 好的範例:
// describeInts describes s with the given prefix, unless s is empty.
func describeInts(prefix string, s []int) {
    if len(s) == 0 {
        return
    }
    fmt.Println(prefix, s)
}
```

Instead of relying on the distinction as a part of the API:

```go
// 不好的範例:
func maybeInts() []int { /* ... */ }

// describeInts describes s with the given prefix; pass nil to skip completely.
func describeInts(prefix string, s []int) {
  // The behavior of this function unintentionally changes depending on what
  // maybeInts() returns in 'empty' cases (nil or []int{}).
  if s == nil {
    return
  }
  fmt.Println(prefix, s)
}

describeInts("Here are some ints:", maybeInts())
```

See [in-band errors] for further discussion.

[in-band errors]: #in-band-errors

<a id="indentation-confusion"></a>

### Indentation confusion

Avoid introducing a line break if it would align the rest of the line with an
indented code block. If this is unavoidable, leave a space to separate the code
in the block from the wrapped line.

```go
// 不好的範例:
if longCondition1 && longCondition2 &&
    // Conditions 3 and 4 have the same indentation as the code within the if.
    longCondition3 && longCondition4 {
    log.Info("all conditions met")
}
```

See the following sections for specific guidelines and examples:

*   [Function formatting](#func-formatting)
*   [Conditionals and loops](#conditional-formatting)
*   [Literal formatting](#literal-formatting)

<a id="func-formatting"></a>

### Function formatting

The signature of a function or method declaration should remain on a single line
to avoid [indentation confusion](#indentation-confusion).

Function argument lists can make some of the longest lines in a Go source file.
However, they precede a change in indentation, and therefore it is difficult to
break the line in a way that does not make subsequent lines look like part of
the function body in a confusing way:

```go
// 不好的範例:
func (r *SomeType) SomeLongFunctionName(foo1, foo2, foo3 string,
    foo4, foo5, foo6 int) {
    foo7 := bar(foo1)
    // ...
}
```

See [best practices](best-practices#funcargs) for a few options for shortening
the call sites of functions that would otherwise have many arguments.

Lines can often be shortened by factoring out local variables.

```go
// 好的範例:
local := helper(some, parameters, here)
good := foo.Call(list, of, parameters, local)
```

Similarly, function and method calls should not be separated based solely on
line length.

```go
// 好的範例:
good := foo.Call(long, list, of, parameters, all, on, one, line)
```

```go
// 不好的範例:
bad := foo.Call(long, list, of, parameters,
    with, arbitrary, line, breaks)
```

Avoid adding inline comments to specific function arguments where possible.
Instead, use an [option struct](best-practices#option-structure) or add more
detail to the function documentation.

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

If the API cannot be changed or if the local call is unusual (whether or not the
call is too long), it is always permissible to add line breaks if it aids in
understanding the call.

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

Note that the lines in the above example are not wrapped at a specific column
boundary but are grouped based on co-ordinate triples.

Long string literals within functions should not be broken for the sake of line
length. For functions that include such strings, a line break can be added after
the string format, and the arguments can be provided on the next or subsequent
lines. The decision about where the line breaks should go is best made based on
semantic groupings of inputs, rather than based purely on line length.

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

### Conditionals and loops

An `if` statement should not be line broken; multi-line `if` clauses can lead to
[indentation confusion](#indentation-confusion).

```go
// 不好的範例:
// The second if statement is aligned with the code within the if block, causing
// indentation confusion.
if db.CurrentStatusIs(db.InTransaction) &&
    db.ValuesEqual(db.TransactionKey(), row.Key()) {
    return db.Errorf(db.TransactionError, "query failed: row (%v): key does not match transaction key", row)
}
```

If the short-circuit behavior is not required, the boolean operands can be
extracted directly:

```go
// 好的範例:
inTransaction := db.CurrentStatusIs(db.InTransaction)
keysMatch := db.ValuesEqual(db.TransactionKey(), row.Key())
if inTransaction && keysMatch {
    return db.Error(db.TransactionError, "query failed: row (%v): key does not match transaction key", row)
}
```

There may also be other locals that can be extracted, especially if the
conditional is already repetitive:

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

`if` statements that contain closures or multi-line struct literals should
ensure that the [braces match](#literal-matching-braces) to avoid
[indentation confusion](#indentation-confusion).

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

Similarly, don't try inserting artificial linebreaks into `for` statements. You
can always let the line simply be long if there is no elegant way to refactor
it:

```go
// 好的範例:
for i, max := 0, collection.Size(); i < max && !collection.HasPendingWriters(); i++ {
    // ...
}
```

Often, though, there is:

```go
// 好的範例:
for i, max := 0, collection.Size(); i < max; i++ {
    if collection.HasPendingWriters() {
        break
    }
    // ...
}
```

`switch` and `case` statements should also remain on a single line.

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

If the line is excessively long, indent all cases and separate them with a blank
line to avoid [indentation confusion](#indentation-confusion):

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

In conditionals comparing a variable to a constant, place the variable value on
the left hand side of the equality operator:

```go
// 好的範例:
if result == "foo" {
  // ...
}
```

Instead of the less clear phrasing where the constant comes first
(["Yoda style conditionals"](https://en.wikipedia.org/wiki/Yoda_conditions)):

```go
// 不好的範例:
if "foo" == result {
  // ...
}
```

<a id="copying"></a>

### Copying

<a id="TOC-Copying"></a>

To avoid unexpected aliasing and similar bugs, be careful when copying a struct
from another package. For example, synchronization objects such as `sync.Mutex`
must not be copied.

The `bytes.Buffer` type contains a `[]byte` slice and, as an optimization for
small strings, a small byte array to which the slice may refer. If you copy a
`Buffer`, the slice in the copy may alias the array in the original, causing
subsequent method calls to have surprising effects.

In general, do not copy a value of type `T` if its methods are associated with
the pointer type, `*T`.

```go
// 不好的範例:
b1 := bytes.Buffer{}
b2 := b1
```

Invoking a method that takes a value receiver can hide the copy. When you author
an API, you should generally take and return pointer types if your structs
contain fields that should not be copied.

These are acceptable:

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

But these are usually wrong:

```go
// 不好的範例:
type Record struct {
  buf bytes.Buffer
  // other fields omitted
}


func (r Record) Process(...) {...} // Makes a copy of r.buf

func Consumer(r Record) {...} // Makes a copy of r.buf
```

This guidance also applies to copying `sync.Mutex`.

<a id="dont-panic"></a>

### Don't panic

<a id="TOC-Don-t-Panic"></a>

Do not use `panic` for normal error handling. Instead, use `error` and multiple
return values. See the [Effective Go section on errors].

Within `package main` and initialization code, consider [`log.Exit`] for errors
that should terminate the program (e.g., invalid configuration), as in many of
these cases a stack trace will not help the reader. Please note that
[`log.Exit`] calls [`os.Exit`] and any deferred functions will not be run.

For errors that indicate "impossible" conditions, namely bugs that should always
be caught during code review and/or testing, a function may reasonably return an
error or call [`log.Fatal`].

**Note:** `log.Fatalf` is not the standard library log. See [#logging].

[Effective Go section on errors]: http://golang.org/doc/effective_go.html#errors
[`os.Exit`]: https://pkg.go.dev/os#Exit

<a id="must-functions"></a>

### Must functions

Setup helper functions that stop the program on failure follow the naming
convention `MustXYZ` (or `mustXYZ`). In general, they should only be called
early on program startup, not on things like user input where normal Go error
handling is preferred.

This often comes up for functions called to initialize package-level variables
exclusively at
[package initialization time](https://golang.org/ref/spec#Package_initialization)
(e.g. [template.Must](https://golang.org/pkg/text/template/#Must) and
[regexp.MustCompile](https://golang.org/pkg/regexp/#MustCompile)).

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

The same convention may be used in test helpers that only stop the current test
(using `t.Fatal`). Such helpers are often convenient in creating test values,
for example in struct fields of [table driven tests](#table-driven-tests), as
functions that return errors cannot be directly assigned to a struct field.

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

In both of these cases, the value of this pattern is that the helpers can be
called in a "value" context. These helpers should not be called in places where
it's difficult to ensure an error would be caught or in a context where an error
should be [checked](#handle-errors) (e.g., in many request handlers). For
constant inputs, this allows tests to easily ensure that the `Must` arguments
are well-formed, and for non-constant inputs it permits tests to validate that
errors are [properly handled or propagated](best-practices#error-handling).

Where `Must` functions are used in a test, they should generally be
[marked as a test helper](#mark-test-helpers) and call `t.Fatal` on error (see
[error handling in test helpers](best-practices#test-helper-error-handling) for
more considerations of using that).

They should not be used when
[ordinary error handling](best-practices#error-handling) is possible (including
with some refactoring):

```go
// 不好的範例:
func Version(o *servicepb.Object) (*version.Version, error) {
    // Return error instead of using Must functions.
    v := version.MustParse(o.GetVersionString())
    return dealiasVersion(v)
}
```

<a id="goroutine-lifetimes"></a>

### Goroutine lifetimes

<a id="TOC-GoroutineLifetimes"></a>

When you spawn goroutines, make it clear when or whether they exit.

Goroutines can leak by blocking on channel sends or receives. The garbage
collector will not terminate a goroutine blocked on a channel even if no other
goroutine has a reference to the channel.

Even when goroutines do not leak, leaving them in-flight when they are no longer
needed can cause other subtle and hard-to-diagnose problems. Sending on a
channel that has been closed causes a panic.

```go
// 不好的範例:
ch := make(chan int)
ch <- 42
close(ch)
ch <- 13 // panic
```

Modifying still-in-use inputs "after the result isn't needed" can lead to data
races. Leaving goroutines in-flight for arbitrarily long can lead to
unpredictable memory usage.

Concurrent code should be written such that the goroutine lifetimes are obvious.
Typically this will mean keeping synchronization-related code constrained within
the scope of a function and factoring out the logic into
[synchronous functions]. If the concurrency is still not obvious, it is
important to document when and why the goroutines exit.

Code that follows best practices around context usage often helps make this
clear. It is conventionally managed with a `context.Context`:

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

There are other variants of the above that use raw signal channels like `chan
struct{}`, synchronized variables, [condition variables][rethinking-slides], and
more. The important part is that the goroutine's end is evident for subsequent
maintainers.

In contrast, the following code is careless about when its spawned goroutines
finish:

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

This code may look OK, but there are several underlying problems:

*   The code probably has undefined behavior in production, and the program may
    not terminate cleanly, even if the operating system releases the resources.

*   The code is difficult to test meaningfully due to the code's indeterminate
    lifecycle.

*   The code may leak resources as described above.

See also:

*   [Never start a goroutine without knowing how it will stop][cheney-stop]
*   Rethinking Classical Concurrency Patterns: [slides][rethinking-slides],
    [video][rethinking-video]
*   [When Go programs end]

[synchronous functions]: #synchronous-functions
[cheney-stop]: https://dave.cheney.net/2016/12/22/never-start-a-goroutine-without-knowing-how-it-will-stop
[rethinking-slides]: https://drive.google.com/file/d/1nPdvhB0PutEJzdCq5ms6UI58dp50fcAN/view
[rethinking-video]: https://www.youtube.com/watch?v=5zXAHh5tJqQ
[When Go programs end]: https://changelog.com/gotime/165

<a id="interfaces"></a>

### Interfaces

<a id="TOC-Interfaces"></a>

Go interfaces generally belong in the package that *consumes* values of the
interface type, not a package that *implements* the interface type. The
implementing package should return concrete (usually pointer or struct) types.
That way, new methods can be added to implementations without requiring
extensive refactoring. See [GoTip #49: Accept Interfaces, Return Concrete Types]
for more details.

Do not export a [test double][double types] implementation of an interface from
an API that consumes it. Instead, design the API so that it can be tested using
the [public API] of the [real implementation]. See
[GoTip #42: Authoring a Stub for Testing] for more details. Even when it is not
feasible to use the real implementation, it may not be necessary to introduce an
interface fully covering all methods in the real type; the consumer can create
an interface containing only the methods it needs, as demonstrated in
[GoTip #78: Minimal Viable Interfaces].

To test packages that use Stubby RPC clients, use a real client connection. If a
real server cannot be run in the test, Google's internal practice is to obtain a
real client connection to a local [test double] using the internal rpctest
package (coming soon!).

Do not define interfaces before they are used (see
[TotT: Code Health: Eliminate YAGNI Smells][tott-438] ). Without a realistic
example of usage, it is too difficult to see whether an interface is even
necessary, let alone what methods it should contain.

Do not use interface-typed parameters if the users of the package do not need to
pass different types for them.

Do not export interfaces that the users of the package do not need.

**TODO:** Write a more in-depth doc on interfaces and link to it here.

[GoTip #42: Authoring a Stub for Testing]: https://google.github.io/styleguide/go/index.html#gotip
[GoTip #49: Accept Interfaces, Return Concrete Types]: https://google.github.io/styleguide/go/index.html#gotip
[GoTip #78: Minimal Viable Interfaces]: https://google.github.io/styleguide/go/index.html#gotip
[real implementation]: best-practices#use-real-transports
[public API]: https://abseil.io/resources/swe-book/html/ch12.html#test_via_public_apis
[double types]: https://abseil.io/resources/swe-book/html/ch13.html#techniques_for_using_test_doubles
[test double]: https://abseil.io/resources/swe-book/html/ch13.html#basic_concepts
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

### Generics

Generics (formally called "[Type Parameters]") are allowed where they fulfill
your business requirements. In many applications, a conventional approach using
existing language features (slices, maps, interfaces, and so on) works just as
well without the added complexity, so be wary of premature use. See the
discussion on [least mechanism](guide#least-mechanism).

When introducing an exported API that uses generics, make sure it is suitably
documented. It's highly encouraged to include motivating runnable [examples].

Do not use generics just because you are implementing an algorithm or data
structure that does not care about the type of its member elements. If there is
only one type being instantiated in practice, start by making your code work on
that type without using generics at all. Adding polymorphism later will be
straightforward compared to removing abstraction that is found to be
unnecessary.

Do not use generics to invent domain-specific languages (DSLs). In particular,
refrain from introducing error-handling frameworks that might put a significant
burden on readers. Instead prefer established [error handling](#errors)
practices. For testing, be especially wary of introducing
[assertion libraries](#assert) or frameworks that result in less useful
[test failures](#useful-test-failures).

In general:

*   [Write code, don't design types]. From a GopherCon talk by Robert Griesemer
    and Ian Lance Taylor.
*   If you have several types that share a useful unifying interface, consider
    modeling the solution using that interface. Generics may not be needed.
*   Otherwise, instead of relying on the `any` type and excessive
    [type switching](https://tour.golang.org/methods/16), consider generics.

See also:

*   [Using Generics in Go], talk by Ian Lance Taylor

*   [Generics tutorial] on Go's webpage

[Generics tutorial]: https://go.dev/doc/tutorial/generics
[Type Parameters]: https://go.dev/design/43651-type-parameters
[Using Generics in Go]: https://www.youtube.com/watch?v=nr8EpUO9jhw
[Write code, don't design types]: https://www.youtube.com/watch?v=Pa_e9EeCdy8&t=1250s

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
